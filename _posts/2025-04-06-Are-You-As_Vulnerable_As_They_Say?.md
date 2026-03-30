---
title: "Are You as Vulnerable as They Say?"
date: 2026-03-23
categories: [Blog Posts]
tags: [Security Awareness, Practical Security, Kill Chain, Defense, VPN, Antivirus, Phishing, User Education]
description: "Attackers use simple, predictable kill chains to target you. Targeted advertising tries to scare you into thinking it's complicated. Here's what actually works."
---

Someone in your life has asked you some version of this: *Do I need a VPN? Is my antivirus good enough? I got an email saying my password was leaked, what do I do?*

Maybe that person is you.

These are the right questions. The problem is where most people go looking for answers. Open your phone and you'll find a VPN ad on the podcast you're listening to, an antivirus renewal notice telling you your subscription lapsed and you're now exposed, and a push notification from an identity monitoring app warning you that your data may have appeared on the dark web. The urgency is constant. The message is consistent: you are at risk, the threats are sophisticated, and you need this product to be safe.

Here's what that messaging leaves out. According to the Verizon Data Breach Investigations Report, credential abuse and phishing account for the majority of confirmed breaches against individuals. The World Economic Forum's Global Cybersecurity Outlook 2026 found that 73% of respondents had been personally affected by cyber-enabled fraud in the past year, with phishing, vishing, and smishing representing the single largest category. Stanford researchers put the human element at the root cause of 88% of cybersecurity breaches. IBM's Cost of a Data Breach Report places phishing as the most expensive and most common initial attack vector.

The attack chain targeting everyday people is short. It is well-documented. And it is far more manageable than a market built on fear would have you believe.

This is what that chain looks like, what breaks each step, and how to cut through what's being sold to you.

---

## What's Being Marketed to You

Before getting into the attacks and defenses, it's worth understanding the landscape of products aimed at everyday consumers, what they do, what they don't do, and why some of them may create more risk than they resolve.

Iowa State University cybersecurity researcher Doug Jacobson published research in early 2025 documenting what he calls the "technology vs. user cycle": security companies market products using fear, blame, and complexity, convincing users they are incapable of managing security independently. The result is not a more secure user. It is an overwhelmed one who either buys products they don't need or disengages entirely, becoming more vulnerable in the process.

The products most aggressively marketed to consumers include:

**VPN services.** Heavily advertised through podcasts, YouTube, and social media. The pitch centers on public Wi-Fi interception and ISP surveillance. As covered below, the threat model used in these ads is largely outdated.

**Third-party antivirus suites.** Often bundled with browser extensions, identity monitoring, VPN access, password managers, and dark web scanning under a single subscription. The upsell structure is the tell. These are loss leaders designed to create a subscription relationship. Some have introduced their own vulnerabilities. Several major antivirus companies have had documented incidents where their software became the vector for compromise rather than the defense against it.

**Identity theft monitoring services.** Legitimate problem, legitimate product category, with a significant free alternative that most people don't know exists. Covered below.

**Browser security extensions.** Marketed as blocking everything dangerous with a single install. Some do useful things. Others collect browsing data as part of their business model. The extension asking for permission to read all data on all websites you visit should be scrutinized regardless of what it claims to do.

**Dark web monitoring.** A feature bundled into many paid identity and antivirus products. The free version of this capability is a website called Have I Been Pwned. More on that below.

**Credit monitoring subscriptions.** Useful after a confirmed identity theft incident. The free alternative, a credit freeze at all three bureaus, is more protective and costs nothing.

None of these products are uniformly bad. Some do exactly what they claim. The issue is that the attacks they're marketed against are not the attacks most likely to affect you, and the defenses that would actually protect you are largely free and require no subscription.

---

## The Attack Chain

In security, a kill chain describes the sequence of steps an attacker must complete to reach their objective. Each step is a break point. Stop the attacker at any link and the chain fails. You don't need to defend every possible surface. You need to break the chain before it reaches what it's after.

For everyday people, the chain is short.

---

### Step 1: Getting Your Credentials

The attacker needs a way in. The most common route doesn't involve watching your internet traffic or sitting in a coffee shop with a laptop. It involves buying a list.

Billions of username and password combinations from past data breaches are available on criminal forums, many of them free. If you've been online for more than a few years, some version of your credentials from some service you used is almost certainly in one of them. An old gaming forum. A retailer breached five years ago. A service you signed up for and forgot. Attackers run automated tools that try those combinations against current services at scale. This is called credential stuffing, and it works because people reuse passwords.

In 2025, a single leak event exposed over 16 billion credentials. That number is not a scare statistic. It is the pool attackers draw from when they run stuffing campaigns against email providers, banks, and social platforms.

The second route is phishing: a fake login page delivered via email, text, or direct message. The goal is to get you to type your credentials into a page the attacker controls. The page looks convincing. The URL is close enough. You're in a hurry.

One thing the standard "just change your password" advice misses: changing a password slightly offers almost no protection. If your password was `BlueSky2019!` and that version leaked, automated cracking tools that have the original can generate and test common variations in seconds. `BlueSky2020!` is not a new password in any meaningful sense. Protection comes from uniqueness across accounts, not from incremental changes to a single password.

**The defense:** a password manager with a unique password for every account. Covered fully in the next section.

---

### Step 2: Logging In

With valid credentials, the attacker logs in. There is no dramatic hacking involved, no exploit code, no technical sophistication required. A valid username and password produces a successful authentication. The service has no reason to flag it.

This is often the easiest step in the entire chain. The work happened in Step 1. Step 2 is just using what was collected.

**The defense:** multi-factor authentication. A second factor means a stolen password alone doesn't produce a successful login.

---

### Step 3: Taking the Keys

If the attacker gains access to your email, the rest of your digital life becomes reachable through a single mechanism: password resets.

Every "forgot my password" flow routes through your inbox. Banking. Investment accounts. Social media. Government accounts. One reset link per account, one at a time, quietly, while you sleep. Email is the master key to everything else you use online.

Attackers who gain email access often move quietly for days. Forwarding rules get created. Financial accounts get reset. Contact lists get harvested. By the time something looks wrong, the access timeline stretches back further than you'd expect and the damage spans more accounts than you realized.

**The defense:** treat your email account as the single highest-priority thing you secure. Multi-factor authentication on email is not optional.

---

### Step 4: The Objective

Once high-value accounts are accessible, the objective varies. Direct financial fraud. New credit lines opened in your name. Account access sold to other parties. Your trusted identity used to target your contacts. The common thread is that by the time you're aware of a problem, the chain has already run its course across multiple accounts.

This is where the attack surface for everyday people ends. It is not ransomware. It is not zero-days. It is not someone watching your Wi-Fi traffic at a coffee shop. It is credential theft, phishing, and account takeover, in that order, running a chain that most people have left open at Step 1.

---

## Breaking the Chain

Each step has a corresponding defense. These are ordered by the leverage they give you.

---

### Defense 1: Password Manager with Unique Passwords
*Breaks Steps 1 and 2*

This is the single highest-impact change most people can make.

A password manager generates a unique, random password for every account and stores it securely. When a service you use gets breached, only that one credential is exposed. There is nothing to reuse, nothing to stuff against other services, nothing to combine with an old version to crack a pattern. The breach is contained to where it happened.

You remember one strong master password. The manager handles everything else, filling in login forms automatically across your devices.

**Bitwarden** is free, open source, and independently audited. **1Password** is excellent if you want something more polished and are willing to pay. Both actively monitor your vault against known breach databases and will flag passwords that have appeared in prior leaks or are being reused across accounts. That monitoring is not a marketing feature. It is a meaningful signal that tells you which credentials need to be rotated before they get used against you.

One critical caveat: the master password is the one place where complexity genuinely matters. Your vault is encrypted, and reputable password managers do not store your master password. In the event of a breach on their end, what an attacker gets is encrypted data. Whether that data stays protected depends entirely on how strong your master password is. Make it long. Make it something only you would construct. Do not reuse it anywhere. This is the single point of entry for everything else you're protecting.

---

### Defense 2: Multi-Factor Authentication
*Breaks Step 2*

Passwords get leaked. Companies get breached. Credentials get phished. Defense in depth means that a stolen password alone is not enough to log in. The attacker needs a second factor, something physically on your person.

There are several types of MFA, and they are not equal:

**Text message codes (SMS)** are the most common and the weakest. A code gets sent to your phone number. The vulnerability is SIM swapping, where an attacker contacts your carrier, impersonates you, and convinces them to transfer your number to a SIM they control. After that, they receive your codes. For most people this remains a low-probability risk, but it is a documented and repeatable attack.

**Email codes** share similar weaknesses to SMS if the receiving email account is itself compromised.

**Authenticator apps** such as Google Authenticator, Authy, or the built-in options in iOS and Android settings generate time-based codes locally on your device. No carrier is involved. SIM swapping doesn't affect them. These are meaningfully stronger than SMS and should be your default wherever the option exists.

**Hardware tokens** such as a YubiKey are the strongest available form. A physical device you plug in or tap. No code to intercept, no app to compromise remotely. Appropriate for anyone handling sensitive accounts professionally or anyone who wants the highest available assurance.

Priority order for enabling MFA:

1. Email, always first. This is the account that controls access to everything else.
2. Banking and financial accounts.
3. Social media, especially accounts with a verified identity or significant following.
4. Government accounts: IRS, Social Security, state benefit portals, anything tied to your identity or benefits.
5. Everything else.

---

### Defense 3: The Pause Habit
*Breaks Step 1 via phishing*

Phishing doesn't succeed because it's technically sophisticated. It succeeds because it's fast. Every convincing phishing message manufactures urgency: your account will be suspended, unusual activity was detected, your payment failed, act now. Urgency is the mechanism. It bypasses the skepticism you'd apply to the same request if you had a moment to think.

The defense is a single habit: when a message creates urgency and asks you to click a link or provide information, close it and navigate to the service yourself. Type the URL. Log in independently. If there's a real problem it will be there. If it disappears when you navigate on your own, it was a phishing attempt.

This applies to phone calls too. No legitimate institution calls you demanding immediate payment, requesting your password, or asking for gift card numbers. Hang up and call back using a number from the official website.

The habit eliminates the entire attack class. You cannot be phished by a link you never clicked.

*For a deeper look at how phishing and social engineering attacks are being deployed right now: [The Signal Campaign Isn't About Signal](/posts/signal-campaign-isnt-about-signal/).*

---

### Defense 4: Keep Your Browser and Devices Updated
*Closes device-level exploitation*

A significant portion of real-world malware infections exploit vulnerabilities in software that has been patched but not yet updated on the target device. The fix exists. It just hasn't been applied. Enabling automatic updates for your operating system, browser, and applications closes that gap for the vast majority of threats you're likely to face.

Your browser and your phone deserve specific attention. Your browser is the primary interface between you and every threat on the web. Your phone carries your authenticator app, your banking app, and in many cases functions as the second factor for accounts across your digital life. A compromised phone can undermine every defense above it on this list.

The planned obsolescence concern is real. Older devices eventually stop receiving updates, and software updates on aging hardware can affect performance. That is a legitimate grievance. The security answer, even so, is to prioritize browser and authentication app updates at minimum, even if you defer the full OS update. If your device no longer receives security patches at all, that is the strongest available argument for replacing it. Partial coverage is better than none.

---

### Defense 5: Free Monitoring That Actually Works
*Reduces impact of Step 1 before it reaches Step 2*

You don't need a subscription to know if your credentials have been exposed.

**Have I Been Pwned** (haveibeenpwned.com) is a free service run by security researcher Troy Hunt that aggregates known breach data and tells you whether your email address has appeared in any of it. You can set up free notifications so you're alerted the moment a new breach includes your address. This is the core capability that paid dark web monitoring services are charging for.

The credit bureaus offer something more protective than monitoring: a free credit freeze. Placing a freeze at all three bureaus, Equifax, Experian, and TransUnion, prevents new credit from being opened in your name without your explicit action to temporarily lift it. This stops new-account fraud before it starts rather than alerting you after the fact. AnnualCreditReport.com, the federally mandated free service, gives you access to your full reports from all three bureaus. Use it.

Paid identity monitoring services add insurance products and broader monitoring around these free capabilities. If you've experienced identity theft before, those additions have value. For most people, the free tools cover the realistic risk.

---

## The Chain as a System

The defenses above are not independent. They stack.

A **password manager** means credential stuffing doesn't work. No two accounts share a password. A breach at one service doesn't cascade to everything else.

**MFA on email and financial accounts** means a stolen password can't complete a login. The attacker needs your physical device too.

The **pause habit** means phishing pages don't get visited. The urgency mechanism fails when you navigate independently.

**Current software** means device-level exploits hit patched targets. The gap closes before it gets used.

**Free monitoring** means you find out about exposures before attackers have had time to run the chain.

None of this requires a subscription to a third-party security suite. The password manager is the one optional cost, and free options exist there too. Together these defenses address the attack chain that the data shows is actually targeting everyday people.

The daily attack surface is not as broad as you've been told. It is narrower, more predictable, and more manageable than the products marketed against it would suggest. The chain is short. Breaking it doesn't require complexity. It requires doing a small number of specific things and understanding why each one matters.

---

## If You Do Nothing Else

1. Get a password manager and use it for every account
2. Enable MFA on your email with an authenticator app
3. Enable MFA on banking, financial, and government accounts
4. Develop the habit of not acting on urgency in messages
5. Turn on automatic updates on your phone and browser
6. Check haveibeenpwned.com and freeze your credit at all three bureaus

Everything else is depth. These six things address the vectors that account for the overwhelming majority of successful attacks against everyday people.

---

*For a recent example of how social engineering targets the human layer even when the technology is sound: [The Signal Campaign Isn't About Signal](/posts/signal-campaign-isnt-about-signal/).*
