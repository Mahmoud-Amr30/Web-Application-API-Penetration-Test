

File inclusion – PoC
Target Application: Damn Vulnerable Web Application (DVWA)
## Vulnerable Page: File Inclusion
## Testing Tool: Burp Suite
HTTP Method: GET
## Security Level: Low
Testing procedure:
- Opened DVWA and selected the File Inclusion page.
- Set the security level to Low.
- Intercepted the HTTP request using Burp Suite.
- Sent the request to Burp Repeater.
- Modified the “page” parameter using a directory traversal payload.
- Sent the modified request.
## Payload Used:
## ../../dvwa-test.txt
HTTP Request:
GET /vulnerabilities/fi/?page=../../dvwa-test.txt HTTP/1.1
## Observation:
The server returned HTTP 200 OK, and the response body contained:
## PATH_TRAVERSAL_TEST_SUCCESS
This confirms that the application successfully accessed and displayed the contents of the
test file
Security impact:
If exploited in a real application, Path Traversal may allow attackers to access sensitive files
outside the intended directory.
Potential impacts include:
Unauthorized access to application files.
Exposure of configuration files.
Disclosure of sensitive information.
Possible exposure of credentials or system information, depending on file permissions
and application behavior.
In this test, access to a controlled test file was successfully demonstrated. Access to
sensitive files was not tested.

## Remediation:
To prevent file inclusion vulnerabilities, developers should:
- Validate all user-supplied file paths.
- Avoid directly using user input in file system operations.
- Use an allowlist of permitted files.
- Normalize and canonicalize file paths before accessing them.
- Restrict file access to the intended application directory.
- Apply proper file system permissions.
- Avoid exposing sensitive files to the web application.
evidence:





