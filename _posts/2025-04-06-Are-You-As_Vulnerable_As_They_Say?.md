---
title: "Are You as Vulnerable as They Say?"
date: 2025-04-06
categories: [Blog Posts]
tags: [Security Awareness, Practical Security, Kill Chain, Defense, VPN, Antivirus, Phishing, User Education]
description: "Attackers use simple, predictable kill chains to target you. Targeted advertising tries to scare you into thinking it's complicated. Here's what actually works."
---

Someone close to you has probably asked you some version of this: *Do I need a VPN? Is my antivirus good enough? I got a text saying my account was compromised, what do I do?*

Maybe that person is you.

The honest answer is that you are probably safer than the products being sold to you want you to believe, and more at risk than you realize in the specific ways that actually matter. The gap between those two things is where most people get lost.

Here is what the data says. According to the Verizon 2025 Data Breach Investigations Report, credential abuse and phishing account for the vast majority of confirmed breaches. Stanford researchers found the human element at the root of 88% of cybersecurity incidents. The World Economic Forum's Global Cybersecurity Outlook 2026 found that 73% of people had been personally affected by cyber-enabled fraud in the past year, with phishing, vishing, and smishing representing the single largest category by far.

The attack surface targeting everyday people is real. It is also narrower, more predictable, and more manageable than a market built around fear would have you believe.

Every attack against an everyday person starts with some version of the same moves: getting your credentials, getting past your login, and working outward from there. Sometimes that starts with a link you clicked. Sometimes it starts with a file you downloaded. Sometimes it starts with a phone call, a text, or a data breach at a company you forgot you had an account with. The vector changes. The chain is the same.

This article walks through that chain, what each step looks like, what breaks it, and which products being marketed to you are solving real problems versus selling manufactured urgency.

---

## The Products Being Marketed to You

Before walking through the attacks, it is worth spending a moment on the landscape of products aimed at everyday consumers. Not because they are all bad, but because understanding what they actually do, and what they don't, makes the rest of this easier to evaluate.

Iowa State University cybersecurity researcher Doug Jacobson published research in early 2025 documenting what he calls the "technology vs. user cycle." Security product marketing relies on fear, blame, and complexity to convince users they cannot manage security independently. The result is not a more secure person. It is an overwhelmed one who either buys products that don't address their actual risk or disengages entirely, becoming more vulnerable in the process. [[Iowa State Research]](https://research.iastate.edu/2025/01/07/selling-fear-marketing-for-cybersecurity-products-often-leaves-consumers-less-secure/)

The products most aggressively marketed to everyday consumers include:

**VPN services** are among the most heavily advertised consumer security products, pushed through podcasts, YouTube channels, and social media sponsorships. The pitch is that someone on the same public Wi-Fi network can intercept your traffic and steal your passwords. The threat model is largely outdated. The vast majority of web traffic today travels over HTTPS, meaning it is already encrypted in transit between your device and the destination server. A passive attacker on the same coffee shop network sees encrypted data, not your credentials. A VPN is useful in specific scenarios: sensitive work on untrusted networks, travel in countries with restrictive internet policies, or masking activity from your ISP. It is not a substitute for any of the defenses covered in this article, and it does nothing against the attacks most likely to affect you.

**Third-party antivirus suites** are typically sold as complete security packages bundling browser extensions, identity monitoring, VPN access, dark web scanning, and password managers under a single subscription. The upsell structure is the tell. Modern operating systems ship with capable built-in protection. Windows Defender is a legitimate, continuously updated security product. macOS ships with XProtect and Gatekeeper. iOS and Android have security models that make traditional malware largely irrelevant on mobile. The concern with many third-party products is not that they fail to detect malware. It is what some of them do alongside that function: browser extensions that redirect searches, data collection practices buried in license agreements, and background processes that have in documented cases introduced their own vulnerabilities. Several major antivirus vendors have had incidents where their software became the vector for compromise rather than a defense against it.

**Identity theft monitoring subscriptions** address a real problem. The free version of that capability is a website called Have I Been Pwned, covered later in this article. Paid services add credit monitoring and insurance products around that core. For most people, the free tools cover the realistic risk.

**Browser security extensions** marketed as blocking everything dangerous with a single install deserve scrutiny. Some do useful things. Others collect your browsing history as part of their business model. An extension requesting permission to read all data on every website you visit should be evaluated carefully regardless of what it claims to do.

**Credit monitoring subscriptions** are useful after a confirmed identity theft event. The free alternative, a credit freeze at all three bureaus, is more protective and costs nothing because it stops new accounts from being opened in your name before the fact rather than alerting you after.

None of these products are universally bad. The issue is that the attacks they are marketed against are often not the attacks most likely to affect you, and the defenses that would actually protect you are largely free.

---

## The End User Defense Chain

In security, a kill chain describes the sequence of steps an attacker must complete to reach their objective. Each step is a break point. Stop the attacker at any link and the chain fails. You do not need to defend every possible surface. You need to break the chain before it reaches what it is after.

For everyday people, the chain is short and well-documented. What follows is each step, what it looks like in practice, and what breaks it.

---

### Credentials: The Starting Point for Most Attacks

The most common way someone gets into your accounts has nothing to do with your internet traffic being watched. It involves buying a list.

Billions of username and password combinations from past data breaches are available on criminal forums, many of them free or cheap. The 2025 DBIR found credential abuse was the initial access vector in 22% of confirmed breaches, and a single leak event in 2025 exposed over 16 billion credentials. [[Verizon DBIR 2025]](https://www.verizon.com/business/resources/reports/dbir/) If you have been online for more than a few years, some version of your login from some service you used is almost certainly in one of those databases. An old forum. A retailer that got breached years ago. A service you signed up for and forgot about. Attackers run automated tools that try those combinations against active services at scale. This is credential stuffing, and it works because most people reuse the same passwords across multiple accounts.

The second route is phishing: a fake login page delivered via email, text, or direct message, designed to get you to type your credentials somewhere the attacker controls. The 2025 DBIR found phishing was the initial vector in 16% of breaches, and the WEF found it was the top form of cyber-enabled fraud affecting individuals in 2025, with 62% of respondents reporting personal exposure. [[WEF Global Cybersecurity Outlook 2026]](https://reports.weforum.org/docs/WEF_Global_Cybersecurity_Outlook_2026.pdf) The page looks convincing. The URL is close enough that you don't notice. You are in a hurry and the message told you to act now.

There are other vectors worth knowing about even if they are less common. Downloading a file from an untrusted source can install software that captures your keystrokes or harvests saved passwords from your browser. A USB drive found in a parking lot is an old trick that still works. Physical theft of an unlocked device hands an attacker everything on it. These are real attack vectors. They are also ones where the same habits that protect you from phishing and credential stuffing tend to reduce the risk considerably.

One thing the standard "change your password regularly" advice misses entirely: changing a password slightly offers almost no protection. If `BlueSky2019!` leaked in a breach, automated tools that have the original can generate and test common variations in seconds. `BlueSky2020!` is not a new password in any meaningful sense. Protection comes from uniqueness across accounts, not from rotation of the same password.

**What breaks this:** A password manager with a unique, randomly generated password for every account. When one service gets breached, only that credential is exposed. There is nothing to reuse, nothing to stuff against other services, nothing to build a pattern from.

**Bitwarden** [[bitwarden.com]](https://bitwarden.com) is free, open source, and independently audited. **1Password** [[1password.com]](https://1password.com) is excellent if you want something more polished and are willing to pay. Both actively monitor your stored passwords against known breach databases and flag credentials that have appeared in prior leaks or are being reused across accounts. That monitoring is a meaningful early warning before those credentials get used against you.

The master password deserves specific attention. Your vault is encrypted and reputable password managers do not store your master password on their servers. If the company were ever breached, what an attacker would have is encrypted data. Whether that data stays protected depends entirely on the strength of your master password. Make it long. Make it something only you would construct. Do not reuse it anywhere. This is the single key to everything else you are protecting.

---

### Getting Past Your Login

With valid credentials, an attacker logs in. There is no dramatic intrusion involved, no technical exploit, no special skill required. A valid username and password produces a successful authentication. The service has no reason to flag it. The 2025 DBIR found that 88% of attacks against web applications involved stolen credentials used exactly this way. [[Verizon DBIR 2025]](https://www.verizon.com/business/resources/reports/dbir/)

This is often the easiest step in the entire chain. The credential acquisition above is the hard part. This step is just using what was already collected.

**What breaks this:** Multi-factor authentication. A second factor means a stolen password alone cannot complete a login. The attacker needs something physically in your possession.

There are several types of MFA and they are not equally strong:

**Text message codes** are the most common and the most vulnerable. The weakness is not just SIM swapping, where an attacker convinces your carrier to transfer your number to a device they control. There are broader phone number attacks worth understanding. Caller ID spoofing allows an attacker to make a call or send a message that appears to come from any number they choose, using cheap VoIP services that require no technical expertise. SS7 exploitation targets weaknesses in the decades-old signaling infrastructure that routes calls between carriers globally, allowing sophisticated attackers to intercept messages at the network level. Third-party spoofing services sell this capability directly for a small fee, with no technical knowledge required. [[FCC on Caller ID Spoofing]](https://www.fcc.gov/spoofing) The practical takeaway is that your phone number is not proof of identity, and neither is a number displayed on your screen calling you. Text-based MFA is still meaningfully better than no MFA. It is just the weakest option available.

**Email codes** share similar weaknesses if the receiving email account is already compromised.

**Authenticator apps** such as Google Authenticator, Authy, or the built-in options in iOS and Android settings generate time-based codes locally on your device. No carrier or network is involved. Phone number attacks do not affect them. These should be your default wherever the option exists.

**Hardware tokens** such as a YubiKey are the strongest form available. A physical device you plug in or tap, with no code to intercept and no app to compromise remotely. Appropriate for anyone handling sensitive accounts professionally or who wants the highest available assurance.

Priority order for enabling MFA:

1. Email, always first. This is the account that resets everything else.
2. Banking and financial accounts.
3. Social media, especially accounts tied to your identity or with a significant audience.
4. Government accounts: IRS, Social Security, state benefit portals, anything tied to your identity or benefits.
5. Everything else.

---

### The Account That Unlocks Everything Else

If an attacker reaches your email, the rest of your digital life becomes accessible through a mechanism most people don't think about until it happens: password resets.

Every "forgot my password" flow routes through your inbox. Banking. Investment accounts. Social media. Government accounts. Healthcare portals. One reset link per service, quietly, while you sleep. Email is the master key to every account you have ever linked to it.

Attackers who gain email access often stay quiet for days. Forwarding rules get created to copy incoming mail somewhere else. Financial accounts get reset one at a time. Contact lists get harvested. By the time something looks visibly wrong, the access may have been running for days and the damage spans more accounts than you would expect.

MFA on your email is the defense that protects the master key. It belongs on your email before it goes anywhere else.

---

### Social Engineering: When the Attack Is the Message

Not every attack starts with a breach database. Some start with a message designed to make you act before you think.

Phishing, vishing (voice calls), and smishing (SMS) all work the same way regardless of the channel: they manufacture urgency. Your account will be suspended. Unusual activity was detected. A package could not be delivered. A charge was made that needs your approval. The urgency is the mechanism. It overrides the skepticism you would apply to the same request if you had a moment to consider it.

The 2025 WEF report found that phishing, vishing, and smishing were the top form of cyber-enabled fraud experienced by individuals, affecting 62% of respondents personally. [[WEF Global Cybersecurity Outlook 2026]](https://reports.weforum.org/docs/WEF_Global_Cybersecurity_Outlook_2026.pdf) That number reflects how effective the urgency mechanism remains even among people who know it exists.

Phone calls deserve specific mention here. As covered in the MFA section, caller ID is not a reliable indicator of who is actually calling. A call displaying your bank's number is not necessarily your bank. No legitimate institution calls demanding immediate payment, requesting your account password, asking you to install software, or requesting gift card numbers. If a call creates urgency around any of those things, hang up and call back using the number on the institution's official website.

**What breaks this:** A single habit. When a message or call creates urgency and asks you to click a link, provide information, or take immediate action, stop. Close it. Navigate to the service directly by typing the address yourself. Log in independently. If there is a real problem it will be there. If it disappears when you navigate on your own, it was an attempt. You cannot be phished by a link you never clicked.

---

### Your Devices: The Foundation Everything Else Sits On

A significant portion of real-world infections exploit software that has been patched but not yet updated on the target device. The fix exists. It just has not been applied. Enabling automatic updates for your operating system, browser, and applications closes that gap for the vast majority of threats you are likely to face.

Your browser and your phone deserve the most attention. Your browser is the primary surface between you and every threat on the web. Your phone carries your authenticator app, your banking app, and in many cases functions as the second factor for accounts across your digital life. A device running outdated software undermines every other defense on this list.

The concern about updates affecting performance on older devices is legitimate and worth acknowledging directly. Software updates on aging hardware can slow things down, and manufacturers do eventually stop releasing updates for older models. If a full system update is not practical, prioritize your browser and authentication apps at minimum. If your device has reached the point where it no longer receives security patches at all, known vulnerabilities are sitting open on something carrying your most sensitive accounts. That is the clearest argument for replacing it when budget allows.

---

### Monitoring: Finding Out Before the Damage Spreads

You do not need a paid subscription to know if your credentials have been exposed.

**Have I Been Pwned** [[haveibeenpwned.com]](https://haveibeenpwned.com) is a free service maintained by security researcher Troy Hunt that aggregates data from known breach events and tells you whether your email address appeared in any of them. Free notifications alert you when a new breach includes your address. This is the core capability that paid dark web monitoring services charge a monthly fee for.

The credit bureaus offer something more protective than monitoring: a free credit freeze. Placing a freeze at Equifax, Experian, and TransUnion prevents new credit from being opened in your name without your explicit action to temporarily lift it. This stops new-account fraud before it starts rather than notifying you after. **AnnualCreditReport.com** [[annualcreditreport.com]](https://www.annualcreditreport.com) is the federally mandated free service giving you access to your full reports from all three bureaus.

Paid identity monitoring services add insurance products and broader monitoring around these free capabilities. If you have experienced identity theft before, or have specific reason to believe your information is actively being misused, those additions have value. For most people, the free stack covers the realistic risk without a subscription.

---

## If You Do Nothing Else

The chain is short. Breaking it does not require a suite of subscriptions or a background in security. It requires doing a small number of specific things and understanding why each one matters.

1. **Get a password manager and use it for every account.** Bitwarden [[bitwarden.com]](https://bitwarden.com) is free. 1Password [[1password.com]](https://1password.com) is excellent if you want to pay. The goal is a unique, randomly generated password for every account you have.
2. **Enable MFA on your email first, with an authenticator app.** This protects the account that resets everything else.
3. **Enable MFA on banking, financial, and government accounts.**
4. **Build the habit of not acting on urgency in messages.** Close it. Navigate directly. Log in independently.
5. **Keep your browser and phone updated.** If your device no longer receives security patches, that is a problem worth solving.
6. **Check Have I Been Pwned [[haveibeenpwned.com]](https://haveibeenpwned.com) and freeze your credit at all three bureaus.** Both are free. Both are more protective than paid alternatives for most people.

The daily attack surface is not as broad as the advertising suggests. It is narrower, more predictable, and more manageable. The chain targeting everyday people is short. Most of the tools that would actually break it cost nothing.
