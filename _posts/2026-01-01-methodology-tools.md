---
title: "Reference — Tools Master List"
date: 2026-01-01
categories:
  - Tools & Methodologies
  - Reference
tags: [Methodology, Tools, nmap, BloodHound, impacket, hashcat, Reference]
description: "Full tool reference organized by phase — recon, web, exploitation, post-exploitation, password attacks — with purpose and command references."
---

> 📁 **Source:** [github.com/sireton/pentest-vault/00-Meta](https://github.com/sireton/pentest-vault)  
> ← [Back to Methodology Index](/posts/methodology-index/)

---

#meta #tools

## Recon & Enumeration
| Tool | Purpose | Command Ref |
|---|---|---|
| nmap | Port scanning | [[01-Recon/Recon Index]] |
| masscan | Fast port sweep | `masscan -p1-65535 <IP> --rate=1000` |
| ffuf | Directory/vhost fuzzing | [[01-Recon/Recon Index]] |
| gobuster | Dir/DNS brute force | `gobuster dir -u <url> -w <wordlist>` |
| amass | Subdomain enumeration | `amass enum -d <domain>` |
| theHarvester | OSINT email/subdomain | `theHarvester -d <domain> -b all` |
| enum4linux-ng | SMB/LDAP enumeration | [[01-Recon/Recon Index]] |

## Web Attacks
| Tool | Purpose | Command Ref |
|---|---|---|
| Burp Suite | Web proxy / scanner | [[06-Web-Attacks/Web Attacks Index]] |
| sqlmap | SQL injection automation | [[06-Web-Attacks/SQLi]] |
| nikto | Web server scanner | `nikto -h <url>` |
| wfuzz | Web fuzzer | `wfuzz -c -w <wordlist> <url>/FUZZ` |
| whatweb | Web tech fingerprint | `whatweb <url>` |

## Exploitation
| Tool | Purpose | Command Ref |
|---|---|---|
| metasploit | Exploitation framework | `msfconsole` |
| searchsploit | Exploit search | `searchsploit <service> <version>` |
| evil-winrm | WinRM shell | `evil-winrm -i <IP> -u <user> -p <pass>` |
| impacket | Windows/AD attacks | [[05-Active-Directory/AD Index]] |
| crackmapexec | SMB/AD Swiss Army knife | [[05-Active-Directory/AD Index]] |

## Post-Exploitation
| Tool | Purpose | Command Ref |
|---|---|---|
| linpeas / winpeas | Privilege esc enumeration | [[03-Post-Exploitation/Post-Exploitation Index]] |
| BloodHound | AD attack path mapping | [[05-Active-Directory/BloodHound]] |
| mimikatz | Credential dumping | [[05-Active-Directory/BloodHound]] |
| rubeus | Kerberos attacks | [[05-Active-Directory/Kerberos Attacks]] |
| chisel | Tunneling/pivoting | [[04-Lateral-Movement/Lateral Movement Index]] |
| ligolo-ng | Layer 3 pivoting | [[04-Lateral-Movement/Lateral Movement Index]] |

## Password Attacks
| Tool | Purpose | Command Ref |
|---|---|---|
| hashcat | Hash cracking (GPU) | `hashcat -m <mode> <hash> <wordlist>` |
| john | Hash cracking (CPU) | `john --wordlist=<wl> <hash>` |
| hydra | Online brute force | `hydra -l <user> -P <wl> <IP> <service>` |
| kerbrute | Kerberos user enum | `kerbrute userenum -d <domain> <wordlist>` |


---

*Previous: [Methodology Reporting](/posts/methodology-reporting/) · Next: [Methodology File Transfers](/posts/methodology-file-transfers/)*
