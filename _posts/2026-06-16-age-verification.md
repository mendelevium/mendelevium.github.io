---
layout: post
title: The Identity Layer Nobody Asked For
date: "2026-06-16"
---
Why mandatory age verification makes children less safe, strips privacy from everyone, and builds surveillance infrastructure that will outlive the panic that created it

---

In 2025, "prove you're an adult" stopped being a checkbox and started becoming a protocol. The UK's Online Safety Act crossed into hard enforcement on 25 July. Australia switched on a nationwide under-16 social-media ban on 10 December. Roughly half of US states now gate adult content behind ID checks. The EU is piloting an age-verification app, France has been pushing the issue for years, and Canada is working through its own cluster of bills. The framing is consistent everywhere: this is about protecting children.

The framing is also, on the evidence accumulated over the first year of real-world deployment, wrong about its central promise. The children these laws target are largely routing around them. The adults who were never the target are handing government IDs and biometric scans to third-party vendors with track records that range from mediocre to catastrophic. And underneath both failures, something more durable is being assembled: a general-purpose age-and-identity layer for the internet, normalized in the name of safety, that no one will dismantle once the moral urgency fades.

This is the argument I want to make as plainly as the evidence allows. Not that child safety is unimportant — it is the most important thing here — but that mandatory age verification is the wrong tool, that it fails on its own terms, that it imposes serious and asymmetric costs on the general population, and that better, less destructive alternatives already exist and are mostly being ignored.

---

## What the laws actually require

A quick map of the landscape, because the specifics matter and the rhetoric tends to flatten them.

**United Kingdom — Online Safety Act 2023.** The Act received Royal Assent in October 2023. Services publishing their own pornographic content (Part 5) had to begin implementing "highly effective age assurance" from January 2025; user-to-user services hosting such content (Part 3) had to comply by 25 July 2025, with no grace period and no phased rollout. Acceptable methods include government-ID upload, credit-card checks, and AI facial age estimation. Penalties run to £18 million or 10% of global turnover, whichever is higher, plus business-disruption measures up to and including ISP-level blocking. The Act is explicitly extraterritorial — it binds any service with UK links regardless of where it is registered, which is why Ofcom's enforcement action against 4chan is in scope at all. By February 2026, Ofcom had opened investigations into more than 90 services and issued its first half-dozen fines.

**Australia — Online Safety Amendment (Social Media Minimum Age) Act 2024.** From 10 December 2025, platforms including Facebook, Instagram, TikTok, YouTube, Snapchat, Reddit, X, Threads, Twitch, and Kick must take "reasonable steps" to keep under-16s from holding accounts, on pain of fines up to AUD 49.5 million. Notably, the law bars *parents* from consenting their children back in, and imposes no penalty on the children or families themselves — the entire compliance burden sits on platforms.

**Canada — Safe Social Media Act.** First stab was **Bill S-210**, the original "Act to restrict young persons' online access to sexually explicit material," which passed the Senate in 2023 and then stalled; **Bill S-209**, its reintroduced 2025 successor, now folding in age-estimation technologies and a narrower definition of explicit content, under Senate consideration into 2026; **Bill C-63**, the Online Harms Act, which died on the order paper at prorogation ahead of the 2025 election; and most recently **Bill C-34** (June 2026), which pairs a kids' social-media ban with mandated age verification and new AI-chatbot rules. Whichever of these the original question meant, the analysis below applies — they share the same architecture and the same failure modes.

**United States.** No federal mandate, but roughly half the states have enacted ID-based checks for adult content, which is why Pornhub now geoblocks several states outright rather than comply. California's Digital Age Assurance Act (AB 1043) takes a different route — device-level signals — with a 2027 deadline that is already shaping Apple's and Google's roadmaps.

The throughline: a patchwork of overlapping, often extraterritorial mandates, converging on the same handful of verification techniques, with early-2027 deadlines acting as a forcing function for the whole industry to lock in technical approaches now.

---

## Argument one: children will not be safer

The promise is prevention. The evidence, one year in, is circumvention.

### The instant, visible workaround

When the UK switched on enforcement, the response was immediate and measurable. Proton reported sustained daily VPN sign-up increases of 1,400–1,800% from UK users — levels the company said it normally associates with civil unrest. NordVPN reported a 1,000% spike. On enforcement day, half of the top ten free apps in the UK App Store were VPNs or identity tools. A VPN relocates your apparent location outside UK jurisdiction in about thirty seconds, and the technique is neither obscure nor expensive. By December 2025, the House of Lords was openly debating VPN circumvention — and, more damningly, child-safety group Childnet reported *increased* VPN use among children in the months after enforcement. The precise population the law was written to protect learned the workaround fastest.

### The "it's working" numbers don't show what they claim

Australia's government announced that platforms had removed roughly 4.7 million under-16 accounts within the first month and declared the policy a success. Account *removal* is not the same as access *prevention*, and the regulator's own follow-up makes the gap explicit. The eSafety Commissioner's first compliance report flagged "poor practices," including platforms letting minors retry the same age-assurance method until they passed, and "insufficient measures" to stop new under-16 accounts from being created at all. Reporting three months in found that a large majority of children with pre-ban accounts still had access to at least one platform, with no discernible drop in harm complaints. Removing five million accounts is impressive theatre. It is not evidence that five million children lost access.

### The bypass toolkit is mature

The 438-strong coalition of security and privacy researchers who signed the February 2026 open letter calling for a moratorium catalogued the bypass methods bluntly, because they aren't speculative. Borrowed or purchased credentials — an older sibling's verified account is the canonical example. Bought identities from the black markets that reliably spring up the moment age-gating creates demand. Props and AI tools, including deepfakes and AI-generated faces that defeat facial age estimation. And, repeatedly documented, *parents themselves* helping children circumvent the checks. The letter's summary judgment is that lying about your age online is, and will remain, easy.

### Deplatforming pushes kids somewhere worse

This is the part that should worry child-safety advocates most, and it is the part the laws ignore.

Deplatforming doesn't dissolve demand; it redirects it. When access to a mainstream, moderated, heavily-resourced platform is gated, determined users — including minors — migrate. They don't migrate *up* to safer ground. They migrate to fringe sites, offshore services, and unregulated corners that escape the mandate precisely because they're too small, too foreign, or too deliberately evasive to be in scope. Those are the environments with the weakest moderation, the worst CSAM controls, the most aggressive malware, and the most scams. A regime that successfully nudges a fourteen-year-old off a major platform's age-gated front door and onto an unmoderated Telegram channel or a malware-laden free-porn mirror has not protected that child. It has degraded their safety while generating a compliance metric that says the opposite.

A countermeasure that the target population defeats in thirty seconds, that the regulator's own audits show leaking, and that displaces risk toward less-safe venues is not a child-protection measure. It is a child-protection *gesture* — and gestures that feel like action are how genuinely harder problems get to be ignored.

---

## Argument two: everyone else loses privacy

Here is the asymmetry at the core of the design. The children mostly route around the wall. The adults — the entire non-target population — walk up to it and hand over identity documents. That trade would be questionable even if the verification infrastructure were secure. It is not.

### The honeypot problem is not hypothetical

Every site that verifies age has to either collect sensitive data (an ID image, a biometric scan, a credit-card record) or trust a third party to do it. Either way, the mandate manufactures honeypots: concentrated stores of exactly the data identity thieves, stalkers, and extortionists want most, distributed across thousands of operators of wildly varying competence. You don't get to choose the security posture of every adult site, dating app, and forum you're now required to verify against. You just get to absorb the breach when one of them fails. And at internet scale, one of them always fails.

Two case studies from 2025 make the point concretely.

**Discord (October 2025).** Discord disclosed that attackers had compromised a third-party customer-service vendor (5CA, based in the Netherlands) and exposed sensitive data for affected users — including roughly 70,000 government-ID images that users had submitted for age-related appeals. The attackers, a group identifying as the Scattered Lapsus$ Hunters, claimed a far larger haul (on the order of 1.5 TB) and used the access in an extortion attempt. The critical detail for policy: Discord itself wasn't breached. *Its vendor was.* Mandatory verification doesn't just expand the attack surface of the platforms you trust; it extends it through every subprocessor, support contractor, and "trusted" age-assurance partner those platforms quietly rely on — exactly the layer users can't see and can't audit.

**Tea (July 2025).** Worth scoping precisely: Tea was an identity-verification case, not strictly age verification — a women's dating-safety app that had required selfies and government IDs to register (a requirement it had already phased out). But it is the cleanest available illustration of where verification data ends up. An unsecured Firebase storage bucket exposed roughly 72,000 images, including about 13,000 selfies and photo IDs. The cache was discovered and dumped on 4chan, and within hours the leaked driver's licenses and image metadata were being assembled into a *searchable map of users' locations*. An app sold as a safety tool became a doxxing engine. The lesson transfers directly: the data you surrender to prove who you are becomes, at the moment of breach, a precision instrument for harming you.

The pattern beneath both: the verification step doesn't merely risk *a* breach, it changes the *character* of breaches. A leaked password is a nuisance you can rotate. A leaked passport, bound to your face and your home location, is permanent and unrotatable. Age-mandate breaches convert recoverable incidents into irreversible ones.

### "We don't retain the data" is a promise, not a control

Vendors and platforms routinely assure users that IDs aren't stored or linked to accounts — Discord and others say exactly this. Those commitments may be sincere. But users have no independent visibility into whether they hold, no way to verify deletion, and no recourse when a subprocessor two hops down the chain logs what it promised to discard. "Trust us, it's ephemeral" is not a security architecture. It's a marketing claim with the durability of the next breach disclosure.

### The experts are not equivocating

The February 2026 moratorium letter — 438 researchers across 32 countries — did not hedge. Its core finding is that no existing age-verification system can reliably confirm a user's age *without* simultaneously collecting sensitive data that becomes a liability, and that there is no scientific evidence these controls improve child safety. NIST's repeated documentation of demographic bias in facial-recognition and age-estimation systems — worse error rates for people of color and for transgender individuals — sits underneath the privacy objection as a second, independent failure: the "privacy-preserving" biometric option is also the discriminatory one.

---

## The unintended side effects, catalogued

Even granting good intentions, a policy should be judged by its full consequence set. Here is the ledger that the "protect the children" framing tends to leave off the page.

| Side effect | Mechanism | Who absorbs it |
|---|---|---|
| **A permanent identity layer** | Verification infrastructure, once built and mandated, doesn't get removed when the panic recedes. The EFF calls age "the first identity attribute to be widely deployed within the operational architecture of the internet." | Everyone, indefinitely |
| **Scope creep** | Laws written for pornography expand to social media, search, AI tools, and user-to-user services. Wikipedia was pulled toward "Category 1" obligations under the UK Act and lost its High Court challenge in August 2025. | Lawful platforms and their users |
| **Death of the small web** | Vague, broad "user-to-user" definitions impose compliance costs that hobbyist forums and niche communities can't bear. UK operators have shut down community sites rather than risk liability. | Independent and community sites |
| **Migration to fringe platforms** | Deplatforming redirects users to unregulated services with weaker safety controls and more malware. | Minors and adults alike |
| **Exclusion and inequality** | Verification presumes a government ID and a supported smartphone. People without either — disproportionately the marginalized — get locked out of lawful services. | The already-excluded |
| **Biometric bias** | Facial age estimation misfires along demographic lines (NIST). | People of color, trans people, the misjudged |
| **Market concentration** | OS-level age signals route control to Apple and Google, entrenching their gatekeeping over distribution *and* identity. | Developers, competitors, the open web |
| **Normalized circumvention** | Mass VPN adoption and credential black markets become routine, eroding the norm that you don't route around the law. | The broader security culture |
| **Chilling effects** | Requiring ID to read lawful content deters lawful access to it — health information, LGBTQ resources, political speech. | Anyone seeking sensitive but legal material |

That last cluster deserves emphasis. "You must show ID to enter" changes behaviour around lawful content the same way it does in physical space — people self-censor, avoid, and stay away from things they have every right to access, because the act of proving identity to reach them feels like surveillance. It is surveillance. The chilling effect isn't a bug to be tuned out; it's an inherent property of putting an identity checkpoint in front of speech.

---

## What actually works better

The strongest objection to critics of age verification is fair: *fine, but children really are encountering harmful content, so what's your alternative?* The honest answer is that better tools exist, several are already deployed, and most have a fraction of the privacy cost. None is perfect. All are less destructive than per-site ID collection.

### 1. Device- and OS-level age signals

Instead of every website running its own ID check, the age determination happens once, at the device or operating-system layer, and is passed downstream as a coarse, bracketed signal. Apple's Declared Age Range API and Google's Play Age Signals API return ranges (e.g., "under 13," "13–15," "16–17," "18+") rather than birthdates or documents. California's AB 1043 codifies this approach. A parent sets the child's age band at device setup; apps query the signal in real time and make access decisions without ever seeing — or storing — an identity document.

**Why it's better:** It collapses thousands of honeypots into one decision point, shares the minimum necessary information (a band, not a passport), and prohibits long-term retention of age data. The friction is paid once.

**Honest tradeoffs:** It is still fundamentally a *declared* or *parental-set* signal — it attests to a setting, not to who is physically holding the phone right now, so it's defeatable by a determined teen with access to an adult's device. And it deepens Apple's and Google's control over the stack, a real competition and centralization concern. It's a mitigation, not a magic fix. But "one privacy-preserving signal, set once, retained never" beats "an ID upload to every adult site you visit" on every axis that matters.

### 2. DNS-level filtering — the most underrated tool in the box

This is the alternative that gets the least airtime and arguably delivers the most protection per unit of privacy cost. The Domain Name System resolves the names you type into the addresses your device connects to. Point your home network at a filtering resolver and inappropriate domains simply never resolve — network-wide, for every device, with nothing installed and no data handed to anyone.

| Service | Resolver addresses | Profile |
|---|---|---|
| **CleanBrowsing (Family)** | `185.228.168.168` / `185.228.169.168` | Blocks adult/explicit sites, forces SafeSearch on major engines, also blocks proxy/VPN bypass domains and mixed-content sites like Reddit |
| **OpenDNS FamilyShield** | `208.67.222.123` / `208.67.220.123` | Free, no account, pre-configured to block adult and proxy domains; ~90% block rate in independent testing, but does *not* force SafeSearch |
| **Cloudflare for Families** | `1.1.1.3` (malware + adult) / `1.1.1.2` (malware only) | Fast, global, zero-config; coarse category filtering |

Set it on the router and it covers the whole house in about five minutes. It's free, it leaks nothing about your identity to any site, and it's trivially reversible by the adult who controls the network. Its limits are honest ones — a tech-literate teen can change a device's DNS, use DoH to bypass it, or tether to mobile data — which is exactly why it belongs in a layered approach rather than being sold as a silver bullet. But as a default first line of defence in the home, it does most of what parents actually want, with none of the surveillance.

### 3. Real parental controls and on-device tooling

The mature parental-control ecosystem — built into iOS Screen Time and Android Family Link, extended by tools like Qustodio for time limits, app blocking, and activity reporting — keeps decisions where developmental context lives: with the parent who knows whether a given child is ready for a given thing. A blanket legal age line treats a 10-year-old and a 15-year-old identically, ignoring exactly the judgment that good parenting supplies. On-device controls are granular, they're already shipped on the devices children use, and they don't require anyone to build a national ID database.

### 4. Attack the root cause: data minimization and a real privacy baseline

The deepest fix is the least discussed. A large share of the harm that age verification is meant to address — manipulative engagement-maximizing design, predatory data collection from minors, addictive feeds — flows from a surveillance-advertising business model that current law permits. Comprehensive data-minimization rules that restrict what platforms may collect and exploit, especially from children, reduce those harms *without* requiring anyone to prove who they are. Strong privacy legislation should be the prerequisite, not an afterthought: without it, an "age signal" is just one more data point to monetize. Regulating platform *design* — duties to act responsibly, transparency obligations, researcher data access, real penalties for engagement dark patterns — targets how these services actually cause harm, instead of erecting an identity checkpoint and calling it safety.

---

## Steelmanning the other side

Intellectual honesty requires engaging the strongest version of the case for mandatory verification, not the weakest.

**"Some friction is better than none."** True, and the strongest point the proponents have. Even a leaky gate deters the casual, accidental encounter — the curious twelve-year-old who stumbles onto something, rather than the determined fifteen-year-old hunting for it. Reducing accidental exposure has real value. The rebuttal is one of proportionality: you can capture most of that deterrence with DNS filtering and device-level controls — which raise the *same* casual-encounter friction — without building a national identity-collection apparatus and without the breach, exclusion, and chilling costs. If the goal is friction against accidents, the cheapest, least invasive tool that produces friction wins. Per-site ID collection is the most invasive tool that produces it.

**"The offline world already does this."** We card people at liquor stores and casinos, the argument goes, so why not online? Because the offline check is ephemeral and local — a bouncer glances at a license and forgets it. The online equivalent uploads a permanent, copyable, breachable digital image of that license to a remote server and a chain of subprocessors. The analogy holds for the *intent* and breaks completely on the *mechanics* — and the mechanics are where all the harm lives.

**"Platforms have failed to self-police, so the state must act."** Also largely true — voluntary moderation has been inadequate. But the conclusion doesn't follow. The choice isn't "ID mandates or nothing." Design regulation, data-minimization law, mandatory parental tooling, and default-on network filtering are all forms of the state acting, none of which require turning the internet into a checkpoint.

The proponents are right that the harm is real and that doing nothing is not acceptable. They are wrong that this specific instrument addresses it. Caring about the problem and opposing this solution are fully compatible positions — indeed, taking child safety seriously is a reason to reject a measure that fails children while costing everyone else their privacy.

---

## The part that outlives the panic

Strip away the specifics and here's what the first year of deployment demonstrates. The targets evade the controls — visibly, immediately, and in growing numbers. The non-targets pay with their most sensitive data, into systems that are already breaching at scale. The infrastructure being built to enforce all this — device-bound identity signals, biometric estimation, ID-collection pipelines, the apparatus for jurisdictional website blocking — is general-purpose, and it doesn't get uninstalled when the headlines move on. As the EFF observed, age is simply the *first* identity attribute being wired into the operating architecture of the internet. Once that architecture exists and everyone is conditioned to authenticate identity to access content, the marginal cost of extending it — to other attributes, other content, other reasons — drops toward zero. There is no putting that toothpaste back in the tube.

Children deserve a safer internet. They will not get one from a system they defeat in thirty seconds, that leaks their parents' passports onto 4chan, and that quietly builds the most consequential surveillance infrastructure of the decade while everyone is looking at the children. The better tools are unglamorous: a DNS setting changed on a router, a device age band set at setup, a parental-control app, a privacy law with teeth. They are also the ones that actually trade off correctly — more protection, less surveillance — which is presumably why they keep losing to the version that photographs well in a press release.
