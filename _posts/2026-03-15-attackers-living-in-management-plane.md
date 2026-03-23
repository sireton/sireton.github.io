---
title: "Attackers Are Living in Your Management Plane"
date: 2026-03-15
categories: [Blog Posts]
tags: [Threat Research, Red Team, VMware, Management Plane, Exploitation, Opinion, Infrastructure]
description: "CVE-2026-22719 in VMware Aria is the latest in a pattern that's been building for years: the tools you use to manage infrastructure are becoming the primary attack surface."
---

There's a pattern in the vulnerability disclosures over the past couple of years that doesn't get discussed as a pattern often enough. It shows up every few months, a new CVE, a new CISA advisory, a new "actively exploited" designation, and each individual incident gets covered, but the thread connecting them tends to get lost in the noise.

The pattern is this: the management and observability layer of enterprise infrastructure is under systematic, deliberate attack. And most organizations still treat those systems like internal tooling.

## The Latest Example

Earlier this month, CISA added CVE-2026-22719 to the Known Exploited Vulnerabilities catalog. It's a command injection vulnerability in VMware Aria Operations, a platform used for monitoring, performance management, and compliance across virtual infrastructure. The specific flaw enables unauthenticated remote code execution during a support-assisted migration workflow.

Let that sit for a second: unauthenticated RCE in a tool that has visibility into your entire virtualization estate.

Aria Operations sits on top of your vCenter infrastructure. It sees every VM, every configuration, every performance metric. It holds credential stores. It has access to the configuration baselines for your virtualized workloads. Getting RCE on Aria isn't just getting a foothold on one server, it's getting a dashboard view of your environment with potential paths to every virtualized workload in it.

## The Broader Pattern

Aria isn't an anomaly. Look at what's been hitting the KEV catalog consistently over the past 18 months:

Cisco SD-WAN got an authentication bypass in February (CVE-2026-20127, CVSS 10.0) that CISA designated an Emergency Directive, meaning federal agencies had to patch within days. SD-WAN is management-plane infrastructure by definition: it sits in the control path of all branch and cloud traffic.

Ivanti Connect Secure, Ivanti Policy Secure, Palo Alto Networks GlobalProtect, Fortinet FortiOS, all management-layer systems, all with critical actively exploited vulnerabilities in the past year. These aren't obscure components. They're the control plane of enterprise networks.

The pattern extends to backup and recovery infrastructure, SIEM platforms, and endpoint management tools. The common thread is that these systems are treated as internal, trusted, and implicitly lower-risk than the production systems they manage. That's exactly what makes them attractive.

## Why the Management Plane Is the Target

From an attacker's perspective, the management plane is the best possible foothold in a mature environment.

Production workloads are increasingly hardened. EDR is deployed broadly. Network segmentation has improved. Getting persistent, stealthy access to a production server is harder than it used to be, not impossible, but the noise floor is higher. Security teams are watching those systems.

The management plane, by contrast, often runs older software with slower patch cycles. It's accessed by a small number of privileged accounts, so anomalous access is harder to detect against a small baseline. It has legitimate reasons to communicate with every part of the environment, which makes network-level detection harder. And it's classified as "internal" infrastructure, so it frequently sits in network segments with less monitoring than internet-facing systems.

An attacker with code execution on your vCenter or SIEM console has, without exaggeration, a better map of your environment than most of your own team does.

## The Defense Posture Problem

The core issue is how organizations tier their patching and monitoring priorities. The mental model is still something like: internet-facing = high risk, internal = lower risk. That model made more sense in a perimeter-defined world. It doesn't hold in environments where remote access, cloud workloads, and management APIs have made "internal" a fuzzy concept.

A Fortinet VPN appliance sitting at the network edge gets patched immediately when a critical CVE drops. A VMware vCenter or an Ivanti management server sitting "inside" the network gets patched when the next maintenance window comes around. That's the wrong priority ordering given what the last several years of active exploitation have shown.

The monitoring gap is equally significant. Production servers have EDR. They have network monitoring. Management systems often have neither. When they do, the alert thresholds are tuned for the wrong threat model, looking for malware behavior rather than the kind of living-off-the-land, API-driven activity that characterizes sophisticated management plane abuse.

## What This Means Practically

If you're doing a red team engagement or a penetration test against an organization with a mature environment, the management plane is where I'd be spending time. Misconfigured vCenter instances, exposed management APIs, Aria or similar observability platforms with default credentials or unpatched CVEs, these are reliably productive targets in environments that have done everything right on the production side.

From a defensive standpoint, the practical priorities are:

Treat management infrastructure with the same patch urgency as internet-facing systems. The exploitability assumption needs to update, these systems are reachable, they're valuable, and they're actively being targeted.

Extend monitoring coverage. Management systems should have the same or greater logging fidelity as production systems. Authentication events, API calls, and configuration changes are all meaningful signals.

Audit access to management systems as aggressively as production systems. Privileged accounts that can access vCenter, backup platforms, or network management tools are crown jewel targets. MFA, session recording, and regular access reviews apply here too.

The adversaries figured out that the easiest path through a hardened environment is the door marked "management." The defensive side is still catching up to that realization.

---

*Sources: [DIESEC, Top Cybersecurity News March 6](https://diesec.com/2026/03/top-5-cybersecurity-news-stories-march-06-2026/) · [CISA KEV Catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)*
