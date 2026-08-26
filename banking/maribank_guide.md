# MariBank (Singapore): A Comprehensive Guide — The Sea Group Digital Bank

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Digital Banking / Banking Architecture — MariBank Deep-Dive, Singapore Digital Bank Landscape, Sea Group (Shopee / Garena / Monee) Ownership, MAS Digital Full Bank Licence (December 2020), Cloud-Native Core (Vendor Unverified — Flagged), High-Yield Savings + Invest + SME Banking, the "Mari" Brand Family
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** August 2026

> **How to read this guide:** this is the dedicated deep-dive on **MariBank** in the repo's digital-bank series — the companion to [trust_bank_guide.md](trust_bank_guide.md) (the SG digital-bank pattern), [green_link_digital_bank_guide.md](green_link_digital_bank_guide.md) and [tonik_digital_bank_guide.md](tonik_digital_bank_guide.md). It follows the repo convention of **verification flags**: every claim is marked **Verified** (primary/press-attested), **Flagged** (secondary, conflicting, or inference), or **Unverified**. The most important flag is at [§1.2](#12-critical-correction-maribank-is-sea-groups-bank-not-grab-singtels): **MariBank is Sea Group's bank, not the Grab–Singtel bank** — a correction this guide exists to make stick. Method note: `web_extract` was degraded to a search-only backend during this pass, so verification used targeted `web_search` against the primary sources (maribank.sg, MAS, ST, CNA, BT, DealStreetAsia, SBR, Fintech News SG, The Digital Banker) plus the repo's own verified guides; nothing was fabricated, and anything unverifiable is flagged inline.

---

## Table of Contents

1. [MariBank Overview](#1-maribank-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [CRITICAL CORRECTION: MariBank Is Sea Group's Bank, Not Grab-Singtel's](#12-critical-correction-maribank-is-sea-groups-bank-not-grab-singtels)
   - 1.3 [Verified Status at a Glance (Overview Table)](#13-verified-status-at-a-glance-overview-table)
   - 1.4 [The Licence: MAS Digital Full Bank, December 2020](#14-the-licence-mas-digital-full-bank-december-2020)
   - 1.5 [The Ownership: Sea Limited and the "Mari" Brand Family](#15-the-ownership-sea-limited-and-the-mari-brand-family)
   - 1.6 [Timeline: 2019–2026](#16-timeline-20192026)
   - 1.7 [What Could NOT Be Verified](#17-what-could-not-be-verified)
2. [The Launch](#2-the-launch)
   - 2.1 [The Soft Launch (2022 — Flagged)](#21-the-soft-launch-2022--flagged)
   - 2.2 [The Public Launch (March 2023)](#22-the-public-launch-march-2023)
   - 2.3 [Launch Table: Event / Date / Notes](#23-launch-table-event--date--notes)
3. [The Products](#3-the-products)
   - 3.1 [Mari Savings Account](#31-mari-savings-account)
   - 3.2 [The Product Family: Mari Invest, Mari Insurance, Business Banking](#32-the-product-family-mari-invest-mari-insurance-business-banking)
   - 3.3 [Products Table: Product / Type / Notes](#33-products-table-product--type--notes)
4. [The Ecosystem](#4-the-ecosystem)
   - 4.1 [CORRECTION: The Sea / Shopee Ecosystem, Not the Grab Ecosystem](#41-correction-the-sea--shopee-ecosystem-not-the-grab-ecosystem)
   - 4.2 [The Sea Group Ecosystem and How MariBank Sits Inside It](#42-the-sea-group-ecosystem-and-how-maribank-sits-inside-it)
   - 4.3 [Ecosystem Table](#43-ecosystem-table)
5. [The Technology](#5-the-technology)
   - 5.1 [Architecture Posture: Cloud-Native, API-First (Consensus — Flagged)](#51-architecture-posture-cloud-native-api-first-consensus--flagged)
   - 5.2 [The Core Banking System: Vendor Unverified (Flagged)](#52-the-core-banking-system-vendor-unverified-flagged)
   - 5.3 [Identity, Payments, and Compliance](#53-identity-payments-and-compliance)
   - 5.4 [Technology Table: Component / Description / Notes](#54-technology-table-component--description--notes)
   - 5.5 [Architecture Risks](#55-architecture-risks)
6. [The Progress](#6-the-progress)
   - 6.1 [Deposits and Customers: The Numbers Problem (Flagged)](#61-deposits-and-customers-the-numbers-problem-flagged)
   - 6.2 [What Is Verified: Capital, AUM, Awards](#62-what-is-verified-capital-aum-awards)
   - 6.3 [The Profitability Question (Flagged)](#63-the-profitability-question-flagged)
   - 6.4 [Progress Table](#64-progress-table)
7. [The Competition](#7-the-competition)
   - 7.1 [The Singapore Digital Bank Landscape](#71-the-singapore-digital-bank-landscape)
   - 7.2 [Trust vs GXS vs MariBank: The Consumer Face-Off](#72-trust-vs-gxs-vs-maribank-the-consumer-face-off)
   - 7.3 [Competition Table: Bank / Focus / Notes](#73-competition-table-bank--focus--notes)
   - 7.4 [MariBank's Competitive Position](#74-maribanks-competitive-position)
8. [The Worked Example: A Sea-Ecosystem Customer Journey](#8-the-worked-example-a-sea-ecosystem-customer-journey)
   - 8.1 [Scenario (Corrected): The Shopee Ecosystem User](#81-scenario-corrected-the-shopee-ecosystem-user)
   - 8.2 [The Journey Flow](#82-the-journey-flow)
   - 8.3 [Lessons From the Journey](#83-lessons-from-the-journey)
9. [The Summary: One Page — "The Superapp's Bank"](#9-the-summary-one-page--the-superapps-bank)
10. [Glossary](#10-glossary)
11. [References and Cross-References](#11-references-and-cross-references)

---

## 1. MariBank Overview

### 1.1 The Short Answer

**MariBank (legal name: MariBank Singapore Pte. Ltd.) is Singapore's third consumer digital bank** — the digital full-bank arm of **Sea Limited**, the NYSE-listed Singapore consumer-internet group behind **Shopee** (e-commerce), **Garena** (digital entertainment) and **Monee** (digital financial services, formerly SeaMoney). MariBank holds one of the two **MAS digital full bank licences** awarded in **December 2020** — the other going to the **Grab–Singtel consortium**, which operates as **GXS Bank**.

The verified story, in brief:

- **Ownership:** Sea Limited (via its digital financial services arm, now branded **Monee**) — *not* Grab and Singtel. Grab + Singtel own **GXS Bank**, MariBank's sibling licence-holder, not MariBank itself. This guide exists partly to fix that conflation (see [1.2](#12-critical-correction-maribank-is-sea-groups-bank-not-grab-singtels)).
- **Licence:** MAS **digital full bank (DFB) licence** — one of four digital-bank-framework licences announced **4 December 2020** (two DFBs: MariBank and GXS Bank; two digital wholesale banks: GLDB and ANEXT Bank). Full DFB licence obligations: S$1.5 billion phased-in base capital, retail deposits allowed, SDIC-insured.
- **Launch:** A 2022 soft/beta phase (flagged — exact date unverified), then a **public launch in March 2023**, initially on an invite-only basis, led by the **Mari Savings Account** at **2.5% p.a.** with no minimum deposit (verified via DealStreetAsia and contemporaneous press of 17 March 2023).
- **Positioning:** "Banking that is simple, reliable and rewarding for everyone" — the high-yield, no-hoops savings account; the **Mari Invest** wealth arm (SavePlus cash-management fund launched September 2023; AUM surpassed **S$953 million** per Singapore Business Review); the **Mari Business Account** and business fixed deposits for SMEs; and an expanding family (credit card and loan products listed on maribank.sg by 2026 — launch details flagged).
- **Current status (2026):** MariBank is the banking anchor of Sea's regional "Mari" banking group — **MariBank Singapore** plus **MariBank Philippines** (formerly SeaBank Philippines, upgraded to a digital bank licence in July 2026). It remains in investment mode and loss-making (unverified at entity level — flagged), does not disclose SG deposits or customer counts consistently (flagged), and competes head-to-head with **Trust Bank** (the largest digital bank by customers) and **GXS Bank** (the Grab–Singtel digital bank).

**Key numbers at a glance (all statuses per [1.3](#13-verified-status-at-a-glance-overview-table)):**

| Number | Value | Status |
|---|---|---|
| Licence award | **4 December 2020** (MAS DFB) | Verified |
| Public launch | **March 2023** (invite-only) | Verified |
| Launch savings rate | **2.5% p.a.** | Verified |
| Peak savings rate | **2.88% p.a.** (mid-2023) | Verified |
| Current rate (Aug 2026) | ~2.28% flat / 0.88% + up-to-2.88% promo | Flagged (aggregator) |
| Mari Invest SavePlus AUM | **> S$953.3M** | Verified (SBR) |
| Parent capital injection | **S$173M** (2023) | Verified (DealStreetAsia) |
| SG deposits / customers | **Not disclosed** | Flagged |
| SG profitability | **No public claim** | Flagged |
| Regional footprint | SG (DFB) + PH (Jul 2026 upgrade) + SeaBank ID | Verified |

### 1.2 CRITICAL CORRECTION: MariBank Is Sea Group's Bank, Not Grab-Singtel's

A recurring error — including in the research brief that triggered this guide — describes MariBank as **"the Grab–Singtel digital bank"** and its ecosystem as the **"Grab ecosystem"**. **Both are wrong.** The verified facts (MAS press release, 4 December 2020; Wikipedia/Sea disclosures; The Diplomat, June 2024; CNA, February 2025):

- **MariBank** = Sea Limited's digital bank. Sea's core businesses: **Shopee** (e-commerce), **Garena** (gaming), **Monee** (digital financial services, ex-SeaMoney). MariBank is the digital banking arm of Monee.
- **GXS Bank** = the **Grab–Singtel** consortium's digital bank (Grab 60%, Singtel 40% — verified via The Digital Banker / FMT, July 2024). GXS integrates with the **Grab superapp** (rides, food delivery) and Singtel.
- The two are **sibling licence-winners from the same December 2020 MAS round** — often confused because they launched within months of each other and are both "Grab-adjacent" in public perception (Grab and Shopee are both Singapore-born superapp-era companies). The Grab–Singtel JV is GXS; the Sea bank is MariBank.

The same correction is documented in the sibling guide [green_link_digital_bank_guide.md](green_link_digital_bank_guide.md) (§1), which also corrects the wider error that "the four December 2020 winners were GXS, MariBank, Trust and ANEXT" — the MAS press release of 4 December 2020 actually named **GXS Bank and MariBank (DFB)** and **Green Link Digital Bank and ANEXT Bank (DWB)**; **Trust Bank** received a full bank licence separately and is the *fifth* digital bank (see [trust_bank_guide.md](trust_bank_guide.md) §1.4 for the full correction).

**Implication for this guide:** wherever a source says "Grab user journey" or "Grab ecosystem integration" in the MariBank context, read **Shopee / Sea ecosystem**. The worked example in [§8](#8-the-worked-example-a-sea-ecosystem-customer-journey) is therefore written around a Shopee-ecosystem user, not a Grab user.

**Brief vs verified (the delta table).** The research brief that commissioned this guide contained a systematic mis-framing. The verified record:

| Brief's claim | Verified reality | Source |
|---|---|---|
| MariBank is "the Grab–Singtel digital bank" | MariBank is **Sea Limited's** digital bank | MAS press release (4 Dec 2020); The Diplomat (Jun 2024); CNA (Feb 2025) |
| Ownership = "Grab + Singtel JV" | Ownership = **Sea** (via Monee/ex-SeaMoney); Grab+Singtel own **GXS Bank** | ST; Fintech News SG; The Digital Banker (Jul 2024) |
| Ecosystem = "Grab ecosystem integration" | Ecosystem = **Sea/Shopee**: Shopee marketplace, ShopeePay/Monee wallet, Garena; regional group SG+PH | BT ("MariBank taps Sea's ecosystem..."); ST (Monee HQ) |
| Worked example = "the Grab user" | Worked example = **the Shopee-ecosystem user** ([§8](#8-the-worked-example-a-sea-ecosystem-customer-journey)) | correction per above |
| Licence: MAS digital full bank, 2020 award | **Confirmed** — DFB, awarded 4 Dec 2020 (one of two DFBs with GXS) | MAS media release |
| Soft launch 2022 / public 2023 | Soft/beta **2022 (date flagged)**; public **March 2023** invite-only | DealStreetAsia; Jom Media (17 Mar 2023); repo consensus |

The delta table is the audit trail: everything in the brief that survived verification is in this guide as **Verified**; everything that did not is corrected here rather than repeated downstream.

### 1.3 Verified Status at a Glance (Overview Table)

| Aspect | Description | Verification status |
|---|---|---|
| **What it is** | Digital bank (Singapore) — MAS-licensed, mobile-first, no physical branches | **Yes** — maribank.sg; MAS digital-bank framework |
| **Legal entity** | MariBank Singapore Pte. Ltd. (digital banking arm of Sea's Monee) | Yes (industry/press; entity-name detail flagged) |
| **Owner** | **Sea Limited** (NYSE: SE) — parent of Shopee, Garena, Monee (ex-SeaMoney) | **Yes** — Sea disclosures, ST, Fintech News SG, The Diplomat |
| **NOT owned by** | Grab / Singtel (they own **GXS Bank**, the other Dec-2020 DFB winner) | **Yes** — MAS press release, The Digital Banker, FMT |
| **Licence** | MAS **digital full bank (DFB)** licence — one of four awarded in the Dec-2020 round (two DFBs: MariBank, GXS; two DWBs: GLDB, ANEXT) | **Yes** — MAS media release, 4 December 2020 |
| **Licence conditions** | S$1.5B phased-in base capital; retail deposits permitted; SDIC deposit insurance applies | Yes — MAS framework (per [full_stack_banking_guide.md](full_stack_banking_guide.md)) |
| **Soft launch** | 2022 beta/pilot phase (limited onboarding) | **Partially** — repo consensus ("pilot Mar 2022", [full_stack_banking_guide.md](full_stack_banking_guide.md)); exact date flagged |
| **Public launch** | **March 2023**, invite-only first, Mari Savings Account at **2.5% p.a.** | **Yes** — DealStreetAsia; Jom Media (17 Mar 2023) |
| **Rate peak** | Raised to **2.88% p.a.** in mid-2023 | Yes — Straits Times (headline verified via search) |
| **Current rate (Aug 2026)** | ~2.28% p.a. flat (Dec 2025, SingSaver) / 0.88% base + up to 2.88% new-user bonus (Aug 2026, Growbeansprout) | **Flagged** — aggregator-reported; rates change |
| **Products** | Mari Savings + debit card; Mari Invest (SavePlus, Income); Mari Business Account + Business Fixed Deposit; overseas transfers; credit card/loan (listed on site) | Mixed — verified individually; credit card/loan flagged |
| **Mari Insurance** | Advertised as part of the Mari family | **Unverified** — no launch detail confirmed in this pass |
| **Core technology** | Cloud-native build (repo + industry consensus); **vendor not publicly disclosed** | **Flagged** — see [§5](#5-the-technology) |
| **Deposits / customers** | Not consistently disclosed; press figures conflict | **Flagged** — see [§6](#6-the-progress) |
| **Profitable?** | No public claim of profitability; in investment mode | **Flagged** — see [6.3](#63-the-profitability-question-flagged) |
| **Awards** | "New Virtual Bank of the Year (Singapore)" (Asian Banking & Finance, 2024); "Best Digital Bank for Integrated Investing Options" (2025) | Yes — maribank.sg |

### 1.4 The Licence: MAS Digital Full Bank, December 2020

**The MAS digital bank framework (June 2019).** In 2019, MAS opened applications for up to **five new digital bank licences** — two **digital full bank (DFB)** licences and three **digital wholesale bank (DWB)** licences — to let non-bank players (fintechs, e-commerce groups, telcos) operate banks in Singapore without an incumbent branch network. DFB holders may take retail deposits and must meet the same capital standards as full banks (S$1.5 billion base capital, phased in); DWB holders may serve SMEs and wholesale customers only (S$100 million base capital; no retail deposits).

**The award (4 December 2020).** MAS announced the four successful applicants:

| Winner | Licence type | Parent |
|---|---|---|
| **GXS Bank** | Digital Full Bank | Grab–Singtel consortium (Grab 60% / Singtel 40%) |
| **MariBank** | Digital Full Bank | **Sea Limited (wholly-owned entity)** |
| Green Link Digital Bank (GLDB) | Digital Wholesale Bank | Linklogis-led consortium (with Greenland Financial, SBI Ven) |
| ANEXT Bank | Digital Wholesale Bank | Ant Group |

MAS noted the applicants "must meet all relevant prudential requirements and licensing pre-conditions before MAS grants them their respective banking licences", with commencement expected "from early 2022". One DWB slot was left unfilled. The award made Sea one of only two non-bank groups in Singapore licensed to take retail deposits from day one of the framework. (Source: MAS media release, 4 December 2020 — verified via search snippet of mas.gov.sg; the release text itself was not re-extracted in full this pass because the extract backend is degraded — the award facts are corroborated by ST, The Diplomat, CNA, Seedly and the repo's [green_link_digital_bank_guide.md](green_link_digital_bank_guide.md).)

**Why Sea qualified.** MAS's framework explicitly wanted applicants with "uberisation" potential — platforms that could embed banking into daily digital life. Sea brought Shopee (Southeast Asia's largest e-commerce platform), a payments/credit track record via SeaMoney (ShopeePay wallets, SPayLater, SeaBank Indonesia, SeaBank Philippines), and Garena's consumer scale. MariBank was the natural consolidation point for that in Singapore.

**Deposit insurance.** MariBank SGD deposits of non-bank depositors are covered by the **Singapore Deposit Insurance Corporation (SDIC)** scheme — capped at **S$75,000** per depositor per bank until April 2024, then raised to **S$100,000** (verified via maribank.sg app-store listing and the ST deposit-cap coverage). Deposit insurance is a core trust mechanism for a branchless bank.

### 1.5 The Ownership: Sea Limited and the "Mari" Brand Family

**The parent.** Sea Limited (NYSE: SE) — founded in Singapore (2009), one of Southeast Asia's largest internet companies. Three business lines:

- **Shopee** — e-commerce marketplace across SEA and Latin America; the region's largest.
- **Garena** — digital entertainment (Free Fire etc.).
- **Monee** — digital financial services (formerly **SeaMoney**; rebranded **Monee** in 2025 — verified via Straits Times and Fintech News SG coverage of Sea's new Singapore financial-services HQ). Monee spans wallets (ShopeePay), credit (SPayLater), SeaBank (Indonesia), **MariBank (Singapore)** and **SeaBank Philippines** (→ MariBank Philippines).

**The "Mari" brand family.** Sea consolidated its Singapore consumer-finance products under the **Mari** brand: **MariBank** (banking), **Mari Invest** (wealth/investment), **Mari Insurance** (insurance — flagged, see [3.2](#32-the-product-family-mari-invest-mari-insurance-business-banking)). The "Mari" name is also applied regionally: **MariBank Philippines** (ex-SeaBank, upgraded to a Philippine digital-bank licence July 2026 — verified via the sibling guide [tonik_digital_bank_guide.md](tonik_digital_bank_guide.md), which tracks the Philippine cohort).

**The regional banking group.** In 2025 Sea confirmed the formation of a **regional banking group under the MariBank brand** (BT: "MariBank taps Sea's ecosystem to build regional banking group from Singapore"; Fintech News SG: "Sea also confirmed the formation of a regional banking group under MariBank, which holds a digital banking licence in Singapore"). The group comprises MariBank Singapore (DFB) and MariBank Philippines; Indonesia's SeaBank is the third regional node. The architecture intent: a shared Sea fintech platform, one brand, multiple licences.

**Capital.** MariBank received parent injections from Sea — DealStreetAsia reported a **S$173 million** top-up (mid-2023, headline: "SG digital lender MariBank bags $173m from parent Sea Ltd"). Sea does not publish MariBank standalone financials in its quarterly earnings, so the full capitalisation is not publicly itemised (flagged). For contrast, GXS Bank's parents (Grab 60% / Singtel 40%) have committed **S$1.5 billion** through March 2027 (The Digital Banker / FMT, July 2024).

### 1.6 Timeline: 2019–2026

| Date | Event |
|---|---|
| Jun 2019 | MAS launches the digital bank framework (2 DFB + 3 DWB licences) |
| Dec 2020 (4th) | MAS awards DFB licences to **GXS Bank (Grab–Singtel)** and **MariBank (Sea)**; DWBs to GLDB and ANEXT |
| 2021 | Sea builds out the bank; MAS pre-conditions (capital, governance, operational readiness) |
| 2022 | **Soft launch / beta** — MariBank app appears; limited onboarding (exact date flagged; repo consensus "pilot Mar 2022") |
| Mar 2023 | **Public launch** — invite-only rollout, Mari Savings Account at 2.5% p.a., no minimum balance |
| Jun–Jul 2023 | Rate raised to **2.88% p.a.** (ST); deposit cap aligned at S$75,000 (SDIC limit at the time) |
| Sep 2023 | **Mari Invest** launches (SavePlus cash-management fund — SBR) |
| 2024 | "New Virtual Bank of the Year (Singapore)" award (Asian Banking & Finance); deposit insurance cap rises to S$100,000 (Apr 2024); Mari Insurance signalled (flagged) |
| 2025 | SeaMoney rebrands to **Monee**; MariBank regional banking group announced; **Mari Invest Income** (global-bond monthly-income product) and SavePlus AUM > S$953M (SBR); "Best Digital Bank for Integrated Investing Options" award; overseas-transfer service — first among SG digital banks (ST) |
| 2025–2026 | Credit card and loan products appear on maribank.sg (details flagged); Mari Business Account + Business Fixed Deposit live |
| Jul 2026 | **MariBank Philippines** (ex-SeaBank) upgrades to a digital bank licence in the Philippines (per [tonik_digital_bank_guide.md](tonik_digital_bank_guide.md)) |

### 1.7 What Could NOT Be Verified

- **Exact soft-launch date (2022)** — press coverage is thin; the repo consensus ("pilot Mar 2022" in [full_stack_banking_guide.md](full_stack_banking_guide.md)) is treated as indicative, flagged.
- **The exact date of general (non-invite) public availability** — DealStreetAsia and Jom Media verify an invite-only March 2023 launch; a June-2026 secondary source (The Kopi Notes) claims "October 2023" for the full public launch — conflicting, flagged.
- **MariBank SG deposits and customer counts** — not consistently disclosed (see [§6](#6-the-progress)).
- **Core-banking vendor** — not publicly confirmed (see [§5](#5-the-technology)).
- **Mari Insurance specifics** — launch date, products, underwriter unverified in this pass.
- **Entity-level financials / profitability** — MariBank does not file standalone public financials in this pass's sources.
- **Current promotional interest-rate structure** — aggregator-reported and volatile (0.88% base + new-user bonus to 2.88%, Aug 2026; 2.28% flat, Dec 2025) — flagged as of writing.

---
## 2. The Launch

### 2.1 The Soft Launch (2022 — Flagged)

MariBank's path to market followed the now-familiar digital-bank playbook: a quiet **beta / soft-launch phase before the public debut**. The verified record:

- The **MAS licence round** was announced December 2020; MAS expected successful applicants to commence "from early 2022".
- The repo's own digital-bank series records a **March 2022 pilot** for MariBank ("pilot Mar 2022", [full_stack_banking_guide.md](full_stack_banking_guide.md) §7) — i.e., the app and onboarding opened to a limited group (Sea staff and early invitees) in early 2022.
- The **Google Play listing** for the MariBank SG app ("sg.com.maribankmobile.digitalbank") is dated to this period; app-store history corroborates a 2022 beta presence, though the exact go-live month is **not independently verified in this pass** and is flagged.

What the soft launch was *for*: proving the onboarding flow (Singpass/KYC), the payments rails (PayNow/FAST), the deposit engine, and regulatory reporting — at small scale, before the marketing budget switched on. This mirrors GXS Bank (Aug 2022 public launch after a staff beta) and Trust Bank (commenced banking business 15 Dec 2021, public launch 1 Sep 2022 — see [trust_bank_guide.md](trust_bank_guide.md)).

### 2.2 The Public Launch (March 2023)

MariBank's public launch was **March 2023**, rolled out **on an invite-only basis to select members of the public** — verified by DealStreetAsia ("MariBank launched its digital banking services in Singapore in March this year to select members of the public on an invite-only basis", from its piece on Sea's S$173M capital injection) and by contemporaneous Singapore press of 17 March 2023 (Jom Media's "Singapore This Week", which highlighted "MariBank's savings account offers a compelling interest rate of 2.5 percent per year without any minimum deposit, monthly salary credit requirement or minimum spend").

**The launch product — Mari Savings Account.** One product, deliberately simple:

- **2.5% p.a.** on SGD deposits — at the time among the highest headline savings rates in Singapore, and notably **no-hoops**: no minimum deposit, no salary-credit condition, no minimum spend, no lock-in (per Jom Media, Mar 2023; later confirmed by [maribank.sg](https://www.maribank.sg/product/mari-savings-account)).
- **Daily interest crediting** — interest accrues daily (a differentiator against incumbent monthly crediting; the account's "earn while you wait" design suited the rate-competition era).
- **SDIC-insured** — up to S$75,000 per depositor (the cap then; raised to S$100,000 from April 2024).
- **Digital onboarding** — Singapore mobile number + Singpass; no branch, no paperwork (per maribank.sg help centre).

**The 2023 rate war.** Within months MariBank raised the headline rate to **2.88% p.a.** (Straits Times: "MariBank raises interest rates on savings account to 2.88% as digital banks seek to draw customers"), and both MariBank and GXS Bank moved to raise their deposit caps to **S$75,000** in July 2023 as they opened to broader public demand. The rate competition of mid-2023 — Trust, GXS and MariBank all pushing 2.5–3.0%+ on first-deposit tranches — defined the SG digital-bank battle (see [§7](#7-the-competition)).

**Later rate trajectory (flagged).** Rates have since normalised with the SGD interest-rate cycle: ~2.28% p.a. flat per SingSaver's December 2025 review, and a 0.88% base + up to 2.88% new-user promotional structure per Growbeansprout (August 2026). **Treat any current rate as of-writing data** — MariBank adjusts rates monthly.

**The launch mechanics.** The March 2023 roll-out was **invite-only** — "select members of the public" (DealStreetAsia) — a deliberate staged approach: (1) prove the onboarding and payments flows at controlled volume; (2) let word-of-mouth and the 2.5% headline rate do the marketing; (3) widen access as deposit caps and risk controls allowed. This mirrored GXS Bank's staged opening (staff beta → public August 2022 → first-come openings as caps were raised in 2023) and Trust's beta-then-public sequence (commenced banking business 15 Dec 2021; public launch 1 Sep 2022 — see [trust_bank_guide.md](trust_bank_guide.md) §1.1). The stage-gating is partly MAS-driven: a new bank's deposit book must stay inside its capital and liquidity envelope, and SDIC-insured deposit caps (S$75,000 then) made the invite model a natural capacity control.

**The launch reception.** Press coverage of the March 2023 launch framed MariBank as the third player in the SG digital-bank race and emphasised the **2.5% no-strings rate** as its differentiator (Jom Media, 17 Mar 2023: "no minimum deposit, monthly salary credit requirement or minimum spend"). The CNA February 2025 landscape piece later grouped MariBank with GXS and Trust as the three retail digital banks that positioned themselves as "transparent" and "fuss-free" versus incumbents. The Straits Times' mid-2023 coverage ("MariBank raises interest rates on savings account to 2.88% as digital banks seek to draw customers") captured the moment the rate war turned MariBank into the price leader of the cohort.

### 2.3 Launch Table: Event / Date / Notes

| Event | Date | Notes |
|---|---|---|
| MAS DFB licence awarded to Sea (MariBank) | **4 Dec 2020** | One of two DFBs (with GXS); MAS press release verified |
| MariBank entity incorporated / build-out | 2021 | Pre-conditions: capital, governance, ops readiness (flagged — entity incorporation date not verified) |
| **Soft launch / beta** | **2022 (early)** | Limited onboarding; repo consensus "pilot Mar 2022" — **date flagged** |
| **Public launch (invite-only)** | **March 2023** | Mari Savings Account at **2.5% p.a.**, no minimum deposit; verified (DealStreetAsia, Jom Media 17 Mar 2023) |
| Rate raised to 2.88% p.a.; deposit cap S$75,000 | **Mid-2023 (Jun–Jul)** | Straits Times; both MariBank and GXS opened to broader public |
| General public availability | Late 2023 (secondary sources claim "Oct 2023" — **conflicting, flagged**) | Kopi Notes (Jun 2026) vs DealStreetAsia (Mar 2023 invite-only) |
| Mari Invest launch | **Sep 2023** | SavePlus fund; verified via SBR |
| SDIC cap raised to S$100,000 | **Apr 2024** | National scheme change, applies to MariBank |
| Overseas transfers live (first among SG digital banks) | 2025 | Straits Times; MariBank was first SG digital bank to offer overseas transfers to retail and business users |

---

## 3. The Products

### 3.1 Mari Savings Account

The flagship. A **plain-vanilla, high-yield SGD savings account**:

- **Rates (history):** 2.5% p.a. at launch (Mar 2023) → 2.88% p.a. (mid-2023) → ~2.28% flat (Dec 2025, SingSaver) → 0.88% base + up to 2.88% new-user bonus (Aug 2026, Growbeansprout). **All current rates flagged as aggregator-reported.**
- **No hoops:** no minimum balance, no salary credit, no card-spend condition (at launch; check current T&Cs — the Aug-2026 structure implies a promo tier).
- **Daily interest** accrual and crediting.
- **SDIC-insured** up to S$100,000 (S$75,000 before April 2024).
- **Mari Debit Card** — issued automatically with the account (Growbeansprout); spending directly from the savings balance; no annual fee, no minimum spend. A numberless-style app-centric card in the digital-bank mould (details flagged).
- **Payments rails:** PayNow and FAST in/out (verified by function; the specific rail documentation is standard for MAS-licensed banks).
- **Eligibility:** Singaporeans, PRs, and foreigners with a valid pass (maribank.sg).

**How the mechanics work (verified pattern, exact T&Cs flagged).** Interest accrues **daily** on the end-of-day balance and is credited monthly — the "earn while you wait" model that made the account a parking spot for idle cash (the same engine design GXS and Trust use; the *daily accrual* is the part that matters architecturally — it forces a near-real-time interest engine in the core, see [§5](#5-the-technology)). There is **no lock-in and no minimum balance**, so the account is fully liquid via PayNow/FAST. The promotional tiers seen in 2026 (0.88% base + up to 2.88% for new users, per Growbeansprout) suggest the account has moved from "one flat rate" to "flat base + acquisition bonus" — a sign the cohort's rate war has normalised into promo-led acquisition (flagged as aggregator data, Aug 2026).

**What it is not:** MariBank does not (yet) run a multi-tier "salary-credit + spend + invest" rate engine of the kind Trust (Linkpoints tiers) or GXS (Grab ecosystem boosts) use. Its launch proposition was deliberately **one flat rate, no conditions** — the "fuss-free" positioning the CNA Feb-2025 landscape piece attributes to the digital-bank cohort.

### 3.2 The Product Family: Mari Invest, Mari Insurance, Business Banking

**Mari Invest (wealth).** Launched **September 2023** (SBR). A robo-lite investment shelf inside the MariBank app, investing from **S$1** with no transaction fees (maribank.sg):

- **Mari Invest SavePlus** — the launch product: a cash-management/SGD money-market-style fund. Initially offered via **Fullerton Fund Management's SGD Cash Fund** (per industry coverage), later joined by the **Lion-MariBank SavePlus Fund SGD**, managed by **Lion Global Investors** (LGI), with a promotional 0.125% p.a. management fee to 30 June 2024 (HardwareZone forum + Turtle Investor coverage — aggregator/community sources, flagged as such). **AUM surpassed S$953.3 million** since September 2023 introduction (SBR, at the Mari Invest Income unveiling).
- **Mari Invest Income** — launched 2025: global-bond exposure with **monthly income** distributions, low barrier to entry (fintechnews.sg; SBR). Aimed at the "consistent income" segment of SG retail investors.
- **Mari Invest SavePlus instant cash-out** — liquidity feature with a S$10,000 daily limit per investor (fintechnews.sg).

The strategic point: Mari Invest converts MariBank from a *deposit* product into a *wealth-distribution* product — the deposit account is the on-ramp, the fund shelf the margin. "Best Digital Bank for Integrated Investing Options" (Asian Banking & Finance Retail Banking Awards 2025, per maribank.sg) is the recognition of that integration.

**Mari Insurance (flagged).** MariBank advertises **Mari Insurance** as part of the Mari family, and Sea has signalled insurance distribution as a natural Monee/Mari extension. **However, this pass could not verify** a specific Mari Insurance product launch, policy types, or underwriter for Singapore — the search trail surfaced only the brand name (maribank.sg navigation) and unrelated TikTok/PH content. **Treat "Mari Insurance" as a brand-family placeholder with unverified specifics**; cross-ref the repo's insurance guides for the SG distribution model ([insurance_software_systems_guide.md](insurance_software_systems_guide.md)).

**Business banking (SME).** MariBank extended into SME banking — a notable strategic choice for a consumer DFB licence:

- **Mari Business Account** — SGD account for Singapore-incorporated businesses, opened and managed entirely in the MariBank app (Airwallex SG review, 2026: "MariBank is a digital bank in Singapore owned by Sea Group, the parent company of Shopee. The Mari Business Account lets Singapore-incorporated businesses open and manage an SGD account entirely through the MariBank app").
- **Mari Business Fixed Deposit** — term-deposit product for business cash (maribank.sg).
- **Overseas transfers** — MariBank was **the first digital bank in Singapore to roll out an overseas transfer service for retail and business users** (ST, via the Monee/HQ coverage).
- The SME push is a direct channel into **Shopee's merchant base** — Sea's ecosystem advantage (see [§4](#4-the-ecosystem)) and the mirror image of GXS's SME FlexiLoan and Trust's business deposits.

**Credit card and loans (2025–2026, flagged).** maribank.sg's fees-and-rates page now lists **savings account, credit cards, and loan** products (2026), and a June-2026 secondary source (The Kopi Notes) claims MariBank rolled out its SG credit card and personal loans "throughout 2024–2025". These are **not independently verified in this pass** (MariBank PH's first credit card, June 2026, is separately documented by Fintech News PH — the SG card timeline is flagged).

### 3.3 Products Table: Product / Type / Notes

| Product | Type | Notes |
|---|---|---|
| **Mari Savings Account** | Retail deposit (savings) | Flagship; 2.5% at launch (Mar 2023) → 2.88% peak (2023) → ~2.28% flat (Dec 2025) / 0.88% + up-to-2.88% promo (Aug 2026 — flagged); SDIC-insured; daily interest; no minimum balance |
| **Mari Debit Card** | Card (debit) | Auto-issued with savings account; no annual fee, no minimum spend (Growbeansprout; details flagged) |
| **Mari Invest — SavePlus** | Wealth (cash-management fund) | Launched **Sep 2023**; Fullerton SGD Cash Fund → Lion-MariBank SavePlus Fund (LGI); AUM > **S$953M** (SBR); invest from S$1; instant cash-out (S$10k/day limit) |
| **Mari Invest — Income** | Wealth (global-bond income) | Launched 2025; monthly income distributions; fintechnews.sg / SBR |
| **Mari Insurance** | Insurance (family brand) | **Unverified** — brand advertised; no product/underwriter confirmed in this pass |
| **Mari Business Account** | SME deposit | Singapore-incorporated businesses; app-only onboarding (Airwallex SG 2026) |
| **Mari Business Fixed Deposit** | SME term deposit | Business cash placement (maribank.sg) |
| **Overseas Transfers** | Payments (cross-border) | First among SG digital banks (ST, 2025); retail + business |
| **Credit card / personal loans (SG)** | Credit | Listed on maribank.sg fees page (2026); SG launch timeline **flagged** (Kopi Notes claim 2024–25 unverified; MariBank PH card launched Jun 2026) |

---

## 4. The Ecosystem

### 4.1 CORRECTION: The Sea / Shopee Ecosystem, Not the Grab Ecosystem

The research brief for this guide framed MariBank's ecosystem play as "the Grab ecosystem integration". **That is GXS Bank's story, not MariBank's.** GXS (Grab 60% / Singtel 40%) is the digital bank embedded in the **Grab superapp** — ride-hailing, food delivery, GrabPay wallets — plus Singtel's telco customer base (see [§7](#7-the-competition)). MariBank is embedded in the **Sea ecosystem**: Shopee's marketplace, SeaMoney/Monee's wallets and credit, Garena's entertainment audience. Any "Grab user journey" for MariBank is a category error; the correct scenario is the **Shopee / Sea user journey** ([§8](#8-the-worked-example-a-sea-ecosystem-customer-journey)).

### 4.2 The Sea Group Ecosystem and How MariBank Sits Inside It

Sea is Southeast Asia's closest thing to a regional superapp *conglomerate*: not one app, but a portfolio of category leaders sharing identity, data and capital. MariBank's ecosystem levers:

- **Shopee (e-commerce).** The region's largest marketplace. Two user surfaces for MariBank: (1) **buyers** — high-frequency wallet/payments users (ShopeePay) who can be offered a savings account that pays daily interest on idle cash; (2) **sellers/merchants** — hundreds of thousands of SMEs across SEA, the natural target for the **Mari Business Account**, business fixed deposits, and (regionally) SPayLater-style credit. The BT line "MariBank taps Sea's ecosystem to build regional banking group from Singapore" is precisely this: distribution via Shopee's merchant base, banking via MariBank.
- **Monee / SeaMoney (digital financial services).** The holding arm that owns MariBank; includes ShopeePay wallet, SPayLater BNPL credit, and the regional SeaBank licences (Indonesia; Philippines → MariBank PH). MariBank is the Singapore-regulated crown jewel of this stack — the entity that can take retail deposits under a MAS DFB licence.
- **Garena (gaming).** Free Fire's consumer base is a brand-distribution channel (promotions, vouchers), though banking-relevant integration is thinner than Shopee's.
- **Regional banking group (2025–).** MariBank Singapore (DFB) + MariBank Philippines (digital bank licence, Jul 2026) + SeaBank Indonesia — a "one brand, many licences" architecture with shared Sea fintech platform services (KYC, payments, data). Singapore is the flagship because of the MAS licence and the mature rails (PayNow, Singpass, SDIC).

**Integration mechanics (verified pattern, detail flagged).** MariBank sits in the same Sea fintech platform as ShopeePay/SPayLater, so the practical integrations are: app-level cross-sell (savings offers inside Shopee flows), wallet-to-bank transfers, merchant onboarding data shared with the business account, and the Monee credit stack feeding lending. The *exact* API surface and data-sharing arrangements are not public — flagged, but the ecosystem logic is corroborated by BT/ST/Fintech News SG coverage.

### 4.3 Ecosystem Table

| Ecosystem layer | Sea asset | MariBank integration (verified/flagged) |
|---|---|---|
| **Commerce** | **Shopee** (marketplace, SEA's largest) | Buyer-side: savings cross-sell on idle cash (pattern verified; detail flagged). Merchant-side: **Mari Business Account / Business FD** for sellers (verified — product exists; channel mechanics flagged) |
| **Payments / wallet** | **ShopeePay** (under Monee/ex-SeaMoney) | Wallet↔bank transfers; shared Sea fintech platform (pattern verified; API detail flagged) |
| **Credit** | SPayLater, SeaBank lending | Regional credit stack; MariBank loan products emerging 2025–26 (**flagged**) |
| **Entertainment** | **Garena** (Free Fire) | Brand distribution / promotions (thin banking link — flagged) |
| **Regional banks** | SeaBank Indonesia; **MariBank Philippines** (ex-SeaBank, DFB upgrade Jul 2026) | Regional banking group under MariBank brand (verified — BT, Fintech News SG, [tonik_digital_bank_guide.md](tonik_digital_bank_guide.md)) |
| **Parent** | **Sea Limited** (NYSE: SE) | Capital, data platform, talent; S$173M injection (DealStreetAsia, 2023) |

---

## 5. The Technology

### 5.1 Architecture Posture: Cloud-Native, API-First (Consensus — Flagged)

MariBank is a **greenfield, born-in-the-cloud digital bank** — this is the repo's consensus across the digital-bank series and is consistent with every public description of Sea's fintech build ([core_banking_systems_guide.md](core_banking_systems_guide.md); [chinese_bank_core_systems_guide.md](chinese_bank_core_systems_guide.md) classifies "GXS (Grab+Singtel) and MariBank (Sea) on cloud-native builds"; [ocbc_software_systems_guide.md](ocbc_software_systems_guide.md) lists "GXS / MariBank — Vendor/cloud-native cores (⚠)"). **No vendor stack is publicly confirmed, so everything below the licence layer is architecture-inference, flagged.**

What the architecture *must* look like for a DFB licence with Sea's constraints:

- **Cloud-native core banking** — a modern SaaS or cloud-native core (the Mambu / Thought Machine / Tuum / 10x class), or a Sea-engineered core on public cloud. The distinguishing design: accounts, ledgers and interest as configurable services rather than monolithic COBOL-era modules (contrast the Temenos/FLEXCUBE-class incumbent stacks — see [temenos_guide.md](temenos_guide.md), [oracle_flexcube_data_model_guide.md](oracle_flexcube_data_model_guide.md)). **The specific vendor is unverified — do not assume Temenos** (the brief asked to check "Temenos-adjacent"; nothing found ties MariBank to Temenos, and Temenos' main SG digital-bank presence is GLDB-on-Temenos per the repo — see [green_link_digital_bank_guide.md](green_link_digital_bank_guide.md)).
- **Microservices + event-driven integration** with the Sea platform — wallet transactions, merchant data, marketing — the standard pattern for ecosystem banks (same class as Trust's Google Cloud/Mambu build, [trust_bank_guide.md](trust_bank_guide.md) §3, and the architecture canon in [core_banking_systems_guide.md](core_banking_systems_guide.md)).
- **High-throughput, low-friction deposit engine** — daily interest accrual, instant PayNow/FAST, near-real-time ledger posting; the deposit product's economics depend on automation (no branches, thin ops).
- **Public-cloud hosting** — most likely on one of the big three hyperscalers; Sea has a long AWS/GCP relationship history, but **no verified MariBank-specific cloud contract** (the data-centre angle is covered in [../technology/singapore_data_centres_guide.md](../technology/singapore_data_centres_guide.md)). MAS's cloud guidance (outsourcing notices) applies.

**What "cloud-native" buys MariBank specifically (the architect's read).** Three properties matter for this bank's business model:

1. **Cost-to-serve.** No branches, no ATMs, thin ops — the marginal cost of opening Nurul's account (see [§8](#8-the-worked-example-a-sea-ecosystem-customer-journey)) is a few cents of compute. The 2.5–2.88% launch rates were *sustainable for a new entrant* only because the operating-cost base is a fraction of an incumbent's — the same logic the repo documents for Trust ([trust_bank_guide.md](trust_bank_guide.md) §3.10).
2. **Time-to-market for products.** MariBank shipped savings → debit card → two invest products → business account → cross-border transfers within ~30 months ([§3](#3-the-products)). That cadence is a microservices/API property: each product is a configuration on the platform, not a multi-year build. This is the direct architectural contrast with the incumbent cores ([temenos_guide.md](temenos_guide.md), [oracle_flexcube_data_model_guide.md](oracle_flexcube_data_model_guide.md)) that the repo's core umbrella ([core_banking_systems_guide.md](core_banking_systems_guide.md)) frames as build-vs-buy-vs-BaaS.
3. **Ecosystem integration.** The Shopee/ShopeePay cross-sell and the regional "one brand, many licences" group (SG + PH) require APIs, not ETL batches — event-driven integration is the enabler of the ecosystem model ([§4](#4-the-ecosystem)).

**A reference stack (illustrative only — flagged).** A MariBank-class build would typically look like: SaaS or containerised core (accounts/ledger/interest) → API gateway → microservices for onboarding (Singpass/Myinfo), payments (PayNow/FAST), funds distribution (Mari Invest), and compliance (AML/KYC screening) → event bus to the Sea platform (wallet, merchant, marketing) → data lake for analytics and MAS reporting. **This is a pattern description, not a verified inventory** — no MariBank-specific stack diagram is public.

### 5.2 The Core Banking System: Vendor Unverified (Flagged)

This is the honest state of the evidence: **MariBank has not publicly disclosed its core-banking vendor.** Searches for "MariBank Thought Machine / Mambu / Temenos / Finxact / Tuum" return no verified contract or case study (unlike Trust Bank, where Google Cloud is confirmed by a Google case study and Mambu is vendor/industry-attested — [trust_bank_guide.md](trust_bank_guide.md) §3.2). The repo's existing guides therefore mark MariBank's core as "vendor/cloud-native cores (⚠)" ([ocbc_software_systems_guide.md](ocbc_software_systems_guide.md)).

Two plausible (unverified) build paths:

1. **A modern vendor core** (Mambu / Thought Machine Vault / Tuum class) configured for the simple savings-ledger product — the cheapest route to a DFB go-live in 2022–23, consistent with the rest of the SG cohort.
2. **Sea-engineered core on the SeaMoney platform** — Sea already runs payments and credit at scale across SEA; a Singapore core could be a hardened, MAS-ready version of the group platform rather than a bought-in system.

Either way the *product* evidence supports: a cloud-native ledger with **daily interest**, **instant payments**, and **API-first integration** — that is the observable behaviour. **Flag this section as inference where not cited.**

### 5.3 Identity, Payments, and Compliance

- **Onboarding / KYC:** Singpass (national digital identity) + Myinfo data pull for instant account opening — the standard SG digital-bank pattern (verified via maribank.sg help centre: "Open a Mari Savings Account digitally with Singpass"). Foreigners with valid passes supported. The identity infrastructure behind this is covered in [../technology/distributed_auth_guide.md](../technology/distributed_auth_guide.md).
- **Payments:** PayNow (instant peer transfers by mobile number/NRIC) and FAST (interbank) in/out; **overseas transfers** (first among SG digital banks, ST 2025) — cross-border rails layered on top.
- **Deposit insurance:** SDIC integration (S$100,000 cap since Apr 2024) — the trust wrapper that lets a branchless bank take retail deposits.
- **AML/CFT and MAS compliance:** standard DFB obligations — transaction monitoring, name screening, MAS 644/outsourcing notices, and the Technology Risk Management (TRM) framework. MariBank, like all digital banks, is subject to MAS's full prudential regime; its small, automated operation makes the tech-risk bar a first-class architecture input.
- **The 2022–23 launch constraint:** MAS's "uberisation" concern (platform data flows) means the Shopee↔MariBank data sharing has to respect PDPA and MAS data rules — an architectural boundary most likely implemented as explicit consent + data minimisation in the API layer (pattern; detail flagged).

**Resilience and MAS expectations.** As a DFB, MariBank operates under the same Technology Risk Management (TRM) expectations as incumbent banks — MAS notices on outsourcing, cyber hygiene, business continuity and incident reporting apply in full, and the *absence of branches* raises the bar on app/API availability: the app is the branch. The repo's SG-bank guides ([dbs_bank_guide.md](dbs_bank_guide.md), [ocbc_software_systems_guide.md](ocbc_software_systems_guide.md)) document the incumbent bar; the digital banks must meet the same bar with a fraction of the engineering headcount, which is precisely why the cloud-native, managed-service pattern (SaaS core, hyperscaler resilience, PaaS security) is the only realistic build — a point the core umbrella makes for the whole cohort ([core_banking_systems_guide.md](core_banking_systems_guide.md)). MariBank's regional group also implies cross-border data flows (SG↔PH) that must stay inside each regulator's data-localisation and PDPA-equivalent rules (pattern — flagged).

### 5.4 Technology Table: Component / Description / Notes

| Component | Description | Notes |
|---|---|---|
| **Core banking** | Cloud-native ledger; savings accounts, daily interest, balances | **Vendor unverified (flagged)** — Mambu/Thought Machine-class or Sea-engineered; not Temenos (nothing ties MariBank to Temenos — contrast GLDB) |
| **Cloud platform** | Public-cloud hosting (hyperscaler) | Specific contract **unverified**; infra context: [../technology/singapore_data_centres_guide.md](../technology/singapore_data_centres_guide.md) |
| **Integration layer** | API-first, event-driven microservices to Sea platform (ShopeePay, merchant data) | Pattern inferred (flagged); consent/data-minimisation boundary per PDPA/MAS |
| **Identity / KYC** | Singpass + Myinfo onboarding; foreigners via manual flow | **Verified** (maribank.sg help centre); see [../technology/distributed_auth_guide.md](../technology/distributed_auth_guide.md) |
| **Payments** | PayNow / FAST domestic; overseas transfers (first among SG digital banks) | Domestic rails standard; overseas transfer **verified** (ST 2025) |
| **Deposit insurance** | SDIC coverage to S$100,000 | **Verified** (app-store listing, SDIC scheme) |
| **Wealth platform** | Mari Invest: fund distribution (Fullerton → Lion Global Investors), S$1 entry, instant cash-out | **Verified** (SBR, fintechnews.sg); the fund-adjacent engine is not bank-core |
| **Compliance / risk** | AML/CFT, TRM, MAS 644-style controls | Standard DFB obligations (inferred from licence class) |

### 5.5 Architecture Risks

- **Single-product dependence:** savings deposits are a commodity; the rate war ([§7](#7-the-competition)) compresses the very margin the model runs on. The wealth (Mari Invest) and SME (Business Account) expansion is the de-risking move.
- **Platform dependency:** MariBank's distribution rides Shopee/Monee. If Sea's ecosystem economics weaken (as they did in the 2022 cost-cutting cycle), bank growth funding is at risk.
- **Unproven core vendor (if vendor-bought):** a SaaS core is fast to launch but the differentiation must come from the integration layer — where Sea's actual engineering lives.
- **Data-boundary risk:** ecosystem data-sharing is the value engine and the regulatory risk surface (PDPA, MAS expectations); get it wrong and it is both a fine and a trust event.
- **Concentration in one rate product:** when the SGD rate cycle turns, deposit flows reverse — every SG digital bank faces this; MariBank's daily-interest, no-hoop product is the most price-elastic of the cohort.

---
## 6. The Progress

### 6.1 Deposits and Customers: The Numbers Problem (Flagged)

The single biggest verification gap in this guide is **MariBank's headline metrics**. The honest state of the evidence:

- **MariBank does not publish standalone SG deposits or customer counts** in any source this pass could verify. Sea's quarterly earnings do not break out MariBank Singapore (they aggregate SeaMoney/Monee group figures).
- **Press-reported figures conflict and are unreliable.** A December 2025 blog (Maxthon) claimed "MariBank: 250,000+ accounts" — but the same piece claimed Trust Bank at "300,000+", which contradicts Trust's own verified disclosures (**1 million+ customers as of February 2025**, per BT/ST/Trust — see [trust_bank_guide.md](trust_bank_guide.md) §1.1). A source that gets the market leader wrong by 3× cannot be trusted for MariBank. **Do not quote the Maxthon numbers.**
- **What is directionally true (flag as inference):** MariBank's no-hoop 2.5–2.88% rates in 2023 attracted meaningful deposit inflow during the SG rate war (this is why GXS and MariBank both hit their SDIC caps and raised them to S$75,000 in July 2023 — ST). By the cohort's own positioning, MariBank is the **third consumer digital bank by scale**, behind Trust (1M+ customers) and ahead-or-level with GXS depending on the metric and date — but every specific number here is unverified.
- **The customer-count hierarchy the repo does verify:** Trust Bank **1M+** (Feb 2025, largest digital bank in SG; ~4th largest retail bank by customers); GXS and MariBank have not published comparable verified counts in this pass.

**Recommendation for readers:** treat MariBank deposit/customer figures as **not publicly established**; use the verified proxies in [6.2](#62-what-is-verified-capital-aum-awards) (capital, AUM, awards) and the competition context in [§7](#7-the-competition) instead.

**Why the numbers are missing (the disclosure gap, compared).** The three consumer digital banks sit on a spectrum of transparency:

- **Trust Bank** — most transparent: files entity-level financials (revenue S$97M FY2024; net loss ~S$123.75M FY2024 per aggregator, flagged — [trust_bank_guide.md](trust_bank_guide.md) §6), publishes customer milestones (411K end-2022 → 974K end-2024 → 1M+ Feb 2025).
- **GXS Bank** — mid: no standalone financials, but parents disclose via earnings calls and press (S$358M raised; S$1.5B commitment; profit-by-March-2027 target; deposit-cap changes to S$75K/S$95K).
- **MariBank** — least: no standalone SG financials, no customer/deposit milestones, no profitability timeline. Public signals are limited to capital injections, awards, product launches, and the Mari Invest AUM figure.

That asymmetry matters for analysts and architects: **MariBank's strategic story is readable (ecosystem, products, regional group), but its operating story is not** — you cannot independently verify growth, funding costs, or credit quality from public data as of August 2026. This is flagged as a genuine limitation of this guide, not a gap in effort.

### 6.2 What Is Verified: Capital, AUM, Awards

These are the progress signals that *are* on the record:

- **Parent capital:** S$173 million injection from Sea (DealStreetAsia, 2023, on the March-2023 launch) — evidence Sea funds the bank from group cash rather than external investors (contrast GXS's S$358M raised across rounds and S$1.5B parent commitments — fintechnews.sg; The Digital Banker).
- **Mari Invest AUM:** SavePlus **surpassed S$953.3 million AUM** since its September 2023 introduction (Singapore Business Review, at the Mari Invest Income launch) — a genuinely large number for an SG robo/cash-fund shelf and the strongest single verified growth data point for the Mari franchise.
- **Awards:** "New Virtual Bank of the Year (Singapore)" — Asian Banking & Finance Retail Banking Awards 2024; "Best Digital Bank for Integrated Investing Options" — 2025 (maribank.sg).
- **Product breadth:** the fastest product cadence of the three consumer digital banks in 2024–26 — savings → debit card → Mari Invest (2 products) → business account + business FD → overseas transfers → credit/loan products — per [§3](#3-the-products) (some items flagged).
- **Regional expansion:** MariBank Philippines (ex-SeaBank) upgraded to a digital bank licence in July 2026 ([tonik_digital_bank_guide.md](tonik_digital_bank_guide.md) — the Philippine cohort's 7th digital bank, with ₱68.2B assets at end-2025), formalising the "regional banking group" announced in 2025.

### 6.3 The Profitability Question (Flagged)

- **No verified claim of profitability.** MariBank has not announced break-even; the sector consensus (CNA Feb-2025 landscape piece; The Diplomat Jun-2024 "slow start" piece) is that all three SG consumer digital banks remain loss-making in their early years — heavy marketing/rate costs, thin lending books.
- **Lending is the missing engine.** MariBank's SG balance sheet is deposit-heavy and credit-light (its credit products only emerged 2025–26, flagged). Net interest income from a high-rate savings book without a matching loan book is structurally thin — the same unit-economics problem the repo documents for Trust and GXS ([trust_bank_guide.md](trust_bank_guide.md) §6.3).
- **Path to profitability (inference):** (1) Mari Invest fee income — AUM-scale fees are the margin that deposits alone can't produce; (2) SME business banking (Business Account + FD, and eventually SME credit to Shopee merchants — Sea's SPayLater know-how); (3) group-level cost sharing — one Sea fintech platform serves five+ licences across SEA, so MariBank SG's marginal cost structure is low by construction.
- **Contrast:** GXS has a public target — **profit by March 2027**, via doubling its loan book every six months (The Digital Banker / FMT, July 2024). MariBank has made no equivalent public commitment (flagged).

### 6.4 Progress Table

| Metric | Figure | Status |
|---|---|---|
| SG deposits | **Not disclosed** | **Flagged** — do not quote third-party figures (they conflict) |
| SG customers | **Not disclosed** | **Flagged** — same caveat; Trust's 1M+ (Feb 2025) is the verified reference point |
| Parent capital injection | **S$173M** (Sea, 2023) | Verified — DealStreetAsia |
| Mari Invest SavePlus AUM | **> S$953.3M** (since Sep 2023) | Verified — Singapore Business Review |
| Awards | Virtual Bank of the Year (2024); Best Digital Bank for Integrated Investing (2025) | Verified — maribank.sg |
| Regional | MariBank PH licence upgrade (Jul 2026); ₱68.2B assets end-2025 | Verified — tonik guide / BSP |
| Profitability | **No claim; in investment mode** | **Flagged** — GXS targets profit by Mar 2027; MariBank has no public target |

---

## 7. The Competition

### 7.1 The Singapore Digital Bank Landscape

Singapore has **five digital banks** (per CNA Feb-2025 landscape piece):

- **Retail (consumer):** **Trust Bank** (Standard Chartered + FairPrice Group — full bank licence, launched Sep 2022), **GXS Bank** (Grab + Singtel — DFB, launched Aug 2022), **MariBank** (Sea — DFB, launched Mar 2023).
- **Wholesale (business):** **ANEXT Bank** (Ant Group — DWB), **Green Link Digital Bank / GLDB** (Linklogis-led — DWB). See [green_link_digital_bank_guide.md](green_link_digital_bank_guide.md) for GLDB (including the recurring "four winners" list error) and the repo's broader digital-bank framing in [core_banking_systems_guide.md](core_banking_systems_guide.md) and [full_stack_banking_guide.md](full_stack_banking_guide.md).

The consumer trio is the fight MariBank is in. Their verified differentiators:

- **Trust Bank** — "The bank that gives you more": rewards banking wired to the NTUC/FairPrice ecosystem (Linkpoints), the largest customer base (1M+ by Feb 2025), a *full* bank licence (not a DFB framework licence), StanChart BaaS plumbing, Google Cloud + (per vendor/industry coverage) Mambu. Cross-ref [trust_bank_guide.md](trust_bank_guide.md).
- **GXS Bank** — the **Grab–Singtel** digital bank: integrated with the Grab superapp (rides, food delivery, GrabPay) and Singtel; savings account (deposit cap now S$95,000 per account, 2026) plus **FlexiLoan** consumer credit; ~S$358M raised, **S$1.5B** parent capital committed through March 2027; public target of **profit by March 2027** (The Digital Banker/FMT). GXS is the bank this guide's brief *thought* MariBank was — the correction in [1.2](#12-critical-correction-maribank-is-sea-groups-bank-not-grab-singtels) is the key takeaway.
- **MariBank** — the **Sea/Shopee** digital bank: high-yield no-hoop savings (rate leader at launch: 2.5% → 2.88% in 2023), Mari Invest wealth shelf (S$953M+ AUM), SME business banking, first-to-market overseas transfers, regional group (SG + PH).

### 7.2 Trust vs GXS vs MariBank: The Consumer Face-Off

- **On rate:** MariBank led the 2023 rate war (2.88%); Trust and GXS matched with tiered structures. By 2025–26 all three have normalised into a 2.0–2.9% band with promo tiers (aggregator data, flagged).
- **On ecosystem:** Trust = grocery/union retail (FairPrice Linkpoints); GXS = transport/food/telco (Grab + Singtel); MariBank = e-commerce/payments (Shopee + Monee). Each bank's ecosystem is its distribution moat and its data moat.
- **On product breadth:** Trust leads on cards/rewards/insurance; GXS leads on consumer credit (FlexiLoan) with a profit date; MariBank leads on wealth (Mari Invest) and SME onboarding plus cross-border transfers.
- **On licence nuance:** Trust holds a full bank licence (unrestricted class — can, e.g., operate branches); GXS and MariBank hold DFB framework licences (phased-in S$1.5B capital; restrictions like no ATMs/branches in the early years — the framework requires a "commence within 18 months... digital-only" posture; exact restriction detail in [full_stack_banking_guide.md](full_stack_banking_guide.md)).
- **On financial disclosure:** Trust files entity-level financials (revenue S$97M in 2024, still loss-making — [trust_bank_guide.md](trust_bank_guide.md)); GXS discloses via Grab/Singtel earnings and press (S$1.5B commitment, profit-by-2027); MariBank discloses least (flagged).

**The 2022–2026 competitive arc.** Three phases define the fight:

1. **2022 — the opening:** Trust (Sep 2022) and GXS (Aug 2022) launched within weeks of each other; MariBank was still in beta. Trust's FairPrice distribution and GXS's Grab integration set the two ecosystem templates.
2. **2023 — the rate war:** MariBank's March 2023 public launch at 2.5% escalated to **2.88% by mid-2023** (ST); GXS matched with its own promos and both banks raised deposit caps to S$75,000 in July 2023 (ST). Deposit caps were the physical constraint: SDIC-insured retail deposits could not exceed the cap without breaking the insurance wrapper, so the banks rationed access (waitlists, invite-only) while the cap was still S$75,000.
3. **2024–2026 — normalisation and expansion:** rates fell from the 2.8%+ peaks as the SGD cycle turned; the trio differentiated into cards/rewards (Trust), consumer credit (GXS FlexiLoan), and wealth/SME/cross-border (MariBank). The SDIC cap rise to S$100,000 (Apr 2024) gave all three headroom; GXS raised its account cap to S$95,000 (2026). The CNA Feb-2025 landscape piece ("Would you put your life savings in a digital bank?") captured the maturing trust question — deposit insurance, app reliability, and the still-open profitability question (see [6.3](#63-the-profitability-question-flagged)).

**The market context (flagged).** Singapore's retail deposit market remains dominated by the incumbents (DBS, OCBC, UOB — see [dbs_bank_guide.md](dbs_bank_guide.md), [ocbc_software_systems_guide.md](ocbc_software_systems_guide.md), [uob_software_systems_guide.md](uob_software_systems_guide.md)); the five digital banks hold a small single-digit share of retail deposits even by 2026, and MAS's own stance (no new digital-bank licences being granted — mas.gov.sg) freezes the competitive set at five. The digital banks compete less with each other than with the incumbents' own digital pushes (DBS digibank, OCBC OneAccount, UOB One) — the rate war of 2023 was the exception that forced incumbents to defend rates. **Specific deposit-share figures were not verified in this pass — flagged.**

### 7.3 Competition Table: Bank / Focus / Notes

| Bank | Focus | Notes |
|---|---|---|
| **Trust Bank** | Rewards mass-market retail (FairPrice/NTUC Linkpoints) | StanChart + FairPrice Group; **full bank licence** (not DFB); launched 1 Sep 2022; **1M+ customers** (Feb 2025) — largest digital bank in SG; Google Cloud + Mambu (flagged); loss-making but scaling (S$97M revenue 2024); 2026 StanChart portfolio transfer (see [trust_bank_guide.md](trust_bank_guide.md)) |
| **GXS Bank** | Grab/Singtel ecosystem retail + SME credit | **Grab 60% / Singtel 40%**; DFB; launched Aug 2022; savings (cap S$95,000, 2026) + FlexiLoan; S$1.5B committed through Mar 2027; **profit target Mar 2027**; the bank the MariBank brief confused with MariBank |
| **MariBank** | Sea/Shopee ecosystem: high-yield savings, wealth, SME | **Sea Limited**; DFB; launched Mar 2023 (invite-only) at 2.5% p.a.; rate leader 2023 (2.88%); Mari Invest AUM > S$953M; Mari Business Account; first SG digital bank with overseas transfers; regional group (SG + PH); **least financial disclosure (flagged)** |
| **ANEXT Bank** | SME/wholesale (Ant Group) | DWB; Ant's distributed stack (OceanBase lineage) — cross-ref [chinese_bank_core_systems_guide.md](chinese_bank_core_systems_guide.md) |
| **GLDB** | SME trade finance (Linklogis-led) | DWB; Temenos-on-Huawei-Cloud per repo — cross-ref [green_link_digital_bank_guide.md](green_link_digital_bank_guide.md) |

### 7.4 MariBank's Competitive Position

**Strengths:** (1) the strongest parent balance sheet of the trio — Sea funds MariBank from group cash, no external capital rounds needed; (2) the deepest e-commerce distribution (Shopee's merchant + buyer base dwarfs Grab's SG user base in raw commerce volume); (3) the wealth wedge — Mari Invest's S$953M AUM is the cohort's most credible non-deposit revenue line; (4) SME + cross-border first-moves that the consumer trio is only now copying.

**Weaknesses:** (1) the least transparent — no entity financials, no customer/deposit disclosures, no public profitability plan; (2) credit-lag — FlexiLoan-style lending arrived late in SG (2025–26, flagged), leaving the balance sheet deposit-heavy; (3) brand — "MariBank" the bank is less known than "Shopee" the app, and the Monee/SeaMoney→Mari rebrand churn adds confusion; (4) rate elasticity — a no-hoop savings product is the most price-sensitive of the cohort, so deposit flight risk is highest when rates fall.

**The strategic read:** MariBank is playing the *regional* game — Singapore is the regulated flagship (MAS DFB = the group's credibility asset), while the economics scale across the MariBank PH + SeaBank ID network. That is a fundamentally different posture from Trust (SG-only rewards bank) and GXS (Grab-ecosystem SG bank with a profit date).

---

## 8. The Worked Example: A Sea-Ecosystem Customer Journey

### 8.1 Scenario (Corrected): The Shopee Ecosystem User

**The brief asked for a "Grab user" journey. Corrected per [1.2](#12-critical-correction-maribank-is-sea-groups-bank-not-grab-singtels): the MariBank customer is a Sea/Shopee user.** Meet **Nurul**, 34, a Singapore-based part-time reseller who sells on **Shopee** (a small cosmetics storefront) and shops on Shopee for household goods. She already uses **ShopeePay** for checkout and has seen MariBank banners inside the Shopee app ("earn up to 2.88% p.a. on your savings, no minimum balance").

### 8.2 The Journey Flow

**Step 1 — Discovery inside the ecosystem (verified pattern).** Nurul sees the MariBank offer in the Shopee app (Sea cross-sell; the exact placement is the app's marketing surface, but ecosystem cross-sell is the documented distribution model — BT/Fintech News SG). The hook: her ShopeePay balance sits idle between purchases; a savings account paying daily interest beats a wallet.

**Step 2 — Digital onboarding (verified mechanics).** She downloads the MariBank app (Play Store id `sg.com.maribankmobile.digitalbank`), opens a **Mari Savings Account** with her SG mobile number + **Singpass** (Myinfo auto-fills her particulars — verified via maribank.sg help centre). Foreigners use the manual flow. No minimum deposit, no branch visit, no salary-credit condition. Account live in minutes.

**Step 3 — The deposit decision (verified product).** She transfers S$10,000 from her DBS account via **PayNow/FAST**. The account earns interest **daily** at the prevailing rate — 2.5% at launch-era, up to 2.88% in mid-2023, ~2.28% flat / 0.88%+bonus promo by 2026 (rates flagged, [3.1](#31-mari-savings-account)). Deposits are **SDIC-insured** (S$100,000 cap since Apr 2024) — the trust anchor for a branchless bank.

**Step 4 — Everyday spending (verified product).** Her **Mari Debit Card** arrives automatically (auto-issued with the account — Growbeansprout): no annual fee, no minimum spend; she pays for groceries and Shopee orders straight from the savings balance. (Card specifics flagged — the SG Mari card detail is thin in this pass.)

**Step 5 — Wealth step-up (verified product).** With the balance sitting idle, Nurul moves S$3,000 into **Mari Invest — SavePlus** (launched Sep 2023; fund shelf via Fullerton → Lion Global Investors; invest from **S$1**, no transaction fees; instant cash-out with S$10,000/day limit — verified via SBR/fintechnews.sg). She can now park *earning cash* (fund) separately from *spending cash* (savings) in one app — the "integrated investing" positioning that won the 2025 award.

**Step 6 — The merchant upgrade (verified product).** Nurul's reselling grows; she incorporates a small company. She opens a **Mari Business Account** in the app (SG-incorporated businesses, app-only onboarding — Airwallex SG 2026) and places working capital in a **Mari Business Fixed Deposit**. She uses **Overseas Transfers** to pay her Malaysia-based supplier — the service MariBank was the **first SG digital bank** to offer (ST 2025).

**Step 7 — The regional option (verified 2026 event).** Her Philippine supplier's sister firm could bank with **MariBank Philippines** (ex-SeaBank, digital-bank licence July 2026) — one Mari brand across the corridor (tonik guide).

**The journey at a glance:**

| Step | Action | Product | Verified? |
|---|---|---|---|
| 1 | Sees savings offer inside Shopee app | Ecosystem cross-sell | Pattern verified (BT/Fintech News SG); exact placement flagged |
| 2 | Opens account with Singpass + mobile number | Mari Savings Account | Verified (maribank.sg help centre) |
| 3 | Transfers S$10,000 via PayNow/FAST | Payments rails | Verified (standard DFB rails) |
| 4 | Spends on auto-issued card | Mari Debit Card | Verified (Growbeansprout); card T&Cs flagged |
| 5 | Moves S$3,000 into cash fund | Mari Invest SavePlus | Verified (SBR; S$1 entry, no fees) |
| 6 | Incorporates; opens business account + FD; pays supplier abroad | Mari Business Account / Business FD / Overseas Transfers | Verified (Airwallex SG 2026; ST 2025) |
| 7 | Regional option for supplier | MariBank Philippines | Verified (tonik guide; Jul 2026 upgrade) |

**What the journey deliberately excludes (flagged):** credit. Nurul takes no loan in this flow — MariBank SG's credit products emerged only in 2025–26 and are the least-documented part of the shelf (see [3.3](#33-products-table-product--type--notes)). A complete "life-cycle" journey would add a step 8 (credit) once the SG loan products are verifiably live; GXS's FlexiLoan is the benchmark for what that step looks like in the Grab ecosystem.

### 8.3 Lessons From the Journey

1. **The ecosystem is the acquisition channel.** Nurul never "researched banks" — she was converted inside Shopee. The bank's marketing budget substitutes ecosystem distribution for branch network; that is the whole digital-bank thesis.
2. **Simple products compound into a relationship.** One no-hoop savings account became: savings + debit card + fund investment + business account + cross-border transfers — five products from one onboarding. The daily-interest, no-minimum design is the "landing" product; wealth and SME are the "expand" products.
3. **Trust is engineered, not marketed.** Singpass onboarding (state-issued identity), SDIC insurance (state-backed deposits), PayNow/FAST rails (state-run payments) — MariBank rides Singapore's public digital infrastructure; its own trust contribution is transparency and reliability.
4. **Unit economics favour the platform owner.** MariBank SG's marginal cost of serving Nurul is near-zero (cloud-native, API-first, no branches — [§5](#5-the-technology)); Sea's group platform amortises KYC, payments and data across five+ regional licences.
5. **The gap is credit.** The journey shows no lending step — MariBank SG's loan products arrived late (2025–26, flagged) and are not yet ecosystem-woven the way GXS FlexiLoan is Grab-woven. Credit is the missing chapter of the Nurul journey (and the missing engine of profitability, [6.3](#63-the-profitability-question-flagged)).
6. **The correction bites.** A "Grab user" journey would belong to GXS (Grab rides → GXS savings → FlexiLoan); the Shopee-user journey is MariBank's. Getting the parent right is not trivia — it determines which ecosystem, which data, and which products the bank actually has.

---

## 9. The Summary: One Page — "The Superapp's Bank"

**MariBank is Sea Limited's Singapore digital full bank** — one of two digital full bank licences MAS awarded on **4 December 2020** (with GXS Bank), publicly launched **March 2023** after a 2022 beta, and the banking anchor of Sea's regional **"Mari" group** (Singapore + Philippines, with SeaBank Indonesia adjacent). It is **not** the Grab–Singtel bank — that is GXS Bank — and this guide's central correction is that the two December-2020 siblings are routinely conflated: **MariBank = Shopee/SeaMoney's bank; GXS = Grab/Singtel's bank.**

**What it sells:** the **Mari Savings Account** (2.5% at launch, 2.88% peak, SDIC-insured, no minimums, daily interest) as the landing product; **Mari Invest** (SavePlus cash fund since Sep 2023, **>S$953M AUM**, plus Mari Invest Income) as the wealth step-up; **Mari Business Account + Business Fixed Deposit** for Shopee's merchant base; and **overseas transfers** — the first among SG digital banks. Credit and insurance (Mari Insurance) are signalled but their SG specifics are **flagged unverified**.

**How it runs:** a **cloud-native, API-first** build on the Sea platform — the specific core vendor is **unverified** (flagged), the architecture is inferred from product behaviour (daily interest, instant payments, Singpass onboarding, SDIC integration, ecosystem cross-sell).

**Where it stands:** the least-disclosing of the three consumer digital banks (no entity financials, no customer/deposit numbers — **flagged**), loss-making and in investment mode like its peers, but with the cohort's strongest wealth metric (S$953M AUM) and the deepest e-commerce distribution (Shopee). Competition: **Trust** (1M+ customers, rewards banking, full licence — the market leader) and **GXS** (Grab+Singtel, S$1.5B committed, profit target March 2027).

**The final word — the superapp's bank.** MariBank is the purest expression of the "superapp bank" thesis in Singapore: not a bank that built an app, but an app-empire that built a bank. Where Trust is a bank wired into a supermarket ecosystem and GXS is a bank wired into a mobility ecosystem, MariBank is the banking layer of Southeast Asia's commerce machine — deposits that pay daily interest on the idle cash of e-commerce, wealth that starts at S$1, business accounts that open for marketplace sellers, and a licence that turns Sea's regional data and distribution into a regulated balance sheet. Whether it becomes the region's most valuable digital bank depends on one thing it has not yet proven: turning deposits and AUM into a credit engine — and telling the market the numbers. Until then, it remains **the superapp's bank** — the most strategically important, least transparent of Singapore's digital banks.

---

## 10. Glossary

- **MariBank** — Sea Limited's Singapore digital bank (MariBank Singapore Pte. Ltd.); MAS digital full bank licence (Dec 2020); launched publicly March 2023. Not owned by Grab/Singtel.
- **Grab** — Singapore superapp company (ride-hailing, food delivery, payments); **60% owner of GXS Bank** (with Singtel), not of MariBank.
- **Singtel** — Singapore's largest telecom; **40% owner of GXS Bank** (with Grab); also a Sea competitor-adjacent telco; not a MariBank owner.
- **MAS** — Monetary Authority of Singapore; central bank and financial regulator; awarded the digital bank licences (Dec 2020) and supervises MariBank under the Banking Act and digital bank framework.
- **Digital bank** — a bank with no (or minimal) physical branch network, operating primarily through mobile/online channels; Singapore has five: Trust, GXS, MariBank, ANEXT, GLDB.
- **Digital full bank (DFB)** — the digital-bank-framework licence class allowing retail deposits (S$1.5B phased-in base capital); held by GXS Bank and MariBank. (Distinct from Trust's unrestricted full bank licence.)
- **Licence** — the MAS-granted authorisation to carry on banking business; MariBank's DFB licence derives from the 4 December 2020 award to Sea Limited.
- **Mari Savings Account** — MariBank's flagship SGD savings account: daily interest, no minimum balance, no salary-credit/spend conditions, SDIC-insured; launched March 2023 at 2.5% p.a.
- **Mari Invest** — MariBank's wealth arm: SavePlus cash-management fund (Sep 2023; >S$953M AUM) and Mari Invest Income (global-bond monthly income, 2025); invest from S$1, no transaction fees.
- **Mari Insurance** — the insurance arm of the Mari brand family; advertised but with unverified SG product specifics in this pass (flagged).
- **Superapp** — an app that bundles many daily services (rides, food, shopping, payments); Grab is the canonical SG superapp; Sea is a "portfolio" superapp-group (Shopee + Garena + Monee) — and MariBank is its banking layer.
- **Ecosystem** — the network of products/platforms a bank draws distribution and data from; MariBank's ecosystem is Sea's (Shopee, ShopeePay, Garena, Monee), not Grab's.
- **Cloud-native** — software designed for public-cloud deployment (microservices, containers, API-first, elastic scale); the architecture class of all the SG digital banks, MariBank included (vendor specifics flagged).
- **Core (core banking system)** — the ledger/accounting engine of a bank (accounts, deposits, interest, balances); MariBank's vendor is **unverified** (flagged); contrast Temenos/FLEXCUBE-class incumbent cores.
- **Deposits** — customer funds held by the bank; MariBank's SG deposits are **not publicly disclosed** (flagged); SDIC-insured to S$100,000.
- **Customers** — retail/business account holders; MariBank's SG count is **not publicly disclosed** (flagged); Trust's verified 1M+ (Feb 2025) is the market reference.
- **Trust (Trust Bank)** — the Standard Chartered + FairPrice Group digital bank; full bank licence; launched Sep 2022; **largest SG digital bank by customers** (1M+).
- **GXS (GXS Bank)** — the **Grab–Singtel** digital bank (Grab 60% / Singtel 40%); DFB; launched Aug 2022; savings + FlexiLoan; S$1.5B committed; profit target March 2027. The bank MariBank is most often confused with.
- **Competition** — the SG digital-bank battle: Trust (rewards/ecosystem), GXS (Grab ecosystem/credit), MariBank (Shopee ecosystem/wealth+SME), plus wholesale ANEXT and GLDB.
- **Singapore** — the city-state whose MAS digital bank framework (2019–2020) created the region's most visible digital-bank testbed; MariBank's licensed home market and the flagship of Sea's regional banking group.

---

## 11. References and Cross-References

**Primary / verified sources (this pass):**
1. MAS — "MAS Announces Successful Applicants of Licences to Operate New Digital Banks in Singapore" (media release, 4 Dec 2020) — mas.gov.sg (award of DFBs to Grab–Singtel consortium and Sea; DWBs to GLDB and ANEXT). Full text not re-extracted this pass (extract backend degraded) — award facts corroborated by ST, The Diplomat, CNA, Seedly.
2. MAS — "Digital Bank Licence" (mas.gov.sg/regulation/banking/digital-bank-licence): four licences issued 2020; no new licences currently being granted.
3. Straits Times — "MAS awards digital full bank licences to Grab-Singtel and Sea..." (4 Dec 2020) and "MariBank raises interest rates on savings account to 2.88%..." and "Singapore's two digital banks opening up to new customers, raising deposit caps to $75,000" (2023) — verified via search.
4. DealStreetAsia — "SG digital lender MariBank bags $173m from parent Sea Ltd" (2023): March-2023 invite-only launch confirmed.
5. Jom Media — "Singapore This Week" (17 Mar 2023): Mari Savings Account 2.5% p.a., no minimum deposit.
6. Singapore Business Review — "MariBank unveils Mari Invest Income": SavePlus AUM > S$953.3M since Sep 2023.
7. Fintech News SG — Mari Invest Income/SavePlus coverage (2025); "Sea Opens New HQ in Singapore, Rebrands SeaMoney to Monee" (2025).
8. Straits Times — "Shopee parent Sea opens new financial services HQ in Spore, rebrands SeaMoney to Monee" (2025): MariBank first SG digital bank with overseas transfers; regional banking group.
9. The Digital Banker / FMT — "Digital bank GXS, set up by Grab and Singtel, targets profit by 2027" (Jul 2024): Grab 60% / Singtel 40%; S$1.5B committed; profit target Mar 2027.
10. CNA — "IN FOCUS: Would you put your life savings in a digital bank?" (Feb 2025): five SG digital banks; retail trio Trust/GXS/MariBank.
11. The Diplomat — "Southeast Asia's Digital Banking Race..." (Jun 2024): GXS = Grab+Singtel; MariBank = Sea/Shopee parent.
12. maribank.sg — homepage, product pages (Mari Savings Account), fees-and-rates page (savings/credit cards/loan), help centre (Singpass onboarding), awards, news-and-media.
13. Growbeansprout (Aug 2026) and SingSaver (Dec 2025) — Mari Savings Account rate reviews (aggregator — flagged).
14. The Kopi Notes (Jun 2026) — "What is MariBank? Singapore's Digital Bank Explained": claims Oct-2023 full public launch and 2024–25 SG credit card/loan rollout (**conflicting/flagged**).
15. Airwallex SG blog (2026) — Mari Business Account review (Sea ownership confirmed).

**Repo cross-references (series context):**
- **Sibling digital-bank guides:** [trust_bank_guide.md](trust_bank_guide.md) (the SG digital-bank pattern — full licence nuance, Mambu/Google Cloud, 1M+ customers, 2026 StanChart transfer), [green_link_digital_bank_guide.md](green_link_digital_bank_guide.md) (the Dec-2020 four-winners correction; GLDB; Temenos-on-Huawei), [tonik_digital_bank_guide.md](tonik_digital_bank_guide.md) (Philippine digital-bank cohort incl. MariBank PH, Jul 2026 licence upgrade).
- **SG banks (light cross-ref):** [ocbc_software_systems_guide.md](ocbc_software_systems_guide.md), [uob_software_systems_guide.md](uob_software_systems_guide.md), [dbs_bank_guide.md](dbs_bank_guide.md) (incumbent response to the digital-bank wave).
- **Core / architecture (light cross-ref):** [core_banking_systems_guide.md](core_banking_systems_guide.md) (cloud-native cores, build-vs-buy-vs-BaaS — umbrella for the vendor-flagged discussion), [temenos_guide.md](temenos_guide.md) (NOT MariBank's core — no verified link; contrast GLDB), [universal_banking_model_guide.md](universal_banking_model_guide.md) (model cross-ref), [full_stack_banking_guide.md](full_stack_banking_guide.md) (MAS framework detail; "pilot Mar 2022" for MariBank), [chinese_bank_core_systems_guide.md](chinese_bank_core_systems_guide.md) (GXS/MariBank as cloud-native builds; ANEXT's Ant stack).
- **Technology guides (../technology/ prefix):** [../technology/distributed_auth_guide.md](../technology/distributed_auth_guide.md) (Singpass/Myinfo identity — MariBank onboarding), [../technology/singapore_data_centres_guide.md](../technology/singapore_data_centres_guide.md) (SG infra angle for the cloud build).

**Methodology and tooling note (this pass).** Research used targeted `web_search` only — `web_extract` was degraded to a search-only backend (no URL content retrieval), so primary-source verification relied on search snippets from mas.gov.sg, straitstimes.com, maribank.sg, channelnewsasia.com, dealstreetasia.com, sbr.com.sg, fintechnews.sg, thedigitalbanker.com, thediplomat.com, plus aggregators (SingSaver, Growbeansprout, Seedly, The Kopi Notes, Airwallex SG blog) and the repo's own verified guides. Facts that appear in exactly one secondary source (e.g., the "October 2023 full public launch" claim; the SG credit-card/loan 2024–25 rollout) are **flagged as single-source** and not asserted. No product fact, launch date, or figure was invented; where the record is silent (core vendor, SG deposits/customers, Mari Insurance specifics, profitability), the guide says so explicitly and flags it.

**Verification flags summary:** ownership (corrected — Sea, not Grab/Singtel); licence (verified — MAS DFB, 4 Dec 2020); public launch (verified — Mar 2023 invite-only); soft launch (2022 — date flagged); current rates (aggregator, as-of); deposits/customers (not disclosed — flagged); core vendor (unverified — flagged); Mari Insurance (unverified — flagged); SG credit card/loan timeline (secondary source — flagged); profitability (no public claim — flagged).
