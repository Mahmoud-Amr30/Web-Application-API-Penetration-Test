# Instructions for AI Assistants

## Confirmed project context
- DEPI Track: Infrastructure & Penetration Testing.
- Project: Project 2 — Web Application & API Penetration Test.
- Target selected by team: DVWA (Damn Vulnerable Web Application).
- Deployment approach selected: Docker Compose. Deployment itself is not yet verified unless `docs/DVWA_DOCKER_LAB.md` contains evidence.
- Team size: six. Sondos is confirmed as the sixth member; her full name and GitHub username remain unknown.
- XMind maps 01 — DEPI Project Phases and 02 — DVWA Target Architecture have been completed. Use the shared link in `README.md`; do not claim that the AI created them.
- Phase 0 intended start: 2026-11-07.
- Current team target for Phase 0 completion: Sunday, 2026-11-15, as reported by the project owner.
- Architecture submission is overdue. The instructor previously requested an architecture with a real implemented part; current acceptance format/status should be reconfirmed.

## State discipline
- Read `PROJECT_STATE.md` before proposing work.
- Distinguish CONFIRMED, NOT YET IMPLEMENTED, UNKNOWN, and NEEDS APPROVAL.
- Do not assume Docker is installed or that DVWA has been deployed.
- Do not invent DVWA version/commit, deployment host, target URL, API capabilities, vulnerabilities, evidence, team assignments, Sondos's identity, or instructor requirements.
- Update the state only when the user provides new facts or actual evidence supports a change.

## Safety
- Do not scan, exploit, or test anything outside the locally deployed DVWA lab and explicitly approved scope.
- Keep the Compose port bound to `127.0.0.1`; never expose DVWA to the public Internet or LAN.
- Never commit raw Burp projects, HAR files, credentials, cookies, tokens, API keys, private keys, or unredacted personal data.
- Never run destructive commands or remove Docker volumes unless the user explicitly requests and understands the consequences.

## Git workflow
- Work on the existing branch `docs/architecture-submission` unless the owner explicitly changes it.
- Do not push or commit without explicit permission.
- Prefer small edits and show a diff after changes.
- Do not overwrite existing files blindly.

## Deliverable discipline
- Treat the uploaded DEPI brief as the source of official week-by-week requirements.
- Keep proposed improvements distinct from DEPI-mandated deliverables.
- Treat the architecture as project/lab architecture unless target-internal details have been observed.
