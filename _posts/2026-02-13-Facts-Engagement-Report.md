---
title: "Facts – Engagement Report"
date: 2026-02-13
categories:
  - Pentest Reports
tags: [HTB, Linux, Medium, Web Exploitation, Mass Assignment, CVE-2024-46987, Privilege Escalation, MinIO, SSH, Credential Exposure]
description: "Penetration test report for HTB Facts. Full-chain root compromise via Camaleon CMS mass assignment (CVE-2024-46987), exposed MinIO credentials, SSH key cracking, and sudo facter misconfiguration."

image:
![Facts – Engagement Report](/assets/img/facts/factslogo.png){: width="300" 

---

<div style="border-left: 4px solid #e74c3c; padding: 0.5rem 1rem; background: rgba(231,76,60,0.05); margin-bottom: 2rem;">
<strong>Classification:</strong> Portfolio Demonstration &nbsp;|&nbsp;
<strong>Author:</strong> Sam Ireton &nbsp;|&nbsp;
<strong>Date:</strong> 2026-02-13 &nbsp;|&nbsp;
<strong>Version:</strong> 1.0
</div>

---

## Report Information

| Field | Detail |
|---|---|
| **Target** | `10.129.8.32` (Facts) |
| **Platform** | Linux — Ubuntu 25.04, nginx 1.26.3, Camaleon CMS v2.9.0 |
| **Assessment Type** | Black Box — No credentials provided |
| **Engagement Scope** | SSH (22), HTTP (80), MinIO (54321) |
| **Assessment Date** | 2026-02-13 |
| **Report Author** | Sam Ireton |

---

## 1. Executive Summary

A black box penetration test was conducted against a Linux host running Camaleon CMS v2.9.0 and an exposed MinIO object storage service. The assessment simulated an unauthenticated external attacker.

The assessment identified **3 Critical** and **1 High** severity findings. The engagement demonstrated a complete five-step exploitation chain progressing from low-privileged web user to root, without requiring any vulnerability in the underlying operating system.

The root cause was a combination of a known CVE in the CMS access control layer, credentials exposed in the administrative interface, insecure storage of SSH private keys in object storage, and an overly permissive sudo configuration.

### Risk Summary

| Severity | Count |
|---|---|
| 🔴 Critical | 3 |
| 🟠 High | 1 |
| 🟡 Medium | 0 |
| 🟢 Low | 0 |
| ℹ️ Informational | 0 |

---

## 2. Scope & Rules of Engagement

### In Scope
- `10.129.8.32` — All exposed services
- Port `22` — SSH
- Port `80` — HTTP / Camaleon CMS
- Port `54321` — MinIO object storage

### Out of Scope
- Denial of service testing
- Excessive brute-force attempts

### Assumptions
- No credentials provided — black box assessment
- Testing conducted from an external network position

---

## 3. Methodology

| Phase | Description |
|---|---|
| **Reconnaissance** | Nmap service scan, web application fingerprinting, CMS version identification |
| **Vulnerability Identification** | CVE research for Camaleon CMS v2.9.0, mass assignment testing, admin panel enumeration |
| **Exploitation** | Mass assignment privilege escalation to admin, MinIO credential extraction |
| **Post-Exploitation** | MinIO bucket access, SSH key retrieval and offline cracking |
| **Privilege Escalation** | sudo misconfiguration abuse via facter `--custom-dir` |
| **Reporting** | Documentation of findings, risk ratings, and remediation guidance |

---

## 4. Service Overview

| Port | Protocol | Service | Version | Notes |
|---|---|---|---|---|
| 22 | TCP | SSH | OpenSSH 9.9p1 | Remote access |
| 80 | TCP | HTTP | nginx 1.26.3 | Camaleon CMS v2.9.0 (Rails) |
| 54321 | TCP | HTTP | MinIO | S3-compatible object storage |

---

## 5. Findings Summary

| ID | Title | Severity | CVSS |
|---|---|---|---|
| F-01 | Mass Assignment in Camaleon CMS (CVE-2024-46987) | 🟠 High | 8.1 |
| F-02 | MinIO Credentials Exposed in Admin Panel | 🔴 Critical | 9.1 |
| F-03 | SSH Private Key Stored in Object Storage | 🔴 Critical | 9.1 |
| F-04 | Sudo Misconfiguration — Arbitrary Root Code Execution via facter | 🔴 Critical | 9.3 |

---

## 6. Detailed Findings

---

### F-01 — Mass Assignment Vulnerability in Camaleon CMS (CVE-2024-46987)

| Field | Detail |
|---|---|
| **Severity** | 🟠 High |
| **CVSS Score** | 8.1 |
| **CVSSv3 Vector** | `AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:N` |
| **CVE** | [CVE-2024-46987](https://nvd.nist.gov/vuln/detail/CVE-2024-46987) |
| **Affected Component** | `/admin/users/<id>/updated_ajax` — Camaleon CMS ≤ 2.9.0 |
| **MITRE ATT&CK** | `T1548 — Abuse Elevation Control Mechanism` |

#### Description

Camaleon CMS versions up to and including 2.9.0 contain a mass assignment vulnerability in the user profile update endpoint. The Rails controller responsible for handling profile updates does not implement strict strong parameter filtering, allowing authenticated users to inject arbitrary model attributes into the update request.

By intercepting a legitimate profile update request in Burp Suite and appending `&password%5Brole%5D=admin` to the request body, a low-privileged account was escalated to administrator without requiring knowledge of any existing admin credentials.

#### Evidence

![Mass Assignment Exploit — Burp Request](/assets/img/facts/mass-assignment.png)
*Figure 1: Intercepted profile update request with injected role=admin parameter*

![Mass Assignment Result — Admin Panel](/assets/img/facts/mass-assignment2.png)
*Figure 2: Successful privilege escalation — administrative panel access confirmed*

#### Impact

Any registered user can escalate to administrator, gaining full CMS control including access to configuration panels, credentials, and server-side functionality.

> **Worst case:** Any authenticated user achieves administrative access, enabling the credential exposure chain documented in F-02 and F-03.

#### Remediation

1. Patch Camaleon CMS to a version after 2.9.0 that addresses CVE-2024-46987
2. Implement strict Rails strong parameter filtering — explicitly permit only safe attributes in the `users_params` method
3. Add server-side role validation — role changes should require admin-level authentication regardless of request parameters
4. Implement automated testing for mass assignment vulnerabilities in the CI/CD pipeline

**References:**
- [CVE-2024-46987 — NVD](https://nvd.nist.gov/vuln/detail/CVE-2024-46987)
- [OWASP — Mass Assignment](https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/)

---

### F-02 — MinIO Credentials Exposed in Administrative Panel

| Field | Detail |
|---|---|
| **Severity** | 🔴 Critical |
| **CVSS Score** | 9.1 |
| **CVSSv3 Vector** | `AV:N/AC:L/PR:H/UI:N/S:C/C:H/I:H/A:N` |
| **CVE** | N/A |
| **Affected Component** | Camaleon CMS Admin → Settings → Filesystem Settings |
| **MITRE ATT&CK** | `T1552.001 — Credentials in Files` · `T1530 — Data from Cloud Storage` |

#### Description

The Camaleon CMS administrative settings panel (accessible after F-01 privilege escalation) exposed active MinIO object storage credentials in plaintext within the Filesystem Settings page. The following values were directly readable:

- AWS S3 Access Key
- AWS S3 Secret Key
- Bucket name (`randomfacts`)
- Internal endpoint (`http://localhost:54321`)

These credentials were used to authenticate to the MinIO service running on port 54321, providing full access to all stored objects.

#### Evidence

![MinIO Credentials in Admin Panel](/assets/img/facts/minio-credentials.png)
*Figure 3: Active MinIO credentials exposed in plaintext within CMS admin panel*

#### Impact

Full authenticated access to the MinIO object storage service, exposing all stored files including the SSH private keys documented in F-03.

> **Worst case:** Complete exfiltration of all stored objects, enabling the SSH authentication bypass and lateral movement chain.

#### Remediation

1. Remove credentials from all web-accessible configuration panels — use environment variables or a secrets manager (HashiCorp Vault, AWS Secrets Manager)
2. Rotate all exposed MinIO access and secret keys immediately
3. Restrict MinIO to localhost or internal network only — port 54321 should not be externally accessible
4. Audit all admin panel pages for plaintext credential exposure

---

### F-03 — SSH Private Key Stored in Object Storage Bucket

| Field | Detail |
|---|---|
| **Severity** | 🔴 Critical |
| **CVSS Score** | 9.1 |
| **CVSSv3 Vector** | `AV:N/AC:L/PR:L/UI:N/S:C/C:H/I:H/A:N` |
| **CVE** | N/A |
| **Affected Component** | MinIO `internal` bucket — `.ssh/` directory |
| **MITRE ATT&CK** | `T1552.004 — Credentials in Files: Private Keys` · `T1078 — Valid Accounts` |

#### Description

The MinIO `internal` bucket contained a `.ssh/` directory storing both an `id_ed25519` private key and the corresponding `authorized_keys` file. The private key was protected with a passphrase, however this passphrase was weak and crackable using standard wordlist techniques.

Successful SSH authentication was obtained as user `william` using the recovered key, providing a persistent interactive shell.

#### Evidence

![SSH Access as william](/assets/img/facts/ssh-access.png)
*Figure 4: Successful SSH authentication as user william using recovered private key*

#### Impact

Persistent authenticated SSH shell access as user `william`, enabling interactive system access, further enumeration, and exploitation of the privilege escalation path documented in F-04.

> **Worst case:** Persistent remote system access as a named user account, with a direct path to root via F-04.

#### Remediation

1. Remove all SSH private keys from object storage immediately
2. Rotate the exposed SSH key pair — revoke the existing `authorized_keys` entry
3. Store SSH private keys only on the systems that require them, with appropriate file permissions (`chmod 600`)
4. Enforce strong passphrases on all SSH private keys
5. Implement object storage access policies — the `.ssh/` directory should never be accessible via storage APIs

---

### F-04 — Sudo Misconfiguration Enabling Arbitrary Root Code Execution

| Field | Detail |
|---|---|
| **Severity** | 🔴 Critical |
| **CVSS Score** | 9.3 |
| **CVSSv3 Vector** | `AV:L/AC:L/PR:L/UI:N/S:C/C:H/I:H/A:H` |
| **CVE** | N/A |
| **Affected Component** | `/etc/sudoers` — `(ALL) NOPASSWD: /usr/bin/facter` |
| **MITRE ATT&CK** | `T1548.003 — Sudo and Sudo Caching` |

#### Description

The `william` user account was configured with passwordless sudo execution of `/usr/bin/facter`. The `facter` tool supports loading custom Ruby fact files from a user-specified directory via the `--custom-dir` flag.

By creating a malicious Ruby file containing a system command and passing its parent directory to `facter --custom-dir`, arbitrary code execution as root was achieved. A SUID bash binary was created, providing a persistent root shell.

```bash
# Malicious Ruby fact
mkdir -p /tmp/exploit_facts
echo 'Facter.add("exploit") { setcode { system("chmod +s /bin/bash") } }' \
  > /tmp/exploit_facts/evil.rb

# Execute via sudo facter
sudo facter --custom-dir /tmp/exploit_facts exploit

# Spawn root shell
bash -p
```

#### Evidence

![Root Shell via facter](/assets/img/facts/root-shell.png)
*Figure 5: Root shell obtained via sudo facter --custom-dir arbitrary code execution*

#### Impact

Full root compromise. Unrestricted system access enabling credential dumping, persistence, lateral movement, and complete loss of system confidentiality, integrity, and availability.

> **Worst case:** Persistent root-level access to the system with ability to install backdoors, exfiltrate all data, and pivot to other network resources.

#### Remediation

1. Remove `/usr/bin/facter` from the sudoers configuration immediately
2. Audit all `NOPASSWD` sudo entries — apply the principle of least privilege
3. If `facter` execution is required by a legitimate process, run it as a dedicated service account with minimal permissions rather than root
4. Implement sudo logging and alerting for all privileged executions
5. Consider replacing `facter` with a more constrained facts collection mechanism

---

## 7. Attack Chain Narrative

The engagement began with service enumeration revealing a Camaleon CMS deployment and an exposed MinIO instance. A registered low-privileged account was obtained, and CVE research for Camaleon CMS v2.9.0 identified a known mass assignment vulnerability.

By intercepting a profile update request in Burp Suite and injecting `role=admin`, administrative privileges were granted to the test account without exploiting any authentication mechanism. This immediately unlocked the CMS admin panel.

Within the admin panel, the Filesystem Settings page exposed active MinIO credentials in plaintext. These credentials were used to authenticate to the MinIO service on port 54321, where the `internal` bucket was found to contain a `.ssh/` directory with an SSH private key.

The private key passphrase was cracked offline using a standard wordlist, enabling SSH authentication as user `william`. Post-authentication enumeration revealed a passwordless sudo permission allowing `facter` execution with `--custom-dir`, enabling arbitrary Ruby code execution as root.

### Attack Chain

```
[Unauthenticated Web User]
        │
        ▼
[F-01: CVE-2024-46987 Mass Assignment] ── Low-priv user → CMS Administrator
        │
        ▼
[F-02: MinIO Credentials in Admin Panel] ── Authenticated MinIO API access
        │
        ▼
[F-03: SSH Key in Object Storage] ─────── Passphrase cracked → SSH as william
        │
        ▼
[F-04: sudo facter --custom-dir RCE] ──── Root shell → Full system compromise
```

**Key Observation:** Each finding was independently significant, but the chain required all four steps. Fixing any single link — patching the CVE, removing credentials from the UI, securing the SSH key, or hardening sudoers — would have broken the attack path. This underscores the importance of defence-in-depth.

---

## 8. Business Impact

- **Confidentiality:** Complete exposure of all system data, stored objects, SSH credentials, and application secrets
- **Integrity:** Root-level write access enabling modification of any system file, binary, or configuration
- **Availability:** Ability to destroy data, disrupt services, or deploy ransomware
- **Compliance:** Exposure of credentials and private keys would trigger breach notification obligations under GDPR and similar frameworks
- **Reputational:** A publicly disclosed compromise of this nature would carry significant customer and partner trust implications

---

## 9. Remediation Roadmap

### Immediate *(24–72 hours)*
- [ ] Patch Camaleon CMS — remediate CVE-2024-46987
- [ ] Rotate all exposed MinIO credentials
- [ ] Remove SSH private keys from MinIO storage
- [ ] Rotate the compromised SSH key pair
- [ ] Remove facter from sudoers configuration
- [ ] Audit system for persistence mechanisms installed during assessment

### Short-Term *(30 days)*
- [ ] Restrict MinIO to internal network only
- [ ] Remove credentials from all web-accessible configuration panels
- [ ] Implement secrets management solution
- [ ] Audit all NOPASSWD sudo entries across all user accounts
- [ ] Harden object storage bucket policies

### Long-Term
- [ ] Implement centralized logging and SIEM alerting
- [ ] Establish patch management process for all CMS and framework dependencies
- [ ] Conduct periodic penetration testing
- [ ] Implement developer security training covering mass assignment and secrets management

---

## 10. Appendices

### Appendix A — Tools Used

| Tool | Purpose |
|---|---|
| nmap | Port and service enumeration |
| Burp Suite | HTTP interception and mass assignment exploitation |
| MinIO Client (mc) | Object storage enumeration and file retrieval |
| John the Ripper / hashcat | SSH key passphrase cracking |
| facter | Privilege escalation vector (target system tool) |

### Appendix B — References
- [CVE-2024-46987 — NVD](https://nvd.nist.gov/vuln/detail/CVE-2024-46987)
- [OWASP — Mass Assignment](https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/)
- [MITRE ATT&CK — T1548.003](https://attack.mitre.org/techniques/T1548/003/)
- [MITRE ATT&CK — T1552.004](https://attack.mitre.org/techniques/T1552/004/)
- [GTFOBins — facter](https://gtfobins.github.io/gtfobins/facter/)

### Appendix C — Methodology Standards
- **PTES** — Penetration Testing Execution Standard
- **OWASP WSTG** — Web Security Testing Guide
- **MITRE ATT&CK** — Technique classification

---

<div style="border-top: 1px solid #444; padding-top: 1rem; margin-top: 2rem; font-size: 0.85rem; color: #888;">
<strong>Disclaimer:</strong> This report was produced against a deliberately vulnerable HackTheBox environment for educational and portfolio demonstration purposes. No unauthorized testing was performed against real-world systems.
</div>
