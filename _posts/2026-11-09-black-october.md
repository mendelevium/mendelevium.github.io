---
layout: post
title: Black October Was an Invoice, Not a Storm
date: "2026-01-09"
---
What actually happened in the worst week of 2026, why it happened, and why the next one will not wait for October


Between October 17 and October 22, 2026, the security industry had the kind of week that usually takes a year to accumulate. A certificate authority's private keys turned out to be recoverable by brute force. The language that runs most of the server-side web shipped emergency patches for two remote code execution bugs that had been sitting in its core for more than a decade. A popular AI gateway library was poisoned on PyPI and Docker Hub. Microsoft described a China-aligned group running an open-source multi-agent framework against thousands of cloud tenants. A Reuters exclusive showed a lone operator breaching more than a hundred corporate websites for roughly fifty dollars a hit in model tokens. GitLab shipped a CVSS 10.0. Node.js shipped a remote code execution in its bundled Brotli decoder. Trend Micro's Zero Day Initiative accidentally posted six unpatched Linux kernel bugs to a public mailing list.

Somebody called it "Black October," and the name stuck before anyone checked where it came from. (Nobody has. One newsroom searched X for the post that supposedly coined it and found nothing.)

The name is evocative and mostly wrong. It suggests a storm: a coordinated assault, a single adversary, a month when the internet was attacked. What the record actually shows is something less cinematic and more important. Black October was an invoice. Most of the debts on it were incurred months or years earlier — a key ceremony in April 2025, a ransomware crew active since August 2024, memory-safety bugs older than some of the developers who now maintain PHP. What changed in October was not the existence of the debt. It was the speed at which someone could find it, and the speed at which someone else could collect.

## What happened

No authority has established that these incidents are connected, and nothing in the public record suggests a coordinated campaign. They are best read as separate events that landed in the same news cycle. Here are the ones that matter, with dates as stated by the primary sources.

| Date (2026) | Incident | What is established | Primary source |
|---|---|---|---|
| Oct 17–20 | **eMudhra CA keys** | Eight intermediate CA private keys (six TLS, two code-signing) generated with broken randomness in an April 30, 2025 ceremony; revoked Oct 18; root distrusted | Mozilla, Google, eMudhra |
| Oct 19 | **Cairn campaign** | One operator used an open-source agent platform to attempt 1,200+ sites and breach 100+ corporate sites for about $5,000 in tokens | Reuters, NorForge |
| Oct 20 | **SafePay advisory (AA26-293A)** | Joint FBI/CISA/USSS/NCA/German advisory on a ransomware group active since Aug 2024 with 600+ victims | Joint advisory |
| Oct 21 | **PHP2Shell** | CVE-2026-109618 and CVE-2026-109619, CVSS 9.2, unauthenticated RCE in the PHP runtime, affecting versions back to 5.4 | Wiz, PHP, NSA |
| Oct 21 | **LiteLLM 1.107.0** | Malicious build published to PyPI (≈1h45m) and Docker Hub (≈2h30m) through a hijacked release workflow | LiteLLM incident report |
| Oct 21 | **Storm-0940 / ARTEX** | Microsoft: China-state-aligned group used an AI pentest framework; several intrusions reached privileged cloud access in under three hours | Microsoft Threat Intelligence |
| Oct 21 | **GitLab, Node.js/Brotli** | CVE-2026-110264 (CVSS 10.0, unauthenticated RCE); CVE-2026-111497 (out-of-bounds write in Brotli decoder) | GitLab, Node.js |
| Oct 21–22 | **Linux kernel ZDI leak** | Six unpatched kernel bugs (two at CVSS 9.0) posted to the public LKML instead of the private security list | Phoronix, LKML |
| Oct 22 | **EDK II PXE overflow** | CVE-2026-97631, CVSS 4.0 score 9.5, pre-boot code execution on network-booting systems | TianoCore advisory |
| Oct 17–late Oct | **AI credential platforms** | OpenCode, Zeabur, Railway, Fireworks AI and others disclosed credential incidents; Zeabur announced it will shut down | Vendor disclosures, Wiz |

Three of these deserve a closer look, because together they explain most of what was genuinely new.

### The key that was all zeros

eMudhra is an Indian trust-services provider whose emSign ECC hierarchy entered browser root stores in mid-2025. On October 17, an independent researcher reported that the private keys for eight of its intermediate CAs could be recovered. According to Mozilla's Dennis Jackson, the keys were P-384 scalars smaller than 256 — effectively all zeros except the last byte. A key like that is not "weak" in the cryptographic sense. It is a key in name only.

The response from browser vendors was exemplary. Mozilla pushed a OneCRL block to standard Firefox users in under three hours from the report. Google updated CRLSet. Apple and Microsoft followed within three days. The root was distrusted. eMudhra states, and public Certificate Transparency logs corroborate, that none of the eight intermediates had issued end-entity certificates, so there is no evidence that anyone exploited the keys before disclosure.

The uncomfortable part is everything before October 17. eMudhra says the flaw went undetected for about eighteen months and through two independent audit cycles. Reporting on the incident also cites a June 2025 bug about CRLs signed with non-matching keys for the TLS intermediates — a symptom that, in hindsight, pointed straight at the problem. The company's own explanation has shifted: one update blames a key wrap/unwrap step that produced a second, weak key set with confusingly similar labels; a leadership note blames "a script error"; a third statement says the exact root cause is not yet confirmed. The keys were generated on FIPS 140-2 Level 3 HSMs. The hardware was fine. The ceremony was not.

The researcher reportedly recovered the keys with AI-assisted tooling, and Jackson reported that local agents recovered a private key on the first attempt from a simple CTF-style prompt. That detail is the whole story in miniature. The bug required no novel capability to find. It required someone to look, and looking just got very cheap. On October 22, eMudhra announced it was leaving the publicly trusted TLS business.

### The bug in the language

PHP2Shell is not a flaw in WordPress, Drupal, or a misconfigured server. As the NSA put it in its October 21 advisory, these are "flaws in PHP itself, not in the hosted application or its configuration." Wiz's write-up describes a family of memory-safety bugs with one shared root: when a request dies abnormally — typically on an out-of-memory error — PHP can leave corrupted internal state that a later request, or another worker, reuses. Depending on the path, the result is a use-after-free, an OPcache type confusion, or remote disclosure of heap memory through the `PHPSESSID` cookie, even on applications that never call `session_start()`.

Wiz says it found the first bug on August 19 using its own tooling plus two frontier models, Gemini 4 Argon and GPT-5.6 Sol, and confirmed unauthenticated RCE against Drupal the next day and WordPress the day after. It reported to PHP on August 24. PHP acknowledged on September 21 and shipped fixes on October 21. That is a normal coordinated-disclosure timeline. It is also a timeline in which the discovery took two days and the fix took two months.

What happened next is contested in the details but clear in the direction. Amazon Threat Intelligence said attackers began probing for the flaws four days before patches shipped; Cloudflare, watchTowr, Patchstack and others saw early attempts as well, all tagged as unsuccessful, and PHP's security team said the leaked proofs of concept were non-functional at patch time. By the weekend, Perplexity researcher Adel Karimi documented working remote code execution against honeypots running PHP 8.4.26. On October 26, CISA added both CVEs to its Known Exploited Vulnerabilities catalog. A follow-on flaw, CVE-2026-115560, surfaced within days. The NSA's advisory went further than these documents usually go: where patching was not immediately possible, it recommended taking affected systems offline.

About exposure, a correction to a widely repeated number: PHP runs on roughly 70% of websites *whose server-side language is known*. That is a market-share figure, not a count of vulnerable hosts and certainly not a count of compromised ones.

### The release pipeline as the attack surface

LiteLLM is an open-source gateway that sits between applications and model providers, which means its hosts tend to hold every AI provider key an organization owns. On October 21, version 1.107.0 was published with a credential stealer and a remote-code loader that runs at Python startup via `cook.pth` and `sitecustomize.py`. Uninstalling the package does not remove it.

LiteLLM's own incident report is unusually detailed, and it is worth reading as a case study in how layered controls fail one layer at a time. Branch protection rejected the first unauthorized workflow change. Five seconds later, a privileged developer's token overrode it. A second developer's token satisfied the production approval gate. The workflow was pointed at a lookalike repository, `Berri-AI/litellm` instead of `BerriAI/litellm`, and the official source tag never contained the malicious files. A third developer's token was then used to delete the workflow records. The project later concluded that its cosign signing key had also been exposed, which means the poisoned images carried a valid signature.

Where did three developers' GitHub CLI tokens come from? LiteLLM believes they were harvested from the logs of an internal deployment that was exposed to the public internet. How the attacker got into that deployment is not confirmed. The report adds a detail that every team running AI coding agents should read twice: Claude Code can embed a user's GitHub CLI OAuth token in response bodies, and those tokens can land in LiteLLM's logs when response bodies are logged. LiteLLM also notes that some of its early assumptions were later found to be "inaccurate or incomplete," so treat the causal chain as the project's current best account, not a final verdict.

## Why it happened

It is tempting to answer this with one word, and the word is "AI." That answer is half right, and the half that is wrong is the half that matters for defense.

**1. Discovery got cheap; remediation did not.** The clearest causal thread in the record is that frontier models and agent tooling found real, critical bugs in mature code. PHP2Shell was found with frontier models. The Brotli flaw was reported by OpenAI Codex Security researchers. The eMudhra keys were reportedly recovered with AI-assisted tooling. Data compiled by Epoch AI from 21 major vendors shows critical CVEs had not exceeded about 100 a month in four years, then rose past 600 a month from spring 2026 (Epoch cautions that vendor labeling and cadence vary). Former CISA director Chris Krebs, as quoted by Contrary Research, put it more starkly: "3% of all critical vulnerabilities ever discovered have been found in the last month." Discovery scaled with compute. Patching still scales with maintainers, release trains, and change windows.

**2. Most of the debt was old.** The eMudhra keys were broken from the day they were generated. PHP2Shell's bugs reach back to PHP 5.4. SafePay had been operating for over two years. Several breaches that surfaced in the same window — ETS, PingCAP, PAN-OS intrusions dating from May — began long before October. The month did not create these exposures. It revealed them.

**3. Disclosure calendars collided.** Oracle, PHP, GitLab, Node.js and TianoCore all published within about 48 hours. Some of that was coincidence of release cycles; some was consequence, as the eMudhra fix rode along in the Node.js release. Some was accident: ZDI's kernel reports went to the wrong list, and Greg Kroah-Hartman replied to each one asking why. I have found no evidence that the clustering was coordinated, and I would treat any claim that it was as speculation.

**4. Credentials became the crown jewels, and they were stored like loose change.** The connective tissue between LiteLLM, OpenCode, Zeabur, Railway and the downstream abuse is not a vulnerability class. It is secrets in places secrets should not be: tokens in logs, provider keys in environment variables, release permissions "temporarily expanded" for two engineers. Wiz's analysis, as reported by Futurum, describes stolen AI credentials being abused within 30 minutes, including one case of about 1.7 million Amazon Bedrock calls from a single address in 13 hours. (Those figures come through a secondary summary; treat them as indicative.) There is now a resale market for model access, which means a stolen provider key is liquid in a way a stolen database password is not.

**5. Attack labor got cheaper too.** The Cairn operator's instructions were, per NorForge, fewer than 100 Chinese characters: find a way to run code, plant an upload shell. The agents did the rest, at roughly 23 hours and $50 per successful intrusion. NorForge reads the worker string `claudecode_deepseek-v4.1-flash` as Claude Code driving DeepSeek's model; that is the researcher's interpretation, not a confirmed attribution. Microsoft's Storm-0940 report describes the same pattern at state scale: an open-source framework that automates the kill chain "with limited human interaction." The important nuance, which several analysts made independently, is that none of this required a novel exploit invented by a model. The agents automated reconnaissance, chaining, and persistence against known weaknesses.

**6. Amplification did the rest.** A video viewed millions of times claimed AI had infiltrated banks; another implied Claude Code could recover years of deleted iPhone photos and drew a community note. Researcher Zack Korman described an "increasing downpour" of messages from frightened people and warned that the nuance gets "lumped into a thing called 'AI scary.'" Unrelated events — French supermarket refrigeration controllers, a hijacked billboard in Giza — were swept under the same label, though no authority has linked them to the rest.

## The case that it was overblown

The strongest counterargument deserves a fair hearing, because much of it is true.

Axios framed the month as "AI's cyber scare" delivering "a Y2K lesson," and the parallel is sound. The patchable layer of the system worked. Mozilla blocked compromised intermediates in under three hours without requiring a browser restart. PHP, GitLab and Node.js shipped coordinated fixes. The NSA, FBI and CISA moved within days. The Netherlands' NCSC lowered its estimate of widespread exploitation. There is still no public evidence that the eMudhra keys were abused, and no public evidence yet of mass compromise through PHP2Shell. Microsoft itself noted that its October 10–14 password-spraying campaign produced no successful logins. Anthropic's Jason Clinton urged leaders to focus on "the basics" rather than AI; security analyst Ken Munro pointed out there was "no business need" for fridge controllers to face the internet in the first place. And a large share of the alarm came from vendors with products to sell — including several of the sources this article relies on.

All of that is correct. It is also an argument about the *outcome* of one month, and the problem Black October revealed is about the *rate*.

Y2K had a date. Teams could see the deadline, budget against it, and declare victory when it passed. There is no December 31 for AI-accelerated discovery. The 600-a-month critical CVE line does not come back down after Cybersecurity Awareness Month ends. The defenses that held in October — browser revocation, coordinated patch releases, government advisories — held because they were exercised at full speed for one week. They are built for bursts. What October announced is that the burst is the new baseline.

And the part that did not hold, the credential layer, has no Y2K analogue at all. You cannot patch a token that was already copied out of a log.

## What this means going forward

**The patch window is now an investigation window.** When a bug can be found in two days and a fix takes two months, the patch no longer marks the start of exposure; it marks the end of the period you need to investigate. Several of October's breaches predated the disclosures by months. "Patch, then hunt backward" has to become the default response to any critical runtime bug, which in turn means log retention measured in quarters, not days.

**Coordinated disclosure is under real strain.** Ninety-day windows assume the reporter is roughly the only one who knows. When multiple parties with frontier models can independently rediscover the same bug, and when a patch can be diffed and weaponized in hours, the embargo protects less than it used to. Expect pressure toward shorter windows, more emergency releases, and more mitigations shipped before root-cause fixes. Expect, too, more accidents like the ZDI posting, because the volume of reports is rising faster than the process handling them.

**AI infrastructure is credential infrastructure.** Any system that brokers model access — gateways, coding agents, hosted dev platforms — is a vault of high-value, instantly monetizable secrets. That means scoping and short lifetimes for provider keys, never logging response bodies that may contain tokens, treating release-workflow permissions as production access, and assuming a signing key is compromised whenever the release pipeline is.

**Ceremonies need adversarial review, not just audits.** The eMudhra keys passed two audit cycles. An auditor checks that a procedure was followed. An adversary checks whether the output is any good. The same tooling that recovered those keys in minutes is available to auditors; root programs should expect CAs to use it on themselves.

**Beware the forcing function.** Days before Black October, Axios reported that executives at major AI labs were running simulations of a large AI-enabled incident and its political fallout, with many participants expecting one within six to twelve months. October was not that incident. But it was close enough to be used as one. The risk now is that a disclosure wave gets treated as a catastrophe, and that the policy response targets the visible symbol — models — rather than the structural causes the record actually shows: unaudited ceremonies, unrotated secrets, overprivileged pipelines, and patch processes built for a discovery rate that no longer exists.

Catastrophes generate attention, not structural change. Black October generated plenty of attention. Whether it generates change depends on whether we read it correctly — not as the month the internet was attacked, but as the month the bill arrived for everything we had deferred.

---

## What is confirmed and what is not

| Claim | Status | Basis |
|---|---|---|
| eMudhra intermediates had recoverable keys; no end-entity certs issued | **Confirmed** | Mozilla, Google, eMudhra, CT logs |
| eMudhra's exact root cause | **Unconfirmed** | Company has given two explanations; full RCA pending via CCADB |
| PHP2Shell found with frontier models; exploited in the wild | **Confirmed** | Wiz; CISA KEV listing Oct 26 |
| Scale of PHP2Shell compromise | **Unknown** | No public count of compromised hosts |
| LiteLLM hijack via stolen developer tokens and lookalike repo | **Confirmed (vendor account)** | LiteLLM report, which notes early assumptions were revised |
| How the attacker first reached LiteLLM's deployment | **Unconfirmed** | Suspected recently patched vulnerability |
| Storm-0940 used ARTEX; some intrusions reached privileged access in <3h | **Reported by Microsoft** | No independent confirmation; attribution is Microsoft's |
| Cairn operator breached 100+ sites at ~$50 each | **Reported** | Reuters / NorForge; model identification is the researcher's inference |
| Incidents were coordinated or linked | **No evidence** | No authority has established a link |
| Origin of the name "Black October" | **Unverified** | No traceable original post found |
| LLMjacking volumes (e.g., 1.7M Bedrock calls) | **Indicative only** | Secondary summary of a Wiz analysis |
