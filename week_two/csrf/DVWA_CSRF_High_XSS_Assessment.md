# Cross-Site Request Forgery (CSRF) Assessment Chained with Cross-Site Scripting (XSS)

**Target Application:** Damn Vulnerable Web Application (DVWA)\
**Assessment Type:** Local Security Lab / Vulnerability Analysis\
**Report Date:** 2026-11-23\
**Classification:** Internal --- Technical Findings

------------------------------------------------------------------------

## 1. Executive Summary & Environment

### 1.1 Objective

The objective of this assessment was to analyze the Cross-Site Request
Forgery (CSRF) protection mechanisms in DVWA across different security
levels, examine the lifecycle of the `user_token` anti-CSRF parameter
across Request/Response cycles, and document how a same-origin
Cross-Site Scripting (XSS) flaw undermines token-based defenses.

### 1.2 Environment

  Item                        Detail
  --------------------------- ----------------------------------------
  Target application          DVWA (Damn Vulnerable Web Application)
  Deployment                  Docker container
  Target URL                  `http://127.0.0.1:8080`
  Target port                 `8080`
  Intercepting proxy          Burp Suite
  Proxy listener              Port `6060`
  Test account                `admin`
  Initial password            `password`
  Password set during test    `Ehab`
  Session cookie (captured)   `PHPSESSID=[REDACTED]`

### 1.3 Summary of Findings

-   **Request Method & Parameters:** In the tested DVWA build, the
    password-change functionality (`/vulnerabilities/csrf/`) processes
    state-changing requests via HTTP `GET` query parameters
    (`password_new`, `password_conf`, `Change`, and `user_token`).
-   **Token Lifecycle (High Security):** DVWA generates a per-session
    anti-CSRF token embedded in the form. The observed token changed
    after a successful request. Replaying a previously consumed token
    should be verified experimentally before describing the behavior as
    single-use.
-   **Same-Origin Weakness (XSS Chaining):** Anti-CSRF tokens rely in
    part on the Same-Origin Policy (SOP) to prevent scripts on external
    origins from reading them. When an XSS vulnerability exists on the
    same origin (`/vulnerabilities/xss_s/` or
    `/vulnerabilities/xss_r/`), script execution takes place in the
    victim's authenticated browser context and may access same-origin
    pages and tokens.

------------------------------------------------------------------------

## 2. Technical Observations & Request/Response Analysis

### 2.1 Request vs. Response Token Comparison (Burp Repeater)

Using Burp Suite Repeater (configured on port `6060`), the
password-change `GET` request was inspected and compared against the
returned HTML response.

**Observed Request Token (Query String):**

``` http
GET /vulnerabilities/csrf/?password_new=Ehab&password_conf=Ehab&Change=Change&user_token=25fecd7365c8430c683a86de2006f8fb HTTP/1.1
Host: 127.0.0.1:8080
Cookie: PHPSESSID=[REDACTED]; security=high
```

**Observed Next-Form Response Token (Hidden Input):**

``` html
<input type='hidden' name='user_token' value='60366632412174e659a57551ba0f1545a' />
<pre>Password Changed.</pre>
```

**Interpretation:**

-   The server accepted the incoming request token
    (`25fecd7365c8430c683a86de2006f8fb`), processed the password change,
    and returned a newly rendered form containing a different token
    (`60366632412174e659a57551ba0f1545a`).
-   The observed token change indicates token renewal between the
    inspected request and subsequent form render. To establish
    single-use enforcement conclusively, replay the original request
    with the old token and record the server's response.
-   Session identifiers are redacted in this report. Preserve the
    original values only in secured lab evidence if required.

### 2.2 Observed Execution Flow via XSS Context

During testing, an authenticated request associated with the Stored XSS
page (`Referer: http://127.0.0.1:8080/vulnerabilities/xss_s/`) triggered
an HTTP `GET` request to the CSRF endpoint:

``` http
GET /vulnerabilities/csrf/?password_new=Ehab&password_conf=Ehab&Change=Change&user_token=[REDACTED] HTTP/1.1
Host: 127.0.0.1:8080
Referer: http://127.0.0.1:8080/vulnerabilities/xss_s/
Cookie: PHPSESSID=[REDACTED]; security=low
```

**Response Observed in HTTP History:**

-   **Status Code:** `200 OK`
-   **Response Body:** `<pre>Password Changed.</pre>`
-   **Result:** The password for `admin` was updated from `password` to
    `Ehab`.

### 2.3 Comparison Between `Low` and `High` Security Levels

  -----------------------------------------------------------------------
  Aspect                  `security=low`          `security=high`
  ----------------------- ----------------------- -----------------------
  HTTP Method Used        `GET` (query            `GET` (query
                          parameters)             parameters)

  Anti-CSRF Token         Not enforced in the     Token validation
  Validation              observed test           enabled

  Token Renewal           No effective token      A different token
                          protection observed     appeared in the next
                                                  form response

  Static Request Replay   Can succeed without a   Expected to fail if the
                          valid token             old token is rejected;
                                                  verify with a replay
                                                  test

  Impact of Same-Origin   Can trigger             Can potentially read
  XSS                     authenticated           same-origin content and
                          state-changing requests obtain a current token
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 3. Environment Setup & Verification Steps

### 3.1 Container Initialization

A clean instance of DVWA was deployed locally on port `8080`:

``` bash
docker rm -f dvwa
docker run -d --name dvwa -p 8080:80 vulnerables/web-dvwa
```

### 3.2 Database & Proxy Setup

1.  Initialized the database via `http://127.0.0.1:8080/setup.php` using
    **Create / Reset Database**.
2.  Authenticated using the default credentials (`admin` / `password`).
3.  Configured the Burp Suite Proxy listener on `127.0.0.1:6060` and
    added `http://127.0.0.1:8080` to **Target \> Scope**.
4.  Inspected requests and responses through **Proxy \> HTTP history**
    and **Repeater** to verify session cookies (`PHPSESSID`,
    `security`), `Referer` headers, and `user_token` values.

### 3.3 Architectural Analysis of the Chained Mechanism

1.  **Trigger Mechanism:** An event-driven HTML element in the Stored
    XSS context can initiate an asynchronous `GET` request to
    `/vulnerabilities/csrf/` using the victim's active browser session.
2.  **Same-Origin Privilege:** Because the request targets
    `http://127.0.0.1:8080`, the browser attaches applicable cookies for
    that origin, subject to cookie attributes and browser policy.
3.  **Token Handling Consideration:** A hardcoded token works only when
    token validation is disabled or when that token remains valid for
    the active session. At a security level where token validation is
    enforced, same-origin script execution may be able to request the
    CSRF page, read its current `user_token` from the returned HTML, and
    use it in a subsequent authenticated request. This demonstrates why
    anti-CSRF tokens do not compensate for an XSS vulnerability on the
    same origin.

------------------------------------------------------------------------

## 4. Impact & Remediation

### 4.1 Security Impact

-   **Unauthorized Account Changes:** An attacker who can cause an
    authenticated victim's browser to issue a state-changing request may
    change account credentials or perform other actions available to
    that user, potentially locking the legitimate user out.
-   **Bypass of Anti-CSRF Controls via XSS:** Per-request anti-CSRF
    tokens are not a reliable defense against same-origin XSS because
    injected script executes within the application's origin and may
    access same-origin resources available to the victim's session.

### 4.2 Remediation Recommendations

1.  **Enforce HTTP `POST` for State-Changing Operations**
    -   Do not perform sensitive state changes, such as password
        updates, through `GET` requests.
    -   Use appropriate request methods and server-side authorization
        and CSRF validation. Remember that changing `GET` to `POST`
        alone does not eliminate CSRF.
2.  **Require Current Password Verification**
    -   Require the user's existing password (`password_current`) before
        processing a password change and validate it server-side.
    -   Treat this as an additional security control, not as a complete
        defense against XSS. Same-origin script execution may still
        interact with the password-change workflow.
3.  **Contextual Output Encoding & Content Security Policy (CSP)**
    -   Apply context-appropriate output encoding to user-controlled
        data before rendering it in HTML.
    -   Validate and sanitize input where appropriate, while recognizing
        that output encoding is the primary defense against injection in
        a given output context.
    -   Implement a restrictive CSP, for example
        `default-src 'self'; script-src 'self'`, and avoid inline event
        handlers. A CSP is a defense-in-depth measure and should not
        replace secure output handling.
4.  **Session & Cookie Hardening**
    -   Configure session cookies with `HttpOnly`, `Secure` (when using
        HTTPS), and an appropriate `SameSite` policy such as `Lax` or
        `Strict`.
    -   Maintain server-side CSRF token validation for state-changing
        requests.
    -   Note that `SameSite` reduces some cross-site request risks but
        does not prevent same-origin XSS from issuing authenticated
        requests.

------------------------------------------------------------------------

## 6. Conclusion

The local DVWA assessment demonstrated that the tested password-change
endpoint uses HTTP `GET` for a state-changing operation and that the
observed behavior differs between Low and High security levels. The
response inspected at High security contained a token value different
from the one submitted in the request. A separate test associated with
the Stored XSS page demonstrated that same-origin script execution can
initiate an authenticated request to the CSRF endpoint.

The findings illustrate that CSRF defenses and XSS prevention must be
addressed together. Applications should use appropriate HTTP methods,
robust server-side CSRF validation, secure session-cookie settings, and
context-aware output encoding. Same-origin XSS must be treated as a
critical weakness because it can undermine token-based protections.

**Assessment Scope:** Local DVWA lab only. The observations and
conclusions apply to the tested build and configuration and should not
be generalized to other applications without verification.
