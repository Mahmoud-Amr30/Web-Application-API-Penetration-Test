# Evidence Directory

This directory holds the team's penetration-testing evidence for the authorized local DVWA lab.

**Golden rule:** the Git repository contains **sanitized evidence only**. Raw captures, Burp projects, and anything with credentials, cookies, or session tokens stay in protected storage outside this repository (see `SECURITY.md`).

## Evidence ID naming conventions

Every evidence item gets a stable ID so findings can reference it unambiguously. Format:

```
PREFIX-NNN-short-description.ext
```

Examples:

- `REQ-001-login-request-redacted.txt` — a redacted HTTP **request** capture.
- `RES-001-login-response-redacted.txt` — a redacted HTTP **response** capture.
- `SCR-001-dvwa-browser.png` — a redacted **screenshot**.
- `LOG-001-docker-compose-ps.txt` — a **log** or command output capture.
- `POC-001-sqli-reproduction.md` — a **proof-of-concept** script or written reproduction for a confirmed finding.

Rules:

- Numbers increment per type (`SCR-001`, `SCR-002`, ...) and are **never reused**.
- Choose a short lowercase description; no spaces (use hyphens).
- The member who collects the evidence assigns the ID and records tester and date next to it.
- If evidence must be replaced after review feedback, use a new number and note the replacement.

## Evidence ownership

- The **collector** (the member who produced the capture) owns the evidence item and is accountable for its redaction and accuracy.
- The **workstream reviewer** for that member's area checks that redaction is complete before it is merged.
- Ownership is recorded in the task/issue and, for findings, in the finding document itself.

## Evidence collection

- Collect only inside the approved scope (`docs/SCOPE_AND_RULES_OF_ENGAGEMENT.md`).
- Record the context for each item: what test produced it, when, on which authorized target, and with which account/role.
- Follow `docs/METHODOLOGY.md`: a finding can only be **Confirmed** when reproducible evidence exists.
- Use the `.github/ISSUE_TEMPLATE/03-test-case.yml` and `04-finding.yml` forms so evidence IDs are linked to tests and findings.

## Redaction requirements

Before any file enters `evidence/sanitized/`:

- Replace passwords, cookies, session identifiers, security tokens, API keys, and any secrets with `REDACTED`.
- Blur or crop screenshots so no credentials, tokens, or unrelated personal data are visible.
- Remove real personal data. Only synthetic test-account data from the lab is acceptable.
- Remove internal host details that are not needed (full local paths, usernames, machine names) unless relevant and approved.
- Redacted HTTP examples should keep only what is needed to reproduce the test: method, path, relevant headers/parameters, and the relevant part of the response.

## Relationship between evidence and findings

- Every finding lists the evidence IDs that support it (`Evidence references` field in the finding template).
- A finding without reproducible evidence stays **Inconclusive** — it cannot be marked **Confirmed**.
- Evidence may support several findings; do not duplicate the same capture under new IDs.
- Evidence linked to a finding that later becomes **Not a finding** is kept and annotated; it is not deleted.

## Safe storage of raw traffic

- The working Burp Suite project file, raw HAR/HTTP exports, and packet captures remain on the collector's own machine or another team-approved protected location (for example an encrypted disk or private password-protected share).
- Raw files are **never** committed to Git, even temporarily. Deleting them in a later commit does not remove them from Git history.
- Only copies that passed the redaction checklist go into `evidence/sanitized/`.

## Allowed and prohibited repository contents

**Allowed (sanitized only):**

- Redacted screenshots (`.png`, `.jpg`).
- Redacted text captures of requests/responses (`.txt`, `.md`).
- Command output logs with secrets removed (`.txt`, `.md`).
- Minimal proof-of-concept scripts or written steps that contain no credentials.

**Prohibited (never commit):**

- Burp Suite project files (`.burp`) or raw state files.
- Unredacted HAR files or raw HTTP captures.
- Credentials, cookies, session tokens, API keys, private keys, `.env` files.
- Database dumps or lab database files.
- Real personal data of anyone.

`.gitignore` blocks most of these extensions automatically, but tools are not a substitute for review: every pull request touching evidence is checked by a second member.
