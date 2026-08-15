🛡️ Glitch — Hackviser Write-up

«Hackviser Warmup Challenge — Web Security & Privilege Escalation»

---

📌 Challenge Information

Property| Details
🏴 Platform| Hackviser
🎯 Challenge| Glitch
📊 Difficulty| Medium
🌐 Category| Web Security / Privilege Escalation
⭐ Points| 43
🐧 Target OS| Linux

---

🎯 Overview

Glitch is a Hackviser warmup machine focused on identifying an outdated web server, obtaining initial access through a known vulnerability, and investigating the target for a local privilege-escalation opportunity.

The challenge demonstrates an important real-world attack chain:

Service Enumeration
        ↓
Outdated Web Server
        ↓
Known Vulnerability
        ↓
Initial Access
        ↓
Local Enumeration
        ↓
Kernel Vulnerability
        ↓
Privilege Escalation
        ↓
Root Access

---

🔎 1. Information Gathering

The first step was identifying the services exposed by the target.

Nmap Scan

nmap -sV goldnertech.hv

Scan Results

Port| State| Service| Version
"22/tcp"| open| SSH| OpenSSH 8.4p1 Debian
"80/tcp"| open| HTTP| Nostromo 1.9.6

The most interesting service was:

80/tcp → HTTP → Nostromo 1.9.6

The outdated Nostromo 1.9.6 version became the primary focus for vulnerability research.

---

🔐 2. Vulnerability Identification

Researching the identified Nostromo version revealed a known vulnerability:

CVE-2019-16278

CVE-2019-16278 affects vulnerable versions of the Nostromo web server and can lead to remote command execution.

Vulnerability Summary

Item| Details
Software| Nostromo
Version| 1.9.6
CVE| CVE-2019-16278
Impact| Remote Command Execution
Attack Type| Remote
Initial Access| Web Service

The vulnerable HTTP service was therefore investigated as the likely initial attack vector.

---

💻 3. Initial Access

Metasploit Framework was used to search for an appropriate module related to the identified service.

msfconsole

Then:

search nostromo

The relevant module was identified as:

exploit/multi/http/nostromo_code_exec

After configuring the module for the Hackviser lab target, the vulnerability was successfully exploited and command execution was obtained.

Initial User

www-data

This confirmed that the initial access was obtained with a low-privileged web-server account.

---

🧭 4. Post-Exploitation Enumeration

After obtaining the initial shell, the next step was understanding the target environment.

Check Current User

whoami

Result:

www-data

Check System Information

uname -a

The system was running a Linux kernel version associated with a known privilege-escalation vulnerability.

This shifted the investigation from initial access to local privilege escalation.

---

⚡ 5. Privilege Escalation

An attempt was made to access sensitive system files:

cat /etc/shadow

The operation was denied because the current account did not have sufficient privileges.

This confirmed that additional privilege escalation was required.

---

🧨 Kernel Vulnerability — Dirty Pipe

The identified kernel version was investigated for known vulnerabilities.

The system was found to be associated with:

«CVE-2022-0847 — Dirty Pipe»

Dirty Pipe is a Linux kernel vulnerability that can allow an unprivileged local user to modify data associated with normally read-only files on vulnerable systems.

In the Hackviser lab environment, this vulnerability provided the privilege-escalation path.

---

👑 6. Root Access

After completing the privilege-escalation stage, the current user was verified again.

whoami

Result:

root

✅ Root access successfully obtained.

This demonstrated a complete compromise of the target from the initial exposed web service.

---

🔗 7. Attack Chain

The complete attack path can be summarized as follows:

┌──────────────────────────────┐
│        Nmap Enumeration      │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│    Nostromo 1.9.6 Detected   │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│       CVE-2019-16278         │
│       Remote Code Execution  │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│       Initial Access         │
│          www-data            │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│     Kernel Enumeration       │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│       CVE-2022-0847         │
│        Dirty Pipe            │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│        Privilege Escalation  │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│          ROOT ACCESS         │
└──────────────────────────────┘

---

📈 8. Attack Summary

Stage| Technique| Result
1️⃣| Service Enumeration| Identified HTTP and SSH
2️⃣| Version Detection| Found Nostromo 1.9.6
3️⃣| Vulnerability Research| Identified CVE-2019-16278
4️⃣| Initial Access| Obtained "www-data"
5️⃣| System Enumeration| Identified vulnerable kernel
6️⃣| Privilege Escalation| Exploited Dirty Pipe in the lab
7️⃣| Verification| Obtained "root"

---

💥 Impact

The combination of an outdated web server and an unpatched Linux kernel resulted in full system compromise.

An attacker could move from:

Unauthenticated Web Access
        ↓
Remote Command Execution
        ↓
Low-Privilege Shell
        ↓
Kernel Privilege Escalation
        ↓
Root

Once root privileges are obtained, the attacker effectively has complete control over the operating system and its accessible data.

---

🛡️ 9. Defensive Recommendations

The attack chain could be mitigated through several security controls:

🔹 Keep Services Updated

Upgrade or replace outdated versions of web servers and other exposed services.

🔹 Patch the Operating System

Apply Linux kernel security updates regularly to prevent exploitation of known vulnerabilities.

🔹 Minimize Exposed Services

Only expose services that are required and restrict administrative services such as SSH where possible.

🔹 Least Privilege

Web applications should operate with the minimum permissions necessary.

🔹 Continuous Vulnerability Management

Regularly scan infrastructure to identify outdated software and known CVEs.

---

🧠 10. Key Takeaways

- 🔍 Service enumeration is an essential first step in a security assessment.
- 🌐 Version information can reveal vulnerable software.
- 🐧 Initial access does not necessarily mean full system compromise.
- ⚡ Local enumeration can reveal privilege-escalation opportunities.
- 🔐 Kernel patching is critical for Linux systems.
- 🛡️ Multiple vulnerabilities can be chained together to create a much larger security impact.

---

🧰 Tools Used

Tool| Purpose
🔎 Nmap| Service and version enumeration
💥 Metasploit Framework| Vulnerability research and lab exploitation
🐧 Linux CLI| System enumeration
🔬 CVE Research| Vulnerability identification

---

📝 Conclusion

The Glitch challenge demonstrated a realistic cybersecurity attack chain beginning with reconnaissance and service enumeration, followed by exploitation of an outdated web service and finally local privilege escalation through a vulnerable Linux kernel.

The main lesson is that security must be approached as a complete process rather than focusing on a single vulnerability.

«One vulnerable service can provide the initial foothold, while an unpatched operating system can turn that foothold into complete system compromise.»

---

📚 References

- CVE-2019-16278 — Nostromo vulnerability
- CVE-2022-0847 — Dirty Pipe
- Hackviser — Glitch Challenge

---

👩‍💻 Write-up Author

V4smin4

Cybersecurity • CTF • Web Security • Linux
