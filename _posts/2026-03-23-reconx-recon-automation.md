---
title: "reconX — Automated Engagement Recon Script"
date: 2026-03-23
categories: [Tools & Methodologies]
tags: [Tools, Recon, Automation, Bash, nmap, ffuf, Enumeration]
description: "reconX is a bash script I built to automate the initial enumeration phase of a pentest engagement — chaining nmap, ffuf, and service-specific enumeration into a structured, timestamped output directory."
---

Early in a pentest engagement, the recon phase is both the most repeatable and the most error-prone. You're running the same nmap flags you always run, the same ffuf wordlists against any web services you find, the same SMB enumeration if port 445 is open. The commands are muscle memory, but the context-switching means things get missed, output files land in random places, and your notes end up inconsistent across engagements.

reconX is the script I wrote to solve that for myself. It handles the mechanical parts of initial enumeration so I can focus on interpreting results rather than managing tooling.

---

## What It Does

Given a target IP or hostname, reconX runs a structured recon workflow and organizes all output into a timestamped engagement directory:

1. **Fast all-port sweep** — `masscan` or fast `nmap` to identify open ports quickly
2. **Deep service scan** — full `-sC -sV` against identified ports
3. **Web detection** — if HTTP/HTTPS ports are found, automatically runs `ffuf` directory fuzzing and `whatweb` fingerprinting
4. **Service-specific enumeration** — SMB, FTP, SNMP, and LDAP modules trigger automatically based on what nmap finds
5. **Structured output** — everything lands in `./engagements/<target>-<timestamp>/` with clean subdirectories per phase

The goal is to arrive at the analysis phase with all the raw data already collected and organized, rather than chasing down which terminal window has which output.

---

## Directory Structure

```
engagements/
└── 10.129.71.236-20260315-143022/
    ├── recon/
    │   ├── allports.nmap
    │   ├── targeted.nmap
    │   └── targeted.xml
    ├── web/
    │   ├── ffuf-80.json
    │   ├── ffuf-443.json
    │   └── whatweb-80.txt
    ├── smb/
    │   ├── enum4linux.txt
    │   └── smbclient-shares.txt
    └── recon_summary.md    ← Auto-generated markdown summary
```

The `recon_summary.md` file is auto-populated with open ports, service versions, web technologies found, and any notable findings (anonymous FTP, null SMB sessions, HTTP titles), formatted for direct pasting into an engagement notes file.

---

## Core Script

> 📁 Full source and usage documentation: [github.com/sireton/reconX](https://github.com/sireton/reconX)

```bash
#!/usr/bin/env bash
# reconX — Automated Engagement Recon
# Usage: ./reconx.sh <target> [--web-only] [--no-smb]

TARGET="$1"
TIMESTAMP=$(date +%Y%m%d-%H%M%S)
OUTDIR="./engagements/${TARGET}-${TIMESTAMP}"
WORDLIST="/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt"

# --- Setup ---
mkdir -p "$OUTDIR"/{recon,web,smb,ftp,notes}
echo "[*] reconX starting — target: $TARGET"
echo "[*] Output directory: $OUTDIR"

# --- Phase 1: Fast port sweep ---
echo "[*] Phase 1: Fast port sweep..."
nmap -p- --min-rate 5000 -oA "$OUTDIR/recon/allports" "$TARGET" > /dev/null 2>&1
PORTS=$(grep "^[0-9]" "$OUTDIR/recon/allports.nmap" | cut -d'/' -f1 | tr '\n' ',' | sed 's/,$//')

if [ -z "$PORTS" ]; then
    echo "[-] No open ports found. Exiting."
    exit 1
fi

echo "[+] Open ports: $PORTS"

# --- Phase 2: Deep service scan ---
echo "[*] Phase 2: Service/version scan on $PORTS..."
nmap -p "$PORTS" -sC -sV -oA "$OUTDIR/recon/targeted" "$TARGET" > /dev/null 2>&1

# --- Phase 3: Web enumeration (if 80/443/8080/8443 open) ---
for PORT in 80 443 8080 8443; do
    if echo "$PORTS" | grep -q "$PORT"; then
        SCHEME="http"
        [ "$PORT" == "443" ] || [ "$PORT" == "8443" ] && SCHEME="https"
        URL="${SCHEME}://${TARGET}:${PORT}"

        echo "[*] Phase 3: Web enumeration on $URL..."
        
        # WhatWeb fingerprint
        whatweb "$URL" -a 3 --log-verbose="$OUTDIR/web/whatweb-${PORT}.txt" > /dev/null 2>&1
        
        # Directory fuzzing
        ffuf -w "$WORDLIST" \
             -u "${URL}/FUZZ" \
             -mc 200,204,301,302,307,401,403 \
             -o "$OUTDIR/web/ffuf-${PORT}.json" \
             -of json \
             -t 40 \
             -s > /dev/null 2>&1

        echo "[+] Web enum complete for port $PORT"
    fi
done

# --- Phase 4: SMB enumeration (if 445 open) ---
if echo "$PORTS" | grep -q "445"; then
    echo "[*] Phase 4: SMB enumeration..."
    enum4linux-ng -A "$TARGET" > "$OUTDIR/smb/enum4linux.txt" 2>&1
    smbclient -N -L "//$TARGET" > "$OUTDIR/smb/smbclient-shares.txt" 2>&1
    echo "[+] SMB enum complete"
fi

# --- Phase 5: FTP check (if 21 open) ---
if echo "$PORTS" | grep -q "21"; then
    echo "[*] Phase 5: FTP anonymous check..."
    timeout 10 ftp -n "$TARGET" <<EOF > "$OUTDIR/ftp/anon-check.txt" 2>&1
quote USER anonymous
quote PASS anon@test.com
ls -la
quit
EOF
    echo "[+] FTP check complete"
fi

# --- Generate summary ---
echo "[*] Generating recon_summary.md..."
{
    echo "# Recon Summary — $TARGET"
    echo "_Generated: $(date)_"
    echo ""
    echo "## Open Ports"
    echo '```'
    grep "^[0-9]" "$OUTDIR/recon/allports.nmap"
    echo '```'
    echo ""
    echo "## Service Versions"
    echo '```'
    grep "^[0-9]" "$OUTDIR/recon/targeted.nmap" | grep "open"
    echo '```'
    echo ""
    echo "## Notes"
    echo "- [ ] Review web findings in \`web/\`"
    echo "- [ ] Check SMB shares for sensitive files"
    echo "- [ ] Search CVEs for identified service versions"
} > "$OUTDIR/recon_summary.md"

echo ""
echo "[+] reconX complete."
echo "[+] Summary: $OUTDIR/recon_summary.md"
echo "[+] Full output: $OUTDIR/"
```

---

## Usage Examples

```bash
# Basic usage — full recon against a target
./reconx.sh 10.129.71.236

# Web-only mode (skip SMB/FTP, useful if you already know the ports)
./reconx.sh 10.129.71.236 --web-only

# With a hostname (auto-resolves)
./reconx.sh target.htb
```

---

## Installation

```bash
git clone https://github.com/sireton/reconX
cd reconX
chmod +x reconx.sh

# Verify dependencies
which nmap ffuf whatweb enum4linux-ng smbclient
```

**Dependencies:** `nmap`, `ffuf`, `whatweb`, `enum4linux-ng`, `smbclient`  
All available via `apt` on Kali/Parrot or the HTB Pwnbox.

---

## Design Decisions

A few choices worth explaining:

**Why bash and not Python?** The dependencies are all CLI tools. Bash keeps it portable and dependency-free beyond what's already on a standard pentest distro. Python would be better for more complex logic; for chaining CLI tools, bash is the right abstraction.

**Why a timestamped directory per engagement?** Engagement directories stack up quickly. Timestamping prevents collisions and makes it easy to diff results between runs against the same target (useful when you're returning to a machine after taking a break).

**Why a markdown summary?** The output feeds directly into my engagement notes structure. The `recon_summary.md` is designed to be dropped into an Obsidian vault or pasted into a report template without reformatting.

---

## Planned Additions

- SNMP enumeration module (onesixtyone + snmpwalk)
- Vhost fuzzing when a hostname is provided
- `--scope` flag for multi-target engagements (reads from a file)
- HTML report output option

---

*See also: [Recon Methodology Notes](/posts/methodology-recon/) · [HTB Curling Write-Up](/posts/Curling-Joomla-RCE-Privileged-Automation-Abuse/) (reconX used for initial enumeration)*
