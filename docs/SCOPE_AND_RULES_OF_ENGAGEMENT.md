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
