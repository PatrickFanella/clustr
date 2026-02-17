---
title: "Tooling, DX, and Engineering Process"
series: "Cutroom Devlog"
part: 5
date: "2026-02-17"
tags: ["devlog", "openwork", "cutroom", "dx", "ci-cd", "testing", "tooling", "makefile"]
summary: "CI pipelines, test infrastructure, build tooling, and what's still missing."
---

## Context

Part 5 of the Cutroom Devlog series. What tooling exists, when it arrived, and what's still missing.

## What We Did

### The Makefile: Single Entry Point

The build system lives in a single root `Makefile` (468 lines, refactored in commit `4eadd22`, Nov 12, 2025). Running `make help` shows ~40 targets organized by category:

**Setup:** `setup`, `check-env`, `check-tools`, `install-tools`
**Docker:** `up`, `down`, `restart`, `rebuild`, `logs-*`, `ps`
**Database:** `migrate-up`, `migrate-down`, `migrate-up-host`, `db-reset`, `db-shell`
**Backend dev:** `dev-backend`, `test-backend`, `test-backend-coverage`, `lint-backend`, `fmt-backend`
**Frontend dev:** `dev-frontend`, `build-frontend`, `test-frontend`, `lint-frontend`
**Code generation:** `generate` (runs `sqlc generate`)
**Data operations:** `crawl`, `precalculate`, `seed`
**Load testing:** `loadtest-smoke`, `loadtest-load`, `loadtest-stress`, `loadtest-soak`
**Monitoring:** `monitoring-up`, `monitoring-down`
**Backups:** `backup-now`, `backups-list`, `backups-download`
**Cleanup:** `clean`, `clean-volumes`, `prune`
**Quick start:** `quickstart` (setup + up + migrate-up)

This Makefile started as 14 lines in `backend/Makefile` (commit `e8ce97f`, May 31) and was moved to the root with unified commands in November. The Copilot agent expanded it during the October sprint (PR #60: enhanced Makefile, dev scripts, pre-commit hooks).

### CI/CD Pipeline

**Timeline of CI evolution:**

| Date | Commit/PR | What Changed |
|------|-----------|-------------|
| Jun 2 | `08c53e5` | First deploy workflow (`.github/workflows/deploy.yml`) |
| Oct 17 | `8130589` | CI workflow added (`ci.yml`) |
| Oct 17 | PR #56 | CI/CD documentation |
| Oct 28 | PR #119 | Security scanning workflow (`security.yml`) |
| Nov 1 | PR #132 | Release process, `publish.yml`, `release.yml` |
| Feb 8 | PR #201 | Bundle size tracking (`bundle-size.yml`) |
| Feb 8 | — | Benchmark workflow (`benchmark.yml`) |

**Current CI workflow (`ci.yml`)** runs on push to `main`/`uplink/deploy-branch` and PRs:

1. **test-backend:** Go 1.24 + PostgreSQL 17 service container. Runs `go mod verify`, `go vet`, `govulncheck`, and `go test -v -race -coverprofile`. Uploads coverage to Codecov.

2. **test-frontend:** Node 20 + `npm ci` + Vitest (`npm run test:run`) with coverage. Also uploads to Codecov.

3. **visual-regression:** Playwright visual snapshot tests against committed baseline images.

4. **build-backend-images:** Matrix build of 3 Docker images (server, crawler, precalculate).

5. **build-frontend-image:** Frontend Docker image build.

**Security scanning (`security.yml`)** runs on push, PR, and daily at 2 AM UTC:
- CodeQL analysis for Go and JavaScript/TypeScript
- `govulncheck` for Go dependencies
- `npm audit` for frontend dependencies
- TruffleHog secret scanning (verified secrets only)
- Trivy Docker image scanning (CRITICAL/HIGH severity)

**Release pipeline (`publish.yml`):**
- Triggered on `v*.*.*` tags or manual dispatch
- Builds multi-arch Docker images (amd64 + arm64)
- Pushes to GitHub Container Registry (GHCR)

### Test Infrastructure

Backend tests arrived late but are now solid:

```
$ go test ./...
# All packages pass. 27 packages tested, 6 have no test files.
# Coverage includes: api, handlers, apierr, cache, circuitbreaker, config,
# crawler, errorreporting, graph, httpx, integrity, logger, metrics,
# middleware, redditapi, scheduler, secrets, server, tracing, utils
```

Backend test coverage gaps (packages with `[no test files]`):
- `cmd/crawler`, `cmd/integrity`, `cmd/precalculate`, `cmd/server` (binary entrypoints)
- `internal/admin`, `internal/authstore`, `internal/db` (generated code)

Frontend tests have broader coverage but more fragility:

```
$ cd frontend && npx vitest run
# Test Files: 8 failed | 26 passed (34 total)
# Tests: 19 failed | 400 passed (419 total)
```

The 19 failing tests break down into:
- Performance benchmarks (timing-dependent): `InstancedNodeRenderer.test.ts` and `Octree.test.ts` have tests like "should complete position updates in <50ms" that fail when system load varies. Inherently flaky.
- DOM query failures: `Sidebar.test.tsx` expects an ARIA label `"Collapse sidebar"` that doesn't match the rendered output. Likely a component change that wasn't reflected in tests.
- Accessibility tests: Some ARIA attribute expectations don't match the current component implementation.

Frontend test tooling:
- **Vitest 4.0** with jsdom environment for unit tests
- **Playwright 1.56** for E2E tests with visual snapshot baselines (11 snapshot images in `e2e/visual.spec.ts-snapshots/`)
- **@testing-library/react** for component testing
- **jest-axe + axe-core** for accessibility testing
- **Coverage thresholds:** 75% lines, 60% functions, 70% branches (enforced in Vitest config)
- **Performance benchmarks:** Playwright-based suite in `frontend/benchmarks/` with fixture generation and baseline comparison

### Code Generation (sqlc)

The sqlc pipeline has been stable since day one:

1. Write SQL queries in `backend/internal/queries/*.sql`
2. Run `make generate` (which runs `sqlc generate`)
3. Generated Go code appears in `backend/internal/db/`

The sqlc configuration (`backend/sqlc.yaml`) reads `migrations/schema.sql` plus all numbered migration files. So the generated code stays in sync with the full schema.

**Current query files:** `graph.sql`, `graph_data.sql`, `comments.sql`, `posts.sql`, `subreddits.sql`, `users.sql`, `crawl_jobs.sql`, `admin.sql`, `admin_jobs.sql`, `integrity.sql`, `metrics.sql`, `oauth.sql`, `scheduled_jobs.sql`

### Environment Configuration

**`.env.example` files** were added in PR #66 (Oct 18):
- `backend/.env.example` — 4,289 bytes, covering all backend config (Reddit OAuth, database, rate limiting, graph parameters, monitoring, security, observability)
- `frontend/.env.example` — 591 bytes, covering `VITE_API_URL` and rendering caps

**Pre-commit hooks** were added in PR #60:
- `scripts/pre-commit` — runs `go fmt` and `go vet` on staged Go files
- `scripts/install-hooks.sh` — hook installation script

### Load Testing

k6 load testing infrastructure was added in PR #244 (Feb 11, 2026):
- `backend/loadtest/smoke.js` — 30-second quick validation
- `backend/loadtest/load.js` — 50 VUs for 5 minutes
- `backend/loadtest/stress.js` — Ramping up to failure point
- `backend/loadtest/soak.js` — Extended duration test
- `backend/loadtest/common.js` — Shared configuration and helpers
- `backend/loadtest/validate.sh` — Results validation script
- `backend/docker-compose.loadtest.yml` — k6 service container

### Bundle Size Tracking

Added in PR #201 (Feb 8, 2026):
- `frontend/.size-limit.json` — size budgets per chunk
- `size-limit` + `@size-limit/preset-app` + `@size-limit/file` packages
- `rollup-plugin-visualizer` for bundle analysis
- CI gate that fails on regression

Current build output:
```
dist/index.html                0.91 kB │ gzip:   0.50 kB
dist/assets/layoutWorker.js   15.52 kB
dist/assets/index.css         34.34 kB │ gzip:   6.60 kB
dist/assets/index.js       1,761.47 kB │ gzip: 499.14 kB
```

The main bundle (1.76 MB, 499 KB gzipped) exceeds Vite's 500 KB warning threshold due to Three.js and D3.js.

## Key Decisions

1. No tests for 4.5 months. From May 31 to October 17 (commit `8130589`), the project had zero CI and zero tests. Fast iteration, but every change was a trust exercise.

2. Copilot-generated test infrastructure. The test suites were largely written by the Copilot SWE agent (PRs #124, #202, #243, #244, #246). This produced broad coverage quickly, but created tests that the human developer(s) may not fully understand or maintain.

3. Timing-dependent performance tests. Tests like "should update 100k node positions in <50ms" are aspirational documentation pretending to be tests. They pass on fast machines and fail on slow ones.

4. Daily security scanning. The `security.yml` workflow runs CodeQL, govulncheck, npm audit, TruffleHog, and Trivy every day at 2 AM UTC. Probably noisy if nobody's watching the alerts.

## What Went Wrong / Friction

- Frontend tests are partially broken. 8 of 34 test files fail (19 of 419 tests). The failures are a mix of timing issues, stale DOM expectations, and accessibility assertion mismatches. These likely accumulated as components were modified without updating the corresponding tests.

- No linting enforcement in CI initially. ESLint was installed in the frontend from day one but wasn't enforced in CI until October. Backend linting (`go vet`, `go fmt`) was added around the same time.

- Bundle size regression. At 1.76 MB uncompressed (499 KB gzipped), the main JavaScript bundle exceeds recommended thresholds. Three.js dominates. Code splitting via dynamic imports would help but isn't implemented.

- Missing from DX: no hot-reload for the backend (requires manual restart), no unified `make dev` that starts both backend and frontend, no database seed data that creates a meaningful test graph.

## What We Learned

- You can ship for months without CI or tests, but the cost comes due. The October sprint spent real effort on bug fixes that tests would have caught earlier.

- The Makefile-as-documentation pattern works. Running `make help` is faster than reading a setup guide, and the target names become the vocabulary for the project's operations.

- Performance benchmarks in tests are fragile. Use relative comparisons (regression detection) or generous thresholds. Absolute timing assertions break across environments.

- sqlc's code generation workflow was the most stable part of the toolchain. 28 schema migrations and the `make generate` pipeline never broke once.

## Receipts

- **Root Makefile:** `Makefile` — 468 lines, 40+ targets
- **CI workflow:** `.github/workflows/ci.yml` — 5 jobs (backend test, frontend test, visual regression, backend images, frontend image)
- **Security workflow:** `.github/workflows/security.yml` — 7 jobs (CodeQL x2, govulncheck, npm audit, TruffleHog, Trivy, summary)
- **Benchmark workflow:** `.github/workflows/benchmark.yml`
- **First CI commit:** `8130589` (Oct 17, 2025) — "CI"
- **Backend test results:** All passing, 27 packages, 6 with no test files
- **Frontend test results:** 8/34 files failed, 19/419 tests failed, 400/419 passed
- **Build output:** Frontend builds successfully (10.19s), Go backend builds successfully (no errors)
- **Bundle size:** 1,761 KB main bundle (499 KB gzipped)
- **sqlc config:** `backend/sqlc.yaml`
- **`.env.example` files:** `backend/.env.example` (4,289 bytes), `frontend/.env.example` (591 bytes)
- **Load test scripts:** `backend/loadtest/smoke.js`, `load.js`, `stress.js`, `soak.js`
- **Pre-commit hooks:** `scripts/pre-commit`, `scripts/install-hooks.sh`
- **Coverage thresholds:** Vitest config — 75% lines, 60% functions, 70% branches
- **PR #60:** Enhanced Makefile and dev scripts (Oct 18, 2025)
- **PR #124:** Test infrastructure (Oct 29, 2025)
- **PR #201:** Bundle size tracking (Feb 8, 2026)
- **PR #202:** Frontend test suite, 78% coverage (Feb 8, 2026)
- **PR #244:** k6 load testing (Feb 11, 2026)
