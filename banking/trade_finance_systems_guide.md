# Trade Finance Systems: A Comprehensive Guide — The Bank Trade-Finance Platform Landscape

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Banking Architecture / Trade Finance — the bank trade-finance systems deep-dive: the trade-systems landscape and its categories, the market (flagged), the vendor profiles (Finastra Trade Innovation, Oracle FLEXCUBE Trade, Temenos T24/Transact trade, Intellect iGTB, EdgeVerve/Infosys Finacle, TCS BaNCS), the platform comparison, the digitization platforms (Contour — status flagged — and Bolero), the trade stack architecture, the selection guidance, and a worked example: a Cymbal Bank's trade-systems landscape and stack design
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** August 2026

---

## Table of Contents

1. [The Systems Overview](#1-the-systems-overview)
   - 1.1 [The Trade-Systems Landscape](#11-the-trade-systems-landscape)
   - 1.2 [The Market (Flagged)](#12-the-market-flagged)
   - 1.3 [The Overview Table](#13-the-overview-table)
   - 1.4 [The Trade-Systems Timeline](#14-the-trade-systems-timeline)
   - 1.5 [The Platform Economics](#15-the-platform-economics)
2. [The Vendor Profiles](#2-the-vendor-profiles)
   - 2.1 [The Finastra Profile: Trade Innovation](#21-the-finastra-profile-trade-innovation)
   - 2.2 [The Oracle Profile: FLEXCUBE Trade](#22-the-oracle-profile-flexcube-trade)
   - 2.3 [The Temenos Profile: T24 Trade](#23-the-temenos-profile-t24-trade)
   - 2.4 [The Intellect Profile: iGTB](#24-the-intellect-profile-igtb)
   - 2.5 [The EdgeVerve Profile: Finacle Trade](#25-the-edgeverve-profile-finacle-trade)
   - 2.6 [The TCS Profile: BaNCS Trade](#26-the-tcs-profile-bancs-trade)
   - 2.7 [The Vendor Table](#27-the-vendor-table)
   - 2.8 [The Adjacent Specialists](#28-the-adjacent-specialists)
3. [The Platform Comparison](#3-the-platform-comparison)
   - 3.1 [The Comparison Table](#31-the-comparison-table)
   - 3.2 [The Comparison Deep-Dive](#32-the-comparison-deep-dive)
   - 3.3 [The Scenario-to-Platform Map](#33-the-scenario-to-platform-map)
4. [The Digitization Platforms](#4-the-digitization-platforms)
   - 4.1 [The Contour Profile (Status Flagged)](#41-the-contour-profile-status-flagged)
   - 4.2 [The Bolero Profile](#42-the-bolero-profile)
   - 4.3 [The Digitization Table](#43-the-digitization-table)
   - 4.4 [The Digitization Integration Pattern](#44-the-digitization-integration-pattern)
5. [The Architecture](#5-the-architecture)
   - 5.1 [The Trade Stack Layers](#51-the-trade-stack-layers)
   - 5.2 [The Architecture Table](#52-the-architecture-table)
   - 5.3 [The Integration Narrative](#53-the-integration-narrative)
   - 5.4 [The Trade Data Model at a Glance](#54-the-trade-data-model-at-a-glance)
   - 5.5 [Deployment and Operating Patterns](#55-deployment-and-operating-patterns)
6. [The Selection Guidance](#6-the-selection-guidance)
   - 6.1 [The Selection Criteria](#61-the-selection-criteria)
   - 6.2 [The Selection Table](#62-the-selection-table)
   - 6.3 [The Core-Embedded vs Standalone Decision](#63-the-core-embedded-vs-standalone-decision)
   - 6.4 [The RFP Annex: What the Bank Should Actually Ask](#64-the-rfp-annex-what-the-bank-should-actually-ask)
7. [The Worked Example: A Cymbal Bank's Trade-Systems Landscape](#7-the-worked-example-a-cymbal-banks-trade-systems-landscape)
   - 7.1 [The Scenario](#71-the-scenario)
   - 7.2 [The Stack Design](#72-the-stack-design)
   - 7.3 [The Lessons](#73-the-lessons)
   - 7.4 [The Current vs Target Estate](#74-the-current-vs-target-estate)
8. [The Summary: One Page](#8-the-summary-one-page)
9. [The Glossary](#9-the-glossary)
10. [The Related Guides and Sources](#10-the-related-guides-and-sources)

---

### How to Read This Guide

This is the dedicated **trade-finance-systems deep-dive** of the trade-finance series. It expands [Section 8 of trade_finance_guide.md](trade_finance_guide.md) — the trade-finance umbrella, which covers the instruments (LC, collections, guarantees), the rules (UCP 600, URC 522, URDG 758), the SWIFT MT700 family, the risks, and digitization — from a ~41-line systems *overview* into full platform depth: the systems landscape and categories, verified vendor profiles, a cross-platform comparison, the digitization platforms (Contour, Bolero), the architecture of a bank trade stack, selection guidance, and a worked example. This is the umbrella-then-deep-dive pattern also used by [chinese_core_banking_vendors_guide.md](chinese_core_banking_vendors_guide.md) (vendor profiles) and [policy_administration_systems_guide.md](policy_administration_systems_guide.md) (platform landscape).

Reading paths:

- **New to trade finance** — start with [trade_finance_guide.md](trade_finance_guide.md) Sections 1–7 (instruments, rules, messages, digitization), then come back here for the systems.
- **Vendor/product research** — [Section 2](#2-the-vendor-profiles) here, then the [comparison](#3-the-platform-comparison); core-system context in [core_banking_systems_guide.md](core_banking_systems_guide.md), [temenos_guide.md](temenos_guide.md) and [oracle_flexcube_data_model_guide.md](oracle_flexcube_data_model_guide.md).
- **Procurement / selection** — [Section 6](#6-the-selection-guidance) is the criteria; [Section 7](#7-the-worked-example-a-cymbal-banks-trade-systems-landscape) is a complete stack-design exercise.
- **Singapore/Asia angle** — [dbs_software_systems_guide.md](dbs_software_systems_guide.md) and [ocbc_software_systems_guide.md](ocbc_software_systems_guide.md) show bank trade operations in practice; the cross-border mechanics are in [../technology/late_arriving_data_guide.md](../technology/late_arriving_data_guide.md).

A note on verification: this guide was researched with targeted web searches in August 2026 (search-only backend — page extraction was degraded, so primary pages could not always be read in full). Facts **verified** from retrievable sources (vendor sites, industry press) are marked as such; items that are widely reported but could not be independently verified in this session are **flagged**; market-share figures are flagged throughout because no audited ranking exists. **Nothing in this guide is fabricated; where a fact could not be verified, the guide says so explicitly rather than guessing.** The verification posture follows [trade_finance_guide.md](trade_finance_guide.md) Section 12 and [chinese_core_banking_vendors_guide.md](chinese_core_banking_vendors_guide.md) Section 11.

Three quick orientation points before the content:

- **This is a systems guide, not a product guide.** The instruments, rules, and messages are covered in the umbrella; here they appear only as what the systems must process (rule timers, MT700 fields, discrepancy workflows).
- **"Flagged" is a status, not a slur.** It marks claims that are press-reported, vendor-claimed, or single-source — the things a diligent reader must re-verify before quoting them in a paper or a procurement.
- **The through-line is the architecture.** Read [Section 5](#5-the-architecture) early even if you start elsewhere: the six-layer stack and the "integration is the architecture" principle are what the vendor sections, comparison, and worked example all hang on.

---

# 1. The Systems Overview

## 1.1 The Trade-Systems Landscape

Trade finance in a bank runs on a **dedicated processing stack**, because the workflow — document examination, contingent limits, SWIFT messaging, guarantee lifecycle, discrepancy management — does not fit a vanilla core-banking loan module. The umbrella guide ([trade_finance_guide.md](trade_finance_guide.md) §8.1) names three layers; this deep-dive expands them into the full landscape of system categories a bank trade business actually runs. **Verified**: these categories match the commonly cited industry segmentation (vendor sites and trade-tech press) — a *bank trade-systems landscape* decomposes into:

1. **Trade processing platforms (the system of record)** — standalone, best-of-breed trade suites that originate, process, and lifecycle LCs, guarantees, and collections; own the trade document of record; generate/parse SWIFT messages; manage discrepancies, amendments, and drawings. This is the *core* of the trade stack. Vendors: Finastra (Trade Innovation), Surecomp (RIVO, DOKA-NG), CGI (Trade360), China Systems (Eximbills), Intellect (iGTB).
2. **Core-banking trade modules** — trade capability embedded in the core for banks that want one system: Temenos T24/Transact (TT modules), Oracle FLEXCUBE (trade finance modules), TCS BaNCS, Infosys Finacle, FIS. Trade runs where the bookings and accounting run — simpler integration, weaker standalone trade depth.
3. **SWIFT connectivity and messaging** — the MT700 family (and the ISO 20022 MX migration) in and out: SWIFT Alliance (Alliance Access/Entry), vendor-native gateways, or a dedicated messaging hub. See [iso_20022_core_processes_guide.md](iso_20022_core_processes_guide.md).
4. **Document management, imaging, and examination** — the document-centric core of trade operations: document capture (scan/OCR), archival, discrepancy checking (increasingly AI-assisted), and e-document workflows (eUCP).
5. **Compliance and screening** — sanctions/KYC screening hooks at issuance and presentation, TBML analytics on invoice/price patterns, trade-based money-laundering detection. See [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md).
6. **Client channels and portals** — corporate trade portals, APIs, and ERP integration where the client applies for LCs, uploads documents, and tracks status.
7. **Digital-trade networks and eBL platforms** — external ecosystems the bank plugs into: Bolero (eBL-as-a-Service), Contour (LC network — **status flagged**, see §4.1), ICE Digital Trade/CargoDocs, bank APIs. The network layer of [trade_finance_guide.md](trade_finance_guide.md) §7.

Architecturally the **trade processing platform is the orchestrator**: it owns the trade document of record and calls out to the core (bookings, accounting), the limits domain (contingent draws — [banking_limits_domain_guide.md](banking_limits_domain_guide.md)), SWIFT (messages), screening engines (compliance), and the payments stack (settlement — [payments_hub_guide.md](payments_hub_guide.md)). The lifecycle it orchestrates is the process view of [end_to_end_banking_processes.md](end_to_end_banking_processes.md).

## 1.2 The Market (Flagged)

**The whole of this subsection is flagged**: trade-finance-systems market sizing is published by research firms and vendors, not by any audited authority, and the numbers disagree across reports. Directional picture only:

- **Market size**: a 2026 market report puts the trade finance *software* market at ~USD 5.2B by 2031, ~3% CAGR (**flagged** — single-report estimate, not cross-verified). The ICC's ~USD 5.2T *trade finance market* figure (umbrella §12) is the product market, not the software market — do not conflate the two.
- **Installed base**: one 2025 report claims the leading third-party platforms (Finastra Fusion Trade Innovation, Surecomp RIVO, CGI Trade360, Intellect iGTB) have a *combined* installed base of **200+ financial institutions** (**flagged** — vendor-side research claim).
- **Who leads**: trade-tech directories and press consistently name **Finastra, Surecomp, CGI, China Systems, and Intellect** as the leading bank-side trade platform vendors (**flagged as directional**; no verified ranking exists — the umbrella flags the same). China Systems claims **150+ customers** worldwide (vendor claim, not audited) and has won GTR's "Best Trade Finance Software Provider" award multiple times (per its own and GTR's coverage).
- **The core-module share**: a large share of trade processing, especially outside the top-tier international banks, runs inside core-banking trade modules (Temenos, Oracle, TCS BaNCS, Finacle, FIS) rather than on standalone platforms — there is no verified split, but every vendor list includes both categories, and the mid-market overwhelmingly runs core-embedded trade.
- **The consolidation signal**: specialist vendors cluster at the edges (eBL: Bolero, ICE; commodity DLT: komgo) while the platform market has consolidated around a handful of suites — Finastra absorbed Misys's TI franchise (2017 merger), Surecomp absorbed DOKO/iTrade into the RIVO line, and the digital-trade consortia (TradeLens, we.trade, Marco Polo) failed (umbrella §7.4).

## 1.3 The Overview Table

| Category | Representative systems | Notes |
|---|---|---|
| **Trade processing platform** (system of record) | Finastra Trade Innovation (TI), Surecomp RIVO / DOKA-NG, CGI Trade360, China Systems Eximbills, Intellect iGTB | Standalone best-of-breed suites: LC/guarantee/collection origination, document checking, SWIFT messaging, lifecycle. **Share figures flagged** (§1.2) |
| **Core-banking trade module** | Temenos T24/Transact (TT), Oracle FLEXCUBE trade, TCS BaNCS trade, Finacle Trade Finance, FIS | Trade embedded where bookings/accounting run; see [core_banking_systems_guide.md](core_banking_systems_guide.md), [temenos_guide.md](temenos_guide.md), [oracle_flexcube_data_model_guide.md](oracle_flexcube_data_model_guide.md) |
| **SWIFT connectivity** | SWIFT Alliance Access/Entry, vendor-native gateways, messaging hubs | MT700 family in/out; MT760 guarantees, MT400/422 collections; ISO 20022 MX migration ([iso_20022_core_processes_guide.md](iso_20022_core_processes_guide.md)) |
| **Document / imaging / examination** | Capture, OCR, archival, discrepancy-checking (AI-assisted), eUCP e-documents | The operational cost center: ~60–70% of first presentations carry discrepancies (**flagged**, umbrella §9.2) |
| **Compliance / screening** | Sanctions list management, transaction monitoring, TBML analytics | Screening at issuance *and* presentation (vessel known only at shipment — umbrella §9.3); see [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md) |
| **Client channel** | Corporate trade portals, APIs, ERP integration | LC application, document upload, status tracking; increasingly API-first (FusionFabric.cloud-style) |
| **Digital-trade networks** | Bolero (eBL), Contour (LC network — **status flagged**), ICE Digital Trade/CargoDocs, komgo, bank APIs | Network layer of digitization ([trade_finance_guide.md](trade_finance_guide.md) §7); MLETR/ETDA legal substrate is the growth driver |
| **Data / reporting** | Trade analytics, regulatory reporting, exposure dashboards | Contingent-vs-funded exposure across the trade book; integrates the limits domain ([banking_limits_domain_guide.md](banking_limits_domain_guide.md)) |

## 1.4 The Trade-Systems Timeline

The landscape did not arrive whole; it is the sediment of four decades of bank automation. A compressed history — **verified** at the level of well-documented industry facts (vendor histories and trade-press retrospectives):

| Era | What happened | Systems consequence |
|---|---|---|
| **1970s–80s: mainframe trade** | Banks automate trade processing on mainframes; document-heavy LC/collection ledgers go electronic internally | The original trade "system of record" generations; much of this code still runs today as legacy |
| **1983–90s: SWIFT goes mainstream** | SWIFT FIN network standardizes bank messaging; MT700 family becomes the LC operational standard | The messaging layer is born; trade systems become message-generating machines ([trade_finance_guide.md](trade_finance_guide.md) §5) |
| **1990s: the standalone trade platforms** | Misys Trade Innovation (TI), China Systems Eximbills, Surecomp DOKO/iTrade, and the specialist back offices reach maturity as packaged products | The standalone-platform category — the category this guide is mostly about |
| **2000s: core vendors bolt on trade** | Temenos T24 TT modules, Oracle FLEXCUBE trade, TCS BaNCS, Finacle add trade to the core pitch | The core-embedded category; the structural fork appears |
| **2007–2010: rules modernize** | UCP 600 (2007), URDG 758 (2010) update the rule stack; systems must encode new timers and notice rules | Rule-as-code becomes a compliance feature of trade platforms |
| **2010s: digitization hype cycle** | Bolero (1998-founded) matures; the DLT consortia launch: Voltron/Contour, we.trade (2017), Marco Polo (2018), TradeLens (2018); Finastra forms (2017); Surecomp consolidates DOKO/iTrade into RIVO | The digitization layer appears — and most of the networks fail (umbrella §7.4) |
| **2020s: consolidation and legal substrate** | TradeLens shuts (2023), we.trade ends (2022), Marco Polo insolvent (2023); MLETR/UK ETDA (2023)/Singapore ETA (2021) legalize eBLs; essDOCS → ICE (2022); Bolero → WiseTech (2022); Contour → Xalts → XDC Ventures (2025, **flagged**); ISO 20022 trade migration looms | Narrow-job digitization (eBL as a service) survives; platforms race on APIs, AI document checking, and ISO 20022 readiness |

The lesson of the timeline: the **platform category is stable and consolidating** (Misys→Finastra, DOKO/iTrade→RIVO, essDOCS→ICE), while the **network category is volatile** (four consortia dead or flagged in three years). A bank architect should treat platform choices as 10-year commitments and network choices as connectors with exit clauses.

## 1.5 The Platform Economics

The deal economics of trade platforms are rarely written down, but they shape every selection. **All figures flagged as directional** — pricing is negotiated per deal, and vendors do not publish rate cards:

| Cost element | Typical shape | Notes |
|---|---|---|
| **Licence / SaaS fee** | Perpetual licence + annual maintenance (~20% of licence), or SaaS subscription per year; volume-based tiers | The trade platform is a *small* ticket next to a core programme — typically single-digit-to-low-double-digit millions of USD over a 5-year term (**flagged** as order-of-magnitude, deal-dependent) |
| **Implementation** | 1–2× the licence value, delivered by the vendor's SI partners or a big-four/regional SI | Integration to core, limits, SWIFT, screening dominates the effort — consistent with §5.3's "integration is the architecture" |
| **Migration** | Legacy data extraction, instrument-by-instrument cutover, parallel run | The most underestimated line; a trade book cannot be cut over mid-credit (§7.3 lesson 5) |
| **Run cost** | Ops headcount (the biggest line, forever), hosting, SWIFT fees, screening licences | The platform changes the ops cost curve via STP — measure *cost per presentation*, not licence cost |
| **Digitization spend** | eBL connectors, network subscriptions (Bolero-class), ISO 20022 workstream | Small absolute, strategic dependency; the §4.4 pattern |
| **Exit cost** | Data export and instrument migration out of the platform | Negotiate at entry; a system of record outlives contracts (§6.4 point 8) |

The economic punchline: **the licence is the smallest cost; the ops headcount and the integration are the largest; the digitization is the cheapest lever with the biggest strategic payoff.** That ordering is why this guide keeps scoring operational depth (§6.2), not brochure features.

---

# 2. The Vendor Profiles

**Verification note**: the six vendors profiled below are the trade-systems vendors named in the task brief, each **verified** against vendor sites and industry press in August 2026. Product names are given as marketed; the umbrella guide's caveat ("market-share figures not verified; rankings directional") applies throughout. Vendor lists in trade press also commonly include **Surecomp** and **China Systems** (profiled in the comparison section) and **FIS/IBSFINtech** at the specialist edges.

## 2.1 The Finastra Profile: Trade Innovation

**The company.** Finastra is the London-headquartered banking software group formed in 2017 from the merger of **Misys** and **D+H** — the largest pure-play banking technology vendor by revenue. Its trade franchise is the direct descendant of **Misys Trade Innovation** (the long-standing "TI" platform), which makes it the most widely cited bank trade-processing suite in the industry press.

**The product.** **Trade Innovation** (part of the **Finastra Fusion** portfolio; the platform was long branded **TI / TI Plus** — the umbrella's phrasing, retained here) is a trade and supply chain finance platform for banks. Per Finastra's own solution literature (verified via finastra.com):

- **Traditional trade**: import/export LC origination and processing, documentary collections, guarantees, amendments, and drawings.
- **Document-centric workflow**: document checking, discrepancy management, and the full presentation lifecycle.
- **SWIFT messaging**: MT700-family generation/parsing, with the ISO 20022 migration path.
- **Digital channels**: API layer over **FusionFabric.cloud** (Finastra's open-banking platform) — corporate self-service LC applications, ERP integration, and fintech ecosystem connectivity (verified: Finastra markets open APIs to a fintech ecosystem via FusionFabric.cloud).
- **Supply chain finance**: partnerships extend the suite — e.g. the **CredAble** SCF integration announced via press release (verified) — reflecting the SCF adjacency of [supply_chain_finance_guide.md](supply_chain_finance_guide.md).

**The position.** Trade Innovation is the reference "standalone platform" in this market: the trade system of record for banks that do *not* want trade inside the core. Its natural competitors are Surecomp RIVO and CGI Trade360 at platform level, and the core vendors (Temenos, Oracle) when the argument is "keep trade where the core is." Client list specifics were not independently verified in this session (**flagged**: no audited client roster retrieved; Finastra publishes named-bank case studies on its site).

**Clients and proof points.** Finastra's trade franchise inherited the Misys TI installed base — a population that, per industry reporting, spans large international and regional banks across Europe, Asia, and the Americas (specific bank names beyond press-covered deals are **flagged as unverified** this session). The credible proof points verified here are structural: (a) the TI franchise's multi-decade survival through two ownership changes (Misys → Finastra) implies a large, renewing installed base; (b) the CredAble partnership shows the SCF expansion path; (c) the FusionFabric.cloud API layer is the most openly marketed API ecosystem among the standalone trade vendors. For a procurement, the reference calls that matter are: a commodity-trade bank (Cymbal Bank-adjacent), a mid-size Asian trade bank, and a European mid-market bank — ask Finastra for those three profiles specifically, and verify them on calls rather than from the marketing site.

### The Finastra Table

| Aspect | Detail |
|---|---|
| **Vendor** | Finastra (Misys + D+H merger, 2017); HQ London |
| **Product** | Trade Innovation (Fusion portfolio; heritage branding TI / TI Plus) |
| **Focus** | Standalone bank trade & SCF platform: LC, guarantees, collections, documents, SWIFT, digital channels |
| **Notes** | API layer via FusionFabric.cloud; SCF partnerships (CredAble — verified); most widely cited suite in press (**share flagged**); ISO 20022 path |

## 2.2 The Oracle Profile: FLEXCUBE Trade

**The company.** Oracle Financial Services (OFS) is a top-tier core-banking vendor; **Oracle FLEXCUBE Universal Banking (FCUBS)** is its flagship core, deployed across hundreds of banks worldwide, with particular strength in the Asia-Pacific, Middle East, and Africa mid/large-bank segment.

**The product.** **FLEXCUBE Trade Finance** is the trade module family of FCUBS. **Verified** via Oracle's own training and documentation (Oracle University, FCUBS 12.0.3): the trade finance modules include **letters of credit (import and export), bank guarantees, bills and collections, and trade origination**. Client-side, **FLEXCUBE Direct Banking Corporate** provides the corporate trade channel (LC initiation, status, document upload). The module family covers:

- **Import/export LC** — issuance, advice, amendments, transfers, and settlement, with UCP 600-aligned processing.
- **Guarantees** — issuance and lifecycle (URDG-aligned), contingent exposure booking.
- **Bills and collections** — documentary collections (D/P, D/A) per URC 522.
- **Trade origination** — the front-end origination flow within the core.
- **Messaging** — SWIFT MT700-family generation within the core's messaging framework; data-model detail in [oracle_flexcube_data_model_guide.md](oracle_flexcube_data_model_guide.md).

**The position.** FLEXCUBE trade is the canonical **core-embedded trade module**: trade books to the same general ledger, limits, and customer master as everything else. It wins where the bank wants one integrated core and where trade volumes are moderate-to-high but not "trade-factory" scale. Its strategic answer to the standalone platforms is integration cost: no separate system of record, no reconciliation between trade platform and core. Trade depth beyond vanilla LC/guarantee/collection (heavy SCF, eBL orchestration, AI document checking) is where standalone platforms argue they lead.

**Clients and proof points.** FLEXCUBE's global core footprint is well documented (hundreds of bank deployments, strongest in Asia-Pacific, the Middle East, and Africa — the *core* footprint is industry-documented; the *trade-module* take-up within that base is less visible publicly and **flagged as not independently audited** this session). The Oracle University course catalogue itself (verified) is evidence of the module family's maturity: dedicated FLEXCUBE trade finance training covering LC, guarantees, bills & collections, and origination implies a sustained institutional user base. Structurally, FLEXCUBE trade is the strongest "single-vendor core-plus-trade" argument in the market — if the bank's core is FLEXCUBE and trade is not a strategic differentiator, the module is the low-integration-risk answer; the standalone platforms must beat it on depth, and the data-model view in [oracle_flexcube_data_model_guide.md](oracle_flexcube_data_model_guide.md) shows exactly what the bank gets.

### The Oracle Table

| Aspect | Detail |
|---|---|
| **Vendor** | Oracle Financial Services (OFS) |
| **Product** | FLEXCUBE Universal Banking (FCUBS) — Trade Finance modules (12.x); FLEXCUBE Direct Banking Corporate for the channel |
| **Focus** | Core-embedded trade: import/export LC, guarantees, bills & collections, trade origination (verified per Oracle docs) |
| **Notes** | Trade where the core is: same GL/limits/customer master; strong Asia/MEA footprint; see [oracle_flexcube_data_model_guide.md](oracle_flexcube_data_model_guide.md) |

## 2.3 The Temenos Profile: T24 Trade

**The company.** Temenos is the Swiss core-banking vendor; **T24** (the long-standing product name, rebranded **Temenos Transact** in 2019) is its flagship core, and the trade franchise is one of its most deployed product families. **Verified** in this session: Temenos publishes a dedicated "Temenos Trade Finance" product paper (May 2020) covering LC-related processing including reimbursement, and press records show banks going live on T24 trade (e.g. Banco de la Nación Argentina upgrading its New York branch to run trade finance and payments on T24 — GTR, verified).

**The product.** The T24/Transact **trade modules (the TT family)** provide:

- **Letters of credit** — import and export, including the reimbursement processing under LC-related transactions called out in Temenos's own paper.
- **Guarantees** — issuance, lifecycle, and contingent-limit booking.
- **Collections and bills** — documentary collections processing.
- **Messaging and integration** — SWIFT connectivity within the Temenos stack, with the Temenos Integration Framework; the data model is documented in [temenos_data_model_guide.md](temenos_data_model_guide.md) and [t24_programming_guide.md](t24_programming_guide.md).
- **Modern delivery** — Temenos now sells trade on the **Temenos platform** (SaaS/cloud) with digital channels (Infinity-class front ends) and API exposure; T24 remains the processing engine underneath.

**The position.** Like FLEXCUBE, T24 trade is the **core-embedded** answer — the umbrella's "core-banking trade modules (Temenos, Oracle, FIS, Silverlake)" tier. It is strong in the mid/large-bank segment worldwide and in Islamic trade products (Murabaha-structured trade — a Temenos differentiator per press coverage; **flagged lightly** as product-name detail varies by release). For a bank already on T24, the TT modules are the default choice; the standalone-platform argument applies when trade needs document-factory depth the core modules don't carry.

**Clients and proof points.** Verified in this session: **Banco de la Nación Argentina (BNA)** upgraded its New York branch onto T24 to run trade finance and payments from a single platform (GTR coverage) — a useful data point for the *cross-border branch trade* pattern. Temenos's marketing additionally cites trade deployments across its large global T24 installed base, but the specific trade-module client list was not independently audited here (**flagged**). What is structurally verifiable: T24's TT modules have shipped for decades within the T24 core (the most deployed core of its class), so the *capability* is mature even where *named trade references* are press-thin; a procurement should specifically ask Temenos for recent trade go-lives (last 3 years, with STP and volume numbers), because the module's maturity is not in doubt but its *modern delivery* (SaaS, APIs, eBL integration) is what a 2026 selection actually tests.

### The Temenos Table

| Aspect | Detail |
|---|---|
| **Vendor** | Temenos (Switzerland) |
| **Product** | T24 / Temenos Transact — Trade Finance (TT) modules; Temenos Trade Finance paper (2020, verified) |
| **Focus** | Core-embedded trade: import/export LC (incl. reimbursement), guarantees, collections; Islamic trade variants |
| **Notes** | Trade where the core is; SaaS/cloud delivery on the Temenos platform; see [temenos_guide.md](temenos_guide.md), [temenos_data_model_guide.md](temenos_data_model_guide.md) |

## 2.4 The Intellect Profile: iGTB

**The company.** **Intellect Design Arena** is the Chennai-headquartered banking technology group (listed on Indian exchanges); **iGTB — Intellect Global Transaction Banking** — is its transaction-banking arm, one of the largest dedicated transaction-banking product houses. **Verified**: Aite (analyst research) profiled "iGTB Trade & Supply Chain Finance" among vendors to watch in its trade finance software market report (per Intellect's own awards page), and iGTB has delivered supply chain finance platforms for major banks — e.g. **Bank of Baroda**'s digital SCF platform (press coverage, verified). iGTB also partners with Microsoft on cloud delivery for corporate banking (verified).

**The product.** The iGTB trade & SCF suite sits inside the broader iGTB transaction-banking platform (payments, cash management, trade, SCF). Trade products include LC processing, guarantees, collections, and the SCF family (receivables/payables finance — the product domain of [supply_chain_finance_guide.md](supply_chain_finance_guide.md)). The umbrella flags that **exact current product names vary by release** (e.g. the eTrade/eSCF family names) — retained here: **flagged**, product naming should be confirmed against current iGTB materials before procurement use.

**The position.** iGTB is the strongest of the "transaction-banking suite" players: it wins where the bank is buying a *corporate banking platform* (payments + cash + trade + SCF together) rather than a trade-only system — a common pattern in India, the Middle East, and Africa. Its trade depth is real but its pitch is the suite; against Finastra/Surecomp it competes on breadth and regional delivery rather than trade-document depth.

**Clients and proof points.** The strongest verified reference is **Bank of Baroda** — iGTB implemented a comprehensive supply chain finance platform for the bank (industry press, verified) — evidence that the suite wins large corporate-banking deals in India. The Aite "vendor to watch" mention (per Intellect's own site) and the Microsoft Cloud for Financial Services partnership add credibility signals. For a Singapore/ASEAN reader: iGTB's franchise is India-centric, with regional expansion into the Middle East and Africa; **ASEAN-specific trade references were not verified this session (flagged)** — a procurement in this region should ask for regional proof points rather than assume them.

### The Intellect Table

| Aspect | Detail |
|---|---|
| **Vendor** | Intellect Design Arena (India); iGTB = Intellect Global Transaction Banking arm |
| **Product** | iGTB Trade & Supply Chain Finance (eTrade-family names vary by release — **flagged**) |
| **Focus** | Transaction-banking suite: trade + SCF + cash/payments together; strong in India/MEA/Asia |
| **Notes** | Aite "vendor to watch" (per Intellect); Bank of Baroda SCF implementation (verified); Microsoft cloud partnership |

## 2.5 The EdgeVerve Profile: Finacle Trade

**The company.** **EdgeVerve Systems** is the product subsidiary of **Infosys**; **Finacle** is its banking product franchise (core banking, digital, payments, trade). Finacle is one of the world's most deployed core platforms, particularly across India, Africa, the Middle East, and Southeast Asia.

**The product.** The **Finacle Trade Finance Solution Suite** (verified via finacle.com) is a **componentized** suite covering the full trade lifecycle: LCs (import/export), guarantees, collections, open account, and supply chain finance, plus digital capabilities to automate the end-to-end lifecycle. It also includes **Finacle Trade Connect** — a **blockchain-based** solution for digitizing and automating inter-organization trade finance on a shared network, announced for global availability in **November 2017** (verified via EdgeVerve press release) — an early bank-side DLT trade play, distinct from the failed consortia because it sits inside a vendor suite rather than a standalone network.

**The position.** Finacle trade is the **core-embedded/suite** answer for the Finacle installed base: banks already on Finacle core (a very large population in Asia/MEA) take trade as componentized modules rather than a separate platform. Its Trade Connect blockchain layer gives it a digitization story in-house. Against the standalone platforms it competes on installed-base leverage and price; its trade-document depth (discrepancy checking, eBL orchestration) is less often cited in press than Finastra/Surecomp's.

**Clients and proof points.** Finacle's trade suite inherits the Finacle core's enormous Asia/MEA installed base (Finacle is one of the world's most deployed cores, with banks across India, Africa, the Middle East, and Southeast Asia — the core's footprint itself is well documented; *trade-suite-specific* client names beyond the core base were **not verified this session, flagged**). Trade Connect's 2017 launch (verified press release) predates most bank DLT trade efforts and gives Finacle a differentiated, if under-publicized, digitization story. For a selection: Finacle is the default candidate whenever the bank's core is Finacle — the alternative (standalone platform) must then justify a second system of record and its reconciliation burden against the suite's componentized coverage.

### The EdgeVerve Table

| Aspect | Detail |
|---|---|
| **Vendor** | EdgeVerve Systems (product subsidiary of Infosys) |
| **Product** | Finacle Trade Finance Solution Suite (componentized); Finacle Trade Connect (blockchain, Nov 2017 — verified) |
| **Focus** | Core-embedded/suite trade for the Finacle installed base: LC, guarantees, collections, open account, SCF |
| **Notes** | Large Asia/MEA footprint via Finacle core; in-suite DLT digitization play |

## 2.6 The TCS Profile: BaNCS Trade

**The company.** **Tata Consultancy Services (TCS)** is India's largest IT services firm; **TCS BaNCS** is its banking product platform, marketed as the **TCS BaNCS Global Banking Platform (GBP)** — an **API-first** core-and-beyond platform (**verified**: TCS markets BaNCS as API-first on the Microsoft Azure marketplace listing).

**The product.** BaNCS trade finance is the trade component of the platform: **LCs (import/export), guarantees, collections, and supply chain finance**, integrated with the core's payments, limits, and AML/KYC capabilities. **Verified** in this session: Janata Sahakari Bank (Pune, a 71-branch cooperative bank) selected TCS BaNCS GBP in 2025 to modernize core banking **and trade finance** plus payments and AML/KYC in a single digital ecosystem (press coverage, multiple outlets). BaNCS also has a long record in corporate/transaction banking deployments, especially in India and the Middle East.

**The position.** Like Finacle, BaNCS is the **core-embedded/suite** answer for the TCS installed base: the "one platform, everything including trade" pitch, API-first for digital channels. It is a serious contender in the mid-market and in markets where Indian-origin platforms have distribution (India, Middle East, Africa, parts of Asia-Pacific). Against Finastra/Surecomp/CGI it competes on platform breadth and services muscle (TCS's SI scale) rather than trade-specific depth.

**Clients and proof points.** The verified 2025 **Janata Sahakari Bank (JSBL)** engagement is a clean reference: a 71-branch cooperative bank selecting BaNCS GBP for core + trade finance + payments + AML/KYC modernization in one ecosystem — precisely the "suite instead of point solutions" profile BaNCS targets. TCS's broader BaNCS franchise spans large corporate-banking deployments in India and the Middle East (well documented at the platform level; **trade-component-specific references beyond JSBL were not verified this session, flagged**). The distinguishing feature for a selection: TCS brings SI-scale implementation muscle — for a bank that wants one partner to deliver core, trade, and integration, BaNCS packages the whole programme; for a bank that wants best-of-breed trade depth, that same packaging is the constraint.

### The TCS Table

| Aspect | Detail |
|---|---|
| **Vendor** | Tata Consultancy Services (TCS) |
| **Product** | TCS BaNCS Global Banking Platform — trade finance component |
| **Focus** | Core-embedded/suite trade: LC, guarantees, collections, SCF; API-first platform |
| **Notes** | Janata Sahakari Bank core+trade modernization (2025, verified); strong India/MEA distribution; TCS services muscle |

## 2.7 The Vendor Table

| Vendor | Product | Focus | Notes |
|---|---|---|---|
| **Finastra** | Trade Innovation (TI / TI Plus heritage; Fusion portfolio) | Standalone bank trade & SCF platform — LC, guarantees, collections, documents, SWIFT, digital channels | Most widely cited suite (**share flagged**); FusionFabric.cloud APIs; SCF partnerships (CredAble) |
| **Oracle (OFS)** | FLEXCUBE Universal Banking — Trade Finance modules (12.x) | Core-embedded trade: import/export LC, guarantees, bills & collections, trade origination | Verified per Oracle docs; strong Asia/MEA; channel via FLEXCUBE Direct Banking |
| **Temenos** | T24 / Transact — Trade Finance (TT) modules | Core-embedded trade incl. LC reimbursement, guarantees, collections; Islamic variants | Temenos Trade Finance paper (2020); SaaS delivery; see [temenos_guide.md](temenos_guide.md) |
| **Intellect (iGTB)** | iGTB Trade & Supply Chain Finance | Transaction-banking suite: trade + SCF + cash/payments | Names vary by release (**flagged**); Bank of Baroda SCF; Aite mention |
| **EdgeVerve / Infosys** | Finacle Trade Finance Suite + Trade Connect | Core-embedded/suite trade for Finacle installed base | Componentized; blockchain Trade Connect (2017); big Asia/MEA base |
| **TCS** | BaNCS Global Banking Platform — trade | Core-embedded/suite trade; API-first platform | Janata Sahakari Bank 2025; India/MEA strength |
| *(Adjacent specialists)* | Surecomp RIVO / DOKA-NG; CGI Trade360; China Systems Eximbills; FIS; IBSFINtech; komgo | Platform or specialist trade/SCF/commodity-DLT | Profiled in [Section 3](#3-the-platform-comparison) and [Section 4](#4-the-digitization-platforms) |

## 2.8 The Adjacent Specialists

The six profiled vendors are the task-brief set; the shortlists in industry press routinely add four more families that a real procurement will meet. Brief profiles (**verified** at the level cited):

**Surecomp** — the Israel-headquartered trade software specialist (offices globally), arguably Finastra's closest standalone-platform rival. **Verified**: current product line is **RIVO** (the collaborative trade finance platform — DZ Bank adopted it in 2024; Investec South Africa commented on it) and **DOKA-NG** (the back-office processing engine — BTPN Indonesia deployed DOKA). The DOKO/iTrade heritage names from the umbrella's §8.2 are the predecessors absorbed into the RIVO line. Surecomp is SWIFT-certified, sells cloud and on-prem, and positions on document-centric workflows and client collaboration rather than platform breadth.

**CGI** — the Canadian IT services group's **Trade360** suite. **Verified**: trade + open account + supply chain finance in one platform; automates LC, guarantees, and compliance checking; SaaS-delivered; used by **ANZ** and **ASB** (New Zealand) among others. Trade360 is the "quiet global back office" — strong conformance, less marketing noise than Finastra.

**China Systems** — the "Eximbills" house (HQ London, Chinese-origin founders, offices worldwide). **Verified**: Eximbills is the trade back office automating the full trade cycle in line with SWIFT, UCP, and ISO 20022; the company claims **150+ customers** (vendor claim) and multiple GTR "Best Trade Finance Software Provider" awards (press-corroborated). The umbrella's §8.2 did not name China Systems; this guide adds it because every bank-market shortlist directory in this session's research (TechVendorIndex-class) includes it.

**FIS and IBSFINtech** — the edges. **FIS** (the US fintech giant) carries trade capability in its banking portfolio (e.g. within its commercial/corporate banking lines) but trade is not its flagship pitch. **IBSFINtech** is the Indian specialist (trade, SCF, treasury for corporates and banks). **komgo** is the commodity-trade DLT platform (trader/bank-backed) — the digital-network edge, adjacent to a Cymbal Bank commodity book; **flagged**: current activity level not verified this session.

The specialist lesson: the platform market is a **long tail with a short head** — Finastra/Surecomp/CGI/China Systems dominate the standalone shortlists, but regional specialists (IBSFINtech in India, local SIs everywhere) win the deals that never reach the press.

---

# 3. The Platform Comparison

## 3.1 The Comparison Table

The comparison below covers the platforms named in the vendor profiles plus the adjacent specialists (Surecomp, CGI, China Systems) that every trade-platform shortlist in the industry press includes. **All "typical bank profile" and "strength" rows are directional** — synthesized from vendor positioning and press coverage, not from audited benchmarks.

| Dimension | Finastra Trade Innovation | Surecomp (RIVO / DOKA-NG) | CGI Trade360 | China Systems Eximbills | Intellect iGTB | Core modules (Temenos / Oracle / Finacle / BaNCS) |
|---|---|---|---|---|---|---|
| **Product type** | Standalone trade & SCF platform | Standalone platform + back office | Standalone suite | Trade back office (standalone) | Transaction-banking suite | Trade embedded in core |
| **Document/LC depth** | High — document checking, discrepancies, presentations | High — document-centric heritage (DOKO/iTrade lineage) | High — LC, guarantees, compliance checking (verified) | High — "industry standard back office" per GTR (verified) | Medium-high — trade within suite | Medium — vanilla LC/guarantee/collection |
| **Channel/digital** | API-first, FusionFabric.cloud ecosystem | RIVO collaborative client portal; bank-integratable | Digital SaaS, 24×7 global processing (verified) | Front-end solutions offered | Suite-wide corporate channels | Core vendor channels (Direct Banking, Infinity) |
| **Cloud/SaaS** | Yes | Yes (cloud and on-prem — verified) | Yes (SaaS — verified, ASB first use) | Yes/on-prem | Yes (Microsoft Cloud partnership) | Temenos SaaS; Oracle/Finacle/BaNCS cloud paths |
| **SWIFT/ISO 20022** | Yes, MT700 family + MX path | SWIFT-certified (verified) | Yes | Yes — SWIFT, UCP, ISO 20022 (verified) | Yes | Yes, via core messaging |
| **SCF breadth** | Yes (CredAble partnership) | Yes | Yes — trade + open account + SCF (verified) | Yes — trade & SCF vendor (verified) | Yes — SCF flagship (BoB deal) | Limited-to-yes by vendor |
| **Digital-network integration** | eBL/network connectors | eBL connectors | eBL connectors | eBL connectors | eBL/network connectors | Emerging |
| **Typical bank profile** | Large international / trade-factory banks | Mid/large banks, document-heavy ops | Mid/large banks (ANZ, ASB — verified) | Global trade back offices (150+ customers, vendor claim) | Corporate-banking suite buyers (India/MEA) | Core-centric mid-market |
| **Key risk** | Suite complexity/cost | Niche scale | SI-led delivery | Niche scale | Product-name churn (**flagged**) | Trade depth ceiling |

## 3.2 The Comparison Deep-Dive

The table compresses five genuinely different buying positions:

**1. Standalone platform vs core-embedded module — the structural fork.** The industry's deepest divide is not vendor-vs-vendor but *architecture*: Finastra/Surecomp/CGI/China Systems are **standalone systems of record**; Temenos/Oracle/Finacle/BaNCS keep trade **inside the core**. A bank choosing "standalone" buys trade depth (document examination, discrepancy workflow, guarantee lifecycle, eBL orchestration) and pays integration cost (limits, GL, customer master, payments — every booking crosses a boundary, see [Section 5](#5-the-architecture)). A bank choosing "core-embedded" buys a single ledger and master, and accepts that exotic trade workflow is constrained by the core's roadmap. This is the same fork the umbrella draws in [trade_finance_guide.md](trade_finance_guide.md) §8.2 — "the core is not the trade system" for Cymbal Banks — and it is the first question of any selection.

**2. Finastra vs Surecomp — the platform fight.** These two are the most commonly named standalone rivals in the press. Finastra Trade Innovation wins on **breadth and ecosystem** (FusionFabric.cloud APIs, SCF partnerships, the Misys TI installed base and brand). Surecomp wins on **document-centric focus and collaboration**: RIVO is explicitly a *collaborative* platform (client-facing, verified with DZ Bank's 2024 adoption and Investec South Africa commentary), and DOKA-NG is the battle-tested back office (BTPN Indonesia, verified). Rule of thumb: a bank wanting a digital client-facing trade journey plus deep back office shortlists both; Finastra tends to win where the bank wants the API ecosystem, Surecomp where the bank wants the document engine and collaborative workflows.

**3. The suites vs the specialists.** CGI Trade360 (ANZ, ASB — verified clients) and China Systems Eximbills (150+ customers, GTR-awarded, verified) are the "global back office" players — less marketing noise, deep installed bases, strong SWIFT/UCP/ISO 20022 conformance. They are routinely on shortlists for banks that process large LC/collection volumes and care more about conformance and throughput than channel glitz. Intellect iGTB is the **suite player**: banks buying a full corporate-banking platform (payments + cash + trade + SCF) get trade as a component — dominant pattern in India/MEA, where the alternative (a trade-only platform plus separate cash/payments) is structurally less attractive.

**4. The core-module trade-off.** Temenos TT, FLEXCUBE trade, Finacle, and BaNCS each process the full vanilla trade lifecycle competently (verified module lists in [Section 2](#2-the-vendor-profiles)); the question is the *ceiling*. Standalone-platform advocates argue: AI document checking, eBL orchestration, network integration (Contour/Bolero-class), and ISO 20022 trade (TSMT) depth arrive first on standalone platforms, and core trade modules track the core's release cadence. Core advocates answer: the reconciliation between a trade platform and the core's GL/limits is where operational risk actually lives — one system, one truth. Both are right about different banks: high-volume trade factories and commodity houses (the Cymbal Bank profile) skew standalone; universal mid-market banks skew core-embedded.

**5. The digitization axis.** Every platform now claims eBL/network integration; what differs is maturity. Finastra and Surecomp ship production eBL connectors (Bolero-class); Finacle has its own blockchain layer (Trade Connect); the core vendors are emerging. The digitization platforms themselves are the subject of [Section 4](#4-the-digitization-platforms) — the pattern is: **the trade platform owns the document of record; the eBL/network platform owns the digital document journey; the bank owns the integration.**

## 3.3 The Scenario-to-Platform Map

Which platform wins which engagement — the pragmatic mapping, **directional** (synthesized from vendor positioning and verified deals, not from audited win-rates):

| The bank's situation | The structural answer | The shortlist | Why |
|---|---|---|---|
| Large international bank, trade is strategic (commodity, confirming, trade factory) | Standalone platform | Finastra TI; Surecomp RIVO/DOKA; CGI Trade360 | Depth and digitization ahead of the core's roadmap; the Cymbal Bank answer (§7) |
| Mid-size bank, document-heavy ops, wants collaboration with clients | Standalone platform, collaborative front | Surecomp RIVO first, Finastra second | RIVO's client-facing collaboration is its verified pitch (DZ Bank, Investec) |
| Bank already on Temenos/Oracle core, vanilla trade book | Core-embedded module | Temenos TT; FLEXCUBE trade | One system of record, no reconciliation; the low-integration-risk answer |
| Bank on Finacle or BaNCS (or buying a platform from TCS/Infosys) | Suite component | Finacle Trade Suite; TCS BaNCS trade | Installed-base leverage; JSBL-style one-platform programmes |
| Corporate-banking suite purchase (payments + cash + trade + SCF) | Transaction-banking suite | Intellect iGTB | Breadth over depth; the BoB pattern |
| Bank buying "global back office" conformance and throughput | Specialist back office | China Systems Eximbills; CGI Trade360 | SWIFT/UCP/ISO 20022 conformance at scale (verified strengths) |
| Bank with heavy commodity-document flows, wants eBL-first | Standalone + eBL layer | Finastra/Surecomp + Bolero | eBL where the book moves; the digitization pattern of §4.4 |

The map collapses to one line: **strategic trade buys a standalone platform; tactical trade stays in the core; suites win when the bank buys the whole corporate-banking shelf.**

---

# 4. The Digitization Platforms

The digitization layer of the trade landscape is where the platforms themselves are not banks' systems but *networks and eBL utilities*. The umbrella ([trade_finance_guide.md](trade_finance_guide.md) §7) covers the failed consortium era (TradeLens, we.trade, Marco Polo) and the surviving narrow-job players (Bolero, ICE CargoDocs); this section profiles the two named in the brief — Contour (**status flagged**) and Bolero.

## 4.1 The Contour Profile (Status Flagged)

**What it was.** Contour is the DLT-based **LC digitization network** — the direct descendant of the **Voltron** pilot, launched as Contour in 2020 on **R3 Corda**, backed by a consortium of systemically important banks (nine backers; Citi, HSBC, Standard Chartered, BNP Paribas and others were publicly associated with it). Its pitch: digitize the LC workflow — application, issuance, amendments, and presentation — on a shared network, replacing paper/email chains between banks, corporates, and their counterparties.

**The status — flagged, and volatile.** Verified timeline from industry press (GTR, Ledger Insights, Trade Finance Global):

1. Contour **announced it would shut down** in 2025 after failing to secure further funding from its bank backers (Ledger Insights/GTR coverage, verified).
2. Singapore-based fintech **Xalts announced the acquisition of Contour Network** (Trade Finance Global coverage, 2025) — a "lifeline" for digital trade finance.
3. **XDC Ventures — the investment arm of the XDC Network blockchain — acquired Contour from Xalts** (GTR, 29 October 2025) and plans to "re-energise" the platform, including a "**Stable-Coin Lab**" initiative bridging bank systems with blockchain settlement (Contour's own site, verified).

**Why it matters as a case study.** Contour's trajectory is the consortium paradox in miniature: strong bank backing, real technology, and a genuinely useful LC digitization workflow — yet it could not fund itself as a standalone network. The umbrella's §7.4 lesson applies: digitization survives as a *monetizable narrow job* (eBL utilities), not as a membership network whose value grows only with adoption. **Do not cite Contour as a stable, active production network without re-verifying its current state** — as of this writing (August 2026) it is in a post-acquisition relaunch whose operating status is uncertain (**flagged**).

## 4.2 The Bolero Profile

**What it is.** **Bolero International** is the London-based eBL/e-document platform — the oldest and most established of the trade digitization utilities (originally SWIFT-backed at its 1998 founding). **Verified**: Bolero is now part of the **WiseTech Global Group** (acquired 2022), and its current product line (per Bolero's own materials and business databases) is:

- **Bolero eBL-as-a-Service** — electronic bills of lading embedded into banks', carriers', and forwarders' platforms via APIs, covering issuance through surrender; the "narrow job" that survived the consortium era.
- **Galileo Multi-Bank platform** — a multi-bank trade finance collaboration platform (LCs, guarantees, collections, ePresentation) that banks offer to their corporate clients under their own brand.
- **Bolero Title Registry** — the application on the Bolero messaging platform that records the current holder of an eBL and only allows the holder to update it — the digital analogue of the paper bill of lading's possession logic (this is what makes eBL *title transfer* work legally).

**The position.** Bolero is the reference **operating** digitization platform: not a consortium (no member-network economics to fail), but a SaaS utility monetizing eBL and e-presentation per transaction/per bank. Its growth driver is the legal substrate — MLETR-style regimes (UK 2023, Singapore ETA 2021) give electronic records the same standing as paper (umbrella §7.5) — and its integration surface is the bank's trade platform (the vendor connectors in §3.1). **Verified as operating** as of this writing; commercial details (pricing, live-bank counts) were not independently audited here (**flagged** if cited).

## 4.3 The Digitization Table

| Platform | Model | Status (as of Aug 2026) | Notes |
|---|---|---|---|
| **Contour** | DLT (R3 Corda) LC network, bank-backed | **Flagged: volatile** — announced closure 2025; acquired by Xalts; then by XDC Ventures (Oct 2025) for relaunch ("Stable-Coin Lab") | Ex-Voltron; the consortium paradox case study; re-verify before citing as active |
| **Bolero** | SaaS eBL/e-document utility; eBL-as-a-Service, Galileo multi-bank, Title Registry | **Operating** (WiseTech Global Group, acquired 2022) | Oldest eBL platform (founded 1998, SWIFT origins); MLETR-driven growth |
| **ICE Digital Trade (CargoDocs/essDOCS)** | eBL/e-document SaaS (acquired by ICE, 2022) | Operating | Umbrella §7; commodity-documentation strength |
| **komgo** | Commodity-trade DLT platform (trader/bank-backed) | Operating (niche) — **flagged**: current activity level unverified this session | Adjacent to commodity trade finance (Cymbal Bank-relevant) |
| **TradeLens** | Maersk–IBM blockchain platform | **Shut down** Q1 2023 | Umbrella §7.3 — "not commercially viable" |
| **we.trade** | European bank consortium (Corda) | **Discontinued** May 2022 | Umbrella §7.3 |
| **Marco Polo** | R3 Corda trade finance network | **Insolvency proceedings reported** 2023 | Umbrella §7.3 — press-reported |

## 4.4 The Digitization Integration Pattern

How an eBL platform actually plugs into the trade stack — the pattern every vendor connector implements (and the worked example in §7 uses for its pilot):

1. **Issuance side** — the corporate (or its forwarder) requests an eBL through the carrier's or the bank's portal; the eBL platform issues the electronic document and records the initial holder in its **title registry** (Bolero's Title Registry model: only the current holder can endorse/transfer — the digital analogue of possession of the paper bill).
2. **Under the LC** — the eBL is presented to the bank's trade platform as part of the document set: the platform ingests it via the eBL connector (API), treats it as the bill of lading for examination purposes (same UCP 600 face-of-document rules — the eUCP layer governs electronic records, umbrella §7.5), and records the presentation.
3. **Endorsement chain** — title moves holder-to-holder (exporter → negotiating bank → issuing bank → importer) through registry transfers, each a verifiable event — replacing the physical endorsement and courier leg of the umbrella's §9.2 step 12.
4. **Surrender and delivery** — on payment/acceptance, the registry transfers title to the buyer (or the buyer's agent), and the carrier releases the goods against the registry record — no paper B/L needed at destination.
5. **The bank's integration surface** — the trade platform's eBL connector handles: eBL creation requests, presentation intake, status events (issued/endorsed/surrendered), and the audit trail back to the platform's message log. The legal substrate (Singapore ETA 2021, MLETR regimes) is what makes the registry record legally equivalent to the paper document — without it, the eBL is just a PDF (umbrella §7.5).

The architectural consequence: **the eBL platform is a system of record for *documents* that the trade platform treats as one more input source** — the trade platform stays the orchestrator; the digitization layer adds a registry-driven document journey alongside the paper one. This is why the §3.1 comparison scores eBL connectors as a differentiator: a vendor with production Bolero-class connectors ships this pattern today; a vendor with a roadmap ships it next year.

---

# 5. The Architecture

## 5.1 The Trade Stack Layers

The umbrella's three-layer view (channel / processing / core-integration — [trade_finance_guide.md](trade_finance_guide.md) §8.1) expands into **six layers** when you account for messaging, compliance, and data. The verified principle is unchanged: **the trade processing platform is the orchestrator** — it owns the trade document of record (the credit, the guarantee, the collection) and calls out to everything else.

1. **Channel layer** — corporate trade portals, APIs, ERP integration; where the client applies, uploads documents, tracks status. Increasingly API-first (FusionFabric.cloud-class) and white-labeled.
2. **Origination layer** — LC/guarantee/collection application, credit and limit checks (contingent exposure), pricing, and commitment. Feeds the limits domain ([banking_limits_domain_guide.md](banking_limits_domain_guide.md)).
3. **Processing layer (system of record)** — the trade platform's core: issuance, amendments, document checking (AI-assisted), discrepancy management, presentation handling, guarantee lifecycle, drawings/settlements, rule timers (UCP 600's 5-banking-day clock, Art 16 single notice).
4. **Messaging layer** — SWIFT connectivity (Alliance-class gateway or vendor-native): MT700 family, MT760 guarantees, MT400/422 collections, and the ISO 20022 MX migration path ([iso_20022_core_processes_guide.md](iso_20022_core_processes_guide.md)).
5. **Integration layer** — the boundaries to the core (bookings, accounting — [core_banking_systems_guide.md](core_banking_systems_guide.md)), limits, payments hub ([payments_hub_guide.md](payments_hub_guide.md)), screening engines, and digital-trade networks (Bolero eBL, network APIs). Synchronous APIs for booking; events/queues for settlement and notification ([../technology/message_queue_data_loss_guide.md](../technology/message_queue_data_loss_guide.md)).
6. **Data and reporting layer** — trade exposure analytics (contingent vs funded), regulatory reporting, reconciliation, and the audit trail that documentary compliance requires.

## 5.2 The Architecture Table

| Layer | Components | Notes |
|---|---|---|
| **Channel** | Corporate trade portal, mobile, APIs, ERP connectors (SAP/Oracle), white-label client journeys | API-first trend; document upload and status tracking; see also [distributed_auth_guide.md](../technology/distributed_auth_guide.md) for identity substrate |
| **Origination** | LC/guarantee application flows, credit & limit checks, pricing, fees, commitment booking | Contingent limit booked at issuance, funded at honouring — [banking_limits_domain_guide.md](banking_limits_domain_guide.md) |
| **Processing (system of record)** | Trade platform: issuance/amendment/drawing workflows, document checking (OCR/AI), discrepancies, presentations, rule timers, guarantee lifecycle | The orchestrator; e.g. Finastra TI, Surecomp RIVO/DOKA, CGI Trade360, Eximbills — or core modules |
| **Messaging** | SWIFT Alliance / vendor gateway; MT700 family, MT760, MT400/422; MX (TSMT) migration | The workflow's nervous system (umbrella §5); message mismatches are where operational risk lives |
| **Integration** | Core booking/GL, limits domain, payments hub, screening engines, eBL/network connectors | APIs for booking, events/queues for settlement; see [../technology/message_queue_data_loss_guide.md](../technology/message_queue_data_loss_guide.md) |
| **Data / reporting** | Trade analytics, contingent-vs-funded exposure, regulatory returns, audit trail | Screening re-runs at presentation (vessel known only at shipment — umbrella §9.3) |

## 5.3 The Integration Narrative

A typical flow through the stack: the corporate applies for an LC in the **channel layer**; origination checks the **limits domain** and books a contingent draw; the **processing layer** owns the credit and generates the **MT700** via the messaging layer; the presentation arrives (paper or eBL via Bolero-class network), is checked in the processing layer against the credit and the rule timers; screening runs at issuance *and* at presentation; on honouring, the contingent exposure converts to funded, settlement passes through the **payments hub** (the cross-border mechanics of [../technology/late_arriving_data_guide.md](../technology/late_arriving_data_guide.md)), and the **data layer** reports the exposure. Every boundary crossing is a contract: the trade platform's API/event surface is the thing a bank architect actually designs — the vendor platform is the engine, the integration layer is the architecture. This matches the process view of [end_to_end_banking_processes.md](end_to_end_banking_processes.md) and the product-factory view of [universal_banking_model_guide.md](universal_banking_model_guide.md).

## 5.4 The Trade Data Model at a Glance

The trade system of record is, underneath, a small number of master data shapes with ferocious lifecycle rules. Understanding the shapes is what makes integration design tractable — and they recur across every vendor (Finastra TI, FLEXCUBE, T24, Eximbills all model these; the vendor-specific data-model detail is in [oracle_flexcube_data_model_guide.md](oracle_flexcube_data_model_guide.md) and [temenos_data_model_guide.md](temenos_data_model_guide.md)):

| Data shape | What it holds | The lifecycle trap |
|---|---|---|
| **Instrument master** (credit/guarantee/collection) | The contract: parties, amount, tenor, expiry, Incoterms, document requirements (46A), conditions (47A), confirmation status | A credit is *amended* many times; every amendment is versioned and affects all downstream steps |
| **Party/relationship data** | Applicant, beneficiary, advising/confirming banks, reimbursing bank, transferor/transferee | KYC/sanctions status changes over the instrument's life — screening re-runs matter |
| **Limit/commitment records** | Contingent draws at issuance, conversion to funded exposure at honouring | The contingent-vs-funded lifecycle ([banking_limits_domain_guide.md](banking_limits_domain_guide.md)); a USD 12.5M LC draws the limit at issuance, not payment (umbrella §9.2) |
| **Document sets / presentations** | The presented documents, examination results, discrepancy records, waivers, disposition | The 5-banking-day clock and the single-notice rule (Art 14(b), Art 16) are *timers on this shape* |
| **Message log** | MT700 family in/out, acknowledgements, rejections | Message mismatches (44C vs B/L date, 71A charges) are where operational risk lives (umbrella §5) |
| **Events/audit trail** | Every state change with actor, timestamp, and rationale | Documentary compliance *is* the audit trail; regulators and correspondents expect it |

## 5.5 Deployment and Operating Patterns

How the stack is deployed is as strategic as what is on it:

- **On-prem platform + in-country ops** — the classic large-bank pattern: the trade platform runs in the bank's data centres, operations staff sit in trade services teams, and the core/SWIFT/limits integrations are internal network calls. Maximum control, maximum ops cost, slowest change.
- **SaaS platform + retained ops** — the vendor runs the platform (Temenos SaaS, CGI Trade360 SaaS, Surecomp cloud — all verified delivery modes), the bank keeps trade operations staff and integration ownership. The dominant 2020s pattern; it moves release cadence to the vendor.
- **Shared service centre / factory** — the multi-country bank consolidates trade ops into one regional factory (Singapore, Poland, India are the classic hubs) and runs the platform centrally with local channel front ends. This is the Cymbal Bank operating model that the six-layer stack supports: one processing layer, many channel layers.
- **Co-sourced digitization** — eBL and network functions run on third-party platforms (Bolero-class) with the bank's platform orchestrating; the bank owns the connectors, not the infrastructure.

The deployment choice interacts with selection (§6): SaaS platforms shorten time-to-value but make the ISO 20022 and eBL roadmaps vendor-dependent; on-prem keeps the bank in control of the integration layer — the layer this guide argues is the real architecture (§5.3).

---

# 6. The Selection Guidance

## 6.1 The Selection Criteria

A trade-platform selection is a *system-of-record* decision: once chosen, the platform owns every credit, guarantee, and collection the bank issues — and the bank's operational risk profile follows it. **Verified** against the vendor materials and press covered in this guide, the criteria that actually separate the candidates are:

1. **Functional coverage** — LC (import/export, transfers, silent confirmations), guarantees/bonds (URDG lifecycle), collections (URC 522), open account, SCF; and the *operational* depth: document checking (AI/OCR-assisted), discrepancy management, presentation handling, rule timers (UCP 600 5-day clock, Art 16 single notice — [trade_finance_guide.md](trade_finance_guide.md) §4).
2. **SWIFT and ISO 20022 conformance** — SWIFT-certified MT700-family processing today, and a credible MX/TSMT migration path (the messaging layer is the workflow's nervous system — umbrella §5).
3. **Architecture and delivery** — SaaS vs on-prem, API surface (channel/ERP integration), eventing for integration, cloud readiness, and the vendor's release cadence. The integration layer is the architecture (§5.3).
4. **Digital-trade and eBL connectivity** — production connectors to Bolero-class eBL platforms and network APIs; the bank's digitization roadmap lives or dies on this surface.
5. **Compliance integration** — screening hooks at issuance and presentation, TBML analytics, audit-trail completeness ([financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md)).
6. **Regional fit and reference base** — the vendor's strength in the bank's markets (Asia/MEA for Oracle/Finacle/BaNCS/iGTB; global platforms for Finastra/Surecomp/CGI/China Systems), plus verifiable same-profile references (commodity trade finance for a Cymbal Bank).
7. **TCO and vendor viability** — licence + implementation + run cost over 7–10 years (a system-of-record platform outlives most core programmes), against the vendor's financial health and product-line commitment. The consolidation history (Misys→Finastra, DOKO/iTrade→Surecomp RIVO, essDOCS→ICE) is the cautionary tale: product-line churn is a real risk.
8. **The core-embedded alternative** — always score the "keep it in the core" option (Temenos TT / FLEXCUBE trade / Finacle / BaNCS) honestly: it trades depth for integration simplicity, and for many banks it is the right answer (§6.3).

## 6.2 The Selection Table

| Criterion | What to look for | Typical weight (**flagged** — indicative) | Notes |
|---|---|---|---|
| Functional coverage & document depth | LC/guarantee/collection/SCF coverage; AI document checking; discrepancy workflow | 25% | The system-of-record floor; verify against a real presentation pack |
| SWIFT / ISO 20022 | SWIFT certification; MX (TSMT) readiness | 15% | Messaging mismatches are where operational risk lives |
| Architecture & delivery | SaaS/on-prem; API/event surface; release cadence | 15% | The integration layer is the architecture (§5.3) |
| Digital-trade connectivity | eBL (Bolero-class) connectors; network APIs | 10% | Digitization roadmap dependency (§4) |
| Compliance integration | Screening hooks at issuance & presentation; TBML analytics | 10% | Screening is a lifecycle activity (umbrella §9.3) |
| Regional fit & references | Same-profile banks; regional delivery | 10% | Commodity-trade references for a Cymbal Bank |
| TCO & vendor viability | 7–10-year cost; product-line commitment | 10% | Consolidation history is the cautionary tale |
| Core-embedded alternative | Honest score of Temenos/Oracle/Finacle/BaNCS modules | 5% | §6.3 — sometimes the right answer |

## 6.3 The Core-Embedded vs Standalone Decision

The first decision is structural, not vendor: **does trade live inside the core or on a standalone platform?**

- **Choose core-embedded** when: the bank is universal mid-market; trade volumes are moderate; integration headcount is scarce; the core's trade modules cover the product set; and the bank has no trade-digitization ambitions beyond what the core ships. Temenos TT, FLEXCUBE trade, Finacle, BaNCS are the candidates.
- **Choose standalone** when: trade is a strategic product (commodity trade finance, trade factories, confirming houses); document volume is high; the bank needs AI document checking, eBL orchestration, and network integration ahead of the core's roadmap; or the core is not a trade-capable vendor. Finastra TI, Surecomp RIVO/DOKA, CGI Trade360, China Systems Eximbills are the candidates — with iGTB as the middle path (trade inside a transaction-banking suite rather than inside the core proper).

The Cymbal Bank profile (Section 7) is firmly in the standalone camp — which is exactly the umbrella's §8.2 verdict: "the core is not the trade system."

## 6.4 The RFP Annex: What the Bank Should Actually Ask

A trade-platform RFP fails when it asks for feature lists and gets brochures. The questions that separate vendors — framed for the system-of-record reality:

1. **Show me a presentation, end to end.** Ask each vendor to walk a real LC presentation pack (yours, sanitized) through their platform: document intake (paper and eBL), examination, discrepancy capture, waiver workflow, and the UCP 600 timers. Who clicks what, and what is automated? This is the single most informative hour of any trade selection.
2. **Where is your AI document checking, really?** Every vendor claims AI; ask for *production* discrepancy-capture rates and false-positive numbers, not roadmap slides (**any vendor figure is flagged** until audited in a PoC).
3. **Show the integration contracts.** The bank's architects should read the vendor's API catalogue and event model for: limits booking, GL booking, SWIFT message flows, screening calls, and eBL connectors. If the vendor cannot show an API contract for contingent-limit draw and release, walk away — that is the core trade lifecycle.
4. **What is your ISO 20022 trade (TSMT) plan and date?** Not "are you ISO 20022-ready" (everyone says yes) but *which* MX messages, *when* in production, and *who* has done it. The messaging layer is the nervous system (umbrella §5).
5. **eBL: name the production integrations.** Bolero-class connector in production with which banks, since when, at what volume? Contour-class networks: are you integrated, and what is your read on their status? (The correct honest answer about Contour in 2026 includes a flag — see §4.1.)
6. **Sizing and performance.** Throughput per ops FTE, batch windows, disaster-recovery RTO/RPO, and the SaaS tenancy model. A trade factory's cost base is STP and staffing; the vendor must model both.
7. **Reference calls of the right profile.** For a Cymbal Bank: a commodity-trade bank, an Asian regional bank, and a bank that migrated *off* a legacy trade system (ask about the migration pain — §7.3 lesson 5).
8. **The exit clause.** Data export, instrument migration back out, and termination terms. A system of record outlives contracts; the exit is part of the deal, not an afterthought.

---

# 7. The Worked Example: A Cymbal Bank's Trade-Systems Landscape

## 7.1 The Scenario

The setting is familiar territory: a **Singapore-based international bank of the Cymbal Bank type** — Cymbal Bank's profile: a corporate & investment bank strong in **commodity trade finance**, with Singapore as its regional commodity hub, running a large LC/guarantee book for commodity traders, corporates, and financial institutions, plus a confirmation business for correspondent banks. The bank is fictionalized as "**the Bank**" to keep the example vendor-neutral, but every product and constraint is the Cymbal Bank reality (the umbrella's worked example in [trade_finance_guide.md](trade_finance_guide.md) §9 runs the same book end-to-end).

**The starting estate (the pain):**

- A **legacy trade platform** (a 1990s-era trade system, MISYS TI-class or older) that owns the LC/guarantee book but has no APIs, no eBL support, and no cloud story; SWIFT via an Alliance gateway; document examination largely manual; screening re-keyed into a separate system.
- The **core** is an international core (FLEXCUBE-class — see [oracle_flexcube_data_model_guide.md](oracle_flexcube_data_model_guide.md)) used for booking and GL; the core's trade module was deliberately *not* used for trade processing because the commodity book's document volumes and confirming operations outgrew it — the classic standalone-platform rationale.
- **Pain points**: STP rates limited by manual document checking; no client self-service (applications by email/fax); no eBL capability while clients move to Bolero-style issuance; ISO 20022 trade migration looming; and discrepancy-handling cost (umbrella §9.2: ~60–70% first-presentation discrepancy rate — **flagged figure**).

**The mandate:** replace the legacy trade platform with a modern standalone trade & SCF platform (a **Finastra Trade Innovation / Surecomp RIVO-class** decision), keep the core for booking, integrate limits, SWIFT, screening, payments, and Bolero eBL, and deliver a digital client journey — the Singapore digital-trade agenda (Singapore ETA 2021 gives eBLs legal standing — umbrella §7.5).

## 7.2 The Stack Design

The target architecture is the six-layer stack of [Section 5](#5-the-architecture), instantiated for the Bank:

| Layer | The Bank's choice | Rationale |
|---|---|---|
| **Channel** | New corporate trade portal + APIs (white-label of the platform vendor's digital layer); ERP connector for top commodity clients | Client self-service LC applications and document upload; the RIVO-style collaborative journey |
| **Origination** | Platform origination with contingent-limit check at application | Limits booked at issuance, funded at honouring — [banking_limits_domain_guide.md](banking_limits_domain_guide.md) |
| **Processing (system of record)** | **New standalone platform** (Finastra TI / Surecomp RIVO-DOKA-class — shortlisted per §6.2 scoring) | Replaces the legacy system; AI-assisted document checking; rule timers (UCP 600 clock) encoded |
| **Messaging** | Platform's SWIFT layer + Alliance gateway for the long tail; MX/TSMT migration workstream | MT700 family today; ISO 20022 trade messages on the roadmap ([iso_20022_core_processes_guide.md](iso_20022_core_processes_guide.md)) |
| **Integration** | APIs to FLEXCUBE-class core (booking/GL), limits domain, payments hub, screening engine; events/queues for settlement notifications | The integration layer is the architecture; see [../technology/message_queue_data_loss_guide.md](../technology/message_queue_data_loss_guide.md) |
| **Digital networks** | **Bolero eBL** connector for eBL issuance/presentation; network APIs for the confirmation business; Contour monitored (**status flagged** — §4.1) | eBL where the commodity book moves; legal substrate ready in Singapore |
| **Data / reporting** | Trade exposure dashboards (contingent vs funded), regulatory returns, screening re-runs at presentation | Screening is lifecycle, not point-in-time (umbrella §9.3) |

**The transition**: the LC/guarantee book is migrated instrument-by-instrument (open credits on the legacy system run to maturity — a trade book cannot be cut over mid-credit), with a parallel-run period where both platforms process new issuance; the confirmation book is migrated first (high-volume, low-document complexity), then the commodity LC book, then guarantees. SWIFT traffic is redirected vendor-gateway-first, then Alliance traffic drains down. The eBL pilot runs with two anchor commodity clients and Bolero issuance before general availability.

## 7.3 The Lessons

1. **The system-of-record decision is a risk decision.** The platform owns every credit, guarantee, and collection — its discrepancy workflow, rule timers, and audit trail *are* the bank's documentary compliance. Score the operational depth, not the demo.
2. **Standalone vs core-embedded is the first question, vendor is the second.** The Bank's commodity book structurally requires a standalone platform (umbrella §8.2); a mid-market universal bank on the same vendors might correctly stay core-embedded (§6.3). Architecture first, logo second.
3. **The integration layer is where the architect earns the fee.** The vendor platform is the engine; the design work is the API/event contracts to core, limits, SWIFT, screening, payments, and eBL — every boundary crossing is a contract (§5.3).
4. **Digitization is a network of narrow jobs, not one network.** Bolero eBL-as-a-Service is a production utility; Contour is a flag-until-verified relaunch (§4.1); the failed consortia are history (umbrella §7.4). The Bank builds connectors, not consortium memberships.
5. **Migration is lifecycle-shaped.** A trade book cannot be cut over mid-credit; open instruments run to maturity on the legacy platform. Parallel-run and instrument-by-instrument migration are the only safe pattern — the same lesson as core replacement programmes ([core_banking_systems_guide.md](core_banking_systems_guide.md)).
6. **ISO 20022 and eBL are on the same roadmap.** The MX/TSMT migration and the eBL/e-presentation agenda both rewrite the message and document journeys — sequence them together or pay twice ([iso_20022_core_processes_guide.md](iso_20022_core_processes_guide.md), [trade_finance_guide.md](trade_finance_guide.md) §7.5).

## 7.4 The Current vs Target Estate

The before/after in one table — the shape of the programme the Bank is actually running:

| Function | Current (legacy) | Target (new stack) | The change |
|---|---|---|---|
| **Trade system of record** | 1990s legacy trade platform (MISYS TI-class) | New standalone platform (Finastra TI / Surecomp RIVO-DOKA-class) | Modern document engine, rule timers, API surface |
| **Client channel** | Email/fax applications, phone status calls | Corporate portal + APIs + ERP connectors | Self-service LC applications, document upload, status push |
| **Document handling** | Manual examination, paper archives | AI-assisted checking, e-presentation, eBL intake | STP up, discrepancy cost down (target metrics **flagged** as programme aspirations) |
| **Messaging** | Alliance gateway, manual re-keying | Platform-native SWIFT layer + MX/TSMT workstream | Message mismatches down; ISO 20022 ready |
| **Core integration** | Nightly batch, reconciliation pain | API booking to FLEXCUBE-class core | Real-time booking, single GL truth |
| **Limits** | Spreadsheet-adjacent tracking | Contingent draw/release via limits domain APIs | Lifecycle exposure management ([banking_limits_domain_guide.md](banking_limits_domain_guide.md)) |
| **Screening** | Re-keyed at issuance only | Engine integration at issuance *and* presentation | Lifecycle screening (umbrella §9.3) |
| **Digitization** | None | Bolero eBL connector, network APIs, Contour monitored (**flagged**) | eBL pilot with anchor commodity clients; connector-first, no consortium commitments |

**The programme shape**: a 24–36 month replacement — 6 months selection and contracting (using [§6.2](#62-the-selection-table) scoring and the [§6.4](#64-the-rfp-annex-what-the-bank-should-actually-ask) RFP), 12 months build and integration (the layer-by-layer work of [§5.3](#53-the-integration-narrative)), 6–9 months parallel run and instrument migration (confirmation book first, then commodity LCs, then guarantees — [§7.2](#72-the-stack-design)), with the eBL pilot and ISO 20022 workstream running in parallel. The single biggest schedule risk is not the platform — it is the integration contracts to core, limits, and screening, which is precisely why this guide keeps saying the integration layer is the architecture.

---

# 8. The Summary: One Page

**Trade finance systems** are the dedicated platform stack behind the bank's LC, guarantee, and collection business — a stack that exists because the trade workflow (document examination, contingent limits, SWIFT messaging, guarantee lifecycle, discrepancy management) does not fit a vanilla core-banking loan module.

**The landscape in one page:**

| Dimension | State of play |
|---|---|
| **The stack** | Six layers: channel, origination, processing (system of record), messaging (SWIFT/ISO 20022), integration (core/limits/payments/screening/eBL), data — the trade platform is the orchestrator (§5) |
| **The structural fork** | Standalone platforms (Finastra TI, Surecomp RIVO/DOKA, CGI Trade360, China Systems Eximbills, iGTB) vs core-embedded modules (Temenos TT, FLEXCUBE trade, Finacle, BaNCS) — architecture first, vendor second (§6.3) |
| **The vendors (verified)** | Finastra Trade Innovation (Misys TI heritage, FusionFabric.cloud); Oracle FLEXCUBE trade modules (LC/guarantees/bills/collections); Temenos T24/Transact TT; Intellect iGTB (suite player); EdgeVerve Finacle (componentized + Trade Connect blockchain); TCS BaNCS (API-first platform) — share figures flagged throughout (§2) |
| **The digitization layer** | Bolero (eBL-as-a-Service, Galileo, Title Registry — operating); Contour (**status flagged**: closure 2025 → Xalts → XDC Ventures relaunch); the consortia are history (TradeLens, we.trade, Marco Polo); the legal substrate (MLETR/Singapore ETA) is the growth driver (§4) |
| **Selection** | Score functional coverage, SWIFT/ISO 20022, architecture, eBL connectivity, compliance, regional fit, TCO — and always score the core-embedded alternative honestly (§6) |
| **For the architect** | The vendor platform is the engine; the integration layer is the architecture; the API/event contracts to core, limits, SWIFT, screening, payments, and eBL are the real design (§5.3, §7.2) |

**The final word: the systems behind the trade.** The letter of credit was perfected in the 19th century and its rules in the 20th; what has changed is the machinery behind it. The bank trade-systems landscape is a mature, consolidated market — a handful of standalone platforms (Finastra, Surecomp, CGI, China Systems), the core-embedded modules of the big cores (Temenos, Oracle, Finacle, BaNCS), and the transaction-banking suites (iGTB) — fighting over a book that is still paper-heavy, discrepancy-prone, and only now being digitized. The digitization platforms taught the industry the hard lesson: networks fail, narrow jobs survive — Bolero sells eBLs, the consortia shut down, and Contour's fate is a flag until it is a fact. For the architect, the trade stack is an orchestration problem: the trade platform owns the document of record, and everything else — core, limits, SWIFT, screening, payments, eBL — is a contract across a boundary. The bank that treats the trade platform as the system of record, the integration layer as the architecture, and digitization as a roadmap of connectors, is the bank that processes a complying presentation in hours while its competitors still count discrepancy days. The systems behind the trade are not glamorous — they are the difference between a trade business that scales and one that drowns in its own paper.

---

# 9. The Glossary

| Term | Definition |
|---|---|
| **Trade finance systems** | The bank-side platforms and modules that originate, process, and lifecycle trade products (LCs, guarantees, collections, SCF) — the dedicated stack trade requires beyond the core |
| **Trade stack** | The layered architecture of a bank's trade business: channel, origination, processing, messaging, integration, data (§5) |
| **Trade platform** | A standalone trade processing system of record (e.g. Finastra Trade Innovation, Surecomp RIVO/DOKA, CGI Trade360, Eximbills) |
| **Landscape** | The full set of system categories and vendors in a domain — here, the bank trade-systems landscape (§1) |
| **Vendor** | The company supplying the software (Finastra, Oracle, Temenos, Intellect, EdgeVerve, TCS, Surecomp, CGI, China Systems) |
| **Platform** | A productized software suite serving a business function — trade processing, transaction banking, eBL |
| **Core banking** | The bank's central system of record for customer master, GL, deposits, loans — with optional trade modules; see [core_banking_systems_guide.md](core_banking_systems_guide.md) |
| **Core-embedded trade** | Trade capability inside the core (Temenos TT, FLEXCUBE trade, Finacle, BaNCS) rather than on a standalone platform |
| **Finastra** | Global banking software group (Misys + D+H, 2017); vendor of Trade Innovation |
| **Trade Innovation** | Finastra's standalone trade & SCF platform (Misys TI heritage; TI/TI Plus branding; FusionFabric.cloud APIs) |
| **Oracle** | Oracle Financial Services — vendor of FLEXCUBE Universal Banking and its trade modules |
| **FLEXCUBE** | Oracle's flagship core banking platform (FCUBS); trade modules cover LC, guarantees, bills & collections, trade origination |
| **Temenos** | Swiss core-banking vendor; T24/Transact with the TT trade module family |
| **T24** | Temenos's flagship core (rebranded Temenos Transact, 2019); runs the TT trade modules |
| **IGTB** | Intellect Global Transaction Banking — Intellect Design Arena's transaction-banking arm; trade & SCF suite |
| **Intellect** | Intellect Design Arena — Indian banking technology group; vendor of the iGTB platform |
| **EdgeVerve** | EdgeVerve Systems — product subsidiary of Infosys; vendor of Finacle |
| **Infosys** | Indian IT services group; parent of EdgeVerve/Finacle |
| **TCS BaNCS** | TCS's banking product platform (Global Banking Platform); trade finance component; API-first |
| **Contour** | DLT (Corda) LC digitization network, ex-Voltron — **status flagged**: closure 2025, Xalts acquisition, XDC Ventures relaunch (Oct 2025) |
| **Bolero** | Bolero International (WiseTech Global Group) — operating eBL/e-document platform: eBL-as-a-Service, Galileo, Title Registry |
| **Digitization** | Replacing paper trade documents/processes with electronic records and automated workflows (eBL, e-presentation, MLETR/ETA regimes) |
| **Architecture** | The layered design of the trade stack and its integration contracts (§5) |
| **Selection** | The criteria-driven choice of a trade platform (or core-embedded module) — §6 |
| **LC** | Letter of credit — a bank's irrevocable undertaking to pay against a complying presentation (UCP 600) |
| **Guarantee** | A bank's undertaking to pay if the principal fails to perform (URDG 758); "bonds" in trade usage |
| **eBL** | Electronic bill of lading — the digital title document; Bolero-class platforms provide the registry/transfer mechanics |
| **STP** | Straight-through processing — automated end-to-end handling without manual intervention; the STP metric of trade ops |
| **ISO 20022 (trade)** | The MX message standard family replacing MT messages; trade services (TSMT) migration is a roadmap item for every vendor here |
| **System of record** | The system that owns the authoritative state of an instrument — the trade platform owns every LC/guarantee/collection |
| **UCP 600** | ICC Uniform Customs and Practice for Documentary Credits (2007) — the LC rule stack the platforms encode (umbrella §4) |
| **eUCP** | The ICC supplement governing electronic presentations under documentary credits — the rule layer for eBL/e-documents |
| **SWIFT** | The interbank messaging cooperative; FIN network and the MT message families that trade systems generate and parse |
| **MT700** | SWIFT "Issue of a Documentary Credit" — the operative LC message the trade platform produces at issuance |
| **Surecomp** | Trade software specialist (Israel); vendor of the RIVO collaborative platform and the DOKA-NG back office |
| **RIVO** | Surecomp's collaborative trade finance platform (DZ Bank 2024, verified) — client-facing, API-integratable |
| **DOKA / DOKA-NG** | Surecomp's trade back-office engine (BTPN Indonesia deployment, verified); DOKO/iTrade are the heritage names |
| **China Systems** | The Eximbills vendor (London HQ); 150+ customers claimed; GTR-awarded trade back office |
| **Eximbills** | China Systems' trade back office: full trade cycle automation to SWIFT, UCP, and ISO 20022 standards |
| **CGI Trade360** | CGI's trade + open account + SCF suite (ANZ, ASB) — LC/guarantee/compliance automation, SaaS-delivered |
| **TBML** | Trade-based money laundering — the invoice/price/document manipulation typologies screening engines detect |
| **FusionFabric.cloud** | Finastra's open-banking API platform — the ecosystem layer around Trade Innovation |

---

# 10. The Related Guides and Sources

## Related Guides

**Banking (this series)** — plain filenames:
- `trade_finance_guide.md` — **the umbrella**: instruments, rules, messages, risks, digitization, and the §8 systems overview this guide expands; cross-reference §8 heavily
- `supply_chain_finance_guide.md`, `supply_chain_finance_technologies_guide.md` — the adjacent receivables/payables product family the trade platforms also serve
- `core_banking_systems_guide.md`, `temenos_guide.md`, `oracle_flexcube_data_model_guide.md` — the cores with trade modules; `temenos_data_model_guide.md`, `t24_programming_guide.md` for T24 depth
- `chinese_core_banking_vendors_guide.md` — the vendor-profile pattern this guide follows; `chinese_bank_core_systems_guide.md` for the Chinese cores
- `murex_mx3_platform_guide.md`, `nasdaq_calypso_guide.md`, `openbankproject_guide.md` — the platform-guide pattern (treasury, post-trade, open banking)
- `banking_limits_domain_guide.md` — contingent-vs-funded exposure (LCs/guarantees are the canonical contingent products)
- `end_to_end_banking_processes.md`, `core_banking_processes_guide.md`, `universal_banking_model_guide.md` — the process/product views the trade stack instantiates
- `payments_hub_guide.md`, `iso_20022_core_processes_guide.md` — settlement rails and message standards
- `financial_risk_compliance_systems_guide.md` — screening/TBML systems in the compliance layer
- `dbs_software_systems_guide.md`, `ocbc_software_systems_guide.md` — bank trade operations in practice (Singapore peers)

**Technology** — `../technology/` prefix:
- `../technology/late_arriving_data_guide.md` — cross-border settlement (the reimbursement step of the trade lifecycle)
- `../technology/distributed_auth_guide.md` — digital identity for trade (KYC/e-signature substrate)
- `../technology/message_queue_data_loss_guide.md` — the integration-layer messaging infrastructure
- `../technology/ica_systems_guide.md` — customs systems, the logistics-side counterpart

## Sources and Verification Notes

**How this guide relates to the umbrella.** [trade_finance_guide.md](trade_finance_guide.md) §8 is the ~41-line systems overview this guide expands into full depth. Everything in the umbrella's §12 that touches systems (vendor list, share flags, Contour status) is carried forward and where possible upgraded: Contour's 2025 acquisition timeline is now verified in detail (§4.1); the Surecomp product line is updated from the heritage names (DOKO/iTrade) to the current RIVO/DOKA-NG line (§2.8); China Systems is added to the vendor set on the strength of this session's shortlist research; and the digitization layer now has the eBL integration pattern worked out in §4.4. Where this guide could not verify something, it says so in the flagged list below — consistent with the series' verification posture.

**Verified facts** (via targeted web searches, August 2026 — search-only backend; page extraction degraded, so some primary pages could not be read in full):

- Finastra Trade Innovation: trade & SCF platform for banks, digital client channels, FusionFabric.cloud API ecosystem (finastra.com solution pages; third-party platform roundups); CredAble SCF partnership (PRNewswire).
- Finastra formation: Misys + D+H merger, 2017 — widely documented industry fact.
- Oracle FLEXCUBE Universal Banking Trade Finance modules: letters of credit, bank guarantees, bills and collections, trade origination; FCUBS 12.0.3 (Oracle University course pages, docs.oracle.com).
- Temenos Trade Finance module: LC-related processing incl. reimbursement (Temenos trade finance paper, May 2020); Banco de la Nación Argentina T24 trade/payments (GTR).
- Intellect iGTB: transaction-banking arm of Intellect Design Arena; trade & SCF suite; Bank of Baroda SCF platform; Aite "vendor to watch" mention (Intellect site, industry press); Microsoft Cloud partnership.
- EdgeVerve/Infosys Finacle: Finacle Trade Finance Solution Suite (componentized); Finacle Trade Connect blockchain solution, global availability announced Nov 2017 (finacle.com, edgeverve.com press release).
- TCS BaNCS: Global Banking Platform, API-first; Janata Sahakari Bank (Pune) selected BaNCS for core + trade finance + payments + AML/KYC modernization (industry press, 2025).
- Surecomp: RIVO collaborative trade finance platform (DZ Bank adoption, June 2024; Investec South Africa commentary); DOKA back office (BTPN Indonesia); SWIFT-certified (surecomp.com, Finextra, IBS Intelligence).
- China Systems: Eximbills trade back office; 150+ customers (vendor claim); GTR awards; SWIFT/UCP/ISO 20022 conformance (chinasystems.com, GTR, SourceForge).
- CGI Trade360: trade + open account + SCF suite; LC/guarantees/compliance checking; clients incl. ANZ and ASB (CGI materials, GTR, press).
- Contour: announced closure 2025 (funding shortfall; Ledger Insights/GTR); Xalts acquisition (Trade Finance Global, 2025); XDC Ventures acquisition from Xalts, 29 Oct 2025, "re-energise" + Stable-Coin Lab (GTR, contour.network).
- Bolero: Bolero International in the WiseTech Global Group; eBL-as-a-Service, Galileo Multi-Bank platform, Title Registry (bolero.net, LinkedIn, Preqin).

**Flagged / not independently verified** (treat with care):

- Market size (~USD 5.2B by 2031, ~3% CAGR) — single market-research report, not cross-verified; "200+ FI combined installed base" (Finastra/Surecomp/CGI/iGTB) — vendor-side research claim.
- Vendor rankings ("Finastra, Surecomp, CGI, China Systems lead the bank market") — directional; no audited ranking exists (consistent with the umbrella's flag).
- China Systems' 150+ customers and GTR award record — vendor-claimed (award record partially corroborated by GTR/Crunchbase).
- Client rosters beyond those verified above (e.g. Finastra's full bank list, Bolero live-bank counts, komgo's activity level) — not independently audited this session.
- Contour's current operating status — volatile post-acquisition relaunch; re-verify before citing as active.
- iGTB product naming (eTrade/eSCF family) — varies by release; confirm against current materials.
- "60–70% first-presentation discrepancy rate" and other umbrella-flagged figures — carried over from [trade_finance_guide.md](trade_finance_guide.md) §12, still flagged.

---

*End of guide — the systems behind the trade.*
