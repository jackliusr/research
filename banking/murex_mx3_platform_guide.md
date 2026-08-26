# The MX.3 Murex Platform — A Capital-Markets Platform Deep-Dive

*A dedicated deep-dive on **Murex** and its flagship platform **MX.3** — the front-to-back capital-markets platform for trading, treasury, risk, and post-trade operations. This is the capital-markets companion to the bank-systems series: it cross-references the [DBS Software Systems Guide](dbs_software_systems_guide.md) (the Murex-at-DBS anchor), the [Core Banking Systems Guide](core_banking_systems_guide.md) (the treasury-adjacent boundary), and the [Nasdaq Calypso Guide](nasdaq_calypso_guide.md) (the main competitor's deep-dive). Where public evidence runs out, this guide says so — the verification convention below is enforced line by line.*

**Verification convention used throughout: ✅ = verified in this research pass (primary sources — murex.com, Murex press releases, industry press — or the sibling guides' verified claims); ⚠ = flagged (inferred, approximate, single-source, or not re-verified in this pass); unmarked = structural/industry knowledge presented as such. The consolidated verification notes are in [§13](#13-verification-notes-and-sources).**

**One deliberate correction up front:** the brief for this guide asked to verify "the founders — Salim Eddé, Laurent Foata". The verified record is **Salim Eddé and Laurent Néel** — the French and English Wikipedia articles, and every other source found, name Laurent Néel, not Laurent Foata, as Murex's co-founder alongside Salim Eddé. "Laurent Foata" appears to be a French growth-equity investor (AgilaGrowth/Ardian) with no documented Murex connection. The guide uses the verified name and flags the discrepancy in [§2.2](#22-founders-and-leadership).

---

## Table of Contents

1. [Scope and Verification Discipline](#1-scope-and-verification-discipline)
2. [The Murex Company](#2-the-murex-company)
3. [MX.3 Overview](#3-mx3-overview)
4. [Architecture](#4-architecture)
5. [Modules](#5-modules)
6. [Market Data](#6-market-data)
7. [Integration](#7-integration)
8. [Clients](#8-clients)
9. [Competition](#9-competition)
10. [Worked Example: A Bank Treasury Implementation](#10-worked-example-a-bank-treasury-implementation)
11. [Summary: One Page](#11-summary-one-page)
12. [Glossary](#12-glossary)
13. [Verification Notes and Sources](#13-verification-notes-and-sources)

---

## 1. Scope and Verification Discipline

### 1.1 What this guide is

This guide is the capital-markets platform deep-dive of the bank-systems series. Where the [Core Banking Systems Guide](core_banking_systems_guide.md) owns the banking book — customer accounts, deposits, loans — this guide owns the **trading book**: the systems a bank uses to trade, price, risk-manage, settle, and report on capital-markets products. Murex's MX.3 is one of the two or three platforms that define that category (alongside Calypso — see [§9](#9-competition) and the [Nasdaq Calypso Guide](nasdaq_calypso_guide.md)).

The guide answers, in order:

- **Who is Murex?** The company: founding, founders, ownership, scale ([§2](#2-the-murex-company)).
- **What is MX.3?** The platform: what it covers, where it sits in the market ([§3](#3-mx3-overview)).
- **How is it built?** The architecture: client-server, tiers, MXML ([§4](#4-architecture)).
- **What does it do?** The modules: risk (VaR, simulation), trading, operations ([§5](#5-modules)).
- **What does it consume?** Market data ([§6](#6-market-data)).
- **What does it talk to?** Integration: interfaces, APIs, SWIFT/FIX ([§7](#7-integration)).
- **Who runs it?** Clients, anchored by DBS since 2014 ([§8](#8-clients)).
- **Who competes with it?** Calypso and the rest ([§9](#9-competition)).
- **What does a real deployment look like?** A worked treasury implementation ([§10](#10-worked-example-a-bank-treasury-implementation)).

### 1.2 Verification status at a glance

| Claim in the brief | Verified finding | Status |
|---|---|---|
| Founded 1986, Paris | ✅ Founded 15 September 1986 in Paris | ✅ Verified |
| Founders: Salim Eddé, Laurent Foata | ❌ Founders are **Salim Eddé and Laurent Néel**; "Laurent Foata" is not a documented Murex founder | ⚠ Brief corrected |
| MX.3 = flagship, front-to-back platform | ✅ MX.3 covers trading, treasury, risk, and post-trade operations end-to-end | ✅ Verified |
| Positioning: trading, risk, treasury | ✅ Murex's own positioning; also post-trade and regulatory | ✅ Verified |
| Client-server architecture | ✅ Tiered, service-oriented; desktop + web clients; Oracle/Sybase/SQL Server DBs | ✅ Verified |
| Front-office pricing, middle risk, back office | ✅ Matches MX.3's coverage (pricing/trade capture; market & credit risk; operations/finance) | ✅ Verified |
| MXML = the Murex language | ✅ MXML (Murex XML) is the platform's interface/scripting/data-exchange language — ecosystem-documented | ⚠ Ecosystem sources, not murex.com in this pass |
| Risk modules: VaR, simulation | ✅ Historical VaR, expected shortfall, stress testing, what-if simulation, P&L explanation | ✅ Verified |
| DBS client since 2014 | ✅ Murex MX.3 adopted by DBS for risk management in 2014 (also AWS move, 2017) | ✅ Verified (cross-ref [DBS guide](dbs_software_systems_guide.md)) |
| Calypso = competitor | ✅ Calypso (founded 2001) → Adenza (2021) → Nasdaq (2023) — main rival | ✅ Verified |
| Market share / adoption numbers | ⚠ Only scale proxies verified (60,000+ daily users, 70 countries); market-share figures are vendor/analyst estimates | ⚠ Flagged |
| Company scale (headcount, offices) | ✅ 3,400+ employees, ~20 offices (2025 industry profiles); 60,000+ daily users | ✅ Verified (single-source variance flagged) |

### 1.3 What is public vs what is not

Murex is **private and famously quiet** — it publishes no revenue figures, no client contracts, and no detailed product documentation. The verified record is assembled from:

- **murex.com** — corporate pages: MX.3 positioning, treasury management, connectivity, architecture, MX.3 APIs, client segments (✅ primary).
- **Murex press releases and client announcements** — DBS/AWS (2017), Aldermore (2025), BIBD, Murex–Accenture partnership (✅ primary).
- **Industry press and analyst sources** — Risk.net, WatersTechnology, IBS Intelligence, Accenture case studies (✅ where cited).
- **Wikipedia (EN/FR)** — company history, founders, founding date (✅ for company facts; ⚠ for anything product-level).
- **The sibling guides** — the [DBS guide](dbs_software_systems_guide.md) §5 and §11 anchor the DBS–Murex facts.

What is **not** public: the internal module catalogue (Murex does not publish a formal "module list" — the guide's module table is reconstructed from Murex's published solution families and flagged accordingly), per-client contract terms, and precise revenue/market-share figures. Anything of that kind below is explicitly ⚠.

### 1.4 Series map

| Angle | Where it lives |
|---|---|
| The bank that anchors Murex in this series (2014 adoption, AWS, GFM) | [DBS Software Systems Guide](dbs_software_systems_guide.md) §5 |
| The core-banking boundary (banking book vs trading book) | [Core Banking Systems Guide](core_banking_systems_guide.md) §2.4 |
| Where trading/treasury sit in the universal banking model | [Universal Banking Model Guide](universal_banking_model_guide.md) |
| The limits/risk angle MX.3 enforces (limits, VaR, credit lines) | [Banking Limits Domain Guide](banking_limits_domain_guide.md) |
| The trading-angle companion (structured products, trade lifecycle) | [Asset Backed Trading Guide](asset_backed_trading_guide.md) |
| The accounting mechanics MX.3's finance modules feed into | [Interest Engines Core Banking Guide](interest_engines_core_banking_guide.md), [Posting Engine Core Banking Guide](posting_engine_core_banking_guide.md), [Core Banking Processes Guide](core_banking_processes_guide.md) |
| The main competitor (Calypso lineage, Nasdaq ownership) | [Nasdaq Calypso Guide](nasdaq_calypso_guide.md) |
| Alternative platforms from the series (core-vendor treasury modules) | [Temenos T24 Guide](temenos_t24_guide.md), [Oracle Flexcube Data Model Guide](oracle_flexcube_data_model_guide.md) |
| The bank-systems pattern this guide follows | [OCBC Guide](ocbc_software_systems_guide.md), [UOB Guide](uob_software_systems_guide.md), [SMBC Guide](smbc_software_systems_guide.md) |
| Market infrastructure MX.3 plugs into (clearing, settlement, rails) | [Financial Infrastructure Guide](financial_infrastructure_guide.md) |
| Identity/access patterns in large platform deployments (light) | [Distributed Auth Guide](../technology/distributed_auth_guide.md) |

### 1.5 Why a capital-markets platform is not a core banking system

A reader coming from the [Core Banking Systems Guide](core_banking_systems_guide.md) or [Oracle Flexcube Data Model Guide](oracle_flexcube_data_model_guide.md) needs the distinction sharp, because it explains everything MX.3 does differently:

| Dimension | Core banking system | Capital-markets platform (MX.3) |
|---|---|---|
| **System of record for** | Customer accounts, deposits, loans (the banking book) | Trades, positions, market risk (the trading book) |
| **Primary users** | Operations, tellers, customer service | Traders, risk analysts, treasury dealers, back-office ops |
| **Data model** | Account-centric (customer → account → balance) | Trade-centric (trade → position → risk/P&L) |
| **Time sensitivity** | Near-real-time to daily | Real-time intraday pricing/risk + heavy EOD batch |
| **What it must compute** | Interest, fees, balances, limits | Pricing models, VaR/ES, sensitivities, XVA |
| **External feeds** | Payments rails, channels | Market data (prices/curves), SWIFT/FIX, clearing |
| **Replacement risk** | Multi-year, massively parallel | Multi-year, massively parallel — same discipline |

The boundary between them is drawn in the [Core Banking Systems Guide](core_banking_systems_guide.md) §2.4 and respected by MX.3's integration design: **the treasury funds the balance sheet; the core records the customer side of it** — and the two systems reconcile on cash flows, not on accounts.

---

## 2. The Murex Company

### 2.1 Founding: Paris, 1986

Murex was founded **in Paris, on 15 September 1986** ✅ (the French Wikipedia article records the incorporation date; the English article and Murex's own corporate history confirm the year and city). The founding context matters: 1986 was the dawn of the modern derivatives era — the first interest-rate swaps had existed for barely five years, the first swaptions were new, and trading floors still ran on mainframe-era calculators and paper tickets. A handful of Parisian engineers saw that the derivatives business would need *software* the way the equity business needed exchanges — and built a company to supply it.

The company grew **organically** (French Wikipedia: "l'entreprise a connu une croissance organique") — no venture capital, no acquisition spree, no IPO ✅. That founding DNA is still visible in how the company behaves today: privately held, product-obsessed, and — by modern fintech standards — extraordinarily patient.

### 2.2 Founders and leadership

**Verified founders: Salim Eddé and Laurent Néel** ✅ (English and French Wikipedia, corroborated by Murex corporate material and multiple interviews).

- **Salim Eddé** — co-founder; a Lebanese-born mathematician/engineer (son of Michel Eddé, a Lebanese lawyer and former minister). Eddé remains the public face of Murex's engineering culture; he is also known for philanthropy (the MIM Museum in Beirut, a major private mineral collection). ✅
- **Laurent Néel** — co-founder; the second name in every independent account of Murex's founding (Wikipedia EN/FR). ✅
- **Elias Eddé** — Salim's brother, co-founder, and **CEO** of Murex ✅ (industry profiles; Murex's leadership is consistently described as Eddé-family-led). The French Wikipedia notes Salim's brothers and brother-in-law joined early — Murex is one of the few fintechs of its scale still run by its founding families.

**⚠️ Brief correction, stated plainly:** the task brief named "Laurent Foata" as a co-founder. No source connects Laurent Foata to Murex — the name that appears in every source is **Laurent Néel**. (The search trail: "Laurent Foata" resolves to a French private-equity investor at AgilaGrowth, ex-Ardian — a different person entirely.) This guide uses **Laurent Néel** as the verified second founder.

**Leadership style note (⚠ industry observation):** Murex has no analyst-relations machine and no marketing-heavy CEO. Its go-to-market is solution-led and partner-led (Accenture describes itself as "Murex's sole global partner" in a case study — ⚠ that is Accenture's own claim, not an independently verified exclusivity arrangement). The company's public output is dominated by product releases, client announcements, and conference appearances by its technologists.

### 2.3 The company today: private, and quietly huge

The verified shape of Murex today ✅:

| Dimension | Verified figure | Source / flag |
|---|---|---|
| Ownership | Private / independent — no external investors, no IPO | ✅ industry profiles ("independent company"); French Wikipedia (organic growth) |
| Headquarters | Paris, France (main office) | ✅ Wikipedia, murex.com |
| Scale | **3,400+ employees, ~20 offices** | ✅ industry profiles (Oct 2025 vintage); ⚠ single-source variance on exact headcount |
| Usage | **60,000+ daily users** in 60+ countries (murex.com); customers across **70 countries** (Wikipedia) | ✅ primary; ⚠ "60 vs 65+ countries" minor variance between sources |
| Client segments | Banks (global/investment/regional), asset managers, pension funds, insurers, energy & commodity firms, clearing houses, central banks | ✅ murex.com client pages |
| Revenue | Not disclosed (private) | ⚠ no public figure — treat any number you see in analyst reports as an estimate |

The "scale-flagged" framing from the brief deserves a sentence of its own: Murex is the rare company that is **top-tier in its niche and still anonymous to the general public**. The 60,000+ daily users figure means MX.3 is in production at a meaningful share of the world's large trading banks — yet because the company is private, unlisted, and unmarketed, *every* aggregate figure (market share, revenue, licence count) is an estimate. This guide flags market-size claims rather than endorsing them (see [§13](#13-verification-notes-and-sources)).

### 2.4 The company table

| Aspect | Description |
|---|---|
| **Founded** | 15 September 1986, Paris, France ✅ |
| **Founders** | Salim Eddé and Laurent Néel ✅ (⚠ brief's "Laurent Foata" corrected — see [§2.2](#22-founders-and-leadership)) |
| **Early team** | Salim's brothers and brother-in-law joined at the start; the Eddé family remains at the helm ✅ |
| **CEO** | Elias Eddé (co-founder, Salim's brother) ✅ |
| **Ownership** | Private / independent; organic growth; no IPO, no VC rounds documented ✅ |
| **Headquarters** | Paris, with ~20 offices near major financial centres (New York, London, Singapore, Tokyo, Hong Kong, etc.) ✅ |
| **Scale** | 3,400+ employees; 60,000+ daily users; customers in ~70 countries ✅ (⚠ headcount single-source) |
| **Flagship product** | MX.3 — the front-to-back capital-markets platform ✅ |
| **Legacy/predecessor platforms** | Earlier Murex platforms (Murex 2000 / MXG lineage) ⚠ — documented mainly in job ads and legacy-project references; MX.3 is the modern platform |
| **Positioning** | Cross-asset trading, treasury, risk, and post-trade technology for financial institutions ✅ |
| **Partners** | Accenture (Murex's claimed sole global partner ⚠), Publicis Sapient (Aldermore treasury project), plus regional specialist implementers ✅ |
| **Cloud posture** | MX.3 on AWS (2017 DBS announcement) and Azure (Microsoft reference architecture); MX.3 SaaS via Murex hosting; MXGO packaged treasury solution ✅ |
| **Public profile** | Intentionally low: no revenue disclosure, no client-contract disclosure ✅ (structural fact) |

### 2.5 Company milestones: what is known, and what is inferred

Because Murex is private and terse, a precise company timeline is not publicly available. The verified anchor points, and the flagged gaps between them:

| Period | Milestone | Status |
|---|---|---|
| 1986 | Founded in Paris by Salim Eddé and Laurent Néel (15 September) | ✅ Wikipedia EN/FR |
| 1986–1990s | Early growth in French and European derivatives/trading desks; the Eddé brothers and early engineers build the first Murex platform | ⚠ general history; specifics not public |
| 1990s–2000s | Expansion across Europe, then Americas and Asia; the earlier platform generations (Murex 2000 / MXG lineage) become the industry workhorse for FX/rates derivatives at major banks | ⚠ ecosystem/job-ad record; platform names flagged |
| 2000s–2010s | MX.3 becomes the flagship platform; growth of the risk franchise (market risk, credit risk, XVA) | ⚠ MX.3 launch era not precisely dated in public sources |
| 2014 | **DBS adopts MX.3 for risk management** — the series anchor | ✅ [DBS guide](dbs_software_systems_guide.md) §1.3/§5 |
| 2017 | MX.3 on AWS announced, DBS as reference client | ✅ WatersTechnology |
| 2020s | Cloud/SaaS push (MX.3 SaaS, MXGO packaged treasury); GPU-accelerated analytics with NVIDIA; Aldermore treasury go-live (2025) | ✅ announcements |
| Today | ~3,400 employees, ~20 offices, 60,000+ daily users, 70+ countries | ✅ (⚠ headcount single-source) |

The honest reading: Murex's **survival** across four decades is verified (1986 → present), its **adoption pattern** is verified in the 2010s anchors (DBS), and everything between is ⚠ — which is exactly the shape of a private company with no analyst-relations function. The [§13](#13-verification-notes-and-sources) claims table keeps these straight.

## 3. MX.3 Overview

### 3.1 The platform: front-to-back by design

MX.3 is Murex's flagship platform ✅ — and the definition of the platform is **front-to-back**: it spans the entire trade lifecycle, from pre-trade analytics through execution, pricing, risk, confirmations, settlement, accounting, and regulatory reporting, on **one data model** ✅ (Murex's own positioning: "enterprise-wide, cross-asset financial technology… supports trading, treasury, risk and post-trade operations"; industry profiles add pricing, market risk, credit risk, collateral management, treasury, operations and regulatory reporting in "a single platform").

Three properties follow from that design and are worth internalising, because they are the reason banks buy MX.3 rather than five point solutions:

1. **One source of truth.** There is no "trading system" and separate "risk system" with reconciliation between them. There is one trade store, one position store, one P&L. The [DBS guide](dbs_software_systems_guide.md) calls out exactly this as the industry pattern for large markets estates — and it is the antidote to the classic bank problem of front-office numbers disagreeing with risk numbers (see [§10](#10-worked-example-a-bank-treasury-implementation)).
2. **Real-time consistency.** Murex's banking client page promises "real-time consistency across trading, risk and operations" ✅ — a trade booked in the morning is in the afternoon's VaR, the evening's settlement run, and the month-end ledger, without batch reconciliation between silos.
3. **Cross-asset coverage.** MX.3 handles FX (spot, forwards, swaps, options), interest-rate derivatives (swaps, caps/floors, swaptions, inflation), credit derivatives, commodities (including energy), equities, money market, repo/securities finance, and structured products ✅ (murex.com lists FXD, IRD, credit and commodity derivatives explicitly; treasury coverage includes Islamic finance conventions ⚠ per Murex's treasury page).

### 3.2 Positioning: trading, risk, treasury

Murex positions MX.3 across three business pillars, which map to the three audiences that buy it ✅:

- **Trading** — the front office: pricing, trade capture, e-trading connectivity, portfolio management for the trading book. This is where MX.3 competes with execution-focused platforms.
- **Risk** — the middle office and the CRO's office: market risk (VaR, expected shortfall, stress), credit risk (counterparty, pre-settlement, limits), XVA, collateral management, and regulatory capital. This is historically Murex's deepest moat — the [DBS guide](dbs_software_systems_guide.md) records DBS adopting MX.3 **for risk management** in 2014, not for trading.
- **Treasury** — the balance-sheet function: cash and liquidity management, funding, money-market and FX for the banking book, ALM-adjacent analytics, plus the packaged **MXGO** treasury offering (⚠ MXGO details are vendor-marketed; verified existence via the 2025 Aldermore announcement, see [§8](#8-clients)).

Beyond the three pillars, Murex sells **post-trade operations** (confirmations, settlement, collateral operations), **finance** (accounting, P&L, general-ledger feeds), and **regulatory reporting** as first-class solution families ✅ — which is why "front-to-back" is the accurate one-word summary of the platform's ambition.

### 3.3 Market segments served

MX.3 is deployed across the financial industry, not just banks ✅:

- **Global/investment banks** — the flagship segment (DBS, and the anonymous majority of the 60,000+ daily users).
- **Regional and mid-size banks** — treasury and risk modernisation (Aldermore, BIBD are documented examples, [§8](#8-clients)).
- **Asset managers, pension funds, insurers** — Murex explicitly names these segments on its client pages ✅ (buy-side risk/valuation use cases).
- **Energy and commodity firms** — MX.3 has an E/CTRM positioning (⚠ vendor-published "rankings" claims are marketing; the existence of the E/CTRM solution family is verified).
- **Clearing houses, central banks** — ⚠ segment claims from Murex's corporate pages; not independently verified.

### 3.4 The MX.3 table

| Aspect | Description |
|---|---|
| **What it is** | Murex's flagship capital-markets platform ✅ |
| **Coverage** | Front-to-back: trading, treasury, risk, post-trade operations, finance, regulatory reporting on one data model ✅ |
| **Asset classes** | FX, interest-rate derivatives, credit, commodities/energy, equities, money market, repo/securities finance, structured products; treasury products incl. Islamic finance conventions ✅ (⚠ equities/structured breadth per vendor pages) |
| **Positioning pillars** | Trading (front office), Risk (middle office/CRO), Treasury (balance sheet) ✅ |
| **Client segments** | Banks (all tiers), asset managers, pension funds, insurers, energy & commodity firms, clearing houses ✅ |
| **Deployment models** | On-premise, private cloud, public cloud (AWS/Azure), Murex-hosted SaaS, and MXGO (packaged treasury) ✅ |
| **Scale evidence** | 60,000+ daily users in 60+ countries; used by banks across ~70 countries ✅ (⚠ exact splits not disclosed) |
| **Business model** | Perpetual/term licences + implementation services via partners; subscription/SaaS emerging (MX.3 SaaS, MXGO) ⚠ |
| **Why banks buy it** | One data model (no front-office/risk reconciliation), cross-asset depth, regulatory coverage, risk analytics depth ✅ |

### 3.5 A note on platform generations ⚠

Murex has been selling capital-markets software since 1986; MX.3 is the modern platform, but the ecosystem still references earlier generations — job ads and legacy-project documentation mention "Murex 2000" / "MXG.2000" as the preceding platform family ⚠ (single-source, ecosystem-level). For the purposes of this guide, "Murex" = MX.3 unless stated otherwise; when you meet a veteran who says "I've been on Murex since MXG", they mean the earlier generation.

---

### 3.6 A trade's journey through MX.3

The fastest way to understand "front-to-back" is to follow one trade through the platform. Take a corporate customer asking for a 5-year EUR interest-rate swap (the bread-and-butter derivative of a bank's treasury-sales desk):

1. **Pre-trade** — the salesperson checks the indicative price (MX.3 pricing), the customer's credit limit (MX.3 credit-risk/limits, cross-ref [Banking Limits Domain Guide](banking_limits_domain_guide.md)), and the desk's risk appetite. One system, no Excel.
2. **Capture** — the trader books the swap: counterparty, notional, maturity, fixed/floating legs, payment dates. MX.3 validates conventions (EUR calendar, ACT/360), generates the cash-flow schedule, and prices it against the current EUR curve ✅.
3. **Intraday risk** — the trade joins the book: positions, sensitivities (DV01, PV01), limit checks, and P&L update immediately (real-time consistency ✅).
4. **Confirmation** — the back office matches the trade with the customer's confirmation (SWIFT MT300-class or electronic matching), exceptions handled in workflow ✅.
5. **Settlement & collateral** — cash flows are scheduled; if the trade is under a CSA, margin calls are computed by the collateral module ✅.
6. **EOD batch** — overnight: positions revalued at closing market data, risk run (VaR/ES/stress) computed, accounting entries posted, Datamart refreshed, regulatory data staged ⚠ batch orchestration.
7. **Reporting** — the trade appears in the desk P&L, the risk pack, the finance ledger feed, and the regulator's trade report ⚠ mapping.

Every step uses the *same trade record* — that is the entire value proposition, and it is why the [DBS guide](dbs_software_systems_guide.md) treats MX.3 as the markets estate's system of record rather than "the trading system" or "the risk system".

---

## 4. Architecture

### 4.1 Client-server, tiered, service-oriented

MX.3 is a **client-server system built on a tiered, service-oriented architecture** ✅ (Murex's own architecture page: "MX.3 relies on a tiered, service-oriented architecture. End users access MX.3 business solutions through a desktop app or web browser. The application tier is composed of presentation, business, orchestration and technical layers.").

Concretely, that means:

- **Clients:** a thick desktop client (the classic Murex GUI, still the workhorse for traders and risk analysts) and a web client for lighter/remote use ✅. This dual-client pattern is unusual in modern platforms and is a legacy strength: the desktop client supports the dense, keyboard-driven workflow traders want, while the web client opens the platform to the rest of the bank.
- **Application tier:** layered — **presentation** (screens, workflows), **business** (pricing, risk, operations logic), **orchestration** (process/state-machine management across modules), **technical** (common services: security, logging, scheduling, messaging) ✅.
- **Data tier:** relational databases — MX.3 runs on **Oracle, Sybase, or SQL Server** ✅ (Microsoft's published MX.3-on-Azure reference architecture documents SQL Server support from MX.3 version 3.1.48 and lists Oracle/Sybase alongside). A dedicated reporting layer (the **Datamart**) serves analytics and regulatory reporting ⚠ (ecosystem-documented; see [§4.4](#44-the-data-layer-datamart-and-databases)).
- **Batch and real-time:** the platform runs a heavy **end-of-day batch chain** (positions, risk, accounting, reporting) alongside real-time intraday processes ⚠ (operations-ecosystem documentation is consistent on this; the EOD batch is a well-known MX.3 operational fact).

### 4.2 Core components: front office, middle office, back office

MX.3 is a three-tier *business* architecture as much as a technical one — the platform is organised around the front/middle/back-office split of a markets bank ✅ (each verified against Murex's published solution families):

- **Front office — pricing and trading.** Trade capture, pricing models per asset class, quote management, e-trading/execution connectivity, position keeping, front-office P&L. "Pricing" here is the deep end: MX.3 prices plain-vanilla and exotic derivatives with model libraries per asset class ✅ (murex.com treasury page: "pricing, trade capture and trade life cycle management in a versatile catalog of traditional and structured treasury products"; NVIDIA's MX.3 collaboration documents pricing evaluation, sensitivity and P&L computation).
- **Middle office — risk.** Market risk (VaR, expected shortfall, stress testing, sensitivities/Greeks, P&L explanation), credit risk (counterparty exposure, limits, CVA/DVA), collateral management, and XVA analytics ✅ (Murex treasury page; IBS Intelligence verified-vendor profile; NVIDIA Grace Hopper testing for XVA and market-risk calibration). The risk engine is the reason DBS chose Murex in 2014 (cross-ref [DBS guide](dbs_software_systems_guide.md) §5).
- **Back office — operations and finance.** Confirmations (SWIFT/electronic matching), settlement, payments, collateral operations, accounting entries, and feeds to the general ledger and regulatory reporting ✅ (Murex operations & finance solution family; ecosystem documentation of SWIFT/FIX/GL/reconciliation interfaces). This is where front-to-back earns its name: the same trade that a trader captured is the same record that settles, posts, and reports.

### 4.3 MXML — the Murex language

**MXML ("Murex XML") is the platform's proprietary data-exchange and scripting language** ⚠ — ecosystem-documented (implementation-partner documentation, job ads, operations guides) rather than fully documented on murex.com in this research pass, but ubiquitous in the MX.3 ecosystem.

What MXML is used for, per the ecosystem record:

- **Interfaces and data exchange** — inbound/outbound messages to and from MX.3 (trade feeds, market data, reference data) are typically MXML-shaped; the "MXML interfaces" pattern appears in every MX.3 operations engagement (✅ per implementation-partner documentation, e.g. AAQUILIX's MX.3 operations services).
- **Batch and automation** — MXML defines batch jobs, transformations, and workflow glue in the EOD chain ⚠.
- **Customisation** — client-specific screens, validations, and calculations are delivered as MXML customisation layers on top of the core product ⚠ (this is a *consulting economy* in its own right — "MXML developer" is a job title).
- **Reporting** — the Datamart's data definitions and report feeds are configured in MXML ⚠.

**Why it matters architecturally:** MXML is the price of MX.3's integration flexibility. Because everything in the platform — trade, price, risk run, interface — is expressed in one structured language, a bank can move data in and out of MX.3 without writing to its database. The cost is a proprietary skill: MXML expertise is scarce, which is why Murex implementations lean on partners (Accenture, Publicis Sapient, regional specialists) and why "MXML" shows up constantly in capital-markets job ads (e.g., Singapore's DBS-adjacent Murex/MXML market — ⚠ job-market observation).

### 4.4 The data layer: Datamart and databases

- **Operational databases:** Oracle, Sybase, SQL Server ✅ (Microsoft Learn architecture: "Murex MX.3 workloads can run on databases like Oracle, Sybase, or SQL Server. With version V3.1.48, SQL Server 2019 Standard is supported").
- **Datamart** ⚠ — the reporting/mart layer that hosts the bank's analytical views (positions, P&L, risk results, regulatory data) separately from the operational store. Ecosystem documentation ("Datamart reporting" is a standard MX.3 support item) confirms it exists; the precise schema is product-internal.
- **Architecture consequence for cloud:** because the platform is database-portable and tiered, Murex has been able to certify MX.3 on AWS (2017, DBS announcement) and Azure (Microsoft reference architecture) ✅ — a significant architectural property in an industry where legacy trading systems are famously hard to move off-premise.

### 4.5 The architecture table

| Layer | Components | Notes |
|---|---|---|
| **Client** | Desktop client, web client | Dense front-office GUI + lightweight web access ✅ |
| **Presentation** | Screens, workflows, dashboards, trader/risk/ops workspaces | Part of the tiered application tier ✅ |
| **Business** | Pricing models, trade capture, risk engines (VaR/simulation), operations logic, accounting | The "front/middle/back" business split lives here ✅ |
| **Orchestration** | Workflow/state machines, trade lifecycle management, EOD batch orchestration | STP across modules ✅ (batch orchestration ⚠ ecosystem) |
| **Technical** | Security/auth, messaging, scheduling, logging, configuration | Common services; MXML for configuration/customisation ⚠ |
| **Data** | Oracle / Sybase / SQL Server; Datamart reporting layer | DB-portable design enables cloud ✅ (Datamart ⚠ ecosystem) |
| **Integration** | Packaged interfaces, MXML, next-gen APIs, SWIFT/FIX/market-data adapters | See [§7](#7-integration) ✅ |

### 4.6 Typical deployment topology

A production MX.3 estate, as described by Murex's architecture material and the ecosystem record, has a recognisable shape (⚠ the specifics below are ecosystem/operational knowledge, not a Murex-published blueprint):

```
Market data feeds ──► [Feed adapters] ──► MX.3 app tier ──► Oracle / Sybase / SQL Server
SWIFT / FIX in/out ─► [Interface layer]        │  (presentation / business /
Bank's APIs ◄──────► [API gateway]             │   orchestration / technical)
                                              ▼
                                   EOD batch chain (positions → risk → accounting → reporting)
                                              ▼
                                   Datamart ──► dashboards / regulatory / GL feeds
```

- **Environments:** DEV → UAT → PROD at minimum, with the configuration/customisation (MXML) promoted through them like code ⚠ ecosystem standard.
- **Servers:** application servers hosting the tiered services; database servers (Oracle/Sybase/SQL Server) with the operational store and the Datamart ✅ databases; batch schedulers driving the EOD chain ⚠.
- **Clients:** desktop client on trader/analyst workstations; web client for broader access ✅.
- **Cloud:** the same topology maps onto AWS (DBS precedent ✅) or Azure (Microsoft reference architecture ✅); database choice is the main portability consideration.
- **Operations reality:** MX.3 estates run a famous EOD batch — risk runs, valuations, accounting — that must finish before market open; operations teams live on batch monitoring, feed health, and interface queues ⚠ (consistent ecosystem record; this is why "Murex ops" is a specialist job family).

---

## 5. Modules

### 5.1 How to read this section (honesty note)

Murex does **not** publish a formal "module list" — its public material is organised as *solution families* (trading, treasury, risk, operations & finance, regulatory) ✅. The table below therefore names **solution families** (verified) and, inside them, the **capabilities** Murex documents (verified); it deliberately avoids inventing product-code module names (e.g., "MX.3 Market Risk Manager") that Murex does not publicly use. If you are in an implementation and someone quotes a formal module name, it is either from Murex's internal catalogue (legitimately) or from a consultant's whiteboard — treat it as ⚠ until confirmed.

### 5.2 The risk modules: VaR, simulation, and the rest

The risk family is Murex's moat, and the verified capability list is rich (murex.com treasury/risk pages; NVIDIA collaboration; IBS Intelligence verified profile):

- **Market risk** ✅ — **historical VaR**, **expected shortfall**, stress testing, sensitivities (Greeks, DV01), P&L explanation, horizon shifting ("shifting horizon" is Murex's own wording for re-evaluating risk at future dates), and **what-if simulation** ("executing what-if simulations" and "applying market data scenarios" are Murex's own phrasing).
- **Simulation and scenario analysis** ✅ — the platform lets users shock market data, run stress scenarios, and reprice the book; this is the machinery behind regulatory stress testing (e.g., CCAR/ICAAP-style programmes ⚠ regulatory mapping is the bank's responsibility, MX.3 provides the engine).
- **Credit risk** ✅ — counterparty exposure, limits, CVA/DVA; the [Banking Limits Domain Guide](banking_limits_domain_guide.md) covers the limits discipline this plugs into.
- **Collateral management** ✅ — margin calls, collateral optimisation, CSA workflows (Murex's operations & finance family explicitly covers collateral management).
- **XVA** ✅ — CVA/DVA/FVA-type analytics; NVIDIA documents Murex testing GPU acceleration for XVA and market-risk calibration on Grace Hopper hardware — a useful datapoint on where Murex's compute pressure sits.
- **Regulatory capital** ⚠ — FRTB/SBM support is widely reported in industry press but not re-verified on murex.com in this pass; flagged.
- **Fund transfer pricing / ALM** ⚠ — treasury-side analytics claimed in vendor material; not independently verified.

### 5.3 The modules table

| Module (solution family) | Function | Notes |
|---|---|---|
| **Trading — pricing & trade capture** | Price FX/rates/credit/commodity products, capture trades, manage the trading book | Cross-asset model library; front-office P&L ✅ |
| **Trading — e-trading & execution** | Connectivity to venues, execution workflows, quote management | ⚠ breadth per vendor pages |
| **Treasury management** | Cash/liquidity, funding, money market, FX for the balance sheet, Islamic finance conventions | Murex's packaged MXGO variant exists ✅ |
| **Market risk** | Historical VaR, expected shortfall, stress testing, sensitivities, P&L explanation, horizon shifting, what-if simulation | The flagship risk module family ✅ |
| **Credit risk** | Counterparty exposure, limits, CVA/DVA | Plugs into the bank's limits framework — see [Banking Limits Domain Guide](banking_limits_domain_guide.md) ✅ |
| **Collateral management** | Margin calls, collateral optimisation, CSA workflows | Operations/finance family ✅ |
| **XVA analytics** | CVA/DVA/FVA-type valuation adjustments | GPU acceleration being explored (NVIDIA) ✅ |
| **Operations** | Confirmations, settlement, payments, STP workflows | SWIFT/FIX/electronic matching ✅ |
| **Finance** | Accounting entries, P&L, general-ledger feeds, cost/result allocation | Feeds the core's ledger — see [Posting Engine Core Banking Guide](posting_engine_core_banking_guide.md) ✅ |
| **Regulatory reporting** | Capital, liquidity, trade and risk reporting (EMIR, SFTR, MAS 610, FRTB ⚠) | Framework coverage per jurisdiction ⚠ mapping is bank-specific |
| **Datamart / reporting** | Analytical data mart, report definitions | ⚠ ecosystem-documented |
| **MXML layer** | Interfaces, batch, customisation, data exchange | The platform's scripting/data language ⚠ ecosystem-documented |

---

## 6. Market Data

### 6.1 How MX.3 handles market data

Market data is the oxygen of a trading/risk platform: every price, every curve, every volatility surface, every FX rate that flows into valuation and risk must be captured, stored, and versioned. MX.3's market-data handling is documented by Murex at two levels ✅:

1. **Consumption** — MX.3 ingests market data from vendor feeds through **packaged interfaces** (Murex's connectivity page: "packaged interfaces between MX.3 and relevant third-party platforms… encompass the data sourcing and trade processing life cycle. This ranges from market and reference data to execution, post-trade processing, clearing and settlement") ✅. The classic vendors in the capital-markets space are the Reuters/Refinitiv (now LSEG) and Bloomberg families, plus regional providers (e.g., Quick for Japan); Murex supports the standard feed formats, but **specific vendor certification lists are ⚠** (vendor material, not re-verified in this pass).
2. **Management** — MX.3 stores market data in its own curve/volatility infrastructure: the platform maintains yield curves, FX forward points, volatility surfaces, and correlations, and it versions them so that historical valuation ("what was the price on day X?") and scenario runs work ✅ (implied by the risk capabilities: "applying market data scenarios, shifting horizon" requires versioned, storable market states — Murex's own wording).

The market-data capability that makes MX.3 interesting to risk teams is **scenario-ing**: because market data lives inside the platform as first-class objects, the risk engine can shock it (parallel shifts, curve twists, vol moves, historical re-runs) without leaving the system ✅.

### 6.2 The market data table

| Category | What MX.3 does | Notes |
|---|---|---|
| **Real-time prices** | Ingest vendor ticks/quotes for pricing and P&L | Feed adapters via packaged interfaces ✅; vendor list ⚠ |
| **Curves & surfaces** | Build/maintain yield curves, FX forwards, vol surfaces, correlations | Core valuation inputs ✅ |
| **Reference data** | Instruments, calendars, holidays, counterparties, settlement conventions | Sourced via data interfaces ✅ |
| **Historical data** | Versioned market states for backtesting, historical VaR, stress re-runs | Implied by historical VaR and scenario features ✅ |
| **Scenario & stress data** | Shocked markets, hypothetical shocks, horizon shifting | Murex's own documented capability ✅ |
| **Governance** | Market-data validation, rate locks (e.g., fixing curves), audit of data used in valuations | ⚠ operational detail, ecosystem-documented |

## 7. Integration

### 7.1 The integration philosophy

A capital-markets platform is useless in isolation: it must consume market data, receive trades from e-trading venues and order-management systems, exchange confirmations with counterparties, settle through payments rails, and feed the general ledger, the data warehouse, and regulators. Murex's integration story has three layers, all verified ✅:

1. **Packaged interfaces** — Murex ships pre-built connectors "between MX.3 and relevant third-party platforms to reduce deployment costs and time to market… encompassing the data sourcing and trade processing life cycle. This ranges from market and reference data to execution, post-trade processing, clearing and settlement" (murex.com connectivity page) ✅. In practice this is the adapter library for the feeds and venues the industry actually uses — SWIFT, FIX, market-data feeds, clearing/settlement utilities (e.g., CLS for FX, DTCC/EMCF-class utilities ⚠ specific-utility coverage per vendor documentation).
2. **MXML** — the platform's own interface language (see [§4.3](#43-mxml--the-murex-language)): any system that can emit or consume MXML-shaped messages can exchange data with MX.3 without touching the database ⚠ ecosystem-documented.
3. **APIs** — Murex's "next-generation APIs" push integration toward programmatic access: "Automate more, operate smarter… leveraging MX.3 as a source of truth with automated data flows. APIs enable straight-through processing — streaming trades, market data, and results across your architecture" (murex.com MX.3 APIs page) ✅. This is the modern face of MX.3 integration and the entry point for the bank's own innovation layer (in-house apps, data platforms, and AI/ML tooling can read MX.3's streams).

The integration boundary that matters for a bank architect: **MX.3 is the system of record for the trading book, but it is not the system of record for the banking book.** It hands the customer-side records to the core — the [Core Banking Systems Guide](core_banking_systems_guide.md) §2.4 states the boundary exactly: "Treasury systems (Murex MX.3, Calypso…) manage the trading book; the core manages the banking book's customer accounts. Treasury *funds* the balance sheet; the core records the customer side of it."

### 7.2 What the interfaces actually carry

Verified categories from Murex's connectivity page + ecosystem documentation ✅:

- **Market and reference data in** — prices, curves, reference data (the [§6](#6-market-data) story).
- **Execution in/out** — orders and fills to/from e-trading venues (FIX is the canonical protocol here ✅ per ecosystem test documentation: "Validation of SWIFT, FIX, and market data feed interfaces… along with downstream connections to general ledger, reconciliation, and reporting systems — ensuring clean straight-through processing").
- **Post-trade out** — confirmations (SWIFT MT300/MT3xx-class, electronic matching), settlement instructions, collateral messages, clearing submissions ✅.
- **Finance out** — accounting entries and P&L to the general ledger; reconciliation feeds; reporting to the Datamart and to regulatory platforms ✅.
- **Banking-book in/out** — funding instructions, nostro/payments to the payments stack (cross-ref [Payments Hub Guide](payments_hub_guide.md) and the [Financial Infrastructure Guide](financial_infrastructure_guide.md) for the rails themselves) ⚠ pattern-level.

### 7.3 The integration table

| Interface | Direction | Protocol/format | Notes |
|---|---|---|---|
| **Market data feeds** | In | Vendor feeds (LSEG/Refinitiv-class, Bloomberg-class ⚠) | Packaged adapters ✅; vendor list ⚠ |
| **Reference data** | In | Vendor/in-house MDM feeds | Calendars, instruments, counterparties ✅ |
| **Execution / e-trading** | In/out | FIX-class | Orders and fills ✅ (protocol ⚠ per venue) |
| **Confirmation & matching** | Out | SWIFT-class, electronic matching | Post-trade STP ✅ |
| **Settlement & clearing** | Out | SWIFT, clearing utilities | CLS-class for FX ⚠ per vendor docs |
| **Payments / nostro** | Out | SWIFT/ISO 20022-class | To the payments stack ⚠ pattern-level |
| **General ledger / finance** | Out | GL feeds (MXML/APIs) | Accounting entries ✅ |
| **Data warehouse / analytics** | Out | APIs, MXML, Datamart extracts | "Streaming trades, market data, results" ✅ |
| **Regulatory reporting** | Out | Jurisdiction formats | EMIR/SFTR/MAS-class ⚠ mapping bank-specific |
| **Bank's own systems** | Both | Next-gen APIs | The innovation boundary ✅ |

### 7.4 The integration scenario: an FX swap end-to-end

To make the interface catalogue concrete, trace an FX swap (spot vs forward legs) through Meridian-style integration:

| Step | System | Interface | Data |
|---|---|---|---|
| Deal done on an e-trading venue | Venue → MX.3 | FIX-class execution feed | Order/fill details |
| Trade captured & enriched | MX.3 | internal | Counterparty, legs, settlement dates |
| Confirmation to counterparty | MX.3 → SWIFT | SWIFT MT300-class (or matching service) | Economic terms |
| Market data for valuation | LSEG-class feed → MX.3 | Packaged market-data adapter ⚠ | FX rates, forward points |
| Funding instructions | MX.3 → payments stack | SWIFT/ISO 20022-class ⚠ | Payment orders (cross-ref [Payments Hub Guide](payments_hub_guide.md)) |
| Accounting | MX.3 → core GL | GL feed (MXML/API) | Journal entries (cross-ref [Posting Engine Core Banking Guide](posting_engine_core_banking_guide.md)) |
| Risk results | MX.3 → risk warehouse | API/Datamart extract | Positions, VaR, scenarios |
| Regulatory | MX.3 → regulator/reporting hub | Jurisdiction formats ⚠ | Trade reports |

The pattern to notice: **MX.3 is the hub.** Every arrow either terminates in or originates from the platform's own interfaces, which is exactly why Murex markets "packaged interfaces to reduce deployment costs and time to market" ✅ — and why a bank's integration architects spend most of their effort on the *other* end of each arrow (the core, the payments hub, the data warehouse), not on MX.3 itself.

---

## 8. Clients

### 8.1 DBS: the series anchor (2014)

The [DBS Software Systems Guide](dbs_software_systems_guide.md) is the anchor for Murex in this series, and its verified record is:

- **2014 — DBS adopts Murex MX.3 for risk management** ✅ (the DBS guide §1.3 lists it as verified: "the Murex MX.3 adoption for risk management in 2014 (Wikipedia, referencing Murex's public record)"; its §3 systems table records "Murex MX.3 (2014, risk)" under Markets/treasury).
- **Scope nuance** — the DBS guide marks the *scope* of Murex at DBS (risk only vs front-to-back) as ⚠ inferred: DBS's public record is thin, and the guide records "no Calypso/Kondor evidence" at DBS (i.e., MX.3 is the documented markets platform there).
- **2017 — MX.3 on AWS** ✅ (WatersTechnology, December 2017: Murex announced MX.3 on AWS with DBS as the reference client; then-DBS group CIO David Gledhill was quoted on expected infrastructure-cost reductions). This makes DBS one of the earliest public MX.3-on-cloud references — an important datapoint for [§10](#10-worked-example-a-bank-treasury-implementation).
- **Context** — DBS's Global Financial Markets (GFM) business runs e-trading, FX, rates and derivatives out of Singapore; MX.3 is the markets estate platform of record in this series (see the [DBS guide](dbs_software_systems_guide.md) §5 and §11).

### 8.2 Other documented clients

Beyond DBS, the publicly documented client set in this research pass (kept deliberately small — Murex does not publish a client list, and naming banks on analyst lists without verification would violate this guide's discipline):

- **Aldermore Bank (UK)** — 2025: Murex's **MX.3 front-to-back-to-risk platform and its packaged treasury solution MXGO** went into production for Aldermore's treasury function, delivered with Publicis Sapient; MX.3 SaaS (Murex-hosted) is part of the deal ✅ (IBS Intelligence + Murex announcement). A clean example of a mid-size bank buying the *treasury* pillar rather than the full trading estate.
- **BIBD (Brunei)** — Islamic-finance treasury implementation ("BIBD Embraces Murex Solution for Bank Treasury with Islamic Finance") ✅ (murex.com case reference) — evidence for the Islamic-finance conventions claim in [§3](#3-mx3-overview).
- **A Canadian bank (unnamed)** — migration to MX.3 documented in an Accenture case study ✅ (bank name withheld in the case study; useful as evidence of the migration/modernisation pattern).
- **The anonymous majority** — 60,000+ daily users across banks, asset managers, insurers, pension funds and energy firms ✅ (murex.com); specific names beyond the above are ⚠ (commonly cited in industry press — BNP Paribas, Societe Generale, Standard Chartered and others appear in analyst databases — but not re-verified in this pass; the [ZoomInfo/Apps Run The World](https://www.appsruntheworld.com/customers-database/vendors/view/murex) class of lists is directional, not evidence).

### 8.3 The clients table

| Client | Region | Notes |
|---|---|---|
| **DBS** | Singapore / Asia | MX.3 for risk management since **2014** ✅; AWS move announced 2017 ✅; see [DBS guide](dbs_software_systems_guide.md) §5 |
| **Aldermore Bank** | UK | 2025 treasury production: MX.3 + MXGO + SaaS with Publicis Sapient ✅ |
| **BIBD** | Brunei | Islamic-finance bank treasury on Murex ✅ |
| **Unnamed Canadian bank** | North America | MX.3 migration via Accenture (case study) ✅ |
| **60,000+ daily users** | Global (~70 countries) | Banks, asset managers, insurers, pension funds, energy firms ✅ (⚠ per-segment split not disclosed) |
| **BNP Paribas / SocGen / StanChart-class names** | Global | ⚠ widely cited in analyst databases and press; not re-verified in this pass |

---

## 9. Competition

### 9.1 Calypso — the perennial rival (now Nasdaq)

Calypso is the competitor the market always pairs with Murex, and the series already owns its deep-dive: the [Nasdaq Calypso Guide](nasdaq_calypso_guide.md). The verified lineage (this pass):

- **Calypso Technology, founded 2001** ✅ — cross-asset trading, treasury, and risk platform with a strong treasury/ALM footprint.
- **2021 — Adenza**: Thoma Bravo merged Calypso ($3.7B acquisition) with AxiomSL (regulatory reporting, $2B, 2020) to form **Adenza Group** ✅.
- **2023 — Nasdaq**: Nasdaq acquired Adenza for **$10.5B** (November 2023); Calypso now lives as **Nasdaq Calypso** within Nasdaq's Capital Markets Technology division ✅.

Why the comparison is structurally interesting:

- **Ownership:** Murex is independent/private; Calypso is now part of a listed exchange group with a compliance-tech sibling (AxiomSL). Buyers weigh Murex's stability-by-independence against Nasdaq's balance-sheet backing and cross-sell.
- **Centre of gravity:** Murex's deepest bench is risk analytics (VaR, XVA) and large-bank front-to-back; Calypso's classic strengths are treasury/liquidity and mid-tier banks, plus post-Adenza regulatory breadth (AxiomSL).
- **The Gartner/analyst picture:** both platforms sit in the same trading-platform and treasury-system markets; "Murex vs Calypso" is a standard RFP shortlist for banks replacing legacy trading/treasury estates (⚠ analyst-market framing; exact market-share splits are not reliably public).

### 9.2 The wider field

Beyond Calypso, the competitive set a bank's RFP will include (⚠ all of these are industry-knowledge placements; none were deep-verified in this pass — see the series guides for the ones that have their own):

- **FIS Front Arena** (and the FIS legacy trio Kondor/Summit) — cross-asset trading/risk, strong in the tier-1 derivative niche but with an ageing-install-base reputation.
- **Finastra Fusion Treasury / Kondor lineage** — mid-market treasury and trading; strong in the treasury-management-system (TMS/TRE) segment.
- **ION / Openlink-class (E/CTRM and treasury)** — energy/commodity and corporate treasury; competes with Murex in the commodity segment (Murex's E/CTRM push).
- **BlackRock Aladdin** — the buy-side giant; overlaps with Murex at asset managers and increasingly at banks' portfolio-risk desks (⚠ buy-side focus).
- **Temenos / Oracle Flexcube treasury modules** — the core-vendor alternative: banks already on [Temenos T24](temenos_t24_guide.md) or [Oracle Flexcube](oracle_flexcube_data_model_guide.md) sometimes buy the vendor's treasury module instead of a standalone platform (⚠ limited derivatives depth — the classic trade-off, see [Core Banking Systems Guide](core_banking_systems_guide.md) §2.4).

### 9.3 The comparison table

| Platform | Strengths | Notes |
|---|---|---|
| **Murex MX.3** | Risk analytics depth (VaR/ES/XVA), front-to-back single data model, cross-asset derivatives, 40-year track record, independence | Private; scarce MXML skills; implementations need partners ✅ |
| **Nasdaq Calypso** | Treasury/ALM strength, mid-tier-bank footprint, AxiomSL regulatory sibling, Nasdaq backing | Owned by Nasdaq since 2023 ($10.5B Adenza deal) ✅; see [Nasdaq Calypso Guide](nasdaq_calypso_guide.md) |
| **FIS Front Arena / Kondor / Summit** | Tier-1 derivatives heritage, FIS ecosystem | ⚠ legacy-install-base reputation |
| **Finastra Fusion (Kondor lineage)** | Mid-market treasury, bank-core adjacency | ⚠ |
| **BlackRock Aladdin** | Buy-side portfolio/risk analytics at scale | ⚠ buy-side centre of gravity |
| **Core-vendor treasury modules (Temenos, Flexcube)** | Cheap, integrated with the core, good for vanilla treasury | ⚠ derivatives/risk depth limits — see [Temenos T24 Guide](temenos_t24_guide.md), [Oracle Flexcube Guide](oracle_flexcube_data_model_guide.md) |

### 9.4 Choosing between Murex and Calypso: the decision lens

When a bank runs the classic "Murex vs Calypso" RFP (⚠ analyst-market framing; both are credible), the decision usually comes down to five questions — and the answers, in practice, are the ones below:

1. **What is the centre of gravity of the estate?** A bank replacing a big derivatives/risk estate leans Murex (risk analytics depth, XVA, large-bank front-to-back). A bank whose pain is treasury/liquidity/ALM leans Calypso's traditional strengths.
2. **What is the regulatory story?** Post-Adenza, Calypso brings AxiomSL's regulatory-reporting muscle; Murex counters with in-platform reporting and its own regulatory track record. ⚠ both claims are vendor-positioning; the bank's own framework mapping decides.
3. **Who will run it?** Murex's MXML skills are scarce and partner-dependent; Calypso's ecosystem has its own (different) skill profile. Implementation risk is a real tiebreaker.
4. **Ownership risk.** Private/family Murex (no one can buy it and change the roadmap overnight) vs Nasdaq-listed Calypso (deep pockets, but an exchange-group strategy overlays the product). Different risk profiles, no objectively right answer.
5. **What does the series already know?** The [Nasdaq Calypso Guide](nasdaq_calypso_guide.md) has the full Calypso side; the DBS precedent in the [DBS guide](dbs_software_systems_guide.md) shows Murex winning a major Asian bank's risk estate in 2014 — and DBS remains a Murex shop with no Calypso evidence ⚠ scope nuance as recorded there.

---

## 10. Worked Example: A Bank Treasury Implementation

### 10.1 The scenario: a mid-size bank's treasury

**Setting** — Meridian Bank (fictional), a mid-size regional bank in Singapore with a growing balance sheet: deposits from retail/SME, an FX and money-market desk, an interest-rate-swap book for corporate clients, and a funding function that must manage liquidity, LCR/NSFR, and MAS reporting. Meridian's current estate is the classic problem: a legacy Kondor/Flexcube-treasury hybrid (⚠ generic) for trading, Excel for risk, and a reconciliation nightmare between front-office P&L, risk numbers, and the ledger.

**Why MX.3, in one paragraph** — the treasury is the *banking book's* market interface: it funds the balance sheet, manages liquidity, and runs the modest trading book. Meridian's drivers mirror the documented Aldermore case ([§8.2](#82-other-documented-clients)): one front-to-back system for treasury operations, real-time risk, and regulatory output, with the [Core Banking Systems Guide](core_banking_systems_guide.md) §2.4 boundary respected — MX.3 owns the trading book; the core (Flexcube-class, cross-ref [Oracle Flexcube Data Model Guide](oracle_flexcube_data_model_guide.md)) owns customer accounts; and the [Banking Limits Domain Guide](banking_limits_domain_guide.md) limits discipline is enforced at the MX.3 layer for markets exposures.

### 10.2 The deployment: MX.3 modules design

Meridian's scope-of-modules (solution-family) design, mapped to the [§5](#5-modules) families:

| MX.3 family | Meridian scope | Notes |
|---|---|---|
| **Treasury management** | Money market, FX spot/forwards/swaps, funding, cash/liquidity | The heart of the deal; MXGO-packaged option evaluated (as Aldermore did) ✅ pattern |
| **Trading — pricing & capture** | Vanilla IRS, FX options (limited), NDFs | Corporate-hedging book; no exotic book in phase 1 |
| **Market risk** | Historical VaR (1-day/10-day), ES, stress scenarios, P&L explanation | Daily EOD VaR + intraday limit checks ✅ |
| **Credit risk & limits** | Counterparty limits for the derivative book, CVA at portfolio level | Plugs into Meridian's group limits framework — cross-ref [Banking Limits Domain Guide](banking_limits_domain_guide.md) |
| **Collateral** | CSA management for the swap book (phase 2) | Phased: defer margin automation |
| **Operations** | Confirmations (SWIFT MT300), settlement instructions, STP | Legacy ops team retrained, not replaced |
| **Finance** | Accounting entries, P&L, GL feeds to the core's ledger | Cross-ref [Posting Engine Core Banking Guide](posting_engine_core_banking_guide.md) for the ledger mechanics |
| **Regulatory** | MAS 610 (risk), EMIR trade reporting, liquidity data for LCR/NSFR | ⚠ mapping is Meridian's responsibility; MX.3 provides data + reporting framework |
| **Datamart / reporting** | Treasury dashboards, risk pack, board pack | Replaces the Excel estate |
| **Integration layer** | LSEG market data ⚠, SWIFT, core-banking GL feeds, payments hub | Via packaged interfaces + MXML + next-gen APIs ✅ |

**Deployment mechanics** — a 12–18 month programme, partner-led (Accenture/Publicis Sapient-class; ⚠ Murex implementations are partner-heavy because MXML skills are scarce):

1. **Wave 0 — foundation (months 1–4):** platform install (on-premise or AWS per the DBS precedent ✅), market-data connectivity, reference-data onboarding, MXML interface skeleton, security/identity (cross-ref [Distributed Auth Guide](../technology/distributed_auth_guide.md) for the pattern of integrating a platform's own auth with the bank's SSO).
2. **Wave 1 — treasury go-live (months 5–9):** money market + FX in production; EOD batch, P&L, GL feeds; parallel-run against legacy.
3. **Wave 2 — derivatives + risk (months 10–14):** IRS/NXD book, daily VaR/ES, stress scenarios, limits enforcement; decommission legacy Kondor-class trading system.
4. **Wave 3 — regulatory + optimisation (months 15–18):** MAS/EMIR reporting, collateral phase 2, dashboarding; legacy Excel risk estate retired.

### 10.3 The lessons (what the series has taught us)

1. **Front-to-back is the point.** Meridian's single biggest win is not any module — it is that the trade booked at 10:00 is the same record that appears in VaR at 17:00, in SWIFT confirmations at 18:00, and in the month-end ledger. The DBS guide's risk-data discipline (BCBS 239-class) is nearly free when the platform is one data model.
2. **The core boundary must be drawn early.** MX.3 funds the balance sheet; the core records the customer side ([Core Banking Systems Guide](core_banking_systems_guide.md) §2.4). Meridian's GL interfaces and nostro/payments flows are where most project risk hides — budget for them ([Payments Hub Guide](payments_hub_guide.md), [Financial Infrastructure Guide](financial_infrastructure_guide.md)).
3. **MXML is a skill, not a feature.** Customisation in MXML is powerful and expensive; the discipline is *configure, then customise, and treat every MXML deviation as a maintenance liability* (⚠ ecosystem wisdom).
4. **Limits and risk are a domain, not a module.** MX.3 enforces limits; the bank designs them ([Banking Limits Domain Guide](banking_limits_domain_guide.md)). VaR methodology choices (historical window, ES vs VaR, horizon) are bank decisions made on MX.3's machinery.
5. **Cloud is proven but not free.** The DBS/AWS precedent (2017 ✅) shows MX.3 moves; but EOD batch tuning, database licensing (Oracle/Sybase/SQL Server), and latency to market-data feeds all need design ([Distributed Auth Guide](../technology/distributed_auth_guide.md) light cross-ref for the platform-security layer).
6. **Phasing protects the bank.** Treasury first (Aldermore-style), derivatives and deep risk second — matches how Murex's own client announcements have unfolded ✅ (DBS: risk first; Aldermore: treasury first).

### 10.4 Operating model, governance, and success measures

A platform is only as good as the operating model around it. For Meridian (and any MX.3 estate), the post-go-live shape looks like this:

- **Ownership:** a markets-platform owner (usually treasury/COO office) accountable for the platform; the risk function owns methodology (VaR window, ES calibration, stress scenarios — MX.3 provides the engine, the bank owns the policy, cross-ref [Banking Limits Domain Guide](banking_limits_domain_guide.md)).
- **Support model:** tier-1 (bank's own front-office/ops support), tier-2 (bank's Murex/MXML specialists), tier-3 (Murex or partner). Murex implementations typically retain a partner for the first years ⚠ ecosystem pattern.
- **Change governance:** MXML customisations are code — versioned, reviewed, tested in UAT, promoted through environments like any software change ⚠ ecosystem standard. Every customisation is a maintenance liability; the roadmap should shrink the customisation surface back toward configuration.
- **Data governance:** market-data validation, reference-data ownership, and the BCBS 239-class risk-data discipline the [DBS guide](dbs_software_systems_guide.md) documents — MX.3 makes the *platform* consistent; the *bank* must make the people/process side consistent.

**KPIs that prove the platform is working:**

| KPI | What it measures | Before → after (Meridian-style) |
|---|---|---|
| **STP rate** | % of trades confirmed/settled without manual touch | ~60% → 95%+ ⚠ illustrative |
| **Reconciliation breaks** | Front-office vs risk vs finance differences | Daily Excel war → near zero ⚠ illustrative |
| **EOD batch window** | Time from market close to risk/accounting ready | 6h → 2h ⚠ illustrative |
| **Regulatory submissions** | Days to produce MAS/EMIR reports | 5 days → T+1 ⚠ illustrative |
| **Limit breaches** | Count/severity of limit violations detected | After-the-fact → intraday ⚠ illustrative |

⚠ The numbers are illustrative, not Murex-published benchmarks — but they are the *shape* of the business case that platforms like MX.3 sell, and they match the efficiency claims in Murex's treasury positioning ("maximizes STP, decreases operational risk, and enhances operational control" — murex.com treasury page ✅).

---

## 11. Summary: One Page

**Murex** — founded in Paris in 1986 by Salim Eddé and Laurent Néel, still private and family-led (CEO Elias Eddé), quietly one of the most consequential software companies in finance: 3,400+ employees, ~20 offices, 60,000+ daily users in 60+ countries. Its flagship platform, **MX.3**, is the capital-markets system of record for a large share of the world's trading banks.

**The platform** — front-to-back by design: trading, treasury, risk, post-trade operations, finance, and regulatory reporting on one data model, across FX, rates, credit, commodities, and treasury products. Three pillars define its positioning: trading (front office), risk (middle office), and treasury (balance sheet).

**The architecture** — client-server, tiered, service-oriented: desktop and web clients over presentation/business/orchestration/technical layers; Oracle, Sybase, or SQL Server underneath; a Datamart for reporting; and **MXML** — the platform's proprietary interface/scripting language — as the glue that makes integration flexible and skills scarce.

**The modules** — the risk family is the moat: historical VaR, expected shortfall, stress testing, what-if simulation, credit risk, collateral, XVA. Around it: pricing/trade capture, treasury management, operations, finance, regulatory reporting, the Datamart.

**The ecosystem** — market data via packaged interfaces and versioned in-platform market states; integration via packaged connectors (market data, execution, SWIFT/FIX, clearing, GL), MXML, and next-generation APIs.

**The clients** — anchored in this series by **DBS, on MX.3 for risk management since 2014**, and on AWS since 2017 (see the [DBS Software Systems Guide](dbs_software_systems_guide.md)); documented regional adopters include Aldermore (treasury, MXGO, SaaS) and BIBD (Islamic treasury); the anonymous majority spans banks, asset managers, insurers, and energy firms.

**The competition** — **Calypso**, now Nasdaq Calypso after the 2023 $10.5B Adenza acquisition (see the [Nasdaq Calypso Guide](nasdaq_calypso_guide.md)), is the perennial rival; FIS Front Arena, Finastra, ION, Aladdin, and core-vendor treasury modules fill out the RFP.

**The pattern for a bank** — a phased treasury implementation (foundation → treasury → derivatives/risk → regulatory) on MX.3's single data model replaces the reconciliation estate, enforces limits, and feeds the core — with MXML customisation cost, the core boundary, and methodology choices as the three risks to manage.

**The final word:** in the capital-markets platform market, MX.3 is the closest thing the industry has to a standard — the platform a bank buys when it wants one system of record from the trader's ticket to the regulator's return. It is, in short, **the front-to-back treasury standard**.

---

## 12. Glossary

| Term | Definition |
|---|---|
| **Murex** | French capital-markets software company founded 1986 in Paris; private, family-led; vendor of MX.3 |
| **MX.3** | Murex's flagship platform: front-to-back capital-markets software (trading, treasury, risk, post-trade, regulatory) |
| **Front-to-back** | Architecture where one platform/data model spans the whole trade lifecycle — front office (trading) through back office (settlement, accounting, reporting) |
| **Trading** | The business of buying/selling financial instruments (FX, derivatives, securities) for clients or the bank's own account |
| **Risk** | The discipline of measuring and managing uncertainty of loss — market risk, credit risk, operational risk, liquidity risk |
| **Treasury** | The balance-sheet function that manages a bank's funding, liquidity, cash, and money-market/FX positions |
| **Client-server** | Architecture where client programs (desktop/web) talk to shared servers (application/database) — MX.3's model |
| **Front office** | The traders and sales desks; in software, the pricing/trade-capture systems they use |
| **Middle office** | Risk management, limits, and trade verification between front and back offices |
| **Back office** | Confirmations, settlement, payments, accounting — the post-trade machinery |
| **Pricing** | Computing the value/price of a financial instrument from market data and models |
| **Valuation** | Marking positions to market; the basis of P&L and risk measurement |
| **VaR** | Value at Risk — a quantile measure of potential loss over a horizon/confidence level (e.g., 99% 10-day) |
| **Expected shortfall (ES)** | Average loss beyond the VaR quantile — a tail-risk measure favoured by regulators |
| **Simulation** | Recomputing portfolio value under hypothetical or historical market scenarios (what-if, stress) |
| **MXML** | Murex XML — the platform's proprietary language for interfaces, batch, configuration, and data exchange |
| **Datamart** | MX.3's analytical reporting layer, separate from the operational database |
| **Market data** | Prices, curves, volatilities, and reference data consumed by valuation and risk |
| **Integration** | Connecting systems: feeds in, messages out, APIs — MX.3's packaged interfaces, MXML, and APIs |
| **Interface** | A defined connection/format between MX.3 and another system (SWIFT, FIX, market-data feed) |
| **DBS** | DBS Bank (Singapore) — the series anchor: MX.3 for risk since 2014, AWS since 2017 |
| **Calypso** | Calypso Technology (2001) → Adenza (2021) → Nasdaq Calypso (2023) — Murex's main rival |
| **Capital markets** | The markets for long-term funding and traded instruments — equities, bonds, FX, derivatives |
| **Derivatives** | Instruments whose value derives from underlying assets/rates — swaps, options, forwards, futures |
| **Repo** | Sale-and-repurchase agreement — collateralised short-term funding; part of securities finance |
| **FX** | Foreign exchange — spot, forwards, swaps, options on currencies |
| **XVA** | Valuation adjustments (CVA, DVA, FVA…) — credit/funding costs embedded in derivative valuation |
| **STP** | Straight-through processing — trade flows from capture to settlement without manual intervention |
| **ALM** | Asset-liability management — managing the balance sheet's rate/liquidity risks |

---

## 13. Verification Notes and Sources

### 13.1 Verified claims (✅)

- Murex founded 15 September 1986 in Paris; founders Salim Eddé and Laurent Néel (Wikipedia EN/FR; corroborating interviews and corporate history).
- Elias Eddé (co-founder, Salim's brother) is CEO; Eddé family leadership (industry profiles, e.g., Craft).
- Company private/independent; organic growth; ~3,400+ employees, ~20 offices; 60,000+ daily users in 60+ countries (murex.com) / customers across ~70 countries (Wikipedia) — minor variance between sources flagged.
- MX.3 is Murex's flagship platform; coverage = trading, treasury, risk, post-trade operations (murex.com); cross-asset (FXD, IRD, credit, commodity derivatives — murex.com).
- Tiered, service-oriented, client-server architecture; desktop + web clients; presentation/business/orchestration/technical layers (murex.com architecture page).
- Databases: Oracle, Sybase, SQL Server; SQL Server 2019 supported from MX.3 v3.1.48 (Microsoft Learn, MX.3-on-Azure reference architecture).
- Risk capabilities: historical VaR, expected shortfall, stress testing, P&L explanation, what-if simulation, market-data scenarios, horizon shifting (murex.com treasury page).
- XVA and market-risk GPU acceleration work with NVIDIA (NVIDIA blog).
- Packaged interfaces for market/reference data, execution, post-trade, clearing, settlement (murex.com connectivity page); next-generation APIs (murex.com MX.3 APIs page).
- SWIFT/FIX/market-data-feed interface testing pattern (implementation-partner documentation).
- DBS: MX.3 for risk management since 2014 (per the [DBS guide](dbs_software_systems_guide.md) verified claims, referencing Wikipedia/Murex public record); MX.3 on AWS announced December 2017 with DBS CIO comment (WatersTechnology).
- Aldermore: MX.3 + MXGO treasury production with Publicis Sapient, MX.3 SaaS, 2025 (IBS Intelligence; Murex announcement).
- BIBD: Islamic-finance bank treasury on Murex (murex.com case reference).
- Calypso lineage: founded 2001; Thoma Bravo → Adenza (Calypso $3.7B 2021, AxiomSL $2B 2020); Nasdaq acquired Adenza $10.5B, November 2023 (Wikipedia/Adenza; Nasdaq newsroom; Markets Media).
- Accenture describes itself as Murex's sole global partner (Accenture case study — vendor claim, flagged as such).

### 13.2 Flagged claims (⚠)

- **Brief correction:** "Laurent Foata" is NOT a verified Murex founder — the verified second founder is Laurent Néel. "Laurent Foata" resolves to a French growth-equity investor (AgilaGrowth/Ardian) unrelated to Murex.
- Formal MX.3 module names (Murex publishes solution families, not a module catalogue); MXML details (ecosystem-documented); Datamart internals; EOD batch specifics.
- Market-data vendor certification lists; specific clearing-utility adapters (CLS-class); regulatory framework coverage (FRTB, EMIR, SFTR, MAS 610 mapping).
- Named clients beyond DBS/Aldermore/BIBD (BNP Paribas, Societe Generale, Standard Chartered appear in analyst databases — directional, not verified here).
- Murex market share / revenue / licence counts — no public figures; treat analyst numbers as estimates.
- Murex 2000 / MXG.2000 legacy platform generation (ecosystem/job-ad references).
- MXGO packaged-treasury detail beyond its verified existence (Aldermore announcement).
- E/CTRM positioning claims and "rankings" (vendor marketing).

### 13.3 Sources

- murex.com — corporate, MX.3, treasury management, architecture, connectivity, MX.3 APIs, client/banks pages.
- Wikipedia (EN): "Murex (financial software)"; Wikipedia (FR): "Murex (entreprise)".
- WatersTechnology — "Murex to Run MX.3 Platform on AWS" (15 Dec 2017).
- Microsoft Learn — "Host a Murex MX.3 workload on Azure using SQL Server" (reference architecture).
- NVIDIA blog — "NVIDIA Grace Hopper Superchip Accelerates Murex MX.3 Analytics".
- IBS Intelligence — Murex/Aldermore announcement; "Best-in-class Treasury & Capital Markets Vendor — Murex" (verified-vendor profile).
- Accenture case study — Canadian bank MX.3 migration.
- Wikipedia — Adenza; Nasdaq newsroom — Adenza acquisition; Markets Media — Nasdaq completes Adenza acquisition.
- Craft / industry company profiles — headcount, offices, leadership.
- Implementation-partner/ecosystem documentation (AAQUILIX, ThoughtCoders, Business Compose) — MXML, Datamart, SWIFT/FIX interface patterns.
- Sibling guides: [DBS Software Systems Guide](dbs_software_systems_guide.md), [Core Banking Systems Guide](core_banking_systems_guide.md), [Nasdaq Calypso Guide](nasdaq_calypso_guide.md), [Banking Limits Domain Guide](banking_limits_domain_guide.md), [Temenos T24 Guide](temenos_t24_guide.md), [Oracle Flexcube Data Model Guide](oracle_flexcube_data_model_guide.md).

---

## Appendix A: The MX.3 Guide in Ten Claims

A quick-recall sheet — the ten claims this guide verified or flagged, in one place:

1. **Murex was founded in Paris in 1986** by Salim Eddé and Laurent Néel ✅ — and remains private, independent, and family-led (CEO Elias Eddé) ✅.
2. **MX.3 is the flagship platform**: front-to-back trading, treasury, risk, and post-trade on one data model ✅.
3. **Positioning pillars**: trading (front office), risk (middle office), treasury (balance sheet) ✅.
4. **Architecture**: client-server, tiered, service-oriented — desktop + web clients, Oracle/Sybase/SQL Server, Datamart ✅ (Datamart ⚠).
5. **MXML is the platform's language** for interfaces, batch, and customisation ⚠ — powerful, scarce-skilled, partner-driven.
6. **Risk is the moat**: historical VaR, expected shortfall, stress testing, what-if simulation, credit risk, collateral, XVA ✅.
7. **Market data is first-class**: packaged feed interfaces in; versioned curves/surfaces/scenarios in the platform ✅ (vendor lists ⚠).
8. **Integration is packaged**: interfaces for market data, execution, SWIFT/FIX, clearing, GL + next-gen APIs ✅.
9. **Clients**: DBS on MX.3 for risk since 2014 ✅ and on AWS since 2017 ✅ (cross-ref the [DBS guide](dbs_software_systems_guide.md)); Aldermore (treasury/MXGO/SaaS, 2025) ✅; BIBD (Islamic treasury) ✅; the rest of the 60,000+ daily users are the anonymous majority.
10. **The competitor is Calypso** — now Nasdaq Calypso after the 2023 $10.5B Adenza acquisition ✅ (see the [Nasdaq Calypso Guide](nasdaq_calypso_guide.md)) — and the final word on MX.3 is that it has become **the front-to-back treasury standard**.

---

*End of guide. Research pass: 10 targeted web searches + series cross-referencing; web_extract backend degraded (search-only), so primary-source quotes are drawn from search-result snippets of murex.com and cited press, and anything not re-verified is flagged ⚠.*
