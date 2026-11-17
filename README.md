# DEPI Project 2 — Web Application & API Penetration Test

**Program:** Digital Egypt Pioneers Initiative (DEPI)

**Track:** Infrastructure & Penetration Testing

**Target:** DVWA (Damn Vulnerable Web Application)

**Deployment approach:** Docker Compose (selected; not yet evidenced as running)

**Repository visibility:** Private

**Current phase:** Phase 0 — Foundation

**Phase 0 team target date:** Sunday, 2026-11-15 (team-reported)

## Project purpose

This repository is the team's shared workspace for planning, implementing, documenting, validating, and presenting an authorized Web Application & API penetration-testing project using DVWA as the primary practice target.

The team will build a local DVWA lab with Docker Compose, map the application and its HTTP behavior, plan and execute the DEPI-required tests within the agreed lab scope, preserve sanitized evidence, validate findings, and prepare the final report and retest materials.

## Current verified status

| Item | Current status |
|---|---|
| Selected DEPI project | Confirmed: Project 2 — Web Application & API Penetration Test |
| Team size | 6 members |
| Target | Confirmed: DVWA |
| Deployment approach | Docker Compose selected; actual deployment still pending evidence |
| XMind maps | Both maps completed by the team |
| Phase 0 start | Saturday, 2026-11-07 (planned start; work remains open) |
| Phase 0 team target | Sunday, 2026-11-15 (reported by project owner) |
| Architecture submission | Overdue; current instructor acceptance/format must be reconfirmed |
| Written lab Rules of Engagement | Draft / needs team-instructor confirmation |
| Burp capture | Not yet evidenced |

## Team

| Member | GitHub |
|---|---|
| Mahmoud Amr Sherif Ahmed | [@Mahmoud-Amr30](https://github.com/Mahmoud-Amr30) |
| Ehab Gaber Abdellatif | [@Ehab226](https://github.com/Ehab226) |
| Mohamed Monir Mahmoud | [@mohamedmonir375-hue](https://github.com/mohamedmonir375-hue) |
| Abdullah Mosad Abdullah | [@zer0-sCripTF](https://github.com/zer0-sCripTF) |
| Seifeldin Shady Abdelsalam | [@Seifeldinshady](https://github.com/Seifeldinshady) |
| Sondos | Full name and GitHub profile pending |

No final task ownership is implied by this roster. Assign owners and reviewers in GitHub Issues / Projects.

## XMind — completed

The team has completed these two maps:

1. **01 — DEPI Project Phases**
2. **02 — DVWA Target Architecture**

[Open the shared XMind workspace](https://app.xmind.com/share/sLwah4wJ?xid=E8jpkK6Z&sheet-id=34e508c5-955b-4ab0-93a2-94036e5eccd9)

The share URL above is the one supplied by the project owner. Confirm both sheets are accessible to all members. Do not recreate the maps; update them only when the team agrees a material change is needed.

## Target and lab

DVWA is the primary training target. The selected deployment approach is Docker Compose. The official DVWA project describes a PHP/MariaDB application and documents a Docker Compose deployment. The recommended local mapping is loopback-only, with the application available at `http://127.0.0.1:4280` when the official Compose configuration is used.

The deployment is **not marked complete** until a team member records the Docker Compose status, browser access, and evidence in `docs/DVWA_DOCKER_LAB.md`.

Official target source: https://github.com/digininja/DVWA

## DEPI deliverables

- **Week 1:** Application map; Burp project file; request/response inventory; authentication/session flow diagram; test plan.
- **Week 2:** PoCs for validated applicable input-based vulnerabilities; payload list; evidence; CVSS scores; remediation notes.
- **Week 3:** API test collection; API findings report; IDOR/BOLA PoCs where confirmed; authentication/session findings; impact statements.
- **Week 4:** Final Web/API Pentest Report; attack-chain diagram; retest checklist; remediation plan; demo video/screenshots.

Important: DVWA's API lab requires additional configuration. The team must verify API coverage against the DEPI Week 3 requirements and document any gaps or approved supplementary lab work. Do not claim full API coverage merely because DVWA is running.

## How work is tracked

- **GitHub Issues:** task and test ownership.
- **GitHub Projects:** shared status board.
- **Pull Requests:** review and merge of documentation/code changes.
- **`PROJECT_STATE.md`:** single source of truth for current project facts.
- **`planning/TEAM_HANDOFF.md`:** immediate Phase 0 tasks and team handoff.
- **`docs/DVWA_DOCKER_LAB.md`:** deployment runbook and real implementation evidence.

## Safety and evidence rules

- Keep DVWA local and loopback-only. Do not expose this intentionally vulnerable app to the public Internet or the wider LAN.
- Only test the agreed DVWA lab and approved local APIs.
- Do not commit raw Burp project files, raw HAR/HTTP captures, credentials, cookies, session tokens, API keys, private keys, or unredacted personal data.
- Commit only sanitized screenshots and minimal redacted request/response examples.
- Keep raw artifacts in approved restricted storage.

See `AGENTS.md`, `CONTRIBUTING.md`, and `SECURITY.md` for collaboration and safety rules.
