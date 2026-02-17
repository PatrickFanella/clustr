---
title: "Architecture & Technical Spine"
series: "Cutroom Devlog"
part: 2
date: "2026-02-17"
tags: ["devlog", "openwork", "cutroom", "architecture", "go", "react", "postgresql", "threejs"]
summary: "How the backend, frontend, database, and infrastructure fit together, and the tradeoffs behind the architectural choices."
---

## Context

Part 2 of the Cutroom Devlog series. Repository structure, core modules, data flow, and why things are the way they are.

## What We Did

### Repository Structure

The repo follows a monorepo pattern with two primary workspaces:

```
reddit-cluster-map/
├── backend/              # Go services (API, crawler, precalculate)
│   ├── cmd/              # Binary entrypoints (server, crawler, precalculate, integrity)
│   ├── internal/         # All business logic packages (~20 packages)
│   ├── migrations/       # 37 SQL migration files (000001–000028)
│   ├── loadtest/         # k6 load testing scripts
│   └── docker-compose.yml
├── frontend/             # React SPA (Vite + TypeScript)
│   ├── src/components/   # UI components (~20)
│   ├── src/rendering/    # Custom WebGL renderers
│   ├── src/spatial/      # Octree spatial index
│   ├── src/hooks/        # React hooks
│   ├── src/workers/      # Web Workers
│   ├── e2e/              # Playwright visual tests
│   └── benchmarks/       # Performance benchmark suite
├── monitoring/           # Prometheus + Grafana configs
├── scripts/              # Deploy, hooks, testing utilities
├── docs/                 # ~60 documentation files
├── .github/workflows/    # CI, security scanning, benchmarks, release
└── Makefile              # 468-line unified build system
```

### Backend Architecture

The Go backend is organized into three separate binaries, all sharing the same internal packages:

**1. API Server** (`backend/cmd/server/main.go`)
- HTTP server on port 8000 using Gorilla Mux
- Serves the graph data, search, export, community, and admin endpoints
- Runs a background scheduler for hourly graph precalculation
- Middleware stack: rate limiting, CORS, security headers, compression, request tracing

**2. Crawler** (`backend/cmd/crawler/main.go`)
- Long-running worker that processes crawl jobs from the `crawl_jobs` table
- OAuth-authenticated Reddit API client with global rate limiting (601ms ticker, ~1.66 rps)
- Retry logic with exponential backoff and Retry-After header support
- Circuit breaker pattern for API resilience

**3. Precalculate** (`backend/cmd/precalculate/main.go`)
- One-shot job that transforms relational data into graph nodes and links
- Runs hourly via Docker Compose restart policy
- Generates: subreddit overlap edges, user-subreddit activity, post/comment reply chains
- Runs Louvain community detection and edge bundle precomputation

**Internal packages** (20+):

| Package | Purpose |
|---------|---------|
| `api` | Route registration, router factory |
| `api/handlers` | HTTP handler functions for all endpoints |
| `apierr` | Structured error codes with request tracing |
| `cache` | Ristretto-based LRU cache (size-bounded, TTL) |
| `circuitbreaker` | Circuit breaker for external API calls |
| `config` | Environment variable loading |
| `crawler` | Reddit API client, job processing, user discovery |
| `db` | sqlc-generated database access layer |
| `graph` | Graph precalculation service, Louvain clustering, versioning |
| `httpx` | HTTP client with retries, compression support |
| `integrity` | Data consistency checks and cleanup |
| `logger` | Structured logging (slog) |
| `metrics` | Prometheus metric collectors |
| `middleware` | Rate limiting, CORS, security headers, compression |
| `redditapi` | Reddit API type definitions |
| `scheduler` | Cron-like job scheduling |
| `secrets` | Credential masking in logs |
| `server` | Server lifecycle, DB initialization |
| `tracing` | OpenTelemetry distributed tracing |

### Database Schema

PostgreSQL 17 with 15+ tables. The schema evolved through 28 migrations:

**Core data tables:**
- `subreddits` — name, title, subscribers, timestamps
- `users` — username, timestamps
- `posts` — reddit_id (TEXT PK), subreddit FK, author FK, score, flair
- `comments` — reddit_id (TEXT PK), post FK, author FK, body, score, depth

**Graph tables:**
- `graph_nodes` — id (TEXT PK with prefixes: `subreddit_*`, `user_*`, `post_*`, `comment_*`), name, val, type, pos_x/y/z
- `graph_links` — source/target FKs with UNIQUE constraint
- `graph_communities` — Louvain-detected communities
- `graph_community_members` — node-to-community membership
- `graph_community_links` — inter-community weighted edges
- `graph_bundles` — precomputed edge bundles with Bezier control points
- `graph_versions` — version tracking for incremental updates
- `graph_diffs` — per-version entity changes (add/remove/update)

**Infrastructure tables:**
- `crawl_jobs` — job queue with priority, retries, visibility timeout
- `scheduled_jobs` — cron-based recurring crawls
- `admin_audit_log` — JSONB audit trail
- `service_settings` — key-value runtime configuration
- `precalc_state` — singleton tracking last precalculation

### Data Flow

```
Reddit API
    ↓ (OAuth + rate-limited HTTP)
Crawler Worker
    ↓ (INSERT/UPSERT)
PostgreSQL (subreddits, users, posts, comments)
    ↓ (batch reads)
Precalculate Service
    ↓ (batch writes)
PostgreSQL (graph_nodes, graph_links, graph_communities)
    ↓ (cached reads)
API Server (/api/graph, /api/communities)
    ↓ (JSON over HTTP)
React Frontend (Three.js / D3.js rendering)
```

### Frontend Architecture

**Stack:** React 19 + Vite 6.4 + TypeScript 5.8 + Tailwind CSS 3.4

**Visualization modes** (all state-driven, no React Router):
1. **3D Graph** — Three.js with custom `InstancedMesh` renderer for 100k+ nodes
2. **2D Graph** — D3.js force simulation with SVG rendering
3. **Dashboard** — Statistical overview
4. **Communities** — Louvain community detection view

**Custom rendering pipeline** (`frontend/src/rendering/`):
- `InstancedNodeRenderer.ts` — GPU-instanced sphere rendering
- `LinkRenderer.ts` — Batched `LineSegments` for GPU-accelerated links
- `EdgeBundler.ts` — Bezier curve edge bundling for dense clusters
- `SDFTextRenderer.ts` — Signed distance field text via `troika-three-text`
- `ForceSimulation.ts` — Web Worker-based force-directed layout

**Spatial indexing** (`frontend/src/spatial/`):
- `Octree.ts` — O(log n) raycasting and frustum culling for 100k+ nodes

State management is props-based with URL serialization for shareable links. No Redux, no external state library. The `App.tsx` component (464 lines) manages all top-level state: view mode, filters, physics parameters, selected node, camera position, LOD tier, and community colors. It's a god component, and we'll come back to that.

### Infrastructure

**Docker Compose** defines 8 services:
- `api` — Go API server with health checks
- `crawler` — Background crawler worker
- `precalculate` — Hourly graph computation (restart policy)
- `backup` — 24-hour backup interval
- `db` — PostgreSQL 17 with WAL mode
- `reddit_frontend` — Nginx serving SPA + reverse proxy
- `prometheus` — Metrics collection
- `grafana` — Dashboards and alerting

**Nginx** (`frontend/nginx.conf`) serves the SPA with HTML5 pushState fallback and proxies all `/api/*` requests to the Go backend using Docker DNS resolution.

## Key Decisions

1. sqlc over ORM. Hand-written SQL with type-safe Go codegen from the very first commit. Tight coupling between SQL files and Go code, but no ORM magic, and every database interaction is traceable to a `.sql` file in `backend/internal/queries/`.
   - Evidence: `backend/sqlc.yaml` present since commit `88cb0b9`

2. Graph as materialized view. Rather than computing the graph on each API request, a precalculation pipeline writes graph data to dedicated tables. This separates read performance from computation cost.
   - Evidence: `backend/cmd/precalculate/main.go` added in commit `c4e3edc` (Jun 7)

3. PostgreSQL as job queue. Instead of introducing Redis or RabbitMQ, the crawl job queue is a database table with status columns (`queued`, `crawling`, `success`, `failed`), visibility timeouts, and priority ordering.
   - Evidence: `backend/internal/queries/crawl_jobs.sql`, migration `000012` (priority), migration `000021` (visibility timeout)

4. Custom WebGL renderer over react-force-graph. The project started with `react-force-graph-3d` but later replaced it with a custom `InstancedMesh` renderer to handle 100k+ nodes.
   - Evidence: PR #203 (Feb 8, 2026), file `frontend/src/components/Graph3DInstanced.tsx`

5. No React Router. View switching (3D, 2D, dashboard, communities, admin) is managed via state in `App.tsx`. URL state serialization handles shareable links via `frontend/src/utils/urlState.ts`.
   - Evidence: `frontend/src/App.tsx` uses conditional rendering based on `mode` state

6. Monorepo with unified Makefile. A 468-line root Makefile with targets for Docker operations, database migrations, backend/frontend dev, code generation, load testing, monitoring, and backups.
   - Evidence: `Makefile` (refactored to root in commit `4eadd22`, Nov 12)

## What Went Wrong / Friction

- The App.tsx god component. At 464 lines, `App.tsx` manages all application state: view mode, filters, physics, selection, camera, LOD, community colors, and keyboard shortcuts. Any new feature touching state has to modify this file.

- No external state management. Avoiding Redux/Zustand/Jotai kept dependencies minimal but resulted in deep prop drilling. Components like `Graph3D` receive 15+ props.

- Schema migration complexity. 37 migration files with numbering gaps (no `000015`) and overlapping concerns. The `graph_nodes` table alone was modified by migrations `000002`, `000003`, `000016`, `000017`, and `000018`. Understanding the current schema means reading across multiple migration files.

- Dual rendering paths. The frontend has both `react-force-graph-3d` and a custom `InstancedMesh` renderer. Both exist in the codebase simultaneously. Nobody reading it would know which one is supposed to win.

## What We Learned

- sqlc's SQL-first approach works well here. The generated code stays in sync with the schema, and raw SQL is easy to optimize (indexes, CTEs, EXISTS subqueries) without fighting an ORM.
- Materializing the graph as database tables separated expensive computation from the API hot path and made caching straightforward.
- PostgreSQL as a job queue works fine at this throughput (Reddit's rate limit caps at ~1.66 rps anyway). But the pattern muddies the schema. Is this a data store or a task broker? Both, apparently.
- A monorepo Makefile that does everything is great for discoverability (`make help`). At 468 lines, it's also a maintenance burden.

## Receipts

- **Root Makefile:** `Makefile` — 468 lines, refactored to root in commit `4eadd22` (Nov 12, 2025)
- **sqlc config:** `backend/sqlc.yaml` — present since first commit `88cb0b9`
- **Backend entrypoints:** `backend/cmd/server/main.go`, `backend/cmd/crawler/main.go`, `backend/cmd/precalculate/main.go`
- **Frontend entry:** `frontend/src/main.tsx`, `frontend/src/App.tsx` (464 lines)
- **API routes:** `backend/internal/api/routes.go` — 30+ endpoints
- **Docker Compose:** `backend/docker-compose.yml` — 8 services
- **Nginx config:** `frontend/nginx.conf` — reverse proxy + SPA fallback
- **Schema:** `backend/migrations/schema.sql` — consolidated schema definition
- **Migration count:** 37 files (000001–000028, up + down pairs)
- **Architecture doc:** `docs/architecture.md` — Mermaid diagrams
- **Overview doc:** `docs/overview.md` — data flow documentation
- **Custom renderers:** `frontend/src/rendering/InstancedNodeRenderer.ts`, `LinkRenderer.ts`, `EdgeBundler.ts`, `SDFTextRenderer.ts`
- **Spatial index:** `frontend/src/spatial/Octree.ts`
- **PR #203:** "Replace react-force-graph-3d with InstancedMesh renderer" (Feb 8, 2026)
