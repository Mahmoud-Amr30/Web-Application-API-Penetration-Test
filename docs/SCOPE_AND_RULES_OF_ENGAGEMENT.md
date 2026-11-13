# Scope & Rules of Engagement — Draft

**Target selected:** DVWA

**Deployment approach:** Docker Compose

**Status:** Draft; team/instructor confirmation required before active testing

## In-scope target

- The team's own local DVWA deployment only.
- Planned local URL: `http://127.0.0.1:4280` when using the official Compose port mapping.
- The DVWA web application and only those local API modules explicitly enabled and documented.

## Explicitly out of scope

- Public websites, third-party systems, unrelated containers, host services, LAN devices, and Internet targets.
- Denial-of-service, stress testing, destructive database actions, persistence, malware, or attacks against real users.
- Accessing or collecting real personal data.
- **Bulk data extraction** — no mass download/exfiltration of application data (for example full database dumps or automated scraping of all pages/records) beyond the minimal data needed to demonstrate a specific validated finding.
- **Social engineering** — no phishing, pretexting, or manipulation of any person, team member, or third party.
- **Credential harvesting** — no collection, retention, or reuse of credentials, cookies, or session tokens belonging to real users; only synthetic lab test accounts are used, and secrets are redacted from evidence.
- **Assets merely discovered during reconnaissance** — any host, service, container, endpoint, or application noticed while working in the lab is NOT automatically in scope. It may be tested only after the team explicitly documents it as an authorized target addition (and instructor approval where required).
- **Disruptive or destructive activities outside approved lab conditions** — no actions that break the lab for other members (resource exhaustion, teardown of shared services, data destruction) outside an explicitly approved destructive exercise with a pre-made backup.

## Safety controls

- Publish DVWA only on `127.0.0.1`; do not bind to `0.0.0.0` or a LAN IP.
- Use synthetic test accounts/data.
- Back up lab state before destructive exercises.
- Redact passwords, cookies, tokens, and secrets from evidence.
- Stop testing if traffic is reaching a non-lab target or unexpected sensitive data appears.

## Still to confirm

- Instructor approval of the local lab scope and testing categories.
- Docker host machine and versions.
- Test accounts and reset procedure.
- Whether the API lab is enabled and the additional DEPI API test coverage required.
- Testing window and team members authorized to perform testing.

This draft is not a substitute for any instructor-required written authorization.
