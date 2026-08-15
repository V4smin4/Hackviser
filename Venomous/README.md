nomous — Hackviser Write-up

«Hackviser Warmup Challenge | Web Security | LFI | Log Poisoning»

---

Challenge Information

Field| Details
Platform| Hackviser
Challenge| Venomous
Difficulty| Easy / Medium
Category| Web Security / LFI / Log Poisoning
Target Service| Nginx
Initial Access| Web Application

---

Overview

Venomous is a Hackviser warmup challenge focused on identifying a Local File Inclusion vulnerability and understanding how file inclusion can become significantly more dangerous when combined with attacker-controlled server logs.

The challenge demonstrates the following attack chain:

Directory Traversal → Local File Inclusion → Log Poisoning → Remote Code Execution

---

1. Information Gathering

The first step was enumerating the target machine and identifying exposed services.

Nmap

nmap -sV <target_ip>

Results

Port| State| Service| Version
"80/tcp"| Open| HTTP| Nginx 1.18.0

Only one exposed service was identified:

80/tcp → HTTP → Nginx 1.18.0

The web application became the primary target for further investigation.

---

2. Web Application Discovery

Visiting the website revealed a business dashboard containing sales information and an Invoice section.

The invoice functionality included a feature for displaying or downloading invoice reports.

During request inspection, the application was found to use a user-controlled parameter:

show-invoice.php?invoice=invoice-8741.html

The parameter appeared to determine which file was loaded by the application.

This behavior suggested a potential Local File Inclusion (LFI) vulnerability.

---

3. Local File Inclusion

The "invoice" parameter was tested for directory traversal behavior.

The goal was to determine whether files outside the intended invoice directory could be accessed.

Testing

../
../../
../../../
../../../../

The application was found to accept traversal sequences and return files outside the expected directory.

This confirmed the presence of a Local File Inclusion / Directory Traversal vulnerability.

---

4. Sensitive File Access

As part of the lab investigation, the vulnerability was tested against a standard Linux system file.

The result confirmed that arbitrary local files could be read through the vulnerable parameter.

Vulnerability Impact

Property| Finding
Vulnerability| Local File Inclusion
Parameter| "invoice"
Impact| Arbitrary Local File Read
Attack Type| Directory Traversal

The vulnerability was therefore more serious than a simple invoice-display issue.

---

5. Log Poisoning

After confirming arbitrary local file inclusion, the next security question was whether an attacker-controlled file could be included.

Web-server logs were investigated because request data can be written into access logs.

The relevant Nginx log location was identified as:

/var/log/nginx/access.log

This created a potential attack chain:

Attacker-Controlled Request
        ↓
Nginx Access Log
        ↓
Local File Inclusion
        ↓
Server-Side Processing

---

6. Remote Code Execution

The LFI vulnerability was combined with log poisoning in the controlled Hackviser environment.

The objective was to demonstrate that attacker-controlled content written to a server log could potentially be processed when the log was included by the vulnerable application.

This resulted in Remote Code Execution (RCE).

Result

The command execution context was associated with the web-server account:

www-data

This demonstrated the significant impact of chaining LFI with log poisoning.

---

7. Initial Access

After achieving command execution, the execution context was verified.

whoami

Result:

www-data

The attacker therefore had command execution under a low-privileged web-server account.

---

8. Post-Exploitation Enumeration

After obtaining command execution, the web application's filesystem was investigated.

The vulnerable application referenced:

show-invoice.php

Further filesystem enumeration helped confirm the application's structure and identify files relevant to the vulnerability.

At this stage, the main objective was understanding the application's security configuration rather than performing further exploitation.

---

9. Attack Chain

flowchart TD
    A[Nmap Enumeration] --> B[Nginx Web Server]
    B --> C[Invoice Functionality]
    C --> D[User-Controlled invoice Parameter]
    D --> E[Directory Traversal]
    E --> F[Local File Inclusion]
    F --> G[Nginx Access Log]
    G --> H[Log Poisoning]
    H --> I[Remote Code Execution]
    I --> J[www-data]

---

10. Attack Summary

Stage| Technique| Result
1| Service Enumeration| Nginx identified
2| Web Enumeration| Invoice functionality discovered
3| Parameter Analysis| "invoice" parameter identified
4| Directory Traversal| Path traversal confirmed
5| LFI Testing| Arbitrary local file read
6| Log Analysis| Nginx access log identified
7| Log Poisoning| Attacker-controlled content introduced
8| Code Execution| RCE demonstrated
9| Access Verification| "www-data" obtained

---

11. Impact

The main security issue was the ability to combine multiple weaknesses into a complete remote-code-execution chain.

Attack Path

Public Web Application
        ↓
User-Controlled File Parameter
        ↓
Directory Traversal
        ↓
Local File Inclusion
        ↓
Nginx Access Log
        ↓
Log Poisoning
        ↓
Remote Code Execution
        ↓
www-data

A vulnerability that initially appeared to provide only local file reading therefore became a much more serious server-compromise risk.

---

12. Defensive Recommendations

🔒 Use an Allowlist

Applications should never directly use arbitrary user-controlled paths.

Only known, predefined invoice identifiers should be accepted.

🛡️ Prevent Directory Traversal

User input should be validated and normalized before being used in filesystem operations.

📁 Restrict Log Access

Web applications should not unnecessarily expose server log files to application-level file inclusion mechanisms.

⚙️ Disable Unnecessary Code Execution

Uploaded or included files should never be interpreted as executable server-side code unless explicitly required.

🔐 Apply Least Privilege

Web-server accounts should have only the permissions required to perform their intended tasks.

🔎 Monitor Suspicious Requests

Repeated traversal patterns and unusual requests should be detected and logged.

---

13. Key Takeaways

- User-controlled filesystem paths are dangerous when not properly validated.
- Directory traversal can lead to Local File Inclusion.
- LFI can sometimes be chained with other weaknesses to achieve code execution.
- Web-server logs can contain attacker-controlled input.
- Web applications should operate with minimal privileges.
- Input validation and secure file-handling practices are essential.

---

14. Tools Used

Tool| Purpose
Nmap| Service and version enumeration
Browser / HTTP Analysis| Web application investigation
Linux CLI| File and system enumeration
Nginx Log Analysis| Understanding log behavior
LFI Testing| Local file inclusion analysis

---

15. Conclusion

The Venomous challenge demonstrated how a Local File Inclusion vulnerability can become significantly more dangerous when combined with attacker-controlled log data.

The challenge reinforces an important security principle:

«A vulnerability should always be evaluated in the context of what other system components it can interact with.»

Proper input validation, strict file-access controls, secure web-server configuration, and least-privilege principles can significantly reduce the risk of this attack chain.

---

References

- Hackviser — Venomous Challenge
- OWASP — Local File Inclusion
- OWASP — Path Traversal
- Nginx Security Documentation

---

Author

V4smin4

"Cybersecurity • CTF • Web Security • Linux • Security Research"
