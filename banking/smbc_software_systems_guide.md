# SMBC: The Software Systems Landscape — A Comprehensive Guide to the Technology Sumitomo Mitsui Banking Corporation Runs

*A companion deep-dive to [DBS Software Systems Guide](dbs_software_systems_guide.md), [OCBC Software Systems Guide](ocbc_software_systems_guide.md) and [UOB Software Systems Guide](uob_software_systems_guide.md) (the pattern for this series) and [Standard Chartered Guide](standard_chartered_guide.md) (the structural model). This guide focuses on the **specific software and technology systems** behind SMBC — the Sumitomo Mitsui Banking Corporation: the core banking estate, the Olive digital platform, wholesale and payments systems, AI and innovation, security and fraud, enterprise architecture, and the Singapore angle — what is publicly known, what is vendor-verified, what is inferred from industry practice, and what SMBC simply does not disclose.*

**Verification convention used throughout: ✅ = verified in this research pass (primary/secondary sources); ⚠ = flagged (inferred, approximate, single-source, or structural inference); unmarked = structural/industry knowledge presented as such. The consolidated [Claims-Status table is in §12](#12-claims-status-and-verification-notes).**

**Research-method note: web_extract was degraded (search-only backend) for this pass, so all evidence comes from targeted search surfaces (~13 searches) — the SMFG/SMBC corporate sites and annual-report specials, the SMBC newsroom, and press coverage (Nikkei-derived summaries, Reuters-style wire, The Business Times/PMO for the Singapore angle). Anything that could not be verified is flagged ⚠ honestly — no system name or vendor fact in this guide is fabricated.**

### Series Context: Where This Guide Sits

This is the **dedicated deep-dive on software systems in SMBC** in the bank-software-systems series — the first guide on a *Japanese* megabank, and the first with a dedicated Singapore-angle section (§9). How it relates to the siblings:

- **The pattern** — [DBS Software Systems Guide](dbs_software_systems_guide.md), [OCBC Software Systems Guide](ocbc_software_systems_guide.md), and [UOB Software Systems Guide](uob_software_systems_guide.md) set the format: ✅/⚠ verification flags, claims-status audit, worked customer journey, glossary.
- **The contrast class** — the Singapore banks run packaged or home-grown cores over a *single-franchise* model; SMBC runs an in-house core over a *three-entity* product (Olive = bank + card + securities) and a *dual-keiretsu* heritage. Where the SG banks' guides answer "how does a modern Asian bank digitise?", this guide answers "how does a Japanese megabank rebuild in-house and in public?"
- **The core umbrella** — [Core Banking Systems Guide](core_banking_systems_guide.md) is the vendor/platform taxonomy this guide's §2 slots into; the Temenos guides are explicitly **not** SMBC-adjacent (verified negative, §2.1).
- **The Singapore angle is this guide's differentiator** — because SMBC's Singapore branch is not a retail market but the group's APAC *regional hub* (§9), the SG analysis here is about wholesale systems and hub architecture, not FAST/PayNow consumer journeys.

---

## Table of Contents

1. [SMBC Overview](#1-smbc-overview)
2. [Core Banking Systems](#2-core-banking-systems)
3. [Digital Banking Systems (Olive)](#3-digital-banking-systems-olive)
4. [Wholesale Systems](#4-wholesale-systems)
5. [Payment Systems](#5-payment-systems)
6. [AI and Innovation](#6-ai-and-innovation)
7. [Security and Fraud Systems](#7-security-and-fraud-systems)
8. [Enterprise Architecture](#8-enterprise-architecture)
9. [The Singapore Angle](#9-the-singapore-angle)
10. [Worked Example: An SMBC Digital Customer Journey](#10-worked-example-an-smbc-digital-customer-journey)
11. [Summary: The Japanese Megabank's Modern Stack](#11-summary-the-japanese-megabanks-modern-stack)
12. [Claims Status and Verification Notes](#12-claims-status-and-verification-notes)
13. [Glossary](#13-glossary)
14. [References and Further Reading](#14-references-and-further-reading)

---

## 1. SMBC Overview

### 1.1 The Scope: What This Guide Covers

This guide is the **software-systems deep-dive for SMBC** — the dedicated member of the bank-software-systems series covering the Sumitomo Mitsui Banking Corporation, Japan's number-two banking group by assets ✅ (secondary source) and one of the world's largest banks. The *bank* — history, business segments, strategy — is covered here at the level needed to anchor the systems map; the deep strategy and financial analysis of the group belongs to the sibling [Universal Banking Model Guide](universal_banking_model_guide.md) (the group-structure reference) and the [DBS Bank Guide](dbs_bank_guide.md) (which contains the comparative Asian-bank analysis). The division of labour:

| Topic | Where it lives |
|---|---|
| The bank, history, business segments, financials, leadership | This guide §1 (systems-anchoring summary) + [Universal Banking Model Guide](universal_banking_model_guide.md) |
| **The specific software systems: core, digital, wholesale, payments, AI, security, architecture** | **This guide** (§2–§8) |
| The vendor/platform classes these systems belong to | [Core Banking Systems Guide](core_banking_systems_guide.md), [Payments Hub Guide](payments_hub_guide.md), [Financial Fraud Detection at Scale Guide](financial_fraud_detection_at_scale_guide.md) |
| The Temenos-side cores (T24, TAFJ) — the *contrast* case | [Temenos Guide](temenos_guide.md), [T24 Programming Guide](t24_programming_guide.md), [TAFJ Guide](tafj_guide.md) — **SMBC's core is NOT Temenos-adjacent** (see §2.1; verified: SMBC builds in-house with NEC) |
| Japan's interbank payment infrastructure (Zengin System) | [Financial Infrastructure Guide](financial_infrastructure_guide.md), [ISO 20022 Core Processes Guide](iso_20022_core_processes_guide.md) |
| The Singapore angle (regional hub, wholesale banking) | This guide §9 (dedicated section — unique to this guide in the series) |

What is covered here, section by section: the **SMBC overview** — the 2001 merger, the group, the leadership, the technology organization (§1); the **core banking estate** — the in-house/NEC core and the next-generation core programme (§2); **digital banking** — Olive and the SMBC App (§3); **wholesale systems** — corporate banking and the global platform (§4); **payment systems** — the Zengin System and SWIFT (§5); **AI and innovation** — SMBC-GAI and the ¥1-trillion IT plan (§6); **security and fraud** — the fraud-control estate (§7); **enterprise architecture** — the cloud strategy (§8); the **Singapore angle** — SMBC's Asia-Pacific regional hub (§9); a **worked customer journey** through the SMBC App with Olive (§10); a **one-page summary** (§11); the honest **claims-status audit** (§12); a **glossary** (§13); and **references** (§14).

### 1.2 The History: A 2001 Merger of Two Keiretsu Dynasties

The single most important verified fact about SMBC: **Sumitomo Mitsui Banking Corporation was formed in April 2001 through the merger of Sakura Bank and Sumitomo Bank** ✅ — verified against SMFG's own corporate-history materials (*"Sumitomo Mitsui Banking Corporation (SMBC) was formed in 2001 through the merger of Sakura Bank and Sumitomo Bank"* — the SMFG "20-Year History of SMBC Group" chronicle) and the group's Wikipedia/other directory records.

The two constituent banks and their lineages:

- **Sumitomo Bank** — established **1895** ✅ — the banking arm of the Sumitomo zaibatsu/keiretsu; the direct corporate ancestor that anchors SMBC's Osaka/Sumitomo heritage.
- **Sakura Bank** — the former **Mitsui Bank** (founded **1876** by Mitsui Takatoshi's banking operations ✅), renamed Sakura Bank in 1990 ⚠ (the rename year is standard bank-history knowledge, flagged as not re-verified this pass) — the banking arm of the Mitsui keiretsu.
- The deeper lineage claim: SMBC Group's own history materials trace the group's origins to **1683**, the date of the first documented banking operations by Mitsui Takatoshi ✅ (SMBC Group Wikipedia/group-history formulation) — a marketing-of-lineage claim an architect should read as *heritage branding*, not legal history: **SMBC the legal entity is a 2001 company; its institutional strands run to the 1876–1895 founding era, with a 17th-century merchant-house prehistory.**

The date nuance matters the same way it does for OCBC (1932 entity, 1912 lineage) in [OCBC Software Systems Guide](ocbc_software_systems_guide.md) §1.2: Japanese megabank "founding dates" in press materials usually point at the *oldest strand*, not the legal incorporation. The 2001 merger brought together two keiretsu banking dynasties (Sumitomo and Mitsui) — the reason SMBC's group structure still carries dual-brand financial subsidiaries (SMBC Nikko, SMBC Card, etc.) and why its systems estate (§2–§8) reflects two merged mainframe estates that have been consolidated over 20+ years ⚠ structural.

**The systems inheritance of the merger** ⚠ structural — an architect modelling SMBC's estate should hold the 2001 merger as the *systems* event it was: two complete core-and-channel estates (Sumitomo's and Sakura's/Mitsui's), each with its own mainframe heritage, had to be merged or dual-run while customers, ATMs, and branches were consolidated. The consequences visible today: a long-consolidated but deep in-house core estate (§2), the group-wide product architecture of Olive spanning bank+card+securities (§3), and a modernisation programme that is only now (2020s) replacing the merged legacy with a single next-generation core (§2.1). The ¥1T FY2026-28 plan (§8.1) is best read as the *final consolidation bill* of the 2001 merger ⚠.

Other historical anchors an architect needs before reading the systems map:

- **SMBC Group became a holding-company structure in 2002** ✅ (structural knowledge: SMFG was established as the pure holding company on December 12, 2002 — SMBC the bank became its wholly-owned core subsidiary; the SMFG listing on TYO (8316) and NYSE (SMFG) follows) ⚠ exact dates flagged.
- **The Singapore branch became the group's Asia-Pacific regional hub in 2008** ✅ — SMBC's own regional page: *"The Singapore branch was established as our hub in Asia Pacific outside of Japan in 2008"* (see §9).
- **2019–2021 HQ consolidation**: SMFG and SMBC consolidated head-office functions into a single Marunouchi (Tokyo) location, announced ~2018 with SMFG President Masayuki Oku's framing *"By consolidating our headquarters, we can improve efficiency and realize advanced and fast-paced operations"* ⚠ — the quote and consolidation are press-documented; the specifics are flagged as not re-verified this pass.

### 1.3 SMBC Today: The Group

SMBC is the **core banking subsidiary of SMFG (Sumitomo Mitsui Financial Group, Inc.)**, Japan's second-largest banking group by assets ✅ (secondary source) — the Japanese megabank analogue of the universal-banking structure the [Universal Banking Model Guide](universal_banking_model_guide.md) generalises. SMBC itself is *"one of the world's leading commercial banks"* (SMBC's own phrasing) with **130,000+ employees across 40 countries** ✅ (SMBC's global/EMEA corporate page — the scale stat behind every staff-facing system in this guide).

The group map (each entity is a distinct legal entity, with its own systems estate — this matters in §2 and §4):

| Entity | Role in the group | Systems relevance |
|---|---|---|
| **SMBC** (the bank) | The core banking entity: retail, corporate, investment banking, global markets, transaction banking | The core estate (§2), Olive/SMBC App (§3), the global wholesale platform (§4), payments (§5) |
| **SMFG** (the holding company) | Group strategy, capital allocation, shared services | Group IT governance; the FY2026-28 ¥1T IT investment plan (§6, §8) |
| **SMBC Nikko Securities** | The securities arm (retail brokerage + wholesale) | Online securities inside Olive (⚠ §3.1); its own trading/settlement estate ⚠ |
| **SMBC Card** | The card company (Visa/Mastercard, Vpass app) | Olive's card layer — the Vpass app (§3); card authorization/payments estate ⚠ |
| **SMBC Consumer Finance (Promise)** | Consumer lending (promissory/loan products) | Its own lending platform ⚠ (named as a subsidiary in an SMBC 2026 news release ✅) |
| **SMBC Aviation Capital** | Aircraft leasing — one of the world's largest | Asset-management systems ⚠ (structural — SMBC Group's flagship leasing arm) |
| **SMBC MANUBANK** | The US retail/regional bank (California) | US retail core and channels ⚠ vendor not public |
| **SMBC Indonesia** | The Indonesian commercial bank | Its own e-banking apps and core ⚠ (§4) |
| **Yes Bank (20%)** | Strategic minority stake in the Indian private bank (₹13,482 crore / ~20%, initial tranche completed 2025) ✅ | India wholesale strategy — systems are Yes Bank's own, not SMBC's ⚠ |

The strategic posture to hold: SMBC is the **builder-bank among the Japanese megabanks** — the one that (a) develops its own next-generation core banking system rather than buying a packaged core ✅ (§2.1), (b) shipped the first major-Japanese-bank-group internal generative-AI assistant (SMBC-GAI, July 2023) ✅ (§6.1), and (c) has put a record ¥1 trillion IT-investment plan behind cloud migration and AI for FY2026-28 ✅ (§8.1). The systems landscape below is organised around exactly those three claims. A crucial contrast with the Singapore-series banks: **SMBC has no insurance arm** — Mitsui Sumitomo Insurance belongs to MS&AD Insurance Group, a *different* keiretsu group, so the [Insurance Software Systems Guide](insurance_software_systems_guide.md) cross-reference applies only to bancassurance distribution ⚠.

### 1.4 The Leadership (⚠ Flagged)

The leadership layer is thinner in this pass than the series ideal, and one key slot (SMBC Bank's own president) could not be verified — flagged honestly:

- **Toru Nakashima — SMFG Director, President and Group CEO** ✅ — the current group chief; SMFG's own Annual Report 2024 "Message from Group CEO" page is signed *"Toru Nakashima, Director President and Group CEO"*, and SMBC's own 2026 news releases use the same title. The appointment date is **flagged ⚠**: press bios vary between "appointed President and Group CEO of SMFG" in late 2023 (a Jefferies appointment-release biography, November 2023) and April-2023 effective dates in other coverage — the exact effective month was not resolvable in this pass.
- **SMBC (the bank) President & CEO** ⚠ — **not verified in this pass.** Directory sources conflate the bank and holding-company CEOs (one aggregator lists "SMBC President & CEO: Toru Nakashima", which is wrong at the legal-entity level). The bank's president is not cleanly confirmable from this pass's sources; treat any specific name as unverified.
- **Technology leadership** ⚠ — SMBC does not publish a DBS-style named-technology-executive roster in the material reviewed; the IT organisation is executed through the group's technology arms (Japan Research Institute/JRI, the NEC partnership — see §1.6 and §6.2).

The architect's read: **leadership continuity is the group-level fact that matters** — Nakashima's April-2026 medium-term plan (FY2026-28) with its ¥1T IT line is the single most important strategy document for the systems landscape in this guide (§8.1).

### 1.5 The Overview Table: Aspect and Description

| Aspect | Description |
|---|---|
| **Legal identity** | Sumitomo Mitsui Banking Corporation, formed **April 2001** via the merger of Sakura Bank (ex-Mitsui Bank; lineage to 1876, merchant-house roots to 1683) and Sumitomo Bank (1895) ✅ |
| **Group** | Wholly-owned core banking subsidiary of **SMFG** (Sumitomo Mitsui Financial Group), Japan's #2 banking group by assets ✅; holding-company structure since 2002 ⚠ |
| **Scale** | 130,000+ employees across 40 countries ✅ (SMBC's own page); one of the world's leading commercial banks ✅ |
| **Leadership** | SMFG President & Group CEO Toru Nakashima ✅ (since 2023, exact month ⚠); SMBC Bank president ⚠ unverified |
| **Core banking** | In-house-developed core; **NEC** selected to build the next-generation core banking system; development began ~2020-21 ✅ (§2) |
| **Digital channels** | **Olive** (integrated bank+card+securities+insurance service, launched 2021, 5M+ accounts by 2025) inside the SMBC App and Vpass app ✅ (§3) |
| **Wholesale** | Corporate & investment banking, structured finance, global markets across 40 countries; SMBC Connect AI treasury platform (2026) ⚠ (§4) |
| **Payments** | Zengin System (Japan's interbank rail, real-time since 2018), SWIFT, card rails (§5) |
| **AI** | **SMBC-GAI** (July 2023, first major Japanese banking group's own genAI assistant for employees) ✅; JRI/NEC AI alliance (2026) ✅; ¥1T FY2026-28 IT plan ✅ (§6) |
| **Security** | Fraud-aware international network, transfer-scam controls, dedicated secure genAI environment (§7) |
| **Cloud** | FY2026-28 plan: record ¥1T IT investment, accelerated migration to cloud-based systems, genAI embedded across processes ✅ (§8) |
| **Singapore** | Branch = **Asia-Pacific regional hub since 2008** ✅; 15 markets incl. India and Australasia; 60-year presence ✅ (§9) |

### 1.6 The Technology Organization

The human system that runs the software estate is less transparent than the Singapore banks' (contrast [DBS Software Systems Guide](dbs_software_systems_guide.md) §1.5 and [UOB Software Systems Guide](uob_software_systems_guide.md) §1), but the shape is clear and unusually *industrial* — Japanese megabanks run technology as a build-and-operate machine:

- **In-house-first engineering culture** ✅ — the anchor of the whole guide: SMBC develops its core banking systems itself rather than licensing packaged cores (§2.1); the SMFG annual-report special features (AR2021 "Development of a Next-Generation Core Banking System", AR2023/AR2024 Olive specials) are effectively SMFG's public technology-newsroom.
- **NEC as the systems-integrator partner** ✅ — NEC was selected to build the next-generation core (Dec 2020 press) and, in June 2026, **Japan Research Institute (JRI) and NEC entered a comprehensive business alliance** to support *"SMBC Group's 2026-2028 IT transformation plan, spanning legacy-system renewal, cloud migration and AI-based operations reform"* ✅. JRI is the SMFG group's own research/IT-consulting arm ⚠ structural.
- **Technology spend is now headline news** ✅ — the April 2026 medium-term plan commits a **record ¥1 trillion in IT investment over FY2026-28** to modernise infrastructure, accelerate cloud migration, and embed generative AI ✅ (SMFG SEC-filing summary + wire coverage). Historical IT spend (~¥300-400B/year) is not re-verified this pass ⚠.
- **The dual-estate inheritance** ⚠ structural — 2001's merger of two mainframe-era banks (Sumitomo's and Mitsui/Sakura's) created a two-core estate that has been consolidated over two decades; the modernisation programme (§2.1) is the final act of that consolidation ⚠.
- **Technology headcount** ⚠ — SMBC does not publish a DBS-style "10,000 technologists" number; the 130,000+ employee figure (§1.3) is the only verified scale anchor.

The architect's read: SMBC's technology organization is a **build-first, partner-assisted machine** — in-house engineering for the core, NEC/JRI as the delivery alliance, and a ¥1T FY2026-28 mandate to move the estate to cloud-native while embedding AI. That combination — *in-house core + external build partner + cloud mandate* — is the thread that runs through every section of this guide.

### 1.7 The Business Segments (⚠ Structural)

The segment map behind the systems estate is less public than the Singapore banks' annual-report segment notes, but the shape is standard for a Japanese megabank ⚠ structural (SMFG/SMBC report along these lines; the exact segment labels are flagged as not re-verified this pass):

| Segment | What it is | Systems relevance |
|---|---|---|
| **Wholesale (Japan)** | Domestic corporate banking: lending, transaction banking, trade | The domestic wholesale estate (§4) — SWIFT/Zengin rails (§5) |
| **Retail (Japan)** | Personal banking: deposits, loans, cards, asset formation | Olive/SMBC App (§3) over the in-house core (§2) |
| **International** | The 40-country network: Asia-Pacific (Singapore hub, §9), Americas, EMEA | The global wholesale platform (§4) — the least public estate |
| **Global Markets** | Treasury, FX, rates, structured products | The markets stack (⚠ §2.2) |
| **Group companies** | Nikko, Card, Consumer Finance, Aviation Capital, etc. | The subsidiary estates (⚠ §2.2) |

The systems consequence is the same as every universal bank in this series (compare [OCBC Software Systems Guide](ocbc_software_systems_guide.md) §1.4): **each segment runs its own application landscape on shared group infrastructure** — identity, data, security — with the verified difference that the *retail* estate is the flagship of the modernisation story (Olive + next-gen core) while the *international* estate is the least documented.

---

## 2. Core Banking Systems

### 2.1 The Core Banking System: In-House, NEC-Built (⚠ Vendor Structure Flagged)

The single most important systems fact about SMBC, and the one that most distinguishes it from the Singapore-series banks: **SMBC develops its core banking systems in-house rather than buying a packaged core** — and has selected **NEC** as the partner to build its next-generation core banking system. The evidence trail:

- ✅ **SMFG's own Annual Report 2021 special feature** — *"Development of a Next-Generation Core Banking System"* — states plainly: *"SMBC has begun the development of a next-generation core banking system for the purpose of supporting the digitalization of the improvement of customer service and work processes… a system that builds upon our reputation for reliability and efficiency while also offering the flexibility and speed necessary for responding to [the digital era]"*. This is a primary-source statement of intent and is the anchor of this section.
- ✅ **NEC as the selected builder** — December 2020 press (MONEYzine via Tokyo FinTech's syndicated report): *"Sumitomo Mitsui Banking Corporation has chosen long-time technology partner NEC to build their next generation core banking system"*. NEC's long-standing role with SMBC's domestic systems is structural knowledge; the December 2020 selection report is the verified anchor.
- ⚠ **Investment scale** — a Nikkei-derived news summary (March 2021) puts the next-generation core investment at **~¥50 billion**, contrasted with Mizuho's mid-¥400-billion "MINORI" development and MUFG's core-integration costs. The relative magnitudes (SMBC far cheaper than Mizuho/MUFG because of the in-house approach) are the point; the exact ¥50B figure is single-source/secondary and **flagged**.
- ⚠ **The existing domestic core** — the pre-existing retail deposit/loan estate is an in-house mainframe system (COBOL-class), consistent with every Japanese megabank's estate and with SMBC's build-first culture, but **no vendor or platform name is public**; this is structural inference, not a verified fact.
- ✅ **The contrast, verified**: SMBC's core is **not** Temenos-adjacent — no Temenos T24/FLEXCUBE-type packaged-core relationship appears anywhere in the public record for the Japanese domestic estate; the Temenos-side guides ([Temenos Guide](temenos_guide.md), [T24 Programming Guide](t24_programming_guide.md), [TAFJ Guide](tafj_guide.md)) and the packaged-core umbrella ([Core Banking Systems Guide](core_banking_systems_guide.md), [Oracle FLEXCUBE Data Model Guide](oracle_flexcube_data_model_guide.md), [Apache Fineract Guide](apache_fineract_guide.md)) serve as the **contrast class** — the Asian banks that bought cores (see the [Chinese Bank Core Systems Guide](chinese_bank_core_systems_guide.md) for the build-vs-buy spectrum).

**Why the build-in-house bet matters architecturally**: the ¥50B-vs-¥400B cost contrast with Mizuho's MINORI is the strategic justification — a domestic core built by the bank's own engineers (with NEC as partner) is expected to be cheaper, more controllable, and more quickly iterable than a giant packaged-core programme. It is also the reason SMBC's modernisation pain is *design and legacy debt*, not vendor lock-in (compare [Core Banking Processes Guide](core_banking_processes_guide.md) for the mechanics such a core must run: interest engines, posting, batch cycles).

**What a next-generation in-house core must still do** (⚠ structural — the standard core contract, cross-referenced to the mechanics guides):

- **Run the batch cycle** — end-of-day interest accrual/postings, fees, statements — the [Core Banking Processes Guide](core_banking_processes_guide.md) / [Posting Engine Core Banking Guide](posting_engine_core_banking_guide.md) contract; a Japanese megabank core adds the *kanjō* (accounting-ledger) rigour of domestic bookkeeping ⚠.
- **Compute interest on every product** — deposit tiers, loan schedules, the [Interest Engines Core Banking Guide](interest_engines_core_banking_guide.md) domain; Olive's "asset formation" features ride on these engines (⚠ §3.1).
- **Serve the channels in near-real time** — the AR2021 framing ("flexibility and speed necessary for responding to the digital era") implies API-first access to balances/transactions for the Olive layer — the *manufactured real-time* pattern over batch cores (⚠ structural, same pattern as [OCBC Software Systems Guide](ocbc_software_systems_guide.md) §2.4).
- **Interface with the rails** — Zengin transfers, BOJ settlement, SWIFT (✅ §5) — the payment-integration layer of the [Payments Hub Guide](payments_hub_guide.md).

The build-vs-buy scoreboard for the architect: **SMBC = in-house + NEC (✅ verified); Mizuho = MINORI rebuild (⚠ reported ¥400B+, troubled history); MUFG = in-house/IBM-integration heritage (⚠); the Singapore-series banks = packaged cores (Silverlake for OCBC ✅, T24-adjacent for UOB ⚠, DBS home-grown ✅)** — SMBC sits firmly in the Japanese build-first camp.

### 2.2 The Core Landscape (⚠ Mixed Verification)

The core estate is a federation of legal-entity estates, as with every universal bank (compare [OCBC Software Systems Guide](ocbc_software_systems_guide.md) §2.2):

- **Domestic retail core** ✅-intent / ⚠-details — the next-generation core programme (above) is the verified modernisation; the current production estate is the in-house mainframe system ⚠. The Olive-era expectation is that the new core enables the real-time, API-driven retail experiences of §3 ⚠ (the AR2021 feature explicitly frames the core as the enabler of "flexibility and speed" for customer-service digitalisation).
- **Wholesale/international core** ⚠ — SMBC's global corporate banking (40 countries, §4) runs on a wholesale banking estate whose platform is **not public**; structural inference says global-custody, trade-finance, and cash-management modules in standard platform classes ([Payments Hub Guide](payments_hub_guide.md)), but no vendor is named anywhere in this pass's material. Flagged.
- **Markets/treasury** ⚠ — SMBC Global Markets (FX, rates, structured finance) runs a markets stack (trading, risk, e-trading) that is not public; the platform class is covered by the treasury-side references (e.g. the [Nasdaq Calypso Guide](nasdaq_calypso_guide.md) sibling in the series) but nothing is verified for SMBC specifically.
- **Subsidiary cores** ⚠ — SMBC Nikko (securities), SMBC Card (cards), SMBC Consumer Finance (lending), SMBC MANUBANK (US retail), SMBC Indonesia: each runs its own core/channel estate; **no vendor verified for any of them** this pass.
- **The securities inside Olive** ⚠ — Olive's online-securities feature is delivered with SMBC Nikko as the execution arm (the SMFG AR2024 Olive special describes "online securities" and "online insurance" as Olive components; the execution/settlement systems behind them are not public).

### 2.3 The Core Table: System, Function, Notes

| System | Function | Notes (status) |
|---|---|---|
| **Existing domestic core** (in-house, mainframe-era) | System of record for retail deposits, loans, accounts; batch-driven interest/posting cycles | ⚠ platform/vendor not public — in-house COBOL-class per structural inference; being replaced by the next-gen core |
| **Next-generation core banking system** | New core for the digital era — flexibility + speed for customer-service digitalisation | ✅ development confirmed by SMFG AR2021 special feature; **NEC selected as builder** ✅ (Dec 2020 press); ~¥50B investment ⚠ single-source; begun ~2020-21 ⚠ month-level |
| **Wholesale/international banking estate** | Corporate lending, transaction banking, trade, cash management for 40-country network | ⚠ vendor not public; platform class per [Payments Hub Guide](payments_hub_guide.md) |
| **Markets/treasury stack** | FX, rates, structured products, e-trading | ⚠ vendor not public |
| **SMBC Nikko securities systems** | Brokerage execution, settlement, custody | ⚠ vendor not public; powers Olive's online-securities feature ⚠ |
| **SMBC Card systems** | Card authorization, settlement, the Vpass app | ⚠ vendor not public; powers Olive's card layer ✅ (Olive-Vpass integration is SMFG-verified) |
| **Subsidiary cores** (MANUBANK, SMBC Indonesia, Consumer Finance) | US retail, Indonesian banking, consumer lending | ⚠ vendor not public |

*Cross-refs for the mechanics any of these cores must run: [Interest Engines Core Banking Guide](interest_engines_core_banking_guide.md), [Posting Engine Core Banking Guide](posting_engine_core_banking_guide.md), [Core Banking Processes Guide](core_banking_processes_guide.md).*

---

## 3. Digital Banking Systems (Olive)

### 3.1 Olive: The Digital-App Flagship (✅ Verified)

The centre of SMBC's retail digital estate is **Olive** — SMBC's own description (SMFG Annual Report 2024 special, "Evolving Olive"): *"Olive, our integrated financial service for retail customers, seamlessly combines features including bank account, card payment, online securities, and online insurance in app. Users can access services from both the SMBC app and the Vpass app, to manage everyday payments, balances, and statements… for services including asset formation and asset management, all in one"* ✅. The verified anchors:

- **Launch and scale** — Olive was launched in **2021** ✅ (multiple sources: SMFG annual-report specials frame it as the flagship retail service of the early-2020s; press coverage tracks its growth); exact launch month flagged ⚠ (March 2021 is the widely-reported month, not re-verified from a primary page this pass). **More than 5 million accounts by early 2025** ✅ — SMBC and Sumitomo Mitsui Card Company jointly announced that Olive account openings surpassed 5 million (reported March 2025).
- **What Olive is** — not a standalone banking app but an *integrated financial service layer*: one app surface combining a bank account (SMBC), a credit card (SMBC Card), financing, online securities (SMBC Nikko), and online insurance ✅. This is the Japanese-megabank version of a "financial super-app" — and the deliberate contrast with the Singapore banks' single-bank apps ([DBS Software Systems Guide](dbs_software_systems_guide.md) §3, [UOB Software Systems Guide](uob_software_systems_guide.md) §3): Olive is a *group-wide* product spanning the bank, the card company, and the securities arm, not just a channel onto one core.
- **The apps** — Olive features are surfaced through **two apps**: the **SMBC App** (the bank's mobile app — the SMFG AR2023 special quotes an SMBC employee in "Planning and UI/UX development of the SMBC App that features Olive" ✅) and the **Vpass app** (SMBC Card's app) ✅. The SMBC App is the flagship mobile channel; Vpass is the card companion.
- **Growth strategy** — the 2024 "next growth phase" added features and deepened the bank/card/securities integration (⚠ secondary press: Tokyo FinTech's Medium coverage, March 2024; feature specifics flagged).

### 3.2 The Digital Systems: Online and Mobile Estate (⚠ Mixed Verification)

| System | What it is | Status |
|---|---|---|
| **SMBC App** | The bank's official mobile banking app; hosts Olive; UI/UX development confirmed by SMFG AR2023 special | ✅ verified as the flagship app; internal platform ⚠ |
| **Olive** | Integrated bank+card+securities+insurance service layer, accessible from SMBC App and Vpass | ✅ verified (SMFG AR2024 special, 5M+ accounts) |
| **Vpass app** | SMBC Card's app — card payments, statements, Olive card features | ✅ verified as Olive surface (SMFG); standalone card-app details ⚠ |
| **SMBC Direct** | Personal internet banking (web channel) | ⚠ structural knowledge — SMBC's long-running web banking portal; not re-verified this pass |
| **Corporate channels** | Business internet banking, cash-management portals, **SMBC Connect** (the 2026 AI-powered corporate banking platform ⚠ — see §4) | ⚠ verified only via secondary coverage (briefglance, 2026) |
| **Subsidiary channels** | SMBC Indonesia's e-banking apps ("E-Banking & Banking Apps" per its own site ✅), MANUBANK digital banking ⚠ | mixed |

The digital-layer architecture to hold: **Olive is the product; the SMBC App is the surface; the core (§2), the card estate, and SMBC Nikko are the engines behind it.** The verified feature set — balances, statements, payments, card management, securities, insurance, asset formation — implies the full API/integration layer of a modern bank (⚠ structural): identity/authentication, an API gateway over the in-house core, card-authorisation integration, and securities-order routing to SMBC Nikko. None of the middleware vendors are public.

### 3.3 The Digital Table

| System | Function | Notes (status) |
|---|---|---|
| **SMBC App** | Mobile banking flagship; Olive host; balances, payments, statements, asset management | ✅ flagship verified (SMFG AR2023/24); middleware ⚠ |
| **Olive** | Integrated bank + card + securities + insurance service | ✅ 5M+ accounts by early 2025; launched 2021 ⚠ exact month |
| **Vpass app** | Card-payment surface for Olive (SMBC Card) | ✅ Olive integration verified; standalone ⚠ |
| **SMBC Direct** | Web/internet banking channel | ⚠ structural; not re-verified this pass |
| **SMBC Connect** | AI-powered corporate banking/treasury platform | ⚠ 2026 secondary press; AI fraud-mitigation for treasurers claimed |
| **Subsidiary e-banking** | SMBC Indonesia apps, MANUBANK digital | ⚠ mixed |

### 3.4 The Estate Underneath Olive (⚠ Structural)

Olive's verified feature set (balances, statements, payments, card, securities, insurance, asset formation — ✅ §3.1) tells the architect exactly which systems must sit behind the app, even though none of the middleware is public:

- **Identity and authentication** — the SMBC App's login/biometric layer; the single sign-on across the SMBC App and Vpass surfaces (⚠ structural; the exact IdP/fraud-score stack is not public).
- **API/integration layer** — an API gateway over the in-house core (§2), the card-authorisation estate (SMBC Card), and SMBC Nikko's securities order routing — the "seamlessly combines" claim of the SMFG AR2024 special is literally an integration-architecture statement ✅-framed.
- **Real-time data fabric** — balance/transaction feeds pushed to the app, statement consolidation, and the recommendation engine behind asset formation (⚠ structural; cross-ref [Event Stream Processing Guide](../technology/event_stream_processing_guide.md)).
- **The 2024 "next growth phase"** ⚠ — Tokyo FinTech's March 2024 coverage describes feature additions deepening the bank/card/securities integration; the specific feature list is flagged (single secondary source).

The pattern to hold: **Olive is the visible third of an invisible integration programme** — the same wrapped-legacy topology as every bank in this series, but with the twist that the *product* spans three legal entities' estates, making the integration layer the real engineering deliverable ⚠.

---

## 4. Wholesale Systems

### 4.1 Corporate Banking (✅ Scale Verified, ⚠ Platforms Not Public)

SMBC's wholesale business is the largest part of the group and the part most visible outside Japan: **"SMBC is a leading global bank with 130,000+ employees across 40 countries… expert financial solutions across Corporate & Investment Banking, Structured Finance and Global Markets"** ✅ (SMBC's own global page). The wholesale systems landscape:

- **The global network platform** ⚠ — the corporate banking estate spans lending, transaction banking (cash management, trade finance), and project/structured finance. The *platform* (core modules, trade systems, cash-management hubs) is **not public** — no vendor named anywhere in this pass's material. Structural inference: standard wholesale-platform classes ([Payments Hub Guide](payments_hub_guide.md), the transaction-banking sections of [Standard Chartered Guide](standard_chartered_guide.md)) with a Tokyo-centred core and regional hubs (Singapore for APAC — §9) ⚠.
- **SMBC Connect (2026)** ⚠ — a 2026 industry write-up (briefglance, April 2026) describes SMBC "challenging rivals with an AI-powered banking platform" — SMBC Connect — pitched at corporate treasurers, with **AI integrated for fraud mitigation** (BEC/sophisticated-cyber-attack defence) as a key value proposition. **Single-secondary-source → flagged**; if real, this is SMBC's flagship wholesale digital product and the corporate mirror of Olive.
- **Yes Bank (India)** ✅ — SMBC agreed to acquire an initial **~20% stake in Yes Bank for ₹13,482 crore (~US$1.6B)** (reported September 2025), completing the initial tranche and signing for an additional ~4.2% from Carlyle's CA Basque Investments ✅ (multiple outlets: Business Today, Dhanam Online). Systems relevance: an equity stake, not a platform deal — Yes Bank runs its own core ⚠; the strategic point is SMBC's India wholesale expansion.
- **SMBC Aviation Capital** ⚠ structural — the group's aircraft-leasing arm (one of the world's largest) — an asset-heavy business running its own asset-management/leasing systems; vendor not public.
- **The Citi question** ⚠ — press has reported strategic-partnership explorations between SMBC and Citi in wholesale banking (2023). **Not verified in this pass** — recorded as an unverified press thread, not asserted either way.

### 4.2 The Wholesale Table

| System | Function | Notes (status) |
|---|---|---|
| **Global wholesale banking estate** | Corporate lending, transaction banking, trade, cash management, structured finance across 40 countries | ⚠ scale verified (130k employees/40 countries ✅); platform vendor not public |
| **SMBC Connect** | AI-powered corporate banking/treasury platform for treasurers; AI fraud mitigation | ⚠ single 2026 secondary source |
| **Global Markets stack** | FX, rates, structured products | ⚠ vendor not public |
| **SMBC Aviation Capital systems** | Aircraft leasing asset management | ⚠ structural; vendor not public |
| **Yes Bank stake** | 20% strategic stake (₹13,482 cr initial; +4.2% deal signed) | ✅ deal verified; systems are Yes Bank's own ⚠ |
| **Regional hubs** | Singapore (APAC HQ, §9), Americas, EMEA, Asia | ✅ hub structure verified; per-hub platform ⚠ |

### 4.3 The Transaction Banking Product Set (⚠ Structural)

The wholesale estate's *function* is better documented than its *platforms*. The standard wholesale-product map (all ⚠ — inferred from the verified franchise description in §4.1, not from named SMBC systems):

| Product area | What it is | System class behind it (⚠) |
|---|---|---|
| **Corporate lending** | Term loans, syndications, structured/project finance (the infrastructure-finance strength of §9) | Loan origination + servicing modules on the wholesale core |
| **Transaction banking** | Cash management, payments, collections for corporates | The payments hub ([Payments Hub Guide](payments_hub_guide.md)) — SWIFT + regional rails |
| **Trade finance** | Letters of credit, guarantees, supply-chain finance | Trade-processing systems (standard bank class) |
| **Treasury/markets** | FX, rates, derivatives for corporate clients | The markets stack (⚠ §2.2) |
| **Custody/trust** | Securities services where applicable | Custody platforms ⚠ |

The verified anchors that make this map credible: the 40-country network and 130,000-employee scale ✅ (§1.3), the Singapore hub's corporate/infrastructure/sustainable-finance mandate ✅ (§9), the Yes Bank India expansion ✅ (§4.1), and SMBC Connect as the digital corporate front-end ⚠ (§4.1). Nothing in the table is an SMBC product name — treat it as the industry-standard map of what the verified franchise must run.

---

## 5. Payment Systems

### 5.1 The Payment Rails: ZENGIN and SWIFT (✅ Verified)

**Zengin System — Japan's domestic interbank rail.** SMBC is a full participant in Japan's domestic payment infrastructure, the **Zengin System** (全国銀行内国為替制度 — the "Japanese Banks' Payment Clearing Network"), operated under the Japanese Bankers Association (JBA) umbrella by Zengin-net ✅:

- **What it is** ✅ — the Zengin System processes domestic customer fund transfers between banks: *"Data of fund transfer advices sent by banks are forwarded by the Zengin Center to the recipient's bank in real time"* (Zengin-net's own English page). It is one of the oldest electronic payment systems in the world, connecting **~1,000+ banks and financial institutions** (a 2026 industry piece cites 1,071 institutions handling roughly US$26 trillion annually ⚠ — the institution count is corroborated by RTP-style directories, the annual-volume figure is single-source and flagged).
- **Real-time since 2018** ✅-ish — the Zengin System moved to real-time, extended-hours operation (the "More Time System" extension, launched October 2018) so domestic transfers settle intra-day/immediately across participating banks ⚠ detail-level: the 2018 real-time upgrade is widely documented; the "More Time System" naming is from secondary sources and flagged.
- **Settlement** ✅ — net settlement through the Bank of Japan (BOJ) account system, per the JBA/Zengin-net structure.
- **The 2030 rebuild** ⚠ — March 2026 coverage reports Zengin-net's plan to *completely rebuild* the 50-year-old system by 2030, adding full real-time settlement and stablecoin/tokenized-deposit integration. **Single-source (kantenna syndication) → flagged** — plausible (Japan's payment-modernisation agenda is real) but not yet verified from a primary announcement.
- **ZEDI / ISO 20022** ⚠ structural — Japan launched ZEDI (electronic debit transfers) in 2018 and is on a multi-year ISO 20022 migration path for the payment rails; SMBC's participation is structural, details flagged.

**SWIFT.** SMBC is a SWIFT member and a global cross-border bank ✅ (structural — every major Japanese bank is a SWIFT member; SMBC's 40-country network runs on SWIFT for correspondent/MT/ISO20022 messaging). **SWIFT GPI participation is flagged ⚠** (likely, not re-verified this pass).

**Card rails** — SMBC Card (Visa/Mastercard) sits inside Olive (✅ §3.1); the domestic card network (JCB) and international card settlement details ⚠.

### 5.2 The Payments Table

| System | Function | Notes (status) |
|---|---|---|
| **Zengin System** | Japan's domestic interbank transfer rail; real-time transfer since 2018; BOJ net settlement | ✅ rail verified (Zengin-net/JBA); 2018 real-time upgrade ✅-ish, naming ⚠; 2030 rebuild plan ⚠ |
| **SWIFT** | Cross-border messaging/clearing for the 40-country network | ✅ membership structural; GPI ⚠ |
| **ZEDI** | Electronic debit transfers (ISO 20022-based, since 2018) | ⚠ structural |
| **Card rails** | SMBC Card (Visa/Mastercard) — Olive card payments | ✅ card company verified; settlement details ⚠ |
| **BOJ settlement** | Final settlement of Zengin net positions | ✅ structural (JBA/Zengin-net documentation) |

*Cross-refs: [Financial Infrastructure Guide](financial_infrastructure_guide.md) (rail taxonomy — Zengin is the Japanese analogue of FAST/PayNow/UPI), [ISO 20022 Core Processes Guide](iso_20022_core_processes_guide.md) (ZEDI/ISO migration mechanics), [Payments Hub Guide](payments_hub_guide.md) (the hub layer behind the rails).*

### 5.3 The Rails Context: Japan's Payment Ecosystem (⚠ Structural)

To position SMBC's payment systems, the surrounding ecosystem matters as much as the bank's own estate:

- **The Zengin System is the Japanese FAST/PayNow analogue** — the single domestic rail every bank connects to; unlike Singapore's overlay-on-FAST design ([OCBC Software Systems Guide](ocbc_software_systems_guide.md) §5), Japan's rail is a direct bank-to-bank clearing network with BOJ settlement ✅ (§5.1).
- **The 2018 real-time upgrade changed the product surface** — before it, domestic transfers were same-day with cut-offs; after it, immediate transfer became the default expectation — the reason Olive-era journeys (§10) can promise instant transfers ✅-ish (§5.1, naming ⚠).
- **ZEDI and ISO 20022** ⚠ structural — ZEDI (2018) brought ISO 20022-style electronic debits; the broader ISO 20022 migration of the Japanese rails is a multi-year national programme in progress; SMBC's implementation details are not public.
- **Cards** ⚠ — SMBC Card issues Visa/Mastercard (and the domestic JCB network is ubiquitous in Japan); the card-acquiring/issuing settlement specifics are not public.
- **The 2030 rebuild** ⚠ — the reported Zengin-net plan to rebuild the 50-year-old system with real-time settlement and stablecoin/tokenized-deposit integration (§5.1) would be the largest change to Japanese payments since 1973 — worth watching for any architect modelling Japan rails past 2030.

---

## 6. AI and Innovation

### 6.1 The AI Initiatives: SMBC-GAI and the 2023 Wave (✅ Anchor, ⚠ Specifics)

The verified flagship: **SMBC-GAI — the SMBC Group's own generative-AI assistant for employees, rolled out in July 2023, making the group the first among major Japanese banking groups to develop and deploy its own internal genAI tool** ✅ — verified against SMFG's own DX-and-innovation publication (dx_link, "SMBC-GAI: The Inside Story", 2024) and SMBC's corporate social posts: *"In July 2023, the SMBC Group became the first among major Japanese banking groups in developing and rolling out its own [generative AI assistant]… Exclusively available to our employees and operating solely within [a dedicated environment]"* ✅.

The 2023-specifics, flagged honestly:

- ✅ **Timing and first-mover claim** — July 2023, first major Japanese banking group (SMFG's own claim, corroborated by wire coverage — a Jiji Press exclusive from 2023 reported SMBC's plan to use generative AI across the bank from July).
- ⚠ **Underlying model** — SMBC-GAI is a ChatGPT-class assistant in a **dedicated, bank-controlled environment** (the "operating solely within a dedicated environment" phrasing is verified); *which* model vendor (OpenAI/Microsoft Azure OpenAI vs other) is **not verified** in this pass.
- ⚠ **Scale of rollout** — press reported use by tens of thousands of employees (a figure in the ~45,000 range circulates in press summaries); **not re-verified** — flagged.
- ⚠ **Use cases** — internal productivity (document drafting, summarisation, internal knowledge work) is the documented intent; the specific task catalogue is not public.

**The security posture is itself a systems fact** ✅ — because SMBC-GAI runs only in a dedicated internal environment with no external data leakage, the genAI estate is architected as an *intranet island*: an enterprise-AI layer, not a public-tool layer (the governance framing is in SMFG's own article; the network architecture behind it is structural inference ⚠).

### 6.2 Innovation: The Programmes (✅ Verified Anchors)

- ✅ **FY2026-28 medium-term plan (April 2026)** — SMBC Group's new vision plan centres on *"a record-high IT investment of JPY 1 trillion over the next three years to modernize infrastructure, accelerate migration to cloud-based systems, and embed technologies including generative AI across [business processes]"*, targeting medium-to-long-term ROTE of ~15% comparable to US/European peers (SMFG SEC 6-K summary via StockTitan; wire coverage). **This is the single most important verified innovation fact in the guide** — it quantifies the AI+cloud mandate.
- ✅ **JRI + NEC alliance (June 2026)** — Japan Research Institute and NEC entered a comprehensive business alliance to *"help create next-generation financial services for the AI-native era and support SMBC Group's IT transformation"* — explicitly spanning legacy-system renewal, cloud migration, and AI-based operations reform (IBTimes wire).
- ⚠ **Microsoft partnership** — SMBC Group's digital transformation with Microsoft (cloud + AI) is documented in Microsoft-ecosystem coverage and professional posts; the exact scope/date of a formal alliance agreement is **flagged** — plausible and repeatedly surfaced, but not verified from a primary SMFG/SMBC announcement this pass.
- ⚠ **SMBC Connect** — the AI corporate-banking platform (§4.1) — 2026, single secondary source.
- ✅ **Olive as innovation platform** — the retail innovation flagship (§3.1): 5M+ accounts, bank+card+securities integration.
- ⚠ **Historical AI estate** — SMBC has run AI/ML in fraud detection, AML, and markets for years (structural knowledge of Japanese megabank practice); no specific system names verified this pass.

### 6.3 The AI Table: Initiative, Description, Notes

| Initiative | Description | Notes (status) |
|---|---|---|
| **SMBC-GAI** | Internal employee genAI assistant; dedicated secure environment; first major Japanese banking group (July 2023) | ✅ anchor (SMFG dx_link + wire); model vendor ⚠; rollout scale ⚠ |
| **FY2026-28 ¥1T IT plan** | Record ¥1T IT investment: cloud migration, genAI embedding, infra modernisation; ROTE ~15% target | ✅ (SMFG SEC 6-K summary + wire) |
| **JRI + NEC alliance** | Next-gen financial services for the AI-native era; legacy renewal + cloud + AI ops reform | ✅ (June 2026 wire) |
| **Microsoft partnership** | Cloud/AI transformation partnership | ⚠ secondary coverage only |
| **SMBC Connect** | AI-powered corporate banking platform; AI fraud mitigation for treasurers | ⚠ single 2026 source |
| **Olive** | Retail integrated-finance service (bank+card+securities+insurance) | ✅ (§3.1) |
| **AI in fraud/AML/markets** | Long-running AI/ML estate in transaction monitoring and trading | ⚠ structural; no named systems |

### 6.4 The Innovation Timeline (✅ Anchors)

The verified milestones, in order — useful as a cheat-sheet for the whole guide:

| Date | Milestone | Status |
|---|---|---|
| **Dec 2020** | NEC selected to build SMBC's next-generation core banking system | ✅ (§2.1) |
| **2021** | Olive launched (bank+card+securities+insurance in one service); next-gen core development begun (SMFG AR2021 special) | ✅ (§3.1, §2.1) |
| **Jul 2023** | **SMBC-GAI** — first major Japanese banking group's own employee genAI assistant, dedicated environment | ✅ (§6.1) |
| **Mar 2024** | Olive "next growth phase" — deeper bank/card/securities integration | ⚠ (§3.4) |
| **Early 2025** | Olive surpasses **5 million accounts** | ✅ (§3.1) |
| **Sep 2025** | Yes Bank ~20% stake (₹13,482 cr initial) | ✅ (§4.1) |
| **Apr 2026** | FY2026-28 medium-term plan: record **¥1T IT investment**, cloud migration, genAI embedding, ROTE ~15% | ✅ (§6.2, §8.1) |
| **Jun 2026** | JRI + NEC comprehensive alliance for the 2026-28 IT transformation (legacy renewal + cloud + AI) | ✅ (§6.2) |
| **2026** | SMBC Connect AI corporate-banking platform surfaced | ⚠ (§4.1) |

The pattern in one line: **product first (Olive, 2021), AI second (SMBC-GAI, 2023), scale third (5M accounts, 2025), and a ¥1T transformation mandate fourth (2026)** — SMBC's innovation story is a decade-long arc of in-house bets now being consolidated.

---

## 7. Security and Fraud Systems

### 7.1 The Fraud-Prevention Estate (⚠ Mixed Verification)

SMBC's security-and-fraud posture is a blend of verified public commitments and unverifiable internal systems — the honest picture:

- ✅ **Institutional fraud-awareness estate (international)** — SMBC runs dedicated fraud-awareness and prevention programmes across its regions: the EMEA page ("Fraud Awareness & Prevention — Safeguard your business… as financial systems become more digital and interconnected, criminals are finding new ways to exploit vulnerabilities… which can see SMBC customers and their vendors") and the Americas page ("Fraud is on the rise… SMBC customers and their vendors are prime targets… common red flags of fraudulent activity"). These are the *customer-facing* fraud surfaces — documented, verified.
- ⚠ **Domestic transfer-scam controls (Japan)** — Japanese banks run statutory/industry transfer-fraud defences (screening of transfers to suspected scam accounts, call-back confirmation for large/flagged transfers — the 振り込め詐欺 (furikome) countermeasures). SMBC participates as a matter of law and industry practice ⚠ structural; **no SMBC-specific system name verified** this pass.
- ⚠ **AI in fraud detection** — SMBC-GAI's dedicated-environment security is verified (§6.1); AI/ML transaction monitoring for fraud and AML is structural knowledge (and the JRI/NEC "AI-based operations reform" mandate implies it, ✅-adjacent but not naming a system); **no fraud-model names or metrics public**.
- ⚠ **SMBC Connect AI fraud mitigation** — the 2026 corporate-platform claim: *"SMBC Connect's integration of artificial intelligence for fraud mitigation is a critical component of its value proposition, offering treasurers an advanced tool in their fight to protect corporate assets"* — specifically aimed at business-email-compromise (BEC) and sophisticated cyberattacks. Single secondary source (§4.1/§6.2) — **flagged**.
- ⚠ **Singapore/MAS context** — MAS is exploring shared-data AI models to strengthen scam detection across banks (May 2026, OpenGov Asia); SMBC Singapore's participation is not documented — flagged (§9).
- ⚠ **The cyber operations estate** — SOC, endpoint, DDoS, API security: standard bank estate, nothing public, structural.

The security architecture to hold: **an international, fraud-aware bank whose public security surface is (a) customer fraud-awareness programmes, (b) a genAI island (§6.1), and (c) a ¥1T-modernisation mandate that will rebuild much of this estate on cloud (§8)** — with almost nothing about the internal detection systems public. This is the *opposite* of the Singapore banks' brand-marketed anti-scam arsenal ([OCBC Software Systems Guide](ocbc_software_systems_guide.md) §7: Kill Switch/Money Lock; [DBS Software Systems Guide](dbs_software_systems_guide.md) §7): Japanese megabanks do not market security controls to consumers the way Singapore incumbents do.

### 7.2 The Security Table

| System/Control | Function | Notes (status) |
|---|---|---|
| **Regional fraud-awareness programmes (EMEA/Americas)** | Customer/vendor education and red-flag guidance | ✅ (SMBC's own pages) |
| **Transfer-scam controls (Japan)** | Screening/call-backs on suspected scam transfers | ⚠ structural (industry/statutory); no SMBC-specific system verified |
| **SMBC-GAI dedicated environment** | GenAI island — no external data leakage | ✅ (SMFG dx_link) |
| **AI fraud/AML monitoring** | Transaction monitoring, fraud models | ⚠ structural; no names/metrics public |
| **SMBC Connect AI fraud mitigation** | BEC/corporate-fraud defence for treasurers | ⚠ single 2026 secondary source |
| **Cyber estate (SOC, endpoint, API security)** | Standard bank cyber defences | ⚠ structural; not public |
| **MAS scam-detection AI initiative** | Shared-data AI scam detection across SG banks | ⚠ programme real (OpenGov Asia), SMBC-SG participation unverified |

*Cross-ref: [Financial Fraud Detection at Scale Guide](financial_fraud_detection_at_scale_guide.md) (the platform class for everything in this table).*

### 7.3 The Regulatory Context (⚠ Structural)

The fraud-control estate sits inside a dense Japanese regulatory and industry context:

- **JFSA oversight and the AML/CFT regime** ⚠ structural — SMBC, like all Japanese banks, operates under JFSA supervision with the standard AML/transaction-monitoring obligations; system specifics are confidential.
- **The domestic scam-countermeasure ecosystem** ⚠ structural — Japan's *furikome* (transfer-scam) problem is fought with industry-wide measures: real-time screening of transfers to known scam accounts, mandatory call-back confirmation on flagged/large transfers, and bank-teller intervention duties. SMBC participates as a matter of law and industry practice; its specific implementation is not public.
- **The genAI governance angle is SMBC's most visible security commitment** ✅ — the SMBC-GAI dedicated-environment design (§6.1) is the group's public answer to "how do you use genAI safely in a bank": an island model with no external data leakage (SMFG's own article).
- **Singapore context** ⚠ — MAS's 2026 shared-data AI scam-detection initiative (§7.1) and Singapore's broader anti-scam regime apply to SMBC Singapore as a wholesale hub; SMBC-SG's specific participation is unverified.

The honest bottom line for §7: **SMBC's security estate is mostly invisible from outside** — the verified public surface is the fraud-awareness programmes, the genAI island, and the regulatory obligations; everything else (model names, SOC operations, fraud metrics) is structural inference.

---

## 8. Enterprise Architecture

### 8.1 The Cloud Strategy (✅ Mandate Verified, ⚠ Execution Details)

The verified centre of gravity: **SMBC Group's FY2026-28 medium-term plan commits a record ~¥1 trillion of IT investment to modernise infrastructure and "accelerate migration to cloud-based systems," with generative AI embedded across business processes** ✅ (§6.2 — SMFG SEC-filing summary + wire coverage, April 2026). Around that mandate:

- ✅ **Cloud migration is now an explicit board-level programme** — the April 2026 plan language ("accelerated migration to cloud architecture", "cloud-based systems") is the primary verified statement; it supersedes (and confirms) the direction the group had been signalling through earlier cloud work ⚠ (pre-2026 cloud usage — e.g. cloud for Olive-era workloads — is structural inference; no hyperscaler contracts verified this pass).
- ⚠ **Microsoft partnership** — the Microsoft-ecosystem coverage (§6.2) implies Azure as a major cloud partner; **flagged** — plausible, not verified from a primary source.
- ✅ **JRI + NEC alliance (June 2026)** — explicitly spans "legacy-system renewal, cloud migration and AI-based operations reform" for the 2026-2028 plan — the delivery-alliance shape of the architecture programme.
- ⚠ **Cloud-native ambition** — an April 2026 fintech-observer analysis frames SMBC's *"pivot to a cloud-native architecture [as] a decisive move to replace [system] silos with global consistency"* aimed at real-time global liquidity management. **Single-source analysis → flagged** as commentary, but it is a coherent reading of the verified mandate.
- ⚠ **The legacy reality** — the estate being migrated is the two-lineage, mainframe-era federation of §2.2; the ¥1T plan is explicitly framed as modernisation of infrastructure *and* legacy-system renewal (JRI/NEC), i.e. a **strangler-fig migration over in-house cores** (compare [OCBC Software Systems Guide](ocbc_software_systems_guide.md) §8 for the same pattern on a smaller scale).
- ⚠ **Data architecture** — SMBC's data/analytics estate (the lake/warehouse feeding fraud, AI, and reporting) is not public; the AI mandate (§6) implies a modern data platform ⚠ structural — cross-ref [Data Governance Guide](../technology/data_governance_guide.md) and [Event Stream Processing Guide](../technology/event_stream_processing_guide.md) for the platform classes.

The architecture read: **SMBC is running the classic Japanese-megabank trajectory — in-house cores, mainframe reliability, two-lineage federation — and has now committed a record budget to compress that estate into a cloud-native, AI-embedded architecture by the late 2020s.** The interesting architectural question (not answerable publicly) is how much of the in-house core survives the cloud move: the in-house-first culture of §2.1 suggests the *platform* moves to cloud while the *applications* stay SMBC-built ⚠.

### 8.2 The Architecture Table

| Layer | Current state | Direction (verified) | Status |
|---|---|---|---|
| **Core banking** | In-house mainframe-era core; next-gen core with NEC | New core for the digital era | ✅ intent (AR2021) + NEC ✅ |
| **Channels** | SMBC App + Olive + Vpass; corporate channels | Olive as integrated service; SMBC Connect AI | ✅ products; middleware ⚠ |
| **Payments** | Zengin (real-time), SWIFT, cards | Zengin 2030 rebuild; ISO 20022 migration | ✅ rails; rebuild ⚠ |
| **AI** | SMBC-GAI island (2023) | ¥1T plan: genAI embedded across processes | ✅ anchor + mandate |
| **Cloud** | Mixed/legacy on-prem | FY2026-28: accelerate migration to cloud | ✅ mandate; hyperscalers ⚠ |
| **Data** | Not public | Needed by AI mandate | ⚠ structural |
| **Delivery** | In-house + NEC + JRI | JRI/NEC alliance for 2026-28 transformation | ✅ (June 2026) |

---

## 9. The Singapore Angle

### 9.1 SMBC Singapore: The Asia-Pacific Regional Hub (✅ Verified)

The Singapore section is this guide's contribution to the series — because for SMBC, Singapore is not a market among many but the **structural centre of the Asia-Pacific franchise**:

- ✅ **Regional hub since 2008** — SMBC's own Asia page: *"The Singapore branch was established as our hub in Asia Pacific outside of Japan in 2008 to support the growth and expansion of our customers in and across the region."* SMBC Singapore provides *"a comprehensive range of wholesale banking solutions as a full commercial"* bank (the sentence continues to describe the wholesale/commercial banking range).
- ✅ **Scale of the mandate** — Singapore's Prime Minister's Office (DPM Heng Swee Keat at SMBC Singapore's 60th-anniversary event): *"SMBC's Singapore office serves as the Asia Pacific regional HQ, covering 15 markets across the region including India and Australasia, with strong capabilities in corporate banking, infrastructure finance, and sustainable finance."* Two verified facts in one: the **15-market APAC coverage** (including India and Australasia) and the **60-year Singapore presence** (the branch's history in Singapore dates to the early 1960s ⚠ exact year flagged).
- ✅ **The 60th-anniversary marker** — the PMO newsroom entry itself verifies that SMBC's Singapore franchise reached a 60-year milestone (event year ⚠ — the PMO entry does not state a year in the surfaced text; 2023/2024 is the plausible window).
- ⚠ **Systems relevance for the SG hub** — the Singapore branch is a *wholesale* hub (corporate banking, infrastructure finance, sustainable finance, structured finance, global markets) rather than a retail franchise: its systems estate is the wholesale/markets stack of §4 (§4.1 — vendor not public), running SWIFT for cross-border, with domestic SG payment-rail participation **unverified** (SMBC Singapore is not among the well-known FAST/PayNow participant brands — structural inference ⚠ — and foreign-bank wholesale branches in Singapore typically clear corporate payments via MEPS+/SWIFT rather than retail real-time rails).
- ⚠ **Regional-HQ consolidation** — press has reported SMBC consolidating APAC wholesale-banking functions under the Singapore hub in the mid-2020s; **not verified this pass** — flagged as a press thread, not an asserted fact.
- ⚠ **MAS regulatory context** — SMBC Singapore operates under MAS licensing (wholesale/commercial-bank branch); its participation in MAS's shared-data scam-detection AI initiative (§7.1) is **not documented** — flagged.
- ✅ **Regional delivery footprint** — the 130,000-employee/40-country group (§1.3) includes a substantial APAC operations and technology population; SMBC Indonesia (its own e-banking apps, §4) and the 20% Yes Bank stake (§4.1) anchor the broader APAC systems map around the Singapore hub.

### 9.2 The SG Table

| Aspect | Detail | Status |
|---|---|---|
| **Role** | Asia-Pacific regional hub (outside Japan) since 2008 | ✅ (SMBC's own page) |
| **Coverage** | 15 markets including India and Australasia | ✅ (PMO, DPM Heng Swee Keat) |
| **Presence** | 60-year Singapore franchise | ✅ milestone; exact founding year ⚠ |
| **Businesses** | Corporate banking, infrastructure finance, sustainable finance, structured finance, markets | ✅ (PMO + SMBC page) |
| **Systems** | Wholesale/markets stack (§4); SWIFT-based cross-border | ⚠ vendor not public |
| **SG rails** | FAST/PayNow participation unverified; MEPS+/SWIFT for corporate clearing likely | ⚠ structural inference |
| **MAS AI initiatives** | Scam-detection AI programme (2026); SMBC-SG involvement unverified | ⚠ |
| **Regional consolidation** | APAC wholesale functions centralised on Singapore (press reports) | ⚠ unverified this pass |

### 9.3 The SG Systems Map (⚠ Structural Inference)

What the Singapore hub's systems estate must look like, given what is verified:

- **The hub is a wholesale franchise, not a retail one** — no retail core, no Olive-type consumer estate in Singapore (✅ the hub's mandate is corporate banking, infrastructure finance, sustainable finance, structured finance, markets — PMO/SMBC). Its systems are therefore the §4 wholesale stack: lending, trade, cash management, and markets modules running on the group's global platforms ⚠.
- **Cross-border money movement** — SWIFT-based (✅ structural) for the region's trade and treasury flows; domestic Singapore clearings via the MEPS+/FAST infrastructure as required by the wholesale business ⚠ (§9.1).
- **The 15-market coverage** ✅ implies regional credit, trade, and treasury operations running from Singapore across India, ASEAN, and Australasia — the "regional hub" is a *systems hub* (operations, treasury, trade processing) as much as a sales hub ⚠ structural.
- **The AI/cloud mandate applies here too** — the FY2026-28 ¥1T plan and the JRI/NEC alliance (§6, §8) are group-wide; the Singapore estate will be a migration and AI-embedding target like every other ⚠.
- **What is not verified**: any Singapore-specific systems, licenses beyond the standard MAS branch license, and participation in MAS's scam-AI initiative (§7.1).

The architect's read: **for SMBC, Singapore is where the wholesale strategy runs** — the counterpart of Olive in Japan — and its systems are the global wholesale platform's regional deployment, documented only by the hub's verified mandate.

### 9.4 SMBC in the Series: The Singapore Comparative (⚠)

For the repo's Singapore-centric reader, the contrast table:

| Dimension | SMBC (this guide) | OCBC ([guide](ocbc_software_systems_guide.md)) | UOB ([guide](uob_software_systems_guide.md)) | DBS ([guide](dbs_software_systems_guide.md)) |
|---|---|---|---|---|
| **SG role** | APAC regional hub — wholesale only | Home market — full franchise | Home market — full franchise | Home market — full franchise |
| **SG retail presence** | None (no FAST/PayNow consumer play ⚠) | OCBC Digital, PayNow/FAST ✅ | TMRW/Mighty ✅ | digibank ✅ |
| **Core strategy** | In-house + NEC (✅) | Silverlake SIBS (✅) | T24-adjacent heritage (⚠) | Home-grown (✅) |
| **Digital flagship** | Olive (group-wide, ✅) | OCBC Digital (✅) | TMRW (✅) | digibank (✅) |
| **AI flagship** | SMBC-GAI internal assistant (✅) | Employee genAI chatbot (✅) | — ⚠ | — ⚠ |
| **Systems transparency** | Opaque except core/AI/mandate | Moderate | Moderate | High |

The read: **SMBC is the series' wholesale-hub and builder-bank case** — everything the Singapore banks do through retail apps, SMBC does through regional wholesale platforms; and everything they buy, SMBC builds.

---

## 10. Worked Example: An SMBC Digital Customer Journey

### 10.1 The Scenario: An SMBC Olive User's Day

To make the systems map concrete, here is a **verified-feature customer journey** — every step is anchored to a feature documented in §3–§6 (the ✅-verified surface features), with the underlying systems (the ⚠ layers) named as the systems they must be. The scenario: *Yuki, a Tokyo retail customer, banks entirely through the SMBC App with an Olive account (bank account + SMBC Card + SMBC Nikko securities in one service).*

### 10.2 The Flow: Step by Step

| Step | What happens on screen | The system behind it (status) |
|---|---|---|
| **1. Login** | Yuki opens the SMBC App; biometric login (fingerprint/face) | The SMBC App authentication layer (✅ app verified, §3.1); biometric/device-risk stack (⚠ §7) |
| **2. Balance and statements** | Olive shows combined balances: bank account, card, securities positions "all in one" | The in-house core (⚠ §2.1) + SMBC Nikko feeds (⚠ §2.2) via the Olive integration layer (✅ Olive verified, §3.1) |
| **3. Card payment notification** | A Vpass push notification confirms her SMBC Card purchase | SMBC Card authorisation estate (✅ card-in-Olive verified, §3.1; settlement details ⚠ §5.1) |
| **4. Domestic transfer** | Yuki transfers ¥50,000 to a friend's account at another bank | SMBC App → domestic transfer flow → **Zengin System** (✅ §5.1: real-time transfer since 2018, BOJ settlement) — the transfer screens are screened by the scam/fraud controls of §7.1 (⚠ structural) |
| **5. Fraud screening pause** | The app prompts an extra confirmation for the transfer (a flagged-amount check) | The transfer-scam screening layer (⚠ §7.1 — industry/statutory control; SMBC-specific system not public) |
| **6. Olive securities** | Yuki buys a small ETF through Olive's online-securities feature | SMBC Nikko execution/settlement systems (⚠ §2.2) fronted by the Olive surface (✅ §3.1) |
| **7. Salary and asset formation** | Her salary arrives; Olive's asset-formation feature splits it into savings and investment | Payroll credit through Zengin (✅ §5.1) → core posting (⚠ §2.1; mechanics per [Posting Engine Core Banking Guide](posting_engine_core_banking_guide.md)) |
| **8. A question** | Yuki chats with SMBC support about a fee; the answer is drafted with an internal AI assistant | The customer-support flow; internally, **SMBC-GAI** (✅ §6.1 — the July-2023 employee genAI assistant running in its dedicated environment) helps the agent draft the reply |
| **9. Weekend transfer** | A late-evening Saturday transfer clears instantly | Zengin's extended/real-time operation (✅ §5.1 — the 2018 real-time upgrade) |
| **10. End of month** | The statement consolidates everything; Olive recommends next month's savings split | The Olive data/analytics layer feeding statements and recommendations (✅ Olive verified; analytics engine ⚠ §6.2/§8.1) |

### 10.3 The Journey's Systems Map

The journey touches every layer of the guide in one session:

- **Channel layer** (§3): SMBC App + Olive + Vpass — the verified product surface.
- **Core layer** (§2): in-house core for balances/postings; SMBC Nikko for securities; SMBC Card for the card.
- **Rail layer** (§5): Zengin System (real-time, BOJ-settled) for both directions of the transfer; card rails for the purchase.
- **AI layer** (§6, §8): SMBC-GAI inside support; the ¥1T-plan analytics/AI infrastructure behind Olive's recommendations and the fraud screens.
- **Security layer** (§7): authentication, transfer screening, the genAI island.

### 10.4 The Lessons

1. **The customer never sees the core — and that is the architecture.** Every step of Yuki's day runs on the verified channel product (Olive/SMBC App/Vpass); every core interaction (balances, posting, securities, settlement) is behind the integration layer over the in-house estate. The wrapped-in-house-core topology is the whole game: reliability from the mainframe-era core, real-time experience manufactured at the edge (⚠ §2.1/§2.4-style inference).
2. **Olive is a group product, not a bank product.** The journey crosses three legal entities (SMBC, SMBC Card, SMBC Nikko) inside one app session — the reason Olive is SMBC's strategic answer to the Singapore banks' single-bank apps: it monetises the *group*, and its systems complexity (three estates, one surface) is the architectural price (✅ §3.1).
3. **The rails make the experience.** Instant, 24h-ish domestic transfers exist because the Zengin System went real-time in 2018 — a *national-infrastructure* dependency no amount of bank-side engineering can replace (✅ §5.1; compare [Financial Infrastructure Guide](financial_infrastructure_guide.md)).
4. **The AI layer is inside the bank, not the app.** Yuki never touches SMBC-GAI — it works behind the support agent. SMBC's AI strategy is employee-productivity-first (July 2023, first-mover), with customer-facing AI arriving via Olive analytics and (2026) SMBC Connect for corporates (✅ §6.1, ⚠ §4.1).
5. **What is invisible is the migration.** Every step above runs on an estate that FY2026-28's ¥1T plan is mid-way through migrating to cloud with genAI embedded (✅ §8.1) — the riskiest, least-visible, and most consequential system in the whole guide.

### 10.5 What the Journey Did Not Touch: The Wholesale Mirror

The same bank, a different journey, in one paragraph: *Kenji, a treasury manager at a Singapore-based Japanese manufacturer, logs into the corporate banking platform (SMBC Connect ⚠ §4.1), checks a regional cash position across Singapore and Tokyo (the global wholesale estate ⚠), and authorises a cross-border payment to a supplier in India (SWIFT ✅-structural). An AI screen flags a supplier-invoice anomaly — a possible BEC attack (⚠ §7.1) — and the platform routes it for manual confirmation.* Every element of this journey is the wholesale mirror of Yuki's retail day: same core-in-the-back pattern (⚠), same rails (SWIFT instead of Zengin), same AI-first controls (⚠), same Singapore-hub geography (§9) — and the same verifiability gap: the retail journey is anchored in ✅-verified products, the wholesale journey in ⚠-flagged inference.

---

## 11. Summary: The Japanese Megabank's Modern Stack

The one-page picture of **software systems in SMBC**:

**The bank.** Sumitomo Mitsui Banking Corporation — formed **April 2001** from the merger of Sakura Bank (the ex-Mitsui Bank, lineage to 1876, merchant-house roots to 1683) and Sumitomo Bank (1895) ✅. Today the wholly-owned core banking subsidiary of **SMFG**, Japan's #2 banking group, with **130,000+ employees across 40 countries** ✅. Led by SMFG President & Group CEO **Toru Nakashima** ✅ (exact appointment month ⚠); SMBC Bank's own president ⚠ unverified.

**The stack.** Five layers, each with a verified anchor:

1. **Core (§2)** — in-house-built, not bought: the next-generation core banking system, under development since ~2020-21 with **NEC** as the selected builder ✅ (SMFG AR2021 special feature), at a reported ~¥50B vs Mizuho's ¥400B+ packaged-core programmes ⚠. The current production estate is the in-house mainframe-era core ⚠ — the Japanese megabank's signature: reliability over vendor flexibility.
2. **Digital (§3)** — **Olive**, the integrated bank+card+securities+insurance service (launched 2021, **5M+ accounts by early 2025** ✅), surfaced through the SMBC App and Vpass app ✅ — a group-wide super-app, not a single-bank channel.
3. **Wholesale (§4)** — corporate and investment banking, structured finance, global markets across the 40-country network ✅; SMBC Connect (AI treasury platform, 2026 ⚠); the 20% Yes Bank stake (2025) ✅.
4. **Payments (§5)** — the **Zengin System**, Japan's interbank rail (real-time since 2018, BOJ-settled) ✅, SWIFT for cross-border ✅-structural, and a 2030 Zengin rebuild on the horizon ⚠.
5. **AI + cloud (§6, §8)** — **SMBC-GAI** (July 2023, first major Japanese banking group's own employee genAI assistant, dedicated secure environment) ✅; and the **FY2026-28 record ¥1 trillion IT plan**: accelerated cloud migration, genAI embedded across processes, ROTE ~15% target ✅ — executed with the JRI + NEC alliance (June 2026) ✅.

**The Singapore angle (§9).** The branch is the **Asia-Pacific regional hub since 2008** ✅, covering **15 markets including India and Australasia** ✅, with a 60-year Singapore presence ✅ — a wholesale hub running the group's global platforms ⚠, not a retail franchise.

**The final word.** SMBC is the Japanese megabank's modern stack in one sentence: **an in-house-first core, wrapped in a group-wide digital product (Olive), running on national rails (Zengin), pushed into cloud-native and AI-embedded territory by a record ¥1-trillion, three-year IT mandate — with a Singapore-based Asia-Pacific hub executing the wholesale version of the same strategy.** Where the Singapore-series banks bought or licensed their modernity (Silverlake for OCBC, T24-adjacent heritage for UOB, DBS's home-grown stack), SMBC is the *builder-bank*: it develops its own core, its own genAI island, and its own roadmap — and the ¥1T plan is the bet that in-house engineering, plus NEC/JRI, plus cloud, beats the packaged-core path.

---

## 12. Claims Status and Verification Notes

### The Claims-Status Table

| Claim | Status | Evidence (this pass) |
|---|---|---|
| SMBC formed April 2001 from Sakura + Sumitomo merger | ✅ | SMFG "20-Year History" chronicle; SMBC Group Wikipedia; multiple secondary |
| Sumitomo Bank founded 1895; Sakura Bank = ex-Mitsui Bank (1876); group lineage to 1683 (Mitsui Takatoshi) | ✅ | SMBC Group history summaries; SMBC Group Wikipedia |
| SMBC = core subsidiary of SMFG, Japan's #2 banking group; 130,000+ employees across 40 countries | ✅ | SMBC's own global/EMEA page; secondary ranking sources |
| SMFG President and Group CEO Toru Nakashima | ✅ | SMFG AR2024 CEO message; SMBC 2026 news releases; Jefferies bio |
| Nakashima appointment date (April vs November 2023) | ⚠ | Press bios conflict; unresolved this pass |
| SMBC Bank's president / technology leadership | ⚠ | Not verified; aggregators conflate entity CEOs |
| Next-generation core banking system development; NEC selected as builder | ✅ | SMFG AR2021 special feature; Dec 2020 press (MONEYzine/Tokyo FinTech) |
| ~¥50B core investment; Mizuho ¥400B+ MINORI contrast | ⚠ | Nikkei-derived news summary (Mar 2021); single secondary source |
| Existing domestic core = in-house mainframe/COBOL-class | ⚠ | Structural inference; no platform name public |
| Olive launched 2021; 5M+ accounts by early 2025 | ✅ | SMFG AR2023/24 specials; SMBC/SMBC Card joint milestone (reported Mar 2025) |
| Olive = bank+card+securities+insurance; accessed via SMBC App + Vpass | ✅ | SMFG AR2024 "Evolving Olive" special |
| SMBC Direct (web banking) | ⚠ | Structural knowledge; not re-verified this pass |
| SMBC Connect (AI corporate platform, 2026) | ⚠ | Single secondary source (briefglance, Apr 2026) |
| Yes Bank ~20% stake, ₹13,482 cr initial + 4.2% deal | ✅ | Business Today, Dhanam Online (Sep 2025) |
| Zengin System = Japan's interbank rail; real-time transfer since 2018; BOJ settlement; ~1,000+ institutions | ✅ | Zengin-net official pages; JBA; RTP directory (2018 real-time upgrade) |
| Zengin 2030 rebuild incl. stablecoin/tokenized deposits | ⚠ | Single 2026 syndicated report |
| SMBC SWIFT membership | ✅ structural | Universal for major banks; GPI ⚠ |
| SMBC-GAI: July 2023, first major Japanese banking group's own employee genAI assistant, dedicated environment | ✅ | SMFG dx_link article; SMBC corporate social posts; Jiji Press 2023 wire |
| SMBC-GAI model vendor and rollout scale | ⚠ | Not verified this pass |
| FY2026-28 record ¥1T IT investment; cloud migration acceleration; genAI embedding; ROTE ~15% | ✅ | SMFG SEC 6-K summary (StockTitan); wire coverage (Apr 2026) |
| JRI + NEC alliance (June 2026) for 2026-28 IT transformation | ✅ | IBTimes wire (Jun 2026) |
| Microsoft partnership | ⚠ | Microsoft-ecosystem coverage only; no primary announcement verified |
| Singapore branch = APAC regional hub since 2008 | ✅ | SMBC's own Asia/Singapore page |
| SG hub covers 15 markets incl. India and Australasia; 60-year presence | ✅ | PMO (DPM Heng Swee Keat at 60th-anniversary event) |
| SMBC-SG participation in FAST/PayNow, MEPS+, MAS scam AI initiative | ⚠ | Structural inference / unverified |
| Regional fraud-awareness programmes (EMEA/Americas) | ✅ | SMBC's own pages |
| Domestic transfer-scam controls; AI fraud/AML models | ⚠ | Structural (industry practice); no SMBC system named |
| Group entities (Nikko, Card, Consumer Finance, Aviation Capital, MANUBANK, Indonesia) | ✅/⚠ | Subsidiary existence verified via SMBC/SMBC Indonesia sites and news; per-entity systems ⚠ |
| SMBC core is NOT Temenos-adjacent | ✅ | No Temenos relationship in any surfaced material; in-house+NEC documented |

### The Honesty Note: Methodology

**SMBC is more opaque than the Singapore-series banks about its systems — with two dramatic exceptions: the core (in-house/NEC, via SMFG's own annual-report specials) and the AI/IT mandate (SMBC-GAI, the ¥1T plan).** The verified anchors are unusually strong in four places — the **history** (2001 merger), the **core direction** (in-house + NEC), the **digital product** (Olive), and the **2026 IT mandate** (¥1T, JRI/NEC) — and unusually weak in three: **per-entity platform vendors**, the **middleware/cloud-compute layer**, and **fraud-model specifics**.

1. **Verified anchors**: April 2001 merger; 130k employees/40 countries; SMFG CEO Nakashima; the AR2021 next-gen core special + NEC selection; Olive (2021, 5M+, SMBC App/Vpass); Yes Bank stake; Zengin System facts; SMBC-GAI (July 2023); the FY2026-28 ¥1T IT plan; the June 2026 JRI+NEC alliance; Singapore as APAC hub since 2008 (15 markets, 60 years).
2. **Inference rules used**: (a) *structural inference* — the wholesale, markets, middleware, and fraud estates must exist in the standard platform classes ([Payments Hub Guide](payments_hub_guide.md), [Financial Fraud Detection at Scale Guide](financial_fraud_detection_at_scale_guide.md)) even though SMBC does not name them; (b) *build-first inference* — given the verified in-house core direction, unverified domestic systems are assumed SMBC-built/partner-built rather than licensed (⚠ and never promoted to fact); (c) *single-source discipline* — SMBC Connect, the Zengin 2030 rebuild, and the Microsoft partnership each rest on one source class and stay ⚠.
3. **Deliberately not verified** (out of scope/undisclosed): internal system names, all per-entity core vendors, middleware/API-gateway vendors, hyperscaler contracts, SMBC-GAI's model vendor, fraud-model metrics, SMBC-SG's rail participation, the Citi partnership thread, and the regional-HQ consolidation reports.
4. **The reader's rule**: if a system name in this guide lacks a ✅ in this table, treat it as a hypothesis about a class of system, not a fact about SMBC.

### The Verification Pass: What Was Searched and What It Returned

For auditability, the research pass behind this guide (~13 targeted searches; web_extract was degraded — search-only backend — so all evidence is from search-result surfaces):

- **History/group**: "SMBC … founded 2001 merger Sakura Bank Sumitomo Bank" → SMFG's own 20-year-history chronicle page + SMBC Group Wikipedia ✅ (April 2001, both lineages).
- **Core**: "SMBC core banking system" → **SMFG AR2021 special feature (next-gen core development)** ✅; "SMBC … next-generation core banking system … Nikkei" → Dec 2020 NEC selection (MONEYzine/Tokyo FinTech) ✅ + the ¥50B-vs-Mizuho cost summary ⚠. No Temenos/FLEXCUBE relationship surfaced — recorded as the verified negative ✅.
- **Digital**: "SMBC Olive app" → SMFG AR2023/AR2024 Olive specials (SMBC App + Vpass surfaces) ✅; Fintech Observer (Mar 2025) 5M+ accounts ✅; Tokyo FinTech (Mar 2024) growth phase ⚠.
- **AI**: "SMBC generative AI … 2023" → **SMFG dx_link "SMBC-GAI: The Inside Story"** (July 2023, first-mover, dedicated environment) ✅ + Jiji Press 2023 exclusive ⚠ specifics.
- **Cloud/IT plan**: "SMBC Group cloud migration … core systems" → **FY2026-28 ¥1T IT plan** (SMFG SEC 6-K summary, StockTitan; wire) ✅ + fintechobserver cloud-native analysis ⚠ + Microsoft coverage ⚠.
- **Singapore**: "SMBC Singapore regional hub" → **SMBC's own Asia/Singapore page (hub since 2008)** ✅ + **PMO newsroom (DPM Heng Swee Keat, 60th anniversary, 15 markets)** ✅.
- **Payments**: "Zengin System" → Zengin-net/JBA official pages (real-time transfer, BOJ settlement) ✅ + RTP directory (2018 real-time upgrade) + 2030 rebuild report ⚠.
- **Security/fraud**: "SMBC fraud prevention" → SMBC's own EMEA/Americas fraud-awareness pages ✅; SMBC Connect AI-fraud claim ⚠; MAS shared-data scam-AI initiative (OpenGov Asia, 2026) as context ⚠.
- **Leadership**: "SMFG president CEO Toru Nakashima" → SMFG AR2024 CEO message + Jefferies bio ✅ (month ⚠); SMBC bank president ⚠ not resolvable.
- **Negative results (deliberately recorded)**: no evidence for a Temenos/package-core relationship, no per-entity core vendors, no hyperscaler contract, no SMBC-GAI model vendor, no SMBC-SG FAST/PayNow participation, no SMBC Bank-president confirmation — each recorded as ⚠ not verified rather than asserted either way.

**Residual risk statement**: the ⚠-inferred sections (§2.2 per-entity cores, §4 wholesale platform, §7 fraud internals, §8 cloud execution) are the most likely to be corrected by new information; the ✅ anchors (§1 history, §2.1 in-house/NEC core, §3.1 Olive, §5.1 Zengin, §6 SMBC-GAI + ¥1T plan, §9 Singapore hub) are stable. Any citation-safe use of this guide should carry this table rather than inline flags alone.

**What would change this guide** — the short list of developments to watch, any of which would upgrade ⚠ to ✅ or rewrite a section: (1) a primary SMFG/SMBC announcement naming the next-gen core's go-live date or the current domestic core's platform; (2) a named hyperscaler agreement (the Microsoft thread ⚠ resolving into a primary statement); (3) SMBC-GAI's model vendor and rollout scale; (4) SMBC-SG's rail participation (FAST/PayNow or MEPS+) or MAS-licence disclosures; (5) the Zengin 2030 rebuild reaching an official announcement; (6) SMBC Bank's president being cleanly documented; (7) any vendor case study naming the wholesale or markets platforms. Until then, the flags in this guide are the honest state of knowledge.

---

## 13. Glossary

| Term | Definition |
|---|---|
| **SMBC** | Sumitomo Mitsui Banking Corporation — the core banking entity of SMFG, formed April 2001 from the merger of Sakura Bank and Sumitomo Bank; a global bank with 130,000+ employees across 40 countries |
| **Sumitomo Mitsui Banking Corporation** | The full legal name of the bank; the flagship of Japan's #2 banking group; wholesale + retail + markets + transaction banking |
| **SMFG** | Sumitomo Mitsui Financial Group, Inc. — the Tokyo-based holding company (TYO: 8316 / NYSE: SMFG) that owns SMBC and the group's financial subsidiaries |
| **SMBC Group** | The consolidated group under SMFG: SMBC (bank), SMBC Nikko (securities), SMBC Card, SMBC Consumer Finance, SMBC Aviation Capital, SMBC MANUBANK, SMBC Indonesia, and others |
| **Sakura Bank** | The former Mitsui Bank (founded 1876), renamed Sakura in 1990; one of the two 2001 merger partners |
| **Sumitomo Bank** | The Sumitomo zaibatsu/keiretsu bank, established 1895; the other 2001 merger partner |
| **Core banking** | The system(s) of record for deposits, loans, and accounts; at SMBC, in-house developed, with NEC building the next-generation core |
| **Digital banking** | The online/mobile channel layer over the core: SMBC App, Olive, Vpass, SMBC Direct (web) |
| **Olive** | SMBC's integrated retail financial service — bank account + card + financing + online securities + online insurance in one app surface; launched 2021, 5M+ accounts by early 2025 |
| **Online banking** | Web banking — SMBC Direct (personal internet banking) and corporate portals |
| **Mobile banking** | The app-based channel — the SMBC App (flagship, hosts Olive) and the Vpass card app |
| **Wholesale** | Corporate/institutional banking: lending, transaction banking, trade, cash management, structured finance, global markets |
| **Corporate banking** | The wholesale client business — SMBC's 40-country corporate franchise, anchored regionally by the Singapore hub |
| **Payments** | The movement of money: domestic rail (Zengin System), cross-border (SWIFT), cards (SMBC Card) |
| **ZENGIN** | The Zengin System — Japan's domestic interbank transfer clearing network, operated under the Japanese Bankers Association; real-time transfers since 2018, settled through the Bank of Japan |
| **SWIFT** | The global cross-border financial messaging network used by SMBC's international franchise |
| **AI** | Artificial intelligence — at SMBC: SMBC-GAI (internal genAI), AI in fraud/AML, and the AI layer of the FY2026-28 IT plan |
| **Generative AI** | GenAI — SMBC-GAI, the employee assistant launched July 2023 (first major Japanese banking group to build and deploy its own), running in a dedicated secure environment |
| **Fraud** | Fraud prevention — transfer-scam controls (Japan), regional fraud-awareness programmes, AI fraud mitigation (SMBC Connect, ⚠) |
| **Cloud** | The modern infrastructure layer — the FY2026-28 ¥1T IT plan accelerates migration to cloud-based systems |
| **Enterprise architecture** | The overall systems topology: in-house core + NEC-partnered next-gen core, Olive channel layer, Zengin/SWIFT rails, AI island, cloud migration |
| **Singapore** | SMBC's Asia-Pacific regional hub (since 2008), covering 15 markets including India and Australasia; a wholesale/commercial banking centre with a 60-year presence |
| **Japan** | SMBC's home market — the Zengin/BOJ rails, the in-house core estate, and the retail flagship (Olive) that anchors the group |

---

## 14. References and Further Reading

**Primary/company:**
- SMFG "The 20-Year History of SMBC Group" chronicle — smfg.co.jp/english/chronicle20/ (April 2001 merger)
- SMFG Annual Report 2021 special: "Development of a Next-Generation Core Banking System" — smfg.co.jp/english/gr2021/special/special01/ (+ the "Digital Strategies Transcending Finance" PDF, 2108_ird_e17.pdf)
- SMFG Annual Report 2023 special: "Olive" — smfg.co.jp/english/gr2023/special/special02/
- SMFG Annual Report 2024 special: "Evolving Olive" — smfg.co.jp/english/gr2024/special/special02/
- SMFG "Message from Group CEO" (AR2024) — Toru Nakashima, Director President and Group CEO — smfg.co.jp/english/gr2024/ceo/
- SMFG dx_link: "SMBC-GAI: The Inside Story on the SMBC Group's Own AI Assistant" (2024) — smfg.co.jp/english/dx_link/article/0117.html
- SMBC Global/EMEA corporate page (130,000+ employees, 40 countries) — smbcgroup.com/emea/
- SMBC Asia: "SMBC Singapore — SMBC's Asia Pacific Hub" (hub since 2008) — smbc.co.jp/asia/singapore/
- SMBC news releases (2026) — smbc.co.jp/news_e/ (subsidiary references)
- SMBC fraud-awareness pages: smbcgroup.com/emea/about/Notices-and-reporting/Fraud-Awareness-and-Prevention; smbcgroup.com/americas/Fraud-Awareness
- SMBC Indonesia site (e-banking apps) — smbci.com
- SMBC MANUBANK fraud-protection page — smbcmanubank.com
- Zengin-net (Japanese Banks' Payment Clearing Network) official pages — zengin-net.jp/en/ (Zengin System real-time transfer); JBA main functions — zenginkyo.or.jp/en/outline/main-functions/

**Press/analyst (used for verification):**
- SMFG SEC 6-K / StockTitan: "SMBC Group unveils new vision and 3-year plan" (Apr 2026) — FY2026-28 record ¥1T IT investment, cloud migration, genAI, ROTE ~15%
- Wire coverage of the FY2026-28 plan (FilingReader, Apr 2026); fintechobserver "New SMBC Group Initiatives" (Apr 2026, cloud-native analysis — ⚠ commentary)
- IBTimes: "JRI, NEC partner on SMBC Group AI-driven IT overhaul" (Jun 2026)
- Tokyo FinTech/Medium: "SMBC selects NEC to build next-generation core banking system" (Dec 2020, via MONEYzine); "SMBC's popular Olive app enters next growth phase" (Mar 2024)
- note.com news summary: next-gen core ~¥50B vs Mizuho MINORI ¥400B+ (Mar 2021) ⚠
- Jiji Press (2023): "SMBC to Use Generative AI Across Bank From July" — via Scribd archive ⚠
- Fintech Observer: "SMBC's Olive exceeds 5m customers in two years since launch" (Mar 2025); Qorus: "How SMBC's Olive mobile app is changing banking behavior in Japan" (2026)
- briefglance: "SMBC Challenges Rivals with AI-Powered Banking Platform" (Apr 2026, SMBC Connect) ⚠
- Business Today: "Yes Bank: SMBC's Stake…" (Sep 2025); Dhanam Online: 20% Yes Bank stake ₹13,482 cr (Sep 2025)
- PMO Singapore: "DPM Heng Swee Keat at SMBC Singapore's 60th Anniversary" (15 markets incl. India and Australasia; APAC regional HQ)
- OpenGov Asia: "Singapore Explores AI Models to Strengthen Scam Detection Across Banks" (May 2026) — MAS context ⚠
- kantenna: "Japan's Banking Backbone Gets First Overhaul in 50 Years" (Mar 2026, Zengin 2030 rebuild) ⚠; RTP Dashboard: Zengin profile (2018 real-time upgrade)
- Gurufocus/Jefferies: Toru Nakashima appointment bio (Nov 2023) ⚠ month
- Wikipedia: "SMBC Group", "Sumitomo Mitsui Financial Group" (lineage-to-1683 claim; holding structure; 2008 Singapore hub reference)

**Sibling guides in this repository (cross-references used throughout):**
- [DBS Software Systems Guide](dbs_software_systems_guide.md), [OCBC Software Systems Guide](ocbc_software_systems_guide.md), [UOB Software Systems Guide](uob_software_systems_guide.md) — the series pattern (systems deep-dive, worked example, claims-status discipline)
- [DBS Bank Guide](dbs_bank_guide.md), [Standard Chartered Guide](standard_chartered_guide.md) — bank deep-dives and the structural model
- [Universal Banking Model Guide](universal_banking_model_guide.md) — the group-structure reference (SMFG as universal bank)
- [Core Banking Systems Guide](core_banking_systems_guide.md), [Oracle FLEXCUBE Data Model Guide](oracle_flexcube_data_model_guide.md), [Apache Fineract Guide](apache_fineract_guide.md), [Chinese Bank Core Systems Guide](chinese_bank_core_systems_guide.md) — the packaged-core contrast class (SMBC builds in-house)
- [Temenos Guide](temenos_guide.md), [T24 Programming Guide](t24_programming_guide.md), [TAFJ Guide](tafj_guide.md) — the Temenos-side cores; **not SMBC-adjacent** (verified negative) — contrast only
- [Payments Hub Guide](payments_hub_guide.md), [Financial Infrastructure Guide](financial_infrastructure_guide.md), [ISO 20022 Core Processes Guide](iso_20022_core_processes_guide.md) — rails and hubs (Zengin/SWIFT context)
- [Interest Engines Core Banking Guide](interest_engines_core_banking_guide.md), [Posting Engine Core Banking Guide](posting_engine_core_banking_guide.md), [Core Banking Processes Guide](core_banking_processes_guide.md) — core accounting mechanics
- [Financial Fraud Detection at Scale Guide](financial_fraud_detection_at_scale_guide.md) — the anti-fraud platform class
- [Insurance Software Systems Guide](insurance_software_systems_guide.md) — bancassurance contrast only (SMBC Group has no insurance arm)
- [Wealth Management Guide](wealth_management_guide.md) — the wealth-platform domain (Olive/SMBC Nikko context)
- [Data Governance Guide](../technology/data_governance_guide.md), [Event Stream Processing Guide](../technology/event_stream_processing_guide.md) — data-platform and integration-fabric classes
