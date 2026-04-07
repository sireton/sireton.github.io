---
title: "Reference — Tools Master List"
categories: [Tools & Methodologies, Reference]
tags: [methodology, tools, nmap, bloodhound, impacket, hashcat, reference]
---

> 📁 **Source:** [github.com/sireton/pentest-vault](https://github.com/sireton/pentest-vault)

Full tool reference organized by phase: recon, web, exploitation, post-exploitation, and password attacks.

← [Back to Methodology Index](/posts/methodology-index/)

---

## Recon & Enumeration

| Tool | Purpose | Command Ref |
|---|---|---|
| nmap | Port scanning | [Recon Index](/posts/methodology-recon/) |
| masscan | Fast port sweep | `masscan -p1-65535 <IP> --rate=1000` |
| ffuf | Directory/vhost fuzzing | [Recon Index](/posts/methodology-recon/) |
| gobuster | Dir/DNS brute force | `gobuster dir -u <url> -w <wordlist>` |
| amass | Subdomain enumeration | `amass enum -d <domain>` |
| theHarvester | OSINT email/subdomain | `theHarvester -d <domain> -b all` |
| enum4linux-ng | SMB/LDAP enumeration | [Recon Index](/posts/methodology-recon/) |

## Web Attacks

| Tool | Purpose | Command Ref |
|---|---|---|
| Burp Suite | Web proxy / scanner | [Web Attacks Index](/posts/methodology-web-attacks/) |
| sqlmap | SQL injection automation | [Web Attacks Index](/posts/methodology-web-attacks/) |
| nikto | Web server scanner | `nikto -h <url>` |
| wfuzz | Web fuzzer | `wfuzz -c -w <wordlist> <url>/FUZZ` |
| whatweb | Web tech fingerprint | `whatweb <url>` |

## Exploitation

| Tool | Purpose | Command Ref |
|---|---|---|
| metasploit | Exploitation framework | `msfconsole` |
| searchsploit | Exploit search | `searchsploit <service> <version>` |
| evil-winrm | WinRM shell | `evil-winrm -i <IP> -u <user> -p <pass>` |
| impacket | Windows/AD attacks | [AD Index](/posts/methodology-active-directory/) |
| crackmapexec | SMB/AD Swiss Army knife | [AD Index](/posts/methodology-active-directory/) |

## Post-Exploitation

| Tool | Purpose | Command Ref |
|---|---|---|
| linpeas / winpeas | Privilege esc enumeration | [Post-Exploitation Index](/posts/methodology-post-exploitation/) |
| BloodHound | AD attack path mapping | [Active Directory](/posts/methodology-active-directory/) |
| mimikatz | Credential dumping | [Active Directory](/posts/methodology-active-directory/) |
| rubeus | Kerberos attacks | [Active Directory](/posts/methodology-active-directory/) |
| chisel | Tunneling/pivoting | [Lateral Movement Index](/posts/methodology-lateral-movement/) |
| ligolo-ng | Layer 3 pivoting | [Lateral Movement Index](/posts/methodology-lateral-movement/) |

## Password Attacks

| Tool | Purpose | Command Ref |
|---|---|---|
| hashcat | Hash cracking (GPU) | `hashcat -m <mode> <hash> <wordlist>` |
| john | Hash cracking (CPU) | `john --wordlist=<wl> <hash>` |
| hydra | Online brute force | `hydra -l <user> -P <wl> <IP> <service>` |
| kerbrute | Kerberos user enum | `kerbrute userenum -d <domain> <wordlist>` |

---

*Previous: [Methodology Reporting](/posts/methodology-reporting/) · Next: [Methodology File Transfers](/posts/methodology-file-transfers/)*
