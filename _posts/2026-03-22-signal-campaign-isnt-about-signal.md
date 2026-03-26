---
title: "The Signal Campaign Isn't About Signal"
date: 2026-03-22
categories: [Blog Posts]
tags: [Threat Intelligence, Social Engineering, Russia, Signal, Phishing, Opinion, OPSEC]
description: "Russian intelligence isn't breaking Signal's encryption. They're breaking the humans who trust it by exploiting one assumption: that encrypted means secure"
---

This week the FBI and CISA issued a joint advisory confirming what Dutch and German intelligence had been warning about for weeks: Russian intelligence services are running a coordinated campaign targeting Signal and WhatsApp users. Thousands of accounts compromised. Government officials, military personnel, journalists, politicians.
The coverage that followed asked predictable questions. Is Signal safe? Is end-to-end encryption broken? Should people stop using it? Those questions dominated the news cycle and almost entirely missed the point.
Signal's encryption is fine. It was never the target. What Russian intelligence is actually doing is simpler, more instructive, and significantly harder to patch: they are exploiting the gap between what people believe encrypted communication protects and what it actually protects. The cryptography held. The mental model didn't. Here's what that actually means.

## What's Actually Happening

What they're doing is simpler than the coverage suggests, and it runs in two distinct stages.

**Stage 1: Credential phishing.** Attackers impersonate Signal support accounts, sending targets messages about suspicious activity or a "possible data leak" and asking them to share their SMS verification code and PIN. If the target complies, the attacker registers the account on a new device. Past messages are gone. The account is now theirs.

**Stage 2: Silent device linking.** Signal has a legitimate feature called "linked devices" that lets you connect additional devices to your account, so your desktop can receive messages alongside your phone. Attackers are sending malicious QR codes disguised as group invite links, security alerts, or legitimate pairing instructions. When a target scans one, they unknowingly add the attacker's device to their account. The attacker can read messages in real time. The victim keeps full access and has no indication anything changed.

*Source: [FBI/CISA Joint Advisory – Compromise of Signal Messaging Application](https://www.cisa.gov)*

APT44 (Sandworm) has taken this further. Dutch intelligence confirmed they have been using devices captured on the battlefield in Ukraine, phones taken from compromised military personnel, to conduct further exploitation via Signal. Those devices were already trusted contacts in target accounts, making the access invisible.

*Source: [AIVD/MIVD Advisory – Russian Digital Espionage Against Signal](https://www.aivd.nl)*

## The Lesson Isn't New

End-to-end encryption protects the message in transit. It does not protect the account. Those are different threat surfaces, and conflating them is exactly what this campaign exploits.

The distinction holds across every tool in the stack. Encrypted email is useless if the attacker has your password. A password manager doesn't help if they have your master credential. The encryption is irrelevant once someone else controls the account.

The advisory names "current and former U.S. government officials" among the compromised. These are not unsophisticated users. The problem is that encryption becomes a psychological anchor. "Uses Signal" gets treated as a proxy for "communication is secure," and that shortcut is what the attackers are targeting.

The campaign started in Ukraine for a reason. Russian intelligence was specifically going after military personnel who had adopted Signal as an upgrade from unencrypted channels. The tools the defenders chose became the attack surface, not because the tools were bad, but because the mental model never caught up with the technology.

## Why This Matters Beyond Espionage

The targets in the advisory are high-value intelligence targets: government officials, military personnel, journalists. Most people reading this aren't in that category.

But the technique scales. The QR code phishing attack in particular requires no nation-state resources. It requires a convincing message and a target who scans without thinking. The same playbook refined against Ukrainian military targets will appear in criminal toolkits, and in some cases already has.

WhatsApp had a variant called "GhostPairing" documented last year, the same device-linking abuse applied broadly. These techniques move from state actors to criminal groups on a compressed timeline. The advisory describes a state-sponsored campaign. What follows is a commoditised one.

*Source: [Malwarebytes – Signal and WhatsApp Accounts Targeted in Phishing Campaign](https://www.malwarebytes.com/blog/news/2026/03/signal-and-whatsapp-accounts-targeted-in-phishing-campaign)*

## What Actually Protects You

The fixes are straightforward. The advisory is clear on all of them.

**Never share your verification code or PIN with anyone.** Signal and WhatsApp will never ask for these via an in-app message. The only legitimate use of your SMS verification code is when you install the app fresh. Any message asking for it is a phishing attempt, regardless of how official it looks.

**Check your linked devices regularly.** In Signal: Settings → Linked Devices. In WhatsApp: Settings → Linked Devices. If you see a device you don't recognise, remove it immediately. This is a five-second check that most people have never done.

**Be skeptical of QR codes from unexpected sources.** The attack works because people have been conditioned to scan without thinking. A QR code sent via Signal, even from a known contact, deserves the same scrutiny as a link from an unknown sender.

**Enable registration lock.** Signal's registration lock feature (Settings → Account → Registration Lock) requires your PIN when someone attempts to register your number on a new device. It does not stop the linked-device attack, but it blocks the credential phishing variant cold.

The campaign is a clean illustration of where the actual attack surface sits. The cryptography held. The mental model didn't. Not because the users were careless, but because they were operating with an incomplete picture of what the tool actually protects.

That mismatch is the vulnerability.

---

*Sources: [FBI/CISA Joint Advisory](https://www.cisa.gov) · [AIVD/MIVD Advisory](https://www.aivd.nl) · [The Hacker News](https://thehackernews.com/2026/03/fbi-warns-russian-hackers-target-signal.html) · [Help Net Security](https://www.helpnetsecurity.com/2026/03/23/russian-hackers-signal-phishing-campaign/) · [Malwarebytes](https://www.malwarebytes.com/blog/news/2026/03/signal-and-whatsapp-accounts-targeted-in-phishing-campaign) · [The Register](https://www.theregister.com/2026/03/09/dutch_spies_say_russian_cybercrims/)*
