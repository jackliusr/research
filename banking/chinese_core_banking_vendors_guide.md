# The Chinese Core-Banking Vendors: A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Core Banking / Banking Architecture — the domestic vendors behind Chinese core-banking systems: Nantian (南天信息), Yusys (宇信科技), Shenzhou Information / DHC (神州信息), GFS (高伟达), Hundsun (恒生电子); the vendor landscape and tiers, the market and its consolidation, company profiles (founding, listing, products, clients), the vendor comparison, a 信创-era vendor-selection worked example for a mid-size Chinese bank, and the glossary
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** August 2026

---

## Table of Contents

1. [The Vendor Landscape](#1-the-vendor-landscape)
   - 1.1 [The Chinese Core Vendors: Tiers](#11-the-chinese-core-vendors-tiers)
   - 1.2 [The Market: Size, Structure, and Consolidation](#12-the-market-size-structure-and-consolidation)
   - 1.3 [The Landscape Table](#13-the-landscape-table)
   - 1.4 [How the Money Flows: The Product+Project Economics](#14-how-the-money-flows-the-productproject-economics)
   - 1.5 [The Vendor Timeline: 1995–2026](#15-the-vendor-timeline-19952026)
2. [The Nantian Profile: 南天信息](#2-the-nantian-profile-南天信息)
   - 2.1 [The Company](#21-the-company)
   - 2.2 [The Products](#22-the-products)
   - 2.3 [The Clients](#23-the-clients)
   - 2.4 [The Nantian Table](#24-the-nantian-table)
   - 2.5 [Nantian's Strategic Position: The SOE-SI Hybrid](#25-nantians-strategic-position-the-soe-si-hybrid)
3. [The Yusys Profile: 宇信科技](#3-the-yusys-profile-宇信科技)
   - 3.1 [The Company](#31-the-company)
   - 3.2 [The Products](#32-the-products)
   - 3.3 [The Clients](#33-the-clients)
   - 3.4 [The Yusys Table](#34-the-yusys-table)
   - 3.5 [Yusys in the Value Chain](#35-yusys-in-the-value-chain)
4. [The Shenzhou Profile: 神州信息](#4-the-shenzhou-profile-神州信息)
   - 4.1 [The Company: DHC Lineage](#41-the-company-dhc-lineage)
   - 4.2 [The Products: The Sm@rt Cores](#42-the-products-the-smrt-cores)
   - 4.3 [The Clients](#43-the-clients)
   - 4.4 [The Shenzhou Table](#44-the-shenzhou-table)
   - 4.5 [The DHC Group Structure](#45-the-dhc-group-structure)
5. [The GFS Profile: 高伟达](#5-the-gfs-profile-高伟达)
   - 5.1 [The Company](#51-the-company)
   - 5.2 [The Products and Positioning](#52-the-products-and-positioning)
   - 5.3 [The GFS Table](#53-the-gfs-table)
   - 5.4 [The Services Vendors' Playbook: GFS, Nantian and Teamax in the 信创 Era](#54-the-services-vendors-playbook-gfs-nantian-and-teamax-in-the-信创-era)
6. [The Hundsun Profile: 恒生电子](#6-the-hundsun-profile-恒生电子)
   - 6.1 [The Company: Founded 1995](#61-the-company-founded-1995)
   - 6.2 [The Products: The Wealth Stack](#62-the-products-the-wealth-stack)
   - 6.3 [Hundsun and the Core: Why Not a Core Vendor](#63-hundsun-and-the-core-why-not-a-core-vendor)
   - 6.4 [The Hundsun Table](#64-the-hundsun-table)
   - 6.5 [Hundsun and the Core Programme: The Integration Interface](#65-hundsun-and-the-core-programme-the-integration-interface)
7. [The Comparison](#7-the-comparison)
   - 7.1 [The Vendor Comparison Table](#71-the-vendor-comparison-table)
   - 7.2 [Reading the Comparison](#72-reading-the-comparison)
   - 7.3 [The Deal-Type Map: Who Wins Which Engagement](#73-the-deal-type-map-who-wins-which-engagement)
8. [The Worked Example: A Vendor Selection for a Mid-Size Chinese Bank](#8-the-worked-example-a-vendor-selection-for-a-mid-size-chinese-bank)
   - 8.1 [The Scenario](#81-the-scenario)
   - 8.2 [The 信创-Era Selection Criteria](#82-the-信创-era-selection-criteria)
   - 8.3 [The Shortlist and Scoring](#83-the-shortlist-and-scoring)
   - 8.4 [The Lessons](#84-the-lessons)
   - 8.5 [The RFP Annex: What the Bank Should Actually Ask](#85-the-rfp-annex-what-the-bank-should-actually-ask)
   - 8.6 [The Phasing Plan](#86-the-phasing-plan)
   - 8.7 [After Selection: The Contract and the First 100 Days](#87-after-selection-the-contract-and-the-first-100-days)
9. [The Summary: The Vendors Behind the Chinese Cores](#9-the-summary-the-vendors-behind-the-chinese-cores)
10. [The Glossary](#10-the-glossary)
11. [The Claims-Status Table](#11-the-claims-status-table)
12. [The References](#12-the-references)

---

### How to Read This Guide

This is the dedicated **vendor-profile deep-dive** for Chinese core-banking technology. It expands [Section 3 of chinese_bank_core_systems_guide.md](chinese_bank_core_systems_guide.md) — the Chinese-core umbrella, which covers the sector, the 数据大集中 history, the self-development (自主研发) phenomenon, 去IOE, digital banks, the 信创 stack, and China-vs-West implications — from a vendor *overview* into full company profiles: founding and listing facts, product lines, client bases, and how to compare the vendors against each other.

Reading paths:

- **New to the domain** — start with [core_banking_systems_guide.md](core_banking_systems_guide.md) (the global core-banking umbrella) and [chinese_bank_core_systems_guide.md](chinese_bank_core_systems_guide.md) Sections 1–2, then come back here for the vendor detail.
- **Vendor/product research** — Sections 2–6 here, then the [comparison](#7-the-comparison) and the [worked example](#8-the-worked-example-a-vendor-selection-for-a-mid-size-chinese-bank).
- **Procurement / selection** — [Section 8](#8-the-worked-example-a-vendor-selection-for-a-mid-size-chinese-bank) is a complete 信创-era selection exercise; the platform context is in [../technology/oracle_database_guide.md](../technology/oracle_database_guide.md) and [../technology/ibm_as400_guide.md](../technology/ibm_as400_guide.md) (the AS/400 heritage that Chinese distributed cores replaced).
- **Singapore/Asia angle** — the bank-systems series ([dbs_bank_guide.md](dbs_bank_guide.md), [ocbc_software_systems_guide.md](ocbc_software_systems_guide.md)) and [insurance_open_source_commercial_guide.md](insurance_open_source_commercial_guide.md) (the same vendor-landscape pattern applied to insurance) are the closest analogues for how a foreign architect should read a domestic-vendor market.

A note on verification: this guide was researched with web searches in August 2026. Facts **verified** from retrievable sources are marked as such; items that are widely reported but could not be independently verified in this session are **flagged**; vendor marketing claims (market-share rankings in particular) are labelled as vendor claims. Every China-specific claim is summarised in the [claims-status table](#11-the-claims-status-table) at the end. **Nothing in this guide is fabricated; where a fact (client name, revenue figure, founding date) could not be verified, the guide says so explicitly rather than guessing.**

**Scope and sources.** The five deep-dived vendors (Nantian, Yusys, Shenzhou, GFS, Hundsun) were selected by the reader; Sunline and GienTech appear as shortlist context and are profiled in the umbrella. Primary sources are the vendors' own sites, Chinese financial press (including coverage of stock-exchange filings and annual reports), and the umbrella guide's verified material. The research backend in this session was search-only (page extraction degraded), so some primary pages (annual-report PDFs, official history pages) could not be read in full; where that mattered, the claim is flagged. Vendor market-share rankings are quoted only as vendor claims because the underlying IDC reports were not accessible. No client claim in this guide is fabricated: client names appear only where verified or explicitly marked as reported/flagged, and the worked example's bank (Huahe Bank) is explicitly fictional.

---

## 1. The Vendor Landscape

### 1.1 The Chinese Core Vendors: Tiers

The domestic core-banking vendor market is **consolidated around a handful of public companies**, all of which also act as system integrators (SI). The Chinese model is "product + project" (产品+项目): the vendor both licenses its core software and delivers it bank-by-bank in long on-site engagements. The umbrella guide ([chinese_bank_core_systems_guide.md](chinese_bank_core_systems_guide.md) §3.1) states this as the defining structural fact of the market; this guide goes one level deeper into the companies.

A useful way to tier the vendors (an author's tiering, consistent with the umbrella's §3.1–3.8 and with the verified company data below — treat the tiers as an analytical convenience, not an official classification):

- **Tier 1 — the core-banking product houses.** Vendors whose packaged core is a first-tier product in the Chinese mid-market and who compete head-on for core rebuilds: **Sunline / Changliang (长亮科技, 300348)** and **Shenzhou Information / DCITS (神州信息, 000555)** with its **Sm@rt** core line, plus the state-affiliated full-stack house **GienTech (中电金信)**. These are the vendors a mid-tier bank's CIO shortlists for a core rebuild (umbrella §3.2–3.4).
- **Tier 2 — the broad banking-IT vendors.** Large, listed, bank-focused IT companies whose portfolios span core-adjacent systems, channels, data, financial cloud and SI services: **Yusys (宇信科技, 300674)**, **Nantian (南天信息, 000948)**, **GFS (高伟达, 300465)**, **Teamax (天阳科技, 300872)** and the regional SI layer (umbrella §3.5–3.6). They implement, localise and extend cores as much as they sell their own.
- **Tier 3 — the channel and securities-niche vendors.** Vendors that appear in "Chinese banking IT" lists but are **not core-banking vendors** in the account-processing sense: **Hundsun (恒生电子, 600570)** (securities/funds/wealth), **Kelan (科蓝软件, 300663)** (internet/mobile-banking channels), **ApexSoft (顶点软件, 603383)** (securities) (umbrella §3.8).
- **The platform giants.** **Huawei** (GaussDB, Kunpeng/Taishan, 华为云) and **ZTE** (GoldenDB) sell the database/hardware layer under the cores — structural players in every 信创-era core deal even though they do not sell packaged cores (umbrella §3.7). **OceanBase** (Ant Group) and **TiDB** (PingCAP) also sit in this layer (umbrella §7.1).

This guide profiles five vendors in depth — **Nantian, Yusys, Shenzhou Information, GFS and Hundsun** — the five the reader asked for. Sunline and GienTech, the two other names that dominate any shortlist conversation, are already profiled in the umbrella (§3.2, §3.4) and appear here in the comparison and the worked example so the picture is complete.

### 1.2 The Market: Size, Structure, and Consolidation

**The overall market.** The Chinese banking-IT solutions market (银行业IT解决方案市场) is the world's largest single-country market for bank technology spending, tracked most prominently by **IDC** in its annual *China Banking IT Solutions Market* reports (market-share data flows from IDC's vendor surveys; the umbrella and this guide rely on vendor-cited versions of that data — see the flags below). The market is segmented into sub-markets: core systems (核心系统), channels, payments, credit/risk, data, cloud and operations.

**The market-share ranking is contested — and that itself is the story.** Two of the vendors in this guide make *conflicting* headline claims:

- **Yusys** states on its own site that it has ranked **No. 1 in IDC's China banking IT solutions market for 7 consecutive years** (公司已连续7年在IDC中国银行业IT解决方案市场排名第一 — vendor claim, from yusys.com.cn).
- **GienTech** publishes an IDC-backed claim of **"nine consecutive championships" (九连冠)** — "in 2025 GienTech firmly ranks No. 1 in the bank-IT industry, the most important full-stack IT service provider in China's banking system" (GienTech press release quoting an IDC research manager, 2025 — vendor claim).

Both cannot be "No. 1 overall" in the same IDC edition. The reconciliation is that IDC publishes multiple report editions, segments and years (overall solutions market vs. specific sub-markets such as core systems or financial cloud; different vintages; vendor vs. vendor+service definitions), and vendors pick the framing that flatters them. **Flag: treat all market-share rankings as vendor marketing unless the underlying IDC report itself is in hand.** What is not contested: the top cluster is a small set of vendors — Yusys, GienTech, Shenzhou Information/DCITS, Sunline, plus (per IDC sub-market reports) Nantian in specific niches such as self-service/equipment-linked services.

**The consolidation-flagged dynamics.** Several forces are consolidating the market (this paragraph is an assessment built on the verified facts in Sections 2–6 and the umbrella, not a single citable source):

1. **信创 (Xinchuang) shrinks the competitive set.** The domestic-substitution programme (umbrella §7.5) effectively excludes foreign packaged cores and foreign database/hardware stacks from new bank builds. The addressable core market is therefore carved up among domestic vendors — a guaranteed market, but one where the *state* increasingly sets the rules (certified product catalogues, 产品名录, and compatibility testing favour vendors with state backing and certified stacks).
2. **State-affiliated grouping.** The strongest structural consolidation is into state-owned technology groups: **GienTech under China Electronics Corporation (CEC)** (umbrella §3.4), **Nantian under Yunnan SASAC via Yunnan Industrial Investment Holding Group** (verified, Section 2.1). State ownership matters in 信创 procurement — a vendor's controlling shareholder is a selection criterion in its own right.
3. **Revenue pressure and capital events.** The vendors' financials are volatile: Yusys reported a **>20% revenue decline in 2024** (reported by Chinese financial press, 2025; see Section 3.1) followed by a rebound narrative in its 2025 annual report (published March 2026), and in 2025 moved to raise capital via a **Hong Kong IPO for an A+H dual listing** (reported). GFS swung between losses and small profits in recent years (reported). Listed-vendor financials are the pressure gauge for the whole market: bank IT budgets in 2024–2025 were tight, and vendors traded growth for profitability (Yusys was publicly criticised in 2025 press for cutting R&D and headcount to protect profit — reported).
4. **The long tail is the volume.** The ~125 city commercial banks and ~3,000 rural institutions (umbrella §1.1) buy from this vendor set; the "banking-as-a-service" shared-platform model (umbrella §3.9) concentrates even the long tail into the same few vendors' platforms.
5. **International expansion as an escape valve.** Yusys (Southeast Asia push, per its own materials), Sunline (international arm) and GienTech (ORIGIEN launched in Hong Kong, umbrella §3.4) are exporting Chinese distributed-core architecture to Asia — the consolidation at home is partially funded by growth abroad.

### 1.3 The Landscape Table

| Vendor | Chinese | Ticker / listing | Tier (this guide's analysis) | Core-banking focus |
|---|---|---|---|---|
| **Sunline / Changliang** | 长亮科技 | 300348 SZSE | 1 — core product house | Parameterised core + GaussDB-based distributed core; the "core product" specialist (umbrella §3.2; founded 2002) |
| **Shenzhou Information (DCITS / DHC lineage)** | 神州信息 | 000555 SZSE | 1 — core product house + SI | **Sm@rtBanking / Sm@rtOneBank** integrated & distributed banking platform (Section 4) |
| **GienTech** | 中电金信 | CEC-affiliated (ex-Pactera; not independently listed) | 1 — state-affiliated full stack | Full-stack fintech: core, channels, data, cloud; **ORIGIEN** platform (umbrella §3.4) |
| **Yusys** | 宇信科技 | 300674 SZSE (ChiNext, Nov 2018) | 2 — broad banking IT | Distributed next-gen core, channels, financial cloud, credit/risk, e-CNY (Section 3) |
| **Nantian** | 南天信息 | 000948 SZSE (listed 1999) | 2 — SI + services + equipment | Financial digital solutions, system integration, IT services; self-service/ATM heritage; core-adjacent (Section 2) |
| **GFS** | 高伟达 | 300465 SZSE (ChiNext) | 2 — banking IT services | Bank IT systems, integration, ops services, cloud services for banks (Section 5) |
| **Teamax** | 天阳科技 | 300872 SZSE | 2 — banking IT services | Credit-card, risk, core implementation for mid-tier banks (umbrella §3.6) |
| **Hundsun** | 恒生电子 | 600570 SHSE (listed 2003) | 3 — securities/wealth niche | Securities, funds, wealth-management, treasury and channel systems for banks — **not the retail core** (Section 6) |
| **Kelan / ApexSoft** | 科蓝软件 / 顶点软件 | 300663 / 603383 | 3 — channel/securities niche | Internet/mobile-banking channels; securities trading (umbrella §3.8) |
| **Huawei / ZTE** | 华为 / 中兴通讯 | — (platform giants) | Platform layer | GaussDB + Kunpeng/Taishan; GoldenDB — the database/hardware under the cores (umbrella §3.7) |

### 1.4 How the Money Flows: The Product+Project Economics

Understanding *how the vendors get paid* explains their behaviour better than any marketing claim:

- **The procurement cycle.** Chinese bank-IT deals run through formal tender processes: 招标 (RFP/announcement) → 入围 (shortlisting) → 评标 (scoring) → 中标 (award). For a core rebuild the RFP is a substantial document, and since the 信创 era it carries **catalogue and certification annexes** (product-catalogue entries, compatibility test reports) that are as important as the functional requirements. The vendor's bid is typically priced as **product + project**: a software-license component plus implementation/delivery services, with multi-year operations & maintenance (运维) fees after go-live.
- **The revenue stack.** A successful Chinese core vendor earns from four layers: (1) **license/solution sales** (the product), (2) **implementation services** (the project — where the SI culture comes from), (3) **operations and maintenance contracts** (the annuity — often the most profitable layer), and (4) for the long tail, **banking-as-a-service subscriptions** — shared/multi-tenant platforms where dozens of small rural banks run one vendor-operated core (umbrella §3.9). This is why the vendors describe themselves as "financial technology facilitators" providing "software products, solutions and implementation, operation, maintenance and testing, system integration" (Yusys's own self-description, verified) — the whole stack is the product.
- **Who gets paid in a core programme.** The app vendor takes license+implementation; the **database/hardware layer** (Huawei GaussDB + Kunpeng/Taishan, ZTE GoldenDB, OceanBase, or domestic DBs like DM/TiDB on Hygon/Phytium servers — umbrella §7.1–7.3) is a separate, often larger, procurement line; Tier-2 SIs (Nantian, GFS, Teamax) take subcontracted delivery; and the bank's own IT team absorbs operations. A foreign architect should model a Chinese core rebuild as **three parallel procurement streams** (application, database+infrastructure, integration services), not one.
- **Revenue-scale context (flagged).** The listed vendors' revenues are reported in the single-digit RMB-billions range (Yusys's 2024 decline of >20% — reported — implies a multi-billion-RMB base; GFS's filings show small absolute numbers and profit swings — Section 5.1). Exact figures belong in the annual reports, which for any procurement due diligence are the only acceptable source; press coverage of vendor financials in 2024–2025 was notably critical (Yusys's profitability-first pivot, Section 3.1), and a vendor's revenue trajectory is a legitimate scored criterion (Section 8.2, criterion 7).

### 1.5 The Vendor Timeline: 1995–2026

The vendor events in this guide, in order (each with its verification status; full detail in the [claims-status table](#11-the-claims-status-table)):

| Year | Event | Status |
|---|---|---|
| 1971 | Heritage factory behind Nantian established (Yunnan) | Reported |
| 1995 | **Hundsun founded** in Hangzhou | **Verified** |
| 1998 | Nantian corporate establishment | Reported (flagged) |
| 1999 | Nantian listed on SZSE (000948); **Yusys founded** (founder 洪卫东) | **Verified** |
| 2002 | Sm@rtBanking documented as covering "nearly all banking business areas" (DHC/Digital China era) | **Verified** (Sina Finance) |
| 2003 | **Hundsun listed** on SHSE (600570) | **Verified** |
| 2007 | Yusys listed on NASDAQ (YTEC) | **Verified** |
| ~2010s / 2023 | Yusys NASDAQ delisting — sources disagree on the year | Flagged |
| ~2013–2014 | Digital China information-services business injected into the 000555 shell (Shenzhou/DCITS listing) | Reported (flagged) |
| 2014 | Ant Group takes control of Hundsun | Reported (flagged) |
| Nov 2018 | **Yusys ChiNext IPO (300674)** | **Verified** |
| Jan 2021 | GienTech brand launched under CEC (ex-Pactera) | Verified (umbrella §3.4) |
| 2024 | Yusys revenue declines >20%; press criticism of R&D cuts | Reported |
| 2025 | Yusys files for HK IPO (A+H plan); GienTech publishes "nine consecutive championships" IDC claim; GFS H1 turnaround reported | Reported / vendor claim |
| Mar 2026 | Yusys 2025 annual report: "rebound" narrative, revenue-mix disclosure | Reported |

The timeline's takeaway: the vendor set is young by Western-core standards (Hundsun 1995, the rest 1998–2002 era), has churned through three capital-markets regimes (state restructuring, NASDAQ, ChiNext, now H-shares), and the 2020s is the decade in which 信创 turned these companies from SI houses into mandated national champions — with everything that implies for their financials, their client claims and their roadmap discipline.

---

## 2. The Nantian Profile: 南天信息

### 2.1 The Company

**Name and listing.** Yunnan Nantian Electronic Information Industry Co., Ltd. (云南南天电子信息产业股份有限公司, "Nantian Information", 南天信息), **SZSE 000948**. Multiple sources verify the A-share listing **on the Shenzhen Stock Exchange in 1999** (company-profile sources; Investing.com profile). The **founding date is the flagged item**: a company analysis source (hotelaah.com, June 2025) states the joint-stock company was **established in 1998**, headquartered in **Kunming, Yunnan**; another company profile (aifun.cc) traces the lineage further back to a Yunnan state factory ("Yunnan Wuyiwei Factory") **established in 1971**, and to more than 30 years in banking automation. The reconciliation: the *enterprise* (joint-stock company) dates to 1998, the *heritage* is older — and one widely-circulated "founded 1998" figure is exactly the kind of number that needs a footnote in vendor decks. **Flag: treat "founded 1998" as the corporate-establishment date, with a longer state-factory lineage; the exact corporate birth year should be confirmed against the company's own prospectus/history page before it is quoted in procurement documents.**

**Ownership.** Controlled by **Yunnan Industrial Investment Holding Group (云南省工业投资控股集团)**, with the **Yunnan Provincial State-owned Assets Supervision and Administration Commission (云南省国资委)** as the ultimate controller (per the hotelaah analysis and the company's own member-company pages). This is a genuinely unusual ownership profile among Chinese banking-IT vendors: Nantian is a **provincial state-owned enterprise (SOE)** rather than a founder-controlled tech company — an asset in Yunnan-linked and 信创-linked procurement, and a reason its strategic centre of gravity is Yunnan-first, national-second.

**Business.** Nantian's verified business description (Investing.com profile, company sources): "mainly engaged in information business, system integration and application software business" for finance, and its information products include **ATMs, printers and POS products** used in banking, insurance, telecom, power and broadcasting industries. Historically, Nantian was the **banking self-service/automation equipment leader** ("a leading IT company in banking automation business of China for more than 30 years" — supplier profile) — i.e., the hardware-and-services house of Chinese banking IT, not a packaged-core product house. In the 2020s it positions itself as a financial digitalisation solutions and services provider (金融行业数字化综合解决方案和服务 — Xueqiu profile, verified), including **Information Technology Application Innovation (信创)** solutions.

### 2.2 The Products

Nantian's product set, from verified sources and with the core-banking relevance assessed:

- **Core Banking Service Grid System (核心银行服务网格系统).** The company's own English-language site lists this under its financial-technology/信创 product line (en.nantian.com.cn, member-company page). This is the closest Nantian comes to a "core" product in the account-processing sense — a service-grid / distributed-services layer for banking rather than the full ledger-and-accounts core of a Sunline or a Sm@rt. **Flag: the product's exact scope (full core vs. core-adjacent services layer) could not be verified in this session; treat it as a distributed banking-services platform product.**
- **Self-service and branch automation hardware + software.** ATMs, printers, POS terminals, VTM-type devices and the software that drives them (verified via Investing.com product description and the banking-automation supplier profile). This is Nantian's heritage franchise and still its differentiator: it understands the physical-branch and self-service channel layer better than any pure software vendor.
- **System integration and IT services.** The classic SI portfolio: integration, application software development, and IT operations/maintenance services (IT运维) for financial institutions (verified business description).
- **Financial digitalisation solutions** — the modern wrapper for the above: digital-bank-enablement projects, data platforms, and 信创-compliant modernisation delivery (company positioning; the 2025 Xueqiu summary "financial industry digital comprehensive solutions and services" is verified).

**Where Nantian sits on cores.** Nantian is best understood as a **Tier-2 banking-IT vendor**: an SI and services house with a hardware heritage and a core-adjacent (service-grid) product, rather than a core product house. In practice Nantian appears in core programmes as **implementation partner, channel/self-service integrator and regional service arm** — including, plausibly, for the tier of Yunnan and southwest-China banks where its SOE ownership and local delivery footprint are decisive. **Flag: no verified, named Nantian core-banking installation (bank + core product) was retrievable in this session; do not cite one without checking the company's client case list.**

### 2.3 The Clients

**Flag: client claims are unverified this session.** No named Nantian banking client (bank name + system) could be retrieved from the search results available in this session. What is *reported* (industry profile materials, not verified): a long customer base across banking, insurance, telecom and power built over 30+ years of banking-automation and SI work, with particular strength in **Yunnan and southwest China** (the SOE home market). The prudent reading for a vendor comparison: treat Nantian's client base as real but *unverifiable at name level* in this session — verify against the company's annual report client case list before relying on any specific bank reference.

### 2.4 The Nantian Table

| Aspect | Description (verified unless flagged) |
|---|---|
| **Chinese name** | 云南南天电子信息产业股份有限公司 (南天信息) |
| **English name** | Yunnan Nantian Electronic Information Industry Co., Ltd. (Nantian Information) |
| **Listing** | SZSE 000948; listed on the Shenzhen Stock Exchange **1999** (verified) |
| **Founded** | Joint-stock company established **1998** (reported, single analysis source — **flagged**); heritage traced to a Yunnan state factory established **1971** (reported) |
| **HQ / ownership** | Kunming, Yunnan; controlled by Yunnan Industrial Investment Holding Group; ultimate controller Yunnan SASAC (reported/company sources — **flagged** at the SASAC level) |
| **Tier** | Tier 2 — SI + services + hardware heritage; core-adjacent, not a core product house (this guide's analysis) |
| **Core products** | Core Banking Service Grid System (company English site — scope **flagged**); financial digitalisation solutions; 信创 solutions |
| **Heritage products** | ATMs, printers, POS terminals and self-service/branch automation software (verified) |
| **Core-banking role** | Implementation/SI partner, channel and self-service integrator, regional service arm; regional strength in southwest China (assessment) |
| **Clients** | Named clients **not verified this session** — verify via annual-report case lists (flagged) |
| **Key strength** | SOE ownership + local delivery footprint + hardware heritage |
| **Key watch-item** | Smallest core-product footprint of the five profiled vendors; revenue dependence on SI/services margins |

### 2.5 Nantian's Strategic Position: The SOE-SI Hybrid

Where Nantian fits, in plain terms:

- **The SOE advantage.** As a provincial-state-owned company (Yunnan Industrial Investment Holding Group / Yunnan SASAC — reported), Nantian is a *policy-comfortable* counterparty: state-linked banks, 信创 review panels and provincial regulators see a vendor owned by the state as lower-risk than a founder-led tech company. For Yunnan and southwest-China institutions this is a decisive procurement factor — and it is why Nantian's strategic centre of gravity is regional rather than national.
- **The hardware heritage.** Nantian's 30+ years in banking automation (ATMs, printers, POS, self-service) (verified product descriptions) give it a franchise the software vendors lack: the **physical-channel and self-service layer**. In the 信创 era this layer itself must go domestic (ATM/VTM software and hardware substitution is a real workstream), which plays directly to Nantian.
- **The core position — honest assessment.** Nantian is *not* a packaged-core product house for the mid-bank tier (no verified named core installation; the Core Banking Service Grid System's scope is flagged in Section 2.2). Its realistic role in core programmes: **implementation partner, channel/self-service integrator, and regional delivery arm** — including as a Tier-2 SI augmenting a Tier-1 winner's bench (the product+project model, Section 1.4). Banks that buy this do so for local delivery, state-linked comfort and channel expertise, not for the core ledger itself.
- **Watch-items for due diligence.** Confirm the corporate founding year against the company's own history page (the 1998 figure is single-source — Section 2.1); verify any claimed client names against the annual report's case list (Section 2.3); and scope-check the "Core Banking Service Grid System" before assuming it competes with Sm@rtOneBank or Sunline's core.

---

## 3. The Yusys Profile: 宇信科技

### 3.1 The Company

**Name and listing.** Beijing Yusys Technology Group Co., Ltd. (北京宇信科技集团股份有限公司, "Yusys", 宇信科技), **SZSE 300674**, listed on the Shenzhen Stock Exchange's **ChiNext board in November 2018** (verified: multiple 2025 press articles — cnfol, investing.com, MSN — confirm the November 2018 ChiNext listing).

**The founding and the listing history — the flagged item.** The company's history traces to **1999**, when founder **洪卫东 (Hong Weidong)** established the predecessor business (Beijing Yusys Hongtai per Chinese financial press; the group's own materials describe 20+ years of financial-core construction experience, consistent with a 1999 origin) — **founded 1999: verified** (multiple sources). The listing history, however, needs care:

- **2007:** listed on **NASDAQ** under ticker **YTEC** (verified via fx168/gelonghui reprints of the 2025 prospectus coverage).
- **Delisting via privatisation:** sources **disagree on the date** — the 2025 press coverage of its Hong Kong IPO prospectus says the NASDAQ listing was privatised/delisted in **2023**, while the commonly cited market record puts the privatisation in the **2010s (circa 2015)**. **Flag: the NASDAQ delisting year is contested between sources; do not quote a year without checking the prospectus.**
- **2018:** A-share ChiNext IPO (verified).
- **2025:** filed for a **Hong Kong IPO** to pursue an **A+H dual listing** (reported, multiple financial-press sources, December 2025 second submission) — the "listed-flagged" item: Yusys is simultaneously an established A-share company and an aspiring H-share issuer.

**Financials and press profile (flagged).** Yusys's 2024 results were weak — Chinese financial press reported a **>20% revenue decline in 2024** and criticised the company for cutting R&D and headcount to defend profit while the controlling shareholder reduced holdings and took dividends (MSN/zh-cn business coverage, 2025 — reported, critical press). Its **2025 annual report (published March 2026)** is presented as a recovery year ("AI-driven fintech innovation, globalisation strategy and overall performance rebound" — minichart.com.sg summary, March 2026), with the revenue mix: **large state-owned banks and major city commercial banks 57.39% of main-business income, smaller banks 29.63%, non-bank/other 12.98%** (reported, from the annual-report summary). The company also holds a **20% stake in Jinshang Consumer Finance (晋商消费金融)** (verified via 21st Century Business Herald reporting). Treat all of this as press-reported financial colour, not audited figures.

**Market position.** Yusys markets itself as the **No. 1 player in IDC's China banking IT solutions market for seven consecutive years** (vendor claim, yusys.com.cn) — the claim that collides with GienTech's "nine consecutive championships" (Section 1.2). Both are vendor marketing on different IDC editions/segments; **flag both**.

### 3.2 The Products

Yusys's product portfolio, from verified vendor materials and the umbrella:

- **Next-Gen Core Business System (新一代核心业务系统).** Per the company's own English site (yusystech.com, verified): "purpose-built for financial institutions seeking to replace closed, costly legacy foreign systems (IBM, Oracle FLEXCUBE, TCS, SAP)… built on a distributed microservices architecture — enabling faster iteration, business continuity, and high availability." This is the flagship: a **distributed, microservices core explicitly positioned as the 信创-era replacement for foreign cores** — the same positioning as Sunline's GaussDB-based core and DCITS's distributed Sm@rt line.
- **Parameterised/rule-based configuration.** Products are configured through parameter and rule frameworks (e.g., designing new loan products and tailoring workflows within parameterisation frameworks) — the umbrella's verified characterisation, and the same "products-not-code" philosophy as Sunline's parameterised core.
- **Channel systems.** Internet banking and mobile banking channel products (verified via the umbrella §3.5 and the company's solution catalogue).
- **Financial cloud (金融云).** Cloud products and services for financial institutions — one of Yusys's two flagships per CB Insights' categorisation (verified secondary).
- **Credit, risk and digital-currency systems.** Credit/loan origination and risk-control systems, plus **digital currency (e-CNY) application solutions** — the 2025 annual-report summary lists core systems, credit, risk control and digital currency as Yusys's leadership areas (reported).
- **AI and overseas expansion.** 2025–2026 messaging: AI-driven upgrades and a globalisation strategy (annual-report summary, reported); the vendor claims 20+ years of financial core-system construction experience and production-grade trust with hundreds of financial clients (vendor-press piece, asiaict.com — vendor claim).

### 3.3 The Clients

**Flagged.** Yusys's client base is verified only at the *aggregate* level:

- **"100+ financial clients"** and **"hundreds of financial institutions"** — vendor claims (yusys.com.cn; asiaict vendor-press).
- **Revenue mix by client type (2025 annual report summary, reported):** large SOE banks + major city commercial banks ≈ 57% of revenue; smaller banks ≈ 30%; non-bank/other ≈ 13% — i.e., Yusys is *top-heavy*: its economics depend on the large-bank and city-bank segment, which aligns with its positioning as the "banking IT leader" and with its presence on joint-stock and city-bank projects (umbrella §3.5).
- **No named, verified bank-client installation** (bank + Yusys product) was retrievable in this session. The company's own customer stories should be checked before any specific bank is named in procurement context.

### 3.4 The Yusys Table

| Aspect | Description (verified unless flagged) |
|---|---|
| **Chinese name** | 北京宇信科技集团股份有限公司 (宇信科技) |
| **English name** | Yusys Technologies Co., Ltd. |
| **Listing** | SZSE **300674** (ChiNext), IPO **November 2018** (verified); earlier NASDAQ listing 2007 (ticker YTEC, verified); H-share IPO filed 2025 for A+H (reported) |
| **Founded** | **1999** by founder 洪卫东 (verified); predecessor business 宇信鸿泰 (reported) |
| **NASDAQ delisting year** | **Flagged — sources disagree (2023 per 2025 press; circa 2015 per common market record)** |
| **Tier** | Tier 2 — broad banking-IT vendor with a flagship distributed core (this guide's analysis) |
| **Core products** | Next-Gen Core Business System (distributed microservices; positioned to replace IBM/Oracle FLEXCUBE/TCS/SAP) (vendor site, verified); financial cloud; channels; credit/risk; e-CNY solutions (reported) |
| **Market-share claim** | "No. 1 in IDC China banking IT solutions market, 7 consecutive years" — **vendor claim, conflicts with GienTech's claim** (Section 1.2) |
| **Clients** | 100+ financial clients (vendor claim); large SOE banks + city commercial banks ≈ 57% of revenue (2025 annual report summary, reported); named installs **not verified** |
| **Financial watch-items** | 2024 revenue declined >20% (reported); 2025 HK IPO attempt; R&D/headcount cuts criticised in 2025 press (reported) |
| **Key strength** | Distributed-core credibility + channel/cloud breadth + top-heavy large-bank client mix |
| **Key watch-item** | Revenue volatility; contested market-rank claims; verifying client references |

### 3.5 Yusys in the Value Chain

- **The breadth play.** Yusys is the most *portfolio-diverse* of the five profiled vendors: distributed core, channels, financial cloud, credit/risk, and e-CNY solutions (Section 3.2). Its pitch to a bank is "one vendor for the whole digital stack" — which is exactly why it scores well on ecosystem breadth in the worked example (Section 8.3) and why it is often the *second* name on a shortlist after the pure product house.
- **The top-heavy client mix.** The 2025 annual-report summary's revenue split (large SOE banks + major city commercial banks ≈ 57%, reported) means Yusys's fortunes track the *big-bank* and *top-city-bank* budgets — a strength (premium references) and a vulnerability (concentration; the small-bank long tail is only ~30% of its book). It also explains the "banking IT leader" market positioning.
- **The 2024–2026 strategy pivot (reported).** Press coverage describes a deliberate shift from growth to profitability — cutting R&D and headcount, tightening cash, and pursuing an H-share listing to fund AI and overseas ambitions (Sections 3.1). For a procurement officer this is a double-edged story: a leaner, cash-disciplined vendor vs. a vendor whose investment capacity and key-person bench are under pressure. The annual reports, not the press, settle the question.
- **Governance notes (reported/verified-secondary).** The founder 洪卫东's share sales were noted critically in 2025 press; the related-party history with 易诚互动 (a former affiliate pursuing its own listing) drew exchange scrutiny (OFweek, 2024); and the 20% stake in Jinshang Consumer Finance (verified secondary, Section 3.1) is a diversification that regulators occasionally question. None of this is disqualifying — but it belongs in the vendor-stability criterion (Section 8.2, criterion 7).
- **Where Yusys wins deals.** Banks wanting core + channels + cloud from one vendor; e-CNY and digital-currency initiatives; and 信创-era rebuilds where the incumbent foreign core (IBM/Oracle FLEXCUBE-era) is being displaced — the "replace the foreign core" positioning is literally its product pitch (Section 3.2).

---

## 4. The Shenzhou Profile: 神州信息

### 4.1 The Company: DHC Lineage

**Name and listing.** Digital China Information Service Group Co., Ltd. (**DCITS**, 神州数码信息服务集团股份有限公司, "Shenzhou Information", 神州信息), **SZSE 000555**. The umbrella guide (verified via en.dcits.com) records: **~20,700 employees, 300+ financial-software products**, and the full banking suite anchored on the **Sm@rt** product family.

**The DHC lineage — verified at the group level.** DHC is the standard English abbreviation for **Digital China (神州数码)**, the legendary Chinese IT distribution and services group that grew out of the Legend (联想) ecosystem. Shenzhou Information is the group's **listed information-services arm**; its banking-software lineage runs through **神州数码融信 (Digital China Financial Information)**, the financial-IT unit that historically built banking core systems for Chinese banks (umbrella §3.3). The **Sm@rt product naming — with the "@" as the brand's signature — is itself verified from 2002-era coverage** (Sina Finance, April 2002: "Sm＠rtBanking覆盖了现代银行几乎所有业务领域，其中包括网络银行等电子商务解决方案" — Digital China's Sm@rtBanking covering nearly all modern banking business areas, including e-banking/electronic-commerce solutions, and "representing the latest application-technology progress of China's banking informatisation"). In short: **DHC/Digital China built the Sm@rt cores; Shenzhou Information (DCITS) is the listed vehicle that inherited them.**

**Listing-history flag.** Ticker 000555's predecessor shell (太光电信, Taiguang Telecom) and the early-2010s restructuring through which Digital China's information-services business was injected into the listed vehicle are **reported but were not independently verified this session** — the standard account is a backdoor/asset-injection listing in the 2013–2014 window. Do not quote the injection year without checking the company's own history disclosure.

### 4.2 The Products: The Sm@rt Cores

- **Sm@rtBanking.** The legacy core line — the integrated banking solution that covered "nearly all banking business areas" in the data-centralisation era (2002-era source, verified). In its generation it was one of the two or three mainstream domestic cores for mid-size banks, alongside Sunline's.
- **Sm@rtOneBank.** The modern flagship: "one-stop banking overall solution" spanning **core banking, channels, payments and data**, with **distributed-core capabilities** for the modernization wave (umbrella §3.3, verified via en.dcits.com). This is the product a mid-tier bank's CIO would evaluate against Sunline's distributed core and GienTech's ORIGIEN in a 信创-era rebuild RFP.
- **Modelb@nk / future-bank architecture (reported).** Third-party materials reference a "**Modelb@nk 5.0 未来银行架构**" with "distributed-core domestic-stack adaptation (分布式核心系统国产适配)" in the 神州信息 orbit (chinayundu automation-product page, a low-quality aggregator). **Flag: treat Modelb@nk as a reported next-generation distributed-core line in the Shenzhou family; verify product name and version against the company's own site before quoting.**
- **Data, agricultural-finance and rural-tier systems.** DCITS has deep data/BI platforms and agricultural-finance systems, historically strong with the rural tier (umbrella §3.3) — the agricultural/rural franchise is a genuine differentiator given the ~3,000-institution rural tier (umbrella §1.1).

### 4.3 The Clients

**Flagged.** The umbrella's assessment — Shenzhou is "historically strong with mid-size city commercial banks and the rural tier; one of the two or three vendors (with Sunline) that a mid-tier Chinese bank's CIO would shortlist for a core rebuild" — is verified as an *industry assessment*, but **no named client installation was verified in this session**. One secondary source (a 2010s-era LinkedIn industry article on Chinese core banking) reports "Digital China recently succeeded with a core banking installation in **Bank of Chengdu (成都银行)** and targets banks around the same size in southern China" — **reported/secondary, not verified; flag it**. The company's own case-study pages should be the source of truth for any procurement reference.

### 4.4 The Shenzhou Table

| Aspect | Description (verified unless flagged) |
|---|---|
| **Chinese name** | 神州数码信息服务集团股份有限公司 (神州信息) |
| **English name** | Digital China Information Service Group Co., Ltd. (DCITS / Shenzhou Information) |
| **Listing** | SZSE **000555**; predecessor-shell restructuring reported ~2013–2014 (**flagged**) |
| **Group lineage** | **DHC = Digital China (神州数码)**; banking software via 神州数码融信 (Digital China Financial Information) (verified at group level) |
| **Tier** | Tier 1 — core product house + SI (this guide's analysis) |
| **Core products** | **Sm@rtBanking** (legacy core, 2002-era verified); **Sm@rtOneBank** (integrated + distributed core, channels, payments, data — verified); Modelb@nk future-bank distributed core (**flagged**) |
| **Scale** | ~20,700 employees; 300+ financial-software products (verified via en.dcits.com) |
| **Clients** | Mid-size city commercial banks + rural tier (industry assessment); Bank of Chengdu reported (secondary, **flagged**); named installs not verified this session |
| **Key strength** | Sm@rt brand heritage, full-suite breadth, rural/agri-finance franchise, data platforms |
| **Key watch-item** | Verifying the current product roadmap (Sm@rtOneBank vs. Modelb@nk) and client references |

### 4.5 The DHC Group Structure

The DHC ecosystem is broader than the listed 000555 shell, and a buyer should understand the map:

- **Digital China (神州数码) — the group.** The Legend/联想-derived IT distribution and services group. Its listed vehicles and units include the **information-services arm (神州信息 / DCITS, 000555)** — the subject of this profile — and the **group/distribution business (神州数码集团, commonly cited as SHSE 000034)** — **flagged: the 000034 ticker and the exact current group structure were not verified this session; check the group's own investor pages.** The banking-software lineage runs through **神州数码融信 (Digital China Financial Information)**, the financial-IT unit that built the Sm@rt cores (verified at group level, Section 4.1).
- **Why the structure matters.** Two procurement implications: (1) the Sm@rt products' roadmap and support sit with the 000555 listed arm — contract counterparties and maintenance obligations should be written against that entity, not the group; (2) the group's broader IT-distribution business (hardware, cloud resale) means a DHC-affiliated bid can bundle infrastructure lines in ways pure software houses cannot — an advantage in 信创-era bundled tenders, and a complexity to price carefully.
- **The brand family.** The "@" naming (Sm@rtBanking, Sm@rtOneBank) is the DHC-era design signature (Section 4.1); newer materials also reference Modelb@nk (flagged, Section 4.2). When a bank's RFP says "神州数码" it usually means the 神州信息/融信 financial-software family — worth confirming in the bid, since the two entities are often conflated in casual usage.

---

## 5. The GFS Profile: 高伟达

### 5.1 The Company

**Name and listing.** 高伟达软件股份有限公司 ("Gaoweida Software Co., Ltd."; the English brand used in market materials is **GFS**), **SZSE 300465** on the ChiNext board (stock code and exchange verified via multiple sources; the board membership is consistent with a 2010s ChiNext IPO — **the IPO year itself was not verified this session**). **The expansion of the "GFS" acronym (e.g., "Global Financial Services" or similar) could not be independently verified — flag it; use "GFS" as the brand without expanding it.**

**Business (verified).** The company's verified business description: main business in **financial information services (金融信息服务)** and **mobile-internet marketing (移动互联网营销)**; main products: **system integration (系统集成), software business (软件业务), IT operations/maintenance services (IT运维服务), and mobile-internet business (移动互联网业务)**. The umbrella guide's §3.6 places 高伟达 in the mid-tier banking-IT services cluster ("banking IT services") alongside Teamax and 润和软件 — the SI layer that implements and localises the top vendors' cores (umbrella §3.6, verified as a category). GFS's own positioning emphasises **bank IT cloud services and bank client resources** (reported via low-quality aggregator pages — **flag the sourcing**).

**Financial profile (reported, weak sourcing).** SEO-aggregator pages report GFS swinging between losses and small profits (e.g., an H1 "turnaround to profit of RMB 14.716 million" in one summary) — **flag: this is low-quality secondary sourcing; use the company's annual reports for any financial claim.**

### 5.2 The Products and Positioning

GFS is best understood as a **Tier-2 banking-IT services vendor**: system integration, software development, IT operations and cloud services for banks, with a customer base concentrated in financial institutions (the "advantage in bank client resources" refrain in market commentary). It is **not** marketed as a packaged-core product house in the Sunline/Sm@rt sense — its role in core programmes is implementation, integration, operations and adjacent software. **No named GFS core-banking installation was verified this session — flag.**

### 5.3 The GFS Table

| Aspect | Description (verified unless flagged) |
|---|---|
| **Chinese name** | 高伟达软件股份有限公司 (高伟达) |
| **English name** | GFS (brand; **acronym expansion unverified**) |
| **Listing** | SZSE **300465** (ChiNext) (verified); IPO year **not verified** |
| **Founded** | **Not verified this session** — check company history page |
| **Tier** | Tier 2 — banking-IT services/SI (this guide's analysis) |
| **Business** | Financial information services; system integration; software; IT ops; mobile-internet marketing (verified) |
| **Core products** | None marketed as a packaged core — implementation/integration/ops role in core programmes (assessment) |
| **Clients** | Bank client base reported as its strength; **named clients not verified** — flag |
| **Financials** | Profit/loss swings reported (weak sourcing — **flagged**); use annual reports |
| **Key strength** | Bank-client relationships; SI/ops delivery capacity |
| **Key watch-item** | No flagship core product; weakest public-information footprint of the five profiled vendors |

### 5.4 The Services Vendors' Playbook: GFS, Nantian and Teamax in the 信创 Era

GFS is the purest example of the services-vendor archetype (with Nantian and Teamax as cousins), and understanding that archetype explains how these companies survive and win in the 信创 era without a flagship core:

- **Partner, don't compete.** Services vendors live in the product+project ecosystem (Section 1.4) as the *delivery bench* of the product houses: when Sunline, Shenzhou or GienTech wins a core deal, the implementation work is frequently subcontracted or co-delivered by Tier-2 SIs. GFS's "bank client resources" (reported) are the asset the product houses pay for — proximity to banks that need local delivery.
- **Own the adjacent workstreams.** Migration services (data migration, 换心-style cutover support), **operations & maintenance** contracts (the annuity layer), bank-cloud operations, and channel/self-service integration are services-vendor territory. Nantian's self-service hardware franchise (Section 2.5) and GFS's IT-ops portfolio (verified business description) are exactly these workstreams.
- **The 信创 migration services play.** Every 信创 substitution (database swap, OS migration, middleware replacement, ATM/VTM domestic-stack refresh) is a *services* engagement regardless of which product wins — and the services vendors are the ones who staff it. The 信创 era has been a net positive for this archetype even where the core product went elsewhere.
- **Why they matter in a selection.** In the worked example (Section 8.3), Nantian and GFS fail Gate 0 as core candidates but remain in the programme as implementation partners. The practical rule: **evaluate the services layer as part of the core deal** — the winning core vendor's bench plus the best local SI is what actually delivers go-live. A bank that scores only the product ignores half the risk.

---

## 6. The Hundsun Profile: 恒生电子

### 6.1 The Company: Founded 1995

**Name and listing.** Hundsun Technologies Inc. (恒生电子股份有限公司, "Hundsun", 恒生电子), **SHSE 600570**. **Founded in 1995 in Hangzhou — verified** (multiple independent sources: Tracxn, Similarweb, AlgoTradingMap, and China Banking News, which adds the detail "first established in Hangzhou in 1995"). **Listed on the Shanghai Stock Exchange in 2003 — verified** (China Banking News). Scale: **~8,000 employees** (reported, AlgoTradingMap). Profile: ranked **43rd on the 2019 FINTECH 100** list (verified via China Banking News). Ownership: **Ant Group (Alibaba's fintech arm) has been Hundsun's controlling shareholder since 2014 — widely reported but not verified this session; flag it.**

Hundsun is the **dominant Chinese securities and funds-market software house** — the vendor of the systems that run the Chinese capital markets: securities trading platforms, fund transfer-agency (TA) systems, investment-management systems, and wealth-management platforms (verified category descriptions; the umbrella §3.8 states the same).

### 6.2 The Products: The Wealth Stack

Hundsun's verified product categories (Tracxn, CIOReview Asia-Pacific profile): **wealth-management products, treasury-management solutions, business bill-management solutions, depository-supervision products, omnichannel-banking solutions, and payment-settlement products**, serving institutions in **securities, futures, funds, trust, insurance, banking, exchanges and private placements**, plus wealth-management tools for individual investors.

The **wealth angle is the banking story**: for banks, Hundsun's products are the **wealth-management (理财), treasury (资金) and channel** layer — the systems through which a bank runs its wealth-management products, bond/treasury operations and omnichannel banking — **not the retail deposit/loan core** (verified category: the umbrella §3.8 explicitly states Hundsun's banking business covers wealth-management, treasury and channel products, not the core). Widely reported flagship product names (UF3.0 securities trading platform, O32/O45 investment-management platforms, TA fund systems, LDP low-latency platform) are industry-standard references but were **not individually verified this session — flag**.

### 6.3 Hundsun and the Core: Why Not a Core Vendor

The single most important thing to understand about Hundsun in a core-banking context: **it is not a core-banking vendor in the account-processing sense.** It appears in every "Chinese banking IT" list because (a) it is the largest financial-software company in China by market position, (b) banks are major clients for its wealth/treasury/channel products, and (c) the Chinese market is segmented — core vs channel vs securities — far more cleanly than in the West (umbrella §3.8). In a core rebuild, Hundsun appears as the **wealth-management platform and channel-systems vendor that must integrate with the new core** — a critical integration partner, not a core competitor. Its 信创 relevance is real (every securities/wealth system must also go domestic), but it is the securities/wealth side of 信创, not the retail-core side.

### 6.4 The Hundsun Table

| Aspect | Description (verified unless flagged) |
|---|---|
| **Chinese name** | 恒生电子股份有限公司 (恒生电子) |
| **English name** | Hundsun Technologies Inc. |
| **Listing** | SHSE **600570**; listed 2003 (verified) |
| **Founded** | **1995, Hangzhou (verified)** |
| **Employees / profile** | ~8,000 (reported); 43rd in 2019 FINTECH 100 (verified) |
| **Ownership** | Ant Group controlling shareholder since 2014 (**flagged — reported, not verified this session**) |
| **Tier** | Tier 3 — securities/funds/wealth systems; **not a retail-core vendor** (umbrella §3.8) |
| **Banking products** | Wealth-management (理财), treasury, bill-management, depository-supervision, omnichannel-banking, payment-settlement (verified categories) |
| **Flagship products** | UF3.0, O32/O45, TA, LDP (**flagged — widely reported, not individually verified**) |
| **Core-banking role** | Integration partner on the wealth/channel layer of core programmes; not a core competitor (assessment) |
| **Clients** | Securities firms, funds, and banks' wealth side (verified category); named bank clients not verified this session |
| **Key strength** | Market dominance in securities/wealth software; capital-markets expertise |
| **Key watch-item** | Scope discipline — Hundsun in a core RFP usually means "wealth platform + integration", not "core" |

### 6.5 Hundsun and the Core Programme: The Integration Interface

For a bank rebuilding its core, Hundsun is not a competitor — it is one of the two or three largest integration workstreams in the programme:

- **What must integrate.** The bank's **wealth-management platform (理财系统)** must be re-pointed at the new core's product factory (products, pricing, holdings, redemptions); **treasury-management systems** need real-time positions, GL postings and settlement feeds from the new core; **omnichannel-banking solutions** (the bank's internet/mobile banking built on Hundsun or channel vendors) must ride the new core's channel APIs; and **payment-settlement products** must align with the new core's payments hub. In the worked example (Section 8.2, criterion 5), prior Hundsun-integration references are a scored criterion precisely because this layer is where Chinese core programmes routinely bleed time.
- **The integration pattern.** In practice the Hundsun layer connects through a message/API layer rather than sharing the core's tables — the wealth platform treats the core as the account system of record. The pattern is analogous to how a Western bank's wealth platforms integrate with Temenos/FLEXCUBE cores ([temenos_guide.md](temenos_guide.md), [oracle_flexcube_data_model_guide.md](oracle_flexcube_data_model_guide.md)) — the difference is the integration *catalogue* is Chinese (CNAPS, UnionPay, NetsUnion, e-CNY, provincial unions, and the Hundsun ecosystem itself).
- **Hundsun's own 信创 obligations.** Hundsun's products must themselves run on domestic stacks (securities/wealth 信创 is a parallel mandate to the banking-core one — umbrella §7.5). This is a reported industry trend (Hundsun's domestic-stack migration of its product line has been widely covered) — **flagged: not verified this session at product-version level.**
- **The architect's takeaway.** Scope Hundsun as a named workstream with its own budget, timeline and joint testing with the core vendor — and require the core vendor to evidence prior Hundsun integrations (as the worked example's criterion 5 does). A core RFP that ignores the wealth layer will discover it in the integration phase, at the worst possible moment.

---

## 7. The Comparison

### 7.1 The Vendor Comparison Table

| Vendor | Focus | Strengths | Notes (verified unless flagged) |
|---|---|---|---|
| **Sunline / Changliang (300348)** | Packaged **core-banking product** (parameterised → distributed) | China's dedicated core product house; GaussDB+Taishan joint distributed core; hundreds of city/rural-bank implementations; industry chronicler | Profile in umbrella §3.2; **not** one of the five deep-dived here, but the benchmark every core RFP compares against |
| **Shenzhou Information / DCITS (000555)** | Full banking suite; **Sm@rt** cores (Sm@rtBanking → Sm@rtOneBank distributed) | DHC/Digital China heritage (verified); 20,700 staff, 300+ products (verified); rural/agri-finance and data strength; SI depth | Named client installs not verified this session (Section 4.3); predecessor-shell listing history flagged |
| **GienTech (中电金信)** | **State-affiliated full-stack** fintech (core, channels, data, cloud) | CEC backing; Pactera legacy reach ("95%+ of Chinese banks" — vendor claim); ORIGIEN platform; natural 信创 beneficiary | Umbrella §3.4; market-rank claims collide with Yusys's (Section 1.2) |
| **Yusys / 宇信科技 (300674)** | Broad banking IT: **distributed next-gen core**, channels, financial cloud, credit/risk, e-CNY | Distributed-core credibility (positioned to replace IBM/FLEXCUBE/TCS/SAP); large-bank-heavy client mix (57% revenue from SOE + major city banks, 2025 AR summary); 100+ financial clients (vendor claim) | 2024 revenue -20%+ (reported); H-share IPO filed 2025 (reported); "7-year IDC No.1" claim contested (Section 3.1) |
| **Nantian / 南天信息 (000948)** | Financial digital solutions, **SI/services + self-service equipment**; core-adjacent service-grid product | Yunnan SOE ownership (state-linked procurement); 30+ years banking-automation heritage (reported); regional delivery in southwest China | "Founded 1998" flagged; named core clients unverified (Section 2.3); smallest core-product footprint of the five |
| **GFS / 高伟达 (300465)** | Banking-IT **services**: integration, software, IT ops, bank-cloud services | Bank client relationships (reported); SI/ops delivery capacity | Weakest public-information footprint; "GFS" acronym expansion unverified; no packaged core (Section 5) |
| **Teamax / 天阳科技 (300872)** | Banking-IT services: credit-card, risk, core implementation | Mid-tier delivery capacity | Umbrella §3.6; not deep-dived here |
| **Hundsun / 恒生电子 (600570)** | **Securities/funds/wealth** systems; banks' wealth, treasury, channel layer | Founded 1995, listed 2003 (verified); dominant in Chinese capital-markets software; 43rd FINTECH 100 2019 (verified) | **Not a retail-core vendor** (Section 6.3); in core programmes it is the wealth-platform/channel integration partner; Ant Group control flagged |

### 7.2 Reading the Comparison

Three patterns jump out of the table:

1. **The core is a two-tier game.** The *product* tier (Sunline, Shenzhou/DCITS, GienTech) owns the packaged cores; the *services* tier (Nantian, GFS, Teamax, Yusys-as-deliverer) owns the delivery. Chinese banks almost never buy "core software" — they buy a **product + project (产品+项目)** bundle, which is why the umbrella's §3.1 calls every vendor an SI.
2. **Ownership predicts positioning.** Founder-led vendors (Sunline, Yusys) innovate on product; state-affiliated vendors (GienTech under CEC, Nantian under Yunnan SASAC) win on policy proximity. A 信创-era RFP is won at the intersection of both.
3. **Hundsun is the outlier that proves the rule.** It is the largest financial-software company in China and still not a core vendor — the cleanest evidence that the Chinese market segments core/channel/securities far more sharply than the West (umbrella §3.8).

### 7.3 The Deal-Type Map: Who Wins Which Engagement

The vendor comparison is only useful when mapped to the actual deal types in the Chinese market (illustrative, based on the profiles above and the umbrella's tier table §1.1):

| Deal type | Typical buyer | Likely winners | Why |
|---|---|---|---|
| **Full core rebuild** (distributed, 信创 stack) | City commercial banks (~RMB 200–600bn assets) | Sunline, Shenzhou/DCITS, GienTech, Yusys | Product + 信创 certification + same-tier references (Section 8.3) |
| **Core rebuild, rural/long tail** | Rural commercial banks, provincial unions, village banks | GienTech (state full-stack), Shenzhou (rural/agri strength), Sunline; increasingly **BaaS platforms** | Shared-platform economics; the 3,000-institution tail (umbrella §1.1, §3.9) |
| **Channel/wealth layer** (riding the core) | Any bank rebuilding | Hundsun (wealth/treasury), Kelan (e-banking), channel vendors | The layer is separate from the core in the Chinese market (umbrella §3.8) |
| **信创 infrastructure substitution** | All banks | Huawei/ZTE (DB+hardware), domestic DBs, plus **services vendors** (Nantian, GFS) for migration | The database/hardware procurement line is separate and often larger than the app line (Section 1.4) |
| **Digital-currency (e-CNY) projects** | Banks with e-CNY mandates | Yusys (positioned), platform vendors | e-CNY is a new build; Yusys lists it among leadership areas (Section 3.2) |
| **Big-bank supplementary SI** | Big Four / joint-stock banks | GienTech, Nantian, GFS, Teamax as delivery partners | The top tier self-develops cores (umbrella §4) but buys services around them |
| **Overseas/export deals** | Southeast-Asian and other banks | Sunline (international arm), Yusys (globalisation push), GienTech (ORIGIEN in HK) | Chinese distributed-core architecture is being exported (Section 1.2, point 5) |

The map's lesson: **no vendor wins every deal type, and the segmentation is cleaner than in the West** — a bank's procurement team should read its own deal type from this map and then score only the vendors that actually compete there.

---

## 8. The Worked Example: A Vendor Selection for a Mid-Size Chinese Bank

### 8.1 The Scenario

**The bank (fictional, familiar context).** "Huahe Bank" (华河银行 — **fictional name; any resemblance to a real institution is coincidental**) is a mid-size **city commercial bank** in a provincial capital in central China. Profile, deliberately drawn from the umbrella's tier table ([chinese_bank_core_systems_guide.md](chinese_bank_core_systems_guide.md) §1.1 — the ~125 city commercial banks):

- ~RMB 400 billion in assets; ~150 branches; ~8 million retail customers; a growing SME-lending book and a wealth-management (理财) business.
- **Legacy estate:** a data-centralisation-era core (built in the 2000s 数据大集中 wave, umbrella §2.2) running on **Oracle on x86** — with the classic Chinese mid-bank ancestry that, in an earlier generation, often meant **AS/400-based systems** for the smaller institutions ([../technology/ibm_as400_guide.md](../technology/ibm_as400_guide.md) covers that heritage; Huahe's own predecessor bank ran one). The core is parameter-poor, batch-heavy, and its vendor has effectively stopped developing it.
- **The trigger:** the **信创 deadline** (umbrella §7.5 — "full domestic substitution targeted by ~2027 for key industries", reported) and a regulator-supervised resilience review after a weekend outage. The board has approved a **core rebuild programme**: new distributed core, domestic stack, go-live phased over 2026–2028.
- **Non-negotiables:** the new core must run on a **信创-certified stack** (domestic database, middleware, OS — umbrella §7.5); it must be **distributed with dual-active (双活) capability**; it must integrate the bank's existing **Hundsun wealth-management platform** and channel systems; and the bank's own IT team (60 people) will not grow.

This is the "familiar context" for the reader: a mid-size bank's core rebuild is the same project everywhere — the difference in China is the **mandated domestic stack, the distributed-first architecture, and the product+project vendor model**.

### 8.2 The 信创-Era Selection Criteria (The Design)

The criteria design below is the deliverable of this section: eight weighted criteria, deliberately ordered so that the **hard 信创 gates come first** and everything else scores within the compliant set. (Weights are the bank's own; the point is the structure, not the numbers.)

**Gate 0 — 信创 compliance (hard gate, not scored).** Each candidate must evidence, before scoring:

- **Product in the financial-信创 product catalogue (产品名录) / certified compatibility** for the target stack — domestic **database** (GaussDB / GoldenDB / OceanBase / DM / TiDB — umbrella §7.1), **middleware** (e.g., TongWeb, or the vendor's own), **OS** (Kylin / UOS / openEuler — umbrella §7.4).
- **A named reference** of the *exact product version* running on the *exact database* the bank chooses. A vendor whose distributed core is GaussDB-certified but not GoldenDB-certified is not a candidate if the bank chooses GoldenDB — the 换心 (heart-transplant) lesson from CITIC Bank is that the database is the critical path (umbrella §3.7, §5).
- **A 信创-architecture statement** covering chips (Kunpeng/Hygon/Phytium — umbrella §7.3), dual-active data centres, and data-residency/regulatory reporting (PBOC/NFRA formats built in).

Candidates failing Gate 0 are excluded regardless of score — this is the single biggest difference from a Western core RFP ([core_banking_systems_guide.md](core_banking_systems_guide.md) for the Western counterpart).

**Scored criteria (weights in parentheses):**

1. **Distributed architecture maturity (20%).** Microservices granularity, account-sharding/cell (单元化) design, dual-active (双活) evidence, peak-TPS headroom (Huahe models a 双11/Spring-Festival spike of ~10x normal — umbrella §1.3), online schema evolution, and the batch-window story (the legacy core's night batch is the bank's biggest pain).
2. **Reference clients at comparable scale (15%).** City commercial banks of RMB 200–600bn assets that went live on the same product in the last 3 years — **not** the vendor's flagship big-bank logo projects. Site visits to two live references are mandatory.
3. **Delivery and local footprint (15%).** On-site implementation teams, provincial presence (Huahe is in central China — a vendor whose bench is Shenzhen/Beijing-only costs more in travel and retention), and the vendor's own capacity for 7×24 operations handover.
4. **Parameterisation depth (10%).** Product-launch speed for Huahe's actual product set (SME loans, deposits, wealth-linked accounts); how many of Huahe's products can be configured without code.
5. **Ecosystem and integration (10%).** Out-of-box connectors for CNAPS, UnionPay, NetsUnion, e-CNY, the local provincial-union interfaces, and — critically — the **Hundsun wealth platform** and the bank's channel systems (umbrella §2, §3.8). Every core vendor claims "open APIs"; the scored evidence is prior integrations with Hundsun products at other banks.
6. **Commercial model (10%).** License + product+project pricing vs. **banking-as-a-service / shared-platform** pricing (umbrella §3.9) for Huahe's size; TCO over 5 years including the domestic-stack infrastructure; penalty/liquidated-damage terms for the phased go-lives.
7. **Vendor stability and ownership (10%).** Listed status, controlling shareholder (SOE vs. founder-led), financial health from **annual reports** (the Yusys 2024 revenue decline and GFS profit swings — Sections 3.1, 5.1 — are exactly why press headlines must be checked against audited numbers), and the vendor's 信创 investment trajectory.
8. **Reference-quality and industry standing (10%).** The vendor's own published architecture material (Sunline's ten-year core-history report is the model), its IDC standing (**ignored as marketing** — Section 1.2), and its product roadmap credibility.

### 8.3 The Shortlist and Scoring

**Gate 0 survivors (illustrative):** **Sunline (300348)**, **Shenzhou Information/DCITS (000555)**, **GienTech**, **Yusys (300674)**. Nantian and GFS fail Gate 0 as *core* candidates (no flagship packaged core at Huahe's tier — Sections 2, 5) but are retained in the programme as **implementation/SI partners** — the 产品+项目 model means the winner's delivery bench can be augmented by a Tier-2 SI, and Nantian's SOE/SI profile is a credible complement if the bank wants a provincial-state-linked partner.

**Illustrative scores (Huahe's weights; the numbers are the bank's judgement, not verified market data):**

| Criterion (weight) | Sunline | Shenzhou | GienTech | Yusys |
|---|---|---|---|---|
| Distributed architecture (20%) | 9 | 8 | 8 | 9 |
| References at scale (15%) | 9 | 8 | 7 | 7 |
| Delivery / local footprint (15%) | 7 | 8 | 8 | 8 |
| Parameterisation (10%) | 9 | 7 | 7 | 8 |
| Ecosystem / Hundsun integration (10%) | 8 | 8 | 8 | 9 |
| Commercial (10%) | 8 | 7 | 7 | 8 |
| Stability / ownership (10%) | 7 | 8 | 9 | 6 |
| Reference quality (10%) | 9 | 8 | 7 | 8 |
| **Weighted total** | **8.4** | **7.8** | **7.7** | **7.9** |

The outcome pattern (illustrative): **Sunline** wins narrowly on product depth and mid-market references — the typical result for a city commercial bank — with **Yusys** close behind on ecosystem breadth, **GienTech** strong on ownership stability, and **Shenzhou** the value pick on SI depth and rural/data strength. In a real programme the tie-breakers are the two live reference visits and the database decision made *before* the vendor decision (Gate 0). Note the corollary: had the bank chosen **GoldenDB** as its database (e.g., via a CITIC-style preference), the shortlist would reshuffle — the database choice, not the vendor marketing, is what actually selects the vendor.

### 8.4 The Lessons

1. **信创 compliance is a gate, not a criterion.** The RFP's first page excludes more vendors than the scoring does. Never let a strong product into scoring without a certified stack for *your* chosen database.
2. **Choose the database before the vendor.** The 换心 experience (CITIC/GoldenDB, PSBC/Huawei-GaussDB — umbrella §3.7) makes the database the critical path; the core vendor must be certified against it.
3. **References at your tier beat logos.** A big-bank flagship project tells you nothing about delivery to a 400bn-asset city bank; two live, same-tier references are worth more than any IDC ranking (which, in any case, is contested vendor marketing — Section 1.2).
4. **The SI layer is half the product.** In the product+project model, the winning vendor's bench and the Tier-2 SI partners (Nantian, GFS, Teamax) determine go-live reality as much as the core code.
5. **Integrations are where programmes die.** The Hundsun wealth platform, channels, CNAPS/UnionPay/NetsUnion and e-CNY connectors must be evidenced, not promised — integration scope is routinely 30%+ of a Chinese core rebuild.
6. **Check annual reports, not press.** Vendor financials drive delivery stability (Yusys's 2024 dip and H-share plans; GFS's swings). For a foreign architect, this is standard vendor-due-diligence discipline — the same as for any Western core vendor ([temenos_guide.md](temenos_guide.md), [oracle_flexcube_data_model_guide.md](oracle_flexcube_data_model_guide.md) for the counterpart discipline on international vendors).
7. **Don't buy the "No. 1" claim.** The Yusys-vs-GienTech ranking collision (Section 1.2) is a standing warning: Chinese vendor market-share claims are marketing artefacts of different IDC editions and segments.
8. **Scope Hundsun correctly.** A core RFP that lists Hundsun as a core competitor is mis-scoped; Hundsun is the wealth-platform integration partner (Section 6.3). Getting this taxonomy right saves months of confusion.

### 8.5 The RFP Annex: What the Bank Should Actually Ask

A clause checklist for the 信创-era core RFP — the evidence a mid-size bank should demand from every shortlisted vendor (this annex is the "design" deliverable of the selection, alongside the criteria in Section 8.2):

1. **Gate-0 evidence pack.** Product-catalogue (产品名录) entry for the financial-信创 programme; compatibility/certification letters for the bank's chosen **database, middleware and OS**; and one named reference running the *exact product version* on the *exact database* — plus a 信创-architecture statement (chips, servers, dual-active sites, data-residency and PBOC/NFRA reporting formats).
2. **Architecture artefacts.** Sharding/cell (单元化) design documents; dual-active (双活) DR test reports with measured RPO/RTO; a capacity model demonstrating the bank's ~10× peak (双11/Spring-Festival) profile; batch-window targets for the bank's actual product set; online schema-evolution and product-launch (parameterisation) demos.
3. **Reference protocol.** Two live, same-tier references (city commercial banks of comparable assets) with named contacts, site-visit dates, 3-year SLA/incident statistics, and the references' own go-live lessons-learned.
4. **Integration evidence.** Prior integrations with **Hundsun wealth products**; certification for CNAPS, UnionPay, NetsUnion, e-CNY and provincial-union interfaces; channel-SDK documentation; and a named integration architect on the bid.
5. **Delivery plan.** Team structure and CVs of the proposed project manager and chief architect; provincial presence/local bench plan; the subcontracting map (which Tier-2 SIs — Nantian/GFS/Teamax-type — will deliver which workstreams); and the knowledge-transfer and operations-handover plan to the bank's 60-person IT team.
6. **Commercial terms.** 5-year TCO including the domestic-stack infrastructure line; liquidated damages per phase; **banking-as-a-service** option pricing for the bank's subsidiaries; and exit/transition terms (source escrow, IP assignment for customisations, data-export guarantees).
7. **Stability evidence.** Three years of audited annual reports; controlling-shareholder structure (SOE vs. founder-led); key-person retention commitments; and the vendor's 信创 investment roadmap.
8. **Governance.** Data-residency commitments (数据安全法/PIPL), code escrow, security-certification status, and regulatory-reporting acceptance criteria.

### 8.6 The Phasing Plan

A realistic phasing for a mid-size bank's core rebuild (2026–2028 horizon, aligned with the reported 信创 ~2027 target):

- **Phase 0 — Foundation (months 0–3).** Choose the **database** (the critical path — Section 8.4, lesson 2); run Gate-0 certification checks; two reference-site visits; freeze the vendor shortlist.
- **Phase 1 — Design and POC (months 3–9).** Solution architecture and data-migration design (换心-style mapping — umbrella §3.7); a sandbox POC on the chosen domestic stack proving the peak-load model; integration design with Hundsun and channels.
- **Phase 2 — Build and configure (months 9–21).** Product configuration (parameterisation) for Huahe's actual products; integration build (Hundsun wealth, channels, payments, e-CNY); parallel-run (双轨运行) design and test data migration; regulatory reporting build.
- **Phase 3 — Phased go-live (months 21–27).** Cutover by product line (deposits → loans → payments), dual-active operation, shadow running against the legacy core, rollback drills.
- **Phase 4 — Stabilise and hand over (months 27–36).** Operations handover to the bank's IT team; 信创 certification completion; legacy-core decommission; lessons-learned and vendor SLA review.

### 8.7 After Selection: The Contract and the First 100 Days

Selection is where the architecture work begins, not ends. What the mid-size bank should lock down immediately after 中标 (award):

- **The contract's hard edges.** Scope the **database and infrastructure procurement as a parallel contract** (Section 1.4) with a named compatibility owner — the classic failure mode is the app vendor and the DB vendor each pointing at the other when certification issues surface. Write **liquidated damages against the phase milestones** (Section 8.5, clause 6), not just the final go-live; Chinese core programmes are phased, so penalties must be phased too. Agree **exit terms up front** — source escrow, customisation IP assignment, and a data-export guarantee — because switching core vendors mid-programme is the worst-case scenario nobody plans for.
- **The first 100 days.** Freeze the **architecture decision record** (database, cell/sharding design, integration pattern with Hundsun and channels); stand up the joint governance board (bank IT + vendor PM + the Tier-2 SI lead); run the **Gate-0 evidence audit** against the delivered product version (certification letters, catalogue entry); and book the two reference-site visits before any build work starts. The first deliverable is not code — it is the **data-migration design** (the 换心 critical path, Section 8.4 lesson 2), reviewed with the bank's own data team and the regulator's reporting requirements in the room.
- **The relationship reality.** In the product+project model the vendor's local bench *is* the product's delivery quality (Section 5.4). The first 100 days should therefore also validate the **named team** — the PM and chief architect from the bid must actually be the ones on site, and the subcontracting map (which SI does which workstream) must be signed off before the second tranche of fees is payable.

---

## 9. The Summary: The Vendors Behind the Chinese Cores

One page, for the busy reader:

**The market.** Chinese core banking runs on a consolidated domestic vendor set — a handful of listed companies operating on a "product + project" (产品+项目) model, selling cores the way the West sells packaged platforms but delivering them like SI firms. The market-share rankings are contested vendor marketing (Yusys's "7-year No. 1" collides with GienTech's "nine championships"); the underlying reality is a stable oligopoly of product houses (Sunline, Shenzhou/DCITS, GienTech) plus a services layer (Yusys, Nantian, GFS, Teamax) under a state-mandated domestic stack. 信创 does not merely favour these vendors — it *is* their market.

**The five profiled vendors, in one line each:**

- **Nantian (南天信息, 000948)** — the Yunnan SOE with a 30-year self-service/hardware and SI heritage; core-adjacent service-grid product; regional southwest strength; the state-owned delivery partner rather than the product winner. Founded-1998 and client claims flagged.
- **Yusys (宇信科技, 300674)** — founded 1999, NASDAQ 2007, ChiNext 2018 (all verified; delisting year flagged), Hong Kong IPO filed 2025; the distributed next-gen core vendor positioned explicitly to replace IBM/Oracle FLEXCUBE/TCS/SAP, with channels, financial cloud and a large-bank-heavy client mix — and a rocky 2024 behind it.
- **Shenzhou Information (神州信息, 000555)** — the DHC/Digital China information-services arm, heir to the **Sm@rt** core family (Sm@rtBanking → Sm@rtOneBank), full-suite breadth, rural/agri-finance franchise; the SI-plus-product house of the 神州数码 ecosystem.
- **GFS (高伟达, 300465)** — the ChiNext-listed banking-IT services firm: integration, software, ops and bank-cloud services; real client relationships, thin public footprint, no flagship packaged core; the acronym's expansion itself is unverified.
- **Hundsun (恒生电子, 600570)** — founded 1995, listed 2003, the giant of Chinese securities/funds/wealth software; for banks it is the wealth-management, treasury and channel layer — the most important integration partner in a core programme, and definitively not a core vendor.

**The pattern to take away.** The Chinese vendor landscape is the mirror image of the Western one: where the West has international packaged cores (Temenos, FLEXCUBE, Finacle) with domestic SIs, China has domestic product houses with state policy as the demand engine. The final word is this: **the vendors behind the Chinese cores are not just software companies — they are the delivery mechanism of a national technology policy, and the ones that will export the Chinese distributed-core architecture to the rest of Asia.** For a Solution Architect watching from Singapore, the practical reading is: know the five names, verify everything they claim, choose the database first, and scope Hundsun as a partner.

---

## 10. The Glossary

| Term | Definition |
|---|---|
| **Nantian** | English brand of 南天信息 — Yunnan Nantian Electronic Information Industry Co., Ltd., SZSE 000948; Yunnan-SOE-owned banking-IT/SI company with a self-service-equipment heritage (Section 2) |
| **南天信息** | Chinese name of Nantian Information (云南南天电子信息产业股份有限公司); listed Shenzhen 1999; corporate founding 1998 (flagged) |
| **Yusys** | English name of 宇信科技 — Beijing Yusys Technology Group Co., Ltd., SZSE 300674; founded 1999; distributed next-gen core + channels + financial cloud (Section 3) |
| **宇信科技** | Chinese name of Yusys Technologies; ChiNext IPO November 2018 (verified); H-share IPO filed 2025 (reported) |
| **Shenzhou Information** | English rendering of 神州信息 (Digital China Information Service Group / DCITS), SZSE 000555; the Sm@rt-core house of the DHC ecosystem (Section 4) |
| **神州信息** | Chinese name of DCITS; the listed information-services arm of the Digital China (神州数码) group |
| **DHC** | Digital China (神州数码) — the Legend-derived IT group; DHC lineage = the source of the Sm@rt banking products now held by Shenzhou Information (Section 4.1) |
| **Sm@rt** | The core-banking product family of the DHC/Shenzhou lineage — Sm@rtBanking (legacy integrated core) and Sm@rtOneBank (modern integrated + distributed banking platform); the "@" is the brand's signature (Section 4.2) |
| **GFS** | English brand of 高伟达 (Gaoweida Software Co., Ltd.), SZSE 300465; banking-IT services (integration, software, ops); acronym expansion unverified (Section 5) |
| **高伟达** | Chinese name of GFS (高伟达软件股份有限公司); ChiNext-listed banking-IT services company |
| **Hundsun** | English name of 恒生电子 (Hundsun Technologies Inc.), SHSE 600570; founded 1995, listed 2003; dominant Chinese securities/funds/wealth software house; not a retail-core vendor (Section 6) |
| **恒生电子** | Chinese name of Hundsun Technologies; the wealth-management/treasury/channel layer vendor in banking core programmes |
| **Core banking** | The account-processing heart of a bank — deposits, loans, payments, general ledger, interest — as distinct from channels and analytics; see [core_banking_systems_guide.md](core_banking_systems_guide.md) |
| **Vendor** | A company supplying core-banking software and/or delivery services; in China, virtually always a product+project (产品+项目) SI-vendor hybrid |
| **信创 / Xinchuang** | 信息技术应用创新 ("IT application innovation") — China's national programme to replace foreign IT (chips, servers, OS, middleware, databases, applications) with domestic technology in critical industries, finance among the first wave; the demand engine of the domestic vendor market (umbrella §7.5) |
| **AS/400** | IBM's midrange server line — a common heritage platform of Chinese banks' earlier-generation cores, since replaced by distributed domestic stacks ([../technology/ibm_as400_guide.md](../technology/ibm_as400_guide.md)) |
| **Distributed core** | A core built as sharded, microservices-based, horizontally scalable services (often with cell/单元化 and dual-active design) rather than a single centralised database; the standard architecture of Chinese next-generation cores (umbrella §5) |
| **Client** | In this guide, a financial institution that buys a vendor's systems — "client base" claims must be verified against annual-report case lists (vendor marketing routinely overstates reach) |
| **Revenue** | Vendor top-line income; the Chinese banking-IT vendors' revenues are volatile (Yusys 2024 -20%+ reported), so annual reports, not press, are the evidence base |
| **A-share** | Shares of mainland-Chinese companies traded on the Shanghai or Shenzhen stock exchanges (e.g., Nantian 000948, Yusys 300674, Shenzhou 000555, GFS 300465, Hundsun 600570); most Chinese banking-IT vendors are A-share listed |
| **Listing** | A company's public stock-market listing; listing history (NASDAQ → ChiNext for Yusys; backdoor restructuring for 000555) is a due-diligence flag item for each vendor |
| **China** | The jurisdiction of this guide's vendor market; the PRC's state technology policy (信创), data-localisation law (数据安全法/PIPL) and bank-tier structure shape every procurement (umbrella §1, §7.5, §8.3) |
| **Landscape** | The structure of a vendor market — tiers, ownership, product segmentation, consolidation — used here to mean the Chinese core-vendor market as a whole (Section 1) |
| **产品+项目 (product + project)** | The Chinese vendor business model: license the software *and* deliver it bank-by-bank in long on-site projects; every Chinese core vendor is also an SI (Sections 1.1, 1.4) |
| **单元化 (cell-based architecture)** | The Chinese distributed-core pattern of dividing accounts/tenants into cells so the system scales horizontally and achieves dual-active availability (umbrella §5.3) |
| **换心 ("heart transplant")** | The Chinese metaphor for replacing the legacy database under a core — the critical path of core modernisation, made famous by CITIC Bank's GoldenDB project (umbrella §3.7) |
| **双活 (dual-active)** | Two active data centres serving traffic simultaneously — the Chinese availability pattern that replaced active-passive DR (umbrella §5) |
| **招标 / 入围 / 评标 / 中标** | The Chinese tender cycle: RFP/announcement → shortlisting → scoring → award (Section 1.4) |
| **IDC** | International Data Corporation — the market-research firm whose *China Banking IT Solutions Market* reports are the source of the contested vendor rankings (Sections 1.2, 3.1) |
| **产品名录 (product catalogue)** | The financial-信创 certified-product catalogue — a vendor's ticket into 信创-era procurement (Sections 1.4, 8.2) |

---

## 11. The Claims-Status Table

Per repo convention (see [green_link_digital_bank_guide.md](green_link_digital_bank_guide.md) §5 and the umbrella's §12), China-specific claims are graded: **Verified** = retrieved from a source in this session (web search); **Reported** = widely cited in industry literature but not independently verified here; **Vendor claim** = company marketing; **Unverified** = could not be checked.

| Claim | Status | Notes |
|---|---|---|
| Nantian = Yunnan Nantian Electronic Information Industry Co., Ltd.; SZSE 000948; listed Shenzhen **1999** | **Verified** | Company-profile sources; Investing.com profile |
| Nantian corporate establishment **1998** | **Reported (flagged)** | Single analysis source (hotelaah, Jun 2025); heritage traced to a 1971 state factory (aifun.cc profile) |
| Nantian controlled by Yunnan Industrial Investment Holding Group; ultimate controller Yunnan SASAC | **Reported** | Company member pages + analysis source |
| Nantian business: information business, system integration, application software; products incl. ATMs, printers, POS | **Verified** | Investing.com profile |
| Nantian "Core Banking Service Grid System" product | **Verified (product name)** | en.nantian.com.cn member-company page; scope (full core vs. services layer) **flagged** |
| Nantian named banking clients | **Unverified** | No named install retrievable this session — flagged in Section 2.3 |
| Yusys founded 1999 (founder 洪卫东) | **Verified** | Multiple 2025 press pieces on its HK IPO prospectus |
| Yusys NASDAQ listing 2007 (YTEC) | **Verified** | fx168/gelonghui prospectus coverage |
| Yusys NASDAQ delisting year | **Flagged — sources disagree** | 2023 per 2025 press; circa 2015 per common market record |
| Yusys A-share ChiNext IPO November 2018 (300674) | **Verified** | Multiple sources (cnfol, investing.com, MSN) |
| Yusys HK IPO filed 2025 (A+H plan) | **Reported** | Financial-press coverage (Dec 2025 second submission) |
| Yusys 2024 revenue decline >20%; R&D/headcount cuts criticised | **Reported** | Chinese financial press (fx168, MSN), 2025 |
| Yusys 2025 AR summary: SOE+major city banks 57.39% / small banks 29.63% / non-bank 12.98% | **Reported** | minichart.com.sg summary of the March 2026 annual report |
| Yusys 20% stake in Jinshang Consumer Finance | **Verified (secondary)** | 21st Century Business Herald reporting |
| Yusys "7 consecutive years No.1 in IDC China banking IT solutions market" | **Vendor claim (flagged)** | yusys.com.cn; conflicts with GienTech's claim |
| Yusys Next-Gen Core Business System (distributed microservices; replaces IBM/Oracle FLEXCUBE/TCS/SAP) | **Vendor claim (product exists)** | yusystech.com; product positioning is vendor marketing |
| Shenzhou/DCITS: ~20,700 staff; 300+ products; Sm@rtOneBank | **Verified** | en.dcits.com (per umbrella §3.3) |
| DHC = Digital China; banking lineage via 神州数码融信 | **Verified (group level)** | Umbrella §3.3 + company lineage |
| Sm@rtBanking covered "nearly all banking business areas" incl. e-banking (2002-era) | **Verified** | Sina Finance April 2002 article |
| Modelb@nk 5.0 future-bank distributed core (Shenzhou family) | **Reported (flagged)** | Low-quality aggregator page only |
| Shenzhou listing via predecessor-shell (太光电信) restructuring ~2013–2014 | **Reported (flagged)** | Not verified this session |
| Bank of Chengdu core via Digital China | **Reported (secondary, flagged)** | LinkedIn industry article (2010s-era) |
| GFS = 高伟达软件股份有限公司; SZSE 300465 (ChiNext) | **Verified** | Multiple sources |
| GFS business: financial information services; system integration; software; IT ops; mobile-internet marketing | **Verified** | Company business description via aggregators |
| GFS "GFS" acronym expansion | **Unverified (flagged)** | Use as brand only |
| GFS founding year; GFS IPO year | **Unverified** | Not retrievable this session |
| GFS named bank clients | **Unverified** | Flagged in Section 5.2 |
| GFS profit/loss swings (e.g., H1 net profit ~RMB 14.7m) | **Reported (weak sourcing)** | SEO-aggregator pages — use annual reports |
| Hundsun founded 1995 in Hangzhou | **Verified** | Tracxn, Similarweb, AlgoTradingMap, China Banking News |
| Hundsun listed Shanghai 2003 (600570) | **Verified** | China Banking News |
| Hundsun ~8,000 employees | **Reported** | AlgoTradingMap |
| Hundsun 43rd on 2019 FINTECH 100 | **Verified** | China Banking News |
| Hundsun product categories (wealth mgmt, treasury, bills, depository supervision, omnichannel banking, payment settlement) | **Verified** | Tracxn / CIOReview profile |
| Hundsun controlled by Ant Group since 2014 | **Reported (flagged)** | Widely reported; not verified this session |
| Hundsun flagship product names (UF3.0, O32/O45, TA, LDP) | **Reported (flagged)** | Industry-standard references; not individually verified |
| Hundsun not a retail-core vendor; bank business = wealth/treasury/channel | **Verified (category)** | Umbrella §3.8 + product category sources |
| Hundsun's own products migrating to domestic stacks (信创) | **Reported (flagged)** | Industry trend; not verified at product-version level this session |
| DHC group structure incl. 神州数码集团 (000034) as the distribution arm | **Reported (flagged)** | Ticker 000034 and exact structure not verified this session |
| Yusys related-party history with 易诚互动 (exchange scrutiny) | **Reported** | OFweek coverage (2024) |
| Yusys founder 洪卫东 share sales (2025 press criticism) | **Reported** | MSN Business coverage |
| Procurement cycle (招标→入围→评标→中标) and product+project pricing model | **Verified (concept)** | Consistent with umbrella §3.1 and vendor self-descriptions |
| Market consolidation narrative (信创 shrink, state grouping, revenue pressure) | **Assessment** | Author's synthesis of the verified facts above, labelled as such |
| Vendor tiering (Tier 1/2/3 + platform giants) | **Assessment** | Author's analytical framework, consistent with umbrella §3 |

---

## 12. The References

**Primary sources (vendor sites and filings):**

1. Nantian Information English site — member companies / core-banking service-grid product: en.nantian.com.cn
2. Yusys Technologies — Next-Gen Core Business System: yusystech.com; company catalogue: yusys.com.cn (incl. "7-year IDC No.1" claim)
3. Shenzhou Information / DCITS: en.dcits.com (20,700 staff, 300+ products, Sm@rtOneBank — per umbrella §3.3)
4. GFS (高伟达): company business descriptions via financial aggregators; annual reports for financials (not directly retrieved this session)
5. Hundsun Technologies: hundsun.com; product categories via Tracxn/CIOReview profiles

**Stock-exchange and financial-press sources:**

6. Yusys prospectus-era coverage: fx168news.com, gelonghui.com, cnfol.com, investing.com, MSN Business (2025) — NASDAQ 2007/YTEC, delisting-year discrepancy, ChiNext Nov 2018, HK IPO filing, 2024 revenue decline
7. Yusys 2025 annual-report summary: minichart.com.sg (March 2026) — revenue mix, AI/globalisation narrative
8. Jinshang Consumer Finance stake: 21jingji.com (21st Century Business Herald)
9. Nantian company/founding/ownership: hotelaah.com analysis (Jun 2025), aifun.cc profile, Investing.com/cn profile, xueqiu.com (SZ000948)
10. Hundsun founding/listing/FINTECH-100: China Banking News (chinabankingnews.com, Aug 2020)
11. Sm@rtBanking 2002-era coverage: Sina Finance (finance.sina.com.cn, April 2002)

**Repo sibling guides (cross-referenced):**

12. [chinese_bank_core_systems_guide.md](chinese_bank_core_systems_guide.md) — the Chinese-core umbrella; its §3 (vendor landscape), §1.1 (bank tiers), §5 (去IOE/distributed), §7.5 (信创) and §12 (claims table) are the backbone this guide expands
13. [core_banking_systems_guide.md](core_banking_systems_guide.md) — the global core-banking umbrella
14. [temenos_guide.md](temenos_guide.md), [oracle_flexcube_data_model_guide.md](oracle_flexcube_data_model_guide.md), [apache_fineract_guide.md](apache_fineract_guide.md), [fineract_database_models_guide.md](fineract_database_models_guide.md), [murex_mx3_platform_guide.md](murex_mx3_platform_guide.md), [nasdaq_calypso_guide.md](nasdaq_calypso_guide.md) — the international platform/vendor-guide pattern this guide follows
15. [../technology/ibm_as400_guide.md](../technology/ibm_as400_guide.md) — the AS/400 heritage angle for Chinese cores
16. [../technology/oracle_database_guide.md](../technology/oracle_database_guide.md), [../technology/singapore_data_centres_guide.md](../technology/singapore_data_centres_guide.md) — technology-layer context
17. [insurance_open_source_commercial_guide.md](insurance_open_source_commercial_guide.md) — the vendor-landscape pattern applied to insurance
18. [dbs_bank_guide.md](dbs_bank_guide.md), [ocbc_software_systems_guide.md](ocbc_software_systems_guide.md) — the bank-systems series

**Verification note.** All facts above were verified via web search in August 2026 (search-only backend; page extraction was not available). Where a claim could not be verified, it is flagged in the text and in the [claims-status table](#11-the-claims-status-table). Vendor market-share rankings are quoted only as vendor claims because the underlying IDC reports were not accessible in this session. Client names are stated only where verified or explicitly flagged as reported; no client claim in this guide is fabricated.

**Related reading in this repository.** The companion guides complete the picture: [chinese_bank_core_systems_guide.md](chinese_bank_core_systems_guide.md) for the sector and stack; [core_banking_systems_guide.md](core_banking_systems_guide.md) for the global discipline; [../technology/ibm_as400_guide.md](../technology/ibm_as400_guide.md) for the heritage platforms the distributed cores replaced; and [insurance_open_source_commercial_guide.md](insurance_open_source_commercial_guide.md) for the same vendor-landscape pattern in insurance. For the reader comparing Chinese vendors with the international alternatives, [temenos_guide.md](temenos_guide.md) and [oracle_flexcube_data_model_guide.md](oracle_flexcube_data_model_guide.md) supply the counterpart profiles.

---

*End of guide — The Chinese Core-Banking Vendors: Nantian, Yusys, Shenzhou, GFS, Hundsun and the landscape around them.*
