# Phase 0 — Team Handoff (Read This First)

**Purpose:** Allow all six members to continue Phase 0 without relying on Abdullah being online.
**Team target:** Sunday, 2026-11-15.
**Important:** The architecture deadline previously mentioned by the instructor (before Tuesday, November 10) has passed. Confirm the current acceptance window and format with the instructor.

## What is already done
- Project 2 selected.
- Target selected: DVWA.
- Deployment approach selected: Docker Compose.
- Six members identified; Sondos's full name/GitHub profile still needs to be added.
- XMind maps completed: `01 — DEPI Project Phases` and `02 — DVWA Target Architecture`.
- XMind link: https://app.xmind.com/share/sLwah4wJ?xid=E8jpkK6Z&sheet-id=34e508c5-955b-4ab0-93a2-94036e5eccd9
- GitHub repository cloned locally; branch `docs/architecture-submission` created.
- Earlier `rsync -avn` was only a dry run. It copied nothing.

## Verified publication status (2026-11-13)

| Step | Status |
|---|---|
| Foundation commit | **Committed** — `b0f74d1 docs: establish Phase 0 project foundation` on `docs/architecture-submission` |
| Remote push | **Completed** — branch is pushed to `origin/docs/architecture-submission` and up to date |
| Pull Request | **Open** — [PR #1](https://github.com/Mahmoud-Amr30/Web-Application-API-Penetration-Test/pull/1) (`docs/architecture-submission` → `main`) |
| Team review | **Pending** — a teammate must review the PR before merge |
| Merge into `main` | **Pending** — only after review approval |

> PR status is recorded above with the real PR number. No false completion status is recorded; review and merge remain pending.

## What is NOT done yet
- DVWA is not yet evidenced as deployed.
- No first Burp request/response evidence has been supplied.
- Final owners/reviewers are not recorded.
- No finding or PoC is confirmed.

## Team tasks to claim (do not leave all tasks with one person)
Each member should claim one task in GitHub Issues. Suggested Phase 0 task set:

| ID | Task | Required output | Owner | Reviewer |
|---|---|---|---|---|
| P0-01 | Apply foundation pack through PR | Updated README, project state, docs | TBD | TBD |
| P0-02 | Run DVWA using official Docker Compose | Running local app + Docker status evidence | TBD | TBD |
| P0-03 | Verify lab isolation | Loopback port mapping evidence + scope notes | TBD | TBD |
| P0-04 | Review architecture and XMind | Review comments / requested corrections | TBD | TBD |
| P0-05 | Fill six-member role matrix and add Sondos | Roles, reviewers, GitHub access | TBD | TBD |
| P0-06 | Create Issues and GitHub Project board | Board with owner/status/priority/phase | TBD | TBD |
| P0-07 | Check DVWA API lab requirements | API coverage gap note and instructor question if needed | TBD | TBD |
| P0-08 | Confirm instructor requirements | Required format, acceptance status, revised date | TBD | TBD |

## Definition of Done for Phase 0
- Updated project foundation is merged to `main` through a reviewed PR.
- All six members have repository access (Sondos after username is known).
- Each task has an owner, reviewer, due date, and visible artifact.
- DVWA Docker Compose deployment is evidenced, not merely planned.
- The port is verified to bind only to `127.0.0.1`.
- Architecture, two XMind maps, lab evidence, and open decisions are linked.
- Instructor has been asked to confirm the overdue Architecture submission format and the practical implementation evidence expected.
- API coverage limitations are explicitly recorded.

## How team members should submit work
1. Claim an Issue and set themselves as Assignee.
2. Create a task branch from `main`.
3. Add/update the relevant document or sanitized evidence.
4. Open a Pull Request and link the Issue.
5. Ask another member to review.
6. Merge after review; update the Project board.

Do not put raw Burp files, cookies, credentials, tokens, private keys, or unredacted data in GitHub.
