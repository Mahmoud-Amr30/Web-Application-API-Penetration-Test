# DVWA Authentication Brute-Force Assessment

## Penetration Testing and Defensive Analysis Report

| Report Attribute          | Detail                                                            |
| ------------------------- | ----------------------------------------------------------------- |
| **Report title**          | DVWA Authentication Brute-Force Assessment                        |
| **Assessment type**       | Web application authentication security assessment                |
| **Target**                | Damn Vulnerable Web Application (DVWA), `/vulnerabilities/brute/` |
| **Environment**           | Isolated local Docker lab at `127.0.0.1:8080`                     |
| **Intercepting proxy**    | Burp Suite at `127.0.0.1:6060`                                    |
| **Tested security level** | DVWA Security Level `Low`; code review of `Medium` and `High`     |
| **Primary tools**         | Burp Suite Repeater and Burp Suite Intruder                       |
| **Wordlist size**         | 51 candidate passwords                                            |
| **Report date**           | November 15, 2026                                                 |
| **Classification**        | Lab exercise / authorized security testing                        |

---

## 1. Executive Summary & Legal/Ethical Framework

### 1.1 Executive Summary

This assessment evaluated the authentication security of the DVWA brute-force login module at `/vulnerabilities/brute/`, hosted locally in Docker at `127.0.0.1:8080`. The objective was to determine whether the login mechanism could be defeated through systematic password guessing, to document the observable differences between failed and successful authentication attempts, and to evaluate the defensive characteristics of DVWA's `Low`, `Medium`, and `High` security levels.

The assessment confirmed that the `Low` security level is highly susceptible to a dictionary-based brute-force attack. The login form submits credentials using the HTTP `GET` method, with the username and password exposed as query-string parameters. Both failed and successful authentication attempts return HTTP `200 OK`, so the HTTP status code alone cannot distinguish success from failure. Instead, the application returns different response bodies and measurable `Content-Length` differences:

- **Failed login:** approximately **4702–4703 bytes**, containing the message `Username and/or password incorrect.`
- **Successful login:** **4741 bytes**, containing `Welcome to the password protected area admin` and the user's avatar.

Using Burp Suite Intruder in **Sniper** mode against only the `password` parameter, with the username fixed to `admin` and a 51-password wordlist, the valid credential was identified by sorting results on **Response Length**. The successful request returned **4741 bytes** and revealed the password **`password`**, which is also a DVWA default credential.

The `Medium` security level introduces basic SQL-injection-oriented input escaping through `mysqli_real_escape_string` and an artificial `sleep(2)` delay on failed authentication. These measures make single-threaded, serial guessing slower, but they do not constitute effective brute-force protection. Because the delay is stateless and applied per request, concurrent or multi-threaded guessing can bypass the intended bottleneck. The `High` security level adds an anti-CSRF `user_token` and a random delay, raising the cost of naive automation but still not providing a complete defense without stateful rate limiting, lockout, MFA, and monitoring.

**Overall risk rating: High** within the intentionally vulnerable lab context. In a production system, the combination of default credentials, credentials in the URL, a response-length oracle, and the absence of effective throttling would be **Critical**.

### 1.2 Scope

**In scope:**

- Authentication behavior of the DVWA login module at `/vulnerabilities/brute/`.
- DVWA instance hosted locally through Docker at `127.0.0.1:8080`.
- Burp Suite interception through the proxy listener at `127.0.0.1:6060`.
- DVWA Security Level `Low` for active testing.
- Source-level defensive review of the `Medium` and `High` implementations of `login.php`.
- A 51-entry password wordlist containing common defaults, test passwords, and DVWA account defaults.
- Manual request analysis in Burp Suite Repeater and automated candidate testing in Burp Suite Intruder.

**Out of scope:**

- Other DVWA vulnerability modules, including SQL injection, command execution, file inclusion, and cross-site scripting.
- The host operating system, Docker daemon, container runtime, and network infrastructure.
- External or internet-facing systems.
- Credential theft, password reuse against real accounts, persistence, lateral movement, or data exfiltration.
- Denial-of-service testing and high-volume stress testing.
- Security assessment of the underlying database or web server configuration beyond what was necessary to understand the authentication behavior.

**Assumptions:**

- The tester had explicit authorization to test the local DVWA instance.
- The Docker environment was isolated from production networks and contained only synthetic lab data.
- The `admin` account and DVWA default users were used solely for lab validation.
- Response lengths are specific to this DVWA build, theme, PHP version, and container configuration; exact byte values may differ across versions.

### 1.3 Legal and Ethical Framework

This assessment was performed under the following principles:

- **Written authorization is mandatory.** Penetration testing must be supported by a signed scope, rules of engagement, testing window, and emergency contact. Authorization for one application or environment does not extend to another.
- **Testing must remain within the isolated local lab.** The target was the loopback address `127.0.0.1:8080` inside a controlled Docker environment. No external host, third-party service, or live production system was targeted.
- **Unauthorized testing is prohibited.** Attempting credential guessing against systems without explicit permission may violate computer-misuse laws, privacy regulations, contractual obligations, and organizational policy.
- **Minimum necessary impact.** Testing was limited to low-volume authentication requests and did not attempt service disruption, data modification, or persistence.
- **Synthetic data only.** The credentials and wordlist used in this exercise are common lab values and DVWA defaults. No real user credentials or breach data were processed.
- **Responsible handling of results.** Authentication evidence, including candidate passwords and response behavior, should be treated as sensitive assessment data and shared only with authorized stakeholders.
- **No legal advice.** Applicable law depends on jurisdiction and organizational context. Legal counsel should review any testing program before execution.

---

## 2. Theoretical Foundation: Brute Force Attacks

### 2.1 Brute-Force Attacks

A **brute-force attack** is an automated or semi-automated attempt to discover valid credentials by systematically trying candidate usernames, passwords, tokens, or other authentication values. Online brute-force attacks target a live authentication service, while offline brute-force attacks target captured password hashes or encrypted data. This report focuses on **online authentication attacks** against a web login form.

Brute-force attacks are effective when one or more of the following conditions exist:

- Weak, default, or predictable passwords are permitted.
- The application does not enforce rate limiting, progressive delays, lockout, or CAPTCHA.
- Authentication responses reveal whether a username, password, or both were incorrect.
- Credentials are transmitted or stored in a manner that exposes them to interception or logging.
- The authentication flow can be automated without significant cost or detection.

Brute force is an umbrella concept. Dictionary attacks, credential stuffing, and password spraying are related techniques that differ in how candidate credentials are selected and distributed.

### 2.2 Dictionary Attacks

A **dictionary attack** uses a curated list of likely passwords rather than attempting every possible character combination. The list may include common passwords, default vendor credentials, language-specific words, keyboard patterns, seasonal values, and organization-specific terms.

Dictionary attacks are efficient because real users tend to choose predictable passwords. In this assessment, the 51-password list included common defaults and test values such as `password`, `123456`, `admin`, `root`, `secret`, and `pass123`, as well as DVWA account defaults including `abc123`, `charley`, and `letmein`. The successful candidate, `password`, was discovered through this dictionary-based approach.

### 2.3 Credential Stuffing

**Credential stuffing** uses previously compromised username-and-password pairs, often obtained from data breaches, and replays them against other services. It relies on password reuse: if a user reused the same credentials across multiple systems, a breach at one service may provide access to another.

Credential stuffing differs from a traditional dictionary attack in that the attacker is not guessing passwords from a generic list. Instead, the attacker is testing known credential pairs at scale. It is often distributed across many source IP addresses and user agents to avoid per-IP rate limits. Defenses include breached-password detection, MFA, device and behavioral risk analysis, and monitoring for unusual authentication patterns.

### 2.4 Password Spraying

**Password spraying** attempts a small number of common passwords against many usernames. Rather than trying many passwords for one account, the attacker tries one or a few likely passwords, such as `Password1` or `Welcome1`, across a large user population.

This technique is designed to avoid account lockout thresholds that trigger after repeated failures against a single account. It is particularly effective against organizations with predictable password policies and weak initial or seasonal passwords. Detection requires correlating failures across many accounts and observing distributed, low-and-slow patterns rather than a single high-volume account attack.

### 2.5 Comparison of Techniques

| Technique | Candidate Selection | Attack Breadth | Typical Detection Signal | Relevance to This Assessment |
|---|---|---|---|---|
| **Brute force** | Systematic guessing of credentials or keys | Can be broad or targeted | High request volume, repeated failures, unusual authentication timing | Umbrella category for the observed attack |
| **Dictionary attack** | Curated list of likely passwords | Many passwords against one or more accounts | Repeated failures against an account; common-password patterns | Directly used with the 51-password wordlist |
| **Credential stuffing** | Known username/password pairs from breaches | Many accounts, often distributed | High failure rate, diverse IPs, known breached credentials | Not performed; no breach data used |
| **Password spraying** | Few common passwords across many usernames | Many accounts, low attempts per account | Failures spread across accounts; lockout evasion | Not performed; scope was a single admin account |

---

## 3. Assessment Methodology (Manual & Automated Analysis — No Raw CLI Commands)

### 3.1 Environment and Preconditions

The DVWA container was reachable at `127.0.0.1:8080`. Burp Suite was configured with a proxy listener at `127.0.0.1:6060`, and the browser was routed through that listener so that requests to DVWA could be intercepted, inspected, and replayed.

Before active testing, the DVWA Security page was set to **Low**. This setting was preserved for the Repeater and Intruder phases so that the observed behavior corresponded to the least restrictive security level. The assessment used the `admin` account and a 51-password wordlist. The wordlist was plain text with one candidate per line, no comments, and no blank entries.

The testing was deliberately low-volume. The goal was to validate the authentication weakness and identify the valid password, not to stress the application or cause a denial of service.

### 3.2 Phase 1: Burp Suite Repeater — Manual Request Analysis

#### 3.2.1 Interception and Request Inspection

A login attempt was submitted through the DVWA brute-force page while Burp Suite was intercepting traffic. The request was captured and sent to **Repeater** for controlled manipulation and comparison.

The first significant finding was the **HTTP method**. The login form did not submit credentials in a `POST` request body. Instead, it used an HTTP **`GET`** request to `/vulnerabilities/brute/`, with the credentials included in the query string. The relevant parameters were:

- `username`
- `password`
- `Login`

The request also carried the DVWA session cookie and the current security-level cookie. Because the credentials were placed in the URL, they would be exposed to browser history, proxy history, web-server access logs, reverse-proxy logs, referrer headers in some configurations, and any monitoring system that records full request URLs.

This is a design weakness independently of brute-force resistance. Authentication is a state-changing operation and should use `POST` over HTTPS, with credentials in the request body and appropriate cache-control protections. A `GET` request is expected to be safe and idempotent; using it for authentication violates that expectation and increases credential exposure.

#### 3.2.2 Baseline Failed and Successful Requests

Two controlled requests were compared in Repeater:

1. A failed login using `admin` with an incorrect password.
2. A successful login using `admin` with the valid lab password.

Both responses returned **HTTP `200 OK`**. This is a critical observation: the application does not use HTTP status codes to distinguish authentication success from failure. A tool or analyst that relies only on the status code would incorrectly treat every attempt as equivalent.

The distinguishing evidence was found in the response body and its length:

| Attempt | Username | Password | HTTP Status | Response Length | Response Body Indicator | Interpretation |
|---|---|---|---|---|---:|---|---|
| 1 | `admin` | Incorrect test value | `200 OK` | **4702–4703 bytes** | `Username and/or password incorrect.` | Authentication failed |
| 2 | `admin` | `password` | `200 OK` | **4741 bytes** | `Welcome to the password protected area admin` and avatar | Authentication succeeded |

The failed responses varied by one byte between observations (**4702–4703 bytes**). This small variation may result from dynamic page rendering, reflected input length, encoding differences, or other request-specific content. The important point is that the successful response was consistently longer at **4741 bytes** and contained an explicit welcome message and avatar. Therefore, response length is a useful triage signal in this lab, but **body content inspection is the authoritative confirmation**.

#### 3.2.3 Why the HTTP 200 Response Matters

Returning `200 OK` for both success and failure is common in web applications, but it creates an **authentication oracle** when the response body or length differs. An attacker can automate requests and classify outcomes using:

- Response length.
- Presence or absence of error text.
- Presence of a welcome message or account-specific content.
- Redirect behavior.
- Response timing.
- Cookie or session changes.

In this case, the response-length delta of roughly 39–40 bytes between failure and success was sufficient for Intruder to rank candidates. The explicit textual difference made confirmation straightforward. From a defensive perspective, authentication responses should be intentionally uniform, and side channels such as length and timing should be minimized where feasible.

### 3.3 Phase 2: Burp Suite Intruder — Automated Candidate Testing

#### 3.3.1 Attack Configuration

The validated request was sent from Repeater to **Intruder**. The attack was configured as follows:

- **Attack type:** Sniper.
- **Target parameter:** `password` only.
- **Fixed username:** `admin`.
- **Fixed login parameter:** `Login`.
- **Payload source:** the 51-password wordlist.
- **Payload processing:** none required for the baseline test.
- **Concurrency:** single-threaded for the initial Low-level comparison to preserve predictable ordering and simplify response-length analysis.
- **Result analysis:** responses sorted by **Response Length**.

Only the password value was marked as a payload position. The username was intentionally held constant so that the attack answered a focused question: given the known account name `admin`, which password in the candidate list is valid? This is a dictionary attack against one account, not a password-spraying campaign.

#### 3.3.2 Results and Validation

After the attack completed, the results were sorted by response length. Most candidates returned the failed-login range of approximately **4702–4703 bytes**. One result returned **4741 bytes**, and inspection of its response body showed the successful-login welcome message and avatar.

| Rank | Password Candidate | Response Length | Response Indicator | Result |
|---:|---|---:|---|---|
| 1 | `password` | **4741 bytes** | `Welcome to the password protected area admin`; avatar present | **Valid credential** |
| — | All other candidates | **4702–4703 bytes** | `Username and/or password incorrect.` | Failed |

The valid password was **`password`**, which is also a DVWA default credential for the `admin` account. The finding was confirmed by replaying the request in Repeater and observing the same successful response. No account lockout, CAPTCHA challenge, MFA prompt, or meaningful rate-limit response was encountered at the `Low` security level.

#### 3.3.3 Interpretation

The attack succeeded because multiple weaknesses combined:

1. **Default credentials were present.** The password `password` was valid for `admin`.
2. **The application exposed a reliable response oracle.** Success and failure produced distinguishable bodies and lengths.
3. **No effective throttling existed at `Low`.** The application did not enforce account lockout, progressive delay, CAPTCHA, or rate limiting.
4. **The authentication request was trivially automatable.** Credentials were accepted as simple query parameters in a `GET` request.
5. **The status code was not a barrier.** Every attempt returned `200 OK`, which did not prevent analysis by response content.

The success is not attributable to Burp Suite itself. Burp Suite merely automated and organized requests that the application was willing to process. The root causes are insecure authentication design, default credentials, and the absence of layered anti-automation controls.

---

## 4. Defensive Code & Security Level Analysis

### 4.1 DVWA `Medium` Implementation

DVWA's `Medium` security level modifies the authentication code in `login.php` in two notable ways:

1. **Basic input escaping:** The username and password inputs are passed through `mysqli_real_escape_string` before being incorporated into the database query.
2. **Artificial delay on failure:** When authentication fails, the code calls `sleep(2)` before returning the error response.

The escaping is primarily relevant to SQL injection. It reduces the likelihood that a quote character or other SQL metacharacter will break out of the query and alter its logic. It does **not** prevent an attacker from submitting valid password candidates, and it does not address credential guessing at all. From a brute-force perspective, the query remains a normal authentication check: a candidate is either valid or invalid.

The `sleep(2)` delay is intended to slow repeated failures. Against a purely serial, single-threaded Intruder attack, it has a noticeable effect. Each failed request occupies a request/worker for approximately two seconds before the error page is returned. For a candidate near the end of a 51-entry list, the cumulative delay alone could approach 100 seconds, before accounting for network, application, and database overhead. This makes manual iteration tedious and reduces the throughput of simple automation.

However, the delay is **not a stateful rate-limiting control**. It is applied independently to each request, with no shared counter, account-level failure history, IP reputation, or adaptive backoff. An attacker can issue multiple authentication requests concurrently. Each request enters its own `sleep(2)` branch, so the total number of attempts per unit time increases with the number of concurrent workers, subject only to the server's capacity and connection limits. In practical terms, the delay converts a fast serial attack into a slower parallel attack; it does not stop the attack.

The design also has operational drawbacks:

- **Worker exhaustion:** Sleeping requests can tie up PHP workers, application threads, or database connections, creating a denial-of-service risk under sustained load.
- **Timing oracle:** Because failures sleep and successes may not, response timing can potentially reveal useful information even when response bodies are similar.
- **No account lockout:** Repeated failures do not lock or suspend the account.
- **No CAPTCHA or MFA:** There is no challenge that requires human interaction or a second factor.
- **No centralized detection:** Failures are not correlated across requests, accounts, or source addresses.

Therefore, `Medium` is harder than `Low` against a naive single-threaded attack, but it remains vulnerable to multi-threaded concurrency, distributed guessing, and low-and-slow campaigns.

### 4.2 DVWA `High` Implementation

The `High` security level raises the automation cost further by introducing an anti-CSRF **`user_token`** and a **random sleep** on failed authentication. The token is session-associated and must be obtained from the page and submitted with the login request. The random delay, often implemented with a randomized sleep interval, adds timing jitter and makes simple fixed-interval automation less effective.

These controls are meaningful speed bumps:

- A token-unaware script cannot simply replay a static request; it must fetch, parse, and refresh the token.
- Random delays complicate timing-based inference and simple throughput estimates.
- The combination increases development effort and reduces the reliability of unsophisticated tools.

They are not complete defenses:

- A session-aware automation script can retrieve and submit a fresh token for each attempt.
- CSRF tokens protect against cross-site request forgery; they are not a substitute for authentication rate limiting.
- Random sleep is still stateless and per request; concurrency can distribute the delay across many workers.
- There is no account lockout, risk-based challenge, MFA requirement, or centralized SIEM correlation.
- The application still returns distinguishable success and failure content.

Consequently, `High` is best understood as a relative hardening level for a deliberately vulnerable training application, not as a production-grade authentication design.

### 4.3 Security Level Comparison

| Security Level | HTTP Method | SQLi Protection | Brute-Force Mitigation | Automation Resistance |
|---|---|---|---|---|
| **Low** | `GET` with query-string credentials | None | None | **Very low.** A 51-password dictionary attack identified the valid password; no lockout, delay, CAPTCHA, or MFA. |
| **Medium** | `GET` with query-string credentials | Basic input escaping with `mysqli_real_escape_string` | Fixed `sleep(2)` on failed authentication | **Low to moderate.** Slows serial, single-threaded attacks; bypassable with concurrent requests and does not prevent distributed guessing. |
| **High** | `GET` with query-string credentials plus anti-CSRF `user_token` | Input escaping with `stripslashes` and `mysqli_real_escape_string` | Anti-CSRF `user_token` plus random sleep on failure | **Moderate.** Token handling and timing jitter raise the cost of naive automation, but session-aware concurrency can still automate guesses; no lockout or MFA. |

### 4.4 Consolidated Findings

| Finding ID | Finding | Qualitative Severity | Evidence |
|---|---|---|---|
| **BF-01** | Default credential accepted for `admin` | **High** in lab; **Critical** in production | Password `password` returned the successful login page at 4741 bytes. |
| **BF-02** | Credentials transmitted in HTTP `GET` query parameters | **High** | `username`, `password`, and `Login` appeared in the URL at Security Level `Low`; credentials are exposed to logs and history. |
| **BF-03** | Authentication outcome distinguishable by response length | **Medium** | Failed responses were 4702–4703 bytes; successful response was 4741 bytes. |
| **BF-04** | No effective anti-automation controls at `Low` | **High** | No lockout, CAPTCHA, MFA, or throttling encountered during the 51-password Intruder run. |
| **BF-05** | Fixed delay at `Medium` is stateless and concurrency-bypassable | **Medium** | `sleep(2)` slows serial attempts but does not coordinate state across requests. |
| **BF-06** | Token and random delay at `High` are not complete brute-force controls | **Medium** | Session-aware automation can refresh `user_token`; no lockout or risk-based challenge. |
| **BF-07** | HTTP `200 OK` used for both success and failure | **Low to Medium** | Status code alone cannot classify authentication outcome; body and length must be inspected. |

---

## 5. Recommended Defensive Controls (Remediation)

The following controls should be implemented as a layered defense. No single control is sufficient. Rate limiting, lockout, MFA, and monitoring address different parts of the attack lifecycle and should be deployed together.

### 5.1 Enforce `POST` over HTTPS

Authentication requests should use HTTP **`POST`** with credentials in the request body, never in the URL. The endpoint must be served exclusively over **HTTPS** with modern TLS configuration.

Recommended implementation points:

- Redirect all HTTP requests to HTTPS and enable HSTS after validating the deployment.
- Set authentication cookies with `Secure`, `HttpOnly`, and an appropriate `SameSite` policy.
- Apply `Cache-Control: no-store` and related headers to authentication responses.
- Avoid placing usernames, passwords, tokens, or session identifiers in query strings.
- Strip sensitive parameters from reverse-proxy and application logs.
- Reject or ignore credentials supplied through query parameters if backward compatibility is not absolutely required.

`POST` over HTTPS does not stop brute force by itself, but it removes unnecessary credential exposure and ensures that authentication traffic is confidential in transit.

### 5.2 Stateful Dynamic Rate Limiting

Implement rate limiting that maintains shared state across requests and considers multiple dimensions:

- Per account.
- Per source IP address.
- Per session or device fingerprint.
- Per ASN or geographic region where appropriate.
- Global application thresholds to detect broad campaigns.

A production-grade design should use a fast shared store such as Redis or an equivalent state service, with a sliding-window or token-bucket algorithm. When a threshold is exceeded:

- Return **HTTP `429 Too Many Requests`**.
- Include a `Retry-After` header.
- Apply **exponential backoff** with jitter rather than a fixed delay.
- Increase friction progressively: short delay, then CAPTCHA, then temporary lockout, then alerting.
- Avoid revealing whether the username exists.
- Protect against attacker-induced lockout of legitimate users by combining account, IP, and risk signals rather than relying on a single counter.

Fixed `sleep()` calls are not a substitute for stateful rate limiting. They consume server workers, create denial-of-service risk, and can be parallelized by an attacker.

### 5.3 Account Lockout and Progressive Authentication Controls

Deploy account lockout or progressive throttling with carefully chosen thresholds:

- Count failures per account and per source, with a rolling time window.
- Apply temporary lockouts after a defined number of failures.
- Use increasing lockout durations for repeated violations.
- Notify the account owner and security team of suspicious activity.
- Provide a secure, audited unlock or recovery process.
- Avoid permanent lockout that could be abused for denial of service.
- Apply stricter thresholds to privileged accounts such as `admin`.

Lockout must be paired with monitoring and user notification. A silent lockout can disrupt business operations, while an unmonitored lockout may simply push an attacker to a different account or source.

### 5.4 Adaptive CAPTCHA

Introduce CAPTCHA only when risk warrants it, rather than on every login. Adaptive challenges preserve usability while raising the cost of automation.

Recommended behavior:

- Trigger a challenge after a small number of failures from the same account, IP, or device.
- Use risk signals such as velocity, reputation, impossible travel, and known automation indicators.
- Prefer privacy-respecting challenge providers and provide an accessible alternative.
- Do not treat CAPTCHA as a primary control; determined attackers can use human-solving services.
- Combine CAPTCHA with rate limiting and MFA rather than deploying it in isolation.

### 5.5 Multi-Factor Authentication

Require **MFA** for all administrative accounts and offer it for all users. Prefer phishing-resistant methods such as **WebAuthn/FIDO2** security keys or platform authenticators. TOTP or push-based verification can be used where appropriate, with attention to MFA fatigue and push-bombing risks.

MFA does not prevent password guessing, but it prevents a guessed password from becoming a complete account compromise. It should be combined with:

- Step-up authentication for sensitive actions.
- Recovery-code controls and secure enrollment.
- Monitoring for MFA enrollment changes and unusual authentication contexts.
- User education about MFA fatigue and suspicious prompts.

### 5.6 Centralized Security Logging and SIEM Alerting

Authentication events should be logged centrally in a structured, tamper-resistant system and correlated in a SIEM or equivalent analytics platform.

At minimum, capture:

- Timestamp and event correlation ID.
- Username or account identifier, with appropriate privacy handling.
- Authentication outcome.
- Source IP address, user agent, and relevant device or session metadata.
- Request method, endpoint, and response status.
- Rate-limit, lockout, CAPTCHA, and MFA events.
- Geographic or ASN enrichment where lawful and appropriate.

Do **not** log plaintext passwords, session tokens, or other secrets.

Alerting should detect:

- A spike in failures for a single account.
- Failures spread across many accounts, consistent with password spraying.
- Many accounts failing from a small set of IPs, consistent with credential stuffing.
- Low-and-slow failures across distributed sources.
- Successful authentication after a burst of failures.
- Login attempts against privileged accounts outside normal patterns.
- Changes to MFA enrollment, password reset, or account recovery settings.

Logs should be retained according to policy, protected from unauthorized modification, and integrated into incident-response playbooks. Detection without a response process provides limited value.

### 5.7 Additional Hardening Measures

The following controls are recommended as part of a defense-in-depth program:

- **Block known breached passwords** and enforce a minimum password length rather than relying on complex composition rules.
- **Store passwords with a modern adaptive hash** such as Argon2id or bcrypt; MD5 is not suitable for password storage.
- **Use generic authentication error messages** and normalize response length and timing where practical.
- **Add CSRF protection to state-changing requests**, but do not mistake it for brute-force protection.
- **Apply WAF or reverse-proxy rules** as compensating controls for known attack patterns, not as the sole defense.
- **Disable or rename default accounts** and force credential rotation during deployment.
- **Restrict administrative interfaces** by network, identity-aware proxy, or VPN where feasible.
- **Conduct regular authentication security testing** with explicit authorization and controlled volume.
- **Retest after remediation** to verify that rate limiting, lockout, MFA, and alerting function as intended.

### 5.8 Remediation Priority Matrix

| Priority | Control | Primary Threat Addressed | Expected Effect |
|---|---|---|---|
| **P1** | HTTPS and `POST`-based authentication | Credential exposure through URLs and logs | Removes cleartext credential placement in query strings |
| **P1** | Stateful rate limiting with HTTP `429` and exponential backoff | Automated guessing, dictionary attacks | Limits attempts per account, IP, and time window |
| **P1** | Account lockout and progressive throttling | Sustained password guessing | Stops repeated failures against targeted accounts |
| **P1** | MFA for privileged and all users | Credential compromise after guessing | Prevents a guessed password from granting full access |
| **P1** | Centralized logging and SIEM alerting | Undetected brute force, spraying, stuffing | Enables rapid detection and response |
| **P2** | Adaptive CAPTCHA | Automated and low-cost guessing | Adds friction when risk increases |
| **P2** | Breached-password blocking and modern password storage | Weak/reused passwords, offline cracking | Reduces credential quality and hash-cracking risk |
| **P3** | WAF and response normalization | Commodity automation, information leakage | Provides supplemental defense and reduces oracle fidelity |

---

## 6. Conclusion

The DVWA brute-force module demonstrates, in a controlled lab, how a weak authentication design can be defeated through a small dictionary of common passwords. At Security Level `Low`, the login form exposed credentials in HTTP `GET` parameters, returned HTTP `200 OK` for both success and failure, and leaked a reliable success signal through response length and body content. A 51-password Burp Suite Intruder attack using Sniper mode against the `password` parameter identified `password` for the `admin` account at a response length of **4741 bytes**, compared with **4702–4703 bytes** for failed attempts.

The `Medium` level improves resistance to naive serial automation by adding input escaping and a two-second failure delay, but the delay is stateless and can be bypassed with concurrent requests. The `High` level adds an anti-CSRF token and random delay, which raise the cost of automation but do not replace stateful rate limiting, lockout, MFA, or monitoring.

The recommended remediation is a layered authentication security program: HTTPS-only `POST` authentication, shared-state dynamic rate limiting with HTTP `429` and exponential backoff, progressive account lockout, adaptive CAPTCHA, phishing-resistant MFA, and centralized SIEM detection. These controls should be validated through authorized retesting and supported by clear legal authorization, scoped rules of engagement, and an accountable incident-response process.

---

## References and Standards

- OWASP Application Security Verification Standard (ASVS), authentication and anti-automation requirements.
- OWASP Authentication Cheat Sheet and Credential Stuffing Prevention Cheat Sheet.
- NIST Special Publication 800-63B, Digital Identity Guidelines: Authentication and Lifecycle Management.
- MITRE ATT&CK technique **T1110**, Brute Force, including password guessing, credential stuffing, and password spraying sub-techniques.
- DVWA source code for the `Low`, `Medium`, and `High` brute-force challenge implementations.
