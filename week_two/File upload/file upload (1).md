

File Upload Vulnerability – PoC
Target: DVWA (Damn Vulnerable Web Application)
## Vulnerability: File Upload Validation Weakness
## Security Level: Low
HTTP Method: POST
## Endpoint: /vulnerabilities/upload/
Parameter: uploaded
## Payload Used:
Original Filename: test.txt
## Original Content:
This is a test file for DVWA File Upload testing.
Modified Filename: test.jpg
The filename was modified through Burp Suite Repeater while keeping the original text
content unchanged.
Purpose: To test whether the application validates the actual file content or relies only on the
filename extension.
## Testing Procedure
- Accessed the File Upload functionality.
- Uploaded a normal text file named test.txt.
- Intercepted the upload request using Burp Suite.
- Sent the request to Burp Repeater.
- Changed the filename from test.txt to test.jpg while keeping the original text content.
- Verified that the uploaded file existed in the uploads directory.
## Observation
The application accepted the text file with a modified image extension. Firefox could not
display the resulting file as an image because its content was not a valid image.
## Security Impact
Insufficient file validation may allow unexpected file types to be uploaded, depending on the
application’s validation and storage controls.
## Remediation

Validate file extensions using an allowlist.
Verify the actual file content, not only its extension.
Generate random filenames.
Store uploaded files outside the web root.
Disable execution permissions in the upload directory.
## Evidence



