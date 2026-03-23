---
title: "The Signal Campaign Isn't About Signal"
date: 2026-03-22
categories: [Blog Posts]
tags: [Threat Intelligence, Social Engineering, Russia, Signal, Phishing, Opinion, OPSEC]
description: "Russian intelligence isn't breaking Signal's encryption. They're breaking the humans who use it. What the current phishing campaign tells us about where the actual attack surface is."
---

This week the FBI and CISA issued a joint advisory confirming what Dutch and German intelligence had been warning about for weeks: Russian intelligence services are running a large-scale campaign against Signal and WhatsApp users, and it's been compromising thousands of accounts. Government officials, military personnel, journalists, politicians.

The news coverage has mostly focused on Signal. Whether it's safe. Whether end-to-end encryption is compromised. Whether people should stop using it.

That's the wrong question, and focusing on it misses what's actually interesting about this campaign.

## What's Actually Happening

The attackers aren't breaking Signal's encryption. They're not exploiting a vulnerability in the app. The cryptography is fine.

What they're doing is simpler and more instructive: they're impersonating Signal support accounts, sending targets messages about suspicious activity on their account or a "possible data leak," and asking the target to share their SMS verification code and PIN. If the target does, the attacker uses those credentials to register the account on a new device. Past messages are gone, but the account is now theirs.

The second technique is more elegant. Signal has a legitimate feature called "linked devices" that lets you connect additional devices to your account — so your desktop can receive messages alongside your phone. The attackers are sending malicious QR codes disguised as group invite links, security alerts, or legitimate Signal pairing instructions. When a target scans one, they unknowingly add the attacker's device to their account. The attacker can now read messages in real time. The victim keeps full access and has no indication anything changed.

The Dutch intelligence agencies were specific in their advisory: APT44 (Sandworm) has even been using devices captured on the battlefield in Ukraine — phones from compromised military personnel — to conduct further exploitation via Signal, because those devices were already trusted contacts in target accounts.

## The Lesson That Isn't New

End-to-end encryption protects the message in transit. It does not protect the account. Those are different things.

The same principle applies to every "secure" communication tool. Encrypted email is useless if an attacker has your email password. A password manager doesn't help if the attacker has your master password. Signal's encryption doesn't help if the attacker is logged into your Signal account.

This sounds obvious. In practice, it's a distinction that a lot of people — including people who should know better, which the "current and former U.S. government officials" in the advisory suggests — don't internalize. The encryption becomes a psychological anchor. People treat "uses Signal" as a proxy for "communication is secure," when the actual security model is more granular than that.

The adversaries clearly understand this. The campaigns started in Ukraine, where they were specifically targeting military personnel who had adopted Signal as a more secure alternative to unencrypted channels. The tools the defenders chose as an upgrade became the attack surface — not because the tools were bad, but because the humans using them hadn't shifted their mental model of what "secure" actually meant in practice.

## Why This Matters Beyond Espionage

The targets in the advisory are high-value intelligence targets: government officials, military personnel, journalists. Most people reading this aren't in that category.

But the technique scales. The QR code phishing attack in particular — tricking a user into scanning a code that silently adds a device to their account — works against anyone. It doesn't require nation-state resources. It requires a convincing message and a target who scans without thinking. The same playbook that Russian intelligence services refined against Ukrainian military targets will appear in criminal toolkits.

WhatsApp had a variant called "GhostPairing" documented last year — the same device-linking abuse, applied broadly. These techniques move from state actors to criminal groups on a compressed timeline now.

## What Actually Protects You

The advisory is clear and the fixes are simple:

**Never share your verification code or PIN with anyone.** Signal and WhatsApp will never ask for these via an in-app message. The only legitimate use of your SMS verification code is when you install the app fresh. Any message asking for it is a phishing attempt, regardless of how official it looks.

**Check your linked devices regularly.** In Signal: Settings → Linked Devices. In WhatsApp: Settings → Linked Devices. If you see a device you don't recognize, remove it immediately. This is a five-second check that most people have never done.

**Be skeptical of QR codes from unexpected sources.** The attack works because people have been conditioned to scan QR codes without thinking. A QR code sent via a Signal message — even from a known contact — deserves the same scrutiny as a link in an email from an unknown sender.

**Registration lock is worth enabling.** Signal has a registration lock feature (Settings → Account → Registration Lock) that requires your PIN when someone attempts to register your number on a new device. This doesn't stop the linked-device attack, but it blocks the account takeover variant.

The campaign is a useful reminder that the human layer of any security system will always be the most interesting attack surface. The cryptography held. The people didn't — not because they were careless, but because they'd been given a mental model of the tool that didn't match its actual security boundaries.

That mismatch is the vulnerability. It's also one of the more fixable ones.

---

*Sources: [The Hacker News](https://thehackernews.com/2026/03/fbi-warns-russian-hackers-target-signal.html) · [Malwarebytes](https://www.malwarebytes.com/blog/news/2026/03/signal-and-whatsapp-accounts-targeted-in-phishing-campaign) · [Help Net Security](https://www.helpnetsecurity.com/2026/03/23/russian-hackers-signal-phishing-campaign/) · [The Register](https://www.theregister.com/2026/03/09/dutch_spies_say_russian_cybercrims/)*
