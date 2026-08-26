# Zero Trust Network Architecture (ZTNA)

**The Zero-Trust Discipline Deep-Dive — the Overview (the Never-Trust-Always-Verify Definition), the History (Kindervag 2010, Google BeyondCorp 2014), the NIST SP 800-207 Standard (August 2020 — the Seven Tenets), the Pillars (Identity, Device, Network, Application, Data — the CISA Model), the Architecture (SDP, Microsegmentation, the Identity-Centric Model), the Vendors (Zscaler, Cloudflare, Netskope, Palo Alto, Cisco, Okta), the Implementation (the CISA Zero Trust Maturity Model), the Banking Angle (the Remote-Access Transformation), a ZTNA Design Worked Example (a Cymbal Bank), and the One-Page Summary — the Final Word: the Trust Perimeter Is Gone**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Technology / Security — the DEDICATED zero-trust deep-dive of the security cluster. Where [security_by_design_guide.md](security_by_design_guide.md) covers the *secure-development discipline* (its §3.7 is the Zero-Trust tie-in this guide was commissioned from — the identity/workload/network/data pillars in NIST SP 800-207 language, with the note that "the standing priority #3 (Zero Trust for data platforms) is the natural next commission from this cluster"), [distributed_auth_guide.md](distributed_auth_guide.md) the *identity* angle (authentication/authorization — the identity pillar's engine), [api_governance_guide.md](api_governance_guide.md) the *API* angle, [threat_modeling_guide.md](threat_modeling_guide.md) the *threat-model* angle (STRIDE meets the ZTNA threat model), and [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) the *resilience* angle (a ZTNA policy engine is an availability dependency), this guide is the *discipline* itself: the ZTNA definition and its history, the NIST standard and its seven tenets, the five pillars, the architecture patterns (SDP, microsegmentation, identity-centric), the vendor landscape, the maturity-model implementation path, the banking transformation, and a full worked design. It is written to stand alone and to cross-ref the cluster heavily.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** the zero-trust canon — John Kindervag's Forrester coinage, *No More Chewy Centers: Introducing The Zero Trust Model of Information Security* (Forrester Research, **September 14, 2010**, ✅ verified this pass via the companion report's own reference list) and the architecture sequel *Build Security Into Your Network's DNA: The Zero Trust Network Architecture* (Forrester, **November 5, 2010**, ✅ extracted this pass); the Google BeyondCorp disclosure, Rory Ward & Betsy Beyer, *BeyondCorp: A New Approach to Enterprise Security*, USENIX *;login:* Vol. 39, No. 6, **December 2014**, pp. 6–11 (✅ verified via usenix.org and research.google); the NIST standard, **SP 800-207 *Zero Trust Architecture*, August 2020** (final 11 August 2020, 59 pp., Scott Rose, Oliver Borchert, Stu Mitchell, Sean Connelly — ✅ verified, full PDF extracted, the seven tenets quoted verbatim in §3); the CISA **Zero Trust Maturity Model** (v1.0 initial release August 2021, opened for public comment September 2021; **v2.0 April 2023**, aligned to OMB M-22-09 — ✅ verified via cisa.gov and the v2 PDF, pillars and cross-cutting capabilities confirmed in §4); the CSA **Software Defined Perimeter** (the *SDP Specification 1.0*, Cloud Security Alliance — April 2014 per NIST SP 800-207's own reference list, ✅); and the vendor landscape (Zscaler Private Access, Cloudflare Access, Netskope, Palo Alto Prisma Access, Cisco Duo/Secure Access, Okta — ✅-structural positioning, verified against vendor pages where possible; see §6). NOTE: this pass had **live web access** (self-hosted Firecrawl backend) — key claims verified on 2026-08-25 by direct page/PDF extraction (csrc.nist.gov, the NIST SP 800-207 PDF, cisa.gov and the ZTMM v2 PDF, usenix.org, research.google, the Forrester 2010 PDF, Wikipedia as secondary). Anything that could not be verified is flagged ⚠ honestly — notably the **Gartner ZTNA coinage** (Gartner's site is paywalled and the search backend repeatedly returned empty for it; the ZTNA *definition itself* is verified via Palo Alto Networks' cyberpedia, but the specific Gartner report/date attribution is ⚠-structural, see §2 and the ledger).
> **Last Updated:** August 2026
> **Companion guides (sibling, same folder — plain filenames):** [Security by Design](security_by_design_guide.md) (**THE commissioning sibling — §3.7 The Zero Trust Tie-In is the direct parent of this guide; §3.2–§3.6 map principle-by-principle onto the pillars — cross-ref §1, §4 and §9**), [Distributed Auth](distributed_auth_guide.md) (**THE identity companion — authentication/authorization, OAuth2/OIDC, JWT/mTLS, RBAC/ABAC — the engine of the identity pillar — cross-ref §4 and §9**), [API Governance](api_governance_guide.md) (the API-security angle — API gateways as policy enforcement points — cross-ref §5 and §9), [Threat Modeling](threat_modeling_guide.md) (the threat-model angle — STRIDE against a ZTNA estate — cross-ref §3 and §9), [Data Architect Skill Gaps](data_architect_skillgaps_guide.md) (**the standing-priority frame — this guide CLOSES priority #3 "Zero Trust for data platforms" — cross-ref §1 and §4**), [Zero-Downtime System Design](zero_downtime_system_design_guide.md) (the resilience angle — the ZTNA policy engine as a single point of failure — light cross-ref in §7 and §9), [Enterprise AI Gateway](enterprise_ai_gateway_guide.md) (the AI-gateway variant of the identity-aware proxy — light cross-ref in §5), [Threat Modeling](threat_modeling_guide.md), [Configuration Management Languages](configuration_management_languages_guide.md) (policy-as-code tooling — light)
> **Companion guides (banking/, prefix `../banking/`):** [Crédit Agricole Software Systems](../banking/credit_agricole_software_systems_guide.md) (the house systems context for §9's worked example), [Banks in Singapore](../banking/banks_in_singapore_guide.md) (the SG bank estate for the regional frame), [Financial Risk & Compliance Systems](../banking/financial_risk_compliance_systems_guide.md) (the operational-risk angle — a ZTNA failure is an op-risk event — cross-ref §8), [RegTech](../banking/regtech_guide.md) (the compliance-machinery angle — cross-ref §8), [Financial Infrastructure](../banking/financial_infrastructure_guide.md) (the legacy extranet/VPN estate — BT Radianz and the financial VPNs ZTNA replaces — cross-ref §8), and the bank software-systems series (HSBC, BNP Paribas, Deutsche Bank, UOB, OCBC, Bank of America…) for the remote-access precedents
> **Companion guides (management/, prefix `../management/`):** [Business Case Development](../management/business_case_development_guide.md) (the ZTNA-investment angle — the cost of a breach vs the cost of the transformation — light cross-ref in §7 and §10)

---

**How to use this guide:** Section 1 is the overview — what zero trust and ZTNA actually are, the never-trust-always-verify definition, and the overview table (aspect / description) that cross-refs security_by_design_guide.md §3.7. Section 2 is the history — Kindervag's 2010 Forrester coinage, Google's BeyondCorp (2014 disclosure), and the history table (year / event / notes). Section 3 is the NIST SP 800-207 standard — the August 2020 publication and the **seven tenets, quoted verbatim**, with the NIST table. Section 4 is the pillars — the CISA model's identity/device/network/application/data pillars, with the pillars table (pillar / focus / controls). Section 5 is the architecture — the software-defined perimeter (SDP), microsegmentation, the identity-centric model, and the architecture table. Section 6 is the vendors — Zscaler, Cloudflare Access, Netskope, Palo Alto, Cisco, Okta — and the vendor table. Section 7 is the implementation — the CISA Zero Trust Maturity Model (2021 draft → v2 2023), the OMB/EO mandate context, and the implementation table. Section 8 is the banking angle — the remote-access transformation in a Cymbal Bank estate, with the banking table. Section 9 is the worked example — a ZTNA design for a Cymbal Bank: the scenario, the identity-first design, and the lessons. Section 10 is the one-page summary — the final word is **the trust perimeter is gone**. The glossary, the claims-status ledger and the cross-references close the file. Cross-references follow the repository convention: sibling guides in `technology/` are plain filenames; guides in `banking/` and `management/` are prefixed `../banking/` and `../management/`; guides in the `ai_llm/` subdirectory are prefixed `ai_llm/`. **Integrity convention:** ✅ = verified this pass (live web) or verified in the cross-referenced guide's ledger; ⚠ = flagged/unverified; ⚠-structural = industry-standard practice widely documented but not attributable to a single primary source verified this pass. **The most important single idea of the whole guide:** zero trust is not a product you buy — it is a set of seven tenets (NIST SP 800-207) that you architect for, five pillars you mature (CISA), and a category of access products (ZTNA) that delivers the remote-access half of it. The network stops being the security boundary; the *decision* becomes the security boundary.

---

## Table of Contents

1. [The ZTNA Overview](#1-the-ztna-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Definition — Never Trust, Always Verify](#12-the-definition--never-trust-always-verify)
   - 1.3 [The Overview Table — Aspect / Description](#13-the-overview-table--aspect--description)
   - 1.4 [Reading the Overview Table](#14-reading-the-overview-table)
   - 1.5 [The Vocabulary — Zero Trust vs ZTA vs ZTNA](#15-the-vocabulary--zero-trust-vs-zta-vs-ztna)
   - 1.6 [The Discipline in the Bank — What a Cymbal Bank Estate Adds](#16-the-discipline-in-the-bank--what-a-cymbal-bank-estate-adds)
   - 1.7 [Why Now — the Forces Behind the Shift](#17-why-now--the-forces-behind-the-shift)
2. [The History](#2-the-history)
   - 2.1 [The Kindervag Coinage — Forrester, 2010](#21-the-kindervag-coinage--forrester-2010)
   - 2.2 [The Google BeyondCorp — 2014](#22-the-google-beyondcorp--2014)
   - 2.3 [From Coinage to Standard — 2010–2020](#23-from-coinage-to-standard--20102020)
   - 2.4 [The History Table — Year / Event / Notes](#24-the-history-table--year--event--notes)
3. [The NIST SP 800-207](#3-the-nist-sp-800-207)
   - 3.1 [The Standard — August 2020](#31-the-standard--august-2020)
   - 3.2 [The Seven Tenets — the Exact List](#32-the-seven-tenets--the-exact-list)
   - 3.3 [The Logical Components — PDP, PE, PA](#33-the-logical-components--pdp-pe-pa)
   - 3.4 [The Three Approaches](#34-the-three-approaches)
   - 3.5 [The NIST Table](#35-the-nist-table)
   - 3.6 [The Threats a ZTA Must Survive — NIST §5](#36-the-threats-a-zta-must-survive--nist-5)
   - 3.7 [The Deployment Use Cases — NIST §4](#37-the-deployment-use-cases--nist-4)
4. [The Pillars](#4-the-pillars)
   - 4.1 [The Five Pillars — the CISA Model](#41-the-five-pillars--the-cisa-model)
   - 4.2 [The Pillars Table — Pillar / Focus / Controls](#42-the-pillars-table--pillar--focus--controls)
   - 4.3 [The Cross-Cutting Capabilities](#43-the-cross-cutting-capabilities)
   - 4.4 [Pillars vs Tenets — the Two Lenses](#44-pillars-vs-tenets--the-two-lenses)
5. [The Architecture](#5-the-architecture)
   - 5.1 [The Software-Defined Perimeter (SDP)](#51-the-software-defined-perimeter-sdp)
   - 5.2 [Microsegmentation](#52-microsegmentation)
   - 5.3 [The Identity-Centric Model](#53-the-identity-centric-model)
   - 5.4 [The Architecture Table](#54-the-architecture-table)
6. [The Vendors](#6-the-vendors)
   - 6.1 [The Landscape — Who Sells What](#61-the-landscape--who-sells-what)
   - 6.2 [The Vendor Table](#62-the-vendor-table)
   - 6.3 [Selection Criteria — What to Buy, What to Build](#63-selection-criteria--what-to-buy-what-to-build)
7. [The Implementation](#7-the-implementation)
   - 7.1 [The CISA Zero Trust Maturity Model](#71-the-cisa-zero-trust-maturity-model)
   - 7.2 [The Mandate Context — EO 14028 and OMB M-22-09](#72-the-mandate-context--eo-14028-and-omb-m-22-09)
   - 7.3 [The Implementation Table](#73-the-implementation-table)
   - 7.4 [The Maturity Ramp — and the Common Pitfalls](#74-the-maturity-ramp--and-the-common-pitfalls)
   - 7.5 [A Sample Scorecard — Meridian Bank Year One](#75-a-sample-scorecard--meridian-bank-year-one)
8. [The Banking Angle](#8-the-banking-angle)
   - 8.1 [The Remote-Access Transformation](#81-the-remote-access-transformation)
   - 8.2 [The Regulatory Frame — MAS, DORA, SWIFT](#82-the-regulatory-frame--mas-dora-swift)
   - 8.3 [The Banking Table](#83-the-banking-table)
9. [The Worked Example — a ZTNA Design for a Cymbal Bank](#9-the-worked-example--a-ztna-design-for-a-cymbal-bank)
   - 9.1 [The Scenario](#91-the-scenario)
   - 9.2 [The Design — Identity-First](#92-the-design--identity-first)
   - 9.3 [The Phased Plan](#93-the-phased-plan)
   - 9.4 [The Threat-Model Cross-Check](#94-the-threat-model-cross-check)
   - 9.5 [The Lessons](#95-the-lessons)
10. [The One-Page Summary](#10-the-one-page-summary)
- [The Glossary](#the-glossary)
- [Claims Status and Verification Notes](#claims-status-and-verification-notes)
- [Cross-References and Further Reading](#cross-references-and-further-reading)

---

## 1. The ZTNA Overview

### 1.1 The Short Answer

**Zero trust is the 2020s operating model for security: "never trust, always verify" — no implicit trust based on network location, every request authenticated, authorized, and encrypted, access decisions made continuously with least privilege, and the blast radius contained by segmentation.** The canonical source is **NIST SP 800-207, *Zero Trust Architecture*, published August 2020** ✅, which defines zero trust as "the term for an evolving set of cybersecurity paradigms that move defenses from static, network-based perimeters to focus on users, assets, and resources" and states plainly that "zero trust assumes there is no implicit trust granted to assets or user accounts based solely on their physical or network location… or based on asset ownership (enterprise or personally owned)." **ZTNA — Zero Trust Network Access** — is the product category that delivers the access-control half of that model: instead of a VPN that puts a remote user *on the network*, a ZTNA service connects the user *to the application only*, with a default-deny posture, per-session decisions, and continuous verification. The one-line summary for the repo: **the network is no longer the security boundary — the access decision is** (this guide is the dedicated deep-dive that [data_architect_skillgaps_guide.md](data_architect_skillgaps_guide.md) priority #3 called for, and the direct descendant of [security_by_design_guide.md](security_by_design_guide.md) §3.7).

This guide sits inside the repo's security cluster as **the zero-trust discipline guide**. It is deliberately *not* a product comparison (that would be stale in a quarter) — it is the architecture deep-dive: where the idea came from (Forrester 2010, Google 2014), what the standard says (NIST SP 800-207, the seven tenets), what the pillars are (CISA's identity/device/network/application/data), how the architecture patterns work (SDP, microsegmentation, identity-centric), how you implement it in stages (the CISA maturity model), and what it means specifically for a bank (the remote-access transformation, and a full Cymbal Bank worked design in §9).

### 1.2 The Definition — Never Trust, Always Verify

The slogan **"never trust, always verify"** is the compressed statement of the whole discipline, and it is worth unpacking precisely because it is so often misread as "trust nobody." It means:

1. **No implicit trust.** Being inside the office LAN, on the corporate Wi-Fi, or behind the firewall grants nothing by itself (NIST tenet 2: "network location alone does not imply trust"). The old model — the "chewy center" of Kindervag's 2010 metaphor — trusted everything inside the perimeter once the perimeter was crossed; the zero-trust model treats the interior as already hostile.
2. **Verify everything, every time.** Authentication and authorization (of the subject *and* the device) are discrete functions performed before a session to a resource is established, and the verification is re-evaluated continuously, not once at login (NIST tenets 3 and 6). A session grant to one resource does not grant another.
3. **Least privilege, per request.** Access is granted with the minimum privileges needed for the task, determined by dynamic policy built from the observable state of the client identity, the application/service, the requesting asset, plus behavioral and environmental attributes (NIST tenet 4).
4. **Protect the resource, not the segment.** Zero trust "focuses on protecting resources (assets, services, workflows, network accounts), not network segments" — because in 2026 the resources live everywhere: cloud, SaaS, branch, home office, and the network location of any of them is no longer a trustworthy signal (NIST abstract, ✅).
5. **Assume breach.** The design assumption is that the attacker is already inside (or will be); therefore the network is segmented to contain lateral movement, everything is logged and inspected, and the enterprise collects as much state information as it can to keep improving the posture (NIST tenets 5 and 7).

The ZTNA product category operationalizes this for *remote and hybrid access*: a user authenticates to a ZTNA service (identity + device posture), the service brokers a connection to the specific application they are authorized for — over an encrypted tunnel that hides the application's IP address entirely (the SDP "dark cloud" concept, §5.1) — and the user never touches the network at all. There is no network-level access to misuse: no lateral movement, no scanning of internal ranges, no "once you're in, you're in." As the Palo Alto cyberpedia puts it (✅ extracted this pass): "ZTNA solutions default to deny, providing only the access to services the user has been explicitly granted" — versus VPNs, "which grant complete access to a LAN."

### 1.3 The Overview Table — Aspect / Description

| Aspect | Description |
|---|---|
| **The zero-trust model** | "Never trust, always verify" — no implicit trust based on network location or asset ownership; every request authenticated, authorized, and encrypted; access decisions continuous and least-privilege (NIST SP 800-207, Aug 2020 ✅) |
| **The ZTNA category** | Zero Trust Network Access — the product category for application-level secure access: default-deny, identity + device posture checks, per-session brokered connections, hidden applications (Gartner-coined term ⚠-structural, see §2.3) |
| **The history** | Forrester's Kindervag coins the Zero Trust Model (Sept 14, 2010 ✅); Google discloses BeyondCorp, the VPN-replacement architecture (Dec 2014 ✅); NIST standardizes (Aug 2020 ✅) |
| **The standard** | NIST SP 800-207 — the seven tenets (all data sources are resources; all communication secured regardless of location; per-session access; dynamic policy; monitor all assets; dynamic authN/authZ; collect and use state information) ✅ verbatim |
| **The pillars** | CISA's five pillars — Identity, Devices, Networks, Applications & Workloads, Data — plus three cross-cutting capabilities (Visibility & Analytics, Automation & Orchestration, Governance) ✅ (CISA ZTMM v2, April 2023) |
| **The architecture patterns** | SDP (software-defined perimeter — the dark cloud), microsegmentation (east-west containment), identity-centric access (BeyondCorp-style) ✅ (NIST §3.1, CSA SDP spec, Google) |
| **The vendors** | Zscaler ZPA, Cloudflare Access, Netskope, Palo Alto Prisma Access, Cisco (Duo / Secure Access / SDA), Okta (identity layer) — ✅-structural positioning |
| **The implementation** | CISA Zero Trust Maturity Model — Traditional → Initial → Advanced → Optimal across the five pillars; mandated for US federal agencies by EO 14028 + OMB M-22-09 (FY2024 goals) ✅ |
| **The banking angle** | The remote-access transformation: VPN/VDI estates → ZTNA; regulatory alignment (MAS TRM/N645, DORA, SWIFT CSP); remote workforce + cloud core-banking + third-party access (§8) |
| **The worked example** | A ZTNA design for a Cymbal Bank — identity-first: Entra/OIDC + Okta-style identity fabric, device posture, ZTNA for remote + third-party access, microsegmented data plane, phased maturity plan (§9) |
| **The security-by-design tie-in** | Every §3.2–§3.6 principle of [security_by_design_guide.md](security_by_design_guide.md) is a ZT pillar in miniature — least privilege = the access engine, fail-secure = deny-by-default, SoD = human-side verification (§3.7 there, cross-ref here) |
| **The threat-model tie-in** | STRIDE against a ZTNA estate ([threat_modeling_guide.md](threat_modeling_guide.md)): the policy engine is a spoofing/tampering target; the broker is a DoS dependency; stolen credentials are the insider-threat residual (NIST §5) |

### 1.4 Reading the Overview Table

The table is the guide in miniature: **history** (who said it and when) → **standard** (what the doctrine is) → **pillars** (what you must mature) → **architecture** (how it is built) → **vendors** (who sells what) → **implementation** (how you get there) → **banking** (what it means here) → **worked example** (how it lands in a Cymbal Bank estate). The two cross-ref rows anchor the guide in its cluster: the security-by-design row is the *commissioning* parent (this guide exists because [security_by_design_guide.md](security_by_design_guide.md) §3.7 named Zero Trust as the standing priority and the natural next commission), and the threat-model row is the *analytical* sibling — a ZTNA estate still has threats, and NIST SP 800-207's own §5 (subversion of the decision process, DoS, stolen credentials, visibility loss, data storage, proprietary lock-in, non-person entities) is a ready-made STRIDE input.

### 1.5 The Vocabulary — Zero Trust vs ZTA vs ZTNA

The three terms are frequently conflated; the distinction matters for architecture conversations:

- **Zero Trust (ZT)** — the *model*: the set of concepts and ideas (the seven tenets) that minimize uncertainty in enforcing accurate, least-privilege per-request access decisions in the face of a network viewed as compromised (NIST's operative definition, ✅).
- **Zero Trust Architecture (ZTA)** — the *enterprise plan*: an enterprise's cybersecurity plan that uses zero trust concepts and encompasses component relationships, workflow planning, and access policies (NIST, ✅). A ZTA is the *whole* estate: identities, devices, networks, applications, data — the five pillars.
- **Zero Trust Network Access (ZTNA)** — the *product category*: the access-control technology that implements the ZT model for remote/hybrid application access — the VPN replacement (Palo Alto cyberpedia, ✅ definition). ZTNA is a *component* of a ZTA, not the whole of it — and notably *not* the whole of it: a ZTA also needs identity governance, device management, data controls, and monitoring. Buying ZTNA alone and calling it "zero trust" is the single most common implementation error (§7.4).

A useful mnemonic: **ZT is the doctrine, ZTA is the architecture, ZTNA is the product.** NIST's three implementation approaches (§3.4) make the same point — enhanced identity governance, micro-segmentation, and software-defined perimeters are three complementary routes to the same tenets, and a full ZTA uses elements of all three.

### 1.6 The Discipline in the Bank — What a Cymbal Bank Estate Adds

For a Cymbal Bank-style bank (global markets, transaction banking, ~100k users across 50+ countries, a heavily regulated perimeter, and a legacy estate of on-prem apps plus a growing cloud footprint), the zero-trust discipline adds four things the generic enterprise does not have:

1. **Regulatory teeth.** The MAS Technology Risk Management notices and Notice 645 (cyber hygiene) push continuous-verification posture; EU DORA (Regulation (EU) 2022/2554, applicable 17 January 2025 — verified in the security_by_design ledger) demands operational resilience that a single-sign-on-then-trust-everything estate cannot demonstrate. Zero trust is how a bank *proves* least-privilege to an examiner rather than asserting it.
2. **The remote-access estate is huge and old.** Banks run some of the largest VPN/VDI estates in any industry — thousands of remote users, third-party vendor access (core-banking vendors, market-data providers), and a financial extranet layer (BT Radianz and the legacy financial VPNs — see [financial_infrastructure_guide.md](../banking/financial_infrastructure_guide.md)). ZTNA is the replacement path for the user-facing half of that estate.
3. **The data is the crown jewels.** A bank's toxic data (client positions, payment rails, trading secrets, PII) is exactly what Kindervag's "inside out" design says to protect first: identify the data, then build the network outward from it (§2.1). The data pillar is where banking zero trust differs most from a generic enterprise deployment.
4. **Third parties are first-class subjects.** Vendors, auditors, regulators, and market infrastructure need access to specific applications — and ZTNA's app-level brokering is the cleanest way to give a vendor access to *one application* without ever placing them on the bank's network. This is the "contracted services/non-employee access" use case NIST SP 800-207 §4.3 describes (✅).

The banking transformation itself is §8; the full design is §9.

### 1.7 Why Now — the Forces Behind the Shift

Zero trust is not a fashion; it is a response to five structural changes that broke the perimeter model, all of which NIST SP 800-207 itself names in its abstract (✅): "Zero trust is a response to enterprise network trends that include remote users, bring your own device (BYOD), and cloud-based assets that are not located within an enterprise-owned network boundary."

1. **The perimeter dissolved.** Applications moved to cloud and SaaS; users moved off the LAN (home, hotels, everywhere); the "inside" is now a mix of office, cloud, and personal devices — there is no single line a firewall can draw around the estate anymore.
2. **The credential became the attack surface.** The dominant breach pattern of the 2010s–2020s is identity-based: phish a credential, enter via VPN or cloud SSO, move laterally. Perimeter security is structurally blind to this — it authenticates the edge, not the request, and it grants network access, which is precisely what the attacker wants.
3. **The blast radius grew.** Flat networks + east-west traffic + compromised hosts = the whole estate from one foothold. The chewy-center problem Kindervag named in 2010 became the defining breach shape of the 2020s.
4. **The regulators moved.** From EO 14028/M-22-09 (US federal, §7.2) to DORA (EU, January 2025) to the MAS notices — the expectation of continuous verification, least privilege, and demonstrated containment is now written into law and supervisory practice (§8.2).
5. **The technology matured.** What was bespoke at Google in 2014 is now a product category: ZTNA brokers, identity fabrics, posture services, and segmentation fabrics are off-the-shelf (§6). The cost of the model dropped from "impossible" to "a budget line."

The "why now" answer for a bank is the intersection: the perimeter model is provably broken (2 and 3), the regulators will ask (4), and the technology now exists to do something about it (5) — while the window before the *next* breach (1 and 3) is the real deadline.

---

## 2. The History

### 2.1 The Kindervag Coinage — Forrester, 2010

**John Kindervag, then a principal analyst at Forrester Research, coined the Zero Trust Model in 2010.** The concept report is *No More Chewy Centers: Introducing The Zero Trust Model of Information Security*, Forrester Research, **September 14, 2010** ✅ (the date is verified this pass from the "Related Research Documents" list inside the November 2010 sequel — see below — which cites it exactly: "For details about the Zero Trust Model, see the September 14, 2010, 'No More Chewy Centers: Introducing The Zero Trust Model Of Information Security' report"). The metaphor is the chocolate egg with a hard shell and a soft center — the network firewall as the hard shell, and the entire interior ("trusted" by virtue of being inside) as the chewy center an attacker gets to once the shell is breached. The model's answer: eliminate the trusted interior entirely.

The sequel report — *Build Security Into Your Network's DNA: The Zero Trust Network Architecture*, Forrester, **November 5, 2010**, by Kindervag with Stephanie Balaouras and Lindsey Coit (✅ extracted this pass) — states the philosophy verbatim: *"Security professionals must stop trusting packets as if they were people. Instead, they must eliminate the idea of a trusted network (usually the internal network) and an untrusted network (external networks). In Zero Trust, all network traffic is untrusted. Thus, security professionals must verify and secure all resources, limit and strictly enforce access control, and inspect and log all network traffic."* The report's three core concepts: (1) no more trusted/untrusted interfaces on security devices; (2) no more trusted/untrusted networks; (3) no more trusted/untrusted users. And the design directive that still defines the discipline: **"design from the inside out"** — start with the toxic data and the resources to protect, then build the network outward from them ("protect the data first and figure out how to do the road-building — the networking — second").

Two provenance notes worth recording in the ledger: (a) the *term* "zero trust" itself predates Kindervag — Stephen Paul Marsh used it in his April 1994 doctoral thesis at the University of Stirling on the mathematical formalization of trust (⚠-secondary, per Wikipedia's cited sources; the *model* as a network-security doctrine is unambiguously Kindervag's 2010 coinage, ✅); (b) the "hard shell, soft center" critique of perimeter security has an even earlier lineage — the Jericho Forum's 2003 "de-perimeterisation" work and a 1994 Network World piece describing firewalls as a hard shell around a soft center (⚠-secondary; flagged in the ledger).

### 2.2 The Google BeyondCorp — 2014

**Google's BeyondCorp is the second pillar of the zero-trust canon: the first large-scale, public, working implementation of the identity-centric model.** The origin story is operational, not academic: after Operation Aurora (the 2009 Chinese APT attack that targeted Google and others), Google began building an access architecture that did not depend on a privileged intranet at all (⚠-secondary per Wikipedia's sourcing; the 2014 paper itself focuses on the architecture, not the trigger). The model: **"We are removing the requirement for a privileged intranet and moving our corporate applications to the Internet"** (✅ verbatim from the paper's abstract).

The canonical disclosure is **Rory Ward and Betsy Beyer, *BeyondCorp: A New Approach to Enterprise Security*, USENIX *;login:* Vol. 39, No. 6, December 2014, pp. 6–11** (✅ verified via usenix.org and research.google — the ;login: article, itself a distillation of the longer version presented at USENIX ATC 2014). The architecture's core components, as described in the paper and its successors: **access proxy** (the single entry point for all applications — no direct device-to-application paths), **identity-aware proxy** (access decisions from identity + device state, not network location), the **device inventory / device state** service (every device tracked, posture-evaluated, and only then admitted), and per-application authorization with **two-step verification** (MFA). Every request to every internal application is authenticated, authorized, and encrypted at the proxy — whether the user is in a Google office, at home, or on a hotel Wi-Fi. There is no VPN, and there is no "inside."

BeyondCorp matters to this guide for three reasons. First, it proved the model at Google scale — tens of thousands of employees, hundreds of applications — long before any vendor shipped a ZTNA product. Second, it defined the *identity-centric* architecture pattern (§5.3) that NIST SP 800-207 later formalized as "ZTA using enhanced identity governance," and that ZTNA products (Cloudflare Access being the closest commercial descendant of the pattern) implement. Third, it established the vocabulary the industry still uses: access proxy, device posture, identity-aware access, VPN replacement.

The architecture's components, as the paper and its successors describe them (⚠-structural for the component detail — the paper's abstract and framing are ✅ quoted above; the component breakdown is the standard reading of the ;login: article and the later BeyondCorp series): an **access proxy** as the single entry point for all internal applications; an **identity-aware proxy** tier that makes access decisions from identity and device state rather than network location; a **device inventory** that tracks and evaluates every device (managed or personal) before admission; **per-application authorization** so a grant to one app is not a grant to the estate; and **two-step verification** (MFA) embedded in the flow. Note on the citation: the December 2014 *;login:* article (Vol. 39, No. 6, pp. 6–11) is the canonical public disclosure and is what this guide cites (✅ verified); the same authors also presented the longer version at USENIX ATC 2014, and Google later productized the model as **BeyondCorp Enterprise** (⚠-structural — the product name and existence are widely documented, not re-verified this pass).

### 2.3 From Coinage to Standard — 2010–2020

The decade between Forrester's coinage and NIST's standard saw the idea move from analyst concept to government doctrine:

- **2013–2014 — the SDP.** The Cloud Security Alliance's Software Defined Perimeter working group produced the *SDP Specification 1.0* (**April 2014** per NIST SP 800-207's own reference list, ✅). SDP is the direct technical ancestor of ZTNA: a controller that authenticates subjects and issues per-connection authorizations to hosts, with the accepting host's ports hidden until authorized — the "dark cloud" (§5.1).
- **2014 — BeyondCorp disclosed** (December, ✅ above).
- **2018–2019 — the Gartner ZTNA term.** Gartner introduced **Zero Trust Network Access (ZTNA)** as the category name for this class of products — widely attributed to Gartner analyst notes beginning with an *Innovation Insight for Zero Trust Network Access* (2018) and the *Market Guide for Zero Trust Network Access* (2019). ⚠ **Flagged:** Gartner's site is paywalled and the search backend repeatedly returned empty for these specific documents, so the exact report title/dates could not be verified this pass; the *definition and category* are verified (Palo Alto cyberpedia ✅), and the Gartner attribution itself is ⚠-structural (universally repeated in industry literature, but no primary source confirmed this pass). See the ledger.
- **2019 — the UK NCSC** published zero-trust guidance recommending the approach for new IT deployments, particularly cloud-heavy ones (⚠-secondary, per Wikipedia).
- **2020 — the standard.** NIST publishes **SP 800-207, *Zero Trust Architecture* (August 2020, final 11 August)** ✅ — the seven tenets, the logical components, the deployment models, the threats. Zero trust went from "a Forrester concept" to "the doctrine," and the US federal government made it law-of-the-land within eighteen months (EO 14028, May 2021; OMB M-22-09, January 2022 — §7.2).
- **2021–2023 — the maturity model.** CISA publishes the Zero Trust Maturity Model (v1.0, August 2021, public comment September 2021; **v2.0, April 2023**) ✅ — the five pillars and the four maturity stages that give agencies (and banks) a roadmap (§4, §7).
- **2025 — the standards drift continues.** ETSI TC CYBER published TS 104 102, the **ZT-Kipling methodology** (September 2025) — a five-step iterative process (define the protected surface; map transaction flows; build the ZTA; create policy; monitor and maintain) that treats zero trust as a continual governance cycle rather than a one-time architecture (⚠-secondary, per Wikipedia's citation of the ETSI document). A useful recent flag for the horizon: the discipline is still actively standardizing, and the "protected surface first" step is Kindervag's "inside out" restated in governance language.

**Adoption** is the one claim this pass could not quantify: the search backend repeatedly returned empty for market-size and adoption-rate queries, so this guide makes **no adoption percentages or market forecasts** — flagged honestly in the ledger rather than padded with unverified numbers.

### 2.4 The History Table — Year / Event / Notes

| Year | Event | Notes |
|---|---|---|
| 1994 | Marsh's thesis uses the term "zero trust" (University of Stirling) | ⚠-secondary (Wikipedia-sourced); the term, not the security model — a provenance footnote |
| 2003 | The Jericho Forum names "de-perimeterisation" | ⚠-secondary; the perimeter critique predates the model |
| 2010 | **Kindervag coins the Zero Trust Model** — *No More Chewy Centers*, Forrester, **Sept 14, 2010** ✅ | The concept report; "stop trusting packets as if they were people"; the chewy-center metaphor |
| 2010 | *Build Security Into Your Network's DNA*, Forrester, **Nov 5, 2010** ✅ | The architecture sequel; "design from the inside out"; no trusted/untrusted networks or users |
| 2013–2014 | **CSA SDP Specification 1.0** (April 2014 per NIST's reference list ✅) | The dark-cloud access model; ZTNA's direct technical ancestor |
| 2014 | **Google BeyondCorp disclosed** — Ward & Beyer, USENIX *;login:* **Dec 2014** ✅ | The identity-centric VPN-replacement model at Google scale; access proxy + device inventory + MFA |
| 2018–2019 | **Gartner coins "ZTNA"** (Innovation Insight / Market Guide) | ⚠-structural — Gartner paywalled, unverified this pass; the category definition itself ✅ via Palo Alto |
| 2019 | UK NCSC zero-trust guidance | ⚠-secondary; cloud-first recommendation |
| 2020 | **NIST SP 800-207, August 2020** ✅ | The seven tenets; PDP/PE/PA components; three approaches; §5 threat list |
| 2021 | **EO 14028** (May 17, 2021) — federal agencies must plan ZTA | ✅ (via the CISA ZTMM v2 PDF); the mandate that made ZT doctrine |
| 2021 | **CISA ZTMM v1.0** — initial release August 2021, public comment September 2021 ✅ | The five pillars; Traditional→Initial→Advanced→Optimal |
| 2022 | **OMB M-22-09** (Jan 26, 2022) — federal ZTA strategy, FY2024 goals ✅ | Aligns the pillars with concrete agency deadlines |
| 2023 | **CISA ZTMM v2.0, April 2023** ✅ | Revised to align with M-22-09; the current roadmap document |
| 2025 | **ETSI TS 104 102, ZT-Kipling methodology** (September 2025) | ⚠-secondary; zero trust as a five-step iterative governance cycle |

---

## 3. The NIST SP 800-207

### 3.1 The Standard — August 2020

**NIST Special Publication 800-207, *Zero Trust Architecture*, was published in August 2020** (final release 11 August 2020; 59 pages; DOI 10.6028/NIST.SP.800-207) ✅ — verified this pass via csrc.nist.gov (the document history shows draft 23 September 2019 → second draft 13 February 2020 → final 11 August 2020) and via full extraction of the PDF itself. The authors: **Scott Rose and Oliver Borchert (NIST), Stu Mitchell (Stu2Labs), and Sean Connelly (DHS/CISA)** ✅. It is the definitive statement of what zero trust *is* — as the abstract puts it, "an evolving set of cybersecurity paradigms that move defenses from static, network-based perimeters to focus on users, assets, and resources," with "no implicit trust granted to assets or user accounts based solely on their physical or network location… or based on asset ownership."

Why it matters architecturally: SP 800-207 is deliberately **technology-agnostic** (the tenets "attempt to be technology agnostic" — ✅ verbatim). It does not mandate a product; it defines the properties an architecture must have, the logical components it must contain, the deployment variations, the threats it must survive, and the migration path from a perimeter-based network. That makes it the perfect *specification* for an architect: vendor claims can be checked against the seven tenets, and a design can be reviewed against §3–§5 the way code is reviewed against a standard. A companion, **SP 800-207A** (a roadmap to implementing ZTA), followed ✅ (referenced on the csrc page).

### 3.2 The Seven Tenets — the Exact List

Section 2.1 of SP 800-207 states the tenets verbatim (✅ verified this pass by direct extraction of the NIST PDF — the list below is quoted from the publication, lightly compressed only where the original elides):

1. **All data sources and computing services are considered resources.** A network may be composed of multiple classes of devices; personally owned devices may be classified as resources if they can access enterprise-owned resources.
2. **All communication is secured regardless of network location.** Network location alone does not imply trust. Access requests from inside enterprise-owned infrastructure must meet the same security requirements as requests from any other network. All communication should protect confidentiality and integrity and provide source authentication.
3. **Access to individual enterprise resources is granted on a per-session basis.** Trust in the requester is evaluated before access is granted, with the least privileges needed to complete the task. Authentication and authorization to one resource does not automatically grant access to a different resource.
4. **Access to resources is determined by dynamic policy** — including the observable state of client identity, application/service, and the requesting asset — and may include other behavioral and environmental attributes. Policy is the set of access rules an organization assigns to a subject, data asset, or application.
5. **The enterprise monitors and measures the integrity and security posture of all owned and associated assets.** No asset is inherently trusted; the enterprise evaluates the security posture of the asset when evaluating a resource request, and applies patches/fixes as needed.
6. **All resource authentication and authorization are dynamic and strictly enforced before access is allowed.** This is a constant cycle of obtaining access, scanning and assessing threats, adapting, and continually reevaluating trust in ongoing communication — with MFA for access to some or all enterprise resources.
7. **The enterprise collects as much information as possible about the current state of assets, network infrastructure and communications and uses it to improve its security posture.** Data about asset posture, network traffic, and access requests is processed and used to improve policy creation and enforcement.

These seven tenets are the *specification*. Every architecture decision in this guide traces back to one of them: ZTNA's per-session brokered connections implement tenets 3 and 6; device-posture checks implement tenet 5; the analytics layer implements tenet 7; and the whole model's "no implicit trust" is tenets 1 and 2. CISA's ZTMM v2 explicitly states it "reflects the seven tenets of zero trust as outlined in NIST SP 800-207" and reproduces the same list ✅ — so the tenet list is doubly verified.

### 3.3 The Logical Components — PDP, PE, PA

SP 800-207 §3 defines the three logical components of a ZTA (✅ — structure verified in the PDF's table of contents and body; the acronyms are the standard ones used throughout the publication):

- **Policy Engine (PE)** — the component that ultimately grants or denies access, using enterprise policy and input from external sources (CDM systems, threat intelligence, SIEM). The PE makes the decision.
- **Policy Administrator (PA)** — the component that generates and issues credentials/session tokens to the subject, and communicates with the PEP; the PA *executes* the PE's decision (it is the only component that talks to the PEP in the abstract model).
- **Policy Enforcement Point (PEP)** — the component that enables, monitors, and terminates connections between the subject and the resource. The PEP *enforces* the decision — and, critically, there is a PEP in front of **every** resource, not one big PEP at the edge.

The separation matters because it is the zero-trust version of "complete mediation" and "separation of duties" from [security_by_design_guide.md](security_by_design_guide.md) §3: the component that decides (PE) is not the component that executes (PA) is not the component that enforces (PEP). Subverting the decision process is NIST's own first-listed threat (§5.1, "Subversion of ZTA Decision Process" — ✅ from the TOC) — so the PE/PA/PEP split is what makes compromise of one component survivable. A ZTNA broker is a packaged PE+PA+PEP; a microsegmentation fabric is PEPs everywhere with a central PE; an identity-aware proxy is a PEP in front of each application with the PE in the identity layer (§5).

### 3.4 The Three Approaches

SP 800-207 §3.1 lists the three ways to build a ZTA (✅ verified in the PDF's TOC: §3.1.1–3.1.3, with the deployment variations in §3.2):

1. **ZTA using enhanced identity governance** — the BeyondCorp pattern: identity is the primary policy variable; every resource is fronted by an access proxy that enforces policy derived from identity, device state, and other attributes. This is the pattern ZTNA products implement.
2. **ZTA using micro-segmentation** — the network/data plane is carved into segments with granular policy between them; east-west traffic is controlled per-flow, not just north-south at the edge. This is the pattern of the software-defined networking vendors (§5.2).
3. **ZTA using network infrastructure and software defined perimeters** — overlay networks and SDP controllers hide resources entirely and broker authorized connections (the dark cloud; §5.1).

A full ZTA, NIST says, "will include elements of all three" (⚠-structural per Wikipedia's paraphrase of §3.1; consistent with the PDF's framing of the approaches as complementary). The deployment variations (§3.2): device agent/gateway-based, enclave-based, resource portal-based, and device application sandboxing — each a different physical shape for the same logical components. A bank's ZTNA rollout is typically device-agent/gateway for managed laptops, enclave-based for the data center, and resource-portal for third parties (§9).

### 3.5 The NIST Table

| NIST SP 800-207 element | What it is | Verified | The architectural consequence |
|---|---|---|---|
| **Publication** | Final August 2020 (11 Aug), 59 pp., DOI 10.6028/NIST.SP.800-207 | ✅ csrc.nist.gov + PDF | The canonical specification; technology-agnostic by design |
| **Definition of ZT** | Defenses move from static network perimeters to users, assets, resources; no implicit trust by location/ownership | ✅ PDF abstract | "The network is not the boundary" is doctrine, not marketing |
| **Tenet 1** | All data sources and computing services are resources | ✅ verbatim | IoT, BYOD, SaaS, actuators all get policy — nothing is "too small to protect" |
| **Tenet 2** | All communication secured regardless of network location | ✅ verbatim | Inside-the-LAN traffic is encrypted and authenticated like internet traffic |
| **Tenet 3** | Per-session access, least privilege, no transitive grants | ✅ verbatim | The session, not the login, is the unit of trust |
| **Tenet 4** | Dynamic policy from identity/app/asset/behavior/environment | ✅ verbatim | ABAC-style attribute policy (cross-ref [distributed_auth_guide.md](distributed_auth_guide.md)) |
| **Tenet 5** | Monitor and measure the posture of all owned/associated assets | ✅ verbatim | CDM-class tooling; device posture gates access |
| **Tenet 6** | Dynamic, strictly enforced authN/authZ before access; MFA | ✅ verbatim | Continuous re-verification, not once-at-login |
| **Tenet 7** | Collect state information and use it to improve posture | ✅ verbatim | Analytics and feedback loops are a first-class component |
| **Logical components** | Policy Engine / Policy Administrator / Policy Enforcement Point | ✅ (PDF structure) | Decide/execute/enforce separation; PEP per resource |
| **Three approaches** | Enhanced identity governance; micro-segmentation; network infra + SDP | ✅ (PDF §3.1) | ZTNA = approach 1 packaged; full ZTA needs elements of all three |
| **Threats (§5)** | Subversion of decision process; DoS; stolen credentials/insider; visibility; data storage; proprietary formats; non-person entities | ✅ (PDF TOC) | The built-in threat model — feed it into STRIDE (cross-ref [threat_modeling_guide.md](threat_modeling_guide.md)) |
| **Migration (§7)** | Identify actors; assess; plan incremental ZTA introduction into perimeter networks | ✅ (PDF TOC §7.3.1) | ZT is a transition, not a big-bang rip-and-replace |

### 3.6 The Threats a ZTA Must Survive — NIST §5

SP 800-207 §5 is the standard's own threat model — the list of what can go wrong *with the zero-trust architecture itself*, which is exactly the input a STRIDE review of a ZTNA design needs ([threat_modeling_guide.md](threat_modeling_guide.md) — cross-ref §9.4, where the list is applied to the worked example). The seven threats, from the PDF's table of contents (✅ verified; the descriptions are this guide's synthesis of each heading plus the standard's treatment):

1. **Subversion of the ZTA decision process** — the attacker targets the policy engine/administrator rather than the resources: compromise the decision point and every downstream grant is attacker-chosen. The architectural answer is the PE/PA/PEP separation (§3.3) plus privileged protection of the identity fabric itself.
2. **Denial-of-service or network disruption** — the ZTA's centralized decision components are a new choke point: take down the policy engine (or its network path) and access stops or — worse — fails open. The availability/security tension this creates is the [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) cross-ref, decided per application class (§9.2).
3. **Stolen credentials / insider threat** — the model's residual risk: no architecture fixes a phished password; MFA, risk-based authentication, and per-session scoping shrink what stolen credentials yield, but the credential is still the primary attack vector (NIST's own §5.3 framing).
4. **Visibility on the network** — ZTA hides resources (the SDP dark cloud) and encrypts traffic; the security team's own visibility can drop if the architecture does not export telemetry. Tenet 7's analytics layer is the countermeasure, and it must be designed in, not bolted on.
5. **Storage of system and network information** — the ZTA collects more state than a perimeter model (tenet 7); that data store becomes a target. The data pillar applies to the ZTA's own data: encrypt, segment, audit the policy/telemetry stores.
6. **Reliance on proprietary data formats or solutions** — vendor lock-in where the ZTA's policy, telemetry, or device state lives in a proprietary format the enterprise cannot export or audit. The §6.3 selection criterion ("the identity fabric, not the broker, owns policy") is the answer.
7. **Use of non-person entities (NPEs) in ZTA administration** — automated entities (scripts, agents, service accounts) administering the ZTA are subjects without human oversight; they need identities, least privilege, and audit like any other subject (the SPIFFE-class service-identity program of §9.2).

The lesson for the architect: a ZTA removes the *old* threats (perimeter breach = whole estate) by creating *new* ones (decision-process subversion, DoS choke point, telemetry loss). The threat list is why §9.4 exists as a formal cross-check rather than a paragraph of optimism.

### 3.7 The Deployment Use Cases — NIST §4

SP 800-207 §4 walks five deployment scenarios (✅ verified from the PDF's TOC); each maps directly onto a real estate shape a bank or enterprise owns:

1. **Enterprise with satellite facilities** — branches/offices that are not on the main campus network. In a ZTA, the satellite is just another set of subjects and devices; its network location grants nothing (tenet 2). The bank analogue: regional offices and the SG branch network ([banks_in_singapore_guide.md](../banking/banks_in_singapore_guide.md)) — the ZTNA broker serves them identically to HQ users.
2. **Multi-cloud / cloud-to-cloud enterprise** — workloads spread across several clouds with no enterprise-owned network between them; identity and policy must span the clouds (tenet 2 again — all communication secured regardless of location). The ZTNA broker and the identity fabric become the common control plane.
3. **Enterprise with contracted services and/or nonemployee access** — vendors and contractors need access without being "employees": brokered, scoped, time-boxed access to the specific applications the contract covers. This is the bank's third-party vendor estate (§8.1, §9.2's resource portal).
4. **Collaboration across enterprise boundaries** — two (or more) enterprises sharing workflows (a bank and a counterparty, a bank and its core-banking vendor) without merging networks; federated identity and mutually brokered access.
5. **Enterprise with public- or customer-facing services** — the ZTA's own §2 note applies: an enterprise "cannot impose internal policies on external actors (e.g., customers)" — customer-facing internet services stay outside the ZTA's policy scope, though they may use the same brokering infrastructure with separate policies.

The use cases matter because they kill the "zero trust is only for remote workers" misreading: the model covers branches, clouds, vendors, partners, and customers — five shapes, one policy engine.

---

## 4. The Pillars

### 4.1 The Five Pillars — the CISA Model

**The CISA Zero Trust Maturity Model organizes the discipline into five pillars: Identity, Devices, Networks, Applications and Workloads, and Data** ✅ — verified this pass directly from the CISA ZTMM v2.0 PDF (April 2023), which states: "The pillars… include Identity, Devices, Networks, Applications and Workloads, and Data. Each pillar includes general details regarding the following cross-cutting capabilities: Visibility and Analytics, Automation and Orchestration, and Governance." The same five-pillar structure is what the repo's [data_architect_skillgaps_guide.md](data_architect_skillgaps_guide.md) refers to when it names the priority this guide closes ("NIST SP 800-207 pillars (identity, device, network, application, data), micro-segmentation of the data plane, identity-aware proxies for data services, continuous verification"), and it is the lens [security_by_design_guide.md](security_by_design_guide.md) §3.7 uses to map its design principles onto zero trust ("the Zero Trust pillars (identity, device, network, application, data — per SP 800-207 and CISA's Zero Trust Maturity Model)"). Note the small but real naming difference: NIST SP 800-207 does not itself enumerate "five pillars" — that is CISA's operationalization of the seven tenets into a manageable programme structure; the *tenets* are the doctrine, the *pillars* are the delivery plan.

Why the pillars matter in practice: they are the **work-breakdown structure** of a zero-trust programme. A bank does not "do zero trust" as one project; it matures each pillar independently (CISA: "Each pillar can progress at its own pace and may progress more quickly than others until cross-pillar coordination is required" — ✅ verbatim from the v2 PDF). The Identity pillar gets the first investment (it is the engine — without strong identity, every other pillar's policy is built on sand); Devices and Networks come next; Applications and Data follow — and the three cross-cutting capabilities (visibility, automation, governance) are what glue them together at the higher maturity stages.

### 4.2 The Pillars Table — Pillar / Focus / Controls

| Pillar | Focus | Representative controls |
|---|---|---|
| **Identity** | Who/what is requesting access; the enterprise's authoritative view of a person or service (NIST tenet 4's "client identity") | Centralized identity fabric (Entra ID / Okta / Ping — cross-ref [distributed_auth_guide.md](distributed_auth_guide.md) for the mechanics: OIDC, SAML, JWT, mTLS); MFA/phishing-resistant MFA (FIDO2/WebAuthn, hardware keys); risk-based authentication; least-privilege RBAC/ABAC policy; service identities and workload identity (SPIFFE/SPIRE) for machine-to-machine |
| **Device** | The requesting asset's state and posture (NIST tenet 5 — "no asset is inherently trusted") | Device inventory + continuous compliance (MDM/UEM, CDM-class tooling); device posture checks (patch level, EDR presence, disk encryption, jailbreak/root detection) feeding access decisions; device trust scores; managed-vs-unmanaged device policy |
| **Network** | The network as transport, not trust boundary (tenets 2 and 7 — secure all communication; segment and observe) | Microsegmentation of the data plane (east-west policy per flow); encrypted communication everywhere (TLS/mTLS, IPsec where required); network visibility/analytics (netflow, DNS telemetry, packet capture for forensics); elimination of network-level implicit trust (no "VLAN = trusted") |
| **Application & Workload** | The applications and workloads that access data — each is a resource (tenets 1 and 3 — per-session, least-privilege access) | ZTNA brokers/identity-aware proxies in front of every app; application-layer authorization (fine-grained, per-transaction where needed); workload identity and mutual TLS; secure application development lifecycle (cross-ref [security_by_design_guide.md](security_by_design_guide.md) — the whole secure-SDLC stack); API security at the gateway (cross-ref [api_governance_guide.md](api_governance_guide.md)) |
| **Data** | The actual information assets — the "inside out" starting point (Kindervag's design directive, §2.1) | Data classification and inventory (the bank's data map); encryption at rest and in transit with KMS/HSM key hierarchies; tokenization/masking/pseudonymization; row/column-level security and dynamic masking; DLP; secure data sharing (delta sharing, clean rooms); audit logging of every data access (cross-ref the skill-gaps guide §1.7 for the full data-plane list) |
| **Cross-cutting: Visibility & Analytics** | Tenet 7 — collect state and use it | SIEM/SOAR pipelines; entity behavior analytics; access-log correlation; the feedback loop that improves policy |
| **Cross-cutting: Automation & Orchestration** | Tenets 5–6 at scale — posture checks and policy decisions that are too fast for humans | Policy-as-code (cross-ref [configuration_management_languages_guide.md](configuration_management_languages_guide.md)); automated incident response; orchestrated identity/device/policy updates |
| **Cross-cutting: Governance** | The human and process layer | Zero-trust strategy and architecture board; policy ownership; metrics and reporting to the board/regulator; the maturity-assessment cadence (this table's own §7.3) |

### 4.3 The Cross-Cutting Capabilities

The three cross-cutting capabilities deserve their own note because they are where most zero-trust programmes quietly stall. **Visibility and Analytics** is tenet 7 operationalized — you cannot enforce dynamic policy (tenet 4) or continuously verify (tenet 6) on data you cannot see; the analytics layer is what turns raw access logs into policy improvements. **Automation and Orchestration** is the scale answer: CISA's own model notes that as agencies transition toward optimal implementations, "associated solutions increasingly rely upon automated processes and systems that more fully integrate across pillars and more dynamically enforce policy decisions" (✅ verbatim). A bank with 100k users cannot manually review device-posture exceptions; the policy engine must consume posture feeds automatically. **Governance** is the least technical and most failure-prone: zero trust "may require a change in an organization's cybersecurity philosophy and culture" (✅ CISA v2, §3) — the governance capability is what forces the strategy, the metrics, and the board-level accountability that make the other seven pillars more than a shopping list. This maps directly onto the repo's management frame: a ZTNA investment without a governance story fails the business-case test ([business_case_development_guide.md](../management/business_case_development_guide.md) — cross-ref §7).

### 4.4 Pillars vs Tenets — the Two Lenses

A recurring confusion worth settling once: **the seven tenets are the "what" (the properties), the five pillars are the "where" (the estate).** Tenets are NIST's normative specification — an architecture either secures all communication regardless of location or it does not. Pillars are CISA's programme structure — the buckets an implementation team owns, budgets, and matures. They map onto each other loosely (identity pillar ≈ tenets 4 and 6; device pillar ≈ tenet 5; network pillar ≈ tenets 2 and 7; application pillar ≈ tenets 1 and 3; data pillar ≈ tenets 1 and 7), but the mapping is a management convenience, not a standard. In an interview or a design review, use tenets to *evaluate* (does this design satisfy tenet 3?) and pillars to *plan* (who owns the device pillar's backlog?). The worked example in §9 uses both: the design narrative is pillar-driven, the review checklist is tenet-driven.

---

## 5. The Architecture

### 5.1 The Software-Defined Perimeter (SDP)

**The software-defined perimeter is ZTNA's direct technical ancestor: an architecture where a controller authenticates subjects and authorizes connections, and the resources themselves are invisible — "dark" — until an authorized connection is brokered.** The canonical artifact is the Cloud Security Alliance's **SDP Specification 1.0** — **April 2014** per NIST SP 800-207's own reference list (✅ — the NIST PDF cites: "Software Defined Perimeter Working Group, 'SDP Specification 1.0,' Cloud Security Alliance, April 2014"); the concept lineage runs back to US Department of Defense work on hiding infrastructure from adversaries (the DISA "Black Cloud" program — ⚠-structural, widely documented but not directly verified this pass). NIST SP 800-207 itself lists SDP as one of the three ZTA approaches (§3.1.3, "ZTA Using Network Infrastructure and Software Defined Perimeters" — ✅).

How it works, in the CSA model: an **SDP controller** (control plane) authenticates the requesting host (the **initiating host, IH**) — typically with identity + device attestation — and, only on success, instructs the **accepting host (AH)** to open a connection to the authorized service, usually via an encrypted tunnel with the service's IP address never exposed to the client's network stack. Before authorization, the AH ignores all inbound traffic (often literally — no open ports, no response to scans: the "dark cloud"). The security properties are exactly what a bank wants for third-party and remote access: **no attack surface on the resource itself** (an attacker cannot scan or connect to what they cannot see), **no network-level access** (the client is never "on the network"), and **per-connection authorization** (each session is individually brokered — tenet 3).

ZTNA is the commercial, cloud-delivered evolution of SDP: the controller becomes the ZTNA service (Zscaler's ZPA, Palo Alto's Prisma Access, Cloudflare Access are all SDP-descended), the IH becomes the connector/agent on the user's device (or a browser-based client), and the AH becomes a connector inside the enterprise network that opens outbound-only connections to the app. The "dark cloud" phrase is used verbatim in Palo Alto's ZTNA explainer (✅ extracted: "Like software-defined perimeters (SDP), ZTNA leverages the concept of a dark cloud, preventing users from seeing any applications and services that they don't have permission to access"). The deployment-shape consequence for the bank: the ZTNA broker is the PEP in front of *every* remote-reachable application — one per app or one per enclave, per NIST's deployment variations (§3.4).

**A ZTNA session, end to end** (the walkthrough every architecture review should be able to recite — the components map to NIST's PE/PA/PEP from §3.3):

1. **The request.** The user opens the client (agent on a managed laptop, or a browser) and asks for *Application X* — never for "the network." The request terminates at the ZTNA broker, not at the application.
2. **The identity check.** The broker redirects to the identity fabric (OIDC/SAML): the user authenticates (password + FIDO2 MFA), and the fabric returns claims — who, which groups, which risk signals (tenet 4's attributes).
3. **The posture check.** The broker queries the device-posture service: is this device managed, patched, EDR-protected, un-jailbroken? (tenet 5.) A non-compliant device gets a reduced policy or a denial — before any connection exists.
4. **The policy decision.** The broker's policy engine evaluates identity claims + device posture + context (time, location, behavior) against the policy catalogue for *this application* — least privilege, per session (tenets 3 and 6). Deny is the default; grant is the exception that must be justified by policy.
5. **The brokered connection.** On grant, the broker instructs the in-network connector (the AH) to open an encrypted connection to the application — outbound-only from the bank's side. The application's address is never revealed to the client; the client is never on the network (the dark cloud).
6. **The session.** The user works inside the application only. There is no lateral movement possible — there is no network to move laterally *on*. The broker re-evaluates continuously (session timeout, new resource request, anomalous behavior → re-auth or terminate; tenet 6).
7. **The audit row.** Every decision, grant, denial, and session event streams to the SIEM (tenet 7) — the evidence trail that doubles as the regulator pack (§8.2).

The walkthrough is also the vendor-evaluation script (§6.3): a "ZTNA" product that skips any of steps 2–5 (no real posture input, no per-session policy, no hidden application) is a VPN in ZTNA clothing.

### 5.2 Microsegmentation

**Microsegmentation is the east-west half of zero trust: carving the data plane into fine-grained segments with per-flow policy, so that a compromised workload can talk only to what its function requires — the blast-radius containment that makes "assume breach" survivable.** NIST SP 800-207 treats it as one of the three implementation approaches (§3.1.2, "ZTA Using Micro-Segmentation" — ✅ from the PDF TOC), and CISA has published dedicated guidance (*Microsegmentation in Zero Trust Part One: Introduction and Planning*, July 2025 — ✅ seen on cisa.gov's related-resources list). The conceptual root is Kindervag's chewy-center critique: a network that trusts everything east-west hands the attacker the whole interior once one host is compromised; segmentation is what makes the interior "crunchy" instead of "chewy."

The mechanics vary by layer. **Network-layer microsegmentation** (the classic form — VMware NSX, Cisco ACI/SDA, open-source policy fabrics) applies per-application-group policy in the distributed virtual switch: the payments app can talk to the ledger DB on 5432, and literally nothing else — enforced in the data path, not by firewall rules that drift. **Identity/application-layer segmentation** (the ZTNA/identity-aware-proxy form) achieves the same containment at the access layer: the client is connected to the app, not the network, so east-west traversal is structurally impossible for remote users. The bank's data-center estate typically needs both: ZTNA for the user-access half (north-south and remote), microsegmentation for the workload half (east-west between tiers — web → app → DB → core-banking host). The data-platform angle is the repo's own standing priority: micro-segmenting the data plane (who can reach the warehouse, the lakehouse, the KMS) is exactly the "Zero Trust for data platforms" gap [data_architect_skillgaps_guide.md](data_architect_skillgaps_guide.md) names.

The honest engineering caveat: microsegmentation is the *hardest* pillar to land operationally. It requires an application dependency map (which workloads actually talk to which, on which ports), a change process for every new dependency, and monitoring that can prove policy is enforced — which is why the CISA maturity model puts full data-plane segmentation at Advanced/Optimal rather than Initial (§7), and why the sensible order is: ZTNA for remote access first (fast, contained, huge risk reduction), then segment the crown-jewel workloads (payments, settlement, the data platform), then the long tail.

### 5.3 The Identity-Centric Model

**The identity-centric model is the BeyondCorp pattern: access decisions are made on identity + device state, evaluated at an access proxy in front of every resource, with no privileged intranet at all** (✅ — the BeyondCorp paper's own abstract: "We are removing the requirement for a privileged intranet and moving our corporate applications to the Internet"). NIST formalizes it as "ZTA using enhanced identity governance" (§3.1.1 ✅); the vendor ecosystem sells it as ZTNA. The architecture in its canonical form:

- **One identity fabric.** A single authoritative source of identity for every subject (employee, contractor, vendor, service), federated to the apps (SAML/OIDC), with MFA as the baseline — the Identity pillar's engine, and the subject of the repo's [distributed_auth_guide.md](distributed_auth_guide.md) in full depth.
- **The access proxy in front of every resource.** No client ever connects directly to an application; every connection terminates at the proxy, which authenticates the subject, checks device posture, evaluates policy (tenet 4), and brokers an encrypted connection to the app — per session (tenet 3), re-evaluated continuously (tenet 6). The app's real address is hidden from the client (SDP dark cloud).
- **Device state as a first-class input.** The access proxy consults the device inventory/compliance service before every grant: a jailbroken phone or an unpatched laptop gets a different (usually empty) policy. This is tenet 5 made operational.
- **No location privilege.** Office, home, coffee shop, and (in the Google original) even untrusted networks get the *same* verification path. There is no "inside" to be on.

The identity-centric model is the *recommended* starting pattern for a bank's remote-access transformation (§8–§9) for a concrete reason: it attacks the largest and most exposed attack surface first (the VPN/VDI remote estate and third-party access), it delivers in months rather than years, and it does not require the application dependency map that microsegmentation demands before it can begin. Cloudflare Access is the closest commercial descendant of the Google pattern; Zscaler, Netskope, and Palo Alto deliver the same model in SASE form; Okta supplies the identity fabric underneath all of them (§6).

### 5.4 The Architecture Table

| Pattern | Core idea | Canon/verified | Delivers | Bank use |
|---|---|---|---|---|
| **SDP (software-defined perimeter)** | Controller brokers per-connection access; resources stay dark until authorized | CSA SDP Spec 1.0, April 2014 ✅ (per NIST's ref list); NIST §3.1.3 ✅ | Invisible attack surface; per-connection authorization; no network-level access | Third-party and remote access to specific apps — the vendor can't see or scan the bank |
| **Microsegmentation** | Per-flow east-west policy; workload talks only to what its function requires | NIST §3.1.2 ✅; CISA microsegmentation guidance (2025) ✅ | Blast-radius containment; assume-breach survivability | Data-center and data-platform segmentation — payments → ledger → core host |
| **Identity-centric model** | Access proxy per resource; decisions on identity + device; no privileged intranet | BeyondCorp, Dec 2014 ✅; NIST §3.1.1 ✅ | VPN replacement; uniform verification everywhere; device posture gating | The remote-access transformation (§8) — ZTNA as the proxy layer |
| **ZTNA (packaged)** | The product category: default-deny app access, identity + posture checks, brokered tunnels | Definition ✅ (Palo Alto cyberpedia); Gartner coinage ⚠-structural | Fast, contained remote-access modernization without a network rebuild | Phase-1 of the §9 design — replace VPN/VDI user access first |
| **Logical components (PE/PA/PEP)** | Decide / execute / enforce separation; PEP per resource | NIST §3 ✅ (PDF structure) | Survivable decision process; complete mediation | The review lens for any vendor architecture (§9.4) |
| **Deployment variations** | Device agent/gateway; enclave; resource portal; app sandboxing | NIST §3.2 ✅ (TOC) | Different physical shapes for the same logic | Agent on managed laptops; enclave in DCs; portal for vendors (§9.2) |

---

## 6. The Vendors

### 6.1 The Landscape — Who Sells What

The ZTNA vendor landscape is best read as **layers, not brands**: an identity layer (everyone's decisions depend on it), an access layer (the ZTNA brokers), and a network/data layer (SASE fabrics and segmentation). The task for an architect is not "which ZTNA vendor" but "which *combination* of layers," because the leading vendors increasingly sell the whole stack under the SASE banner. Positioning verified this pass is **✅-structural** (industry-standard positioning, cross-checked against vendor pages where extractable — Palo Alto's own cyberpedia page was extracted ✅ and confirms both the ZTNA category definition and Prisma Access's positioning); individual vendor *claims* (e.g., "Forrester New Wave leader") are flagged as vendor claims, not verified facts.

- **Zscaler** — the cloud-ZTNA pioneer: **Zscaler Private Access (ZPA)**, built on the Zscaler "Zero Trust Exchange" cloud, positions itself as the purest ZTNA play (app access brokered through the Zscaler cloud; no network access, ever). ⚠-structural positioning.
- **Cloudflare** — **Cloudflare Access**: the identity-aware proxy in front of any application (the BeyondCorp pattern as a service), part of the Cloudflare One SASE platform; strongest in the "put a proxy in front of every app, including non-HTTP via Cloudflare Tunnel" pattern. ⚠-structural positioning.
- **Netskope** — **Netskope One** (formerly Netskope Intelligent SSE / NewEdge): SASE with strong ZTNA plus a deep CASB/SWG heritage — the "secure the data inside the apps" angle, relevant to a bank's SaaS estate. ⚠-structural positioning.
- **Palo Alto Networks** — **Prisma Access**: ZTNA 1.0/2.0 within the Prisma SASE platform, riding the firewall/next-gen-firewall heritage; the vendor's own materials coin "ZTNA 2.0" (continuous trust, all-app support, zero-trust for data) and claim leadership in the Forrester New Wave: ZTNA (Q3 2021) — the latter is a **vendor claim**, flagged. Definition and positioning ✅ (their cyberpedia page extracted this pass).
- **Cisco** — the estate play: **Duo** (zero-trust access / MFA — the identity+device check), **Cisco Secure Access** (SSE/ZTNA), and **SDA (SD-Access)** for campus microsegmentation; strong where the bank already runs Cisco networking. ⚠-structural positioning.
- **Okta** — the identity layer, not a network vendor: **Okta Identity Cloud / Identity Engine** provides the identity fabric (SSO, MFA, lifecycle, policy) that the ZTNA brokers consume; the "Okta + ZTNA vendor" pairing is the standard reference architecture for the identity-centric model. ⚠-structural positioning.
- **The hyperscalers/clouds** — every major cloud ships ZTNA-flavored access (AWS Verified Access, Azure AD/Entra Application Proxy + Conditional Access, Google BeyondCorp Enterprise — Google productized its own architecture) ⚠-structural; for a bank already standardized on a cloud identity provider, these are often the natural first step rather than a new vendor.

A market-shape note, flagged honestly: the SASE/ZTNA market is consolidating fast, and *this guide deliberately makes no market-share, ranking, or "leader" claims* — those change quarterly and the search backend could not verify current figures this pass (see the ledger). What is stable is the layer model above.

### 6.2 The Vendor Table

| Vendor | Product(s) | Layer | Positioning (✅-structural unless noted) | Bank-fit note |
|---|---|---|---|---|
| **Zscaler** | ZPA (Zscaler Private Access), Zero Trust Exchange | Access (cloud ZTNA) | The cloud-brokered pure-ZTNA play — no network access ever | Strong for the remote-workforce transformation; cloud-first, no on-prem box |
| **Cloudflare** | Cloudflare Access, Cloudflare One, Tunnel | Access (identity-aware proxy) | The BeyondCorp pattern as a service; proxy in front of any app | Excellent for app-by-app rollout; the closest commercial BeyondCorp |
| **Netskope** | Netskope One (SSE/SASE, CASB, ZTNA) | Access + data | SASE with CASB/SWG heritage — data-aware ZTNA | Good where SaaS data protection matters (the Data pillar) |
| **Palo Alto Networks** | Prisma Access (ZTNA 1.0/2.0), Prisma SASE | Access + network | NGFW heritage; coined "ZTNA 2.0"; claims Forrester New Wave ZTNA leadership (Q3 2021 — **vendor claim**) | Natural where PAN firewalls already anchor the edge |
| **Cisco** | Duo, Cisco Secure Access, SDA | Identity + access + network | The estate play across identity (Duo), SSE (Secure Access), segmentation (SDA) | Natural where Cisco networking + Duo are already in place |
| **Okta** | Okta Identity Cloud / Identity Engine | Identity (fabric) | The identity layer under everything; not a network ZTNA | The identity pillar's engine — pairs with any ZTNA broker (§9) |
| **Hyperscalers** | Entra/AD Conditional Access + App Proxy, AWS Verified Access, BeyondCorp Enterprise | Identity + access | Cloud-native ZTNA-flavored access | Often the first step for a bank standardized on a cloud IdP |

### 6.3 Selection Criteria — What to Buy, What to Build

The selection framework this guide recommends (⚠-structural — standard architecture practice, not a verified vendor claim):

1. **Check the vendor against the tenets, not the marketing.** Can the product demonstrate per-session authorization (tenet 3), dynamic policy inputs from identity/device/behavior (tenet 4), continuous re-evaluation (tenet 6), and a feedback loop of telemetry (tenet 7)? A ZTNA product that only does SSO + a tunnel is a VPN with better branding.
2. **Decide the identity fabric first.** The ZTNA broker consumes identity; if the bank's identity layer is not yet a single fabric with MFA (cross-ref [distributed_auth_guide.md](distributed_auth_guide.md)), the ZTNA purchase will underdeliver. Okta/Entra + a broker is the standard pairing; the broker should integrate with the chosen fabric via OIDC/SAML, not impose its own identity silo.
3. **Match the layer to the estate.** Greenfield cloud estate → hyperscaler-native or Cloudflare-style proxy; heavy on-prem estate with firewalls → Palo Alto/Cisco; SaaS-heavy → Netskope-class SSE; all of them pair with the identity fabric.
4. **Run a pilot on one crown-jewel app.** The §9 worked example's phase 1: pick one high-value, high-risk application (payment initiation, settlement, or the data platform), put the ZTNA broker in front of it for a pilot cohort, and measure: time-to-access, support tickets, audit completeness, and the security delta (no more network-path exposure). The pilot's evidence is the business case ([business_case_development_guide.md](../management/business_case_development_guide.md) — the cost-of-breach vs cost-of-transformation frame).
5. **Beware the "one product = zero trust" fallacy.** ZTNA is the access pillar's tool, not the whole ZTA. The vendor table's identity/data layers and the cross-cutting capabilities (§4.3) still need ownership. A bank that buys ZTNA and stops is a bank that has modernized its remote access and *kept* its chewy center.

---

## 7. The Implementation

### 7.1 The CISA Zero Trust Maturity Model

**The CISA Zero Trust Maturity Model (ZTMM) is the roadmap: five pillars, three cross-cutting capabilities, and four maturity stages — Traditional → Initial → Advanced → Optimal — with concrete example controls at each stage.** ✅ Verified this pass: the CISA page (cisa.gov/resources-tools/resources/zero-trust-maturity-model) confirms "Version 1.0 of the ZTMM opened for public comment in September 2021," and "Version 2.0 incorporates alignment to OMB M-22-09, published in January 2022" (revision date April 11, 2023); the v2 PDF itself confirms the version history (v1.0 initial release **August 2021**, v2.0 **April 2023**) and the stage model ("The three stages of the ZTM journey that advance from a Traditional starting point to Initial, Advanced, and Optimal" — ✅ verbatim; the model's own text refers to the four stages including Traditional). The ZTMM is explicitly built on EO 14028's requirement that agencies develop ZTA plans, and it states that each pillar "can progress at its own pace" — the license for the phased, pillar-by-pillar implementation the bank in §9 needs.

Why the ZTMM matters beyond the US federal government: it is the *only* widely adopted zero-trust roadmap with concrete example controls per pillar per stage, which makes it the practical scoring tool for a bank's own posture assessment — and it is what the repo's skill-gaps guide references when framing "Zero Trust for data platforms" (MAS TRM / Notice 645 alignment is the Singapore mirror, ⚠-structural per the security_by_design ledger). A bank can score itself Traditional→Optimal on each pillar, prioritize the gaps, and report progress to the board in a regulator-friendly vocabulary.

### 7.2 The Mandate Context — EO 14028 and OMB M-22-09

The US federal mandates are worth understanding even outside the US, because they set the global expectation of what "implementing zero trust" means as a programme:

- **Executive Order 14028, "Improving the Nation's Cybersecurity"** — signed **May 17, 2021** (86 Fed. Reg. 26633) ✅ (cited in the CISA ZTMM v2 PDF), §3(b)(ii) requires federal agencies to develop plans to implement a Zero Trust Architecture.
- **OMB Memorandum M-22-09, "Moving the U.S. Government Toward Zero Trust Cybersecurity Principles"** — **January 26, 2022** ✅ (cited in the CISA ZTMM v2 PDF and the whitehouse.gov PDF), sets the federal ZTA strategy and requires agencies to meet specific security goals **by the end of Fiscal Year 2024**, aligned to the ZTMM pillars.
- The wider ecosystem followed: the **DoD Zero Trust Reference Architecture v2.0 (July 2022)** and the **NSA "Embracing a Zero Trust Security Model" v1.0 (February 2021)** ✅ (both cited in the CISA ZTMM v2 PDF's reference list) — defense and intelligence interpretations of the same doctrine.

For a bank, the takeaway is not the US deadlines; it is the *programme shape*: a mandated strategy document, pillar-aligned objectives with deadlines, a maturity assessment, and reporting. That is exactly the shape regulators elsewhere (MAS, EBA/DORA, HKMA) are converging on — the bank that has done a CISA-style self-assessment has the evidence pack ready for the next exam.

### 7.3 The Implementation Table

| Stage | What it means (per CISA ZTMM v2 ✅) | Identity | Device | Network | App/Workload | Data |
|---|---|---|---|---|---|---|
| **Traditional** | The starting point: perimeter-based, implicit trust | Manual provisioning; password-only; no risk signals | Basic inventory; little posture enforcement | Flat or coarse zones; firewall at edge only | VPN-based access; app-level authz rare | Data scattered; encryption partial; access by network zone |
| **Initial** | First ZT capabilities adopted | SSO + MFA for some apps; centralized directory | Device inventory + compliance baseline; MDM for managed fleet | Some segmentation; TLS everywhere; logging enabled | First ZTNA broker for remote access; per-app policy emerging | Classification started; encryption at rest/in transit for crown jewels; first access logging |
| **Advanced** | ZT enforced broadly, integrated with automation | Risk-based auth; ABAC attributes; service identities | Posture checks feed access decisions; automated response to non-compliant devices | Microsegmentation for crown-jewel workloads; east-west policy enforced | ZTNA default for all remote/third-party access; API security enforced (cross-ref [api_governance_guide.md](api_governance_guide.md)) | RLS/column-level security; tokenization; data-access analytics |
| **Optimal** | Fully automated, continuously verified, cross-pillar integrated | Continuous verification; identity analytics; decentralized/self-sovereign patterns where appropriate | Fully automated posture-to-policy loop; zero standing device trust | Dynamic, adaptive segmentation; encrypted everything; full visibility | All access brokered; workload identity everywhere (SPIFFE); policy-as-code | Data-centric security: encryption + masking + audit on every access; automated policy from classification |

The table is compressed from the ZTMM's per-pillar example tables (✅ the v2 PDF contains per-pillar tables — "Table 2: Identity Pillar… Table 6: Data" and "Table 7: Cross-Cutting Capabilities" — verified from the PDF's table of contents); the row content is this guide's synthesis of the model's example controls, flagged ⚠-structural as a synthesis rather than a quote.

### 7.4 The Maturity Ramp — and the Common Pitfalls

The implementation path this guide recommends (synthesis of the ZTMM's incrementalism + NIST §7's migration guidance + the repo's own security cluster):

1. **Assess and score.** Run a pillar-by-pillar Traditional→Optimal self-assessment (or use the CISA-style scorecard); publish it. The assessment *is* the first governance artifact (§4.3).
2. **Fix the identity pillar first.** Single fabric, MFA everywhere, lifecycle automation. Nothing downstream works without it.
3. **ZTNA the remote-access estate.** Replace VPN/VDI user access app-by-app with a ZTNA broker (the §9 worked example's phase 1); third-party access next (NIST §4.3's non-employee use case).
4. **Segment the crown jewels.** Microsegment the data center and the data platform east-west; start with payments/settlement/ledger flows.
5. **Automate and close the loop.** Posture-to-policy automation, policy-as-code, analytics feeding policy — the cross-cutting capabilities.
6. **Re-score annually.** The maturity model is a cadence, not a destination.

The pitfalls (each with its repo cross-ref):

- **Buying ZTNA and declaring victory** — ZTNA is one pillar's tool; the data pillar and the cross-cutting capabilities still need owners (this is precisely the "Zero Trust for data platforms" gap the skill-gaps guide flags — a ZTNA purchase does not close it).
- **Treating identity as an afterthought** — a broker without a strong identity fabric is a fancier VPN; the identity pillar is [distributed_auth_guide.md](distributed_auth_guide.md)'s entire subject.
- **Skipping the dependency map before segmentation** — microsegmentation without an application dependency map produces outages, not security; hence its Advanced/Optimal placement.
- **Making the policy engine a single point of failure** — the PE/PDP availability question is a resilience question: [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) (the fail-secure/fail-open tension — a ZTNA broker that fails open on a timeout is a security control that just turned itself off; one that fails closed is an availability incident; the design must decide per application, with recorded exceptions per [security_by_design_guide.md](security_by_design_guide.md) §3.4).
- **Ignoring the threat model** — NIST §5's threat list (subversion of the decision process, DoS, stolen credentials, visibility loss, proprietary lock-in, non-person entities) is a ready-made STRIDE input ([threat_modeling_guide.md](threat_modeling_guide.md)); the §9.4 cross-check shows how.
- **No business case** — the transformation is multi-year and competes for budget against revenue work; the [business_case_development_guide.md](../management/business_case_development_guide.md) frame (cost of breach vs cost of transformation, pilot evidence) is what keeps the programme alive past year one.

### 7.5 A Sample Scorecard — Meridian Bank Year One

To make the maturity model concrete (and to preview §9's worked example), here is the year-one scorecard shape for the Meridian Bank scenario — the CISA-style Traditional→Optimal assessment the programme's phase 0 produces, with the year-one targets and the gaps that drive the phase plan:

| Pillar | Year-0 score (as assessed) | Year-1 target | The gap that drives the phase plan |
|---|---|---|---|
| **Identity** | Initial | Advanced | No single fabric; MFA only partial; no service identities → phase 1's entire scope |
| **Devices** | Initial | Initial→Advanced | MDM exists but posture does not yet feed access decisions → phase 2's device-posture loop |
| **Networks** | Traditional | Initial | Flat data center zones; edge firewall only → phase 3's segmentation |
| **Applications & Workloads** | Traditional | Initial→Advanced | VPN-based access; no per-app brokering → phase 2's ZTNA rollout |
| **Data** | Traditional | Initial | Data map started; no RLS/tokenization on the platform → phase 3's data-plane controls |
| **Cross-cutting** | Traditional | Initial | SIEM exists, no behavior analytics; policy changes are manual → phase 4's automation |

The scorecard is the governance artifact (§4.3): it is published, it drives the phase plan (§9.3), and it is re-scored annually. Three practical notes: (a) score honestly — the scorecard's only value is as a gap-finder, and inflating it just moves the failure later; (b) score per pillar, not "overall" — the model's whole point is that pillars advance at different speeds (✅ CISA: "Each pillar can progress at its own pace"); (c) pair each score with the *evidence* that justifies it (a control inventory, an access-review report), because the scorecard doubles as the regulator evidence pack (§8.2).

---

## 8. The Banking Angle

### 8.1 The Remote-Access Transformation

**In banking, the zero-trust transformation is above all a remote-access transformation — and the remote-access estate is one of the largest and oldest attack surfaces a bank owns.** The legacy shape is familiar to anyone who has worked in a Cymbal Bank institution: a VPN concentrator estate (often two or three generations of vendor hardware) that terminates thousands of concurrent tunnels; a VDI/remote-desktop tier that gives users a full desktop inside the network; third-party vendors (core-banking vendors, market-data providers, auditors) with their own VPN accounts or extranet paths; and a financial-extranet layer (BT Radianz and the legacy financial VPNs — see [financial_infrastructure_guide.md](../banking/financial_infrastructure_guide.md)) connecting the bank to venues, market data, and counterparties. Every one of those paths grants *network-level* access: once the tunnel is up, the user (or the attacker using stolen credentials) is "inside," with the whole interior as the chewy center. The industry's high-profile bank breaches of the 2010s–2020s repeatedly traced back to exactly this shape: a VPN account, a phished credential, a lateral move.

The ZTNA transformation replaces that shape application-by-application: **the remote user (or vendor) authenticates to the identity fabric, passes device-posture checks, and is brokered a connection to the one application they are authorized for — nothing else.** The bank's applications become dark to everyone who is not authorized; the vendor who maintains the payment engine can reach the payment engine and cannot even see the trading floor; the auditor who reviews the AML system reaches the AML system only. The estate-level consequences, in order of impact:

1. **The credential-to-network pivot dies.** Stolen credentials now yield only the applications the identity was authorized for, under the same MFA/device checks as a legitimate session — the lateral-move economics collapse (NIST tenet 3: no transitive grants).
2. **Third-party access is finally controllable.** Each vendor gets an identity, a device-posture policy, an application scope, and a time-boxed entitlement — instead of "a VPN account" (NIST §4.3's non-employee use case).
3. **The VDI tier can shrink.** Many VDI workloads exist only to give remote users a safe desktop *inside* the network; ZTNA gives them the application without the network, so VDI retreats to the genuinely thin-client use cases (legacy apps that cannot be brokered, data-heavy desktops).
4. **The perimeter estate simplifies.** VPN concentrators, remote-access firewalls, and the DMZ rules they drag behind them shrink toward zero — which is also a resilience win (fewer boxes to patch, fewer fail-open paths) and a cost win (the [business_case_development_guide.md](../management/business_case_development_guide.md) frame: recurring VPN/VDI licensing vs a ZTNA service).
5. **The data pillar becomes the differentiator.** A bank's toxic data (client positions, payment rails, PII) is the "inside out" starting point of the whole model (§2.1): ZTNA gets the *access* right, and the data-plane controls (encryption, tokenization, row/column-level security, audit) make the *data* survivable even if an app is compromised — the "Zero Trust for data platforms" priority from [data_architect_skillgaps_guide.md](data_architect_skillgaps_guide.md), which a ZTNA rollout alone does not close.

### 8.2 The Regulatory Frame — MAS, DORA, SWIFT

The banking angle is regulatory before it is technical. The frame (each item flagged per its verification status):

- **MAS (Singapore) — the Technology Risk Management notices and Notice 645 (cyber hygiene).** The MAS TRM framework and Notice 645 push exactly the zero-trust properties — strong authentication, least privilege, continuous monitoring, resilience — without using the term (⚠-structural, per the [security_by_design_guide.md](security_by_design_guide.md) §3.7 ledger; note its §8.4 flag that the older Notice 655 was cancelled/superseded in the 2024 MAS notice renumbering — check the current notice number before quoting it in a design document). For a Singapore-based CIB, Notice 645's access-control and cyber-hygiene expectations are the practical exam a ZTNA programme helps pass.
- **EU DORA — the Digital Operational Resilience Act (Regulation (EU) 2022/2554, applicable 17 January 2025).** ✅ (verified in the security_by_design ledger). DORA demands operational resilience with documented, tested ICT risk controls — including access management and the protection of critical systems. A ZTNA estate is how a bank *demonstrates* least-privilege access and rapid containment to a DORA examiner; equally, a ZTNA policy engine that becomes a single point of failure is a DORA resilience finding (the [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) cross-ref again).
- **SWIFT CSP — the Customer Security Programme.** SWIFT's Customer Security Controls Framework requires member banks to secure their local SWIFT infrastructure (⚠-structural — widely documented control framework, not re-verified this pass): restricting access to the SWIFT environment, MFA, and segregation of the interface from other systems are CSP controls that align directly with the network/application pillars. The op-risk framing lives in [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md) — a ZTNA failure is an operational-risk event with regulatory-reporting consequences, and the compliance machinery that must consume the audit trail is [regtech_guide.md](../banking/regtech_guide.md)'s subject.
- **The international convergence.** The US federal mandates (EO 14028, M-22-09 — §7.2) set the programme shape; MAS, the EBA, and the HKMA are converging on the same posture language (⚠-structural). The CISA maturity-model vocabulary (§7.3) is becoming the common scorecard — worth adopting as the bank's own reporting format before a regulator asks for it.

### 8.3 The Banking Table

| Bank estate element | The legacy (implicit-trust) shape | The ZTNA shape | Pillar(s) | Cross-ref |
|---|---|---|---|---|
| **Remote workforce** | VPN concentrator → full network access; once in, everything is reachable | ZTNA broker → per-application access; device posture + MFA per session; no network access | Identity, Device, Application | [distributed_auth_guide.md](distributed_auth_guide.md) (the MFA/authN engine); this guide §5.3 |
| **Third-party vendors** | Vendor VPN accounts; shared credentials; "trusted partner" network paths | Brokered per-application access, time-boxed, posture-checked, fully audited — the vendor never touches the network | Identity, Application, Data | NIST §4.3 ✅; this guide §9.2 |
| **VDI / remote desktops** | Full desktops inside the network as the price of remote work | VDI shrinks to genuinely thin-client cases; apps brokered directly | Application, Network | This guide §8.1 |
| **Data center / core banking** | Coarse zones; east-west traffic trusted once inside | Microsegmentation per workload flow (payments → ledger → core host); east-west policy enforced | Network, Data | This guide §5.2; [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) (the availability side) |
| **Data platforms (warehouse/lakehouse)** | Network-zone access to the data estate | Identity-aware data access; RLS/column security; tokenization; data-access audit; the data pillar made real | Data | [data_architect_skillgaps_guide.md](data_architect_skillgaps_guide.md) §1.7 — the standing priority this guide's cluster serves |
| **Financial extranet** | BT Radianz-class financial VPNs as the connectivity layer | The extranet stays (it is connectivity, not user access), but user-facing access to extranet services moves through the ZTNA layer | Network | [financial_infrastructure_guide.md](../banking/financial_infrastructure_guide.md) |
| **Regulatory evidence** | Access reports reconstructed after the fact | Per-request audit logs, posture history, exception registers — evidence as a by-product of the architecture | Cross-cutting (Governance) | [regtech_guide.md](../banking/regtech_guide.md); [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md) |

---

## 9. The Worked Example — a ZTNA Design for a Cymbal Bank

### 9.1 The Scenario

**The context:** a Cymbal Bank global corporate and investment bank — let us call the design artifact *Meridian Bank* (following the repo's house style of named worked examples; the scenario is fictional but the estate shape is the one familiar from [credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md) and the [banks_in_singapore_guide.md](../banking/banks_in_singapore_guide.md) cluster): ~90,000 users across 50+ countries; a global markets division (front-office trading, e-trading platforms, market-data infrastructure), a transaction-banking division (payments, cash management, trade finance), and the corporate functions; a hybrid estate of on-prem data centers, private cloud, and a growing public-cloud footprint; a core-banking/payments tier with mainframe-class systems; a data platform (warehouse + lakehouse) that is the target of the repo's standing priority; and a remote-access estate of legacy VPN concentrators, a large VDI tier, and hundreds of third-party vendor connections. **The mandate:** the board has approved a three-year zero-trust programme; the CISO's office has asked for the target architecture. This section is that architecture — identity-first, pillar-structured, tenet-reviewed.

**The design principle from the start:** do not buy "zero trust" — build the five pillars and buy the components (§6.3's selection criteria). The programme's first artifact is the CISA-style maturity self-assessment (§7.4 step 1): the scoring shows Identity at Initial, Devices at Initial, Network at Traditional, Applications at Traditional (the VPN estate), Data at Traditional — which dictates the order of work: identity first, then ZTNA the remote/third-party access, then segment the data center and the data platform, then automate.

### 9.2 The Design — Identity-First

**Layer 1 — the identity fabric (the Identity pillar's engine).** One authoritative identity platform (an Okta-class or Entra-ID-class fabric — the mechanics are [distributed_auth_guide.md](distributed_auth_guide.md)'s subject: OIDC/SAML federation, JWT claims, mTLS for services) federating every application; phishing-resistant MFA (FIDO2/WebAuthn hardware keys) as the baseline for all staff and vendors; risk-based authentication (location, device, behavior) feeding the access decisions; a service-identity program (SPIFFE/SPIRE-style workload identity) for machine-to-machine — the machine identities are where a bank's API estate ([api_governance_guide.md](api_governance_guide.md)) and its agentic-AI future ([enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md)) will live. **The rule:** no ZTNA broker is purchased until the fabric can express "who + device + context" as a policy input — tenet 4.

**Layer 2 — the ZTNA access layer (the Application pillar).** A ZTNA broker (the reference design uses the Cloudflare-Access-style identity-aware-proxy pattern for app-by-app speed, with a Zscaler-class cloud broker evaluated for the full remote-workforce estate — the §6.2 layer model) fronting every remote-reachable application: the e-trading platform, the payment-initiation systems, the AML/case-management apps, the market-data portals, the data-platform UIs. Deployment shapes per NIST §3.2: **device agent** on managed laptops (the majority of the remote workforce), **resource portal** for third-party vendors (a branded portal that brokers access to exactly the apps the vendor contract covers), and **enclave-based** for the data-center tier. Every broker terminates at the identity fabric; every session is per-application (tenet 3); every grant re-checks posture and risk continuously (tenets 5 and 6).

**Layer 3 — device posture (the Device pillar).** MDM/UEM (Intune/Jamf-class) with a compliance baseline for every managed device; EDR on all endpoints; the posture service feeds the brokers (unpatched laptop → reduced or denied access; jailbroken device → denied); a managed-vs-unmanaged policy (unmanaged devices get browser-based access to a whitelisted subset, never the full agent path). The posture-to-policy loop is the automation the maturity model's Advanced stage requires.

**Layer 4 — the segmented data plane (the Network and Data pillars).** East-west microsegmentation of the data center and cloud estates (NSX/ACI-class policy fabrics): the payments app can reach the ledger DB on its port, and nothing else; the data platform sits behind identity-aware access with row/column-level security, tokenization for the PII columns, KMS/HSM-backed encryption, and audit on every data access — the standing priority's data-plane controls ([data_architect_skillgaps_guide.md](data_architect_skillgaps_guide.md) §1.7), layered on top of the ZTNA access layer rather than assumed to come with it.

**Layer 5 — visibility and governance (the cross-cutting capabilities).** All broker decisions, posture events, and data accesses stream to the SIEM/SOAR estate; entity-behavior analytics correlate them (tenet 7); a zero-trust architecture board owns the policy catalogue, the exception register (per [security_by_design_guide.md](security_by_design_guide.md) §3.4's recorded-exception discipline), and the annual maturity re-score. The resilience question is decided per application in the design, not in the incident: the broker's fail-secure/fail-open posture is an explicit, recorded decision per application class ([zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) — the availability/security tension), with the trading floor treated differently from back-office batch.

### 9.3 The Phased Plan

| Phase | Scope | Pillar(s) | Exit criteria |
|---|---|---|---|
| **0 — Assess (months 0–2)** | CISA-style maturity self-assessment; application inventory; the "toxic data first" map (Kindervag's inside-out) | All | Published scorecard + the crown-jewel application list (payments, settlement, data platform) |
| **1 — Identity hardening (months 2–8)** | Single fabric; MFA everywhere (staff + vendors); lifecycle automation; service identities | Identity | 100% MFA on all remote access; no shared accounts in scope; the fabric can emit tenet-4 attributes |
| **2 — ZTNA the remote estate (months 6–18)** | Broker in front of the crown-jewel apps; pilot cohort → full remote workforce; vendor portal replaces vendor VPNs | Application, Device | VPN user access retired for the pilot app classes; vendor network access retired; audit shows per-application sessions |
| **3 — Segment the data plane (months 12–30)** | Microsegmentation of DC/cloud; identity-aware data access; RLS/tokenization/audit on the data platform | Network, Data | East-west policy enforced on crown-jewel flows; data-platform access is identity-aware and fully audited |
| **4 — Automate and close the loop (months 24–36)** | Posture-to-policy automation; policy-as-code; behavior analytics feeding policy; annual re-score cadence | Cross-cutting | Advanced-stage score on all pillars; the broker is no longer a manual exception shop |

The phases overlap deliberately (phase 2 starts while 1 is finishing); the sequencing rule is that **no phase starts on a pillar whose prerequisite is missing** — ZTNA before identity would repeat the classic failure of buying the tool before the foundation.

### 9.4 The Threat-Model Cross-Check

The design is reviewed against NIST §5's threat list and the repo's STRIDE lens ([threat_modeling_guide.md](threat_modeling_guide.md)) before sign-off:

| Threat (NIST §5 / STRIDE) | The design's answer | Residual risk / flag |
|---|---|---|
| **Subversion of the decision process (Spoofing/Tampering)** | PE/PA/PEP separation; the identity fabric is the single authority; brokers are read-only consumers of policy | Compromise of the identity fabric itself is the catastrophic case — flagged: privileged-access protection for the fabric admins (PAM), hardware-key MFA for them, and SoD per [security_by_design_guide.md](security_by_design_guide.md) §3.6 |
| **Denial-of-service / network disruption (DoS)** | Broker redundancy across regions; recorded fail-open/fail-closed per application class | Trading-floor availability vs security tension — flagged as an explicit, board-reviewed exception register |
| **Stolen credentials / insider (Spoofing/Repudiation)** | MFA + device posture + risk-based auth; per-session scopes; full audit | Credential phishing of the MFA itself (AiTM) — flagged: phishing-resistant FIDO2 rollout is phase-1 non-negotiable |
| **Visibility loss (Information Disclosure)** | All broker/access/data events to SIEM; behavior analytics | Legacy systems that cannot emit logs — flagged: log-and-lift bridges for the mainframe tier |
| **Proprietary formats / lock-in** | Tenet-based evaluation (§6.3) — the brokers must not become the policy silo | Vendor consolidation risk — flagged: the identity fabric, not the broker, owns policy |
| **Non-person entities (NPEs) in administration** | Service identities (SPIFFE) for workloads; no shared admin accounts | Machine identity sprawl — flagged: an inventory and lifecycle process for service identities is phase-4 scope |

The second review lens is the seven tenets themselves — the design is checked against NIST's specification, not just against the threat list. The tenet-by-tenet review checklist:

| Tenet (NIST §2.1, ✅ verbatim) | The Meridian design's answer | Verified in the design by… |
|---|---|---|
| **1 — All data sources and computing services are considered resources** | Every app, data store, API, and service in scope is brokered or segmented; the "toxic data first" map (§9.1) enumerates the resources | The application inventory from phase 0; the crown-jewel list |
| **2 — All communication is secured regardless of network location** | Broker tunnels are encrypted end-to-end; east-west flows are mTLS where enforced; office and home traffic take the same path | The broker/segmentation configs; the "no location privilege" rule (§5.3) |
| **3 — Access is granted on a per-session basis, least privilege** | Every broker session is per-application; no transitive grants; vendor access is time-boxed | Session logs showing per-app scopes; the vendor-portal entitlements |
| **4 — Access is determined by dynamic policy** | The identity fabric emits identity + device + context attributes; the brokers consume them as policy inputs | The policy catalogue and its attribute sources (§9.2 layer 1) |
| **5 — The enterprise monitors the integrity and posture of all assets** | MDM/UEM compliance + EDR feeds the posture service; non-compliant devices get reduced/denied access | The posture-to-policy loop (§9.2 layer 3) |
| **6 — All authN/authZ are dynamic and strictly enforced before access** | MFA (FIDO2) on every session; continuous re-check; risk-based authentication | The broker decision logs; the MFA adoption metric from phase 1 |
| **7 — The enterprise collects state information and uses it to improve posture** | All decisions/events stream to SIEM; behavior analytics feed policy updates; annual re-score | The SIEM pipeline and the §7.5 scorecard cadence |

The dual review (threats + tenets) is the design's sign-off ritual: the threat table catches what can *attack* the architecture; the tenet table catches where the architecture *fails its own specification*. A design that passes both is ready for the phase plan; one that fails either goes back to the drawing board — which is the entire point of reviewing before building.

### 9.5 The Lessons

1. **Identity is the load-bearing wall.** Every pillar's policy is expressed in identity terms; a bank that half-builds the identity fabric will watch every downstream pillar underdeliver. This guide's identity cross-ref ([distributed_auth_guide.md](distributed_auth_guide.md)) is not decorative — it is the engineering reference for the most important layer.
2. **ZTNA is the entry point, not the destination.** The broker transforms the remote-access estate (the biggest, oldest attack surface) fast and contained — but the data pillar (the standing priority) and the cross-cutting capabilities are where the multi-year value lives. The §9.3 plan sequences ZTNA early precisely so the harder pillars have their foundation laid.
3. **The maturity model is the governance.** Scoring the pillars annually gives the board and the regulator a vocabulary, a progress story, and an evidence trail; the programme survives leadership changes because the scorecard is the artifact, not a person's enthusiasm.
4. **Assume breach — and prove containment.** The design's worth is measured in blast radius: how far does a compromised laptop, vendor, or workload get? Microsegmentation and per-application brokering are the containment answers; the §9.4 review keeps the residual risks explicit and owned.
5. **The trust perimeter is gone — but the decision perimeter is everywhere.** The final architecture has no "inside" — and every resource has a PEP, every session a decision, every access an audit row. That is not a weaker posture; it is the 2020s definition of a strong one.

---

## 10. The One-Page Summary

**Zero Trust Network Architecture in one page:**

- **The definition.** Zero trust is "never trust, always verify": no implicit trust based on network location or asset ownership; every request authenticated, authorized, and encrypted; access decisions continuous, least-privilege, and policy-driven. **ZTNA** is the product category that delivers the remote-access half: default-deny, per-application, brokered connections — the VPN replacement.
- **The canon.** Kindervag coined the model at Forrester in 2010 (*No More Chewy Centers*, Sept 14, 2010 — the chewy-center critique; *Build Security Into Your Network's DNA*, Nov 5, 2010 — "design from the inside out"). Google proved it at scale (BeyondCorp, disclosed December 2014 — the VPN-replacement, identity-centric model). NIST standardized it (SP 800-207, August 2020 — the seven tenets). CISA made it a programme (ZTMM v1 2021 → v2 April 2023 — five pillars, four stages). Gartner named the category (ZTNA — ⚠-structural attribution, flagged in the ledger).
- **The seven tenets** (NIST SP 800-207, verified verbatim): all data sources and computing services are resources; all communication is secured regardless of network location; access is granted per-session with least privilege; access is determined by dynamic policy; the enterprise monitors the integrity and posture of all assets; authentication and authorization are dynamic and strictly enforced; the enterprise collects state information and uses it to improve its posture.
- **The five pillars** (CISA): Identity, Devices, Networks, Applications & Workloads, Data — matured Traditional → Initial → Advanced → Optimal, glued by Visibility & Analytics, Automation & Orchestration, and Governance.
- **The architecture.** Three complementary patterns: the SDP (dark cloud — resources invisible until brokered), microsegmentation (east-west blast-radius containment), and the identity-centric model (access proxy per resource, no privileged intranet). PE/PA/PEP separation makes the decision process survivable.
- **The vendors.** Layers, not brands: identity fabric (Okta, Entra), access brokers (Zscaler ZPA, Cloudflare Access, Netskope, Palo Alto Prisma Access, Cisco), segmentation (Cisco SDA, NSX/ACI), hyperscaler-native options — evaluated against the tenets, not the marketing.
- **The implementation.** Score yourself on the CISA maturity model; fix identity first; ZTNA the remote and third-party access; segment the crown jewels; automate; re-score annually. The US federal mandates (EO 14028, OMB M-22-09) set the programme shape the regulators everywhere now expect.
- **The banking angle.** The remote-access transformation: VPN/VDI estates → per-application brokering; third-party access finally scoped; the data pillar protecting the crown jewels; MAS, DORA, and SWIFT CSP as the regulatory floor. The worked example (Meridian Bank, §9) shows the identity-first design, the phased plan, and the threat-model cross-check.
- **The one-line answer to "is this the same as a firewall refresh?"** No. The firewall protected the perimeter; ZTNA abolishes the perimeter's authority. The network is no longer the security boundary — the access decision is.

**The final word: the trust perimeter is gone.** Not moved, not shrunk, not virtualized — gone. What replaces it is a decision at every resource, a verification on every session, and an audit row on every access. That is the whole discipline, and it is why this guide closes the repo's standing priority: zero trust is not a product in the stack — it is the stack's new operating model.

---

## The Glossary

- **Zero trust (ZT)** — the security model: no implicit trust granted to assets or accounts based on network location or ownership; every request is authenticated, authorized, and encrypted (NIST SP 800-207 ✅). The doctrine.
- **ZTNA / Zero Trust Network Access** — the product category for zero-trust remote application access: default-deny, identity + device-posture checks, per-session brokered connections; the VPN replacement (definition ✅ via Palo Alto cyberpedia; Gartner coinage ⚠-structural).
- **Never trust, always verify** — the discipline's slogan: verification is continuous and location-independent; nothing is trusted because it is "inside."
- **Kindervag** — John Kindervag, the Forrester analyst who coined the Zero Trust Model in 2010 (*No More Chewy Centers*, Sept 14, 2010 ✅).
- **Forrester** — the research firm that published the coinage reports (2010 ✅) and later the Forrester New Wave: ZTNA evaluations.
- **NIST** — the US National Institute of Standards and Technology; author of SP 800-207 ✅.
- **SP 800-207** — NIST Special Publication 800-207, *Zero Trust Architecture*, August 2020 ✅ — the seven tenets, logical components, approaches, threats.
- **Tenet** — one of the seven normative properties a ZTA must satisfy (NIST §2.1 ✅).
- **BeyondCorp** — Google's identity-centric, VPN-free access architecture, disclosed December 2014 (Ward & Beyer ✅).
- **Google** — the company whose BeyondCorp proved the model at scale and whose BeyondCorp Enterprise productized it.
- **VPN** — the legacy remote-access technology ZTNA replaces: grants network-level access (too much, after authentication) — the implicit-trust model's access tool.
- **SDP / Software-Defined Perimeter** — the CSA architecture (SDP Specification 1.0, April 2014 per NIST's reference list ✅) where a controller brokers per-connection access and resources stay dark; ZTNA's technical ancestor.
- **Microsegmentation** — fine-grained east-west segmentation with per-flow policy; the blast-radius containment pattern (NIST §3.1.2 ✅).
- **Identity** — pillar 1: the authoritative "who" (people and services), with authentication, MFA, and attributes — the engine of the model.
- **Device** — pillar 2: the requesting asset's inventory, posture, and compliance — "no asset is inherently trusted."
- **Network** — pillar 3: transport and segmentation, no longer a trust boundary.
- **Application** — pillar 4: applications and workloads as resources with per-session, least-privilege access.
- **Data** — pillar 5: the protected surface itself — classification, encryption, masking, row/column-level security, audit.
- **Pillar** — one of the five CISA programme areas (Identity, Devices, Networks, Applications & Workloads, Data) ✅.
- **CISA** — the US Cybersecurity and Infrastructure Security Agency; author of the Zero Trust Maturity Model (v1 2021, v2 April 2023 ✅).
- **Maturity model** — the CISA roadmap: five pillars × four stages (Traditional → Initial → Advanced → Optimal) with example controls ✅.
- **Zscaler** — vendor; Zscaler Private Access (ZPA) cloud ZTNA (⚠-structural positioning).
- **Cloudflare** — vendor; Cloudflare Access, the identity-aware-proxy ZTNA (⚠-structural positioning).
- **Netskope** — vendor; Netskope One SSE/SASE with ZTNA and CASB heritage (⚠-structural positioning).
- **Palo Alto** — vendor; Prisma Access ZTNA 1.0/2.0 within Prisma SASE (definition ✅ via their cyberpedia; leadership claims are vendor claims).
- **Cisco** — vendor; Duo (identity/MFA), Secure Access (SSE), SDA (segmentation) (⚠-structural positioning).
- **Okta** — vendor; the identity fabric (Identity Cloud / Identity Engine) under the ZTNA brokers (⚠-structural positioning).
- **Remote access** — the access pattern ZTNA transforms: from network-level (VPN/VDI) to application-level brokering.
- **Perimeter** — the old security boundary (the firewall line); in zero trust, abolished as a trust authority — "the trust perimeter is gone."
- **Implicit trust** — the assumption zero trust eliminates: that being inside the network, or previously verified, grants access.
- **PDP / PE / PA / PEP** — policy decision point (decides), policy engine (the decision logic), policy administrator (executes), policy enforcement point (enforces, per resource) — the NIST logical components ✅.
- **Dark cloud** — the SDP property: unauthorized clients see nothing — no ports, no responses, no apps (✅ per Palo Alto's ZTNA explainer).
- **Assume breach** — the operating assumption: design for containment because the attacker is or will be inside.
- **SASE** — Secure Access Service Edge: the convergence of WAN and security services (ZTNA included) into cloud-delivered form (⚠-structural).
- **Broker** — the ZTNA service that sits between subject and application: authenticates, checks posture, evaluates policy, and brokers the encrypted connection (the packaged PE/PA/PEP).
- **Connector** — the SDP-descended component inside the enterprise network that opens the outbound-only connection to the application on the broker's instruction; the application's address never reaches the client.
- **Posture** — the device's compliance state (managed, patched, EDR-protected, not jailbroken) as evaluated before and during access (tenet 5).
- **Identity fabric** — the centralized identity platform (Okta/Entra-class) that all brokers consume: SSO, MFA, lifecycle, attributes — the Identity pillar's engine.
- **ZTNA 2.0** — Palo Alto's term for next-generation ZTNA (continuous trust evaluation, all application types, data protection inside sessions) — ⚠ vendor terminology, not a standard.
- **East-west / north-south** — traffic between workloads (east-west, the lateral-movement path microsegmentation controls) vs traffic into/out of the estate (north-south, what the edge firewall historically controlled).
- **Chewy center** — Kindervag's 2010 metaphor for the trusted interior of the perimeter model: hard shell (firewall), soft center (everything inside) ✅.
- **VDI** — Virtual Desktop Infrastructure, the legacy remote-work pattern that gives users a desktop inside the network; ZTNA shrinks it to genuinely thin-client cases (§8.1).

---

## Claims Status and Verification Notes

**Verified this pass (live web, 2026-08-25):**

- ✅ **NIST SP 800-207** — published August 2020 (final 11 Aug 2020), 59 pp., authors Rose/Borchert/Mitchell/Connelly; abstract and the seven tenets quoted verbatim from the extracted PDF (csrc.nist.gov + nvlpubs.nist.gov).
- ✅ **Kindervag coinage** — *No More Chewy Centers: Introducing The Zero Trust Model of Information Security*, Forrester, September 14, 2010 (date verified via the November 5, 2010 sequel's "Related Research Documents" section); *Build Security Into Your Network's DNA: The Zero Trust Network Architecture*, Forrester, November 5, 2010 (full PDF extracted — the "stop trusting packets" philosophy and "design from the inside out" quoted from it).
- ✅ **BeyondCorp** — Ward & Beyer, USENIX *;login:* Vol. 39, No. 6 (December 2014), pp. 6–11; abstract quoted (usenix.org + research.google).
- ✅ **CISA ZTMM** — v1.0 initial release August 2021, public comment September 2021; v2.0 April 2023 aligned with OMB M-22-09; five pillars + three cross-cutting capabilities + four stages; pillar quotes from the extracted v2 PDF (cisa.gov).
- ✅ **EO 14028** (May 17, 2021), **OMB M-22-09** (Jan 26, 2022, FY2024 goals), **DoD ZT RA v2.0** (July 2022), **NSA ZT model** (Feb 2021) — cited within the CISA ZTMM v2 PDF.
- ✅ **CSA SDP Specification 1.0 — April 2014** — per NIST SP 800-207's reference list.
- ✅ **ZTNA category definition** — "default to deny… only the access to services the user has been explicitly granted" (Palo Alto Networks cyberpedia, extracted).
- ✅ **Repo facts** — sibling filenames, §3.7 content, the skill-gaps priority #3, and banking-folder filenames verified by direct file reads.

**Flagged ⚠ / ⚠-structural (honest gaps):**

- ⚠ **Gartner ZTNA coinage** — the category name is universally attributed to Gartner (2018 Innovation Insight / 2019 Market Guide for Zero Trust Network Access), but Gartner's site is paywalled and the search backend repeatedly returned empty for these documents; the attribution is recorded as ⚠-structural. No Gartner report title/date is asserted as fact in this guide.
- ⚠ **Adoption/market statistics** — none included; the search backend could not verify current figures, and the guide prefers no number over a fabricated one.
- ⚠ **Vendor positioning** — ✅-structural (industry-standard, cross-checked where extractable); vendor claims (Forrester New Wave leadership, ZTNA 2.0 coinage) are flagged as vendor claims.
- ⚠ **Marsh 1994 / Jericho Forum 2003 / Operation Aurora trigger / NCSC 2019 / ETSI 2025** — secondary-sourced (Wikipedia citations), not directly verified this pass.
- ⚠ **DISA Black Cloud origins of SDP** — widely documented, not directly verified this pass.
- ⚠ **MAS TRM / Notice 645 / SWIFT CSP / DORA-for-ZTNA alignment** — per the security_by_design_guide.md ledger (itself verified on 2026-08-24); the MAS notice numbering changed in 2024 (Notice 655 cancelled/superseded — check current numbers before quoting in a design doc).
- ⚠ **The implementation-table row content (§7.3)** — a synthesis of the ZTMM's example controls, not a quote.
- ⚠ **Meridian Bank** — a fictional worked example; no real Cymbal Bank system is described.

---

## Cross-References and Further Reading

**Siblings (technology/, plain filenames):** [security_by_design_guide.md](security_by_design_guide.md) — the commissioning guide; §3.7 is this guide's parent section (the Zero Trust tie-in), §3.2–§3.6 the principle-level pillars, §8 the regulatory floor. [distributed_auth_guide.md](distributed_auth_guide.md) — the identity pillar's engineering reference (authN/authZ, OIDC/SAML/JWT/mTLS, RBAC/ABAC). [api_governance_guide.md](api_governance_guide.md) — the API-security angle (gateways as PEPs, API authz). [threat_modeling_guide.md](threat_modeling_guide.md) — STRIDE against the ZTNA estate (§9.4). [data_architect_skillgaps_guide.md](data_architect_skillgaps_guide.md) — the standing-priority frame this guide closes (priority #3: Zero Trust for data platforms). [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) — the resilience angle (fail-secure vs fail-open, the policy engine as a dependency). [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md) — the identity-aware-proxy pattern in the AI-gateway variant. [configuration_management_languages_guide.md](configuration_management_languages_guide.md) — policy-as-code tooling. Also of interest from the security cluster: the threat-model, secure-coding, and DevSecOps material inside security_by_design_guide.md §4–§6.

**Banking (../banking/ prefix):** [credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md) — the house-systems context for §9. [banks_in_singapore_guide.md](../banking/banks_in_singapore_guide.md) — the SG estate frame. [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md) — the op-risk framing of a ZTNA failure. [regtech_guide.md](../banking/regtech_guide.md) — the compliance machinery consuming the audit trail. [financial_infrastructure_guide.md](../banking/financial_infrastructure_guide.md) — the legacy financial-extranet/VPN layer (§8.1). The bank software-systems series (HSBC, BNP Paribas, Deutsche Bank, UOB, OCBC, Bank of America…) for remote-access estate precedents.

**Management (../management/ prefix):** [business_case_development_guide.md](../management/business_case_development_guide.md) — the ZTNA-investment angle (cost of breach vs cost of transformation; pilot evidence).

**Primary sources (all verified this pass unless flagged):** NIST SP 800-207 (csrc.nist.gov/pubs/sp/800/207/final; nvlpubs.nist.gov PDF) ✅ · CISA Zero Trust Maturity Model v2.0 (cisa.gov page + PDF) ✅ · Kindervag, *No More Chewy Centers* (Sept 14, 2010) ✅ via *Build Security Into Your Network's DNA* (Forrester, Nov 5, 2010, PDF) ✅ · Ward & Beyer, *BeyondCorp: A New Approach to Enterprise Security*, USENIX *;login:* Dec 2014 ✅ · CSA SDP Specification 1.0 (April 2014, per NIST ref list) ✅ · Palo Alto Networks cyberpedia, *What Is Zero Trust Network Access (ZTNA)* ✅ · Wikipedia, "Zero trust architecture" (secondary — Marsh 1994, Jericho Forum, Operation Aurora, NCSC 2019, ETSI TS 104 102 2025) ⚠-secondary · Gartner ZTNA documents ⚠ not directly accessible.

**The canonical document stack (the five to read if you read five):** (1) NIST SP 800-207 — the standard, §2.1 (tenets) and §3 (components) are the mandatory reading; (2) the CISA Zero Trust Maturity Model v2.0 PDF — the pillars and the Traditional→Optimal example controls; (3) Kindervag's *Build Security Into Your Network's DNA* (the 2010 PDF is publicly mirrored) — the "inside out" philosophy in the original voice; (4) Ward & Beyer's *BeyondCorp* (*;login:* Dec 2014) — the working proof of the identity-centric model; (5) OMB M-22-09 — the programme shape (strategy → pillar objectives → deadlines → reporting) that every follow-on regulator expects. Everything else in this guide is commentary on those five.

**Series note:** this guide closes the standing priority #3 ("Zero Trust for data platforms") named in [data_architect_skillgaps_guide.md](data_architect_skillgaps_guide.md) and completes the security cluster's zero-trust arc that began in [security_by_design_guide.md](security_by_design_guide.md) §3.7. The natural next commissions in the same list remain platform engineering for data (priority #4) and business-case/TCO modeling (priority #2) — each already has its seed references in this guide's cross-refs (the identity-aware data plane here, the business-case frame in §7). Until then, this guide stands as the cluster's zero-trust reference: the tenets to argue with, the pillars to plan with, the maturity model to score with, and the worked example to adapt — because in 2026, the trust perimeter is gone, and the decision perimeter is the architecture.

*End of guide — verified 2026-08-25 with live web access; all ✅ claims trace to the primary sources in the ledger above.*
