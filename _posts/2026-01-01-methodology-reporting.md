---
title: "08 — Reporting"
date: 2026-01-04
categories:
  - Tools & Methodologies
  - Methodology
tags: [Methodology, Reporting, CVSS, Evidence Capture, Pentest Report]
description: "Reporting methodology — CVSS scoring reference, evidence capture workflow, finding documentation checklist, and report template links."
---

> 📁 **Source:** [github.com/sireton/pentest-vault/08-Reporting](https://github.com/sireton/pentest-vault)  
> ← [Back to Methodology Index](/posts/methodology-index/)

---

#reporting #phase

## Checklist
- [ ] Executive Summary written (non-technical, business impact)
- [ ] All findings documented with CVSS scores
- [ ] Evidence captured (screenshots, command output)
- [ ] Reproduction steps written
- [ ] Remediation recommendations for every finding
- [ ] Attack chain narrative written
- [ ] Appendices: tools used, scope, methodology

---

## CVSS Quick Reference

| Score | Severity |
|---|---|
| 9.0 – 10.0 | 🔴 Critical |
| 7.0 – 8.9 | 🟠 High |
| 4.0 – 6.9 | 🟡 Medium |
| 0.1 – 3.9 | 🟢 Low |
| 0.0 | ℹ️ Informational |

---

## Evidence Capture
```bash
{ date; whoami; hostname; id; } | tee evidence_$(date +%Y%m%d_%H%M%S).txt
```

---

## Templates
- [[Templates/HTB Walkthrough Template]]
- [[Templates/Engagement Report Template]]
- [[Templates/ProLab Report Template]]
- [[Templates/Vulnerability Write-Up Template]]


---

*Previous: [Methodology Pillaging](/posts/methodology-pillaging/) · Next: [Methodology Tools](/posts/methodology-tools/)*
