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

Recent exploitation of platforms like Ivanti Connect Secure reinforces the point. In multiple incidents, initial access was achieved through exposed or unpatched management interfaces, even in environments that had otherwise invested heavily in hardening production systems.

The deeper issue is not just technical exposure, but organizational assumption. These platforms are still often treated as internal infrastructure and therefore as lower priority. In practice, that leads to slower patching, lighter monitoring, and broader trust boundaries than would be acceptable for more obviously exposed systems.

That disconnect between how these systems are perceived and what they actually represent is creating a reliable path for intrusion.

---

## The Latest Example

Earlier this month, CISA added CVE-2026-22719 to the Known Exploited Vulnerabilities catalog. VMware Aria Operations contains a command injection vulnerability that can allow an unauthenticated actor to execute arbitrary commands during a support-assisted product migration workflow.

This is not just another remote code execution bug. It is unauthenticated code execution in a platform designed to monitor, assess, and help manage virtual infrastructure.

Aria Operations sits close to the center of the virtualization stack. It has access to inventory, configuration data, performance telemetry, and, in many deployments, integrations that allow it to interact with other systems across the environment. In practice, that makes it an aggregation point for operational visibility.

Compromising a system like this does not just provide access to one workload. It provides context, visibility, and potentially privileged pathways across the environment.

---

## The Broader Trend

Aria is not an isolated case. The same trend appears across the management and control layer of enterprise environments.

Ivanti Connect Secure became a high-profile example after CISA issued Emergency Directive 24-01 in response to active exploitation. Palo Alto PAN-OS has also faced exploited vulnerabilities involving management interfaces and GlobalProtect-related exposure. Fortinet products continue to appear in KEV as well, including management and security infrastructure that organizations rely on to administer and protect the environment.

These platforms are not peripheral systems. They are administrative gateways, policy engines, orchestration points, and remote access layers. They exist specifically to observe, control, or broker access across the estate.

That is what makes them so valuable to an attacker. A foothold in the management plane is not simply a foothold inside the network. It is a foothold inside the systems that define how the network is operated.

---

## Why the Management Plane Works So Well for Attackers

From an attacker’s perspective, the management plane often offers a more efficient path to meaningful access than traditional production targets.

Production systems are increasingly hardened. EDR coverage is more widespread. Segmentation is more common. Detection strategies are more likely to be tuned around workloads that process business data or support customer-facing services.

Management systems often operate under a different set of assumptions.

They may follow slower patch cycles because of perceived operational sensitivity. They are often accessed by a smaller number of privileged administrators, which narrows the behavioral baseline. They maintain legitimate communication paths across broad portions of the environment, which makes network-based detection more difficult. In some environments, they also receive less endpoint visibility or less tailored monitoring than production workloads.

An attacker with code execution on a management platform is not just inside the network. They are operating from a position of trust, with built-in visibility and administrative reach.

---

## The Defense Posture Gap

The issue is not simply exposure. It is prioritization.

Many organizations still implicitly sort risk into two buckets: internet-facing systems and everything else. That model no longer holds when management interfaces are reachable, administrative APIs are exposed, and control systems aggregate privileged access to large portions of the environment.

In practice, this creates a consistent gap.

Edge-facing systems often receive immediate attention when a critical vulnerability is disclosed. Internal management platforms are more likely to wait for a maintenance window, even when the downstream impact of compromise is greater.

Monitoring follows the same pattern. Production systems often get full telemetry, tuned alerting, and more frequent review. Management platforms may operate with thinner visibility or detection logic that is not designed to identify administrative misuse, low-noise abuse of trusted tools, or suspicious API activity.

Part of that gap is also economic. Once a system is viewed as lower risk, budget, staffing, and operational attention often follow that same assumption. NIST has explicitly framed patching as a cost of doing business and noted that organizations often face internal disagreement about its value and urgency. GAO has likewise found that staffing shortages and logging challenges materially hinder incident response readiness. In other words, the problem is not only that management systems are underestimated. It is also that scarce resources are often allocated elsewhere first.

This is not just a tooling problem. It is a governance and prioritization problem.

---

## What This Means Practically

From an offensive perspective, the management plane is a reliable place to spend time in otherwise mature environments.

Exposed management APIs, observability platforms with weak access controls, unpatched administrative interfaces, and overly trusted orchestration systems can all provide high-value entry points. These weaknesses often exist in environments that have done a much better job hardening production workloads.

From a defensive perspective, the priorities are straightforward.

Treat management infrastructure with the same patch urgency as internet-facing systems. The assumption that these systems are harder to reach or less valuable to an attacker is no longer defensible.

Extend monitoring coverage to management platforms. Authentication activity, administrative actions, API usage, and configuration changes should be logged and reviewed with high fidelity.

Apply strict access controls. Accounts with access to management systems are high-impact targets and should be protected accordingly with MFA, restricted access paths, session visibility, and regular access review.

The most efficient path through a hardened environment is often not through the systems doing the work. It is through the systems managing them.

Attackers have already adjusted to that reality. Defensive strategy is still catching up.

---

## References

1. CISA Known Exploited Vulnerabilities Catalog  
   [https://www.cisa.gov/known-exploited-vulnerabilities-catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)

2. CISA KEV entry including VMware Aria Operations / CVE-2026-22719  
   [https://www.cisa.gov/known-exploited-vulnerabilities-catalog?field_date_added_wrapper=all&items_per_page=20&search_api_fulltext=vmware&sort_by=field_date_added](https://www.cisa.gov/known-exploited-vulnerabilities-catalog?field_date_added_wrapper=all&items_per_page=20&search_api_fulltext=vmware&sort_by=field_date_added)

3. NVD entry for CVE-2026-22719  
   [https://nvd.nist.gov/vuln/detail/CVE-2026-22719](https://nvd.nist.gov/vuln/detail/CVE-2026-22719)

4. Broadcom advisory for VMware Aria Operations  
   [https://support.broadcom.com/web/ecx/support-content-notification/-/external/content/SecurityAdvisories/0/36947](https://support.broadcom.com/web/ecx/support-content-notification/-/external/content/SecurityAdvisories/0/36947)

5. CISA Emergency Directive 24-01 on Ivanti Connect Secure and Policy Secure  
   [https://www.cisa.gov/news-events/directives/ed-24-01-mitigate-ivanti-connect-secure-and-ivanti-policy-secure-vulnerabilities](https://www.cisa.gov/news-events/directives/ed-24-01-mitigate-ivanti-connect-secure-and-ivanti-policy-secure-vulnerabilities)

6. CISA advisory on threat actors exploiting Ivanti vulnerabilities  
   [https://www.cisa.gov/news-events/cybersecurity-advisories/aa24-060b](https://www.cisa.gov/news-events/cybersecurity-advisories/aa24-060b)

7. Palo Alto Networks advisory for PAN-OS management web interface authentication bypass  
   [https://security.paloaltonetworks.com/CVE-2024-0012](https://security.paloaltonetworks.com/CVE-2024-0012)

8. Palo Alto Networks advisory for CVE-2024-3400 in the GlobalProtect feature of PAN-OS  
   [https://security.paloaltonetworks.com/CVE-2024-3400](https://security.paloaltonetworks.com/CVE-2024-3400)

9. NIST SP 800-40 Rev. 4, Guide to Enterprise Patch Management Planning  
   [https://csrc.nist.gov/pubs/sp/800/40/r4/final](https://csrc.nist.gov/pubs/sp/800/40/r4/final)

10. NIST announcement summarizing patching as a cost of doing business  
    [https://www.nist.gov/news-events/news/2022/04/final-publications-enterprise-patch-management-released](https://www.nist.gov/news-events/news/2022/04/final-publications-enterprise-patch-management-released)

11. GAO-24-105658, Cybersecurity: Federal Agencies Made Progress, but Need to Fully Implement Incident Response Requirements  
    [https://www.gao.gov/products/gao-24-105658](https://www.gao.gov/products/gao-24-105658)
- Palo Alto Networks Security Advisories (GlobalProtect)  
  https://security.paloaltonetworks.com/

- Fortinet PSIRT Advisories (FortiOS vulnerabilities)  
  https://www.fortiguard.com/psirt
