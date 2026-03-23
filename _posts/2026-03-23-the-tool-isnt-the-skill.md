---
title: "The Tool Isn't the Skill"
date: 2026-03-23
categories: [Blog Posts]
tags: [Opinion, Methodology, Mindset, Penetration Testing]
description: "Why the security industry's obsession with tooling is producing a generation of operators who can run scripts but can't think through a problem."
pin: false
---

There's a pattern I've noticed that bothers me more the longer I spend in this field: people who can run a tool perfectly and have no idea what it's actually doing.

Ask someone how Kerberoasting works and you'll often get: *"You use Rubeus or GetUserSPNs to request service tickets, then crack them offline."* That's correct. It's also a description of running a tool. Ask them *why* the KDC hands out service tickets encrypted with the service account's hash to any authenticated user — ask them what assumption the Kerberos designers made in 1993 that makes this attack possible — and the room goes quiet.

This isn't a criticism of people learning. It's a criticism of how we've structured learning.

## The Lab-to-Script Pipeline

The dominant path into offensive security right now runs through platforms like HackTheBox, TryHackMe, and certification prep content. These are genuinely good resources. But the incentive structure they create, largely unintentionally, rewards getting the flag over understanding the path. You learn which tool produces the right output for a given scenario. You learn the shape of the problem before you learn why the problem exists.

I went through this myself. Early on, I could enumerate an AD environment with BloodHound, identify a Kerberoastable user, crack the hash, and move laterally — all following a workflow I'd internalized. What took longer to develop was the ability to look at a novel configuration I'd never seen before and reason about *why* it might be exploitable, without a template to follow.

The difference between those two states is the difference between a technician and an analyst. Both matter. Only one scales.

## Why It Matters More Than It Used To

This gap was always present, but it's becoming more consequential. Defensive tooling is getting better. EDR solutions are increasingly capable of detecting the exact patterns that scripted attacks produce. The attacks that work reliably now are often the ones that understand the underlying protocol well enough to blend in — to look like legitimate behavior because they're operating within the same trust assumptions the protocol was built on.

If you understand *why* Pass-the-Hash works — that NTLM authentication doesn't require the plaintext password, only the hash, because of how the challenge-response handshake is structured — you can reason about when to use it, when not to, and what a defender watching that traffic would see. If you only know that `evil-winrm -H <hash>` works, you're running blind.

The defensive side of this is equally important. A network security team that only knows how to read SIEM alerts, without understanding the underlying technique those alerts are trying to catch, will always be one signature refresh behind.

## What Actually Builds the Skill

The most useful thing I've done for my own development isn't grinding more boxes. It's reading. Not blog posts — primary sources. RFC documents. Microsoft's own documentation on how Kerberos is implemented in Windows environments. The original research papers behind vulnerabilities like PrintNightmare and ZeroLogon. Understanding not just *that* a thing is vulnerable but *where the assumption broke down*.

The second most useful thing has been trying to explain what I'm doing while I'm doing it — writing it up, building notes that someone else could follow. If you can't explain why a technique works without referencing the tool name, you probably don't fully understand it yet.

The third is intentional constraint. Do a box without using automated enumeration tools. Force yourself to do manually what linpeas does automatically. It's slower, less efficient, and extremely educational.

## The Tool is a Force Multiplier

None of this is an argument against tooling. Tools are how you operate at scale. Knowing that `impacket-secretsdump` can dump an entire domain's worth of credentials in seconds is practically useful. The point is that the tool is a force multiplier on understanding — and if the underlying understanding is shallow, the multiplier doesn't help as much as you think.

The operators I've seen do consistently good work are the ones who treat every tool as a question: *What is this doing, and why does it work?* That curiosity, applied consistently, compounds. The gap between someone who runs tools and someone who understands what they're running widens significantly over two or three years.

The field is moving fast enough that the people who understand the fundamentals will keep adapting. The people who've memorized workflows will keep needing new ones.

---

*If you've been through this yourself — going from tooling to understanding — I'd be curious what actually moved the needle for you.*
