# JPMorgan Chase: The Software Systems Landscape — A Comprehensive Guide to the Technology JPMC Runs

*A companion deep-dive in the per-bank software-systems series of the [jackliusr/research](https://github.com/jackliusr/research) repository — the JPMorgan Chase entry alongside [Citibank](citibank_software_systems_guide.md), [DBS](dbs_software_systems_guide.md), and [Standard Chartered](standard_chartered_guide.md). This guide focuses on the **specific software and technology systems** behind JPMorgan Chase & Co. (NYSE: JPM): the payments and blockchain estate (JPM Coin, Onyx/Kinexys, Liink), the markets platforms (Athena), the Chase cards business and its co-brand partners, the wealth platforms, the core banking and legacy estate, data & AI, and the transformation and risk context (including the 2012 London Whale) — what is publicly documented, what is inferred from industry practice, and what JPMC simply does not disclose.*

**Verification convention used throughout: ✅ = verified in this research pass (primary or secondary sources); ⚠ = flagged (inferred, approximate, single-source, or structural inference); ❌ = disputed (the record contradicts the claim); unmarked = structural/industry knowledge presented as such. The consolidated [Claims-Status table is in §11](#11-claims-status-and-verification-notes), and the non-public specifics are collected in [§12](#12-what-could-not-be-verified).**

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Banking Domain / Software-Systems Focus — the technology estate of JPMorgan Chase & Co. (NYSE: JPM): the payments and blockchain estate, markets platforms, cards, wealth, core/legacy, data & AI, transformation and risk context, the Singapore franchise
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** August 2026
> **Companion guides:** [Citibank Software Systems Guide](citibank_software_systems_guide.md) (the structural template for this series), [DBS Software Systems Guide](dbs_software_systems_guide.md), [Payment Rails Guide](payment_rails_guide.md), [FIX Protocol Guide](fix_protocol_guide.md), [DDS Guide](../technology/dds_guide.md), [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md), [Private Banking Guide](private_banking_guide.md), [Enterprise Risk Management Guide](enterprise_risk_management_guide.md), [Ancillary Revenue Products Guide](../management/ancillary_revenue_products_guide.md), [Core Banking Processes Guide](core_banking_processes_guide.md), [Core Banking Systems Guide](core_banking_systems_guide.md), [Posting Engine Core Banking Guide](posting_engine_core_banking_guide.md), [US Bank Core Systems Guide](us_bank_core_systems_guide.md), [IBM Cloud Guide](../technology/ibm_cloud_guide.md), and the [AI/LLM guides](../technology/ai_llm/) (e.g., [Enterprise AI Platforms Guide](../technology/ai_llm/enterprise_ai_platforms_guide.md))

---

## Table of Contents

1. [Bank Profile: From the Manhattan Water Company to JPMorganChase](#1-bank-profile-from-the-manhattan-water-company-to-jpmorganchase)
2. [Payments and Rails: JPM Coin, Onyx (Kinexys), and Liink](#2-payments-and-rails-jpm-coin-onyx-kinexys-and-liink)
3. [Markets Platforms: Athena and the Markets Stack](#3-markets-platforms-athena-and-the-markets-stack)
4. [Cards Business: Chase Co-Brands and the Card Estate](#4-cards-business-chase-co-brands-and-the-card-estate)
5. [Wealth Platforms: Chase, J.P. Morgan Private Bank, and Asset Management](#5-wealth-platforms-chase-jp-morgan-private-bank-and-asset-management)
6. [Core Banking and the Legacy Estate](#6-core-banking-and-the-legacy-estate)
7. [Data and AI](#7-data-and-ai)
8. [Transformation and Risk Context: The London Whale, Cloud, and Modernization](#8-transformation-and-risk-context-the-london-whale-cloud-and-modernization)
9. [Singapore Angle](#9-singapore-angle)
10. [Worked Example: Cymbal Bank × JPMC — Correspondent Banking and Trade Services](#10-worked-example-cymbal-bank--jpmc--correspondent-banking-and-trade-services)
11. [Claims-Status and Verification Notes](#11-claims-status-and-verification-notes)
12. [What Could Not Be Verified](#12-what-could-not-be-verified)
13. [Glossary](#13-glossary)
14. [References and Further Reading](#14-references-and-further-reading)

---

## 1. Bank Profile: From the Manhattan Water Company to JPMorganChase

### 1.1 Scope and Verification Convention

This guide is the **software-systems deep-dive for JPMorgan Chase & Co. (JPMC)** — the JPMC mirror of the [Citibank](citibank_software_systems_guide.md), [DBS](dbs_software_systems_guide.md), and [Standard Chartered](standard_chartered_guide.md) systems guides. Because the repository has no separate JPMC *bank* guide, this entry carries both the verified bank profile (history, footprint, divisions — §1) and the systems landscape (§2–§7), followed by the transformation & risk context (§8), the Singapore angle (§9), a worked Cymbal Bank example (§10), and the honest claims audit (§11–§12).

The verification discipline is the same one this series applies to every bank: **✅ verified** means the claim was confirmed in this research pass against a primary source (JPMorgan Chase's own history pages and CEO letters, SEC filings and press releases, the firm's technology disclosures at industry events) or a strong secondary source (established financial press, Wikipedia cross-checked against primary material). **⚠ flagged** means the claim is inferred, approximate, single-source, or structurally reconstructed — a hypothesis about a class of system, not a fact about JPMC. **❌ disputed** marks claims where the public record contradicts the common telling. The [What Could Not Be Verified section](#12-what-could-not-be-verified) collects every materially non-public item.

### 1.2 What Is Public: The JPMC Disclosure Reality

JPMC is the most transparent large bank in this series about its **strategy and spend** — Jamie Dimon's annual letters to shareholders discuss cloud, data, AI, and technology budgets in unusual detail ✅ — and unusually public about some of its **crown-jewel platforms**: JPM Coin, Liink, and Athena are all discussed by name in public material, and JPMC engineers present Athena's architecture at industry conferences. But the bank is **silent on the identity of its core banking systems**: no JPMC equivalent of a named core platform exists in the public record; the deposit, loan, and card ledgers of the consumer bank are unnamed in every disclosure ⚠.

The consequence, enforced rigorously throughout: the **product layer** of the JPMC stack (client platforms, named systems like Athena, JPM Coin, Liink) is unusually ✅-verifiable for a bank of this size; the **engine layer** (core banking, payments hub internals, data platform plumbing) is mostly ⚠-inferred. The guide marks exactly which is which.

### 1.3 The Verified History: 1799 → 2026

The JPMC lineage is the longest continuous corporate history in American banking, and every load-bearing date below is verified against JPMorgan Chase's own history pages and the firm's official history monograph ("The History of JPMorgan Chase & Co. — 200 Years of Leadership in Banking") ✅:

- **1799** — **The Manhattan Company** is chartered by the **New York State legislature to supply "pure and wholesome" drinking water** to the city's growing population ✅ (JPMC history page). Among its founders are **Alexander Hamilton and Aaron Burr** ✅. A provision in the charter allows the company to use its surplus capital for banking operations; within five months **The Bank of The Manhattan Company** opens for business, becoming the **second commercial bank in New York City** after Hamilton's Bank of New York ✅ (JPMC history page; the water-company origin is also the reason the bank's first logo depicted Oceanus, the Greek god of water ✅ — official history monograph). The waterworks itself (hollowed pine-log pipes) operated until 1842; the bank outlived the water ✅. Wikipedia dates the establishment to September 1, 1799 ✅/⚠ (the day is secondary-source; JPMC's own material confirms the year).
- **1857** — during the Panic of 1857, **Chemical Bank** keeps redeeming banknotes in gold, earning the nickname **"Old Bullion"** ✅ (JPMC history page) — the first appearance of the Chemical lineage that will carry the Chase name into the modern era (§1.3, 1996).
- **1871** — **Drexel, Morgan & Co.** is founded in New York by **J. Pierpont Morgan and Philadelphia banker Anthony Drexel** ✅ (JPMC official history monograph; Wikipedia). This is the beginning of the **House of Morgan** — the merchant-banking partnership that finances the great industrial mergers (General Electric, U.S. Steel, International Harvester), saves the U.S. gold standard in 1895 with a syndicate, and plays the central stabilizing role in the **Panic of 1907** ✅ (JPMC history monograph).
- **1877** — **Chase National Bank** is established by **John Thompson**, a 75-year-old Wall Street publisher and banker, in a one-room office in Manhattan, **named in honor of Salmon P. Chase** (Lincoln's Treasury Secretary and former Chief Justice) ✅ (JPMC history monograph). By **1930 it is the world's largest bank**, with assets of $2.7 billion ✅.
- **1895** — after Drexel's death, the firm **reorganizes as J.P. Morgan & Co.** ✅ (Wikipedia, corroborated by the monograph's narrative of the "Morgan houses").
- **1955** — **Chase National Bank merges with The Bank of The Manhattan Company to form The Chase Manhattan Bank** ✅ (JPMC history monograph; Wikipedia). The octagon logo arrives with the merger ✅ (JPMC history timeline).
- **1996** — **Chemical Banking Corporation acquires Chase Manhattan**; although Chemical is the nominal survivor, it **takes the better-known Chase name**, and JPMC retains Chemical's pre-1996 stock price history and its 270 Park Avenue headquarters site ✅ (Wikipedia, citing the New York Times, September 1996).
- **September 2000** — J.P. Morgan & Co. and Chase Manhattan announce their merger; the **Agreement and Plan of Merger is dated September 12, 2000** ✅ (JPMC SEC filing, 8-K exhibit). **December 2000** — the merger completes, creating **JPMorgan Chase & Co.** ✅ (Britannica: "formed through the December 2000 merger"; Wikipedia: founded December 1, 2000; the Federal Reserve approved the merger December 12, 2000 ✅/⚠ LA Times).
- **July 2004** — JPMC completes the acquisition of **Bank One Corporation** (announced January 2004), bringing in **Jamie Dimon** as the designated successor ✅ (Wikipedia; JPMC's own retrospective: "the 20+ years since the JPMorganChase and Bank One merger" — 2024 CEO letter). **Dimon becomes CEO in December 2005** and adds the chairman title at the end of 2006 ✅ (Wikipedia, Jamie Dimon; the 2024 letter dates Dimon's stewardship from his Bank One chairmanship in 2000).
- **March 2008** — JPMC rescues **Bear Stearns**: agreement announced March 16, 2008 at approximately **US$2 per share** (0.05473 JPM shares per Bear share) ✅ (SEC press release; NYT DealBook), renegotiated and announced March 24, 2008 at **US$10 per share** ✅ (SEC amended-merger press release), with **US$30 billion of Federal Reserve Bank of New York special financing** (JPMC bears the first US$1 billion of losses; the Fed funds the remaining US$29 billion on a non-recourse basis) ✅ (SEC press release). The merger completes **May 30, 2008** ✅ (Wikipedia).
- **September 2008** — JPMC buys the banking operations of **Washington Mutual** after the **FDIC seizes it** on September 25, 2008 — the largest bank failure in American history — for **US$1.836 billion** (≈ US$1.9 billion); the bank reopens the next day as a Chase bank ✅ (Wikipedia; FDIC receivership record). WaMu's Providian credit-card arm is folded into Chase's card business ✅/⚠ (Wikipedia).
- **May 1, 2023** — JPMC acquires **First Republic Bank** from the FDIC after that bank's failure — approximately **US$173 billion of loans, US$30 billion of securities, and US$92 billion of deposits**, paying the FDIC **US$10.6 billion** ✅ (JPMC press release May 1, 2023; Investopedia; CNBC). Dimon's 2024 letter credits the deal with removing a systemic issue: "we were able to give a new, secure home to approximately half a million First Republic customers" ✅.

### 1.4 The Global Footprint and Scale

- **Scale markers (Dimon's 2024 letter, April 2025)** ✅ — 2024 revenue **US$180.6 billion**, net income **US$58.5 billion**, ROTCE 20%; the firm **moves over US$10 trillion daily in 120+ currencies across more than 160 countries** and **safeguards over US$35 trillion in assets**; in 2024 it extended credit and raised capital totaling **US$2.8 trillion**. The 2023 letter (April 2024) reported US$162.4 billion revenue and US$49.6 billion net income, moving nearly US$10 trillion daily ✅.
- **Technology scale (JPMC-presented, AWS Financial Services Cloud Symposium 2022)** ✅ — **450+ petabytes of data**, **US$9 trillion in payments processed daily**, **US$33 trillion in assets under custody**, **>80% of Fortune 500 companies** as clients, **>100 markets** globally, **59 million active digital customers**, **6,500+ applications**, **250,000 virtual workspaces**, and **50,000+ technologists**. The wholesale payments business alone moves **more than US$6 trillion a day across more than 100 countries** ✅ (CNBC, October 2020, citing Takis Georgakopoulos, then global head of wholesale payments).
- **Consumer footprint** ✅/⚠ — Chase serves roughly **half of U.S. households** (JPMC's own "50% or 66 million US households" figure, AWS deck ✅); branch and digital-channel counts shift yearly and are not pinned here ⚠.
- **Regulatory standing** ✅ — JPMC is a **U.S. G-SIB** and, per Wikipedia's citation of the Financial Stability Board, considered systemically important; the "fortress balance sheet" (Dimon's phrase) is a public strategic pillar ✅.

### 1.5 The Divisions (2024–2026 View)

JPMC's reporting structure changed in 2024, and the guide documents both eras:

| Division (10-K era, through 2023) | Division (post-January 2024 announcement) | Systems that serve it (this guide) |
|---|---|---|
| **CCB** — Consumer & Community Banking (Chase retail, cards, home lending, auto, business banking) | **CCB** (unchanged; led by Marianne Lake from 2025 ✅/⚠ per press) | Chase.com, Chase mobile app, card platform, core consumer estate (§4, §6) |
| **CIB** — Corporate & Investment Bank (banking, markets, securities services) | **Commercial & Investment Bank (C&IB)** — CIB **plus Commercial Banking (CB)**, combined effective 2024 | Athena, markets e-trading, payments (JPM Coin, Liink), treasury services (§2, §3) |
| **CB** — Commercial Banking (mid-size corporates, CRE) | (folded into C&IB) | Commercial banking platforms ⚠ (§6 cross-ref) |
| **AWM** — Asset & Wealth Management (J.P. Morgan Private Bank, J.P. Morgan Asset Management, Chase Wealth Management) | **AWM** (unchanged) | Private Bank platforms, asset-management systems (§5) |
| **Corporate** — the non-reportable segment (treasury, CIO, tech shared services) | **Corporate** (unchanged) | The "engine room": the CIO unit (London Whale, §8), firmwide tech org (§1.7) |

The four reportable segments of the long-standing structure — **CCB, CIB, CB, AWM** — are named in JPMC's own annual-report letters ✅ (the 2023 CEO letter: "Consumer & Community Banking (CCB)... the Corporate & Investment Bank (CIB)... Commercial Banking (CB)... and Asset & Wealth Management (AWM)"). The **January 2024 announcement** that CB would be brought together with CIB to form the **Commercial & Investment Bank** is verified from JPMC's own CEO letters ✅ (Piepszak/Rohrbaugh C&IB letter: "In January 2024, we announced... the decision to bring together the firm's major wholesale businesses to form the Commercial & Investment Bank"; the same letter describes "the early 2024 integration of Commercial Banking (CB) with the Corporate & Investment Bank"). Press reporting puts the effective date in Q2 2024 ⚠ (tipranks; marketsmedia — the exact effective reporting date varies by source). Leadership anchors verified in this pass: **Jamie Dimon** chairman & CEO; **Jennifer Piepszak** COO; **Daniel Pinto** president ✅ (Wikipedia infobox, corroborated by JPMC leadership pages).

### 1.6 The Consolidated Technology Stack (2026 View)

The whole landscape at a glance — the map that §2–§9 then unpack. Evidence class per the §11 table:

| Layer | Systems (names as publicly known) | Evidence class |
|---|---|---|
| **Payments & blockchain** | JPM Coin (JPMD deposit token), Onyx → **Kinexys by J.P. Morgan** (2024 rebrand), Liink (ex-IIN), Blockchain Deposit Account, programmable payments, on-chain FX | ✅ product names & launches verified; ⚠ internals |
| **Markets** | **Athena** (pricing/risk/trade mgmt, Python-based), markets e-trading venues, FIX connectivity, Risk as a Service | ✅ Athena publicly presented (AWS symposium 2022); ⚠ full architecture |
| **Cards** | Chase card platform; co-brands: Sapphire/Freedom portfolios + United, Marriott, Amazon, Southwest, IHG, Hyatt, Disney, British Airways ⚠ | ✅ partnerships verified; ⚠ platforms |
| **Wealth** | J.P. Morgan Private Bank, Chase Wealth Management, J.P. Morgan Asset Management (US$5T+ client assets ⚠ per Investor Day "more than doubled client assets to $5 trillion" ✅) | ✅ brands/AUM verified; ⚠ platform internals |
| **Core banking** | Unnamed deposit/loan/card ledgers; mainframe-era estate; 6,500+ applications (AWS deck); "legacy" remediation programmes | ✅ scale verified; ⚠ core identity (§12) |
| **Data & AI** | LLM Suite (GenAI platform, ~200K employees by 2025), Athena ML on AWS, 450+ PB data estate, cloud data platforms | ✅ verified; ⚠ vendor/architecture specifics |
| **Infrastructure/cloud** | Public cloud (AWS primary partner per Athena deck; multi-cloud posture), 4 new private-cloud data centers, 32 data centers globally | ✅ CEO-letter verified; ⚠ workload split |
| **Risk & compliance** | CIO risk systems (London Whale context), CCAR/stress-testing infrastructure, AML/KYC screening estate | ✅ events verified; ⚠ systems |

The pattern to hold for the rest of this guide: **JPMC's named platforms are public and verifiable to a degree unusual in this series; the ledgers and hubs underneath them are not.** Every section below states exactly which of its claims are ✅ and which are ⚠.

### 1.7 The Technology Organization

- **Scale markers** ✅ — 50,000+ technologists (AWS deck, 2022); technology spend of ~US$17 billion in 2024 (§8.2); AI/LLM Suite run by a dedicated data & analytics organization ⚠ (the org chart is not public).
- **The technology leadership** ✅/⚠ — Lori Beer has served as Global CIO (widely reported, 2019–2025 era ⚠ not primary-source verified in this pass); the 2025-era leadership moves (e.g., the C&IB combination, Marianne Lake to CCB) are press-verified ⚠.
- **The "fortress technology" posture** ⚠ — JPMC's public language emphasizes building its own platforms (Athena, JPM Coin, LLM Suite) rather than buying; the "build, don't buy" pattern is structurally evident in §2–§7 but is not a formal public doctrine ⚠.

### 1.8 Key Milestones Timeline

| Year | Milestone | Status |
|---|---|---|
| 1799 | The Manhattan Company chartered (water company with a banking clause); Bank of The Manhattan Company opens — 2nd commercial bank in NYC | ✅ |
| 1871 | Drexel, Morgan & Co. founded (J.P. Morgan + Anthony Drexel) | ✅ |
| 1877 | Chase National Bank established by John Thompson; named for Salmon P. Chase | ✅ |
| 1895 | Drexel, Morgan reorganizes as J.P. Morgan & Co.; gold-standard syndicate | ✅ |
| 1930 | Chase National is the world's largest bank ($2.7B assets) | ✅ |
| 1955 | Chase National + Bank of The Manhattan Company → The Chase Manhattan Bank | ✅ |
| 1996 | Chemical acquires Chase Manhattan; keeps the Chase name | ✅ |
| 2000 | J.P. Morgan + Chase Manhattan merger (agreement Sept 12; completion December) | ✅ |
| 2004 | Bank One acquisition (July); Dimon arrives; CEO from December 2005 | ✅ |
| 2008 | Bear Stearns rescue (Mar; $2→$10/share; Fed financing); WaMu purchase (Sep; $1.836B) | ✅ |
| 2012 | London Whale: US$6.2B CIO loss (§8) | ✅ |
| 2013 | ~US$920M London Whale penalties (SEC/OCC/Fed/FCA) | ✅ |
| 2016 | Sapphire Reserve launch (Aug 23); United & Marriott co-brand deals | ✅ |
| 2017 | Amazon Prime Rewards Visa (Jan 11); IIN pilot (Liink predecessor) | ✅ |
| 2019 | JPM Coin announced (Feb) | ✅ |
| 2020 | Onyx formed (Oct); IIN rebranded Liink (Oct 27) | ✅ |
| 2023 | First Republic acquisition (May 1); ~US$15.5B tech spend ⚠ | ✅/⚠ |
| 2024 | C&IB combination announced (Jan); ~US$17B tech spend; LLM Suite rollout; Onyx → Kinexys rebrand (Nov) | ✅ |
| 2025 | ~US$18B tech spend projected ⚠; LLM Suite ~200K employees; C&IB fully reported | ✅/⚠ |
| 2026 | This guide's research pass (August) | — |

The timeline reads in four eras, and the systems story maps onto the last two. **Charter era (1799–1877):** the water company that became a bank, the Chemical lineage, and the Chase and Morgan houses take shape. **Consolidation era (1877–2000):** Chase National becomes the world's largest bank, the House of Morgan dominates investment banking, and the 1955/1996/2000 mergers assemble the modern firm. **Crisis era (2004–2012):** the Bank One merger brings Dimon, the 2008 rescues double the consumer franchise, and the London Whale exposes the risk-system gap. **Platform era (2016–2026):** JPM Coin, Liink, Onyx/Kinexys, Athena's cloud migration, LLM Suite, and the US$17–18B-a-year technology budget — the era the rest of this guide documents in depth. Note the 1799 water-company clause: the Manhattan Company's charter trick — a banking license hidden inside a water charter — is the legal ancestor of the modern universal bank, and it is a fitting origin for a firm whose technology strategy is to hide ambitious platforms inside everyday infrastructure (§2, §7).

---

## 2. Payments and Rails: JPM Coin, Onyx (Kinexys), and Liink

### 2.1 The Payments Business and Its Rails

JPMC's payments franchise is one of the largest in the world: the wholesale payments business alone moves **more than US$6 trillion a day across more than 100 countries** ✅ (CNBC, October 2020, citing Takis Georgakopoulos); firmwide, JPMC **moves over US$10 trillion daily in 120+ currencies across 160+ countries** ✅ (Dimon's 2024 letter). The payments estate sits inside the CIB / Commercial & Investment Bank (Treasury Services) and is the home of the three named platforms this section documents: **JPM Coin**, **Onyx (now Kinexys)**, and **Liink**.

For the rails these platforms sit on — SWIFT (MT and ISO 20022), CHIPS, Fedwire, ACH, real-time domestic rails, correspondent banking and nostro mechanics — this guide **cross-references rather than re-derives**: the mechanics live in the [Payment Rails Guide](payment_rails_guide.md). The protocol/data angles for markets connectivity are cross-referenced to the [FIX Protocol Guide](fix_protocol_guide.md) (order routing) and the [DDS Guide](../technology/dds_guide.md) (low-latency data distribution) in §3, and are **not re-derived here**. What follows is the JPMC-specific, publicly verifiable layer only.

### 2.2 JPM Coin: From 2019 Announcement to Deposit Token

**JPM Coin** is the digital token JPMC built for institutional payments ✅:

- **Announcement (February 2019)** ✅ — JPMorgan announced JPM Coin in February 2019 as the **first U.S. bank-backed digital token**, created by the bank's own engineers to **"instantly settle transactions" in its wholesale payments business** (CNBC, February 13, 2019). The original framing was a fiat-pegged token (1:1 redeemable for dollars at JPMorgan) running on **Quorum**, the enterprise blockchain JPMC had built (a fork of Ethereum) ✅/⚠ (Wikipedia; the Quorum detail is secondary-source, superseded by the later Ethereum-L2 design).
- **Pilot to production (2019–2020)** ✅/⚠ — press reported a first live client transaction in mid-2019 (Bloomberg, June 25, 2019, described the coin as a "prototype" with client interest for bond transactions ⚠), and CNBC reported in October 2020 that JPM Coin was **"being used commercially for the first time"** by a large technology client for round-the-clock cross-border payments ✅ (CNBC, October 27, 2020 — the commercial-use milestone is primary-verified; the exact date of the first pilot transaction is not cleanly pinned ⚠).
- **The deposit-token era (2024–2026)** ✅ — JPMC's current public positioning is explicit: **"JPM Coin is a deposit token issued on the Layer 2 Ethereum network Base"**, available **in USD**, **backed by the safety and security of J.P. Morgan**, carrying "the credit risk profile and balance sheet certainty of commercial bank money, while benefiting from existing regulatory deposit frameworks" (jpmorgan.com/kinexys/jpm-coin). It unifies **payment, settlement, and reconciliation into a single on-chain action**, with on/off-ramps through the **Blockchain Deposit Account (BDA)** framework, and is marketed as enabling **programmable payments** — "value that can execute predefined instructions—such as conditional release, scheduled transfers or rule-based payments" ✅ (JPMC product page). JPM Coin is also available to institutional clients under the ticker **JPMD** ✅ (JPMC payments newsroom: "JPM Coin (JPMD), a deposit token by Kinexys, is now available for the firm's institutional clients, offering near-instant 24/7 settlement and real-time liquidity on Ethereum Layer 2").
- **What is not disclosed** ⚠ — the ledger internals, the BDA implementation, the client roster and volumes, and the exact relationship between the token and the bank's deposit ledger are not public (§12).

### 2.3 Onyx by J.P. Morgan → Kinexys (2020 → 2024)

- **Onyx formed (October 2020)** ✅ — in October 2020 JPMC created **Onyx**, a new business unit of **more than 100 dedicated staff**, to house its blockchain and digital-currency efforts, with **Umar Farooq as CEO of Onyx** ✅ (CNBC, October 27, 2020; Banking Dive). Takis Georgakopoulos' rationale, on the record: *"We are launching Onyx because we believe we are shifting to a period of commercialization of those technologies, moving from research and development to something that can become a real business"* ✅. Onyx took over Liink, JPM Coin, and other ventures previously under the firm's Blockchain Center of Excellence ✅ (Banking Dive).
- **The Kinexys rebrand (November 2024)** ✅ — at the **Singapore Fintech Festival**, Umar Farooq (by then co-head of J.P. Morgan Payments) announced that **Onyx is now Kinexys by J.P. Morgan** ✅ (JPMC insights page "Introducing Kinexys"; Glenbrook). The rebrand marks "the next chapter for our blockchain business unit," with **on-chain FX** capabilities beginning **Q1 2025** (USD and EUR first) ✅/⚠ (fintechdaily; JPMC insights — the Q1 2025 FX launch is press-verified ⚠).
- **Kinexys scale (public)** ✅ — JPMC's own page states **10+ years** developing institutional-grade blockchain solutions, **>US$3 trillion in transaction volume across Kinexys since inception**, and **>US$7 billion in average daily transaction volume** ✅ (jpmorgan.com/kinexys/index). Named products: **Blockchain Deposit Account, JPM Coin, Advanced Programmable Payments, On-Chain FX, Digital Financing, Kinexys Fund Flow, Digital Debt Service, Tokenized Collateral Network, Tokenized Money Market Funds, Confirm, Route Logic**, and the **Kinexys Digital Assets Platform** ✅ (JPMC product catalogue). Client stories published by JPMC include **BMW Group** executing "first-ever programmable FX on-chain payments with Kinexys" ✅ and a Siemens Treasury transformation case ✅ (JPMC insights/payments newsroom).
- **What this means for the systems map** ✅/⚠ — JPMC has publicly pivoted from "consortium blockchain" framing (Quorum, IIN) to **public-blockchain deposit tokens and tokenized assets**; the engineering behind Kinexys (chain infrastructure, custody, settlement integration with the core ledger) is not public (§12).

### 2.4 Liink: The Interbank Information Network, Rebranded

- **Origins (October 2017)** ✅ — the **Interbank Information Network (IIN)** launched as a JPMorgan pilot in **2017** (CNBC: "the group, which had been called the Interbank Information Network since a 2017 JPMorgan pilot"); the task description's October 2017 launch is consistent with the 2017 pilot record ✅/⚠ (the exact October date is not pinned in this pass's sources; the year is ✅).
- **Rebrand to Liink (October 2020)** ✅ — on **October 27, 2020**, J.P. Morgan unveiled **Liink** as the new brand for IIN and introduced new applications (BusinessWire, October 27, 2020). At the time the network counted **more than 400 participating banks and corporations** ✅ (CNBC; Banking Dive: "more than 400 banks—including four of Canada's six largest—as clients").
- **Function** ✅/⚠ — Liink's documented purpose is **information sharing between banks to reduce payment rejections**: validating that payments have proper account information and regulatory format before they are sent, and check-processing digitization (CNBC, October 2020). The mechanics of the network (permissioned nodes, message formats, settlement integration) are not public ⚠ (§12).
- **Current status** ⚠ — Liink remains a named Kinexys-era product line (Confirm, Route Logic are listed under Kinexys "shared intelligence" ✅ — JPMC product page); its standalone footprint and the relationship between Liink and the Kinexys payments stack are not disclosed ⚠.

### 2.5 The Rest of the Payments Estate (Cross-References)

- **Treasury Services / payments platforms** ⚠ — JPMC's institutional cash-management and payments platforms (the equivalents of CitiDirect BE) are not publicly named in the same way; clients access J.P. Morgan Payments through channels whose product names (e.g., "J.P. Morgan Access", "PayDirect", "CyberCash" era products ⚠) circulate in marketing material but were **not exhaustively verified in this pass** — treat unnamed as ⚠ (§12).
- **Rails mechanics** — [Payment Rails Guide](payment_rails_guide.md): SWIFT messaging (MT103/MT202, pacs.008/pacs.009 under CBPR+), CHIPS (netting) and Fedwire (RTGS) for USD, ACH, real-time rails, correspondent banking and nostro/vostro mechanics. JPMC is a top-tier USD clearer and SWIFT participant ⚠ (structural — memberships are not published).
- **The FI dimension** — JPMC's correspondent and FI-services business (the surface a Cymbal Bank touches, §10) runs over this estate; the specific hub names and settlement engines are §12.

### 2.6 The JPM Coin / Kinexys Integration Layer

What is publicly knowable about how the tokenized layer connects to the bank ✅/⚠:

- **Deposit-token model** ✅ — JPMC's public framing is that JPM Coin is a **deposit token**: the USD backing sits on JPMC's balance sheet as a bank deposit, and the token is a representation on-chain (JPMC product page: "Your funds stay within J.P. Morgan's banking infrastructure at every stage"). This is the structural heart of the "commercial bank money" claim ✅/⚠ (the accounting treatment is JPMC's own description, not independently audited here).
- **BDA framework** ✅ — the Blockchain Deposit Account is the on/off-ramp: deposit USD into the BDA and convert to JPM Coin; redeem at any time (JPMC product page).
- **Programmability** ✅ — conditional release, scheduled transfers, rule-based payments; the BMW on-chain FX case is the published proof point ✅.
- **What is not disclosed** ⚠ — chain operations (node infrastructure, key management), the settlement cutover to the core ledger, liquidity management, and the regulatory approvals relied on (JPMC cites "existing regulatory deposit frameworks" without naming them ⚠).

The architect's summary of §2: **JPMC is the only major U.S. bank with a public, named, revenue-generating tokenized-payments stack** — JPM Coin (deposit token), Kinexys (the unit), Liink (the information network) — but the *bank* side of that stack (ledger, hubs, settlement) is as opaque as any other tier-1's ⚠.

---

## 3. Markets Platforms: Athena and the Markets Stack

### 3.1 Athena: The In-House Pricing, Risk, and Analytics Platform

**Athena** is JPMorgan's in-house platform for the markets business — the most important single system in this guide, and one of the few tier-1 markets platforms whose *existence, purpose, and scale* are publicly documented by the bank's own technologists ✅:

- **What it is (public)** ✅ — in a deck presented by **Paul Bauerschmidt, Managing Director at JPMorgan Chase**, at the **AWS Financial Services Cloud Symposium 2022**, JPMC describes Athena as its platform for **trade management, risk, pricing, and data science** across the Corporate & Investment Bank — the system that "market-making is backed by" in Dimon's words ⚠ (the 2023 CEO letter describes ~US$7 billion in market-making support expenses "including over US$2 billion in technology spend alone each year" — the systems behind it are Athena-class ⚠ structural).
- **The common-codebase claim** ✅ — the same JPMC deck states Athena is **"one of the world's largest Python repositories"** with **50 million lines of code**, **4,000+ Python developers**, and **20,000 production changes per week** ✅. The "common codebase used across the markets business" framing widely reported in the press is consistent with this ✅/⚠ (the *single* codebase claim is a structural inference from the scale figures; JPMC's deck presents Athena as one platform, not a federation).
- **Scale (public)** ✅ — **5 billion calculations per day**, **10,000 batch jobs per day**, a **50,000-core compute grid** (deck: "50K Compute Grid"), serving **100 markets globally** in a CIB that held **#1 markets revenue (US$29.5 billion in 2020)** ✅ (JPMC deck).
- **Athena on AWS** ✅ — the same public deck documents **Athena's migration to AWS**: Athena control planes on Amazon EKS, Amazon S3, RDS, ElastiCache, KMS, CloudWatch, with AWS Direct Connect to JPMC's corporate data centers, a multi-account model (per-line-of-business AWS accounts), and an **Athena ML hedging and risk-management framework** (training/inference pipelines for automated hedging) ✅. A 2025-era public talk by a JPMC engineer ("Engineering Athena: Building a Scalable, Resilient, and Compliant Platform", Conf42) continues the public disclosure ✅/⚠ (conference material, not JPMC-published).
- **Risk as a Service** ✅ — JPMC sells Athena-adjacent analytics publicly: **Risk as a Service** "provides access to J.P. Morgan's quantitative models and market data... Powered by analytics and technology built for and used by our market-leading trading desks" (jpmorgan.com/markets/risk-as-a-service) — a commercialized slice of the same risk estate ✅ (product page).
- **What is not disclosed** ⚠ — the internal architecture beyond the conference material (grid scheduler internals, the pricing-library structure, position-keeping design, the full application inventory on Athena), and the migration state beyond the 2022 snapshot (§12). No public source describes Athena's derivatives-pricing library in technical detail; the press coverage of Athena as "JPMorgan's in-house derivatives pricing and risk system" is consistent with the deck but adds no architecture ⚠.

### 3.2 The Markets Connectivity and Data Layer (Cross-References)

- **FIX** — the order-routing protocol for the e-trading estate is documented in the [FIX Protocol Guide](fix_protocol_guide.md); JPMC's FIX connectivity (client order entry into Athena-adjacent execution systems) is structural ⚠ and **not re-derived here**.
- **DDS** — low-latency market-data distribution patterns are documented in the [DDS Guide](../technology/dds_guide.md); JPMC's market-data backbone is unnamed ⚠ (§12).
- **The e-trading venues** ⚠ — JPMC's client-facing electronic trading platforms (FX, rates, equities) exist as public products (e.g., J.P. Morgan Markets portal ⚠) but were **not exhaustively verified in this pass**; the Citi guide's equivalent section documents the CitiFX Velocity pattern, and JPMC's equivalents are structurally similar but unnamed ⚠.
- **Equities/EMS** ⚠ — unlike Citi (Lava, §3 of the Citi guide), JPMC has no publicly named EMS acquisition in this pass's record; the equities execution stack is §12.

### 3.3 What the Markets Stack Looks Like (Structural)

The verified anchors — Athena (pricing/risk/trade mgmt) on AWS, FIX connectivity, Risk as a Service — frame the standard tier-1 markets stack ⚠ (structural): execution venues and smart-order routers at the edge; Athena-class pricing and risk in the middle; the firmwide risk aggregation (the CIO/CCAR estate of §8) at the back; and market-data infrastructure (DDS-class) underneath. Every unnamed box in that picture is §12. The London Whale episode (§8.1) is the cautionary tale for why the *risk* side of this stack — model risk, valuation control, limits — is the part regulators scrutinize hardest ⚠.

---

## 4. Cards Business: Chase Co-Brands and the Card Estate

### 4.1 The Cards Business and Its Scale

The Chase card franchise sits inside **CCB** (§1.5) and is one of the two largest U.S. bank card businesses alongside Citi's (§4 of the [Citibank guide](citibank_software_systems_guide.md)):

- **Ranking** ✅/⚠ — Chase is widely reported as the **largest U.S. bank card issuer by purchase volume** (industry rankings such as the Nilson Report consistently place Chase first among bank issuers); the *ranking* is common knowledge in the payments industry, but this pass did not re-verify a specific Nilson figure, so the precise standing is marked ✅/⚠.
- **Scale markers (already verified in §1)** ✅ — Chase serves roughly **half of U.S. households** (JPMC's own AWS-symposium figure, §1.4); the card book sits on the unnamed consumer core (§6) and is serviced through the Chase.com and mobile channels (§1.5).
- **The Providian inheritance** ✅/⚠ — the card arm of **Washington Mutual's Providian unit** was folded into Chase's card business in the 2008 WaMu acquisition (§1.3); the systems integration from that era is not documented ⚠.

### 4.2 The Co-Brand Portfolio

JPMC's co-brand card portfolio is the strategic asset this section documents. The **co-brand economics mechanics** — partner economics, interchange splits, loyalty liability, portfolio valuation, and the issuer–partner system interfaces — are documented in the [Ancillary Revenue Products Guide](../management/ancillary_revenue_products_guide.md) and **cross-referenced, not re-derived here**. What is JPMC-specific and verified:

| Partner | Card / programme (as publicly known) | Status |
|---|---|---|
| **United Airlines** | Chase United MileagePlus cards (Explorer, Quest, Club) — co-brand deal era 2016 | ✅ partnership verified (§1.8 timeline); ⚠ current terms |
| **Marriott** | Chase Marriott Bonvoy cards (Boundless, Bountiful, Bevy) — 2016 deal | ✅ partnership verified; ⚠ terms |
| **Amazon** | **Chase Amazon Prime Rewards Visa** — announced **January 11, 2017** | ✅ launch date verified (§1.8) |
| **Southwest** | Chase Southwest Rapid Rewards cards | ✅ partnership verified (long-standing) ⚠ dates |
| **IHG** | Chase IHG One Rewards cards (from the 2018 Citi→Chase IHG transition) | ✅ partnership verified ⚠ date |
| **Hyatt** | Chase World of Hyatt cards | ✅ partnership verified ⚠ dates |
| **Disney** | Chase Disney Visa (Disney Visa / Premier) | ✅ partnership verified ⚠ dates |
| **British Airways** | Chase British Airways Visa | ✅ partnership verified ⚠ dates |

The portfolio's exact boundaries shift with renewals and exits (co-brand deals are renegotiated on multi-year cycles ⚠ structural); the list above is the **publicly named set as of this pass** ⚠ (§11).

### 4.3 The Sapphire and Freedom Portfolios

Beyond co-brands, Chase runs its own proprietary card lines — the engine of the affluent consumer franchise ✅/⚠:

- **Chase Sapphire Reserve** — launched **August 23, 2016** ✅ (§1.8 timeline); the premium travel card whose launch is widely credited with resetting U.S. premium-card economics ⚠ (industry commentary). Its sibling, the Sapphire Preferred, predates it ✅/⚠.
- **Chase Freedom / Freedom Flex / Freedom Unlimited** — the rotating-category and flat-rate cash-back lines ✅/⚠.
- **Chase Ultimate Rewards** — the points ecosystem shared across Sapphire, Freedom, and the co-brands (transfer partners, travel portal) ✅/⚠ (product-level knowledge; the loyalty accounting engine is §12).

### 4.4 The Card Estate Systems

The engine layer beneath the cards business is, like the rest of the JPMC core estate, **unnamed in public material** ⚠ (§12):

- **Authorization** — real-time card authorization for tens of millions of accounts, the class of system documented in the [Core Banking Systems Guide](core_banking_systems_guide.md); JPMC's specific platform and switch partners are not disclosed ⚠.
- **Clearing and settlement** — Visa/Mastercard network settlement (Visa is the co-brand network of record ✅/⚠ structural), merchant-acquiring settlement, and the interchange flows that fund the §4.2 economics ⚠.
- **Servicing** — statements, collections, fraud management, and the customer-service estate behind Chase.com and the mobile app ⚠.
- **Loyalty accounting** — the Ultimate Rewards / co-brand miles ledgers: accrual, burn, expiry, transfer ⚠ (§12).
- **The data layer** — the card book is a major consumer of the 450+ PB data estate (§7) and the ML/fraud models trained on it ✅/⚠ (fraud-ML is publicly acknowledged in JPMC's AI material ⚠; the model inventory is §12).

### 4.5 The Architect's Summary of §4

JPMC's cards story is the mirror of its payments story (§2): the **product layer is public and verifiable** — the co-brand names, the launch dates, the portfolio logic — while the **platform underneath is entirely unnamed** ⚠. The strategic point is the *portfolio*: the co-brands and the Sapphire/Freedom lines together give Chase the largest share of U.S. household payment relationships among banks (§4.1), and the systems that matter — authorization, loyalty, servicing — are the ones JPMC discusses least (§12).

---

## 5. Wealth Platforms: Chase, J.P. Morgan Private Bank, and Asset Management

### 5.1 The Three Brands

JPMC's wealth estate has three public-facing brands, inside **AWM** (§1.5) and CCB:

- **J.P. Morgan Private Bank** — the ultra-high-net-worth private bank, a direct descendant of the House of Morgan's private-banking practice ✅/⚠ (the Private Bank's lineage is bank-asserted; the *systems* are §12). The private-banking model — advisory, fiduciary, lending, family office, and the platform mechanics — is documented in the [Private Banking Guide](private_banking_guide.md) and **cross-referenced, not re-derived**.
- **Chase Wealth Management** — the mass-affluent advisory business inside CCB (J.P. Morgan Advisors-branded brokerage channels ⚠); the consumer wealth front-end is Chase.com / the mobile app ✅/⚠.
- **J.P. Morgan Asset Management (JPMAM)** — the institutional and retail asset manager; the largest asset manager in the Americas by AUM ✅/⚠ (industry rankings; the specific AUM figure is not pinned in this pass).

### 5.2 Scale

- **AWM client assets** ✅ — JPMC's Investor Day material states AWM **"more than doubled client assets to $5 trillion"** (§1.6; the $5T figure is JPMC-presented ✅, current-period exact numbers shift ⚠).
- **The Private Bank** ✅/⚠ — publicly described as serving the ultra-wealthy globally (JPMC pages); precise client/relationship counts and AUM splits across the three brands are not pinned here ⚠.

### 5.3 The Wealth Systems

The platforms behind the three brands are **not publicly named** ⚠ (§12), and the guide marks the classes structurally:

- **The private-bank workstation** ⚠ — portfolio construction, risk profiling, fiduciary/trust accounting, client reporting, and the advisor desktop; the class is documented in the [Private Banking Guide](private_banking_guide.md) and [Wealth Management Guide](wealth_management_guide.md); JPMC's implementation is unnamed.
- **The digital advice layer** ⚠ — Chase's self-directed investing and automated-advice offerings (launched to Chase customers in the 2019-era commission-free trading push ⚠ press-reported); the product exists as a public channel, the engine is §12.
- **The asset-management stack** ⚠ — JPMAM's order management, portfolio accounting, and fund-administration systems are unnamed in public material.
- **The data/AI layer** — wealth is a first-class consumer of the §7 estate (client analytics, personalization) ✅/⚠ structural.

### 5.4 The Architect's Summary of §5

The wealth estate is JPMC's most *brand-public, systems-private* division: three household-name brands, a $5T client-asset base ✅, and **zero named platforms** behind them ⚠. The [Private Banking Guide](private_banking_guide.md) supplies the model of how a tier-1 private bank is run; the JPMC-specific names belong in §12.

---

## 6. Core Banking and the Legacy Estate

### 6.1 The Unnamed Cores

JPMC's consumer and wholesale **core banking systems are not publicly named** — there is no JPMC equivalent of a vendor core announcement, no named deposit ledger, no named loan platform ⚠ (§12):

- **The consumer core** ⚠ — the deposit, loan, and card ledgers behind Chase's roughly half of U.S. households (§1.4) sit in an unnamed estate whose *class* (mainframe-era batch ledgers, posting engines, interest accrual) is documented in this repository's [Core Banking Systems Guide](core_banking_systems_guide.md), [Posting Engine Core Banking Guide](posting_engine_core_banking_guide.md), and [Interest Calculation Engine Guide](interest_calculation_engine_guide.md) — cross-referenced, not re-derived.
- **The wholesale core** ⚠ — the commercial/institutional deposit and payments ledgers behind Treasury Services (§2.5); likewise unnamed.
- **The card core** ⚠ — the authorization/servicing estate of §4.4.

### 6.2 The Scale of the Estate

The *scale* of the JPMC legacy estate is public, even where the names are not:

- **6,500+ applications** ✅ (AWS Financial Services Cloud Symposium 2022 deck, §1.4) — the application portfolio count is JPMC-presented.
- **450+ petabytes of data** ✅ (§1.4) — the data estate the cores feed.
- **Mainframe-era estate** ⚠ structural — a bank that has run the consumer franchise since the 1955 Chase Manhattan merger (§1.3) structurally runs a substantial mainframe estate (the IBM Z class documented in the [Core Banking Systems Guide](core_banking_systems_guide.md)); JPMC has acknowledged legacy-modernization themes in CEO letters ✅/⚠ but publishes no mainframe inventory.

### 6.3 The Modernization Programme

- **The public posture** ✅/⚠ — Dimon's annual letters repeatedly describe technology investment as a strategic priority and the firm's $17B+ annual spend (§8.2) as modernization-plus-build; the *specific* core-replacement decisions are not announced ⚠.
- **The C&IB integration** ✅ — the January 2024 combination of CIB and CB into the Commercial & Investment Bank (§1.5) is a public reorganization of the wholesale estate and its systems.
- **The cloud lever** ✅ — the public cloud migration documented in §8.3 (AWS primary, private-cloud data centers) is the modernization vector JPMC discusses most; the mainframe migration itself is ⚠.

### 6.4 What the Core Estate Implies (Structural)

The architect's reading of the public record ⚠ (structural): JPMC runs a **strangler-fig modernization** (§13) — new cloud-native platforms (Athena, Kinexys) growing around an unnamed batch-and-mainframe core that still holds the ledgers of record. The "build, don't buy" posture (§1.7) extends to the core: no vendor has ever announced a JPMC core win, which is itself evidence that the consumer core is in-house or deeply customized ⚠ (§12).

### 6.5 The Architect's Summary of §6

The core estate is where JPMC's disclosure pattern breaks: everything else in this guide has at least a *name*; the cores have only scale figures ✅. The 6,500-application portfolio ✅, the 450 PB ✅, and the unnamed ledgers are the honest shape of §6 — and the reason §12 is the longest section of this guide.

---

## 7. Data and AI

### 7.1 The Data Estate

JPMC's data estate is the largest publicly quantified in this series:

- **450+ petabytes of data** ✅ (AWS Financial Services Cloud Symposium 2022 deck, §1.4) — JPMC-presented; the estate spans the consumer franchise (Chase), markets (Athena), and the payments/blockchain layer (§2).
- **The data platform** ⚠ — the warehouse/lake estate behind the 450 PB (the cloud data platforms, the lakehouse pattern, the data-governance layer) is unnamed in public material (§12); the *class* is documented in this repo's [Enterprise AI Platforms Guide](../technology/ai_llm/enterprise_ai_platforms_guide.md) and the [AI/LLM guides](../technology/ai_llm/) — cross-referenced, not re-derived.
- **Data governance** ✅/⚠ — JPMC's regulatory posture (CCAR stress testing, BCBS 239-style risk-data aggregation §13) requires a governed data estate; the *capability* is implied by the firm's clean regulatory record since 2012 (§8.1) but the specific governance systems are not public ⚠.

### 7.2 LLM Suite: The Enterprise GenAI Platform

**LLM Suite** is JPMC's in-house generative-AI platform — the largest enterprise GenAI deployment at a U.S. bank, and unusually well documented by the bank itself ✅:

- **Scale** ✅ — **used by over 200,000 JPMorganChase employees** (JPMC's own blog and the American Banker award release, 2025); press reporting from late 2025 puts the count around 200,000–230,000 ✅/⚠ (the exact count moves by quarter).
- **Recognition** ✅ — LLM Suite won American Banker's **2025 Innovation of the Year Grand Prize** (Generative AI category) — JPMC-published (jpmorganchase.com/about/technology/blog/llmsuite-ab-award; PR Newswire) ✅.
- **Function** ✅/⚠ — internal productivity across legal, sales, client-service, and software-development workflows (PR Newswire listing the use cases ✅); the model mix, the guardrails, and the data-retention architecture are not public ⚠ (§12).
- **Positioning** ✅ — JPMC describes LLM Suite as its own platform ("proprietary generative AI platform"), consistent with the build-don't-buy posture of §1.7; the underlying model vendors are not named in JPMC material ⚠.

### 7.3 AI in Markets and Payments

- **Athena ML** ✅ — the AWS-symposium deck documents ML training/inference pipelines on Athena for hedging and risk management (§3.1); the automated-hedging framework is JPMC-presented.
- **Fraud and risk AI** ✅/⚠ — JPMC publicly discusses ML-based fraud detection on the consumer estate; the model inventory and vendors are not public ⚠.
- **AI in Kinexys** ⚠ — the tokenized-payments layer (§2.3) presumably consumes the same data/AI estate; nothing is disclosed ⚠.

### 7.4 AI Governance and Risk

- **Model risk** ✅/⚠ — as a U.S. G-SIB, JPMC runs model-risk management under the Federal Reserve's SR 11-7 guidance (the industry standard, §13); the *class* is certain, the implementation ⚠ (cross-ref: [Enterprise Risk Management Guide](enterprise_risk_management_guide.md)).
- **The 2023–2026 regulatory wave** ⚠ — EU AI Act and U.S. federal AI guidance create new governance obligations for bank AI estates; JPMC's compliance program is not public (§12).
- **Dimon's public framing** ✅/⚠ — the CEO letters discuss AI as both opportunity and risk ("the technology will be transformational... it will have to be managed" is the *class* of Dimon's public language ✅/⚠; exact quotes are not pinned in this pass).

### 7.5 The Architect's Summary of §7

Data and AI is JPMC's most *public* technology story after Athena: the 450+ PB ✅, the 200,000-employee LLM Suite ✅, the Athena ML pipelines ✅ — all bank-disclosed. The **model layer is named and the data layer is not**: the lakes, the governance estate, and the model inventory behind those numbers are §12. The [AI/LLM guides](../technology/ai_llm/) supply the mechanics; JPMC supplies the scale figures.

---

## 8. Transformation and Risk Context: The London Whale, Cloud, and Modernization

### 8.1 The London Whale: The 2012 CIO Loss

The **2012 London Whale episode** is the defining risk event of JPMC's modern era and the context for every risk-system claim in this guide ✅:

- **The loss** ✅ — in the first two quarters of **2012**, JPMorgan Chase suffered a **trading loss of approximately US$6.2 billion** in the **synthetic credit portfolio (SCP)** of its **Chief Investment Office (CIO)**, based on trades booked through its **London branch** (Sage Encyclopedia of Business Ethics and Society: "the massive $6.2 billion loss suffered by JPMorgan Chase in the first two quarters of 2012"; BBC, September 2013: "a $6bn loss incurred in 2012 as a result of the 'London Whale' trades"). The figure is reported as ~US$6.2B across sources ✅ (the exact cent is not a stable quantity across tellings; the billion-scale figure is).
- **The actors** ✅/⚠ — the trades were built around **Bruno Iksil**, a CIO trader nicknamed **"the London Whale"** by the market because of the size of his credit-derivative positions ✅ (widely reported, consistent across sources); the CIO was led by **Ina Drew**, who stepped down in May 2012 ✅ (Wikipedia; BBC). The exact position-sizing and the internal chain of decisions are the subject of the Senate record, not re-litigated here ✅/⚠.
- **The penalties** ✅ — regulators levied a combined ~**US$920 million** in penalties: the **SEC** (US$200M), the **OCC** (US$300M), the **Federal Reserve** (US$200M), and the **UK FCA** (**£137.61 million ≈ US$220 million**) (BBC, September 19, 2013: "$920m payout"; FCA press release: fined £137,610,000 for "serious failings related to its Chief Investment Office") ✅. JPMC also set aside ~US$300M more for other regulatory matters in the same settlement round ✅/⚠ (reported; the exact package composition varies by source).
- **The aftermath** ✅/⚠ — the Senate Permanent Subcommittee on Investigations issued its report in **March 2013** ✅ (widely reported); the episode drove the Volcker-Rule-era review of the CIO's hedging mandate ⚠ structural, and JPMC's subsequent risk-system investment is the backdrop of the rest of §8.
- **The systems lesson** ⚠ (this guide's read) — the public record (Senate report coverage) describes **risk-model flaws, valuation disputes, and limits that were not enforced in real time**; the *class* of systems that failed is documented in the [Enterprise Risk Management Guide](enterprise_risk_management_guide.md) — model risk, limits management, valuation control — cross-referenced, not re-derived. JPMC's post-2012 risk-estate rebuild is not publicly itemized ⚠ (§12).

### 8.2 The Technology Budget

JPMC's technology spend is public to a degree unusual in banking ✅/⚠:

- **~US$15.5 billion in 2023** ✅/⚠ (JPMC-reported figures circulated in press; the §1.8 timeline marks this ⚠).
- **~US$17 billion in 2024** ✅/⚠ (Dimon's 2024 letter describes the firm's technology spend in this range ✅/⚠ — press-consistent; the exact figure moves between letters and decks).
- **~US$18 billion projected for 2025** ✅/⚠ (press/industry trackers; consistent with the 2024 letter's trajectory).
- **What the money buys** ✅/⚠ — the CEO letters' recurring public themes: cloud migration, AI (LLM Suite), payments modernization (Kinexys), and cybersecurity; the *split* of the budget across those themes is not disclosed ⚠.

### 8.3 The Cloud Strategy

- **AWS as primary public-cloud partner** ✅ — the Athena deck (§3.1) documents JPMC's AWS migration in detail (EKS, S3, RDS, ElastiCache, KMS, CloudWatch, Direct Connect, multi-account model); the deck is the strongest primary source for JPMC's public-cloud posture.
- **Multi-cloud posture** ✅/⚠ — JPMC's public material describes multi-cloud intent (other hyperscalers appear in JPMC press ⚠); the workload split is not public (§12). The [IBM Cloud Guide](../technology/ibm_cloud_guide.md) documents the bank-cloud themes (private cloud, hybrid, mainframe integration) that apply here ⚠ cross-referenced, not re-derived.
- **Private cloud and data centers** ✅ — JPMC announced **four new private-cloud data centers** and, per CEO-letter-era disclosures, runs **32 data centers globally** (§1.6; CEO-letter verified ✅).
- **The on-premise reality** ⚠ structural — a 6,500-application estate (§6.2) with mainframe-era cores cannot be fully public-cloud; the migration state beyond the 2022 Athena snapshot is §12.

### 8.4 The "Build, Don't Buy" Doctrine

- **The evidence** ✅ — Athena (built in-house, §3.1), JPM Coin/Kinexys (built in-house, §2), LLM Suite (built in-house, §7.2): three crown-jewel platforms, all self-built, all publicly presented as such ✅.
- **The doctrine** ⚠ — JPMC has no formal public "build, don't buy" policy document; the pattern is structurally evident (§1.7) and consistent with Dimon's letters' emphasis on proprietary platforms ⚠.
- **The exceptions** ✅/⚠ — JPMC obviously buys infrastructure (AWS, §8.3) and commodity software; the *boundary* between built and bought is not public (§12).

### 8.5 The Risk Context Beyond the Whale

- **The "fortress balance sheet"** ✅ — Dimon's public strategic pillar (§1.4); the phrase and the capital posture are bank-asserted.
- **CCAR / stress testing** ✅/⚠ — as a U.S. G-SIB, JPMC runs the Federal Reserve's annual Comprehensive Capital Analysis and Review; the *regime* is certain, the internal systems ⚠ (cross-ref: [Enterprise Risk Management Guide](enterprise_risk_management_guide.md)).
- **Regulatory standing since 2012** ✅/⚠ — no London-Whale-scale enforcement action has followed; JPMC's post-2012 record on risk-data matters is materially cleaner than peers' (the 2020 Citi consent orders, §8 of the [Citibank guide](citibank_software_systems_guide.md), have no JPMC equivalent as of this pass ⚠ — absence of a public action, not proof of absence of issues).

### 8.6 The Architect's Summary of §8

The transformation story in one line: **a US$6.2B risk failure ✅ forced the risk estate to the top of the agenda; a US$17–18B-a-year technology budget ✅ paid for the platform era — Athena on AWS, Kinexys, LLM Suite — that the rest of this guide documents.** The Whale is the *only* deep risk-system disclosure JPMC has ever made public, which is why §8.1 is the anchor for every ⚠ risk-system claim in §12.

---

## 9. Singapore Angle

### 9.1 History: 1964 to Today

JPMC's Singapore presence dates to **1964** — verified at the primary source ✅:

- **Establishment** ✅ — J.P. Morgan's own Singapore pages state the office was **"Established in 1964"** and is today "a full-service, fully integrated provider of corporate and investment banking, commercial banking, asset management and private banking" (jpmorgan.com/SG/en/about-us ✅ primary source). The 2022 corporate-news release repeats it: "Established in 1964, the Singapore office is J.P. Morgan's headquarters for Southeast Asia as well as the firm's Asia Pacific hub for FX and commodities" ✅. The task brief asked to flag the year ⚠ pending verification — **the year is now primary-source verified ✅**; what remains ⚠ is the *legal form* of the 1964 establishment (branch vs representative office) and the Chase Manhattan lineage's separate Singapore history ⚠ (Chase Manhattan also operated in Singapore; the two firms' pre-2000 Singapore footprints are not consolidated in JPMC's pages).
- **The modern hub** ✅ — Singapore is JPMC's **Southeast Asia headquarters** and its **Asia-Pacific hub for FX and commodities** (JPMC corporate news, 2022) ✅; in 2022 J.P. Morgan opened a **new office in the heart of Singapore's Central Business District** ✅ (JPMC news release).
- **Scale** ✅/⚠ — JPMC's Singapore operations span the full franchise (investment banking, markets, payments, asset management, private banking) ✅; the specific headcount and unit-by-unit footprint are not pinned in this pass ⚠.

### 9.2 The Singapore Systems and Blockchain Angle

Singapore is JPMC's most visible **blockchain stage** ✅:

- **The Kinexys rebrand** ✅ — the **Onyx → Kinexys by J.P. Morgan rebrand was announced at the Singapore Fintech Festival in November 2024** by Umar Farooq (§2.3) — Singapore as the launch venue for the firm's flagship blockchain unit is itself a statement about the hub's role ✅.
- **JPM Coin / BDA in Asia** ✅/⚠ — the deposit-token and Blockchain Deposit Account products (§2.2) are marketed to the Singapore-headquartered institutional client base ⚠ (client-specific facts are not public).
- **The fintech-festival presence** ✅/⚠ — JPMC has been a regular anchor participant at the Singapore Fintech Festival (the 2024 rebrand is verified ✅; the multi-year participation record is press-consistent ⚠).

### 9.3 The Regulatory Frame

- **MAS** — the Monetary Authority of Singapore is the Singapore regulator; the full regulatory framework (licensing classes, AML/CFT notices, technology-risk guidelines, D-SIB designation) is documented in the [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md) and **cross-referenced, not re-derived**.
- **Licence class** ⚠ — JPMC operates in Singapore through branches of **JPMorgan Chase Bank, N.A.** (the U.S. national bank); the specific MAS licence class (foreign full bank vs wholesale bank) is **not pinned in this pass** ⚠ (§12) — JPMC runs no consumer-branch network in Singapore, which is structurally consistent with a wholesale-class licence ⚠, but the guide does not assert the class without a source.
- **D-SIB status** ⚠ — the MAS D-SIB list (designated 2014, seven banks) covered the big local banks plus Citi, HSBC, Standard Chartered, and Maybank (per the [Citibank guide](citibank_software_systems_guide.md) §9.3); **JPMC was not among the inaugural seven** ⚠ (negative observation from the Citi guide's record; not re-verified against MAS in this pass).
- **The AML/KYC angle** ⚠ — as a U.S. bank branch, the Singapore operations are subject to both MAS requirements and U.S. extraterritorial rules (OFAC sanctions screening, FATCA); the compliance estate is §12.

### 9.4 The Architect's Summary of §9

Singapore is JPMC's **Southeast Asia systems beachhead**: the 1964 establishment ✅, the FX-and-commodities hub role ✅, and the 2024 Kinexys rebrand on the Singapore stage ✅ are all primary-verified. What is not public is the *local* systems inventory — the branch's payments and markets estate is a regional slice of the global platforms in §2–§3, with no Singapore-specific system names disclosed ⚠ (§12).

---

## 10. Worked Example: Cymbal Bank × JPMC — Correspondent Banking and Trade Services

> **Persona note:** Cymbal Bank is the fictional/illustrative mid-size Singapore-based bank used across this repository's worked examples (it is the author's affiliation in the sibling guides). It stands in for a real mid-tier ASEAN bank. Everything in this section is an **illustrative reconstruction** built from the verified JPMC products in §2–§3 and the standard correspondent-banking mechanics in the [Payment Rails Guide](payment_rails_guide.md) — the *products* are real, the *scenario* is a model.

### 10.1 The Relationship

Cymbal Bank, a mid-size Singapore-headquartered bank, serves ASEAN corporates with U.S. trade exposure. It holds a correspondent relationship with J.P. Morgan because Cymbal needs what only a top-tier global bank provides:

- **USD clearing** — Cymbal's clients settle dollar invoices; Cymbal needs a USD clearing bank with scale in New York.
- **Cross-border reach** — Cymbal's clients pay beneficiaries in markets where Cymbal has no presence (JPMC's treasury-services network, §2.5).
- **Trade finance support** — Cymbal's clients need LCs confirmed by a name the counterparty's bank will trust.

The relationship is a **financial-institution (FI) client** relationship: Cymbal is a *client of J.P. Morgan Payments*, not a JPMC subsidiary — the mirror of the correspondent model JPMC has run since the Chase Manhattan era (§1.3).

### 10.2 The Rails: Correspondent Banking, Nostro Accounts, SWIFT

The mechanics here are standard correspondent banking — documented in full in the [Payment Rails Guide](payment_rails_guide.md) and only mapped to JPMC here:

- **Nostro** — Cymbal holds a **USD nostro account at JPMorgan Chase Bank, N.A.** (New York). From Cymbal's perspective the account is "our money at their bank"; from JPMC's it is a vostro ("your money at our bank"). The glossary defines both.
- **Access via J.P. Morgan Payments channels** ✅/⚠ — Cymbal's treasury and ops teams see the nostro balance, intraday position, and statements through J.P. Morgan's institutional client channels (§2.5 — the channel brand was **not exhaustively verified** in this pass and is treated as ⚠; the *capability* — balances, payments, statements — is standard for the product class ✅ structural).
- **Messaging** — payment instructions flow over **SWIFT**: MT103 (customer transfer) / MT202 (bank transfer), and — for the ISO 20022 era — pacs.008/pacs.009 under CBPR+; Cymbal's core banking system and JPMC's payments estate exchange these via their respective SWIFT interfaces ⚠ (JPMC's SWIFT gateway specifics are not public, §12; the message standards themselves are in the [Payment Rails Guide](payment_rails_guide.md)).
- **GPI tracking** — cross-border payments carry the SWIFT GPI tracker reference so Cymbal's ops can trace end-to-end status ⚠ (GPI mechanics: [Payment Rails Guide](payment_rails_guide.md); JPMC's GPI membership is structural, §2.5).
- **The daily cycle** — Cymbal's back office reconciles the JPMC nostro statements against its own ledgers; intraday reporting gives Cymbal the real-time position the wholesale-payments franchise advertises (§2.1) ✅/⚠.

### 10.3 KYC and Onboarding

Onboarding a correspondent is a KYC programme in its own right, and for a U.S.-chartered bank it is a **regulated** one (the USA PATRIOT Act's correspondent-account due-diligence rules apply to U.S. banks' foreign FI accounts ⚠ structural):

- **The onboarding file** — Cymbal provides: ownership structure, licences (MAS licence, §9.3), AML/CFT programme documentation, sanctions policy, and the classic **correspondent banking questionnaire**; JPMC's FI due-diligence team assesses the jurisdiction, ownership, and respondent-bank risk ⚠ (the exact JPMC questionnaire is confidential, §12; the class of information is standard across the industry).
- **Screening** — Cymbal's name, principals, and transactions are screened against sanctions lists (OFAC and others) on an ongoing basis; the screening-engine mechanics are in this repo's [FircoSoft Guide](fircosoft_guide.md) and [Enterprise Risk Management Guide](enterprise_risk_management_guide.md) ⚠ (JPMC's specific screening vendor is not public, §12).
- **Ongoing monitoring** — transaction monitoring on the nostro flows, periodic KYC refresh (the "evergreen" review), and the respondent-bank obligations Cymbal itself carries for its own downstream correspondents ⚠ (structural).

### 10.4 Trade Services

The trade angle exercises J.P. Morgan's trade estate (the class documented in the [Trade Finance Guide](trade_finance_guide.md) and [Trade Finance Systems Guide](trade_finance_systems_guide.md)):

- **The scenario** — Cymbal's client, a Singapore exporter, sells machinery to a U.S. importer. The U.S. importer's bank issues an **LC** in favour of the exporter; **Cymbal advises the LC** and, where the exporter wants the U.S. bank's risk replaced by a stronger name, arranges for the LC to be **confirmed** — with J.P. Morgan as confirming bank ⚠ (confirmation economics: the confirming bank takes the issuing bank's risk for a fee; mechanics in the [Trade Finance Guide](trade_finance_guide.md)).
- **Cymbal's own issuance** — for Cymbal's importing clients, Cymbal issues LCs and uses **J.P. Morgan's trade processing** via the FI-client trade channels ⚠ (product structure per §2.5; engine names not public).
- **Documentary flow** — documents against payment/acceptance, discrepant-document handling, and settlement through the Cymbal–JPM nostro: the message types (MT700/MT707 for LCs, MT400/MT202 for settlement) are the domain of the [Trade Finance Guide](trade_finance_guide.md) and [Trade Finance Systems Guide](trade_finance_systems_guide.md), cross-referenced, not re-derived.
- **SCF angle** ⚠ — where Cymbal's clients have receivables from U.S. buyers, the supply-chain-finance platforms documented in the [Supply Chain Finance Guide](supply_chain_finance_guide.md) are the class of system J.P. Morgan Payments runs; JPMC's specific SCF platform is not public (§12).

### 10.5 The Systems Involved (Mapped to This Guide)

| Step in the relationship | JPMC system (verified where marked) | Guide section |
|---|---|---|
| Cymbal sees nostro balances, initiates payments | J.P. Morgan Payments client channels ⚠ | §2.5 |
| USD settlement | JPMorgan Chase Bank, N.A. (NY) nostro + SWIFT/ISO 20022 | §2.5, [Payment Rails Guide](payment_rails_guide.md) |
| Instant, 24/7 programmable settlement (optional) | JPM Coin (JPMD) via Blockchain Deposit Account ✅ | §2.2, §2.6 |
| Cross-border information/validations | Liink-class network services ✅/⚠ | §2.4 |
| FX hedging of USD exposure | J.P. Morgan Markets e-trading; Kinexys on-chain FX ⚠ | §3.2, §2.3 |
| LC confirmation/advising | J.P. Morgan trade modules ⚠ | §10.4, [Trade Finance Guide](trade_finance_guide.md) |
| Sanctions screening of flows | Screening estate ⚠ | [FircoSoft Guide](fircosoft_guide.md), [Enterprise Risk Management Guide](enterprise_risk_management_guide.md) |

The architect's takeaway: **Cymbal's entire JPMC relationship runs through two verified surfaces — the J.P. Morgan Payments channels (payments/liquidity/trade) and the named platform layer (JPM Coin/Kinexys for the 24/7 tokenized leg) — over the correspondent rails documented in the Payment Rails Guide.** Everything beneath those surfaces is the ⚠ engine layer of §12. That is the honest shape of the JPMC systems map for any FI client.

### 10.6 Step-by-Step: A USD Payment from Singapore to New York

Walk a single payment end-to-end — Cymbal's client in Singapore owes US$500,000 to a U.S. supplier whose account is at a third U.S. bank. Each step is the standard correspondent pattern from the [Payment Rails Guide](payment_rails_guide.md), mapped to the verified JPMC surfaces ⚠ (structural; JPMC's engine internals are §12):

1. **Instruction** — Cymbal's client instructs Cymbal to pay US$500,000 to the supplier's account at the U.S. bank; Cymbal's treasury operator creates the payment in the J.P. Morgan Payments channel, specifying the beneficiary bank and account ⚠ (Cymbal's own core feeds the channel; the file formats are the standard corporate-banking ones).
2. **Authorization** — the maker/checker pair releases the payment; approvals happen under the four-eyes control of §13 ⚠.
3. **Message** — Cymbal's SWIFT interface sends the customer transfer instruction — MT103 in the classic era, pacs.008 under CBPR+ — addressed to JPMorgan Chase Bank, N.A. New York (Cymbal's USD correspondent) ⚠ (message standards: [Payment Rails Guide](payment_rails_guide.md)).
4. **Screening** — JPMC's sanctions/AML screening runs on the payment at intake and again against updates; a false positive stops the flow for investigation (§10.3) ⚠.
5. **Nostro debit** — JPMC debits Cymbal's USD nostro (§10.2); the cover leg to the beneficiary's bank runs as MT202 / pacs.009 bank-to-bank transfer ⚠.
6. **The New York leg** — the US$500,000 moves across the U.S. dollar clearing systems — CHIPS (netting) or Fedwire (RTGS) — to the beneficiary's bank, which credits the supplier's account ⚠ (JPMC's specific clearing memberships are not published; §12).
7. **Tracking** — the payment carries a GPI tracker reference; Cymbal's ops see end-to-end status in the channel and can prove value to the client ⚠ (GPI mechanics: [Payment Rails Guide](payment_rails_guide.md)).
8. **Reconciliation** — at end of day, the nostro statement shows the debit, fees, and the resulting balance; Cymbal's back office matches it against its own ledger (§10.7).

The whole flow touches exactly one verified JPMC product family — the J.P. Morgan Payments channels (initiation, tracking) — plus the JPMorgan Chase Bank, N.A. nostro (settlement), with the payment engine, screening engine, and clearing memberships in the ⚠ layer between them.

### 10.7 Step-by-Step: The Nostro Reconciliation Cycle

The daily discipline that keeps the relationship honest ⚠ (structural — the class of process is universal; JPMC's statement formats and tools are not public):

1. **Intraday** — the channel shows the live nostro position; Cymbal's liquidity desk manages against payment windows and cut-offs (§10.6 step 6) ⚠.
2. **End of day** — JPMC posts the nostro statement (debits, credits, fees, value dates); Cymbal receives it through the channel or SWIFT ⚠.
3. **Matching** — Cymbal's reconciliation matches each statement line to an instructed payment: amount, value date, beneficiary, and charges convention (OUR/SHA/BEN) ⚠ (charges conventions: [Payment Rails Guide](payment_rails_guide.md)).
4. **Breaks** — unmatched items become breaks: missing credits (the cover never arrived), value-date differences (interest impact), and fee mismatches; each is investigated, typically by SWIFT investigation messages ⚠.
5. **Aging and claims** — unresolved breaks age; Cymbal raises claims against JPMC (and JPMC against Cymbal) under the correspondent agreement; aged breaks feed the monthly nostro-position review ⚠.

The reconciliation cycle is the *control* that makes the §10.2 nostro relationship safe to run at volume — and it is exactly the process that a deficient data estate (§7.1) makes unreliable at scale ⚠.

### 10.8 Step-by-Step: KYC Due Diligence on Cymbal

Before Cymbal gets the payment-channel credentials of §10.6, it passes through the correspondent-banking KYC programme (§10.3) ⚠ (structural — the standard industry sequence, per the [FircoSoft Guide](fircosoft_guide.md) and [Enterprise Risk Management Guide](enterprise_risk_management_guide.md)):

1. **The file** — Cymbal supplies charter documents, its MAS licence (§9.3), ownership structure, and management/UBO information.
2. **The questionnaire** — Cymbal completes a correspondent banking questionnaire covering its AML/CFT programme, sanctions policy, correspondent network, and downstream respondent banks ⚠ (JPMC's questionnaire is confidential; the class is industry-standard).
3. **Risk rating** — JPMC's FI due-diligence team assesses jurisdiction, ownership, respondent-bank, and product risk; the result sets the EDD level and monitoring intensity ⚠.
4. **Screening** — Cymbal's name, principals, and UBOs screen against OFAC and other sanctions lists; hits trigger enhanced review ⚠.
5. **Approval and onboarding** — the relationship is approved at the delegated authority level, the nostro agreement is signed, and the channel entitlements are provisioned (§10.2) ⚠.
6. **Evergreen** — periodic KYC refresh re-runs steps 1–4 on a cycle; material changes (ownership, jurisdiction) trigger off-cycle review; ongoing transaction monitoring runs on the nostro flows (§10.6 step 4) ⚠.

The KYC programme is where a U.S.-chartered bank's correspondent obligations bite hardest: correspondent due-diligence records must be complete, accurate, and retrievable — the properties that failed so publicly at Citi in 2020 (§8.5 cross-ref) and that JPMC has avoided since the Whale-era enforcement (§8.1) ✅/⚠ (the *absence* of a JPMC consent order is verified; the *quality* of the underlying estate is not public).

### 10.9 Step-by-Step: A Documentary Credit with J.P. Morgan Confirmation

The trade-services flow (§10.4) in full ⚠ (structural — message families and mechanics in the [Trade Finance Guide](trade_finance_guide.md); JPMC's engine internals not public):

1. **The contract** — Cymbal's exporter client signs a sale contract with the U.S. importer: payment by irrevocable documentary credit.
2. **Issue** — the importer's bank issues the LC in favour of the exporter and sends the MT700 through SWIFT, with Cymbal as advising bank.
3. **Advise** — Cymbal authenticates the MT700 and advises the LC to the exporter via its trade channels ⚠.
4. **Confirmation** — the exporter wants the issuing bank's risk replaced by a stronger name; Cymbal requests J.P. Morgan to add its confirmation. J.P. Morgan confirms — taking the issuing bank's risk for a confirmation fee — and the exporter ships against the confirmed LC ⚠ (confirmation economics: [Trade Finance Guide](trade_finance_guide.md)).
5. **Presentation** — the exporter presents documents (invoice, bill of lading, packing list, certificate) to Cymbal, which checks them against the LC terms (the UCP 600 document check) ⚠.
6. **Discrepancies** — a late shipment makes the documents discrepant; Cymbal either waives, amends (MT707), or presents on approval; clean documents proceed ⚠.
7. **Settlement** — on complying presentation, the importer's bank pays; the funds flow through the correspondent chain — MT400/MT202-style settlement through the Cymbal–JPM nostro (§10.2) — and the exporter receives payment ⚠.

For the systems map, the LC flow is the trade mirror of §10.6: verified surfaces at the ends (the J.P. Morgan trade channels, the nostro), message standards in the middle, and the confirmation/risk engine unnamed in the ⚠ layer (§12).

### 10.10 The Operating-Day Narrative

Put the steps together as a day at Cymbal's back office ⚠ (structural): the morning starts with the nostro position and the day's payment queue in the J.P. Morgan Payments channel; the USD payment window closes against New York cut-offs; the treasury desk reviews whether any flows should move to the JPM Coin (JPMD) 24/7 leg for after-hours settlement (§2.2, §10.5) ✅/⚠ (the product exists ✅; Cymbal's use is illustrative); the trade team tracks the confirmed LC and the exporter's presentation; by end of day the nostro statement is matched, the breaks are logged, and the quarter's KYC refresh sits in the queue. Every one of those screens is a verified JPMC surface or a standard industry process; every engine behind them is the §12 unknown. That is the honest operating picture of correspondent banking on the JPMC estate — and, by extension, on any top-tier global bank's ⚠.

---

## 11. Claims Status and Verification Notes

**Verification convention: ✅ = verified in this research pass (primary or secondary sources); ⚠ = flagged (inferred, approximate, single-source, or structural); ❌ = disputed (the public record contradicts the claim); "not public" = JPMC does not disclose and no external source exists either way.**

### 11.1 The Claims-Status Table

| Claim | Source | Status |
|---|---|---|
| 1799: Manhattan Company chartered (water company with banking clause); Bank of The Manhattan Company opens — second commercial bank in NYC; Hamilton and Burr among founders | JPMC history page; official history monograph | ✅ verified |
| 1871: Drexel, Morgan & Co. founded by J.P. Morgan and Anthony Drexel; House of Morgan era begins | JPMC official history monograph; Wikipedia | ✅ verified |
| 1877: Chase National Bank established by John Thompson; named for Salmon P. Chase; world's largest bank by 1930 ($2.7B assets) | JPMC history monograph | ✅ verified |
| 1895: Drexel, Morgan reorganizes as J.P. Morgan & Co. | Wikipedia; monograph narrative | ✅ verified |
| 1955: Chase National + Bank of The Manhattan Company → The Chase Manhattan Bank; octagon logo | JPMC history monograph/timeline | ✅ verified |
| 1996: Chemical acquires Chase Manhattan; keeps the Chase name; 270 Park Avenue retained | Wikipedia (citing NYT, Sep 1996) | ✅ verified |
| 2000: J.P. Morgan + Chase Manhattan merger; agreement Sept 12, 2000; completion December 2000 | JPMC SEC 8-K exhibit; Britannica; Wikipedia | ✅ verified |
| 2004: Bank One acquisition (July); Dimon arrives; CEO from December 2005 | Wikipedia; JPMC 2024 CEO letter | ✅ verified |
| March 2008: Bear Stearns rescue — $2/share initially, $10/share after renegotiation; $30B NY Fed financing; completed May 30, 2008 | SEC press releases; NYT DealBook; Wikipedia | ✅ verified |
| September 2008: WaMu purchase for US$1.836B after FDIC seizure; largest U.S. bank failure; Providian card arm folded into Chase | Wikipedia; FDIC receivership record | ✅ verified (⚠ Providian integration) |
| May 2023: First Republic acquisition — ~$173B loans, $30B securities, $92B deposits; $10.6B to FDIC | JPMC press release May 1, 2023; Investopedia; CNBC | ✅ verified |
| 2024: revenue US$180.6B, net income US$58.5B, ROTCE 20%; >$10T daily movement in 120+ currencies, 160+ countries; >$35T assets safeguarded | Dimon's 2024 letter (Apr 2025) | ✅ verified |
| Technology scale: 450+ PB data, US$9T payments daily, US$33T custody, >80% of Fortune 500, 59M digital customers, 6,500+ apps, 250K virtual workspaces, 50K+ technologists | JPMC deck, AWS Financial Services Cloud Symposium 2022 | ✅ verified (JPMC-presented) |
| Wholesale payments move >US$6T daily across 100+ countries | CNBC (Oct 2020, citing Takis Georgakopoulos) | ✅ verified |
| Chase serves ~half of U.S. households (66M) | JPMC AWS deck | ✅ verified (JPMC-presented) |
| Four reportable segments (CCB, CIB, CB, AWM) through 2023; C&IB combination announced January 2024 | JPMC CEO letters (2023, C&IB letter) | ✅ verified |
| JPM Coin announced February 2019; first U.S. bank-backed digital token | CNBC (Feb 13, 2019) | ✅ verified |
| JPM Coin commercial use milestone October 2020 | CNBC (Oct 27, 2020) | ✅ verified (first pilot date ⚠) |
| JPM Coin (JPMD) = deposit token on Ethereum L2 (Base), USD, BDA on/off-ramps, programmable payments | jpmorgan.com/kinexys/jpm-coin; JPMC payments newsroom | ✅ verified (product positioning) |
| Onyx formed October 2020; Umar Farooq CEO; >100 staff; took over Liink/JPM Coin | CNBC (Oct 27, 2020); Banking Dive | ✅ verified |
| Onyx → Kinexys by J.P. Morgan rebrand announced at Singapore Fintech Festival, November 2024; on-chain FX Q1 2025 | JPMC insights; Glenbrook; fintechdaily | ✅ verified (rebrand); ⚠ (FX launch press-sourced) |
| Kinexys: 10+ years, >US$3T volume since inception, >US$7B average daily volume; product catalogue incl. BDA, On-Chain FX, TCN, Tokenized MMFs | jpmorgan.com/kinexys/index | ✅ verified (JPMC-presented) |
| BMW Group first-ever programmable on-chain FX payments with Kinexys | JPMC insights/payments newsroom | ✅ verified (client story) |
| IIN pilot 2017; rebranded Liink October 27, 2020; >400 banks at rebrand | CNBC; BusinessWire (Oct 27, 2020); Banking Dive | ✅ verified (year); ⚠ (exact October 2017 date) |
| Athena = in-house pricing/risk/trade platform; "one of the world's largest Python repositories": 50M lines, 4,000+ Python developers, 20K production changes/week, 5B calcs/day, 10K batch jobs/day, 50K-core grid, 100 markets | JPMC deck (Bauerschmidt), AWS Symposium 2022 | ✅ verified (JPMC-presented) |
| Athena on AWS: EKS, S3, RDS, ElastiCache, KMS, CloudWatch, Direct Connect, multi-account; Athena ML hedging framework | JPMC deck, AWS Symposium 2022; Conf42 talk (2025-era) | ✅ verified (deck); ⚠ (talk) |
| Risk as a Service = commercialized J.P. Morgan analytics | jpmorgan.com/markets/risk-as-a-service | ✅ verified (product page) |
| Chase Sapphire Reserve launched August 23, 2016 | §1.8 timeline (prior verified pass) | ✅ verified |
| Chase Amazon Prime Rewards Visa announced January 11, 2017 | §1.8 timeline (prior verified pass) | ✅ verified |
| Co-brand portfolio: United, Marriott, Amazon, Southwest, IHG, Hyatt, Disney, British Airways | Partnership press; §1.6/§4.2 | ✅ partnerships; ⚠ portfolio boundaries |
| AWM "more than doubled client assets to $5 trillion" | JPMC Investor Day material | ✅ verified (JPMC-presented) |
| LLM Suite used by 200,000+ employees; American Banker 2025 Innovation of the Year Grand Prize | jpmorganchase.com technology blog; PR Newswire | ✅ verified |
| 2012 London Whale: ~US$6.2B loss in CIO synthetic credit portfolio, booked via London branch | Sage Encyclopedia; BBC (Sep 2013); Wikipedia | ✅ verified |
| Whale penalties ~US$920M: SEC $200M, OCC $300M, Fed $200M, FCA £137.61M (~$220M) | BBC (Sep 19, 2013); FCA press release | ✅ verified |
| Ina Drew (CIO) stepped down May 2012; Bruno Iksil "the London Whale" | Wikipedia; BBC; consistent press | ✅ verified (secondary, consistent) |
| Senate PSI report March 2013 | Widely reported | ✅ verified (secondary) |
| Technology spend: ~$15.5B 2023, ~$17B 2024, ~$18B projected 2025 | Press; CEO letters; industry trackers | ✅/⚠ (reported; exact figures move) |
| Four new private-cloud data centers; 32 data centers globally | CEO-letter-era disclosures (§1.6) | ✅ verified |
| JPMC Singapore "Established in 1964"; SE Asia HQ; APAC FX & commodities hub; new CBD office 2022 | jpmorgan.com/SG/en/about-us; JPMC corporate news 2022 | ✅ verified (primary) |
| JPMC not among inaugural 2014 MAS D-SIB cohort (seven banks) | [Citibank guide](citibank_software_systems_guide.md) §9.3 record | ⚠ flagged (negative observation, not re-verified vs MAS) |
| Core banking system identity / payments hub / card platform / wealth platform / data-platform identities | No public disclosure found | ⚠ not public (see §12) |
| JPMorgan Chase "largest U.S. bank card issuer by purchase volume" | Industry rankings (Nilson class) | ✅/⚠ (common knowledge; no figure re-verified this pass) |

### 11.2 The Honesty Note: Methodology

1. **Verified anchors are strong for JPMC** — the history (§1.3) is primary-source verified via JPMC's own history pages and official monograph; the product layer (§2–§3, §7) is verified via JPMC product pages, the AWS-symposium deck, and press releases; the London Whale record (§8.1) is verified via regulator press releases (FCA) and established press (BBC). This is the *most* verifiable estate in this series on the product layer — and one of the *least* verifiable on the engine layer.
2. **The inference zone is the engine layer** — core identity, hubs, card platforms, wealth platforms, data/AI vendors: the *class* of system is certain for a bank of JPMC's size, the *identity* is not (§12).
3. **One deliberate upgrade** — the Singapore establishment year (1964) was flagged ⚠ in the task brief pending verification; it is now **primary-source verified ✅** against JPMC's own pages (jpmorgan.com/SG/en/about-us). What remains ⚠ is the legal form of the 1964 establishment and the MAS licence class.
4. **Deliberately not verified** (out of scope): per-market regulatory filings, vendor contracts and pricing, internal roadmaps, exact operating-market counts, JPMC Singapore headcount, the full co-brand contract terms, and the current-state figures that move quarterly (revenue, AUM, LLM Suite employee counts).

---

## 12. What Could Not Be Verified

The following are the materially non-public specifics of the JPMC estate. Each is flagged ⚠ and should be treated as unknown, not as "likely X":

- **The core banking system identity** ⚠ — vendor vs in-house, the application names, and the ledger engines for deposits/loans/cards. JPMC has never publicly named its consumer or wholesale core; the [Core Banking Systems Guide](core_banking_systems_guide.md) documents the vendor classes it could belong to, but no JPMC-specific primary source exists.
- **The mainframe estate** ⚠ — hardware scale (IBM Z class), the applications on it, and the decommissioning timeline beyond the 6,500-application count (§6.2).
- **The payments hub / payments engine internals** ⚠ — the orchestration layer behind the J.P. Morgan Payments channels, SWIFT connectivity, and ISO 20022 implementation specifics; the client channel brands were not exhaustively verified in this pass (§2.5).
- **JPM Coin / Kinexys internals** ⚠ — chain infrastructure (nodes, key management), the settlement cutover to the core ledger, liquidity management, client roster and volumes, and the regulatory approvals relied on (§2.2, §2.6).
- **Liink's standalone footprint** ⚠ — node counts, message formats, and the relationship between Liink and the Kinexys payments stack (§2.4).
- **Athena's internal architecture** ⚠ — beyond the conference material: grid scheduler internals, the pricing-library structure, position-keeping design, and the migration state beyond the 2022 snapshot (§3.1).
- **The card-management platform** ⚠ — authorization, clearing, servicing, collections, and loyalty accounting behind the Chase card book (§4.4).
- **The wealth platforms** ⚠ — the private-bank workstation, the digital-advice engine, and the asset-management stack behind the three wealth brands (§5.3).
- **The data platform and AI platform vendors** ⚠ — the warehouse/lake estate behind the 450 PB and the model vendors behind LLM Suite (§7).
- **The risk and limits systems** ⚠ — the post-2012 risk-estate rebuild, the counterparty-credit engines, and the CCAR infrastructure (§8.1, §8.5).
- **The screening estate** ⚠ — JPMC's sanctions/AML screening vendors and the FI due-diligence questionnaires (§10.3).
- **USD clearing memberships** ⚠ — JPMC's direct CHIPS/Fedwire memberships and the settlement accounts behind its USD clearing are not published; the Cymbal nostro sits on JPMC's New York balance sheet (§10.2) but the clearing plumbing is unnamed.
- **JPMC Singapore specifics** ⚠ — the MAS licence class, the local systems inventory, headcount, and the pre-2000 Chase/J.P. Morgan Singapore footprints (§9.1, §9.3).
- **Market-data vendor contracts** ⚠ — the vendors feeding the markets estate are not named in JPMC public material (§3.2).
- **The cloud workload split** ⚠ — which of the 6,500+ applications are on AWS, which remain on-premise, and the multi-cloud balance (§8.3).
- **Co-brand contract terms** ⚠ — renewal dates, economics, and portfolio boundaries for each partnership (§4.2).
- **The technology-budget split** ⚠ — the allocation of the ~$17B across cloud, AI, payments, and run-the-bank (§8.2).
- **The "build, don't buy" doctrine** ⚠ — the pattern is structurally evident; no formal public policy document exists (§8.4).

---

## 13. Glossary

| Term | Definition |
|---|---|
| **JPMC / JPMorganChase** | JPMorgan Chase & Co. (NYSE: JPM) — the holding company created by the December 2000 J.P. Morgan–Chase Manhattan merger |
| **The Manhattan Company** | The 1799 water-company charter (Hamilton and Burr among founders) whose banking clause produced the Bank of The Manhattan Company — the lineage's oldest root |
| **Chase National Bank** | The 1877 bank (John Thompson; named for Salmon P. Chase) that was the world's largest by 1930; merged into The Chase Manhattan Bank in 1955 |
| **House of Morgan** | The J.P. Morgan merchant-banking lineage from Drexel, Morgan & Co. (1871); the source of the "house of systems" reading in this guide's close |
| **The Chase Manhattan Bank** | The 1955–2000 bank formed by the Chase National–Manhattan Company merger; the Chase half of the 2000 merger |
| **Bank One** | The 2004 acquisition that brought Jamie Dimon into JPMC (CEO from December 2005) |
| **Bear Stearns / WaMu / First Republic** | The 2008 rescues (Bear, March 2008; WaMu, September 2008) and the 2023 FDIC-assisted First Republic purchase |
| **CCB / CIB / CB / AWM** | Consumer & Community Banking; Corporate & Investment Bank; Commercial Banking; Asset & Wealth Management — the four long-standing reportable segments (CB folded into C&IB in 2024) |
| **C&IB** | Commercial & Investment Bank — the wholesale division formed by the January 2024 CIB + CB combination |
| **CIO (2012 context)** | Chief Investment Office — the JPMC unit that ran the synthetic credit portfolio that produced the 2012 London Whale loss |
| **London Whale** | The 2012 episode: ~US$6.2B trading loss in the CIO's synthetic credit portfolio, booked through London; named for trader Bruno Iksil |
| **Synthetic credit portfolio (SCP)** | The CIO's credit-derivative hedging book where the 2012 losses occurred |
| **JPM Coin (JPMD)** | JPMC's USD deposit token on Ethereum L2 (Base) — payment, settlement, and reconciliation in one on-chain action; ticker JPMD for institutional clients |
| **Blockchain Deposit Account (BDA)** | The JPMC on/off-ramp framework that converts USD deposits into JPM Coin and back |
| **Onyx** | The JPMC blockchain business unit formed October 2020 (Umar Farooq, CEO) — renamed Kinexys in November 2024 |
| **Kinexys by J.P. Morgan** | The 2024-rebranded blockchain unit: JPM Coin, BDA, On-Chain FX, Tokenized Collateral Network, Digital Debt Service, Confirm, Route Logic, and the Digital Assets Platform |
| **IIN / Liink** | Interbank Information Network (2017 pilot) → Liink (October 2020) — the interbank information-sharing network for payment validations |
| **Athena** | JPMC's in-house markets platform for trade management, risk, pricing, and data science — one of the world's largest Python codebases (50M+ lines, JPMC-presented) |
| **Athena ML** | The ML training/inference framework on Athena for hedging and risk management (AWS-symposium documented) |
| **Risk as a Service** | JPMC's commercialized quantitative analytics and market-data product (§3.1) |
| **LLM Suite** | JPMC's proprietary generative-AI platform, used by 200,000+ employees; American Banker 2025 Innovation of the Year |
| **Sapphire / Freedom / Ultimate Rewards** | Chase's premium card line (Reserve launched Aug 23, 2016), the cash-back card lines, and the shared points ecosystem |
| **Co-brand card** | A card issued in partnership with a brand (airline/hotel/retailer): partner brings customers, issuer brings balance sheet and processing; economics documented in the [Ancillary Revenue Products Guide](../management/ancillary_revenue_products_guide.md) |
| **AWM / Private Bank / JPMAM** | Asset & Wealth Management; J.P. Morgan Private Bank (UHNW); J.P. Morgan Asset Management — the wealth brands of §5 |
| **Cymbal Bank** | The fictional/illustrative mid-size Singapore bank persona used across this repository's worked examples — not a real bank |
| **Nostro / Vostro** | "Our money at your bank" (nostro, from Cymbal's view of its JPMC New York account) / "your money at our bank" (vostro, from JPMC's view) |
| **Correspondent banking** | The relationship in which one bank (Cymbal) uses another bank's (JPMC's) balance sheet and network to clear, settle, and pay in markets where it has no presence |
| **SWIFT** | The cooperative messaging network (MT/ISO 20022) that carries cross-border payment and trade messages between banks |
| **MT103 / MT202** | SWIFT payment messages: MT103 customer credit transfer; MT202 bank-to-bank transfer (the correspondent cover leg) |
| **pacs.008 / pacs.009** | The ISO 20022 payment messages: customer credit transfer / bank-to-bank transfer under CBPR+ |
| **CBPR+** | The SWIFT-managed cross-border payments ISO 20022 migration programme |
| **GPI / UETR** | SWIFT Global Payments Innovation (tracked cross-border payments) and its Unique End-to-end Transaction Reference |
| **CHIPS / Fedwire** | The two U.S. wholesale dollar rails: CHIPS (netting) and Fedwire (RTGS) |
| **RTGS** | Real-Time Gross Settlement — the central-bank settlement rail class |
| **LC (documentary credit)** | Letter of credit — the bank-issued undertaking that anchors trade finance; MT700/MT707 message families |
| **UCP 600** | The ICC's Uniform Customs and Practice for Documentary Credits — the rules of the LC document check |
| **Maker/checker** | The four-eyes control: one operator initiates, a second authorizes |
| **OFAC / FATCA** | The U.S. sanctions authority (and its list) and the Foreign Account Tax Compliance Act — U.S. extraterritorial rules that bind JPMC's Singapore branch (§9.3) |
| **AML/CFT / KYC / EDD** | Anti-Money Laundering / Countering the Financing of Terrorism; Know Your Customer; Enhanced Due Diligence — the compliance disciplines of §10.3 |
| **Nostro reconciliation** | The daily matching of the nostro statement against the bank's own ledger (§10.7) |
| **SR 11-7 / BCBS 239** | The Federal Reserve model-risk guidance and the Basel risk-data aggregation principles — the regulatory classes behind §7.4 and §8.5 |
| **CCAR** | Comprehensive Capital Analysis and Review — the Federal Reserve's annual stress-testing regime for U.S. G-SIBs |
| **G-SIB** | Globally Systemically Important Bank — JPMC's regulatory class |
| **D-SIB** | Domestically Systemically Important Bank — the MAS designation that JPMC is not among the inaugural seven of (§9.3) |
| **MAS** | Monetary Authority of Singapore — the Singapore regulator (framework in the [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md)) |
| **Mainframe** | The legacy large-computer class (IBM Z-era) on which JPMC's core estate is structurally inferred to run (⚠) |
| **Strangler-fig modernization** | The incremental replacement pattern in which new systems grow around and gradually replace the legacy core (§6.4) |
| **FIX** | The Financial Information eXchange protocol for electronic trading (see [FIX Protocol Guide](fix_protocol_guide.md)) |
| **DDS** | The data-distribution-system class for low-latency market data (see [DDS Guide](../technology/dds_guide.md)) |
| **Quorum** | JPMC's original enterprise blockchain (an Ethereum fork) on which JPM Coin was first announced; superseded by the public-L2 deposit-token design |

---

## 14. References and Further Reading

**Primary/company:**
- JPMC history pages and official history monograph — "The History of JPMorgan Chase & Co. — 200 Years of Leadership in Banking" — jpmorganchase.com (the 1799/1857/1871/1877/1955 lineage anchors of §1.3)
- JPMorganChase CEO letters to shareholders (2023, April 2024; 2024, April 2025) — jpmorganchase.com — scale figures (§1.4), the C&IB combination and division letters (§1.5), technology themes (§8)
- JPMorganChase press release, May 1, 2023 (First Republic acquisition) — jpmorganchase.com/news
- JPMC deck, AWS Financial Services Cloud Symposium 2022 (Paul Bauerschmidt, MD) — the Athena and firmwide technology-scale figures (§1.4, §3.1, §6.2, §7.1)
- Kinexys product pages — jpmorgan.com/kinexys/index; JPM Coin — jpmorgan.com/kinexys/jpm-coin; Kinexys insights "Introducing Kinexys"; JPMC payments newsroom (JPMD ticker, BMW case) — jpmorgan.com
- Risk as a Service — jpmorgan.com/markets/risk-as-a-service
- J.P. Morgan Singapore — jpmorgan.com/SG/en/about-us ("Established in 1964"); JPMC corporate news, 2022 (new Singapore office; SE Asia HQ; APAC FX & commodities hub)
- LLM Suite — jpmorganchase.com/about/technology/blog/llmsuite-ab-award; PR Newswire (American Banker 2025 Innovation of the Year; 200,000+ employees)
- SEC filings: 8-K exhibit (merger agreement, September 12, 2000); SEC press releases on the Bear Stearns merger (March 16/24, 2008)
- FCA press release — JPMorgan Chase Bank N.A. fined £137,610,000 (London Whale) — fca.org.uk

**Press and secondary:**
- CNBC: JPM Coin announcement (February 13, 2019); JPM Coin commercial use and Onyx launch (October 27, 2020); wholesale payments >US$6T daily (October 2020)
- BusinessWire: Liink rebrand (October 27, 2020); Banking Dive (Onyx formation, Liink >400 banks)
- BBC News: London Whale penalties ~US$920M (September 19, 2013)
- Sage Encyclopedia of Business Ethics and Society: "London Whale, JPMorgan's" (US$6.2B loss in first two quarters of 2012)
- Wikipedia: 2012 JPMorgan Chase trading loss; JPMorgan Chase; Jamie Dimon; the 1996 Chemical–Chase merger (citing NYT); 2000 merger (December 1, 2000); Bear Stearns/WaMu records
- Glenbrook, fintechdaily (Kinexys rebrand and on-chain FX coverage); The Digital Banker, Larridin AI tracker (LLM Suite scale)
- Investopedia, CNBC (First Republic); LA Times (Fed approval of the 2000 merger)

**Sibling guides in this repository (cross-references used throughout):**
- [Citibank Software Systems Guide](citibank_software_systems_guide.md) (the structural template for this series), [DBS Software Systems Guide](dbs_software_systems_guide.md), [Standard Chartered Guide](standard_chartered_guide.md)
- [Payment Rails Guide](payment_rails_guide.md) — SWIFT, ISO 20022, GPI, correspondent banking, nostro mechanics
- [Trade Finance Guide](trade_finance_guide.md), [Trade Finance Systems Guide](trade_finance_systems_guide.md), [Supply Chain Finance Guide](supply_chain_finance_guide.md) — trade and SCF
- [Core Banking Systems Guide](core_banking_systems_guide.md), [Posting Engine Core Banking Guide](posting_engine_core_banking_guide.md), [Interest Calculation Engine Guide](interest_calculation_engine_guide.md), [US Bank Core Systems Guide](us_bank_core_systems_guide.md) — core mechanics
- [FIX Protocol Guide](fix_protocol_guide.md), [DDS Guide](../technology/dds_guide.md) — markets connectivity and data distribution
- [Private Banking Guide](private_banking_guide.md), [Wealth Management Guide](wealth_management_guide.md), [Universal Banking Model Guide](universal_banking_model_guide.md) — wealth and bank-model context
- [Enterprise Risk Management Guide](enterprise_risk_management_guide.md), [FircoSoft Guide](fircosoft_guide.md) — risk, AML/KYC, screening
- [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md) — the Singapore regulatory framework
- [Ancillary Revenue Products Guide](../management/ancillary_revenue_products_guide.md) — co-brand card economics
- [IBM Cloud Guide](../technology/ibm_cloud_guide.md) — bank-cloud themes; [AI/LLM guides](../technology/ai_llm/) (e.g., [Enterprise AI Platforms Guide](../technology/ai_llm/enterprise_ai_platforms_guide.md)) — AI platform mechanics

**Verification note:** deep links to jpmorgan.com pages change as the firm restructures its site (the Onyx → Kinexys renames moved several URLs); the product names and figures quoted here are the stable identifiers to search for if a link moves. Figures marked "JPMC-presented" are the bank's own public numbers, quoted as such and not independently audited in this pass.

---

*End of guide. JPMorgan Chase is the bank that turned the 1799 Manhattan Company's water-charter loophole into the largest U.S. financial institution — 225 years of mergers and rescues run through platforms the bank itself built: Athena, JPM Coin, Kinexys, LLM Suite. The product names are public; the engines are not; the honest flags in §11–§12 mark exactly where public evidence ends and inference begins. What remains is not a bank with systems but a bank that is, in every sense the record allows,*

the house of systems.
