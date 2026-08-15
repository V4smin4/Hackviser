Glitch — Hackviser Write-up

«Hackviser Warmup Challenge | Web Security | Privilege Escalation»

---

Challenge Information

Field| Details
Platform| Hackviser
Challenge| Glitch
Difficulty| Medium
Category| Web Security / Privilege Escalation
Points| 43
Target OS| Linux

---

Overview

Glitch is a Hackviser warmup challenge focused on identifying an outdated web server, obtaining initial access through a known vulnerability, and investigating the target system for a privilege-escalation opportunity.

The challenge demonstrates how multiple vulnerabilities can be chained together:

Outdated Web Service → Remote Code Execution → Initial Access → Kernel Vulnerability → Privilege Escalation → Root

---

1. Information Gathering

The first step was to enumerate the target and identify exposed services.

Nmap

nmap -sV goldnertech.hv

Results

Port| State| Service| Version
"22/tcp"| Open| SSH| OpenSSH 8.4p1 Debian
"80/tcp"| Open| HTTP| Nostromo 1.9.6

The most interesting service was the HTTP server running:

Nostromo 1.9.6

Because the version was outdated, vulnerability research was performed against it.

---

2. Vulnerability Identification

Research into Nostromo 1.9.6 revealed a known vulnerability:

CVE-2019-16278

Property| Information
Software| Nostromo
Version| 1.9.6
CVE| CVE-2019-16278
Impact| Remote Code Execution
Attack Vector| HTTP

The vulnerability provided a potential path to obtain initial access to the target.

---

3. Initial Access

Metasploit Framework was used to search for an exploit related to the identified Nostromo version.

msfconsole

Then:

search nostromo

The relevant module was:

exploit/multi/http/nostromo_code_exec

The module was configured for the Hackviser lab target and successfully provided command execution.

Initial Access

The obtained shell was running as:

www-data

This confirmed successful initial access with a low-privileged web-server account.

---

4. Post-Exploitation Enumeration

After obtaining the initial shell, the next step was to enumerate the target system.

Check Current User

whoami

Output:

www-data

Check Kernel Information

uname -a

The output revealed a Linux kernel version that required further vulnerability research.

This indicated that the next stage of the challenge was likely local privilege escalation.

---

5. Privilege Escalation

An attempt was made to access the system's password-hash file:

cat /etc/shadow

The operation was denied because the current user did not have sufficient privileges.

Further investigation of the Linux kernel version revealed a known vulnerability:

CVE-2022-0847 — Dirty Pipe

Dirty Pipe is a Linux kernel vulnerability that can allow an unprivileged local user to modify data associated with normally read-only files on affected systems.

Within the Hackviser lab environment, the vulnerable kernel provided the privilege-escalation path.

---

6. Root Access

After completing the privilege-escalation stage, the current user was checked again:

whoami

Output:

root

«Root access successfully obtained.»

This confirmed complete compromise of the target system.

---

7. Attack Chain

The complete attack path can be visualized as follows:

flowchart TD
    A[Nmap Enumeration] --> B[Nostromo 1.9.6]
    B --> C[CVE-2019-16278]
    C --> D[Remote Code Execution]
    D --> E[Initial Access: www-data]
    E --> F[Kernel Enumeration]
    F --> G[CVE-2022-0847]
    G --> H[Dirty Pipe]
    H --> I[Privilege Escalation]
    I --> J[Root Access]

---

8. Attack Summary

Stage| Technique| Result
1| Service Enumeration| HTTP and SSH identified
2| Version Detection| Nostromo 1.9.6 identified
3| Vulnerability Research| CVE-2019-16278 identified
4| Initial Access| "www-data" shell obtained
5| System Enumeration| Vulnerable kernel identified
6| Vulnerability Research| CVE-2022-0847 identified
7| Privilege Escalation| Dirty Pipe used in the lab
8| Verification| "root" access obtained

---

9. Impact

The attack chain resulted in full system compromise.

An attacker was able to move from an exposed vulnerable web service to a low-privileged shell and then escalate privileges to "root".

Attack Path

Exposed Web Service
        ↓
Nostromo 1.9.6
        ↓
CVE-2019-16278
        ↓
Remote Code Execution
        ↓
www-data
        ↓
Vulnerable Linux Kernel
        ↓
CVE-2022-0847
        ↓
Privilege Escalation
        ↓
root

Once root privileges are obtained, the attacker has administrative control over the operating system and its accessible resources.

---

10. Defensive Recommendations

Keep Software Updated

Upgrade outdated web servers and remove unsupported versions from production environments.

Patch Linux Kernels

Apply security updates regularly to protect against known kernel vulnerabilities.

Minimize Attack Surface

Only expose services that are required and restrict unnecessary network access.

Apply Least Privilege

Web applications and services should run with only the permissions they actually require.

Continuous Vulnerability Management

Regular vulnerability scanning and patch management can help identify vulnerable services before they are exploited.

---

11. Key Takeaways

- Service enumeration is an essential first step during a security assessment.
- Software version information can reveal potential attack vectors.
- Initial access does not necessarily provide administrative privileges.
- Local system enumeration can reveal privilege-escalation opportunities.
- Kernel vulnerabilities can turn a low-privileged foothold into full system compromise.
- Keeping operating systems and exposed services patched is critical.

---

12. Tools Used

Tool| Purpose
Nmap| Service and version enumeration
Metasploit Framework| Vulnerability research and lab exploitation
Linux CLI| System and privilege enumeration
CVE Research| Vulnerability identification

---

13. Conclusion

The Glitch challenge demonstrated a realistic cybersecurity attack chain beginning with reconnaissance and service enumeration, followed by exploitation of an outdated web service and finally local privilege escalation through a vulnerable Linux kernel.

The main lesson is that vulnerabilities should not be viewed in isolation.

A vulnerable exposed service can provide the initial foothold, while an unpatched operating system can turn that foothold into complete system compromise.

---

References

- CVE-2019-16278 — Nostromo vulnerability
- CVE-2022-0847 — Dirty Pipe
- Hackviser — Glitch Challenge

---

Author

V4smin4

"Cybersecurity • CTF • Web Security • Linux"
