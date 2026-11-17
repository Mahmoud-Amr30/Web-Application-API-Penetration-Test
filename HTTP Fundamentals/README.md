# HTTP Fundamentals & Authentication / Session Flow

**Team:** Team 1 — Establishing Theoretical Concepts  
**Target Project:** Project 2 — Web Application & API Penetration Test (DVWA Lab)

---

## 1. Introduction to HTTP and HTTPS

**HTTP (Hypertext Transfer Protocol)** is the main communication protocol used by web applications. It allows data to be exchanged between a Client (such as a web browser) and a Server.

HTTP follows a communication model called **Request / Response**:

1. The Client sends a Request to access a page or perform an operation.
2. The Server receives and processes the request.
3. The Server sends back a Response containing the requested data or a message describing the result.

**HTTPS** is HTTP secured using TLS encryption. It protects data while it is being transferred between the browser and the server, such as login credentials and session cookies.

### Main Differences

| HTTP | HTTPS |
|---|---|
| Data is transferred without transport-layer encryption. | Data is protected using TLS encryption. |
| No TLS-based confidentiality or integrity. | Provides confidentiality, integrity, and server authentication. |

---

## 2. HTTP Request and HTTP Response

### 2.1 HTTP Request

An HTTP Request is a message sent by the browser to the server.

It usually consists of:

- **Request Method:** Defines the requested operation.
- **URL / Path:** Specifies the requested resource.
- **Headers:** Provide additional information about the request.
- **Body:** Contains submitted data, commonly used with methods such as POST.

Example:

\`\`\`http
GET /login.php HTTP/1.1
Host: 127.0.0.1:4280
Accept: text/html
User-Agent: Mozilla/5.0
Cookie: PHPSESSID=example_session_id
\`\`\`

### 2.2 HTTP Response

An HTTP Response is the message returned by the server.

It usually consists of:

- **Status Code:** Indicates the result of the request.
- **Headers:** Provide additional information about the response.
- **Body:** Contains the returned page or data.

Example:

\`\`\`http
HTTP/1.1 200 OK
Content-Type: text/html
Set-Cookie: PHPSESSID=example_session_id
\`\`\`

---

## 3. HTTP Request Methods

| Method | Purpose |
|---|---|
| GET | Retrieve data or display a page. |
| POST | Submit data or perform an operation. |
| PUT | Replace an entire resource. |
| PATCH | Partially modify a resource. |
| DELETE | Delete a resource. |
| HEAD | Retrieve response headers without the response body. |
| OPTIONS | Identify communication options available for a resource. |
| CONNECT | Establish a tunnel to a server. |
| TRACE | Perform a diagnostic trace of the request path. |

### Practical Examples

- Opening a login page: `GET /login.php`
- Submitting login credentials: `POST /login.php`
- Updating a user through an API: `PUT /users/10`
- Deleting a user: `DELETE /users/10`

**Security Note:** Using a particular HTTP method does not automatically make an operation secure. The server must still enforce proper validation and authorization.

---

## 4. HTTP Headers

Headers provide additional information about HTTP requests and responses.

### Common Request Headers

- Host
- User-Agent
- Accept
- Content-Type
- Authorization
- Cookie
- Referer

### Common Response Headers

- Content-Type
- Set-Cookie
- Cache-Control
- Location
- Content-Length

---

## 5. Cookies and Sessions

### 5.1 Cookies

Cookies are small pieces of data stored by the browser and automatically sent to the server with matching requests.

**Common Uses:**

- Maintaining login state.
- Saving user preferences.
- Associating requests with a session.

### Important Cookie Attributes

| Attribute | Purpose |
|---|---|
| HttpOnly | Blocks JavaScript access to the cookie. |
| Secure | Sends the cookie only over HTTPS. |
| SameSite | Reduces certain CSRF risks. |

### 5.2 Sessions

HTTP is **stateless**, which means each request is independent by default.

Sessions allow the server to maintain a user's state across requests using a **Session ID**.

### Typical Session Workflow

1. Login succeeds.
2. Server creates a session.
3. Server sends the Session ID using `Set-Cookie`.
4. Browser stores the session cookie.
5. Browser resends it using `Cookie` in later requests.
6. Server uses the Session ID to retrieve session data and confirm authentication.

---

## 6. Authentication and Authorization

- **Authentication:** Verifies a user's identity.

  Example: Are the username and password correct?

- **Authorization:** Determines what an authenticated user may access.

  Example: A normal user cannot access the admin dashboard.

---

## 7. Authentication / Session Flow Diagram

The following diagram illustrates the login and protected-resource workflow.

\`\`\`mermaid
flowchart TD
    A[User opens login page] --> B[Login page displayed]
    B --> C[User enters credentials]
    C --> D[Browser sends POST request]
    D --> E[Server validates credentials]
    E -->|Invalid| F[Login failed]
    F --> B
    E -->|Valid| G[Server creates session]
    G --> H[Server sends Set-Cookie]
    H --> I[Browser stores session cookie]
    I --> J[User requests protected page]
    J --> K[Browser sends Cookie]
    K --> L[Server validates Session ID]
    L --> M{Session valid?}
    M -->|No| N[Access denied]
    M -->|Yes| O[Server checks authorization]
    O --> P{User permitted?}
    P -->|No| Q[Access denied]
    P -->|Yes| R[Protected resource returned]
\`\`\`

**Observation:** A failed login returns the user to the login page. A successful login establishes an authenticated session.

---

## 8. Explanation of the Authentication Flow

| Stage | Description |
|---|---|
| 1. Login Page | The user opens the login page. |
| 2. Submit Credentials | The user enters a username and password. |
| 3. POST Request | The browser sends credentials to the server. |
| 4. Validation | The server verifies the submitted credentials. |
| 5. Session Creation | The server creates a session after successful authentication. |
| 6. Set-Cookie | The server sends the Session ID to the browser. |
| 7. Store Cookie | The browser stores the session cookie. |
| 8. Authenticated Request | The browser sends the cookie when requesting a protected page. |
| 9. Session Validation | The server verifies the Session ID. |
| 10. Authorization | The server checks the user's permissions. |
| 11. Response | The server grants or denies access. |

---

## 9. Practical Implementation Using Burp Suite and DVWA

DVWA was used as a local training environment on Kali Linux, deployed through Docker.

Burp Suite was used to intercept and analyze HTTP traffic.

### Capture 1 — Before Correcting the Security Level

\`\`\`http
GET /login.php HTTP/1.1
Host: 127.0.0.1:4280
Accept-Language: en-US,en;q=0.9
User-Agent: Mozilla/5.0
Cookie: security=impossible; PHPSESSID=b7117080de1bd5e520e831bd5c548c56
\`\`\`

### Capture 2 — After Setting Security Level to Low

The security level was changed to `Low` through `security.php`.

\`\`\`http
GET /security.php HTTP/1.1
Host: 127.0.0.1:4280
Cookie: PHPSESSID=9a510373f54d7acd8ab9da923ea7029c; security=low
\`\`\`

### Request Analysis

- `GET /login.php`: Requests the login page.
- `Host`: Identifies the local DVWA instance.
- `security=`: A DVWA-specific, client-editable cookie controlling vulnerability difficulty.
- `PHPSESSID`: The PHP session identifier connecting the request to a session.

**Security Observation:**

The `security` cookie is client-editable. A real application should never allow the client to determine its own security posture.

The PHPSESSID value changed between the two captures because a new browser session had started.

Whether it also changes at the exact moment of login, mitigating Session Fixation, is a question for formal testing in Week 3.

### What Was Monitored Using Burp Suite?

- Requests sent by the browser.
- Data submitted during login.
- Cookies exchanged.
- Server responses and status codes.
- Changes in requests before and after authentication.

**Security Note:** PHPSESSID is sensitive. Anyone who obtains a valid session identifier may be able to use that session. Real session IDs should never be published or shared.

### Lab Build Record

| Item | Value |
|---|---|
| Host OS | Kali Linux |
| Image | ghcr.io/digininja/dvwa:latest |
| Commit (HEAD at pull time) | a1978a2fed195acd093010af6395f86a281537ca |
| Commit Date | 18 Oct 2026 |
| Port Binding | 127.0.0.1:4280 |
| Port Exposure | Loopback only — verified via docker ps and ss -tlnp |
| DB Container | mariadb:10 |
| Database Network | Internal network only; no published port |

---

## 10. Important Security Best Practices

1. Use HTTPS throughout the entire application, not only on the login page.
2. Store passwords using secure password-hashing algorithms instead of plain text.
3. Apply appropriate HttpOnly, Secure, and SameSite cookie attributes.
4. Regenerate the Session ID after successful login to reduce Session Fixation risk.
5. Properly invalidate sessions during logout and enforce session expiration.
6. Verify user permissions on every request to protected resources.
7. Never rely on hiding pages or interface elements as a security mechanism for access control.

---

## 11. Conclusion

HTTP is the foundation of communication between web browsers and servers.

Understanding Requests, Responses, Methods, and Headers is essential for analyzing how web applications operate.

Cookies and Sessions help maintain user state even though HTTP is inherently stateless.

Understanding Authentication and Authorization is especially important in Web Application Penetration Testing because it helps testers identify weaknesses in:

- Login mechanisms.
- Session management.
- Access control.

---

**Project:** Project 2 — Web Application & API Penetration Test  
**Lab:** DVWA on Kali Linux  
**Tools:** Burp Suite and Docker