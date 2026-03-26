---
title: "engageinit — Engagement Note Scaffolder"
date: 2026-03-23
categories:
  - Tools & Methodologies
  - Tools
tags: [Tools, Automation, Bash, Reporting, Notes, Obsidian]
description: "engageinit is a bash script that generates a complete, pre-structured engagement directory from a target name — pre-filled markdown templates, evidence capture scripts, and a ready-to-use notes vault."
---

One thing I've noticed across engagements is that the quality of reporting is heavily influenced by note-taking that happens in the moment, not afterward. Screenshots get lost, command output disappears from terminal history, the exact syntax of a working payload is forgotten. The fix isn't discipline — it's removing friction from the note-taking process at the point when you're in the middle of exploiting a target and the last thing you want to do is organize files.

engageinit solves the setup problem. Run it with a target name before you start an engagement and you have a complete, pre-structured workspace ready to go — no manual directory creation, no hunting for templates, no missing front matter.

---

## What It Generates

```
engagements/
└── Curling-20260315/
    ├── 00-Scope.md            ← Pre-filled with target, date, scope
    ├── 01-Recon.md            ← Recon phase notes template
    ├── 02-Foothold.md         ← Foothold template with checklist
    ├── 03-Post-Exploitation.md
    ├── 04-Lateral-Movement.md
    ├── 05-Privilege-Escalation.md
    ├── 06-Findings.md         ← CVSS-ready finding blocks
    ├── evidence/              ← Screenshots, command output
    │   └── capture.sh         ← One-command timestamped evidence capture
    ├── loot/                  ← Hashes, credentials, flags
    ├── tools/                 ← Staging area for payloads
    └── report/
        ├── Engagement-Report-Template.md
        └── Walkthrough-Template.md
```

Every markdown file opens with front matter pre-populated with the target name, date, and your author name — ready for the blog publishing workflow.

---

## The Evidence Capture Helper

The most-used part of engageinit in practice is `evidence/capture.sh`:

```bash
#!/usr/bin/env bash
# Usage: ./capture.sh "description of what you're capturing"
# Drops timestamped command output into evidence/

DESC="${1:-capture}"
OUTFILE="./$(date +%Y%m%d_%H%M%S)-${DESC// /_}.txt"

{
    echo "=== Evidence Capture: $DESC ==="
    echo "=== Timestamp: $(date) ==="
    echo "=== Host: $(hostname) | User: $(whoami) ==="
    echo ""
} >> "$OUTFILE"

echo "[*] Capturing to $OUTFILE"
echo "[*] Paste command output, then Ctrl+D to save"
cat >> "$OUTFILE"
echo "[+] Saved: $OUTFILE"
```

Run it from the `evidence/` directory with a description, paste your command output, `Ctrl+D` to save. Every piece of evidence is timestamped and described, which makes the reporting phase substantially faster.

---

## The Script

> 📁 Full source and usage documentation: [github.com/sireton/engageinit](https://github.com/sireton/engageinit)

```bash
#!/usr/bin/env bash
# engageinit — Engagement Note Scaffolder
# Usage: ./engageinit.sh <TargetName> [--author "Your Name"]

TARGET="${1:-Target}"
AUTHOR="${3:-Sam Ireton}"
DATE=$(date +%Y%m%d)
ENGDIR="./engagements/${TARGET}-${DATE}"

# --- Directory structure ---
mkdir -p "$ENGDIR"/{evidence,loot,tools,report}

echo "[*] Initializing engagement workspace for: $TARGET"

# --- Scope file ---
cat > "$ENGDIR/00-Scope.md" << EOF
---
target: $TARGET
date: $(date +%Y-%m-%d)
author: $AUTHOR
status: In Progress
---

# Engagement Scope — $TARGET

## Target
- **IP/Hostname:** 
- **OS:** 
- **Platform:** 

## Engagement Type
- [ ] Black Box
- [ ] Grey Box  
- [ ] White Box

## In Scope
- 

## Out of Scope
- DoS testing
- Social engineering

## Rules of Engagement
- 

## Credentials Provided
- None (black box) / [list if grey/white box]
EOF

# --- Phase templates ---
for PHASE in "01-Recon" "02-Foothold" "03-Post-Exploitation" "04-Lateral-Movement" "05-Privilege-Escalation"; do
    cat > "$ENGDIR/${PHASE}.md" << EOF
---
target: $TARGET
phase: ${PHASE#*-}
date: $(date +%Y-%m-%d)
author: $AUTHOR
---

# ${PHASE#*-} — $TARGET

## Checklist
- [ ] 

## Commands Run

\`\`\`bash
# 
\`\`\`

## Findings / Notes

## Evidence
- See \`evidence/\` directory

EOF
done

# --- Findings template ---
cat > "$ENGDIR/06-Findings.md" << EOF
---
target: $TARGET
date: $(date +%Y-%m-%d)
author: $AUTHOR
---

# Findings — $TARGET

## Summary

| ID | Title | Severity | CVSS |
|---|---|---|---|
| F-01 | | 🔴 Critical | |

---

## F-01 — [Title]

| Field | Detail |
|---|---|
| **Severity** | 🔴 Critical |
| **CVSS Score** | |
| **CVSSv3 Vector** | \`AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H\` |
| **Affected Component** | |
| **MITRE ATT&CK** | \`TXXXX — Name\` |

### Description

### Evidence
\`\`\`bash
# 
\`\`\`

### Impact
> **Worst case:** 

### Remediation
1. 

EOF

# --- Evidence capture script ---
cat > "$ENGDIR/evidence/capture.sh" << 'EOF'
#!/usr/bin/env bash
DESC="${1:-capture}"
OUTFILE="./$(date +%Y%m%d_%H%M%S)-${DESC// /_}.txt"
{ echo "=== $DESC | $(date) | $(whoami)@$(hostname) ==="; echo ""; } >> "$OUTFILE"
echo "[*] Capturing to $OUTFILE — Ctrl+D to save"
cat >> "$OUTFILE"
echo "[+] Saved."
EOF
chmod +x "$ENGDIR/evidence/capture.sh"

# --- Loot tracker ---
cat > "$ENGDIR/loot/loot.md" << EOF
# Loot — $TARGET

## Credentials
| Username | Password / Hash | Source | Used For |
|---|---|---|---|
| | | | |

## Flags
| Type | Value | Location |
|---|---|---|
| User | | |
| Root | | |

## SSH Keys
- 

## Interesting Files
- 
EOF

echo ""
echo "[+] Engagement workspace ready: $ENGDIR/"
echo "[+] Start with: $ENGDIR/00-Scope.md"
echo "[+] Evidence capture: $ENGDIR/evidence/capture.sh \"description\""
```

---

## Usage

```bash
# Basic
./engageinit.sh Curling

# With custom author name
./engageinit.sh "Active-Directory-Lab" --author "Sam Ireton"

# Output lives in ./engagements/<target>-<date>/
```



## Design Notes

**Why markdown and not a database or web app?** Markdown files work everywhere, sync with git, render in Obsidian, and can be pushed directly to a Jekyll blog. The entire workflow from engagement notes to published write-up is one format.

**Why a flat phase structure instead of nested directories?** When you're mid-engagement, navigating a deep directory tree in a terminal is friction. A flat set of markdown files in one directory is faster to work with under time pressure.

---

*See also: [Pentest Methodology Notes](/posts/methodology-index/)*
