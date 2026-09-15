# SignalPlus: The Institutional Crypto-Options Trading-Technology Company — A Comprehensive Guide

**The Business, Founders, Funding, Products, Technology, Market Position, Regulatory Perimeter and Regulated-Institution Risk of SignalPlus (signalplus.com) — from a 2021 Hong Kong Founding by Chris Yu and James Shan to the US$11M Series B of January 2025, the US$40M HashKey-Led Series B+ of May 2026, the US$50M Series B1 at a US$500M Post-Money Valuation of June 2026, the US$160 Billion Q4-2025 Platform-Volume Claim, the Deribit Block-RFQ Franchise, and a Cymbal Bank Digital-Asset Desk Adoption Worked Example**

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Banking Domain / Digital-Asset Trading-Technology Company Deep-Dive — the crypto-options software business model (terminal plus enterprise automation licence), option pricing and volatility-surface technology, exchange connectivity and DMA, automated market-making and delta-hedging robots, the funding history from Seed to Series B1, the Hong Kong SFC and Singapore MAS perimeters around a software vendor, the vendor-diligence disciplines a regulated desk must apply to an uninspectable third-party pricing model, and the Cymbal Bank lens
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026
> **Companion guides (sibling, same folder — the banking cluster):** [Fireblocks](fireblocks_guide.md) (the digital-asset custody and infrastructure vendor genre — the custody layer of the same institutional stack; cross-ref §1, §6, condensed) · [Bitunix](bitunix_guide.md) (a crypto exchange's own software stack and the CEX worked-example conventions — the venue side of the integrations this guide discusses; cross-ref §6, §7, condensed) · [Market Making in Singapore](market_making_singapore_guide.md) (the market-making firms that are SignalPlus's target buyers and the MAS conduct context; cross-ref §6, §8) · [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) (the Singapore licensing regime — owns the MAS detail; cross-ref §9, condensed) · [Operational Resilience Framework](operational_resilience_framework_guide.md) (owns the third-party resilience and important-business-service framework; cross-ref §10, condensed)
> **Companion guides (management/, prefix `../management/`):** [Vendor Management](../management/vendor_management_guide.md) (the general vendor-diligence FRAMEWORK — segmentation, selection, contracting, VRM/TPRM, exit; this guide does NOT rebuild it — cross-ref §10)
> **Companion guides (technology/, prefix `../technology/`):** [Trading System Software Architecture](../technology/trading_system_software_architecture_guide.md) (the architecture vocabulary — matching engines, market-data fan-out, latency budgets; cross-ref §5, condensed, not re-derived) · [Cybersecurity](../technology/cybersecurity_guide.md) (the security-discipline themes — key management, API credential custody, ISO/IEC 27001, SOC reporting; cross-ref §5, §10, condensed)

---

**How to use this guide:** Section 1 is the overview — what SignalPlus is and is not, the namesake disambiguation, the key-facts table, why a bank or fund should care, and the evidence base with source-quality ratings. Section 2 is the company, the founders and the team. Section 3 is the funding history — the round-by-round table, the aggregation risk on the widely quoted "US$23M total", the corrected 2026 position, the two-rounds-or-one question, and the US$5B valuation error. Section 4 is the product suite — every module with what the company says versus what independent evidence exists. Section 5 is the technology — pricing, volatility surfaces, Greeks, connectivity, hedging mechanics, the latency and AI claims, and a plain list of what is not disclosed. Section 6 is the business model and the customers, including the customer-roster handling rule and the Goldman Sachs correction. Section 7 is the market it sits in — crypto-options scale, venue concentration, institutional share, onshoring, block venues, and the clearing model. Section 8 is the competitive landscape with a positioning table. Section 9 is the regulatory perimeter — the SFC and MAS regimes, and the software-versus-dealing classification question. Section 10 is the regulated-institution angle and the vendor-diligence disciplines. Section 11 is the Cymbal Bank worked example. Section 12 is the claims audit (✅ verified / ⚠ flagged / ❌ refuted or corrected). Section 13 is "What Could Not Be Verified". Section 14 is the glossary. Section 15 is cross-references and further reading. Section 16 is the closing summary. **The one framing point to get right before anything else:** SignalPlus is a **software and trading-technology company**. It is **not** an exchange, **not** a custodian and — on all public evidence examined for this guide — **not** a principal market maker trading its own book. It sells the technology that lets other firms market-make. AppWorks, an investor, states the posture explicitly: "instead of focusing on proprietary trading, the team was steadfast in perfecting their software solutions as a best-in-class offering" (AppWorks, "Why We Invested", 27 March 2025 — investor marketing, but a specific and checkable characterisation). Every marketing page examined in this pass sells tooling to third parties. No page examined offers to trade with the reader, take the other side of a trade, hold client assets or manage client money. Where that changes — if it changes — §9 and §12 say so.

---

## Table of Contents

1. [The Overview and the Identity](#1-the-overview-and-the-identity) — 1.1 [The Short Answer — What SignalPlus Is and Is Not](#11-the-short-answer--what-signalplus-is-and-is-not) · 1.2 [The Namesake Disambiguation Note](#12-the-namesake-disambiguation-note) · 1.3 [The Key-Facts Table](#13-the-key-facts-table) · 1.4 [Why a Bank or a Fund Should Care](#14-why-a-bank-or-a-fund-should-care) · 1.5 [The Evidence Base at a Glance](#15-the-evidence-base-at-a-glance)
2. [The Company, the Founders and the Team](#2-the-company-the-founders-and-the-team) — 2.1 [The Founding Year: 2021](#21-the-founding-year-2021) · 2.2 [Chris Yu, Co-Founder and CEO](#22-chris-yu-co-founder-and-ceo) · 2.3 [James Shan, Co-Founder and COO](#23-james-shan-co-founder-and-coo) · 2.4 [The Wider Leadership Team](#24-the-wider-leadership-team) · 2.5 [Headquarters, Footprint and the Headcount Question](#25-headquarters-footprint-and-the-headcount-question) · 2.6 [The Corporate and Legal Entity Structure](#26-the-corporate-and-legal-entity-structure)
3. [The Funding and the Investors](#3-the-funding-and-the-investors) — 3.1 [The Round-by-Round Table](#31-the-round-by-round-table) · 3.2 [The Aggregation Risk on the US$23M Total](#32-the-aggregation-risk-on-the-us23m-total) · 3.3 [The Corrected 2026 Position — One Round or Two?](#33-the-corrected-2026-position--one-round-or-two) · 3.4 [The US$5 Billion Valuation Error](#34-the-us5-billion-valuation-error) · 3.5 [Strategic Versus Financial Investors](#35-strategic-versus-financial-investors) · 3.6 [What the Funding History Implies About Stage](#36-what-the-funding-history-implies-about-stage)
4. [The Product Suite](#4-the-product-suite) — 4.1 [The Dashboard, the Volatility Lab and the Workspace](#41-the-dashboard-the-volatility-lab-and-the-workspace) · 4.2 [Smart Dealing and the Execution Algorithms](#42-smart-dealing-and-the-execution-algorithms) · 4.3 [Risk Scenario and Stress Simulation](#43-risk-scenario-and-stress-simulation) · 4.4 [Dynamic Delta Hedge (DDH)](#44-dynamic-delta-hedge-ddh) · 4.5 [Automation — the Market-Making Robot and the Enterprise Tier](#45-automation--the-market-making-robot-and-the-enterprise-tier) · 4.6 [The Structured Product Pricer and Risk Management Engine](#46-the-structured-product-pricer-and-risk-management-engine) · 4.7 [API, Enterprise Deployment and the Front-to-Back Claim](#47-api-enterprise-deployment-and-the-front-to-back-claim) · 4.8 [The Product-Surface Table — Productised Versus Aspirational](#48-the-product-surface-table--productised-versus-aspirational)
5. [The Technology](#5-the-technology) — 5.1 [The Pricing and Volatility-Surface Question](#51-the-pricing-and-volatility-surface-question) · 5.2 [Greeks and Risk Analytics](#52-greeks-and-risk-analytics) · 5.3 [Market-Data Ingestion and Exchange Connectivity](#53-market-data-ingestion-and-exchange-connectivity) · 5.4 [Execution and Hedging Mechanics](#54-execution-and-hedging-mechanics) · 5.5 [Latency and Performance Claims, Each Attributed](#55-latency-and-performance-claims-each-attributed) · 5.6 [The AI and Machine-Learning Claims](#56-the-ai-and-machine-learning-claims) · 5.7 [What Is Simply Not Disclosed](#57-what-is-simply-not-disclosed) · 5.8 [The Technology-Facts Table](#58-the-technology-facts-table)
6. [The Business Model and the Customers](#6-the-business-model-and-the-customers) — 6.1 [How It Makes Money — What Is Actually Documented](#61-how-it-makes-money--what-is-actually-documented) · 6.2 [The Free Terminal as Distribution](#62-the-free-terminal-as-distribution) · 6.3 [The Target Segments](#63-the-target-segments) · 6.4 [The Exchange Integrations, Checked Against the Venue Side](#64-the-exchange-integrations-checked-against-the-venue-side) · 6.5 [The Customer Roster — Company-Reported and Not First-Party Confirmed](#65-the-customer-roster--company-reported-and-not-first-party-confirmed) · 6.6 [The Goldman Sachs Correction](#66-the-goldman-sachs-correction) · 6.7 [The Business-Model Table](#67-the-business-model-table)
7. [The Market It Sits In](#7-the-market-it-sits-in) — 7.1 [The Scale of the Crypto Options Market](#71-the-scale-of-the-crypto-options-market) · 7.2 [Venue Concentration and the Erosion Trend](#72-venue-concentration-and-the-erosion-trend) · 7.3 [The Institutional Share](#73-the-institutional-share) · 7.4 [The Onshoring Trend — IBIT Options, CME and the Regulated Alternative](#74-the-onshoring-trend--ibit-options-cme-and-the-regulated-alternative) · 7.5 [The Block and OTC Venues](#75-the-block-and-otc-venues) · 7.6 [The Clearing and Settlement Model, and Why It Matters](#76-the-clearing-and-settlement-model-and-why-it-matters) · 7.7 [The Statistical Caveat — Self-Reported and Unaudited](#77-the-statistical-caveat--self-reported-and-unaudited)
8. [The Competitive Landscape](#8-the-competitive-landscape) — 8.1 [The Direct Competitors — Analytics and Pricing Providers](#81-the-direct-competitors--analytics-and-pricing-providers) · 8.2 [The Adjacent Substitutes — Execution, OMS and Routing](#82-the-adjacent-substitutes--execution-oms-and-routing) · 8.3 [The Exchanges' Own Tooling](#83-the-exchanges-own-tooling) · 8.4 [The Build-It-Yourself Option](#84-the-build-it-yourself-option) · 8.5 [The Live-Auction Question](#85-the-live-auction-question) · 8.6 [The Positioning Table](#86-the-positioning-table)
9. [The Regulatory Perimeter](#9-the-regulatory-perimeter) — 9.1 [The Company's Own Licensing Status, or Its Absence](#91-the-companys-own-licensing-status-or-its-absence) · 9.2 [The Hong Kong SFC Virtual-Asset Regime](#92-the-hong-kong-sfc-virtual-asset-regime) · 9.3 [The May 2026 Convergence — Dealing, Custody, Advisory, Management](#93-the-may-2026-convergence--dealing-custody-advisory-management) · 9.4 [The Singapore MAS Angle](#94-the-singapore-mas-angle) · 9.5 [Software Versus Dealing — the Classification Analysis](#95-software-versus-dealing--the-classification-analysis) · 9.6 [Why the Analysis Matters Even With No Licence](#96-why-the-analysis-matters-even-with-no-licence)
10. [The Regulated-Institution Angle](#10-the-regulated-institution-angle) — 10.1 [The Vendor-Diligence View, and Where the General Framework Lives](#101-the-vendor-diligence-view-and-where-the-general-framework-lives) · 10.2 [The Uninspectable Model — Technology and Model Risk](#102-the-uninspectable-model--technology-and-model-risk) · 10.3 [The Valuation-Governance Question](#103-the-valuation-governance-question) · 10.4 [Counterparty, Operational and Concentration Risk](#104-counterparty-operational-and-concentration-risk) · 10.5 [Business Continuity and Key-Person Risk](#105-business-continuity-and-key-person-risk) · 10.6 [Confidential Position-Level Data Leaving the Firm](#106-confidential-position-level-data-leaving-the-firm) · 10.7 [Exit and Portability](#107-exit-and-portability)
11. [The Cymbal Bank Worked Example](#11-the-cymbal-bank-worked-example) — 11.1 [The Scenario](#111-the-scenario) · 11.2 [The Requirements](#112-the-requirements) · 11.3 [The Due Diligence Against Section 10](#113-the-due-diligence-against-section-10) · 11.4 [The Technology and Data Validation Plan](#114-the-technology-and-data-validation-plan) · 11.5 [The Regulatory Sign-Off Path](#115-the-regulatory-sign-off-path) · 11.6 [The Risk-Limit and Governance Design](#116-the-risk-limit-and-governance-design) · 11.7 [The Alternatives and the Cost-Benefit Comparison](#117-the-alternatives-and-the-cost-benefit-comparison) · 11.8 [The Recommendation](#118-the-recommendation) · 11.9 [What the Bank Cannot Establish](#119-what-the-bank-cannot-establish)
12. [The Claims Audit — Verified, Flagged, Rejected](#12-the-claims-audit--verified-flagged-rejected) — 12.1 [The Verified Claims (✅)](#121-the-verified-claims-) · 12.2 [The Flagged Claims (⚠)](#122-the-flagged-claims-) · 12.3 [The Rejected or Corrected Claims (❌)](#123-the-rejected-or-corrected-claims-)
13. [What Could Not Be Verified](#13-what-could-not-be-verified) — 13.1 [Unaudited Financials and True Scale](#131-unaudited-financials-and-true-scale) · 13.2 [True Customer Concentration](#132-true-customer-concentration) · 13.3 [Architecture Internals](#133-architecture-internals) · 13.4 [Performance Claims](#134-performance-claims) · 13.5 [Roster Confirmation](#135-roster-confirmation) · 13.6 [The Limitations of This Research Pass](#136-the-limitations-of-this-research-pass)
14. [The Glossary](#14-the-glossary)
15. [Cross-References and Further Reading](#15-cross-references-and-further-reading)
16. [The Closing Summary](#16-the-closing-summary)

---

## 1. The Overview and the Identity

### 1.1 The Short Answer — What SignalPlus Is and Is Not

**SignalPlus** (signalplus.com) is a Hong Kong-headquartered **technology company that builds institutional-grade software for trading crypto options and derivatives**. Its flagship artefact is a trading terminal and analytics dashboard that sits **on top of** third-party exchanges — not beside them, not instead of them. The company's own "About" wording, repeated across every release from 2022 to 2026, is a technology claim: "a venture-backed technology company building institutional-grade trading software focusing on digital assets" (SignalPlus, press release, 24 January 2025). **What it is:** a software vendor. Pricing, volatility analytics, risk attribution, scenario analysis, multi-leg execution algorithms, automated delta-hedging, an automated market-making robot, and a structured-product pricer/risk engine — sold as a free terminal plus API-licensed enterprise automation.

**What it is not:**
- **Not an exchange.** It has no matching engine of its own and lists no instruments. It connects to venues that do.
- **Not a custodian.** No wallet, no key custody, no asset safekeeping service appears anywhere in its product set. For that layer, see [Fireblocks](fireblocks_guide.md). ⚠ Note the one adjacent fact worth tracking: to trade on a client's behalf, the automation connects to exchanges using **client API keys** — a credential-custody question, not a client-asset-custody business (§10.6).
- **Not, on the public evidence, a principal market maker.** The market-making robot is licensed software that the client deploys under its own accounts and its own risk. AppWorks' investment note makes the distinction the point of the thesis. This is consistent across all sources examined. If SignalPlus ever trades its own book at scale, that would change its regulatory perimeter materially (§9.5).

**Founding, base and founder:** "Formed in 2021" is stated identically on the company's own company page and in its August 2022 Series A release; the January 2025 Series B release says "since its inception in 2021"; the June 2026 B1 release says "Headquartered in Hong Kong". Co-founders are **Chris Yu** (Co-Founder & CEO) and **James Shan** (Co-Founder & COO) — Shan's COO title is confirmed by the Series B release's investor quote ("We've been working with Chris and James for over a year", Jessica Liu, Partner of AppWorks) and by investor and third-party profiles (§2.3).

### 1.2 The Namesake Disambiguation Note

**Read this before searching for the company.** "SignalPlus" is a generic, weakly distinctive name, and it collides with unrelated usage in several fields — "signal plus noise" phrasing, product names in electronics and signal processing, clinical and diagnostics branding, and any number of small software products. This research pass attempted a targeted check for a competing company trading under the same name in financial services and **found no evidence of one, but also no definitive negative** (⚠ the search returned no usable results on that specific query; absence of evidence here is not evidence of absence). To be certain a reader is looking at the company this guide is about, anchor on these identifiers, not on the name alone:

| Identifier | Value | Status |
|---|---|---|
| Website | signalplus.com (terminal at t.signalplus.com) | ✅ company-published |
| Hong Kong entity | SIGNALPLUS HK Limited, incorporated 27 January 2022, BRN 73764584, CR no. 3125033 | ⚠ registry-aggregator derived; not queried directly at the Companies Registry in this pass |
| LinkedIn | linkedin.com/company/signalplushk | ✅ company-controlled |
| X / Twitter | @SignalPlus_Web3 | ✅ |
| Medium | medium.com/@signalplus_web3 | ✅ |

Note the entity-versus-business distinction a reader should not confuse: the **technology company** described here is a software vendor. It is a different species of business from a **Hong Kong SFC-licensed virtual asset trading platform** that might appear in a search for the same name, and it is a different company from any similarly named exchange. If a search result describes a venue, an exchange, a custodian or a licensed dealer, it is not this company.

### 1.3 The Key-Facts Table

| Item | Detail | Status and source |
|---|---|---|
| What it is | Software/trading-technology vendor for crypto options and derivatives (analytics, execution, automation, structured-product pricing) | ✅ company's own repeated self-description |
| What it is not | Not an exchange, not a custodian, not a principal market maker on the public evidence | ✅ consistent across all sources examined |
| Founded | 2021 | ✅ company statements (2022 and 2025 releases; company page) |
| Hong Kong entity incorporated | 27 January 2022 (SIGNALPLUS HK Limited) | ⚠ registry aggregators |
| Headquarters | Hong Kong | ✅ company statements |
| Founders | Chris Yu (Co-Founder & CEO); James Shan (Co-Founder & COO) | ✅ multiple first-party and investor sources |
| Exchange integrations claimed | Binance, Bybit, Deribit, OKX, Paradigm | ✅ claimed by company; ✅ Deribit integration confirmed on Deribit's own site (22 December 2022) |
| Named platform users (company-reported) | Cumberland, FalconX, Galaxy Digital | ⚠ reported by the company; not confirmed first-party |
| Funding announced | Seed (2022, lead disputed); US$12M Series A (Aug 2022); US$11M Series B (24 Jan 2025); US$40M Series B+ (20 May 2026, HashKey-led); US$50M Series B1 (1 Jun 2026) | ✅ as announcements; ⚠ B+ and B1 relationship unresolved (§3.3) |
| Headline valuation | US$500M post-money (1 June 2026) | ✅ company release; ❌ a "$5 billion" figure circulated by one outlet (§3.4) |
| Claimed 2025 volumes | US$160bn platform volume in Q4-2025; ~US$70bn Block-RFQ cleared via Deribit | ⚠ company-reported; unaudited |

### 1.4 Why a Bank or a Fund Should Care

Three reasons, in descending order of practical weight. **First, this is the most explicitly documented case of the "sell the shovels" layer in crypto derivatives.** The two questions a digital-asset desk actually faces are where to trade and what to trade on. Where to trade is dominated by a handful of venues; what to trade on — pricing, surface construction, risk attribution, hedging automation — is exactly what a small number of vendors now sell, and SignalPlus is one of the few with named institutional users and a public funding record. If a desk is going to buy rather than build, this is the class of vendor it will be buying from, and a bank that has an opinion on this vendor has an opinion on the whole category.

### 1.5 The Evidence Base at a Glance

Every factual claim in this guide carries one of three marks and, where relevant, a source-quality rating. The rating scheme used throughout:

- **A — primary and adversarial-free:** regulator publications, company-registry records, an exchange's own site, or an exchange's own newsroom.
- **B — first-party but self-interested:** the company's own press releases, website, blog and social content. Factually usable, unwittingly promotional, and unaudited.
- **C — reputable third-party:** established financial press, data providers whose business depends on accuracy, or an investor's own published note (self-interested in a different direction).
- **D — weak:** affiliate review sites, aggregator entries, AI-generated content farms, or anything that could not be traced to a named, responsible publisher.

| Evidence class | What it supports here | Rating |
|---|---|---|
| Company press releases 2022, 2023, 2025, 2026 (GlobeNewswire, PRNewswire) | founding, founders, funding, products, volume claims, roster | B |
| Deribit Insights, 22 December 2022 | the Deribit integration, with a Deribit executive quote | A |
| Deribit homepage (September 2026) | venues' own institutional feature set, 2025 volume and share self-claim | A (for what Deribit itself claims) |
| HashKey Group newsroom, 20 May 2026 | the Series B+ investment and the strategic partnership | B (counterparty-published) |
| CoinLaw options-market statistics, last updated 27 May 2026 | venue volumes, IBIT vs Deribit open interest, Coinbase-Deribit | C |
| Hong Kong Government press release, 26 May 2026 | VA dealing/custody/advisory/management regime direction | A |
| SFC virtual-assets pages and circular 23EC28 | VATP licensing under AMLO from 1 June 2023 | A |

---

## 2. The Company, the Founders and the Team

### 2.1 The Founding Year: 2021

The founding year is unusually well anchored for an early-stage private company, because it appears identically in sources of different kinds and six years apart:

- "Formed in 2021, SignalPlus is a technology company looking to develop an open, accessible technology platform to democratize and empower proficient crypto options trading" — SignalPlus, Series A release, 24 August 2022. ✅; "Since its inception in 2021, SignalPlus has been a forceful innovator" — SignalPlus, Series B release, 24 January 2025. ✅
- The company's own `/company` page states "Formed in 2021" in the first paragraph. ✅; The company's LinkedIn page states "Incorporated in 2021". ✅ (company-controlled)

### 2.2 Chris Yu, Co-Founder and CEO

Chris Yu is the company's public face and the author of nearly every first-party quote in the record: the December 2022 Deribit integration, the January 2025 Series B, the November 2025 institutional-customer release, and the June 2026 B1 close. **Background, as reported:** prior trading career in **macro trading, specifically FX and FX options**, at **Goldman Sachs** and **Morgan Stanley**. This claim appears in: the Asia Blockchain Summit speaker biography ("a rich background in macro trading from Goldman Sachs and Morgan Stanley. His expertise spans FX, FX Options, and interest rate products"); the AppWorks investment note ("Chris honed his capital markets expertise at Goldman Sachs and Morgan Stanley"); Dealroom's profile; and repeatedly in interview coverage (technode.global, 8 May 2025 — "As a veteran of the finance industry, with a background in macro trading from Goldman Sachs and Morgan Stanley"). ✅ **Multi-source consistent, but note the sourcing shape: every version traces to company-supplied biography.** No independent employment record was obtained in this pass. That does not make it false — it makes it ⚠ *company-supplied biography, multi-source-corroborated but not independently verified*.

**Entrepreneurial history:** AppWorks describes a path from a JP Morgan pitch competition with James Shan at Shanghai Jiao Tong University in 2007, into several ventures together — a hedge fund, a government technology platform, and an MCN business, the last of which produced a trade-sale exit — before "by 2021 his journey triangulated towards the area of crypto options" (AppWorks, 27 March 2025). ⚠ investor-note quality: specific, plausible, unverifiable.

### 2.3 James Shan, Co-Founder and COO

James Shan is the **Co-Founder and COO**. His existence and title are the most lightly documented part of the leadership story relative to his seniority, which is itself a diligence observation: he is quoted in none of the press releases examined in this pass. He appears in:

- the AppWorks note as the operations/product half of the founding pair ("Armed with his extensive knowledge of capital markets and confidence in James' immense product and technical expertise"); ✅; the January 2025 Series B release **indirectly**, via AppWorks' Jessica Liu: "We've been working with **Chris and James** for over a year"; ✅
- Dealroom: "co-founded by CEO Chris Yu and COO James Shan"; ⚠ aggregator; third-party and profile content repeating "Shan is a serial entrepreneur" and "Shan from mobile internet and enterprise SaaS". ⚠

**Background, as reported:** "James went on to build successful ventures in mobile internet and enterprise SaaS, culminating in co-founding a company that reached Series C backing from Temasek and Hillhouse Capital" (AppWorks, 27 March 2025). ⚠ The company is not named in the note and was **not identified in this pass** — so the single most impressive-sounding line in his biography (Series C from Temasek and Hillhouse) is currently unverifiable. A bank's diligence file should either obtain the company name and confirm it, or discount the line entirely.

### 2.4 The Wider Leadership Team

| Person | Role | Evidence | Assessment |
|---|---|---|---|
| Chris Yu | Co-Founder & CEO | Quoted in first-party releases 2022-2026; investor notes; interview coverage | ✅ well evidenced |
| James Shan | Co-Founder & COO | AppWorks note; indirect mention in the 2025 release; aggregator profiles | ⚠ evidenced but never quoted first-party in the releases examined |
| Augustine (surname not disclosed in the source) | Partner and CFO | Keynote text carried in a company release, 4 May 2023; describes "over 10 years as a macro rates trader at GS, and then nearly another decade afterwards as a CIO on the buyside" | ⚠ first-party, 2023, surname not given; current tenure unconfirmed |
| Jaewon Yu | Chief Commercial Officer, appointed 16 September 2025 | PRNewswire release, 16 September 2025: "more than two decades of experience in traditional finance global markets… including Head of FX Trading Asia" | ✅ appointment verified; tenure ongoing |

Two structural observations. **First:** the CCO hire (September 2025) and the Series B1 (June 2026) fit the classic revenue-scaling sequence — you hire commercial leadership, then raise growth capital. **Second:** every senior figure in the public record comes from **capital markets or enterprise software**, with a shared FX/rates/options lineage at the top. That is a coherent team for the product, and it is also a team with **no publicly visible chief risk officer, chief information security officer or head of compliance**. For a vendor whose clients are regulated desks, those absences are questions, not disqualifications (§10). The "over 70% STEM backgrounds" workforce claim (January 2025 release) is ⚠ company-reported, unaudited and undefined as to what counts as STEM.

### 2.5 Headquarters, Footprint and the Headcount Question

**Headquarters: Hong Kong.** Stated on the company's own "About SignalPlus" block — "Headquartered in Hong Kong, SignalPlus builds institutional-grade derivatives trading infrastructure for the converging capital markets" — and in the June 2026 release's dateline. ✅ **Headcount.** The company does not publish a headcount in any release examined. The only number found is **~29 employees**, from a third-party profile site of review-grade quality. ⚠ Do not treat it as fact — but do note that it is *plausible and consistent* with everything else: a company that raised US$12M in 2022 and US$11M in 2025, then took strategic capital in 2026, is a lean organisation. If the number is anywhere near right, a bank is dealing with a company of a few dozen people (§10.5).

**A vanity metric to exclude from the file.** LinkedIn follower and employee-tag counts are not headcount. A follower count measures marketing reach, and the employee figure on a LinkedIn company page splits only into "members who list the company" — it silently excludes contractors, includes departed staff and is routinely inflated by sales and community activity. Use it for nothing.

### 2.6 The Corporate and Legal Entity Structure

What the public record supports:

- **SIGNALPLUS HK Limited** — private company limited by shares, incorporated **27 January 2022**, company number **3125033**, BRN **73764584**, registered in **Central & Western district** (reported registered address at Haleson Building). ⚠ Four registry-aggregator sites agree on these identifiers across separate queries, which is reasonably reliable; the Hong Kong Companies Registry itself was not queried directly in this pass.
- **"SignalPlus Ltd."** — the legal name used in the August 2022 GlobeNewswire release, with a contact address at signalplus.com. ⚠ First-party usage, but no jurisdiction or registration number is given. Whether this is the Hong Kong entity referred to informally, a Singapore entity, or a third entity cannot be resolved from the public record examined.
- **No group structure is published.** No parent, no holding company, no subsidiary list, no Cayman/BVI holding layer is disclosed anywhere in the sources examined. For a company that has raised US$113M (if the 2026 rounds are sequential), an undisclosed offshore holding structure above the Hong Kong operating company would be entirely conventional — and its absence from the public record is simply an information gap, not evidence of anything.

---

## 3. The Funding and the Investors

### 3.1 The Round-by-Round Table

| Round | Announced | Amount | Lead | Other participants named | Status |
|---|---|---|---|---|---|
| Seed | 2021-2022 | Not separately disclosed; aggregate "over $12M raised in aggregate" with Series A | **GBV Capital (per the 2022 release)** — but **HashKey's 2026 release says its fund "led the seed round"** | — | ⚠ **LEAD DISPUTED — see note below** |
| Series A | 12 Aug 2022 (aggregators); release 24 Aug 2022 | US$12M (aggregate >$12M across Seed + A) | **HashKey Capital** | GBV Capital (seed lead), Stratified Capital (advisor), Incuba Alpha, Chiron Partners, **Paradigm Co**, SNZ Holding, Baboon.VC | ✅ company release |
| Series B (up-round) | 24 Jan 2025 | US$11M | **AppWorks and OKX Ventures** (co-leads) | Avenir Group; HashKey | ✅ company release |
| Series B+ | 20 May 2026 | **US$40M**, of which HashKey Group contributed US$20M | A fund managed by **HashKey Capital** | — (strategic partnership announced alongside) | ✅ HashKey Group newsroom |
| Series B1 | 1 Jun 2026 | **US$50M** at **US$500M post-money** | **HashKey Capital** | BlockBooster; AppWorks (follow-on) | ✅ company release |

**The seed-lead conflict is worth stating plainly.** The August 2022 release says "HashKey Capital (Series A lead), **GBV Capital (Seed round lead)**". HashKey's own May 2026 announcement says "The fund managed by HashKey **led the seed round** for SignalPlus. It also made additional investments in the additional investment rounds." Two issuers, two different seed leads. Plausible reconciliations exist (a seed extension with a different lead; HashKey leading an early tranche and GBV a subsequent one; a straightforward misstatement in one release). ⚠ **Not resolvable from the public record.** Flag it in any diligence file rather than picking a side.

### 3.2 The Aggregation Risk on the US$23M Total

The "**~US$23M total raised**" figure is the single most repeated number about this company and it is now wrong. It traces to a simple sum:

- US$12M Series A (August 2022) + US$11M Series B (January 2025) = **US$23M**.

That is exactly the arithmetic an aggregator performs, and it is what CypherHunter, Gate's fundraising calendar, and third-party review content report. The Gate entry makes the derivation explicit: "Following this transaction, SignalPlus's total funding has reached $23 million." A third-party review dated **April 2026** still states "The platform has raised $23M in total funding" — i.e. **after** the January 2025 round but with **no awareness of the May 2026 round**, let alone June. **The corrected position as of September 2026 depends on an unresolved question** (§3.3):

| Scenario | Sum | Total raised | Comment |
|---|---|---|---|
| Aggregator figure, stale | 12 + 11 | **US$23M** | ❌ accurate only to January 2025 |
| B+ and B1 are sequential rounds | 12 + 11 + 40 + 50 | **US$113M** | Upper bound on the disclosed figure ⚠ |
| B+ and B1 describe the same capital event | 12 + 11 + 50 | **US$73M** | Lower bound on the disclosed figure ⚠ |
| Series A and B were formally "up-rounds" of undisclosed size | — | — | ⚠ the 2022 release says "over $12M raised in aggregate" across Seed **and** Series A, so the Seed tranche is inside the $12M, not additive to it |

Note also that the Series B was explicitly framed as an "**up-round**" — a structure that means the round was raised inside the existing valuation envelope rather than as a new priced round. That is a company doing a bridge-style raise in a difficult market (January 2025), which is a useful signal about the company's trajectory between 2022 and 2025.

### 3.3 The Corrected 2026 Position — One Round or Two?

Two announcements, thirteen days apart, both HashKey-led: **20 May 2026 (HashKey Group newsroom):** "a fund managed by HashKey Capital, its asset management arm, has finalized an investment agreement with SignalPlus… the fund led SignalPlus's Series B+ funding round with a $40 million investment, of which HashKey Group contributed $20 million", alongside "a comprehensive strategic partnership". The release describes SignalPlus as "a leading institutional-grade infrastructure platform in the crypto derivatives space that focuses on option trading systems, automated market-making, and structured product solutions", says its services "are deeply integrated into the mainstream crypto options markets and it holds a leading market share in Asia", and carries a quote from **Dr Xiao Feng, Chairman and CEO of HashKey Group**. ✅ counterparty-published.

**1 June 2026 (company release, Hong Kong):** "the closing of its US$50M Series B1 funding round at a post-money valuation of US$500M… led by HashKey Capital, with follow-on participation from BlockBooster and AppWorks. This investment reflects institutional conviction in SignalPlus's evolution from a digital assets specialist into an institutional-grade, multi-asset trading infrastructure provider." **Goldman Sachs served as the sole financial advisor on the transaction.** ✅ company-published. **Are these the same round reported twice? The sources are ambiguous and this guide does not resolve it.** The considerations:

- **For "two tranches of one round":** different amounts (US$40M vs US$50M), different labels (B+ vs B1), thirteen days apart, same lead, and the B1 label itself implies a prior B-series tranche ("B1" reads as the first close of a B extension). Under this reading the US$50M is the round and the US$40M is HashKey's own slice, with US$20M coming from HashKey Group and US$20M from the fund it manages — which reconciles the two numbers almost exactly if BlockBooster and AppWorks supplied the remaining US$10M.
- **For "two sequential rounds":** a US$40M B+ followed by a US$50M B1 would put US$90M of HashKey-led capital into a company in two weeks at an undisclosed and disclosed valuation respectively — unusual but not impossible for a strategic investor consolidating a position alongside a commercial partnership.
- **What would settle it:** the company's own cap table or a founder statement. Neither is public. ⚠ **State the ambiguity; do not pick.**

**Valuation arithmetic, clearly labelled:** at a US$500M post-money valuation on a US$50M round, the round implies **10% dilution for the incoming investors** if the round is read as a single clean priced round (500 ÷ 50 = 10, computed this pass). That is a normal, non-aggressive Series B1 dilution. ⚠ It says nothing about the earlier rounds' pricing, the option pool, liquidation preferences or the actual ownership — none of which is public.

### 3.4 The US$5 Billion Valuation Error

One outlet reported the valuation as **US$5 billion**. This guide treats it as an error, and the evidence is unusually clean because the error is **internal to the article**:

- **en.theblockbeats.news/news/62584** (dated 2026-06-02) carries the headline "SignalPlus has completed its Series B funding round with a valuation of **$500 million**", a summary line stating "US$50 million Series B1 funding round", and then a body paragraph reading: "announced today that it has completed a $50 million Series B1 funding round, bringing its post-money valuation to **$5 billion**." ❌ **Headline US$500M, body US$5B. One of the two is wrong and the headline agrees with the company's own release.**; The same US$5B figure was repeated by at least one content-aggregation site ("SignalPlus secures $50M Series B1 funding at $5B valuation"), which is the ordinary propagation pattern for a typo: the aggregator copies the body, not the headline. ❌
- Against it: the company's own release is explicit and repeated — "US$50M Series B1 funding round at a post-money valuation of **US$500M**" — and reputable coverage (odaily, pulsealternative, a Binance Square summary, dailyaibrief) all carry **US$500M**. ✅

### 3.5 Strategic Versus Financial Investors

The investor list is **unusual for an early-stage software company, and the composition is the story**. Roughly in order of how much the identity matters:

| Investor | Type | Why the identity is material |
|---|---|---|
| **HashKey Capital** (and HashKey Group) | **Crypto-native strategic, and now a listed, licensed operator** | Led the Series A, participated in the Series B, led the B+ / B1. HashKey Holdings Limited trades as **3887.HK** and operates **licensed** digital-asset platforms. A licensed venue group is simultaneously an investor, a commercial partner and a potential competitor (§10.1) |
| **OKX Ventures** | **Crypto-native strategic — an exchange's venture arm** | Co-led the January 2025 round. OKX is one of the exchanges SignalPlus integrates. An exchange invested in the software layer that routes to exchanges, including its rivals |
| **AppWorks** | VC (Taiwan/SE Asia) | Led the 2025 round and followed on in 2026; published a detailed investment thesis; a conventional institutional VC |
| **BlockBooster** | VC / incubator hybrid | Followed on in the B1. Its CEO describes it as "a full-stack alternative asset manager pairing an incubation engine with capital management" |
| **Tencent** | Corporate strategic | Named in the company's own "About" block as a backer. ⚠ No round, date or amount is attributed to Tencent anywhere in the sources examined — treat as *named-backer claim only* |
| Avenir Group | Financial investor | Participated in the January 2025 round |
| GBV Capital, Stratified Capital, Incuba Alpha, Chiron Partners, SNZ Holding, Baboon.VC | Financial / early-stage specialists | Seed and Series A participants. Stratified Capital is described as an "advisor" as well as a participant |
| **Paradigm Co** | ⚠ **Ambiguous** | Named among Series A investors in the 2022 release as "Paradigm Co". Separately, "Paradigm" appears in the January 2025 release among **exchange integrations**. See the conflation note below |

**Why a venue investor is materially different from a VC.** A pure financial investor wants an exit; its interests and the company's are aligned. An **exchange** investor owns a venue that competes with the other venues the company integrates, and sits on the same side of the market structure that the vendor's neutrality depends on. An exchange group that is also a **licensed** operator and that has launched its own "digital asset derivatives and structured solutions business" (HashKey Capital, 27 August 2026) is a strategic shareholder in a position, in principle, to compete with the product it funds. That is not an accusation — it is the standard conflict a diligence file records and then tests: does the vendor's venue coverage stay genuinely vendor-neutral once a shareholder owns a venue? No public evidence was found either way. ⚠ **Unresolved.** **The Paradigm conflation — call it out explicitly.** Aggregator investor lists include "Paradigm" as a SignalPlus **investor**. The company's own 2022 release names "**Paradigm Co**" among the Series A participants, while its January 2025 release lists "**Paradigm**" among the **exchange integrations** ("Binance, Bybit, Deribit, OKX, and Paradigm"). These are two different claims about a similarly named entity, and the public record does not establish whether "Paradigm Co" the investor and "Paradigm" the venue are the same, related or unrelated organisations. ⚠ **Probable conflation between an investor name and a venue name.** Do not write "Paradigm is an investor and a partner" without checking; the honest formulation is: *the 2022 release names "Paradigm Co" as a Series A participant; the 2025 release names "Paradigm" as an exchange integration; the relationship between the two names is not established.*

### 3.6 What the Funding History Implies About Stage

Four readings, each defensible from the record: **1. This is a company that took strategic capital early and kept taking it.** HashKey's fund was in the seed (on its own account) and led the Series A, was in the Series B, and led both 2026 rounds. OKX Ventures co-led 2025. Tencent is claimed as a backer. That is a company that has consistently sold equity to industry participants rather than to generalist growth funds. Practically: **the strategic investors are load-bearing, and their interests must be modelled** (§3.5).

**2. The 2025 round was defensive in shape.** The January 2025 raise was US$11M, explicitly an "up-round", and the release's own framing ("Despite a challenging macro and operating environment") tells you the market was difficult. US$11M for a company that had raised US$12M in 2022 is a company that grew deliberately rather than a company that scaled on capital. **3. The 2026 step-change is real in absolute terms and unclear in structure.** Whether the 2026 capital is US$50M (one round) or US$90M (two), going from US$11M to a multiple of that within eighteen months, at a US$500M headline valuation, is a genuine inflection — and the sequence (strategic partnership with a listed venue group, then a priced round with an investment bank as sole financial advisor) reads like a company being groomed for a larger event. ⚠ Any inference about an eventual IPO or strategic sale is speculation and is labelled as such.

**4. The valuation multiple cannot be sanity-checked.** No revenue figure has ever been disclosed — not ARR, not bookings, not gross margin, not a customer count. A US$500M post-money valuation on undisclosed revenue is not assessable as rich or cheap. ⚠ **This is the single largest analytical hole in the company's public profile**, and §13 records it as such.

---

## 4. The Product Suite

The product surface is documented in three places: the company's homepage (the six-module dashboard carousel plus the automation and structured-product sections), the January 2025 and June 2026 releases (the enterprise framing), and the August 2022 Series A release (the original architecture intent: "a technology stack that centers around a Wall-Street caliber options trading dashboard, with a full suite of modular and powerful toolkits wrapped around a user-centric interface", plus "open API services [that] will allow for user subscriptions into an advanced suite of functions including automated option market making, dynamic delta hedging, and structured products pricing"). The 2022 intent and the 2026 surface match each other closely — a seven-year-consistent roadmap is mildly reassuring as evidence that these are real products rather than repositioning, though it is first-party on both ends. Every item below is **B-rated (company-published)** unless stated otherwise.

### 4.1 The Dashboard, the Volatility Lab and the Workspace

The homepage presents six modules in a rotating carousel, and they constitute the free terminal:

| Module | What the company says | Independent evidence |
|---|---|---|
| **Live Risk / PnL** | "Real-time inventory, intraday risk exposure, and detailed PL breakdown by Greeks across expiries and strikes"; PL attribution, strike analysis, theta analysis | ⚠ company-published; a third-party review (April 2026, D-rated) repeats the description having evidently worked through the UI, which is weak corroboration that the screens exist and are populated |
| **Volatility Lab** | "Visualized volatility profile including Term Structure, Model Volatility Smile, Model Volatility Surface, 7D RV Momentum" | ⚠ same; the vocabulary is standard options-desk vocabulary, which is consistent with the founders' stated backgrounds |
| **Smart Dealing** | Multi-leg order execution using embedded algos to minimise slippage; "auto-calculate gamma- and vega-neutral offsets with a few simple clicks" | ⚠ company-published |
| **Risk Scenario** | "Powerful simulation tools and scenario analyses to stress-test your PL and exposures well ahead of time" | ⚠ company-published |
| **Dynamic Delta Hedge (DDH)** | "Industry-leading, highly customizable DDH robot… across all market conditions"; "real-time trade notifications via Telegram confirmations" | ⚠ company-published; the Telegram confirmation detail is credible product specificity |
| **Customisable workspace** | "Single workspace view to see all your inventory, order history, and real-time market data in one place" | ⚠ company-published |

Two things are notable. **First, the modules are exactly what a sell-side options desk would recognise** — Greeks by expiry and strike, P&L attribution, term structure, model smile and surface, realised-vol momentum, gamma/vega-neutral multi-leg offsets. The product is a reconstruction of a bank options desk's tooling, which is the entire commercial premise. **Second, the "model" qualifier is doing real work in "Model Volatility Smile" and "Model Volatility Surface."** The company is not claiming to show you the market's surface; it is claiming to show you **its own constructed surface**. That single word is the crux of §10.2 and §10.3 — a bank adopting this platform is adopting a model, not just a screen.

### 4.2 Smart Dealing and the Execution Algorithms

**Company claim:** execute multi-legged orders "using the dashboard's embedded algorithms to power better executions while minimizing slippage", with automatic gamma- and vega-neutral offset calculation. The January 2025 release frames the whole suite as "a full suite of pricing, analytics, and execution algos with multi-vendor and DMA access, perfect for high-touch trading strategies with a zero cost commitment".

### 4.3 Risk Scenario and Stress Simulation

**Company claim:** "Powerful simulation tools and scenario analyses to stress-test your PL and exposures well ahead of time" — i.e. what-if scenario analysis beyond instantaneous Greeks: spot shocks, vol shocks, time decay and combinations. The June 2026 release elevates this to a differentiator: the terminal "delivers unparalleled risk attribution, position analytics, and comprehensive 'what-if' scenario analysis **exceeding the standard of current market alternatives**." ⚠ **Comparatively unsubstantiated marketing.** "Unparalleled" and "exceeding the standard of alternatives" are claims about competitors that no source examined tests. The functionality is plausible and the underlying technique is standard; the comparative claim is not evidenced and should be struck from any internal summary.

### 4.4 Dynamic Delta Hedge (DDH)

**Company claim:** a "highly customizable DDH robot to reliably automate your delta hedging needs across all market conditions", with real-time trade notifications by Telegram. **What it actually is, mechanically:** a service that, given a book, continuously computes net delta and sends orders to the venue(s) to offset it, on a schedule or trigger you configure. This is the most mundane and arguably the most useful product on the list — delta hedging is a 24/7 operational burden in a market that never closes, and it is exactly the burden a two-person desk cannot carry.

**Why it is also the highest-risk product on the list, from a bank's perspective:** to hedge on your behalf, the robot must have **trading-enabled API credentials to your exchange account**. That is a live order-execution capability delegated to a third party's code. Every control that follows from that — key scoping, withdrawal prohibition, IP allow-listing, order-size and rate limits, a client-side kill switch, credential rotation, sub-account separation, an immutable audit trail of who or what sent each order — is the substance of the diligence (§10.6, §11.3).

### 4.5 Automation — the Market-Making Robot and the Enterprise Tier

**Company claim, from the homepage:** "our comprehensive, battle-tested, AI-powered trading robot to fully automate your crypto options market making and delta hedging needs reliably regardless of market conditions", with:

- "100% automated protocol with full hedging capabilities"; "24/7 uptime trading across major venues with minimal human oversight";
- "**200+ user-defined and bespoke parameters**"; "automated and fully-customizable market making services via API subscriptions across all supported exchanges".

**This is the revenue product.** Where the dashboard is given away, the automation tier is the thing that is licensed — the January 2025 release is explicit that the automation suite is "available to jump-start and scale a market-making operation right from its inception", and the June 2026 release says the risk-management and pricing infrastructure "enables professional clients and centralized exchanges to operate automated risk management and structured product pricing around the clock".

### 4.6 The Structured Product Pricer and Risk Management Engine

**Company claim:** "a comprehensive structured product solution [that] provides issuers with an effective and efficient way to manage risk through a fully automated hedging protocol", comprising a "Wall Street-grade pricer, expandable to all popular TradFi structured products"; an "in-house risk management engine [that] ensures best risk transfer pricing"; "full model & pricing transparency that ensure[s] maximum margin capture"; and "bespoke product design and terms". **Why this is the most strategically interesting product in the suite:** the June 2026 release says the risk-management and pricing infrastructure serves "professional clients **and centralized exchanges**" — i.e. this is sold to **venue operators** who want to list structured or exotic products but lack the pricing and hedging capability in-house. AppWorks says the same thing less guardedly: "they've built automated risk management for exotic structured products, playing a crucial role in helping exchanges expand their options offerings." An exchange licensing a third party's pricer and hedging engine to launch products is a **significantly deeper dependency** than a desk licensing a dashboard, and it tells you the company's actual moat: not the UI, but the pricing and hedging engine underneath.

**"Full model & pricing transparency"** is the most useful sentence on the whole homepage, and it deserves this response from a buyer: **ask what it means.** Transparency to whom, in what form, under what documentation — a model specification, a calibration report, a source-code escrow, or a screen that shows a number without its inputs? The phrase is unquantified (⚠) and, if the answer is "the client sees our price", it is not transparency; it is an output.

### 4.7 API, Enterprise Deployment and the Front-to-Back Claim

**API:** the August 2022 release describes "the platform's open API services [that] will allow for user subscriptions into an advanced suite of functions including automated option market making, dynamic delta hedging, and structured products pricing" — i.e. a subscription model per function. The homepage describes "API subscriptions across all supported exchanges". ✅ company-published, consistent across four years. **The "front-to-back" claim.** The company's LinkedIn description says the mission is "to offer an integrated digital assets options platform with **full front to back-end services**" (⚠ company-controlled, undated). A "front-to-back" claim in a sell-side context normally means pricing/quoting through to confirmation, settlement, clearing and books-and-records. **Here the accurate meaning is narrower**, because the company does not clear, settle or hold anything: front-to-back in this context plausibly means from market data and pricing through to execution and hedging, all inside the tool, with settlement remaining at the venue. ⚠ **The phrase overstates and should be read narrowly.** Do not let "front-to-back" into a bank document without the scope written down, because a reader will assume post-trade processing that is not being provided by this vendor.

### 4.8 The Product-Surface Table — Productised Versus Aspirational

| Product | Company claim | Independent evidence | Assessment |
|---|---|---|---|
| Trading Terminal / Dashboard (free) | Pricing, analytics, liquidity aggregation, multi-vendor connectivity, "all free of charge" | Reviews describe the UI as functional; the free positioning is repeated across four years of company material | ✅ **Productised, live, free** |
| Smart Dealing | Multi-leg algos, gamma/vega-neutral offsets, slippage minimisation | None; execution-quality claims untested | ⚠ **Productised; performance unproven** |
| Risk Scenario | Scenario and stress simulation | None; "exceeding the standard of market alternatives" unsubstantiated | ⚠ **Productised; comparative claim unsupported** |
| Dynamic Delta Hedge | Configurable hedging robot, Telegram notifications | None | ⚠ **Productised; behaviour untested independently** |
| Market-making robot / automation tier | 100% automated, 24/7, 200+ parameters, API subscription | None | ⚠ **Productised (it is the revenue product); "battle-tested" and "AI-powered" unevidenced** |
| Structured Product Pricer & Risk Engine | Pricer, hedging protocol, bespoke design, model transparency | None | ⚠ **Productised for exchanges and issuers per company claims; not independently verified** |
| Trading Compass (AI news and sentiment) | Named in third-party profiles as a 2024 module | ⚠ **Not found in any company release or on the homepage examined** | ⚠ **Existence uncertain from first-party sources** |
| QuantLab | "Proprietary engine" behind volatility analysis, back-testing and pricing | Recurring first-party mentions (2022, 2025, 2026) | ⚠ **Named engine, no specification** |
| SignalPlus 2.0 (agentic AI) | Announced 1 June 2026; volatility-market-structure analysis, strategy back-testing, "actionable trading modules" | ⚠ announced only; "will be releasing" | ⚠ **Announced, not shipped as of September 2026** |
| Structured commodity products | Announced 1 June 2026 as a suite expansion | ⚠ announced only | ⚠ **Announced, not evidenced** |
| Coincall integration | Appears in third-party profile and review content | ⚠ not in company releases examined | ⚠ **Unconfirmed from first-party sources** |
| Claimed certifications (ISO, AICPA SOC, Leviathan MASE, Astra A+, GDPR) | Logos displayed under "Industry Security Certifications" | ⚠ **logos only — no certificate numbers, scope, auditor or dates** | ⚠ **Unverifiable as displayed** |

**Bottom line on the product suite:** the core (terminal, volatility lab, risk/PnL, DDH, automation) is **consistent across four years of first-party material and matches what professional options traders actually need**, which is the strongest thing that can be said for it. The comparative, performance, AI and certification claims are **marketing**, and each should be converted into a written question before any commercial conversation advances.

---

## 5. The Technology

### 5.1 The Pricing and Volatility-Surface Question

This is the heart of the product and the largest disclosure gap in the company's public profile. The company states what it produces — "Model Volatility Smile", "Model Volatility Surface", term structure, 7D realised-vol momentum, and a "Wall Street-grade pricer" — and says nothing about how. ⚠ **The public record does not disclose:**

- the model family (parametric surface fit such as SVI/SABR, local volatility, stochastic volatility, or a practical interpolation-plus-arbitrage-repair approach); the **calibration** method, the weightings applied to illiquid strikes, or how the surface is kept **arbitrage-free** (butterfly and calendar arbitrage constraints);
- **wing extrapolation** behaviour — how the surface is extended beyond traded strikes, which is where bad surfaces do their damage in a risk report; the treatment of venue fragmentation (V1: do Binance, Bybit, Deribit and OKX quotes go into one surface or several — and if several, which one is used for risk?);
- the **snapshot and revision policy**: what is stored, how often, whether a price shown yesterday can be reproduced today.

### 5.2 Greeks and Risk Analytics

The dashboard presents "detailed PL breakdown by Greeks across expiries and strikes" and offers gamma- and vega-neutral offsets. ⚠ Not disclosed: whether the Greeks are analytic or finite-difference; whether they are computed off the same surface used for pricing or a separate one; whether second-order and cross-sensitivities (gamma, vanna, volga) are exposed; the theta convention (calendar vs trading day); and whether risk is aggregated in delta-equivalent units, USD or contract terms. This is ordinary technology that has ordinary answers — but a bank validating a third-party risk number needs the answers, because **a P&L attribution that cannot be reconciled to a second, independent implementation is not a control** (§10.3).

### 5.3 Market-Data Ingestion and Exchange Connectivity

The marketing claim — "multi-vendor and DMA access" and "liquidity aggregation" — implies ingestion of public order-book and trade data from each venue plus authenticated private channels for positions and orders. ⚠ Not disclosed: feed redundancy, venue failover behaviour, message-ordering handling, tick storage and replay, reference data (expiries, contract multipliers, settlement conventions), and how a venue outage surfaces in the client's risk view. For a 24/7 hedging robot these are the questions that decide whether it behaves correctly on the worst day.

### 5.4 Execution and Hedging Mechanics

The observable claims are multi-leg order execution with embedded algos, automatic gamma-/vega-neutral offset calculation, and a DDH robot with Telegram notifications. The mechanically necessary fact the company does not foreground but which follows from its own description: **the automation operates the client's exchange accounts through API credentials** ("bind it to their Deribit account API", per the Deribit integration announcement). ⚠ Not disclosed: order types used, child-order logic, rate-limit behaviour, error and partial-fill handling, whether hedging can be disabled by the client instantly, and what happens when the robot and a human trader disagree. §10.6 treats the credential custody question as the central operational risk of the product.

### 5.5 Latency and Performance Claims, Each Attributed

| Claim | Source and quality | Assessment |
|---|---|---|
| US$160bn platform volume in Q4-2025 | Company release, 1 Jun 2026 (B) | ⚠ unaudited, self-defined "platform volume" |
| ~US$70bn of Block-RFQ transactions cleared via Deribit alone | Company release, 1 Jun 2026 (B) | ⚠ unaudited; venue-side confirmation not found in this pass |
| Terminal volumes grew at **74% quarterly CAGR since 2023** | Company release, 1 Jun 2026 (B) | ⚠ **arithmetically implausible if read literally** — see below |
| "Market share multiply tenfold over the past 18 months" | Company release, 1 Jun 2026 (B) | ⚠ no denominator, no methodology, unaudited |
| "since 2016" / "85%+ market share" / "$1,875B 2025 volume" | Deribit's own site (A for what Deribit claims) | ⚠ venue self-report, and in tension with third-party data (§7.2) |

**The 74% quarterly CAGR deserves its own note**, because it is the single most aggressive number in the company's material and it is cheap to test. A 74% quarterly growth rate compounded over eight quarters is **84×**; over twelve quarters **770×**; over sixteen quarters **7,060×** (computed this pass). For the claim to be literally true from a 2023 start, terminal volume in Q4-2025 would have to be hundreds of times a Q1-2023 base that was itself small — which is possible from a low base, and impossible to check because **no absolute quarterly series is published and "terminal volumes" is never defined** (notional? premium? contract count? block-inclusive or not?). ⚠ **Record the claim; record that it cannot be reconciled; do not repeat it as a growth rate in any bank document.**

### 5.6 The AI and Machine-Learning Claims

The AI claims are: the automation suite described as an "**AI-powered** trading robot" (homepage); "Trading Compass" — "AI-powered live news aggregation, sentiment analysis and smart content labelling" (⚠ named only in third-party profile content, not found on the homepage or in any release examined); **QuantLab** — "the company's proprietary QuantLab engine" (first-party, recurring 2022-2026); and **SignalPlus 2.0**, a "platform upgrade integrating **agentic AI** into its existing product workflow… users will soon be able to analyze volatility market structure, perform strategy back-testing, and create actionable trading modules" (company release, 1 Jun 2026). **The position this guide takes: these are unproven marketing unless evidenced.** ⚠ No model card, no methodology, no benchmark, no description of training data, no statement of what is deterministic versus learned, no accuracy or error-rate disclosure, and no independent evaluation was found for any of them. The B1 release itself uses the future tense for the flagship AI feature ("**will be releasing**", "users **will soon** be able to"), which is itself the clearest available evidence that the agentic capability is **announced, not shipped** as of September 2026.

### 5.7 What Is Simply Not Disclosed

Rather than paraphrasing generic marketing, this guide states plainly that the following are **not disclosed anywhere in the sources examined** (⚠ all items):

- **Deployment and hosting model** — multi-tenant SaaS vs single-tenant vs on-premise or VPC; data residency; environment separation.
- **Architecture** — services, data stores, message flow, scaling model; whether the platform is a monolith or a service mesh (see [Trading System Software Architecture](../technology/trading_system_software_architecture_guide.md) for the vocabulary, not re-derived here).
- **SLA and resilience commitments** — uptime target, RTO, RPO, incident notification, maintenance windows, DR design, for a system whose marketing promise is "24/7 uptime".
- **The pricing model and calibration methodology** (§5.1).
- **Security certification scope** — the logos name ISO, AICPA SOC, Leviathan MASE, Astra A+ and GDPR with no certificate number, no scope statement, no auditor and no date (§4.8).
- **Incident history** — no breach, outage or model-failure disclosure was found. ⚠ Absence of disclosure is not absence of incidents.
- **Financials** — no revenue, margin, customer-count or retention data (§13.1).
- **Headcount and organisation** (§2.5).

### 5.8 The Technology-Facts Table

| Domain | Position | Evidence status |
|---|---|---|
| Pricing engine | Exists and is central to the pitch; used for options pricing, structured-product pricing and automation | ✅ existence (first-party, recurring); ❌ methodology undisclosed |
| Volatility surface | Vendor-constructed "model" surface, term structure, smile | ✅ existence; ❌ model family and calibration undisclosed |
| Connectivity | Multi-venue; Deribit confirmed venue-side; others company-claimed | ✅ Deribit; ⚠ others |
| Execution algos | Present in the terminal | ✅ existence; ⚠ no TCA or fill-quality evidence |
| Hedging automation | DDH robot; market-making robot; client API credentials | ✅ existence; ⚠ control design undisclosed |
| SLA / BC / DR | Not disclosed | ⚠ |
| AI/ML | Claimed across the automation suite, QuantLab and SignalPlus 2.0 | ❌ unevidenced |
| Latency claims | None published | ✅ (absence noted as a positive) |

---

## 6. The Business Model and the Customers

### 6.1 How It Makes Money — What Is Actually Documented

Three revenue mechanisms appear in first-party material, and nothing else does:

- **API subscriptions for the automation tier.** "Our platform provides automated and fully-customizable market making services via **API subscriptions** across all supported exchanges" (homepage). The August 2022 release described the same mechanism as the platform's original intent: "the platform's open API services will allow for **user subscriptions** into an advanced suite of functions including automated option market making, dynamic delta hedging, and structured products pricing."; **Enterprise licensing of the risk-management and pricing infrastructure**, sold to "professional clients **and centralized exchanges**" to "operate automated risk management and structured product pricing around the clock" (company release, 1 Jun 2026) — the structured-product pricer and risk engine (§4.6).
- **The terminal, given away free.** "Our flagship crypto trading dashboard offers a full suite of pricing, analytics, and execution algos with multi-vendor and DMA access, perfect for high-touch trading strategies with a **zero cost commitment**" (company releases, Jan 2025 and Nov 2025), and "the product is offered **completely free of charge** to all users, no strings attached" (CFO keynote, April 2023).

⚠ **Not documented anywhere:** actual prices, annual contract value, minimum commitment, contract term, per-seat vs per-volume vs per-venue charging, gross margin, whether exchanges or their clients pay for the structured-product engine, or the split between the three mechanisms. A third-party profile refers to an enterprise tier by the name "Core+" (⚠ D-rated, unconfirmed first-party). **A revenue model that is documented only as "subscriptions plus enterprise licences, price undisclosed" is a legitimate business model and an unanalysable one.**

### 6.2 The Free Terminal as Distribution

The free-terminal strategy is explicit and consistent across four years, and the company's own explanation is a distribution argument: "we are big believers of Web3's focus on user empowerment, and this is our way of contributing to the crypto ecosystem, and levelling the playing field for the average folk" (CFO keynote text, 2023). The commercial logic underneath is standard and sound — **the terminal is the land, the automation is the expand** — and the Deribit integration notice shows the funnel in operation: existing Deribit users bind an API key, and "institutional users at Deribit can email bd@signalplus.com… and request the institutional version of the SignalPlus platform" (Deribit Insights, 22 December 2022).

### 6.3 The Target Segments

The company's own "About" block names them: "hedge funds, market makers, proprietary trading desks, and asset managers across digital **and traditional** financial markets" (1 Jun 2026). Earlier material adds "institutional and retail crypto traders" (Jan 2025 release) — the retail mention reflects the free-terminal ambition, not the revenue model. The behavioural segmentation that actually matters: **the buyer is a firm that wants to run an options market-making or volatility book and does not have a full bank-style options stack.** That describes most crypto-native market makers and essentially every newly formed digital-asset desk at a traditional institution — which is precisely why the vendor is interesting to a bank (§1.4).

### 6.4 The Exchange Integrations, Checked Against the Venue Side

The company claims integration across "Binance, Bybit, Deribit, OKX, and Paradigm" (24 Jan 2025 release). Checked against the venue side:

| Venue | Company claim | Venue-side evidence | Status |
|---|---|---|---|
| **Deribit** | Integrated | **Deribit's own newsroom, 22 December 2022**, "SignalPlus to Expand Offerings through Deribit Integration", with a quote from **Luuk Strijers, then Chief Commercial Officer of Deribit** | ✅ **Confirmed by the venue** |
| Binance | Integrated | None found in this pass | ⚠ company-claimed only |
| Bybit | Integrated | None found in this pass | ⚠ company-claimed only |
| OKX | Integrated | None found in this pass (OKX **Ventures** invested in the company, which is investment, not integration confirmation) | ⚠ company-claimed only |
| Paradigm | Integrated | None found; and the name is conflated with a Series A investor, "Paradigm Co" (§3.5) | ⚠ company-claimed and name-ambiguous |

**One confirmed venue integration out of five claimed** is the honest summary. That is not a criticism of the others — a venue has no obligation to announce a client's integration, and the Deribit one was announced because it was jointly promoted. But a diligence file should record which claims are venue-confirmed and which are vendor-asserted, because they are not the same evidence.

### 6.5 The Customer Roster — Company-Reported and Not First-Party Confirmed

**The handling rule for this guide, applied throughout:** a roster claim resting only on the company's own press release is **COMPANY-REPORTED, not verified.** It is marked ⚠ with the wording *"reported by the company; not confirmed first-party"* in every place it appears here. What the company says, and how:

- **1 June 2026 release:** "the SignalPlus platform now servicing the industry's largest options players including **Cumberland, FalconX, and Galaxy Digital**." ⚠ reported by the company; not confirmed first-party.
- **20 November 2025 release** ("SignalPlus: Enabling Institutional Excellence in Digital Asset Trading"): "Servicing a roster of some of the industry's biggest players, including **Cumberland, FalconX, Galaxy, and Goldman Sachs**", carried with **attributed quotes** from **Roman Makarov, Head of Crypto Options Trading, Cumberland DRW** ("We have partnered with SignalPlus over the past year, and they have been consistently reliable in meeting our requirements"); **Jason Urban, Global Head of Trading at Galaxy**; **Matt Long, General Manager of APAC, Middle East & Europe at FalconX**; and **Max Minton, Head of Digital Assets in Asia Pacific, Goldman Sachs**. ⚠ reported by the company; not confirmed first-party.
- **A different, weaker register:** the homepage's "Trusted by Industry Leaders" paragraph names **no** clients at all, and the November 2025 release's "About" block says only that the company "partners with the industry's leading exchanges and trading institutions". The **named** roster appears only in press releases. That pattern — anonymous on the product site, specific in the press release — is the ordinary pattern for a company whose clients prefer not to be publicised, and it is *not* an indication of falsity. It is, however, exactly why the claims are flagged rather than accepted.

**On the strength of an attributed quote.** A named executive's quote is stronger evidence than a logo or a list, because it is falsifiable by the named person and because reputational cost attaches to it. It is still **not first-party confirmation**, because the quote is published inside a document the vendor wrote and distributed, and this research pass did not obtain any corresponding statement from Cumberland, FalconX or Galaxy Digital. The correct treatment: record the quote as a company-distributed testimonial from a named individual, and if the relationship is material to a decision, request confirmation directly from the counterparty.

### 6.6 The Goldman Sachs Correction

**The dispatcher's brief for this guide implied that Goldman Sachs was among the customer roster. It was, but not in the round where you might expect it — and the distinction matters, so both facts are stated here separately.**

- **On the Series B1 round (announced 1 June 2026): Goldman Sachs served as the SOLE FINANCIAL ADVISOR on the transaction.** That is the company's own wording and it is unambiguous: "Goldman Sachs served as the sole financial advisor on the transaction." ✅ An advisory mandate is **not** a customer relationship, not an investment and not a validation of the product. A financial advisor's role on a private round is corporate finance work — placement and process — and it says nothing about whether the advisor uses the platform.
- **In a separate company release dated 20 November 2025**, the company names Goldman Sachs in its user roster and carries a quote from a Goldman Sachs employee: "We leverage SignalPlus' analytics as we use tools that can help inform us in navigating an emerging and growing asset class" — **Max Minton, Head of Digital Assets in Asia Pacific, Goldman Sachs.** ⚠ **This is a company-distributed statement; it is not independently confirmed**, and no first-party Goldman Sachs publication confirming platform usage was found in this pass.

**How to write it in a bank document:** *Goldman Sachs was the sole financial advisor on SignalPlus's Series B1 round (June 2026). SignalPlus's own November 2025 release additionally names Goldman Sachs in its user roster with an attributed quote; that roster claim is company-reported and not first-party confirmed.* ❌ **Do not write "Goldman Sachs is a SignalPlus customer"** and ❌ **do not write "Goldman Sachs invested in SignalPlus"** — neither appears in the record. The presence of a bulge-bracket name as advisor is sometimes read internally as a quality signal; here it is evidence about the *transaction*, not about the *product*.

### 6.7 The Business-Model Table

| Dimension | Position | Status |
|---|---|---|
| Revenue mechanisms | API subscriptions (automation); enterprise licensing of pricing/risk infrastructure to clients and exchanges; free terminal | ✅ documented |
| Pricing | Not disclosed | ⚠ |
| Customer count, ACV, retention | Not disclosed | ⚠ |
| Revenue and margin | Never disclosed | ⚠ |
| Target segments | Hedge funds, market makers, prop desks, asset managers, exchanges | ✅ company-stated |
| Named users | Cumberland, FalconX, Galaxy Digital (+ Goldman Sachs in a separate release) | ⚠ reported by the company; not confirmed first-party |
| Confirmed venue integration | Deribit | ✅ venue-side |
| Conflict exposure | Strategic investors include an exchange's venture arm and a listed licensed venue group that launched a derivatives business in August 2026 | ✅ facts verified; ⚠ implications untested |

---

## 7. The Market It Sits In

### 7.1 The Scale of the Crypto Options Market

The most defensible recent figures, with dates and provenance:

| Metric | Figure | Source, date, quality |
|---|---|---|
| BTC + ETH options volume, top five venues, **H1 2026** | **US$864.6bn** | CoinGlass data reported 4 August 2026 (C) |
| Deribit options volume, H1 2026 | **US$425.9bn** (49.3% share) | Same (C) |
| Deribit monthly options share | **56.3% in January 2026 falling to 41.8% in June 2026** | Same (C) |
| Bybit options share, H1 2026 | 22.3% overall; **38% of ETH options** vs Deribit's 29% | Same (C) |
| Deribit total traded volume, **2025** | **~US$1,875bn** (options + futures), up from US$1.185tn in 2024 | Deribit's own site (A for its claim); CoinLaw (C) |
| Options open interest, April 2026: IBIT vs Deribit | **IBIT US$27.61bn vs Deribit US$26.9bn** — IBIT overtook Deribit for the first time | CoinLaw (C) |
| CME crypto futures and options, Q2 2026 | ADV 250K contracts, +32% YoY; average daily OI 216K contracts; ~US$459.2bn notional in the quarter | CME newsletter (A for CME's own data); the notional figure via secondary coverage (C) |

### 7.2 Venue Concentration and the Erosion Trend

Crypto options are the most concentrated major market in digital assets. Deribit has led since 2016 and describes itself as having "~85% market share in BTC and ETH options" and "85%+" on the same page (Deribit's own site, September 2026 — ⚠ venue self-report). The third-party picture is materially different and much more useful: **49.3% for H1 2026, down from 56.3% in January to 41.8% in June** (CoinGlass). ⚠ **The gap between the venue's own "85%" and the third-party "49.3%" is a live illustration of why crypto market statistics must be dated and sourced rather than quoted** (§7.7). The two numbers may also measure different baskets (BTC+ETH only vs all instruments), which is precisely the kind of definitional ambiguity that makes a figure unusable without its methodology. **One structural fact that must not be lost:** on **14 August 2025, Coinbase closed its acquisition of Deribit for US$2.9bn** (US$700m cash plus 11 million Coinbase Class A shares) — per CoinLaw (C). ✅ That places the largest offshore crypto options venue inside a **US-listed, regulated** group, which changes the venue's regulatory posture and, over time, its product and client composition. It also means the venue SignalPlus routes roughly US$70bn of quarterly Block-RFQ through (company claim) is now owned by a listed competitor-adjacent entity.

### 7.3 The Institutional Share

CoinLaw reports that "Deribit derives **80% of its volume and open interest from institutional participants**, a share stable through Q1 2026", and that "**Paradigm's institutional network routes 33 to 36% of Deribit's volume** from quantitative funds and trading desks". ⚠ **Both are secondary, single-source and undated as to methodology** — treat them as directional. If either is close to right, the implication for this guide is direct: the crypto options market is **primarily an institutional market already**, which is why a vendor selling institutional-grade tooling has a market at all, and why the buyer of such tooling will typically be a firm with its own risk function that will want to validate what it buys.

### 7.4 The Onshoring Trend — IBIT Options, CME and the Regulated Alternative

The most important market-structure development since 2025 is that **regulated, onshore options on crypto have become large enough to compete with the offshore native venues.** Per CoinLaw's dated updates (all C-quality secondary):

- **April 2026:** IBIT options open interest reached **US$27.61bn, overtaking Deribit's US$26.9bn** for the first time — ending the offshore venue's multi-year lead in *Bitcoin* options open interest.
- **13 October 2025:** CME launched SOL and XRP options.
- **11 March 2026:** an **SEC–CFTC MOU**; **20 March 2026:** a CFTC margin/collateral FAQ; **March 2026:** a Cboe rule filing on multi-asset crypto trust options.

### 7.5 The Block and OTC Venues

Institutional options size does not trade on the screen. It trades as a **block** — a negotiated, privately quoted, large trade printed off the order book — and the venues and protocols that matter here are:

- **Deribit Block Trades and Combos**, advertised by the venue itself alongside "Colo Services in LD4, Portfolio Margin Model… Multicast" (Deribit's own site, September 2026 — ✅ venue-published).
- **Paradigm**, a block/OTC venue for crypto derivatives (appears among the partners listed by Laevitas; and named by SignalPlus as an integration).
- **RFQ protocols generally**, in which a client requests a quote from multiple dealers and receives competitive prices.

SignalPlus's claim that "nearly **US$70 billion in Block-RFQ transactions [were] cleared via Deribit alone**" (Q4 2025, company-reported ⚠) is best understood as evidence about *workflow share in the institutional block channel*, which is the highest-value segment of the market and far harder to enter than the retail screen. If even partially accurate, it is the strongest single fact about the company's real market position. ⚠ **It is unaudited, definitionally vague ("cleared" by whom, on whose books?) and was not confirmed by Deribit in this pass.**

### 7.6 The Clearing and Settlement Model, and Why It Matters

**State this plainly, because it is the structural difference a bank must internalise:** crypto options traded on the native venues have **no central counterparty clearing house and no central securities depository**. Trades are principal-to-principal between the client and the venue/broker under that venue's margin model, with the venue taking on the role of clearing, margining and, in effect, guaranteeing performance within its own rules. Deribit's own material describes its "Portfolio Margin Model" and its co-location and block facilities (venue-published, ✅) but no CCP sits behind the contract in the way a clearing house sits behind a listed, cleared option. Three consequences:

- **Counterparty risk is direct and uncentralised.** A bank's exposure is to the venue or dealer, not to a clearing house with a default fund and a rulebook. For an offshore venue this is a counterparty question before it is a technology question.
- **The onshore alternative is structurally different, not merely regulated differently.** CME crypto options are centrally cleared and margined within a CFTC-regulated framework — a materially different risk profile for the same economic exposure, which is what makes §7.4 a genuine choice rather than a compliance preference.
- **Settlement mechanics are venue-specific and change without notice.** ⚠ Note for the file: **Deribit announced on 27 August 2026 that it would discontinue daily proof-of-reserves publication** (headline observed on Deribit's own newsroom, ✅ for the announcement). Whatever the reason, a venue reducing published disclosure at the same time as it is being absorbed into a listed group is a reminder that the offshore native market's transparency is voluntary and revisable.

### 7.7 The Statistical Caveat — Self-Reported and Unaudited

**Every volume figure in the crypto market is self-reported or third-party-estimated, and none is audited to a financial-reporting standard.** Concretely:

- Exchange volume is published by the exchange, voluntarily and on its own definitions. Third parties (CoinGlass, CoinGecko, CoinLaw) either scrape or estimate, and their totals for the same market disagree — as §7.2 shows with "85%+" against "49.3%".
- **"Volume" is not a standardised term.** Notional USD, contract count, premium traded and "platform volume" are different measures, and a vendor's own aggregation across venues has to pick one.
- Open interest on offshore venues depends on the venue's own reporting; the reported figures and their reconciliation to actual collateral are not independently attested.
- The consequence for this guide: **every SignalPlus volume claim is ⚠ company-reported and unaudited, and every market figure is dated for that reason.** Where a figure's definition could not be established, the guide says so instead of quoting it cleanly.

---

## 8. The Competitive Landscape

Each named competitor below was checked to exist as described as of September 2026; where the check failed or the entity could not be verified, the guide says so rather than naming it. The categories matter more than the names, because the buyer's real choice is between categories.

### 8.1 The Direct Competitors — Analytics and Pricing Providers

- **Block Scholes** — crypto derivatives data and research ("Crypto Derivatives Intelligence"), supplying options data and analytics, **with its crypto implied-volatility data available on the Bloomberg Terminal since March 2026** (blockscholes.com, ✅ verified this pass, including a published communication dated 13 March 2026). The site states it "has regulatory permissions to conduct specific regulated activities in the UK" (✅ company-stated). **This is the closest analogue to SignalPlus's analytics layer — and it does not sell execution or automation.**; **Laevitas** — crypto derivatives data and analytics: "complete option chains and detailed trade flows including blocks and strategies… implied volatility to Greeks", across "15+ exchanges", with a REST API, strategy builder and back-testing (laevitas.ch, ✅ verified this pass). Partner logos include Bybit, Deribit, OKX, Paradigm, Greeks Live and Orbit. **Data and analytics; no execution or hedging automation.**
- **Amberdata** — offers "AD Derivatives: Institutional Grade Crypto Options Analytics" with spot/vol/derivatives data history and analytics (amberdata.io/ad-derivatives, ✅ verified this pass). ⚠ One secondary source (a vendor blog) reports that **Kaiko acquired Amberdata on 2 June 2026** — **single-source and unverified in this pass**; a diligence file must confirm current ownership before relying on Amberdata's independence.
- **Genesis Volatility** — ⚠ **NOT VERIFIED.** The site did not load in this pass and no primary confirmation of its current existence as a crypto options analytics provider was obtained. **It is named here only to record that it could not be verified, and should not be treated as a live competitor without a fresh check.**
- **On-chain options venues (e.g. Aevo, reported to hold 79% of the on-chain options market — CoinLaw, C-rated)** are a structurally different substitute: they compete for the trade itself rather than for the tooling, and their volumes are small relative to the centralised venues.

### 8.2 The Adjacent Substitutes — Execution, OMS and Routing

- **Talos** — an institutional digital-asset trading technology platform: "connectivity (including FIX), multi-dealer RFQ, execution algos, and smart order routing", with explicit positioning for **banks** ("Talos helps banks enter the digital assets market quickly and efficiently. Whether launching a market-making desk or looking to add digital assets to your offering, leverage an advanced suite of algos, smart order routing, liquidity aggregation, and multi-dealer RFQ"), white-label and portfolio/risk modules (talos.com, ✅ verified this pass). **This is the most credible single competitor for a bank's business, and it is a broader platform rather than a crypto-options specialist.** A bank choosing between Talos and SignalPlus is choosing between a general institutional trading stack and a specialist options engine.
- **CoinRoutes** — algorithmic execution and routing, integrated with Deribit options (Deribit's own newsroom, "Deribit Options Integrated on CoinRoutes", 20 August 2026, ✅ venue-published). An execution-layer substitute.
- **Amber Group** — an institutional digital-asset firm offering "Digital Wealth Management", asset management and "Advisory, Liquidity, Investment & Research" (ambergroup.io, ✅ verified this pass). **Not a software substitute**: Amber is a principal trading and asset-management business, i.e. what a bank would *hire* rather than what it would *buy software from* — but it competes for the same outcome (options risk transferred away) and is therefore the "don't build, outsource" alternative.
- **Greeks Live** and **Orbit Markets** — ⚠ verified only as partner logos on Laevitas's site; existence as described is not independently established in this pass. Do not list them as competitors without a fresh check.

### 8.3 The Exchanges' Own Tooling

The largest structural threat, and the one the company names implicitly by calling itself a bridge over venues that "lack the expertise to build the requisite option trading & risk management tools" (AppWorks, 27 March 2025 — the investor's framing of the opportunity):

- **Deribit** ships its own institutional toolkit: "Position Builder, Deribit Metrics, and other top-notch analytic tools", plus "Block Trades, Combos… Portfolio Margin Model… Colo Services in LD4… Multicast" (Deribit's own site, ✅). A venue already selling block trades, portfolio margin and analytics to institutions is offering, natively, a meaningful subset of what the terminal provides.
- **Bybit, Binance and OKX** all operate options markets and in-house order/position interfaces, and Bybit in particular has invested in matching-engine latency and unified margin (reported 4 August 2026).

### 8.4 The Build-It-Yourself Option

The realistic alternative for a bank with an existing TradFi options business is to **extend its own stack** rather than buy. What that requires: market-data ingestion from each venue; a surface construction and calibration process; a coherent Greeks engine; portfolio risk and scenario infrastructure; multi-venue order management; a delta-hedging service that runs 24/7 with monitoring and a kill switch; and the operational staff to run it through weekends and holidays. For a bank that already has the first five for listed equity or FX options, the incremental work is the crypto-specific connectivity and the 24/7 operations — substantial, but not a greenfield build.

### 8.5 The Live-Auction Question

The competitive set resolves into a simple map: **data-only** (Block Scholes, Laevitas), **execution-only or execution-first** (Talos, CoinRoutes), **venue-in-house** (Deribit and the other option venues), **outsource-the-outcome** (Amber Group and dealer desks), and **build**. SignalPlus's proposition is the union of the first two **plus automation and structured-product pricing** — a genuinely narrower field than any single category suggests, because few vendors sell pricing *and* execution *and* a hedging robot *and* a structured-product engine. ⚠ **But "few competitors in the exact combination" is not the same as "durable advantage"**: each component is individually available elsewhere, integration is replicable, and the venues themselves provide a partial substitute. The moat claimed by the company and its investors is domain expertise plus the workflow depth of the automation tier, and the switching cost of re-plumbing a market-making operation. That is a real but modest moat.

### 8.6 The Positioning Table

| Vendor / substitute | Category | Options-specialist? | Execution? | Automation / hedging? | Structured-product pricing? | Verified this pass? |
|---|---|---|---|---|---|---|
| **SignalPlus** | Options trading-technology platform | **Yes** | Yes (multi-leg algos, multi-venue) | Yes (DDH, market-making robot, 200+ params) | Yes | ✅ (existence of claims; capability unverified) |
| Block Scholes | Derivatives data and research | Yes | No | No | No | ✅ |
| Laevitas | Derivatives data and analytics | Yes | No | No | No | ✅ |
| Amberdata | Crypto data / options analytics | Partly | No | No | No | ✅ (ownership change ⚠ unverified) |
| Talos | Institutional trading platform / OMS | No (general digital assets) | Yes (FIX, RFQ, SOR, algos) | White-label auto-hedging advertised | No (not options-specific) | ✅ |
| CoinRoutes | Execution and routing | No | Yes | No | No | ✅ (via Deribit) |
| Deribit in-house | Venue tooling | Yes | Yes (Block Trades, Combos, portfolio margin) | No (no automation sold) | ⚠ Not disclosed | ✅ |
| Genesis Volatility | — | — | — | — | — | ⚠ **Could not be verified — do not cite** |

---

## 9. The Regulatory Perimeter

### 9.1 The Company's Own Licensing Status, or Its Absence

**No licence, authorisation or regulatory registration for SignalPlus was identified in this pass.** ⚠ **State the limitation honestly: a direct query of the Hong Kong SFC's public register of licensed corporations and of its lists of licensed virtual asset trading platforms was not completed** (the register is published dynamically and was not successfully queried here), so the correct formulation is *"no licence was found"*, not *"the company holds no licence"*. What the record does support:

- **The company describes itself as a technology company in every release, never as a licensed or regulated entity.** Its own "About" wording is a software claim: "SignalPlus builds institutional-grade derivatives trading infrastructure… Its platform provides professional options analytics, real-time risk management, and execution tools to hedge funds, market makers, proprietary trading desks, and asset managers" (1 Jun 2026). ✅; **No regulatory status is asserted anywhere in the material examined** — no "regulated by", no licence number, no compliance page, no terms-of-business page reviewed in this pass. ⚠ Absence of a claim is not proof of absence of a licence, but a company that held one would ordinarily say so.
- **The contrast with its own shareholders is instructive.** HashKey Holdings Limited trades on the Hong Kong exchange as **3887.HK** and operates **licensed digital-asset platforms** ("The Company provides trading facilitation services, on-chain services, and asset management services through licensed digital asset platforms" — HashKey Group, 20 May 2026, ✅). OKX Ventures' parent operates under licences in multiple jurisdictions. **SignalPlus's strategic investors are inside the perimeter; SignalPlus is not.** That is normal for a software vendor and it is the crux of the analysis in §9.5.

### 9.2 The Hong Kong SFC Virtual-Asset Regime

The relevant Hong Kong perimeter, verified in this pass:

- **The SFC operates a licensing regime for centralised virtual asset trading platforms (VATPs) under the Anti-Money Laundering and Counter-Terrorist Financing Ordinance (Cap. 615), effective 1 June 2023**, per the SFC's own implementation circular (23EC28, ✅ regulator-published). Under it, "centralised virtual asset trading platforms operating in Hong Kong will need to apply to the Securities and Futures Commission (SFC) for a licence" (circular wording). The SFC publishes **lists of virtual asset trading platforms** (licensed, deemed-licensed and application-stage) on its own site (✅).
- **The SFC's stated ambition is breadth, not narrowness:** "The SFC's regulatory regime seeks to capture all the dimensions of the public's interface with virtual assets, providing for investor protection and market integrity while managing key risks" (SFC virtual-assets pages, ✅). That sentence is the reason a software vendor's perimeter question is worth asking at all.
- The SFC also maintains an **Alert List** and an **enforcement actions** page for unlicensed activity (✅).

**Does a software vendor inside that regime need a licence?** Not for selling software. The VATP regime catches **platform operators** — entities that provide a trading venue, match orders, hold client assets or custody. SignalPlus does none of those things (§1.1): it has no venue, no matching engine and no custody. On the evidence available, **its activity sits outside the VATP perimeter.**

### 9.3 The May 2026 Convergence — Dealing, Custody, Advisory, Management

Hong Kong's virtual-asset perimeter is being **widened right now**, and this is the most important regulatory development for any firm considering a platform of this class. **On 26 May 2026, the FSTB and the SFC jointly published consultation conclusions on regulatory regimes for virtual asset advisory service providers and virtual asset management service providers** (HK Government press release, ✅ regulator-published). The verified points:

- 51 submissions were received in the one-month consultation completed in late January 2026; the majority supported **separate licensing regimes** for VA advisory and VA management service providers, "which model on the regulatory framework applicable to the conventional securities market".
- Adhering to "**same activity, same risks, same regulation**", the scopes of the proposed regimes "will be aligned with that for **Type 4 (advising on securities)** and **Type 9 (asset management)** regulated activities" under the Securities and Futures Ordinance (Cap. 571).
- The FSTB and SFC "are finalising the legislative proposals for the regulatory regimes for **VA dealing, custodian, advisory and management service providers**, with a target of introducing the relevant bill into the Legislative Council within this year" — i.e. **a four-regime perimeter (dealing, custody, advisory, management) is in active legislative preparation as of May 2026** (✅ regulator-published; the bill was not yet enacted in the sources examined).

### 9.4 The Singapore MAS Angle

SignalPlus has a Singapore footprint on the record: the August 2022 and January 2025 releases were datelined Singapore, and the media contact numbers in the 2025, 2025-November and 2026 releases are **Singapore numbers (+65)**. ⚠ No MAS licence, exemption or registration was identified in this pass. The applicable Singapore regime for digital-asset services is the **Payment Services Act** digital-payment-token framework, under which dealing in, exchanging and custodial services for DPTs are licensable — **this repository's [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) guide owns that detail and it is not re-derived here.** The relevant point for this company is the same as in Hong Kong: **pure software licensing is not, of itself, a licensable payment service.** A vendor that only licenses tooling and never deals, never holds funds and never manages client assets sits outside the regime. That conclusion holds *provided* the vendor's actual conduct matches its licence-free self-description — which is why §9.5 exists.

### 9.5 Software Versus Dealing — the Classification Analysis

This is the crux for a regulated institution. The perimeter question is not "is SignalPlus licensed?" — it is "**could any part of what SignalPlus does be characterised as dealing, advising, managing or operating a venue, either now or as the product evolves?**" Worked through honestly:

| Activity | Is SignalPlus doing it? | Analysis |
|---|---|---|
| **Operating a trading venue** | No | No matching engine, no order book of its own, no venue. Orders go to third-party exchanges. ✅ Outside the VATP regime |
| **Custody of client assets** | No | No wallets, no key custody, no asset safekeeping. ⚠ It does hold **API credentials** with trading rights, which is a *systems* control issue, not an asset-custody business (§10.6) |
| **Dealing as principal** | No evidence | Nothing in the record shows the company taking the other side of a trade, warehousing risk, or quoting as a dealer. AppWorks states the opposite posture explicitly (§1.1). ⚠ Not disproved, but unevidenced |
| **Advising on virtual assets** | **Borderline, product-dependent** | The analytics are tools, not advice. But "**actionable trading modules**" (SignalPlus 2.0, announced 1 Jun 2026) and the "Trading Compass" news/sentiment labelling (⚠ third-party-attested name only) sit close to the line if they ever become **personalised recommendations about specific transactions for a specific client**. Tools and generic research are not advice; bespoke recommendations are |
| **Managing assets / exercising discretion** | **Borderline, contract-dependent** | The automation runs a market-making or hedging protocol under **client-supplied parameters** and **client-held accounts**, which is tooling. It becomes management if the vendor **sets the parameters for the client, tunes them with discretion, or accepts responsibility for the outcome**. The homepage's own language — "we provide automated and fully-customizable market making **services**" — is ambiguous on precisely this point |
| **Guaranteeing performance / taking credit risk** | No evidence | No source examined shows the company standing behind a loss or extending credit. If it ever did, it would move toward dealing |

**The conclusion, stated plainly:** on the public evidence, SignalPlus's activity is **software licensing and technology provision, and therefore outside the SFC's VATP regime, outside the VA dealing regime and outside the MAS DPT licensing perimeter.** ⚠ But that conclusion rests on **conduct that is only partially observable from outside**, and the two boundary products — the parameterised automation service and the announced agentic-AI recommendations — are the ones to watch. The controlling mitigations are contractual and architectural, and a buyer should require them in writing: **the client is at all times the principal and the account holder; the client retains full discretion over parameters, limits and strategies; the vendor provides no personalised recommendation to transact in a specific instrument; the vendor takes no principal position, extends no credit and guarantees no outcome.** If a vendor will not put those in a contract, the classification question is the vendor's problem and should not be the bank's.

### 9.6 Why the Analysis Matters Even With No Licence

Three reasons a bank's compliance function should care even though the vendor almost certainly needs no licence:

- **A bank cannot outsource regulated activity to an unregulated party.** If the desk delegates discretionary decision-making to a vendor's engine, the bank has not escaped the perimeter — it has created an unauthorised, unsupervised actor inside its own process. The outsourcing/third-party expectations of MAS (and the SFC, and prudential supervisors generally) apply to the *bank's* use of the vendor regardless of the vendor's own status. The [Operational Resilience Framework](operational_resilience_framework_guide.md) guide owns the framework here.
- **The perimeter is moving, in the direction of inclusion** (§9.3). A vendor relationship designed in 2026 against a tooling model should be documented so that if the vendor's product shifts toward advising or managing, the bank has notice, audit and change-of-control rights rather than a fait accompli.
- **Disclosure risk is asymmetric.** If a bank's regulator later forms the view that the bank outsourced part of a regulated activity to an unlicensed third party, the cost lands on the bank. The remedy is cheap: write the four control clauses into the contract up front, and record the classification analysis in the vendor file.

---

## 10. The Regulated-Institution Angle

### 10.1 The Vendor-Diligence View, and Where the General Framework Lives

**The general vendor-diligence framework is owned elsewhere in this repository and is not rebuilt here.** [Vendor Management](../management/vendor_management_guide.md) covers the discipline end to end — §1 the discipline itself, §2 strategy and segmentation, §3 selection and sourcing, §4 contracting, §5 onboarding and implementation, §6 performance management, §7 relationship management, **§8 vendor risk management (VRM/TPRM)**, §9 offboarding and exit, §11 a worked core-banking example. [Operational Resilience Framework](operational_resilience_framework_guide.md) owns the third-party resilience layer — §10 third-party resilience and supply chain, §4-§6 business impact analysis, important business services and severe-but-plausible scenarios, §8 DR tiers and RTO/RPO. **Use those guides for the process; use this section only for the crypto-options-specific overlay** that the general framework does not anticipate. The overlay items, in the order they should worry a desk:

| Overlay risk | Why it is specific to this vendor class | Where handled |
|---|---|---|
| Uninspectable pricing model | The thing being bought is a model, not just a screen | §10.2 |
| Valuation governance | The bank's IPV and P&L may depend on a third party's surface | §10.3 |
| Concentration on a small vendor | ~29 employees claimed (⚠); single-platform dependency | §10.4 |
| Position data and API credentials | The vendor sees every desk's book and holds trading keys | §10.6 |

**One conflict item that belongs in the register now:** SignalPlus's strategic investors include **OKX Ventures** and the **HashKey group**, and **HashKey Capital launched a "Digital Asset Derivatives and Structured Solutions Business" on 27 August 2026** (HashKey Group newsroom, ✅ verified). An investor that has entered the same product space as the company it funds is a standard conflict to record, to test, and to revisit — not an allegation. ⚠ No evidence of any adverse effect on the vendor's neutrality was found in this pass.

### 10.2 The Uninspectable Model — Technology and Model Risk

The single most important sentence in this section: **the bank would be importing a pricing model it cannot inspect, using it in risk reporting and P&L attribution, and potentially using it as the reference price for valuing its own positions.** That is a model-risk event whatever the vendor's quality. Concretely, a bank adopting this platform must:

- **Treat the vendor's surface as a model in the model inventory.** It has inputs, assumptions, a calibration and outputs that feed risk and P&L. Model-risk governance (identification, inventory, independent validation, change control, periodic review) applies whether the model was built in-house or bought. ⚠ On the evidence available, the vendor publishes **no** model documentation, no methodology, no version history and no calibration change log — so validation cannot start from documentation and must be done **black-box on outputs** (§11.4).
- **Establish what the vendor will and will not disclose.** "Full model & pricing transparency" (§4.6) is unquantified. The correct ask is a written model specification, a calibration report with parameters, a version and change-notification commitment, and — for a large enough contract — source-code escrow or a model-risk attestation. ⚠ Whether any of that is available was not established in this pass; it is a negotiation question, not a published fact.
- **Separate the decision from the model.** No bank should run an automated hedging or quoting strategy off an unvalidated third-party surface without an independent second price. If the bank's risk system takes the vendor's number as truth, the bank has outsourced its valuation judgement, which is not a technology decision and cannot be delegated to a supplier.

### 10.3 The Valuation-Governance Question

**How does a bank independently validate a third-party crypto-options price?** This is the practical question the whole section reduces to, and it has a known good answer: **you reprice, independently, against more than one source, and you govern the exceptions.** The mechanics a bank should require:

- **A second independent price source** — the bank's own model, a second analytics vendor (Block Scholes, Laevitas, Amberdata-type providers exist for exactly this; §8.1), or exchange marks where the instrument is liquid. Repricing against the *same* vendor's model is circular and worthless.
- **Instrument-level revaluation on a fixed schedule**, not a spot check: a defined basket (vanillas across strikes and expiries, plus the specific structures the desk runs), repriced at a defined frequency, with results retained as evidence.
- **Published tolerances and an escalation path** — e.g. a tolerance in premium terms or vol points, with exceptions investigated, root-caused and reported to the valuation committee. Tolerances must be set before results are seen.
- **IPV/P&L crossover discipline** — the same independent price must be available to the finance/second-line valuation process, not only to the traders, so that front-office and independent valuation of the same book can be compared.
- **Documented snapshot and reproducibility policy from the vendor** (§5.1) — a valuation control that cannot reproduce yesterday's price is not a control.

⚠ **Nothing in the vendor's public material addresses any of this**, which does not mean the capability is absent; it means the bank must establish it contractually and evidence it itself.

### 10.4 Counterparty, Operational and Concentration Risk

- **Counterparty risk is the venue's, not the vendor's** (§7.6). SignalPlus holds no client assets, so the counterparty exposure is to Deribit/Binance/Bybit/OKX — a *venue* diligence exercise, for which the repo's [Bitunix](bitunix_guide.md) (exchange software stack and CEX diligence shape) and [Fireblocks](fireblocks_guide.md) (custody/infrastructure) guides are the sibling references, without re-deriving the venue landscape here.
- **Operational and concentration risk is squarely the vendor's.** A single supplier, reportedly a few dozen people (⚠), providing software that sits between the bank and its execution venues, on which a desk's pricing, risk and automated hedging all depend. Lose the supplier and the desk loses its analytics, its hedging automation and — critically — its **parameter set and surface history**. The general concentration test in [Vendor Management](../management/vendor_management_guide.md) §8 applies; the specific question is **what the desk can still do on the day the vendor stops answering**.
- **Fourth-party risk is invisible.** The vendor's own dependencies — cloud regions, exchange API rate limits and deprecations, upstream market-data sources, and any subcontractors — are not disclosed (⚠ §5.7). A bank that cannot see the subcontracting chain cannot assess it, and should say so in the risk acceptance rather than assume there is none.
- **The conflict noted in §10.1** belongs in the concentration assessment: an investor that is also a venue group and has launched a competing derivatives business is a governance question about the vendor's neutrality, not merely a shareholder fact.

### 10.5 Business Continuity and Key-Person Risk

- **The product promise is 24/7, the disclosure is nil.** The automation is marketed as "24/7 uptime trading across major venues with minimal human oversight", and ⚠ **no SLA, no uptime target, no RTO, no RPO, no incident-notification commitment, no maintenance window and no DR description is published.** For a system whose failure mode is an unhedged book over a weekend, that is the most material gap in the vendor's public profile after the model disclosure gap.
- **Key-person risk is concentrated in two founders.** Chris Yu is the public face, the principal quote and (by all accounts) the technical authority on the pricing model; AppWorks' note describes him personally intervening on the volatility model mid-fundraise. The departure or incapacity of either founder is a business-continuity event for a company of this size, and the mitigation (documented model, documented parameter sets, escrow, transition commitments) is not in evidence. ⚠
- **The bank's own continuity plan must not depend on the vendor's.** The correct design is that the desk can price, risk and hedge manually — badly and slowly, but manually — on the day the platform is down. A desk whose hedging capability exists only inside a third-party robot has not bought automation; it has bought an outage. The framework lives in [Operational Resilience Framework](operational_resilience_framework_guide.md) §7-§8; the design specifics are §11.6.

### 10.6 Confidential Position-Level Data Leaving the Firm

**Exposure one: position data.** To compute cross-venue risk, the platform must ingest the bank's positions and orders across every connected venue. That means a third party — and, in a multi-client deployment, a single operator — accumulates a view of many desks' books. The risks are: (a) **confidentiality** — the bank's positioning is proprietary and its leakage is market-moving; (b) **aggregation and inference** — an operator seeing many books can infer market positioning beyond any individual client, and the vendor's own investors include venue operators; (c) **competitive use**, whether or not it ever happens, is a reputational and legal exposure the bank has created. Contractual mitigations to require: named confidentiality of position data, prohibitions on aggregation for any commercial purpose, restrictions on sharing with affiliates and investors, access controls and logging, breach notification timelines, and an audit right. **Exposure two: the API credentials.** This is the sharper one, because it is an **operational and cyber** risk rather than a contractual one. A delta-hedging or market-making robot must be able to send orders on the bank's exchange accounts. That credential is a live execution capability. The controls that must be evidenced, tested and re-tested:

| Control | Requirement |
|---|---|
| Withdrawal permission | Trading-enabled keys only; **withdrawals disabled on the account** |
| Scope | Per-venue sub-accounts or dedicated accounts, so a compromised key cannot reach the bank's other activity |
| IP allow-listing | Exchange-side allow-list locked to the vendor's egress IPs |
| Limits | Exchange-side order-size and rate limits set below the bank's risk appetite |
| Kill switch | A bank-side, human-operated means of disabling the robot **independently of the vendor** — and a tested procedure for it |

⚠ **None of these controls is described in the vendor's public material**, and the certification logos (§4.8) do not substitute for them: an ISO or SOC display says nothing about whether the vendor's own systems, if compromised, would hand an attacker trading capability in the bank's accounts. That question — asked directly, in writing, and answered under a contractual commitment — is the single most valuable hour of due diligence on this vendor.

### 10.7 Exit and Portability

- **What is portable:** positions and collateral, because they live at the venues, not with the vendor. This is the vendor's structural advantage to a client and its disadvantage as a lock-in: the assets are not held.
- **What is not obviously portable:** the **surface history and snapshot archive**, the **calibrated parameter sets** for the market-making and hedging robots, and the desk's institutional knowledge of how those parameters behave. ⚠ Nothing examined in this pass commits the vendor to export, transition assistance or continuity of access after termination. A contract without an export clause leaves the parameter set behind.
- **Practical exit design:** require a documented data-export format and a defined post-termination access window; require the parameter sets to be documented in a human-readable form the desk retains; run at least one dry migration (to a rival platform or to the bank's own model) before you need it; and set the parallel-run and re-migration expectations before signing, not after. The general exit discipline is in [Vendor Management](../management/vendor_management_guide.md) §9.

---

## 11. The Cymbal Bank Worked Example

> **Everything in this section is fictional and illustrative.** Cymbal Bank is a fictional institution used across this repository as the worked-example persona. The vendor profile is real and the risks are real; **the bank, its desk, its positions and every number below are invented for the purposes of showing the decision structure.** No figure in §11.7 is a quote, an estimate from the vendor, or a market rate established anywhere in this guide.

### 11.1 The Scenario

Cymbal Bank's digital-asset unit has run a small spot and perpetual book for two years. The treasury committee has now asked for two things the desk cannot currently deliver: **a hedging capability on crypto optionality** (so the book can be defended against a large downside move without unwinding spot), and **the ability to price and issue crypto-linked structured notes** for a small set of private-banking clients who have asked for defined-risk exposure. The desk has two quantitative analysts, no options pricing stack, no volatility surface, and a mandate to be live within three quarters. Management has asked whether to buy a platform of the SignalPlus class, build, or decline the business.

### 11.2 The Requirements

| # | Requirement | Rationale |
|---|---|---|
| 1 | Multi-venue options pricing and analytics with a coherent volatility surface | Options on BTC/ETH trade across several venues; a single-venue surface misprices risk |
| 4 | The ability to hedge delta overnight and at weekends | Crypto trades 24/7; a two-person desk cannot |
| 5 | Structured-note pricing with a defensible, documented model and margin attribution | Required to issue the notes at all |
| 6 | Independent price verification for every position the desk holds | Valuation governance (§10.3) |
| 7 | Contractual controls over API credentials and position data | Cyber and confidentiality (§10.6) |

### 11.3 The Due Diligence Against Section 10

Twenty questions, each mapped to the discipline that requires it. Illustrative answers show the shape of a real response; where the honest answer is "unknown", the guide says unknown.

| # | Question to the vendor | Discipline | Illustrative outcome |
|---|---|---|---|
| 1 | Which model family constructs the surface, and how is it kept arbitrage-free? | §10.2 | Partial answer; no written specification |
| 2 | Will you provide a model specification and a calibration report under NDA? | §10.2 | Expect negotiation, not a refusal |
| 6 | What is the deployment and tenancy model; is data logically segregated? | §10.4 | Unknown — material |
| 7 | What are the SLA, uptime target, RTO, RPO, incident-notification and maintenance windows? | §10.5 | Not published; demand in contract |
| 8 | Do you hold trading-enabled API keys, and with what scoping and controls? | §10.6 | Yes by design; controls to be evidenced |
| 9 | Do you prohibit withdrawal permissions on client keys? | §10.6 | Require contractually |
| 10 | How is bank-side kill-switch independence achieved and how often is it tested? | §10.6 | Unknown |
| 12 | Will you contractually prohibit aggregation of our position data for any purpose? | §10.6 | Require contractually |
| 17 | Which reference clients will speak to us directly? | §6.5 | Company-reported roster only; ask for direct confirmation |
| 19 | What proportion of the automation's parameters do reference clients actually run? | §4.5 | Unknown — the sharpest question on the list |

### 11.4 The Technology and Data Validation Plan

The bank's plan, stated as a gate sequence rather than a wish list:

- **Stage 0 — free terminal (weeks 1-4).** Run the free dashboard and Volatility Lab against the desk's existing book. No credentials beyond read-only. Purpose: confirm the analytics are usable and the surface resembles the venues' own marks.
- **Stage 1 — independent repricing (weeks 2-12).** Reprice a fixed basket on a fixed schedule — **300 instruments across strikes and expiries, on 20 valuation dates** (illustrative) — against (a) the bank's own Black-Scholes/local-vol implementation for vanillas, and (b) a **second, unrelated data provider** for its surface. Record the distribution of differences, not just the average. **Gate: 95% of vanillas within 25 basis points of premium, and no systematic smile skew**, or the adoption stops here.
- **Stage 2 — hedging robot in shadow mode (weeks 4-16).** The DDH robot runs with **order submission disabled** on a live book: it computes and logs the hedge it *would* have sent, and the desk compares against its own manual hedge. Gate: no material divergence in a volatile window, and correct behaviour during a venue outage.
- **Stage 3 — limited live hedging (weeks 16-26).** One sub-account, scoped trading-only keys, exchange-side order-size caps, a bank-side kill switch, and a daily key-and-permission reconciliation. Position and limit reporting to risk daily.
- **Stage 4 — structured note pricing (weeks 20-30).** Only after Stage 1's repricing tolerance is met, and only with model documentation (item 2 of §11.3) in hand. First issuance is capped at a size the desk can hold unhedged if the hedging path fails.
- **Cross-cutting evidence:** every stage produces a documented artefact for the model-risk and valuation files, and a **90-day parallel run** before the vendor's surface is used in any external reporting.

### 11.5 The Regulatory Sign-Off Path

- **The vendor requires no licence and the bank does not need one either** — the bank's own dealing is licensed activity conducted by the bank; the vendor supplies tooling (§9.5). This must be **recorded as a documented classification opinion**, not assumed, and re-tested if the vendor's product moves toward advice or discretion.
- **Approvals for the bank:** new-product approval for options and structured notes; **model-risk sign-off** for the imported surface (treated as a third-party model in the inventory); **operational-resilience sign-off** for the third-party dependency and its important-business-service mapping ([Operational Resilience Framework](operational_resilience_framework_guide.md) §5, §10); **third-party risk assessment and risk acceptance** ([Vendor Management](../management/vendor_management_guide.md) §8); legal review of the credential, data and exit clauses.
- **Notification:** a material outsourcing of a critical function may trigger notification or approval obligations in the bank's own jurisdiction. ⚠ This guide does not assert which apply to a fictional bank; a real firm must test that against its own regime.

### 11.6 The Risk-Limit and Governance Design

| Control | Design |
|---|---|
| Automation allow-list | Only named strategies may run unattended; **the market-making robot is explicitly excluded** until the desk has an options desk's staffing |
| Notional and Greek caps | Hard caps on vega, gamma and delta exposure at book and per-venue level, enforced pre-trade |
| Per-venue limits | No single venue above a set share of exposure, given the concentration and counterparty facts in §7.2 and §7.6 |
| Kill switch | Bank-side, tested monthly, independent of the vendor; documented in the desk's runbook |
| Credential governance | Keys inventoried, scoped, allow-listed, rotated on schedule; withdrawals disabled; sub-accounts per strategy |
| Reconciliation | Daily position and P&L reconciliation between the vendor's view and the bank's books, with a defined break-resolution path |
| Manual fallback | A documented, rehearsed manual pricing and hedging procedure usable when the platform is unavailable |

### 11.7 The Alternatives and the Cost-Benefit Comparison

**All figures illustrative and deliberately round. They are not quotes, not benchmarks and not claims about any vendor's pricing.**

| Option | Year 1 (illustrative) | Year 2 (illustrative) | Time to live | Principal risks |
|---|---|---|---|---|
| **A. Buy the platform** | US$520k: US$300k licence and enterprise tier, US$120k integration and data, US$100k run and oversight | US$400k/yr: US$300k licence, US$100k run | 2-3 quarters | Model you cannot inspect; vendor concentration; credential exposure; parameter opacity |
| **B. Buy analytics only, hedge manually** | US$180k: US$80k data and analytics subscriptions, US$60k second price vendor, US$40k tooling | US$180k/yr | 1 quarter | Two analysts cannot cover 24/7 hedging; overnight gap risk remains |
| **C. Build in-house** | US$750k: 2.5 FTE quantitative and engineering (loaded), US$150k market-data and infrastructure, plus a measured **9-month** delay to first live hedge | US$650k/yr | 3-4 quarters | Delivery risk; hiring risk; permanent maintenance burden; the delay itself is a risk cost |
| **D. Do not enter; outsource the exposure to a dealer** | Dealer spread, no fixed cost | Dealer spread | Immediate | Ongoing spread cost; no capability built; client demand unmet; concentration on a single dealer |

### 11.8 The Recommendation

**Adopt in stages; do not adopt wholesale.** Specifically:

- **Start with the free terminal (Stage 0), at zero cost commitment** — evaluate the analytics before any procurement. This is genuinely free and requires no justification beyond a read-only key.
- **Proceed to independent repricing (Stage 1) as a hard gate.** If the surface does not reconcile within the stated tolerance against a second source, stop. Nothing else matters if the pricing does not survive this test.
- **Adopt the DDH robot only after shadow mode, and only with scoped credentials, exchange-side limits and a tested bank-side kill switch.**; **Adopt the structured-product pricer only with model documentation in hand.** If the vendor will not provide documentation adequate for the bank's model-risk process, the bank should not use the vendor's price as the basis of a client-facing note.
- **Do not adopt the market-making robot.** A 200+ parameter market-making engine operated by two analysts on a bank's balance sheet is a risk the desk is not staffed to carry.
- **Never use the vendor's surface as the bank's independent price.** The second source is mandatory, not optional, and it is the control that keeps the whole arrangement honest.

### 11.9 What the Bank Cannot Establish

Even with a thorough process, the following remain unknown to the bank, and the adoption decision must be taken with that acknowledged: **the vendor's financial health and runway** (no revenue, margin or retention data is public); **the true customer concentration** (the roster is company-reported, not confirmed first-party); **the internals of the pricing model** (validated black-box on outputs only); **behaviour in a genuine tail event** (no independent test evidence exists, and a shadow run covers only the market conditions that occurred during it); **the depth of the team** (a few dozen staff claimed, ⚠, with key-person concentration in two founders); **the structure of the 2026 funding** (the B+ and B1 relationship is genuinely ambiguous, §3.3); **which venue integrations beyond Deribit are live** (one of five is venue-confirmed, §6.4); and **whether the vendor's other investors' interests will remain aligned with the bank's** (§10.1). A bank that writes those eight gaps into its risk acceptance — rather than assuming them away — has done the work this vendor class requires.

---

## 12. The Claims Audit — Verified, Flagged, Rejected

Source-quality ratings as defined in §1.5: **A** regulator/registry/venue-published, **B** company-published (first-party, self-interested), **C** reputable third-party, **D** weak or untraceable. **Dates matter more than anything else on this page:** marketing and funding claims go stale fastest, and the single most-repeated figure about this company was already three years out of date when this guide was written.

### 12.1 The Verified Claims (✅)

| # | Claim | Source | Quality | Date |
|---|---|---|---|---|
| 1 | SignalPlus is a Hong Kong technology company building options/derivatives trading software, founded 2021 | Company releases and company page (consistent across four years) | B | 2022-2026 |
| 2 | Co-founders: Chris Yu (CEO) and James Shan (COO) | Company releases; AppWorks investment note; aggregator profiles | B/C | 2022-2026 |
| 5 | Core terminal offered free, "zero cost commitment" | Company site, releases, CFO keynote | B | 2023-2025 |
| 6 | **Deribit integration, announced jointly, with a Deribit executive quote** | **Deribit's own newsroom** | **A** | 22 Dec 2022 |
| 7 | US$11M Series B up-round led by AppWorks and OKX Ventures, with Avenir Group and HashKey | Company release | B | 24 Jan 2025 |
| 8 | US$40M Series B+ led by a HashKey Capital-managed fund (HashKey Group contributing US$20M) plus a strategic partnership | HashKey Group newsroom | B (counterparty) | 20 May 2026 |
| 9 | US$50M Series B1 at a US$500M post-money valuation, led by HashKey Capital, with BlockBooster and AppWorks | Company release | B | 1 Jun 2026 |
| 10 | Goldman Sachs served as **sole financial advisor** on the B1 round | Company release | B | 1 Jun 2026 |
| 12 | Hong Kong entity SIGNALPLUS HK Limited incorporated 27 Jan 2022, company no. 3125033, BRN 73764584 | HK registry aggregators (four agreeing sources) | C | 2026 |
| 13 | Crypto options market H1 2026: US$864.6bn across the top five venues; Deribit 49.3% (56.3% in Jan, 41.8% in Jun); Bybit 22.3%, Binance 13.4%, OKX 13.3% | CoinGlass data reported in press | C | 4 Aug 2026 |
| 15 | HK FSTB/SFC consultation conclusions on VA advisory and VA management regimes published, modelled on SFO Type 4 and Type 9, with VA dealing/custody regimes being finalised | HK Government press release | **A** | 26 May 2026 |
| 16 | VATP licensing regime under AMLO (Cap. 615) effective 1 Jun 2023; SFC publishes lists of VATPs | SFC circular and SFC virtual-assets pages | **A** | 2023 |
| 17 | HashKey Capital launched a digital asset derivatives and structured solutions business | HashKey Group newsroom | **A** | 27 Aug 2026 |
| 18 | Named competitors exist as described: Block Scholes, Laevitas, Amberdata (AD Derivatives), Talos, CoinRoutes, Amber Group | Their own websites and Deribit's newsroom | A/C | Sep 2026 |
| 19 | The company holds no licence that could be identified; no licence is claimed anywhere in its own material | Absence of claim across all sources examined | ⚠ (negative finding, not a register query) | Sep 2026 |

### 12.2 The Flagged Claims (⚠)

| # | Claim | Why it is flagged | Source | Date |
|---|---|---|---|---|
| 1 | US$160bn platform volume in Q4-2025 | Unaudited; "platform volume" undefined (notional vs premium vs contracts) | Company | 1 Jun 2026 |
| 2 | ~US$70bn of Block-RFQ cleared via Deribit alone | Unaudited; "cleared" undefined; no venue-side confirmation found | Company | 1 Jun 2026 |
| 3 | 74% quarterly CAGR in terminal volumes since 2023 | No absolute series or definition; ≈770× over twelve quarters if compounded literally | Company | 1 Jun 2026 |
| 5 | **Customer roster: Cumberland, FalconX, Galaxy Digital** | **Reported by the company; not confirmed first-party** | Company | 1 Jun 2026 |
| 6 | Goldman Sachs named in the user roster with an attributed employee quote | Company-distributed statement; not independently confirmed | Company | 20 Nov 2025 |
| 9 | "AI-powered" robot, QuantLab engine, SignalPlus 2.0 agentic AI | No model documentation, methodology, benchmark or independent evaluation; 2.0 in future tense | Company | 2025-2026 |
| 10 | Certifications: ISO, AICPA SOC, Leviathan MASE, Astra A+, GDPR | Displayed as logos with no number, scope, auditor or date | Company site | 2026 |
| 15 | The relationship between the US$40M Series B+ (20 May 2026) and the US$50M Series B1 (1 Jun 2026) | **Genuinely ambiguous**; same lead, two amounts, thirteen days apart; possibly the same capital event described twice | Both releases | 2026 |
| 16 | "Paradigm" as an investor | The 2022 release names "Paradigm Co" as a Series A participant; the 2025 release names "Paradigm" as a venue integration; the relationship is unestablished | Company releases vs aggregator lists | 2022-2026 |
| 17 | Seed-round lead | Conflict: the 2022 release names GBV Capital as seed lead; HashKey's 2026 release says its fund led the seed | Two issuers | 2022 vs 2026 |
| 18 | Deribit's "~85%+ market share in BTC and ETH options" | Venue self-report, in tension with CoinGlass's 49.3% for H1 2026; different baskets may be measured | Deribit site vs third-party | Sep 2026 vs Aug 2026 |

### 12.3 The Rejected or Corrected Claims (❌)

| # | Claim | Verdict and reason |
|---|---|---|
| 1 | "**~US$23M raised in total**" (CypherHunter, Gate, CoinCarp and review content; still repeated in an April 2026 review) | ❌ **Stale.** The figure is $12M Series A (2022) + $11M Series B (2025) and predates the 2026 rounds. Corrected totals are **~US$73M** (if B+ and B1 are one event) or **~US$113M** (if sequential) — both flagged ⚠ |
| 2 | "**US$5 billion valuation**" (en.theblockbeats.news/news/62584, and a content aggregator repeating it) | ❌ **Probable error.** The same article's own headline says US$500M while its body says US$5B; the company release, odaily, pulsealternative, Binance Square and dailyaibrief all say **US$500M** |
| 3 | "Goldman Sachs is a SignalPlus customer" or "Goldman Sachs invested in the B1 round" | ❌ **Not supported.** Goldman Sachs was the **sole financial advisor** on the B1 round. A separate November 2025 company release names it in the roster, flagged ⚠ and unconfirmed |
| 4 | SignalPlus is an exchange, a custodian, or a principal market maker | ❌ **Not supported by any source examined.** It is a software vendor: no venue, no matching engine, no client-asset custody, no proprietary trading book in evidence |
| 5 | SignalPlus holds an SFC licence or is a licensed VATP | ❌ **No such licence found**, and none is claimed. ⚠ The caveat stands: a direct query of the SFC's registers was not completed in this pass (§9.1) |
| 6 | "Full front-to-back services" | ❌ **Overstated** as commonly read; settlement, clearing and custody remain at the venues (§4.7) |
| 7 | "Paradigm is a SignalPlus investor" (as aggregator lists present it) | ⚠/❌ **Unestablished**, and probably a conflation of an investor name ("Paradigm Co", 2022) with a venue name ("Paradigm", 2025) |
| 8 | "$23M total" carried forward into 2026 analysis | ❌ The failure mode, not the number: an undated aggregator total is wrong within eighteen months. Date every funding figure |

**The pattern worth carrying forward:** the erroneous and stale figures all run in the direction that **flatters the company** — a ten-fold-higher valuation, a total raised that understates the new rounds, a self-reported market share above the third-party number. That is not evidence of bad faith by the company; it is evidence that **funding and volume claims circulate faster than they are corrected, and that a diligence file must date every one of them.**

---

## 13. What Could Not Be Verified

### 13.1 Unaudited Financials and True Scale

**No revenue, margin, bookings, ARR, customer count or retention figure has ever been disclosed.** No audited financial statement exists in the public record. The volume claims (**US$160bn in Q4-2025**, **~US$70bn of Deribit Block-RFQ**, the **74% quarterly CAGR**, the **tenfold share growth**) are self-reported, undefined as to measure and unaudited — and, as §5.5 shows, at least one of them cannot be reconciled arithmetically. A US$500M post-money valuation on undisclosed revenue is **not assessable**, in either direction.

### 13.2 True Customer Concentration

The roster is **reported by the company; not confirmed first-party** (§6.5). What is unknown: how many paying customers exist, what share of revenue the largest one represents, whether the named firms are terminal users, automation licensees or both, whether those relationships are current as of September 2026, and whether any of them has been lost. Concentration is the metric that most affects both the vendor's durability and the buyer's negotiating position, and it is invisible.

### 13.3 Architecture Internals

Not disclosed: the model family and calibration for the volatility surface; the Greeks computation method; the deployment and tenancy model; data residency; the SLA, RTO, RPO and DR design; the versioning and change-notification policy; the security-certification scope behind the logos; incident history; and headcount and organisational structure. ⚠ Absence of disclosure is not evidence of a weakness — but it does mean **every claim about how the platform behaves must be established contractually and tested by the buyer**, which is precisely what §11.4 sets out.

### 13.4 Performance Claims

No independent test of the execution algorithms, the DDH robot or the market-making robot was found: no execution-quality or TCA analysis, no fill-rate or slippage data, no back-tested or out-of-sample performance, no behaviour report for a fast market or a venue outage, and no evidence for any "AI" attribute. The certification logos carry no scope. **Nothing on this list should be assumed to work as marketed until the buyer has run it in shadow mode.**

### 13.5 Roster Confirmation

No first-party statement from Cumberland, FalconX, Galaxy Digital, Goldman Sachs, Binance, Bybit, OKX or Paradigm confirming the relationship was located in this pass. The named quotes in the 20 November 2025 release are company-distributed testimonials from named individuals; they are stronger than a logo and weaker than confirmation, and they should be treated accordingly **(§6.5: reported by the company; not confirmed first-party)**.

### 13.6 The Limitations of This Research Pass

Stated plainly so a reader can weigh the whole guide: **web search returned empty results intermittently** and several checks had to be retried or rerouted through direct page extraction; **the Hong Kong SFC's public registers were not successfully queried**, so the licensing finding is "**no licence found**" rather than "no licence exists"; **the Hong Kong Companies Registry was not queried directly**, so entity details rest on registry aggregators; the **Deribit Block-RFQ figure has no venue-side confirmation**; the **Kaiko–Amberdata transaction** rests on one secondary source; **Genesis Volatility could not be verified** and is therefore not treated as a competitor; and **no contact was made with the company, its investors or its reported customers** — this is a public-record guide, not an interview-based one. No fact, figure, source or URL in this guide was invented; where a check failed, it is recorded as failed.

---

## 14. The Glossary

For a banking reader coming to crypto derivatives for the first time.

| Term | Definition |
|---|---|
| **Delta** | The sensitivity of an option's price to a change in the underlying price. Delta hedging = trading the underlying to neutralise that sensitivity |
| **Gamma** | The rate of change of delta. High gamma means the hedge must be rebalanced frequently, which is exactly the burden automation is sold to remove |
| **Vega** | Sensitivity to implied volatility. A "vega-neutral" offset is a trade structure designed not to add or remove volatility exposure |
| **Volatility surface** | The mapping of implied volatility across both strike and expiry. The "smile" is the strike dimension (why out-of-the-money options carry higher implied vol), the "term structure" the expiry dimension |
| **Model surface / model smile** | A **constructed** surface produced by fitting a model to market quotes, as opposed to the raw quotes themselves. SignalPlus markets a "model volatility surface" — the model, not the market, is what the bank would be relying on |
| **Implied vs realised volatility** | Implied is what the market price implies about the future; realised is what actually happened. The gap is the traded edge — and the "priced volatility" of this guide's closing line |
| **DVOL** | Deribit's implied volatility index — the closest thing the crypto options market has to an equity-market VIX |
| **Delta hedging / DDH** | The practice of trading the underlying to keep a book's delta near zero. "Dynamic Delta Hedge" is SignalPlus's automated version |
| **Block trade / Block-RFQ** | A large, privately negotiated trade printed off the order book, typically via a request-for-quote process with several dealers. The institutional channel |
| **Perpetual swap ("perp")** | A futures-like contract with no expiry, kept near spot by funding payments. It dominates crypto derivatives volume and is not an option |
| **CCP (central counterparty)** | The clearing house that stands between buyer and seller and guarantees performance. **Crypto's native options venues have none**; CME does (§7.6) |
| **DMA (direct market access)** | Direct order submission from a firm's own system to an exchange. ⚠ In vendor marketing the term is often used loosely for API access under client credentials — ask which is meant |
| **TCA** | Transaction cost analysis: the measurement of execution quality against a benchmark. ⚠ No TCA evidence for this vendor's algos was found |
| **IPV (independent price verification)** | The second-line process by which a bank independently validates the marks used for valuation — the control at the centre of §10.3 |
| **Model risk** | The risk of adverse consequences from decisions based on incorrect or misused models; governed by inventory, independent validation and change control. A bought model is still the bank's model risk |
| **VATP** | Virtual asset trading platform — the Hong Kong SFC's licensed category under the AMLO regime (§9.2) |
| **AMLO** | Anti-Money Laundering and Counter-Terrorist Financing Ordinance (Cap. 615), the Hong Kong statute under which VATPs are licensed |
| **SFO Type 4 / Type 9** | Hong Kong regulated activities: advising on securities (Type 4) and asset management (Type 9) — the models for the proposed VA advisory and management regimes (§9.3) |
| **MAS / DPT** | Singapore's Monetary Authority and its digital payment token framework under the Payment Services Act — owned by the [MAS guide](mas_regulations_guidelines_guide.md) |
| **Priced volatility** | The implied volatility at which a trade is actually executable — the number that matters when the model is put on the screen and a real order goes in the market |

---

## 15. Cross-References and Further Reading

**Within this repository — the vendor-management and resilience framework (the process owners):**
- [Vendor Management](../management/vendor_management_guide.md) — the general vendor-diligence framework: selection, contracting, VRM/TPRM (§8) and exit (§9). **This guide deliberately does not rebuild it** (§10.1).
- [Operational Resilience Framework](operational_resilience_framework_guide.md) — third-party resilience and the supply chain (§10), business impact analysis, DR tiers and RTO/RPO (§7-§8).

**Sibling vendor deep-dives (the same shelf):**
- [Fireblocks](fireblocks_guide.md) — the digital-asset custody and infrastructure vendor genre; the custody layer of the stack this vendor's software trades on.
- [Bitunix](bitunix_guide.md) — a crypto exchange's own software stack, and the CEX worked-example conventions used here.
- [Market Making in Singapore](market_making_singapore_guide.md) — the market-making firm landscape; the buyers this vendor sells to, and the MAS conduct context.
- [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) — the Singapore licensing regime (§9.4, condensed here).

**Technology foundations (condensed in this guide, not re-derived):**
- [Trading System Software Architecture](../technology/trading_system_software_architecture_guide.md) — matching engines, market-data fan-out and latency budgets (§5.7).
- [Cybersecurity](../technology/cybersecurity_guide.md) — key management, credential custody, ISO/IEC 27001 and SOC reporting (§5.7, §10.6).

**Primary sources used in this pass:** signalplus.com (home, /company) · SignalPlus press releases of 24 Aug 2022 (GlobeNewswire), 4 May 2023, 24 Jan 2025, 16 Sep 2025, 20 Nov 2025 and 1 Jun 2026 (PRNewswire) · HashKey Group newsroom, 20 May 2026 and 27 Aug 2026 · Deribit Insights, 22 Dec 2022 and Deribit's own site, Sep 2026 · AppWorks, "Why We Invested", 27 Mar 2025 · HK Government press release, 26 May 2026 · SFC virtual-assets pages and circular 23EC28 · CoinLaw options-market statistics (updated 27 May 2026) · CoinGlass-derived venue data reported 4 Aug 2026 · CME Group quarterly crypto newsletters, 2026 · vendor sites for Block Scholes, Laevitas, Amberdata, Talos, CoinRoutes and Amber Group, Sep 2026 · registry aggregators for SIGNALPLUS HK Limited.

---

## 16. The Closing Summary

SignalPlus is a Hong Kong software company that sells options pricing, risk analytics, execution and automated hedging to the firms that trade crypto derivatives. It is not an exchange, not a custodian and not a market maker; it is the tooling layer above the venues, which is a narrower and more defensible business than any of those. Founded in 2021 by a former macro and FX-options trader and a serial software entrepreneur, it has raised repeatedly from crypto-native strategic investors, ended up with a listed licensed venue group as its lead backer and commercial partner, and now claims — unaudited, and impossible to reconcile arithmetically in at least one case — US$160bn of quarterly platform volume and a US$500M post-money valuation on undisclosed revenue. The verified facts are fewer than the company's press releases suggest, and the guide has kept them separate: one venue integration confirmed by the venue itself; four rounds announced but two of them ambiguous as to whether they are one event or two; a customer roster that is **reported by the company and not confirmed first-party**; a pricing model that is the whole product and is nowhere documented; and a licensing position that is, correctly, no licensing position at all. For a regulated institution the attraction is real — buying twelve months of options infrastructure for a licence fee instead of a hiring round — and so is the price of it: a model you cannot inspect, a vendor of a few dozen people on whom pricing, risk and 24/7 hedging all depend, trading-enabled API credentials leaving the building, position-level data accumulating at a third party whose own investors include venues, and no published SLA, exit or portability terms. Every one of those gaps is closable by contract, testing and disclosure. None of them closes itself.

The decision is not whether this vendor is good. It is whether your institution can *evidence* that it is, before the desk relies on it — because in this market the only thing you can genuinely verify for yourself, and the only thing that survives a vendor's marketing, a funding announcement and a redrawn regulatory perimeter, is the price at which a real trade actually fills: **"Buy the tooling, validate the number, and never confuse a vendor's model with the market's, because the only figure that cannot be marked up is the priced volatility."**
