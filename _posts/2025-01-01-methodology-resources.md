---
title: "Reference — Wordlists & Resources"
date: 2025-01-01
categories:
  - Tools & Methodologies
  - Reference
tags: [Methodology, Wordlists, SecLists, hashcat, Resources, Reference]
description: "Essential wordlist paths, online resources, and hashcat mode quick reference for pentest engagements."
---

> 📁 **Source:** [github.com/sireton/pentest-vault/00-Meta](https://github.com/sireton/pentest-vault)  
> ← [Back to Methodology Index](/posts/methodology-index/)

---

#meta #wordlists

## Essential Wordlists (SecLists)

### Directory/File Fuzzing
```
/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
/usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt
/usr/share/seclists/Discovery/Web-Content/common.txt
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

### DNS / Subdomains
```
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
/usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
```

### Usernames / Passwords
```
/usr/share/wordlists/rockyou.txt
/usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-10000.txt
/usr/share/seclists/Usernames/Names/names.txt
/usr/share/seclists/Usernames/xato-net-10-million-usernames.txt
```

---

## Online Resources

| Resource | URL | Best For |
|---|---|---|
| HackTricks | https://book.hacktricks.xyz | Everything |
| GTFOBins | https://gtfobins.github.io | SUID/sudo abuse |
| LOLBAS | https://lolbas-project.github.io | Windows LOLBins |
| PayloadsAllTheThings | https://github.com/swisskyrepo/PayloadsAllTheThings | Payloads |
| RevShells | https://www.revshells.com | Shell generator |
| CyberChef | https://gchq.github.io/CyberChef | Encoding/decoding |
| ExploitDB | https://www.exploit-db.com | CVE research |
| IppSec Search | https://ippsec.rocks | HTB technique lookup |
| PortSwigger Academy | https://portswigger.net/web-security | Web attacks |

---

## Hashcat Modes Quick Reference

| Hash Type | Mode |
|---|---|
| NTLM | 1000 |
| NTLMv1 | 5500 |
| NTLMv2 | 5600 |
| Kerberos TGS (Kerberoast) | 13100 |
| Kerberos AS-REP (AS-REP Roast) | 18200 |
| MD5 | 0 |
| SHA1 | 100 |
| SHA256 | 1400 |
| bcrypt | 3200 |
| SHA512crypt (Linux $6$) | 1800 |


---

*Previous: [Methodology File Transfers](/posts/methodology-file-transfers/) · Next: [Methodology Index](/posts/methodology-index/)*
