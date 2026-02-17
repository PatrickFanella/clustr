---
title: "Feature Development Timeline (Phase 2) + Product Direction"
series: "Cutroom Devlog"
part: 4
date: "2026-02-17"
tags: ["devlog", "openwork", "cutroom", "features", "copilot", "phase2", "rendering", "performance"]
summary: "The post-gap revamp, the Copilot sprint, the org transfer, and the 2.0 rendering engine rewrite. August 2025 through February 2026."
---

## Context

Part 4 of the Cutroom Devlog series. After a 67-day gap following Phase 1, development resumed in August 2025 with intermittent bursts: summer revamp (Aug 2025), Copilot-driven sprint (Oct-Nov 2025), and the 2.0 rendering engine push (Feb 2026).

## What We Did

### Sub-Phase A: The Summer Revamp (Aug 15–26, 2025)

After 67 days of silence, development resumed with five commits over 11 days:

| Commit | Date | Message |
|--------|------|---------|
| `77e1ddc` | Aug 15 | "revamped" |
| `5246340` | Aug 15 | "authcodes" |
| `166922f` | Aug 19 | "she runnin" |
| `f3c286b` | Aug 20 | "alter brewing" |
| `065be3a` | Aug 20 | "fixing pre calc" |
| `85c34f5` | Aug 22 | "precalc db align" |
| `24ae93c` | Aug 26 | "optimization graph won't load" |

The commit messages are informal but tell a story: the project was being revived, the OAuth authentication was being fixed ("authcodes"), the precalculation pipeline had broken during the gap ("fixing pre calc", "precalc db align"), and performance problems were emerging ("optimization graph won't load").

Then another 49-day gap: August 26 → October 14.

### Sub-Phase B: The Copilot Sprint (Oct 14 – Nov 1, 2025)

This is where the project's development model changed. Three things happened at once:

**1. The org transfer.** The repository moved from `onnwee/reddit-cluster-map` to `subculture-collective/reddit-cluster-map`. This is visible in PR URLs — PRs before #57 reference `onnwee/`, while #57+ reference `subculture-collective/`.

**2. A new collaborator appeared.** Patrick Fanella (`PatrickFanella`) began merging PRs and applying Copilot suggestions starting around October 18. His contributions were primarily code review and merge operations: applying suggestions, merging branches, updating configs. He contributed 46 commits total.

**3. GitHub Copilot SWE agent became the primary code generator.** Starting October 17, `copilot-swe-agent[bot]` began generating PRs at scale. In the commit log, this bot contributed 413 commits, more than all human contributors combined (226 human commits). The Copilot agent's contributions included:

**Bug fixes (Oct 17–19):**
- PR #53: Fix graph node position errors (safety checks, `QueryContext`)
- PR #55: Fix CommunityMap build errors
- PR #54: Fix Dockerfile Go toolchain mismatch
- PR #70: Fix stale DOM element references in Graph2D
- PR #68: Fix tick handler memory leak
- PR #72: Fix React key usage in VirtualList
- PR #76: Fix idle check interval (unmount issue)
- PR #84: Clear simulation tick listeners on cleanup
- PR #94: Fix setTimeout memory leak in Communities component

**Performance (Oct 17–20):**
- PR #62: Client-side LOD rendering and frame throttling
- PR #57: Reliable position updates with batching
- PR #58: Graph API query timeout handling
- PR #98: Graph node indexing optimization
- PR #99: API cache key fixes with metrics

**Security (Oct 18):**
- PR #61: Rate limiting, CORS, security headers, input validation

**DX improvements (Oct 18–19):**
- PR #60: Enhanced Makefile, dev scripts, pre-commit hooks
- PR #66: `.env.example` files for backend and frontend
- PR #78: Sane defaults for database config

**Community detection (Oct 24–25):**
- PR #103: CommunityMap v1 polish (tooltips, auto-fit, zoom)
- PR #105: Label positioning fixes
- PR #106: Community aggregation API endpoints + Louvain integration
- PR #112: Degree sum caching for O(n²) elimination
- PR #114: Redundant query elimination in community detection

**Observability (Oct 26):**
- PR #115: Prometheus metrics pipeline + Grafana dashboards
- PR #118: Structured logging, distributed tracing (OpenTelemetry), error reporting (Sentry)

On October 17, 22 issues (#35-#51) were batch-created covering the full roadmap: security hardening, observability, analytics, docs, testing, DX, performance, frontend UX, API endpoints, job management, and crawler improvements. These issues drove the Copilot agent's work.

### Sub-Phase C: Hardening & Docs (Oct 27 – Nov 13, 2025)

The sprint continued:

| PR | Date | Feature |
|----|------|---------|
| #119 | Oct 28 | Security hardening: dependency scanning, admin auth tests |
| #120 | Oct 28 | Persistent filters, legend, shareable URLs with camera state |
| #121 | Oct 28 | OAuth token refresh with proactive renewal |
| #122 | Oct 29 | Web control panel for crawl jobs |
| #123 | Oct 29 | Data integrity checks and cleanup operations |
| #124 | Oct 29 | Test infrastructure: unit, integration, e2e |
| #125 | Oct 29 | Job prioritization, visibility timeout, scheduled crawls |
| #126 | Oct 30 | Search and export API endpoints |
| #127 | Oct 30 | Comprehensive documentation: architecture, runbooks, contribution guide |
| #131 | Nov 1 | Performance profiling infrastructure |
| #132 | Nov 1 | Release process: versioning, tags, changelog |
| #133 | Nov 1 | Security audit and penetration testing |

Then November 12–13 saw deployment-focused work:
- Commit `cd1b238`: "moved docs and delete package-lock"
- Commits for CI updates, Go version fix, Docker container naming
- The root Makefile was created (commit `4eadd22`)
- Restart policies and health checks added

After November 13: another 86-day gap until February 7, 2026.

### Sub-Phase D: The 2.0 Push (Feb 7–12, 2026)

On February 7, commit `c1898ec` landed with the message **"moving on from mvp"**. The idea was to move past the MVP.

On the same day, 58 issues (#138–#195) were batch-created, organized into six epics:

- **E1:** Large-Scale Rendering Engine (issues #145–#155)
- **E2:** Backend Scalability & Streaming (issues #158–#165)
- **E3:** Graph Data Pipeline & Precalculation (issues #167–#173)
- **E4:** Frontend UX & Interaction Polish (issues #174–#183)
- **E5:** Testing, CI/CD & Quality (issues #184–#189)
- **E6:** Operational Maturity & Documentation (issues #190–#195)

Over the next 5 days, Copilot SWE agent executed against these epics:

**Rendering engine (Feb 7–8):**
- PR #196: Edge bundling for dense link clusters
- PR #203: Replace react-force-graph-3d with InstancedMesh renderer (100k+ nodes)
- PR #204/#205: GPU-accelerated link rendering with LineSegments
- PR #210: Web Worker-based force simulation
- PR #208: Node interaction (hover/click/select) for instanced rendering
- PR #209: Physics simulation stabilization
- PR #213: Barnes-Hut optimization (O(n²) → O(n log n))
- PR #214: Octree spatial index for O(log n) raycasting
- PR #215: Camera-distance-based node size scaling
- PR #216: Performance HUD with <1% overhead

**Backend scalability (Feb 8–9):**
- PR #199: Structured API error codes
- PR #200: Brotli/gzip compression
- PR #206: Query optimization (EXISTS subqueries, covering indexes, 5s timeout)
- PR #217: Size-bounded LRU cache (replacing unbounded)
- PR #218: GiST spatial index for viewport queries
- PR #219: Precomputed edge bundle metadata
- PR #221: ETag + stale-while-revalidate caching
- PR #222: Incremental graph precalculation
- PR #220: Tiered graph API (overview + spatial drill-down)
- PR #232: Cursor-based pagination
- PR #224: NDJSON streaming (after 7 failed attempts — PRs #223, #225–#231)

**Frontend UX (Feb 9–12):**
- PR #233: SDF text rendering (replacing SpriteText)
- PR #234: Adaptive LOD with FPS-based tier management
- PR #235: Graph versioning with incremental diff API
- PR #236: Dark/light theme with system preference
- PR #237: Instant search with autocomplete
- PR #238: Collapsible sidebar redesign
- PR #239: Graph minimap for navigation
- PR #240: Keyboard navigation and shortcuts
- PR #241: Rich node inspector panel
- PR #242: Mobile and touch support
- PR #245: WCAG 2.1 AA accessibility compliance

**Quality (Feb 8–12):**
- PR #201: Bundle size tracking and CI gate
- PR #202: Frontend test suite (78% coverage)
- PR #243: Performance benchmark suite
- PR #244: k6 load testing infrastructure
- PR #246: Visual regression testing

## Key Decisions

1. Leaning heavily on AI-assisted development. The Copilot SWE agent produced the majority of the codebase. This was deliberate: batch-create issues, let the agent execute. The tradeoff is real though. Velocity up, human understanding of the code down.

2. The InstancedMesh rewrite (PR #203). Replacing `react-force-graph-3d` with custom Three.js rendering using `InstancedMesh`. Moved the performance ceiling from ~10k nodes to 100k+.

3. Org transfer to subculture-collective. The project moved from a personal account to an organization, suggesting intent to grow the team or present the project more professionally.

4. Epic-driven roadmap. The February 2026 push organized 58 issues into 6 epics with clear scope boundaries. Having that structure made the week productive in a way the earlier sprints weren't.

## What Went Wrong / Friction

- Three extended gaps. Jun 9 → Aug 15 (67 days), Aug 26 → Oct 14 (49 days), Nov 13 → Feb 7 (86 days). That's 202 days of inactivity, more than half the project's calendar age. Whatever momentum was built in each sprint was mostly gone by the next.

- NDJSON streaming: 8 attempts. PRs #223 through #231 were all attempts to implement NDJSON streaming. Seven failed before PR #224 succeeded. The branch names tell the story: `implement-ndjson-streaming`, `implement-ndjson-streaming-again`, `implement-ndjson-streaming-yet-again`, `implement-ndjson-streaming-another-one`, `implement-ndjson-streaming-one-more-time`, `implement-ndjson-streaming-please-work`.

- Copilot-generated documentation volume. The agent produced implementation summary documents for everything: `BENCHMARK_IMPLEMENTATION.md`, `INSPECTOR_IMPLEMENTATION.md`, `LOAD_TESTING_IMPLEMENTATION.md`, `IMPLEMENTATION_SUMMARY.md`, plus per-PR docs in `docs/`. Many of these describe intermediate states rather than final implementations.

- The "precalc db align" problem (Aug 2025). The precalculation pipeline broke during the summer gap. Commits `065be3a` ("fixing pre calc") and `85c34f5` ("precalc db align") show schema and precalculation code drifting out of sync. Then `24ae93c` ("optimization graph won't load") shows the graph rendering was broken too.

## What We Learned

- AI-assisted development at this scale works for additive features (new endpoints, new components, new tests) but breaks down on cross-cutting concerns. NDJSON streaming needed coordinated changes across backend response format, frontend parsing, and error handling. The agent couldn't hold all that in its head at once.

- Sprint-based development with long gaps is fragile. Each return to the project meant re-learning the codebase and fixing breakage that accumulated during the gap.

- The project's direction got clearer over time: it started as a data pipeline (crawl → store → serve) and moved toward a visualization platform where the rendering performance became the interesting problem.

- The 2.0 push (Feb 2026) got more done in 5 days than any previous sprint. The difference was structured planning (epics + issues) combined with AI execution. Human direction plus AI implementation velocity worked better than either alone.

## Receipts

- **Summer revamp:** commits `77e1ddc` through `24ae93c` (Aug 15–26, 2025)
- **Copilot sprint start:** commit `af3953c` (Oct 17, 2025, first copilot-swe-agent commit)
- **Org transfer evidence:** PR #57+ reference `subculture-collective` (Oct 18, 2025)
- **Patrick Fanella first commit:** `93320d0` (Oct 18, 2025)
- **Issues batch-created:** #35–#51 (Oct 17, 2025), #138–#195 (Feb 7, 2026)
- **"moving on from mvp":** commit `c1898ec` (Feb 7, 2026)
- **InstancedMesh renderer:** PR #203 (Feb 8, 2026)
- **NDJSON streaming attempts:** PRs #223–#231 (all closed), PR #224 (merged Feb 10)
- **Total Copilot commits:** 413 (copilot-swe-agent[bot]) + 53 (Copilot)
- **Total human commits:** ~226 (⓪ηηωεε忧世 170, Patrick Fanella 46, 0nnwee 7, onnwee 3)
- **Contributor stats:** `git shortlog -sn --all`
- **Epic structure:** Issues #139 (E1), #140 (E2), #141 (E3), #142 (E4), #143 (E5), #144 (E6)
- **Inactivity gaps:** Jun 9→Aug 15 (67d), Aug 26→Oct 14 (49d), Nov 13→Feb 7 (86d)
