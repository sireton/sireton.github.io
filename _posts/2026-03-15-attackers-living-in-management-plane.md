---
title: "Attackers Are Living in Your Management Plane"
date: 2026-03-15
categories: [Blog Posts]
tags: [Threat Research, Red Team, VMware, Management Plane, Exploitation, Opinion, Infrastructure]
description: "CVE-2026-22719 in VMware Aria is the latest in a pattern that's been building for years: the tools you use to manage infrastructure are becoming the primary attack surface."
---

There is a recurring pattern in modern vulnerability disclosures. A new CVE surfaces, CISA issues an advisory, and another platform is labeled actively exploited. These cases are usually discussed on their own, but together they reveal a broader shift in attacker focus.

The systems used to manage and observe enterprise infrastructure are becoming a primary attack surface.

That shift is visible across CISA’s Known Exploited Vulnerabilities catalog, which has repeatedly included technologies such as VPN platforms, firewalls, and remote access systems. These are not ordinary application-layer services. They sit in the control path of the enterprise, with visibility into critical operations and the ability to influence how large portions of the environment function.

Recent exploitation of platforms like Ivanti Connect Secure reinforces the point where initial access was achieved through exposed or unpatched management interfaces, even in environments that had otherwise invested heavily in hardening production systems.

The deeper issue is not just technical exposure, but organizational assumption. These platforms are still often treated as internal infrastructure and therefore as lower priority. In practice, that leads to slower patching, lighter monitoring, and broader trust boundaries than would be acceptable for more obviously exposed systems.

That disconnect between how these systems are perceived and what they actually represent is creating a reliable path for intrusion.

---

## The Latest Example

Earlier this month, CISA added CVE-2026-22719 to the Known Exploited Vulnerabilities catalog. Broadcom describes it as a command injection vulnerability in VMware Aria Operations that can allow an unauthenticated attacker to execute arbitrary commands during a support-assisted migration workflow ([CISA KEV](https://www.cisa.gov/known-exploited-vulnerabilities-catalog?field_date_added_wrapper=all&items_per_page=20&search_api_fulltext=vmware&sort_by=field_date_added); [Broadcom advisory](https://support.broadcom.com/web/ecx/support-content-notification/-/external/content/SecurityAdvisories/0/36947)).

That is not just another remote code execution flaw. It is unauthenticated code execution in a platform designed to monitor, assess, and help manage virtual infrastructure.

Aria Operations sits close to the center of the virtualization stack. In many environments, platforms like it aggregate performance data, inventory context, and operational visibility across large portions of the estate. A compromise here does not just create access to one workload. It creates visibility into the environment and potentially privileged pathways through it.

## The Broader Trend

Aria is not an isolated case. The same trend appears across the management and control layer of enterprise environments. CISA issued Emergency Directive 24-01 because Ivanti Connect Secure and Policy Secure vulnerabilities were being actively exploited, and later published joint guidance on the broader intrusion activity tied to those flaws ([CISA ED 24-01](https://www.cisa.gov/news-events/directives/ed-24-01-mitigate-ivanti-connect-secure-and-ivanti-policy-secure-vulnerabilities); [CISA AA24-060B](https://www.cisa.gov/news-events/cybersecurity-advisories/aa24-060b)).

Palo Alto also disclosed CVE-2024-3400, a command injection vulnerability in the GlobalProtect feature of PAN-OS that could allow unauthenticated code execution on affected firewalls under specific conditions. Fortinet disclosed FG-IR-24-535, an authentication bypass vulnerability affecting FortiOS and FortiProxy that it said was being exploited in the wild ([Palo Alto CVE-2024-3400](https://security.paloaltonetworks.com/CVE-2024-3400); [Fortinet FG-IR-24-535](https://www.fortiguard.com/psirt/FG-IR-24-535)).

These platforms are not peripheral systems. They are administrative gateways, policy enforcement points, and remote access layers. They exist to observe, control, and broker access across the environment. That is exactly why they are so valuable to an attacker. A foothold in the management plane is not simply a foothold inside the network. It is a foothold inside the systems that shape how the network is operated.

## Why the Management Plane Works So Well for Attackers

From an attacker’s perspective, the management plane often offers a more efficient path to meaningful access than traditional production targets.

Production systems are increasingly hardened. EDR coverage is more widespread. Segmentation is more common. Detection strategies are more likely to be tuned around business workloads and user-facing services.

Management systems often operate under a different set of assumptions.

They may follow slower patch cycles because of perceived operational sensitivity. They are often accessed by a smaller number of privileged administrators, which narrows the behavioral baseline. They also maintain legitimate communication paths across broad portions of the environment, which makes suspicious activity harder to distinguish from expected administration. NIST’s enterprise patch management guidance explicitly frames patching as a cost of doing business and a necessary preventive maintenance function, showing that delayed patching is usually a prioritization problem, not just a technical one ([NIST SP 800-40 Rev. 4](https://csrc.nist.gov/pubs/sp/800/40/r4/final)).

An attacker with code execution on a management platform is not just inside the network. They are operating from a position of trust, with built-in visibility and administrative reach.


## The Defense Posture Gap

The issue is not simply exposure. It is prioritization.

For a number valid time, fiscal, and resource restrictions, many organizations still sort risk into two rough categories: internet-facing systems and everything else. That model assumes internal systems are harder to reach, less likely to be targeted directly, and therefore safer to patch, monitor, and review on a slower cycle. That assumption breaks down when management interfaces are reachable, administrative APIs are exposed, and control systems aggregate privileged access across large portions of the environment.

That is why recent exploitation of platforms such as Ivanti Connect Secure, PAN-OS with GlobalProtect exposure, and Fortinet infrastructure is so significant. These are not ordinary business applications. They are systems used to administer access, enforce policy, and control enterprise operations. When vulnerabilities in those platforms are actively exploited, it demonstrates that attackers are deliberately targeting the systems organizations often treat as lower-priority internal infrastructure ([CISA ED 24-01](https://www.cisa.gov/news-events/directives/ed-24-01-mitigate-ivanti-connect-secure-and-ivanti-policy-secure-vulnerabilities); [CISA AA24-060B](https://www.cisa.gov/news-events/cybersecurity-advisories/aa24-060b); [Palo Alto CVE-2024-3400](https://security.paloaltonetworks.com/CVE-2024-3400); [Fortinet FG-IR-24-535](https://www.fortiguard.com/psirt/FG-IR-24-535)).

The operational result is a consistent defensive gap. Edge-facing systems often receive immediate attention when a critical vulnerability is disclosed because their exposure is obvious. Internal management platforms are more likely to wait for a maintenance window, even though the impact of compromise may be broader. The issue is not that defenders ignore these platforms entirely. It is that urgency is often assigned based on visibility of exposure rather than concentration of control.

Monitoring often follows the same logic. Production systems tend to receive fuller telemetry, more mature alert tuning, and more frequent review because they are seen as the most likely targets or the most operationally sensitive assets. Management platforms, by contrast, may operate with thinner visibility or detection logic that is not designed to catch low-noise abuse of trusted tools and administrative interfaces. That creates a condition where some of the most privileged systems in the environment are also some of the least scrutinized.

Part of that gap is also economic. Once a system is viewed as lower risk, budget, staffing, and operational attention often follow the same assumption. NIST’s patch management guidance makes clear that patching requires enterprise planning, governance, and sustained operational commitment ([NIST SP 800-40 Rev. 4](https://csrc.nist.gov/pubs/sp/800/40/r4/final)). In other words, when management systems are deprioritized in the risk model, they are often deprioritized in resourcing as well.

This is not just a tooling problem. It is a governance and prioritization problem.


## What This Means Practically

From an offensive perspective, the management plane is a reliable place to spend time in otherwise mature environments.

Exposed management APIs, observability platforms with weak access controls, unpatched administrative interfaces, and overly trusted orchestration systems can all provide high-value entry points. These weaknesses often persist in environments that have done a much better job hardening production workloads.

From a defensive perspective, the answer is not simply to "patch faster." It is to patch more deliberately, monitor more intelligently, and reduce the blast radius of the systems that cannot be remediated immediately.

**1. Treat management systems as high-consequence infrastructure.**  
The first step is classification. Management systems should be treated as high-consequence infrastructure, even when they are not directly internet-facing. If a platform can administer identity, networking, virtualization, backup, security tooling, or remote access, it should be placed in the same decision tier as edge infrastructure for vulnerability triage and change review.

**2. Prioritize by exploitability, exposure, and concentration of control.**  
In lean environments, not every vulnerability can be remediated at once, so the most effective approach is to rank management-plane issues by a combination of exploitability, exposure, and concentration of control. A flaw in a system that brokers administrative access or holds broad operational visibility should move ahead of a less critical issue on an ordinary internal server, even if both are technically "internal."

**3. Use compensating controls when patching is delayed.**  
When patching cannot happen immediately, the goal should be to make exploitation harder and post-exploitation movement noisier. That can include restricting management access to dedicated admin subnets or VPN paths, limiting internet reachability, enforcing MFA, reducing standing privileges, disabling unused interfaces and features, and tightening firewall rules around administrative services. In many cases, segmentation and access reduction buy more risk reduction in the short term than waiting for the next maintenance window.

**4. Focus monitoring on the highest-value signals.**  
Monitoring also has to become more selective and more realistic. Organizations with limited staff are unlikely to review everything, so the focus should be on the highest-value signals: administrative logins, configuration changes, creation of new privileged accounts, API activity, changes to federation or authentication settings, and unusual access from systems that do not normally administer infrastructure. Better targeted logging on a small number of critical platforms is often more effective than broad but shallow visibility across everything.

**5. Build patch discipline around planning, not just urgency.**  
Resourcing constraints do not remove the need for patch discipline. They make patch planning more important. NIST’s enterprise patch management guidance emphasizes that patching is a governance and operational planning function, not just a technical task ([NIST SP 800-40 Rev. 4](https://csrc.nist.gov/pubs/sp/800/40/r4/final)). In practice, that means defining in advance which management platforms must be patched on an accelerated timeline, which ones require pre-staged rollback plans, and which temporary controls must be applied when remediation is delayed.

**6. Sequence remediation for constrained teams.**  
For smaller teams, the practical model is not perfection. It is sequencing. First reduce exposure. Then reduce privilege. Then improve detection. Then patch in the highest-impact order possible. That approach is less elegant than a fully mature vulnerability management program, but it is far more realistic under tight time, workforce, and fiscal constraints.

The most efficient path through a hardened environment is often not through the systems doing the work. It is through the systems managing them. Closing that gap starts with treating those platforms not as background infrastructure, but as high-priority security boundaries in their own right.

## References

- **CISA KEV**: [CISA Known Exploited Vulnerabilities search showing VMware-related entries](https://www.cisa.gov/known-exploited-vulnerabilities-catalog?field_date_added_wrapper=all&items_per_page=20&search_api_fulltext=vmware&sort_by=field_date_added)

- **Broadcom advisory**: [VMware Aria Operations security advisory for CVE-2026-22719](https://support.broadcom.com/web/ecx/support-content-notification/-/external/content/SecurityAdvisories/0/36947)

- **CISA ED 24-01**: [Emergency Directive 24-01 for Ivanti Connect Secure and Policy Secure](https://www.cisa.gov/news-events/directives/ed-24-01-mitigate-ivanti-connect-secure-and-ivanti-policy-secure-vulnerabilities)

- **CISA AA24-060B**: [Cybersecurity Advisory on Ivanti exploitation](https://www.cisa.gov/news-events/cybersecurity-advisories/aa24-060b)

- **Palo Alto CVE-2024-3400**: [Palo Alto Networks advisory](https://security.paloaltonetworks.com/CVE-2024-3400)

- **Fortinet FG-IR-24-535**: [Fortinet PSIRT advisory](https://www.fortiguard.com/psirt/FG-IR-24-535)

- **NIST SP 800-40 Rev. 4**: [Guide to Enterprise Patch Management Planning](https://csrc.nist.gov/pubs/sp/800/40/r4/final)

- **CISA KEV Catalog**: [CISA Known Exploited Vulnerabilities Catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)
