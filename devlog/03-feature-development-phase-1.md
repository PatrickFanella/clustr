---
title: "Feature Development Timeline (Phase 1)"
series: "Cutroom Devlog"
part: 3
date: "2026-02-17"
tags: ["devlog", "openwork", "cutroom", "features", "mvp", "graph", "crawler"]
summary: "From init to a working 3D graph in 10 days: the graph API, precalculation pipeline, and the 'it works' moments."
---

## Context

Part 3 of the Cutroom Devlog series. May 31 to June 9, 2025: empty repo to deployed app with a working crawler, graph pipeline, and 3D visualization. One developer. (`0nnwee` / `⓪ηηωεε忧世`)

## What We Did

### Day 1–2: Backend Foundation (May 31 – Jun 1)

The first two days produced the entire backend core:

- **Commit `88cb0b9`** (May 31): Skeleton — Go server, empty Dockerfile, first DB schema
- **Commit `e8ce97f`** (May 31): Complete crawler system — OAuth auth, Reddit API client, job queue, rate limiter, worker. 43 files, 2,463 insertions.
- **Commit `7ecba63`** (May 31): Refactored utilities into proper packages
- **Commit `5adfaef`** (May 31): User subreddit discovery
- **Commit `9c8b9a5`** (Jun 1): "make logging sane again" — cleaned up log output

By end of day 2, the backend could:
1. Accept crawl job requests via POST `/api/crawl`
2. Authenticate with Reddit's OAuth API
3. Fetch subreddit info, posts, and comments
4. Extract user participation data
5. Store everything in PostgreSQL
6. Rate-limit outbound requests to ~1.66 rps

### Day 3–5: Deploy Battle (Jun 1 – Jun 3)

Covered in Part 1. Key outcome: Docker Compose deployment working after 6 attempts across 3 days. PR #1 merged.

### Day 4: Frontend Scaffolding (Jun 2)

Commit `08c53e5` added the React frontend (PR #2):
- Vite + React + TypeScript + Tailwind CSS
- GitHub Actions deploy workflow
- No visualization yet — just the app shell

### Day 8: The Graph Milestone (Jun 7)

June 7 had four features ship in one day:

**1. Graph API Endpoint** (`96df4e3`)
The `/api/graph` endpoint was added, returning nodes and links as JSON. This single commit also introduced `react-force-graph-3d` to the frontend and created the first `Graph3D.tsx` component.

Key files added:
- `backend/internal/api/handlers/graph.go` — SQL queries returning graph data
- `backend/internal/db/graph.sql.go` — sqlc-generated graph queries
- `backend/internal/queries/graph.sql` — Raw SQL for node/link retrieval
- `frontend/src/components/Graph3D.tsx` — 56 lines, the first visualization

This was merged as PR #3. The `package-lock.json` diff alone was 9,452 lines due to new Three.js dependencies.

**2. Graph ID Type Fix** (`e8563fe`)
Immediately after PR #3 merged, a fix was needed — the graph node ID type was wrong. This small commit (3 insertions, 2 deletions) fixed the `graph.go` handler.

**3. Graph Precalculation** (`c4e3edc`, `923cdb1`)
Two commits introduced the precalculation pipeline:

- `backend/cmd/precalculate/main.go` — A standalone binary to compute the graph
- `backend/internal/graph/service.go` — Graph service with batch node/link generation
- `backend/internal/graph/job.go` — Background job scheduler
- Database migrations for `graph_nodes` and `graph_links` tables
- Dropped the old `subreddit_edges` table in favor of the new graph model

Rather than computing graph data on each request, the precalculation service writes results to dedicated tables. The API reads from those tables with caching.

Commit `923cdb1` was substantial: 327 insertions, 327 deletions across 18 files. It restructured the entire graph data model.

**4. Graph3D Component Polish** (`8d5fc24`)
The Graph3D component was improved (55 insertions, 48 deletions) — likely adding interactive features, styling, and node rendering improvements.

**Total for Jun 7:** 8 commits, 4 merged PRs (#3, #6, #7, #8). The project went from backend-only to full-stack with visualization in a single day.

### Day 9: Crawl Job Improvements (Jun 8)

**Stale subreddit requeuing** (`e1b774e`, PR #12):
Added logic to re-enqueue subreddits that haven't been crawled recently. The `STALE_DAYS` config parameter (default 30) determines when a subreddit's data is considered stale.

**Major restructuring** (`4de386e`):
A 718-insertion commit that:
- Created a dedicated crawler binary (`backend/cmd/crawler/main.go`) — separating it from the API server
- Added a frontend Dockerfile
- Restructured Docker Compose to run crawler as its own service
- Added 6 more database migrations (000001–000007)
- Expanded graph data queries with content graph support

This was the first time the backend was split into separate deployable services.

### Day 10: Precalculation Tuning (Jun 8–9)

Three more commits fine-tuned the precalculation pipeline:
- `7be5f90` (Jun 8) — "precalculate"
- `bc0d77d` (Jun 8) — "precalculation"
- `89cf951` (Jun 9) — "precalc adjust"

Then silence. The next commit wouldn't come for over two months.

### The First Working Version

By June 9, 2025, the project had:

| Feature | Status |
|---------|--------|
| Reddit OAuth crawler | Working |
| PostgreSQL storage | Working |
| Graph precalculation | Working |
| REST API with graph endpoint | Working |
| 3D visualization (react-force-graph-3d) | Working |
| Docker Compose deployment | Working |
| CI/CD | Basic deploy workflow only |
| Tests | None |
| Error handling | Minimal |
| Documentation | README only |

## Key Decisions

1. Separate crawler from API server. Commit `4de386e` (Jun 8) split the crawler into its own binary and Docker service. The crawler is a long-running worker with different resource needs and failure modes than the API server. They belong in separate processes.

2. Precalculation over real-time computation. The graph could have been computed on each API request. Instead, the precalculation pipeline runs periodically and writes results to dedicated tables. Freshness traded for performance.

3. react-force-graph-3d for initial visualization. This library wraps Three.js and gives you a force-directed 3D graph with minimal code. It got the project to "visible results" fast, but would later become a performance bottleneck for large graphs.

4. Content graph as optional. The `DETAILED_GRAPH` config flag (default `false`) controls whether posts and comments appear in the graph. By default, only users and subreddits are rendered, keeping things manageable.

## What Went Wrong / Friction

- The two-month gap. After June 9, the project went silent until August 15. That's the longest inactivity period in the project's history. No issues, PRs, or commits explain why.

- No tests during Phase 1. 15+ PRs merged without a single test file. The crawler hits a live API, the precalculation pipeline writes to production tables, and the API serves uncached JSON. Any of these could fail silently.

- Multiple deploy branches. PRs #6, #7, #9, #10, #11, #13 show a pattern of merging `main` into `deploy` and back again. Code was being synchronized between branches rather than flowing in one direction.

- Commit message quality. Several commits have single-word messages: "requeue", "precalculate", "precalculation". Combined with large diffs, this makes it hard to understand what changed or why.

- Database migration numbering. Migrations jump from `000003` to `000005` in the initial setup, and some were renumbered within the same commit. The migration history is confusing from the start.

## What We Learned

- Getting to "visible results" matters more than getting the architecture right. The June 7 graph milestone — seeing Reddit communities rendered as a 3D network — was the first proof of concept. Everything before that was invisible plumbing.

- Splitting services early (crawler vs. API) paid off. A long-running rate-limited worker has nothing in common with a request-response server.

- The gap between "it works locally" and "it works deployed" was three days of debugging for a project that was functionally complete in two.

- Phase 1 established the core pipeline that lasted the entire project: Go backend → PostgreSQL → precalculated graph → React frontend. No future refactor touched this shape.

## Receipts

- **Phase 1 span:** May 31 – June 9, 2025 (10 active days)
- **Total commits (Phase 1):** ~30 commits across main and feature branches
- **PRs merged:** #1 (deploy), #2 (frontend), #3 (graph API + visualization), #6–#16 (various)
- **Key milestone commit:** `96df4e3` (Jun 7) — "added graph api endpoint and graph component"
- **Graph3D component:** `frontend/src/components/Graph3D.tsx` (56 lines initially)
- **Precalculation:** `backend/cmd/precalculate/main.go` (61 lines), `backend/internal/graph/service.go` (93 lines)
- **Crawler separation:** `backend/cmd/crawler/main.go` added in `4de386e` (Jun 8)
- **Graph API handler:** `backend/internal/api/handlers/graph.go`
- **Graph SQL queries:** `backend/internal/queries/graph.sql`
- **Stale requeuing:** `e1b774e` (Jun 8) — crawler worker, 69 new lines
- **Post-Phase 1 gap:** June 9 → August 15, 2025 (67 days of inactivity)
- **Contributor:** Single developer (`0nnwee` / `⓪ηηωεε忧世`)
