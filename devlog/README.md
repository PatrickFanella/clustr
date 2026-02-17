# Cutroom Devlog — Clustr

A devlog series about building [Clustr](https://github.com/subculture-collective/reddit-cluster-map) — a full-stack app for crawling, analyzing, and visualizing Reddit communities as interactive network graphs.

## Project at a Glance

- Crawl Reddit communities, compute relationship graphs, render as interactive 3D/2D visualizations
- Stack: Go 1.24 (backend) · React 19 + Three.js + D3.js (frontend) · PostgreSQL 17 · Docker Compose
- Repo created: June 1, 2025
- Total commits: 695+ across all branches
- Contributors: 1 primary developer, 1 collaborator (26-day active period), 466 AI-generated commits (GitHub Copilot SWE agent)
- Current state: backend builds and tests pass; frontend builds, 95.5% tests pass; no live demo
- Codebase: 30+ API endpoints, 5 visualization modes, 28 database migrations, custom WebGL renderer for 100k+ nodes
- Key gaps: 19 failing frontend tests, no git tags/releases, dual rendering path, 1.76 MB bundle
- Development model: human direction + AI implementation (issue-driven Copilot workflows)
- Org: `subculture-collective` (transferred from `onnwee` in Oct 2025)

---

## Series Index

| Part | Title | Focus |
|------|-------|-------|
| [01](01-origin-story.md) | Origin Story | Why this existed, first commits, deploy struggle, initial README |
| [02](02-architecture-and-technical-spine.md) | Architecture & Technical Spine | Repo structure, Go backend, React frontend, data flow, PostgreSQL schema |
| [03](03-feature-development-phase-1.md) | Feature Development Timeline (Phase 1) | May 31 – Jun 9, 2025: init → crawler → graph API → 3D visualization |
| [04](04-feature-development-phase-2.md) | Feature Development Timeline (Phase 2) + Product Direction | Aug 2025 – Feb 2026: revamp, Copilot sprint, org transfer, 2.0 rendering engine |
| [05](05-tooling-dx-and-engineering-process.md) | Tooling, DX, and Engineering Process | Makefile, CI/CD, tests, sqlc, load testing, bundle tracking |
| [06](06-failures-bugs-and-hard-lessons.md) | Failures, Bugs, and Hard Lessons | Memory leaks, stale closures, 8 NDJSON attempts, build/run results |
| [07](07-team-dynamics-and-ai-development.md) | Team Dynamics, AI Tooling, and Development Velocity | Contributor timeline, team changes, AI-augmented workflows, velocity analysis |
| [08](08-postmortem-and-whats-next.md) | Postmortem + What's Next | Current state assessment, top 10 engineering tasks, presentation roadmap |

---

## Timeline

| Date Range | Phase | Key Milestones |
|------------|-------|----------------|
| May 31, 2025 | Inception | `88cb0b9` — first commit: Go server skeleton, Dockerfile, schema |
| May 31, 2025 | Core backend | `e8ce97f` — complete crawler system (2,463 insertions, 43 files) |
| Jun 1–3, 2025 | Deploy battle | `9f57ae8`→`4639066` — 6 deploy attempts across 3 days; "it's working!" |
| Jun 2, 2025 | Frontend init | `08c53e5` — Vite + React + Tailwind scaffolding (PR #2) |
| Jun 7, 2025 | Graph milestone | `96df4e3` — Graph API + react-force-graph-3d visualization (PR #3) |
| Jun 7, 2025 | Precalculation | `923cdb1` — graph precalculation pipeline (PR #8) |
| Jun 8, 2025 | Service split | `4de386e` — crawler separated into own binary/container |
| Jun 9 → Aug 15 | Gap (67 days) | No commits |
| Aug 15–26, 2025 | Revamp | `77e1ddc`→`24ae93c` — OAuth fixes, precalc repairs, "graph won't load" |
| Aug 26 → Oct 14 | Gap (49 days) | No commits |
| Oct 14–17, 2025 | Hardening begins | `f38c197` — efficiency improvements; `8130589` — CI added |
| Oct 17, 2025 | AI sprint starts | `af3953c` — first copilot-swe-agent commit; issues #35–#51 batch-created |
| Oct 18, 2025 | Org transfer + new collaborator | Repo moves to `subculture-collective`; Patrick Fanella joins |
| Oct 18–26, 2025 | Bug fix + feature sprint | PRs #53–#118: security, DX, community detection, observability |
| Oct 28 – Nov 1 | Hardening sprint | PRs #119–#133: security audit, release process, profiling |
| Nov 12–13, 2025 | Deploy + restructure | Root Makefile, Docker naming, CI updates; Patrick Fanella's last commits |
| Nov 13 → Feb 7 | Gap (86 days) | No commits |
| Feb 7, 2026 | "Moving on from MVP" | `c1898ec`; 58 issues + 6 epics created (#138–#195) |
| Feb 7–12, 2026 | 2.0 rendering engine | PRs #196–#246: InstancedMesh, Web Workers, octree, LOD, theme, a11y |
| Feb 12, 2026 | Last main commit | PR #246: visual regression testing merged |

---

## How this series was written

Everything is sourced from git history, GitHub PRs/issues (via `gh` CLI), file contents, and build/test output from running the project locally. Where information was unavailable from the repo, it's noted.

Written February 17, 2026.
