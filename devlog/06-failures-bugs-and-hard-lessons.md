---
title: "Failures, Bugs, and Hard Lessons"
series: "Cutroom Devlog"
part: 6
date: "2026-02-17"
tags: ["devlog", "openwork", "cutroom", "bugs", "failures", "debugging", "postmortem"]
summary: "Recurring bug categories, fragile areas of the codebase, and what happened when we tried to build and test everything from scratch."
---

## Context

Part 6 of the Cutroom Devlog series. Recurring bugs, fragile areas, and a build/test attempt from scratch.

## What We Did

### Build/Run Attempt (Feb 17, 2026)

We followed the README instructions to build and test the project. Here are the results:

**Go backend build:**
```bash
$ cd backend && go build ./...
# (no output — success)
```
The Go backend compiles cleanly. All dependencies resolve, no build errors.

**Go backend tests:**
```bash
$ cd backend && go test ./...
# 27 packages tested
# 21 packages pass
# 6 packages have [no test files]
# 0 failures
```
All backend tests pass. The packages without test files are binary entrypoints (`cmd/*`), generated code (`internal/db`), and some infrastructure packages (`internal/admin`, `internal/authstore`).

**Frontend build:**
```bash
$ cd frontend && npx vite build
# ✓ 761 modules transformed
# ✓ built in 10.19s
# Warning: Some chunks are larger than 500 kB after minification
```
The frontend builds successfully. The main chunk is 1,761 KB (499 KB gzipped), exceeding Vite's recommended threshold. Three.js is most of that.

**Frontend tests:**
```bash
$ cd frontend && npx vitest run
# Test Files: 8 failed | 26 passed (34)
# Tests: 19 failed | 400 passed (419)
# Duration: 43.97s
```

Failing test files and root causes:

1. `InstancedNodeRenderer.test.ts` — performance timing assertions. "should complete position updates quickly" expected <50ms, got 50.23ms. "should update 100k node positions in less than 50ms" expected <350ms, got 435ms.

2. `Octree.test.ts` — performance timing assertions. "should handle 100k nodes build in reasonable time (<300ms)" got 340ms. "should query frustum efficiently for 100k nodes (<15ms)" got 15.02ms.

3. `Sidebar.test.tsx` — DOM query mismatch. `getByLabelText('Collapse sidebar')` fails because the rendered output doesn't have an element with that ARIA label. Almost certainly a component change that wasn't reflected in the test.

4. `accessibility.test.tsx` — ARIA attribute mismatches in rendered components.

5. Other test files — various assertion failures where component rendering expectations are out of sync with actual output.

The 19 failures split into two categories:
- Inherently flaky (performance tests with tight thresholds): 4 failures
- Stale test expectations (component changed, test not updated): 15 failures

### Recurring Bug Categories

The git history, PRs, and issue tracker show several recurring bug patterns:

#### 1. Memory Leaks in React Components

The most frequent bug category. Multiple PRs to address:

- **PR #68:** Fix tick handler memory leak in Graph2D — the D3 simulation's tick handler wasn't being cleaned up on component unmount
- **PR #76:** Fix idle check interval — `setInterval` running after component unmount
- **PR #84:** Clear simulation tick listeners on cleanup
- **PR #94:** Fix `setTimeout` memory leak in Communities component

Same root cause every time: D3.js and Three.js manage their own lifecycles, and those conflict with React's. Event listeners, timers, and animation frames registered during mount weren't being cleaned up during unmount.

**Related commits:** `ff0331b`, `8c16c12`, `a05ccc9`, `562107b`, `387a0ce`, `10a1255`

#### 2. Stale Closure / Reference Bugs

React components using D3 and Three.js suffered from stale closure variables:

- **PR #70:** Fix stale DOM element references in Graph2D tick callback — D3 selections captured during render became stale on re-render
- **PR #88:** Fix render path chain to use refs instead of closure-captured variables
- **PR #90:** Fix Graph2D closure issue with D3 selection refs

The pattern: D3 callbacks captured variables from the React render closure. When React re-rendered (new props, state changes), the D3 callbacks still referenced old values. The fix was consistently the same: use React refs instead of closure variables.

**Related commits:** `a39506b`, `49713ae`, `d19db65`, `20d0a43`

#### 3. Precalculation Pipeline Breakage

The graph precalculation pipeline broke at least three times:

- **Aug 2025:** Commits `065be3a` ("fixing pre calc") and `85c34f5` ("precalc db align") — schema and precalculation code drifted out of sync during the summer gap
- **Aug 2025:** Commit `24ae93c` ("optimization graph won't load") — the graph visualization failed to render, likely due to data format issues from the precalculation
- **Oct 2025:** PR #53 — position column errors required safety checks; PR #57 — position updates needed batching

Root cause: the precalculation pipeline writes to database tables that the API reads and the frontend renders. Any schema change or data format change must be coordinated across all three layers simultaneously.

#### 4. NDJSON Streaming: 8 Failed Attempts

NDJSON streaming for the graph API took 8 separate PR attempts:

| PR | Branch | Result |
|----|--------|--------|
| #223 | `copilot/implement-ndjson-streaming` | Closed |
| #225 | `copilot/implement-ndjson-streaming-again` | Closed |
| #226 | `copilot/add-ndjson-streaming-support` | Closed |
| #227 | `copilot/add-ndjson-streaming-support-again` | Closed |
| #228 | `copilot/implement-ndjson-streaming-another-one` | Closed |
| #229 | `copilot/implement-ndjson-streaming-yet-again` | Closed |
| #230 | `copilot/implement-ndjson-streaming-one-more-time` | Closed |
| #231 | `copilot/implement-ndjson-streaming-please-work` | Closed |
| **#224** | `copilot/add-ndjson-streaming-api` | **Merged** |

The branch names speak for themselves. NDJSON streaming requires coordinating Go's HTTP flushing behavior, JSON serialization, content-type headers, and frontend parsing. The Copilot agent couldn't get all the pieces working together.

The successful PR (#224) was merged on February 10, 2026, but carries a `[WIP]` tag in its title, suggesting it may not be fully complete.

#### 5. Dockerfile and Go Version Issues

- **PR #54:** Fix Go toolchain mismatch in Dockerfiles — the Go version specified didn't match the available toolchain
- **Commit `3c51d9a`** (Nov 12): "fix: update Go version from invalid 1.24.9 to valid 1.23"

The Go ecosystem's version management (go.mod `go` directive vs. `toolchain` directive) caused ongoing confusion about which Go version was actually needed.

#### 6. React Component Type Errors

- **PR #55:** Fix CommunityMap build errors — TypeScript type mismatches
- **PR #72:** Fix React key usage in VirtualList — using array indices as keys
- **Commit `bd101f9`** (Oct 17): "types" — generic type fixing
- **PR #105:** Label positioning fixes — Set/Map pattern replacing array-based logic

### Fragile Areas of the Codebase

Ranked by frequency of bug fixes and PRs touching each area:

1. `frontend/src/components/Graph2D.tsx` — touched by PRs #62, #68, #70, #80, #82, #84, #86, #88, #90, #92. Ten separate bug fix PRs for one component. D3 + React is inherently fragile.

2. `frontend/src/components/CommunityMap.tsx` — PRs #55, #103, #105. Build errors, positioning bugs, and type issues.

3. `backend/internal/graph/service.go` — the precalculation service broke during schema changes and required careful coordination with the database and API layers.

4. `frontend/src/components/Graph3D.tsx` / `Graph3DInstanced.tsx` — two implementations of the same thing (react-force-graph + custom InstancedMesh) coexisting in the codebase.

### Missing Error Handling

Several areas where errors fail silently:

- The crawler's OAuth token refresh: if the refresh fails, the crawler keeps going with an expired token until it gets a 401, then retries
- The precalculation pipeline: a mid-batch failure can leave the graph in an inconsistent state
- Frontend data fetching: the original implementation had no error boundaries (added later in PR #197)

## Key Decisions

1. Fixing symptoms vs. root causes. The memory leak and stale closure bugs were fixed individually (10+ PRs) rather than by redesigning the D3/React integration. Each fix was small and safe, but the same class of bug can recur in any new component that uses D3.

2. Keeping performance tests. Despite the flaky timing assertions, the performance tests stay in the codebase. They document performance targets even when they fail as tests.

3. Shipping with broken tests. The current state has 19 failing tests. Either the CI pipeline isn't enforcing test passage, or the failures are known and accepted.

## What Went Wrong / Friction

- Graph2D is the single most fragile component. D3 force simulation, React lifecycle, SVG rendering, and event handling all in one file. Bugs are easy to introduce and hard to isolate.

- AI-generated tests don't survive code changes. The Copilot agent wrote tests for components at a point in time. When later PRs (also from Copilot) modified those components, the tests weren't always updated. That's how you get 15 stale-expectation failures.

- No integration tests for the full pipeline. There are unit tests for individual packages and E2E visual tests for the frontend, but nothing that exercises the full flow: insert crawl data → run precalculation → fetch graph API → verify response structure.

## What We Learned

- D3.js and React have different rendering models (imperative vs. declarative). Integrating them requires careful lifecycle management. Refs are essential; closure-captured variables are traps. This came up over and over.

- Cross-cutting features like NDJSON streaming are where AI assistance breaks down. They need coordinated changes across multiple layers at once. Sequential, additive changes are where AI works well.

- Performance tests with absolute thresholds are CI poison. Store a baseline, compare against it, alert on regression. Anything else breaks across environments.

- The precalculation pipeline is a coordination bottleneck. Any schema change must ripple through: SQL queries → sqlc generation → precalculation logic → API handlers → frontend data parsing.

## Receipts

- **Build attempt commands:**
  - `cd backend && go build ./...` — success
  - `cd backend && go test ./...` — 27 packages, all pass
  - `cd frontend && npx vite build` — success, 10.19s, 1,761 KB main bundle
  - `cd frontend && npx vitest run` — 8/34 files fail, 19/419 tests fail
- **Memory leak PRs:** #68, #76, #84, #94
- **Stale closure PRs:** #70, #88, #90
- **NDJSON attempts:** PRs #223–#231 (closed), PR #224 (merged with [WIP] tag)
- **Precalculation fixes:** commits `065be3a`, `85c34f5`, `24ae93c` (Aug 2025)
- **Go version fix:** commit `3c51d9a` (Nov 12, 2025)
- **Graph2D bug fixes:** PRs #62, #68, #70, #80, #82, #84, #86, #88, #90, #92
- **CommunityMap fixes:** PRs #55, #103, #105
- **Failing test files:** `InstancedNodeRenderer.test.ts`, `Octree.test.ts`, `Sidebar.test.tsx`, `accessibility.test.tsx`
- **Performance thresholds:** Octree 100k build <300ms (got 340ms), frustum query <15ms (got 15.02ms)
