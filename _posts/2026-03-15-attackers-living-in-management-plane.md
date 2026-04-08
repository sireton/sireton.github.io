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

They may follow slower patch cycles because of perceived operational sensitivity. They are often accessed by a smaller number of privileged administrators, which narrows the behavioral baseline. They also maintain legitimate communication paths across broad portions of the environment, which makes suspicious activity harder to distinguish from expected administration. NIST’s enterprise patch management guidance explicitly frames patching as a cost of doing business and a necessary preventive maintenance function, reinforcing the point that delayed patching is usually a prioritization problem, not just a technical one ([NIST SP 800-40 Rev. 4](https://csrc.nist.gov/pubs/sp/800/40/r4/final)).

An attacker with code execution on a management platform is not just inside the network. They are operating from a position of trust, with built-in visibility and administrative reach.

## The Defense Posture Gap

The issue is not simply exposure. It is prioritization.

Many organizations still sort risk into two rough categories: internet-facing systems and everything else. That model no longer holds when management interfaces are reachable, administrative APIs are exposed, and control systems aggregate privileged access across large portions of the environment.

In practice, this creates a consistent gap. Edge-facing systems often receive immediate attention when a critical vulnerability is disclosed. Internal management platforms are more likely to wait for a maintenance window, even when the downstream impact of compromise may be greater. The repeated appearance of management and remote access technologies in CISA’s Known Exploited Vulnerabilities catalog is one indication that this older prioritization model is no longer sufficient ([CISA KEV Catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)).

Monitoring often follows the same logic. Production systems tend to receive fuller telemetry, more mature alert tuning, and more frequent review. Management platforms may operate with thinner visibility or detection logic that is not designed to catch low-noise abuse of trusted tools and administrative interfaces.

Part of that gap is also economic. Once a system is viewed as lower risk, budget, staffing, and operational attention often follow the same assumption. NIST’s patch management guidance does not reduce this to money alone, but it does make clear that patching requires enterprise planning, governance, and sustained operational commitment. In other words, the problem is not only that management systems are underestimated. It is also that scarce resources are often allocated elsewhere first ([NIST SP 800-40 Rev. 4](https://csrc.nist.gov/pubs/sp/800/40/r4/final)).

This is not just a tooling problem. It is a governance and prioritization problem.

## What This Means Practically

From an offensive perspective, the management plane is a reliable place to spend time in otherwise mature environments.

Exposed management APIs, observability platforms with weak access controls, unpatched administrative interfaces, and overly trusted orchestration systems can all provide high-value entry points. These weaknesses often persist in environments that have done a much better job hardening production workloads.

From a defensive perspective, the priorities are straightforward.

Treat management infrastructure with the same patch urgency as internet-facing systems. The assumption that these systems are harder to reach or less valuable to an attacker is no longer defensible. Extend monitoring coverage to management platforms so authentication activity, administrative actions, API usage, and configuration changes are logged and reviewed with high fidelity. Apply strict access controls to accounts that can reach these systems, because those accounts represent disproportionate risk if they are compromised ([NIST SP 800-40 Rev. 4](https://csrc.nist.gov/pubs/sp/800/40/r4/final); [CISA KEV Catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)).

## References

- **CISA KEV**: CISA Known Exploited Vulnerabilities search showing VMware-related entries  
  https://www.cisa.gov/known-exploited-vulnerabilities-catalog?field_date_added_wrapper=all&items_per_page=20&search_api_fulltext=vmware&sort_by=field_date_added

- **Broadcom advisory**: VMware Aria Operations security advisory for CVE-2026-22719  
  https://support.broadcom.com/web/ecx/support-content-notification/-/external/content/SecurityAdvisories/0/36947

- **CISA ED 24-01**: Emergency Directive 24-01 for Ivanti Connect Secure and Policy Secure  
  https://www.cisa.gov/news-events/directives/ed-24-01-mitigate-ivanti-connect-secure-and-ivanti-policy-secure-vulnerabilities

- **CISA AA24-060B**: Cybersecurity Advisory on Ivanti exploitation  
  https://www.cisa.gov/news-events/cybersecurity-advisories/aa24-060b

- **Palo Alto CVE-2024-3400**: Palo Alto Networks advisory  
  https://security.paloaltonetworks.com/CVE-2024-3400

- **Fortinet FG-IR-24-535**: Fortinet PSIRT advisory  
  https://www.fortiguard.com/psirt/FG-IR-24-535

- **NIST SP 800-40 Rev. 4**: Guide to Enterprise Patch Management Planning  
  https://csrc.nist.gov/pubs/sp/800/40/r4/final

- **CISA KEV Catalog**: CISA Known Exploited Vulnerabilities Catalog  
  https://www.cisa.gov/known-exploited-vulnerabilities-catalog
