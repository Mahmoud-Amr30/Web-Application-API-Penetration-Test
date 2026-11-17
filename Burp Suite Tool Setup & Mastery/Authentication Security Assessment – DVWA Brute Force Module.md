# Authentication Security Assessment – DVWA Brute Force Module

Nov 16, 2026 · @Mahmoud

## Executive Summary

This assessment examined the authentication and brute-force protection behavior of a local, intentionally vulnerable training application (DVWA) using Burp Suite as an intercepting proxy. Testing found that the login endpoint accepted non-standard HTTP methods (PUT, DELETE) in place of the form's intended method, while session token substitution was correctly rejected. All testing was conducted on a self-hosted instance for educational purposes; no production or third-party system was involved.

## Scope and Objectives

**Scope:** the Brute Force module of a locally hosted DVWA (Damn Vulnerable Web Application) instance, used strictly as an educational target inside an isolated lab environment. No external, production, or third-party system was accessed or affected.

**Objective:** to evaluate, from a defensive standpoint, how the login endpoint validates the HTTP method, the submitted credentials, and the session identifier — and to document the methodology and findings without providing attack implementation steps.

## Testing Environment and Tools

| Component | Detail |
| --- | --- |
| Browser | Chromium |
| Target application | DVWA (Damn Vulnerable Web Application) |
| Target address | 127.0.0.1:8081 (local only) |
| Intercepting proxy | Burp Suite, listener bound to 127.0.0.1:8081 |
| DVWA security level | "low" (observed via the `security` cookie value on captured requests) |

## Methodology

Traffic between the browser and the local application was intercepted and inspected using Burp Suite's proxy. Starting from a baseline login request, individual requests were replayed with exactly one parameter altered at a time — the HTTP verb, the password value, or the session cookie — while every other field was held constant, and the resulting server response (status code, headers, and body) was recorded for each variation.

This approach isolates the effect of a single parameter change per observation and does not rely on any scripted, automated, or brute-force tooling. It is descriptive and observational by design: this report documents what was observed and what it indicates about the application's defenses, and does not provide step-by-step instructions for carrying out an attack.

## Test Case Observations

| # | HTTP Method | Credentials | Session Cookie | Server Response | Observation |
| --- | --- | --- | --- | --- | --- |
| 1 | POST (baseline) | admin / password123 | Original PHPSESSID | 200 OK | Baseline capture; parameters submitted as URL query-string values. |
| 2 | POST | admin / password123 | PHPSESSID replaced with an arbitrary value | 302 Found → redirected to login.php | The forged session identifier was not honored; the server redirected to the login page instead of serving the authenticated page. |
| 3 | PUT (method changed from POST) | admin / password | Original PHPSESSID | 200 OK | Server processed the request and returned the page despite the non-standard method. |
| 4 | DELETE (method changed from POST) | admin / password | Original PHPSESSID | 200 OK | Server again returned the page normally despite the non-standard method. |

## Defensive Analysis

**HTTP method is not enforced.** The endpoint returned 200 OK to both PUT and DELETE requests carrying the same query-string parameters as the original POST. This indicates the server-side handler reads input from a source populated regardless of verb (such as PHP's `$_REQUEST`, or parameters taken from the query string irrespective of method) rather than restricting the form logic to the single verb the form was designed to submit. On a production system, this weakens the value of verb-based access rules, complicates logging and WAF policies that assume a fixed method per route, and is generally a sign of loose input-handling discipline around the endpoint.

**Session validation is effective.** When PHPSESSID was replaced with an arbitrary value, the server did not honor it — it returned a 302 redirect to the login page rather than serving authenticated content. This is a positive control: it shows the session token is validated server-side and is not trusted simply because a client supplies a value.

**Context on credential responses.** Because this module is deliberately configured at DVWA's "low" security level, built-in protections such as rate limiting, lockout, and CAPTCHA are intentionally disabled by the training application itself. The page returning 200 OK regardless of whether the submitted password was correct reflects this training configuration and the page's design (it renders a result message in the body rather than changing HTTP status), not a defect introduced by the method or password changes made during testing.

## Risk Assessment

| Finding | Risk Level (if seen on a production system) | Rationale |
| --- | --- | --- |
| Lack of HTTP method enforcement | Low–Medium | Does not by itself grant access here, but indicates weak server-side input handling that could compound other issues on a real endpoint. |
| No rate limiting / lockout at "low" security | Informational (by design) | Expected behavior of this intentionally vulnerable training module; mirrors a known class of real-world weak-authentication findings rather than a defect introduced during this exercise. |
| Session token handling | No risk / positive control | A forged PHPSESSID value was correctly rejected, confirming session identifiers are validated server-side. |

DVWA is an intentionally vulnerable training application, not a production system; these findings reflect expected teaching scenarios rather than a real-world incident.

## Recommendations

- Enforce an explicit allow-list of HTTP methods at the web server or application framework level, rejecting any verb an endpoint was not designed to accept.
- Validate credentials and process form logic consistently regardless of which HTTP method delivered the parameters, rather than relying on verb-specific code paths.
- In any production authentication system, apply rate limiting, account lockout, and/or CAPTCHA after repeated failed attempts, independent of a training tool's "security level" concept.
- Continue validating session identifiers exclusively server-side, as already observed here, and never trust a client-supplied identifier at face value.
- Restrict intentionally vulnerable training tools, run at reduced security settings, to isolated, non-internet-facing lab environments.

## Conclusion

This was a controlled, local exercise against an intentionally vulnerable training application, conducted to build practical familiarity with how authentication endpoints can be examined defensively using a proxy tool. The findings — permissive HTTP method handling paired with effective session-token validation — illustrate a common real-world authentication weakness pattern in a safe, educational setting, and reinforce the value of standard defensive controls: strict method enforcement, consistent credential validation, and server-side session management.
