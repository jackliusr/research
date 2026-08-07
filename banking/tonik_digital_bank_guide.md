# Tonik Digital Bank, Inc. (Philippines): A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Crédit Agricole CIB, Singapore
> **Context:** Digital Banking / Banking Architecture — Tonik Deep-Dive, Philippine Digital Banking Landscape, BSP Digital Bank Licence Framework, Lending-First Neobank Model, Cloud-Native Core Banking (Finastra Essence + Google Cloud), AI-Driven Unit Economics
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** August 2026

---

## Table of Contents

1. [Tonik Overview](#1-tonik-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [Verified Status at a Glance](#12-verified-status-at-a-glance)
   - 1.3 [The Founding Story: A Neobank for the Philippines](#13-the-founding-story-a-neobank-for-the-philippines)
   - 1.4 [The Founder: Greg Krasnov](#14-the-founder-greg-krasnov)
   - 1.5 [The Licence: BSP Digital Banking License No. 001](#15-the-licence-bsp-digital-banking-license-no-001)
   - 1.6 [The Launch: February–March 2021](#16-the-launch-februarymarch-2021)
   - 1.7 ["First Digital Bank in the Philippines" — The Claim Verified](#17-first-digital-bank-in-the-philippines--the-claim-verified)
   - 1.8 [Timeline: 2018–2026](#18-timeline-20182026)
   - 1.9 [Current Status (as of August 2026)](#19-current-status-as-of-august-2026)
   - 1.10 [What Could NOT Be Verified](#110-what-could-not-be-verified)
2. [The Business Model](#2-the-business-model)
   - 2.1 [Lending-First, Not User-Platform-First](#21-lending-first-not-user-platform-first)
   - 2.2 [The Target Customer](#22-the-target-customer)
   - 2.3 [The Product Portfolio: Savings](#23-the-product-portfolio-savings)
   - 2.4 [The Product Portfolio: Credit](#24-the-product-portfolio-credit)
   - 2.5 [The Product Portfolio: Cards, Payments, Insurance](#25-the-product-portfolio-cards-payments-insurance)
   - 2.6 [Revenue Model and Unit Economics](#26-revenue-model-and-unit-economics)
   - 2.7 [The Customer Value Proposition](#27-the-customer-value-proposition)
3. [The Technology](#3-the-technology)
   - 3.1 [Architecture Overview: Born in the Cloud](#31-architecture-overview-born-in-the-cloud)
   - 3.2 [Core Banking: Finastra Essence — Not Thought Machine Vault](#32-core-banking-finastra-essence--not-thought-machine-vault)
   - 3.3 [Cloud Platform: Google Cloud](#33-cloud-platform-google-cloud)
   - 3.4 [API-First and the Apigee Gateway](#34-api-first-and-the-apigee-gateway)
   - 3.5 [Digital Onboarding and eKYC](#35-digital-onboarding-and-ekyc)
   - 3.6 [Payments: InstaPay, PESONet and QR Ph](#36-payments-instrapay-pesonet-and-qr-ph)
   - 3.7 [AI Everywhere: Underwriting, Service, Fraud](#37-ai-everywhere-underwriting-service-fraud)
   - 3.8 [Technology Partners](#38-technology-partners)
   - 3.9 [Security and Fraud](#39-security-and-fraud)
   - 3.10 [Architecture Risks](#310-architecture-risks)
4. [Funding and Investors](#4-funding-and-investors)
   - 4.1 [Funding History](#41-funding-history)
   - 4.2 [Total Raised and the Investor List](#42-total-raised-and-the-investor-list)
   - 4.3 [Valuation and the "Unicorn" Question](#43-valuation-and-the-unicorn-question)
5. [Performance and Milestones](#5-performance-and-milestones)
   - 5.1 [Customer Growth](#51-customer-growth)
   - 5.2 [Deposits](#52-deposits)
   - 5.3 [Loans](#53-loans)
   - 5.4 [Revenue and Profitability](#54-revenue-and-profitability)
   - 5.5 [Rankings and Awards](#55-rankings-and-awards)
6. [The Philippine Digital Banking Landscape](#6-the-philippine-digital-banking-landscape)
   - 6.1 [The Philippine Banking Sector](#61-the-philippine-banking-sector)
   - 6.2 [Financial Inclusion: From 70% Unbanked to the 2023 Targets](#62-financial-inclusion-from-70-unbanked-to-the-2023-targets)
   - 6.3 [OFW Remittances: The Macro Backdrop](#63-ofw-remittances-the-macro-backdrop)
   - 6.4 [The BSP Digital Bank Framework](#64-the-bsp-digital-bank-framework)
   - 6.5 [The Licensed Digital Banks](#65-the-licensed-digital-banks)
   - 6.6 [The PH Digital Bank Race: Comparison Table](#66-the-ph-digital-bank-race-comparison-table)
   - 6.7 [Near-Banks: GCash, CIMB, OwnBank and the Rural-Bank Digital Operators](#67-near-banks-gcash-cimb-ownbank-and-the-rural-bank-digital-operators)
   - 6.8 [Challenges](#68-challenges)
7. [The Architect's Perspective](#7-the-architects-perspective)
   - 7.1 [Tonik as an Architecture Case Study](#71-tonik-as-an-architecture-case-study)
   - 7.2 [Cloud-Native Core Lessons](#72-cloud-native-core-lessons)
   - 7.3 [Build vs Buy: The Bought-Core Lesson](#73-build-vs-buy-the-bought-core-lesson)
   - 7.4 [Designing for the Underbanked: Credit-Led Architecture](#74-designing-for-the-underbanked-credit-led-architecture)
   - 7.5 [Architecture Risks and Mitigations](#75-architecture-risks-and-mitigations)
   - 7.6 [The Architect's Checklist](#76-the-architects-checklist)
8. [Future Outlook](#8-future-outlook)
   - 8.1 [Product Expansion](#81-product-expansion)
   - 8.2 [Geographic Expansion](#82-geographic-expansion)
   - 8.3 [The Profitability Path: 2026 and Beyond](#83-the-profitability-path-2026-and-beyond)
   - 8.4 [Consolidation in the PH Digital Bank Market](#84-consolidation-in-the-ph-digital-bank-market)
   - 8.5 [Trends: AI and Embedded Finance](#85-trends-ai-and-embedded-finance)
9. [Glossary](#9-glossary)
10. [References](#10-references)

---

## 1. Tonik Overview

### 1.1 The Short Answer

**Tonik Digital Bank, Inc. ("Tonik") is the Philippines' first private, standalone digital-only bank** — holder of **BSP Digital Banking License No. 001**, the first digital bank licence issued to a purely digital, privately-owned institution by the Bangko Sentral ng Pilipinas (BSP), the Philippine central bank. It is also widely described as the **first "neobank" in Southeast Asia** to operate under a full digital bank licence with deposit-taking powers.

The verified story, in brief:

- **Founding:** Tonik was founded in **2018** by Ukrainian serial fintech entrepreneur **Greg Krasnov** (Grygorii Krasnov), who structured it as Singapore-incorporated **Tonik Financial Pte Ltd** with the operating bank, Tonik Digital Bank, Inc., headquartered in **Pasig City, Metro Manila**. Krasnov's thesis: the Philippines had the biggest consumer-credit gap in Southeast Asia and a 70%-unbanked adult population, making it the ideal first market for a lending-led neobank.
- **Licence path:** Tonik first received a **rural bank licence** from the BSP in **December 2019** (a deliberate staging step), piloted its app in Q4 2020, launched publicly in **February–March 2021**, and was upgraded to a full **digital bank licence in June 2021** — the first private digital bank licensed under the BSP's new digital bank framework (Circular No. 1105, December 2020). The state-owned **Overseas Filipino Bank** (OFBank, a Landbank subsidiary) received the very first digital bank licence on 25 March 2021; Tonik's is licence **No. 001 among standalone/private digital banks** (per Tonik's own materials).
- **Positioning:** A **credit-led, mass-market consumer digital bank** — built on lending (99% of revenue), not on user counts or idle deposits. Krasnov's own framing: "revenue on a loan client is 20× higher than on a payment client." Products span savings (Tonik Account, Stash, Group Stash, Time Deposit, Tendo Stash), credit (Credit Builder personal loan, Shop Installment Loan, employer-channel salary-deduction loans via Tendo), a debit card, and QR Ph payments.
- **Technology:** Cloud-native stack on **Google Cloud** (API gateway **Apigee**), running on **Finastra Essence** core banking (confirmed by Finastra press materials, May 2025) — *not* Thought Machine Vault as some industry coverage had speculated. Heavy **AI** adoption across underwriting, collections, quality control and customer service.
- **Funding:** ~**US$169M+** raised across Series A (US$21M, Sequoia India + Point72, June 2020), Pre-Series B (US$17M, iGlobe-led, May 2021), and Series B (US$131M, Mizuho-led, February 2022), plus a reported US$12M top-up (flagged). Backers include Sequoia India (now Peak XV), Point72 Ventures, iGlobe Partners, Insignia Ventures Partners, Camerton Holdings and Mizuho Bank.
- **Performance (April 2026, per Tonik):** loan portfolio **US$110M** (+2.3× YoY); annualised revenue run-rate **US$60M+** (99% lending); net interest margin **51%**; lending RAROC **25–27%**; loan-to-deposit ratio **82%**; **net cash-flow profitable in Q1 2026** — the first standalone digital bank in the Philippines to reach profitability, and (per Tonik) one of the fastest non-ecosystem neobanks globally to do so. The regulated bank subsidiary reported IFRS profitability for Q1 2026.
- **Current status (2026):** still the **smallest balance-sheet player** among Philippine digital banks by assets (₱8.25B end-2025 vs Maya's ₱77.7B) — deliberately — but the **profitability leader**, with a business model explicitly designed around capital efficiency rather than deposit market share.

### 1.2 Verified Status at a Glance

| Attribute | Verified status |
|---|---|
| Is Tonik a real, operating bank? | **Yes** — Tonik Digital Bank, Inc., BSP-regulated, PDIC-insured |
| Licence type | **BSP digital bank licence** — "BSP Digital Banking License No. 001" (first standalone/private digital bank licence; June 2021) |
| Was Tonik the *first* BSP digital bank licence holder? | **No** — state-owned **Overseas Filipino Bank** received the first licence (25 March 2021); Tonik was the **first private/standalone** one |
| Founded / incorporated | **2018** (Tonik Financial Pte Ltd, Singapore); operating bank Tonik Digital Bank, Inc. (Philippines); HQ Pasig City |
| Rural bank licence | **December 2019** (BSP) — staging licence before the digital bank category matured |
| Public launch | **February 2021** (app live, per Tonik's own release) / **March 2021** (public launch, Manila Bulletin 19 Mar 2021) |
| Founder | **Greg Krasnov** (Grygorii Krasnov), Ukrainian serial fintech entrepreneur — ex-Bank of America (London) and Innova Capital (Warsaw) PE; founded Platinum Bank (Ukraine, 2006) and FORUM ventures (2015) — **not** an ex-Citi banker (the common "ex-Citi" claim is unverified/false in retrieved sources) |
| Core banking | **Finastra Essence** (cloud-native) — confirmed by Finastra press release, 8 May 2025; **not Thought Machine Vault** |
| Cloud | **Google Cloud** (confirmed by Google Cloud customer case study; Apigee API gateway) |
| Total funding | ~**US$169M+** (US$21M + US$17M + US$131M, plus a reported US$12M raise — flagged) |
| Customers | **1M+** (June 2023, PR Newswire/Tonik); later counts not publicly disclosed (flagged) |
| Profitable? | **Yes (Q1 2026)** — net cash-flow positive; bank subsidiary IFRS-profitable for Q1 2026 (Tonik announcement, May 2026) |
| Market position | First-mover digital bank; **sixth-largest digital bank by assets** (end-2025); profitability leader among PH digital banks |
| Deposit insurance | **PDIC**, currently up to **₱1,000,000** per depositor (raised from ₱500,000 under RA 12000, 2025) |

### 1.3 The Founding Story: A Neobank for the Philippines

**The opportunity.** When Krasnov began building Tonik in 2018, the Philippines was the archetypal underbanked market: a 2019 BSP report found **~70% of Filipino adults unbanked** (BSP FIS 2019: only 29% had bank accounts), and consumer lending penetration was "five times lower than the regional average" (Krasnov, CEO Magazine, September 2021). The Philippines is also a **remittance economy** — millions of overseas Filipino workers (OFWs) send money home (US$38.3B in personal remittances in 2024) to families who largely lack formal bank accounts. The COVID-19 pandemic and the 2020 Luzon enhanced community quarantine then supercharged demand for digital financial services exactly as the BSP finalised its digital bank framework.

**The staging strategy.** Rather than wait for the digital bank category (approved December 2020), Tonik took a **rural bank licence from the BSP in December 2019** — a lower hurdle that let it build, test (pilot Q4 2020) and prepare while the regulator finalised the digital bank rules. This "licence ladder" (rural → digital) is an under-appreciated piece of Tonik's first-mover advantage: by the time digital bank licences were issued in 2021, Tonik had an app, a brand, and a deposit base.

**The founding thesis.** Tonik was deliberately **not** built as a payments wallet (that space belonged to GCash) and **not** as a deposit-gathering machine (that was the incumbents' game). It was built as a **consumer credit bank for the 90% of Filipinos unserved by formal bank lenders** — using low-cost retail deposits (3–6%) to fund high-yield unsecured consumer loans, a structural margin edge of 10+ points over the 15%+ funding costs of non-bank lenders (Tonik profitability release, May 2026).

### 1.4 The Founder: Greg Krasnov

**Grygorii ("Greg") Krasnov** is a Ukrainian serial fintech and banking entrepreneur. Verified career arc (Finastra bio, FinTech Magazine, The Official Board, CEO Magazine):

- **Early career:** launched his first business at 18 (coding and PC repair) to fund college; then spent ~10 years in **private equity / leveraged buyouts at Bank of America (London)** and **Innova Capital (Warsaw)** as Managing Director, Private Equity. *Note: the widely-repeated "ex-Citi banker" description of Krasnov is **not** supported by any source retrieved in this research pass — his institutional pedigree is Bank of America and Innova Capital, not Citi.*
- **2006:** founded **Platinum Bank**, a Ukrainian consumer bank — his first bank-building experience, and the template for the lending-first model later applied at Tonik.
- **2015:** launched fintech venture firm **FORUM**, through which he invested in and co-founded five fintech startups, including **Credolab** (alternative credit scoring), **FLOW**, **Solarhome** and **AsiaKredit** (Asian digital lending) — a portfolio that effectively pre-built Tonik's credit-scoring playbook.
- **2018:** founded **Tonik** (incorporated Tonik Financial Pte Ltd in Singapore); serves as Founder, Chairman and CEO. *Note: Wikipedia's infobox lists "founded 2021-03-18" (the launch date) while the Official Board and FinTech.ph record the company's founding in 2018 — this guide follows the 2018 founding date, flagged as the company-founding (not launch) date.*
- Krasnov has said his management style is assembling teams stronger than himself in verticals ("I don't claim to be an expert on everything"), and he has been profiled as "the banker who brought digital banking to the Philippines" (FinTech Magazine, April 2022).

Tonik's Philippine country management is led by **Maria Lourdes Pineda** (President and Country Manager — Philippines), per Wikipedia's company infobox (flagged).

### 1.5 The Licence: BSP Digital Banking License No. 001

**The BSP digital bank framework.** The BSP formally created the **"digital bank" bank-category in December 2020** via **Circular No. 1105** ("Guidelines on the Establishment of Digital Banks"), published after the framework was announced in late 2020. A digital bank is defined by the BSP as "a bank that offers financial products and services that are processed end-to-end through a digital platform and/or electronic channels with no physical presence" — no branches, no physical touchpoints. The framework was part of the BSP's **Digital Payments Transformation Roadmap** targeting 50% of retail payment transactions digital and 70% of Filipino adults with financial accounts by 2023.

**The licence sequence (verified):**

| Date | Event |
|---|---|
| Dec 2020 | BSP Circular No. 1105 creates the digital bank licence category |
| 25 Mar 2021 | **Overseas Filipino Bank** (state-owned, Landbank subsidiary) receives the **first** digital bank licence |
| Jun 2021 | **Tonik** receives a digital bank licence — the **first private/standalone** digital bank licensed; issued as **Digital Banking License No. 001** (per Tonik) |
| Jun–Oct 2021 | Licences also granted to **UNObank**, **UnionDigital Bank** (UnionBank subsidiary), **GoTyme Bank**, and **Maya Bank** (last of the 2021 wave) |
| Aug 2021 | BSP announces a surprise **3-year moratorium** on new digital bank licences, then caps the count at **six** (Oct 2021) |
| Jan 2025 | Moratorium lifted; BSP reopens applications with **4 new slots** (max 10 digital banks) |
| Jul 2026 | **MariBank Philippines** (formerly SeaBank, Sea Group's rural-bank digital operator) upgrades to a digital bank licence — the **7th** licensed digital bank |

**Why "No. 001" matters — and the nuance.** Tonik's own materials (and much press coverage) describe it as "the first digital bank licence issued in the Philippines." The precise, verifiable claim is narrower: OFBank, a state-owned entity, was the first *licence holder*; **Tonik was the first private/standalone digital bank** to be licensed, and its licence carries the **No. 001** designation in the BSP's registry of standalone digital bank licences. This guide flags the distinction explicitly because both claims circulate and neither is wrong once the "private/standalone" qualifier is applied to Tonik's.

### 1.6 The Launch: February–March 2021

Tonik's app went live and began taking deposits in **February 2021** (Tonik's own May 2026 release: "Since launch in February 2021..."), with the formal public launch celebrated in **March 2021** (Manila Bulletin, "A revolutionary digital-only bank launches in the Philippines," 19 March 2021; ABS-CBN reported the 6% time deposit offer on 18 March 2021). Wikipedia's infobox uses 18 March 2021 as the "founded" date.

Launch impact was immediate and unprecedented for a Philippine bank launch:

- **₱1 billion (US$20M) in deposits in under a month** — Tonik called it "a historic record for any new bank launching in the country" (April 2021).
- The launch offer of **up to 6% p.a. time deposit interest** was described by the Philippine Daily Inquirer as "staggering" — roughly 3× the prevailing bank deposit rates.
- Tonik partnered with **Globe myBusiness** for distribution and aligned its launch messaging with the BSP's financial inclusion push (Philippine Star).

### 1.7 "First Digital Bank in the Philippines" — The Claim Verified

The claim needs careful unpicking because the Philippine market had several simultaneous "firsts":

| Claim | Verdict |
|---|---|
| "First digital-only bank in the Philippines" (launch timing) | **True with nuance** — CIMB Bank Philippines launched a digital-only presence in 2019 (under a commercial bank licence), and RCBC's DiskarTech (July 2020) and EastWest's Komo (May 2020) ran digital-only *services* before Tonik. Tonik was the first **BSP-licensed digital-bank-category** institution to launch publicly and the first **standalone private** digital bank operating end-to-end digitally. |
| "First neobank in Southeast Asia" | **Widely used, partially defensible** — Tonik is cited (Wikipedia, FinTech Magazine) as the first *licensed* neobank in SEA. Singapore's MAS digital banks launched later (2022); Indonesia's (Bank Jago etc.) were conversions. Tonik's claim to be SEA's first *licensed, standalone, deposit-taking* neobank is the strongest form of the claim. |
| "First digital bank licence (No. 001)" | **First private/standalone** — OFBank (state-owned) was licensed first in March 2021. |
| "First PH digital bank to reach profitability" | **True for standalone digital banks** (Q1 2026) — per Tonik's announcement; ecosystem-backed players had not publicly claimed profitability as of this pass (flagged). |

**Bottom line:** Tonik's honest positioning is *"the Philippines' first standalone private digital bank — BSP Digital Banking License No. 001"*, and SEA's first licensed neobank. The unqualified "first digital bank in the Philippines" phrasing is loose (CIMB's 2019 digital-only operation and OFBank's earlier licence both predate Tonik's launch), though Tonik was the first under the digital bank *category* to go to market.

### 1.8 Timeline: 2018–2026

| Year | Milestone |
|---|---|
| 2018 | Tonik founded by Greg Krasnov; holding company Tonik Financial Pte Ltd incorporated in Singapore |
| Dec 2019 | BSP grants Tonik a **rural bank licence** (staging step) |
| Q4 2020 | Pilot of the proprietary mobile app |
| Dec 2020 | BSP Circular No. 1105 creates the digital bank licence category |
| Feb–Mar 2021 | **Public launch**; ₱1B deposits in under a month |
| Jun 2021 | **Digital banking licence granted** (first private/standalone; "License No. 001") |
| Aug 2021 | ₱3.5B (US$68M) customer deposits; BSP announces 3-year licence moratorium |
| Feb 2022 | **US$131M Series B** led by Mizuho Bank (total funding >US$169M) |
| Jun 2023 | **1 million customers** milestone (PR Newswire) |
| 2023–2025 | Expansion beyond Metro Manila (Cebu); loan portfolio ×7 in two years; 1M+ cumulative loans disbursed; unit operational cost ÷5 |
| May 2025 | Finastra announces deepened Essence partnership; "Best Core Banking System Initiative" (RBI Awards 2025); IBSi Digital Banking Awards 2025 CX winner (Asia) |
| Q1 2026 | **Net cash-flow profitable**; bank subsidiary IFRS-profitable for Q1 2026 (announced May 2026); loan book US$110M; revenue run-rate US$60M+ |

### 1.9 Current Status (as of August 2026)

Tonik enters the second half of 2026 as the **profitability leader among Philippine digital banks but the smallest by balance sheet** — assets of ₱8.25B at end-2025 (BSP published financials), versus Maya Bank (₱77.7B), MariBank (₱68.2B), GoTyme (₱50.1B), UNObank (₱11.9B) and UnionDigital (₱11.8B). This is by design: Tonik's leadership has repeatedly said it measures success in **lending productivity and capital efficiency** (RAROC ~25–27%, net LTV/CAC 23×), not deposit share. Its next-phase priorities (per the May 2026 release): scaling employer-channel lending via **Tendo**, expanding the **merchant installment network**, and deepening **revolving credit** products for repeat borrowers.

### 1.10 What Could NOT Be Verified

- **Post-2023 customer counts.** The 1M-customer figure (June 2023) is solid; later registered-user numbers are not publicly disclosed by Tonik (flagged).
- **The US$12M additional raise** (reported by Fintech News Philippines, undated snippet in this pass) — amount, round name and date unverified.
- **Valuation.** No public valuation figure for Tonik was verified in this pass; the "unicorn" label sometimes attached to it (US$1B+) is **not** supported by retrieved sources and is likely a misreading of its total funding.
- **Krasnov's "ex-Citi" background** — asserted in some coverage, contradicted by every detailed bio retrieved (Bank of America + Innova Capital). Treated as unverified/unfounded.
- **Group Stash member-count mechanics and current Stash/TD rate tiers** — rates change frequently (launch was up to 6% p.a. on time deposits); current tiers from tonikbank.com were not scraped in detail (flagged).
- **Internet penetration and app-store ranking specifics** for the Philippines — quoted figures (see §6.8, §5.5) are flagged estimates from secondary sources.

---

## 2. The Business Model

### 2.1 Lending-First, Not User-Platform-First

The single most important fact about Tonik's business model is its **founding choice to be a credit institution, not a user platform**. Krasnov's stated logic (Tonik profitability release, May 2026): "While much of the narrative around the Philippine digital banking market has been defined by user growth, deposits, and payment volumes, Tonik was designed from day one as a lending institution — focused on deploying capital efficiently into consumer credit for the 90% of Filipinos unserved by formal bank lenders... revenue on a loan client is 20× higher than on a payment client."

This manifests in every dimension of the model:

- **99% of revenue comes from lending** (April 2026 run-rate), not interchange, fees or idle-deposit spread.
- **Loan-to-deposit ratio of 82%** — the highest among Philippine digital banks — meaning deposits are deployed, not parked.
- **Net interest margin of 51%** and **lending RAROC of 25–27%** — headline numbers for any bank, let alone a five-year-old neobank (these are Tonik-claimed figures; see §5.4 for caveats).
- **Net LTV/CAC of 23×** — the "upsell flywheel": a borrower's lifetime value is 23× the cost of acquiring them, justifying aggressive but disciplined customer acquisition.

### 2.2 The Target Customer

Tonik targets the **mass-market Filipino retail consumer**, explicitly the segment the formal banking system underserves:

- **The underbanked mass market** — the ~44% of Filipino adults still without accounts (2021 BSP FIS) and the ~70% who have never taken a bank loan (Krasnov, LinkedIn, 2026).
- **OFW families** — households receiving remittances (US$38.3B in 2024) who need safe deposit products and credit that local lenders won't provide; salary-deduction employer-channel lending (Tendo) is built for formal-sector workers in this segment.
- **Young professionals and first-time borrowers** — thin-file customers with no credit history, whom Tonik's AI underwriting can score from alternative data.
- **Small merchants** — via the Shop Installment Loan network (merchant-embedded consumer financing at point of sale).

The design principle is **"credit inclusion where the gap is"**: payment inclusion was already being solved by GCash and the e-wallets; credit inclusion was not.

### 2.3 The Product Portfolio: Savings

Tonik's deposit products are simple, high-yield and social — designed to attract the sticky, low-cost retail deposits that fund the loan book (3–6% cost of funds per Tonik's own figures). Verified product set from tonikbank.com (2026) and press coverage:

- **Tonik Account** — the core savings account: fully digital onboarding, no maintaining-balance friction, interest-bearing.
- **Stash** — a high-yield, goal-based savings product (savings "pots"); at launch the headline rates reached **up to 6% p.a.** on time deposits (ABS-CBN, March 2021; PDI called the rates "staggering"). Current tiered rates are published on tonikbank.com and change with the BSP rate cycle (flagged; exact current tiers not scraped in this pass).
- **Group Stash** — a **social savings** product: a shared savings pot with multiple members (family/friends), each member saving toward a common goal. This is Tonik's signature social-banking feature — group accountability as a savings driver in a collectivist market — and was a first in the Philippine market. (Member-count limits and interest mechanics unverified in detail — flagged.)
- **Time Deposit** — fixed-term deposits, the launch hero product at up to 6% p.a.
- **Tendo Stash** — a savings product tied to **Tendo**, Tonik's employer-channel platform (salary-linked savings/investment pot for formal-sector employees).
- **Debit Card** — a physical debit card (Visa-network; issued per tonikbank.com product menu) linked to the Tonik Account; the bank remains branchless but issues plastic for ATM/merchant use. (Card network and fees flagged.)
- **QR Ph** — participation in the national QR payment standard (BSP QR Ph), enabling QR-based payments from the app.

### 2.4 The Product Portfolio: Credit

The engine room. Verified loan products (tonikbank.com 2026 + profitability release):

- **Credit Builder Loan** — the flagship unsecured personal loan, marketed as "build credit for your financial ever after," with loan amounts up to **₱50,000** for eligible borrowers; instalment repayment in-app. Designed as a thin-file credit-building ladder.
- **Shop Installment Loan** — **merchant-embedded consumer financing**: point-of-sale instalment loans offered at partner merchants (the "merchant installment network"), financing consumer goods purchases.
- **Employer-channel lending (Tendo)** — **salary-deduction loans** distributed through employers: the employer deducts repayments from payroll, structurally reducing credit risk and collections cost. This is Tonik's highest-quality channel and a stated near-term growth priority.
- **Loan Repayment** — in-app repayment flows (not a separate product; listed for completeness).

Cumulative performance (per Finastra release, May 2025): **over 1 million loans disbursed since 2021**, loan portfolio **×7 in two years** (2023–2025), and by April 2026 the portfolio stood at **US$110M, +2.3× YoY — the growth leader among Philippine digital banks**.

### 2.5 The Product Portfolio: Cards, Payments, Insurance

- **Cards:** physical debit card (issued digitally, delivered physically); no credit card as of this pass (the credit line is served through instalment loans instead — flagged).
- **Payments:** InstaPay/PESONet rails (inbound/outbound transfers), QR Ph payments, and card payments. Tonik is **not** a payments/e-wallet business — payments are plumbing, not product.
- **Insurance:** Tonik has publicly discussed insurance distribution partnerships over the years (typical neobank cross-sell), but **no specific live insurance partnership was verified** in this pass — flagged. Note Tonik's *staged* insurance ambitions appear under "future outlook" (§8.1), not as a current verified product.

### 2.6 Revenue Model and Unit Economics

| Element | Detail (verified where stated) |
|---|---|
| Primary revenue | **Net interest income from consumer loans** — 99% of the US$60M+ annualised revenue run-rate (Apr 2026, Tonik-claimed) |
| Funding | **Retail deposits at 3–6%** (BSP-licensed deposit franchise) funding loans at consumer-credit yields — a structural >10-point margin edge over non-bank lenders' 15%+ funding costs |
| Cost-to-serve | **÷5 reduction in unit operational service costs** since 2021 (Finastra release, May 2025); AI-driven service: 90% of inquiries handled in-app, 75% resolved autonomously; customer-care productivity ×4 |
| Risk | AI underwriting on thin-file data (Credolab heritage); employer salary-deduction and merchant-embedded channels structurally derisk the book; cost of risk is deducted in the "positive cash net income after all costs including cost of risk" profitability claim (Q1 2026) |
| Efficiency metrics | NIM 51%; lending RAROC 25–27%; net LTV/CAC 23×; LDR 82% (all April 2026, Tonik-claimed) |
| Fees | Transaction/service fees exist (published in Tonik's "Fees & Charges") but are a minor revenue line; the model is spread-driven, not fee-driven |

**The unit-economics story in one line:** Tonik earns consumer-credit yields on deposits it funds at near-savings rates, using AI to underwrite borrowers the incumbents reject, and cloud + AI to serve them at a fifth of the historical unit cost.

### 2.7 The Customer Value Proposition

- **High rates:** launch-era up to 6% p.a. on deposits — roughly 3× prevailing bank rates, still the market's rate-setter among standalone digital banks (tiers change with BSP policy; flagged).
- **Convenience:** fully mobile onboarding and banking (eKYC in ~minutes), in-app loans, no branches, no minimum-balance friction — banking "anytime" on a phone (Google Cloud case study framing).
- **Trust:** a **BSP-regulated bank** (not an e-money wallet), with deposits insured by the **Philippine Deposit Insurance Corporation (PDIC)** — currently up to **₱1,000,000 per depositor** (raised from ₱500,000 by Republic Act No. 12000, effective 2025). The "licensed bank, not a wallet" distinction is central to Tonik's trust pitch versus GCash-style e-wallets.
- **Credit access:** for the majority of Filipinos who have never had a bank loan, Tonik offers a credible path to first credit and credit-building — the emotional core of "build credit for your financial ever after."
- **Social savings:** Group Stash turns saving into a shared, accountable social act — a culturally tuned feature the incumbents don't offer.

---

## 3. The Technology

### 3.1 Architecture Overview: Born in the Cloud

Tonik is a **cloud-native, API-first, mobile-first bank** with no branches and no data centres of its own. The architecture is a textbook **composable banking** stack (see [core_banking_systems_guide.md](core_banking_systems_guide.md) for the composable-banking framework): a SaaS core, a public cloud, an API gateway, and a thin proprietary layer of apps, decisioning and integrations. The three verified pillars:

| Pillar | Technology | Evidence |
|---|---|---|
| Core banking | **Finastra Essence** (cloud-native core) | Finastra press release, 8 May 2025; RBI "Best Core Banking System Initiative" award 2025 (with Finastra) |
| Cloud platform | **Google Cloud** | Google Cloud customer case study (cloud.google.com/customers/tonik) |
| API management | **Google Cloud Apigee** | Google Cloud case study; "powered by Google Cloud Apigee" |
| Payments rails | **InstaPay / PESONet / QR Ph** (BSP national rails) | Product menu; national scheme participation |
| AI layer | Proprietary + Essence-supported ML/genAI | Finastra release (5× efficiency gains; 75% autonomous resolution) |

### 3.2 Core Banking: Finastra Essence — Not Thought Machine Vault

**This is the guide's most important verification correction.** The task hypothesis (and some industry chatter) held that Tonik runs on **Thought Machine Vault**, the UK cloud-native core. The verified reality: **Tonik's core is Finastra's Essence** (the cloud-native evolution of the FusionBanking/Fusion Essence product family — see the vendor comparison in [temenos_guide.md](temenos_guide.md) and the core-systems landscape in [core_banking_systems_guide.md](core_banking_systems_guide.md)).

Primary-source evidence:

- **Finastra press release (8 May 2025):** "Tonik... continues to set new benchmarks in digital banking with rapid growth, operational excellence, and groundbreaking customer experience enhancements — **powered by Finastra's Essence core banking solution**." The release details Tonik's deepened engagement through Finastra's **"Agile Streams"** program across "functional, operational, and architectural capabilities."
- **Awards:** Tonik and Finastra jointly won **"Best Core Banking System Initiative"** at the 16th Annual **Retail Banker International (RBI) Awards 2025** — an award specifically about the core system.
- **Finastra customer story PDF** (July 2025): "Tonik drives financial inclusion and innovation through next-gen digital banking" — a dedicated Finastra-Tonik customer story.

No credible source in this research pass ties Tonik to Thought Machine Vault. (The Vault hypothesis likely arises from Tonik being a well-known cloud-native neobank + Vault's marquee SEA customer, GXS Bank in Singapore — a different bank entirely; see [green_link_digital_bank_guide.md](green_link_digital_bank_guide.md) for the SG landscape.) For the Thought Machine/Vault vendor profile, see the vendor table in [us_bank_core_systems_guide.md](us_bank_core_systems_guide.md).

**What Essence gives Tonik:** a rich retail deposit + lending product catalogue out of the box, cloud deployment (Essence runs on hyperscaler infrastructure), and open APIs — the "rich, broad and deep banking functionality" Finastra's EVP cites as the reason the partnership works at scale. Tonik's proprietary differentiation lives *around* the core: AI underwriting, the app, the merchant/employer channels — not inside the ledger.

### 3.3 Cloud Platform: Google Cloud

**Google Cloud is confirmed as Tonik's cloud** (Google Cloud customer case study, cloud.google.com/customers/tonik). The case study's framing: "Tonik provides consumers with the convenience of banking anytime, powered by Google Cloud Apigee. All they need to do is download the Tonik app... verify their identity for account opening, and start banking."

The Google Cloud relationship provides:

- **Apigee API gateway** — the API management layer through which the mobile app and partner integrations consume banking capabilities (the case study names Apigee specifically).
- **Hyperscaler scale and resilience** — elastic compute/storage for a fast-growing deposit and loan book without capacity planning on-prem.
- **A path to AI** — BigQuery analytics and Google's AI stack for the ML/genAI initiatives Tonik describes (underwriting, service, fraud — see §3.7).

*Caveat flagged:* a September 2021 CEO Magazine interview contains a sentence fragment referencing **Microsoft Azure** ("...Microsoft Azure to help power these solutions"), suggesting either a multi-cloud period or an early-stage Azure component before the Google Cloud consolidation. The current, primary-source-verified position is Google Cloud (Apigee), with the Azure reference flagged as an unresolved historical detail.

### 3.4 API-First and the Apigee Gateway

Tonik's architecture is **API-first by construction**: the mobile app, the Tendo employer platform, merchant partners (Shop Installment Loan), and payment rails all integrate through APIs rather than screen-scraping or file batch. The **Apigee gateway** is the single choke-point for external API traffic — authentication, rate limiting, traffic management and analytics. This API-first posture is what makes the **embedded-finance plays** (merchant checkout financing, employer payroll integration) cheap to add — the same composable pattern documented in [programmable_business_bank_guide.md](programmable_business_bank_guide.md).

### 3.5 Digital Onboarding and eKYC

Onboarding is fully digital: download app → verify identity → start banking (Google Cloud case study). The Philippines' **eKYC** environment is governed by BSP regulations on electronic know-your-customer (allowed since BSP Circular No. 706 and operationalised through the "e-KYC" guidelines and the national ID system):

- **ID verification** uses government IDs; the rollout of the **Philippine national ID (PhilSys)** has progressively strengthened the eKYC base since 2021.
- **Face/liveness checks and OCR** are standard in the local eKYC stack (vendor specifics for Tonik flagged).
- eKYC is the **make-or-break for financial inclusion**: it is what lets a rural borrower open an account without a branch visit — the entire Tonik model depends on it.

### 3.6 Payments: InstaPay, PESONet and QR Ph

Tonik plugs into the Philippines' national payment rails — see [financial_infrastructure_guide.md](financial_infrastructure_guide.md) for the rails taxonomy and [mojaloop_guide.md](mojaloop_guide.md) for the InstaPay design (a BancNet/Vocalink-built instant payment switch — the Philippines' equivalent of Singapore's FAST):

- **InstaPay** — the 24/7 real-time low-value retail rails (₱500k per-transaction cap): used for instant deposits/withdrawals, transfers to/from other banks and e-wallets (GCash, Maya wallets interoperate via InstaPay), and merchant QR payments. InstaPay volumes in the Philippines have grown explosively (hundreds of millions of transactions monthly by 2025 — flagged secondary estimate).
- **PESONet** — the batch, higher-value rails (no per-transaction cap): used for larger transfers and salary payments, including the Tendo employer-channel disbursements.
- **QR Ph** — the BSP's national QR standard: Tonik supports QR Ph payments from the app, interoperable with GCash/Maya/other banks' QR.

**The GCash question:** Tonik is *not* integrated as a "GCash wallet inside Tonik" — it competes with the GCash e-wallet for deposits, while remaining interoperable with it through the rails (InstaPay transfers between Tonik and GCash work like any bank-to-wallet transfer). Tonik's launch distribution partnership with **Globe myBusiness** (May 2021) gave it merchant-side reach without making GCash (Globe-adjacent via Mynt, but owned by Globe's rival ecosystem in practice — Mynt is Globe-backed, actually — flagged) a dependency. *Note: Mynt (GCash) is backed by Globe Telecom and Ayala; the launch partnership was with Globe myBusiness, Globe's SME arm — a distribution alignment, not a technology integration.*

### 3.7 AI Everywhere: Underwriting, Service, Fraud

Tonik is openly **AI-first** — its May 2026 announcement claims "AI-driven risk management" as one of the three structural decisions behind profitability. Verified AI claims (Finastra release, May 2025; Tonik materials):

- **Underwriting:** five years of data and model iteration enabling **profitable lending to thin-file borrowers**; alternative-data scoring lineage from Krasnov's earlier fintech **Credolab**. AI underwriting is the core differentiator in a market where most adults have no credit history.
- **Operational efficiency:** "gains of over **5× in efficiency** across quality control, underwriting, and advanced voice analysis."
- **Customer service:** gen-AI-powered service where **90% of inquiries are handled in-app and 75% resolved autonomously**; customer-care productivity up **4×** with lower response times and costs.
- **Fraud:** AI-based fraud detection on the transaction stream (details proprietary — flagged).

This is the practical, ROI-driven AI adoption pattern — contrast with the governance-heavy FEAT-style adoption in Singapore (see [dbs_bank_guide.md](dbs_bank_guide.md) for the incumbent AI context).

### 3.8 Technology Partners

| Partner | Role | Evidence |
|---|---|---|
| **Finastra** | Core banking (Essence) + Agile Streams engineering engagement | Finastra press release 8 May 2025; RBI 2025 award |
| **Google Cloud** | Cloud platform + Apigee API gateway | Google Cloud customer case study |
| **BancNet / BSP rails** | InstaPay/PESONet/QR Ph connectivity | National scheme participation (scheme operator BancNet built InstaPay) |
| **Credolab (affiliate)** | Alternative credit-scoring DNA (Krasnov co-founded) | Founder biographies — strategic lineage, not a live vendor contract (flagged) |
| **Tendo** | Employer-channel platform (salary-deduction lending) | Tonik product/release materials |

### 3.9 Security and Fraud

- **BSP-regulated bank-grade controls**: capital adequacy, risk management and AML/CFT obligations identical to any Philippine bank; the "regulated bank, not a wallet" trust position (§2.7).
- **PDIC-insured deposits** (₱1M per depositor) — the safety net that makes high rates trustworthy.
- **eKYC identity verification** at onboarding (§3.5) plus transaction monitoring for AML; AI fraud detection on streams (flagged proprietary).
- **24/7 fraud hotline** (Tonik footer: fraud concerns handled by a 24/7 hotline; regular customer service 6:00–21:00).
- Emerging-market fraud realities: SIM-swap, social-engineering and phishing attacks are endemic in the Philippines; digital banks there routinely fight synthetic-ID fraud. Tonik's AI voice analysis and device/behavioural signals (per Finastra release) are part of the countermeasure stack (specifics flagged).

### 3.10 Architecture Risks

- **Vendor concentration — core:** Finastra Essence is the single most consequential dependency; Essence's roadmap, SaaS posture and commercial terms shape Tonik's agility. Mitigation: the "Agile Streams" co-engineering engagement deepens Tonik's influence over its own core.
- **Vendor concentration — cloud:** Google Cloud is a single-cloud dependency (with a historical Azure fragment — flagged). A cloud-agnostic core would ease exit, but no such plan is public.
- **Connectivity and device reality:** the Philippines' internet penetration (~74% of population, 2024 — flagged secondary estimate) and mobile-data quality constrain app-first banking; Tonik's target segment skews to lower-end devices and data plans. Mitigation: lightweight app design, USSD-free but offline-tolerant flows (flagged as Tonik-specific implementation details are not public).
- **Fraud and scam economics:** thin-file lending attracts both fraud rings and legitimate demand; AI models must continuously re-train as fraud adapts.
- **Regulatory dependence:** the digital bank licence and PDIC status are the trust moat — any BSP rule change (e.g., rate caps, capital floors, the 2025 reopening of licences) directly moves Tonik's competitive and cost position.

---

## 4. Funding and Investors

### 4.1 Funding History

Verified rounds (all from press/primary sources):

| Round | Date | Amount | Lead / participants |
|---|---|---|---|
| Series A | **June 2020** | **US$21M** | **Sequoia India + Point72 Ventures** (with "significant participation" from others — iGlobe, Insignia, Camerton per later disclosures) |
| Pre-Series B | **May 2021** | **US$17M** | iGlobe Partners-led (finews.asia, May 2021) |
| Series B | **February 2022** | **US$131M** | **Mizuho Bank-led** (TechNode, DealStreetAsia, Verdict — Feb 2022) |
| Top-up (reported) | ~2025–2026 | **US$12M** | Reported by Fintech News Philippines — round name/date unverified (flagged) |

*Correction to the common narrative:* the US$21M round was the **Series A** (June 2020, Sequoia India + Point72), not a "seed" — the task brief's "US$21M seed" framing is wrong. Whether a smaller seed round preceded it is not documented in retrieved sources (flagged). The "US$35M Series A" figure sometimes cited appears to be a conflation of the US$21M Series A + US$17M Pre-Series B.

### 4.2 Total Raised and the Investor List

- **Total: ~US$169M+** (US$21M + US$17M + US$131M + US$12M reported top-up). The task brief's "~US$116M" total is incorrect; the ~US$169M figure (pre-top-up) is well-documented.
- **Investors (verified list):** Sequoia Capital India (**Peak XV** after the 2023 Sequoia split), **Point72 Ventures**, **iGlobe Partners**, **Insignia Ventures Partners**, **Camerton Holdings**, and **Mizuho Bank** (strategic, Japan's major banking group — its first such fintech investment in the region per coverage).
- **Strategic logic of the investor set:** Sequoia/Point72/Insignia are classic growth-stage tech investors; iGlobe is a Singapore-based deep-tech fund; Camerton is the Krasnov-affiliated fintech holding (founder participation); Mizuho brings banking credibility and Japan-Philippines corridor potential.

### 4.3 Valuation and the "Unicorn" Question

- **No public valuation** for Tonik was verified in this research pass. The Series B was reported as "US$131M at an undisclosed valuation."
- The **"first Philippine digital bank unicorn"** label attached to Tonik in some coverage is **not supported** by retrieved sources — no US$1B+ valuation event was found, and Tonik's own materials never claim unicorn status. The label appears to conflate "most-funded PH fintech" with "unicorn." Tonik's positioning is profitability-first, not valuation-first — consistent with Krasnov's public stance that "profitability in digital banking is a function of what you choose not to do."

---

## 5. Performance and Milestones

### 5.1 Customer Growth

| Milestone | Date | Source |
|---|---|---|
| Launch: ₱1B (US$20M) deposits in <1 month | Apr 2021 | BusinessMirror, Tonik |
| ~100k customers (early-growth phase) | 2021–2022 | Not directly verified in this pass (flagged); Tonik reported strong early adoption but the specific 100k milestone was not found in retrieved sources |
| **1,000,000 customers** | **June 2023** | PR Newswire ("Tonik Hits 1M customers", 6 Jun 2023) |
| 1,000,000 cumulative loans disbursed | by May 2025 | Finastra press release |
| Post-2023 customer counts | not disclosed | flagged |

The 1M-customer milestone (June 2023, ~27 months post-launch) made Tonik one of the fastest-growing neobanks globally at the time, per its own Series B-era framing. Note the honest caveat: "customers" in these releases means registered/onboarded users, not necessarily active depositors — an industry-standard but loose metric.

### 5.2 Deposits

- **₱1B (US$20M)** in retail deposits in under a month (April 2021) — a Philippine launch record.
- **₱3.5B (US$68M)** by August 2021 (Philippine Star).
- **~US$150M in consumer deposits** by the 2023–2024 period (Krasnov podcast interview — flagged secondary source).
- End-2025 assets: **₱8.25B** (BSP published financials) — the smallest among licensed digital banks, deliberately, given the 82% LDR (deposits are deployed into loans, not hoarded).

### 5.3 Loans

- **US$110M loan portfolio (April 2026), +2.3× YoY** — the growth leader among Philippine digital banks (Tonik release, May 2026).
- **1M+ cumulative loans disbursed since 2021**; portfolio **×7 in two years** (Finastra, May 2025).
- Channels: digital personal loans (Credit Builder), merchant instalments (Shop Installment Loan), employer salary-deduction (Tendo).
- **LDR 82%** — highest among PH digital banks; loan yield reflected in a **51% NIM**.

### 5.4 Revenue and Profitability

The headline verified performance story (Tonik announcement, May 2026; cross-checked with Krasnov's LinkedIn, which states "Q1 26: $60M+ Revenue Run-Rate · 27% RAROC · Net Cash Flow Profitable as of Q1 2026"):

- **Revenue:** annualised run-rate **US$60M+**, **99% from lending** (April 2026).
- **Profitability:** **positive cash net income in Q1 2026** — after all costs including cost of risk — making Tonik **the first standalone digital bank in the Philippines to achieve profitability**, and "one of the fastest non-ecosystem neobanks globally" to do so (Tonik's claim). The regulated bank subsidiary, Tonik Digital Bank, Inc., also achieved **IFRS profitability for Q1 2026**.
- **Efficiency:** NIM 51%; lending RAROC 25–27%; net LTV/CAC 23×; unit operational service costs ÷5 since 2021; AI service automation (75% autonomous resolution).
- **What "profitable" does and doesn't mean:** the Q1 2026 claims are **Tonik-reported** and have not been independently audited in this pass (flagged). They are cash/IFRS profitability of the operating bank and group — distinct from the "path to profitability" posture of most digital banks globally, which remain in heavy investment phase. For the PH cohort context: no other Philippine digital bank had publicly claimed profitability as of this pass (Maya, GoTyme and MariBank are all still investing for growth — see [6.8](#68-challenges)).

**The profitability formula (Tonik's own framing):** AI-driven risk management (thin-file lending at profitable risk-adjusted yields) + portfolio diversification across three credit channels (structural derisking) + a BSP deposit franchise funding at 3–6% versus 15%+ for non-bank lenders (structural margin edge).

### 5.5 Rankings and Awards

- **"Best Core Banking System Initiative"** — 16th Annual Retail Banker International (RBI) Awards 2025 (with Finastra).
- **IBSi Digital Banking Awards 2025** — Regional Winner (Asia) and Category Winner for **"Customer Experience."**
- **"Best Customer Service for Digital Banks 2023"** — Philippine Daily Inquirer + Statista customer-choice awards (per FinTech.ph listing).
- IBS Intelligence named Tonik among "5 Top FinTechs in the Philippines to watch out for in 2021."
- App-store rankings: Tonik is consistently among the top finance apps in the Philippines, but **specific ranking positions were not verified** in this pass (flagged); it consistently trails GCash and Maya in consumer mindshare by download volume.

---

## 6. The Philippine Digital Banking Landscape

### 6.1 The Philippine Banking Sector

The Philippines has a **large, fragmented banking system** — roughly **500+ banking institutions** regulated by the BSP (flagged as approximate; the BSP's supervised banking network comprises ~45 universal/commercial banks, ~80–90 thrift/rural categories, and ~400 rural and cooperative banks). The market is dominated by a handful of universal banks:

- **BDO Unibank** (SM Group) — the largest bank in the Philippines by assets.
- **BPI** (Ayala Group) — #2; parent of the **UnionDigital** digital bank.
- **Metrobank** (GT Capital) — #3.
- **PNB** — state-adjacent, government-owned (the Philippine National Bank).
- **Landbank** and **Development Bank of the Philippines** — state banks (Landbank owns **Overseas Filipino Bank**, the first digital bank licence holder).

Below the universal banks sit the thrift banks, the **~400 rural banks** (the licensed-but-small institutions that OwnBank, NetBank, Salmon and MariBank/SeaBank's original licence came from), and the e-money issuers (GCash/Mynt, Maya wallet). The universal banks have responded to digital competition with their own apps and digital sub-brands (BPI's UnionDigital, UnionBank's own push, RCBC's DiskarTech, EastWest's Komo) — the incumbents are not standing still.

### 6.2 Financial Inclusion: From 70% Unbanked to the 2023 Targets

The BSP's Financial Inclusion Survey (FIS) series is the authoritative measure (verified figures):

| Survey year | Adults with bank accounts | Unbanked adults |
|---|---|---|
| 2017 FIS | **22.6%** | **77.4%** |
| 2019 FIS | **~29%** | **~71%** ← the "70% unbanked" figure Tonik's founding narrative cites |
| 2021 FIS | **56%** | **44%** (COVID-era digital surge) |
| BSP target | **70% by 2023** (Digital Payments Transformation Roadmap + NSFI) | — |

The 2021 jump (29% → 56% account ownership) was driven by COVID-era digitisation, the digital banks' launch wave, and GCash/Maya wallet adoption. The BSP's **National Strategy for Financial Inclusion (NSFI)** coordinates the inclusion push across agencies; the Digital Payments Transformation Roadmap set the "50% digital transactions / 70% account ownership by 2023" targets (outcomes by 2023: partially met — flagged; account ownership rose further but the "70%" headline was achieved in the 2022–2023 window per BSP claims, exact latest FIS figure flagged).

**Why this matters for Tonik:** the inclusion push is Tonik's demand-side tailwind — every newly banked Filipino is a potential depositor and, more importantly for Tonik, a potential first-time borrower (70% of Filipinos have never had a bank loan — Krasnov).

### 6.3 OFW Remittances: The Macro Backdrop

The Philippines is one of the world's largest remittance markets (verified BSP figures):

- **2024 personal remittances: US$38.34B** (record, +3% from US$37.21B in 2023); **cash remittances US$34.49B** in 2024.
- **2025 cash remittances: US$35.63B** (new record, ~₱2.07 trillion).
- ~10% of the population works abroad; remittances are ~8–9% of GDP.
- Middle East sources ~18% of cash remittances (US$6.13B of US$34.49B in 2024) — and are exposed to Gulf conflict risk (Inquirer, 2026).

Remittances matter to Tonik twice over: they fund household deposits (families park remittance inflows in high-yield accounts) and they create the credit demand (consumption smoothing against remittance cycles) that Tonik's loan book serves. The employer-channel (Tendo) model also aligns with formal-sector OFW-family borrowers.

### 6.4 The BSP Digital Bank Framework

Verified framework timeline (see also §1.5):

1. **Dec 2020 — Circular No. 1105** creates the digital bank category: "a bank that offers financial products and services that are processed end-to-end through a digital platform... with no physical presence." Digital banks must have paid-in capital, BSP approval of business plans, and are subject to the same prudential rules as other banks.
2. **2021 — the licence wave:** OFBank (25 Mar), Tonik (Jun), UNObank, UnionDigital, GoTyme, Maya (Sep, the last of the wave) = **six licences**.
3. **Aug–Oct 2021 — moratorium and cap:** BSP surprised the market with a **3-year moratorium** on new digital bank licences and capped the total at **six**, "to preserve competition" and let the cohort mature.
4. **Jan 2025 — reopening:** BSP reopened applications with **four new slots** (maximum ten digital banks), triggering a new race.
5. **Jul 2026 — the 7th licence:** **MariBank Philippines** (Sea Group's digital operator, formerly SeaBank under a rural bank licence) upgraded to a digital bank licence.

**Key regulatory dynamics:** (a) the digital bank category is a *licence type*, not a technology standard — rural banks and commercial banks that operate digitally (OwnBank, CIMB, SeaBank-pre-2026) are *not* digital-bank licensees; (b) the moratorium created a **regulatory moat** for the original six — no new entrant could licence between Aug 2021 and Jan 2025; (c) the 2025 reopening adds competitive pressure for Tonik in exactly the period it is claiming profitability.

### 6.5 The Licensed Digital Banks

As of July 2026 there are **seven BSP-licensed digital banks** (BSP published financial statements; Wikipedia "Digital banks in the Philippines"):

| # | Digital bank | Backer | Licence date | End-2025 assets |
|---|---|---|---|---|
| 1 | **Overseas Filipino Bank** | Landbank (state) | 25 Mar 2021 (first) | ₱5.55B |
| 2 | **Tonik Digital Bank** | Sequoia/Peak XV, Point72, iGlobe, Insignia, Camerton, Mizuho | Jun 2021 (first private) | ₱8.25B |
| 3 | **UNObank** | UNOAsia (ex-Citibank founders) | 2021 | ₱11.94B |
| 4 | **UnionDigital Bank** | UnionBank (Ayala/Aboitiz family group) | 2021 | ₱11.83B |
| 5 | **GoTyme Bank** | Gokongwei (Robinsons) + Tyme (South Africa) | 2021 | ₱50.11B |
| 6 | **Maya Bank** | PLDT/Voyager (MVP group), PayMaya | Sep 2021 (last of wave) | ₱77.66B |
| 7 | **MariBank Philippines** | Sea Group (ex-SeaBank, rural-bank licensee) | Jul 2026 (upgrade) | ₱68.20B |

*Ranking note:* by end-2025 assets Tonik is **6th** of 7 — the smallest private digital bank by balance sheet, deliberately (credit-led, 82% LDR), versus Maya/MariBank/GoTyme which run deposit-heavy models.

### 6.6 The PH Digital Bank Race: Comparison Table

| Attribute | **Tonik** | **GoTyme** | **Maya** | **MariBank (ex-SeaBank)** | **UNObank** | **UnionDigital** | **OFBank** |
|---|---|---|---|---|---|---|---|
| Backers | Sequoia/Peak XV, Point72, iGlobe, Insignia, Camerton, Mizuho | Gokongwei group + Tyme (SA) | PLDT/Voyager, MVP | Sea Group (Garena/Shopee) | UNOAsia (ex-Citi execs) | UnionBank (BPI family group) | Landbank (state) |
| Launch | **Feb–Mar 2021** (first mover) | Oct 2022 | 2022 | 2022 (as SeaBank) | Early 2022 | 2021–22 | 2021 (converted) |
| Model | **Lending-first** | Ecosystem retail (Robinsons malls) | Payments/wallet-first (PayMaya) | Ecosystem retail (Shopee) | Savings-first, high rates | Incumbent digital arm | State remittance/inclusion bank |
| Customers | 1M+ (Jun 2023; later undisclosed) | **5.1M** (Dec 2024), ₱24B deposits | ~2.5M+ bank customers (flagged) | undisclosed (flagged) | undisclosed | undisclosed | state service |
| End-2025 assets | ₱8.25B | ₱50.11B | ₱77.66B | ₱68.20B | ₱11.94B | ₱11.83B | ₱5.55B |
| Profitability | **Q1 2026 (first standalone)** | No public claim | No public claim | No public claim | No public claim | No public claim | state-subsidised |
| Signature product | Credit Builder Loan, Group Stash | GoTyme "everyday banking" + rewards | Maya Savings, Maya Credit | Shopee-integrated savings | High-yield savings | Corporate/digital lending | OFW-focused |

**The strategic shapes of the race:**

- **Ecosystem banks (GoTyme, Maya, MariBank)** — distribution-led: Robinsons retail, PayMaya's wallet, Shopee's commerce. They win on customer count and deposits; they pay for it with higher cost-to-serve and thinner lending.
- **Standalone banks (Tonik, UNObank)** — balance-sheet-led: no captive distribution, so they must win on product, rates and unit economics. Tonik's answer is credit; UNObank's is deposit rates.
- **Incumbent arms (UnionDigital, OFBank)** — compliance-adjacent: UnionDigital extends UnionBank's franchise; OFBank serves the state's inclusion/remittance agenda.

**The GCash factor:** GCash (Mynt, Globe + Ayala-backed) is the Philippines' dominant e-wallet with tens of millions of users and **no banking licence** — it partners with banks (CIMB historically) for deposits. GCash is simultaneously Tonik's biggest payments competitor and a funnel partner (instapay transfers). Its scale (flagged: 90M+ registered users claims) dwarfs every digital bank; the digital banks' counter is the **trust of a regulated deposit franchise** and **credit** — which e-wallets cannot offer directly.

### 6.7 Near-Banks: GCash, CIMB, OwnBank and the Rural-Bank Digital Operators

A crucial taxonomy point: **not every "digital bank" in the Philippines holds a digital bank licence**:

- **CIMB Bank Philippines** — Malaysian CIMB Group's subsidiary; digital-only operations since **2019** (OCTO app) under a **commercial bank licence**. A digital operator, not a digital-bank licensee. Awarded "Best Digital Bank 2021" by Global Banking and Finance Review.
- **OwnBank** — **Rural Bank of Cavite City** operating digitally; holds a **rural bank licence**, not a digital bank licence (though press often lumps it in; the task brief's premise that OwnBank is one of the "6 digital banks licensed" is incorrect).
- **NetBank** — Community Rural Bank of Romblon, digitally transformed (rural licence).
- **Salmon** — Rural Bank of Sta. Rosa (Laguna), tech-transformed (rural licence).
- **DiskarTech** (RCBC) and **Komo** (EastWest) — digital *services* of licensed universal banks, not separate licences.
- **GCash/Mynt** — e-money issuer licence (BSP e-money regulations), not a bank; Mynt's banking ambitions are pursued through partnerships.

This matters because the BSP's licence taxonomy (digital bank vs rural bank vs e-money) determines deposit insurance treatment, capital requirements and product scope — and the "digital bank race" table in press coverage routinely mixes all three categories.

### 6.8 Challenges

- **Profitability:** as of this pass, **only Tonik claims profitability** among the digital banks; Maya, GoTyme, MariBank and UNObank remain in investment mode (Fintech News Philippines, Feb 2026: "Philippines Digital Banks Face Profit Challenges As New Rivals Loom"). The cohort's deposit-heavy models carry heavy marketing/rewards costs; thin lending books cap net interest income.
- **Competition from the wallets and incumbents:** GCash and Maya wallet dominate the payments layer; BDO/BPI/Metrobank counter with their own apps; the 2025 licence reopening adds four potential rivals.
- **Trust and fraud:** scam/social-engineering fraud is endemic; a single high-profile fraud incident can dent the "safe digital bank" trust position; PDIC insurance and BSP regulation are the counterweights.
- **Infrastructure:** internet penetration (~74%, flagged) and data affordability still exclude a chunk of the target segment; interoperability (InstaPay caps, QR fragmentation) constrains the experience.
- **Regulatory risk:** BSP rate caps on consumer loans (the Truth in Lending / usury framework historically caps finance charges), capital requirements, and AML burdens; the digital bank framework's evolution post-2025.
- **Funding environment:** post-2022 the global fintech funding winter hit SEA; Tonik's reported US$12M top-up (flagged) suggests a leaner fundraising climate than the 2020–2022 era.

---

## 7. The Architect's Perspective

### 7.1 Tonik as an Architecture Case Study

For a banking architect, Tonik is valuable as a **complete, profitable, small-balance-sheet neobank** — proof that the "buy the core, rent the cloud, differentiate with data/AI" pattern can reach profitability in ~5 years in an emerging market. The architectural essence:

- **Composable banking in production** — SaaS core (Finastra Essence) + hyperscaler (Google Cloud) + API gateway (Apigee) + proprietary AI layer. See [core_banking_systems_guide.md](core_banking_systems_guide.md) for the composable-banking framework this instantiates.
- **Credit-led product architecture** — the ledger is a means to an end; the *decisioning* (AI underwriting, channel integrations) is the moat. This inverts the typical "deposits-first" digital bank architecture.
- **Embedded distribution** — Tendo (employer) and Shop Installment (merchant) channels are embedded-finance patterns (see [programmable_business_bank_guide.md](programmable_business_bank_guide.md)): banking capabilities delivered inside employers' payroll and merchants' checkouts through APIs.

### 7.2 Cloud-Native Core Lessons

1. **A cloud-native core compresses time-to-market.** Tonik went from founding (2018) to a licensed, funded, operating bank (Feb 2021) in under three years — including a licence change (rural → digital). A greenfield in-house core would have added years. Essence's out-of-the-box deposit/loan product catalogue meant Tonik's engineering focused on the moat (AI, channels), not the ledger.
2. **The core is a commodity; the data layer is the moat.** Five years of thin-file underwriting data and models (the "AI-driven risk management" profitability pillar) are what competitors can't buy — the ledger is table stakes.
3. **Unit-cost economics are an architecture outcome.** The ÷5 unit operational cost reduction and 75% autonomous service resolution are direct products of cloud automation + gen-AI — architecture choices, not marketing.
4. **License-adjacent strategy has an architectural corollary:** the rural-bank staging licence let Tonik build and pilot the *full stack* (app, eKYC, rails integration) before the digital licence arrived — the platform was launch-ready the day the licence landed.

### 7.3 Build vs Buy: The Bought-Core Lesson

The task brief hypothesised **Thought Machine Vault**; the verified reality is **Finastra Essence** (see §3.2). Whichever vendor, the *pattern* is the same and it is the pattern's lesson that matters:

| | Build (incumbent style — e.g. DBS) | Buy (Tonik style) |
|---|---|---|
| Time-to-market | 5–10 years (in-house cores) | <3 years to licensed launch |
| Control | Total | Vendor-constrained (roadmap, pricing, SaaS SLAs) |
| Cost profile | Huge capex + permanent headcount | Opex subscription; cost scales with usage |
| Innovation locus | In-house engineering | Proprietary layer around the core + vendor co-engineering |
| Risk | Delivery risk, talent risk | **Vendor concentration** (see [us_bank_core_systems_guide.md](us_bank_core_systems_guide.md) vendor table; [dbs_bank_guide.md](dbs_bank_guide.md) for the build-side comparator) |

Tonik's mitigation is the **"Agile Streams" co-engineering engagement** with Finastra — buying influence over the core without owning it. For an architect this is the modern answer to vendor lock-in anxiety: *deepen the partnership rather than repatriate the platform* — as long as the proprietary differentiation layer stays in-house.

### 7.4 Designing for the Underbanked: Credit-Led Architecture

The underbanked-market design principles Tonik demonstrates:

- **Design for thin-file credit, not just deposits.** The profitable insight: payment inclusion was commoditising (wallets), credit inclusion was not. The architecture therefore invests in alternative-data decisioning (Credolab lineage), channel-integrated collection (salary deduction), and merchant-embedded point-of-sale lending.
- **Trust-by-licence.** In a fraud-prone market, the BSP licence + PDIC insurance are architectural *features* — they are the trust layer no e-wallet can replicate. Architecturally this means regulatory reporting, AML and data-protection capabilities are first-class citizens from day one.
- **Mobile-native, data-poor reality.** The target customer has a mid-range Android phone, intermittent connectivity and no credit history; the app and APIs must be lightweight, resilient and tolerant of poor networks (see [financial_infrastructure_guide.md](financial_infrastructure_guide.md) for emerging-market rails context).
- **Distribution via rails, not branches.** Employer and merchant channels are the "branch network" of the digital bank — implemented as API integrations (Tendo payroll APIs, merchant checkout APIs), which is why the API gateway is a strategic asset.

### 7.5 Architecture Risks and Mitigations

| Risk | Tonik specifics | Mitigation posture |
|---|---|---|
| Core vendor concentration | Finastra Essence is a single point of dependency | Agile Streams co-engineering; keep differentiation out of the core |
| Single-cloud dependency | Google Cloud (historical Azure fragment flagged) | Cloud-agnostic core eases exit; no public multi-cloud plan |
| AI model drift / fraud adaptation | Thin-file underwriting models face adversarial fraud | Continuous re-training; voice/behavioural signals; 24/7 fraud ops |
| Regulatory dependence | BSP licence, PDIC, rate caps, 2025 licence reopening | Compliance as architecture; scenario planning for rate caps |
| Connectivity constraints | ~74% internet penetration (flagged) | Lightweight app; rail-based (InstaPay/PESONet) fallbacks |
| Scale economics | Small balance sheet (₱8.25B assets) | Lending-first deployment (82% LDR); opex discipline via AI |

### 7.6 The Architect's Checklist

For an architect evaluating or building a similar digital bank (e.g., in another ASEAN market):

1. **Verify the licence path first** — can you stage licences (rural → digital) like Tonik did, or is the digital licence the only gate? The licence determines the timeline, capital and trust position.
2. **Choose the core by the market's product mix, not vendor hype** — Tonik's Essence choice (rich retail deposits + loans, cloud-native) fitted a credit-led model; validate the vendor's reference customers in your market segment before signing.
3. **Put the moat outside the core** — underwriting, channels, service AI, fraud: the proprietary layer is where LTV/CAC and RAROC are won.
4. **Design distribution as API integrations** — employer, merchant and wallet channels should be pluggable via an API gateway (Apigee pattern), not bespoke point-to-point builds.
5. **Model unit economics from day one** — Tonik's profitability is an outcome of design choices (LDR 82%, AI cost-to-serve, 99% lending revenue), not luck; build the P&L model before the code.
6. **Plan for the regulator's next move** — the BSP's moratorium-then-reopening cycle shows licence scarcity is temporary; your competitive window closes when the cap lifts.

---

## 8. Future Outlook

### 8.1 Product Expansion

Tonik's stated next-phase priorities (May 2026 release): **scaling employer-channel lending via Tendo**, **expanding the merchant installment network**, and **deepening revolving credit products** (repeat-borrower lines) to accelerate the lifetime-value flywheel (net LTV/CAC 23×). Natural extensions beyond credit:

- **Wealth:** savings-adjacent investment products (the Tendo Stash is already a savings/investment hybrid for employees); a full wealth stack would mirror GoTyme/Maya's moves but Tonik has made no public announcement (flagged).
- **Insurance:** distribution partnerships have been a stated ambition since the Series A era; no live verified partnership in this pass (flagged).
- **Cards:** a credit card is the obvious revolving-credit vehicle; Tonik currently offers debit only (flagged as expected direction, not announced).

### 8.2 Geographic Expansion

Krasnov has long framed Tonik's ambition as "financial inclusion in Southeast Asia" (Finastra release language: "a vision to drive financial inclusion in Southeast Asia"). However:

- **No verified concrete expansion plan** (market, licence vehicle, timing) was found in this research pass. The Philippines remains the sole operating market.
- The **Tendo platform** (employer-channel lending) is the most portable asset — it is a channel technology that could extend to other SEA markets with similar salary-deduction cultures.
- Structural barriers: the BSP licence doesn't port; a new market means a new licence fight (or a partnership), new rails, and new thin-file data — the moat doesn't travel automatically.

### 8.3 The Profitability Path: 2026 and Beyond

- **Verified:** net cash-flow profitable Q1 2026; bank subsidiary IFRS-profitable Q1 2026; US$60M+ revenue run-rate (99% lending); 25–27% RAROC (Tonik-claimed, May 2026).
- **The task brief's "2026–2027 profitability target" is thus already achieved** — ahead of the brief's expectation — with the caveat that the claims are company-reported and unaudited in this pass (flagged).
- **Sustaining profitability** requires: loan growth (US$110M portfolio is tiny vs a US$50–100B credit gap), stable funding costs (deposit competition from GoTyme/Maya/MariBank), and cost-of-risk discipline as the book seasons into its first full credit cycle.
- **The strategic claim:** Tonik says it is "the only player that is both cleanly profitable and structurally positioned with a digital bank deposit license to scale into the $50–100 billion credit gap" — i.e., profitability is the *funding* for growth without external subsidy.

### 8.4 Consolidation in the PH Digital Bank Market

The Philippine digital bank market is heading into a shakeout (Fintech News PH, Feb 2026: "Digital Banks Face Profit Challenges As New Rivals Loom"):

- **The 2025 reopening** (4 new slots, max 10) ends the regulatory moat — new entrants (possibly regional banks, telco-wallet alliances, e-commerce giants) will contest the same underbanked segment.
- **Likely winners:** ecosystem banks with captive distribution (Maya via PayMaya wallet, MariBank via Shopee, GoTyme via Robinsons) and Tonik with its profitability + credit moat. 
- **Pressure points:** UNObank (deposit-rate model, thin credit) and UnionDigital (overlapping with parent UnionBank's franchise) are the most exposed to consolidation logic; OFBank is state-owned and will persist on mandate.
- **For Tonik:** consolidation is an opportunity (buying loan books, partner channels) and a threat (ecosystem rivals' cost advantages in acquisition).

### 8.5 Trends: AI and Embedded Finance

- **AI:** Tonik is already at the frontier of practical banking AI (5× ops efficiency, 75% autonomous service, AI underwriting, voice analysis). The next frontier: agentic service, AI-driven collections, hyper-personalised credit lines, and AI credit scoring at national-ID scale as PhilSys adoption matures. Regulatory context: the BSP's AI/AML expectations will formalise as usage scales.
- **Embedded finance:** Tendo (employer) and Shop Installment (merchant) are live embedded-finance rails; the programmable-banking pattern (see [programmable_business_bank_guide.md](programmable_business_bank_guide.md)) predicts the next wave: lending embedded in e-commerce checkout (Shopee/Lazada partnerships), remittance-linked credit, and salary-advance products for the gig economy.
- **The synthesis:** Tonik's 2026–2028 trajectory tests whether a **standalone, credit-led, AI-native digital bank** can out-earn ecosystem banks in an emerging market — the answer will shape how the next generation of emerging-market neobanks are architected and funded.

---

## 9. Glossary

- **Apigee** — Google Cloud's API management platform; Tonik's API gateway for app/partner integrations.
- **Bangko Sentral ng Pilipinas (BSP)** — the Philippines' central bank and banking regulator; issues bank licences, including digital bank licences.
- **BDO / BPI / Metrobank / PNB** — the Philippines' largest universal banks (SM, Ayala, GT Capital groups; PNB state-related); the incumbent competitive backdrop.
- **BSP Digital Banking License No. 001** — Tonik's licence designation as the first standalone/private digital bank licensed by the BSP.
- **Circular No. 1105** — the BSP's December 2020 guidelines creating the digital bank bank-category.
- **CIMB** — Malaysian banking group whose Philippine subsidiary operates digital-only (OCTO app) under a commercial bank licence — a "near-bank," not a digital-bank licensee.
- **Cloud-native** — software designed for cloud deployment (containers, managed services, elastic scaling, CI/CD) rather than ported from on-premises.
- **Composable banking** — assembling a bank from best-of-breed, replaceable components (core, payments, cards, KYC) rather than a monolithic vendor suite.
- **Cost-to-serve** — the cost of serving one customer; Tonik cut unit operational service costs 5× since 2021 via cloud + AI.
- **Credit Builder Loan** — Tonik's flagship unsecured personal loan (up to ₱50,000), positioned as credit-building.
- **Digital bank** — BSP-defined category: a bank processing products end-to-end digitally with no physical presence; a specific licence type in the Philippines.
- **eKYC** — electronic know-your-customer: digital identity verification (ID OCR, liveness checks) enabling branchless onboarding.
- **Embedded finance** — financial products delivered inside non-financial experiences (payroll, merchant checkout); Tonik's Tendo and Shop Installment channels.
- **Essence (Finastra)** — cloud-native core banking platform (Fusion Essence lineage); **Tonik's verified core banking system** (not Thought Machine Vault).
- **Finastra** — global fintech software vendor (FusionBanking/Essence, lending, payments); Tonik's core-banking partner.
- **GCash / Mynt** — the Philippines' dominant e-wallet (Globe + Ayala-backed); an e-money issuer, not a bank; Tonik's biggest payments competitor.
- **Google Cloud** — Tonik's cloud platform (Apigee API gateway, compute, analytics, AI).
- **GoTyme** — Gokongwei + Tyme (South Africa) digital bank; the ecosystem-retail rival (5.1M customers, Dec 2024).
- **Group Stash** — Tonik's shared social-savings product (multiple members saving toward a common goal).
- **InstaPay** — the Philippines' 24/7 real-time low-value payment rails (BancNet/Vocalink-built; ₱500k per-transaction cap).
- **MariBank Philippines** — Sea Group's Philippine digital bank (ex-SeaBank under a rural bank licence); upgraded to digital bank licence July 2026.
- **Maya (Bank + wallet)** — PLDT/Voyager MVP group's digital bank (licence Sep 2021) and e-wallet; the largest digital bank by assets.
- **NSFI** — National Strategy for Financial Inclusion: the BSP-coordinated national inclusion framework.
- **OFW** — Overseas Filipino Worker; remittance senders whose inflows fund the Philippine economy (US$38.3B in 2024).
- **OwnBank** — Rural Bank of Cavite City operating digitally; a rural-bank licensee, **not** a digital-bank licensee.
- **PDIC** — Philippine Deposit Insurance Corporation; insures deposits, now up to **₱1,000,000** per depositor (raised from ₱500,000, 2025).
- **Peak XV (Sequoia India)** — the VC firm (ex-Sequoia Capital India) that led Tonik's US$21M Series A.
- **PESONet** — the Philippines' batch, higher-value payment rails (no per-transaction cap; salary/bulk payments).
- **QR Ph** — the BSP's national QR payment standard.
- **RAROC** — Risk-Adjusted Return on Capital; Tonik reports 25–27% lending RAROC.
- **Remittance** — money sent home by OFWs; the Philippines received US$38.34B in personal remittances (2024).
- **SeaBank** — Sea Group's Philippine digital operator before its 2025 rename to MariBank Philippines.
- **Shop Installment Loan** — Tonik's merchant-embedded point-of-sale instalment loan product.
- **Stash** — Tonik's high-yield goal-based savings product.
- **Tendo** — Tonik's employer-channel platform (salary-deduction lending and savings; Tendo Stash).
- **Thought Machine Vault** — cloud-native core platform; **often incorrectly attributed to Tonik** — Tonik actually runs Finastra Essence.
- **Time Deposit** — fixed-term deposit; Tonik's launch hero product at up to 6% p.a.
- **Underbanked / unbanked** — adults without (or with limited) formal financial access; 44% of Filipino adults were unbanked at the 2021 FIS.
- **UNObank** — UNOAsia's Philippine digital bank (savings-rate-led model).
- **UnionDigital** — UnionBank's licensed digital bank subsidiary.

---

## 10. References

**Primary / authoritative:**

1. Tonik — "Tonik Becomes the First Standalone Digital Bank in the Philippines to Achieve Profitability" (tonikbank.com/news, May 2026): BSP Digital Banking License No. 001; Q1 2026 net cash-flow profitability; IFRS profitability of the bank subsidiary; loan portfolio US$110M (+2.3× YoY); revenue run-rate US$60M+ (99% lending); NIM 51%; RAROC 25%; LDR 82%; net LTV/CAC 23×; launch "February 2021"; funding-cost advantage (3–6% vs 15%+); Tendo/merchant/revolving priorities; offices (Pasig City, Singapore, Chennai); PDIC ₱1M.
2. Finastra press release, 8 May 2025 — "Tonik achieves record growth and innovation milestones with Finastra's Essence software": **Essence core banking confirmed**; 1M+ cumulative loans; loan portfolio ×7 in two years; unit costs ÷5; AI efficiency 5×; 90%/75% service automation; RBI 2025 "Best Core Banking System Initiative" (with Finastra); IBSi Digital Banking Awards 2025 CX winner (Asia); Agile Streams engagement.
3. Google Cloud — Tonik customer case study (cloud.google.com/customers/tonik): **Google Cloud + Apigee**; app-based onboarding.
4. BSP — Published financial statements of digital banks (bsp.gov.ph): Maya Bank ₱77.66B; MariBank ₱68.20B; GoTyme ₱50.11B; UNObank ₱11.94B; UnionDigital ₱11.83B; **Tonik ₱8.25B**; OFBank ₱5.55B (end-2025; cited via Wikipedia tables, July 2026).
5. Tonik — "Tonik becomes PH's first neobank to secure digital bank license" (tonikbank.com/news, June 2021): digital bank licence; March 2021 public launch; ₱1B deposits in <1 month.
6. Tonik — "Tonik raises US$21M Series A led by Sequoia India and Point72" (12 June 2020); The Asian Banker reprint.
7. Tonik — "Tonik Hits 1M customers" (PR Newswire APAC, 6 June 2023): 1M customer milestone.
8. Wikipedia — "Tonik (bank)" (retrieved Aug 2026): founding/launch dates, licence story, backers, offices, rural bank licence Dec 2019, pilot Q4 2020, ₱3.5B deposits Aug 2021, pre-Series B US$17M, 6% launch rates; infobox "founded 2021-03-18" (launch date) flagged.
9. Wikipedia — "Digital banks in the Philippines" (retrieved Aug 2026): Circular No. 1105 (Dec 2020); six licences 2021; 3-year moratorium (Aug 2021); cap of six (Oct 2021); reopening Jan 2025 (four slots, max ten); **7th licence — MariBank upgrade, July 2026**; 2019 BSP "70% unbanked"; 2023 targets; CIMB/NetBank/OwnBank/Salmon/DiskarTech/Komo as non-digital-bank-licensed operators.
10. BSP Financial Inclusion Surveys — 2017 (22.6% banked), 2019 (~29% banked), 2021 (56% account ownership / 44% unbanked) — via PNA, Philstar, MoneyMax, FINEX (2019→2021 improvement).
11. BSP remittance data — personal remittances 2024 US$38.34B (+3%, record; fintechnews.ph, Manila Standard); cash remittances 2024 US$34.49B; **cash remittances 2025 US$35.63B record** (Tribune, Feb 2026); Middle East 17.77% share (Inquirer).

**Press and industry (secondary):**

12. The CEO Magazine — "The Bank of the Future: Greg Krasnov" (27 Sep 2021): PH third-largest SEA economy; 71% unbanked of 72M adults; BSP rules Nov 2020; "biggest credit gap in SEA"; Azure fragment (flagged).
13. Finastra — Greg Krasnov bio: FORUM, Credolab, FLOW, Solarhome, AsiaKredit; Platinum Bank; first business at 18; 10 years PE/LBO at Bank of America (London) and Innova Capital (Warsaw).
14. FinTech Magazine — "The Banker who brought Digital Banking to the Philippines" (20 Apr 2022): founder career detail.
15. The Official Board — Greg Krasnov bio: Platinum Bank CEO; Innova Capital MD; **joined Tonik 2018** (corroborates the 2018 founding date).
16. TechNode Global / DealStreetAsia / Verdict — "Mizuho Bank leads Tonik's US$131M Series B" (9–10 Feb 2022).
17. finews.asia — "Neobank Tonik Raises Funds Following Philippines Launch" (11 May 2021): US$17M pre-Series B, iGlobe-led; total >US$44M at the time.
18. Philippine Star — Tonik deposits ₱3.5B (22 Aug 2021); ₱1B deposits milestone (May 2021); Globe myBusiness partnership.
19. BusinessMirror (28 Apr 2021) — ₱1B retail deposits in <1 month; ABS-CBN News (18 Mar 2021) — time deposit rates up to 6%; Philippine Daily Inquirer (20 Mar 2021) — "staggering" rates.
20. BusinessWorld — "BSP releases guidelines for establishment of digital banks" (4 Dec 2020); "BSP grants license to Tonik Digital Bank" (16 Dec 2020 — rural bank licence Dec 2019 context); "BSP caps digital bank licenses at six" (5 Oct 2021).
21. Rappler — "As digital banks gain popularity, will the big banks follow?" — six 2021 licences: UNObank, UnionDigital, GoTyme, OFBank, Tonik, Maya.
22. BusinessWorld — "GoTyme Bank reaches 5.1 million customers, ₱24 billion in deposits" (12 Dec 2024).
23. Fintech News Philippines — "Philippines Digital Banks Face Profit Challenges As New Rivals Loom" (11 Feb 2026); "Digital Banks in the Philippines (2026)" — six-bank taxonomy; reported Tonik US$12M raise (flagged).
24. Manila Bulletin — "A revolutionary digital-only bank launches in the Philippines" (19 Mar 2021) — launch date anchor.
25. Philippine Daily Inquirer — "In surprise move, BSP sets 3-year moratorium on new digital banking licenses" (19 Aug 2021); "Bangko Sentral to license 4 new digital banks" (9 Aug 2024); "Philippines gets 7th digital bank as MariBank upgrades license" (23 Jul 2026).
26. Greg Krasnov LinkedIn (retrieved Aug 2026): Q1 2026 US$60M+ revenue run-rate; 27% RAROC; net cash-flow profitable Q1 2026; "70% of Filipinos have never had a bank loan"; "lending-first, AI-driven bank."
27. FinTech.ph listing — Tonik founded 2018; PDI/Statista "Best Customer Service for Digital Banks 2023."

**Sibling repository guides (cross-referenced):**

- [core_banking_systems_guide.md](core_banking_systems_guide.md) — composable banking framework; Finastra/Essence profile; digital bank case.
- [temenos_guide.md](temenos_guide.md) — core vendor comparison table (Finastra vs Temenos vs Thought Machine).
- [us_bank_core_systems_guide.md](us_bank_core_systems_guide.md) — core systems vendor table (Thought Machine Vault profile).
- [financial_infrastructure_guide.md](financial_infrastructure_guide.md) — payment rails taxonomy (InstaPay/PESONet context).
- [mojaloop_guide.md](mojaloop_guide.md) — InstaPay design notes (BancNet/Vocalink-built IPS comparator).
- [green_link_digital_bank_guide.md](green_link_digital_bank_guide.md) — the SG digital bank licence history (contrast with the BSP framework).
- [dbs_bank_guide.md](dbs_bank_guide.md) — the incumbent in-house-core comparator (build vs buy).
- [programmable_business_bank_guide.md](programmable_business_bank_guide.md) — embedded/programmable banking patterns (Tendo, Shop Installment).

---

*End of guide. Verification discipline per repository convention: claims marked "(flagged)" were not independently confirmed in the August 2026 research pass; primary anchors are Tonik's own announcements, Finastra/Google Cloud vendor materials, BSP publications and the Philippine business press. The two most consequential corrections to common narratives: (1) Tonik's core banking is **Finastra Essence**, not Thought Machine Vault; (2) Tonik holds the **first private/standalone** digital bank licence (No. 001) — the state-owned Overseas Filipino Bank held the very first licence (March 2021).*
