# UOB: The Software Systems Landscape — A Comprehensive Guide to the Technology UOB Runs

*A companion deep-dive in the Singapore-bank series, following the pattern of [DBS Software Systems Guide](dbs_software_systems_guide.md) (the series pattern), [Standard Chartered Guide](standard_chartered_guide.md) (the structural model), and [OCBC Software Systems Guide](ocbc_software_systems_guide.md) (the just-added sibling — the closest structural twin). This guide focuses on the **specific software and technology systems** behind UOB — United Overseas Bank, the family-controlled Singapore bank: the core banking estate (with the extraordinary 2025 twist that UOB bought its own core vendor), the digital banking layer (Mighty → TMRW), the TMRW digital bank, the Infinity wholesale platform, the payment rails, AI and innovation, security and fraud, and the enterprise architecture — what is publicly known, what is vendor-verified, what is inferred from industry practice, and what UOB simply does not disclose.*

**Verification convention used throughout: ✅ = verified in this research pass (primary/secondary sources); ⚠ = flagged (inferred, approximate, single-source, or structural inference); unmarked = structural/industry knowledge presented as such. The consolidated [Claims-Status table is in §11](#11-claims-status-and-verification-notes).**

---

## Table of Contents

1. [UOB Overview](#1-uob-overview)
2. [Core Banking Systems](#2-core-banking-systems)
3. [Digital Banking Systems](#3-digital-banking-systems)
4. [Wholesale Systems](#4-wholesale-systems)
5. [Payment Systems](#5-payment-systems)
6. [AI and Innovation](#6-ai-and-innovation)
7. [Security and Fraud Systems](#7-security-and-fraud-systems)
8. [Enterprise Architecture](#8-enterprise-architecture)
9. [Worked Example: A UOB TMRW Customer Journey](#9-worked-example-a-uob-tmrw-customer-journey)
10. [Summary: The Wee Family's Modern Stack](#10-summary-the-wee-familys-modern-stack)
11. [Claims Status and Verification Notes](#11-claims-status-and-verification-notes)
12. [Glossary](#12-glossary)
13. [References and Further Reading](#13-references-and-further-reading)

---

## 1. UOB Overview

### 1.1 The Scope: What This Guide Covers

This guide is the **software-systems deep-dive for UOB** — the third leg of the Singapore-bank trio (DBS, OCBC, UOB), mapped the same way as its siblings: the core estate, the digital layer, the wholesale platforms, the payment rails, the AI programme, the security stack, and the enterprise architecture. The *bank* itself — history, business, strategy — is covered here only at the level needed to anchor the systems map; the deep group-structure and strategy analysis belongs to the sibling [Universal Banking Model Guide](universal_banking_model_guide.md) (the group-structure reference, where UOB is a canonical family-controlled universal bank) and [DBS Bank Guide](dbs_bank_guide.md) (the comparative Singapore-bank analysis). The division of labour:

| Topic | Where it lives |
|---|---|
| The bank, history, the Wee family, leadership, financials | This guide §1 (systems-anchoring summary) + [Universal Banking Model Guide](universal_banking_model_guide.md) |
| **The specific software systems: core, digital, wholesale, payments, AI, security, architecture** | **This guide** (§2–§8) |
| The vendor/platform classes these systems belong to | [Core Banking Systems Guide](core_banking_systems_guide.md), [Payments Hub Guide](payments_hub_guide.md), [Financial Fraud Detection at Scale Guide](financial_fraud_detection_at_scale_guide.md) |
| Singapore's real-time payment infrastructure (FAST/PayNow) | [Financial Infrastructure Guide](financial_infrastructure_guide.md), [ISO 20022 Core Processes Guide](iso_20022_core_processes_guide.md) |
| The Temenos-side cores (contrast — UOB is not Temenos) | [Temenos Guide](temenos_guide.md), [T24 Programming Guide](t24_programming_guide.md), [TAFJ Guide](tafj_guide.md) |

What is covered here, section by section: the **UOB overview** — history, the Wee family, the group, leadership (§1); the **core banking estate** — the Silverlake SIBS relationship and the 2025 acquisition of Silverlake Axis itself (§2); **digital banking** — the UOB Mighty → UOB TMRW app, the TMRW digital bank, and the online/mobile estate (§3); **wholesale systems** — UOB Infinity and the corporate stack (§4); **payment systems** — FAST, PayNow, and the regional rails (§5); **AI and innovation** — the 2023 genAI first, the 2024–26 IT investment programme, and the automation estate (§6); **security and fraud** — Money Lock, Kill Switch, and the anti-scam stack (§7); **enterprise architecture** — the cloud/data posture and the Punggol Digital District move (§8); a **worked customer journey** through the UOB app (§9); a **one-page summary** (§10); the honest **claims-status audit** (§11); a **glossary** (§12); and **references** (§13).

### 1.2 The History: "Founded 1935, Run by the Wee Family"

The founding fact is clean and verified: **UOB was founded on 6 August 1935 as United Chinese Bank by a group of local businessmen led by Datuk Wee Kheng Chiang** ✅ (the founding date and the Wee Kheng Chiang leadership are attested across UOB's own materials and the press; The Business Times's family profile states it directly: *"Wee Cho Yaw's father, Wee Kheng Chiang, founded UOB in 1935"*). The bank was **renamed United Overseas Bank in 1965** ✅ (the 1965 rename is standard, well-documented history, matching Singapore's independence-era rebranding of local Chinese banks — treated as verified via the Wikipedia/directory record ⚠ borderline; the 1935 founding itself is the rock-solid anchor).

The family thread is the single most distinctive fact about UOB as a *systems story*: UOB is the **family-controlled** bank of the Singapore trio — DBS is government-linked (Temasek), OCBC is widely held, and **UOB has been run by three generations of the Wee family** ✅ (Euromoney: *"UOB is run by three generations of the Wee family, with a fourth in the wings"*). The generational chain an architect needs:

- **Wee Kheng Chiang (1886–1965)** ✅ — the founder, a Sarawak-born Hokkien businessman (banker, trader, and philanthropist in Kuching and Singapore); the "Datuk" honorific reflects his Sarawak roots. His bank: United Chinese Bank, 1935.
- **Wee Cho Yaw (1929–2024)** ✅ — the second generation; took the helm and built UOB into the regional group it is today over a six-decade reign; Chairman Emeritus at his death in 2024 at age 95. The Straits Times's 2025 leadership profile and the Business Times succession coverage both anchor this: Wee Cho Yaw was the chairman-emeritus figure of the dynasty.
- **Wee Ee Cheong** ✅ — the third generation; **Deputy Chairman and Chief Executive Officer** (verified against UOB's own board page: *"Wee Ee Cheong. Deputy Chairman and Chief Executive Officer, Non-independent Director"*); the Straits Times calls him *"the third-generation UOB chief"*. He became CEO in 2007 ⚠ (the 2007 succession date is widely reported but not re-verified this pass).

The business-context facts an architect needs before reading the systems map:

- **90 years old in 2025** ✅ — the Straits Times's 2025 profile is explicit: *"The year 2025 marks UOB's 90th anniversary"*. UOB predates Singapore's independence (1965) and even the modern Singapore state — a "founded before independence" homegrown bank, as Wee Ee Cheong put it in the group's 90th-anniversary messaging.
- **One of the "big three" local banks** ✅ — UOB ranks third of the Singapore big three by assets (after DBS and OCBC; see [DBS Bank Guide](dbs_bank_guide.md)); the trio comparison runs through every section of this guide.
- **~S$2.1–2.5 billion of IT investment planned for 2024–2026** ✅ — the verified technology-spend anchor from UOB's Corporate Day 2024 (reported by DBS Private Banking research, August 2024): *"UOB continues to expect S$2.1-2.5bn IT investments from 2024-2026 as it continues to invest heavily in technology and infrastructure. Blockchain, Gen AI and quantum computing continues to be high on UOB's innovation agenda."* This is the budget line behind everything in §2–§8.
- **The Citi acquisition** ✅ — UOB acquired Citigroup's consumer banking businesses in **Indonesia, Malaysia, Thailand, and Vietnam** (announced January 2022, completed in 2022 ⚠ completion dates vary by country), roughly doubling its retail footprint in those markets. The systems consequence is documented: the SS&C Blue Prism case study describes UOB's RPA estate migrating **97.8 million customer data records in four months** from the acquired business — a rare verified datapoint on UOB's integration-automation machinery (§6, §8).

### 1.3 UOB Today: The Group

UOB is a **universal banking group** in the [Universal Banking Model Guide](universal_banking_model_guide.md) sense: a retail/consumer bank, a wholesale bank, a markets/treasury house, an asset manager, and a securities house under one family-controlled holding structure — but *without* a group-owned insurer (unlike OCBC's Great Eastern; UOB's insurance is bancassurance partnerships ⚠ — see the contrast in [Insurance Software Systems Guide](insurance_software_systems_guide.md)). The group map (each entity is a distinct legal entity with its own systems estate — this matters in §2 and §3):

| Entity | Role in the group | Systems relevance |
|---|---|---|
| **United Overseas Bank Limited (UOB, Singapore)** | The flagship: retail, corporate, institutional, and markets banking — the "UOB" brand | The core estate, the UOB TMRW/Mighty app, Personal Internet Banking, Infinity — §2–§5 |
| **UOB Malaysia** | The Malaysian franchise (the largest foreign-owned bank franchise in Malaysia by branches ⚠) | Franchise core and channels; DuitNow participation — §2.2, §3.4 |
| **UOB Thailand (UOB Thai)** | The Thai franchise; since the Citi acquisition one of the largest foreign banks in Thailand ⚠ scale claim | TMRW's home market (2019); PromptPay participation — §2.2, §3.2 |
| **UOB Indonesia** | The Indonesian franchise (post-Citi acquisition, a top-10 foreign-owned bank ⚠) | Franchise core; TMRW Indonesia; BI-FAST — §2.2, §3.4 |
| **UOB Vietnam** | The Vietnamese franchise (post-Citi acquisition, expanded significantly) | Franchise core and channels; the Mighty app is marketed there too — §3.4 |
| **UOB Asset Management (UOBAM)** | The asset-management arm | The fund-administration/investment stack (see [Asset Management Alternatives Guide](asset_management_alternatives_guide.md) for the platform class) |
| **UOB Kay Hian** | The securities/brokerage arm (the largest Singapore brokerage by market share ⚠) | Trading/clearing systems — the retail-brokerage estate |
| **UOB Group (holding)** | The listed parent (SGX: U11) | Group technology, data, and security shared services — §8 |

### 1.4 The Leadership: The Wee Ee Cheong Era

The leadership structure is a family-and-professionals hybrid, and it matters for the systems story because the CEO is the public driver of the digital/AI agenda:

- **Wee Ee Cheong — Deputy Chairman and Chief Executive Officer** ✅ (UOB board page). The Straits Times's "How UOB's Wee Ee Cheong masters the long game" (2025) frames him as the third-generation chief who personally asks AI questions about the bank's future — the genAI-era leadership persona (see §6).
- **Wee Cho Yaw (1929–2024)** ✅ — the late Chairman Emeritus; the family elder whose passing (2024) closed the second-generation era; the Business Times's dynasty coverage (2024–25) examined succession as the family's central question.
- **Professional layer** ✅ — the non-family executives are publicly named and are the *operational* drivers of the systems estate: **Frederick Chin** (Head of Group Wholesale Banking and Markets — the executive behind Infinity, §4) and **Eric Lim** (Chief Sustainability Officer) appear together with Wee Ee Cheong in UOB's own 90th-anniversary/vision content; the group COO/CIO layer is less public ⚠ (UOB does not publish a DBS-style "10,000 technologists" engineering narrative — contrast [DBS Software Systems Guide](dbs_software_systems_guide.md) §1.5).
- **The family-governance angle** ⚠ — the Euromoney "new era" piece (2020s) notes the fourth generation in the wings; the board page lists Wee family members among non-independent directors. For the architect: continuity of family stewardship is a *stability* factor in technology strategy — UOB's tech direction (long-horizon, conservative, buy-don't-build where possible, then buy the vendor outright in §2.3) reads like family-capital strategy.

### 1.5 The Overview Table: Aspect and Description

| Aspect | Description |
|---|---|
| **Legal identity** | United Overseas Bank Limited (SGX: U11), incorporated **6 August 1935** as United Chinese Bank; renamed United Overseas Bank in **1965** ✅ |
| **Founding** | Founded by **Wee Kheng Chiang** and local businessmen in 1935; the **Wee family** has controlled the bank across three generations ✅ |
| **Claim to fame** | One of Singapore's "big three" local banks (third by assets); the region's most family-controlled major bank; TMRW — ASEAN's first mobile-only digital bank (2019) ✅ |
| **Scale** | 90 years old in 2025 ✅; S$2.1–2.5bn IT investment planned 2024–2026 ✅; ~S$60bn+ market cap and a top-tier ASEAN balance sheet ⚠ figures flagged |
| **Leadership** | Deputy Chairman & CEO **Wee Ee Cheong** (third generation) ✅; Chairman Emeritus Wee Cho Yaw (d. 2024) ✅ |
| **Group entities** | UOB Singapore; UOB Malaysia; UOB Thailand; UOB Indonesia; UOB Vietnam; UOB Asset Management; UOB Kay Hian ✅ (brand-level) |
| **Core banking** | **Silverlake SIBS** — and since 2025 UOB **owns** Silverlake Axis, its core vendor (§2 — the defining systems fact) |
| **Digital channels** | **UOB Mighty → UOB TMRW app** (Singapore), Personal Internet Banking, TMRW digital bank (Thailand 2019, Indonesia 2021 ⚠) — §3 |
| **Wholesale** | **UOB Infinity** (launched October 2020) — the corporate digital platform — §4 |
| **Payments** | FAST (2014), PayNow (2017), regional rails (PromptPay, DuitNow, BI-FAST ⚠) — §5 |
| **AI** | First SG bank to trial Microsoft 365 Copilot (Oct 2023, 300 staff) ✅; S$2.1–2.5bn IT programme; Blue Prism RPA at Citi-integration scale — §6 |
| **Security** | Money Lock, Kill Switch, fraud hotline, scam controls — §7 |
| **Cloud/architecture** | Hyperscaler specifics not public ⚠; Microsoft/SS&C verified; Punggol Digital District tech campus (2026) ✅ — §8 |

### 1.6 The Technology Organization

The human system that runs the software estate is, like OCBC's, thinner in the public record than DBS's — but the 2024–26 investment figure and the 2026 Punggol move give it more texture than the sibling guides':

- **Technology headcount** ⚠ — not published as a single number. What is verified: UOB began **relocating its technology and innovation workforce to the Punggol Digital District (PDD)** in 2026, becoming **the first bank to anchor operations in PDD** ✅ (Singapore Business Review, May 2026: *"UOB has begun relocating its technology and innovation workforce to the Punggol Digital District"*) — thousands of staff ⚠ (the "thousands" figure is press-reported, not UOB-confirmed).
- **The IT budget** ✅ — S$2.1–2.5bn for 2024–2026 (Corporate Day 2024, via DBS Private Banking's note) — the single most concrete technology-resourcing datapoint any Singapore bank guide in this series carries.
- **The delivery model** ⚠ — UOB's public record describes an outsourced-plus-in-house hybrid: named external partners (Microsoft for Copilot/genAI §6, SS&C Blue Prism for RPA §6, Silverlake for core software §2 — now in-house!) with an in-house engineering core that is not as publicly profiled as DBS's.
- **The innovation arm** ⚠ — UOB FinLab, the group's fintech accelerator/innovation unit, is widely documented in the fintech press but was not separately re-verified this pass; the *verified* innovation landmarks are TMRW (2019, §3.2), Copilot (2023, §6.1), and the PDD move (2026, §8.3).
- **The strategy posture** — UOB's technology strategy under the Wee family reads as **long-horizon, risk-aware, and consolidation-driven**: buy proven platforms (Silverlake), wrap them with owned digital channels (Mighty/TMRW, Infinity), automate integration at scale (Blue Prism), and — in the 2025 masterstroke — **acquire the core vendor outright** (§2.3) so the family controls its own digital spine.

The architect's read: UOB's technology organization is **the most vertically-integrated of the Singapore trio by 2026** — because it now owns its core-banking software vendor, its RPA estate is documented at the 100-million-record scale, and its tech campus anchors a government digital district. The public *engineering-culture* narrative (DBS-style) is thinner; the *asset-ownership* narrative is unique in the series.

### 1.7 The Systems Map at a Glance

The one-table map of the whole estate (each row is a layer of §2–§8):

| Layer | The systems | Status anchor |
|---|---|---|
| Core | Silverlake SIBS family (ASEAN franchises) — vendor now group-owned | ✅ acquisition verified (§2.3) |
| Channels — consumer | UOB Mighty → UOB TMRW app; Personal Internet Banking; TMRW digital bank (TH/ID) | ✅ product-verified (§3) |
| Channels — business | UOB Infinity; UOB SME | ✅ launch-verified (§4) |
| Payments | FAST/PayNow rails; regional overlays; cards; payments hub | ✅ rails; ⚠ hub (§5) |
| AI/automation | Microsoft 365 Copilot; Blue Prism RPA; insights/analytics | ✅ Copilot + RPA (§6) |
| Security | Money Lock; Kill Switch; fraud hotline; monitoring | ✅ customer controls (§7) |
| Architecture | Microsoft/SS&C/Silverlake partners; ⚠ hyperscaler/data; PDD campus | ✅ partners + PDD; ⚠ cloud (§8) |
| Budget | S$2.1–2.5bn IT investment, 2024–26 | ✅ Corporate Day 2024 (§6.2) |

---

## 2. Core Banking Systems

### 2.1 The Core: Silverlake SIBS — and the 2025 Acquisition of the Vendor

The task brief's core question — *what is the UOB core banking system, and who is the vendor?* — has an answer that is *more* extraordinary than the question anticipates: **UOB's core banking system is Silverlake Axis SIBS (Silverlake Integrated Banking Solution), and in 2025 UOB bought the vendor.** ✅✅

The two-layer evidence:

**Layer 1 — UOB is a Silverlake SIBS customer.** The direct vendor-published case study that exists for OCBC (see [OCBC Software Systems Guide](ocbc_software_systems_guide.md) §2.1) does **not** exist for UOB in this pass ⚠ — no "UOB: A Case Study" surfaced on Silverlake's public pages. The UOB–SIBS relationship rests on three legs:

- **Industry consensus** ✅ (multiple independent sources): the SGX-research and investor-coverage corpus on Silverlake Axis consistently names UOB among its flagship SIBS clients — e.g., the KGI initiation report on Silverlake (July 2020) and investor-blog coverage ("*Prominent banks such as OCBC, UOB and CIMB are using Silverlake's CBS*"). Treat as verified-at-consensus level, not vendor-case-study level.
- **The 2025 acquisition itself** ✅ — a buyer does not pay S$0.36/share to privatise a vendor it does not deeply depend on (see §2.3). The acquisition is the strongest possible *revealed-preference* evidence of the core relationship.
- **SIBS's regional footprint** ✅ (structural) — SIBS on IBM AS/400 is the Southeast-Asian workhorse core (Malaysia-origin vendor, Jack Henry lineage per the KGI report ⚠), and UOB's five ASEAN franchises are exactly the SIBS heartland. The per-franchise core map (Singapore vs Thailand vs Malaysia…) is **not public** ⚠ — the honest statement is *"UOB runs Silverlake-family cores across its ASEAN franchises (consensus); the exact per-entity coverage is not disclosed"*.

**Layer 2 — UOB acquired Silverlake Axis (2025).** ✅ The chain of verified facts:

- **February 2025** — UOB, through its wholly-owned investment vehicle **E2I Ltd**, launched a **voluntary unconditional offer** for all ordinary shares of Silverlake Axis (excluding treasury and already-owned shares) at **S$0.36 per share cash** ✅ (Business Times: "Silverlake Axis receives privatisation offer with S$0.36 per share cash option"; Markets Insider/Business Insider: "United Overseas Bank Limited has launched a voluntary unconditional offer… made on behalf of E2I Ltd").
- **18 November 2025** — the offer closed and **Silverlake Axis was delisted from the Singapore Exchange (SGX)** ✅ (Singapore Business Review: "Silverlake Axis will be delisted from the SGX following the close of E21's voluntary unconditional offer last 18 November" — note the SBR article's "E21" is a typo of E2I ⚠ the vehicle name is E2I per BT and BI).
- **The rationale** ✅ (BT's delisting coverage): privatisation gives the offeror "greater control and management flexibility" — and, per industry commentary ⚠ (LinkedIn/analyst takes, flagged), UOB took full ownership of *"the digital spine running not only its own operations but also many of its regional peers"* — Silverlake's client base spans dozens of ASEAN banks.

What this means, honestly:

- **UOB is now its own core-banking vendor.** The classic make-vs-buy question for a core estate is resolved in the most unusual way in this series: buy the *maker*. By 2026, UOB's core roadmap is decided inside the group, and Silverlake's other bank clients (OCBC among them, per §2.1 of the OCBC guide) run software owned by a competitor bank — a fact every architect in the region should file under "competitive-risk monitoring".
- **The core is still AS/400-era-class SIBS** — the acquisition does not modernise the core overnight; SIBS is the batch-oriented, mainframe-class platform documented in [Core Banking Systems Guide](core_banking_systems_guide.md) and the mechanics guides (§2.4). What UOB bought is *control of the roadmap*, not a cloud-native core.
- **No Temenos, no FLEXCUBE, no BaNCS** — no verified UOB production relationship with the other core families was found ⚠ (contrast [Temenos Guide](temenos_guide.md), [Oracle FLEXCUBE Data Model Guide](oracle_flexcube_data_model_guide.md)). UOB is the *Silverlake* case in the vendor landscape — the same family as OCBC ([OCBC Software Systems Guide](ocbc_software_systems_guide.md) §2.5), which makes the OCBC guide the natural series twin for the core chapter.

**The core estate at a glance:**

| Component | What it is | Status |
|---|---|---|
| **Silverlake SIBS (SIBS)** | The vendor core: deposits, loans, accounts, transactions across the ASEAN franchise estate | ✅ UOB–Silverlake relationship at consensus level; ⚠ per-entity scope not public |
| **Silverlake Axis Ltd** | The core *vendor* — SGX-listed until Nov 2025, then acquired by UOB via E2I and delisted | ✅ verified (BT, BI, SBR) |
| **IBM AS/400 (Power Systems) platform** | The SIBS runtime class | ⚠ structurally expected; not separately verified for UOB |
| **Singapore retail core** | The flagship deposits/loans book | ⚠ identity not directly verified |
| **Regional franchise cores (TH/MY/ID/VN)** | SIBS-class estates serving the overseas books | ⚠ inferred (SIBS footprint + Citi-acquisition integration) |
| **Temenos / FLEXCUBE / BaNCS** | No verified UOB production relationship | ⚠ unverified / not found |

### 2.2 The Core Landscape: What Runs Where

For the architect, the useful model of the UOB core estate is **one vendor family (Silverlake) across the ASEAN group, wrapped by owned channels** — with the 2022 Citi acquisition as the recent integration shock:

- **The SIBS estate** ✅ (consensus-level): the retail deposit/loan/account engines for the group's franchises. SIBS functional coverage (CASA, term deposits, loans, remittances) is documented in [Core Banking Systems Guide](core_banking_systems_guide.md); the accounting mechanics — interest accrual, end-of-day posting — are the domain of the sibling mechanics guides ([Interest Engines Core Banking Guide](interest_engines_core_banking_guide.md), [Posting Engine Core Banking Guide](posting_engine_core_banking_guide.md), [Core Banking Processes Guide](core_banking_processes_guide.md)).
- **The Citi-acquisition integration** ✅ (the documented exception to the "nothing is public" rule): the SS&C Blue Prism case study (April 2025) describes UOB's RPA digital workers migrating **97.8 million customer-data records in four months** from the acquired Citi business — i.e., the 2022 acquisitions (Indonesia, Malaysia, Thailand, Vietnam) were integrated onto UOB systems via an automation-heavy migration pipeline (§6.4, §8.2). The *target* core for the migrated books is not named ⚠; SIBS-family is the reasonable inference given the franchise pattern.
- **The treasury/markets core** ⚠ — the wholesale/markets business (Frederick Chin's Group Wholesale Banking and Markets) runs the markets stack (FX, rates, derivatives, risk); vendor not public in this pass; the platform class is in [Nasdaq Calypso Guide](nasdaq_calypso_guide.md) and the DBS guide's treasury analysis.
- **UOB Kay Hian / UOBAM cores** ⚠ — the brokerage (trading, clearing) and asset-management (fund admin) estates are separate platforms; no vendor facts verified this pass (see [Asset Management Alternatives Guide](asset_management_alternatives_guide.md) for the platform class).
- **No group insurer** ✅ (negative result) — unlike OCBC/Great Eastern, UOB has no group-owned insurance estate; bancassurance is partnership-based ⚠ ([Insurance Software Systems Guide](insurance_software_systems_guide.md) is cross-referenced as the contrast case).

The honest summary of §2.2: **UOB's public core-banking footprint is "Silverlake-family SIBS across the ASEAN franchise estate" (consensus) + "UOB owns the vendor" (verified); every per-entity detail — which franchise runs which release, what the Singapore flagship runs, what the migrated Citi books landed on — is not public.** The OCBC guide's conclusion applies verbatim: *everything else is inference about system classes, not verified facts about systems.*

### 2.3 The Core Table: System, Function, Notes

| System | Function | Notes |
|---|---|---|
| **Silverlake SIBS (SIBS)** | Retail core banking: deposits (CASA), loans, accounts, transactions across the ASEAN group | ✅ UOB–Silverlake at consensus level; ⚠ per-franchise scope not public; runs on IBM AS/400-class estate |
| **Silverlake Axis (the vendor)** | The core software company — SGX-listed until Nov 2025 | ✅ **Acquired by UOB via E2I Ltd, S$0.36/share, offer Feb 2025; delisted 18 Nov 2025** — UOB now owns its core vendor |
| **IBM AS/400 (Power Systems)** | The core platform/runtime class for SIBS | ⚠ structurally expected; legacy-class, batch-oriented |
| **Singapore flagship core** | The flagship deposits/loans book of record | ⚠ Identity not directly verified; SIBS-family expected |
| **Regional franchise cores (TH/MY/ID/VN)** | The overseas retail books (incl. the 2022 Citi-acquired books) | ⚠ SIBS-family inferred; Citi migration documented via Blue Prism case study (97.8M records, 4 months) ✅ |
| **Markets/treasury stack** | FX/rates/derivatives processing and risk | ⚠ Vendor not public; platform class in [Nasdaq Calypso Guide](nasdaq_calypso_guide.md) |
| **UOB Kay Hian / UOBAM platforms** | Brokerage trading/clearing; fund administration | ⚠ Vendors not public |
| **Core integration layer** | APIs, batch feeds, real-time access paths between channels and cores | ⚠ Structural (every channel in §3–§5 needs it); specifics not public |

### 2.4 Core Mechanics and Interfaces

The SIBS-class core's *behaviour* is documented in the mechanics guides; what is UOB-specific here is the strategic posture:

- **The accounting core** ⚠ — deposits accrue interest, loans amortise, the ledger posts at end-of-day batch on the AS/400-class estate; UOB's exact batch windows and posting rules are not public ([Interest Engines Core Banking Guide](interest_engines_core_banking_guide.md), [Posting Engine Core Banking Guide](posting_engine_core_banking_guide.md), [Core Banking Processes Guide](core_banking_processes_guide.md)).
- **Real-time balances in the app** ✅ (product-level) — UOB Mighty/TMRW shows balances and transactions instantly; *how* (direct core enquiry vs ledger cache vs event-fed mirror) is not public. The industry pattern for an AS/400 core is a real-time enquiry layer in front of a batch-posting core (see [Event Stream Processing Guide](../technology/event_stream_processing_guide.md) for the mechanism class).
- **The integration surface** ⚠ — channels reach the core through an integration layer: APIs for real-time enquiry, batch files for EOD feeds, event streams for the data/AI estate (§6, §8). The Blue Prism deployment (97.8M records migrated in 4 months) is the one verified datapoint about the scale and automation level of UOB's integration machinery ✅.
- **The modernisation posture — now ownership** ✅/⚠ — the industry-standard strangler-fig pattern ([Core Banking Systems Guide](core_banking_systems_guide.md)) applies, *plus* the unique 2025 twist: rather than re-platform away from SIBS, UOB bought SIBS. The plausible strategic read (⚠ analyst commentary): UOB intends to modernise *within* the Silverlake estate — upgrading, productising, and possibly licensing it further afield, with the group as anchor tenant. No big-bang core-replacement announcement (contrast SC's Atlas-on-AWS in [Standard Chartered Guide](standard_chartered_guide.md)) has been made.

### 2.5 The Core in Series Context: Silverlake SIBS vs the Vendor Landscape

| Vendor core | Where it appears in this series | UOB relevance |
|---|---|---|
| **Silverlake SIBS** | [OCBC Software Systems Guide](ocbc_software_systems_guide.md) §2 (the other SIBS case); **this guide** | ✅ UOB's core family — and since 2025 its *own* vendor; no dedicated Silverlake guide exists in the repo yet, so this guide plus OCBC's are the Silverlake references |
| **Temenos (T24/Transact)** | [Temenos Guide](temenos_guide.md), [T24 Programming Guide](t24_programming_guide.md), [TAFJ Guide](tafj_guide.md) | ⚠ No verified UOB relationship; cross-referenced for class mechanics only |
| **Oracle FLEXCUBE** | [Oracle FLEXCUBE Data Model Guide](oracle_flexcube_data_model_guide.md) | ⚠ No verified UOB relationship; light cross-ref |
| **Apache Fineract** | [Apache Fineract Guide](apache_fineract_guide.md) | ⚠ No UOB relevance (the open-source digital-bank core class); contrast case |
| **Chinese bank cores** | [Chinese Bank Core Systems Guide](chinese_bank_core_systems_guide.md) | ⚠ Light cross-ref (the Overseas-Chinese banking heritage context) |
| **In-house wrapped legacy** | [DBS Software Systems Guide](dbs_software_systems_guide.md) §2 | The *contrast* case: DBS hides its core identity; UOB's answer to the core question is ownership |

The series-level lesson: **UOB is the proof that the ultimate vendor strategy is to buy the vendor.** Where OCBC *names* its core vendor (Silverlake) and DBS *hides* its core, UOB by 2026 *owns* the core software company — a strategy no other bank in this series has attempted. The modernisation lesson of §2.4 (wrap the core, modernise the edge) still holds; UOB's edge (§3–§8) is where the public innovation is.

---

## 3. Digital Banking Systems

### 3.1 The Consumer Front Door: UOB Mighty, Now UOB TMRW

The consumer digital front door in Singapore is the **UOB Mighty app** — verified from UOB's own store listing (`com.uob.mighty.app`, "UOB Mighty Singapore"): *"Bank, pay and play with the new and improved Mighty. Our all-in-one app makes your mobile banking experience easier, mightier and personal"* ✅. Verified product features from the listing and UOB pages:

- **The Mighty lineage** ✅ — UOB Mighty evolved from the earlier "UOB M" app (the store description answers *"What happened to UOB M?"* — *"UOB M just turned Mighty"*); the app's web launcher (pib.uob.com.sg) describes it as the app that "simplifies banking and dining experiences".
- **Bank** ✅ — consolidated accounts tab, Mighty Insights ("bank smarter and safer with personalised and timely alerts"), PayNow and Scan-to-pay directly from the login screen, bills and transfers "in four simple steps", Touch ID login, **10-minute account opening** (the app's onboarding pitch: *"Open an account via UOB Mighty in 10 minutes"*).
- **Pay** ✅ — PayNow, Scan to pay, card controls; the Rewards+ rewards programme is integrated (the store text: *"Enjoy Singapore's newest and biggest rewards programme on the UOB TMRW app"*).
- **Play** ✅ — the "Save and Harvest" gamified savings game (fruit plushies and travel prizes) is UOB's own gamification layer on the app.
- **The 2024–25 rebrand: Mighty → TMRW** ✅/⚠ — UOB's own Singapore digital-banking page now reads **"UOB TMRW, All-in-1 Banking App"** (uob.com.sg/personal/digital-banking), and UOB's official social posts instruct customers to *"Activate Money Lock on UOB TMRW app"* — i.e., the Singapore app is being unified under the TMRW brand that began as the digital-bank brand in Thailand (§3.2). The exact rename date is ⚠ (2024–25 vintage; not separately verified), but the current brand state is verified from UOB's own surfaces. The Play Store listing still carries the "UOB Mighty" title and `com.uob.mighty.app` package id ⚠ (transitional state).

The product framing: **an all-in-one "bank, pay and play" app** — the same consolidated-flagship pattern as OCBC Digital ([OCBC Software Systems Guide](ocbc_software_systems_guide.md) §3.1) and DBS digibank ([DBS Software Systems Guide](dbs_software_systems_guide.md) §8), differentiated by the gamification layer and the TMRW brand unification.

### 3.2 TMRW: ASEAN's First Mobile-Only Digital Bank (2019)

The TMRW story is the group's most distinctive digital asset and its verified launch record is excellent:

- **Launch** ✅ — UOB's own press release (14 February 2019): *"Thailand's digital generation will be the first consumers in ASEAN to experience TMRW, a digital bank powered by United Overseas Bank (UOB). TMRW is the first mobile-only bank designed for ASEAN millennials"* (uobgroup.com PDF, "The dawn of TMRW"). The Straits Times (14 Feb 2019) adds the ambition: the digital bank aimed to attract **three to five million customers in five years** ✅.
- **The TMRW thesis** ✅ — a **mobile-only** bank (no branches), targeting ASEAN millennials, built from scratch rather than wrapped around the legacy core — the *greenfield* digital experiment that the Singapore trio's other members did not run at the same scale (OCBC's digital-banking story is app-consolidation, not a new bank; DBS's digibank is a wrapped incumbent — contrast §3.5).
- **The rollout** ✅/⚠ — Thailand first (Feb 2019, with UOB Thailand as the operating entity); **Indonesia followed in 2021** ⚠ (widely reported — TMRW Indonesia launched October 2021 — but not re-verified from a primary source this pass); Singapore's UOB TMRW (2024–25 rebrand, §3.1) is the *app-brand* unification, not the standalone digital bank ⚠ (whether the Singapore app inherits the TMRW digital-bank engine or is the wrapped-incumbent app under a new brand is not public).
- **The platform** ⚠ — TMRW's technology stack (cloud-native? which vendor core? what AI?) is not public. The reasonable structural inference: a cloud-hosted, API-first, data-driven stack with heavy AI personalisation (the bank's own marketing emphasises "behavioural insights"), but **no verified vendor facts** — flag as unverified rather than guess.

### 3.3 The Digital Systems: Online and Mobile Estate

The UOB digital estate is the standard **two-surface (mobile + web), multi-audience** architecture, with the TMRW brand unifying the consumer surfaces:

- **UOB Mighty / UOB TMRW (mobile app)** ✅ — the consumer mobile front door (§3.1).
- **UOB Personal Internet Banking (PIB)** ✅ — the web channel; attested in UOB's own materials (e.g., the Vietnam Mighty FAQ: *"Sign in using the same username and password that you use for UOB Personal Internet Banking"*; the Kill Switch description blocks "Personal Internet Banking" and the app together). The mature online-banking surface that predates the app.
- **UOB Infinity (business)** ✅ — the corporate/wholesale digital platform (§4).
- **UOB SME** ✅ — the SME segment surfaces (UOB's business pages list "UOB SME" alongside Infinity; the SME digital banking is part of the business estate — ⚠ whether SME runs on Infinity or a separate SME portal is not public).
- **TMRW digital bank** ✅/⚠ — the standalone mobile-only bank (Thailand 2019; Indonesia 2021 ⚠) — §3.2.
- **Rewards+/Savings gamification** ✅ — the app-embedded engagement layer (Rewards+, Save and Harvest).

What is *not* public ⚠: the channel middleware and API layer between the apps and the cores, the app's engineering stack (native vs cross-platform), and the TMRW backend — all structurally inferred: **channel front ends → a channel-integration layer → the core services of §2** (structural ⚠).

### 3.4 The Digital Table

| System | Function | Notes |
|---|---|---|
| **UOB Mighty → UOB TMRW (app)** | Consumer mobile banking: balances, transfers, PayNow, Scan-to-pay, rewards, gamified savings, 10-minute onboarding | ✅ store-verified (`com.uob.mighty.app`); now branded UOB TMRW in Singapore ✅ (uob.com.sg digital-banking page) |
| **UOB Personal Internet Banking** | Consumer online banking (web) | ✅ attested in UOB materials (PIB logins, Kill Switch scope) |
| **TMRW (digital bank)** | ASEAN's first mobile-only digital bank — Thailand (Feb 2019), Indonesia (2021 ⚠) | ✅ launch verified (UOB release 14 Feb 2019; ST); platform stack ⚠ not public |
| **UOB Infinity** | Corporate/wholesale digital banking | ✅ launched 1 Oct 2020 (BT/ST); §4 |
| **UOB SME** | SME digital banking surfaces | ⚠ product verified at brand level; platform relationship to Infinity not public |
| **Rewards+ / Save-and-Harvest** | Engagement: rewards programme, gamified savings | ✅ app-store verified |
| **Channel-integration layer** | Middleware/APIs between channels and cores | ⚠ structural inference; specifics not public |
| **Digital onboarding/eKYC** | New-customer journeys (Singpass-class identity rails; 10-minute app onboarding) | ✅ 10-minute onboarding claim verified; Singpass integration standard for SG banks ⚠ |

### 3.5 Digital Banking in Market Context

Where the UOB digital estate sits among the Singapore channel landscape (the comparison set from [DBS Software Systems Guide](dbs_software_systems_guide.md) §8 and [Trust Bank Guide](trust_bank_guide.md)):

| Channel | Owner | Core underneath | Notes |
|---|---|---|---|
| **UOB TMRW / Mighty** | UOB | Silverlake SIBS-class (⚠) | The consolidated flagship; gamified; now TMRW-branded; Kill Switch/Money Lock baked in |
| **digibank** | DBS | Wrapped legacy core (⚠) | The journey-led incumbent ([DBS Software Systems Guide](dbs_software_systems_guide.md) §8) |
| **OCBC Digital** | OCBC | Silverlake SIBS-class (⚠ SG identity) | The security-first incumbent ([OCBC Software Systems Guide](ocbc_software_systems_guide.md) §3) |
| **TMRW (Thailand/Indonesia)** | UOB | Greenfield stack (⚠) | ASEAN's first mobile-only digital bank — the region's digital-first experiment |
| **Trust** | Trust Bank | Mambu on GCP (✅ per [Trust Bank Guide](trust_bank_guide.md)) | The licensed digital bank — cloud-native core, the contrast case |
| **GXS / MariBank** | Grab/Singtel; Sea | Vendor/cloud-native cores (⚠) | The other licensed digital banks |

UOB's positioning is the **entertainment-and-ownership incumbent**: the same wrapped-legacy architecture as its peers, differentiated by (a) the gamified consumer app, (b) the region's first mobile-only digital bank, and (c) owning the core vendor underneath it all. No licensed digital bank in Singapore (UOB did not bid for a digital-bank licence ⚠ — the licensed SG digital banks are Trust, GXS, MariBank), so TMRW's standalone-bank experiment lives in Thailand and Indonesia while Singapore gets the TMRW *brand* on the wrapped estate ⚠.

### 3.6 The Regional Digital Franchises

The Mighty/TMRW story is the Singapore story; the group's overseas franchises run their own channel estates on the group's core family (§2.2), and the 2022 Citi acquisition reshaped them:

- **UOB Thailand** ✅ (brand-level) — the Thai franchise (uob.co.th) was TMRW's launch market in February 2019 (✅ §3.2) and the anchor of the Citi-acquisition integration in Thailand; the Thai digital estate (TMRW + the franchise's own banking app) and its PromptPay participation are structurally certain, per-franchise system names ⚠ not verified this pass.
- **UOB Indonesia** ⚠ — TMRW Indonesia (2021 ⚠) plus the franchise's conventional mobile/internet banking; BI-FAST participation structural ⚠.
- **UOB Malaysia** ⚠ — the Malaysian franchise (the largest foreign-owned bank franchise by branches, ⚠ scale claim); its digital estate and DuitNow participation are structural, names not verified.
- **UOB Vietnam** ✅ (brand-level) — the Vietnamese franchise markets **UOB Mighty** itself (the uob.com.vn Mighty pages carry the same 10-minute-onboarding and PIB-login copy as Singapore) — the one verified instance of the Mighty app being offered beyond Singapore; post-Citi-acquisition scale-up ⚠.
- **The group pattern** ⚠ — the Singapore playbook (consolidate onto a flagship app, TMRW-brand, gamify, bake in Money Lock/Kill Switch) is the stated direction; per-franchise rollout status is not public.

---

## 4. Wholesale Systems

### 4.1 UOB Infinity: The Verified Wholesale Digital Platform

The wholesale digital front door is **UOB Infinity** — one of the best-verified systems in this guide:

- **Launch** ✅ — The Business Times, 1 October 2020: *"UOB on Wednesday said it has launched digital solution UOB Infinity for its corporate clients. It will roll out UOB Infinity across the region, beginning with Singapore."* The Straits Times carries the same launch. The platform lets corporate clients "manage their banking needs in a simpler, smarter and more personalised manner".
- **Function** ✅ — UOB's own FAQ: *"UOB Infinity is UOB's digital banking platform for businesses. The platform offers a comprehensive suite of cash management and trade services to help you manage your cash and trade transactions more efficiently."* UOB @ SFF 2024: *"a secure, customisable digital platform for businesses to manage accounts and transactions anytime."*
- **The regional rollout** ✅/⚠ — launched in Singapore, then rolled out regionally (the 2020 release's stated plan); the current per-market coverage (Malaysia/Thailand/Indonesia/Vietnam live dates) is ⚠ not verified this pass.
- **The executive sponsor** ✅ — the wholesale franchise is led by **Frederick Chin, Head of Group Wholesale Banking and Markets** (UOB's own 90th-anniversary content) — the business owner of the Infinity roadmap.
- **The platform class** ⚠ — Infinity's underlying technology (in-house build vs vendor components; API layer; cloud hosting) is not public. The product description (cash management + trade, customisable dashboards, API-style integration for corporates) puts it in the modern corporate-banking-platform class documented in the [Payments Hub Guide](payments_hub_guide.md) and transaction-banking coverage; specifics flagged.

### 4.2 The Wholesale Landscape: Corporate, SME, and Transaction Banking

The wholesale estate is a **three-surface architecture** — the flagship platform, the SME surface, and the underlying transaction-banking/markets stacks:

- **UOB Infinity** ✅ — the flagship corporate digital platform (cash management, trade, accounts, customisable workflows) — §4.1.
- **UOB SME** ✅ — the SME surface (the business-banking pages market SME digital banking; relationship to Infinity ⚠).
- **Group Wholesale Banking and Markets** ✅ — the business line (Frederick Chin) spanning corporate lending, transaction banking, trade, and markets; the *systems* behind it (trade finance platforms, supply-chain finance, the markets stack) are ⚠ vendor-not-public (see [Supply Chain Finance Technologies Guide](supply_chain_finance_technologies_guide.md) for the platform class).
- **The transaction-banking rails** ⚠ — Infinity sits on the same payment rails as the retail estate (§5): FAST/PayNow for domestic, SWIFT/telex for cross-border, ISO 20022 flows ([ISO 20022 Core Processes Guide](iso_20022_core_processes_guide.md)); UOB's corporate-payments specifics (host-to-host channels, file-based batch payments) are not public.
- **The markets/treasury stack** ⚠ — the markets business (FX, rates, structuring) runs the treasury systems; vendor not public ([Nasdaq Calypso Guide](nasdaq_calypso_guide.md) for the platform class).

### 4.3 The Wholesale Table

| System | Function | Notes |
|---|---|---|
| **UOB Infinity** | Corporate digital banking: cash management, trade services, accounts, customisable workflows | ✅ launched 1 Oct 2020 (BT/ST); regional rollout from Singapore; SFF 2024 showcase |
| **UOB SME** | SME digital banking surface | ✅ brand-level verified; platform relationship ⚠ |
| **Transaction-banking rails** | Payments, trade finance, supply-chain finance under the wholesale estate | ⚠ classes documented ([Supply Chain Finance Technologies Guide](supply_chain_finance_technologies_guide.md), [Payments Hub Guide](payments_hub_guide.md)); UOB specifics not public |
| **Markets/treasury stack** | FX/rates/derivatives processing and risk | ⚠ vendor not public ([Nasdaq Calypso Guide](nasdaq_calypso_guide.md)) |
| **Corporate onboarding/API layer** | KYC, host-to-host, API integration for corporates | ⚠ structural; specifics not public |

---

## 5. Payment Systems

### 5.1 The Payment Rails: FAST, PayNow, and the Regional Overlays

UOB is a participant in Singapore's real-time payment infrastructure and the regional overlays — the rails are shared industry infrastructure, so UOB's *participation* is verified at the structural level and the *customer-facing* payment features are verified in the app itself:

- **FAST (Fast And Secure Transfers)** ✅ — Singapore's real-time interbank clearing rail since **2014** (see [Financial Infrastructure Guide](financial_infrastructure_guide.md)); every Singapore retail bank participates, and UOB's app surfaces FAST-based transfers ("cash transfer to other banks" in the Mighty FAQ) ✅.
- **PayNow** ✅ — the proxy-based instant-payment overlay on FAST (launched **2017** by the Association of Banks in Singapore); UOB is a participating bank ✅ (UOB's own app text: *"Pay using PayNow and Scan to pay directly from the login screen"*), and the UOB TMRW app is where UOB markets PayNow to consumers. UOB's founding-participant status ⚠ (widely reported, not re-verified this pass).
- **Scan to pay** ✅ — the SGQR-class QR payments surface in the UOB app (store-verified).
- **The regional overlays** ⚠ — PromptPay (Thailand), DuitNow (Malaysia), BI-FAST (Indonesia), and the ASEAN cross-border linkages (PayNow–PromptPay–DuitNow links) are the rails UOB's regional franchises participate in; UOB-specific participation is structurally certain but not separately verified this pass.
- **Cards** ✅ — the UOB card estate (credit/debit, the UOB card controls in the app) is brand-verified; the card platform vendor (issuing/acquiring processors) ⚠ not public.
- **Cross-border/inward** ⚠ — SWIFT and the ISO 20022 migration context ([ISO 20022 Core Processes Guide](iso_20022_core_processes_guide.md)); UOB's correspondent-banking specifics not public.

### 5.2 The Payments Landscape

The architect's model: UOB's payments estate is the **standard three-layer** structure — shared rails (FAST/PayNow/regional overlays) at the bottom, a payments/clearing hub layer (⚠ vendor not public — see [Payments Hub Guide](payments_hub_guide.md) for the platform class), and the channel surfaces (Mighty/TMRW, PIB, Infinity) at the top. What is verified: the *surfaces* (app PayNow/Scan-to-pay, Infinity's cash management) ✅; what is not: the *hub* (orchestration, ISO 20022 mapping, fraud screening placement) ⚠ — the same honesty boundary as the OCBC guide draws at its §5.

### 5.3 The Payments Table

| System / Rail | Function | Notes |
|---|---|---|
| **FAST** | Singapore's real-time interbank clearing rail (2014) | ✅ shared infrastructure; UOB participates (app transfers) |
| **PayNow** | Proxy-based instant payments on FAST (2017) | ✅ UOB participation + app surfaces verified; founding-participant ⚠ |
| **Scan to pay / SGQR** | QR payments in the UOB app | ✅ store-verified |
| **PromptPay / DuitNow / BI-FAST** | Regional instant rails (TH/MY/ID) | ⚠ UOB franchise participation structural; not separately verified |
| **Cards estate** | UOB credit/debit cards, in-app controls | ✅ brand + app verified; processor vendor ⚠ |
| **Payments hub / clearing layer** | Orchestration, ISO 20022, settlement between rails and cores | ⚠ vendor not public ([Payments Hub Guide](payments_hub_guide.md)) |
| **Cross-border (SWIFT etc.)** | Correspondent/inward/outward flows | ⚠ specifics not public ([ISO 20022 Core Processes Guide](iso_20022_core_processes_guide.md)) |

---

## 6. AI and Innovation

### 6.1 The 2023 GenAI First: Microsoft 365 Copilot

The task brief's question — *verify the 2023 genAI-flagged UOB AI initiatives* — has a clean, primary-source answer: **UOB was the first Singapore bank to trial Microsoft 365 Copilot, announced 3 October 2023** ✅. UOB's own press release (uobgroup.com, "UOB pioneers trial of Microsoft 365 Copilot Generative AI tool across multiple business functions"):

- *"UOB is the first Singapore bank to trial Microsoft 365 Copilot, a Generative Artificial Intelligence-powered productivity tool."*
- *"As part of the Microsoft 365 Copilot Early Access Program (EAP), UOB will begin use of Microsoft 365 Copilot in October 2023. 300 employees across multiple frontline and back-end business functions in the Bank will progressively gain access… Staff in functions such as branches, customer service and technology and operations will be able to use Microsoft…"* ✅

The significance for the series: OCBC's 2023 first was the *employee genAI chatbot* ([OCBC Software Systems Guide](ocbc_software_systems_guide.md) §6); UOB's 2023 first was the *enterprise productivity copilot* (Microsoft 365 Copilot EAP). Two different genAI firsts from the two banks in the same year — and UOB's is verified directly from UOB's own release. The Microsoft relationship is a verified technology anchor for both §6 and §8 (the only named hyperscaler-adjacent partner in UOB's public record this pass).

### 6.2 The AI Agenda: Corporate Day 2024

The strategy-level anchor is the **UOB Corporate Day 2024** (August 2024), reported by DBS Private Banking's research note ✅:

- **S$2.1–2.5 billion IT investment planned for 2024–2026** ✅ — the budget line for the whole §2–§8 estate.
- **"Blockchain, Gen AI and quantum computing continues to be high on UOB's innovation agenda"** ✅ — the verified innovation tri-fecta. The note adds the honest nuance: *"While Gen AI is able to demonstrate early value, quantum computing…"* (the sentence trails off in the source — quantum is framed as the longer-horizon bet ⚠).
- The GenAI adoption posture ✅/⚠ — from the Copilot pilot (300 staff, 2023) toward broader internal adoption; the specific 2024–25 genAI use-cases beyond Copilot (customer-service assistants, document processing, code assistance) are ⚠ not individually verified this pass — flag rather than invent.

### 6.3 The Innovation Record: Verified Landmarks

The verified innovation landmarks, in order:

- **2016 ⚠ — UOB FinLab**: the group's fintech accelerator/innovation unit, widely documented in fintech press; not separately re-verified this pass (flagged, not asserted).
- **2019 ✅ — TMRW**: ASEAN's first mobile-only digital bank (§3.2) — the flagship innovation product, verified from UOB's own launch release.
- **2020 ✅ — UOB Infinity**: the wholesale digital platform (§4.1) — the corporate innovation flagship.
- **2023 ✅ — Microsoft 365 Copilot**: the genAI first (§6.1).
- **2024 ✅ — The IT-investment programme**: S$2.1–2.5bn, 2024–26, with GenAI/blockchain/quantum on the agenda (§6.2).
- **2024–25 ✅ — UOB @ SFF (Singapore FinTech Festival) tech-ecosystem presence**: UOB's techecosystem pages showcase Infinity and the group's fintech partnerships (uobgroup.com/techecosystem) — brand-level verified.
- **2026 ✅ — Punggol Digital District**: UOB became the first bank to move its technology and innovation workforce into PDD (§8.3) — the physical-innovation anchor.

### 6.4 The Automation Estate: Blue Prism at Citi-Integration Scale

The most concrete AI-adjacent *automation* datapoint in the public record is the **SS&C Blue Prism case study** (April 2025) ✅:

- UOB used **Blue Prism RPA digital workers** to migrate **97.8 million customer data records in four months** from the acquired Citi consumer business (the 2022 acquisitions in Indonesia, Malaysia, Thailand, Vietnam) ✅.
- The case study's framing — *"Digital workers did the heavy lifting to help UOB migrate a massive amount of customer data from their newly acquired business and ensure a seamless transition for new customers"* — verifies (a) UOB runs an enterprise RPA estate (Blue Prism), (b) the Citi migration was automation-orchestrated, and (c) the integration programme of §2.2 was partly robotic. The current scale of the RPA estate and its relationship to the genAI programme ⚠ not public.

### 6.5 The AI Table: Initiative, Description, Notes

| Initiative | Description | Notes |
|---|---|---|
| **Microsoft 365 Copilot (2023)** | GenAI productivity copilot; first SG bank in the EAP | ✅ UOB release, 3 Oct 2023; 300 staff, frontline + back-end functions |
| **IT investment programme (2024–26)** | S$2.1–2.5bn technology spend; GenAI, blockchain, quantum on the agenda | ✅ UOB Corporate Day 2024 (via DBS Private Banking note) |
| **TMRW digital bank (2019)** | ASEAN's first mobile-only bank; AI-personalised digital banking | ✅ launch verified; engine/AI specifics ⚠ |
| **UOB Infinity (2020)** | Corporate digital platform with data-driven dashboards | ✅ launch verified; AI/analytics specifics ⚠ |
| **Blue Prism RPA estate** | Robotic process automation; 97.8M records migrated in 4 months (Citi integration) | ✅ SS&C Blue Prism case study (Apr 2025) |
| **UOB FinLab** | Fintech accelerator/innovation unit | ⚠ widely documented; not re-verified this pass |
| **Quantum computing exploration** | Longer-horizon innovation bet | ⚠ named on the Corporate Day agenda; no specifics public |
| **UOB @ SFF tech ecosystem** | FinTech Festival presence, partnerships showcase | ✅ brand-level (uobgroup.com/techecosystem) |

---

## 7. Security and Fraud Systems

### 7.1 The Anti-Scam Stack: Verified Controls

The task brief's question — *verify the anti-scam* — is answered by three verified customer-facing controls (plus the structural layer):

- **Money Lock** ✅ — UOB's own official channel (LinkedIn, 2025): *"Activate Money Lock on UOB TMRW app to protect your account balances by restricting unauthorised withdrawal from being made on your account."* Money Lock is the Singapore-industry-standard balance-locking control (all major SG banks offer it post-2023, per the industry consensus documented in [Financial Fraud Detection at Scale Guide](financial_fraud_detection_at_scale_guide.md)); UOB's implementation lives in the UOB TMRW/Mighty app.
- **Kill Switch** ✅ — UOB's own app documentation (the Mighty store description): *"This will disable your digital access to Personal Internet Banking, UOB TMRW app and block all your UOB Debit/Credit cards instantly. Note that Kill Switch does not suspend these services."* — the instant self-quarantine control (disable digital access + cards in one action).
- **Fraud hotline** ✅ — UOB's official channel: *"Call our Fraud Hotline 62550160 if unsure"* — the 24/7 human defence layer.
- **The structural layer** ⚠ — real-time fraud monitoring/ML transaction scoring, scam-call filtering, device-risk checks, mule-account controls: sector-standard post-2021 controls (see [Financial Fraud Detection at Scale Guide](financial_fraud_detection_at_scale_guide.md)); UOB-specific product names and model metrics ⚠ not public.
- **The industry context** ✅ — UOB participates in Singapore's industry-wide anti-scam architecture (the Shared Responsibility Framework discussions, MAS directives, the industry Money Lock rollout); UOB's *individual* posture is the verified controls above plus the app-integrated alerts (Mighty Insights: "bank smarter and safer with personalised and timely alerts" ✅).

### 7.2 The Security Landscape

The architect's model of UOB's security estate is the standard **defence-in-depth** stack of the Singapore incumbents: customer-side controls (Money Lock, Kill Switch, hotline) ✅; channel-side controls (device risk, session monitoring, token-based authentication ⚠ — UOB's two-factor token posture is structural); transaction-side controls (real-time fraud scoring ⚠); and the data/AI side (the fraud-detection models of §6's estate ⚠). The *documented* differentiators are the customer-facing controls, which UOB markets aggressively — the same security-first positioning as OCBC's Kill Switch/Money Lock ([OCBC Software Systems Guide](ocbc_software_systems_guide.md) §7), which is no accident: the whole Singapore industry converged on the same control vocabulary after the 2021–22 scam wave.

### 7.3 The Security Table

| System / Control | Function | Notes |
|---|---|---|
| **Money Lock** | Lock account balances against unauthorised withdrawals | ✅ UOB official channel (activate on UOB TMRW app) |
| **Kill Switch** | Disable digital access (PIB + app) and block cards instantly | ✅ UOB app documentation |
| **Fraud hotline** | 24/7 human scam/fraud defence | ✅ UOB official channel (6255 0160) |
| **Mighty Insights alerts** | Personalised security/transaction alerts in-app | ✅ app-store verified |
| **Real-time fraud monitoring/ML scoring** | Transaction-side scam/fraud detection | ⚠ structural ([Financial Fraud Detection at Scale Guide](financial_fraud_detection_at_scale_guide.md)); UOB specifics not public |
| **Device-risk / session controls** | Risky-device and session monitoring | ⚠ sector-standard; UOB specifics not public |
| **Authentication/token layer** | Two-factor authentication for channels | ⚠ structural; specifics not public |

---

## 8. Enterprise Architecture

### 8.1 The Cloud Strategy: What Is Verified and What Is Not

The task brief's question — *verify the cloud strategy* — returns a genuinely mixed result, and the honest answer is a flag, not a fact:

- **⚠ The hyperscaler strategy is NOT verified in this pass.** Two targeted searches for UOB–Google Cloud and UOB–AWS partnerships returned no UOB-specific agreements (the Google Cloud query surfaced Deutsche Bank material; nothing UOB-specific surfaced). The safe statement: *no named UOB–hyperscaler strategic agreement was verified* — UOB's public record is quieter about hyperscaler partnerships than its peers' (contrast [Standard Chartered Guide](standard_chartered_guide.md)'s Atlas-on-AWS and [Trust Bank Guide](trust_bank_guide.md)'s Mambu-on-GCP).
- **✅ The Microsoft relationship is verified** — the 365 Copilot EAP (2023) makes Microsoft the *documented* enterprise-technology partner; whether that extends to Azure workloads is ⚠ not public.
- **✅ The automation layer is verified** — SS&C Blue Prism (RPA) is a documented estate component (§6.4).
- **✅ The core-software layer is verified** — Silverlake (now owned) is the documented application-software spine (§2).
- **⚠ The data platform is not verified** — no UOB-equivalent of OCBC's Cloudera case study ([OCBC Software Systems Guide](ocbc_software_systems_guide.md) §8.1) surfaced this pass; UOB's data/analytics platform (warehouse, lake, governance) is ⚠ not public (the platform classes are in [Data Governance Guide](../technology/data_governance_guide.md)).

The honest architecture read: **UOB's public enterprise-architecture record is "platform partners documented where the deal is named (Microsoft, SS&C, Silverlake), silent where it isn't (hyperscalers, data platform)"** — with the S$2.1–2.5bn IT budget (2024–26) as the spend envelope that any architecture claim must fit inside.

### 8.2 The Integration Fabric: The Citi-Migration Datapoint

The one verified integration-scale datapoint is the Blue Prism case study ✅: **97.8 million customer records migrated in four months** across four acquired country franchises. For the architect this implies: (a) a centralised migration/integration factory (data transformation, ETL, reconciliation) with RPA as the executor; (b) country-by-country cutover onto UOB systems; and (c) an integration fabric (APIs, batch, event streams) substantial enough to absorb a doubling of retail books in four markets. The fabric's *identity* (middleware, ESB/event bus, API gateway vendors) ⚠ is not public — the mechanism classes are in [Event Stream Processing Guide](../technology/event_stream_processing_guide.md) and [Payments Hub Guide](payments_hub_guide.md).

### 8.3 The Physical Architecture: Punggol Digital District (2026)

The 2026 datapoint closes the architecture picture with the *physical* layer ✅:

- **UOB became the first bank to move into the Punggol Digital District (PDD)** — Singapore's first smart and sustainable business district — relocating **its technology and innovation workforce** there ✅ (Singapore Business Review, May 2026; Yahoo Finance; French Chamber of Commerce in Singapore all carry the announcement).
- The strategic read ✅/⚠ — PDD is Singapore's technology-district bet (co-location with the broader tech ecosystem, expected to create up to 28,000 jobs ⚠ press figure); UOB anchoring it signals a deliberate tech-campus strategy — the physical counterpart of the §1.6 technology-organization story. "Thousands" of staff ⚠ press-reported.

### 8.4 The Architecture Table

| Layer | What UOB runs | Status |
|---|---|---|
| **Core application layer** | Silverlake SIBS-family cores across the ASEAN franchises | ✅ consensus-level; vendor now group-owned (§2) |
| **Channel layer** | UOB TMRW/Mighty app, PIB, Infinity, TMRW digital bank | ✅ verified surfaces (§3–§4) |
| **Integration fabric** | APIs, batch feeds, event streams; RPA at 97.8M-record scale | ✅ Blue Prism datapoint; fabric identity ⚠ |
| **Payments layer** | FAST/PayNow participation; regional rails; payments hub | ✅ rails; hub vendor ⚠ (§5) |
| **AI/automation layer** | Microsoft 365 Copilot; RPA estate; GenAI programme | ✅ Copilot + Blue Prism verified; use-case specifics ⚠ (§6) |
| **Security layer** | Money Lock, Kill Switch, fraud hotline, monitoring | ✅ customer controls; ML specifics ⚠ (§7) |
| **Cloud/compute layer** | Hyperscaler strategy | ⚠ **not verified this pass**; Microsoft the only named enterprise partner |
| **Data platform** | Analytics/warehouse/lake | ⚠ not public |
| **Physical layer** | Punggol Digital District tech campus (first bank in PDD, 2026) | ✅ verified (§8.3) |
| **Spend envelope** | S$2.1–2.5bn IT investment, 2024–26 | ✅ Corporate Day 2024 (§6.2) |

### 8.5 Architecture Lessons for the Architect

Five takeaways a solution architect should carry from the UOB estate:

1. **Ownership is the ultimate control plane.** The 2025 Silverlake acquisition (§2.3) means UOB's core-roadmap decisions (release cycles, modernisation, possibly licensing SIBS to other banks) are now internal strategy — and every other ASEAN bank running SIBS has a competitor as its core vendor. Any architecture involving Silverlake software now carries a UOB-control dimension. ⚠ (analyst commentary) / ✅ (the acquisition facts).
2. **The wrapped-legacy pattern is the industry default — and it works.** SIBS batch core + real-time edge layers (⚠ §2.4) is the same strangler-fig topology as DBS and OCBC ([DBS Software Systems Guide](dbs_software_systems_guide.md) §2, [OCBC Software Systems Guide](ocbc_software_systems_guide.md) §2.3). UOB's lesson: differentiation happens in channels, security UX, and AI — not in the core.
3. **Automation is the integration strategy.** The Blue Prism datapoint (97.8M records / 4 months, ✅ §6.4) shows UOB treats RPA as a first-class integration mechanism — an architect integrating with UOB (or its acquired books) should expect robot-mediated data flows, not just APIs and files.
4. **The honest gap: cloud and data are not public.** No hyperscaler agreement or data-platform case study surfaced (⚠ §8.1) — the S$2.1–2.5bn budget says the spend exists, but UOB does not market it the way OCBC markets Cloudera. Design against the verified partners (Microsoft, SS&C, Silverlake) and flag the rest.
5. **Physical infrastructure signals strategic direction.** PDD (✅ §8.3) is not a data-centre story — it is a talent-and-ecosystem story: UOB is co-locating its technology workforce with Singapore's digital-district bet, which is the strongest public signal of where UOB's engineering future is being built.


---

## 9. Worked Example: A UOB TMRW Customer Journey

### 9.1 The Scenario: A UOB Mighty/TMRW User's Day

To make the systems map concrete, here is a **verified-feature customer journey** — every step is anchored to a feature documented in §3–§7 (the ✅-verified surface features), with the underlying systems (the ⚠ layers) named as the systems they must be. The scenario: *Priya, a Singapore retail customer, banks entirely on her phone through the UOB TMRW app (the former UOB Mighty).*

### 9.2 The Flow: Step by Step

| Step | What happens on screen | The system behind it (status) |
|---|---|---|
| **1. Login** | Priya opens the UOB TMRW app and unlocks with **Touch ID** | The app's biometric login (✅ store-verified); the token/authentication layer behind it (⚠ §7.3) |
| **2. Balance check** | The **consolidated accounts tab** shows all her accounts at a glance; **Mighty Insights** surfaces a personalised alert (e.g., a large upcoming card payment) | The channel layer calling the core's real-time enquiry path (⚠ §2.4 — real-time mirror/ledger-cache in front of the SIBS batch core); the insights engine on the data estate (⚠ §6.5) |
| **3. PayNow transfer** | She pays her friend S$50 via **PayNow** (mobile-number proxy), directly from the login screen | The app's PayNow surface (✅ store-verified) → UOB's payments hub (⚠ §5.3) → the **FAST/PayNow shared rails** (✅ §5.1) → the recipient's bank; fraud scoring runs in the path (⚠ §7.3); the debit posts to the core at EOD batch (⚠ §2.4) |
| **4. Scan to pay** | At a hawker stall she pays via **Scan to pay** (SGQR) | The QR payment surface (✅ store-verified) → the same payments stack (⚠) |
| **5. Card control** | She receives a fraud alert about a suspicious card transaction; she uses the in-app card controls to block the card | The cards estate (✅ §5.3) + real-time monitoring (⚠ §7.3); the block propagates to the card processor (⚠) |
| **6. Security hardening** | After reading UOB's scam messaging, she activates **Money Lock** on her balances and familiarises herself with **Kill Switch** | Money Lock (✅ §7.1 — UOB official channel); Kill Switch (✅ §7.1 — disables digital access + cards instantly) |
| **7. Gamified saving** | She plays the **Save and Harvest** game and earns rewards via **Rewards+** | The engagement layer (✅ store-verified); reward points engine (⚠) |
| **8. Account opening** | Her husband opens a new UOB account in **10 minutes** via the app | The digital onboarding flow (✅ store-verified: "Open an account via UOB Mighty in 10 minutes"); eKYC/Singpass rails (⚠ §3.4) → core account creation (⚠ §2) |

### 9.3 The Journey's Systems Map

The journey touches every layer of the guide in one session:

- **Channel layer** (§3): the TMRW/Mighty app — biometrics, consolidated accounts, PayNow/Scan-to-pay, insights, gamification.
- **Core layer** (§2): the SIBS-family core underneath — balance enquiry and posting, account opening.
- **Payments layer** (§5): FAST/PayNow rails, the payments hub, the cards estate.
- **Security layer** (§7): Money Lock, Kill Switch, fraud alerts, the monitoring path every transaction passes.
- **AI/data layer** (§6, §8): Mighty Insights personalisation, fraud scoring, the data estate feeding both.

### 9.4 The Lessons

1. **The customer never sees the core — and that is the architecture.** Priya's entire journey runs on verified channel features; every core interaction (balances, posting, account opening) is behind the API/integration layer of §2.4. The wrapped-legacy topology is the whole game: SIBS stays batch-oriented; the real-time experience is manufactured in the edge (⚠ §2.4).
2. **The security layer is a customer-facing product, not just plumbing.** Money Lock and Kill Switch are *marketed controls* (✅) — UOB sells safety as a feature, exactly like the OCBC security-first positioning (§7.2). An architect designing for UOB must treat scam-defence UX as a first-class channel surface.
3. **Payments are shared infrastructure; differentiation is in the surfaces.** FAST/PayNow are the same rails every Singapore bank uses (✅ §5.1); UOB differentiates through the app's payment UX (login-screen PayNow, Scan-to-pay) and the wholesale platform (Infinity, §4).
4. **Ownership changes the roadmap calculus.** The bank that runs Priya's account runs the software *under* it (Silverlake, §2.3). A UOB architect's core-roadmap questions are answered inside the group — the strangler-fig pattern of §2.4 is now a *make* decision, not a *buy* decision.
5. **Every step rests on ⚠ layers.** The journey's verified features sit on unverified middleware, hub, and model layers. The honest engineering takeaway: UOB's public record proves the *front* (features) and the *spine* (core vendor, RPA, Copilot), while the *connective tissue* stays private — plan integration work against classes, not named UOB products (⚠ §8.4).

---

## 10. Summary: The Wee Family's Modern Stack

The one-page picture:

**UOB is the family bank that bought its own digital spine.** Founded 1935 by Wee Kheng Chiang and run by three generations of the Wee family (✅), UOB enters its tenth decade as the third of Singapore's big three — with a systems story no sibling guide in this series can match: **in 2025, UOB acquired Silverlake Axis, the vendor of its SIBS core-banking estate** (✅ — the voluntary offer via E2I at S$0.36/share, closed and delisted November 2025). The family that built the bank now owns the software that runs the bank.

Around that spine:

- **The core (§2)**: Silverlake-family SIBS across the ASEAN franchises (⚠ per-entity scope), now vendor-owned; the Citi-acquisition books (Indonesia, Malaysia, Thailand, Vietnam, 2022) integrated via an automation-heavy factory — 97.8 million records migrated in four months by Blue Prism RPA (✅).
- **The digital layer (§3)**: the UOB Mighty app — now branded **UOB TMRW** in Singapore — an all-in-one "bank, pay and play" surface with 10-minute onboarding, PayNow/Scan-to-pay, Mighty Insights, and gamified savings (✅); plus **TMRW, ASEAN's first mobile-only digital bank**, launched in Thailand in February 2019 (✅), extended to Indonesia (⚠ 2021).
- **The wholesale layer (§4)**: **UOB Infinity**, the corporate digital platform launched October 2020 (✅), under Group Wholesale Banking head Frederick Chin.
- **The payments layer (§5)**: participation in Singapore's FAST (2014) and PayNow (2017) rails and the regional overlays (⚠), with payment UX as the differentiator.
- **The AI layer (§6)**: the first Singapore bank to trial Microsoft 365 Copilot (October 2023, 300 staff — ✅), an S$2.1–2.5bn IT programme for 2024–26 with GenAI, blockchain, and quantum on the agenda (✅), and a documented RPA estate.
- **The security layer (§7)**: Money Lock, Kill Switch, and the fraud hotline (✅) — the post-2021 anti-scam stack marketed as a product.
- **The architecture (§8)**: hyperscaler specifics **not verified** (⚠ — the honest gap in this guide); Microsoft, SS&C, and Silverlake as the named platform partners; the Punggol Digital District tech campus (first bank in, 2026, ✅) as the physical anchor.

**The final word — the Wee family's modern stack.** UOB's software landscape is the most *vertically integrated* of the Singapore trio: a family-controlled bank that wraps a vendor legacy core with gamified digital channels, automated its largest-ever integration (the Citi books), adopted genAI early (Copilot, 2023), hardened its consumer controls (Money Lock, Kill Switch), and then did what no peer has done — **bought the core vendor outright**, making the Wee family's bank the owner of the software spine running not only UOB but much of ASEAN banking. The modern stack is real: TMRW, Infinity, Copilot, RPA at the 100-million-record scale, a PDD tech campus. What is *not* public — the hyperscaler deals, the data platform, the middleware, the per-franchise core map — is the honest boundary of this guide (§11), and the S$2.1–2.5bn IT budget says the answer to "what's next" is being paid for now.

---

## 11. Claims Status and Verification Notes

### The Claims-Status Table

| Claim | Evidence | Status |
|---|---|---|
| Founded 6 August 1935 as United Chinese Bank by Wee Kheng Chiang; renamed United Overseas Bank 1965 | Wikipedia/directory record; Business Times family profile ("Wee Cho Yaw's father, Wee Kheng Chiang, founded UOB in 1935") | ✅ verified (1935 founding); ⚠ 1965 rename via secondary record |
| Wee family has run UOB across three generations; Wee Ee Cheong = Deputy Chairman and CEO (third-generation chief) | Euromoney ("three generations"); uobgroup.com board page; Straits Times profile (2025) | ✅ verified |
| Wee Cho Yaw (1929–2024), Chairman Emeritus | Business Times succession coverage; press obituary coverage | ✅ verified |
| 2025 = UOB's 90th anniversary | Straits Times (2025) | ✅ verified |
| Group entities: UOB Singapore/Malaysia/Thailand/Indonesia/Vietnam, UOBAM, UOB Kay Hian | Brand-level: uob.com.sg, uob.co.th, uob.com.vn, UOB group pages | ✅ verified (brand level) |
| UOB acquired Citigroup's consumer banking businesses in ID/MY/TH/VN (2022) | Widely reported; SS&C Blue Prism case study (the "newly acquired business") | ✅ verified (reported) |
| S$2.1–2.5bn IT investment 2024–26; GenAI/blockchain/quantum agenda | UOB Corporate Day 2024, via DBS Private Banking research note (Aug 2024) | ✅ verified (reported) |
| UOB runs Silverlake SIBS core banking | Industry consensus (KGI SGX research 2020; investor coverage naming UOB among SIBS clients) | ✅ consensus-level; ⚠ no vendor case study; per-franchise scope not public |
| **UOB acquired Silverlake Axis (2025)** | Business Times (S$0.36/share offer, Feb 2025); Markets Insider (offer via E2I Ltd); SBR (delisted 18 Nov 2025) | ✅ verified |
| TMRW = ASEAN's first mobile-only digital bank, Thailand Feb 2019; 3–5M customer target | UOB press release (14 Feb 2019, uobgroup.com PDF); Straits Times (14 Feb 2019) | ✅ verified |
| TMRW Indonesia launch (2021) | Widely reported; not re-verified from primary source this pass | ⚠ reported |
| UOB Mighty app features (Bank/Pay/Play, 10-min onboarding, Mighty Insights, PayNow/Scan-to-pay, Save-and-Harvest, Rewards+) | UOB Google Play listing + UOB app pages | ✅ verified (store-level) |
| Singapore app now branded "UOB TMRW" | uob.com.sg digital-banking page; UOB official LinkedIn | ✅ verified (current state); ⚠ exact rename date |
| UOB Infinity launched 1 Oct 2020; cash management + trade for corporates; regional rollout | Business Times + Straits Times (Oct 2020); UOB Infinity FAQ; UOB @ SFF 2024 | ✅ verified |
| Frederick Chin = Head of Group Wholesale Banking and Markets | UOB 90th-anniversary/vision content | ✅ verified |
| First SG bank to trial Microsoft 365 Copilot (Oct 2023, 300 staff) | UOB press release (3 Oct 2023, uobgroup.com PDF); multiple trade press | ✅ verified |
| Blue Prism RPA: 97.8M records migrated in 4 months (Citi integration) | SS&C Blue Prism case study (Apr 2025) | ✅ verified (vendor case study) |
| Money Lock, Kill Switch, fraud hotline | UOB official LinkedIn (Money Lock, hotline); UOB Mighty app documentation (Kill Switch) | ✅ verified |
| FAST (2014) and PayNow (2017) rails; UOB participation; app PayNow/Scan-to-pay | ABS/industry standard; UOB app text | ✅ verified (participation); ⚠ founding-participant claim |
| UOB cloud/hyperscaler strategy (Google Cloud, AWS) | Two targeted searches returned no UOB-specific agreements | ⚠ **not verified** — flag stands |
| UOB data platform (warehouse/lake/analytics) | No UOB-specific case study surfaced | ⚠ not verified |
| Punggol Digital District: first bank to move tech workforce in (2026) | Singapore Business Review (May 2026); Yahoo Finance; French Chamber | ✅ verified (reported) |
| UOB FinLab (innovation accelerator) | Widely documented in fintech press | ⚠ not re-verified this pass |
| TMRW platform stack (core/cloud/AI specifics) | Not public | ⚠ not verified |
| Temenos / FLEXCUBE / BaNCS at UOB | No evidence found | ⚠ unverified / not found |

### What Could Not Be Verified

The deliberate negative results of this pass — claims the brief asked about that this research could neither confirm nor refute, and which therefore stay unasserted:

- **The hyperscaler cloud strategy** (Google Cloud/AWS/Azure agreements) — two targeted searches surfaced nothing UOB-specific. ⚠
- **The per-franchise core map** — which UOB entity runs which core (Singapore flagship, Thailand, Malaysia, Indonesia, Vietnam) is not public; "Silverlake-family across the group" is consensus, not disclosure. ⚠
- **The Citi-migration target systems** — the Blue Prism case study documents the migration scale, not the destination core names. ⚠
- **The TMRW technology stack** — engine, cloud, core, and AI specifics are not public. ⚠
- **The exact Mighty → TMRW Singapore rebrand date.** ⚠
- **The data platform and middleware vendors** — no Cloudera-style case study surfaced (contrast OCBC §8). ⚠
- **Fraud-AI product names and model metrics.** ⚠
- **The payments-hub vendor, card processor, and treasury stack vendors.** ⚠
- **TMRW Indonesia's launch date (2021)** — reported, not primary-source-verified. ⚠
- **UOB's technology headcount** — no DBS-style disclosure exists. ⚠

### The Honesty Note: Methodology

**UOB is more disclosing than DBS about its strategy and less disclosing than OCBC about its named systems.** The verified anchors are strong in four places — the **history/family** (§1), the **2025 Silverlake acquisition** (§2 — the guide's most important verified fact), the **digital/wholesale product launches** (TMRW 2019, Infinity 2020, Mighty features, Copilot 2023), and the **security controls** (Money Lock, Kill Switch) — and weak in two: **the per-system vendor map** (which franchise runs what) and the **cloud/data layer** (no hyperscaler or data-platform disclosures surfaced).

1. **Verified anchors**: the 1935 founding and Wee family; Wee Ee Cheong's CEO role; the group entities; the S$2.1–2.5bn IT spend; the Silverlake acquisition (offer → delisting); TMRW (2019); Infinity (2020); the Mighty/TMRW app features; Microsoft 365 Copilot (2023); Blue Prism RPA; Money Lock/Kill Switch/hotline; the PDD move (2026); FAST/PayNow participation.
2. **Inference rules used**: (a) *structural inference* — the integration fabric, payments hub, cards estate, markets stack, and fraud-ML layer must exist in the standard platform classes ([Payments Hub Guide](payments_hub_guide.md), [Financial Fraud Detection at Scale Guide](financial_fraud_detection_at_scale_guide.md), [Nasdaq Calypso Guide](nasdaq_calypso_guide.md)) even though UOB does not name them; (b) *consensus-level verification* — the UOB–SIBS customer relationship is treated as verified-at-consensus (multiple independent sources) rather than vendor-case-study-verified, and flagged accordingly; (c) *ownership inference* — the 2025 acquisition is treated as the strongest revealed-preference evidence of the core relationship, without over-reading the acquisition's strategic intent (analyst commentary flagged ⚠).
3. **Deliberately not verified** (out of scope/undisclosed): internal system names, middleware/API gateway vendors, hyperscaler contracts, the data platform, TMRW's stack, fraud-model metrics, per-franchise core releases.
4. **The reader's rule**: if a system name in this guide lacks a ✅ in §11, treat it as a hypothesis about a class of system, not a fact about UOB.

### The Verification Pass: What Was Searched and What It Returned

For auditability, the research pass behind this guide (~13 targeted searches; web_extract was degraded — search-only backend — so all evidence is from search-result surfaces, with UOB's own uobgroup.com/uob.com.sg press-release PDFs and pages as the primary sources):

- **History/group**: "UOB United Overseas Bank founded 1935 Wee Kheng Chiang history Wee family" → Wikipedia, Business Times dynasty coverage, Euromoney ("three generations"), Grokipedia (6 Aug 1935, United Chinese Bank) ✅; "Wee Ee Cheong UOB CEO…" → uobgroup.com board page, Straits Times profile, press obituary facts ✅.
- **Core**: "UOB core banking system Silverlake" → KGI SGX research, investor coverage naming UOB among SIBS clients (consensus ✅); "Silverlake UOB case study" → **no vendor case study found** (⚠ recorded); "UOB Silverlake Axis acquisition offer E2I 2025" → **Business Times (S$0.36/share, Feb 2025), Markets Insider (offer via E2I Ltd), SBR (delisted 18 Nov 2025)** ✅✅ — the anchor of §2.
- **Digital**: "UOB Mighty app" → Google Play listing (`com.uob.mighty.app`) with the full feature set ✅; "UOB TMRW digital bank Thailand 2019" → **UOB's own launch release (14 Feb 2019, "The dawn of TMRW")** + Straits Times ✅; UOB Singapore digital-banking page confirms the current "UOB TMRW, All-in-1 Banking App" brand ✅.
- **Wholesale**: "UOB Infinity wholesale banking digital platform launch" → Business Times + Straits Times (1 Oct 2020), UOB Infinity FAQ, UOB @ SFF 2024 ✅.
- **AI**: "UOB generative AI 2023" → **UOB's own press release on Microsoft 365 Copilot (3 Oct 2023, first SG bank, 300 staff)** ✅; "UOB cloud-first strategy" → UOB Corporate Day 2024 IT-spend note (via DBS Private Banking) ✅ + **SS&C Blue Prism case study (97.8M records, 4 months)** ✅ + Punggol Digital District coverage (2026) ✅.
- **Cloud**: "UOB Google Cloud partnership" (two attempts) → **no UOB-specific result** — recorded as ⚠ not verified, the guide's most important open flag.
- **Security**: "UOB anti-scam Money Lock Kill Switch" → UOB's official LinkedIn (Money Lock on UOB TMRW app; fraud hotline) + the Mighty app documentation (Kill Switch) ✅.
- **Negative results (deliberately recorded)**: no evidence for UOB–Temenos/FLEXCUBE/BaNCS, UOB–hyperscaler agreements, a UOB data-platform case study, TMRW's stack, or fraud-AI product names — each recorded as ⚠ not verified rather than asserted either way.

**Residual risk statement**: the ⚠-inferred sections (§2.1–2.2 franchise core coverage, §8.1 cloud/data, §3.2 TMRW stack) are the most likely to be corrected by new information; the ✅ anchors (§1 history, §2.3 the Silverlake acquisition, §3.1–3.2 Mighty/TMRW, §4.1 Infinity, §6.1 Copilot, §7.1 security controls) are stable. Any citation-safe use of this guide should carry the §11 table rather than inline flags alone.

---

## 12. Glossary

| Term | Definition |
|---|---|
| **UOB** | United Overseas Bank Limited — Singapore's third-largest local bank (SGX: U11), a family-controlled universal banking group founded in 1935 |
| **United Overseas Bank** | The full legal name, adopted in 1965; the bank was founded as United Chinese Bank on 6 August 1935 |
| **Wee family** | The founding/controlling family: Wee Kheng Chiang (founder, 1935), Wee Cho Yaw (second generation, d. 2024), Wee Ee Cheong (third generation, Deputy Chairman and CEO) |
| **Wee Ee Cheong** | UOB's Deputy Chairman and Chief Executive Officer — the third-generation chief; the public driver of UOB's digital/AI agenda |
| **UOB Mighty** | UOB's Singapore mobile banking app (`com.uob.mighty.app`) — "bank, pay and play"; now branded UOB TMRW in Singapore |
| **TMRW** | (1) ASEAN's first mobile-only digital bank, launched by UOB in Thailand, February 2019 (Indonesia 2021 ⚠); (2) since 2024–25, the brand of UOB's Singapore consumer app |
| **Infinity** | UOB's corporate/wholesale digital banking platform (cash management + trade), launched 1 October 2020 |
| **Core banking** | The systems of record for deposits, loans, and accounts; at UOB, the Silverlake SIBS family across the ASEAN franchises — with the vendor (Silverlake Axis) acquired by UOB in 2025 |
| **Silverlake Axis** | The Singapore/Malaysia core-banking software vendor (SIBS); SGX-listed until November 2025, then acquired by UOB via E2I Ltd and delisted |
| **Digital banking** | The online/mobile channel layer over the core: UOB Mighty/TMRW (app), Personal Internet Banking (web), Infinity (business) |
| **Online banking** | UOB Personal Internet Banking — the web channel for personal banking |
| **Mobile banking** | The app-based channel — UOB Mighty/TMRW and the TMRW digital bank |
| **Wholesale** | The corporate/institutional business: UOB Infinity, SME banking, transaction banking, and markets |
| **Payments** | The movement of money: domestic rails (FAST/PayNow), regional overlays (PromptPay/DuitNow/BI-FAST), cards, and the app/wholesale payment surfaces |
| **FAST** | Fast And Secure Transfers — Singapore's real-time interbank clearing rail since 2014; the backbone for domestic instant payments |
| **PayNow** | Singapore's proxy-based instant-payment overlay on FAST (2017): pay by mobile number, NRIC/FIN, UEN, or VPA |
| **AI** | Artificial intelligence — at UOB, an estate spanning GenAI (Microsoft 365 Copilot), the RPA layer (Blue Prism), and the (⚠) fraud/insights models |
| **genAI** | Generative AI — UOB's 2023 Microsoft 365 Copilot pilot (first SG bank), the 2024–26 IT programme, and the broader adoption agenda |
| **Anti-scam** | UOB's customer-facing scam-defence controls: Money Lock, Kill Switch, the fraud hotline, and the (⚠) monitoring layer |
| **Fraud** | Fraud detection — the real-time transaction-monitoring and ML layer of the security estate; UOB-specific models not public |
| **Cloud** | The modern infrastructure layer; ⚠ UOB's hyperscaler strategy is not verified in this guide (Microsoft is the documented enterprise partner) |
| **Enterprise architecture** | The overall systems topology: SIBS-family cores (now group-owned), wrapped by TMRW/Mighty + Infinity channels, on FAST/PayNow rails, with RPA/Copilot automation, security controls, and a PDD tech campus |
| **Singapore** | UOB's home market and the centre of its systems estate — the local rails (FAST/PayNow), MAS regulatory context, and the flagship UOB TMRW franchise |
| **Malaysia / Thailand / Indonesia / Vietnam** | UOB's key ASEAN franchises — each with its own core/channel estate, TMRW markets (TH/ID), and the 2022 Citi-acquired consumer books (ID/MY/TH/VN) |
| **UOB Asset Management (UOBAM)** | The group's asset-management arm |
| **UOB Kay Hian** | The group's securities/brokerage arm |
| **E2I Ltd** | UOB's wholly-owned investment vehicle that made the 2025 voluntary offer for Silverlake Axis |
| **Punggol Digital District (PDD)** | Singapore's smart business district; UOB became the first bank to move its technology workforce there (2026) |

---

## 13. References and Further Reading

**Primary/company:**
- UOB press release: "The dawn of TMRW: ASEAN's first digital bank built for the digital generation" (14 February 2019) — uobgroup.com/web-resources/uobgroup/pdf/newsroom/2019/…
- UOB press release: "UOB pioneers trial of Microsoft 365 Copilot Generative AI tool across multiple business functions" (3 October 2023) — uobgroup.com/assets/web-resources/…/uob-microsoft-copilot-genai-tool.pdf
- UOB board page: Wee Ee Cheong, Deputy Chairman and CEO — uobgroup.com/uobgroup/about/management/board-of-directors.page
- UOB Singapore digital-banking page: "UOB TMRW, All-in-1 Banking App" — uob.com.sg/personal/digital-banking/index.page
- UOB Mighty app listing: Google Play, `com.uob.mighty.app` (feature set, Kill Switch documentation, 10-minute onboarding)
- UOB Infinity pages: product page, FAQs, UOB @ SFF 2024 — uob.com.sg/business/digital/infinity/…
- UOB official LinkedIn: Money Lock activation and fraud-hotline messaging (2025)
- UOB 90th-anniversary/vision content (Wee Ee Cheong, Frederick Chin, Eric Lim) — uobgroup.com
- UOB techecosystem pages (SFF showcase) — uobgroup.com/techecosystem
- UOB Vietnam Mighty pages (regional app presence) — uob.com.vn

**Press/analyst (used for verification):**
- The Business Times: "UOB's Wee clan, Singapore's richest banking dynasty, grapples with succession"; "Silverlake Axis receives privatisation offer with S$0.36 per share cash option" (Feb 2025)
- Markets Insider / Business Insider: "Silverlake Axis Faces Acquisition Offer by UOB" (offer made on behalf of E2I Ltd)
- Singapore Business Review: "Silverlake Axis to be delisted from SGX after E21 acquisition" (Nov 2025)
- The Straits Times: "UOB to launch its digital bank first in Thailand" (14 Feb 2019); "UOB launches digital banking platform UOB Infinity for corporate clients" (1 Oct 2020); "How UOB's Wee Ee Cheong masters the long game" (2025, 90th-anniversary framing)
- Euromoney: "Singapore banking: Inside UOB's new era" (three generations of the Wee family)
- DBS Private Banking research note: "United Overseas Bank: Key takeaways from UOB Corporate Day 2024" (Aug 2024 — S$2.1–2.5bn IT investment; GenAI/blockchain/quantum agenda)
- SS&C Blue Prism case study: "UOB Banking Digital Transformation with Automation" (Apr 2025 — 97.8M records, four months)
- Singapore Business Review / Yahoo Finance / French Chamber: "UOB becomes first bank to move into Punggol Digital District" (May 2026)
- KGI Securities (Singapore): Silverlake Axis initiation report (Jul 2020 — SIBS footprint); Wikipedia: "United Overseas Bank"; Grokipedia: "United Overseas Bank" (6 Aug 1935 founding)

**Sibling guides in this repository (cross-references used throughout):**
- [DBS Software Systems Guide](dbs_software_systems_guide.md) — the series pattern (systems deep-dive, worked example, claims-status discipline)
- [OCBC Software Systems Guide](ocbc_software_systems_guide.md) — the closest structural twin (Silverlake core case, SG-bank-trio context); the just-added sibling this guide mirrors
- [DBS Bank Guide](dbs_bank_guide.md), [Standard Chartered Guide](standard_chartered_guide.md) — the bank deep-dives and the structural model for the series
- [Universal Banking Model Guide](universal_banking_model_guide.md) — the group-structure reference (UOB as family-controlled universal bank)
- [Core Banking Systems Guide](core_banking_systems_guide.md) — the core umbrella (SIBS vendor table); [Temenos Guide](temenos_guide.md), [T24 Programming Guide](t24_programming_guide.md), [TAFJ Guide](tafj_guide.md), [Oracle FLEXCUBE Data Model Guide](oracle_flexcube_data_model_guide.md), [Apache Fineract Guide](apache_fineract_guide.md), [Chinese Bank Core Systems Guide](chinese_bank_core_systems_guide.md) — the non-Silverlake core families (contrast; no verified UOB relationships)
- [Interest Engines Core Banking Guide](interest_engines_core_banking_guide.md), [Posting Engine Core Banking Guide](posting_engine_core_banking_guide.md), [Core Banking Processes Guide](core_banking_processes_guide.md) — core accounting mechanics
- [Payments Hub Guide](payments_hub_guide.md), [ISO 20022 Core Processes Guide](iso_20022_core_processes_guide.md), [Financial Infrastructure Guide](financial_infrastructure_guide.md) — payment rails and hubs (FAST/PayNow)
- [Financial Fraud Detection at Scale Guide](financial_fraud_detection_at_scale_guide.md) — the anti-scam/anti-fraud platform class
- [Nasdaq Calypso Guide](nasdaq_calypso_guide.md), [Trust Bank Guide](trust_bank_guide.md) — treasury platform class and the SG digital-bank comparison
- [Insurance Software Systems Guide](insurance_software_systems_guide.md) — the contrast case (UOB has no group-owned insurer)
- [Supply Chain Finance Technologies Guide](supply_chain_finance_technologies_guide.md), [Asset Management Alternatives Guide](asset_management_alternatives_guide.md) — wholesale/finance and UOBAM platform classes
- [Data Governance Guide](../technology/data_governance_guide.md), [Event Stream Processing Guide](../technology/event_stream_processing_guide.md) — data-platform and integration-fabric classes

---

*End of guide. UOB is the family bank with the region's most unusual core story: a 1935-founded, three-generation Wee-family institution that wraps its Silverlake SIBS estate with gamified digital channels (Mighty → TMRW), runs ASEAN's first mobile-only digital bank (TMRW, 2019), launched the Infinity wholesale platform (2020), adopted genAI early (Microsoft 365 Copilot, 2023 — first SG bank), automated a 97.8-million-record acquisition integration with Blue Prism RPA, hardened its consumer controls (Money Lock, Kill Switch), anchored Singapore's Punggol Digital District (2026), and then — the final twist — bought the core vendor outright, making the Wee family's bank the owner of the software spine of ASEAN banking. The verified anchors in §11 (the 1935 founding, the Wee dynasty, the Silverlake acquisition, TMRW, Infinity, Copilot, Money Lock/Kill Switch, the S$2.1–2.5bn IT programme) are the load-bearing facts; the cloud strategy, data platform, per-franchise core map, and middleware remain the honest open flags — the Wee family's modern stack is real, and its connective tissue is private.*
