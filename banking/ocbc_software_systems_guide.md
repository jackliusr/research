# OCBC: The Software Systems Landscape — A Comprehensive Guide to the Technology OCBC Runs

*A companion deep-dive to [DBS Software Systems Guide](dbs_software_systems_guide.md) (the pattern for this series) and [Standard Chartered Guide](standard_chartered_guide.md) (the structural model for a software-systems guide). This guide focuses on the **specific software and technology systems** behind OCBC — the Oversea-Chinese Banking Corporation: the core banking estate, digital banking, wealth systems, payment rails, AI and innovation, security and fraud, and the enterprise architecture — what is publicly known, what is vendor-verified, what is inferred from industry practice, and what OCBC simply does not disclose.*

**Verification convention used throughout: ✅ = verified in this research pass (primary/secondary sources); ⚠ = flagged (inferred, approximate, single-source, or structural inference); unmarked = structural/industry knowledge presented as such. The consolidated [Claims-Status table is in §11](#11-claims-status-and-verification-notes).**

---

## Table of Contents

1. [OCBC Overview](#1-ocbc-overview)
2. [Core Banking Systems](#2-core-banking-systems)
3. [Digital Banking Systems](#3-digital-banking-systems)
4. [Wealth Systems](#4-wealth-systems)
5. [Payment Systems](#5-payment-systems)
6. [AI and Innovation](#6-ai-and-innovation)
7. [Security and Fraud Systems](#7-security-and-fraud-systems)
8. [Enterprise Architecture](#8-enterprise-architecture)
9. [Worked Example: An OCBC Digital Customer Journey](#9-worked-example-an-ocbc-digital-customer-journey)
10. [Summary: The Oldest Local Bank's Modern Stack](#10-summary-the-oldest-local-banks-modern-stack)
11. [Claims Status and Verification Notes](#11-claims-status-and-verification-notes)
12. [Glossary](#12-glossary)
13. [References and Further Reading](#13-references-and-further-reading)

---

## 1. OCBC Overview

### 1.1 The Scope: What This Guide Covers

This guide is the **software-systems deep-dive for OCBC** — the mirror image of the software sections of the [Standard Chartered Guide](standard_chartered_guide.md) and the [DBS Software Systems Guide](dbs_software_systems_guide.md), applied to Singapore's oldest local bank. The *bank* itself — history, business segments, strategy — is covered here at the level needed to anchor the systems map; the deep strategy and financial analysis of the group belongs to the sibling [Universal Banking Model Guide](universal_banking_model_guide.md) (the group-structure reference) and the [DBS Bank Guide](dbs_bank_guide.md) (which contains the comparative Singapore-bank digital analysis). The division of labour:

| Topic | Where it lives |
|---|---|
| The bank, history, business segments, financials, leadership | This guide §1 (systems-anchoring summary) + [Universal Banking Model Guide](universal_banking_model_guide.md) |
| **The specific software systems: core, digital, wealth, payments, AI, security, architecture** | **This guide** (§2–§8) |
| The vendor/platform classes these systems belong to | [Core Banking Systems Guide](core_banking_systems_guide.md), [Payments Hub Guide](payments_hub_guide.md), [Wealth Management Guide](wealth_management_guide.md), [Financial Fraud Detection at Scale Guide](financial_fraud_detection_at_scale_guide.md) |
| The insurance arm's systems (Great Eastern) | [Insurance Software Systems Guide](insurance_software_systems_guide.md) |
| Singapore's real-time payment infrastructure (FAST/PayNow) | [Financial Infrastructure Guide](financial_infrastructure_guide.md), [ISO 20022 Core Processes Guide](iso_20022_core_processes_guide.md) |

What is covered here, section by section: the **OCBC overview** — history, the group, the divisions (§1); the **core banking estate** — the Silverlake-verified core and the per-franchise landscape (§2); **digital banking** — the OCBC Digital app and the online/mobile estate (§3); **wealth systems** — Bank of Singapore and the Premier/Treasures stack (§4); **payment systems** — FAST, PayNow, and the rails OCBC participates in (§5); **AI and innovation** — the AI Lab heritage, the 2023 genAI chatbot rollout, and the 2025–26 genAI wave (§6); **security and fraud** — the post-2021 anti-scam stack (§7); **enterprise architecture** — the data platform and cloud strategy (§8); a **worked customer journey** through OCBC Digital (§9); a **one-page summary** (§10); the honest **claims-status audit** (§11); a **glossary** (§12); and **references** (§13).

### 1.2 The History: "The Oldest Local Bank"

The single most important verified fact about OCBC: **OCBC is the longest-established Singapore bank, formed in 1932 from the merger of three local banks, the oldest of which was founded in 1912** ✅. This exact formulation appears in OCBC's own media materials (the July 2023 brand-refresh release: *"OCBC is the longest established Singapore bank, formed in 1932 from the merger of three local banks, the oldest of which was founded in 1912"*) and is repeated across directory and analyst sources.

The three constituent banks:

- **Chinese Commercial Bank** — founded **1912** ✅ — the oldest strand, hence the "founded 1912" date the group claims.
- **Ho Hong Bank** — founded 1917 ✅.
- **Oversea-Chinese Bank** — founded 1919 ✅.

The **Oversea-Chinese Banking Corporation** itself was incorporated in **1932** when the three merged ✅. The "1912 vs 1932" nuance matters for any architect writing about the group: **OCBC the legal entity is a 1932 company; its institutional lineage runs to 1912**, which is why OCBC bills itself as Singapore's oldest local bank (predating UOB, founded 1935, and DBS, founded 1968 — see [DBS Software Systems Guide](dbs_software_systems_guide.md) §1). The group's later expansion anchors are equally public: the **Bank of Singapore** (the private-banking subsidiary built from the 2009–10 acquisition of ING Asia Private Bank ⚠ — the rebrand and ownership are well documented; the exact deal-close month is flagged), **Great Eastern** (the insurance arm, majority-controlled since 2004 ⚠ stake figure), and **OCBC NISP** (the Indonesian bank, majority-controlled in the 2000s ⚠ stake figure) — plus the Malaysian franchise, the Hong Kong franchise, and OCBC China (formerly Wing Hang Bank, acquired 2014 ⚠) ⚠.

The business-context facts an architect needs before reading the systems map:

- **Group CEO Helen Wong** ✅ — took the helm in 2021 (the Straits Times/Forbes coverage: *"Ms Helen Wong… became OCBC's chief executive in 2021"*); under her leadership OCBC reported total income of US$10 billion for FY2023 ✅.
- **~30,000 employees group-wide** ✅ — OCBC's own 2021 release refers to *"all 30,000 employees across the OCBC Group, including employees in its overseas operations such as Malaysia, Indonesia, China and Hong Kong"*. This is the headcount behind every staff-facing system in this guide (the genAI rollout of §6, the training estate of §6.3).
- **Market position** ✅ (secondary source): *"OCBC Bank is the second largest financial services group in Southeast Asia by assets"* — Cloudera's own 2022 case-study line, which squares with the standard Singapore ranking (DBS first, OCBC second, UOB third; see [DBS Bank Guide](dbs_bank_guide.md)).

### 1.3 OCBC Today: The Group

OCBC is a **universal banking group**: a retail/consumer bank, a wholesale bank, a markets/treasury house, a private bank, an insurer, and an asset manager under one holding structure — the architecture that the [Universal Banking Model Guide](universal_banking_model_guide.md) generalises. The group map (each entity is a distinct legal entity, with its own systems estate — this matters in §2):

| Entity | Role in the group | Systems relevance |
|---|---|---|
| **OCBC Bank** (the flagship, Singapore) | Retail, consumer, corporate, markets banking — the "OCBC" brand | The core estate, OCBC Digital, Internet Banking, the payments stack — §2–§5 |
| **Bank of Singapore** | The private-banking subsidiary — *"Asia's global private bank"* ✅ | A separate wealth platform with its own onboarding stack — §4 |
| **Great Eastern** | The insurance arm (life + general), Singapore/Malaysia | Policy administration, claims, and actuarial systems — see [Insurance Software Systems Guide](insurance_software_systems_guide.md) |
| **OCBC NISP** | The Indonesian bank (PT Bank OCBC NISP) | Its own core/channel estate, Indonesian rails (BI-FAST) — §2.2 |
| **OCBC Malaysia / OCBC China / OCBC Hong Kong** | Regional franchises ⚠ | Franchise cores and regional channels — §2.2 |

The strategic posture to hold: OCBC is the **digital challenger among the Singapore incumbents** — the bank that has consistently positioned itself (a) as the *first* to ship specific digital/AI firsts (first genAI chatbot for all employees globally, 2023 ✅; the 2018 AI Lab ✅), and (b) as the *safest* digital bank in the post-scam era (the Kill Switch / Money Lock security narrative of §7). The systems landscape below is organised around exactly those two claims.

### 1.4 The Group Structure: Divisions

OCBC's financial reporting groups the businesses into four customer/business segments (verified against the Annual Report segment note ✅):

- **Global Consumer/Private Banking (GCPB)** ✅ — *"provides a full range of products and services to individual customers"* (AR 2024 wording): retail deposits, loans, cards, the OCBC Digital channel, Premier/Treasures wealth, and the private-banking books (Bank of Singapore sits inside the consumer/private-banking reporting line ⚠ — the AR bundles consumer and private banking in one segment).
- **Global Wholesale Banking (GWB)** ✅ — corporate and institutional: corporate lending, transaction banking, trade, cash management, the OCBC Business channel.
- **Global Markets (GM)** ✅ — treasury, FX, rates, structuring, and markets activities.
- **Insurance** ✅ — Great Eastern (life and general insurance).
- **Others** ✅ — the holding/central items.

(Historical reporting used different names — Global Consumer Financial Services, Global Corporate Banking, Global Treasury — ⚠ the segment names changed across reporting cycles; the AR 2023/2024 labels above are the current verified ones.)

The systems consequence: **each division runs its own application landscape on shared group infrastructure.** The retail core (§2), the corporate channels (§3), the markets/treasury stack (⚠ vendor not public — see §2.2), and the insurance estate ([Insurance Software Systems Guide](insurance_software_systems_guide.md)) are separate estates that share identity, data, and security layers. This is the classic universal-bank systems topology (compare [Standard Chartered Guide](standard_chartered_guide.md) §4–§8).

### 1.5 The Overview Table: Aspect and Description

| Aspect | Description |
|---|---|
| **Legal identity** | Oversea-Chinese Banking Corporation Limited, incorporated **1932** from the merger of Chinese Commercial Bank (1912), Ho Hong Bank (1917), Oversea-Chinese Bank (1919) ✅ |
| **Claim to fame** | Singapore's longest-established local bank ("the oldest local bank") ✅ |
| **Scale** | Second-largest financial services group in Southeast Asia by assets ✅ (secondary source); ~30,000 employees ✅; US$10B total income FY2023 ✅ |
| **Leadership** | Group CEO Helen Wong since 2021 ✅ |
| **Group entities** | OCBC Bank, Bank of Singapore, Great Eastern, OCBC NISP, OCBC Malaysia/China/Hong Kong ⚠ details flagged |
| **Divisions (AR 2023/24)** | Global Consumer/Private Banking; Global Wholesale Banking; Global Markets; Insurance; Others ✅ |
| **Core banking** | Silverlake Axis SIBS on IBM AS/400 (vendor-verified case study) — §2 |
| **Digital channels** | OCBC Digital app (mobile), Internet Banking, OCBC Business app — §3 |
| **Wealth** | Bank of Singapore private bank + Premier/Treasures; open-architecture platform — §4 |
| **Payments** | FAST (2014 rail), PayNow (2017 overlay), PayAnyone (retired 2023) — §5 |
| **AI** | AI Lab@TOV (2018), first-SG-bank genAI chatbot for all employees (2023), agentic-AI onboarding — §6 |
| **Security** | Kill Switch, Money Lock, OneToken, anti-scam unit — §7 |
| **Data/cloud** | Cloudera-based data platform, legacy-data-warehouse re-platform, cloud migration at scale — §8 |

### 1.6 The Technology Organization

The human system that runs the software estate is thinner in the public record than DBS's (contrast [DBS Software Systems Guide](dbs_software_systems_guide.md) §1.5), but the shape is clear:

- **~30,000 employees group-wide** ✅ (verified §1.2) — of whom the technology and operations population is a meaningful minority ⚠ (OCBC does not publish a DBS-style "10,000 technologists" number; the technology headcount is not public).
- **Group Operations & Technology** ⚠ — OCBC's technology function sits in a group operations-and-technology organisation (structural; the exact name/org chart is flagged — OCBC's public materials reference technology and operations leadership without the granularity DBS provides).
- **Engineering centres** ⚠ — OCBC runs technology delivery across Singapore, Malaysia, and Indonesia with additional regional delivery (the 2026 migration write-up's "Singapore, Malaysia, and Hong Kong" footprint is the closest verified-scale statement; the specific engineering-hub map is flagged).
- **The AI talent pipeline is the best-documented HR-system** ✅ — the AI Lab's training mandate (2018), the postgraduate scholarship (2019), and the 2023 genAI rollout to all employees (§6) describe a deliberate build-your-own-AI-bench strategy that the technology organization executes.
- **Tech spend** ⚠ — OCBC does not publish a clean technology-spend number in the material reviewed; the group's digital/AI investment is usually reported through programme announcements (the AI Lab, the cloud/data platform, WoW) rather than a capex line.

The architect's read: OCBC's technology organization is **smaller in public footprint and more operations-led than DBS's engineering culture**, but it is the same *pattern* — a central technology group running a shared estate for the divisions of §1.4, with the AI/security programmes acting as the public flagship of the org's capability.

---

## 2. Core Banking Systems

### 2.1 The Core: Vendor-Verified — Silverlake SIBS on IBM AS/400

The task brief's core question — *what is the OCBC core banking system, and who is the vendor?* — has a rare, clean answer: **Silverlake Axis SIBS (Silverlake Integrated Banking Solution) running on the IBM AS/400 (Power Systems) platform** ✅ — but with a scope caveat that must be stated before the claim is used.

The evidence is vendor-published and therefore carries both weight and bias:

- **Silverlake Axis's own case study, "OCBC: A Truly World-Class Core Banking System"** ✅ — Silverlake's solution page states: *"For OCBC's technological transformation and refresh, the bank decided on Silverlake Axis Integrated Banking Solution (SIBS) on an IBM AS400 (Power Systems) Platform."*
- The case study is **excerpted from an IDC case study, "The Tale of Two Super-Regional Banks and Their Core Banking Transformation" (September 2013), sponsored by Silverlake Axis** ✅ — the OCBC excerpt itself is dated March 2015. The IDC framing ("super-regional banks", "successful cross border implementation of best-in-class products and front end platforms") indicates the SIBS estate serves OCBC's **regional banking franchises** — the classic Southeast-Asia pattern where SIBS on AS/400 is the workhorse core for the Malaysia/Indonesia books ⚠ (the case study does not name the franchise explicitly; Silverlake's known regional client footprint and the "cross border implementation" phrasing make the regional interpretation the reasonable one — flagged).

What this means, honestly:

- **OCBC's core is a named vendor core** — the opposite of DBS's undocumented in-house estate (see [DBS Software Systems Guide](dbs_software_systems_guide.md) §2.2). SIBS is the flagship product of **Silverlake Axis** (the Malaysian core-banking vendor, publicly listed on SGX), and it is a **mainframe-class AS/400 estate**, not a cloud-native core.
- **Scope caveat ⚠**: the IDC/Silverlake case study does not enumerate *which* OCBC entities run SIBS. Industry consensus places SIBS at the OCBC regional franchises (Malaysia, Indonesia); **the identity of the Singapore flagship retail core is not directly verified in this pass** — the honest statement is *"OCBC runs Silverlake SIBS on IBM AS/400 (vendor-verified); the exact per-franchise coverage is not public"* (⚠). No Temenos, FLEXCUBE, or BaNCS production-core relationship for OCBC was found in this pass (⚠ unverified — contrast [Temenos Guide](temenos_guide.md) and [Oracle FLEXCUBE Data Model Guide](oracle_flexcube_data_model_guide.md) for those platforms' client profiles).
- **The AS/400 legacy implication**: an AS/400-based core is the poster child of the *hard-to-modernise* estate class — batch-oriented (end-of-day posting), RPG/COBOL application code, tight coupling to the hardware line. Every modernisation fact in §3–§8 (real-time balances, event streaming, cloud migration) has to be read against this constraint: OCBC's digital layer is built *in front of* a legacy core, exactly the DBS strangler-fig topology ([DBS Software Systems Guide](dbs_software_systems_guide.md) §2.3) — but with a vendor-named core rather than an anonymous one.

**The core estate at a glance:**

| Component | What it is | Status |
|---|---|---|
| **SIBS (Silverlake Integrated Banking Solution)** | The vendor core: deposits, loans, accounts, transactions | ✅ vendor-verified (Silverlake/IDC case study); ⚠ per-franchise scope |
| **IBM AS/400 (Power Systems) platform** | The core's hardware/runtime estate | ✅ vendor-verified |
| **Singapore retail core** | The flagship deposits/loans book | ⚠ identity not directly verified |
| **Regional franchise cores (MY/ID/HK)** | SIBS-class estates serving the overseas books | ⚠ inferred from the IDC "super-regional/cross-border" framing |
| **Bank of Singapore core** | Private-banking wealth platform, open architecture | ⚠ vendor not public (see §4) |
| **Temenos / FLEXCUBE / BaNCS** | No verified OCBC production relationship | ⚠ unverified / not found |

### 2.2 The Core Landscape: What Runs Where

For the architect, the useful model of the OCBC core estate is **one vendor family (Silverlake) plus several satellite estates** — not a single group-wide core:

- **The Silverlake/SIBS estate** ✅ (vendor-verified at group level; ⚠ per-franchise): the retail deposit/loan/account engines for the regional banking group. SIBS is the AS/400-era workhorse; its functional coverage (CASA, term deposits, loans, remittances, Islamic windows where applicable) is documented in [Core Banking Systems Guide](core_banking_systems_guide.md)'s vendor table. The accounting mechanics — interest accrual, end-of-day posting, balance enquiry — are the domain of the sibling mechanics guides: [Interest Engines Core Banking Guide](interest_engines_core_banking_guide.md) and [Posting Engine Core Banking Guide](posting_engine_core_banking_guide.md).
- **The Singapore flagship core** ⚠: not vendor-verified in this pass. The safe statement is that the Singapore retail book sits on a core of the same vintage/class (AS/400- or mainframe-generation), wrapped by the API/channel layer that OCBC Digital calls (§3). Whether Singapore is also SIBS, or a separate system, is not public ⚠.
- **Bank of Singapore** ⚠: a private bank needs a wealth-accounting platform (portfolio, custody, securities processing, discretionary mandates), not a vanilla retail core. The Asian Private Banker industry survey ("In Asia, private banks typically use one of two software systems — Avaloq or Temenos" ⚠) frames the market; **BOS's chosen platform is not public in this pass** — flag as unverified rather than guess (contrast the verified Avaloq-at-Maybank datapoint in the same coverage).
- **Great Eastern** ⚠: the insurance estate — policy administration, claims, actuarial, investment — is a wholly separate domain; see [Insurance Software Systems Guide](insurance_software_systems_guide.md) for the platform classes (the OCBC group's insurer is the anchor case there).
- **OCBC NISP** ⚠: the Indonesian franchise runs its own core; Silverlake's SIBS-family presence in the Indonesian market and the "super-regional" framing make a Silverlake-family core the reasonable inference, but **no direct NISP–Silverlake contract was verified in this pass** — flagged.
- **Markets/treasury core** ⚠: the Global Markets division runs the treasury/markets stack (FX, rates, derivatives processing, risk) — vendor not public in this pass; the platform classes are in [Nasdaq Calypso Guide](nasdaq_calypso_guide.md) and the DBS guide's §5 analysis.

The honest summary of §2.2: **OCBC's public core-banking footprint is "Silverlake SIBS on IBM AS/400", verified by the vendor and IDC; everything else about the core estate is inference about system classes, not verified facts about systems.**

### 2.3 The Core Table: System, Function, Notes

| System | Function | Notes |
|---|---|---|
| **Silverlake SIBS (SIBS)** | Retail core banking: deposits (CASA), loans, accounts, transactions for the regional banking group | ✅ Vendor-verified (Silverlake Axis + IDC 2013 case study, 2015 excerpt); runs on IBM AS/400 (Power Systems) |
| **IBM AS/400 (Power Systems)** | The core platform/runtime for SIBS | ✅ vendor-verified; legacy-class estate, batch-oriented |
| **Singapore retail core** | The flagship Singapore deposits/loans book of record | ⚠ Identity not directly verified; same vintage/class as SIBS expected |
| **Bank of Singapore wealth platform** | Private-banking accounting: portfolios, custody, securities, mandates | ⚠ Vendor not public; Avaloq/Temenos are the two common Asia private-bank platforms (industry survey) |
| **Great Eastern policy/claims estate** | Insurance policy administration, claims, actuarial | Cross-ref [Insurance Software Systems Guide](insurance_software_systems_guide.md); separate domain |
| **OCBC NISP core** | Indonesian retail core | ⚠ Silverlake-family inferred; not directly verified |
| **Markets/treasury stack** | FX/rates/derivatives processing and risk | ⚠ Vendor not public; platform classes in [Nasdaq Calypso Guide](nasdaq_calypso_guide.md) |
| **Core integration layer** | APIs, batch feeds, and real-time access paths between channels and cores | ⚠ Structural (every channel in §3 needs it); specifics not public |

### 2.4 Core Mechanics and Interfaces

For the architect, the OCBC core's *behaviour* matters more than its brand. The SIBS/AS/400 class of core (documented in [Core Banking Systems Guide](core_banking_systems_guide.md) and the mechanics guides) has a well-understood shape, and OCBC's public digital claims let us infer where OCBC sits on it:

- **The accounting core** ⚠ (mechanics are generic to the class, verified in the sibling guides): deposits accrue interest, loans amortise, and the ledger posts — classically at end-of-day batch on an AS/400 estate; the exact batch windows, accrual calendars, and posting rules at OCBC are not public ([Interest Engines Core Banking Guide](interest_engines_core_banking_guide.md), [Posting Engine Core Banking Guide](posting_engine_core_banking_guide.md), [Core Banking Processes Guide](core_banking_processes_guide.md)).
- **Real-time balance inquiry** ⚠ — OCBC Digital shows balances and transactions instantly (the product claim is verified — §3.1's personalisation pitch and the app's feature set); *how* instant is achieved — direct core enquiry, a real-time mirror/ledger cache, or event-fed balances — is not public. The industry pattern for an AS/400 core is a **real-time enquiry layer in front of a batch-posting core** (a ledger cache fed by event streams; see [Event Stream Processing Guide](../technology/event_stream_processing_guide.md) for the mechanism class).
- **The integration surface** ⚠ — channels (OCBC Digital, Internet Banking, OCBC Business) reach the core through an integration layer: APIs for real-time enquiry, batch files for end-of-day feeds, and increasingly event streaming for the data/AI estate (§8). This is the layer the 2026 migration write-up describes at "tens of thousands of producers and consumers" scale — the fabric, not the core, is what is being modernised.
- **The modernisation posture** ⚠ — consistent with the industry consensus (strangler-fig; see [Core Banking Systems Guide](core_banking_systems_guide.md)): the core is wrapped, new capabilities are built outside it, and core functions are progressively re-platformed. OCBC has not announced a named big-bang core replacement programme (unlike SC's Atlas-on-AWS in [Standard Chartered Guide](standard_chartered_guide.md) §4.1); its public record describes the *refresh* of the SIBS estate (the Silverlake case study's "technological transformation and refresh") rather than a vendor-swap ⚠.

### 2.5 The Core in Series Context: Silverlake SIBS vs the Vendor Landscape

For the reader coming from the sibling core guides, OCBC is the *Silverlake* case in the vendor landscape — the cleanest public instance of the Southeast-Asian SIBS/AS-400 pattern:

| Vendor core | Where it appears in this series | OCBC relevance |
|---|---|---|
| **Silverlake SIBS** | **This guide (OCBC)** | ✅ OCBC's vendor-verified core; AS/400-class, batch-oriented, regional-scope |
| **Temenos (T24/Transact)** | [Temenos Guide](temenos_guide.md), [T24 Programming Guide](t24_programming_guide.md), [TAFJ Guide](tafj_guide.md) | ⚠ No verified OCBC relationship; the Temenos cores are cross-referenced for the *class* mechanics only (T24/TAFJ/JBASE — see [JBASE Universe Guide](../technology/jbase_universe_guide.md)) |
| **Oracle FLEXCUBE** | [Oracle FLEXCUBE Data Model Guide](oracle_flexcube_data_model_guide.md) | ⚠ No verified OCBC relationship; data-model mechanics cross-referenced lightly |
| **Apache Fineract** | [Apache Fineract Guide](apache_fineract_guide.md) | ⚠ No OCBC relevance (the open-source digital-bank core class); contrast case |
| **Chinese bank cores** | [Chinese Bank Core Systems Guide](chinese_bank_core_systems_guide.md) | ⚠ Light cross-ref (OCBC China/Wing Hang franchise context) |
| **In-house wrapped legacy** | [DBS Software Systems Guide](dbs_software_systems_guide.md) §2 | The *contrast* case: DBS hides its core identity; OCBC names its vendor |

The series-level lesson: **OCBC is the proof that "vendor core" and "modern bank" are not opposites** — the bank runs a named legacy vendor core *and* the region's first bank-wide genAI rollout (§6); the modernisation is in the edge layers, not the core. That is the pattern every sibling guide's architecture chapter (DBS §9, StanChart §4) describes, and OCBC is its most vendor-transparent instance.

---
## 3. Digital Banking Systems

### 3.1 The OCBC Digital App: The Verified Flagship

The consumer digital front door is the **OCBC Digital app** — verified in OCBC's own words: *"the OCBC Digital app, which is the bank's mobile app for digital banking services"* (OCBC statement quoted by the Straits Times, September 2023, when the bank retired the separate Pay Anyone app and consolidated its functions into OCBC Digital) ✅. Supporting evidence across this pass:

- **The app's store presence** ✅ — the Google Play listing (`com.ocbc.mobile`, "OCBC Digital - Mobile Banking") and the iOS App Store listing ("OCBC Digital - Mobile Banking") carry the *"all-new OCBC app… customisable shortcuts and a personalised experience"* marketing line — the app is OCBC's personal-banking mobile front door.
- **The 2023 rebrand wave** ✅ — OCBC unified its brand in July 2023 (the group's own media release, "OCBC unifies brand", covers the logo refresh and the group positioning); the app's naming aligned with that wave ⚠ (the exact app-rename date is not separately verified, but the "OCBC Digital" name is attested from August 2023 coverage onward — Mothership, August 2023, describes the *"OCBC Digital app"* security feature).
- **App functionality (verified product features)** ✅ — one-tap logins with OCBC OneToken (the token-based two-factor authentication that also names the bank's anti-scam controls, §7); in-app calling added progressively from November 2025 (retail customers calling the bank directly inside the OCBC and OCBC Business apps — Wikipedia's OCBC Bank article, updated 2025–26); proactive security checks that block login when a risky third-party app is detected on the device (Mothership, August 2023).
- **Pay Anyone consolidation** ✅ — the standalone Pay Anyone P2P app was discontinued in September 2023; functions consolidated into OCBC Digital (Straits Times, September 2023).

The product framing OCBC itself uses: *"the all-new OCBC app that is designed to understand your banking needs, wants and quirks"* ✅ — a personalisation pitch that, architecturally, means the app sits on the data/AI estate of §6–§8 (recommendations, fraud scoring, insights) as much as on the core of §2.

### 3.2 The Digital Systems: Online and Mobile Estate

The OCBC digital estate is a **two-surface (mobile + web), multi-audience** architecture — the standard incumbent-bank channel topology (compare [DBS Software Systems Guide](dbs_software_systems_guide.md) §8):

- **OCBC Digital (mobile app)** ✅ — retail/personal mobile banking (§3.1): balances, transfers, PayNow, cards, investments, wealth.
- **OCBC Internet Banking** ✅ (product attested in OCBC's own security messaging — *"you will not be able to log in to our Internet Banking and/or the OCBC Digital app"* — and in the app's "what has changed" support pages): the web channel for personal banking; the mature online-banking surface that predates the app and continues alongside it.
- **OCBC Business (app + online)** ✅ — the SME/corporate channel; the November 2025 in-app-calling coverage names "the OCBC and OCBC Business apps" side by side; the OCBC business-banking pages market the OCBC Business app's sales/expense-trend dashboards ✅ (the e-commerce customer story on the business-banking site is OCBC's own example of app-embedded analytics).
- **Retired/consolidated apps** ✅ — **Pay Anyone** (P2P payments; discontinued September 2023, folded into OCBC Digital) is the documented example; the pattern of consolidating point apps into the flagship is OCBC's stated direction ⚠.
- **OneToken** ✅ — the bank's token-based authentication layer (soft token on the device, push-style approval) that sits across mobile and internet banking; also brand-marketed as an anti-scam control (§7).

What is *not* public ⚠: the channel middleware and API layer between the apps and the cores — OCBC does not publish a DBS-style "API platform" narrative (contrast [DBS Software Systems Guide](dbs_software_systems_guide.md) §8.4), and the mobile app's own engineering stack (native vs cross-platform, backend services) is undisclosed. The architecture is inferred: **channel front ends → a channel-integration layer → the core services of §2** (structural ⚠).

### 3.3 The Digital Table

| System | Function | Notes |
|---|---|---|
| **OCBC Digital (mobile app)** | Personal mobile banking: balances, transfers, PayNow, cards, wealth, insights | ✅ the bank's mobile app for digital banking (OCBC/ST, 2023); OneToken login; in-app calling from Nov 2025; risky-app login blocks |
| **OCBC Internet Banking** | Personal online banking (web) | ✅ attested in OCBC security messaging and support pages; the pre-app online surface |
| **OCBC Business (app + online)** | SME/corporate digital banking: payments, trade, cash views, embedded analytics | ✅ product verified; the corporate counterpart of OCBC Digital |
| **Pay Anyone (retired)** | Standalone P2P payment app | ✅ discontinued Sep 2023; consolidated into OCBC Digital |
| **OCBC OneToken** | Token-based two-factor authentication across channels | ✅ product verified; doubles as an anti-scam control (§7) |
| **Channel-integration layer** | Middleware/APIs between channels and cores | ⚠ Structural inference; specifics not public |
| **Digital onboarding/eKYC** | New-customer journeys (Singpass-class identity rails) | ⚠ Singpass integration is standard for SG banks; not separately verified this pass |

### 3.4 The Regional Digital Franchises

The OCBC Digital story is the Singapore story; the group's overseas franchises run their own channel estates on the group's core family (§2.2):

- **OCBC NISP (Indonesia)** ⚠ — the Indonesian franchise runs its own mobile banking and internet banking (the OCBC Indonesia presence is verified at brand level — e.g., the NISP TikTok channel markets its 12-currency multi-currency account and real-time rates ⚠ promotional source); the app's architecture and the franchise's participation in Indonesia's BI-FAST instant-payment rail are structurally certain but not separately verified this pass.
- **OCBC Malaysia / OCBC China / OCBC Hong Kong** ⚠ — regional digital channels on the regional cores; no per-franchise system names were verified in this pass.
- **The consolidation direction** ✅ — OCBC's Singapore pattern (retire point apps, consolidate into the flagship) is the group's stated direction ⚠ — the Pay Anyone consolidation (2023) is the documented example; whether the same consolidation is underway franchise-by-franchise is not public.

### 3.5 Digital Banking in Market Context

Where OCBC Digital sits among the Singapore channel landscape (the comparison set from [DBS Software Systems Guide](dbs_software_systems_guide.md) §8 and [Trust Bank Guide](trust_bank_guide.md)):

| Channel | Owner | Core underneath | Notes |
|---|---|---|---|
| **OCBC Digital** | OCBC | Silverlake SIBS-class (⚠ SG identity) | The consolidated flagship; OneToken; anti-scam controls baked in |
| **digibank** | DBS | Wrapped legacy core (⚠) | The journey-led incumbent ([DBS Software Systems Guide](dbs_software_systems_guide.md) §8) |
| **UOB TMRW / UOB app** | UOB | Legacy wrapped (⚠) | The third incumbent; TMRW as the digital-first brand |
| **Trust** | Trust Bank | Mambu on GCP (✅ per [Trust Bank Guide](trust_bank_guide.md)) | The licensed digital bank — cloud-native core, the contrast case |
| **GXS / MariBank** | Grab/Singtel; Sea | Vendor/cloud-native cores (⚠) | The other licensed digital banks |

OCBC's positioning in this landscape is the **security-first incumbent**: the same wrapped-legacy architecture as DBS and UOB, differentiated by the documented anti-scam controls (§7) and the AI firsts (§6) rather than by a greenfield core. No digital-bank licence, no Mambu-class core in the OCBC estate — the same structural conclusion the DBS guide draws for DBS ⚠.

---

## 4. Wealth Systems

### 4.1 Wealth Management: Bank of Singapore and the Premier/Treasures Stack

OCBC's wealth estate spans three layers — the **private bank** (Bank of Singapore), the **affluent mass-affluent tiers** (OCBC Premier, OCBC Treasures ⚠ brand tiers, widely documented), and the **mass-market wealth features inside OCBC Digital** (investments, insurance, robo-advisory-style tools ⚠ product names not verified this pass). The systems story, layer by layer:

- **Bank of Singapore (BOS)** ✅ — *"OCBC's private banking subsidiary"*, self-described as *"Asia's global private bank"*; a high-net-worth private bank serving clients across Asia and the Middle East. Its platform is **open-architecture** ✅ (Wikipedia: *"Through its open-architecture platform, Bank of Singapore offers clients access to a range of investment products and wealth planning solutions"*) — meaning the product shelf is assembled from third-party funds/managers rather than only OCBC-house products, which architecturally implies a **wealth-platform core with an open product catalogue** rather than a closed bank-owned shelf.
- **The onboarding modernisation (agentic AI)** ✅ — The Asian Banker (2025–26): OCBC's private-banking onboarding platform, using agentic AI, completes **account opening in 15 business days versus an industry median of about six weeks**. This is the verified anchor for both the wealth stack and the AI estate (§6): a document-heavy, regulatory-heavy private-bank onboarding journey re-platformed around AI agents.
- **The advisory digitisation** ✅ — the Straits Times/Business Times coverage of OCBC moving its wealth-advisory process online: a *"complex face-to-face process involving more than 50 pages of documents and a comprehensive financial needs analysis (FNA)"* turned into a digital journey — the paper-era advisory stack (FNA forms, product suitability documents) re-platformed as an online workflow ⚠ date of that specific article flagged (June, year not verified in snippet).
- **GenAI for advisers** ✅ — OCBC's consumer-banking unit rolled out a **Gen AI-powered skills-training programme for its wealth-adviser force in Singapore** (per OCBC's own 2026 release on avatar banking, which describes it as an "earlier this year" initiative), and The Business Times reported OCBC training wealth advisers using generative AI with *"anonymised proprietary insights on customer behaviours to generate realistic training scenarios"* — i.e., AI-generated client-role-play training ⚠ the two pieces may describe the same programme (both 2025–26 vintage).
- **BOS's core platform** ⚠ — not verified: the private-banking core (portfolio accounting, custody, securities processing, mandates) is a distinct system from the retail core (§2.2); Asian Private Banker's survey frames Avaloq vs Temenos as the two dominant Asia private-bank platforms, but **no BOS–vendor contract was verified in this pass** — flagged, not guessed.

The wealth systems consequence: **wealth is the estate where OCBC's AI story is most concrete** — agentic onboarding, genAI adviser training, AI-generated role-plays — and where the *platform* story is least public. The domain mechanics (portfolio accounting, suitability, discretionary mandates) are covered in the sibling [Wealth Management Guide](wealth_management_guide.md).

### 4.2 The Wealth Table

| System | Function | Notes |
|---|---|---|
| **Bank of Singapore platform** | Private banking: portfolios, custody, securities, discretionary mandates, onboarding | ✅ BOS as the private-banking subsidiary verified; open-architecture shelf; ⚠ core vendor not public |
| **Agentic-AI onboarding platform** | Private-bank account opening: document processing, KYC/AML workflow, AI-assisted review | ✅ 15 business days vs ~6-week industry median (The Asian Banker) |
| **Digital advisory workflow** | Wealth advisory/FNA online: the 50+-page paper process re-platformed | ✅ digitisation verified; ⚠ article date flagged |
| **GenAI adviser training** | AI-generated realistic client scenarios for wealth-adviser upskilling | ✅ (OCBC release 2026; BT coverage); anonymised customer-behaviour insights |
| **OCBC Premier / Treasures** | Affluent/mass-affluent tiers with dedicated RMs and wealth products | ⚠ brand tiers documented; systems not separately public |
| **Wealth features in OCBC Digital** | Mass-market investments/insurance/wealth tools in the retail app | ⚠ product names not verified this pass |
| **Wealth domain mechanics** | Suitability, product governance, portfolio rebalancing | Cross-ref [Wealth Management Guide](wealth_management_guide.md) |

### 4.3 The Wealth Stack in Context: Scale and Architecture

Two verified numbers anchor the wealth estate's scale, and one architectural fact frames its design:

- **BOS's AUM ambition** ✅ — OCBC's July 2023 brand-refresh release states Bank of Singapore *"likewise aims to increase its AUM — to USD145 billion by end 2025"* — the private bank is a US$100B+-scale wealth franchise, which puts its platform requirements (multi-currency portfolio accounting, custody across markets, discretionary mandates, reporting at scale) firmly in the tier-1 Asian private-bank class.
- **The onboarding transformation is the systems headline** ✅ — the agentic-AI onboarding platform (15 business days vs ~6-week industry median, The Asian Banker) is the concrete evidence that the wealth estate is being modernised at its most painful point: the document/KYC/AML-heavy account-opening journey. The domain's document-processing mechanics are covered in [Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md); OCBC's application of *agentic* (multi-step, tool-using) AI to it is the OCBC-specific part.
- **Open architecture means integration-heavy** ⚠ — BOS's open-architecture shelf (third-party funds, external managers) forces a platform design that is less about proprietary products and more about **product onboarding, order routing, and reconciliation** across external counterparties — the integration burden of a wealth platform, not a retail core (cross-ref [Wealth Management Guide](wealth_management_guide.md) for the platform classes: portfolio accounting, order management, CRM, suitability engines).
- **The private-bank core question stays open** ⚠ — as in §2.2: no verified vendor for BOS's core in this pass. The Asian Private Banker survey (Avaloq vs Temenos as the dominant Asian private-bank platforms) frames the market; BOS's choice is not public.

The architect's read: **the wealth estate is where OCBC's modernisation and its secrecy meet** — the AI-era results (agentic onboarding, genAI training) are public and impressive, while the platform layer underneath (core, order routing, custody interfaces) is entirely unstated.

---

## 5. Payment Systems

### 5.1 The Payment Rails: FAST, PayNow, and the Ecosystem

OCBC's payment systems are best understood as **participant rails plus bank-owned overlays** — OCBC does not run the national rails, it plugs into them (the pattern generalised in [Financial Infrastructure Guide](financial_infrastructure_guide.md)):

- **FAST (Fast And Secure Transfers)** ✅ — Singapore's real-time interbank clearing rail, live since **2014** (ABS/industry-standard facts; RTP Dashboard: *"FAST (Fast And Secure Transfers, 2014) provides the real-time clearing rail"*). FAST is the settlement backbone for all domestic instant payments; every OCBC Digital transfer and PayNow payment settles over it.
- **PayNow** ✅ — the **2017** proxy-based overlay that rides on FAST: transfers by mobile number, NRIC/FIN, UEN, or Virtual Payment Address without revealing account numbers (ABS: *"inter-bank peer-to-peer and person-to-merchant transactions"*; OCBC's own PayNow help pages describe exactly this). **OCBC is a participating bank** ✅ (its help-and-support PayNow pages are live and product-level); the *founding* participant claim (PayNow launched July 2017 with the then-major banks) is industry-standard knowledge ⚠ flagged as reported rather than separately re-verified this pass.
- **Cross-border links** ✅ — PayNow's bilateral linkages: **India's UPI, Thailand's PromptPay, Malaysia's DuitNow** (RTP Dashboard) — meaning OCBC's PayNow surface can reach those schemes' participating wallets/banks, with the settlement/forex mechanics handled by the scheme links ⚠ OCBC's specific participation in each link is not separately verified.
- **Bank-owned payment surfaces** ✅ — **PayAnyone** (OCBC's P2P app, retired September 2023 with functions folded into OCBC Digital — the clearest verified payment-product fact); OCBC Digital's transfer/PayNow journeys; the OCBC Business channel for corporate payments ⚠ corporate payment rails (GIRO, FAST bulk, trade payments) not separately verified this pass.
- **Cards** ⚠ — the card-acquiring/issuing estate (Visa/Mastercard/UnionPay/NETS acceptance, card management, 3-D Secure) is standard-issue for a bank of OCBC's size but no vendor/scheme contract was verified this pass — flagged as structural.
- **ISO 20022** ⚠ — Singapore's rails and cross-border messaging moved to ISO 20022 (see [ISO 20022 Core Processes Guide](iso_20022_core_processes_guide.md)); OCBC's participation is structural (the rails are ISO 20022-native) rather than separately verified.

### 5.2 The Payments Table

| System | Function | Notes |
|---|---|---|
| **FAST** | Singapore's real-time interbank clearing rail (2014) | ✅ rail verified (ABS/RTP Dashboard); all OCBC domestic instant payments settle over it |
| **PayNow** | Proxy-based instant payments overlay on FAST (2017): mobile/NRIC/UEN/VPA | ✅ rail verified; OCBC participating bank with live product pages; founding-participant ⚠ reported |
| **PayNow cross-border links** | UPI (India), PromptPay (Thailand), DuitNow (Malaysia) linkages | ✅ links verified; ⚠ OCBC-specific participation not re-verified |
| **PayAnyone (retired)** | OCBC's P2P payment app | ✅ discontinued Sep 2023; consolidated into OCBC Digital |
| **OCBC Digital / Business payment journeys** | Transfers, PayNow, bill payments, corporate payments | ✅ product surfaces verified |
| **Card estate** | Issuing/acquiring, card management, 3-D Secure | ⚠ structural; no vendor/scheme contract verified this pass |
| **Payments hub / integration** | Internal payment orchestration between channels, core, and rails | ⚠ inferred (the [Payments Hub Guide](payments_hub_guide.md) class); specifics not public |

### 5.3 The Regional Payment Rails

The group's franchises plug into their home markets' rails, mirroring the Singapore pattern:

- **Indonesia — BI-FAST** ⚠ — Bank Indonesia's real-time retail payment rail (live 2021); OCBC NISP, as a major Indonesian bank, participates structurally ⚠ (participation not separately verified this pass; BI-FAST coverage is effectively universal among Indonesian banks of NISP's tier).
- **Malaysia — DuitNow** ⚠ — the Malaysian instant-payment/QR rail (PayNet); OCBC Malaysia's participation is structurally expected ⚠ (and DuitNow is one end of the PayNow–DuitNow cross-border link of §5.1, which OCBC's Singapore side can reach).
- **Cross-border instant payments** ✅ — the PayNow–UPI, PayNow–PromptPay, and PayNow–DuitNow links mean OCBC's PayNow surface extends into India, Thailand, and Malaysia for participating customers (rails verified; OCBC's per-link onboarding is ⚠).
- **SWIFT and the corporate rails** ⚠ — cross-border wires (SWIFT), GIRO (local bulk debits/credits), and trade payments are structural for a wholesale bank of OCBC's size (see [ISO 20022 Core Processes Guide](iso_20022_core_processes_guide.md) and [Financial Infrastructure Guide](financial_infrastructure_guide.md)); OCBC-specific implementations were not verified this pass.

### 5.4 The Payments Architecture Pattern

Reading the verified pieces back against the platform class (the pattern generalised in [Payments Hub Guide](payments_hub_guide.md)):

1. **The rails are external** — FAST/PayNow (and BI-FAST/DuitNow regionally) are industry utilities; OCBC's payment systems are the *participant edge*: channel journeys, proxy registration (mobile number/NRIC/UEN ↔ account), and settlement integration.
2. **A payments hub must exist between channel and rail** ⚠ — the ISO 20022 message orchestration, the FAST/PayNow interfaces, the fraud-screening hook (§7), and the core postings do not wire together directly in any bank of OCBC's size; the hub is structurally certain, vendor/implementation not public.
3. **The scam controls are payment-system features** — Money Lock (restricting which funds are movable via the rail), Kill Switch (freezing the participant edge), and the fraud-AI screen are all *payment-pipeline* controls, which is why they appear in §5's journey (§9) as much as §7's security narrative.
4. **ISO 20022 is the language** ✅-ish ⚠ — the SG rails and cross-border messaging are ISO 20022-based (verified for the rails; OCBC's end-to-end ISO readiness is structural ⚠).

### 5.5 Cards and Merchant Payments

The card estate — the payment surface beyond FAST/PayNow — is structurally certain and vendor-silent:

- **Issuing** ⚠ — OCBC issues Visa/Mastercard credit and debit cards (the card products are public marketing; the card-management platform, personalisation/limits engine, and 3-D Secure stack are not verified this pass). The card-authorisation path (issuer → scheme → acquirer) is the standard four-party model, documented in [Financial Infrastructure Guide](financial_infrastructure_guide.md).
- **Acquiring** ⚠ — OCBC's merchant-acquiring business (NETS and scheme acceptance in Singapore; the QR-code ecosystem with SGQR) is standard-issue ⚠.
- **Card services in the app** ✅ — the OCBC Digital app's card-management surface is verified (the app's own support pages walk through fee waivers, card services, and OneToken-backed approvals — "Tap 'More' > Card services > Request fee waiver"); the *backend* card systems are ⚠.
- **The card-scam nexus** ⚠ — card fraud control (real-time card-screening, tokenisation for e-commerce) is part of the fraud estate of §7; OCBC-specific implementations not public.

---

## 6. AI and Innovation

### 6.1 The OCBC AI Heritage: The AI Lab Era

OCBC's AI story is unusually well-documented for an incumbent — and it is the one area where OCBC's public record outpaces both DBS's and StanChart's (per the sibling guides). The verified timeline:

- **AI Lab@TOV, 2018** ✅ — OCBC's own release: the bank launched an **AI Lab** (at The Open Vault @ OCBC — the bank's FinTech innovation space) as *"a specialised training centre to further develop the data science skills of data analysts and technologists within the bank"* — i.e., both a delivery unit and an internal training estate.
- **Postgraduate AI Scholarship, 2019** ✅ — OCBC launched a postgraduate scholarship to deepen its AI talent pool (with the Singapore universities); the careers-blog story "From Learning AI to doing AI" follows a scholarship graduate into the AI Lab ✅ — note the "Learning AI" phrasing, the exact name the task brief associates with OCBC's in-house AI-learning tooling ⚠ (see §6.2 for what is and is not verified about that name).
- **The AI Lab as a talent factory** ✅ — the 2018 release explicitly frames the Lab as the mechanism for converting analysts/technologists into AI practitioners at scale.

### 6.2 The 2023 GenAI Wave: The Employee Chatbot Rollout

The flagship AI fact of this pass: **OCBC was the first Singapore bank to roll out a generative-AI chatbot to all employees globally — 2023** ✅. The OCBC newsroom release is titled exactly *"OCBC First To Launch AI Chatbot For Global Employees"* (2023), describing the rollout of a generative-AI chatbot to staff worldwide *"driving productivity gains with generative AI tools"*.

The task brief's specific claims, checked honestly:

- **The "~30,000 staff" scale** — the group's ~30,000-employee headcount is verified ✅ (OCBC's 2021 release, §1.2). The chatbot rollout *to all employees globally* therefore covers a workforce of that order ⚠ — the exact "30,000 staff" figure attached to the chatbot rollout itself is flagged (the headcount is verified; its coupling to the AI announcement is the reporter's arithmetic, not an OCBC quote in the snippets seen).
- **The "Learning AI" name** — the phrase "Learning AI" is verified as OCBC's own **careers-blog framing** ("From Learning AI to doing AI") and the AI Lab's training-centre mandate ✅; whether "Learning AI" is the *product name* of an in-house learning tool is **not verified** ⚠. The verified in-house tool is the **genAI chatbot for employees (2023)** — OCBC's public record describes an AI-learning *programme* (Lab + scholarship + training) and a genAI *assistant*, not a separately named learning product. This guide therefore uses "Learning AI" as the programme framing with the flag, and treats the 2023 chatbot as the verified tool.
- **The "first Singapore bank" claim** ✅ — it is OCBC's own headline claim, and no earlier SG-bank employee-wide genAI rollout was surfaced in this pass.

### 6.3 The GenAI Innovation Wave (2024–2026)

The 2023 chatbot was the beachhead; the verified follow-ons are the substance of OCBC's genAI strategy:

- **Agentic AI in private banking** ✅ — The Asian Banker: OCBC's agentic-AI onboarding platform cuts BOS account opening to **15 business days vs ~6 weeks industry median** (§4.1) — from genAI assistant to agentic workflow.
- **GenAI for the wealth-adviser force** ✅ — the Gen AI-powered skills-training programme for Singapore wealth advisers (OCBC's own 2026 release; The Business Times: realistic training scenarios generated from anonymised customer-behaviour insights).
- **Avatar banking: OCBC WoW** ✅ — OCBC unveiled **OCBC WoW** avatar banking in 2026, explicitly building *"on earlier investments in AI, digital and data"* (OCBC newsroom) — the AI estate's consumer-facing frontier ⚠ product details beyond the unveiling not yet public.
- **AI in fraud/security** — see §7 (the anti-scam stack is the AI estate's most visible consumer surface) ⚠ specifics flagged there.

### 6.4 The AI Table: Initiative, Description, Notes

| Initiative | Description | Notes |
|---|---|---|
| **AI Lab@TOV (2018)** | In-house AI delivery + training centre for data analysts/technologists | ✅ OCBC release 2018; the origin of the "learning AI" mandate |
| **Postgraduate AI Scholarship (2019)** | Talent pipeline: MSc-AI graduates into the AI Lab | ✅ OCBC release 2019; "From Learning AI to doing AI" careers story |
| **GenAI chatbot for all employees (2023)** | First SG bank to roll out a generative-AI chatbot to staff globally; productivity use cases | ✅ OCBC release 2023; ~30,000-employee workforce ⚠ figure coupling flagged |
| **"Learning AI" programme framing** | OCBC's own phrase for the AI upskilling journey (Lab → scholarship → practice) | ✅ phrase verified in OCBC careers blog; ⚠ product-name status not verified |
| **Agentic-AI private-bank onboarding** | AI agents run the BOS account-opening workflow | ✅ 15 business days vs ~6-week median (The Asian Banker) |
| **GenAI wealth-adviser training** | AI-generated client role-play scenarios from anonymised insights | ✅ OCBC 2026 release + Business Times |
| **OCBC WoW avatar banking (2026)** | Avatar-based banking experience; builds on the AI/data estate | ✅ OCBC release 2026; ⚠ details early |
| **AI in fraud detection** | Machine learning across transaction monitoring, scam detection, anti-mule controls | ⚠ product names not public; see §7 |

### 6.5 The Innovation Ecosystem: The Open Vault and the AI Culture

OCBC's AI programme did not appear from nowhere in 2023; it sits on an innovation ecosystem with a documented spine:

- **The Open Vault @ OCBC (TOV)** ✅ — the bank's FinTech innovation space, home of the AI Lab (the 2018 release's "AI Lab@TOV" naming is the verified anchor): a co-working/partnership venue where the bank works with fintech startups — the standard incumbent-bank open-innovation pattern (compare DBS Asia X in [DBS Software Systems Guide](dbs_software_systems_guide.md) §9.4) ⚠ TOV's full partnership catalogue not verified this pass.
- **The AI Lab as delivery + academy** ✅ — the 2018 release's dual mandate (build AI capabilities *and* train the bank's analysts/technologists) is the design that made the 2023 genAI rollout possible: by 2023 the bank had a standing AI organisation and an AI-literate workforce to absorb a group-wide chatbot.
- **The 2023 rollout as culture change** ✅ — "first Singapore bank to roll out a generative-AI chatbot to all employees globally" is, read architecturally, a *change-management* fact as much as a systems fact: a group-wide tool, group-wide training, and group-wide productivity use cases (§6.2).
- **The 2025–26 wave** ✅ — agentic onboarding (§4), the genAI adviser-training programme, and the WoW avatar-banking unveiling (§6.3) show the estate moving from assistant (chatbot) to agent (onboarding workflow) to experience (avatar) — the maturity curve of the OCBC AI estate in three acts ⚠ the maturity framing is this guide's synthesis, flagged as interpretation.

---
## 7. Security and Fraud Systems

### 7.1 The Anti-Scam Stack: From the 2021 Scams to Kill Switch, Money Lock, OneToken

OCBC's security estate is the most publicly narrated part of its systems landscape — because the bank lived through Singapore's most damaging banking scam episode and rebuilt around it. The honest timeline:

- **The December 2021 SMS-phishing wave** ⚠ — OCBC customers were hit by a large SMS-phishing campaign (fake bank alerts redirecting to phishing sites; one-time passwords harvested). The commonly cited toll — **~469 customers and ~S$13.7 million in losses** — was widely reported in Singapore media and parliamentary discourse ⚠ (reported figures, flagged; not re-verified from primary sources this pass). The episode is the *event* behind everything below; the sector-wide response (industry kill-switch commitments, MAS guidance) followed in 2022 ✅-ish ⚠ (industry response is well documented; the MAS specifics are flagged).
- **The Kill Switch** ✅ — OCBC introduced a **"kill switch"** — a customer-activatable control that instantly freezes the account/online access when a customer suspects compromise (self-imposed freeze; liftable via verified channels). It is brand-marketed in OCBC's own anti-scam campaign materials ("safety features like Money Lock, Kill Switch, and OneToken" — the bank's anti-scam advertising) and was one of the first bank kill switches rolled out in Singapore post-2021 ⚠ the exact launch month is flagged (2022).
- **Money Lock** ✅ — the account-funds-lock feature (customer sets aside funds in a locked balance that cannot be moved via digital channels — defeatable only through verified physical/secure channels). Again brand-marketed (OCBC anti-scam ads name it). The Money Lock concept became an industry standard in Singapore (MAS encouraged it across banks) ⚠ industry-wide adoption flagged.
- **OneToken** ✅ — the token-based authentication control (§3.2) also sold as an anti-scam feature: transaction approval on the device with visible details, blocking credential-harvesting paths.
- **App-level device protections** ✅ — the August 2023 security feature that **blocks login to Internet Banking / OCBC Digital when a risky, non-official-app-store app is detected** on the device (Mothership, August 2023) — a device-risk check in the channel layer, an unusual and publicised control.
- **The anti-scam operations unit** ✅ — the Straits Times (November 2025) profiled **OCBC's anti-scam unit** and its investigators — a human operations layer (investigation, victim intervention, "we couldn't save her from herself" cases) on top of the automated controls. The unit is the operational arm of the bank's scam response; the ST piece is the verified evidence of its existence and role.
- **Anti-mule controls** ⚠ — OCBC publicly participates in the industry's anti-mule efforts (detecting and freezing mule accounts that launder scam proceeds; Singapore banks report and restrict mule accounts under ABS/MAS-coordinated measures). **Specific OCBC anti-mule product names or metrics were not verified in this pass** — flagged: the *capability* is structural and well-documented sector-wide (see [Financial Fraud Detection at Scale Guide](financial_fraud_detection_at_scale_guide.md)), the *OCBC-specific* implementation is not public.
- **Fraud AI** ⚠ — OCBC's AI estate (§6) is described by the bank as including fraud/scam detection, but the *specific* fraud-AI product names and model metrics were not verified in this pass — flagged (the general AI-in-fraud platform class is in [Financial Fraud Detection at Scale Guide](financial_fraud_detection_at_scale_guide.md)).

### 7.2 The Security Table

| System / control | Function | Notes |
|---|---|---|
| **Kill Switch** | Customer-activatable freeze of online access/account when compromise is suspected | ✅ brand-marketed (OCBC anti-scam ads); ⚠ launch month flagged (2022 era) |
| **Money Lock** | Locked-funds balance not movable via digital channels | ✅ brand-marketed; industry-wide concept post-2021 |
| **OneToken** | Token-based 2FA for logins and transaction approval | ✅ verified product; anti-scam framing |
| **Risky-app login block** | Device-risk check blocking login when risky third-party apps are present | ✅ Mothership (Aug 2023); OCBC's own messaging |
| **Anti-scam unit** | Human investigation/intervention layer for scam cases | ✅ Straits Times (Nov 2025) |
| **Anti-mule controls** | Detection/restriction of mule accounts laundering scam funds | ⚠ structural, sector-wide; OCBC specifics not public |
| **Fraud AI / transaction monitoring** | ML-based scam/fraud detection across payment flows | ⚠ capability implied by §6 estate; specifics not public |
| **Dec-2021 scam response context** | The ~469-customer / ~S$13.7M phishing episode that drove the rebuild | ⚠ reported figures, flagged |

### 7.3 The Regulatory and Industry Context

OCBC's security estate is inseparable from the Singapore scam-response ecosystem it operates in — the context an architect needs to read the controls correctly:

- **The sector-wide kill-switch mandate** ⚠-✅ — following the December 2021 scam wave, MAS and the Association of Banks in Singapore drove a sector-wide response: all retail banks were directed to implement customer-activatable **kill switches** and to harden token/OTP controls (industry-wide, 2022). OCBC's Kill Switch is the *bank-specific* implementation of this sector mandate ⚠ (the mandate's exact MAS instrument and dates are flagged; the direction is well documented).
- **Money Lock as an industry standard** ⚠ — the account-funds-lock concept was subsequently pushed across Singapore banks (MAS encouraged banks to offer money-locking features); OCBC's Money Lock is the OCBC-branded implementation, and the feature is now near-universal among SG retail banks.
- **The Shared Responsibility Framework (SRF)** ⚠ — IMDA/MAS's phishing-scam liability framework (announced 2024, effective December 2024) allocates loss responsibility between banks, telcos, and consumers for phishing scams; it formalises exactly the controls OCBC markets (the "bank-side" duties include kill-switch availability, transaction limits, and fraud monitoring). The SRF is the regulatory capstone of §7 ⚠ specifics flagged (see [Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md) for the compliance-platform class).
- **Anti-mule as a sector programme** ⚠ — ABS-coordinated mule-account reporting and restrictions are sector practice; OCBC participates structurally, specifics not public (§7.1).
- **The systems consequence**: the security estate is *regulation-shaped* — every control in §7.2 maps to a regulatory expectation (kill switch → SRF/mandate; Money Lock → SRF; transaction monitoring → MAS notice on fraud detection; device-risk checks → phishing defence expectations). OCBC's public marketing of these controls is, in part, the bank demonstrating compliance-plus ⚠ framing, flagged as interpretation.

---

## 8. Enterprise Architecture

### 8.1 The Data Platform and Cloud Strategy

The enterprise-architecture layer is where OCBC's public record is *thin but real* — two verified anchors and a clear inference around them:

- **The Cloudera data platform (verified)** ✅ — Cloudera's own case study, "OCBC Bank Accelerates Its Data Strategy with Cloudera" (December 2022), documents OCBC's enterprise data platform build. The follow-on is the **May 2025 Asian Banker award** ("Best Data and Analytics Infrastructure Initiative"): OCBC and Cloudera *"re-platformed its legacy data warehouse into a real-time, analytics-ready infrastructure using a hybrid open-source stack and data mesh principles"*, *"supporting hundreds of users"* with improved data-provisioning times ✅. Reading the two together: **OCBC's data estate is a hybrid open-source, data-mesh-architected platform on Cloudera**, replacing a legacy warehouse — the spine that the AI estate (§6) and the anti-fraud estate (§7) consume.
- **The cloud-migration-at-scale claim** ⚠ — a 2026 industry write-up on OCBC's migration describes moving *"tens of thousands of producers and consumers across Singapore, Malaysia, and Hong Kong"* with zero data loss — i.e., an event-streaming-scale migration of the integration fabric. This is flagged: the *scale and direction* (large-scale streaming migration across the three hubs) are plausible and sourced from an industry case-study summary, but the specific hyperscaler/vendor names in it were not verified this pass.
- **The multi-cloud/hyperscaler question** ⚠ — **not verified in this pass.** Unlike DBS's documented multi-cloud posture ([DBS Software Systems Guide](dbs_software_systems_guide.md) §1.4), no OCBC–AWS/Google/Azure strategic agreement surfaced in the snippets reviewed. The honest statement: OCBC's *data* cloud is Cloudera-based (verified); its *compute* cloud strategy is not publicly pinned in this pass — flagged.
- **The legacy-core tension** ⚠ — every modernisation fact above (§3 channels, §6 AI, §8 data) operates *in front of* the AS/400-era core of §2. The architecture is the standard **strangler-fig over a vendor legacy core** (the pattern generalised in [Core Banking Systems Guide](core_banking_systems_guide.md)): new capabilities land in the cloud/data/AI layer, the core is wrapped and progressively decomposed, and the migration fabric (event streaming) carries the traffic. This is inferred from the verified pieces — the *shape* is certain, the *specifics* (API gateway vendors, event-bus vendor, microservice catalogue) are not public.

### 8.2 The Architecture Table

| Layer | System | Notes |
|---|---|---|
| **Data platform** | Cloudera-based, hybrid open-source stack, data-mesh principles | ✅ Cloudera (Dec 2022) + Asian Banker award (May 2025); legacy-warehouse re-platform |
| **Integration fabric** | Event-streaming migration across SG/MY/HK; tens of thousands of producers/consumers | ⚠ industry write-up (2026); scale/direction plausible, vendor specifics unverified |
| **Compute cloud** | Hyperscaler strategy | ⚠ not verified this pass |
| **Core estate** | Silverlake SIBS on IBM AS/400 (§2) | ✅ vendor-verified; the legacy anchor being wrapped |
| **Channel layer** | OCBC Digital, Internet Banking, OCBC Business (§3) | ✅ product layer verified; middleware ⚠ |
| **AI estate** | AI Lab, genAI chatbot, agentic onboarding (§6) | ✅ programme layer verified |
| **Security layer** | Kill Switch, Money Lock, OneToken, device-risk, anti-scam unit (§7) | ✅ controls verified; fraud-AI internals ⚠ |
| **Architecture pattern** | Strangler-fig over a vendor legacy core; data-mesh analytics; event-driven integration | ⚠ structural inference from verified pieces |

### 8.3 The Reference Architecture: How the Verified Pieces Fit

Synthesising the verified anchors of §2–§8 into the OCBC reference architecture (the OCBC-specific version of the DBS diagram in [DBS Software Systems Guide](dbs_software_systems_guide.md) §9.1 — presented as this guide's structural reading, ⚠ where inference):

| Layer | Verified components | Inferred/not public |
|---|---|---|
| **Channels** | OCBC Digital, Internet Banking, OCBC Business, OneToken (§3) ✅ | Channel middleware, mobile engineering stack ⚠ |
| **Payments edge** | FAST/PayNow participation, PayAnyone legacy (§5) ✅ | Payments hub, ISO 20022 orchestration, card platform ⚠ |
| **Core services** | Silverlake SIBS on AS/400 (regional, vendor-verified §2) ✅ | SG flagship core identity, BOS core, markets stack ⚠ |
| **Data & AI** | Cloudera data platform, data-mesh, AI Lab/genAI estate (§6, §8) ✅ | Model registry, feature store, ML-ops specifics ⚠ |
| **Security** | Kill Switch, Money Lock, OneToken, device-risk, anti-scam unit (§7) ✅ | Fraud-model internals, case-management stack ⚠ |
| **Integration fabric** | "Tens of thousands of producers/consumers" migration across SG/MY/HK (§8.1) ⚠ | Event-bus vendor, API-gateway vendor ⚠ |
| **Infrastructure** | Cloud migration at scale (§8.1) ⚠ | Hyperscaler strategy, region/DR topology ⚠ |

Three architectural truths fall out of this map:

1. **The estate is layered like every incumbent's — but the flags differ by layer.** OCBC's public record is strongest at the *edges* (channels, security controls, AI programmes) and at the *data* layer (Cloudera), and weakest in the *middle* (payments hub, middleware, core identities). An architect sizing OCBC's estate should treat the middle layers as standard-issue but *unverifiable* — the honest position of §11.
2. **The data platform is the strategic spine.** The Cloudera/data-mesh build (✅) is what feeds the AI estate (§6), the fraud screens (§7), and the personalisation promises (§3.1) — the same "data is the moat" conclusion the DBS guide draws ([DBS Software Systems Guide](dbs_software_systems_guide.md) §7), and the most clearly documented modernisation in the OCBC estate.
3. **Modernisation is edge-first, core-last.** Every verified modernisation (digital channels, AI, data platform, security) sits outside the core; the core itself is being *refreshed and wrapped* (Silverlake case study ✅), not replaced — the strangler-fig at its most orthodox. The risk profile this implies (legacy-core dependency behind a fast-moving edge) is exactly the one the DBS outages exposed ([DBS Bank Guide](dbs_bank_guide.md) §5.5) ⚠ comparative observation, flagged.

---

## 9. Worked Example: An OCBC Digital Customer Journey

### 9.1 The Scenario

**The customer:** a Singapore retail user of OCBC Digital — salaried, holds a CASA account and a credit card, has OneToken enabled, has enrolled in PayNow with her mobile number, and has set aside funds in Money Lock. **The journey:** a Saturday morning — she checks her balance, receives her pay, pays a friend via PayNow, spots a suspicious "phishing" SMS, and uses the Kill Switch before calling the bank.

This journey is deliberately the OCBC-verified-surface version of the DBS digibank journey in [DBS Software Systems Guide](dbs_software_systems_guide.md) §10 — the same retail story, told through OCBC's documented controls, so the two guides are directly comparable.

### 9.2 The Flow (with verification flags per step)

| Step | What happens | Systems involved | Evidence |
|---|---|---|---|
| 1. **Login** | She opens OCBC Digital; approves login with **OneToken** (device token) — the device-risk check passes (no risky third-party apps) | OCBC Digital app, OneToken, device-risk check, channel-integration layer | ✅ app + OneToken + risky-app block verified (§3, §7) |
| 2. **Balance check** | Balance and recent transactions render — the app reads the account position through the integration layer; for a legacy core, the read is either a real-time enquiry or a near-real-time mirror (⚠ the balance path's real-time-ness is not public) | Core services (§2), channel layer | ⚠ core path inferred |
| 3. **Pay arrives** | Salary credits via FAST-originated bulk/individual credit; the post settles in the core and appears in the app | FAST rail, core posting, event/data pipeline | ✅ rails verified (§5); ⚠ posting specifics |
| 4. **PayNow payment** | She pays a friend using the friend's mobile number: PayNow proxy lookup → **FAST** clearing → the core posts the debit and the friend's bank posts the credit — near-instant | PayNow overlay, FAST rail, OCBC Digital journey, core | ✅ rails verified; ⚠ internal orchestration |
| 5. **Fraud-screen pass** | The payment crosses transaction-monitoring/fraud scoring — the ML layer (the §6/§7 AI estate) screens the behaviour; no alert, payment proceeds | Fraud AI / transaction monitoring, data platform | ⚠ OCBC-specific internals not public |
| 6. **The scam attempt** | A spoofed SMS claims her account is compromised and asks her to click a link and approve a token prompt. She notices the red flags (the bank's anti-scam education) and does *not* approve | (the attack surface the controls defend) | ✅ context verified (§7.1) |
| 7. **Kill Switch** | She activates the **Kill Switch** in the app — online access/account actions freeze instantly; the locked state blocks any further digital movement | Kill Switch control, channel layer, core freeze | ✅ control verified; ⚠ internal mechanics |
| 8. **Money Lock intact** | Her locked funds were never exposed to the digital channel — Money Lock held them aside | Money Lock | ✅ control verified |
| 9. **Call the bank** | She calls OCBC — the **anti-scam unit** takes the case, verifies identity, reviews the attempted compromise, and guides the lift of the Kill Switch | Anti-scam unit, CRM/case systems | ✅ unit verified (ST Nov 2025) |
| 10. **Recovery** | After verification, access is restored; the fraud case feeds the data estate for future detection | Data platform, fraud models | ⚠ internals inferred |

### 9.3 The Lessons

1. **The rails are the shared public good** — every money movement in the journey (salary, PayNow) runs on FAST; OCBC's own systems are the participant edge, not the backbone (§5). The bank's payment reliability is the rail's reliability plus its own edge.
2. **The verified differentiator is the security layer, not the core** — the journey's OCBC-specific steps (6–9) are all post-2021 anti-scam controls. An architect reading OCBC's public record sees the same wrapped-legacy-core pattern as every incumbent (DBS included), but a **consumer-facing security narrative** that DBS and StanChart do not market with the same specificity.
3. **AI shows up where the documents are** — the journey's hidden AI (fraud scoring) is complemented by OCBC's *documented* AI wins: employee chatbot, agentic onboarding, adviser training (§6). The pattern: OCBC applies AI at the friction points (onboarding documents, adviser training, scam detection) rather than advertising a single "AI platform".
4. **The legacy core is invisible by design** — in the whole journey, the core appears only as "balance" and "post" operations. That is the point of the strangler-fig architecture (§8): the AS/400 estate is a service provider, not a customer surface — and the reason OCBC can ship the §6/§7 innovation while running a vendor legacy core.

### 9.4 The Corporate Sibling: The Same Journey Through OCBC Business

The retail journey is one half of the estate; the corporate journey through **OCBC Business** (the SME/corporate channel of §3.2) is the mirror — the same architecture wearing a different suit (compare the DBS IDEAL parallel in [DBS Software Systems Guide](dbs_software_systems_guide.md) §10.5):

1. **Login**: OCBC Business with OneToken-class 2FA ⚠ → the same channel-integration layer.
2. **Balances**: corporate cash positions across accounts and entities — the same core services, aggregated with business views ⚠ (the OCBC Business app's sales/expense-trend dashboards are OCBC's own marketing example of embedded analytics ✅).
3. **Payment**: a supplier payment → the same payment edge: FAST/PayNow (UEN-based PayNow for corporate proxies), GIRO/SWIFT for the heavier flows ⚠ — with **maker/checker entitlements and corporate limits** layered on (the standard corporate-controls pattern ⚠).
4. **Trade/FX** ⚠: trade finance and FX flows through the wholesale estate (§2.2 markets stack) — the wholesale banking systems of the GWB division (§1.4).
5. **ERP integration** ⚠: corporate clients' ERP/treasury systems connecting to OCBC Business via APIs — the standard corporate-banking integration surface; OCBC's API/developer programme for corporates was not verified this pass.

The architect's takeaway, mirroring the DBS guide's: **OCBC Digital and OCBC Business are one architecture at two scales** — one core family, one payment edge, two engagement layers. The security layer (§7) is shared; only the entitlements, limits, and workflows (maker/checker, delegation of authority) differ. That is the OCBC platform thesis in one sentence.

### 9.5 What the Journey Reveals About the Stack

Reading the two journeys back against §1–§8, the system-level truths about OCBC:

1. **The core is a service, not a spectacle** — the AS/400 estate appears only as balance/post operations; its vendor identity (Silverlake, ✅) is *more* public than DBS's anonymous core, yet equally invisible in the customer experience — the strangler-fig working as designed (§2, §8).
2. **The security layer is the system of record for trust** — the journey's most OCBC-specific steps (Kill Switch, Money Lock, the anti-scam unit) are where OCBC's public narrative and its architecture coincide: safety is engineered into the channel, the payment edge, and the operations unit simultaneously (§7).
3. **The data/AI estate is the silent co-star** — fraud scoring, personalisation, the adviser-training content, and the onboarding agents all run on the Cloudera/data-mesh platform (§8.1) and the AI estate (§6); the customer sees effects, not systems.
4. **The payment edge is thin by design** — OCBC does not own the rails (FAST/PayNow are utilities); its payment systems are the participant edge plus the bank-owned overlays (PayAnyone's legacy, Money Lock's restrictions). The reliability question for OCBC is therefore the *edge*'s reliability, not the rail's (§5).
5. **The whole estate is one regulation-shaped organism** — the SRF and MAS expectations (§7.3) dictate the shape of the security controls, the transaction monitoring, and even the marketing; an architect reading the OCBC estate without the regulatory context reads only half the design.

---

## 10. Summary: The Oldest Local Bank's Modern Stack

**One-page summary.** OCBC — the Oversea-Chinese Banking Corporation, Singapore's longest-established bank (1912 lineage, 1932 incorporation) — runs a systems estate that is best described as **a vendor legacy core under a modern digital/AI layer, differentiated by security and by AI-firsts**.

- **The core is named, rare among peers**: **Silverlake SIBS on IBM AS/400** (vendor + IDC verified) — a legacy-class estate for the regional books; the Singapore flagship core's identity is not public (⚠). No Temenos/FLEXCUBE/BaNCS relationship verified.
- **The channels are consolidated and public**: **OCBC Digital** (the bank's mobile app), **OCBC Internet Banking**, **OCBC Business**, OneToken authentication; the Pay Anyone app retired into OCBC Digital (2023) ✅.
- **The rails are Singapore's rails**: **FAST** (2014) and **PayNow** (2017), with cross-border links to UPI/PromptPay/DuitNow ✅.
- **The wealth estate is where AI is most concrete**: Bank of Singapore's agentic-AI onboarding (15 days vs six weeks), genAI adviser training, an open-architecture private-bank platform (vendor ⚠) ✅.
- **AI is the documented moat**: AI Lab (2018) → first-SG-bank employee genAI chatbot (2023) → avatar banking OCBC WoW (2026) ✅, on a **Cloudera-based, data-mesh data platform** (✅) that replaced a legacy warehouse.
- **Security is the public brand**: Kill Switch, Money Lock, OneToken, device-risk login blocks, an anti-scam unit — all built in the shadow of the December 2021 phishing episode ⚠ figures flagged.
- **The architecture** is the standard incumbent strangler-fig: new capabilities on the data/AI/cloud estate, a wrapped AS/400-era core, event-streaming integration fabric (⚠ scale flagged, vendor specifics not public).

**The one-page scorecard:**

| Dimension | OCBC's position | Evidence class |
|---|---|---|
| **Core** | Silverlake SIBS on IBM AS/400 (regional); vendor-named, legacy-class, wrapped | ✅ vendor-verified (§2) |
| **Channels** | OCBC Digital consolidated flagship; Internet Banking; OCBC Business; OneToken | ✅ (§3) |
| **Rails** | FAST/PayNow participant; UPI/PromptPay/DuitNow cross-border reach | ✅ rails; ⚠ participation details (§5) |
| **Wealth** | BOS open-architecture private bank; agentic onboarding; genAI adviser training | ✅ programme-level; ⚠ core vendor (§4) |
| **AI** | AI Lab (2018) → employee genAI chatbot (2023, first SG bank) → agentic → WoW (2026) | ✅ programme-level (§6) |
| **Security** | Kill Switch, Money Lock, OneToken, device-risk, anti-scam unit | ✅ controls; ⚠ internals (§7) |
| **Data** | Cloudera data platform, data-mesh, legacy-warehouse re-platform | ✅ (§8) |
| **Cloud/compute** | Migration at scale across SG/MY/HK | ⚠ scale flagged; hyperscalers not public (§8) |

**The final word: the oldest local bank runs a modern stack where the legacy is vendor-named, the innovation is AI-first, and the public narrative is safety.** OCBC is the mirror-image of DBS in this series: DBS hides a mostly in-house estate behind journey stories ([DBS Software Systems Guide](dbs_software_systems_guide.md) §11); OCBC names its core vendor, publishes its AI firsts, and markets its scam defences — while keeping the middleware, the wealth core, and the cloud compute layer out of the public record. For the architect, the OCBC estate is the cleanest available study of a **large Southeast-Asian incumbent modernising in front of a vendor legacy core** — with the security layer as its most distinctive system of all.

---

## 11. Claims Status and Verification Notes

**Verification convention: ✅ = verified in this research pass (primary/secondary sources); ⚠ = flagged (inferred, approximate, single-source, or structural); "not public" = OCBC does not disclose and no external source exists either way.**

### The Claims-Status Table

| Claim | Source | Status |
|---|---|---|
| OCBC is the longest-established Singapore bank; formed 1932 from the merger of three local banks, the oldest founded 1912 (Chinese Commercial Bank 1912; Ho Hong 1917; Oversea-Chinese 1919) | OCBC's own brand-refresh media release (Jul 2023); multiple directory/analyst sources | ✅ verified |
| Group CEO Helen Wong since 2021; US$10B total income FY2023 | Straits Times/Forbes power-list coverage | ✅ verified (reported) |
| ~30,000 employees across the OCBC Group | OCBC release (2021, sustainability training) | ✅ verified (reported) |
| Second-largest financial services group in Southeast Asia by assets | Cloudera case study (Dec 2022) | ✅ verified (secondary source) |
| Segments: Global Consumer/Private Banking, Global Wholesale Banking, Global Markets, Insurance, Others | OCBC Annual Reports 2023/2024 (segment note) | ✅ verified |
| Core = Silverlake Axis SIBS on IBM AS/400 (Power Systems) | Silverlake Axis case study "OCBC: A Truly World-Class Core Banking System" (excerpted from IDC case study, Sep 2013; Mar 2015) | ✅ vendor-verified; ⚠ per-franchise scope |
| Singapore flagship core identity; NISP core; BOS core vendor; markets/treasury stack vendor | No primary source in this pass | ⚠ not public / inferred |
| OCBC Digital = the bank's mobile app for digital banking; Pay Anyone discontinued Sep 2023 and consolidated into OCBC Digital | Straits Times (Sep 2023), quoting OCBC; App Store/Play Store listings | ✅ verified |
| OneToken; risky-app login block | OCBC anti-scam advertising; Mothership (Aug 2023) | ✅ verified |
| In-app calling added Nov 2025 (OCBC and OCBC Business apps) | Wikipedia (OCBC Bank, 2025–26 update) | ✅ verified (secondary) |
| Bank of Singapore = OCBC's private banking subsidiary, open-architecture platform | bankofsingapore.com; Wikipedia | ✅ verified |
| Agentic-AI onboarding: 15 business days vs ~6-week industry median | The Asian Banker | ✅ verified (reported) |
| GenAI wealth-adviser skills training | OCBC release (2026, avatar banking); Business Times | ✅ verified (reported) |
| FAST (2014) and PayNow (2017) rails; UPI/PromptPay/DuitNow links; OCBC PayNow participation | ABS; RTP Dashboard; OCBC PayNow help pages | ✅ verified |
| PayNow founding-participant claim | Industry-standard knowledge | ⚠ reported, not re-verified |
| AI Lab@TOV (2018); postgraduate AI scholarship (2019); "From Learning AI to doing AI" blog | OCBC newsroom/careers | ✅ verified |
| First SG bank to roll out genAI chatbot to all employees globally (2023) | OCBC newsroom release (2023) | ✅ verified (OCBC's claim) |
| "Learning AI" as a named in-house tool; 30,000-staff figure attached to the chatbot rollout | Only the headcount (~30k, 2021) and the careers-blog phrase verified | ⚠ name/figure coupling flagged |
| OCBC WoW avatar banking (2026) | OCBC newsroom | ✅ verified |
| Kill Switch, Money Lock (brand-marketed anti-scam controls) | OCBC anti-scam campaign materials | ✅ verified (controls); ⚠ launch dates |
| Anti-scam unit and investigators | Straits Times (Nov 2025) | ✅ verified |
| Dec 2021 phishing episode: ~469 customers, ~S$13.7M | Widely reported (SG media/parliamentary discourse) | ⚠ reported figures |
| Anti-mule controls; fraud-AI product specifics | Sector-wide practice; no OCBC specifics surfaced | ⚠ structural / not public |
| Cloudera data platform; legacy-warehouse re-platform; data-mesh; Asian Banker award (May 2025) | Cloudera blog (Dec 2022); The Asian Banker (May 2025) | ✅ verified |
| Cloud migration of tens of thousands of producers/consumers across SG/MY/HK | Industry case-study write-up (2026) | ⚠ single-source, scale flagged |
| Hyperscaler/multi-cloud strategy (AWS/Google/Azure) | Not found in this pass | ⚠ not verified |
| Temenos / FLEXCUBE / BaNCS at OCBC | No evidence found | ⚠ unverified / not found |

### What Could Not Be Verified

The deliberate negative results of this pass — claims the brief asked about that this research could neither confirm nor refute, and which therefore stay unasserted:

- **A named "Learning AI" product/tool** — the phrase is OCBC's careers-blog framing ("From Learning AI to doing AI"); no OCBC release naming a tool "Learning AI" surfaced. The verified tool is the 2023 employee genAI chatbot. ⚠
- **The ~30,000-staff figure attached to the 2023 genAI rollout** — the headcount is verified; the coupling to the chatbot announcement is not in OCBC's own words. ⚠
- **The Singapore flagship core's identity** (is it SIBS too? another system?) — not public. ⚠
- **Bank of Singapore's core platform** (Avaloq? Temenos? in-house?) — not public. ⚠
- **OCBC NISP's core and BI-FAST participation** — inferred, not verified. ⚠
- **OCBC's hyperscaler agreements** (AWS/Google/Azure) — no strategic agreement surfaced this pass. ⚠
- **The payments-hub and middleware vendors** (ISO 20022 orchestration, API gateway, event bus) — not public. ⚠
- **Fraud-AI product names and model metrics** — not public. ⚠
- **The December 2021 scam episode's exact figures** (~469 customers / ~S$13.7M) — widely reported, not primary-source-verified here. ⚠
- **Great Eastern / OCBC NISP stake percentages and acquisition dates** — flagged at §1.2; not re-verified this pass. ⚠
- **OCBC's technology headcount and spend** — no DBS-style disclosure exists. ⚠

### The Honesty Note: Methodology

**OCBC is more disclosing than DBS and less disclosing than Standard Chartered about its systems.** The verified anchors are unusually strong in three places — the **core vendor** (Silverlake/IDC), the **AI programme** (OCBC newsroom releases), and the **security controls** (brand-marketed + press) — and unusually weak in two: **per-franchise core coverage** and the **middleware/cloud compute layer**.

1. **Verified anchors**: the 1912/1932 history; the ~30,000 headcount and segment structure; Silverlake SIBS on AS/400; OCBC Digital + Pay Anyone consolidation; OneToken; FAST/PayNow participation; AI Lab (2018), the employee genAI chatbot (2023), agentic onboarding, WoW (2026); Kill Switch/Money Lock/anti-scam unit; the Cloudera data platform.
2. **Inference rules used**: (a) *structural inference* — the middleware, payments hub, card estate, and markets stack must exist in the standard platform classes ([Payments Hub Guide](payments_hub_guide.md), [Nasdaq Calypso Guide](nasdaq_calypso_guide.md)) even though OCBC does not name them; (b) *vendor-market inference* — where an industry survey names the two dominant platforms (Avaloq/Temenos for Asian private banks), the claim is flagged ⚠ and never promoted to fact about BOS; (c) *scope caution* — the Silverlake case study's franchise coverage is treated as the vendor says it, not as a group-wide statement.
3. **Deliberately not verified** (out of scope/undisclosed): internal system names, middleware/API gateway vendors, hyperscaler contracts, BOS's core platform, NISP core specifics, fraud-model metrics, exact scam-episode figures, and the current WoW product details.
4. **The reader's rule**: if a system name in this guide lacks a ✅ in §11, treat it as a hypothesis about a class of system, not a fact about OCBC.

### The Verification Pass: What Was Searched and What It Returned

For auditability, the research pass behind this guide (~15 targeted searches; web_extract was unavailable — search-only backend — so all evidence is from search-result surfaces):

- **History/group**: "OCBC Bank history founded 1912 oldest local bank Singapore merger 1932" → OCBC's own brand-refresh release (the 1912/1932 formulation), plus directory sources ✅.
- **Core**: "OCBC core banking system Silverlake Axis Temenos implementation" → **Silverlake Axis's OCBC case study (SIBS on IBM AS400)** and the IDC-study provenance ✅ — the anchor of §2. No Temenos/FLEXCUBE/BaNCS OCBC relationship surfaced ⚠.
- **Digital**: "OCBC Digital app rename … 2023" → Straits Times (Sep 2023) Pay Anyone consolidation, naming OCBC Digital as the bank's mobile app; App Store/Play listings ✅; Mothership (Aug 2023) for the risky-app feature.
- **Payments**: "OCBC PayNow founding participant 2017 FAST" → ABS PayNow pages, RTP Dashboard (FAST 2014 / PayNow 2017 / UPI-PromptPay-DuitNow links), OCBC PayNow help pages ✅.
- **AI**: "OCBC 'Learning AI' … 2023" (no direct hit) → followed by site:ocbc.com newsroom search → **the 2023 employee-genAI-chatbot release**, AI Lab@TOV (2018), AI scholarship (2019), the "From Learning AI to doing AI" careers blog, and the 2026 WoW release ✅. The "Learning AI" tool-name claim stays ⚠.
- **Wealth**: "Bank of Singapore … Avaloq/Temenos" → Asian Private Banker industry survey (Avaloq vs Temenos as the two Asia private-bank platforms) ⚠; **The Asian Banker agentic-AI onboarding piece** (15 days vs six weeks) ✅; ST/BT wealth-advisory-online coverage ✅.
- **Security**: "OCBC anti-scam kill switch money lock fraud detection AI" → OCBC anti-scam advertising naming Kill Switch/Money Lock/OneToken ✅; Straits Times anti-scam-unit profile (Nov 2025) ✅.
- **Architecture**: "OCBC … cloud/data" → **Cloudera case study (Dec 2022)** and the **Asian Banker data-infrastructure award (May 2025)** ✅; a 2026 industry write-up on the tens-of-thousands-scale migration ⚠ single-source; no hyperscaler agreement surfaced ⚠.
- **Negative results (deliberately recorded)**: no evidence for OCBC–Temenos/FLEXCUBE/BaNCS, OCBC–Avaloq, an OCBC hyperscaler strategic agreement, a named "Learning AI" product, or fraud-AI product names — each recorded as ⚠ not verified rather than asserted either way.

**Residual risk statement**: the ⚠-inferred sections (§2.2 franchise coverage, §4 BOS platform, §8 compute-cloud) are the most likely to be corrected by new information; the ✅ anchors (§1 history, §2.1 Silverlake, §3.1 OCBC Digital, §5.1 rails, §6 AI, §7 controls, §8.1 Cloudera) are stable. Any citation-safe use of this guide should carry the §11 table rather than inline flags alone.

---

## 12. Glossary

| Term | Definition |
|---|---|
| **OCBC** | Oversea-Chinese Banking Corporation — Singapore's longest-established bank (SGX: O39), formed 1932 from the merger of three local banks, the oldest founded 1912; a universal banking group (retail, wholesale, markets, private banking, insurance) |
| **Oversea-Chinese Banking Corporation** | The full legal name of the OCBC group; incorporated 1932 via the merger of Chinese Commercial Bank (1912), Ho Hong Bank (1917), and Oversea-Chinese Bank (1919) |
| **Bank of Singapore** | OCBC's private-banking subsidiary ("Asia's global private bank") — high-net-worth wealth management on an open-architecture platform |
| **Great Eastern** | Great Eastern Holdings — OCBC's insurance arm (life and general), Singapore/Malaysia; the group's Insurance segment |
| **OCBC NISP** | PT Bank OCBC NISP — OCBC's Indonesian bank; a distinct franchise with its own core and channel estate |
| **Core banking** | The system(s) of record for deposits, loans, and accounts; at OCBC, vendor-verified as Silverlake Axis SIBS on IBM AS/400 for the regional estate |
| **Digital banking** | The online/mobile channel layer over the core: OCBC Digital (app), OCBC Internet Banking, OCBC Business |
| **OCBC Digital** | OCBC's mobile banking app — "the bank's mobile app for digital banking services" (OCBC/ST 2023); inherited Pay Anyone's functions in the Sep 2023 consolidation |
| **Online banking** | Internet Banking — OCBC's web channel for personal (and business) banking |
| **Mobile banking** | The app-based channel — OCBC Digital and OCBC Business apps |
| **Wealth management** | The advisory/investment layer: Bank of Singapore (private), Premier/Treasures (affluent), and wealth features inside OCBC Digital |
| **Payments** | The movement of money: domestic rails (FAST/PayNow), cards, and the bank's payment surfaces (OCBC Digital journeys, retired Pay Anyone) |
| **FAST** | Fast And Secure Transfers — Singapore's real-time interbank clearing rail since 2014; the backbone for all domestic instant payments |
| **PayNow** | Singapore's proxy-based instant-payment overlay on FAST (2017): pay by mobile number, NRIC/FIN, UEN, or VPA; cross-border links to UPI, PromptPay, DuitNow |
| **AI** | Artificial intelligence — at OCBC, an estate spanning the AI Lab (2018), fraud/scam detection, and the genAI programme |
| **Learning AI** | OCBC's own phrase for its AI upskilling journey ("From Learning AI to doing AI"); ⚠ verified as programme framing, not as a named tool |
| **genAI** | Generative AI — OCBC's 2023 employee chatbot (first SG bank to roll one out to all staff globally), agentic onboarding, adviser training, avatar banking (WoW, 2026) |
| **Anti-scam** | OCBC's post-2021 scam-defence estate: Kill Switch, Money Lock, OneToken, device-risk checks, and the anti-scam unit |
| **Anti-mule** | Controls to detect and restrict mule accounts laundering scam proceeds; sector-wide practice, OCBC specifics not public |
| **Fraud** | Fraud detection — the ML transaction-monitoring layer of the AI estate; OCBC-specific models not public |
| **Cloud** | The modern infrastructure layer: Cloudera-based data platform (verified), event-streaming integration fabric (flagged), hyperscaler strategy not public |
| **Enterprise architecture** | The overall systems topology: strangler-fig modernisation over the vendor legacy core, data-mesh analytics, channel/security layers |
| **Singapore** | The group's home market and the centre of its systems estate — the local rails (FAST/PayNow), MAS regulatory context, and the flagship OCBC Digital franchise |

---

## 13. References and Further Reading

**Primary/company:**
- OCBC newsroom: "OCBC First To Launch AI Chatbot For Global Employees" (2023) — ocbc.com/group/media/release/2023/…
- OCBC newsroom: "OCBC launches AI Lab" (2018, AI Lab@TOV); "OCBC Bank launches postgraduate scholarship to deepen AI talent pool" (2019); "OCBC unveils Avatar Banking with OCBC WoW" (2026)
- OCBC careers blog: "From Learning AI to doing AI"
- OCBC media release: "OCBC unifies brand" (July 2023, brand refresh and logo) — ocbc.com.mo/file/news/20230703…
- OCBC Annual Reports 2023 and 2024 — segment note (Global Consumer/Private Banking; Global Wholesale Banking; Global Markets; Insurance; Others)
- OCBC personal-banking pages: PayNow help; "The all new OCBC app. Inspired by you." (what has changed / what you will love); business-banking digital pages — ocbc.com
- OCBC anti-scam campaign materials (Kill Switch, Money Lock, OneToken)
- OCBC release (2021): sustainability training for all 30,000 employees
- bankofsingapore.com — "Asia's global private bank"

**Press/analyst (used for verification):**
- Silverlake Axis: "OCBC: A Truly World-Class Core Banking System" (Mar 2015), excerpted from IDC case study "The Tale of Two Super-Regional Banks and Their Core Banking Transformation" (Sep 2013), sponsored by Silverlake Axis
- Straits Times: "OCBC to discontinue Pay Anyone app; services consolidated on main digital banking app" (Sep 2023); "OCBC moves wealth advisory service online" (BT/ST); "'We couldn't save her from herself': How scam victim went from $130k in savings to $600 in 2 months" (Nov 2025, anti-scam unit)
- Mothership: "OCBC app new security feature prompts users to delete unofficial 3rd-party apps" (Aug 2023)
- The Business Times: "OCBC to train wealth advisers using generative AI"
- The Asian Banker: "How is OCBC using agentic AI to cut private banking onboarding times?"; "Best Data and Analytics Infrastructure Initiative goes to OCBC and Cloudera" (May 2025)
- Cloudera: "OCBC Bank Accelerates Its Data Strategy with Cloudera" (Dec 2022)
- ABS: PayNow pages; RTP Dashboard: FAST/PayNow system profile; Wikipedia: "OCBC Bank", "PayNow", "Bank of Singapore"; Asian Private Banker: "A tale of two giants: Avaloq vs. Temenos"; Straits Times/Forbes: Helen Wong profile

**Sibling guides in this repository (cross-references used throughout):**
- [DBS Software Systems Guide](dbs_software_systems_guide.md) — the pattern for this guide (systems deep-dive, worked example, claims-status discipline)
- [DBS Bank Guide](dbs_bank_guide.md), [Standard Chartered Guide](standard_chartered_guide.md) — the bank deep-dives and the structural model for the series
- [Universal Banking Model Guide](universal_banking_model_guide.md) — the group-structure reference (OCBC as universal bank)
- [Core Banking Systems Guide](core_banking_systems_guide.md), [Temenos Guide](temenos_guide.md), [Oracle FLEXCUBE Data Model Guide](oracle_flexcube_data_model_guide.md), [Apache Fineract Guide](apache_fineract_guide.md), [Chinese Bank Core Systems Guide](chinese_bank_core_systems_guide.md) — core platform classes and vendors (contrast with Silverlake SIBS)
- [T24 Programming Guide](t24_programming_guide.md), [TAFJ Guide](tafj_guide.md), [JBASE Universe Guide](../technology/jbase_universe_guide.md) — the Temenos-side cores (only if OCBC were Temenos-adjacent; no verified relationship)
- [Payments Hub Guide](payments_hub_guide.md), [ISO 20022 Core Processes Guide](iso_20022_core_processes_guide.md), [Financial Infrastructure Guide](financial_infrastructure_guide.md) — payment rails and hubs (FAST/PayNow)
- [Wealth Management Guide](wealth_management_guide.md) — the wealth-platform domain (BOS context)
- [Insurance Software Systems Guide](insurance_software_systems_guide.md) — the Great Eastern angle (policy administration, claims)
- [Financial Fraud Detection at Scale Guide](financial_fraud_detection_at_scale_guide.md) — the anti-scam/anti-fraud platform class
- [Interest Engines Core Banking Guide](interest_engines_core_banking_guide.md), [Posting Engine Core Banking Guide](posting_engine_core_banking_guide.md), [Core Banking Processes Guide](core_banking_processes_guide.md) — core accounting mechanics
- [Nasdaq Calypso Guide](nasdaq_calypso_guide.md), [Trust Bank Guide](trust_bank_guide.md) — treasury platform class and the SG digital-bank comparison
- [Data Governance Guide](../technology/data_governance_guide.md), [Event Stream Processing Guide](../technology/event_stream_processing_guide.md) — data-platform and integration-fabric classes

---

*End of guide. OCBC is the oldest local bank running a genuinely modern stack: a vendor-named legacy core (Silverlake SIBS on AS/400) wrapped by a consolidated digital channel (OCBC Digital), Singapore's payment rails (FAST/PayNow) at its edge, a Cloudera-based data platform underneath, an AI programme that shipped the region's first bank-wide employee genAI chatbot, and a security layer — Kill Switch, Money Lock, OneToken, the anti-scam unit — that is the most distinctive system of all. The verified anchors in §11 (the 1912/1932 history, the Silverlake core, OCBC Digital, the 2023 AI first, the anti-scam controls, the Cloudera estate) are the load-bearing facts; everything else is honest inference about a bank that names its core vendor, publishes its AI firsts, and keeps the middleware and cloud compute layer private.*
