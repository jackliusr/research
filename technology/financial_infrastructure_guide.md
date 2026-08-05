# Financial Infrastructure: A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Crédit Agricole CIB, Singapore  
> **Context:** Financial Market Infrastructure (FMI) — the formal taxonomy (CPMI-IOSCO PFMI), the technology layers, the global landscape, and the banking architect's view  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** August 2026

---

## Table of Contents

1. [What Is Financial Infrastructure?](#1-what-is-financial-infrastructure)
2. [The FMI Taxonomy: Five System Types](#2-the-fmi-taxonomy-five-system-types)
   - 2.1 [Payment Systems](#21-payment-systems)
   - 2.2 [Central Securities Depositories (CSDs)](#22-central-securities-depositories-csds)
   - 2.3 [Securities Settlement Systems (SSSs)](#23-securities-settlement-systems-ssss)
   - 2.4 [Central Counterparties (CCPs)](#24-central-counterparties-ccps)
   - 2.5 [Trade Repositories (TRs)](#25-trade-repositories-trs)
3. [The PFMI Framework: Principles, Responsibilities, Systemic Importance](#3-the-pfmi-framework-principles-responsibilities-systemic-importance)
4. [FMI Risk Management](#4-fmi-risk-management)
   - 4.1 [The Risk Layers](#41-the-risk-layers)
   - 4.2 [Settlement Risk: Herstatt, DVP, PVP](#42-settlement-risk-herstatt-dvp-pvp)
   - 4.3 [Cyber Risk: The SWIFT Bangladesh Case](#43-cyber-risk-the-swift-bangladesh-case)
5. [The Market Technology Infrastructure](#5-the-market-technology-infrastructure)
   - 5.1 [Exchanges and Trading Venues](#51-exchanges-and-trading-venues)
   - 5.2 [Market Data Infrastructure](#52-market-data-infrastructure)
   - 5.3 [Reference Data](#53-reference-data)
   - 5.4 [Messaging Networks](#54-messaging-networks)
   - 5.5 [Connectivity](#55-connectivity)
6. [The Global Financial Infrastructure Landscape](#6-the-global-financial-infrastructure-landscape)
   - 6.1 [United States](#61-united-states)
   - 6.2 [Europe and the UK](#62-europe-and-the-uk)
   - 6.3 [Asia](#63-asia)
   - 6.4 [The Cross-Border Layer](#64-the-cross-border-layer)
   - 6.5 [Worked Examples: Money Through the Plumbing](#65-worked-examples-money-through-the-plumbing)
7. [The Modern Evolution of Financial Infrastructure](#7-the-modern-evolution-of-financial-infrastructure)
8. [The FMI Technology Stack](#8-the-fmi-technology-stack)
9. [The Banking Context: How Banks Sit on the Infrastructure](#9-the-banking-context-how-banks-sit-on-the-infrastructure)
10. [The Bank Architect's View](#10-the-bank-architects-view)
11. [The Singapore Context](#11-the-singapore-context)
12. [The Future of Financial Infrastructure (2026+)](#12-the-future-of-financial-infrastructure-2026)
13. [The Umbrella Summary: One-Page Map](#13-the-umbrella-summary-one-page-map)
14. [Glossary](#14-glossary)
15. [Conclusion](#15-conclusion)

---

## 1. What Is Financial Infrastructure?

**Financial infrastructure** is the system of systems that moves money, securities, and risk between financial institutions — and between institutions and their customers. It is the shared plumbing of the financial system: the networks, depositories, clearing houses, exchanges, data feeds, and message standards that every bank, broker, asset manager, and corporation relies on, directly or indirectly, to transact.

The term carries two related meanings, and both are covered in this guide:

1. **Financial Market Infrastructures (FMIs)** — the formal, regulated, systemically important systems, defined and governed by the CPMI-IOSCO *Principles for Financial Market Infrastructures* (PFMI). Per the PFMI, an FMI is *"a multilateral system among participating institutions, including the operator of the system, used for the purposes of clearing, settling, or recording payments, securities, derivatives, or other financial transactions."* There are five FMI types: **payment systems (PS), central securities depositories (CSD), securities settlement systems (SSS), central counterparties (CCP), and trade repositories (TR)**. These are the legal-and-operational cores of market plumbing — usually operated by central banks, exchanges, or industry utilities, and directly overseen by regulators.

2. **The broader market technology infrastructure** — the wider, less formally bounded layer of exchanges and trading venues, market data feeds and distribution, reference data, messaging networks (SWIFT, FIX, ISO 20022), and connectivity (co-location, extranets, direct market access) that surrounds and connects the FMIs. This is the "market infrastructure" of trading and post-trade operations — the plumbing that FMIs plug into.

A useful mental model: **FMIs are the settlement layer of record** (the systems that make transfers final and positions official), while **market technology infrastructure is the communication and discovery layer** (how participants find prices, exchange instructions, and connect). Both together are what practitioners mean when they say "market infrastructure."

```
┌──────────────────────────────────────────────────────────────────────────┐
│                    MARKET TECHNOLOGY INFRASTRUCTURE                        │
│   Exchanges & venues │ Market data (vendor/exchange feeds) │ Reference     │
│   data │ Messaging (SWIFT/FIX/ISO 20022) │ Connectivity (DMA, co-lo, extranet)│
├──────────────────────────────────────────────────────────────────────────┤
│                    FORMAL FMIs (CPMI-IOSCO PFMI)                           │
│   ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐    │
│   │ Payment  │ │   CSD    │ │   SSS    │ │   CCP    │ │  Trade       │    │
│   │ Systems  │ │(securities│ │(securities│ │(clearing/│ │  Repository  │    │
│   │ (RTGS,   │ │ custody) │ │ settlement)│ │ novation)│ │ (derivatives │    │
│   │ retail)  │ │          │ │          │ │          │ │  records)    │    │
│   └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────────┘    │
├──────────────────────────────────────────────────────────────────────────┤
│   BANKING SYSTEM:  commercial banks, brokers, asset managers, corporates  │
│   (participants, members, and users of the infrastructure above)           │
└──────────────────────────────────────────────────────────────────────────┘
```

### 1.1 Why it matters

Financial infrastructure is where **systemic risk** concentrates. A failure or compromise of a major FMI — an RTGS going down, a CCP defaulting, a CSD losing records — can freeze settlement across an entire market in minutes. This is why:

- **Central banks operate or oversee** the most critical FMIs (Fedwire, TARGET2/T2, CHAPS, BOJ-NET, MEPS+ are all central-bank-owned).
- **The 2008 crisis produced the PFMI**: after the GFC, the G20 mandated that all systemically important FMIs comply with a single global framework — the 24 Principles (see [Section 3](#3-the-pfmi-framework-principles-responsibilities-systemic-importance)) — and that standardized OTC derivatives be cleared through CCPs.
- **FMIs are "too important to fail"**: unlike banks, FMIs cannot be rescued by resolution in the same way — their failure is addressed through recovery tools (loss allocation, liquidity facilities) designed in advance.
- **They are high-value cyber targets**: the 2016 SWIFT/Bangladesh Bank heist (see [Section 4.3](#43-cyber-risk-the-swift-bangladesh-case)) showed that the interbank messaging layer itself can be weaponized.

### 1.2 Related guides in this repository

This guide is the discipline-level umbrella for several deeper sibling guides. Where a topic is covered in depth elsewhere, this guide references it rather than duplicating it:

| Topic | Sibling guide |
|---|---|
| Payment orchestration, ISO 20022, SWIFT GPI, CBPR+ | `payments_hub_guide.md` |
| Open-source retail payment switch (instant payments, interoperability) | `mojaloop_guide.md` |
| Core banking, open banking, embedded finance | `core_banking_systems_guide.md` |
| Repo/securities trading context (payments flows in collateral markets) | `asset_backed_trading_guide.md` |
| Low-latency trading, market data, co-location, deterministic systems | `low_latency_cpp_development_guide.md` |
| DLT, blockchain, CBDC mechanics | `blockchain_technology_guide.md` |
| Collateral management, SIMM, uncleared margin rules (UMR) | `nasdaq_calypso_guide.md` |
| Regulatory reporting, transaction monitoring, risk systems | `financial_risk_compliance_systems_guide.md` |
| Data models for banking and insurance (reference data, positions) | `data_models_banking_insurance_guide.md` |
| PKI, certificates, HSM, secure containers | `container_certificates_guide.md` |
| Event-driven processing, streaming risk analytics | `event_stream_processing_guide.md`, `complex_event_processing_guide.md` |
| ML in risk/surveillance | `reinforcement_learning_algorithms_guide.md` |

---

## 2. The FMI Taxonomy: Five System Types

The PFMI defines five types of FMI. Every formal FMI in the world is one (or a combination) of these. The table below summarizes them; each is then covered in detail.

| # | FMI type | What it does | Examples | PFMI principles that apply most directly |
|---|---|---|---|---|
| 1 | **Payment system (PS)** | Transfers funds between participants (banks) using central bank or commercial bank money; settles the cash leg of everything else | Fedwire, CHIPS, T2, CHAPS, MEPS+, FAST, FedNow, UPI, SWIFT (as a messaging + settlement utility) | P1–P10 (esp. P8 finality, P9 money settlement, P10 deliveries) |
| 2 | **Central securities depository (CSD)** | Safekeeps securities in book-entry form; operates the securities accounts; provides asset servicing | DTC (US), Euroclear, Clearstream, CDP (SG), JASDEC (JP), NSDL/CDSL (IN) | P11 (CSDs), P15–P17 |
| 3 | **Securities settlement system (SSS)** | Settles securities transactions (often inside or beside a CSD) — moves securities vs cash, typically DVP | T2S, CREST, Fedwire Securities (via DTC), Austraclear | P8–P10, P12 (exchange-of-value) |
| 4 | **Central counterparty (CCP)** | Interposes itself between buyer and seller of a trade; clears and nets obligations; manages default risk via margin and default funds | LCH, CME Clearing, ICE Clear, Eurex Clearing, SGX-DC, JSCC | P4–P7 (credit/collateral/margin/liquidity), P13–P14 (default management, segregation/portability) |
| 5 | **Trade repository (TR)** | Records OTC derivatives transactions (and increasingly other trades) centrally; provides the data backbone for regulators | DTCC Data Repository, CME TR, ICE Trade Vault, DTCC Data Repository (Singapore) | P23–P24 (disclosure) |

> **Note on hybrids.** Many real-world entities combine types: DTCC = CSD (DTC) + SSS + CCP (NSCC, FICC); SGX = exchange + CSD (CDP) + CCP (SGX-DC); Euroclear = ICSD + CSDs + SSS across several markets. The taxonomy classifies *functions*, not corporate entities.

### 2.1 Payment Systems

A payment system is a multilateral system that transfers funds between participants. The two fundamental design dimensions are **value** (large-value/interbank vs retail/low-value) and **timing** (real-time gross vs deferred net vs instant).

**Large-value payment systems (LVPS) / RTGS.** Real-time gross settlement systems transfer high-value payments one at a time, continuously, with **final settlement in central bank money** — the highest-quality settlement asset. Gross means each payment settles individually (no netting); real-time means immediate. RTGS is the "spine" of every modern financial system: the cash leg of securities, FX, money-market, and CLS settlements all settle through RTGS accounts.

- **Fedwire Funds Service** (US, Federal Reserve) — the dollar RTGS; ~$4 trillion/day; operated by the New York Fed; 21.5 hours/day on business days.
- **CHIPS** (US, The Clearing House) — the private-sector dollar large-value system; netting engine that settles continuously through the day and pays out a single net position over Fedwire at end of day; ~$1.8 trillion/day. Designed for high volume at lower cost than Fedwire.
- **T2** (euro area, Eurosystem) — the euro RTGS, successor to TARGET2 (live March 2023), consolidated with liquidity management; closely integrated with T2S (securities) and TIPS (instant).
- **CHAPS** (UK, Bank of England) — sterling RTGS; real-time since 1996.
- **BOJ-NET** (Japan, Bank of Japan) — yen RTGS.
- **MEPS+** (Singapore, MAS) — SGD RTGS and interbank clearing backbone (see [Section 11](#11-the-singapore-context)).
- **HVPS** within China's CNAPS — the large-value RMB RTGS.

**The world's major RTGS systems:**

| System | Currency / market | Operator | Notes |
|---|---|---|---|
| **Fedwire Funds** | USD (US) | Federal Reserve (NY Fed) | ~$4T/day; 21.5h business days; collateralized intraday overdrafts |
| **CHIPS** | USD (US) | The Clearing House | Private netting engine; final payout over Fedwire; ~$1.8T/day |
| **T2** (ex-TARGET2) | EUR (euro area) | Eurosystem | Live March 2023; integrated liquidity management with T2S/TIPS |
| **CHAPS** | GBP (UK) | Bank of England | Real-time since 1996 |
| **BOJ-NET** | JPY (Japan) | Bank of Japan | RTGS since 2001; also settles JGBs |
| **MEPS+** | SGD (Singapore) | MAS | Also clears FAST/GIRO net positions |
| **CHATS** | HKD/USD/EUR/CNH (HK) | HKMA | Multi-currency RTGS |
| **HVPS (CNAPS)** | CNY (China) | PBOC | Large-value RTGS within CNAPS |
| **RITS** | AUD (Australia) | RBA | Also the platform for NPP settlement |
| **RTGS (India)** | INR | RBI | 24x7 since 2020 |

**The instant/retail wave:**

| Scheme | Market | Launch | Notes |
|---|---|---|---|
| **FAST** | Singapore | 2014 | Settles via MEPS+ |
| **SEPA Instant (SCT Inst)** | euro area | 2017 | ~10-second response; TIPS settlement |
| **FedNow** | US | 2023 | Fed-operated; alongside The Clearing House's RTP (2017) |
| **UPI** | India | 2016 | World's largest instant scheme by volume (NPCI) |
| **PayNow** | Singapore | 2017 | Proxy-based layer on FAST |
| **PIX** | Brazil | 2020 | The fastest-growing instant scheme globally |
| **NPP** | Australia | 2018 | New Payments Platform, 24/7 |
| **FPS** | Hong Kong | 2018 | Faster Payment System, multi-currency |

RTGS is where **intraday liquidity** lives: banks must prefund their settlement positions or pledge collateral for intraday credit from the central bank (Fedwire's intraday overdrafts; T2's marginal lending facility; MEPS+ intraday liquidity). See [Section 4.1](#41-the-risk-layers) for the liquidity dimension.

**Retail / instant payment systems.** Retail systems move lower-value, high-volume payments for consumers and corporates. The modern generation is **instant/real-time** — 24/7/365, credit-push, near-instant finality:

- **FAST** (Singapore, 2014) — SGD instant transfers; settles through MEPS+.
- **PayNow** (Singapore, 2017) — proxy-based (mobile number/NRIC/UEN) instant scheme layered on FAST.
- **SEPA Instant / SCT Inst** (euro area, 2017) — instant euro credit transfers, ~10-second response.
- **FedNow** (US, launched July 2023) — the Fed's instant rail; alongside The Clearing House's **RTP** (2017).
- **UPI** (India, NPCI, 2016) — the world's largest instant payment scheme by volume.
- **PayNet/MEPS retail, NEFT** (India, 24x7 since 2019), **PayID/NPP** (Australia), **PIX** (Brazil) — the global wave of instant rails.

For the open-source implementation of an interoperable retail switch (the software that connects banks/DFSPs to instant schemes), see `mojaloop_guide.md`. Instant systems pose distinctive risk-management challenges — liquidity, fraud, and irrevocability with no time-of-day "off switch" — discussed in [Section 7](#7-the-modern-evolution-of-financial-infrastructure).

**Card networks.** Visa and Mastercard operate the two global four-party card rails (cardholder → issuer → network → acquirer → merchant). Card networks are payment systems in the PFMI sense only loosely — they clear and settle (net) between members, and in many jurisdictions the settlement is via RTGS or a dedicated settlement bank. The card rail's economics (interchange, scheme fees), authorization/clearing/settlement message flows (ISO 8583, now ISO 20022 for some flows), and fraud controls are a domain of their own; the payments hub perspective is in `payments_hub_guide.md`.

**ACH (Automated Clearing House).** Batch, deferred-net, low-cost retail/commercial payments: **NACHA** in the US (ACH network, operated by the Fed and The Clearing House), **GIRO** in Singapore, BECS in Australia, the EACHA community in Europe. ACH moves recurring debits (bills, payroll, insurance premiums) and credits in end-of-day batches; settlement is netted (ACH in the US settles next-day; Same-Day ACH added intraday windows). ACH is the "batch engine" counterpart to RTGS's real-time engine.

### 2.2 Central Securities Depositories (CSDs)

A CSD is the system that **holds securities in book-entry form** — the official record of who owns what. Physical certificates were the original form; the CSD is the dematerialization of the market. The CSD:

- **Safekeeps** securities: maintains securities accounts for participants (banks, brokers, and other CSDs), records holdings, and guarantees the integrity of the record (the "single source of truth" for ownership).
- **Settles securities transactions** (often via an integrated SSS — see 2.3): moves securities between accounts against payment (DVP).
- **Provides asset servicing**: corporate actions (dividends, interest, redemptions, rights issues, splits), proxy voting, tax services, and account statements.

Major CSDs:

- **DTC** (US) — the Depository Trust Company, part of **DTCC** (formed 1999 from the merger of DTC and NSCC); the world's largest CSD by value; US equities, corporate/municipal debt, money-market instruments.
- **Euroclear** — the Brussels-based international CSD (ICSD) plus national CSDs: Euroclear Bank (ICSD for international bonds), Euroclear UK & International (operator of **CREST**), Euroclear Belgium/Netherlands/France/Finland/Sweden.
- **Clearstream** — Deutsche Börse's CSD business: Clearstream Banking Frankfurt (German CSD) and Clearstream Banking Luxembourg (ICSD).
- **CDP** (Singapore) — The Central Depository (Pte) Limited, wholly owned by SGX; the CSD for the Singapore market.
- Others: JASDEC (Japan), NSDL/CDSL (India), CMU (HK debt), CCDC (China bonds), Austraclear (Australia), SIX SIS (Switzerland).

| CSD | Market | Notes |
|---|---|---|
| **DTC** | US | Depository Trust Company, part of DTCC; the world's largest CSD by value; equities, corporate/municipal debt, money-market instruments |
| **Euroclear Bank** | International (ICSD) | Brussels; international bonds; also settles via T2S |
| **Euroclear UK & International (CREST)** | UK/Ireland | The UK SSS+CSD; DVP in central bank money |
| **Clearstream Frankfurt / Clearstream Lux** | Germany / International (ICSD) | Deutsche Börse group; the Lux ICSD is the other eurobond hub |
| **CDP** | Singapore | SGX subsidiary; the SG securities CSD |
| **JASDEC** | Japan | Japan Securities Depository Center |
| **NSDL / CDSL** | India | The two Indian depositories (dematerialized equities/bonds) |
| **CMU** | Hong Kong | Central Moneymarkets Unit — debt securities |
| **CCDC / CSDC** | China | Bonds (CCDC) / equities (CSDC) |
| **Austraclear** | Australia | ASX-owned CSD/SSS |
| **SIX SIS** | Switzerland | Swiss CSD |

**CSDs also connect to each other**: cross-border settlement happens via CSD links and through the ICSDs (Euroclear Bank, Clearstream Lux) which hold securities for each other and for national CSDs. This "CSD of CSDs" web is how a eurobond issued in one market settles in another.

### 2.3 Securities Settlement Systems (SSSs)

An SSS settles securities transactions — the mechanical process of exchanging securities and cash between buyer and seller, legally structured as **delivery versus payment (DVP)** so that delivery happens if and only if payment happens (see [Section 4.2](#42-settlement-risk-herstatt-dvp-pvp)). In many markets the SSS is embedded in the CSD (DTC settles inside DTCC; CREST is Euroclear's UK SSS; CDP settles SGX trades). In the euro area, **T2S (TARGET2-Securities)** is the landmark standalone SSS: a single Eurosystem platform (live June 2015, fully migrated by 2017) that settles euro-denominated securities in central bank money for ~20+ CSDs, harmonizing settlement cycles and DVP across the eurozone.

**The settlement process:**

1. **Trade capture & matching** — buyer and seller (or their custodians) agree on the trade details; unmatched/affirmed late trades are the #1 cause of fails.
2. **Pre-settlement** — position checks, availability of securities (the seller must have them), cash availability, standing settlement instructions (SSIs) applied.
3. **Settlement** — securities and cash move simultaneously (DVP) in the SSS/CSD books; in central bank money (via the RTGS or the SSS's central bank account) or commercial bank money (via settlement banks).
4. **Finality** — settlement becomes irrevocable and unconditional (PFMI Principle 8). Once final, the transfer cannot be unwound except by a new transaction.

**Settlement cycles.** The industry standard moved from T+3 to **T+2** (2014-2017 wave), and then:

- **US, Canada: T+1 since 28 May 2024** (SEC adopted to reduce credit and liquidity risk in the chain).
- **India: T+1 since January 2023** (phased), with optional T+0 for select liquid stocks.
- **EU + UK + Switzerland: T+1 targeted for 11 October 2027** (proposed via a CSDR amendment; ESMA and the European Commission aligned on this date) — the next big post-trade change for European banks.
- **T+0 / instant settlement** for some instruments (money-market funds, tokenized securities — see [Section 7](#7-the-modern-evolution-of-financial-infrastructure)).

**Settlement fails and buy-ins.** When a seller cannot deliver (or a buyer cannot pay) on settlement day, the trade **fails**. Fails create liquidity and credit risk (the buyer has paid exposure without receiving securities). Mitigations: penalty regimes (CSDR cash penalties in the EU since Feb 2022; DTCC/NSCC charges in the US), and **buy-ins** — a mechanism where the buyer (or the market, via a CCP or exchange) purchases the securities in the open market after a grace period, with any extra cost borne by the failing seller (CSDR's mandatory buy-in framework; SEC/DTCC close-out rules). T+1 compressed the time available to fix fails, making pre-settlement automation (matching, allocation, standing instructions) critical.

### 2.4 Central Counterparties (CCPs)

A CCP is an FMI that **interposes itself between the buyer and seller of a trade**: after a trade is executed, the CCP becomes the buyer to every seller and the seller to every buyer. This is **novation** — the original contract is legally replaced by two contracts with the CCP. The CCP's job is to guarantee performance even if one side defaults.

**Why CCPs exist:** before centralized clearing, a dealer default could cascade through the bilateral web of exposures (the "domino" problem). By concentrating counterparty risk in one place with robust risk management, CCPs *reduce* systemic risk — provided they are themselves well-managed (hence PFMI Principles 4-7, 13-14).

**The CCP risk machinery:**

- **Netting** — the CCP nets all positions per member per product: multilateral netting collapses thousands of gross obligations into a handful of net positions, cutting settlement flows and credit exposure by orders of magnitude.
- **Margin** — the two-tier collateral engine:
  - **Initial margin (IM)** — collateral posted against potential future exposure over a margin period of risk (typically 2-5 days for derivatives); calculated per portfolio (the CCP's internal model or ISDA SIMM for uncleared — see `nasdaq_calypso_guide.md`).
  - **Variation margin (VM)** — daily (sometimes intraday) mark-to-market cash flows: losers pay, winners receive; keeps the CCP's exposure near zero.
  - **Default fund (guarantee fund)** — mutualized contributions from all members, sized to cover the default of the largest member(s) under stress.
- **Default management** — if a member defaults: the CCP closes out/hedges the defaulting member's portfolio, applies the **loss waterfall**: (1) defaulting member's IM → (2) defaulting member's default fund contribution → (3) CCP "skin in the game" (its own capital) → (4) non-defaulting members' default fund contributions → (5) assessment rights (additional calls on survivors) → (6) last-resort tools (variation margin gains haircutting, forced allocation).
- **Segregation and portability** (PFMI P14) — client positions and collateral are segregated from the clearing member's own (and, where chosen, from other clients: "LSOC" — legal segregation with operational commingling in the US; full segregation elsewhere), so a clearing member's failure does not trap client assets, and client positions can be ported to a surviving member.
- **Liquidity** — a CCP must be able to pay VM to winners even if the defaulting member's cash is gone: committed liquidity lines from banks and central bank access are essential (PFMI P7).

**The clearing lifecycle (day in the life of a cleared trade):**

1. **Trade** — executed on a venue or bilaterally; submitted to the CCP (directly or via a clearing member).
2. **Novation** — the CCP becomes counterparty to both sides; the original contract is replaced by two CCP contracts.
3. **Position keeping & netting** — the CCP nets positions per member per product; margin is calculated on the net portfolio.
4. **Initial margin call** — IM (and default fund contributions) must be posted before or during the trade's life; IM is re-calculated (typically daily, sometimes intraday) as positions move.
5. **Variation margin** — daily mark-to-market cash flows settle the P&L; the CCP's exposure to each member returns to ~zero each day.
6. **Settlement** — physical (securities delivered via the CSD, DVP) or cash-settled (index/futures final settlement).
7. **Default (if it happens)** — close-out, waterfall application, portability of client positions, and restoration of a matched book.

**Major CCPs:**

| CCP | Owner | Products |
|---|---|---|
| **LCH Ltd / LCH SA** | LSEG | Rates (SwapClear — the world's largest rates CCP), CDS, FX, repo; LCH SA for euro clearing (post-Brexit) |
| **CME Clearing** | CME Group | Futures/options (rates, equity, FX, commodities), now also cleared OTC rates/swaps |
| **ICE Clear Credit / ICE Clear Europe / ICE Clear US** | ICE | Credit default swaps (CDS), energy, rates |
| **Eurex Clearing** | Deutsche Börse | European rates, equities, listed derivatives; growing OTC rates franchise |
| **SGX-DC** (SGX Derivatives Clearing) | SGX | Asian derivatives — equity index futures/options, FX, commodities (incl. iron ore, freight) |
| **JSCC** | Japan | Equity, bonds, derivatives |
| **OCC** | US options | Listed options and security futures |
| **NSCC / FICC** | DTCC | US equities (NSCC), US treasuries and repos (FICC-GSD), MBS (FICC-MBS) |
| **CCIL, ASX Clear, HKEX Clearing, SIX x-clear, CCDC/SHCH** | various | Regional cash and derivatives clearing |

**The clearing mandate.** Post-2008, the G20 (Pittsburgh 2009) required that all **standardized OTC derivatives be cleared through CCPs** and reported to trade repositories. This became law as **Dodd-Frank Title VII** (US, 2010) and **EMIR** (EU, 2012). Uncleared derivatives face penal capital charges and the **uncleared margin rules (UMR)**: phased-in initial margin (using **ISDA SIMM**) and variation margin for the largest counterparties (phases ran 2016-2020, with later phases through September 2022 for smaller firms). UMR forced banks to build collateral management and SIMM infrastructure — see `nasdaq_calypso_guide.md`.

### 2.5 Trade Repositories (TRs)

A trade repository is an FMI that **maintains a centralized electronic record (database) of transactions** — the legal requirement born from the G20's "no data without reporting" principle: if a market is opaque, regulators cannot see risk building. TRs are the data layer of OTC derivatives:

- **What is recorded:** the full terms of each derivative transaction (counterparties, product, notional, maturity, price, collateral), updated through the trade's life (valuations, margin, events, termination).
- **Who reports:** banks and dealers (and, under EMIR Refit, non-financial counterparties above thresholds) report both cleared and uncleared trades.
- **Who reads:** regulators — TRs give supervisors a market-wide view to detect concentration, leverage, and interconnectedness. Under **EMIR**, EU authorities (ESMA) access TR data; under **Dodd-Frank**, the CFTC; under **MAS Notice 677**, MAS (Singapore).
- **Major TRs:** DTCC Data Repository (global derivatives, the largest), CME Group Trade Repository, ICE Trade Vault, plus regional/licensed TRs such as DTCC Data Repository (Singapore) Pte Ltd — MAS-licensed for OTC derivatives reporting in Singapore (see [Section 11](#11-the-singapore-context)).

TRs are the least "risky" FMI type operationally (they hold data, not money or securities) but the most important for **regulatory transparency** — and the data quality problems (duplicate reports, LEI mismatches, reconciliation breaks) are a chronic industry issue. EMIR Refit (2024) and the CFTC's rewrites aimed at fixing exactly that: fewer fields, more structured data, JSON/ISO 20022-style formats, and mandatory reconciliation.

---

## 3. The PFMI Framework: Principles, Responsibilities, Systemic Importance

The **Principles for Financial Market Infrastructures** (PFMI) — published April 2012 by the **CPMI** (Committee on Payments and Market Infrastructures, BIS, formerly CPSS) and **IOSCO** (International Organization of Securities Commissions) — is the global standard for FMIs. It replaced the 2001 Core Principles for Systemically Important Payment Systems, the 2001 Recommendations for Securities Settlement Systems, and the 2004 Recommendations for Central Counterparties, unifying them after the 2008 crisis. The PFMI was amended in 2016-2017 (cyber resilience annex, and clarification of "financial market infrastructure" definitions) and is periodically reviewed (a review cycle ran 2023-2024).

### 3.1 The 24 Principles

The principles are grouped into five categories. Every FMI type applies a subset — payment systems apply P1-P10; CSDs add P11; CCPs add P4-P7 and P13-P14; TRs apply P1-P3, P17, P20-P24.

| Cat. | # | Principle | What it requires |
|---|---|---|---|
| **General organization** | 1 | **Legal basis** | The FMI's rules, procedures, and contracts are enforceable in all relevant jurisdictions |
| | 2 | **Governance** | Clear governance arrangements promoting safety, efficiency, and the interests of participants and the public |
| | 3 | **Framework for comprehensive risk management** | A framework to manage legal, credit, liquidity, operational, general business, custody, investment, and other risks |
| **Credit & liquidity risk** | 4 | **Credit risk** | Measures to limit and manage credit exposures to participants (settlement and pre-settlement), incl. limits and collateralization |
| | 5 | **Collateral** | Accept only high-quality collateral with low credit/liquidity risk; manage concentration and apply haircuts |
| | 6 | **Margin** | For CCPs: margin covers current and potential future exposure across products; calibrated to stressed 2+ day MPOR |
| | 7 | **Liquidity risk** | Maintain sufficient liquid resources to settle same-day obligations under the default of the largest participant(s) |
| **Settlement** | 8 | **Settlement finality** | Clear and certain final settlement, at minimum by end of value date; finality protected by law |
| | 9 | **Money settlements** | Settle the cash leg in central bank money where practical; otherwise in assets with negligible credit/liquidity risk |
| | 10 | **Physical deliveries** | Clear and certain physical delivery, with disclosure of delivery obligations and risks |
| **CSD & exchange of value** | 11 | **Central securities depositories** | Integrity of securities issues, no double-creation, safeguarding of securities, accurate records, protection of participant assets |
| | 12 | **Exchange-of-value settlement systems** | Eliminate principal risk for DVP/PVP transactions (settle both legs together) |
| | 13 | **Participant-default rules** | Effective, clearly disclosed rules and procedures for managing a participant default |
| | 14 | **Segregation and portability** | Enable segregation of customer positions/collateral and their portability to a solvent participant |
| | 15 | **General business risk** | Hold sufficient liquid net assets funded by equity (e.g., CCP capital for 12 months of operating costs) |
| | 16 | **Custody and investment risks** | Safeguard participants' assets; invest only in low-risk, liquid instruments |
| | 17 | **Operational risk** | Robust operational and cyber risk management: identify, monitor, manage; business continuity and disaster recovery with defined recovery time objectives (systemically important FMIs: same-day resumption, 2-hour recovery for critical markets) |
| **Access & transparency** | 18 | **Access and participation** | Objective, risk-based, publicly disclosed participation requirements |
| | 19 | **Tiered participation** | Understand and manage the risks of indirect participants (correspondents, clients of members) |
| | 20 | **FMI links** | Assess and manage risks arising from links to other FMIs |
| | 21 | **Efficiency and effectiveness** | Meet participant and market needs while promoting efficiency |
| | 22 | **Communication procedures and standards** | Use internationally accepted communication procedures and standards (ISO 20022 etc.) |
| | 23 | **Disclosure of rules and market data** | Publish rules, procedures, fees, and price data |
| | 24 | **Disclosure of market data by trade repositories** | Provide timely, accurate data to regulators and the public in line with policy |

### 3.2 The five responsibilities of authorities

The PFMI also sets out **five responsibilities** for central banks, regulators, and other authorities (collectively "authorities"):

- **Responsibility A:** Regulate, supervise, and oversee FMIs within their mandate.
- **Responsibility B:** Have (and use) appropriate powers and resources to do so.
- **Responsibility C:** Publicly disclose their policies for FMI regulation/oversight.
- **Responsibility D:** Apply the Principles consistently across FMI types (domestically and, where relevant, internationally).
- **Responsibility E:** Cooperate with other authorities (cross-border, cross-sector) — e.g., through memoranda of understanding and crisis coordination.

### 3.3 How the PFMI is applied and assessed

Compliance is not a checkbox: authorities assess each FMI against the applicable principles through a structured process:

- **Self-assessment + regulatory assessment** — the FMI publishes a detailed disclosure framework (its rules, risk management, and how each principle is met); the authority independently verifies, often with on-site inspections and stress-test reviews. CPMI-IOSCO publishes the assessments for systemically important FMIs.
- **Jurisdictional implementation** — each authority embeds the PFMI in its own regime: the Fed's Policy on Payment System Risk (US), the ECB's oversight framework for T2/T2S/TIPS, the Bank of England's FMIs supervision, MAS' oversight of MEPS+/FAST/CDP/SGX-DC.
- **Recovery and resolution planning** — since 2014 (FSB Key Attributes + CPMI-IOSCO guidance), systemically important FMIs must have recovery plans (loss/liquidity allocation tools) and authorities must hold resolution powers — the FMI equivalent of "living wills."
- **The assessment cycle** — the PFMI itself is periodically reviewed (the 2023-2024 review focused on cyber resilience, crisis management, and adaptation to new technologies), and FMIs are re-assessed after material changes (new products, new links, new technology).

### 3.4 FMIs as systemically important: "too important to fail"

FMIs concentrate risk by design — that is their function. The 2008 crisis showed both sides: Lehman's default was absorbed by CCPs and settlement systems (which held up), while the bilateral OTC derivatives web nearly froze the system (which is why the G20 mandated clearing). Since 2012:

- **Designation:** Jurisdictions formally designate systemically important FMIs. In the US, **Dodd-Frank Title VIII** lets the FSOC designate financial market utilities (FMUs) — CHIPS, CLS, DTC, NSCC, FICC, CME, ICE Clear Credit are designated; designated FMUs gain Federal Reserve access (accounts, discount window, payment services).
- **Recovery and resolution:** Because FMIs cannot fail the way banks do, the **FSB Key Attributes (2014)** and CPMI-IOSCO guidance (2014, 2017) require FMIs to have **recovery** plans (loss- and liquidity-allocation tools: assessments, haircuts, tear-ups, partial settlement) and authorities to have **resolution** powers over them.
- **Systemic risk channels:** an FMI failure propagates via (1) settlement gridlock (RTGS down ⇒ all downstream settlement stops), (2) CCP default ⇒ member losses and collateral calls, (3) CSD failure ⇒ ownership records lost/contested, (4) data loss (TR) ⇒ regulator blindness.

---

## 4. FMI Risk Management

### 4.1 The Risk Layers

**Credit risk — "a participant defaults."** An FMI's core risk: a member fails to pay or deliver. Each FMI type manages it differently:

- RTGS: no credit risk by design (gross settlement, prefunded or collateralized intraday credit) — but the *intraday* credit extended by the central bank must be collateralized.
- SSS/CSD: managed via DVP (both legs move together) and position limits.
- CCP: the full machinery — IM + VM + default fund + waterfall (see [Section 2.4](#24-central-counterparties-ccps)). The **loss waterfall** is the CCP's answer to "who pays if the defaulting member's margin is insufficient": defaulting member's IM → its default fund contribution → CCP skin-in-the-game → survivors' default fund → assessments. Stress tests calibrate the default fund to the largest member (Cover 2 in Europe: the two largest members' defaults).
- Settlement systems: principal risk (see 4.2) eliminated by DVP/PVP; replacement-cost risk remains (a trade's market move between execution and settlement).

*Credit risk by FMI type:*

| FMI type | Credit exposure | Key mitigants |
|---|---|---|
| RTGS (PS) | Minimal by design — gross settlement, prefunded/collateralized intraday credit | Collateralized intraday overdrafts, queuing, liquidity-saving mechanisms |
| Netting PS (CHIPS, ACH) | Settlement risk within the day; participant failure before net payout | Netting caps, collateral, loss-sharing rules, prefunding |
| SSS/CSD | Principal risk eliminated by DVP; replacement cost until settlement | DVP (Model 1/2/3), position limits, buy-ins |
| CCP | Default of member(s): IM insufficient under stress | IM + VM + default fund + waterfall + assessments + stress tests (Cover 2) |
| TR | None material (data only) | Data quality, access controls |

**Liquidity risk — "settlement liquidity."** FMIs need money *when* it is needed, not just *eventually*. RTGS participants need **intraday liquidity** — hence central banks provide intraday credit facilities (Fedwire overdrafts, T2 marginal lending, MEPS+ intraday facilities), and banks queue/optimize payments (liquidity-saving mechanisms, payment queueing, netting offsets) to recycle the same cash several times a day. The intraday liquidity problem is the *sequencing* problem: a bank must pay its obligations (settle its trades, fund its CLS positions, pay its customers' transfers) before its own credits arrive; an intraday liquidity shortfall produces failed settlements and penalty regimes, and in a stress scenario, gridlock. Regulators therefore require banks to report intraday liquidity (Basel III's intraday liquidity monitoring tools; the FSB's guidance) and FMIs to measure participants' liquidity demands. CCPs must be able to pay VM to winners immediately after a default: committed bank lines and, ideally, central bank accounts. The **liquidity crisis scenario** is the synchronized default of multiple members in a stress event — the default fund sizes and liquidity lines are calibrated against it.

**Operational risk — uptime, processing, and people.** FMIs target extreme availability (five nines; see [Section 8](#8-the-fmi-technology-stack)) and must recover within defined times under PFMI P17. Operational risk also includes processing errors (wrong settlement instructions, duplicate payments) and insider/social-engineering threats.

**Cyber risk — the top operational threat.** FMIs are high-value targets because a single compromise can move real money or corrupt the record of ownership. The **FSB's cyber resilience framework** (2018) and the **PFMI cyber annex** (2016) require FMIs to: identify critical functions, protect, detect intrusions, respond, recover — and to test (intelligence-led red-team testing, "frequent testing, rarely failing"). The defining incident is the **2016 Bangladesh heist** (Section 4.3).

**Legal risk.** Enforceability of netting and finality across jurisdictions (PFMI P1): a bankruptcy court must respect the FMI's netting and finality rules, or settlement unwinds. This is why jurisdictions pass special legislation (e.g., the EU's Settlement Finality Directive; the US Bankruptcy Code's safe harbors for financial contracts; Singapore's Payment Services Act and Securities and Futures Act).

**Custody and investment risk.** The CSD/CCP holds participants' assets (securities, collateral); those assets must be legally protected (PFMI P16), segregated from the FMI's own assets, and invested only in safe, liquid instruments. The 2008 lesson (Lehman, Madoff) drove the segregation and portability requirements (P14).

### 4.2 Settlement Risk: Herstatt, DVP, PVP

**Herstatt risk** — named after **Bankhaus Herstatt** (Cologne, Germany), which collapsed on **26 June 1974**. Herstatt's counterparties in New York had paid dollars to Herstatt (via CHIPS) for Deutsche marks that were due later the same day in Frankfurt; when the German regulators closed the bank after the dollar leg paid but before the mark leg settled, the counterparties lost the full dollar amount. This is **principal risk** in FX settlement: one leg of a cross-currency deal settles (and becomes final) in one time zone while the other leg is still outstanding hours later.

The industry's answer is **Continuous Linked Settlement (CLS)** — the FX settlement FMI:

- Founded 1997 by a consortium of major banks (following the 1996 G10 "Allsopp" report on settling FX), live **September 2002**.
- **Payment versus payment (PVP):** CLS settles both legs of an FX trade simultaneously in its own books, using central bank money held in CLS accounts at each issuing central bank. Both legs settle, or neither does — eliminating principal risk for the covered currencies (~18 major currencies).
- CLS is itself a designated systemically important FMI (US, UK oversight) and a PFMI-compliant payment system. It also nets: members' gross obligations are netted per currency per day, cutting settlement flows by ~95%.
- The **PVP principle generalizes**: any two-sided settlement where both legs must move together. For securities, the analog is **DVP**.

**Delivery versus payment (DVP)** for securities: the mechanism in SSSs where securities are delivered only against payment, and payment only against delivery, so that neither side faces principal risk. The BIS's 1992 report defined three models:

- **DVP Model 1:** gross, real-time, trade-by-trade settlement of both legs (e.g., T2S, CREST, Fedwire-style links).
- **DVP Model 2:** securities gross, cash netted at end of day.
- **DVP Model 3:** both legs netted, settled end of day (e.g., NSCC's continuous net settlement).

DVP runs in **central bank money** (best: T2S, CREST via Bank of England, DTC via the Fed) or **commercial bank money** (settlement banks guarantee the cash leg). PFMI P9 prefers central bank money; P12 requires principal risk to be eliminated in exchange-of-value systems.

### 4.3 Cyber Risk: The SWIFT Bangladesh Case

**The 2016 Bangladesh Bank heist** is the canonical FMI cyber case study:

- **What happened (February 2016):** attackers compromised Bangladesh Bank's SWIFT connectivity (credentials stolen, malware — a variant of the "Evinced"/"BANKSWIFT" family — deployed on the bank's SWIFT Alliance servers). They sent **fraudulent SWIFT MT202 messages** to the New York Fed, instructing transfers of ~$951 million from Bangladesh Bank's Fed account to accounts in the Philippines, Sri Lanka, and elsewhere. **$81 million** was actually moved (the rest was blocked by a typo — "Shalika Foundation" instead of "Shalika" — and by luck); most was laundered through the Philippines casino system.
- **Why it succeeded:** weak endpoint security (no segregation of the SWIFT environment), compromised credentials, and the absence of independent verification of payment instructions outside the SWIFT path. The legitimate-looking messages passed because the payments were *authentic* (came from the real SWIFT keys) — the fraud was at the *instruction* layer, not the network layer.
- **The fallout:** SWIFT launched its **Customer Security Programme (CSP)** — mandatory KYC-SA (Security Attestation), the Customer Security Controls Framework (CSCF, 28 mandatory + optional controls), and third-party assurance (SWIFT CSP assessments). Banks worldwide had to harden their SWIFT environments: separate infrastructure, dual control, anomaly detection on outgoing messages, and reconciliation of nostro accounts. The heist also catalysed the industry's move toward **payment validation/confirmation services** (e.g., SWIFT gpi's confirmation-of-credit, and the push for ISO 20022's richer, more traceable data).
- **Lesson for architects:** the interbank messaging layer is part of the attack surface; controls belong *around* it (segmentation, monitoring, confirmation-of-payee, anomaly detection), not only inside it. See `container_certificates_guide.md` for the PKI/HSM hygiene that underpins this.

*The pattern since 2016:* SWIFT disclosed a series of related intrusions (banks in Bangladesh, Vietnam, Ecuador, Taiwan, India — the "BANKSWIFT" malware family and copycats), all sharing the same anatomy: compromised operator credentials or infrastructure, forged/authentic-looking payment instructions, and suppression of confirmations. The defense-in-depth canon that emerged: **segregate** the SWIFT environment (separate from the general network), **verify** outbound messages independently (reconciliation against an independent log; "confirmation of payee" style checks), **restrict** human access (dual control, four-eyes on high-value payments), **monitor** for anomalies (unusual message patterns, new beneficiaries, changed SSIs), and **test** continuously (the SWIFT CSP's required annual attestation and independent assurance). For the architect, the Bangladesh case is the canonical example of why the *control plane* (who can instruct money movement) is as important as the *data plane* (the messages themselves).

---

## 5. The Market Technology Infrastructure

This is the broader layer — what practitioners mean by "market infrastructure" beyond the formal FMI taxonomy: the venues, data, and networks on which trading and post-trade operations run.

### 5.1 Exchanges and Trading Venues

- **Primary exchanges** — the regulated venues that list and trade securities and derivatives: NYSE, Nasdaq, LSE, Deutsche Börse, Euronext, SGX, HKEX, JPX, ASX, CME, ICE Futures, Eurex. Exchanges are *not* FMIs in the PFMI taxonomy for their trading function (their clearing/settlement arms are), but they are core market infrastructure: they run matching engines, publish prices, and enforce market rules and surveillance.
- **Alternative venues** — ATS/MTFs (alternative trading systems in the US, multilateral trading facilities in Europe): dark pools (no pre-trade transparency — e.g., Liquidnet, internalization engines), systematic internalizers, and broker crossing networks. Fragmentation of liquidity across venues drove the need for consolidated market data (Section 5.2).
- **The regulatory frame for venues:** in the US, **Regulation NMS** (2005) mandates best execution across venues and the consolidated tape (SIP); in Europe, **MiFID II** (2018) imposes pre/post-trade transparency, venue categorisation (RM/MTF/OTF), and transaction reporting to regulators. Both regimes turned venue fragmentation into a data problem (order-routing logic, smart order routers, market data consolidation) — the architect's venue layer is as much about data and routing as about connectivity.
- **Trading architecture** — matching engines, order books, market-by-price/market-by-order feeds, and the low-latency race (co-location, kernel-bypass networking, FPGA) are covered in depth in `low_latency_cpp_development_guide.md`. The trading → clearing → settlement flow is: venue match → trade report → CCP (novation + netting) → CSD/SSS (DVP settlement).
- **Repo and money markets** — the funding side of market infrastructure: electronic repo platforms (e.g., Eurex Repo, GC Pooling, BrokerTec), tri-party repo (FICC-sponsored, Euroclear/Clearstream tri-party), and money-market trading; see `asset_backed_trading_guide.md` for the collateral/payments context.

### 5.2 Market Data Infrastructure

- **Consolidated feeds:** the official combined price stream per market — the US **SIP** (Securities Information Processor, run by the exchanges under Regulation NMS), the EU's **consolidated tape** (equities from 2025, bonds under the CMU agenda). Consolidated feeds give the *reference* price; they are slower than direct feeds.
- **Vendor feeds:** **Bloomberg** (the desktop/financial-information standard), **LSEG/Refinitiv** (formerly Thomson Reuters), FactSet, Morningstar, Moody's/S&P — the distribution layer that packages exchange data, contributed data (quotes from dealers), news, and analytics into terminal and enterprise feeds.
- **Direct exchange feeds:** raw, low-latency feeds from the venue itself — Nasdaq ITCH/OUCH, CME MDP, Eurex ECS — consumed via co-location and high-performance messaging (FAST/FIX-encoded; see `low_latency_cpp_development_guide.md` for the engineering). These are the "real" prices for algorithmic traders; the latency hierarchy is direct feed < consolidated feed < vendor redistribution.
- **Market data management (MDM):** the enterprise layer that governs entitlements (who may see what — exchange licensing is per-user and per-use), captures and normalizes feeds, manages the cost (market data is a top-5 bank cost), and redistributes internally with full audit (a legal/licensing obligation). MDM systems (e.g., Thomson Reuters RMDS, Bloomberg's Data License, Activ Financial) are the "data bus" of a trading floor. Two details architects meet constantly: **entitlement** (exchange vendors audit every redistribution — "no redistribution without a license" is enforced with usage logs and, increasingly, automated metering) and **latency hierarchy** (direct exchange feed < consolidated feed < vendor redistribution — each hop adds microseconds that matter for algos; see `low_latency_cpp_development_guide.md`).
- **Reference price & analytics:** index providers (S&P DJI, MSCI, FTSE Russell, Nareit), benchmark administrators (now regulated under EU BMR), and pricing/valuation services (e.g., Markit, ICE Data) that price illiquid instruments.

### 5.3 Reference Data

Reference data is the *context* that makes transactions meaningful — the "golden copy" problem: every bank, CSD, and TR needs the same unambiguous description of an instrument, a counterparty, or a location, but each maintains its own copy, and copies drift. The infrastructure answer is **standards + utilities**:

- **Instrument symbology:** ISIN (ISO 6166 — the international securities identifier, assigned by national numbering agencies, e.g., S&P for the US), CUSIP (US/Canada), SEDOL (UK), FIGI (Bloomberg's open Financial Instrument Global Identifier), plus exchange tickers. Mapping between symbologies is a reference-data service in itself.
- **Counterparty/entity data:** the **Legal Entity Identifier (LEI)** — ISO 17442, a 20-character code identifying legal entities in financial transactions, issued by GLEIF-accredited local operating units (the LEI system went live 2014 under FSB endorsement). LEIs are mandatory in MiFID II/EMIR reporting, MAS trade reporting, and are being embedded into ISO 20022 messages.
- **Standing settlement instructions (SSIs),** account data, tax classification (e.g., FATCA/CRS), sanctions and PEP lists — the operational reference data that payment and settlement systems consume.
- **Message standards as reference data:** ISO 15022 (securities messaging, the predecessor of ISO 20022 in the securities space) and ISO 20022 define the fields; the *values* (identifiers, codes, dates) come from reference data. See `data_models_banking_insurance_guide.md` for the data-modeling view.

*The golden-copy problem in practice:* instrument masters drift across systems unless governed as a single logical dataset — the classic architecture is a **golden copy** (one authoritative record per instrument/counterparty) fed by utility data (ANNA/ISO numbering agencies for ISINs, GLEIF for LEIs, the exchange for symbology, the CSD for corporate actions), with downstream systems consuming governed subscriptions rather than maintaining local copies. Reference data failures are a top cause of settlement fails (wrong SSI), reporting errors (bad LEI), and valuation disputes (wrong instrument attributes) — which is why reference data governance sits inside the FMI risk story: bad data *is* operational risk.

### 5.4 Messaging Networks

- **SWIFT** — the interbank messaging cooperative (founded 1973, Brussels; owned by ~11,000 member institutions): operates SWIFTNet, the FIN messaging service (MT messages: MT103 customer transfer, MT202 interbank transfer, MT5xx securities, MT9xx statements), the ISO 20022-based MX messages (pacs/camt/pain), **SWIFT gpi** (Global Payments Innovation — tracking, fee transparency, same-day credit for cross-border payments, live 2017), and the **Customer Security Programme**. SWIFT is a *messaging* network, not a settlement system: settlement happens in the RTGS systems and correspondent accounts the messages trigger. See `payments_hub_guide.md` for the payment-hub view (MT/MX coexistence, CBPR+, UETR).
*SWIFT message families (the MT vs MX view):*

| Family | Purpose | Examples |
|---|---|---|
| **MT1xx** | Customer payments | MT103 (customer transfer), MT101 (request for transfer) |
| **MT2xx** | Interbank/financial transfers | MT202 (bank transfer), MT202 COV (cover — the instrument used in the Bangladesh heist) |
| **MT3xx** | FX / money market | MT300 (FX confirmation), MT320 (loan/deposit) |
| **MT4xx** | Collections | MT400/410 |
| **MT5xx** | Securities | MT502 (order), MT514/515 (trade confirmations), MT54x (settlement instructions) |
| **MT6xx** | Treasury / derivatives | MT600/620 (commodity/interest-rate confirmations) |
| **MT7xx** | Documentary credits | MT700 (letter of credit) |
| **MT9xx** | Statements / cash management | MT900/910 (debit/credit advices), MT950 (statement) |
| **MX (ISO 20022)** | The successor generation | pacs.008 (FIToFICustomerCreditTransfer), pacs.009 (cover), camt.05x (statements), pain.001 (initiation), reda/acmt/admi |

- **FIX** — the Financial Information eXchange protocol (1992, Fidelity/Salomon; now the FIX Trading Community): the *de facto* standard for pre-trade and trade messaging (orders, executions, indications) between buy side, sell side, and venues. FIX 4.2/4.4 dominate; FIX 5.0+ and FIXatdl for algo specs; FAST for compressed market data. The trading complement to SWIFT's post-trade role.
- **FpML** — Financial products Markup Language (1999, JPMorgan-initiated; ISDA-maintained): XML for OTC derivatives — trade confirmation, valuation, collateral terms — the *product description* standard that feeds CCPs, TRs, and valuation engines.
- **ISO 20022** — the universal financial messaging methodology (XML/JSON message catalogue): the *global payments standard*, adopted by SWIFT (CBPR+), T2/T2S/TIPS, Fedwire/CHIPS (2025 migration), MEPS+, SEPA, and instant schemes worldwide. ISO 20022 is the convergence point: one standard for payments, securities, cards, and trade services — the "single language" of financial infrastructure (see `payments_hub_guide.md`).

### 5.5 Connectivity

How participants physically reach the infrastructure:

- **Direct market access (DMA)** — a client sends orders directly to a venue using the venue's own protocols (via a broker's sponsored access or its own membership), skipping the broker's order desk; low latency, but the client carries execution risk (hence broker risk controls and kill switches).
- **Co-location** — placing servers inside the exchange's data center (NYSE Mahwah, Nasdaq Carteret, CME Aurora, LSE Basildon, Eurex Frankfurt) to shave microseconds off the feed and order path; the arms race is detailed in `low_latency_cpp_development_guide.md`.
- **Extranets / community networks** — the legacy financial VPNs: **BT Radianz** (the former Reuters/Equant network, now part of BT, connecting thousands of venues/vendors), **SIX Exfeed** (Swiss market data network), Verizon/Orange financial extranets, and the SWIFT network itself. These provide deterministic, low-latency, high-security connectivity between banks, venues, and vendors.
- **Cloud connectivity** — the new layer: banks connect on-premise cores to cloud-based analytics and data via **private interconnects** (AWS Direct Connect, Azure ExpressRoute, GCP Interconnect) and cloud market data (exchange feeds delivered into the cloud). Regulatory and latency constraints make hybrid the norm: low-latency trading stays on-prem/co-lo; analytics, reporting, and DR move to cloud. See the cloud and network guides in this repository for the platform view.

---

## 6. The Global Financial Infrastructure Landscape

### 6.1 United States

| Layer | System | Operator / notes |
|---|---|---|
| Large-value payments | **Fedwire Funds** | NY Fed RTGS; ~$4T/day; 21.5h business days; intraday overdrafts collateralized |
| Large-value payments (private) | **CHIPS** | The Clearing House; continuous net settlement, final payout over Fedwire; ~$1.8T/day |
| Instant payments | **FedNow** (2023), **RTP** (2017) | Fed vs The Clearing House — the US's two instant rails |
| ACH | **NACHA network** | Batch net settlement via FedACH and EPN; Same-Day ACH windows |
| CSD / SSS | **DTC / DTCC** | World's largest CSD; settles US equities, corporate/municipal debt, money-market instruments; DVP in central bank money via the Fed |
| Clearing (CCPs) | **NSCC, FICC** (DTCC); **CME Clearing; ICE Clear; OCC** | Equities (NSCC — the largest CCP by volume), US treasuries/repos (FICC), listed derivatives (CME/ICE), options (OCC) |
| Trade repositories | **DTCC Data Repository, CME TR, ICE Trade Vault** | CFTC/Dodd-Frank reporting |
| Exchanges | NYSE, Nasdaq, Cboe, CME, ICE, CBOE | SIP consolidated tape; Reg NMS fragmentation |
| Oversight | Federal Reserve, CFTC, SEC, FSOC (Title VIII FMU designation) | PFMI assessment via the Fed's Policy on Payment System Risk (PSR) |

US-specific notes: the Fed's **Policy on Payment System Risk** implements PFMI for US FMIs; **Dodd-Frank Title VII** (2010) mandated OTC clearing and reporting; **T+1** since 28 May 2024.

### 6.2 Europe and the UK

| Layer | System | Operator / notes |
|---|---|---|
| Large-value payments | **T2** (euro, live March 2023) | Eurosystem RTGS, successor to TARGET2; integrated liquidity management; part of the T2-T2S-CLI (consolidated liquidity management) architecture |
| Instant payments | **TIPS** (2018) | TARGET Instant Payment Settlement; the Eurosystem's instant rail (with SEPA Instant scheme) |
| Securities settlement | **T2S** (2015-2017) | TARGET2-Securities: single Eurosystem platform settling euro securities in central bank money for ~20+ CSDs |
| CSDs / ICSDs | **Euroclear, Clearstream** | Euroclear (BR ICSD + national CSDs incl. CREST), Clearstream (Frankfurt CSD + Lux ICSD) |
| CCPs | **LCH Ltd/LCH SA, Eurex Clearing, ICE Clear Europe, Euronext Clearing, SIX x-clear** | Rates (SwapClear), European equity/rates (Eurex), CDS/energy (ICE) |
| UK payments | **CHAPS** (RTGS), **Faster Payments** (instant), **Bacs** (ACH) | Bank of England; FPS operated by Pay.UK |
| UK securities | **CREST** (Euroclear UK & International) | The UK/Irish CSD and SSS |
| Trade repositories | **EU: ESMA-registered TRs (DTCC, CME, ICE, Regis-TR); UK: FCA-recognised TRs** | EMIR / UK EMIR reporting |
| Oversight | ECB (Eurosystem), ESMA, national supervisors; Bank of England (CHAPS, CREST, LCH) | PFMI assessments; the ECB's T2/T2S/TIPS oversight |

European notes: **EMIR** (2012) + **EMIR Refit** (2024) govern clearing and reporting; **CSDR** (2014, with the 2022 penalty regime and the 2027 T+1 amendment) governs settlement discipline; post-Brexit, euro clearing migrated partly to **LCH SA** and **Eurex** under ECB location policy pressure; the **Capital Markets Union (CMU)** agenda pushes consolidated tape and settlement harmonization; **T+1 on 11 October 2027** is the next milestone.

### 6.3 Asia

| Market | Payments | CSD/SSS | CCPs / exchanges | Notes |
|---|---|---|---|---|
| **China** | **CNAPS** (HVPS large-value RTGS + BEPS bulk), **IBPS** (internet banking), Alipay/WeChat Pay (retail dominance) | **CSDC** (CSDCC — equities), **CCDC** (bonds), Shanghai Clearing House (SHCH) | Shanghai/ Shenzhen exchanges, **SHCH**, China Financial Futures Exchange | e-CNY (digital yuan) pilots since 2020 — the largest retail CBDC program |
| **Japan** | **BOJ-NET** (RTGS since 2001), Zengin (retail) | **JASDEC** | **JSCC**, JPX (TSE/OSE) | BOJ-NET funds + JGB settlement; JSCC covers equities and derivatives |
| **Singapore** | **MEPS+** (RTGS), **FAST/PayNow** (instant), GIRO (ACH) | **CDP** (SGX CSD) | **SGX** + **SGX-DC** (derivatives CCP), SGX Securities Clearing | See [Section 11](#11-the-singapore-context) |
| **Hong Kong** | **CHATS** (HKMA RTGS: HKD/USD/EUR/CNH), Faster Payment System (FPS, instant, 2018) | **CMU** (Central Moneymarkets Unit — debt CSD), HKSCC (equities) | **HKEX** group, HKCC/SEOCH (derivatives/options clearing) | RMB hub; CMU is the region's bond infrastructure |
| **India** | **RTGS** (RBI, 24x7), **NEFT** (24x7), **UPI/IMPS** (NPCI) | **NSDL, CDSL** | **CCIL** (G-Secs/FX/repo), NSE Clearing, ICCL | UPI is the world's largest instant scheme; India moved to **T+1 (Jan 2023)** |
| **Australia** | **RITS** (RBA RTGS), New Payments Platform (NPP, instant) | **Austraclear** (ASX) | **ASX Clear**, ASX Clear (Futures) | CHESS replacement (DLT-based post-trade) is ASX's long-running project |

### 6.4 The Cross-Border Layer

Cross-border payments and securities flows sit on top of the domestic systems:

- **Correspondent banking** — the network of nostro/vostro accounts: Bank A holds an account with Bank B in Bank B's country; SWIFT messages instruct the movement; settlement happens in the domestic RTGS systems of each leg. This is still how most cross-border payments work (MT103 customer transfers, MT202 interbank), with charges conventions (OUR/BEN/SHA) and the de-risking problem (banks exiting correspondent relationships in high-risk corridors).
- **CLS** — the FX settlement FMI (PVP; see [Section 4.2](#42-settlement-risk-herstatt-dvp-pvp)): ~18 currencies, ~$6T/day gross handled through netting.
- **SWIFT** — the messaging backbone for correspondent banking and securities (plus gpi for tracking).
- **ICSDs and CSD links** — Euroclear Bank and Clearstream Lux move international securities between domestic CSDs (and T2S now settles many eurobond trades).
- **The G20/FSB agenda** — after the 2008 crisis the G20 rebuilt FMI standards (PFMI, CCP clearing mandate); since 2020 the **FSB cross-border payments roadmap** (19 building blocks, four themes: commitment, legal/regulatory frameworks, FMI capacity, market adoption; targets to 2027) drives cost/speed/transparency/access improvements, with ISO 20022 harmonization and instant-rail interconnection (Project Nexus — see [Section 11](#11-the-singapore-context)) as headline workstreams.
- **Geopolitical layer** — alternative messaging/settlement arrangements (China's CIPS for RMB, Russia's SPFS), sanctions enforcement embedded in SWIFT/CHIPS operations, and CBDC-based cross-border alternatives (mBridge) are reshaping the map.

---

### 6.5 Worked examples: money through the plumbing

The best way to make the taxonomy concrete is to walk real transactions through it.

**Example 1 — A cross-border payment (SGD → USD, corporate client).**

1. A Singapore corporate instructs its bank (Bank A) to pay USD 10M to a US supplier's bank (Bank B). Initiation via API/ISO 20022 `pain.001` (or an MT103-equivalent) into Bank A's payments hub.
2. Bank A has no direct USD account relationship with Bank B, so it uses its correspondent: Bank C (a US money-center bank) — the *nostro* account relationship (Bank A's USD account at Bank C).
3. Bank A sends a SWIFT MT103 (customer transfer) plus MT202 COV (cover) to Bank C over SWIFTNet. The message carries the UETR for gpi tracking.
4. Bank C debits Bank A's nostro, credits Bank B, and sends an MT103 to Bank B — settlement of the *interbank* leg happens when Bank C's books move, and ultimately Bank C's own USD position settles through **Fedwire or CHIPS** with its counterparties (its correspondent balances net out in the RTGS/netting system).
5. Bank B credits the supplier. Bank A's hub reconciles via camt.05x statements and gpi tracking events. *The domestic USD leg settled in US RTGS infrastructure; the SGD leg would have moved through MEPS+ if there had been an SGD transfer.*

**Example 2 — A securities trade (US equity, T+1).**

1. A fund manager buys 100k shares of a NYSE stock from a dealer. Order via FIX to the venue (or OTC internalized); the trade prints on the tape (consolidated SIP feed).
2. The trade is submitted to **NSCC** (DTCC's equity CCP) by both sides' clearing members: novation, netting, and margin — NSCC becomes counterparty to both.
3. Trade details are matched/affirmed (the T+1 deadline: affirm by ~9pm ET on trade date in the new cycle).
4. On T+1, **DTC** (the CSD) settles: the seller's clearing member delivers shares (book-entry transfer in DTC) against payment — **DVP** — with the cash leg settling in central bank money via the Fed (the members' Fed accounts). NSCC nets the cash across all members' trades (continuous net settlement, Model 3).
5. The fund's custodian credits its account; the dealer's fail if it cannot deliver → buy-in/penalty mechanisms apply (SEC/DTCC rules). *The same pattern runs on T2S in Europe, CREST in the UK, CDP in Singapore — DVP in central bank money through the local CSD/SSS.*

**Example 3 — An FX trade (EUR/USD spot).**

1. A bank sells EUR 100M to a client. The trade is confirmed (SWIFT MT300 or FpML/CLS confirmation).
2. If both currencies are CLS-eligible, the trade is submitted to **CLS** before the CLS cutoff: CLS nets it against the bank's other trades in each currency.
3. At settlement, CLS instructs the central banks (ECB for EUR, Fed for USD): each bank pays its net EUR obligation *into* CLS and receives USD — simultaneously, in CLS's books, using central bank money held at the ECB and the Fed. **PVP**: both legs settle or neither does. Herstatt risk is gone.
4. Non-CLS currencies settle bilaterally — with the full Herstatt risk, which is why PVP extension (more currencies, more participants) is a standing industry goal.

**Example 4 — An OTC derivatives trade (interest-rate swap).**

1. Two banks execute a 10-year EUR swap (electronically or by voice); the trade is confirmed via FpML/ISDA Confirm and captured in the trade/risk systems (see `nasdaq_calypso_guide.md`).
2. If the swap is *cleared* (standardized tenor under EMIR/Dodd-Frank): it is submitted to a CCP (LCH SwapClear or Eurex); novation happens; IM and VM are posted daily (collateralized in cash or high-quality securities); the CCP reports the trade to a **trade repository** (DTCC Data Repository, etc.).
3. If it is *uncleared*: no CCP — but UMR requires variation margin daily and initial margin (calculated with **SIMM** for the largest firms), held in segregation with a third-party custodian; the trade is still reported to a TR under EMIR (and, for Singapore counterparties, MAS Notice 677 to the MAS-licensed TR).
4. Regulators read the TR data for market-wide exposure and concentration analysis; the bank values the swap daily against market data (vendor curves), and its capital reflects the clearing status (cleared = 2% QCCP risk weight; uncleared = higher, plus CVA charges).

---

## 7. The Modern Evolution of Financial Infrastructure

The forces reshaping FMIs and market infrastructure:

**Real-time everything.** Instant payments went from novelty to default expectation: FAST, SEPA Instant, FedNow, UPI, PayNow, PIX, NPP. The operational consequence for FMIs: **24/7 settlement** (or 24x7x365 for instant rails), new liquidity models (prefunding vs intraday credit), and fraud controls designed for irrevocable instant credit (mule accounts, real-time screening, velocity checks). The open-source retail switch view is in `mojaloop_guide.md`.

**T+1 (and toward T+0).** The US/Canada moved to T+1 (28 May 2024); the EU/UK/CH target **11 October 2027**; India is at T+1 with T+0 pilots. Shorter cycles compress the time to match, allocate, and fund — fail prevention (automated matching, standing instructions, pre-funding checks) matters more than fail cure. Tokenized securities (below) push toward T+0/instant settlement for select assets.

**DLT/blockchain in FMIs.** From experiments to production pilots:

- **Project Ubin** (MAS, 2016-2020) — five phases exploring DLT for interbank payments and DVP; concluded that DLT could work for wholesale settlement but needs governance and standards.
- **Project Guardian** (MAS, 2022-) — tokenization of real-world assets (bonds, funds, FX) across banks (DBS, JPMorgan, BNY Mellon, etc.); the industry's flagship tokenized-asset pilot program.
- **Project Dunbar** (MAS + BISIH + RBA + SARB + BoT, 2022) — multi-CBDC shared-ledger settlement across jurisdictions.
- **FMI DLT pilots:** **SDX** (SIX Digital Exchange, Switzerland) — the first regulated DLT exchange+CSD (FINMA-approved, 2021; folded into SIX Swiss Exchange in 2024, signalling consolidation into mainstream venues); Euroclear's **Digital Securities Issuance** (World Bank digital bond, 2024); DTCC's **Project Ion** (DLT settlement prototype for US equities); NSD (Russia) and various central-bank experiments; **Fnality** (tokenized commercial-bank money for settlement, BoE-approved) and **JPM Coin** (wholesale payments token).
- The architectural debate: DLT replaces the *intermediary ledger* with a *shared ledger* — but FMIs' core functions (finality, netting, risk management, legal certainty) still need a system of record; the question is where the ledger lives and who can write to it. See `blockchain_technology_guide.md` for the technology.

**CBDCs.** Two tracks:

- **Retail CBDC** — digital cash for the public: **e-CNY** (PBOC, largest program), Nigeria's eNaira, the Bahamas' Sand Dollar, Jamaica's JAM-DEX; Europe (digital euro, in preparation), US (the Fed has published research — Project Hamilton with MIT — but no decision).
- **Wholesale CBDC** — central bank money for interbank settlement on tokenized rails: **Project Jasper** (Bank of Canada, 2016-17), **Project Ubin** (MAS), **Project Dunbar**, **mBridge** (BISIH + PBoC + HKMA + BoT + CBUAE, launched 2021; Saudi Central Bank joined 2024; reached minimum-viable-product stage mid-2024, moving toward production), **Project Agorá** (BISIH + seven central banks, 2024 — tokenized commercial bank money + wholesale CBDC on a unified ledger). Wholesale CBDC's promise: DVP/PVP across borders in central bank money with programmability — the "new correspondent banking."

**Open banking / embedded finance.** Payments increasingly originate outside traditional bank rails: PSD2 open banking APIs (Europe), MAS' open banking APIs (Singapore), embedded finance inside e-commerce/marketplaces. For FMIs this means the *channel layer* proliferates while the *settlement layer* consolidates — the same RTGS/instant rails underneath, more access points on top. See `core_banking_systems_guide.md`.

**AI/ML in FMIs.** Surveillance (market abuse detection on exchange data), risk analytics (CCP stress testing, margin model challengers), anomaly detection in payment flows (fraud, mule accounts, sanctioned entities), ops automation (reconciliation, exception handling, helpdesk), and predictive liquidity management. See `financial_risk_compliance_systems_guide.md`, `event_stream_processing_guide.md`, and `reinforcement_learning_algorithms_guide.md`. FMIs are conservative adopters (explainability, auditability, model risk management are hard requirements — see `nasdaq_calypso_guide.md` for the risk-system context).

---

## 8. The FMI Technology Stack

The engineering patterns behind FMIs — the same concerns that drive banking platforms, but at the extremes of availability, determinism, and auditability.

**High-availability core.** FMIs run active-active or active-passive clusters with **five-nines (99.999%) or better** uptime targets and PFMI P17 recovery objectives (same-day resumption; 2-hour recovery of critical functions; zero data loss in most designs). This means: redundant sites (same-city and remote), synchronous replication for the ledger, automated failover with no split-brain, and chaos-tested operations. See `low_latency_cpp_development_guide.md` for the real-time engineering.

**Real-time messaging.** The settlement core is event-driven: payment messages, settlement instructions, margin calls, and confirmations flow through deterministic, low-latency messaging (often proprietary or ultra-low-latency middleware — Solace, Kafka at the analytics edge, exchange-style binary protocols). Message ordering, exactly-once semantics, and idempotency are non-negotiable.

**Settlement engines (the ledger core).** The heart of an RTGS or SSS is a double-entry ledger engine that: validates (funds/positions/collateral), applies settlement logic (gross vs net; queuing and liquidity-saving algorithms), moves value atomically (both legs of DVP/PVP), and writes the audit trail. **RTGS vs netting:** gross settles each payment individually (no risk, needs liquidity); netting (CHIPS, NSCC, ACH) aggregates obligations and settles net positions (liquidity-efficient, needs risk controls and finality rules). Modern engines support both and switch by queue priority. The design invariants architects must respect: **atomicity** (a DVP settlement either fully happens or fully does not — no partial states), **finality** (once settled, irrevocable — the ledger must never "unsettle" except by a new transaction), **idempotency** (retries must not double-settle), and **determinism** (identical inputs produce identical outputs — the property that makes DLT consensus and replay testing possible; see `low_latency_cpp_development_guide.md`).

**Queueing and batch.** Real-time by day, batch by night: netting cycles, end-of-day position reconciliation, corporate actions, statements, regulatory files, and reporting runs. The batch layer (Control-M-class schedulers — see the `control_m` guides in this repo) is where "the day" is closed: finality, nostro reconciliation, and settlement-date accounting.

**Security.** HSMs (hardware security modules) for key custody and signing, PKI for message authentication (SWIFT keys, FIX certificates), TLS everywhere, and privileged-access management. See `container_certificates_guide.md` for certificate/PKI hygiene. FMIs are held to the highest bar: the SWIFT CSP controls, ISO 27001, and (increasingly) sector-specific cyber frameworks (FSB, PFMI cyber annex).

**Resilience: DR, BCP, cyber recovery.** The operational-risk principle (P17) translated into engineering: dual data centers, workload portability, cyber recovery environments (immutable backups, offline rebuild capability — "recover from a cyber event as from a fire"), and continuous testing (red-team exercises, tabletop exercises with the regulator). The 2016 Bangladesh heist and later SWIFT intrusions made "assume breach" the design posture.

**Regulatory reporting.** Every FMI and its participants report: trade reports to TRs (EMIR/Dodd-Frank/MAS Notice 677), transaction monitoring and surveillance data, liquidity reports, and PFMI disclosure frameworks. Reporting systems are first-class citizens, not afterthoughts — see `financial_risk_compliance_systems_guide.md`.

**The data layer.** FMIs are data businesses: settlement and position data (the ledger), reference data (instruments, participants, LEIs), market data (prices for margin calls and valuations), and analytics (stress tests, risk dashboards). The canonical models are covered in `data_models_banking_insurance_guide.md`; the streaming/CEP patterns in `event_stream_processing_guide.md` and `complex_event_processing_guide.md`.

```
                    FMI TECHNOLOGY STACK (conceptual)
┌──────────────────────────────────────────────────────────────┐
│  ACCESS: SWIFT / FIX / ISO 20022 / APIs │ HSM+PKI security    │
├──────────────────────────────────────────────────────────────┤
│  ORCHESTRATION: messaging bus, queues, workflow, idempotency  │
├──────────────────────────────────────────────────────────────┤
│  SETTLEMENT CORE: double-entry ledger, DVP/PVP engines,       │
│  RTGS vs netting queues, liquidity-saving mechanisms          │
├──────────────────────────────────────────────────────────────┤
│  RISK: credit limits, collateral/margin (IM/VM), default fund,│
│  liquidity monitoring, stress tests                            │
├──────────────────────────────────────────────────────────────┤
│  DATA: positions, settlement, reference data, market data,    │
│  reporting warehouse, audit trail                              │
├──────────────────────────────────────────────────────────────┤
│  PLATFORM: active-active HA, DR/BCP, cyber recovery, batch    │
└──────────────────────────────────────────────────────────────┘
```

---

## 9. The Banking Context: How Banks Sit on the Infrastructure

A bank relates to financial infrastructure in three roles at once:

**As a participant/member** — the direct, legally binding role:

- **RTGS membership** — a settlement account at the central bank (Fedwire, T2, MEPS+, CHAPS): the bank's own payments settle there; membership brings collateral requirements, intraday liquidity rules, and access to central bank money.
- **CCP clearing membership** — clearing members post IM/VM/default-fund, clear their own and clients' trades, and carry default-management obligations (assessments). Non-member banks clear through a member (agency/omnibus clearing).
- **CSD participant** — a securities account at the CSD (DTC, Euroclear, Clearstream, CDP): the bank holds its own and clients' securities there, settles DVP, and services corporate actions.
- **SWIFT membership** — BIC (bank identifier code), SWIFTNet access, FIN/MX messaging, and (post-2016) the Customer Security Programme obligations.
- **Scheme membership** — card schemes (Visa/Mastercard), ACH (NACHA), instant schemes (FAST, FedNow, SEPA Instant) — each with its own rules, fees, and risk requirements.

**As a user of the infrastructure** — every product a bank sells runs on the plumbing:

| Bank business line | Infrastructure it consumes |
|---|---|
| Payments & cash management | RTGS, ACH, instant rails, card networks, SWIFT (gpi), correspondent banking |
| Securities / custody | CSD accounts, SSS (DVP), asset servicing, corporate actions |
| Markets (rates, FX, credit, equities) | Exchanges/venues, market data, CCP clearing, CLS (FX), TR reporting |
| Repo / collateral | Tri-party agents (Euroclear/Clearstream/FICC), CCP repo clearing, RTGS cash leg |
| Derivatives | CCP clearing (rates/CDS), TR reporting, uncleared margin (UMR/SIMM) |
| Trade finance | SWIFT (MT7xx), documentary systems, bank guarantees |
| Treasury / liquidity | RTGS intraday liquidity, money markets, CLS, central bank facilities |

**The bank's FMI dependencies** (the risk side):

- **Counterparty risk to CCPs** — a CCP default (or a large member's default) hits the bank through margin calls, default-fund assessments, and position close-outs. Basel III treats qualifying CCP exposures at a preferential 2% risk weight but requires banks to manage CCP credit risk actively (and to hold capital for default fund contributions).
- **Intraday liquidity risk** — the bank must fund its settlement positions every day; a delayed RTGS credit can cascade into failed settlements (the "settlement gridlock" problem). Banks optimize with payment queueing, liquidity-saving mechanisms, and CLS-style netting.
- **Operational dependency** — the bank's own systems must interoperate with FMI release cycles (ISO 20022 migrations, T2/T2S changes, T+1), and FMI outages propagate directly into the bank's settlement positions.
- **Cyber risk** — the bank is exposed both through its own systems and through the FMI/messaging layer (the Bangladesh pattern: compromise the bank's SWIFT environment, move money from its RTGS account).
- **Concentration risk** — using a single CCP, a single CSD, a single correspondent creates concentration: regulators increasingly ask for contingency plans (alternate CCP access, multiple correspondents, CSD link fallbacks).
- **Legal/regulatory risk** — finality rules, netting enforceability, reporting obligations (EMIR/MAS Notice 677), and sanctions compliance all bind the bank to the infrastructure's legal design.

*The bank's FMI balance sheet:* viewed structurally, a bank's FMI relationships are an asset-and-liability map — **assets** are its settlement accounts (RTGS balances, CSD positions, CLS positions, collateral at CCPs), and **liabilities** are its obligations (payments queued, trades to deliver, margin to post, default-fund exposures). Treasury and risk run this map daily: intraday liquidity forecasting (when do credits arrive vs obligations fall due), collateral location (which assets sit where, and can they be mobilized for margin calls at 8am in three time zones), and contingency plans (if a CCP or CSD fails, or a correspondent is sanctioned). This is why the *liquidity* and *collateral* disciplines — not just connectivity — are the substance of bank-FMI integration.

---

## 10. The Bank Architect's View

Designing bank systems that integrate with FMIs — the practical architecture concerns for a solution architect (the lens of this repository's author):

**1. Messaging standards: ISO 20022 as the canonical model.** The direction of travel is unambiguous: ISO 20022 everywhere (SWIFT CBPR+, T2/T2S/TIPS, Fedwire/CHIPS, MEPS+, instant schemes). The architect's move: model payments and settlement instructions in ISO 20022 *internally* as the canonical format, and translate at the edges (MT↔MX coexistence, legacy card formats, scheme-specific quirks) rather than letting each FMI's dialect leak into the core. See `payments_hub_guide.md` for the hub pattern.

**2. Settlement integration.** The bank's systems must handle: RTGS queueing and liquidity (intraday position management, collateral for intraday credit), DVP settlement instructions (SSIs per CSD/market), netting cycles (ACH/CHIPS-style end-of-day settlement), and fail management (T+1 timelines, buy-ins, penalties). Settlement integration is not a "message in/message out" problem — it is a *liquidity and position* problem: the architect must expose settlement positions and cash forecasts to treasury in real time.

**3. Connectivity.** A layered connectivity strategy: SWIFT (with gpi and CSP compliance) for interbank; FIX for trading; direct venue/CCP/CSD connectivity (or via extranet — BT Radianz-class networks); market data feeds (vendor + direct, with MDM/entitlement governance — see `low_latency_cpp_development_guide.md`); and cloud private interconnects for analytics/DR. Every connection needs versioned protocols, failover, and full audit.

**4. Regulatory reporting.** Design reporting as a product: trade reports to TRs (EMIR/Dodd-Frank/MAS 677 — with LEI hygiene and reconciliation against the TR's records), transaction monitoring, liquidity and capital reports. The data must be *the same data* the bank uses to operate — reporting from a separate silo guarantees reconciliation pain. See `financial_risk_compliance_systems_guide.md`.

**5. The FMI landscape map for a bank.** Every bank should maintain an explicit map of the FMIs it touches, by region and business line — who the CCP, CSD, RTGS, TR, and scheme counterparties are, the legal basis, the operational dependencies, and the contingency plans. For a global bank (e.g., a French bank with an Asian hub): euro FMIs (T2/T2S, LCH/Eurex, Euroclear/Clearstream) for the home market; US FMIs (Fedwire/CHIPS, DTCC, CME/ICE) for USD; Asia FMIs (MEPS+/FAST, CDP/SGX, CHATS, BOJ-NET, RITS) for the region. The map drives: membership strategy, collateral location, intraday liquidity planning, and DR design.

**6. Resilience and change management.** FMI changes (T+1, ISO 20022 migrations, new CCPs, instant rails) arrive on hard deadlines and touch every downstream system. The architect's toolkit: a release calendar of FMI changes, impact analysis per system, certification environments (SWIFT's, the CCPs', the central banks'), and rehearsed rollback. FMI change is a *program*, not a project.

**7. Integration patterns that recur.** Most bank-FMI integrations reduce to a small set of patterns, worth recognizing to avoid reinventing them: **request-reply** (an instruction and its acknowledgement — SWIFT MT/MX, ISO 20022 pain/pacs), **event subscription** (settlement status, margin calls, corporate actions, market data — streaming/CEP; see `event_stream_processing_guide.md`), **batch file exchange** (ACH files, end-of-day statements, reporting), **idempotent outbox** (the bank's instruction ledger feeding the FMI, with exactly-once delivery), and **reconciliation loop** (bank records vs FMI records — the daily nostro/position rec is the cheapest systemic-risk control a bank runs). Every FMI interface in the map in point 5 can be classified into these patterns, which lets one integration framework (messaging hub + reconciliation engine) serve them all.

**8. The vendor and make-vs-buy layer.** Banks rarely build FMI connectivity from scratch: they buy SWIFT (Alliance, Alliance Lite2, cloud connectors), payments hubs (see `payments_hub_guide.md`), settlement/custody systems (Euroclear/Clearstream/DTCC interfaces via custodians or sub-custodians), clearing connectivity (CCP APIs and FIX/ISO 20022 clearing messages), and market data platforms. The architect's job is the *integration architecture* around them — canonical data model, idempotency, monitoring, and the reconciliation loop — so that the FMI map stays coherent even as vendors change.

---

## 11. The Singapore Context

Singapore's financial infrastructure is compact, modern, and closely overseen by MAS — a useful model of the whole taxonomy in one market.

| Layer | System | Notes |
|---|---|---|
| RTGS | **MEPS+** (MAS Electronic Payment System, since 2006) | SGD real-time gross settlement; also clears retail batches (FAST, GIRO) through its settlement engine; intraday liquidity via MAS facilities |
| Instant payments | **FAST** (2014), **PayNow** (2017) | FAST = 24/7 instant SGD transfers, settled through MEPS+; PayNow = proxy scheme (mobile/NRIC/UEN) operated by BCS under ABS, interoperable with FAST; PayNow linked to Thailand (PromptPay), India (UPI), Malaysia (DuitNow) for cross-border instant |
| ACH/batch | **GIRO** | Recurring debit/credit (bills, payroll); batch, net settlement via MEPS+ |
| Card | NETS, Visa, Mastercard | Domestic debit (NETS) plus global schemes |
| CSD | **CDP** (The Central Depository (Pte) Ltd, SGX subsidiary) | Book-entry CSD for Singapore equities, bonds, ETFs; settlement via DVP in central bank money (MEPS+) |
| SSS | CDP settlement + SGX's clearing | T+2 (T+1 discussions under the global wave) |
| CCPs | **SGX-DC** (SGX Derivatives Clearing) | Clears SGX derivatives — equity index futures/options (Nifty, MSCI Asia), FX, commodities (iron ore, freight); also SGX Securities Clearing for cash equities; plus regional clearing (CDP as CSD for SGX-listed) |
| Trade repository | **DTCC Data Repository (Singapore) Pte Ltd** (MAS-licensed TR) | OTC derivatives reporting under **MAS Notice 677** (trade reporting) — banks report all OTC derivatives (FX, rates, credit, equity, commodity) to the licensed TR; MAS uses the data for market surveillance |
| Exchanges | **SGX** (Singapore Exchange) | The integrated venue: securities + derivatives + clearing + depository under one group |

**MAS as regulator and operator.** MAS wears both hats: it *operates* MEPS+ (and the USD/CNY settlement arrangements), and it *oversees* the payment systems (FAST, GIRO under the **Payment Services Act 2019**, which replaced the Payment Systems (Oversight) Act) and the securities infrastructure (CDP, SGX-DC under the **Securities and Futures Act**). MAS applies the **PFMI** to Singapore's systemically important FMIs (MEPS+, FAST, CDP, SGX-DC), is a member of CPMI and IOSCO, and participates in FSB workstreams. A Singapore bank's FMI map is short but dense: MEPS+ (SGD cash), FAST/PayNow (retail instant), CDP (securities), SGX-DC (derivatives clearing), DTCC TR Singapore (derivatives reporting), plus SWIFT, CLS (SGD is a CLS currency), and the global FMIs for USD/EUR/CNH.

**The regulatory specifics a Singapore architect lives with:**

- **MAS Notice 677 (Trade Reporting)** — banks must report all OTC derivatives (FX, rates, credit, equity, commodity) to a MAS-licensed trade repository (DTCC Data Repository (Singapore) is the principal one), with LEI-based counterparty identification, daily valuation reporting, and strict timelines (T+1 for new trades). The reported data feeds MAS' surveillance and the FSB's global derivatives data agenda (the CPMI-IOSCO "unique trade identifier/unique product identifier" standards).
- **MAS Notices 610/622/623** — the AML/CFT notices for payment service providers (CDD, monitoring, STR, sanctions screening) — the risk layer around the payment infrastructure (see `financial_risk_compliance_systems_guide.md`).
- **The Payment Services Act (PSA)** — licensing for e-money, domestic/cross-border transfers, merchant acquisition, and DPT (digital payment token) services: the regime under which fintechs and stablecoin issuers plug into FAST/MEPS+ rails (directly or via sponsoring banks).
- **SGX/CDP rulebooks** — clearing membership, settlement cycles (T+2 for equities), buy-in and fail procedures, and the CDP's role in corporate actions; SGX moved to T+2 in 2016, and T+1 discussions follow the global wave.

**MAS' innovation agenda:** beyond the DLT projects below, MAS runs **Project Orchid** (exploring a retail "usable money" design for Singapore — purpose-bound money, programmable payments), the **stablecoin framework** (2023 — MAS-regulated single-currency stablecoins, SCS, with reserve and redemption rules), and the **Financial Services and Markets Act** reforms consolidating oversight. Singapore's positioning: a *tokenized-asset hub* — the world's densest concentration of regulated DLT experiments (Ubin, Guardian, Orchid, Dunbar, Nexus participation, mBridge engagement via BIS).

**Singapore's innovation track record — Ubin → Guardian → mBridge/Nexus:**

- **Project Ubin** (2016-2020, MAS + industry): five phases demonstrating DLT-based interbank payments (RTGS-on-DLT), DVP with a CSD, and cross-border settlement; the findings fed directly into the global wholesale-CBDC agenda.
- **Project Guardian** (2022-): MAS' flagship asset-tokenization program — industry pilots tokenizing fixed income, funds, FX, and insurance across banks (DBS, JPMorgan, BNY Mellon, HSBC, Standard Chartered, Marketnode, etc.); the reference point for "tokenized securities need the same FMIs, tokenized."
- **Project Dunbar** (2022, MAS + BISIH + RBA + SARB + BoT): multi-CBDC settlement on a shared ledger — the direct precursor to mBridge-style platforms.
- **mBridge** (BISIH + PBoC + HKMA + BoT + CBUAE, 2021-; Saudi Central Bank joined 2024; MVP mid-2024): the multi-CBDC cross-border payments platform. Singapore has engaged with the mBridge agenda through its BIS work (and its own Ubin/Dunbar learnings) rather than as a founding participant; MAS' own cross-border instant-payments play is **Project Nexus** (BIS Innovation Hub initiative connecting domestic instant systems — India, Malaysia, Philippines, Singapore, Thailand — targeting live cross-border instant payments) plus the PayNow bilateral links.
- **MAS digital agenda beyond CBDC:** Project Orchid (retail CBDC/usable-money explorations), the digital-asset regulation framework (Payment Services Act licensing for DPT services, and the new Stablecoin Framework, 2023), and Singapore's positioning as a tokenized-asset hub (see `blockchain_technology_guide.md`).

For a Singapore-based architect, the practical takeaway: the island's infrastructure is *small enough to map completely* — a bank can and should have an end-to-end picture of its MEPS+/FAST/CDP/SGX-DC/TR dependencies, which is exactly the discipline the PFMI and MAS oversight assume.

---

## 12. The Future of Financial Infrastructure (2026+)

- **Real-time + T+1 convergence.** Settlement keeps compressing: instant for retail (24/7), T+1 for securities (US done; EU/UK 11 Oct 2027; India beyond), T+0 for tokenized assets. The boundary between "payment" and "settlement" is blurring — the same instant rails increasingly carry securities legs (DVP on instant rails, e.g., "delivery vs payment on FedNow-class systems" experiments).
- **Tokenization of everything.** The "everything tokenized" trend: bonds, funds, money-market instruments, private credit, FX, and real estate on DLT rails. Tokenized securities still need settlement finality, custody, and legal certainty — the debate is whether incumbent CSDs/CCPs run the tokenized layer (Euroclear DSI, DTCC's tokenization work, SIX SDX) or new entrants do. See `blockchain_technology_guide.md`.
- **DLT FMIs reaching production.** SDX (now inside SIX), mBridge's MVP, Euroclear/DTCC pilots, and wholesale stablecoins (Fnality, JPM Coin) are crossing from PoC to production; the pattern is *hybrid* — tokenized layers on top of central bank money and existing legal frameworks, not greenfield replacements.
- **Cross-border instant payments.** The FSB roadmap's 2027 targets, ISO 20022 harmonization (the "one standard" end-state), instant-rail interconnection (Nexus), and CBDC bridges (mBridge) attack the correspondent-banking cost base. Expect the "corridor approach": fastest progress in high-volume, politically aligned corridors (ASEAN, GCC, APAC).
- **Wholesale CBDC settlement.** Central banks are converging on wholesale tokenized central bank money as the settlement asset for tokenized markets (Project Agorá's unified ledger, mBridge, Ubin's legacy). Retail CBDC remains politically contingent (the US has no decision; digital euro is in preparation; e-CNY continues); wholesale CBDC is the pragmatic, bank-facing front.
- **AI/ML inside FMIs.** Real-time risk (anomaly detection in payment flows, fraud and mule detection on instant rails), CCP stress-testing and margin-model challengers, surveillance at exchange scale, and ops automation. The constraints are model risk management and explainability — see `financial_risk_compliance_systems_guide.md`, `complex_event_processing_guide.md`, `reinforcement_learning_algorithms_guide.md`.
- **Consolidation vs fragmentation.** Two forces pull against each other: consolidation (fewer, bigger CCPs and CSDs; the EU's CMU push; T2S-style centralization; the "clearing house of clearing houses" idea) and fragmentation (geopolitical: alternative messaging/settlement systems, clearing relocation post-Brexit, regional CBDC blocs, sanctions-driven decoupling). Architects must design for *both*: standard interfaces (ISO 20022) that keep options open.
- **Cyber resilience as the top operational risk.** The FSB cyber framework, PFMI cyber annex, and SWIFT CSP are the floor, not the ceiling: expect cyber recovery environments, intelligence-led red-team testing, and "assume breach" architectures to become standard (see `container_certificates_guide.md`).
- **A 2026-2027 watchlist.** Concrete near-term milestones to plan around: the **EU/UK/CH T+1 go-live (11 October 2027)** and its knock-on effects on Asian/US cross-border settlement windows; the **ISO 20022 end-state** (SWIFT CBPR+ full migration, Fedwire/CHIPS on ISO 20022, T2/T2S already there — the last MT-only corridors closing); **EMIR Refit** reporting (new data model, reconciliation, JSON/ISO 20022-based) and the parallel CFTC rewrites; **CSDR penalties and buy-ins** maturing in Europe; **mBridge's path to production** and Nexus-style instant-rail interconnection going live in ASEAN; **tokenized money-market funds and bonds** reaching critical mass (Project Guardian-style pilots moving to issuance at scale); and **AI-driven FMI operations** (anomaly detection on instant rails, CCP stress-test challengers) crossing the model-risk governance bar.
- **The FMI of the future:** 24/7/365, instant, programmable (smart-contract settlement), tokenized, data-rich (real-time reporting, embedded supervision), and AI-monitored — but still built on the same foundations: finality, DVP/PVP, margin, default management, and legal certainty. The PFMI is being reviewed to keep pace; the next decade's FMIs will look different, but the five types will remain the taxonomy.

---

## 13. The Umbrella Summary: One-Page Map

```
                        FINANCIAL INFRASTRUCTURE — THE MAP
┌──────────────────────────────────────────────────────────────────────────┐
│  MARKET TECHNOLOGY INFRASTRUCTURE (the communication & discovery layer)  │
│  Exchanges/venues │ Market data (SIP, vendors, direct feeds, MDM) │      │
│  Reference data (ISIN/CUSIP/LEI/SSI) │ Messaging (SWIFT, FIX, FpML,      │
│  ISO 20022) │ Connectivity (DMA, co-lo, extranets, cloud)                │
├──────────────────────────────────────────────────────────────────────────┤
│  FORMAL FMIs (the settlement layer of record — PFMI, CPMI-IOSCO)         │
│                                                                          │
│  PAYMENT SYSTEMS                 ── large-value RTGS (Fedwire, CHIPS,    │
│                                     T2, CHAPS, BOJ-NET, MEPS+, CNAPS)    │
│                                     retail/instant (FAST, FedNow, UPI,   │
│                                     SEPA Instant, PayNow)                │
│                                     cards (Visa, MC) · ACH (NACHA, GIRO) │
│  CSDs  ── safekeeping + asset servicing  (DTC, Euroclear, Clearstream,   │
│          CDP, JASDEC, NSDL/CDSL, CMU, CCDC, Austraclear)                 │
│  SSSs  ── DVP securities settlement (T2S, CREST, DTC/DTCC, CDP, RITS)    │
│  CCPs  ── novation, netting, margin, default waterfall (LCH, CME, ICE,   │
│          Eurex, SGX-DC, JSCC, OCC, NSCC/FICC)                            │
│  TRs   ── OTC derivatives records (DTCC DR, CME TR, ICE Trade Vault)     │
├──────────────────────────────────────────────────────────────────────────┤
│  CROSS-BORDER LAYER: correspondent banking · SWIFT · CLS (PVP) ·         │
│  ICSDs (Euroclear Bank, Clearstream Lux) · FSB roadmap · CBDC bridges    │
├──────────────────────────────────────────────────────────────────────────┤
│  BANKS (participants & users): RTGS accounts, CCP membership, CSD        │
│  accounts, SWIFT, schemes — payments→RTGS/ACH/cards; securities→CSD/SSS/ │
│  CCP; derivatives→CCP/TR; FX→CLS                                         │
└──────────────────────────────────────────────────────────────────────────┘
```

The one-sentence version: **payment systems move cash with finality, CSDs/SSSs move securities with finality, CCPs make trading safe from default, trade repositories make markets visible, and the market-technology layer (venues, data, reference data, messaging, connectivity) lets it all talk — with banks as the participants and users sitting on top.**

---

## 14. Glossary

| Term | Definition |
|---|---|
| **FMI** | Financial Market Infrastructure — a multilateral system among participating institutions for clearing, settling, or recording payments, securities, derivatives, or other financial transactions (PFMI definition) |
| **PFMI** | Principles for Financial Market Infrastructures — the CPMI-IOSCO global standard (2012): 24 principles for FMIs + 5 responsibilities for authorities |
| **CPMI / IOSCO** | Committee on Payments and Market Infrastructures (BIS) / International Organization of Securities Commissions — the joint standard-setters |
| **RTGS** | Real-Time Gross Settlement — settlement of payments individually and continuously in real time, in central bank money (Fedwire, T2, CHAPS, MEPS+, BOJ-NET) |
| **ACH** | Automated Clearing House — batch, deferred-net retail/commercial payments (NACHA, GIRO, Bacs, EACHA) |
| **CSD** | Central Securities Depository — holds securities in book-entry form; safekeeping, settlement, asset servicing (DTC, Euroclear, Clearstream, CDP) |
| **SSS** | Securities Settlement System — settles securities trades, typically DVP (T2S, CREST, DTC/DTCC) |
| **CCP** | Central Counterparty — interposes between buyer and seller; novation, netting, margin, default management (LCH, CME, ICE, Eurex, SGX-DC) |
| **TR** | Trade Repository — centralized record of OTC derivatives transactions (DTCC DR, CME TR, ICE Trade Vault) |
| **DVP** | Delivery versus Payment — securities delivered only against payment; eliminates principal risk in securities settlement |
| **PVP** | Payment versus Payment — both legs of an FX trade settle simultaneously (CLS); eliminates FX principal (Herstatt) risk |
| **T+1 / T+2** | Settlement cycle: settlement on the first/second business day after the trade date (US T+1 since May 2024; EU/UK/CH T+1 on 11 Oct 2027) |
| **Herstatt risk** | FX principal risk — named after Bankhaus Herstatt (1974); one leg settles, the other does not |
| **CLS** | Continuous Linked Settlement — the PVP FX settlement FMI (live 2002, ~18 currencies) |
| **Novation** | The CCP's replacement of the original trade with two contracts (CCP-buyer, CCP-seller) |
| **Netting** | Offsetting obligations to a net position; multilateral netting collapses gross obligations (CHIPS, NSCC, ACH) |
| **Default waterfall** | The CCP loss-allocation order: defaulting member's IM → its default fund → CCP skin-in-the-game → survivors' default fund → assessments |
| **IM / VM** | Initial margin (collateral for potential future exposure) / variation margin (daily mark-to-market settlement) |
| **SIMM** | ISDA Standard Initial Margin Model — the standard IM calculation for uncleared derivatives under UMR |
| **UMR** | Uncleared Margin Rules — phased margin requirements for non-cleared OTC derivatives (EMIR/Dodd-Frank) |
| **CBDC** | Central Bank Digital Currency — digital central bank money (retail: e-CNY; wholesale: mBridge, Ubin, Agorá) |
| **LEI** | Legal Entity Identifier — ISO 17442 20-character identifier for legal entities in financial transactions (GLEIF) |
| **ISIN / CUSIP / SEDOL / FIGI** | Instrument identifiers: international (ISO 6166), US/Canada, UK, and Bloomberg's open global identifier |
| **SWIFT** | The interbank messaging cooperative (1973): MT/MX messages, SWIFTNet, gpi, Customer Security Programme |
| **SWIFT gpi** | Global Payments Innovation — tracking, fee transparency, same-day credit for cross-border payments (2017) |
| **FIX** | Financial Information eXchange — the trading message protocol (orders/executions) between buy side, sell side, venues |
| **FpML** | Financial products Markup Language — XML standard for OTC derivatives (confirmation, valuation, collateral) |
| **ISO 20022** | The global financial messaging standard (XML/JSON); the convergence point for payments, securities, cards (CBPR+, T2, Fedwire, MEPS+) |
| **ISO 15022** | The legacy securities messaging standard (predecessor to ISO 20022 in the securities space) |
| **T2S** | TARGET2-Securities — the Eurosystem's securities settlement platform (euro DVP in central bank money, 2015-2017) |
| **T2 / TIPS** | The euro RTGS (live 2023) / TARGET Instant Payment Settlement (2018) |
| **DMA** | Direct Market Access — sending orders directly to a venue's matching engine via sponsored access or own membership |
| **Co-location** | Placing servers inside the exchange's data center to minimize latency |
| **MDM** | Market Data Management — entitlements, capture, normalization, and redistribution of market data |
| **SSI** | Standing Settlement Instruction — the pre-agreed account details used to settle a trade |
| **UETR** | Unique End-to-end Transaction Reference — the identifier that makes SWIFT gpi tracking possible |
| **CBPR+** | SWIFT's Cross-Border Payments and Reporting ISO 20022 usage guidelines (live Nov 2022; end-state Nov 2025) |
| **Settlement finality** | The point at which settlement becomes irrevocable and unconditional (PFMI P8) |
| **Buy-in** | Purchasing undelivered securities in the market after a fail, at the failing party's cost (CSDR, SEC/DTCC rules) |
| **FSB** | Financial Stability Board — the G20's standard-setter for financial stability (cross-border payments roadmap, cyber framework, Key Attributes) |

---

## 15. Conclusion

Financial infrastructure is the quiet layer that makes modern finance possible: payment systems that settle trillions daily in central bank money, CSDs and SSSs that keep the record of ownership, CCPs that make default survivable, trade repositories that make risk visible — all wrapped in the market-technology layer of venues, data, reference data, messaging, and connectivity. The PFMI gave the world a single yardstick for all of it; the G20 reforms made CCP clearing and trade reporting the law of the land.

For the architect, three threads matter. **First, standards are destiny:** ISO 20022, LEI, and harmonized messaging are the interfaces the next decade is being built on — design for them. **Second, risk is structural:** finality, DVP/PVP, margin, and default waterfalls are the properties that make the system safe; every integration should respect where finality happens and who bears which risk. **Third, the layer is changing:** real-time settlement, T+1, tokenization, wholesale CBDC, and AI-driven risk are not separate projects — they are the same infrastructure being rebuilt to be instant, programmable, and data-rich, without losing the legal certainty it was built on.

Singapore — with MEPS+/FAST/PayNow, CDP, SGX-DC, MAS oversight, and a two-decade record of DLT experiments (Ubin → Guardian → Dunbar/Nexus) — is as good a place as any to watch that rebuild happen close-up. The plumbing is becoming the product.

---
