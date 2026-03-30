---
title: "enumkit — Interactive Pentest Enumeration Script"
date: 2026-01-18
categories:
  - Tools & Methodologies
  - Tools
tags: [Tools, Recon, Automation, Bash, nmap, ffuf, SMB, Enumeration]
description: "enumkit is an interactive bash script that runs a structured nmap, web, and SMB enumeration workflow against a target and prints a parsed findings summary on screen when complete. Designed to work standalone or alongside an engagekit workspace."
---

The early recon phase of a pentest has a predictable shape. Fast port sweep, deep service scan, web directory fuzzing if HTTP is open, SMB enumeration if 445 is up. The commands are the same every time. What changes is where the output ends up and how long it takes to make sense of it after the fact.

enumkit handles the mechanical side of that workflow and puts a parsed summary on screen when it finishes, so you start the analysis phase with the key findings already surfaced rather than buried in raw tool output.

## How It Works

Run it with a target IP or hostname and it walks you through a short setup before touching anything:

```bash
./enumkit.sh 10.10.10.1
```

The interactive flow covers output location, which modules to run, and the specific options for each:

```
╔══════════════════════════════════════╗
║           enumkit v1.0               ║
╚══════════════════════════════════════╝

[*] Target: 10.10.10.1

── Output Location ─────────────────────
  Where should output go?
    1) Create output in current directory (./enumkit-output/)
    2) Specify a custom directory
    3) Use an existing engagekit workspace

── Modules ─────────────────────────────
  Run nmap? [Y/n]
  Run web enumeration (ffuf + whatweb)? [Y/n]
  Run SMB enumeration? [Y/n]

── Nmap Options ────────────────────────
  Also run UDP top-100 scan? [y/N]
  Run vuln scripts after service scan? [y/N]

── Web Options ─────────────────────────
  Use default wordlist (raft-medium-directories)? [Y/n]
  Scan which protocols?
    1) HTTP and HTTPS
    2) HTTP only
    3) HTTPS only

── SMB Options ─────────────────────────
  Run enum4linux-ng? [Y/n]
  Run smbclient share list? [Y/n]

── Summary ─────────────────────────────
  Start enumeration? [Y/n]
```

Every question has a sensible default so you can hit Enter through most of it on a standard engagement. The confirmation step at the end shows exactly what will run before anything fires.

## Modules

**nmap** always runs a fast all-port sweep followed by a deep `-sC -sV` service scan against the identified ports. UDP top-100 and vuln scripts are optional and off by default.

**Web** runs whatweb for technology fingerprinting and ffuf for directory fuzzing against any HTTP or HTTPS ports found by nmap. If nmap did not run, it falls back to probing the common web ports. You can specify a custom wordlist or filter to HTTP or HTTPS only.

**SMB** runs enum4linux-ng and smbclient against the target. If nmap output is available and port 445 was not found, the script warns you before running anyway rather than silently skipping.

Every tool is checked for availability before use. If something is missing it warns and moves on rather than failing the whole run.

## Output Structure

```
enumkit-output/
└── scans/
    ├── nmap/
    │   ├── allports.nmap / .xml / .gnmap
    │   ├── targeted.nmap / .xml / .gnmap
    │   ├── udp-top100.*
    │   └── vuln.*
    ├── web/
    │   ├── whatweb-80.txt
    │   ├── ffuf-80.json
    │   └── ffuf-443.json
    └── smb/
        ├── enum4linux.txt
        └── smbclient-shares.txt
```

## The Findings Summary

When all selected modules finish, enumkit parses the raw output and prints a structured summary:

```
╔══════════════════════════════════════╗
║         Findings Summary             ║
╚══════════════════════════════════════╝

  Target: 10.10.10.1

── Open Ports & Services ───────────────
  22     ssh          OpenSSH 7.4 (protocol 2.0)
  80     http         Apache httpd 2.4.6
  445    microsoft-ds Windows Server 2016

── Notable Flags ────────────────────────
  [SSH]   Port 22 open — try default/weak creds, check banner for version
  [SMB]   Port 445 open — check null session, guest access, share permissions

── Web Technologies ─────────────────────
  Port 80:
    Title      : Welcome to AcmeCorp
    Server     : Apache/2.4.6 (CentOS)
    Powered-By : PHP/5.4.16

── Web Directories Found ────────────────
  [200] /admin   (1342 bytes)
  [301] /images  (234 bytes)
  [403] /backup  (289 bytes)

── SMB Shares ───────────────────────────
  ADMIN$   Disk   Remote Admin
  C$       Disk   Default share
  IPC$     IPC    Remote IPC
  Files    Disk

  [!] Null session succeeded — anonymous SMB access may be available
```

The notable flags section surfaces actionable intel automatically. It checks for anonymous FTP login, null SMB sessions, and flags open ports for SSH, RDP, WinRM, MSSQL, and LDAP with a one-line note on what to look at next. The web section pulls HTTP titles, server headers, and CMS detection from whatweb output. ffuf results are capped at 20 lines on screen with a pointer to the full JSON for anything beyond that.

The value here is not that the script makes decisions for you. It is that by the time you sit down to start working through the target, the relevant data is already organised and the obvious next steps are already written out.

## Integration with engagekit

enumkit is designed to pair with [engagekit](/posts/engagekit/). The workflow is:

```bash
# Scaffold the engagement workspace first
python engagekit.py AcmeCorp -c "Acme Corporation" -s internal

# Run enumkit and point it at the workspace when prompted
./enumkit.sh 10.10.10.1
# Output location → option 3 → enter the engagekit workspace path
```

When pointed at an engagekit workspace, enumkit drops output directly into the existing `scans/nmap`, `scans/web`, and `scans/smb` subfolders rather than creating a new directory. The findings summary then gives you everything you need to start filling in `notes/01-Recon.md`.

They can also run completely independently. enumkit creates its own output structure if no workspace is provided, and engagekit does not require enumkit to function.

## Installation

```bash
git clone https://github.com/sireton/enumkit
cd enumkit
chmod +x enumkit.sh
```

Dependencies: `nmap`, `ffuf`, `whatweb`, `enum4linux-ng`, `smbclient`. All available via `apt` on Kali, Parrot, or the HTB Pwnbox.

Full source: [github.com/sireton/enumkit](https://github.com/sireton/enumkit)

*See also: [engagekit — Pentest Engagement Workspace Scaffolder](/posts/engagekit/)*
