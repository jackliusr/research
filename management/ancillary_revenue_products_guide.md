# Ancillary Revenue Products — The Fare's Hidden Half

**The Airline and Travel Ancillary-Product Playbook — the Non-Ticket Revenue Categories (Baggage, Seats, Priority, Onboard Retail, Wi-Fi, Lounges, Hotels and Cars, Travel Insurance, Loyalty-Miles Sales), the Economics (Per-Passenger Benchmarks, Unbundling, the ULCC vs Legacy Models), the Merchandising (Fare Families, Offers, Point-of-Sale Upsell), the Distribution (Direct vs GDS vs OTA, EMD, NDC, ONE Order), the Loyalty Engine (Frequent-Flyer Programs, Miles Sales, Co-Branded Credit Cards), the Insurance and Payments Overlays, the Singapore Angle (SIA Group), and a Cymbal Bank Airline Co-Brand Card Worked Example — Verified Against Primary Sources**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Management / Industry Research — the ancillary-revenue product playbook of the airline and travel industry: what ancillary revenue is and how the industry classifies it (a la carte fees, commission-based products, frequent-flyer/loyalty revenue), the global and per-airline economics (the annual CarTrawler Yearbook of Ancillary Revenue by IdeaWorksCompany; the $109.5B (2019) → $102.8B (2022) → $117.9B (2023) → $148.4B (2024) global series; per-passenger benchmarks from Ryanair, Frontier, Spirit, AirAsia, Allegiant and the US legacies), fare unbundling (the ULCC à-la-carte model and the legacy basic-economy response), merchandising mechanics (fare families, offer construction, point-of-sale up-sell, offer management systems), distribution plumbing (direct vs GDS vs OTA, the EMD standard, IATA NDC and ONE Order), the loyalty engine and its banking face (frequent-flyer programs, miles sales to partners, co-branded credit cards, program valuations and securitization), travel insurance and payments overlays, the Singapore angle (SIA group: KrisFlyer, Kris+, Scoot, Amex/UOB co-brands), and a Cymbal Bank worked example: standing up an airline co-brand card program end to end.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** ideaworkscompany.com (the IdeaWorksCompany press releases — "Airline Ancillary Revenue Reaches Record $117.9 Billion Worldwide for 2023" (31 Oct 2023) and "Airline Ancillary Revenue Skyrockets to $148.4 Billion Worldwide for 2024" (29 Oct 2024); the 2024 and 2025 editions of the CarTrawler Yearbook of Ancillary Revenue by IdeaWorksCompany), iata.org (NDC Resolution 787, ONE Order Resolution 797, the EMD standard, the IATA annual reviews), investor.ryanair.com (FY24/FY25 annual reports), news.aa.com (Basic Economy launch press release), the airline and bank co-brand pages (americanexpress.com/sg, uob.com.sg, singaporeair.com), and the trade press (Simple Flying, PaxEx.Aero, PhocusWire, The Points Guy, Reuters). NOTE: this pass had **live web access** — the key figures and dates below were verified against the primary sources on 2026-08-29; anything that could not be re-verified in this pass is flagged ⚠ honestly. No dates, numbers or claims were invented.
> **Last Updated:** August 2026
> **Companion guides (sibling, same folder — the management cluster):** [E-Commerce Experience Guide](ecommerce_experience_guide.md) (the merchandising/checkout mechanics — offer construction, up-sell flows, payment checkout, reconciliation — cross-ref §3, §8, do NOT re-derive) · [Business Case Development](business_case_development_guide.md) (the business-case discipline behind any ancillary/co-brand investment — cross-ref §10)
> **Companion guides (banking/, prefix `../banking/`):** [Private Banking Guide](../banking/private_banking_guide.md) (premium-card co-brand conventions, the Cymbal Bank persona — cross-ref §5, §6, §10) · [Payment Rails Guide](../banking/payment_rails_guide.md) (card rails, interchange, clearing and settlement — cross-ref §8) · [FircoSoft Guide](../banking/fircosoft_guide.md) (sanctions/PEP screening in the card-program onboarding overlay — cross-ref §8, §10) · [Insurance Products, Processes and Compliance Guide](../banking/insurance_products_processes_compliance_guide.md) (the insurance product shelf and claims/regulatory process — cross-ref §7)
> **Companion guides (technology/, prefix `../technology/`):** [Enterprise Middleware Integration Platform Guide](../technology/enterprise_middleware_integration_platform_guide.md) (the integration backbone for offer/order APIs — cross-ref §4, §10) · [API Governance Guide](../technology/api_governance_guide.md) (the API lifecycle for NDC/offer endpoints — cross-ref §4) · [Legacy Integration Patterns Guide](../technology/legacy_integration_patterns_guide.md) (PNR/EMD estate modernization patterns — cross-ref §4) · [Health Insurance Guide](../technology/health_insurance_guide.md) (insurance product/claims modelling parallels — cross-ref §7) · [Personalization Engines Guide](../technology/personalization_engines_guide.md) (offer personalization — cross-ref §3)

---

**How to use this guide:** Section 1 is the concept — the definition, the industry taxonomy and the verified global numbers. Section 2 is the economics — per-passenger benchmarks, ancillary share of revenue, and the two unbundling models (ULCC à-la-carte and legacy basic economy). Section 3 is the merchandising — fare families, offer construction and point-of-sale up-sell; the checkout mechanics themselves live in the E-Commerce Experience sibling and are cross-referenced, not re-derived. Section 4 is the distribution — direct vs GDS vs OTA, the EMD standard, and IATA's NDC and ONE Order programs, with the technology siblings cross-referenced for the API/integration layer. Section 5 is the loyalty engine — frequent-flyer programs and the sale of miles to partners. Section 6 is the co-brand — the airline credit-card partnership, its economics and the securitization facts; the premium-card conventions are cross-referenced to the Private Banking sibling. Section 7 is the insurance overlay and Section 8 the payments overlay, both deliberately condensed with cross-references. Section 9 is the Singapore angle — the SIA group. Section 10 is the Cymbal Bank worked example — standing up an airline co-brand card program. Section 11 is the claims audit (✅/⚠/❌), Section 12 is "What Could Not Be Verified", Section 13 the glossary, Section 14 the sources, and Section 15 the closing summary. Cross-references follow the repository convention: sibling guides in `management/` are plain filenames; guides in `banking/` are prefixed `../banking/`, `technology/` guides `../technology/`, and `singapore/` guides `../singapore/`. **Integrity convention:** ✅ = verified this pass against a primary source; ⚠ = flagged/approximate/unverified; ⚠-knowledge = well-documented industry knowledge not re-verified this pass; ❌ = could not verify or conflicting sources.

---

**Table of Contents**

1. [The Concept — Ancillary Revenue in Airlines and Travel](#1-the-concept--ancillary-revenue-in-airlines-and-travel)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Taxonomy — How the Industry Classifies Non-Ticket Revenue](#12-the-taxonomy--how-the-industry-classifies-non-ticket-revenue)
   - 1.3 [The Global Numbers — the CarTrawler/IdeaWorksCompany Series (Verified)](#13-the-global-numbers--the-cartrawlerideaworkscompany-series-verified)
   - 1.4 [Why It Matters to a Bank](#14-why-it-matters-to-a-bank)
   - 1.5 [A Short History of Ancillary Revenue](#15-a-short-history-of-ancillary-revenue)
   - 1.6 [The Key Facts at a Glance](#16-the-key-facts-at-a-glance)
2. [The Economics — Per-Passenger Benchmarks and Fare Unbundling](#2-the-economics--per-passenger-benchmarks-and-fare-unbundling)
   - 2.1 [The Per-Passenger Metric](#21-the-per-passenger-metric)
   - 2.2 [Ancillary Share of Total Revenue](#22-ancillary-share-of-total-revenue)
   - 2.3 [The ULCC Unbundling Model — Ryanair, Spirit, Frontier, Allegiant](#23-the-ulcc-unbundling-model--ryanair-spirit-frontier-allegiant)
   - 2.4 [The Legacy-Carrier Response — Basic Economy and Branded Fares](#24-the-legacy-carrier-response--basic-economy-and-branded-fares)
   - 2.5 [The Alchemy — Fares Down, Fees Up](#25-the-alchemy--fares-down-fees-up)
   - 2.6 [The Model Comparison — ULCC vs Legacy](#26-the-model-comparison--ulcc-vs-legacy)
3. [The Merchandising — Fare Families, Offers and Point-of-Sale Upsell](#3-the-merchandising--fare-families-offers-and-point-of-sale-upsell)
   - 3.1 [Fare Families and Bundles — Basic Economy to Premium](#31-fare-families-and-bundles--basic-economy-to-premium)
   - 3.2 [Offer Construction — the IATA Offer](#32-offer-construction--the-iata-offer)
   - 3.3 [Up-Sell and Cross-Sell at the Point of Sale](#33-up-sell-and-cross-sell-at-the-point-of-sale)
   - 3.4 [Merchandising Engines — OMS, Dynamic Pricing, Personalization](#34-merchandising-engines--oms-dynamic-pricing-personalization)
   - 3.5 [Measuring the Merchandising — Instrumentation](#35-measuring-the-merchandising--instrumentation)
4. [The Distribution — Direct, GDS, OTA, EMD, NDC, ONE Order](#4-the-distribution--direct-gds-ota-emd-ndc-one-order)
   - 4.1 [The Channels — Direct vs GDS vs OTA](#41-the-channels--direct-vs-gds-vs-ota)
   - 4.2 [The EMD Standard](#42-the-emd-standard)
   - 4.3 [NDC — the Offer/Order API Standard (Verified)](#43-ndc--the-offerorder-api-standard-verified)
   - 4.4 [ONE Order — Replacing PNR + E-Ticket + EMD (Verified)](#44-one-order--replacing-pnr--e-ticket--emd-verified)
   - 4.5 [The Integration Layer — Cross-Referenced](#45-the-integration-layer--cross-referenced)
   - 4.6 [The Settlement Layer — BSP, ARC and Airline Clearing](#46-the-settlement-layer--bsp-arc-and-airline-clearing)
5. [The Loyalty — Frequent-Flyer Programs and the Sale of Miles](#5-the-loyalty--frequent-flyer-programs-and-the-sale-of-miles)
   - 5.1 [The Program Model — Miles as a Liability and an Asset](#51-the-program-model--miles-as-a-liability-and-an-asset)
   - 5.2 [Miles Sales to Partners — the Hidden Revenue Line](#52-miles-sales-to-partners--the-hidden-revenue-line)
   - 5.3 [Program Valuations — 2020 Collateral and 2026 Rankings (Verified)](#53-program-valuations--2020-collateral-and-2026-rankings-verified)
   - 5.4 [Cross-Reference — the Premium-Card Conventions](#54-cross-reference--the-premium-card-conventions)
   - 5.5 [The Program Economics — Accrual, Redemption, Breakage](#55-the-program-economics--accrual-redemption-breakage)
6. [The Co-Brand — Airline Credit-Card Partnerships](#6-the-co-brand--airline-credit-card-partnerships)
   - 6.1 [The Partnership Structure — Airline, Bank, Network](#61-the-partnership-structure--airline-bank-network)
   - 6.2 [The Economics — Interchange, Miles Cost, Breakage](#62-the-economics--interchange-miles-cost-breakage)
   - 6.3 [The Named Partnerships (Verified)](#63-the-named-partnerships-verified)
   - 6.4 [The Securitization Wave — Loyalty Programs as Collateral (Verified)](#64-the-securitization-wave--loyalty-programs-as-collateral-verified)
7. [The Insurance — Travel-Insurance Ancillaries](#7-the-insurance--travel-insurance-ancillaries)
   - 7.1 [The Products](#71-the-products)
   - 7.2 [Distribution at Checkout and Attach Rates](#72-distribution-at-checkout-and-attach-rates)
   - 7.3 [Claims Flow and the Cross-References](#73-claims-flow-and-the-cross-references)
8. [The Payments — Ancillary Payments, Chargebacks, Fraud](#8-the-payments--ancillary-payments-chargebacks-fraud)
   - 8.1 [How Ancillaries Get Paid For](#81-how-ancillaries-get-paid-for)
   - 8.2 [Chargebacks and Fraud — the Fee-Specific Risks](#82-chargebacks-and-fraud--the-fee-specific-risks)
   - 8.3 [Cross-References](#83-cross-references)
9. [The Singapore Angle — the SIA Group](#9-the-singapore-angle--the-sia-group)
   - 9.1 [The Group — SIA, Scoot and the Network](#91-the-group--sia-scoot-and-the-network)
   - 9.2 [KrisFlyer and Kris+](#92-krisflyer-and-kris)
   - 9.3 [The Co-Brand Cards — Amex and UOB (Verified)](#93-the-co-brand-cards--amex-and-uob-verified)
   - 9.4 [SIA Ancillary Practices — Seats, Bags, Upgrades](#94-sia-ancillary-practices--seats-bags-upgrades)
   - 9.5 [The Singapore Co-Brand Competitive Set](#95-the-singapore-co-brand-competitive-set)
10. [The Cymbal Bank Worked Example — An Airline Co-Brand Card Program](#10-the-cymbal-bank-worked-example--an-airline-co-brand-card-program)
    - 10.1 [The Scenario](#101-the-scenario)
    - 10.2 [The Card Economics — a Worked P&L](#102-the-card-economics--a-worked-pl)
    - 10.3 [Miles Accrual and Settlement Mechanics](#103-miles-accrual-and-settlement-mechanics)
    - 10.4 [The Partnership Agreement — Liability and Funding](#104-the-partnership-agreement--liability-and-funding)
    - 10.5 [The Lessons](#105-the-lessons)
    - 10.6 [The Systems Map — Cross-Referenced](#106-the-systems-map--cross-referenced)
11. [The Claims Audit](#11-the-claims-audit)
12. [What Could Not Be Verified](#12-what-could-not-be-verified)
13. [The Glossary](#13-the-glossary)
14. [The Sources — Primary References](#14-the-sources--primary-references)
15. [The Closing Summary — The Fare's Hidden Half](#15-the-closing-summary--the-fares-hidden-half)

---

## 1. The Concept — Ancillary Revenue in Airlines and Travel

### 1.1 The Short Answer

Ancillary revenue is the airline-industry term for **all cashflow that is not the fare** — the money earned beyond transporting a passenger from A to B. The IdeaWorksCompany definition, used in the annual yearbook it co-publishes with CarTrawler, is deliberately broad: ancillary revenue is generated by "activities and services that yield cashflow beyond the transportation of customers from A to B," including commissions from hotel bookings, the sale of frequent-flyer miles to partners, co-branded credit-card programs, and a la carte fees paid directly by passengers (baggage, assigned seats, buy-on-board). ✅ (IdeaWorksCompany press release, 29 Oct 2024.)

The category has grown from a niche to the industry's second revenue engine: the CarTrawler Worldwide Estimate put global airline ancillary revenue at **$117.9 billion for 2023** and a projected **$148.4 billion for 2024**, against a global airline revenue base IATA put at roughly $996 billion for 2024 — meaning ancillaries contributed about **14.9% of global airline revenue** in 2024, with individual airlines ranging from 2% to 56.4% of their own revenue. ✅ (IdeaWorksCompany/CarTrawler press releases, 31 Oct 2023 and 29 Oct 2024; IATA Industry Statistics Fact Sheet, June 2024.)

### 1.2 The Taxonomy — How the Industry Classifies Non-Ticket Revenue

The industry-standard taxonomy (IdeaWorksCompany's) splits ancillary revenue into three broad buckets. ✅ where the bucket is named in the primary sources below; the finer-grained splits are ⚠-knowledge.

| Bucket | What it contains | Examples | Mark |
| --- | --- | --- | --- |
| **A la carte fees** | Optional extras passengers pay for directly, "the share directly paid by consumers" | Checked and cabin baggage, assigned seats, priority boarding, buy-on-board food and drink, onboard Wi-Fi, lounge day-passes | ✅ |
| **Commission-based products** | Third-party travel products sold through the airline | Car rental, hotels, airport transfers/ride-hailing, travel insurance | ✅ |
| **Frequent-flyer / loyalty revenue** | Money earned from the loyalty program itself | Sale of miles to partners (hotels, car rental, banks), co-branded credit-card remuneration, mileage breakage | ✅ |

The 29 Oct 2024 press release names all three buckets explicitly: revenue "generated by a la carte items (such as baggage and assigned seats) along with commissions generated by car and hotel travel sales, and co-branded credit card programs associated with frequent flyer programs," plus "commissions gained from hotel bookings, the sale of frequent flyer miles to partners." ✅ A finer-grained split sometimes seen in the industry literature — four buckets, separating "unbundling of formerly bundled services" (e.g., premium-economy seats sold as an upgrade of a standard fare) from pure a la carte — is ⚠-knowledge and not needed for this guide's purposes.

Two structural facts about the mix are worth stating early. First, **a la carte is the largest slice for most airlines**: "for most of the world's airlines, it's the a la carte portion that represents the largest slice of the ancillary revenue pie." ✅ Second, the **loyalty bucket is the largest for the US majors**: "US major airlines are also boosting ancillary revenue... accomplished through the seemingly perpetual cash generator of co-branded credit cards. Frequent flyer program revenue in this category exceeds $30 billion. On average, these carriers generate $35 per passenger from their loyalty program activities." ✅ Both quotes are from the 29 Oct 2024 IdeaWorksCompany press release.

### 1.3 The Global Numbers — the CarTrawler/IdeaWorksCompany Series (Verified)

The annual estimate is published jointly by **CarTrawler** (the Dublin-based B2B car-rental/mobility technology provider, founded 2004) and **IdeaWorksCompany** (Jay Sorensen's Shorewood, Wisconsin airline-consulting firm, founded 1996, "the foremost consultant on ancillary revenue"). ✅ Both facts are stated in the bylines of the joint press releases. The companion **CarTrawler Yearbook of Ancillary Revenue by IdeaWorksCompany** is the annual airline-by-airline dataset: the 2024 edition covers 68 airlines' 2023 results; the 2025 edition covers 61 airlines' 2024 results. ✅ (ideaworkscompany.com yearbook pages, Sep 2024 and Sep 2025.)

The verified global series (CarTrawler Worldwide Estimate of Ancillary Revenue):

| Year | Global ancillary revenue | Note | Mark |
| --- | --- | --- | --- |
| 2014 | ~$38B (approx., from the published chart) | pre-growth baseline | ⚠ (chart value) |
| 2019 | **$109.5B** | pre-pandemic record | ✅ |
| 2022 | **$102.8B** | pandemic recovery year | ✅ |
| 2023 | **$117.9B** | record at the time (projected 31 Oct 2023, confirmed 29 Oct 2024) | ✅ |
| 2024 | **$148.4B** | projected 29 Oct 2024; the 2025 Yearbook later confirmed 2024 results "surpassing $148 billion" | ✅ |

Sources: IdeaWorksCompany press releases "Airline Ancillary Revenue Reaches Record $117.9 Billion Worldwide for 2023" (31 Oct 2023: "$117.9 billion... compared to $102.8 billion for 2022 and well above the previous $109.5 billion record in 2019") and "Airline Ancillary Revenue Skyrockets to $148.4 Billion Worldwide for 2024" (29 Oct 2024: "$148.4 billion... compared to $117.9 billion for 2023"); PhocusWire coverage of the 2025 Yearbook ("Total global ancillary revenue for airlines broke records in 2024, surpassing $148 billion"). ✅ Note the brief's anchor figures turned out to be mislabeled in the briefing material: **$102.8B is the 2022 figure, not 2023, and $117.9B is 2023, not 2024** — the corrected series above is what the primary sources actually say.

The yearbook also gives the concentration facts: for 2023, the **top-10 airline companies generated $54.1 billion** in ancillary revenue, "far in excess of the $38.4 billion result for the top 10 companies for the pre-pandemic year of 2019," and **total loyalty revenue for the top-10 programs was $32.2 billion**, up 18.6% on 2022. ✅ (2024 CarTrawler Yearbook of Ancillary Revenue, report page, ideaworkscompany.com.)

### 1.4 Why It Matters to a Bank

Three reasons this product category sits in a banking knowledge repo. **First**, the largest single ancillary line for the US majors is co-branded credit-card remuneration — a banking product by construction (see §6). **Second**, the miles that banks buy from airlines are a financial instrument: an accounting liability on the airline's books, a funding/breakage asset on the bank's card P&L, and — as 2020 proved — securitizable collateral worth tens of billions (see §5.3). **Third**, the airline's ancillary checkout is a payments-processing surface (fees, refunds, chargebacks, fraud) that a bank's acquiring and issuing businesses touch directly (see §8). The worked example in §10 ties all three together from Cymbal Bank's seat.

### 1.5 A Short History of Ancillary Revenue

Ancillary revenue is not new — commissions on hotel and car sales and cargo-related extras are as old as commercial aviation — but the modern, fee-driven version is a product of the past two decades. The verified milestones:

| Year | Milestone | Mark |
| --- | --- | --- |
| 2007–2008 | Ryanair and the European LCCs begin charging separately for checked bags and seats; the modern a la carte wave starts | ⚠-knowledge |
| 2008 | The US DOT requires airlines to disclose optional fees to consumers | ⚠-knowledge |
| 2007–2013 | IATA's EMD standard is developed and pushed into the agency settlement plans (BSP full-EMD target: December 2013) | ⚠ (see §4.2) |
| 2012 | Delta introduces basic economy, the first US legacy unbundling | ✅ |
| 2012 | IATA member airlines adopt NDC Resolution 787, the offer-distribution standard | ✅ |
| 2017 | American and United launch basic economy; the Big Three are now all unbundling | ✅ |
| 2018 | KrisPay (later Kris+) launches in Singapore — loyalty currency as a payments app | ⚠ |
| 2019 | Global ancillary revenue hits the pre-pandemic record of $109.5B | ✅ |
| June 2020 | United raises a $5B loan against MileagePlus, valued at $21.9B | ✅ |
| Sep–Oct 2020 | Delta's SkyMiles-backed financing is upsized from $6.5B to $9B | ✅ |
| 2020–2021 | ~$25.8B of loyalty-backed securitizations across Delta, United and American | ✅ |
| Oct 2023 | CarTrawler/IdeaWorksCompany estimate: $117.9B for 2023 | ✅ |
| Oct 2024 | Estimate: $148.4B for 2024; ancillaries = 14.9% of global airline revenue | ✅ |
| Nov 2024 | Spirit files Chapter 11; US ULCC consolidation begins | ✅ |
| 2025 | Spirit relaunches with Go Big bundles; the 2025 Yearbook confirms 2024 actuals "surpassing $148B"; NDC adoption rises but stays ~20% of global bookings | ✅ for the yearbook figure; ⚠ for NDC share |
| May 2026 | Spirit reported to cease operations; the ULCC shakeout completes | ⚠ |

The pattern across the timeline is layering: each wave — LCC unbundling, then legacy basic economy, then loyalty monetization, then retailing standards (NDC/ONE Order) — added a new revenue layer on top of the previous ones, which is why the global figure kept compounding through the 2010s and after the pandemic dip.

### 1.6 The Key Facts at a Glance

| Fact | Value | Mark |
| --- | --- | --- |
| Global ancillary revenue, 2024 (projected, confirmed >$148B actual) | $148.4B | ✅ |
| Global ancillary revenue, 2023 | $117.9B | ✅ |
| Global ancillary revenue, 2022 | $102.8B | ✅ |
| Pre-pandemic record, 2019 | $109.5B | ✅ |
| Ancillary share of global airline revenue, 2024 | 14.9% (per-airline range 2%–56.4%) | ✅ |
| Top-10 airlines' ancillary revenue, 2023 | $54.1B (vs $38.4B in 2019) | ✅ |
| Top-10 loyalty programs' revenue, 2023 | $32.2B | ✅ |
| Ryanair ancillary share of revenue, FY25 | ≈34% (€4.72B of ≈€13.95B) | ✅ |
| Frontier ancillary per passenger | $67.57 FY2025 / $70.29 FY2024 | ⚠ |
| US-major loyalty revenue / per passenger | >$30B; ~$35 per passenger | ✅ |
| One-way global fare vs a la carte, 2024 | $158 fare vs $24.97 a la carte (2024 $) | ✅ |
| NDC | IATA Resolution 787, 2012 | ✅ |
| ONE Order | IATA Resolution 797 | ✅ |
| MileagePlus valuation (2020) | $21.9B (12× 2019 EBITDA) | ✅ |
| SkyMiles valuation (2026 ranking) | $31.7B (On Point Loyalty) | ✅ |
| Loyalty-backed debt raised 2020–21 | ~$25.8B (Delta $9B, United $6.8B, American $10B) | ✅ |
| Delta/Amex remuneration, 2024 | $7.4B ($10B long-term target) | ✅ |
| SIA co-brand issuers (Singapore) | Amex and UOB | ✅ |

## 2. The Economics — Per-Passenger Benchmarks and Fare Unbundling

### 2.1 The Per-Passenger Metric

The industry's standard productivity metric is **ancillary revenue per passenger** (total ancillary revenue ÷ passengers carried), published airline-by-airline in the CarTrawler Yearbook. It is the honest comparator because total ancillary dollars scale with airline size; per-passenger figures reveal how aggressive the merchandising actually is.

| Airline | Ancillary per passenger | Period | Mark |
| --- | --- | --- | --- |
| Frontier Airlines | $67.57 (FY2025), down ~4% from $70.29 (FY2024) | FY2024/FY2025 | ⚠ (DWU Consulting analysis of airline filings, not the yearbook) |
| Spirit Airlines | historically among the highest US per-passenger figures in the Big Front Seat era (order of $60+/pax) | early 2020s | ⚠-knowledge (widely reported; not re-verified this pass) |
| Allegiant Air | order of ~$60/pax in recent years | 2022–2024 | ⚠-knowledge |
| Ryanair | ≈ €24/pax (€4.72B ÷ 200M passengers, FY25) | FY2025 | ✅ components verified (annual report figures; division is arithmetic) |
| US major airlines | ~$35 per passenger from **loyalty activities alone** (co-brand + miles sales) | 2024 | ✅ (IdeaWorksCompany press release, 29 Oct 2024) |
| AirAsia Group | historically the yearbook's poster child for very high per-passenger ancillary among Asian LCCs | 2019–2023 | ⚠-knowledge (specific figure not re-verified this pass) |

The global a la carte average tells the same story at macro scale: **a la carte revenue per passenger rose from $13.74 in 2015 to $24.97 in 2024** (2024 dollars). ✅ (IdeaWorksCompany press release chart data, 29 Oct 2024.)

### 2.2 Ancillary Share of Total Revenue

Ancillary's share of an airline's revenue is the second headline metric. The verified anchors:

- **Globally, ancillaries were 14.9% of airline revenue in 2024**, with individual airlines ranging from a low of 2% to a high of **56.4%** of their own revenue. ✅ (press release; the range footnote cites the 2023 CarTrawler Yearbook of Ancillary Revenue.)
- **Ryanair generates roughly a third of its revenue from ancillaries.** From the FY25 Annual Report (year ended 31 March 2025): scheduled revenue €9,230M and ancillary revenue €4,719M — ancillary is 4,719 / (9,230 + 4,719) ≈ **34% of group revenue** (≈ €13.95B). FY24: €4,299M ancillary of ~€13.4B (32%). ✅ (investor.ryanair.com annual reports; the "roughly a third" framing matches the arithmetic.)
- **Low-cost carriers as a class** were projected to exceed **30% of global passenger traffic in 2024**, and their a la carte intensity is what lifts the global average. ✅ (press release, 29 Oct 2024.)
- **US majors' loyalty revenue exceeds $30 billion** in aggregate (co-branded cards plus miles sales) — for them, the loyalty bucket rivals or exceeds fee revenue. ✅ (press release, 29 Oct 2024.)

### 2.3 The ULCC Unbundling Model — Ryanair, Spirit, Frontier, Allegiant

The ultra-low-cost model is "unbundling": publish a bare fare, then charge separately for everything that a legacy fare used to include. The classic Ryanair playbook — priority boarding, checked and cabin baggage fees, seat-selection fees, onboard food/drink/duty-free sales, and booking fees — is ⚠-knowledge (universally documented for two decades; the verified numbers are in §2.1–2.2 above). Two structural notes:

- **A la carte is the engine, not the fare.** The 29 Oct 2024 press release: "about 45% of consumers just buy a fare and skip the optional extras... more than 50% pay for baggage, assigned seats, buy-on-board and a wide range of other a la carte services... consumers spending more for extra comfort and convenience subsidize the low fares purchased by the minority of travelers." ✅
- **Spirit's Big Front Seat era ended in 2025.** Spirit's relaunch replaced the single Big Front Seat product with four fare bundles — **Go Big** (Big Front Seat plus bags, snacks, drinks, priority boarding, priority check-in and onboard Wi-Fi), Go Comfy, Go Savvy and Go (bare). PaxEx.Aero: the Go Big bundle is "now nearly identical to the first class offerings of Delta, American, and United, at least in markets where meals are not served." ✅ (PaxEx.Aero, 2025.)
- **Frontier's "New Frontier" bundle shift** reshaped its revenue mix through 2025: as passengers bought bags and seats inside bundles rather than à la carte, ancillary revenue per passenger fell ~4% to $67.57 from $70.29 — "the clearest early sign that the New Frontier shift changes, and partly cannibalizes, the classic ULCC fee engine." ⚠ (DWU Consulting analysis of Frontier filings; secondary source.)
- **Spirit's financial stress is context for the pivot**: Spirit Airlines filed for Chapter 11 bankruptcy protection in November 2024, emerging in 2025 after the failed JetBlue merger. ⚠ (widely reported by Reuters etc.; re-verification pending this pass — see §12.)

### 2.4 The Legacy-Carrier Response — Basic Economy and Branded Fares

The legacies answered ULCC unbundling with **basic economy**: a restricted, non-refundable, no-seat-assignment (or paid-seat) fare that competes with Spirit/Frontier on price while keeping the full-service product above it.

- **Delta was the first US legacy to introduce basic economy, in 2012**, initially on routes where it competed with Spirit. ✅ (Wikipedia, "Basic economy class"; contemporaneous coverage.)
- **American announced Basic Economy on 18 January 2017**, on sale February 2017 in 10 select markets, "no-frills tickets... [giving] customers the option to pay for the services they want." ✅ (American Airlines press release, news.aa.com.)
- **United followed in 2017.** ✅ (InsideFlyer, Feb 2017; Wikipedia.)
- Above basic economy, the legacies sell a **branded-fare ladder** — American: Main Cabin → Main Cabin Extra → Premium Economy → Business → First; Delta: Main Cabin → Comfort+ → Premium Select → Delta One; United: Economy → Economy Plus → Premium Plus → Polaris. ⚠-knowledge (product names are public and stable, but not re-verified this pass).
- The scale effect: for the US Big Three, the co-brand card + miles-sales engine is the growth line — the IdeaWorks press release explicitly credits "the seemingly perpetual cash generator of co-branded credit cards" for US-major ancillary growth. ✅

### 2.5 The Alchemy — Fares Down, Fees Up

The industry's favourite chart (IdeaWorksCompany, from IATA Economic Performance data): the global average one-way fare fell from **$270 in 2015 to $158 in 2024** (2024 dollars) — "$112 back in the pockets of consumers for each trip" — while a la carte revenue per passenger grew by just over $11 in the same window. The quoted conclusion: **"Low fares can't exist without the support of a la carte revenue... That's the amazing alchemy of ancillary revenue. As it has grown... passenger fares have dropped by a larger amount."** ✅ (29 Oct 2024 press release.)

### 2.6 The Model Comparison — ULCC vs Legacy

The two unbundling models described in §2.3–2.4 are best read side by side:

| Dimension | ULCC model (Ryanair, Spirit, Frontier, Allegiant, Scoot) | Legacy model (Delta, American, United, SIA) |
| --- | --- | --- |
| Base fare | Bare — excludes bags, seat selection, priority, often even cabin bags | Fuller — but basic economy strips it back to near-ULCC level |
| Primary ancillary engine | A la carte fees and bundles | Co-brand/loyalty remuneration plus fees |
| Ancillary share of revenue | High end of the 2%–56.4% range; Ryanair ≈34% ✅ | Typically 10–20% (US majors lean on loyalty ✅) |
| Per-passenger profile | Fee-heavy: US ULCCs ~$60–70 ⚠ (Frontier $67.57 ⚠; Spirit historically ~$60+ ⚠-knowledge) | ~$35/passenger from loyalty alone ✅ (US majors) |
| Distribution emphasis | Direct-first | GDS/OTA-heavy plus direct, with NDC growing |
| Pricing style | Dynamic, fee-for-everything | Branded fare families with attribute ladders |
| Loyalty engine | Simpler programs (some have none) | Full FFP with co-brand cards and miles sales |

The convergence is the story: the ULCCs are adding bundles that look like legacy products (Spirit's Go Big ≈ first class ✅), and the legacies are unbundling down to ULCC prices (basic economy ✅). The middle of the market — where the two models collide — is precisely where the merchandising layer (§3) earns its keep.

## 3. The Merchandising — Fare Families, Offers and Point-of-Sale Upsell

This section covers the product-design and merchandising layer. The **checkout mechanics** — payment forms, one-page checkout, wallet flows, reconciliation — are deliberately **not re-derived here**: they live in the sibling [E-Commerce Experience Guide](ecommerce_experience_guide.md) (same folder), which this section cross-references.

### 3.1 Fare Families and Bundles — Basic Economy to Premium

A **fare family** is a set of fare products with a consistent attribute ladder (baggage allowance, seat selection, changeability, priority, miles accrual), priced so that each rung is a natural up-sell. The canonical US ladder is basic economy → main cabin → premium cabin (see §2.4); the ULCC version is bare fare → bundle tiers (Spirit's Go / Go Savvy / Go Comfy / Go Big from 2025, ✅ PaxEx.Aero). In Europe the same idea is branded as fare families on legacy carriers (e.g., Lufthansa Group Light/Classic/Flex, ⚠-knowledge) and as bundles on LCCs. Two merchandising facts matter for a bank reading this:

- **Ancillaries are priced and merchandised like retail, not like regulated tariffs.** Nothing stops an airline from re-pricing a bag fee or a bundle day by day, segment by segment, device by device — which is why the offer layer (§3.2) and the merchandising engine (§3.4) exist.
- **The bundle is the conversion tool.** The 2024 press release's "more than 50% pay for extras" ✅ describes a world in which the extras are presented as pre-packaged bundles at booking, not as a post-hoc fee list.

### 3.2 Offer Construction — the IATA Offer

Under IATA's retailing standards, the sellable unit is an **Offer**: an airline product (or combination) with a price, conditions and ancillaries attached, built at the moment of shopping. Offer construction covers: fare selection, availability, price (fare + taxes/fees + ancillaries), and condition-of-sale terms, assembled into a single machine-readable payload. ⚠-knowledge (the Offer/Order vocabulary is IATA's; the standards detail lives in the NDC layer, §4.3). The merchandising rules that shape offers — bundle composition, attach rules ("priority boarding only with a bag"), fare-family gating — are airline commercial policy implemented in the offer-management system.

### 3.3 Up-Sell and Cross-Sell at the Point of Sale

The ancillary sale happens in four distinct moments, each with its own conversion economics (⚠-knowledge; mechanics cross-ref'd to [E-Commerce Experience Guide](ecommerce_experience_guide.md)):

1. **The shopping/booking flow** — fare-family comparison, "add bag + seat + priority for $X" bundle prompts, travel-insurance checkbox, car/hotel cross-sell (the CarTrawler model: airlines embed car rental at booking and earn commission ✅ — CarTrawler's own positioning).
2. **The seat map** — paid seat selection presented visually; the industry's highest-margin a la carte screen.
3. **Post-booking offers** — email/app "manage booking" flows that sell upgrades, bags, lounges and insurance in the weeks before departure; the same surface used for check-in upsells (priority boarding at 24h).
4. **Onboard** — buy-on-board food/drink/duty-free and in-flight Wi-Fi purchases at the seat or via the app.

The retailing principle underneath all four: **the ancillary is sold at the moment of highest intent, with one-click friction**, and every screen is instrumented (A/B testing, conversion funnels). The conversion-optimization playbook — page design, bundle framing, default-checkbox governance, post-purchase flows — is exactly the material in the [E-Commerce Experience Guide](ecommerce_experience_guide.md) §merchandising/checkout sections; see that guide rather than this one.

### 3.4 Merchandising Engines — OMS, Dynamic Pricing, Personalization

The technology layer that runs the above:

- **Offer Management Systems (OMS)** — the airline-retailing platform that assembles offers from fares, availability, ancillaries and customer context, and serves them to every channel (web, app, GDS, NDC API). ⚠-knowledge.
- **Dynamic pricing of ancillaries** — bag fees, seat prices and bundles priced by route, date, load factor, customer segment and device; seat-map prices already vary seat-by-seat. ⚠-knowledge.
- **Personalization** — customer-context-driven offer selection (elite status, past purchases, origin-destination, time-to-departure); the airline analogue of the engines covered in [Personalization Engines Guide](../technology/personalization_engines_guide.md). ⚠-knowledge.

For a bank, the merchandising layer matters because it **determines what appears on the payment slip**: bundle composition changes the transaction amount, refund rules change chargeback exposure (§8), and miles-earn logic on co-brand cards is often tiered by what was purchased (miles on ancillaries vs fares — §6, §10).

### 3.5 Measuring the Merchandising — Instrumentation

Every ancillary surface is instrumented, and the metrics form a small, stable set (⚠-knowledge — the metric definitions are industry-standard):

- **Attach rate** — the share of bookings that take at least one paid ancillary (the "more than 50% pay for extras" ✅ statistic is the macro version of this).
- **Ancillary revenue per passenger / per booking** — the output measure (§2.1); the yearbook's headline per-airline metric.
- **Take-rate and average order value** — the merchant-style metrics from the e-commerce playbook, applied to the ancillary basket.
- **Conversion by surface** — booking flow vs seat map vs post-booking vs onboard; the seat map typically converts highest because intent and visual merchandising coincide.
- **A/B test lift** — bundle framing, price anchoring, urgency messaging; the same experimentation discipline as any retailer.

The instrumentation, experimentation and analytics mechanics are exactly the material in the [E-Commerce Experience Guide](ecommerce_experience_guide.md) (funnels, A/B testing, conversion optimization) and [Personalization Engines Guide](../technology/personalization_engines_guide.md) (recommendation and targeting); the airline-specific twist is only that the "product" is a bundle of fees attached to a flight, and the "cart" is a PNR or an Order (§4.4).

## 4. The Distribution — Direct, GDS, OTA, EMD, NDC, ONE Order

### 4.1 The Channels — Direct vs GDS vs OTA

Airline distribution is a three-layer market (⚠-knowledge — the channel economics below are stable, well-documented industry knowledge):

- **Direct** (airline.com, mobile app, call centre) — the highest-margin channel: no GDS/agent fees, full merchandising freedom, first access to new ancillaries and bundles. Airlines push hard here (fare discounts for direct, app-only offers).
- **GDS** (Amadeus, Sabre, Travelport) — the legacy indirect backbone: travel agencies and corporate booking tools query GDSs for availability and fares. GDSs charge airlines per-segment booking fees and historically carried only EDIFACT content with limited ancillary merchandising. Airlines pay more to distribute through GDSs, which is precisely why NDC (§4.3) exists.
- **OTA/aggregators** (Expedia, Booking.com, plus metasearch) — either GDS-fed or increasingly NDC-connected; the battleground where bundles and ancillaries get displayed or suppressed depending on the API deal in place.

The strategic fact for this guide: **ancillaries are the reason the distribution war is being fought.** A GDS that cannot sell the bag, the seat and the bundle is a channel that caps ancillary revenue; NDC and ONE Order are IATA's answer, and the offer/order APIs they define are the same interfaces a bank's travel partners will integrate.

### 4.2 The EMD Standard

The **Electronic Miscellaneous Document (EMD)** is the IATA standard for electronically documenting ancillary sales — "all other sales and transactions between airlines and passengers besides electronic tickets." ✅ (IATA portal FAQ; Wikipedia.) EMD comes in two flavours: **EMD-A** (associated with a specific ticket/PNR, e.g., a bag fee on an existing booking) and **EMD-S** (standalone, e.g., a lounge pass or an ancillary bought without a flight). ⚠-knowledge. The industry migrated from paper Miscellaneous Charges Orders (MCOs) to EMD through the late 2000s, with full BSP (billing and settlement plan) implementation targeted by **December 2013**. ⚠ (IATA EMD implementation guidance; the 2013 deadline is from secondary documentation.) EMD is, in effect, the "ancillary ticket" — and ONE Order (§4.4) exists to retire it.

### 4.3 NDC — the Offer/Order API Standard (Verified)

- **What:** NDC (New Distribution Capability) is the IATA program to develop and drive adoption of an XML-based data-transmission standard that lets airlines distribute rich offers (fares + ancillaries + conditions) directly to any seller — agency, OTA, aggregator — independent of the GDS's legacy format. ✅ (iata.org, "Distribution with Offers & Orders (NDC)".)
- **Launch:** IATA member airlines adopted the foundational standard, **Resolution 787 "Enhanced Airline Distribution," in 2012**. ✅ (IATA Resolution 787 PDF; third-party chronology.) IATA filed the resolution with the US Department of Transportation on **11 March 2013** for antitrust review. ✅ (transportation.gov.) The NDC XML schema has since evolved through numbered releases (e.g., 21.3, 24.1). ⚠ (third-party documentation.)
- **Adoption as of 2025–2026 (modest, but rising):** the 2025 state-of-play estimates put NDC at **roughly 20% of global bookings** (~18% of US agency bookings, ~30% leisure, ~6% corporate), with legacy GDS-originated NDC transactions still a small share; ARC separately reported NDC adoption rising among traditional US agencies in 2025. ⚠ (TWAI "The State of Airline Retailing in 2025"; Travel Weekly/ARC, 2025 — both secondary, figures approximate.) The verdict for this guide: **NDC is real, standardized and growing, but a decade-plus in it still carries only a minority of indirect bookings** — exactly the kind of claim this guide flags rather than overstates.

### 4.4 ONE Order — Replacing PNR + E-Ticket + EMD (Verified)

- **What:** ONE Order is the IATA industry initiative to "phase out the current booking (PNRs) and ticketing records (e-tickets and electronic miscellaneous documents, or EMDs)" and replace them with **a single retail-and-customer-focused Order record** — an XML-based standard combining itinerary, payment, ancillaries and accounting into one record. ✅ (iata.org, "Fulfilment with Orders (ONE Order)".)
- **The standard:** adopted as **Resolution 797** by IATA's Passenger Services Conference — "establish a single order process related to the delivery of airline products and services and related accounting processes, with the introduction of new messaging and business process standards." ✅ (IATA Resolution 797 PDF.) Resolution 797 dates from the late-2010s standardization push (PSC(38)797); the precise adoption year is flagged in §12.
- **Adopters:** named-airline ONE Order implementations have been announced over 2021–2026 (Qantas was widely reported as the first end-to-end adopter; American Airlines and Air France-KLM have publicly worked toward ONE Order-aligned offer/order architectures). ⚠ (trade-press reports; not re-verified at primary sources this pass — see §12.)
- **Why it matters:** ONE Order collapses three legacy records (PNR + e-ticket + EMD) into one order object — the same consolidation a bank sees when a payments hub collapses auth/capture/settlement records. It is the data-model prerequisite for true airline retailing (one order, one payment, one refund, one chargeback).

### 4.5 The Integration Layer — Cross-Referenced

The offer/order API surface is an enterprise-integration problem, and this repo already covers it — do not re-derive here:

- [Enterprise Middleware Integration Platform Guide](../technology/enterprise_middleware_integration_platform_guide.md) — the integration backbone (ESB/API gateway, message transformation, orchestration) that NDC/ONE Order endpoints plug into.
- [API Governance Guide](../technology/api_governance_guide.md) — the API lifecycle, versioning and security governance an NDC program needs (schema versioning 21.3 → 24.1 is exactly its subject matter).
- [Legacy Integration Patterns Guide](../technology/legacy_integration_patterns_guide.md) — strangler-fig and anti-corruption-layer patterns for the PNR/EMD estate that ONE Order replaces.

### 4.6 The Settlement Layer — BSP, ARC and Airline Clearing

Behind the offer/order standards sits the money plumbing: the agency settlement systems that move ticket-and-ancillary funds between sellers and airlines.

- **BSP (IATA Billing and Settlement Plan)** — the clearing house that nets and settles agent ticket/EMD sales to airlines in most of the world; agencies remit to the BSP, airlines receive net-of-fees on the settlement cycle. ⚠-knowledge.
- **ARC (Airlines Reporting Corporation)** — the US equivalent: the accredited-agency settlement system through which US ticket and ancillary sales flow. ⚠-knowledge.
- **What changes with NDC/ONE Order** — the data flowing through the settlement layer shifts from ticket/EMD records to offer/order records, and direct-connect sales increasingly bypass the agency plans entirely (settling airline-to-airline or via payment providers). The clearing-and-settlement mechanics themselves (netting, settlement cycles, liquidity) are the [Payment Rails Guide](../banking/payment_rails_guide.md) subject matter; the airline-specific wrapper is BSP/ARC. ⚠-knowledge.

For a bank, the settlement layer is where an airline's "ancillary receivable" becomes cash — and where a co-brand or acquiring partner sees the counterparty's true cash cycle.

## 5. The Loyalty — Frequent-Flyer Programs and the Sale of Miles

### 5.1 The Program Model — Miles as a Liability and an Asset

A frequent-flyer program (FFP) issues **miles** for flying and partner spend; members redeem miles for awards; status tiers (Silver/Gold/Platinum-style) gate benefits. For accounting purposes miles are a **deferred revenue liability** on the airline's books until redeemed or expired; for the airline's commercial team they are also a **sales currency** sold to partners. ⚠-knowledge (the accounting treatment is standard; not re-verified against a specific airline 10-K this pass). The scale facts are verified: for 2023, **total loyalty revenue for the world's top-10 programs was $32.2 billion**, up 18.6% over 2022 (2024 Yearbook), and **US-major loyalty revenue exceeded $30 billion** in aggregate in 2024. ✅ (IdeaWorksCompany.)

### 5.2 Miles Sales to Partners — the Hidden Revenue Line

The quietest large revenue line in travel: airlines **sell miles in bulk to partners** — banks (for co-brand cards), hotel programs, car-rental firms, retailers — at a contracted price per mile, and partners distribute the miles as rewards. The airline books the sale as revenue with a deferred liability for expected redemption; the partner carries the miles cost as a marketing expense. The price of a mile in such contracts is typically on the order of 1–2.5 US cents ⚠-knowledge (contract-dependent, rarely public). The single biggest buyer category is banks: Delta's American Express remuneration — the money Amex pays Delta for SkyMiles — was **$7.4 billion in 2024** ($1.9B in Q2 2024; $1.8B in the September quarter, +6% YoY), with Delta management stating a long-term goal of **$10 billion** from the SkyMiles/Amex relationship. ✅ (Delta Air Lines quarterly results; earnings-call coverage.)

### 5.3 Program Valuations — 2020 Collateral and 2026 Rankings (Verified)

Loyalty programs are valued as stand-alone businesses, and 2020 turned those valuations into collateral:

- **United MileagePlus — $21.9 billion (2020).** United's June 2020 SEC Form 8-K: "Multiplying MPH 2019 EBITDA by a factor of 12 equates to a MileagePlus valuation of approximately $21.9 billion," supporting a $5 billion loan underwritten by Goldman Sachs, Barclays and Morgan Stanley. ✅ (SEC.gov 8-K, 12 June 2020.)
- **Delta SkyMiles — ~$26–27 billion (2020–2021).** SkyMiles was widely valued around $26–27B in contemporaneous analyst and consultancy estimates. ⚠ (secondary sources — TravelSort, TravelPulse/On Point Loyalty; the primary verifiable anchors are the $9B SkyMiles-backed financing and Delta's $6.1B loyalty revenue in 2019.)
- **2026 rankings — On Point Loyalty Top 100:** Delta SkyMiles **$31.7B** (world's most valuable), American AAdvantage **$26.7B**, United MileagePlus **$25.3B**. ✅ (On Point Loyalty 2026 ranking as reported by multiple outlets, April 2026.)
- **The securitization wave (2020–2021):** Delta ($9.0B), United ($6.8B) and American ($10.0B) raised roughly **$25.8 billion** of debt backed by loyalty-program revenue between June 2020 and March 2021 — Delta's deal was announced at $6.5B in September 2020 and upsized to $9B. ✅ (SEC filings and contemporaneous reporting — Bloomberg/Yahoo Finance, Wolf Street; DWU Consulting summary.)

The banking moral: **a loyalty program is a financial asset that can be pledged, securitized and sold — and its largest counterparty is a bank.**

### 5.4 Cross-Reference — the Premium-Card Conventions

The premium-card conventions this repo already documents — annual-fee tiers, interchange economics, spend-based earn structures, travel-benefit packaging — are covered in [Private Banking Guide](../banking/private_banking_guide.md) (the premium-card co-brand section) and are **not re-derived here**. Section 6 applies them to the airline co-brand specifically, and §10 works the numbers.

### 5.5 The Program Economics — Accrual, Redemption, Breakage

The full program loop, in four moves (⚠-knowledge — the mechanisms are standard; the §5.2–5.3 figures are the verified anchors):

1. **Accrual.** Members earn miles by flying (per mile flown or per dollar of fare, tiered by status) and by spending with partners (hotels, car rental, dining, and — biggest of all — co-brand cards). Earn rates are a commercial choice, and airlines have converged on revenue-based earn for flights and category-multiplier earn for cards.
2. **Redemption.** Miles are spent on award seats, upgrades, and increasingly on non-flight rewards (shopping, experiences — the Kris+ model, §9.2). Award inventory is capacity-controlled, which lets airlines manage redemption cost.
3. **Breakage.** Miles expire (or devalue) unredeemed; the un-redeemed portion is never a cost. Breakage is the quiet third leg of program economics alongside accrual volume and redemption cost — and it is why programs push members toward low-value redemptions.
4. **Miles inflation.** Award prices creep upward over time (de facto devaluation), which keeps redemption cost below accrual growth; members experience this as "needing more miles for the same seat."

The banking translation of this loop is the co-brand card's earn-rate/breakage trade-off in §6.2 and the worked numbers in §10.2.

## 6. The Co-Brand — Airline Credit-Card Partnerships

### 6.1 The Partnership Structure — Airline, Bank, Network

The airline co-brand card is a three-party product: the **airline** licenses its brand and miles currency; the **bank** issues the card, prices credit, and carries the regulatory and credit risk; the **network** (Visa/Mastercard/Amex) provides the rails. The airline-bank agreement typically specifies: exclusivity (one bank per market/portfolio), term (commonly ~10 years), miles pricing (per-mile price the bank pays, or a revenue share), marketing commitments, and cardmember benefit funding. ✅ for the exclusivity/term facts where named below; ⚠-knowledge for the general contract shape. The airline's remuneration is booked as ancillary/loyalty revenue — for Delta, $7.4B from Amex in 2024 ✅ — which is why the IdeaWorks yearbook counts co-brand remuneration inside ancillary revenue. ✅

### 6.2 The Economics — Interchange, Miles Cost, Breakage

The bank's P&L on a co-brand card (detail worked in §10; conventions from [Private Banking Guide](../banking/private_banking_guide.md) and [Payment Rails Guide](../banking/payment_rails_guide.md)):

- **Revenue:** interchange (1.5–3% on spend, network- and card-type-dependent ⚠-knowledge), interest on revolvers, annual fees, and fees (late, foreign-transaction).
- **Cost of miles:** the bank pays the airline a contracted **per-mile price** (order of 1–2.5¢ ⚠-knowledge) for every mile awarded, or shares revenue; plus the cost of miles awarded at the bank's own earn rates (e.g., 1 mile per $1 → at 2¢/mile, 2% of spend).
- **Breakage:** miles that expire or go unredeemed are pure margin for whoever holds the liability — a reason programs push miles into low-value redemptions and why "miles inflation" (award price creep) is a permanent feature. ⚠-knowledge.
- **The flywheel:** the airline gets upfront cash (miles sales) + ongoing remuneration; the bank gets card spend, balances and fees; the member gets miles; and the airline's ancillary/loyalty revenue line grows — the "$10 billion long-term" Delta/Amex ambition. ✅

### 6.3 The Named Partnerships (Verified)

- **Delta / American Express:** exclusive US co-brand partner since the 1990s; remuneration $7.4B (2024), $1.9B (Q2 2024), $1.8B (Sep quarter 2024), $10B long-term target. ✅ (Delta results; coverage.)
- **American / Citi:** 5 December 2024 — "Citi to become the exclusive issuer of the AAdvantage co-branded card portfolio in the U.S." under a 10-year agreement, with Mastercard as network (e.g., the Citi/AAdvantage Executive World Legend Mastercard); American describes a "nearly 40-year relationship." ✅ (American Airlines and Citi press releases, 2024–2026.)
- **United / Chase:** Chase is the US co-brand issuer for MileagePlus (United Explorer, Club cards); joint benefit enhancements announced 2025 and February 2026 ("Chase and United Airlines Announce New and Enhanced Travel Benefits..."). ✅ (chase.com, media.chase.com, united.com newsroom.)
- **Singapore / Amex and Singapore / UOB:** see §9.3. ✅

### 6.4 The Securitization Wave — Loyalty Programs as Collateral (Verified)

Verified in §5.3: Delta $9.0B, United $6.8B, American $10.0B of loyalty-backed debt raised 2020–2021, ~$25.8B total. The structural lesson for a bank: **the miles-purchase contract between airline and bank is itself a revenue stream that can be pledged** — the co-brand relationship is simultaneously a product, a funding source for the airline, and collateral. ⚠-knowledge (the legal mechanics are deal-specific).

## 7. The Insurance — Travel-Insurance Ancillaries

### 7.1 The Products

Travel insurance is a **commission-based ancillary** in the IdeaWorks taxonomy — the airline earns a commission on policies sold through its booking flow rather than manufacturing the risk itself. ✅ (the taxonomy's commission bucket, §1.2.) The standard product set sold at airline checkout: **trip cancellation/interruption** (the core product), **medical and emergency-assistance** cover, **baggage and delay** cover, and increasingly **cancel-for-any-reason** riders and flight-specific covers (e.g., "missed connection"). ⚠-knowledge (product names vary by underwriter; the set is stable industry knowledge). Distribution economics: the airline/OTA typically earns a commission in the tens of percent of premium, with the underwriter carrying the risk; policy terms are jurisdiction-regulated (e.g., the Insurance Act / MAS in Singapore; state regulation in the US). ⚠-knowledge.

### 7.2 Distribution at Checkout and Attach Rates

The insurance sale happens **inside the booking flow** — the classic pre-ticketed checkbox ("Add travel insurance for $X") and, on some carriers, a post-booking "protect your trip" prompt. Attach rates vary widely by channel, route and price point — single-digit to ~30%+ on some ULCC flows, higher in Asia-Pacific and on long-haul — and are a standard conversion metric in the merchandising layer (§3.3). ⚠-knowledge (attach-rate ranges are carrier-proprietary; the mechanism is not). Two merchandising facts worth flagging: default-checkbox/opt-out presentation is regulatorily sensitive in several markets (the e-commerce sibling covers the checkbox-governance angle), and insurance is the ancillary with the **highest regulatory weight** — it is a licensed product in every serious market, so the airline is a distributor, never an underwriter.

### 7.3 Claims Flow and the Cross-References

Claims run from the passenger → the airline's insurance partner (or its TPA) → the underwriter, with the airline's role limited to evidence (PNR, e-ticket/EMD, boarding records) and the channel. The claims lifecycle, regulatory compliance and product-governance material is already in this repo — do NOT re-derive it here:

- [Insurance Products, Processes and Compliance Guide](../banking/insurance_products_processes_compliance_guide.md) — the insurance product shelf, claims processing and the compliance overlay (distribution, conduct, disclosures).
- [Health Insurance Guide](../technology/health_insurance_guide.md) — the policy-administration and claims-modelling systems parallels (eligibility, adjudication, reimbursement).

## 8. The Payments — Ancillary Payments, Chargebacks, Fraud

Condensed by design: the rails, interchange and settlement mechanics live in [Payment Rails Guide](../banking/payment_rails_guide.md), and the screening overlay in [FircoSoft Guide](../banking/fircosoft_guide.md).

### 8.1 How Ancillaries Get Paid For

Ancillaries are paid overwhelmingly by card, at four surfaces: (1) **at booking**, as part of the offer/bundle price (single auth, single capture); (2) **post-booking**, through "manage booking" (upgrades, bags, insurance — usually a separate card auth); (3) **at check-in/airport**, kiosk or agent (bag fees, upgrades, priority); and (4) **onboard**, via the app or terminal (buy-on-board, Wi-Fi — increasingly app-based, which cuts terminal costs and enables one-click). The payment record ties to the **EMD** (§4.2) or, under ONE Order (§4.4), to the order itself. ⚠-knowledge (the payment surfaces are standard industry practice; the EMD/order linkage is IATA-standard).

### 8.2 Chargebacks and Fraud — the Fee-Specific Risks

Ancillary payments have a worse fraud/chargeback profile than tickets, for structural reasons:

- **Card-not-present (CNP) fraud at booking** — high-value bundles and "instant upgrade" products are attractive CNP targets; the airline is the merchant of record and eats chargebacks it cannot rebut. ⚠-knowledge.
- **Account takeover of loyalty accounts** — stolen frequent-flyer credentials are used to redeem miles for ancillaries or to attach paid seats/bags to bookings; the airline's fraud team and the bank's card-fraud team share this problem. ⚠-knowledge.
- **Refund/chargeback abuse** — "services not as described" disputes on seat selection, priority boarding and Wi-Fi (the service was delivered but subjective); airlines fight back with evidence (boarding pass, seat-map records, delivery logs). ⚠-knowledge.
- **The regulatory overlay** — the US DOT's refund rules and the EU's rules on optional fees (transparency, opt-in presentation) shape both product design and dispute outcomes. ⚠-knowledge.

The response pattern — velocity checks, 3-D Secure on ancillaries, refund-before-dispute workflows, evidence-pack automation — is standard card-acquiring discipline; the rails and screening specifics are in the cross-referenced guides, and the onboarding/fraud overlay for a card program appears in §10.

### 8.3 Cross-References

- [Payment Rails Guide](../banking/payment_rails_guide.md) — card rails, four-party model, interchange, clearing and settlement.
- [FircoSoft Guide](../banking/fircosoft_guide.md) — sanctions/PEP screening across the card program's onboarding and transaction-monitoring layers.

## 9. The Singapore Angle — the SIA Group

### 9.1 The Group — SIA, Scoot and the Network

Singapore Airlines (SIA) is the flag carrier and full-service network airline of Singapore; **Scoot is the group's low-cost arm**, operating medium/long-haul LCC routes from Singapore, created in 2011–2012 and merged with the former Tigerair in 2017. ⚠-knowledge (the Scoot/Tigerair timeline is well documented but not re-verified this pass). The group's ancillary economics mix the legacy playbook (seat fees, bag fees, upgrades on SIA) with an LCC playbook (Scoot's à-la-carte bundles), plus a loyalty-and-lifestyle engine (KrisFlyer + Kris+) that is unusually advanced for a non-US carrier. For Singapore's macro context — aviation's weight in the economy, Changi's role — see [Singapore GDP and Industry Distribution](../singapore/sg_gdp_industry_distribution.md).

### 9.2 KrisFlyer and Kris+

- **KrisFlyer** is the group's frequent-flyer program, covering SIA, Scoot and airline partners. ⚠-knowledge (program facts stable; membership numbers not re-verified this pass).
- **Kris+** is Singapore Airlines' **lifestyle rewards app** — "Kris+, previously known as KrisPay, is Singapore Airlines' lifestyle app." ✅ (singaporeair.com FAQ.) It launched as KrisPay in 2018 and was rebranded/expanded as **Kris+ in October 2020** ✅ (mainlymiles, 10 Oct 2020): members pay at 1,800+ dining and retail merchants in Singapore via the app to earn miles ("up to 6 miles per S$1" per the app listing ⚠), can convert KrisPay miles to KrisFlyer miles, and the app fronts KrisShop (the group's retail) and Pelago (the group's experiences marketplace). ✅ for the app's existence and merchant network; ⚠ for the earn-rate specifics.

### 9.3 The Co-Brand Cards — Amex and UOB (Verified)

- **American Express Singapore Airlines KrisFlyer Credit Card** — "Earn up to 2 KrisFlyer miles on eligible transactions" (Amex SG product page); miles credit directly into the KrisFlyer account. ✅ (americanexpress.com/sg.)
- **KrisFlyer UOB Credit Card** — earn up to 3 miles per S$1 on dining, Kris+ and more; 1.2 miles/S$1 base; 3 miles/S$1 on eligible SIA, Scoot, KrisShop, Kris+ and Pelago purchases; 2.4 miles/S$1 on dining/food-delivery/online shopping/travel/transport. ✅ (uob.com.sg; SingSaver product sheet.)
- **SIA's co-brand card portal** lists the partner banks' cards for KrisFlyer earn. ✅ (singaporeair.com, "Co-brand Cards".)

### 9.4 SIA Ancillary Practices — Seats, Bags, Upgrades

SIA's own ancillary practices follow the global legacy pattern: **seat-selection fees** on standard seats for lower fare classes (free selection for elites and premium cabins), **baggage fees** embedded in the fare family (the group introduced "Lite" fares with no checked baggage on selected routes), paid **upgrades** (cash or miles, including waitlist upgrades), advance seat/comfort products (e.g., extra-legroom seats), and lounge/experience sales via Kris+ and KrisShop. ⚠-knowledge (the product set is public and stable, but specific fee tables were not re-verified this pass). The structural point for a bank: SIA's co-brand card economics mirror the US model at smaller scale — miles sold to Amex and UOB, remuneration booked as ancillary revenue — and Kris+ turns the loyalty currency into a Singapore domestic payment rail of its own, which is exactly the kind of loyalty-as-payments architecture a bank should study before building one (§10).

### 9.5 The Singapore Co-Brand Competitive Set

The Singapore co-brand and miles-transfer market around KrisFlyer (⚠-knowledge except where the ✅ marks apply):

- **Direct SIA co-brands (verified ✅):** the Amex Singapore Airlines KrisFlyer card (up to 2 mpd ✅) and the KrisFlyer UOB card (up to 3 mpd, with 3 mpd on SIA-group spend ✅) — the two issuers SIA lists on its own co-brand portal ✅.
- **The transfer-card layer:** most other premium cards in Singapore earn bank points that convert into KrisFlyer miles at fixed ratios — a de facto second co-brand market where the bank (not SIA) sets the earn rate and pays the miles cost. ⚠-knowledge (the structure is standard; specific issuers not named here to keep this guide's bank mentions to co-brand partners).
- **The domestic-spend angle:** Kris+ is the differentiator — miles earnable on Singapore dining/retail spend via the app (1,800+ merchants ✅), which competes with bank-card earn on the same transactions and makes the loyalty currency a payments rail (§9.4).
- **What this means for a new entrant:** a Cymbal Bank co-brand in Singapore would price against verified anchors — 1.2–3 mpd structures, S$192-class annual fees (illustrative §10.1) — and would need a reason to exist beyond the two incumbents (e.g., deeper integration with the airline's ancillary checkout, or a bundle with the bank's other travel products).

## 10. The Cymbal Bank Worked Example — An Airline Co-Brand Card Program

### 10.1 The Scenario

Cymbal Bank (the repository's fictional bank persona — see [Private Banking Guide](../banking/private_banking_guide.md) for the persona conventions) is launching an **airline co-brand card in Singapore** with an airline partner. For this worked example the airline is **fictional** (call it "the airline"); **every number below is illustrative**, built to show the mechanics and the P&L shape, not to describe any real program. The structure follows the verified real-world conventions from §6: exclusive issuer agreement, ~10-year term, miles purchased from the airline at a contracted per-mile price, co-brand earn rates, and network rails.

**The commercial terms (illustrative, ⚠):**

| Term | Value | Basis |
| --- | --- | --- |
| Agreement term | 10 years, exclusive in Singapore | convention from Citi/American 2024 ✅ |
| Miles price paid by Cymbal Bank to the airline | S$0.012 per mile (≈ US$0.009) | ⚠ illustrative (real contracts are confidential; US order of magnitude 1–2.5 US¢ ⚠-knowledge) |
| Earn rate | 1.2 miles per S$1 on general spend; 3 miles per S$1 on airline/ancillary spend | mirrors the public KrisFlyer UOB structure ✅ (uob.com.sg) |
| Annual fee | S$192 (waived first year) | ⚠ illustrative |
| Sign-up bonus | 30,000 miles on first spend | ⚠ illustrative |
| Breakage assumption | 12% of awarded miles never redeemed | ⚠ illustrative (industry range ~10–20% ⚠-knowledge) |

### 10.2 The Card Economics — a Worked P&L

Per-card, per-month economics for a steady-state portfolio of **100,000 cards** (illustrative, ⚠):

| Line | Per card / month | Note |
| --- | --- | --- |
| Average spend | S$2,000 | ⚠ |
| **Interchange income** (1.0% of spend) | S$20.00 | SG credit-card interchange ~1% ⚠-knowledge; see [Payment Rails Guide](../banking/payment_rails_guide.md) |
| **Interest income** (30% of cards revolve; avg revolver balance S$3,000; 26.8% APR) | S$20.10 | 0.30 × 3,000 × 26.8%/12 ⚠ |
| **Annual fee** (80% of cards pay) | S$12.80 | S$192 × 0.8 ÷ 12 ⚠ |
| **Other fees** (late, FX) | S$3.00 | ⚠ |
| **Total revenue** | **S$55.90** | |
| Miles cost (2,400 miles × S$0.012) | (S$28.80) | 1.2 mpd × S$2,000 ⚠ |
| Less breakage benefit (12%) | S$3.46 | miles that never redeem are margin ⚠ |
| Net miles cost | (S$25.34) | |
| Card production, servicing, fraud/ops | (S$6.00) | ⚠ |
| Marketing amortization (acquisition spread) | (S$4.00) | ⚠ |
| Funding cost on revolving balances | (S$1.50) | ⚠ |
| **Total cost** | **(S$36.84)** | |
| **Net contribution per card per month** | **S$19.06** | ≈ S$229/card/year ⚠ |
| **Portfolio annual contribution (100,000 cards)** | **≈ S$22.9M** | ⚠ |

Three structural observations, all of which the real-world facts in §6 corroborate:

1. **The miles line is the swing factor.** At S$0.012/mile and 1.2 mpd, miles consume more than the interchange income alone — the card is only profitable because interest, annual fees and breakage sit on top. This is exactly why co-brand programs chase **revolvers and fee-payers**, not transactors, and why earn rates are "up to" figures with category caps (as on the real KrisFlyer UOB card ✅).
2. **Breakage is bank margin.** Each point of breakage is worth ~S$0.29/card/month here (2,400 miles × S$0.012 × 1%) — a portfolio-scale lever worth tens of thousands a year at 100k cards, and the reason airlines and banks both fight over redemption economics.
3. **The airline's side of the same ledger** shows the other half of the flywheel: the airline books 2,400 miles × S$0.012 = **S$28.80/card/month ≈ S$34.6M/year** of miles-sale revenue from Cymbal Bank (illustrative ⚠), plus sign-up-bonus payments (30,000 miles × S$0.012 = S$360 per new card), plus revenue from card-linked benefits the airline funds (priority boarding vouchers, lounge access). That is ancillary revenue in the strict IdeaWorks sense — co-brand remuneration counted inside the yearbook's numbers. ✅ (taxonomy, §1.2.)

### 10.3 Miles Accrual and Settlement Mechanics

The operational flow (⚠-knowledge mechanics; the posting conventions cross-ref [Posting Rules Mechanics Guide](../banking/posting_rules_mechanics_guide.md)):

1. **Accrual at posting.** When a card transaction posts, the accrual engine computes miles = spend × earn rate (with category multipliers from MCC/merchant codes — dining 3 mpd, travel 3 mpd, general 1.2 mpd). Cross-border and excluded categories (cash advances, fees) earn nothing.
2. **Award to the member.** Miles are credited to the member's frequent-flyer account. Mechanically the bank sends a **batch file or API call to the airline's loyalty platform** (daily or monthly); the airline credits the account and the member sees miles land within days. The airline's loyalty system is the system of record for the member balance.
3. **Settlement with the airline.** The bank pays the airline for miles awarded on a contracted schedule — typically **monthly in arrears** against the award batch, or a **prepaid bulk purchase** (bank buys a miles inventory up front, draws it down, true-ups at period end). The airline recognizes the payment as revenue with a deferred liability for expected redemptions; the bank capitalizes it as a marketing/liability cost and recognizes it as miles are actually awarded. ⚠-knowledge (accounting treatment varies by contract and standard).
4. **Reconciliation.** Award batches must reconcile to card transactions (the bank's ledger) and to member credits (the airline's ledger) — the same three-way reconciliation discipline as payments, per the [E-Commerce Experience Guide](ecommerce_experience_guide.md) checkout/reconciliation sections and [Payment Rails Guide](../banking/payment_rails_guide.md).
5. **Redemption is the airline's event.** When a member redeems miles for a flight or upgrade, the airline consumes its redemption liability; the bank is not party to it. The bank's only redemption exposure is where the contract makes the bank fund certain redemptions (rare) or where miles are redeemable for bank products (e.g., statement credits — increasingly common "pay-with-miles" features).

### 10.4 The Partnership Agreement — Liability and Funding

The agreement anatomy (⚠-knowledge general shape, with the verified real-world anchors named):

- **Exclusivity and term.** One issuer per market/portfolio, ~10-year terms with renewal options — as in the verified Citi/American exclusive 10-year deal (Dec 2024) ✅ and the Chase/United relationship ✅.
- **Miles pricing and true-up.** Per-mile price (or revenue share on the airline's ancillary spend), periodic true-ups, and clawback clauses if the airline devalues the program (a real risk in co-brands).
- **Liability split.** Credit risk, regulatory compliance and AML/KYC sit with the **bank** (the issuer); the airline is responsible for the loyalty program's own terms, award fulfilment and redemption inventory. Fraud liability follows network rules (the [Payment Rails Guide](../banking/payment_rails_guide.md) four-party model), and the onboarding/screening overlay follows [FircoSoft Guide](../banking/fircosoft_guide.md).
- **Funding.** The bank funds card receivables (revolving balances) and the miles inventory; the airline receives a stream of miles payments it can — as 2020 proved — pledge or securitize (Delta $9B, United $6.8B, American $10B ✅). A bank structuring a co-brand should price the miles receivable with the same care it prices any purchased-receivable book.
- **Data and governance.** Spend data flows to the airline for personalization (subject to PDPA/consent in Singapore); both parties agree on card-member communication rights, benefit funding (who pays for lounge access, priority boarding vouchers), and termination/change-of-control provisions.

### 10.5 The Lessons

1. **The co-brand is the largest single ancillary contract most airlines sign** — the Delta/Amex $7.4B annual line ✅ is bigger than most airlines' entire fee revenue. Banks are not vendors to airlines; they are co-owners of the loyalty P&L.
2. **The economics hinge on miles price, earn rate and breakage** — three variables that must be modelled jointly at contract time, because they interact (higher earn → more miles cost; higher breakage → less net cost).
3. **The payment and settlement plumbing is standard banking** (posting, reconciliation, chargebacks) applied to a travel product — the repo's payment and e-commerce siblings carry the mechanics; this guide carries the product context.
4. **The airline side is pure ancillary revenue** — which is why the yearbook counts it, and why airline CFOs will fight to keep the miles-sale contract out of the fare regulators' reach.
5. **Loyalty receivables are collateral** — the 2020 securitizations ✅ show that a well-structured co-brand contract is a funding asset, not just a marketing cost.

### 10.6 The Systems Map — Cross-Referenced

The systems that stand up the worked example, mapped to this repo's existing guides rather than re-derived:

- **Card issuing and payments platform** — authorization, posting, statements, interest; the rails and interchange mechanics are in [Payment Rails Guide](../banking/payment_rails_guide.md), and the posting rules (earn, fees, interest, reversal) in [Posting Rules Mechanics Guide](../banking/posting_rules_mechanics_guide.md).
- **Miles accrual engine** — computes earn at posting from MCC/category multipliers; the same pattern as any rewards engine, with the airline-specific twist of category caps and exclusions (§10.3).
- **Loyalty integration** — the batch file or API that awards miles into the airline's loyalty platform; built on the [Enterprise Middleware Integration Platform Guide](../technology/enterprise_middleware_integration_platform_guide.md) backbone, governed by [API Governance Guide](../technology/api_governance_guide.md), and (on the airline side) facing the PNR/EMD legacy estate that [Legacy Integration Patterns Guide](../technology/legacy_integration_patterns_guide.md) describes.
- **Reconciliation and settlement** — three-way matching of card transactions ↔ award batches ↔ airline credits; the discipline of the [E-Commerce Experience Guide](ecommerce_experience_guide.md) reconciliation sections applied to miles instead of goods.
- **Compliance overlay** — onboarding, AML/KYC and sanctions screening per [FircoSoft Guide](../banking/fircosoft_guide.md); card-issuing conduct and disclosure per the [Private Banking Guide](../banking/private_banking_guide.md) premium-card conventions.
- **Fraud and disputes** — CNP/account-takeover controls and chargeback evidence-packs (§8.2), reusing the bank's standard card-fraud tooling.

The architecture lesson is that a co-brand card is a **thin banking layer over a thick integration problem**: the banking components are commodity, the airline-side integration (loyalty platform, offer/order APIs, settlement) is where programs succeed or fail.

## 11. The Claims Audit

Every key factual claim in this guide, its status, and where it was verified. ✅ = verified this pass against a primary source; ⚠ = flagged/approximate/unverified; ⚠-knowledge = well-documented industry knowledge not re-verified this pass; ❌ = could not verify or conflicting sources.

| # | Claim | Status | Source |
| --- | --- | --- | --- |
| 1 | IdeaWorksCompany (Jay Sorensen, Shorewood WI, founded 1996) is the publisher of the ancillary-revenue yearbook; CarTrawler (Dublin, founded 2004) co-publishes/sponsors it | ✅ | Joint press releases 31 Oct 2023 & 29 Oct 2024 (ideaworkscompany.com); About sections |
| 2 | "CarTrawler Yearbook of Ancillary Revenue by IdeaWorksCompany" is the annual publication name | ✅ | ideaworkscompany.com yearbook report pages (2024, 2025 editions) |
| 3 | 2024 Yearbook covers 68 airlines' 2023 results; 2025 Yearbook covers 61 airlines' 2024 results | ✅ | ideaworkscompany.com / dpdcart store listings |
| 4 | Global ancillary revenue: 2019 = $109.5B (record); 2022 = $102.8B; 2023 = $117.9B; 2024 = $148.4B (projected, later confirmed "surpassing $148B") | ✅ | Press releases 31 Oct 2023 & 29 Oct 2024; PhocusWire on the 2025 Yearbook |
| 5 | The briefing note's labels ($102.8B for 2023; $117.9B for 2024) were wrong — the sources say $102.8B = 2022 and $117.9B = 2023 | ✅ | Press releases (corrected series in §1.3) |
| 6 | Ancillaries were 14.9% of global airline revenue in 2024; per-airline range 2%–56.4% | ✅ | Press release 29 Oct 2024 (range footnote cites 2023 Yearbook) |
| 7 | Top-10 airlines' ancillary revenue $54.1B in 2023 (vs $38.4B in 2019); top-10 loyalty programs' revenue $32.2B (+18.6% YoY) | ✅ | 2024 Yearbook report page |
| 8 | US-major loyalty revenue exceeds $30B; ~$35 per passenger from loyalty activities | ✅ | Press release 29 Oct 2024 |
| 9 | A la carte per passenger $13.74 (2015) → $24.97 (2024), 2024 dollars; global one-way fare $270 (2015) → $158 (2024) | ✅ | Press release 29 Oct 2024 (chart data; fares from IATA Economic Performance reports) |
| 10 | ~45% of consumers buy fare only; >50% pay for a la carte extras | ✅ | Press release 29 Oct 2024 |
| 11 | LCC share of global traffic projected to exceed 30% in 2024 | ✅ | Press release 29 Oct 2024 |
| 12 | Ryanair ancillary revenue: FY24 €4.30B; FY25 €4.72B (scheduled €9.23B); ≈34% of group revenue in FY25 ("roughly a third") | ✅ | Ryanair FY24/FY25 Annual Reports (investor.ryanair.com); arithmetic |
| 13 | Ryanair FY26: group revenue €15.54B, scheduled revenue €10.56B (ancillary implied ≈ €4.98B) | ✅ for the revenue lines (FY26 results press release, 18 May 2026); implied ancillary = derived ⚠ |
| 14 | Ryanair ancillary model: priority boarding, bag fees, seat fees, onboard sales | ⚠-knowledge | Universal documentation; revenue numbers verified above |
| 15 | Spirit: Big Front Seat replaced 2025 by Go Big / Go Comfy / Go Savvy / Go bundles | ✅ | PaxEx.Aero |
| 16 | Spirit filed Chapter 11 on 18 Nov 2024 after the blocked JetBlue merger (Jan 2024) | ✅ | Multiple outlets incl. legaldive, bagsthatfly |
| 17 | Spirit's second Chapter 11 (Aug 2025) and cessation of operations (May 2026) | ⚠ | Secondary blogs (elevenflo, omniflights); not re-verified at court dockets |
| 18 | Frontier ancillary per passenger: $70.29 (FY2024) → $67.57 (FY2025), ~4% decline | ⚠ | DWU Consulting analysis (secondary) |
| 19 | Delta introduced US basic economy in 2012 (first US legacy) | ✅ | Wikipedia "Basic economy class"; contemporaneous coverage |
| 20 | American announced Basic Economy 18 Jan 2017, on sale Feb 2017 in 10 markets | ✅ | American Airlines press release (news.aa.com / GCS) |
| 21 | United introduced basic economy in 2017 | ✅ | InsideFlyer; Wikipedia |
| 22 | EMD is the IATA standard for electronic documentation of ancillary sales; EMD-A/EMD-S split; BSP EMD deadline Dec 2013 | ✅ for definition (IATA portal FAQ); ⚠ for EMD-A/S and the 2013 deadline (secondary docs) |
| 23 | NDC = IATA program for an XML standard for airline offer distribution; Resolution 787 adopted 2012; filed with US DOT 11 Mar 2013 | ✅ | iata.org; IATA Resolution 787 PDF; transportation.gov |
| 24 | NDC adoption ≈20% of global bookings in 2025 (~18% US agency, ~30% leisure, ~6% corporate) | ⚠ | TWAI "State of Airline Retailing 2025" (secondary); ARC/Travel Weekly report of rising adoption |
| 25 | ONE Order = IATA initiative replacing PNR + e-ticket + EMD with a single Order record; XML-based | ✅ | iata.org "Fulfilment with Orders (ONE Order)" |
| 26 | ONE Order adopted as Resolution 797 | ✅ | IATA Resolution 797 PDF |
| 27 | Named ONE Order adopters (Qantas first end-to-end; American, Air France-KLM working toward it) | ⚠ | Trade press; not re-verified at primary sources this pass |
| 28 | United MileagePlus valued ≈$21.9B (12× 2019 EBITDA) supporting a $5B loan (June 2020) | ✅ | United SEC Form 8-K (sec.gov, 12 June 2020) |
| 29 | Delta SkyMiles valued ~$26–27B in 2020–2021 estimates | ⚠ | Secondary (TravelSort, TravelPulse/On Point Loyalty) |
| 30 | On Point Loyalty 2026: SkyMiles $31.7B, AAdvantage $26.7B, MileagePlus $25.3B | ✅ | 2026 ranking coverage (multiple outlets, Apr 2026) |
| 31 | Loyalty-backed securitization 2020–21: Delta $9.0B (upsized from $6.5B), United $6.8B, American $10.0B; ≈$25.8B total | ✅ | SEC filings; Bloomberg/Yahoo Finance; Wolf Street; DWU summary |
| 32 | Delta/Amex remuneration $7.4B in 2024; $1.9B Q2 2024; $1.8B Sep-quarter 2024; $10B long-term target | ✅ | Delta quarterly results (news.delta.com PDF); earnings-call coverage |
| 33 | American/Citi: 5 Dec 2024 exclusive-issuer 10-year agreement; Mastercard network; "nearly 40-year relationship" | ✅ | American Airlines & Citi press releases (Dec 2024, 2026) |
| 34 | United/Chase co-brand (Explorer, Club cards); benefit enhancements 2025 and Feb 2026 | ✅ | chase.com, media.chase.com, united.com newsroom |
| 35 | SIA group: Scoot = LCC arm (2011–12 launch, Tigerair merger 2017) | ⚠-knowledge | Well documented; not re-verified this pass |
| 36 | Kris+ = SIA lifestyle app, formerly KrisPay; rebranded Oct 2020; 1,800+ SG merchants | ✅ | singaporeair.com FAQ; mainlymiles (10 Oct 2020); Google Play listing |
| 37 | Amex KrisFlyer card (up to 2 mpd); KrisFlyer UOB card (up to 3 mpd; 1.2 base; 3 on SIA/Scoot/KrisShop/Kris+/Pelago; 2.4 dining etc.) | ✅ | americanexpress.com/sg; uob.com.sg; SingSaver |
| 38 | SIA ancillary practices (seat fees, Lite fares, upgrades, Kris+/KrisShop/Pelago) | ⚠-knowledge | Public product set; fee tables not re-verified |
| 39 | Travel insurance is a commission-based ancillary; sold at checkout; attach rates single-digit to ~30%+ | ⚠-knowledge | Industry practice; taxonomy bucket ✅ |
| 40 | Ancillary payment surfaces (booking, manage-booking, airport, onboard) and chargeback/fraud profile | ⚠-knowledge | Industry practice |

## 12. What Could Not Be Verified

The following could not be verified to primary-source standard in this pass, and are flagged rather than asserted:

- **Precise adoption year of ONE Order Resolution 797.** The IATA resolution PDF confirms the content and the PSC(38) numbering, and the program is documented on iata.org, but a single authoritative "adopted in year X" statement was not captured this pass. The program is commonly dated to the 2016–2018 standardization window ⚠.
- **Named ONE Order airline adopters.** Trade-press reports name Qantas (first end-to-end) and American/Air France-KLM as working toward ONE Order-aligned architectures; none was re-verified at an airline or IATA primary page this pass ⚠.
- **NDC adoption percentages.** The ~20%-of-global-bookings (2025) figure comes from a consultancy analysis, not from IATA or a GDS; treat as approximate ⚠.
- **Delta SkyMiles' 2020–2021 valuation of ~$26–27B.** Multiple secondary sources agree on the range, but the underlying analyst report (Cowen-era) was not located at primary source this pass; the verified anchors are the $9B financing and $6.1B 2019 loyalty revenue ⚠.
- **Frontier's FY2024/FY2025 per-passenger ancillary ($70.29 → $67.57).** Secondary analysis of filings; the yearbook figure for Frontier was not available to this pass ⚠.
- **Spirit's 2025 second Chapter 11 and May 2026 cessation of operations.** Reported by multiple outlets but not verified at court dockets or SEC filings this pass; the November 2024 Chapter 11 IS verified ✅.
- **Kris+ earn rate ("up to 6 miles per S$1") and the 2018 KrisPay launch year.** From app-store and blog sources; SIA's own pages confirm the app and rebrand but not the earn-rate cap ⚠.
- **SIA/Scoot/Tigerair exact dates (2011–12 launch; 2017 merger) and SIA fee tables.** Well-documented industry knowledge, not re-verified this pass ⚠-knowledge.
- **EMD BSP implementation deadline (December 2013) and the EMD-A/EMD-S split.** The EMD standard itself is verified ✅ at IATA; these details rest on secondary documentation ⚠.
- **Airline per-passenger benchmarks for AirAsia, Spirit and Allegiant (specific figures).** These are ⚠-knowledge ranges; the yearbook's per-airline tables are a paid publication and were not accessible this pass.
- **Miles pricing, breakage rates, interchange rates and all worked-example numbers in §10.** Illustrative by design; real contracts are confidential ⚠.
- **2014 global ancillary revenue (~$38B) read off the press-release chart.** Chart value, not a stated figure ⚠.

## 13. The Glossary

| Term | Definition |
| --- | --- |
| **A la carte revenue** | Ancillary fees paid directly by passengers for optional extras (baggage, seats, buy-on-board, Wi-Fi). |
| **Ancillary revenue** | All airline cashflow beyond the transportation of passengers — a la carte fees, commissions on third-party products, and loyalty/miles-sale revenue. |
| **Attach rate** | The percentage of bookings to which an ancillary (e.g., travel insurance) is sold. |
| **Basic economy** | A restricted, lowest-price fare class that unbundles seat selection, bags and changeability; the legacy carriers' answer to ULCC pricing. |
| **Branded fares / fare families** | A ladder of fare products with consistent attribute bundles (bags, seats, flexibility, miles) at increasing prices. |
| **Breakage** | Miles (or points) that are awarded but never redeemed; pure margin for the liability holder. |
| **CarTrawler** | Dublin-based B2B travel-technology provider (car rental/mobility); co-publisher of the annual ancillary yearbook. |
| **Co-brand card** | A credit card issued by a bank under an airline's brand and miles currency, with the airline receiving remuneration. |
| **Deferred revenue liability** | The accounting liability airlines carry for miles sold/awarded but not yet redeemed. |
| **Direct channel** | The airline's own sales surface (website, app, call centre) — highest-margin distribution. |
| **EMD (Electronic Miscellaneous Document)** | The IATA standard electronic document for ancillary sales; EMD-A (associated with a ticket) and EMD-S (standalone). |
| **Fare unbundling** | Removing services from the base fare and selling them separately; the ULCC business model. |
| **FFP (frequent-flyer program)** | The airline loyalty program that issues and redeems miles. |
| **GDS (Global Distribution System)** | The legacy indirect distribution networks (Amadeus, Sabre, Travelport) used by travel agencies. |
| **IdeaWorksCompany** | Jay Sorensen's airline-consulting firm (Shorewood, Wisconsin); author of the ancillary yearbook and worldwide estimates. |
| **Interchange** | The fee the card-issuing bank receives from the merchant's bank on every card transaction. |
| **Kris+** | Singapore Airlines' lifestyle rewards app (formerly KrisPay). |
| **Loyalty revenue** | Airline revenue from miles sales to partners and co-brand remuneration. |
| **Miles price** | The contracted per-mile amount a partner (e.g., a bank) pays the airline for miles. |
| **NDC (New Distribution Capability)** | IATA's XML standard (Resolution 787) for airlines to distribute rich offers to any seller. |
| **Offer** | The IATA retailing unit: an airline product with price, conditions and ancillaries, built at shopping time. |
| **OMS (Offer Management System)** | The airline platform that assembles and serves offers across channels. |
| **ONE Order** | IATA's initiative (Resolution 797) replacing PNR + e-ticket + EMD with a single Order record. |
| **PNR (Passenger Name Record)** | The legacy reservation record for a passenger's booking. |
| **Securitization** | Pledging a revenue stream (e.g., loyalty-program cashflows) as collateral for debt. |
| **ULCC (ultra-low-cost carrier)** | The most aggressive unbundlers (Spirit, Frontier, Ryanair, Allegiant, Wizz Air, Scoot). |

## 14. The Sources — Primary References

**IdeaWorksCompany / CarTrawler (the yearbook and estimates):**
- IdeaWorksCompany press release, 31 Oct 2023 — "Airline Ancillary Revenue Reaches Record $117.9 Billion Worldwide for 2023": https://ideaworkscompany.com/wp-content/uploads/2023/10/Press-Release-178-Worldwide-Estimate-2023.pdf
- IdeaWorksCompany press release, 29 Oct 2024 — "Airline Ancillary Revenue Skyrockets to $148.4 Billion Worldwide for 2024": https://ideaworkscompany.com/wp-content/uploads/2024/10/Press-Release-188-Worldwide-Estimate-2024.pdf
- 2024 CarTrawler Yearbook of Ancillary Revenue — report page: https://ideaworkscompany.com/2024-cartrawler-yearbook-of-ancillary-revenue-report/
- 2025 Yearbook of Ancillary Revenue — store/excerpt: https://ideaworkscompany.dpdcart.com/product/246134 ; excerpt PDF: https://ideaworkscompany.com/wp-content/uploads/2025/08/2025-Yearbook-of-Ancillary-Revenue-Excerpt-1.pdf
- IdeaWorksCompany Reports index: https://ideaworkscompany.com/reports/

**IATA (distribution standards):**
- NDC program page: https://www.iata.org/en/programs/airline-distribution/retailing/ndc
- Resolution 787 (Enhanced Airline Distribution) PDF: https://www.iata.org/contentassets/6de4dce5f38b45ce82b0db42acd23d1c/ndc-resolution-787.pdf
- ONE Order program page: https://www.iata.org/en/programs/airline-distribution/retailing/one-order/
- Resolution 797 (ONE Order) PDF: https://www.iata.org/contentassets/72cbd60393ff42b5975d90ce9e049a7d/oneorder-resolution-797.pdf
- EMD definition (IATA portal FAQ): https://portal.iata.org/faq/articles/en_US/FAQ/What-is-an-Electronic-Miscellaneous-Document-EMD-1415811054748
- US DOT filing of Resolution 787: https://www.transportation.gov/airconsumer/iata-resolution-787-order-show-cause

**Airlines (investor relations and press):**
- Ryanair FY25 Annual Report: https://investor.ryanair.com/wp-content/uploads/2025/05/Ryanair-2025-Annual-Report.pdf ; FY26 results: https://corporate.ryanair.com/news/ryanair-2025-26-pat-rises-40-to-e2-26bn-pre-except-traffic-grows-4-to-208m-despite-boeing-delays/
- American Airlines Basic Economy press release (18 Jan 2017): https://news.aa.com/news/news-details/2017/American-Airlines-Introduces-New-Basic-Economy-Fare/default.aspx
- American/Citi exclusive co-brand agreement (5 Dec 2024): https://americanairlines.gcs-web.com/news-releases/news-release-details/american-airlines-and-citi-extend-and-expand-co-branded-card
- Delta September-quarter 2024 results (Amex remuneration): https://news.delta.com/sites/default/files/2024-10/deltaairlines_announces_september_quarter_2024_results_vf.pdf
- United SEC Form 8-K (12 June 2020, MileagePlus $21.9B / $5B loan): https://www.sec.gov/Archives/edgar/data/100517/000110465920073190/tm2022354d3_8k.htm
- Chase/United co-brand: https://media.chase.com/news/united-refresh-2025 ; https://www.united.com/en/us/newsroom/announcements/cision-125442
- Delta SkyMiles financing upsized to $9B: https://finance.yahoo.com/news/delta-upsizes-loyalty-program-backed-072207557.html ; Wolf Street analysis: https://wolfstreet.com/2020/10/15/whats-behind-the-billions-that-airlines-raised-via-frequent-flier-programs-youre-the-collateral/

**Singapore Airlines group:**
- Kris+ (lifestyle app) pages: https://www.singaporeair.com/en_UK/us/ppsclub-krisflyer/use-miles/krisplus/ ; KrisPay/Kris+ FAQ: https://kflogin.singaporeair.com/en_UK/my/ppsclub-krisflyer/use-miles/krispay/
- SIA co-brand card portal: https://www.singaporeair.com/en_UK/sg/ppsclub-krisflyer/earn-miles/earn-on-the-ground/co-brand-cards/
- Amex Singapore Airlines KrisFlyer card: https://www.americanexpress.com/sg/credit-cards/singapore-airlines-krisflyer-credit-card/
- KrisFlyer UOB card: https://www.uob.com.sg/personal/cards/travel/krisflyer-card.page

**Trade press and secondary (flagged ⚠ where used for figures):**
- Simple Flying (2023 estimate coverage): https://simpleflying.com/airline-ancillary-revenues-record-118-billion/
- PhocusWire (2025 Yearbook coverage): https://www.phocuswire.com/airlines-set-ancillary-records-lead-into-revenue-streams
- PaxEx.Aero (Spirit Go Big bundles): https://paxex.aero/spirit-fare-bundles/
- TWAI — State of Airline Retailing 2025 (NDC adoption): https://www.twai.com/Resources/the-state-of-airline-retailing-in-2025
- Travel Weekly/ARC (NDC adoption rising): https://www.travelweekly.com/Travel-News/Travel-Technology/ARC-reports-rising-NDC-adoption-2025
- DWU Consulting (loyalty securitization; Frontier per-pax): https://dwuconsulting.com/dwu-ai/airline-loyalty-program-securitization ; https://dwuconsulting.com/dwu-ai/frontier-airlines-strategy
- On Point Loyalty 2026 ranking coverage: https://aerospaceglobalnews.com/news/worlds-most-valuable-airline-loyalty-programmes-2026/
- Wikipedia — Basic economy class: https://en.wikipedia.org/wiki/Basic_economy_class ; Electronic miscellaneous document: https://en.wikipedia.org/wiki/Electronic_Miscellaneous_Document
- Spirit Chapter 11 coverage: https://www.legaldive.com/news/spirit-airlines-files-for-bankruptcy-months-after-jetblue-merger-demise/733210/
- mainlymiles (Kris+ rebrand): https://mainlymiles.com/2020/10/10/singapore-airlines-rolls-out-kris-as-its-new-krispay-lifestyle-app/

**Repository cross-references (in-text):** [E-Commerce Experience Guide](ecommerce_experience_guide.md) · [Business Case Development](business_case_development_guide.md) · [Private Banking Guide](../banking/private_banking_guide.md) · [Payment Rails Guide](../banking/payment_rails_guide.md) · [FircoSoft Guide](../banking/fircosoft_guide.md) · [Posting Rules Mechanics Guide](../banking/posting_rules_mechanics_guide.md) · [Insurance Products, Processes and Compliance Guide](../banking/insurance_products_processes_compliance_guide.md) · [Enterprise Middleware Integration Platform Guide](../technology/enterprise_middleware_integration_platform_guide.md) · [API Governance Guide](../technology/api_governance_guide.md) · [Legacy Integration Patterns Guide](../technology/legacy_integration_patterns_guide.md) · [Health Insurance Guide](../technology/health_insurance_guide.md) · [Personalization Engines Guide](../technology/personalization_engines_guide.md) · [Singapore GDP and Industry Distribution](../singapore/sg_gdp_industry_distribution.md)

## 15. The Closing Summary — The Fare's Hidden Half

Ancillary revenue is the fare's shadow economy: $117.9 billion in 2023, a projected $148.4 billion in 2024, 14.9% of everything airlines earn — and for the most aggressive carriers, more than half of their revenue. The playbook that produces those numbers is now a mature product discipline: unbundle the fare, merchandise the extras like retail, distribute through every channel the standards allow (EMD today, NDC and ONE Order tomorrow), and — above all — turn the loyalty program into a financial product whose biggest customer is a bank. The co-brand card is where the two industries fuse: airlines sell miles, banks sell cards, and both monetize breakage; Delta's $7.4-billion-a-year American Express line, the $21.9-billion MileagePlus valuation and the $25.8-billion securitization wave are the proof that loyalty cashflows are banking-grade collateral. For a bank like Cymbal Bank, the airline co-brand is not a travel perk — it is a portfolio, a payments surface and a funding asset in one contract, with the airline's ancillary revenue on one side of the ledger and the card economics on the other. Understand the products, price the miles, respect the breakage, and the airline's hidden half of the fare becomes the bank's visible half of the P&L: the fare's hidden half.
