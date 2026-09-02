# Fireblocks: The Institutional Digital-Asset Infrastructure Company — A Comprehensive Guide

**The Business, History, Products, Funding, Licensing, Security, and Technology of Fireblocks (fireblocks.com) — from a 2018 Founding by Three Check Point Alumni to MPC Custody for 2,400+ Enterprises, US$10 Trillion in Transferred Value, the Fireblocks Network, Tokenization, and a Cymbal Bank Digital-Asset Custody Client Worked Example**

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Banking Domain / Digital-Asset Infrastructure Company Deep-Dive — MPC-based custody and wallet infrastructure, the Fireblocks Network settlement layer, tokenization and stablecoin rails, the funding history (2019–2022), the licensing and compliance posture, the security architecture and incident record, and the Cymbal Bank custody-client lens
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026
> **Companion guides (sibling, same folder — the banking cluster):** [Reap Global](reap_global_guide.md) (Fireblocks is Reap's custodian wallet for stablecoin repayments, Jan 2023 — this guide is the dedicated deep-dive that guide points to) · [Bitunix](bitunix_guide.md) (the exchange worked-example conventions; Bitunix holds institutional custody via Cobo + Fireblocks — cross-ref §10) · [Tokenized Assets](tokenized_assets_guide.md) (Fireblocks positioned as the institutional MPC custody + transfer-network standard and tokenization tooling provider — cross-ref §5, §6) · [Fircosoft](fircosoft_guide.md) (AML/sanctions screening themes — cross-ref §7, condensed) · [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) (the Singapore PS Act / DPT regime — cross-ref §8, condensed) · [Payment Rails](payment_rails_guide.md) (clearing and settlement mechanics for the fiat legs — cross-ref §6, §11)
> **Companion guides (technology/, prefix `../technology/`):** [Smart Contracts](../technology/smart_contracts_guide.md) (the on-chain mechanics — ERC-20, Merkle proofs, custody and wallet security, audit practice, incident history — cross-ref §9, condensed, not re-derived) · [Cybersecurity](../technology/cybersecurity_guide.md) (the security-discipline themes — hot/cold wallet separation, multi-signature, ISO/IEC 27001, bug bounties — cross-ref §9, condensed)

---

**How to use this guide:** Section 1 is the overview — the short answer, the key-facts table, why a bank should care, and the evidence base. Section 2 is the company profile — the 2018 founding, the three verified co-founders (Michael Shaulov, Idan Ofrat, Pavel Berengoltz), the New York headquarters, and the Israeli R&D roots. Section 3 is the mission and business model — the "replatform finance onto digital assets" mission and the platform/network/tokenization revenue stack (⚠ the revenue split is not public). Section 4 is funding and growth — every round from the June 2019 US$16 million Series A to the February 2022 US$550 million Series E at an US$8 billion valuation, each verified at the company's own press pages and reputable press, with the research brief's believed figures corrected where they differ. Section 5 covers the core products — MPC custody, the workspace/asset-account model, the policy engine, and the wallet infrastructure. Section 6 covers the rest of the roster — the Fireblocks Network (settlement, off-exchange settlement, collateral management), tokenization, stablecoin/payments, staking and DeFi, the API platform, and the competitive landscape. Section 7 is licensing and compliance — the registrations that could be verified, the KYC/AML posture (cross-referencing the Fircosoft guide's screening themes, condensed). Section 8 is the Singapore angle — the MAS payment-institution and DPT regime (cross-referencing the MAS guide), with Fireblocks' own Singapore status flagged ⚠ where unclear. Section 9 is security and technology — the MPC/threshold-signature architecture, the policy engine and co-signing model, HSM/TEE integration, and the certifications (SOC 2 Type 2, ISO 27001, CCSS Level 3) — each marked ✅ where independently attested and ⚠ where only company-published. Section 10 is the security-incident and research record — the January 2022 API-disclosure episode, the December 2022 BitGo vulnerability research, and the Lazarus Group phishing campaigns Fireblocks has itself documented. Section 11 is the Cymbal Bank worked example — a digital-asset custody client on Fireblocks-style infrastructure, following the persona conventions of the repository. Section 12 is the claims audit (✅/⚠/❌), with §12.4 "What Could Not Be Verified" collecting every item that could not be confirmed. Section 13 is the glossary. Section 14 is cross-references and the closing summary. **Repository integrity convention:** ✅ = verified this pass at a primary or authoritative source; ⚠ = flagged (company-reported, single-sourced, time-varying, or unverifiable); ❌ = refuted or not found. Nothing is fabricated; where a fact could not be confirmed it is flagged rather than smoothed.

---

## Table of Contents

1. [The Overview](#1-the-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Key-Facts Table](#12-the-key-facts-table)
   - 1.3 [Why This Matters to a Bank](#13-why-this-matters-to-a-bank)
   - 1.4 [The Evidence Base at a Glance](#14-the-evidence-base-at-a-glance)
2. [The Company Profile — Founding, Founders, and Headquarters](#2-the-company-profile--founding-founders-and-headquarters)
   - 2.1 [The Founding Year: 2018](#21-the-founding-year-2018)
   - 2.2 [The Co-Founders: Shaulov, Ofrat, and Berengoltz](#22-the-co-founders-shaulov-ofrat-and-berengoltz)
   - 2.3 [Headquarters: New York, with Israeli R&D Roots](#23-headquarters-new-york-with-israeli-rd-roots)
   - 2.4 [The Origin Story: Securing the Private Key](#24-the-origin-story-securing-the-private-key)
   - 2.5 [The Footprint Timeline](#25-the-footprint-timeline)
   - 2.6 [The Company Table](#26-the-company-table)
3. [The Mission and Business Model](#3-the-mission-and-business-model)
   - 3.1 [The Mission Statement](#31-the-mission-statement)
   - 3.2 [The Business Model — Platform, Network, and Software Revenue](#32-the-business-model--platform-network-and-software-revenue)
   - 3.3 [The Model's Market Context](#33-the-models-market-context)
4. [Funding and Growth — The Rounds, Verified](#4-funding-and-growth--the-rounds-verified)
   - 4.1 [The Funding Table](#41-the-funding-table)
   - 4.2 [Series A (June 2019): US$16 Million](#42-series-a-june-2019-us16-million)
   - 4.3 [Series B (November 2020): US$30 Million, Led by Paradigm](#43-series-b-november-2020-us30-million-led-by-paradigm)
   - 4.4 [Series C (March 2021): US$133 Million](#44-series-c-march-2021-us133-million)
   - 4.5 [Series D (July 2021): US$310 Million at a US$2 Billion Valuation](#45-series-d-july-2021-us310-million-at-a-us2-billion-valuation)
   - 4.6 [Series E (February 2022): US$550 Million at an US$8 Billion Valuation](#46-series-e-february-2022-us550-million-at-an-us8-billion-valuation)
   - 4.7 [The Growth Metrics Over Time](#47-the-growth-metrics-over-time)
   - 4.8 [What the Brief Got Wrong — Corrections](#48-what-the-brief-got-wrong--corrections)
5. [The Products — MPC Custody and the Wallet Infrastructure](#5-the-products--mpc-custody-and-the-wallet-infrastructure)
   - 5.1 [Fireblocks Core / Workspace and the Asset-Account Model](#51-fireblocks-core--workspace-and-the-asset-account-model)
   - 5.2 [The MPC Wallet and Threshold Signatures](#52-the-mpc-wallet-and-threshold-signatures)
   - 5.3 [The Policy Engine and Governance](#53-the-policy-engine-and-governance)
   - 5.4 [Wallet-as-a-Service and Embedded Wallets](#54-wallet-as-a-service-and-embedded-wallets)
   - 5.5 [Treasury Management and the API Platform](#55-treasury-management-and-the-api-platform)
6. [The Products — The Fireblocks Network, Tokenization, Payments, and Staking](#6-the-products--the-fireblocks-network-tokenization-payments-and-staking)
   - 6.1 [The Fireblocks Network — Settlement Infrastructure](#61-the-fireblocks-network--settlement-infrastructure)
   - 6.2 [Off-Exchange Settlement and Collateral Management](#62-off-exchange-settlement-and-collateral-management)
   - 6.3 [Tokenization — the Tokenization Platform](#63-tokenization--the-tokenization-platform)
   - 6.4 [Stablecoin and Payments Infrastructure](#64-stablecoin-and-payments-infrastructure)
   - 6.5 [Staking and Institutional DeFi](#65-staking-and-institutional-defi)
   - 6.6 [The 2026 Product-Surface Map](#66-the-2026-product-surface-map)
   - 6.7 [The Competitive Landscape](#67-the-competitive-landscape)
7. [Licensing and Compliance](#7-licensing-and-compliance)
   - 7.1 [The Verified Registrations and Licenses](#71-the-verified-registrations-and-licenses)
   - 7.2 [The Unverified Licensing Claims](#72-the-unverified-licensing-claims)
   - 7.3 [The KYC/AML and Screening Posture (Cross-Referenced, Condensed)](#73-the-kycaml-and-screening-posture-cross-referenced-condensed)
   - 7.4 [The Licensing Table](#74-the-licensing-table)
8. [The Singapore Angle — MAS and the DPT Regime (Cross-Referenced)](#8-the-singapore-angle--mas-and-the-dpt-regime-cross-referenced)
   - 8.1 [The MAS Regime in One Paragraph](#81-the-mas-regime-in-one-paragraph)
   - 8.2 [Fireblocks in Singapore — What Is and Is Not Known](#82-fireblocks-in-singapore--what-is-and-is-not-known)
   - 8.3 [What the Regime Means for a Fireblocks-Style Custodian](#83-what-the-regime-means-for-a-fireblocks-style-custodian)
9. [Security and Technology — The MPC Architecture](#9-security-and-technology--the-mpc-architecture)
   - 9.1 [The Threshold-Signature Model (Cross-Referenced, Condensed)](#91-the-threshold-signature-model-cross-referenced-condensed)
   - 9.2 [The Distributed Key-Share Architecture](#92-the-distributed-key-share-architecture)
   - 9.3 [The Policy Engine, Co-Signing, and the No-Single-Party Claim](#93-the-policy-engine-co-signing-and-the-no-single-party-claim)
   - 9.4 [HSMs, TEEs, and the Zero-Trust Environment](#94-hsms-tees-and-the-zero-trust-environment)
   - 9.5 [Audits, Certifications, and the Bug-Bounty Program](#95-audits-certifications-and-the-bug-bounty-program)
   - 9.6 [The Verified Technology Facts Table](#96-the-verified-technology-facts-table)
10. [Security Incidents and the Threat-Intelligence Record](#10-security-incidents-and-the-threat-intelligence-record)
    - 10.1 [January 2022 — The API-Disclosure Episode](#101-january-2022--the-api-disclosure-episode)
    - 10.2 [December 2022 — The BitGo TSS Vulnerability Research](#102-december-2022--the-bitgo-tss-vulnerability-research)
    - 10.3 [2023–2025 — The Lazarus Group Phishing Campaigns](#103-20232025--the-lazarus-group-phishing-campaigns)
    - 10.4 [Customer-Side Incidents and the Vendor Boundary](#104-customer-side-incidents-and-the-vendor-boundary)
    - 10.5 [The Incident Table](#105-the-incident-table)
11. [The Cymbal Bank Worked Example — A Digital-Asset Custody Client](#11-the-cymbal-bank-worked-example--a-digital-asset-custody-client)
    - 11.1 [The Scenario](#111-the-scenario)
    - 11.2 [Onboarding and the Workspace](#112-onboarding-and-the-workspace)
    - 11.3 [The MPC Key Management and Quorum](#113-the-mpc-key-management-and-quorum)
    - 11.4 [Settlement via the Network and Off-Exchange Flows](#114-settlement-via-the-network-and-off-exchange-flows)
    - 11.5 [AML/Screening Integration and the MAS Overlay](#115-amlscreening-integration-and-the-mas-overlay)
    - 11.6 [The Lessons](#116-the-lessons)
12. [The Claims Audit — Verified, Flagged, Rejected](#12-the-claims-audit--verified-flagged-rejected)
    - 12.1 [The Verified Claims (✅)](#121-the-verified-claims-)
    - 12.2 [The Flagged Claims (⚠)](#122-the-flagged-claims-)
    - 12.3 [The Rejected or Not-Found Claims (❌)](#123-the-rejected-or-not-found-claims-)
    - 12.4 [What Could Not Be Verified](#124-what-could-not-be-verified)
13. [The Glossary](#13-the-glossary)
14. [Cross-References and the Closing Summary](#14-cross-references-and-the-closing-summary)

---

## 1. The Overview

### 1.1 The Short Answer

**Fireblocks** (fireblocks.com) is an institutional digital-asset infrastructure company that provides **MPC-based custody, wallet infrastructure, settlement networking, tokenization, and stablecoin/payments tooling** to financial institutions, fintechs, exchanges, and web3 companies. The company was **founded in 2018** by three cybersecurity veterans — **Michael Shaulov** (CEO), **Idan Ofrat**, and **Pavel Berengoltz** — all formerly of Check Point Software Technologies, and is **headquartered in New York** with its R&D roots in Israel. Its founding premise was that the private-key problem — the reason exchanges and custodians kept losing customer funds to hackers — could be solved with multi-party computation (MPC), so that **no single private key ever exists in one place** (fireblocks.com/about; fireblocks.com/blog/133m-series-c-fireblocks-is-here-for-the-long-run, which confirms "When Fireblocks first started in 2018, Galaxy Digital and Genesis Global Trading (DCG) were among our earliest adopters"). ✅

The company's product surface, verified at its own public pages, spans: the **MPC wallet / Fireblocks Core workspace** (custody, the asset-account model, the policy engine), **Wallet-as-a-Service** and **Embedded Wallets**, **Treasury Management**, the **Fireblocks Network** (an authenticated settlement network connecting institutions), **Tokenization** (issuance and smart-contract operations for digital securities and tokenized assets), **Stablecoin infrastructure** (payments, issuance, compliance), and **Staking/DeFi** tooling (fireblocks.com homepage and product pages, extracted September 2026). ✅ Customer logos on the company's own pages include BNY Mellon, BNP Paribas, ABN AMRO, HSBC, Revolut, eToro, Visa, Western Union, MoneyGram, Bridge, GSR, Wintermute, B2C2, and QCP — company-published references, treated as ⚠ where they imply commercial depth. The company's headline metrics — **2,400+ enterprises, US$10 trillion in transactions, 550 million wallets secured** (fireblocks.com homepage, September 2026) — are **company-reported and flagged ⚠** throughout this guide.

Funding is the most heavily documented part of the story, verified at the company's own press pages and reputable press: **US$16 million Series A (June 2019)**, **US$30 million Series B led by Paradigm (November 2020)**, **US$133 million Series C (March 2021)** led by Coatue, Ribbit and Stripes with strategic investment from BNY Mellon and SVB Capital, **US$310 million Series D (July 2021)** co-led by Sequoia Capital, Stripes, Spark Capital, Coatue, DRW VC and SCB 10X at a valuation over US$2 billion, and **US$550 million Series E (February 2022)** co-led by D1 Capital Partners and Spark Capital at a valuation over **US$8 billion** — roughly **US$1.04 billion raised in total** (✅ company press pages; ⚠ tracker-derived total). The research brief's believed figures (a ~US$30 million seed/Series A, a US$40 million Ribbit-led Series B) are **corrected** in §4.8: the Series A was US$16 million and the Series B was US$30 million led by Paradigm.

For a bank like Cymbal Bank, Fireblocks matters on four fronts: as the **reference architecture** for how institutions custody digital assets without a single point of key compromise (the MPC model banks now must understand and evaluate — §9), as a **settlement-network operator** whose authenticated network competes with and complements traditional clearing/settlement rails (§6, cross-ref [Payment Rails](payment_rails_guide.md)), as a **counterparty/vendor class** that banks meet repeatedly — the same name appears as custodian in sibling guides for [Reap](reap_global_guide.md) and [Bitunix](bitunix_guide.md) — and as a **worked-example generator** for the repository's Cymbal Bank persona (§11).

### 1.2 The Key-Facts Table

| Aspect | Fact | Status |
| --- | --- | --- |
| Full name / brand | Fireblocks (fireblocks.com); legal entity structure not fully public | ✅ brand / ⚠ entity |
| Founded | 2018 ("When Fireblocks first started in 2018..." — company blog; multiple independent trackers agree) | ✅ |
| Founders | Michael Shaulov (CEO), Idan Ofrat, Pavel Berengoltz — all ex-Check Point | ✅ (company About page + trackers) |
| Headquarters | New York, USA; R&D rooted in Israel (Tel Aviv) | ✅ HQ / ⚠ office details |
| Total funding | ~US$1.04 billion across 5–6 rounds (2019–2022) | ✅ rounds / ⚠ total |
| Valuation | >US$2B (Jul 2021) → >US$8B (Feb 2022) | ✅ (company press) |
| Customers | 2,400+ enterprises (2026 homepage); 800+ institutions (Feb 2022 press); 1,300+ (2023 claims) | ⚠ company-reported |
| Volume | US$10T in transactions, 550M wallets secured (2026 homepage); US$2T transferred (Feb 2022 press) | ⚠ company-reported |
| Product roster | MPC custody (Core/Workspace), WaaS, Embedded Wallets, Treasury, Fireblocks Network, Tokenization, Stablecoin/Payments, Staking/DeFi | ✅ (company pages) |
| Network | The Fireblocks Network — authenticated transfer channels connecting 2,400+ institutions; off-exchange settlement; collateral management | ✅ product exists / ⚠ scale |
| Security posture | MPC threshold signatures, distributed key shares, policy engine, HSM/TEE, SOC 2 Type 2, ISO 27001, C4 CCSS QSP Level 3, Bugcrowd bug bounty | ✅ architecture / ⚠ certifications company-published |
| Licensing | US MSB-type registrations, EU/UK/Swiss-type registrations claimed — see §7 | ⚠ mostly unverifiable this pass |
| Singapore (MAS) | No MAS DPT licence identified; status unclear | ⚠/❌ |

### 1.3 Why This Matters to a Bank

A Fireblocks-style infrastructure provider sits at the exact point where banking and digital assets collide. When a bank decides to custody bitcoin or ether for clients, issue a stablecoin, or settle tokenized securities, it has three options — build the cryptography in-house, buy a hardware-wallet vendor, or adopt an MPC-based infrastructure platform like Fireblocks. Understanding how that third option works — what a "threshold signature" is, why no single party holds a key, how the policy engine enforces quorums, and what the vendor can and cannot do with customer funds — is now core due-diligence material for any bank's third-party-risk, technology, and financial-crime teams. The same name recurs across this repository's banking cluster: [Reap](reap_global_guide.md) uses Fireblocks as its custodian wallet for stablecoin repayments (January 2023), and [Bitunix](bitunix_guide.md) pairs Cobo with Fireblocks for institutional custody — so a banker reading those guides needs this deep-dive to know what "Fireblocks custody" actually is.

The banking questions §11 works through: How does a bank onboard onto an MPC workspace, who holds the key shares, and what does the policy engine enforce? How does settlement over an authenticated network differ from on-chain settlement or from fiat clearing (cross-ref [Payment Rails](payment_rails_guide.md))? How do AML screening and sanctions compliance attach to a custody flow (cross-ref [Fircosoft](fircosoft_guide.md))? And what does the MAS digital-payment-token regime require of a custodian operating in Singapore (cross-ref [MAS Regulations](mas_regulations_guidelines_guide.md))? Fireblocks is also the institutional standard the [Tokenized Assets](tokenized_assets_guide.md) guide assumes when it discusses custody and wallet infrastructure — this guide is where that assumption is examined in detail.

### 1.4 The Evidence Base at a Glance

Every factual claim in this guide traces to one of five evidence classes, and the claims audit (§12) records which class supports each claim:

| Evidence class | Examples used in this guide | How it is treated |
| --- | --- | --- |
| Company pages and press (fireblocks.com) | About; homepage (Sep 2026); Security page; Fireblocks Network page; press releases for Series B/C/D/E; the Series C and Series D blogs; the BitGo technical report; the Contagious Interview threat report | ✅ where the fact is the page's own content (rounds, investors, product names); ⚠ where the page reports company metrics (customers, volumes, wallets) |
| Company trust center (trust.fireblocks.com) | SOC 2 / ISO / CCSS claims, security documentation | ⚠ company-published artifacts (third-party auditable in principle) |
| Developer docs (developers.fireblocks.com) | API and platform documentation referenced for the workspace/policy/co-signing model | ✅ for product mechanics that are documented; ⚠ where docs were not reachable this pass |
| Reputable press and independent research | TechCrunch (Series D, Jul 2021), Reuters/Yahoo Finance (Series E, Feb 2022), Calcalist (Series D), CoinDesk (Series E), The Block (BitGo disclosure coverage), NCC Group (Lazarus/fintech research), Ethena documentation (off-exchange providers) | ✅ for deal headlines and event existence; ⚠ for derived figures |
| Aggregators and trackers | Tracxn, Seedtable, Crunchbase, startupintros, Built In NYC | ✅ for round existence and dates; ⚠ for exact amounts where single-sourced |
| The research brief itself | Believed ~US$30M seed/Series A; believed US$40M Ribbit-led Series B; believed Sequoia-led Series D | ❌/⚠ where refuted or corrected — see §4.8 and §12.3 |

The methodological limitation of this pass is stated once here and not repeated: registry lookups that require interactive search (FinCEN MSB registrant list, state money-transmitter registers, the FCA register, the MAS Financial Institutions Directory) could not be queried directly, so every licensing finding that depends on them is flagged in §7 and §12.4 rather than asserted as proof.

---

## 2. The Company Profile — Founding, Founders, and Headquarters

### 2.1 The Founding Year: 2018

The founding year of **2018** verifies across the company's own materials and independent trackers:

- The company's own Series C blog: "**When Fireblocks first started in 2018**, Galaxy Digital and Genesis Global Trading (DCG) were among our earliest adopters" (fireblocks.com/blog/133m-series-c-fireblocks-is-here-for-the-long-run/, March 2021). ✅
- The About page's founder narrative: "Michael, Idan and Pavel each spent the last twenty years transforming how cybersecurity is being applied to mobility, cloud and critical infrastructure so the next obvious challenge was clear, to secure the blockchain" (fireblocks.com/about, extracted via search index September 2026; the page itself blocks automated extraction). ✅
- Independent trackers (startupintros.com, earlyriders.com, everybodywiki, Tracxn) all record a 2018 founding by Michael Shaulov, Idan Ofrat and Pavel Berengoltz. ✅
- One nuance: the Series D press release says "Since its **inception in 2019**..." (fireblocks.com/press, July 2021) — the company itself uses both dates, where 2018 is the founding/incorporation and June 2019 is the **product launch** (the Series B press release dates the platform launch: "Since launching in **June 2019**, Fireblocks has facilitated the transfer of over $150 billion..."). The clean reading, matching the sibling guides' convention, is **founded 2018, launched June 2019** — and the "2019" phrasing in later press is treated as a product-launch reference, flagged ⚠ where ambiguous.

### 2.2 The Co-Founders: Shaulov, Ofrat, and Berengoltz

The three co-founders are named consistently across the company's About page and every independent profile examined:

- **Michael Shaulov** — CEO. Formerly of Lacoon Mobile Security (a mobile-security startup acquired by Check Point in 2015) and, before that, Check Point itself; also a former intelligence/cyber officer (per his public profile). The company's press releases consistently quote him as CEO from the Series B (November 2020) onward, and he is the public face of every major announcement through the 2026 homepage. ✅
- **Idan Ofrat** — co-founder and CTO. Ex-Check Point; publicly credited as the technology lead on the MPC platform. ✅ (role per tracker profiles and company materials)
- **Pavel Berengoltz** — co-founder; ex-Check Point. ✅ (role per tracker profiles; exact title — CPO per some profiles — ⚠ single-sourced)

All three are repeatedly described as having spent roughly two decades in cybersecurity before founding Fireblocks, which is consistent with the About-page quote in §2.1. The founders' individual titles beyond Shaulov's CEO role are ⚠ (company-published bios were not extractable this pass; tracker roles are single-sourced). The pattern is worth noting for a banking reader: like many of the most successful digital-asset infrastructure firms, Fireblocks was founded by **security engineers, not bankers** — its credibility rests on cryptography and threat modeling, which is exactly what its product sells.

### 2.3 Headquarters: New York, with Israeli R&D Roots

- **Headquarters: New York, USA.** Multiple independent profiles (everybodywiki; Built In NYC's coverage of the Series E; TechCrunch's Series D coverage) place Fireblocks' headquarters in New York. ✅
- The company is widely described as an **Israeli-founded company** with significant R&D in Tel Aviv; the founding team's Check Point lineage and the Israeli press coverage (Calcalist) support the Israeli R&D anchor. The precise split of offices (New York HQ, Tel Aviv R&D, plus regional offices such as London and Singapore reported over time) is ⚠ — the current office list was not verifiable from primary pages this pass.

### 2.4 The Origin Story: Securing the Private Key

The origin story, as told by the company and repeated in press coverage, is a private-key problem statement. Through 2018–2019 the digital-asset industry suffered a string of high-profile exchange and custodian hacks in which **private keys stored on a single server or a single hardware wallet were stolen wholesale** (the Coincheck hack of January 2018, among others, is the era's canonical example). Fireblocks' founding insight was that **multi-party computation (MPC) could split the signing operation itself** so that no complete private key ever exists in one place — each party holds a share, and a threshold of shares jointly produces a signature without ever reconstructing the key (fireblocks.com/security; fireblocks.com/what-is-mpc). The company then productized this as custody infrastructure for institutions, launched the platform in June 2019, and grew through the 2020–2021 institutional wave into the settlement network and tokenization provider it is today.

Two details sharpen the story. First, the company's **earliest adopters were crypto-native institutions** — Galaxy Digital and Genesis (DCG) — before the bank wave (BNY Mellon, SCB 10X, and others) arrived with the 2021 rounds; the customer base's center of gravity moved from crypto-native to traditional-finance over 2020–2022 (Series B/C/D press). Second, the company's **research posture is part of the product**: its cryptography team has published technical security research (most notably the December 2022 BitGo threshold-signature vulnerability report, §10.2), which functions both as threat intelligence and as a marketing signal that the firm understands the cryptography it sells. This origin story is consistent across primary and secondary sources; the "founded to end private-key theft" framing is the company's own and is treated as its stated mission rather than an audited fact.

### 2.5 The Footprint Timeline

| Year | Event | Status |
| --- | --- | --- |
| 2018 | Company founded by Shaulov, Ofrat, Berengoltz; MPC custody concept developed | ✅ |
| Jun 2019 | Platform launch; Series A US$16M | ✅ |
| Nov 2020 | Series B US$30M (Paradigm); Network at 160+ participants; US$150B transferred | ✅ (round) / ⚠ (metrics) |
| Mar 2021 | Series C US$133M; BNY Mellon and SVB Capital strategics; US$400B transferred | ✅ (round) / ⚠ (metrics) |
| Jul 2021 | Series D US$310M at >US$2B valuation; SCB 10X becomes the third bank investor; US$1T transferred; 500+ businesses | ✅ (round) / ⚠ (metrics) |
| Feb 2022 | Series E US$550M at >US$8B valuation; 800+ institutions; Aave Arc institutional DeFi launch | ✅ (round) / ⚠ (metrics) |
| Dec 2022 | BitGo TSS vulnerability disclosed by Fireblocks' cryptography research team | ✅ (report) / ⚠ (severity) |
| 2023 | Contagious Interview / Lazarus recruiting-campaign threat reports; stablecoin-treasury customer wave (Reap-type) | ✅ (reports) |
| 2025–2026 | Payments and stablecoin expansion (MoneyGram, Western Union logos); 2,400+ enterprises, US$10T, 550M wallets claimed | ⚠ company-reported |
| Jan 2023 | Reap moves treasury and stablecoin repayments onto Fireblocks custody — the sibling-cluster custodian-wallet example | ✅ (Reap PR, cross-ref [Reap Global](reap_global_guide.md)) |
| Jan 2024 | Chainalysis screening integrated with the Fireblocks custody flow in Reap's stack | ✅ (Reap media chronology, cross-ref [Reap Global](reap_global_guide.md)) |
| 2025–2026 | Bitunix-type exchanges pair Cobo with Fireblocks for institutional custody | ✅ (Bitunix About page / H1 2026 blog, cross-ref [Bitunix](bitunix_guide.md)) |

### 2.6 The Company Table

| Aspect | Fact | Status |
| --- | --- | --- |
| Founded | 2018 (product launch June 2019) | ✅ |
| Founders | Michael Shaulov (CEO), Idan Ofrat (CTO), Pavel Berengoltz | ✅ names / ⚠ exact titles |
| Pre-Fireblocks background | Check Point Software Technologies (all three); Shaulov via Lacoon Mobile Security | ✅ |
| HQ | New York, USA | ✅ |
| R&D anchor | Israel (Tel Aviv), per press and team profiles | ⚠ |
| Legal entity | Not fully public; US entity assumed for HQ purposes | ⚠ |
| Earliest adopters | Galaxy Digital, Genesis Global Trading (DCG) — company's own words | ✅ |
| Platform launch | June 2019 | ✅ (Series B press release) |

---

## 3. The Mission and Business Model

### 3.1 The Mission Statement

Fireblocks' mission, as expressed across its own pages, is to **replatform the financial system onto digital assets** — to make it possible for "every business to become a crypto business" (Series E press release, February 2022) by providing "the industry's most scalable and secure platform" for stablecoin payments, settlement, custody, tokenization, and trading operations (About/press boilerplate, September 2026). The homepage tagline in 2026 is **"Infrastructure that powers financial possibility"** (fireblocks.com, extracted September 2026). ✅ as stated; the mission's *success* is of course ⚠ — the company's own metrics are examined in §4.7.

Two structural commitments recur in company materials and are worth recording because they define the business model:

1. **Institutions remain in full control of their assets** — the "direct custody" doctrine: Fireblocks positions its platform as enabling banks and fintechs to custody their own digital assets (using their own balance sheet and brand) rather than outsourcing to sub-custodians (Series C blog, March 2021; the "Introducing Direct Custody" whitepaper). This is the philosophical opposite of the sub-custody model and explains why banks, not just crypto-native firms, are the target market.
2. **Open access for all institutional players** — the Network is designed as a neutral, horizontal layer connecting exchanges, OTC desks, lenders, and banks (Series C blog). ✅ as stated company-side.

### 3.2 The Business Model — Platform, Network, and Software Revenue

Fireblocks does not publish a revenue breakdown, so the model below is reconstructed from public product structure and flagged ⚠ where inferential:

| Revenue stream | What it is | Status |
| --- | --- | --- |
| Platform subscription fees | Per-workspace/per-seat SaaS fees for the Fireblocks console, custody, policy engine, treasury management, and API access | ⚠ inferred from product structure; amounts not public |
| Network transaction fees | Fees for transfers/settlement over the Fireblocks Network (the "authenticated transfer channels" between institutions) | ⚠ inferred; pricing not public |
| Tokenization and software revenue | Tokenization platform licensing, smart-contract tooling, WaaS/Embedded-Wallet licensing to fintechs | ⚠ inferred; ABN AMRO and BNY Mellon-type engagements suggest enterprise licensing |
| Stablecoin/payments services | Stablecoin issuance, payments orchestration, and compliance tooling fees | ⚠ inferred |

The only public statement of scale is on the 2026 homepage: 2,400+ enterprises, US$10 trillion in transactions, 550 million wallets secured (⚠ company-reported). No revenue, EBITDA, or profitability figure has ever been published — the company is venture-backed and its unit economics are not public. ⚠

What the funding history implies about the model's economics is worth one paragraph. The rounds tell a story of capital deployed into R&D and customer success rather than into asset inventories or market making: Fireblocks repeatedly stated that new capital would fund "R&D resources that extend the platform's throughput, scalability, and performance" and customer-success headcount (Series D blog, July 2021). That is the profile of a **software-and-network business**, not a balance-sheet business — which is exactly what the "direct custody" doctrine requires, because the assets sit on the customers' balance sheets, not Fireblocks'. The commercial corollary: Fireblocks' revenue is subscription-and-fee based and therefore recurring in principle, but the 2022–2024 crypto winter tested whether customers would keep paying for infrastructure when their own volumes collapsed — a question the company's survival (and the 2026 homepage's growth claims) answers only in the company's own telling. ⚠ all of this is inference from public signals; no financial statements exist to confirm it.

### 3.3 The Model's Market Context

Fireblocks operates in the "institutional digital-asset infrastructure" layer, alongside custodians (Coinbase Custody, BitGo), hardware vendors (Ledger Enterprise), and other MPC platforms (Cobo, Copper, and, in the qualified-custody niche, Anchorage). Its differentiation claims are: (a) **MPC rather than hardware-only** key management; (b) the **Network**, which turns custody into a settlement graph among authenticated institutions (100% of the crypto-lending market and ~90% of crypto liquidity venues claimed as reachable in 2021 — ⚠ company claim, Series C blog); (c) **breadth** — custody, network, tokenization, payments, and DeFi on one platform, which is why the [Tokenized Assets](tokenized_assets_guide.md) guide treats it as the institutional standard; and (d) a **research and standards voice** — publishing security research on competitors (BitGo, §10.2) and threat intelligence (Lazarus, §10.3) positions Fireblocks as the security conscience of the industry, which is itself a commercial asset. The competitive and regulatory context — MiCA in the EU, the MAS DPT regime in Singapore, US state and federal money-transmitter rules — is developed in §6.7, §7, and §8.

---

## 4. Funding and Growth — The Rounds, Verified

### 4.1 The Funding Table

Every round below was verified at Fireblocks' own press page or blog and corroborated by reputable press (TechCrunch, Reuters/Yahoo Finance, Calcalist, CoinDesk). Amounts and investors are ✅ where the company's own press release states them; dates and valuations are ✅ where company press and press coverage agree, ⚠ where only one source class provides them.

| Round | Date | Amount | Lead / co-lead investors | Strategic / notable participation | Valuation | Status |
| --- | --- | --- | --- | --- | --- | --- |
| Series A | June 2019 | US$16 million | Cyberstarts, Tenaya Capital (per tracker and later company lists) | — | Not disclosed | ✅ amount (trackers + company's cumulative math); ⚠ lead attribution |
| Series B | November 2020 | US$30 million | Paradigm | Cyberstarts, Tenaya Capital, Swisscom, Galaxy Digital, DCG, Cedar Hill Capital | Not disclosed | ✅ (company press release) |
| Series C | March 2021 | US$133 million | Coatue, Ribbit, Stripes | BNY Mellon, SVB Capital | Not disclosed | ✅ (company blog) |
| Series D | July 2021 | US$310 million | Sequoia Capital, Stripes, Spark Capital, Coatue, DRW VC, SCB 10X | BNY Mellon, SVB Capital (earlier investors) | "over US$2 billion" (TechCrunch: US$2.2B) | ✅ (company press + TechCrunch/Calcalist) |
| Series E | February 2022 | US$550 million | D1 Capital Partners, Spark Capital | General Atlantic, Index Ventures, Mammoth, CapitalG (Alphabet), Altimeter, Iconiq, Canapi, Parafi | "over US$8 billion" | ✅ (company press + Reuters/CoinDesk) |

Cumulative: the Series D press release states **US$489 million** raised through July 2021; the Series E press release and trackers put the total at **just over US$1 billion (US$1.04 billion per Tracxn)**. ✅ company-stated milestones; ⚠ tracker-derived total.

### 4.2 Series A (June 2019): US$16 Million

The Series A was **US$16 million, announced June 2019** (Seedtable; gate.com's fundraising calendar dates it June 11, 2019; Crunchbase lists a Series A with MState and Eight Roads Ventures among participants). The research brief believed a "~US$30 million" seed/Series A backed by Cyberstarts and Tenaya — the **amount is refuted** (US$16 million, not US$30 million), while **Cyberstarts and Tenaya are confirmed** as early investors by the Series B press release's list of "existing investors" and the Series D press release's cumulative investor list. Whether a separate, smaller seed round preceded the Series A is ⚠ (Tracxn counts six rounds including one seed; Seedtable counts five) — the existence and size of a distinct seed round could not be confirmed at a primary source. A useful consistency check: the Series B press release states cumulative fundraising of US$46 million after the round, and US$16 million + US$30 million = US$46 million — so the Series A amount is corroborated by the company's own cumulative arithmetic. ✅/⚠

### 4.3 Series B (November 2020): US$30 Million, Led by Paradigm

Verified at the company's own press release (fireblocks.com/press/fireblocks-raises-30-million-in-series-b-funding-led-by-paradigm): "Fireblocks... announced today it has raised **$30 million in Series B funding, bringing its cumulative fundraising to $46 million**. This round was led by **Paradigm** with participation from existing investors, **Cyberstarts, Tenaya Capital, Swisscom, Galaxy Digital, Digital Currency Group (DCG), and Cedar Hill Capital**," with **Fred Ehrsam** (Paradigm co-founder and Coinbase co-founder) joining the board. The release also states the platform had transferred "over **$150 billion** in digital assets" since its June 2019 launch and that the **Fireblocks Network** counted 160+ active institutional participants (⚠ company metrics). The research brief believed a US$40 million Ribbit-led Series B — **both the amount and the lead are refuted**; Ribbit entered later, as a Series C co-lead (§4.4). ✅

### 4.4 Series C (March 2021): US$133 Million

Verified at the company's own blog (fireblocks.com/blog/133m-series-c-fireblocks-is-here-for-the-long-run): "Fireblocks has raised **$133 million in a Series C funding round led by Coatue, Ribbit, and Stripes** with strategic investment from **BNY Mellon and SVB Capital**," bringing cumulative funding to **US$179 million**. The post is datable to March 2021 and contains the primary-source founding confirmation quoted in §2.1, the "200+ customers" and "US$400 billion transferred" metrics (⚠ company-reported), and the Network claim of access to "100% of the crypto-lending market and ~90% of crypto liquidity venues" (⚠). The BNY Mellon investment is the first of the "three global banks" the Series D release later counts (BNY Mellon, SVB Capital, SCB 10X) — the bank-investor pattern is one of the clearest signals of Fireblocks' traditional-finance strategy. The research brief believed a US$133 million round with "Sequoia/others" — the **amount is confirmed**; the investor list is **corrected** (Coatue/Ribbit/Stripes led; Sequoia joined at the Series D). ✅/⚠

### 4.5 Series D (July 2021): US$310 Million at a US$2 Billion Valuation

Verified at the company's press release (fireblocks.com/press/fireblocks-solidifies-unicorn-status-with-310-million-series-d-at-2-billion-valuation) and the companion blog, dated July 27, 2021 (TechCrunch: "Crypto infrastructure provider Fireblocks raises $310M, triples valuation to $2.2B in five months"): **US$310 million, co-led by Sequoia Capital, Stripes, Spark Capital, Coatue, DRW VC and SCB 10X** — SCB 10X being the venture arm of Siam Commercial Bank, "the third global bank to invest in Fireblocks alongside BNY Mellon and SVB Capital" (✅ company-stated). Valuation: **"over $2 billion"** (company) / **US$2.2 billion** (TechCrunch). Cumulative: **US$489 million** (company). The release's investor roll-up confirms the earlier roster: "Cyberstarts, Eight Roads, Tenaya Capital, Swisscom Ventures, Paradigm, Ribbit Capital, and Coatue." The research brief believed a Sequoia-led US$310 million round — the **amount and Sequoia's presence are confirmed**; the "Sequoia-led" framing is refined to co-led with five others. The TechCrunch headline's "triples valuation in five months" also implies the Series C-era valuation was roughly US$700–800 million — an inference (⚠), not a company figure.

### 4.6 Series E (February 2022): US$550 Million at an US$8 Billion Valuation

Verified at the company's press release (fireblocks.com/press/fireblocks-raises-550-million-in-series-e-funding-to-become-the-highest-valued-digital-asset-infrastructure-provider) and corroborated by Reuters ("Crypto firm Fireblocks raises $550 million, company valued at $8 billion", February 2022) and CoinDesk: **US$550 million, co-led by D1 Capital Partners and Spark Capital**, with participation from **General Atlantic, Index Ventures, Mammoth, CapitalG (Alphabet's independent growth fund), Altimeter, Iconiq Strategic Partners, Canapi Ventures, and Parafi Capital** — at a valuation **over US$8 billion**, making Fireblocks "the highest valued digital asset infrastructure provider to date" (company). The release's supporting metrics: 800+ institutions, US$2 trillion transferred, 20+ blockchains, 1,000+ cryptocurrencies, US$45 billion in assets under custody on the platform, and the Aave Arc launch making Fireblocks "the first to enable decentralized finance for institutional players" (⚠ company-reported metrics). The D1 quote — "an estimated **15% of daily crypto transaction volume** secured through their infrastructure" (Dan Sundheim, D1 founder) — is an investor quote and is single-sourced (⚠). Some trackers date the round January 27, 2022 (Tracxn) versus the February press coverage — ⚠ single-day dating variance. The research brief's "D1 Capital-led, ~US$550M at ~US$8B" belief is **confirmed**, refined to co-led with Spark Capital. ✅

### 4.7 The Growth Metrics Over Time

All of the following are company-reported and therefore ⚠, but the trajectory is internally consistent and useful:

| Date (source) | Customers | Value transferred | Other |
| --- | --- | --- | --- |
| Nov 2020 (Series B press) | — | >US$150B | Network: 160+ participants |
| Mar 2021 (Series C blog) | 200+ | US$400B | "millions of wallets" |
| Jul 2021 (Series D press) | 500+ digital-asset businesses | US$1T | — |
| Feb 2022 (Series E press) | 800+ | US$2T | US$45B AUM on platform; 20+ chains; 1,000+ assets |
| Sep 2026 (homepage) | 2,400+ enterprises | US$10T | 550M wallets secured |
| 2023 (company claims, per §1.2) | 1,300+ institutions | — | — |

No third-party audit of these figures has been found; they are treated throughout as the company's own account of its scale. ⚠. The implied growth rate — from 200+ customers and US$400 billion transferred in March 2021 to 2,400+ enterprises and US$10 trillion in 2026 — is consistent with the institutional-stablecoin boom of 2023–2026, but every data point remains company-reported.

The **wallets-and-AUM trajectory** is the second dimension of the growth story, and it is where the WaaS/Embedded-Wallet lines (§5.4) show up in the numbers:

| Date (source) | Wallets / AUM | Notes |
| --- | --- | --- |
| Mar 2021 (Series C blog) | "millions of wallets" | ⚠ company-reported; no precise count published |
| Feb 2022 (Series E press) | US$45B assets under custody on the platform | ⚠ company-reported; the only AUM figure ever published |
| Sep 2026 (homepage) | 550M wallets secured | ⚠ company-reported; directionally consistent with consumer-facing WaaS deployments (Revolut-type) |

None of the wallet or AUM figures is audited; the trajectory is recorded because it is the company's own evidence that the platform grew from institutional custody into embedded-finance distribution — the 550M-wallet claim would be implausible for the 2021-era product surface but is at least coherent with the 2026-era WaaS footprint. ⚠ throughout.

Two readings of the customer-number trajectory are worth stating. The generous reading: Fireblocks rode the institutional wave it helped create — banks (BNY Mellon, ABN AMRO, BNP Paribas, HSBC per homepage logos), fintechs (Revolut, Bridge, Checkout.com), exchanges (Bitunix-type venues), and payments firms (Visa, Western Union, MoneyGram) all appear as public references, and the "550M wallets" figure is at least directionally consistent with consumer-facing WaaS deployments. The cautious reading: the metrics are unaudited, the "enterprises" definition is not disclosed (a demo workspace may count differently from a paying contract), and the US$10T cumulative-transactions figure compounds every transfer ever made through the platform — a throughput statistic, not a stock of assets. A banker should treat all of these as marketing-grade evidence and demand the audited artifacts (SOC 2 report, penetration-test summaries) through the Trust Center instead. ⚠ throughout.

### 4.8 What the Brief Got Wrong — Corrections

The research brief supplied believed figures that this pass verified, corrected, or refuted:

- **Believed: "seed/Series A (~2019, Cyberstarts/Tenaya, ~US$30M)"** → **Corrected**: Series A was **US$16 million in June 2019**; Cyberstarts and Tenaya are confirmed early investors; a distinct seed round's existence is ⚠. (§4.2)
- **Believed: "Series B (2020, Ribbit Capital, ~US$40M)"** → **Refuted**: Series B was **US$30 million led by Paradigm** (November 2020); Ribbit's first documented participation is the Series C co-lead. (§4.3)
- **Believed: "Series C (2021, ~US$133M, investors incl. Sequoia/others)"** → **Corrected**: amount confirmed; leads were **Coatue, Ribbit, Stripes** with BNY Mellon and SVB Capital as strategics; Sequoia appears at the Series D. (§4.4)
- **Believed: "Series D (2021, ~US$310M at ~US$2B valuation, Sequoia-led)"** → **Confirmed, refined**: amount, valuation and Sequoia confirmed; the round was **co-led by six investors**. (§4.5)
- **Believed: "Series E (2022, ~US$550M at ~US$8B valuation, D1 Capital-led)"** → **Confirmed, refined**: co-led by **D1 Capital Partners and Spark Capital**. (§4.6)

---

## 5. The Products — MPC Custody and the Wallet Infrastructure

### 5.1 Fireblocks Core / Workspace and the Asset-Account Model

The custody product, historically branded **Fireblocks Core** and now marketed as the workspace/console, organizes a customer's digital-asset operations into a **workspace** containing **vaults and asset accounts** — logically separated buckets (e.g., per product line, per client, per business unit) each with its own policies and approval workflows (fireblocks.com platform pages and developer documentation; product mechanics ✅ as documented, details ⚠ where docs were not reachable). The workspace model is the unit of governance: policies attach to accounts, and the console is where users initiate transactions, which then route through the policy engine (§5.3) and the mobile approval flow before any signature is produced. For a bank, the workspace is conceptually the digital-asset analogue of a payment-operations hierarchy: accounts map to books, users map to roles, and policies map to the bank's own approval matrices.

### 5.2 The MPC Wallet and Threshold Signatures

The core technology claim — verified at the company's security page and its "what is MPC" explainer — is that Fireblocks uses **multi-party computation (MPC)** to generate and use keys such that **the private key never exists in its entirety in any single location**: key shares are distributed across environments (Fireblocks' servers, the customer's environment, and approval devices), and a **threshold of shares** jointly computes a signature without ever reconstructing the key (fireblocks.com/security: "No single point of private key compromise with key shares distributed across environments"; "Standard-setting MPC key management... eliminating single points of failure in key storage and transaction signing"). ✅ as company-stated; the cryptographic family (threshold ECDSA for bitcoin/ethereum-style chains, GGN18-style protocols per the broader literature) is ⚠ at the level of specific scheme names, which the company's public pages do not enumerate in this pass. The on-chain mechanics this builds on — ECDSA, addresses, transaction broadcasting — are the subject of the [Smart Contracts](../technology/smart_contracts_guide.md) guide, cross-referenced, condensed, not re-derived.

### 5.3 The Policy Engine and Governance

The **Policy Engine** is Fireblocks' control plane: "Control exactly how digital assets move with programmable policies and workflows... From multi-party approval requirements to transaction limits and custom rules" (fireblocks.com/security). Key mechanics, per the same page: **multi-device approval** — transactions initiated from the web console must be approved on mobile devices using trusted execution environments (TEE), biometric authentication (Face ID/PIN), and system integrity checks, so "approvers understand and consciously authorize each transaction"; and the **Enterprise Policy Governance Engine** — "Granular controls prevent internal collusion, ensuring approval and signing quorums are enforced for all transactions." ✅ as company-stated. This is the mechanism behind the recurring industry claim that **Fireblocks itself cannot move a customer's funds** — the customer's own approvers must co-sign under the customer's policy — which is treated here as a platform-design claim that is structurally consistent with the documented architecture (✅ architecture / ⚠ absolute framing; see §9.3).

For a bank, the policy engine is where the custody product becomes a *controls* product rather than a *cryptography* product: the same engine that enforces transaction limits and quorums can encode the bank's own risk appetite (per-currency exposure caps, counterparty limits, time-of-day restrictions, dual-control over policy changes themselves). The audit trail the engine produces — who initiated, who approved, which policy matched, what the signature ceremony did — is the evidence the bank's internal audit and its MAS-mandated record-keeping will cite, so policy design is best treated as a governance exercise with the bank's compliance team in the room from day one (extended in §11.2).

### 5.4 Wallet-as-a-Service and Embedded Wallets

Two product lines extend custody into fintech distribution (fireblocks.com/products): **Wallet-as-a-Service** ("Protect, scale and optimize your crypto wallets") — white-label wallet infrastructure that lets fintechs offer crypto to end customers without building key management; and **Embedded Wallets** ("Create white-labelled wallets for consumer and retail apps"). Both are documented product pages ✅; commercial terms and the customer split between them are not public (⚠). These lines are what make the "wallets secured: 550M" metric plausible as a platform footprint (⚠ company-reported) — Revolut's consumer crypto offering, for instance, is the archetype of the WaaS distribution model.

### 5.5 Treasury Management and the API Platform

**Treasury Management** (fireblocks.com/products/treasury-management) is the operations layer — "Secure, control and automate operations for your digital assets" — covering balance management, transfers, staking, and reporting across the workspace. The **API platform** (developers.fireblocks.com) exposes the same primitives programmatically: workspace management, vault accounts, transaction creation and signing, webhooks, and the Network APIs; the docs are the authoritative reference for the co-signing model (✅ product exists; ⚠ this pass could not fully page the developer docs, so API specifics are summarized conservatively). The API-first design is what makes Fireblocks embeddable — the [Reap](reap_global_guide.md) guide's stablecoin treasury runs on exactly this kind of integration, and the [Tokenized Assets](tokenized_assets_guide.md) guide's custody section assumes the same pattern.

---

## 6. The Products — The Fireblocks Network, Tokenization, Payments, and Staking

### 6.1 The Fireblocks Network — Settlement Infrastructure

The **Fireblocks Network** is the company's settlement layer: an authenticated network over which member institutions transfer digital assets among themselves. Company descriptions across the press and product pages record: "an institutional asset transfer system... the only secure global network for crypto market participants and intermediaries to connect and settle (both on and off-chain)" (Series C blog, March 2021); "Trusted by 2,400 fintechs, retail apps, PSPs, banks, exchanges and institutional traders" (Network page, 2026 — ⚠ company metric); "Transact with **verified counterparties, not just wallet addresses**, to eliminate settlement risks" (Network page — ✅ the design property); and "The only network with **patented technology** for securing crypto asset transactions" (Network page — ⚠ company claim). The 2022 press recorded Network members including BNY Mellon, Revolut, Galaxy Digital, Crypto.com, BlockFi, Deribit, eToro, CoinShares, SwissBorg, 3AC, and B2C2 (Series E press, February 2022 — note 3AC's subsequent collapse in 2022 is a reminder that Network membership is not a credit endorsement).

Two distinctive Network properties matter for a banking reader (cross-ref [Payment Rails](payment_rails_guide.md) for the fiat-side clearing mechanics this complements):

1. **Counterparty-risk mitigation via authenticated channels**: because both ends of a transfer are known Network members, destination-address risk ("am I sending to the right wallet?") is structurally reduced — the security page's "Secure Transfer Environment... completely mitigates the security risks associated with destination addresses on the Fireblocks Network." ✅ as designed.
2. **Settlement finality**: Network transfers settle on-chain (the Network is a coordination layer, not a separate ledger — the transfer still lands as a blockchain transaction) or through off-exchange arrangements (§6.2). ⚠ the "instant settlement" framing in company materials is marketing shorthand for the speed of the orchestration layer.

### 6.2 Off-Exchange Settlement and Collateral Management

**Off-exchange settlement** is a documented Fireblocks product with its own page (fireblocks.com/platforms/off-exchange): "Keep principal in **MPC-secured shared wallets** and **mirror collateral to supported exchanges** for instant spot, margin, and derivatives execution. Exchanges get **onchain proof of full collateralization**, and you can shift capital across venues as opportunities arise." The mechanism: a trading venue and its clients hold assets under joint MPC policy control so that assets do not need to move on-chain for every trade, reducing transfer costs and hot-wallet exposure. ✅ the product is documented at the primary source; ✅ third-party corroboration exists (Ethena's documentation lists Fireblocks among its off-exchange settlement providers alongside Copper and Ceffu; The Paypers reported Bitget's 2025 integration of Fireblocks for off-exchange institutional trading). ⚠ the failure-mode and legal-entity details of off-exchange structures (what happens to mirrored collateral if the venue fails) are not fully public — and the 3AC/Bitget-era history of exchange credit risk is the banker's cautionary cross-reference. **Collateral management** — using digital assets as collateral in lending and margin flows — is part of the same Network story ("access to 100% of the crypto-lending market" claim, Series C blog). The mechanics of settlement finality and clearing are cross-referenced, condensed, to [Payment Rails](payment_rails_guide.md) rather than re-derived here.

The risk framing a bank must keep: off-exchange settlement is a **credit-engineering trade**, not a security feature. The structure removes on-chain transfer risk and hot-wallet concentration, but it substitutes venue credit risk (the mirrored collateral sits under the venue's operational control) and legal risk (whose balance sheet holds the collateral in a venue default, and which jurisdiction's insolvency law applies). The 2022 industry history — venues that froze or lost client assets — is the cautionary tale, and the Fireblocks-style answer (on-chain proof of collateralization) only proves the collateral *exists*, not that it is *reachable* in a default. This is precisely the kind of arrangement a bank's credit and legal teams must price and document before treasury adopts it.

### 6.3 Tokenization — the Tokenization Platform

The **Tokenization** product (fireblocks.com/products/tokenization) covers "tokenize assets and manage smart contract operations" — issuance tooling, mint/burn controls, and smart-contract lifecycle management for tokenized securities and RWAs. The flagship customer reference is **ABN AMRO**, described on the homepage as "Europe's first major bank to issue tokenized bonds by leveraging Fireblocks' platform to tokenize, issue, and custody digital securities." ✅ the product and the ABN AMRO reference exist as published. The research brief's shorthand "FTS" (Fireblocks Tokenization Solution) did **not** verify at a primary source this pass (❌/⚠ — see §12.3), and the tokenization platform's deep mechanics — issuance standards, registry models, custody of the issuer key — are cross-referenced to the [Tokenized Assets](tokenized_assets_guide.md) guide rather than re-derived. WisdomTree's product head is quoted on the Network page praising cross-ecosystem connectivity — a further signal of the tokenized-funds use case (⚠ customer-quote depth).

### 6.4 Stablecoin and Payments Infrastructure

The **Stablecoin infrastructure** line (fireblocks.com/solutions/stablecoin-infrastructure) covers holding, moving, issuing, and managing stablecoins "with built-in compliance," and the **Payments** platform (fireblocks.com/platforms/payments) orchestrates "global stablecoin payments across any blockchain, payment rail and currency" — with the Network page quantifying the payments reach as "40+ providers, 60+ currencies, 100+ countries, and thousands of tokens" (⚠ company metric). Customer references include Bridge ("cut bulk settlement times from 12+ hours to under 90 minutes... millions of stablecoin transactions" — ⚠ customer-story metrics), MoneyGram and Western Union (2026 homepage logos), and Revolut. This is the product line behind the [Reap](reap_global_guide.md) cross-reference — Reap's January 2023 announcement that it moved treasury and stablecoin repayments onto Fireblocks custody — and behind the sibling [Payment Rails](payment_rails_guide.md) guide's stablecoin-settlement themes. ✅ product exists; ⚠ all volume/efficiency figures are company/customer-published.

The stablecoin angle also explains Fireblocks' post-2023 growth narrative: as regulated stablecoin regimes matured (the EU's MiCA and the various stablecoin laws that followed), issuers and distributors needed infrastructure that could mint, hold, move, and burn tokens under policy control — and Fireblocks' custody-plus-compliance bundle is the turnkey answer the homepage's "Stablecoin infrastructure" line sells. The "built-in compliance" claim (screening, travel rule, KYC tooling embedded in the transfer flow, §7.3) is the differentiator versus plain wallet infrastructure, and it is the reason the [Tokenized Assets](tokenized_assets_guide.md) guide treats stablecoin infrastructure and Fireblocks as effectively synonymous in its custody section. ⚠ the claim is company-stated; the regulatory-readiness depth of the compliance tooling is not independently audited this pass.

### 6.5 Staking and Institutional DeFi

Fireblocks' **DeFi** product line (fireblocks.com/platforms/defi) provides "full-stack protection for DeFi operations": granular policy controls, smart-contract safeguards (whitelisted interfaces, token-approval monitoring and revocation), and real-time risk tracking. The **DeFi Security Suite** and the 2022 Aave Arc launch ("Fireblocks became the first to enable decentralized finance for institutional players" — a permissioned DeFi pool where Fireblocks policy controls governed participation) anchor the staking/DeFi story (Series E press). ⚠ the Aave Arc program itself has since wound down per industry reporting — the launch fact is ✅, its current status is ⚠. Staking is offered as part of treasury management; yields and supported networks are time-varying and not audited here (⚠).

### 6.6 The 2026 Product-Surface Map

| Product line | 2026 homepage positioning | Status |
| --- | --- | --- |
| Payments | "Enable faster, safer global stablecoin payments" | ✅ |
| Treasury Management | "Secure, control and automate operations for your digital assets" | ✅ |
| Wallet-as-a-Service | "Protect, scale and optimize your crypto wallets" | ✅ |
| Tokenization | "Tokenize assets and manage smart contract operations" | ✅ |
| Embedded Wallets | "Create white-labelled wallets for consumer and retail apps" | ✅ |
| Fireblocks Network | "Connect directly to the entire digital asset ecosystem" | ✅ |
| MPC custody (Core/Workspace) | Core of the security stack ("Standard-setting MPC key management") | ✅ |
| DeFi / Staking | DeFi Security Suite, policy-governed DeFi access | ✅ |

### 6.7 The Competitive Landscape

Fireblocks competes in three overlapping markets — custody/key management, settlement networks, and tokenization tooling — and the sibling guides supply the comparison points:

| Competitor class | Examples | How they differ from Fireblocks | Cross-ref |
| --- | --- | --- | --- |
| MPC custody platforms | Cobo, Copper (ClearLoop) | Same cryptographic family; Cobo is the other half of the Bitunix custody pairing; Copper's ClearLoop competes directly on off-exchange settlement | [Bitunix](bitunix_guide.md) |
| TSS/qualified custodians | BitGo, Coinbase Custody, Anchorage | BitGo is the rival whose TSS Fireblocks' researchers audited in 2022; Coinbase/Anchorage hold US charters Fireblocks does not claim | §10.2 |
| Settlement networks | Exchange-internal settlement, OTC networks, fiat clearing | The Network's verified-counterparty model competes with and complements traditional clearing | [Payment Rails](payment_rails_guide.md) |
| Tokenization platforms | Issuance platforms in the RWA space | Fireblocks bundles custody + network + tokenization; flagship reference is ABN AMRO's tokenized bonds | [Tokenized Assets](tokenized_assets_guide.md) |

The honest competitive summary for a bank: Fireblocks is the deepest *infrastructure-only* stack (no trading book, no consumer brand, no bank charter), which is both its pitch and its limitation — it must win as a neutral layer while rivals bundle custody with exchange, lending, or charter advantages. ⚠ comparative market share is not public; the table records positioning, not rankings.

The products-versus-competitors view, grounded only in what this guide and its siblings already state (the [Tokenized Assets](tokenized_assets_guide.md) guide's custody-competition material supplies the bank-platform entries):

| Competitor | Segment | How it compares with Fireblocks | Grounding |
| --- | --- | --- | --- |
| BitGo | TSS custody / qualified custody | The rival whose TSS wallets Fireblocks' researchers audited in 2022 (the missing-zero-knowledge-proof finding); BitGo Prime adds lending and trading around custody | §10.2; [Tokenized Assets](tokenized_assets_guide.md) |
| Copper (ClearLoop) | MPC custody / off-exchange settlement | Same cryptographic family; ClearLoop is the direct competitor on off-exchange settlement, listed alongside Fireblocks among Ethena's off-exchange providers | §6.2, §6.7 |
| Ceffu | Exchange-linked custody / off-exchange | Also listed among Ethena's off-exchange settlement providers — the exchange-affiliated custody model versus Fireblocks' neutral layer | §6.2 |
| Taurus | Bank-grade tokenization + custody (Swiss) | The bank-platform alternative in the tokenization race — used by Deutsche Bank per the sibling guide (⚠) — competing on the ABN AMRO-type use case | [Tokenized Assets](tokenized_assets_guide.md) |
| Metaco (Ripple) | Custody / issuance orchestration | Acquired by Ripple (2023); the custody-ops platform banks license (BNP Paribas-linked per the sibling guide, ⚠) versus Fireblocks' custody + network + tokenization bundle | [Tokenized Assets](tokenized_assets_guide.md) |
| Coinbase Custody / Coinbase Prime | Regulated custody + exchange | The ETF-custody default per the sibling guide, with a US charter and an exchange behind it — the bundled model Fireblocks positions its neutral layer against | §3.3, §6.7; [Tokenized Assets](tokenized_assets_guide.md) |

---

## 7. Licensing and Compliance

### 7.1 The Verified Registrations and Licenses

The honest headline for this section is that **Fireblocks' licensing posture could not be fully verified from primary regulator registers this pass** (interactive registry lookups were not available), and the company's own pages do not publish a definitive license list — the company instead operates a **Trust Center** (trust.fireblocks.com, powered by SafeBase) where security and compliance artifacts are made available to customers and prospects under NDA. ✅ the Trust Center exists; ⚠ its contents were not accessible without registration this pass. What can be stated:

- **US federal**: Fireblocks is widely described in press and industry coverage as holding FinCEN **money-services-business (MSB) registrations** appropriate to a digital-asset custodian/transmitter, and as operating state money-transmitter arrangements where its business model requires them. ⚠ — no FinCEN registry query was possible this pass; treat as press-consistent but unverified.
- **New York**: Fireblocks is **not** publicly listed among BitLicense holders in coverage examined; its New York posture (whether it operates under a limited-purpose trust charter, a BitLicense, or an exemption) is **not verified** (⚠/❌ not found this pass).
- **EU/UK/Swiss**: Industry directories and press have, at various times, associated Fireblocks with **FCA registration in the UK**, **Lithuanian VASP-type registration**, and **Swiss VQF self-regulation** membership for its European entities. None of these could be confirmed on regulator registers this pass → all ⚠, and the company's current entity map is explicitly listed in §12.4 as unverified.
- **Company posture**: Fireblocks' own framing is that it provides **infrastructure to licensed entities** rather than positioning itself as a licensed custodian in every market — its customers (banks, exchanges, PSPs) hold the licenses and Fireblocks is the technology layer beneath them. That framing is consistent with the "direct custody" doctrine (§3.1) and is ✅ as the company's stated positioning, ⚠ as a legal characterization.

### 7.2 The Unverified Licensing Claims

Claims that appear in secondary sources but could not be verified at primary sources this pass: specific state money-transmitter licenses; a UK FCA cryptoasset registration number; a Lithuanian VASP registration; Swiss VQF membership; and any Australian or Canadian registration. None are asserted here; each is flagged ⚠ and carried to §12.4. ❌/⚠

### 7.3 The KYC/AML and Screening Posture (Cross-Referenced, Condensed)

Fireblocks' compliance tooling (documented on its product pages) includes **transaction screening and sanctions screening integrated into the transfer flow** — the 2026 security page lists "Transaction Scanning & DeFi Threat Detection" and the stablecoin line markets "built-in compliance," and the Network page states "With built-in **AML, KYC, Travel Rule**, and more, compliance is built-in to every transaction" (✅ as company-stated). The company has long offered **travel-rule tooling** (the FATF travel rule for VASP-to-VASP transfers) as part of its platform and integrates with third-party screening vendors — the [Reap](reap_global_guide.md) guide's January 2024 item (Chainalysis screening integrated with Fireblocks custody) is the sibling-cluster example. These themes — sanctions screening, transaction monitoring, travel-rule data sharing — are cross-referenced, condensed, to the [Fircosoft](fircosoft_guide.md) guide's AML/screening treatment rather than re-derived here. ✅ product pages document the features; ⚠ effectiveness and coverage are not auditable this pass.

### 7.4 The Licensing Table

| Jurisdiction / regime | Finding | Status |
| --- | --- | --- |
| US federal (FinCEN MSB) | Press-consistent; registry not queryable this pass | ⚠ |
| US state money-transmitter licenses | Individual licenses not verified | ⚠ |
| New York BitLicense | No public evidence found | ❌ not found |
| UK FCA registration | Reported in industry coverage; not register-verified | ⚠ |
| EU (Lithuania-type VASP) | Reported; not register-verified | ⚠ |
| Switzerland (VQF) | Reported; not register-verified | ⚠ |
| Singapore (MAS) | See §8 | ⚠/❌ |
| Company positioning | Infrastructure to licensed customers, not a licensed custodian everywhere | ✅ as stated / ⚠ as law |

---

## 8. The Singapore Angle — MAS and the DPT Regime (Cross-Referenced)

### 8.1 The MAS Regime in One Paragraph

Singapore's Payment Services Act (PS Act) regulates digital payment token (DPT) services — dealing, custody, and transmission of DPTs — through a licensing regime administered by MAS, with a transitional no-approval-required regime that ended, and a final regime in which DPT service providers must hold a **Major Payment Institution (MPI) licence** or a standard PI licence, plus comply with MAS's AML/CFT notices and, since 2022, its DPT-specific consumer-access measures. The regime's details are not re-derived here — they are cross-referenced, condensed, to the [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) guide and the [Tokenized Assets](tokenized_assets_guide.md) guide's §13 Singapore deep-dive.

Two regime features shape everything that follows. First, **custody is itself a regulated activity** under the PS Act when provided as a DPT service — so a firm that merely holds tokens for customers in Singapore may need a licence even if it never trades; this is the provision that makes the "infrastructure, not licensed custodian" boundary (§7.1, §8.2) legally load-bearing rather than semantic. Second, the **transitional-regime history** means that firms which operated during the grace period, or which applied and were never granted, are not visible as licensees today — which is why Fireblocks' absence from the current register (if confirmed) would not by itself prove it never sought entry. Both features are the MAS guide's material, condensed here for the Fireblocks-specific discussion in §8.2.

### 8.2 Fireblocks in Singapore — What Is and Is Not Known

- Fireblocks has, per industry reporting, maintained a **Singapore presence** (office/team) as part of its APAC expansion, and has participated in Singapore ecosystem events — including, per industry coverage, involvement around **MAS' Project Guardian** tokenization initiatives, where infrastructure providers have participated in pilots. ⚠ — participation specifics were not verified at a primary source this pass.
- **No MAS licence or registration for Fireblocks was found** in coverage examined; the MAS Financial Institutions Directory could not be queried directly, so this is a "not found this pass" (❌/⚠) rather than a proof of absence.
- The consistent reading, consistent with the company's global positioning (§7.1), is that Fireblocks operates as **infrastructure to MAS-licensed customers** rather than as a licensed DPT provider itself — but that reading is inferential and is flagged ⚠. The sibling [Bitunix](bitunix_guide.md) guide's Singapore restriction (no Singapore-directed services) and the [Reap](reap_global_guide.md) guide's MAS-exemption analysis are the closest comparable treatments of the "operates near but not inside the MAS regime" pattern.

The comparative point worth making: in the Singapore ecosystem, the *licensed* custodians and DPT providers (the bank-owned or exchange-owned names on the MAS register) are typically Fireblocks' *customers*, while Fireblocks itself sits outside the register — the same infrastructure-versus-license boundary that holds in every other jurisdiction examined in §7. That division of labor is stable in the industry as a whole, but it is exactly the kind of arrangement MAS' outsourcing and technology-risk expectations scrutinize when a regulated bank (Cymbal Bank) puts material custody operations on a foreign vendor's platform (§8.3, §11.5). ⚠ the characterization is inferential; the MAS register was not queryable this pass.

### 8.3 What the Regime Means for a Fireblocks-Style Custodian

For Cymbal Bank's own MAS-supervised planning (§11.5), the Singapore angle resolves into three questions: (1) does Cymbal Bank's use of a Fireblocks-style vendor shift any regulated activity to the vendor (if the vendor is not licensed in Singapore, the licensed bank remains the regulated party — the vendor is a technology supplier, which is exactly the "infrastructure, not licensed custodian" model §7.1 describes); (2) what does the bank's own AML/CFT notice require of the custody flow (travel-rule data, screening, record-keeping — cross-ref [Fircosoft](fircosoft_guide.md)); and (3) how do MAS' outsourcing and technology-risk guidelines apply to a foreign infrastructure vendor with key materiality (vendor due diligence, data residency, business continuity). ⚠ the application of specific MAS notices to a custody-vendor arrangement is a legal-interpretation question the repository does not resolve; it is flagged and left to counsel.

---

## 9. Security and Technology — The MPC Architecture

### 9.1 The Threshold-Signature Model (Cross-Referenced, Condensed)

The on-chain mechanics that MPC signing rests on — ECDSA signatures, addresses, transaction broadcasting, Merkle proofs — are the subject of the [Smart Contracts](../technology/smart_contracts_guide.md) guide and are cross-referenced, condensed, not re-derived here. The one paragraph a banker needs: a blockchain transaction must be signed with a private key; whoever holds that key controls the funds; the entire custody industry is a contest over where that key lives. Fireblocks' answer is that the key never fully exists — a **threshold signature** is computed jointly by parties holding **shares**, and the resulting signature is indistinguishable on-chain from a single-key signature. ✅ the concept is standard cryptography; ✅ the company documents MPC-based signing; ⚠ scheme-level specifics (which MPC protocol, which threshold parameters) are not enumerated on the public pages examined.

Why this matters architecturally: in a single-key model, security is a *storage* problem (put the key somewhere safe), and the hot/cold spectrum — online hot wallets versus air-gapped cold storage — is the classic answer (cross-ref [Cybersecurity](../technology/cybersecurity_guide.md), condensed). In a threshold model, security becomes a *distribution and governance* problem: the key material never has a single location to steal, but the parties holding shares, the devices running the protocol, and the policies governing when signing may happen become the attack surface. That shift — from "where is the key?" to "who may command the shares, and under what rules?" — is the conceptual bridge every banker must cross to evaluate MPC custody seriously, and it is why §9.3 (the policy engine) and §10.3 (the human layer) matter as much as the cryptography itself.

### 9.2 The Distributed Key-Share Architecture

Per fireblocks.com/security: "**Distributed Wallet Infrastructure** — No single point of private key compromise with key shares distributed across environments," and "**Zero-Trust Architecture** — Critical operations such as sensitive code execution and hardware key management performed inside secure enclaves." The operational model: key shares live in multiple environments (Fireblocks' infrastructure, the customer's environment, and approval devices), and signing requires a threshold of shares to participate — so the compromise of any single environment does not yield a usable key. ✅ as company-stated; the "no single party holds the key" design is the platform's central security claim, and it is treated as architecture ✅ with the caveat that the *governance* of who may command the shares is the customer's policy problem (§9.3). The cybersecurity-guide themes this touches — hot/cold separation, multi-signature — map imperfectly onto threshold custody (there is no single "cold key" to air-gap when the key does not exist), which is a nuance the [Cybersecurity](../technology/cybersecurity_guide.md) guide's framework should be read alongside (⚠).

### 9.3 The Policy Engine, Co-Signing, and the No-Single-Party Claim

The claim that **Fireblocks itself cannot move customer funds** is the most important security statement to evaluate carefully. Structurally: transactions are initiated by the customer's operators, must pass the customer's policy-engine rules (limits, approvers, quorums), and must be approved on the customer's mobile devices (TEE-backed, biometric-bound) before the MPC signing protocol runs — so a Fireblocks insider alone could not produce a valid signature without the customer's approvers. ✅ the architecture is consistent with the claim as a design property. Two caveats: (a) it is a platform-design claim, not an audited guarantee — the January 2022 episode (§10.1) showed that even infrastructure vendors can have API-level weaknesses; and (b) "Fireblocks cannot move funds" is about *keys*, not about *access* — a customer whose own approvers are socially engineered (the Lazarus campaigns, §10.3) is still exposed through the customer's own side of the quorum. ⚠ absolute framings are flagged; the design property itself is ✅.

### 9.4 HSMs, TEEs, and the Zero-Trust Environment

The security page describes defense-in-depth layers: **hardware security modules (HSMs)** and **secure enclaves** for hardware key management; **trusted execution environments (TEE)** on approval devices with biometric authentication and system-integrity checks; a **24/7 Security Operations Center**; **transaction simulation and DeFi threat detection** for malicious-contract protection; and the **Fireblocks Security Posture Management (FSPM)** tool for configuration risk visibility. ✅ as company-stated; the depth of each control is not independently testable this pass (⚠).

The architectural reading a security architect should take away: Fireblocks blends two traditionally separate worlds. The **key-material world** borrows from hardware security — HSMs and enclaves protect the shares that do exist at rest, and the zero-trust posture means sensitive code runs in isolated environments. The **authorization world** borrows from enterprise identity — the policy engine, multi-device approval, and biometric binding are identity-and-access-management concepts applied to transaction signing. The fusion is the product's real innovation: a bank that already runs IAM discipline for its payment systems can recognize most of the control vocabulary, even though the underlying cryptography (threshold signatures) is unfamiliar. ⚠ the security page's claims are company-published; the fusion reading is this guide's analysis, not a company statement.

### 9.5 Audits, Certifications, and the Bug-Bounty Program

The security page states: "Fireblocks maintains **SOC 2 Type 2 certification (zero material findings)**, **ISO 27001** and related certifications, and is the world's first **C4 CCSS QSP Level 3** certified company," plus regular penetration testing by "world-class providers" and "**NCC-audited cryptographic libraries**" (⚠ company-published). The house convention applies: SOC 2 Type 2 and ISO 27001 are third-party-audited standards, so the *certificates* are independently attestable in principle — but the company's claim of holding them is company-published this pass (⚠, with the note that SOC 2/ISO attestation reports are standard vendor-diligence artifacts a bank can request via the Trust Center). The **CCSS QSP Level 3** claim (Cryptocurrency Certification Consortium's security standard) is unusual and ⚠ single-sourced to the company. Separately, Fireblocks runs a **managed bug-bounty program on Bugcrowd** (bugcrowd.com/engagements/fireblocks-mbb-og) — ✅ the engagement's existence verifies at the Bugcrowd platform, and it is consistent with the cybersecurity-guide theme that bug bounties are a standard institutional-security signal (cross-ref [Cybersecurity](../technology/cybersecurity_guide.md), condensed). The smart-contracts guide's audit-practice and incident-history treatment is cross-referenced from [Smart Contracts](../technology/smart_contracts_guide.md).

### 9.6 The Verified Technology Facts Table

| Fact | Verdict |
| --- | --- |
| MPC-based key management; private key never assembled in one place | ✅ (company docs + standard cryptography) |
| Threshold signing; key shares across environments | ✅ (company docs) |
| Policy engine with quorums and multi-device approval | ✅ (company docs) |
| TEE/biometric-bound approval devices | ✅ (company docs) |
| HSM and secure-enclave integration | ✅ (company docs) |
| "Fireblocks cannot move customer funds" | ✅ as design property / ⚠ as absolute guarantee |
| SOC 2 Type 2, ISO 27001, CCSS QSP Level 3 | ⚠ company-published (third-party auditable in principle) |
| NCC-audited cryptographic libraries | ⚠ company-published |
| 24/7 SOC, penetration testing program | ⚠ company-published |
| Bugcrowd managed bug-bounty engagement | ✅ (Bugcrowd platform listing) |

---

## 10. Security Incidents and the Threat-Intelligence Record

### 10.1 January 2022 — The API-Disclosure Episode

In January 2022, an independent security researcher publicly disclosed a vulnerability class involving Fireblocks' **admin/API infrastructure**; the reporting at the time described a researcher demonstrating that certain Fireblocks-internal admin capabilities could be reached, or that a design flaw in the platform's API layer existed. Fireblocks responded that the issue had been **remediated before disclosure, that no customer funds were lost, and that no customer environments were compromised**, disputing elements of the researcher's framing; the researcher maintained his characterization. The episode is ⚠ **contested** in its details (the exact technical scope and severity were never jointly confirmed), and — critically for the record — **no primary source states that Fireblocks itself was hacked or that any customer lost funds** as a result. ✅ the episode occurred and was publicly disputed; ⚠ the technical details; ❌ the "Fireblocks hacked" framing, which no primary source supports. The incident-history context (how custody vendors' incidents are assessed) is cross-referenced to [Smart Contracts](../technology/smart_contracts_guide.md).

How a due-diligence reader should weight the episode: it is the standard pattern of an infrastructure-vendor disclosure dispute — a researcher finds something, the vendor says "already fixed / not exploitable / no impact," and the record preserves both versions. The absence of any customer-loss claim, then or since, is the material fact; the episode's real lesson is procedural — that Fireblocks' *admin and API surface* is part of its attack surface, which is why the policy engine, device controls, and the bug-bounty program (§9.5) exist. A bank's vendor questionnaire should ask for the post-incident remediation evidence (penetration-test summaries, the SOC 2 report covering the period) rather than treating either the researcher's or the company's framing as settled fact. ⚠ as contested.

### 10.2 December 2022 — The BitGo TSS Vulnerability Research

In December 2022, Fireblocks' **cryptography research team** published a technical report (fireblocks.com/blog/bitgo-wallet-zero-proof-vulnerability; press release "Fireblocks Researchers Uncover BitGo Wallet Vulnerability") claiming to have found a critical vulnerability in **BitGo's threshold-signature-scheme (TSS) wallets**: per Fireblocks, the absence of a mandatory **zero-knowledge proof** in the TSS protocol's key-generation flow "allows an attacker to steal the private key share used by the client, regardless of their key storage methods and security measures," with the team verifying the issue "on the BitGo REST API and backend for both testnet and mainnet accounts" on December 5, 2022. BitGo responded publicly at the time; the episode is recorded here as **Fireblocks' account of a competitor's vulnerability** (✅ the report exists and was published by Fireblocks; ⚠ the severity dispute between the two vendors is not resolved here). For a banker, the episode matters for two reasons: it demonstrates that MPC/threshold-signature implementations are *implementations* — subtle protocol flaws are possible and must be audited (cross-ref the smart-contracts guide's audit-practice treatment) — and it shows Fireblocks using public research as a competitive and marketing instrument, which a vendor-diligence reader should weigh accordingly.

### 10.3 2023–2025 — The Lazarus Group Phishing Campaigns

Fireblocks has published its own **threat reports** documenting North-Korean-linked (Lazarus Group) social-engineering campaigns against crypto firms — most notably the "**Contagious Interview**" pattern: fake recruiting/impersonation campaigns in which attackers pose as recruiters or employees, distribute trojanized applications (macOS and Windows), and target **employees of crypto companies — including firms using Fireblocks' platform** — with the goal of stealing credentials and signing capabilities (fireblocks.com/blog/contagious-interview-recruiting-scam, "Disrupting a Recruiting Impersonation Scam: Anatomy of Operation Contagious Interview"). The March 2023 wave of Lazarus recruiting-themed phishing against crypto firms is independently corroborated by NCC Group's research ("How the Lazarus Group Targets Fintech") and industry reporting. Fireblocks' public threat intelligence repeatedly warned that these campaigns target the *human* layer — the same approvers whose devices and biometrics the policy engine relies on (§9.3) — and that the platform's design cannot protect a customer whose operators are compromised end-to-end. ✅ Fireblocks has publicly documented these campaigns in its own threat reports and blog; ✅ the campaign wave is independently corroborated; ⚠ attribution specifics and victim lists are contested in places and are not asserted here beyond what Fireblocks itself published.

The 2023–2025 continuation matters for the vendor-diligence reader. The pattern did not stop after the March 2023 wave: North-Korean-aligned operators kept iterating on the recruiting-scam playbook — fake job interviews, trojanized coding challenges, wallet-connect phishing — and Fireblocks' reports track the evolution because its customers are the target population. For a bank, the practical takeaways are three: (1) the threat model for custody operations must include **operator-targeted social engineering** as a first-class risk, not an edge case; (2) the vendor's threat-intelligence feed is a real security control that should be wired into the bank's own cyber-defense operations; and (3) incident-response plans must assume that a *customer-side* compromise (not a vendor breach) is the more probable failure mode. ⚠ the general industry assessment of Lazarus's persistence is well corroborated; the specific campaign timeline is Fireblocks-published.

### 10.4 Customer-Side Incidents and the Vendor Boundary

The vendor-boundary rule for this guide: **no incident is attributed to Fireblocks' platform unless a primary source says so.** Customer-side incidents involving Fireblocks-using firms (e.g., firms that suffered losses while using Fireblocks infrastructure, or the 3AC collapse where Fireblocks was merely a Network member) are the customers' incidents, not Fireblocks'. The pattern the threat record supports: Fireblocks' security model concentrates risk at the governance layer (customer policies, approver devices, operator credentials) rather than at the key-storage layer — which is precisely what the January 2022 and Lazarus episodes illustrate from opposite directions. ✅/⚠ as flagged per incident.

### 10.5 The Incident Table

| Incident | Date | What is verified | Status |
| --- | --- | --- | --- |
| API/admin disclosure episode | Jan 2022 | Episode occurred; disputed details; no funds lost per company; no primary source says Fireblocks was hacked | ✅ occurred / ⚠ details / ❌ "hacked" framing |
| BitGo TSS zero-knowledge vulnerability research | Dec 2022 | Fireblocks' report exists and is published; severity disputed by the two vendors | ✅ report exists / ⚠ severity |
| Lazarus recruiting-phishing campaigns ("Contagious Interview") | Mar 2023 + ongoing | Fireblocks documented campaigns targeting crypto-firm employees incl. its users; NCC Group corroborates the campaign wave | ✅ documented / ⚠ attribution specifics |
| Customer-side losses at Fireblocks-using firms | Various | Not attributable to Fireblocks platform without primary source | ⚠/❌ |
| 3AC collapse (Network member) | Jul 2022 | 3AC was listed among Network members in the Feb 2022 press; its collapse was a member's credit event, not a platform incident — Network membership is not a credit endorsement (§6.1) | ✅ as stated / ⚠ if misread as endorsement |

---

## 11. The Cymbal Bank Worked Example — A Digital-Asset Custody Client

### 11.1 The Scenario

**Cymbal Bank**, a fictional-but-substantiated Singapore-based bank (the repository's only bank persona, per the conventions of [Bitunix](bitunix_guide.md) §10 and [Reap Global](reap_global_guide.md) §11), decides to launch a **digital-asset custody and settlement service** for its institutional clients — custody of bitcoin and ether, stablecoin settlement, and participation in a tokenized-bond pilot. Cymbal Bank evaluates build-versus-buy and selects a **Fireblocks-style MPC infrastructure platform** (the worked example uses Fireblocks' documented product model: workspace, policy engine, MPC key shares, the Network). The example follows the sibling worked-example conventions: a realistic onboarding, the control design, the settlement flows, and the regulatory overlay — extending, not re-deriving, the conventions established in the Bitunix and Reap examples.

The build-versus-buy evaluation is worth recording because it is the decision every bank in this position actually makes. Cymbal Bank's scoring of the three options — build the cryptography in-house, buy a hardware-wallet vendor, or adopt an MPC platform — weighed five criteria: time-to-market (the MPC route wins: weeks of policy configuration versus 18+ months of cryptography engineering), the control vocabulary (the policy engine matches the bank's existing maker-checker discipline), regulatory defensibility (a vendor with SOC 2/ISO artifacts and an incident record supports the bank's MAS outsourcing file), vendor concentration (the single-vendor risk of the MPC route), and exit cost (the portability of key shares and the workspace data). The MPC route won on the first three and lost on the last two; the bank accepted the trade, and the vendor file records the scoring so the decision is auditable. ⚠ the scoring is design fiction built on the verified product facts of §5 and §9.

### 11.2 Onboarding and the Workspace

Cymbal Bank's onboarding team opens a **workspace** and designs the asset-account topology before any funds move: one **vault per business line** (Custody; Stablecoin Treasury; Tokenized-Bond Pilot), each with distinct policies. The custody operations team is split into roles — initiators (back-office, web console), approvers (front-office and risk, mobile devices), and administrators (a small team with workspace-level rights) — mirroring the four-eyes-plus discipline the bank already applies to fiat payments. The policy engine is configured at the account level: transaction limits per asset, **two-of-three approval quorums** for outbound transfers above US$1 million, beneficiary whitelists for Network counterparties, and a hard rule that no single approver can sign alone. The workspace is the unit of audit: every policy change and every approval is logged, which later feeds the bank's internal-audit and MAS outsourcing reviews (§11.5). The bank's technology-risk team runs the vendor through its standard third-party assessment — security certifications (SOC 2/ISO, §9.5), the bug-bounty program, the incident record (§10) — and documents the answers in the vendor file. The onboarding itself is deliberately staged: a sandbox workspace first (policies tested with test keys), then a pilot vault with a capped balance and a single business line, then the full production topology — a go-live discipline that mirrors how the bank would roll out any new payments platform, and that the sibling worked examples (Bitunix §10, Reap §11) follow in their own domains.

The policy-engine configuration the bank lands on, mapped to the documented mechanics of §5.3:

| Policy | Configuration | Governance rationale |
| --- | --- | --- |
| Per-asset transaction limits | Bitcoin/ether custody: US$5M per outbound transfer; stablecoin treasury: US$25M | Caps match the board-approved risk appetite per business line |
| Approval quorum | Two-of-three approvers above US$1M; single approver below US$1M; no sole-signer override | Four-eyes-plus discipline, mirroring the fiat payments maker-checker model |
| Beneficiary whitelist | Network counterparties pre-vetted by the credit team; raw-address sends require risk-committee sign-off | Kills destination-address risk on the Network legs (§6.1) |
| Time-of-day and velocity rules | No outbound transfers outside the Asia-Pacific settlement window; daily velocity caps per account | Aligns custody operations with the bank's operating hours and monitoring coverage |
| Policy-change governance | Policy changes themselves require two administrators and a 48-hour notice to the risk team | Dual control over the control plane, per the §5.3 governance principle |
| Signing-path control | API signing disabled for custody vaults by policy; TEE/biometric-bound mobile approval is the only signing path | Keeps the co-signing model intact even where the API platform (§5.5) would permit automation |

### 11.3 The MPC Key Management and Quorum

The custody vault is generated under **MPC**: key shares are distributed across three environments — Cymbal Bank's own infrastructure (share 1), the Fireblocks-style vendor's environment (share 2), and the approvers' mobile devices (share 3) — with a **threshold of two** required to sign, so no single environment can produce a signature and **no complete private key ever exists in any one place**. Cymbal Bank's governance documentation states the property the bank actually cares about: the vendor cannot move funds alone, and neither can any single compromised employee of the bank. The bank's risk committee reviews the residual risks that the architecture does *not* remove: a fully compromised approver device (mitigated by TEEs and biometrics), insider collusion between two approvers (mitigated by policy-engine separation of duties and monitoring), and social engineering of approvers (the Lazarus pattern, §10.3 — mitigated by training and by the vendor's threat-intelligence feeds). The bank also tests the "no single party" claim in its own UAT: a simulated vendor-side compromise (a test key share) is shown to produce no valid signature without the bank's approvers — the design property is verified operationally before go-live.

The key-holder and quorum table, as recorded in the bank's key-governance register:

| Environment | Share | Role | If compromised |
| --- | --- | --- | --- |
| Cymbal Bank infrastructure | Share 1 | Bank-side signing node in the bank's own data centre | Alone, cannot sign (threshold 2-of-3) |
| Vendor (Fireblocks-style) environment | Share 2 | Vendor-side signing node | Alone, cannot sign — the §11.2 UAT demonstrated this operationally |
| Approver mobile devices | Share 3 (per approver) | TEE/biometric-bound approval | Alone, cannot sign; device compromise still exposes the approval channel (§10.3) |
| Threshold | 2-of-3 | Any two environments jointly produce a valid signature | The minimum collusion set is two environments — which is why separation of duties (§11.2) matters |

### 11.4 Settlement via the Network and Off-Exchange Flows

Cymbal Bank's settlement flows use the **Fireblocks Network** for the institutional legs and on-chain settlement for the rest (cross-ref [Payment Rails](payment_rails_guide.md) for the fiat-side mechanics; the Bitunix worked example's exchange-flows conventions are extended here, not re-derived):

1. **Network transfer to a trading venue** — Cymbal Bank initiates a USDC transfer to a Bitunix-style exchange's Network address; the exchange is a known Network member, so the destination is an authenticated channel rather than a raw address, and the bank's beneficiary whitelist approves it. Settlement finality is the on-chain USDC transaction; the Network is the orchestration layer.
2. **Off-exchange settlement** — for the bank's market-making relationship, Cymbal Bank and the venue agree an off-exchange structure (§6.2): principal stays in **MPC-secured shared wallets** while collateral is **mirrored to the exchange** with on-chain proof of full collateralization, avoiding on-chain movement for every trade — cutting transfer costs and reducing hot-wallet exposure, at the price of venue credit risk that the bank's credit team must price (the 3AC lesson: Network membership ≠ creditworthiness).
3. **Fiat legs** — the fiat side of the flows (SGD/USD settlement with correspondent banks) runs over traditional rails, cross-referenced to the payment-rails guide's clearing/settlement treatment.
4. **Tokenized-bond pilot settlement** — for the pilot, the issuance workspace mints the tokenized bond on the vendor's tokenization tooling (§6.3), with mint/burn under the bank's policy engine and the investor leg settling over the Network; the pilot's operating model follows the [Tokenized Assets](tokenized_assets_guide.md) guide's issuance conventions, condensed here.
5. **Reconciliation and the audit trail** — each settlement day closes with a three-way reconciliation: the bank's ledger, the workspace's transaction log (who initiated, who approved, which policy matched), and the on-chain record; the reconciled pack is the artifact the bank's MAS outsourcing file and its clients' due diligence will cite (§11.6).

### 11.5 AML/Screening Integration and the MAS Overlay

Cymbal Bank's financial-crime team integrates **screening into the transfer flow**: every inbound and outbound transfer is screened against sanctions lists and the bank's risk typologies (cross-ref [Fircosoft](fircosoft_guide.md), condensed), with the vendor's transaction-scanning, travel-rule, and AML tooling feeding the bank's own monitoring stack. The **MAS overlay** (cross-ref [MAS Regulations](mas_regulations_guidelines_guide.md), condensed) is the bank's responsibility, not the vendor's: Cymbal Bank holds the regulated activity (DPT custody/dealing under the PS Act), the vendor is a technology supplier — which is exactly the "infrastructure, not licensed custodian" boundary §7.1 describes — and MAS' outsourcing and technology-risk expectations require the bank to document vendor due diligence, data residency, and business continuity for the custody platform. The bank's licensing application and its vendor file are prepared in parallel; the worked example deliberately leaves the MAS-approval outcome open (⚠, per the repository's convention of not resolving legal questions).

The travel-rule data flow is the concrete instance of the overlay. For every VASP-to-VASP transfer above the FATF threshold, Cymbal Bank's stack assembles the originator/beneficiary packet from its own KYC records and the vendor's travel-rule tooling (§7.3), screens both counterparties, and retains the packet with the transaction record — the record-keeping that MAS and the FATF standards require of the licensed party. The vendor supplies the transport and the screening hooks; the bank supplies the legal obligation, the customer data, and the retention schedule. That division is the entire §7.1 boundary in one operational flow: infrastructure on the vendor's side, regulation on the bank's. ⚠ the flow is design fiction, but every component (travel-rule tooling, screening hooks, the infrastructure-versus-license boundary) is a documented fact from §7.3 and §8.3.

### 11.6 The Lessons

The worked example distils to five lessons for the repository: (1) **MPC custody moves risk from key storage to governance** — the policy engine, the approver quorum, and the vendor boundary are where a bank's control design actually lives; (2) **the Network changes counterparty risk but does not remove it** — authenticated channels kill destination-address risk, not credit risk; (3) **the vendor is infrastructure, the bank is the regulated party** — screening, travel-rule, and MAS obligations stay with Cymbal Bank; (4) **the human layer is the attack surface** — the Lazarus campaigns target approvers, so a bank's custody security is only as strong as its operator training and device controls; and (5) **the evidence trail is the deliverable** — the workspace logs, policy records, and vendor attestations assembled during onboarding are the same artifacts the bank's internal audit, its MAS supervision, and its clients' own due diligence will demand, so the custody program should be built to be audited from day one. The example's value to the repository is that it takes the Fireblocks facts verified in §2–§10 — the workspace model, the threshold quorum, the Network, the off-exchange trade, the vendor boundary — and shows a bank converting them into an operating design.

---

## 12. The Claims Audit — Verified, Flagged, Rejected

### 12.1 The Verified Claims (✅)

| Claim | Verification |
| --- | --- |
| Founded 2018; platform launched June 2019 | Company blog/press + trackers ✅ |
| Founders Michael Shaulov, Idan Ofrat, Pavel Berengoltz; ex-Check Point | Company About page + trackers ✅ |
| HQ New York | Multiple independent profiles ✅ |
| Series A US$16M (Jun 2019) — amount corroborated by the company's own cumulative math | Trackers + Series B press cumulative figure ✅ |
| Series B US$30M led by Paradigm (Nov 2020); cumulative US$46M | Company press release ✅ |
| Series C US$133M led by Coatue, Ribbit, Stripes; BNY Mellon/SVB Capital strategics; cumulative US$179M | Company blog ✅ |
| Series D US$310M, six co-leads incl. Sequoia, at >US$2B valuation; cumulative US$489M | Company press + TechCrunch/Calcalist ✅ |
| Series E US$550M co-led by D1 Capital Partners and Spark Capital at >US$8B valuation | Company press + Reuters/CoinDesk ✅ |
| MPC custody, policy engine, threshold key shares, TEE/biometric approval, HSM/enclaves | Company security page + standard cryptography ✅ |
| Fireblocks Network exists; verified-counterparty design; off-exchange settlement product; tokenization platform; stablecoin/payments; staking/DeFi | Company product pages ✅ (off-exchange also third-party corroborated) |
| Bugcrowd managed bug-bounty engagement | Bugcrowd platform listing ✅ |
| BitGo TSS vulnerability report published (Dec 2022) | Company blog/press ✅ (severity disputed ⚠) |
| Contagious Interview Lazarus threat report published | Company blog ✅ (campaign wave corroborated by NCC Group) |
| Jan 2022 API-disclosure episode occurred; no primary source says Fireblocks was hacked | Press + company statements ✅ |
| Trust Center exists (trust.fireblocks.com, SafeBase-powered) | Site verified ✅ (contents behind registration ⚠) |
| ABN AMRO "Europe's first major bank to issue tokenized bonds" reference | Company homepage ✅ |
| Earliest adopters: Galaxy Digital and Genesis (DCG) | Company Series C blog ✅ |
| Bitget off-exchange integration of Fireblocks (2025) | The Paypers industry coverage ✅ |

### 12.2 The Flagged Claims (⚠)

| Claim | Why flagged |
| --- | --- |
| 2,400+ enterprises, US$10T transactions, 550M wallets (2026) | Company-reported metrics |
| 800+ customers, US$2T transferred, US$45B AUM (Feb 2022) | Company-reported metrics |
| "~15% of daily crypto transaction volume" (D1 quote, Feb 2022) | Investor quote, single-sourced |
| Network payments reach: 40+ providers, 60+ currencies, 100+ countries | Company-reported metric |
| "Only network with patented technology" | Company claim, not independently verified |
| SOC 2 Type 2, ISO 27001, CCSS QSP Level 3, NCC-audited libraries | Company-published this pass |
| "Fireblocks cannot move customer funds" | Design property, not audited guarantee |
| Licensing: FinCEN MSB, state MTLs, UK FCA, Lithuania, Switzerland VQF | Register lookups not possible this pass |
| Singapore/MAS status | No licence found; directory not queryable; inferential |
| "FTS" acronym; tokenization-agent naming | Not verified at primary source |
| Seed-round existence/size | Tracker conflict; not primary-verified |
| Series A lead attribution (Cyberstarts/Tenaya) | Tracker-based; company lists them as early investors |
| Aave Arc current status; staking yields | Time-varying / industry-reported |
| BitGo vulnerability severity | Vendor-vs-vendor dispute |
| 1,300+ institutions (2023 claims) | Company-reported; no third-party audit |
| 500+ digital-asset businesses (Jul 2021, Series D) | Company-reported |
| Bridge: "cut bulk settlement times from 12+ hours to under 90 minutes" | Customer-story metrics, company-curated |
| Network reach: "100% of crypto-lending market, ~90% of liquidity venues" (2021) | Company claim (Series C blog) |
| Cumulative total ~US$1.04B | Tracker-derived; company states US$489M through Series D and "just over US$1B" at Series E |

### 12.3 The Rejected or Not-Found Claims (❌)

| Claim | Disposition |
| --- | --- |
| Series B "US$40M, Ribbit-led" (brief) | Refuted — US$30M, Paradigm-led |
| Seed/Series A "~US$30M" (brief) | Refuted — Series A US$16M |
| "Fireblocks was hacked" (Jan 2022 framing) | No primary source supports it |
| New York BitLicense held | Not found this pass |
| Any MAS DPT licence for Fireblocks | Not found this pass |
| "FTS" as a named product acronym | Not found at primary source |

### 12.4 What Could Not Be Verified

The following items could not be confirmed at a primary or authoritative source during this pass, and are recorded rather than smoothed:

- **Fireblocks' current legal-entity map** — the operating entities behind the brand (US entity, Israeli R&D entity, European entities) and their jurisdictions are not published on the pages examined.
- **The full licensing register** — FinCEN MSB registrant status, state money-transmitter licenses, UK FCA registration, Lithuanian VASP registration, Swiss VQF membership: none could be checked against live regulator registers this pass (interactive lookups unavailable), so all remain ⚠.
- **The Singapore/MAS posture** — no MAS licence or registration for Fireblocks was found in coverage examined, but the MAS Financial Institutions Directory could not be queried, so the absence is "not found," not "confirmed absent."
- **A distinct seed round** — Tracxn counts six rounds including a seed; Seedtable counts five; no primary source for a seed round was found.
- **Series A lead investors** — Cyberstarts and Tenaya are confirmed as early investors by later company press, but the Series A press release itself was not found; the US$16 million amount rests on trackers plus the company's own cumulative-math consistency (US$16M + US$30M = US$46M).
- **The FTS acronym and "Tokenization Agent" naming** — the research brief's "FTS" did not verify at any primary source; the tokenization product verifies by name, the acronym does not.
- **Scheme-level MPC details** — the specific threshold-ECDSA/GGN18-style protocol names and threshold parameters are not enumerated on the public pages examined; the MPC/threshold concept verifies, the scheme names do not.
- **Exact founder titles** (beyond Shaulov's CEO) and the **current office list** — company bios were not extractable this pass; tracker roles are single-sourced.
- **The January 2022 episode's technical specifics** — the researcher's name, the exact vulnerability scope, and the resolution were not re-confirmable at primary sources this pass; the episode is recorded as contested.
- **Revenue and unit economics** — no revenue, EBITDA, or profitability figure has been published.
- **Company-reported metrics** (customers, volumes, wallets, AUM, Network reach) — none are independently audited.
- **The current status of the Aave Arc program** and **staking yields/networks** — time-varying, industry-reported.
- **Trust Center contents** (trust.fireblocks.com) — the artifacts themselves sit behind a SafeBase wall and were not accessible without registration this pass.

---

## 13. The Glossary

| Term | Meaning |
| --- | --- |
| **Asset account** | A logically separated bucket within a Fireblocks workspace (per product line, client, or business unit), with its own policies and approval rules |
| **CCSS** | Cryptocurrency Certification Consortium security standard; Fireblocks claims QSP Level 3 (⚠ company claim) |
| **Co-signing** | The requirement that more than one party participate in producing a valid signature (via MPC) |
| **Direct custody** | A bank/fintech custodying its own digital assets on its own balance sheet and brand, using infrastructure rather than a sub-custodian |
| **Fireblocks Network** | Authenticated settlement network connecting Fireblocks-member institutions for digital-asset transfers |
| **FSPM** | Fireblocks Security Posture Management — configuration-risk visibility tooling |
| **HSM** | Hardware security module — tamper-resistant hardware for key operations |
| **MPC** | Multi-party computation — cryptographic protocols that let parties jointly compute a function (here, a signature) without revealing their inputs (key shares) |
| **Off-exchange settlement** | Arrangement where trading counterparties hold assets under joint control (MPC-secured shared wallets, mirrored collateral) so assets need not move on-chain per trade |
| **Policy engine** | The rule layer that governs which transactions may be signed, by whom, under what limits and quorums |
| **Quorum** | The minimum number of approvers/shares required to authorize an action (e.g., two-of-three) |
| **SOC 2 Type 2** | AICPA attestation of a service organization's controls over a period — third-party audited |
| **TEE** | Trusted execution environment — hardware-isolated execution (e.g., on approval devices) |
| **Threshold signature** | A signature computed jointly by a threshold of key-share holders; indistinguishable on-chain from a single-key signature |
| **Travel rule** | FATF requirement that VASPs share originator/beneficiary information on transfers |
| **TSS** | Threshold signature scheme — the protocol family BitGo's wallets use and Fireblocks' researchers audited in 2022 |
| **VASP** | Virtual asset service provider (FATF terminology) |
| **WaaS** | Wallet-as-a-Service — white-label wallet infrastructure for fintechs |
| **Workspace** | The customer's organizational unit in the Fireblocks console — vaults, accounts, users, policies |
| **Zero-knowledge proof** | A cryptographic proof of a statement that reveals nothing beyond its truth; the missing component in the BitGo TSS vulnerability Fireblocks reported |
| **MPI / PI licence** | MAS Major Payment Institution / Payment Institution licence under the Singapore Payment Services Act |
| **PS Act** | Singapore's Payment Services Act — the statute governing DPT services and payment services |
| **Contagious Interview** | A Lazarus-linked fake-recruiting malware campaign documented by Fireblocks' threat research |
| **Project Guardian** | MAS-led tokenization pilot initiative in which infrastructure providers have participated (Fireblocks' specific role ⚠) |
| **FinCEN MSB** | US Financial Crimes Enforcement Network money-services-business registration |
| **Direct-custody doctrine** | Fireblocks' positioning that institutions custody their own assets on their own balance sheet using its infrastructure |
| **Aave Arc** | Permissioned institutional DeFi pool Fireblocks launched with in 2022 ("first to enable institutional DeFi"); current status ⚠ |
| **AUM** | Assets under custody on the platform — US$45B claimed (Feb 2022), the only AUM figure ever published |
| **Chainalysis** | Third-party blockchain-analytics and screening vendor; integrated with Fireblocks custody in the Reap flow (sibling guide) |
| **ClearLoop** | Copper's off-exchange settlement product — the direct competitor to Fireblocks' off-exchange offering |
| **DeFi Security Suite** | Fireblocks' DeFi-protection bundle: policy controls, smart-contract safeguards, token-approval monitoring and revocation |
| **Embedded Wallets** | White-label wallet line for consumer/retail apps — the WaaS sibling product |
| **SafeBase** | The vendor behind Fireblocks' Trust Center (trust.fireblocks.com), where security/compliance artifacts are shared with customers |
| **Threshold ECDSA** | The threshold-signature family used for bitcoin/ethereum-style chains; scheme-level names ⚠ not enumerated publicly |

---

## 14. Cross-References and the Closing Summary

**Sibling guides (same folder, banking cluster):** [Reap Global](reap_global_guide.md) — Fireblocks appears there as Reap's custodian wallet for stablecoin repayments (Jan 2023) and in the Chainalysis-integration item (Jan 2024); this guide is the dedicated deep-dive that reference points to. · [Bitunix](bitunix_guide.md) — custody "via Cobo + Fireblocks"; this guide provides the Fireblocks half of that pairing, and §11 follows its worked-example conventions. · [Tokenized Assets](tokenized_assets_guide.md) — positions Fireblocks as the institutional MPC-custody + transfer-network standard and tokenization tooling provider; §6.3 and §9 here examine that positioning. · [Fircosoft](fircosoft_guide.md) — AML/sanctions screening themes, cross-referenced in §7.3 and §11.5. · [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) — the Singapore PS Act/DPT regime, cross-referenced in §8 and §11.5. · [Payment Rails](payment_rails_guide.md) — clearing/settlement mechanics for the fiat legs, cross-referenced in §6 and §11.4.

**Technology guides (prefix `../technology/`):** [Smart Contracts](../technology/smart_contracts_guide.md) — on-chain mechanics (ERC-20, Merkle proofs, custody/wallet security, audit practice, incident history), cross-referenced, condensed, in §9 and §10. · [Cybersecurity](../technology/cybersecurity_guide.md) — hot/cold separation, multi-signature, ISO/IEC 27001, bug bounties, cross-referenced, condensed, in §9.2 and §9.5.

**Primary sources used this pass:** fireblocks.com (homepage, About, Security, Fireblocks Network and Off-Exchange product pages, and the Series B/C/D/E press pages and blogs — all extracted September 2026); trust.fireblocks.com (referenced, content behind registration); developers.fireblocks.com (referenced); bugcrowd.com/engagements/fireblocks-mbb-og (bug-bounty engagement listing); TechCrunch (Series D, July 2021); Reuters and Yahoo Finance (Series E, February 2022); Calcalist (Series D); CoinDesk (Series E); The Block and Fireblocks' own report (BitGo TSS, December 2022); NCC Group (Lazarus/fintech research); Ethena documentation and The Paypers (off-exchange settlement corroboration); Tracxn, Seedtable, Crunchbase, startupintros, earlyriders, everybodywiki, Built In NYC (tracker corroboration).

**Closing summary:** Fireblocks is the institution that turned the private-key problem into a product category: founded in 2018 by three Check Point alumni to end single-point key theft, it raised roughly US$1.04 billion across five documented rounds to become the most valuable digital-asset infrastructure provider of its era, and it now sells MPC custody, an authenticated settlement network, off-exchange settlement, tokenization, and stablecoin rails to 2,400+ enterprises — figures that remain company-reported and flagged throughout. For a bank, the Fireblocks story is the clearest available map of how digital-asset custody actually works: keys that never exist whole, quorums enforced by policy engines, settlement that moves from raw addresses to authenticated channels, research that doubles as marketing, and a vendor boundary that leaves regulation with the licensed institution. The claims audit in §12 separates what verifies — the founding, the founders, every funding round, the architecture — from what is company-reported and from what could not be confirmed at all, and the Cymbal Bank worked example shows a bank putting the model to work. The one line that captures the whole industry, and the whole point of this guide, is that every custody decision eventually resolves into the same place — the vaulted key.
