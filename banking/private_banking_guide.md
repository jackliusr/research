# Private Banking: A Comprehensive Guide

**The Private-Banking Deep-Dive — the HNW/UHNW Segments and Thresholds, the Relationship-Manager Model, the Product Shelf (DPM, Advisory, Brokerage, Structured Products, Alternatives, Lombard Lending, Trusts), the Verified Platform Landscape (Avaloq, Temenos, Finacle, Objectway, Profile), the Regulation and Compliance Overlay (KYC/AML, FATCA/CRS, Suitability), the Singapore Hub, and a Cymbal Bank Onboarding Worked Example — Verified Against Primary Sources**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Banking Domain / Wealth Management — the private-banking deep-dive of the repository's wealth cluster: the business of serving high-net-worth (HNW) and ultra-high-net-worth (UHNW) individuals and families — the segment definitions and thresholds (Capgemini World Wealth Report bands, the MAS/SFA statutory anchors), the private-bank vs wealth-manager vs retail-premium distinction, the relationship-manager (RM) coverage model, open architecture vs in-house products, the booking-centre model (Singapore, Hong Kong, Switzerland), the product shelf (discretionary portfolio management, advisory mandates, brokerage, structured products, alternative-investment access, Lombard lending, deposits/treasury, trust and estate structuring), the verified platform landscape (Avaloq, Temenos, Infosys Finacle/EdgeVerve, Objectway, Profile Software), the regulation and compliance overlay (KYC/AML cross-referenced to the Fircosoft guide, FATCA/CRS cross-referenced to the Cayman/BVI guide, MiFID II / HK SFC / SG FAA suitability, MAS expectations cross-referenced to the MAS guide, and the booking-centre vs advice-centre split), the Singapore hub (MAS survey AUM, the bank landscape, family offices), and a Cymbal Bank private-client onboarding worked example. The broad wealth-management sibling ([Wealth Management](wealth_management_guide.md)) covers the whole wealth industry; this guide is the private-banking focus and cross-references it rather than duplicating it.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** capgemini.com (World Wealth Report 2026 press release, 4 June 2026 — the HNWI definition and wealth bands, verified directly this pass), nec.com and avaloq.com press releases (NEC–Avaloq acquisition: announced 5 October 2020, closed 23 December 2020), avaloq.com (platform, clients, scale figures), temenos.com (About — "since 1993") and the Temenos AG article (founding, T24, Odyssey acquisition, 2024 results), the Finacle article (1999 launch, EdgeVerve 2015), objectway.com (founding 1990, scale, client logos, 2026 FNZ/SLIB transactions), profilesw.com (founding 1990, offices), the MAS Singapore Asset Management Survey 2024 media coverage (S$6.1 trillion at end-2024, released 16 July 2025) and the Survey 2025 announcement (S$6.7 trillion), DBS's accredited-investor regime FAQ (the SFA AI thresholds), the MiFID II article (Directive 2014/65/EU applied 3 January 2018), the Lombard-credit article (secured lending mechanics, citing PostFinance, KPMG, Rothschild & Co), finews.asia (Asia private-banking AUM league table 2024; the Endowus RM survey), and the sibling guides in this repository (cross-referenced, not re-derived — per the conventions below). Facts verified this pass are marked ✅ with the source named in the Claims Audit (§12); anything that could not be re-verified is flagged ⚠ honestly and listed again in §12.4. No dates, numbers or claims were invented.
> **Last Updated:** August 2026
> **Companion guides (sibling, same folder — the wealth, funds and capital-markets clusters):** [Wealth Management](wealth_management_guide.md) (the broad wealth-industry sibling — segments, advisory models, DPM, wealthtech, Singapore context — cross-ref, do not duplicate) · [Investment Portfolio Operations](investment_portfolio_operations_guide.md) (the investops lifecycle behind DPM — order → execution → allocation → confirmation → settlement → custody → corporate actions → reconciliation → NAV — cross-ref §6.1) · [Market Making in Singapore](market_making_singapore_guide.md) (margin/collateral mechanics and the house-style conventions this guide mirrors — cross-ref §7, §12) · [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) (the family-office angle — SFO exemption, 13O/13U, 2,000+ SFOs — cross-ref §10.4, do not re-derive) · [Private Equity](private_equity_guide.md) and [Private Equity in Singapore](private_equity_singapore_guide.md) (fund structures and the Singapore fund regime — cross-ref §6.5) · [Citadel LLC](citadel_llc_guide.md) (the alternatives/prime-brokerage archetype — cross-ref §6.5) · [Cayman/BVI Master-Feeder](cayman_bvi_master_feeder_guide.md) (offshore structures and the FATCA/CRS tax content — cross-ref §6.7, §9.2, do not re-derive) · [FircoSoft](fircosoft_guide.md) (sanctions/PEP/EDD screening — cross-ref §9.1, do not re-derive) · [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) (the Singapore regulatory overlay and the Cymbal Bank persona conventions — cross-ref §9.4, do not re-derive) · [Banks in Singapore](banks_in_singapore_guide.md) (the licence tiers and the bank landscape — cross-ref §5.2, §10.2) · [Singapore Trust Companies](singapore_trust_companies_guide.md) (the TCA 2005 trustee layer — cross-ref §6.7, §10.4) · [Singapore Private Markets](singapore_private_markets_guide.md) (the VCC/13O/13U fund regime — cross-ref §10.4) · [Online Investment Trading Platforms](online_investment_trading_platforms_guide.md) (the brokerage layer — cross-ref §6.3)
> **Companion guides (technology/, prefix `../technology/`):** [Zero Downtime System Design](../technology/zero_downtime_system_design_guide.md) (always-on banking estates — the private-bank platform availability requirement) · [SecOps Guide](../technology/secops_guide.md) and [Cybersecurity Guide](../technology/cybersecurity_guide.md) (the client-data protection overlay) · [Architecture Decision Record](../technology/architecture_decision_record_guide.md) (the platform-selection ADR discipline behind §8) · [FinOps Guide](../technology/finops_guide.md) (the cost economics of running a wealth platform at scale)

---

**How to use this guide:** Section 1 is the overview — the short answer and the key-facts table. Section 2 is the private-banking model — the HNW/UHNW segment definitions and thresholds (Capgemini World Wealth Report bands verified directly; the MAS/SFA statutory anchors), the private-bank vs wealth-manager vs retail-premium tier distinction, and the segment economics. Section 3 is the relationship-manager model and the advisory continuum — the RM coverage model, the RM-to-client ratio convention, and execution-only/advisory/discretionary. Section 4 is the open-architecture vs in-house-product tension — the manufacturer vs distributor problem. Section 5 is the booking-centre model — Singapore, Hong Kong, Switzerland, and the booking-centre vs advice-centre split. Section 6 is the product shelf — DPM, advisory mandates, brokerage, structured products, alternatives access, deposits/treasury, trust and estate. Section 7 is Lombard and securities-backed lending — the verified mechanism and the LTV/haircut conventions. Section 8 is the platforms — Avaloq, Temenos, Infosys Finacle/EdgeVerve, Objectway and Profile Software, each verified at primary sources, with a comparison table. Section 9 is regulation and compliance — KYC/AML (cross-referenced to the Fircosoft guide), FATCA/CRS/AEOI (cross-referenced to the Cayman/BVI guide), suitability (MiFID II, the HK SFC, the SG FAA), MAS expectations (cross-referenced to the MAS guide), and cross-border/booking considerations. Section 10 is the Singapore angle — the hub in numbers, the bank landscape, the AUM estimates, family-office demand, and the MAS wealth-management agenda. Section 11 is the Cymbal Bank worked example — a private-client onboarding journey: segmentation, EDD, a Lombard facility with a worked LTV/loan-amount computation, a DPM mandate, and periodic reporting. Section 12 is the claims audit (✅/⚠/❌), with §12.4 "What Could Not Be Verified". Section 13 is the glossary. Section 14 is cross-references. Section 15 is the closing summary. Cross-references follow the repository convention: sibling guides in `banking/` are plain filenames; guides in `technology/` are prefixed `../technology/`. **Integrity convention:** ✅ = verified this pass against a primary or cited source (source named in §12.1); ⚠ = flagged/unverified — press estimate, vendor figure, contested, or not re-verified live; ❌ = refuted or rejected. Nothing in this guide was invented; figures that could not be re-verified are marked ⚠ and listed again in §12.4. The only bank persona in this guide is Cymbal Bank.

---

## Table of Contents

1. [The Overview](#1-the-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Key-Facts Table](#12-the-key-facts-table)
   - 1.3 [Why Private Banking Matters to a Bank](#13-why-private-banking-matters-to-a-bank)
2. [The Private-Banking Model — Segments and Thresholds](#2-the-private-banking-model--segments-and-thresholds)
   - 2.1 [The HNW/UHNW Threshold Conventions — Capgemini World Wealth Report](#21-the-hnwuhnw-threshold-conventions--capgemini-world-wealth-report)
   - 2.2 [Where the Definitions Diverge (⚠)](#22-where-the-definitions-diverge-)
   - 2.3 [The MAS and SFA Thresholds](#23-the-mas-and-sfa-thresholds)
   - 2.4 [Private Bank vs Wealth Manager vs Retail-Premium Tier](#24-private-bank-vs-wealth-manager-vs-retail-premium-tier)
   - 2.5 [The Segment Economics at a Glance](#25-the-segment-economics-at-a-glance)
3. [The Relationship-Manager Model and the Advisory Continuum](#3-the-relationship-manager-model-and-the-advisory-continuum)
   - 3.1 [The RM Role and the Coverage Model](#31-the-rm-role-and-the-coverage-model)
   - 3.2 [The RM-to-Client Ratio Convention (⚠)](#32-the-rm-to-client-ratio-convention-)
   - 3.3 [The Advisory Continuum — Execution-Only, Advisory, Discretionary](#33-the-advisory-continuum--execution-only-advisory-discretionary)
   - 3.4 [The Digital Pressure on the RM Model](#34-the-digital-pressure-on-the-rm-model)
4. [Open Architecture vs In-House Products — the Manufacturer vs Distributor Tension](#4-open-architecture-vs-in-house-products--the-manufacturer-vs-distributor-tension)
   - 4.1 [The Two Poles](#41-the-two-poles)
   - 4.2 [Why the Tension Exists](#42-why-the-tension-exists)
   - 4.3 [The Market Evidence](#43-the-market-evidence)
5. [The Booking-Centre Model](#5-the-booking-centre-model)
   - 5.1 [What Booking Means](#51-what-booking-means)
   - 5.2 [The Three Hubs — Singapore, Hong Kong, Switzerland](#52-the-three-hubs--singapore-hong-kong-switzerland)
   - 5.3 [Booking Centre vs Advice Centre](#53-booking-centre-vs-advice-centre)
6. [The Product Shelf](#6-the-product-shelf)
   - 6.1 [Discretionary Portfolio Management (DPM)](#61-discretionary-portfolio-management-dpm)
   - 6.2 [Advisory Mandates](#62-advisory-mandates)
   - 6.3 [Brokerage](#63-brokerage)
   - 6.4 [Structured Products](#64-structured-products)
   - 6.5 [Alternative-Investment Access](#65-alternative-investment-access)
   - 6.6 [Deposits and Treasury](#66-deposits-and-treasury)
   - 6.7 [Trust, Estate and Structuring](#67-trust-estate-and-structuring)
7. [Lombard and Securities-Backed Lending](#7-lombard-and-securities-backed-lending)
   - 7.1 [The Mechanism (Verified)](#71-the-mechanism-verified)
   - 7.2 [Loan-to-Value and Haircuts (⚠ Conventions)](#72-loan-to-value-and-haircuts--conventions)
   - 7.3 [The Margin-Call Machinery](#73-the-margin-call-machinery)
   - 7.4 [Why Clients Use It](#74-why-clients-use-it)
8. [The Platforms — Verified Profiles](#8-the-platforms--verified-profiles)
   - 8.1 [Avaloq](#81-avaloq)
   - 8.2 [Temenos](#82-temenos)
   - 8.3 [Infosys Finacle / EdgeVerve](#83-infosys-finacle--edgeverve)
   - 8.4 [Objectway](#84-objectway)
   - 8.5 [Profile Software](#85-profile-software)
   - 8.6 [The Comparison Table](#86-the-comparison-table)
   - 8.7 [The Market-Share Question (⚠)](#87-the-market-share-question-)
9. [Regulation and Compliance](#9-regulation-and-compliance)
   - 9.1 [KYC/AML — CDD, EDD, PEPs and Beneficial Ownership](#91-kycaml--cdd-edd-peps-and-beneficial-ownership)
   - 9.2 [FATCA, CRS and AEOI](#92-fatca-crs-and-aeoi)
   - 9.3 [Suitability and Client-Advisory Rules — MiFID II, the HK SFC, the SG FAA](#93-suitability-and-client-advisory-rules--mifid-ii-the-hk-sfc-the-sg-faa)
   - 9.4 [MAS Expectations](#94-mas-expectations)
   - 9.5 [Cross-Border and Booking Considerations](#95-cross-border-and-booking-considerations)
10. [The Singapore Angle](#10-the-singapore-angle)
    - 10.1 [The Hub in Numbers](#101-the-hub-in-numbers)
    - 10.2 [The MAS-Regulated Bank Landscape](#102-the-mas-regulated-bank-landscape)
    - 10.3 [The Private-Banking AUM Estimates (⚠)](#103-the-private-banking-aum-estimates-)
    - 10.4 [Family-Office Demand](#104-family-office-demand)
    - 10.5 [The MAS Wealth-Management Agenda](#105-the-mas-wealth-management-agenda)
11. [The Cymbal Bank Worked Example — A Private-Client Onboarding Journey](#11-the-cymbal-bank-worked-example--a-private-client-onboarding-journey)
    - 11.1 [The Scenario](#111-the-scenario)
    - 11.2 [Segmentation and Onboarding](#112-segmentation-and-onboarding)
    - 11.3 [EDD and the Compliance Overlay](#113-edd-and-the-compliance-overlay)
    - 11.4 [The Lombard Facility — A Worked LTV Computation](#114-the-lombard-facility--a-worked-ltv-computation)
    - 11.5 [The DPM Mandate](#115-the-dpm-mandate)
    - 11.6 [Periodic Reporting and the Operating Rhythm](#116-periodic-reporting-and-the-operating-rhythm)
12. [The Claims Audit — Verified, Flagged, Rejected](#12-the-claims-audit--verified-flagged-rejected)
    - 12.1 [The Verified Claims (✅)](#121-the-verified-claims-)
    - 12.2 [The Flagged Claims (⚠)](#122-the-flagged-claims-)
    - 12.3 [The Rejected Claims (❌)](#123-the-rejected-claims-)
    - 12.4 [What Could Not Be Verified](#124-what-could-not-be-verified)
13. [Glossary](#13-glossary)
14. [Cross-References and Further Reading](#14-cross-references-and-further-reading)
15. [Closing Summary](#15-closing-summary)

---

## 1. The Overview

### 1.1 The Short Answer

Private banking is the relationship-led business of managing the financial lives of high-net-worth (HNW) and ultra-high-net-worth (UHNW) individuals and families — a single named relationship manager (RM) as the client's gateway to the bank's full balance sheet: investments, lending, deposits, foreign exchange, structured products, alternative-investment access and trust/estate structuring. The segment definitions that the industry actually uses were verified directly this pass against the Capgemini World Wealth Report 2026 (the 30th edition, published 4 June 2026): an **HNW individual (HNWI)** holds **investable assets of US$1 million or more**, excluding primary residence, collectibles, consumables and consumer durables, and HNWIs are split into three wealth bands — **Ultra-HNWIs (US$30 million or more)**, **Mid-Tier Millionaires (US$5–30 million)** and **Millionaires Next Door (US$1–5 million)** ✅ (Capgemini WWR 2026 press release). The same report puts the global HNWI population at **25.3 million** (up nearly 2 million in 2025), their combined wealth at a record **US$98.3 trillion** (+8.7% in 2025), and the UHNWI population at roughly **250,000** (+9.4%) ✅ (Capgemini WWR 2026).

The business model rests on a few durable structures: the **RM coverage model** (one banker per client household, backed by product and investment specialists), the **open-architecture vs in-house-product tension** (should the bank distribute third-party funds or manufacture its own?), and the **booking-centre model** (the account is booked — legally and on the balance sheet — in a hub such as Singapore, Hong Kong or Switzerland, which may be a different jurisdiction from where the advice is given). The product shelf is broad but has a clear core: **discretionary portfolio management (DPM)** is the flagship mandate, **Lombard lending** (securities-backed borrowing) is the flagship credit product, and **alternative-investment access** (private equity, hedge funds, private credit) is the flagship differentiator — 88% of HNWIs say they work with multiple wealth firms specifically to access better alternative-investment opportunities ✅ (Capgemini WWR 2026).

The platform layer that runs this business is dominated by a handful of verified vendors: **Avaloq** (founded 1985 in Zürich; acquired by NEC — announced 5 October 2020, closed 23 December 2020), **Temenos** (founded November 1993 in Geneva), **Infosys Finacle** (launched 1999, an EdgeVerve/Infosys product since 2015), **Objectway** (founded 1990 in Italy) and **Profile Software** (founded 1990 in Athens) — each profiled against primary sources in Section 8. And for the Singapore angle that anchors this repository: the Singapore asset-management industry stood at **S$6.1 trillion of AUM at end-2024, up 12%** ✅ (MAS Singapore Asset Management Survey 2024, released 16 July 2025), with the 2025 survey reporting S$6.7 trillion (+10%) ✅ (MAS announcement, 2026), and more than **2,000 single family offices** at end-2024, up ~43% year-on-year ✅ (Reuters, cross-ref [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §7.3).

### 1.2 The Key-Facts Table

| Fact | Value | Status / Source |
| --- | --- | --- |
| HNWI definition (Capgemini) | Investable assets ≥ US$1m, excluding primary residence, collectibles, consumables, consumer durables | ✅ Capgemini WWR 2026 press release (4 Jun 2026) |
| Wealth bands (Capgemini) | Ultra-HNWI ≥ US$30m; Mid-Tier Millionaires US$5–30m; Millionaires Next Door US$1–5m | ✅ Capgemini WWR 2026 |
| Global HNWI population / wealth | 25.3m HNWIs; US$98.3tn wealth (2025, +8.7%); ~250k UHNWIs (+9.4%) | ✅ Capgemini WWR 2026 |
| Alternatives motivation | 88% of HNWIs use multiple firms to access better alternative investments; exclusive single-firm relationships fell 39% (2019) → 19% (2025) | ✅ Capgemini WWR 2026 |
| SG statutory anchor (nearest to a "PB threshold") | SFA accredited investor: net personal assets > S$2m (primary-residence equity capped at S$1m) OR net financial assets > S$1m OR income ≥ S$300k (preceding 12 months) | ✅ DBS accredited-investor FAQ (SFA-based); cross-ref [MAS guide](mas_regulations_guidelines_guide.md) §2.3 |
| RM-to-client ratio | Commonly cited ~1:40–1:100 client households; no authoritative public standard; UHNW books smaller | ⚠ industry convention, varies by bank |
| Avaloq | Founded 1985, Zürich (as BZ Informatik); renamed Avaloq 1996; NEC acquisition announced 5 Oct 2020, closed 23 Dec 2020 | ✅ Wikipedia (citing BBC/American Banker); NEC press release 23 Dec 2020; avaloq.com |
| Temenos | Founded November 1993, Geneva (Koukis/Goodall); T24 launched 30 Sep 2003; Odyssey (wealth) acquired 2010 | ✅ Temenos AG article; temenos.com ("since 1993") |
| Finacle | Launched 1999; EdgeVerve (Infosys subsidiary) since 2015; wealth-management capability added 2008 | ✅ Finacle article |
| Objectway | Founded 1990, Italy; 200+ clients in 16+ countries; >€1tn AUM on platform | ✅ objectway.com (history page, homepage) |
| Profile Software | Founded 1990, Athens; products incl. Axia (core), IMSplus (investment mgmt), FMS.next (funds); Singapore office | ✅ profilesw.com; ⚠ office list via secondary source |
| Singapore AM industry AUM | S$6.1tn at end-2024 (+12%); S$6.7tn (+10%) per the 2025 survey | ✅ MAS Singapore Asset Management Survey 2024 (16 Jul 2025); MAS Survey 2025 announcement |
| Family offices in Singapore | 2,000+ SFOs at end-2024, ~+43% YoY | ✅ Reuters (cross-ref hedge-funds guide §7.3) |
| Lombard lending | Short-term secured lending against pledged securities/life policies; used by UBS, Barclays, Julius Baer, JPMorgan, Rothschild for UHNW clients | ✅ Lombard-credit article (citing PostFinance, KPMG, Rothschild & Co) |
| MiFID II | Directive 2014/65/EU; applied from 3 January 2018 | ✅ MiFID II article (EUR-Lex cross-ref) |
| Asia private-banking AUM (league tables) | Asia's top 10 private banks > US$2tn at end-2024 (+13% YoY) — finews.asia; Asian Private Banker frames Asia-wide PB AUM crossing US$3tn | ⚠ press league tables, methodology differs |

### 1.3 Why Private Banking Matters to a Bank

Private banking is the most profitable way a bank can serve individuals, and it is why the universal banks all run dedicated private-bank arms (cross-ref the retail-bank wealth funnels in [Wealth Management](wealth_management_guide.md) §1.3 and [Banks in Singapore](banks_in_singapore_guide.md) §7). The economics: a UHNW household concentrates deposits, custody assets, lending (Lombard and mortgages), investment fees (advisory/DPM), insurance and structuring fees in one relationship, with far lower per-client servicing cost than retail at the same revenue. The strategic importance is defensive as well as offensive: the HNWI segment is where the bank's balance sheet, its markets franchise and its trust/estate franchise intersect — and where 97% of firms still segment clients primarily by assets under management, a behaviour the 2026 World Wealth Report explicitly flags as failing to capture behavioural signals ✅ (Capgemini WWR 2026). For Cymbal Bank (the repository's bank persona — see the [MAS guide](mas_regulations_guidelines_guide.md) §1 for the persona conventions), private banking is the client-facing layer that consumes the entire repository: the investops lifecycle of [Investment Portfolio Operations](investment_portfolio_operations_guide.md), the margin mechanics of [Market Making in Singapore](market_making_singapore_guide.md), the screening estate of [FircoSoft](fircosoft_guide.md), the structures of [Cayman/BVI Master-Feeder](cayman_bvi_master_feeder_guide.md), and the regulatory overlay of the [MAS guide](mas_regulations_guidelines_guide.md).

---

## 2. The Private-Banking Model — Segments and Thresholds

### 2.1 The HNW/UHNW Threshold Conventions — Capgemini World Wealth Report

The most widely cited definitional convention in the industry is Capgemini's, and this pass verified it directly at the source (the World Wealth Report 2026 press release, capgemini.com, 4 June 2026):

- **HNWI** = high-net-worth individual with **investable assets of USD 1 million or more**, excluding primary residence, collectibles, consumables and consumer durables ✅ (Capgemini WWR 2026).
- The report segments HNWIs into three wealth bands: **Ultra-HNWIs (USD 30 million or more)**, **Mid-Tier Millionaires (USD 5–30 million)** and **Millionaires Next Door (USD 1–5 million)** ✅ (Capgemini WWR 2026).

These are exactly the "classic USD 1m / USD 30m / USD 5m figures" — and they are the anchor for this guide's use of the terms HNW (US$1m+), mid-tier (US$5–30m) and UHNW (US$30m+). The 2026 edition's headline numbers (all ✅, Capgemini WWR 2026 press release): global HNWI wealth reached **US$98.3 trillion** in 2025 (+8.7%, the largest single-year increase since 2018); the global HNWI population grew by nearly **2 million to 25.3 million**; the **UHNWI population stood at roughly 250,000** (+9.4%, the fastest-growing segment for the second consecutive year, with UHNWI wealth +9.7%); and wealth is concentrated — **the top 1% of HNWIs account for 34.8% of HNWI wealth**. Regional colour: Asia-Pacific posted the highest regional HNWI wealth growth (+10.5%) and population growth (+9.4%) in 2025, with Japan and China adding 436,000 and 154,000 millionaires respectively; the US added 736,000 to reach 8.7 million ✅ (Capgemini WWR 2026).

### 2.2 Where the Definitions Diverge (⚠)

The Capgemini convention is the industry's common language, but it is not the only one, and the differences matter for segmentation, product design and regulatory treatment:

- **Bank entry thresholds for "private banking" vary.** Many private banks set their client-entry minimum well above US$1m — commonly US$1m–US$5m of investable assets, with separate tiers (e.g., "private banking" vs "private client"/"family office") above that ⚠ (bank-policy convention; no public standard; each bank's threshold is its own).
- **Other research houses use different UHNW cut-offs.** The US$30m UHNW figure is shared by Capgemini and Knight Frank's The Wealth Report ⚠ (Knight Frank's exact definition not re-verified this pass), while some studies and vendors use US$5m or US$10m as the UHNW boundary ⚠.
- **"Affluent" and "mass affluent" sit below the HNW line** — commonly US$100k–US$1m (the sibling [Wealth Management](wealth_management_guide.md) guide §1.2 tables the full segment ladder) — and are served by retail-premium tiers, not private banks ⚠ (widely used convention, not a single standard).
- **"Investable assets" itself varies** — Capgemini excludes the primary residence, collectibles, consumables and consumer durables ✅; some banks include business-owner liquidity differently, and some private-bank entry tests count total relationship assets including mortgages ⚠ (bank-policy variation).

### 2.3 The MAS and SFA Thresholds

Singapore has **no official MAS definition of "private banking client"** — the regulator does not set an entry threshold for the private-banking segment ⚠ (nothing in the MAS public rulebook captured this pass sets one; the MAS guide's notice map is cross-referenced rather than re-derived). The nearest statutory anchors are the **Securities and Futures Act (SFA)** investor-classification thresholds, which determine who can be sold non-retail products and services without the full retail suitability wrapper:

- **Accredited investor (AI)**, SFA definition as currently applied: an individual with (a) **net personal assets exceeding S$2 million** (with the net equity of the primary residence capped at S$1 million of that threshold), or (b) **net financial assets exceeding S$1 million** (net of related liabilities), or (c) **income of not less than S$300,000 in the preceding 12 months** ✅ (DBS's accredited-investor regime FAQ, which summarises the SFA thresholds; cross-ref the [MAS guide](mas_regulations_guidelines_guide.md) §2.3 for the SFA statutory framing). These are also the thresholds banks use to gate products such as hedge funds and private equity for individuals (cross-ref [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §2 for the investor-classification overlay).
- **Institutional investor** and **expert investor** are the other SFA classes ⚠ (definitions not re-extracted this pass; the sibling guides carry the fund-side usage).

For practical segmentation, most Singapore private banks map their own tiers onto a combination of the SFA classes (for product eligibility) and their own AUM bands (for service levels) ⚠ (bank-policy convention).

### 2.4 Private Bank vs Wealth Manager vs Retail-Premium Tier

The three labels describe overlapping but distinct businesses; the sibling [Wealth Management](wealth_management_guide.md) guide §1.3 and §2 cover the full continuum, and this subsection draws the private-banking boundary:

| Dimension | Retail-premium tier | Wealth manager | Private bank |
| --- | --- | --- | --- |
| Client | Mass affluent / affluent (commonly US$100k–1m ⚠) | HNW (US$1m+) | HNW and UHNW (US$1m+; entry often US$1–5m ⚠) |
| Delivery | Packaged "Premier" tiers, a shared advisor, digital-first | Advisory/DPM mandates, product platforms, often fee-based | Dedicated RM + specialist team; relationship-led, full balance sheet |
| Product breadth | Funds, cards, mortgages, basic investments | Investments + planning; custody often elsewhere (EAM model) | Investments + lending + deposits/treasury + FX + structures + insurance |
| Credit | Mortgages, unsecured | Little or none | Lombard, mortgages, structured financing |
| Booking model | Local retail booking | Often platform/custody-based | Booking-centre model (§5) — SG/HK/CH |
| Economics | Thin margins, volume | Fee margin on AUM | Highest per-client revenue: fees + lending spread + product margins |

The private bank's defining feature is the **relationship**, not the product: the RM owns the client, and the bank monetises the relationship across the whole balance sheet. The wealth manager (and the EAM — external asset manager — variant, which runs discretion over assets custodied at a private bank) is investment-centric; the retail-premium tier is a marketing overlay on the retail bank. Cross-ref [Wealth Management](wealth_management_guide.md) §1.2–§1.3 and §2.4 for the advisory-model comparison table and the fee benchmarks; they are not re-derived here.

### 2.5 The Segment Economics at a Glance

The revenue stack of a private-banking relationship, in rough order of recurrence: (1) **investment fees** — advisory wrap and DPM management fees (commonly 0.4–1.5% of AUM ⚠, per the fee benchmarks in [Wealth Management](wealth_management_guide.md) §2.3); (2) **net interest income** — the spread on Lombard lending and deposits; (3) **product margins** — structured products, fund distribution and insurance commissions (declining under fee-transparency regulation ⚠); and (4) **transaction and FX income**. On the cost side, the RM-to-client ratio (§3.2) is the leverage variable, and the 2026 World Wealth Report quantifies the operational drag: **41% of advisors' time is consumed by operational tasks**, 60% of wealth-management executives say their firms lack a unified client view, and only 17% of HNWIs describe their advisory experience as seamless and personalised ✅ (Capgemini WWR 2026). These are the economics that platform selection (§8) and the operating model (§11) exist to serve.
---

## 3. The Relationship-Manager Model and the Advisory Continuum

### 3.1 The RM Role and the Coverage Model

The relationship manager (private banker) is the client's single named point of contact — the person who knows the family, its businesses, its plans and its risk tolerance, and who coordinates the specialists behind the bank's wall: the **investment counsellor** (portfolio construction and the DPM mandate), the **product specialists** (structured products, alternatives, FX), the **credit officers** (Lombard and mortgage underwriting), the **trust and estate advisors** (structures, wills, succession), and the **bankers' bankers** in the booking centre (operations, settlement, reporting). The model is deliberately a pod: the RM is the visible face; the pod is the delivery machine. Cross-ref the advisory-process and client-lifecycle sections of [Wealth Management](wealth_management_guide.md) §4 for the process detail; this guide keeps the coverage-model view.

Two structural facts shape the model:

1. **The RM is a revenue owner, not an order-taker.** Private-bank RM compensation is heavily tied to net-new money (NNM), revenue and client satisfaction ⚠ (industry-standard design; individual bank scorecards are not public). The RM's book is the bank's franchise: when an RM moves banks, clients frequently move with them — the reason the industry runs on **team moves** and non-solicit clauses ⚠ (well-documented industry behaviour; no single source verified this pass).
2. **The coverage model is tiered by client size.** UHNW clients get a dedicated RM plus a family-office-style team (and often a dedicated banker in each booking centre); HNW clients share the RM's book; the retail-premium tier gets a shared advisor. The tiering is what makes the RM-to-client ratio the key leverage variable.

### 3.2 The RM-to-Client Ratio Convention (⚠)

There is **no authoritative public standard** for how many client households one RM can cover — banks do not publish their coverage ratios — and every figure in circulation is a consulting or press estimate ⚠. The commonly cited convention is roughly **one RM per 40–100 client households** for a mainstream private-banking book, with **UHNW books running materially smaller** (a dedicated RM per family, or a team of several bankers per family for the very largest) and mass-affluent coverage running far larger (one advisor to hundreds or thousands of clients) ⚠ (industry convention; not verified against any primary source this pass — see §12.2). What is verified: the Capgemini WWR 2026 methodology surveyed **1,317 relationship managers across 24 markets** (✅, Capgemini WWR 2026 press release), which at least confirms the RM population is a measurable, surveyed population; and the report's finding that **97% of firms still segment clients primarily by AUM** (✅) implies coverage ratios are still AUM-driven rather than behaviour-driven.

### 3.3 The Advisory Continuum — Execution-Only, Advisory, Discretionary

The service model spans a continuum of client control, and the regulatory obligations rise with the bank's influence:

- **Execution-only.** The client decides; the bank executes. Minimal suitability obligations; self-directed channel (cross-ref [Online Investment Trading Platforms](online_investment_trading_platforms_guide.md) for the brokerage layer). Rarely the core of a private-banking relationship — it is the entry channel.
- **Advisory.** The RM recommends; the client approves each transaction. Requires suitability assessment (know the client's objectives, financial situation, knowledge and experience — §9.3) and records of the advice given. This is the dominant private-banking model in Asia, where clients expect a person to call ⚠ (regional practice; cross-ref [Wealth Management](wealth_management_guide.md) §2.1).
- **Discretionary (DPM).** The client delegates investment decisions to the bank within a written mandate and Investment Policy Statement (IPS); the bank trades without per-trade approval and reports periodically. DPM is the flagship private-banking mandate — the fastest-growing mandate type in Asia and the cleanest recurring-fee line (cross-ref [Wealth Management](wealth_management_guide.md) §5.3 for the DPM deep-dive; the operational lifecycle is Section 6.1 of this guide and [Investment Portfolio Operations](investment_portfolio_operations_guide.md)).

The continuum maps directly to revenue and to suitability burden: execution-only earns pennies per trade; DPM earns recurring basis points with the highest retention and share of wallet ✅-⚠ (the economics are detailed in the sibling's §2.3; the fee ranges there are ⚠ indicative).

### 3.4 The Digital Pressure on the RM Model

The RM model is under measurable pressure from digital and independent channels, and the evidence this pass is concrete:

- **The Endowus survey (⚠ platform-commissioned):** polling 500 HNW respondents, it found 37% of Singapore-based and 57% of Hong Kong-based investors preferred digital platforms for accessing private markets and hedge funds; only 1 in 5 Singapore respondents preferred a relationship manager; and while 45% of Singapore respondents currently used private bankers to invest in private assets, only 21% considered it their preferred route (⚠ survey-sourced; the survey was commissioned by a digital wealth platform, and finews.asia itself flags the bias — cross-ref §12.2).
- **The Capgemini multi-firm data (✅):** exclusive single-firm HNWI relationships halved from 39% in 2019 to 19% in 2025, and 88% of HNWIs work with multiple firms to access alternatives — the RM's monopoly on the client is gone; the RM now competes for wallet share.
- **The augmentation answer (✅):** three-quarters of advisors (76%) want AI-enabled systems to automate routine work so they can focus on client relationships — the industry's own stated direction of travel (Capgemini WWR 2026).

The architectural consequence for a bank: the RM desktop must be an augmented-advice cockpit (client view, suitability state, mandate compliance, next-best-action), not a CRM form — the operating-model implications are worked in Section 11.

---

## 4. Open Architecture vs In-House Products — the Manufacturer vs Distributor Tension

### 4.1 The Two Poles

Every private bank faces the same structural choice in its product shelf: **distribute third-party products (open architecture)** or **manufacture and distribute its own (in-house, closed architecture)** — or, as most large banks do, run a hybrid with a bias. The poles:

- **Pure open architecture.** The bank's shelf is a curated panel of the best funds, notes and mandates from across the industry; revenue comes from distribution fees and platform fees; the client gets best-of-breed selection and visible conflicts. The cost: the bank gives up manufacturing margins and the ability to differentiate on proprietary products.
- **Pure in-house.** The bank's asset-management arm manufactures the funds and structured notes the private bank distributes; revenue includes manufacturing margins and the distribution is captive. The cost: conflicts of interest (distributing the house product because it is the house product), concentration risk, and suitability pressure from regulators.
- **The hybrid (the actual industry state ⚠).** Almost all large private banks run an open-architecture shelf with a tilt toward in-house products — "open architecture with a house bias." The bias is strongest in structured products and private-market funds, where the manufacturing margin is largest ⚠ (industry practice; the specific house-bias ratios are proprietary and not public).

### 4.2 Why the Tension Exists

The manufacturer-vs-distributor tension is structural, not accidental:

1. **Margin.** Distribution of a third-party fund earns a trailer/commission; manufacturing the same fund earns manufacturing margin *plus* distribution margin. The difference is the whole private-banking P&L debate (cross-ref the fee benchmarks in [Wealth Management](wealth_management_guide.md) §2.3, which are ⚠ indicative).
2. **Regulation.** MiFID II's inducement rules (the EU) and MAS's fair-dealing expectations (Singapore) push firms to disclose or ban payments that create conflicts — retrocessions, soft commissions, and in-house-biased advice (cross-ref §9.3–9.4 and the [MAS guide](mas_regulations_guidelines_guide.md) §4.6 for the FEAT framework). The regulatory direction of travel is fee transparency, which erodes the hidden manufacturing margin.
3. **Client trust.** The 2026 World Wealth Report's finding that 88% of HNWIs use multiple firms *specifically* to access better products (✅, Capgemini WWR 2026) is direct market evidence that clients price the distributor's conflicts: they spread assets to escape captive shelves.
4. **Product governance.** Both regimes require every shelf product to carry a defined target market and risk rating, with post-sale monitoring and recall capability — a product-master architecture problem, not a strategy problem (cross-ref [Wealth Management](wealth_management_guide.md) §3.5 for the shelf-governance mechanics).

### 4.3 The Market Evidence

Three verified data points frame the tension this pass:

- **88% of HNWIs work with multiple wealth firms to gain better access to alternative investments** ✅ (Capgemini WWR 2026) — the single strongest signal that distribution breadth, not manufacture, is what clients pay for.
- **Two in three HNWIs (68%) intend to increase their private-equity exposure** ✅ (Capgemini WWR 2026) — the demand side of why private-market access (in-house or panel) is the battleground product.
- **Between 2022 and 2025, an estimated US$1.5 trillion in new assets flowed to competitors of traditional firms** — WealthTechs, single-family offices and robo-advisory platforms capturing clients underserved on product breadth or advice quality ✅ (Capgemini WWR 2026, CEO quote).

For an architect, the tension resolves into a platform requirement: the shelf must support **third-party products and in-house products on equal rails** (same onboarding, valuation, custody, reporting), with the house bias expressed in distribution rules and product governance, not in the plumbing. The platforms in Section 8 differ precisely here — how natively they onboard external funds and mandates alongside the bank's own.

---

## 5. The Booking-Centre Model

### 5.1 What Booking Means

"Booking" is the legal and operational location of the client relationship: the **booking centre** is the jurisdiction whose licensed entity holds the client's account, carries the assets on its balance sheet, executes the legal contracts, and answers to the local regulator. It is distinct from the **advice centre** (where the RM sits and the advice is given) and from the **service centre** (where operations run). The split exists because clients are mobile and hubs are specialised: an Asian entrepreneur may be advised in Singapore, booked in Singapore or Hong Kong, and hold structures in the Cayman Islands — with the bank's booking entity in each hub deciding which product shelf, which deposit-protection regime and which tax-reporting obligations apply.

The booking decision is driven by five things: **regulation** (which licensed entity can offer what to which client class), **tax** (withholding, FATCA/CRS reporting, estate exposure — cross-ref §9.2 and [Cayman/BVI Master-Feeder](cayman_bvi_master_feeder_guide.md) §3.4), **client preference** (jurisdiction stability, confidentiality, proximity), **balance-sheet economics** (funding, liquidity, capital treatment), and **group structure** (which subsidiary owns the client relationship).

### 5.2 The Three Hubs — Singapore, Hong Kong, Switzerland

The classic private-banking booking centres map to the repository's conventions:

| Hub | Booking profile | Notes |
| --- | --- | --- |
| **Switzerland** | The historic offshore-wealth booking centre; bank–client confidentiality tradition now fully overlaid by FATCA/CRS/AEOI (cross-ref [Cayman/BVI Master-Feeder](cayman_bvi_master_feeder_guide.md) §3.4) | Home of the platform vendors (§8) and of the classic universal private banks ⚠ (Swiss specifics not re-verified this pass) |
| **Singapore** | The Asian onshore booking centre of choice: MAS-regulated banks (full/wholesale licence tiers — cross-ref [Banks in Singapore](banks_in_singapore_guide.md) §3), no capital-gains tax, estate duty abolished 2008 (cross-ref [Singapore Trust Companies](singapore_trust_companies_guide.md) §1.4), S$6.1tn AM industry (§10) | The booking entity is usually a MAS-licensed full or wholesale bank; the advice centre may be the same or a branch elsewhere |
| **Hong Kong** | The other Asian booking centre; the tax-competition rival that Singapore answered in August 2026 with a competitiveness package (cross-ref [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §8.4) | HK specifics ⚠ not re-verified this pass |

The Singapore booking-centre mechanics are the ones this repository verifies deepest — the licence tiers, the MAS FI Directory, and the Cymbal Bank persona conventions all live in [Banks in Singapore](banks_in_singapore_guide.md) and the [MAS guide](mas_regulations_guidelines_guide.md), and are cross-referenced, not re-derived, here.

### 5.3 Booking Centre vs Advice Centre

The regulatory crux of cross-border private banking is that **the licence follows the activity**: giving advice to a client in a jurisdiction is generally a regulated activity *in that jurisdiction*, while the account sits in the booking centre's licence. The practical patterns:

- **Advice and booking in the same hub** — the clean case (Singapore client, Singapore bank, Singapore booking).
- **Booking here, advice there** — a Swiss bank's Singapore branch booking a client advised by an RM in the branch; the branch holds the licence for both, so the split is internal (the "branch model" ⚠ — standard industry structure, cross-ref [Banks in Singapore](banks_in_singapore_guide.md) §3.1 for the wholesale-branch tier).
- **Cross-border solicitation** — an RM in one jurisdiction actively soliciting a client in another raises licensing and marketing rules in the client's jurisdiction (e.g., the EU's cross-border marketing rules under MiFID II; the HK SFC's conduct requirements) ⚠ (regime-specific detail not re-derived this pass).
- **The MAS angle** — MAS regulates the Singapore booking entity and the Singapore advice; cross-border business into Singapore from unlicensed foreign entities is the classic regulatory risk that the MAS guide's licensing chapters cover (cross-ref [MAS guide](mas_regulations_guidelines_guide.md) §2.3–§2.6).

For an architect, the booking-centre model is a **multi-entity, multi-jurisdiction data problem**: the same client, the same mandate, and the same positions exist in the advice centre's CRM, the booking centre's books, and the reporting estate — which is precisely why the platform vendors in Section 8 market multi-booking-centre support, and why the worked example in Section 11 books a Singapore relationship through a MAS-licensed entity.
---

## 6. The Product Shelf

### 6.1 Discretionary Portfolio Management (DPM)

DPM is the flagship mandate: the client signs a **mandate agreement** and an **Investment Policy Statement (IPS)** — investment objective, risk tolerance, asset-class ranges, allowed and prohibited instruments, benchmarks, rebalancing bands — and the bank's portfolio managers manage the portfolio within those rails without per-trade approval. The operational lifecycle is the investops lifecycle of the sibling [Investment Portfolio Operations](investment_portfolio_operations_guide.md) guide and is **cross-referenced, not re-derived** here: portfolio construction → order → execution → allocation → confirmation/affirmation → settlement → custody → corporate actions → reconciliation → valuation/NAV → performance and reporting. The private-banking specifics this guide adds:

- **Mandate types.** Discretionary mandates range from model-portfolio execution (client's IPS mapped to a house model) to fully bespoke mandates (single-family-portfolio construction with direct alternatives allocations) ⚠ (industry convention).
- **Fee.** Typically a management fee of ~0.4–1.5% of AUM ⚠ (indicative benchmarks, per [Wealth Management](wealth_management_guide.md) §2.3), often tiered and negotiated at UHNW scale.
- **Suitability overlay.** DPM is the highest-suitability-burden mandate: the suitability assessment (§9.3) is done at mandate inception, and the mandate itself must stay within the client's risk profile — the ongoing compliance check is mandate-vs-profile, not trade-vs-profile (cross-ref [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §9 for the guideline-checking machinery).
- **The Cymbal Bank operating rhythm** for a DPM book is worked in §11.5–11.6.

### 6.2 Advisory Mandates

In advisory mode the RM recommends and the client approves. The bank must record the recommendation, the suitability basis and the client's decision (or refusal) — the audit trail is the regulatory deliverable (§9.3). Advisory is the dominant Asian private-banking model ⚠ (regional practice), and the economics are a wrap fee (~0.5–1.5% ⚠) plus product margins. The [Wealth Management](wealth_management_guide.md) sibling §4 carries the full advisory-process detail and a worked suitability example; this guide cross-references it.

### 6.3 Brokerage

The execution layer — equities, bonds, FX, funds and derivatives execution for self-directed and advisory clients. The private-bank brokerage differs from retail brokerage in what it sits next to: the same positions feed the Lombard collateral engine (§7), the custody estate, and the reporting. The platform and order-routing mechanics are covered by [Online Investment Trading Platforms](online_investment_trading_platforms_guide.md) and the FIX-based order infrastructure; cross-ref, do not re-derive.

### 6.4 Structured Products

Structured notes — principal-protected notes (PPNs), yield-enhancement notes, autocallables and reverse convertibles — are the classic Asian private-banking staple: a derivative embedded in a note wrapper, sold with a defined coupon and defined downside ⚠ (product mechanics are standard market knowledge; the sibling [Wealth Management](wealth_management_guide.md) §3.1 tables the product set). The private-banking specifics:

- **Issuer risk + product risk + complexity risk** — the client bears the issuer's credit (the bank or a third-party issuer) and the derivative's behaviour; suitability regimes require complexity to be matched to client knowledge (cross-ref §9.3).
- **The margin story** — structured products carry embedded margins (commonly cited ~1–3% ⚠, per [Wealth Management](wealth_management_guide.md) §2.3), which is why they are the classic house-bias product (§4).
- **Regulatory memory** — structured-product mis-selling was a defining post-2008 scandal in Asia (the Lehman minibonds episode in Hong Kong/Singapore ⚠ — well-documented history, not re-verified this pass), and the remediation regimes that followed still shape product governance.

### 6.5 Alternative-Investment Access

Alternatives access is the private bank's competitive differentiator — 88% of HNWIs work with multiple firms specifically to access better alternative opportunities ✅ (Capgemini WWR 2026), and 68% intend to increase private-equity exposure ✅ (Capgemini WWR 2026). The product set and its repository homes:

- **Private equity / private credit** — fund and co-investment access; the structures (LP/GP, subscription lines, carry) and the Singapore fund regime live in [Private Equity](private_equity_guide.md) and [Private Equity in Singapore](private_equity_singapore_guide.md); the SFA investor-classification gate (§2.3) governs who may be sold them.
- **Hedge funds** — direct mandates and fund-of-one structures; the manager landscape and the MAS fund-management regime live in [Hedge Funds in Singapore](hedge_funds_singapore_guide.md); the multi-strategy archetype is profiled in [Citadel LLC](citadel_llc_guide.md).
- **Offshore fund vehicles** — the Cayman/BVI master-feeder architecture that most alternatives flow through is the subject of [Cayman/BVI Master-Feeder](cayman_bvi_master_feeder_guide.md) (structures, side pockets, the FATCA/CRS and PFIC/UBTI tax drivers).
- **The bank's role** — the private bank distributes, holds custody, provides the subscription financing (often via the Lombard line or a subscription facility) and reports the positions; the actual fund administration is the funds cluster's business.

The demand data is unambiguous and verified; the supply mechanics (allocation committees, secondary-market access, private-markets platforms) are ⚠ industry practice not re-verified this pass.

### 6.6 Deposits and Treasury

The liability side of the relationship: multi-currency deposits, time deposits, cash management for the family's operating entities, FX conversion and hedging, and occasionally treasury services for the family business. Deposits are the funding source for the bank's Lombard book, which is why the RM relationship and the balance sheet are inseparable. The treasury and ALM mechanics of the bank side live in [Treasury & ALM](treasury_alm_guide.md) (cross-ref); the client-side deposit product is standard banking ⚠.

### 6.7 Trust, Estate and Structuring

The structuring layer that makes private banking "private": trusts, foundations, holding companies, family offices and succession planning. The repository homes, cross-referenced not re-derived:

- **The Singapore trust layer** — the Trust Companies Act 2005 regime, the MAS-licensed trustee industry, private trust companies (PTCs) and the trust-plus-fund architecture: [Singapore Trust Companies](singapore_trust_companies_guide.md) (the dedicated sibling; note its name-collision warning about "Trust Bank").
- **Offshore structures** — Cayman/BVI vehicles, master-feeder architecture, economic substance: [Cayman/BVI Master-Feeder](cayman_bvi_master_feeder_guide.md).
- **Family offices** — the SFO exemption, the 13O/13U fund tax incentives and the 2023–2025 tightening: [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §7 (summarised in §10.4 of this guide).

The bank's structuring revenue (trustee fees, structure administration, custody of structure assets) is a stable, sticky fee line — and the AML burden of structures (beneficial-ownership look-through, §9.1) is the price of that stickiness.

---

## 7. Lombard and Securities-Backed Lending

### 7.1 The Mechanism (Verified)

**Lombard lending** (also called a Lombard loan or securities-backed line) is a form of **short-term secured lending against movable collateral — today mostly account balances, securities or life-insurance policies** ✅ (Lombard-credit article, citing PostFinance's "Lombard loans: money loaned against a pledge of collateral", KPMG's "Lombard loans", and Rothschild & Co's "Free up cash without selling your portfolio"). The term derives from the Lombard merchants and bankers of Northern Italy, whose secured-lending techniques spread across medieval Europe — London's Lombard Street (1598) is named for them ✅ (Lombard-credit article, citing etymonline). In the private-banking context, the mechanics are:

1. The client pledges an investment portfolio (or a defined sub-set of it) to the bank as collateral.
2. The bank advances a loan up to a **loan-to-value (LTV) ceiling** — a percentage of the collateral's current market value that varies by asset class (the **haircut** is the complement: a 70% LTV means a 30% haircut).
3. The client draws the cash for any purpose — consumption, business liquidity, a property purchase, a fund subscription — **without selling the portfolio**, so no capital-gains event is triggered ✅ (Lombard-credit article: "investments do not have to be sold, and thus do not trigger capital gains tax").
4. The line is priced at a reference rate plus a spread; because it is secured against high-quality liquid assets, the rate is lower than unsecured borrowing ✅ (Lombard-credit article).
5. The bank monitors the collateral continuously; if the portfolio falls and the LTV breaches its ceiling, the bank issues a **margin call** — the client must top up cash/collateral or repay, failing which the bank sells pledged assets.

The private banks that offer it to UHNW clients include UBS, Barclays, Julius Baer, JPMorgan and Rothschild & Co ✅ (Lombard-credit article) — the mechanism is standard across the industry, and the repo's margin/collateral mechanics (portfolio margin, haircuts, intraday margin calls, liquidation rights) are cross-referenced from [Market Making in Singapore](market_making_singapore_guide.md) §10.3 rather than re-derived.

### 7.2 Loan-to-Value and Haircuts (⚠ Conventions)

The LTV ceilings below are **directional market conventions, not verified law or any single bank's published grid** ⚠ — every bank sets its own credit policy, and the grid varies by institution, client and market conditions. The shape, however, is standard across the industry:

| Collateral class | Typical LTV ceiling ⚠ | Implied haircut ⚠ |
| --- | --- | --- |
| Cash / money-market funds | 90–100% | 0–10% |
| Government bonds (high-grade) | 80–95% | 5–20% |
| Investment-grade corporate bonds | 70–85% | 15–30% |
| Blue-chip / index equities | 60–80% | 20–40% |
| Single-name / small-cap equities | 40–60% | 40–60% |
| Funds (multi-asset) | 50–70% | 30–50% |
| Hedge-fund / PE fund interests | 0–30% | 70–100% (often unmargined) |
| Structured products | 0–50% | 50–100% (often unmargined) |

Additional overlays are universal in design ⚠: **concentration haircuts** (a single issuer capped, e.g., at 10–20% of the collateral pool), **currency mismatch haircuts** (borrowing in one currency against collateral in another), and **volatility-based re-pricing** (haircuts widen in stress — the 2008 lesson that margin calls cluster exactly when liquidity vanishes; cross-ref [Market Making in Singapore](market_making_singapore_guide.md) §10.3 and [Citadel LLC](citadel_llc_guide.md) §2.4 for the stress-case convention).

### 7.3 The Margin-Call Machinery

The Lombard control loop is a continuous collateral-management process, not a periodic review:

1. **Collateral valuation** — daily (or intraday in stress) mark-to-market of the pledged pool; the valuation hierarchy and pricing discipline are cross-referenced from [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §6.
2. **LTV computation** — loan balance ÷ collateral value, tested against the ceiling per the applicable haircut grid.
3. **The margin call** — on breach, the client is given a cure period (commonly a small number of business days ⚠) to top up cash, pledge more collateral or repay part of the loan; the bank holds the contractual right to sell pledged assets if the call is not met (cross-ref the liquidation-rights convention in [Market Making in Singapore](market_making_singapore_guide.md) §10.3).
4. **The hard floor** — below a second, lower threshold (the "sell-out" level ⚠ convention), the bank liquidates without further notice. The two-threshold design (call level, then sell-out level) is standard industry practice ⚠.

### 7.4 Why Clients Use It

The verified benefits (✅, Lombard-credit article): no forced sale of investments, hence **no capital-gains trigger**; **lower rates** than unsecured borrowing because the loan is secured against high-quality liquid assets; and **fast access to cash** (funds can be accessed quickly). The classic uses in private banking: liquidity without selling (the entrepreneur who will not crystallise a gain), bridge financing ahead of a sale, and — increasingly — **subscription financing into alternatives** (drawing the Lombard line to fund a private-equity capital call rather than selling the public portfolio). The risks are the mirror image: leverage amplifies losses, margin calls cluster in downturns, and a client who cannot meet a call loses pledged assets at the worst moment — the reason suitability and credit policy (not the RM's optimism) set the LTV.
---

## 8. The Platforms — Verified Profiles

This section profiles the private-banking/wealth platform vendors the task brief named, **each verified at primary sources this pass** (vendor sites and press) — nothing here is from memory. The comparison table (§8.6) and the market-share caveat (§8.7) follow. The vendor landscape sits inside the wider wealth-platform picture of [Wealth Management](wealth_management_guide.md) §6 (the wealthtech stack and core-wealth-platform selection criteria); this section is the verified vendor-by-vendor profile.

### 8.1 Avaloq

- **Founded:** 1985, in Zürich, under the name **BZ Informatik Aktiengesellschaft**, as the information-technology subsidiary of BZ Bank; founder **Francisco Fernandez**. The name was changed to **Avaloq in 1996**; the company split from BZ Bank in 2001 when employees acquired 100% of the share capital ✅ (Avaloq article, citing BBC News, American Banker and Avaloq's own history page).
- **The NEC acquisition:** NEC Corporation announced the acquisition of Avaloq Group AG on **5 October 2020** ✅ (NEC press release, 5 Oct 2020; avaloq.com news; Warburg Pincus announcement — the seller), with press reporting the price at **US$2.2 billion** ✅ (FT via the Avaloq article). The transaction **closed on 23 December 2020** — "Avaloq's acquisition by NEC successfully completed", announced jointly by Avaloq and NEC on that date ✅ (NEC press release, 23 Dec 2020; avaloq.com news). **⚠-correction:** the "completed in 2021" version of the timeline that circulates in some briefs is **❌ wrong** — the NEC press record is unambiguous that closing was 23 December 2020 (see §12.3).
- **Today (⚠ vendor figures unless noted):** avaloq.com (homepage, this pass) states **170+ clients across 35 countries** and **US$5.1 trillion of assets under management on Avaloq software**, with client logos including HSBC, LGT, Coutts, Deutsche Bank, RBC, DBS, Barclays, Maybank, Vontobel, Nomura and Raiffeisen ✅ (avaloq.com homepage — the client names are the vendor's own published list; the scale figures are vendor-reported ⚠). The **Avaloq Banking Suite** is the core-banking and wealth-management platform (front-to-back: client engagement, portfolio management, banking operations, payments, treasury), delivered on-premises, in the cloud, or as BPO/BPaaS — Avaloq has run **business-process-outsourcing centres including Singapore since 2015** ✅ (Avaloq article: "In 2015, Avaloq went live with its two BPO centers in Germany and Singapore").
- **Private-banking fit:** the archetypal Swiss-origin private-banking core — multi-booking-centre support (Switzerland, Singapore, Hong Kong deployments are its heritage ⚠ — deployment lists per client are not fully public), strong securities/portfolio accounting, and a large BPO estate. Market-share claims ⚠ (§8.7).

### 8.2 Temenos

- **Founded:** **November 1993 in Geneva** by **George Koukis and Kim Goodall**, who acquired the rights to the GLOBUS banking platform; the company took the name Temenos (after the Temenos Academy) and was **listed on the SIX Swiss Exchange in 2001** ✅ (Temenos AG article; temenos.com About page: "a true pioneer in core banking software since 1993"). Temenos's own About page adds the current scale framing: **over 950 core-banking and 600 digital-banking clients** ⚠ (vendor figure, temenos.com).
- **The banking products:** **T24** (the flagship core-banking package, launched 30 September 2003, based on GLOBUS) — now marketed as Temenos Transact — plus Temenos Digital, Temenos Payments and Temenos Fund Administration ✅ (Temenos AG article).
- **The wealth line:** Temenos's wealth-management capability traces to the **2010 acquisition of Odyssey Financial Technologies (Luxembourg)** ✅ (Temenos AG article, acquisitions table) — the origin of the **Temenos WealthSuite/Wealth** portfolio-management line ⚠ (the current product branding was not re-verified this pass). Temenos also acquired **Multifonds** (fund administration) in 2015 and **sold Multifonds to Montagu Private Equity in February 2025 for approximately US$400 million**, refocusing on core and digital banking ✅ (Temenos AG article).
- **Scale and governance colour:** revenue **US$1.04 billion in 2024** ✅ (Temenos Annual Report 2024 via the Temenos AG article); the claim of use by **3,000+ financial institutions in 145 countries** and "41 of the top 50 banks" is ⚠ vendor-claimed; and the **February 2024 Hindenburg Research report** alleging accounting irregularities — denied by Temenos, with an independent review in April 2024 finding the allegations "inaccurate and misleading" — is verified as an event ✅ (Temenos AG article), with the merits ⚠ contested.
- **Private-banking fit:** a universal-banking-core vendor whose wealth module (Odyssey-derived) serves private banks, but whose centre of gravity is retail/corporate core banking ⚠ (positioning assessment). The Temenos data model is profiled in depth in the sibling [Temenos Data Model](temenos_data_model_guide.md) (cross-ref).

### 8.3 Infosys Finacle / EdgeVerve

- **Launched:** **1999** as a core-banking software suite by **Infosys**; in 2002 Infosys consolidated its banking products (BankAway, PayAway) under the Finacle brand; **since 2015 Finacle sits inside EdgeVerve Systems, Infosys's software-products subsidiary** ✅ (Finacle article).
- **Wealth capability:** Finacle added **wealth management, Islamic banking, mobile banking and rural banking capabilities in 2008** ✅ (Finacle article — the Finacle 10 launch). The current product family (Finacle Core Banking, Finacle Digital Engagement Hub, etc.) ⚠ (current product naming not re-verified this pass).
- **Scale:** **6,000 employees in 2015** ✅ (Finacle article); the claim of use by banks in **over 100 countries (2020)** ⚠ (vendor claim via the Finacle article).
- **Private-banking fit:** a universal-banking core with wealth modules — strong in retail/universal banking across emerging markets; its private-banking depth is less specialised than Avaloq's or Objectway's ⚠ (positioning assessment; no wealth-specific market data verified this pass). The task brief's "Finacle/Edge" reading maps to EdgeVerve as the operating subsidiary ✅ (Finacle article).

### 8.4 Objectway

- **Founded:** **1990, in Italy**; founder and Group CEO **Luigi Marciano**; the company's own history page frames four phases — pioneering wealth fintech (1990–2001), Italian market leader (2002–2012), EMEA SaaS leader (2013–2021), global growth (2022–) ✅ (objectway.com history page).
- **Scale (⚠ vendor figures):** **200+ clients in 16+ countries**, **over €1 trillion of assets under management on its platform**, **800+ employees** representing 30 nationalities, supporting **100,000+ investment professionals** ✅ (objectway.com — the numbers are the vendor's own, hence ⚠ for the AUM figure; the client logos include KBC, BNP Paribas, EFG, Rathbones, Raymond James, National Bank of Canada, Hauck Aufhäuser Lampe, Oddo BHF and BNY Pershing ✅ objectway.com).
- **Scope:** the **Objectway Growth Platform** for **wealth management, asset management and banking** — a front-to-back SaaS platform covering private and investment banks, retail/neo banks, wealth managers, asset managers, depositary banks, life/pension insurers and investment platforms ✅ (objectway.com).
- **2026 moves:** Objectway announced the **acquisition of a Swiss private-banking technology business from FNZ** (strengthening its Swiss/Liechtenstein footprint and end-to-end private-bank offering) and **exclusive negotiations to acquire SLIB**, the French capital-markets software specialist held by BNP Paribas and Natixis ✅ (objectway.com news, 2026).
- **Private-banking fit:** the European wealth-tech specialist — front-to-back wealth/private-banking platform with a strong European cross-border and now Swiss private-banking pedigree ✅ (objectway.com positioning).

### 8.5 Profile Software

- **Founded:** **1990**, Athens, Greece; "a specialised financial software solutions provider with offices in key financial centres and a presence in 50+ countries across Europe, the Middle East, Asia, Africa and the Americas, delivering solutions to the Investment Management and Banking industries" ✅ (profilesw.com, About page).
- **Products (⚠ product-list detail via PitchBook, not re-verified at vendor pages this pass):** **Axia** (core banking), **IMSplus** (investment management), **FMS.next** (fund management and payments), **RiskAvert** (risk), a **native wealth-management mobile app**, and a markets suite ⚠ (PitchBook company profile; product currency not re-verified this pass).
- **Offices:** internationalbanker.com lists Geneva, Dubai, London, **Singapore**, Athens and Nicosia ⚠ (secondary source; the Singapore office's status was not re-verified at profilesw.com this pass).
- **Private-banking fit:** a regional (EMEA) wealth/banking software player — investment-management and funds strength, with banking (Axia) and wealth-app coverage; smaller footprint than the leaders ⚠ (positioning assessment).

### 8.6 The Comparison Table

| Vendor | Founded (✅) | Origin / HQ | Core scope | Booking-centre support | Bank segments served |
| --- | --- | --- | --- | --- | --- |
| **Avaloq** | 1985 (Zürich, as BZ Informatik) | Switzerland; NEC-owned since 23 Dec 2020 | Front-to-back core banking + wealth (Avaloq Banking Suite) + BPO/BPaaS | Strong multi-booking-centre heritage (CH/SG/HK) ⚠ | Private banks, wealth managers, universal banks (HSBC, LGT, Coutts, DBS, Nomura, Vontobel — vendor list ✅) |
| **Temenos** | Nov 1993 (Geneva) | Switzerland; SIX-listed | Universal core banking (Transact/T24) + digital + wealth (Odyssey-derived WealthSuite ⚠) + fund admin (Multifonds, sold 2025) | Multi-entity/multi-country core banking; wealth booking via WealthSuite ⚠ | Retail, corporate, universal and private banks; 3,000+ FIs ⚠ vendor claim |
| **Finacle (EdgeVerve/Infosys)** | 1999 | India (Infosys); EdgeVerve since 2015 | Universal core banking + digital engagement; wealth capability since 2008 | Multi-branch/multi-country universal banking ⚠ | Retail/universal banking across 100+ countries ⚠ vendor claim; wealth less specialised ⚠ |
| **Objectway** | 1990 | Italy | Front-to-back wealth, asset & banking platform (SaaS) | European cross-border wealth; Swiss PB business acquired from FNZ (2026) | Private/investment banks, wealth managers, asset managers, depositary banks (KBC, EFG, Rathbones, Raymond James ✅ vendor list) |
| **Profile Software** | 1990 | Athens, Greece | Banking (Axia), investment management (IMSplus), funds (FMS.next), risk, wealth app | Regional EMEA footprint; Singapore office ⚠ (secondary) | Investment managers, funds, banks — regional scale ⚠ |

### 8.7 The Market-Share Question (⚠)

No reliable, independently audited market-share ranking for private-banking platforms was verified this pass ⚠. The industry press (finews, Asian Private Banker, IBS Intelligence) publishes vendor-commissioned or analyst-framed share figures, but the methodologies (installed-base counts vs AUM on platform vs revenue) are not comparable, and no primary source captured this pass publishes a definitive ranking. The honest summary: **Avaloq is the most visible private-banking-specialist core** (its client list and BPO estate are the strongest primary-source evidence), **Temenos is the largest banking-software vendor by revenue** (US$1.04bn in 2024 ✅, Temenos Annual Report 2024), and **Objectway is the strongest European wealth-specialist challenger** — but any finer ranking is ⚠ and belongs in §12.4.
---

## 9. Regulation and Compliance

### 9.1 KYC/AML — CDD, EDD, PEPs and Beneficial Ownership

The AML/KYC overlay for private banking is the same discipline the repository derives in full in the screening siblings, and this guide **cross-references rather than re-derives**:

- **The four-layer stack** — CDD → Screening → Transaction Monitoring → Sanctions Compliance, with case management as the connective fifth layer — is laid out in [RegTech](regtech_guide.md) §4.1 ✅; the screening discipline itself (exact vs fuzzy matching, threshold tuning, disposition, list-change rescreening) is derived in [FircoSoft](fircosoft_guide.md) §3 and §5 ✅.
- **CDD vs EDD for private clients:** standard onboarding is CDD — identity, address, source of wealth, source of funds. **EDD** applies to higher-risk clients: **PEPs** (politically exposed persons), high-risk jurisdictions, complex structures, and clients whose source of wealth is hard to evidence. EDD customers get stricter screening thresholds and more frequent refresh ✅ (fircosoft_guide.md §5.1, citing regtech_guide.md §4.2). The **PEP** concept — a person entrusted with a prominent public function, whose family and close associates are also treated as higher risk — is standard FATF-derived practice ✅ (cross-ref [FircoSoft](fircosoft_guide.md) §1.1/§5.1; not re-derived here).
- **Beneficial ownership** is the private-banking crux: the client may be a trust, a holding company or a family office, and the bank must look through to the **ultimate beneficial owners** (UBOs) — the layered-KYC problem is worked in the fund context in [Cayman/BVI Master-Feeder](cayman_bvi_master_feeder_guide.md) §10.5, and the same look-through logic applies to private-client structures ⚠ (pattern transfer; the Cayman 25% ownership/control threshold for the beneficial-ownership register is verified there).
- **The Singapore binding layer:** **MAS Notice 626** (Prevention of Money Laundering and Countering the Financing of Terrorism – Banks) is the bank AML/CFT rulebook — CDD, beneficial ownership, ongoing monitoring, STR filing, sanctions screening, record-keeping, board/MLRO accountability ✅ (mas_regulations_guidelines_guide.md, verified from the on-disk MAS catalogue, via fircosoft_guide.md §4.5); MAS maintains its own Lists of Designated Individuals and Entities ✅ (cross-ref).

### 9.2 FATCA, CRS and AEOI

The tax-transparency overlay is verified in the offshore-structures sibling and **cross-referenced, not re-derived**:

- **FATCA** — the Foreign Account Tax Compliance Act, Chapter 4 of the US Internal Revenue Code (IRC §§1471–1474) — imposes **30% withholding on withholdable payments to non-compliant foreign financial institutions (FFIs)**; FFIs must register with the IRS and report US accountholders, or rely on their jurisdiction's intergovernmental agreement (IGA) with the United States ✅ (cayman_bvi_master_feeder_guide.md §3.4, verified at law.cornell.edu). For a private bank, FATCA classification is part of the client-onboarding data model: every account holder is classified (US person vs non-US person, FFI vs non-FFI) and reported.
- **CRS** — the OECD Common Reporting Standard — requires the same account-holder due diligence and reporting among participating jurisdictions; Cayman and the BVI implement it, and Singapore does too ✅ (cayman_bvi_master_feeder_guide.md §3.4; cross-ref the [MAS guide](mas_regulations_guidelines_guide.md) for the Singapore implementing regulations ⚠ — the specific Singapore CRS regulations were not re-extracted this pass).
- **AEOI** (automatic exchange of information) is the umbrella: FATCA is the US bilateral variant; CRS is the multilateral standard ✅ (standard terminology; the mechanics live in the sibling).
- **The private-banking consequence:** the Swiss-era model of confidential, unreported offshore accounts is dead; every private-banking relationship now carries an annual reporting obligation, and the platform's tax-reporting engine (classification, account reports, self-certifications) is a core requirement — the worked example touches it in §11.6.

### 9.3 Suitability and Client-Advisory Rules — MiFID II, the HK SFC, the SG FAA

**The EU — MiFID II.** Directive 2014/65/EU (MiFID II) and Regulation (EU) No 600/2014 (MiFIR) **applied from 3 January 2018** ✅ (MiFID II article; EUR-Lex cross-ref). The suitability rule — Article 25(2) — requires an investment firm providing **investment advice or portfolio management** to obtain the information necessary to assess the client's **knowledge and experience**, **financial situation** (including ability to bear losses) and **investment objectives** (including risk tolerance), and to act only where the transaction or mandate is suitable ✅ (MiFID II framework — the Article 25(2) reference and the delegated-rule detail (Commission Delegated Regulation (EU) 2017/565, Articles 54–55) are standard statutory references ⚠ not re-extracted this pass). MiFID II also brought client categorisation (retail / professional / eligible counterparty), product governance (target market and negative target market), and inducement restrictions — the regulatory machinery behind §4.

**Hong Kong — the SFC.** The SFC's conduct requirements impose suitability obligations on licensed intermediaries when making recommendations or soliciting: the recommendation must be reasonable having regard to the client's financial situation, investment experience and investment objectives (the SFC Code of Conduct suitability requirements) ⚠ (regime well documented, but not re-extracted at sfc.hk this pass — flagged in §12.4).

**Singapore — the FAA and SFA.** The **Financial Advisers Act 2001** is the statute governing financial-advisory services (including advising on investment products); its requirements sit alongside the SFA's investor-classification regime (§2.3). The MAS overlay — including the Guidelines on Fair Dealing (the "FAIR" outcomes: fair dealing, suitability, transparency) and the advisory-conduct expectations — is covered in the [MAS guide](mas_regulations_guidelines_guide.md) §2.6 (the FAA) and §4.6 (FEAT) and is **cross-referenced, not re-derived** ✅ (the statute year and the regime map are sibling-verified).

**The common core:** suitability = know your client (knowledge/experience, financial situation, objectives/risk tolerance), match the product or mandate to that profile, and **document the match**. The differences that matter to an architect: the EU's rules are the most codified (delegated regulation detail), Hong Kong's are conduct-based, and Singapore's are statute-plus-guideline — but the data model is the same client-risk-profile object feeding every sale, every mandate and every product-governance check.

### 9.4 MAS Expectations

The full MAS overlay — the statutes (Banking Act, SFA, FAA, FSMA 2022), the notices (including Notice 626 for AML/CFT and the banking 600-series), the guidelines (TRMG, BCM, Outsourcing, IAC, FEAT), and the industry expectations — is the subject of the [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) guide and is **cross-referenced, not re-derived** here. The private-banking-relevant highlights, all sibling-verified ✅:

- **Notice 626** — the bank AML/CFT rulebook (§9.1).
- **The FAA regime and fair dealing** — suitability and disclosure for advisory business (§9.3).
- **The CMS licence under the SFA** — dealing in capital-markets products, fund management and advising on corporate finance are licensable activities; a private bank's Singapore booking entity holds the relevant licences (cross-ref [Banks in Singapore](banks_in_singapore_guide.md) §5 and the [MAS guide](mas_regulations_guidelines_guide.md) §2.3).
- **The IAC guidelines** (Individual Accountability and Conduct, 2020) — senior-manager accountability, which in private banking maps to named RMs and named compliance officers ✅ (cross-ref [MAS guide](mas_regulations_guidelines_guide.md) §4.5).
- **The TRMG and BCM guidelines** — technology risk and business continuity for the platform estate (cross-ref the technology companions).

### 9.5 Cross-Border and Booking Considerations

The "booking centre vs advice centre" split (§5.3) is where regulation and operating model meet. The design rules an architect must encode:

1. **Licence follows activity** — the advice is regulated where the advice is given; the account is regulated where it is booked. A Singapore-booked relationship advised from Singapore is clean; a book advised from an unlicensed location is not ⚠ (regime principle; the per-jurisdiction detail is cross-referenced).
2. **Client mobility vs booking stability** — a client who relocates (the classic Asia story: Hong Kong to Singapore flows, cross-ref [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §8.4) triggers re-booking decisions: which entity holds the account, which tax-reporting regime applies, and whether the existing mandates remain executable.
3. **Tax-transparency portability** — FATCA/CRS reporting follows the tax residence of the account holder, not the booking centre; the booking entity reports through its own jurisdiction's exchange (cross-ref §9.2).
4. **The MAS lens** — MAS regulates the Singapore booking entity's conduct wherever the client sits; the extra-territorial reach of MAS expectations (and the 2023+ scrutiny of family-office-linked flows, cross-ref [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §7.3) is why Singapore-booked private banks run their compliance estate from Singapore.

---

## 10. The Singapore Angle

### 10.1 The Hub in Numbers

The verified numbers for the Singapore wealth complex:

- **Asset management industry AUM: S$6.1 trillion at end-2024, up 12% year-on-year** ✅ (MAS Singapore Asset Management Survey 2024, released 16 July 2025 — as reported by MAS and the financial press including finews.asia; the survey covers the whole AM industry, of which private banking is the distribution arm, not the AUM base). The **2025 survey reports S$6.7 trillion, +10%** ✅ (MAS announcement, 2026 — via MAS's own channels; treat the +10% as MAS-reported).
- **The S$4 trillion milestone:** Singapore's AM industry crossed S$4 trillion in AUM in the early 2020s (the 2019–2021 survey years) ⚠ — the exact crossing year was not re-verified this pass; the verified figures above (S$6.1tn, S$6.7tn) show the milestone is long since surpassed. This is flagged in §12.4.
- **Family offices: more than 2,000 single family offices as at end-2024, up ~43% year-on-year** ✅ (Reuters, cross-ref [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §7.3) — the demand engine of the private-banking hub.
- **The wealth funnel:** the sibling [Wealth Management](wealth_management_guide.md) §7 carries the Singapore wealth-hub context (players, MAS regulation, SGX/VCC) and is cross-referenced rather than duplicated.

### 10.2 The MAS-Regulated Bank Landscape

The bank landscape is verified in [Banks in Singapore](banks_in_singapore_guide.md) and summarised here: roughly **160 commercial banks and ~45 merchant banks** (⚠ approximate MAS statistics) in a city-state of ~6 million people; **196 MAS-licensed financial institutions across seven licence categories** (2026) ✅ (MAS Financial Institutions Directory, via the sibling); the **big three** — DBS, OCBC and UOB — dominate domestic banking, and each runs a private-banking franchise (DBS Private Bank and Treasures Private Client; **Bank of Singapore** (OCBC's private-bank arm); UOB Private Bank), alongside the global private banks (UBS, HSBC, Citi, Julius Baer, and the rest) ⚠ (the per-bank wealth-franchise list is from the sibling's flagged landscape, cross-ref [Banks in Singapore](banks_in_singapore_guide.md) §7 and [Wealth Management](wealth_management_guide.md) §7.3). The **booking-centre architecture** (§5) is what the wholesale-bank licence tier exists to serve: a foreign bank's Singapore branch books Asian private-banking business under a MAS licence (cross-ref [Banks in Singapore](banks_in_singapore_guide.md) §3.1).

### 10.3 The Private-Banking AUM Estimates (⚠)

Singapore-specific private-banking AUM is **not published by MAS** and every figure in circulation is a press or league-table estimate ⚠:

- **Asia-wide league tables:** finews.asia's 2024 Private Banking AUM League Table put **Asia's top 10 private banks at a record >US$2 trillion of AUM at end-2024, +13% year-on-year** ⚠ (finews.asia league table — press methodology); Asian Private Banker frames Asia's private-banking AUM as having **crossed US$3 trillion** ⚠ (league-table methodology, title of the 2024 APB insights page).
- **The Singapore slice:** estimates of Singapore-booked private-banking AUM cluster around the US$1 trillion order of magnitude ⚠ (press estimates vary; no authoritative public count — flagged in §12.4).
- **The structural point that IS verified:** the MAS survey shows the industry's AUM is substantially sourced from outside Singapore (~39% from Asia-Pacific outside Singapore at last sibling-verified reading ✅, cross-ref [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §8.3) — Singapore is a booking and management hub for regional wealth, which is exactly the private-banking model of §5.

### 10.4 Family-Office Demand

Family offices are the fastest-growing client segment of the Singapore private-banking market, and the regime is verified in the siblings and **cross-referenced, not re-derived**:

- **Scale:** 2,000+ SFOs at end-2024, +43% YoY ✅ (Reuters, cross-ref [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §7.3).
- **The SFO licensing exemption:** MAS formalised a **class exemption** for single family offices managing only the family's own monies — finalised in MAS's response of **6 November 2024** after the 31 July–30 September 2023 consultation — with the family definition, the 10% key-employee allowance, the **banking-anchor requirement (the SFO must bank with an MAS-regulated bank)**, and the 14-day notification rule ✅ (Allen & Gledhill / CNP Law via the hedge-funds guide §7.1; the operative implementation date of the exemption ⚠ was not verified).
- **The tax-incentive route:** sections 13O/13U of the Income Tax Act 1947 (min S$5m / S$50m AUM, ≥2 / ≥3 investment professionals, tiered local business spending), with revised criteria effective **1 January 2025** and the schemes extended to **31 December 2029** ✅ (cross-ref [Private Equity](private_equity_guide.md) §8.2 and [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §7.2).
- **The trust layer:** the trustee industry that holds family-office structures operates under the **Trust Companies Act 2005** with MAS licensing since 1 February 2006 ✅ (cross-ref [Singapore Trust Companies](singapore_trust_companies_guide.md) §2).
- **The 2024 tightening:** the 2023 money-laundering shock (the August 2023 arrests, press-reported seized assets of roughly S$2.8–3 billion ⚠ press estimates) put family-office-linked flows under enhanced scrutiny, and MAS's 2024 responses (the SFO exemption response and the revised 13O/13U criteria) are the codified result ✅-⚠ (timeline verified in the hedge-funds guide §7.3; cross-ref, do not re-derive). The 2024 MAS measures touching **trust companies** serving family offices ⚠ were not re-verified at the primary source this pass (see §12.4).

### 10.5 The MAS Wealth-Management Agenda

The MAS wealth-management agenda, as verified through the siblings: the **SFO framework** (§10.4), the **family-office tax-incentive stewardship** (13O/13U, extended to 2029 ✅), the **AML/CFT overlay** (Notice 626 and the designated-lists regime ✅), the **fair-dealing and conduct expectations** (FAIR/FEAT ✅, cross-ref [MAS guide](mas_regulations_guidelines_guide.md) §4.6), and the **competitiveness response** to Hong Kong — the 19 August 2026 package (performance-profit tax exemption for fund managers and investment professionals, wider ONE Pass visa access, a hedge-fund anchoring programme) ✅ (cross-ref [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §8). For a private bank, the agenda reads as one message: Singapore wants the *regulated* management of regional wealth — booking, advice, custody and structures under MAS supervision — and the family-office banking-anchor requirement makes the MAS-regulated private bank the compulsory gateway (✅, cross-ref §10.4).
---

## 11. The Cymbal Bank Worked Example — A Private-Client Onboarding Journey

This section follows the repository's worked-example conventions (extending the format established in [Market Making in Singapore](market_making_singapore_guide.md) §10 and [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §12): a named client type, a step-by-step journey, worked numbers in fenced code blocks, and the investops/margin mechanics cross-referenced rather than re-derived. The only bank persona is Cymbal Bank.

### 11.1 The Scenario

**The Client:** Mr. A. Rahman, 58, a Singapore resident and the founder of a regional logistics group he sold in 2023 for cash and equity. He and his wife have two adult children; the family runs a **single family office (SFO)** — the Rahman Family Office — which manages the family's investments, and a **family trust** (Rahman Family Trust) settled for succession planning, administered by a MAS-licensed trust company.

**The ask:** move the family's core banking and investments to Cymbal Bank — a MAS-licensed bank operating in Singapore (the repository's persona; see [MAS guide](mas_regulations_guidelines_guide.md) §1 for the persona conventions) — including a discretionary mandate, a Lombard facility, and the SFO's banking account. Target relationship: **US$40 million** of investable assets across the family, the SFO and the trust.

### 11.2 Segmentation and Onboarding

**Step 1 — segmentation.** Cymbal Bank's onboarding engine classifies the relationship against the verified conventions of Section 2:

| Dimension | Value | Classification |
| --- | --- | --- |
| Individual investable assets (Mr. Rahman) | ~US$14m | Mid-tier millionaire (US$5–30m) per the Capgemini bands ✅ (Capgemini WWR 2026) |
| Household + SFO + trust relationship | US$40m total | **UHNW band (≥ US$30m)** per the Capgemini convention ✅ (Capgemini WWR 2026); Cymbal's internal "Family Office" tier ⚠ (bank-policy convention) |
| SFA investor class (Singapore) | Net financial assets > S$1m; net personal assets > S$2m | **Accredited investor** — full non-retail shelf available ✅ (DBS FAQ on the SFA AI regime; §2.3) |

**Step 2 — the onboarding data model.** The account-opening pack collects, per the repository's KYC conventions (§9.1): identity and nationality (Mr. and Mrs. Rahman, the children), the **source of wealth** (the 2023 sale — sale agreement, tax records), the **source of funds** for the initial transfer, and the **structure register**: the SFO (which, under the 6 November 2024 MAS SFO exemption, must bank with an MAS-regulated bank ✅ — Cymbal Bank is that bank), the trust (settlor, trustee, beneficiaries, protector), and the UBO chain through both ✅-⚠ (look-through per §9.1; the structure documents are verified by the compliance team).

### 11.3 EDD and the Compliance Overlay

Mr. Rahman is **not a PEP**, but the relationship is **EDD-flagged** for three reasons ⚠ (Cymbal's risk-rating policy, following the standard EDD triggers of §9.1): (1) **structure complexity** — the SFO + trust + cross-border components require full beneficial-ownership look-through; (2) **cross-border flows** — the family holds a European property and a Cayman-domiciled fund interest (cross-ref [Cayman/BVI Master-Feeder](cayman_bvi_master_feeder_guide.md) for the vehicle mechanics); (3) **jurisdiction touchpoints** — a legacy account in a higher-risk jurisdiction that must be closed as part of the move.

The EDD workstream, all cross-referenced to the screening estate rather than re-derived:

- **Onboarding screening** — Mr. and Mrs. Rahman, the children, the SFO, the trust, the trustee and the Cayman fund are screened against the sanctions lists (OFAC SDN, UN Consolidated List, EU lists, MAS designated lists) and PEP/adverse-media databases at account open, with exact-vs-fuzzy matching and list-change rescreening ✅ (fircosoft_guide.md §3, §5; §9.1).
- **Beneficial-ownership attestation** — the trust deed, the SFO's family-ownership confirmation, and the Cayman fund's register (25% ownership/control threshold ✅, cross-ref [Cayman/BVI Master-Feeder](cayman_bvi_master_feeder_guide.md) §5) are reviewed; the UBOs are recorded as Mr. Rahman (and, through the trust, the beneficiaries).
- **Ongoing monitoring** — the relationship is placed on the EDD refresh cycle (more frequent KYC refresh and stricter screening thresholds ✅, cross-ref §9.1), and every payment in the flow of funds is transaction-screened ✅ (fircosoft_guide.md §5.2).
- **FATCA/CRS classification** — every account holder (individuals, SFO, trust) is classified and self-certified; the trust's controlling persons are reported; annual reports flow through the Singapore AEOI channel (cross-ref §9.2).

### 11.4 The Lombard Facility — A Worked LTV Computation

The Client draws a Lombard facility against a **US$12 million pledged portfolio** held at Cymbal Bank. The LTV ceilings follow the ⚠ directional conventions of §7.2 — every number below is Cymbal's credit policy for this client, not a verified industry standard:

```text
PLEDGED PORTFOLIO (initial)                Value      LTV cap (Cymbal policy, ⚠)   Loan capacity
  Cash / money-market funds               US$ 2.0m         100%                        US$ 2.00m
  Government bonds (AAA)                  US$ 2.0m          90%                        US$ 1.80m
  Blue-chip global equities (index)       US$ 5.0m          70%                        US$ 3.50m
  Single-name equity (legacy holding)     US$ 2.5m          50%                        US$ 1.25m
  Structured note (autocallable)          US$ 0.5m           0% (unmargined)           US$ 0.00m
  TOTAL                                   US$12.0m                                   US$ 8.55m

FACILITY:            cap US$ 6.0m (credit discretion below the US$8.55m collateral capacity)
DRAWDOWN:            US$ 6.0m (funds a private-equity capital-call programme — cross-ref §6.5)
UTILISATION:         6.0 / 8.55 = 70.2% of collateral capacity   → within limits
COVENANT TRIGGERS:   utilisation > 90% of capacity = margin call;  > 100% = sell-out right
                     (two-threshold design, ⚠ convention per §7.3)
```

**The stress case (-30% market shock on the pledged portfolio):**

```text
PORTFOLIO AFTER -30% SHOCK               Value      LTV cap   Loan capacity
  Cash / money-market funds             US$ 1.40m    100%      US$ 1.400m
  Government bonds                      US$ 1.40m     90%      US$ 1.260m
  Blue-chip equities                    US$ 3.50m     70%      US$ 2.450m
  Single-name equity                    US$ 1.75m     50%      US$ 0.875m
  Structured note                       US$ 0.35m      0%      US$ 0.000m
  TOTAL                                 US$ 8.40m              US$ 5.985m

UTILISATION:  6.0 / 5.985 = 100.3%  →  breaches the 100% sell-out level
MARGIN CALL:  Cymbal issues a same-day call; to restore the 90% buffer the client must
              post ~US$ 0.7m of cash (6.0 / 0.90 = 6.667m capacity needed; deficit
              6.667 - 5.985 = 0.682m), or repay part of the line, or pledge more
              eligible collateral. The bank holds the contractual right to sell
              pledged assets if the call is not met (cross-ref §7.3 and the
              liquidation-rights convention in [Market Making in Singapore]
              (market_making_singapore_guide.md) §10.3).
```

The mechanics behind the computation — daily collateral mark-to-market (the valuation hierarchy of [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §6), the margin-call loop, and the stress-testing discipline (cross-ref [Market Making in Singapore](market_making_singapore_guide.md) §10.3 and [Citadel LLC](citadel_llc_guide.md) §2.4 for the 2008-style stress convention) — are cross-referenced, not re-derived. The lesson the example encodes: **the LTV cap is per-asset, the margin call is portfolio-level, and the haircut grid — not the client's optimism — sets the leverage.**

### 11.5 The DPM Mandate

The Client signs a **discretionary mandate** with an **Investment Policy Statement**:

| IPS element | The Rahman mandate |
| --- | --- |
| Objective | Growth with capital preservation; fund the family's multi-generational spending and the PE capital-call programme |
| Risk tolerance | Moderate-to-growth (client risk profile from the suitability assessment, §9.3) |
| Benchmark | 60% MSCI ACWI / 40% Bloomberg Global Aggregate (net, USD) ⚠ (benchmark convention) |
| Asset-class ranges | Global equities 40–70%; fixed income 20–50%; alternatives 0–15%; cash 0–10% |
| Exclusions | No crypto assets; no single issuer > 5% of the portfolio; no tobacco/alcohol issuers; no short selling |
| Fee | 0.75% p.a. of AUM (⚠ indicative private-bank DPM fee band, cross-ref §6.1) |
| Rebalancing | Threshold bands: ±5 percentage points absolute, or ±20% relative, whichever triggers first ⚠ (industry convention) |

The mandate's operating lifecycle is the investops lifecycle — **portfolio construction → order → execution → allocation → confirmation → settlement → custody → corporate actions → reconciliation → valuation → performance and reporting** — fully derived in [Investment Portfolio Operations](investment_portfolio_operations_guide.md) and cross-referenced, not re-derived here. The private-banking specifics Cymbal adds: the mandate is **monitored against the client's risk profile continuously** (suitability-at-inception plus mandate-vs-profile monitoring, §9.3), the alternatives sleeve (up to 15%) is drawn through the funds cluster's structures (cross-ref §6.5), and the Lombard line of §11.4 is integrated — the PE capital calls are funded from the line, not by forced sales, which is the point of the whole structure (§7.4).

### 11.6 Periodic Reporting and the Operating Rhythm

The relationship runs on a fixed reporting and review cadence:

| Rhythm | Deliverable | Cross-ref |
| --- | --- | --- |
| **Daily** | Lombard collateral mark-to-market and utilisation vs the 90%/100% triggers (§11.4) | §7.3; [Market Making in Singapore](market_making_singapore_guide.md) §10.3 |
| **Monthly** | Portfolio valuation statement (positions, cash, income, fees) | [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §6 |
| **Quarterly** | Performance report vs benchmark with attribution (Brinson framework — cross-ref investops §7.2); DPM mandate review; suitability refresh for the advisory sleeve | §9.3; [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §7 |
| **Annually** | Full suitability review (FAA/MAS conduct expectations ✅, cross-ref §9.3–9.4); KYC refresh per the EDD cycle (§11.3); FATCA/CRS self-certification refresh (§9.2); facility review (Lombard limit, pricing) | §9 |
| **Event-driven** | List-change rescreening (fircosoft_guide.md §5.1); material-event KYC refresh (new beneficiary, new jurisdiction, new structure) | §9.1; [FircoSoft](fircosoft_guide.md) §5 |

The operating rhythm is deliberately boring — the value of private banking is that nothing surprising happens to the client's money, and everything surprising is caught by a control loop. That is the operational translation of the quiet wealth this guide describes.
---

## 12. The Claims Audit — Verified, Flagged, Rejected

### 12.1 The Verified Claims (✅)

| Claim | Source |
| --- | --- |
| HNWI = investable assets ≥ US$1m, excluding primary residence, collectibles, consumables, consumer durables; bands: Ultra-HNWI ≥ US$30m, Mid-Tier US$5–30m, Millionaires Next Door US$1–5m | Capgemini World Wealth Report 2026 press release (capgemini.com, 4 Jun 2026) |
| Global HNWI wealth US$98.3tn in 2025 (+8.7%); 25.3m HNWIs (+~2m); ~250k UHNWIs (+9.4%); top 1% = 34.8% of HNWI wealth | Capgemini WWR 2026 press release |
| 88% of HNWIs use multiple firms for better alternatives access; exclusive relationships 39% (2019) → 19% (2025); 97% of firms segment by AUM; 60% lack a unified client view; 41% of advisor time on operational tasks; 76% of advisors want AI automation; 68% of HNWIs intend to increase PE exposure; US$1.5tn flowed to non-traditional firms 2022–2025 | Capgemini WWR 2026 press release |
| Avaloq founded 1985 in Zürich as BZ Informatik AG (IT subsidiary of BZ Bank; founder Francisco Fernandez); renamed Avaloq 1996; split from BZ Bank 2001 | Avaloq article (citing BBC News, American Banker, Avaloq history); cross-checked avaloq.com |
| NEC announced the Avaloq acquisition on 5 Oct 2020 (price ~US$2.2bn, FT); closing announced 23 Dec 2020 | NEC press releases (5 Oct 2020; 23 Dec 2020); avaloq.com news; Warburg Pincus announcement; FT via Avaloq article |
| Avaloq today: 170+ clients across 35 countries; US$5.1tn AUM on Avaloq software (⚠ vendor figure); client logos incl. HSBC, LGT, Coutts, Deutsche Bank, RBC, DBS, Barclays, Maybank, Vontobel, Nomura, Raiffeisen; BPO centres incl. Singapore since 2015 | avaloq.com (homepage); Avaloq article (Singapore BPO) |
| Temenos founded November 1993 in Geneva by George Koukis and Kim Goodall (GLOBUS heritage); listed on SIX 2001; T24 launched 30 Sep 2003; Odyssey Financial Technologies (Luxembourg) acquired 2010; Multifonds acquired 2015, sold Feb 2025 to Montagu (~US$400m); revenue US$1.04bn 2024; Hindenburg report Feb 2024 and the independent-review response | Temenos AG article (citing Temenos Annual Report 2024, press); temenos.com About ("since 1993") |
| Finacle launched 1999 by Infosys; consolidated banking products under Finacle 2002; wealth-management capability added 2008; part of EdgeVerve Systems (Infosys subsidiary) since 2015; 6,000 employees 2015 | Finacle article (citing Moneycontrol, Business Standard, The Economic Times) |
| Objectway founded 1990 in Italy; founder/CEO Luigi Marciano; 200+ clients in 16+ countries; >€1tn AUM on platform; 800+ employees; front-to-back wealth/asset/banking platform; 2026: FNZ Swiss private-banking tech acquisition; SLIB (capital markets) negotiations | objectway.com (history page, homepage, news) |
| Profile Software founded 1990, Athens; 50+ countries; products Axia/IMSplus/FMS.next/RiskAvert (⚠ list via PitchBook); offices incl. Geneva, Dubai, London, Singapore, Athens, Nicosia (⚠ secondary) | profilesw.com; PitchBook; internationalbanker.com |
| Singapore AM industry AUM S$6.1tn at end-2024 (+12%) — MAS Singapore Asset Management Survey 2024 released 16 Jul 2025; Survey 2025: S$6.7tn (+10%) | MAS (survey release; 2025 survey announcement); finews.asia |
| SFA accredited investor: net personal assets > S$2m (primary-residence equity capped at S$1m) OR net financial assets > S$1m OR income ≥ S$300k in preceding 12 months | DBS accredited-investor regime FAQ (SFA-based) |
| MiFID II (Directive 2014/65/EU + Reg 600/2014) applied from 3 Jan 2018; suitability under Article 25(2) for advice/portfolio management | MiFID II article (EUR-Lex cross-ref); ⚠ delegated-rule detail (Reg 2017/565 Arts 54–55) not re-extracted |
| Lombard lending = short-term secured lending against securities/life policies; no forced sale → no capital-gains trigger; lower rates; fast access; used by UBS, Barclays, Julius Baer, JPMorgan, Rothschild | Lombard-credit article (citing PostFinance, KPMG, Rothschild & Co) |
| 2,000+ SFOs in Singapore at end-2024, ~+43% YoY; MAS SFO exemption finalised 6 Nov 2024; 13O/13U revised criteria effective 1 Jan 2025, schemes extended to 31 Dec 2029 | Reuters via [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §7 (Allen & Gledhill, CNP Law, MAS FDD Cir 10/2024) |
| Notice 626 = the Singapore bank AML/CFT rulebook (CDD, beneficial ownership, STR, sanctions screening); MAS designated lists | [MAS guide](mas_regulations_guidelines_guide.md) via [FircoSoft](fircosoft_guide.md) §4.5 |
| Screening discipline (CDD→screening→transaction monitoring→sanctions; PEP/EDD; list-change rescreening; fuzzy matching) | [FircoSoft](fircosoft_guide.md) §3/§5; [RegTech](regtech_guide.md) §4 |
| FATCA = IRC Chapter 4 §§1471–1474, 30% withholding on non-compliant FFIs; CRS parallel; Cayman/BVI IGAs | [Cayman/BVI Master-Feeder](cayman_bvi_master_feeder_guide.md) §3.4 (law.cornell.edu) |
| FAA 2001 statute year and the MAS guideline map (FAIR/FEAT/IAC/TRMG/BCM); Cymbal persona conventions | [MAS guide](mas_regulations_guidelines_guide.md) §2.6, §4–§5 |
| ~160 commercial banks + ~45 merchant banks (⚠ approx.); 196 MAS-licensed FIs across seven licence categories (2026); big three DBS/OCBC/UOB | [Banks in Singapore](banks_in_singapore_guide.md) §1 (MAS FI Directory) |
| Endowus survey findings (37% SG / 57% HK prefer digital platforms for private markets; 1-in-5 SG prefer an RM) | finews.asia (29 May 2024) — ⚠ platform-commissioned survey |
| Asia's top 10 private banks > US$2tn AUM at end-2024 (+13%); APB frames Asia PB AUM crossing US$3tn | finews.asia league table 2024; Asian Private Banker — ⚠ press methodology |

### 12.2 The Flagged Claims (⚠)

| Claim | Why flagged |
| --- | --- |
| RM-to-client ratio (~1:40–1:100; UHNW books smaller) | Industry convention only — no bank publishes coverage ratios; no primary source verified this pass |
| Private-bank client-entry thresholds (US$1m–US$5m) and bank tier names | Bank-policy variation; no public standard |
| Knight Frank's UHNW definition and other research-house cut-offs | Not re-verified this pass |
| MAS has no official "private banking" threshold | Negative claim — cannot be fully proven from the sources captured this pass |
| Lombard LTV/haircut grid (cash 90–100% … single-name 40–60%) | Directional market convention; each bank's credit policy differs; no bank grid published |
| Margin-call cure periods and two-threshold (call/sell-out) design | Standard practice but bank-specific; not verified against any published policy this pass |
| S$4 trillion AM-industry milestone year (2019–2021) | Not re-verified this pass; the S$6.1tn/S$6.7tn figures are the verified ones |
| Singapore-booked private-banking AUM (~US$1tn order of magnitude) | Press estimates vary; no authoritative public count |
| Avaloq "US$5.1tn AUM on software"; Temenos "3,000+ FIs / 41 of top 50 banks"; Objectway ">€1tn AUM"; Finacle "100+ countries" | Vendor-reported figures |
| Temenos WealthSuite current branding and wealth-module depth | Product branding not re-verified this pass (Odyssey 2010 acquisition is verified) |
| HK SFC suitability requirements | Not re-extracted at sfc.hk this pass (well-documented regime) |
| 2024 MAS measures touching trust companies serving family offices | Not re-verified at the primary source this pass (the SFO/13O/13U timeline is sibling-verified) |
| Endowus survey as market evidence | Platform-commissioned; finews.asia itself flags the bias |
| Asia PB AUM league tables (finews/APB) | Press methodologies differ; not audited figures |
| Structured-product margins (1–3%) and DPM fee bands (0.4–1.5%) | Indicative benchmarks from the sibling's flagged tables |

### 12.3 The Rejected Claims (❌)

| Claim | Verdict | Basis |
| --- | --- | --- |
| "The NEC–Avaloq acquisition was completed in 2021" | ❌ — the closing was announced 23 December 2020 | NEC press release, 23 Dec 2020 ("Avaloq's acquisition by NEC successfully completed"); avaloq.com news |
| "Avaloq was founded as Avaloq" | ❌ — founded 1985 as BZ Informatik AG; the Avaloq name was adopted in 1996 | Avaloq article (BBC/American Banker; Avaloq history page) |
| "Temenos was founded in 2001" | ❌ — founded November 1993; 2001 is the IPO/listing year | Temenos AG article; temenos.com ("since 1993") |

### 12.4 What Could Not Be Verified

This subsection collects every item this pass could not confirm against a primary or reliable secondary source, so the reader can distinguish verified fact from honest uncertainty:

1. **The RM-to-client ratio** — no authoritative figure exists; the ~1:40–1:100 convention is uncited industry lore. The Capgemini RM survey (1,317 RMs) is verified; the ratio is not.
2. **Singapore-booked private-banking AUM** — MAS does not publish it; press estimates vary around the US$1 trillion order of magnitude.
3. **The exact year Singapore's AM industry crossed S$4 trillion** — the verified figures are S$6.1tn (end-2024) and S$6.7tn (2025 survey); the milestone year is not.
4. **Vendor market shares in private-banking platforms** — no audited public ranking exists; §8.7's ordering is qualitative.
5. **Avaloq's full booking-centre deployment list** — the CH/SG/HK heritage is asserted from the vendor's client set and history; per-bank deployment detail is not public.
6. **Temenos WealthSuite's current branding and feature set** — the Odyssey (2010) origin is verified; the current product line was not re-verified.
7. **Objectway's and Profile Software's current client counts and Singapore office status** — vendor and secondary figures only.
8. **The HK SFC suitability rule text** — not re-extracted this pass.
9. **The 2024 MAS trust-company measures detail** — the SFO exemption and 13O/13U timeline are sibling-verified; the trust-company consultation specifics were not re-verified.
10. **Knight Frank's UHNW definition and other alternate thresholds** — flagged, not re-verified.
11. **Lombard LTV grids, cure periods and call/sell-out thresholds at any named bank** — directional conventions only; the worked example's numbers are Cymbal credit policy.
12. **Structured-product and insurance commission rates in Asia** — indicative benchmarks only (cross-ref the sibling's flagged tables).
13. **The exact SFO-exemption implementation date** — flagged in the hedge-funds sibling; not re-verified here.

---

## 13. Glossary

| Term | Definition |
| --- | --- |
| HNW / HNWI | High-net-worth individual — investable assets ≥ US$1m (Capgemini convention, excluding primary residence etc.) ✅ |
| UHNW / UHNWI | Ultra-high-net-worth individual — investable assets ≥ US$30m (Capgemini) ✅ |
| Mid-tier millionaire | US$5–30m band (Capgemini) ✅ |
| Relationship manager (RM) | The client's named banker; owner of the client relationship and revenue |
| Advisory continuum | Execution-only → advisory → discretionary (DPM), with rising suitability burden |
| DPM | Discretionary portfolio management — client delegates decisions within a mandate/IPS |
| IPS | Investment Policy Statement — the mandate's written investment rules |
| Lombard loan | Secured borrowing against pledged securities/life policies ✅ (Lombard-credit article) |
| LTV | Loan-to-value — loan ÷ collateral value; the haircut is 1 − LTV |
| Haircut | The discount applied to collateral value (a 70% LTV = 30% haircut) |
| Margin call | Demand to top up collateral/cash or repay when utilisation breaches the trigger |
| Booking centre | The jurisdiction/entity where the client account legally sits |
| Advice centre | Where the RM sits and advice is given (may differ from booking) |
| Open architecture | Distributing third-party products alongside (or instead of) in-house ones |
| Manufacturer vs distributor | The bank's dual role: making products (asset management) vs distributing them (private bank) |
| EDD | Enhanced due diligence — the higher KYC standard for PEPs, complex structures, high-risk clients |
| PEP | Politically exposed person — higher-risk client class |
| UBO | Ultimate beneficial owner — the natural person behind a structure |
| FATCA | US Foreign Account Tax Compliance Act (IRC Ch. 4 §§1471–1474); 30% withholding on non-compliant FFIs ✅ |
| CRS | OECD Common Reporting Standard — multilateral automatic exchange of account information |
| AEOI | Automatic exchange of information — the umbrella for FATCA/CRS |
| MiFID II | Directive 2014/65/EU, applied 3 Jan 2018 — the EU's investment-services rulebook, incl. suitability ✅ |
| FAA | Financial Advisers Act 2001 — the SG statute for financial-advisory services ✅ |
| SFA | Securities and Futures Act 2001 — the SG capital-markets statute (accredited-investor classes) ✅ |
| Notice 626 | MAS's bank AML/CFT notice (CDD, beneficial ownership, STR, screening) ✅ |
| SFO | Single family office — manages only the family's own money; MAS class exemption (6 Nov 2024) ✅ |
| 13O / 13U | Income Tax Act 1947 fund tax incentives (onshore S$5m / enhanced S$50m tiers) ✅ |
| TCA 2005 | Trust Companies Act 2005 — the SG trustee-licensing statute ✅ |
| Avaloq / Temenos / Finacle / Objectway / Profile | The verified platform vendors of Section 8 |
| Cymbal Bank | The repository's bank persona — the only bank persona in this guide |

---

## 14. Cross-References and Further Reading

**Repository guides (banking siblings — plain filenames):**
- [Wealth Management](wealth_management_guide.md) — the broad wealth-industry sibling: segments, advisory models, fee benchmarks, DPM, wealthtech, Singapore context (cross-ref §2–§6, §10; do not duplicate)
- [Investment Portfolio Operations](investment_portfolio_operations_guide.md) — the investops lifecycle behind DPM: order → execution → settlement → custody → corporate actions → reconciliation → NAV → performance (cross-ref §6.1, §11.5–11.6)
- [Market Making in Singapore](market_making_singapore_guide.md) — margin/collateral mechanics, the house-style conventions, and the claims-audit format this guide mirrors (cross-ref §7, §12)
- [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) — the family-office angle: SFO exemption, 13O/13U, the 2023–2025 tightening, the HK rivalry (cross-ref §10.4)
- [Private Equity](private_equity_guide.md) and [Private Equity in Singapore](private_equity_singapore_guide.md) — fund structures, the subscription-line model, the Singapore fund regime (cross-ref §6.5)
- [Citadel LLC](citadel_llc_guide.md) — the multi-strategy alternatives archetype and the stress-case conventions (cross-ref §6.5, §11.4)
- [Cayman/BVI Master-Feeder](cayman_bvi_master_feeder_guide.md) — offshore structures and the FATCA/CRS/PFIC/UBTI tax content (cross-ref §6.7, §9.2, §11.3)
- [FircoSoft](fircosoft_guide.md) — the screening estate: CDD/EDD, PEPs, sanctions lists, list-change rescreening (cross-ref §9.1, §11.3)
- [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) — the Singapore regulatory overlay, Notice 626, the FAA/SFA map, and the Cymbal Bank persona conventions (cross-ref §9.4)
- [Banks in Singapore](banks_in_singapore_guide.md) — the licence tiers (full/wholesale/offshore/digital) and the bank landscape (cross-ref §5.2, §10.2)
- [Singapore Trust Companies](singapore_trust_companies_guide.md) — the TCA 2005 trustee layer and the trust-plus-fund architecture (cross-ref §6.7, §10.4)
- [Singapore Private Markets](singapore_private_markets_guide.md) — the VCC and 13O/13U fund regime (cross-ref §10.4)
- [Online Investment Trading Platforms](online_investment_trading_platforms_guide.md) — the brokerage layer (cross-ref §6.3)
- [RegTech](regtech_guide.md) — the AML-KYC stack (CDD → screening → transaction monitoring → sanctions) (cross-ref §9.1)
- [Temenos Data Model](temenos_data_model_guide.md) — the Temenos core-banking data model (cross-ref §8.2)
- [Treasury & ALM](treasury_alm_guide.md) — the bank-side funding and ALM mechanics behind the deposit/Lombard book (cross-ref §6.6)

**Repository guides (technology — prefix `../technology/`):**
- [Zero Downtime System Design](../technology/zero_downtime_system_design_guide.md) — always-on banking estates (the private-bank platform availability requirement)
- [SecOps Guide](../technology/secops_guide.md) and [Cybersecurity Guide](../technology/cybersecurity_guide.md) — client-data protection and the TRMG overlay
- [Architecture Decision Record](../technology/architecture_decision_record_guide.md) — the ADR discipline behind platform selection (§8)
- [FinOps Guide](../technology/finops_guide.md) — the cost economics of running a wealth platform at scale

**Primary and press sources used this pass:**
- capgemini.com — World Wealth Report 2026 press release (4 June 2026; the HNWI definition, wealth bands, and all WWR figures cited)
- nec.com — "NEC acquires a leading Swiss financial software company, Avaloq" (5 Oct 2020) and "Avaloq's acquisition by NEC successfully completed" (23 Dec 2020); avaloq.com news pages; warburgpincus.com (5 Oct 2020)
- avaloq.com — homepage (client logos, 170+ clients, US$5.1tn figure)
- temenos.com — About (Vision & Strategy: "since 1993"); Wikipedia Temenos AG (founding, T24, Odyssey 2010, Multifonds 2015/2025, 2024 revenue, Hindenburg)
- Wikipedia: Avaloq; Finacle; Temenos AG; MiFID II; Lombard credit — used for verification with citation chains into primary press (BBC, American Banker, FT, Reuters, EUR-Lex, PostFinance, KPMG, Rothschild & Co)
- objectway.com — homepage, history page, 2026 FNZ and SLIB news
- profilesw.com — About; PitchBook (product list); internationalbanker.com (offices)
- mas.gov.sg and the financial press — Singapore Asset Management Survey 2024 (S$6.1tn, released 16 Jul 2025) and Survey 2025 (S$6.7tn) via MAS channels and finews.asia
- dbs.com.sg — accredited-investor regime FAQ (the SFA thresholds)
- finews.asia — 2024 Private Banking AUM League Table (Asia >US$2tn); Endowus survey coverage (29 May 2024)
- asianprivatebanker.com — Asia 2024 private-banking AUM insights page (US$3tn framing)
- Sibling repository guides — as cross-referenced above (MAS survey figures, SFO count, Notice 626, FATCA/CRS, screening discipline, margin mechanics, the Cymbal conventions)

---

## 15. Closing Summary

Private banking is the repository's wealth-cluster deep-dive made concrete: a relationship-led business that monetises the whole bank balance sheet for clients the industry defines with verified precision — investable assets of US$1 million or more, with the Ultra-HNW band at US$30 million and the mid-tier between, per the Capgemini World Wealth Report 2026 that this guide verified directly. The model rests on structures this guide verified one by one: the RM coverage model (whose ratio is honestly flagged as convention, not fact), the open-architecture vs in-house tension (with the market evidence — 88% of HNWIs multi-bank for alternatives, exclusive relationships halved to 19% — coming straight from the same report), the booking-centre architecture of Singapore, Hong Kong and Switzerland, a product shelf anchored by DPM and Lombard lending (the secured-lending mechanism verified at source, its LTV grid honestly flagged as convention), and a platform landscape — Avaloq (1985, Zürich; NEC's since 23 December 2020, not 2021), Temenos (November 1993, Geneva), Finacle (1999, EdgeVerve), Objectway (1990) and Profile Software (1990) — profiled only against primary sources. The Singapore hub carries the numbers that matter to this repository: S$6.1 trillion of asset-management AUM at end-2024 (S$6.7 trillion on the 2025 survey), more than 2,000 single family offices, the SFA accredited-investor thresholds as the statutory anchor, and a MAS overlay — Notice 626, the FAA, the SFO exemption — that is cross-referenced from the siblings rather than re-derived. The Cymbal Bank worked example binds it together: an UHNW family with an SFO and a trust, onboarded under EDD, financed with a Lombard line whose 70.2% utilisation becomes a 100.3% sell-out breach under a 30% shock, invested under a DPM mandate that runs on the investops lifecycle, and reported on a cadence that catches everything before the client notices. What could not be verified — the RM ratio, the Singapore-booked AUM, vendor market shares, the LTV grids of any named bank — is flagged ⚠ and listed in §12.4, not guessed. That is the integrity convention of this repository, and it is the discipline private banking itself sells: verified facts, honest uncertainty, and the quiet wealth.
