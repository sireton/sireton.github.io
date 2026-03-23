---
title: "Security Hygiene That Actually Matters (And What Doesn't)"
date: 2026-03-23
categories: [Blog Posts]
tags: [Security Awareness, Practical Security, Defense, User Education]
description: "Most security advice for regular people is either obvious, impractical, or wrong. Here's what the research and real-world attack data says actually moves the needle."
---

Most security advice aimed at regular people is either so obvious it's useless, so paranoid it's impractical, or just wrong. The gap between what security professionals tell people to do and what the threat data says would actually protect them is significant — and worth addressing honestly.

This isn't for security professionals. It's the advice I give people when they ask me, as someone who works in this field, what they should actually do.

## The Things That Actually Matter

### A Password Manager — Non-Negotiable

Credential stuffing is the dominant attack vector against consumer accounts. The attack is simple: attackers buy or scrape credential databases from breached services and try username/password combinations against other services. It works because people reuse passwords.

The fix is using a unique, random password for every account. The only way to do that without losing your mind is a password manager. Bitwarden is free, open source, and well-audited. 1Password is excellent if you're willing to pay. Either one is fine.

The threat model for most people does not include someone cracking their master password — it includes their Netflix credentials from a 2019 breach being tried against their email. A password manager eliminates that attack entirely.

**What doesn't matter here:** Password complexity requirements that don't change reuse behavior. Changing passwords every 90 days. Security questions with real answers (make up answers and store them in your password manager).

### Multi-Factor Authentication on Email and Financial Accounts

If someone controls your email, they can reset every other account you have. Email is the master key. Protecting it with a second factor — an authenticator app, not SMS if you can avoid it — is the highest-impact single security action most people can take.

The same applies to anything with financial access: banking, brokerage accounts, payment apps.

Authenticator apps (Google Authenticator, Authy, the built-in ones in iOS/Android) are meaningfully better than SMS codes because SMS is vulnerable to SIM-swapping attacks, where an attacker convinces your carrier to transfer your number to a SIM they control. For most people this is a theoretical risk rather than a practical one, but the apps are also easier to use, so there's no reason not to.

**What doesn't matter as much as you'd think:** MFA on low-value accounts where you don't reuse the password and there's nothing sensitive to access. Prioritize your email, banking, and anything with stored payment methods.

### Software Updates — Especially Your Phone

The single most common vector for serious device compromise is unpatched vulnerabilities in software and operating systems. This is well-documented: a significant portion of real-world exploits in the wild target vulnerabilities that have had patches available for months.

Enabling automatic updates for your operating system and applications eliminates this class of attack for the vast majority of threats most people face. The counterargument — that updates sometimes break things — is real but vastly outweighed by the risk of running unpatched software.

Phone OS updates are particularly important. Mobile devices carry credentials, payment information, authentication apps, and location data. They're high-value targets and older software versions have known, publicly documented vulnerabilities.

## The Things People Worry About That They Probably Shouldn't

### Public Wi-Fi (Mostly)

The threat model for public Wi-Fi used to be passive eavesdropping — someone on the same network capturing your traffic. In 2026, essentially all web traffic is TLS-encrypted. The passive capture attack is significantly less effective than it was a decade ago.

This doesn't mean public Wi-Fi is risk-free. Man-in-the-middle attacks are still possible in some configurations, and captive portal setups can be manipulated. But the risk for someone browsing, checking email, and doing normal tasks is substantially lower than the security industry has historically communicated.

Using a VPN on public Wi-Fi adds a layer of protection and costs little — but it's not the critical-priority item it's often presented as.

### Antivirus for Most People

Modern operating systems (Windows 11, macOS, iOS, Android) ship with meaningful built-in security. For most people doing normal things, the built-in protections plus updated software and good credential hygiene provide reasonable protection against the threats they're likely to face.

Third-party AV products aren't useless, but the marginal benefit for a typical user is smaller than the marketing suggests — and some products have historically introduced their own vulnerabilities.

The bigger risk for most people isn't sophisticated malware; it's phishing and credential theft, which AV doesn't primarily address.

## The One Behavioral Thing

Most successful attacks against individuals don't involve exploiting technical vulnerabilities — they involve convincing the person to do something. A link in an email. A fake login page. A call from "Microsoft support."

The single most protective thing you can develop is the habit of pausing before you act on urgency. Phishing and social engineering attacks rely on creating a sense of immediacy — your account will be suspended, your payment failed, your package couldn't be delivered. That urgency is the mechanism, not the content.

If you get an email claiming your bank account has been locked and you need to click a link immediately, the right move is to close the email and navigate directly to your bank's website. Not because every such email is malicious, but because the habit of not clicking links in urgency-generating emails eliminates the entire attack class.

This is not glamorous advice. It doesn't involve any technology. It is, according to the data, more protective than most technical controls for the threat model most people face.

## The Priority List

If you do nothing else:

1. Get a password manager and actually use it for everything
2. Enable MFA on your email with an authenticator app
3. Enable MFA on financial accounts
4. Turn on automatic updates on your phone and computer
5. Develop a habit of not acting on urgency in messages

Everything else is secondary. These five things address the vectors that account for the vast majority of successful attacks against individuals. The rest is depth.

---

*If you're a security professional reading this and thinking "that's an oversimplification" — you're right, it is. It's also what I tell my family. The threat model matters.*
