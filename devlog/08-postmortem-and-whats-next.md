---
title: "Postmortem + What's Next (Actionable Roadmap)"
series: "Cutroom Devlog"
part: 8
date: "2026-02-17"
tags: ["devlog", "openwork", "cutroom", "postmortem", "roadmap", "next-steps"]
summary: "Where the project stands today, what to fix first, and how to present it."
---

## Context

Part 8 (final) of the Cutroom Devlog series. Where the project stands as of February 17, 2026, what it would take to revive active development, and concrete next steps.

## What We Did

### Current State Assessment

What works:
- Go backend compiles and all tests pass (21 test packages, 0 failures)
- Frontend builds (10.19s build time)
- 400 of 419 frontend tests pass (95.5%)
- Docker Compose defines a complete deployment stack (8 services)
- CI pipeline covers backend tests, frontend tests, visual regression, and Docker builds
- Security scanning runs daily (CodeQL, govulncheck, npm audit, TruffleHog, Trivy)
- ~60 documentation files in `docs/`
- `.env.example` files for both backend and frontend
- Load testing infrastructure (k6, 4 test scenarios)

What's broken or incomplete:
- 19 frontend tests fail (performance timing, stale DOM expectations, accessibility assertions)
- Main JavaScript bundle is 1.76 MB (499 KB gzipped), no code splitting
- NDJSON streaming carries a `[WIP]` tag (PR #224)
- Several open issues from the February roadmap remain unaddressed (#138, #139, #140, #141, #142, #144)
- No live demo or public deployment
- `react-force-graph-3d` still in dependencies alongside the custom InstancedMesh renderer. Two rendering paths, no clear winner.
- 2 open Copilot PRs (#247: WebSocket endpoint, #248: SLO/SLI tracking) still pending

What's missing:
- No React Router, all routing is manual state in `App.tsx`
- No external state management (Redux, Zustand, Jotai)
- No integration tests for the full pipeline (crawl → precalculate → API → render)
- No onboarding tour (issue #178 open)
- No marketing landing page (issue #193 open)
- No automated release pipeline for the frontend
- No WebSocket support for incremental updates (PR #247 still open)

### Health Metrics

| Metric | Value | Assessment |
|--------|-------|------------|
| Backend build | Pass | Healthy |
| Backend tests | 21/21 pass | Healthy |
| Frontend build | Pass (with size warning) | Needs attention |
| Frontend tests | 400/419 pass (95.5%) | Degraded |
| CI workflows | 5 workflows configured | Healthy |
| Documentation | ~60 files | Excessive (needs pruning) |
| Dependencies | Current (dependabot active) | Healthy |
| Bundle size | 1,761 KB / 499 KB gz | Needs optimization |
| Last commit (main) | Feb 12, 2026 | 5 days ago |
| Open issues | 14 open | Moderate backlog |
| Open PRs | 2 (both Copilot) | Low |
| Migration count | 28 (37 files) | Complex schema |
| Total commits | 695+ | Substantial history |

### Strengths

1. Solid backend. The Go backend is well-structured, the sqlc data layer is reliable, and all backend tests pass.

2. The rendering pipeline. The custom InstancedMesh renderer, octree spatial index, edge bundler, SDF text renderer, and Web Worker force simulation can handle 100k+ node graphs.

3. Tooling. The Makefile provides a single entry point for all operations. CI/CD covers testing, security, and Docker builds. Load testing is ready.

4. Feature breadth. Multiple visualization modes (3D, 2D, dashboard, communities), search, export, keyboard shortcuts, dark/light theme, mobile support, accessibility, shareable URLs.

### Weaknesses

1. Bus factor of 1. One active human contributor. All institutional knowledge is in one head.

2. Test suite degradation. 19 failing tests signal a maintenance gap. Performance tests with absolute thresholds are inherently flaky.

3. Documentation entropy. Dozens of implementation summary files describe intermediate states. The ratio of useful docs to noise is declining.

4. Dual rendering path. Both `react-force-graph-3d` and custom InstancedMesh exist. Nobody reading the code knows which one is supposed to be the real one.

5. No live demo. For a visualization project, the inability to see it running is a problem.

## Top 10 Next Engineering Tasks

The most impactful engineering tasks in priority order:

### 1. Fix the 19 failing frontend tests
Impact: High | Effort: Low-Medium

- Replace absolute performance thresholds with relative regression detection (compare against stored baseline)
- Update `Sidebar.test.tsx` to match current component ARIA structure
- Fix accessibility test assertions to match current component output
- Target: 419/419 tests passing

Files: `frontend/src/rendering/InstancedNodeRenderer.test.ts`, `frontend/src/spatial/Octree.test.ts`, `frontend/src/components/Sidebar.test.tsx`, `frontend/src/test/accessibility.test.tsx`

### 2. Remove dual rendering path
Impact: High | Effort: Medium

- Commit to the custom InstancedMesh renderer as the primary 3D path
- Remove or deprecate `react-force-graph-3d` dependency
- Update `Graph3D.tsx` to use `Graph3DInstanced.tsx` exclusively
- This removes ~500 KB from the bundle and eliminates ambiguity

Files: `frontend/src/components/Graph3D.tsx`, `frontend/src/components/Graph3DInstanced.tsx`, `frontend/package.json`

### 3. Implement code splitting
Impact: High | Effort: Medium

- Dynamic import for visualization modes: load 3D renderer only when 3D view is selected
- Dynamic import for Three.js (the largest dependency)
- Move D3 to a dynamic import for 2D view
- Target: initial load under 300 KB gzipped

Files: `frontend/src/App.tsx`, `frontend/vite.config.ts`

### 4. Add full-pipeline integration test
Impact: High | Effort: Medium

- Create a test that: seeds sample data → runs precalculation → queries `/api/graph` → validates response structure
- Can run in CI with the PostgreSQL service container
- This catches the class of bugs that broke the project in August 2025

Files: new test in `backend/internal/graph/integration_test.go` or `backend/e2e/`

### 5. Clean up documentation
Impact: Medium | Effort: Low

- Remove or consolidate implementation summary files (`IMPLEMENTATION_SUMMARY_*.md`, `CHANGES_SUMMARY.md`, `PR_SUMMARY.md`, `FIX_SUMMARY.md`)
- Ensure `docs/` files reflect current state, not historical intermediate states
- Update README to remove references to stale docs

Files: `docs/`, root-level `*.md` files

### 6. Complete NDJSON streaming
Impact: Medium | Effort: Medium

- Remove `[WIP]` tag from PR #224
- Add client-side NDJSON parsing in the frontend
- Add streaming error handling and fallback to batch loading
- Write integration test for streaming endpoint

Files: `backend/internal/api/handlers/graph.go`, `frontend/src/App.tsx`

### 7. Add React Router or proper view management
Impact: Medium | Effort: Medium

- Replace the manual `mode` state switching in `App.tsx` with React Router
- This enables proper URL routing, browser back/forward, and lazy loading per route
- Reduces `App.tsx` from 464 lines to a manageable size

Files: `frontend/src/App.tsx`, `frontend/src/main.tsx`, `frontend/package.json`

### 8. Extract state management
Impact: Medium | Effort: Medium

- Move application state out of `App.tsx` into a lightweight state manager (Zustand is a good fit for its simplicity)
- This eliminates the prop drilling and the god component anti-pattern
- Group state into domains: graph data, UI state, filter state, camera state

Files: `frontend/src/App.tsx`, new `frontend/src/store/` directory

### 9. Deploy a public demo
Impact: High | Effort: Medium-High

- Set up a VPS or cloud deployment running Docker Compose
- Seed with pre-crawled data (or crawl a small set of subreddits)
- Add to README: "Live Demo: [URL]"
- This matters more than any other task for a visualization project

Depends on: backend + frontend both working, seeded data, domain/hosting

### 10. Add contributing workflow
Impact: Medium | Effort: Low

- Ensure `CONTRIBUTING.md` has clear first-contribution instructions
- Add "good first issue" labels to accessible issues
- Set up issue templates for bug reports and feature requests
- This lowers the barrier for external contributors

Files: `CONTRIBUTING.md`, `.github/ISSUE_TEMPLATE/`

## Presentation improvements

If you want this to look good in a portfolio, demo, or pitch:

1. Live demo URL in README. Nothing sells a visualization project like seeing it.

2. 30-second GIF/video in README. Show the 3D graph rotating, zooming, and clustering. A visual project needs visual proof.

3. Prune root-level markdown files. The root currently has `BENCHMARK_IMPLEMENTATION.md`, `IMPLEMENTATION_SUMMARY.md`, `INSPECTOR_IMPLEMENTATION.md`, `LOAD_TESTING_IMPLEMENTATION.md`, and others. These are development artifacts, not project documentation. Move or remove them.

4. Version badge that links to a real release. The README has a version badge (v0.1.0) but no actual releases/tags. Create a proper GitHub Release.

5. Architecture diagram in README. The `docs/architecture.md` has Mermaid diagrams, but the README should include a simplified architecture image.

6. All tests passing in CI. A green CI badge is the minimum. Fix the 19 failing tests.

7. Clear "what this is not" section. The project is a data pipeline + visualization tool. It's not a social media analytics platform, not a Reddit replacement, not a production SaaS. Setting expectations up front prevents confusion.

## What Went Wrong / Friction

- Scope creep without completion. 28 database migrations, 30+ API endpoints, 5 visualization modes, 60 docs files. But 19 tests are broken and there's no live demo. Breadth over depth.

- Documentation as exhaust. Many docs were generated as side effects of PRs rather than written on purpose.

- No release process in practice. Despite PR #132 adding versioning and changelog infrastructure (Nov 1, 2025), there are zero git tags and zero GitHub Releases.

## What We Learned

- A project's health is not its feature count. Does it build? Do tests pass? Can someone new run it in 15 minutes? Is there a live demo? Clustr fails some of these basic checks.

- AI-assisted development produces breadth (many features, many tests, many docs) but can't prioritize. A human has to decide: fix the broken tests before adding new features.

- For a visualization project, a live demo beats everything. People need to see it.

- The interesting technical work is there: the graph precalculation pipeline, the custom WebGL renderer, the Louvain clustering. The path forward is to polish and present what exists, not to add more features.

## Receipts

- **Current branch:** `uplink/deploy-branch` (active deploy branch)
- **Main branch:** `main` (default, last commit Feb 12, 2026)
- **Open issues:** 14 (including epics #138–#144)
- **Open PRs:** 2 (#247 WebSocket, #248 SLO/SLI)
- **Backend health:** Builds, all tests pass
- **Frontend health:** Builds, 95.5% tests pass
- **Bundle size:** 1,761 KB (499 KB gzipped)
- **Migration count:** 28 (37 files including up/down pairs)
- **Total API endpoints:** 30+ (documented in `backend/internal/api/routes.go`)
- **Total frontend components:** ~20 (documented in `frontend/src/components/`)
- **Documentation files:** ~60 in `docs/`, ~10 root-level implementation notes
- **Last activity:** Feb 12, 2026 (PR #246 merge)
- **No git tags or GitHub Releases exist**
- **Repo:** `subculture-collective/reddit-cluster-map`
