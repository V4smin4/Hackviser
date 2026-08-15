Glitch — Hackviser Write-up

Challenge Information

- Platform: Hackviser
- Challenge: Glitch
- Difficulty: Medium
- Category: Web Security / Privilege Escalation
- Points: 43

Overview

Glitch is a Hackviser warmup challenge focused on identifying an outdated web server, gaining initial access through a known vulnerability, and then investigating the system for a privilege-escalation opportunity.

The challenge demonstrates how an outdated service combined with an unpatched Linux kernel can lead to full system compromise.

1. Information Gathering

The first step was enumerating the target to identify exposed services.

nmap -sV goldnertech.hv

The scan revealed:

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u2
80/tcp open  http    nostromo 1.9.6

The interesting service was HTTP running Nostromo 1.9.6.

2. Vulnerability Identification

Researching the identified version revealed a known vulnerability affecting Nostromo 1.9.6:

CVE-2019-16278

The vulnerability can allow remote command execution under vulnerable configurations.

The service was therefore investigated as the likely initial attack vector.

3. Initial Access

Metasploit was used to identify an appropriate module for the vulnerable Nostromo version.

msfconsole
search nostromo

The relevant module was:

exploit/multi/http/nostromo_code_exec

After configuring the target and local connection parameters, the exploit successfully provided command execution on the target.

The initial shell was running with the privileges of:

www-data

This provided the foothold required for further enumeration.

4. System Enumeration

After obtaining access, the operating system and kernel information were checked:

uname -a

The system was running a Linux kernel version associated with the Dirty Pipe vulnerability.

The next stage was therefore investigating possible local privilege-escalation opportunities.

5. Privilege Escalation

The system was checked for sensitive files and available privileges.

For example:

cat /etc/shadow

Access was denied while operating as the low-privileged web user.

Further investigation identified the Linux kernel version as potentially vulnerable to:

CVE-2022-0847 — Dirty Pipe

Dirty Pipe is a Linux kernel vulnerability that can allow an unprivileged local user to modify the contents of normally read-only files under vulnerable conditions.

The vulnerability was used in the lab environment to demonstrate privilege escalation.

6. Root Access

After successfully completing the privilege-escalation stage, the current user was verified:

whoami

Result:

root

This confirmed successful escalation from the initial web-server account to full administrative privileges.

7. Attack Chain

The complete attack path can be summarized as:

Nmap Enumeration
       ↓
Nostromo 1.9.6
       ↓
CVE-2019-16278
       ↓
Initial Access as www-data
       ↓
Linux Kernel Enumeration
       ↓
CVE-2022-0847 (Dirty Pipe)
       ↓
Privilege Escalation
       ↓
Root Access

Impact

The attack chain resulted in complete compromise of the target system.

An attacker could move from an exposed vulnerable web service to a low-privileged shell and eventually obtain root-level access through an unpatched kernel vulnerability.

Key Takeaways

- Always identify the exact versions of exposed services.
- Outdated web servers can provide an initial entry point.
- Initial access should always be followed by systematic local enumeration.
- Linux kernel vulnerabilities can create serious privilege-escalation risks.
- Regular patching and vulnerability management are essential for preventing chained attacks.

Tools Used

- Nmap
- Metasploit Framework
- Linux command-line tools
- Vulnerability research

---

Author: V4smin4

Platform: Hackviser
