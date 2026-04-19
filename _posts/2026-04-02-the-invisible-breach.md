---
layout: post
title: The Invisible Breach
date: "2026-04-02"
---
How Supply Chain Attacks Are Quietly Becoming the Most Dangerous Threat in Software Security

---

## Introduction: The Trust Trap

There's a thought experiment worth sitting with before reading further: how many lines of code are you actually running right now? On your development machine, across your CI/CD pipelines, in your production services?

The answer — when you account for every transitive dependency — is almost certainly in the **tens of millions**. And you've reviewed, at best, a few thousand of those lines yourself.

That gap is where supply chain attacks live.

Unlike a direct breach — where an adversary tries to crack your perimeter, phish your employees, or exploit a misconfigured server — a supply chain attack doesn't need to target you at all. Instead, it targets the shared infrastructure of trust that the entire software ecosystem depends on: open-source packages, CI/CD tooling, build pipelines, and registry systems. When an attacker compromises a dependency you use, they don't need to get past your defenses. You invited them in.

This article examines three supply chain attacks in increasing order of sophistication and ambition: **xz Utils (2024)**, **LiteLLM (March 2026)**, and **Axios (March 2026)**. After dissecting how each worked, we'll make the case — backed by technical reality — that these attacks are not isolated incidents but the opening act of a new era in which LLMs dramatically lower the barrier to discovering and exploiting supply chain vulnerabilities at scale.

---

## Part I: A Taxonomy of Trust

Software supply chain attacks are not new. The term itself entered mainstream security discourse after the **SolarWinds** breach of 2020, where a nation-state actor inserted a backdoor into the build system of a widely-used IT monitoring product, silently compromising thousands of organizations — including multiple US federal agencies — for months. But the pattern has evolved rapidly.

Modern supply chain attacks generally fall into a few categories:

**Account takeover attacks** target the credentials of trusted package maintainers, using compromised accounts to publish malicious versions directly to registries like PyPI or npm. Once published, those versions are indistinguishable from legitimate releases — because they *come from* the legitimate account.

**Dependency confusion attacks** exploit the way package managers resolve names across private and public registries, tricking build systems into downloading attacker-controlled packages that share names with internal ones.

**Upstream compromise** goes deeper, inserting malicious code into the source repository or CI/CD pipeline of a legitimate project so that the attack is baked into the official release artifact.

**Social engineering of maintainers** — the most patient form — involves cultivating trust with open-source maintainers over months or years before being granted enough access to introduce a backdoor directly into the codebase.

The xz Utils attack is the definitive example of this last category. It nearly worked.

---

## Part II: The xz Utils Attack (2024) — The Long Game

### Background

In late March 2024, Microsoft engineer Andres Freund was investigating unusual SSH latency on a Debian system and noticed that the `sshd` process was consuming abnormally high CPU. Following the thread led him to something extraordinary: a backdoor in **xz Utils**, a ubiquitous data compression library present on virtually every Linux distribution in the world.

The backdoor hadn't been injected by a hacker who had momentarily breached the project. It had been planted by **Jia Tan**, a contributor who had spent approximately **two years** building trust with the xz Utils maintainer before being granted commit access.

### The Attack in Detail

Beginning in 2022, an account registered as **JiaT75** began contributing high-quality patches to the xz Utils project. The maintainer, Lasse Collin, who had been under social pressure from what appeared to be a coordinated campaign of fake user accounts pushing him to add new contributors, eventually gave Jia Tan co-maintainer access.

In early 2024, Jia Tan modified the project's build system in a subtle way: a **malicious macro** was embedded in files distributed with the source tarball but not present in the Git repository itself. This is a crucial detail — most automated security scanners check the repository, not the distributed tarball. The macro only activated under specific conditions: Debian and RPM-based Linux distributions, x86-64 architecture, and only at install time. This was not a broad credential sweeper. It was a **targeted surgical payload**.

The backdoor was designed to intercept RSA key operations in OpenSSH (via a patched version of `systemd` that linked against the compromised liblzma) and allow the attacker to authenticate without a valid key — essentially, a skeleton key for SSH on any vulnerable system.

Had this reached stable versions of Debian, Fedora, and Ubuntu in production, the blast radius would have been staggering. Freund's chance discovery — driven by curiosity about a few hundred milliseconds of latency — was one of the closest calls in open-source security history.

### Why It Almost Worked

- **Patience**: The two-year timeline meant there was no anomalous spike of activity to flag.
- **Quality**: Jia Tan's legitimate contributions were good. The account passed every human review.
- **Stealth**: The payload was split across the tarball and the build system, making it invisible to standard repository auditing.
- **Targeting**: It specifically avoided activating in environments where security researchers are likely to run it.
- **State-level resources**: Most analysts assess this as a nation-state operation. The level of planning and the specific target (SSH authentication) point to intelligence collection, not financial crime.

---

## Part III: The LiteLLM Attack (March 24, 2026) — The AI Infrastructure Heist

If xz was about patience, the LiteLLM compromise was about precision targeting.

### Background: Why LiteLLM Matters

LiteLLM is a Python library and proxy server that provides a unified interface to over 100 LLM providers — OpenAI, Anthropic, AWS Bedrock, Google Vertex AI, and many more. It is most often used by developers as a gateway for client applications to call any number of large language models from over 100 providers. It handles **API keys for all of those providers simultaneously**. Architecturally, it occupies the most credential-rich position imaginable in an AI-native stack.

LiteLLM's PyPI package has about 480 million downloads, making it a very valuable target.

### The Attack Chain

The breach of LiteLLM was not a direct attack — it was the *third* domino in a multi-step cascade orchestrated by a threat group now tracked as **TeamPCP**.

**Stage 1 — Trivy (March 19):** TeamPCP rewrote Git tags in the `trivy-action` GitHub Action repository to point to a malicious release carrying a credential-harvesting payload. Trivy is a popular open-source vulnerability scanner. Critically, LiteLLM's CI/CD pipeline ran Trivy as part of its build process, pulling it from apt without a pinned version. The compromised action exfiltrated the PYPI_PUBLISH token from the GitHub Actions runner environment.

**Stage 2 — Checkmarx KICS (March 23):** The same infrastructure was used to compromise Checkmarx KICS (Keep Infrastructure as Code Secure), another security scanning tool, registering the impersonator domain `checkmarx.zone`.

**Stage 3 — LiteLLM (March 24):** With the stolen PyPI token, the attackers published two malicious LiteLLM versions: **1.82.7** and **1.82.8**. These compromised versions appeared to have included a credential stealer designed to encrypt and exfiltrate data via a POST request to `models.litellm.cloud`, which is not an official BerriAI/LiteLLM domain.

### The Payload

The malware was technically sophisticated in several ways.

The collected data was encrypted with a hardcoded 4096-bit RSA public key using AES-256-CBC (random session key, encrypted with the RSA key), bundled into a tar archive, and POSTed to `https://models.litellm.cloud/` — a domain that is not part of legitimate litellm infrastructure.

Version 1.82.8 introduced an especially nasty delivery mechanism: a **`.pth` file**. As ARMO's analysis explains, Python's `.pth` files are processed by the `site` module during interpreter startup — without any import statement required. The result was that *any* Python process on an affected system would trigger the malware, not just processes importing LiteLLM. This created an accidental fork bomb: the spawned child process re-triggered the `.pth` file, causing exponential process creation that crashed machines with OOM errors. This was actually how the attack was first discovered — a developer noticed their machine ran out of RAM.

If a Kubernetes service account token was present, the malware read all cluster secrets across all namespaces and attempted to create a privileged Alpine pod on every node in kube-system, mounting the host filesystem and installing a persistent backdoor.

### The Cover-Up Attempt

When community members began reporting the compromise on GitHub, the attackers posted 88 bot comments from 73 unique accounts in a 102-second window. The accounts used were previously compromised developer accounts, not purpose-created profiles. Using the compromised maintainer account, the attackers closed the issue as "not planned."

### Attribution

TeamPCP orchestrated one of the most sophisticated multi-ecosystem supply chain campaigns publicly documented to date. The tooling development arc — version 1 in December 2025, hardened version 2 in February, deployment in March — reflects a small, motivated developer team with a security research background, not a large-scale criminal operation.

Notably, a component called `hackerbot-claw` uses an AI agent (`openclaw`) for automated attack targeting. Aikido researchers documented this as one of the first cases of an AI agent used operationally in a supply chain attack. We will return to this.

---

## Part IV: The Axios Attack (March 31, 2026) — Nation-State Scale

Seven days after LiteLLM, the other shoe dropped — and it was significantly larger.

### Background: Axios at Scale

Axios is the most widely used HTTP client in the JavaScript ecosystem. With over 100 million weekly downloads, Axios occupies a position in the JavaScript ecosystem that is hard to overstate — it is present in frontend frameworks, backend services, and enterprise applications worldwide. It is a transitive dependency for an almost uncountable number of projects. If you maintain any Node.js codebase of appreciable size, you almost certainly depend on Axios somewhere in your tree.

### The Attack

On March 30, 2026, StepSecurity identified two malicious versions of the widely-used Axios HTTP client library published to npm: `axios@1.14.1` and `axios@0.30.4`. The malicious versions injected a new dependency, `plain-crypto-js@4.2.1`, which was never imported anywhere in the Axios source code. Its sole purpose was to execute a postinstall script that acted as a cross-platform remote access trojan (RAT) dropper, targeting macOS, Windows, and Linux.

The pre-staging was methodical: an earlier "clean" version of `plain-crypto-js` (4.2.0) had been published 18 hours prior, likely to give it a brief history on the registry before the malicious 4.2.1 version arrived. This is a known technique to defeat "brand-new package" detection heuristics.

The attacker compromised the `jasonsaayman` npm account, the primary maintainer of the Axios project. The account's registered email was changed to `ifstap@proton.me` — an attacker-controlled ProtonMail address.

### The Payload: WAVESHAPER.V2

This is where the Axios attack diverges sharply from LiteLLM in terms of attribution and ambition.

Google Threat Intelligence Group (GTIG) attributes this activity to UNC1069, a financially motivated North Korea-nexus threat actor active since at least 2018, based on the use of WAVESHAPER.V2, an updated version of a backdoor previously used by this threat actor.

The malware dropper cleaned up after itself: any post-infection inspection of `node_modules/plain-crypto-js/package.json` would show a completely clean manifest — no postinstall script, no `setup.js` file, and no indication that anything malicious was ever installed. Running `npm audit` or manually reviewing the installed package directory would not reveal the compromise.

Within two seconds of `npm install`, the malware was already calling home to the attacker's server before npm had even finished resolving dependencies.

The operational sophistication is notable: both the 1.x and 0.x release branches were hit simultaneously, maximizing coverage across projects that had not upgraded from the legacy branch. The malicious window ran from approximately 00:21 to 03:20 UTC — overnight hours, minimizing the time before maintainers could respond.

Within Huntress's partner base alone, at least 135 endpoints across all operating systems contacted the attacker's command-and-control infrastructure during the exposure window.

---

## Part V: The Coming Storm — LLMs as Force Multipliers for Supply Chain Attackers

The three attacks above represent a progression: from a years-long patient operation (xz) to a multi-week coordinated campaign (LiteLLM) to a precisely timed nation-state operation (Axios). But there's a fourth dimension to consider, one that was barely visible in the LiteLLM case but will become impossible to ignore: **AI-assisted attack automation**.

### The Research Problem is Solved

For decades, finding vulnerabilities in complex software required rare, expensive human expertise. A skilled security researcher could maybe audit a few thousand lines of code per day meaningfully — catching subtle logic flaws, off-by-one errors, type confusion bugs, and trust boundary violations. This scarcity of human expertise was, paradoxically, a constraint that also protected defenders: attackers faced the same scarcity.

LLMs have dramatically changed this equation. Modern code-capable models can:

- **Read and reason about codebases at superhuman speed.** A model can ingest an entire dependency tree and flag suspicious patterns in seconds.
- **Generate exploit code from vulnerability descriptions.** The gap between "here is the flaw" and "here is working proof-of-concept code" has collapsed.
- **Identify attack surface systematically.** CI/CD configuration files, package publishing workflows, token scopes — all of these can be systematically enumerated and analyzed for weaknesses.
- **Generate plausible social engineering content.** The fabrication of a believable long-term maintainer persona, complete with quality code contributions and a consistent online presence, is increasingly within the reach of automated systems.

The LiteLLM attackers had already taken a step in this direction. Their `hackerbot-claw` component, which used an AI agent for automated attack targeting, represents a proof of concept. It will not remain a proof of concept for long.

### The Economics of Scale

Supply chain attacks historically required high investment for uncertain payoff. The xz attack required approximately two years of sustained, sophisticated effort by what is almost certainly a well-funded nation-state team. That same investment, amplified by LLM-assisted tooling, could potentially be replicated by a much smaller team in a fraction of the time.

Consider what an LLM-augmented attacker workflow looks like:

1. **Discovery:** Automated scanning of GitHub Actions workflows across millions of repositories to identify unpinned actions, wide token scopes, or PYPI/npm publish tokens in CI environments.
2. **Prioritization:** Ranking targets by download count, ecosystem centrality (how many projects depend on this package transitively), and credential richness (does this package typically run in environments with cloud credentials?).
3. **Vulnerability identification:** Per-target analysis of CI/CD pipeline configurations to identify the exact injection point.
4. **Payload adaptation:** Automatic generation of platform-specific payloads with anti-forensic cleanup.
5. **Execution and suppression:** Automated bot account deployment to suppress community disclosure attempts.

Steps 1 through 4 are all tasks where current LLMs are already capable assistants. As model capabilities improve — particularly in the domains of multi-step reasoning, tool use, and code generation — the degree of human oversight required to run this pipeline decreases. A plausible near-future scenario is a largely automated supply chain attack operation that requires only a small team to set strategic objectives, review outputs, and manage infrastructure.

### The Attack Surface is Expanding

Meanwhile, the attack surface is growing faster than the security community's capacity to defend it. The explosion of AI tooling — LLM wrappers, MCP servers, agent frameworks, vector databases — has created an enormous new ecosystem of packages that are often:

- **Credential-rich by design** (they hold API keys for multiple LLM providers)
- **Maintained by small teams** with limited security resources
- **Downloaded millions of times a month** because of rapid AI adoption
- **Deeply embedded in infrastructure** through transitive dependencies

LiteLLM was the perfect illustration: a library whose core *purpose* is to centralize API keys for dozens of AI providers, running in environments with cloud credentials and Kubernetes access. Compromising it is not just compromising one package — it is compromising the credential store for every AI service that package touches.

As organizations instrument their infrastructure with AI agents and LLM orchestration, they are creating new high-value nodes in their dependency graphs. Attackers, guided by LLMs that can systematically map these new attack surfaces, will find them.

---

## Part VI: Best Practices for Defenders

The threat is real and growing. The good news is that supply chain attacks, unlike zero-days, often have effective mitigations that can be implemented before the attack arrives. The following recommendations are grounded in what actually failed in the xz, LiteLLM, and Axios incidents.

### 1. Pin Everything. Every Time.

The single highest-impact mitigation for dependency confusion and account takeover attacks is **version pinning with integrity verification**.

```bash
# Python — use pip-compile or uv with a lockfile
pip install litellm==1.82.6  # Don't use "litellm" alone

# Node.js — commit your lockfile and use npm ci, never npm install in CI
npm ci  # Respects package-lock.json exactly

# GitHub Actions — pin actions to a full commit SHA
uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683  # v4.2.2
# Not: uses: actions/checkout@v4
```

LiteLLM was compromised through its own CI pipeline using an unpinned Trivy action. The users most impacted were those who ran `pip install litellm` without a pinned version. Both of these failures are solved by pinning.

### 2. Use Lockfiles and Verify Hashes

For Python projects, use `pip-compile` with `--generate-hashes` to produce a lockfile that includes cryptographic hashes for every dependency. For Node.js, commit `package-lock.json` and use `npm ci`. For Rust, `Cargo.lock`. For Go, `go.sum`. These lockfiles ensure that even if a registry account is compromised and a new version is published, your build will fail rather than silently pulling the malicious version.

```bash
# Python: generate hash-verified lockfile
pip-compile --generate-hashes requirements.in -o requirements.txt

# Install with hash verification
pip install --require-hashes -r requirements.txt
```

### 3. Monitor Outbound Network Traffic from CI

One of the most reliable indicators of a compromised dependency is **unexpected outbound connections during build time**. In both the LiteLLM and Axios incidents, the malware called home to attacker-controlled domains within seconds of installation. A CI runner that has no legitimate reason to make outbound HTTP requests to arbitrary domains should be configured to block or alert on such traffic.

Tools like **StepSecurity's Harden-Runner** — which detected the Axios attack — operate precisely by monitoring egress traffic during GitHub Actions runs and alerting on anomalous connections. Adopting egress filtering on your CI runners is one of the most effective detection controls available.

### 4. Treat CI/CD Secrets with Zero Trust

The LiteLLM attack succeeded because a single GitHub Actions secret — the PyPI publish token — was accessible to the CI runner that also pulled an unpinned, attacker-controlled tool. Apply **least privilege** principles to CI secrets:

- Limit token scopes to the minimum required (PyPI trusted publishing avoids long-lived tokens entirely)
- Use environment protection rules so publish tokens are only accessible in protected branches
- Rotate secrets regularly and alert on unexpected access
- Consider using **OpenID Connect (OIDC)** for short-lived, just-in-time cloud credentials in CI rather than long-lived API keys

### 5. Implement Software Bill of Materials (SBOM)

An SBOM is a machine-readable inventory of every component in your software, including transitive dependencies. When a supply chain incident is disclosed, an SBOM allows you to answer the question "are we affected?" in minutes rather than hours.

```bash
# Generate an SBOM for a Python project
pip install cyclonedx-bom
cyclonedx-py requirements requirements.txt -o bom.json

# For Node.js
npx @cyclonedx/cyclonedx-npm --output-file bom.json
```

Integrate SBOM generation into your CI pipeline and subscribe to security feeds (OSV, GitHub Advisory Database, Sonatype OSS Index) that can be queried against your SBOM automatically.

### 6. Run a Dependency Security Proxy / Registry Firewall

Tools like **Sonatype Repository Firewall**, **JFrog Xray**, and **Socket** sit between your build system and the public registry and block packages that exhibit malicious characteristics in real time. Socket, for instance, performs behavioral analysis of packages — detecting things like new network calls, new postinstall scripts, or obfuscated code — rather than relying solely on known CVEs.

This is a particularly important layer because it catches *zero-day* supply chain attacks (like Axios) where no CVE yet exists.

### 7. Verify Packages Against Source Repositories

Both the LiteLLM and Axios attacks shared a common tell: **the malicious versions were not present in the official Git repository**. There was no corresponding Git tag for `axios@1.14.1` or `litellm==1.82.8`. Tooling that compares published registry artifacts against tagged source commits would have flagged both immediately.

For Python specifically, **PEP 740 (Attestations)** and PyPI's support for **Sigstore**-based publish attestations are moving toward a world where you can verify that a published package was built by an authorized CI workflow from a specific source commit. Adopting these attestations where available — and preferring packages that publish them — meaningfully reduces the risk of account-takeover-based attacks.

### 8. Harden Maintainer Account Security

The xz attack required multi-year social engineering; the LiteLLM and Axios attacks required compromising a single maintainer account. For open-source maintainers of widely-used packages:

- **Mandatory MFA** for all publishing-capable accounts (npm now requires this for popular packages; PyPI has made it available)
- **Hardware security keys** (FIDO2/WebAuthn), not TOTP, for registry account authentication
- **Separate publishing tokens** per CI environment, rotated regularly
- **Multi-party publishing controls** — require two maintainers to approve a release for high-criticality packages

### 9. Behavioral Monitoring on Developer Machines and Production

The Axios attack's malware called home within two seconds of `npm install`. In production environments, implementing **network egress monitoring** and **process behavioral analysis** (tools like Falco for Kubernetes, or EDR solutions on developer machines) can catch post-exploitation activity even when prevention fails.

Watch for:
- Unexpected outbound HTTPS connections from build processes
- New `.pth` files appearing in Python site-packages
- Unexpected `postinstall` scripts in `node_modules`
- Kubernetes pods appearing in `kube-system` that weren't in your manifests
- New systemd user services on developer machines

### 10. Apply the Principle of Ephemerality

One reason the Axios attack was somewhat contained is that many organizations use **ephemeral CI runners** — GitHub-hosted runners that are destroyed after each job. Ephemeral environments limit the persistence window for malware. Where possible:

- Use ephemeral runners for CI (GitHub-hosted or self-hosted ephemeral VMs)
- Run builds in isolated containers with minimal filesystem access
- Avoid sharing credentials between CI stages unless strictly necessary

---

## Conclusion: Security at the Speed of Trust

The three attacks examined here — xz Utils, LiteLLM, and Axios — represent a spectrum of sophistication and attribution: a patient nation-state operation, a coordinated criminal campaign targeting AI infrastructure, and a North Korean APT operation against one of the most downloaded JavaScript packages in the world. What unites them is that **none of them required breaking your code**. They only required breaking your trust.

The emergence of LLM-assisted attack tooling — already visible in LiteLLM's `hackerbot-claw` component — threatens to industrialize this attack class. Tasks that previously required scarce human expertise (finding unpinned actions at scale, reasoning about which packages are most credential-rich, adapting payloads to evade detection) are increasingly automatable. The attacker who once needed a nation-state budget to run a sophisticated supply chain campaign may soon need only a modest infrastructure investment and access to capable AI tools.

The defensive posture required in this environment is not one of reactive patching. It is one of **systemic skepticism toward the build process itself**: treating your dependency graph as an attack surface, your CI/CD pipeline as a trust boundary, and your package registry as an adversarial environment. The mitigations outlined above are not exotic — pinning versions, verifying hashes, monitoring egress, separating secrets — but they require discipline and tooling investment.

The code you ship is only as trustworthy as the code you build on. In a world where LLMs can systematically find the weakest link in that chain, the cost of ignoring the supply chain is about to become much harder to absorb.
