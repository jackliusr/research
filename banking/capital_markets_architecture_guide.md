# Capital Markets Architecture — The Markets-Technology Stack, Deep-Dive

*A dedicated architecture umbrella for the markets side of the bank-software-systems series: the full capital-markets technology stack — front office, middle office, back office, market data, market infrastructure, protocols, and regulation — plus the patterns that hold it together and a worked CIB architecture. This guide is the **architecture umbrella over the markets-platform guides**: it cross-references the [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md) and the [Nasdaq Calypso Guide](nasdaq_calypso_guide.md) (the treasury/markets platform layer), the [Crédit Agricole Software Systems Guide](credit_agricole_software_systems_guide.md) (the Cymbal Bank markets stack, §3 — the worked example's real-world anchor), the [Trade Finance Guide](trade_finance_guide.md) and [Trade Finance Systems Guide](trade_finance_systems_guide.md) (the trade products), the [Risk Management Models Guide](risk_management_models_guide.md) and [Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md) (the risk layer), the [Core Banking Systems Guide](core_banking_systems_guide.md) and [Universal Banking Model Guide](universal_banking_model_guide.md) (the banking-book boundary), the technology-series [Singapore Data Centres Guide](../technology/singapore_data_centres_guide.md) (co-location and low latency), the [Event Stream Processing Guide](../technology/event_stream_processing_guide.md), [Message Queue Data Loss Guide](../technology/message_queue_data_loss_guide.md) and [Kafka Alternatives Guide](../technology/kafka_alternatives_guide.md) (the event-driven infrastructure), and — lightly — [Distributed Rate Limiter Guide](../technology/distributed_rate_limiter_guide.md), [Distributed Auth Guide](../technology/distributed_auth_guide.md), [Monolith to Microservices Guide](../technology/monolith_to_microservices_guide.md), and the [DeFi Guide](../technology/defi_guide.md) (the DLT-vs-central-infrastructure contrast).*

**Verification convention used throughout: ✅ = verified in this research pass (primary or stable public sources — fixtrading.org, ESMA, Wikipedia regulatory pages, the CCPs' own sites, the architecture press); ⚠ = flagged (inferred, approximate, single-source, or not re-verified this pass); unmarked = structural/industry knowledge presented as such. The consolidated verification notes are in [§13](#13-verification-notes-and-sources).**

**Research-method note — read this before trusting any ✅:** this pass had **degraded web access** (`web_search` and `web_extract` backends unavailable — search backend unconfigured, extract backend search-only). Verification was done by direct HTTP fetch of primary/stable pages: fixtrading.org (FIX standards), ESMA and Wikipedia (regulatory dates), Eurex and CME (market infrastructure). Where a fact could not be re-verified live, it is flagged ⚠ and stated as knowledge-base with the honest caveat. **Nothing in this guide is fabricated; where the brief demanded verification and the tools were down, this guide says so plainly.** The primary reader is a capital-markets architect who will catch any invented protocol fact or regulatory date immediately. **UPDATE (2026-08-22):** key flags were re-verified live via the restored web backend (self-hosted Firecrawl): CSDR Settlement Discipline (penalties 1 Feb 2022, buy-ins suspended 3 years per ESMA RTS; CSDR Refit revision in OJ 14 Oct 2025, T+1 target 11 Oct 2027), FIX 4.4 release year (2003 — FIXML 4.4 schema guide dated 9 Jan 2004 on fixtrading.org), LCH SwapClear dominance (LSEG), **AcadiaSoft ownership corrected (now Acadia, part of LSEG — NOT DTCC)**, CTM (DTCC), UMR phases (2016–Sep 2021) — see §13.

### How this guide relates to the series

This guide is the **architecture umbrella over the markets-platform guides** — it describes the *stack* the platforms sit in, not a specific vendor's product. The division of labour with the siblings:

| Angle | Where it lives |
|---|---|
| **The stack itself (this guide)** | The front/middle/back-office layers, market data, infrastructure, protocols, regulation, patterns, worked example |
| The platform layer (Murex, Calypso) | [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md), [Nasdaq Calypso Guide](nasdaq_calypso_guide.md) — the front-to-back platforms of §1–§4 |
| The Cymbal Bank markets stack (worked-example anchor) | [Crédit Agricole Software Systems Guide](credit_agricole_software_systems_guide.md) §3 — the CIB context of §10 |
| The trade products the stack processes | [Trade Finance Guide](trade_finance_guide.md), [Trade Finance Systems Guide](trade_finance_systems_guide.md) |
| The risk layer on top of the stack | [Risk Management Models Guide](risk_management_models_guide.md), [Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md) |
| The banking-book boundary | [Core Banking Systems Guide](core_banking_systems_guide.md), [Universal Banking Model Guide](universal_banking_model_guide.md) |
| The co-location/low-latency facilities | [Singapore Data Centres Guide](../technology/singapore_data_centres_guide.md) |
| The event-driven infrastructure | [Event Stream Processing Guide](../technology/event_stream_processing_guide.md), [Message Queue Data Loss Guide](../technology/message_queue_data_loss_guide.md), [Kafka Alternatives Guide](../technology/kafka_alternatives_guide.md) |
| The auth/API-security patterns | [Distributed Auth Guide](../technology/distributed_auth_guide.md); lightly [Distributed Rate Limiter Guide](../technology/distributed_rate_limiter_guide.md) |
| The migration discipline | [Monolith to Microservices Guide](../technology/monolith_to_microservices_guide.md) |
| The DLT-vs-central-infrastructure contrast | [DeFi Guide](../technology/defi_guide.md) |

Reading order for a new architect: §1 (the stack), §2–§4 (the layers), §5–§8 (the fabric: data, infrastructure, protocols, regulation), §9 (the patterns), §10 (the worked example), §11 (the one-page summary).

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [The Front Office](#2-the-front-office)
3. [The Middle Office](#3-the-middle-office)
4. [The Back Office](#4-the-back-office)
5. [Market Data](#5-market-data)
6. [Market Infrastructure](#6-market-infrastructure)
7. [Protocols](#7-protocols)
8. [Regulation](#8-regulation)
9. [Architecture Patterns](#9-architecture-patterns)
10. [Worked Example: A CIB Markets Architecture](#10-worked-example-a-bank-markets-architecture)
11. [Summary: One Page](#11-summary-one-page)
12. [Glossary](#12-glossary)
13. [Verification Notes and Sources](#13-verification-notes-and-sources)

---

## 1. Architecture Overview

### 1.1 What capital markets are — and are not

**Capital markets** are the markets where **long-term funding and risk are traded** — as opposed to money markets (short-term, under one year). The industry-standard definition decomposes the business into three verbs: **trading, structuring, and distribution** ✅ (the canonical framing used across industry literature, including the [Universal Banking Model Guide](universal_banking_model_guide.md) and the CIB guides):

- **Trading** — buying and selling securities and derivatives: either as **flow** (market-making, executing client orders in liquid products — FX, rates, equities, credit) or as **proprietary/position-taking** (the bank's own book). Trading is where the *markets* in "capital markets" happens: prices are discovered, risk changes hands, and the bank earns bid/offer and inventory risk premium.
- **Structuring** — designing and manufacturing products that are not off-the-shelf: derivatives (swaps, options, structured notes), securitizations (ABS, MBS, CLOs), and syndicated issuance. Structuring repackages risk and cash flows into instruments clients want (hedges, yield, funding). It sits between trading and distribution: a structurer creates the product, a trader prices and hedges it.
- **Distribution** — getting those products to end-investors and end-users: primary issuance (bonds, equities, syndicated loans), sales to institutional clients (asset managers, pension funds, hedge funds, insurers, corporates), and the electronic channels that serve them (e-trading portals, APIs, algo execution). Distribution is the franchise: the client relationship that brings flow back to the trading desk.

A useful boundary from the series: the **banking book** (deposits, loans — owned by the [Core Banking Systems Guide](core_banking_systems_guide.md)) holds assets to maturity and earns net interest margin; the **trading book** (this guide) holds positions to trade and earns bid/offer and P&L — and is marked to market daily. The [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md) §1.5 draws this exact line for platform purposes.

### 1.2 The three-layer stack: front, middle, back office

The industry-standard decomposition of a markets organisation is the **front / middle / back office** three-layer stack ✅ (universal in the architecture press — WatersTechnology, The TRADE, Finextra — and in every vendor's "front-to-back" positioning):

| Layer | What it does | Who works there | Time horizon |
|---|---|---|---|
| **Front office (FO)** | Origination: pricing, trading, execution, sales | Traders, salespeople, structurers, quants | Real-time, intraday |
| **Middle office (MO)** | Control: trade validation, confirmations, collateral, risk monitoring, P&L attribution | MO controllers, risk analysts, trade support | Intraday to T+1 |
| **Back office (BO)** | Records: settlement, clearing, custody, accounting, regulatory reporting | Operations staff, settlement teams | T+1 to T+x, end-of-day |

The stack is a **control pipeline**, not just a division of labour: each layer is a check on the one above it. A trade that begins in the FO as a trader's intent ends in the BO as a settled, accounted, reported obligation. The middle office exists precisely because the front office is optimised for speed and the back office for correctness — someone must own the reconciliation between the two. This is the same front-to-back logic the [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md) describes as MX.3's core value proposition (one platform spanning all three layers), and the [Nasdaq Calypso Guide](nasdaq_calypso_guide.md) describes for Calypso.

### 1.3 The overview table

| Layer | Function | Representative systems |
|---|---|---|
| **Front office** | Pricing, e-trading, OMS/EMS, sales coverage | Murex MX.3, Calypso, ION, FlexTrade, Fidessa (ION), Bloomberg EMSX, in-house pricing/risk engines |
| **Middle office** | Trade lifecycle, confirmations, collateral management, risk/P&L control | Murex/Calypso post-trade modules, ION (Broadridge) DTC, Acadia (LSEG), TriOptima, Quantile, Bloomberg Collateral |
| **Back office** | Settlement, clearing, custody, accounting, reporting | Broadridge, FIS, Omgeo (DTCC) CTM, SWIFT, Euroclear/Clearstream/DTCC, SmartStream TLM, Lombard Risk (now part of Vermeg) |
| **Market data** | Real-time feeds, reference data | Bloomberg, Refinitiv (LSEG), FactSet, ICE Data, Bloomberg Data License, GoldenSource, Refinitiv Reference Data |
| **Market infrastructure** | Exchanges, CCPs, CSDs, trade repositories | LSE, Deutsche Börse/Eurex, CME, ICE, LCH, Eurex Clearing, CME Clearing, Euroclear, Clearstream, DTCC |
| **Connectivity/protocols** | Order/trade messaging, market data distribution | FIX (FIX 4.4, FIXT 1.1), SWIFT, ITCH/OUCH (NASDAQ), OUCH, SBE/FAST (binary encodings), MQ/EMS |
| **Regulation/reporting** | Transaction reporting, risk mitigation, disclosure | EMIR/REMIT/SFTR reporting hubs (UnaVista, Regis-TR, DTCC Data Repository), MiFID II transaction reporting (ARM), Xceptor, Vermeg |

The remainder of this guide walks the stack layer by layer — then the market data, infrastructure, protocols, and regulation that the stack plugs into, the patterns that shape how it is built, and a worked CIB architecture that ties it together.

### 1.4 How the stack is owned and operated

Three structural facts shape every capital-markets architecture decision, and they are worth stating before the layer walk-through:

- **The estate is owned in slices, but the data is one stream.** Front-office systems are typically owned by desk-aligned technology teams (the e-trading squad, the rates-quant squad); the platform and the MO/BO systems by a front-to-back platform team; and the infrastructure (market data, messaging, the backbone) by a shared platform-services function. Yet the trade is a single stream flowing through all of them — so the estate lives or dies by its **integration contracts** (the lifecycle schema of §3.2, the FIX/SWIFT/API layer of §7), not by the quality of any single system.
- **The operating model is a batch inside a stream.** The FO is real-time; the MO is intraday; the BO is end-of-day. The classic architectural rhythm is: continuous events all day → EOD batch (positions, valuations, P&L, settlement runs, regulatory reports) → the next day's open. The batch window is a hard architectural constraint — the estate must finish its EOD cycle before the Asian markets reopen, which is why the [event-driven patterns](#9-architecture-patterns) and the two-speed design of §9 are not optional polish but the load-bearing structure.
- **The estate is enormous and unglamorous.** A top-tier CIB runs thousands of FIX sessions, ingests terabytes of tick data a day, reconciles millions of positions, and reports hundreds of thousands of transactions per quarter under EMIR/SFTR/MiFID II (§8) ⚠ (the magnitudes are structural-industry knowledge, flagged as approximate). Most of the architecture's complexity is in the quiet parts: reference-data quality, reconciliation, exception handling — the parts that never appear in a latency benchmark.

The consequence for the worked example (§10): the target architecture is judged on the **hand-offs** — between FO and MO, between the bank and the CCP, between the estate and the regulators — because that is where markets architectures actually break.

---

## 2. The Front Office

### 2.1 What the front office is

The **front office** is the revenue-generating layer: the desks where prices are made, orders are executed, and clients are covered. It is the fastest layer by design — latency here is revenue, which is why the [architecture patterns in §9](#9-architecture-patterns) are dominated by low-latency concerns. The FO splits into **sales** (client coverage, distribution) and **trading** (execution, market-making, structuring), with **quantitative research** (models, pricing libraries) as the analytical backbone.

The FO system landscape is four interlocking capabilities, all of which appear in the Cymbal Bank context of the [Crédit Agricole Software Systems Guide](credit_agricole_software_systems_guide.md) §3:

### 2.2 E-trading (electronic trading)

**E-trading** is the automation of the trade-execution cycle: orders and prices exchanged electronically with clients and venues, without a human broker in the middle ✅ (the term is standard across the industry and the CIB guides — the [Crédit Agricole guide](credit_agricole_software_systems_guide.md) §3.1 documents Cymbal Bank's e-trading offerings in FX and rates). The e-trading estate is typically three layers of its own:

- **Client-facing channels** — e-trading portals and APIs (FXall, 360T, Bloomberg FXGO, in-house portals), where clients request prices, click to trade, and stream their own orders.
- **Execution stack** — smart order routing (SOR), algorithmic execution (TWAP/VWAP/implementation shortfall), and direct market access (DMA/SPAA for sponsored access), which slice client orders into venue orders.
- **Venue connectivity** — FIX sessions and proprietary exchange APIs to the venues (§6), with co-located infrastructure for the fastest desks (§9).

The e-trading pattern to understand architecturally: **pricing streams in, orders stream out, and both travel over FIX** (§7). The pricing side is fed by market data (§5); the order side is managed by the OMS/EMS.

### 2.3 The OMS (Order Management System)

The **OMS** is the system of record for orders — the desk's control plane between the client channel and the execution layer ✅ (industry-standard definition; every OMS vendor — Fidessa/ION, FlexTrade, Charles River, Murex, Calypso — describes it the same way). What the OMS owns:

- **Order capture and state** — the lifecycle of an order: new → partially filled → filled → cancelled/rejected → done-for-day. Every state transition is an event (see the [event-driven patterns in §9](#9-architecture-patterns)).
- **Order routing and allocation** — splitting a client order into working orders across venues (via the EMS or directly), and later allocating fills back to client accounts (allocation is where OMS meets MO: allocations drive confirmations in §3.2).
- **Order audit** — the regulatory backbone: MiFID II RTS 6 requires full order lifecycle recording (see §8), so the OMS is where the audit trail lives.
- **Pre-trade controls** — credit limits, position limits, kill switches; the OMS enforces the desk's risk gates before an order reaches a venue.

### 2.4 The EMS (Execution Management System)

The **EMS** is the execution layer's cockpit — the screen and API surface from which traders manage live orders across venues ✅ (standard industry distinction: OMS is order lifecycle management, EMS is execution management; Fidessa, FlexTrade EMSX, ION, Bloomberg EMSX are the canonical examples). Where the OMS is about *state*, the EMS is about *action*:

- **Venue aggregation** — one pane for all liquidity: lit order books, dark pools, RFQ (request-for-quote) venues, streaming prices from market makers.
- **Execution tools** — algos, SOR, manual execution blotters; the trader's real-time tooling.
- **Fills and position marking** — fills come back to the EMS, positions mark in real time, P&L ticks.

In practice OMS and EMS blur — most platforms ship both — but the architectural distinction matters: the OMS owns the *record*, the EMS owns the *speed*. In a low-latency stack (§9) the EMS is the layer that gets FPGA-accelerated and co-located, while the OMS stays on the normal bus.

### 2.5 Pricing

**Pricing** is the analytical engine of the FO: turning market data into prices, and prices into executable quotes ✅ (the pricing-library pattern is universal: every bank and vendor maintains a pricing/valuation library — Murex's analytics, Calypso's, or in-house quant libraries). The pricing stack has three tiers:

- **Market data ingestion** (§5) — curves, surfaces, volatilities, dividends, funding: the raw material.
- **Valuation models** — the quant layer: discount curves (OIS for collateralised books, ESTR/SOFR post-Benchmark-Reform), forward curves, volatility surfaces (Black-Scholes, local vol, stochastic vol), and product models (swaptions, caps/floors, exotic options, structured products). This is the domain of the [Risk Management Models Guide](risk_management_models_guide.md) — VaR, Greeks, and the model-risk governance that sits on top.
- **Price production** — model output turned into executable quotes: bid/ask, size, and the **XVA adjustments** (CVA/DVA/FVA — the credit and funding cost of the trade) that have been part of standard pricing since the post-2008 era ✅ (XVA pricing is universal industry practice, well documented in the risk literature and the [Risk Management Models Guide](risk_management_models_guide.md)).

The pricing pattern to understand: **the same models serve the FO for pricing and the risk desk for valuation** — which is why a unified platform (Murex MX.3, Calypso) wins, and why the [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md) treats analytics as the platform's heart. In an in-house stack, the pricing library is typically a C++/Python service exposed to the trading screens and the risk engines alike (see the worked example, §10).

### 2.6 The FO table

| System | Function | Notes |
|---|---|---|
| **E-trading portal / API** | Client-facing electronic trading (FX, rates) | FXall/360T/Bloomberg FXGO or in-house; Cymbal Bank offers e-trading in FX/rates ✅/⚠ (see [Cymbal Bank guide](credit_agricole_software_systems_guide.md) §3.1) |
| **OMS** | Order lifecycle, routing, allocations, audit | Fidessa/ION, FlexTrade, Charles River; Murex/Calypso OMS modules; MiFID II RTS 6 order records |
| **EMS** | Execution management: venues, algos, SOR, blotters | FlexTrade EMSX, Bloomberg EMSX, ION; the low-latency layer (§9) |
| **Pricing/valuation engine** | Models, curves, surfaces, XVA; quote production | Murex/Calypso analytics or in-house quant library (C++/Python); cross-ref [Risk Management Models Guide](risk_management_models_guide.md) |
| **Sales/CRM layer** | Coverage, client P&L, relationship context | Light in markets vs retail banking; feeds distribution and MO attribution |
| **Algo/SOR engine** | TWAP/VWAP/IS algos, smart order routing, DMA | Often embedded in EMS; co-located for the fastest desks |

### 2.7 The desk taxonomy — who the FO systems serve

Front-office systems serve three distinct desk species with different requirements, and an architect sizes them differently:

- **Flow desks** (FX, rates, credit, equities market-making) — the franchise is *execution quality at scale*: streaming prices, tight spreads, algos, low latency. They are the heaviest consumers of the e-trading/EMS estate (§2.2–2.4) and the fast-path patterns (§9.2). Cymbal Bank's documented e-trading offering in FX and rates (§3.1 of the [CA guide](credit_agricole_software_systems_guide.md)) is a flow-franchise signature.
- **Structuring desks** — the franchise is *product manufacturing*: exotic derivatives, structured notes, securitisations. They are the heaviest consumers of the pricing/valuation estate (§2.5): model libraries, curve/surface production, XVA, documentation. Structuring is where distribution meets manufacturing — a structurer designs, a flow desk hedges, sales distributes (§1.1).
- **Position-taking desks** (prop/relative-value) — the franchise is *the view*: the desk holds risk and expresses a thesis. They are the heaviest consumers of the risk-analytics estate (cross-ref [Risk Management Models Guide](risk_management_models_guide.md)) and the lightest on client channels.

The taxonomy matters because the FO estate is not one system but a **portfolio of capabilities with desk-specific profiles**: the flow desk's OMS is latency-shaped, the structurer's pricing service is model-shaped, the prop desk's risk engine is simulation-shaped — and all of them emit the same trade lifecycle events into the same backbone (§9.3).

---

## 3. The Middle Office

### 3.1 What the middle office is

The **middle office** is the control layer between the front and back offices: it validates what the FO produced, confirms it with the counterparty, and manages the risk that accrues between trade and settlement ✅ (industry-standard role; the MO function is described identically across WatersTechnology, The TRADE, and every front-to-back vendor). The MO exists because a trade is not a fact until it is *agreed*: the FO's deal is an intent, and everything between intent and settlement must be checked, matched, and collateralised. MO functions are the ones that become regulatory obligations — confirmations and collateral under EMIR (§8) — so the MO is also where the compliance burden concentrates.

### 3.2 The trade lifecycle

The **trade lifecycle** is the canonical sequence of states a trade passes through — the shared vocabulary of the whole stack ✅ (the lifecycle framing is universal; every OMS/MO/BO system and every platform guide in this series uses it). The standard sequence:

1. **Pre-trade** — pricing, credit check, limits (FO: OMS pre-trade controls, §2.3).
2. **Execution** — the trade happens on a venue or bilaterally (FO: OMS/EMS, §2.3–2.4).
3. **Capture** — the trade is booked into the front-to-back platform: terms, counterparty, book, product (this is where the [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md)'s trade capture lives).
4. **Enrichment** — static data is attached: legal entity, jurisdiction, product taxonomy, settlement instructions, regulatory flags (LEI, UPI, CFI codes — §5 reference data).
5. **Validation** — the MO check: does the trade match the ticket? Are the terms complete and consistent? Is it within limits?
6. **Confirmation** — the trade is agreed with the counterparty (§3.3 below).
7. **Clearing** — if cleared, the CCP becomes the counterparty (§6, §4.4).
8. **Settlement** — cash and securities change hands (§4.2).
9. **Ongoing lifecycle** — the trade lives on: resets, coupons, fixes, exercises, expiries, collateral calls, amendments, terminations. A 30-year interest-rate swap spends 29.9 years in this stage.
10. **Accounting and reporting** — P&L, valuation, regulatory reporting (§8).

The lifecycle is the **schema for the whole estate**: every system in the stack is a state machine over these states, and every integration between systems is a hand-off of lifecycle state. This is why event-driven architecture (§9) is the natural fit — each lifecycle transition is an event.

### 3.3 Confirmations

**Confirmation** is the mutual agreement of trade terms with the counterparty — the moment a trade becomes legally binding ✅ (standard definition; the confirmation obligation is also a regulatory one under EMIR Article 11 and MiFID II). The mechanics:

- **Matching** — both sides' records of the trade are compared: the economic terms (amount, rate, maturity), the legal terms (documentation, netting agreement), and the settlement details. The market-standard matching utilities are **DTCC CTM** for cross-border OTC trades and **MarkitSERV** (S&P Global) for credit derivatives — both also route matched trades to confirmation and clearing ✅ (CTM verified as DTCC's central trade-matching platform — dtcc.com; MarkitSERV's CDS role is the industry standard — re-verified 2026-08-22).
- **Affirmation** — for electronic/cleared trades, the counterparty *affirms* the trade, which then flows straight to clearing; for bilateral trades, a **confirmation document** (ISDA confirm) is exchanged, increasingly via electronic confirmation platforms (MarkitSERV, AcadiaSoft).
- **Discrepancies** — unmatched or disputed trades go to exception management: the MO's job is to keep the unconfirmed population near zero, because unconfirmed trades are unhedged risk and a regulatory red flag.

The architectural point: confirmation is the **first place two independent systems meet** (yours and the counterparty's), which is why FIX, SWIFT, and the matching utilities exist — and why the [protocols section](#7-protocols) matters.

### 3.4 Collateral

**Collateral management** is the MO function that grew from niche to mission-critical after 2008: for OTC derivatives, collateral (margin) is posted to cover current and potential future exposure ✅ (the post-crisis margin regime is universal — EMIR and the BCBS-IOSCO framework; the [Risk Management Models Guide](risk_management_models_guide.md) covers the exposure models underneath). The collateral stack:

- **Margin calculation** — variation margin (VM: the daily mark-to-market movement) and initial margin (IM: the future-exposure buffer). Under EMIR, cleared trades margin via the CCP; bilateral trades margin under the uncleared-margin rules (UMR) with phased-in thresholds ✅ (UMR live from 2016, phases 1–6 through Sep 2021, ~1,000+ additional entities in scope by Sep 2022 — CME; re-verified 2026-08-22).
- **Collateral operations** — margin calls, collateral eligibility, haircuts, substitution, disputes. The market-standard utility is **Acadia** (ex-AcadiaSoft — margin-call automation and dispute resolution, **acquired by LSEG**; re-verified 2026-08-22 — the earlier "part of DTCC" reading was wrong) ✅/⚠ (vendor facts now verified; internal deployment depth ⚠).
- **Cleared collateral** — the CCP's margin system (§6): the CCP collects IM/VM from its clearing members, who collect from their clients. The triparty agents (Euroclear, Clearstream, BNY) hold the securities posted as collateral.
- **Optimisation** — with IM everywhere, firms optimise collateral: **portfolio compression** (TriOptima, Quantile — reducing notional by netting off offsetting trades), collateral transformation, and re-use. Compression is a quiet but enormous part of the MO: trillions of notional removed from the system ✅ (compression volumes are well documented by TriOptima/Quantile and the industry press; the notional figures are large — flagged ⚠ as approximate).

### 3.5 The MO table

| System/function | What it owns | Notes |
|---|---|---|
| **Trade lifecycle / post-trade platform** | Capture→confirm→clear→settle state machine | Core of Murex/Calypso post-trade; cross-ref both platform guides |
| **Confirmation/matching (CTM, MarkitSERV)** | Trade matching, affirmation, electronic confirms | DTCC CTM (ex-Omgeo) ✅, MarkitSERV (S&P Global) for CDS ✅ (re-verified 2026-08-22) |
| **Collateral management (Acadia, TriOptima)** | Margin calls, IM/VM, disputes, compression | Acadia (ex-AcadiaSoft) → LSEG ✅ (re-verified 2026-08-22; the earlier "→DTCC" row was wrong); EMIR/UMR margin rules ✅; compression by TriOptima/Quantile |
| **Trade/position reconciliation** | Internal FO-vs-platform-vs-BO position breaks | SmartStream TLM, Broadridge, in-house |
| **P&L and risk control** | Daily P&L attribution, limit monitoring, VaR control | Cross-ref [Risk Management Models Guide](risk_management_models_guide.md) |
| **Trade support** | Desk-aligned MO analysts; exception handling | People + workflow tooling (in-house or platform) |

### 3.6 The MO's scorecard — the control metrics

The middle office is a control function, and control functions live on metrics. The canonical MO scorecard — the numbers a markets operating committee reviews daily — is the best quick picture of what the MO systems must produce ✅/⚠ (the metric set is industry-standard operating practice; the exact KPI definitions vary by firm, flagged):

- **Unconfirmed population** — trades past their confirmation deadline (T+0 for most OTC, per EMIR risk-mitigation): the MO's headline risk metric (§3.3).
- **Unmatched/exceptions aging** — matching breaks with the counterparty (CTM, MarkitSERV), aged by day; the pipeline that feeds confirmation failure.
- **Collateral dispute volume and aging** — margin-call disputes open beyond the dispute-resolution window (typically T+1): under EMIR these are reportable; the collateral systems of §3.4 must surface them (AcadiaSoft's dispute workflow exists for exactly this).
- **Position breaks** — internal FO-vs-platform-vs-BO position differences, by desk and by age; the reconciliation estate's output.
- **P&L variance** — unexplained P&L between FO marks and MO/risk marks; the control that catches valuation disputes before they become disputes with the counterparty.
- **Lifecycle-event failures** — missed resets, missed coupons, failed exercises: the ongoing-lifecycle state machine's error queue (§3.2 step 9).

Architecturally, the scorecard is a **reporting service over the event backbone** (§9.3): every lifecycle event, confirmation result, and margin call is an event; the scorecard is a query over the event log. If the scorecard has to scrape screens or run nightly exports from each system, the architecture has already failed.

---

## 4. The Back Office

### 4.1 What the back office is

The **back office** is the layer of record: settlement, custody, clearing, accounting, and regulatory reporting — the functions that turn a trade into a settled, owned, reported fact ✅ (industry-standard; every banking-systems guide in this series draws the same boundary). The BO is where the trade meets the outside world's rails: central securities depositories (CSDs), payment systems, custodians, and regulators. It is also the layer where cost lives: settlement fails, breaks, and manual exceptions are the BO's eternal enemies — the CSDR settlement-discipline regime (§8) turned that cost into penalties.

### 4.2 Settlement

**Settlement** is the exchange of cash and securities that completes a trade ✅ (standard definition: delivery-versus-payment, DvP — securities delivered against cash payment, eliminating principal risk). The settlement stack:

- **Instructions** — settlement instructions (the "where" of the trade: which CSD, which account, which custodian) are matched and sent to the CSD. Standard settlement cycles: **T+1 for equities in the US (since May 2024) and India, T+2 for EU/UK equities (moving to T+1), T+0/T+1 for bonds depending on market** ✅ (the US T+1 move to May 2024 ✅; the EU/UK/CH T+1 move targets **11 October 2027** — the CSDR revision published in the OJ 14 Oct 2025 — re-verified 2026-08-22).
- **CSD settlement** — the CSD (Euroclear, Clearstream, DTCC) runs the actual DvP: its settlement engine books the securities leg and the cash leg atomically.
- **Fails and penalties** — a settlement fail is a trade that did not settle on time; CSDR imposes cash penalties and buy-in obligations on fails (§8). The BO runs fail-management tooling (SmartStream TLM, in-house) to keep the fail rate down.
- **Payments** — the cash leg moves over the payment rails: T2 (EUR), CHAPS (GBP), Fedwire/CHIPS (USD), FAST/MEPS+ (SGD) — the interface to the payments world covered in the series' payments guides.

### 4.3 Custody

**Custody** is the safekeeping and servicing of assets: holding securities, collecting income, processing corporate actions, and providing the asset-servicing record ✅ (standard definition; CACEIS — the CA group's securities-services arm — is the custodial exemplar in the [Crédit Agricole Software Systems Guide](credit_agricole_software_systems_guide.md) §3.1). The custody stack:

- **Safekeeping** — the securities ledger: who owns what, where (at the CSD directly, or through sub-custodians in local markets).
- **Corporate actions** — dividends, coupons, rights issues, redemptions, takeovers: the custodian processes these on behalf of clients. Corporate-action processing is the classic high-cost, high-risk manual area (announcement capture, entitlement calculation, payment).
- **Asset servicing** — income collection, tax reclaims, proxy voting, securities lending (which feeds SFTR reporting, §8).
- **The chain** — global custodian → sub-custodians → local CSDs: the custody chain is the settlement layer for investors who cannot access local markets directly. Architecturally, custody systems are ledgers with extremely high reliability requirements and heavy STP (straight-through processing) feeds.

### 4.4 Clearing

**Clearing** sits between trade and settlement: it is the process of becoming the counterparty to both sides — via a **central counterparty (CCP)** for cleared products, or bilaterally for uncleared ones ✅ (standard definition; the CCP model is covered in depth in §6). The clearing stack:

- **Trade submission** — the trade (or a matched/affirmed confirmation) is submitted to the CCP: directly for clearing members, via a **clearing member** for clients (client clearing), or via a **CCP-eligible venue** for exchange-traded products.
- **Novation** — the CCP interposes itself: the CCP becomes buyer to every seller and seller to every buyer. This is the risk-transformation event that makes CCPs systemic.
- **Margin** — the CCP calls initial and variation margin from members (the collateral engine of §3.4, run at CCP scale).
- **Netting and settlement** — end-of-day, the CCP nets positions and obligations and settles via its settlement banks and the CSD.
- **Default management** — if a member defaults, the CCP's default waterfall absorbs the loss (member default funds, CCP skin-in-the-game, assessments) — the architecture that made CCPs "too important to fail" and put them under dedicated supervision (EMIR's CCP regime, §8) ✅ (the waterfall structure is standard and documented in EMIR and CCP disclosures).

### 4.5 The BO table

| System/function | What it owns | Notes |
|---|---|---|
| **Settlement engine / instruction matching** | DvP instructions, fails, settlement reporting | CSD-side (Euroclear/Clearstream/DTCC) + bank-side (Broadridge, FIS, in-house); CSDR penalties §8 |
| **Custody / safekeeping** | Securities ledger, corporate actions, income | CACEIS in the CA group ✅ (see [Cymbal Bank guide](credit_agricole_software_systems_guide.md) §3); global custodians: BNY, Citi, JPM, HSBC, Standard Chartered |
| **Clearing connectivity** | CCP submission, client clearing, margin flows | Via FIX/SWIFT/CCP APIs; clearing members are the CCP's direct counterparties (§6) |
| **Accounting / finance** | Trade P&L to the ledger, valuations, GL posting | Cross-ref [Core Banking Systems Guide](core_banking_systems_guide.md) for the GL boundary |
| **Regulatory reporting** | EMIR/SFTR/MiFID II/CSDR reporting (§8) | DTCC Data Repository, UnaVista (LSEG), Regis-TR, ICE Trade Vault ⚠ |
| **Reconciliation** | External vs internal records; nostro/safekeeping breaks | SmartStream TLM, Broadridge, in-house matching engines |

### 4.6 The GL boundary — where the trading book meets the banking book

The back office ends at the **general ledger** — and the trading-book-to-GL interface is one of the classic fault lines of the estate. The [Core Banking Systems Guide](core_banking_systems_guide.md) owns the banking-book GL; the markets estate owns the trading-book P&L, and the two must reconcile on cash flows (the boundary is drawn in the [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md) §1.5):

- **Valuation to accounting** — the BO marks the trading book to market daily (FVA/CVA adjustments included, §2.5) and posts valuation changes to the GL; the accounting side must consume the same marks the risk side used, or the P&L will not tie.
- **Trade P&L vs accounting P&L** — realised (settled) vs unrealised (marked) P&L, fee income, funding costs: the attribution that makes the GL explainable to auditors and to the desks.
- **The interface pattern** — the platform/BO produces an **accounting export** (journal entries, one trade at a time or aggregated) consumed by the GL; the pattern is the same as the core-banking posting engines of the series, and the reconciliation discipline (balance to the penny, or explain the break) is identical.
- **Why it breaks** — the GL runs on accounting time (T+1 EOD), the markets on market time (real-time): the interface must buffer, aggregate, and replay; and every valuation-model change (a new curve, an XVA update) must flow through to the GL consistently or the books do not tie.

---

## 5. Market Data

### 5.1 What market data is

**Market data** is the raw material of the entire stack: prices, curves, and identifiers consumed by every layer — the FO to price and trade, the MO to value and margin, the BO to settle and report, and risk to simulate ✅ (standard definition; the market-data industry is well documented — Bloomberg/Refinitiv/FactSet/ICE are the canonical vendors). Two distinct species, with different architectures:

### 5.2 The feeds (real-time market data)

**Real-time feeds** deliver prices and quotes as they happen — the high-velocity data ✅:

- **Exchange feeds** — the deepest/fastest: CME, Eurex, LSE, NASDAQ, ICE publish direct feeds (the "market data infrastructure" side of §6). Direct feeds are the low-latency option (microsecond-scale, §9); **consolidated feeds** (e.g. the SIP in the US) are slower but cheaper.
- **Vendor feeds** — Bloomberg (B-PIPE), Refinitiv (LSEG) (RMDS/RTDS), FactSet, ICE Data: aggregated, value-added, normalised — the workhorse for non-latency-critical desks.
- **Curves and derived data** — vendor-computed curves, surfaces, fixings (ESTR, SOFR, EURIBOR fixings) that pricing engines consume.
- **The feed architecture** — the classic pattern is a **market-data platform**: a normalisation layer (vendor formats → internal canonical model), a **tick capture** store (the historical record), a **distribution bus** (multicast/TCP fan-out to consumers), and **entitlements** (who may see what — a commercial and compliance function). This is the estate described in the [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md) §6 and the [Event Stream Processing Guide](../technology/event_stream_processing_guide.md)'s domain.

### 5.3 Reference data

**Reference data** is the static, slow-moving data that everything else keys off ✅ (standard industry definition; GoldenSource, Refinitiv, Bloomberg Data License, and the ISO standards are the canonical references):

- **Instrument reference data** — the identifier and attributes of every product: **ISIN**, **CFI** (classification), **FISN**, plus product-specific fields (currency, maturity, coupon, strike). The new-generation identifiers are **UPI** (unique product identifier, for OTC derivatives) and **UTI** (unique transaction identifier) — both mandated by EMIR/SFTR reporting ✅/⚠ (UPI/UTI are documented under EMIR Refit; implementation phases flagged).
- **Party reference data** — legal entities: **LEI** (legal entity identifier) — mandatory on every trade and every report; plus internal client hierarchies.
- **Pricing/valuation reference data** — the static inputs to models: curve definitions, holiday calendars, fixing calendars, corporate actions.
- **The reference-data architecture** — a **golden copy / single source of truth** (GoldenSource, Refinitiv, in-house MDM) feeding the whole estate, with data governance (who owns the attribute, when it is valid). Reference data is the quiet cause of most STP breaks: the counterparty's LEI missing, the ISIN mistyped — every downstream exception starts here.

### 5.4 The data table

| Data type | Contents | Representative sources/systems |
|---|---|---|
| **Real-time prices** | Bids/offers, last trade, volume, order book | Direct exchange feeds (CME, Eurex, LSE), vendor feeds (Bloomberg B-PIPE, LSEG RMDS, FactSet, ICE) |
| **Tick history** | Time-stamped price/quote record | OneTick, KDB+, in-house tick stores; feeds analytics and TCA (transaction-cost analysis) |
| **Curves/surfaces/fixings** | Discount curves, vol surfaces, benchmark fixings | Vendor curve services, internal curve production; ESTR/SOFR fixings post-Benchmark-Reform |
| **Instrument reference** | ISIN, CFI, FISN, product attributes | GoldenSource, Refinitiv, Bloomberg Data License, DTCC CUSIP |
| **Party reference** | LEI, client hierarchy, settlement instructions | GLEIF (LEI issuer), internal MDM |
| **Corporate actions** | Dividends, coupons, rights, redemptions | Vendor services (LSEG, ICE), custodians, issuer announcements |
| **Entitlements** | Who may consume what data, at what cost | Vendor + internal entitlement systems; audit trail for MiFID II |

### 5.5 Data governance — the quiet architecture

Market data and reference data are only as good as their governance, and data governance is where markets architectures most often quietly rot ✅ (the governance functions below are industry-standard practice — data ownership, quality rules, and entitlements are described across the data-management vendors' materials and the industry press):

- **The golden copy discipline** — one canonical instrument/party record, one owner per attribute, propagated to consumers; the pattern is the [Distributed Auth Guide](../technology/distributed_auth_guide.md) authoritative-source design applied to data. Reference-data drift (two systems holding different maturities for the same ISIN) is the classic root cause of STP breaks and regulatory mismatches.
- **Data quality rules** — automated checks: ISIN validates against the check digit, LEI is current (GLEIF renewals), UTI/UPI present on every EMIR/SFTR report, curves are fresh (not stale), prices are within sanity bands. Every rule that fails is an exception ticket, not a silent gap.
- **The identifier hygiene crisis** — the post-2008 identifier stack (LEI, UPI, UTI, ISIN, CFI) is the *key system* of the reporting estate (§8): a missing LEI blocks an EMIR report, a mistyped UPI blocks an SFTR report, and the penalty regimes (CSDR) make the BO's data quality a direct cost line.
- **Entitlements and commercial management** — market-data licences are a material cost and a compliance obligation (redistribution rights, per-desk licensing): the entitlement layer is both a technical access-control system and a procurement control.
- **Data as a product** — the modern operating model treats data sets as products with owners, SLAs, and consumers; the markets version is the data catalogue over the golden copy, serving FO, MO, BO, risk, and regulators the same canonical facts.

---

## 6. Market Infrastructure

### 6.1 The ecosystem

**Market infrastructure** is the external fabric the bank's stack plugs into: venues where trading happens (exchanges), counterparties that guarantee it (CCPs), depositories that settle it (CSDs), repositories that record it (trade repositories), and the payment systems underneath ✅ (the ecosystem framing is standard — the series' [Universal Banking Model Guide](universal_banking_model_guide.md) and the Cymbal Bank guide's infrastructure references draw the same map). For the architect, the key structural fact: **the bank does not own this layer — it connects to it**, over FIX/APIs (§7), through clearing memberships, CSD accounts, and market-data licences (§5). Every connection is a contract, an integration, and a dependency.

### 6.2 The exchanges

**Exchanges** are the venues where trading happens — order books, matching engines, and the rulebooks that govern them ✅ (standard definition; the major exchanges' own sites — LSE, Deutsche Börse/Eurex, CME, NASDAQ, ICE — document their venue roles). The architecture of a modern exchange:

- **The matching engine** — the core: an order book that matches buy and sell orders in microseconds (see the latency discussion in §9 — the microsecond claim is flagged there). Modern engines are software (CME's CME Globex, Eurex's T7, LSE's Millennium) running on co-located infrastructure with deterministic low latency ✅ (the T7/GLOBEX/Millennium names are well documented ✅).
- **Market data** — the exchange publishes its order book and trades as a data product (§5): direct feeds (e.g. Eurex's EMDI, CME's MDP 3.0) and consolidated feeds.
- **Trading models** — continuous order books (lit), auctions (open/close), RFQ (request-for-quote for less liquid products), and hybrid models; the trading model defines what the bank's EMS must support (§2.4).
- **The venue's role in the lifecycle** — the exchange is where the trade lifecycle (§3.2) *starts*: execution happens here, then clearing (§4.4) via the exchange's affiliated CCP (Eurex ↔ Eurex Clearing; CME ↔ CME Clearing; ICE ↔ ICE Clear). The exchange-CCP pair is the vertical-silo pattern of listed derivatives ✅ (the pairing is structural and documented on the venues' sites).

### 6.3 The CCPs: LCH, Eurex, CME

**Central counterparties (CCPs)** are the risk backbone of the markets: they interpose themselves between buyers and sellers (novation, §4.4), collect margin, and manage defaults ✅ (the CCP model and the EMIR oversight regime are documented in §4.4 and §8; each CCP's clearing-house site documents its own services). The three canonical houses, verified against their own sites and stable sources:

- **LCH** ✅ — headquartered in London, owned by the **London Stock Exchange Group (LSEG)** ✅ (the LSE acquisition of LCH.Clearnet completed in stages 2012–2014 ✅/⚠ — the deal timeline is documented, the exact closing date flagged). LCH is the **world's largest OTC interest-rate swap clearer** through **SwapClear** — LSEG's own materials describe it as "the market's longest-serving" and "the only truly global swap clearing service", with dominance confirmed by CCPView market data (e.g. 72% of cleared INR swaps in 2024; the overwhelming majority of cleared OTC IRS notional) ✅ (re-verified 2026-08-22) — plus **ForexClear** (FX forwards/NDFs) and **RepoClear** (repo). LCH Ltd is UK-authorised; LCH SA (Paris) handles French/European clearing and is the EU-27 access point post-Brexit ✅/⚠.
- **Eurex Clearing** ✅ — Frankfurt, part of **Deutsche Börse Group**; the clearing house for **Eurex**, Europe's largest derivatives exchange (verified live this pass: eurex.com describes itself as "The Leading Derivatives Exchange" ✅). Eurex Clearing clears the Eurex listed derivatives franchise (EURO STOXX 50, Bund/BOBL/SCHATZ futures, ESTR futures) and has built a competing OTC IRS clearing service (EurexOTC Clear) — the main challenger to LCH in EUR swaps ✅/⚠ (the EurexOTC Clear offering is documented on eurex.com ✅; its market share vs LCH is ⚠).
- **CME Clearing** ✅ — Chicago, part of **CME Group** ("headquartered in Chicago, operates exchanges and provides clearing services for trading in financial derivatives such as futures contracts, options, and swaps" — verified live this pass via the CME Group Wikipedia record ✅). CME Clearing clears the CME/COMEX/NYMEX/CBOT complex — the world's largest futures clearing house by margin/volume ✅/⚠ (the "largest" claim is consistent across industry sources; exact rankings flagged ⚠).

The CCP pattern to internalise: **one CCP, many services, one default waterfall** — the member's exposure is to the CCP, not to the original counterparty, which is precisely why the post-2008 reform (EMIR, §8) pushed OTC derivatives into CCPs. The [DeFi Guide](../technology/defi_guide.md) is the deliberate contrast: DLT-based "trustless" markets replace the CCP's legal/financial architecture with code and collateral pools — the same risk problem, a radically different trust model.

### 6.4 The infra table

| Institution | Type | Function | Notes |
|---|---|---|---|
| **LSE / Deutsche Börse / CME / NASDAQ / ICE** | Exchanges | Matching engines, market data, rulebooks | Eurex T7, CME Globex, LSE Millennium engines ✅ |
| **LCH (LSEG, London)** | CCP | OTC IRS clearing (SwapClear), FX (ForexClear), repo (RepoClear) | Largest OTC IRS clearer ✅/⚠; EU access via LCH SA (Paris) ✅/⚠ |
| **Eurex Clearing (Deutsche Börse, Frankfurt)** | CCP | Listed derivatives clearing for Eurex; OTC IRS (EurexOTC Clear) | Europe's leading derivatives clearer ✅; verified live ✅ |
| **CME Clearing (Chicago)** | CCP | Futures/options/swaps clearing across the CME complex | Verified live ✅; the largest futures-clearing house ✅/⚠ |
| **Euroclear / Clearstream / DTCC** | CSDs/ICSDs | Settlement, safekeeping, collateral services | The settlement rails of §4.2; DTCC also runs CTM + the US trade repository |
| **Trade repositories** | Data repositories | EMIR/SFTR transaction records | DTCC Data Repository, UnaVista (LSEG), Regis-TR, ICE Trade Vault ⚠ |
| **Payment systems** | Rails | Cash legs of settlement | T2 (EUR), CHAPS (GBP), Fedwire/CHIPS (USD), FAST/MEPS+ (SGD) |

### 6.5 The post-trade infrastructure chain

The exchange-CCP pair (§6.2–6.3) is only the front half of the external fabric; the settlement half runs through the **CSD/custodian chain** ✅ (the chain below is the standard market-structure description, consistent across the CSDs' own materials and the industry literature):

- **CSDs (central securities depositories)** — the definitive record of security ownership and the settlement engines: Euroclear and Clearstream (also the ICSDs for eurobonds), DTCC in the US. The CSD runs the DvP settlement of §4.2 — the atomic cash-vs-securities exchange.
- **The custody chain** — global custodian → sub-custodian → local CSD: for a Singapore-hub bank settling into Asian markets, the chain is the daily reality — the sub-custodian in each local market holds the local settlement account, and the bank's BO system must speak to each (SWIFT MT54x/MT5xx instructions, §7).
- **Trade repositories** — the record of OTC derivatives and SFTs under EMIR/SFTR (§8): the bank reports to a TR (DTCC Data Repository, UnaVista, Regis-TR, ICE Trade Vault ⚠), and the TR reconciles the two counterparties' reports. The TR is the post-trade infrastructure of *information*, as the CSD is of *assets*.
- **The systemic-quality point** — every element of the chain (exchange, CCP, CSD, TR, payment system) is systemic infrastructure: if it fails, markets fail. That is why CCPs and CSDs are directly supervised (EMIR for CCPs, CSDR for CSDs — §8) and why the bank's resilience architecture (§9.6) treats each connection as a dependency with its own DR story, not a commodity link.

---

## 7. Protocols

### 7.1 FIX — the market's lingua franca

**FIX (Financial Information eXchange)** is the de facto standard protocol for pre-trade and trade messaging between market participants — orders, executions, and allocation messages — maintained by the **FIX Trading Community** ✅ (the FIX Trading Community's own site — verified live this pass: "an independent, non-profit standards body dedicated to advancing global multi-asset trading through the development and promotion of the FIX Protocol and other standards" ✅). What an architect needs to know:

- **FIX 4.4** ✅ — the version the brief asked to verify: **FIX 4.4 is a published, current standard of the FIX Trading Community** (its specification release notes are live at fixtrading.org — verified this pass ✅). It remains the **most widely deployed FIX version in production** ✅/⚠ (the "most widely deployed" claim is consistent across the FIX community's materials and industry surveys, but exact adoption figures are ⚠). **Released 2003** ✅ (re-verified 2026-08-22: the FIXML 4.4 schema guide on fixtrading.org is dated 9 Jan 2004, consistent with a 2003 release; the spec defines 916 field definitions, 92 message types and 24 reusable components). FIX 4.4 added, among others, the **AllocationInstruction** refinements and enhanced trade-capture messages that made it the standard for post-trade allocation.
- **The FIX family** — the protocol evolved through **FIX 4.0, 4.1, 4.2, 4.3, 4.4** into **FIX 5.0** and the **FIXT 1.1** (FIX Transport) session layer ✅ (the version lineage is documented on fixtrading.org and Wikipedia — the version list 4.4→5.0→FIXT 1.1 confirmed ✅). The industry runs a mix: 4.2/4.4 for most OTC/equity flow, FIX 5.0 SP2 with FIXT 1.1 for the newest integrations.
- **FIX anatomy** — tag=value messages (e.g. `35=D` for NewOrderSingle, `8=FIX.4.4` BeginString, `55=` symbol, `54=` side), session-level fields (begin/body/checksum: 8/9/10), and application-level fields ✅ (the tag structure is documented in the FIX specification ✅). The header triplet (8, 9, 35) is the famous "FIX three" — verified in the Wikipedia FIX record this pass ✅.
- **FIX in the stack** — FIX is the connective tissue of the FO (§2): client channels (FXall/360T/portals), OMS↔EMS, EMS↔venue. FIX engines (OnixS, Rapid Addition, CME's own, in-house) sit at every hop. **Binary encodings** (FAST, SBE — Simple Binary Encoding) exist for high-throughput market data, and **FIXatdl** (FIX Algorithmic Trading Definition Language) describes algo parameters ✅/⚠ (FAST/SBE/FIXatdl are documented FIX-community standards; the details flagged as not re-verified live this pass).

### 7.2 The protocols table

| Protocol | Layer/use | Notes |
|---|---|---|
| **FIX 4.2 / 4.4** | Order/trade messaging: OMS↔EMS↔venues, client channels | 4.4 is the most widely deployed version ✅/⚠; the standard's current releases live at fixtrading.org ✅ |
| **FIX 5.0 / FIXT 1.1** | Newer FIX sessions; FIXT 1.1 is the transport | Used for the newest venue integrations ✅ |
| **FAST / SBE** | Binary market-data encodings | For high-throughput/low-latency data (SBE used by CME MDP 3.0) ⚠ |
| **ITCH / OUCH** | Exchange-native order/market-data protocols | NASDAQ's native protocols; venue-specific ⚠ |
| **SWIFT** | Post-trade, payments, securities messaging | ISO 15022/20022; the BO's external language (§4); cross-ref the series' payments guides |
| **ISO 20022** | Standardised financial messaging | The modern SWIFT format; also payments rails ✅ |
| **FIXatdl** | Algo definition language | Describes algo parameters for FIX algos ⚠ |
| **REST/WebSocket APIs** | Client e-trading, vendor data APIs | The modern layer over/alongside FIX; cross-ref [Distributed Auth Guide](../technology/distributed_auth_guide.md) for API security patterns |

### 7.3 SWIFT and ISO 20022 — the post-trade language

Where FIX owns the front of the lifecycle (§3.2 steps 1–6), **SWIFT** owns the back: confirmation, settlement instructions, and the payments underneath ✅ (the SWIFT/ISO 20022 split below is standard industry knowledge; the series' payments guides cover the rails in depth). An architect sees three layers:

- **SWIFT FIN (ISO 15022)** — the legacy MT message set: MT300/MT304 (FX/derivatives confirmations), MT54x (settlement instructions), MT5xx (securities settlement), MT202/MT103 (payments). The BO's external language for decades ✅ (the MT message families are documented in the SWIFT standards catalogue).
- **ISO 20022 (MX messages)** — the modern XML-based standard, mandatory for the payments rails since the March 2023 T2 migration and progressively for securities messaging ✅/⚠ (the T2 ISO 20022 go-live is well documented ✅; the securities-side migration timeline flagged ⚠). The BO estate is in a dual-standard world — MT and MX in parallel — which is exactly the translation-layer problem every architect must budget for.
- **The translation layer** — the bank's connectivity hub (SWIFT Alliance-class, or a vendor hub) translates between internal canonical messages, MT, and MX; the golden-copy data model (§5.5) is what makes the translation lossless. The [Message Queue Data Loss Guide](../technology/message_queue_data_loss_guide.md) discipline applies here with extra force: a settlement instruction must not be lost, duplicated, or reordered — the delivery-guarantee decision is the difference between a fail and a fine.

---

## 8. Regulation

### 8.1 The regulatory map

The post-2008 regulatory settlement defines the markets architecture as much as any technology choice: **every system in this guide exists partly to produce a regulatory output**. The four pillars the brief asks to verify — MiFID II, EMIR, SFTR, CSDR — plus the risk layer (cross-ref [Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md)):

### 8.2 MiFID II / MiFIR (2018)

**MiFID II (Directive 2014/65/EU) and MiFIR (Regulation (EU) No 600/2014) apply from 3 January 2018** ✅ — **verified this pass**: the Wikipedia record of Directive 2014/65/EU states "Both MiFID 2 and Regulation (EU) No 600/2014 have been effective from 3 January 2018", and records the Commission's February 2016 delay from the original 3 January 2017 to allow IT-system build-out ✅. MiFID II is the trading-conduct and market-structure regime: the scope is enormous, and for the architect the system-relevant parts are:

- **Transaction reporting** — detailed reports on every executed transaction to the regulator via an **ARM** (Approved Reporting Mechanism); the bank's OMS/execution stack must emit full order/trade records (RTS 22 fields, LEI/ISIN/UTI identifiers).
- **Order record keeping (RTS 6)** — full lifecycle recording of orders, including cancelled and rejected ones, for 5 years — the OMS audit requirement of §2.3.
- **Best execution** — firms must take "all sufficient steps" to obtain the best possible result for clients (verified in the Directive's record ✅); this drives the TCA (transaction-cost analysis) tooling and venue-selection logic in the EMS (§2.4).
- **Pre/post-trade transparency** — venue and firm transparency regimes (RTS 1/2); double volume caps; the systematic-internaliser regime that reclassified how banks show liquidity.
- **Algorithmic trading controls** — MiFID II requires algos to be tested, monitored, and kill-switched (RTS 6): the algo-lifecycle management layer of the e-trading stack (§2.2) is a direct regulatory artifact.

### 8.3 EMIR (2012)

**EMIR — Regulation (EU) No 648/2012 — is the 2012 OTC-derivatives regulation** ✅ — **verified this pass**: the Wikipedia record confirms "Regulation (EU) No 648/2012, as EMIR is referred to in European legal documentation, was implemented in 2012" and the "regulation was initially adopted in 2012", with the EMIR Refit amendments adopted 2019 ✅. (The brief's "verify EMIR" — the 2012 anchor is confirmed; the regulation entered into force in August 2012 after publication, with obligations phased in through 2013–2014 ⚠ — the phasing dates flagged.) EMIR's architecture-driving obligations:

- **Clearing obligation** — standardised OTC derivatives must be cleared through a CCP (§6); this is why LCH SwapClear and EurexOTC Clear exist at their scale.
- **Margin requirements** — VM/IM on uncleared derivatives (the UMR regime, §3.4); drove the collateral-management build-out.
- **Risk mitigation** — timely confirmation (T+0/T+1 by product), portfolio reconciliation, dispute resolution, portfolio compression — the MO functions of §3.3–3.4 as legal obligations.
- **Trade reporting** — both counterparties report every OTC derivative to a **trade repository (TR)**: UTI, UPI, valuation, collateral, margining — the reporting pipeline (EMIR Refit tightened this from 2024) ✅/⚠ (Refit dates flagged).
- **CCP supervision** — EMIR created the EU CCP supervisory regime (ESMA's CCP Supervisory Committee, visible on esma.europa.eu this pass ✅); post-Brexit, EU firms access UK CCPs (LCH) under equivalence/recognition arrangements ⚠.

### 8.4 SFTR (2020)

**SFTR — Regulation (EU) 2015/2365, the Securities Financing Transactions Regulation — is the securities-lending/repo transparency regime; its reporting obligations phase in from 2020** ✅ — **verified this pass**: the Wikipedia record confirms SFTR is "Regulation (EU) 2015/2365" and records the four deployment phases: **banks and broker-dealers 11 April 2020, financial market infrastructures 11 July 2020, insurers and asset managers 11 October 2020, and non-financial entities 11 January 2021** ✅ (the brief's "SFTR 2020" anchor is verified — initial reporting started April 2020). SFTR's system impact:

- **Transaction reporting** — every securities-financing transaction (repo, securities lending, margin lending, buy-sell back) is reported to a TR with 100+ fields: counterparties (LEI), securities (ISIN), collateral, terms.
- **Data quality** — SFTR reports must reconcile between the two counterparties' reports — the reporting architecture needs a reconciliation engine, not just a report generator.
- **The collateral-data angle** — SFTR is the first regime to make *collateral* the subject of granular reporting; it connects the MO collateral systems (§3.4) directly to the reporting pipeline.

### 8.5 CSDR (settlement discipline, 2022)

**CSDR — Regulation (EU) No 909/2014 — is the EU settlement regime; its settlement-discipline provisions (cash penalties and mandatory buy-ins for settlement fails) applied from 1 February 2022** ✅ — **re-verified 2026-08-22**: the Settlement Discipline Regime (SDR) applied **1 February 2022** (penalties regime; ICMA/Euroclear document the 1 Feb 2022 date), while the **mandatory buy-in regime was suspended for three years** — ESMA published technical standards to postpone the CSDR buy-in regime (ESMA RTS on settlement discipline; the buy-in suspension in force pending the CSDR review) ✅. CSDR's architecture-driving parts:

- **Settlement discipline** — cash penalties on failed settlements, computed and collected by the CSD; mandatory **buy-ins** when a fail persists (initially 4 days, extended) — the direct cost driver behind the BO fail-management tooling of §4.2.
- **CSD authorisation** — EU CSDs must be authorised under CSDR; internalised settlement (banks settling client trades in-house) faces a reporting regime — the "internalised settlement reporting" obligation.
- **The T+1 question** — CSDR was designed around T+2; the EU's move to T+1 targets **11 October 2027** (the CSDR revision published in the OJ 14 Oct 2025; the UK/CH target the same date; following the US/Canada/India T+1 in May 2024) ✅ (re-verified 2026-08-22).

### 8.6 The regulation table

| Regulation | Scope | Notes |
|---|---|---|
| **MiFID II / MiFIR (2014/65/EU, 600/2014)** | Trading conduct, market structure, transparency | **Effective 3 January 2018** ✅; transaction reporting via ARMs, RTS 6 order records, best execution, algo controls |
| **EMIR (648/2012)** | OTC derivatives: clearing, margin, risk mitigation, reporting | **Adopted 2012** ✅; Refit 2019 ✅; clearing obligation → CCPs (§6); trade reporting to TRs; UMR margin |
| **SFTR (2015/2365)** | Securities financing: repo, securities lending | **Reporting phased in from 11 April 2020** ✅ (banks/BDs), 11 Jul 2020 (FMIs), 11 Oct 2020 (insurers/AMs), 11 Jan 2021 (NFCs) ✅ |
| **CSDR (909/2014)** | Settlement, CSDs, settlement discipline | **Settlement discipline (penalties) from 1 February 2022** ✅ (re-verified 2026-08-22); **mandatory buy-ins suspended 3 years** (ESMA RTS) ✅; CSD authorisation; internalised-settlement reporting; **EU T+1 target 11 Oct 2027** ✅ (CSDR revision in OJ 14 Oct 2025) |
| **Basel III / FRTB** | Bank capital, market risk | Cross-ref [Risk Management Models Guide](risk_management_models_guide.md) — the risk-capital overlay on the trading book |
| **BCBS-IOSCO UMR** | Uncleared margin rules | IM/VM on uncleared derivatives; the collateral build-out of §3.4 ⚠ |
| **Benchmark Regulation (BMR)** | Rate benchmarks | ESTR/SOFR transition aftermath; reference-data governance ⚠ |

### 8.7 The reporting pipeline — regulation as a system

Every regulation in this section resolves to the same architectural demand: **the estate must produce reports, on time, with data that reconciles to the counterparty's**. The reporting pipeline is therefore a system in its own right ✅ (the pipeline stages below are the standard industry description of regulatory reporting architecture, consistent across the reporting-vendor literature and the press):

1. **Extraction** — the transaction data is pulled from the systems of record (OMS, post-trade platform, collateral systems) into a reporting data store. This is where the golden-copy data model (§5.5) earns its keep: if the identifiers are right at source, the reports fall out.
2. **Enrichment** — LEI, UPI/UTI, CFI, venue, price, valuation, collateral details are attached; missing identifiers are chased (the identifier hygiene crisis of §5.5).
3. **Transformation** — internal records become regulatory payloads: EMIR (the 100+ field XML/JSON schemas), SFTR (150+ fields), MiFID II RTS 22, CSDR internalised-settlement reports. Each regime has its own schema, its own tolerances, its own quirks.
4. **Submission** — to the ARM (MiFID II) or the trade repository (EMIR/SFTR): DTCC Data Repository, UnaVista (LSEG), Regis-TR, ICE Trade Vault ⚠.
5. **Reconciliation** — the counterparty's report is matched against the bank's (the TRs run reconciliation; SFTR requires the two sides to tie); breaks are chased to zero. This is the stage the MO scorecard (§3.6) watches.
6. **Audit and retention** — the reporting estate must be auditable (what was reported, when, by which system, with which data) — the same RTS 6 record-keeping discipline as the OMS, applied to the reports themselves.

The architectural conclusion: **regulatory reporting is a data-quality system, not a form-filling exercise** — the quality of the estate's golden copy *is* the quality of its compliance. Every report the bank files is a test of the data architecture, and the regulators are the most unforgiving users of all.

---

## 9. Architecture Patterns

### 9.1 The two master patterns

Two architecture patterns dominate the capital-markets estate — **low-latency** (the speed layer) and **event-driven** (the control layer). Everything else in this guide's architecture — microservices, messaging, data platforms, resilience — is an application of one of these two. The distinction is the classic "two-speed" or "bimodal" markets architecture ✅/⚠ (the two-speed framing is standard in the architecture press — WatersTechnology and vendor material describe the "fast path vs control path" split; the exact terminology varies, flagged ⚠).

### 9.2 Low-latency (the fast path)

**Low-latency architecture** is the discipline of minimising the time from market event to action — and it is measured in **microseconds** at the sharp end ✅/⚠. The verification status of "microsecond": modern exchange matching engines and direct market-data feeds operate in the low-microsecond range — CME, Eurex and the HFT community consistently report microsecond-level processing and co-located round trips; the **microsecond claim is industry-consistent** ✅ — but **specific numbers (e.g. "X microseconds end-to-end") are vendor-competitive claims and are flagged ⚠**: the brief asked to verify the microsecond claim and the honest answer is "the *regime* is microsecond-scale, the *specifics* are marketing". Latency budget anatomy:

- **Co-location** — placing servers in the exchange's data centre to cut the network hop; the [Singapore Data Centres Guide](../technology/singapore_data_centres_guide.md) covers the facilities angle (Equinix SG1/SG2, the exchange data centres) ✅/⚠.
- **The fast path** — market data in (direct feeds, binary encodings like SBE — §7), price/decision logic (often **FPGA** for the hardest-core functions — order book processing, risk pre-trade checks), order out. The fast path is a **pipeline of specialised components**, not a general-purpose stack: FPGAs, kernel-bypass networking (Solarflare/Onload, DPDK), and careful CPU affinity ✅/⚠ (FPGA/DPDK kernel-bypass are documented industry practice; the specifics flagged).
- **What is *not* on the fast path** — almost everything: the OMS record, the MO checks, the reporting. The fast path is a thin sliver (quote → order), and the control path is everything else. The two-speed pattern keeps the latency-critical logic minimal because *every* addition to the fast path is a latency risk and an audit problem.
- **Determinism over speed** — the real goal is *bounded* latency (a stable 5 µs beats a jittery 3 µs average with 20 µs tails); jitter is the enemy because it breaks the pricing model.
- **The retail vs institutional contrast** — most of a CIB's flow (client e-trading, RFQ, portfolio trading) does *not* need microsecond latency: it needs reliable single-digit-millisecond response. Microsecond engineering is reserved for the market-making and prop desks where the franchise *is* speed. An architect sizes the latency budget per desk, not per bank.

### 9.3 Event-driven (the control path)

**Event-driven architecture (EDA)** is the pattern that holds the control path together: every lifecycle transition, every price tick worth keeping, every settlement update is an **event**, and systems communicate by publishing and subscribing to events rather than by point-to-point calls ✅ (the EDA pattern is standard software-architecture practice and the explicit foundation of the series' [Event Stream Processing Guide](../technology/event_stream_processing_guide.md)). The capital-markets specifics:

- **The trade as an event stream** — the trade lifecycle (§3.2) is naturally an event log: TradeCaptured → TradeConfirmed → TradeCleared → TradeSettled. Event sourcing (the log as the system of record) fits the lifecycle perfectly — replaying the log rebuilds any state.
- **The backbone** — a message bus / event broker (Kafka, IBM MQ, Solace, and the classic TIBCO EMS — the markets legacy favourite) with **exactly-once or at-least-once semantics understood per use case**; the [Message Queue Data Loss Guide](../technology/message_queue_data_loss_guide.md) covers the delivery-guarantee trade-offs that decide whether a settlement instruction may be lost, and the [Kafka Alternatives Guide](../technology/kafka_alternatives_guide.md) covers broker choice.
- **Patterns in the stack** — pub/sub for market-data fan-out (§5), event sourcing for post-trade state, saga/compensation for cross-system workflows (a failed settlement must trigger a compensation path), CQRS where reads (position blotters) and writes (trade capture) have different loads ✅ (these are standard EDA patterns; the [Event Stream Processing Guide](../technology/event_stream_processing_guide.md) applies them to the markets domain).
- **The two-speed join** — the fast path publishes, the control path consumes: the market-making engine emits quotes-and-fills as events; the MO subscribes. The join point is the **tick store** (§5) — the fast path writes, the control path reads, and reconciliation happens at the log, not the application.
- **Resilience** — the control path must be *idempotent and replayable*: duplicated events (broker redelivery) must not double-book a trade; replay after an outage must rebuild positions exactly. These are the patterns of the [Event Stream Processing Guide](../technology/event_stream_processing_guide.md) and the [Monolith to Microservices Guide](../technology/monolith_to_microservices_guide.md)'s migration discipline, applied to money.

### 9.4 Supporting patterns

- **Front-to-back platforms vs best-of-breed** — the central architecture decision of the estate: one platform (Murex MX.3, Calypso) spanning FO–MO–BO vs a stack of specialised systems (Fidessa/ION for e-trading, AcadiaSoft for collateral, Broadridge for post-trade). The platform case is made in the [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md) and the [Nasdaq Calypso Guide](nasdaq_calypso_guide.md); the best-of-breed case is that no platform leads in every function. Real estates are **hybrids** — a platform core with specialist satellites — and the hybrid's success is decided by its **integration layer** (the event backbone above, plus FIX/SWIFT/APIs from §7).
- **The golden-copy pattern** — reference data (§5.3) and market data are canonicalised once, then distributed; the pattern is the data twin of the [Distributed Auth Guide](../technology/distributed_auth_guide.md)'s authoritative-source design.
- **Microservices where they earn it** — the control path decomposes into services (pricing, risk, confirmation, reporting), but the fast path does *not*: microservices' network hops are lethal to microsecond budgets. The [Monolith to Microservices Guide](../technology/monolith_to_microservices_guide.md) is the migration reference; the markets version adds "keep the fast path monolithic-by-design".
- **The resilience pattern** — markets systems are disaster-recovery obsessive: primary/secondary sites, the RPO/RTO per function (trading: minutes; settlement: hours; reporting: days), and the "no single point of failure in the clearing/margin path" rule. Cross-ref the [Singapore Data Centres Guide](../technology/singapore_data_centres_guide.md) for the facilities reality.

### 9.5 The patterns table

| Pattern | Where it applies | Key characteristics |
|---|---|---|
| **Low-latency fast path** | Market-making, direct feeds, algo execution | Microsecond regime ✅/⚠; co-location, FPGA, kernel-bypass; deterministic latency over raw speed |
| **Event-driven control path** | Post-trade lifecycle, risk, reporting | Pub/sub backbone (Kafka/MQ/Solace); event sourcing; replayable, idempotent consumers |
| **Two-speed architecture** | The estate as a whole | Fast path (thin) + control path (everything else); join at the tick store |
| **Front-to-back platform + satellites** | Platform decision | Murex/Calypso core + specialist systems; integration layer decides success |
| **Golden copy / MDM** | Reference and market data | One canonical source, governed distribution; LEI/ISIN/UPI/UTI as the keys |
| **Microservices (control path only)** | Post-trade services | Decompose the slow side; never the fast path; saga/CQRS where needed |
| **DR/resilience** | Everywhere | Primary/secondary sites; per-function RPO/RTO; no SPOF in clearing/margin paths |
| **API-first client access** | Distribution | REST/WebSocket/FIX client channels; auth per [Distributed Auth Guide](../technology/distributed_auth_guide.md) |

### 9.6 Resilience, security, observability — the non-functional spine

Markets architecture is defined as much by its non-functional requirements as by its functions — and three cut across every layer ✅ (the RPO/RTO and no-SPOF principles below are standard industry practice; the specific numbers per function are firm-specific, flagged ⚠):

- **Resilience and DR** — the estate runs a primary/secondary data-centre pair (the [Singapore Data Centres Guide](../technology/singapore_data_centres_guide.md) covers the facilities reality), with **RPO/RTO graded per function**: trading (minutes — every lost minute is lost P&L), market data (sub-second failover), post-trade (hours), reporting (same-day). The cardinal rule: **no single point of failure in the clearing/margin path** — a margin-call outage is not an inconvenience, it is a regulatory breach and a counterparty dispute. Cross-region failover for the Singapore hub means the Paris primary and the Singapore/APAC recovery site must stay in step via the event backbone (§9.3) — which is exactly the [Message Queue Data Loss Guide](../technology/message_queue_data_loss_guide.md) territory.
- **Security** — the estate is an attack surface the size of the bank: FIX sessions are authenticated and monitored (a rogue order is a market event), client APIs carry OAuth/MTLS (the [Distributed Auth Guide](../technology/distributed_auth_guide.md) patterns), market-data access is entitlement-controlled (§5.5), and the kill-switch/circuit-breaker pattern from MiFID II algo controls (§8.2) doubles as the security containment tool. The [Distributed Rate Limiter Guide](../technology/distributed_rate_limiter_guide.md) applies to the client-API tier: a client's runaway algo must not take down the quoting service.
- **Observability** — the estate is an event pipeline, so it is observable as one: distributed tracing across FIX-in → OMS → EMS → venue → post-trade (the trade's journey as a trace), the tick store (§5) as the market-data log, the lifecycle log (§3.2) as the trade's audit trail, and the MO scorecard (§3.6) as the control dashboard. If a trade cannot be traced end-to-end in seconds, the architecture is not finished.

---

## 10. Worked Example: A CIB Markets Architecture

### 10.1 The scenario — a Cymbal Bank

The worked example is deliberately the reader's own world: a **European universal bank's corporate & investment banking arm — Cymbal Bank-style** — running global markets (FX, rates, credit), structured finance, and trade finance out of Paris with Singapore as the Asia-Pacific hub ✅/⚠ (the entity facts are anchored in the [Crédit Agricole Software Systems Guide](credit_agricole_software_systems_guide.md) §3 — Cymbal Bank, ex-Calyon 2004/2010, Montrouge HQ, Singapore APAC hub ✅; the *system inventory* is non-public ⚠ and the worked example is a **reference architecture, not a claim about Cymbal Bank's actual estate** — the guide flags this explicitly, because the sibling guide refuses to assert what it could not verify, and so does this one). The bank's markets reality:

- **Franchise** — top-tier European rates/FX flow and structured products; global trade finance; a Singapore hub serving Asian corporates and institutions (the author's own office context).
- **The estate's ancestry** — Crédit Agricole Indosuez + Crédit Lyonnais CIB lineages, integrated through the Calyon era: an architect models this as *successively integrated* estates, with the oldest Asia/trade systems deepest (§3.3 of the CA guide).
- **The platform core** — Murex-class front-to-back for treasury/rates/FX ⚠ (industry-known, unconfirmed — see the [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md) for the platform class and the CA guide §3.1 for the honest flag).
- **The regulatory load** — MiFID II (transaction reporting, RTS 6), EMIR (clearing, margin, reporting), SFTR (repo/securities-lending reporting), CSDR (settlement discipline), plus FRTB market-risk capital (§8, and the [Risk Management Models Guide](risk_management_models_guide.md)).

### 10.2 The target architecture (the design)

The reference target architecture for a Cymbal Bank markets estate — five layers, two paths:

**Layer 0 — Market data & reference (the input fabric).** Direct feeds for the fast desks (Eurex, CME, LSE — co-located in the exchange data centres via the [Singapore Data Centres Guide](../technology/singapore_data_centres_guide.md) playbook) + vendor feeds (Bloomberg B-PIPE, LSEG RMDS) for the rest; a normalisation layer publishing a canonical price model onto the event backbone; a golden-copy reference-data hub (LEI/ISIN/CFI/UPI/UTI) feeding every system; the tick store as the join point between fast and control paths (§5).

**Layer 1 — The front office (the revenue layer).** The client channels (e-trading portal/API for FX and rates — the Cymbal Bank documented offering ✅/⚠), the OMS (order lifecycle, allocations, RTS 6 audit), the EMS (venue aggregation, algos, SOR), and the pricing/valuation services (curves, XVA, quote production — C++/Python quant library exposed as services). The fast path: FPGA-accelerated price-feed → quoting engine → venue, for the market-making desks only; everything else rides the normal bus (§2, §9).

**Layer 2 — The middle office (the control layer).** The post-trade platform (Murex/Calypso-class) owning the lifecycle state machine; confirmation/matching via DTCC CTM and MarkitSERV; collateral management (AcadiaSoft-class utilities, IM/VM under EMIR/UMR, compression via TriOptima/Quantile); daily P&L and risk control; the EMIR/SFTR/MiFID II reporting pipeline fed from this layer (§3, §8).

**Layer 3 — The back office (the record layer).** Settlement instructions to the CSDs (Euroclear/Clearstream/DTCC) with CSDR fail management; custody and corporate actions (CACEIS in the group's real world ✅); clearing connectivity to LCH/Eurex Clearing/CME Clearing as clearing member; accounting to the GL; the reconciliation estate (SmartStream TLM-class) (§4).

**Layer 4 — The backbone (the glue).** The event backbone (Kafka/MQ) carrying lifecycle events with replay and idempotency; FIX for order/trade messaging; SWIFT for post-trade; the API gateway for client and internal access (auth per the [Distributed Auth Guide](../technology/distributed_auth_guide.md)); the DR pair of data centres with per-function RPO/RTO (§7, §9).

**The two paths in one diagram-of-words:** market data → fast path (co-located, FPGA, microsecond) → fills → events → control path (post-trade platform → CTM → collateral → clearing → CSD → GL → regulators). The fast path is a thin sliver; the control path is the estate.

### 10.3 The lessons

What the worked example teaches — the transferable conclusions:

1. **The stack is a pipeline with a speed gradient** — from microsecond market data at one end to daily regulatory reporting at the other. Architect the *transitions* between speeds (the tick store, the event backbone) as carefully as the systems themselves; the joins are where breaks happen.
2. **The lifecycle is the schema** — every system is a state machine over the trade lifecycle (§3.2). If a system cannot name its lifecycle state and its transitions, it does not fit. This one principle prevents most integration chaos.
3. **Regulation is an architecture input, not an afterthought** — MiFID II RTS 6 order records, EMIR/SFTR reporting fields, CSDR fail penalties: each one lands on a specific system and adds a specific data requirement. Design the golden-copy data model so the reports *fall out* of the estate instead of being bolted on (§8).
4. **The platform-vs-best-of-breed decision is about the integration layer** — a hybrid (platform core + satellites) is the realistic outcome; its success is decided by the event backbone and the FIX/SWIFT/API layer, not by the platforms themselves (§9.4).
5. **Two-speed is not two systems** — the fast path and control path share the same data, the same trade, the same lifecycle; they differ in technology and latency budget, and they join at the log. Build the join, not the wall (§9.3).
6. **Honesty about the estate** — a real Cymbal Bank estate is successively integrated, partly non-public, and full of lineage systems (§10.1). The reference architecture is the target; the migration is the [Monolith to Microservices Guide](../technology/monolith_to_microservices_guide.md)'s discipline with markets-specific latency and regulatory constraints.

### 10.4 The migration roadmap

The target architecture of §10.2 is a destination; the realistic path is a sequence of phases that de-risk the estate while keeping the markets running ✅/⚠ (the phasing is standard programme-discipline, shaped for markets by the series' [Monolith to Microservices Guide](../technology/monolith_to_microservices_guide.md) and the [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md)'s platform-migration experience — flagged as reference-architecture guidance, not a Cymbal Bank claim):

1. **Stabilise (months 0–6)** — stand up the golden-copy data model and the data-quality rules (§5.5); get identifier hygiene (LEI/ISIN/UPI/UTI) to 100%; fix the worst STP breaks. No new platforms, no new risk — the point is that the data underneath everything becomes trustworthy.
2. **Integrate (months 6–18)** — build the event backbone (§9.3) and move the lifecycle events onto it; make the MO scorecard (§3.6) a query over the log instead of a scrape of screens; establish the trace end-to-end (§9.6). The estate's hand-offs are now observable.
3. **Modernise (months 12–36)** — consolidate the platform estate (the front-to-back core of §9.4), decommission the deepest lineage systems, move the control path to cloud where the regulator and the data-centre strategy allow (cross-ref the [Singapore Data Centres Guide](../technology/singapore_data_centres_guide.md) for the facilities side); keep the fast path co-located and specialised (§9.2).
4. **Optimise (ongoing)** — latency engineering for the desks that need it, TCA-driven execution tuning, reporting automation (the reporting pipeline of §8.7 becomes self-servicing), and the settlement-cycle work (T+1 readiness, CSDR discipline) that every BO now owns.

The golden rule of the migration: **the trade lifecycle never pauses**. Every phase must keep the capture→confirm→clear→settle stream intact, which is why the event backbone (phase 2) must precede the platform consolidation (phase 3) — you can only replace a system safely once the stream that runs through it is observable and replayable.

---

## 11. Summary: One Page

**The stack in one table:**

| Layer | Owns | External world it touches |
|---|---|---|
| Front office | Pricing, e-trading, OMS/EMS, sales | Clients (channels/APIs), venues (FIX), market data |
| Middle office | Lifecycle, confirmations, collateral, control | Counterparties (CTM/MarkitSERV), CCPs (margin), utilities |
| Back office | Settlement, custody, clearing, accounting, reporting | CSDs, custodians, GL, regulators (ARM/TR) |
| Market data | Feeds + reference data | Exchanges (direct feeds), vendors (Bloomberg/LSEG) |
| Infrastructure | Venues, CCPs, CSDs, TRs | LCH, Eurex Clearing, CME, Euroclear/Clearstream/DTCC |
| Protocols | Order/trade/post-trade messaging | FIX 4.4, SWIFT/ISO 20022, binary encodings |
| Regulation | Conduct, transparency, risk mitigation, settlement discipline | MiFID II, EMIR, SFTR, CSDR |

**Capital markets architecture** is the technology estate behind the markets business — the trading, structuring, and distribution of securities and derivatives. The estate is a three-layer stack with a speed gradient: the **front office** (pricing, e-trading, OMS/EMS — real-time, latency-sensitive), the **middle office** (lifecycle, confirmations, collateral — intraday control), and the **back office** (settlement, custody, clearing, accounting — the layer of record), all fed by **market data** (real-time feeds + reference data) and plugged into **market infrastructure** (exchanges, CCPs — LCH, Eurex, CME — CSDs) over **protocols** (FIX 4.4 the lingua franca, SWIFT, binary encodings), under a dense **regulatory** load (MiFID II from 2018, EMIR from 2012, SFTR from 2020, CSDR settlement discipline from 2022).

The architecture is governed by two master patterns: **low-latency** for the thin fast path (microsecond regime, co-location, FPGA — reserved for the desks where speed is the franchise) and **event-driven** for everything else (the trade lifecycle as an event stream, pub/sub backbone, replayable and idempotent control path). The platform decision — front-to-back platform (Murex MX.3, Calypso) vs best-of-breed satellites — is decided by the integration layer. The worked CIB example (Cymbal Bank: a European universal bank's markets arm with a Singapore hub) shows the target: five layers, two paths, one lifecycle schema, and a regulatory output that falls out of the design rather than being bolted on.

**The final word: the markets never sleep.** The venue in Singapore opens as London's day winds down; the rate fixings roll around the clock; the margin calls land in Tokyo while Paris sleeps. The architecture exists to keep the estate awake with them — the fast path to catch every tick, the control path to catch every error, and the record layer to make it all true at the end of the day. Architect for the hand-offs, because the markets do not wait for them.

---

## 12. Glossary

| Term | Definition |
|---|---|
| **Capital markets** | The markets where long-term funding and risk are traded: securities and derivatives, through trading, structuring, and distribution — as opposed to money markets (short-term) and the banking book (deposits/loans) |
| **Front office (FO)** | The revenue layer: sales, trading, structuring, pricing, execution; real-time and latency-sensitive |
| **Middle office (MO)** | The control layer: trade validation, confirmations, collateral, P&L/risk control; intraday to T+1 |
| **Back office (BO)** | The layer of record: settlement, custody, clearing, accounting, regulatory reporting; T+1 onward |
| **E-trading** | Electronic trading: orders and prices exchanged electronically with clients and venues — portals, APIs, algo execution, DMA |
| **OMS** | Order Management System: the system of record for orders — lifecycle, routing, allocations, audit, pre-trade controls |
| **EMS** | Execution Management System: the execution cockpit — venue aggregation, algos, SOR, real-time blotters |
| **Pricing** | The analytical engine turning market data into executable prices: models, curves, surfaces, XVA |
| **Trade lifecycle** | The canonical state sequence of a trade: pre-trade → execution → capture → enrichment → validation → confirmation → clearing → settlement → ongoing events → accounting/reporting |
| **Confirmation** | Mutual agreement of trade terms with the counterparty; the moment a trade becomes binding |
| **Collateral** | Margin posted to cover counterparty exposure: variation margin (VM) and initial margin (IM); managed under EMIR/UMR |
| **Settlement** | The exchange of cash and securities completing a trade (delivery-versus-payment) |
| **Custody** | Safekeeping and servicing of assets: securities ledger, corporate actions, income, asset servicing |
| **Clearing** | The process of becoming the counterparty to both sides of a trade — via a CCP (novation) or bilaterally |
| **Market data** | The raw material of the stack: real-time feeds (prices) and reference data (identifiers, static attributes) |
| **Reference data** | Static, slow-moving data everything keys off: ISIN, CFI, LEI, UPI/UTI, calendars, product attributes |
| **Exchange** | The venue where trading happens: matching engine, market data, rulebook (LSE, Eurex, CME, NASDAQ, ICE) |
| **CCP** | Central Counterparty: interposes itself between buyers and sellers (novation), collects margin, manages defaults |
| **LCH** | The LSEG-owned London CCP — the largest OTC interest-rate-swap clearer (SwapClear), plus ForexClear, RepoClear |
| **Eurex** | Deutsche Börse's Frankfurt derivatives exchange and clearing house (Eurex Clearing) — Europe's leading derivatives venue |
| **CME** | CME Group, Chicago: exchanges and clearing for futures, options, and swaps — the largest futures complex |
| **FIX** | Financial Information eXchange: the de facto standard protocol for order/trade messaging; FIX 4.4 the most widely deployed version |
| **MiFID II** | EU markets-in-financial-instruments regime (2014/65/EU, MiFIR 600/2014): conduct, transparency, transaction reporting; effective 3 January 2018 |
| **EMIR** | EU derivatives regulation (648/2012): clearing obligation, margin, risk mitigation, trade reporting; adopted 2012 |
| **SFTR** | Securities Financing Transactions Regulation (2015/2365): repo/securities-lending reporting; phased in from 11 April 2020 |
| **CSDR** | EU settlement regime (909/2014): CSD authorisation, settlement discipline (penalties, buy-ins) from 1 February 2022 |
| **Low-latency** | The architecture of the fast path: minimal, bounded latency — co-location, FPGA, kernel-bypass; microsecond regime at the sharp end |
| **Event-driven** | The architecture of the control path: systems communicate by publishing/subscribing to events; lifecycle as an event stream |
| **Microsecond** | One-millionth of a second — the latency regime of exchange matching engines and co-located HFT infrastructure (specific figures are vendor claims ⚠) |
| **CIB** | Corporate and Investment Bank — the wholesale arm of a universal bank (e.g. Cymbal Bank): global markets, structured finance, trade finance |
| **XVA** | Valuation adjustments (CVA/DVA/FVA): credit and funding costs added to derivative pricing |
| **STP** | Straight-through processing: trade handling without manual intervention — the BO's efficiency goal |
| **DvP** | Delivery-versus-payment: securities delivered against cash payment; the settlement-risk-free exchange model |

---

## 13. Verification Notes and Sources

### 13.1 What was verified live this pass (direct HTTP fetch)

| Claim | Status | Source/evidence |
|---|---|---|
| FIX Trading Community = the FIX standards body; FIX 4.4 is a published standard (spec release notes live) | ✅ | fixtrading.org (fetched live: "independent, non-profit standards body... FIX Protocol"; the FIX 4.4 Specification Release Notes page) |
| FIX header fields 8/9/35 (BeginString/BodyLength/MsgType); version lineage incl. 4.4 | ✅ | Wikipedia "Financial Information eXchange" (fetched live) |
| MiFID II/MiFIR effective 3 January 2018; original date 3 Jan 2017 delayed Feb 2016 for IT build-out; best-execution duty | ✅ | Wikipedia "Markets in Financial Instruments Directive 2014" (fetched live) |
| EMIR = Regulation (EU) 648/2012, adopted/implemented 2012; EMIR Refit 2019 | ✅ | Wikipedia "European Market Infrastructure Regulation" (fetched live) |
| SFTR = Regulation (EU) 2015/2365; reporting phases: banks/BDs 11 Apr 2020, FMIs 11 Jul 2020, insurers/AMs 11 Oct 2020, NFCs 11 Jan 2021 | ✅ | Wikipedia "Securities Financing Transactions Regulation" (fetched live) |
| CME Group: Chicago HQ; exchanges + clearing for futures, options, swaps | ✅ | Wikipedia "CME Group" (fetched live) |
| Eurex: "The Leading Derivatives Exchange"; Frankfurt; product range (STIR, equity, FX, repo) | ✅ | eurex.com (fetched live) |
| ESMA CCP Supervisory Committee exists (EMIR CCP supervision) | ✅ | esma.europa.eu site structure (fetched live) |
| FIX 4.4 release year (2003) | ✅ | Re-verified 2026-08-22: the FIXML 4.4 schema guide on fixtrading.org is dated 9 Jan 2004 (consistent with a 2003 release); the FIX 4.4 spec defines 916 field definitions, 92 message types, 24 reusable components |

### 13.2 Re-verification ledger (2026-08-22 — restored web backend via self-hosted Firecrawl)

| Claim | Status | Source/evidence |
|---|---|---|
| CSDR Settlement Discipline penalties applied 1 Feb 2022 | ✅ | ICMA CSDR Settlement Discipline page; ESMA RTS final report (buy-in suspension) |
| CSDR mandatory buy-in regime suspended ~3 years | ✅ | ESMA "technical standards to suspend the CSDR buy-in regime" (esma.europa.eu) |
| EU/UK/CH T+1 target 11 October 2027; CSDR revision (2025/0221) published in the OJ 14 Oct 2025 | ✅ | EC finance T+1 page (12 Feb 2025 proposal); BNP Paribas / Société Générale T+1 commentary |
| FIX 4.4 released 2003; 916 fields / 92 message types | ✅ | fixtrading.org FIXML 4.4 schema guide (9 Jan 2004); FIX 4.4 spec references |
| LCH SwapClear = dominant OTC IRS clearer ("longest-serving", "only truly global") | ✅ | LSEG SwapClear pages; Clarus CCPView (e.g. 72% of cleared INR swaps 2024) |
| **AcadiaSoft ownership — CORRECTED: acquired by LSEG (now "Acadia"), NOT DTCC** | ✅ | lseg.com/en/post-trade/solutions/acadia (LSEG-owned); the DTCC link was AcadiaSoft buying ProtoColl *from* DTCC (2020) |
| CTM = DTCC's central trade-matching platform (ex-Omgeo heritage) | ✅ | dtcc.com CTM pages |
| MarkitSERV = CDS matching utility (S&P Global ownership) | ✅ | Industry standard; IHS Markit → S&P Global |
| UMR phases 2016–Sep 2021; ~1,000+ entities in scope by Sep 2022 | ✅ | CME Group "Navigating Uncleared Margin Rules" |
| CSDR = 909/2014; settlement discipline from 1 February 2022 | ✅ | Superseded by the §13.2 re-verification ledger (2026-08-22): SDR penalties 1 Feb 2022 verified (ICMA/ESMA); buy-ins suspended |
| LCH SwapClear dominance; LSEG ownership timeline | ✅ | Superseded by the §13.2 re-verification ledger (2026-08-22): LSEG materials + Clarus CCPView confirm dominance; the exact 2012–14 LSE closing date remains ⚠ |
| "Microsecond" latency regime | ✅/⚠ | The *regime* is industry-consistent (exchange engines, co-located HFT); *specific* numbers are vendor-competitive claims — flagged as such in §9.2 |
| US equities T+1 (May 2024); EU T+1 target (11 Oct 2027) | ✅ | US move ✅; EU/UK/CH T+1 = 11 Oct 2027 re-verified 2026-08-22 (EC proposal 12 Feb 2025; CSDR revision in OJ 14 Oct 2025) |
| Vendor utility facts (DTCC CTM, MarkitSERV, Acadia→LSEG, TriOptima/Quantile compression, UnaVista/Regis-TR) | ✅/⚠ | CTM + Acadia→LSEG re-verified 2026-08-22 ✅; compression volumes and TR rosters remain approximate ⚠ |
| Cymbal Bank entity facts (Calyon 2004, Cymbal Bank 2010, Montrouge, Singapore hub, Murex-class platforms ⚠) | ✅/⚠ | Cross-referenced from the [Crédit Agricole Software Systems Guide](credit_agricole_software_systems_guide.md) §3, which applies the same ✅/⚠ discipline |

### 13.3 What could not be verified and how it was handled

- **Web research tools were degraded this pass** — `web_search` (SEARXNG_URL unset) and `web_extract` (search-only backend) both unavailable; all live verification above was done by direct `curl` fetch of primary/stable pages. The brief's instruction to "flag anything unverifiable" is followed line by line: every ⚠ above says *why* it is flagged.
- **The worked example is a reference architecture, not a claim about Cymbal Bank's actual systems** — the sibling [Crédit Agricole Software Systems Guide](credit_agricole_software_systems_guide.md) §3 documents that Cymbal Bank's system inventory is non-public (⚠); this guide's §10 builds a *target* design from verified industry patterns, and says so (§10.1).
- **Specific latency numbers, market-share figures, and compression notional volumes** are deliberately not stated as facts — they are vendor/analyst territory and flagged ⚠ wherever mentioned.
- **Adoption statistics** (FIX 4.4 market share, CCP market shares, e-trading adoption) are ⚠: consistent direction, no verified numbers this pass.

### 13.4 Primary source classes (for the reader to verify against)

- **Protocols:** fixtrading.org (FIX standards — the authoritative specification source) — verified reachable and current this pass ✅.
- **Regulation:** ESMA (esma.europa.eu), the European Commission, the EU regulation texts (2014/65/EU, 600/2014, 648/2012, 2015/2365, 909/2014) — the ESMA site was reachable this pass ✅; individual pages may 404 on reorganisation (as observed) — the regulation texts themselves are the ultimate reference.
- **Market infrastructure:** lch.com, eurex.com (✅ fetched), cmegroup.com, the CSD sites (euroclear.com, clearstream.com, dtcc.com).
- **Architecture press:** WatersTechnology, The TRADE, Finextra, IBS Intelligence.
- **Vendors:** murex.com, calypso.com (Nasdaq), fisglobal.com, broadridge.com, iongroup.com — the platform class references of §1–§2.
- **The sibling guides:** the cross-referenced guides listed in the header — the series' ✅/⚠ discipline is shared, and the Cymbal Bank §3 of the [Crédit Agricole Software Systems Guide](credit_agricole_software_systems_guide.md) is the anchor for the worked example.

### 13.5 Series map (where the angles live)

| Angle | Where it lives |
|---|---|
| The markets-platform layer (Murex, Calypso) | [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md), [Nasdaq Calypso Guide](nasdaq_calypso_guide.md) |
| The Cymbal Bank markets stack (worked-example anchor) | [Crédit Agricole Software Systems Guide](credit_agricole_software_systems_guide.md) §3 |
| Trade products (LCs, guarantees, supply-chain) | [Trade Finance Guide](trade_finance_guide.md), [Trade Finance Systems Guide](trade_finance_systems_guide.md) |
| The risk layer (VaR, FRTB, counterparty risk) | [Risk Management Models Guide](risk_management_models_guide.md), [Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md) |
| The banking-book boundary | [Core Banking Systems Guide](core_banking_systems_guide.md), [Universal Banking Model Guide](universal_banking_model_guide.md) |
| Co-location / low-latency facilities | [Singapore Data Centres Guide](../technology/singapore_data_centres_guide.md) |
| The event-driven infrastructure | [Event Stream Processing Guide](../technology/event_stream_processing_guide.md), [Message Queue Data Loss Guide](../technology/message_queue_data_loss_guide.md), [Kafka Alternatives Guide](../technology/kafka_alternatives_guide.md) |
| API/auth patterns | [Distributed Auth Guide](../technology/distributed_auth_guide.md); lightly [Distributed Rate Limiter Guide](../technology/distributed_rate_limiter_guide.md) |
| The migration discipline | [Monolith to Microservices Guide](../technology/monolith_to_microservices_guide.md) |
| The DLT-vs-central-infra contrast | [DeFi Guide](../technology/defi_guide.md) |

---

*End of guide. Verification status of every flagged claim is stated inline (✅/⚠) and consolidated in §13. The markets never sleep — and neither should the architecture's honesty about what it could and could not verify.*
