# Decision Log

| ID | Date | Decision | Status / notes |
|---|---|---|---|
| DEC-001 | 2026-11-07 or earlier | DEPI Project 2 selected | Confirmed by project owner |
| DEC-002 | 2026-11-08 or earlier | DVWA selected as the primary target | Confirmed by project owner; exact version/commit still to record |
| DEC-003 | 2026-11-13 | Docker Compose selected for DVWA deployment | Confirmed approach; lab deployment not yet evidenced |
| DEC-004 | 2026-11-08 or earlier | XMind 01 — DEPI Project Phases completed | Confirmed by project owner |
| DEC-005 | 2026-11-08 or earlier | XMind 02 — DVWA Target Architecture completed | Confirmed by project owner |
| DEC-006 | 2026-11-13 | Phase 0 team target is Sunday 2026-11-15 | Reported by project owner; not a confirmed final course deadline |
| DEC-007 | 2026-11-13 | Keep DVWA local and loopback-only | Safety requirement for the deliberately vulnerable application |
| DEC-008 | 2026-11-13 | Repository remains private | See details below |
| DEC-009 | 2026-11-13 | GitHub Issues are the primary task-tracking source of truth | See details below |
| DEC-010 | 2026-11-13 | Raw Burp projects and unredacted traffic must not be committed to Git | See details below |
| DEC-011 | 2026-11-13 | Team collaboration uses a lightweight branch and Pull Request workflow | See details below |

Add new rows for changes. Do not silently overwrite prior decisions. Exact selection dates can be corrected by the team if meeting notes establish a more precise date.

## Decision details — DEC-008 to DEC-011

### DEC-008 — Repository remains private

- **Decision:** The team repository stays private for the whole project.
- **Reason:** The repository contains penetration-testing methodology, sanitized evidence, and findings for a deliberately vulnerable application. Public exposure is unnecessary and creates risk.
- **Chosen approach:** Keep `Mahmoud-Amr30/Web-Application-API-Penetration-Test` private; share access only with the six team members (and the instructor if required).
- **Alternatives:** A public showcase repository was considered and rejected — course grading and team safety do not require public visibility.
- **Impact:** Members need explicit collaborator access; Sondos cannot be added until her GitHub username is known. The team must document plan limitations for private repositories (for example branch protection, see `docs/GITHUB_WORKFLOW.md`).
- **Status:** Agreed as of 2026-11-13.

### DEC-009 — GitHub Issues are the primary task-tracking source of truth

- **Decision:** All project tasks, test cases, research, and findings are tracked as GitHub Issues; GitHub Projects is the shared board over them.
- **Reason:** The team needs one shared, auditable place for ownership and status that does not depend on any single member being online.
- **Chosen approach:** One issue per task with a single assignee; use the repository's issue templates; statuses on the Project board follow `docs/GITHUB_WORKFLOW.md`.
- **Alternatives:** Chat threads, local spreadsheets, or the CSV backlog alone were rejected as primary sources — they do not scale to six members or survive offline gaps. `planning/TASK_BACKLOG.csv` remains a planning aid, not the tracker.
- **Impact:** Work not tracked in an issue is not counted as project work. Members claim tasks by self-assigning.
- **Status:** Agreed as of 2026-11-13.

### DEC-010 — Raw Burp projects and unredacted traffic must not be committed to Git

- **Decision:** Burp Suite project files, raw HAR/HTTP captures, packet captures, and any material containing credentials, cookies, or session tokens are never committed to Git.
- **Reason:** Even in a private educational repository, committing secrets or raw traffic creates permanent history exposure and bad habits for real engagements.
- **Chosen approach:** Raw artifacts stay in protected local/approved storage; only sanitized copies enter `evidence/sanitized/` using the ID conventions in `evidence/README.md`. `.gitignore` blocks the common file types and review is the backstop.
- **Alternatives:** Committing raw files to a separate private repo was considered and rejected — it duplicates the risk and review burden.
- **Impact:** Findings must reference sanitized evidence IDs; raw evidence retention becomes each collector's local responsibility.
- **Status:** Agreed as of 2026-11-13; consistent with `SECURITY.md`.

### DEC-011 — Team collaboration uses a lightweight branch and Pull Request workflow

- **Decision:** All changes reach `main` through a short-lived branch and a reviewed Pull Request with one independent approving review.
- **Reason:** A six-member beginner team needs a simple, uniform process that makes every contribution visible and reviewable without heavy tooling.
- **Chosen approach:** The workflow defined in `docs/GITHUB_WORKFLOW.md` (branch naming, commit prefixes, PR linked to its issue, one reviewer, merge after approval). Native branch protection on `main` may be unavailable on the current GitHub plan for a private repository, so the rule is enforced by team agreement.
- **Alternatives:** Direct pushes to `main` (rejected — no review trail) and full GitFlow-style branching (rejected as too heavy for documentation work).
- **Impact:** Slightly more Git steps per change; in exchange, every member's contribution is auditable and no one can silently change the shared record.
- **Status:** Agreed as of 2026-11-13.
