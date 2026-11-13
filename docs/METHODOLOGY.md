# Testing Methodology — Validation Lifecycle

**Target:** DVWA (local Docker Compose lab, loopback-only)

**Status:** Working document; refine as the team gains experience with the lab.

This document defines how the team plans, executes, and validates tests so results are trustworthy and comparable. It describes the team's process — it does not claim that every DEPI test item is supported by DVWA. Where the running target does not support a planned test area, the result is recorded as **Not applicable** and the coverage gap is documented for instructor follow-up.

## Result vocabulary

Every executed test ends in exactly one of these statuses. This vocabulary is shared by test cases, findings, and reports.

| Status | Meaning |
|---|---|
| **Confirmed** | The behavior was reproduced at least once by the tester and independently verified by a reviewer, with sanitized evidence and a defensible explanation of security impact. |
| **Not observed** | The test was executed as designed within the approved scope and the vulnerable behavior did not occur. This is a result, not a failure of the test. |
| **Inconclusive** | The test could not produce a reliable answer — for example, a broken test setup, an ambiguous response, or missing evidence. The test must be repeated or redesigned before the finding is decided. |
| **Not applicable** | The test area is unsupported by the current target or environment (for example an API capability DVWA does not provide), so it was not executed. The gap is documented and, where relevant, raised with the instructor. |
| **Blocked** | The test was intended but could not be started — for example, missing tooling, missing authorization confirmation, or the lab being unavailable. Record the blocker. |

## Finding quality gate

**A vulnerability cannot be classified as Confirmed unless all of the following are true:**

1. It is **reproducible** — the steps reliably produce the behavior, ideally repeated by more than one tester.
2. It has **evidence** — sanitized evidence IDs (`REQ-`, `RES-`, `SCR-`, `LOG-`, `POC-`) that demonstrate the behavior.
3. It has a **defensible explanation of security impact** — the tester can explain what an attacker could realistically achieve and why it matters.
4. It has been **independently reviewed** — a second member has checked the reproduction and impact reasoning.

Anything failing this gate stays **Inconclusive**, **Not observed**, or is marked **Not a finding** during review. False positives are expected and are not embarrassing; unvalidated "Confirmed" findings are the real risk.

## Testing lifecycle

### 1. Scope verification

Before any test session:

- Confirm the target is the approved local DVWA lab at `http://127.0.0.1:4280` (loopback-only).
- Confirm the Rules of Engagement (`docs/SCOPE_AND_RULES_OF_ENGAGEMENT.md`) cover the planned test category.
- Confirm the lab is actually running (`docker compose ps` or a local request) before blaming the target for failures.

### 2. Test design

- Turn each DEPI coverage area into one or more concrete test cases using the `03-test-case` issue template.
- Define preconditions, exact procedure, and the expected result **before** executing.
- Identify which areas the target is likely to support; flag likely **Not applicable** areas early so the instructor question is not left to Week 3.

### 3. Execution

- Run the test exactly as designed; if you deviate, record the deviation.
- One accountable owner per test; a second member may repeat key tests for confirmation.
- Keep all traffic inside the approved lab. If traffic unexpectedly reaches a non-lab target, stop immediately.

### 4. Evidence collection

- Capture evidence during the test, not afterwards from memory.
- Apply the evidence naming conventions from `evidence/README.md`.
- Sanitize immediately (redact credentials, cookies, tokens) — raw artifacts stay out of Git and out of `evidence/sanitized/`.

### 5. Result validation

- Compare the observed result against the expected result defined in the test design.
- Assign one of the five result statuses.
- Record the result in the test-case issue with evidence IDs.

### 6. False-positive elimination

Common false-positive sources in a DVWA-style lab:

- Tool output that "looks vulnerable" but the payload did not actually execute (for example, a payload reflected but not interpreted).
- Expected application behavior mistaken for a flaw (the app is deliberately vulnerable — check that the observed behavior is the *vulnerability being tested*, not a different known one).
- Configuration-dependent results (DVWA security level, PHP settings) that change behavior between testers.

Countermeasures: verify the effect manually (did the injected code actually execute?), repeat the test, and have a second member confirm. If it cannot be settled, record **Inconclusive**.

### 7. Finding documentation

- Confirmed behaviors become findings using the `04-finding` issue template.
- Each finding includes: affected asset, category, severity, description, reproduction steps, expected vs. observed result, impact, evidence references, and proposed remediation.
- Findings get stable IDs (`FIND-001`, ...) assigned by the reporting workstream to avoid numbering conflicts.

### 8. Severity assessment

- Use CVSS as the scoring method for Week 2 deliverables and apply the same thinking throughout.
- Severity reflects realistic impact **in this lab's context**; a finding that would be critical in production may rate differently here — explain the reasoning either way.
- Severity is proposed by the tester and confirmed during review; disagreements are resolved in the team, not silently.

### 9. Remediation

- For each Confirmed finding, write a concrete remediation note (what a developer would change and why).
- In DVWA's case, remediation notes can reference the secure-code implementations the DVWA project itself ships for comparison, where applicable.
- Remediation notes feed the Week 4 remediation plan.

### 10. Retesting

- After a fix or configuration change, rerun the original test procedure and record whether the finding persists.
- Retest results use the same vocabulary (**Confirmed / Not observed / Inconclusive / Not applicable / Blocked**) and new evidence IDs.
- The retest checklist is a Week 4 deliverable; keep it updated as findings are retested throughout, not only at the end.

## Honest coverage

- The four-week DEPI coverage plan is tracked in `planning/TASK_BACKLOG.csv` and `planning/DELIVERABLES_MATRIX.md`.
- Tests the target cannot support are recorded as **Not applicable** with a short gap note — never faked, never silently skipped.
- Coverage claims in the final report are limited to what was actually tested and validated by this methodology.
