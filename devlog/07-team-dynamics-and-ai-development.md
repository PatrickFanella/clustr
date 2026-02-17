---
title: "Team Dynamics, AI Tooling, and Development Velocity"
series: "Cutroom Devlog"
part: 7
date: "2026-02-17"
tags: ["devlog", "openwork", "cutroom", "team", "ai", "copilot", "velocity", "collaboration"]
summary: "Who contributed, when, how the team changed, and what 466 AI-generated commits look like in practice."
---

## Context

Part 7 of the Cutroom Devlog series. Who contributed, when, how the team changed, and what role AI tooling played. All from git history, PR records, and GitHub metadata.

## What We Did

### Contributor Timeline

The project's contributor history, extracted via `git shortlog -sn --all`:

| Contributor | Commits | Active Period |
|------------|---------|---------------|
| copilot-swe-agent[bot] | 413 | Oct 17, 2025 – Feb 12, 2026 |
| ⓪ηηωεε忧世 (onnwee) | 170 | May 31, 2025 – Feb 10, 2026 |
| Copilot | 53 | Oct 28, 2025 – Feb 9, 2026 |
| Patrick Fanella | 46 | Oct 18 – Nov 12, 2025 |
| 0nnwee | 7 | May 31 – Jun 1, 2025 |
| dependabot[bot] | 3 | Oct 19, 2025 – Feb 8, 2026 |
| onnwee | 3 | Oct 19 – Oct 20, 2025 |

**Note:** `0nnwee`, `onnwee`, and `⓪ηηωεε忧世` are the same person (GitHub login: `onnwee`) with different git author configurations at different times. Combined: **180 commits**.

`copilot-swe-agent[bot]` and `Copilot` represent GitHub's AI agent operating under different attribution formats. Combined: **466 commits**.

### Phase 1: Solo Development (May 31 – Jun 9, 2025)

Built entirely by one developer. The commit authorship used the git name `0nnwee`, later changing to `⓪ηηωεε忧世` starting June 1 (when commit signing or config changed). Every commit, every architectural decision, every line of code from one person.

This phase produced the backend architecture, crawler, precalculation pipeline, frontend scaffolding, and deployment infrastructure.

### Phase 2: The Gap and Revamp (Jun 9 – Oct 14, 2025)

Development paused for 67 days, resumed briefly (Aug 15-26) with 7 solo commits, then paused again for 49 days. The commit messages during the August revamp ("she runnin", "alter brewing") suggest someone working alone, iterating fast.

### Phase 3: Team Expansion (Oct 14 – Nov 13, 2025)

Three things changed simultaneously around October 17-18:

**1. The repo moved to an organization.** The repository transferred from `onnwee/reddit-cluster-map` to `subculture-collective/reddit-cluster-map`. This is evidenced by PR URLs — PRs before #57 use the `onnwee` namespace, while #57 onward use `subculture-collective`.

**2. A collaborator joined.** Patrick Fanella (`PatrickFanella`) appeared as a contributor on October 18. His commit history shows:

| Date | Activity |
|------|----------|
| Oct 18 | First commits: applying Copilot suggestions, merging PRs (#58, #60, #64) |
| Oct 18–19 | PR reviews and merges, `.env.example` updates, Makefile refinements |
| Oct 24–25 | CommunityMap updates, community API reviews, label fixes |
| Oct 26 | Metrics issues filed and fixed |
| Oct 28–29 | Security hardening reviews, OAuth token refresh |
| Oct 31 | Security audit and release process issues |
| Nov 12 | Last commits: "moved docs and delete package-lock" (#134), deploy branch merges (#135, #136) |

Patrick's contributions were concentrated on code review, PR merges, applying Copilot suggestions, and configuration updates. He created and closed issues (#107, #109, #111, #113, #130) and managed several deployment-related PRs.

**3. AI-assisted development began.** The `copilot-swe-agent[bot]` started generating code and PRs on October 17. The workflow was deliberate: issues were batch-created (#35-#51) and assigned to the Copilot agent for implementation.

### Patrick Fanella's Departure

Patrick Fanella's last commits are dated **November 12, 2025** (PRs #134, #135, #136 — all deployment-related merges and documentation moves). After this date, no further commits from this contributor appear in the repository.

From repo evidence:
- His active contribution period was **26 days** (Oct 18 – Nov 12)
- He contributed **46 commits** during this period
- His work focused on code review, PR management, and deployment operations
- There are no issues, comments, or commits explaining the departure
- The project continued with the original developer and AI tooling after this point

The reasons for the departure aren't documented in the repository.

### Phase 4: Solo + AI (Nov 13, 2025 – Feb 2026)

After November 13, the project went quiet for 86 days, its longest gap. When development resumed on February 7, 2026, it was the original developer (`onnwee`) and the Copilot SWE agent. Patrick Fanella does not appear in the February 2026 commit history.

The February sprint was the most productive period:
- 58 issues created in one day
- 50+ PRs merged in 5 days
- `onnwee` handled strategic direction, issue creation, and PR reviews; `copilot-swe-agent[bot]`/`Copilot` handled implementation

### AI Development Patterns

The Copilot SWE agent's contributions follow identifiable patterns:

**Issue → Branch → PR.** Human creates an issue with clear acceptance criteria. Copilot creates a branch (named `copilot/<issue-slug>`), starts with an "Initial plan" commit showing the agent's understanding of the task, then implements and opens a PR. Human reviews, applies suggestions, merges.

**Trial and error.** For complex features, the agent often needed multiple attempts. The NDJSON streaming example (8 attempts) is extreme, but 2-3 attempt PRs are common for cross-cutting changes.

**Documentation-heavy PRs.** The Copilot agent generates more docs than a human would: implementation summaries, integration guides, PR summaries, changes summary files. The repo accumulated `IMPLEMENTATION_SUMMARY_*.md`, `CHANGES_SUMMARY.md`, `PR_SUMMARY.md`, and similar files describing intermediate states rather than final implementations.

**"Changes before error encountered."** Multiple commits have this message (e.g., `9495bfb`, `276b84d`, `0c24526`). The Copilot agent hit errors during execution and committed partial work before failing. These partial commits were sometimes merged despite being incomplete.

### Velocity Analysis

Commits per active period:

| Period | Duration | Commits | Commits/Day |
|--------|----------|---------|-------------|
| Phase 1 (May 31 – Jun 9) | 10 days | ~30 | 3.0 |
| Summer revamp (Aug 15–26) | 11 days | 7 | 0.6 |
| Copilot sprint (Oct 14 – Nov 13) | 30 days | ~350 | 11.7 |
| 2.0 push (Feb 7–12) | 5 days | ~120 | 24.0 |

The AI-augmented phases show higher commit velocity. The February 2026 sprint, 24 commits per day, would be unsustainable for a human. This velocity came from the Copilot agent executing against well-defined issues while the human focused on review and direction.

Commit velocity is not value delivery, though. Many Copilot commits are "Initial plan" messages, documentation files, or multiple rounds of code review feedback. The actual feature-per-day rate is lower than the raw numbers suggest.

### Contributor Diversity

The project had a narrow contributor base:
- **1 consistent contributor** (onnwee) across the entire project lifetime
- **1 collaborator** (Patrick Fanella) for 26 days
- **AI tooling** (Copilot) providing the majority of code volume
- **3 dependabot PRs** for security updates

No external contributors, no community PRs. The repository is under the `subculture-collective` organization, but no other organization members contributed to the codebase based on commit history.

## Key Decisions

1. Adopting AI-first development. Using GitHub Copilot SWE agent as the primary code generator was a deliberate bet. Issues were written specifically to be actionable by the agent.

2. Batch issue creation. Creating 22 issues at once (Oct 17) and 58 issues at once (Feb 7) enabled parallel AI execution, but also created a "quantity over quality" dynamic. Many issues were addressed superficially rather than deeply.

3. Maintaining human review. Despite the AI generating most code, every PR was reviewed and merged by a human. Quality control preserved, but the human reviewer became the bottleneck.

## What Went Wrong / Friction

- Knowledge concentration. One human developer, bus factor of 1. The AI generates code but doesn't carry institutional knowledge between sessions.

- AI-generated code understanding. When 466 of 695 commits are AI-generated, the human developer's understanding of the codebase is shallower than if they'd written it all. That's a maintenance risk.

- Documentation noise. The Copilot agent generates implementation summaries and integration guides for every PR. Dozens of markdown files that become misleading as the code evolves.

- Incomplete commits getting merged. The "changes before error encountered" pattern means the AI agent sometimes committed broken code, and those commits made it into the main branch.

## What We Learned

- AI-assisted development works best with clear, atomic issues. The Copilot agent handled "add this endpoint", "fix this bug", "write tests for this component" fine. It couldn't do "implement NDJSON streaming across the full stack."

- A single human reviewer can sustain a surprisingly high merge rate when AI generates the PRs. The February sprint averaged 10 merged PRs per day.

- Team attrition in a small project is a structural risk. When a two-person team becomes a one-person team, you can maintain velocity with AI tooling, but there's no second perspective on code review or architectural decisions.

- The development model evolved from "solo hacking" (Phase 1) to "human director + AI implementer" (Phase 4). That model seems sustainable for continued development, though "sustainable" and "good" aren't the same thing.

## Receipts

- **Contributor stats:** `git shortlog -sn --all` — 413 copilot-swe-agent, 170 ⓪ηηωεε忧世, 53 Copilot, 46 Patrick Fanella, 7 0nnwee, 3 dependabot, 3 onnwee
- **Total commits:** 695 (all branches combined)
- **Repo creation:** June 1, 2025 (GitHub, originally under `onnwee`)
- **Org transfer:** Evidenced by PR namespace change at PR #57 (Oct 18, 2025)
- **Patrick Fanella active period:** Oct 18 – Nov 12, 2025 (46 commits)
- **Patrick Fanella last commit:** PR #136 merge (Nov 12, 2025)
- **First Copilot commit:** `af3953c` (Oct 17, 2025)
- **Issue batch 1:** Issues #35–#51 (Oct 17, 2025)
- **Issue batch 2:** Issues #138–#195 (Feb 7, 2026)
- **"Changes before error encountered" commits:** `9495bfb`, `276b84d`, `0c24526`
- **NDJSON streaming attempts:** 8 branches, 7 closed PRs, 1 merged (PR #224)
- **Peak velocity:** Feb 7–12, 2026 — ~120 commits in 5 days
