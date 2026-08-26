# Penetration Testing Execution Standard (PTES): A Comprehensive Guide

> **A comprehensive guide to the Penetration Testing Execution Standard — the industry's most-referenced framework for executing penetration tests. Covers the standard's seven phases in depth (pre-engagement through reporting), the PTES Technical Guidelines, comparisons with OWASP WSTG / NIST SP 800-115 / OSSTMM, modern extensions (cloud, API, AI/LLM, MITRE ATT&CK alignment), the tooling landscape, and practical execution with a banking and regulated-industry focus (MAS TRM, MAS FSM-N06 Cyber Hygiene, DORA, CBEST).**

**Author:** Jack Liu Shurui — Solution Architect, Cymbal Bank, Singapore
**Path:** `technology/penetration_testing_execution_standard_guide.md` | **Version:** 1.0 — August 2026
**Series:** Cybersecurity / Offensive Security · **Domain:** Penetration Testing · Application Security
**Reading time:** ~50 minutes

---

## Table of Contents

1. [What Is PTES?](#1-what-is-ptes)
2. [Why PTES Exists](#2-why-ptes-exists)
3. [PTES vs Other Testing Frameworks](#3-ptes-vs-other-testing-frameworks)
4. [The Seven Phases at a Glance](#4-the-seven-phases-at-a-glance)
5. [Phase 1: Pre-engagement Interactions](#5-phase-1-pre-engagement-interactions)
6. [Phase 2: Intelligence Gathering](#6-phase-2-intelligence-gathering)
7. [Phase 3: Threat Modeling](#7-phase-3-threat-modeling)
8. [Phase 4: Vulnerability Analysis](#8-phase-4-vulnerability-analysis)
9. [Phase 5: Exploitation](#9-phase-5-exploitation)
10. [Phase 6: Post-Exploitation](#10-phase-6-post-exploitation)
11. [Phase 7: Reporting](#11-phase-7-reporting)
12. [PTES in the Modern Landscape (2026)](#12-ptes-in-the-modern-landscape-2026)
13. [Running a Pentest Engagement in Practice](#13-running-a-pentest-engagement-in-practice)
14. [PTES in Banking and Regulated Industries](#14-ptes-in-banking-and-regulated-industries)
15. [The Tools Landscape by Phase](#15-the-tools-landscape-by-phase)
16. [Worked Example: A Bank Web-Application Pentest](#16-worked-example-a-bank-web-application-pentest)
17. [The Future: Where PTES Is Heading](#17-the-future-where-ptes-is-heading)
18. [Glossary](#18-glossary)
19. [References and Further Reading](#19-references-and-further-reading)

---

## 1. What Is PTES?

The **Penetration Testing Execution Standard (PTES)** is a practitioner-developed framework for planning, executing, and reporting penetration tests. It was created to answer a simple question that the industry could not answer consistently in the late 2000s: *what does a professional, defensible penetration test actually look like?*

PTES was developed by a team of information security practitioners — roughly two dozen senior penetration testers and consultants, including recognised names in offensive security — who pooled their engagement experience into a public, vendor-neutral standard. The core standard was first published in 2009, and the companion **PTES Technical Guidelines** — the per-phase "how to" documentation — were released in 2012. The project lives at ptes.org, which remains the canonical reference even though the working group has been dormant for years (the standard is complete; it did not need continuous maintenance to stay relevant).

### 1.1 What the Standard Covers

PTES is really three artefacts published under one name:

| Artefact | What it is | What it gives you |
|---|---|---|
| **The Main Standard (7 phases)** | A high-level definition of the entire penetration-testing lifecycle, from the first client conversation to the final report | A common process language: everyone — client, tester, auditor, regulator — can refer to the same named phases |
| **The PTES Technical Guidelines** | Deep per-phase technique documentation: what to do, what to look for, what to test in each phase | Technical depth guidance that converts the phase names into concrete testing activity |
| **The Deliverables Definition** | A standardised report structure and engagement-documentation expectations | Standardized reporting: clients receive comparable, complete, evidence-backed deliverables |

The seven phases that form the backbone of the standard are:

```
Pre-engagement Interactions → Intelligence Gathering → Threat Modeling →
Vulnerability Analysis → Exploitation → Post-Exploitation → Reporting
```

### 1.2 A Standard, Not a Methodology

A useful distinction: PTES is an **execution standard** — it defines *what a complete test must contain and in what order*, and it prescribes *the level of rigor expected* at each step. It is deliberately not a rigid playbook like a compliance checklist; it expects the tester to apply professional judgement within each phase. This is exactly why it has aged well: the phase structure is process-shaped, not tool-shaped. Nmap's replacement, the next generation of web scanners, and the rise of cloud and AI systems have all changed *what happens inside* the phases, but not the phases themselves.

### 1.3 Who Uses It

- **Pentest vendors and consultancies** — most commercial penetration-testing methodologies are PTES-aligned or PTES-derived; the "7 phases" language appears in virtually every proposal and Statement of Work (SOW).
- **In-house security teams** — as the template for internal red-team/pentest programs and for scoping external engagements.
- **Procurement and vendor risk** — RFPs routinely ask for "PTES-aligned methodology", because the phase structure gives buyers a way to compare otherwise incomparable vendor proposals.
- **Auditors and regulators** — the report structure PTES defines maps cleanly to what audit and regulatory reviewers expect to see (scope, methodology, findings, risk ratings, remediation).

---

## 2. Why PTES Exists

### 2.1 The Problem: Inconsistent Penetration Testing

Before PTES, penetration testing was delivered the way jazz was played — same instrument, no shared sheet music. Every firm ran its own process, and the quality gap between vendors was enormous:

- **No common language.** One vendor's "network assessment" covered a port scan and a vulnerability scan; another's meant full exploitation with domain-admin proof. Clients could not compare proposals because the words meant different things to different vendors.
- **Variable quality.** A "pentest" could be a Nessus report printed and bound, or a six-week adversary simulation — both sold under the same name. There was no floor on rigor.
- **Scope disputes.** Without a standardised scoping discipline, engagements routinely drifted: testers touched systems the client never authorised, or missed the systems the client *thought* were in scope. Both failure modes are legal and reputational hazards.
- **No verification standard.** Findings were asserted without reproduction steps or evidence, so clients could not verify, remediate, or retest meaningfully.
- **Legal ambiguity.** Testing without clear, written rules of engagement is unauthorized access — the difference between a pentest and a crime is a signed piece of paper. The industry lacked a standard template for that paper.

### 2.2 What PTES Provides

| Problem | PTES response |
|---|---|
| No common language | A fixed 7-phase lifecycle with defined names and boundaries — the shared vocabulary of the industry since |
| Variable quality | Minimum expectations per phase in the Technical Guidelines (e.g., intelligence gathering is not optional; exploitation must prove, not guess) |
| Scope disputes | Phase 1 (Pre-engagement Interactions) makes scope, rules of engagement, and authorisation first-class deliverables before any packet is sent |
| Unverifiable findings | Reporting phase mandates evidence, reproduction steps, and retest process |
| Legal ambiguity | Written RoE + authorization letter + NDA as standard pre-engagement deliverables |
| Incomparable vendor bids | Buyers can structure RFPs and SOWs around the 7 phases and compare like-for-like |

The commercial value is real: PTES gave the market a way to buy and sell penetration testing as a *defined service* rather than a trust-me engagement, and it gave testers a defensible methodology to fall back on when a finding or a scope decision is challenged.

---

## 3. PTES vs Other Testing Frameworks

PTES does not exist in a vacuum. The penetration-testing framework landscape has four major players, plus a set of adjacent standards and certifications. Understanding where each one fits is essential for choosing (and defending) a testing approach — especially in a regulated environment where the choice will be reviewed by auditors.

### 3.1 The Landscape

**OWASP WSTG — Web Security Testing Guide.** The Open Worldwide Application Security Project's testing bible for *web applications*. A structured checklist of ~90–100 test cases across categories (Information Gathering, Configuration Management, Authentication, Authorization, Session Management, Input Validation, Cryptography, Business Logic, Client-side, API testing in v4.2+), each with an objective, test steps, and expected results. The current stable release is v4.2 (2021), with v5.0 in development (verify current status). WSTG is the de-facto checklist for the web-application slice of any pentest.

**NIST SP 800-115 — Technical Guide to Information Security Testing and Assessment.** US-government guidance (2008) for security testing and assessment, covering a four-phase process — Planning, Discovery, Attack, Reporting — with chapters on network, host, application, and wireless testing. It is control- and process-oriented (it pairs with NIST SP 800-53 control CA-8, "Penetration Testing"), aimed at federal agencies and their contractors. It is guidance-level: it tells you what an assessment should include, not how to run each individual test.

**OSSTMM — Open Source Security Testing Methodology Manual.** ISECOM's methodology, current major version 3 (2010). The most academically rigorous of the four: it treats security testing as a scientific, measurable discipline across five channels (Human, Physical, Wireless, Telecommunications, Data Networks) and produces quantified security metrics (e.g., RAV — Risk Assessment Values) rather than finding lists. Heavy process overhead; strong for formal certification programs (OPST/OPSE) and for organizations that want measurable security posture, weaker as a day-to-day execution checklist.

**PTES.** The execution-focused standard: 7 phases, pragmatic depth, and the most widely referenced framework in commercial pentesting, RFPs, and vendor SOWs. It deliberately sits at the process level — it tells you *what to do in what order* and leaves technique selection to the tester, which is why it composes cleanly with the other frameworks.

### 3.2 Comparison Table

| Framework | Steward | Focus | Phase/structure | Depth & style | Best for |
|---|---|---|---|---|---|
| **PTES** | PTES working group (community) | Full pentest execution lifecycle | 7 phases: Pre-engagement → Intelligence → Threat Modeling → Vulnerability Analysis → Exploitation → Post-Exploitation → Reporting | Process-level; deep per-phase technical guidelines; practitioner tone | Commercial & in-house pentests, RFPs/SOWs, vendor selection, regulated-industry evidence |
| **OWASP WSTG** | OWASP Foundation | Web application testing | ~12 test categories, ~100 individual test cases | Checklist-level; each test has objective + steps + expected result | Web-app security testing, OWASP Top 10 coverage, developer-facing verification |
| **NIST SP 800-115** | NIST (US government) | Technical security testing & assessment (all layers) | 4 phases: Planning, Discovery, Attack, Reporting | Guidance-level; control-aligned (SP 800-53 CA-8) | US federal/government engagements, control-assessment programs |
| **OSSTMM** | ISECOM | Holistic security testing methodology | 5 channels (human, physical, wireless, telecom, data networks) | Methodology-heavy; quantified metrics (RAV), scientific rigor | Formal methodology certifications, posture measurement, low-finding environments |

### 3.3 How They Combine (and Adjacent Standards)

In practice the frameworks are complementary, not competing — a professional engagement usually layers several of them:

- **PTES is the umbrella.** It provides the engagement structure (Phase 1–7) that the others slot into.
- **WSTG is the checklist inside Phase 4.** When the PTES Vulnerability Analysis phase hits a web application, the tester works the WSTG test cases.
- **MITRE ATT&CK is the language for Phases 3 and 6.** Threat modeling (Phase 3) maps assets to adversary techniques; post-exploitation (Phase 6) is structured as technique chains. (See §12.5.)
- **NIST SP 800-115** remains the reference for government-aligned programs; many federal contractors run PTES-style engagements mapped to 800-115's four phases for compliance reporting.

Adjacent instruments worth knowing: **PCI DSS Requirement 11.3** (annual penetration testing + retest after significant changes, with ASV scans under 11.2), **OWASP ASVS** (verification levels for application security), **ISSAF** (an older, now largely dormant framework), **CREST / CHECK** (UK-aligned scheme-based testing), and the certifications discussed in §17.4. For the banking angle, the relevant overlay is regulatory security-testing rules (MAS TRM, DORA, CBEST — see §14), which *require* testing and, in most cases, *expect* it to follow a recognized standard such as PTES.

---

## 4. The Seven Phases at a Glance

The entire PTES standard is the elaboration of one line: a penetration test runs seven phases, in order, and each phase produces a defined output that feeds the next.

```
 1. Pre-engagement   2. Intelligence      3. Threat           4. Vulnerability
    Interactions         Gathering            Modeling            Analysis
  (business & legal)   (reconnaissance)    (what to attack)    (what's weak)
        │                   │                   │                   │
        ▼                   ▼                   ▼                   ▼
 5. Exploitation ──▶ 6. Post-Exploitation ──▶ 7. Reporting
  (prove it)          (what access means)      (the deliverable)
```

| Phase | Core question | Primary output |
|---|---|---|
| 1. Pre-engagement Interactions | What are we allowed to test, when, and under what rules? | Signed contract, RoE, scope document, authorization letter |
| 2. Intelligence Gathering | What does the target look like from outside? | Target profile: attack surface inventory, tech stack, entry points |
| 3. Threat Modeling | What would an attacker want, and how would they get it? | Prioritized attack scenarios |
| 4. Vulnerability Analysis | Which weaknesses actually exist? | Verified vulnerability list with severity ratings |
| 5. Exploitation | Can the weaknesses be proven exploitable? | Proof of compromise: access obtained, evidence |
| 6. Post-Exploitation | What could an attacker *do* with that access? | Impact assessment: depth of compromise, business systems at risk |
| 7. Reporting | What did we find, and what should be done? | Executive summary + technical report + remediation roadmap |


### 4.1 Typical Effort Allocation

The seven phases are not equal in cost. A typical commercial engagement (web-app or network) spends tester-days roughly as follows — useful for estimating SOWs and for spotting proposals whose phase allocation is unrealistic:

| Phase | Typical share of tester-days | Notes |
|---|---|---|
| 1. Pre-engagement | 5–10% | Small in days, huge in leverage — scope disputes cost far more later |
| 2. Intelligence gathering | 10–15% | Heavily automated with modern tooling, but the analysis is manual |
| 3. Threat modeling | 5–10% | Small but decisive — it sets the test plan for Phases 4–6 |
| 4. Vulnerability analysis | 25–35% | The largest block: scanning + manual verification + research |
| 5. Exploitation | 15–20% | Highly variable — depends on what Phase 4 verified |
| 6. Post-Exploitation | 10–15% | Lateral movement and business-impact assessment |
| 7. Reporting | 15–20% | Underestimated by every first-time buyer |

A bid that allocates 60% to "scanning days" and 2% to reporting is not a PTES-aligned bid — reporting is a fifth of the work.
Two properties of the phase model matter in practice:

- **It is iterative, not strictly linear.** Intelligence gathering often reveals new assets that refine the threat model; exploitation findings feed back into vulnerability analysis; and reporting is a running activity from day one (evidence must be captured as it is produced, not reconstructed afterwards). The phases are a *discipline*, not a waterfall.
- **The later phases depend entirely on the earlier ones.** A test that skips threat modeling produces a list of findings with no business narrative; a test that skips pre-engagement produces legal risk. The rigor of the front phases is what makes the back phases defensible — this is the PTES philosophy in one sentence.
## 5. Phase 1: Pre-engagement Interactions

The first phase is the most important phase in the standard — and the one most often rushed. Every other phase happens inside the legal and business envelope built here. PTES puts it first because a penetration test is an *authorized intrusion*: the authorization, its boundaries, and its rules are the entire difference between a professional engagement and a criminal act. In banking, this phase also carries regulatory weight — the pre-engagement documentation becomes audit evidence (§14.4).

### 5.1 Scope Definition

Scope is the precise answer to "what are we allowed to test?" — agreed in writing before testing begins:

| Scope element | What it captures | Banking example |
|---|---|---|
| **In-scope targets** | The explicit list of what may be tested | `internet-banking.examplebank.com`, the `/api/v2` endpoints, the mobile app build for iOS/Android |
| **IP ranges / domains** | Network-level boundaries, expressed as CIDR blocks and DNS zones | `203.0.113.0/24` (DMZ), `*.examplebank.com`, the SWIFT interface IPs if agreed |
| **Out-of-scope targets** | Explicitly excluded systems — never implied, always written | The HR systems, the card-scheme network segment, any third-party-hosted vendor apps |
| **Exclusions** | Systems that must not be touched even if discovered in-scope during testing | The core banking mainframe, the payment switch, the failover/DR site during business hours |
| **Sensitive systems** | Systems whose testing needs special handling (outages, data sensitivity) | The customer database, the fraud-detection platform, the FX trading engine |
| **The "crown jewels" discussion** | The conversation that identifies what the business truly cannot lose — these get priority in threat modeling (§7.1) and post-exploitation impact assessment (§10.5) | Customer accounts + balances, the payment rails, the RTGS connectivity, the KYC repository |

A common scoping failure mode is ambiguity by omission: "the network" as a scope statement. PTES-aligned scoping requires enumeration of targets at a level of precision that a third party could verify — and it expects the *client* to confirm the list, because only the client knows all the systems and their criticality.

### 5.2 Rules of Engagement (RoE)

The RoE document answers "how may we test?" — the constraints that keep the engagement safe, available, and legal:

- **Time windows** — when testing may run (e.g., 09:00–18:00 SGT business days for the production portal; 24/7 for the staging environment).
- **Blackout periods** — windows where testing is forbidden regardless of scope: month-end close for a bank, settlement windows, the last hour before market open, regulator-mandated freeze periods.
- **Production impact limits** — the acceptable ceiling on resource consumption (e.g., "no more than 50 concurrent requests per second to the production API"), and the obligation to stop immediately on any sign of degradation.
- **DoS testing allowance** — denial-of-service testing is *never assumed*: it requires explicit permission, and in most banking engagements it is either prohibited outright or restricted to controlled, rate-limited simulation against non-production environments.
- **Social engineering permission** — phishing, vishing, and physical pretexting are separate authorizations, often with their own sub-rules (which staff may be targeted, whether the client's own awareness team must be informed, whether simulated phishing must be pre-announced to the CISO).
- **Physical testing permission** — lock-picking, tailgating, badge-cloning, and site entry are their own legal category; they require written authorization, often site-level coordination, and insurance considerations.
- **Technique constraints** — what is disallowed even in-scope: destructive exploits, persistence beyond the engagement window, exfiltration of real data, attacks on third parties reachable from the target (pivoting boundaries), use of specific tools the client's SOC flags.

### 5.3 Legal Authorizations

- **Written authorization** — the engagement contract and authorization letter, signed by someone with actual authority to grant it (in a bank: the CISO or a delegated executive, not the IT helpdesk manager). The letter names the testers, the targets, the window, and the authorized activities.
- **Third-party authorization** — when the target is not wholly owned by the client (cloud providers, SaaS vendors, hosting providers, subsidiaries with separate legal entities, shared infrastructure), authorization must flow from the *actual* system owner. This is where cloud pentesting policies (§12.2) matter: AWS, Azure, and GCP each have published rules for customer-initiated testing.
- **Insurance** — professional indemnity / cyber liability coverage for the testing firm, and clarity on who carries the risk if an authorized test causes an incident.
- **Data handling** — contractual terms for what happens to any data encountered: no real customer data in findings, encryption of evidence, retention and destruction schedules, and the legal basis for processing if PII is unavoidably captured (PDPA-relevant in Singapore — cross-reference the data-protection discussion in the [LLM development risks guide](llm_development_risks_security_guide.md)).

### 5.4 Communication and Reporting Structure

- **Points of contact** — a named liaison on each side: the tester's engagement lead and the client's security liaison (the person who can make decisions mid-engagement).
- **Escalation paths** — who to call, in order, if something goes wrong (an outage, an unexpected production impact, a finding that suggests an active external attacker, discovery of an out-of-scope critical issue).
- **Status reporting cadence** — daily status summaries, weekly formal updates, and the debrief schedule; in banking engagements the cadence is often tighter (daily 09:00 stand-up) because production systems are involved.

### 5.5 The Commercial / Project Side

Cost, timeline, and resources are pre-engagement outputs too: the number of tester-days per phase, the duration (a typical web-app pentest runs 1–3 weeks of active testing), the environment access needed (VPN, staging credentials, architecture diagrams — or none, for a black-box test), and the retest window (§11.5). PTES treats the commercial agreement as part of the same conversation as scope — a proposal that cannot be scoped precisely cannot be priced honestly.

### 5.6 Pre-engagement Deliverables

| Deliverable | Contents | Purpose |
|---|---|---|
| **Engagement contract** | Services, fees, timeline, liability, confidentiality, IP in findings | The commercial + legal backbone |
| **Rules of Engagement (RoE)** | Time windows, techniques, constraints, escalation, stop conditions | The operational rulebook |
| **Scope document** | In-scope/out-of-scope targets, IP ranges, domains, exclusions | The technical boundary |
| **Authorization letter** | Sign-off on targets + testers + window by an authorized officer | The legal authorization |
| **NDA** | Mutual confidentiality terms | Data protection from day one |

In banking practice, the same packet (usually with a third-party-risk assessment of the vendor attached) is what the vendor-management function files before a testing firm touches anything — see §14.5.

### 5.7 The Pre-engagement Questionnaire

The PTES technical guidelines for pre-engagement effectively form a questionnaire the tester works through with the client before anything is signed. The high-value questions:

- **Business:** what does the organization do, and which systems would hurt most if compromised? (The crown-jewels conversation, §5.1.)
- **Technical:** what are the exact targets — domains, ranges, applications, APIs? Which versions? Are there test environments with production-like data?
- **Operational:** when is the business quiet enough to test? Which windows are sacred (settlement, month-end close, market hours)?
- **Legal:** who signs the authorization, and does the authority cover all listed targets, including subsidiaries and cloud providers?
- **Security operations:** will the SOC be informed? How will testing traffic be flagged so the tester is not mistaken for a real attacker?
- **Constraints:** what is forbidden even in scope — DoS, social engineering, physical, destructive payloads, real-data handling?

The output of the questionnaire is the pre-engagement packet (§5.6). The conservative default: if a question cannot be answered, the corresponding scope element stays out of scope.

---

## 6. Phase 2: Intelligence Gathering

Intelligence Gathering is the reconnaissance phase: building a picture of the target *before* any testing begins, using everything that is learnable about it. PTES's technical guideline for this phase is one of its deepest — it expects systematic enumeration at four layers (network, host, application, personnel) and distinguishes clearly between passive and active collection.

### 6.1 Passive vs Active Intelligence

- **Passive intelligence** — collecting information about the target without touching it: public records, search engines, certificate transparency logs, DNS records, job postings. No packets are sent to the target's infrastructure. Legally low-risk, operationally undetectable, and often remarkably rich.
- **Active intelligence** — directly probing the target: port scans, service fingerprinting, banner grabbing. This is where engagement boundaries and rate limits actually bind; active recon against production banking infrastructure must stay inside the RoE's time windows and technique constraints.

The PTES guideline's stance: *start passive, go active only when the passive picture is exhausted and the RoE permits it.*

### 6.2 Intelligence Sources (OSINT)

Open-source intelligence is the backbone of this phase — see the dedicated [OSINT guide](osint_guide.md) in this series for the full tradecraft. The core sources:

| Source | What it reveals | Example tools / venues |
|---|---|---|
| **Search engines** | Indexed web properties, cached pages, exposed admin panels, interesting file types | Google/Bing dorking (`site:`, `filetype:`, `inurl:`) |
| **WHOIS / RDAP** | Domain registration, registrar, sometimes the org's other domains | `whois`, RDAP queries |
| **DNS enumeration** | Hostnames, subdomains, MX/NS/TXT records, mail and VPN entry points | `dig`, Amass, dnsrecon, certificate transparency (crt.sh) |
| **Certificate transparency** | Every TLS certificate ever issued for the domains — a goldmine of subdomains | crt.sh, Censys |
| **Shodan / Censys** | Internet-exposed services: banners, ports, device types, misconfigurations | Shodan, Censys |
| **Social media / professional networks** | Employee names, roles, tech-stack chatter, vendor relationships | LinkedIn, X, GitHub |
| **Leaked credentials** | Credentials from past breaches that may still work (HIBP, breach corpora) | HaveIBeenPwned, breach databases |
| **GitHub code search** | Accidental commits of keys, configs, internal hostnames, API endpoints | GitHub search, gitleaks against public repos |
| **Job postings** | The tech stack in plain sight: "we run Kubernetes, Kafka, Oracle, and need a Go developer for our payments platform" | Job boards, company career pages |
| **The dark web angle** | Mentions of the organization in forums, marketplaces, leaked corpora — for banks, a standing monitoring requirement, not just a pentest input | Dark-web monitoring services (typically commercial) |

### 6.3 Active Techniques

- **Port scanning** — systematic discovery of open ports across the scoped ranges: Nmap for depth (service/version detection, scripted checks), Masscan for speed over large ranges.
- **Service fingerprinting** — identifying the exact software and version behind each open port (Nmap `-sV`, banner analysis), which later maps to known CVEs in Phase 4.
- **Network mapping** — building the topology: which hosts sit in the DMZ, which internal services leak through, load-balancer and WAF presence (detectable via header and behaviour analysis), adjacent infrastructure.
- **Banner grabbing** — reading service banners and response headers (`Server:`, `X-Powered-By:`, TLS certificate details) for version and framework identification.

### 6.4 The Intelligence Output: The Target Profile

Intelligence Gathering concludes with a structured target profile, which drives everything that follows:

- **Attack surface inventory** — every discovered host, service, endpoint, and domain, with its exposure.
- **Entry points** — the places an attacker (or tester) could get in: VPN portals, web apps, APIs, mail gateways, exposed management interfaces, third-party integrations.
- **Technology stack** — frameworks, versions, middleware, cloud provider, CDN/WAF — the raw material for vulnerability research in Phase 4.
- **Personnel information** — names, roles, contact patterns — the raw material for social-engineering vectors (used only if Phase 1 authorized them).

### 6.5 The PTES Technical Guideline Depth: The Four Enumeration Layers

The PTES technical guidelines structure intelligence gathering into four layers, each with its own technique catalogue:

| Layer | Scope | Representative activities |
|---|---|---|
| **Network** | The organization's footprint on the internet | IP-range discovery, DNS zone transfers (rarely permitted anymore), BGP/ASN enumeration, cloud asset discovery |
| **Host** | Individual systems | Port scanning, service fingerprinting, OS detection, banner grabbing, TLS analysis |
| **Application** | Web apps, APIs, mobile backends | Subdomain enumeration, robots.txt/sitemap, tech-stack fingerprinting, API endpoint discovery, directory brute-forcing (active — rate-limited per RoE) |
| **Personnel** | People as a vector | OSINT on employees, phishing-target identification, role-based targeting (only under social-engineering authorization) |

The output quality bar is high: PTES expects the profile to be *sufficient to plan the attack*, not merely a scan report. The difference shows up in Phase 3 — threat modeling against a rich target profile produces prioritized, realistic scenarios; threat modeling against a port list produces generic checklists.

---

## 7. Phase 3: Threat Modeling

Threat Modeling is the analysis phase: before probing for weaknesses, decide *what an attacker would actually do to this organization* and rank those scenarios by likelihood and impact. It is the phase that turns a pentest from a vulnerability sweep into a security assessment. PTES positions it between intelligence and testing deliberately — the scenarios produced here decide what Phase 4 and 5 focus on, which is why the phase output is "prioritized attack scenarios: what to test first."

### 7.1 Asset Identification

What matters to the business? The crown jewels identified in Phase 1 are the anchor: for a bank, customer accounts and balances, payment initiation and settlement rails, the RTGS connection, KYC/AML data, the trading platform, and the systems whose downtime directly triggers regulatory notification. Everything else is supporting cast. Assets get ranked by confidentiality, integrity, and availability impact — the same CIA lens auditors use.

### 7.2 Attack Surface Analysis

For each crown jewel, enumerate the ways in: the internet-facing applications that touch it, the APIs that move money, the integrations with third parties, the internal network paths from any compromised workstation, the people who can authorize payments, the physical sites where terminals and printers live. The attack surface is the bridge between Phase 2's target profile and Phase 4's testing list.

### 7.3 Threat Actor Modeling

Who would attack, and with what capability?

| Threat agent | Motivation | Capability | Typical target |
|---|---|---|---|
| **Organized cybercrime** | Money — fraud, ransomware, card theft | High; well-resourced, professional tooling | Payment rails, customer credentials, Ransomware-ready crown jewels |
| **Insiders** | Fraud, sabotage, espionage | Varies; privileged by position | Payment systems, customer data, trading systems |
| **Nation-state / APT** | Espionage, destabilization, intellectual property | Very high; zero-days, stealth | SWIFT infrastructure, research, regulator-touching systems |
| **Hacktivists** | Reputation damage | Medium | Public-facing services, defacement |
| **Script kiddies / automated threats** | Opportunism, credential stuffing | Low; commodity tools | Any exposed service; bots at scale |

### 7.4 Attack Vector Identification and Business Impact Assessment

For each asset, list the plausible vectors (phishing → workstation → lateral movement → payment system; exposed API → broken auth → account takeover; supply-chain vendor compromise → third-party access), then assess business impact: *what actually happens if this is compromised?* For a bank, the impact dimensions are financial loss, regulatory action (MAS expects notification of significant incidents), customer harm and compensation, reputational damage, and legal liability. Impact assessment is where the tester must think like a banker — a "CVSS 9.9 in a non-financial subsystem" is less urgent than a "CVSS 6.5 in the payment initiation path," and the threat model should say so.

### 7.5 Threat Modeling Methodologies

- **STRIDE** — the classic Microsoft classification of threat types per element: Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege. Fast, structured, and excellent for decomposing a specific application or API.
- **Attack trees** — hierarchical "how would I get X?" structures: the root is the goal (e.g., "initiate an unauthorized transfer"), branches are the alternative paths, leaves are concrete techniques. Attacks trees make the chaining of findings visible — which is exactly what post-exploitation (§10) will later validate.
- **MITRE ATT&CK mapping** — the modern standard. Map the organization's crown jewels to ATT&CK tactics (Initial Access, Execution, Persistence, Privilege Escalation, Defense Evasion, Credential Access, Discovery, Lateral Movement, Collection, Command & Control, Exfiltration, Impact) and select the techniques a realistic adversary would chain. PTES's threat-modeling phase aligns naturally with ATT&CK: Phase 3 chooses the scenarios, and Phases 5–6 execute them as ATT&CK technique chains. See §12.5 for the full mapping.

### 7.6 The Output: Prioritized Attack Scenarios

The phase ends with a ranked scenario list — usually 3–10 scenarios — each with: the asset targeted, the threat agent assumed, the attack path (often an ATT&CK chain), the expected impact, and the testing priority. These scenarios become the test plan for Phases 4–6. A banking example set might be: (1) account takeover via the internet-banking portal and its APIs; (2) unauthorized transfer via API abuse / broken business logic; (3) lateral movement from a compromised workstation to the payment domain; (4) customer-data exfiltration via an exposed internal API; (5) ransomware simulation on a critical application server. Scenario (1) is what the web-app test focuses on; scenario (3) drives the internal network test.

---

## 8. Phase 4: Vulnerability Analysis

Vulnerability Analysis is the identification phase: finding, verifying, and rating the weaknesses that exist in the scoped target. It is the phase most people picture when they hear "pentest," but PTES's discipline matters here more than anywhere: the *output is a verified, prioritized list* — not a raw scanner dump.

### 8.1 Automated Vulnerability Scanning

Scanners (Nessus, OpenVAS, Qualys) sweep the target for known issues: missing patches, weak configurations, known CVEs, default credentials. The scanner output is a starting point, never a conclusion:

- **Nessus** (Tenable) — the commercial de-facto standard; deep plugin coverage, policy-based.
- **OpenVAS** (Greenbone) — the leading open-source scanner; free, capable, weaker reporting.
- **Qualys** — SaaS scanner, common in enterprises that already run Qualys VM for continuous scanning.
- **Web-focused tools** — Nikto (basic web scanner), Burp Suite's active scanner, nuclei (fast template-based scanning with a huge community template library).

### 8.2 Manual Verification: The False-Positive Problem

Every scanner produces false positives — and in modern environments the noise is brutal (WAF-answered requests, version-guessing errors, checks that cannot see application context). PTES expects every finding that will appear in the report to be *manually verified*: confirm the service is real, the version claim is right, the vulnerability is actually reachable in this deployment, and — critically — whether it is exploitable *in this application's context* (a scanner cannot tell you that the "SQL injection" it found is actually behind parameterized queries). The verification pass is what separates a professional finding from a scanner dump, and it is where tester experience shows.

### 8.3 Vulnerability Research

For each verified issue, research the surrounding facts: the CVE record and CVSS score in NVD, the exploit availability in ExploitDB or Metasploit modules, the vendor advisory and patch status, and the real-world exploit activity (is this CVE being used in the wild right now? For banking threat modeling, active exploitation is a priority multiplier). This research feeds both the severity rating and the Phase 5 exploitation plan.

### 8.4 Fuzzing

Fuzzing — feeding malformed, unexpected, or boundary inputs to parsers and APIs — finds the bugs scanners miss because they are not yet known (no CVE, no signature). In web/API testing this is structured fuzzing of parameters, file uploads, and serialization boundaries; in protocol testing it targets parsers (network services, file parsers in internal tools). Fuzzing output is raw: crashes, hangs, unexpected status codes, behavioural anomalies — each then triaged for security significance.

### 8.5 Configuration Review

A large fraction of serious findings are configuration failures, not code flaws: default credentials, exposed management interfaces, missing security headers, permissive CORS, debug mode in production, verbose error pages, weak TLS settings, over-privileged service accounts. Configuration review runs alongside scanning and is especially important in banking, where hardened baseline expectations (CIS benchmarks, bank-specific hardening standards) are usually documented — the test can compare reality to the stated standard. Cross-reference the [OpenSCAP guide](openscap_guide.md) in this series for automated configuration/compliance assessment.

### 8.6 The Output: Verified Findings with CVSS

The phase output is a verified vulnerability list, each finding carrying:

- A precise description and affected component
- Reproduction evidence (from manual verification)
- A **CVSS** score (Common Vulnerability Scoring System — the industry severity metric, currently v3.1/v4.0) — CVSS captures base technical severity; the *business* severity is assigned later in reporting (§11.3), combining CVSS with the Phase 3 impact assessment
- References (CVE, vendor advisory, exploit availability)

### 8.7 The PTES Technical Guideline: Testing Types

The technical guidelines scope Phase 4 by test type, each with its own technique catalogue and toolchain:

| Test type | Focus | Representative checks |
|---|---|---|
| **Network** | Infrastructure, services, segmentation | Port/service inventory, known-vuln checks, TLS/crypto posture, SNMP/management exposure |
| **Web application** | Apps and APIs | The OWASP Top 10 checklist (injection, broken auth, XSS, IDOR, SSRF, business logic…) via WSTG test cases — the PTES Phase 4 web layer and WSTG are effectively the same work |
| **Wireless** | Wi-Fi and radio-adjacent exposure | Rogue APs, weak WPA2/3 configs, client attacks, (with RoE) on-site assessment |
| **Mobile** | iOS/Android apps and their backends | Insecure storage, weak certificate pinning, reverse-engineering resistance, API abuse via the app |
| **Social engineering** | People as a control | Simulated phishing, pretexting — only with explicit Phase 1 authorization |
| **Physical** | Sites and facilities | Tailgating, badge cloning, open workstations — only with explicit Phase 1 authorization and site coordination |

### 8.8 CVSS Severity Bands

CVSS scores map to severity bands that reports and remediation SLAs reference (v3.1 bands below; v4.0 — released 2023 — uses a different qualitative model, so the report must state the version used):

| CVSS v3.1 base score | Severity | Typical banking remediation SLA |
|---|---|---|
| 9.0–10.0 | Critical | Fix within 24–72 hours, or mitigate with compensating controls immediately |
| 7.0–8.9 | High | Fix within 30 days |
| 4.0–6.9 | Medium | Fix within 90–180 days |
| 0.1–3.9 | Low | Fix at the next scheduled change window |

Two caveats the report should state explicitly: CVSS measures *technical* severity in isolation (§11.3 — business risk is a separate rating), and CVSS scores are only as good as the context the analyst supplies — a raw NVD score for a component in an unusual deployment is a starting point, not a verdict.
## 9. Phase 5: Exploitation

Exploitation is where the test stops being theoretical: the tester actively proves that a verified vulnerability is exploitable and gains access. Everything before this phase produced candidates; this phase produces *proof*.

### 9.1 The Goal: Confirm Exploitability

The core deliverable of Phase 5 is confirmation — for each high-value finding, demonstrate that an attacker could actually leverage it. A SQL injection that is verified present but not demonstrably exploitable gets a different treatment than one that yields a database dump. Exploitation turns "the scanner says this endpoint is vulnerable" into "here is the evidence, here is the access obtained."

### 9.2 Exploitation Techniques

| Technique | What it is | When to use it |
|---|---|---|
| **Known exploits** | Public exploit code for known CVEs (ExploitDB, Metasploit modules, PoC repositories) | Fast, high-confidence verification for patched-vs-unpatched checks — but must be reviewed before running; PoCs are frequently unreliable or destructive |
| **Custom exploits** | Exploits written for this engagement (logic flaws, chained issues, bespoke injection payloads) | Where no public PoC exists — the highest-value work in the phase, and where the tester's skill shows |
| **Metasploit / exploitation frameworks** | Post-exploitation-ready frameworks with modular payloads and staged access | Standard for network-layer exploitation and initial footholds; Framework edition is open source, Pro adds features |
| **Chaining vulnerabilities** | Combining several medium findings into one high-impact compromise (e.g., IDOR + weak MFA reset + missing rate limit = account takeover) | The realistic attacker path — most serious compromises are chains, not single bugs |

### 9.3 The Constraints: Stay in Scope, Do No Harm

PTES's exploitation phase is explicitly bounded by the Phase 1 RoE, and this is where the "proof vs damage" balance is sharpest:

- **Stay in scope** — exploitation must not cross into out-of-scope systems, third parties, or the production core (payment switch, mainframe) unless explicitly authorized. Pivoting discovered during exploitation is reported, not pursued, without a scope change.
- **Do no harm** — the proof of compromise must never become the compromise: no destructive payloads, no data destruction, no service outages, no real data exfiltration, and production impact kept inside the agreed limits. If a proof requires a risky action, the tester escalates to the client liaison first (§13.3).
- **Least-impact proof** — demonstrate access with minimal footprint (a marker file, a benign query, a screenshot of a dashboard) rather than maximum damage. In banking, "proof of access to the customer database" is a count of records *visible*, never an actual extraction (see §10.4).

### 9.4 The Output: Proof of Compromise

Each successful exploitation produces: the access obtained (user context, host, network position), the technique chain used, and **evidence** — screenshots, tool transcripts, packet captures, timestamps — captured properly at the moment of success. Evidence hygiene is a Phase 5 discipline: it is captured during exploitation so the Phase 7 report can reproduce and defend every claim. A common professional standard: every finding in the final report must trace back to evidence captured in this phase.

---

## 10. Phase 6: Post-Exploitation

Post-Exploitation answers the question that matters to boards: *so what?* Once access is obtained, what can an attacker actually do to this organization? This phase converts "we got in" into a quantified business impact assessment — without actually causing the damage.

### 10.1 Privilege Escalation

From the initial foothold, escalate access: **horizontal** (move from one user's account to another's — e.g., exploiting an IDOR to reach a colleague's session) and **vertical** (raise privilege — e.g., a service-account compromise to local administrator, or a misconfigured sudo to root). Every escalation step is an ATT&CK Privilege Escalation technique and gets documented with evidence.

### 10.2 Lateral Movement (Pivoting)

Move through the network from the compromised host to others — the attacker behaviour that turns a single beachhead into an enterprise compromise. The compromised host becomes a pivot point: route traffic through it (SOCKS proxying, port forwarding), harvest credentials from it (LSASS, browser stores, config files), and use those credentials to reach internal systems. In banking, the target of lateral movement is usually the network segment where money moves or customer data lives — and the test's job is to demonstrate *whether the path exists* (segmentation effectiveness) without touching production transactions.

### 10.3 Persistence Mechanisms

Establish (and then remove) mechanisms that survive reboots and re-authentication: scheduled tasks, services, registry/autorun entries, SSH keys, web shells, or valid-account manipulation. PTES includes persistence in post-exploitation because a realistic adversary establishes it — but every persistence mechanism must be fully removed at engagement end and listed in the report, so the client can verify the environment is exactly as it was found.

### 10.4 Data Exfiltration Proof

The crown-jewels access proof: demonstrate that the attacker *could* exfiltrate the sensitive data — without actually taking it. The professional pattern:

- Prove read access to the sensitive store (customer database, payment files, KYC documents) with evidence that specific records are accessible.
- Prove the *channel* exists (the data could reach the attacker's position — e.g., an internal API returns customer PII to the compromised app tier).
- **Do not extract real data.** Use markers or synthetic records where a proof needs a payload; never copy real customer data out of the environment. This is a hard rule in banking engagements and a contractual term in the RoE (§5.3 data handling).

### 10.5 Covering Tracks and Business-Impact Demonstration

- **Covering tracks** — PTES's stance is measured: an adversary *would* clean logs and remove artifacts, but the tester's priority is evidence preservation for the client. Minimal cleanup is performed (and disclosed); the client's SOC team is usually given the full IOC list so they can improve detection — many banking engagements deliberately leave the tester's artifacts in place so the SOC can practice detection.
- **Business impact demonstration** — the phase's showpiece: what an attacker could do with this access. For banks this means **ransomware simulation** (demonstrating write/encrypt capability on a critical application server using a harmless marker), **data-breach proof** (demonstrating the reachable customer-data set), or **fraud-path demonstration** (proving the payment-initiation path is reachable). The demonstration is always simulation — the impact is proven, not inflicted.

### 10.6 The Output: The Impact Assessment

Post-Exploitation closes with the impact assessment: the **depth of compromise** (from initial foothold to final position — "workstation → domain user → payment-domain server"), the **business systems at risk** (which crown jewels were reachable, with evidence), and the **evidence chain** (every step documented: technique, host, credentials used, timestamps — the complete attack path, which the report will present as one coherent narrative). This evidence chain is the most valuable artefact the engagement produces: it is what the executive summary (§11.1) and the remediation roadmap (§11.4) are built from.

---

## 11. Phase 7: Reporting

Reporting is the final phase and the deliverable — the entire engagement, compressed into documents that different audiences can act on. PTES's reporting guideline is unambiguous about the stakes: *the report is the deliverable — a great test with a poor report fails.* Testing that cannot be understood, verified, and acted on has no value to the client, and in a regulated environment, a poor report is a compliance failure (§14.4).

### 11.1 The Executive Summary

Written in business language for the audience that pays and decides: risk, impact, and the top findings. No exploit syntax, no CVE jargon — the executive summary says: "Your internet-banking portal can be compromised to the point of transferring funds from any customer account; the root causes are X, Y, and Z; the business risk is material; here is what we recommend doing in the next 90 days." It should stand alone (a board member should understand it without reading the technical report) and it should be *honest* — it states overall security posture, not just a findings count.

### 11.2 The Technical Report

The detailed findings section: for each finding, the **vulnerability** (what and where), **evidence** (screenshots, transcripts), **reproduction steps** (how to verify independently — this is what makes retesting meaningful), the **CVSS score** (base technical severity), and **remediation guidance** (concrete, actionable fixes, ideally with verification steps for each fix).

### 11.3 Risk Ratings: CVSS and Business Risk

| Dimension | What it measures | Who uses it |
|---|---|---|
| **CVSS base score** | Technical severity of the vulnerability in isolation (attack vector, complexity, impact on CIA) | Engineers, patch prioritization |
| **Business risk rating** | CVSS adjusted by the Phase 3 business-impact assessment: what the vulnerable system *does* for the bank | Executives, risk committee, regulators |

A common finding-level structure: a high-CVSS flaw in a low-criticality system may be rated medium business risk; a medium-CVSS flaw in the payment path may be rated high business risk. The report explains the delta — that reasoning is exactly what regulators and auditors look for (§14.4).

### 11.4 The Remediation Roadmap

Prioritized fixes: grouped by business risk (critical first), each with the owning team indicated and a suggested target timeframe (e.g., critical: 30 days; high: 90 days; medium: 180 days — cadences commonly mapped to banking change-management cycles). The roadmap turns findings into a project plan and gives the client's remediation tracker its backlog.

### 11.5 Retesting (Verification of Fixes)

A pentest is not complete when the report is delivered — PTES's reporting phase includes the **retest**: the agreed verification of fixes, typically 2–6 weeks after remediation, re-testing the specific findings (a focused re-test, not a full re-engagement). The retest result — verified-fixed, partially-fixed, or not-fixed — closes each finding. For banking, the retest is a control in itself: MAS-aligned security-testing programs expect remediation to be *validated*, and the retest report is the evidence (§14.2, and see the notice-era expectation that a single annual report without post-change coverage invites examiner questions).

### 11.6 The PTES Reporting Guideline: Required Elements

The PTES reporting guideline defines the report structure — the elements that make a report complete and comparable across vendors:

| Report element | Contents |
|---|---|
| **Executive summary** | Business-language risk overview, top findings, posture statement |
| **Scope & methodology** | What was tested (the Phase 1 scope), what was excluded, the phases followed, tools and versions used |
| **Findings** | Verified findings with evidence, reproduction steps, CVSS, business risk |
| **Risk assessment** | The risk-rating methodology and the finding-level ratings |
| **Remediation** | Prioritized fixes, owners, timeframes |
| **Appendix: evidence & tools** | Raw evidence, tool outputs, target inventory, engagement metadata (dates, testers) |

Good report hygiene, per PTES and professional practice: findings are deduplicated (one root cause = one finding), every finding is reproducible from the report alone, evidence files are attached and encrypted, and the report distinguishes verified-exploited findings from verified-but-not-exploited ones.

### 11.7 Sample Finding Write-up

What a professional finding entry looks like in the technical report — reproduced here as the template for the worked example in §16:

```
Finding F-03 — Account Takeover via MFA-Reset Brute Force + BOLA
Affected: https://online.examplebank.com/api/v2/accounts/{id} (PUT)
Severity: CVSS 9.1 (Critical) · Business risk: High (payment path)
Evidence: [burp-transcript-0317.log] · [screenshot-account-switch.png]

Reproduction:
  1. POST /api/v2/auth/reset {account: 1000234}  → token sent via SMS
  2. Brute-force the 6-digit token (no rate limit; ~10k requests in 4 min)
  3. PUT /api/v2/accounts/1000234 with the reset session — arbitrary ID accepted
  4. GET /api/v2/accounts/1000234/transactions — full history returned

Impact: full takeover of any customer account; unauthorized transfers possible.
Remediation: server-side authorization check on the account-ID parameter
(owner-only); rate-limit and lock out the reset flow; anomaly detection on
reset storms.
Verification (retest): repeat steps 2–3 with a second test account — expect 403.
```

---

## 12. PTES in the Modern Landscape (2026)

PTES is a 2012-era standard, yet in 2026 it remains the most-referenced penetration-testing execution framework in the industry — the "PTES 7 phases" language is standard boilerplate in RFPs and vendor SOWs (§14.5), and most commercial methodologies are PTES-shaped underneath. Its durability is structural: the phases describe *process*, and process outlives tools. What has changed since 2012 is what lives *inside* the phases — the modern additions below are all implemented as extensions of the seven-phase model, which is exactly how they should be read.

### 12.1 Why a 2012 Standard Is Still the Reference

The failure modes PTES was built to fix — inconsistent scoping, no common language, unverifiable findings — are still the failure modes of the market, and no successor standard has displaced it. OWASP WSTG covers only the web slice; NIST SP 800-115 is government-oriented; OSSTMM is methodology-heavy; ATT&CK is a technique catalogue, not an engagement process. PTES remains the only widely adopted *end-to-end engagement process* — so the market standardized on it. Expect to see "PTES-aligned" in any serious pentest RFP for the foreseeable future.

### 12.2 Cloud Pentesting

The shared responsibility model reshaped Phase 1 (scope) and Phase 4 (what is testable): the customer controls and can test the application layer, data, and configuration, while the provider's hypervisor/control-plane layer is off-limits. Cloud providers codify this:

- **AWS penetration testing policy** — AWS permits customers to test their own resources **without prior approval** for a defined list of services (EC2, RDS/Aurora, CloudFront, API Gateway, Lambda, Elastic Load Balancing, and Route 53 hosted zones — verify the current list), subject to standing prohibitions: no DNS zone walking, no DDoS simulation, no attacks on AWS infrastructure or other customers, no social engineering of AWS staff. Testing outside the approved list requires an AWS support case. This is the modern "third-party authorization" (§5.3) in practice: AWS's policy *is* the authorization for the provider layer.
- **Azure** — the Azure Penetration Testing Rules of Engagement require notification (a testing request form) before testing, with a similar spirit: you may test your own resources; the provider's shared infrastructure is excluded.
- **GCP** — Google's Cloud Penetration Testing policy permits customer testing of most resources without prior approval (verify current terms).

For a bank's cloud estates, the Phase 1 scope must now distinguish cloud-native assets (serverless functions, managed databases, container platforms) and carry the provider policy references into the RoE.

### 12.3 Containers, Kubernetes, APIs

- **Container/K8s testing** — image vulnerabilities, supply-chain integrity (see the [container certificates guide](container_certificates_guide.md) in this series), RBAC misconfigurations, exposed kubelets/APIs, insecure secrets handling, and cluster escape paths. K8s testing typically appears as a Phase 4 sub-domain with its own technique catalogue (e.g., the MITRE ATT&CK for Containers matrix).
- **API pentesting** — the modern application layer: broken object-level authorization (BOLA/IDOR on API resources), excessive data exposure, missing rate limiting, insecure pagination, JWT weaknesses, and schema/versioning attacks. API security is a first-class Phase 4 discipline — see the API security treatment in the [spec-driven development frameworks guide](spec_driven_development_frameworks_guide.md).

### 12.4 AI / LLM Security Testing

The newest Phase 4 sub-domain: security testing of AI/LLM systems. Key activities: **prompt injection testing** (direct and indirect injection, jailbreaks), **adversarial testing** (manipulating inputs to force wrong outputs), model-output handling, data-poisoning resistance, and excessive-agency checks on LLM-powered agents. These map naturally onto the PTES phases: intelligence gathering profiles the AI assets, threat modeling uses the OWASP Top 10 for LLM Applications as its scenario source, and exploitation proves e.g. a prompt-injection chain that exfiltrates context data. Deep treatments live in this series' [LLM development risks guide](llm_development_risks_security_guide.md) and [adversarial ML attacks guide](adversarial_ml_attacks_guide.md). For banks, LLM security testing is increasingly mandated by the same regulatory instruments that mandate pentests (§14.2) — regulators treat AI systems as part of the critical-systems estate.

### 12.5 MITRE ATT&CK Alignment: The Modern Execution Standard

ATT&CK (MITRE's Adversarial Tactics, Techniques, and Common Knowledge) has become the *technique language* of offensive security, and PTES engagements are now routinely run ATT&CK-aligned — the phases map to tactics cleanly:

| PTES phase | ATT&CK alignment |
|---|---|
| 2. Intelligence Gathering | Discovery + Reconnaissance (the ATT&CK Reconnaissance tactic) |
| 3. Threat Modeling | Technique selection: which tactics/techniques the assumed adversary would use |
| 5. Exploitation | Initial Access, Execution, Credential Access |
| 6. Post-Exploitation | Privilege Escalation, Defense Evasion, Credential Access, Lateral Movement, Collection, Exfiltration, Impact |
| 7. Reporting | The evidence chain *is* a documented technique chain — ATT&CK IDs in every finding |

The practical payoff: findings tagged with ATT&CK IDs are immediately actionable for defenders (they map to detection analytics), and the attack-path narrative in the report doubles as a detection-engineering brief. ATT&CK-driven execution — adversary emulation and breach-and-attack-simulation (BAS) — is discussed in §17.3.
## 13. Running a Pentest Engagement in Practice

### 13.1 The Engagement Lifecycle

The PTES phases map to a project lifecycle that repeats on every engagement, whether the test is a two-day web-app check or a six-week red team:

```
Scoping meeting → RoE agreement → Authorization → Recon → Testing
→ Debrief → Report → Remediation → Retest → Close
```

| Lifecycle step | PTES anchor | Deliverable |
|---|---|---|
| Scoping meeting | Phase 1 | Draft scope, target list, crown-jewels discussion |
| RoE agreement | Phase 1 | Signed RoE, authorization letter, NDA |
| Authorization | Phase 1 | Green light + environment access (credentials, VPN, staging access) |
| Recon | Phase 2 | Target profile |
| Testing | Phases 3–6 | Verified findings, evidence, attack-path narrative |
| Debrief | Phase 7 | Verbal readout of top findings (before the written report) |
| Report | Phase 7 | Executive summary + technical report |
| Remediation | post-engagement | Client fixes, tracked in the findings tracker |
| Retest | Phase 7 | Retest report closing each finding |
| Close | — | Report archive, evidence destruction per RoE, lessons learned |

### 13.2 Project Management

A pentest is a project and is run like one: **milestones** per phase with agreed dates (the report date is contractual, not aspirational), **status calls** on a fixed cadence (daily during active testing, weekly otherwise — in banking, daily 09:00 during production-window testing), and **issue tracking**. The operational heart is the **findings tracker**: a shared, live register of every finding with status (new → verified → exploited → reported → acknowledged → fixed → retested → closed), severity, owner, and evidence references. The findings tracker is the single source of truth during the engagement and the seed of the Phase 7 report; for banking clients it also becomes the remediation backlog that audit will later inspect.

### 13.3 Team Roles and Communication

- **Tester side**: the **lead tester** (owns methodology, quality, and the report; is the single voice to the client) and the **testers** (each owning a scope slice — network, web/API, mobile, or social engineering).
- **Client side**: the **liaison** (the engagement owner who can make decisions) and the **technical point of contact** (the person who can unblock — firewall changes, test-account provisioning, environment access).
- **Critical-finding escalation**: the rule that never varies — *any* finding that indicates immediate risk (active compromise, exploitable crown-jewel path, evidence of an external attacker already present) triggers an **immediate phone call** to the liaison, not a note in the daily status. The daily status covers progress and blockers; the critical-finding call is a separate, urgent channel defined in Phase 1 (§5.4).

### 13.4 The Pentest Types

Knowledge levels — how much the testers are told before testing:

| Type | Information given | Typical use |
|---|---|---|
| **Black box** | Nothing beyond the target name/domain | Realistic external-attacker view; slower, less deep per dollar |
| **White box** | Full knowledge: architecture, credentials, source code | Deepest coverage in limited time; the standard for regulated environments where coverage evidence matters |
| **Gray box** | Partial knowledge (e.g., valid low-privilege user, API docs) | The pragmatic middle: realistic attacker-with-inside-knowledge; the most common commercial choice |

Test positioning and target:

| Dimension | Variants |
|---|---|
| **Position** | External (from the internet), internal (from inside the network — often from a simulated compromised workstation) |
| **Target** | Network, web application, mobile, wireless, social engineering, physical, API, cloud, container/K8s — usually combined into one engagement |

**Pentest vs red team** — the distinction that keeps appearing in RFPs:

| | Pentest (PTES-style) | Red team |
|---|---|---|
| **Scope** | Known, documented, bounded by RoE | Broad; the whole organization is fair game within limits |
| **Objectives** | Find and verify vulnerabilities | Achieve defined goals (reach the crown jewels) by any means |
| **Evasiveness** | Noisy, non-evasive; detection is not a concern | Evasive; avoiding detection by the client's SOC is part of the exercise |
| **Duration** | Days to weeks | Weeks to months |
| **Output** | Vulnerability-focused report | Adversary-simulation report: what was achieved, how, and whether detection worked |
| **Relationship** | Complementary | Complementary — red teaming is usually layered on top of a pentest program (see §14.2: "red team for crown jewels") |

A red team runs PTES-shaped processes internally (its phases 1–7 discipline still applies), but its success criteria, evasiveness, and scope discipline are different — which is why the two are bought as separate services.

### 13.5 Common Engagement-Management Pitfalls

- **Scope creep by omission** — an ambiguous target list ("the network") interpreted differently by each side mid-engagement. Fixed by the written scope document plus a change-control rule: any new target requires a scope amendment, never a verbal okay.
- **Testing inside a business-critical window** — the blackout list exists but nobody checked the calendar. Fixed by calendar join and a pre-engagement confirmation call.
- **The silent SOC** — the client's SOC was not told about the test and incident response gets triggered against the testers. Fixed by a test-notification procedure in the RoE (§5.2).
- **Evidence lost to log rotation** — screenshots and transcripts captured late or never. Fixed by making evidence capture a running discipline from Phase 5 onwards.
- **Report-to-test drift** — the report describes findings the testers cannot reproduce from their own notes. Fixed by the findings tracker (§13.2) as the single source of truth.
- **Retest not budgeted** — remediation verified late or never. Fixed by pricing the retest into the SOW: it is a PTES deliverable (§11.5), not an option.

---

## 14. PTES in Banking and Regulated Industries

### 14.1 Why Banks Pentest

Banks pentest for three converging reasons: **regulation requires it** (the driver below), **fraud economics demand it** (a single payment-path flaw is a direct loss event), and **supervisory expectations treat it as table stakes** (examiners read pentest reports as primary evidence of security control effectiveness). In Singapore, the regulatory anchor is the MAS Technology Risk Management (TRM) guidelines; regionally and globally, parallel instruments (BNM RMiT, HKMA, DORA, CBEST, NYDFS) impose the same shape of obligation.

### 14.2 The Regulatory Drivers

**MAS TRM Guidelines** — the Technology Risk Management Guidelines, 6th edition (issued January 2023; verify against the current text, as paragraph numbering evolves between editions), are the primary Singapore instrument. The security-testing expectations, in the security/cybersecurity control sections, require financial institutions to:

- Conduct **regular security testing, including penetration testing, of critical systems** — internet-facing systems and critical internal systems are expected to be tested **at least annually**, by **independent qualified assessors** (the independence point rules out testing your own systems with your own staff in most interpretations).
- Test **new applications before production deployment** (security testing is embedded in the SDLC, aligning with the TRM's secure-development expectations).
- Re-test / re-assess **after significant or material changes** to critical systems (the post-change coverage point — a single annual report is not complete evidence if major changes occurred mid-year).
- Base the program on **risk**: higher-risk, higher-criticality systems get more frequent and deeper testing.

Related and newer: **MAS FSM-N06 (Cyber Hygiene)** — issued under the Financial Services and Markets Act 2022 regime (the cyber-hygiene obligations; NOT to be confused with **MAS Notice 658**, which is the *outsourcing/third-party risk* notice, issued December 2023 and effective 11 December 2024) — codifies minimum cyber-hygiene expectations for financial institutions, including periodic **security testing of critical systems by independent assessors** alongside patching, hardening, and monitoring obligations. In practice, MAS-regulated banks run an annual (or more frequent, for the largest institutions) independent penetration-testing program on critical systems, with the TRM/FSM-N06 evidence file assembled from the engagement packet (§5.6) and the final report.

The wider regional/global picture for a Singapore-based bank:

| Instrument | Jurisdiction | Key security-testing requirement (as published — verify current text) |
|---|---|---|
| MAS TRM Guidelines (6th ed., 2023) | Singapore | Periodic security testing incl. pentests of critical systems; annual for internet-facing/critical internal; independent assessors; post-change retesting |
| MAS FSM-N06 (Cyber Hygiene) | Singapore | Minimum cyber hygiene incl. periodic security testing of critical systems by independent assessors (Note: MAS Notice 658 is the outsourcing/third-party risk notice, effective 11 Dec 2024 — a common mix-up) |
| BNM RMiT (2019) | Malaysia | Annual independent penetration testing and vulnerability assessments of critical systems |
| DORA (EU, applied Jan 2025) | EU | Threat-led penetration testing (TLPT) every 3 years for significant entities (Art. 24–26); CTAs (Cyber Threat Intelligence) |
| CBEST (BoE) | UK | Intelligence-led penetration testing of systemic importance; CREST-administered |
| NYDFS 23 NYCRR 500 | New York | Annual penetration testing and vulnerability assessments (under the cybersecurity regulation) |
| PCI DSS v4.0, Req. 11.3 | Global (card) | Annual pentest + retest after significant changes (segmentation testing under 11.4) |

### 14.3 Banking Standard Practice

- **Annual/periodic pentests of critical systems** — the recurring baseline, run by external firms (independence), PTES-aligned, with the report and remediation evidence filed for MAS review.
- **Post-major-change retests** — any significant change to a critical system (a core-banking upgrade, a new internet-facing API, an infrastructure migration) triggers a focused retest or re-engagement, closing the loop the TRM expects.
- **Third-party / vendor pentest requirements** — vendor systems touching bank data are tested per the bank's third-party-risk program, either by the vendor's own independent tests (reviewed by the bank) or by the bank's testers under contract (§14.5).
- **Red team for crown jewels** — larger institutions run periodic red-team exercises against crown-jewel scenarios (payment rails, account-takeover chains), typically every 1–3 years on top of the annual pentest baseline — exactly the layered model §13.4 describes.

### 14.4 Banking Pentest Considerations

- **Production impact risk** — testing live systems means the RoE must include **change-window requirements** (testing may be restricted to approved change windows or off-peak hours), **test data requirements** (dedicated test accounts, synthetic records — never real customer accounts in exploitation), and **rollback/stop conditions**. Banks routinely require a pre-engagement walkthrough of the exact tests planned against production systems.
- **Data protection** — no real customer data in testing: exploitation and exfiltration proofs use markers and synthetic records (§10.4); any PII unavoidably captured is handled per the data-handling terms in the RoE and the bank's PDPA obligations (the data-protection reasoning parallels the LLM-guide treatment in this series — [LLM development risks guide](llm_development_risks_security_guide.md)).
- **Compliance evidence** — the pentest report is audit evidence: the scope document, RoE, authorization letter, report, and retest results form the security-testing evidence file that MAS, internal audit, and external auditors examine. The report must therefore be regulator-grade: methodology stated, scope defensible, findings reproducible, remediation tracked to closure. Cross-reference the compliance-systems treatment in the [financial risk & compliance systems guide](../banking/financial_risk_compliance_systems_guide.md).
- **Vendor pentests (third-party risk)** — per MAS's outsourcing/vendor-management expectations, vendor systems that process bank data are subject to security testing requirements: contracts specify testing rights (the bank may test, or may review the vendor's independent test reports), and critical vendors' test results feed the bank's own risk register.

### 14.5 PTES as the Vendor SOW Standard

In the procurement reality, PTES is the *contractual* standard: RFPs for pentest services specify "PTES-aligned methodology" as a qualification criterion, and vendor SOWs commit to the seven phases by name. The buyer-side checklist for a PTES-aligned vendor bid:

1. Does the proposal enumerate all seven phases, with effort allocated per phase (not just "testing days")?
2. Does it commit to the Phase 1 deliverables (RoE, scope doc, authorization letter) before testing starts?
3. Does the reporting section match the PTES report elements (§11.6)?
4. Is the retest included and priced?
5. Are the testers qualified (OSCP/CREST/GPEN — §17.4) and independent of the systems under test?

A vendor that cannot answer "yes" to all five is not delivering a PTES-aligned engagement — which, for a MAS-regulated bank, is a compliance gap in the security-testing evidence file before the first packet is sent.

### 14.6 The MAS Evidence Pack: A Checklist

For a Singapore-regulated bank, the security-testing evidence file that MAS examiners and internal audit review is assembled almost entirely from PTES deliverables:

1. Engagement contract + NDA (the vendor relationship, §14.5)
2. Scope document showing critical systems covered — internet-facing and critical internal (§14.2)
3. Signed RoE and authorization letter (independence and authority)
4. Tester qualifications (OSCP / CREST / GPEN — §17.4)
5. The final report: executive summary + technical findings + remediation roadmap
6. The findings tracker with remediation owners and dates
7. The retest report closing each finding
8. Post-change coverage: retests triggered by material changes during the year
9. Third-party / vendor test reports for critical outsourced systems

If any item is missing, the annual evidence file has a gap — exactly the kind of gap examiners probe. The PTES phase structure is the natural filing system: items 1–4 are Phase 1 outputs, items 5–6 are Phase 7 outputs, and item 8 is the Phase 7 retest discipline applied continuously through the year.

---

## 15. The Tools Landscape by Phase

The tooling per phase — the landscape any PTES-aligned engagement draws from (all commercial tools listed have free/community tiers or trials unless noted; verify licensing before use):

### 15.1 Reconnaissance (Phase 2)

| Tool | Purpose | Open source? |
|---|---|---|
| **Nmap** | Port scanning, service/version detection, scripted checks (NSE) — the network-recon workhorse | Yes (GPL) |
| **Masscan** | Extremely fast port scanning over large ranges | Yes |
| **Shodan / Censys** | Internet-wide exposure search: find the org's exposed services, banners, devices | No (commercial; free tiers) |
| **Amass** | Attack-surface enumeration: subdomains, DNS, certificate-transparency, API-integrated OSINT | Yes (OWASP) |
| **Recon-ng** | Modular OSINT framework with a large module library | Yes |
| **theHarvester** | Email/domain/subdomain harvesting from public sources | Yes |
| **Maltego** | Entity-relationship graph mining for OSINT correlation (CE edition free) | No (commercial; free CE) |

### 15.2 Vulnerability Analysis (Phase 4)

| Tool | Purpose | Open source? |
|---|---|---|
| **Nessus** | Commercial vulnerability scanner — the de-facto standard (Essentials tier free) | No |
| **OpenVAS** | Leading open-source vulnerability scanner (Greenbone) | Yes |
| **Qualys** | SaaS vulnerability management + scanning | No |
| **Nikto** | Web-server scanner (misconfigurations, outdated software) | Yes |
| **Burp Suite** | The web-app/API testing platform — proxy, scanner, repeater, extensions | Community free; Pro commercial |
| **nuclei** | Fast template-based vulnerability scanning (huge community template library) | Yes (ProjectDiscovery) |

### 15.3 Exploitation (Phase 5)

| Tool | Purpose | Open source? |
|---|---|---|
| **Metasploit Framework** | Exploitation framework: modules, payloads, post-modules (Pro adds features) | Framework: Yes |
| **Burp Suite Pro** | Web exploitation: manual attack tooling, scanner, extensions (SQLi/SSRF/XSS chains) | No |
| **SQLmap** | Automated SQL injection detection and exploitation | Yes |
| **impacket** | Python toolkit for Windows protocol attacks (SMB, Kerberos, WMI, PsExec-style) | Yes |
| **Custom code** | Bespoke exploits and PoCs for this engagement's findings | — |

### 15.4 Post-Exploitation (Phase 6)

| Tool | Purpose | Open source? |
|---|---|---|
| **Mimikatz** | Windows credential harvesting (LSASS, Kerberos tickets) — heavily detected; modern engagements use variants/alternatives | Yes (source) |
| **BloodHound** | Active Directory attack-path mapping (collector + graph analysis) — the AD lateral-movement planner | Yes (CE) |
| **Cobalt Strike** | Commercial C2/red-team platform — the red-team standard (beacon, staging, evasive C2) | No |
| **PowerSploit / Empire** | PowerShell post-exploitation frameworks (Empire is the maintained successor line) | Yes |
| **LinPEAS / WinPEAS** | Privilege-escalation enumeration scripts (PEASS-ng project) | Yes |

### 15.5 Reporting (Phase 7)

| Tool | Purpose | Open source? |
|---|---|---|
| **PwnDoc / PwnDoc-ng** | Pentest reporting platform: findings database, report generation, team collaboration | Yes |
| **Serpico** | Simplified pentest reporting generator (templates, evidence embedding) | Yes |
| **Custom pipelines** | Markdown/LaTeX report pipelines with screenshots and auto-CVSS — common in mature teams | — |

The tooling rule to internalize: *tools are phase accelerators, not phase substitutes*. A scanner does not do Phase 4 (verification is manual), a framework does not do Phase 5 (chaining is judgement), and a reporting tool does not do Phase 7 (the analysis is the deliverable).

---

## 16. Worked Example: A Bank Web-Application Pentest

A complete PTES-aligned engagement for a fictional bank ("Example Bank") internet-banking portal — the shape any reader can adapt.

### 16.1 Pre-engagement

- **Scope:** `online.examplebank.com` (internet-banking portal), the `/api/v2/*` REST APIs, the mobile-app backend endpoints; staging environment `staging.examplebank.com` for destructive checks.
- **RoE:** business hours 09:00–18:00 SGT, Mon–Fri; no DoS testing; max 100 req/s against the APIs; no exploitation touching production customer data; blackout window: month-end close (last 3 business days); social engineering out of scope.
- **Authorizations:** signed RoE + authorization letter from the CISO; vendor NDA; test accounts (2 customer accounts with synthetic data) provisioned by the client.
- **Deliverables:** contract, RoE, scope document, authorization letter, NDA — filed with vendor management.

### 16.2 Intelligence Gathering

Subdomain enumeration via certificate transparency and Amass → 14 hostnames, including a forgotten `dev-portal.examplebank.com`. Tech-stack fingerprinting: the portal runs a Java/Spring backend behind an F5 WAF, the APIs use OAuth2 + JWT, and the mobile backend exposes a GraphQL endpoint. Job postings confirm a Kafka-based payments platform. Attack surface: 3 web apps, 2 API families, 1 GraphQL endpoint, the WAF as a choke point.

### 16.3 Threat Modeling

Assets: customer accounts/balances (crown jewels), payment initiation, session store. Attack scenarios (STRIDE + ATT&CK mapping): (1) **account takeover** — phishing/credential stuffing → broken authentication → session hijack (ATT&CK: Initial Access + Credential Access); (2) **API abuse** — BOLA on `/api/v2/accounts/{id}` → read/modify other customers' data (Broken Object Level Authorization — ATT&CK: Collection); (3) **data exposure** — GraphQL introspection + excessive data exposure → mass PII retrieval (ATT&CK: Collection/Exfiltration). Priority: 1 > 2 > 3.

### 16.4 Vulnerability Analysis

Automated scanning (Burp scanner + nuclei templates) + manual WSTG pass on the portal and APIs. Key verified findings: (a) **BOLA on the transfer-API** — `PUT /api/v2/accounts/{id}` accepts any customer ID (manual verification confirmed with the test account); (b) **weak MFA reset flow** — the reset token is 6 digits with no rate limiting; (c) **GraphQL introspection enabled** in production with an overly broad schema; (d) missing `Cache-Control` on PII responses; (e) WAF bypass via HTTP request smuggling on a legacy endpoint. CVSS assigned per finding; business risk elevated for (a)/(b) because they sit on the payment path.

### 16.5 Exploitation

Scenario 1 (account takeover) proven as a chain: (b) brute-forced the MFA reset token for the test account → (a) changed the account's registered device → (c) enumerated the account's transaction history via GraphQL. Result: **full account takeover of any customer account** with no user interaction beyond the reset flow — proof captured with screenshots and Burp transcripts.

### 16.6 Post-Exploitation

From the taken-over account: demonstrated access to the internal customer-API gateway (the portal's backend calls an internal API reachable via SSRF on a PDF-generation feature — a fourth finding, (f) SSRF), proving lateral reach from the web tier into the internal services network. Exfiltration proof: the GraphQL endpoint returns full PII for the test account — demonstrated with the synthetic account only, zero real data extracted. Impact assessment: account takeover → unauthorized transfers possible; SSRF → internal network foothold; the crown jewels (customer accounts + payment initiation) are reachable from the internet-facing portal with a realistic attack chain.

### 16.7 Reporting

**Executive summary** (board language): "Example Bank's internet-banking portal can be fully compromised — an attacker can take over any customer account and initiate transfers, and can reach internal systems from the web tier. The root causes are an authorization flaw in the transfer API, a weak MFA-reset flow, and an exposed internal-API path. We recommend fixing these within 30–60 days; the fixes are listed in the roadmap."

**Technical report structure** (the PTES skeleton):

```
1. Executive Summary
   1.1 Engagement overview · 1.2 Top findings · 1.3 Posture statement
2. Scope & Methodology
   2.1 In-scope/out-of-scope · 2.2 Phases executed · 2.3 Tools & versions
3. Findings
   3.1 Account takeover via MFA-reset brute force + BOLA (CVSS 9.1, High business risk)
   3.2 BOLA on transfer API (CVSS 8.1, High)
   3.3 GraphQL introspection + excessive data exposure (CVSS 6.5, Medium)
   3.4 SSRF via PDF generator → internal API access (CVSS 8.6, High)
   3.5 Missing cache headers on PII (CVSS 3.1, Low)
   3.6 WAF bypass via request smuggling (CVSS 7.5, Medium)
   Each: description, evidence, reproduction steps, CVSS breakdown, remediation
4. Risk Assessment
   4.1 Methodology (CVSS + business impact) · 4.2 Risk register
5. Remediation Roadmap
   5.1 Critical (30 days): fix BOLA + MFA reset · 5.2 High (90 days): SSRF + WAF · …
6. Appendix
   6.1 Evidence archive · 6.2 Target inventory · 6.3 Engagement metadata
```

**Retest:** 4 weeks after remediation — all critical/high findings verified fixed; the BOLA fix regression-tested across the full API surface. Retest report closes every finding and is filed with the annual MAS evidence pack.

### 16.8 Engagement Timeline

A realistic three-week shape for this engagement (typical for a bank web-app/API test):

| Week | Days | Activity (PTES phase) |
|---|---|---|
| 0 | 2–3 | Scoping, RoE, authorization, environment provisioning (Phase 1) |
| 1 | 1–2 | Intelligence gathering and target profile (Phase 2) |
| 1 | 1 | Threat modeling and scenario sign-off with the client (Phase 3) |
| 1–2 | 4–5 | Scanning, manual testing, verification (Phase 4) |
| 2 | 2–3 | Exploitation of the priority scenarios (Phase 5) |
| 2–3 | 2 | Post-exploitation: lateral reach, impact assessment (Phase 6) |
| 3 | 1 | Debrief — verbal readout of top findings (Phase 7) |
| 3 | 2–3 | Report writing, QA, delivery (Phase 7) |
| +4–6 wks | 1–2 | Retest and close (Phase 7) |

Total ≈ 15–18 tester-days over three weeks plus the retest — a realistic commercial envelope for a PTES-aligned web-app engagement.

---

## 17. The Future: Where PTES Is Heading

### 17.1 Continuous Testing and Pentest-as-a-Service

The industry is moving from periodic point-in-time tests toward **continuous testing**: automated pentest platforms (e.g., Cobalt, HackerOne, Intruder, Detectify — commercial), which run scanner-class checks continuously against internet-facing assets and human pentests on demand. This is "pentest-as-a-service": subscription testing with a standing scope, recurring human validation, and a live findings portal instead of an annual report. PTES's phase discipline survives in this model — the automation covers the continuous Phase 4 baseline, while the human engagements still run Phases 1–7 — but the *cadence* of pre-engagement and reporting becomes ongoing rather than annual. Regulators are moving the same direction: "periodic" is increasingly read as "continuous for internet-facing systems."

### 17.2 The AI Era

- **AI-assisted pentesting** — LLM-assisted reconnaissance analysis, report drafting, exploit scaffolding, and (experimentally) autonomous agents that execute parts of Phases 2–4. Expect productivity gains first, full autonomy last — exploitation judgement remains human for now.
- **LLM security testing as a service line** — prompt-injection and adversarial testing of AI systems (OWASP Top 10 for LLM Applications) is becoming a standard Phase 4 sub-domain in its own right, driven by both risk and regulation (§12.4).
- **Adversarial ML** — attacks on the ML pipeline itself (data poisoning, model extraction, evasion) — see the [adversarial ML attacks guide](adversarial_ml_attacks_guide.md).
- **The "AI red team"** — the emerging discipline of adversarial testing of AI systems end-to-end: model behaviour, RAG data flows, agent permissions, and the human-AI boundary — often run under the same PTES-shaped engagement structure with LLM-specific scenarios in the threat model.

### 17.3 ATT&CK-Driven Testing and BAS

Adversary emulation is becoming the standard way to run Phases 5–6: instead of "test everything," emulate specific known adversary groups (from ATT&CK's software/group catalogues) against the crown-jewel scenarios from Phase 3. Alongside it, **breach and attack simulation (BAS)** tools (e.g., SafeBreach, AttackIQ, Picus — commercial) continuously replay ATT&CK techniques against production environments to verify detection and prevention controls. The synthesis: PTES phases provide the engagement structure, ATT&CK provides the technique language, and BAS provides the continuous control-validation loop between human engagements.

### 17.4 Standards Consolidation and Certifications

PTES has no formal certification body — the ecosystem consolidated around the practitioner certifications instead, all of which train PTES-shaped processes:

| Certification | Issuer | Focus |
|---|---|---|
| **OSCP** (Offensive Security) | OffSec | Hands-on exploitation + report writing — the entry credential for offensive roles; exam is a live pentest |
| **CREST CSTM / CRT** | CREST | Certified Security Tester / Registered Tester — scheme-based, UK/global; required for CBEST and many regulated engagements |
| **GPEN** (SANS) | GIAC | Penetration testing methodology + tools, exam-based |
| **OSEP / CRTP / PNPT** | OffSec / Altered Security / TCM | Advanced offensive tracks (evasion, AD attacks) for senior roles |

The consolidation trend: regulated buyers increasingly require *scheme-based* credentials (CREST, and the UK CHECK scheme) or equivalent documented PTES-aligned experience — the certification becomes the short-hand for "this vendor can execute the standard." PTES itself needs no update for this; it is the substrate the certifications teach.

### 17.5 The Verdict

PTES will not be replaced — it will be absorbed. Its seven phases are the industry's shared process vocabulary, and every evolution of the last decade (cloud, ATT&CK, BAS, AI testing, continuous platforms) has been expressed *in terms of* the phases rather than against them. For practitioners and buyers alike, the practical skill is not learning a new standard — it is executing the existing one with modern depth: PTES-shaped process, WSTG-level technique, ATT&CK-language evidence, and regulator-grade reporting.

---

## 18. Glossary

| Term | Definition |
|---|---|
| **PTES** | Penetration Testing Execution Standard — the 7-phase industry framework for planning, executing, and reporting penetration tests |
| **RoE** | Rules of Engagement — the pre-engagement document defining when, how, and under what constraints testing may occur |
| **OSINT** | Open-Source Intelligence — information collected from public sources (see the [OSINT guide](osint_guide.md)) |
| **CVSS** | Common Vulnerability Scoring System — the industry metric for technical vulnerability severity (v3.1/v4.0) |
| **CVE** | Common Vulnerabilities and Exposures — the public registry of named vulnerabilities, each with a CVE-ID |
| **Exploit** | Code or technique that takes advantage of a vulnerability to achieve unauthorized access/effect |
| **Privilege escalation** | Gaining higher or different privileges than initially obtained (horizontal = same level, other users; vertical = higher level) |
| **Lateral movement** | Moving from one compromised system to others across the network |
| **Persistence** | Mechanisms that maintain access across reboots/re-authentication |
| **Pivoting** | Routing traffic through a compromised host to reach otherwise unreachable systems |
| **Red team** | An adversary-simulation exercise: evasive, objective-driven, broader than a pentest |
| **Black box / white box / gray box** | Knowledge levels: no prior info / full info / partial info given to testers |
| **ATT&CK** | MITRE's Adversarial Tactics, Techniques, and Common Knowledge — the technique catalogue/language for adversary behaviour |
| **BAS** | Breach and Attack Simulation — tools that continuously replay attacker techniques to validate controls |
| **WSTG** | OWASP Web Security Testing Guide — the web-application testing checklist |
| **OSSTMM** | Open Source Security Testing Methodology Manual (ISECOM) — the methodology-heavy testing standard |
| **MITRE** | The US non-profit operator of ATT&CK, CVE (via CNA program), and other security knowledge bases |
| **Crown jewels** | The assets an organization cannot afford to lose — the priority targets of a test |
| **SOW** | Statement of Work — the contractual scope/services description in vendor engagements |
| **RFP** | Request for Proposal — the procurement document in which buyers specify (e.g.) PTES-aligned methodology |
| **MAS TRM** | Monetary Authority of Singapore Technology Risk Management Guidelines (6th ed., 2023) — Singapore's technology-risk rulebook |
| **Retest** | The post-remediation verification pass closing each finding |
| **WAF** | Web Application Firewall — an intermediary filtering web traffic; a choke point testers must understand |
| **BOLA / IDOR** | Broken Object Level Authorization / Insecure Direct Object Reference — accessing objects by ID without authorization checks |
| **SSRF** | Server-Side Request Forgery — forcing the server to make requests to internal resources |
| **C2** | Command and Control — the attacker's channel to compromised systems |
| **Red team vs pentest** | Evasive objective-driven adversary simulation vs scoped vulnerability verification (see §13.4) |
| **TLPT** | Threat-Led Penetration Testing — DORA's intelligence-driven testing regime (EU) |
| **CBEST** | The Bank of England's intelligence-led penetration-testing framework (UK) |
| **PDPA** | Personal Data Protection Act — Singapore's data-protection law governing PII handling in testing |
| **NVD** | National Vulnerability Database — NIST's CVE-enriched vulnerability reference database |

---

## 19. References and Further Reading

### Primary sources
- PTES official site — ptes.org (the 7-phase standard and the PTES Technical Guidelines)
- OWASP Web Security Testing Guide (WSTG) — owasp.org (v4.2 current stable; v5.0 in development — verify)
- NIST SP 800-115 — Technical Guide to Information Security Testing and Assessment (2008; revision draft — verify current status)
- ISECOM OSSTMM v3 — isecom.org
- MITRE ATT&CK — attack.mitre.org (enterprise, mobile, ICS, containers matrices)
- CVSS — first.org (v3.1/v4.0 specifications); NVD — nvd.nist.gov
- ExploitDB — exploit-db.com

### Regulatory
- MAS Technology Risk Management Guidelines, 6th Edition (January 2023) — mas.gov.sg (verify current edition/paragraphs)
- MAS FSM-N06 Cyber Hygiene (under the FSMA 2022 regime; MAS Notice 658 is the outsourcing notice, effective 11 Dec 2024 — not a cyber-hygiene notice)
- AWS Penetration Testing policy — aws.amazon.com/security/penetration-testing (approved-service list and prohibitions — verify current)
- Azure Penetration Testing Rules of Engagement / GCP Cloud Penetration Testing policy (verify current terms)
- DORA (EU) Art. 24–26 TLPT; Bank of England CBEST; BNM RMiT; NYDFS 23 NYCRR 500; PCI DSS v4.0 Req. 11

### Sibling guides in this repository
- [OSINT guide](osint_guide.md) — the reconnaissance tradecraft behind PTES Phase 2
- [OpenSCAP guide](openscap_guide.md) — automated configuration/compliance assessment (Phase 4 companion)
- [LLM development risks & security guide](llm_development_risks_security_guide.md) — LLM security testing (Phase 4 for AI systems)
- [Adversarial ML attacks guide](adversarial_ml_attacks_guide.md) — adversarial ML testing
- [Spec-driven development frameworks guide](spec_driven_development_frameworks_guide.md) — API security context
- [Container certificates guide](container_certificates_guide.md) — container/K8s supply-chain security
- [Vulnerability scanning guide](vuln_scanning_c_proc_cobol.md) — scanning in legacy (C/COBOL) estates
- [Financial risk & compliance systems guide](../banking/financial_risk_compliance_systems_guide.md) — compliance evidence context
- [Core banking systems guide](../banking/core_banking_systems_guide.md) — the systems pentests protect

---

> **Honesty footer:** All product, standard, and regulatory facts above are as of August 2026. PTES itself is a static standard (core 2009, technical guidelines 2012). Regulatory instruments (MAS TRM 6th edition paragraphs, MAS Notice 658/FSM-N06 references, AWS/Azure/GCP testing policies, DORA TLPT details) evolve — verify against the official current text before relying on them for compliance decisions. This guide is technical reference material, not legal advice.
