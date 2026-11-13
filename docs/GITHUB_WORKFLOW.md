# GitHub Workflow — Team Collaboration Guide

This guide explains how our six-member team uses GitHub day to day. No prior Git/GitHub experience is assumed. If anything here is unclear, ask in the team chat before guessing.

Our repository is **private**, so everything here stays within the team.

## Core concepts

### GitHub Issues

An **Issue** is one unit of work: a task, a research question, a test case, or a finding. Issues are our task list and discussion threads in one place.

- Create issues with one of our templates (Task / Research / Test case / Finding) — the templates prompt you for the information reviewers need.
- Every issue gets exactly **one assignee** (the owner) and, where applicable, a named reviewer.
- Per team decision **DEC-009**, GitHub Issues are the primary task-tracking source of truth. If work is not in an issue, it is not tracked.

### GitHub Projects

A **Project** is a shared status board built on top of Issues. We use it to see who is doing what at a glance.

Suggested board statuses:

| Status | Meaning |
|---|---|
| **Backlog** | Known work, not yet started. |
| **Ready** | Defined and unblocked; an owner can pick it up. |
| **In Progress** | An assignee is actively working on it. |
| **In Review** | A pull request is open and awaiting review. |
| **Blocked** | Waiting on something (note what, in a comment). |
| **Done** | Merged/verified; the issue can be closed. |

Suggested priorities: **P0** (urgent — blocks other work or a deadline), **P1** (required this phase/week), **P2** (important, can wait), **P3** (nice to have).

### Assignee

The **assignee** is the single accountable owner of an issue. One issue = one assignee. Helpers can be mentioned in comments, but accountability stays with the assignee.

### Reviewer

The **reviewer** is a second member who checks the work before it merges. For findings and important changes, the reviewer must be someone **other than the author** (independent review). Reviewers leave comments on the pull request and either request changes or approve.

### Labels

Labels classify issues at a glance. Suggested starter set:

- `type:task`, `type:research`, `type:test-case`, `type:finding`
- `phase-0`, `week-1`, `week-2`, `week-3`, `week-4`
- `priority:P0` … `priority:P3`
- `blocked`

The task template already applies `type:task`; add the phase/week label when creating the issue.

## Task ownership rules

1. Claim an issue by setting yourself as assignee **before** starting work.
2. Never start work on an issue someone else already owns without agreeing in comments first.
3. Move the issue through the Project board statuses as you work.
4. If you become blocked, set status **Blocked** and comment why.
5. Work that is not in an issue does not exist — open an issue first, even for small changes.

## Branch naming

Create a branch from `main` for each issue, before changing anything:

```bash
git checkout main
git pull origin main
git checkout -b <branch-name>
```

Pattern: `<area>/<short-description>`, lowercase, hyphens, no spaces.

Examples:

- `docs/project-documentation` — documentation work
- `docs/phase-0-handoff` — Phase 0 handoff pack
- `test/web-xss` — web application testing work
- `test/api-authorization` — API/authorization testing work
- `report/final-report` — final report assembly

Tip: number the branch after the issue if you like, e.g. `docs/42-team-roles`.

## Commit naming

Small, focused commits with a clear type prefix:

- `docs:` documentation changes
- `test:` test cases, test work, testing scripts
- `evidence:` sanitized evidence additions
- `fix:` corrections
- `chore:` housekeeping (`.gitignore`, templates, settings)

Example: `docs: add testing methodology and evidence conventions`

Write the message so a teammate understands the change without opening the diff.

## Pull Request workflow

1. Push your branch: `git push -u origin <branch-name>`.
2. Open a Pull Request (PR) from your branch **to `main`** using `.github/PULL_REQUEST_TEMPLATE.md`.
3. **Connect the PR to its Issue:** put `Closes #<issue-number>` in the PR description (GitHub then links and auto-closes the issue on merge). If the PR only partly completes the issue, write `Related to #<issue-number>` instead.
4. Request the reviewer as a reviewer on the PR.
5. The reviewer reads the diff, checks the checklist, and comments.
6. Make requested changes as new commits on the same branch — the PR updates automatically.
7. Do **not** merge your own PR without an approving review.

## Review process

A reviewer checks:

- Does the change do what the linked issue says?
- Is the content accurate (no invented findings, endpoints, or results)?
- Is any evidence sanitized (no credentials, cookies, tokens, raw Burp/HAR files)?
- Does the change stay inside the approved DVWA lab scope?

Respond to every review comment — either by fixing it or by replying with your reasoning. Resolve conversations before merge.

## Merge process

- Merge only after at least one approving review.
- Merge on GitHub (the web interface) using the merge button; keep it simple and consistent for the whole team.
- After merging, delete the feature branch if GitHub offers it, and update the Project board (issue auto-closes if you wrote `Closes #`).

## Branch protection — plan limitation

GitHub's native **branch protection** (for example "require review before merging to `main`") is a paid feature for **private repositories**: it needs GitHub Pro (personal account) or GitHub Team (organization). On a Free personal-account private repository it is generally not available.

- Check `Settings → Branches` in the repository. If the option to require reviews is unavailable on our current plan, we enforce the same rules **by team agreement**: nobody pushes directly to `main`, every change goes through a PR with one approving review.
- This does not change the rule, only how it is enforced.

## Evidence security

- Only **sanitized** evidence may be committed (`evidence/sanitized/`). See `evidence/README.md`.
- Never commit raw Burp projects, raw HAR/HTTP captures, credentials, cookies, session tokens, API keys, private keys, or real personal data — not even "temporarily".
- `.gitignore` blocks the common file types, but review is the real safety net.

## How each member demonstrates their contribution

Contribution is visible automatically if you follow the workflow:

- **Issues you own** show what you were accountable for.
- **Commits** carry your name on the changes you wrote.
- **PRs you opened** and **reviews you gave** show both authorship and independent review work.
- Document your own contribution: keep your issue comments updated (what you did, what you found, blockers), and record yourself as tester/owner in documents like `docs/DVWA_DOCKER_LAB.md`.

Do not commit on someone else's behalf from your own account — it hides their authorship. Each member pushes their own work under their own account.
