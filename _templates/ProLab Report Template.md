---
title: "<ProLab Name> – Lab Assessment Report"
date: <YYYY-MM-DD>
categories:
  - Pentest Reports
  - ProLabs
tags: [HTB, ProLab, <lab-name>, Active Directory, <technique1>]
description: "Assessment report for HTB <ProLab Name>. Full attack path across <X> hosts covering <key techniques>."
image:
  path: /assets/img/<lab-name>/report-card.png
  alt: "<ProLab Name> – HackTheBox Pro Lab Report"
---

<!--
NOTE ON PROLAB PUBLISHING:
- Do NOT publish flags, hashes, or working credentials
- Do NOT publish step-by-step exploitation for active labs
- DO publish methodology, attack chain, and findings
- DO publish after HTB retires the lab
-->

<div style="border-left: 4px solid #3498db; padding: 0.5rem 1rem; background: rgba(52,152,219,0.05); margin-bottom: 2rem;">
<strong>Author:</strong> Sam Ireton &nbsp;|&nbsp;
<strong>Date:</strong> YYYY-MM-DD &nbsp;|&nbsp;
<strong>Lab:</strong> HTB ProLab — <Lab Name>
</div>

---

## Lab Overview

| Field | Detail |
|---|---|
| **Lab Name** | <ProLab Name> |
| **Provider** | HackTheBox |
| **Difficulty** | Beginner / Intermediate / Advanced / Expert |
| **Hosts in Scope** | X |
| **Flags Captured** | X / X |
| **Certificate Earned** | Yes / No |
| **Assessment Period** | YYYY-MM-DD — YYYY-MM-DD |

---

## 1. Executive Summary

<ProLab Name> simulates a [describe scenario — e.g., multi-host corporate AD environment]. The assessment resulted in compromise of **X of X** hosts. The primary attack path involved:

1. [Key entry technique]
2. [Key lateral movement technique]
3. [Final escalation technique]

### Skills Demonstrated

| Domain | Techniques |
|---|---|
| **Reconnaissance** | |
| **Initial Access** | |
| **Lateral Movement** | |
| **Privilege Escalation** | |
| **Active Directory** | |

---

## 2. Network Map

```
[External / Entry Point]
        │
        ▼
[DMZ / Perimeter Hosts]
        │
        ▼
[Internal Network]
   ├── [Host 1 — Role]
   ├── [Host 2 — Role]
   └── [DC-01 — Domain Controller]
```

| Host | Role | OS | Status |
|---|---|---|---|
| Host-01 | | Linux | ✅ Compromised |
| DC-01 | Domain Controller | Windows Server | ✅ Compromised |

---

## 3. Attack Path

### Phase 1 — Initial Access
Describe how the first host was compromised.

### Phase 2 — Internal Enumeration
How was the internal network mapped after foothold?

```bash
# Pivoting command used
```

### Phase 3 — Lateral Movement
How was access extended? What credential technique was used?

### Phase 4 — Domain Compromise
Path to Domain Admin. BloodHound paths identified. Final technique used.

---

## 4. Key Findings

### F-01 — <Finding Title>

| Field | Detail |
|---|---|
| **Severity** | 🔴 Critical |
| **Host(s)** | |
| **Category** | [e.g., Weak Credentials / Misconfiguration] |
| **MITRE ATT&CK** | `TXXXX — Technique` |

**Description:**

**Impact:**

**Remediation:**

---

*(Duplicate finding block as needed)*

---

## 5. Credentials Captured

> ⚠️ All values redacted — never publish real hashes or flags.

| Host | Account | Type | Method |
|---|---|---|---|
| Host-01 | [REDACTED] | [REDACTED] | Secretsdump |

---

## 6. Tools & Techniques

| Tool | Purpose |
|---|---|
| nmap | Network enumeration |
| BloodHound | AD attack path mapping |
| impacket | Windows/AD exploitation |
| crackmapexec | SMB/AD enumeration |
| ligolo-ng | Pivoting |

---

## 7. Key Learnings

### What Worked Well
- [Effective approach]

### What I'd Do Differently
- [Honest reflection — shows analytical maturity]

### Real-World Applicability
How do these techniques map to real enterprise environments and why should defenders care?

---

## 8. Certificate

![<ProLab Name> Certificate](/assets/img/<lab-name>/certificate.png)
*Completion certificate — <ProLab Name> Pro Lab*

---

## References

- [HTB ProLab Page](https://app.hackthebox.com/prolabs)
- [BloodHound Documentation](https://bloodhound.readthedocs.io/)
- [MITRE ATT&CK](https://attack.mitre.org/)
- [Impacket Suite](https://github.com/fortra/impacket)

---

<div style="border-top: 1px solid #444; padding-top: 1rem; margin-top: 2rem; font-size: 0.85rem; color: #888;">
<strong>Disclaimer:</strong> This report documents activity within the HackTheBox Pro Lab environment. No unauthorized testing was conducted. Flags and credentials have been redacted per HTB guidelines.
</div>
