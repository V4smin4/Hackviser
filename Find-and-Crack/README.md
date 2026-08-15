🔎 Find and Crack — Hackviser Write-up

«Hackviser Warmup Challenge | Web Security | Privilege Escalation | Password Cracking»

---

Challenge Information

Field| Details
Platform| Hackviser
Challenge| Find and Crack
Difficulty| Medium
Category| Web Security / Privilege Escalation / Password Cracking
Points| 43
Target OS| Linux

---

Overview

Find and Crack is a Hackviser warmup challenge that demonstrates how an outdated web application can provide an initial foothold, followed by privilege escalation through an insecure "sudo" configuration.

After gaining administrative access, a protected backup archive was discovered and investigated to identify sensitive information stored inside the organization's asset inventory.

Attack Path

Web Enumeration
       ↓
GLPI Discovery
       ↓
Known Vulnerability
       ↓
Initial Access
       ↓
Configuration File Discovery
       ↓
Sudo Enumeration
       ↓
Misconfigured find Permission
       ↓
Root Access
       ↓
Backup Archive Analysis
       ↓
Sensitive Asset Information

---

1. Information Gathering

The first step was enumerating the target to identify exposed services.

Nmap

nmap -sV energysolutions.hv

Results

Port| State| Service| Version
"80/tcp"| Open| HTTP| Apache httpd 2.4.56
"3306/tcp"| Open| MySQL| MariaDB 10.5.x

The HTTP service was the primary point of interest.

---

2. Web Application Discovery

Visiting the website revealed an Energy Solutions Inc. landing page.

An IT Management section led to a login interface for:

«GLPI»

GLPI is an open-source IT asset management and service management platform.

Because asset-management systems can contain sensitive organizational information, the application was investigated further.

---

3. Vulnerability Identification

The installed GLPI environment was researched for known vulnerabilities.

The investigation identified a vulnerability affecting the htmLawed component used by vulnerable GLPI installations.

The corresponding Metasploit module was located during vulnerability research.

exploit/linux/http/glpi_htmlawed_php_injection

The vulnerability could provide remote command execution on affected installations.

---

4. Initial Access

Metasploit Framework was used in the Hackviser lab environment to validate the vulnerability.

msfconsole

The relevant module was searched for using:

search glpi

After configuring the module for the lab target, successful command execution was obtained.

Initial User

www-data

This provided the initial foothold on the Linux server.

---

5. Post-Exploitation Enumeration

After obtaining access, the application files were inspected to understand how GLPI was configured.

A database configuration file was located under the GLPI installation:

/var/www/html/glpi/config/config_db.php

The file contained database connection information used by the application.

Database Information

Parameter| Value
Database Host| localhost
Database User| glpiuser
Database| glpi

«Note: Credentials discovered during the lab are intentionally not reproduced here.»

This demonstrates why application configuration files should never expose credentials unnecessarily.

---

6. Privilege Enumeration

The next step was checking the privileges available to the compromised account.

The system's "sudo" configuration revealed that the current user could execute the "find" utility with elevated privileges without entering a password.

Finding

NOPASSWD

was associated with:

/bin/find

This represented a serious privilege-escalation misconfiguration.

---

7. Privilege Escalation

The "find" utility can be abused when incorrectly granted unrestricted "sudo" privileges.

The issue was verified against the Hackviser lab environment, resulting in administrative access.

Privilege Verification

whoami

Result:

root

«Root access successfully obtained.»

The attack had now progressed from an externally accessible web application to full administrative access on the server.

---

8. Backup Archive Discovery

With administrative privileges, the filesystem was searched for backup files.

A backup archive was discovered under the root user's directory:

/root/backup.zip

The archive contained internal IT asset information.

Archive Contents

monitors.csv
computers.csv
network-devices.csv
printers.csv

These files represented different categories of organizational assets.

---

9. Password-Protected Archive

The backup archive was protected by a password.

A password-auditing process was performed against the archive in the controlled Hackviser environment using a standard password wordlist.

The password was successfully recovered during the lab.

«Sensitive credentials are omitted from this public write-up.»

The archive could then be examined to understand the information stored inside the backup.

---

10. Data Analysis

The most interesting file was:

computers.csv

Reviewing the records revealed an asset associated with:

Ethan Friedman

The record contained a suspicious note indicating that the device may have been involved in unauthorized cryptocurrency-mining activity.

Finding

Field| Finding
Asset Type| Laptop
Operating System| Linux
Status| In use
Location| HQ
Security Note| Suspicious activity

This was the final information needed to complete the challenge.

---

11. Attack Chain

flowchart TD
    A[Nmap Enumeration] --> B[Apache / GLPI]
    B --> C[GLPI Vulnerability]
    C --> D[Initial Access]
    D --> E[www-data]
    E --> F[Configuration File]
    F --> G[Sudo Enumeration]
    G --> H[Misconfigured find]
    H --> I[Privilege Escalation]
    I --> J[Root]
    J --> K[Backup Archive]
    K --> L[Archive Analysis]
    L --> M[Sensitive Asset Information]

---

12. Attack Summary

Stage| Technique| Result
1| Service Enumeration| HTTP and MariaDB identified
2| Web Enumeration| GLPI discovered
3| Vulnerability Research| GLPI vulnerability identified
4| Initial Access| "www-data" obtained
5| Configuration Enumeration| Database configuration discovered
6| Sudo Enumeration| "find" identified with elevated permissions
7| Privilege Escalation| "root" access obtained
8| File Discovery| "backup.zip" discovered
9| Archive Analysis| Internal asset records recovered
10| Data Analysis| Suspicious asset identified

---

13. Impact

The attack chain demonstrated how an exposed and vulnerable IT management application can become a gateway to sensitive internal information.

The compromise progressed through:

External Web Application
        ↓
Remote Code Execution
        ↓
Low-Privilege Account
        ↓
Sudo Misconfiguration
        ↓
Root Access
        ↓
Backup Data
        ↓
Sensitive Internal Information

This illustrates the importance of securing not only the web application itself, but also the underlying operating system and backup infrastructure.

---

14. Defensive Recommendations

🔐 Patch Web Applications

Keep GLPI and all third-party components updated and remove vulnerable versions.

🛡️ Review Sudo Rules

Avoid granting unrestricted "sudo" access to utilities that can execute arbitrary commands.

📁 Protect Configuration Files

Database configuration files should be protected with appropriate filesystem permissions.

💾 Secure Backups

Backup archives containing organizational information should use strong passwords and appropriate encryption.

🔎 Monitor Asset Systems

IT asset-management platforms should be monitored for unauthorized access and suspicious changes.

🔑 Use Strong Credentials

Weak or reused passwords can make protected archives vulnerable to password-guessing attacks.

---

15. Key Takeaways

- Web applications can provide a direct entry point into internal infrastructure.
- Vulnerable third-party components should be patched regularly.
- Configuration files can expose valuable information if improperly protected.
- "sudo" misconfigurations are a common privilege-escalation risk.
- Backup files can contain highly sensitive organizational information.
- Security assessments should continue beyond initial access and include post-exploitation enumeration.

---

16. Tools Used

Tool| Purpose
Nmap| Service and version enumeration
Metasploit Framework| Vulnerability research and lab exploitation
Linux CLI| System enumeration
Sudo Enumeration| Privilege analysis
Password Auditing Tools| Controlled archive password auditing
CSV Analysis| Reviewing recovered asset information

---

17. Conclusion

The Find and Crack challenge demonstrated a complete attack chain beginning with web-service enumeration and vulnerability discovery, followed by initial access, privilege escalation, and analysis of sensitive backup data.

The key lesson is that security weaknesses often become significantly more dangerous when chained together.

«A vulnerable web application + excessive system privileges + poorly protected backups can result in significant information exposure.»

---

References

- Hackviser — Find and Crack Challenge
- GLPI Security Research
- Linux Sudo Security
- General Password Security Best Practices

---

Author

V4smin4

"Cybersecurity • CTF • Web Security • Linux • Security Research"
