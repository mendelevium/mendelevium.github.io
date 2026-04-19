---
layout: post
title: The Invisible Leak
date: "2026-04-19"
---
DNS, ISP Transparency, and the Quiet Erosion of Your Privacy

> *Every website you visit. Every app you open. Every domain you touch. Before a single byte of content reaches you, a request goes out into the open — and someone is almost certainly watching.*

---

## Preface: The Protocol That Forgot Privacy

When the Domain Name System was designed in 1983 by Paul Mockapetris, the internet was a collegial academic network of a few hundred machines. Privacy wasn't an afterthought — it wasn't a thought at all. DNS was built to be fast, distributed, and resilient. It was built for a world that no longer exists.

Forty years later, DNS remains the foundational directory of the internet, handling over **620 billion queries per day** globally. And for most of those queries, the privacy model is essentially unchanged from 1983: requests travel in plaintext, unencrypted, unauthenticated, and fully visible to anyone positioned between you and the resolver.

This is the story of DNS leakage — what it is, why it has gotten significantly worse as ISPs  have adopted "transparency" programs, what it means for your security and privacy, and what you can actually do about it.

---

## Part I: Understanding DNS — The Phone Book You Can't Opt Out Of

### How DNS Actually Works

When you type `example.com` into your browser, your computer doesn't know where to find it. IP addresses are what machines route; domain names are what humans remember. DNS is the translation layer.

The resolution process unfolds in a chain:

1. **Stub Resolver** — Your OS checks its local cache. If it has a recent answer, done. If not, it forwards the query to a *recursive resolver* — typically one configured automatically by your ISP via DHCP.

2. **Recursive Resolver** — This server (usually operated by your ISP, or a public provider like `8.8.8.8`) does the heavy lifting. It queries *root nameservers*, then *Top-Level Domain (TLD) nameservers* (e.g., `.com`), then the *authoritative nameserver* for the specific domain.

3. **Authoritative Nameserver** — Returns the actual IP address. The recursive resolver caches it and sends it back to your machine.

4. **Your Browser Connects** — Only now does your browser open a TCP/TLS connection to the destination.

The critical point: **steps 1 through 3 are, by default, entirely unencrypted and unprotected**. The query for `example.com` travels over UDP port 53 in plaintext.

### What a DNS Query Reveals

A single DNS query contains:
- The **full domain name** being resolved (e.g., `mail.proton.me`, `careers.competitor.com`, `therapy-finder.health`)
- Your **source IP address**
- A **transaction ID** (trivially forgeable, but tracked)
- A **timestamp** (when correlated with resolver logs)

Your DNS history is, in many ways, more revealing than your browsing history. Browsers increasingly send HTTPS traffic everywhere, obscuring page-level content. But DNS happens *before* HTTPS. It reveals not just what you visited, but *that you intended to visit it* — even if you never completed the connection.

A week of DNS logs from a typical user can reveal:
- Medical conditions (resolving health portal domains, telehealth services)
- Financial behavior (brokerage platforms, loan services, debt management)
- Political and religious affiliations (news sources, community organizations)
- Relationship status and personal struggles (dating apps, counseling platforms)
- Professional activities (competitor research, job boards, recruiter communications)

---

## Part II: DNS Leakage — Definitions and Mechanics

### What Is a DNS Leak?

The term "DNS leak" has both a narrow technical definition and a broader practical meaning that is increasingly important.

**Narrow definition:** A DNS leak occurs when a device configured to use a VPN or privacy-enhancing tool continues to send DNS queries *outside* that protected tunnel — typically to the ISP's default resolver — exposing browsing activity despite the user's belief that it is protected.

**Broader definition (and the one that matters more today):** Any scenario in which DNS queries reach an unintended or untrusted resolver, or are logged by an intermediary without the user's knowledge or meaningful consent.

### Mechanisms of DNS Leakage

DNS leaks occur through several distinct technical pathways:

#### 1. VPN Split-Tunneling Misconfiguration
Many VPN clients route only certain traffic through the tunnel while leaving the default route — including DNS — on the physical interface. Unless the VPN client explicitly forces DNS traffic through the tunnel and overrides the system resolver, queries escape to the ISP resolver.

#### 2. WebRTC and Browser-Level Leaks
WebRTC, the protocol powering browser-based video and audio, can trigger DNS resolution via the operating system's stub resolver rather than the browser's configured resolver. Even with a VPN active, WebRTC STUN requests can leak the real local IP and trigger DNS queries on the unprotected interface.

#### 3. IPv6 Leakage
Many VPN implementations tunnel IPv4 traffic while leaving IPv6 unprotected. If a domain resolves to an AAAA (IPv6) record and the system has IPv6 connectivity outside the tunnel, the DNS query and subsequent traffic bypass the VPN entirely. This affects a surprising number of consumer-grade VPN products.

#### 4. NXDOMAIN Hijacking and Transparent Proxying
Some ISPs intercept all outbound UDP port 53 traffic via transparent DNS proxies — regardless of what resolver the user has configured. A user who sets their DNS to `1.1.1.1` may find their queries are silently redirected to the ISP's own resolver without any indication that this is occurring. This is both a privacy violation and a form of DNS leak by redirection.

#### 5. Operating System Fallback Behavior
Windows, in particular, implements a feature called "Smart Multi-Homed Name Resolution" (SMHNR), which sends DNS queries to *all* configured resolvers simultaneously and accepts whichever responds first. This means queries can leak to interfaces and resolvers the user has not prioritized — including ISP resolvers — even when a privacy-conscious resolver is configured.

#### 6. Search Domain Poisoning
Corporate and home routers frequently push search domain suffixes (e.g., `corp.internal`) via DHCP. Poorly scoped search domain configurations can cause unqualified hostnames to be appended with unexpected suffixes and resolved externally, leaking internal naming structure to upstream resolvers.

---

## Part III: The Rise of DNS Transparency — ISPs as Surveillance Infrastructure

### From Passive Logging to Active Programs

For much of the internet's history, ISP DNS logging was a passive, incidental byproduct of infrastructure operation. Logs were retained for short periods for debugging and abuse response. The idea of *systematically* using DNS data for commercial or government purposes existed, but was not widely practiced.

That has changed dramatically.

### Commercial DNS Monetization

Beginning in the early 2010s — and accelerating significantly after the FCC's 2017 rollback of broadband privacy rules in the United States — ISPs have increasingly treated DNS query data as a revenue stream.

The model works as follows:

1. **Collection** — All DNS queries from customers are logged at the recursive resolver level. At scale, a major ISP like Comcast, AT&T, or Verizon handles hundreds of millions of queries per day.

2. **Profiling** — Queries are correlated with subscriber accounts (tied to billing address, real identity) and analyzed to build behavioral profiles: interest categories, lifestyle segments, purchase intent signals.

3. **Monetization** — Profiles are sold to data brokers, used directly for targeted advertising, or licensed to third-party analytics firms.

Verizon's "Relevant Mobile Advertising" program, AT&T's "Internet Preferences" program, and similar initiatives by Comcast have all used or proposed using browsing and DNS data for advertising purposes. The legal basis varies by jurisdiction, and opt-out mechanisms — when they exist — are often buried in terms of service.

### Government-Mandated DNS Logging and "Lawful Transparency"

Beyond commercial monetization, a parallel trend has emerged: government-mandated DNS logging under the guise of security or legal compliance.

**United Kingdom — Investigatory Powers Act (2016)**  
Often called the "Snoopers' Charter," this legislation requires ISPs to retain "Internet Connection Records" — which include DNS query logs — for 12 months and make them accessible to a wide range of government agencies without requiring a judicial warrant.

**Australia — Telecommunications (Interception and Access) Act**  
Australia's metadata retention scheme requires telecommunications providers to retain metadata including destination IP addresses and connection logs for two years. DNS data falls within scope.

**European Union — Network and Information Security Directive (NIS2)**  
While framed around cybersecurity, NIS2 and related EU DNS4EU initiatives push for centralized DNS infrastructure that provides governments with visibility into DNS traffic for "threat intelligence" purposes. The line between security monitoring and surveillance is, charitably, blurry.

**United States — NSA/PRISM Era and Beyond**  
Edward Snowden's 2013 disclosures revealed that the NSA operated programs collecting DNS query data at scale through programs like MUSCULAR, which tapped directly into the backbone links between data centers. Post-Snowden reforms were narrow; bulk collection of metadata — including DNS — remains legally permissible under authorities like EO 12333.

**Russia, China, Turkey, and Others**  
Authoritarian states have gone furthest, implementing DNS-level filtering, mandatory resolution through state-controlled resolvers, and systematic logging of all DNS traffic as a matter of official policy. Russia's RuNet isolation architecture and China's Great Firewall both operate substantially through DNS interception and manipulation.

### The "Transparency" Framing Problem

The term "DNS transparency" has been co-opted in a way that deserves scrutiny. In cryptography and protocol design, "transparency" is a positive property — it means operations are auditable and verifiable. Certificate Transparency (CT), for example, is genuinely good: it creates a public, append-only log of issued TLS certificates that enables detection of misissued certificates.

But "DNS transparency" in the ISP context means something different and more troubling: **it means that your DNS queries are transparent to the ISPs** — not transparent to you in any auditable sense. It is surveillance wearing the clothing of an engineering virtue.

---

## Part IV: The Security and Privacy Risk Landscape

### Threat Model: Who Are You Protecting Against?

Understanding DNS leakage risks requires clarity about threat actors. These are not equivalent:

| Threat Actor | Capability | Motivation |
|---|---|---|
| Your ISP | Passive DNS logging, traffic analysis | Commercial monetization, regulatory compliance |
| Government agencies | Compelled disclosure, direct access | Law enforcement, intelligence gathering |
| Network adversaries (MITM) | DNS spoofing, query interception | Targeted attacks, credential harvesting |
| Data brokers | Purchase of ISP data | Profiling for resale |
| Malicious resolvers | Response manipulation, logging | Phishing, malware delivery |
| Employers / network admins | DNS-level filtering and logging | Policy enforcement, productivity monitoring |

Most users face meaningful exposure in several of these categories simultaneously.

### Attack Vectors Enabled by DNS Leakage

#### DNS Cache Poisoning
An attacker who can observe a DNS query — its transaction ID and source port — can race to inject a forged response before the legitimate reply arrives. A poisoned cache entry redirects all users of that resolver to an attacker-controlled IP. In 2008, Dan Kaminsky's discovery of a fundamental DNS cache poisoning vulnerability triggered a global emergency patch cycle affecting virtually every DNS implementation. The underlying protocol weakness — no cryptographic authentication of responses — remains in plain DNS.

#### DNS Hijacking for Credential Harvesting
By poisoning a resolver's cache for `bank.example.com`, an attacker serves a convincing phishing page at the correct URL. Because the domain resolves to the wrong IP, HTTPS certificate warnings may (or may not) appear depending on whether the attacker has obtained a certificate. Lack of DNSSEC validation makes this attack easier.

#### DNS Tunneling for Data Exfiltration
Malware and attackers can use DNS as a covert channel to exfiltrate data or receive command-and-control instructions, encoding data in subdomain names (`c29tZS1kYXRh.evil.com`). Because DNS traffic is frequently allowed through firewalls that block other protocols, this technique is disturbingly effective. Organizations that log and analyze DNS can detect anomalous query patterns; those that don't are blind to this vector.

#### ISP NXDOMAIN Hijacking
Rather than returning a proper NXDOMAIN response for non-existent domains, many ISPs hijack these responses to serve ad-laden "search" pages. Beyond the obvious annoyance, this behavior:
- Breaks software that relies on NXDOMAIN responses for service discovery
- Creates a vector for phishing (users may click links on the "search" page)
- Leaks even failed resolution attempts to the ISP's advertising infrastructure

#### Timing and Correlation Attacks
Even without reading query content, an observer can correlate the *timing* and *frequency* of DNS queries against known patterns. When does a journalist resolve domains associated with a particular whistleblower platform? When does an employee resolve a competitor's job application portal? These correlations are actionable even in the absence of content.

#### OSINT and Deanonymization
Researchers, law enforcement, and malicious actors routinely use passive DNS databases — repositories of historical DNS query/response pairs — to map infrastructure, track threat actors, and identify relationships between domains and IP addresses. Data contributed to these databases often comes from ISP resolver logs (sold or leaked) and from compromised resolvers.

### Real-World Consequences

These risks are not theoretical:

- **Kazakhstan (2019):** The government mandated that all internet users install a state-issued "security certificate" — effectively a root CA — enabling man-in-the-middle attacks on all HTTPS traffic, with DNS as the entry point for traffic interception.
- **Turkey (2014, 2022):** The Turkish government repeatedly ordered ISPs to block Twitter and other services via DNS hijacking, demonstrating how DNS control translates directly to censorship capability.
- **US ISP DNS Sale (2017):** Following the FCC rule rollback, multiple major US ISPs publicly announced or quietly implemented programs to monetize DNS and browsing data.
- **BGP/DNS Hijacking of MyEtherWallet (2018):** Attackers hijacked BGP routes for AWS DNS servers, redirecting MyEtherWallet users to a phishing site, stealing approximately $150,000 in cryptocurrency in under two hours.

---

## Part V: The Technical Countermeasures

### 1. DNS over HTTPS (DoH)

**RFC 8484 | Standardized 2018**

DoH encapsulates DNS queries within HTTPS connections — the same protocol used for ordinary web traffic. Queries travel encrypted to a DoH-capable resolver, indistinguishable from regular HTTPS traffic.

**Advantages:**
- Queries are encrypted in transit; ISPs and network observers cannot read query content
- Traffic is encrypted from the system to the resolver, preventing MITM
- Blends with normal HTTPS traffic, making it difficult to block without collateral damage
- Supported natively in Firefox, Chrome, Edge, and Windows 11

**Limitations:**
- Shifts trust from the ISP to the DoH resolver (Cloudflare, Google, NextDNS, etc.) — you must trust whoever operates it
- Does not prevent the resolver from logging queries
- Can be blocked by deep packet inspection if the resolver's IP is known and blocked
- Does not protect against a malicious or compromised DoH resolver

**Implementation:**
```
# Example: systemd-resolved with DoT (Linux)
[Resolve]
DNS=1.1.1.1#cloudflare-dns.com 1.0.0.1#cloudflare-dns.com
DNSOverTLS=yes

# Firefox: about:config
network.trr.mode = 2  # DoH preferred, fallback to system
network.trr.uri = https://mozilla.cloudflare-dns.com/dns-query
```

### 2. DNS over TLS (DoT)

**RFC 7858 | Standardized 2016**

DoT encrypts DNS queries within a TLS connection on a dedicated port (TCP 853). Unlike DoH, it uses a distinct port, making it identifiable — and blockable — by network operators.

**Advantages:**
- Strong encryption with TLS; resistant to eavesdropping
- Clear separation of concerns (port 853 is exclusively DNS)
- Easier to audit and control at the network level

**Limitations:**
- Port 853 is frequently blocked by corporate firewalls and in restrictive network environments
- More easily distinguishable and blockable than DoH
- Same resolver-trust problem as DoH

### 3. DNS over QUIC (DoQ)

**RFC 9250 | Standardized 2022**

DoQ carries DNS over the QUIC transport protocol, which offers TLS 1.3 encryption, reduced connection latency (0-RTT handshakes), and multiplexing without head-of-line blocking.

**Advantages:**
- Encrypted (TLS 1.3 minimum)
- Lower latency than DoT (no TCP handshake overhead)
- Resilient to packet loss

**Limitations:**
- Nascent ecosystem; limited resolver and client support as of 2025
- QUIC's UDP-based transport is increasingly throttled or blocked on some networks

### 4. DNSSEC

**RFC 4033–4035 | Widely deployed from ~2010**

DNSSEC adds cryptographic signatures to DNS records, enabling resolvers to verify that responses are authentic and unmodified. It does **not** encrypt queries — it provides integrity, not confidentiality.

**What DNSSEC solves:**
- Cache poisoning (forged responses are rejected if signatures don't validate)
- DNS hijacking by intermediate resolvers (response integrity is verifiable)

**What DNSSEC does not solve:**
- Eavesdropping (queries are still plaintext)
- Malicious-but-legitimate resolvers (they can still log what they receive)
- ISP surveillance

**Deployment gap:** Despite being standardized since the early 2000s, DNSSEC adoption remains incomplete. Approximately 40% of global DNS traffic is DNSSEC-validated as of 2024, but many domains still lack DNSSEC signatures, and many resolvers skip validation.

### 5. Oblivious DNS over HTTPS (ODoH)

**RFC 9230 | 2022**

ODoH is an elegant cryptographic protocol that separates knowledge of *who is asking* from knowledge of *what is being asked* by introducing an untrusted proxy:

1. The client encrypts the DNS query with the resolver's public key
2. The encrypted query is sent to an **ODoH Proxy** — a separate entity that knows the client's IP but cannot read the query
3. The proxy forwards it to the **ODoH Resolver**, which can read the query but sees only the proxy's IP, not the client's
4. The response is encrypted back through the chain

**The key property:** No single entity knows both who is asking and what they are asking. Even a colluding proxy and resolver cannot reconstruct the full picture unless they share logs — which the protocol is designed to make difficult to hide.

**Current state:** Cloudflare operates an ODoH service. Adoption is growing but remains limited to privacy-focused applications and technically sophisticated users.

### 6. Encrypted Client Hello (ECH) — The Missing Piece

Even with DoH protecting DNS queries, a subtle privacy leak remains: the TLS **Server Name Indication (SNI)** field. When your browser opens an HTTPS connection, it sends the target domain name in plaintext in the TLS ClientHello message — so that servers hosting multiple domains know which certificate to present.

SNI is visible to network observers even when DNS is encrypted. An ISP cannot read the query, but can read the TLS handshake.

**ECH (formerly ESNI)** encrypts the SNI field using a public key published in the domain's DNS record (requiring HTTPS DNS record types and DNSSEC). When combined with DoH:

- DNS query: encrypted via DoH ✓
- TLS handshake SNI: encrypted via ECH ✓
- The resulting traffic reveals the *outer* domain (typically a CDN like Cloudflare) but not the actual destination ✓

ECH is supported in Firefox, Chrome (behind a flag), and Cloudflare's infrastructure. Full ecosystem deployment is ongoing.

### 7. Running Your Own Recursive Resolver

For advanced users, running a local recursive resolver eliminates dependence on a third-party resolver for confidentiality:

**Options:**
- **Unbound** — Widely-used, high-performance validating resolver with DNSSEC support
- **Pi-hole + Unbound** — Network-wide DNS filtering with local recursive resolution
- **BIND 9** — The reference implementation, powerful but complex

**What this achieves:** Queries leave your machine and go directly to authoritative servers (root → TLD → authoritative), bypassing your ISP's resolver entirely. Your ISP still sees the *destination IPs* of DNS traffic but not the query content if you use DoT/DoH for the upstream leg.

**What this doesn't achieve:** Queries to authoritative nameservers are still in plaintext. Each authoritative server you contact sees your IP. For most threat models, this is acceptable; for high-sensitivity use cases, combine with Tor or ODoH.

```bash
# Unbound local resolver config snippet (/etc/unbound/unbound.conf)
server:
    verbosity: 1
    interface: 127.0.0.1
    port: 5335
    do-ip4: yes
    do-udp: yes
    do-tcp: yes
    do-ip6: no

    # DNSSEC validation
    auto-trust-anchor-file: "/var/lib/unbound/root.key"

    # Harden against cache poisoning
    harden-glue: yes
    harden-dnssec-stripped: yes
    use-caps-for-id: yes   # 0x20 encoding randomization

    # Reduce attack surface
    hide-identity: yes
    hide-version: yes

    # Cache settings
    cache-min-ttl: 3600
    cache-max-ttl: 86400
    prefetch: yes
```

### 8. VPN with DNS Leak Protection

If using a VPN, verify that:

1. **DNS requests are routed through the VPN tunnel** — Not split-tunneled or sent to the local network's resolver
2. **Kill switch is enabled** — Drops internet traffic if the VPN disconnects, preventing leak exposure during reconnection
3. **IPv6 is disabled or tunneled** — Prevents IPv6 DNS leakage
4. **WebRTC is disabled in browser settings** — Prevents browser-level IP and DNS leaks

**Verification tools:**
- `dnsleaktest.com` — Tests which resolver(s) your queries reach
- `ipleak.net` — Comprehensive IP, DNS, and WebRTC leak test
- `browserleaks.com` — Detailed browser fingerprint and leak analysis

**Important caveat:** A VPN shifts the trust relationship from your ISP to your VPN provider. If your VPN provider logs DNS queries (many do, despite "no-log" marketing claims), you have not meaningfully improved your privacy — you've merely changed who is surveilling you. Verify VPN providers through independent audits, not marketing materials.

---

## Part VI: Choosing a Resolver — The Trust Trade-Off

If you cannot (or choose not to) run your own recursive resolver, you must place trust in a third-party resolver. This is unavoidable — the question is *who* you trust and on what basis.

| Resolver | Operator | DoH | DoT | DoQ | Logging Policy | DNSSEC |
|---|---|---|---|---|---|---|
| `1.1.1.1` | Cloudflare | ✓ | ✓ | ✓ | Purged within 24h; KPMG-audited | ✓ |
| `8.8.8.8` | Google | ✓ | ✓ | — | Logged; anonymized after 24–48h | ✓ |
| `9.9.9.9` | Quad9 (non-profit) | ✓ | ✓ | ✓ | No PII logged | ✓ |
| `94.140.14.14` | AdGuard | ✓ | ✓ | ✓ | No logging (unverified by audit) | ✓ |
| NextDNS | NextDNS Inc. | ✓ | ✓ | ✓ | Configurable; logs visible to user | ✓ |

**Recommendation framework:**
- **Default privacy:** Cloudflare `1.1.1.1` with DoH/DoT — well-audited, fast, established no-logging commitment
- **Non-profit preference:** Quad9 `9.9.9.9` — non-profit operator, malware blocking, strong privacy stance
- **Maximum control:** Self-hosted Unbound with Cloudflare or Quad9 as upstream DoT/DoH forwarder
- **Maximum privacy:** ODoH via Cloudflare + Fastly proxy partnership

---

## Part VII: Organizational and Enterprise Considerations

DNS privacy is not only a personal concern. Organizations face distinct DNS-related security risks:

### Internal DNS Leakage
Corporate devices configured with split-DNS (internal domains resolved locally, public domains via ISP) frequently leak internal DNS structure when employees work remotely. Queries for `internal.corp.example.com` may reach public resolvers, exposing organizational structure, application names, and internal service topology.

**Mitigation:** Enforce VPN-based DNS resolution for all corporate devices when outside the office network; use DNSSEC-signed internal zones; monitor for anomalous query patterns.

### DNS as an Exfiltration Channel
Attackers who have compromised internal systems routinely use DNS tunneling to exfiltrate data, since port 53 is almost universally permitted through firewalls. Each query to an attacker-controlled authoritative server carries a small payload of exfiltrated data in the subdomain label.

**Detection signals:**
- Abnormally long subdomain labels (> 50 characters)
- High query rates to a single second-level domain
- High entropy in subdomain strings (base64/hex encoded data)
- Queries for non-existent domains following a pattern

**Mitigation:** DNS traffic analysis (Zeek/Bro, Cisco Umbrella, Infoblox BloxOne); RPZ (Response Policy Zones) to block known malicious domains; anomaly detection on query volumes.

### Third-Party Resolver Risk for Organizations
Large organizations that outsource DNS resolution to cloud providers (Google, Cloudflare, AWS Route 53 Resolver) are exposing their full query profile to those providers. This includes competitive intelligence signals, merger and acquisition research activity, and partner relationship data visible in DNS patterns.

---

## Part VIII: A Practical Prevention Checklist

### For Individual Users

- [ ] **Enable DoH or DoT** in your operating system and/or browser, pointed at a trustworthy resolver
- [ ] **Verify your VPN doesn't leak DNS** using `dnsleaktest.com`
- [ ] **Disable WebRTC** in Firefox (`media.peerconnection.enabled = false` in `about:config`) or use a WebRTC control extension in Chrome
- [ ] **Disable IPv6** if your VPN doesn't tunnel it, or verify your VPN client handles it correctly
- [ ] **Enable ECH** in Firefox (enabled by default in recent versions) and Chrome (via `chrome://flags`)
- [ ] **Consider Pi-hole + Unbound** for network-wide encrypted recursive resolution
- [ ] **Audit your router's DNS settings** — many home routers ignore client-configured DNS and forward everything to the ISP resolver
- [ ] **Use a VPN provider with independently audited no-logging** — look for audits, not marketing claims
- [ ] **Run a DNS leak test** periodically, especially after VPN updates or OS upgrades

### For Developers and System Administrators

- [ ] **Sign your domains with DNSSEC** — protects users of your services from spoofing
- [ ] **Publish HTTPS DNS records** with ECH public keys for your domains
- [ ] **Monitor your organization's DNS traffic** for exfiltration patterns
- [ ] **Implement RPZ filtering** to block known malicious resolver responses
- [ ] **Audit split-DNS configurations** for internal domain leakage
- [ ] **Use DNS over TLS for upstream resolver communication** in Unbound/BIND configurations
- [ ] **Log and alert on high-entropy subdomain queries** as a DNS tunneling detection signal

---

## Conclusion: The Quiet Infrastructure of Surveillance

DNS was designed in a different world, and it shows. Every day, billions of people trust a 40-year-old unauthenticated plaintext protocol with some of their most sensitive behavioral data, with virtually no indication that this is happening and no meaningful ability to opt out of the default.

The rise of ISP DNS transparency programs — dressed in the language of security and operational necessity — has transformed this incidental exposure into systematic commercial and governmental surveillance infrastructure. The logs exist. They are retained. They are analyzed, sold, and compelled by subpoena. The query you sent three years ago asking about a health condition, a competitor, a legal resource, or a dissident journalist is, in many cases, still sitting in a database somewhere.

The tools to change this exist. DoH, DoT, ODoH, ECH, DNSSEC, self-hosted resolvers — these are not exotic technologies. They are increasingly built into the browsers and operating systems you already use. The friction between "exposed by default" and "protected with intentional configuration" has never been lower.

The internet's directory doesn't have to be an open book. But closing it requires understanding that it's open in the first place.