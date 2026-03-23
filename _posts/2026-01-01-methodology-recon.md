---
title: "01 — Recon & Enumeration"
date: 2026-01-11
categories:
  - Tools & Methodologies
  - Methodology
tags: [Methodology, Recon, nmap, ffuf, SMB, LDAP, DNS, Enumeration]
description: "Recon phase methodology — full checklist, nmap workflow, web fuzzing, SMB/DNS/LDAP enumeration commands with annotated explanations."
---

> 📁 **Source:** [github.com/sireton/pentest-vault/01-Recon](https://github.com/sireton/pentest-vault)  
> ← [Back to Methodology Index](/posts/methodology-index/)

---

## Checklist

Before moving to the Foothold phase, confirm you have:

- [ ] Host discovery / ping sweep complete
- [ ] Full port scan (all 65535 ports)
- [ ] Service/version detection on all open ports
- [ ] OS fingerprinting
- [ ] Web tech fingerprinting (if ports 80/443/8080 open)
- [ ] DNS enumeration (if DNS/domain info available)
- [ ] SMB/NFS enumeration (if ports 139/445/2049 open)
- [ ] SNMP enumeration (if port 161 open)
- [ ] LDAP enumeration (if port 389/636 open)
- [ ] Subdomain/vhost fuzzing (if hostname known)

The goal of recon is to build a complete picture of the attack surface *before* attempting to exploit anything. Rushing to exploitation with an incomplete port scan is how you miss findings.

---

## Nmap Workflow

### 1. Fast All-Port Sweep
```bash
nmap -p- --min-rate 5000 -oA allports <IP>
```
`--min-rate 5000` speeds up the scan significantly. On stable networks (lab environments) this is fine; on unstable or production networks, lower to `--min-rate 1000` to reduce packet loss.

### 2. Deep Service Scan on Open Ports
```bash
# Extract ports from allports scan, then run targeted
PORTS=$(grep "^[0-9]" allports.nmap | cut -d'/' -f1 | tr '\n' ',' | sed 's/,$//')
nmap -p $PORTS -sC -sV -oA targeted <IP>
```
`-sC` runs default scripts (often reveals software versions, authentication methods, SMB info). `-sV` probes service versions. `-oA` saves in all formats — the `.xml` output is useful if you want to parse it later.

### 3. UDP Scan (Top 100)
```bash
sudo nmap -sU --top-ports 100 <IP>
```
Often skipped and often where findings hide — SNMP (161), TFTP (69), and others are UDP. Requires root/sudo.

### 4. Vulnerability Scripts
```bash
nmap -p <ports> --script vuln <IP>
```
Useful for quick CVE checks but noisy and slow. Run after you've identified services you want to investigate further.

---

## Web Fuzzing (FFUF)

### Directory Fuzzing
```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt \
     -u http://<IP>/FUZZ \
     -mc 200,204,301,302,307,401,403
```

### Extension Fuzzing
```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt \
     -u http://<IP>/FUZZ \
     -e .php,.txt,.bak,.html,.config
```
Extension fuzzing often finds backup files (`.bak`, `.old`), config files, and source code that wasn't meant to be accessible.

### Vhost Fuzzing
```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
     -u http://<domain>/ \
     -H "Host: FUZZ.<domain>" \
     -fs <default_response_size>
```
The `-fs` filter removes responses matching the default size (what you get for an invalid vhost). Check the size of an initial request first.

### Parameter Fuzzing (GET)
```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
     -u "http://<IP>/page.php?FUZZ=value" \
     -fs <default_size>
```

---

## SMB Enumeration

```bash
# List shares (unauthenticated)
smbclient -N -L //<IP>

# Full enumeration
enum4linux-ng -A <IP>

# CrackMapExec share enum
crackmapexec smb <IP> --shares

# Nmap SMB scripts
nmap -p 445 --script smb-enum-shares,smb-enum-users <IP>
```

**What to look for:** Anonymous/guest read access on shares, `IPC$` and `ADMIN$` availability, usernames from user enumeration, domain/workgroup name.

---

## DNS Enumeration

```bash
# Zone transfer attempt (often fails on hardened DNS, but worth trying)
dig axfr @<IP> <domain>

# Subdomain brute force
gobuster dns -d <domain> -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Reverse lookup
dig -x <IP> @<nameserver>
```

---

## SNMP Enumeration

```bash
# Community string brute force
onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt <IP>

# Full walk (once community string found)
snmpwalk -v2c -c <community> <IP>
```
SNMP v1/v2c with default community strings (`public`, `private`) can expose system information, running processes, network interfaces, and installed software.

---

## LDAP Enumeration

```bash
# Anonymous bind
ldapsearch -x -H ldap://<IP> -b "DC=<domain>,DC=<tld>"

# Authenticated
ldapsearch -x -H ldap://<IP> -D "user@domain.com" -w password -b "DC=domain,DC=com"
```

LDAP enumeration against a domain controller with an anonymous bind (if allowed) can return user accounts, group memberships, computer objects, and sometimes credentials in description fields — a common misconfiguration.

---

## Wordlists Quick Reference

```
# Directories
/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt

# Files
/usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt

# DNS/Subdomains
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Passwords
/usr/share/wordlists/rockyou.txt
```

---

*Next phase: [Foothold →](/posts/methodology-foothold/)*  
*See also: [Web Attacks Methodology](/posts/methodology-web-attacks/) · [AD Methodology](/posts/methodology-active-directory/)*
