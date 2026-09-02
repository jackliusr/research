# TD Securities: The Software Systems Landscape — A Comprehensive Guide to the Technology TD's Investment Bank Runs

*A companion deep-dive in the per-bank software-systems series of the [jackliusr/research](https://github.com/jackliusr/research) repository — the TD Securities entry alongside [Citibank](citibank_software_systems_guide.md), [JPMorgan Chase](jpmorgan_chase_software_systems_guide.md), [Morgan Stanley](morgan_stanley_software_systems_guide.md), [DBS](dbs_software_systems_guide.md), and [Standard Chartered](standard_chartered_guide.md). This guide focuses on the **specific software and technology systems** behind TD Securities, the investment-banking and capital-markets arm of Toronto-Dominion Bank (TD Bank Group): the markets platforms (TDFX, TD One Portal, TDS Automated Trading), the equities and research estate acquired with TD Cowen, the global transaction banking platforms (TDFacilitrade, the FI banking network), the electronic fixed-income market-making build-out (Headlands), the core/legacy themes of the parent group, and the risk & regulatory context — above all the October 2024 U.S. anti-money-laundering settlement that reshaped the group's U.S. franchise — what is publicly documented, what is inferred from industry practice, and what TD simply does not disclose.*

**Verification convention used throughout: ✅ = verified in this research pass (primary or secondary sources); ⚠ = flagged (inferred, approximate, single-source, or structural inference); ❌ = disputed (the record contradicts the claim); unmarked = structural/industry knowledge presented as such. The consolidated [Claims-Status table is in §11](#11-claims-status-and-verification-notes), and the non-public specifics are collected in [§12](#12-what-could-not-be-verified).**

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Banking Domain / Software-Systems Focus — the technology estate of TD Securities (the investment-banking arm of Toronto-Dominion Bank / TD Bank Group): markets platforms, fixed income & equities infrastructure, global transaction banking, data, core/legacy themes, risk & regulatory context (the 2024 U.S. AML settlement), the Singapore franchise
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026
> **Companion guides:** [Citibank Software Systems Guide](citibank_software_systems_guide.md), [JPMorgan Chase Software Systems Guide](jpmorgan_chase_software_systems_guide.md) and [Morgan Stanley Software Systems Guide](morgan_stanley_software_systems_guide.md) (the structural models for this series), [Core Banking Systems Guide](core_banking_systems_guide.md), [Payment Rails Guide](payment_rails_guide.md), [FIX Protocol Guide](fix_protocol_guide.md), [DDS Guide](../technology/dds_guide.md), [Market Data Consumption Guide](market_data_consumption_guide.md), [Treasury & ALM Guide](treasury_alm_guide.md), [Enterprise Risk Management Guide](enterprise_risk_management_guide.md), [FircoSoft Guide](fircosoft_guide.md), [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md)

---

## Table of Contents

1. [Bank Profile: From the Bank of Toronto to TD Securities](#1-bank-profile-from-the-bank-of-toronto-to-td-securities)
2. [The Systems Landscape: What Is Public and What Is Not](#2-the-systems-landscape-what-is-public-and-what-is-not)
3. [Markets Platforms: TDFX, TD One Portal, and the Electronic Trading Estate](#3-markets-platforms-tdfx-td-one-portal-and-the-electronic-trading-estate)
4. [Fixed Income and Equities Infrastructure: TDS Automated Trading and TD Cowen](#4-fixed-income-and-equities-infrastructure-tds-automated-trading-and-td-cowen)
5. [Global Transaction Banking: Cash Management, FI Banking, and Trade Platforms](#5-global-transaction-banking-cash-management-fi-banking-and-trade-platforms)
6. [Treasury, Rates, and ALM Context](#6-treasury-rates-and-alm-context)
7. [Data, Core, and Legacy Themes: The TD Bank Group Technology Estate](#7-data-core-and-legacy-themes-the-td-bank-group-technology-estate)
8. [Risk and Regulatory Context: The October 2024 AML Settlement](#8-risk-and-regulatory-context-the-october-2024-aml-settlement)
9. [The Singapore Angle](#9-the-singapore-angle)
10. [Worked Example: Cymbal Bank × TD Securities — Correspondent Banking and Markets](#10-worked-example-cymbal-bank--td-securities--correspondent-banking-and-markets)
11. [Claims Status and Verification Notes](#11-claims-status-and-verification-notes)
12. [What Could Not Be Verified](#12-what-could-not-be-verified)
13. [Glossary](#13-glossary)
14. [References and Further Reading](#14-references-and-further-reading)

---

## 1. Bank Profile: From the Bank of Toronto to TD Securities

### 1.1 Scope and Verification Convention

This guide is the **software-systems deep-dive for TD Securities** — the investment-banking and capital-markets arm of Toronto-Dominion Bank ("TD Bank Group", TSX/NYSE: TD), one of Canada's Big Five banks and a global systemically important bank (G-SIB) ✅. Because the repository has no separate TD *bank* guide, this entry carries both the verified bank profile (history, footprint, divisions — §1) and the systems landscape (§2–§7), followed by the risk & regulatory context (§8), the Singapore angle (§9), a worked Cymbal Bank example (§10), and the honest claims audit (§11–§12).

The verification discipline is the same one this series applies to every bank: **✅ verified** means the claim was confirmed in this research pass against a primary source (TD Securities' own pages, TD Bank Group disclosures, FinCEN/OCC/Federal Reserve/DOJ enforcement documents) or a strong secondary source (established financial press, Wikipedia where it cites primary material). **⚠ flagged** means the claim is inferred, approximate, single-source, or structurally reconstructed — the reader should treat it as a hypothesis about a class of system, not a fact about TD. **❌ disputed** marks claims where the public record contradicts the common telling. The [What Could Not Be Verified section](#12-what-could-not-be-verified) collects every materially non-public item.

### 1.2 What Is Public: The TD Disclosure Reality

TD Securities sits on the **opaque end of the spectrum** this series has documented. It is **far less transparent than Citi** about its client platforms: TD does not publish a CitiDirect-BE-style flagship corporate channel, and most of its electronic client surfaces (TDFX, TD One Portal, TDS Automated Trading) are named on its website but documented thinly, with no published architecture, volumes, or roadmap. There is **no TD equivalent of Citi's Velocity 3.0 launch narrative** or JPMorgan's Athena story — no public single-dealer-platform consolidation announcement, no named core migration, no published technology-spend split for the securities division.

What *is* public and verifiable: the **business-line structure** (Corporate & Investment Banking, Global Markets, Global Transaction Banking, Research & Strategy, Sustainable Finance & Advisory), a set of **named electronic products** (TDFX, TD One Portal, TDFacilitrade, TDS Automated Trading, the structured-notes portal), the **acquisition trail** (Newcrest 2000, Headlands 2021, Cowen 2023) that reveals strategy, and — unusually for this series — a **deep enforcement record** in the United States: the October 2024 AML settlement put TD's U.S. compliance systems, monitoring gaps, and remediation requirements into the public record in extraordinary detail (§8).

The consequence, enforced rigorously throughout: the **product layer** of the TD stack (business lines, named platforms, client surfaces) is mostly ✅-verifiable at the level of *existence and positioning*; the **engine layer** (pricing, OMS/EMS, core, hubs, data platform, risk systems) is almost entirely ⚠-inferred, with the notable exception of the U.S. transaction-monitoring estate, which the 2024 enforcement record forced into the open.

### 1.3 The Verified History: 1855 → 2026

The TD lineage runs through two 19th-century Toronto banks, and every load-bearing date is verified ✅:

- **1855** — the **Bank of Toronto** is founded ✅ (TD Bank Group's own history, cited via Wikipedia: the predecessor banks "were established in the mid-19th century, the former in 1855 and the latter in 1869"). The Bank of Toronto was a commercial bank built around the grain and mercantile trade of Upper Canada ⚠ (secondary characterisation; the founding year is the verified fact).
- **1869** — the **Dominion Bank** is founded ✅.
- **1954–1955** — the two banks agree to merge: the agreement is reached in 1954, accepted by the Canadian Minister of Finance on **November 1, 1954**, and made official on **February 1, 1955**, creating the **Toronto-Dominion Bank** ✅. The merged bank became the **fourth-largest bank in Canada with assets of C$1.1 billion** ✅. The merger's motivation — two smaller banks combining to compete with the larger ones — is documented in TD's own history ✅.
- **1967** — TD opens its new head office, the **Toronto-Dominion Centre** in downtown Toronto ✅.
- **1968** — TD partners with **Chargex**, the credit-card association that became Visa ✅.
- **1976** — TD pilots its first ATM, the **TD 360**, which is renamed the **Green Machine** — "a name it continues to carry" ✅. (The green machine is TD's ATM brand, not a system this guide documents further; it is also the brand colour of the group, and this guide's closing line returns to it.)
- **1987** — **Toronto Dominion Securities Inc.** is established as the bank's securities arm ✅ — the founding year this guide's title rests on. Two independent confirmations: TD's own history ("In 1987, Toronto Dominion Securities Inc. was established by the bank") and TD Securities' own website ("Since opening our securities arm in 1987, we have developed extensive capabilities across crucial industries"). ⚠ **Note on the "Gardiner Watson" lead:** the task brief suggested checking a 1987 Gardiner Watson acquisition; this pass found **no primary source confirming a Gardiner Watson acquisition in 1987**, and the two verified sources above date the securities arm's founding to 1987 without reference to it. The Gardiner Watson connection is treated as unverified lore (see §11).
- **1996** — TD acquires **Waterhouse Investor Services**, building the discount-brokerage franchise later sold to Ameritrade (2006) to form TD Ameritrade ✅.
- **1998** — the proposed TD–CIBC merger is **blocked by the Government of Canada** ✅ — the same policy decision that blocked the BMO–RBC merger.
- **2000** — **Toronto-Dominion Securities buys Newcrest Capital for C$224 million (75% in stock, 25% in cash)** ✅, expanding its Canadian investment-banking and capital-markets presence. In the same year TD acquires **Canada Trust**, rebranding most of its Canadian commercial banking as TD Canada Trust ✅.
- **2002** — TD acquires **Stafford Trading** and **Letco Trading** ✅ (electronic/derivatives market-making heritage, per the historical record).
- **2004–2008** — TD builds its U.S. retail franchise: majority stake in **Banknorth** (2004, US$3.8B), rebranded **TD Banknorth** (2005), full ownership (April 2007), acquisition of **Commerce Bancorp** (2007), and the 2008 merger of the two into **TD Bank, N.A.** (legal name effective May 31, 2008) ✅.
- **2011** — TD acquires **MBNA Canada's credit-card business** (December 1, 2011) ✅.
- **2019** — the **Financial Stability Board designates TD a G-SIB** ✅.
- **March 2021** — TD Securities announces a definitive agreement to acquire **Headlands Tech Global Markets, LLC**, the quantitative fixed-income trading firm; the deal **closes July 2021** and the business is renamed **TDS Automated Trading** ✅ (TD Securities' own automated-trading page — primary source).
- **February 28, 2022** — TD makes a **US$13.4 billion offer for First Horizon Corp.**, the U.S. regional bank ✅. **May 4, 2023** — TD announces the deal **will not proceed due to regulatory uncertainty** ✅ — the first major casualty of the AML investigations that culminated in 2024 (§8).
- **August 2, 2022** — TD agrees to acquire **Cowen Inc.** for **US$1.3 billion in cash (US$39 per share)** ✅; the deal **closes in March 2023** and Cowen is rebranded **TD Cowen**, a division of TD Securities ✅. To fund the purchase TD sells over 28 million Charles Schwab shares, cutting its stake from 13.4% to 12% ✅.
- **October 10, 2024** — the **U.S. AML settlement**: TD Bank N.A. and TD Bank USA N.A. resolve coordinated actions with the DOJ, FinCEN, the OCC, and the Federal Reserve for approximately **US$3.09 billion** in total announced penalties (§8) ✅.
- **February 2025** — **Raymond Chun becomes CEO of TD Bank Group** (Bharat Masrani's retirement was announced September 2024) ✅; TD announces its intention to sell its remaining **10.1% Charles Schwab stake for about US$14.6 billion**, with proceeds earmarked for buybacks and U.S. simplification — explicitly following the US$3.09B money-laundering fine ✅.
- **2025–2026** — TD Bank, N.A. remains under the OCC asset cap and the monitorships of §8; in 2026 the group's Canadian mobile banking app is ranked **first among Canadian banks** in the J.D. Power Canada Digital Banking & Credit Card Mobile App Satisfaction Study (score 690) ✅ — a small, verifiable technology signal in an otherwise closed estate.

### 1.4 The Global Footprint

- **TD Securities: 40 cities / 6,500+ professionals** ✅/⚠ — Wikipedia's infobox ("offices in 40 cities worldwide with over 6,500 professionals") versus TD Securities' own "By the Numbers" (16 offices in key financial centres; 4,900+ full-time employees worldwide, as of October 2021) ⚠ — the two figures differ because they measure different things (all offices vs key financial centres; 2021 vs current headcount); both are reported, neither is authoritative for 2026.
- **TD Securities offices** ✅/⚠ — the verified anchors from public sources: **Toronto** (headquarters, Toronto-Dominion Centre), **New York** (the U.S. hub, 31 West 52nd Street is the reported address ⚠), **London**, **Singapore**, **Hong Kong**, **Tokyo**, **Sydney**, **Calgary**, **Montreal**, **Vancouver**, **Chicago**, **Houston**, **Boston** — the list matches the task brief and is consistent with TD Securities' "Find us worldwide" positioning ✅/⚠ (the office-by-office list is assembled from secondary sources and the firm's own "key financial centres" framing; the exact current list is not published as a stable inventory ⚠).
- **TD Bank Group: 102,218 FTE employees, C$2.095 trillion in assets (2025), ~28 million customers** ✅ (Wikipedia, citing TD's annual reporting). **TD Bank, N.A.: US$366 billion in assets (2025), 1,158 locations across 15 U.S. states and Washington D.C., headquartered in Mount Laurel, New Jersey** ✅ — the U.S. retail bank that anchors §8's regulatory story and carries the AML remediation burden.
- **The Asia-Pacific network** ✅/⚠ — TD Securities operates across North America, Europe, and Asia-Pacific (Wikipedia); the specific regional hubs are Toronto/London/Singapore for the Financial Institutions transaction-banking business ✅ (TD Securities' GTB page names exactly these three cities for its FI team).

### 1.5 The Business Lines

TD Securities' public structure (tdsecurities.com) is a five-line capital-markets stack ✅:

| Business line | Scope (verified publicly) | Systems that serve it (this guide) |
|---|---|---|
| **Corporate & Investment Banking** | M&A advisory, equity and debt capital markets, corporate lending | Client-facing banking/portal surfaces ⚠ (§3 cross-refs) |
| **Global Markets** | Rates, FX, credit, equities, commodities, prime brokerage, securitization, electronic execution | TDFX, TD One Portal, TDS Automated Trading, FIX (§3–§4) |
| **Global Transaction Banking** | Corporate cash management, FI banking (Toronto/London/Singapore), trade & working capital, export & agency finance | TDFacilitrade, FI payment/clearing services (§5) |
| **Research & Strategy** | Equity research, fixed-income research, economics | TD One Portal research surfaces ⚠ (§3) |
| **Sustainable Finance & Advisory** | Green/social bond underwriting, ESG advisory | — |

Verified leadership anchors ✅: **Tim Wiggan, President & CEO of TD Securities** (Wikipedia infobox); **David Swinburne, Managing Director and Global Head of Commodities**; **Martin Mannion and Matthew Schrager, Managing Directors and Co-Heads of TDS Automated Trading**; **Michael Stockton, Global Head of Export & Agency Finance**; **Cavin Wong, Regional Head of Global Transaction Banking, Asia-Pacific** (all from TD Securities' own pages). At the group level, **Raymond Chun is CEO of TD Bank Group** (February 2025) ✅.

### 1.6 The Consolidated Technology Stack (2026 View)

The whole landscape at a glance — the map that §2–§7 then unpack. Evidence class per the §11 table:

| Layer | Systems (names as publicly known) | Evidence class |
|---|---|---|
| **Markets client surfaces** | TDFX (eFX portal); TD One Portal (research/securitization reports); structured-notes portal; equities electronic trading (global electronic trading offering) | ✅ names & positioning verified; ⚠ internals |
| **Electronic market-making** | TDS Automated Trading (ex-Headlands Tech Global Markets, 2013→2021): automated market-making in municipal and investment-grade corporate bonds | ✅ acquisition & mandate verified; ⚠ platform internals |
| **Global Transaction Banking** | TDFacilitrade (trade finance); FI banking network (Toronto/London/Singapore): CAD/USD liquidity, CAD payment clearing, CLS service, daylight credit | ✅ products verified; ⚠ engines |
| **Equities/research (TD Cowen)** | TD Cowen broker-dealer: IB, equity/credit research, sales & trading, prime brokerage, outsourced trading, global clearing | ✅ division scope verified; ⚠ platforms |
| **FIX/connectivity** | FIX-based client and venue connectivity | ⚠ structural (class certain, catalogue not public) |
| **Data** | Market data vendors (Bloomberg/LSEG-class) ⚠; research content distribution via TD One Portal | ⚠ not public |
| **Core/legacy (group)** | TD Bank Group's large in-house IT organization, mainframe-era retail/core estate, the U.S. AML remediation programme (transaction monitoring, SAR/CTR reporting, lookback) | ✅ remediation scope verified via enforcement record; ⚠ system identities |
| **Risk & compliance** | U.S. transaction monitoring, SAR/CTR reporting, sanctions screening; the 2024 consent-order remediation estate | ✅ deficiencies verified via FinCEN/OCC/DOJ documents; ⚠ vendor/tooling details |

The pattern to hold for the rest of this guide: **TD Securities' client-facing products are named and verifiable at the surface; the engines underneath them are not.** Every section below states exactly which of its claims are ✅ and which are ⚠.

### 1.7 Key Milestones Timeline

| Year | Milestone | Status |
|---|---|---|
| 1855 | Bank of Toronto founded | ✅ |
| 1869 | Dominion Bank founded | ✅ |
| 1955 | Merger creates Toronto-Dominion Bank (Feb 1); 4th largest in Canada | ✅ |
| 1976 | TD 360 ATM piloted; renamed "Green Machine" | ✅ |
| 1987 | Toronto Dominion Securities Inc. established (TD Securities founded) | ✅ |
| 1996 | Waterhouse Investor Services acquired (TD Waterhouse) | ✅ |
| 2000 | Newcrest Capital acquired (C$224M); Canada Trust acquired | ✅ |
| 2002 | Stafford Trading and Letco Trading acquired | ✅ |
| 2004–2008 | U.S. build-out: Banknorth → TD Banknorth → TD Bank, N.A. (2008) | ✅ |
| 2011 | MBNA Canada card business acquired (Dec 1) | ✅ |
| 2019 | G-SIB designation by the FSB | ✅ |
| 2021 | Headlands Tech Global Markets acquired (closed Jul 2021) → TDS Automated Trading | ✅ |
| 2022 | First Horizon US$13.4B offer (Feb 28); Cowen US$1.3B agreement (Aug 2) | ✅ |
| 2023 | First Horizon terminated (May 4); Cowen closes (Mar) → TD Cowen | ✅ |
| 2024 | U.S. AML settlement, ~US$3.09B announced total (Oct 10); Schwab stake sale plans | ✅ |
| 2025 | Raymond Chun becomes group CEO (Feb); Schwab 10.1% stake sale (~US$14.6B) | ✅ |
| 2026 | J.D. Power #1 Canadian bank mobile-app satisfaction; remediation under monitorships continues | ✅/⚠ |

The timeline reads in four eras, and the systems story maps onto the last two. **Charter era (1855–1954):** the Bank of Toronto and the Dominion Bank build the balance sheet and branch network that become TD. **Consolidation era (1955–1999):** the 1955 merger, the Toronto-Dominion Centre, the Green Machine ATMs, and the blocked 1998 CIBC merger shape the modern group. **Securities-build era (1987–2021):** TD Securities is founded (1987), buys Newcrest (2000) and the trading houses (2002), builds the U.S. retail franchise (2004–2008), and buys Headlands (2021) for electronic fixed income. **Regulated-modernization era (2022–2026):** the First Horizon collapse (2023), the Cowen acquisition (2023), the October 2024 AML settlement (§8), and the Schwab exit (2025) — the era the rest of this guide documents in depth. The systems-relevant rows — 1987, 2000, 2002, 2021, 2022–2025 — are the anchors the §2–§8 sections hang their verified claims on.

---

## 2. The Systems Landscape: What Is Public and What Is Not

### 2.1 The Honest Opening Statement

**TD Securities does not publish an internal system map.** There is no public inventory of the platforms behind its trading desks, no named core or OMS/EMS, no published data-centre or cloud architecture, and no technology-spend disclosure specific to the securities division ⚠. What the public record *does* support is a **layered description**: (1) a verified **product layer** of named, marketed client surfaces (TDFX, TD One Portal, TDFacilitrade, TDS Automated Trading, the structured-notes portal — §3–§5); (2) a verified **acquisition layer** that reveals strategy (Headlands 2021 for electronic fixed income, Cowen 2023 for U.S. equities/research/prime brokerage — §4); and (3) an **enforcement layer** that forced the U.S. retail bank's transaction-monitoring and reporting estate into the open (§8). Everything else — pricing engines, order management, risk systems, the connectivity catalogue — is ⚠ inferred industry-practice architecture.

The discipline this section enforces, and the rest of the guide repeats: **a named TD product is ✅-verifiable as a product; its engine is ⚠-unknowable from public sources.** The architect should read every "platform" claim below with that split in mind.

### 2.2 The Verified Product Layer

The following client-facing names are verified against TD Securities' own pages ✅ (each is expanded in §3–§5):

| Product | What it is (as publicly described) | Evidence |
|---|---|---|
| **TDFX** | TD Securities' electronic foreign-exchange portal for clients (deal.tdsecurities.com/tdfx) — part of the FX offering alongside spot, forwards, NDFs, options, and swaps | ✅ TD Securities Global Markets page |
| **TD One Portal** | The client portal (portal.tdsecurities.com) through which subscribing clients access securitization reports and documentation; also the research/content surface (the portal's "alpha" content paths serve strategy and research content) | ✅ TD Securities pages; portal URL verified |
| **TDS Automated Trading** | The electronic market-making business formed from Headlands Tech Global Markets (founded 2013; acquired 2021): fully automated electronic market-making in municipal and investment-grade corporate bonds, with quantitative strategies across financial products | ✅ TD Securities Automated Trading page |
| **TDFacilitrade** | The trade-finance product brand covering import/export/standby letters of credit and documentary collections | ✅ TD Securities GTB page |
| **Structured Notes portal** | The client surface for structured notes (portal.tdsecurities.com/notes) | ✅ TD Securities Equities page link |
| **Global Electronic Trading** | The equities electronic-trading offering (execution services across the equities business) | ✅ TD Securities Equities page (positioning only) |

**The "TD ThinkPipes" question** ⚠ — the task brief identified "TD ThinkPipes" as a public electronic fixed-income platform name. This research pass could **not verify that name at any primary source**: it does not appear on TD Securities' current pages (the fixed-income offering is described via "Automated Trading", "Credit Products", "Interest Rate Products", "Money Markets", "Municipal Products", and "Securitized Products"), and no archived tdsecurities.com URL containing "pipes" was found in the Wayback Machine. The name is flagged ⚠ in §11 and treated in this guide as **reported/unverified** — readers should not cite it as a confirmed TD product name without a primary source. ⚠ **Similarly unverified:** "Autex"-style block-trading platform branding for TD (the equities page lists "Global Portfolio Trading" and "Global Electronic Trading" without naming a block-trading system), and any public execution-algorithm catalogue.

### 2.3 The Inferred Engine Layer (Structural)

Below the product layer sits the engine layer, which is **structurally certain as a class and unknowable as an instance** ⚠. A capital-markets estate of TD Securities' size (6,500+ professionals across 40 cities; C$8.099B revenue in 2023 ✅/⚠ Wikipedia-reported) requires, as a matter of industry practice:

- **Pricing and risk engines** for rates, FX, credit, commodities, and equities — the class of system documented in the [Capital Markets Architecture Guide](capital_markets_architecture_guide.md) ⚠ (vendor identities not public).
- **OMS/EMS and order routing** for the cash equities, ETF, options, and delta-one businesses, connected to venues over **FIX** — the protocol mechanics are **not re-derived here**; they live in the [FIX Protocol Guide](fix_protocol_guide.md) ⚠ (TD's FIX connectivity catalogue is not published).
- **Market data infrastructure** — the feeds, entitlements, and tick capture that every trading floor runs; the consumption mechanics are **not re-derived here**; they live in the [Market Data Consumption Guide](market_data_consumption_guide.md) and the [DDS Guide](../technology/dds_guide.md) (for the brokerless data-bus pattern that trading floors use) ⚠ (TD's vendors and topology are not public; §7).
- **The transaction-banking estate** — the payments, clearing, and trade engines behind the GTB business, sitting on the correspondent rails documented in the [Payment Rails Guide](payment_rails_guide.md) ⚠.
- **The group core** — the retail/commercial core banking estate of the parent group, documented by cross-reference in §7 ⚠.

The honest statement for §2: **every engine row above is a class, not a TD fact.** The remainder of this guide names the few instances the public record supports (TDS Automated Trading's mandate, the TD Cowen division scope, the 2024 enforcement record) and flags everything else.

### 2.4 Cross-References (Not Re-Derived)

Per this series' division of labour, the following mechanics are **cross-referenced, not re-derived**, throughout this guide:

- **FIX protocol** (sessions, order flow, message standards) — [FIX Protocol Guide](fix_protocol_guide.md)
- **Market data value chain** (venues → vendors → platforms → decisions) — [Market Data Consumption Guide](market_data_consumption_guide.md)
- **Data-centric low-latency distribution** (the DDS brokerless bus pattern trading floors use) — [DDS Guide](../technology/dds_guide.md)
- **Payment rails** (SWIFT, ISO 20022, GPI, correspondent banking, nostro mechanics, clearing) — [Payment Rails Guide](payment_rails_guide.md)
- **Treasury & ALM** (funding, FTP, liquidity, IRRBB) — [Treasury & ALM Guide](treasury_alm_guide.md)
- **Enterprise risk management** (three lines of defence, risk appetite, ICAAP) — [Enterprise Risk Management Guide](enterprise_risk_management_guide.md)
- **Sanctions screening / AML mechanics** (watch-list filtering, name matching, FircoSoft's product family) — [FircoSoft Guide](fircosoft_guide.md)
- **Singapore regulation** (MAS licensing, notices, TRMG, outsourcing) — [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md)
- **Core banking systems** (core platform types, vendor landscape, modernization) — [Core Banking Systems Guide](core_banking_systems_guide.md)

---

## 3. Markets Platforms: TDFX, TD One Portal, and the Electronic Trading Estate

### 3.1 TDFX: The Electronic FX Portal

**TDFX** is TD Securities' client-facing electronic foreign-exchange platform ✅ — the only TD electronic *trading* portal whose URL is published on TD's own Global Markets page (deal.tdsecurities.com/tdfx, under the Foreign Exchange offering alongside spot, forwards, NDFs, options, and swaps). What the public record supports:

- **Existence and positioning** ✅ — TDFX is listed as part of the FX product suite for corporate and institutional clients; the FX page describes 24-hour global coverage, G10 and emerging-market expertise, and reliable execution and settlement.
- **Functionality** ⚠ — TD publishes **no feature documentation** for TDFX (no workflow descriptions, no API/FIX connectivity catalogue, no supported-product list). What is structurally certain for an eFX portal of this class ⚠: streaming executable quotes, deal tickets with full trade capture, and post-trade confirmations — the standard SDP/e-commerce FX pattern.
- **The "TD ThinkPipes" confusion** ⚠ — see §2.2: the reported fixed-income platform name could not be verified; TDFX is the verified electronic-trading name in the public record.
- **FIX connectivity** ⚠ — an eFX portal of this class is FIX-connected as a matter of industry practice ([FIX Protocol Guide](fix_protocol_guide.md)); TD's specific connectivity catalogue is not public (§12).

### 3.2 TD One Portal: The Client Content and Documentation Surface

**TD One Portal** (portal.tdsecurities.com) is TD Securities' client portal ✅:

- **Verified uses** ✅ — TD's Securitization page: "Subscribing clients can access reports and documentation through the TD One Portal"; the equities page links the **Structured Notes** surface at portal.tdsecurities.com/notes.
- **The "alpha" content paths** ✅/⚠ — archived portal URLs (portal.tdsecurities.com/alpha/content/strategy/…) show a content-management structure serving strategy and research content by numeric IDs — consistent with a research-distribution surface ⚠ (the archived paths are real URLs; the content behind them is client-gated).
- **What it is not** ⚠ — TD One Portal is *not* documented as a transaction platform: no payment, trade, or cash-management functionality is publicly described for it. It belongs to the research/content/documentation class, not the transaction-banking class (§5).
- **The equities electronic-trading surface** ⚠ — TD's equities page lists "Global Electronic Trading" and "Global Portfolio Trading" as offerings but publishes no portal name for them; whether equities execution runs through TD One Portal or a separate surface is not public (§12).

### 3.3 The Electronic Trading Estate: What the Acquisition Trail Reveals

TD's electronic-markets strategy is best read from its acquisitions, which are ✅-verified:

- **Headlands Tech Global Markets (2021)** ✅ — founded 2013, acquired by TD Securities in a deal announced March 2021 and closed July 2021; proprietary software for **fully automated electronic market-making in municipal and investment-grade corporate bonds**; renamed **TDS Automated Trading** and folded into TD's integrated Global Fixed Income group (TD Securities' own page — primary source). This is the single clearest public statement of TD's electronic fixed-income architecture: a quantitative, automated market-making platform for U.S. credit products (§4.1).
- **Stafford Trading and Letco Trading (2002)** ✅ — market-making acquisitions in the derivatives/electronic space (Wikipedia history; the specific platforms were absorbed into TD's trading estate ⚠).
- **Cowen (2023)** ✅ — brought TD Cowen's U.S. equities execution, research, and prime-brokerage platforms (§4.2).
- **The 2002–2026 pattern** ⚠ — TD's electronic strategy is *buy rather than build* at the platform layer (Headlands for credit market-making, Cowen for equities), while the client surfaces (TDFX, TD One Portal) are in-house branded. This is a structural inference from the verified acquisition record, not a disclosed strategy ⚠.

### 3.4 The Markets Stack: Public vs Not

| Component | Status in the public record |
|---|---|
| eFX portal: **TDFX** (deal.tdsecurities.com/tdfx) | ✅ verified (TD Securities FX page) |
| Client portal: **TD One Portal** (portal.tdsecurities.com); structured-notes surface | ✅ verified (TD pages); ⚠ internals |
| **TDS Automated Trading** (ex-Headlands): automated muni/IG-corporate market-making | ✅ verified (TD page; acquisition) |
| **Global Electronic Trading / Global Portfolio Trading** (equities) | ✅ verified as offerings; ⚠ no platform name |
| "**TD ThinkPipes**" fixed-income platform name | ⚠ not verified in this pass (§11) |
| FIX and API connectivity catalogue | ⚠ structural (class certain, catalogue not public) |
| Pricing engines, OMS/EMS, algorithmic stack, market data infrastructure | ⚠ not public (§12) |
| Equities block/Autex-style trading platform branding | ⚠ not verified (§12) |

The table's dividing line is the guide's general one: every row above the line is a named, marketed TD product; every row below it is a *class* of system that the products require to function, whose vendor identities and internal names are not published (§12). For the architect this means the markets estate is describable at the surface — product names, mandates, acquisition trail — and opaque at the engine.

---

## 4. Fixed Income and Equities Infrastructure: TDS Automated Trading and TD Cowen

### 4.1 TDS Automated Trading: The Electronic Credit Market-Maker

The fixed-income electronic infrastructure is the best-documented engine in TD's markets estate, thanks to the Headlands acquisition ✅:

- **The asset** ✅ — Headlands Tech Global Markets, LLC (HTGM), founded 2013, built proprietary software for **fully automated electronic market-making in municipal and investment-grade corporate bonds** — i.e., a quantitative, algo-driven pricing and quoting platform for U.S. credit products, the class of system that replaced manual bond dealing on dealer platforms.
- **The integration** ✅ — after the July 2021 close, HTGM became **TDS Automated Trading**, part of TD Securities' integrated Global Fixed Income group, tasked with "implementing quantitative trading strategies across a range of financial products" and accelerating TD's "data-driven innovation and technology strategy" (TD's own words).
- **Leadership** ✅ — Martin Mannion and Matthew Schrager, Managing Directors and Co-Heads of TDS Automated Trading (TD Securities biographies).
- **The engine layer** ⚠ — TD publishes no detail on the HTGM platform itself: no latency figures, no venue-connectivity list, no architecture. The *class* (automated RFQ/streaming market-making for muni/IG credit) is verified; the *instance* is not (§12).
- **Why it matters to the systems map** ✅/⚠ — TDS Automated Trading is the verified answer to the question "does TD run electronic fixed-income market-making?" — yes, in U.S. municipal and investment-grade credit, since 2021. It is also the closest thing to a named TD trading *engine* in the public record — a rare ✅ in this guide's engine layer.

### 4.2 TD Cowen: The U.S. Equities, Research, and Prime Brokerage Platform

The Cowen acquisition (announced August 2, 2022; closed March 2023; US$1.3B, US$39/share) gave TD Securities a full U.S. equities and advisory platform under the **TD Cowen** brand ✅:

- **Division scope** ✅ (TD Cowen's public profile): investment banking, **equity and credit research**, sales and trading, **prime brokerage**, outsourced trading, global clearing, and commission management — plus an investment-management segment (alternative investments). Cowen's heritage (founded 1918; SG Cowen 1998–2006 under Société Générale; Nasdaq: COWN after the 2006 IPO) is documented ✅.
- **The research franchise** ✅/⚠ — TD Cowen's equity and credit research is the public backbone of TD's Research & Strategy line (§1.5); the distribution surface is TD One Portal-class ⚠ (§3.2). TD Cowen's research in emerging industries (the cannabis-sector leadership is Cowen's own claim) is a business fact, not a systems fact ✅/⚠.
- **Prime brokerage** ✅/⚠ — TD Securities' Global Markets page confirms prime-brokerage offerings (capital introduction, custody and clearing, financing solutions, securities lending, synthetic prime/portfolio swaps, technology and reporting) delivered through **TD Securities Inc. in Canada and TD Securities (USA) LLC in the U.S.** — with the Cowen heritage providing the U.S. clearing and financing plumbing ⚠ (platform internals not public). The prime-brokerage mechanics themselves are the domain of the [Capital Markets Architecture Guide](capital_markets_architecture_guide.md), cross-referenced, not re-derived.
- **The integration question** ⚠ — how deeply TD Cowen's systems are integrated with TD Securities' Canadian estate (single OMS/EMS? shared risk platform? consolidated clearing?) is **not disclosed**; the public record supports the brand and the division scope, not the integration depth (§12).

### 4.3 The Fixed Income Offering (Public Catalogue)

TD's fixed-income client offering (Global Markets page) ✅: **Automated Trading** (TDS Automated Trading), **Credit Products**, **Interest Rate Products**, **Money Markets**, **Municipal Products**, and **Securitized Products**. The securitization business is publicly positioned as **one of the largest North American asset-backed commercial paper (ABCP) conduit sponsors** ✅, with term ABS structuring/distribution and balance-sheet funding — the product layer of a substantial fixed-income machine whose risk and pricing engines are ⚠ (§12).

### 4.4 Cross-Reference: FIX and Market Data

The protocol and data mechanics behind §4.1–§4.3 are **not re-derived here**: FIX session/order mechanics live in the [FIX Protocol Guide](fix_protocol_guide.md), and the market-data value chain (feeds, vendors, entitlements, tick capture) lives in the [Market Data Consumption Guide](market_data_consumption_guide.md). For TD specifically: the connectivity catalogue of TDS Automated Trading and TD Cowen's execution platforms is not public ⚠, and the market-data vendor contracts (Bloomberg, LSEG/Refinitiv-class) behind TDFX, TD One Portal, and the trading floors are not named in TD's public material ⚠ (§7, §12).

---

## 5. Global Transaction Banking: Cash Management, FI Banking, and Trade Platforms

### 5.1 The GTB Division and Its Public Shape

**Global Transaction Banking (GTB)** is TD Securities' transaction-banking line ✅, with four publicly named practice areas (tdsecurities.com): **Corporate Cash Management**, **Financial Institutions**, **Trade & Working Capital**, and **Export & Agency Finance**. For the rails these sit on — SWIFT, ISO 20022, GPI, local clearing — this guide **cross-references rather than re-derives**: the mechanics live in the [Payment Rails Guide](payment_rails_guide.md). What follows is the TD-specific, publicly verifiable layer only.

### 5.2 Corporate Cash Management

- **Public scope** ✅ — TD's GTB page describes tailored cash-management solutions: **Collections**, **Disbursements**, **Information and Control Services**, and **Liquidity Management and Short-Term Investments** — the standard corporate cash-management catalogue.
- **Platforms** ⚠ — TD publishes **no client channel name** for corporate cash management: there is no TD equivalent of CitiDirect BE or a publicly documented host-to-host/API catalogue (§12). The class of system (an e-banking channel with balances, payments, and reporting; host-to-host file/API integration; maker/checker controls) is industry-standard ⚠ (the maker/checker and host-to-host patterns are documented in the [Payment Rails Guide](payment_rails_guide.md) and the [DBS guide](dbs_software_systems_guide.md) §10.5).

### 5.3 Financial Institutions: The FI Banking Network

This is the practice area most relevant to the §10 worked example, and it is publicly anchored ✅:

- **The network** ✅ — the FI team is "located in **Toronto, London and Singapore**" (TD's own words), delivering international payment clearing, treasury settlement, and liquidity management to bank clients.
- **The catalogue** ✅ — **CAD/USD Liquidity** (reporting, deposit accounts, liquidity management tools, daylight credit) and **CAD Payment Clearing** (treasury settlement, **CLS service**, commercial payments, draft and cash letters). The CLS reference is notable: TD is publicly positioned as a CLS settlement participant for FX PvP settlement ✅/⚠ (the service is named on TD's page; the membership detail is not re-verified here).
- **The FI-client model** ✅/⚠ — a bank client of TD's FI business (like Cymbal Bank in §10) gets CAD/USD liquidity and clearing services — i.e., TD acts as a correspondent for Canadian-dollar and U.S.-dollar flows, the mirror of the correspondent relationships TD itself holds in other currencies ⚠ (structural).
- **What is not public** ⚠ — the FI client portal name, the SWIFT BIC infrastructure behind the clearing services, and the nostro/liquidity engine are all undisclosed (§12).

### 5.4 Trade & Working Capital: TDFacilitrade

- **Public scope** ✅ — **Import Letters of Credit**, **Export Letters of Credit**, **Standby Letters of Credit/Guarantees**, **Documentary Collections**, and the **TDFacilitrade** product brand (tdsecurities.com trade page, with downloadable product documents).
- **TDFacilitrade** ✅/⚠ — the named trade-finance product brand; TD publishes no platform documentation (no workflow, no message-format catalogue) ⚠. The mechanics it rests on — MT700/MT707 for LCs, MT400/MT202 for settlement, UCP 600 document checks, the confirmation economics — are the domain of the [Trade Finance Guide](trade_finance_guide.md) and [Trade Finance Systems Guide](trade_finance_systems_guide.md), cross-referenced, not re-derived.
- **Awards** ✅ — Trade Finance Global "Excellence in Trade (North America)" (2023); shortlisted for Global Trade Review's "Best Trade Finance Bank, Americas" (2023–2024) — TD's own award claims.

### 5.5 Export & Agency Finance

- **Public scope** ✅ — medium- and long-term loan structuring and underwriting (8 to 20+ years) backed by **Export Credit Agencies (ECAs)** and sovereigns/supranationals/agencies (SSAs); Global Head Michael Stockton.
- **Systems** ⚠ — the loan booking, agency-servicing, and risk systems behind this business are not public (§12).

### 5.6 The GTB Systems Map: Public vs Not

| Component | Status |
|---|---|
| GTB practice areas (cash mgmt, FI, trade, export/agency) | ✅ verified (TD page) |
| FI network cities: Toronto, London, Singapore | ✅ verified (TD page) |
| FI catalogue: CAD/USD liquidity, CAD clearing, CLS service, daylight credit | ✅ verified (TD page) |
| **TDFacilitrade** trade brand; LC/standby/collections product documents | ✅ verified (TD page) |
| Corporate cash-management client channel (portal name, host-to-host/API) | ⚠ not public (§12) |
| FI payment/clearing engines, SWIFT infrastructure, nostro book | ⚠ not public (§12) |

---

## 6. Treasury, Rates, and ALM Context

### 6.1 The Treasury Function at TD

The treasury and asset-liability management function of a bank group like TD — funding, funds-transfer pricing, liquidity risk (LCR/NSFR), and interest-rate risk (IRRBB) — is the domain of the [Treasury & ALM Guide](treasury_alm_guide.md), **not re-derived here**. The TD-specific facts that intersect with this guide:

- **Rates is a core TD Securities product line** ✅ — "Interest Rate Products" and "Money Markets" are named fixed-income offerings (§4.3), and the group's balance sheet (C$2.095 trillion in assets, 2025) makes the group treasury one of the largest in Canada ✅/⚠ (scale figure Wikipedia-reported).
- **The 2024 settlement's liquidity angle** ✅/⚠ — the Federal Reserve's October 2024 action requires TD to **certify that sufficient resources are allocated to AML remediation before issuing dividends or capital distributions** (Fed release) — a direct constraint on the group treasury's dividend policy, and a reminder that regulatory remediation now sits inside TD's capital-allocation process ✅ (the *requirement* is primary-source verified; its *execution* inside TD's ALCO process is not public ⚠).
- **CLS and settlement liquidity** ✅/⚠ — the GTB FI catalogue includes CLS service (§5.3), which is a settlement-liquidity matter: CLS is the PvP settlement system that eliminates FX settlement risk ⚠ (the mechanics are in the [Payment Rails Guide](payment_rails_guide.md); TD's membership status is not re-verified here).
- **Treasury systems** ⚠ — TD does not publicly name its treasury workstations, ALM models, FTP engines, or liquidity reporting platforms; the vendor classes (Murex/Calypso-class) are documented in the [Treasury & ALM Guide](treasury_alm_guide.md) and [Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md), but TD's instances are not public (§12).

### 6.2 The Rates Desk in the Public Record

Beyond the product names (§4.3), the rates business is documented only through the awards TD publishes ✅/⚠: **#1 Overall Canadian Fixed-Income Service Quality** (Coalition Greenwich study, 2018–2021) and **#1 Announced M&A Transactions** (Refinitiv, 2017–2021) — market-positioning facts, not systems facts. The rates pricing, hedging, and risk engines are ⚠ (§12).

---

## 7. Data, Core, and Legacy Themes: The TD Bank Group Technology Estate

This section combines the two layers of the TD stack that the public record defines: the **data estate** (market data, data governance, and what the 2024 enforcement record revealed about TD's monitoring data) and the **core/legacy estate** of the parent group (the in-house IT organization, the 2024 AML-driven remediation investment, and the systems consequence for TD Securities). The retail core and the wholesale estate are structurally separate but share the group's risk, finance, and compliance infrastructure (§7.6) ⚠.

### 7.1 Market Data (Cross-Reference)

The market-data value chain — venue feeds, consolidated tapes, vendors (Bloomberg, LSEG/Refinitiv, FactSet, ICE Data Services), feed types, and consumption by trading platforms, portfolio management, and risk — is **not re-derived here**; it lives in the [Market Data Consumption Guide](market_data_consumption_guide.md). The DDS-based, brokerless low-latency distribution pattern that trading floors use is documented in the [DDS Guide](../technology/dds_guide.md). For TD specifically:

- **Vendors** ⚠ — TD's public material does **not name** its market-data vendors. That a capital-markets firm of TD's class consumes Bloomberg and LSEG/Refinitiv-class data is structurally certain ⚠; the contract and entitlement specifics are not public (§12).
- **Content distribution** ✅/⚠ — TD One Portal's research/content surface (§3.2) is the verified client-facing distribution channel for TD's research; the underlying content-management and entitlement platforms are not public ⚠.
- **The automated-trading data angle** ✅/⚠ — TDS Automated Trading's mandate ("data-driven innovation", TD's own words) implies a serious market-data and historical-data estate for the quant credit market-making; nothing about it is published ⚠ (§12).

### 7.2 Data Governance and the Enforcement Shadow

TD's data story, like Citi's (§7 of the [Citibank guide](citibank_software_systems_guide.md)), is dominated by regulator-mandated remediation — but with a different centre of gravity. Where Citi's findings were about *enterprise data governance* (2020 consent orders), TD's 2024 findings are about *transaction data being systematically ignored* ✅:

- **92% of U.S. transaction volume unmonitored** ✅ (DOJ): from January 1, 2018 to April 12, 2024, TD Bank intentionally did not automatically monitor all domestic ACH transactions, most check activity, and numerous other transaction types — **approximately US$18.3 trillion of transaction activity** went unmonitored.
- **No new monitoring scenarios** ✅ (DOJ): no new transaction-monitoring scenarios were added and no material changes made from at least 2014 through late 2022.
- **The data-governance review** ✅ (FinCEN): for the first time, FinCEN imposed **accountability and data governance reviews** — the data governance review is explicitly tasked with "identify[ing] and fix[ing] the root causes of many gaps in its AML program."
- **The SAR lookback** ✅ (FinCEN): an independent consultant under the FinCEN monitor conducts a historical analysis of TD's transaction data ("SAR lookback") to remediate missed SAR filings.
- **The ERM context** — the enterprise-risk-management discipline these findings sit on (three lines of defence, risk appetite, the second line's independence) is **not re-derived here**; it lives in the [Enterprise Risk Management Guide](enterprise_risk_management_guide.md). The findings above are, in ERM terms, a textbook second-line failure: the AML/BSA program (a second-line control) was starved, static, and bypassed by the business for a decade ✅/⚠ (characterisation structural; the underlying facts are primary-source verified).

### 7.3 What the Enforcement Record Reveals About the Monitoring Estate

For the systems map, the 2024 record is a rare **primary-source window into a specific, named class of TD systems** — the U.S. retail bank's transaction monitoring and regulatory reporting estate ✅:

- **Transaction monitoring** — a monitoring platform existed but was configured to cover only ~8% of volume (the 92% gap), with static scenarios and no adaptation to new products (Zelle launched without coverage) ✅ (DOJ).
- **SAR/CTR reporting** — the SAR filing process failed at scale (thousands of suspicious transactions totalling ~US$1.5 billion never filed) and CTRs were delayed and sometimes misleading ✅ (FinCEN).
- **Internal unusual-transaction reports** — branches were instructed to stop filing internal reports on certain suspicious customers ✅ (DOJ).
- **The "flat cost paradigm"** ✅ — senior executives enforced a budget mandate that AML budget not increase year-over-year, postponing and cancelling AML projects — the business-side cause of the systems failures.
- **What is still not public** ⚠ — the *names* of the monitoring platform(s), the vendor(s) (the FircoSoft guide documents the screening-vendor class, not TD's contract), the alert-management tooling, and the lookback infrastructure (§12). The screening mechanics themselves are **not re-derived here** — see the [FircoSoft Guide](fircosoft_guide.md).

### 7.4 The Group IT Organization

TD Bank Group runs one of the **largest in-house IT organizations in Canadian banking** ✅/⚠ — the group employs over 102,000 people and operates retail, commercial, wealth, and wholesale businesses across Canada and the U.S., and its technology organization is correspondingly large; however, TD does not publish a technology-spend figure comparable to Citi's ~US$12B/yr disclosures ⚠, and the securities division's technology headcount is not split out (§12). What the public record supports:

- **Scale markers** ✅/⚠ — ~28 million customers, 1,060 Canadian branches, 1,158 U.S. locations, C$2.095 trillion assets (2025) — the franchise that the group core serves (Wikipedia/TD reporting). A technology estate serving that franchise is structurally certain to be mainframe-era at the core ⚠.
- **Mainframe and core** ⚠ — like every Big Five Canadian bank, TD's deposit, loan, and card cores are structurally inferred to run on mainframe-era systems with decades of accumulated customisation; **TD has never publicly named its core banking system** — the vendor classes it could belong to are documented in the [Core Banking Systems Guide](core_banking_systems_guide.md), but no TD-specific primary source exists (§12).
- **A public technology positive (2026)** ✅ — the J.D. Power Canada Digital Banking & Credit Card Mobile App Satisfaction Study ranked **TD first among Canadian banks** (score 690) — a verified, if partial, signal that the client-facing digital layer is competitive even while the core estate is opaque.

### 7.5 The 2024 AML-Driven Remediation Investment

The largest *verified* technology-investment theme in the group is the U.S. AML remediation ✅:

- **The monitorships** ✅ — a four-year independent monitorship under FinCEN, a three-year independent compliance monitor under the DOJ plea agreement, and the OCC's consent-order remediation programme (§8) — each requiring systems work: monitoring reconfiguration, SAR/CTR remediation, the SAR lookback, and the data-governance review.
- **The Fed's structural requirements** ✅ — a new U.S.-based remediation office and the **relocation of the U.S.-relevant AML compliance program components into the U.S.** — an organizational/technology relocation with direct systems consequences (which monitoring and reporting functions run where) ✅ (Fed release).
- **The investment magnitude** ⚠ — TD has publicly signalled large remediation spending since 2024 (press-reported figures around US$500M annually have circulated ⚠), but this pass did **not verify a specific published remediation-budget figure**; the *fact* of material remediation investment is certain (the consent orders require it), the *number* is not (§11–§12).
- **Cloud/modernization** ⚠ — TD's public cloud and modernization narrative is thin: no Google-Cloud-style strategic agreement announcement comparable to Citi's was found in this pass; the group's digital-modernization efforts are marketing-flavoured rather than architectural ⚠ (§12).

### 7.6 The Systems Consequence for TD Securities

The group-level themes matter to this guide because TD Securities sits inside them: the wholesale estate (TDFX, TD One Portal, TDS Automated Trading, TD Cowen, GTB) is *structurally separate* from the retail core, but shares the group's risk, finance, and compliance infrastructure ⚠ — and, since October 2024, shares the group's regulatory overhang: the U.S. enforcement actions apply to the *bank*, and TD Securities' U.S. operations (TD Securities (USA) LLC, TD Cowen) are part of the same U.S. regulatory perimeter ✅/⚠ (the perimeter is structural; the specific compliance-system linkages are not disclosed).

### 7.7 Cross-References

Core mechanics (posting, interest, ledgers, core vendor classes, strangler-fig modernization) are **not re-derived here** — see [Core Banking Systems Guide](core_banking_systems_guide.md). The treasury/ALM context is in [Treasury & ALM Guide](treasury_alm_guide.md) (§6). TD's specific core-identity question and modernization roadmap sit in §12 ⚠.

---

## 8. Risk and Regulatory Context: The October 2024 AML Settlement

### 8.1 The Settlement Package (Verified, Primary Sources)

On **October 10, 2024**, four U.S. authorities announced coordinated actions against TD Bank, N.A. and TD Bank USA, N.A. (collectively "TD Bank") and their parent — the defining regulatory event of TD's modern era, fully documented in primary sources ✅:

| Authority | Action (primary-source verified) | Reference |
|---|---|---|
| **FinCEN** | Record **US$1.3 billion** civil penalty against TD Bank, N.A. and TD Bank USA, N.A. for BSA violations — **the largest penalty against a depository institution in U.S. Treasury/FinCEN history**; a **four-year independent monitorship**; a "SAR lookback" of missed filings; and — for the first time — **accountability and data-governance reviews** | FinCEN release, Oct 10 2024 |
| **OCC** | **US$450 million** civil money penalty + **cease and desist order** + **growth restriction (asset cap)** on TD Bank, N.A. and TD Bank USA, N.A. for BSA/AML program deficiencies | OCC News Release 2024-116 |
| **Federal Reserve** | **US$123.5 million** fine against Toronto-Dominion Bank; requirements for a new U.S. remediation office, relocation of U.S.-relevant AML program components to the U.S., dividend/resource certification, and an independent board/management review. The Fed states: **"The penalties announced by the agencies total approximately $3.09 billion."** | Fed press release, Oct 10 2024 |
| **DOJ** | TD Bank N.A. pleaded guilty to **conspiracy to fail to maintain a BSA-compliant AML program, fail to file accurate CTRs, and launder money**; TD Bank US Holding Company pleaded guilty to causing those failures. Total financial penalty: **US$1,886,945,780.40** (forfeiture US$452,432,302.00 + criminal fine US$1,434,513,478.40); a **three-year independent compliance monitor**; US$123.5M of the forfeiture credited toward the FRB resolution; a 20% cooperation discount | DOJ Office of Public Affairs, Oct 10 2024 (press release 24-1286) |

**Reading the totals carefully** ✅/⚠ — each component figure above is primary-source verified. The agencies' own stated total is **"approximately US$3.09 billion"** (Fed release), and press coverage (e.g., CNN, October 10, 2024: "TD Bank hit with record $3 billion fine") reported the package at roughly US$3 billion. ⚠ **The arithmetic is not clean:** summing the four headline numbers (US$1.3B + US$450M + US$123.5M + US$1.887B) exceeds US$3.09B because the DOJ's US$123.5M forfeiture credit *is* the Fed penalty, and the Fed's "approximately" language leaves room. This guide states the components and the agencies' own total, and flags the reconciliation as ⚠ rather than forcing a single "correct" number (see §11). The investigative background is documented in U.S. and Canadian press — the WSJ's fentanyl-proceeds reporting and the Toronto Star's May 2024 examination of the U.S. franchise's AML failures ⚠ (secondary sources; consistent with, but not part of, the enforcement record).

### 8.2 The Regulators and the Coordinated Record

- **The four regulators** ✅ — FinCEN (Treasury), the OCC, the Federal Reserve Board, and the DOJ (with the U.S. Attorney's Office for the District of New Jersey; investigated with IRS Criminal Investigation, FDIC-OIG, and the DEA). OSFI, TD's Canadian home supervisor, can act in implementing the Fed's requirements ✅.
- **The precedent** ✅ — per Attorney General Merrick Garland: TD Bank became **"the largest bank in U.S. history to plead guilty to Bank Secrecy Act program failures, and the first US bank in history to plead guilty to conspiracy to commit money laundering."**
- **The asset cap** ✅ — the OCC imposed a **growth restriction (asset cap)** on TD Bank N.A. — the enforcement tool that, combined with the consent order, blocks the U.S. retail franchise from growing until BSA/AML remediation satisfies the OCC. This is the structural reason the First Horizon deal was already dead by May 2023 and why the group's U.S. strategy shifted to simplification and the Schwab exit (§1.3) ✅/⚠ (the causal chain is widely reported; the First Horizon termination itself was announced as "regulatory uncertainty").
- **The coordinated record** ✅ — the four agencies announced their actions on the same day, with the DOJ plea agreement, the FinCEN penalty, the OCC consent order, and the Fed's action cross-referencing one another: the DOJ's US$123.5M forfeiture credit is explicitly applied to the Fed's resolution, and the Fed's release states the agencies' combined total as "approximately $3.09 billion" — the single most heavily documented regulatory day in TD's history.

### 8.3 The AML Program Failures (Verified from the Enforcement Record)

The failures the enforcement record documents — each a systems finding as much as a compliance finding ✅:

- **Static, under-scoped transaction monitoring** ✅ (DOJ) — from 2014 through 2022 the transaction monitoring program was "effectively static": no new monitoring scenarios added, no material changes made; **92% of total transaction volume (≈ US$18.3 trillion, Jan 1 2018 – Apr 12 2024) went unmonitored**; ACH, most checks, and other transaction types were intentionally not automatically monitored; Zelle was launched without monitoring coverage.
- **Starved resourcing** ✅ (DOJ) — the "flat cost paradigm": a budget mandate that AML spend not increase year-over-year despite growing profits and risk, with AML projects postponed and cancelled.
- **SAR/CTR failures at scale** ✅ (FinCEN) — willful failure to file SARs on thousands of suspicious transactions totalling ~US$1.5 billion; delayed and sometimes misleading CTRs; a pattern of noncompliance with SAR filing requirements (OCC); instructions to stop filing internal unusual-transaction reports on certain customers (DOJ).
- **Known high-risk activity not acted on** ✅ (FinCEN) — funnel-account activity involving high-risk countries; the Da Ying Sze case (2017–2021: over US$400 million in largely cash transactions; Sze not identified in more than 500 CTRs totalling over US$400 million; pleaded guilty to money laundering in 2022); a 2021 employee who laundered narcotics proceeds for bribes, opening accounts for shell companies.
- **The criminal networks** ✅ (DOJ) — three networks moved more than **US$670 million** through TD Bank accounts between 2019 and 2023: over US$470 million in cash deposits into nominee accounts (operators gave employees gift cards worth over US$57,000), ~US$120 million through a high-risk jewelry business's shell accounts, and ~US$39 million via a Colombia ATM-withdrawal scheme involving five TD employees; two dozen-plus individuals charged.
- **"Trillions of dollars in transactions annually" unmonitored** ✅ (FinCEN) — the scale line quoted across the record.
- **P2P rails launched without coverage** ✅ (FinCEN) — Zelle (and the Venmo-style peer-to-peer flows the bank handled) operated without adequate transaction monitoring; the monitoring estate did not keep pace with new products.
- **Money moved after closure decisions** ✅ (DOJ) — more than **US$5 billion** in transaction activity flowed through TD Bank accounts between the bank's decision to close them and the actual closures — a timed, direct consequence of the monitoring gap.
- **Suspicious volume processed anyway** ✅ (OCC) — the OCC found **hundreds of millions of dollars were processed with suspicious indicia**, alongside a pattern of SAR noncompliance and CTR violations.
- **The remediation machinery** ✅ (FinCEN/DOJ) — the SAR lookback, the four-year FinCEN monitorship, the three-year DOJ compliance monitor, the OCC consent-order programme, and the first-time accountability and data-governance reviews.

### 8.4 The OCC's Deficiency Catalogue

The OCC's consent order and penalty announcement enumerate the BSA/AML program deficiencies it found — the fullest public catalogue of what was wrong with the U.S. monitoring estate ✅ (OCC News Release 2024-116):

- **Internal controls** ✅ — the bank's internal controls were found inadequate to ensure compliance with the BSA and its own AML program.
- **Risk management and risk assessments** ✅ — risk-management processes and the AML risk assessment were deficient, including the assessment of products, customers, and geographies (the funnel-account and high-risk-country exposure of §8.3).
- **Customer due diligence and risk ratings** ✅ — CDD was inadequate and customer risk ratings did not accurately reflect risk — the rating failure that let high-risk accounts persist for years.
- **SAR identification, evaluation, and reporting** ✅ — the identification, evaluation, and timely reporting of suspicious activity failed, producing the pattern of SAR noncompliance and the CTR violations.
- **Governance, staffing, and training** ✅ — board/management governance, staffing levels, and training of the AML function were all found deficient — the organizational mirror of the DOJ's "flat cost paradigm."
- **Independent testing** ✅ — the required independent testing of the AML program was inadequate, so the deficiencies compounded without detection.
- **The volume finding** ✅ — the OCC separately found that **hundreds of millions of dollars were processed through TD Bank with suspicious indicia**, and that the bank engaged in a **pattern of noncompliance with SAR filing requirements** and violated CTR obligations.

The catalogue matters to the systems map because each line item maps to a concrete systems remediation: re-rated customers (data quality), re-scoped monitoring scenarios (rules), re-engineered SAR/CTR pipelines (reporting), and restored independent testing (controls) — the programme §8.5 describes.

### 8.5 The Remediation Machinery as Systems Work

The settlement's remedial requirements are, in systems terms, a **multi-year re-engineering programme for the U.S. monitoring estate** ✅ (FinCEN/DOJ/OCC/Fed releases):

- **The SAR lookback** ✅ (FinCEN) — an independent consultant under the FinCEN monitor performs a historical analysis of TD's transaction data to remediate missed SAR filings: a data-archaeology project over years of transaction history, reconstructing what the monitoring estate should have caught.
- **The data-governance review** ✅ (FinCEN, first-time) — a root-cause review explicitly tasked with "identify[ing] and fix[ing] the root causes of many gaps in its AML program" — a data-quality programme in regulator's language.
- **The monitorships** ✅ — four years under the FinCEN independent monitor, three years under the DOJ compliance monitor, and the OCC's consent-order remediation programme, each with reporting obligations that keep the estate under continuous external inspection.
- **The Fed's structural requirements** ✅ — a new U.S.-based remediation office, the **relocation of the U.S.-relevant AML program components into the U.S.**, dividend/resource certification, and an independent board/management review: organizational changes with direct systems consequences (which monitoring and reporting functions run where, and how they are staffed).
- **The systems consequence** ✅/⚠ — the monitoring scenarios must be re-built and extended to ACH, checks, and P2P rails; alert tooling and case management re-provisioned; SAR/CTR pipelines re-engineered; and independent testing restored. The *requirements* are primary-source verified; the internal project names, vendors, and timelines are not (§12).
- **The group-level consequence** ✅ — the remediation is now part of the group's stated 2025–2026 U.S. strategy: the proceeds of the Charles Schwab stake sale were explicitly earmarked for U.S. simplification "following the US$3.09B money-laundering fine" (§1.3) — remediation resourcing has become a board-level capital-allocation matter, not a compliance-budget line.

### 8.6 Cross-References (Not Re-Derived)

The AML mechanics behind these findings — watch-list screening, name matching, the FircoSoft product family, sanctions regimes — are **not re-derived here**; they live in the [FircoSoft Guide](fircosoft_guide.md). The enterprise-risk-management frame (three lines of defence, the second line's independence, risk appetite, governance) lives in the [Enterprise Risk Management Guide](enterprise_risk_management_guide.md). The payment rails the monitoring gap ran on (ACH, checks, P2P rails like Zelle/Venmo) are documented in the [Payment Rails Guide](payment_rails_guide.md). The systems consequence for this guide's subject: **the October 2024 record is the most detailed public description of any TD system estate that exists** — and it describes the *retail bank's* monitoring estate, not TD Securities' trading systems, which remain ⚠ (§12).

---

## 9. The Singapore Angle

### 9.1 The Verified Singapore Presence

TD Securities' Singapore presence is real and publicly anchored ✅, though **thinly documented**:

- **The FI banking hub** ✅ — TD Securities' Global Transaction Banking Financial Institutions team is located in **Toronto, London and Singapore** (TD's own words, GTB page) — Singapore is one of the three cities of the FI/correspondent-banking network (§5.3).
- **Regional leadership** ✅ — **Cavin Wong, Regional Head of Global Transaction Banking, Asia-Pacific**, is contactable via a Singapore telephone number (+65) on TD Securities' own page — direct evidence of a Singapore-anchored APAC GTB leadership role.
- **The broader APAC footprint** ✅/⚠ — TD Securities operates across Asia-Pacific with offices including Singapore, Hong Kong, Tokyo, and Sydney (§1.4); the *specific* Singapore headcount and unit inventory are not published ⚠.
- **What the Singapore office does** ✅/⚠ — the verified public anchors are GTB/FI (liquidity, clearing, trade finance) and the broader APAC markets coverage; the task brief's lead that the Singapore office runs "rates/FX/credit sales, structured products, Asian credit" is **consistent with industry practice for a Canadian dealer's APAC hub but was not verified in any public source in this pass** ⚠ (§11–§12).
- **The FI hub's systems relevance** ✅/⚠ — Singapore is one of the three cities (with Toronto and London) from which TD's FI business serves bank clients with CAD/USD liquidity, clearing, and CLS-related services (§5.3): the hub implies local SWIFT/clearing access patterns, MAS-regulated outsourcing and technology-risk obligations, and connectivity back to Toronto — all structural ⚠ (TD publishes no Singapore unit inventory, technology or otherwise; see §9.3 and §12).

### 9.2 The Establishment Year (Unverified)

**The establishment year of the Singapore office / TD Securities (Singapore) Pte. Ltd. could not be verified in this pass** ⚠. The task brief's lead — late 1980s or early 1990s, following TD Securities' 1987 founding — is plausible but **not confirmed by any primary or reliable secondary source found**; no MAS directory entry, TD anniversary page, or press record pinning the Singapore opening year was located. The reader should treat the year as **unknown** (§11). What *is* verified is the *current* Singapore presence (§9.1). The same disclosure silence covers the Singapore office's systems: no published local platform inventory, outsourcing register, or technology-risk disclosures exist in the public record ⚠.

### 9.3 The MAS Context (Cross-Reference)

The MAS regulatory framework — licensing, notices, technology risk management (TRMG), outsourcing guidelines, and the enforcement record — is **not re-derived here**; it lives in the [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md). For TD specifically:

- **Licensing** ⚠ — TD Securities' Singapore operations (a capital-markets services holder of MAS licences for dealing in securities, per industry structure ⚠) sit inside that framework like every other MAS-regulated firm; the specific licence numbers were **not verified in this pass** (§11).
- **Outsourcing and technology-risk obligations** ⚠ — as a MAS-regulated operation, TD Securities' Singapore unit sits inside the MAS outsourcing guidelines and TRMG framework for any locally hosted or outsourced systems; the specifics (licence numbers, notified outsourcing arrangements, local data-residency choices) are not public (§12).
- **The Cymbal Bank connection** — the §10 worked example treats TD's Singapore office as a dealing/GTB counterparty to Cymbal Bank under MAS regulation; the MAS obligations each party carries (notice compliance, outsourcing rules, technology risk management) are the domain of the [MAS guide](mas_regulations_guidelines_guide.md), cross-referenced, not re-derived.

---

## 10. Worked Example: Cymbal Bank × TD Securities — Correspondent Banking and Markets

> **Persona note:** Cymbal Bank is the fictional/illustrative mid-size Singapore-based bank used across this repository's worked examples (it is the author's affiliation in the sibling guides). It stands in for a real mid-tier ASEAN bank. Everything in this section is an **illustrative reconstruction** built from the verified TD products in §3–§5 and the standard correspondent-banking mechanics in the [Payment Rails Guide](payment_rails_guide.md) — the *products* are real, the *scenario* is a model.

### 10.1 The Relationship

Cymbal Bank, a mid-size Singapore-headquartered bank, serves ASEAN corporates with Canadian and U.S. trade exposure. It holds a correspondent and markets relationship with TD Securities because Cymbal needs what a Canadian-anchored dealer provides:

- **CAD liquidity and clearing** — Cymbal's clients invoice in Canadian dollars for commodities and machinery; Cymbal needs a CAD clearing bank with scale in Toronto — TD's GTB FI catalogue (CAD/USD liquidity, CAD payment clearing, §5.3) is exactly this product.
- **USD support** — TD Securities (USA) LLC and TD Bank N.A.'s U.S. network provide the dollar leg, including the daylight-credit and liquidity tools of the FI catalogue ✅/⚠ (product catalogue verified; Cymbal's specific terms are the model).
- **FX execution** — Cymbal's treasury hedges CAD/USD and USD/SGD exposure; TD's FX franchise (TDFX, §3.1) is the execution surface.
- **Trade finance** — Cymbal's clients need LCs confirmed or advised by a name their counterparties trust; TDFacilitrade (§5.4) is the product family.
- **Research** — Cymbal's investment desk consumes TD Cowen's equity/credit research (§4.2) through the TD One Portal surface (§3.2) ⚠ (distribution channel structural).

The relationship is a **financial-institution (FI) client** relationship: Cymbal is a *client of TD's GTB and Global Markets businesses*, not a TD subsidiary — the mirror of the correspondent model TD itself runs through its FI network (§5.3).

### 10.2 The Rails: Correspondent Banking, Nostro Accounts, SWIFT

The mechanics here are standard correspondent banking — documented in full in the [Payment Rails Guide](payment_rails_guide.md) and only mapped to TD here:

- **Nostro** — Cymbal holds a **CAD nostro account at TD (Toronto)** and, for the dollar leg, a **USD nostro at TD Bank, N.A.** From Cymbal's perspective these are "our money at their bank"; from TD's they are vostro balances. The glossary defines both.
- **Access** ⚠ — Cymbal's treasury sees balances and initiates payments through TD's corporate/FI channels; **TD publishes no FI-portal name** (§5.3), so this step is structural: the class of system (an FI e-banking channel with maker/checker controls, statements, and intraday reporting) is standard, the TD instance is not public.
- **Messaging** — payment instructions flow over **SWIFT**: MT103 (customer transfer) / MT202 (bank transfer), and pacs.008/pacs.009 under CBPR+ in the ISO 20022 era; Cymbal's core and TD's payments estate exchange these via their respective SWIFT interfaces ⚠ (TD's SWIFT gateway specifics are not public, §12; the message standards are in the [Payment Rails Guide](payment_rails_guide.md)).
- **GPI tracking** — cross-border payments carry the SWIFT GPI tracker reference so Cymbal's ops can trace end-to-end status ⚠ (GPI mechanics: [Payment Rails Guide](payment_rails_guide.md); TD's GPI participation is structural, not verified in this pass).
- **The daily cycle** — Cymbal's back office reconciles the TD nostro statements against its own ledgers; the CAD leg settles through Canada's Lynx RTGS and the legacy ACSS/automated clearing ⚠ (rail mechanics: [Payment Rails Guide](payment_rails_guide.md)).

### 10.3 KYC and Onboarding

Onboarding a correspondent is a KYC programme in its own right, and for a U.S.-connected relationship it is a **regulated** one (the USA PATRIOT Act's correspondent-account due-diligence rules apply to U.S. banks' foreign FI accounts ⚠ structural):

- **The onboarding file** — Cymbal provides: ownership structure, licences (MAS capital-markets/full-bank licensing, §9.3), AML/CFT programme documentation, sanctions policy, and the classic **correspondent banking questionnaire**; TD's FI due-diligence team assesses jurisdiction, ownership, and respondent-bank risk ⚠ (TD's questionnaire is confidential, §12; the class of information is standard across the industry).
- **Screening** — Cymbal's name, principals, and transactions are screened against sanctions lists (OFAC and others) on an ongoing basis; the screening-engine mechanics are in this repo's [FircoSoft Guide](fircosoft_guide.md) and [Enterprise Risk Management Guide](enterprise_risk_management_guide.md) ⚠ (TD's screening vendor is not public, §12 — and, given §8, the *quality* of such screening is exactly what the 2024 record put in question for the U.S. retail bank; the wholesale/FI perimeter is a separate estate ⚠).
- **Ongoing monitoring** — transaction monitoring on the nostro flows, periodic KYC refresh (the "evergreen" review), and the respondent-bank obligations Cymbal carries for its own downstream correspondents ⚠ (structural).

### 10.4 Trade Services

The trade angle exercises TD's GTB trade estate (§5.4):

- **The scenario** — Cymbal's client, a Singapore commodities exporter, sells to a Canadian importer. The importer's bank issues an **LC** in favour of the exporter; **Cymbal advises the LC** and, where the exporter wants the issuing bank's risk replaced by a stronger name, arranges for the LC to be **confirmed** — with TD Securities as confirming bank ⚠ (confirmation economics: the confirming bank takes the issuing bank's risk for a fee; mechanics in the [Trade Finance Guide](trade_finance_guide.md)).
- **Cymbal's own issuance** — for Cymbal's importing clients, Cymbal issues LCs and uses **TD's trade processing** via the TDFacilitrade product family ⚠ (product structure per §5.4; engine names not public).
- **Documentary flow** — documents against payment/acceptance, discrepant-document handling, and settlement through the Cymbal–TD nostro: the message types (MT700/MT707 for LCs, MT400/MT202 for settlement) are the domain of the [Trade Finance Guide](trade_finance_guide.md) and [Trade Finance Systems Guide](trade_finance_systems_guide.md), cross-referenced, not re-derived.
- **Export & agency finance** — for Cymbal's infrastructure clients, TD's ECA-backed term lending (§5.5) is the product class ⚠ (structural).

### 10.5 The Systems Involved (Mapped to This Guide)

| Step in the relationship | TD system (verified where marked) | Guide section |
|---|---|---|
| Cymbal sees CAD/USD nostro balances, initiates payments | FI/corporate channel ⚠ (no public name) | §5.3, §10.2 |
| CAD clearing and liquidity | GTB FI catalogue (CAD/USD liquidity, CAD payment clearing, CLS service) ✅ | §5.3 |
| FX hedging of CAD/USD exposure | TDFX ✅/⚠ (portal verified; feature docs not public) | §3.1 |
| LC confirmation/advising | TDFacilitrade ✅/⚠ (brand verified; engine not public) | §5.4, [Trade Finance Guide](trade_finance_guide.md) |
| Research consumption | TD Cowen research via TD One Portal ✅/⚠ | §3.2, §4.2 |
| Sanctions screening of flows | Screening estate ⚠ (vendor not public) | [FircoSoft Guide](fircosoft_guide.md), [Enterprise Risk Management Guide](enterprise_risk_management_guide.md) |
| USD settlement | TD Bank, N.A. nostro + SWIFT/ISO 20022 | §10.2, [Payment Rails Guide](payment_rails_guide.md) |

The architect's takeaway: **Cymbal's entire TD relationship runs through three verified product surfaces — the GTB FI catalogue (CAD/USD liquidity and clearing), TDFX (FX execution), and TDFacilitrade (trade) — over the correspondent rails documented in the Payment Rails Guide.** Everything beneath those surfaces is the ⚠ engine layer of §12. That is the honest shape of the TD systems map for any FI client.

### 10.6 Step-by-Step: A CAD Payment from Singapore to Toronto

Walk a single payment end-to-end — Cymbal's client in Singapore owes C$500,000 to a Canadian supplier whose account is at a third Canadian bank. Each step is the standard correspondent pattern from the [Payment Rails Guide](payment_rails_guide.md), mapped to the verified TD surfaces ⚠ (structural; TD's engine internals are §12):

1. **Instruction** — Cymbal's client instructs Cymbal to pay C$500,000 to the supplier's account at the Canadian bank; Cymbal's treasury operator creates the payment in TD's corporate/FI channel, specifying the beneficiary bank and account ⚠ (Cymbal's own core feeds the channel; the file formats are the standard corporate-banking ones).
2. **Authorization** — the maker/checker pair releases the payment under entitlement controls ⚠ (the four-eyes pattern of §5.2).
3. **Message** — Cymbal's SWIFT interface sends the customer transfer instruction — MT103 in the classic era, pacs.008 under CBPR+ — addressed to TD, Toronto (Cymbal's CAD correspondent) ⚠ (message standards: [Payment Rails Guide](payment_rails_guide.md)).
4. **Screening** — TD's sanctions/AML screening runs on the payment at intake and again against updates; a false positive stops the flow for investigation (§10.3) ⚠.
5. **Nostro debit** — TD debits Cymbal's CAD nostro (§10.2); the cover leg to the beneficiary's bank runs as MT202 / pacs.009 bank-to-bank transfer ⚠.
6. **The Toronto leg** — the C$500,000 moves across Canada's wholesale rails — **Lynx** (the Bank of Canada's RTGS) — to the beneficiary's bank, which credits the supplier's account ⚠ (rail mechanics: [Payment Rails Guide](payment_rails_guide.md); TD's clearing memberships are not published, §12).
7. **Tracking** — the payment carries a GPI tracker reference; Cymbal's ops see end-to-end status in the channel and can prove value to the client ⚠.
8. **Reconciliation** — at end of day, the nostro statement shows the debit, fees, and the resulting balance; Cymbal's back office matches it against its own ledger (§10.7).

The whole flow touches exactly zero *named* TD transaction engines — the FI channel, screening estate, and clearing memberships all sit in the ⚠ layer between the verified product catalogue (§5.3) and the rails.

### 10.7 Step-by-Step: The Nostro Reconciliation Cycle

The daily discipline that keeps the relationship honest ⚠ (structural — the class of process is universal; TD's statement formats and tools are not public):

1. **Intraday** — the channel shows the live nostro position; Cymbal's liquidity desk manages against payment windows and cut-offs (§10.6 step 6) ⚠.
2. **End of day** — TD posts the nostro statement (debits, credits, fees, value dates); Cymbal receives it through the channel or SWIFT ⚠.
3. **Matching** — Cymbal's reconciliation matches each statement line to an instructed payment: amount, value date, beneficiary, and charges convention (OUR/SHA/BEN) ⚠ (charges conventions: [Payment Rails Guide](payment_rails_guide.md)).
4. **Breaks** — unmatched items become breaks: missing credits, value-date differences (interest impact), and fee mismatches; each is investigated, typically by SWIFT investigation messages ⚠.
5. **Aging and claims** — unresolved breaks age; Cymbal raises claims against TD (and TD against Cymbal) under the correspondent agreement; aged breaks feed the monthly nostro-position review ⚠.

The reconciliation cycle is the *control* that makes the §10.2 nostro relationship safe to run at volume — and it is exactly the class of process that a deficient monitoring/data estate (the §8 findings, in the U.S. retail perimeter) makes unreliable at scale ⚠.

### 10.8 Step-by-Step: KYC Due Diligence on Cymbal

Before Cymbal gets the channel credentials of §10.6, it passes through the correspondent-banking KYC programme (§10.3) ⚠ (structural — the standard industry sequence, per the [FircoSoft Guide](fircosoft_guide.md) and [Enterprise Risk Management Guide](enterprise_risk_management_guide.md)):

1. **The file** — Cymbal supplies charter documents, its MAS licences (§9.3), ownership structure, and management/UBO information.
2. **The questionnaire** — Cymbal completes a correspondent banking questionnaire covering its AML/CFT programme, sanctions policy, correspondent network, and downstream respondent banks ⚠ (TD's questionnaire is confidential; the class is industry-standard).
3. **Risk rating** — TD's FI due-diligence team assesses jurisdiction, ownership, respondent-bank, and product risk; the result sets the EDD level and monitoring intensity ⚠.
4. **Screening** — Cymbal's name, principals, and UBOs screen against OFAC and other sanctions lists; hits trigger enhanced review ⚠.
5. **Approval and onboarding** — the relationship is approved at the delegated authority level, the nostro agreement is signed, and the channel entitlements are provisioned ⚠.
6. **Evergreen** — periodic KYC refresh re-runs steps 1–4 on a cycle; material changes (ownership, jurisdiction) trigger off-cycle review; ongoing transaction monitoring runs on the nostro flows (§10.6 step 4) ⚠.

The KYC programme is where the §8 findings cast their longest shadow for a U.S.-connected bank: correspondent due-diligence records must be complete, accurate, and retrievable — precisely the properties the 2024 record found deficient in the U.S. retail perimeter ✅/⚠ (the findings are verified; their remediation status is not public).

### 10.9 Step-by-Step: A Documentary Credit with TD Confirmation

The trade-services flow (§10.4) in full ⚠ (structural — message families and mechanics in the [Trade Finance Guide](trade_finance_guide.md); TD's engine internals not public):

1. **The contract** — Cymbal's exporter client signs a sale contract with the Canadian importer: payment by irrevocable documentary credit.
2. **Issue** — the importer's bank issues the LC in favour of the exporter and sends the MT700 through SWIFT, with Cymbal as advising bank.
3. **Advise** — Cymbal authenticates the MT700 and advises the LC to the exporter via its trade channels (the TDFacilitrade-class workflows for FI clients, §5.4) ⚠.
4. **Confirmation** — the exporter wants the issuing bank's risk replaced by a stronger name; Cymbal requests TD to add its confirmation. TD confirms — taking the issuing bank's risk for a confirmation fee — and the exporter ships against the confirmed LC ⚠ (confirmation economics: [Trade Finance Guide](trade_finance_guide.md)).
5. **Presentation** — the exporter presents documents (invoice, bill of lading, packing list, certificate) to Cymbal, which checks them against the LC terms (the UCP 600 document check) ⚠.
6. **Discrepancies** — a late shipment makes the documents discrepant; Cymbal either waives, amends (MT707), or presents on approval; clean documents proceed ⚠.
7. **Settlement** — on complying presentation, the importer's bank pays; the funds flow through the correspondent chain — MT400/MT202-style settlement through the Cymbal–TD nostro (§10.2) — and the exporter receives payment ⚠.

For the systems map, the LC flow is the trade mirror of §10.6: verified product surfaces at the ends (TDFacilitrade, the nostro), message standards in the middle, and the confirmation/risk engine unnamed in the ⚠ layer (§12).

### 10.10 The Operating-Day Narrative

Put the steps together as a day at Cymbal's back office ⚠ (structural): the morning starts with the CAD and USD nostro positions and the day's payment queue in the FI channel; the CAD payment window closes against Toronto cut-offs and the Lynx session; the trade team tracks the confirmed LC and the exporter's presentation; the FX desk hedges the day's USD/CAD exposure on TDFX; by end of day the nostro statements are matched, the breaks are logged, and the quarter's KYC refresh sits in the queue. Every one of those screens is a verified TD product surface or a standard industry process; every engine behind them is the §12 unknown. That is the honest operating picture of correspondent banking on the TD estate — and, by extension, on any Canadian-anchored dealer's ⚠.

---

## 11. Claims Status and Verification Notes

**Verification convention: ✅ = verified in this research pass (primary or secondary sources); ⚠ = flagged (inferred, approximate, single-source, or structural); ❌ = disputed (the public record contradicts the claim); "not public" = TD does not disclose and no external source exists either way.**

### 11.1 The Claims-Status Table

| Claim | Source | Status |
|---|---|---|
| Bank of Toronto founded 1855; Dominion Bank founded 1869 | TD Bank Group history (via Wikipedia, primary-cited); Wikipedia | ✅ verified |
| 1954 merger agreement; Finance Minister approval Nov 1 1954; Toronto-Dominion Bank created Feb 1 1955; 4th-largest Canadian bank, C$1.1B assets | TD history; Wikipedia | ✅ verified |
| 1976: TD 360 ATM piloted, renamed "Green Machine" | TD Wikipedia (primary-cited history) | ✅ verified |
| 1987: Toronto Dominion Securities Inc. established | TD history ("In 1987, Toronto Dominion Securities Inc. was established by the bank"); tdsecurities.com ("Since opening our securities arm in 1987") | ✅ verified (two independent sources) |
| 1987 "Gardiner Watson" acquisition lore | No primary source found in this pass | ⚠ unverified (see §12) |
| 2000: Newcrest Capital acquired for C$224M (75% stock / 25% cash); Canada Trust acquired | Wikipedia (TD history); press | ✅ verified |
| 2002: Stafford Trading and Letco Trading acquired | Wikipedia (TD history) | ✅ verified |
| 2004–2008: Banknorth → TD Banknorth → Commerce Bancorp → TD Bank, N.A. (May 31 2008) | Wikipedia; TD | ✅ verified |
| 2019: G-SIB designation by the Financial Stability Board | FSB designation; Wikipedia | ✅ verified |
| March/July 2021: Headlands Tech Global Markets acquired, renamed TDS Automated Trading | tdsecurities.com automated-trading page (primary) | ✅ verified |
| Aug 2022: Cowen acquired for US$1.3B (US$39/share); March 2023 close → TD Cowen, division of TD Securities | Wikipedia; TD announcements | ✅ verified |
| Feb 2022: First Horizon US$13.4B offer; May 2023 termination ("regulatory uncertainty") | TD announcements; press | ✅ verified |
| Oct 10 2024: FinCEN record US$1.3B penalty vs TD Bank N.A. / TD Bank USA N.A.; four-year monitorship; SAR lookback; first-time accountability and data-governance reviews | FinCEN release (Oct 10 2024) | ✅ verified |
| Oct 10 2024: OCC US$450M civil money penalty + cease-and-desist + asset cap | OCC News Release 2024-116 | ✅ verified |
| Oct 10 2024: Fed US$123.5M fine vs Toronto-Dominion Bank; U.S. remediation office; AML component relocation; dividend certification | Federal Reserve press release (Oct 10 2024) | ✅ verified |
| Oct 10 2024: DOJ guilty pleas; US$1,886,945,780.40 total (US$452,432,302.00 forfeiture + US$1,434,513,478.40 fine); three-year monitor; US$123.5M forfeiture credit to the Fed; 20% cooperation discount | DOJ Office of Public Affairs (press release 24-1286) | ✅ verified |
| "Approximately US$3.09 billion" combined total | Fed release ("approximately $3.09 billion"); press ("record $3 billion fine") | ✅ verified (agencies' own total); ⚠ arithmetic (components overlap via the forfeiture credit; see §11.2) |
| 92% of U.S. transaction volume unmonitored Jan 1 2018 – Apr 12 2024 (~US$18.3T); no new monitoring scenarios 2014–late 2022; Zelle launched without coverage; >US$5B activity after closure decisions | DOJ plea documents/release | ✅ verified |
| Da Ying Sze case (2017–2021, >US$400M, 500+ CTRs); 2021 bribed-employee case; three criminal networks >US$670M 2019–2023; two dozen-plus charged | FinCEN release; DOJ | ✅ verified |
| Product names and positioning: TDFX, TD One Portal, TDFacilitrade, TDS Automated Trading, structured-notes portal | tdsecurities.com product pages (primary) | ✅ verified (names/positioning); ⚠ internals |
| "TD ThinkPipes" fixed-income platform name | Not found at any primary source; absent from current tdsecurities.com and Wayback captures | ⚠ unverified (see §12) |
| Singapore office establishment year | No primary or reliable secondary source found | ⚠ unverified (see §12) |
| Market-data vendor contracts (Bloomberg/LSEG-class) | Not named in TD public material | ⚠ not public (see §12) |
| AML remediation spend (~US$500M/yr, press-reported) | Press reports only; no primary figure verified | ⚠ reported/unverified (see §12) |
| 6,500+ professionals / 40 cities | Wikipedia infobox | ✅/⚠ (secondary; TD's own "16 offices / 4,900+ FTE, Oct 2021" differs by scope and date) |
| J.D. Power 2026: #1 Canadian bank mobile-app satisfaction (score 690) | J.D. Power study, as reported | ✅ verified (secondary) |
| TD Cowen integration depth with the Canadian estate | Not disclosed | ⚠ not public (see §12) |

### 11.2 Verification Notes and Methodology

1. **The verified anchors are strong where the record exists** — the history (§1.3) is verified via TD's own history page and tdsecurities.com; the acquisition trail (Newcrest 2000, Headlands 2021, Cowen 2023) via TD pages and Wikipedia; and the October 2024 enforcement record via four primary sources (FinCEN, OCC, Fed, DOJ) — the deepest regulatory documentation of any TD system estate that exists.
2. **The inference zone is the engine layer** — pricing, OMS/EMS, core, hubs, data platform, risk systems: the *class* of system is certain for a bank of TD's size, the *identity* is not (§12).
3. **One deliberate ⚠ on the headline number** — the "~US$3.09 billion" total is the agencies' own framing ("approximately"), not a clean sum of the components: the DOJ's US$123.5M forfeiture credit *is* the Fed penalty, so summing the headline figures double-counts it. This guide states the components and the total separately and does not force a single reconciliation.
4. **Single-source items** — several figures rest on one secondary source (Wikipedia-reported C$8.099B 2023 revenue, the 6,500/40-city infobox, the C$2.095T group assets) and are marked ✅/⚠ accordingly.
5. **Deliberately not verified** (out of scope): vendor contracts and pricing, internal roadmaps, per-unit headcount and technology-spend splits, MAS licence numbers, and the exact current office inventory.

---

## 12. What Could Not Be Verified

The following are the materially non-public specifics of the TD estate. Each is flagged ⚠ and should be treated as unknown, not as "likely X":

- **"TD ThinkPipes"** ⚠ — the reported name of a public electronic fixed-income platform could not be verified at any primary source: it does not appear on tdsecurities.com's current pages (the fixed-income offering is named via "Automated Trading", "Credit Products", "Interest Rate Products", "Money Markets", "Municipal Products", and "Securitized Products"), and no archived tdsecurities.com URL containing "pipes" was found in the Wayback Machine (§2.2). Readers should not cite it as a confirmed TD product name.
- **The 1987 "Gardiner Watson" acquisition lore** ⚠ — the task brief suggested a Gardiner Watson acquisition in TD Securities' founding year; no primary source confirms it, and TD's own history and website date the securities arm's 1987 founding without reference to it (§1.3).
- **The Singapore establishment year** ⚠ — TD Securities (Singapore) Pte. Ltd.'s opening year is unverified; no MAS directory entry, TD anniversary page, or press record pins it down (§9.2).
- **Internal trading platforms and vendors** ⚠ — the OMS/EMS, pricing engines, risk systems, algorithmic stack, and the connectivity catalogue behind TDFX, TDS Automated Trading, TD Cowen's execution platforms, and the trading floors; TD publishes no FIX/API catalogue (§2–§4).
- **TDS Automated Trading engine internals** ⚠ — the ex-Headlands platform's architecture, latency figures, venue connectivity, and staffing; the mandate is verified, the instance is not (§4.1).
- **TD Cowen integration depth** ⚠ — how deeply TD Cowen's systems are integrated with TD Securities' Canadian estate (single OMS/EMS? shared risk platform? consolidated clearing?) is undisclosed (§4.2).
- **The group core-banking system name** ⚠ — TD has never publicly named its core banking system; the vendor classes are documented in the [Core Banking Systems Guide](core_banking_systems_guide.md), but no TD-specific primary source exists (§7.4).
- **AML transaction-monitoring platform names and vendors** ⚠ — the names of the monitoring platform(s), the alert-management tooling, the lookback infrastructure, and the screening vendor behind the U.S. estate; the [FircoSoft Guide](fircosoft_guide.md) documents the industry class, not TD's contract (§7.3, §8.5).
- **SWIFT gateway infrastructure** ⚠ — TD's SWIFT connectivity, BIC infrastructure, and ISO 20022 implementation specifics behind the FI clearing services (§5.3, §10.2).
- **The FI client portal name** ⚠ — TD publishes no portal name for its corporate cash-management or FI channels — there is no TD equivalent of CitiDirect BE (§5.2–§5.3).
- **The host-to-host/API catalogue** ⚠ — no public file-format, API, or host-to-host integration catalogue exists for TD's GTB products (§5.2).
- **Market-data vendor contracts** ⚠ — the vendors feeding TDFX, TD One Portal, and the trading floors (Bloomberg/LSEG-class) are not named in TD's public material (§7.1).
- **Technology-spend and headcount splits** ⚠ — TD does not publish a securities-division technology-spend figure or a technology-headcount split; the group's IT organization is large but not itemized (§7.4).
- **The remediation-spend figure** ⚠ — press-reported figures around US$500M annually have circulated since 2024; no primary published remediation-budget figure was verified in this pass (§7.5).
- **Cloud/modernization roadmap** ⚠ — TD's public cloud and modernization narrative is thin; no strategic-agreement announcement comparable to peers' was found in this pass (§7.5).
- **MAS licence numbers and Singapore technology filings** ⚠ — the specific licence numbers, notified outsourcing arrangements, and TRMG-related disclosures of the Singapore unit are not public (§9.3).
- **CLS membership detail and GPI participation** ⚠ — the CLS service is named on TD's GTB page, but TD's membership status and SWIFT GPI participation specifics were not re-verified in this pass (§5.3, §10.2).
- **Treasury/ALM system instances** ⚠ — TD does not name its treasury workstations, ALM models, FTP engines, or liquidity-reporting platforms (§6.1).

**The disclosure reality:** TD publishes **no internal system map**. The product layer is named and marketed; the engine layer is structurally inferred from industry practice. The single exception is the U.S. transaction-monitoring and reporting estate, which the October 2024 enforcement record forced into the public record in exceptional detail (§8) — and even there, the *platform names and vendors* remain unnamed (above). For the architect this means: describe the surfaces with confidence, model the engines as classes, and treat any specific TD system name not listed in §2.2 as unverified until a primary source appears.

---

## 13. Glossary

Terms used in this guide, in the series' standard glossary style. Section references point to where each term is used in the TD context.

| Term | Definition |
|---|---|
| **TDFX** | TD Securities' client-facing electronic FX portal (deal.tdsecurities.com/tdfx) — spot, forwards, NDFs, options, swaps (§3.1) |
| **TD One Portal** | TD Securities' client portal (portal.tdsecurities.com) for securitization reports, documentation, and research/content distribution (§3.2) |
| **TDFacilitrade** | TD's trade-finance product brand covering import/export/standby letters of credit and documentary collections (§5.4) |
| **TDS Automated Trading** | TD's electronic market-making business formed from Headlands Tech Global Markets (2021): fully automated market-making in municipal and investment-grade corporate bonds (§4.1) |
| **Headlands Tech Global Markets** | The quantitative fixed-income trading firm (founded 2013) TD acquired in 2021 and renamed TDS Automated Trading (§3.3) |
| **TD Cowen** | The division of TD Securities created by the 2023 Cowen acquisition: U.S. investment banking, equity/credit research, sales & trading, prime brokerage, outsourced trading, global clearing (§4.2) |
| **GTB** | Global Transaction Banking — TD Securities' transaction-banking line: corporate cash management, FI banking, trade & working capital, export & agency finance (§5.1) |
| **FI banking** | Financial Institutions banking — the correspondent-banking practice (Toronto/London/Singapore) serving bank clients with CAD/USD liquidity and clearing (§5.3) |
| **ABCP** | Asset-Backed Commercial Paper — TD Securities is publicly positioned as one of the largest North American ABCP conduit sponsors (§4.3) |
| **Nostro / Vostro** | "Our money at your bank" (nostro, from Cymbal's view of its TD account) / "your money at our bank" (vostro, from TD's view) (§10.2) |
| **Correspondent banking** | The relationship in which one bank (Cymbal) uses another bank's (TD's) balance sheet and network to clear, settle, and pay in markets where it has no presence (§10.1) |
| **SWIFT** | The cooperative messaging network (MT/ISO 20022) that carries cross-border payment and trade messages between banks |
| **ISO 20022** | The global payments message standard (CBPR+ cross-border migration); pacs.008/pacs.009 for payments |
| **CBPR+** | The SWIFT-managed cross-border payments ISO 20022 migration programme |
| **MT103 / MT202** | SWIFT payment messages: MT103 customer credit transfer; MT202 bank-to-bank transfer (the correspondent cover leg) |
| **pacs.008 / pacs.009** | The ISO 20022 payment messages: customer credit transfer / bank-to-bank transfer under CBPR+ |
| **GPI** | SWIFT Global Payments Innovation — tracked, same-day-usable cross-border payments (§10.2) |
| **Lynx** | The Bank of Canada's RTGS system — the wholesale CAD settlement rail (§10.6) |
| **RTGS** | Real-Time Gross Settlement — the central-bank settlement rail class (Lynx, Fedwire, MEPS+) |
| **FIX** | The Financial Information eXchange protocol for electronic trading (see [FIX Protocol Guide](fix_protocol_guide.md)) |
| **SAR** | Suspicious Activity Report — the U.S. filing that flags suspicious transactions; TD's SAR process failed at scale (thousands of filings missed, ~US$1.5B) (§8.3) |
| **CTR** | Currency Transaction Report — the U.S. filing for large cash transactions; TD's CTRs were delayed and sometimes misleading (§8.3) |
| **CDD** | Customer Due Diligence — the KYC discipline the OCC found inadequate, including customer risk ratings (§8.4) |
| **UBO** | Ultimate Beneficial Owner — the natural person(s) behind a legal entity, identified in KYC/onboarding (§10.8) |
| **FinCEN** | The U.S. Treasury's Financial Crimes Enforcement Network — imposed the record US$1.3B penalty (§8.1) |
| **OCC** | The U.S. Office of the Comptroller of the Currency — imposed the US$450M civil money penalty and asset cap (§8.1) |
| **FRB** | The Federal Reserve Board — fined Toronto-Dominion Bank US$123.5M and required the U.S. remediation office (§8.1) |
| **DOJ** | The U.S. Department of Justice — secured the guilty pleas and the US$1,886,945,780.40 financial penalty (§8.1) |
| **CMP** | Civil Money Penalty — the OCC's enforcement instrument (US$450M against TD Bank, N.A.) |
| **Consent order / C&D** | Regulatory enforcement instruments: a consent order is agreed between regulator and bank; a cease and desist (C&D) order commands corrective action |
| **Asset cap** | The OCC growth restriction on TD Bank, N.A. — blocks U.S. retail growth until BSA/AML remediation satisfies the OCC (§8.2) |
| **Monitorship** | Independent third-party oversight: four years under FinCEN's monitor, three under the DOJ compliance monitor (§8.5) |
| **G-SIB** | Global Systemically Important Bank — TD's FSB designation since 2019 (§1.3) |
| **NDF** | Non-Deliverable Forward — an FX derivative settled in a reference currency; part of the TDFX product suite (§3.1) |
| **Delta one** | The linear-derivative class (futures, swaps, ETFs) in TD's equities offering (§1.5) |
| **RFQ** | Request for Quote — the electronic trading workflow in which a client requests a price; the standard flow for automated bond market-making (§4.1) |
| **Market-making** | Quoting continuous buy/sell prices to provide liquidity — TDS Automated Trading's fully automated mandate (§4.1) |
| **Sanctions screening** | The watch-list filtering of names and transactions against OFAC and other sanctions lists (see [FircoSoft Guide](fircosoft_guide.md)) |
| **Three lines of defence** | The risk-governance model (business / risk & compliance / internal audit) — the frame in which the AML failures are a second-line failure (§7.2) |
| **Green Machine** | The 1976 TD 360 ATM brand that became TD's ATM name — this guide's bookend (§1.3) |

---

## 14. References and Further Reading

**Primary regulatory sources (October 2024 AML settlement):**

1. FinCEN — "FinCEN Assesses Record $1.3 Billion Penalty Against TD Bank" (October 10, 2024) — https://www.fincen.gov/news/news-releases/fincen-assesses-record-13-billion-penalty-against-td-bank
2. OCC — News Release 2024-116, "OCC Assesses $450 Million Civil Money Penalty Against TD Bank, N.A." (October 10, 2024) — https://www.occ.gov/news-issuances/news-releases/2024/nr-occ-2024-116.html
3. Federal Reserve — enforcement action press release, TD Bank / Toronto-Dominion Bank (October 10, 2024) — https://www.federalreserve.gov/newsevents/pressreleases/enforcement20241010a.htm
4. DOJ Office of Public Affairs — "TD Bank Pleads Guilty to Bank Secrecy Act and Money Laundering Conspiracy Violations" (October 10, 2024; press release 24-1286) — https://www.justice.gov/opa/pr/td-bank-pleads-guilty-bank-secrecy-act-and-money-laundering-conspiracy-violations-18b

**TD Securities and TD Bank Group sources:**

5. TD Securities — corporate site and product pages (Global Markets, FX/TDFX, Fixed Income, Equities, Prime Brokerage, Securitization, GTB/FI, Trade, Automated Trading) — https://www.tdsecurities.com ⚠ (deep page paths as consulted in this pass were not archived; the product names are verified against the live pages)
6. TDFX portal — https://deal.tdsecurities.com/tdfx ✅ (URL published on TD's FX page); TD One Portal — https://portal.tdsecurities.com ✅; structured-notes surface — https://portal.tdsecurities.com/notes ✅
7. TD Bank Group — corporate history (1855 / 1869 / 1955 / 1976 / 1987 / 2000 timeline) — https://www.td.com (history pages; also cited via Wikipedia)
8. Wikipedia — "Toronto-Dominion Bank", "TD Bank, N.A.", "TD Securities", "TD Cowen" (as consulted; primary citations therein)

**Press and awards:**

9. CNN — "TD Bank hit with record $3 billion fine" (October 10, 2024) ⚠ (headline as cited in §8.1; URL not re-verified in this pass)
10. The Wall Street Journal — fentanyl-proceeds reporting on TD's AML failures (2024) ⚠ (secondary investigative background)
11. Toronto Star — May 2024 examination of TD's U.S. AML failures ⚠ (secondary)
12. J.D. Power — 2026 Canada Digital Banking & Credit Card Mobile App Satisfaction Study (TD ranked #1 among Canadian banks, score 690) ⚠ (as reported)
13. TD-published awards: Coalition Greenwich #1 Overall Canadian Fixed-Income Service Quality (2018–2021); Refinitiv #1 Announced M&A Transactions (2017–2021); Trade Finance Global "Excellence in Trade (North America)" (2023) ⚠ (TD-published claims)

**Method note:** the Wayback Machine CDX API (https://web.archive.org) was used for tdsecurities.com URL archaeology (portal.tdsecurities.com paths); general web-search tooling was unreliable in this pass, so verification leaned on the primary sources above and the research notes compiled for this entry. Sibling guides in this repository (cross-referenced, not re-derived): [Payment Rails Guide](payment_rails_guide.md), [FIX Protocol Guide](fix_protocol_guide.md), [FircoSoft Guide](fircosoft_guide.md), [Enterprise Risk Management Guide](enterprise_risk_management_guide.md), [Core Banking Systems Guide](core_banking_systems_guide.md), [Treasury & ALM Guide](treasury_alm_guide.md), [Market Data Consumption Guide](market_data_consumption_guide.md), [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md).

---

**End of guide.** TD Securities names its client surfaces — TDFX, TD One Portal, TDFacilitrade, TDS Automated Trading — and keeps the engines underneath them unnamed; the deepest public view of any TD system estate is the October 2024 enforcement record, with its record US$1.3 billion FinCEN penalty, its US$450 million OCC civil money penalty, and the US$18.3 trillion monitoring gap the DOJ documented. The flags in §11–§12 mark exactly where public evidence ends and inference begins. Beyond that line, the real system map of the bank lives behind the green machine.
