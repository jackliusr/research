# Reap: The Stablecoin-Native Corporate Card and Payments Infrastructure Company — A Comprehensive Guide

**The Business, History, Products, Funding, Licensing, and Technology of Reap (reap.global) — from Hong Kong Credit-Card Bill Payments (2018) to Stablecoin-Powered Card Issuing, the $600 Million Payward (Kraken Parent) Acquisition (2026), and a Cymbal Bank Corporate-Spend Worked Example**

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Banking Domain / Fintech Company Deep-Dive — corporate cards and card-issuance platforms, spend management, cross-border payments, stablecoin settlement, Visa principal membership, Hong Kong MSO licensing, the MAS regime comparison, and the Cymbal Bank corporate-spend lens
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026
> **Companion guides (sibling, same folder — the payments cluster):** [Payment Rails](payment_rails_guide.md) (cross-border rails, clearing and settlement mechanics — cross-ref §6, §7, §9) · [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) (the Singapore licensing regime and the Cymbal Bank persona conventions — cross-ref §9) · [Ancillary Revenue Products](ancillary_revenue_products_guide.md) (card-program economics — cross-ref §6) · [Fircosoft](fircosoft_guide.md) (sanctions/AML screening themes — cross-ref §9) · [Citadel LLC](citadel_llc_guide.md) and [Resona Merchant Bank Asia](resona_merchant_bank_asia_guide.md) (the genre precedents for this guide's structure and honesty framing)
> **Companion guides (technology/, prefix `../technology/`):** [Enterprise Middleware & Integration Platforms](../technology/enterprise_middleware_integration_platform_guide.md) (API/integration-platform themes — cross-ref §8) · [Financial Management Systems](../technology/financial_management_systems_guide.md) (accounting/ERP systems — cross-ref §7)

---

**How to use this guide:** Section 1 is the overview — the short answer, the key-facts table, and why a bank should care. Section 2 is the company profile — the 2018 founding, the two verified co-founders (Daren Guo and Kevin Kang), the origin story, and the recognition record. Section 3 is geography and footprint — the Hong Kong headquarters, the Singapore office history, the later markets, and the 2026 regional teams. Section 4 is the mission and business model — from card-based bill payments to stablecoin-native infrastructure. Section 5 is funding *and* the acquisition — the 2019 seed, the 2020 seed extension, the US$40 million Series A of October 2022, the growth metrics, and the up-to-US$600 million Payward acquisition of 2026 that made Reap part of the Kraken parent's family. Section 6 covers the card products and card-issuing economics (cross-referencing the ancillary-revenue guide's card-program economics rather than re-deriving them). Section 7 covers payments, expense management, and accounting integrations (cross-referencing the payment-rails and financial-management-systems guides). Section 8 covers the API platform and embedded finance (cross-referencing the middleware guide). Section 9 is licensing and compliance — Hong Kong MSO and Visa principal status, Mexico money-transmitter registration, the MAS question in Singapore, and the AML/KYC posture (cross-referencing the Fircosoft guide's screening themes, condensed). Section 10 is technology — the public facts about the platform, the partner-ecosystem chronology, and everything not disclosed flagged ⚠. Section 11 is the Cymbal Bank worked example — a fictional-but-substantiated corporate-spend client on a Reap-style program, following the persona conventions used across the repository. Section 12 is the claims audit (✅/⚠/❌), with §12.4 "What Could Not Be Verified" collecting every item that could not be confirmed. Section 13 is the glossary. Section 14 is cross-references and the closing summary. **Integrity convention:** ✅ = verified this pass against a primary or cited source (source named in §12); ⚠ = flagged/unverified — company self-reported, press estimate, or not re-verified live; ❌ = refuted or not found despite targeted search. Nothing in this guide was invented; figures that could not be re-verified are marked ⚠ and listed in §12.4. **A note on the brief:** the research brief that commissioned this guide assumed a Singapore headquarters, a Y Combinator affiliation, and co-founders including "Daniel Kuo" and "Adil Mohammed." Targeted verification against primary sources (reap.global, company press releases, the 2019–2022 terms of service, MAS registers, and the Wayback Machine) found the opposite on each count — the company is Hong Kong-founded and Hong Kong-headquartered, no YC affiliation could be found, and the only co-founders documented anywhere in primary sources are Daren Guo and Kevin Kang. Those corrections are made explicitly, not silently, in §2, §5, and §12.

---

## Table of Contents

1. [The Overview](#1-the-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Key-Facts Table](#12-the-key-facts-table)
   - 1.3 [Why This Matters to a Bank](#13-why-this-matters-to-a-bank)
   - 1.4 [The Evidence Base at a Glance](#14-the-evidence-base-at-a-glance)
2. [The Company Profile — Founding, Founders, and Origin](#2-the-company-profile--founding-founders-and-origin)
   - 2.1 [The Founding Year: 2018](#21-the-founding-year-2018)
   - 2.2 [The Co-Founders: Daren Guo and Kevin Kang](#22-the-co-founders-daren-guo-and-kevin-kang)
   - 2.3 [The Origin Story: A Toronto Production Company](#23-the-origin-story-a-toronto-production-company)
   - 2.4 [The "Other" Names: Daniel Kuo and Adil Mohammed](#24-the-other-names-daniel-kuo-and-adil-mohammed)
   - 2.5 [The Recognition Record](#25-the-recognition-record)
3. [Geography and Footprint](#3-geography-and-footprint)
   - 3.1 [Headquarters: Hong Kong (Not Singapore)](#31-headquarters-hong-kong-not-singapore)
   - 3.2 [The Singapore Office History](#32-the-singapore-office-history)
   - 3.3 [The Footprint Timeline](#33-the-footprint-timeline)
   - 3.4 [The 2026 Regional Teams](#34-the-2026-regional-teams)
4. [The Mission and Business Model](#4-the-mission-and-business-model)
   - 4.1 [The Mission Statement](#41-the-mission-statement)
   - 4.2 [The Model in Three Eras](#42-the-model-in-three-eras)
   - 4.3 [The Revenue Model](#43-the-revenue-model)
   - 4.4 [The Market Context Reap Cites](#44-the-market-context-reap-cites)
5. [Funding, Growth, and the Payward Acquisition](#5-funding-growth-and-the-payward-acquisition)
   - 5.1 [The 2019 Seed: US$1 Million, Fresco Capital](#51-the-2019-seed-us1-million-fresco-capital)
   - 5.2 [The 2020 Seed Extension: US$5 Million](#52-the-2020-seed-extension-us5-million)
   - 5.3 [The 2022 Series A: US$40 Million](#53-the-2022-series-a-us40-million)
   - 5.4 [The Y Combinator Question](#54-the-y-combinator-question)
   - 5.5 [Growth Metrics: Customers and Volume](#55-growth-metrics-customers-and-volume)
   - 5.6 [The Announcement: May 7, 2026](#56-the-announcement-may-7-2026)
   - 5.7 [The Closing: July 1, 2026](#57-the-closing-july-1-2026)
   - 5.8 [What the Deal Means](#58-what-the-deal-means)
6. [The Products — Corporate Cards and Card Issuing](#6-the-products--corporate-cards-and-card-issuing)
   - 6.1 [The Reap Card (2021–Present)](#61-the-reap-card-2021present)
   - 6.2 [The Card-Issuing Platform](#62-the-card-issuing-platform)
   - 6.3 [Card-Program Economics (Cross-Referenced)](#63-card-program-economics-cross-referenced)
7. [The Products — Payments, Expense Management, and Integrations](#7-the-products--payments-expense-management-and-integrations)
   - 7.1 [Reap Pay and Bill Pay — Cross-Border Payments](#71-reap-pay-and-bill-pay--cross-border-payments)
   - 7.2 [Expense Management](#72-expense-management)
   - 7.3 [Accounting and ERP Integrations (Cross-Referenced)](#73-accounting-and-erp-integrations-cross-referenced)
8. [The API Platform and Embedded Finance](#8-the-api-platform-and-embedded-finance)
   - 8.1 [The API-First Stack](#81-the-api-first-stack)
   - 8.2 [Embedded Finance](#82-embedded-finance)
   - 8.3 [Integration-Platform Themes (Cross-Referenced)](#83-integration-platform-themes-cross-referenced)
   - 8.4 [The 2026 Product-Surface Map](#84-the-2026-product-surface-map)
9. [Licensing and Compliance](#9-licensing-and-compliance)
   - 9.1 [Hong Kong: MSO Licence and Visa Principal Issuer](#91-hong-kong-mso-licence-and-visa-principal-issuer)
   - 9.2 [Mexico: Money Transmitter Registry and Visa Principal Member](#92-mexico-money-transmitter-registry-and-visa-principal-member)
   - 9.3 [Singapore: The MAS Question](#93-singapore-the-mas-question)
   - 9.4 [AML/KYC Posture (Cross-Referenced, Condensed)](#94-amlkyc-posture-cross-referenced-condensed)
   - 9.5 [The Three-Jurisdiction Licensing Comparison](#95-the-three-jurisdiction-licensing-comparison)
10. [Technology — Public Facts](#10-technology--public-facts)
    - 10.1 [The Platform Architecture Themes](#101-the-platform-architecture-themes)
    - 10.2 [The Verified Technology Facts](#102-the-verified-technology-facts)
    - 10.3 [What Is Not Disclosed](#103-what-is-not-disclosed)
    - 10.4 [The Partner-Ecosystem Chronology](#104-the-partner-ecosystem-chronology)
11. [The Cymbal Bank Worked Example — A Corporate-Spend Client on a Reap-Style Program](#11-the-cymbal-bank-worked-example--a-corporate-spend-client-on-a-reap-style-program)
    - 11.1 [The Scenario](#111-the-scenario)
    - 11.2 [The Card Program](#112-the-card-program)
    - 11.3 [Settlement and Treasury](#113-settlement-and-treasury)
    - 11.4 [Reconciliation and Accounting Sync](#114-reconciliation-and-accounting-sync)
    - 11.5 [KYC/AML and the Regulatory Overlay](#115-kycaml-and-the-regulatory-overlay)
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

**Reap** (reap.global; legal entity **Reap Technologies Limited**, Hong Kong) is a fintech company that began in 2018 as a Hong Kong credit-card bill-payment platform for small businesses and has evolved into a **stablecoin-native corporate card, card-issuing, and cross-border payments infrastructure company**. As of 2026 — following its acquisition by **Payward, Inc.**, the parent of the Kraken crypto exchange — Reap positions itself as the payments layer that connects card networks, traditional banking rails, and stablecoin settlement on a single API: corporate credit cards collateralised by stablecoins, card-issuing programs for other fintechs, cross-border payouts, and treasury management. The company was **founded and is headquartered in Hong Kong** (an office in Singapore existed from 2019, but Singapore was never the headquarters — the research brief's assumption to the contrary is corrected in §2–§3).

The arc is short but eventful: a 2018 founding in Hong Kong; a US$1 million seed (2019) led by Fresco Capital and a US$5 million seed extension (2020) backed by Global Founders Capital, Bertelsmann Asia Investments, and Index Venture; the launch of the **Reap Card**, a Visa corporate credit card, in November 2021; a **US$40 million Series A** in October 2022 led by Acorn Pacific Ventures, Arcadia Funds, and HashKey Capital; a pivot into stablecoin-powered products (USDC/USDT repayments via Fireblocks in 2023, Chainalysis-backed compliance in 2024, Solana-based treasury management in 2025); a Hong Kong global-headquarters expansion in November 2025; Mexico money-transmitter and Visa principal-member licenses in 2026; and finally the **Payward acquisition — up to US$600 million, announced May 7, 2026, and completed July 1, 2026** — making Reap the cards-and-payments arm of the Kraken parent's B2B infrastructure platform.

For a bank, Reap is interesting on four fronts: as a **model** (stablecoin-collateralised card programs challenge classic issuer economics), as a **counterparty/partner** (embedded card issuing, cross-border payouts), as a **regulatory weathervane** (what an MSO-plus-Visa-principal structure looks like versus the MAS licensed-PSP route), and as a **worked-example generator** for the Cymbal Bank persona used throughout this repository (§11).

### 1.2 The Key-Facts Table

| Aspect | Fact | Status |
| --- | --- | --- |
| Full name | Reap Technologies Limited (HK); operating as "Reap"; acquired by Payward, Inc. (2026) | ✅ |
| Founded | 2018, Hong Kong | ✅ |
| Co-founders | Daren Guo and Kevin Kang (the only co-founders documented in primary sources) | ✅ |
| "Daniel Kuo" / "Adil Mohammed" | Not found in any primary source examined | ❌ |
| Headquarters | Hong Kong — expanded global HQ at 1063 King's Road, Quarry Bay (Nov 2025) | ✅ |
| Y Combinator affiliation | No evidence found (YC directory search blocked; no press or company mention) | ❌ |
| Seed (2019) | US$1 million, led by Fresco Capital; Hustle Fund, K3 Ventures, Oyster Ventures | ✅ |
| Seed extension (2020) | US$5 million; Global Founders Capital, Bertelsmann Asia Investments, Index Venture, Fresco Capital, Hustle Fund | ✅ |
| Series A (Oct 2022) | US$40 million (equity + debt), led by Acorn Pacific Ventures, Arcadia Funds, HashKey Capital; returning Hustle Fund, Fresco Capital, Abacus Ventures; co-investor Payment Asia | ✅ |
| Reap Card launch | November 2021, Visa corporate credit card, Hong Kong | ✅ |
| Card network | Visa — Reap is a Visa Principal Issuer in Hong Kong and a Visa Principal Member in Mexico (May 2026) | ✅ |
| Customers | 20,000+ (company claim, Oct 2022) | ⚠ |
| Team size | ~40 (2022) → ~70 (Jan 2024) → 200+ (2025) → 300 (May/Jul 2026) | ✅/⚠ |
| Acquisition | Payward to acquire Reap for up to US$600 million (cash + stock), announced May 7, 2026; completed July 1, 2026 | ✅ |
| HK licensing | Reap (Remit) Limited — Money Service Operator licence no. 19-08-02816; Reap Technologies Limited — Visa Principal Issuer | ✅ |
| Mexico licensing | Money Transmitter Registry (Mar 2026); Vulnerable Activities Registrations (cards issuing, virtual assets); Visa Principal Member (May 2026) | ✅ |
| Singapore (MAS) | No MAS licence or exemption found; website states services "not directed at persons in Singapore" | ⚠/❌ |

### 1.3 Why This Matters to a Bank

For a bank like Cymbal Bank, the Reap story matters on four distinct fronts. **First**, as a competitive model: Reap demonstrates that a non-bank can assemble card issuing (Visa principal membership), money transmission (HK MSO, Mexico registry), and settlement (stablecoins) without a banking licence — the exact "unbundled issuer" pattern that banks increasingly meet in corporate spend. **Second**, as a counterparty/partner class: Reap-style card-issuing platforms are the infrastructure behind a growing share of fintech-branded corporate cards, and banks that want to stay in the issuance flow must interoperate with them (sponsoring, BIN sponsorship, settlement accounts). **Third**, as a regulatory reference: comparing Reap's Hong Kong/Mexico structure with the MAS licensed-payment-institution route (cross-ref [MAS Regulations](mas_regulations_guidelines_guide.md)) illuminates how the same product can be legal in one hub and out-of-scope or disclaimed in another — Reap's own website explicitly disclaims Singapore-directed services. **Fourth**, as a worked-example generator: §11 builds a Cymbal Bank corporate-spend client on a Reap-style program, from card issuance and settlement to reconciliation and KYC, following the persona conventions of [Citadel LLC](citadel_llc_guide.md) and [Resona Merchant Bank Asia](resona_merchant_bank_asia_guide.md).

### 1.4 The Evidence Base at a Glance

Every factual claim in this guide traces to one of five evidence classes, and the claims audit (§12) records which class supports each claim:

| Evidence class | Examples used in this guide | How it is treated |
| --- | --- | --- |
| Company press releases (reap.global/newsroom) | Series A (Oct 2022), Fireblocks (Jan 2023), Triple-A (May 2023), Chainalysis (Jan 2024), Solana treasury (Mar 2025), HK HQ (Nov 2025), Fincra MoU (Sep 2025), Global Dollar Network (Dec 2025), Mexico MTR (Mar 2026), Visa Mexico (May 2026), TerraPay (May 2026), USYC (Jun 2026), Payward acquisition + closing (May/Jul 2026), Hyperliquid (Aug 2026) | ✅ where the fact is the release's own content; ⚠ where the release reports company metrics |
| Company legal documents (via Wayback Machine) | Terms & Conditions (Dec 2022), About pages (Jul 2019, 2023, 2026) | ✅ primary-source legal statements (licence numbers, entity numbers) |
| Regulator registers | MAS PS Act exemption-notification register (extracted live) | ✅/❌ — the one register queryable this pass |
| Independent press | Bloomberg (May 2026), American Banker (Jul 2026), Forbes (Oct 2022), SCMP (Nov 2025), PYMNTS (Sep 2025), El Economista (May 2026) | ✅ for deal headlines and event existence; ⚠ for projections |
| The research brief itself | "Daniel Kuo"/"Adil Mohammed", YC affiliation, Singapore HQ, ~US$4.5M seed | ❌ where refuted; ⚠ where unconfirmable |

The methodological limitation of this pass is stated once here and not repeated: registry lookups that require interactive search (MAS Financial Institutions Directory, HK Customs MSO register, YC directory) could not be queried directly, and every negative finding that depends on them is flagged in §12.4 rather than asserted as proof.

---

## 2. The Company Profile — Founding, Founders, and Origin

### 2.1 The Founding Year: 2018

The founding year of **2018** verifies cleanly across multiple primary sources. The company's own Series A press release states "Since Reap's inception in 2018, it has focused on creating novel financial technologies to enable access to payables management, international payments and collections through a centralized software platform" (Reap press release, October 27, 2022). The current About Us page repeats it: "Before founding Reap in 2018, he was employee #90 at Stripe..." (reap.global/resources/company/about-reap). The 2020 seed announcement likewise describes the company as having been "initialized two years ago" (Jumpstart, July 14, 2020, citing company statements). The 2019 seed press release adds a datable institutional marker: Reap "joined the Cyberport (Hong Kong) Incubation Programme in October 2018" (Reap seed release, July 8, 2019, archived at riseconf.com). ✅ The legal entity — Reap Technologies Limited — was incorporated in Hong Kong under Company Registry number **2714427**, with a registered address at 25E, 23/F, One Taikoo Place, 979 King's Road, Quarry Bay, Hong Kong, per the company's own Terms & Conditions (December 2022 capture, Wayback Machine). ✅

### 2.2 The Co-Founders: Daren Guo and Kevin Kang

The verified co-founders are **Daren Guo** and **Kevin Kang** — the only two people ever named as co-founders in any primary source examined for this guide (2019 About page, 2019/2020 seed releases, the 2022 Series A release, the 2026 acquisition releases, and the current About Us page).

- **Daren Guo — Co-Founder.** The 2019 About page describes him as having "serendipitously joined the payments world as the first member of Stripe's Growth team in San Francisco before leading Stripe's launch in HK, Singapore and China" (Wayback capture, July 2019). The current About page says he was "employee #90 at Stripe, helping scale the company globally and launch its APAC operations" and that "Under Daren's leadership, Reap secured $40M in Series A funding" (reap.global, 2026). He is the public face of the company in virtually every press release, and remains CEO after the Payward acquisition ("maintaining its brand, leadership team led by Daren Guo" — Payward acquisition-completion release, July 1, 2026). ✅
- **Kevin Kang — Co-Founder.** The 2019 About page describes him as "a former finance and investment professional with experience in North America and Asia," with a masters degree in business from France and Singapore (Wayback capture, July 2019). He is quoted in the 2020 seed announcement ("We want to help businesses across Asia access these tools via a cloud-based platform...", Jumpstart, July 14, 2020), co-signs the 2022 Series A release ("Daren Guo & Kevin Kang, Co-Founders of Reap"), and appears in the 2024 Chainalysis release ("Kevin Kang, Co-Founder of Reap"). The current About page describes him as steering "strategy and operations," with a background in "Canadian investment banking and Southeast Asian infrastructure investment." ✅

Both co-founders share the origin story: they previously ran a small production company together (see §2.3). The 2019 seed release confirms: "The co-founders, Daren and Kevin, leveraged personal experience from a production company they previously ran together" (Reap seed release, July 8, 2019). ✅

### 2.3 The Origin Story: A Toronto Production Company

Reap's origin story is unusually well documented because the company told it on its own 2019 About page: in **2010** the founders started "a boutique production company specialising in wedding videography and photography in Toronto, Canada." The business hit the classic small-firm cash-flow wall — "Suppliers and employees demanded on time payments with no credit terms, and customers seemed to always pay late" — and, lacking the financial history for a loan, the founders relied on personal credit cards. The insight that became Reap: **a credit card is the most accessible form of short-term business credit, so let businesses pay any expense with one** — including expenses that merchants cannot accept cards for, by paying the recipient's bank account from the cardholder's card (Wayback capture of reap.global/about-reap/about-us, July 2019; the same narrative appears in the 2019 seed release). ✅ This "buyer-initiated payment" concept — turning cards into trade-financing facilities — is the intellectual root of everything Reap later built, up to and including the stablecoin-collateralised cards of 2026.

### 2.4 The "Other" Names: Daniel Kuo and Adil Mohammed

The research brief for this guide listed "Daniel Kuo" and "Adil Mohammed" as believed co-founders, and instructed that the claim be verified. It could not be: **no primary source examined names either person in connection with Reap**. The 2019 About page (which listed the whole early team — Daren Guo and Kevin Kang as co-founders, Dennis Tse and John Pham as software engineers, Guillaume Surrusca as marketing lead, and Jieren Chen as technical advisor) contains neither name; neither do the 2019, 2020, 2022, 2024, or 2026 company releases; neither does the current About page. Targeted searches for the names paired with "Reap" returned no corroborating coverage. The honest conclusion, recorded in §12.3 as ❌: the claimed co-founders could not be found in any primary source and appear to be a misremembering. (They are not, for example, officers on the public HK registry record examined via the company's own terms-of-service disclosures.) This guide therefore treats Daren Guo and Kevin Kang as the verified founding pair — the same pair the company itself has named consistently for seven years.

### 2.5 The Recognition Record

The current About page (reap.global/resources/company/about-reap, 2026) records third-party recognition that is checkable but mostly single-sourced (⚠ where noted):

- **Forbes Asia "100 to Watch" (2023):** "In 2023, Reap was named to Forbes Asia's 100 to Watch" — company page claim; the Forbes Asia list is real but Reap's inclusion is not independently re-verified this pass. ⚠
- **Tatler Gen.T Honouree:** Daren Guo "was recognised as a Gen.T Honouree by Tatler" — company page claim. ⚠
- **Founder press appearances:** Daren Guo "has been featured by Forbes Asia, The Economist, and more"; Kevin Kang "has represented Reap at DigFin, Cointelegraph, and fintech conferences worldwide" — consistent with the press coverage linked from Reap's own media page (Forbes' October 26, 2022 piece "Hong Kong Fintech Startup Reap Raises $40 Million For Web3 Payments" is linked there). ✅ for the Forbes piece's existence; ⚠ for the broader claim.
- **The 2021 Visa-launch press echo:** Hong Kong media covered the card launch (hk01, October 20, 2021: "Reap partners with Visa to launch smart corporate credit card to tackle capital turnover problems faced by SMEs"), linked from Reap's media page. ✅

None of this recognition changes any substantive fact in this guide; it is recorded because the repository's genre convention (per [Citadel LLC](citadel_llc_guide.md) and [Resona Merchant Bank Asia](resona_merchant_bank_asia_guide.md)) is to note what a company says about itself *and* how far verification could go.

---

## 3. Geography and Footprint

### 3.1 Headquarters: Hong Kong (Not Singapore)

The research brief assumed a Singapore headquarters with a Hong Kong presence. The verified record is the reverse: **Reap was founded in, and is headquartered in, Hong Kong**, with Singapore as one of its earliest *offices*. Evidence:

- The 2019 seed release: "Reap is a Hong Kong-based FinTech company" and "Based in Hong Kong, Reap's team includes a blend of key members from finance and marketing plus elite engineers from the payment and data security industries" (July 8, 2019). ✅
- The 2022 Series A release is datelined "27 October 2022, Hong Kong." ✅
- The November 5, 2025 release announces the "expansion of its global headquarters and new office opening in Hong Kong" — a 10,014 sq ft office at 1063 King's Road, Quarry Bay — with the boilerplate "Founded and headquartered in Hong Kong, Reap employs over 200 people worldwide." ✅
- The Payward acquisition releases (May and July 2026) repeat: "Founded and headquartered in Hong Kong, Reap employs 300 people worldwide." ✅
- The current site footer: "© 2026 Reap Technologies Limited" — the HK entity — and the disclaimer "This website and the services described herein are not directed at persons in Singapore." ✅

The 2025 HK expansion release explains the choice: Hong Kong offers "deep multicurrency liquidity, regulatory clarity, and global market connectivity," ranks "as the fourth largest foreign exchange hub worldwide, with over US$660 billion in average daily turnover (October 2024, TMA Semi-Annual Treasury Markets Survey 2024)," and is home to "more than 70 of the world's 100 largest banks (Hong Kong Monetary Authority 2025)." ✅ (company release, citing TMA/HKMA)

### 3.2 The Singapore Office History

Singapore was an early *office*, never the headquarters. The 2019 seed release notes Reap "is present in Hong Kong and Singapore" (July 2019). The 2020 seed coverage states the company "currently provides innovative financial technology solutions in Hong Kong, Singapore, and Vietnam, and will soon launch in Malaysia and South Korea as well" (Jumpstart, July 14, 2020). The 2022 Series A release lists the team as "spread across Hong Kong, Australia, Canada, Japan, Malaysia, Singapore and Vietnam" (October 27, 2022). ✅ Notably, the *services* side of the Singapore story has since been walked back in public: the 2026 website carries the disclaimer that its services "are not directed at persons in Singapore," and no MAS licence or exemption for a Reap entity could be found (see §9.3 for the full treatment). The honest summary: Singapore was an early Asian office market, but by 2026 Reap's regulated services are explicitly disclaimed there while Hong Kong and Mexico carry the licences.

### 3.3 The Footprint Timeline

| Period | Verified footprint | Source |
| --- | --- | --- |
| Oct 2018 | Joins Cyberport Incubation Programme (HK) | 2019 seed release ✅ |
| 2019 | Hong Kong + Singapore offices | 2019 seed release ✅ |
| 2020 | HK, Singapore, Vietnam; Malaysia and South Korea "soon" | Jumpstart, Jul 2020 ✅ |
| Oct 2021 | Visa card partnership announced; HK SME market focus | hk01, Oct 20 2021 ✅ |
| 2022 | Team of 40+ across HK, Australia, Canada, Japan, Malaysia, Singapore, Vietnam (7 countries) | Series A release ✅ |
| Jan 2024 | "Team of over 70, working across offices in eight countries" | Chainalysis release ✅ |
| Sep 2025 | Fincra MoU: Africa corridor for cards and cross-border payments | Fincra release + TechAfricaNews ✅/⚠ |
| Sep 2025 | US card-program expansion via processor partner Thredd (press-reported) | PYMNTS, Sep 17 2025 ✅/⚠ |
| Nov 2025 | Global HQ expanded in Hong Kong (Quarry Bay); 200+ people | HK HQ release ✅ |
| Mar 2026 | Local teams in Mexico, Brazil, Colombia (Latin America expansion) | Mexico release ✅ |
| May/Jul 2026 | 300 people worldwide; standalone brand under Payward | Acquisition releases ✅ |
| Aug 2026 | Hyperliquid integration live (USDC funding capability) | Hyperliquid release ✅ |

### 3.4 The 2026 Regional Teams

The About page (2026) names four regional leaders, which corroborates the "local teams" claim in the March 2026 Mexico release and shows how the LATAM expansion is staffed:

- **Jan Pekka Heinvirta** — Head of Mexico, Corporate Development.
- **Lucas Chaquea** — Head of LATAM, Corporate Development.
- **Jonatan Ojeda Serdan** — Regulatory Compliance Head, LATAM.
- **Arthur Ribeiro** — Head of Brazil, Corporate Development.

These names are from the company's own About page (✅ as published); their individual credentials and the size of their teams are not public ⚠. The November 2025 HK HQ release adds one verified headcount datapoint for the home market: Reap "grew its local Hong Kong team headcount by 58 per cent from the start of 2025 (as of September 2025)" ⚠ (company-reported).

---

## 4. The Mission and Business Model

### 4.1 The Mission Statement

Reap's mission, as stated on its current About page: **"To enhance the efficiency of money movement by bridging traditional financial infrastructure and the digital asset ecosystem."** The page's narrative elaborates: "Witnessing countless businesses hindered by outdated financial systems, we set out to create a more equitable landscape... Today, Reap stands as a catalyst for efficient, borderless payments... Our ultimate ambition? To craft a connected world that streamlines the efficient and unrestricted exchange of value, transcending all barriers for businesses worldwide" (reap.global/resources/company/about-reap, 2026). ✅ The Mexico release (March 2026) gives the same mission in slightly different words: "Our mission is to transform the financial landscape for efficient money movement, by merging traditional finance with digital assets, bridging disparate economies, and connecting key financial markets." ✅

### 4.2 The Model in Three Eras

Reap's business model has moved through three clearly distinguishable eras, each visible in the primary record:

1. **Era 1 — Card-based bill payments (2018–2020).** The original product let SMEs and individuals "pay any expenses with credit cards": the payer's card (Visa/Mastercard debit or credit) funded a payment to a recipient's bank account, in the recipient's preferred currency, cross-border if needed — "turning credit cards into trade-financing facilities" (2019 seed release). Products: **Reap Pay** (pay out) and **Reap Collect** (request payments). The 2020 coverage adds that Reap was "recognized by Visa as a Business Payments Solution Provider" and delivered its solution "alongside its technology partner, Stripe" (Jumpstart, July 14, 2020 — note that Daren Guo is a Stripe alumnus). ✅

2. **Era 2 — Corporate cards + spend management (2021–2023).** The November 2021 launch of the **Reap Card** — a HKD/USD-denominated Visa corporate credit card with bundled expense-management software — moved Reap from "pay bills with your card" to "here is a corporate card for your company." The 2022 Series A release describes "the industry's first Reap Visa Corporate Card ('Reap Card') platform" and a customer base split between Web2 clients (Acer, Buy and Ship, Pirata Group, Sompo Insurance, Wallem Group) and Web3 clients (Amber Group, Animoca Brands, Binance, Gnosis, LayerZero). ✅ The card page of the era (Wayback, 2023) adds the product details: free card, no annual fee, repay monthly statement "in fiat or crypto currency," virtual card on activation, worldwide shipping, contactless payments, and Xero sync. ✅

3. **Era 3 — Stablecoin-native infrastructure (2023–present).** From January 2023 (Fireblocks stablecoin repayments) through 2024 (Chainalysis compliance), 2025 (Solana treasury management, Global Dollar Network membership, HK HQ expansion, profitability), and 2026 (Mexico licences, Payward acquisition), Reap repositioned itself as "global financial infrastructure powered by stablecoins": stablecoin-collateralised USD/HKD Visa credit cards, stablecoin-funded fiat bill payments, card-issuing APIs, payments APIs, and an agentic-payments product (reap.global homepage and press releases, 2026). ✅

### 4.3 The Revenue Model

Reap has never published a full income statement, so the revenue model must be assembled from disclosed fragments (⚠ where inferred):

- **Interchange and scheme economics on the card portfolio.** As a Visa principal issuer, Reap earns interchange on card spend and pays scheme/processing fees — the standard issuer economics analysed in the [Ancillary Revenue Products](ancillary_revenue_products_guide.md) guide's card-program section (§10 there: the airline co-brand card program; the economics of interchange, interest, and fees). Reap's own card terms (December 2022) show a credit product with statements, credit limits, and repayment — the classic revolving-credit issuer model, plus a "Secured Reap Card Agreement" variant collateralised by customer assets. ✅/⚠ (structure verified from terms; rates not disclosed)
- **Payment fees on Reap Pay / Bill Pay.** The 2022 terms state: "Reap will charge a Fee for each Payment at the time of Payment creation... The applicable Fee will be clearly disclosed to you in advance each time you initiate a Payment." The fee schedule itself is not published. ✅/⚠
- **Platform/API fees on card issuing and payments APIs.** The 2026 homepage's "See pricing" page is gated; the card-issuing and payments products are API-priced ("One stack, direct or embedded"). ⚠ amounts not public.
- **Interest/credit income.** The Reap Card is a credit product ("delivers credit for businesses" — 2022 release) and the 2026 cards are "backed by stablecoin collateral" (a secured-credit structure). ⚠ economics not disclosed.
- **Yield on treasury.** The June 2026 USYC integration ("Reap Integrates USYC to Advance Treasury Capabilities") and the 2025 Solana treasury-management announcement indicate Reap itself runs yield-bearing stablecoin treasuries and sells treasury tooling — a stablecoin-era float income line. ⚠ amounts not disclosed.

Growth is corroborated only by company statements: "double revenue over the past year" (Oct 2022); "grew processing volumes by 8X in 2 years (to end of 2023)"; "In 2025, Reap achieved profitability and is on track to grow another 6X in volumes from 2024" (Nov 2025); "grew 3X (200%) year-on-year in both revenue and volumes" (Mar 2026); "nearly tripled revenue and volumes in 2025" (May 2026, Daren Guo quote). All are ⚠ single-sourced company claims — no audited figures are public.

### 4.4 The Market Context Reap Cites

Reap's own releases anchor its strategy in two external market datapoints, both ⚠ (industry estimates, company-curated):

- **The stablecoin/crypto card market:** "the global stablecoin and crypto card market now exceeding $18 billion annually" (Daren Guo quote, Payward acquisition release, May 7, 2026). This is the deal's own framing of the addressable market Reap's card stack serves.
- **B2B stablecoin payments growth:** a company release of June 23, 2025 — "B2B Stablecoin Payments Surge 30x to $3 Billion Monthly Volume in 2025" — cites an industry trajectory for B2B stablecoin settlement volumes. ⚠ The release title is verified; the underlying third-party methodology is not reproduced in Reap's materials, so the 30x/$3B figures are treated as company-curated industry claims.
- **The "permissionless neobank" commentary:** Reap's media page links a Pantera Capital piece (February 11, 2026, "Building Permissionless Neobanks") and a Bloomberg Opinion essay (January 6, 2026, "Five Questions About Money to Ponder in 2026") as third-party context. Their presence on the media page verifies that Reap curates this commentary; the essays' arguments are not adopted as facts here. ⚠

None of these figures is needed for any verified claim in this guide; they are recorded so a reader can see the market narrative Reap itself operates with, alongside the audit's ⚠ treatment of company-sourced numbers.

## 5. Funding, Growth, and the Payward Acquisition

### 5.1 The 2019 Seed: US$1 Million, Fresco Capital

Reap's first institutional round was a **US$1 million seed, announced at the RISE conference in Hong Kong on July 8, 2019, led by Fresco Capital**, with Hustle Fund and K3 Ventures (Singapore) and Oyster Ventures (San Francisco) participating (Reap seed release, July 8, 2019, archived at riseconf.com). The same release confirms the Cyberport Incubation Programme membership from October 2018 and notes the company had "successfully handled tens of millions of dollars in transaction volume for both domestic and international payments" by mid-2019. ✅ (Note: the research brief's "believed seed ~USD 4.5M" does not match the primary record; the verified seed figures are US$1 million in 2019 and US$5 million in 2020 — see §5.2.)

### 5.2 The 2020 Seed Extension: US$5 Million

On **July 14, 2020**, Reap announced it had "completed its seed fund raising," raising **US$5 million** from Global Founders Capital, Bertelsmann Asia Investments (BAI), Index Venture, Fresco Capital, and Hustle Fund (PRNewswire release, July 14, 2020; corroborated by Jumpstart's same-day coverage and by Bertelsmann's own announcement, "BAI Invests In Chinese Fintech Company Reap"). ✅ The Jumpstart coverage adds the period context: cumulative transaction volume "amounting to more than US$100 million since it was initialized two years ago," and a "transaction volume... CAGR of 40% per month over the past year." ⚠ (company-reported growth rate). BAI's release notes the round's framing around helping small businesses through the COVID-19 crisis. ✅ The names appear with slight variance across sources ("Index Venture" in the PRNewswire/Jumpstart text; Bertelsmann's release describes a "second round of fundraising"), which is flagged rather than smoothed: the investor set is consistent (GFC, BAI, an Index entity, Fresco, Hustle), the exact legal name of the Index entity is not.

### 5.3 The 2022 Series A: US$40 Million

On **October 27, 2022**, Reap announced a **US$40 million Series A, structured as a combination of equity and debt**, led by **Acorn Pacific Ventures, Arcadia Funds, and HashKey Capital**, with Hustle Fund, Fresco Capital, and Abacus Ventures as returning investors and Payment Asia as a co-investor (Reap press release, October 27, 2022). ✅ The release is the richest single source on the company's mid-life state:

- "Since Reap's inception in 2018..." — founding year confirmation.
- The **Reap Card launched in November 2021**, "powered by the Visa network," delivering credit for businesses.
- Customers: "over 20,000 since launch"; Web2 clients Acer, Buy and Ship, Pirata Group, Sompo Insurance, Wallem Group; Web3 clients Amber Group, Animoca Brands, Binance, Gnosis, LayerZero.
- "Reap has been able to double revenue over the past year." ⚠
- Team "of over 40 spread across Hong Kong, Australia, Canada, Japan, Malaysia, Singapore and Vietnam."

The brief's assumption that the Series A was "led by a major VC" is corrected here: the lead is a consortium of specialist Asia-focused investors (Acorn Pacific Ventures, Arcadia Funds, HashKey Capital) — reputable, but not a household-name global venture firm. ✅/⚠ as stated. Independent corroboration exists for the headline: Forbes covered the round the same week ("Hong Kong Fintech Startup Reap Raises $40 Million For Web3 Payments," October 26, 2022, linked from Reap's media page) ✅ — though the coverage repeats company figures rather than adding independent ones.

### 5.4 The Y Combinator Question

The research brief asked to verify a Y Combinator connection (batch/year). **No such connection could be found.** The YC companies directory search page could not be scraped directly (blocked), but: (1) no Reap press release, About page, funding announcement, or terms page in any era mentions Y Combinator; (2) targeted web searches pairing "Reap" with "Y Combinator," "YC," and "batch" returned no corroborating result; (3) the investors named across all rounds (Fresco Capital, Hustle Fund, K3 Ventures, Oyster Ventures, Global Founders Capital, BAI, Index Venture, Acorn Pacific Ventures, Arcadia Funds, HashKey Capital, Abacus Ventures, Payment Asia) contain no YC-affiliated fund; and (4) the 2019 About page's investor section lists only Cyberport, Fresco, Oyster, and Hustle. The claim is recorded as ❌ (not found) in §12.3, with the honest caveat that absence of evidence is not proof of absence — but the YC alumni claim rests on nothing verifiable.

### 5.5 Growth Metrics: Customers and Volume

All growth metrics are company-reported and single-sourced; none are audited. Collected for the audit (§12.2):

| Metric | Figure | Date/period | Source |
| --- | --- | --- | --- |
| Cumulative processed volume | > US$100 million | by mid-2020 | Jumpstart, Jul 2020 ⚠ |
| Monthly volume growth | 40% CAGR per month (prior year) | 2020 | Jumpstart, Jul 2020 ⚠ |
| Customers | 20,000+ | Oct 2022 | Series A release ⚠ |
| Revenue growth | "doubled" YoY | Oct 2022 | Series A release ⚠ |
| Processing volumes | 8X growth in 2 years | to end-2023 | HK HQ release, Nov 2025 ⚠ |
| Profitability | "achieved profitability" in 2025 | 2025 | HK HQ release, Nov 2025 ⚠ |
| Volume growth | "on track to grow another 6X in volumes from 2024" | 2025 | HK HQ release, Nov 2025 ⚠ |
| Revenue + volume growth | 3X (200%) YoY | 2025 | Mexico release, Mar 2026 ⚠ |
| Revenue + volume growth | "nearly tripled" | 2025 | May 2026 acquisition release (Daren Guo quote) ⚠ |
| Stablecoin-funded flows | "billions" processed | 2024 and 2025 | Company PRs (Nov 2025, May/Jul 2026) ⚠ |

The only externally checkable volume datapoint in the entire record is the 2026 deal itself: Payward's May 2026 release cites a "global stablecoin and crypto card market now exceeding $18 billion annually" (industry figure, not Reap's own volume). Reap's own payment-volume numbers remain unverified by any third party.

The funding story ends where the corporate story begins: on May 7, 2026, Payward announced its acquisition of Reap for up to US$600 million — the subject of §5.6–§5.8 — which closed on July 1, 2026. The investor set that carried Reap from seed to Series A (Fresco Capital, Hustle Fund, K3 Ventures, Oyster Ventures, Global Founders Capital, BAI, Index Venture, Acorn Pacific Ventures, Arcadia Funds, HashKey Capital, Abacus Ventures, Payment Asia) is also displayed on the current About page's "Backed by" row, which lists Acorn Pacific Ventures, Abacus, Arcadia Fund, BAI, Fresco, GFC, Index Ventures, HashKey Capital, Hustle Fund, and Payment Asia (reap.global/resources/company/about-reap, 2026). ✅ — note the About page spells the 2020-round entity "Index Ventures," the same name-variance flagged in §12.2 (item 10); the underlying legal entity is still not confirmed. ⚠

### 5.6 The Announcement: May 7, 2026

On **May 7, 2026**, Payward, Inc. announced a definitive agreement to acquire Reap Technologies Holdings Limited for **up to US$600 million, payable in a mix of cash and Payward stock**, in a transaction valuing Payward's equity at US$20 billion (Reap press release / Business Wire, May 7, 2026). The deal was first reported by Bloomberg under the headline "Kraken Parent to Buy Reap Technologies for $600 Million" (May 7, 2026, linked from Reap's own media page). ✅ Both the company release and the independent press agree on the headline figure. The "up to" formulation reflects the cash-and-stock mix and any earn-out structure; the fully-diluted economics are not public. ⚠

Key facts from the announcement release:

- The acquisition "expands Payward Services, the company's B2B infrastructure platform, unlocking globally regulated infrastructure for card issuance and stablecoin payments."
- Payward is the parent of Kraken and the family behind NinjaTrader, Breakout, xStocks, Bitnomial, and CF Benchmarks; its Co-CEO is Arjun Sethi. The release adds that the Reap deal "follows Payward's acquisitions of NinjaTrader, Bitnomial, and Backed," continuing "its strategy of expanding the platform through capability-focused transactions." ✅
- Payward's shared-architecture claims, quoted for the record: "One global liquidity pool; One unified risk and margin engine; One collateral and settlement system; One compliance and licensing framework" — and the scale datapoints "Krak shipped to 110 countries on Day 1, xStocks crossed $29 billion in cumulative volume in its first year, 1,900 B2B partners run on Payward's shared infrastructure today" (Arjun Sethi quote). ⚠ (company-reported scale figures; not independently verified)
- The strategic quote: "Card networks, banking rails, and blockchains on a single API, settling in stablecoins, licensed from Hong Kong to Mexico" (Arjun Sethi) — a one-sentence summary of Reap's verified positioning (§9, §10). ✅ as a quote.
- Advisors: PJT Partners (financial) and Jones Day (legal) for Payward; CRB Securities and Goldman Sachs (Asia) L.L.C. (financial) and Latham & Watkins (legal) for Reap. ✅
- Daren Guo's quote frames the strategic logic: "With the global stablecoin and crypto card market now exceeding $18 billion annually, Reap nearly tripled revenue and volumes in 2025, and expanded our licensing footprint from Asia to South America."
- The deal was "expected to close in the second half of 2026," subject to closing conditions and regulatory approvals. ✅ (it closed early, on July 1, 2026 — §5.7)

### 5.7 The Closing: July 1, 2026

On **July 1, 2026**, Payward announced completion of the acquisition (Reap press release / Business Wire, July 1, 2026; corroborated by American Banker, "Payward closes $600 million acquisition of Reap," July 2026). ✅ Key terms of the closing release:

- Reap "will continue to operate as a standalone brand within the Payward ecosystem, maintaining its brand, leadership team led by Daren Guo, and go-to-market approach."
- The combined licensing footprint: "Reap's existing licenses accelerate Payward's expansion across APAC and the Americas, as Payward's EU and US licenses open new corridors for Reap in Europe and the US."
- Arjun Sethi (Payward Co-CEO) frames the platform logic: "With Reap, partners can issue cards, originate cross-border payments, and manage treasury against on-chain liquidity that settles in near real time, programmatically, through one API rather than a stack of correspondent banks and regional processors."
- Reap's About boilerplate at closing: "Founded and headquartered in Hong Kong, Reap employs 300 people worldwide."
- The site's footer now reads "by Payward" and lists the Payward family (Kraken, Kraken Pro, Krak, xStocks, NinjaTrader, CF Benchmarks, Breakout, Payward Services). ✅

### 5.8 What the Deal Means

For the repository's purposes, the acquisition is significant for three reasons. **First**, it converts Reap from an independent fintech into the cards-and-payments arm of a crypto-exchange parent — a structure banks will increasingly meet on both sides of a transaction (as counterparty, as BIN sponsor, as correspondent). **Second**, the deal value (up to US$600 million, Bloomberg-reported) is the only third-party-checkable valuation event in Reap's history; pre-acquisition valuation claims are otherwise absent from the public record (§12.2). **Third**, the acquisition narrative explicitly positions stablecoins as "the settlement medium" of the combined platform — a useful, verified articulation of where the industry's corporate-payments layer is heading, and the exact theme §11's worked example builds on.

The closing release also confirms the *platform* logic the deal formalises: "With Reap, partners can issue cards, originate cross-border payments, and manage treasury against on-chain liquidity that settles in near real time, programmatically, through one API rather than a stack of correspondent banks and regional processors" (Arjun Sethi, Payward Co-CEO, July 1, 2026). That sentence — one API replacing a correspondent-bank stack — is the single most direct public statement of what Reap-style infrastructure displaces, and it is the premise §11's Cymbal Bank worked example tests from the bank's side.

---

## 6. The Products — Corporate Cards and Card Issuing

### 6.1 The Reap Card (2021–Present)

The **Reap Card** launched in November 2021 as "the industry's first Reap Visa Corporate Card" platform (Series A release, Oct 2022). ✅ Verified product facts across eras:

- **Era 2 (2021–2023):** "A HKD/USD-denominated Visa corporate card that's accepted worldwide and backed by a powerful complimentary expense management software" (Wayback of reap.global, Nov 2023). No registration or annual fee; repayment of the monthly statement "in fiat or crypto currency"; virtual card available immediately on activation; worldwide card shipment; contactless payments; real-time spend dashboard; spend controls and approval policies; Xero sync (Wayback of reap.global/reap-card, 2023). The December 2022 Terms & Conditions add the legal structure: "The Reap Card is a credit card issued by Reap in partnership with Visa in Hong Kong," with a Program Account, Card Accounts, credit limits, a Total Spending Limit, AutoPay, dispute processes, and a separate "Secured Reap Card Agreement" for collateral-backed credit. ✅
- **Era 3 (2025–present):** "USD and HKD Visa credit cards backed by stablecoin collateral. Control spend, monitor usage, and settle flexibly" (reap.global homepage, 2026). The May 2026 Visa principal-member release describes the stablecoin mechanic: "cardholders to retain a seamless payment experience, using stablecoins as credit collateral behind the scenes." ✅
- **Network and issuing status:** Visa network throughout; Reap became a **Visa Principal Issuer in Hong Kong** (status confirmed in the May 26, 2026 release: "Reap is already a Visa Principal Issuer in Hong Kong") and a **Visa Principal Member in Mexico** (granted May 2026). A principal member "can directly manage risk, settle funds with Visa, and can meet strict regulatory standards... without relying on third-party sponsors." ✅ The historical card era also involved a "partnership with Visa" per the 2022 terms; the point at which Reap upgraded from sponsor-issued to principal-issued is not dated in the public record. ⚠
- **Customers on the card:** Acer, Pirata Group, Sompo Insurance, Wallem Group (Web2); Amber Group, Animoca Brands, Binance, Gnosis, LayerZero, Trust Wallet, Coinmarketcap (Web3) — all from company releases (2022, 2025). ⚠ single-sourced.
- **The 2021 Visa survey as launch context:** before the card went live, Reap and Visa published a joint survey (December 9, 2021, "Reap-Visa Survey: HK SMEs Demand Smart Business Credit Cards") on HK SME financial-management pain — the market-research basis the company cited for the card. ✅ (release exists on Reap's newsroom) / ⚠ (survey methodology not re-examined).

### 6.2 The Card-Issuing Platform

Reap's **card-issuing platform** lets other businesses launch branded card programs on Reap's infrastructure. Verified facts:

- 2023-era description: "Kickstart card program in a flash. Empower global innovators with API, production, and support. Creating a card program that suits your business like a second skin" (Wayback of reap.global, Nov 2023). The January 2024 Chainalysis release describes it as: "Our card issuing solutions allow businesses to securely issue and curate their own credit card programs, providing seamless experiences for customers." ✅
- 2026-era description: "Launch branded card programs that let your users spend digital assets instantly" and "Launch your own digital assets-enabled card program" (reap.global, 2026); the November 2025 release: "Reap's Card Issuance platform also enables businesses to launch white-labeled credit cards through an API-first system." ✅
- The KAST testimonial on the 2026 homepage ("Running a card program is complex, but Reap makes it easy. They got us from 0 to 1 fast...") is an example of a card-program client; its claims are marketing material. ⚠
- The May 2026 release adds the Mexican scale ambition: "Based on growth projections, Reap expects to reach around a quarter of a million new card users in the market [Mexico], with its first clients to arrive in Q2 2026," and cites an El Economista report on Reap "plans to reach 250,000 card users in Mexico through partnerships with neobanks" (May 27, 2026). ⚠ projection, not actuals.
- **The US expansion datapoint (press-reported):** PYMNTS reported on September 17, 2025 that "Thredd Helps Reap Expand Card Programs to US" — naming Thredd (a card-processing platform) as the processor partner for Reap's US card-program expansion. This is the only public naming of a processor behind Reap's card stack, and it comes from third-party coverage linked on Reap's own media page, not from a Reap release. ✅ (coverage exists) / ⚠ (terms, timeline, and Reap's own confirmation not public). It does not resolve the pre-principal-issuer BIN-sponsor question for the historical HK card (§12.4).

### 6.3 Card-Program Economics (Cross-Referenced)

This guide does not re-derive card-program economics; it cross-references the repository's treatment. The [Ancillary Revenue Products](ancillary_revenue_products_guide.md) guide, §10 ("The Cymbal Bank Worked Example — An Airline Co-Brand Card Program"), lays out the issuer-side economics that Reap's model maps onto: interchange revenue on spend, interest on revolving balances, fees (late, FX, cash-advance), and the loyalty/rewards liability — plus the co-brand/program economics where the program owner shares economics with the issuing bank. Reap's structure is the non-bank variant of exactly that model: as Visa principal issuer it takes the issuer seat directly (interchange, scheme fees, settlement risk), and its "Secured Reap Card Agreement" + 2026 stablecoin-collateral model replace unsecured revolving credit with collateralised credit — which changes the loss-given-default profile but not the revenue stack. For the worked example in §11, the ancillary-revenue guide's card-program framing is assumed as read.

---

## 7. The Products — Payments, Expense Management, and Integrations

### 7.1 Reap Pay and Bill Pay — Cross-Border Payments

**Reap Pay** (the historical product) and **Bill Pay** (the 2026 product) are the money-movement spine of the company:

- **Era 1/2 — Reap Pay:** "Payment solution that enables you to settle bills directly from your corporate card to the recipients' bank account — in their preferred currency" (Wayback of reap.global/reap-pay, 2023). The December 2022 terms formalise it: Reap Pay makes "Payments to third-party Recipients, from a User's Payment Method bearing any Accepted Card Brands (debit or credit card) to a bank account legally held by the Recipient"; Reap Collect is the mirror (request payments from customers' cards to your bank account). Cross-border variants ran through the licensed remittance subsidiary: "All payments to non-Hong Kong recipients are conducted via our wholly-owned subsidiary Reap (Remit) Limited... holder of Money Service Operator License number 19-08-02816." ✅
- **Era 3 — Bill Pay and Payments API:** "Use stablecoins to fund local or cross-border business payments" and "Automate fiat payouts at scale — settled directly from your on-chain treasury" (reap.global, 2026). The May 2026 TerraPay partnership "to Expand Local Payout Corridors Globally" and the March 2026 Mexico release's mention of local partners Rio and Monato (Mexico's local transfer and real-time payment networks) extend the payout-corridor story. ✅
- **The rails mechanics** (cross-referenced, not re-derived): the [Payment Rails](payment_rails_guide.md) guide's §5 (clearing and settlement) and §8 (a rail-selection design) explain the mechanics that Reap's architecture wraps — card scheme clearing for the card leg, local ACH/real-time rails for the payout leg, and stablecoin settlement rails for the treasury leg. Reap's public description matches a multi-rail orchestration pattern: "Its platform integrates card networks, traditional finance rails, and stablecoin-native settlement into a single API-driven infrastructure" (May 2026 acquisition release). ✅/⚠ (mechanism described by the company; the specific rails and partners per corridor are not all public)

### 7.2 Expense Management

Expense management has been bundled with the card product since 2021 and is now a standalone product line (reap.global/products/expense-management). Verified features, assembled from the 2023 card page and the November 2025 release:

- Real-time spend tracking and dashboards ("Gauge all spend in real-time. Get notify for all your company spendings and oversight finances with intuitive dashboard" — Wayback 2023).
- Spend controls: "Manage your team's budget with precise spend controls and approval policies"; multi-user permissions; "rule-based approval policies according to different spending thresholds"; per-employee cards with permission rights (Wayback 2023).
- Receipts and reimbursement automation: "digitalise and automate traditional reimbursement processes. Say goodbye to manual reconciliation, invoice matching, and receipt chasing" (Wayback 2023).
- 2025-era AI roadmap: "AI-powered expense management... The system will automate scaled card issuance, match receipts to transactions in real time, flag anomalies with timely alerts, and streamline bookkeeping (categorization, invoice processing, accounting sync, approvals), supported by human oversight" (HK HQ release, Nov 2025). ✅/⚠ (roadmap language, not a shipped-feature changelog)

### 7.3 Accounting and ERP Integrations (Cross-Referenced)

The one accounting integration named across all eras is **Xero**: "Sync Reap with Xero or other accounting systems, we will blend into your workflow" (Wayback 2023); "Connect and synchronise with accounting software like Xero to streamline your accounting process seamlessly" (Wayback 2023). The November 2025 release references "accounting sync" generically in the AI-expense roadmap, and the 2022 Series A release notes Web3 customers could "plug immediately into their internal systems to be up and running on day 1 of implementation" — indicating an API-driven sync layer. ✅ The full current integration catalogue (which ERPs beyond Xero: QuickBooks, NetSuite, SAP, etc.) is **not published** ⚠. The integration architecture themes are cross-referenced to the [Financial Management Systems](../technology/financial_management_systems_guide.md) guide (§1, the financials stack — GL, sub-ledgers, close; §5, the ERP vendor landscape) and the [Enterprise Middleware & Integration Platforms](../technology/enterprise_middleware_integration_platform_guide.md) guide (§6, the banking angle; §7, integration-estate rationalization): a card/payments platform syncing into a client's GL is an integration-fabric problem — connectors, mapping, idempotent sync, and audit trails — not a product problem.

---

## 8. The API Platform and Embedded Finance

### 8.1 The API-First Stack

Reap's developer surface is a public, documented API estate:

- **Documentation:** docs.reap.global (card-issuing API guide, API reference, changelog, quick-start guide, sandbox). The site's developer menu links to the same surfaces via reap.readme.io (ReadMe-hosted docs). ✅
- **Products exposed via API:** Card Issuing ("Launch branded card programs that let your users spend digital assets instantly"; "one API integration" for embedded finance) and Payments ("Direct API mentions coming soon" — i.e., the payments API is currently contact-gated: "Payments — Talk to us. Contact us to access the documents"). ✅
- **Sandbox:** "Simulate key card workflows" / "Test your card issuing flow" (docs.reap.global/quickstart, /transactions/testing-in-sandbox). ✅
- **Agentic Payments:** a 2026 product line — "Agentic spends on any merchant, anywhere. Agent-native checkout live today" — with an "Agentic Accelerator Program" waitlist. The November 2025 release ties this to Model Context Protocol (MCP): "Reap is also leaning heavily into AI-powered expense management across its platforms and agentic flows through Model Context Protocol (MCP)." ✅/⚠ (product claims; the MCP integration detail is company-described)
- **The 2026 platform positioning:** "one API rather than a stack of correspondent banks and regional processors" (Payward closing release, July 2026); "card networks, banking rails, and blockchains on a single API, settling in stablecoins" (May 2026 release). ✅

### 8.2 Embedded Finance

The 2026 homepage's embedded-finance product line is explicit: "Embedded Finance — Launch branded financial products on Reap's infrastructure, one API integration," with two embeddable modules: **Card Issuing** ("Launch your own digital assets-enabled card program") and **Payments** ("Automated global fiat payouts at scale using stablecoins"). The KAST testimonial (a card-program client "from 0 to 1 fast") and the use-case tiles (exchanges/wallets, neobanks/fintechs, import-export/PSPs, accounting/marketing) sketch the target segments. ✅ (product existence) / ⚠ (client counts, volumes, and the specific economics of the embedded program are not public)

### 8.3 Integration-Platform Themes (Cross-Referenced)

The [Enterprise Middleware & Integration Platforms](../technology/enterprise_middleware_integration_platform_guide.md) guide's taxonomy (API gateways, iPaaS, event-driven integration, B2B gateways — §3–§4 there) is the correct lens for Reap's stack: a card-issuing API plus a payments API plus accounting sync is an integration fabric aimed at *other businesses'* products, i.e., the "integration platform as a product" pattern (the guide's §7 worked example shows the same pattern from a bank's internal perspective). This guide does not re-derive those themes; §8.1's verified API facts are the Reap-specific payload on top of the middleware guide's framework.

### 8.4 The 2026 Product-Surface Map

The 2026 site navigation (cached from reap.global during this pass) exposes the full product surface — a useful cross-check that the sections above cover the whole catalogue, including the **Business Account** and **Rewards** lines that are easy to miss:

| Product line (site nav) | One-line description (site text) | Covered in this guide |
| --- | --- | --- |
| Business Account | "The unified account layer for stablecoin-enabled businesses" | §7 (payments/treasury themes) |
| Cards | "USD and HKD Visa credit cards backed by stablecoin collateral" | §6.1 |
| Bill Pay | "Use stablecoins to fund local or cross-border business payments" | §7.1 |
| Expense Management | AI-powered expense management with approvals and reporting | §7.2 |
| Card Issuing (embedded) | "Launch branded card programs that let your users spend digital assets instantly" | §6.2, §8.2 |
| Payments (embedded) | "Automate fiat payouts at scale — settled directly from your on-chain treasury" | §7.1, §8.2 |
| Agentic Payments | "Agentic spends on any merchant, anywhere. Agent-native checkout live today" | §8.1 |
| Sentry | "Fraud & risk management" | §9.4 |
| Rewards | Loyalty/rewards product tile (reap.global/products/reap-rewards) | ⚠ existence only — no details published |

The nav also confirms the developer surfaces (docs.reap.global + reap.readme.io mirrors) and the pricing page's registration gate. Two lines are notable for what they say about the 2026 model: the Business Account ("unified account layer") is the account-issuance claim that sits *alongside* — not instead of — the licensed remittance/card structure of §9, and the Rewards tile is the loyalty/rewards liability that the card economics of §6.3 anticipate. ⚠ for both: the Business Account's regulatory classification and Rewards' economics are not public.

## 9. Licensing and Compliance

### 9.1 Hong Kong: MSO Licence and Visa Principal Issuer

Reap's Hong Kong licensing posture verifies from the company's own legal documents and press releases:

- **Money Service Operator (MSO) licence.** Reap's December 2022 Terms & Conditions state: "All payments to non-Hong Kong recipients are conducted via our wholly-owned subsidiary **Reap (Remit) Limited**, a company incorporated in Hong Kong under Company Registry number 2749310 and **holder of Money Service Operator License number 19-08-02816**" (Reap Terms & Conditions, December 2022 capture, Wayback Machine). ✅ This is a primary-source statement of the HK MSO licence held by the remittance subsidiary — the licence under the Hong Kong Anti-Money Laundering and Counter-Terrorist Financing Ordinance, administered by the Customs and Excise Department, required for money-service operators (remittance and money-changing). The current Customs register itself could not be queried live for this pass (registry URL changed; ⚠), so the licence number is verified via the company's own terms rather than the regulator's register.
- **Visa Principal Issuer in Hong Kong.** Confirmed in Reap's own May 26, 2026 release: "Reap is already a Visa Principal Issuer in Hong Kong and now becomes one of a few stablecoin payments companies globally to have a Visa Principal Issuer License in both Asia and the Americas." ✅ (company statement; Visa's membership directory is not public)
- **Stored Value Facility (SVF) licence.** The research brief asked whether an SVF licence (HKMA) is held. **No evidence of an SVF licence was found**, and Reap's own 2022 terms disclaim being "a bank, remittance agent, payment system, stored value facility or payment gateways as defined by the applicable laws in the jurisdictions where we are duly incorporated and operating" — an explicit statement that it was *not* operating as a licensed SVF as of December 2022. ⚠/✅ as dated. (The SVF licence matters for stored-value/prepaid models; Reap's credit-card model does not require one, and the terms confirm the company's own position.)

### 9.2 Mexico: Money Transmitter Registry and Visa Principal Member

Reap's 2026 Americas expansion is licence-led, all verified from company releases:

- **Money Transmitter Registry (March 23, 2026):** "having obtained a Money Transmitter Registry in Mexico, formally enabling the company to operate localized money transmission services in the country." The release adds two further registrations: "Reap also holds a **Vulnerable Activities Registration for Credit Cards Issuing** and a **Vulnerable Activities Registration for Virtual Assets** in Mexico" — the Mexican AML ("vulnerable activities") regime registrations. ✅
- **Visa Principal Member in Mexico (May 26, 2026):** "granted Visa Principal Issuer Membership in Mexico... dual Principal License holder in Mexico and Hong Kong." ✅
- The Mexico release also names local partners (Rio; Monato for local transfer/real-time payment networks) and positions Mexico as "Reap's cross-border payments hub for the Americas." ✅

### 9.3 Singapore: The MAS Question

The research brief asked to verify Reap's MAS licensing status (Major Payment Institution vs other) against the registry. The verified record is thin and leans negative:

- **No MAS licence found.** The MAS Financial Institutions Directory could not be queried by name server-side for this pass (the directory's search is JavaScript-driven; a direct query returned the full directory rather than filtered results — ⚠ methodology limitation). However, the MAS **list of entities that notified under the Payment Services (Exemption for Specified Period) Regulations 2019** — the register of pre-PS-Act operators that ran under temporary exemption while applying — was extracted live and does **not** include any Reap entity (only eight entities are listed; none is Reap-related). ✅/❌ That means Reap Singapore never operated under the temporary exemption regime, which in turn means either it did not provide regulated payment services in Singapore within the PS Act's scope during the exemption window, or it did so without notifying — the latter being a breach. The first reading is more plausible: Reap's Singapore office was a sales/operations outpost for a Hong Kong-licensed business.
- **The company's own posture.** The 2026 website footer carries the disclaimer: "This website and the services described herein are **not directed at persons in Singapore**. Nothing contained on this website constitutes an offer, solicitation or invitation to provide regulated products & services in Singapore." ✅ This is a deliberate carve-out: Reap does not market regulated services to Singapore persons. Meanwhile the homepage security section claims "Licensed in Mexico, Hong Kong, Singapore, and more" — a company claim that this guide could not corroborate with any MAS record ⚠, and which sits awkwardly with the footer disclaimer.
- **The MAS-regime comparison** (cross-referenced, not re-derived): the [MAS Regulations](mas_regulations_guidelines_guide.md) guide explains the PS Act licensing tiers — Standard Payment Institution (SPI) vs Major Payment Institution (MPI) — thresholds, and the AML/CFT obligations. For a Hong Kong MSO-licensed remittance-and-card business, the Singapore analogue would be an MPI licence for account-issuance/money-transfer services; no such licence appears on the record, and the company's own disclaimers indicate it does not currently hold itself out as providing regulated services in Singapore. The honest status line: **no MAS licence, no MAS exemption notification, company disclaims Singapore-directed services — the "licensed in Singapore" homepage claim is unverified ⚠.**

### 9.4 AML/KYC Posture (Cross-Referenced, Condensed)

Reap's AML/KYC posture is assembled from its own documents and partnerships; the screening *themes* are cross-referenced to the [Fircosoft](fircosoft_guide.md) guide (§3–§5 there: sanctions-list screening, name-matching, fuzzy logic, false-positive management) rather than re-derived:

- **KYC onboarding.** The December 2022 terms require, at minimum: full name/business name, email, phone, identification document number, company registry number, business nature and address, authorized representative details, and recipient bank details, with an express KYC statement: "To help the government fight financial crime, Hong Kong law requires us to obtain, verify, and record information that identifies each natural person and business entity that opens a Program Account or uses a Reap Card" — including beneficial-owner identification (the terms define a Beneficial Owner at the >25% threshold). ✅ (matches the Fircosoft guide's screening-and-identity themes)
- **Sanctions screening.** The terms oblige the company and its users to ensure no authorized user is on "an OFAC list, or any law, regulation, or other list of any government agency that prohibits or limits us from providing Accounts or Cards," and Reap may decline transactions involving sanctioned persons (December 2022 terms). ✅
- **On-chain transaction monitoring.** Since January 2024, Reap has used **Chainalysis risk solutions** for transaction monitoring and illicit-activity detection, integrated with **Fireblocks** as custodian wallet: "all transactions passing through Fireblocks are automatically screened by Chainalysis, promptly flagging any suspicious fund flows" (Reap-Chainalysis release, January 30, 2024). ✅
- **Fraud and card risk.** The 2026 product line includes **Sentry** ("fraud & risk management" product tile), the homepage claims PCI DSS compliance ("Protecting your data and enabling you to build PCI DSS compliant solutions"), and real-time risk detection "with tools like Chainalysis." ✅/⚠ (product claims; certification documents not published)
- **The condensed takeaway** (cross-ref [Fircosoft](fircosoft_guide.md) §9's "clean list" framing): Reap's control stack is a standard layered screening-and-monitoring design — identity/KYC at onboarding, sanctions screening at account and transaction level, on-chain monitoring on the digital-asset leg, and chargeback/dispute handling on the card leg. Nothing in the public record suggests weaknesses; equally, nothing beyond these vendor names and terms clauses is public. ⚠ depth-of-evidence limitation.

### 9.5 The Three-Jurisdiction Licensing Comparison

The licence table below is the regulatory core of the guide, condensed from §9.1–§9.3. It is the reference the Cymbal Bank worked example (§11.5) reasons from:

| Jurisdiction | Instrument held | Holder / status | Evidence |
| --- | --- | --- | --- |
| Hong Kong | Money Service Operator licence no. 19-08-02816 (remittance) | Reap (Remit) Limited (HK CR 2749310) | Reap Terms & Conditions, Dec 2022 ✅ (registry not re-queried ⚠) |
| Hong Kong | Visa Principal Issuer | Reap Technologies Limited | Company release, May 26, 2026 ✅ |
| Hong Kong | SVF licence (HKMA) | Not held / disclaimed | Company terms, Dec 2022 ✅ (as dated) |
| Mexico | Money Transmitter Registry | Reap (Mexico entity per company release) | Company release, Mar 23, 2026 ✅ |
| Mexico | Vulnerable Activities Registrations (cards issuing; virtual assets) | Same | Company release, Mar 23, 2026 ✅ |
| Mexico | Visa Principal Member | Same | Company release, May 26, 2026 ✅ |
| Singapore | MAS licence (SPI/MPI) | None found; services disclaimed | MAS exemption register (live) + site footer ✅/❌ |
| EU / US | Licences of Payward family | Available to Reap post-acquisition (payward.com) | Payward closing release, Jul 1, 2026 ✅/⚠ |

Reading the table vertically tells the strategy: Hong Kong and Mexico carry the regulated instruments; Singapore is explicitly disclaimed; and the EU/US corridor is supplied post-acquisition by the Payward family's own licences rather than by new Reap filings. ⚠ on the final row: Payward's licence inventory is described in its own releases, not independently enumerated here.

---

## 10. Technology — Public Facts

### 10.1 The Platform Architecture Themes

Reap's public technology narrative is consistent across the 2025–2026 releases and maps onto the repository's integration-platform framework ([Enterprise Middleware & Integration Platforms](../technology/enterprise_middleware_integration_platform_guide.md) — cross-referenced, not re-derived):

- **Multi-rail orchestration:** "Its platform integrates card networks, traditional finance rails, and stablecoin-native settlement into a single API-driven infrastructure, supporting corporate card issuing, card programs, cross-border payouts, and treasury management" (May 2026 acquisition release). ✅
- **Stablecoin settlement layer:** "Stablecoins are the settlement substrate" (Arjun Sethi quote, May 2026); Reap's treasury management runs "primarily on Solana" (company release, March 24, 2025: "Reap Announces Scalable Stablecoin Treasury Management Primarily on Solana"). ✅
- **Custody and screening:** Fireblocks as custodian wallet; Chainalysis screening integrated at the custody layer (Jan 2024 release). ✅
- **AI/agentic layer:** AI "at its core" internally; agentic flows via Model Context Protocol (MCP); an "Agentic Payments" product with agent-native checkout (Nov 2025 and 2026 releases). ✅/⚠
- **Programmability narrative:** "Continuous markets. Programmable money. Autonomous execution" (Payward release, May 2026) — the vendor's framing, recorded as such. ⚠

### 10.2 The Verified Technology Facts

| Fact | Detail | Status |
| --- | --- | --- |
| Public API docs | docs.reap.global — card-issuing API guide, API reference, changelog, quick start, sandbox | ✅ |
| API product surface | Card Issuing (self-serve docs) and Payments (contact-gated) | ✅ |
| Card network | Visa (HK principal issuer; Mexico principal member) | ✅ |
| Stablecoin rails | USDC/USDT repayments (2023); Solana treasury management (2025); USDC funding on Hyperliquid (Aug 2026) | ✅ |
| Custody | Fireblocks (custodian wallet) | ✅ |
| On-chain screening | Chainalysis risk solutions | ✅ |
| Compliance partners | Triple-A (MAS-licensed crypto payment gateway, 2023 partnership), Chainalysis, Global Dollar Network (Dec 2025) | ✅ |
| Payout corridors | TerraPay partnership (May 2026); Fincra MoU for Africa (Sep 2025); Rio/Monato in Mexico | ✅ |
| Treasury yield | USYC integration (Jun 2026) | ✅ |
| Expense/ERP sync | Xero (named); "other accounting systems" (unnamed) | ✅/⚠ |
| Agentic payments | "Agent-native checkout live today" + MCP | ✅/⚠ |

### 10.3 What Is Not Disclosed

The following technology facts are **not public** and are flagged ⚠ rather than guessed: the card-issuing processor/BIN-sponsor stack beneath the Visa principal membership (the US-expansion processor Thredd is press-named in 2025 — §6.2 — but the HK principal-issuer stack is not); the specific cloud infrastructure and hosting; the core-banking/ledger engine (a proprietary ledger is implied by the "programmable money" narrative but never named); the full ERP/accounting connector catalogue; the payments API's underlying rail partners per corridor; the ML models behind Sentry; and the geographic data-residency arrangements. Where the [Enterprise Middleware](enterprise_middleware_integration_platform_guide.md) or [Financial Management Systems](financial_management_systems_guide.md) guides supply the architectural vocabulary, this guide uses it; where Reap is silent, this guide says so.

### 10.4 The Partner-Ecosystem Chronology

Every dated partner milestone below is verified from Reap's own newsroom listing (reap.global/resources/company/media, cached during this pass) — the dates and release titles are ✅; the substantive claims inside each release carry their own flags as cited elsewhere in this guide:

| Date | Milestone (newsroom title) | Where covered |
| --- | --- | --- |
| Oct 20, 2021 | Visa card partnership announced (HK press, hk01) | §6.1 |
| Dec 9, 2021 | Reap–Visa survey on HK SME card demand | §6.1 |
| Jan 11, 2023 | Fireblocks: stablecoin repayments on the Reap Card | §4.2, §10.1 |
| May 11, 2023 | Triple-A partnership: pay fiat bills with digital currencies | §10.2 table |
| Jan 30, 2024 | Chainalysis: compliance and risk management | §9.4 |
| Mar 4, 2025 | Title sponsorship, CUPRA FIP padel tournaments (Asia) ⚠ marketing | §10.4 (this row) |
| Mar 24, 2025 | Solana-based stablecoin treasury management | §10.1 |
| Jun 23, 2025 | "B2B Stablecoin Payments Surge 30x to $3 Billion Monthly Volume in 2025" | §4.4 |
| Sep 11, 2025 | Fincra MoU: cards + cross-border payments for Africa | §7.1 |
| Sep 17, 2025 | Thredd (press): US card-program expansion | §6.2 |
| Nov 5, 2025 | HK global-HQ expansion; Reap Direct platform detailed | §3, §4.2, §7.2 |
| Dec 3, 2025 | Joins Global Dollar Network (stablecoin payment infrastructure) | §10.2 table |
| Mar 23, 2026 | Mexico Money Transmitter Registry + Vulnerable Activities registrations | §9.2 |
| May 7, 2026 | Payward acquisition announced (up to US$600M) | §5.6 |
| May 14, 2026 | TerraPay partnership: local payout corridors | §7.1 |
| May 26, 2026 | Visa Principal Member in Mexico | §9.2 |
| Jun 23, 2026 | USYC integration for treasury yield | §4.3 |
| Jul 1, 2026 | Payward acquisition completed | §5.7 |
| Aug 11, 2026 | Hyperliquid: USDC funding capability live | §10.2 table |

The sequence shows the arc this guide narrates: Visa-anchored cards (2021) → stablecoin repayments (2023) → compliance hardening (2024) → treasury/scale plays (2025) → Americas licensing and acquisition (2026).

---

## 11. The Cymbal Bank Worked Example — A Corporate-Spend Client on a Reap-Style Program

*Fictional-but-substantiated: the client, the numbers, and the flows below are constructed for illustration from the verified facts in §4–§10 and the repository's persona conventions (see [Citadel LLC](citadel_llc_guide.md) §10 and [Resona Merchant Bank Asia](resona_merchant_bank_asia_guide.md) §7). Nothing in this section describes an actual Reap client.*

### 11.1 The Scenario

**Meridian Robotics Pte. Ltd.** is a Singapore-incorporated, mid-market robotics exporter: 140 employees, offices in Singapore, Hong Kong, and Vietnam, monthly operating spend of about US$1.2 million across cloud services, supplier invoices, travel, and per-diem, and a treasury that increasingly holds USDC because its largest customer pays in stablecoin. Meridian wants corporate cards for its 60 spenders, cross-border bill payments in six currencies, and a clean feed into its Xero books — without taking a banking relationship it doesn't already have. Cymbal Bank, Meridian's primary bank, does not itself offer a stablecoin-collateralised card program, so the relationship team evaluates a **Reap-style program** (the verified Reap model of §6–§8) as the recommended structure, and designs the bank's involvement around it: Cymbal remains the fiat settlement bank for the program's HKD/USD funding legs and provides the reconciliation overlay.

### 11.2 The Card Program

The program follows the verified Reap card structure (§6.1): a Visa corporate credit card, USD-denominated for Meridian, issued under a principal-issuer arrangement, with the credit line **collateralised by Meridian's USDC holdings** (the 2026 stablecoin-collateral model) rather than by unsecured revolving credit. Meridian gets 60 cards — physical for travellers, virtual for the rest — each with per-card limits, MCC controls (no gambling, no cash advances), and rule-based approval policies above US$5,000 (the expense-management features of §7.2). The economics are the issuer economics of the [Ancillary Revenue Products](ancillary_revenue_products_guide.md) guide §10: interchange on spend, interest/fees on balances, FX fees on non-USD transactions — here with the twist that the credit risk is secured by stablecoin collateral, so the loss-given-default profile resembles secured lending rather than unsecured card lending. Cymbal Bank's role: it does not underwrite the collateralised line (the program operator does), but it holds the fiat settlement accounts into which card repayments and disbursements net, and it reviews the program operator's issuer risk framework as part of its third-party-risk process.

### 11.3 Settlement and Treasury

The settlement flow is the multi-rail pattern of §7.1/§10.1, read against the [Payment Rails](payment_rails_guide.md) guide §5 (clearing and settlement):

1. **Card leg:** Meridian's employees spend on Visa; the scheme clears and settles the merchant transactions to the program operator's settlement account (in HKD/USD) on the scheme's settlement cycle.
2. **Funding leg:** Meridian's treasury posts USDC as collateral to the custody wallet (Fireblocks-style custodian, §10.2); the program operator screens the incoming wallet flows (Chainalysis-style, §9.4) before accepting them as collateral.
3. **Repayment leg:** Monthly statements are repaid "in fiat or crypto" (§6.1) — Meridian repays in USDC from its treasury; the operator converts via its liquidity rails and nets into the fiat settlement accounts Cymbal holds.
4. **Payout leg:** Meridian's cross-border bills (suppliers in Vietnam, Japan, Germany) are paid via the Bill Pay-style product: stablecoin-funded fiat payouts into local rails, with FX applied at the payout leg (the TerraPay-style corridor partners of §7.1).

The treasury outcome is the verified "settle flexibly" promise: Meridian earns yield on USDC while it sits as collateral (the USYC-style treasury integration of §10.2), and its working-capital cycle is no longer tied to a bank's credit decision — the collateralised line is instant.

### 11.4 Reconciliation and Accounting Sync

Meridian's finance team reconciles the program through three control points, each mapped to verified Reap features and the repository's FMS themes ([Financial Management Systems](../technology/financial_management_systems_guide.md) §1 — GL, sub-ledgers, close):

1. **Transaction-level feed:** every card transaction and every payout flows through the API into Meridian's systems with a stable identifier, so the card sub-ledger is always current — no month-end receipt chasing (the "say goodbye to manual reconciliation" promise of §7.2).
2. **Xero sync:** approved transactions post to Xero with the correct GL mapping (the named integration of §7.3), categorised by the AI layer's rules with human approval on exceptions (the Nov 2025 roadmap, §7.2).
3. **Collateral reconciliation:** Cymbal Bank's overlay — the value-add that keeps the bank in the flow — is a daily reconciliation between (a) the collateralised credit utilisation, (b) the stablecoin collateral value (marked daily, with margin calls if the stablecoin peg or collateral ratio moves), and (c) the fiat settlement balances at Cymbal. This is the classic bank reconciliation duty applied to a stablecoin-collateralised program, and it is where Cymbal's control framework earns its fee.

### 11.5 KYC/AML and the Regulatory Overlay

The KYC/AML overlay follows the verified posture of §9.4, condensed to the themes of the [Fircosoft](fircosoft_guide.md) guide (§3–§5): Meridian is onboarded with corporate KYC (beneficial owners at the >25% threshold, the December 2022 terms' standard); all authorised users are screened against sanctions lists (OFAC-style) at issuance and continuously; on-chain flows are screened at the custody layer before collateral is accepted; and the card transactions run standard fraud/chargeback controls. The regulatory overlay is where the jurisdictions differ, and the contrast is the pedagogical point of this worked example:

- **Hong Kong leg:** the program operator holds the MSO licence for the remittance legs and the Visa principal issuer status for the card legs (§9.1) — the structure is legal and licensed.
- **Mexico leg (if Meridian expands there):** the money-transmitter registry and Visa principal membership cover the Americas hub (§9.2).
- **Singapore leg:** Meridian is Singapore-incorporated, but the program is *not* sold to it as a Singapore-regulated service — consistent with Reap's own disclaimer that its services "are not directed at persons in Singapore" (§9.3). Cymbal Bank's compliance team therefore documents the arrangement carefully: Meridian accesses the program through the Hong Kong entity; Cymbal's Singapore branch provides only the fiat settlement accounts and reconciliation services; and the question of whether any part of the activity constitutes regulated payment services in Singapore is answered by reference to the [MAS Regulations](mas_regulations_guidelines_guide.md) guide's scope analysis. The honest, audit-ready conclusion for Cymbal: the *bank's* services are MAS-scoped and compliant; the *program's* services are Hong Kong/Mexico-scoped; and the gap between the two is documented rather than assumed away.

The worked example closes where every corporate-spend program closes: with the ledger. The card program, the settlement flows, the collateral, and the compliance overlay all resolve into one question — does the client's spend ledger tell the truth, in real time, at month-end? On a Reap-style program with Cymbal's reconciliation overlay, it does — and that, ultimately, is the product.

---

## 12. The Claims Audit — Verified, Flagged, Rejected

### 12.1 The Verified Claims (✅)

| # | Claim | Verification |
| --- | --- | --- |
| 1 | Founded 2018, Hong Kong | Company PRs (2022, 2026), About page, 2019 seed release ✅ |
| 2 | Co-founders Daren Guo and Kevin Kang | 2019 About page (Wayback), 2019/2020 seed releases, 2022/2024/2026 PRs ✅ |
| 3 | Reap Technologies Limited = HK CR 2714427 | Company Terms & Conditions (Dec 2022, Wayback) ✅ |
| 4 | 2019 seed US$1M led by Fresco Capital (Hustle Fund, K3 Ventures, Oyster Ventures) | 2019 seed release (riseconf.com PDF) ✅ |
| 5 | 2020 seed US$5M (GFC, BAI, Index Venture, Fresco, Hustle Fund) | PRNewswire Jul 14 2020; Jumpstart; Bertelsmann announcement ✅ |
| 6 | Series A US$40M (equity+debt), led by Acorn Pacific Ventures, Arcadia Funds, HashKey Capital; Payment Asia co-investor | Reap PR Oct 27 2022 ✅ |
| 7 | Reap Card launched Nov 2021, Visa corporate credit card | Series A release; card terms Dec 2022 ✅ |
| 8 | "The Reap Card is a credit card issued by Reap in partnership with Visa in Hong Kong" | Company terms (Dec 2022) ✅ |
| 9 | Reap (Remit) Limited holds HK MSO licence 19-08-02816 (CR 2749310) | Company terms (Dec 2022) ✅ |
| 10 | Visa Principal Issuer in Hong Kong; Visa Principal Member in Mexico (May 2026) | Company PR May 26 2026 ✅ |
| 11 | Mexico Money Transmitter Registry + Vulnerable Activities Registrations (Mar 2026) | Company PR Mar 23 2026 ✅ |
| 12 | Payward to acquire Reap for up to US$600M (cash + stock), announced May 7 2026 | Company PR; Bloomberg headline; Business Wire ✅ |
| 13 | Acquisition completed July 1 2026; Reap standalone brand; Daren Guo leads | Company PR Jul 1 2026; American Banker ✅ |
| 14 | Hong Kong global HQ expansion Nov 2025 (1063 King's Road, Quarry Bay) | Company PR Nov 5 2025 ✅ |
| 15 | Global HQ is Hong Kong ("Founded and headquartered in Hong Kong") | PRs 2025–2026 ✅ |
| 16 | Partners: Visa, Fireblocks, Chainalysis, Circle, Solana, TerraPay, Global Dollar Network, Triple-A | Company PRs 2023–2026 ✅ |
| 17 | Xero accounting sync (named integration) | Wayback of reap.global 2023 ✅ |
| 18 | Public API docs + sandbox for card issuing | docs.reap.global (live) ✅ |
| 19 | Not on MAS PS Act exemption-notification list | MAS register (extracted live) ✅/❌ |
| 20 | Website disclaims Singapore-directed services | reap.global footer (live) ✅ |
| 21 | Visa partnership press coverage at card launch (hk01, Oct 20 2021) | Reap media page link ✅ |
| 22 | Reap–Visa joint survey on HK SME card demand (Dec 9 2021) | Reap newsroom ✅ |
| 23 | Forbes coverage of the Series A (Oct 26 2022) | Reap media page link ✅ |
| 24 | Newsroom chronology dates (Fireblocks, Triple-A, Chainalysis, Solana, Fincra, GDN, Mexico, TerraPay, Visa MX, USYC, Hyperliquid, Payward) | reap.global/media listing, cached ✅ |

### 12.2 The Flagged Claims (⚠)

| # | Claim | Why flagged |
| --- | --- | --- |
| 1 | 20,000+ customers (Oct 2022) | Company PR only; no third-party confirmation |
| 2 | "Doubled revenue" (2022); 8X volumes to 2023; 6X growth 2024→2025; 3X/200% growth 2025; profitability 2025 | All company PRs; no audited figures |
| 3 | "Billions" in stablecoin-funded flows (2024, 2025) | Company PRs; undefined metric |
| 4 | "Licensed in ... Singapore" (homepage security claim) | No MAS record found; contradicts footer disclaimer |
| 5 | 250,000 card users projected in Mexico | Company projection + El Economista report; not actuals |
| 6 | KAST and other client testimonials | Marketing material |
| 7 | PCI DSS compliance claim | Company claim; no certification published |
| 8 | Up-to-US$600M deal economics ("up to" structure, stock mix) | Headline verified; fully-diluted economics not public |
| 9 | Reap's valuation prior to acquisition | Not public anywhere |
| 10 | Index Venture's exact legal name in the 2020 round | Source variance (PRNewswire/Jumpstart vs BAI) |
| 11 | Full ERP/accounting integration catalogue | Only Xero named publicly |
| 12 | Currency list supported | HKD/USD cards named; payout-currency list not published |
| 13 | Card-issuing processor/BIN-sponsor stack beneath Visa principal status | Not disclosed |
| 14 | MSO licence registry status (current) | Verified via company terms (2022); Customs register not queryable this pass |
| 15 | SVF licence | No evidence found; company disclaimed SVF status in 2022 terms |
| 16 | Forbes Asia "100 to Watch" (2023) and Tatler Gen.T honouree | Company About page claims; not independently re-verified |
| 17 | Payward scale figures (1,900 B2B partners; Krak in 110 countries; xStocks US$29B) | Arjun Sethi quotes in Payward release; company-reported |
| 18 | Thredd as US card-program processor (Sep 2025) | PYMNTS coverage linked on Reap media page; not a Reap release |
| 19 | "B2B Stablecoin Payments Surge 30x to $3 Billion Monthly" (Jun 2025) | Company-curated industry claim; methodology not reproduced |

### 12.3 The Rejected or Not-Found Claims (❌)

| # | Claim | Finding |
| --- | --- | --- |
| 1 | Y Combinator affiliation (any batch/year) | No evidence in any primary source or targeted search; investors list contains no YC fund |
| 2 | "Daniel Kuo" and "Adil Mohammed" as co-founders | Not found in any primary source (2019 About page, all PRs, current site) |
| 3 | Singapore as headquarters | Refuted — founded and headquartered in Hong Kong; Singapore was an office |
| 4 | Seed of ~US$4.5M | Refuted — verified seeds are US$1M (2019) + US$5M (2020) |
| 5 | Series A "led by a major VC" | Corrected — led by Acorn Pacific Ventures, Arcadia Funds, HashKey Capital (specialist Asia VCs) |
| 6 | MAS licence or exemption for a Reap entity | Not found on MAS exemption register; no FID record retrievable; company disclaims SG services |

### 12.4 What Could Not Be Verified

- **The MAS Financial Institutions Directory entry for any Reap entity.** The directory's search is JavaScript-driven and could not be filtered server-side during this pass; the definitive "no MAS licence" statement therefore rests on the exemption-register absence plus the company's own disclaimers, not on a direct directory lookup. A reader with interactive access to eservices.mas.gov.sg/fid should confirm.
- **The Hong Kong Customs MSO register entry.** The licence number is verified from Reap's own terms (Dec 2022); the regulator's live register URL changed and could not be queried. Current licence status should be re-confirmed against the Customs register.
- **The Y Combinator alumni directory.** The YC companies page blocks scraping; the negative finding rests on the absence of any mention in Reap's materials and search results, which is strong but not a directory query.
- **Reap's actual payment volumes, revenue, and profitability.** Every figure is company-reported; there is no audited financial statement, and as a private company (now a subsidiary of a private parent) none is expected.
- **The identity of the historical BIN sponsor / issuing partner** for the pre-principal-issuer Reap Card era.
- **The current (2026) full product pricing** — the pricing page is gated behind registration.
- **Whether "Index Venture" (2020 round) is Index Ventures** or a differently-named entity — the legal entity could not be confirmed. (The 2026 About page's "Backed by" row renders the name as "Index Ventures," which narrows but does not close the question.)
- **The specific ML models and data-residency details** behind the Sentry fraud product and the AI expense-management layer.
- **The Thredd arrangement's terms** (US card-program expansion, Sep 2025): the processor is press-named, but the commercial terms, the timeline, and whether the US programs run under Reap's or Thredd's licences are not public.
- **The Business Account's regulatory classification** — the 2026 product line includes an account layer, but Reap publishes no regulatory treatment for it, and the MSO/principal-issuer licences of §9 do not obviously cover account-issuance.
- **The Rewards product's economics** — the loyalty tile exists on the site; its liability, funding, and partner structure are unpublished.

---

## 13. The Glossary

| Term | Meaning |
| --- | --- |
| **Agentic Payments** | Reap's 2026 product for AI-agent-initiated spending ("agent-native checkout"); tied by the company to Model Context Protocol (MCP). |
| **BIN (Bank Identification Number)** | The first six digits of a card number identifying the issuer; BIN sponsorship is when a licensed issuer lends its BIN to a program manager. |
| **Bill Pay** | Reap's 2026 product for paying fiat bills (local or cross-border) funded by stablecoins. |
| **Business Account** | Reap's 2026 "unified account layer for stablecoin-enabled businesses"; regulatory treatment unpublished. ⚠ |
| **Buyer-initiated payment** | Reap's founding concept: the payer's card funds a payment to a recipient's bank account, letting cardholders pay parties that cannot accept cards. |
| **Card program** | The end-to-end arrangement to issue and manage a card portfolio — network membership, processing, settlement, compliance, and client management. |
| **Card-issuing platform / BaaS** | An API product that lets another business launch branded card programs on the operator's infrastructure. |
| **Cyberport** | Hong Kong's digital-technology incubation hub; Reap joined its incubation programme in October 2018. |
| **Embedded finance** | Financial products (cards, payments) delivered inside another business's product via API. |
| **Fincra** | African payments company; signed an MoU with Reap (Sep 2025) for cards and cross-border payments between Africa and Asia. |
| **Fireblocks** | Digital-asset custody and settlement platform; Reap's custodian wallet for stablecoin operations. |
| **Fircosoft** | A sanctions-screening software vendor; the repository's reference guide for screening themes (cross-ref §9.4). |
| **Global Dollar Network (GDN)** | A stablecoin payments network; Reap joined in December 2025. |
| **Hyperliquid** | An on-chain trading/perps platform; Reap went live with USDC funding capability on it in August 2026. |
| **Interchange** | The fee a merchant's bank pays the cardholder's bank on a card transaction; the core issuer revenue line. |
| **KAST** | A card-program client quoted in Reap's 2026 homepage testimonial ("They got us from 0 to 1 fast"); marketing material. ⚠ |
| **MAS (Monetary Authority of Singapore)** | Singapore's central bank and financial regulator; administers the Payment Services Act licensing regime (SPI/MPI). |
| **MCP (Model Context Protocol)** | An open protocol for connecting AI agents to tools/data; Reap cites it for agentic payment flows. |
| **Money Service Operator (MSO)** | A Hong Kong licence (Customs and Excise Department) for remittance and money-changing services; held by Reap (Remit) Limited. |
| **Money Transmitter Registry (Mexico)** | Mexico's registration for money-transmission services (with the relevant authorities); Reap obtained it in March 2026. |
| **MPI (Major Payment Institution)** | The upper MAS licence tier under the PS Act for payment service providers above transaction thresholds. |
| **Multi-rail orchestration** | Routing payment legs across different rails (card scheme, local ACH/real-time, stablecoin) per transaction. |
| **Principal issuer / principal member** | A card-network member that issues cards and settles directly with the network without a third-party sponsor. |
| **Reap Card** | Reap's Visa corporate credit card (Nov 2021–), HKD/USD-denominated, stablecoin-collateralised since the 2025–2026 era. |
| **Reap Collect** | The historical request-to-pay product: collect card payments from customers into your bank account. |
| **Reap Direct** | Reap's 2025 platform operating across fiat and stablecoin: cards, cross-border payments, expense management. |
| **Reap Pay** | The historical bill-payment product: settle bank transfers from your corporate card, in the recipient's currency. |
| **Rewards** | Reap's 2026 loyalty/rewards product tile; economics unpublished. ⚠ |
| **Sentry** | Reap's 2026 fraud and risk management product line. |
| **Stablecoin** | A cryptocurrency pegged to a stable asset (e.g., USDC, USDT to the US dollar); used by Reap as collateral and settlement medium. |
| **SVF (Stored Value Facility)** | A Hong Kong Monetary Authority licence for stored-value/prepaid facilities; no evidence Reap holds one. |
| **TerraPay** | A cross-border payments network; partnered with Reap in May 2026 to expand local payout corridors. |
| **Thredd** | A card-processing platform; press-named (Sep 2025) as the processor for Reap's US card-program expansion. ⚠ |
| **Triple-A** | A crypto-payments gateway (MAS-licensed); Reap partnered with it in May 2023 to let businesses pay fiat bills with digital currencies. |
| **USYC** | A yield-bearing stablecoin (USD Yield Coin); Reap integrated it in June 2026 for treasury capabilities. |
| **Vulnerable Activities Registration** | Mexico's AML registration for designated "vulnerable activities," including card issuing and virtual assets. |
| **Wayback Machine** | The Internet Archive's web archive; used throughout this guide to verify historical Reap pages (2019, 2022, 2023 captures). |
| **YC (Y Combinator)** | The US startup accelerator; no verifiable affiliation with Reap was found. |

---

## 14. Cross-References and the Closing Summary

**Cross-references (repository convention: sibling `banking/` guides by plain filename; `technology/` guides prefixed `../technology/`):**

- [Payment Rails](payment_rails_guide.md) — the mechanics of cross-border rails, clearing and settlement (§5), and rail selection (§8); cited in §7.1 and §11.3.
- [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) — the PS Act, SPI/MPI licensing, and the Cymbal Bank persona conventions; cited in §1.3, §9.3, and §11.5.
- [Ancillary Revenue Products](ancillary_revenue_products_guide.md) — card-program economics (§10 there, the co-brand card program); cited in §4.3, §6.3, and §11.2.
- [Fircosoft](fircosoft_guide.md) — sanctions/AML screening themes; cited (condensed) in §9.4 and §11.5.
- [Citadel LLC](citadel_llc_guide.md) and [Resona Merchant Bank Asia](resona_merchant_bank_asia_guide.md) — the genre precedents: numbered ToC, byline, claims audit, glossary, worked-example conventions.
- [Enterprise Middleware & Integration Platforms](../technology/enterprise_middleware_integration_platform_guide.md) — API/integration-platform themes; cited in §7.3, §8.3, and §10.1.
- [Financial Management Systems](../technology/financial_management_systems_guide.md) — accounting/ERP systems, GL and close; cited in §7.3 and §11.4.

**Primary sources used this pass:** reap.global (homepage, About Us, media page, newsroom releases — Series A Oct 2022, Fireblocks Jan 2023, Triple-A May 2023, Chainalysis Jan 2024, Solana treasury Mar 2025, HK HQ Nov 2025, Fincra MoU Sep 2025, Global Dollar Network Dec 2025, Mexico MTR Mar 2026, Visa Mexico May 2026, TerraPay May 2026, USYC Jun 2026, Payward acquisition May 2026, Payward closing Jul 2026, Hyperliquid Aug 2026); Reap Terms & Conditions (Dec 2022 capture) and About pages (2019, 2020, 2023, 2026 captures) via the Wayback Machine; the 2019 seed release PDF (riseconf.com); PRNewswire seed release (Jul 2020); Jumpstart (Jul 2020); Bertelsmann announcement; MAS Payments regulation pages and exemption register (extracted live); Business Wire and American Banker coverage of the Payward closing; Bloomberg headline (via Reap's media page); Forbes (Oct 2022) and PYMNTS (Sep 2025) coverage linked from Reap's media page.

**The closing summary.** Reap's seven-year arc — from a Hong Kong startup that let small businesses pay anything with a credit card, through the Reap Card and a US$40 million Series A, to a stablecoin-native card-and-payments infrastructure business acquired by Payward for up to US$600 million — is a compressed lesson in how the corporate-payments stack is being rebuilt. The verified record is unusually clear on the fundamentals: founded 2018 by Daren Guo and Kevin Kang; Hong Kong-founded and headquartered; Hong Kong MSO and Visa principal licensing; Mexico money-transmitter and Visa principal membership; a documented partner ecosystem (Visa, Fireblocks, Chainalysis, Circle, Solana, TerraPay, Global Dollar Network); and a corporate history free of the Y Combinator affiliation and the "Daniel Kuo / Adil Mohammed" founders the research brief expected. What cannot be verified — volumes, revenue, profitability, the Singapore licence claim — is flagged rather than smoothed, per this repository's honesty convention. For Cymbal Bank, the takeaway is structural: the corporate card has become a programmable control point where card networks, banking rails, and stablecoin settlement meet, and a bank's role in that world is decided by how well it reconciles, screens, and settles across all three. Every program, every collateral line, and every cross-border payout eventually resolves into the same place — the spend ledger.
