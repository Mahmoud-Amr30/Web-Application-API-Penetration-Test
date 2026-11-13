# Project State — DEPI Project 2

**Last updated:** 2026-11-13

**Current phase:** Phase 0 — Foundation

**Current team target:** Sunday, 2026-11-15 (reported by project owner)

## Confirmed facts
- Selected DEPI project: Project 2 — Web Application & API Penetration Test.
- Team size: six.
- Team members: Mahmoud Amr Sherif Ahmed, Ehab Gaber Abdellatif, Mohamed Monir Mahmoud, Abdullah Mosad Abdullah, Seifeldin Shady Abdelsalam, Sondos.
- Sondos's full name and GitHub username are not yet supplied.
- Target selected: DVWA.
- Deployment approach selected: Docker Compose.
- Two XMind maps have been completed: 01 — DEPI Project Phases and 02 — DVWA Target Architecture.
- XMind workspace: https://app.xmind.com/share/sLwah4wJ?xid=E8jpkK6Z&sheet-id=34e508c5-955b-4ab0-93a2-94036e5eccd9
- Existing GitHub repo is private: `Mahmoud-Amr30/Web-Application-API-Penetration-Test`.
- Local clone exists under `~/Documents/Freelance/DEPI_Work/Web-Application-API-Penetration-Test`.
- A local branch `docs/architecture-submission` was created.
- The previously reported `rsync -avn` was a dry run; it did not copy files.
- OpenCode had not been used before the user's latest report; do not assume it has reviewed or edited the repository.

## Not yet implemented / not evidenced
- DVWA Docker deployment has not yet been evidenced.
- Docker and Docker Compose availability on the selected host has not been reported.
- No Burp traffic capture has been reported.
- No application map based on actual browsing has been reported. The XMind target architecture is a planning map, not proof of a running lab.
- No findings or PoCs are confirmed.

## Decisions
- DEC-001: Project 2 selected.
- DEC-002: DVWA selected as primary target.
- DEC-003: Docker Compose selected as deployment approach.
- DEC-004: Both XMind maps completed; shared link recorded above.
- DEC-005: Use GitHub Issues/Projects/PRs to coordinate team work.
- DEC-006: Keep the DVWA port loopback-only; do not expose it to LAN or Internet.

## Schedule
- Phase 0 intended start: Saturday 2026-11-07.
- The instructor earlier requested an architecture with an implemented part and asked for it before Tuesday 2026-11-10. That date has passed.
- The project owner reports the current Phase 0 team target is Sunday 2026-11-15.
- Current instructor acceptance status and the exact revised architecture-submission requirement are not recorded. Reconfirm if possible.

## Pending decisions / details
- Docker host machine and Docker/Compose versions.
- Exact DVWA repository commit/version deployed.
- Whether DVWA API lab prerequisites are met and how Week 3 API coverage gaps will be handled.
- Written Rules of Engagement / allowed tests / test accounts.
- Final owners and reviewers for all six members.
- Sondos's full name and GitHub username.

## Immediate priorities
1. Add this foundation pack to `docs/architecture-submission` and review the diff.
2. Have one teammate deploy DVWA using the official Docker Compose instructions and capture actual proof.
3. Have the team assign Phase 0 owners using `planning/TEAM_HANDOFF.md`.
4. Open GitHub Issues and a Project board so each member can work independently.
5. Reconfirm architecture submission format/acceptance with the instructor.
