---
title: "<Machine Name> – <One Line Describing Attack Path>"
date: <YYYY-MM-DD>
categories:
  - HTB - Write Ups
tags: [HTB, <OS>, <Difficulty>, <technique1>, <technique2>, <tool1>]
description: "<One sentence summary. Describe the full attack chain briefly.>"
image:
  path: /assets/img/<machine-name>/card.png
  alt: "<Machine Name> – HackTheBox"
---

## Overview

> **Difficulty:** `Easy / Medium / Hard / Insane` &nbsp;|&nbsp; **OS:** `Linux / Windows` &nbsp;|&nbsp; **Release:** `YYYY-MM-DD`

Brief 2–3 sentence description of the machine. What real-world scenario does it simulate?

In addition to this walkthrough, a formal engagement report was produced documenting findings and remediation guidance.

📄 **[View the Engagement Report](/posts/<machine-name>-Engagement-Report/)**

---

## Attack Chain Summary

```
[Recon] → [describe] → [describe] → [Root/SYSTEM]
```

| Phase | Vector | Result |
|---|---|---|
| Initial Access | | |
| Privilege Escalation | | |

---

## Tools & Techniques

| Tool | Purpose |
|---|---|
| `nmap` | Service and version enumeration |
| | |

---

## 1. Reconnaissance & Enumeration

### Port Scan

```bash
nmap -p- --min-rate 5000 -oA allports <IP>
nmap -p <ports> -sC -sV -oA targeted <IP>
```

![Nmap scan results](/assets/img/<machine-name>/nmap.png)
*Figure 1: Service enumeration results*

**Key findings:**
- Port `XX` — Service/version — Notable observation

### Web Enumeration *(if applicable)*

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -u http://<IP>/FUZZ
```

---

## 2. Foothold

### Vulnerability Identified

Describe the vulnerability. Explain *why* it exists — what should have been done differently defensively?

```bash
# Command used
```

![Evidence screenshot](/assets/img/<machine-name>/foothold.png)
*Figure 2: [Describe what the screenshot shows]*

### Exploitation

Walk through the exploitation steps. Explain each command's purpose.

```bash
# Step 1 — reason
command here

# Step 2 — reason
command here
```

---

## 3. Post-Exploitation & Privilege Escalation

### Local Enumeration

```bash
id && whoami && hostname
sudo -l
find / -perm -4000 2>/dev/null
```

### Privilege Escalation Vector

Describe the privesc path and root cause. What MITRE technique does this map to?

> **MITRE ATT&CK:** `T1XXX — Technique Name`

```bash
# PrivEsc commands
```

![Privilege escalation](/assets/img/<machine-name>/privesc.png)
*Figure 3: [Describe the escalation evidence]*

### Root / SYSTEM Access

```bash
whoami && id && hostname && cat /root/root.txt
```

![Root access](/assets/img/<machine-name>/root.png)
*Figure 4: Root shell obtained*

---

## 4. Key Takeaways

1. **[Lesson 1]** — Why this matters in real engagements
2. **[Lesson 2]** — Explanation
3. **[Lesson 3]** — Explanation

---

## 5. Remediation (Brief)

| Finding | Recommendation |
|---|---|
| | |

*Full remediation detail in the [Engagement Report](/posts/<machine-name>-Engagement-Report/).*

---

## References

- [HTB Machine Page](https://app.hackthebox.com/machines/<n>)
- [CVE-XXXX-XXXX](https://nvd.nist.gov/vuln/detail/CVE-XXXX-XXXX)

---

*This write-up is based on a retired Hack The Box machine and is intended solely for educational and professional demonstration purposes.*
