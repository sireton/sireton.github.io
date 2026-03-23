---
title: "07 — Pillaging"
date: 2025-12-05
categories:
  - Tools & Methodologies
  - Methodology
tags: [Methodology, Pillaging, Credential Hunting, SSH Keys, Browser Passwords, AWS]
description: "Pillaging methodology — Linux and Windows credential hunting, browser data, shell history, config files, SSH keys, cloud credentials."
---

> 📁 **Source:** [github.com/sireton/pentest-vault/07-Pillaging](https://github.com/sireton/pentest-vault)  
> ← [Back to Methodology Index](/posts/methodology-index/)

---

#pillaging #phase

## Checklist
- [ ] Browser saved passwords & history
- [ ] SSH keys
- [ ] Config files (web apps, databases)
- [ ] Shell history (.bash_history / PowerShell)
- [ ] Password managers (KeePass .kdbx)
- [ ] Scripts with embedded credentials
- [ ] Cloud credentials (AWS, Azure, GCP)
- [ ] Wi-Fi passwords

---

## Linux
```bash
find / -name "id_rsa" 2>/dev/null
find / -name "*.pem" 2>/dev/null
cat ~/.bash_history; cat ~/.zsh_history
grep -r "password" /var/www/html/ 2>/dev/null
cat /var/www/html/wp-config.php
cat ~/.aws/credentials
find / -name "*.kdbx" 2>/dev/null
```

## Windows
```powershell
type $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
cmdkey /list
netsh wlan show profile name="<ssid>" key=clear
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
findstr /spin "password" C:\*.txt C:\*.xml C:\*.ini C:\*.config 2>nul
.\lazagne.exe all
```


---

*Previous: [Methodology Web Attacks](/posts/methodology-web-attacks/) · Next: [Methodology Reporting](/posts/methodology-reporting/)*
