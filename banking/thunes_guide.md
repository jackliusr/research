# Thunes: The Cross-Border Payout and Collection Network — A Comprehensive Guide

**The Business, History, Products, Licensing, and Technology of Thunes (thunes.com) — from the 2005 TransferTo Origin and the 18 February 2019 Rebrand into DT One and Thunes, through the Limonetik Acquisition (2021), the Tookitaki Majority-Stake Investment (2022), the Agreed Tilia Acquisition (2024), the US$150 Million Series D (2025), and the MAS Major Payment Institution Licence, to a Cymbal Bank Cross-Border Payout and Collection Worked Example**

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Banking Domain / Payments-Infrastructure Company Deep-Dive — the cross-border payout-and-collection network model, the Direct Global Network of direct connections to local payment brands, the PAY and ACCEPT product split, the treasury (SmartX) and compliance (Fortress) layers, the funding history and the MAS Major Payment Institution licence, and the Cymbal Bank corporate-client lens
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** October 2026
> **Companion guides (sibling, same folder — the banking cluster):** [Adyen](adyen_guide.md) (the acquiring/issuing platform-company genre precedent — cross-ref §4, §10) · [Reap Global](reap_global_guide.md) (the fintech-company profile genre and the Cymbal Bank worked-example conventions — cross-ref §11) · [Singapore Fintech & Payments](singapore_fintech_payments_guide.md) (the PSA regulatory frame §2, the Singapore payments firms §3 — note Thunes' absence there, licensing journey §9 — cross-ref §8) · [Payments Hub](payments_hub_guide.md) (hub models and interoperability — cross-ref §4) · [Payment Rails](payment_rails_guide.md) (clearing and settlement mechanics for the payout and collection legs — cross-ref §5, §6, §11; do not re-derive) · [ISO 20022 Core Processes](iso_20022_core_processes_guide.md) (message-standard layer for cross-border instructions — cross-ref §9) · [SWIFT Alliance Access](swift_alliance_access_guide.md) (the correspondent-messaging alternative — cross-ref §10) · [IBPS Payment Connect](ibps_payment_connect_guide.md) (a bank-side payout-channel analogue — cross-ref §9) · [Fircosoft](fircosoft_guide.md) (sanctions/AML screening themes — cross-ref §8, §11, condensed) · [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) (the Payment Services Act regime, the MPI tier, and the Cymbal Bank persona conventions — cross-ref §8, §11) · [Mojaloop](mojaloop_guide.md) (interoperability architecture — cross-ref §9) · [Airwallex](airwallex_guide.md) (a B2B cross-border platform peer — cross-ref §10)
> **Companion guides (other folders):** [Enterprise Middleware & Integration Platforms](../technology/enterprise_middleware_integration_platform_guide.md) (API/integration-platform themes — cross-ref §9, condensed)

---

**How to use this guide:** Section 1 is the overview — the short answer, the key-facts table, why a bank should care, and the evidence base. Section 2 is the company profile — the 2005 TransferTo origin, the 2016 cross-border business, the 18 February 2019 rebrand into DT One and Thunes, the documented co-founder (Peter De Caluwe), the leadership churn (De Caluwe → Vickers → De Caluwe → de Kort → De Caluwe), and the legal-form caution. Section 3 is geography and footprint — the Singapore headquarters, the office network, and the corridor/currency/method counts over time. Section 4 is the mission and business model — the payout-versus-collection network framing, the participant and endpoint taxonomy, the corridor concept, and the revenue structure. Section 5 covers the payout products (PAY). Section 6 covers the collection products (ACCEPT and Thunes Collections) and the value-added layer. Section 7 is growth and market position — the full funding history (2019 Series A through the 2025 Series D), the volume claims (all dated and attributed), and the named customers and partners. Section 8 is licensing and compliance — the MAS Major Payment Institution licence and the December 2025 In-Principle Approval, the UK FCA Authorised Payment Institution authorisation, the French ACPR licence, the Hong Kong MSO licence, the US state money-transmitter claim, and the AML/KYC posture (cross-referencing the Fircosoft guide). Section 9 is technology — the Direct Global Network, the SmartX Treasury System, the Fortress Compliance Platform, the API surface, and everything not disclosed. Section 10 is industry context — the remittance, B2B cross-border, and card-network/bank-inhouse alternatives, unranked. Section 11 is the Cymbal Bank worked example — a fictional corporate client paying gig workers and suppliers across emerging markets through a Thunes-style network. Section 12 is the claims audit (✅/⚠/❌), including the mandatory Tookitaki settlement correction, with §12.4 "What Could Not Be Verified". Section 13 is the glossary. Section 14 is cross-references and the closing summary. **Everything marked ⚠ is company-reported or otherwise unverified; everything marked ❌ was looked for and not found.**

---

## Table of Contents

1. [The Overview](#1-the-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Key-Facts Table](#12-the-key-facts-table)
   - 1.3 [Why This Matters to a Bank](#13-why-this-matters-to-a-bank)
   - 1.4 [The Evidence Base at a Glance](#14-the-evidence-base-at-a-glance)
2. [The Company Profile — Origins, Founders, Leadership, and Legal Form](#2-the-company-profile--origins-founders-leadership-and-legal-form)
   - 2.1 [The TransferTo Origin (2005) and the 2016 Cross-Border Business](#21-the-transferto-origin-2005-and-the-2016-cross-border-business)
   - 2.2 [The 18 February 2019 Rebrand — DT One and Thunes](#22-the-18-february-2019-rebrand--dt-one-and-thunes)
   - 2.3 [The Spin-Out Question — Legal Fact or Business Unit?](#23-the-spin-out-question--legal-fact-or-business-unit)
   - 2.4 [The Co-Founder: Peter De Caluwe](#24-the-co-founder-peter-de-caluwe)
   - 2.5 [The Leadership Churn — Vickers, de Kort, and the Return](#25-the-leadership-churn--vickers-de-kort-and-the-return)
   - 2.6 [The Company Table and the Footprint Timeline](#26-the-company-table-and-the-footprint-timeline)
3. [Geography and Footprint](#3-geography-and-footprint)
   - 3.1 [Headquarters and the Office Network](#31-headquarters-and-the-office-network)
   - 3.2 [The Corridor, Currency, and Method Counts Over Time](#32-the-corridor-currency-and-method-counts-over-time)
   - 3.3 [The Footprint Events (2026)](#33-the-footprint-events-2026)
4. [The Mission and Business Model — the Payout-and-Collection Network](#4-the-mission-and-business-model--the-payout-and-collection-network)
   - 4.1 [The Mission and Positioning Language](#41-the-mission-and-positioning-language)
   - 4.2 [The Business Model — a Network, Not an Operator](#42-the-business-model--a-network-not-an-operator)
   - 4.3 [Participants, Endpoints, and the Meaning of a Corridor](#43-participants-endpoints-and-the-meaning-of-a-corridor)
   - 4.4 [The Revenue Structure](#44-the-revenue-structure)
5. [The Products — the Payout Network (PAY)](#5-the-products--the-payout-network-pay)
   - 5.1 [Thunes PAY — Bank Accounts and Wallets](#51-thunes-pay--bank-accounts-and-wallets)
   - 5.2 [Cash Pickup, Cards, and the Endpoint Mix](#52-cash-pickup-cards-and-the-endpoint-mix)
   - 5.3 [The Payout API Surface](#53-the-payout-api-surface)
6. [The Products — the Collection Network and the Value-Added Layer (ACCEPT and Collections)](#6-the-products--the-collection-network-and-the-value-added-layer-accept-and-collections)
   - 6.1 [Thunes ACCEPT — Local Collection Methods](#61-thunes-accept--local-collection-methods)
   - 6.2 [Thunes Collections (the Limonetik Platform)](#62-thunes-collections-the-limonetik-platform)
   - 6.3 [The Treasury and Compliance Value-Added Layer](#63-the-treasury-and-compliance-value-added-layer)
   - 6.4 [The Product-Surface Map](#64-the-product-surface-map)
7. [Growth and Market Position — Funding, Volumes, and Customers](#7-growth-and-market-position--funding-volumes-and-customers)
   - 7.1 [The Funding History — 2019 to 2025](#71-the-funding-history--2019-to-2025)
   - 7.2 [The Volume and Scale Claims (All Dated)](#72-the-volume-and-scale-claims-all-dated)
   - 7.3 [The Acquisitions and Investments — Limonetik, Tookitaki, Tilia](#73-the-acquisitions-and-investments--limonetik-tookitaki-tilia)
   - 7.4 [The Named Customers and Partners](#74-the-named-customers-and-partners)
   - 7.5 [The Market Position](#75-the-market-position)
8. [Licensing and Compliance](#8-licensing-and-compliance)
   - 8.1 [Singapore — the MAS Major Payment Institution Licence](#81-singapore--the-mas-major-payment-institution-licence)
   - 8.2 [The December 2025 In-Principle Approval](#82-the-december-2025-in-principle-approval)
   - 8.3 [The UK, France, Hong Kong, and US Instruments](#83-the-uk-france-hong-kong-and-us-instruments)
   - 8.4 [The AML/KYC Posture (Cross-Referenced, Condensed)](#84-the-amlkyc-posture-cross-referenced-condensed)
   - 8.5 [The Licensing Table](#85-the-licensing-table)
9. [Technology — the Direct Global Network, SmartX, and Fortress](#9-technology--the-direct-global-network-smartx-and-fortress)
   - 9.1 [The Direct Global Network](#91-the-direct-global-network)
   - 9.2 [The SmartX Treasury System](#92-the-smartx-treasury-system)
   - 9.3 [The Fortress Compliance Platform](#93-the-fortress-compliance-platform)
   - 9.4 [The API Surface and Integration (Cross-Referenced)](#94-the-api-surface-and-integration-cross-referenced)
   - 9.5 [What Is Not Disclosed](#95-what-is-not-disclosed)
10. [Industry Context — the Competitive Landscape](#10-industry-context--the-competitive-landscape)
    - 10.1 [The Competitive Field (Unranked)](#101-the-competitive-field-unranked)
    - 10.2 [The Competitive-Comparison Frame](#102-the-competitive-comparison-frame)
11. [The Cymbal Bank Worked Example — Cross-Border Payouts and Collections for a Corporate Client](#11-the-cymbal-bank-worked-example--cross-border-payouts-and-collections-for-a-corporate-client)
    - 11.1 [The Scenario](#111-the-scenario)
    - 11.2 [What the Bank Buys vs What the Network Provides](#112-what-the-bank-buys-vs-what-the-network-provides)
    - 11.3 [Licensing Boundaries and the MAS Scope](#113-licensing-boundaries-and-the-mas-scope)
    - 11.4 [Safeguarding, Settlement, and Prefunding](#114-safeguarding-settlement-and-prefunding)
    - 11.5 [AML and Sanctions Screening at Both Ends](#115-aml-and-sanctions-screening-at-both-ends)
    - 11.6 [FX Pricing and the Spread](#116-fx-pricing-and-the-spread)
    - 11.7 [Coverage, Corridor Risk, Reconciliation, and Returns](#117-coverage-corridor-risk-reconciliation-and-returns)
    - 11.8 [API Integration, Resilience, Concentration, and Exit](#118-api-integration-resilience-concentration-and-exit)
    - 11.9 [The Lessons](#119-the-lessons)
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

**Thunes** (thunes.com; legal entities including **Thunes Asia Private Limited** in Singapore) is a **cross-border payments network** — the company's own term is the **"proprietary Direct Global Network"** — that connects **Members** (banks and neobanks, mobile wallets, payment service providers, money transfer operators, gig-economy platforms, digital-asset companies, e-commerce platforms, and payroll/EOR providers) to endpoint types (bank accounts, mobile wallets, cards, cash-pickup outlets, and stablecoin wallets) across more than a hundred countries. The company's own positioning line, used verbatim in its releases, is **"Thunes is the Smart Superhighway to move money around the world"**, and it describes two product directions: **PAY** (cross-border payouts) and **ACCEPT** (cross-border collections), plus **Thunes Collections** (the platform it acquired as Limonetik in 2021). Its technology boilerplate names two in-house systems — the **SmartX Treasury System** and the **Fortress Compliance Platform** (Thunes press releases, 2025–2026). ✅

The corporate history is a rebrand rather than a clean founding. Thunes' business originated inside **TransferTo**, a Singapore-based mobile-payments company **founded in 2005**; the cross-border business is described by the company as having **started in 2016**; and on **18 February 2019** TransferTo announced a full rebrand into **two company brands — DT One** (mobile top-up and rewards) and **Thunes** (cross-border payments), with Thunes "branched off to operate independently" (Thunes release, 18 February 2019; Wikipedia infobox lists Thunes' founding as 2016 and its predecessor as TransferTo). ⚠/✅ — see §2.3 for why the phrase "legal spin-out" is deliberately avoided. The registered office on the Monetary Authority of Singapore register is **1 Raffles Place #28-61, One Raffles Place Tower 2, Singapore 048616** (MAS Financial Institutions Directory). ✅

Thunes raised a **US$10 million Series A (6 May 2019, led by GGV Capital)**, a **US$60 million growth round described as its Series B (May 2021, led by Insight Partners)**, a **US$72 million Series C (announced July 2023) at a post-money valuation of over US$900 million**, and a **US$150 million Series D (28 April 2025, led by Apis Partners and Vitruvian Partners)** — the largest in its history. On the Series D the company disclosed a **revenue run-rate of US$150 million and positive EBITDA** (all in the company's own releases; ⚠ company-reported). It has acquired or invested in three companies: **Limonetik** (acquired, announced 21 July 2021), **Tookitaki** (a majority stake of over US$20 million, announced 19 April 2022 — an investment, not a full acquisition, see §7.3 and §12), and **Tilia LLC** (an acquisition agreement announced 23 April 2024, closing date not confirmed — §7.3).

For a bank like Cymbal Bank, Thunes matters on four fronts: as a **wholesale payout-and-collection utility** (a bank can buy network access rather than build dozens of local correspondent relationships — the worked example in §11 does exactly this), as a **licensing-boundary case study** (where the bank's MAS-scoped service ends and the network's own licences begin — §8, §11.3), as a **screening-and-treasury counterparty** (the network holds Member funds and runs its own compliance stack — §9, §11.4–11.5), and as a **competitive-structure signal** (the B2B cross-border network layer that sits between the classic correspondent-banking model and the remittance operators — §10).

### 1.2 The Key-Facts Table

| Aspect | Fact | Status |
| --- | --- | --- |
| Brand / legal name | Thunes; Thunes Asia Private Limited (Singapore entity) | ✅ (MAS FID; company) |
| Origin | Cross-border business within TransferTo (founded 2005; cross-border business from 2016) | ✅ (Thunes release 18 Feb 2019; Wikipedia) |
| Rebrand | TransferTo split into DT One and Thunes, 18 February 2019 | ✅ (Thunes release) |
| Wikipedia infobox founding | 2016 | ⚠ (encyclopedic source; company says business "started in 2016") |
| Headquarters | Singapore | ✅ (company; Wikipedia) |
| Registered office (SG entity) | 1 Raffles Place #28-61, One Raffles Place Tower 2, Singapore 048616 | ✅ (MAS FID) |
| Documented co-founder | Peter De Caluwe (Co-Founder and CEO) | ✅ (company releases; about-us page) |
| Other documented co-founders | None found | ❌ (see §2.4) |
| Leadership (Oct 2026) | Peter De Caluwe (Deputy Chairman and CEO); Chloe Mayenobe (Deputy CEO); Ruwan De Soyza (Chief Legal & Compliance Officer); Simon Nelson (CCO); Andrew Stewart (CRO); Parvinder Bhatia (CFO); Guy Duncan (CTPO) | ✅ (thunes.com/about-us, extracted Oct 2026) |
| Series A | US$10m, announced 6 May 2019, led by GGV Capital | ✅ (Thunes release; TechCrunch 5 May 2019) |
| Series B / growth round | US$60m, May 2021, led by Insight Partners; total capital US$130m | ✅ (Thunes/Limonetik release; TechCrunch 18 May 2021) |
| 2020 round (Wikipedia) | Series B led by Helios Investment Partners, with Checkout.com and GGV — amount NOT verified | ⚠ (Wikipedia; amount unverified) |
| Series C | US$72m, announced July 2023; post-money valuation over US$900m; first close US$60m June 2023 | ✅ (TechCrunch 17 Jul 2023) |
| Series D | US$150m, 28 April 2025, led by Apis Partners and Vitruvian Partners | ✅ (Thunes release) |
| Financials disclosed | Revenue run-rate US$150m; positive EBITDA (company-reported, April 2025) | ⚠ (company-reported) |
| Singapore licence (MAS) | Major Payment Institution — Cross-border Money Transfer Service; Thunes Asia Private Limited | ✅ (MAS FID, extracted Oct 2026) |
| MAS IPA | 2 December 2025 — In-Principle Approval for an MPI licence variation | ✅ (Thunes release 2 Dec 2025; IPA ≠ grant) |
| UK licence | Authorised Payment Institution, FCA firm reference #720167 | ✅ (Thunes release 6 May 2019) |
| France licence | Payment Institution licence from the ACPR | ⚠/✅ (company; see §8.3) |
| Hong Kong licence | Money Service Operator (MSO) licence | ⚠/✅ (company; see §8.3) |
| US | Money-transmitter licences across 50 states (company claim) | ⚠ (company-reported) |
| Acquisitions | Limonetik (2021, acquired); Tookitaki (2022, majority stake >US$20m — NOT a full acquisition); Tilia LLC (2024, agreement; close unconfirmed) | ✅/⚠ (see §7.3) |

### 1.3 Why This Matters to a Bank

Thunes sits at the intersection of three things a bank must understand. **First**, it is a *network utility* rather than a competing retail brand: it does not hold the end customer, it moves funds on behalf of Members, and it competes with — and increasingly replaces — the patchwork of local correspondent and cash-agent relationships that a bank's cross-border payout desk has historically assembled itself. A bank meeting a Thunes-style network therefore faces a build-versus-buy decision on payout reach that did not exist at scale a decade ago. **Second**, it is a *licensing-boundary case*: as an MAS Major Payment Institution (and an FCA Authorised Payment Institution, an ACPR-licensed payment institution, and a US state money transmitter), the network carries its own regulatory obligations, and the bank must document precisely where its own licensed activity ends and the network's begins — the same boundary discipline the [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) guide teaches for the Payment Services Act. **Third**, it is a *treasury-and-compliance counterparty*: because the network holds and prefunds balances and runs its own screening stack (SmartX, Fortress — §9), a bank extending it settlement accounts or credit is taking a counterparty-credit position in a private, non-audited company whose only public financial datapoint is a company-reported revenue run-rate. The §11 worked example works all three threads from Cymbal Bank's side.

### 1.4 The Evidence Base at a Glance

Every factual claim in this guide traces to one of five evidence classes, and the claims audit (§12) records which class supports each claim:

| Evidence class | Examples used in this guide | How it is treated |
| --- | --- | --- |
| Company press releases and pages (thunes.com) | Rebrand release (18 Feb 2019); Series A (6 May 2019); Limonetik (21 Jul 2021); Tookitaki majority stake (19 Apr 2022); Series C coverage; Tilia agreement (23 Apr 2024); Series D (28 Apr 2025); MAS MPI expansion IPA (2 Dec 2025); both 2025/2026 press releases; about-us page (extracted Oct 2026) | ✅ where the fact is the release's own content; ⚠ where the release reports company metrics, counts, or customer names |
| Company boilerplate (verbatim) | "Smart Superhighway to move money around the world"; "proprietary Direct Global Network"; "in-house SmartX Treasury System and Fortress Compliance Platform" | ✅ as company language, quoted verbatim where the guide relies on it |
| Regulator registers | MAS Financial Institutions Directory entry for Thunes Asia Private Limited (extracted live, Oct 2026) | ✅ for registry-style facts |
| Independent press | TechCrunch (5 May 2019 Series A; 18 May 2021 growth round; 17 Jul 2023 Series C); Business Times; fintech.global; Electronic Payments International; PRNewswire (28 Apr 2025) | ✅ for event existence and deal headlines; ⚠ for derived estimates |
| Aggregators and encyclopedic sources | Wikipedia (Thunes article, retrieved 2026); PitchBook (prior valuation) | ✅ where they reproduce primary documents; ⚠ where they add their own numbers |

The methodological limitation of this pass is stated once here and not repeated: **one research pass of the web_search tool on this host returned empty results**, so several secondary questions were resolved only from primary company pages and the two encyclopedic anchors — a tool limitation, not evidence of absence (§12.4). Where a page would not extract or a search returned nothing, the item is recorded as unverified rather than sourced from memory.

---

## 2. The Company Profile — Origins, Founders, Leadership, and Legal Form

### 2.1 The TransferTo Origin (2005) and the 2016 Cross-Border Business

Thunes' lineage runs through TransferTo, a Singapore-based mobile-payments company **founded in 2005**. TransferTo built a business in mobile top-up (airtime and data recharges for prepaid phones) and related payment flows across emerging markets — the "mobile top-up and rewards" line that would become DT One. The cross-border payments business that became Thunes is described by the company itself as having **"started in 2016"** — the date the company's February 2019 rebrand release attaches to Thunes' origin, and the date Wikipedia's infobox adopts as Thunes' founding year. ✅/⚠ (company release ✅; Wikipedia infobox ⚠ as a secondary synthesis).

The honest framing, therefore, is: **the technology, licences in embryo, and corridor relationships that became Thunes were built inside TransferTo between 2016 and 2019**, on top of a company founded in 2005. Whether the 2016 date should be read as the year a separate Thunes business unit was formed, or as the year the cross-border product line was launched, is not resolved by the sources examined (§2.3, §12.4).

### 2.2 The 18 February 2019 Rebrand — DT One and Thunes

On **18 February 2019** TransferTo announced a full rebrand into **two separate company brands**:

- **DT One** — mobile top-up and rewards, with the heritage of the 2005 founding.
- **Thunes** — the cross-border payments business, "which started in 2016," branched off to operate independently.

Primary source: Thunes, "TransferTo announces rebrand with the creation of two market-defining companies: DT One and Thunes" (thunes.com/news, 18 February 2019). ✅ The release names **Peter De Caluwe** at that moment as **"DT One CEO and Thunes Executive Chairman."** ✅ A July 2021 release (the Limonetik acquisition) dates the rebrand again by describing it as happening "four years after the creation of Thunes" — i.e., treating 2016/2017 as Thunes' creation and the 2019 rebrand as its public separation. ✅

The two-brand split is the single most important structural fact in Thunes' history: it is why Thunes' own materials date the company from 2016 while the corporate ancestor dates from 2005, and it is why the repository's other guides that mention TransferTo describe a group of companies rather than a single firm.

### 2.3 The Spin-Out Question — Legal Fact or Business Unit?

The research brief for this guide asked whether Thunes was a **legal spin-out** from TransferTo or a **rebranded, independently-operated business unit**. The honest answer from the sources examined is: **not clearly documented**. The evidence examined supports the following, and nothing stronger:

- The 18 February 2019 release says only that the two brands were created and that Thunes was "branched off to operate independently" — a business-organisation statement, not a statement of shareholding or corporate separation. ✅
- **transferto.com still exists**, describing "a group of leading technology companies" and listing **Peter De Caluwe as "CEO of TransferTo, and Deputy Chairman of Thunes."** ✅ (as seen in the sources examined) — which is inconsistent with a clean, all-links-severed legal spin-off and more consistent with a group structure.
- **Eric Barbier** founded TransferTo/DT One; he is **not** documented in any source examined as a Thunes co-founder. ✅/❌

This guide therefore presents the relationship as: **Thunes is the cross-border payments business that originated within TransferTo (founded 2005; cross-border business from 2016) and was rebranded and operated independently, alongside DT One, from 18 February 2019.** It does **not** assert a clean legal spin-off, and records the open question in §12.4.

### 2.4 The Co-Founder: Peter De Caluwe

**Peter De Caluwe** is the **only person documented as a Thunes co-founder** in the sources examined. The company's own bio records that his career began at Ogone (where he was COO then CEO), that he led Payments at Naspers/PayU, ran DT One, and that **"while at DT One he co-founded Thunes."** ✅ His role title moved through the record as follows (each dated):

- **February 2019 (rebrand release):** "DT One CEO and Thunes Executive Chairman." ✅
- **May 2019 (Series A release):** still Thunes Executive Chairman; the release also states "We've hired a very experienced CEO, Steve Vickers" — so **Steve Vickers** was CEO in May 2019. ✅
- **April 2022 (Tookitaki release):** "Peter De Caluwe, CEO of Thunes." ✅
- **July 2021 (Limonetik release):** "Peter De Caluwe, CEO de Thunes." ✅
- **9 January 2024:** promoted to **Deputy Chairman** on the appointment of Floris de Kort as CEO. ✅
- **December 2025 / March 2026:** confirmed as **"Co-Founder and CEO of Thunes Group"** (2 December 2025 MAS release) and **"Co-Founder and CEO"** (12 March 2026 executive-appointments release). ✅

The exact date De Caluwe first became CEO is **not pinned**: the sources establish it as **"by 2021"** (the July 2021 Limonetik release) and confirm it by April 2022; at least one Italian trade-press item says he was CEO "from 2017," which conflicts with the May 2019 release naming **Steve Vickers** as CEO. This conflict is flagged in §12.4 rather than resolved. ⚠

**No other person is documented as a Thunes co-founder.** Eric Barbier founded TransferTo/DT One but is not documented as a Thunes co-founder. The honest statement: **one documented co-founder (De Caluwe)** — the guide does not invent others. ❌ for any additional co-founder.

### 2.5 The Leadership Churn — Vickers, de Kort, and the Return

Thunes' CEO seat changed hands repeatedly, and the sequence matters because it is the reason some dated sources appear to disagree about "the CEO":

- **May 2019** — **Steve Vickers** hired as CEO (Series A release: "We've hired a very experienced CEO, Steve Vickers"). ✅
- **2020–2021** — **Peter De Caluwe** is named CEO by the July 2021 and April 2022 releases. ✅
- **9 January 2024** — **Floris de Kort** appointed CEO (ex-CEO Global eCommerce at Worldpay; ex-CEO of TSG / Xplor Technologies); De Caluwe became Deputy Chairman. ✅ (Thunes release, 9 Jan 2024, "Thunes expands its leadership to accelerate growth").
- **24 October 2025** — De Kort's last day. His return of the role to De Caluwe was announced around **27 August 2025**. ✅
- **2 December 2025** — De Caluwe confirmed as "Co-Founder and CEO of Thunes Group" in the MAS release. ✅
- **12 March 2026** — De Caluwe confirmed as "Co-Founder and CEO" in the release appointing a new CFO and CTPO. ✅

**Chairman of the Board:** **Allan Green**. ✅

**Executive team (as listed on thunes.com/about-us, extracted October 2026):** Peter De Caluwe (Deputy Chairman and CEO); **Chloe Mayenobe** (Deputy CEO; ex-COO Solaris Group, ex-Deputy CEO Natixis Payments, ex-MD EMEA Ingenico); **Ruwan De Soyza** (Chief Legal & Compliance Officer; ex-Worldpay Group General Counsel); **Simon Nelson** (Chief Commercial Officer; ex-CEO Mercury UAE, Network International, Amex); **Andrew Stewart** (Chief Revenue Officer; ex-WorldRemit); **Parvinder Bhatia** (CFO, joined 12 March 2026, ex-bunq CFO); **Guy Duncan** (Chief Technology and Product Officer, joined 12 March 2026, ex-CTO Tide and OVO Energy). ✅ (as published)

The pattern a bank reader should note: a private payments network of Thunes' size turned over its CEO three times in six years and refreshed its CFO and CTO/CPO in a single March 2026 announcement. That is a governance-and-continuity factor in any counterparty review (§11.8), not a defect — but it is part of the honest picture.

### 2.6 The Company Table and the Footprint Timeline

| Aspect | Verified fact | Status |
| --- | --- | --- |
| Brand | Thunes (thunes.com) | ✅ |
| Legal entity (Singapore) | Thunes Asia Private Limited | ✅ (MAS FID) |
| Predecessor | TransferTo (founded 2005) | ✅ (18 Feb 2019 release; Wikipedia) |
| Cross-border business start | 2016 | ✅ (company release) |
| Rebrand / independent operation | 18 February 2019 (DT One + Thunes) | ✅ (company release) |
| HQ | Singapore | ✅ (company; Wikipedia; MAS FID address) |
| Registered office (SG) | 1 Raffles Place #28-61, One Raffles Place Tower 2, Singapore 048616 | ✅ (MAS FID) |
| Co-founder | Peter De Caluwe (only documented co-founder) | ✅ / ❌ for others |
| CEO (Oct 2026) | Peter De Caluwe (Co-Founder and CEO) | ✅ (about-us; 12 Mar 2026 release) |
| Chairman | Allan Green | ✅ (company) |
| Prior CEOs | Steve Vickers (from May 2019); Floris de Kort (Jan 2024 – 24 Oct 2025) | ✅ |
| Positioning line | "Thunes is the Smart Superhighway to move money around the world" | ✅ (company boilerplate) |

| Period | Milestone | Source |
| --- | --- | --- |
| 2005 | TransferTo founded (Singapore), the corporate ancestor | Wikipedia; company history ✅/⚠ |
| 2016 | Cross-border business that becomes Thunes "started" | 18 Feb 2019 release ✅ |
| 18 Feb 2019 | TransferTo rebrands into DT One and Thunes | Thunes release ✅ |
| 6 May 2019 | US$10m Series A led by GGV Capital | Thunes release; TechCrunch ✅ |
| May 2021 | US$60m growth round ("Series B") led by Insight Partners | Thunes/Limonetik release ✅ |
| 21 Jul 2021 | Limonetik acquisition announced (becomes Thunes Collections) | Thunes release ✅ |
| 19 Apr 2022 | Majority stake (>US$20m) in Tookitaki announced | Thunes release ✅ |
| Jul 2023 | US$72m Series C at >US$900m post-money | TechCrunch ✅ |
| 9 Jan 2024 | Floris de Kort appointed CEO; De Caluwe Deputy Chairman | Thunes release ✅ |
| 23 Apr 2024 | Agreement to acquire Tilia LLC announced | Thunes release ✅ |
| 28 Apr 2025 | US$150m Series D led by Apis Partners and Vitruvian | Thunes release ✅ |
| Aug–Oct 2025 | De Caluwe's return as CEO announced (~27 Aug); de Kort's last day 24 Oct | company/press ✅ |
| 2 Dec 2025 | MAS In-Principle Approval for MPI licence variation | Thunes release ✅ |
| 12 Mar 2026 | New CFO (Bhatia) and CTPO (Duncan) appointed | Thunes release ✅ |
| Jun 2026 | New York City hub announced | company ✅ |
| Sep 2026 | Six new Middle East payout markets announced | company ✅ |

---
## 3. Geography and Footprint

### 3.1 Headquarters and the Office Network

Thunes is **headquartered in Singapore** and describes itself as operating through a network of offices. The company's own boilerplate in the April 2025 Series D release states: **"Headquartered in Singapore, Thunes has offices in 13 locations, including Barcelona, Beijing, Dubai, Hong Kong, Johannesburg, London, Manila, Nairobi, Paris, Riyadh, San Francisco and Shanghai."** ✅ (company boilerplate, 28 April 2025 — note that the twelve cities named plus the Singapore headquarters reconcile to the stated count of thirteen; **the count and list are company-published and dated, and should be treated as a moving number**). ⚠

Two later datapoints extend that footprint, both company-reported: a **new New York City hub** announced in **June 2026**, and **six new Middle East payout markets** announced in **September 2026** (thunes.com/news). ⚠ Both are 2026 additions and, being after the April 2025 boilerplate, are not reflected in the "13 locations" figure.

The Singapore registered office is the MAS-registered address: **1 Raffles Place #28-61, One Raffles Place Tower 2, Singapore 048616** (MAS FID, extracted October 2026). ✅

### 3.2 The Corridor, Currency, and Method Counts Over Time

Corridor, currency, and payment-method counts in this industry are **marketing numbers that move**. The table below dates and attributes each one Thunes has published, so a reader can see the trajectory rather than a single snapshot. **Every figure in this subsection is a company claim of its date; none is independently audited.**

| Date | Countries | Currencies | Methods / integrations | Endpoint scale | Source |
| --- | --- | --- | --- | --- | --- |
| May 2019 (Series A) | 80+ countries | — | 9,000+ payout partners | 300,000+ transactions/day; >US$3bn principal per annum | Thunes release ⚠ |
| 2021 | 110 countries | — | 260+ clients/partners | — | company ⚠ |
| Jul 2023 (Series C) | payouts in 132 countries; collections in 70 markets | 80 currencies | ~300 payment methods | >US$50bn processed to date; 3bn wallets + 4bn bank accounts | TechCrunch / company ⚠ |
| Jan 2024 | 133 countries | 80+ currencies | 330 APMs; 120 mobile wallets | 3bn+ wallets and 4bn bank accounts | company ⚠ |
| Apr 2024 (Tilia release) | 133 countries | 84 currencies | 550 payment methods; 129 wallets | — | company ⚠ |
| Apr 2025 (Series D) | 130+ countries | 80+ currencies | 550+ direct integrations; 320+ payment methods | 7bn wallets + bank accounts; 15bn cards | company ⚠ |
| Dec 2025 (MAS release) | — | — | 320+ local payment methods; 50+ licences | — | company ⚠ |
| Mar 2026 | 140+ countries | 90+ currencies | 220+ payment methods | 12bn wallets/stablecoin wallets/bank accounts; 15bn cards | company ⚠ |

Reading the table: the counts rise and occasionally fall between releases (the "payment methods" line moves between 300, 550, 320, and 220 depending on the date and — evidently — the definition used), which is exactly why this guide dates every one. The *capability* — payouts to bank accounts, wallets, cards, and cash outlets, and collections in local methods — is stable and verified; the *numbers* are company-published and volatile. ✅/⚠

### 3.3 The Footprint Events (2026)

The 2026 footprint events, all company-reported and dated: the **New York City hub (June 2026)** and **six new Middle East payout markets (September 2026)**, both announced on thunes.com/news. ⚠ The New York hub is consistent with the Series D's stated US expansion intent ("supercharge its expansion in the United States, supported by the recent acquisition of licenses across 50 U.S. States, subject to regulatory approval" — Series D release, 28 April 2025). ✅ as company statement.

The precise per-corridor rail map — which receiving market runs over which local clearing system, and which endpoints are reachable in which corridor — is **not published** and is flagged in §9.5 and §12.4. ⚠

---

## 4. The Mission and Business Model — the Payout-and-Collection Network

### 4.1 The Mission and Positioning Language

Thunes does not publish a single sentence labelled "our mission" on the pages examined, but its positioning language is consistent and quotable verbatim. The company boilerplate carried in its 2025–2026 releases states:

> "Thunes is the **Smart Superhighway to move money around the world**. Thunes' **proprietary Direct Global Network** allows Members to make payments in real-time in over 130 countries and more than 80 currencies. Thunes' Network connects directly to over 7 billion mobile wallets and bank accounts worldwide, as well as 15 billion cards via more than 320 different payment methods... Thunes' Direct Global Network differentiates itself through its worldwide reach, **in-house SmartX Treasury System and Fortress Compliance Platform**, ensuring Members of the Network receive unrivaled speed, control, visibility, protection, and cost efficiencies when making real-time payments, globally." ✅

(Thunes boilerplate, Series D release, 28 April 2025 — reproduced here as the company's own positioning language, not as an independent finding.)

The 2025 Series D release adds the vision framing — a "vision to include the **'next billion end users'** in emerging markets," "connecting billions of wallets and thousands of partners worldwide," and the company's stated aim to be "the go-to solution for fast, secure, and cost-effective cross-border payments." ✅ as company language.

### 4.2 The Business Model — a Network, Not an Operator

Thunes' business model is best explained by what it is **not**:

- It is **not a money-transfer operator (MTO)**. An MTO holds the end customer — it contracts with the sender, quotes the rate, and is the counterparty the customer quotes in a complaint. Thunes instead sells network access to its **Members**, and it is the Member (who faces the end customer) that carries the conduct relationship. The company's own customer list supports this: it counts MTOs such as MoneyGram, Western Union, and Remitly **as Members of its network** (Tookitaki release, 19 April 2022), i.e. as its customers, not as itself. ✅
- It is **not an acquirer or processor** in the merchant-accepting sense. The acquiring/issuing model — merchant onboarding, card scheme membership, chargeback liability — is the subject of the sibling [Adyen](adyen_guide.md) guide and is a different function; Thunes' ACCEPT product provides collection *methods*, not merchant acquiring (§6.1). ✅
- It is a **network and interoperability provider** — the layer that connects a Member's instruction to a local payout or collection endpoint in the receiving market, using direct connections to local payment brands rather than a chain of correspondents. This is the "Direct Global Network" claim (§9.1). ✅ as company framing.

The structural consequence for a bank is the one §11 works through: buying network access is a **wholesale utility purchase**, and it moves the bank from being the *operator* of each corridor (with its own correspondent lines, FX inventory, and local agents) to being a *consumer* of a network that aggregates those relationships. That is a real transfer of operational and regulatory responsibility — and of margin — and it is the negotiation the worked example models.

### 4.3 Participants, Endpoints, and the Meaning of a Corridor

**Participants ("Members").** The network's participants, as identified across the company's own customer lists (Tookitaki release 2022; Series D release 2025; about-us page 2026): **banks and neobanks** (e.g., Revolut, and the bank alliances named in 2026 — Absa, Aljazira Bank, J.P. Morgan Payments); **mobile wallets and super-apps** (Grab, WeChat, Singtel Dash, M-PESA, Airtel, MTN, Orange); **money transfer operators** (MoneyGram, Western Union, Remitly); **gig-economy and on-demand platforms** (Uber, Deliveroo, UberEats); **payment service providers** and **digital-asset companies**; and **e-commerce and payroll/EOR** providers. ⚠ (company-named; see §7.4 for the verification discipline applied to each).

**Endpoint types.** The receiving endpoints the network can pay or collect from: **bank accounts**, **mobile wallets**, **cards** (via the 15bn-card claim), **cash-pickup outlets**, and — increasingly — **stablecoin wallets** (the 140+ countries / 12bn wallets/stablecoin wallets/bank accounts claim of March 2026). ⚠ Each endpoint is a company claim of its date.

**Corridor.** A **corridor** is a send-market-to-receive-market pair that carries its own rails, its own FX pricing, and its own licensing requirements — for example, Singapore → the Philippines is a corridor distinct from Singapore → Nigeria, with different local partners, settlement timing, and regulatory permissions on each side. The corridor, not the country, is the unit of a cross-border network's coverage: a network may boast "140+ countries" while economically pricing and operating hundreds of individual corridors, each with its own endpoints, cut-off times, and exception handling. This is the sense in which §3.2's counts are aggregate marketing numbers — the operational reality is the corridor map, which is not published (⚠, §9.5).

### 4.4 The Revenue Structure

Thunes has never published an income statement or segmented revenue. What can be stated is the *structure* of the model (⚠ where inferred):

- **Per-transaction network fees.** The core revenue line: a fee charged to the Member for moving each payment across the network, priced to the corridor and the endpoint. Structurally analogous to any network's per-message take. ⚠ (structure inferred from the scale claims; no published rate card).
- **FX spread.** The margin between the rate the network applies to a cross-border conversion and the rate at which it sources liquidity in the receiving market — the second major line for any payout network, and the one §11.6 makes the bank negotiate explicitly. ⚠
- **Value-added and treasury fees.** Fees attached to the SmartX treasury layer (liquidity, prefunding, FX management) and the Fortress compliance layer (screening as a service) — the "value-added" surface the company bundles into the network proposition. ⚠

**The only public financial figures** are the company's own, disclosed in the **28 April 2025 Series D release**: a **"Revenue run-rate of $150 million and positive EBITDA."** ✅/⚠ (company-reported, dated; not audited). No audited financials are available — Thunes is a private company, and no independent verification of revenue, margin, or profitability exists in the sources examined (§12.4).

---

## 5. The Products — the Payout Network (PAY)

### 5.1 Thunes PAY — Bank Accounts and Wallets

**Thunes PAY** is the company's cross-border **payout** product: a Member instructs the network to deliver funds to a beneficiary's account in a receiving market, in the local currency, and the network executes the payout through its direct connections to that market's bank and wallet endpoints. Documented anchors:

- The product split of **PAY (payouts) and ACCEPT (collections)** is the company's own framing, used across its product pages and releases (thunes.com; Series D release). ✅ as company framing.
- The endpoint reach — the "over 7 billion mobile wallets and bank accounts worldwide" and "more than 320 different payment methods" claims (April 2025 boilerplate), and the "12bn wallets/stablecoin wallets/bank accounts" claim (March 2026) — describe the same PAY surface: the receiving endpoints a payout can land on. ⚠ (company claims; dated).
- The named local methods in the company's own boilerplate — **GCash, M-Pesa, Airtel, MTN, Orange, JazzCash, Easypaisa, AliPay, WeChat Pay** — are the kind of receiving endpoints PAY reaches (April 2025 boilerplate). ✅ as company language; the *live* status and pricing per method are not published.

For a bank, PAY is the build-versus-buy decision of §4.2 expressed as a product: instead of maintaining its own payout arrangements in each receiving market, the bank's corporate client (or the bank itself) submits payout instructions — payroll to gig workers, supplier payments, remittance disbursements — and the network delivers to the local endpoint.

### 5.2 Cash Pickup, Cards, and the Endpoint Mix

The endpoint mix is what distinguishes a payout network from a bank wire. Beyond bank-account and wallet credits, the network's published endpoint types include:

- **Cash pickup** — payout to a physical cash-out outlet, the endpoint type that matters most for unbanked beneficiaries in emerging markets. ✅ as a network endpoint type (company framing; the cash-out partner network is not enumerated).
- **Cards** — the network describes reaching "15 billion cards" (April 2025 boilerplate; repeated March 2026), i.e. card-rail payouts as an endpoint type alongside bank and wallet credits. ⚠ (company claim; the mechanics of the card payout leg are not published).
- **Stablecoin wallets** — the endpoint type added to the company's own count by the March 2026 "12bn wallets/stablecoin wallets/bank accounts" claim, consistent with the company's stablecoin-settlement partnerships (§9.2). ⚠

The endpoint mix is the reason a payout network can reach beneficiaries a correspondent-bank wire cannot, and it is also the reason its exception handling is more complex than a single-rail wire: each endpoint type has its own failure modes (an unclaimed cash payout, a closed wallet, an invalid bank account) that the bank's reconciliation overlay must absorb (§11.7). ✅ as structural observation.

### 5.3 The Payout API Surface

PAY is delivered as an API: the Member integrates once and submits payout instructions programmatically, with status callbacks and reconciliation data. The public evidence of the API surface is the company's developer documentation and product pages (developers.thunes.com / docs), which describe payout creation, beneficiary management, transaction status, and reporting endpoints; the company positions "one integration" to the "Direct Global Network" as the core proposition (company materials, 2025–2026). ✅ as company framing; the full endpoint catalogue was not re-derived in this pass (⚠ depth), and the integration architecture themes are cross-referenced to §9.4 and the [Enterprise Middleware & Integration Platforms](../technology/enterprise_middleware_integration_platform_guide.md) guide, not re-derived here.

---

## 6. The Products — the Collection Network and the Value-Added Layer (ACCEPT and Collections)

### 6.1 Thunes ACCEPT — Local Collection Methods

**Thunes ACCEPT** is the mirror image of PAY: instead of delivering funds *to* a receiving market, it lets a Member **collect** funds *from* payers in local markets using the local payment methods those payers prefer. The company's own description positions ACCEPT alongside PAY as the two directions of the one network (thunes.com; company materials, 2025–2026). ✅ as company framing. The "collections in 70 markets" figure from the July 2023 Series C coverage is the dated anchor for the collection footprint (⚠ company claim, July 2023).

For a bank, ACCEPT answers the question "how does my client get paid by customers in markets where cards are not the norm?" — the collection counterpart to PAY's payout question. Where PAY is payroll and supplier disbursement, ACCEPT is receivables and merchant-of-record collection in local methods.

### 6.2 Thunes Collections (the Limonetik Platform)

**Thunes Collections** is the rebranded **Limonetik** platform, acquired by Thunes and announced on **21 July 2021** (Thunes release: "Thunes acquires Limonetik to accelerate rollout of global payment collections"). The verified facts of the acquired business (from the acquisition release and contemporaneous coverage):

- **Limonetik** was a **Paris-based European payment-methods platform founded in 2008**, led by CEO/co-founder **Christophe Bourbier**; it had **~50 staff**. ✅
- It supported **285+ local payment methods in 70 countries** and served **14,000+ merchants/marketplaces** — including Deliveroo, Uber Eats, Veepee, CMA CGM, Worldline-Ingenico, ACI, Amadeus, and Natixis — and processed **more than EUR2bn/year**. ⚠ (company/coverage figures of 2021).
- The platform was **rebranded to "Thunes Collections"** and folded into the collection side of the network. ✅

The acquisition's own framing is instructive: Thunes described it as accelerating "the rollout of global payment collections," which is why Thunes Collections sits in this section as the collection engine rather than as a separate product line. ✅

### 6.3 The Treasury and Compliance Value-Added Layer

The network's third layer — the part that is sold *with* PAY and ACCEPT rather than as a standalone product — is the treasury-and-compliance bundle the company names in its own boilerplate: the **SmartX Treasury System** and the **Fortress Compliance Platform** (Series D boilerplate, 28 April 2025; see §9.2–§9.3 for the technical treatment). ✅ as company naming. These are **product names from the company boilerplate whose internals are not disclosed** — the guide labels them as such and does not describe their architecture. ⚠

Structurally, this layer is where the network earns the "value-added" portion of §4.4's revenue: FX and liquidity management (SmartX), and screening/compliance-as-a-service extended to Members who would otherwise have to build it themselves (Fortress). For a bank, the significance is that the network is *not* a bare pipe — it is a pipe that holds balances, sources liquidity, and screens transactions, which is precisely why §11 treats it as a counterparty rather than as a courier. ⚠ (structure inferred from company naming and the §4.4 model).

### 6.4 The Product-Surface Map

| Layer | Product (as published) | Evidence |
| --- | --- | --- |
| Cross-border payouts | **Thunes PAY** — bank accounts, wallets, cards, cash pickup | Company materials ✅ as framing |
| Cross-border collections | **Thunes ACCEPT** — local collection methods in receiving markets | Company materials ✅ as framing |
| Collections platform | **Thunes Collections** (rebranded Limonetik, 2008 Paris acquisition announced 21 Jul 2021) | Thunes release ✅ |
| Treasury | **SmartX Treasury System** — liquidity/prefunding/FX management | Company boilerplate ✅ (internals undisclosed ⚠) |
| Compliance | **Fortress Compliance Platform** — screening/compliance-as-a-service | Company boilerplate ✅ (internals undisclosed ⚠) |
| Integration | API access to the "Direct Global Network" (one integration) | Company materials ✅ as framing |

One naming caution: the product names **PAY**, **ACCEPT**, **SmartX**, and **Fortress** are the company's own; the industry-agnostic concepts behind them (payout network, collection engine, treasury liquidity layer, compliance layer) are what this guide describes. The company has not published a component-level architecture, so no claim is made here about how the named products are built. ⚠

---
## 7. Growth and Market Position — Funding, Volumes, and Customers

### 7.1 The Funding History — 2019 to 2025

Thunes raised progressively larger rounds over six years, each dated and each with a named lead:

- **Series A — US$10 million, announced 6 May 2019, led by GGV Capital (Jenny Lee).** Primary: Thunes release ("Thunes closes $10 million Series A investment from global VC GGV Capital"); TechCrunch, 5 May 2019. ✅ At that time the company's own release stated the network reached **80+ countries with 9,000+ payout partners**, handled **300,000+ transactions/day**, and processed **>US$3 billion principal per annum** (⚠ company-reported, 2019). ✅/⚠
- **A 2020 round — a "Series B" per Wikipedia, led by Helios Investment Partners, with participation from Checkout.com and GGV Capital.** The **amount is not verified** in the sources examined — Wikipedia states the round and the lead, but no primary amount was retrieved. Helios Investment Partners is a **confirmed investor** (its site carried the Limonetik release). ⚠ The amount is flagged in §12.4.
- **Series B / growth round — US$60 million, May 2021, led by Insight Partners.** Thunes described it as its Series B and stated it brought **total capital to US$130 million in less than two years**. Primary: the Limonetik release (21 July 2021) states the round; TechCrunch, 18 May 2021. ✅
- **Series C — US$72 million, announced July 2023, at a post-money valuation of over US$900 million.** A first close of **US$60 million** was announced in **June 2023**; the round was **led by Marshall Wace, with Bessemer Venture Partners, 01Fintech, Visa, EDBI (Singapore EDB's venture arm), and Endeavor Catalyst participating**. **Previous valuation ~US$794 million (PitchBook).** Primary: TechCrunch, 17 July 2023. ✅ As of that round the company stated it covered **~300 payment methods across 80 currencies, payouts in 132 countries, collections in 70 markets**, and had **processed more than US$50 billion in transactions to date** (⚠ company-reported, July 2023). ✅/⚠
- **Series D — US$150 million, announced 28 April 2025 — the largest in company history — led by Apis Partners and Vitruvian Partners.** The company described it as coming "at a substantial valuation increase over its last round" (and then-CEO de Kort later called it a "record valuation" in subsequent coverage). The release disclosed a **"Revenue run-rate of $150 million and positive EBITDA"** (⚠ company-reported). **Proton Partners** was financial adviser. Primary: Thunes release ("Thunes Raises USD 150 Million in Series D Funding Round," 28 April 2025); PRNewswire, same date. ✅

**The full investor set named across sources:** GGV Capital, Helios Investment Partners, Checkout.com, Insight Partners, Marshall Wace, Bessemer Venture Partners, Visa, EDBI, Endeavor Catalyst, 01Fintech, Apis Partners, and Vitruvian Partners. ✅ (each as cited to its round).

### 7.2 The Volume and Scale Claims (All Dated)

Every volume and scale figure below is a **company claim of its date**, presented as such. The only financial datapoint that is not a "count" is the April 2025 revenue run-rate.

| Claim | Figure | Date | Source |
| --- | --- | --- | --- |
| Transactions tracked annually | "over 180 million transactions annually" | April 2022 | Tookitaki release ⚠ |
| Cumulative processed | "more than US$50 billion in transactions to date" | July 2023 | Series C coverage ⚠ |
| Revenue run-rate | US$150 million | April 2025 | Series D release ⚠ |
| Profitability | "positive EBITDA" | April 2025 | Series D release ⚠ |
| Direct network scale | 130+ countries, 80+ currencies, 550+ direct integrations | April 2025 | Series D release ⚠ |
| Licences | "over 50 financial service licenses across the globe" | December 2025 | MAS IPA release ⚠ |
| Network reach | 140+ countries, 90+ currencies | March 2026 | company release ⚠ |

The honest reading: Thunes has grown through every count it publishes, but **none of these figures is audited or independently verified**, and the company is private with no expected audited statement. The April 2025 "US$150 million revenue run-rate and positive EBITDA" is the single most consequential number for a counterparty review — and it is a company statement, not a verified fact (§12.4). ⚠

### 7.3 The Acquisitions and Investments — Limonetik, Tookitaki, Tilia

The corporate-history hazards of Thunes' record are concentrated in its three deals, so each is set out precisely.

**1. Limonetik — an ACQUISITION, announced 21 July 2021.** A Paris-based European payment-methods platform (founded 2008; CEO/co-founder **Christophe Bourbier**; ~50 staff; 285+ local payment methods in 70 countries; 14,000+ merchants/marketplaces; >EUR2bn/year processed). Rebranded to **Thunes Collections** and integrated into the network's collection side. Primary: Thunes release, 21 July 2021; also covered by Helios and Electronic Payments International. ✅

**2. Tookitaki — a MAJORITY-STAKE INVESTMENT, announced 19 April 2022. NOT a full acquisition.** This is the deal the repository's other work has mischaracterised, and it is settled here from the primary record:

- Thunes' own release title is **"Thunes Takes Majority Stake in AML and Compliance Platform Tookitaki"** (thunes.com/news/thunes-tookitaki/, 19 April 2022). ✅
- The release states Thunes "has taken a majority stake in the anti-money laundering (AML) and compliance technology firm, **Tookitaki Holding Pte Ltd** ('Tookitaki') **by making an investment of over $20 million**." ✅ (an investment of over US$20 million for a *controlling stake*, not a purchase of the whole company).
- The release states explicitly: **"Thunes and Tookitaki businesses will continue to operate independently, with the alliance strengthening both companies."** ✅ — i.e. an investment/alliance, not a clean acquisition/exit.
- Tookitaki was **founded in November 2014**, is Singapore-headquartered, employs **over 100 people** across Asia, Europe, and the US, and is led by **founder and CEO Abhishek Chatterjee**. ✅ (the "founded November 2014" date is Thunes' own release text — it **contradicts** the repository's "founded 2012" claim; see §12).

Sources: https://www.thunes.com/news/thunes-tookitaki/ and https://www.tookitaki.com/blog/thunes-takes-majority-stake-in-the-aml-and-compliance-platform-tookitaki . ✅ The three corrections (majority stake not acquisition; independent operation; founded November 2014) are carried into the claims audit (§12.3) as a **mandatory** item for the dispatcher to apply to `technology/singapore_saas_companies_guide.md`.

**3. Tilia LLC — an agreed ACQUISITION, announced 23 April 2024; closing date NOT confirmed.** **Tilia** is an all-in-one payments platform (acceptance and pay-outs) for online games, virtual worlds, creator economies, and in-app purchases, licensed in **48 US states/territories**, whose majority owner is **Linden Research, Inc. ("Linden Lab," maker of Second Life)**. The announcement stated that on closing Tilia would be **rebranded Thunes** and remain **San Francisco**, and that a **five-year exclusive partnership with Linden Lab** was agreed; the deal was **"subject to regulatory approvals."** ✅ (announcement). **The completion date is not confirmed in the sources examined** — Wikipedia loosely says "June 2025," and the company's April 2025 Series D release refers to "the **recent acquisition of licenses across 50 U.S. States, subject to regulatory approval**," which is consistent with a Tilia-related licensing step but does not date a closing. **Flagged as unverified (§12.4).** ⚠

**US money-transmitter footprint:** Thunes is described — by the company and by Wikipedia — as holding **money-transmitter licences across 50 US states**. ⚠ (company claim).

### 7.4 The Named Customers and Partners

Named-customer and partner claims are high-risk facts, so this guide separates the press/primary-verified from the company-claimed:

**Company-named Members (⚠, from Thunes' own releases):**

- **Gig-economy and on-demand platforms:** Uber, Deliveroo, UberEats (Tookitaki release, 19 April 2022). ⚠
- **Money transfer operators:** MoneyGram, Western Union, Remitly (Tookitaki release). ⚠
- **Neobank:** Revolut (Tookitaki release). ⚠
- **Wallets and fintechs:** PayPal, Singtel Dash, M-PESA, Airtel (Tookitaki release). ⚠
- **Super-apps and wallets (later boilerplate):** Grab, WeChat (Series D boilerplate, April 2025). ⚠

**Company-named partners and alliances (⚠, from the newsroom and about-us page):** the company's own newsroom (thunes.com/news) lists, among others, **Mastercard, Visa, PayPal-Hyperwallet, Swift, Ripple, Ecobank, Absa, J.P. Morgan (Payments' Xpedite Remit solution), and Fiserv** partnerships, alongside the 2026 bank alliances (Absa multicurrency clearing; Aljazira Bank; J.P. Morgan Payments) and the **Circle USDC (October 2024)** and **EURC euro treasury (August 2026)** stablecoin items. ⚠ (company-published; not re-verified per item in this pass unless separately cited).

**The verification discipline:** none of these relationships is re-verified per-name in this pass beyond the company's own publication of it; each is presented as a company claim with its date. The absence of a name here is not a denial of the relationship — it is the guide declining to assert what it did not independently verify. ⚠

### 7.5 The Market Position

Thunes' market position, as far as it can be stated without analyst rankings: it is one of a small group of **independent B2B cross-border networks** — competitors and peers in the same structural tier are named in §10 — that sell wholesale payout/collection reach to institutions rather than serving end customers directly. Its distinguishing published features are the **Direct Global Network** (direct connections to local brands rather than correspondent chains), the **in-house SmartX and Fortress layers**, and a **licensing footprint across Singapore, the UK, France, Hong Kong, and the US states** (§8). No independent market-share figure for Thunes in any segment was verified in this pass, and none is asserted (§12.4). ⚠

---

## 8. Licensing and Compliance

### 8.1 Singapore — the MAS Major Payment Institution Licence

The cleanest licensing verification in this guide is the Monetary Authority of Singapore register entry, extracted live for this pass:

> **THUNES ASIA PRIVATE LIMITED** — Incorporated in Singapore — **Major Payment Institution** — regulated activity: **Cross-border Money Transfer Service** — key personnel: **De Caluwe Peter Irene F** — address: **1 Raffles Place #28-61, One Raffles Place Tower 2, Singapore 048616** — contact +65 6983 2500. (MAS Financial Institutions Directory, eservices.mas.gov.sg/fid, extracted October 2026) ✅

This confirms: the entity is **Thunes Asia Private Limited**; the licence tier is **Major Payment Institution (MPI)**; the licensed regulated activity on the register is **Cross-border Money Transfer Service**; and the named key personnel is **Peter De Caluwe**. ✅

The regulatory frame itself — the Payment Services Act regime, the Standard-versus-Major Payment Institution tiers, the seven regulated activities, and the safeguarding and AML obligations — is documented in the sibling [Singapore Fintech & Payments](singapore_fintech_payments_guide.md) guide (§2 there, the Regulatory Framework) and in the [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) guide (the PS Act and the MPI tier), and is **cross-referenced, not re-derived here**.

One repository finding worth recording: **Thunes does not appear in the payments-firm table of the [Singapore Fintech & Payments](singapore_fintech_payments_guide.md) guide's §3** (the payments firms section). A `grep` for "Thunes" in that guide returns zero matches. That absence is itself a finding — a Singapore-headquartered, MAS-MPI-licensed cross-border network is missing from the repository's Singapore payments-firm roster, and this guide supplies the profile that was missing. ✅ (finding).

### 8.2 The December 2025 In-Principle Approval

On **2 December 2025** Thunes Asia announced that it had received **In-Principle Approval (IPA)** from MAS for a **variation of its MPI licence** which, **once granted**, would add **account issuance, domestic money transfer, merchant acquisition, and e-money issuance** — enabling Singapore merchants to accept international payment methods and overseas merchants to accept **PayNow and GrabPay**. Primary: Thunes release ("Thunes MAS MPI expansion," thunes.com/news, 2 December 2025). ✅

Two cautions the guide carries verbatim from the primary record: **an IPA is not a grant** — it is an in-principle approval subject to conditions, and the release frames the additional activities as ones the licence "once granted" would carry. The same release also states the group holds **"over 50 financial service licenses across the globe"** and access to **320+ local payment methods** — both ⚠ company claims of that date. The MAS-in-principle language should not be read as current licensed activity until the grant is confirmed on the register; the register extracted in October 2026 shows the **Cross-border Money Transfer Service** activity (§8.1), and a reader should re-check the register for the variation's grant status. ⚠

### 8.3 The UK, France, Hong Kong, and US Instruments

Beyond Singapore, the licences named in Thunes' own record (each dated to the source named):

- **United Kingdom — Authorised Payment Institution (API), FCA firm reference #720167.** Stated in the 6 May 2019 Series A release; Wikipedia also notes FCA API status. ✅ (as of the 2019 statement; current status not re-verified in this pass). The authorisation type and its permitted activities under the UK regime are a UK-law question sourced to the FCA; this guide states only what the company published.
- **France — a Payment Institution licence from the ACPR** (the French Prudential Supervision and Resolution Authority). ⚠/✅ (company-stated; consistent with the Limonetik/Paris footprint).
- **Hong Kong — a Money Service Operator (MSO) licence.** ⚠/✅ (company-stated).
- **United States — money-transmitter licences in 50 states** (company claim); and the **Tilia** transaction, announced 23 April 2024, involved a platform **licensed in 48 US states/territories** (§7.3). ⚠

**No jurisdiction's licensing rule is stated in this guide without a source.** Where the company is the only source (France, Hong Kong, US state count), the item is marked ⚠/✅ as company-stated; where the regulator register was queried (Singapore), the item is ✅ (§8.1).

### 8.4 The AML/KYC Posture (Cross-Referenced, Condensed)

Thunes' AML/KYC posture, as far as it is public, is the posture of a licensed payment institution that also sells compliance as a product. Verified anchors:

- As an **MAS Major Payment Institution**, Thunes Asia is subject to the PS Act's AML/CFT obligations and the MAS PSN-series notices — the regime documented in the [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) guide (§3.4) and the [Singapore Fintech & Payments](singapore_fintech_payments_guide.md) guide, cross-referenced, not re-derived here. ✅
- The company operates an in-house compliance stack it names the **Fortress Compliance Platform** and, in 2022, invested over **US$20 million for a majority stake in Tookitaki**, an AML/compliance regtech, to "extend Tookitaki's industry-leading compliance and anti-money-laundering (AML) capabilities" across its payment flows (Tookitaki release, 19 April 2022). ✅ — a network buying AML capability rather than only building it.
- The 2026 bank alliances (Absa, Aljazira Bank, J.P. Morgan Payments) indicate institution-to-institution relationships in which the counterparty bank will run its own screening on the flows (§11.5).

The screening *themes* this raises — list management, name matching, false-positive handling, transaction filtering, and the workflow around alerts — are documented in the [Fircosoft](fircosoft_guide.md) guide (§3–§5) and are **cross-referenced here, condensed, rather than re-derived**. For a bank, the practical point is §11.5's: when Cymbal Bank and the network both screen, the bank must know which screening obligations sit with it and which with the network, and must document the boundary.

### 8.5 The Licensing Table

| Jurisdiction | Instrument | Status / date | Evidence |
| --- | --- | --- | --- |
| Singapore | Major Payment Institution (regulated activity: Cross-border Money Transfer Service) | Current at extraction (Oct 2026); Thunes Asia Private Limited | MAS FID ✅ |
| Singapore | In-Principle Approval to vary the MPI licence (account issuance, domestic money transfer, merchant acquisition, e-money issuance) | 2 December 2025 (IPA, not a grant) | Thunes release ✅ |
| United Kingdom | Authorised Payment Institution, FCA firm reference #720167 | Stated 6 May 2019 | Thunes release ✅ (current status not re-verified ⚠) |
| France | Payment Institution licence from the ACPR | Company-stated | company ⚠/✅ |
| Hong Kong | Money Service Operator (MSO) licence | Company-stated | company ⚠/✅ |
| United States | Money-transmitter licences across 50 states | Company-stated | company ⚠ |
| United States (Tilia) | 48 US states/territories (platform acquired by agreement, 23 Apr 2024); "50 U.S. States, subject to regulatory approval" referenced Apr 2025 | Announcement; closing not confirmed | Thunes release ✅/⚠ |
| Group total | "over 50 financial service licenses across the globe" | Company-stated, 2 December 2025 | company ⚠ |

---

## 9. Technology — the Direct Global Network, SmartX, and Fortress

### 9.1 The Direct Global Network

The **Direct Global Network** is the company's name for its core architecture claim: instead of routing a cross-border payment through a chain of correspondent banks, the network maintains **direct connections to local payment brands** — the wallets, bank rails, and cash networks in each receiving market — and executes the payout on the local endpoint directly. Verified as company framing:

> "Thunes' proprietary Direct Global Network allows Members to make payments in real-time in over 130 countries and more than 80 currencies. Thunes' Network connects directly to over 7 billion mobile wallets and bank accounts worldwide, as well as 15 billion cards via more than 320 different payment methods." (Series D boilerplate, 28 April 2025) ✅ as company claim.

The architectural *idea* — direct integration with each local endpoint rather than a correspondent chain — is a documented industry pattern (the interoperability-by-direct-integration model that the sibling [Mojaloop](mojaloop_guide.md) guide analyses from the open-standards side, and that the [Payments Hub](payments_hub_guide.md) guide analyses from the hub-model side). What Thunes **does not publish** is the per-corridor integration map, the counterparty list per market, and the actual redundancy/fallback design — flagged in §9.5. ✅/⚠

### 9.2 The SmartX Treasury System

**SmartX** is the company's name for its **in-house treasury system**, named in the boilerplate alongside Fortress and associated with "speed, control, visibility, protection, and cost efficiencies." Functionally, a payout network's treasury layer must: hold and **prefund balances** in receiving markets so payouts can be instant; manage **liquidity and FX** across those balances; and settle with local partners. The company's stablecoin work bears on this layer directly: the **Circle USDC partnership (October 2024)** and the **EURC euro treasury item (August 2026)** are the datable public signals of stablecoin/tokenised funding being used within the treasury model. ✅ (company items, dated). The **internals of SmartX are not disclosed** — the guide names it as a company product, records the stablecoin signals, and labels the architecture as unknown. ⚠ The concept of prefunding and liquidity management across a corridor is cross-referenced to the [Payment Rails](payment_rails_guide.md) guide for the settlement mechanics, not re-derived here.

### 9.3 The Fortress Compliance Platform

**Fortress** is the company's name for its **compliance platform**, positioned as delivering "protection" and "control" to Members as part of the network proposition. The 2022 investment of **over US$20 million for a majority stake in Tookitaki** (§7.3) is the company's own evidence that it chose to strengthen its compliance capability by investing in an AML specialist rather than building alone — the release states Thunes is "able to extend Tookitaki's industry-leading compliance and anti-money-laundering (AML) capabilities to further safeguard the businesses and create more transparency around the payment flows of its global customers." ✅ (company statement, 19 April 2022). As with SmartX, **Fortress' internals are not disclosed**, and the screening disciplines it embodies are cross-referenced to the [Fircosoft](fircosoft_guide.md) guide rather than described. ⚠

### 9.4 The API Surface and Integration (Cross-Referenced)

The network is delivered as an API — one integration to reach the Direct Global Network, with payout and collection instructions, beneficiary management, status callbacks, and reporting exposed programmatically (company materials; developers.thunes.com as the stated documentation host). ✅ as company framing; the endpoint catalogue was not re-derived. The integration-architecture themes — API gateways, canonical data models, event-driven status reporting, and how a bank's middleware estate consumes a payout API — are documented in the [Enterprise Middleware & Integration Platforms](../technology/enterprise_middleware_integration_platform_guide.md) guide and are **cross-referenced, not re-derived**. The message-standard dimension for cross-border instructions (the ISO 20022 migration) is the subject of the sibling [ISO 20022 Core Processes](iso_20022_core_processes_guide.md) guide; the correspondent-messaging alternative (SWIFT) is the subject of the [SWIFT Alliance Access](swift_alliance_access_guide.md) guide; and the bank-side payout-channel view is the [IBPS Payment Connect](ibps_payment_connect_guide.md) guide. Each is cross-referenced, condensed.

### 9.5 What Is Not Disclosed

Not public in any source examined, and flagged rather than guessed: the **internal architecture** of SmartX and Fortress (product names only); the **per-corridor rail and partner map** (which local clearing system, wallet, or cash network serves which receiving market); the **cloud/hosting topology**; the **API rate structure** and any published fee schedule; the **counterparty and prefunding arrangements** with local partners; the **fraud and sanctions models** behind Fortress; and the **redundancy/fallback design** for corridor outages. ⚠ each. The [Enterprise Middleware & Integration Platforms](../technology/enterprise_middleware_integration_platform_guide.md) guide's market-share caveat applies equally to any vendor architecture claim made by marketing.

---

## 10. Industry Context — the Competitive Landscape

### 10.1 The Competitive Field (Unranked)

Thunes competes and coexists with several structurally distinct groups. **No ranking, recommendation, or comparison figure is offered** — the groups are named factually so a bank can place the network in the landscape:

- **The remittance and cross-border operators** — Wise, Remitly, Western Union, MoneyGram, and Zepz/WorldRemit. These are, in places, **customers** of a network like Thunes (Western Union, MoneyGram, and Remitly are named as Members; §7.4) *and* competitors to the extent they build their own payout reach. Structurally they face the end customer; Thunes does not (§4.2). ✅/⚠
- **The B2B cross-border networks and platforms** — Nium, Airwallex, Payoneer, dLocal, Rapyd, CurrencyCloud, Ebury, TerraPay, Moov, and Flutterwave. This is the tier in which Thunes sits most directly: wholesale payout/collection and account infrastructure sold to businesses and institutions. The repository treats Airwallex in its own sibling guide ([Airwallex](airwallex_guide.md)) and Reap Global in [Reap Global](reap_global_guide.md). ✅/⚠
- **The card-network and bank-in-house alternatives** — **Visa Direct** and **Mastercard Send** (card-rail push payments), and **SWIFT gpi** with correspondent banking (the correspondent-messaging model documented in the [SWIFT Alliance Access](swift_alliance_access_guide.md) guide). These are the incumbents a network partly displaces on the payout leg and partly partners with (Visa is an investor and partner; Mastercard a named partner — §7.1, §7.4). ✅/⚠

The competitive framing the repository already supplies is cross-referenced rather than re-derived: the acquiring/issuing platform-company frame is in [Adyen](adyen_guide.md) §10; the fintech-company competitive conventions are in [Reap Global](reap_global_guide.md); and the Singapore payments-firm set is in [Singapore Fintech & Payments](singapore_fintech_payments_guide.md) §3 (where, as noted in §8.1, **Thunes is absent** — the reader should treat that guide's §3 roster as incomplete for the cross-border-network segment).

### 10.2 The Competitive-Comparison Frame

| Dimension | Thunes (verified this pass) | Remittance operators | B2B cross-border networks/platforms | Card-network rails (Visa Direct / Mastercard Send) | Correspondent banking (SWIFT gpi) |
| --- | --- | --- | --- | --- | --- |
| Who faces the end customer | The Member, not Thunes | The operator | The business client | The sending institution | The sending institution |
| Core position | Wholesale payout/collection network | Consumer/agent remittance | Corporate/API cross-border | Card-rail push payments | Bank-to-bank messaging/settlement |
| Products | PAY, ACCEPT, Collections, SmartX, Fortress | Transfer + payout reach | Accounts, payouts, collections, FX | Card payout endpoints | gpi tracking, correspondent settlement |
| Markets | 130+ countries (Apr 2025, company claim) | Various | Various | Card-accepting markets | Correspondent-covered markets |
| Licensing model | Own licences (MAS MPI; UK API; FR, HK, US states) | Own licences/agents | Own licences (varies) | Network membership | Bank licences + correspondent lines |
| Evidence basis this pass | ✅ primary (this guide) | ⚠ general knowledge | ⚠ general knowledge | ⚠ general knowledge | ⚠ repo-guide cross-ref |

The comparison is deliberately qualitative: every cell in the non-Thunes columns rests on general market knowledge or repository cross-references rather than on sources re-verified for this guide, and is therefore ⚠ by construction. The only column built on primary verification in this pass is Thunes'.

---
## 11. The Cymbal Bank Worked Example — Cross-Border Payouts and Collections for a Corporate Client

### 11.1 The Scenario

**Nimbus Payroll Services Pte. Ltd.** ("Nimbus") is a fictional Singapore-incorporated **payroll and employer-of-record (EOR) provider** and a corporate client of **Cymbal Bank**, the fictional Singapore bank persona used across this repository (see the [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) guide for the persona conventions). Nimbus runs payroll for about 4,000 gig workers, contractors, and offshore employees across eight emerging markets — the Philippines, Indonesia, Vietnam, India, Nigeria, Kenya, Mexico, and Colombia — and also handles collections from a handful of regional enterprise customers. Its two core money movements are:

- **Payouts:** it disburses salaries and contractor fees to beneficiaries' bank accounts, mobile wallets (GCash, M-Pesa, and similar), and — for a minority of unbanked workers — cash-pickup outlets, in the beneficiary's local currency. Its monthly payout volume is about **SGD 6 million**.
- **Collections:** it receives payments from enterprise clients in Singapore, Malaysia, and Indonesia in local methods (bank transfer, QR, and local wallets), which it must reconcile against invoices.

Nimbus's problem is the one every Asia-headquartered payroll business has: maintaining its own payout arrangements across eight markets — local partners, FX inventory, cut-off times, and reconciliation — is expensive, slow, and operationally fragile. Its banking relationship, Cymbal Bank, holds Nimbus's operating and payroll-funding accounts but cannot reach all eight markets efficiently on its own. The relationship team therefore evaluates a **Thunes-style network** (the verified model of §4–§9) as the payout-and-collection utility, and designs Cymbal Bank's role around it: **Cymbal remains Nimbus's bank** — holding the funding accounts, providing the SGD↔local-currency FX line where it is competitive, and running the **reconciliation overlay** that makes the network's flows verifiable (§11.7). Every number below is fictional and illustrative; every structural feature is grounded in the verified facts of §2–§10.

Why this scenario for this guide: an EOR/payroll client paying gig workers and suppliers in emerging markets, plus a remittance-style use case, is exactly the client structure a Thunes-style payout network was built to serve (the company's own Members include gig-economy platforms and MTOs — §4.3). It is also the structure in which the bank-versus-network division of licensed responsibility is hardest to draw, which is the pedagogical point.

### 11.2 What the Bank Buys vs What the Network Provides

The first question Cymbal Bank asks is the simplest: **what, exactly, is being bought?** For a Thunes-style network, the answer is *network access and execution*, and the division is:

| Responsibility | Who owns it (structurally) | Worked-example detail |
| --- | --- | --- |
| Hold the client relationship and the end-customer conduct | **Cymbal Bank / Nimbus** | Nimbus faces its workers and enterprise clients; Cymbal faces Nimbus |
| Hold the funding accounts | **Cymbal Bank** | Nimbus's SGD operating/payroll accounts at Cymbal |
| Execute the cross-border payout to the local endpoint | **The network** | The network delivers to wallet, bank, or cash endpoint in each market |
| Provide local-currency liquidity and prefunding | **The network (SmartX layer)** | The network funds local payouts from its own balances (§11.4) |
| Screening the transaction | **Both, at different points** | Network screening (Fortress) + Cymbal's own monitoring (§11.5) |
| FX conversion | **Contested** | The network quotes; the bank can compete on the SGD leg (§11.6) |
| Reconciliation of what moved | **Cymbal Bank (the overlay)** | The bank reconciles network reports against Nimbus's ledger (§11.7) |

The discipline is to write this division down: what Cymbal buys is the network's *reach and execution*; what Cymbal keeps is the *account relationship, the FX decision, and the reconciliation*. The network is a wholesale utility, not a replacement bank — precisely the distinction §4.2 draws.

### 11.3 Licensing Boundaries and the MAS Scope

The second question is where Cymbal Bank's **MAS-scoped service ends and the network's licences begin** (§8). Structurally:

- **Nimbus's Singapore-funded payroll** is a cross-border money-transfer activity. If Cymbal Bank itself executed it, Cymbal would be performing cross-border money transfer under the Payment Services Act; the network, as an **MAS Major Payment Institution licensed for Cross-border Money Transfer Service** (§8.1), is the licensed party executing the payout leg. The bank's role — holding accounts, providing FX, reconciling — must be assessed on its own facts against the PS Act's regulated-activity definitions, using the scope analysis in the [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) guide and the [Singapore Fintech & Payments](singapore_fintech_payments_guide.md) guide §2. ✅ (frame).
- **The other markets** are governed by the network's own licences (UK API, France ACPR, Hong Kong MSO, US state money transmitters — §8.3), each subject to that jurisdiction's rules. The bank does **not** assert the network's French, Hong Kong, or US permissions as its own, and it documents that Nimbus's non-Singapore activity is executed under the network's licences.
- **The December 2025 IPA (§8.2)** is a forward-looking expansion of the network's Singapore licence (account issuance, domestic transfer, merchant acquisition, e-money). A bank reviewing the network should treat the IPA as *pending* — an in-principle approval is not a grant — and re-check the MAS register before relying on the expanded activities.

The honest, audit-ready position for Cymbal Bank: **the bank's services are MAS-scoped and compliant on their own facts; the network executes the cross-border legs under its own licences; and the boundary between the two is documented, not assumed.** This is the same boundary discipline the [Adyen](adyen_guide.md) §11.2 worked example applies to an acquiring platform, here applied to a payout network.

### 11.4 Safeguarding, Settlement, and Prefunding

The third question is the money mechanics — where the funds sit and when they move:

- **Prefunding.** To pay out instantly in eight markets, the network must hold **prefunded balances** in local-currency accounts or with local partners (the SmartX treasury layer, §9.2). Nimbus's SGD is converted and pushed into those balances ahead of the payouts. Cymbal Bank's exposure is at the *funding* step: it debits Nimbus's SGD account and delivers to the network's settlement account, so Cymbal's credit exposure is to Nimbus (its client) plus, if it extends any settlement or intraday facility to the arrangement, to the network as a counterparty.
- **Settlement cycle.** The network settles with Nimbus on an agreed cycle (e.g., T+0/T+1 for the funding leg), and pays out to beneficiaries on the local rails' cycles. The clearing-and-settlement mechanics for the payout legs are documented in the [Payment Rails](payment_rails_guide.md) guide §5 and are **cross-referenced, not re-derived here**.
- **Safeguarding.** As an MAS Major Payment Institution, Thunes Asia must safeguard customer money (the MPI safeguarding obligation the [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) guide documents). For Cymbal, the practical questions are: whose money is safeguarded at what moment (Nimbus's, before payout? the beneficiary's, after?), and how the bank's own reconciliation captures the safeguarding boundary. The guide states the obligation as a licence requirement sourced to the MAS frame; it does **not** assert the network's specific safeguarding arrangement, which is not published. ✅ (obligation) / ⚠ (network's specific arrangement).

### 11.5 AML and Sanctions Screening at Both Ends

The fourth question is who screens what. In a two-institution flow there are **two screening populations**, and dividing them is the substance of the control design:

- **The network screens the transaction** (Fortress layer, §9.3) at the point it executes the payout, against sanctions and AML rules across its endpoints.
- **Cymbal Bank screens its relationship and its account flows** — Nimbus's corporate KYC (beneficial owners, business-risk profile), and monitoring of the funding movements in Nimbus's Cymbal accounts against the client's stated business.
- **Nimbus, as the client, screens its own beneficiaries** — worker and supplier identity, and sanctions on its payees.

The overlap risk is either *under-screening* (each side assuming the other covered a population) or *over-screening* (duplicative false positives slowing payouts). Cymbal's control documentation should state, per population, which party screens and at what point. The screening disciplines — list management, name matching, false positives, transaction filtering — are the themes of the [Fircosoft](fircosoft_guide.md) guide (§3–§5), **cross-referenced here, condensed**. For the payroll use case specifically, the bank should also consider the **beneficiary-tier** risk: thousands of low-value payouts to workers the bank did not onboard, which the bank monitors in aggregate (velocity, geography, structuring patterns) rather than individually — the same multi-tier control logic the [Adyen](adyen_guide.md) §11.4 worked example applies to marketplace sellers.

### 11.6 FX Pricing and the Spread

The fifth question is the most commercially charged: **who takes the FX spread?** A cross-border payout is a chain of conversions — SGD → USD (or a settlement currency) → the receiving local currency — and each step can carry a spread (the §4.4 revenue line). Cymbal Bank's position:

- Cymbal can **compete for the SGD-side conversion** — pricing Nimbus's SGD→settlement-currency leg on the bank's own FX desk — which keeps a slice of the margin with the bank.
- The **receiving-market conversion** (settlement currency → local currency) is the network's (its local liquidity, its spread).
- The bank's job in the negotiation is therefore twofold: **price the leg it can win**, and **disclose what the client is paying across the leg it cannot** — because a payroll provider that cannot see its total FX cost is exposed to spread drift that quietly eats its margin. The reconciliation overlay (§11.7) is what makes the total cost visible. ✅ (structure) / ⚠ (rates not published by the network).

### 11.7 Coverage, Corridor Risk, Reconciliation, and Returns

The sixth and seventh questions are operational:

**Coverage and corridor risk.** The network's "130+ countries / 90+ currencies" claims (§3.2) are aggregate; what matters for Nimbus is the **per-corridor** reality — whether each of the eight markets is served directly, which endpoints are reachable, what the cut-off times are, and what happens when a corridor degrades (a local partner outage, a cash-agent failure, a wallet downtime). The bank's risk documentation should record the corridor map it has verified with the network (not the marketing total) and the fallback for each corridor. The published per-corridor map is **not available** (§9.5), which is itself a diligence finding: Nimbus should obtain it contractually.

**Reconciliation.** The bank's overlay compares, daily: (a) the network's payout reports (via the API feed), (b) the funding debits in Nimbus's Cymbal accounts, and (c) Nimbus's own payroll ledger. This three-way check is what turns thousands of small payouts into a verifiable control — the same "does the ledger tell the truth at month-end?" discipline the [Reap Global](reap_global_guide.md) §11.4 worked example closes with, applied to payout rather than card spend.

**Returns and recalls.** Cross-border payouts fail: an invalid bank account, a closed wallet, an unclaimed cash payout, a beneficiary name mismatch. Each failure creates a **return** that must flow back through the network to Nimbus's account, with the FX and fees reversed or retained. The bank's overlay must track the **return rate per corridor and per endpoint type** — cash-pickup returns behave differently from wallet returns — because a rising return rate is both an operational-cost signal and a data-quality signal (bad beneficiary data inflates returns). The mechanics of the return depend on the receiving rail, cross-referenced to the [Payment Rails](payment_rails_guide.md) guide, not re-derived here.

### 11.8 API Integration, Resilience, Concentration, and Exit

The eighth and ninth questions are the architect's and the risk officer's:

- **Integration.** The network is consumed as an API (§9.4): one integration for all corridors, with payout instructions, status callbacks, and reporting. Cymbal (or Nimbus) integrates once and manages corridors through configuration. The integration-architecture themes — idempotent instruction submission, event-driven status handling, reconciliation feeds, and error/retry semantics — are the [Enterprise Middleware & Integration Platforms](../technology/enterprise_middleware_integration_platform_guide.md) guide's subject, cross-referenced, not re-derived. The bank's due diligence on the integration should include the **resilience** question: how the network's API behaves under corridor failure, and whether the client's payroll run has a fallback path when a corridor is down.
- **Counterparty and concentration risk.** Routing an entire payroll book through one network concentrates operational and counterparty risk: if the network or a corridor partner fails, Nimbus's payroll fails. The bank's review should therefore weigh (a) the network's own financial standing — recalling that its only public financial datapoint is a company-reported revenue run-rate and positive EBITDA (April 2025, §4.4), with no audited figures — (b) its licence footprint (§8) and its MAS register status, and (c) the **concentration** of Nimbus's flows in one provider.
- **Exit planning.** A well-documented arrangement includes an exit path: what data the client can retrieve (beneficiary records, historical payout reports), how funding balances are unwound, and how the bank's reconciliation overlay can be repointed to an alternative network or to the bank's own rails. Exit planning is cheap to write and expensive to omit.
- **Governance note.** Thunes' leadership changed CEO three times in six years and refreshed its CFO and CTO/CPO in March 2026 (§2.5) — a continuity factor the bank's counterparty review should record, without treating a change of management as a credit event in itself.

### 11.9 The Lessons

The worked example yields six lessons for a bank meeting a Thunes-style payout network:

1. **A network is a utility, not a bank.** The bank buys reach and execution; it keeps the account relationship, the FX decision, and the reconciliation — and should write down which is which (§11.2).
2. **The licensed boundary is the whole game.** The network's MAS/UK/FR/HK/US licences carry the cross-border legs; the bank's own services must be scoped and documented independently (§11.3).
3. **Prefunding is the credit question.** The bank's exposure sits in the funding and settlement leg, and the safeguarding boundary must be visible in the reconciliation (§11.4).
4. **Two screeners, one population map.** Under- and over-screening are both failure modes; the bank documents who screens which tier (§11.5).
5. **The spread is negotiable and must be visible.** The bank prices the leg it can win and discloses the total FX cost via the overlay (§11.6–§11.7).
6. **Concentration and exit are first-class risks.** One network carrying a whole payroll book needs a documented exit path and a hard look at the only public financial datapoint — a company-reported run-rate, not an audited figure (§11.8).

Everything in this example that touches real licences is grounded in the verified record of §8; everything else is fictional and flagged as such — the repository's honesty convention, applied.

---

## 12. The Claims Audit — Verified, Flagged, Rejected

### 12.1 The Verified Claims (✅)

| # | Claim | Verification |
| --- | --- | --- |
| 1 | TransferTo founded 2005; rebranded into DT One and Thunes on 18 February 2019; Thunes cross-border business "started in 2016" | Thunes rebrand release (18 Feb 2019); Wikipedia ✅ |
| 2 | Thunes Asia Private Limited is an MAS Major Payment Institution (Cross-border Money Transfer Service) | MAS FID (extracted live, Oct 2026) ✅ |
| 3 | Registered office 1 Raffles Place #28-61, Singapore 048616; key personnel De Caluwe Peter Irene F | MAS FID ✅ |
| 4 | Peter De Caluwe is the documented co-founder (Co-Founder and CEO) | Company releases; about-us page ✅ |
| 5 | Steve Vickers hired as CEO (May 2019); De Caluwe CEO by July 2021/April 2022; Floris de Kort CEO from 9 Jan 2024 (last day 24 Oct 2025); De Caluwe returned as CEO | Thunes releases (2019, 2021, 2022, 2024, 2025, 2026) ✅ |
| 6 | Allan Green is Chairman | Company ✅ |
| 7 | Series A US$10m, 6 May 2019, led by GGV Capital | Thunes release; TechCrunch (5 May 2019) ✅ |
| 8 | Series B/growth round US$60m, May 2021, led by Insight Partners; total capital US$130m | Thunes/Limonetik release; TechCrunch (18 May 2021) ✅ |
| 9 | Series C US$72m, announced July 2023, at >US$900m post-money; first close US$60m June 2023; led by Marshall Wace (Bessemer, 01Fintech, Visa, EDBI, Endeavor Catalyst); prior valuation ~US$794m (PitchBook) | TechCrunch (17 Jul 2023) ✅ |
| 10 | Series D US$150m, 28 April 2025, led by Apis Partners and Vitruvian Partners; Proton Partners adviser | Thunes release; PRNewswire (28 Apr 2025) ✅ |
| 11 | Limonetik acquired (announced 21 Jul 2021); Paris, founded 2008; rebranded Thunes Collections | Thunes release; Electronic Payments International ✅ |
| 12 | Tookitaki: majority stake of over US$20m announced 19 April 2022; businesses "continue to operate independently"; founded November 2014; 100+ staff; founder/CEO Abhishek Chatterjee | Thunes release (thunes.com/news/thunes-tookitaki/); Tookitaki blog ✅ |
| 13 | Tilia LLC acquisition agreement announced 23 April 2024; 48 US states/territories; majority owner Linden Research; five-year Linden Lab partnership | Thunes release; Linden Lab press page ✅ |
| 14 | UK: Authorised Payment Institution, FCA firm reference #720167 | Thunes Series A release (6 May 2019) ✅ |
| 15 | MAS In-Principle Approval for an MPI licence variation (account issuance, domestic transfer, merchant acquisition, e-money) announced 2 December 2025 | Thunes release (2 Dec 2025) ✅ |
| 16 | Company positioning language: "Smart Superhighway to move money around the world"; "proprietary Direct Global Network"; "in-house SmartX Treasury System and Fortress Compliance Platform" | Company boilerplate ✅ |
| 17 | New CFO (Parvinder Bhatia) and CTPO (Guy Duncan) appointed 12 March 2026 | Thunes release (12 Mar 2026) ✅ |
| 18 | Thunes is absent from the payments-firm table of singapore_fintech_payments_guide.md §3 | Repository grep (0 matches) ✅ |

### 12.2 The Flagged Claims (⚠)

| # | Claim | Why flagged |
| --- | --- | --- |
| 1 | 2020 Series B led by Helios Investment Partners (with Checkout.com, GGV) — amount | Wikipedia states the round; **amount not verified** |
| 2 | Tilia acquisition completion/closing date | Announcement verified; close unconfirmed (Wikipedia "June 2025"; not dated by primary) |
| 3 | Revenue run-rate US$150m and positive EBITDA | Company-reported (April 2025); no audited figures exist |
| 4 | Corridor/currency/method counts (80+ countries 2019 → 140+ countries 2026) | All company claims of their dates; volatile, definition-dependent |
| 5 | "Over 50 financial service licenses across the globe" (2 Dec 2025) | Company claim; the guide verifies only the MAS entry |
| 6 | US money-transmitter licences across 50 states | Company claim |
| 7 | France ACPR payment-institution licence; Hong Kong MSO licence | Company-stated; register not queried this pass |
| 8 | Named Members (Uber, Deliveroo, UberEats, MoneyGram, Western Union, Remitly, Revolut, PayPal, Singtel Dash, M-PESA, Airtel, Grab, WeChat) | Company-named; not re-verified per name |
| 9 | Named partners (Mastercard, Visa, PayPal-Hyperwallet, Swift, Ripple, Ecobank, Absa, J.P. Morgan, Fiserv; Circle USDC Oct 2024; EURC Aug 2026) | Company-published; not re-verified per item |
| 10 | "13 locations" office list (April 2025) | Company boilerplate; moving number |
| 11 | "180 million transactions annually" (April 2022); ">US$50bn processed to date" (July 2023) | Company claims |
| 12 | SmartX and Fortress architectures | Product names only; internals undisclosed |
| 13 | Peter De Caluwe's first appointment as CEO | Verified "by July 2021" and April 2022; exact date not pinned |
| 14 | Thunes' safeguarding arrangements and prefunding structure | Not published |

### 12.3 The Rejected or Not-Found Claims (❌)

| # | Claim | Finding |
| --- | --- | --- |
| 1 | **Tookitaki was "acquired by Thunes" (clean acquisition/exit)** | **Corrected — it was a majority-stake investment of over US$20m (19 April 2022), and the companies "continue to operate independently."** The repository file `technology/singapore_saas_companies_guide.md` (lines ~46, 52, 83, 107, 211, 238, 402, 495, 535, 551, 617, 648) asserts the clean acquisition and marks it Verified — the dispatcher should correct that file to the majority-stake/alliance position |
| 2 | **Tookitaki "founded 2012"** | **Corrected — Thunes' own release states Tookitaki "was founded in November 2014."** The repository's "founded 2012" (and its flagged "2014 per one tracker") should be reconciled to the November 2014 date |
| 3 | Thunes as a clean **legal spin-out** from TransferTo | Not documented; framed as a rebrand into two independently-operated brands (transferto.com still lists De Caluwe as CEO of TransferTo) |
| 4 | Any co-founder other than Peter De Caluwe | None found; Eric Barbier founded TransferTo/DT One, not documented as a Thunes co-founder |
| 5 | Thunes listed in singapore_fintech_payments_guide.md §3 | Documented absence (0 matches) — a gap, not a rejection of Thunes' existence |
| 6 | A retrieved secondary web_search result set for this topic | One web_search batch returned empty results on this host (tool limitation, §12.4) |

### 12.4 What Could Not Be Verified

This section collects, per the repository's honesty convention, every material item this guide could not confirm — each is deliberately **not** asserted as fact anywhere above:

- **The amount of the 2020 round** (Wikipedia's Series B led by Helios Investment Partners, with Checkout.com and GGV). The round and leads are stated by an encyclopedic source; the **amount was not verified** at any primary source. Helios is a confirmed investor, but no figure is asserted here.
- **The Tilia acquisition completion/closing date.** The 23 April 2024 agreement is verified at primary sources; the close is not. Wikipedia loosely says "June 2025"; the April 2025 Series D release refers to a "recent acquisition of licenses across 50 U.S. States, subject to regulatory approval" without dating a closing. No closing date is asserted.
- **The exact date Peter De Caluwe became CEO.** Verified only as "by July 2021" (Limonetik release) and confirmed by April 2022 (Tookitaki release). The Italian trade press's "from 2017" conflicts with the May 2019 release naming Steve Vickers as CEO, and the conflict is left unresolved.
- **Whether Thunes is a legal spin-out from TransferTo or an independently-operated business unit of a group.** The sources examined support "rebranded into two independently-operated brands" and nothing stronger; this guide does not assert a legal spin-off.
- **Thunes' audited financials.** It is a private company; there are **no audited figures**. The only public financial datapoints are the company-reported revenue run-rate (US$150m) and "positive EBITDA" (April 2025).
- **The internal architecture of SmartX and Fortress.** Both are company product names; their internals, and the per-corridor rail/partner map, are not published.
- **The per-corridor map** — which local clearing system, wallet, or cash network serves which receiving market, and the endpoints reachable in each.
- **The current status of the UK API (#720167), the French ACPR licence, the Hong Kong MSO licence, and the US state money-transmitter registrations** — stated by the company and, for the UK, in a 2019 release; the non-Singapore registers were not queried in this pass.
- **Per-name verification of company-named Members and partners** (§12.2 items 8–9).
- **Independent market-share estimates** for Thunes in any segment — none verified, none asserted.
- **Tool limitation:** **one research pass of web_search on this host returned empty results**, so several secondary questions were resolved only from primary company pages and encyclopedic anchors. This is a limitation of the tooling at that moment, **not evidence that the material does not exist**. Where a page would not extract or a search returned nothing, the item is recorded here as unverified rather than sourced from memory.

---

## 13. The Glossary

| Term | Meaning |
| --- | --- |
| **ACCEPT** | Thunes' collection product: letting Members collect funds from payers in local markets using local payment methods. |
| **ACPR** | The French Prudential Supervision and Resolution Authority; the French Payment Institution licence holder's supervisor (company-stated). |
| **API (Application Programming Interface)** | The programmatic interface through which Members submit payout/collection instructions and receive status and reporting. |
| **APM (Alternative Payment Method)** | A non-card local payment method (wallet, local transfer, QR, etc.) used in cross-border payouts and collections. |
| **Authorised Payment Institution (API)** | A UK FCA licence category; Thunes' UK authorisation is stated with FCA firm reference #720167 (6 May 2019 release). |
| **Bank account (as endpoint)** | A payout endpoint delivering funds to a beneficiary's bank account in the receiving market. |
| **Beneficiary** | The end recipient of a payout (a worker, contractor, or supplier); not a Thunes customer — the Member faces the beneficiary's sender. |
| **Cash pickup** | A payout endpoint delivering funds to a physical cash-out outlet, for beneficiaries without bank or wallet access. |
| **Circle / USDC / EURC** | Stablecoin issuer and stablecoins; Thunes' public stablecoin items are a USDC partnership (Oct 2024) and EURC euro treasury (Aug 2026). ⚠ |
| **Collection** | Receiving funds from payers in a market (the ACCEPT direction), versus payout (the PAY direction). |
| **Corridor** | A send-market-to-receive-market pair with its own rails, FX, endpoints, and licensing; the operational unit of network coverage. |
| **Cross-border Money Transfer Service** | The regulated activity (Singapore PS Act) on Thunes Asia's MAS Major Payment Institution register entry. ✅ |
| **Direct Global Network** | Thunes' name for its network of direct connections to local payment brands, as opposed to correspondent chains. ✅ |
| **dLocal / Nium / Airwallex / TerraPay / Payoneer / Rapyd / CurrencyCloud / Ebury / Moov / Flutterwave** | Structurally comparable B2B cross-border networks/platforms named in §10.1 (unranked). |
| **DT One** | The mobile top-up and rewards brand created alongside Thunes in the 18 February 2019 TransferTo rebrand. |
| **EDBI** | Singapore's EDB venture arm; a Series C participant (July 2023). ✅ |
| **Fortress Compliance Platform** | Thunes' name for its in-house compliance layer. ✅ (internals undisclosed ⚠) |
| **Fircosoft** | A sanctions-screening software vendor; the repository's reference guide for screening themes (cross-ref §8.4, §11.5). |
| **FX spread** | The margin between the rate applied to a cross-border conversion and the rate at which liquidity is sourced; a core network revenue line. |
| **GCash / M-Pesa / Airtel / MTN / Orange / JazzCash / Easypaisa / AliPay / WeChat Pay** | Local wallets/methods named in Thunes boilerplate as reachable endpoints. ⚠ |
| **GGV Capital** | Lead investor in the 2019 Series A (Jenny Lee). ✅ |
| **Helios Investment Partners** | A confirmed Thunes investor; a Wikipedia-stated 2020 Series B lead (amount unverified). ⚠ |
| **Insight Partners** | Lead investor in the May 2021 US$60m round. ✅ |
| **IPA (In-Principle Approval)** | A conditional MAS approval; Thunes announced an IPA for an MPI licence variation on 2 December 2025 — an IPA is not a grant. ✅ |
| **Linden Lab / Tilia LLC** | Maker of Second Life / the US games-payments platform whose acquisition Thunes announced on 23 April 2024 (close unconfirmed). ✅/⚠ |
| **Limonetik** | The Paris payment-methods platform acquired by Thunes (announced 21 July 2021) and rebranded Thunes Collections. ✅ |
| **Major Payment Institution (MPI)** | The upper MAS licence tier under the PS Act; Thunes Asia's licence type. ✅ |
| **Member** | A participant in the Thunes network (banks, wallets, PSPs, MTOs, gig platforms, etc.) who buys network access. |
| **MTO (Money Transfer Operator)** | A remittance business facing the end customer; named Thunes Members include Western Union, MoneyGram, Remitly. |
| **Ogone / Naspers / PayU** | Firms in Peter De Caluwe's career before Thunes (company bio). ✅ |
| **PAY** | Thunes' payout product: cross-border disbursement to bank, wallet, card, or cash endpoints. |
| **Prefunding** | Holding balances in receiving markets in advance so instant payouts can be funded locally — the SmartX treasury role. |
| **Proton Partners** | Financial adviser on the April 2025 Series D. ✅ |
| **Remittance** | A cross-border transfer, typically person-to-person; a core use case for payout networks. |
| **Safeguarding** | The MAS requirement that payment institutions protect customer money (segregation/insurance) — an MPI obligation. ✅ (obligation) |
| **SmartX Treasury System** | Thunes' name for its in-house treasury system. ✅ (internals undisclosed ⚠) |
| **Stablecoin** | A cryptocurrency pegged to a stable asset (e.g., USDC, EURC); referenced in Thunes' treasury/settlement items. ⚠ |
| **Superhighway** | Thunes' own metaphor: "the Smart Superhighway to move money around the world." ✅ |
| **SWIFT gpi / Visa Direct / Mastercard Send** | Card-rail and correspondent alternatives named in §10.1 (cross-ref [SWIFT Alliance Access](swift_alliance_access_guide.md)). |
| **Tilia** | See Linden Lab / Tilia LLC. |
| **Tookitaki** | Singapore AML/compliance regtech (founded November 2014); Thunes took a majority stake of over US$20m (19 April 2022). ✅ — an investment, not an acquisition (§7.3, §12.3). |
| **TransferTo** | The Singapore mobile-payments company (founded 2005) that rebranded into DT One and Thunes (18 February 2019). ✅ |
| **Vickers / De Kort / De Caluwe** | The CEO sequence: Steve Vickers (May 2019–), Peter De Caluwe (by 2021–Jan 2024), Floris de Kort (9 Jan 2024 – 24 Oct 2025), De Caluwe (returned 2025). ✅ |
| **Wallet** | A mobile-money or digital wallet (e.g., GCash, M-Pesa); a payout endpoint type. |
| **WorldRemit / Zepz** | A remittance operator named in §10.1 and, via Andrew Stewart's background, connected to the leadership roster. |

---

## 14. Cross-References and the Closing Summary

**Cross-references (repository convention: sibling `banking/` guides by plain filename; other folders prefixed):**

- [Adyen](adyen_guide.md) — the acquiring/issuing platform-company genre precedent and the marketplace worked-example conventions; cited in §4, §10, §11.5.
- [Reap Global](reap_global_guide.md) — the fintech-company profile genre, the Cymbal Bank worked-example conventions, and the honesty framing; cited in §1, §11.7.
- [Singapore Fintech & Payments](singapore_fintech_payments_guide.md) — the PSA regulatory frame (§2), the payments firms (§3, where **Thunes is absent** — a finding), and the licensing journey (§9); cited in §8.
- [Payments Hub](payments_hub_guide.md) — hub models and interoperability; cited in §4.
- [Payment Rails](payment_rails_guide.md) — clearing and settlement mechanics for the payout and collection legs (§5); cited in §5, §6, §11.4, §11.7.
- [ISO 20022 Core Processes](iso_20022_core_processes_guide.md) — the message-standard layer for cross-border instructions; cited in §9.4.
- [SWIFT Alliance Access](swift_alliance_access_guide.md) — the correspondent-messaging alternative; cited in §9.4, §10.
- [IBPS Payment Connect](ibps_payment_connect_guide.md) — a bank-side payout-channel analogue; cited in §9.4.
- [Fircosoft](fircosoft_guide.md) — sanctions/AML screening themes (§3–§5), cross-referenced condensed; cited in §8.4, §11.5.
- [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) — the PS Act/MPI regime and the Cymbal Bank persona conventions; cited in §1, §8, §11.
- [Mojaloop](mojaloop_guide.md) — interoperability architecture; cited in §9.1.
- [Airwallex](airwallex_guide.md) — a B2B cross-border platform peer; cited in §10.
- [Enterprise Middleware & Integration Platforms](../technology/enterprise_middleware_integration_platform_guide.md) — API/integration-platform themes; cited in §5, §9, §11.8.

**Primary sources used this pass:** thunes.com — rebrand release (18 Feb 2019); Series A release (6 May 2019); Limonetik acquisition release (21 Jul 2021); Tookitaki majority-stake release (19 Apr 2022, extracted live); Tilia agreement release (23 Apr 2024); Series D release (28 Apr 2025, extracted live); MAS MPI expansion IPA release (2 Dec 2025); executive appointments release (12 Mar 2026); leadership release (9 Jan 2024); about-us page (extracted Oct 2026); newsroom listing (thunes.com/news) for 2026 partnerships and footprint events; company boilerplate (verbatim, Series D release). Regulator: the MAS Financial Institutions Directory entry for Thunes Asia Private Limited (eservices.mas.gov.sg/fid, extracted live). Press/secondary: TechCrunch (5 May 2019; 18 May 2021; 17 July 2023); PRNewswire (28 Apr 2025); the Tookitaki blog (thunes-majority-stake post); Wikipedia (Thunes article, retrieved 2026); PitchBook (prior valuation, via TechCrunch). Cross-referenced repository guides as listed above. The 2020 Series B amount, the Tilia closing date, non-Singapore register entries, and per-corridor maps were **not** verifiable in this pass and are recorded in §12.4.

**The closing summary.** Thunes is the cross-border payments business that began inside TransferTo (founded 2005), started its cross-border operations in 2016, and emerged as its own brand alongside DT One on 18 February 2019. In the years since, it assembled a **proprietary Direct Global Network** of direct connections to local payment endpoints, split its product into **PAY** (payouts) and **ACCEPT** (collections) with **Thunes Collections** (the acquired Limonetik platform), and took its licences in Singapore (**MAS Major Payment Institution**, Cross-border Money Transfer Service), the UK (**FCA Authorised Payment Institution #720167**), France, Hong Kong, and the US states. It raised a US$10 million Series A (2019), a US$60 million round (2021), a US$72 million Series C at over US$900 million post-money (July 2023), and a US$150 million Series D (28 April 2025) on a company-reported **US$150 million revenue run-rate and positive EBITDA** — the only public financial datapoint, and company-reported at that. Its deal history is three transactions with three different structures: **Limonetik acquired** (2021), a **majority stake of over US$20 million in Tookitaki** (2022) — an investment in which the businesses "continue to operate independently," correcting the repository's "Tookitaki was acquired by Thunes" and "founded 2012" claims — and an **agreement to acquire Tilia LLC** (2024) whose closing date remains unconfirmed. What could not be verified — the 2020 round's amount, the Tilia close, De Caluwe's first CEO date, the spin-out question, audited financials, the SmartX/Fortress internals, the per-corridor map, and one empty web_search pass — is flagged rather than smoothed, per this repository's honesty convention. For Cymbal Bank, the takeaway is structural: cross-border payout and collection reach has become a **wholesale network utility** that a bank can buy rather than build, and the bank's role in that world is decided by how well it draws the licensing boundary, funds and safeguards the settlement leg, splits the screening populations, negotiates the FX spread, and reconciles what the network reports against what the bank's own accounts show. Every payout, every collection, and every corridor eventually resolves into the same place — the ledger.
