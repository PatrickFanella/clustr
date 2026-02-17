---
title: "Origin Story"
series: "Cutroom Devlog"
part: 1
date: "2026-02-17"
tags: ["devlog", "openwork", "cutroom", "origin", "reddit", "graph-visualization"]
summary: "How Clustr started, what the first commits looked like, and the three-day fight to get it deployed."
---

## Context

Part 1 of the Cutroom Devlog series. First commits, first deploy, first struggles.

## What We Did

### The Idea

Crawl Reddit, map the relationships between communities, render it as an interactive 3D graph. The GitHub repo description:

> A full-stack application for collecting, analyzing, and visualizing Reddit communities and their user interactions as network graphs.

The repo was created on June 1, 2025 under the `onnwee` GitHub account. The first commit (`88cb0b9`) landed on May 31, local development starting before the repo was even on GitHub.

### Day One: Init to Auto-Crawl

The first commit (`88cb0b9`, May 31) packed a lot into one commit:

- A Go backend with `cmd/server/main.go` (34 lines)
- A Dockerfile (empty — placeholder)
- A docker-compose.yml (also empty)
- An API handler for subreddits
- A `db.go` with PostgreSQL connection setup
- A `schema.sql` with the first table definition
- sqlc configuration for query generation

Hours later, the second commit (`e8ce97f`, May 31) dropped 2,463 insertions across 43 files. This single commit introduced:

- The full crawler system: OAuth authentication (`auth.go`), Reddit API client (`reddit.go`), job queue (`queue.go`, `jobs.go`), rate limiting (`ratelimit.go`), and worker orchestration (`worker.go`)
- User subreddit discovery (`user_subs.go`)
- API handlers for posts, comments, crawl jobs, users, and edges
- Complete database schema with 6 tables: subreddits, users, posts, comments, crawl_jobs, subreddit_edges
- All corresponding sqlc queries and generated Go code
- A working Docker Compose configuration with PostgreSQL
- A Makefile with basic build targets

Two more commits that same day modularized utilities (`7ecba63`) and added user subscription crawling (`5adfaef`). By the end of May 31, the project had a working crawler, API, and database — all from one developer.

### The README

The initial README (`98a39d1`, May 31) declared the project's scope:

- Crawl subreddits for posts and comments
- Store normalized data in PostgreSQL
- Build a graph based on shared participation
- Serve it via API for a frontend to render

This wasn't aspirational. The backend already supported most of this by commit 6.

### Deployment Struggle

The next phase was getting it deployed. The commit messages:

| Commit | Date | Message |
|--------|------|---------|
| `9f57ae8` | Jun 1 | "set up for deploy try 1" |
| `edb0f1f` | Jun 1 | "deploy try 2" |
| `c72aca4` | Jun 1 | "try 3" |
| `375154a` | Jun 2 | "try 3" (again) |
| `17aa016` | Jun 2 | "next day still trying" |
| `4639066` | Jun 3 | "it's working!" |

Three days of fighting Docker Compose networking, deploy scripts, and Makefile targets. The deploy script (`scripts/deploy.sh`) was rewritten multiple times. The docker-compose.yml was restructured repeatedly (77 insertions, 40 deletions in the final push).

PR #1 merged the deploy branch on June 2.

### The Frontend Arrives

The same day as the successful deploy, a frontend was added (`08c53e5`, Jun 2):

- Vite + React + TypeScript + Tailwind CSS
- A GitHub Actions deploy workflow
- 4,505 insertions (mostly the `package-lock.json`)

This was merged as PR #2 from `feature/frontendinit`. At this point the frontend was a shell. No graph visualization yet, just scaffolding.

## Key Decisions

1. Go for the backend. The first commit chose Go. The project needed OAuth HTTP clients, database access, and a REST API. Go fit: fast compilation, good HTTP standard library, solid PostgreSQL ecosystem via `lib/pq`.

2. sqlc for database access. Rather than an ORM, the project used sqlc from day one (`sqlc.yaml` in the first commit). It generates type-safe Go code from SQL queries. The pattern stuck through the entire project lifetime.

3. PostgreSQL as the only data store. No Redis, no external cache, no message queue. The crawl job queue was implemented directly as a database table (`crawl_jobs`). Simpler deployment, but the coupling persisted.

4. Monorepo. Backend and frontend in the same repository from the start. Easier deployment coordination, but a single history that interleaves unrelated changes.

5. Docker Compose for deployment. Not Kubernetes, not a PaaS. Docker Compose from day one kept infrastructure simple at the cost of scalability options.

## What Went Wrong / Friction

- The deployment took three days. The backend was built in a single day, but Docker Compose networking, volume mounts, and environment variables ate three more. The commit messages say it plainly: "next day still trying."

- The first Dockerfile was empty. Commit `88cb0b9` included a `Dockerfile` and `docker-compose.yml` that were both 0 bytes. Placeholders that needed to be filled before anything could run.

- Monolithic second commit. The `e8ce97f` commit (2,463 insertions, 43 files) bundled the entire crawler, API expansion, and database schema into one commit. Good luck bisecting that later.

- No tests. The first 15+ commits contained zero test files. The crawler was hitting a live API, writing to a real database, and serving results over HTTP, all with no test coverage.

## What We Learned

- Building the backend took one day. Deploying it took three.
- Starting with sqlc paid off. The SQL-first approach meant the data model was defined clearly from day one, and the generated Go code stayed in sync with the schema.
- Empty placeholder files (that 0-byte Dockerfile) create false confidence. No Dockerfile is better than one that does nothing.
- The full pipeline (Reddit API → PostgreSQL → graph computation → frontend rendering) touches every layer. Everything had to come together at once.

## Receipts

- **First commit:** `88cb0b9` (May 31, 2025) — "init", 11 files, 118 insertions
- **Auto-crawl commit:** `e8ce97f` (May 31, 2025) — "auto crawl", 43 files, 2,463 insertions
- **README added:** `98a39d1` (May 31, 2025) — 130 lines of project documentation
- **Deploy attempts:** `9f57ae8`, `edb0f1f`, `c72aca4`, `375154a`, `17aa016` (Jun 1–2, 2025)
- **Deploy success:** `4639066` (Jun 3, 2025) — "it's working!"
- **PR #1:** Deploy branch merge (Jun 2, 2025)
- **Frontend scaffolding:** `08c53e5` (Jun 2, 2025) — "frontend added and auto deploy configured"
- **PR #2:** Frontend init merge (Jun 2, 2025)
- **Repo creation date:** June 1, 2025 (GitHub metadata)
- **Original org:** `onnwee` (later transferred to `subculture-collective`)
- **Key files:** `backend/cmd/server/main.go`, `backend/internal/crawler/`, `backend/migrations/schema.sql`, `backend/sqlc.yaml`
