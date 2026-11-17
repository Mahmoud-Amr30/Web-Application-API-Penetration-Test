# Team Roles — Six-Member Workstream Allocation

The team has six members. Final assignments have not been supplied yet; this document does not invent member skills or silently assign roles.

| Member | GitHub | Phase 0 workstream preference | Assigned responsibility | Reviewer |
|---|---|---|---|---|
| Mahmoud Amr Sherif Ahmed | @Mahmoud-Amr30 | TBD | TBD | TBD |
| Ehab Gaber Abdellatif | @Ehab226 | TBD | TBD | TBD |
| Mohamed Monir Mahmoud | @mohamedmonir375-hue | TBD | TBD | TBD |
| Abdullah Mosad Abdullah | @zer0-sCripTF | Architecture/context owner (temporary) | TBD | TBD |
| Seifeldin Shady Abdelsalam | @Seifeldinshady | TBD | TBD | TBD |
| Sondos | Full name and GitHub username pending | TBD | TBD | TBD |

## Suggested workstreams (not assignments)

The six workstreams below are suggestions for allocation, not assignments. Every assignment stays **TBD until the team agrees**. Skills vary between members — no assumption of equal cybersecurity experience is made; workstreams that need more background should be paired with a reviewer who has it.

### WS-1 — Project Coordination & GitHub Maintenance

- **Responsibilities:** Repository settings and structure, issue/Project board upkeep, template maintenance, PR hygiene, tracking overdue follow-ups (for example the architecture submission format).
- **Expected deliverables:** Maintained board and issues; updated templates; coordination notes in `planning/TEAM_HANDOFF.md`.
- **Required knowledge:** Basic Git/GitHub (covered by `docs/GITHUB_WORKFLOW.md`); organized communication.
- **Dependencies:** All other workstreams report status to this one; depends on all six members having repository access.
- **Suggested reviewer relationship:** Another coordinator-minded member reviews structural/organizational changes; the repository owner reviews access-related changes.

### WS-2 — Lab Environment & Target Management

- **Responsibilities:** Deploy and operate the DVWA Docker Compose lab, verify loopback-only exposure, record implementation evidence, own lab start/stop and reset procedures.
- **Expected deliverables:** Evidence pack in `docs/DVWA_DOCKER_LAB.md` and `evidence/sanitized/` (docker ps, versions, commit, port binding).
- **Required knowledge:** Docker/Docker Compose basics; willingness to follow the official DVWA runbook exactly.
- **Dependencies:** Depends on an agreed host machine and Docker availability; blocks all testing workstreams until evidenced.
- **Suggested reviewer relationship:** A member from WS-3 (who will test against the lab) verifies the evidence.

### WS-3 — Reconnaissance & Application Mapping

- **Responsibilities:** Build the actual application map from observed pages/modules, request/response inventory, and the authentication/session flow documentation (Week 1 deliverables).
- **Expected deliverables:** Application map, request/response inventory with evidence IDs, auth/session flow document, first-cut test plan.
- **Required knowledge:** HTTP fundamentals, Burp Proxy basics.
- **Dependencies:** Requires the lab running (WS-2) and an agreed scope; feeds test design in WS-4/WS-5.
- **Suggested reviewer relationship:** WS-4 lead reviews the map for testing relevance; WS-6 QA-checks evidence IDs.

### WS-4 — Web Application Security Testing

- **Responsibilities:** Design, execute, and validate the Week 2 input-based tests (SQLi, XSS, CSRF, SSRF, file upload, path traversal, LFI/RFI) per `docs/METHODOLOGY.md`.
- **Expected deliverables:** Test-case issues, payload list, PoCs where confirmed, sanitized evidence, CVSS assessments, remediation notes.
- **Required knowledge:** Burp Suite (Repeater/Intruder), the vulnerability classes being tested, disciplined evidence handling.
- **Dependencies:** Requires WS-2 (lab) and WS-3 (map/inventory); uses the test-case and finding templates.
- **Suggested reviewer relationship:** Cross-review between WS-4 members; independent review of every finding by a member not involved in its test.

### WS-5 — API, Authentication & Access Control Testing

- **Responsibilities:** Week 3 areas — JWT, session hijacking/fixation, IDOR/BOLA, broken authentication, excessive data exposure, rate limiting, business logic, API request testing.
- **Expected deliverables:** Test-case issues, API test collection, API findings report, auth/session findings with impact statements.
- **Required knowledge:** HTTP/session mechanics, authorization concepts; API tooling as needed.
- **Dependencies:** Requires WS-2, WS-3, and the P0-08 API-prerequisite check (mod_rewrite/vendor dependencies) before claiming API coverage; unsupported areas are marked **Not applicable** with a gap note.
- **Suggested reviewer relationship:** A WS-4 member cross-reviews authorization/session findings; instructor question routed through WS-1 if coverage gaps need approval.

### WS-6 — Evidence, Quality Assurance & Reporting

- **Responsibilities:** Enforce evidence naming/redaction, run the methodology's quality gate, QA documents, assign stable finding IDs, assemble Week 4 reporting (final report, executive summary, attack chain, retest checklist, demo materials).
- **Expected deliverables:** Reviewed evidence directory, finding ID register, QA checklists, assembled Week 4 deliverables.
- **Required knowledge:** Attention to detail; reporting/writing; basic understanding of all test areas.
- **Dependencies:** Consumes outputs of WS-3/WS-4/WS-5; coordinates final deadlines with WS-1.
- **Suggested reviewer relationship:** Each report section is reviewed by the workstream that produced the underlying tests; the final report gets one full independent review.

## Team allocation rules

1. **One accountable owner per task.** Every task has exactly one owner (assignee); helpers can contribute but ownership does not diffuse.
2. **Independent review for important findings.** No member reviews their own finding; at least one other member validates each Confirmed finding (quality gate in `docs/METHODOLOGY.md`).
3. **Document your own contribution.** Members record their own work in issues, documents, and evidence ownership — never commit under someone else's account.
4. **No single point of knowledge.** Each workstream keeps its notes in this repository so any member can pick it up; avoid being the only person who knows how something works.
5. **Roles remain TBD until the team agrees.** Nothing in this document assigns permanent roles; allocation happens by explicit team agreement, recorded in GitHub Issues and in the roster table above.
6. Members can swap workstreams by agreement; every swap updates the roster table and the related issues.

Each member should claim one workstream/task in GitHub. Every task needs one owner, one reviewer, a due date, and a concrete artifact. Team members can swap tasks by agreement. Add Sondos to the private GitHub repository after obtaining her GitHub username.
