# Bitunix: The Fast-Growing Crypto Derivatives Exchange — A Comprehensive Guide

**The Business, History, Products, Licensing, Security, and Technology of Bitunix (bitunix.com) — from a 2021 Founding to 5 Million+ Registered Users, the CoinGlass Top-7 2025 Derivatives-Volume Ranking, Proof of Reserves, the $30 Million Care Fund, and a Cymbal Bank Crypto-Exchange Corporate-Client Worked Example**

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Banking Domain / Crypto-Exchange Company Deep-Dive — the centralized-exchange (CEX) business model, spot and derivatives trading, perpetual swaps, copy trading and earn products, VASP/MSB licensing, the KYC/AML overlay, the MAS digital-payment-token regime comparison, the enforcement record of the exchange industry (Binance, FTX), proof-of-reserves and custody mechanics, and the Cymbal Bank corporate-client lens
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026
> **Companion guides (sibling, same folder — the banking cluster):** [Reap Global](reap_global_guide.md) (the fintech-company profile template and the Cymbal Bank worked-example conventions — cross-ref §10) · [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) (the Singapore licensing regime and the Cymbal Bank persona conventions — cross-ref §7) · [Fircosoft](fircosoft_guide.md) (sanctions/AML screening themes — cross-ref §7, condensed) · [Payment Rails](payment_rails_guide.md) (clearing and settlement mechanics for the fiat legs — cross-ref §8, §10)
> **Companion guides (technology/, prefix `../technology/`):** [Smart Contracts](../technology/smart_contracts_guide.md) (the on-chain mechanics — Merkle proofs, ERC-20, custody and wallet security — cross-ref §8, condensed, not re-derived) · [Cybersecurity](../technology/cybersecurity_guide.md) (the security-discipline themes — hot/cold wallet separation, multi-signature, ISO/IEC 27001, bug bounties — cross-ref §8, condensed)

---

**How to use this guide:** Section 1 is the overview — the short answer, the key-facts table, why a bank should care, and the evidence base. Section 2 is the company profile — the 2021 founding, the founders question (⚠ the founders are not publicly named), and the legal-entity/jurisdiction question (registration in Saint Vincent and the Grenadines, operations reported in the Seychelles — the believed-Seychelles anchor is corrected with flags). Section 3 is the mission and business model — the "smarter, safer, more accessible" mission and the exchange revenue model. Section 4 covers the core products — spot trading, USDT-margined perpetual futures with leverage up to 200x, and the trading surface. Section 5 covers the rest of the roster — copy trading, Bitunix Earn, VIP tiers, and the launchpad/pool question. Section 6 is growth and industry context — the user/volume claims (⚠ company-reported), the CoinGlass/CoinMarketCap/CoinGecko rankings (⚠ where press-estimated), and the competitive landscape versus Binance, Bybit, OKX and peers. Section 7 is licensing and compliance — the MSB/VASP registrations that could be verified, the KYC/AML posture (cross-referencing the Fircosoft guide's screening themes, condensed), and the Singapore/MAS question (cross-referencing the MAS guide). Section 8 is technology — the platform facts that are public (⚠ where not disclosed), the security posture (cold storage, custodians, the Care Fund, Proof of Reserves), and the on-chain mechanics cross-referenced to the smart-contracts guide. Section 9 is the regulatory-headwinds record of the exchange industry — Binance's 2023 resolutions, the FTX collapse, Bybit/OKX market exits, and MiCA — each verified and flagged. Section 10 is the Cymbal Bank worked example — a crypto-exchange corporate client on Bitunix-style flows, following the persona conventions of the repository. Section 11 is the claims audit (✅/⚠/❌). Section 12 is "What Could Not Be Verified". Section 13 is the glossary. Section 14 is cross-references and the closing summary. **The integrity convention of this repository applies throughout:** ✅ = verified this pass at a primary or authoritative source; ⚠ = flagged (company-reported, single-sourced, time-varying, or unverifiable); ❌ = refuted or not found. Nothing is fabricated; where a fact could not be confirmed it is flagged rather than smoothed.

---

## Table of Contents

1. [The Overview](#1-the-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Key-Facts Table](#12-the-key-facts-table)
   - 1.3 [Why This Matters to a Bank](#13-why-this-matters-to-a-bank)
   - 1.4 [The Evidence Base at a Glance](#14-the-evidence-base-at-a-glance)
2. [The Company Profile — Founding, Founders, and Legal Entity](#2-the-company-profile--founding-founders-and-legal-entity)
   - 2.1 [The Founding Year: 2021](#21-the-founding-year-2021)
   - 2.2 [The Founders: Not Publicly Named](#22-the-founders-not-publicly-named)
   - 2.3 [The Legal Entity and Jurisdiction: SVG Registration, Seychelles Operations](#23-the-legal-entity-and-jurisdiction-svg-registration-seychelles-operations)
   - 2.4 [The Company Table](#24-the-company-table)
   - 2.5 [The Footprint Timeline](#25-the-footprint-timeline)
3. [The Mission and Business Model](#3-the-mission-and-business-model)
   - 3.1 [The Mission Statement](#31-the-mission-statement)
   - 3.2 [The Business Model — an Exchange's Revenue Stack](#32-the-business-model--an-exchanges-revenue-stack)
   - 3.3 [The Model's Market Context](#33-the-models-market-context)
4. [The Products — Spot, Futures, and Derivatives](#4-the-products--spot-futures-and-derivatives)
   - 4.1 [Spot Trading](#41-spot-trading)
   - 4.2 [Futures and Perpetual Swaps](#42-futures-and-perpetual-swaps)
   - 4.3 [The Trading Surface — Charts, TradingView, and Order Types](#43-the-trading-surface--charts-tradingview-and-order-types)
   - 4.4 [Trading Pairs, Volumes, and Fees](#44-trading-pairs-volumes-and-fees)
   - 4.5 [The Product-Surface Map](#45-the-product-surface-map)
5. [The Products — Copy Trading, Earn, and the Rest of the Roster](#5-the-products--copy-trading-earn-and-the-rest-of-the-roster)
   - 5.1 [Copy Trading](#51-copy-trading)
   - 5.2 [Bitunix Earn — Savings, Staking, and Dual Investment](#52-bitunix-earn--savings-staking-and-dual-investment)
   - 5.3 [VIP Tiers and the Launchpad/Pool Question](#53-vip-tiers-and-the-launchpadpool-question)
   - 5.4 [Fiat On/Off-Ramps and P2P](#54-fiat-onoff-ramps-and-p2p)
6. [Growth, Rankings, and the Competitive Landscape](#6-growth-rankings-and-the-competitive-landscape)
   - 6.1 [The Growth Claims](#61-the-growth-claims)
   - 6.2 [The Rankings — CoinGlass, CoinMarketCap, CoinGecko, DefiLlama](#62-the-rankings--coinglass-coinmarketcap-coingecko-defillama)
   - 6.3 [The Competitive Landscape — Binance, Bybit, OKX, and the Mid-Tier](#63-the-competitive-landscape--binance-bybit-okx-and-the-mid-tier)
   - 6.4 [The Competitive-Comparison Table](#64-the-competitive-comparison-table)
7. [Licensing and Compliance](#7-licensing-and-compliance)
   - 7.1 [The Verified Registrations — FinCEN MSB, FINTRAC, AUSTRAC, and the 2026 Footprint](#71-the-verified-registrations--fincen-msb-fintrac-austrac-and-the-2026-footprint)
   - 7.2 [The Unverified Licensing Claims — Lithuania, the Seychelles, and the Philippines](#72-the-unverified-licensing-claims--lithuania-the-seychelles-and-the-philippines)
   - 7.3 [The KYC/AML Posture (Cross-Referenced, Condensed)](#73-the-kycaml-posture-cross-referenced-condensed)
   - 7.4 [The Singapore Angle — MAS and the DPT Regime (Cross-Referenced)](#74-the-singapore-angle--mas-and-the-dpt-regime-cross-referenced)
   - 7.5 [The Licensing Table](#75-the-licensing-table)
8. [Technology — Platform, Security, and Proof of Reserves](#8-technology--platform-security-and-proof-of-reserves)
   - 8.1 [The Platform Facts That Are Public](#81-the-platform-facts-that-are-public)
   - 8.2 [The Security Posture — Custody, Cold Storage, and the Care Fund](#82-the-security-posture--custody-cold-storage-and-the-care-fund)
   - 8.3 [Proof of Reserves and the On-Chain Mechanics (Cross-Referenced)](#83-proof-of-reserves-and-the-on-chain-mechanics-cross-referenced)
   - 8.4 [What Is Not Disclosed](#84-what-is-not-disclosed)
   - 8.5 [The Verified Technology Facts Table](#85-the-verified-technology-facts-table)
9. [Regulatory Headwinds — The Industry Enforcement Record](#9-regulatory-headwinds--the-industry-enforcement-record)
   - 9.1 [Binance — The November 2023 Resolutions](#91-binance--the-november-2023-resolutions)
   - 9.2 [FTX — The November 2022 Collapse](#92-ftx--the-november-2022-collapse)
   - 9.3 [Bybit, OKX, and the Market-Exit Pattern](#93-bybit-okx-and-the-market-exit-pattern)
   - 9.4 [MiCA and the EU Regime](#94-mica-and-the-eu-regime)
   - 9.5 [What the Record Means for Bitunix](#95-what-the-record-means-for-bitunix)
10. [The Cymbal Bank Worked Example — A Crypto-Exchange Corporate Client](#10-the-cymbal-bank-worked-example--a-crypto-exchange-corporate-client)
    - 10.1 [The Scenario](#101-the-scenario)
    - 10.2 [Onboarding and the KYC/AML Overlay](#102-onboarding-and-the-kycaml-overlay)
    - 10.3 [The Banking Services and Settlement Flows](#103-the-banking-services-and-settlement-flows)
    - 10.4 [The Risk and Monitoring Overlay](#104-the-risk-and-monitoring-overlay)
    - 10.5 [The Lessons](#105-the-lessons)
11. [The Claims Audit — Verified, Flagged, Rejected](#11-the-claims-audit--verified-flagged-rejected)
    - 11.1 [The Verified Claims (✅)](#111-the-verified-claims-)
    - 11.2 [The Flagged Claims (⚠)](#112-the-flagged-claims-)
    - 11.3 [The Rejected or Not-Found Claims (❌)](#113-the-rejected-or-not-found-claims-)
12. [What Could Not Be Verified](#12-what-could-not-be-verified)
13. [The Glossary](#13-the-glossary)
14. [Cross-References and the Closing Summary](#14-cross-references-and-the-closing-summary)

---

## 1. The Overview

### 1.1 The Short Answer

**Bitunix** (bitunix.com) is a centralized cryptocurrency exchange (CEX) that offers spot trading, USDT-margined perpetual futures, copy trading, and earn products to a reported **5 million+ registered users across 150+ countries** (⚠ company-reported, H1 2026). The company states that it was **established in 2021** and launched globally in October 2022 (bitunix.com/about-us; CoinMarketCap exchange profile). It is **registered in Saint Vincent and the Grenadines (SVG)** — not, as the research brief believed, in the Seychelles — with operations reported in the Seychelles (⚠ secondary-source only; see §2.3). The **founders are not publicly named** in any primary source examined (⚠, §2.2).

Bitunix's positioning is the **derivatives-first mid-tier exchange**: up to 200x leverage on perpetual futures, deep-liquidity claims (slippage under 0.01% on BTC/ETH — ⚠ company claim), TradingView Pro integration, and an aggressive growth narrative — CoinGlass's 2025 Annual Report ranked Bitunix **No. 7 by derivatives trading volume and No. 10 by open interest** for 2025 (bitunix.com blog, citing coinglass.com/learn/2025-annual-report-en), and the company claims over **US$300 billion in H1 2026 trading volume** with a peak daily volume of US$5.34 billion (⚠ company-reported). Its compliance surface includes FinCEN MSB and FINTRAC registrations (⚠ as reported by third-party review; see §7.1), 2026 registrations in Canada, Australia, South Africa and Mauritius (company blog ✅ as published), mandatory KYC in two tiers powered by Sumsub, a Merkle-tree Proof of Reserves, and a **US$30 million "Bitunix Care Fund"** (company claims — the Care Fund figure has evolved from earlier US$5 million insurance-fund reporting; see §8.2).

For a bank like Cymbal Bank, Bitunix matters on four fronts: as a **counterparty class** (crypto exchanges are among the most risk-intensive corporate clients a bank can onboard — the worked example in §10 shows why), as a **regulatory weathervane** (an SVG-registered, derivatives-first exchange navigating MSB/VASP registrations, restricted jurisdictions including Singapore, and the post-Binance/FTX enforcement climate), as a **technology benchmark** (proof-of-reserves, custody partnerships, and the on-chain transparency mechanics that banks increasingly must read), and as a **worked-example generator** for the repository's Cymbal Bank persona conventions.

### 1.2 The Key-Facts Table

| Aspect | Fact | Status |
| --- | --- | --- |
| Full name / brand | Bitunix (bitunix.com); no operating legal-entity name published on the site | ✅ brand / ⚠ entity |
| Founded | 2021 ("established in 2021"; global launch October 2022) | ✅ (company + CoinMarketCap) |
| Founders | Not publicly named in any primary source examined | ⚠ |
| Registration jurisdiction | Saint Vincent and the Grenadines (SVG) | ✅ (CoinMarketCap, Tracxn, NewsBTC) |
| Operations base | Reported in the Seychelles (secondary sources only) | ⚠ |
| Users | 5M+ registered across 150+ countries (H1 2026) | ⚠ company-reported |
| Trading pairs | 1,200+ claimed (2026); ~503 live pairs on CoinGecko (Sep 2026 snapshot) | ⚠/✅ |
| Leverage | Up to 200x on perpetual futures | ✅ company claim / ⚠ per-product |
| H1 2026 volume | > US$300B; peak daily volume US$5.34B | ⚠ company-reported |
| 2025 rankings | CoinGlass No. 7 derivatives volume, No. 10 open interest | ✅/⚠ (as published by CoinGlass via company blog) |
| Registrations | FinCEN MSB (US), FINTRAC (Canada) — third-party reported; AUSTRAC (Australia), MSB (Canada), FSP (South Africa), Investment Dealer (Mauritius) — company blog, H1 2026 | ⚠/✅ |
| KYC | Mandatory, two tiers (KYC 1 / KYC 2), Sumsub-powered | ✅ (company blog) |
| Proof of Reserves | Merkle-tree PoR; July 2026 ratios BTC 110%, ETH 104%, USDT 104%; audited reserve value ~US$192.8M (CMC-sourced) | ✅/⚠ company + aggregator |
| Protection fund | "Bitunix Care Fund" — US$30M USDC claimed (2026); earlier US$5M insurance-fund reporting (2024–2025) | ⚠ company-reported |
| Custody | Cobo and Fireblocks (institutional custody partners); hot/cold wallet separation; multi-signature | ✅ company claim |
| Singapore | Singapore is a restricted jurisdiction; no MAS licence identified; no Singapore-directed services | ⚠/❌ |

### 1.3 Why This Matters to a Bank

A Bitunix-style exchange is the **archetype of the crypto-exchange corporate client** a bank's financial-crime, credit, and product teams must be able to assess: it is a high-volume, high-velocity money-movement business incorporated in a light-touch jurisdiction, serving retail derivatives traders globally, holding customer assets in custody, and operating under a patchwork of registrations rather than a single comprehensive licence. The banking questions are the ones §10 works through: How do you onboard a VASP whose ultimate beneficial owners are not public? How do you screen, monitor, and settle for a client whose daily flows include on-chain transfers, P2P fiat, and derivatives margin movements? How do you document the Singapore restriction — a jurisdiction where the exchange does not hold an MAS licence and does not serve residents (cross-ref [MAS Regulations](mas_regulations_guidelines_guide.md))? And how do you read the exchange's own transparency machinery — Proof of Reserves, custody attestations, the Care Fund — as evidence in a bank's third-party-risk process rather than as marketing? These are exactly the questions the Cymbal Bank worked example in §10 answers.

### 1.4 The Evidence Base at a Glance

Every factual claim in this guide traces to one of five evidence classes, and the claims audit (§11) records which class supports each claim:

| Evidence class | Examples used in this guide | How it is treated |
| --- | --- | --- |
| Company pages and blogs (bitunix.com) | About Us; H1 2026 Growth Review (Jul 2026); Compliance/KYC blog (2026); CoinGlass-ranking blog (Jun 2026); Proof of Reserves page | ✅ where the fact is the page's own content; ⚠ where the page reports company metrics |
| Market-data aggregators | CoinMarketCap exchange profile (founded 2021, launched Oct 2022, SVG registration); CoinGecko exchange page (503 pairs, 24h volume snapshot, exchange reserves); Tracxn profile (SVG, founded 2021) | ✅ for registry-style facts; ⚠ for time-varying volumes |
| Independent press and review | NewsBTC review (Oct 2025 — SVG registration, Seychelles operations, FinCEN/FINTRAC/Philippines VASP claims, US/Singapore restrictions, 2M users, $5B daily volume) | ✅/⚠ — press-reported facts flagged where single-sourced |
| Regulator and enforcement records | US DOJ press release on Binance (Nov 21, 2023); Reuters/AP/NPR on the same; the CoinGlass 2025 Annual Report (via company blog) | ✅ for the enforcement events; ⚠ for derived estimates |
| The research brief itself | Believed Seychelles registration; believed Lithuanian VASP; ~1M+ users and hundreds-of-millions USD 24h volume | ⚠/❌ where unconfirmable or refuted — see §11.3 and §12 |

The methodological limitation of this pass is stated once here: registry lookups that require interactive search (the SVG International Business Companies registry, the Seychelles FSA register, FinCEN's MSB registrant list, FINTRAC's MSB list, the MAS Financial Institutions Directory) could not be queried directly, so every licensing finding that depends on them is flagged in §12 rather than asserted as proof.

---

## 2. The Company Profile — Founding, Founders, and Legal Entity

### 2.1 The Founding Year: 2021

The founding year of **2021** verifies cleanly across primary and independent sources:

- The company's own About page: "**Bitunix was established in 2021** and is one of the fastest-growing cryptocurrency spot and derivatives exchanges in the world, serving over 150 countries and boasting more than 5 million users" (bitunix.com/about-us, extracted September 2026). ✅
- The company's H1 2026 Growth Review repeats it: "Since its inception in 2021, Bitunix has been driven by a singular, unwavering mission" (bitunix.com/hub/blog/bitunix-features/bitunix-h1-2026-growth-review, July 8, 2026). ✅
- CoinMarketCap's exchange profile: "Bitunix was **founded in 2021 and launched globally in October 2022**" (coinmarketcap.com/exchanges/bitunix/). ✅ — this gives the useful distinction between incorporation (2021) and the global product launch (October 2022).
- CoinGecko's exchange page: "Bitunix is a centralized cryptocurrency exchange **established in 2021**" (coingecko.com/en/exchanges/bitunix). ✅
- Tracxn's company profile: "Bitunix is a funded company based in Saint Vincent And The Grenadines, **founded in 2021**" (tracxn.com/d/companies/bitunix). ✅
- A consistency check from the company's own anniversary blog: the June 2026 CoinGlass-ranking post celebrates the "**4th anniversary**" (bitunix.com/hub/blog/bitunix-features/bitunix-ranked-top-10-open-interest-top-7-volume-coinglass-2025, June 2026), which is consistent with a 2021–2022 founding-to-launch arc. ✅

The believed anchor for this guide — "founded ~2021" — is therefore confirmed; the more precise reading is **incorporated/established 2021, globally launched October 2022**.

### 2.2 The Founders: Not Publicly Named

The research brief asked to verify the founders and to flag where they are not publicly named. **The founders are not publicly named in any primary source examined for this guide.** ⚠

- Bitunix's About page names no founder, CEO, or leadership team member; the "Our Story" section is anonymous corporate copy ("Guided by the mission of making cryptocurrency trading smarter, safer, and more accessible...", bitunix.com/about-us). ⚠
- No company press release, blog post, or support article examined in this pass names a founder or executive. ⚠
- Targeted web searches pairing "Bitunix" with "founder", "CEO", "co-founder", and "team" returned no corroborating naming (search backend returned empty result sets for these queries — an absence of evidence, recorded as ⚠ rather than ❌, because the individuals may simply be low-profile). ⚠
- Aggregator profiles are equally silent: Tracxn lists no founders; PitchBook's profile describes the company generically without naming management. ⚠

This anonymity is itself a material fact for a bank: a corporate client whose ultimate beneficial owners are not publicly identifiable requires the bank to complete UBO identification through the client's own registry extract and declarations rather than through public sources — the exact problem the Cymbal Bank worked example in §10 addresses.

### 2.3 The Legal Entity and Jurisdiction: SVG Registration, Seychelles Operations

The research brief believed the legal entity was registered in the Seychelles. **The verified record is different: the company is registered in Saint Vincent and the Grenadines (SVG), with operations reported in the Seychelles.** The believed anchor is therefore corrected with flags:

- **CoinMarketCap:** "Bitunix is a global exchange, **registered in Saint Vincent and the Grenadines (SVG)**" (coinmarketcap.com/exchanges/bitunix/). ✅
- **Tracxn:** "based in **Saint Vincent and the Grenadines**" (tracxn.com). ✅
- **CoinGecko:** "registered in **Saint Vincent and the Grenadines**" (coingecko.com/en/exchanges/bitunix). ✅
- **NewsBTC review (October 2025):** "Bitunix is a centralized cryptocurrency exchange **registered in Saint Vincent and the Grenadines, with operations in the Seychelles**" (newsbtc.com/bitunix-review/). ✅ for the SVG registration (corroborated by the aggregators); ⚠ for the Seychelles operations claim, which is single-sourced in this pass.

Two things could **not** be verified and are flagged rather than guessed: (1) the **operating legal-entity name(s)** — bitunix.com publishes no entity name, company number, or registered address on its About, Terms, or support pages examined this pass ⚠; and (2) the **registry records themselves** — the SVG International Business Companies (IBC) registry and the Seychelles Financial Services Authority register are not freely queryable online, so the registration could not be confirmed at the registry level ⚠ (see §12). SVG is a common incorporation venue for crypto exchanges (an IBC regime with light-touch disclosure); the Seychelles is likewise a common operational base for crypto businesses. Both are flagged as jurisdictions of convenience rather than as regulated financial centres — a distinction that matters in §7 and §10.

### 2.4 The Company Table

| Aspect | Verified fact | Status |
| --- | --- | --- |
| Brand | Bitunix (bitunix.com) | ✅ |
| Established | 2021 | ✅ (company, CoinMarketCap, CoinGecko, Tracxn) |
| Global launch | October 2022 | ✅ (CoinMarketCap) |
| Founders | Not publicly named | ⚠ |
| Registration | Saint Vincent and the Grenadines | ✅ (CoinMarketCap, CoinGecko, Tracxn, NewsBTC) |
| Operations base | Reported in the Seychelles | ⚠ (NewsBTC, single-sourced) |
| Operating entity name | Not published on the site | ⚠ |
| Workforce | 800+ professionals (H1 2026 claim) | ⚠ company-reported |
| Support languages | 18+ languages; 24/7 support (H1 2026 claim) | ⚠ company-reported |
| Users | 5M+ across 150+ countries (2026); 2M+ (CoinMarketCap profile); 3M+ (June 2026 blog boilerplate) | ⚠ company-reported, evolving |

The user-count evolution is itself a data point: the CoinGlass-ranking blog (June 2026) still carried the boilerplate "trusted by over 3 million users across more than 100 countries," while the H1 2026 Growth Review (July 2026) and the current About page claim 5M+ across 150+ countries — a jump of roughly two million users within weeks, or a boilerplate refresh, or both. All three figures are company-reported and single-sourced ⚠.

### 2.5 The Footprint Timeline

| Period | Milestone | Source |
| --- | --- | --- |
| 2021 | Established | About page; CoinMarketCap; CoinGecko; Tracxn ✅ |
| Oct 2022 | Global launch | CoinMarketCap ✅ |
| 2023 | "Began drawing broader attention on CoinMarketCap and CoinGecko" | Company CoinGlass blog ⚠ |
| 2024 | Similarweb: No. 3 globally for visit growth — 46.4M visits, 472% YoY | Similarweb via company blog ⚠ |
| Oct 2025 | NewsBTC review: 2M users; US$5B+ daily volume; FinCEN/FINTRAC/Philippines-VASP claims; 125x leverage; >95% cold storage | NewsBTC ⚠/✅ |
| Mar 3, 2026 | PoR update: US$128.5M+ assets; BTC 109% / ETH 100% / USDT 109% | Company PoR press copies ⚠ |
| May 2026 | Multi-aggregator rankings compiled (CMC #37/#30; CoinGecko #13/#21; CryptoRank #26/#10; CoinGlass #10; CoinRanking #27; Messari #35) | H1 2026 blog ⚠ company-curated |
| Jun 2026 | CoinGlass 2025 Annual Report blog (No. 7 volume / No. 10 open interest); 4th-anniversary "Ultra 4ward" campaign (4M USDT) | Company blog ✅/⚠ |
| Jul 2026 | H1 2026 review: 5M users; US$300B+ volume; US$5.34B peak daily; PoR US$192.8M (110/104/104); ISO/IEC 27001:2022; Canada/Australia/South Africa/Mauritius registrations | Company blog ⚠/✅ |
| Sep 2026 | CoinGecko live snapshot: 503 pairs; ~US$293.7M 24h volume; ~US$256.7M reserves | CoinGecko ✅/⚠ (time-varying) |

The timeline reads as the standard mid-tier exchange growth arc: quiet 2021–2022 founding and launch, 2023–2024 visibility growth, 2025 aggregator-rank recognition, and a 2026 marketing-and-compliance push — with every quantitative cell after October 2022 carrying a ⚠.

---

## 3. The Mission and Business Model

### 3.1 The Mission Statement

Bitunix's mission, as stated on its About page and repeated in its 2026 reviews: **"making cryptocurrency trading smarter, safer, and more accessible"** (bitunix.com/about-us; bitunix-h1-2026-growth-review). ✅ (as published — the mission is the company's own claim, verified only as its stated mission). The supporting vision statement is asset-security-centric: **"Take 'User asset security first' as its core... Let your assets be protected by world-class standards"** (About page). ✅ (as published). The company's slogan line is "**better liquidity means better trading**," and its tagline on the About page banner is "Empowering Traders with Smarter and Safer Crypto Solutions." ✅ (as published).

### 3.2 The Business Model — an Exchange's Revenue Stack

Bitunix has never published a financial statement, so its revenue model must be assembled from disclosed fragments (⚠ where inferred):

- **Trading fees.** The core revenue line. The fee schedule published in the NewsBTC review (October 2025) shows a VIP-tiered maker/taker structure: at VIP 0, spot maker/taker fees of 0.0800%/0.1000% and futures maker/taker fees of 0.0200%/0.0600%, declining to spot 0.0100%/0.0325% and futures 0.0060%/0.0300% at VIP 7 (newsbtc.com/bitunix-review/). ⚠ (press-reported schedule; the company's own fee page exists — "Bitunix Trading Fees Explained," July 2026 — but was not extractable this pass). The futures-taker discount relative to spot taker (0.06% vs 0.10% at VIP 0) is the classic derivatives-first pricing posture: attract high-velocity futures flow with cheaper execution and monetize it through volume. ⚠/✅ as structure.
- **Withdrawal fees.** Per-transaction network-cost recovery: e.g., BTC US$0.000035, ETH (ERC-20) US$0.00175, LTC US$0.002 (NewsBTC, October 2025 ⚠). Deposits are free.
- **Derivatives economics.** Funding rates on perpetual swaps, liquidation fees, and the spread on the tiered-risk-limit book are standard exchange revenue lines; Bitunix does not disclose these numbers ⚠ (domain knowledge, flagged).
- **Earn-product spreads.** Bitunix Earn (Easy Earn savings, Dual Investment) pays users APRs (e.g., USDT 3%–11.6%, USDC 1.2%–8.6%, ETH 1.3%–9.3%, BTC 0.7%–5% — NewsBTC ⚠) and earns the difference between the yield it generates on deposited assets and what it pays out ⚠ (inferred).
- **Listing and campaign economics.** Token Splash (deposit/trading tasks for newly listed tokens), the Task Center, and the anniversary reward campaigns (e.g., the 4M USDT "Ultra 4ward" campaign) monetize listings and user acquisition — the standard mid-tier exchange playbook of paying for liquidity and attention ⚠ (inferred).
- **VIP program.** VIP tiers (0–7) tied to trading volume and holdings; the company claims its VIP user base grew 15% quarter-on-quarter in H1 2026 ⚠ (company-reported).

The honest summary: the model is a classic **fee-for-order-flow exchange** — derivatives-led, retail-focused, growth-funded by aggressive campaigns — but none of the economics (revenue, margin, funding costs) is publicly audited. ⚠

### 3.3 The Model's Market Context

Bitunix's own materials anchor the model in the derivatives market's scale: the CoinGlass-ranking blog cites Cointelegraph/CoinGlass reporting that **total crypto derivatives trading volume reached about US$85.7 trillion in 2025, averaging ~US$264.5 billion per day, with peak days near US$748.3 billion** (cointelegraph.com/news/crypto-derivatives-86t-2025-binance-volume-coinglass, as cited in the Bitunix blog). ⚠ (industry estimate, company-curated; the headline is verified as published, the methodology is CoinGlass's). The company also cites Similarweb's finding that bitunix.com ranked **No. 3 globally for visit growth in 2024 vs 2023, at 46.4 million visits and 472% YoY growth** (company blog citing similarweb.com) ⚠. Both figures are recorded as the market narrative Bitunix operates with, not as verified facts about its own economics.

---

## 4. The Products — Spot, Futures, and Derivatives

### 4.1 Spot Trading

Bitunix's spot market is the entry surface of the exchange, verified from company pages and the NewsBTC hands-on review:

- **Asset and pair counts.** The company claims "over 1,200 trading pairs" (About page and H1 2026 blog, 2026) ⚠; the NewsBTC review (October 2025) reported "over 300 assets and 700+ trading pairs" ⚠; the live CoinGecko snapshot at the time of this pass listed **503 coins and 503 trading pairs**, with BTC/USDT the most active pair (coingecko.com/en/exchanges/bitunix) ✅/⚠ (time-varying aggregator data). The discrepancy between company claims and aggregator counts is normal (aggregators list only pairs meeting their listing criteria) but is flagged rather than reconciled: the company's 1,200+ figure is a marketing claim. ⚠
- **USDT-quoted book.** The NewsBTC review noted that "all trades are tied to USDT" — a single-quote-currency design that simplifies liquidity but limits base-currency diversification (newsbtc.com/bitunix-review/). ⚠ (hands-on observation, press-reported).
- **Spot Auto-Invest.** A dollar-cost-averaging (DCA) feature — recurring purchases on a schedule — exists and was tested by the reviewer. ✅/⚠ (feature existence press-verified; no company spec sheet extracted this pass).
- **Flash swaps.** The H1 2026 blog credits "instant flash swap integrations" as a driver of spot participation ⚠ (company claim).

### 4.2 Futures and Perpetual Swaps

Derivatives are Bitunix's specialty and the product the entire growth narrative hangs on:

- **Product type.** The exchange offers **perpetual futures only** — no dated futures, no options, no swaps — per the NewsBTC hands-on review ("Bitunix currently offers only perpetual futures... no options or swaps," October 2025) ⚠ (as of that date; the product set may have evolved). ✅/⚠
- **Margin and leverage.** Contracts are **USDT-margined**. The company's current materials claim "**up to 200x leverage**" (About page, H1 2026 blog) ✅ (as published); the NewsBTC review reported "up to 125x" (October 2025) ⚠. The two figures are flagged rather than reconciled — either the maximum tier was raised, or different contracts carry different caps; per-contract maxima are not published in one place. ⚠
- **Risk controls.** A **tiered risk limit system** applies to all futures trading (Bitunix support article, "Bitunix Futures Liquidation Mechanism and Tiered Risk Limit," linked from the NewsBTC review) ✅/⚠ (support article exists; contents not fully extracted this pass). Order types include market, limit, trigger, and trailing stop, plus stop-loss, take-profit, and reverse orders (NewsBTC hands-on ⚠).
- **Contract breadth.** "Over 300 futures contracts" (NewsBTC, October 2025) ⚠.
- **Derivatives volume position.** This is the product line that produced the CoinGlass 2025 rankings (No. 7 by derivatives volume, No. 10 by open interest — §6.2) and the claimed US$300B+ H1 2026 volume. ✅/⚠ as ranked/claimed.

### 4.3 The Trading Surface — Charts, TradingView, and Order Types

The trading experience is the exchange's differentiation claim, assembled from company pages and the hands-on review:

- **TradingView Pro integration** on web and mobile (About page; company blog "Bitunix x TradingView Integration," June 2026) ✅ (as published). The NewsBTC review confirmed TradingView integration with 20+ technical indicators and multi-chart functionality. ✅/⚠
- **Ultra K-Line chart system** — the company's branded charting surface ("the most friendly and efficient trading experience with comprehensive tools," About page) ✅ (as published).
- **Multi-chart windows** — up to 16 charts within a single window (NewsBTC ⚠).
- **Matching-engine claims.** "Millisecond-level matching" and "mainstream coins have a slippage of less than 0.01%" (About page) ⚠ (company claims; the H1 2026 blog repeats the sub-0.01% slippage claim for BTC/ETH ⚠). No independent latency benchmark exists ⚠.
- **Mobile apps** for iOS and Android with the core surfaces (spot, futures, copy trading, account management); the reviewer tested the app and reported it responsive (NewsBTC ⚠).
- **Newer tooling.** H1 2026 proprietary tools: **Super Alert** (multi-condition market monitoring combining up to 5 technical parameters), **Fixed Risk**, and **Multi-Trade** (H1 2026 blog ⚠ company claims).

### 4.4 Trading Pairs, Volumes, and Fees

- **Volumes.** The live CoinGecko snapshot at this pass's search time reported **24h volume of ~US$293.7 million** and **exchange reserves of ~US$256.7 million** (coingecko.com/en/exchanges/bitunix, September 2026) ⚠ (time-varying aggregator data). The company's own materials claim **US$5 billion average daily trading volume** (About page) and a **US$5.34 billion peak daily volume** in H1 2026 (H1 2026 blog) ⚠. The gap is expected — aggregators capture spot volume, while Bitunix's volume is derivatives-led — but the company's headline volume figure is not independently auditable and is marked ⚠ throughout.
- **Fees.** VIP-tiered maker/taker schedule (§3.2): VIP 0 spot 0.08%/0.10%, futures 0.02%/0.06%, down to VIP 7 spot 0.01%/0.0325%, futures 0.006%/0.03% (NewsBTC, October 2025) ⚠ press-reported. The company's own fee-education blog exists (July 2026) but was not extractable this pass ⚠.

### 4.5 The Product-Surface Map

The product surface, compiled from the About page, the H1 2026 blog, and the NewsBTC hands-on review — the full roster a bank would map before onboarding the exchange as a client:

| Product line | One-line description | Status |
| --- | --- | --- |
| Spot trading | Immediate buy/sell of crypto; USDT-quoted book; 503–1,200+ pairs depending on source | ✅/⚠ (§4.1) |
| Futures (perpetual swaps) | USDT-margined perpetuals; up to 200x leverage (125x per Oct 2025 review); tiered risk limits; no options/dated futures | ✅/⚠ (§4.2) |
| Copy trading | Mirror lead traders' futures positions; weekly profit sharing; leader ROI/PnL stats | ✅/⚠ (§5.1) |
| Bitunix Earn | Easy Earn flexible savings; fixed-term; Dual Investment; APRs (USDT 3–11.6%, etc.) | ✅/⚠ (§5.2) |
| Spot Auto-Invest | DCA recurring purchases | ✅/⚠ (§4.1) |
| P2P marketplace | 76 fiat currencies supported | ✅ as published (§5.4) |
| Third-party fiat deposits | 81 fiat currencies supported | ✅ as published (§5.4) |
| Token Splash | New-listing campaigns (deposit and trading tasks) | ✅/⚠ (§5.3) |
| Task Center | Reward tasks; Futures Volume challenge | ⚠ (§5.3) |
| VIP tiers | VIP 0–7 fee discounts tied to volume | ⚠ (§5.3) |
| TradingView Pro | Charting integration on web and mobile | ✅/⚠ (§4.3) |
| Ultra K-Line | Branded charting system | ✅ as published (§4.3) |
| Super Alert / Fixed Risk / Multi-Trade | H1 2026 proprietary trading tools | ⚠ (§4.3) |

Two absences are notable: no formal **launchpad/IDO** surface and no formal **on-chain staking** product appear anywhere in the verified roster — the believed anchors for both resolve to Token Splash and Earn respectively (§5.2, §5.3, §11.3). ⚠

---

## 5. The Products — Copy Trading, Earn, and the Rest of the Roster

### 5.1 Copy Trading

Copy trading is one of Bitunix's four headline product lines (spot, futures, copy trading, Earn — per the About page) and is futures-focused:

- **Mechanics.** Followers select a lead trader, allocate funds, and mirror the leader's futures positions; the company publishes a guide covering "follower settings, lead trader rules, crypto futures copy trading execution, slippage limits, and **weekly profit sharing**" (bitunix.com blog, August 2026 — title verified) ✅/⚠ (guide exists; the profit-sharing rate itself is not extracted).
- **Trader transparency.** Leader profiles show ROI, profit/loss, follower counts, and history — the NewsBTC reviewer confirmed the feature worked and praised the statistics (newsbtc.com/bitunix-review/ ⚠).
- **Caveats.** The company's own marketing and the review both note that past performance does not guarantee future results — the standard copy-trading risk framing. ✅ (as published).

### 5.2 Bitunix Earn — Savings, Staking, and Dual Investment

The Earn product family covers the believed anchor's "staking" item, with a naming nuance:

- **Easy Earn (flexible savings).** Idle crypto deposited for interest. APRs published in the NewsBTC review: USDC 1.2%–8.6%, USDT 3%–11.6%, ETH 1.3%–9.3%, BTC 0.7%–5%, BARD 1%–10% (newsbtc.com/bitunix-review/, October 2025) ⚠ (press-reported, time-varying rates).
- **Fixed-term investment.** The About page mentions "flexible savings and fixed-term investment solutions" ✅ (as published); terms and rates are not published in one place ⚠.
- **Dual Investment.** A structured product — e.g., deposit USDT with a target BTC price and a settlement date; if the market doesn't hit the target you keep the USDT plus yield, if it does you are converted into BTC plus yield (NewsBTC hands-on ⚠). Classic dual-currency structured product mechanics.
- **The "staking" question.** The believed anchor listed staking as a product. The company's materials examined this pass describe **savings/earn rather than on-chain staking** (no validator-staking product, no staking APR schedule, no supported-proof-of-stake asset list was found). "Staking" in the loose marketing sense is covered by Earn; a formal staking product could not be verified ⚠ (see §12).
- **Spot Auto-Invest** (§4.1) is also grouped under the passive-income surface by the company and the review.

### 5.3 VIP Tiers and the Launchpad/Pool Question

- **VIP program.** Eight tiers (VIP 0–7) with fee discounts tied to volume (§3.2, §4.4) and, per the H1 2026 blog, 15% QoQ VIP-user growth ⚠ (company claim). The program is standard exchange loyalty economics; the qualification thresholds are not published in one place ⚠.
- **The launchpad/pool question.** The believed anchor mentioned a launchpad/pool. The verified analogue is **Token Splash** — the new-listing promotion surface with Deposit and Trading tasks for featured tokens (NewsBTC hands-on; company blog references) ✅/⚠ — plus the **Task Center** (rewards for deposit/trade tasks and the Futures Volume challenge with real-world prizes, per NewsBTC ⚠). A formal launchpad (token-sale/IDO platform) as seen on Binance Launchpad or Bybit Launchpool could **not** be verified ⚠ (see §12).

### 5.4 Fiat On/Off-Ramps and P2P

- **P2P marketplace** supporting **76 fiat currencies**, and **third-party deposits** supporting **81 fiat currencies** (About page, extracted September 2026) ✅ (as published; the third-party payment-provider names are not listed ⚠).
- The NewsBTC review confirms fiat deposit options exist and that deposits are free, with withdrawal fees varying by asset and network (§3.2). ⚠/✅
- Fiat rails matter for the banking story: P2P and third-party deposit rails are where the exchange touches the traditional payments system — the subject of §7 and the §10 worked example.

---

## 6. Growth, Rankings, and the Competitive Landscape

### 6.1 The Growth Claims

All growth metrics are company-reported or press-reported and none is audited. Collected for the audit (§11):

| Metric | Figure | Period | Source |
| --- | --- | --- | --- |
| Registered users | 2M+ | 2024–2025 era | CoinMarketCap exchange profile ⚠ |
| Registered users | 2M users; US$5B+ daily volume (from US$1B in 2024) | Oct 2025 | NewsBTC review ⚠ |
| Registered users | 3M+ across 100+ countries | Jun 2026 | Bitunix CoinGlass blog boilerplate ⚠ |
| Registered users | 5M+ across 150+ countries | Jul 2026 | H1 2026 Growth Review ⚠ |
| Trading volume | > US$300B (H1 2026); peak daily US$5.34B | Jul 2026 | H1 2026 Growth Review ⚠ |
| Average daily volume | US$5B+ (site claim) | 2026 | About page ⚠ |
| Synthetic/tokenized TradFi volume | US$6.44B total; US$180M daily | H1 2026 | H1 2026 Growth Review ⚠ |
| Web traffic growth | 46.4M visits; 472% YoY; No. 3 globally | 2024 vs 2023 | Similarweb via Bitunix blog ⚠ |
| Workforce | 800+ professionals | H1 2026 | H1 2026 Growth Review ⚠ |
| KOL/affiliate network | 2,000+ members | H1 2026 | H1 2026 Growth Review ⚠ |
| VIP users | +15% QoQ | H1 2026 | H1 2026 Growth Review ⚠ |
| Product releases | 90+ features in H1 2026 | 2026 | H1 2026 Growth Review ⚠ |

The only externally checkable growth datapoints in the entire record are the aggregator snapshots: CoinMarketCap's profile (2M+ users, 700+ pairs), CoinGecko's live page (503 pairs; ~US$293.7M 24h volume; ~US$256.7M reserves — September 2026 snapshot ⚠ time-varying), and the CoinGlass 2025 Annual Report rankings (§6.2). Everything else is the company's own growth narrative, marked ⚠ throughout.

### 6.2 The Rankings — CoinGlass, CoinMarketCap, CoinGecko, DefiLlama

- **CoinGlass 2025 Annual Report (the strongest third-party datapoint).** Per the company's June 2026 blog citing the CoinGlass 2025 Annual Report (coinglass.com/learn/2025-annual-report-en), Bitunix ranked **No. 7 by trading volume and No. 10 by open interest among global crypto derivatives exchanges in 2025** (bitunix.com/hub/blog/bitunix-features/bitunix-ranked-top-10-open-interest-top-7-volume-coinglass-2025). ✅/⚠ — the report exists and is linked at a primary aggregator URL; the ranking's *meaning* (volume-share estimates, methodology) is the aggregator's and is treated as a press/aggregator estimate. ⚠ where market-share claims derive from it.
- **The May 2026 multi-aggregator table (company blog, ⚠ company-curated):** CoinMarketCap #37 spot / #30 futures; CoinGecko #13 spot / #21 futures; CryptoRank #26 spot / #10 futures; CoinGlass #10 futures; CoinRanking #27; Messari #35 (H1 2026 Growth Review). These are the company's own compilation of aggregator positions at a point in time; the positions are real aggregator rankings but the table is company-curated ⚠.
- **DefiLlama.** The H1 2026 blog claims Bitunix was a **Top-3 exchange globally for single-month CEX inflow** on DefiLlama ⚠ (company claim; DefiLlama's CEX-transparency metrics are aggregator data but the specific claim was not re-verified this pass).
- **About-page badges.** "Top 15 CoinGecko," "Top 10 CoinGlass," "ranks among the top 15 exchanges globally on CoinMarketCap and CoinGecko" (About page) ⚠ (self-reported badges, consistent with the tables above but presented as static claims).
- **The trajectory narrative.** The CoinGlass blog's growth story: 2023 — "began drawing broader attention on CoinMarketCap and CoinGecko"; 2024 — Similarweb's fastest-growing list (No. 3, §6.1); 2025 — CoinGlass No. 7 volume / No. 10 open interest; H1 2026 — US$300B+ volume (⚠ company-sourced throughout).

### 6.3 The Competitive Landscape — Binance, Bybit, OKX, and the Mid-Tier

The verified public facts about the competitive field (each flagged where the figure is an aggregator estimate):

- **The tier structure.** Binance is the industry leader in both spot and derivatives (the CoinGlass 2025 chart, reproduced in Bitunix's blog, shows Binance leading, followed by OKX and Bybit ⚠ as an aggregator estimate). Bitunix's own positioning — No. 7 derivatives volume in 2025 — puts it in the **second tier of derivatives venues**, alongside the Bitget/MEXC/Gate/ KuCoin cluster, behind Binance, OKX, Bybit, and (in some periods) Hyperliquid and Bitget. ⚠ (tier placement is an aggregator-based inference).
- **Binance.** The dominant CEX; also the subject of the November 2023 US resolutions (§9.1) — the enforcement event that reset the industry's compliance expectations. ✅
- **Bybit.** A top-tier derivatives peer; announced its exit from the French market effective August 13, 2024, citing the French regulator and the MiCA transition (§9.3) ✅ — a concrete example of the market-exit pattern that mid-tier exchanges (including Bitunix) manage around.
- **OKX.** A top-tier derivatives exchange; commonly cited as one of the top-three derivatives venues by volume ⚠ (aggregator-based). No Bitunix-specific head-to-head market-share figure is publicly available; the company's own materials avoid naming peers' market shares ⚠.
- **What differentiates Bitunix in the field (company claims, ⚠):** derivatives-first perpetuals with up to 200x leverage; deep-liquidity/slippage claims; TradingView Pro; aggressive listing and campaign economics (Token Splash, Task Center, anniversary giveaways); 24/7 multilingual support (14–18 languages); and a transparency stack (PoR, Care Fund, Hacken audits) aimed directly at the trust deficit the FTX-era left behind.
- **The market-share caveat.** Any statement of Bitunix's share of global derivatives volume is an aggregator-derived estimate (CoinGlass, CryptoRank, DefiLlama) and is marked ⚠; the exchange publishes no audited share figure.

### 6.4 The Competitive-Comparison Table

The table below compares Bitunix with its three most-cited peers on the dimensions this guide actually verified. **Caveat:** the Binance/Bybit/OKX cells are general industry knowledge (widely documented, but not re-verified against primary sources in this pass) and are marked ⚠-knowledge accordingly; the Bitunix column carries this guide's own verification marks.

| Dimension | Bitunix | Binance | Bybit | OKX |
| --- | --- | --- | --- | --- |
| Founded | 2021, SVG-registered ✅ | 2017 ⚠-knowledge | 2018 ⚠-knowledge | 2017 ⚠-knowledge |
| Derivatives focus | Perpetuals-first; up to 200x; no options (Oct 2025) ✅/⚠ | Full suite (spot, perps, options) ⚠-knowledge | Full suite, derivatives-led ⚠-knowledge | Full suite, derivatives-led ⚠-knowledge |
| 2025 derivatives volume rank | CoinGlass No. 7 volume / No. 10 open interest ✅/⚠ | CoinGlass No. 1 (per Bitunix-cited chart) ⚠ | Top-3 per the same chart ⚠ | Top-3 per the same chart ⚠ |
| US market | Prohibited (press-reported) ⚠ | Restricted after the 2023 resolution ✅-knowledge | Restricted ⚠-knowledge | Restricted ⚠-knowledge |
| Licensing posture | SVG incorporation + MSB-type registrations; no EU/SG licence ⚠/❌ | Post-2023 global compliance overhaul under monitorship ✅ | France exit 2024; Dubai VARA era ⚠-knowledge | Market pruning; no verified Bitunix-comparable list ⚠ |
| Proof of Reserves | Merkle-tree PoR with published ratios + GitHub tool ✅/⚠ | PoR program exists ⚠-knowledge | PoR program exists ⚠-knowledge | PoR program exists ⚠-knowledge |
| Protection fund | Care Fund US$30M claimed (2026) ⚠ | Industry fund (SAFU-type) ⚠-knowledge | Protection fund ⚠-knowledge | Protection fund ⚠-knowledge |
| Enforcement events | None found this pass ⚠ (absence) | Nov 2023 DOJ/CFTC resolutions >US$4B ✅ | France regulatory exit Aug 2024 ✅/⚠ | None verified this pass ⚠ |

Reading the table vertically shows the strategic position: Bitunix competes on **derivatives accessibility (high leverage, low fees, copy trading) for the permissive-market retail base**, exactly where the giants cannot fully follow without the licensing overhead — and where the giants' enforcement history is the cautionary tale. ⚠ (analysis).

---

## 7. Licensing and Compliance

### 7.1 The Verified Registrations — FinCEN MSB, FINTRAC, AUSTRAC, and the 2026 Footprint

The licensing picture is a patchwork, and each patch carries its own evidence quality:

- **FinCEN MSB (US).** The NewsBTC review (October 2025) states Bitunix "holds licenses as a Money Services Business (MSB) with FinCEN (US)" (newsbtc.com/bitunix-review/). ⚠ (press-reported; FinCEN's MSB registrant list is not freely searchable by brand name, so the registration could not be confirmed at the registry — see §12). Note the apparent paradox: the same review states residents of the US and its territories are prohibited from using the platform — an MSB registration without US market access is a known posture (a federal registration is not a state-licence suite, and most US states require money-transmitter licences the exchange does not claim) ⚠.
- **FINTRAC (Canada).** The same review states Bitunix holds an MSB registration with FINTRAC ⚠ (press-reported, registry not queried this pass). Independently, the company's H1 2026 blog lists "**Canada: Money Services Business (MSB) Registration**" as a 2026 milestone ✅ (as published by the company) — which either dates the registration to H1 2026 or reflects a renewal/expansion; the two sources are not reconciled ⚠.
- **The H1 2026 regulatory milestones (company blog, ✅ as published, ⚠ registry-unverified):** "**Australia: AUSTRAC Registration; South Africa: Financial Services Provider (FSP) License; Mauritius: Investment Dealer License**" (bitunix-h1-2026-growth-review, July 2026). These are the company's own claims about its expanded compliance footprint "across multiple continents"; none was verified against the national registers this pass ⚠.
- **The compliance framing.** The H1 2026 blog's bullet is "Fortified Compliance & Trust — Expanded the global regulatory footprint with new licenses and registrations across multiple continents alongside an **ISO/IEC 27001:2022 certification**" ⚠ (certification claim; certificate not independently verified).

### 7.2 The Unverified Licensing Claims — Lithuania, the Seychelles, and the Philippines

- **Lithuania (believed anchor).** The research brief believed a Lithuanian VASP registration existed. **No primary or reputable secondary source found in this pass mentions a Bitunix entity registered with Lithuania's Financial Crime Investigation Service (FCIS) as a VASP.** Targeted searches ("Bitunix Lithuania", "Bitunix VASP", "Bitunix UAB") returned empty result sets. Recorded as ⚠/❌ (not found; absence of evidence flagged, see §12). ⚠
- **The Philippines.** The NewsBTC review states Bitunix holds a "**Virtual Asset Service Provider (VASP) in the Philippines**" registration ⚠ (press-reported, single-sourced; the Philippines' SEC/Central Bank registers were not queried this pass).
- **The Seychelles.** The operational base is reported in the Seychelles (§2.3) but **no Seychelles FSA licence or registration for a Bitunix entity could be verified** ⚠. The Seychelles does license some crypto businesses (e.g., under its Virtual Asset Service Provider regime), but no such record was found for Bitunix this pass ⚠.

### 7.3 The KYC/AML Posture (Cross-Referenced, Condensed)

Bitunix's AML/KYC posture is assembled from its own compliance blog and support materials; the screening *themes* are cross-referenced to the [Fircosoft](fircosoft_guide.md) guide (§3–§5 there: sanctions-list screening, name-matching, fuzzy logic, false-positive management) rather than re-derived:

- **Mandatory, tiered KYC.** "Mandatory KYC to ensure safety and compliance" (company boilerplate ✅ as published). Two tiers per the company's compliance blog: **KYC 1 (Basic)** — government-issued ID + biometric selfie, automated processing in minutes, crypto withdrawals up to **500,000 USDT/24h**; **KYC 2 (Advanced)** — adds proof of address (utility bill/bank statement), manual review in 1–3 hours, withdrawals up to **5,000,000 USDT/24h** (bitunix.com compliance blog, 2026) ✅ (as published). The NewsBTC review adds the historical nuance that account withdrawal caps also depend on security-verification strength (e.g., Google-Authenticator-only accounts capped at ~US$10K equivalent until KYC is completed) ⚠ (press-reported, October 2025).
- **Identity-verification vendor.** "Automated identity verification powered by **Sumsub**" (company compliance blog) ✅ (as published — a named, reputable KYC vendor; the *configuration* is not public ⚠).
- **Institutional verification.** Business accounts require institutional certification — company name, website, registered address, purpose of account opening, contact person (NewsBTC ⚠; consistent with the exchange's institutional/VIP push).
- **Monitoring.** The company states it monitors "abnormal activity patterns," applies "risk-based checks where required," and manages "automated suspicious activity reports (SARs) for relevant cases" (company compliance blog ✅ as published; NewsBTC corroborates the SAR statement ⚠). The company explicitly does **not** publish granular detection rules or internal thresholds ⚠.
- **The condensed takeaway** (cross-ref [Fircosoft](fircosoft_guide.md) §9's "clean list" framing): Bitunix's control stack is a standard layered design — identity/KYC at onboarding (Sumsub), risk-based monitoring on behaviour, SAR filing, restricted jurisdictions (US, Hong Kong, mainland China, Singapore, sanctioned countries — NewsBTC ⚠), and account-security tooling (2FA, anti-phishing codes, withdrawal-address whitelisting, phone binding — NewsBTC/company ✅/⚠). Nothing in the public record suggests systemic AML weaknesses; equally, nothing beyond these vendor names, tier tables, and policy statements is public — no independent AML audit, no regulator confirmation, no published screening configuration ⚠ (depth-of-evidence limitation).

### 7.4 The Singapore Angle — MAS and the DPT Regime (Cross-Referenced)

The Singapore question, treated per the repository's convention (cross-ref [MAS Regulations](mas_regulations_guidelines_guide.md) — the PS Act, the SPI/MPI tiers, and the DPT measures — rather than re-deriving the regime):

- **The regime in one line (cross-referenced).** Under Singapore's Payment Services Act 2019, providing digital payment token (DPT) services — including operating a DPT exchange or facilitating DPT transfers — requires a **Major Payment Institution (MPI) licence** for DPT services (or, below thresholds, a Standard Payment Institution licence). No crypto exchange serves the Singapore retail market lawfully without such a licence (MAS guide §2.4, §5.5). ✅ (cross-referenced).
- **Bitunix's status: restricted, not licensed.** The NewsBTC review lists **Singapore among the jurisdictions bound by regulatory restrictions** — i.e., Singapore residents are not served by Bitunix (newsbtc.com/bitunix-review/ ⚠ press-reported). Consistent with that, **no evidence was found that any Bitunix entity holds an MAS licence or has notified under the PS Act** ⚠ (negative finding: the MAS Financial Institutions Directory could not be queried interactively this pass — see §12 — so this is an absence-of-evidence flag, not a registry proof).
- **The honest status line.** Bitunix appears to be **not present in the Singapore market by design**: Singapore users are restricted, no MAS licence is claimed anywhere in the company's materials, and the company's compliance narrative emphasises registrations elsewhere (US MSB, Canada, Australia, South Africa, Mauritius — §7.1). The believed anchor's "Singapore angle" therefore resolves to: *the exchange's SG status is "not licensed, not serving residents, not claiming otherwise"* — with the registry-level confirmation flagged ⚠ rather than asserted. For a Singapore bank like Cymbal Bank, this posture is the cleanest possible compliance story from the bank's side (the client is not SG-facing), but it also means the bank cannot rely on MAS supervision of the exchange as a mitigating control — the §10 worked example reasons from exactly this.

### 7.5 The Licensing Table

The table below is the regulatory core of the guide, condensed from §7.1–§7.4 — the reference the Cymbal Bank worked example (§10.2) reasons from:

| Jurisdiction | Instrument | Status | Evidence |
| --- | --- | --- | --- |
| US | FinCEN MSB registration (claimed) | ⚠ press-reported; registry not queried | NewsBTC (Oct 2025) |
| Canada | FINTRAC MSB registration (claimed) | ⚠ press-reported + company blog (H1 2026) | NewsBTC; H1 2026 blog |
| Philippines | VASP registration (claimed) | ⚠ press-reported, single-sourced | NewsBTC |
| Australia | AUSTRAC registration (claimed, H1 2026) | ✅ as published / ⚠ registry-unverified | H1 2026 blog |
| South Africa | FSP licence (claimed, H1 2026) | ✅ as published / ⚠ registry-unverified | H1 2026 blog |
| Mauritius | Investment Dealer licence (claimed, H1 2026) | ✅ as published / ⚠ registry-unverified | H1 2026 blog |
| Lithuania | VASP registration (believed anchor) | ❌ not found in any source | This pass's searches |
| Seychelles | FSA licence / registration | ⚠ not found; operations base single-sourced | NewsBTC; no FSA record |
| Singapore | MAS MPI licence (DPT) | ⚠ not held; residents restricted (press) | NewsBTC; absence-of-evidence |
| EU | MiCA CASP authorisation | ⚠ not claimed; no EU licence asserted | Company materials |

Reading the table vertically tells the strategy: **registrations cluster in federal-filing and light-touch regimes; no comprehensive market-access licence (EU, Singapore) is claimed; the believed Lithuanian anchor fails; and every cell below the company's own blog claims carries a ⚠ or ❌.** This is the licensing reality a bank documents before it touches the settlement accounts (§10).

---

## 8. Technology — Platform, Security, and Proof of Reserves

### 8.1 The Platform Facts That Are Public

The public technology record is thin but consistent (⚠ where the company does not disclose):

- **Trading stack.** TradingView Pro integration, the proprietary Ultra K-Line chart system, multi-chart windows, millisecond-level matching claims, and sub-0.01% slippage claims on mainstream assets (§4.3 — all ⚠ company claims except the TradingView integration, which the NewsBTC hands-on review corroborated ✅/⚠).
- **Client surfaces.** Web platform plus iOS/Android apps (NewsBTC-tested ✅/⚠); 24/7 support in 14–18 languages (About page/H1 2026 blog ⚠).
- **The 2026 tooling layer.** Super Alert, Fixed Risk, Multi-Trade — proprietary H1 2026 features (company blog ⚠).
- **API documentation.** Bitunix maintains a developer/API surface (referenced in its support and integration materials), but the API documentation itself was **not extracted or verified this pass** ⚠ — the API's endpoints, rate limits, and authentication model are recorded as not-disclosed here rather than described from memory.
- **Not disclosed (⚠):** the matching-engine architecture and language, order-routing design, the ledger/database stack, cloud infrastructure and hosting regions, data-residency arrangements, the risk/ML models behind monitoring, and the custody key-management topology beyond the vendor names. Where the repository's [Cybersecurity](../technology/cybersecurity_guide.md) guide supplies the vocabulary (hot/cold separation, multi-signature, ISO/IEC 27001, bug bounties), this guide uses it; where Bitunix is silent, this guide says so.

### 8.2 The Security Posture — Custody, Cold Storage, and the Care Fund

- **Institutional custody partners.** "The wallet collaborates with industry-leading custodian institutions, **COBO and Fireblocks**" (About page ✅ as published; the H1 2026 blog repeats both names under "Institutional Custody"). Both Cobo and Fireblocks are real, well-known digital-asset custody/infrastructure firms ✅ (vendor existence verified); the *commercial terms* and *which assets sit with which custodian* are not disclosed ⚠.
- **Hot/cold separation and multi-signature.** "Adopt the separation of hot and cold wallets... Multi-signature Technology" (About page ✅ as published); the NewsBTC review adds ">95% of user assets [in] cold wallets" and "MPC cold storage" ⚠ (press-reported figures).
- **The Bitunix Care Fund.** The company's 2026 materials describe a "**$30 Million USDC Care Fund**" — a dedicated insurance reserve "standing ready as an internal capital backstop" (company compliance blog; H1 2026 blog ✅ as published). ⚠ Notes: (1) the fund's size is company-reported and the wallet/address is not published ⚠; (2) earlier reporting described a **US$5 million insurance fund** — a 2024–2025-era partnership with **Nemean** ("Bitunix Adds Proof of Reserves, $5M Insurance Fund & Security Upgrades... partners with Nemean for $5M insurance," cexfinder.com, snippet-verified ⚠) — so the protection-fund figure has evolved from US$5M (Nemean-era) to US$30M USDC (2026-era); the relationship between the two arrangements is not public ⚠.
- **Audits and certifications (all ⚠ company/vendor-claimed, none independently re-verified):** **Hacken** security audits (the NewsBTC review links a Hacken audit page, hacken.io/audits/bitunix/pt-bitunix-web-api-apr2025/ ✅ link exists); a **Cer.Live AA rating** (company claim ⚠); **ISO/IEC 27001:2022** certification (company claim ⚠ — no certificate or registrar name published).
- **No known breach.** The NewsBTC review states Bitunix "hasn't been hacked" to date (October 2025) ⚠ (press-reported negative; absence of a public breach record is corroborated by the absence of any breach coverage in this pass's searches ✅/⚠).
- **Account-security tooling.** 2FA, anti-phishing codes, withdrawal-address whitelisting, phone binding (NewsBTC/company ✅/⚠).
- **The condensed security reading** (cross-ref [Cybersecurity](../technology/cybersecurity_guide.md) §6 — the frameworks map — and §4.4 — bug bounties): Bitunix's posture is the standard exchange template — institutional custodians, cold-storage majority, multi-sig, a protection fund, third-party audits, ISO certification claims, and user-side account controls. The gaps a bank would probe are the standard ones: no published bug-bounty program found this pass ⚠, no independent certification verification ⚠, no published key-management topology ⚠, and a protection fund whose size changed from US$5M to US$30M across 18 months without public explanation ⚠.

### 8.3 Proof of Reserves and the On-Chain Mechanics (Cross-Referenced)

The on-chain mechanics are cross-referenced to the repository's [Smart Contracts](../technology/smart_contracts_guide.md) guide (§5 — the ERC-20 token standards that USDT/USDC run on; §8 — audit practice and the vulnerability classes; §9 — the incident history that motivates PoR) rather than re-derived:

- **The Merkle-tree PoR.** Bitunix publishes a **Merkle-tree Proof of Reserves** claiming 1:1 backing of user deposits, with the claims that "Bitunix does not re-hypothecate or leverage customer funds" and that users can verify their balances are included in the tree without exposing private information (company compliance blog ✅ as published). The mechanics — a Merkle root committing to a tree of user-balance hashes, published alongside on-chain wallet balances — are the standard PoR construction described in the smart-contracts guide's audit/transparency themes (§8 there); this guide does not re-derive the cryptography. ✅/⚠ (the *construction* is standard; the *attestation's independence* — who audits the tree, how often, against which wallets — is not fully public ⚠).
- **The numbers.** March 2026 audit timestamp: **US$128.5M+ total, with 109% BTC, 100% ETH, 109% USDT reserve ratios** (company PoR update, March 3, 2026, distributed via press release — bitfeed.news/advertorial copies ⚠ company-sourced); July 2026 update: **BTC 110%, ETH 104%, USDT 104%, total audited reserve value US$192,828,146** (H1 2026 blog, citing CoinMarketCap as the data source ⚠). The reserve ratios are the company's own published figures; the on-chain wallet balances behind them were not independently re-aggregated this pass ⚠.
- **The verification tooling.** A native PoR verification tool is published on the company's GitHub (**github.com/BitunixOfficial/proof-of-reserves/** — link verified via the NewsBTC review ✅), alongside the proof-of-reserve page on bitunix.com (bitunix.com/proof-of-reserve ✅). The GitHub presence is itself a transparency datapoint ✅ (existence verified; code quality/recency not audited this pass ⚠).
- **Why PoR matters in context.** The post-FTX industry norm is that an exchange's *claims* of solvency are worthless without verifiable on-chain backing — the exact lesson of the smart-contracts guide's incident history (§9 there: FTX's collapse was a books-vs-wallets failure). Bitunix's PoR stack is its answer to that trust deficit: Merkle tree + published ratios + GitHub tooling + custodians. The honest caveat stands: PoR proves *reserves exist at a snapshot*, not *liabilities are correctly computed* or *the business is solvent* — a distinction the §10 worked example carries into the bank's due-diligence process. ⚠

### 8.4 What Is Not Disclosed

The following technology facts are **not public** and are flagged rather than guessed: the matching-engine and ledger architecture; the cloud/hosting stack and data-residency; the custody key-management topology (beyond "Cobo and Fireblocks"); the identities and terms of the third-party fiat payment providers behind the 81-currency deposit rails; the P2P marketplace's operational model and screening; the copy-trading profit-share economics; the risk-monitoring ML models and thresholds; the API documentation (not extracted this pass); the bug-bounty program (none found ⚠); the Care Fund's wallet address and governance; and the identities of the auditors behind the PoR attestations and the ISO/IEC 27001 certificate. Each of these is recorded in §12 ("What Could Not Be Verified") as well.

### 8.5 The Verified Technology Facts Table

| Fact | Detail | Status |
| --- | --- | --- |
| Public charting | TradingView Pro integration; Ultra K-Line; multi-chart windows (16) | ✅/⚠ (§4.3) |
| Matching engine | "Millisecond-level matching"; slippage <0.01% on mainstream assets | ⚠ company claims (§4.3) |
| Client surfaces | Web platform + iOS/Android apps | ✅/⚠ (NewsBTC-tested) |
| Custody | Cobo and Fireblocks; hot/cold wallet separation; multi-signature | ✅ as published (vendor names) / ⚠ topology |
| Cold-storage share | >95% of user assets in cold wallets; MPC | ⚠ press-reported (§8.2) |
| Protection fund | US$30M USDC Care Fund (2026); US$5M Nemean insurance fund (2024–25) | ⚠ company-reported (§8.2) |
| Proof of Reserves | Merkle tree; 1:1 claim; Mar 2026 ratios 109/100/109; Jul 2026 ratios 110/104/104; GitHub verification tool | ✅/⚠ (§8.3) |
| Audits and certifications | Hacken audits; Cer.Live AA rating; ISO/IEC 27001:2022 | ⚠ claimed, not independently verified (§8.2) |
| KYC/identity | Sumsub-powered; KYC 1/KYC 2 tiers; withdrawal caps 500K/5M USDT per 24h | ✅ as published (§7.3) |
| Monitoring | Abnormal-activity monitoring; automated SARs | ✅ as published; rules not disclosed ⚠ (§7.3) |
| API surface | Developer/API documentation exists; not verified this pass | ⚠ (§8.1) |
| Breach history | No public breach record found | ⚠ absence of evidence (§8.2) |

The table is the bank's technology checklist in miniature: the *names* (custodians, vendors, tools) are checkable; the *depth* (topology, thresholds, attestation independence) is not — which is precisely why the §10 worked example treats the technology record as an input to due diligence rather than a substitute for it.

---

## 9. Regulatory Headwinds — The Industry Enforcement Record

The exchange industry's regulatory climate is the backdrop against which every Bitunix claim in §6–§8 must be read. The events below are verified at primary or reputable sources and are presented as industry context — none of them involves Bitunix, and no enforcement action against Bitunix was found in this pass (recorded as an absence-of-evidence flag ⚠ in §12).

### 9.1 Binance — The November 2023 Resolutions

On **November 21, 2023**, Binance Holdings Limited and its CEO Changpeng Zhao pleaded guilty to US federal charges in a resolution totalling **over US$4 billion**:

- The US Department of Justice's own press release: "Binance and CEO Plead Guilty to Federal Charges in $4B Resolution" — Binance pleaded guilty to violations related to the **Bank Secrecy Act (BSA)**, **failure to register as a money transmitting business**, and the **International Emergency Economic Powers Act (IEEPA)** (justice.gov/archives/opa/pr/binance-and-ceo-plead-guilty-federal-charges-4b-resolution, November 21, 2023). ✅
- Zhao pleaded guilty to a BSA violation — "causing Binance to fail to implement an effective anti-money laundering program" — and stepped down as CEO (justice.gov/criminal/case/united-states-v-changpeng-zhao; Reuters, "Binance's Zhao pleads guilty, steps down to settle US illicit finance probe," November 21, 2023 — the settlement figure reported as US$4.3 billion). ✅
- The DOJ resolution also included monitorship terms (three- and five-year monitors, per Fortune's contemporaneous analysis ⚠/✅ as reported). ✅/⚠

Why this matters for Bitunix: Binance's resolution is the template enforcement event for the whole CEX industry — it established that **operating a global exchange without effective AML controls and without US registration is a federal crime even when the exchange has no US headquarters**, and it pushed every major exchange to harden KYC, add PoR, and prune restricted jurisdictions (including the US). Bitunix's own posture — US users prohibited, MSB registration claimed, mandatory KYC, PoR — is legible as a direct response to this precedent. ✅/⚠ (the causal link is this guide's analysis, flagged).

### 9.2 FTX — The November 2022 Collapse

On **November 11, 2022**, FTX Trading Ltd filed for Chapter 11 bankruptcy protection in the US, with founder and CEO Sam Bankman-Fried resigning the same day (Reuters, "Crypto exchange FTX files for bankruptcy as wunderkind CEO exits," November 11, 2022). ✅ Contemporaneous reporting put the shortfall in FTX's accounts at roughly **US$8 billion** (CBS News, November 2022 ✅/⚠ as reported at the time). FTX had been the third-largest exchange by volume with over one million users before the collapse (Wikipedia's bankruptcy-of-FTX article, ⚠ encyclopedic source). Bankman-Fried was subsequently convicted on fraud charges and sentenced in 2024 — a matter of public record ⚠/✅ (widely reported; the sentence's exact terms are not re-litigated here).

Why this matters for Bitunix: FTX is the industry's defining **counterparty-trust failure** — the exchange that collapsed because customer funds were not where the books said they were. The post-FTX era is precisely why Bitunix publishes Proof of Reserves, a Care Fund, custody partners, and Hacken audits (§8.2–§8.3); it is also why a bank onboarding any exchange client treats **books-vs-wallets verification** as a first-order control (§10). ✅ (the causal reading is this guide's analysis; the events are verified).

### 9.3 Bybit, OKX, and the Market-Exit Pattern

- **Bybit and France.** Bybit announced on **August 1, 2024** that it was ending operations in France, effective **August 13, 2024**, attributing the exit to regulatory developments from the French regulator (Autorité des marchés financiers / AMF) and the MiCA transition (Cryptoslate, "Bybit to exit France amid regulatory hurdles"; CCN, "Bybit Bids Adieu to France"). ✅/⚠ (the event is verified across multiple outlets; the precise regulatory trigger is press-characterised).
- **The pattern.** Bybit's France exit is one instance of a recurring industry pattern in 2024–2026: exchanges **withdrawing from jurisdictions rather than obtaining licences**, particularly in the EU (pre/post-MiCA), India, and other markets with tightening regimes. OKX and others made similar market-pruning moves in the same period ⚠ (widely reported; no single primary source was pulled for OKX in this pass, so it is flagged rather than detailed).
- **Why it matters for Bitunix.** Bitunix's own restricted-jurisdiction list (US, Hong Kong, mainland China, Singapore, sanctioned countries — §7.3) is the same playbook: **serve the permissive markets, restrict the regulated ones, and collect registrations where registration is light-touch** (MSB-type filings). The strategy is common; it is also exactly what a bank's financial-crime team must test, because "we don't serve that jurisdiction" is only as strong as the controls that enforce it. ⚠ (analysis).

### 9.4 MiCA and the EU Regime

The **Markets in Crypto-Assets Regulation (MiCA)** — Regulation (EU) 2023/1114 — is the EU's unified crypto rulebook:

- **Timeline (verified):** MiCA was adopted on May 31, 2023, published in the Official Journal on June 9, 2023, and **entered into force on June 29, 2023**; the stablecoin provisions (Titles III and IV — asset-referenced tokens and e-money tokens) became applicable on **June 30, 2024**; the crypto-asset-service-provider (CASP) provisions (Title V) became fully applicable on **December 30, 2024** (eur-lex.europa.eu/eli/reg/2023/1114/oj; esma.europa.eu MiCA page; scorechain.com summary). ✅
- **The single-passport design.** Once a CASP is authorised in one member state, it can serve the whole EU (Binance Academy's MiCA explainer ✅ as published). The consequence for non-EU exchanges is binary: get authorised somewhere in the EU, or exit the EU market — the Bybit-France story in miniature. ✅/⚠
- **Why it matters for Bitunix.** Bitunix's H1 2026 licensing narrative (§7.1) names registrations in Canada, Australia, South Africa, and Mauritius — **no EU CASP licence is claimed**, and no EU market access is claimed. Consistent with the Bybit pattern, the EU appears to be a market Bitunix serves only at the margins or not at all. The MiCA regime also matters as the global template: the EU's stablecoin and CASP rules are the direction of travel for the licensing patchwork Bitunix currently navigates. ✅/⚠ (analysis).

### 9.5 What the Record Means for Bitunix

The verified enforcement record of 2022–2026 sets the industry's risk baseline: **Binance proved that AML failures are criminal; FTX proved that unverifiable books are fatal; Bybit's France exit proved that market access can be revoked by regulation; MiCA proved that the patchwork is consolidating into licensing regimes.** Bitunix's public posture — restricted jurisdictions, mandatory KYC, PoR, a protection fund, custody partners, and multi-continent registrations — is coherent with that baseline. What the public record does **not** show is any regulator's independent confirmation of Bitunix's controls, any audited financial statement, or any enforcement action involving Bitunix (⚠ absence-of-evidence flags, §12). For a bank, that is the honest state of the evidence: the exchange presents the *appearance* of a compliance-hardened, transparency-forward venue, and the verification burden sits with the bank's own due diligence (§10).

---

## 10. The Cymbal Bank Worked Example — A Crypto-Exchange Corporate Client

*Fictional-but-substantiated: the client, the numbers, and the flows below are constructed for illustration from the verified facts in §1–§9 and the repository's persona conventions (see [Reap Global](reap_global_guide.md) §11, [Citadel LLC](citadel_llc_guide.md), and [Resona Merchant Bank Asia](resona_merchant_bank_asia_guide.md)). Nothing in this section describes an actual Bitunix client relationship.*

### 10.1 The Scenario

**Halcyon Exchange Pte. Ltd.** is a Singapore-incorporated holding entity for a mid-tier crypto derivatives exchange of the Bitunix type (per the verified profile of §1–§9): registered in an offshore jurisdiction, operational staff in a second jurisdiction, ~5 million registered users, derivatives-led volume, a Merkle-tree Proof of Reserves, a US$30M protection fund, KYC in two tiers via a named vendor, and a restricted-jurisdiction list that includes the US and Singapore. Halcyon approaches Cymbal Bank for corporate banking: fiat settlement accounts for its P2P and third-party deposit rails, US-dollar and Singapore-dollar treasury accounts, and an FX/remittance line for its operational expenses. Halcyon's pitch is the standard one: it is growing fast (company-reported), it is "regulated" (a list of MSB-type registrations), and it is transparent (PoR links). Cymbal Bank's answer is the standard bank's: *we will onboard you, on terms — and the terms are set by what we can verify, not by what you claim.*

### 10.2 Onboarding and the KYC/AML Overlay

The onboarding follows the repository's screening themes ([Fircosoft](fircosoft_guide.md) §3–§5, condensed) and the MAS expectations of the [MAS guide](mas_regulations_guidelines_guide.md) (§7 — the obligations × controls map):

- **Corporate KYC and the UBO problem.** Halcyon's ultimate beneficial owners are not publicly identifiable (the founders of the underlying exchange are not publicly named — §2.2 — a direct parallel to the real Bitunix record). Cymbal Bank therefore requires, at minimum: (1) the offshore registry extract for the operating entity, (2) a certified UBO declaration naming every individual with >10% ownership (Cymbal applies a stricter threshold than the regulatory 25% floor, consistent with the repo's conventions), (3) independent confirmation from a reputable corporate-services provider, and (4) a management attestation signed by two authorised signatories. The public anonymity is not, by itself, a declination reason — but it converts what would be a low-touch onboarding into an enhanced-due-diligence (EDD) file, reviewed by the Financial Crime Committee rather than the relationship team alone. ✅/⚠ (design fiction, built on the verified anonymity fact).
- **Screening.** The entity, its signatories, its directors, and its wallet addresses' counterparties are screened against sanctions lists (OFAC-style, the Fircosoft themes: exact and fuzzy name-matching, list currency, false-positive management) at onboarding and continuously thereafter. The restricted-jurisdiction attestation is contractual: Halcyon warrants it does not serve US, Hong Kong, mainland China, Singapore, or sanctioned-jurisdiction residents — and the bank tests the warranty through sampling of the client's published product documentation and through its own transaction monitoring (§10.4).
- **The licensing overlay.** The bank's compliance team documents the *actual* licensing state from §7: MSB-type registrations in several jurisdictions (⚠ registry-unverified), no MAS licence, Singapore explicitly restricted. The bank does not accept "registered" as "regulated": each registration is categorised (federal MSB filing vs state money-transmitter licences vs full VASP licence), and the categories drive the risk rating. The Singapore angle is resolved per the MAS guide's scope analysis: **Cymbal's services to Halcyon are MAS-scoped banking services; Halcyon's services are not SG-directed; the gap is documented, not assumed away** (the same structure the Reap worked example uses at its Singapore leg — reap_global_guide.md §11.5).

### 10.3 The Banking Services and Settlement Flows

The banking product set follows the verified flows of §5.4 and the rails of the [Payment Rails](payment_rails_guide.md) guide (§3 — the SG FAST/PayNow rails; §5 — clearing and settlement):

1. **Fiat settlement accounts.** Halcyon holds US-dollar and Singapore-dollar accounts at Cymbal Bank for its fiat on/off-ramps (the P2P and third-party deposit rails of §5.4 — 76 and 81 fiat currencies respectively in the real Bitunix profile ⚠). Settlement into these accounts is the point where the exchange's crypto flows touch the banking system — the highest-risk leg of the relationship.
2. **P2P settlement mechanics.** For P2P trades, the pattern is: a buyer credits the seller's bank account (via FAST/PayNow for SG rails), and the exchange's internal ledger releases the crypto; the bank sees a high-volume, low-value inbound/outbound retail pattern on Halcyon's accounts. Cymbal's overlay is a **velocity and pattern baseline**: expected daily credit counts, value bands, and counterparty concentration, with alerts on deviation (the monitoring themes of §10.4).
3. **Treasury and FX.** Halcyon converts fee income and operational cash across USD/SGD via Cymbal's FX desk; the bank offers the exchange a same-day settlement window and, separately, a working-capital facility — **declined at this stage** because the bank cannot yet independently verify the exchange's solvency beyond its PoR snapshots (§8.3). The PoR is treated as *evidence in the file*, not as *collateral*: a Merkle tree proves reserves at a timestamp, not liability computation or going-concern solvency — the exact caveat §8.3 states.
4. **The settlement calendar.** Daily sweeps, T+0 for FAST/PayNow legs, and a monthly reconciliation pack in which Halcyon must demonstrate that its internal ledger's fiat balances match Cymbal's account balances — the bank's classic reconciliation duty applied to an exchange client.

### 10.4 The Risk and Monitoring Overlay

- **Transaction monitoring.** Cymbal's AML engine monitors Halcyon's accounts for the patterns that matter for an exchange: rapid pass-through (funds arriving and leaving without economic purpose), structuring (sub-threshold cash-like movements), concentration (a small set of counterparties dominating flows), and destination risk (payments to high-risk jurisdictions). Alerts feed the case-management workflow — the false-positive economics of the [Fircosoft](fircosoft_guide.md) guide applied to a single high-volume client.
- **The on-chain overlay.** For the crypto legs, Cymbal's due diligence reads Halcyon's published PoR and wallet addresses (github.com/BitunixOfficial/proof-of-reserves/ in the real profile — §8.3) and monitors for material drawdowns or unexplained wallet movements; a sustained divergence between published reserve ratios and observed balances would trigger a relationship review. This is the bank reading the client's on-chain transparency machinery as control evidence — the §8.3 point, operationalised.
- **Periodic review cadence.** EDD refresh every six months (not annually), because the client's risk profile moves with the market and with the licensing patchwork (§7). Each refresh re-runs the sanctions screen, re-checks the restricted-jurisdiction warranty against the client's current published materials, and re-scoring the client on the bank's VASP matrix.
- **Exit conditions.** Pre-agreed triggers: loss of a named registration; a material PoR-ratio breach; an enforcement action against the exchange (the §9 record makes this a live possibility class, not a theoretical one); a sanctioned-jurisdiction breach found in monitoring; or the appointment of a receiver/insolvency professional. The exit plan includes wind-down mechanics for the settlement accounts and a communications protocol — because in the post-FTX world, the bank that froze an exchange's accounts *before* the collapse is the bank that protected its depositors.

### 10.5 The Lessons

The worked example closes with the four lessons the real Bitunix record teaches a bank:

1. **Verify, don't import.** The exchange's own claims (5M users, US$300B volume, "regulated", "1:1 backed") are marketing-grade until proven; the bank's file must distinguish ✅ (independently checkable: founding year, aggregator rankings, vendor names, GitHub tooling) from ⚠ (company-reported: users, volumes, fund size, ISO certification) from ❌ (not found: founders, Lithuanian VASP, launchpad).
2. **The restricted-jurisdiction warranty is a control, not a comfort.** "We don't serve Singapore" is only as good as the monitoring that enforces it — for the exchange's own compliance and for the bank's.
3. **PoR is evidence, not collateral.** Merkle trees prove snapshots; banks lend against audited reality, and until an exchange produces independently audited financials, the relationship is a payments-and-settlement one, not a credit one.
4. **The patchwork is the risk.** An MSB-registration footprint (§7) is a strategy for permissive markets, not a substitute for licensing; the §9 record (Binance, FTX, Bybit, MiCA) is the proof that the patchwork consolidates, and the bank must be positioned for the consolidation — including the possibility that its client exits jurisdictions, changes entity structure, or becomes the subject of an enforcement action. The bank that has already documented its exit conditions and its verification baseline is the bank that survives the next FTX.

---

## 11. The Claims Audit — Verified, Flagged, Rejected

### 11.1 The Verified Claims (✅)

| # | Claim | Verification |
| --- | --- | --- |
| 1 | Founded/established 2021; global launch October 2022 | About page; H1 2026 blog; CoinMarketCap; CoinGecko; Tracxn ✅ |
| 2 | Registered in Saint Vincent and the Grenadines | CoinMarketCap; CoinGecko; Tracxn; NewsBTC ✅ |
| 3 | Mission: "making cryptocurrency trading smarter, safer, and more accessible" | About page; H1 2026 blog ✅ (as published) |
| 4 | Product lines: spot, futures, copy trading, Earn | About page; H1 2026 blog; NewsBTC ✅ |
| 5 | Perpetual-futures-only derivatives (as of Oct 2025); USDT-margined | NewsBTC hands-on ✅/⚠ |
| 6 | Tiered risk limits on futures | Bitunix support article (linked) ✅/⚠ |
| 7 | TradingView integration | About page; company blog; NewsBTC ✅/⚠ |
| 8 | Mandatory two-tier KYC (KYC 1/KYC 2); Sumsub vendor; withdrawal limits 500K/5M USDT | Company compliance blog ✅ (as published) |
| 9 | PoR: Merkle tree, 1:1 claim, published ratios (Mar 2026: 109/100/109; Jul 2026: 110/104/104) | Company PoR page/blog; press copies ✅/⚠ |
| 10 | PoR verification tool on GitHub (github.com/BitunixOfficial/proof-of-reserves/) | NewsBTC-linked URL ✅ |
| 11 | Custody partners Cobo and Fireblocks | About page; H1 2026 blog ✅ (as published) |
| 12 | CoinGlass 2025 Annual Report ranks Bitunix No. 7 volume / No. 10 open interest | Company blog citing coinglass.com/learn/2025-annual-report-en ✅/⚠ |
| 13 | H1 2026 registrations claimed: Canada MSB, AUSTRAC, South Africa FSP, Mauritius Investment Dealer; ISO/IEC 27001:2022 | H1 2026 blog ✅ (as published) / ⚠ registry-unverified |
| 14 | Binance Nov 21, 2023 guilty plea and >US$4B resolution | DOJ press release; Reuters ✅ |
| 15 | FTX Chapter 11 filing Nov 11, 2022; ~US$8B shortfall reported | Reuters; CBS News ✅/⚠ |
| 16 | Bybit France exit announced Aug 1, 2024, effective Aug 13, 2024 | Cryptoslate; CCN ✅/⚠ |
| 17 | MiCA: in force June 29, 2023; stablecoin rules June 30, 2024; CASP rules Dec 30, 2024 | EUR-Lex; ESMA ✅ |

### 11.2 The Flagged Claims (⚠)

| # | Claim | Why flagged |
| --- | --- | --- |
| 1 | 5M+ users across 150+ countries (2026) | Company-reported; 2M (CMC), 3M (Jun 2026 boilerplate), 5M (Jul 2026) inconsistency |
| 2 | US$300B+ H1 2026 volume; US$5.34B peak daily; US$5B+ average daily | Company-reported; not reconcilable with aggregator spot volumes (~US$294M 24h on CoinGecko) |
| 3 | Up to 200x leverage | Company claim 2026; NewsBTC reported 125x (Oct 2025); per-contract caps not published |
| 4 | 1,200+ trading pairs | Company claim; 503 live on CoinGecko; 700+ per NewsBTC |
| 5 | "Millisecond-level matching"; slippage <0.01% | Company claims; no independent benchmark |
| 6 | FinCEN MSB and FINTRAC registrations | Press-reported (NewsBTC); registries not queryable this pass |
| 7 | Philippines VASP registration | Press-reported, single-sourced |
| 8 | Seychelles operations base | Single-sourced (NewsBTC); no FSA record found |
| 9 | US$30M Care Fund (2026) vs US$5M Nemean insurance fund (2024–25) | Company-reported; size changed without public explanation; wallet not published |
| 10 | ISO/IEC 27001:2022 certification; Cer.Live AA rating; Hacken audits | Vendor/company claims; certificates not independently verified |
| 11 | No breach to date | Absence-of-evidence (press-reported) |
| 12 | Similarweb No. 3 visit growth; 46.4M visits; 472% YoY | Company-cited third-party data |
| 13 | Derivatives market US$85.7T in 2025 | CoinGlass/Cointelegraph industry estimate, company-curated |
| 14 | Singapore restriction; US/HK/China restriction | Press-reported (NewsBTC); not verified against the exchange's own current jurisdiction list |
| 15 | Singapore/MAS: no licence | Absence-of-evidence; MAS FID not queryable this pass |
| 16 | Earn APRs (USDT 3–11.6%, etc.); fee schedule (VIP 0–7) | Press-reported (NewsBTC); time-varying; company fee page not extracted |
| 17 | "No re-hypothecation" claim | Company claim; not independently audited |
| 18 | P2P 76 fiat currencies; third-party deposits 81 | Company page as published; provider identities not disclosed |

### 11.3 The Rejected or Not-Found Claims (❌)

| # | Claim | Finding |
| --- | --- | --- |
| 1 | Legal entity registered in the Seychelles (believed anchor) | Corrected — registration is Saint Vincent and the Grenadines (multiple sources); Seychelles appears only as an operations base (⚠ single-sourced) |
| 2 | Lithuanian VASP registration (believed anchor) | Not found in any source; targeted searches returned empty |
| 3 | Founders publicly named (believed anchor implied verifiable founders) | Not found — no founder/executive named in any primary source examined |
| 4 | Formal staking product (believed anchor) | Not found — Earn/savings products verified; no on-chain staking product documented |
| 5 | Formal launchpad/pool (believed anchor) | Not found — Token Splash and Task Center are the verified analogues; no IDO/launchpad platform |
| 6 | "~1M+ users" and "hundreds of millions USD 24h volume" (believed anchors) | Superseded — company claims 5M+ users; aggregator 24h spot volume ~US$294M (Sep 2026); no exact match to the brief's figures |

---

## 12. What Could Not Be Verified

The following items could not be confirmed in this pass, despite targeted effort. Each is flagged ⚠ in the body of the guide; none is asserted as fact:

- **The identity of the founders and any executive/leadership team.** No primary source names them; targeted searches returned empty result sets. The individuals may simply be low-profile, so this is recorded as ⚠ (not found), not ❌ (refuted).
- **The operating legal-entity name(s), company number, and registered address.** bitunix.com publishes no entity details on the About, Terms, or support pages examined this pass. The SVG registration is corroborated by three aggregators and one press review, but the registry record itself (SVG International Business Companies registry) could not be queried.
- **The Seychelles operations base and any Seychelles FSA licence.** Single-sourced (NewsBTC); no Seychelles registry record found.
- **The Lithuanian VASP registration (believed anchor).** Not found in any source; the believed anchor could not be confirmed and is recorded ❌ in §11.3.
- **The FinCEN MSB, FINTRAC, and Philippines VASP registrations.** Press-reported only (NewsBTC); FinCEN's MSB registrant list, FINTRAC's MSB search, and the Philippines' registers are not freely queryable by brand name, so registry-level confirmation was not possible.
- **The H1 2026 registrations (Canada MSB, AUSTRAC, South Africa FSP, Mauritius Investment Dealer) and the ISO/IEC 27001:2022 certification.** Company-blog claims; no certificate, licence number, or registry entry was independently verified.
- **Any MAS licence or PS Act notification for a Bitunix entity, and any MAS Investor-Alert-List entry.** Both are absence-of-evidence findings; the MAS Financial Institutions Directory is JavaScript-driven and could not be filtered server-side this pass. (Note: absence from the alert list is also unverified — no claim either way.)
- **The exchange's own current restricted-jurisdiction list.** The US/HK/mainland-China/Singapore restriction is press-reported (NewsBTC); the exchange's own live jurisdiction terms were not extracted this pass.
- **The Care Fund's current size, wallet address, and governance.** US$30M USDC is company-reported (2026); the earlier US$5M Nemean insurance fund is snippet-verified only (cexfinder.com could not be scraped in full this pass). The relationship between the two arrangements is unexplained in public sources.
- **The PoR attestation's independence.** The Merkle-tree construction is standard and the ratios are published, but who audits the tree, how often, and against which wallet set is not public; the on-chain balances were not independently re-aggregated this pass.
- **The trading-volume figures.** Company claims (US$5B+ average daily; US$300B+ H1 2026) are not reconcilable with aggregator spot-volume snapshots (~US$294M 24h on CoinGecko, September 2026) and are not independently auditable.
- **The fee schedule and Earn APRs as current.** The VIP 0–7 table and the Earn APRs are press-reported (NewsBTC, October 2025) and time-varying; the company's own fee page (July 2026 blog) could not be extracted this pass.
- **The 200x vs 125x leverage discrepancy.** Company materials say up to 200x; the October 2025 hands-on review says up to 125x; per-contract maximum-leverage tiers are not published in one place.
- **The formal staking product and a formal launchpad/IDO platform.** Neither is documented; Earn and Token Splash/Task Center are the verified analogues.
- **The API documentation and its capabilities.** Not extracted or verified this pass.
- **The bug-bounty program.** None was found in this pass's searches; absence of evidence only.
- **No enforcement action against Bitunix.** None was found in this pass; recorded as an absence-of-evidence flag, not as a clean-bill assertion.
- **The identities and terms of the third-party fiat payment providers** behind the 81-currency deposit rails, and the P2P marketplace's operational and screening model.
- **The PitchBook profile's description** ("emerging crypto social platform") — it does not match the verified exchange profile and was treated as unreliable; not used for any claim in this guide.

---

## 13. The Glossary

| Term | Meaning |
| --- | --- |
| **AUSTRAC** | Australia's financial-intelligence and AML regulator; its registration is claimed by Bitunix for H1 2026 (⚠ registry-unverified). |
| **Bitunix Care Fund** | Bitunix's claimed protection reserve — US$30M USDC in 2026 materials; earlier reporting described a US$5M insurance fund with Nemean. ⚠ |
| **CASP (Crypto-Asset Service Provider)** | The MiCA licence category for EU crypto service providers; rules applied from December 30, 2024. |
| **CEX (Centralized Exchange)** | A custody-based crypto exchange where the platform holds user assets and matches orders; Bitunix's category. |
| **Copy trading** | Automatically mirroring a lead trader's futures positions; one of Bitunix's four headline product lines. |
| **DCA (Dollar-Cost Averaging)** | Investing fixed amounts on a schedule; Bitunix's Spot Auto-Invest implements it. |
| **DPT (Digital Payment Token)** | MAS's term for crypto assets under the Payment Services Act; DPT services require an MPI licence in Singapore. |
| **Dual Investment** | A structured product where the payout is in either the deposited asset or the target asset depending on the market at settlement. |
| **Easy Earn** | Bitunix's flexible-savings product (idle crypto earning APRs). |
| **EDD (Enhanced Due Diligence)** | The deeper KYC/AML review applied to high-risk clients such as crypto exchanges. |
| **FinCEN** | The US Financial Crimes Enforcement Network; MSB registration with FinCEN is claimed for Bitunix (⚠ press-reported). |
| **FINTRAC** | Canada's financial-intelligence unit; MSB registration claimed (⚠ press-reported / company blog). |
| **FSP (Financial Services Provider)** | The South African licence category claimed by Bitunix for H1 2026 (⚠ registry-unverified). |
| **FTX** | The exchange whose November 2022 collapse (Chapter 11, ~US$8B reported shortfall) reset the industry's trust standards. |
| **Funding rate** | The periodic payment between long and short perpetual-futures positions that keeps the contract price anchored to spot. |
| **Hacken** | A blockchain security auditor; Bitunix publishes Hacken audit pages (⚠ company/vendor-claimed). |
| **IEEPA (International Emergency Economic Powers Act)** | The US statute Binance was found to have violated in the November 2023 resolution. |
| **ISO/IEC 27001:2022** | The information-security management standard; Bitunix claims certification (⚠ not independently verified). |
| **KYC 1 / KYC 2** | Bitunix's two verification tiers: Basic (ID + biometric selfie; 500,000 USDT/24h withdrawals) and Advanced (+ proof of address; 5,000,000 USDT/24h). |
| **Launchpad / Launchpool** | Token-sale/listing platforms on other exchanges (Binance, Bybit); no formal equivalent verified at Bitunix — Token Splash is the nearest analogue. ⚠ |
| **Merkle-tree Proof of Reserves (PoR)** | A cryptographic proof that an exchange's on-chain reserves cover its published user balances; the construction is cross-referenced to the smart-contracts guide. |
| **MiCA (Markets in Crypto-Assets Regulation)** | Regulation (EU) 2023/1114; in force June 29, 2023; stablecoin rules June 30, 2024; CASP rules December 30, 2024. |
| **MPI (Major Payment Institution)** | The MAS licence tier required for DPT exchange services in Singapore. |
| **MPC (Multi-Party Computation)** | A key-management technique that splits signing across parties; reported in Bitunix's cold-storage design (⚠ press-reported). |
| **MSB (Money Services Business)** | The US FinCEN registration category (and the Canadian FINTRAC category) for money-transmission businesses. |
| **Nemean** | The insurance partner behind the earlier US$5M Bitunix insurance fund (⚠ snippet-verified). |
| **Open interest** | The total value of open derivatives positions; Bitunix ranked No. 10 globally on this metric in CoinGlass's 2025 report. |
| **P2P (Peer-to-Peer) trading** | Direct user-to-user fiat/crypto exchange on the platform; Bitunix supports 76 fiat currencies via P2P (company page). |
| **Perpetual swap (perp)** | A futures contract with no expiry, kept near spot by funding payments; Bitunix's only derivatives type (as of Oct 2025). ⚠ |
| **PoR (Proof of Reserves)** | See Merkle-tree Proof of Reserves. |
| **SAR (Suspicious Activity Report)** | The filing made to financial intelligence units on suspicious transactions; Bitunix states it manages automated SARs. |
| **Sumsub** | The KYC/identity-verification vendor Bitunix says powers its automated verification. |
| **SVG (Saint Vincent and the Grenadines)** | The registration jurisdiction of the exchange per CoinMarketCap, CoinGecko, Tracxn, and NewsBTC. |
| **Token Splash** | Bitunix's new-listing promotion surface (deposit and trading tasks for featured tokens). |
| **TradingView** | The charting platform; Bitunix integrates TradingView Pro on web and mobile. |
| **UBO (Ultimate Beneficial Owner)** | The natural person(s) who ultimately own/control a client; the verification challenge for an exchange whose founders are not public. |
| **Ultra K-Line** | Bitunix's branded charting system. |
| **VASP (Virtual Asset Service Provider)** | The generic licence/registration category for crypto businesses; claimed by Bitunix in the Philippines (⚠ press-reported). |
| **VIP tiers** | Bitunix's eight fee-discount levels (VIP 0–7) tied to trading volume. |

---

## 14. Cross-References and the Closing Summary

**Cross-references (repository convention: sibling `banking/` guides by plain filename; `technology/` guides prefixed `../technology/`):**

- [Reap Global](reap_global_guide.md) — the fintech-company profile template and the Cymbal Bank worked-example conventions; the model for this guide's structure (§1, §10) and the source of the Singapore-leg structure used in §10.2.
- [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) — the PS Act, the SPI/MPI tiers, and the DPT measures; cross-referenced (condensed) in §7.4 and §10.2.
- [Fircosoft](fircosoft_guide.md) — sanctions/AML screening themes; cross-referenced (condensed) in §7.3 and §10.2/§10.4.
- [Payment Rails](payment_rails_guide.md) — the SG FAST/PayNow rails and clearing/settlement mechanics; cross-referenced in §10.3.
- [Smart Contracts](../technology/smart_contracts_guide.md) — the on-chain mechanics (ERC-20 stablecoins, Merkle proofs, audit practice, the incident history); cross-referenced (condensed, not re-derived) in §8.3.
- [Cybersecurity](../technology/cybersecurity_guide.md) — the security-discipline themes (hot/cold separation, multi-signature, ISO/IEC 27001, bug bounties, frameworks); cross-referenced (condensed) in §8.1–§8.2.

**Primary sources used this pass:** bitunix.com (About Us; H1 2026 Growth Review, July 8, 2026; the Compliance/KYC blog; the CoinGlass-ranking blog, June 2026; the Proof of Reserves page); coinmarketcap.com/exchanges/bitunix; coingecko.com/en/exchanges/bitunix; tracxn.com/d/companies/bitunix; newsbtc.com/bitunix-review/ (October 2025); the US Department of Justice press release on Binance (November 21, 2023) and United States v. Changpeng Zhao case page; Reuters and CBS News coverage of FTX's Chapter 11 filing (November 11, 2022); Cryptoslate and CCN on Bybit's France exit (August 2024); EUR-Lex (Regulation (EU) 2023/1114), ESMA and Scorechain on MiCA; cexfinder.com (snippet only — the US$5M Nemean insurance-fund headline); github.com/BitunixOfficial/proof-of-reserves (link verified via NewsBTC); hacken.io/audits/bitunix (link verified via NewsBTC); bitfeed.news and advertorial copies of the March 2026 PoR update.

**The closing summary.** Bitunix is the archetype of the new wave of crypto exchanges: a 2021-founded, SVG-registered, derivatives-first venue that has ridden the post-FTX trust recovery to a claimed 5 million users, a CoinGlass top-10 derivatives position, a Merkle-tree Proof of Reserves, and a US$30 million protection fund — while its founders stay anonymous, its licensing stays a patchwork of MSB-type registrations, its headline volumes stay company-reported, and its Singapore status stays "restricted, not licensed." The verified record is clear on the fundamentals: founded 2021, launched October 2022, registered in Saint Vincent and the Grenadines, perpetual-futures-led with up to 200x leverage, two-tier mandatory KYC on Sumsub, Cobo and Fireblocks custody, PoR with published ratios, and a compliance narrative aimed squarely at the Binance/FTX lessons. What cannot be verified — founders, the Seychelles and Lithuanian claims, the registrations' registry status, the volume figures, the fund's economics — is flagged rather than smoothed, per this repository's honesty convention. For Cymbal Bank, the lesson is the one the §10 worked example operationalises: an exchange's claims are marketing until proven, its PoR is evidence rather than collateral, its restricted-jurisdiction list is a control to be tested, and its licensing patchwork is a risk to be documented — because in an industry whose giants were felled by AML failures and unverifiable books, the bank that verifies before it trusts, monitors what it cannot verify, and pre-agrees its exit conditions is the bank that holds the exchange's edge.
