---
title: "engagekit — Pentest Engagement Folder Creator"
date: 2026-01-18
categories:
  - Tools & Methodologies
  - Tools
tags: [Tools, Automation, Python, Pentest, Reporting, Notes, Engagement]
description: "engagekit is a cross-platform Python script that builds a structured pentest engagement workspace in seconds with pre-populated markdown templates, organised scan folders, a loot tracker, and a timestamped evidence capture helper, all named and dated for the engagement automatically."
---


## The Problem It Solves

engagekit solves the slow process of engagment setup for evidence gathering. Run it once before you start and the workspace is there, already organised, already labelled with the client name and date, ready to receive output.

## What It Generates

```
AcmeCorp-20260330/
├── README.md                        ← Workspace quick reference
├── 00-Scope.md                      ← Fill this out before anything else
├── notes/
│   ├── 01-Recon.md
│   ├── 02-Foothold.md
│   ├── 03-Post-Exploitation.md
│   ├── 04-Lateral-Movement.md
│   ├── 05-Privilege-Escalation.md
│   └── 06-Findings.md               ← CVSS and MITRE ATT&CK ready
├── scans/
│   ├── nmap/
│   ├── web/
│   ├── smb/
│   └── ldap/
├── evidence/
│   └── capture.sh  /  capture.bat
├── loot/
│   └── loot.md
└── tools/
```

Every markdown file opens pre-populated with the client name, tester, date, and scope. The `06-Findings.md` file has CVSS vector fields and MITRE ATT&CK mapping slots built in so you are not formatting finding blocks from scratch at the end of a long day.

The `loot/loot.md` file carries a warning header reminding you it contains sensitive material that should never be committed to a public repo. That reminder is intentional and worth having in the file itself.

## Scope-Aware Folder Structure

One of the things I wanted to avoid was a generic folder dump that does not match the engagement type. An internal network assessment and a web application test produce completely different tool output and do not need the same scan subfolders.

engagekit takes a `--scope` flag that controls which `scans/` subfolders get created:

| Scope flag | Subfolders created | Best for |
|---|---|---|
| `full` | nmap, web, smb, ldap | General or unknown scope |
| `internal` | nmap, smb, ldap | Internal network assessments |
| `external` | nmap, web | External perimeter |
| `webapp` | web | Web application only |
| `ad` | nmap, smb, ldap | Active Directory focused |

## Installation

No external dependencies. Python 3.8 or later is all you need.

```bash
git clone https://github.com/sireton/engagekit
cd engagekit
```

**Optional: add to PATH so you can call it from anywhere**

```bash
# Linux / macOS
cp engagekit.py ~/.local/bin/engagekit
chmod +x ~/.local/bin/engagekit
```

## Usage

```bash
python engagekit.py <target> [options]
```

A few representative examples:

```bash
# Quickest form — target name only
python engagekit.py AcmeCorp

# Full client engagement with all options
python engagekit.py AcmeCorp -c "Acme Corporation" -s internal -a "Sam Ireton"

# Web application scope
python engagekit.py ClientPortal -c "Client Web Portal" -s webapp

# Custom output directory
python engagekit.py AcmeCorp -o ~/Engagements

# Preview without creating anything
python engagekit.py AcmeCorp --dry-run

# Folder structure only, skip markdown templates
python engagekit.py AcmeCorp --no-templates
```

The `--dry-run` flag is worth mentioning specifically. It prints the full list of directories and files that would be created without writing a single byte to disk. Useful for confirming the output path and structure before committing to it, especially when you are specifying a custom output directory for the first time.

## The Evidence Capture Helper

The `evidence/` folder contains either `capture.sh` (Linux/macOS) or `capture.bat` (Windows), generated automatically based on the OS the script detects at runtime.

The idea is straightforward: you run it with a short description of what you are capturing, paste command output, and save. It writes a timestamped file with a header containing the engagement name, description, and host details.

```bash
# Linux / macOS
cd AcmeCorp-20260330/evidence/
./capture.sh "nmap full port scan"
# Paste nmap output
# Ctrl+D to save
# Creates: 20260330_143022-nmap_full_port_scan.txt
```

The output looks like this:

```
======================================================
  Engagement : Acme Corporation
  Capture    : nmap full port scan
  Timestamp  : Mon Mar 30 14:30:22 UTC 2026
  Host       : kali | User: kali
======================================================

Starting Nmap 7.94 ...
[output continues]
```

On Windows the same concept works with `capture.bat`, saving via `Ctrl+Z` and Enter instead of `Ctrl+D`. When you are working across both a Windows host and a Linux VM, engagekit detects which OS it is running on and generates the appropriate helper automatically.

## Practical Workflow

This is the order I use it:

1. Run engagekit with the target name and scope before touching any tools
2. Fill out `00-Scope.md`: IP range, in/out of scope, rules of engagement, any credentials provided
3. Work through `notes/01` through `notes/05` as the engagement progresses, keeping commands and findings in the relevant phase file
4. Drop raw tool output directly into `scans/<tooltype>/` — nmap XML files go in `scans/nmap/`, ffuf output in `scans/web/`, and so on
5. Use `evidence/capture.sh` to save key command output with timestamps as you go, not after the fact
6. Track everything sensitive in `loot/loot.md` as you find it
7. Build findings in `notes/06-Findings.md` throughout the engagement so the CVSS blocks and MITRE mappings are done before you sit down to write the report

The last point is the one that saves the most time. Filling in CVSS vectors and MITRE technique IDs when you are actively exploiting something takes seconds. Reconstructing them from memory at report time takes considerably longer.

## Cross-Platform Behaviour

The script detects the OS at runtime using Python's `platform.system()`. On Windows it sanitizes folder names to strip characters that are illegal in Windows paths (`<>:"/\|?*`), and generates `capture.bat` instead of `capture.sh`. On Linux and macOS it generates `capture.sh` and marks it executable automatically so you do not need a manual `chmod`.

The folder structure and markdown content are identical regardless of platform.

## Source

Full source and documentation: [github.com/sireton/engagekit](https://github.com/sireton/tools-methodologies/tree/Practice/Tools/vuln%20testing/EngageKit)

