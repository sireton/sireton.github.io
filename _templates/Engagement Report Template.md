---
title: "<Target Name> – Penetration Test Report"
date: <YYYY-MM-DD>
categories:
  - Pentest Reports
tags: [<OS>, <technique1>, <technique2>]
description: "Penetration test report for <target>. Findings include <brief summary>."
image:
  path: /assets/img/<target-name>/report-card.png
  alt: "Pentest Report – <Target Name>"
---

<div style="border-left: 4px solid #e74c3c; padding: 0.5rem 1rem; background: rgba(231,76,60,0.05); margin-bottom: 2rem;">
<strong>Classification:</strong> Portfolio Demonstration &nbsp;|&nbsp;
<strong>Author:</strong> Sam Ireton &nbsp;|&nbsp;
<strong>Date:</strong> YYYY-MM-DD &nbsp;|&nbsp;
<strong>Version:</strong> 1.0
</div>

---

## Report Information

| Field | Detail |
|---|---|
| **Target** | `<hostname / IP>` |
| **Platform** | `<OS and version>` |
| **Assessment Type** | Black Box / Grey Box / White Box |
| **Assessment Date** | YYYY-MM-DD |
| **Report Author** | Sam Ireton |
| **Related Walkthrough** | [Technical Write-Up](/posts/<machine-name>-Write-Up/) |

---

## 1. Executive Summary

A penetration test was conducted against **<target>** to evaluate exposure to real-world attack techniques. The assessment identified **X critical**, **X high**, **X medium**, and **X low** severity findings.

When chained together, these findings allowed [worst-case outcome in plain language].

### Risk Summary

| Severity | Count |
|---|---|
| 🔴 Critical | X |
| 🟠 High | X |
| 🟡 Medium | X |
| 🟢 Low | X |
| ℹ️ Informational | X |

---

## 2. Scope & Rules of Engagement

### In Scope
- `<IP/hostname>` — <description>

### Out of Scope
- Denial of service testing
- Social engineering

### Assumptions
- <e.g., No credentials provided — black box>

---

## 3. Methodology

| Phase | Description |
|---|---|
| **Reconnaissance** | Passive and active enumeration of exposed services |
| **Vulnerability Identification** | Manual and tool-assisted identification of weaknesses |
| **Exploitation** | Controlled exploitation of confirmed vulnerabilities |
| **Post-Exploitation** | Assessment of access scope and credential exposure |
| **Privilege Escalation** | Evaluation of escalation to administrative access |
| **Reporting** | Documentation of findings and remediation guidance |

---

## 4. Service Overview

| Port | Service | Version | Notes |
|---|---|---|---|
| 22 | SSH | | |
| 80 | HTTP | | |

---

## 5. Findings Summary

| ID | Title | Severity | CVSS |
|---|---|---|---|
| F-01 | | 🔴 Critical | X.X |
| F-02 | | 🟠 High | X.X |
| F-03 | | 🟡 Medium | X.X |

---

## 6. Detailed Findings

---

### F-01 — <Finding Title>

| Field | Detail |
|---|---|
| **Severity** | 🔴 Critical |
| **CVSS Score** | X.X |
| **CVSSv3 Vector** | `AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H` |
| **CVE** | CVE-XXXX-XXXX |
| **Affected Component** | `<endpoint / service>` |
| **MITRE ATT&CK** | `TXXXX — Technique Name` |

#### Description
Clear description of the vulnerability and why it is exploitable.

#### Evidence

```bash
# Command demonstrating the issue
```

![Evidence](/assets/img/<target-name>/finding-01.png)
*Figure 1: [Describe what is demonstrated]*

#### Impact
> **Worst case:** [One sentence worst realistic outcome]

#### Remediation
1. [Primary fix]
2. [Secondary hardening]
3. [Detection recommendation]

**References:**
- [CVE / Advisory link]()
- [MITRE ATT&CK — TXXXX](https://attack.mitre.org/techniques/TXXXX/)

---

### F-02 — <Finding Title>

| Field | Detail |
|---|---|
| **Severity** | 🟠 High |
| **CVSS Score** | X.X |
| **CVSSv3 Vector** | `AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:N` |
| **CVE** | N/A |
| **Affected Component** | `<component>` |
| **MITRE ATT&CK** | `TXXXX — Technique Name` |

#### Description

#### Evidence

```bash
# Command
```

![Evidence](/assets/img/<target-name>/finding-02.png)
*Figure 2: [Description]*

#### Impact
> **Worst case:** [One sentence]

#### Remediation
1. [Fix]
2. [Hardening]

---

*(Duplicate finding block as needed)*

---

## 7. Attack Chain Narrative

```
[Starting Position]
      │
      ▼
[F-0X: Description] ── Impact
      │
      ▼
[F-0X: Description] ── Impact
      │
      ▼
[Full Compromise]
```

**Key Observation:** Each finding was moderate in isolation. The combination created a critical risk path — demonstrating the importance of defence-in-depth.

---

## 8. Business Impact

- **Confidentiality:** [Data exposed]
- **Integrity:** [Data/systems modified]
- **Availability:** [Service disruption potential]
- **Compliance:** [GDPR / PCI-DSS / ISO 27001 implications]

---

## 9. Remediation Roadmap

### Immediate *(24–72 hours)*
- [ ] [Critical items]

### Short-Term *(30 days)*
- [ ] [High severity items]

### Long-Term
- [ ] Centralised logging and alerting
- [ ] Periodic penetration testing
- [ ] Privileged access management review

---

## 10. Appendices

### Appendix A — Tools Used

| Tool | Purpose |
|---|---|
| nmap | Port and service enumeration |

### Appendix B — References
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [MITRE ATT&CK](https://attack.mitre.org/)
- [PTES](http://www.pentest-standard.org/)

---

<div style="border-top: 1px solid #444; padding-top: 1rem; margin-top: 2rem; font-size: 0.85rem; color: #888;">
<strong>Disclaimer:</strong> This report was produced against a deliberately vulnerable environment for educational and portfolio demonstration purposes. No unauthorized testing was performed against real-world systems.
</div>
