🐝 Bee — Hackviser Write-up

«Hackviser Warmup Challenge | Web Security | SQL Injection | File Upload»

---

Challenge Information

Field| Details
Platform| Hackviser
Challenge| Bee
Difficulty| Easy / Medium
Category| Web Security / SQL Injection / File Upload
Target| InnovifyAI
Initial Access| Web Application

---

Overview

Bee is a Hackviser warmup challenge focused on identifying weaknesses in a web application and chaining multiple vulnerabilities together.

The challenge demonstrates how insufficient server-side input validation and insecure file-upload handling can lead to remote code execution.

Attack Path

Web Enumeration
       ↓
InnovifyAI Application
       ↓
Login Panel
       ↓
SQL Injection
       ↓
Authentication Bypass
       ↓
Authenticated Dashboard
       ↓
File Upload Functionality
       ↓
Unsafe File Handling
       ↓
Remote Code Execution
       ↓
Application Configuration Discovery

---

1. Information Gathering

The first step was enumerating the target machine.

Nmap

nmap -sV <target_ip>

Results

Port| State| Service
"80/tcp"| Open| HTTP
"3306/tcp"| Open| MySQL

The HTTP service was the primary point of interest.

---

2. Web Application Discovery

Visiting the web service revealed a corporate landing page for:

«InnovifyAI»

The application contained a Login feature.

The login interface redirected to a separate dashboard host.

During testing, the dashboard hostname was not initially resolving correctly, so the local environment was configured to resolve the hostname to the target machine.

---

3. Login Functionality

The login form contained two main input fields:

- Email
- Password

The application appeared to perform authentication against a backend database.

The email field also used browser-side validation.

Important Security Observation

Client-side validation is not a security boundary.

A browser can enforce restrictions such as:

<input type="email">

but these restrictions can be modified or bypassed by the client.

Therefore, validation must always be implemented and enforced on the server side.

---

4. SQL Injection

The login functionality was tested for SQL Injection.

The application was found to improperly handle user-controlled input when constructing database queries.

This created an authentication-bypass condition.

Vulnerability

Property| Details
Vulnerability| SQL Injection
Affected Area| Login Form
Impact| Authentication Bypass
Root Cause| Unsafe database query construction

The vulnerability allowed authentication controls to be bypassed in the Hackviser lab environment.

---

5. Authentication Bypass

After successfully bypassing the vulnerable login mechanism, access to the application's dashboard was obtained.

The dashboard provided authenticated functionality that was not available from the public landing page.

This demonstrated the impact of the SQL Injection vulnerability:

Unauthenticated User
        ↓
SQL Injection
        ↓
Authentication Bypass
        ↓
Authenticated Dashboard

---

6. File Upload Discovery

The authenticated dashboard contained a Settings section with a profile-picture upload feature.

The upload functionality was investigated to determine whether the application properly validated uploaded files.

Security Testing

The application was tested with files that were not normal image files.

The server-side validation was found to be insufficient.

This created a potentially dangerous file-upload condition.

---

7. File Upload Vulnerability

The upload functionality accepted an unexpected file type.

The key security issue was not simply that a file could be uploaded, but that uploaded content could potentially be processed by the web server.

Security Issue

Property| Details
Feature| Profile Image Upload
Validation| Insufficient server-side validation
Risk| Arbitrary File Upload
Potential Impact| Remote Code Execution

A secure implementation should validate:

- File extension
- MIME type
- File contents
- File size
- Storage location
- Execution permissions

---

8. Remote Code Execution

The vulnerable upload functionality was chained with the previous authentication bypass to demonstrate remote code execution in the controlled Hackviser environment.

The uploaded file was processed by the server, resulting in command execution under the web-server account.

Initial Execution Context

www-data

This demonstrated the difference between:

File Upload

and:

File Upload + Server-Side Execution

The second scenario can result in a serious compromise.

---

9. Application Configuration Discovery

After obtaining command execution, the application filesystem was enumerated.

A database configuration file was identified:

/var/www/dashboard.innovifyai.hackviser/db_connect.php

The configuration contained database connection information.

Security Finding

Information| Status
Database Host| Discovered
Database Name| Discovered
Database Username| Discovered
Database Password| Exposed in configuration

Sensitive credentials are intentionally not reproduced in this public write-up.

---

10. Attack Chain

flowchart TD
    A[Nmap Enumeration] --> B[InnovifyAI]
    B --> C[Login Panel]
    C --> D[SQL Injection]
    D --> E[Authentication Bypass]
    E --> F[Dashboard Access]
    F --> G[File Upload]
    G --> H[Insufficient Validation]
    H --> I[Remote Code Execution]
    I --> J[www-data]
    J --> K[Application Enumeration]
    K --> L[Database Configuration]
    L --> M[Sensitive Credentials Exposed]

---

11. Attack Summary

Stage| Technique| Result
1| Service Enumeration| HTTP and MySQL identified
2| Web Enumeration| InnovifyAI discovered
3| Input Testing| Login functionality investigated
4| SQL Injection| Authentication bypass achieved
5| Dashboard Enumeration| Upload functionality discovered
6| File Upload Testing| Insufficient validation identified
7| Server-Side Execution| Remote code execution demonstrated
8| Filesystem Enumeration| Application configuration discovered
9| Configuration Analysis| Database credentials exposed

---

12. Impact

The vulnerabilities could be chained into a serious application compromise:

Public Web Application
        ↓
SQL Injection
        ↓
Authentication Bypass
        ↓
Dashboard Access
        ↓
Unsafe File Upload
        ↓
Remote Code Execution
        ↓
Application Data Exposure

The most important finding was that multiple relatively simple security weaknesses could be chained together to significantly increase the overall impact.

---

13. Defensive Recommendations

🔐 Use Parameterized Queries

Applications should use prepared statements instead of constructing SQL queries directly from user input.

🛡️ Enforce Server-Side Validation

Client-side validation should only improve user experience.

It should never be relied upon as a security mechanism.

📁 Secure File Uploads

Uploaded files should:

- Be validated server-side.
- Be renamed to safe random filenames.
- Be stored outside executable web directories.
- Have execution disabled.
- Be restricted to expected file types.

🔑 Protect Configuration Files

Database credentials should not be unnecessarily exposed in application directories.

Secrets should be stored securely and should not be committed to public repositories.

🔍 Monitor Web Applications

Logging and monitoring should detect suspicious authentication attempts, unexpected uploads, and unusual command execution.

---

14. Key Takeaways

- Client-side validation can be bypassed and should never be treated as a security control.
- SQL Injection can completely bypass authentication when database queries are improperly constructed.
- File uploads are dangerous when server-side validation is insufficient.
- Uploaded files should never be allowed to execute as server-side code.
- Configuration files can expose sensitive database credentials.
- Vulnerabilities become significantly more dangerous when chained together.

---

15. Tools Used

Tool| Purpose
Nmap| Service enumeration
Browser DevTools| Client-side validation analysis
Linux CLI| Post-exploitation enumeration
Web Application Testing| Authentication and upload testing
SQL Injection Testing| Authentication-bypass analysis

---

16. Conclusion

The Bee challenge demonstrated how weaknesses in different layers of a web application can be chained together.

The initial SQL Injection vulnerability allowed authentication controls to be bypassed, while the insecure file-upload functionality provided a path toward server-side code execution.

The main lesson is:

«Security controls must be enforced on the server, and every user-controlled input should be treated as untrusted.»

---

References

- Hackviser — Bee Challenge
- OWASP — SQL Injection
- OWASP — File Upload Security
- OWASP — Input Validation

---

Author

V4smin4

"Cybersecurity • CTF • Web Security • SQL • Linux"
