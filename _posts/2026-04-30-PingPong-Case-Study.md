---
title: "PingPong - A Case Study in Multi-Forest Trust Collapse and ADCS Exploitation"
date: 2026-04-30
categories:
  - Case Studies
  - Labs
tags: [active-directory, adcs, kerberos, multi-forest, privilege-escalation, threat-research, red-team, blue-team]
description: "This case study traces how PKI trust, cross-forest ACL inheritance, gMSA credential scope, JEA artifact hygiene, and service account privilege design each failed independently before failing together."
---
 
![](/assets/img/PingPong/pingpong%20complete.png)

**Source:** [Hack The Box: PingPong](https://app.hackthebox.com/machines/PingPong)

---
  
PingPong is a Season 10 Windows machine rated Insane, built around a bidirectional Active Directory forest trust and a PKI infrastructure that fails at both ends of the engagement chain. Starting from an assumed-breach credential set, the path chains ADCS template abuse, cross-forest ACL exploitation, gMSA credential theft, JEA artifact disclosure, Kerberos delegation abuse, token impersonation, DCSync, and a second ADCS stage to reach domain administrator across both forests. A full engagement report will be published after the machine retires.

*This is a spoiler-safe engagement analysis of Hack The Box's PingPong (Season 10, Insane, Windows, Active). Flags, credentials, exact commands, endpoint paths, payloads, usernames, and machine-specific identifiers are intentionally omitted. The techniques, control failures, and threat parallels discussed are real. The progressive analysis and research is the lesson.*
 
---
 
## The Engagement 
 
PingPong doesn't hand you a path. From the first moment I had connectivity, it was clear this was going to require building a mental model of the environment before anything useful could happen. Two forest environments. A bidirectional trust relationship. Separate domain controllers, separate PKI infrastructure, and a service account design that crossed the forest boundary in ways that turned out to matter enormously.
 
I treated this the same way I'd treat a real engagement: enumerate deliberately, form hypotheses, test assumptions in order of confidence. My goal wasn't to find the first thing that looked broken. It was to understand what the environment was supposed to do, so that when something deviated from that model, I'd recognize why it mattered rather than just stumbling into it.

 The following is a phase-by-phase account of how I mapped the model to real world configurations and where it broke down. The controls here weren't absent. They were present and individually reasonable but collectively insufficient. PKI trust, cross-forest ACL design, service account privilege scope, and administrative interface artifact hygiene each failed in ways that only become visible when the full trust surface is evaluated as a trust system rather than a collection of independent components. The final section returns to that argument with the specific lessons the path produced.
 
---
 
## Phase 1: PKI Enumeration and First Trust Failure
 
**Attack Classification:** [Initial Access via Certificate Template Abuse (ESC13)](https://posts.specterops.io/adcs-esc13-abuse-technique-fda4272fbd53), [T1649](https://attack.mitre.org/techniques/T1649/) [[2]](https://attack.mitre.org/techniques/T1649/)
 
The first thing I reached for after validating my assumed-breach credentials was the certificate authority. ADCS gets audited less frequently than domain controllers, but it holds equivalent trust in the environment. If the CA issues a certificate to someone who shouldn't have one, every relying system accepts it without question, because the CA said it was valid.
 
I enumerated the available certificate templates looking for the standard misconfiguration markers: permissive enrollment rights, SAN control granted to the enrollee, Extended Key Usage scope that doesn't match the template's stated purpose, and write ACLs held by non-PKI-administrative accounts. What I found fell into the ESC13 category [[3]](https://posts.specterops.io/adcs-esc13-abuse-technique-fda4272fbd53), a template whose EKU configuration allowed an issued certificate to be used for a purpose beyond what the template was designed for. The enrollment rights were broader than they should have been, and the issued certificate opened a service door that my credential set alone wouldn't have.
 
I requested the certificate, used it to authenticate, and had a working WinRM session on the primary domain controller. The CA validated my request and issued the material. The DC accepted it without complaint. That's the point: the failure isn't in the downstream systems. It's in the CA, and because the CA is trusted, every relying party inherits that failure.
 
This isn't a theoretical risk. In April 2022, Mandiant reported that APT29 (Cozy Bear) exploited misconfigured certificate templates to impersonate admin users, requesting certificates as low-privileged users and specifying high-privileged accounts in the Subject Alternative Name (SAN) field, allowing them to authenticate as those accounts. [[4]](https://www.semperis.com/blog/esc1-attack-explained/) In April 2024, Google Cloud reported that UNC5330 used an LDAP bind account to exploit a vulnerable Windows certificate template, creating a computer object and impersonating a domain administrator. [[5]](https://cloud.google.com/blog/topics/threat-intelligence/ivanti-post-exploitation-lateral-movement) The technique is in active use by nation-state actors. PingPong models it accurately.
 
> **Note Worthy:** CA request logs are where a defender would have seen this. A certificate request from a non-standard principal type, or one with a SAN value inconsistent with the requestor's identity, is anomalous. The event data exists. In most organizations it sits in compliance storage and nothing alerts on it.
{: .prompt-tip }
 
---
 
## Phase 2: Mapping the Forest Trust
 
**Attack Classification:** [Domain Trust Discovery (T1482)](https://attack.mitre.org/techniques/T1482/)
 
Once I was on the primary DC, I didn't want to spend too much time digging deeper into that forest as I was still building the mental map of the trust relationship so I needed to look outward. The bidirectional trust was the most interesting structural fact about this environment, and I wanted to understand it before committing to any specific path forward.
 
I ran trust enumeration against both forests, built out the topology, and started asking the question that matters in any multi-forest environment: where does trust flow, and where is it not enforced? A bidirectional trust means authentication can move in both directions. Whether an attacker can use that is a function of the access controls on each side. Those controls are often designed with the individual forests in mind. They're rarely designed with the cross-forest composition in mind.
 
I used BloodHound-style graph analysis to map the paths. Not to find a direct route to domain admin, but to understand the permission landscape. What groups in the trusted forest controlled access to sensitive resources? Which of those groups had ACLs reachable from a principal in the primary forest? Where did the trust model have gaps the forest architects hadn't explicitly accounted for?
 
That work is slow and deliberate, and it's also what separates a functional attack chain from a lucky stumble. Mandiant documented this same reconnaissance pattern from APT29 (Cozy Bear) during their analysis of the European diplomatic entity incident, noting numerous LDAP queries with atypical properties performed against the Active Directory system as the attacker mapped the environment. [[6]](https://cloud.google.com/blog/topics/threat-intelligence/apt29-windows-credential-roaming/)
 
---
 
## Phase 3: Cross-Forest gMSA Abuse
 
**Attack Classification:** [Credential Access via gMSA Group Membership Manipulation / ACL Abuse (T1556)](https://attack.mitre.org/techniques/T1556/)
 
The cross-forest enumeration surfaced a group-managed service account in the trusted forest. gMSAs are a meaningful security improvement over traditional service accounts: passwords rotate automatically, and credential readability is restricted to principals in a designated reader group. [[7]](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/group-managed-service-accounts/group-managed-service-accounts/group-managed-service-accounts-overview) The model is sound.
 
What I found was that the reader group controlling access to this gMSA's managed password had an ACL that a principal reachable from the primary forest could write to. Specifically, the ACL allowed a foreign principal to add members to that group.
 
- This is a cross-forest ACL misconfiguration: the group lives in the trusted forest, but control over its membership extends into the primary forest, and nobody had modeled what that implied.
I added the appropriate principal to the reader group from the primary forest side, then queried the gMSA's `msDS-ManagedPassword` attribute from the trusted forest and obtained the managed password material. The gMSA's credential was mine.
 
The failure is subtle and worth reviewing for a moment. The gMSA password protection worked exactly as designed. The group membership controlled access exactly as designed. The misconfiguration was entirely in the ACL that governed who could modify that group from across the trust boundary. Reviewed in isolation within the trusted forest, every control looks correct. The break only appears when the cross-forest ACL surface is modeled holistically. Enterprise AD security reviews are almost never scoped that way.
 
> **Note Worthy:** Any read of `msDS-ManagedPassword` by a principal outside the designated reader group should be an immediate alert. Changes to the membership of groups that control gMSA access should be monitored as write-to-sensitive-group events. Cross-forest group membership changes specifically represent trust boundary modifications that most defenders have never baselined.
{: .prompt-tip }
 
---
 
## Phase 4: JEA and the Credential Left Behind
 
**Attack Classification:** [Credential Access via Unsecured Credentials in Files (T1552.001)](https://attack.mitre.org/techniques/T1552/001/) / JEA Endpoint Abuse
 
The gMSA had access to a Just Enough Administration (JEA) endpoint on the primary domain controller. JEA is a PowerShell remoting configuration designed to restrict what a connecting principal can execute. [[8]](https://learn.microsoft.com/en-us/powershell/scripting/learn/remoting/jea/overview) The endpoint I was looking at had done its job: the command surface was constrained to a limited set of operations, and I couldn't run arbitrary code through it.
 
What I could do was read.
 
PowerShell maintains a persistent history file for each user context. The file is written to a known path, it persists across sessions, and it isn't scrubbed when a JEA session terminates. A prior session in this service account context had executed commands that included credential material, and the history file still had it. I read the file through the JEA session and recovered credentials for a user account in the trusted forest.
 
I want to be precise about why this matters defensively. JEA worked. The execution restriction it was designed to provide was in place. The failure was a category error in the security model: the assumption was that constraining what an interface can execute also constrains what information it exposes. Those are different properties. Execution restrictions don't sanitize the file system context. Artifacts persist. Credentials left in history files remain accessible to any principal with read access to the relevant path, regardless of what the interface will let them run.
 
Mandiant documented an analogous failure when analyzing APT29's (Cozy Bear) operations against a European diplomatic entity, where the attack stood out for the abuse of Windows Credential Roaming, a feature used to persist certificates and credential material with the user across a domain. [[6]](https://cloud.google.com/blog/topics/threat-intelligence/apt29-windows-credential-roaming/) The specific mechanism differs from what I found here, but the category is identical: credential artifacts persisted in a location that the interface restriction didn't cover, and those artifacts were accessible to any principal who could reach the path.
 
> **Note Worthy:** File access telemetry is the detection opportunity here. A service account reading a user's PSReadLine history file has no legitimate operational purpose. Sysmon Event ID 11 [[9]](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon) captures file creation, and file access events from unexpected security contexts against known history file paths are detectable if the Sysmon configuration targets them. Most don't.
{: .prompt-tip }
 
---
 
## Phase 5: Delegation Abuse Through the Database Layer
 
**Attack Classification:** [Lateral Movement via Resource-Based Constrained Delegation (T1550.003)](https://attack.mitre.org/techniques/T1550/003/) / S4U2Self + S4U2Proxy
 
The credentials I recovered in Phase 4 belonged to a user account with `GenericWrite` access on a database service account in the trusted forest. `GenericWrite` on a service account object is the permission required to configure resource-based constrained delegation (RBCD). With RBCD configured, the gMSA could request Kerberos service tickets on behalf of other users to that database service, impersonating them in that context.
 
I configured the RBCD relationship, then used the gMSA to request a service ticket impersonating a user with database administrator rights via two chained Kerberos extensions:
 
- **S4U2Self**: allows a service to request a ticket to itself on behalf of an arbitrary user
- **S4U2Proxy**: uses that ticket to request access to another service, carrying the impersonated user's identity
Chained together, those two extensions produced a service ticket to the database service as a privileged database user, and I had interactive access.
 
The database service was running with `SeImpersonatePrivilege` and had command execution capability enabled. That combination was the pivot point to local system access, which I'll cover in Phase 6. But I want to pause here on the delegation piece, because the control failure is significant.
 
RBCD is presented as a safer alternative to unconstrained delegation, and it is. But "safer" doesn't mean "safe by default." The ability to configure RBCD on a service account is a privilege. `GenericWrite` on a service account object should be treated as a privileged access grant, not an administrative convenience. When that permission exists because of a permissions design oversight rather than an intentional grant, and when it's combined with a gMSA credential obtained through a cross-forest ACL gap, the chain from assumed-breach to database administrator is shorter than any enterprise architect would expect.
 
In May 2024, the Ascension Health system suffered a ransomware attack that originated in part through Kerberos service account credential abuse, and IBM's 2025 X-Force Threat Intelligence Index reported that 30% of all intrusions in 2024 involved stolen or abused credentials as the leading entry point. [[10]](https://www.blackfog.com/kerberoasting-attack-explained/) The specific technique differs from what I used here, but the control failure is structurally identical: service accounts with insufficient privilege isolation and insufficient authentication monitoring create a path from lateral access to administrative compromise.
 
---
 
## Phase 6: Token Impersonation and the Potato Problem
 
**Attack Classification:** [Privilege Escalation via Access Token Manipulation (T1134.001)](https://attack.mitre.org/techniques/T1134/001/) / SeImpersonatePrivilege Abuse
 
From my database service context, I had `SeImpersonatePrivilege`. This is a default grant on most SQL Server and IIS service accounts in Windows. It exists because those services legitimately need to impersonate clients, and it's also the primitive the entire Potato family of privilege escalation tools is built around.
 
The concept is straightforward: coerce a SYSTEM-level service to authenticate to an endpoint you control, capture the resulting access token, and use it to create a new process in the SYSTEM security context. The Potato family has evolved through successive iterations, each adapting to Microsoft's attempts to close specific coercion paths:
 
- **RottenPotato**: the original primitive, targeting Windows token impersonation via NTLM reflection
- **JuicyPotato**: broadened COM server targeting for wider Windows version coverage
- **SweetPotato**: adapted for environments where earlier variants were patched
- **GodPotato**: targets the DCOM activation service, effective across modern Windows Server versions [[11]](https://www.ionsec.io/resources/reverse-engineering-godpotato-msascui-exe)

The underlying privilege is the same across all of them. `SeImpersonatePrivilege` is often granted to web and database services by default. That's the critical observation: this isn't a misconfiguration someone made. It's a Windows default. A service account running SQL Server holds this privilege because Microsoft's design expects it to. An attacker who reaches code execution in that service context gets the primitive for free.
 
I ran the escalation and had a SYSTEM shell on the trusted forest's domain controller within minutes of having database access. The escalation was a consequence of the privilege design, not a separate vulnerability.
 
> **Note Worthy:** Process creation telemetry is the detection opportunity here. Sysmon Event ID 1 [[9]](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon), filtered for child processes spawned by database service accounts with elevated integrity levels, is the signal. A privileged subprocess with a database service parent has no legitimate explanation.
{: .prompt-tip }
 
---
 
## Phase 7: Credential Extraction via DCSync
 
**Attack Classification:** [Credential Dumping via Directory Replication (T1003.006)](https://attack.mitre.org/techniques/T1003/006/)
 
With SYSTEM on the trusted forest's domain controller, I ran a DCSync operation. DCSync abuses the Directory Replication Service remote protocol to request domain password hashes without requiring interactive access to the DC. Any principal with `Replicating Directory Changes` and `Replicating Directory Changes All` permissions can make the request. A local SYSTEM session on the DC itself satisfies that requirement trivially.
 
I wasn't trying to dump every account. I was looking for a specific one: a user in the trusted forest who held CA Managers rights in the primary forest. The cross-forest trust topology I'd mapped in Phase 2 told me that account existed. The DCSync gave me its credential material.
 
This is the moment where the path snapped into focus. The entire lateral movement chain through the trusted forest was in service of obtaining this one set of credentials. CA Managers in the primary forest can modify certificate templates, and with that access the second ADCS stage, and the end of the engagement, was visible.
 
APT29 (Cozy Bear) maintains a documented toolset that includes BloodHound, Mimikatz, Rubeus, and SharpView, tools that collectively enable credential extraction and domain replication abuse at scale. [[12]](https://cyble.com/threat-actor-profiles/apt-29/) DCSync is a standard component of that arsenal, not because it's novel but because it works consistently in environments where the detection doesn't exist.
 
> **Note Worthy:** Windows generates Event ID 4662 [[13]](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/audit-policy-recommendations) for directory replication requests. A DrsGetNCChanges request from a non-DC source is definitionally anomalous. The detection rule is trivial to write. In most environments, nobody has written it.
{: .prompt-tip }
 
---
 
## Phase 8: ESC4 and the Close
 
**Attack Classification:** [Privilege Escalation via Certificate Template ACL Abuse (ESC4)](https://posts.specterops.io/certified-pre-owned-d95910965cd2), [T1649](https://attack.mitre.org/techniques/T1649/)
 
CA Managers access in the primary forest meant I could modify certificate template attributes. A template with write-accessible ACLs held by a non-PKI-administrative principal is the ESC4 misconfiguration: I could rewrite the template's enrollment rules, its EKU scope, and its SAN behavior, then request a certificate under those modified rules to authenticate as any principal in the forest.
 
I modified a template to permit a Subject Alternative Name specifying the domain administrator's UPN. I requested a certificate. The CA issued it. I used PKINIT to obtain a Kerberos TGT for the domain administrator account and authenticated to the primary DC via WinRM.
 
The engagement started with a misconfigured certificate template and ended with a misconfigured certificate template. PKI was the opening move and the closing move. This engagement used two ADCS techniques specifically: ESC13 for initial access and ESC4 for the final forest-level escalation. That symmetry isn't accidental. It reflects the reality that ADCS is the most underaudited high-privilege component in most Active Directory environments. The broader research community has now catalogued 16 distinct ADCS privilege escalation techniques (ESC1 through ESC16), and recent research from BeyondTrust presented at SO-CON 2025 demonstrated that these on-premises misconfigurations can lead to full compromise of cloud-based infrastructure in hybrid deployments. [[14]](https://www.catonetworks.com/blog/cato-ctrl-preventing-privilege-escalation-via-active-directory-certificate-services-adcs/)
 
> **Note Worthy:** Event ID 4899 [[13]](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/audit-policy-recommendations) is generated on the CA when a certificate template is modified. A template modification followed within minutes by a certificate request from a new principal type is a near-certain indicator of ESC4 in progress. The telemetry exists. The alert does not. {: .prompt-tip }
 
---
 
## Final Thoughts
 
Eight phases, two forests, two PKI exploitation stages at opposite ends of the chain, and one assumed-breach credential set that became domain administrator across both environments.
 
When I look back at the path, what stands out isn't any single technique. It's how many of the controls that failed were correctly designed for the scope they were built to address. The gMSA password was protected. The JEA endpoint restricted execution. The certificate template had an intended purpose. The database service delegation was constrained rather than unconstrained. Each of those represents a real security improvement over a worse baseline. None of them held up when evaluated across the full trust surface.
 
This is a lesson that transfers directly to real security work. The controls are often present. The gap is in the composition. Nobody threat-modeled what happens when a cross-forest ACL gap meets a gMSA reader group meets a JEA artifact meets a delegation misconfiguration meets an ADCS template with write access. Each link in that chain looked acceptable in isolation. The chain was the vulnerability.
 
APT29 (Cozy Bear) has demonstrated exactly this methodology in documented real-world campaigns: certificate template abuse for initial access [[4]](https://www.semperis.com/blog/esc1-attack-explained/), credential artifact exploitation for lateral movement [[6]](https://cloud.google.com/blog/topics/threat-intelligence/apt29-windows-credential-roaming/), and domain replication for credential extraction [[12]](https://cyble.com/threat-actor-profiles/apt-29/). The techniques aren't unique to a Hack The Box machine. They're the operational playbook of a well-resourced threat actor operating against enterprise Windows environments today.
 
The way to break a chain like this isn't to find one link and call it fixed. It's to audit the trust model holistically: across both forests, across the PKI layer, across the service account permission design, and across the application artifact surface. That audit has to include questions that single-forest security reviews don't ask.
 
Audit the PKI. Model the forest trust. Restrict the service accounts. Monitor the transitions. [[15]](https://csrc.nist.gov/publications/detail/sp/800-92/final) That is the work.
 
---
 
## Works Cited
 
1. Hack The Box: PingPong Machine: [https://app.hackthebox.com/machines/PingPong](https://app.hackthebox.com/machines/PingPong)
2. MITRE ATT&CK, T1649: Steal or Forge Authentication Certificates: [https://attack.mitre.org/techniques/T1649/](https://attack.mitre.org/techniques/T1649/)
3. SpecterOps, ADCS ESC13 Abuse Technique (Jonas Bülow Knudsen): [https://posts.specterops.io/adcs-esc13-abuse-technique-fda4272fbd53](https://posts.specterops.io/adcs-esc13-abuse-technique-fda4272fbd53)
4. Semperis, ESC1 Attack Explained (APT29 April 2022 Mandiant incident context): [https://www.semperis.com/blog/esc1-attack-explained/](https://www.semperis.com/blog/esc1-attack-explained/)
5. Google Cloud / Mandiant, Cutting Edge Part 4: Ivanti Connect Secure VPN Post-Exploitation Lateral Movement Case Studies (UNC5330 ADCS abuse): [https://cloud.google.com/blog/topics/threat-intelligence/ivanti-post-exploitation-lateral-movement](https://cloud.google.com/blog/topics/threat-intelligence/ivanti-post-exploitation-lateral-movement)
6. Google Cloud / Mandiant, They See Me Roaming: Following APT29 by Taking a Deeper Look at Windows Credential Roaming: [https://cloud.google.com/blog/topics/threat-intelligence/apt29-windows-credential-roaming/](https://cloud.google.com/blog/topics/threat-intelligence/apt29-windows-credential-roaming/)
7. Microsoft, Group Managed Service Accounts Overview: [https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/group-managed-service-accounts/group-managed-service-accounts/group-managed-service-accounts-overview](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/group-managed-service-accounts/group-managed-service-accounts/group-managed-service-accounts-overview)
8. Microsoft, Just Enough Administration (JEA): [https://learn.microsoft.com/en-us/powershell/scripting/learn/remoting/jea/overview](https://learn.microsoft.com/en-us/powershell/scripting/learn/remoting/jea/overview)
9. Microsoft, Sysmon Documentation: [https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
10. BlackFog, Kerberoasting Attack Explained (Ascension Health 2024 breach and IBM X-Force 2025 context): [https://www.blackfog.com/kerberoasting-attack-explained/](https://www.blackfog.com/kerberoasting-attack-explained/)
11. IonSec, Reverse Engineering GodPotato (SeImpersonatePrivilege, DCOM abuse, and Potato family analysis): [https://www.ionsec.io/resources/reverse-engineering-godpotato-msascui-exe](https://www.ionsec.io/resources/reverse-engineering-godpotato-msascui-exe)
12. Cyble, APT29 Threat Profile (toolset and TTP documentation): [https://cyble.com/threat-actor-profiles/apt-29/](https://cyble.com/threat-actor-profiles/apt-29/)
13. Microsoft, Audit Policy Recommendations (Windows Security Event Reference): [https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/audit-policy-recommendations](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/audit-policy-recommendations)
14. Cato CTRL, Preventing Privilege Escalation via ADCS (ESC1 through ESC16, BeyondTrust SO-CON 2025 research): [https://www.catonetworks.com/blog/cato-ctrl-preventing-privilege-escalation-via-active-directory-certificate-services-adcs/](https://www.catonetworks.com/blog/cato-ctrl-preventing-privilege-escalation-via-active-directory-certificate-services-adcs/)
15. NIST SP 800-92, Guide to Computer Security Log Management: [https://csrc.nist.gov/publications/detail/sp/800-92/final](https://csrc.nist.gov/publications/detail/sp/800-92/final)
## Additional Resources
 
- SpecterOps, Certified Pre-Owned: Abusing Active Directory Certificate Services (foundational ESC1-ESC8 research): [https://specterops.io/assets/resources/Certified_Pre-Owned.pdf](https://specterops.io/assets/resources/Certified_Pre-Owned.pdf)
- SpecterOps, Certified Pre-Owned blog post (ESC4 overview): [https://posts.specterops.io/certified-pre-owned-d95910965cd2](https://posts.specterops.io/certified-pre-owned-d95910965cd2)
- MITRE ATT&CK, T1482 Domain Trust Discovery: [https://attack.mitre.org/techniques/T1482/](https://attack.mitre.org/techniques/T1482/)
- MITRE ATT&CK, T1552.001 Unsecured Credentials: Credentials in Files: [https://attack.mitre.org/techniques/T1552/001/](https://attack.mitre.org/techniques/T1552/001/)
- MITRE ATT&CK, T1550.003 Use Alternate Authentication Material: Pass the Ticket: [https://attack.mitre.org/techniques/T1550/003/](https://attack.mitre.org/techniques/T1550/003/)
- MITRE ATT&CK, T1134.001 Access Token Manipulation: Token Impersonation/Theft: [https://attack.mitre.org/techniques/T1134/001/](https://attack.mitre.org/techniques/T1134/001/)
- MITRE ATT&CK, T1003.006 OS Credential Dumping: DCSync: [https://attack.mitre.org/techniques/T1003/006/](https://attack.mitre.org/techniques/T1003/006/)
- MITRE ATT&CK, T1556 Modify Authentication Process: [https://attack.mitre.org/techniques/T1556/](https://attack.mitre.org/techniques/T1556/)
- Microsoft, Active Directory Certificate Services Documentation: [https://learn.microsoft.com/en-us/windows-server/identity/ad-cs/](https://learn.microsoft.com/en-us/windows-server/identity/ad-cs/)
 
