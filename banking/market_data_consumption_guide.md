# Market Data Consumption: The Market-Data Value Chain — A Comprehensive Guide

**The Market-Data Value Chain (Venue → Feed → Vendor → Platform → Decision), the Exchanges and the Consolidated Tape (Reg NMS, the CTA/CQ and UTP SIPs, the 2020 Market Data Infrastructure Rule and its fate, the CT Plan and DataCT), the EU Consolidated Tape under ESMA (EuroCTP), the Data Vendors (Bloomberg, LSEG/Refinitiv, FactSet, ICE Data Services), the Feed Types (Level 1/Level 2, market-by-order vs market-by-price, indices, FX, corporate actions), the Consumption by Trading Platforms (OMS/EMS, low-latency direct feeds, multicast, co-location, tick capture), the Consumption by Portfolio Management (positions, valuations, P&L, IBOR/ABOR), the Consumption by Risk (real-time risk, VaR, the Greeks, limits), the Consumption by Analytics (quant research, backtesting, time-series stores), the Data-Management Layer (golden source, entitlements, governance), the Cymbal Bank Worked Example, the Claims Audit, and the Glossary — from the Exchange's Tick to the Trader's Screen, the Risk Engine's VaR and the Quant's Backtest, Verified Against Primary Sources**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Banking Domain / Capital Markets — the market-data deep-dive of the repository's capital-markets cluster: how market data is produced by the venues, consolidated by the national market system plans and the SIPs, packaged by the data vendors, and consumed by the four platform classes that run a modern bank or asset manager — trading (OMS/EMS and the low-latency execution stack), portfolio management (positions, valuations, P&L), risk (real-time risk, VaR and stress inputs, the Greeks, limit monitoring) and analytics (quant research, backtesting, data science). The guide verifies the US consolidated-tape structure (Reg NMS, the CTA/CQ and UTP plans, Tapes A/B/C, the 2020 SEC Market Data Infrastructure Rule, the D.C. Circuit litigation and the new CT Plan with its independent administrator DataCT), the EU consolidated tape under ESMA (the MiFIR review, EuroCTP for equities, Ediphy fairCT for bonds, Etrading Software for OTC derivatives), the vendor facts (LSEG/Refinitiv, ICE/Interactive Data, FactSet), the Greeks terminology, and the public facts of SIP fees and revenue distribution. The protocol mechanics (FIX market-data messages, the DDS bus, the trade lifecycle, IBOR/ABOR, VaR governance) are cross-referenced to the sibling guides — never re-derived. Everything verified this pass is marked ✅ with the source named in the Claims Audit (§11.1); anything that could not be re-verified is flagged ⚠ honestly and listed again in §12.1. No dates, numbers or facts were invented.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** the SEC final rule "Market Data Infrastructure" (Release 34-90610, File No. S7-03-20, December 2020 — https://www.sec.gov/files/rules/final/2020/34-90610.pdf), the SEC Regulation NMS page (https://www.sec.gov/rules-regulations/2005/06/regulation-nms), the CTA Plan site (ctaplan.com — Network A/B, the CTS/CQS feeds, participants, SIP metrics), the UTP Plan site (utpplan.com — Tape C, the UQDF/UTDF feeds, Nasdaq as administrator, the CT Plan transition), the CT Plan site (thectplanllc.com — the 2023 SEC order 34-101672, the Q2 2027 launch, the DataCT independent administrator) and consolidatedtape.com (DataCT), the ESMA Consolidated Tape Providers page and press releases (esma.europa.eu — EuroCTP selected 19 December 2025 and authorised 27 July 2026; Ediphy fairCT for bonds; Etrading Software for OTC derivatives), the LSEG press release on the completion of the Refinitiv acquisition and The TRADE's coverage (29 January 2021, ~US$27bn), the ICE investor-relations press releases on the Interactive Data acquisition (26 October 2015 announcement; completion, US$5.2bn) and the accompanying SEC Form 8-K exhibit, the FactSet Q4/FY2025 earnings release (investor.factset.com, 18 September 2025), the Sidley Austin client alert on the D.C. Circuit's May 2022 decision in Nasdaq v. SEC, MLex coverage of the partial vacatur of the SEC's SIP-governance order, Investopedia and Wikipedia (Greeks — secondary), and the plan websites' published SIP metrics. Everything verified this pass is marked ✅ with the source named in the Claims Audit (§11.1); anything that could not be re-verified is flagged ⚠ honestly and listed again in §12.1. No dates, numbers or facts were invented.
> **Last Updated:** August 2026
> **Companion guides (sibling, same folder — the capital-markets cluster):** [FIX Protocol](fix_protocol_guide.md) (the protocol itself — the market-data messages 35=V/W/X live there, §6 — cross-ref, do not re-derive) · [Investment Portfolio Operations](investment_portfolio_operations_guide.md) (the trade lifecycle, IBOR/ABOR §6.5, reference data §8, corporate actions §4 — cross-ref, do not re-derive) · [Enterprise Risk Management](enterprise_risk_management_guide.md) (VaR/stress-test governance — cross-ref §7.3, do not re-derive) · [Market Making in Singapore](market_making_singapore_guide.md) (low-latency and co-location — cross-ref §7.3, do not re-derive) · [Capital Markets Architecture](capital_markets_architecture_guide.md) (front/middle/back architecture and market data — cross-ref §5 and §11) · [Financial Trading Order Infrastructure](financial_trading_order_infrastructure.md) (OMS/EMS and order routing — cross-ref §5) · [Kafka Guide](kafka_guide.md) (the streaming/analytics lane — cross-ref §8)
> **Companion guides (technology/, prefix `../technology/`):** [DDS Guide](../technology/dds_guide.md) (the real-time market-data bus; its §10 is the Cymbal Bank market-data worked example this guide extends — cross-ref, do not re-derive) · [Large-Scale Web Data Acquisition](../technology/large_scale_web_data_acquisition_guide.md) (the data-acquisition engineering side — cross-ref §8) · [AI/LLM cluster](../technology/ai_llm/) (the data-science and RAG guides, e.g. `rag/` and the `llm_*` and `closed_loop_data_engineering_guide.md` — cross-ref §8)

---

**How to use this guide:** Section 1 is the overview — the one-paragraph answer and the key-facts table. Section 2 is the market-data ecosystem — the venues, the US consolidated tape (Reg NMS, the CTA/CQ and UTP plans, the three tapes, the SIPs, the 2020 Market Data Infrastructure Rule and its litigation fate, the new CT Plan and DataCT), and the EU consolidated tape under ESMA (the MiFIR framework and the three CTP selections). Section 3 is the data vendors — Bloomberg, LSEG/Refinitiv, FactSet and ICE Data Services, with the comparison table. Section 4 is the feed types — Level 1 vs Level 2, market-by-order vs market-by-price, indices, FX rates and corporate actions. Section 5 is consumption by trading platforms — OMS/EMS integration, execution (cross-referenced to the FIX guide), low-latency consumption (direct vs consolidated feeds, multicast, co-location), and tick data and tick-capture stores. Section 6 is consumption by portfolio management — positions and valuations, P&L construction, desk-level analytics, cross-referenced to the investops guide's lifecycle and IBOR/ABOR content. Section 7 is consumption by risk — real-time risk, VaR and stress inputs, the Greeks, limit monitoring, cross-referenced to the ERM guide. Section 8 is consumption by analytics — quant research, backtesting, data science, historical data and time-series stores. Section 9 is the data-management layer — golden source, reference data, entitlements and licensing, data governance and BCBS 239 alignment. Section 10 is the Cymbal Bank worked example — the market-data architecture that extends the DDS guide's §10 design. Section 11 is the claims audit (✅/⚠/❌). Section 12 closes the guide: §12.1 "What Could Not Be Verified", §12.2 the glossary, §12.3 cross-references and §12.4 the closing summary. Cross-references follow the repository convention: sibling guides in `banking/` are plain filenames; guides in `technology/` are prefixed `../technology/`. **Integrity convention:** ✅ = verified this pass against a primary or cited source (URL given in §11.1); ⚠ = flagged/unverified/approximate; ⚠-secondary = well-documented industry knowledge or secondary-source fact not re-verified live this pass. No dates, numbers or facts were invented.

---

## Table of Contents

1. [The Overview](#1-the-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Key-Facts Table](#12-the-key-facts-table)
2. [The Market-Data Ecosystem](#2-the-market-data-ecosystem)
   - 2.1 [The Venues and the Producers](#21-the-venues-and-the-producers)
   - 2.2 [Reg NMS and the Consolidated Tape](#22-reg-nms-and-the-consolidated-tape)
   - 2.3 [The Plans and the SIPs — CTA/CQ and UTP](#23-the-plans-and-the-sips--ctacq-and-utp)
   - 2.4 [The Three Tapes and the Three Feed Families](#24-the-three-tapes-and-the-three-feed-families)
   - 2.5 [The 2020 Market Data Infrastructure Rule and Its Fate](#25-the-2020-market-data-infrastructure-rule-and-its-fate)
   - 2.6 [The CT Plan and DataCT — the SIP Modernization](#26-the-ct-plan-and-datact--the-sip-modernization)
   - 2.7 [The EU Consolidated Tape under ESMA](#27-the-eu-consolidated-tape-under-esma)
3. [The Data Vendors](#3-the-data-vendors)
   - 3.1 [The Vendor Layer](#31-the-vendor-layer)
   - 3.2 [Bloomberg](#32-bloomberg)
   - 3.3 [LSEG / Refinitiv](#33-lseg--refinitiv)
   - 3.4 [FactSet](#34-factset)
   - 3.5 [ICE Data Services](#35-ice-data-services)
   - 3.6 [The Vendor Comparison Table](#36-the-vendor-comparison-table)
4. [The Feed Types](#4-the-feed-types)
   - 4.1 [Level 1 — Top of Book](#41-level-1--top-of-book)
   - 4.2 [Level 2 — Depth of Book](#42-level-2--depth-of-book)
   - 4.3 [Market-by-Order vs Market-by-Price](#43-market-by-order-vs-market-by-price)
   - 4.4 [Index Data](#44-index-data)
   - 4.5 [FX Rates](#45-fx-rates)
   - 4.6 [Corporate Actions and Reference-Data Feeds](#46-corporate-actions-and-reference-data-feeds)
5. [The Consumption by Trading Platforms](#5-the-consumption-by-trading-platforms)
   - 5.1 [The OMS/EMS Integration](#51-the-omsems-integration)
   - 5.2 [Execution and the FIX Market-Data Messages — Cross-Referenced](#52-execution-and-the-fix-market-data-messages--cross-referenced)
   - 5.3 [Low-Latency Consumption — Direct Feeds vs Consolidated](#53-low-latency-consumption--direct-feeds-vs-consolidated)
   - 5.4 [Multicast Distribution and Co-Location — Cross-Referenced](#54-multicast-distribution-and-co-location--cross-referenced)
   - 5.5 [Tick Data and Tick-Capture Stores](#55-tick-data-and-tick-capture-stores)
6. [The Consumption by Portfolio Management](#6-the-consumption-by-portfolio-management)
   - 6.1 [Positions and Valuations — Cross-Referenced](#61-positions-and-valuations--cross-referenced)
   - 6.2 [P&L Construction from Market Data](#62-pl-construction-from-market-data)
   - 6.3 [Desk-Level Analytics](#63-desk-level-analytics)
7. [The Consumption by Risk](#7-the-consumption-by-risk)
   - 7.1 [Real-Time Risk](#71-real-time-risk)
   - 7.2 [VaR and Stress-Test Inputs — Cross-Referenced](#72-var-and-stress-test-inputs--cross-referenced)
   - 7.3 [The Greeks and Sensitivities](#73-the-greeks-and-sensitivities)
   - 7.4 [Limit Monitoring](#74-limit-monitoring)
8. [The Consumption by Analytics](#8-the-consumption-by-analytics)
   - 8.1 [Research and Quant Workflows](#81-research-and-quant-workflows)
   - 8.2 [Backtesting](#82-backtesting)
   - 8.3 [Data Science and the AI Cross-References](#83-data-science-and-the-ai-cross-references)
   - 8.4 [Historical Data and Time-Series Stores](#84-historical-data-and-time-series-stores)
9. [The Data-Management Layer](#9-the-data-management-layer)
   - 9.1 [Golden Source and Reference Data](#91-golden-source-and-reference-data)
   - 9.2 [Entitlements and Licensing](#92-entitlements-and-licensing)
   - 9.3 [Data Governance — Lineage, Quality and BCBS 239](#93-data-governance--lineage-quality-and-bcbs-239)
10. [The Cymbal Bank Worked Example](#10-the-cymbal-bank-worked-example)
    - 10.1 [The Architecture at a Glance](#101-the-architecture-at-a-glance)
    - 10.2 [The Feed-Handler Layer](#102-the-feed-handler-layer)
    - 10.3 [The Bus — Extending the DDS Guide's §10](#103-the-bus--extending-the-dds-guides-10)
    - 10.4 [The Consumers — Trading, PMS, Risk, Analytics](#104-the-consumers--trading-pms-risk-analytics)
    - 10.5 [The Latency Tiers](#105-the-latency-tiers)
    - 10.6 [Entitlements](#106-entitlements)
11. [The Claims Audit](#11-the-claims-audit)
    - 11.1 [The Claims Audit — Verified, Flagged, Rejected](#111-the-claims-audit--verified-flagged-rejected)
12. [What Could Not Be Verified, the Glossary, Cross-References and the Closing Summary](#12-what-could-not-be-verified-the-glossary-cross-references-and-the-closing-summary)
    - 12.1 [What Could Not Be Verified](#121-what-could-not-be-verified)
    - 12.2 [The Glossary](#122-the-glossary)
    - 12.3 [Cross-References](#123-cross-references)
    - 12.4 [The Closing Summary](#124-the-closing-summary)

---

## 1. The Overview

### 1.1 The Short Answer

**Market data is the raw material of every revenue-generating and risk-controlling decision in a capital-markets firm, and consuming it well is an architecture problem, not a wiring problem.** The value chain runs: **venue → feed → vendor → platform → decision**. The exchanges and trading venues produce the ticks; the national market system (in the US, the CTA/CQ and UTP plans and their securities information processors, or SIPs) consolidate them into the protected, regulatory tapes (Tape A/B/C, the NBBO); the data vendors (Bloomberg, LSEG/Refinitiv, FactSet, ICE Data Services) aggregate, enrich, normalize and license those feeds alongside their own proprietary data; and the four platform classes consume the result at four different cadences — trading platforms at microseconds-to-milliseconds (direct feeds, multicast, co-location), portfolio-management platforms at mark-to-market and P&L intervals (intraday and end-of-day), risk platforms at near-real-time (VaR, stress, the Greeks, limit monitoring) and analytics platforms at batch and research time (historical tick data, time-series stores, backtesting). The 2020s are reshaping the production side of the chain: the US SIPs are being modernized into a single CT Plan with an independent administrator (DataCT) from 2027, the EU is building its first consolidated tape under ESMA (EuroCTP for equities, selected December 2025 and authorised July 2026), and the vendor layer is consolidating (LSEG's ~US$27bn acquisition of Refinitiv closed 29 January 2021; ICE's US$5.2bn acquisition of Interactive Data closed December 2015). This guide verifies that production side and then maps, platform class by platform class, how the data is actually consumed.

### 1.2 The Key-Facts Table

| Aspect | Fact | Status |
| --- | --- | --- |
| The value chain | Venue → feed → vendor → platform → decision | ✅ this guide §1–§9 |
| US consolidated tape | Three tapes: Tape A (NYSE-listed), Tape B (listed on exchanges other than NYSE and Nasdaq), Tape C (Nasdaq-listed); governed by the CTA Plan (A/B), the CQ Plan (quotes for A/B) and the UTP Plan (C) | ✅ SEC Release 34-90610; ctaplan.com; utpplan.com |
| The SIPs | CTA/CQ SIP run under NYSE-family administration (NYSE LLC administers Network A, NYSE American Network B); UTP SIP administered by Nasdaq; both publish the NBBO and LULD price bands | ✅ ctaplan.com; utpplan.com |
| Reg NMS | Adopted by the SEC in 2005 (final release June 2005, Release 34-51808); Rule 611 order-protection and the NBBO framework | ✅ sec.gov Regulation NMS page |
| 2020 Market Data Infrastructure Rule | Release 34-90610 (December 2020, effective 8 June 2021): competing consolidators, depth-of-book (five levels) + odd-lot content in consolidated data, new round-lot sizes; upheld by the D.C. Circuit in Nasdaq v. SEC, 24 May 2022 | ✅ SEC 34-90610; Sidley alert |
| SIP governance order | SEC's May 2020 NMS Plan Governance Order (Release 34-88827) — single-plan consolidation and one-third non-SRO committee representation; challenged (NYSE v. SEC, No. 21-1169) and partially vacated | ✅/⚠ Sidley alert (order); MLex (partial vacatur — date ⚠) |
| SIP modernization | The CT Plan: unified successor to the three equity data plans, per the SEC's September 2023 order (Release 34-101672); expected launch Q2 2027 (April 2027) with independent administrator DataCT | ✅ thectplanllc.com; consolidatedtape.com |
| EU consolidated tape | MiFIR review framework (Regulation (EU) 2024/791); ESMA selects one CTP per asset class for five-year mandates: EuroCTP (equities, selected 19 Dec 2025, authorised 27 Jul 2026), Ediphy fairCT (bonds, selected 3 Jul 2025), Etrading Software (OTC derivatives, selected 6 Jul 2026) | ✅ esma.europa.eu |
| LSEG/Refinitiv | All-share acquisition of Refinitiv completed 29 January 2021, valued at ~US$27bn | ✅ LSEG press release; The TRADE |
| ICE/Interactive Data | Announced 26 October 2015, completed December 2015, ~US$5.2bn (cash + stock) | ✅ ir.theice.com; SEC Form 8-K exhibit |
| FactSet | FY2025 (ended 31 Aug 2025): GAAP revenues US$2,321.7M (+5.4%); ASV US$2,405.6M; organic ASV US$2,370.9M (+5.7%) | ✅ investor.factset.com (18 Sep 2025) |
| Bloomberg terminals | Widely-cited estimates only: ~350,000 (Bloomberg marketing copy) to 355,000+ (2025 secondary estimates) | ⚠ secondary estimates |
| SIP economics | SIP revenues ~US$400M/yr average 2007–2017 per the SIP operating committees' 2018 joint report; fees public via plan fee schedules | ✅/⚠ committees' report via Wikipedia; ctaplan.com/utpplan.com publish pricing |
| The Greeks | Delta, gamma, theta, vega, rho — first- and second-order sensitivities of option value to spot, time, volatility and rate | ✅ Investopedia; Wikipedia (Greeks (finance)) |
| Direct-feed pricing | Proprietary exchange feed commercial terms generally NOT public | ⚠ not public |

---

## 2. The Market-Data Ecosystem

### 2.1 The Venues and the Producers

Market data originates at the **venues**: exchanges (NYSE, Nasdaq, Cboe, the regional and newer entrants such as MEMX, IEX, LTSE, 24X and the Texas Stock Exchange — the full participant roster is listed in §2.3), alternative trading systems (ATSs), dark pools, electronic communication networks (ECNs), and — for FX — the dealer banks, ECNs such as EBS and Refinitiv Matching and the multi-dealer platforms. Each venue produces a **proprietary (direct) feed** of its own trades and quotes, and each contributes its trades and quotes to the **consolidated feed** under the national market system. The venue is also the **licensor**: exchange market-data revenue is a significant and growing line of business for the exchange groups — which is precisely why the SIP governance fights in §2.5 and §2.6 have been so bitter, and why the SEC's 2020 rulemaking targeted the conflict of interest between the exchanges' role as SIP operators and their role as sellers of proprietary data.

The production side of the value chain is where the regulated plumbing lives. In the US, the **Securities Acts Amendments of 1975** added Section 11A to the Securities Exchange Act of 1934, authorizing the SEC to facilitate a national market system for the collection, consolidation and dissemination of market information ✅ (Wikipedia, Securities information processor — secondary; the SEC's own rule releases describe the same statutory basis). The **CTA Plan** and **UTP Plan** were established in the late 1970s, each with an exclusive contract to consolidate and distribute market data for its set of securities ✅ (ctaplan.com — "Since the late 1970s, all SEC-registered exchanges and market centers that trade Network A or Network B securities send their trades and quotes to a central consolidator"). The European equivalent only arrived in the 2020s (§2.7) — the EU ran for decades without any consolidated tape at all, which is the entire point of the MiFIR review.

### 2.2 Reg NMS and the Consolidated Tape

**Regulation NMS** (Regulation National Market System) was adopted by the SEC in 2005 ✅ (sec.gov Regulation NMS page, final release June 2005). It redesigned the national market system rules adopted under Section 11A and added new substantive rules, the most famous of which is **Rule 611, the order protection rule** (17 CFR 242.611): trading centers must not execute orders at prices inferior to the protected quotations displayed by other trading centers ✅ (ecfr.gov — 17 CFR 242.611). The companion rules created the modern consolidated-data obligations:

- **Rule 601** (now part of the Reg NMS subpart): the requirement that exchanges and associations file transaction-reporting plans.
- **Rule 602**: the requirement that exchanges disseminate their quotations to the consolidated tape — no exchange may "prohibit, condition or otherwise limit" any vendor's ability to receive and redistribute consolidated data ✅ (SEC Rule 611 final-rule PDF, quoted text).
- **Rule 603**: the consolidated-quotation and last-sale reporting framework, including the exclusive-SIP model that the 2020 rule later amended.

Regulation NMS also gave the **NBBO** (national best bid and offer) its modern definition — the best protected bid and offer across all exchanges — and assigned the SIPs the job of calculating and disseminating it ✅ (Wikipedia, SIP article — secondary; ctaplan.com/utpplan.com both confirm the SIP role). The NBBO is the regulatory price: best-execution obligations, Rule 611's protected quotations, the Limit Up/Limit Down (LULD) price bands and short-sale restrictions all key off it, and the SIPs compute and distribute all of it ✅ (ctaplan.com; utpplan.com).

### 2.3 The Plans and the SIPs — CTA/CQ and UTP

The US consolidated-data architecture is organized as **joint-industry NMS plans** filed with and approved by the SEC under Section 11A. There are **three equity data plans** ✅ (SEC Release 34-90610, §I.A):

| Plan | Governs | Network / Tape |
| --- | --- | --- |
| **CTA Plan** (Consolidated Tape Association) | Consolidated transaction reports (trades) and quotations for NYSE-listed and other non-Nasdaq listed securities | Network A (NYSE-listed) and Network B (Bats, NYSE Arca, NYSE American and other regional exchanges) |
| **CQ Plan** (Consolidated Quotation Plan) | The collection, consolidation, processing and dissemination of quotation information for Tape A and Tape B securities | Tapes A and B |
| **UTP Plan** (Unlisted Trading Privileges Plan) | The collection, consolidation, processing and dissemination of last-sale and quotation information for Nasdaq-listed securities traded on exchanges on a UTP basis | Tape C |

**The SIPs.** Each plan selects a **securities information processor (SIP)** — a plan processor — that physically receives every trade and quote, consolidates them, computes the NBBO and the other regulatory outputs, and disseminates the consolidated streams. Today:

- The **CTA/CQ SIP** is run under NYSE-family administration: **the New York Stock Exchange LLC is the Administrator of Network A and NYSE American is the Administrator of Network B** ✅ (ctaplan.com). The historical operator of the infrastructure is SIAC (Securities Industry Automation Corporation), the NYSE-affiliated company that built and ran the tape systems ⚠-secondary (Wikipedia).
- The **UTP SIP** is administered by **Nasdaq**, which offers the system "in its capacity as the Administrator for the Nasdaq UTP Plan" ✅ (utpplan.com).

**Plan participants.** Every SEC-registered exchange and market center that trades the securities is a participant. The current CTA participant roster (ctaplan.com) includes 24X National Exchange, the Cboe family (BYX, BZX, EDGA, EDGX, Cboe Exchange), FINRA, IEX, LTSE, MEMX, MIAX Pearl, the Nasdaq family (Nasdaq, TEX, ISE, PHLX), the NYSE family (NYSE LLC, NYSE American, NYSE Arca, NYSE National, NYSE Texas) and the Texas Stock Exchange ✅. The UTP Plan's participants are the same universe of exchanges plus FINRA ✅ (utpplan.com governance page).

**Governance.** Each plan is run by an **Operating Committee** of the participant exchanges (and FINRA) that sets policy and oversees the processor, supported by an **Advisory Committee** of market participants — the CTA and UTP Advisory Committees include representatives of a retail institution, the vendor community, the institutional brokerage community, the ATSs and an individual investor ✅ (ctaplan.com; utpplan.com). This governance structure became a battlefield in the 2020 rulemaking and its litigation (§2.5).

**SIP performance facts** (published on the plan websites, which is itself a transparency requirement): the CTA SIP reports a 99.98% availability requirement with a hot/hot backup and a 10-minute recovery target, capacity of 2,700,000 quote messages and 425,000 trade messages per 100 milliseconds, and a median CQS/CTS latency under 20 microseconds ✅ (ctaplan.com); the UTP SIP reports the same 99.98% availability requirement, capacity of 2,700,700 quote and 2,734,100 trade messages per 100 ms, 100% availability in 53 of the last 63 quarters, and median latency of ~12.7–16.3 microseconds ✅ (utpplan.com, as of Q2 2026). These numbers matter for the consumption story: even the *slow* consolidated feeds are microsecond-class infrastructure, yet the proprietary direct feeds are faster still — which is the entire basis of the low-latency consumption pattern in §5.3.

### 2.4 The Three Tapes and the Three Feed Families

The SEC's own rule text states the split cleanly: *"The Equity Data Plans disseminate SIP data over three separate networks: (1) Tape A for securities listed on the New York Stock Exchange ('NYSE'); (2) Tape B for securities listed on exchanges other than NYSE and Nasdaq; and (3) Tape C for securities listed on Nasdaq"* ✅ (SEC Release 34-90610, §I.A). The data on each tape is disseminated as two streams ✅:

| Tape | Listed securities | Quote stream | Trade stream |
| --- | --- | --- | --- |
| **Tape A** | NYSE-listed | **CQS** (Consolidated Quote System) — quotes and the NBBO | **CTS** (Consolidated Tape System) — trades |
| **Tape B** | Listed on exchanges other than NYSE and Nasdaq (NYSE Arca, NYSE American, Bats/Cboe listings, regionals) | CQS | CTS |
| **Tape C** | Nasdaq-listed | **UQDF** (UTP Quotation Data Feed) — quotes and the NBBO | **UTDF** (UTP Trade Data Feed) — trades |

So a market-data consumer receives **three feed families** for US equities: the CTA/CQ family (Tapes A and B) and the UTP family (Tape C) ✅ (Wikipedia, National market system plan — secondary; consistent with ctaplan.com and utpplan.com). Beyond the equity tapes sits **OPRA** (Options Price Reporting Authority), the plan for all US exchange-traded options ✅ (Wikipedia — secondary). The consumption implication: a firm trading the whole US equity market must ingest at least the CTA/CQ and UTP consolidated feeds *plus* the direct feeds of every venue it trades on — the consolidation burden is structural, and it is the reason feed handlers and normalized internal symbologies exist (§5.3, §10.2).

### 2.5 The 2020 Market Data Infrastructure Rule and Its Fate

In **December 2020** the SEC adopted the **Market Data Infrastructure Rule** — Release 34-90610, File No. S7-03-20 — amending Regulation NMS "to modernize the national market system for the collection, consolidation, and dissemination" of NMS market data, effective **8 June 2021** ✅ (SEC 34-90610). The rule's three headline changes ✅ (SEC 34-90610; Sidley alert):

1. **Competing consolidators.** It removed the exclusivity of the SIPs as distributors of consolidated data: any entity may register with the SEC (new Rule 614 and Form CC) as a **competing consolidator** and distribute consolidated market data, creating a decentralized consolidation model.
2. **Richer consolidated content.** Consolidated data was expanded to include **five levels of depth-of-book** data and certain **odd-lot quotations** — previously depth and odd lots were only available from proprietary feeds.
3. **Round-lot reform.** New round-lot sizes that vary with the price of the security (replacing the uniform 100-share round lot for all equities above US$1), plus a raft of definitional changes (core data, protected bid/offer, auction information, regulatory data).

**The litigation.** The exchange groups challenged the rules. In **The Nasdaq Stock Market LLC et al. v. SEC**, No. 21-1100, a unanimous D.C. Circuit panel **upheld the 2020 rules on 24 May 2022**, rejecting the exchanges' arguments that the rules were arbitrary and capricious ✅ (Sidley alert; Wikipedia, SIP article — secondary). A *separate* front was the SEC's **May 2020 NMS Plan Governance Order** (Release 34-88827) — issued before the 2020 rule — which ordered the three equity data plans consolidated into a **single NMS plan**, cut affiliated-exchange voting power to no more than two votes, required **one-third non-SRO (non-exchange) voting representation** on the plan's operating committee (institutional investors, retail and institutional broker-dealers, a vendor, an issuer, a retail-investor representative), and required an independent plan administrator ✅ (Sidley alert, describing Release 34-88827 and the required representation categories). The exchanges challenged that order too — **New York Stock Exchange LLC et al. v. SEC**, No. 21-1169 — oral argument was held 24 March 2022 ✅ (Sidley alert). The D.C. Circuit ultimately **partially vacated** the SEC's order: the court held the SEC could not give SIP-committee seats (with voting power) to non-SRO stakeholders such as institutional investors ✅ (MLex: "US federal court partially vacates SEC order regarding market data" — the vacatur itself verified; the decision date and full scope ⚠, not re-verified this pass). The bottom line for consumers: the 2020 content-and-competition reforms stand ✅, the governance-reform limb was cut back in court ✅/⚠, and the modernization then proceeded through a different vehicle — the CT Plan (§2.6).

### 2.6 The CT Plan and DataCT — the SIP Modernization

The current state of US SIP modernization is the **CT Plan** — a new national market system plan that is the **unified successor to the three existing equity data plans (CTA/CQ and UTP)** ✅ (thectplanllc.com). Verified facts from the CT Plan's own site and its FAQ:

- **Legal basis:** the CT Plan was established pursuant to a **2023 SEC order** that directed the US equities exchanges and FINRA to act jointly in developing a single new NMS plan for real-time consolidated equity market data ✅ (thectplanllc.com FAQ, citing SEC filing 34-101672). Wikipedia independently records the September 2023 order ✅-secondary (Wikipedia, SIP article).
- **Governance:** an **Operating Committee** of all US equity exchanges and FINRA, plus an industry **Advisory Committee** — the same participant universe as today's plans ✅ (thectplanllc.com).
- **The administrator:** the Operating Committee ran an **RFP for an independent Administrator** (selection goal Q1 2026), and the chosen administrator is **DataCT** — a new, independent entity whose site (consolidatedtape.com) already publishes contracts, fees and technical specs ✅ (thectplanllc.com home page: "When the CT Plan begins operations in April 2027, there will be a new independent administrator, DataCT").
- **Timeline:** the CT Plan is expected to begin disseminating data in **Q2 2027 (April 2027)** ✅ (thectplanllc.com home page and FAQ); the current CTA/CQ and UTP plans will be phased out after launch, and the legacy sites (ctaplan.com, utpplan.com) now point their "New CT Plan" links at thectplanllc.com ✅ (utpplan.com).
- **Continuity commitments:** no changes to feed formats, latency, resiliency or capacity — the transition is administrative ✅ (thectplanllc.com FAQ); the 15-minute delayed public data remains free ✅ (thectplanllc.com FAQ).
- **Fees:** the Operating Committee was to submit a proposed fee schedule to the SEC by November 2025, subject to SEC approval ✅ (thectplanllc.com FAQ).
- **The odd-lot thread:** separately, the UTP and CTA/CQ Operating Committees in July 2025 sought a temporary exemption from certain requirements of SEC amendments to Regulation NMS in Release 34-101070 related to odd-lot depth ✅ (utpplan.com odd-lot page, exemption request dated 9 July 2025).

### 2.7 The EU Consolidated Tape under ESMA

The EU consolidated tape is a creature of the **MiFIR review** — Regulation (EU) 2024/791 (with Directive (EU) 2024/790 amending MiFID II) — which created the legislative framework for **consolidated tape providers (CTPs)** ✅ (esma.europa.eu CTP page, citing Regulation (EU) 2024/791). The model: **one CTP per asset class**, selected by ESMA through a public procurement-style procedure under the EU Financial Regulation, awarded a **five-year mandate**, then authorised and directly supervised by ESMA ✅ (esma.europa.eu CTP page). The CTPs collect market data from trading venues and approved publication arrangements (APAs) and consolidate it per instrument into a continuous electronic live stream for the public ✅ (esma.europa.eu CTP page).

The verified selection record:

| Asset class | Selection procedure launched | Selected provider | Status |
| --- | --- | --- | --- |
| **Bonds** | 3 January 2025 | **Ediphy (fairCT)** — announced 3 July 2025 | authorisation ongoing ✅ |
| **Equities (shares and ETFs)** | 20 June 2025 | **EuroCTP** — selected 19 December 2025; **authorised as EuroCTP B.V. on 27 July 2026** | authorisation completed ✅ |
| **OTC derivatives** | 5 January 2026 | **Etrading Software (Netherlands) B.V.** — announced 6 July 2026 | authorisation ongoing ✅ |

The equities provider is the one most relevant to this guide's consumption story: **EuroCTP is a joint venture based in the Netherlands with 15 European exchange groups as shareholders** ✅ (ESMA press release, 19 December 2025). ESMA's Executive Director Natasha Cazenave framed the selection as a milestone for the Savings and Investment Union ✅ (ESMA press release). Supporting milestones: the European Commission adopted the CTP technical standards in June 2025 (published in the Official Journal on 3 November 2025) ✅, and ESMA published the first list of data contributors to the equity CTP on 10 July 2025 ✅ (esma.europa.eu CTP page). The CTP input/output messaging uses new **ISO 20022 messages** developed at ESMA's request ✅ (esma.europa.eu CTP page, ISO 20022 RMG approval). The exact go-live date for the equity tape's live stream was not re-verified this pass ⚠ (industry press had anticipated a July 2026 launch ⚠-secondary) — the verified facts are the December 2025 selection and the 27 July 2026 authorisation.

---

## 3. The Data Vendors

### 3.1 The Vendor Layer

Between the feeds and the consumers sit the **data vendors**: the firms that take exchange and SIP feeds, add their own content (news, fundamentals, estimates, indices, analytics, pricing services), normalize the symbology and entitlements, and redistribute to the buy side and sell side under license. The vendor layer is the monetization surface of the whole value chain — most institutions never touch a SIP or an exchange feed directly; they buy the vendor's consolidated product. Four vendors dominate the institutional conversation (Bloomberg, LSEG/Refinitiv, FactSet, ICE Data Services); market-share rankings among them are press estimates and are flagged ⚠ rather than asserted ✅ (see §12.1).

### 3.2 Bloomberg

Bloomberg's flagship is the **Bloomberg Terminal** (the Professional service) — the iconic black-and-orange multi-monitor workstation whose value proposition is not just data but the workflow: real-time prices, news, analytics, the chat/messaging layer (Instant Bloomberg), and the buy-side OMS (Bloomberg AIM — cross-referenced in the sibling [Investment Portfolio Operations](investment_portfolio_operations_guide.md) guide §10.4). Bloomberg also runs the **B-PIPE** (Bloomberg Professional Information Pipeline) data feed for enterprise distribution, and its **Data License** service for bulk reference and historical data.

**Subscriber numbers are the classic ⚠ item of this guide.** Bloomberg does not publish terminal counts in its financial statements (it is a private company), so every figure in circulation is a widely-cited secondary estimate: Bloomberg's own marketing pages say the Terminal sits "on the desks of 350,000 of the world's most influential decision makers" ⚠ (bloomberg.com professional page — marketing copy, not an audited figure), and secondary trackers cite "over 355,000 Terminal users" citing Bloomberg (May 2025) ⚠ (tradersdna.com — secondary aggregator). Treat any single Bloomberg subscriber number as an estimate, not a verified fact ⚠.

### 3.3 LSEG / Refinitiv

The London Stock Exchange Group's **~US$27bn all-share acquisition of Refinitiv** — announced August 2019 — **completed on 29 January 2021** ✅ (LSEG press release "LSEG all-share acquisition of Refinitiv transaction — completion"; The TRADE, 29 January 2021: "LSEG completes $27 billion Refinitiv acquisition"). The deal made LSEG one of the world's largest financial-data and analytics groups: Refinitiv brought the former Thomson Reuters Financial & Risk business — the **Eikon/Workspace** desktop, the Elektron real-time distribution platform, the FX matching platform, and the LSEG pricing and reference-data franchises — into the LSEG group alongside the exchange. The relevant consumer-facing product today is **LSEG Workspace** (successor to Eikon), and the real-time data platform is LSEG's **Real-Time** (Elektron) SDK and feeds ✅ (lseg.com product pages — see §11.1). The exact revenue split between the exchange and data businesses was not re-verified this pass ⚠.

### 3.4 FactSet

FactSet is the buy-side analytics and data platform of record for much of the institutional asset-management industry. Verified from the company's own **Q4/FY2025 earnings release of 18 September 2025** ✅ (investor.factset.com):

- **FY2025 GAAP revenues: US$2,321.7 million**, up 5.4% year over year (Q4 2025 revenues US$596.9M, +6.2%) ✅.
- **ASV (Annual Subscription Value): US$2,405.6 million** at 31 August 2025, versus US$2,255.4M a year earlier; **organic ASV US$2,370.9M, +5.7%** ✅.
- GAAP diluted EPS US$15.55 (+11.8%); adjusted diluted EPS US$16.98 (+3.2%) ✅.
- 45+ consecutive years of increased revenues; 29th consecutive year of increased adjusted diluted EPS ✅.
- Buy-side clients represent ~82% of ASV; client count +9.5%, users +9.7% in FY2025 ✅.
- FY2026 guidance: GAAP revenues US$2,423–2,448M, organic ASV growth 4–6% ✅.

The sibling [Investment Portfolio Operations](investment_portfolio_operations_guide.md) guide §13.1 also verifies FactSet's purchase of **CUSIP Global Services** (completed 1 March 2022, ~US$1.925bn) ✅ — a reference-data fact this guide cross-references rather than re-derives.

### 3.5 ICE Data Services

**Intercontinental Exchange (ICE)** entered the data-vendor tier through the **Interactive Data Corporation (IDC) acquisition**: a definitive agreement was announced **26 October 2015** ✅ (ICE press release, ir.theice.com), and the **~US$5.2bn cash-and-stock transaction completed in December 2015** ✅ (ICE completion press release, ir.theice.com: "completed its previously announced $5.2 billion acquisition"; the SEC Form 8-K exhibit breaks the consideration down: ~US$3.65bn in cash plus ~6.47 million ICE shares valued at ~US$1.55bn as of 23 October 2015 ✅). Interactive Data brought the **pricing and valuation services** franchise (bond evaluations, reference data, real-time feeds) into ICE, forming today's **ICE Data Services** division alongside the exchange data (NYSE proprietary feeds, ICE Consolidated Feed) and indices (ICE Data Indices). ICE also acquired the **SIP-related and index assets** over the years (e.g., the 2022 acquisition of Black Knight's index business ⚠-secondary — not re-verified this pass), but the verified anchor fact is the IDC deal.

### 3.6 The Vendor Comparison Table

| Vendor | Flagship products | Verified anchor facts | Market share |
| --- | --- | --- | --- |
| **Bloomberg** | Terminal, B-PIPE, Data License, AIM (OMS) | Private company; terminal count is estimate-only: ~350,000 (marketing) to 355,000+ (secondary 2025) | ⚠ estimates |
| **LSEG / Refinitiv** | Workspace (ex-Eikon), Real-Time/Elektron feeds, FX matching | Refinitiv acquisition completed 29 January 2021, ~US$27bn all-share | ⚠ estimates |
| **FactSet** | FactSet workstation/analytics, data feeds, CUSIP Global Services (since 2022) | FY2025 revenue US$2,321.7M (+5.4%); ASV US$2,405.6M; organic ASV +5.7% | ⚠ estimates |
| **ICE Data Services** | Consolidated Feed, NYSE proprietary feeds, bond evaluations, indices | Interactive Data acquired for ~US$5.2bn, announced 26 Oct 2015, completed Dec 2015 | ⚠ estimates |

Other important names in the institutional vendor layer (Morningstar, Moody's, S&P Global Market Intelligence, MSCI, FactSet's rivals, the pricing-service specialists) are referenced where relevant but are not profiled here ⚠-secondary where any specific claims about them would be made — none are asserted in this guide.

---

## 4. The Feed Types

### 4.1 Level 1 — Top of Book

**Level 1** data is the top-of-book snapshot: the best bid and best offer (price and size) plus the last trade, per instrument. It is the smallest, cheapest, highest-volume-consumed market-data product in existence — the NBBO itself is a Level 1 construct, and the SIP quote streams (CQS, UQDF) are Level 1 streams ✅ (ctaplan.com/utpplan.com: the SIP quote feeds carry each member's best bid and offer and the NBBO). Level 1 is what most of a firm consumes: fair-value marks, P&L, compliance checks, portfolio valuation, simple display. In FIX terms, a Level 1 subscription is the classic `MarketDataRequest` (35=V) with `MDUpdateType` and the top-of-book `MDEntryType`s, delivered as `MarketDataSnapshotFullRefresh` (35=W) and `MarketDataIncrementalRefresh` (35=X) — the message mechanics are in the sibling [FIX Protocol](fix_protocol_guide.md) guide §6, not re-derived here.

### 4.2 Level 2 — Depth of Book

**Level 2** adds the depth of the order book — multiple price levels on both sides (the "book"), plus sometimes the identities of the market participants behind the quotes (in markets where that is visible). The economics of Level 2 are where the US market-data wars were fought: depth was historically available *only* from the exchanges' proprietary feeds, and the SEC's 2020 Market Data Infrastructure Rule changed the baseline by requiring the consolidated data to include **five levels of depth-of-book** plus odd-lot quotations ✅ (SEC Release 34-90610; Sidley alert). Depth data is consumed by market makers, algos, smart order routers and execution analytics — anyone whose edge depends on seeing beyond the top of book (the market-making consumption pattern is cross-referenced to the sibling [Market Making in Singapore](market_making_singapore_guide.md) guide §7.3). Level 2 also implies a *stateful* consumer: a book must be rebuilt and maintained from incremental updates, which is why the QoS design in the sibling DDS guide's §10.3 gives the L2 depth topic RELIABLE delivery with KEEP_LAST 5 while plain L1 ticks are BEST_EFFORT — freshness for L1, completeness for L2.

### 4.3 Market-by-Order vs Market-by-Price

Depth feeds come in two wire-level flavors:

- **Market-by-price (MBP):** the feed carries the aggregated book — each price level with its aggregate size — and updates are level deltas (add/change/delete at a price). The SIP depth content and many consolidated products are MBP-style.
- **Market-by-order (MBO):** the feed carries every individual order at every level, each with its own order ID, so a consumer can track order arrivals, cancellations and executions through the book. MBO is the richest and heaviest form of equity market data — the full "firehose" — and it is what the fastest proprietary feeds and the modern binary protocols deliver (CME's MDP 3.0-style binary multicast designs are the derivatives-market analogue, covered in the sibling [FIX Protocol](fix_protocol_guide.md) guide §8).

The consumption consequence is architectural: MBO feeds require per-order book reconstruction, much higher bandwidth and much more careful gap handling than MBP; firms typically ingest MBO for the handful of venues where their microstructure edge lives, and MBP everywhere else. No single primary source defines "MBP vs MBO" as a canonical pairing — the terms are standard industry usage ⚠-secondary (widely documented in exchange feed specifications, e.g., the UTP and proprietary feed specs published on utpplan.com and the exchanges' sites) — but the distinction itself is not controversial.

### 4.4 Index Data

**Index data** is a derived product class: index levels and constituents, produced by index administrators (S&P Dow Jones Indices, MSCI, FTSE Russell, Nasdaq, STOXX, ICE Data Indices, and the exchange-owned families) rather than by the matching engines. It is consumed everywhere the markets' benchmarks are consumed: performance measurement and attribution (cross-referenced to the sibling [Investment Portfolio Operations](investment_portfolio_operations_guide.md) guide §7 — the Brinson framework is built on benchmark returns), passive replication, derivatives valuation (index levels are the underlying of index futures and options), and risk (index-based factor exposures, beta). Index data has its own latency profile — a level tick is irrelevant; what matters is the official **closing level** computed by the administrator on the official methodology, delivered end-of-day or intraday as a separate feed — and its own data-quality discipline (revisions, corporate-action adjustments, constituent changes). The index *methodology* facts (who publishes which index family) were not re-verified this pass ⚠ beyond what is common knowledge; no specific index levels or methodologies are asserted here.

### 4.5 FX Rates

FX market data has a different production structure from equities: there is no consolidated tape — the "market" is the interbank dealer network, the ECNs (EBS, LSEG/Refinitiv Matching), the multi-dealer platforms and the continuous dealer quotes ✅-secondary (industry-standard description; the platform names are widely documented). Consumption comes in three forms:

- **Streaming spot and forward rates** for pricing, hedging and P&L — typically consumed as a vendor's consolidated FX feed (Bloomberg FXGO/FX feeds, LSEG FX, Refinitiv Matching data) ✅-secondary.
- **Fixings and benchmarks** — the daily reference rates (e.g., the ECB euro foreign-exchange reference rates, WM/Refinitiv and other fixings) that price portfolios, funds and derivatives — consumed once a day and subject to benchmark-regulation governance (the EU Benchmarks Regulation applies to administrators) ✅-secondary.
- **Forward points and cross rates** for the rates curve.

No specific FX rate levels or fixing values are asserted in this guide; the structural claim — that FX lacks a regulated consolidated tape and is vendor-mediated — is well-documented industry knowledge ⚠-secondary rather than a re-verified primary fact.

### 4.6 Corporate Actions and Reference-Data Feeds

Corporate actions are the event data that changes *what* a position means: dividends, splits, mergers, rights issues, name changes, ISIN changes. They are not tick data — they arrive on their own announcement-driven feeds (SWIFT MT564/MT566, ISO 20022 seev.xxx, or vendor corporate-action services) and they trigger entitlement processing, position adjustments and valuation changes. This guide does **not** re-derive the corporate-action types, the mandatory-vs-voluntary election, the announcement-to-payment lifecycle or the message flows — all of that lives in the sibling [Investment Portfolio Operations](investment_portfolio_operations_guide.md) guide §4 (with the verified MT564/MT566 and ISO 20022 seev.xxx message mapping in its §4.4) and its reference-data section §8 (security master, ISIN/CUSIP/SEDOL/LEI identifiers). The consumption-side point here is simple: **every downstream consumer in this guide assumes the corporate-action feed has been applied** — a trading desk's price, a PMS's position, a risk engine's exposure and a quant's history are all wrong if the dividend or split adjustment is missing, which is why corporate actions are a golden-source and data-governance problem (§9.1, §9.3) as much as a feed problem.

---

## 5. The Consumption by Trading Platforms

### 5.1 The OMS/EMS Integration

The trading platform is the front-office consumer with the most demanding latency and the most complex integration surface. The order management system (OMS) holds the order lifecycle and compliance state; the execution management system (EMS) provides the trading blotter, the venue connectivity and the execution tools; both consume market data continuously. The platform architecture — OMS/EMS roles, order routing, smart order routing (SOR), matching engines, ECN/ATS connectivity — is documented in the sibling [Financial Trading Order Infrastructure](financial_trading_order_infrastructure.md) guide and cross-referenced in the [FIX Protocol](fix_protocol_guide.md) guide §7; this section covers only the *market-data consumption* pattern:

- **The blotter needs prices, not ticks.** The EMS displays each order's current market (L1), the book (L2) for the working venue, and the order's own execution status (from FIX `ExecutionReport`s). The integration pattern is: market-data bus feeds the EMS; FIX sessions feed the order state; the EMS correlates them per order (the correlation of market prices to order state is exactly what the sibling DDS guide's §10.2 topics — `MarketDataTick` and `OrderState` — are designed to carry).
- **Pre-trade checks need the NBBO.** The NBBO is the reference for limit checks, marketability checks and best-execution analytics; the OMS/EMS typically subscribes to the SIP-derived NBBO stream (or the vendor's consolidated product) rather than computing it from direct feeds.
- **SOR needs every venue's book.** A smart order router consumes the direct feeds of all venues it routes to, builds a consolidated picture per symbol, and computes where to send each child order — the routing logic itself is the sibling infrastructure guide's subject, but its data dependency is this guide's: SOR is a Level 2/aggregated-book consumer.
- **Algos need ticks plus state.** Algorithmic execution (VWAP, TWAP, implementation shortfall, icebergs) consumes tick streams to compute benchmarks and participation, and consumes order-state updates to adapt.

### 5.2 Execution and the FIX Market-Data Messages — Cross-Referenced

The FIX protocol is the language of execution, and it also carries market data: `MarketDataRequest` (35=V), `MarketDataSnapshotFullRefresh` (35=W), `MarketDataIncrementalRefresh` (35=X), `MarketDataRequestReject` (35=Y), plus the quote messages (35=S, 35=R) — the tag-level mechanics, the `MDEntryType` values and the `MDUpdateType` semantics are all verified in the sibling [FIX Protocol](fix_protocol_guide.md) guide §6 and are **not re-derived here**. The consumption pattern worth stating: FIX market data is a *session-based request/reply and incremental* model — appropriate for desktop tools, soft real-time apps and vendor connectivity — while the highest-rate venues have moved to binary multicast protocols (e.g., CME MDP 3.0, covered in the FIX guide §8) precisely because a FIX session per consumer does not scale to tick rates. The architectural rule that follows (and that the DDS guide's §10.5 boundary encodes): **FIX stays at the venue boundary; internal fan-out happens on the market-data bus.**

### 5.3 Low-Latency Consumption — Direct Feeds vs Consolidated

For the latency-sensitive consumer — market makers, high-frequency and execution-focused desks — the choice between **direct (proprietary) feeds** and the **consolidated SIP feeds** is the first architectural decision:

- **Consolidated feeds** (CTA/CQ, UTP) give the complete protected market — the NBBO, every venue's protected quotes, the regulatory outputs (LULD bands, halts) — from one or two connections, at microsecond-class latencies (the CTA SIP publishes median latency under 20µs; the UTP SIP ~12.7–16.3µs ✅ per the plan websites' published metrics, §2.3). The cost is that consolidated data waits for the slowest participant's contribution to be included, and the SIP itself is a single processing hop.
- **Direct feeds** (each exchange's proprietary stream) give that venue's data *before* it reaches the SIP — the exchange publishes its own feed directly from the matching engine — plus depth and order-level content the SIP does not carry. The latency edge is real and is the entire basis of the co-location and feed-handler industry (§5.4), but a firm trading everywhere must ingest *N* direct feeds and reconstruct the consolidated view itself.

The consumption pattern is therefore **both**: direct feeds for the venues where speed matters, consolidated feeds for the regulatory NBBO and for everything else — a pattern this guide's worked example implements in §10. The fees involved split the same way: SIP fees are public and regulated (plan fee schedules, §9.2), while direct-feed commercial terms are negotiated and generally not public ⚠.

### 5.4 Multicast Distribution and Co-Location — Cross-Referenced

Two pieces of infrastructure make low-latency consumption physical:

- **Multicast distribution.** Exchange feeds and SIP feeds are delivered over UDP multicast (the UTP SIP publishes UDP/IP addresses and a transmission schedule for its feeds ✅ per utpplan.com; the binary multicast design of modern venue feeds is covered in the sibling [FIX Protocol](fix_protocol_guide.md) guide §8). Multicast is one-to-many at line rate, but it is lossy by design — which is why feed handlers must implement sequence-number gap detection and recovery (snapshot/refresh services, TCP retransmission channels), and why the internal bus QoS in the DDS guide's §10.3 treats L2 as RELIABLE.
- **Co-location.** The latency race is won in the exchange's data centre: matching-engine proximity, low-latency networking (kernel bypass), and the physical infrastructure that the sibling [Market Making in Singapore](market_making_singapore_guide.md) guide §7.3 documents (with SGX's Reach engine and co-location as the Singapore reference point) and the sibling [DDS Guide](../technology/dds_guide.md) §10 carries into the internal distribution design. This guide does not re-derive any of that engineering; it only records the consumption pattern: co-located feed handlers → kernel-bypass → the internal bus → the trading applications, all within the same data centre.

### 5.5 Tick Data and Tick-Capture Stores

Everything a trading platform *sees* is worth *keeping*: the tick history is the firm's evidence base for best-execution analysis, TCA (transaction cost analysis), microstructure research, backtesting and regulatory reconstruction. **Tick capture** is the discipline of recording every tick (L1 and L2, consolidated and direct) with exchange timestamps, and it runs as a permanent tap on the market-data bus — the compliance recorder in the DDS guide's §10.2 is exactly this component, writing the audit/analytics log (there, to Kafka). **Tick-capture stores** are the storage tier: high-throughput append-only systems designed for tens of millions of events per day per symbol universe, queried by symbol/time ranges. The storage landscape is covered in §8.4 (kdb+/q, Parquet/Delta, time-series databases); the trading-platform point is that tick capture is *not* an analytics afterthought — it is a regulated expectation (best-execution record-keeping) and the raw material for the analytics consumption in §8.

---

## 6. The Consumption by Portfolio Management

### 6.1 Positions and Valuations — Cross-Referenced

The portfolio-management platform (PMS — e.g., Aladdin, Charles River, Bloomberg AIM, SimCorp Dimension, Advent/Geneva, Eagle; the landscape is compared in the sibling [Investment Portfolio Operations](investment_portfolio_operations_guide.md) guide §10) consumes market data to answer one question continuously: **what do we own, and what is it worth?** The lifecycle — order → execution → allocation → confirmation/affirmation → settlement → custody — and the IBOR vs ABOR books-of-record debate (the golden-source question of whether the investment book or the accounting book is the single source of positions, cash and valuations) are both documented in the sibling investops guide (§2 and §6.5) and are **not re-derived here**. The consumption pattern this guide adds:

- **Valuation consumes multiple price types.** Mark-to-market uses the last trade or the closing price for listed securities; bid/offer mid for many instruments; evaluated prices from pricing services (ICE Data Services' bond evaluations are the classic example, §3.5) for illiquid bonds; model prices for derivatives; FX rates for translation. The fair-value hierarchy (ASC 820 / IFRS 13) that governs which price source is appropriate sits in the investops guide §6.4 ✅ — the PMS must tag every position with the price *source and level* it used, which makes valuation a market-data-governance consumer (§9.3).
- **End-of-day is a hard deadline.** The daily NAV/valuation run (and the T+1 settlement regime that compresses the whole operations day — verified in the investops guide §3) means the PMS's official marks are produced from a controlled **official close** snapshot — the closing prices, fixings and corporate-action-adjusted state — not from the raw tick stream. Most firms run a separate official-pricing feed (end-of-day vendor files, index closing levels, FX fixings) into the PMS, distinct from the real-time bus.

### 6.2 P&L Construction from Market Data

P&L is market data applied to positions: **P&L = Σ (position × price change) + (income and corporate-action adjustments)**. The construction pattern:

- **Realized vs unrealized.** Realized P&L comes from executions (FIX `ExecutionReport`s — cross-referenced to the [FIX Protocol](fix_protocol_guide.md) guide §6); unrealized P&L comes from market data (marks vs cost). Both must agree with the books — the reconciliation discipline is in the investops guide §5.
- **The P&L explain.** A desk-level P&L report "explains" the day's P&L by risk factor (price, FX, rates, volatility, time) — which requires the *same* market-data inputs that the risk engine uses for sensitivities (§7.3): the attribution of P&L to delta/gamma/vega/theta/rho is the portfolio-management view of the risk Greeks.
- **Intraday vs official.** Trading desks consume intraday marks from the real-time bus for flash P&L; the official P&L is produced from the official close. The two must reconcile to a tolerance — a standing data-quality control.

### 6.3 Desk-Level Analytics

Beyond valuation and P&L, the portfolio-management desk consumes market data for: **exposure monitoring** (position concentrations vs market caps, sector exposures, beta), **performance and attribution** (benchmark index data — §4.4 — feeding the Brinson attribution framework verified in the investops guide §7.2), **what-if analytics** (reprice the portfolio under shifted markets — the same scenario machinery the risk function runs in §7.2), and **client and mandate reporting**. The analytics consumption is characterized by *interactive queries over portfolio + market data* rather than raw ticks — a very different access pattern from §5, and the reason PMS analytics typically sit on a data platform (in-memory caches, the vendor's cloud analytics, or the firm's lakehouse) rather than on the real-time bus.

---

## 7. The Consumption by Risk

### 7.1 Real-Time Risk

The risk function consumes market data to keep the firm inside its risk appetite while the market moves. **Real-time risk** is the discipline of computing exposures and limits against the live market rather than the end-of-day state: the trading desks' intraday exposures, the market makers' inventory and quote risk, the FX and rates desks' open positions — all marked to the real-time stream. The consumption pattern: a risk engine subscribes to the same market-data bus as the trading platforms (the DDS guide's §10.2 risk engine subscribes to ticks, order state and position updates alike), re-marks positions on every relevant tick (or on throttled intervals), recomputes exposure aggregates, and compares them against limits (§7.4). The risk engine is also the classic consumer of the *market state* beyond prices: trading halts, LULD bands, venue status and circuit breakers (the SIPs disseminate the LULD bands and halts ✅ per ctaplan.com/utpplan.com) — because a halted stock must not be marked from stale trades.

### 7.2 VaR and Stress-Test Inputs — Cross-Referenced

Value-at-Risk and stress testing consume market data in bulk, historically: VaR needs the **historical return series** of every risk factor in the book (the time-series stores of §8.4 are the data source), and stress testing needs **scenario data** — either historical scenarios (a crash window replayed) or hypothetical shocks (parallel rate shifts, vol shocks, FX jumps) applied to current levels. The governance of the risk discipline — the VaR methodology choices (historical simulation, Monte Carlo, parametric), the model-validation and backtesting obligations, the stress-testing framework and the risk-appetite structure — is the subject of the sibling [Enterprise Risk Management](enterprise_risk_management_guide.md) guide and is **not re-derived here**. The market-data-specific points this guide adds:

- **Factor mapping is a data-management problem.** The book's instruments must be mapped to the risk-factor history (the same instrument reference data of §9.1); a new issue with no history is priced off proxies — a data-quality decision the risk function must document.
- **Data gaps are risk.** A VaR history with missing or bad days understates correlation and tail risk; the risk data feeds are therefore subject to the strictest completeness controls in the firm (§9.3).

### 7.3 The Greeks and Sensitivities

The **Greeks** are the standard sensitivities of derivative value to its inputs — the quantities a derivatives risk system computes from the market data (underlying price, volatility surface, rates, time) and the model. The terminology is standard and verified ✅:

| Greek | Symbol | Sensitivity of option value to | Order |
| --- | --- | --- | --- |
| **Delta** | Δ | changes in the underlying spot price | first |
| **Gamma** | Γ | changes in delta as the underlying moves (second derivative of value w.r.t. spot) | second |
| **Theta** | Θ | the passage of time (time decay) | first |
| **Vega** | ν (V) | changes in implied volatility | first |
| **Rho** | ρ | changes in the risk-free interest rate | first |

Verified against Investopedia ("The Greeks — delta, gamma, theta, vega, and rho — provide a way to measure the sensitivity of an option's price to quantifiable factors") ✅ and Wikipedia's "Greeks (finance)" article (delta, vega, theta and rho as the first-order derivatives, gamma as the second-order derivative of the value function) ✅. The consumption pattern: the risk engine needs the **volatility surface** (a market-data product in its own right — implied vols by strike and tenor, sourced from the options market or a vendor) and the **rates curve** alongside the underlying prices to compute the Greeks; the Greeks then feed the limit framework (delta limits, vega limits), the P&L explain (§6.2) and the hedging decisions.

### 7.4 Limit Monitoring

Limit monitoring is where real-time risk meets the trading desk: pre-trade and intraday limits (position limits, concentration limits, VaR limits, loss limits, margin limits) checked against the live position marked to live prices. The consumption pattern:

- **Pre-trade:** the OMS/EMS asks the risk engine before sending an order — the order's *projected* position marked at the current price must pass the limit. This is the compliance-and-guidelines layer the investops guide §9 documents for the buy side (mandate checking), applied here to the sell-side desk limits.
- **Intraday:** the risk engine re-checks limits on every position update and on limit breaches — the DDS guide's §10.2 design has the risk engine publish `PositionUpdate` instances that the trading apps and compliance both consume, which is the mechanical pattern.
- **Breach handling:** a breach triggers alerts, order throttling or kill switches — the *action* side is the risk-operations and trading-controls world (cross-referenced to the [Enterprise Risk Management](enterprise_risk_management_guide.md) guide), but the *data* side is this guide's: breach detection is only as good as the mark, which is only as good as the feed and the entitlement that lets the risk engine see it (§9.2).

---

## 8. The Consumption by Analytics

### 8.1 Research and Quant Workflows

The analytics function is the *historical* consumer: where trading consumes the present tick and risk consumes the recent window, the research desk and the quant team consume **everything, forever** — the full tick history, the reference data, the corporate actions, the fundamentals, the news — in batch. The quant workflow consumes market data through a research stack: a data platform that can answer "what did this symbol's book look like at 10:00:03.412 on any day in the last five years" and "what is the historical correlation matrix of these 500 factors". The engineering of acquiring and storing data at that scale — crawling, ingestion, cleaning, deduplication, versioning — is the subject of the sibling [Large-Scale Web Data Acquisition](../technology/large_scale_web_data_acquisition_guide.md) guide, which this guide cross-references rather than re-derives. The consumption pattern here: **research consumes datasets, not feeds** — the deliverable of the market-data chain to analytics is a *curated, queryable dataset* (tick data, EOD data, fundamentals, events) with clean symbology and documented adjustments.

### 8.2 Backtesting

Backtesting is the quant's falsification loop: take a strategy rule, replay it over historical market data, measure the hypothetical P&L. The market-data consumption requirements are brutal and specific:

- **Survivorship and look-ahead bias are data problems.** A backtest is only as honest as its data: delisted securities must stay in the history (survivorship bias), and only data available *at the time* (point-in-time) may be used (look-ahead bias). Point-in-time reference data — the corporate-action and identifier history of §4.6 and §9.1 — is therefore a first-class backtesting input, not an afterthought.
- **Tick-level replay.** Strategy backtests on microstructure signals (order flow, queue position, latency arbitrage) need the MBO/MBP tick histories of §4.3 and the tick-capture stores of §5.5 — often replayed at original timing through a simulation harness that models venue behaviour.
- **Cost models.** Execution costs (spread, impact, fees) are estimated from the market data itself — the spread data from the L1/L2 history, the impact from the volume profile — which makes the cost model a market-data consumer too.
- **The backtesting platform pattern.** The sibling [DDS Guide](../technology/dds_guide.md) §10.5 notes the division of labour that analytics relies on: real-time distribution on the bus, **replay and batch analytics in Kafka and the data platform** — the analytics lane is the batch lane, fed by the tick tap.

### 8.3 Data Science and the AI Cross-References

The data-science layer consumes the same datasets through statistical and machine-learning workflows: feature engineering from tick and EOD data, model training on return/label history, and production inference on live features. The AI/LLM cluster of this repository documents the modern machinery this layer uses — this guide cross-references rather than re-derives:

- The **RAG guides** under [../technology/ai_llm/rag/](../technology/ai_llm/rag/) (e.g., `rag_with_data_streaming_guide.md`, `rag_vs_long_context_llms_guide.md`, `production_grade_agentic_rag_guide.md`) — retrieval over documents and data, including the pattern of grounding financial models on retrieved market/company data.
- The **LLM guides** under [../technology/ai_llm/](../technology/ai_llm/) (e.g., `llm_optimization_complete_guide.md`, `llm_latency_optimization_guide.md`, `llm_evaluation_frameworks_guide.md`) — the model-side engineering.
- **`closed_loop_data_engineering_guide.md`** under [../technology/ai_llm/](../technology/ai_llm/) — the data-engineering loop that keeps training and inference data fresh, which is the AI-side mirror of this guide's data-management layer (§9).

The market-data-specific points: financial ML features must be built from **adjusted, point-in-time data** (the same bias discipline as §8.2); and production inference consumes the real-time bus (a feature store fed by the same normalized stream the trading apps see — §10.4).

### 8.4 Historical Data and Time-Series Stores

The storage tier for market data is specialized because the access pattern is specialized: **high-ingest, time-ordered, symbol-partitioned, range-queried**. The landscape:

- **kdb+/q** (Kx) — the institutional tick-database standard: an in-memory columnar time-series database designed for exactly this workload (millions of inserts/sec, chained intraday-to-historical databases), long the default for sell-side tick history and TCA. kdb+/q's market-data role is well-documented industry knowledge ⚠-secondary (no specific product claims are asserted beyond its role as the canonical tick/time-series platform).
- **Time-series databases** — InfluxDB, TimescaleDB (PostgreSQL extension), and the cloud TSDBs: general-purpose time-series stores used for the operational and analytics time-series (market data at lower cardinality, system metrics, intraday analytics) ✅-secondary (the products are real and documented on their own sites; no version or benchmark claims are made here).
- **Columnar/analytics formats** — Parquet (with Delta Lake / Iceberg table formats) and Arrow: the lakehouse path, where tick and EOD history is stored as partitioned Parquet in object storage and queried with Spark/DuckDB-class engines; the Delta/iceberg table formats provide the ACID and time-travel semantics that point-in-time analytics need ⚠-secondary.
- **The data-acquisition engineering** that fills these stores — batching, rate limiting, retries, deduplication, schema evolution — is cross-referenced to the sibling [Large-Scale Web Data Acquisition](../technology/large_scale_web_data_acquisition_guide.md) guide.

No performance benchmarks or vendor claims beyond the products' existence and roles are asserted here ⚠-secondary where specifics would otherwise be needed.

---

## 9. The Data-Management Layer

### 9.1 Golden Source and Reference Data

Every consumer in §5–§8 believes it is looking at *the same* instrument when it looks at a symbol — that belief is manufactured by the data-management layer. The **golden source** is the firm's single authoritative copy of each data domain: the security master (instrument reference data — identifiers, terms, calendars, settlement conventions), the counterparty master, the price/valuation sources, the corporate-action history. The identifiers and their issuing bodies (ISIN/ANNA, CUSIP/FactSet, SEDOL/LSEG, LEI/GLEIF) are verified in the sibling [Investment Portfolio Operations](investment_portfolio_operations_guide.md) guide §8.2 and are **not re-derived here**. The consumption pattern this guide adds:

- **Normalization at the edge.** Feed handlers map every external symbology (venue symbols, vendor IDs) onto the firm's internal identifiers *once*, at ingest — so that a trading blotter, a risk engine and a backtest all join on the same key. The DDS guide's §10.2 `MarketDataTick` topic keyed by symbol is the bus-level expression of this.
- **The golden-source debate.** For positions and cash, the IBOR vs ABOR question (which book is the source of truth) is documented in the investops guide §6.5; for market data the equivalent question is *which price is official* — the answer is usually a controlled pricing hierarchy (primary exchange close → consolidated close → vendor evaluated price → model), with the hierarchy itself governed.
- **Corporate actions are a golden-source domain.** As §4.6 noted, the corporate-action feed must be applied once, correctly, and published to all consumers — the investops guide §4 documents the event lifecycle and messages.

### 9.2 Entitlements and Licensing

Market data is licensed, not bought, and the licensing model drives an entire control discipline. The verified public facts:

- **SIP fees are public and regulated.** The CTA and UTP plans publish their fee schedules and data policies on their websites (ctaplan.com/pricing; utpplan.com data policies and fee schedule) ✅ — the fees are filed with the SEC under the NMS plan process and are subject to regulatory review ✅ (Section 11A plan-approval process, per ctaplan.com and utpplan.com). The 2018 joint report of the SIP operating committees put average annual SIP revenue at **nearly US$400 million over 2007–2017** ✅-secondary (Wikipedia, SIP article, citing the committees' 2018 report) — the SIPs redistribute that revenue to the participant exchanges ✅-secondary (Wikipedia), which is the economic core of the §2.5 governance fight.
- **Direct-feed and vendor terms are not public.** Proprietary exchange feed pricing and vendor redistribution terms are negotiated bilaterally and are generally confidential — any specific number in circulation is press-reported ⚠ and none are asserted here ✅/⚠ (SEC filings occasionally reveal anchor contracts, but the standard commercial terms are not public ⚠).
- **Redistribution rules.** Market-data licenses are per-use: display vs non-display (machine-readable) entitlements, per-user vs per-device fees, professional vs non-professional subscriber classes, and strict redistribution prohibitions — the plan websites publish the vendor/subscriber agreement templates (e.g., the UTP Plan Vendor Agreement and Subscriber Agreement on utpplan.com ✅ — the *existence* of the regime is verified; the specific commercial terms of any given contract are ⚠). Non-professionals typically receive real-time prices at no direct cost through their brokers because the low per-user fees are absorbed ✅ (ctaplan.com: "Most nonprofessionals do not pay fees because the low cost allows for brokerage firms to include real-time prices as part of their service").
- **The compliance burden.** A firm redistributing data (to a client portal, a mobile app, an algorithm, a vendor) must report usage (the UTP plan's NORA reporting application and MISU reporting programme are live examples ✅ per utpplan.com) and pay the corresponding fees — the "data-piracy" enforcement pages on the plan sites (utpplan.com data-piracy reporting) show the stakes ✅.

### 9.3 Data Governance — Lineage, Quality and BCBS 239

Data governance is the management layer that makes the value chain trustworthy:

- **Lineage.** Every price, position and P&L number should trace back to its source feed, transform and timestamp — the auditability requirement that the investops guide §8.4 states for reference data, applied here across the whole market-data estate (which feed, which vendor, which version of the security master produced this mark).
- **Quality.** Validation on ingest (price sanity, symbol validity, duplicate detection), completeness monitoring (the deadline/QoS signals of the DDS guide's §5.5 — a missing tick stream is detected by deadline, not by prayer), and timeliness controls (stale-price checks — the risk engine's "widened VaR until the feed returns" behaviour in the DDS guide's §10.2 is the behavioural consequence).
- **BCBS 239 alignment.** For banks, the Basel Committee's **BCBS 239** — "Principles for effective risk data aggregation and risk reporting" (published January 2013) — requires precisely this discipline at the *risk-data* level: data architecture and IT infrastructure, accuracy and integrity, completeness, timeliness, adaptability ✅-secondary (the principles themselves are standard, published by the BIS; the specific principle list is well-documented — the January 2013 publication date is the commonly cited one ⚠-secondary). The market-data reading of BCBS 239: the risk numbers in §7 must be produced from governed, complete, timely data — the same golden-source and lineage discipline of §9.1, applied to the risk book of record.

---

## 10. The Cymbal Bank Worked Example

### 10.1 The Architecture at a Glance

Cymbal Bank's market-data estate serves four consumer classes at four latency tiers: the **trading floor** (equities, FX, fixed income desks; market making; algos), the **portfolio-management platform** (the IBOR/PMS used by the asset-management arm), the **risk engines** (real-time risk, VaR, stress) and the **analytics/research platform** (tick history, backtesting, data science). The bus mechanics — the DDS domain design, the QoS profile, the security model, the FIX integration boundary — are fully worked in the sibling [DDS Guide](../technology/dds_guide.md) §10 and are **not re-derived here**; this section extends that design with the *consumption* architecture: where the feeds come from, who consumes what, at what latency, under what entitlements.

### 10.2 The Feed-Handler Layer

At the edge sit the **feed handlers** — one per source, co-located where speed matters (cross-ref [Market Making in Singapore](market_making_singapore_guide.md) §7.3 for the co-location engineering):

- **Direct feeds** for the venues Cymbal Bank trades most (the primary equity venues, the FX ECNs) — MBO where the microstructure edge lives, MBP elsewhere (§4.3), with gap detection and recovery on every multicast stream (§5.4).
- **Consolidated feeds** — the CTA/CQ and UTP SIP streams (Tapes A/B/C, §2.4) for the NBBO, the LULD bands and the full protected market; one handler pair per tape family.
- **Vendor feeds** — the LSEG/Refinitiv and Bloomberg enterprise feeds for the cross-asset coverage, news and analytics the exchange feeds cannot provide (§3); the pricing-service files (ICE Data Services evaluations for the bond book, §3.5) ingested end-of-day into the official-pricing path.
- **Reference and event feeds** — the corporate-action and reference-data services (§4.6, §9.1), ingested into the golden source, not into the tick path.

Every handler normalizes onto Cymbal Bank's internal symbology at ingest (§9.1) and publishes onto the bus as the DDS guide's §10.2 topics — `MarketDataTick` (L1), `MarketDataTick` L2 depth variant, `VenueStatus`, `Heartbeat` — with the §10.3 QoS profile (BEST_EFFORT/KEEP_LAST 1 for L1 ticks, RELIABLE/KEEP_LAST 5 for depth, deadline 1s/100ms, EXCLUSIVE ownership with active/standby failover).

### 10.3 The Bus — Extending the DDS Guide's §10

The DDS guide's §10 design (Domain 100, partitions `PROD/EQ`, `PROD/FX`, `PROD/FI`, DDS-Security with PKI-DH authentication, permissions and AES-GCM/AES-GMAC) is the bus. This guide extends it with three consumption-side additions:

- **A second, lower-QoS domain for the analytics tap.** Tick capture must not compete with the trading QoS: a separate tap process (the compliance recorder pattern of DDS §10.2) subscribes on Domain 100 and forwards to the **Kafka lane** (DDS §10.5) — tick history, TCA and backtest data flow through Kafka and the data platform (§8.4), never through the trading domain. The analytics consumers read from the lakehouse/time-series stores, not from the bus.
- **An official-close topic.** A daily `OfficialClose` publication (the controlled close snapshot — official prices, fixings, corporate-action-adjusted state, §6.1) is published once after the close for the PMS, risk EOD runs and fund accounting — a separate, RELIABLE, TRANSIENT topic that the trading domain never sees.
- **The NBBO as a first-class topic.** The SIP-derived NBBO is republished as its own topic (`NBBOUpdate`, keyed by symbol) so that pre-trade checks, SOR and best-execution analytics all consume one consistent NBBO stream rather than each computing it (§5.1).

### 10.4 The Consumers — Trading, PMS, Risk, Analytics

| Consumer | Subscribes to | Consumes | Cadence |
| --- | --- | --- | --- |
| **EMS / trading blotter** | `MarketDataTick` L1, `OrderState`, `NBBOUpdate` | display prices, marketability, pre-trade checks | tick |
| **SOR and algos** | L2 depth, `MarketDataTick`, `OrderState`, `VenueStatus` | aggregated books, benchmarks, participation | tick |
| **Market-making desks** | L2 depth (RELIABLE), direct-feed MBO where traded | book reconstruction, re-pricing | tick (microsecond-class) |
| **PMS / IBOR** | `MarketDataTick` (throttled), `OfficialClose`, corporate-action-adjusted golden source | marks, valuations, P&L, NAV inputs | intraday + official close |
| **Risk engines** | `MarketDataTick`, `OrderState`, `PositionUpdate`, `VenueStatus`, `NBBOUpdate`, halt/LULD state | real-time exposure, limit checks, scenario marks | tick / throttled |
| **Analytics / research** | Kafka lane (tick tap), historical stores, reference data | backtests, TCA, features, ML training | batch / replay |

### 10.5 The Latency Tiers

The estate is explicitly tiered so that the cost of the fastest tier is contained:

- **Tier 0 — co-located direct consumption (microseconds).** The market-making and HFT-adjacent desks consume the direct MBO feeds in the venue data centre, through the feed handlers, with kernel-bypass networking (cross-ref [Market Making in Singapore](market_making_singapore_guide.md) §7.3) — the bus QoS for this tier is the DDS guide's §10.3 profile.
- **Tier 1 — the trading floor (milliseconds).** The EMS, SOR and algos consume the bus in the primary data centre.
- **Tier 2 — risk and PMS (seconds to minutes).** The risk engines and the PMS subscribe on the same bus but with throttled consumption and their own aggregation — they do not need every tick, they need the state to be current (the DEADLINE QoS turns "is the feed current?" into a measured signal, DDS §5.5).
- **Tier 3 — analytics (batch).** The data platform consumes the Kafka lane and the official-close publications; nothing on this tier touches the real-time bus.

### 10.6 Entitlements

Cymbal Bank's market-data entitlement model follows the licensing regime of §9.2: a central **entitlement service** (the DDS-Security permissions of DDS §10.4) maps every consumer to the feeds it is licensed for — the market-making desk has the direct-feed entitlements, the PMS has the vendor and official-pricing entitlements, the analytics platform has the historical/redistribution entitlements (including the client-facing reporting entitlements). Usage reporting feeds the plan reporting obligations (the NORA/MISU-style reporting of §9.2) and the vendor audits. The design rule: **a consumer cannot subscribe to a topic it is not entitled to** — enforced at the bus (DDS permissions) and at the data platform (row/column-level entitlements on the lakehouse), so the licensing position and the technical access are the same control.

---

## 11. The Claims Audit

### 11.1 The Claims Audit — Verified, Flagged, Rejected

Every key factual claim in this guide, its status and its source ✅⚠❌:

| # | Claim | Status | Source |
| --- | --- | --- | --- |
| 1 | US consolidated tape: Tape A (NYSE-listed), Tape B (listed on exchanges other than NYSE and Nasdaq), Tape C (Nasdaq-listed) | ✅ | SEC Release 34-90610 (sec.gov/files/rules/final/2020/34-90610.pdf) |
| 2 | Three equity data plans: CTA Plan (Tapes A/B trades), CQ Plan (Tape A/B quotes), UTP Plan (Tape C) | ✅ | SEC Release 34-90610 |
| 3 | CTA: NYSE LLC Administrator of Network A, NYSE American Administrator of Network B; CTS/CQS streams; since the late 1970s | ✅ | ctaplan.com |
| 4 | UTP: Nasdaq the Administrator; Tape C; UQDF/UTDF feeds; NBBO dissemination | ✅ | utpplan.com |
| 5 | SIP publishes NBBO, LULD price bands, short-sale restrictions, regulatory halts | ✅ | ctaplan.com; utpplan.com |
| 6 | SIP metrics: CTA 99.98% availability, capacity 2.7M quote msgs/100ms, median latency <20µs; UTP 99.98% availability, 2,700,700 quote / 2,734,100 trade msgs/100ms, median ~12.7–16.3µs, 100% availability in 53 of last 63 quarters | ✅ | ctaplan.com; utpplan.com (as of Q2 2026) |
| 7 | Reg NMS adopted 2005 (final release June 2005); Rule 611 order protection rule (17 CFR 242.611) | ✅ | sec.gov Regulation NMS page; ecfr.gov §242.611 |
| 8 | Rule 602: exchanges may not prohibit/limit vendor redistribution of consolidated data | ✅ | SEC Rule 611 final-rule PDF (34-90610 family) |
| 9 | 2020 Market Data Infrastructure Rule: Release 34-90610, File S7-03-20, December 2020, effective 8 June 2021; competing consolidators (Rule 614, Form CC), five levels of depth + odd lots, round-lot reform | ✅ | SEC 34-90610; Sidley alert |
| 10 | D.C. Circuit upheld the 2020 rules in Nasdaq v. SEC, No. 21-1100, 24 May 2022 | ✅ | Sidley alert (sidley.com); Wikipedia (secondary) |
| 11 | NMS Plan Governance Order: Release 34-88827, May 2020 — single-plan consolidation, ≤2 votes per exchange group, one-third non-SRO committee representation, independent administrator | ✅ | Sidley alert |
| 12 | NYSE v. SEC, No. 21-1169 — challenge to the Governance Order, argued 24 March 2022 | ✅ | Sidley alert |
| 13 | D.C. Circuit partially vacated the SEC's SIP-governance/committee-seats order | ✅/⚠ | MLex headline (vacatur ✅; date and scope ⚠ not re-verified) |
| 14 | CT Plan: unified successor to CTA/CQ + UTP; per SEC 2023 order (34-101672); launch Q2 2027 (April 2027); independent administrator DataCT; RFP selection by Q1 2026; fee schedule filed Nov 2025; no feed-format changes; 15-min delayed data remains free | ✅ | thectplanllc.com (home + FAQ); consolidatedtape.com; utpplan.com ("New CT Plan" link) |
| 15 | SIPs sought temporary odd-lot-depth exemption (Release 34-101070 amendments), request dated 9 July 2025 | ✅ | utpplan.com odd-lot page |
| 16 | EU CTP framework: MiFIR review (Regulation (EU) 2024/791); one CTP per asset class; five-year mandates; ESMA selects/authorises/supervises | ✅ | esma.europa.eu CTP page |
| 17 | EU equity CTP: EuroCTP selected 19 December 2025; joint venture, Netherlands, 15 exchange groups; authorised as EuroCTP B.V. 27 July 2026 | ✅ | ESMA press releases (19/12/2025; 27/07/2026) |
| 18 | EU bond CTP: Ediphy (fairCT) selected 3 July 2025; OTC derivatives CTP: Etrading Software (Netherlands) B.V. selected 6 July 2026 | ✅ | esma.europa.eu CTP page |
| 19 | CTP technical standards adopted June 2025, published OJ 3 Nov 2025; first equity data-contributor list 10 July 2025; ISO 20022 CTP messages | ✅ | esma.europa.eu CTP page |
| 20 | LSEG all-share acquisition of Refinitiv completed 29 January 2021, ~US$27bn | ✅ | LSEG press release (lseg.com); The TRADE (29 Jan 2021) |
| 21 | ICE/Interactive Data: announced 26 October 2015; completed December 2015; ~US$5.2bn (~US$3.65bn cash + ~6.47M shares valued ~US$1.55bn) | ✅ | ir.theice.com press releases; SEC Form 8-K exhibit |
| 22 | FactSet FY2025 (ended 31 Aug 2025): revenue US$2,321.7M (+5.4%); ASV US$2,405.6M; organic ASV US$2,370.9M (+5.7%); EPS US$15.55 GAAP / US$16.98 adj.; FY2026 guidance US$2,423–2,448M | ✅ | investor.factset.com (18 Sep 2025) |
| 23 | Bloomberg terminal count ~350,000 (marketing) to 355,000+ (secondary 2025) | ⚠ | bloomberg.com marketing copy; tradersdna.com (secondary) |
| 24 | Greeks: delta (spot sensitivity), gamma (delta sensitivity), theta (time), vega (volatility), rho (rate); first/second-order derivatives | ✅ | Investopedia (Greeks); Wikipedia (Greeks (finance)) |
| 25 | SIP revenues ~US$400M/yr average 2007–2017 per 2018 SIP committees' joint report; SIPs redistribute revenue to participants | ⚠-secondary | Wikipedia (SIP article) citing the committees' 2018 report |
| 26 | SIP fee schedules public on plan sites (ctaplan.com/pricing; UTP Data Policies); vendor/subscriber agreement templates public (UTP Vendor Agreement, Subscriber Agreement); NORA/MISU usage reporting live | ✅ | ctaplan.com; utpplan.com |
| 27 | Non-professionals typically receive real-time prices at no direct cost; brokers absorb the low per-user fees | ✅ | ctaplan.com |
| 28 | Direct-feed and vendor commercial terms generally not public | ⚠ | industry practice — no public source |
| 29 | MBP vs MBO feed taxonomy; kdb+/q, InfluxDB, TimescaleDB, Parquet/Delta roles | ⚠-secondary | industry-standard usage; product sites — not re-verified live |
| 30 | BCBS 239 principles (risk data aggregation/reporting) | ⚠-secondary | BIS publication (Jan 2013) — not re-verified live this pass |
| 31 | Vendor market shares | ⚠ | press estimates — not asserted |

---

## 12. What Could Not Be Verified, the Glossary, Cross-References and the Closing Summary

### 12.1 What Could Not Be Verified

**The discipline, restated.** A claim in this guide is ✅ only when it was verified this pass against a named source — the Claims Audit's ✅ column (§11.1) is the positive counterpart of this section; a claim is ⚠ when it rests on a secondary source, an estimate, marketing copy, or a source that could not be re-fetched; and nothing that could not be verified was asserted as fact. Every ⚠ raised in §1–§11 is collected here, with the reason and with what would settle it.

| # | Item | Flagged where | Why it could not be verified | What would settle it |
| --- | --- | --- | --- | --- |
| 1 | Bloomberg terminal subscriber counts (~350,000 marketing copy; 355,000+ secondary 2025 estimate) | §1.2, §3.2, §3.6, §11.1 #23 | Bloomberg is private; no audited subscriber figure exists in public filings; both figures are estimates (marketing / aggregator) | An audited Bloomberg disclosure — unlikely; treat as estimate permanently |
| 2 | SIP average annual revenue ~US$400M over 2007–2017 | §1.2, §9.2, §11.1 #25 | Source is the SIP operating committees' 2018 joint report as cited by Wikipedia; the original report was not retrieved this pass | The committees' 2018 joint report itself, or the SEC filing quoting it |
| 3 | Partial vacatur of the SEC's May 2020 NMS Plan Governance Order | §2.5, §11.1 #13 | The vacatur is verified via the MLex headline; the decision's date and full scope were not re-verified this pass | The D.C. Circuit's opinion in NYSE v. SEC, No. 21-1169 |
| 4 | EuroCTP live-stream go-live date | §2.7 | ESMA's selection (19 Dec 2025) and authorisation (27 Jul 2026) are ✅; the operational go-live of the live stream was not re-verified | ESMA / EuroCTP operational announcements |
| 5 | Direct-feed and vendor commercial terms | §1.2, §5.3, §9.2, §11.1 #28 | Negotiated bilaterally and confidential; there is no public price book for proprietary feeds | Signed contracts, or SEC-filed anchor agreements where they exist |
| 6 | Vendor market shares | §3.1, §3.6, §11.1 #31 | Rankings are press estimates; vendors publish no consistent market-share metric | A consistent industry survey; not re-verified this pass |
| 7 | LSEG revenue split between exchange and data businesses | §3.3 | Not re-verified this pass | LSEG annual-report segment disclosures |
| 8 | ICE's 2022 acquisition of Black Knight's index business | §3.5 | Secondary coverage only; not re-verified this pass | ICE press release or Form 8-K |
| 9 | SIAC as the historical CTA/CQ infrastructure operator | §2.3 | Wikipedia-secondary; not re-verified against a primary source | NYSE / SIAC historical records |
| 10 | MBP vs MBO as a canonical taxonomy | §4.3, §11.1 #29 | Standard industry usage documented across exchange feed specifications, but no single primary source defines the pairing | A named exchange feed specification (e.g., the UTP depth spec) |
| 11 | Index-administrator and methodology facts | §4.4 | Common knowledge only; no specific index levels or methodologies are asserted, so there is nothing concrete to verify | Index-administrator methodology documents (only if specifics are ever added) |
| 12 | FX market structure (no regulated consolidated tape; vendor-mediated distribution) | §4.5 | Industry-standard description; no primary regulator source was retrieved this pass | ESMA / ECB / FCA market-structure documentation |
| 13 | Tick/time-series product roles (kdb+/q, InfluxDB, TimescaleDB, Parquet/Delta/Iceberg, Arrow) | §8.4, §11.1 #29 | Product existence and role are well documented; no version, benchmark or vendor claims are asserted and none were re-verified | Product documentation and benchmarks (only if specific claims are added) |
| 14 | BCBS 239 publication details (January 2013) | §9.3, §11.1 #30 | The principles are standard and published by the BIS; the January 2013 date is the commonly cited one but the document was not re-fetched this pass | The BIS BCBS 239 page |
| 15 | CT Plan proposed fee schedule and its approval | §2.6 | The Operating Committee was to file a fee schedule by November 2025 subject to SEC approval; the filing and its outcome were not re-verified | SEC approval order; thectplanllc.com fee page |
| 16 | Plan-site metrics and rosters currency (SIP availability/latency/capacity, participant lists) | §2.3, §11.1 #6 | Verified as published on ctaplan.com / utpplan.com as of this pass (Q2 2026 metrics); the sites are living documents | The plan websites themselves |
| 17 | Non-professional subscriber fee treatment | §9.2, §11.1 #27 | Verified to ctaplan.com's own statement; the broker-absorption practice is as the plan describes it | ctaplan.com (verified); broker disclosures |
| 18 | Commercial terms of individual contracts under the public agreement templates | §9.2, §11.1 #26 | The *existence* of the regime (templates, NORA/MISU reporting) is ✅; negotiated terms under it are not public | Signed agreements (confidential) |

**What this guide deliberately did NOT claim.** The negative space — everything a reader might expect a market-data guide to assert that this one refuses to:

- No specific index levels, index methodologies or FX fixing values.
- No exchange, SIP or vendor price lists beyond the public plan fee schedules cited in §9.2.
- No latency or throughput benchmarks of any product or venue beyond the SIP metrics the plan sites publish (§2.3).
- No Bloomberg financials, and no subscriber count asserted as fact (§3.2).
- No kdb+/q, database or vendor performance numbers (§8.4).
- No vendor market-share ranking asserted (§3.1).
- No prediction of the CT Plan's or EuroCTP's operational dates beyond what the plan sites and ESMA have published (§2.6, §2.7).

Where a number was needed and could not be verified, the claim was downgraded to ⚠ or removed — never estimated.

**The ❌ column.** This pass rejected no claims outright: every claim that could not be verified was downgraded to ⚠ rather than asserted. Two cross-referenced facts are ✅ in their home guides and are deliberately **not re-derived here** — FactSet's purchase of CUSIP Global Services (~US$1.925bn, completed 1 March 2022; verified in the sibling [Investment Portfolio Operations](investment_portfolio_operations_guide.md) guide §13.1) and the corporate-action message mappings (verified in the same guide §4.4).

### 12.2 The Glossary

| Term | Meaning | Where it matters in this guide |
| --- | --- | --- |
| ABOR (Accounting Book of Record) | The book of record maintained by the accounting / fund-administration side — the official positions, cash and valuations | §6.1 (vs IBOR) |
| ASV (Annual Subscription Value) | FactSet's forward-looking measure of subscription revenue for the next twelve months | §3.4 |
| ATS (Alternative Trading System) | A non-exchange trading system registered under Reg ATS | §2.1 |
| BCBS 239 | Basel Committee principles for effective risk-data aggregation and risk reporting | §9.3 |
| Best execution | The regulatory duty to seek the most favourable terms reasonably available — benchmarked on the NBBO and tick data | §5.1, §5.5 |
| B-PIPE | Bloomberg's enterprise real-time data feed product | §3.2 |
| Circuit breaker | A market-wide or single-stock trading halt mechanism (with LULD bands) | §7.1 |
| Closing price | The official end-of-day price per venue — the primary input to official marks | §6.1 |
| Co-location | Placing the firm's servers in the venue's data centre for lowest latency | §5.4 |
| Competing consolidator | An entity registered under new Rule 614 (2020 rule) allowed to distribute consolidated market data alongside the SIPs | §2.5 |
| Consolidated feed / consolidated data | The NMS data combined by the SIPs from all venues' contributions — the regulatory picture of the market | §2.2–§2.6, §5.3 |
| Consolidated tape | The combined trade and quote record for a market — the US tapes A/B/C; the EU's new CTP tapes | §2.2, §2.7 |
| Corporate action | An event (dividend, split, merger) that changes what a position means | §4.6 |
| CQ Plan (Consolidated Quotation Plan) | The NMS plan for Tape A/B quotation data | §2.3 |
| CQS (Consolidated Quote System) | The SIP quote stream for Tapes A and B | §2.4 |
| CTA Plan (Consolidated Tape Association) | The NMS plan for Tape A/B trade and quote data | §2.3 |
| CTP (Consolidated Tape Provider) | The EU's single-per-asset-class tape operator selected by ESMA | §2.7 |
| CTS (Consolidated Tape System) | The SIP trade stream for Tapes A and B | §2.4 |
| DDS (Data Distribution Service) | The OMG real-time publish-subscribe standard — the bus of the worked example | §10, DDS guide |
| Delta / Gamma / Theta / Vega / Rho | The Greeks — option-value sensitivities to spot, delta, time, volatility and rate | §7.3 |
| Direct feed (proprietary feed) | A venue's own feed, published ahead of the SIP with depth and order-level content | §5.3, §10.2 |
| ECN (Electronic Communication Network) | An electronic order-matching system (FX ECNs: EBS, Refinitiv Matching) | §2.1, §4.5 |
| EMS (Execution Management System) | The front-office execution tool: blotter, venue connectivity, execution analytics | §5.1 |
| Entitlement | The licensed right to consume a data product, enforced technically | §9.2, §10.6 |
| Evaluated price | A pricing-service estimate of a security's value (e.g., bond evaluations) | §3.5, §6.1 |
| Feature store | The serving layer that makes engineered market-data features available to ML inference | §8.3 |
| Feed handler | The ingest component that receives, gap-checks, normalizes and republishes a feed | §10.2 |
| FIX (Financial Information eXchange) | The messaging protocol of order flow and, in 35=V/W/X, of market data | §5.2, FIX guide |
| Fixing | A daily official reference rate (e.g., ECB reference rates, WM/Refinitiv fixings) | §4.5 |
| Flash P&L | Intraday P&L computed from live marks, before the official close | §6.2 |
| Fund accounting | The back-office valuation and NAV production that consumes the official close | §6.1, §10.3 |
| GLEIF / LEI | The Global Legal Entity Identifier Foundation and the legal-entity identifier it issues | §9.1 |
| Golden source | The firm's single authoritative copy of a data domain | §9.1 |
| Heartbeat | A periodic liveness signal on a feed — the basis of staleness detection | §10.2, DDS guide |
| Historical simulation | The VaR methodology that replays historical return series (a §7.2 data consumer) | §7.2 |
| IBOR (Investment Book of Record) | The front-office book of record — positions and valuations as the desk sees them | §6.1 |
| Kafka lane | The streaming / analytics distribution path fed by the tick tap, separate from the trading bus | §8.2, §10.3 |
| Kernel bypass | Low-latency networking that avoids the OS kernel's copying (the co-location tier) | §5.4, §10.5 |
| Kill switch | The breach-response control that throttles or halts order flow (§7.4's action side) | §7.4 |
| Lakehouse | The object-storage + table-format architecture (Delta/Iceberg + Parquet) for analytics data | §8.4 |
| Last trade | The most recent execution price for a security — a Level 1 element and mark input | §4.1 |
| Level 1 | Top-of-book data: best bid / offer and last trade | §4.1 |
| Level 2 | Depth-of-book data: multiple price levels | §4.2 |
| LULD (Limit Up/Limit Down) | The regulatory price-band mechanism computed from the NBBO | §2.2, §7.1 |
| Market-by-order (MBO) | Depth feed carrying every order with its order ID | §4.3 |
| Market-by-price (MBP) | Depth feed carrying aggregated price levels | §4.3 |
| Marketability check | The pre-trade test that an order is executable at the current NBBO | §5.1 |
| Multicast | One-to-many UDP delivery used for tick distribution — lossy by design, gap-handled by feed handlers | §5.4 |
| NAV (Net Asset Value) | The fund's per-share valuation produced from official marks and fixings | §6.1 |
| NBBO (National Best Bid and Offer) | The best protected bid / offer across all venues — the regulatory price | §2.2, §5.1 |
| NMS plan | A joint-industry plan filed with the SEC under Section 11A (CTA/CQ, UTP, CT) | §2.3 |
| Non-display use | Machine-readable consumption of data without a screen (algos, risk engines) | §9.2 |
| NORA / MISU | UTP-plan usage-reporting applications for vendor and subscriber compliance | §9.2 |
| Odd lot / round lot | Sub-100-share orders vs the standard 100-share unit (and the 2020 round-lot reform) | §2.5 |
| Odd-lot depth | Depth content from odd-lot quotations — added to consolidated data by the 2020 rule | §2.5 |
| Official close | The controlled end-of-day price snapshot used for official valuations | §6.1, §10.3 |
| Official pricing path | The separate end-of-day vendor files, fixings and closing levels feeding the PMS and fund accounting | §6.1, §10.3 |
| OMS (Order Management System) | The system holding order lifecycle and compliance state | §5.1 |
| OPRA (Options Price Reporting Authority) | The plan for US exchange-traded options data | §2.4 |
| PMS (Portfolio Management System) | The buy-side portfolio platform (Aladdin, Charles River, AIM, SimCorp, …) | §6.1 |
| Point-in-time data | Data as it existed at a historical moment — the anti-look-ahead-bias requirement | §8.2 |
| Protected quotation | A quotation entitled to Rule 611 order-protection — the NBBO's building block | §2.2 |
| QoS (Quality of Service) | DDS delivery policies (BEST_EFFORT / RELIABLE, KEEP_LAST, DEADLINE) | §10.3, DDS guide |
| Redistribution | Passing licensed data to another consumer (client portal, app, algo) under reporting obligations | §9.2 |
| Reg NMS | The SEC's 2005 Regulation National Market System | §2.2 |
| Replay | Re-running recorded ticks through a simulation harness for backtesting | §8.2 |
| Security master | The instrument reference-data record (identifiers, terms, calendars) | §9.1 |
| SIP (Securities Information Processor) | The plan processor that consolidates and disseminates NMS data | §2.3 |
| Snapshot | A full-state publication used to recover a stream after a gap (multicast recovery) | §5.4 |
| SOR (Smart Order Router) | Routing logic that consumes every venue's book to choose destinations | §5.1 |
| Spread | The bid-ask difference — the transaction-cost component estimated from L1/L2 history | §8.2 |
| Stress scenario | A hypothetical or historical shock applied to current levels for stress testing | §7.2 |
| Symbology | The identifier schemes (venue symbols, vendor IDs, ISIN/CUSIP/SEDOL/LEI) normalized at ingest | §9.1 |
| Tape A / Tape B / Tape C | The three US consolidated tapes: NYSE-listed / other-than-NYSE-and-Nasdaq / Nasdaq-listed | §2.4 |
| TCA (Transaction Cost Analysis) | Execution-quality measurement built on tick and order data | §5.5 |
| Tick | A single market-data event: a quote update or a trade | §5.5 |
| Tick-capture store | The high-throughput append-only storage tier for tick history | §5.5, §8.4 |
| Time-series database | A database optimized for time-ordered, range-queried data | §8.4 |
| UQDF (UTP Quotation Data Feed) | The SIP quote stream for Tape C | §2.4 |
| UTDF (UTP Trade Data Feed) | The SIP trade stream for Tape C | §2.4 |
| UTP Plan (Unlisted Trading Privileges Plan) | The NMS plan for Nasdaq-listed (Tape C) data | §2.3 |
| VaR (Value at Risk) | The standard quantile-based risk measure | §7.2 |
| VenueStatus | The bus topic carrying trading halts, LULD state and venue availability | §10.2, §7.1 |
| Working order | An order actively seeking execution — correlated with live prices on the blotter | §5.1 |

### 12.3 Cross-References

This guide deliberately re-derives nothing: each sibling owns its domain and this guide cites it. Repository convention — sibling guides in `banking/` are plain filenames; guides in `technology/` are prefixed `../technology/`.

| Guide | Owns (do not re-derive) | Cross-referenced in |
| --- | --- | --- |
| [FIX Protocol](fix_protocol_guide.md) | The protocol itself: market-data messages 35=V/W/X/Y, quote messages, tag semantics (§6); binary protocols such as CME MDP 3.0 (§8) | §4.1, §4.3, §5.2 |
| [Investment Portfolio Operations](investment_portfolio_operations_guide.md) | Trade lifecycle (§2, §3), IBOR vs ABOR (§6.5), corporate actions and MT564/MT566 / seev.xxx (§4), reference data and identifiers (§8), fair-value hierarchy (§6.4), Brinson attribution (§7), PMS landscape (§10), CUSIP Global Services (§13.1) | §3.4, §4.6, §6.1, §9.1 |
| [Enterprise Risk Management](enterprise_risk_management_guide.md) | VaR methodology, model validation, stress-testing and risk-appetite governance | §7.2, §7.4 |
| [Market Making in Singapore](market_making_singapore_guide.md) | Co-location engineering, low-latency market making, SGX Reach | §5.4, §10.2, §10.5 |
| [Capital Markets Architecture](capital_markets_architecture_guide.md) | Front / middle / back architecture and the market-data plumbing across it | §1, §10 |
| [Financial Trading Order Infrastructure](financial_trading_order_infrastructure.md) | OMS/EMS roles, order routing, SOR, matching engines | §5.1 |
| [Kafka Guide](kafka_guide.md) | The streaming / analytics lane | §8.2, §10.3 |
| [DDS Guide](../technology/dds_guide.md) | The real-time bus: domain design (§10.2), QoS (§10.3), DDS-Security (§10.4), the FIX integration boundary (§10.5), deadline semantics (§5) — §10 of that guide is the Cymbal Bank worked example this guide extends | §5.1, §5.4, §8.2, §10.1–§10.6 |
| [Large-Scale Web Data Acquisition](../technology/large_scale_web_data_acquisition_guide.md) | Data-acquisition engineering: crawling, ingestion, cleaning, deduplication, versioning | §8.1, §8.4 |
| AI/LLM cluster ([../technology/ai_llm/](../technology/ai_llm/)) | RAG guides (`rag/`), the LLM engineering guides, `closed_loop_data_engineering_guide.md` | §8.3 |

### 12.4 The Closing Summary

**The chain, verified.** Market data flows **venue → feed → vendor → platform → decision**, and this guide verified the production side against primary sources: the three US equity data plans and their SIPs (CTA/CQ and UTP; Tapes A, B and C), Reg NMS and the NBBO, the 2020 Market Data Infrastructure Rule (upheld by the D.C. Circuit in May 2022), the partial vacatur of the 2020 governance order, and the modernization now underway — the CT Plan with DataCT as its independent administrator, disseminating from April 2027. Across the Atlantic, the EU's first consolidated tape is being built under ESMA: EuroCTP for equities (selected 19 December 2025, authorised 27 July 2026), Ediphy's fairCT for bonds and Etrading Software for OTC derivatives. The vendor layer was verified to its own primary sources: LSEG's ~US$27bn acquisition of Refinitiv (completed 29 January 2021), ICE's ~US$5.2bn acquisition of Interactive Data (completed December 2015) and FactSet's FY2025 results.

**The four platform classes.** Trading consumes the present tick — direct feeds, multicast, co-location, tick capture — at microseconds to milliseconds; portfolio management consumes intraday marks and the official close; risk consumes near-real-time state for exposure, VaR, the Greeks and limit monitoring; analytics consumes everything, forever, in batch. Each consumption pattern was mapped in §5–§8 and cross-referenced to the siblings that own the protocol mechanics, the operations lifecycle, the risk governance and the market-making engineering — nothing re-derived.

**The data-management layer.** Underneath all four classes sits the layer that makes the chain trustworthy: the golden source and normalized symbology (§9.1); the entitlement and licensing regime — public, regulated SIP fees versus negotiated, confidential direct-feed and vendor terms, with redistribution reporting (§9.2); and governance aligned to BCBS 239 — lineage, quality, completeness, timeliness (§9.3).

**The worked example.** Cymbal Bank's estate makes the abstraction concrete: feed handlers normalizing every source at the edge, the DDS bus extended with the analytics tap, the official-close topic and the first-class NBBO topic, four consumer classes at four latency tiers, and entitlements enforced at the bus and the lakehouse — a consumer cannot subscribe to what it is not licensed for (§10).

**The four classes at a glance.**

| Class | Cadence | Primary market-data inputs | Key sections |
| --- | --- | --- | --- |
| Trading (OMS/EMS, SOR, algos, market making) | microseconds–milliseconds | direct feeds (MBO/MBP), SIP consolidated data, NBBO, depth, venue status | §5, §10.4–§10.5 |
| Portfolio management (PMS/IBOR) | intraday + official close | marks, closing prices, fixings, corporate-action-adjusted golden source | §6, §10.4 |
| Risk (real-time, VaR, stress, limits) | tick / throttled | ticks, positions, halt/LULD state, historical series, scenarios | §7, §10.4 |
| Analytics (research, backtesting, data science) | batch / replay | full tick history, point-in-time reference data, time-series stores | §8, §10.3–§10.4 |

**The discipline.** Every claim was verified ✅ against a named source or flagged ⚠ honestly; the boundaries are collected in §12.1; the rejected column is empty; and no date, number or fact was invented.

The result is a map of the whole chain — from the exchange's tick to the trader's screen, the risk engine's VaR and the quant's backtest — and of the layer that keeps it honest. For the firm that consumes it well — governed, entitled, complete and timely — market data is not a cost line to be managed: it is the market's pulse.
