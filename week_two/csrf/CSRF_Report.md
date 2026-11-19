# Cross-Site Request Forgery (CSRF) Vulnerability Report

## 1. Executive Summary & Scope

| Field | Details |
|---|---|
| **Target URL** | `http://127.0.0.1:8080/vulnerabilities/csrf/` (Damn Vulnerable Web Application - DVWA) |
| **Vulnerability Type** | Cross-Site Request Forgery (CSRF) |
| **Assessment Context** | Local DVWA instance (authorized lab environment assumed) |
| **Report Date** | 2026-11-19 |

### Tested Levels/Scenarios

| Level/Scenario | Description | Outcome Summary |
|---|---|---|
| **Low** | Direct password change request without anti-CSRF token or Referer validation. Request intercepted and parameters modified (`password_new` / `password_conf` set to `Pass123`). | **Confirmed exploitable.** Password change was confirmed. |
| **Medium (Referer validation)** | Referer header validation behavior tested in Burp Repeater. | **Partially observed.** Referer validation mechanism was tested; specific accept/reject outcomes per Referer value are **Not observed / Needs verification**. |
| **Medium (Chaining: Stored XSS → CSRF action)** | Chained with Stored XSS at `/vulnerabilities/xss_s/` using a URL-encoded `<img src=x onerror=...>` payload (including `%26` and other encoded characters) to trigger the password change action. | **Confirmed impact via chaining.** Server returned `Password Changed.` response. This represents a same-origin chained exploitation path, not direct cross-site forgery from an external attacker-controlled origin. |

### Summary of Findings

- **Low:** The CSRF password change functionality accepted a direct request without any anti-CSRF token or Referer enforcement. Intercepting and modifying `password_new` and `password_conf` to `Pass123` resulted in a confirmed password change, indicating high risk of account takeover when a victim is authenticated.
- **Medium:** Referer-based validation was present/tested in Burp Repeater, but the exact behavior for missing/foreign/valid Referer values was not fully documented. Despite this control, the password change action could be triggered from a same-origin context by chaining with Stored XSS (`/vulnerabilities/xss_s/`) using a URL-encoded payload, producing a `Password Changed.` response. This demonstrates that Referer checks alone are insufficient when other injection vulnerabilities allow same-origin request execution.

**Overall risk (scoped to provided observations):** High for Low security level (direct CSRF). Medium security reduces direct cross-site risk in principle via Referer validation, but observed chaining with Stored XSS achieved the same sensitive outcome from a same-origin context.

---

## 2. Technical Breakdown & Exploitation Methodology

### 2.1 Target Functionality
- **Endpoint:** `http://127.0.0.1:8080/vulnerabilities/csrf/`
- **Sensitive action:** Change user password
- **Observed parameters:** `password_new`, `password_conf` (provided). Other parameters (e.g., submit/action parameter names, HTTP method details beyond GET usage at Low): **Not observed / Needs verification**.

### 2.2 Manual Testing & Burp Suite Workflow

#### Low Security Level
1. **Baseline access:** Navigated to the DVWA CSRF page at `http://127.0.0.1:8080/vulnerabilities/csrf/` while authenticated to DVWA (session details: **Not observed / Needs verification**).
2. **Initiate password change:** Submitted a password change request through the application interface.
3. **Intercept in Burp Suite:** Captured the outgoing request in Burp Suite Proxy (Interception enabled).
4. **Parameter tampering:** Modified the request parameters:
   - `password_new` = `Pass123`
   - `password_conf` = `Pass123`
5. **Forward request:** Released the modified request to the server.
6. **Validate outcome:** Observed confirmation that the password change succeeded (exact response body/title details beyond confirmation: **Not observed / Needs verification**).

**Observation:** No anti-CSRF token was required and no Referer validation was enforced at Low, allowing direct manipulation and successful processing of the state-changing request.

#### Medium Security Level — Referer Validation (Burp Repeater)
1. **Capture request:** Obtained a legitimate CSRF/password-change request (source/context: **Not observed / Needs verification**).
2. **Send to Repeater:** Forwarded the request to Burp Suite Repeater for controlled header manipulation.
3. **Test Referer behavior:** Modified/tested the `Referer` header to evaluate validation behavior (specific test cases, header values, and corresponding responses: **Not observed / Needs verification**).
4. **Assess control:** Determined that Referer-based validation was part of the Medium-level behavior; however, complete pass/fail matrix was not provided.

#### Medium Security Level — Stored XSS Chaining (Burp Suite / Browser Context)
1. **Identify injection point:** Used the Stored XSS functionality at `/vulnerabilities/xss_s/` (specific vulnerable parameter/field and storage behavior: **Not observed / Needs verification**).
2. **Craft payload:** Prepared a URL-encoded `<img src=x onerror=...>` payload to trigger the password change action from within the application’s origin. The payload used `%26` and other encoded characters (exact full payload string: **Not observed / Needs verification**).
3. **Submit/store payload:** Injected the payload via the Stored XSS page so it would execute when rendered (submission/request details: **Not observed / Needs verification**).
4. **Trigger execution:** Loaded/rendered the stored content in an authenticated context (victim/session context details: **Not observed / Needs verification**), causing the browser to execute the injected `onerror` logic.
5. **Validate outcome:** Observed server/application response indicating `Password Changed.`

**Observation:** Although Medium employed Referer validation against direct cross-site requests (behavior details partially unverified), chaining with Stored XSS enabled same-origin execution of the password change action, bypassing the practical protection intent of Referer-only CSRF defenses.

---

## 3. Proof of Concept (PoC) & Payloads

### 3.1 Low — Direct CSRF (Intercept & Modify)

**Target:** `http://127.0.0.1:8080/vulnerabilities/csrf/`  
**Observed vulnerable behavior:** No anti-CSRF token; no Referer validation; password change processed via direct request manipulation.

**Parameters (observed):**
- `password_new=Pass123`
- `password_conf=Pass123`

**Other request components (method, full query/body, cookies, additional parameters):** Not observed / Needs verification.

**Step-by-step reproduction:**
1. Log in to DVWA and ensure security level is set to **Low** (UI/navigation steps: **Not observed / Needs verification**).
2. Browse to `http://127.0.0.1:8080/vulnerabilities/csrf/`.
3. Configure the browser to proxy traffic through Burp Suite and enable interception.
4. Submit a password change request from the CSRF page.
5. In Burp Proxy, intercept the request and modify:
   - `password_new` to `Pass123`
   - `password_conf` to `Pass123`
6. Forward the modified request to the server.
7. Confirm the application indicates the password was changed (observed).

**Evidence of success (as provided):** Password change confirmed after forwarding modified request.

### 3.2 Medium — Referer Header Validation Testing (Burp Repeater)

**Target:** `http://127.0.0.1:8080/vulnerabilities/csrf/`

**Step-by-step reproduction (as observed):**
1. Capture a password change request for the CSRF endpoint.
2. Send the request to **Burp Suite Repeater**.
3. Modify/test the `Referer` header values and resend to observe validation behavior.

**Exact Referer test values and server responses:** Not observed / Needs verification.

### 3.3 Medium — Stored XSS Chain to Trigger CSRF Action

**Targets:**  
- Stored XSS injection point: `http://127.0.0.1:8080/vulnerabilities/xss_s/`  
- CSRF action endpoint: `http://127.0.0.1:8080/vulnerabilities/csrf/`

**Payload (as provided):**
- Type: `<img src=x onerror=...>` (event-handler based)
- Encoding: URL-encoded form used, including `%26` and other encoded characters
- **Exact full payload string:** Not observed / Needs verification
- **Decoded/injected JavaScript logic (exact):** Not observed / Needs verification

**Encoding notes (provided):**
- Ampersand/parameter separators and/or script characters were URL-encoded (explicitly noted: `%26` present, plus other encoded characters). Exact encoding scheme and full encoded payload are not provided.

**Step-by-step reproduction (as observed):**
1. Log in to DVWA with Medium security level configured (configuration steps: **Not observed / Needs verification**).
2. Navigate to the Stored XSS page: `http://127.0.0.1:8080/vulnerabilities/xss_s/`.
3. Submit the URL-encoded `<img src=x onerror=...>` payload designed to trigger the password change request to `/vulnerabilities/csrf/` (exact submitted fields/request: **Not observed / Needs verification**).
4. Cause the stored payload to render/execute in an authenticated browser context.
5. Observe the application/server response indicating `Password Changed.`

**Evidence of success (as provided):** Response contained `Password Changed.`

---

## 4. Impact & False Positive Validation

### 4.1 Confirmed State Changes / Technical Impact

| Level/Scenario | Confirmed Impact | Evidence Basis (provided) |
|---|---|---|
| **Low (Direct CSRF)** | Password change successfully performed with attacker-chosen values (`Pass123` / `Pass123`). | Request intercepted/modified and password change confirmed. |
| **Medium (Stored XSS → CSRF chain)** | Password change action executed, resulting in `Password Changed.` response. | Stored XSS payload execution in authenticated context triggered CSRF action. |

**Security impact:** Unauthorized password change can lead to account takeover, loss of account integrity, and potential loss of confidentiality/availability for the affected user account. Actual account compromise beyond the confirmed password change action: **Not observed / Needs verification**.

### 4.2 False Positive Validation and Exploitation Distinction

- **Low — Direct cross-site exploitation:** **Confirmed (not a false positive).** The sensitive action succeeded without anti-CSRF token or Referer validation when parameters were modified and submitted, demonstrating missing CSRF defenses.
- **Medium — Direct cross-site exploitation (external origin):** **Not conclusively confirmed/denied from provided data.** Referer validation was tested in Burp Repeater, but specific outcomes for cross-origin/missing Referer cases were not provided. Therefore, direct cross-site exploitability at Medium is **Not observed / Needs verification**.
- **Medium — Same-origin chaining (Stored XSS + CSRF action):** **Confirmed exploitable path via chaining.** The observed `Password Changed.` result occurred through execution of a stored payload within the application context (`/vulnerabilities/xss_s/` impacting `/vulnerabilities/csrf/`). This is distinct from a pure cross-site request initiated by an external third-party site:
  - **Direct CSRF:** Attacker site causes victim browser to send a cross-site request to the target.
  - **XSS + CSRF chaining (observed):** Attacker-controlled script executes in the target’s origin (via Stored XSS), then performs the sensitive action with same-origin context/privileges; Referer-based CSRF defenses may not prevent this.
- **Mitigated/non-exploitable cases:** No specific mitigated request/response cases were provided. Any claim that Medium fully blocks direct cross-site CSRF requires verification with documented Referer test results (**Not observed / Needs verification**).

### 4.3 Confidence Assessment
- **High confidence:** Low-level direct CSRF leads to password change; Medium-level chained Stored XSS leads to `Password Changed.` response.
- **Medium/Unverified:** Exact HTTP requests/responses, session handling, full payload strings, Referer validation decision matrix, and whether direct external-origin CSRF is blocked at Medium.

---

## 5. CVSS v3.1 Score & Vector

CVSS v3.1 base metrics are estimated from provided observations. Environmental and temporal factors are not included.

| Tested Level/Scenario | CVSS v3.1 Vector | Base Score | Severity | Justification |
|---|---|---:|---|---|
| **Low — Direct CSRF (confirmed)** | `CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:U/C:H/I:H/A:H` | **8.8** | **High** | **AV:N**: exploitable remotely over the network. **AC:L**: no special conditions beyond a victim visiting/loading attacker-controlled content. **PR:N**: attacker needs no privileges on the target; relies on victim’s authenticated session. **UI:R**: victim interaction required. **S:U**: impact contained to the vulnerable component/user context as observed. **C/I/A:H**: confirmed unauthorized password change enables account takeover potential (confidentiality, integrity, and availability of the account). |
| **Medium — Direct cross-site CSRF (Referer validation tested; outcome unverified)** | `Not observed / Needs verification` | **N/A** | **N/A** | Referer validation was tested in Burp Repeater, but specific behaviors (e.g., missing/empty/foreign Referer blocked or allowed) were not provided. A defensible CVSS score for direct cross-site CSRF at Medium requires verified request/response evidence. |
| **Medium — Chained exploitation (Stored XSS → CSRF action, confirmed outcome)** | `CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:U/C:H/I:H/A:H` | **8.8** | **High** | The observed end-to-end impact (unauthorized password change / `Password Changed.`) is high. **Note:** this scenario depends on chaining with Stored XSS (`/vulnerabilities/xss_s/`) to achieve same-origin execution; it does not by itself prove direct external cross-site CSRF at Medium. The vector reflects the confirmed impact path from the provided observations, not a standalone bypass of a robust anti-CSRF token mechanism. |

**Scoring notes:**
- If direct external-origin CSRF at Medium is verified as blocked by Referer validation, the direct CSRF risk for Medium would be reduced; however, the chained Stored XSS path remains a critical application risk and should be remediated at the XSS source and by implementing token-based CSRF defenses.
- Exact exploit preconditions (e.g., attacker ability to store XSS payload, victim browsing to stored content) beyond those stated are **Not observed / Needs verification**.

---

## 6. Remediation Notes

### 6.1 Primary CSRF Defenses (Code/Architecture)

1. **Implement anti-CSRF tokens (synchronizer token pattern or equivalent):**
   - Generate a cryptographically strong, unpredictable token per session (or per request/form where appropriate).
   - Embed the token in the password change form/request as a hidden field or custom header.
   - Validate token server-side on every state-changing request; reject missing/invalid/mismatched tokens.
   - Ensure tokens are tied to the user session and rotated appropriately (e.g., after login and/or sensitive actions).

2. **Use safe HTTP methods and strict request handling:**
   - Require `POST` (or another non-safe method) for password changes; do not perform state changes via `GET`.
   - Enforce server-side validation of expected method and content type; reject unexpected methods/parameters.

3. **Re-authentication and step-up verification for sensitive actions:**
   - Require the current password (and/or MFA, if applicable) before allowing password changes.
   - This limits impact even if a CSRF or session-riding issue exists.

4. **Server-side validation of password change workflow:**
   - Validate `password_new` and `password_conf` equality, strength policy, and authorization context server-side.
   - Return generic error messages that do not leak sensitive state; ensure consistent handling.

### 6.2 Defense-in-Depth Measures

1. **Cookie security attributes:**
   - Set session cookies with `Secure`, `HttpOnly`, and an appropriate `SameSite` policy (`Lax` or `Strict` where compatible). Note: `SameSite` is defense-in-depth and not a replacement for anti-CSRF tokens.

2. **Origin/Referer validation as supplemental control:**
   - Validate `Origin` (preferred) and/or `Referer` headers for state-changing requests as an additional layer.
   - Handle missing headers according to a secure policy; do not rely solely on Referer checks (observed Medium behavior and chaining demonstrate limitations).

3. **Eliminate XSS to prevent chaining/bypass paths:**
   - Remediate Stored XSS at `/vulnerabilities/xss_s/` with context-aware output encoding, input validation/sanitization where appropriate, and secure templating.
   - Implement a Content Security Policy (CSP) to reduce impact of injection flaws (defense-in-depth; not a substitute for proper encoding).

4. **Logging, monitoring, and alerting:**
   - Log password change events (timestamp, account identifier, source IP, user agent, success/failure) without logging secrets.
   - Alert on anomalous password change patterns (e.g., unusual frequency, missing expected headers/tokens in rejected requests).

5. **Verification and regression testing:**
   - Retest Low and Medium scenarios after remediation: confirm anti-CSRF token enforcement, rejection of forged cross-site requests, and inability to trigger password changes via XSS-chained requests.
   - Include automated tests for CSRF token presence/validation and security regression tests for stored XSS.

### 6.3 DVWA-Specific Remediation Goal (Lab Context)
- Move the CSRF module to a secure implementation pattern: require `POST`, enforce a per-session anti-CSRF token, validate `Origin`/`Referer` as supplemental checks, and require current-password verification for password changes. Ensure Stored XSS modules are also remediated to prevent same-origin chaining.
