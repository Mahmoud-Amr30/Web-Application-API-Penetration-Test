# Sanitized Evidence

This folder contains **only** evidence that has passed the redaction checklist in [`../README.md`](../README.md).

## Before you add anything here

- [ ] Every credential, cookie, session token, and security token is replaced with `REDACTED`.
- [ ] Screenshots show no credentials, tokens, or personal data (blur/crop if needed).
- [ ] Only synthetic lab test-account data is present.
- [ ] The file has a proper evidence ID name (`REQ-NNN`, `RES-NNN`, `SCR-NNN`, `LOG-NNN`, `POC-NNN` + short description).
- [ ] The collector, date, and originating test are recorded in the related issue/finding.
- [ ] A second member reviewed the file before merge.

## What never goes here

- Raw Burp Suite project files, raw HAR/HTTP captures, or packet captures.
- Lab database files (`.sqlite`, `.db`, SQL dumps).
- Anything containing a real secret — even "just temporarily".

Raw material stays in protected storage outside this repository. If a raw file is accidentally committed, notify the team immediately and follow `SECURITY.md`.
