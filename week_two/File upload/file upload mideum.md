

## File Upload Vulnerability – Medium Security
1-Target
Application: Damn Vulnerable Web Application (DVWA)
## Module: File Upload
## Security Level: Medium
## Environment: Local Docker Environment
## Endpoint: /vulnerabilities/upload/
HTTP Method: POST
Parameter: uploaded
2-Vulnerability Description:
The File Upload functionality in DVWA Medium Security attempts to restrict uploaded files to
JPEG and PNG formats by checking the MIME type provided in the HTTP request.
However, the application trusts the client-supplied MIME type instead of verifying the actual
file content. An attacker can modify the MIME type in the HTTP request using Burp Suite and
bypass the file upload validation.
3-Testing Procedure
Set DVWA Security Level to Medium.
Open the File Upload module.
Create a valid PNG image and upload it.
Verify that the PNG image is successfully uploaded.
Create a text file named test.txt.
Attempt to upload the text file normally.
Observe that the application rejects the file and displays:
Your image was not uploaded. We can only accept JPEG or PNG images.
Intercept the upload request using Burp Suite.
Send the request to Burp Repeater.
Modify the file’s Content-Type header from text/plain to image/png, while keeping the
filename and actual text content unchanged.
Send the modified request.
Observe that the application accepts the file.
Verify the uploaded file inside the server’s upload directory.
4-Payload Used
## Original File:
Filename: test.txt
Content-Type: text/plain
File Content: This is a test file for DVWA File Upload testing.
## Modified Request:

Filename: test.txt
Content-Type: image/png
File Content: This is a test file for DVWA File Upload testing.
Only the Content-Type value was modified.
5-Observation / Results
A valid PNG image was successfully uploaded.
A text file uploaded normally was rejected.
After changing the MIME type from text/plain to image/png using Burp Suite Repeater, the
text file was successfully uploaded.
The file was found in the server directory:
## /var/www/html/hackable/uploads/
Reading the uploaded file confirmed that its actual content was plain text, not an image.
6-Remediation
Validate the actual file content on the server side.
Use file signature verification (magic bytes) instead of trusting the client-supplied MIME
type.
Allow only explicitly permitted file extensions.
Verify the file format using a trusted image-processing library.
Generate random filenames for uploaded files.
Store uploaded files outside the web root whenever possible.
Disable script execution in the upload directory.
Apply appropriate file size restrictions.
7-Conclusion
The DVWA Medium File Upload test demonstrated that MIME type validation can be
bypassed by modifying the Content-Type header in the HTTP request.
Although the application rejected a text file during normal upload, it accepted the same file
after its MIME type was changed to image/png.
This confirms that relying only on client-supplied MIME types is insufficient for secure file
upload validation.
8-Evidence / Screenshots







