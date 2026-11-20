file inclusion - medium security

## 1-Target

Application: Damn Vulnerable Web Application (DVWA)

Module: File Inclusion

- Security Level: Medium

Environment: Local Docker Environment

Endpoint: /vulnerabilities/fi/

HTTP Method: GET

## 2-Vulnerability Description

Path Traversal is a vulnerability that allows an attacker to access files outside the intended application directory by manipulating file paths.

In DVWA Medium Security, the application attempts to prevent directory traversal by removing the ../ sequence from the user-supplied input.

However, the filtering mechanism uses a single string replacement operation and does not adequately validate the resulting path. This allows an attacker to bypass the filter using nested directory traversal sequences.

## 3-Testing Procedure

- Set the DVWA Security Level to Medium.

- Open the File Inclusion module.

- Select one of the available file links.

- Intercept the HTTP request using Burp Suite.

- Identify the page parameter in the GET request.

- Send the request to Burp Repeater.

- First, test a normal directory traversal payload.

- Observe that the application does not display the requested file contents.

- Next, test a nested directory traversal payload.

- Send the modified request.

- Observe that the contents of /etc/passwd are successfully displayed in the HTTP response.

- Verify the file contents directly inside the DVWA Docker container.

## 4-Payloads Used

Payload 1 – Normal Directory Traversal:

../../../../etc/passwd

Result: Blocked by the Medium filtering mechanism.

Payload 2 – Nested Directory Traversal:


....//....//....//....//....//etc/passwd

Result: Successfully bypassed the filter and displayed the contents of /etc/passwd.

## 5-Observation / Results

## Two different payloads were tested:

- The normal traversal payload was blocked by the application’s filtering mechanism.

- The nested traversal payload successfully bypassed the filter.

- The HTTP response contained entries from the /etc/passwd file, including system accounts such as root, daemon, and www-data.

- The same file contents were verified directly inside the Docker container.

These results confirm that the Medium Security filtering mechanism can be bypassed.

## 6-Remediation

- Avoid using blacklist-based filtering for file paths.

- Use an allowlist of permitted files whenever possible.

- Canonicalize and normalize file paths before validation.

- Verify that the resolved path remains inside the intended application directory.

- Use PHP functions such as real path() for path validation.

- Restrict the web server’s filesystem permissions.

- Avoid accepting arbitrary file paths directly from user input.

## 7-Conclusion

The DVWA Medium Security test demonstrated that the File Inclusion functionality is vulnerable to a Path Traversal filter bypass.

Although the application blocks a normal traversal payload, the nested directory traversal technique successfully bypasses the filtering mechanism and allows access to the /etc/passwd file.

This demonstrates that simple string replacement is not a reliable security mechanism for preventing Path Traversal attacks.

## 8-Evidence / Screenshots
