---
title: "Facts – Engagement Report" 
date: 2026-02-13
categories: [Engagement Reports]
tags: [Web Exploitation, Privilege Escalation, Credential Exposure, Linux]
description: "Full-chain compromise of a Linux host via Camaleon CMS mass assignment vulnerability, exposed MinIO credentials, SSH key abuse, and sudo misconfiguration."

---

**Assessment Date:** 2026-02-13  
**Target Host:** 10.129.8.32  
**Platform:** Linux (Ubuntu 25.04)  
**Assessment Type:** Web Application & Host Security Assessment  

---

## Executive Summary

A security assessment was conducted against a Linux host running **Camaleon CMS v2.9.0** and an exposed **MinIO object storage service**.

The engagement identified a chained exploitation path that resulted in **full root compromise**. The attack chain leveraged:

- A mass assignment vulnerability in Camaleon CMS (CVE-2024-46987)
- Exposure of MinIO object storage credentials in the admin panel
- Retrieval and cracking of SSH private keys
- A sudo misconfiguration permitting arbitrary Ruby execution via `facter`

The vulnerabilities collectively allowed complete loss of confidentiality, integrity, and system control.

---

## Scope

**In Scope:**

- All exposed services on 10.129.8.32
- SSH (22)
- HTTP (80)
- MinIO (54321)

**Out of Scope:**

- Denial of Service
- Excessive brute-force attempts

---

## Service Overview

| Port | Service | Version | Notes |
|------|---------|----------|--------|
| 22 | SSH | OpenSSH 9.9p1 | Remote access |
| 80 | HTTP | nginx 1.26.3 | Rails CMS |
| 54321 | HTTP | MinIO | S3-compatible object storage |

---

# Findings

---

## 1. Mass Assignment Vulnerability in Camaleon CMS

**Severity:** High  
**CVE:** CVE-2024-46987  
**Category:** Access Control / Privilege Escalation  
**Affected Endpoint:** `/admin/users/<id>/updated_ajax`

### Description

Camaleon CMS ≤ 2.9.0 contains a mass assignment vulnerability allowing authenticated users to modify restricted parameters during profile updates.

By intercepting a legitimate update request and injecting a `role=admin` parameter, administrative privileges were granted to a low-privileged account.

The vulnerability stems from improper strong parameter filtering within the Rails controller.

### Impact

- Unauthorized administrator access  
- Access to backend configuration panels  
- Exposure of sensitive credentials  

### Evidence

![Mass Assignment Exploit](/assets/img/facts/mass-assignment.png)
![Mass Assignment Exploit](/assets/img/facts/mass-assignment2.png)

---

## 2. Exposure of MinIO Object Storage Credentials

**Severity:** Critical  
**Category:** Sensitive Information Disclosure  

### Description

Administrative configuration panels exposed active MinIO credentials, including:

- Access Key
- Secret Key
- Bucket Name
- Internal endpoint

Using these credentials, authenticated API access to internal object storage was achieved.

The `internal` bucket contained a `.ssh` directory with private keys.

### Impact

- Credential compromise  
- Direct authentication bypass  
- Lateral movement to system shell  

### Evidence

![MinIO Credentials](/assets/img/facts/minio-credentials.png)

---

## 3. SSH Private Key Exposure & Authentication Bypass

**Severity:** Critical  
**Category:** Credential Exposure  

### Description

The MinIO `internal` bucket contained:

- `id_ed25519`
- `authorized_keys`

The private key was protected with a weak passphrase, which was cracked using standard wordlist techniques.

Successful SSH authentication was obtained as a valid system user.

### Impact

- Remote shell access  
- User-level compromise  
- Authentication control bypass  

### Evidence

![SSH Access](/assets/img/facts/ssh-access.png)

---

## 4. Sudo Misconfiguration – Arbitrary Root Code Execution

**Severity:** Critical  
**Category:** Privilege Escalation  

### Description

The compromised user account was permitted to execute:
(ALL) NOPASSWD: /usr/bin/facter


`facter` supports loading custom Ruby facts using `--custom-dir`.

By creating a malicious Ruby fact file, arbitrary code execution as root was achieved, resulting in a SUID-enabled shell and full system compromise.

### Impact

- Root-level execution  
- Ability to modify system binaries  
- Full system takeover  

### Evidence

![Root Shell](/assets/img/facts/root-shell.png)

---

# Attack Chain Summary

1. Mass assignment → Administrator privileges  
2. Admin panel → MinIO credential exposure  
3. MinIO access → SSH private key retrieval  
4. SSH authentication → User shell access  
5. Sudo misconfiguration → Root compromise  

This demonstrates a failure of defense-in-depth controls.

---

# Risk Overview

| Vulnerability | Severity | Business Impact |
|--------------|----------|----------------|
| Mass Assignment (CVE-2024-46987) | High | Privilege escalation |
| Exposed MinIO Credentials | Critical | Credential disclosure |
| SSH Key Exposure | Critical | Authentication bypass |
| Sudo Misconfiguration | Critical | Full system compromise |

---

# Remediation Recommendations

## Immediate Actions

- Patch Camaleon CMS to a non-vulnerable version
- Rotate all exposed MinIO credentials
- Remove SSH private keys from object storage
- Rotate SSH host and user keys
- Audit system for persistence mechanisms

## Application Hardening

- Enforce strict strong parameter validation
- Implement server-side role validation
- Restrict administrative configuration exposure

## Infrastructure Hardening

- Restrict MinIO to internal-only access
- Remove secrets from web panels
- Use centralized secret management (Vault, environment variables)
- Harden sudo configuration (avoid direct binary execution)

## Long-Term Security Improvements

- Implement centralized logging & monitoring
- Enforce least privilege access controls
- Conduct periodic security assessments
- Implement secrets rotation policies

---

# Conclusion

The assessment demonstrated how multiple moderate-to-severe misconfigurations can be chained to achieve complete system compromise.

While each vulnerability independently posed risk, the combination enabled escalation from web user to root-level system control.

This case highlights the importance of:

- Secure development practices  
- Credential management discipline  
- Least privilege enforcement  
- Defense-in-depth architecture  

Immediate remediation is strongly recommended.

---

*Report prepared for portfolio demonstration purposes.*


