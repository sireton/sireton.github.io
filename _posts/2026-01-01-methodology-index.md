---
title: "Methodology — Index & Overview"
date: 2026-01-01
categories:
  - Tools & Methodologies
  - Methodology
tags: [Methodology, CPTS, Pentest, Reference]
description: "A living methodology reference built around the HTB CPTS curriculum. Covers the full engagement lifecycle from recon through reporting."
---

This is a living reference document built from my pentest methodology vault — a structured set of notes I maintain and update after every engagement or training module. It's organized around the HTB CPTS curriculum but draws from real engagement experience and continuous research.

Each phase below links to a dedicated post with full command references, checklists, and annotated examples.

> 📁 **Full vault source:** [github.com/sireton/pentest-vault](https://github.com/sireton/pentest-vault)  
> The GitHub repo mirrors this structure and is kept in sync with updates.

---

## Engagement Phases

| Phase | Description | Post |
|---|---|---|
| 🔍 Recon & Enumeration | Service discovery, web fingerprinting, SMB/LDAP/DNS enum | [View →](/posts/methodology-recon/) |
| 🚪 Foothold | Service exploitation, CVE workflow, web-based entry | [View →](/posts/methodology-foothold/) |
| 🖥️ Post-Exploitation | Situational awareness, privesc vectors, credential access | [View →](/posts/methodology-post-exploitation/) |
| ↔️ Lateral Movement | Pivoting, tunneling, pass-the-hash, remote execution | [View →](/posts/methodology-lateral-movement/) |
| 🏰 Active Directory | AD attack chain, Kerberos attacks, BloodHound, DCSync | [View →](/posts/methodology-active-directory/) |
| 🌐 Web Attacks | SQLi, LFI, file upload, SSRF, XXE | [View →](/posts/methodology-web-attacks/) |
| 💰 Pillaging | Credential hunting, sensitive file discovery | [View →](/posts/methodology-pillaging/) |
| 📝 Reporting | Finding documentation, CVSS, evidence capture | [View →](/posts/methodology-reporting/) |

---

## Meta / Reference

| Resource | Description | Post |
|---|---|---|
| 🧰 Tools Master List | Full tool index with purpose and command references | [View →](/posts/methodology-tools/) |
| 📦 File Transfer Methods | Linux/Windows transfer techniques for every scenario | [View →](/posts/methodology-file-transfers/) |
| 📚 Wordlists & Resources | Essential wordlists, online resources, hashcat modes | [View →](/posts/methodology-resources/) |

---

## Templates

The engagement and reporting templates I use for every write-up on this blog are also available in the repo:

- `Engagement Report Template` — Full pentest report with CVSS findings, executive summary, remediation roadmap
- `HTB Walkthrough Template` — Technical write-up structure with attack chain summary
- `ProLab Report Template` — Multi-host lab assessment format
- `Vulnerability Write-Up Template` — CVE/technique deep-dive format

> 📁 [github.com/sireton/pentest-vault/Templates](https://github.com/sireton/pentest-vault)

---

## About This Methodology

This methodology is built around the [HTB CPTS](https://academy.hackthebox.com/preview/certifications/htb-certified-penetration-testing-specialist) curriculum and is actively updated as I work through engagements and labs. It's not intended to be exhaustive — it's the set of techniques and commands I've verified, understand, and actually use.

Each phase document includes:
- A checklist of what to cover before moving to the next phase
- Annotated command blocks with explanations of *why*, not just *what*
- MITRE ATT&CK technique mappings
- Links to relevant case studies on this blog where the technique appeared in practice

---

*Last updated: March 2026 · Based on HTB CPTS methodology*
