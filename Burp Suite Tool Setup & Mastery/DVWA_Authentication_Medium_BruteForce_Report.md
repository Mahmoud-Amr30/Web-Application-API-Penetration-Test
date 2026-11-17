# DVWA Authentication Brute-Force Assessment — Medium Security

## Technical Penetration Testing and Defensive Analysis Report

| Report Attribute | Detail |
|---|---|
| **Report title** | DVWA Authentication Brute-Force Assessment — Medium Security |
| **Assessment type** | Web application authentication security assessment |
| **Target application** | Damn Vulnerable Web Application (DVWA) |
| **Target module** | `/vulnerabilities/brute/` |
| **Target address** | Local Docker instance at `127.0.0.1:8080` |
| **Tested security level** | `Medium` |
| **Primary tooling** | Burp Suite Intruder and Burp Suite Repeater |
| **Attack type** | Cluster Bomb |
| **Payload configuration** | Runtime file payloads for usernames and passwords |
| **Username wordlist** | `http_default_users.txt` |
| **Password wordlist** | `http_default_pass.txt` |
| **Successful request** | Request **#15** |
| **Valid credentials** | `admin` / `password` |
| **Report date** | November 15, 2026 |
| **Classification** | Isolated lab exercise / authorized security testing |

---

## 1. Executive Summary & Assessment Scope

### 1.1 Executive Summary

This assessment evaluated the authentication security of the DVWA brute-force module at `/vulnerabilities/brute/` while DVWA was configured at **Security Level `Medium`**. The target was a locally hosted Docker instance at `127.0.0.1:8080`. The objective was to determine whether the security enhancements introduced at the `Medium` level meaningfully prevented credential guessing, and to analyze the defensive value of the corresponding PHP source-code changes.

A Burp Suite Intruder **Cluster Bomb** attack was performed using two independent runtime-file payload sets: `http_default_users.txt` for usernames and `http_default_pass.txt` for passwords. This configuration tested combinations of candidate usernames and passwords rather than limiting the assessment to a single known account. The server consistently returned **HTTP `200 OK`** for both failed and successful authentication attempts, so HTTP status codes alone could not distinguish the outcome.

Failed attempts produced a baseline response length of **4711–4712 bytes**, while the successful request produced **4749 bytes**. Request **#15** returned the successful response and used the credentials **`admin` / `password`**, confirming that the `Medium` security level did not prevent brute-force compromise. The successful response contained the authenticated welcome content and avatar, while failed responses contained the generic authentication error message.

The `Medium` implementation improves on `Low` by adding input escaping through `stripslashes` and `mysqli_real_escape_string`, removing explicit database error output, and adding a mandatory `sleep(2)` delay in the failure branch. These changes reduce SQL-injection exposure and make a purely serial attack slower. They do not, however, implement stateful rate limiting. The delay is applied independently to each request, so a multi-threaded or otherwise concurrent attack can distribute attempts across many execution threads and continue at a rate limited primarily by server capacity.

**Overall finding:** The `Medium` security level raises the cost of simple, single-threaded automation but remains vulnerable to dictionary-based credential guessing when the attacker can issue concurrent requests. The successful compromise of `admin` / `password` demonstrates that an artificial per-request delay is not a substitute for stateful rate limiting, account lockout, adaptive challenge mechanisms, and monitoring.

### 1.2 Assessment Scope

**In scope:**

- Authentication behavior of DVWA's `/vulnerabilities/brute/` login module.
- DVWA Security Level `Medium`.
- Local Docker deployment reachable at `127.0.0.1:8080`.
- Burp Suite Intruder Cluster Bomb attack with separate username and password runtime-file payloads.
- Differential analysis of HTTP status codes, response lengths, and response bodies.
- Source-code review of the `Low` and `Medium` implementations of the brute-force challenge.
- Defensive evaluation of the `sleep(2)` control and recommendations for stronger mitigations.

**Out of scope:**

- Other DVWA modules and vulnerabilities.
- The Docker host, container runtime, network stack, and underlying operating system.
- External, internet-facing, or production systems.
- Denial-of-service testing and high-volume stress testing.
- Exploitation of SQL injection beyond source-code analysis.
- Credential theft, password reuse against real accounts, persistence, or lateral movement.
- Assessment of database hardening, web-server configuration, or infrastructure controls beyond the authentication module.

**Assumptions and constraints:**

- The assessment was performed in an isolated local lab with explicit authorization.
- The DVWA instance contained only synthetic lab data.
- The wordlists were test-oriented and included default or commonly used lab credentials.
- Response lengths are specific to this DVWA build, theme, PHP version, Docker image, and wordlist ordering. Exact values may differ in other environments.
- The request number **#15** reflects the execution order observed in this assessment and depends on wordlist ordering and request scheduling.
- No raw shell or terminal commands were used in the preparation of this report.

### 1.3 Key Findings

- **Default credentials were accepted:** `admin` / `password` successfully authenticated at Security Level `Medium`.
- **Cluster Bomb automation succeeded:** The attack tested combinations from separate username and password wordlists rather than relying on a single account/password pair.
- **HTTP status codes were not a reliable success indicator:** Both successful and failed attempts returned HTTP `200 OK`.
- **Response length exposed the outcome:** Failed attempts clustered at **4711–4712 bytes**, while the successful attempt returned **4749 bytes**.
- **The `sleep(2)` control was not stateful:** It slowed each failed request but did not enforce a shared per-account or per-IP attempt budget.
- **Concurrency neutralized the delay:** Multi-threaded execution allowed requests to proceed in parallel, each carrying its own delay rather than sharing a global throttle.
- **Input escaping reduced SQL-injection risk but did not address brute force:** `stripslashes` and `mysqli_real_escape_string` are relevant to query safety, not credential guessing.

---

## 2. Practical Attack Methodology (Burp Suite Intruder — Cluster Bomb)

### 2.1 Environment and Preconditions

DVWA was hosted in a local Docker container and accessed at `127.0.0.1:8080`. The application was configured through the DVWA Security page to **Medium** before testing. A browser routed through Burp Suite was used to access the brute-force module, and the resulting login request was captured for analysis and automation.

The assessment deliberately used a small, controlled lab environment. The goal was to validate the authentication weakness and observe the behavior of the `Medium` controls, not to generate production-scale traffic or cause service disruption.

The captured request used the application's normal login flow. The relevant inputs were:

- `username`
- `password`
- `Login`

At this DVWA version and security level, the request continued to use HTTP `GET` with query-string parameters. The `Medium` hardening changes were implemented server-side in `login.php`; they did not change the fundamental request structure observed at `Low`.

### 2.2 Burp Suite Intruder Configuration

The request was sent from Burp Suite Repeater to **Intruder**. The attack was configured as a **Cluster Bomb**, which iterates through combinations of multiple payload sets. This differs from Sniper, which targets one position at a time. Cluster Bomb was appropriate because the assessment intentionally tested separate username and password lists.

| Configuration Item | Setting |
|---|---|
| **Attack type** | Cluster Bomb |
| **Payload position 1** | `username` query parameter |
| **Payload position 2** | `password` query parameter |
| **Payload set 1 source** | Runtime file: `http_default_users.txt` |
| **Payload set 2 source** | Runtime file: `http_default_pass.txt` |
| **Payload encoding** | Standard request encoding; no custom transformations required |
| **Concurrency** | Multi-threaded request execution |
| **Result analysis** | HTTP status, response length, response body |
| **Target security level** | `Medium` |

The use of **Runtime file** payloads allowed each list to be supplied independently at attack time. Payload set 1 varied the username, and payload set 2 varied the password. Burp Suite then generated the cross-product of the two lists: for each username in `http_default_users.txt`, it attempted the password candidates from `http_default_pass.txt`. This is materially different from fixing `admin` and guessing only passwords; it tests whether a combination of default account names and default passwords can be discovered.

The wordlists were plain text with one candidate per line and no comments or blank entries. The attack was configured to run with multiple concurrent requests. That concurrency is central to interpreting the result: the `Medium` delay is applied to each failed request, not to a shared account-level or IP-level counter.

### 2.3 Findings and Differential Analysis

The server returned **HTTP `200 OK`** for failed and successful authentication attempts. There was no redirect, no `401 Unauthorized`, no `403 Forbidden`, and no `429 Too Many Requests` response to distinguish the result. As at the `Low` level, the application encoded authentication outcome in the response body rather than in the HTTP status line.

The baseline failed response length was **4711–4712 bytes**. This narrow one-byte variation is consistent with minor dynamic page rendering or request-specific content and meant that response length alone should be interpreted with care. The failed responses contained the generic authentication error:

> `Username and/or password incorrect.`

Request **#15** returned a distinct response length of **4749 bytes** and contained the authenticated welcome content and avatar:

> `Welcome to the password protected area admin`

The successful credentials were **`admin` / `password`**.

| Result | Request Position | Username | Password | HTTP Status | Response Length | Response Body Indicator | Interpretation |
|---|---:|---|---|---|---:|---|---|
| Baseline failures | Other requests | Various | Various | `200 OK` | **4711–4712 bytes** | `Username and/or password incorrect.` | Authentication failed |
| **Successful compromise** | **#15** | **`admin`** | **`password`** | **`200 OK`** | **4749 bytes** | **Welcome message and avatar** | **Authentication succeeded** |

The successful response was approximately **37–38 bytes longer** than the failed baseline. That difference was sufficient to separate the valid request from the failed cluster when results were sorted by **Response Length**. Body inspection confirmed the finding and ruled out a coincidental length variation.

Because Cluster Bomb used two runtime wordlists, the result also demonstrates the risk of combining default account names with default passwords. The specific request number **#15** is an artifact of wordlist ordering and execution scheduling; it should not be interpreted as a universal position. The important finding is that the credential pair was present in the tested lists and was accepted by the application.

### 2.4 Attack Outcome

The attack achieved authentication compromise under `Medium` security with a modest wordlist and a standard Burp Suite workflow. The `sleep(2)` control increased the delay associated with failed requests, but it did not prevent the attack from completing. The multi-threaded configuration allowed multiple requests to be in flight, each independently entering the server-side delay, rather than forcing the entire credential search into a single serialized sequence.

This result is consistent with the source-code behavior analyzed in Section 3: the application delays individual failed requests but does not maintain shared state that would cap the number of failures per account, session, or source address.

---

## 3. PHP Source Code Analysis & Comparison (Low vs. Medium)

### 3.1 Low Security Implementation

The `Low` implementation represents the least restrictive authentication design in the exercise. Its principal weaknesses are as follows:

- **Direct input handling via `$_GET`:** The username and password are read directly from request parameters without validation, normalization, or type checking.
- **Direct SQL query concatenation:** User-controlled values are incorporated into the authentication query without parameterization or escaping. This creates a potential SQL-injection condition in addition to the credential-guessing weakness.
- **Verbose database failure behavior:** The implementation may expose database error output when a query fails, which can assist an attacker in understanding query structure and backend behavior.
- **No artificial delay on failure:** Failed authentication returns immediately. There is no throttling, progressive delay, account lockout, or challenge mechanism in the failure path.
- **No shared failure state:** The application does not count failures per account, per IP address, or per session.
- **Trivial automation:** Because the endpoint accepts ordinary requests and returns immediate results, both serial and parallel guessing are inexpensive to perform.

The `Low` design is useful for demonstrating the vulnerability, but it is not representative of a defensible authentication implementation. The absence of input sanitization is a SQL-injection concern; the absence of throttling and lockout is a brute-force concern. These are separate weaknesses that require separate controls.

### 3.2 Medium Security Implementation

The `Medium` implementation introduces several changes intended to reduce the attack surface:

1. **Input escaping:** The username and password are processed with `stripslashes` and `mysqli_real_escape_string` before being used in the authentication query.
2. **Reduced error disclosure:** Explicit error-printing behavior is removed, limiting the amount of database or query information returned to the client.
3. **Artificial failure delay:** A mandatory `sleep(2)` call is placed in the failure branch (`else`) so that an unsuccessful authentication attempt takes approximately two seconds before the error response is returned.

These changes have different security effects:

- **`stripslashes` and `mysqli_real_escape_string`** reduce the likelihood that quote characters and other SQL metacharacters will break out of the query. They improve resistance to SQL injection in this specific code path. They are not equivalent to parameterized queries and should not be treated as a complete SQL-injection defense in a production system.
- **Removal of explicit error output** reduces information leakage. An attacker gains less insight into the database structure or query failure reason.
- **`sleep(2)`** increases the wall-clock time of a failed request. Against a purely serial attack, it reduces the number of guesses per minute and makes manual enumeration slower.
- **None of these changes introduce stateful authentication controls.** There is still no account-level failure counter, no IP-based rate limit, no lockout, no CAPTCHA, and no MFA requirement.

The `Medium` code therefore addresses some SQL-injection and information-disclosure concerns while offering only a limited, per-request obstacle to brute force. The delay is not a security boundary; it is an artificial performance penalty that can be distributed across concurrent requests.

### 3.3 Why Medium Is Harder Than Low — and Why It Is Still Insufficient

`Medium` is harder than `Low` for a simple, single-threaded Intruder attack. Each failed request waits approximately two seconds. For a candidate late in a wordlist, those delays accumulate and make the attack appear slow. A casual tester or an unsophisticated script may conclude that the control is effective.

The delay does not survive concurrency. If an attacker issues multiple requests in parallel, each request executes its own failure branch independently. There is no global semaphore, shared attempt counter, or per-account queue that coordinates the delay. The effective throughput is therefore determined by the number of concurrent workers and the server's capacity, not by the two-second delay alone.

The design also has secondary costs:

- **Resource consumption:** Sleeping requests occupy application workers, threads, or database connections. A sustained attack can consume server capacity and create a denial-of-service risk.
- **Timing observations:** A delay on failure but not on success can create a timing side channel. An attacker may use response timing as an additional signal even when response bodies are similar.
- **False sense of security:** Because the control is visible in source code, it may be mistaken for rate limiting. It is not. It does not track identities, failed-attempt history, or source reputation.
- **No account protection:** The application does not suspend or lock an account after repeated failures.
- **No human challenge:** There is no CAPTCHA or step-up verification when risk increases.

The practical lesson is that **per-request delay is not the same as rate limiting**. A rate limit is a shared policy that constrains how many attempts an actor can make in a time window. A sleep call is local execution behavior that can be parallelized, distributed, or absorbed by additional workers.

---

## 4. Technical Comparison Table

| Security Level | Input Sanitization | SQLi Vulnerability | Response Throttling (`sleep(2)`) | Automation/Concurrency Resistance |
|---|---|---|---|---|
| **Low** | None. Values are read directly from `$_GET` and used without escaping. | **High.** Direct string concatenation creates a potential SQL-injection condition. | **None.** Failed authentication returns immediately with no artificial delay. | **Very low.** Serial and parallel automation can guess credentials quickly; no lockout or rate limit. |
| **Medium** | `stripslashes` and `mysqli_real_escape_string` are applied to input values. | **Reduced, but not eliminated.** Escaping helps this code path but is not equivalent to parameterized queries. | **Fixed two-second delay** in the failure (`else`) branch. | **Low.** Defeats naive serial attacks but is bypassed by multi-threaded Cluster Bomb requests; no shared per-IP or per-account state. |

---

## 5. Defensive Conclusion & Bypass Analysis

### 5.1 Why the `sleep(2)` Delay Failed

The `Medium` sleep control failed to stop the Cluster Bomb attack because it was **stateless and request-local**. The server delayed each failed request independently, but it did not establish a shared policy across requests. The following conditions allowed the attack to succeed:

- **Per-thread rather than per-actor enforcement:** The delay applied to the thread or worker handling a failed request. Multiple concurrent requests each received their own delay rather than competing for a shared attempt budget.
- **No per-IP or per-account counter:** The application did not track how many failures originated from a source address, session, or account within a time window.
- **No lockout threshold:** Repeated failures did not trigger account suspension, progressive lockout, or administrator notification.
- **No challenge escalation:** The application did not introduce CAPTCHA, MFA, or step-up verification after suspicious behavior.
- **A reliable response oracle remained:** Failed and successful attempts were distinguishable by response length and body content, allowing automated classification.
- **Default credentials remained valid:** `admin` / `password` was present in the tested runtime wordlists and was accepted by the application.
- **Concurrency absorbed the delay:** A multi-threaded attack maintained multiple in-flight requests, so the two-second delay reduced the speed of individual requests without imposing a global limit on the attack.

The attack did not “break” the `sleep(2)` function. It simply avoided the serialized conditions under which the delay would have been most effective. The application was willing to process concurrent authentication attempts, and the delay had no shared state with which to coordinate them.

### 5.2 Recommended Defensive Controls

#### Stateful Rate Limiting

Implement rate limiting as a **shared, stateful control** at the application, reverse-proxy, API-gateway, or identity-provider layer. The control should track attempts across multiple dimensions:

- Account identifier.
- Source IP address.
- Session or device fingerprint.
- ASN, network, or geographic risk signals where appropriate.
- Global thresholds for broad campaigns.

Use a sliding-window or token-bucket algorithm backed by a shared store such as Redis or an equivalent service. When a threshold is exceeded:

- Return **HTTP `429 Too Many Requests`**.
- Include a `Retry-After` header.
- Apply **exponential backoff with jitter**.
- Escalate friction progressively: delay, then CAPTCHA, then temporary lockout, then security alert.
- Avoid disclosing whether a username exists.
- Combine account, IP, and risk signals so an attacker cannot trivially lock out a legitimate user.

A stateful rate limiter should be evaluated under concurrent load, not just serial requests. The test should verify that the limit is enforced across threads, processes, and application instances.

#### Account Lockout

Deploy progressive account lockout with controlled thresholds:

- Count failures per account within a rolling window.
- Apply temporary lockout after a defined number of failures.
- Increase lockout duration for repeated violations.
- Notify the account owner and security team.
- Provide a secure recovery and unlock workflow.
- Apply stricter thresholds to privileged accounts such as `admin`.
- Avoid permanent lockout that could be abused for denial of service.

Lockout should complement, not replace, rate limiting. Rate limiting controls the rate of attempts; lockout changes the account state after sustained failure.

#### Adaptive CAPTCHA

Introduce CAPTCHA based on risk rather than on every login:

- Trigger a challenge after repeated failures from the same account, IP, or device.
- Use risk signals such as request velocity, source reputation, and automation indicators.
- Prefer privacy-respecting providers and provide an accessible alternative.
- Do not rely on CAPTCHA alone; it can be outsourced to human solvers.
- Combine CAPTCHA with rate limiting, lockout, and MFA.

#### Additional Layered Controls

- **MFA:** Require phishing-resistant MFA for administrators and offer it to all users. MFA prevents a guessed password from becoming a complete account compromise.
- **HTTPS and `POST`:** Move credentials out of the URL and into an encrypted request body. Apply HSTS and secure cookie attributes.
- **Uniform responses:** Reduce differences in response body, length, and timing between success and failure.
- **Prepared statements:** Replace string concatenation with parameterized queries. Escaping is not a complete substitute for query parameterization.
- **Modern password storage:** Use Argon2id or bcrypt rather than legacy fast hashes such as MD5.
- **Centralized logging and SIEM alerting:** Record authentication successes, failures, lockouts, rate-limit events, and CAPTCHA challenges; alert on spikes, spraying patterns, and success after repeated failure.
- **WAF or reverse-proxy rules:** Use as compensating controls for known patterns, not as the primary authentication defense.

### 5.3 Remediation Priorities

| Priority | Control | Primary Effect |
|---|---|---|
| **P1** | Stateful rate limiting with HTTP `429` and exponential backoff | Caps attempts per account, IP, and time window |
| **P1** | Progressive account lockout | Stops sustained guessing against an account |
| **P1** | MFA for administrative accounts | Prevents password-only compromise |
| **P1** | Centralized logging and SIEM alerting | Enables detection and response |
| **P2** | Adaptive CAPTCHA | Adds friction when risk increases |
| **P2** | HTTPS with `POST` authentication | Removes credentials from URLs and protects transit |
| **P2** | Parameterized SQL queries | Eliminates residual SQL-injection risk |
| **P3** | Uniform response and timing behavior | Reduces the fidelity of the authentication oracle |

### 5.4 Conclusion

The `Medium` security level improves on `Low` by adding input escaping, reducing error disclosure, and introducing a two-second failure delay. Those changes are meaningful for SQL-injection resistance and for slowing naive serial automation. They do not, however, constitute brute-force protection.

The Cluster Bomb attack succeeded because the application accepted concurrent authentication requests without shared failure state, exposed a distinguishable success response, and allowed default credentials. The valid pair `admin` / `password` was identified at Request **#15** with a response length of **4749 bytes**, compared with the failed baseline of **4711–4712 bytes**.

The appropriate conclusion is not that `sleep(2)` is useless, but that it is insufficient in isolation. Defensible authentication requires **stateful rate limiting, progressive account lockout, adaptive CAPTCHA, MFA, secure transport, and centralized monitoring**. These controls should be tested against concurrent, distributed, and low-and-slow attack patterns, not only against a single-threaded client. Within the intentionally vulnerable DVWA lab, the `Medium` level remains a useful demonstration of why per-request delays cannot substitute for shared security policy.

---

## References and Standards

- OWASP Application Security Verification Standard (ASVS), authentication and anti-automation requirements.
- OWASP Authentication Cheat Sheet and Credential Stuffing Prevention Cheat Sheet.
- NIST Special Publication 800-63B, Digital Identity Guidelines: Authentication and Lifecycle Management.
- MITRE ATT&CK technique **T1110**, Brute Force, including password guessing, credential stuffing, and password spraying.
- DVWA source code for the `Low` and `Medium` brute-force challenge implementations.
