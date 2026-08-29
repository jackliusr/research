# Citibank: The Software Systems Landscape — A Comprehensive Guide to the Technology Citi Runs

*A companion deep-dive in the per-bank software-systems series of the [jackliusr/research](https://github.com/jackliusr/research) repository — the Citi entry alongside [DBS](dbs_software_systems_guide.md) and [Standard Chartered](standard_chartered_guide.md). This guide focuses on the **specific software and technology systems** behind Citibank and Citigroup: the Treasury & Trade Solutions platforms (CitiDirect BE, WorldLink), the markets platforms (Citi Velocity, Lava), the cards estate, wealth platforms, the core banking and legacy mainframe estate, data & AI, and the transformation programmes that followed the October 2020 consent orders — what is publicly documented, what is inferred from industry practice, and what Citi simply does not disclose.*

**Verification convention used throughout: ✅ = verified in this research pass (primary or secondary sources); ⚠ = flagged (inferred, approximate, single-source, or structural inference); ❌ = disputed (the record contradicts the claim); unmarked = structural/industry knowledge presented as such. The consolidated [Claims-Status table is in §11](#11-claims-status-and-verification-notes), and the non-public specifics are collected in [§12](#12-what-could-not-be-verified).**

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Banking Domain / Software-Systems Focus — the technology estate of Citibank, N.A. and Citigroup Inc. (NYSE: C): TTS platforms, markets platforms, cards, wealth, core/legacy, data & AI, transformation and risk context, the Singapore franchise
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** August 2026
> **Companion guides:** [DBS Software Systems Guide](dbs_software_systems_guide.md) and [Standard Chartered Guide](standard_chartered_guide.md) (the structural models for this series), [Core Banking Systems Guide](core_banking_systems_guide.md), [Payment Rails Guide](payment_rails_guide.md), [Trade Finance Guide](trade_finance_guide.md), [Trade Finance Systems Guide](trade_finance_systems_guide.md), [FIX Protocol Guide](fix_protocol_guide.md), [Private Banking Guide](private_banking_guide.md), [Enterprise Risk Management Guide](enterprise_risk_management_guide.md), [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md), [Universal Banking Model Guide](universal_banking_model_guide.md), [IBM Cloud Guide](../technology/ibm_cloud_guide.md), and the [AI/LLM guides](../technology/ai_llm/)

---

## Table of Contents

1. [Bank Profile: From City Bank of New York to Citi](#1-bank-profile-from-city-bank-of-new-york-to-citi)
2. [Treasury & Trade Solutions (TTS): CitiDirect BE and WorldLink](#2-treasury--trade-solutions-tts-citidirect-be-and-worldlink)
3. [Markets Platforms: Citi Velocity and Lava](#3-markets-platforms-citi-velocity-and-lava)
4. [Cards Business: Co-Brand Economics](#4-cards-business-co-brand-economics)
5. [Wealth Platforms](#5-wealth-platforms)
6. [Core Banking and the Legacy Estate](#6-core-banking-and-the-legacy-estate)
7. [Data and AI](#7-data-and-ai)
8. [Transformation and Risk Context: 2020 Consent Orders, Modernization, and Cloud](#8-transformation-and-risk-context-2020-consent-orders-modernization-and-cloud)
9. [Singapore Angle](#9-singapore-angle)
10. [Worked Example: Cymbal Bank × Citi — Correspondent Banking and Trade Services](#10-worked-example-cymbal-bank--citi--correspondent-banking-and-trade-services)
11. [Claims Status and Verification Notes](#11-claims-status-and-verification-notes)
12. [What Could Not Be Verified](#12-what-could-not-be-verified)
13. [Glossary](#13-glossary)
14. [References and Further Reading](#14-references-and-further-reading)

---

## 1. Bank Profile: From City Bank of New York to Citi

### 1.1 Scope and Verification Convention

This guide is the **software-systems deep-dive for Citibank/Citi** — the Citi mirror of the [DBS](dbs_software_systems_guide.md) and [Standard Chartered](standard_chartered_guide.md) systems guides. Because the repository has no separate Citi *bank* guide, this entry carries both the verified bank profile (history, footprint, divisions — §1) and the systems landscape (§2–§7), followed by the transformation & risk context (§8), the Singapore angle (§9), a worked Cymbal Bank example (§10), and the honest claims audit (§11–§12).

The verification discipline is the same one this series applies to every bank: **✅ verified** means the claim was confirmed in this research pass against a primary source (Citigroup's own heritage pages and press releases, OCC and Federal Reserve enforcement documents, Citi product pages) or a strong secondary source (National Library Board Singapore, established financial press). **⚠ flagged** means the claim is inferred, approximate, single-source, or structurally reconstructed — the reader should treat it as a hypothesis about a class of system, not a fact about Citi. **❌ disputed** marks claims where the public record contradicts the common telling (there is at least one such claim in this guide — see §11). The [What Could Not Be Verified section](#12-what-could-not-be-verified) collects every materially non-public item.

### 1.2 What Is Public: The Citi Disclosure Reality

Citi sits between the two extremes this series has documented. It is **far more transparent than DBS** about its platforms: CitiDirect BE, WorldLink, Citi Velocity, and Lava are all publicly named products with public marketing material, and Citi publishes technology-spend figures in earnings disclosures. But it is **far less transparent than Standard Chartered about its *core***: there is no Citi equivalent of SC's "Atlas on AWS" narrative — Citi does not publicly name its core banking system, its mainframe estate, its payments hub, or its card-management platform, and the September 2020-era disclosures around "decades-old technology" (Jane Fraser's phrase, widely reported) are strategic, not architectural.

The consequence, enforced rigorously throughout: the **product layer** of the Citi stack (channels, client-facing platforms) is mostly ✅-verifiable; the **engine layer** (core, hubs, data platform, risk systems) is mostly ⚠-inferred, with the notable exception of the compliance-driven remediation programmes that regulators forced into the public record in 2020.

### 1.3 The Verified History: 1812 → 2026

The Citi lineage is the longest continuous corporate history in American banking, and every load-bearing date is verified against Citi's own heritage pages ✅:

- **June 16, 1812** — the **City Bank of New York** comes into existence when its state charter passes the New York legislature ✅ (Citi heritage: "A new bank born in New York"; the 1811 petition, Samuel Osgood's plan, and the June 16, 1812 charter are all documented on Citigroup's own heritage page). The first president was **Samuel Osgood**, the statesman and retired colonel who had served as the first U.S. Postmaster General ✅ (Wikipedia, corroborated by Citi's narrative). The 200-year story Citi celebrated in 2012 runs from this charter.
- **July 1865** — City Bank converts its **state charter to a national charter** under the National Bank Act, taking the new name **National City Bank of New York** ✅ (Citi heritage: "A banker's bank: National charter and expansion of services"). This is the origin of the "national" lineage that ends in "Citibank, N.A." Note the precision: the 1865 name was *National City Bank of New York* — the "First National City Bank" name came later (1955, after the merger with the First National Bank of the City of New York ⚠ secondary sources; shortened to "First National City Bank" in 1962, per the heritage record's own 1961 usage ✅).
- **1897** — a dedicated **foreign exchange department** and correspondent links forged overseas ✅ (Citi heritage) — the corporate ancestor of today's correspondent-banking and Treasury & Trade Solutions businesses (§2, §10).
- **1902** — the **International Banking Corporation (IBC)**, a Connecticut-chartered bank founded in 1901 to promote trade with Asia, opens in Singapore ✅ (Citi heritage; see §9).
- **1914** — National City Bank's first big overseas push, in Latin America — widely documented as the **first overseas branch of a U.S. bank** ✅ (Citi heritage; corroborated by secondary sources).
- **1955** — becomes **First National City Bank of New York** after merging with the First National Bank of the City of New York ⚠ (secondary; consistent with the heritage record).
- **1976** — renamed **Citibank, N.A.** ✅ (National Library Board Singapore; Wikipedia; multiple corroborating sources — the rename is also attributed in the lore to avoiding confusion with Cleveland's National City Corp.).
- **1998** — **Citigroup** is created by the merger of Citicorp and Travelers Group ✅ (Citi heritage: "Momentous encounter leads to merger").
- **2012** — Citi celebrates its 200th anniversary ✅ (Citi heritage).
- **March 1, 2021** — **Jane Fraser** becomes CEO, the first woman to lead a major U.S. bank ✅ (Citi heritage; universally documented).

### 1.4 The Global Footprint

- **Citi does business in more than 160 countries and jurisdictions** ✅ — this is Citi's own boilerplate (e.g., the February 2023 CitiFX release: "Citi does business in more than 160 countries and jurisdictions"). The on-the-ground institutional network is smaller than the client-serving footprint; Citi's public language distinguishes "doing business in" from "operating in" ⚠ (the exact operating-market count varies by disclosure year and definition).
- **Consumer banking is far narrower**: Citibank has branches in ~19 countries, concentrated in six U.S. metropolitan areas (New York, Chicago, Los Angeles, San Francisco, Washington D.C., Miami) ⚠ (Wikipedia; consistent with Citi's post-2014 exit from mass-market retail in many countries).
- **Regional structure**: Citi organizes into four regions — North America; Latin America; Europe, Middle East & Africa (EMEA); and Asia Pacific (with Singapore as a key hub; §9) ✅ (Citi's public reporting structure).

### 1.5 The Five Divisions (Post-2021)

Citi's March 2021 organizational redesign (announced under Fraser, effective 2021–2022) replaced the old two-segment consumer/institutional split with **five business lines**, each head reporting to the CEO ✅ (The Banker: "Inside Citi's rebuild"; finews.asia; Citi disclosures):

| Division | Scope (verified publicly) | Systems that serve it (this guide) |
|---|---|---|
| **Services** | Institutional services: **Treasury and Trade Solutions (TTS)** + **Securities Services** (custody, fund services) | CitiDirect BE, WorldLink, SWIFT/ISO 20022 rails (§2) |
| **Markets** | FX, rates, equities, commodities, credit; the markets e-trading estate | Citi Velocity 3.0, Lava/LavaFlow, FIX (§3) |
| **Banking** | Investment banking, corporate banking, commercial banking | Client-facing banking platforms ⚠ (§3/§6 cross-refs) |
| **Wealth** | Citi Global Wealth — private bank, affluent, and wealth platforms | Wealth portals/platforms ⚠ (§5) |
| **US Personal Banking** | U.S. retail: branded cards, retail banking, mortgage | Cards co-brand estate, retail core ⚠ (§4, §6) |

Verified leadership anchors: Shahmir Khaliq as head of Services (TTS + Securities Services), Andrew Morton as head of Markets, and — for Wealth — **Andy Sieg, who joined Citi in September 2023 as Head of Wealth** from Merrill ✅ (Citi Private Bank bio; Citi leadership pages). The "Banking" head has rotated (Peter Babej served as interim head at the 2021 redesign ⚠). The pre-2021 structure this replaced was the five-segment consumer/institutional split introduced under Michael Corbat (2019–2021) ⚠ (well documented in press).

### 1.6 The Consolidated Technology Stack (2026 View)

The whole landscape at a glance — the map that §2–§9 then unpack. Evidence class per the §11 table:

| Layer | Systems (names as publicly known) | Evidence class |
|---|---|---|
| **Channels (institutional)** | CitiDirect BE (launched ~2009; being reengineered as "next-gen CitiDirect"); CitiDirect BE Mobile app; Citi Velocity (web/desktop/mobile) | ✅ product names & launches verified; ⚠ internals |
| **Channels (retail/wealth)** | Citi mobile app, citi.com, Citigold/Private Bank portals | ✅ products; ⚠ stack |
| **TTS/payments** | CitiDirect BE, WorldLink, SWIFT (GPI member), ISO 20022, local rails (ACH, FAST, etc.) | ✅ platforms verified; ⚠ hub internals |
| **Markets** | Citi Velocity 3.0 (2023 consolidation of CitiFX Pulse, Citi Velocity Trading/Research, CitiFX Click); Lava/LavaFlow EMS (acquired 2004); FIX connectivity | ✅ verified; ⚠ engine internals |
| **Cards** | Co-brand platforms for American Airlines, Costco and others; Visa/Mastercard scheme rails | ✅ partnerships verified; ⚠ platforms |
| **Wealth** | Citi Private Bank, Citigold (since 1982), wealth platforms | ✅ brands; ⚠ platform internals |
| **Core banking** | Mainframe-era legacy core; unnamed vendor/in-house mix; "retire and simplify" programme (~400 legacy apps retired in 2023) | ✅ spend/retirement verified; ⚠ core identity |
| **Data & AI** | Data governance remediation (consent order), GenAI coding tools for ~30,000 developers (2024) | ✅ verified; ⚠ platform vendors |
| **Infrastructure/cloud** | Google Cloud strategic agreement (2024); Microsoft heritage (CitiDirect BE built with Microsoft, 2009); AWS reported ⚠ | ✅ GCP agreement verified; ⚠ multi-cloud specifics |
| **Risk & compliance** | Remediation programmes under the Oct 2020 OCC/Fed consent orders | ✅ enforcement verified; ⚠ systems |

The pattern to hold for the rest of this guide: **Citi's client-facing platforms are public and verifiable; the engines underneath them are not.** Every section below states exactly which of its claims are ✅ and which are ⚠.

### 1.7 The Technology Organization

The organization that runs this estate is only partially public:

- **Technology & Business Enablement (TBE)** ⚠ — Citi's internal name for its combined technology-and-operations organization is widely reported; in June 2025 the bank appointed **Tim Ryan as head of technology and business enablement**, responsible for Citi's technology and "legacy franchise" teams — the internal term for the run-the-bank legacy estate ⚠ (finainews, flagged — the appointment and the "legacy franchise" naming are reported but not primary-source verified in this pass).
- **The legacy franchise is a public concept, not a public inventory** ⚠ — Citi's earnings presentations and the Fraser-era disclosures consistently separate the "legacy franchise" (run-the-bank) from the modernized estate, but the *inventory* of what sits in each bucket is not published.
- **Scale markers** ✅/⚠ — ~US$10–12B/yr technology spend (§6.2); ~30,000 developers using GenAI coding tools (§7.2); the decommissioning programme running since ~2021 (§6.2). A consolidated engineering headcount is not published ⚠.

### 1.8 Key Milestones Timeline

| Year | Milestone | Status |
|---|---|---|
| 1812 | City Bank of New York chartered (June 16); Samuel Osgood first president | ✅ |
| 1865 | National charter; renamed National City Bank of New York (July) | ✅ |
| 1897 | Dedicated foreign exchange department; overseas correspondent links | ✅ |
| 1901 | International Banking Corporation (IBC) founded under a Connecticut charter to promote trade with Asia | ✅ |
| 1902 | IBC opens in Singapore — the first American bank there (§9) | ✅ |
| 1914 | First overseas branch of a U.S. bank (Latin America push) | ✅ |
| 1955 | First National City Bank of New York (First National Bank merger) | ⚠ |
| 1961 | First National City Bank launches the negotiable certificate of deposit | ✅ |
| 1962 | Name shortened to First National City Bank | ⚠ |
| 1976 | Renamed Citibank, N.A. | ✅ |
| 1977 | Retail pivot; 24-hour ATM service | ✅ |
| 1982 | Citigold inaugurated in Hong Kong — the affluent-wealth brand (§5) | ✅ |
| 1998 | Citicorp–Travelers merger creates Citigroup | ✅ |
| 2004 | Lava Trading Inc. acquired (announced July, closed August) | ✅ |
| 2009 | CitiDirect BE launched with Microsoft collaboration | ✅ |
| 2012 | 200th anniversary | ✅ |
| 2016 | Costco co-brand portfolio acquired from Amex (June close) | ✅ |
| 2020 | OCC US$400M penalty + C&D; Fed C&D (Oct 7); living will rejected (Dec) | ✅ |
| 2021 | Jane Fraser CEO (Mar 1); five-division redesign | ✅ |
| 2022 | Nov: Fed/FDIC flag resolution-plan shortcoming on data quality; Citi Singapore marks 120 years | ✅ |
| 2023 | Velocity 3.0 consolidation (Feb); Andy Sieg head of wealth (Sep); ~400 legacy apps retired | ✅ |
| 2024 | Google Cloud strategic agreement; AA co-brand extension | ✅ |
| 2026 | Citi exclusive AAdvantage issuer (Apr 24 per specialist press); next-gen CitiDirect transition; Custody+ launch | ✅/⚠ |

The timeline reads in four eras, and the systems story maps onto the last two. **Charter era (1812–1865):** the state-chartered City Bank becomes the nationally chartered National City Bank — the legal ancestry of today's Citibank, N.A. **International era (1897–1914):** the foreign-exchange department, the IBC, and the first overseas branch build the correspondent network that §2 and §10 still exercise. **Conglomerate era (1976–1998):** the Citibank rename, the retail pivot, and the Citicorp–Travelers merger assemble the universal-bank shape of §1.5. **Regulated-modernization era (2004–2026):** the platform acquisitions and launches (Lava 2004, CitiDirect BE 2009, Velocity 3.0 2023) sit alongside the enforcement record (2020 consent orders, 2020/2022 resolution-plan findings) and the response to it (multi-year modernization, Google Cloud, next-gen CitiDirect) — the era the rest of this guide documents in depth. Note the 1897/1902/1914 cluster: correspondent banking and trade finance predate every modern Citi product in this guide by a century ✅ (heritage-verified dates). The systems-relevant rows — 2004, 2009, 2016, 2020–2026 — are the anchors the §2–§8 sections hang their verified claims on.

---

## 2. Treasury & Trade Solutions (TTS): CitiDirect BE and WorldLink

### 2.1 The TTS Division and Its Systems

**Treasury and Trade Solutions (TTS)** is Citi's institutional cash-management, payments, and trade business, sitting inside the **Services** division (post-2021 structure, §1.5) ✅. TTS is one of the largest transaction-banking franchises in the world and is the natural home of the two platforms this section documents. For the rails these platforms sit on — SWIFT, ISO 20022, GPI, ACH, real-time domestic rails — this guide **cross-references rather than re-derives**: the mechanics live in the [Payment Rails Guide](payment_rails_guide.md) (messaging, clearing, settlement, correspondent banking) and the [Trade Finance Guide](trade_finance_guide.md) / [Trade Finance Systems Guide](trade_finance_systems_guide.md) (documentary credits, guarantees, supply chain finance, and their platforms). What follows is the Citi-specific, publicly verifiable layer only.

### 2.2 CitiDirect BE: The Flagship Corporate Banking Platform

**CitiDirect BE** is Citi's next-generation corporate/institutional e-banking platform ✅:

- **Launch and channel history** ✅ — Microsoft's September 2009 announcement (Hong Kong) describes **CitiDirect BE as Citi's "newly launched next-generation e-banking platform," serving corporate, financial institution, and public sector clients, "also optimized for the white-label business model,"** supported by a multiyear Microsoft collaboration. CitiDirect (the pre-BE platform) had already been Citi's web-based corporate banking channel for years — the 2009 launch was the "BE" (next-generation) relaunch of that channel ✅/⚠ (the exact year of the original CitiDirect web launch is not pinned in this pass; Citi's own material describes the platform family as having served clients "for decades" ⚠).
- **Functionality** ✅ — Citi's own description: *"CitiDirect BE provides one-click access to global transaction capabilities"* across accounts, payments, receivables, liquidity, trade, FX, and reporting, in multiple currencies and geographies (CitiDirect product pages and brochure). It is a web-based platform giving "around the globe, centralized access to your account information in real time" (citidirect.com).
- **The next generation** ✅ — Citi has publicly stated it is "reimagining its legacy CitiDirect BE platform... CitiDirect is being totally reengineered from the ground up" (Citi insights page, "Transformation is on the Rise"), and client-facing FAQ documents describe the migration to a **"next-gen CitiDirect"** digital banking experience with a transitional period during which some payment methods still require final submission in the CitiDirect BE screens (Citi client FAQ PDFs, 2025–26 era ⚠ dated by document availability). The next-gen platform is a flagship of the modernization agenda in §8.
- **The "BE" suffix and the channel lineage** ⚠ — Citi's public material does not expand the "BE" acronym; this guide treats it as the next-generation marker of the platform family (the Microsoft 2009 release itself describes CitiDirect BE as "next-generation"), and the pre-BE CitiDirect web channel had served corporate clients for years before the 2009 relaunch ✅/⚠ (the original launch year is not pinned; Citi's own material describes the platform family as having served clients "for decades" ⚠). The lineage therefore runs: CitiDirect (web channel) → CitiDirect BE (2009 relaunch, Microsoft collaboration, white-label capability) → next-gen CitiDirect (announced reengineering, §8 flagship). Each step is a *channel* generation; none of them names the payments engine behind the screens ⚠.

### 2.3 CitiDirect BE Mobile

The mobile extension of the platform is public and verifiable ✅:

- **CitiDirect BE Mobile app** (Google Play: `com.citi.mobile.cdbe`) — payment review, authorization, initiation, and release from mobile; streamlined login with **biometric authentication in approved markets**; ability to approve/reject new users; payment status, balances, and intraday cash positions; push notifications ✅ (app store listing).
- **CitiDirect BE Mobile Token** — QR-code and biometric login for the desktop platform via the mobile app ✅ (Citi TTS "get started with mobile token" page).
- The mobile channel is a maker/checker enabler: authorizing executives can approve payments away from the desk — the same entitlement pattern documented in the [DBS guide](dbs_software_systems_guide.md) §10.5 for IDEAL, and the standard pattern for corporate banking channels ⚠ (structural).

The security model in practice binds the desktop session to the phone: the Mobile Token generates a QR code and biometric login for the desktop platform, so a payment release on the road requires possession of the registered device in addition to the operator's credentials — a possession-plus-knowledge second-factor pattern structurally identical to the token/OTP model documented for DBS IDEAL in the [DBS guide](dbs_software_systems_guide.md) §10.5 ⚠ (structural — Citi's exact token algorithms and device-binding details are not public).

### 2.4 WorldLink: Cross-Border Payments

**WorldLink Payment Services** is Citi's cross-border payment and disbursement service, delivered through CitiDirect and as an integrated/API offering ✅:

- **Public facts** ✅ (CitiDirect product pages): WorldLink offers **funds transfers in over 135 currencies covering more than 150 countries**, **remote cheques in over 30 countries**, and **on-site cheques**; it is initiated and tracked online, and is described as giving clients "complete control over cross-border payments via the Web from anywhere in the world."
- **Who uses it** ✅/⚠ — WorldLink is marketed to corporates, financial institutions, NGOs, and government entities for global disbursements (payroll, supplier payments, claims, and beneficiary payments in local currency). The "emerging payments" positioning — paying into markets where the client has no local account — is the platform's classic use case ⚠ (product positioning, structurally documented across TTS marketing material; the specific client-segment mix is not published).
- **Mechanics** — WorldLink is a payment *initiation and disbursement* service layered on Citi's correspondent banking network and local clearing rails; the settlement mechanics (nostro accounts, SWIFT MT/ISO 20022 messages, local ACH/RTGS) are the domain of the [Payment Rails Guide](payment_rails_guide.md) and are not re-derived here.

**How a WorldLink disbursement runs** ⚠ (structural reconstruction: the product facts above are ✅-verified, the step flow is the standard disbursement pattern): (1) the client uploads or enters a payment file in CitiDirect BE — or integrates host-to-host/API (§2.6) — specifying the beneficiary and the local-currency payout; (2) WorldLink routes the instruction through Citi's correspondent network and in-country clearing arrangements; (3) the beneficiary receives local currency in-country, or a remote cheque where the market requires one; (4) the client tracks status and confirmation in CitiDirect BE. The layer that actually moves the money — Citi's payments hub, the nostro relationships behind each corridor, and the local clearing memberships — is the ⚠ engine layer of §12.

### 2.5 Payments and Trade Services (Cross-References)

The TTS payments and trade estate beyond CitiDirect BE and WorldLink is documented by cross-reference, per this series' division of labour:

- **Rails mechanics** — [Payment Rails Guide](payment_rails_guide.md): SWIFT messaging (MT and ISO 20022), SWIFT GPI tracking, correspondent banking and nostro mechanics, ACH/RTGS, and real-time rails. Citi participates in SWIFT GPI ⚠ (structural — GPI adoption is universal among top-tier transaction banks; not independently verified in this pass) and runs ISO 20022 for cross-border flows ⚠ (structural for a bank of Citi's class).
- **Trade finance** — [Trade Finance Guide](trade_finance_guide.md) and [Trade Finance Systems Guide](trade_finance_systems_guide.md): documentary credits, guarantees, collections, and supply chain finance platforms. Citi's trade products (LCs, guarantees, SCF) are initiated through CitiDirect BE and the TTS trade modules ⚠ (product structure; engine names not public).
- **Securities Services** — custody and fund services run under Services/TS alongside TTS ⚠ (division structure verified; platform internals not public; the custody-modernization angle is documented in Citi's 2026 "Custody+" launch ✅/⚠ — near/real-time custody solutions announced 2026, per Citi press; flagged as newly announced and not deeply verified in this pass).

### 2.6 The TTS Integration Layer

Beyond the screens, the TTS estate connects to client systems and to Citi's own engines — the integration layer is partly public, partly not:

- **Host-to-host and ERP integration** ⚠ — corporate clients integrate CitiDirect-style channels with their ERP/treasury workstations (SAP, Oracle-class) via host-to-host file transfers, SWIFT, and APIs. This is the standard TTS pattern (documented in the [Payment Rails Guide](payment_rails_guide.md)); Citi's specific integration catalogue is not published ⚠.
- **CitiConnect** ⚠ — Citi's public API/connectivity brand for TTS integration is widely referenced in TTS marketing material, but was **not independently verified in this pass**; treat it as the reported name of the API layer, flagged (§12).
- **White-label capability** ✅ — the Microsoft 2009 release states CitiDirect BE is "optimized for the white-label business model" — i.e., Citi licenses the platform to other financial institutions to run under their own brand. This is the verified ancestor of Citi's bank-client (FI) TTS business, which is exactly the kind of relationship the §10 worked example exercises.
- **Security products** ✅/⚠ — the mobile token and biometric login are verified (§2.3); the broader "CitiDirect Security" product family (hardware tokens, entitlement administration) is publicly marketed ⚠ but not exhaustively verified here.

### 2.7 The TTS Client Operating Model (Structural)

The day-to-day operating model of a TTS client like Cymbal Bank (§10) runs on a pattern that is standard across the industry and only partly visible in Citi's public material ⚠:

- **Onboarding and integration** — the FI client is onboarded under the KYC discipline of §10.3, then connected to CitiDirect BE via credentials, host-to-host files, SWIFT, or API (§2.6); the integration catalogue is not published ⚠.
- **The operating day** — operators work in maker/checker pairs; liquidity managers monitor intraday positions; the treasury team initiates FX and payments; every action is entitlement-controlled (§2.3) and auditable ⚠ (structural).
- **The FI dimension** — a bank client adds two layers a corporate client does not have: its own downstream client base (Cymbal's corporates) and its own nostro/vostro book with Citi (§10.2). This is the verified white-label heritage of the platform (§2.6): Citi runs the engine, the FI client's brand faces the end customer ✅/⚠ (white-label capability verified in the Microsoft 2009 release; the current FI-client product catalogue is not published).
- **Reporting and reconciliation** — statements, intraday positions, and GPI tracking data flow back through CitiDirect BE; the reconciliation cycle is worked out in §10.7.

The operating model is the *contract* between the verified surface (CitiDirect BE) and the unverified engine (§12): everything the client touches is public and documented; everything that executes the payment is not.

---

## 3. Markets Platforms: Citi Velocity and Lava

### 3.1 Citi Velocity: The Institutional Trading Platform

**Citi Velocity** is Citi's client-facing markets platform ✅:

- **What it is** ✅ — "Citi Velocity is the award-winning content, data, analytics and trading platform for Institutional and Corporate clients" (citivelocity.com). Citi's Markets/FX pages describe the **Citi Velocity Trading platform** as "an intuitive, efficient, end-to-end FX solution" with client-configurable user permissioning across entities ✅.
- **Mobile** ✅ — the **Citi Velocity app** (iOS App Store; Android equivalent) provides trading tools, market data, and research content feeds to institutional users on mobile.
- **Velocity 3.0 (2023)** ✅ — in February 2023 Citi announced it had **consolidated its portfolio of eFX trading platforms into Velocity 3.0, a new single-dealer platform (SDP)** (BusinessWire): Velocity 3.0 delivers **CitiFX Pulse, Citi Velocity Trading, Citi Velocity Research, and CitiFX Click** in one platform, rebuilt on **HTML5** on a strategic web codebase, accessible "across all popular operating systems from any browser" and deployable "across devices — whether web, desktop, or mobile." Performance claims in the release: 50% faster cold start, 71% faster deal booking, 94% faster order placement response ✅ (as reported by Citi).
- **Awards** ✅ (as reported) — Global Finance FX Tech Awards 2023 "Best Bank Platform" (Velocity 3.0); FX Markets e-FX Awards 2022 "Best Single-Dealer Platform" (Velocity 2.0).
- **The 2023 performance numbers, read carefully** ✅/⚠ — the release's claims (50% faster cold start, 71% faster deal booking, 94% faster order placement response) are Citi's own before/after figures for the new HTML5 codebase, not third-party benchmarks; an architect should treat them as vendor-reported targets and re-measure in acceptance ⚠ (single-source).

### 3.2 The e-FX Lineage: CitiFX Pulse → Velocity 3.0

The lineage behind the brief's "CitiFX Velocity" reference: Citi's classic e-FX platforms were **CitiFX Pulse** (corporate clients), **Citi Velocity Trading** and **Citi Velocity Research** (institutional clients), and **CitiFX Click**; the 2023 consolidation folded them into Velocity 3.0 ✅ (BusinessWire names exactly these four as the components of Velocity 3.0). **The specific pre-2023 branding "CitiFX Velocity" was not independently confirmed in this pass** ⚠ — Citi's own 2023 release does not use that name for the consolidated platform; treat "CitiFX Velocity" as the informal/common reference to the older institutional e-FX platform, flagged, unless a primary source is produced. Velocity 3.0 is the FX/rates/LD-facing SDP; the brief's "rates/LD" coverage is consistent with Citi's multi-asset client platform positioning ⚠ (citivelocity.com describes content/data/analytics/trading without naming asset classes exhaustively; FX is the documented core).

### 3.3 Lava and LavaFlow: The Equities EMS

**Lava** is the execution-management system (EMS) heritage in Citi's equities business, with a verified — and commonly misdated — acquisition history:

- **Founded 1999** ✅ — Lava Trading Inc. was founded in 1999 as a provider of stock-trading technology to hedge funds and other institutional groups (WSJ, Hedgeweek, TA Associates).
- **Acquired by Citigroup in 2004, not 2007** ✅/❌ — the WSJ reported "Citigroup to Buy Lava Trading" in July 2004; TA Associates (the private-equity backer) records that Lava "became an independent subsidiary of Citigroup Inc. in August 2004"; Hedgeweek confirms the 2004 agreement. **The commonly cited "2007" acquisition year is disputed by the record** ❌ — this guide follows the 2004 primary/secondary sources and flags the 2007 claim as unsubstantiated (see §11). The reported price was roughly **US$500M** ⚠ (tradersmagazine; single-source, flagged).
- **LavaFlow** ✅/⚠ — Lava's EMS product, **LavaFlow**, became known for its market-participant ID (MPID) concept for order attribution; it was widely used by sell-side and buy-side desks in the 2000s (tradersmagazine). Specifics of LavaFlow's current role inside Citi's equities stack are **not public** ⚠.
- **The dark-pool chapter** ⚠ — Citi used Lava to build an ECN/dark-pool presence in the mid-2000s and later exited that business ("Lava Cools As Citi Exits Dark Pool Trading", tradersmagazine) ⚠ (dated secondary reporting; the exact exit timeline is not re-verified here).

### 3.4 FIX and Connectivity (Cross-Reference)

The FIX protocol mechanics — session management, order flow, and the standard itself — are **not re-derived here**; they live in the [FIX Protocol Guide](fix_protocol_guide.md). For this guide's purposes: Citi Velocity and the Lava estate connect to clients and venues over FIX and proprietary APIs ⚠ (structural — an SDP/EMS of this class is FIX-connected as a matter of industry practice; Citi does not publish its connectivity catalogue), and the OMS/EMS/pricing-engine internals behind Velocity 3.0 and Lava are non-public (§12).

### 3.5 The Markets Stack: Public vs Not

| Component | Status in the public record |
|---|---|
| Client single-dealer platform: **Citi Velocity / Velocity 3.0** (web/desktop/mobile, HTML5) | ✅ verified (Citi/BusinessWire 2023) |
| Corporate e-FX: **CitiFX Pulse** (now inside Velocity 3.0) | ✅ verified (named in 2023 release) |
| Institutional trading/research surfaces: **Citi Velocity Trading / Research** | ✅ verified (named in 2023 release) |
| Equities EMS heritage: **Lava / LavaFlow** (founded 1999, acquired 2004) | ✅ verified (acquisition); ⚠ current role |
| FIX and API connectivity to clients/venues | ⚠ structural (class certain, catalogue not public) |
| Pricing engines, e-trading OMS, market data infrastructure, algorithmic stack | ⚠ not public (§12) |
| The brief's "rates/LD" coverage of Velocity | ⚠ consistent with positioning; FX is the documented core |
| Market data and research content feeds on Velocity | ✅ positioning verified; ⚠ content-platform internals |
| Entitlement/permissioning model (client-configurable user permissioning) | ✅ (Citi FX page wording); ⚠ engine |
| Algorithmic execution and post-trade connectivity on Velocity/Lava | ⚠ not public (§12) |
| FIX/REST API catalogue for markets connectivity | ⚠ structural (class certain, catalogue not public) |

The table's dividing line is the guide's general one: every row above the line is a named, marketed Citi product; every row below it is a *class* of system that the products require to function, whose vendor identities and internal names are not published (§12). For the architect this means the markets estate is describable at the surface — screens, workflows, message protocols — and opaque at the engine: pricing, risk, and execution internals.

---

## 4. Cards Business: Co-Brand Economics

### 4.1 The Co-Brand Model and Citi's Portfolio

Co-brand (or affinity) cards are credit cards issued in partnership with a brand — an airline, retailer, or merchant — where the **partner contributes its customer base, marketing, and purchase economics, and the issuer contributes the balance sheet, capital, card management, and regulatory infrastructure**. The economics are shared: the partner typically earns a share of interchange, interest, and fees (often in the form of loyalty currency purchases — e.g., the airline "buys" miles from the issuer), and the issuer keeps the credit risk and the residual. This is the standard co-brand structure; the mechanics of the universal-bank model that houses it are in the [Universal Banking Model Guide](universal_banking_model_guide.md). **Note:** a search of that guide for "co-brand/affinity" content returned no dedicated section in this pass ⚠ — the repo's co-brand content lives, for now, in the verified Citi facts below.

Citi is one of the largest co-brand issuers in the United States; its branded-cards portfolio (AAdvantage, Costco, Home Depot, Best Buy, Exxon and others ⚠ portfolio composition beyond the two verified below is inferred from public marketing, not exhaustively verified) sits inside **US Personal Banking** (§1.5).

### 4.2 American Airlines: The AAdvantage Partnership

The American Airlines relationship is the longest-running and most consequential co-brand in Citi's portfolio, and its public facts are verified ✅:

- **>37 years of partnership** ✅ — American Airlines and Citi announced (December 2024, AA newsroom) the extension of their "more than 37-year co-branded partnership" for another decade.
- **Exclusive issuer from 2026** ✅ — under the extended agreement, **Citi becomes the exclusive issuer of the AAdvantage co-branded card portfolio in the U.S.** in 2026, ending the decade-long dual-issuer arrangement in which Barclays also issued AAdvantage cards (AA newsroom; Business Traveller; the precise conversion date of April 24, 2026 is reported by specialist press ⚠).
- **Why it matters to the systems map** — the portfolio runs on Citi's card-management and servicing platforms (§4.4 ⚠), and the partnership economics (mile purchases, Admirals Club access, elite-status fast tracks — AA and Citi expanded the partnership to American's Admirals Club network in 2026 ✅ per AA newsroom) are among the largest loyalty-purchase arrangements in U.S. cards ⚠ (magnitude inferred).

### 4.3 Costco: The 2016 Portfolio Acquisition

The Costco relationship is the cleanest public example of co-brand portfolio economics changing hands ✅:

- **February 2016** — Citi announced a definitive agreement with **American Express to acquire Costco's U.S. co-brand credit card portfolio** (Citi press release) ✅.
- **June 2016 close** — the transaction closed and Citi **converted the acquired Amex accounts to Visa** — the **Costco Anywhere Visa Card by Citi** — with **more than US$10.5 billion of credit card receivables** acquired ✅ (Citi/marketscreener; electronicpaymentsinternational).
- **The systems angle** ✅/⚠ — the 2016 conversion was a mass account-migration programme: re-issuance of ~11M+ cards ⚠ (card count reported in press, flagged), re-platforming of the portfolio onto Citi's card systems, and new scheme rails (Visa vs Amex) — a classic "portfolio conversion" programme whose execution systems are not public ⚠ (§12).

### 4.4 The Systems Underneath (⚠)

Citi does not publicly name its card-management platform, authorization engine, or card fraud-detection systems ⚠ — these are the §12 non-public items. What is structurally certain (⚠): the co-brand books run on Visa/Mastercard scheme rails; servicing, collections, and loyalty accounting (miles/points accrual and settlement) are core card-platform functions; and the card ledger is part of the wider consumer banking estate (§6). The [Payment Rails Guide](payment_rails_guide.md) documents the scheme/clearing mechanics that are not re-derived here.

### 4.5 The Card Transaction Lifecycle (Structural)

For the architect, the co-brand economics above translate into a transaction lifecycle whose *classes* of system are standard, even though Citi's instances are not public ⚠:

1. **Authorization** — a Costco member swipes the Costco Anywhere Visa: the merchant acquirer routes the authorization to the scheme (Visa), which routes to Citi's authorization engine; the card ledger is checked for credit, fraud scoring runs in real time ⚠ (class certain; Citi specifics in §12).
2. **Clearing and settlement** — transaction files flow through the scheme's clearing; Citi settles with the acquirer through the scheme settlement systems; the [Payment Rails Guide](payment_rails_guide.md) documents these mechanics (not re-derived here).
3. **Loyalty accounting** — for AA cards, miles accrue on every dollar; Citi records a loyalty liability and periodically purchases miles from American (the economics in §4.1) ⚠ (structural; the AA/Citi purchase agreement terms are confidential).
4. **Billing, servicing, collections** — statements, payments, disputes, and collections run on the card servicing platform ⚠.
5. **The portfolio-conversion pattern** — the 2016 Costco migration (§4.3) is the worked example: mass re-issuance, account re-platforming, scheme switch (Amex→Visa), and loyalty-program rebranding in a single cutover programme ⚠ (execution specifics not public).

### 4.6 The 2026 AAdvantage Conversion (⚠)

The counterpart to the 2016 Costco migration is the 2026 AAdvantage conversion — the mirror-image programme in which Citi *gains* the exclusive portfolio:

- **What happens** ⚠ — from April 24, 2026 (specialist-press date, flagged), Citi becomes the exclusive U.S. issuer of AAdvantage co-branded cards, ending the decade-long dual-issuer arrangement under which Barclays also issued AAdvantage cards ✅/⚠ (AA newsroom confirms exclusivity from 2026; the precise date is specialist-press reporting).
- **What it implies systemically** ⚠ — the Barclays-issued AAdvantage accounts must migrate onto Citi's card-management and servicing platforms (§4.4): account conversion, re-issuance, scheme-rail continuity, miles-ledger transfer into Citi's loyalty accounting, and the servicing/collections handover — the same portfolio-conversion pattern §4.5 step 5 describes, in reverse direction ⚠ (execution specifics not public).
- **Why it is flagged** — neither Citi nor American Airlines publishes the conversion programme's systems plan; the fact of the migration is verified, the mechanics are the standard portfolio-conversion class ⚠.

---

## 5. Wealth Platforms

### 5.1 Citi Global Wealth: The Division

Wealth became one of the five divisions in the 2021 redesign, and Citi's wealth ambitions have been a stated strategic priority since ✅:

- **Citi Global Wealth** is the division name; it spans the private bank, affluent (Citigold), and broader wealth businesses ✅ (Citi reporting; the private-banking heritage is documented on Citi's heritage pages: "Citi's private banking roots date back to the 1820s"; **Citigold was inaugurated in Hong Kong in 1982** ✅).
- **Andy Sieg** has been **Head of Wealth since September 2023** (joined from Merrill, where he was President of Merrill Wealth Management) ✅ (Citi Private Bank bio; Citi leadership).
- Wealth sits on its own platform estate, separate from retail banking — the division-level separation is verified; the platform names are not public ⚠.

### 5.2 Platform Landscape (⚠) and Cross-Reference

The wealth platforms (client portals, advisory/workstation tools, portfolio accounting, and the digital wealth surfaces in the Citi mobile app) are **not publicly named** ⚠ — they belong in §12. The domain mechanics — private-bank operating models, client reporting, and platform classes — are **not re-derived here**; they live in the [Private Banking Guide](private_banking_guide.md). The [DBS guide](dbs_software_systems_guide.md) §8 and the [Wealth Management Guide](wealth_management_guide.md) in this repository document the platform classes Citi's estate belongs to ⚠ (structural).

### 5.3 The Wealth Digital Surfaces (⚠)

The client-visible wealth surfaces are branded and public — the Citi mobile app's wealth section, Citigold online, and the Citi Private Bank portal — but the platforms behind them are not named ⚠. Structurally, a wealth estate of this class contains: client onboarding and suitability/risk profiling; portfolio accounting and reporting; advisory and model-portfolio tooling; and the digital-advice surfaces in the mobile app ⚠ (class certain; Citi specifics in §12). The domain mechanics for each class are cross-referenced in §5.2 rather than re-derived here.

---

## 6. Core Banking and the Legacy Estate

### 6.1 The Mainframe Estate (⚠)

The single most important honest statement about Citi's core: **Citi does not publicly name its core banking system or its mainframe estate** ⚠. What is publicly knowable:

- **"Decades-old technology"** — the Fraser-era transformation narrative repeatedly cites legacy technology as the root of Citi's operational and compliance problems (widely reported; the phrase and sentiment are on the public record in earnings calls and the 2020–22 transformation disclosures) ✅/⚠ (the *sentiment* is verified; the precise quote is flagged).
- **Structural inference** ⚠ — a bank of Citi's age (1812 lineage, mainframe-era automation from the 1960s–80s) runs a mainframe-based core estate for deposits, loans, cards, and payments; the consent-order record (§8) corroborates that the data and reporting layers around this estate were deficient. The specific systems (IBM Z hardware estate, the core application names, the ledger engines) are **not public** ⚠.
- **Repo cross-reference** — this repository has no dedicated mainframe systems guide under technology/ (only a mainframe file-sharing notification file); the mainframe-era core mechanics (posting, ledgers, batch, core vendor classes) are documented in the [Core Banking Systems Guide](core_banking_systems_guide.md), and U.S. core context in [US Bank Core Systems Guide](us_bank_core_systems_guide.md) — cross-referenced, not re-derived.

### 6.2 Modernization: Retire-and-Simplify

The modernization programme is publicly measurable through spend and retirement statistics ✅:

- **Technology spend** ✅ — Citi reported approximately **US$9.9B (2021), US$11.2B (2022), US$12B (2023), and US$11.8B (2024)** in annual technology spend (CIO Dive; infotechlead; bankingdive, citing CFO Mark Mason). More than half of the 2023 spend went to modernization initiatives ✅ (CIO Dive).
- **Legacy retirement** ✅/⚠ — Citi retired **~300 applications in a single year (2023 reporting cycle)** and **~390–400 legacy applications in 2023** (CIO Dive: "retired nearly 400 legacy apps"; okoone: "390 in 2023" — the exact count varies by reporting cut ⚠); the decommissioning programme has been running since ~2021, with Q4 2024 described as the "best quarter" for legacy decommissioning since it began ✅ (eFinancialCareers, citing Citi's Q4 presentation).
- **The stated goal** ✅ — Fraser: *"At the end of the work, we will have a simpler firm that can operate faster, better serve our clients and unlock value for our shareholders"* (CIO Dive). The transformation themes per CFO Mark Mason: "platform and process simplification, security and infrastructure modernization, client experience enhancements and data improvements" ✅ (CIO Dive).

**Reading the numbers** — the spend series (~US$9.9B/2021 → US$11.8B/2024) is Citi-reported and mixes run-the-bank and modernization spend; the retirement counts (~300 in one reporting cycle, ~390–400 in 2023) vary by cut because Citi reports by fiscal period and programme milestone rather than by a single published inventory ⚠ (CIO Dive, okoone, eFinancialCareers as cited in §14). The two series together — rising spend, rising retirement — are the publicly measurable proxy for the "retire and simplify" strategy; the *target architecture* those retirements serve is not published ⚠ (§12).

### 6.3 Cross-References

Core mechanics (posting, interest, ledgers, core vendor classes, strangler-fig modernization) are **not re-derived here** — see [Core Banking Systems Guide](core_banking_systems_guide.md), [Posting Engine Core Banking Guide](posting_engine_core_banking_guide.md), and [Interest Calculation Engine Guide](interest_calculation_engine_guide.md). Citi's specific core-identity question (vendor vs in-house; which legacy platforms) is unresolved in public sources and sits in §12 ⚠.

---

## 7. Data and AI

### 7.1 Data Governance: The Consent-Order Shadow

Citi's data story is dominated by the regulator-mandated remediation that put data governance into the public record ✅:

- The **October 2020 OCC and Federal Reserve actions** (§8.1) explicitly cite **data governance deficiencies, data quality errors, and failure to produce timely and accurate management and regulatory reporting** as core findings ✅ (OCC release; OCC Cease and Desist Order ea2020-056).
- The **November 2022 resolution-plan shortcoming** identified by the Fed and FDIC was again about **data quality and data management** — "previously identified by the Board in its October 2020 enforcement action" ✅ (Federal Reserve release, 23 Nov 2022).
- The risk-data/BCBS 239 context (risk data aggregation and reporting) is **not re-derived here** — see the [Enterprise Risk Management Guide](enterprise_risk_management_guide.md).

### 7.2 AI at Scale

Citi's AI posture is increasingly public ✅:

- **GenAI coding tools for ~30,000 developers** ✅ — CEO Jane Fraser said on the Q4 2024 earnings call that Citi had armed **30,000 developers with generative AI coding tools** as part of the modernization push (bankingdive; widely reported).
- **Spend framing** ✅ — CFO Mark Mason described the 2024 tech spend as investing in "digital innovation, product development, client experience and cybersecurity" (bankingdive).
- **Scope of AI use** ⚠ — Citi's internal AI/ML estate (models, platforms, governance) beyond the coding-assistant headline is **not public** ⚠; Citi's AI disclosures are compliance-flavoured (responsible-AI statements) rather than architectural.

### 7.3 Cross-References (AI/LLM Guides)

The AI/LLM platform mechanics are **not re-derived here** — the repository's guides under [technology/ai_llm/](../technology/ai_llm/) cover the relevant classes: [Enterprise AI Platforms](../technology/ai_llm/enterprise_ai_platforms_guide.md), [Production-Ready LLM Agents](../technology/ai_llm/production_ready_llm_agents_guide.md), [LLM Evaluation Frameworks](../technology/ai_llm/llm_evaluation_frameworks_guide.md), and [Enterprise AI Gateway](../technology/enterprise_ai_gateway_guide.md). Citi's specific AI platform choices are not public ⚠ (§12).

---

## 8. Transformation and Risk Context: 2020 Consent Orders, Modernization, and Cloud

### 8.1 The October 2020 Consent Orders (Verified)

On **October 7, 2020**, U.S. regulators took coordinated action against Citi — the pivotal regulatory event of the modern era, fully documented in primary sources ✅:

- **OCC: US$400M civil money penalty against Citibank, N.A.** (Sioux Falls, South Dakota charter) ✅ — the OCC assessed the penalty "related to deficiencies in enterprise-wide risk management, compliance risk management, data governance, and internal controls," based on "the bank's unsafe or unsound banking practices for its long-standing failure to establish effective risk management and data governance programs and internal controls," including a violation of **12 CFR Part 30, Appendix D** (the OCC Heightened Standards) ✅ (OCC News Release 2020-132, 7 Oct 2020). The OCC simultaneously issued a **cease and desist order** requiring "broad and comprehensive corrective actions," **non-objection before significant new acquisitions**, and reserved authority for business restrictions or management/board changes ✅ (same release).
- **Federal Reserve: cease and desist order against Citigroup Inc.** (the holding company) ✅ — the Fed's order requires the firm "to enhance its firm-wide risk management and internal controls," citing that the firm "has not taken prompt and effective actions to correct practices previously identified by the Board in the areas of compliance risk management, data quality management, and internal controls" ✅ (Federal Reserve press release, 7 Oct 2020).
- **The US$400M figure, precisely** ✅ — the $400M is the **OCC's civil money penalty** (paid to the U.S. Treasury); the Fed's action carried **no separate monetary penalty**. The press summarised this as "US$400M in penalties" — accurate as a total only because the Fed fine was zero; this guide states the breakdown rather than the shorthand ⚠ (the "combined penalties" framing is common in press but imprecise).
- **Cross-reference** — the enterprise-risk-management mechanics this action sits on are in the [Enterprise Risk Management Guide](enterprise_risk_management_guide.md) (not re-derived here).

### 8.2 The Resolution-Plan ("Living Will") Saga

The consent orders did not end the regulatory pressure; the resolution-plan record extended it ✅:

- **December 2020** — the FDIC and Fed **rejected Citigroup's "living will" (resolution plan)**, with the FDIC calling Citi's data controls "deficient" — a downgrade from the prior pass (FT; Yahoo Finance) ✅.
- **November 2022** — the Fed and FDIC identified a **shortcoming in Citi's 2022 resolution plan**, again tied to **data quality and data management concerns previously identified in the October 2020 enforcement action**, requiring urgent remediation ✅ (Federal Reserve release bcreg20221123a; Reuters, 23 Nov 2022).
- The living-will/resolution-planning mechanism itself is the domain of the [Enterprise Risk Management Guide](enterprise_risk_management_guide.md) ⚠ cross-ref.

### 8.3 Digital-Transformation Programs

The transformation agenda is publicly measurable, with specifics flagged where disclosure stops:

- **Scale** ✅ — ~US$10–12B/yr technology spend since 2021 (§6.2); the 2020–2022 "transformation" plan under Fraser targeted simplification and efficiency (public investor-day framing; the *specific* cost-save targets are flagged ⚠ — Citi's efficiency-ratio targets are in earnings disclosures, not re-verified here).
- **Legacy modernization** ✅ — ~400 legacy applications retired in 2023; decommissioning since 2021 (§6.2).
- **Client-platform modernization** ✅ — the next-gen CitiDirect reengineering (§2.2) and Velocity 3.0 (§3.1) are the two flagship client-facing examples.
- **What is not disclosed** ⚠ — the multi-year *roadmaps* (system-by-system milestones, the target architecture, the core-replacement decision itself) are not public; Citi has announced no "Atlas-on-AWS"-style named core migration (§12).
- **The Fraser pillars, publicly restated** ✅/⚠ — Fraser's public framing (a simpler firm that operates faster and serves clients better; CIO Dive) and Mason's four themes (platform and process simplification, security and infrastructure modernization, client experience enhancements, data improvements) are on the record (§6.2); the *programme structure* under them — workstreams, sponsors, system-level milestones — is not ⚠.
- **Named, datable proof points** ✅ — next-gen CitiDirect (§2.2), Velocity 3.0 (§3.1), and the 2026 Custody+ launch (§2.5) are the client-facing deliverables the market can hold Citi to; each is a channel/platform modernization rather than a core-replacement event ⚠ (consistent with the record — no named core migration exists).
- **The regulatory driver stays in the picture** ✅ — the 2020 consent orders and the 2022 resolution-plan shortcoming (§8.1–§8.2) remain the binding constraints the transformation must satisfy; Citi's public commentary consistently ties modernization to remediating those findings ⚠ (the causal framing is Citi's own, per earnings commentary).

### 8.4 Cloud Strategy

- **Google Cloud: verified strategic agreement (2024)** ✅ — Citi and Google Cloud announced a **strategic, multi-year agreement to modernize Citi's technology infrastructure** through cloud and AI, including migrating workloads and applications to Google Cloud (Citi press release, 2024; CIO Dive; PR Newswire).
- **Microsoft heritage** ✅ — the 2009 CitiDirect BE launch was built on a multiyear Microsoft collaboration (Microsoft newsroom) — Citi's cloud/tech history is not Google-only.
- **AWS** ⚠ — Citi's use of AWS is widely reported in industry press but was **not independently verified in this pass**; treat Citi's posture as **multi-cloud with Google Cloud as the announced strategic partner** ✅/⚠.
- **Cross-reference** — the bank-cloud themes (regulatory cloud, workload migration, hybrid/residency carve-outs) are documented in the [IBM Cloud Guide](../technology/ibm_cloud_guide.md) and are **not re-derived here** ⚠.
- **What the 2024 agreement actually says** ✅ (Citi press release) — the Google Cloud agreement covers modernizing Citi's technology infrastructure "through cloud and AI," including migrating workloads and applications to Google Cloud — i.e., a workload-migration agreement with an AI component, not a single-cloud mandate; Citi's public language leaves room for other providers ✅/⚠.
- **Why the Microsoft heritage matters** ✅/⚠ — the 2009 CitiDirect BE collaboration (Microsoft newsroom) predates the Google Cloud agreement by fifteen years and shows the estate has always been multi-vendor at the platform level; whether next-gen CitiDirect runs on Google Cloud is not public ⚠.
- **The honest cloud statement** ⚠ — publicly verified: Google Cloud is the announced strategic partner (2024); Microsoft is the verified 2009-era platform collaborator; AWS usage is reported but unverified; the actual workload distribution across clouds is not disclosed (§12).

---

## 9. Singapore Angle

### 9.1 1902: The IBC Arrives

Citi's Singapore story is one of the best-documented franchise histories in the bank, verified across Citi's own pages and Singapore's National Library Board ✅:

- **1902** — the **International Banking Corporation (IBC)** — the Connecticut-chartered bank founded in 1901 to promote trade with Asia — established operations in Singapore, making it **the first American bank in Singapore** ✅ (NLB; Wikipedia; Citi heritage "IBC pioneers trade finance in Asia"; Citi Singapore's own "since 1902" positioning). The precise opening date of **1 July 1902** is reported by secondary sources ⚠ (Wikipedia/LiquiSearch); Citi's own material confirms the year but not the day.
- **The early business** ✅/⚠ — IBC's first Singapore branch stood at 1 Prince Street and was "primarily involved in the trade financing of Malayan rubber and tin exports" ⚠ (Wikipedia; consistent with the heritage page's trade-finance framing — the street address and commodity mix are secondary-source detail).
- **120 years in 2022** ✅ — Citi Singapore ran a "120 Years of Citi in Singapore" programme in 2022, confirming the continuous 1902–2022 franchise (Citibank Singapore site).

### 9.2 The Regional-Hub Role

- **Citi's own words** ✅ (Citigroup global-presence page for Singapore): *"Citi has been in Singapore since 1902 and is today a full-service bank, offering consumers, corporations, governments and institutions in Singapore with a full suite of financial products and services. Singapore is a significant hub for Citi globally where a number of our regional and global client coverage and product teams are based."* Citi Singapore adds that the country "houses a number of our regional and global client coverage and product units, and it is home to many of our Asia and ASEAN business heads" ✅.
- **What the hub hosts** ⚠ — the specific regional units (TTS regional HQ functions, markets desks, the ASEAN cluster leadership) are not exhaustively published; the hub role itself is verified, the unit-by-unit inventory is not ⚠.
- **The full-service footprint** ✅ — consumer, corporate, government, and institutional banking in one franchise — the Singapore version of the universal model (§1.5); the retail side is one of the ~19-country consumer footprint's Asian anchors ⚠.

### 9.3 The MAS Context (Cross-Reference)

- **Licensing** ✅ — Citibank (Singapore) operates under MAS regulation as a full-bank licence holder (MAS Financial Institutions Directory lists Citibank under Full Bank; capitalmarkets.sg confirms MAS-regulated status).
- **D-SIB status** ✅ — MAS's **inaugural list of domestic systemically important banks (D-SIBs), published April 2014, included Citibank** among the seven: DBS, OCBC, UOB, Citibank, Maybank, Standard Chartered, and HSBC (Straits Times; Yahoo Finance; centralbanking.com). D-SIB designation carries additional capital and supervisory measures ⚠ (the specific buffers applicable to Citi Singapore are in MAS notices, not re-verified here).
- **Cross-reference** — the MAS regulatory framework (licensing, technology risk management, outsourcing, D-SIB policy) is **not re-derived here**; it lives in the [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md). Citi Singapore's systems therefore sit inside that framework like every other MAS-licensed bank ⚠ (structural).

---

## 10. Worked Example: Cymbal Bank × Citi — Correspondent Banking and Trade Services

> **Persona note:** Cymbal Bank is the fictional/illustrative mid-size Singapore-based bank used across this repository's worked examples (it is the author's affiliation in the sibling guides). It stands in for a real mid-tier ASEAN bank. Everything in this section is an **illustrative reconstruction** built from the verified Citi products in §2–§3 and the standard correspondent-banking mechanics in the [Payment Rails Guide](payment_rails_guide.md) — the *products* are real, the *scenario* is a model.

### 10.1 The Relationship

Cymbal Bank, a mid-size Singapore-headquartered bank, serves ASEAN corporates with U.S. trade exposure. It holds a correspondent relationship with Citi because Cymbal needs what only a top-tier global bank provides:

- **USD clearing** — Cymbal's clients settle dollar invoices; Cymbal needs a USD clearing bank with scale in New York.
- **Cross-border reach** — Cymbal's clients pay beneficiaries in markets where Cymbal has no presence (WorldLink's core use case, §2.4).
- **Trade finance support** — Cymbal's clients need LCs confirmed by a name the counterparty's bank will trust.

The relationship is a **financial-institution (FI) client** relationship: Cymbal is a *client of Citi's TTS business*, not a Citi subsidiary — the mirror of the correspondent model Citi itself has used since the 1897 forex/correspondent era (§1.3).

### 10.2 The Rails: Correspondent Banking, Nostro Accounts, SWIFT

The mechanics here are standard correspondent banking — documented in full in the [Payment Rails Guide](payment_rails_guide.md) and only mapped to Citi here:

- **Nostro** — Cymbal holds a **USD nostro account at Citibank, N.A.** (New York). From Cymbal's perspective the account is "our money at their bank"; from Citi's it is a vostro ("your money at our bank"). The glossary defines both.
- **Access via CitiDirect BE** ✅ (real product) — Cymbal's treasury and ops teams see the nostro balance, intraday position, and statements in CitiDirect BE (§2.2), and initiate/release payments with maker/checker controls and the CitiDirect BE Mobile app for approvals (§2.3).
- **Messaging** — payment instructions flow over **SWIFT**: MT103 (customer transfer) / MT202 (bank transfer), and — for the ISO 20022 era — pacs.008/pacs.009 under CBPR+; Cymbal's core banking system and Citi's payments estate exchange these via their respective SWIFT interfaces ⚠ (Citi's SWIFT gateway specifics are not public, §12; the message standards themselves are in the [Payment Rails Guide](payment_rails_guide.md)).
- **GPI tracking** — cross-border payments carry the SWIFT GPI tracker reference so Cymbal's ops can trace end-to-end status in CitiDirect BE ⚠ (GPI mechanics: [Payment Rails Guide](payment_rails_guide.md); Citi's GPI membership is structural, §2.5).
- **The daily cycle** — Cymbal's back office reconciles the CitiDirect BE nostro statements against its own ledgers; Citi's intraday reporting gives Cymbal the real-time position the 2009-era "real time" CitiDirect promise describes (§2.2).

### 10.3 KYC and Onboarding

Onboarding a correspondent is a KYC programme in its own right, and for a U.S.-chartered bank it is a **regulated** one (the USA PATRIOT Act's correspondent-account due-diligence rules apply to U.S. banks' foreign FI accounts ⚠ structural):

- **The onboarding file** — Cymbal provides: ownership structure, licences (MAS full-bank licence, §9.3), AML/CFT programme documentation, sanctions policy, and the classic **correspondent banking questionnaire**; Citi's FI due-diligence team assesses the jurisdiction, ownership, and respondent-bank risk ⚠ (the exact Citi questionnaire is confidential, §12; the class of information is standard across the industry).
- **Screening** — Cymbal's name, principals, and transactions are screened against sanctions lists (OFAC and others) on an ongoing basis; the screening-engine mechanics are in this repo's [FircoSoft Guide](fircosoft_guide.md) and [Enterprise Risk Management Guide](enterprise_risk_management_guide.md) ⚠ (Citi's specific screening vendor is not public, §12).
- **Ongoing monitoring** — transaction monitoring on the nostro flows, periodic KYC refresh (the "evergreen" review), and the respondent-bank obligations Cymbal itself carries for its own downstream correspondents ⚠ (structural).

### 10.4 Trade Services

The trade angle exercises Citi's TTS trade estate (§2.5):

- **The scenario** — Cymbal's client, a Singapore exporter, sells machinery to a U.S. importer. The U.S. importer's bank issues an **LC** in favour of the exporter; **Cymbal advises the LC** and, where the exporter wants the U.S. bank's risk replaced by a stronger name, arranges for the LC to be **confirmed** — with Citi as confirming bank ⚠ (confirmation economics: the confirming bank takes the issuing bank's risk for a fee; mechanics in the [Trade Finance Guide](trade_finance_guide.md)).
- **Cymbal's own issuance** — for Cymbal's importing clients, Cymbal issues LCs and uses **Citi's trade processing** via the TTS trade modules in CitiDirect BE ⚠ (product structure per §2.5; engine names not public).
- **Documentary flow** — documents against payment/acceptance, discrepant-document handling, and settlement through the Cymbal–Citi nostro: the message types (MT700/MT707 for LCs, MT400/MT202 for settlement) are the domain of the [Trade Finance Guide](trade_finance_guide.md) and [Trade Finance Systems Guide](trade_finance_systems_guide.md), cross-referenced, not re-derived.
- **SCF angle** ⚠ — where Cymbal's clients have receivables from U.S. buyers, the supply-chain-finance platforms documented in the [Supply Chain Finance Guide](supply_chain_finance_guide.md) are the class of system Citi's TTS runs; Citi's specific SCF platform is not public (§12).

### 10.5 The Systems Involved (Mapped to This Guide)

| Step in the relationship | Citi system (verified where marked) | Guide section |
|---|---|---|
| Cymbal sees nostro balances, initiates payments | CitiDirect BE ✅ | §2.2 |
| Treasury approvals on the road | CitiDirect BE Mobile ✅ | §2.3 |
| USD settlement | Citibank N.A. (NY) nostro + SWIFT/ISO 20022 | §2.5, [Payment Rails Guide](payment_rails_guide.md) |
| Beneficiary payouts in exotic currencies | WorldLink ✅ | §2.4 |
| FX hedging of USD exposure | Citi Velocity (SDP) ✅/⚠ | §3.1 |
| LC confirmation/advising | TTS trade modules ⚠ | §2.5, [Trade Finance Guide](trade_finance_guide.md) |
| Sanctions screening of flows | Screening estate ⚠ | [FircoSoft Guide](fircosoft_guide.md), [Enterprise Risk Management Guide](enterprise_risk_management_guide.md) |

The architect's takeaway: **Cymbal's entire Citi relationship runs through two verified client surfaces — CitiDirect BE (payments/liquidity/trade) and Citi Velocity (markets) — over the correspondent rails documented in the Payment Rails Guide.** Everything beneath those surfaces is the ⚠ engine layer of §12. That is the honest shape of the Citi systems map for any FI client.

### 10.6 Step-by-Step: A USD Payment from Singapore to New York

Walk a single payment end-to-end — Cymbal's client in Singapore owes US$500,000 to a U.S. supplier whose account is at a third U.S. bank. Each step is the standard correspondent pattern from the [Payment Rails Guide](payment_rails_guide.md), mapped to the verified Citi surfaces ⚠ (structural; Citi's engine internals are §12):

1. **Instruction** — Cymbal's client instructs Cymbal to pay US$500,000 to the supplier's account at the U.S. bank; Cymbal's treasury operator creates the payment in CitiDirect BE, specifying the beneficiary bank and account ⚠ (Cymbal's own core feeds the channel; the file formats are the standard corporate-banking ones).
2. **Authorization** — the maker/checker pair releases the payment; if the operator is remote, approval happens on CitiDirect BE Mobile with biometric login (§2.3).
3. **Message** — Cymbal's SWIFT interface sends the customer transfer instruction — MT103 in the classic era, pacs.008 under CBPR+ — addressed to Citibank, N.A. New York (Cymbal's USD correspondent) ⚠ (message standards: [Payment Rails Guide](payment_rails_guide.md)).
4. **Screening** — Citi's sanctions/AML screening runs on the payment at intake and again against updates; a false positive stops the flow for investigation (§10.3) ⚠.
5. **Nostro debit** — Citi debits Cymbal's USD nostro (§10.2); the cover leg to the beneficiary's bank runs as MT202 / pacs.009 bank-to-bank transfer ⚠.
6. **The New York leg** — the US$500,000 moves across the U.S. dollar clearing systems — CHIPS (netting) or Fedwire (RTGS) — to the beneficiary's bank, which credits the supplier's account ⚠ (Citi's specific clearing memberships are not published; §12).
7. **Tracking** — the payment carries a GPI tracker reference; Cymbal's ops see end-to-end status in CitiDirect BE and can prove value to the client ⚠ (GPI mechanics: [Payment Rails Guide](payment_rails_guide.md)).
8. **Reconciliation** — at end of day, the nostro statement in CitiDirect BE shows the debit, fees, and the resulting balance; Cymbal's back office matches it against its own ledger (§10.7).

The whole flow touches exactly two verified Citi products — CitiDirect BE (initiation, tracking) and the Citibank N.A. nostro (settlement) — with the payment engine, screening engine, and clearing memberships in the ⚠ layer between them.

### 10.7 Step-by-Step: The Nostro Reconciliation Cycle

The daily discipline that keeps the relationship honest ⚠ (structural — the class of process is universal; Citi's statement formats and tools are not public):

1. **Intraday** — CitiDirect BE shows the live nostro position; Cymbal's liquidity desk manages against payment windows and cut-offs (§10.6 step 6) ⚠.
2. **End of day** — Citi posts the nostro statement (debits, credits, fees, value dates); Cymbal receives it through the channel or SWIFT ⚠.
3. **Matching** — Cymbal's reconciliation matches each statement line to an instructed payment: amount, value date, beneficiary, and charges convention (OUR/SHA/BEN) ⚠ (charges conventions: [Payment Rails Guide](payment_rails_guide.md)).
4. **Breaks** — unmatched items become breaks: missing credits (the cover never arrived), value-date differences (interest impact), and fee mismatches; each is investigated, typically by SWIFT investigation messages ⚠.
5. **Aging and claims** — unresolved breaks age; Cymbal raises claims against Citi (and Citi against Cymbal) under the correspondent agreement; aged breaks feed the monthly nostro-position review ⚠.

The reconciliation cycle is the *control* that makes the §10.2 nostro relationship safe to run at volume — and it is exactly the process that a deficient data estate (the §8.1 findings) makes unreliable at scale ⚠.

### 10.8 Step-by-Step: KYC Due Diligence on Cymbal

Before Cymbal gets the CitiDirect BE credentials of §10.6, it passes through the correspondent-banking KYC programme (§10.3) ⚠ (structural — the standard industry sequence, per the [FircoSoft Guide](fircosoft_guide.md) and [Enterprise Risk Management Guide](enterprise_risk_management_guide.md)):

1. **The file** — Cymbal supplies charter documents, its MAS full-bank licence (§9.3), ownership structure, and management/UBO information.
2. **The questionnaire** — Cymbal completes a correspondent banking questionnaire covering its AML/CFT programme, sanctions policy, correspondent network, and downstream respondent banks ⚠ (Citi's questionnaire is confidential; the class is industry-standard).
3. **Risk rating** — Citi's FI due-diligence team assesses jurisdiction, ownership, respondent-bank, and product risk; the result sets the EDD level and monitoring intensity ⚠.
4. **Screening** — Cymbal's name, principals, and UBOs screen against OFAC and other sanctions lists; hits trigger enhanced review ⚠.
5. **Approval and onboarding** — the relationship is approved at the delegated authority level, the nostro agreement is signed, and the CitiDirect BE entitlements are provisioned (§2.2–§2.3) ⚠.
6. **Evergreen** — periodic KYC refresh re-runs steps 1–4 on a cycle; material changes (ownership, jurisdiction) trigger off-cycle review; ongoing transaction monitoring runs on the nostro flows (§10.6 step 4) ⚠.

The KYC programme is where the §8.1 data-governance findings bite hardest for a U.S.-chartered bank: correspondent due-diligence records must be complete, accurate, and retrievable — precisely the properties the 2020 orders found deficient ✅/⚠ (findings verified; their remediation status is not public).

### 10.9 Step-by-Step: A Documentary Credit with Citi Confirmation

The trade-services flow (§10.4) in full ⚠ (structural — message families and mechanics in the [Trade Finance Guide](trade_finance_guide.md); Citi's engine internals not public):

1. **The contract** — Cymbal's exporter client signs a sale contract with the U.S. importer: payment by irrevocable documentary credit.
2. **Issue** — the importer's bank issues the LC in favour of the exporter and sends the MT700 through SWIFT, with Cymbal as advising bank.
3. **Advise** — Cymbal authenticates the MT700 and advises the LC to the exporter via its trade channels (the CitiDirect BE trade modules for FI-client workflows, §2.5) ⚠.
4. **Confirmation** — the exporter wants the issuing bank's risk replaced by a stronger name; Cymbal requests Citi to add its confirmation. Citi confirms — taking the issuing bank's risk for a confirmation fee — and the exporter ships against the confirmed LC ⚠ (confirmation economics: [Trade Finance Guide](trade_finance_guide.md)).
5. **Presentation** — the exporter presents documents (invoice, bill of lading, packing list, certificate) to Cymbal, which checks them against the LC terms (the UCP 600 document check) ⚠.
6. **Discrepancies** — a late shipment makes the documents discrepant; Cymbal either waives, amends (MT707), or presents on approval; clean documents proceed ⚠.
7. **Settlement** — on complying presentation, the importer's bank pays; the funds flow through the correspondent chain — MT400/MT202-style settlement through the Cymbal–Citi nostro (§10.2) — and the exporter receives payment ⚠.

For the systems map, the LC flow is the trade mirror of §10.6: verified surfaces at the ends (CitiDirect BE trade modules, the nostro), message standards in the middle, and the confirmation/risk engine unnamed in the ⚠ layer (§12).

### 10.10 The Operating-Day Narrative

Put the steps together as a day at Cymbal's back office ⚠ (structural): the morning starts with the nostro position and the day's payment queue in CitiDirect BE; the USD payment window closes against New York cut-offs; the trade team tracks the confirmed LC and the exporter's presentation; by end of day the nostro statement is matched, the breaks are logged, and the quarter's KYC refresh sits in the queue. Every one of those screens is a verified Citi surface or a standard industry process; every engine behind them is the §12 unknown. That is the honest operating picture of correspondent banking on the Citi estate — and, by extension, on any top-tier global bank's ⚠.

---

## 11. Claims Status and Verification Notes

**Verification convention: ✅ = verified in this research pass (primary or secondary sources); ⚠ = flagged (inferred, approximate, single-source, or structural); ❌ = disputed (the public record contradicts the claim); "not public" = Citi does not disclose and no external source exists either way.**

### 11.1 The Claims-Status Table

| Claim | Source | Status |
|---|---|---|
| City Bank of New York founded June 16, 1812; Samuel Osgood first president | Citigroup heritage page "A new bank born in New York"; Wikipedia | ✅ verified |
| July 1865: national charter under the National Bank Act; renamed National City Bank of New York | Citigroup heritage page "A banker's bank"; NLB Singapore | ✅ verified |
| 1955: renamed First National City Bank of New York (after First National Bank merger); 1962 shortening | Modern History Project; Wikipedia; consistent with heritage 1961 usage | ⚠ flagged (secondary) |
| 1976: renamed Citibank, N.A. | NLB Singapore; Simple English Wikipedia; populartimelines | ✅ verified (secondary, consistent) |
| 1998 Citicorp–Travelers merger creates Citigroup; 2012 bicentennial; Fraser first female CEO of a major US bank (1 Mar 2021) | Citigroup heritage index | ✅ verified |
| Citi does business in more than 160 countries and jurisdictions | Citi boilerplate (e.g., BusinessWire CitiFX release, Feb 2023) | ✅ verified |
| Consumer branches in ~19 countries; six US metros | Wikipedia | ⚠ flagged (secondary, approximate) |
| Five divisions post-2021: Services, Markets, Banking, Wealth, US Personal Banking; heads report to CEO | The Banker "Inside Citi's rebuild"; finews.asia | ✅ verified |
| Andy Sieg Head of Wealth since September 2023 | Citi Private Bank bio | ✅ verified |
| CitiDirect BE = Citi's next-generation e-banking platform, launched ~2009, Microsoft collaboration | Microsoft newsroom (15 Sep 2009); CitiDirect pages | ✅ verified (2009 launch) |
| "Next-gen CitiDirect" reengineering of the legacy CitiDirect BE | Citi insights "Transformation is on the Rise"; Citi client FAQ PDFs | ✅ verified (programme); ⚠ dates |
| CitiDirect BE Mobile app (payments, biometrics, push) and Mobile Token | Google Play listing; Citi TTS mobile-token page | ✅ verified |
| WorldLink: funds transfers in 135+ currencies across 150+ countries; remote/on-site cheques; via CitiDirect | CitiDirect product pages (citidirect.com) | ✅ verified |
| Citi Velocity = content, data, analytics and trading platform for institutional/corporate clients; Velocity 3.0 consolidation (Feb 2023) of CitiFX Pulse, Citi Velocity Trading/Research, CitiFX Click; HTML5, web/desktop/mobile | citivelocity.com; citigroup.com Markets/FX; BusinessWire (23 Feb 2023) | ✅ verified |
| Pre-2023 "CitiFX Velocity" platform branding | Not found in this pass (Citi's 2023 release names CitiFX Pulse / Citi Velocity Trading / CitiFX Click) | ⚠ unverified |
| Lava Trading Inc. founded 1999 | WSJ (2004); Hedgeweek; TA Associates portfolio | ✅ verified |
| Lava acquired by Citigroup in **2004** (announced July, closed August 2004); ~US$500M price | WSJ "Citigroup to Buy Lava Trading"; TA Associates; Hedgeweek; tradersmagazine (price) | ✅ verified (2004); ⚠ price single-source |
| Lava acquired by Citigroup in **2007** | No source found; contradicts 2004 record | ❌ disputed |
| American Airlines–Citi co-brand: >37-year partnership extended (2024); Citi exclusive AAdvantage issuer in US from 2026 | American Airlines newsroom (2024, 2026); Business Traveller | ✅ verified |
| Costco: Citi acquired the US co-brand portfolio from Amex (announced Feb 2016, closed Jun 2016; >US$10.5B receivables; Visa conversion) | Citi press release (2016); marketscreener; electronicpaymentsinternational | ✅ verified |
| Technology spend ~US$9.9B/2021, $11.2B/2022, $12B/2023, $11.8B/2024; >half to modernization | CIO Dive; infotechlead; bankingdive (Mason quotes) | ✅ verified (reported) |
| ~390–400 legacy applications retired in 2023; decommissioning programme since 2021 | CIO Dive; okoone; eFinancialCareers | ✅/⚠ verified (count varies by cut) |
| Oct 7, 2020: OCC US$400M CMP against Citibank N.A. + C&D; Fed C&D against Citigroup Inc. (no separate Fed fine) | OCC News Release 2020-132; OCC C&D ea2020-056; Federal Reserve press release (7 Oct 2020) | ✅ verified |
| "US$400M combined penalties" shorthand | Press framing (e.g., Balch); accurate only as OCC-only total | ⚠ flagged (imprecise framing) |
| Dec 2020: FDIC/Fed rejected Citi's living will; FDIC called data controls "deficient" | FT; Yahoo Finance | ✅ verified (reported) |
| Nov 2022: resolution-plan shortcoming tied to data quality/data management from Oct 2020 action | Federal Reserve release bcreg20221123a; Reuters | ✅ verified |
| Google Cloud strategic multi-year agreement (2024) to modernize infrastructure and migrate workloads | Citi press release (2024); CIO Dive; PR Newswire | ✅ verified |
| AWS usage at Citi | Industry press only; not verified in this pass | ⚠ unverified |
| Citibank in Singapore since 1902 (IBC; first American bank in Singapore); 120-year anniversary 2022 | Citi heritage 1902 page; Citigroup global-presence page; NLB; Citi Singapore | ✅ verified (year); ⚠ 1 July 1902 date secondary |
| Exact 1 July 1902 opening date | Wikipedia/LiquiSearch | ⚠ flagged (secondary) |
| Singapore = significant Citi hub; regional and global product teams based there; full-service bank | Citigroup global-presence page; Citibank Singapore | ✅ verified |
| Citibank a MAS full-bank licensee and inaugural D-SIB (2014, among seven) | MAS FID; Straits Times; Yahoo Finance; centralbanking.com | ✅ verified |
| Core banking system / mainframe estate / payments hub / card platform / wealth platform / data-platform identities | No public disclosure found | ⚠ not public (see §12) |

### 11.2 The Honesty Note: Methodology

1. **Verified anchors are strong for Citi** — the history (§1.3) is primary-source verified via Citigroup's own heritage pages; the 2020 enforcement record (§8.1) is primary-source verified via OCC and Federal Reserve documents; the client platforms (§2–§3) are verified via Citi product pages and press releases. This is a *more* verifiable estate than DBS's (the [DBS guide](dbs_software_systems_guide.md) §11 makes the same comparison from the other side).
2. **The inference zone is the engine layer** — core identity, hubs, card platforms, wealth platforms, data/AI vendors: the *class* of system is certain for a bank of Citi's size, the *identity* is not (§12).
3. **One deliberate correction** — the Lava acquisition year: the common "2007" telling is ❌ disputed by the 2004 record; this guide follows the sources, not the lore.
4. **Deliberately not verified** (out of scope): per-market regulatory filings, vendor contracts and pricing, internal roadmaps, exact operating-market counts, Citi Singapore headcount, and the full co-brand portfolio list.

---

## 12. What Could Not Be Verified

The following are the materially non-public specifics of the Citi estate. Each is flagged ⚠ and should be treated as unknown, not as "likely X":

- **The core banking system identity** ⚠ — vendor vs in-house, the application names, and the ledger engines for deposits/loans/cards. Citi has never publicly named its core; the [Core Banking Systems Guide](core_banking_systems_guide.md) documents the vendor classes it could belong to, but no Citi-specific primary source exists.
- **The mainframe estate** ⚠ — hardware scale (IBM Z), the applications on it, and the decommissioning timeline beyond the public "retire and simplify" statistics (§6.2). The repo's mainframe content is limited to a file-sharing notification; no dedicated mainframe guide exists under technology/.
- **The payments hub / payments engine internals** ⚠ — the orchestration layer behind CitiDirect BE, WorldLink, and SWIFT connectivity; Citi's SWIFT gateway and ISO 20022 implementation specifics.
- **CitiDirect BE's internal architecture** ⚠ — beyond the verified Microsoft collaboration (2009) and the public next-gen reengineering, the current stack (legacy components, cloud migration state) is not public.
- **The card-management platform** ⚠ — the platform behind the AA/Costco books: authorization, clearing, servicing, collections, loyalty accounting.
- **The wealth platforms** ⚠ — private-bank and Citigold platform names and architectures.
- **The data platform and AI platform vendors** ⚠ — the data warehouse/lake estate and the GenAI platform behind the 30,000-developer coding tools (§7).
- **Citi Velocity / Lava engine internals** ⚠ — pricing, OMS/EMS internals, the connectivity catalogue, and LavaFlow's current role inside Citi's equities stack.
- **The "CitiFX Velocity" branding** ⚠ — not confirmed in this pass (see §3.2).
- **Exact footprint counts** ⚠ — operating-market counts by year; Citi Singapore's headcount and the unit-by-unit inventory of the hub.
- **AWS usage specifics** ⚠ — reported, not verified (§8.4).
- **Transformation roadmap details** ⚠ — system-by-system milestones and any named core-replacement decision; Citi has announced none publicly.
- **WorldLink client-segment mix and volumes** ⚠ — marketing describes the segments; Citi publishes no volume breakdown.
- **Correspondent-banking KYC specifics** ⚠ — Citi's internal due-diligence questionnaires and screening vendor contracts (the [FircoSoft Guide](fircosoft_guide.md) documents the industry-standard class, not Citi's contract).
- **USD clearing memberships** ⚠ — Citi's direct CHIPS/Fedwire memberships and the settlement accounts behind its USD clearing are not published; the Cymbal nostro sits on Citi's New York balance sheet (§10.2) but the clearing plumbing is unnamed.
- **Risk and limits systems** ⚠ — the counterparty credit limits, the credit engine behind LC confirmation and FI FX lines, and the real-time risk checks on payment flows.
- **Citi Singapore's consumer core** ⚠ — the retail core and card systems behind the Singapore consumer franchise (§9.2) are unnamed, as are the wealth platforms (§5).
- **The private-bank workstation** ⚠ — the advisory, portfolio-construction, and client-reporting tooling behind Citi Global Wealth.
- **Market data vendor contracts** ⚠ — the vendors feeding Citi Velocity's content/data layer are not named in Citi's public material (§3).
- **The Google Cloud agreement terms** ⚠ — value, duration, and workload scope beyond "strategic, multi-year" are not disclosed (§8.4).
- **The GenAI platform vendors** ⚠ — which LLM and developer-tooling vendors back the ~30,000-developer rollout (§7.2) is not public.
- **The legacy-app inventory** ⚠ — the ~390–400 applications retired in 2023 are counted, not named; no published inventory of the legacy franchise exists (§6.2).

---

## 13. Glossary

| Term | Definition |
|---|---|
| **City Bank of New York** | The 1812 predecessor of Citibank — chartered by the New York legislature on June 16, 1812; Samuel Osgood, first president |
| **National City Bank of New York** | The name taken in July 1865 when City Bank converted to a national charter under the National Bank Act |
| **First National City Bank (of New York)** | The 1955–1976 name (1955 merger with the First National Bank of the City of New York; shortened 1962) |
| **Citibank, N.A.** | The name since 1976; "N.A." = National Association — Citigroup's primary U.S. banking subsidiary |
| **Citigroup Inc.** | The holding company created by the 1998 Citicorp–Travelers merger (NYSE: C) |
| **IBC** | International Banking Corporation — the Connecticut-chartered bank (1901) that opened Citi's Singapore franchise in 1902 and pioneered its Asian trade finance |
| **TTS** | Treasury and Trade Solutions — Citi's cash management, payments, and trade business inside the Services division |
| **CitiDirect BE** | Citi's corporate/institutional e-banking platform (launched ~2009; "BE" generation), being reengineered as next-gen CitiDirect |
| **CitiDirect BE Mobile** | The mobile app for payment review/authorization/initiation/release, biometrics, and intraday positions; also the Mobile Token for QR/biometric desktop login |
| **WorldLink** | Citi's cross-border payment/disbursement service — funds transfers in 135+ currencies across 150+ countries, remote/on-site cheques, delivered via CitiDirect |
| **Citi Velocity** | Citi's content, data, analytics and trading platform for institutional and corporate clients (FX core) |
| **Velocity 3.0** | The 2023 single-dealer platform consolidating CitiFX Pulse, Citi Velocity Trading/Research, and CitiFX Click on HTML5 (web/desktop/mobile) |
| **SDP** | Single-Dealer Platform — a bank's own electronic trading venue for its clients (Velocity 3.0) |
| **CitiFX Pulse** | Citi's e-FX platform for corporate clients, folded into Velocity 3.0 |
| **Lava / LavaFlow** | Lava Trading Inc. (founded 1999; acquired by Citigroup 2004) and its EMS product LavaFlow, known for the MPID order-attribution concept |
| **EMS** | Execution Management System — order-routing/execution software for trading desks (LavaFlow's class) |
| **MPID** | Market-Participant ID — the four-letter identifier that identifies the firm behind an order (popularized by LavaFlow) |
| **Co-brand (affinity) card** | A credit card issued in partnership with a brand (airline/retailer): partner brings customers and marketing, issuer brings balance sheet and processing; economics shared |
| **AAdvantage** | American Airlines' loyalty programme; the basis of Citi's largest co-brand portfolio (Citi exclusive US issuer from 2026) |
| **Costco Anywhere Visa Card by Citi** | The card created when Citi bought the Costco co-brand portfolio from Amex in 2016 (>US$10.5B receivables) and converted it to Visa |
| **Citi Global Wealth / Citigold / Citi Private Bank** | The wealth division (post-2021) and its affluent/private segments; Citigold inaugurated in Hong Kong 1982 |
| **Nostro / Vostro** | "Our money at your bank" (nostro, from Cymbal's view of its Citibank N.Y. account) / "your money at our bank" (vostro, from Citi's view) |
| **Correspondent banking** | The relationship in which one bank (Cymbal) uses another bank's (Citi's) balance sheet and network to clear, settle, and pay in markets where it has no presence |
| **SWIFT** | The cooperative messaging network (MT/ISO 20022) that carries cross-border payment and trade messages between banks |
| **ISO 20022** | The global payments message standard (CBPR+ cross-border migration); pacs.008/pacs.009 for payments |
| **GPI** | SWIFT Global Payments Innovation — tracked, same-day-usable cross-border payments |
| **LC (documentary credit)** | Letter of credit — the bank-issued undertaking that anchors trade finance; MT700/MT707 message families |
| **Consent order / C&D** | Regulatory enforcement instruments: a consent order is agreed between regulator and bank; a cease and desist (C&D) order commands corrective action |
| **CMP** | Civil Money Penalty — the OCC's US$400M fine against Citibank N.A. (Oct 7, 2020) |
| **Living will / resolution plan** | The plan a large bank must file showing how it could be resolved in failure; Citi's was rejected Dec 2020 and flagged again Nov 2022 |
| **12 CFR Part 30, Appendix D** | The OCC Heightened Standards for large banks — the rule Citi was found to have violated in 2020 |
| **D-SIB** | Domestically Systemically Important Bank — MAS designation (2014) covering Citibank among seven in Singapore |
| **MAS** | Monetary Authority of Singapore — Citi Singapore's regulator (full-bank licence) |
| **FIX** | The Financial Information eXchange protocol for electronic trading (see [FIX Protocol Guide](fix_protocol_guide.md)) |
| **Mainframe** | The legacy large-computer class (IBM Z-era) on which Citi's core estate is structurally inferred to run (⚠) |
| **GenAI** | Generative AI — Citi armed ~30,000 developers with GenAI coding tools (2024) |
| **Cymbal Bank** | The fictional/illustrative mid-size Singapore bank persona used across this repository's worked examples — not a real bank |
| **MT103 / MT202** | SWIFT payment messages: MT103 customer credit transfer; MT202 bank-to-bank transfer (the correspondent cover leg) |
| **pacs.008 / pacs.009** | The ISO 20022 payment messages: customer credit transfer / bank-to-bank transfer under CBPR+ |
| **CBPR+** | The SWIFT-managed cross-border payments ISO 20022 migration programme |
| **GPI tracker (UETR)** | The Unique End-to-end Transaction Reference that lets SWIFT GPI payments be tracked end-to-end |
| **Maker/checker** | The four-eyes control: one operator initiates, a second authorizes (CitiDirect BE, IDEAL) |
| **OFAC** | The U.S. Office of Foreign Assets Control — the sanctions list U.S. banks screen against |
| **AML/CFT** | Anti-Money Laundering / Countering the Financing of Terrorism — the compliance programmes correspondent banks must document (§10.3) |
| **KYC / EDD** | Know Your Customer / Enhanced Due Diligence — the onboarding and monitoring discipline for FI clients |
| **Nostro reconciliation** | The daily matching of the nostro statement against the bank's own ledger (§10.7) |
| **White-label** | A platform licensed to another institution to run under its own brand (CitiDirect BE, per the 2009 Microsoft release) |
| **Host-to-host** | Direct system-to-system integration between a client's ERP/TMS and the bank channel (files, APIs, SWIFT) |
| **CHIPS / Fedwire** | The two U.S. wholesale dollar rails: CHIPS (netting) and Fedwire (RTGS) |
| **RTGS** | Real-Time Gross Settlement — the central-bank settlement rail class (Fedwire, MEPS+, etc.) |
| **Strangler-fig modernization** | The incremental replacement pattern in which new systems grow around and gradually replace the legacy core (§6.3) |
| **BCBS 239** | The Basel risk-data aggregation and reporting principles behind the data-governance findings (§7.1) |
| **Custody+** | Citi's near/real-time custody solutions announced 2026 — newly announced, ⚠ not deeply verified (§2.5) |
| **TBE / Legacy franchise** | Technology & Business Enablement — Citi's reported technology-and-ops organization; the internal name for the run-the-bank legacy estate ⚠ (§1.7) |

---

## 14. References and Further Reading

**Primary/company:**
- Citigroup Heritage pages — "A new bank born in New York" (1812); "A banker's bank: National charter and expansion of services" (1865); "IBC pioneers trade finance in Asia" (1902); "Business blooms in Latin America" (1914); heritage index (1998 merger, 2012 bicentennial, 2021 Fraser CEO) — citigroup.com/global/about-us/heritage
- OCC News Release 2020-132 (7 Oct 2020): "OCC Assesses $400 Million Civil Money Penalty Against Citibank"; OCC Cease and Desist Order ea2020-056 — occ.gov
- Federal Reserve press release (7 Oct 2020): "Federal Reserve announces enforcement action against Citigroup Inc."; Federal Reserve release bcreg20221123a (23 Nov 2022, resolution-plan shortcoming) — federalreserve.gov
- Citi press releases: "Citi to Acquire Costco U.S. Co-Brand Card Portfolio" (2016); "Citi and Google Cloud Announce Strategic Agreement to Modernize Citi's Technology Infrastructure" (2024) — citigroup.com
- BusinessWire (23 Feb 2023): "Citi Consolidates eFX Trading Platforms into Velocity 3.0, a New Single Dealer Platform"
- Microsoft newsroom (15 Sep 2009): "Microsoft Announces Major Initiative to Support CitiDirect BE"
- CitiDirect product pages (citidirect.com): CitiDirect Online Banking, WorldLink Payment Services (135+ currencies / 150+ countries), CitiDirect BE Mobile app and Mobile Token pages
- citivelocity.com; citigroup.com/global/businesses/markets/fx (Citi Velocity Trading)
- Citigroup global-presence page for Singapore; Citibank Singapore "120 Years of Citi in Singapore"; MAS Financial Institutions Directory
- Citi insights: "Transformation is on the Rise: The New CitiDirect Experience"; CitiDirect next-gen client FAQs
- American Airlines newsroom: AA–Citi co-brand extension (Dec 2024); AA–Citi Admirals Club expansion (2026)
- Citi Private Bank: Andy Sieg bio (Head of Wealth, joined Sept 2023)

**Press/analyst (used for verification):**
- WSJ: "Citigroup to Buy Lava Trading" (2004); Hedgeweek: "Citigroup to acquire Lava Trading"; TA Associates portfolio: Lava Trading Inc. (subsidiary of Citigroup from August 2004); tradersmagazine: "Lava Cools As Citi Exits Dark Pool Trading"
- The Banker: "Inside Citi's rebuild" (five business lines); finews.asia: "Citi Rejigs Global Organizational Structure"
- CIO Dive: Citi tech spend ($9.9B/2021, $11.2B/2022, $12B/2023; ~400 legacy apps retired); "Citi taps Google Cloud for app migration, AI adoption"; bankingdive: "Citi deploys AI coding tools to 30K developers" (Q4 2024); infotechlead: "$12B investment in digital technology" (2023); eFinancialCareers: Citi legacy-decommissioning progress
- FT: "US regulators rebuff Citigroup's 'living will' resolution plan" (Dec 2020); Reuters: "Bank regulators tell Citigroup to take urgent action to fix resolution plan" (23 Nov 2022); Yahoo Finance (Dec 2020 living will)
- Straits Times: "MAS names inaugural list of domestic systemically important banks" (2014); "Citi: Global bank, anchored in Singapore"; centralbanking.com; Yahoo Finance (MAS D-SIB list)
- National Library Board Singapore (NLB): "Citibank" (1812, 1865, 1976, 1902 Singapore); Wikipedia: "Citibank", "Citibank Singapore", Simple English Wikipedia (1976 rename)
- Business Traveller: "Citi Takes Over American Airlines Co-Branded Card Portfolio"; flightsmilesandpoints (2026 conversion date); marketscreener / electronicpaymentsinternational (Costco close, June 2016)
- Balch (law firm): "OCC Issues $400 Million Civil Penalty in Consent Order with Citibank" (Oct 2020); okoone: Citi tech-overhaul analysis

**Sibling guides in this repository (cross-references used throughout):**
- [DBS Software Systems Guide](dbs_software_systems_guide.md), [Standard Chartered Guide](standard_chartered_guide.md) — the structural models for this series
- [Core Banking Systems Guide](core_banking_systems_guide.md), [US Bank Core Systems Guide](us_bank_core_systems_guide.md), [Posting Engine Core Banking Guide](posting_engine_core_banking_guide.md), [Interest Calculation Engine Guide](interest_calculation_engine_guide.md) — core mechanics
- [Payment Rails Guide](payment_rails_guide.md) — SWIFT, ISO 20022, GPI, correspondent banking, nostro mechanics
- [Trade Finance Guide](trade_finance_guide.md), [Trade Finance Systems Guide](trade_finance_systems_guide.md), [Supply Chain Finance Guide](supply_chain_finance_guide.md) — trade and SCF
- [FIX Protocol Guide](fix_protocol_guide.md) — the FIX protocol mechanics
- [Private Banking Guide](private_banking_guide.md), [Wealth Management Guide](wealth_management_guide.md), [Universal Banking Model Guide](universal_banking_model_guide.md) — wealth and bank-model context
- [Enterprise Risk Management Guide](enterprise_risk_management_guide.md), [FircoSoft Guide](fircosoft_guide.md) — risk, AML/KYC, screening
- [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md) — the Singapore regulatory framework
- [IBM Cloud Guide](../technology/ibm_cloud_guide.md) — bank-cloud themes; [AI/LLM guides](../technology/ai_llm/) — AI platform mechanics

**Primary-source URLs (as consulted, plus pattern-derived deep links):**
- Citigroup Heritage index — https://www.citigroup.com/global/about-us/heritage (the 1812/1865/1902/1914/1998/2012/2021 pages are the entries linked from this index)
- OCC News Release 2020-132 — https://www.occ.gov/news-issuances/news-releases/2020/nr-occ-2020-132.html ⚠ (URL follows the OCC's standard release pattern; verify before citing)
- OCC Cease and Desist Order ea2020-056 — https://www.occ.gov/static/enforcement-actions/ea2020-056.pdf ⚠ (pattern-derived from the document ID cited in the file)
- Federal Reserve, 7 Oct 2020 enforcement release — https://www.federalreserve.gov/newsevents/pressreleases/enforcement20201007a.htm ⚠ (pattern-derived); Federal Reserve release bcreg20221123a — https://www.federalreserve.gov/newsevents/pressreleases/bcreg20221123a.htm ⚠ (pattern-derived)
- CitiDirect product pages — https://www.citidirect.com; Citi Velocity — https://www.citivelocity.com; Citi Markets/FX — https://www.citigroup.com/global/businesses/markets/fx
- BusinessWire, 23 Feb 2023 (Velocity 3.0) — https://www.businesswire.com (release dated 23 Feb 2023); Microsoft newsroom, 15 Sep 2009 (CitiDirect BE) — https://news.microsoft.com (release dated 15 Sep 2009)
- American Airlines newsroom (AA–Citi co-brand, Dec 2024; Admirals Club, 2026) — https://news.aa.com; Citi press releases — https://www.citigroup.com/global/news/press-releases
- MAS Financial Institutions Directory — https://eservices.mas.gov.sg/fid; National Library Board Singapore — https://www.nlb.gov.sg
- Note on deep links: OCC/Fed document URLs follow the agencies' standard patterns; the document IDs in this guide (§8.1–§8.2) are the stable identifiers to search if a deep link moves.

---

*End of guide. Citi is the oldest bank in America and, in its modern era, one of the most regulated — a 1812 charter run through 2020 consent orders, a US$400M penalty, and a rejected living will, answered with US$12B-a-year technology spend, thousands of retired legacy applications, and the reengineering of the very platforms clients see. The product names are public; the engines are not. The honest flags in §11–§12 mark exactly where public evidence ends and inference begins — and where the real system map of the bank lives: inside the global ledger.*
