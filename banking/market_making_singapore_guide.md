# Market Making and Electronic Liquidity in Singapore: A Comprehensive Guide

**The Proprietary-Trading and Electronic-Market-Making Landscape in Singapore — Optiver as the Reference Firm, the Global Peer Set (Flow Traders, IMC, Jane Street, Jump, SIG, Tower, HRT, XTX, DRW), the Home-Grown and Regional Firms, the SGX Market-Making Schemes, the MAS Regulatory Overlay, the Low-Latency and Clearing Infrastructure, and a Cymbal Bank Prime-Brokerage Worked Example**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Banking Domain / Institutional Investment & Capital Markets — the proprietary trading and electronic market-making landscape in Singapore: Optiver as the reference market maker, the global electronic-liquidity peer set and their Singapore footprints, the home-grown and regional prop shops, the SGX designated-market-maker and incentive schemes, the SGX/CDP/SGX-DC clearing and co-location infrastructure, the MAS regulatory overlay (condensed from the sibling hedge-funds guide), the Singapore talent market, and the Cymbal Bank institutional lens
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** August 2026
> **Companion guides (sibling, same folder — the capital-markets cluster):** [Citadel LLC](citadel_llc_guide.md) (the prime-brokerage worked-example format and the talent-section conventions — cross-ref §10, §11) · [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) (the MAS fund-management and CMS-licence regime — condensed and cross-referenced in §9, not re-derived) · [FIX Protocol](fix_protocol_guide.md) (the electronic-trading messaging standard — cross-ref §3, §8) · [Financial Trading Order Infrastructure](financial_trading_order_infrastructure.md) (OMS/EMS, order routing, exchange infrastructure) · [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) (the Cymbal Bank persona conventions and the Singapore regulatory overlay)
> **Companion guides (technology/, prefix `../technology/`):** [Low-Latency C++ Development](../technology/low_latency_cpp_development_guide.md) (the C++ latency engineering behind market-making systems — cross-ref §3, §8) · [DDS Guide](../technology/dds_guide.md) (the data-centric publish-subscribe middleware used for real-time market-data distribution — cross-ref §3, §8) · [Zero Downtime System Design](../technology/zero_downtime_system_design_guide.md) (always-on trading estates) · [Quantitative Developer Skillset](../technology/quantitative_developer_skillset_guide.md) (the quant-research skill stack)

---

**How to use this guide:** Section 1 is the overview — the short answer and the key-facts table. Section 2 is the reference firm — Optiver's history, financials, the market-making model, and the Singapore office. Section 3 is the SGX market-making context — the exchange's designated-market-maker schemes and incentive programs. Section 4 is the global peer landscape — nine verified firms and their Singapore footprints. Section 5 is the home-grown and regional landscape. Section 6 is the comparison table. Section 7 is the infrastructure — SGX, CDP, SGX-DC, co-location and the low-latency angle (cross-referenced, not re-derived). Section 8 is the MAS regulatory context (condensed and cross-referenced). Section 9 is the talent market. Section 10 is the Cymbal Bank worked example — a market maker as prime-brokerage client, extending the Citadel-guide conventions rather than re-deriving them. Section 11 is the claims audit (✅/⚠/❌), with §11.4 "What Could Not Be Verified". Section 12 is the glossary. Section 13 is cross-references. Section 14 is the closing summary. Cross-references follow the repository convention: sibling guides in `banking/` are plain filenames; guides in `technology/` are prefixed `../technology/`. **Integrity convention:** ✅ = verified this pass against a primary or cited source (source named in §11); ⚠ = flagged/unverified — press estimate, contested, or not re-verified live; ❌ = refuted or rejected. Nothing in this guide was invented; figures that could not be re-verified are marked ⚠ and listed again in §11.4.

---

## Table of Contents

1. [The Overview](#1-the-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [What This Guide Covers](#12-what-this-guide-covers)
   - 1.3 [The Key-Facts Table](#13-the-key-facts-table)
   - 1.4 [Why This Matters to a Bank](#14-why-this-matters-to-a-bank)
2. [The Reference Firm — Optiver: Profile, History and the Market-Making Model](#2-the-reference-firm--optiver-profile-history-and-the-market-making-model)
   - 2.1 [The Founding and the Name](#21-the-founding-and-the-name)
   - 2.2 [The Firm Today — Scale, Products and Financials](#22-the-firm-today--scale-products-and-financials)
   - 2.3 [The Market-Making Model — How It Works](#23-the-market-making-model--how-it-works)
   - 2.4 [The Asia-Pacific Footprint and the Singapore Office](#24-the-asia-pacific-footprint-and-the-singapore-office)
   - 2.5 [The SGX Role](#25-the-sgx-role)
   - 2.6 [The Regulatory Record](#26-the-regulatory-record)
3. [The SGX Market-Making Context — Schemes, Obligations and Incentives](#3-the-sgx-market-making-context--schemes-obligations-and-incentives)
   - 3.1 [The Designated Market-Maker Regime (SGX-ST Rules)](#31-the-designated-market-maker-regime-sgx-st-rules)
   - 3.2 [The ETF DMM Scheme and the 2025 Streamlining](#32-the-etf-dmm-scheme-and-the-2025-streamlining)
   - 3.3 [The Derivatives Market-Making Programmes](#33-the-derivatives-market-making-programmes)
   - 3.4 [What Is Public — and What Is Not](#34-what-is-public--and-what-is-not)
4. [The Peer Landscape — The Global Electronic Market Makers](#4-the-peer-landscape--the-global-electronic-market-makers)
   - 4.1 [Flow Traders](#41-flow-traders)
   - 4.2 [IMC](#42-imc)
   - 4.3 [Jane Street](#43-jane-street)
   - 4.4 [Jump Trading](#44-jump-trading)
   - 4.5 [SIG — Susquehanna International Group](#45-sig--susquehanna-international-group)
   - 4.6 [Tower Research Capital](#46-tower-research-capital)
   - 4.7 [Hudson River Trading (HRT)](#47-hudson-river-trading-hrt)
   - 4.8 [XTX Markets](#48-xtx-markets)
   - 4.9 [DRW](#49-drw)
5. [The Peer Landscape — Home-Grown and Regional Firms](#5-the-peer-landscape--home-grown-and-regional-firms)
   - 5.1 [AlphaGrep](#51-alphagrep)
   - 5.2 [QCP Capital](#52-qcp-capital)
   - 5.3 [Other Regional Names](#53-other-regional-names)
6. [The Comparison Table — Firms × Products × Regions × SG Footprint](#6-the-comparison-table--firms--products--regions--sg-footprint)
7. [The Infrastructure — SGX, CDP, SGX-DC and the Co-Location Angle](#7-the-infrastructure--sgx-cdp-sgx-dc-and-the-co-location-angle)
   - 7.1 [The Exchange Group — SGX, SGX-ST, SGX-DT, CDP and SGX-DC](#71-the-exchange-group--sgx-sgx-st-sgx-dt-cdp-and-sgx-dc)
   - 7.2 [Reach and the SGX Data Centre](#72-reach-and-the-sgx-data-centre)
   - 7.3 [The Low-Latency Angle (Cross-Referenced)](#73-the-low-latency-angle-cross-referenced)
   - 7.4 [Clearing and Settlement — Central Clearing via SGX-DC](#74-clearing-and-settlement--central-clearing-via-sgx-dc)
8. [The MAS Regulatory Context (Condensed)](#8-the-mas-regulatory-context-condensed)
   - 8.1 [The CMS Licence and Dealing in Capital-Markets Products](#81-the-cms-licence-and-dealing-in-capital-markets-products)
   - 8.2 [The Market-Conduct Overlay](#82-the-market-conduct-overlay)
   - 8.3 [Where the Full Regime Lives](#83-where-the-full-regime-lives)
9. [The Talent Market in Singapore](#9-the-talent-market-in-singapore)
   - 9.1 [The Hiring Landscape](#91-the-hiring-landscape)
   - 9.2 [Compensation — Not Publicly Disclosed](#92-compensation--not-publicly-disclosed)
   - 9.3 [The Culture Markers](#93-the-culture-markers)
10. [The Cymbal Bank Worked Example — A Market Maker as Prime-Brokerage Client](#10-the-cymbal-bank-worked-example--a-market-maker-as-prime-brokerage-client)
    - 10.1 [The Scenario](#101-the-scenario)
    - 10.2 [The Prime-Brokerage Product Set for a Market Maker](#102-the-prime-brokerage-product-set-for-a-market-maker)
    - 10.3 [Margin — Portfolio Margin and Intraday Margin Calls](#103-margin--portfolio-margin-and-intraday-margin-calls)
    - 10.4 [Intraday Settlement and the Trade Lifecycle](#104-intraday-settlement-and-the-trade-lifecycle)
    - 10.5 [Clearing — Central Clearing via SGX-DC vs Bilateral](#105-clearing--central-clearing-via-sgx-dc-vs-bilateral)
    - 10.6 [The KYC/AML and Regulatory Overlay](#106-the-kycaml-and-regulatory-overlay)
    - 10.7 [Trading Connectivity](#107-trading-connectivity)
11. [The Claims Audit — Verified, Flagged, Rejected](#11-the-claims-audit--verified-flagged-rejected)
    - 11.1 [The Verified Claims (✅)](#111-the-verified-claims-)
    - 11.2 [The Flagged Claims (⚠)](#112-the-flagged-claims-)
    - 11.3 [The Rejected Claims (❌)](#113-the-rejected-claims-)
    - 11.4 [What Could Not Be Verified](#114-what-could-not-be-verified)
12. [Glossary](#12-glossary)
13. [Cross-References and Further Reading](#13-cross-references-and-further-reading)
14. [Closing Summary](#14-closing-summary)

---

## 1. The Overview

### 1.1 The Short Answer

**Singapore is Asia's electronic-liquidity capital** — the city-state where the world's largest proprietary trading and market-making firms run their Asia-Pacific books against the region's most important derivative and ETF franchises. The reference firm for this guide, **Optiver**, is the archetype: a privately held Amsterdam market maker, founded on 9 April 1986 on the floor of the European Options Exchange, that now quotes prices in listed derivatives, cash equities, ETFs, bonds and FX on more than 50 exchanges worldwide — and that opened its Singapore office in 2021, the fifth Optiver office in Asia-Pacific after Australia (1996), Taipei (2005), Hong Kong (2007) and Shanghai (2012) ✅ (optiver.com press release, 6 April 2021; Wikipedia).

The Singapore market-making landscape sits on a precise scaffold:

- **The exchange:** **Singapore Exchange (SGX)** operates the cash-equity market (SGX-ST), the derivatives market (SGX-DT), the securities depository (**CDP** — Central Depository (Pte) Limited) and the derivatives clearing house (**SGX-DC** — SGX Derivatives Clearing), and is an approved exchange and approved clearing house under the Securities and Futures Act 2001 ✅ (SGX group structure; cross-ref §7).
- **The schemes:** SGX runs **designated market-maker (DMM)** regimes for listed securities and ETFs under Chapters 5 and 6 of the SGX-ST Rules, plus market-making programmes on the derivatives side; the ETF DMM obligations were streamlined with effect from 1 December 2025 ✅ (SGX RegCo announcement, 5 September 2025; Allen & Gledhill, 28 January 2026).
- **The firms:** the global peer set is dense — Flow Traders (SGX ETF DMM since 2007 in Asia), IMC, Jane Street, Jump, SIG, Tower Research, HRT, XTX and DRW all trade the region, with verifiable Singapore footprints for most of them (⚠ where the office year is not public), plus a home-grown tier led by AlphaGrep (founded 2010) and digital-asset specialists such as QCP Capital.
- **The regulator:** the **Monetary Authority of Singapore (MAS)** licences dealing in capital-markets products under the **Capital Markets Services (CMS)** regime of the SFA; Optiver Singapore Trading Pte. Ltd. is a live CMS licensee for dealing in exchange-traded and OTC derivatives contracts ✅ (MAS Financial Institutions Directory).

For a bank like Cymbal Bank, this landscape is a client segment, a market-structure counterparty set, and a technology benchmark — the subject of the worked example in §10.

### 1.2 What This Guide Covers

This guide covers the market-making landscape in Singapore in five layers, mirroring the repository's capital-markets cluster:

1. **The reference firm** — Optiver: history, scale, financials, the electronic market-making model (quoting both sides, inventory risk, hedging), the Asia-Pacific footprint and the Singapore office (§2).
2. **The exchange context** — SGX's DMM schemes and incentives, verified at primary sources (§3).
3. **The peer landscape** — nine global firms and the home-grown/regional tier, each verified at primary sources for founding year, HQ, products and Singapore footprint (§4–§5), consolidated in the comparison table (§6).
4. **The infrastructure and regime** — SGX/CDP/SGX-DC, the Reach trading engine and co-location, and the MAS overlay (§7–§8), cross-referencing the sibling low-latency and MAS guides rather than re-deriving them.
5. **The bank's view** — the talent market and the Cymbal Bank worked example (§9–§10).

### 1.3 The Key-Facts Table

| Aspect | Fact | Status |
| --- | --- | --- |
| Reference firm | Optiver Holding B.V. — proprietary trading firm and market maker, Amsterdam | ✅ |
| Optiver founded | 9 April 1986, by Johann Kaemingk, Ruud Vlek and Chris Oomen, as an options market maker on the European Options Exchange (EOE, now Euronext) | ✅ |
| Name origin | Dutch *optieverhandelaar* — "option trader" | ✅ |
| Optiver scale | Revenue €3.5B, net income €1.4B, total equity €4.9B, 2,112 employees (2024, Annual Review) | ✅ |
| Optiver venues | Trades on 50+ exchanges; leading MM in Nasdaq-100, Russell 2000, E-mini S&P 500 options; top-3 Eurex on-screen MM in single-stock/equity-index derivatives | ✅/⚠ |
| Optiver APAC | Australia 1996, Taipei 2005, Hong Kong 2007, Shanghai 2012, Singapore 2021 | ✅ |
| Optiver Singapore | Office announced 6 April 2021; entity Optiver Singapore Trading Pte. Ltd. incorporated 1 March 2021; now FICC (FX + commodities) hub at CapitaGreen | ✅ |
| Optiver SG licensing | MAS CMS licensee — dealing in exchange-traded derivatives contracts and OTC derivatives contracts | ✅ |
| Optiver regulatory record | 2012 settlement: US$14M fine + US$1M disgorgement for 2007 NYMEX "banging the close" manipulation | ✅ |
| SGX DMM regime | SGX-ST Rules Chapters 5 & 6 (Designated Market-Maker); ETF DMM obligations streamlined effective 1 Dec 2025 | ✅ |
| SGX engine | Reach: live 15 Aug 2011, <90µs latency, part of S$250M technology investment; co-location at SGX data centre | ✅ |
| Clearing | CDP (securities), SGX-DC (derivatives) — SGX subsidiaries; SGX approved exchange/clearing house under SFA | ✅/⚠ |
| Global peers | Flow Traders (2004, Amsterdam; SG 2007), IMC (1989, Amsterdam; no SG office listed ⚠), Jane Street (1999, NYC; SG office — year ⚠), Jump (1999, Chicago; SG entity 2011), SIG (1987, Bala Cynwyd; no SG office listed ⚠), Tower (1998, NYC; SG office — year ⚠), HRT (2002, NYC; SG office — year ⚠), XTX (2015, London; SG FX pricing engine 2018 ⚠), DRW (1992, Chicago; SG presence ⚠) | ✅/⚠ per firm |
| Home-grown | AlphaGrep (founded 2010, Indian-origin, SG HQ), QCP Capital (founded 2017, SG digital-asset firm) | ✅/⚠ |

### 1.4 Why This Matters to a Bank

For Cymbal Bank, the market-making landscape matters on four fronts. **First**, as a client segment: market makers are voracious consumers of prime-brokerage and clearing services — margin (portfolio margin, intraday calls), intraday settlement, and central-clearing access through the bank's membership (the worked example in §10). **Second**, as market-structure counterparties: a bank's execution, smart-order-routing and market-data architecture must interoperate with the electronic-liquidity ecosystem that these firms constitute (§7). **Third**, as a competitive benchmark: the low-latency C++, co-location and market-data engineering these firms deploy defines the state of the art that sell-side technology teams are measured against (cross-referenced in §7.3). **Fourth**, as a regulatory weathervane: the SGX DMM obligations, the MAS conduct regime, and the clearing-house margin dynamics directly shape the operating environment of a bank's electronic-trading businesses (§3, §8).

---

## 2. The Reference Firm — Optiver: Profile, History and the Market-Making Model

### 2.1 The Founding and the Name

Optiver was founded on **9 April 1986** in Amsterdam by **Johann Kaemingk, Ruud Vlek and Chris Oomen** as a market maker in options on the **European Options Exchange (EOE)** — the Amsterdam options floor that later became part of Euronext ✅ (Wikipedia, citing the founders' biographies and optiver.com; Optiver's "About Us"). The firm's name is a contraction of the Dutch *optieverhandelaar*, "option trader" ✅ (Bloomberg, 2015, cited by Wikipedia). The founding premise — a firm that makes two-sided prices in listed options, using its own capital at its own risk — has not changed in forty years; only the venue has, from the EOE floor to fully electronic trading on more than 50 exchanges and trading platforms ✅ (Markets Media, "Trader Profile: Optiver", 2013, cited by Wikipedia).

The group structure: **Optiver Holding B.V.**, a privately held Dutch company, with group CEO **Jan Boomaars** (a former Goldman Sachs partner hired as Europe CEO in 2015, later group CEO) ✅ (Bloomberg, 18 September 2015, cited by Wikipedia; optiver.com).

### 2.2 The Firm Today — Scale, Products and Financials

Optiver is one of the largest non-bank electronic market makers in the world. Verified scale figures (Optiver Annual Review 2024, published March 2025, via Wikipedia and optiver.com):

- **Revenue: €3.5 billion (2024)**; operating income €1.8 billion; **net income €1.4 billion**; total equity €4.9 billion ✅.
- **Employees: 2,112 (2024)** — up from "nearly 1,200" at the time of the April 2021 Singapore announcement ✅.
- **Products:** listed derivatives (options and futures), cash equities, exchange-traded funds (ETFs), bonds, and foreign exchange ✅ (Wikipedia; optiver.com).
- **Venues:** more than 50 exchanges and trading platforms globally ✅; in Europe it is one of the three most active on-screen market makers in single-stock and equity-index derivatives at Eurex; in the US it is a leading market maker in Nasdaq-100, Russell 2000 and E-mini S&P 500 options and one of the biggest market makers in Treasury options on CME; it entered FX options in 2017 ✅ (Markets Media 2013; Risk.net 2022; Bloomberg 2022; all cited by Wikipedia).

The firm is a member of the European Principal Traders Association (FIA EPTA), the FIA Principal Traders Group (PTG) in the US and FIA Japan ✅ (epta.fia.org member list, cited by Wikipedia). In 2023 Optiver became the first market maker to join the Plato Partnership as a full member ✅ (The TRADE, 2023, cited by Wikipedia). Recent developments verified against optiver.com: in August 2026 Optiver announced a majority stake in Northpool, a European energy trading firm (optiver.com news, 18 August 2026) — a signal of expansion beyond listed markets.

The firm's public milestone trail, verified via Wikipedia's citations into the financial press:

- **November 2016** — reported to have joined a consortium to build a faster data-transmission network between Chicago and Tokyo (Bloomberg) — the cross-continental low-latency play that mirrors the SGX "connect global hubs" ambition (§7.2).
- **December 2017** — partnered with Equiduct to offer a one-stop shop for best execution in European equities and ETFs; **June 2019** — joined Virtu in funding Equiduct.
- **June 2022** — joined Aquis Exchange as a member; **September 2022** — became a trading member of the Vienna Stock Exchange.
- **November 2018** — partnered with Utrecht University on an "Algorithms in Finance" course; **October 2025** — partnered with IIT Bombay to establish an AI Innovation Lab (optiver.com).
- **2022** — opened a new London office with capacity for ~200 employees; **2023** — moved into new downtown Chicago offices with room for ~600 (Wikipedia, citing press).

The pattern is consistent: expand venue membership, invest in connectivity and talent, and keep the firm private and founder-owned. Optiver's 2025 results (announced 31 March 2026, the firm's 40th anniversary) were described by the firm as "robust"; specific 2025 figures were not captured in the sources reviewed this pass ⚠.

### 2.3 The Market-Making Model — How It Works

The market-making model that Optiver runs — and that every firm in this guide runs in some variant — is worth stating precisely, because it drives every downstream requirement (margin, clearing, connectivity) discussed in §10:

- **Quoting both sides.** The market maker continuously publishes a two-sided quote — a bid and an offer — in a product, earning the spread when both sides are taken. The obligation is contractual where the firm is a designated market maker (e.g., SGX ETF DMM, §3) and voluntary elsewhere; either way, the business is "providing liquidity to financial markets using its own capital, at its own risk" (Optiver's own formulation, April 2021 press release).
- **Inventory risk.** The spread alone is not the profit; the risk is. Every time one side of the quote is hit, the firm carries inventory — a long or short position — that can move against it before the other side is filled. The whole discipline of the model is *pricing* the quote so that the expected cost of carrying inventory is inside the spread, and *managing* the inventory once it appears.
- **Hedging.** Inventory is hedged dynamically: an option position is delta-hedged in the underlying, vega-hedged with other options, and so on. The hedging flow — futures, the underlying cash instrument, correlated products — is often the largest component of a market maker's executed volume, and it is why market makers are simultaneously liquidity providers and large flow consumers (a fact a prime broker must model; §10).
- **Electronic market making.** At Optiver's scale, all of this is automated: pricing models (fair-value engines per product), risk limits per desk and per trader, and order-entry/quote-management systems that re-price on every market-data tick. The latency engineering that makes this viable — co-location, kernel-bypass networking, lock-free C++, the FIX/binary order-entry protocols, and the real-time data distribution in between — is documented in the sibling guides and cross-referenced in §7.3 rather than re-derived here.

The economics are simple in shape: gross spread capture, minus adverse-selection losses (when informed flow hits the quote), minus inventory-carry cost, minus technology and clearing costs. The moat is speed, models and risk discipline — which is why the peer set in §4 all look alike at the balance-sheet level (private, own-capital, technology-heavy) while differing in product emphasis.

### 2.4 The Asia-Pacific Footprint and the Singapore Office

Optiver was one of the first global market makers in Asia-Pacific, and its expansion sequence is verified against the firm's own April 2021 press release:

| Office | Year | Status |
| --- | --- | --- |
| Australia (Sydney) | 1996 | ✅ (firm's own release; "the firm first made its presence in the region in Australia in 1996") |
| Taipei | 2005 | ✅ |
| Hong Kong | 2007 | ✅ |
| Shanghai | 2012 | ✅ |
| Singapore | 2021 | ✅ (announced 6 April 2021; entity incorporated 1 March 2021) |
| London (Europe, listed for context) | 2022 | ✅ (new London office, ~200 capacity) |

**The Singapore office** — the fact set, all verified this pass:

- **Announcement:** 6 April 2021 — "Optiver expands presence in Asia-Pacific," with the "anticipated opening of an office in Singapore in 2021," the ninth office globally and the fifth in APAC ✅ (optiver.com/insights/news/optiver-expands-presence-in-asia-pacific/).
- **Entity:** Optiver Singapore Trading Pte. Ltd., incorporated **1 March 2021**; a sister IP entity (Optiver Singapore IP Pte. Ltd.) was incorporated the same day ✅ (ACRA records via sgpbusiness.com/EMIS). ⚠ Headcount for the Singapore office is not publicly disclosed — flagged honestly; only the global figure (2,112 at end-2024) is verified.
- **Mandate at opening:** Singapore MD **Zhivko Zhelev** said the firm would focus on "derivatives trading in commodities and equity products both on Singaporean exchanges and across other markets around the world" ✅ (optiver.com, 6 April 2021).
- **Mandate today:** optiver.com's Singapore location page (2026) describes Singapore as "central to Optiver's FICC trading in APAC," with "a focus on FX and commodities," teams across trading, technology and research — plus, notably, a Singapore-based **Digital Assets Trader (Crypto Options)** role on the firm's job board ✅ (optiver.com/about-us/locations/singapore/). Address: 138 Market Street #25-01, CapitaGreen.
- **Licensing:** Optiver Singapore Trading Pte. Ltd. is a live **MAS CMS licensee** — dealing in capital markets products: exchange-traded derivatives contracts and OTC derivatives contracts ✅ (MAS Financial Institutions Directory, eservices.mas.gov.sg).

**Why Singapore (per the firm's own words):** group CEO Jan Boomaars, announcing the office, said Optiver had "a long history of doing business in Asia Pacific," that Singapore's "strategic significance in global financial markets continues to grow," and that the firm had been "tremendously impressed by the talent in Singapore" ✅ (optiver.com, 6 April 2021). The official framing: Singapore was chosen as a "strategic location from which to further strengthen access to Asia's financial markets, attract local talent and operate in close proximity to regional business partners" ✅ (same release). That is the same gateway logic that drives every firm in §4–§5 — and the same logic Cymbal Bank's institutional franchise is built on.

### 2.5 The SGX Role

What Optiver actually does on SGX is only partially public, and the guide flags the boundary honestly:

- **Verified:** Optiver's Singapore mandate explicitly includes trading "on Singaporean exchanges" (the 2021 announcement), the firm holds the MAS licence that covers dealing in exchange-traded derivatives, and its APAC positioning statement highlights "significant market share in Hong Kong, Japanese and Korean equity derivatives and ETF markets" ✅ (optiver.com "Trade with us in Asia-Pacific").
- **Not verified:** Optiver's specific SGX market-maker registrations (which SGX derivatives contracts it is an appointed market maker for, and any SGX ETF DMM role) are **not publicly disclosed** — SGX does not publish a complete public register of derivatives market makers with per-firm contract assignments, and Optiver does not list SGX appointments on its site ⚠. The guide therefore states what is public (the mandate, the licence, the FICC focus) and flags the rest.

The one firm whose SGX ETF DMM role *is* public is Flow Traders Asia (verified via SGX DMM-obligation announcements, §3.2) — a reminder that SGX disclosure practice varies by scheme and by firm.

### 2.6 The Regulatory Record

Optiver's one major enforcement episode is well documented and worth recording for the conduct overlay in §8 and §11:

- **The scheme:** for 11 days in March 2007, Optiver engaged in 19 instances of "banging the close" on three NYMEX-listed futures contracts (WTI crude oil, New York Harbor heating oil and reformulated gasoline blendstock) — trading large volumes near the close to manipulate closing prices ✅ (CFTC press release 5521-08, charging Optiver and employees).
- **The settlement:** in 2012 Optiver paid a **US$14 million fine plus US$1 million disgorgement**; it was barred from trading US oil futures in the last three minutes of the session for two years, and several traders received bans of 2–8 years from commodities trading ✅ (CFTC press release 6239-12; Reuters, 2012; Wikipedia).
- **Context:** the episode is the industry's cautionary tale about the line between legitimate liquidity provision and manipulation of the close — precisely the conduct risk that MAS's market-conduct regime and a bank's own surveillance overlay are designed to catch (§8, §10.6).

---

## 3. The SGX Market-Making Context — Schemes, Obligations and Incentives

### 3.1 The Designated Market-Maker Regime (SGX-ST Rules)

SGX's market-making framework for the securities market is codified in the **Singapore Exchange Securities Trading Limited (SGX-ST) Rules**, Chapters 5 and 6, titled "Designated Market-Maker" ✅ (rulebook.sgx.com). The verified mechanics:

- A **Designated Market-Maker (DMM)** is a trading member approved by SGX-ST to quote two-sided prices in specified securities; the applicant's name is entered in the **Register of Designated Market-Makers**, and SGX-ST notifies all DMMs and trading members of the effective date of registration ✅ (SGX-ST Rules, Chapter 5).
- DMMs must provide an independent auditor's report to SGX-ST on request, covering financial standing, personnel or internal control procedures (obligation added 3 June 2019) ✅ (SGX-ST Rules, Chapter 6).
- The rules govern registration, quotation obligations (bid/offer spread and size requirements per security), cessation and resumption of quotations, and disciplinary liability — a former DMM remains liable for obligations incurred during its registration period ✅ (SGX-ST Rules, Chapters 5–6).

The cash-equity DMM regime exists to support liquidity in specified securities (typically less-liquid mainboard and Catalist names, and new listings); the more visible scheme in practice is the ETF DMM programme (§3.2).

### 3.2 The ETF DMM Scheme and the 2025 Streamlining

The ETF DMM scheme is the scheme where SGX's disclosure practice is best documented, and where a named firm (Flow Traders Asia) is publicly identifiable ✅:

- **The obligations:** ETF DMMs commit to continuous two-sided quotations in specified ETFs listed and/or quoted on SGX-ST, within prescribed maximum spread and minimum size parameters, in exchange for fee rebates on qualifying trades.
- **The 2025 streamlining:** on **5 September 2025** SGX RegCo sought market feedback on proposed changes to streamline ETF DMM requirements; the resulting amendments to the SGX-ST Rules took effect on **1 December 2025**, removing the administrative requirements for notification and announcement when ETF DMMs cease or resume bid/offer quotations ✅ (SGX RegCo announcement via links.sgx.com, 5 September 2025; Allen & Gledhill client update, 28 January 2026).
- **Named DMM:** **Flow Traders Asia Pte. Ltd.** is verifiably an SGX ETF DMM — an SGX announcement of 11 March 2025 records Flow Traders Asia advising that it did not send ask-side market-making orders for a ticker on SGX between ~09:30 and ~11:00 that day due to a technical issue, under the heading "Designated Market Maker (DMM) Obligations" ✅ (links.sgx.com corporate announcement, SG231127OTHRRRP8 series).

The precise current fee-rebate parameters of the ETF DMM scheme (rebate rates, spread caps per ETF tier) are **not published in a consolidated public register** in the sources captured this pass ⚠ — the verified facts are the rule chapters, the 2025 streamlining, and the named-DMM disclosure practice.

### 3.3 The Derivatives Market-Making Programmes

On the derivatives side, SGX operates market-making arrangements for listed derivatives (futures and options on the SGX-DT market), under which registered market makers quote two-sided prices in assigned contracts in exchange for fee concessions and other incentives. What is verified:

- The SGX Derivatives market (SGX-DT) hosts the region's flagship listed-derivatives complex — equity-index futures and options, FX futures (including the USD/CNH and INR pairs), and commodity derivatives — and market-making support exists for these contracts ✅ (sgx.com derivatives pages; cross-ref §7.1).
- The SGX-ST DMM rule chapters apply to the securities market; the derivatives-side market-making arrangements are governed separately under SGX-DT rules and contract-level agreements, whose parameters are not consolidated in a single public document in the sources captured this pass ⚠.

**Honest boundary:** the guide cannot verify the current per-contract incentive parameters of SGX's derivatives market-making programme, nor produce a complete public register of which firms are appointed market makers for which derivatives contracts. Where a firm's own disclosures identify SGX activity (Optiver's "Singaporean exchanges" mandate, §2.4; Flow Traders Asia's ETF DMM role), the guide states them; everything else is flagged ⚠ and collected in §11.4.

### 3.4 What Is Public — and What Is Not

| Element | Status |
| --- | --- |
| SGX-ST Rules Ch. 5 & 6 — DMM regime | ✅ (rulebook.sgx.com) |
| ETF DMM streamlining, effective 1 Dec 2025 | ✅ (SGX RegCo; Allen & Gledhill) |
| Flow Traders Asia as SGX ETF DMM | ✅ (SGX DMM-obligation announcement, Mar 2025) |
| Current ETF DMM fee-rebate parameters | ⚠ not consolidated publicly |
| Derivatives market-making programme parameters | ⚠ not consolidated publicly |
| Complete public register of derivatives market makers | ⚠ not found this pass |

---

## 4. The Peer Landscape — The Global Electronic Market Makers

Every firm below was verified this pass at primary sources (firm websites, corporate registries, financial press, Wikipedia as last resort). Where a founding year or office date could not be verified, the guide says "not publicly disclosed" and flags ⚠ — nothing was invented.

### 4.1 Flow Traders

- **Founded:** 2004, Amsterdam ✅ (flowtraders.com; Wikipedia). **HQ:** Amsterdam.
- **Products:** the ETF/ETP specialist — market making in exchange-traded products (ETFs, ETCs, ETNs), now extended to fixed income and digital assets ✅ (LinkedIn company description; flowtraders.com).
- **Structure:** listed on Euronext Amsterdam; one of the few publicly listed market makers.
- **Singapore presence:** **Flow Traders Asia Pte. Ltd. incorporated 25 April 2007**; "in June 2007 Flow Traders Asia in Singapore was established" ✅ (ACRA via sgpbusiness.com; exchangetradedfunds.com, 2008; The TRADE). The firm "has been operating as a Singapore-based market maker since 2007" ✅ (The TRADE).
- **SGX role:** verified **SGX ETF designated market maker** (§3.2) ✅.

### 4.2 IMC

- **Founded:** 1989, as "International Marketmaker's Combination", Amsterdam ✅ (imc.com; Wikipedia). **HQ:** Amsterdam.
- **Products:** options, ETFs and equities market making; 1,600+ employees; Amsterdam office 500+ colleagues across 60+ nationalities ✅ (imc.com; Wikipedia).
- **Singapore presence:** **none listed** — IMC's official office list (imc.com, 2026) covers Amsterdam, Chicago, Sydney, Mumbai, London, Zug, Aarhus, Hong Kong, New York and Seoul (Seoul opened 2020); Singapore does not appear ✅ (verified negative against imc.com). APAC is run from Sydney and Hong Kong. ⚠ A small SG-linked entity could exist without being listed; none was found this pass.

### 4.3 Jane Street

- **Founded:** 1999 (incorporated 31 August 1999), New York City ✅ (Wikipedia, citing corporate registry). **HQ:** 250 Vesey Street, NYC.
- **Founders:** Tim Reynolds, Robert Granieri, Marc Gerstein, Michael Jenkins — three of them former Susquehanna traders ✅ (Wikipedia).
- **Products:** the ETF/equity-options powerhouse — started in ADRs, moved into ETFs (now the main focus), equity options, fixed income, futures, commodities; ~US$2 trillion of equity volume per month in 2024; record Q2 2025 net trading revenue of US$10.1 billion ✅ (Wikipedia, citing Bloomberg and press).
- **Technology:** OCaml as the internal language — a distinctive engineering culture marker ✅ (Wikipedia).
- **Singapore presence:** a Singapore office exists and is growing — Bloomberg/Business Times reported in 2025 that Jane Street plans to move to a larger Singapore office at IOI Central Boulevard Towers, doubling capacity, as part of its "latest Asia expansion" ✅ (The Business Times, 2025). ⚠ The office's **establishment year is not publicly disclosed** in the sources captured this pass; the firm's first Asian office was Tokyo (2010) ✅ (The Business Times).

### 4.4 Jump Trading

- **Founded:** 1999, Chicago, by former CME pit traders Paul Gurinas and Bill DiSomma (who met in the Deutsche Mark pit) ✅ (Wikipedia). **HQ:** Chicago.
- **Products:** HFT and algorithmic trading across futures, options, equities and digital assets (Jump Crypto); heavy investment in low-latency infrastructure including microwave links ✅ (Wikipedia; tradermath.org).
- **Singapore presence:** **Jump Trading Pacific Pte. Ltd. incorporated 15 March 2011**, at Asia Square Tower 1, 8 Marina View ✅ (ACRA via opengovsg.com and sgpbusiness.com).

### 4.5 SIG — Susquehanna International Group

- **Founded:** May 1987, by a group of college friends including Jeff Yass and Arthur Dantchik (named after the Susquehanna River) ✅ (Wikipedia; sig.com). **HQ:** Bala Cynwyd, Pennsylvania (Philadelphia area).
- **Products:** the options specialist — designated primary market maker in ~600 equity options and ~45 index options on CBOE, AMEX, PHLX and ISE; also equities, fixed income, energy, private equity/venture ✅ (Wikipedia). Revenue US$7.2 billion (2024); 3,500+ employees ✅.
- **Singapore presence:** **none listed** — SIG's verified office set (Wikipedia, citing sig.com and press) covers New York, Chicago, San Francisco, Philadelphia, Boston, Stamford, Sydney, Dublin, London, Shanghai, Beijing, Hong Kong and Tokyo ✅ (verified negative). ⚠ No SIG Singapore entity was found this pass.

### 4.6 Tower Research Capital

- **Founded:** February 1998, New York, by Mark Gorton and Alistair Brown ✅ (Wikipedia). **HQ:** New York City; ~1,400+ employees (2026); 11 offices ✅.
- **Products:** one of the oldest automated/HFT firms; quant trading across global markets, connected to 150+ venues; crypto market making via internal team Limestone Trading (expanded 2025) ✅ (Wikipedia, citing Bloomberg).
- **Singapore presence:** Singapore is on Tower's official office list (tower-research.com/offices) ✅. ⚠ The **office's establishment year is not publicly disclosed** — the task brief's working assumption of "~2007–2010" could not be verified and is treated as unconfirmed (§11.4).
- **Regulatory record:** Latour Trading (Tower subsidiary) fined US$16 million in 2014 for net-capital violations; Tower fined US$67.4 million in 2019 for E-mini spoofing, with traders criminally charged ✅ (Wikipedia, citing WSJ/DOJ) — relevant to the conduct overlay in §8.

### 4.7 Hudson River Trading (HRT)

- **Founded:** 2002, New York City ✅ (Wikipedia). **HQ:** NYC.
- **Products:** algorithmic market making across asset classes; >15% of US daily equity volume as of 2021; 2025 net trading revenue US$12.3 billion (Bloomberg) ✅; mid-frequency + HFT mix; ~25% of capital held overnight ✅ (Wikipedia).
- **Singapore presence:** Singapore is on HRT's office list (14 offices worldwide incl. Singapore, Shanghai, Mumbai, Dublin, London) ✅ (Wikipedia, citing Bloomberg). ⚠ The **office's establishment year is not publicly disclosed**.

### 4.8 XTX Markets

- **Founded:** 30 January 2015, London, by Alex Gerko (spin-off of GSA Capital) ✅ (Wikipedia). **HQ:** London; ~300 employees ✅.
- **Products:** machine-learning-driven electronic liquidity provision — FX (the signature franchise: 2018's third-largest FX liquidity provider with 7.36% Euromoney share; largest FX spot LP globally by 2019), plus equities, fixed income, commodities and crypto; trades >US$250 billion/day across 35 countries ✅ (Wikipedia).
- **Singapore presence:** a **Singapore office exists** at 50 Collyer Quay #07-03, OUE Bayfront ✅ (xtxmarkets.com/contact). In 2018 the firm set up a **foreign-exchange pricing engine in Singapore in conjunction with MAS** ✅ (Wikipedia, citing The TRADE). ⚠ The office's **establishment year is not publicly disclosed** in the sources captured this pass.

### 4.9 DRW

- **Founded:** 1992, Chicago, by Don Wilson (former CME options trader; the name is his initials) ✅ (Wikipedia). **HQ:** Chicago (540 W Madison).
- **Products:** fixed income, derivatives, energy, agriculture and digital assets; crypto subsidiary **Cumberland** (launched 2014) — a major institutional crypto liquidity provider ✅ (Wikipedia). ~2,000 employees (2025) ✅.
- **Singapore presence:** Singapore is on DRW's official office list — 8 Marina View, Asia Square Tower 1, #32-05 ✅ (drw.com/offices). ⚠ The **office's establishment year is not publicly disclosed**.

---

## 5. The Peer Landscape — Home-Grown and Regional Firms

### 5.1 AlphaGrep

- **Founded:** 2010, by **Mohit Mutreja and Parshant Mittal**, University of Pennsylvania alumni who returned to India "with the aim of building an Indian-origin quant firm" ✅ (PR Newswire, 2023; Trading Interview profile). The firm is Indian-origin with a **Singapore-headquartered group** (AlphaGrep Pte. Ltd., registered in Singapore) ✅ (alphagrep.com; LinkedIn).
- **Products:** quantitative trading and market making across asset classes on 30+ exchanges; low-latency proprietary technology; an investment-management arm (AlphaGrep Investment Management) launched 2023 ✅ (alphagrep.com; PR Newswire).
- **Scale:** 250+ people across 10 offices — Shanghai, Chicago, **Singapore (5 Temasek Boulevard, Suntec Tower 5)**, London, Gurugram, Mumbai, Bangalore and Switzerland ✅ (alphagrep.com).
- **⚠ note:** the founding-year and founder claims were verified against PR Newswire and the firm's LinkedIn (founded 2010) ✅; the "Singapore HQ" framing is the firm's own positioning — the legal-group structure (which entity is the ultimate parent) is not fully public ⚠.

### 5.2 QCP Capital

- **Founded:** 2017, Singapore, by **Darius Sit** (a former Dymon Asia and BNP Paribas trader) ✅ (Vulcan Post profile; IQ.wiki; LinkedIn). One of the first digital-asset trading firms in Singapore ✅ (LinkedIn, Melvin Deng's profile).
- **Products:** institutional digital-asset trading and market making — spot, derivatives (options/futures) and structured products; "global market maker in digital asset derivatives" ✅ (qcp.capital; LinkedIn).
- **Scale/status:** over US$1.5 billion in assets deployed (per a 2023 profile); CEO Melvin Deng (joined 2021) ✅ (Vulcan Post; LinkedIn). QCP is the Singapore home-grown answer to the crypto-market-making arms of the global firms (Jump Crypto, Cumberland/DRW, §4.4, §4.9).

### 5.3 Other Regional Names

Beyond AlphaGrep and QCP, the SG electronic-liquidity ecosystem includes:

- **Virtu Financial** — the US-listed global market maker (founded 2008; HQ New York) operates an Asia-Pacific business out of Singapore; ⚠ office year and SGX role not re-verified this pass (see §11.4).
- **The international crypto market-making set** — Cumberland (DRW), Wintermute and B2C2 all trade digital assets out of or into Singapore; ⚠ their SG office years are not consolidated publicly.
- **The hedge-fund-adjacent prop shops** — Singapore's proprietary desks inside the multi-strategy managers (Citadel, Point72, Millennium — see the sibling [Hedge Funds in Singapore](hedge_funds_singapore_guide.md)) are liquidity consumers rather than market makers and are out of scope here, but they compete for the same engineering talent (§9).

The honest summary: the verified *home-grown* market-making tier is thin — AlphaGrep and QCP are the two verifiable names; the rest of the landscape is global firms with Singapore offices (§4).

---

## 6. The Comparison Table — Firms × Products × Regions × SG Footprint

| Firm | Founded | HQ | Main products | SG footprint | Notable traits |
| --- | --- | --- | --- | --- | --- |
| Optiver | 1986 | Amsterdam | Listed derivatives, cash equities, ETFs, bonds, FX | Office since 2021 (entity incorporated 1 Mar 2021); MAS CMS licensee (ETD + OTC derivatives) | The reference firm; FICC (FX + commodities) hub in SG; 50+ exchanges; €3.5B revenue (2024) |
| Flow Traders | 2004 | Amsterdam | ETFs/ETPs, fixed income, digital assets | Since 2007 (Flow Traders Asia Pte Ltd, incorporated 25 Apr 2007); **SGX ETF DMM** ✅ | Listed on Euronext; the ETF-liquidity specialist |
| IMC | 1989 | Amsterdam | Options, ETFs, equities | None listed (per imc.com) ⚠ | 1989 original "International Marketmaker's Combination"; APAC via Sydney/HK/Seoul |
| Jane Street | 1999 | New York | ETFs, equity options, fixed income, futures, commodities | Office exists; larger IOI Central Boulevard Towers move (2025) doubling capacity; **year ⚠ not public** | OCaml; ~US$2T/mo equity volume (2024); record Q2 2025 revenue |
| Jump Trading | 1999 | Chicago | Futures, options, equities, crypto | Jump Trading Pacific Pte Ltd, incorporated 15 Mar 2011 | Jump Crypto; microwave/laser low-latency infrastructure |
| SIG | 1987 | Bala Cynwyd, PA | Options (~600 equity + 45 index), equities, FI, PE | None listed (per Wikipedia office set) ⚠ | The options-specialist giant; US$7.2B revenue (2024) |
| Tower Research | 1998 | New York | HFT/quant across global markets; crypto (Limestone) | Office listed (tower-research.com); **year ⚠ not public** | 150+ venues; US$67.4M spoofing fine (2019) |
| HRT | 2002 | New York | Algorithmic MM, multi-asset | Office listed; **year ⚠ not public** | >15% of US equity volume (2021); US$12.3B revenue (2025) |
| XTX Markets | 2015 | London | FX, equities, fixed income, commodities, crypto | Office at OUE Bayfront ✅; MAS-backed FX pricing engine (2018); **year ⚠ not public** | ML-driven; largest FX spot LP globally (2019) |
| DRW | 1992 | Chicago | Fixed income, derivatives, energy, ags, digital assets | Office at Asia Square Tower 1 ✅; **year ⚠ not public** | Cumberland crypto unit (2014); 2,000 employees |
| AlphaGrep | 2010 | Singapore (group) | Multi-asset quant trading + market making | **SG-headquartered**; Suntec Tower 5 office ✅ | Indian-origin (Mumbai roots); founded by Penn alumni |
| QCP Capital | 2017 | Singapore | Digital-asset spot, derivatives, structured | **SG-headquartered** ✅ | One of SG's first crypto trading firms; US$1.5B+ deployed |

Reading the table: the global firms cluster in Marina Bay's Asia Square (Jump, DRW) and the CBD (Optiver at CapitaGreen, XTX at OUE Bayfront), the regional firms are Singapore-headquartered, and two major global names (IMC, SIG) verifiably list no Singapore office — their Asian liquidity runs out of Hong Kong, Tokyo, Sydney and Shanghai.

---

## 7. The Infrastructure — SGX, CDP, SGX-DC and the Co-Location Angle

### 7.1 The Exchange Group — SGX, SGX-ST, SGX-DT, CDP and SGX-DC

**Singapore Exchange Limited (SGX Group)** is the exchange conglomerate formed on **1 December 1999** through the merger of the Stock Exchange of Singapore (SES), the Singapore International Monetary Exchange (SIMEX, founded 1984) and Securities Clearing and Computer Services (SCCS); it listed on its own bourse on **23 November 2000** ✅ (Wikipedia, citing SGX corporate history). The group structure that matters to a market maker, verified ✅ (sgx.com via Wikipedia):

- **SGX-ST** (Singapore Exchange Securities Trading) — the securities market.
- **SGX-DT** (Singapore Exchange Derivatives Trading) — the derivatives market.
- **Central Depository (Pte) Limited (CDP)** — the subsidiary responsible for securities clearing, settlement and depository services (the CSD for SGX-listed securities).
- **SGX-DC** (SGX Derivatives Clearing) — the subsidiary for derivatives clearing and settlement operations (the CCP for SGX-listed derivatives).
- **SGX AsiaClear** — OTC clearing services (oil swaps, forward freight agreements, iron ore) launched May 2006 ✅.

SGX is Singapore's **Approved Exchange and Approved Clearing House** under the SFA, regulated by MAS ✅ (cross-ref §8). For a market maker the practical map is: trade on SGX-ST/SGX-DT → clear via CDP (securities, DvP) or SGX-DC (derivatives, daily variation margin) → settle in Singapore dollars or US dollars (SGX operates dual-currency trading) ✅.

Scale context, verified via Wikipedia's SGX infobox and corporate history: SGX listed ~776 companies with a combined market capitalisation of ~US$644.8 billion; the group is ASEAN's second-largest exchange by market capitalisation (after Indonesia), a member of the World Federation of Exchanges, and trades in both Singapore dollars and US dollars ✅. The derivatives market (SGX-DT) is the group's international franchise — the segment where the market makers in §4 concentrate, and where the CCP relationship (§7.4) matters most.

The derivatives product family itself is documented on SGX's derivatives pages (sgx.com/derivatives) — equity-index, FX and commodity derivatives forming the contract set that the market makers in §4 quote; the specific contract roster changes with the product calendar and is not re-verified here ⚠ beyond the page's existence ✅ (sgx.com search-verified this pass).

### 7.2 Reach and the SGX Data Centre

The latency story on SGX is the **Reach** initiative — a S$250 million technology investment to build "the world's fastest trading engine" and a purpose-built data centre ✅ (SGX announcement via Wikipedia; Markets Media; NVIDIA case study). Verified facts:

- **SGX Reach** went live on **15 August 2011** — an ultra-low-latency trading engine (sub-90-microsecond) with roughly ten times the capacity of its predecessor ✅ (Markets Media, "Ultrafast Trading Engine Debuts"; MarketScreener, "SGX's Reach trading engine goes live").
- The platform was delivered by **NASDAQ OMX** (based on the GENIUM engine) with Voltaire and HP ✅ (Wikipedia).
- The **SGX data centre** houses SGX's trading, market data and clearing infrastructure and offers **co-location** — hosting customer systems in the same facility for low-latency access to trading platforms and market-data engines ✅ (sgx.com "Co-Location" page; datacentermap.com profile).
- The Reach investment also aimed to "seamlessly connect trading communities in global financial hubs to Singapore" — i.e., the international low-latency connectivity play (cross-border microwave/fibre links to regional venues) ✅ (NVIDIA/SGX case study).

### 7.3 The Low-Latency Angle (Cross-Referenced)

The engineering that a market maker runs on top of this infrastructure is documented in the sibling guides and is **cross-referenced, not re-derived**, per the repository convention:

- **[FIX Protocol](fix_protocol_guide.md)** — the order-entry and market-data messaging layer: session lifecycle, NewOrderSingle/ExecutionReport, and crucially the performance encodings — **FIXP** (the FIX Performance Session Layer, §5.3 of that guide), **FAST** and **SBE** (§5.5–5.6) — which are how Optiver-style firms shrink message size and latency on venue links.
- **[Low-Latency C++ Development](../technology/low_latency_cpp_development_guide.md)** — the latency-hierarchy thinking (§1.2 of that guide), the memory hierarchy and cache-line discipline (§3), and lock-free concurrency (§5) that underpin the quote-management systems described in §2.3.
- **[DDS Guide](../technology/dds_guide.md)** — the data-centric publish-subscribe middleware (DCPS model, §1.4–1.5 of that guide) used for internal real-time market-data distribution between the market-data layer and the pricing/risk layer; its Deadline/Latency-Budget QoS (§5.5 of that guide) maps directly onto the re-quote-on-tick requirement of a market maker.
- **[Zero Downtime System Design](../technology/zero_downtime_system_design_guide.md)** — the always-on requirement: a market maker's quoting engine cannot be down during trading hours without breaching DMM obligations (§3.1) or bleeding the book.

The SGX-specific takeaway: co-location at the SGX data centre plus the Reach engine defines the venue-side latency budget; the firm-side budget (tick-to-quote) is the subject of the technology guides above.

### 7.4 Clearing and Settlement — Central Clearing via SGX-DC

For derivatives — the core of the SGX market-making complex — clearing is **central counterparty (CCP) clearing through SGX-DC** ✅:

- SGX-DC interposes itself between buyer and seller (novation), collects **initial margin** and **daily variation margin**, and operates a default-management waterfall (default fund, clearing-member contributions) ✅ (standard CCP mechanics; SGX-DC structure per §7.1).
- A market maker that is not itself a clearing member clears through a **clearing member (CM)** — which is where a bank like Cymbal Bank enters the picture: the bank's derivatives-clearing business provides the market maker's access to SGX-DC (worked example, §10).
- For cash equities and ETFs, settlement is **DvP through CDP** on the local settlement cycle (T+2 for SGX securities as of the current convention ⚠ — settlement calendars change; verify against the current SGX calendar before relying on it).
- The 2008 Lehman crisis and the subsequent OTC-clearing mandates pushed more of the ecosystem toward CCP clearing; SGX AsiaClear (2006) was an early mover in clearing OTC swaps and freight ✅ (Wikipedia).

---

## 8. The MAS Regulatory Context (Condensed)

This section condenses the MAS regime to what a market maker (and its bank) must hold in mind; the full regime is documented in the sibling [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) guide (§2 — the CMS-licence architecture, base capital, the RFMC/LFMC tiers and the 2024 RFMC repeal) and [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md), and is **not re-derived** here.

### 8.1 The CMS Licence and Dealing in Capital-Markets Products

- A firm that deals in capital-markets products in Singapore — including exchange-traded and OTC derivatives — requires a **Capital Markets Services (CMS) licence** under the **Securities and Futures Act 2001 (SFA)** ✅ (MAS; cross-ref hedge-funds guide §2).
- The market-making cohort's licences are public: **Optiver Singapore Trading Pte. Ltd.** is a live CMS licensee for dealing in exchange-traded derivatives contracts and OTC derivatives contracts ✅ (MAS Financial Institutions Directory). Other firms' SG entities hold equivalent or related licences (futures broking, fund management) as their businesses require.
- A licensed firm must meet MAS's base-capital and risk-management requirements and comply with the relevant MAS notices and guidelines on business conduct, AML/CFT and technology risk (cross-ref [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md)).

### 8.2 The Market-Conduct Overlay

The conduct overlay is where the Optiver "banging the close" episode (§2.6) and the Tower spoofing fine (§4.6) become directly relevant:

- The SFA prohibits market manipulation, false trading and spoofing-type conduct; MAS and SGX jointly enforce the market-conduct regime, with SGX RegCo supervising trading members and DMMs ✅ (cross-ref MAS guide).
- A concrete SG episode: after the October 2013 speculative crash in Blumont, Asiasons and LionGold, SGX and MAS jointly issued a February 2014 consultation proposing market-integrity enhancements — a minimum trading price, short-position reporting, and the creation of three independent regulatory bodies ✅ (Wikipedia, citing SGX/MAS).
- For a bank clearing and financing market makers, the overlay is operational: trade surveillance, order-to-trade monitoring, and DMM-obligation monitoring feed the bank's own conduct risk framework (§10.6).

### 8.3 Where the Full Regime Lives

- Fund-management licensing (RFMC repeal of 1 August 2024, LFMC tiers, base capital S$250k–S$1m+): [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §2.
- The Payment Services Act and the digital-payment-token (DPT) measures — relevant to the crypto-market-making names (Jump Crypto, QCP, Cumberland): [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §8.2 and [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md).
- The client-assets and margin rules governing prime brokerage: [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) and [Citadel LLC](citadel_llc_guide.md) §9.3.

---

## 9. The Talent Market in Singapore

The talent conventions follow [Citadel LLC](citadel_llc_guide.md) §8: what is verifiable is the hiring pattern and the culture markers; **compensation is not publicly disclosed and is flagged ⚠** throughout.

### 9.1 The Hiring Landscape

- The Singapore offices of the global firms (§4) recruit the same narrow talent pool: quantitative researchers, low-latency C++ engineers, and traders — drawn from NUS, NTU, SMU, and from the region's quant programs (plus global transfers).
- The hiring signal is verified: Optiver's Singapore careers page (2026) lists quantitative-trading internships, legal, and digital-assets trading roles (including a Singapore **Digital Assets Trader (Crypto Options)** role) ✅ (optiver.com); Jane Street's 2025 Singapore expansion "will expand headcount" ✅ (The Business Times); XTX, Jump, Tower, HRT and DRW all post SG roles on their careers pages ✅.
- SGX and the EDB/GXS-style industry machinery promote Singapore as the regional trading-technology hub — the same promotion machinery documented for fund management in the hedge-funds guide §3.5.
- The recruiting pattern is quant-competition driven: firms run trading-game and puzzle-based assessment processes (Optiver's "Quantitative Trading Internship (Singapore)" and Jane Street's market-prediction competition are public examples ✅ — optiver.com; Wikipedia). Internships are the primary conversion channel into full-time trader and engineer roles, as they are at Citadel Securities (cross-ref Citadel guide §8.1).

### 9.2 Compensation — Not Publicly Disclosed

- No Singapore office of any firm in §4–§5 publishes compensation. ⚠ All compensation statements in this guide would be estimates and are therefore omitted — the honest position is "not publicly disclosed."
- The one verified regional data point is Australian, not Singaporean: Bloomberg reported in March 2023 that Optiver's Sydney office was paying some new graduate traders ~A$400,000 (≈US$270,000 at the time), citing the firm's rookie pay ✅ (Bloomberg, 9 March 2023, cited by Wikipedia). Singapore pay levels are not public ⚠.
- Industry-wide, the market-making compensation model is the same as Citadel Securities-style firms: high base plus a large performance-linked component tied to desk P&L — the structure is documented in the Citadel guide §8.2 conventions.

### 9.3 The Culture Markers

- **Ownership:** the global firms are private and founder-owned (Optiver — privately held; Jane Street — profit-sharing across the firm; AlphaGrep/QCP — founder-run) ✅ (Wikipedia; firm sites).
- **Engineering-first:** OCaml at Jane Street, C++ at Optiver/XTX, lock-free and latency-obsessed everywhere — the technology guides (§7.3) are the cultural reference.
- **Risk as a discipline:** market-making firms run tight risk limits and treat a losing day as a process question — the 2007 NYMEX episode (§2.6) is the cautionary tale of what happens when the discipline bends.

---

## 10. The Cymbal Bank Worked Example — A Market Maker as Prime-Brokerage Client

This worked example **extends** the prime-brokerage conventions established in [Citadel LLC](citadel_llc_guide.md) §10 (the hedge fund as PB client) rather than re-deriving them: the client is now a market maker — an Optiver-style firm — and the differences that matter are margin intensity, intraday settlement, and clearing access.

### 10.1 The Scenario

An Optiver-style market maker ("the Client") runs its Singapore FICC book from CapitaGreen: FX, commodities and equity-index derivatives quoted on SGX (via its CMS licence, §8.1) and on other venues, with a global hedging book. It engages **Cymbal Bank** as its Singapore prime broker and clearing intermediary:

- **Clearing:** the Client is not an SGX-DC clearing member; Cymbal Bank clears its SGX derivatives through SGX-DC as the Client's clearing member (§7.4).
- **Financing:** the Client's margin accounts — portfolio margin across products, with intraday calls (§10.3).
- **Settlement:** intraday settlement of cash and securities positions (§10.4).
- **Connectivity:** FIX sessions from the Client's OMS/EMS to Cymbal Bank's PB and clearing systems (§10.7).

### 10.2 The Prime-Brokerage Product Set for a Market Maker

| Product | What it means for a market maker | Notes |
| --- | --- | --- |
| Derivatives clearing (SGX-DC access) | The bank's clearing membership is the Client's gateway to CCP clearing | The core relationship driver ✅ |
| Portfolio margin | Cross-product netting of options/futures/equity risk (delta, vega, correlation) | Far more capital-efficient than product-by-product margin — the Client negotiates hard on this |
| Intraday margin and collateral calls | Margin is marked intraday; the CCP (SGX-DC) calls variation margin, and the bank passes calls through — sometimes within minutes | The 15-minute SGX-DC intraday margin cycle is the operational heart of the relationship ⚠ (cycle specifics vary; verify current SGX-DC schedule) |
| Securities financing | Borrow/lend for the Client's hedging inventory and ETF market-making | ETF DMM activity (Flow Traders-style) generates constant borrow demand |
| Custody (CDP accounts) | Safe-keeping of SGX-listed securities held through CDP | Standard DvP custody ✅ |
| FX and payments | Settlement currency conversion (SGD/USD dual-currency market) | SGX dual-currency trading ✅ |
| Reporting and risk analytics | Intraday P&L, margin utilization, exposure and stress reporting | A market maker's own risk room expects bank-grade intraday data |

### 10.3 Margin — Portfolio Margin and Intraday Margin Calls

The market maker's margin profile is the opposite of a hedge fund's: **high turnover, low overnight carry, and extreme intraday volatility of margin requirements**.

- **Portfolio margin:** the Client's options and futures positions are margined on a portfolio basis — SGX-DC's risk-based margin methodology (SPAN-style) computes initial margin from scenario-based risk arrays across the portfolio; the bank's credit department overlays its own haircuts and concentration limits ✅ (SGX-DC margin methodology; cross-ref Citadel guide §10.3 conventions).
- **Intraday margin calls:** because a market maker's inventory explodes during volatility (the very moments liquidity is needed), variation margin moves intraday. The bank must: (1) pass SGX-DC intraday calls to the Client in real time; (2) fund the gap if the Client's cash is in transit; (3) hold the right to liquidate or reduce positions if a call is missed. The intraday funding facility is a priced product — and the reason banks prize market-maker clients (they monetize intraday balance-sheet usage).
- **The stress case:** the 2008-style scenario (from the Citadel guide's history) — a volatility spike, margin requirements doubling intraday, and a funding squeeze — is exactly the stress test Cymbal Bank runs before setting the Client's limits ✅ (cross-ref Citadel guide §2.4, §10.3).

**An illustrative (not real) margin timeline** — the shape of a market maker's day, with figures chosen only to show the mechanics, not to represent any actual client:

- 09:00 — the Client quotes SGX equity-index options with a portfolio of ~10,000 contracts; SGX-DC initial margin ~S$50M, fully collateralised.
- 10:30 — a regional macro shock; volatility doubles; the Client's inventory (and its hedging futures) grows as it absorbs flow; SGX-DC's intraday margin re-computation raises initial margin to ~S$90M.
- 10:45 — Cymbal Bank passes the S$40M variation call; the Client's cash at the bank covers it, but the bank's intraday facility is now drawn.
- 14:00 — the Client hedges down; margin falls back toward S$55M; the bank releases collateral and recycles the cash.
- The bank's P&L on the day: financing on the drawn intraday facility, clearing fees on the volume, and zero credit loss — provided the margin discipline held at 10:45. That discipline is the entire product.

### 10.4 Intraday Settlement and the Trade Lifecycle

- **Trade capture:** the Client's OMS/EMS sends orders and allocations to Cymbal Bank over **FIX sessions** (Logon → NewOrderSingle → ExecutionReport, gap-fill recovery) — the lifecycle is documented in [FIX Protocol](fix_protocol_guide.md) §4–§6 and applies verbatim to the market maker ↔ bank link.
- **Intraday settlement:** cash settled on a T+1/T+0 basis per product ⚠ (verify against the current SGX/CDP calendar); the bank offers intraday sweeps so the Client can reuse cash across venues within the day — a service market makers explicitly buy.
- **Fails management:** the bank monitors and finances settlement fails on the securities side; for a market maker the fail is usually a borrow problem (a short position whose locates dried up), and the securities-financing desk manages the recall risk (cross-ref Citadel guide §10.5).

### 10.5 Clearing — Central Clearing via SGX-DC vs Bilateral

- **Central clearing (the default for SGX listed derivatives):** trades are novated to SGX-DC; the clearing member (Cymbal Bank) faces SGX-DC, the Client faces the bank. Margin is computed by SGX-DC's risk system; the bank's own credit risk to the Client is the difference between what the bank must post to SGX-DC and what it has collected from the Client (the "un-margined gap").
- **Bilateral (the OTC margin):** for the Client's OTC derivatives (its CMS licence covers OTC dealing), uncleared margin rules require initial margin exchange between the Client and its counterparties, with the bank facilitating collateral logistics (segregated accounts, eligible collateral schedules) ✅ (uncleared-margin framework; cross-ref MAS guide).
- **The bank's risk principle:** the bank never wants to be structurally long the Client's credit — margin is collected to the penny, intraday, or positions are reduced. That is the same discipline documented for the Citadel-style hedge fund in the sibling guide, applied at higher frequency.

### 10.6 The KYC/AML and Regulatory Overlay

Extending the Citadel-guide overlay (its §10.6) to a market-making client:

- **KYC on the firm:** constitutive documents of the SG entity, the group structure (Optiver Holding B.V. → Optiver Singapore Trading Pte. Ltd.), the ownership register, and the firm's regulatory status across jurisdictions (MAS CMS licence verified via the public directory ✅; plus home-jurisdiction registrations).
- **KYC on control:** beneficial-ownership analysis of the founders/principals; for listed Flow Traders, the listed-company disclosure regime simplifies this; for private firms, the register and the licensing file carry the analysis.
- **AML/CFT and sanctions:** transaction monitoring across the Client's high-velocity flow (a market maker's thousands of trades/day are pattern-based, not narrative-based — the monitoring is statistical); sanctions screening on counterparties, venues and clearing members; FATCA/CRS for any pooled vehicles.
- **Conduct and reputational overlay:** onboarding a firm with an enforcement record — Optiver's 2012 NYMEX settlement (§2.6), Tower's 2019 spoofing fine (§4.6) — requires enhanced due diligence: the settlement terms, the remediation, and the current compliance control framework are assessed before limits are set ✅.
- **DMM-obligation monitoring:** where the Client holds SGX DMM obligations (§3), the bank's surveillance desk watches for obligation breaches as a conduct signal (a DMM that stops quoting during stress is a risk event, not just a regulatory one).
- **The digital-assets overlay:** a market maker with a crypto-options book (Optiver's SG job board, Jump Crypto, QCP, Cumberland) triggers the MAS digital-payment-token and virtual-asset-service-provider requirements — cross-referenced to the hedge-funds guide §8.2 rather than re-derived — plus the bank's own heightened due-diligence standards for digital-asset counterparties.

### 10.7 Trading Connectivity

The worked-example connectivity, in one line: **Client OMS/EMS ⇄ FIX (FIXP/FAST for performance) ⇄ Cymbal Bank PB & clearing ⇄ SGX (Reach, co-located) and SGX-DC**, with market data over binary/FAST feeds and margin/settlement in the bank's back office — every component documented in [FIX Protocol](fix_protocol_guide.md), [Financial Trading Order Infrastructure](financial_trading_order_infrastructure.md), and the technology guides cross-referenced in §7.3. The bank's co-location footprint at the SGX data centre (§7.2) is what lets it offer the Client venue-grade latency on the clearing link — the productized version of the low-latency engineering in [Low-Latency C++ Development](../technology/low_latency_cpp_development_guide.md).

---

## 11. The Claims Audit — Verified, Flagged, Rejected

### 11.1 The Verified Claims (✅)

| Claim | Source(s) |
| --- | --- |
| Optiver founded 9 April 1986 by Johann Kaemingk, Ruud Vlek and Chris Oomen; options MM on the European Options Exchange (EOE, now Euronext); name from Dutch *optieverhandelaar* | Wikipedia (Optiver), citing optiver.com, Bloomberg (2015), Anders Invest |
| Optiver HQ Amsterdam; privately held; products: listed derivatives, cash equities, ETFs, bonds, FX; trades 50+ exchanges | Wikipedia (Optiver); Markets Media (2013) |
| Optiver 2024 financials: revenue €3.5B, net income €1.4B, equity €4.9B, 2,112 employees | Optiver Annual Review 2024 (optiver.com PDF, March 2025) via Wikipedia; optiver.com results release (27 Mar 2025) |
| Optiver APAC sequence: Australia 1996, Taipei 2005, Hong Kong 2007, Shanghai 2012, Singapore 2021 | optiver.com press release, "Optiver expands presence in Asia-Pacific" (6 Apr 2021) |
| Optiver Singapore: entity incorporated 1 March 2021; office at 138 Market Street #25-01 CapitaGreen; today an FICC (FX + commodities) hub; Singapore digital-assets (crypto options) role posted | ACRA via sgpbusiness.com/EMIS; optiver.com/about-us/locations/singapore/ (2026) |
| Optiver Singapore Trading Pte. Ltd. is a MAS CMS licensee — dealing in exchange-traded derivatives contracts and OTC derivatives contracts | MAS Financial Institutions Directory (eservices.mas.gov.sg) |
| Optiver 2012 settlement: US$14M fine + US$1M disgorgement for March 2007 NYMEX "banging the close" (19 instances, 3 contracts); 2-year end-of-session bar; trader bans 2–8 years | CFTC press releases 5521-08 and 6239-12; Reuters (2012) |
| Optiver group CEO Jan Boomaars; 2026 majority stake in Northpool | Bloomberg (2015) via Wikipedia; optiver.com news (18 Aug 2026) |
| Optiver milestones: Chicago–Tokyo network consortium (2016); Equiduct best-execution partnership (2017) and funding (2019); Aquis membership (2022); Vienna Stock Exchange trading member (2022); Utrecht University course (2018); IIT Bombay AI Lab (2025); London office ~200 capacity (2022); Chicago office ~600 capacity (2023) | Wikipedia (Optiver), citing Bloomberg, Automated Trader, The TRADE, Holland FinTech, optiver.com |
| SGX formed 1 Dec 1999 (SES + SIMEX + SCCS); listed 23 Nov 2000; dual-currency trading | Wikipedia (Singapore Exchange), citing SGX corporate history |
| SGX group structure: SGX-ST, SGX-DT, CDP (securities clearing/settlement/depository), SGX-DC (derivatives clearing), SGX AsiaClear (May 2006) | Wikipedia (Singapore Exchange), citing sgx.com |
| SGX Reach: live 15 Aug 2011; sub-90µs; ~10x capacity; S$250M Reach initiative; NASDAQ OMX GENIUM platform; purpose-built data centre with co-location | Markets Media; MarketScreener (SGX release); NVIDIA case study; Wikipedia |
| SGX-ST Rules Chapters 5 & 6 — the Designated Market-Maker regime (register, obligations, auditor's report since 3 Jun 2019) | rulebook.sgx.com |
| ETF DMM obligations streamlined: RegCo consultation 5 Sep 2025; SGX-ST Rules amended effective 1 Dec 2025 | SGX RegCo announcement (links.sgx.com); Allen & Gledhill (28 Jan 2026) |
| Flow Traders Asia is an SGX ETF DMM (11 Mar 2025 DMM-obligation announcement) | links.sgx.com corporate announcement |
| Flow Traders founded 2004, Amsterdam; ETF/ETP specialist; Flow Traders Asia Pte. Ltd. incorporated 25 Apr 2007 ("June 2007" establishment) | flowtraders.com; Wikipedia; ACRA via sgpbusiness.com; exchangetradedfunds.com (2008); The TRADE |
| IMC founded 1989 as "International Marketmaker's Combination", Amsterdam; offices per imc.com: Amsterdam, Chicago, Sydney, Mumbai, London, Zug, Aarhus, HK, NY, Seoul — no Singapore listed | imc.com/contact/offices (2026); Wikipedia (IMC Financial Markets) |
| Jane Street founded 1999 (incorporated 31 Aug 1999), NYC; founders incl. ex-Susquehanna traders; OCaml; ~US$2T/mo equity volume 2024; record Q2 2025 revenue US$10.1B | Wikipedia (Jane Street Capital), citing Bloomberg and press |
| Jane Street SG office exists and is expanding (IOI Central Boulevard Towers move 2025, doubling capacity); first Asian office Tokyo 2010 | The Business Times (2025) |
| Jump Trading founded 1999, Chicago (Gurinas/DiSomma); Jump Trading Pacific Pte. Ltd. incorporated 15 Mar 2011 (Asia Square Tower 1) | Wikipedia (Jump Trading); ACRA via opengovsg.com/sgpbusiness.com |
| SIG founded May 1987 (Yass/Dantchik), Bala Cynwyd; ~600 equity + 45 index options; US$7.2B revenue 2024; office set incl. Sydney/HK/Tokyo/Shanghai/Beijing — no Singapore listed | Wikipedia (Susquehanna International Group); sig.com |
| Tower Research founded Feb 1998 (Gorton/Brown), NYC; Singapore on official office list; Latour US$16M fine 2014; US$67.4M spoofing fine 2019 | Wikipedia (Tower Research Capital); tower-research.com/offices |
| HRT founded 2002, NYC; Singapore on office list; >15% of US equity volume (2021); US$12.3B 2025 revenue | Wikipedia (Hudson River Trading), citing Bloomberg |
| XTX Markets founded 30 Jan 2015 by Alex Gerko, London; largest FX spot LP globally (2019); Singapore office at 50 Collyer Quay #07-03 OUE Bayfront; 2018 FX pricing engine set up with MAS | Wikipedia (XTX Markets); xtxmarkets.com/contact |
| DRW founded 1992 by Don Wilson, Chicago; Cumberland crypto unit 2014; Singapore office at 8 Marina View, Asia Square Tower 1 #32-05 | Wikipedia (DRW Trading Group); drw.com/offices |
| AlphaGrep founded 2010 by Mohit Mutreja and Parshant Mittal (Penn alumni); 250+ people, 10 offices incl. Singapore (Suntec Tower 5) | PR Newswire (2023); alphagrep.com; LinkedIn |
| QCP founded 2017, Singapore, by Darius Sit (ex-Dymon Asia/BNP); digital-asset trading and market making; US$1.5B+ deployed | Vulcan Post; IQ.wiki; LinkedIn; qcp.capital |
| SFA market-conduct context: SGX/MAS Feb 2014 consultation after the Blumont/Asiasons/LionGold crash (minimum trading price, short-position reporting) | Wikipedia (Singapore Exchange), citing SGX/MAS |

### 11.2 The Flagged Claims (⚠)

| Claim | Why flagged |
| --- | --- |
| Optiver's specific SGX market-maker registrations (which SGX derivatives contracts it is appointed for; any SGX ETF DMM role) | Not publicly disclosed; SGX publishes no complete public register of derivatives market makers with per-firm contract assignments |
| Optiver Singapore office headcount | Not publicly disclosed; only the global figure (2,112 at end-2024) is verified |
| SGX derivatives market-making programme parameters (fee concessions, quote obligations per contract) | Not consolidated in a public document captured this pass |
| Current ETF DMM fee-rebate parameters | Not published in a consolidated public register captured this pass |
| Jane Street Singapore office establishment year | Not publicly disclosed in sources captured this pass (Tokyo 2010 is verified as first Asian office) |
| Tower Research Singapore office establishment year | Not publicly disclosed; the task brief's "~2007–2010" assumption is unverified |
| HRT Singapore office establishment year | Not publicly disclosed |
| XTX Singapore office establishment year | Not publicly disclosed; the MAS-backed FX pricing engine (2018) is verified |
| DRW Singapore office establishment year | Not publicly disclosed |
| IMC and SIG Singapore presence | Verified negatives against official office lists; a small unlisted entity cannot be ruled out |
| SGX T+2 settlement cycle for securities | Standard convention but settlement calendars change; flagged for re-verification |
| "15-minute SGX-DC intraday margin cycle" (worked example) | Cycle specifics vary and were not re-verified against a current SGX-DC schedule |
| SGX as "Approved Exchange / Approved Clearing House" | Standard SFA fact, cross-referenced rather than re-verified this pass |
| Virtu Financial's Singapore office year and SGX role | Not re-verified this pass |
| Singapore market-maker compensation | Not publicly disclosed by any firm; Australian graduate-pay data point (A$400k, Bloomberg 2023) is verified but SG pay is not |
| SGX EDB-promotion machinery parallels | Cross-referenced from the hedge-funds guide rather than re-verified |
| AlphaGrep ultimate-parent structure | The "Singapore HQ" framing is the firm's own positioning; full legal-group structure not public |

### 11.3 The Rejected Claims (❌)

| Claim | Verdict | Basis |
| --- | --- | --- |
| "Optiver's Singapore office dates from ~2008–2010" | ❌ — the verified record is 2021 (announced 6 April 2021; entity incorporated 1 March 2021) | optiver.com press release; ACRA records |
| "AlphaGrep was founded by Prashant Baid" | ❌ — PR Newswire and the firm's LinkedIn identify founders Mohit Mutreja and Parshant Mittal (founded 2010) | PR Newswire (2023); LinkedIn |

---

### 11.4 What Could Not Be Verified

This subsection collects every item this pass could not confirm against a primary or reliable secondary source, so the reader can distinguish verified fact from honest uncertainty:

1. **Optiver's SGX appointment list** — which SGX derivatives contracts Optiver is a registered/appointed market maker for, and whether it holds any SGX ETF DMM role. The mandate ("trading on Singaporean exchanges"), the CMS licence and the FICC focus are verified; the appointments are not public.
2. **Optiver Singapore headcount** — not disclosed; only the global 2024 figure (2,112) is verified.
3. **SGX derivatives market-making programme parameters** — the fee/incentive schedule and per-contract quote obligations are not in any consolidated public document captured this pass.
4. **Current ETF DMM fee-rebate parameters** — the 2025 streamlining and the named DMM (Flow Traders Asia) are verified; rebate rates and spread caps are not.
5. **Jane Street Singapore office year** — the office's existence and 2025 expansion are verified via The Business Times; the establishment year is not public.
6. **Tower Research Singapore office year** — the office is listed on tower-research.com; the year is not public (the brief's ~2007–2010 assumption is unconfirmed).
7. **HRT Singapore office year** — office listed; year not public.
8. **XTX Singapore office year** — office address verified on xtxmarkets.com; the MAS-backed 2018 FX pricing engine is verified; the office's opening year is not public.
9. **DRW Singapore office year** — office address verified on drw.com; year not public.
10. **IMC and SIG Singapore entities** — both firms' official office sets exclude Singapore; a small unlisted entity cannot be categorically ruled out.
11. **Singapore market-maker compensation** — no firm publishes SG pay; the verified regional data point is Optiver Sydney graduate pay (~A$400,000, Bloomberg, March 2023).
12. **SGX securities settlement cycle (T+2) and SGX-DC intraday margin-cycle specifics** — standard conventions cited with ⚠ for calendar re-verification.
13. **Virtu Financial's Singapore operations** — office year and SGX role not re-verified this pass.
14. **AlphaGrep group legal structure** — the Singapore-headquartered positioning is the firm's own; the ultimate-parent entity chain is not fully public.

---

## 12. Glossary

| Term | Definition |
| --- | --- |
| Market maker | A firm that continuously quotes two-sided prices (bid and offer) in a product, earning the spread and providing liquidity (Optiver, Flow Traders, Jane Street) |
| Designated Market-Maker (DMM) | An SGX-ST-approved trading member with contractual quotation obligations in specified securities/ETFs (SGX-ST Rules Ch. 5–6) |
| DMM obligations | The spread, size and uptime commitments a DMM must meet; breaches are disclosed via SGX announcements |
| ETF DMM | A DMM registered for exchange-traded funds; Flow Traders Asia is a verified SGX ETF DMM |
| Inventory risk | The risk that a market maker's position moves against it between the two sides of the spread being filled |
| Hedging | Offsetting inventory risk with correlated instruments (delta hedging an option with its underlying) |
| Adverse selection | The loss a market maker suffers when informed flow (rather than noise flow) hits its quote |
| EOE | European Options Exchange — the Amsterdam options exchange where Optiver started in 1986, now Euronext |
| FICC | Fixed Income, Currencies and Commodities — Optiver Singapore's current mandate |
| CMS licence | Capital Markets Services licence — MAS's licence for regulated activities including dealing in capital-markets products (SFA) |
| SFA | Securities and Futures Act 2001 — the statute governing Singapore's capital markets |
| SGX | Singapore Exchange Limited — the exchange group formed 1 December 1999 |
| SGX-ST | SGX Securities Trading — the securities market |
| SGX-DT | SGX Derivatives Trading — the derivatives market |
| CDP | Central Depository (Pte) Limited — SGX's securities clearing, settlement and depository subsidiary |
| SGX-DC | SGX Derivatives Clearing — SGX's derivatives CCP |
| CCP | Central counterparty — the clearing house that interposes itself between buyer and seller (novation) |
| Reach | SGX's ultra-low-latency trading engine (live 15 Aug 2011, sub-90µs) |
| Co-location | Housing trading servers in the exchange's data centre to minimize latency |
| Variation margin | Daily (and intraday) collateral transfers reflecting mark-to-market moves |
| Initial margin | Collateral posted up front against potential future exposure |
| Portfolio margin | Margin computed across a portfolio's correlated risks (delta, vega, scenario arrays) rather than product-by-product |
| Un-margined gap | The difference between what a clearing member posts to the CCP and what it has collected from its client |
| Novation | The CCP's substitution of itself as counterparty to both sides of a trade |
| Banging the close | Manipulating closing prices by trading large volumes near the close (Optiver's 2007 NYMEX episode) |
| Spoofing | Placing orders with no intention of execution to distort prices (Tower's 2019 fine) |
| FIX | Financial Information eXchange — the electronic-trading messaging standard (see the sibling FIX guide) |
| FIXP / FAST / SBE | Performance-oriented FIX encodings used for low-latency messaging (see the sibling FIX guide §5.3, §5.5–5.6) |
| DDS | Data Distribution Service — the OMG publish-subscribe middleware for real-time market-data distribution (see the sibling DDS guide) |
| GENIUM | The NASDAQ OMX trading platform on which SGX Reach is based (delivered with Voltaire and HP) |
| Prime brokerage | The bundled service set (execution, clearing, custody, margin, financing, reporting) banks sell to trading firms |
| Rehypothecation | A prime broker's re-use of client collateral to fund its own book (cross-ref Citadel guide §10.4) |
| Cymbal Bank | The repository's bank persona — the only bank persona in this guide |

---

## 13. Cross-References and Further Reading

**Repository guides (banking siblings — plain filenames):**
- [Citadel LLC](citadel_llc_guide.md) — the prime-brokerage worked-example format (§10), the talent-section conventions (§8), and the market-structure context for electronic market making
- [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) — the MAS fund-management and CMS-licence regime (§2), the DPT/digital-asset regime (§8.2), and the Singapore hub context
- [FIX Protocol: The Language of Electronic Trading](fix_protocol_guide.md) — the messaging layer (sessions, NewOrderSingle/ExecutionReport, FIXP/FAST/SBE) behind every venue and PB link
- [Financial Trading Order Infrastructure](financial_trading_order_infrastructure.md) — OMS/EMS, order routing, exchange infrastructure
- [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) — the Cymbal Bank persona conventions and the full Singapore regulatory overlay
- [Private Equity](private_equity_guide.md) and [Asset Management & Alternatives](asset_management_alternatives_guide.md) — the wider alternatives context

**Repository guides (technology — prefix `../technology/`):**
- [Low-Latency C++ Development](../technology/low_latency_cpp_development_guide.md) — the C++ latency engineering behind market-making systems (§7.3)
- [DDS Guide](../technology/dds_guide.md) — the publish-subscribe middleware for real-time market-data distribution (§7.3)
- [Zero Downtime System Design](../technology/zero_downtime_system_design_guide.md) — always-on trading estates (§7.3)
- [Quantitative Developer Skillset](../technology/quantitative_developer_skillset_guide.md) — the quant-research skill stack (§9)

**Primary and press sources used this pass:**
- optiver.com (About Us; locations/Singapore; "Optiver expands presence in Asia-Pacific", 6 Apr 2021; "Optiver reports strong financial results for 2024", 27 Mar 2025; Northpool announcement, 18 Aug 2026; Optiver Annual Review 2024 PDF)
- eservices.mas.gov.sg — MAS Financial Institutions Directory (Optiver Singapore Trading Pte. Ltd. CMS licence)
- rulebook.sgx.com — SGX-ST Rules Chapters 5 and 6 (Designated Market-Maker)
- links.sgx.com — SGX RegCo announcements (ETF DMM streamlining, 5 Sep 2025; Flow Traders Asia DMM-obligation notice, Mar 2025)
- sgx.com — Co-Location page; derivatives pages; group structure (via Wikipedia)
- flowtraders.com; imc.com; janestreet.com; jumptrading.com; sig.com; tower-research.com; hudsonrivertrading.com; xtxmarkets.com; drw.com; alphagrep.com; qcp.capital
- ACRA records via sgpbusiness.com, opengovsg.com and EMIS (incorporation dates for Flow Traders Asia, Jump Trading Pacific, Optiver Singapore)
- CFTC press releases 5521-08 and 6239-12 (Optiver); DOJ/Bloomberg/WSJ (Tower spoofing); Bloomberg (HRT, Jane Street, Optiver Sydney pay)
- The Business Times (Jane Street Singapore expansion, 2025); The TRADE (Flow Traders Asia, XTX/MAS pricing engine); Markets Media and MarketScreener (SGX Reach); NVIDIA case study (SGX data centre); Allen & Gledhill (ETF DMM streamlining)
- Wikipedia (Optiver; Singapore Exchange; Flow Traders; IMC Financial Markets; Jane Street Capital; Jump Trading; Susquehanna International Group; Tower Research Capital; Hudson River Trading; XTX Markets; DRW Trading Group) — used for cross-checking and citation chains into primary press

---

## 14. Closing Summary

Singapore's market-making landscape is a compact, high-velocity ecosystem built on a precise scaffold: an exchange group (SGX, with SGX-ST, SGX-DT, CDP and SGX-DC) that invested S$250 million in an ultra-low-latency engine and a co-location data centre; a designated-market-maker regime under SGX-ST Rules Chapters 5 and 6, streamlined for ETFs in December 2025; and a MAS licensing and conduct overlay that every firm in this guide sits inside. Optiver is the reference firm for good reason — the Amsterdam pioneer of 1986 that now quotes on 50+ exchanges, opened its Singapore office in 2021 (not 2008–2010, as a naive reading would guess), and runs its Asian FICC book from CapitaGreen under a CMS licence that the guide verified directly in MAS's public directory. The peer set resolves cleanly into verified fact and honest uncertainty: Flow Traders has been an SGX ETF market maker since 2007 and is the only firm whose SGX DMM registration is publicly named; Jump's Singapore entity dates to 2011; DRW and XTX publish their Singapore addresses; IMC and SIG list no Singapore office at all; and the office years for Jane Street, Tower, HRT, XTX and DRW in Singapore are not publicly disclosed — flagged ⚠ rather than guessed. The home-grown tier is thin but real: AlphaGrep (2010, Indian-origin, Singapore-headquartered) and QCP Capital (2017, Singapore's digital-asset pioneer). For Cymbal Bank, the commercial translation is the worked example: a market maker is the highest-intensity prime-brokerage client there is — portfolio margin that moves intraday, clearing access to SGX-DC that only a clearing-member bank can sell, settlement that must recycle cash within the day, and a KYC/AML overlay shaped by the firms' own enforcement records. The technology that makes the whole machine run — co-location, lock-free C++, FIXP/FAST encodings, DDS data distribution — lives in the sibling guides and is cross-referenced, not re-derived. What this guide verified, it verified against primary sources; what it could not, it flagged. The result is a map of who stands between buyers and sellers in Singapore — and of the infrastructure, the regulation and the talent that keep them all at the liquidity benches.
