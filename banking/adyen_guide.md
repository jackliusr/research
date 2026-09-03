# Adyen: The Single-Platform Payments Company — A Comprehensive Guide

**The Business, History, Products, Licensing, and Technology of Adyen N.V. (adyen.com) — from a 2006 Amsterdam Founding by Pieter van der Does and Arnout Schuijff, through the 2017 Dutch Banking Licence and the June 2018 Euronext IPO, to H1 2026 Processed Volume of €803.8 Billion, the Talon.One and Orb Acquisitions, and a Cymbal Bank Omni-Channel Merchant Worked Example**

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Banking Domain / Payments-Platform Company Deep-Dive — the unified-commerce acquiring model, the single-platform architecture, the local-acquiring and banking-licence strategy (DNB/ECB, PRA/FCA, OCC, MAS), the marketplace and embedded-finance product stack (Adyen for Platforms, Issuing, Capital, Accounts), the regulatory and AML overlay, and the Cymbal Bank corporate-client lens
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026
> **Companion guides (sibling, same folder — the banking cluster):** [Reap Global](reap_global_guide.md) (the fintech-company profile genre and the Cymbal Bank worked-example conventions — cross-ref §11) · [Payment Rails](payment_rails_guide.md) (clearing and settlement mechanics for the acquiring and payout legs — cross-ref §5, §8, §11; do not re-derive) · [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) (the Singapore Payment Services Act regime and the Cymbal Bank persona conventions — cross-ref §8, §11) · [Fircosoft](fircosoft_guide.md) (sanctions/AML screening themes — cross-ref §8, condensed) · [Bitunix](bitunix_guide.md) (the structural template for this guide's header, ToC, claims-audit and honesty conventions)
> **Companion guides (other folders):** [Ancillary Revenue Products](../management/ancillary_revenue_products_guide.md) (merchant/card-acquiring and card-program economics — cross-ref §4, §5, §11; do not re-derive) · [Enterprise Middleware & Integration Platforms](../technology/enterprise_middleware_integration_platform_guide.md) (API/integration-platform themes — cross-ref §9, condensed)

---

**How to use this guide:** Section 1 is the overview — the short answer, the key-facts table, why a bank should care, and the evidence base. Section 2 is the company profile — the 2006 founding, the two verified co-founders (Pieter van der Does and Arnout Schuijff, both ex-Bibit), the Amsterdam headquarters, and the legal form (Adyen N.V., a Dutch credit institution). Section 3 is geography and footprint — headquarters, regional leadership, country offices, and the processing footprint. Section 4 is the mission and business model — the "single platform" framing and the acquiring revenue model (take rate, local acquiring, platform-of-platforms). Section 5 covers the payments products — online payments, in-person/point-of-sale terminals, and unified commerce — verified at adyen.com and docs.adyen.com. Section 6 covers the financial-products roster — Adyen for Platforms, Issuing, Capital, Accounts, Payouts, and the 2026 additions (Agentic, Intelligent Money Movement, Talon.One, Orb). Section 7 is growth and market position — the funding history (2011 Index Ventures round, the 2014 General Atlantic round, the 2015 ICONIQ round at a $2.3 billion valuation — and the absence of KKR), the June 2018 IPO, processed-volume claims (⚠ company-reported), and named customers (✅ where press-verified, ⚠ where company-claimed). Section 8 is licensing and compliance — the 2017 Dutch banking licence under DNB/ECB supervision, the 2021 US Federal Foreign Branch, the 2023 UK PRA/FCA authorisation, the Singapore MAS Major Payment Institution licence (cross-referencing the MAS guide), and the AML/KYC posture (cross-referencing the Fircosoft guide, condensed). Section 9 is technology — the platform facts that are public, the data and risk layers, and everything not disclosed (⚠). Section 10 is industry context — Adyen versus Stripe, PayPal/Braintree, Worldpay, Checkout.com, and Fiserv. Section 11 is the Cymbal Bank worked example — a fictional omni-channel retail-and-marketplace corporate client on an Adyen-style platform. Section 12 is the claims audit (✅/⚠/❌), closing with §12.4 "What Could Not Be Verified". Section 13 is the glossary. Section 14 is cross-references and the closing summary.

---

## Table of Contents

1. [The Overview](#1-the-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Key-Facts Table](#12-the-key-facts-table)
   - 1.3 [Why This Matters to a Bank](#13-why-this-matters-to-a-bank)
   - 1.4 [The Evidence Base at a Glance](#14-the-evidence-base-at-a-glance)
2. [The Company Profile — Founding, Founders, Headquarters, and Legal Form](#2-the-company-profile--founding-founders-headquarters-and-legal-form)
   - 2.1 [The Founding Year: 2006](#21-the-founding-year-2006)
   - 2.2 [The Co-Founders: Pieter van der Does and Arnout Schuijff](#22-the-co-founders-pieter-van-der-does-and-arnout-schuijff)
   - 2.3 [The Bibit Heritage](#23-the-bibit-heritage)
   - 2.4 [Headquarters and Legal Form: Adyen N.V., Amsterdam](#24-headquarters-and-legal-form-adyen-nv-amsterdam)
   - 2.5 [The Company Table](#25-the-company-table)
   - 2.6 [The Footprint Timeline](#26-the-footprint-timeline)
3. [Geography and Footprint](#3-geography-and-footprint)
   - 3.1 [Headquarters and the Office Network](#31-headquarters-and-the-office-network)
   - 3.2 [Regional Leadership and Country Managers (September 2026)](#32-regional-leadership-and-country-managers-september-2026)
   - 3.3 [The Processing Footprint — Local Acquiring and Licence Passporting](#33-the-processing-footprint--local-acquiring-and-licence-passporting)
4. [The Mission and Business Model — the Single-Platform Model](#4-the-mission-and-business-model--the-single-platform-model)
   - 4.1 [The Mission Statement](#41-the-mission-statement)
   - 4.2 [The Business Model — the Single-Platform Payments Model](#42-the-business-model--the-single-platform-payments-model)
   - 4.3 [The Revenue Model — Processing Fees, Take Rate, and the Platform of Platforms](#43-the-revenue-model--processing-fees-take-rate-and-the-platform-of-platforms)
   - 4.4 [The Market Context](#44-the-market-context)
5. [The Products — the Payments Platform](#5-the-products--the-payments-platform)
   - 5.1 [Online Payments — Gateway, Processing, and Checkout](#51-online-payments--gateway-processing-and-checkout)
   - 5.2 [In-Person Payments — Terminals and Point-of-Sale](#52-in-person-payments--terminals-and-point-of-sale)
   - 5.3 [Unified Commerce — One Platform Across Channels](#53-unified-commerce--one-platform-across-channels)
   - 5.4 [Cross-Border and Local Payment Methods](#54-cross-border-and-local-payment-methods)
6. [The Products — the Financial-Products Suite](#6-the-products--the-financial-products-suite)
   - 6.1 [Adyen for Platforms — Marketplace Payments and Payouts](#61-adyen-for-platforms--marketplace-payments-and-payouts)
   - 6.2 [Adyen Issuing](#62-adyen-issuing)
   - 6.3 [Adyen Capital](#63-adyen-capital)
   - 6.4 [Adyen Accounts, Payouts, and Embedded Financial Products](#64-adyen-accounts-payouts-and-embedded-financial-products)
   - 6.5 [The 2026 Product Surface — Agentic, Personalize, Talon.One, and Orb](#65-the-2026-product-surface--agentic-personalize-talonone-and-orb)
   - 6.6 [The Product-Surface Map](#66-the-product-surface-map)
7. [Growth and Market Position — Funding, IPO, Volumes, and Customers](#7-growth-and-market-position--funding-ipo-volumes-and-customers)
   - 7.1 [The Funding History — 2011 to 2015](#71-the-funding-history--2011-to-2015)
   - 7.2 [The KKR Question](#72-the-kkr-question)
   - 7.3 [The IPO — June 13, 2018, Euronext Amsterdam](#73-the-ipo--june-13-2018-euronext-amsterdam)
   - 7.4 [Processed Volume and Financial Scale (2025–2026)](#74-processed-volume-and-financial-scale-20252026)
   - 7.5 [The Named Customers — Verified and Company-Claimed](#75-the-named-customers--verified-and-company-claimed)
   - 7.6 [The Market Position](#76-the-market-position)
8. [Licensing and Compliance](#8-licensing-and-compliance)
   - 8.1 [The Dutch Banking Licence — DNB and ECB Supervision](#81-the-dutch-banking-licence--dnb-and-ecb-supervision)
   - 8.2 [The UK Authorisation — PRA and FCA (2023)](#82-the-uk-authorisation--pra-and-fca-2023)
   - 8.3 [The US Federal Foreign Branch — OCC (2021)](#83-the-us-federal-foreign-branch--occ-2021)
   - 8.4 [The Singapore Angle — MAS Major Payment Institution (Cross-Referenced)](#84-the-singapore-angle--mas-major-payment-institution-cross-referenced)
   - 8.5 [The Enforcement Climate — DNB's Wwft Record 2025–2026](#85-the-enforcement-climate--dnbs-wwft-record-20252026)
   - 8.6 [The KYC/AML Posture (Cross-Referenced, Condensed)](#86-the-kycaml-posture-cross-referenced-condensed)
   - 8.7 [The Licensing Table](#87-the-licensing-table)
9. [Technology — Platform Architecture, Data, and Risk](#9-technology--platform-architecture-data-and-risk)
   - 9.1 [The Architecture Claims That Are Public](#91-the-architecture-claims-that-are-public)
   - 9.2 [The API and Integration Surface (Cross-Referenced)](#92-the-api-and-integration-surface-cross-referenced)
   - 9.3 [Data, Risk, and the Machine-Learning Layer](#93-data-risk-and-the-machine-learning-layer)
   - 9.4 [What Is Not Disclosed](#94-what-is-not-disclosed)
10. [Industry Context — the Competitive Landscape](#10-industry-context--the-competitive-landscape)
    - 10.1 [The Competitive Field](#101-the-competitive-field)
    - 10.2 [The Competitive-Comparison Table](#102-the-competitive-comparison-table)
11. [The Cymbal Bank Worked Example — An Omni-Channel Merchant on an Adyen-Style Platform](#11-the-cymbal-bank-worked-example--an-omni-channel-merchant-on-an-adyen-style-platform)
    - 11.1 [The Scenario](#111-the-scenario)
    - 11.2 [Onboarding and the KYC/AML Overlay](#112-onboarding-and-the-kycaml-overlay)
    - 11.3 [The Acquiring and Settlement Flows](#113-the-acquiring-and-settlement-flows)
    - 11.4 [The Marketplace Payout Mechanics](#114-the-marketplace-payout-mechanics)
    - 11.5 [Reconciliation and the Bank's Control Overlay](#115-reconciliation-and-the-banks-control-overlay)
    - 11.6 [The Lessons](#116-the-lessons)
12. [The Claims Audit — Verified, Flagged, Rejected](#12-the-claims-audit--verified-flagged-rejected)
    - 12.1 [The Verified Claims (✅)](#121-the-verified-claims-)
    - 12.2 [The Flagged Claims (⚠)](#122-the-flagged-claims-)
    - 12.3 [The Rejected or Not-Found Claims (❌)](#123-the-rejected-or-not-found-claims-)
   - 12.4 [What Could Not Be Verified](#124-what-could-not-be-verified)
13. [The Glossary](#13-the-glossary)
14. [Cross-References and the Closing Summary](#14-cross-references-and-the-closing-summary)

---

## 1. The Overview

### 1.1 The Short Answer

**Adyen N.V.** (adyen.com; Euronext Amsterdam: ADYEN) is a Dutch payments-technology company that operates as a **licensed acquiring bank and full-stack financial-technology platform** under a single global product: merchants connect once and accept payments online, in-app, and in person, then layer on payouts, bank accounts, card issuing, and financing — the model the company has marketed for two decades as the **"single platform"** (the phrase appears verbatim in Adyen's own press materials, e.g. the June 2021 US-branch release and the August 2026 H1-results release). The company was **founded in 2006 in Amsterdam** by **Pieter van der Does** and **Arnout Schuijff**, both veterans of Bibit, a Dutch payments firm (verified: adyen.com/about/team; Wikipedia; Forbes). It went public on **June 13, 2018, on Euronext Amsterdam** at €240 per share — a €7.1 billion market capitalisation — and its shares roughly doubled on day one (Adyen press release; Euronext press release; Reuters). ✅

Adyen's structural claim to distinctiveness is that it **integrated the roles most merchants previously stitched together** — payment gateway, payment service provider (PSP), processor, risk, and — since its **2017 Dutch banking licence** — acquirer and settlement bank — into one licensed stack with direct connections to the card schemes (Adyen Knowledge Hub, "Adyen's Banking License," April 2024). The company reports **€803.8 billion processed volume and €1,302.9 million net revenue for H1 2026** (+24% and +19% year-on-year respectively), and **€1,394.3 billion processed volume and €2,364.2 million net revenue for FY2025** (Adyen H1 2026 financial-results press release, August 13, 2026; Adyen FY2025 results, February 2026 — ⚠ company-reported, but published through regulated investor channels). In July 2026 it completed its **first acquisitions in company history** — Talon.One (omnichannel loyalty/promotions) and Orb (usage-based billing) — after two decades of organic-only growth, and it has been launching what it calls "the complete financial operating system for modern commerce": marketplace payments (Adyen for Platforms), Issuing, Capital, Accounts, and 2026 additions including Adyen Agentic and Intelligent Money Movement (H1 2026 release; Wikipedia).

For a bank like Cymbal Bank, Adyen matters on four fronts: as a **counterparty and partner class** (a licensed acquiring bank and platform-of-platforms that both competes with banks for merchant acquiring and depends on banks for correspondent and settlement relationships — the worked example in §11 shows both faces), as a **regulatory reference** (the Dutch-licence-plus-MAS-MPI structure is a template for how a non-bank fintech grows into a supervised credit institution — cross-ref the [MAS guide](mas_regulations_guidelines_guide.md) §2.4 for the Singapore analogue), as a **technology benchmark** (single-API, single-contract, single-data-model architecture that banks now must interoperate with — cross-ref the [middleware guide](../technology/enterprise_middleware_integration_platform_guide.md)), and as a **worked-example generator** for this repository's Cymbal Bank persona conventions (§11).

### 1.2 The Key-Facts Table

| Aspect | Fact | Status |
| --- | --- | --- |
| Full name / brand | Adyen N.V. (brand "Adyen"); Amsterdam, Netherlands | ✅ (DNB register; company) |
| Founded | 2006, Amsterdam | ✅ (Wikipedia; adyen.com/about/team; multiple) |
| Co-founders | Pieter van der Does and Arnout Schuijff (both ex-Bibit) | ✅ (Wikipedia; Forbes; team page for van der Does) |
| Legal form | Naamloze vennootschap (N.V.); Dutch credit institution ("Bank – CI" in the DNB register); KvK 34259528; LEI 724500973ODKK3IFQ447; DNB relation F0001 | ✅ (DNB public register) |
| Listing | Euronext Amsterdam since June 13, 2018; ticker ADYEN; ISIN NL0012969182; AEX component | ✅ (Euronext press release; Adyen press release) |
| IPO | Priced €240/share; market cap €7.1bn; total offering €849m; shares ~doubled on debut (~$17bn value per Reuters) | ✅ (Adyen; Euronext; Reuters) |
| Banking licences | EU (Dutch, DNB) 2017; US Federal Foreign Branch (OCC) 2021; UK bank authorisation (PRA/FCA) September 2023 | ✅ (Adyen press releases) |
| Singapore (MAS) | Adyen Singapore Pte. Ltd. — Major Payment Institution under the Payment Services Act; cross-border money transfer from January 28, 2020; expanded May 1, 2021 to merchant acquisition and domestic money transfer (first global payments provider licensed for merchant acquisition in Singapore) | ✅ (Adyen press release May 4, 2021; MAS register entry 229387) |
| FY2025 processed volume | €1,394.3bn (+8% reported incl. a single large-volume customer; +21% excluding) | ⚠ company-reported (regulated release) |
| H1 2026 processed volume | €803.8bn (+24% YoY) | ⚠ company-reported (regulated release) |
| H1 2026 net revenue | €1,302.9m (+19% YoY; +21% constant currency) | ⚠ company-reported (regulated release) |
| FY2025 net revenue | €2,364.2m (+21% constant currency) | ⚠ company-reported (regulated release) |
| Take rate | ~17 bps blended (17.1 bps H2 2025 vs 16.2 bps H2 2024) | ⚠ company-reported |
| Profitability | Profitable since 2011 | ✅ (Recode 2017, via Wikipedia) |
| Workforce | 4,345 (2024 annual report); 4,771 at end-2025 | ⚠ company-reported |
| Funding (verified rounds) | 2011: ~€16m Series A led by Index Ventures; Dec 2014: $250m led by General Atlantic (with Temasek, Index Ventures, Felicis) at ~$1.5bn valuation; Sept 2015: ICONIQ-led round at $2.3bn valuation | ✅/⚠ (2014 round multi-primary; 2011 round secondary-sourced) |
| KKR investment | No evidence found in any primary or reputable secondary source | ❌ (believed by the research brief; not found) |
| First acquisitions | Talon.One (loyalty/promotions) and Orb (billing) — announced April/June 2026, closed July 1, 2026 | ✅ (Adyen H1 2026 release; Adyen press; Wikipedia) |

### 1.3 Why This Matters to a Bank

Adyen sits at the intersection of three things a bank must understand. **First**, it is the purest large-scale expression of the *unbundled-then-rebundled* acquiring model: where banks historically bundled acquiring with treasury and lending, Adyen rebuilt the merchant stack on a single licensed platform and then re-attached banking functions (settlement, accounts, issuing, capital) — but as platform services rather than as a relationship bank. A commercial bank meeting an Adyen-type merchant therefore meets a client that *is itself a regulated credit institution*, which changes the onboarding, counterparty-credit, and monitoring conversation entirely. **Second**, Adyen is a *platform-of-platforms*: it processes for marketplaces and payment facilitators that in turn onboard thousands of sub-merchants, so a bank analysing Adyen-style flow must reason about a multi-tier merchant chain — exactly the structure the [MAS guide](mas_regulations_guidelines_guide.md) and the [ancillary-revenue guide](../management/ancillary_revenue_products_guide.md) analyse from the Singapore and economics angles respectively. **Third**, Adyen is a *licensing weathervane*: its path — sponsor-bank dependence, to payment-institution-style operation, to a 2017 Dutch banking licence, to a 2021 US branch and a 2023 UK authorisation, to a 2020–2021 Singapore MAS Major Payment Institution licence — is the documented route map for how a payments platform buys regulatory independence, and its compliance record (including the Dutch regulator's intense Wwft enforcement climate of 2025–2026, §8.5) shows what that independence costs.

### 1.4 The Evidence Base at a Glance

Every factual claim in this guide traces to one of five evidence classes, and the claims audit (§12) records which class supports each claim:

| Evidence class | Examples used in this guide | How it is treated |
| --- | --- | --- |
| Company press releases and pages (adyen.com) | H1 2026 results (Aug 13, 2026); FY2025 results (Feb 2026); "Adyen granted US branch license" (Jun 2021); "Adyen granted UK banking authorization" (Sep 2023); Singapore MPI release (May 2021); Knowledge Hub "Adyen's Banking License" (Apr 2024); about/team page (extracted Sep 2026); licences/emea page | ✅ where the fact is the release's own content; ⚠ where the release reports company metrics or claims customers |
| Official documentation (docs.adyen.com) | Adyen for Platforms model docs; Payouts docs; developer portal | ✅ for product existence and scope as documented |
| Regulator registers and releases | DNB public register entry for Adyen N.V. (Bank – CI, KvK 34259528, F0001); MAS Financial Institutions Directory entry 229387 (Adyen Singapore Pte. Ltd., Major Payment Institution); DNB enforcement news (Volksbank, Jan 2025); press on bunq and ABN AMRO fines | ✅ for registry-style facts; ✅ for the enforcement events as reported |
| Independent press | Reuters (IPO debut Jun 2018; 2017 pan-European licence; Issuing launch Nov 2019); FT (Sept 2015 $2.3bn valuation); WSJ (Dec 2014 round); TechCrunch (2014); Bloomberg (eBay, Jan 2018); eBay Inc. newsroom (Jan 31, 2018); Euronext press release (Jun 13, 2018); Recode (2017) | ✅ for event existence and deal headlines; ⚠ for derived estimates |
| Aggregators and encyclopedic sources | Wikipedia (Adyen article, retrieved 2026); financecharts; fobi; CB Insights; fxcintel/substack reproductions of Adyen releases | ✅ where they reproduce primary documents; ⚠ where they add their own numbers |

The methodological limitation of this pass is stated once here and not repeated: the Dutch Chamber of Commerce (KvK) register, the full DNB enforcement archive, and several press archives were not directly queryable in this pass, so every finding that would depend on them (notably the question of any DNB fine specifically against Adyen, §8.5) is flagged in §12.4 rather than asserted.

---

## 2. The Company Profile — Founding, Founders, Headquarters, and Legal Form

### 2.1 The Founding Year: 2006

The founding year of **2006** verifies cleanly across primary and independent sources:

- Wikipedia's Adyen article: "**Adyen was founded in 2006** by Pieter van der Does, the current Co-CEO, and Arnout Schuijff. Headquartered in Amsterdam..." (en.wikipedia.org/wiki/Adyen, retrieved September 2026). ✅
- Adyen's own team page: Pieter van der Does "was CCO at Bibit **before co-founding Adyen in 2006**" (adyen.com/about/team, extracted September 2026). ✅
- The same team page adds a corroborating growth anchor: "Since then, Adyen has grown from a startup into a globally operating business, averaging double-digit annual growth **since 2007**." ✅
- Aggregator profiles agree: financecharts records "2006: Adyen founded in Amsterdam, Netherlands by Pieter van der Does, Arnout Schuijff, and co-founders, following their prior work at Bibit Global Payment Services" (financecharts.com, ⚠ secondary). ✅ for 2006 and the two founders.

The believed anchor — "founded 2006" — is therefore confirmed. The company describes 2007 as its first full growth year, which is consistent with a 2006 incorporation and product-build year.

### 2.2 The Co-Founders: Pieter van der Does and Arnout Schuijff

The two co-founders named in the research brief both verify:

- **Pieter van der Does** (born 1969) — co-founder and, as of September 2026, **Co-CEO** of Adyen. Adyen's team page: "With over two decades of experience in the payments industry, Pieter van der Does, was CCO at Bibit before co-founding Adyen in 2006" and "He has earned a degree in Economics from the University of Amsterdam" (adyen.com/about/team). Forbes maintains a profile of him as a Dutch billionaire whose fortune derives from Adyen's post-IPO stock performance (forbes.com/profile/pieter-van-der-does — the Forbes profile pages for both founders note the company's market value "more than doubled since it went public in June 2018"). ✅
- **Arnout Schuijff** — co-founder and long-time CTO of Adyen (Wikipedia names him as co-founder; Forbes carries a matching profile, forbes.com/profile/arnout-schuijff). His technical leadership is the consistent half of the founding story — van der Does commercial, Schuijff technical. ✅ for the co-foundership; the current (2026) exact role title is not restated on the public team page examined, so it is flagged rather than guessed. ⚠

No other co-founder is named consistently in any primary source examined; where sources say "and co-founders" (financecharts), no names are given. The honest finding: **two documented co-founders, van der Does and Schuijff** — any additional founding-team members are not publicly documented as co-founders. ✅/⚠

### 2.3 The Bibit Heritage

The origin story is the same one the founders tell in every venue: Adyen was founded **out of Bibit**, a Dutch payments company, after its sale. Verified fragments:

- Adyen's team page confirms van der Does was **CCO at Bibit** before founding Adyen (adyen.com/about/team). ✅
- Financecharts records the founders as having founded Adyen "following their prior work at **Bibit Global Payment Services**" (financecharts.com, ⚠ secondary).
- Goldman Sachs' "Talks at GS" page on van der Does summarises the canonical narrative: "Adyen has scaled from an Amsterdam startup to a global fintech by building every layer of payments in-house and focusing relentlessly on merchants" (goldmansachs.com, ⚠ promotional summary). ✅ for the in-house-everything framing as company positioning.

The precise corporate history of Bibit — including the widely repeated detail that it was acquired by the Royal Bank of Scotland in 2004 — could **not** be verified at a primary source in this pass and is therefore flagged rather than asserted (§12.4). What is verified is the lineage claim itself: the founders came from a Dutch payments firm called Bibit, and Adyen's thesis — build the payments stack rather than resell someone else's — is the direct inheritance of that experience.

### 2.4 Headquarters and Legal Form: Adyen N.V., Amsterdam

The company is a Dutch **naamloze vennootschap** (public limited company), **Adyen N.V.**, headquartered in **Amsterdam, Netherlands**. Registry-level verification:

- The DNB public register entry (openbaar register, WFTDG register, relation number F0001): "Statutaire naam: **Adyen N.V.** … Statutaire zetel: **AMSTERDAM, NEDERLAND**; **KvK: 34259528**; LEI-code: **724500973ODKK3IFQ447**; Relatienummer DNB: F0001; Type … **Bank – CI**" (dnb.nl/openbaar-register/registerdetailpagina/?registerCode=WFTDG&relationNumber=F0001). ✅ — the KvK number (34259528) is the Dutch Chamber of Commerce registration; "Bank – CI" denotes a credit institution.
- Wikipedia and the company's own releases consistently place the headquarters in Amsterdam. ✅
- Adyen's licences page adds the regulatory summary: "Adyen N.V. is authorised as a credit institution under the supervision of Dutch Central Bank (De Nederlandsche Bank) ('DNB'). This is commonly referred to as a 'banking license' and includes the ability to provide cross-border acquiring, payment and banking services in all EEA countries in accordance with the passporting rules under CRD IV" (adyen.com/licenses/emea). ✅

The often-quoted street address (Simon Carmiggeltstraat 6-50, 1011 DJ Amsterdam) was not re-verified at a primary source in this pass and is noted in §12.4 rather than asserted.

### 2.5 The Company Table

| Aspect | Verified fact | Status |
| --- | --- | --- |
| Legal name | Adyen N.V. | ✅ (DNB register; company releases) |
| Brand | Adyen (adyen.com) | ✅ |
| Founded | 2006, Amsterdam | ✅ (Wikipedia; team page; aggregators) |
| Founders | Pieter van der Does, Arnout Schuijff | ✅ (Wikipedia; Forbes; team page) |
| Corporate form | Dutch N.V. (naamloze vennootschap), credit institution | ✅ (DNB register) |
| KvK / LEI / DNB relation | 34259528 / 724500973ODKK3IFQ447 / F0001 | ✅ (DNB register) |
| HQ | Amsterdam, Netherlands | ✅ (DNB register; Wikipedia) |
| Listing | Euronext Amsterdam: ADYEN (ISIN NL0012969182); AEX index component | ✅ (Euronext; Wikipedia) |
| Listing date | June 13, 2018 | ✅ (Euronext press release; Adyen) |
| Banking supervision | DNB (EU credit institution, licence since 2017); UK: PRA/FCA (2023); US: OCC Federal Foreign Branch (2021) | ✅ (DNB register; Adyen releases) |
| Management Board (Sep 2026) | Pieter van der Does (Co-founder & Co-CEO); Ingo Uytdehaage (Co-CEO); Roelant Prins (CCO); Mariëtte Swart (CRCO); Brooke Nayden (CHRO); Tom Adams (CTO); Hwa Tsao (Interim CFO) | ✅ (adyen.com/about/team, extracted Sep 2026) |
| Workforce | 4,345 (2024 annual report); 4,771 end-2025 | ⚠ company-reported |
| Profitability | Since 2011 | ✅ (Recode 2017) |
| Mission framing | "Financial technology platform of choice … end-to-end payments capabilities, data-driven insights, and financial products in a single global solution" | ✅ (company boilerplate in releases) |

### 2.6 The Footprint Timeline

| Period | Milestone | Source |
| --- | --- | --- |
| 2006 | Founded in Amsterdam by van der Does and Schuijff | Wikipedia; team page ✅ |
| 2007 | "Averaging double-digit annual growth since 2007" | Team page ✅ |
| 2011 | Profitable for the first time; Ingo Uytdehaage joins as CFO | Recode (2017); team page ✅ |
| 2012 | First global offices: San Francisco, Paris, London | Wikipedia ✅ (annual-report-sourced) |
| Dec 2014 | US$250m round led by General Atlantic, with Temasek, Index Ventures, Felicis; ~$1.5bn valuation | Adyen/Index/GA releases; WSJ ✅ |
| Sep 2015 | ICONIQ-led round at $2.3bn valuation ("sixth-largest European unicorn") | FT; Wikipedia ✅ |
| 2016 | Acquiring licence in Brazil via BIN sponsorship | Annual report 2019 (via Wikipedia) ✅ |
| Jun 2017 | European (Dutch) banking licence — "pan-European licence to bypass banks" | Reuters Jun 26, 2017; Adyen Knowledge Hub ✅ |
| Sep 2017 | Direct card acquiring added in Singapore, Hong Kong, Australia, New Zealand | Finextra; Straits Times ✅ |
| Jan 2018 | eBay signs Adyen as primary payments processing partner | eBay Inc. newsroom; Bloomberg ✅ |
| Jun 13, 2018 | IPO on Euronext Amsterdam; €240/share; €7.1bn market cap; shares ~double on debut | Adyen; Euronext; Reuters ✅ |
| 2019 | Offices in Tokyo and Mumbai; Africa expansion; Adyen Issuing launched (Nov 2019) | Adyen press; Reuters ✅ |
| H2 2020 | Android POS devices launched worldwide; Dubai office opened | Annual report/shareholder letter (via Wikipedia); Silicon Canals ✅ |
| Jan 28, 2020 | MAS MPI licence (cross-border money transfer) on the day the PSA came into force | Adyen press (May 4, 2021) ✅ |
| May 1, 2021 | Singapore MPI licence expanded to merchant acquisition and domestic money transfer | Adyen press (May 4, 2021) ✅ |
| Jun 14, 2021 | OCC approves US Federal Foreign Branch (Fed approval May 24, 2021) | Adyen press ✅ |
| 2021 | eBay transitions majority of marketplace customers to Adyen | Wikipedia (Bloomberg-sourced) ✅ |
| Sep 7, 2023 | UK branch granted banking authorisation by PRA and FCA | Adyen press ✅ |
| 2024 | BILL card-issuing partnership; net revenue past €1.9–2.0bn scale (annual-report metrics) | PRNewswire; Wikipedia infobox ⚠ |
| Jul 2025 | Adyen Capital Canada launched (embedded SME lending via platforms) | Fintech.ca; FF News ✅ |
| Feb 2026 | FY2025 results: net revenue €2,364.2m; processed volume €1,394.3bn | Adyen FY2025 release ⚠/✅ |
| Apr–Jun 2026 | Talon.One and Orb acquisitions announced (first in company history) | Adyen press; Wikipedia ✅ |
| Jul 1, 2026 | Talon.One and Orb acquisitions closed; Gayathri Rajan named CPO | Adyen press; Wikipedia ✅ |
| Aug 13, 2026 | H1 2026 results: net revenue €1,302.9m (+19%); processed volume €803.8bn (+24%) | Adyen H1 2026 release ⚠/✅ |
| Sep 1, 2026 | Hwa Tsao assumes full-time interim CFO role (CFO Ethan Tandowsky departed Aug 31, 2026) | Adyen press; Wikipedia ✅ |

---

## 3. Geography and Footprint

### 3.1 Headquarters and the Office Network

Adyen is **headquartered in Amsterdam** (§2.4) and describes itself as operating "with offices around the world" (standard boilerplate in its 2021–2026 press releases). ✅ The commonly cited figure of offices in twenty-three countries comes from Wikipedia's article text ("the company employs over 4,300 people in offices in twenty-three countries," citing a February 2025 Reuters article on the FY2024 results) — credible but treated as ⚠ secondary, because Adyen's own releases in this pass do not state a current office-country count. The company's country-manager roster (§3.2) is the better primary evidence of footprint: as of September 2026 Adyen publicly lists country managers or general managers for at least the following markets: India, Canada, Germany, China, Mexico, Australia & New Zealand, Spain, Belgium & Netherlands, France, the Middle East, Nordics & Baltics, Japan, South-East Asia & Hong Kong, Poland & Czech Republic, Italy, the UK, and Malaysia (adyen.com/about/team, extracted September 2026). ✅

The 2012 global expansion (San Francisco, Paris, London), the 2019 additions (Tokyo, Mumbai), and the 2020 Dubai office are the datable office milestones (Wikipedia, annual-report-sourced ✅; Silicon Canals for Dubai ✅). The regional-leadership layer (presidents for EMEA, APAC, and Latin America) and a country-manager layer for the largest markets are the current operating structure.

### 3.2 Regional Leadership and Country Managers (September 2026)

The team page (adyen.com/about/team, extracted September 2026) documents the leadership structure used in this guide: a Management Board (Pieter van der Does, Co-founder & Co-CEO; Ingo Uytdehaage, Co-CEO — CFO from 2011, appointed co-CEO in 2023; Roelant Prins, CCO; Mariëtte Swart, Chief Risk & Compliance Officer; Brooke Nayden, CHRO; Tom Adams, CTO — ex-Cash App/Block; Hwa Tsao, Interim CFO), a Global Leadership tier (Alexa von Bismarck, President EMEA; Warren Hayashi, President APAC — marked "outgoing"; Gary Yang, SVP Global Account Management and Partnerships, APAC; Thais Fischberg, President Latin America), and the country-manager tier listed in §3.1. ✅ Notable for a bank reader: the chief-risk-and-compliance function sits on the Management Board (Mariëtte Swart was Adyen's General Counsel and, in the 2021 Singapore release, "Chief Legal & Compliance Officer" — the title has since evolved to Chief Risk & Compliance Officer), reflecting the regulatory weight a banking-licensed payments platform must carry.

### 3.3 The Processing Footprint — Local Acquiring and Licence Passporting

What is verifiable about Adyen's *processing* footprint (as opposed to its office footprint) is the licence-and-scheme layer, because that is what Adyen publishes:

- **Local acquiring** is Adyen's core footprint claim: direct connections to the card schemes in the markets where it holds acquiring rights, rather than routing through a sponsor acquirer per market. The Knowledge Hub article states it plainly: "When we became a global acquirer, we directly connected our customers to card schemes" (adyen.com/knowledge-hub/adyen-banking-license, April 2024). ✅ as company framing.
- The 2016 Brazil BIN-sponsorship entry, the September 2017 expansion of direct acquiring to Singapore, Hong Kong, Australia, and New Zealand, and the 2021 US Federal Foreign Branch and Singapore MPI expansions are the datable footprint events (Wikipedia/annual-report ✅; Finextra ✅; Adyen press ✅).
- The EEA dimension is passporting under CRD IV from the Dutch licence: "cross-border acquiring, payment and banking services in all EEA countries in accordance with the passporting rules under CRD IV" (adyen.com/licenses/emea). ✅
- The H1 2026 release adds two 2026 footprint items: "Secured direct access to France's domestic interbank clearing system, and obtained a Retail Payment Services license from the Central Bank of the UAE to expand localized processing capabilities" (Adyen H1 2026 release). ✅ as company-reported.

The exact number of markets in which Adyen holds direct acquiring licences, and the map of which merchant categories run through local versus cross-border acquiring, are not published in the sources examined and are flagged in §12.4. ⚠

---

## 4. The Mission and Business Model — the Single-Platform Model

### 4.1 The Mission Statement

Adyen does not publish a single sentence called "our mission" on the pages examined in this pass, but its standard boilerplate — carried in every press release from 2021 through 2026 — functions as the mission statement and is verified verbatim:

> "Adyen (ADYEN:AMS) is the financial technology platform of choice for leading companies. By providing **end-to-end payments capabilities, data-driven insights, and financial products in a single global solution**, Adyen helps businesses achieve their ambitions faster." ✅

(From the H1 2026 results release, the UK-authorisation release, the US-branch release, and the Singapore release — adyen.com/press-and-media.)

The **single-platform** claim — the phrase in this guide's title — appears explicitly in Adyen's own materials in at least three verified places: the June 2021 US-branch release ("Benefits include increased operational scalability **via Adyen's single platform**"), the April 2024 Knowledge Hub article on the banking licence (the whole article is an argument that one platform should own gateway, risk, processing, acquiring, and settlement), and the August 2026 H1 release ("**Intelligent Money Movement**, bringing enterprise payments, liquidity management, and payouts together on a single platform"). ✅ The corollary company claims — one contract, one integration, one data model, one reporting view across channels — are marketing-consistent across those releases. ⚠ where quantified.

### 4.2 The Business Model — the Single-Platform Payments Model

The business model is best read as a vertical-integration argument. The verified elements:

- **One integration, many rails.** A merchant integrates once with Adyen and can then accept cards, wallets, bank transfers, and local methods online, in-app, and in store, and add payouts and financial products without new contracts — the "single platform" value proposition. ✅ (company framing, all releases).
- **The acquirer role.** Adyen's 2017 banking licence made it an acquiring bank in its own right: "In 2017, we received our first banking license in the EU… Before this, we did what many other payment service providers do: we used sponsor banks to facilitate money movement and provide compliance oversight" (Knowledge Hub, April 2024). ✅ The company's own framing of the pre-licence era (sponsor-bank dependence) is consistent with the industry's standard structure for PSPs (cross-ref the [ancillary-revenue guide](../management/ancillary_revenue_products_guide.md) §8 for the acquiring-side economics this guide does not re-derive).
- **The settlement role.** The banking licence added settlement to the stack: "Our banking license connects our customers directly to real-time banking and card schemes… payments are processed up to three days faster than the industry standard" (Knowledge Hub, April 2024). ✅ as company claim.
- **The platform-of-platforms role.** Through Adyen for Platforms (§6.1), Adyen does not merely acquire for merchants; it acquires *for platforms that onboard merchants* — marketplaces, gig-economy apps, crowdfunding sites, and payment facilitators — processing on behalf of their sub-merchants and paying them out. This is the "platform of platforms" reading of the model, and it is the layer a bank meets when a fintech client says "our payments run on Adyen." ✅ (docs.adyen.com/adyen-for-platforms-model).

### 4.3 The Revenue Model — Processing Fees, Take Rate, and the Platform of Platforms

Adyen's revenue model is the classic acquirer/PSP model with a banking-licence extension, and its disclosed economics are:

- **Net revenue** — the metric Adyen reports (rather than gross processed volume): processing fees (a blended fee per transaction, often quoted as a percentage-plus-fixed), plus scheme, FX, and value-added service fees, plus financial-product income (interest-like and card income from Capital, Accounts, Issuing). The blended outcome is the **take rate**: net revenue divided by processed volume, reported at **16.2 bps in H2 2024 and 17.1 bps in H2 2025** (company-reported, H2 2025 release as reproduced by analyst commentary — ⚠/✅). FY2025 net revenue of €2,364.2m on €1,394.3bn volume implies a full-year take rate of roughly 17 bps — consistent with the half-year disclosures. ⚠ company-reported.
- **The 1.4% claim.** The Knowledge Hub article claims merchants gain "additional income of 1.4%" from direct scheme connection — an uplift (lower fees, higher conversion) attributed to Adyen's local-acquiring model. ⚠ company claim, not independently verified.
- **Volume growth quality.** Adyen's reported growth is deliberately split around "a single large-volume customer" (widely reported in the press as eBay's gradual in-housing of payments processing — see §7.5): FY2025 processed volume grew **8% reported but 21% excluding that customer**; H1 2025 grew 5% reported but 23% excluding it (Adyen releases). The "excl. single large-volume customer" framing is Adyen's own, published in every release since 2024. ⚠/✅ — the metric is company-defined; the identity of the customer is not named by Adyen.
- **Financial-products income.** Since the banking licence, revenue increasingly includes the embedded-finance stack — the UK authorisation release (September 2023) describes the "recently launched embedded financial product suite" (bank accounts, cards, Capital) that platforms can resell to SMB users; the economics of such programs (interchange, interest, fees) are the card-program economics of the [ancillary-revenue guide](../management/ancillary_revenue_products_guide.md) §10, cross-referenced, not re-derived here. ✅ for product existence; ⚠ for the revenue split, which Adyen does not segment publicly in the sources examined.

### 4.4 The Market Context

Adyen's own market framing, verified across its materials: enterprises increasingly demand one platform that works across online and in-store channels in many countries (its "unified commerce" thesis), platforms demand embedded finance to monetise beyond the transaction (the "$110 billion market opportunity" the Knowledge Hub article cites for embedded finance in the US, UK, and Europe — ⚠ company-cited research), and SMBs on platforms demand speed ("75% of SMBs say they need same-day settlement or faster" — ⚠ company-cited survey). The structural context a bank should add is the industry one: the acquiring market's economics (interchange, scheme fees, processing cost, fraud cost) are documented in the [ancillary-revenue guide](../management/ancillary_revenue_products_guide.md), and the clearing/settlement mechanics that the banking licence lets Adyen control directly are documented in the [Payment Rails](payment_rails_guide.md) guide §5. Adyen's strategy — build or buy the whole stack, take the licence, sell the platform — is the 2026 endpoint of the vertical-integration arc this guide traces from §2 to §7.

---

## 5. The Products — the Payments Platform

### 5.1 Online Payments — Gateway, Processing, and Checkout

Adyen's online-payments product is the documented core: a payment gateway and processing platform that merchants integrate server-to-server, with hosted checkout components, and which connects directly to the card schemes and to local payment methods. Verified anchors:

- Adyen's Knowledge Hub defines the gateway layer explicitly ("the multi-channel gateway," "processing," "global acquiring" as components of the one platform — Knowledge Hub, April 2024). ✅
- The public developer documentation (docs.adyen.com, live and extracted for the platform model and payouts pages) documents the API surface — Checkout APIs, payment-methods management, reporting — consistent with the single-API claim; the specific endpoint catalogue is not re-derived in this guide (cross-ref the [middleware guide](../technology/enterprise_middleware_integration_platform_guide.md) §3.7 for API-gateway themes). ✅ for existence; ⚠ for depth claims.
- The company's method coverage claim has evolved in the releases: 2018-era materials cited "more than 150 currencies and over 200 payment methods" (eBay-era press); 2021 materials say "consumers' globally preferred payment methods"; current pages do not state a fixed count in the extracts examined. The count is therefore time-varying and ⚠; the *capability* (cards + wallets + local methods, online and in-app) is ✅.

### 5.2 In-Person Payments — Terminals and Point-of-Sale

Adyen's in-person product line is verified at the milestone and product level:

- **Android POS terminals, launched worldwide in H2 2020** (Adyen shareholder letter/annual-report via Wikipedia ✅; the H2 2020 shareholder letter is cited by Wikipedia for the global launch).
- The **Adyen Terminal API** and terminal-management surface are documented in the developer portal (docs.adyen.com — terminal API pages exist under the docs domain; ⚠ specifics not extracted this pass).
- The company's 2026 partner motion adds **Toast**: "Expanded global partnership with Toast into the U.S. following success across international markets" (H1 2026 release) — a restaurant-POS ecosystem partnership. ✅ as company-reported.
- Adyen also connects its platform to third-party terminal portfolios (the "point of sale" solutions the Wikipedia infobox lists); the current certified-hardware catalogue is not enumerated in the sources examined. ⚠

The unified-commerce point — the same merchant account, same reporting, same risk profile across online and in-store — is the product thesis (Wikipedia/PYMNTS coverage of the embedded-finance and unified-commerce strategy ✅/⚠; Adyen releases ✅ as company framing).

### 5.3 Unified Commerce — One Platform Across Channels

"Unified commerce" is Adyen's term for running online, in-person, and in-app payments on one platform with one view of the customer. Verified at the framing level: the Wikipedia article's products section ("expanded its core business… into being a full-stack financial technology platform… including developing Unified Commerce solutions to merge online and in-store payments") and the company's own channel language across releases ("frictionless payments across online, mobile, and in-store channels" — 2021 releases; "omnichannel loyalty" in the Talon.One acquisition framing — 2026). ✅ for the concept as company strategy; ⚠ for any quantified unified-commerce benefit (Adyen has historically cited higher conversion and lower fraud for unified merchants, but no current figure was verified in this pass).

### 5.4 Cross-Border and Local Payment Methods

The cross-border engine is the combination of local acquiring (§3.3), local payment methods (the 2018-era "over 200 payment methods" claim, ⚠ time-varying), and — since the banking licences — local settlement and payouts ("direct access to France's domestic interbank clearing system," H1 2026 ✅ as company-reported). The mechanics of how a cross-border card transaction clears and settles across schemes and local rails are documented in the [Payment Rails](payment_rails_guide.md) guide §5 and are cross-referenced, not re-derived, here.

---

## 6. The Products — the Financial-Products Suite

### 6.1 Adyen for Platforms — Marketplace Payments and Payouts

**Adyen for Platforms** is the verified, current product name (docs.adyen.com/adyen-for-platforms-model; adyen.com/en_GB/industries/platforms):

> "Onboard users of your platform or marketplace, process payments for them, and pay out their funds. Adyen for Platforms is an end-to-end payment solution for peer-to-peer marketplaces, on-demand services, crowdfunding platforms, or any other platform business model." ✅ (docs.adyen.com/adyen-for-platforms-model, extracted September 2026)

The documented mechanics: hosted onboarding of sub-merchants, payments processed by the platform's Adyen account, and **split settlements / payouts** of funds to sub-merchants' bank accounts (docs.adyen.com/payouts distinguishes "Payouts combined with Payments" — paying a beneficiary who is not onboarded — from "Payouts as part of Adyen for Platforms" — paying out a sub-merchant). ✅ These are the marketplace-payout mechanics the Cymbal Bank worked example in §11.4 builds on.

### 6.2 Adyen Issuing

**Adyen Issuing** — virtual and physical card issuing for merchants and platforms — was **launched in November 2019** (Reuters, November 14, 2019: "Adyen keeps focus on organic growth as it launches cards product" ✅) and remains a current product (adyen.com/issuing; the Knowledge Hub describes it as enabling businesses to "create and manage their card program"). The 2024 **BILL partnership** for "advanced card issuing capabilities" (PRNewswire via Wikipedia ✅) shows the issuing product being sold into financial-operations platforms. Card-program economics are cross-referenced to the [ancillary-revenue guide](../management/ancillary_revenue_products_guide.md) §10.

### 6.3 Adyen Capital

**Adyen Capital** — embedded business financing/cash advances to merchants, offered through platforms — is verified at adyen.com/capital and by the 2023 UK release ("Capital, which provides SMBs with direct access to cash advances when they need them") and the **July 2025 Adyen Capital Canada launch** for platform customers (Fintech.ca/FF News coverage of the launch ✅). The model is underwriting against the merchant's own payment data on the platform — the data advantage of running processing and lending on one stack. ⚠ where Adyen's own materials describe lending decisions; the credit product's funding and risk-retention structure are not disclosed in the sources examined (⚠, §12.4).

### 6.4 Adyen Accounts, Payouts, and Embedded Financial Products

The wider financial-products roster, verified at adyen.com product pages and in releases:

- **Adyen Accounts** (adyen.com/accounts): "embedded bank accounts" for platforms — "you can make it easy for sellers to buy and sell on the platform and make funds instantly available in the seller's account" (Knowledge Hub, April 2024; product page live). ✅
- **Payouts** (adyen.com/en_GB/payouts; docs.adyen.com/payouts): disbursing funds to suppliers, vendors, staff, or sub-merchants, consolidated "regardless of where they were processed" (Knowledge Hub). ✅
- The **"Embedded Financial Products" (EFP) suite** as a named bundle — Accounts + Issuing + Capital — is verified in the September 2023 UK release and in PYMNTS' April 2025 coverage of Adyen's embedded-finance demand. ✅
- The UK authorisation release ties the suite to the licence: the licence "allows Adyen to continue to offer its recently launched embedded financial product suite in the UK" — a useful illustration of why a payments platform takes banking licences at all (§8). ✅

### 6.5 The 2026 Product Surface — Agentic, Personalize, Talon.One, and Orb

The H1 2026 release (August 13, 2026) is the primary source for the current product frontier ✅ (all as company-reported):

- **Adyen Agentic** — "enabling enterprise merchants to securely process payments across AI agent protocols" — launched H1 2026; Adyen also "joined the x402 Foundation and Open Standard, driving open protocols for HTTP payments in agentic commerce." ⚠/✅ company-reported; the agentic-payments category is nascent and Adyen's role in the x402 standard is a company claim.
- **Intelligent Money Movement** — "bringing enterprise payments, liquidity management, and payouts together on a single platform" (note the explicit single-platform language). ✅ company-reported.
- **Adyen Uplift and Dynamic Identification** — conversion optimisation ("increased customer conversion by 0.9 percentage points on average by the end of H1" — ⚠ company metric).
- **Adyen Personalize** — "leveraging our unified network dataset to help merchants deliver real-time, tailored shopping experiences." ⚠ company-reported.
- **Talon.One and Orb** — acquisitions announced April and June 2026, closed **July 1, 2026** ("subsequent to H1" per the release; Wikipedia and Adyen's closing release confirm they are "the first acquisitions in the company's history"). Talon.One brings omnichannel loyalty/promotions; Orb brings usage-based billing — both being embedded onto the platform. ✅

### 6.6 The Product-Surface Map

| Layer | Verified product (2026) | Evidence |
| --- | --- | --- |
| Online payments | Gateway + processing + checkout components (single API) | docs.adyen.com; Knowledge Hub ✅ |
| In-person | Adyen Android POS terminals (global since H2 2020); Terminal API; Toast partnership (US, 2026) | Annual report/shareholder letter; H1 2026 release ✅/⚠ |
| Unified commerce | One platform across online/in-store/in-app | Company framing; Wikipedia ✅/⚠ |
| Platform payments | Adyen for Platforms (onboarding, processing, payouts, split settlements) | docs.adyen.com/adyen-for-platforms-model ✅ |
| Issuing | Adyen Issuing (virtual/physical cards; launched Nov 2019; BILL partnership 2024) | Reuters 2019; adyen.com/issuing ✅ |
| Capital | Embedded cash advances (incl. Adyen Capital Canada, Jul 2025) | adyen.com/capital; releases ✅ |
| Accounts | Embedded bank accounts for platforms | adyen.com/accounts ✅ |
| Payouts | Enterprise payouts / disbursements | adyen.com/payouts; docs.adyen.com/payouts ✅ |
| Risk | Adyen Protect (risk management) | adyen.com/protect (linked from Knowledge Hub) ✅ |
| 2026 additions | Adyen Agentic; Intelligent Money Movement; Personalize; Uplift/Dynamic Identification; Talon.One; Orb | H1 2026 release ✅/⚠ |

One naming caution for the reader: the research brief for this guide suggested a product called "revenue recovery" and a "revenue platform" framing. **No current Adyen product by the name "revenue recovery" was found** at adyen.com, docs.adyen.com, or in any release examined in this pass; the closest verified concepts are the conversion/revenue-optimisation features (Adyen Uplift, Dynamic Identification) and the financial-products suite. The finding is recorded as ❌-not-found in §12.3.

---

## 7. Growth and Market Position — Funding, IPO, Volumes, and Customers

### 7.1 The Funding History — 2011 to 2015

Adyen raised relatively little private capital for a company of its scale, and the verified record is:

- **2011 — Series A of ~€16 million, led by Index Ventures.** Two independent secondary sources state this round (financecharts: "In 2011, the company raised €16 million in a Series A round led by Index Ventures"; b2bleadnavigator agrees on the lead and amount, adding Felicis Ventures participation). ⚠ secondary-sourced in this pass — no primary 2011 press release was retrieved — but consistent across sources. (Note: the aggregator tables on fobi/CB Insights date an Index Ventures "Series A" to June 2014 and omit the 2011 round entirely — the cap-table dating varies by aggregator; the 2011 Index-led round is the best-supported reading.)
- **December 2014 — US$250 million led by General Atlantic**, with participation from Temasek, Index Ventures, and Felicis Ventures. Verified at four primary/secondary sources: Adyen's own release ("Adyen raises $250 million in funding to accelerate growth of its global payments platform"), Index Ventures' reprint of the same release, General Atlantic's media page, and WSJ's contemporaneous coverage ("Payment Startup Adyen Raises $250 Million at $1.5 Billion Valuation"). ✅ Valuation ~$1.5bn (WSJ; fobi table).
- **September 2015 — ICONIQ Capital round at a $2.3 billion valuation.** The FT headline is the anchor: "Payments company Adyen valued at $2.3bn after Iconiq injection" (ft.com, September 2015 — the round was raised from an investment vehicle representing Silicon Valley principals, i.e. ICONIQ). ✅ CB Insights records the September 2015 valuation at $2,300m; Wikipedia notes the $2.3bn valuation made Adyen the sixth-largest European unicorn at the time. ✅
- **Profitability.** The company has been profitable since 2011 (Recode, April 2017, reproduced via Wikipedia) — i.e. it raised growth capital while already profitable, which is the structural reason its private rounds were modest relative to peers. ✅

### 7.2 The KKR Question

The research brief believed a 2011 round involved "KKR and/or General Atlantic." The verified record corrects this: **no KKR investment in Adyen appears in any primary or secondary source examined in this pass** — not in the company's funding releases, not in the aggregator cap tables (fobi, CB Insights, ZoomInfo, Caplight), not in press coverage of the 2011, 2014, or 2015 rounds. General Atlantic *is* verified, but as the **December 2014** lead, not a 2011 investor. The "KKR" element of the belief is recorded as ❌-not-found in §12.3. The 2011 round's verified lead is Index Ventures (§7.1).

### 7.3 The IPO — June 13, 2018, Euronext Amsterdam

The IPO verifies cleanly at primary sources:

- Adyen's own release: "Adyen's IPO priced at **€240 per share with a market capitalization of €7.1 billion**, marking its listing on Euronext Amsterdam" (adyen.com/press-and-media/ipo-priced-240-euro-per-share). ✅
- Euronext's release (June 13, 2018): "Adyen N.V. (ticker symbol: **ADYEN**)… began trading today on Euronext Amsterdam. The listing follows the company's Initial Public Offering (IPO), with a **total offering of €849 million**. Based on the offering price set at €240 per share, market capitalisation…" ✅
- Reuters on the debut: "Shares in Adyen… surged more than 70 percent at the start of trading" and "saw its value double to **$17 billion** in a market debut reminiscent of the 1999 tech bubble" (Reuters, June 13, 2018). ✅ — the first-day close roughly doubled the IPO price.
- Wikipedia (2018 sources): the IPO took place on June 13, 2018; the company had announced the listing in May 2018. ✅ ISIN NL0012969182; AEX index component (Euronext; Wikipedia). ✅

Post-IPO valuation history is press territory and volatile — the shares more than doubled in value in the years after the IPO (Forbes founder profiles), fell sharply around 2023 results announcements, and have traded widely since; no September-2026 market-cap figure was verified in this pass, so none is asserted (⚠, §12.4).

### 7.4 Processed Volume and Financial Scale (2025–2026)

All figures in this subsection are **company-reported through regulated investor releases** (⚠-with-source: they are the company's own numbers, published under its disclosure obligations, reproduced here from Adyen's press releases):

- **FY2025 (published February 2026):** net revenue **€2,364.2m** (+21% constant-currency); processed volume **€1,394.3bn** (+8% reported; **+21% excluding a single large-volume customer**); point-of-sale volume **€311bn** (+34% for the full year); take rate 17.1bps in H2 2025; workforce **4,771 at end-2025**; 2026 net-revenue-growth guidance of 20–22% constant-currency (as initially guided).
- **H1 2026 (published August 13, 2026):** net revenue **€1,302.9m** (+19% YoY; +21% constant-currency); processed volume **€803.8bn** (+24% YoY); EBITDA **€641.5m** with margin **49%** (50% excluding one-time transaction costs); free-cash-flow conversion 86%; CapEx ~5% of net revenue in H1. FY2026 guidance revised to **21–23% net-revenue growth constant-currency including the Talon.One/Orb contributions**, EBITDA margin "above 55% by 2028" (medium-term target), and CapEx ~7% of net revenue in 2026 (data-centre investment pulled forward from 2027).
- **Historical anchors:** revenue above €1.3bn in 2022 (company, via Wikipedia); Q3 2025 net revenue €598.4m (+20%) (Reuters, October 2025); a record **US$43bn processed over the 2025 Black Friday/Cyber Monday weekend** (company claim, December 2025, via Retail Insider — ⚠).

### 7.5 The Named Customers — Verified and Company-Claimed

Named-customer claims are the highest-risk facts in any payments-company profile, so this guide separates what is press-verified from what is only company-claimed:

**Press/primary-verified relationships (✅/⚠ as marked):**

- **eBay** — the flagship: eBay's own newsroom announced on January 31, 2018 that "eBay has signed an agreement with Adyen… to become its primary payments processing partner" (ebayinc.com/stories/news/ebay-to-intermediate-payments-on-its-marketplace-platform), with Bloomberg covering the same day ("EBay Rises to Record High on Shift to Adyen; PayPal Tumbles"). eBay began intermediation in North America in H2 2018 and transitioned a majority of marketplace customers to Adyen in 2021. ✅ Adyen's own releases since 2024 refer only to "a single large-volume customer" whose volumes drag the reported growth rate — widely reported (but never confirmed by Adyen) to be eBay as it gradually moved processing in-house; the identity is ⚠ press-inferred, and the §12.4 list records it as such.
- **Netflix, Facebook/Meta, Uber** — Reuters' IPO-debut coverage states Adyen "processes payments for Netflix, Facebook and eBay" (June 2018), and Business Insider's pre-IPO coverage describes a company "backed by Mark Zuckerberg and used by Uber." ✅/⚠ — the payment relationships are press-verified; their current (2026) status is not.
- **Spotify, Microsoft, H&M, L'Oréal, Cathay Pacific, Grab, Klook, Singapore Airlines** — appear in Adyen's own "About" boilerplate across 2021–2026 releases ("works with the likes of Meta, Uber, H&M, eBay, and Microsoft"; "serves customers including Facebook, Uber, Spotify, L'Oréal, Cathay Pacific, Grab, Klook and Singapore Airlines"). ⚠ company-claimed; plausible and in several cases press-corroborated historically, but not re-verified per-name in this pass.
- **2026 wins named in the H1 2026 release** — Aritzia, OpenAI, Xiaomi, and GOV.UK Pay (public sector). ⚠ company-reported.
- **Customers named in the research brief but not verified in this pass** — McDonald's, for example: no primary or press source was retrieved in this pass confirming a McDonald's–Adyen relationship; recorded as ⚠-not-found in §12.3 rather than asserted. (Absence of evidence in a bounded research pass is not proof of absence.)

### 7.6 The Market Position

Adyen's market position, as far as it can be verified without analyst reports (⚠ where estimated): it is one of the largest European payments companies by market capitalisation and processed volume; its €1.4 trillion annual processed volume (FY2025, company-reported) places it in the top tier of global merchant acquirers alongside the US giants, while its net-revenue growth (19–21%) and 49–50% EBITDA margins (H1 2026) are the metrics the market watches. The company's own framing of its position is enterprise-and-platform focused ("the financial technology platform of choice for leading companies"). No independent market-share figure (e.g., share of global card-acquiring volume) was verified in this pass and none is asserted — see §12.4.

---

## 8. Licensing and Compliance

### 8.1 The Dutch Banking Licence — DNB and ECB Supervision

The research brief asked to verify Adyen's Dutch banking licence "around 2015-2017" and its current supervisory posture. The verified record:

- **2017**: Adyen received its first (EU/Dutch) banking licence. Three primary/independent anchors: (1) Adyen's Knowledge Hub — "In 2017, we received our first banking license in the EU" (April 2024); (2) Adyen's US-branch release — "the European banking license (obtained in 2017)"; (3) Reuters, June 26, 2017 — "Dutch payments processor takes pan-European licence to bypass banks" (Reuters article ID USKBN19E1Y2, cited by Wikipedia). ✅ The precise licence-grant month within 2017 was not pinned in this pass (⚠, §12.4).
- **Current status**: the DNB public register lists **Adyen N.V., Amsterdam, KvK 34259528, LEI 724500973ODKK3IFQ447, relation number F0001, type "Bank – CI"** (credit institution) (dnb.nl public register, extracted September 2026). ✅
- **Passporting**: "authorised as a credit institution under the supervision of [DNB]… includes the ability to provide cross-border acquiring, payment and banking services in all EEA countries in accordance with the passporting rules under CRD IV" (adyen.com/licenses/emea). ✅
- **ECB/SSM**: as a euro-area credit institution, Adyen sits under the Single Supervisory Mechanism umbrella; a secondary source states it is "under DNB and ECB Single Supervisory Mechanism supervision" (europealternatives.com ⚠). The precise significance classification (direct ECB vs delegated DNB day-to-day supervision of a less-significant institution) was not verified at an ECB/SSM primary source in this pass — flagged in §12.4 rather than asserted. ⚠

The licence's strategic meaning is stated in Adyen's own words: it replaced sponsor-bank dependence with direct settlement ("we used sponsor banks to facilitate money movement and provide compliance oversight" before the licence — Knowledge Hub) and put Adyen under "direct regulatory oversight from banking authorities" (same article). ✅ as company framing.

### 8.2 The UK Authorisation — PRA and FCA (2023)

Verified at Adyen's own release (adyen.com/press-and-media/adyen-granted-uk-banking-authorization, London, September 7, 2023): Adyen's UK branch "has been granted authorization as a UK bank… granted by the Prudential Regulation Authority (PRA) and the Financial Conduct Authority (FCA) under Part 4A of FSMA 2000," allowing it to continue UK operations post-Brexit beyond the Temporary Permissions Regime and to offer its embedded financial-product suite (Accounts, Issuing, Capital) in the UK. ✅

### 8.3 The US Federal Foreign Branch — OCC (2021)

Verified at Adyen's own release (adyen.com/press-and-media/adyen-granted-us-branch-license, San Francisco, June 14, 2021): the US Office of the Comptroller of the Currency approved Adyen's application to establish a **Federal Foreign Branch in San Francisco**; the Federal Reserve approved the application on May 24, 2021. The release frames the licence as letting Adyen run US activities "in line with those conducted in Europe under its European banking license (obtained in 2017)." ✅

### 8.4 The Singapore Angle — MAS Major Payment Institution (Cross-Referenced)

The Singapore licensing picture is verified at both the regulator register and the company release — this is the cleanest licensing verification in the whole guide:

- **MAS Financial Institutions Directory, entry 229387: "ADYEN SINGAPORE PTE. LTD." — Licence Type/Status: Major Payment Institution** (eservices.mas.gov.sg/fid, extracted September 2026). ✅
- **Adyen's release** (Singapore, May 4, 2021): Adyen Singapore Pte Ltd received MAS approval to carry out **merchant acquisition services** under the Payment Services Act 2019 (PSA); effective **May 1, 2021**, its Major Payment Institution licence was "expanded to include merchant acquisition and domestic money transfer services, on top of the previously licensed activity of cross-border money transfer service"; Adyen Singapore is "**the first global payments provider to be licensed for the newly regulated activity of merchant acquisition service under the PSA**." The release also records that the PSA came into force on **January 28, 2020** and Adyen Singapore received its MPI licence for cross-border money transfer "on the same day," with safeguarding arrangements in place for customer funds. ✅
- The historical layer: Adyen expanded direct card acquiring into Singapore in September 2017 under the pre-PSA regime (Finextra, September 7, 2017 ✅) — so its Singapore presence predates the PSA licensing era by three years.

The PSA/MPI regime itself — the Act, the licence tiers, and the transition from the old Money-Changing and Remittance Businesses Act — is documented in the [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) guide (§2.4 on the Payment Services Act 2019; the MPI tier and its obligations) and is cross-referenced, not re-derived, here. For the repository's Cymbal Bank persona conventions (a fictional Singapore bank), that guide is also the authority; this guide follows its usage (§11).

### 8.5 The Enforcement Climate — DNB's Wwft Record 2025–2026

The research brief asked to verify "the Dutch regulator's 2023-2024 actions" against Adyen — in particular a believed 2024 €9.1 million DNB fine for Wwft failings. **That specific fine could not be verified in this pass**: targeted searches of DNB's enforcement pages, Adyen's press pages, and reputable press returned no DNB enforcement action naming Adyen (see §12.4 for the honest statement of what was and was not searched). What *is* verified is the enforcement climate any Dutch credit institution — Adyen included — operates in, all at press/regulator level:

- **De Volksbank — €20 million in fines (January 22, 2025)**: €15m for conduct-of-business failings plus €5m for deficient anti-money-laundering controls (DNB news). ✅
- **bunq B.V. — €2.6 million fine (imposed May 2025, disclosed August 2025)** for "serious deficiencies" in AML controls (Willkie compliance-concourse summary of the DNB disclosure; press). ✅
- **ABN AMRO — €8.5 million fine (imposed July 2025/announced July 2026)** for serious shortcomings in AML controls over high-risk clients between September 2023 and September 2024, including Russia-sanctions-circumvention screening failures (Straits Times/Reuters/DNB coverage, July 2026). ✅
- **The pattern**: DNB's Wwft enforcement against Dutch financial institutions — banks and fintechs alike — has been active and escalating through 2025–2026, and any banking-licensed payments company in the Netherlands (Adyen included) sits inside that supervisory perimeter by construction (DNB register: "Bank – CI"). ✅

Adyen's own compliance disclosures (annual-report risk sections describing DNB dialogue on Wwft execution) were not retrievable in full in this pass, so the guide's position is: **no Adyen-specific DNB fine verified (⚠), regulator-active climate verified (✅)** — see §12 and §12.4.

### 8.6 The KYC/AML Posture (Cross-Referenced, Condensed)

Adyen's KYC/AML posture, as far as it is public, is the posture of a regulated credit institution plus a platform: it is directly supervised under Dutch law (Wwft — the Dutch AML/CFT act) and under each local licence (UK, US branch, Singapore MPI), and it must run customer due diligence on its direct merchants *and* — through Adyen for Platforms — oversee the onboarding standards of the platforms whose sub-merchants it processes for. The screening, list-management, and transaction-monitoring disciplines this requires are documented in the [Fircosoft](fircosoft_guide.md) guide (§3–§5: name matching, list management, transaction filtering, workflow) and are cross-referenced here, condensed, rather than re-derived. The Singapore overlay — what a Major Payment Institution must do under MAS notices (the PSN series) — is in the [MAS guide](mas_regulations_guidelines_guide.md) §3.4. For a bank reader the practical point is §11's: when Cymbal Bank serves a merchant whose acquiring runs on Adyen-style infrastructure, the bank must know which KYC/AML obligations sit with the platform (merchant onboarding, sub-merchant oversight, Wwft/PSA duties) and which sit with the bank itself (account-opening CDD, transaction monitoring on the settlement flows) — and must document the boundary.

### 8.7 The Licensing Table

| Jurisdiction | Instrument | Status / date | Evidence |
| --- | --- | --- | --- |
| EU / Netherlands | Full banking licence (credit institution) from DNB; EEA passporting under CRD IV | 2017 (grant); current per DNB register ("Bank – CI", F0001) | Adyen Knowledge Hub; Reuters Jun 2017; adyen.com/licenses/emea; DNB register ✅ |
| UK | UK bank authorisation of the UK branch (PRA + FCA, Part 4A FSMA) | September 7, 2023 | Adyen release ✅ |
| US | Federal Foreign Branch (OCC approval; Federal Reserve May 24, 2021) | June 14, 2021 | Adyen release ✅ |
| Singapore | Major Payment Institution under the PSA (cross-border money transfer from Jan 28, 2020; + merchant acquisition and domestic money transfer from May 1, 2021) | 2020–2021 | MAS FID entry 229387; Adyen release May 4, 2021 ✅ |
| Brazil | Acquiring via BIN sponsorship | 2016 | Annual report (via Wikipedia) ✅ |
| HK / Australia / NZ | Direct card acquiring | September 2017 | Finextra; Straits Times ✅ |
| UAE | Retail Payment Services licence (Central Bank of the UAE) | 2026 | H1 2026 release ✅/⚠ |
| DNB enforcement (Adyen-specific) | No fine or measure verified in this pass | — | ⚠/❌-not-found, §12.4 |

---

## 9. Technology — Platform Architecture, Data, and Risk

### 9.1 The Architecture Claims That Are Public

Adyen publishes little engineering detail, so this section separates the architecture claims that are documented from the implementation that is not:

- **One platform, one integration.** The single-platform claim (§4.1) has a concrete architectural reading that is publicly documented: one API and one merchant account across channels and geographies, with the gateway, processing, acquiring, and (post-2017) settlement layers operated by Adyen itself rather than subcontracted per market. ✅ as company claim, consistent across releases and the docs portal; the internal implementation is not disclosed. ⚠
- **Local acquiring / direct scheme connectivity.** "When we became a global acquirer, we directly connected our customers to card schemes" (Knowledge Hub, April 2024) — i.e., Adyen holds scheme membership or acquiring licences per market (§3.3, §8.7) rather than routing through a single sponsor. ✅ as company claim; the per-scheme membership map is not published. ⚠
- **In-house build.** The Goldman Sachs Talks-at-GS summary of van der Does describes Adyen as "building every layer of payments in-house" (goldmansachs.com ⚠ promotional); Adyen's own materials consistently describe owning gateway, risk, processing, acquiring, and settlement (Knowledge Hub). ✅ as company claim.
- **Banking-licence integration.** Since 2017 the settlement layer runs on Adyen's own banking infrastructure: direct connection "to real-time banking and card schemes," same-day settlement potential, and "payments processed up to three days faster than the industry standard" (Knowledge Hub, April 2024). ✅ as company claim; the "three days faster" benchmark is ⚠ company-defined.
- **Data centres and CapEx.** "The majority of our CapEx is invested in data centers," with 2026 CapEx guided to ~7% of net revenue as Adyen pulls 2027 investment forward "to secure compute and storage availability" (H1 2026 release). ✅ as company-reported — a rare public signal that Adyen runs its own processing infrastructure rather than renting pure-cloud capacity end-to-end.
- **Terminal technology.** Android-based POS terminals (global since H2 2020) with the Terminal API (§5.2) — the in-person hardware/software stack. ✅/⚠.

### 9.2 The API and Integration Surface (Cross-Referenced)

Adyen's public developer documentation (developers.adyen.com; docs.adyen.com) documents a server-to-server API family — payments/checkout, the Terminal API, the Management API, Adyen for Platforms (hosted onboarding, transfers, payouts), Issuing, and reporting — with SDKs and hosted components. The pages for Adyen for Platforms and Payouts were extracted and verified live in this pass (§6.1); the full endpoint catalogue was not. The integration-architecture themes this raises — API gateways, canonical data models, event-driven settlement reporting, and how an enterprise middleware estate consumes a payments API — are documented in the [Enterprise Middleware & Integration Platforms](../technology/enterprise_middleware_integration_platform_guide.md) guide (its §3.7 on the API gateway's job and §4 on vendor landscape) and cross-referenced, not re-derived, here. For a bank, the operational consequence of the single-API design is the one §11.5 exploits: a merchant's entire multi-channel payment ledger is addressable through one feed, which makes the bank's reconciliation overlay (§11.5) dramatically simpler than with a multi-vendor stack.

### 9.3 Data, Risk, and the Machine-Learning Layer

The public, verified elements of Adyen's data-and-risk layer:

- **Adyen Protect** — the risk-management product (adyen.com/protect, linked from the Knowledge Hub article as "risk management") covering fraud screening and chargeback management on the platform. ✅ product existence.
- **Network data as product input.** Adyen's 2026 messaging is explicitly data-driven: "Adyen Personalize, leveraging our unified network dataset" and "Adyen Uplift and Dynamic Identification" (conversion optimisation), plus the risk models behind Protect — all running on the transaction data the single platform accumulates across merchants and channels. ✅/⚠ company-reported; the models themselves are not described.
- **The compliance-data overlay.** As a supervised credit institution Adyen runs the standard AML stack (CDD, transaction monitoring, sanctions screening, SAR/STR reporting under Wwft and local equivalents) — the disciplines cross-referenced to the [Fircosoft](fircosoft_guide.md) guide (§3–§5) in §8.6, not re-derived here.
- **Data-centre capital intensity** (§9.1) is the physical layer under the data claims. ✅/⚠ company-reported.

### 9.4 What Is Not Disclosed

The following are not public in any source examined and are flagged rather than guessed: the internal platform architecture (service boundaries, message bus, database topology), the scheme-connectivity map per market, the fraud-model feature sets and performance, the machine-learning infrastructure behind Personalize/Uplift, the cloud versus owned-data-centre split (only the CapEx signal above is public), the exact settlement-banking topology under the licence, and the cybersecurity control details (the DDoS attacks reported against Adyen in 2026 — a LinkedIn/press item surfaced in search, not re-verified — indicate the threat surface exists, but no control detail is public). ⚠ each. The [middleware guide](../technology/enterprise_middleware_integration_platform_guide.md) §4.5's market-share caveat applies equally to any vendor architecture claim made by marketing.

---

## 10. Industry Context — the Competitive Landscape

### 10.1 The Competitive Field

Adyen's competitive set, read from the market structure rather than from any single analyst ranking (⚠ where figures appear): the global merchant-payments field splits into the **pure-play platforms** (Adyen, Stripe, Checkout.com), the **network-era giants and their spin-offs** (PayPal and its Braintree unit, Worldpay — now part of Worldpay Inc. after its FIS carve-out, Fiserv's Carat/Clover, Global Payments), the **bank-owned acquirers** (the large European and US bank acquirers), and the **local/regional specialists**. Adyen's positioning within that field is the most distinct part of its story and is verified from its own materials: it competes at the *enterprise and platform* tier with a *licensed-bank* structure, direct scheme connectivity, and a financial-products suite — a combination none of the pure-play rivals matches on all four axes:

- **Stripe** — the closest pure-play rival at the platform/developer tier, with comparable global ambitions and a far larger SMB/long-tail developer base, but (as of the sources examined) no EU banking licence of equivalent standing and a less pronounced in-person terminal estate; Stripe's private-market valuation history has dwarfed Adyen's public market cap at times — ⚠ analyst/press figures, not re-verified this pass.
- **PayPal / Braintree** — the volume giant; Braintree competes head-on with Adyen for platform and marketplace payments; PayPal's scale and two-sided network are structural advantages, its merchant-acquiring economics are not directly comparable to Adyen's take rate. ⚠ where compared quantitatively.
- **Worldpay (Worldpay Inc.)** — the largest pure merchant acquirer by volume (⚠ industry claims); strong in enterprise and in-person; historically a processing-first business rather than a single-platform product company; Adyen's unified-commerce-plus-financial-products pitch is the differentiation.
- **Checkout.com** — the European challenger with enterprise focus and its own (UK/EMI-era) licensing path; smaller scale than Adyen (⚠), competing on similar enterprise deals.
- **Fiserv (Clover/Carat), Global Payments** — the US acquiring incumbents with enormous SME distribution and integrated-software ecosystems; Adyen meets them at the large-retail/unified-commerce tier (the Toast partnership, §5.2, is itself a competitive alliance against incumbents' restaurant stacks).

The market-context economics — interchange, scheme fees, acquiring margins, take-rate pressure — are documented in the [ancillary-revenue guide](../management/ancillary_revenue_products_guide.md) (§8 on payments/chargebacks in the ancillary context; §10 for card-program economics) and the [Payment Rails](payment_rails_guide.md) guide (§5), and are not re-derived here.

### 10.2 The Competitive-Comparison Table

| Dimension | Adyen | Stripe | PayPal/Braintree | Worldpay | Checkout.com | Fiserv/Global Payments |
| --- | --- | --- | --- | --- | --- | --- |
| Structure | Licensed acquiring bank (EU/UK/US branch/SG MPI) | Payment platform (licences via partners/banks in most markets) | Licensed (EU bank licence via PayPal; Braintree PSP) | Licensed acquirer | Licensed (varies by market) | Licensed acquirers/banks |
| Core tier | Enterprise + platforms + unified commerce | Developers, platforms, SMB→enterprise | Two-sided network, SMB, platforms (Braintree) | Enterprise + SME acquiring | Enterprise + digital | SME + enterprise, integrated software |
| Single-platform claim | Explicit, verified in own releases | Strong product integration | Partial (PayPal vs Braintree split) | Partial (processing-centric) | Partial | Portfolio-based ⚠ |
| Financial products | Accounts, Issuing, Capital, Payouts | Stripe Capital, Issuing, Treasury | PayPal Working Capital, Business accounts | Some (Worldpay for platforms) | Some | Clover Capital etc. |
| POS/in-person | Android terminals + Toast alliance | Terminal (Stripe Terminal) | Zettle | Strong terminal estate | Light | Clover, Carat (very strong) |
| In-house vs sponsor | Banking licence, direct schemes, own data centres (CapEx ~7% of revenue, 2026) | Cloud-native, partner-banked | Mixed | Mixed | Mixed | Mixed |
| Evidence basis this pass | ✅ primary (this guide) | ⚠ general knowledge, not re-verified | ⚠ general knowledge | ⚠ general knowledge | ⚠ general knowledge | ⚠ general knowledge |

The comparison table is deliberately qualitative: every cell in the rival columns rests on general market knowledge rather than on sources re-verified for this guide, and is therefore ⚠ by construction. The only column built on primary verification in this pass is Adyen's.

---

## 11. The Cymbal Bank Worked Example — An Omni-Channel Merchant on an Adyen-Style Platform

### 11.1 The Scenario

**Tropika Marketplace Pte. Ltd.** ("Tropika") is a fictional Singapore-incorporated omni-channel retail group — 40 physical stores across Singapore and Malaysia, a regional e-commerce site, and a third-party marketplace arm with ~2,000 registered sellers. It is a corporate client of **Cymbal Bank**, the fictional Singapore bank persona used across this repository (see the [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) guide for the persona conventions). Tropika's payment architecture is an Adyen-style one: a single licensed payments platform with direct acquiring in Singapore (an MAS **Major Payment Institution** licensed for merchant acquisition — the verified Adyen Singapore structure of §8.4), online and in-store processing on one account, and an Adyen-for-Platforms-style split-payout engine for its marketplace sellers. Cymbal Bank is Tropika's principal bank: it holds Tropika's operating accounts, provides the treasury and FX lines, and — the point of this example — has been asked to act as the settlement overlay that reconciles the platform's payout flows. Every number below is fictional and illustrative; every structural feature is grounded in the verified facts of §2–§9.

Why this scenario for this guide: Tropika is exactly the client a bank like Cymbal meets in Singapore in 2026 — a merchant that has outsourced acquiring to a licensed platform and now needs a bank to make sense of the money movements that platform generates. The exercise mirrors the worked-example conventions of [Reap Global](reap_global_guide.md) §11 and [Bitunix](bitunix_guide.md) §10: on-board the client, map the flows, apply the regulatory overlay, and find the bank's control value-add.

### 11.2 Onboarding and the KYC/AML Overlay

Cymbal Bank onboards Tropika as a corporate client in the normal way — corporate KYC with beneficial-owner identification at the >25% threshold, sanctions screening of the group and its principals (the [Fircosoft](fircosoft_guide.md) guide §3–§5 disciplines, cross-referenced, condensed), and an assessment of the group's business-risk profile (retail, marketplace intermediation, cross-border sales). The acquiring-specific overlay is where the Adyen-style structure changes the bank's work, and it has three parts:

1. **The platform is a regulated counterparty, not just a vendor.** Tropika's acquiring runs on a platform that is itself an MAS Major Payment Institution (§8.4) and, in the real-world analogue, a Dutch credit institution under DNB supervision (§8.1). Cymbal's third-party-risk process therefore treats the platform as a *regulated financial institution counterparty*: Cymbal records its licence numbers (MAS FID entry 229387 in the real-world case), reviews its regulatory standing, and documents that merchant onboarding and CDD for the acquiring relationship sit with the platform under the PSA — while Cymbal's own CDD sits on the *bank accounts* Tropika holds at Cymbal. The boundary is documented, not assumed (the same boundary discipline the MAS guide's scope analysis teaches).
2. **The marketplace tier.** Tropika's marketplace arm means funds flow from thousands of third-party sellers. Under an Adyen-for-Platforms-style arrangement, the platform (as the licensed party) onboarded each seller through hosted onboarding with its own KYC checks; Tropika as platform operator remains responsible for its seller community's conduct. Cymbal's AML team must decide how much of that tier to look through — standard practice is to require the platform's seller-onboarding standards, its monitoring outputs, and its complaint/refund data as part of the client's periodic review, without Cymbal re-KYC-ing 2,000 sellers.
3. **Transaction monitoring on the bank layer.** Cymbal monitors the *bank-account* flows: settlement sweeps from the acquiring platform into Tropika's Cymbal accounts, payout batches to sellers, and FX conversions. The bank's monitoring question — do the aggregated flows match the client's stated business? — is answered with the reconciliation overlay of §11.5.

### 11.3 The Acquiring and Settlement Flows

Tropika's daily payment flows, mapped against the [Payment Rails](payment_rails_guide.md) guide §5 (clearing and settlement mechanics, cross-referenced, not re-derived):

1. **The card leg.** A customer buys SGD 120 in-store in Singapore: the card scheme (Visa/Mastercard) clears the transaction, and the licensed platform — the acquirer of record — settles the merchant's net funds into the platform's settlement account on the scheme's cycle. Online sales across Malaysia, Indonesia, and Thailand route through the same acquiring account (local acquiring where the platform holds rights; cross-border acquiring elsewhere — the footprint of §3.3).
2. **The platform-to-merchant leg.** The platform nets each merchant's daily sales (minus its acquiring fee — the take-rate economics of §4.3, typically a blended fee per transaction) and pays Tropika's settlement into Tropika's Cymbal Bank account, usually T+1 or faster (the platform's own banking licence lets it settle same-day where the rails allow — §9.1).
3. **The store leg.** In-store terminals (the Android-POS estate of §5.2) feed the same merchant account as the web store — one ledger, one settlement — which is the unified-commerce property that makes §11.5's reconciliation tractable.
4. **The payout legs.** Marketplace seller payouts (11.4), supplier payments, and staff disbursements leave from the same platform or from Cymbal accounts via local rails (FAST/PayNow in Singapore) or cross-border rails.

Cymbal Bank's role in this flow is settlement-bank-plus-treasury: it receives the platform's settlement sweeps, provides the SGD operating accounts and the multi-currency accounts for Tropika's regional sales, and prices FX on the conversion legs. The acquiring economics — what the platform earns from each transaction and what it passes through — are the merchant-acquiring economics of the [Ancillary Revenue Products](../management/ancillary_revenue_products_guide.md) guide (§8 on payments/chargebacks/fraud in the ancillary context), cross-referenced, not re-derived.

### 11.4 The Marketplace Payout Mechanics

The marketplace tier is where the Adyen-style structure is most visible and where Cymbal's control framework earns its keep. Structurally (grounded in §6.1's verified mechanics): when a shopper buys from seller #1,847 on Tropika's marketplace, the platform processes the payment, holds the funds under the safeguarding arrangements its licence requires (the PSA safeguarding duty Adyen Singapore states in its May 2021 release, §8.4), deducts the platform's fee and Tropika's commission, and schedules a **split payout** of the seller's net proceeds to the seller's registered bank account — in Singapore, usually same-day via FAST/PayNow. Operationally, Cymbal sees three control points:

1. **Safeguarding verification.** The platform holds seller funds between sale and payout; under the PSA the funds must be safeguarded (segregated or insured) rather than commingled with operating cash. Cymbal's assurance work — either directly or through the client — verifies the safeguarding arrangement annually, because if the platform failed, seller funds would be the first contested pool.
2. **Payout-batch reconciliation.** Every payout batch the platform initiates should reconcile to the day's sales ledger minus fees and refunds. Cymbal runs a daily three-way check between (a) the platform's payout report (via its API feed — the single-API property of §9.2), (b) the debit movements in Tropika's settlement account at Cymbal, and (c) the marketplace's own order ledger.
3. **The seller-tier risk question.** Sellers are onboarded by the platform (hosted onboarding with the platform's KYC), not by Cymbal — but payouts to 2,000 seller accounts are money movements the bank's monitoring systems will see regardless. Cymbal's documented position: the PSA-licensed platform holds the CDD duty for sellers; Cymbal monitors for anomalous aggregate patterns (velocity, round-tripping, concentration) and escalates through Tropika to the platform where needed. The boundary discipline is the same as §11.2's, and the MAS-guide cross-reference (§8.4) is the authority for where the duties sit.

### 11.5 Reconciliation and the Bank's Control Overlay

The commercial heart of the worked example is the reconciliation overlay — the bank product that exists *because* of the platform structure:

1. **The daily three-way reconciliation** (above) between platform ledger, bank account, and client ledger. Because the platform exposes one API feed for all channels (online, in-store, marketplace — §9.2), the feed is complete and standardised; Cymbal's overlay is a straight-through comparison rather than a multi-vendor scrape. This is the bank's answer to the classic acquiring-client problem: "the platform says we settled SGD 1.84m, our bank account shows SGD 1.82m, and finance wants to know why by month-end."
2. **Fee and take-rate verification.** The difference between gross sales and net settlement is the platform's take — fees, scheme costs, FX, chargebacks. Cymbal's overlay recomputes the expected take from the transaction feed against the contracted rate card (the ~17 bps blended economics of §4.3 as the industry reference point, with per-transaction variation by method and market) and flags drift. Fee drift is where acquiring clients quietly lose margin; the bank that audits it has a product.
3. **Chargeback and refund hygiene.** Marketplace chargebacks flow back through the platform to the responsible seller (the [ancillary-revenue guide](../management/ancillary_revenue_products_guide.md) §8 documents the chargeback lifecycle in the ancillary-payments context). Cymbal's overlay tracks chargeback ratios per channel and per seller cohort, because sustained chargeback spikes at the acquiring level are both a fraud signal and a scheme-risk signal that can threaten the client's acquiring relationship — and, through it, the settlement flows the bank depends on.
4. **Treasury and FX.** Tropika's regional sales produce multi-currency balances that the platform converts or holds; Cymbal's treasury overlay consolidates the FX exposure and prices the conversions Tropika chooses to run through the bank rather than the platform — the bank's legitimate slice of the stack.

The overlay's output is a daily exception report and a month-end pack that lets Tropika's CFO sign off the platform ledger against the bank statement — the same "does the ledger tell the truth at month-end?" discipline the Reap worked example closes with ([Reap Global](reap_global_guide.md) §11.5), applied to a merchant-acquiring rather than a corporate-spend context.

### 11.6 The Lessons

The worked example yields five lessons for a bank meeting an Adyen-style client:

1. **The platform is a regulated peer, not a pass-through.** A licensed-acquirer client (MAS MPI here; DNB credit institution in the EU analogue) carries its own supervisory obligations; the bank's job is boundary documentation plus assurance, not re-doing the platform's KYC.
2. **The bank's value moves up the stack.** When acquiring is outsourced, the bank cannot compete on processing — but settlement accounts, FX, safeguarding assurance, fee audit, and reconciliation are services the platform structure *creates demand for*.
3. **Multi-tier flow needs multi-tier controls.** Marketplace payouts mean the bank monitors a seller tier it did not onboard; the control answer is aggregate monitoring plus contractual look-through, with the PSA/MAS overlay as the authority for who owes what.
4. **Single-API architecture is a bank asset.** The platform's one-feed design (§9.2) is what makes straight-through reconciliation possible; banks should prize and price that property when structuring overlay services.
5. **The honest-readiness rule.** Everything in this example that touches real licences is grounded in the verified record of §8; everything else is fictional and flagged as such — the repository's honesty convention, applied.

---

## 12. The Claims Audit — Verified, Flagged, Rejected

### 12.1 The Verified Claims (✅)

| # | Claim | Verification |
| --- | --- | --- |
| 1 | Founded 2006 in Amsterdam | Wikipedia; adyen.com/about/team; financecharts ✅ |
| 2 | Co-founders Pieter van der Does and Arnout Schuijff | Wikipedia; Forbes; team page ✅ |
| 3 | van der Does was CCO at Bibit before founding Adyen | adyen.com/about/team ✅ |
| 4 | Legal entity Adyen N.V., Amsterdam; KvK 34259528; LEI 724500973ODKK3IFQ447; DNB relation F0001, "Bank – CI" | DNB public register ✅ |
| 5 | HQ Amsterdam | DNB register; Wikipedia ✅ |
| 6 | Listed Euronext Amsterdam, ticker ADYEN, ISIN NL0012969182, since June 13, 2018 | Euronext press release; Adyen press ✅ |
| 7 | IPO priced €240/share; €7.1bn market cap; total offering €849m | Adyen press; Euronext press ✅ |
| 8 | Shares ~doubled on debut (~$17bn value) | Reuters (Jun 13, 2018) ✅ |
| 9 | Profitable since 2011 | Recode (2017) ✅ |
| 10 | Dec 2014 US$250m round led by General Atlantic (Temasek, Index Ventures, Felicis) at ~$1.5bn valuation | Adyen/Index/GA releases; WSJ ✅ |
| 11 | Sept 2015 ICONIQ round at $2.3bn valuation | FT headline; CB Insights ✅ |
| 12 | EU (Dutch) banking licence obtained 2017 | Adyen Knowledge Hub; Adyen US release; Reuters Jun 2017 ✅ |
| 13 | EEA passporting under CRD IV from Dutch licence | adyen.com/licenses/emea ✅ |
| 14 | UK banking authorisation (PRA/FCA, Part 4A FSMA), Sept 7, 2023 | Adyen UK release ✅ |
| 15 | US Federal Foreign Branch (OCC), June 14, 2021 (Fed May 24, 2021) | Adyen US release ✅ |
| 16 | Adyen Singapore Pte. Ltd. = MAS Major Payment Institution (FID entry 229387) | MAS FID (extracted live) ✅ |
| 17 | Singapore MPI licence: cross-border money transfer from Jan 28, 2020; expanded May 1, 2021 to merchant acquisition + domestic money transfer; first global payments provider licensed for merchant acquisition in SG | Adyen Singapore release (May 4, 2021) ✅ |
| 18 | eBay signed Adyen as primary payments processing partner (Jan 31, 2018) | eBay Inc. newsroom; Bloomberg ✅ |
| 19 | Adyen Issuing launched November 2019 | Reuters (Nov 14, 2019) ✅ |
| 20 | Android POS terminals launched worldwide H2 2020 | H2 2020 shareholder letter (via Wikipedia) ✅ |
| 21 | "Single platform" language in Adyen's own releases (2021, 2024, 2026) | US-branch release; Knowledge Hub; H1 2026 release ✅ |
| 22 | FY2025: net revenue €2,364.2m; processed volume €1,394.3bn (+8% reported/+21% excl. single large customer); POS €311bn (+34%) | Adyen FY2025 release ⚠-company/✅-published |
| 23 | H1 2026: net revenue €1,302.9m (+19%); processed volume €803.8bn (+24%); EBITDA €641.5m (49%/50%) | Adyen H1 2026 release ⚠-company/✅-published |
| 24 | Talon.One and Orb acquisitions closed July 1, 2026 — first acquisitions in company history | Adyen H1 2026 release; Adyen closing release; Wikipedia ✅ |
| 25 | Ingo Uytdehaage co-CEO (CFO from 2011, appointed co-CEO 2023); Management Board roster as listed | adyen.com/about/team (Sep 2026) ✅ |
| 26 | Adyen for Platforms product (onboarding, processing, payouts) | docs.adyen.com/adyen-for-platforms-model ✅ |
| 27 | Adyen Capital Canada launch (July 2025) | Fintech.ca; FF News ✅ |
| 28 | DNB Wwft enforcement climate: Volksbank €20m (Jan 2025); bunq €2.6m (May 2025); ABN AMRO €8.5m (July 2026) | DNB news/press ✅ |

### 12.2 The Flagged Claims (⚠)

| # | Claim | Why flagged |
| --- | --- | --- |
| 1 | 2011 round of ~€16m led by Index Ventures | Secondary sources only (financecharts, b2bleadnavigator); no primary release retrieved; aggregator cap tables date rounds differently |
| 2 | FY2025/H1 2026 financials | Company-reported through regulated releases; not independently audited in this pass |
| 3 | "Single large-volume customer" identity (press-said eBay) | Adyen never names the customer; identity is press inference |
| 4 | Take rate 16.2–17.1 bps | Company-reported; blended metric |
| 5 | "1.4% additional income" and "three days faster" claims | Company claims (Knowledge Hub); no independent verification |
| 6 | Named customers in Adyen boilerplate (Meta, Uber, H&M, Microsoft, Spotify, L'Oréal, Cathay Pacific, Grab, Klook, Singapore Airlines, Casper, Bonobos) | Company-claimed; not re-verified per-name this pass |
| 7 | 2026 new-logos (Aritzia, OpenAI, Xiaomi, GOV.UK Pay) | Company-reported in H1 2026 release |
| 8 | Workforce 4,345 (2024) / 4,771 (end-2025) | Company-reported |
| 9 | "Over 200 payment methods / 150+ currencies" | Time-varying company claim |
| 10 | Offices in 23 countries | Wikipedia-sourced secondary claim; company does not state a current count |
| 11 | ECB/SSM classification detail | Secondary source only; significance classification not verified at primary source |
| 12 | Post-IPO market-cap history and any Sept-2026 market cap | Press-estimate territory; none verified this pass |
| 13 | $43bn Black Friday/Cyber Monday 2025 volume | Company claim via Retail Insider |
| 14 | "$110bn embedded-finance opportunity" and "75% of SMBs want same-day settlement" | Company-cited research, not independently verified |
| 15 | Adyen Capital lending/risk-retention structure | Not disclosed in sources examined |
| 16 | Any DNB enforcement action specifically against Adyen (incl. the believed €9.1m fine) | Not found in any source examined this pass — see §12.3/§12.4 |

### 12.3 The Rejected or Not-Found Claims (❌)

| # | Claim | Finding |
| --- | --- | --- |
| 1 | KKR invested in Adyen (believed 2011) | No KKR involvement found in any primary/secondary source; the 2014 round was General Atlantic-led, the 2011 round Index Ventures-led |
| 2 | 2011 round "believed ~€55m" | No source supports €55m; best-supported figure is ~€16m (Index Ventures-led) |
| 3 | Dutch banking licence "around 2015" | Corrected — licence obtained 2017 (three sources) |
| 4 | A 2024 DNB fine of €9.1m on Adyen for Wwft failings | Not found in DNB enforcement pages, Adyen releases, or press searched this pass; recorded not-found rather than asserted |
| 5 | Adyen product named "revenue recovery" | No such product found at primary sources; closest verified concepts are Adyen Uplift/Dynamic Identification and the financial-products suite |
| 6 | McDonald's as a verified Adyen customer | Not verified in any source retrieved this pass (absence of evidence, not proof of absence) |

### 12.4 What Could Not Be Verified

This section collects, per the repository's honesty convention, every material item this guide could not confirm — each is deliberately *not* asserted as fact anywhere above:

- **Any DNB enforcement action specifically against Adyen N.V.** — including the believed 2024 €9.1 million fine for Wwft failings that the research brief asked to verify. Targeted searches of DNB's enforcement pages, Adyen's press pages, and reputable press returned no DNB measure naming Adyen in this pass. The relevant DNB enforcement-archive pages proved partially unavailable (the 2024 enforcement-measures listing returned "page not available"), and Adyen's annual-report risk disclosures could not be retrieved in full. A reader with interactive access to DNB's enforcement archive (dnb.nl open-book supervision / enforcement pages) should confirm before any Adyen-specific enforcement statement is made. The *climate* findings (Volksbank, bunq, ABN AMRO fines) are verified and cited; the Adyen-specific question is not.
- **The 2011 funding round at primary source level.** The ~€16m Index Ventures-led Series A rests on two consistent secondary sources; no 2011 primary release was retrieved, and aggregator cap tables disagree on round dating.
- **The identity of Adyen's "single large-volume customer"** (press-inferred to be eBay). Adyen never names it; the press inference is flagged, not asserted.
- **The current (September 2026) market capitalisation and share price.** Post-IPO valuation history is volatile press territory; no current figure was verified and none is asserted.
- **The ECB/SSM significance classification** of Adyen N.V. (direct ECB supervision versus DNB day-to-day supervision of a less-significant institution). The DNB register entry and licence status are verified; the SSM classification detail is not.
- **The precise 2017 licence-grant month** for the Dutch banking licence (the year is verified across three sources).
- **Adyen's current office-country count.** The "23 countries" figure is Wikipedia-sourced (secondary); the company's own pages list country managers but no total count.
- **The Amsterdam street address** (Simon Carmiggeltstraat 6-50) — not re-verified at a primary source this pass.
- **The Bibit corporate history** — including the widely repeated claim that Bibit was acquired by the Royal Bank of Scotland in 2004. The founders' Bibit lineage is verified; Bibit's corporate history is not.
- **Per-name verification of company-boilerplate customers** (Meta, Uber, H&M, Microsoft, Spotify, L'Oréal, Cathay Pacific, Grab, Klook, Singapore Airlines, and the 2026 new logos Aritzia, OpenAI, Xiaomi, GOV.UK Pay). eBay, Netflix, Facebook, and Uber have press verification as cited; the rest are company claims.
- **McDonald's (and any other brief-suggested customer not listed above)** as an Adyen customer — no source retrieved this pass confirms it.
- **The current certified terminal hardware catalogue** and the per-market scheme-membership map.
- **Adyen's internal architecture and risk-model details** — service topology, fraud-model features, Personalize/Uplift ML infrastructure, cloud-versus-owned-data-centre split, settlement-banking topology, cybersecurity controls.
- **Adyen Capital's funding and risk-retention structure**, and the revenue split between processing fees and financial-product income (Adyen does not segment publicly in the sources examined).
- **The 2026 DDoS incident** surfaced in one search result (LinkedIn/press item) — not re-verified; no control detail is public regardless.
- **Independent market-share estimates** for Adyen in global acquiring — none verified; none asserted.

---

## 13. The Glossary

| Term | Meaning |
| --- | --- |
| **Acquiring (merchant acquiring)** | The business of processing card payments for merchants: the acquirer connects merchants to the card schemes, takes settlement risk, and credits merchants' settlement accounts. Adyen's core role since its 2017 banking licence. |
| **Adyen Accounts** | Adyen's embedded bank-account product (adyen.com/accounts), letting platforms offer sellers instant account balances. |
| **Adyen Agentic** | Adyen's 2026 product for processing payments across AI-agent protocols; company links it to the x402 open standard. ⚠ |
| **Adyen Capital** | Adyen's embedded financing/cash-advance product for merchants, distributed through platforms (incl. Adyen Capital Canada, July 2025). |
| **Adyen for Platforms** | The verified product for platforms/marketplaces: onboard users, process payments for them, pay out their funds (split settlements). |
| **Adyen Issuing** | Virtual/physical card issuing for merchants and platforms (launched November 2019). |
| **Adyen Personalize** | 2026 product leveraging Adyen's network dataset for real-time tailored shopping experiences. ⚠ |
| **Adyen Protect** | Adyen's risk-management product (fraud screening, chargebacks). |
| **Adyen Uplift / Dynamic Identification** | 2026 conversion-optimisation features (company cites a +0.9pp conversion uplift by end-H1 2026). ⚠ |
| **AEX** | The Amsterdam Exchange index; Adyen has been an AEX component since its 2018 listing. |
| **Bank – CI** | "Bank – credit institution" — Adyen's registration type in the DNB public register. |
| **Bibit** | The Dutch payments company where the founders worked before Adyen (van der Does was CCO); its own corporate history is unverified here. ⚠ |
| **BIN sponsorship** | Using another licensed institution's Bank Identification Number to acquire/issue; Adyen used it for Brazil (2016) in the pre-licence era. |
| **bps (basis points)** | One-hundredth of a percentage point; Adyen's take rate is quoted in bps (~17). |
| **Co-CEO** | Adyen's dual-CEO structure: Pieter van der Does and Ingo Uytdehaage (appointed co-CEO 2023). |
| **CRD IV** | The EU Capital Requirements Directive IV, under which Adyen's Dutch licence passports across the EEA. |
| **DNB (De Nederlandsche Bank)** | The Dutch central bank and prudential regulator; Adyen's EU supervisor as a credit institution. |
| **EFP (Embedded Financial Products)** | Adyen's bundle of Accounts + Issuing + Capital sold to platforms (UK launch framing, 2023). |
| **Euronext Amsterdam** | The Amsterdam stock exchange; ADYEN has traded there since June 13, 2018. |
| **FCA / PRA** | The UK Financial Conduct Authority and Prudential Regulation Authority; jointly authorised Adyen's UK branch as a bank (Sept 2023). |
| **Federal Foreign Branch** | The US OCC licence type Adyen obtained in 2021 (San Francisco). |
| **Fircosoft** | Sanctions-screening software vendor; the repository's reference guide for screening themes (cross-ref §8.6). |
| **ICONIQ Capital** | The investment firm (representing Silicon Valley principals) behind Adyen's September 2015 round at a $2.3bn valuation. |
| **Intelligent Money Movement** | Adyen's 2026 product bringing enterprise payments, liquidity management, and payouts together "on a single platform." |
| **KvK** | The Dutch Chamber of Commerce (Kamer van Koophandel); Adyen N.V. is registered as 34259528. |
| **Local acquiring** | Acquiring in the country of the transaction via direct scheme connections, rather than routing cross-border through a sponsor. |
| **MAS / PSA / MPI** | Monetary Authority of Singapore / Payment Services Act 2019 / Major Payment Institution — Adyen Singapore's licence tier (cross-ref the MAS guide). |
| **Net revenue** | Adyen's reported revenue metric: processing and service fees net of scheme/other costs, plus financial-product income (as Adyen defines it). |
| **OCC** | US Office of the Comptroller of the Currency — approved Adyen's Federal Foreign Branch (2021). |
| **Orb** | Enterprise usage-based billing platform; acquired by Adyen (closed July 1, 2026) — its first acquisition. |
| **Processed volume** | The gross value of payments Adyen processes (€803.8bn H1 2026, company-reported). ⚠ |
| **Safeguarding** | The PSA requirement that payment-service providers protect customer funds (segregation/insurance); Adyen Singapore states compliance in its 2021 release. |
| **Single platform** | Adyen's own term for one API/one account across channels, geographies, and financial products — the guide's organising concept. |
| **Talon.One** | Omnichannel loyalty/promotions software; acquired by Adyen (closed July 1, 2026). |
| **Take rate** | Net revenue ÷ processed volume; Adyen's blended rate ~16–17 bps. |
| **Unified commerce** | Running online, in-person, and in-app commerce on one platform with one view of the customer. |
| **Wwft** | The Dutch Anti-Money Laundering and Anti-Terrorist Financing Act — the AML/CFT statute under which DNB supervises Dutch institutions including Adyen. |

---

## 14. Cross-References and the Closing Summary

**Cross-references (repository convention: sibling `banking/` guides by plain filename; other folders prefixed with their folder):**

- [Reap Global](reap_global_guide.md) — the fintech-company profile genre, the Cymbal Bank worked-example conventions, and the honesty framing; cited in §1, §11.
- [Bitunix](bitunix_guide.md) — the structural template (header, ToC, claims-audit, glossary, closing conventions); cited in §1, §11.
- [Payment Rails](payment_rails_guide.md) — clearing and settlement mechanics (§5 there) for the acquiring and payout legs; cited in §4, §5, §10, §11.
- [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) — the PSA/MPI regime (§2.4) and the Cymbal Bank persona conventions; cited in §1, §8, §11.
- [Fircosoft](fircosoft_guide.md) — sanctions/AML screening themes (§3–§5), cross-referenced condensed; cited in §8, §11.
- [Ancillary Revenue Products](../management/ancillary_revenue_products_guide.md) — merchant/card-acquiring economics (§8) and card-program economics (§10); cited in §4, §5, §6, §10, §11; not re-derived.
- [Enterprise Middleware & Integration Platforms](../technology/enterprise_middleware_integration_platform_guide.md) — API-gateway and integration-platform themes (§3.7, §4); cited in §1, §5, §9.

**Primary sources used this pass:** adyen.com (press releases: H1 2026 results Aug 13, 2026; UK banking authorization Sep 7, 2023; US branch license Jun 14, 2021; Singapore MPI licensing May 4, 2021; IPO pricing; US$250m funding round; Talon.One/Orb closing — via press-and-media pages; Knowledge Hub "Adyen's Banking License" Apr 16, 2024; about/team page and licences/emea page, extracted September 2026); docs.adyen.com (Adyen for Platforms model; Payouts); developers.adyen.com; eBay Inc. newsroom (Jan 31, 2018); Euronext press release (Jun 13, 2018); the DNB public register (Adyen N.V. entry); the MAS Financial Institutions Directory (entry 229387); DNB enforcement news (de Volksbank); press: Reuters (Jun 2017 licence; Jun 2018 IPO debut; Nov 2019 Issuing; Oct 2025 Q3 results), FT (Sept 2015 valuation), WSJ (Dec 2014 round), Bloomberg (Jan 2018 eBay), TechCrunch (Dec 2014), Recode (Apr 2017), Finextra (Sep 2017), Straits Times (Jul 2026 ABN AMRO), plus aggregator reproductions of Adyen's FY2025/H1 2025 releases (publicnow, substack, fxcintel) and the Wikipedia Adyen article (retrieved 2026) as the synthesis anchor for dated milestones.

**The closing summary.** Adyen's twenty-year arc — from a 2006 Amsterdam startup founded by two Bibit veterans to a Dutch credit institution processing €803.8 billion in a single half-year, listed on Euronext Amsterdam since June 2018, licensed in the EU, the UK, the US, and Singapore, and in 2026 making its first acquisitions (Talon.One and Orb) while launching agentic-payments and money-movement products — is the cleanest documented example in payments of what vertical integration plus a banking licence buys a platform company. The verified record is unusually strong on the fundamentals: founded 2006 by Pieter van der Does and Arnout Schuijff; Amsterdam headquarters; a 2017 DNB banking licence with EEA passporting; a 2021 US Federal Foreign Branch and a 2023 UK PRA/FCA authorisation; an MAS Major Payment Institution licence in Singapore that made Adyen the first global payments provider licensed for merchant acquisition there; a December 2014 General Atlantic round and a September 2015 ICONIQ round at a $2.3 billion valuation — and no KKR involvement anywhere in the record, correcting the research brief; an IPO at €240 per share on June 13, 2018; the eBay partnership that made it a household name in payments; and a financial-products stack (Platforms, Issuing, Capital, Accounts) that has turned the company into a platform-of-platforms. What could not be verified — the Adyen-specific DNB enforcement question, the 2011 round at primary level, current market capitalisation, per-name customer status — is flagged rather than smoothed, per this repository's honesty convention. For Cymbal Bank, the takeaway is structural: the acquiring relationship that used to be a bank's private franchise has become a licensed platform service, and a bank's role in that world is decided by how well it draws the compliance boundary, reconciles the platform ledger against the bank account, and prices the overlay services the platform structure creates. Every merchant, every marketplace payout, and every cross-border settlement eventually resolves into the same architecture — the single platform.
