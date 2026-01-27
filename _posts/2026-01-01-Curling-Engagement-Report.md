---
title: Curling Engagement Report
date: 2026-01-01
categories:
  - Reporting
  - Pentest Reports
tags: [Privilege Escalation]
---

# Curling Penetration Test Report

**Assessment Type:** External Penetration Test

**Author:** Sam Ireton

**Date:** 2026-01-01

## Target: Curling (10.129.71.236)

---

## 1. Executive Summary

An external penetration test was conducted against a Linux-based web application environment to evaluate exposure to unauthorized access, credential compromise, and privilege escalation.

The assessment identified multiple high-risk security issues that, when chained together, allowed a complete system compromise. An unauthenticated attacker could gain administrative access, execute arbitrary commands, extract credentials, and escalate privileges to root.

---

## 2. Scope & Assumptions

### In Scope
- Public web application (HTTP)
- Secure Shell (SSH)

### Out of Scope
- Denial-of-service testing
- Social engineering

### Assumptions
- No credentials were provided
- Target represents a production-like environment

---

## 3. Methodology

The assessment followed a structured penetration testing lifecycle aligned with industry standards:

1. Reconnaissance & Enumeration
2. Vulnerability Identification
3. Exploitation
4. Post-Exploitation
5. Privilege Escalation
6. Risk & Impact Assessment

---

## 4. Findings Summary

| ID   | Finding                         | Risk     |
|------|---------------------------------|----------|
| F-01 | Sensitive Information Disclosure | High     |
| F-02 | Insecure CMS Administration      | Critical |
| F-03 | Weak Credential Storage          | High     |
| F-04 | Insecure Privileged Automation   | Critical |

---

## 5. Detailed Findings

### F-01: Sensitive Information Disclosure

**Description:**
Encoded credentials were exposed through publicly accessible application content.

**Impact:**
Allowed attackers to obtain valid administrative credentials.

**Risk Rating:** High

---

### F-02: Insecure CMS Administration

**Description:**
Administrative access permitted unrestricted modification of server-side templates, enabling arbitrary code execution.

**Impact:**
Remote command execution on the underlying server.

**Risk Rating:** Critical

---

### F-03: Weak Credential Storage

**Description:**
Backup files containing credentials were weakly obfuscated and accessible after initial compromise.

**Impact:**
Enabled unauthorized SSH access and lateral movement.

**Risk Rating:** High

---

### F-04: Insecure Privileged Automation

**Description:**
A root-level automated process relied on user-controlled configuration files when retrieving data via `curl`.

**Impact:**
Allowed unauthorized access to sensitive system files and full privilege escalation.

**Risk Rating:** Critical

---

## 6. Attack Chain Summary

1. Public information disclosure exposed administrative credentials
2. CMS administrative access enabled remote code execution
3. Credential recovery enabled SSH access
4. Insecure automation enabled privilege escalation to root

This illustrates how multiple moderate issues can compound into a critical security failure.

---

## 7. Business Impact

A successful attack could using these methodologies could result in:
- Loss of system confidentiality and integrity
- Unauthorized access to sensitive data
- Potential lateral movement into internal environments
- Regulatory and reputational damage

---

## 8. Recommendations

1. Remove sensitive information from publicly accessible content
2. Enforce least privilege for application administrators
3. Secure credential storage using approved secrets management solutions
4. Restrict write access to configuration files used by privileged processes
5. Monitor and audit privileged automation activities

---

## 9. Disclaimer

This assessment was conducted against a deliberately vulnerable environment for educational and demonstration purposes. No unauthorized testing was performed against real-world systems.
