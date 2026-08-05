# Airwallex: Global Payments and Financial Infrastructure — A Comprehensive Guide

**Cross-Border Payments, Multi-Currency Accounts, Cards, FX, and Embedded Finance — the Australian-Founded Fintech That Grew from Melbourne to a US$11B Dual-HQ Platform**

> **Author:** Jack Liu Shurui — Solution Architect at Crédit Agricole CIB, Singapore  
> **Context:** Banking Innovation / Fintech Infrastructure — Cross-Border Payments, Multi-Currency Accounts, Virtual Accounts, Embedded Finance, FX, Licensing, Competitive Landscape  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** August 2026

---

## Table of Contents

1. [What Is Airwallex?](#1-what-is-airwallex)
   - 1.1 [Definition](#11-definition)
   - 1.2 [The Founding Story](#12-the-founding-story)
   - 1.3 [The Co-Founders](#13-the-co-founders)
   - 1.4 [The Mission and Positioning](#14-the-mission-and-positioning)
2. [Company Status: From Melbourne Startup to US$11B Platform](#2-company-status-from-melbourne-startup-to-us11b-platform)
   - 2.1 [The Valuation Timeline](#21-the-valuation-timeline)
   - 2.2 [Funding History and Investors](#22-funding-history-and-investors)
   - 2.3 [Headquarters: Melbourne → Singapore → San Francisco](#23-headquarters-melbourne--singapore--san-francisco)
   - 2.4 [Scale: Revenue, Volume, Customers, People](#24-scale-revenue-volume-customers-people)
   - 2.5 [Profitability](#25-profitability)
   - 2.6 [The Path to IPO](#26-the-path-to-ipo)
   - 2.7 [The 2025–2026 Regulatory and Political Scrutiny](#27-the-20252026-regulatory-and-political-scrutiny)
3. [The Product Portfolio at a Glance](#3-the-product-portfolio-at-a-glance)
4. [Global Accounts: Multi-Currency Business Accounts](#4-global-accounts-multi-currency-business-accounts)
   - 4.1 [The Multi-Currency Wallet](#41-the-multi-currency-wallet)
   - 4.2 [Local Account Details (Local Receiving Accounts)](#42-local-account-details-local-receiving-accounts)
   - 4.3 [Virtual Accounts for Collections and Reconciliation](#43-virtual-accounts-for-collections-and-reconciliation)
   - 4.4 [Account Features: Balances, FX, Interest](#44-account-features-balances-fx-interest)
   - 4.5 [Airwallex Yield](#45-airwallex-yield)
5. [Payments: Transfers, Mass Payments, Payment Links, and Acceptance](#5-payments-transfers-mass-payments-payment-links-and-acceptance)
   - 5.1 [International Transfers (Global Transfers)](#51-international-transfers-global-transfers)
   - 5.2 [Mass Payments (Batch Payments)](#52-mass-payments-batch-payments)
   - 5.3 [Payment Links (Request Payments)](#53-payment-links-request-payments)
   - 5.4 [Payment Acceptance (Acquiring)](#54-payment-acceptance-acquiring)
6. [Cards: Corporate Spend and Programmable Issuing](#6-cards-corporate-spend-and-programmable-issuing)
   - 6.1 [The Corporate Card](#61-the-corporate-card)
   - 6.2 [Spend Controls](#62-spend-controls)
   - 6.3 [Programmable Cards for Platforms](#63-programmable-cards-for-platforms)
7. [Treasury and FX](#7-treasury-and-fx)
   - 7.1 [FX at Interbank Plus a Margin](#71-fx-at-interbank-plus-a-margin)
   - 7.2 [FX Risk Management](#72-fx-risk-management)
   - 7.3 [FX at the Point of Payment](#73-fx-at-the-point-of-payment)
8. [Airwallex for Platforms: Embedded Finance and BaaS](#8-airwallex-for-platforms-embedded-finance-and-baas)
   - 8.1 [The Platform Value Proposition](#81-the-platform-value-proposition)
   - 8.2 [The API Suite](#82-the-api-suite)
   - 8.3 [The Banking-as-a-Service Angle](#83-the-banking-as-a-service-angle)
   - 8.4 [The Developer Platform](#84-the-developer-platform)
9. [Integrations and Ecosystem](#9-integrations-and-ecosystem)
   - 9.1 [E-Commerce Platforms](#91-e-commerce-platforms)
   - 9.2 [ERP and Accounting Software](#92-erp-and-accounting-software)
   - 9.3 [Payments Platforms and the Shopify Partnership](#93-payments-platforms-and-the-shopify-partnership)
10. [Business Model and Customer Segments](#10-business-model-and-customer-segments)
    - 10.1 [Revenue Model](#101-revenue-model)
    - 10.2 [Customer Segments](#102-customer-segments)
    - 10.3 [Geographic Focus](#103-geographic-focus)
    - 10.4 [Pricing and Plans](#104-pricing-and-plans)
11. [Technology and Architecture](#11-technology-and-architecture)
    - 11.1 [One Platform, One API](#111-one-platform-one-api)
    - 11.2 [The Proprietary Core](#112-the-proprietary-core)
    - 11.3 [The Global Network: Local Rails and Clearing](#113-the-global-network-local-rails-and-clearing)
    - 11.4 [Banking Partners and Correspondent Network](#114-banking-partners-and-correspondent-network)
    - 11.5 [Compliance Architecture](#115-compliance-architecture)
12. [Licensing and Regulatory Footprint](#12-licensing-and-regulatory-footprint)
    - 12.1 [The License Map](#121-the-license-map)
    - 12.2 [The Licensing Strategy: License Acquisition vs Bank Partnership](#122-the-licensing-strategy-license-acquisition-vs-bank-partnership)
13. [Competitive Landscape](#13-competitive-landscape)
    - 13.1 [vs Traditional Banks](#131-vs-traditional-banks)
    - 13.2 [vs Payments Fintechs](#132-vs-payments-fintechs)
    - 13.3 [vs Business Banking Fintechs](#133-vs-business-banking-fintechs)
    - 13.4 [The Comparison Table](#134-the-comparison-table)
    - 13.5 [Airwallex Differentiation](#135-airwallex-differentiation)
14. [Banking Context](#14-banking-context)
    - 14.1 [A Bank-Adjacent Fintech, Not a Bank](#141-a-bank-adjacent-fintech-not-a-bank)
    - 14.2 [Airwallex vs Digital Banks](#142-airwallex-vs-digital-banks)
    - 14.3 [Airwallex in the Singapore Ecosystem](#143-airwallex-in-the-singapore-ecosystem)
    - 14.4 [Airwallex for Corporate Treasury](#144-airwallex-for-corporate-treasury)
15. [The Architect's View: Integrating Airwallex](#15-the-architects-view-integrating-airwallex)
    - 15.1 [Integration Patterns](#151-integration-patterns)
    - 15.2 [Authentication and Security](#152-authentication-and-security)
    - 15.3 [Webhooks, Idempotency, and Reconciliation](#153-webhooks-idempotency-and-reconciliation)
    - 15.4 [Data Residency](#154-data-residency)
    - 15.5 [Risk and Concentration](#155-risk-and-concentration)
    - 15.6 [Vendor Selection Framework](#156-vendor-selection-framework)
16. [Worked Example: A Singapore E-Commerce Business Pays a Chinese Supplier](#16-worked-example-a-singapore-e-commerce-business-pays-a-chinese-supplier)
17. [Airwallex in Singapore and Asia](#17-airwallex-in-singapore-and-asia)
    - 17.1 [Singapore Operations](#171-singapore-operations)
    - 17.2 [The APAC Strategy and the China Corridor](#172-the-apac-strategy-and-the-china-corridor)
    - 17.3 [Competition in Asia](#173-competition-in-asia)
    - 17.4 [The Regulatory Environment](#174-the-regulatory-environment)
18. [The Future: 2026 and Beyond](#18-the-future-2026-and-beyond)
    - 18.1 [IPO Path and Profitability](#181-ipo-path-and-profitability)
    - 18.2 [Product Expansion: Toward a Full-Stack Financial Platform](#182-product-expansion-toward-a-full-stack-financial-platform)
    - 18.3 [Embedded Finance Growth](#183-embedded-finance-growth)
    - 18.4 [AI in Airwallex](#184-ai-in-airwallex)
    - 18.5 [Global Expansion](#185-global-expansion)
19. [Glossary](#19-glossary)
20. [Sources and Notes](#20-sources-and-notes)

---

## 1. What Is Airwallex?

### 1.1 Definition

Airwallex is a global financial technology company that provides cross-border payments and financial infrastructure for businesses. Founded in 2015 in Melbourne, Australia, it is now dual-headquartered in Singapore and San Francisco, and operates as one of the world's largest privately held fintech platforms, valued at US$11 billion as of June 2026.

The company builds and operates a licensed, API-first payments platform that lets businesses:

- **Hold, receive, and pay** in multiple currencies through multi-currency business accounts ("Global Accounts") with local receiving details in major markets;
- **Move money across borders** at near-interbank FX rates over local payment rails instead of expensive SWIFT correspondent chains;
- **Issue corporate cards** (physical and virtual) with programmable spend controls;
- **Accept payments** online (cards, wallets, and 160+ local payment methods);
- **Embed finance** into their own products through a full API suite — accounts, payments, cards, FX, and payouts — for SaaS platforms, marketplaces, and enterprises.

Airwallex describes itself not as a bank but as the *financial infrastructure* layer for global business — "the banking platform building the infrastructure that powers every business to be a global business." In the taxonomy of this repository, Airwallex is a **licensed payments institution / payment-adjacent fintech** rather than a licensed bank: it does not take deposits in the banking sense, does not lend, and is not covered by deposit insurance schemes; customer funds are held with partner banks and safeguarded under e-money and payments regulation. See [programmable_business_bank_guide.md](programmable_business_bank_guide.md) where Airwallex appears among the business-banking fintechs that built programmability from day one, and [payments_hub_guide.md](payments_hub_guide.md) where it appears as a SaaS payment orchestration / cross-border platform.

### 1.2 The Founding Story

The origin story is a Melbourne coffee shop. Jack Zhang, a Melbourne-based entrepreneur, found it absurdly expensive and slow to pay a Chinese supplier for coffee cups — the FX margin, SWIFT correspondent fees, and settlement delays of the traditional cross-border banking chain ate into a small business's economics. Zhang and his then-business partner Max Li concluded that the problem was not their bank but the *infrastructure*: a decades-old correspondent banking network built for large corporate treasury desks, not for SMEs and digital businesses.

Their response was to build a payments company that treats the world's payment rails as one programmable network: obtain local clearing access and banking partnerships in every major market, hold funds locally, convert currency at interbank-level rates with a thin margin, and expose the whole engine through clean APIs. The founding insight — that a technology company could assemble its own global clearing network more cheaply and more transparently than banks pass money through each other — remains the core of Airwallex's design.

### 1.3 The Co-Founders

Airwallex was created by **five co-founders** (note: several early media reports and the task brief refer to "Xijing Wang"; the documented name is **Xijing Dai**, also known as Jacob Dai):

| Co-Founder | Role | Background |
|---|---|---|
| **Jack Zhang** | Co-Founder & CEO | The driving force; also invests via Capital 49; the public face of the company's strategy and AI vision |
| **Lucy Liu** | Co-Founder & President | Early investor in the company (famously backed it with her own savings); leads commercial/customer-facing strategy; a prominent figure in APAC fintech |
| **Max Li** | Co-Founder | University of Melbourne network; long-time operator (earlier COO) |
| **Xijing (Jacob) Dai** | Co-Founder & Group CTO | The engineering anchor; leads the platform and infrastructure organization |
| **Ki-lok Wong** | Co-Founder | Melbourne University network; founding CTO who built the early payments engine |

All five remain associated with the company (some in executive roles, others more removed from day-to-day operations as the company scaled to ~2,800 employees). The founding team's composition — Chinese-Australian founders who experienced the pain of the Australia–China trade corridor — shaped Airwallex's early strategic focus on APAC cross-border flows, which remains its differentiator.

### 1.4 The Mission and Positioning

Airwallex's long-standing mission is **"to empower businesses to operate anywhere in the world."** The company has since sharpened the phrasing to "operate anywhere, anytime," with AI as a stated lever — the 2025 positioning is: *"The future of finance won't be defined by borders. Airwallex isn't either... connecting banking, payments, software, and AI."*

Three positioning pillars recur across the company's materials:

1. **Cross-border-first design.** Everything — accounts, cards, FX, payouts — is built for businesses that operate across multiple currencies, rather than a domestic bank account with international transfers bolted on.
2. **One platform, one API.** A single integration (REST APIs, webhooks, SDKs) over a global network of licensed entities, banking partners, and local rails, replacing the patchwork of bank accounts, SWIFT, and payment gateways a global business would otherwise assemble.
3. **Infrastructure, not a bank.** Airwallex wants to be the payments and financial *layer* underneath businesses and other platforms — including being embedded inside other companies' products (embedded finance), rather than competing with its customers.

---

## 2. Company Status: From Melbourne Startup to US$11B Platform

### 2.1 The Valuation Timeline

Airwallex became Australia's fastest-growing tech unicorn in 2019 (crossing US$1B valuation after its Series C). Since then the valuation has climbed through every funding round:

| Round | Date | Amount | Valuation | Notes |
|---|---|---|---|---|
| Seed / early | 2015–2017 | ~US$10M+ (reported) | — | Early backers included 1835i (ANZ's fintech venture fund) and Australian angels |
| Series A | 2017 | ~US$10M | — | Tencent participation put Airwallex on the map in Asia |
| Series B | Jun 2018 | US$80M | — | Led by Sequoia Capital China (now HongShan) with Tencent and DST Global |
| Series C | Mar 2019 | US$100M | US$1B+ | Led by DST Global; made Airwallex a unicorn |
| Series D | 2021 | US$100M | — | Led by Greenoaks Capital; 1835i and ANZi (ANZ's venture arm) among existing backers |
| Series E | Oct 2021 | US$200M | US$5.5B | Led by Lone Pine Capital; Salesforce Ventures, G Squared, 1835i, DST, Sequoia Capital China, Tencent |
| Series E2 | Oct 2022 | US$100M | **US$5.6B** | "Same valuation" extension; Square Peg, Salesforce Ventures, HongShan (ex-Sequoia China), Lone Pine, Hermitage; later disclosed Hostplus (Australian super fund) and a North American pension fund |
| Series F | May 2025 | US$300M | US$6.2B | Growth round ahead of US expansion |
| Series G | Dec 2025 | US$330M | **US$8B** | ~30% step-up; total funding reached ~US$1.5B |
| **Series H** | **Jun 2026** | **US$320M** | **US$11B** | Led by Addition; Baillie Gifford, Hummingbird, QED Investors, T. Rowe Price; explicitly tied to the "AI-agent-run finance" strategy |

**Current status (August 2026):** US$11B post-money valuation after the June 2026 Series H — one of the largest private fintech valuations globally. Total capital raised is approximately **US$1.8 billion**.

### 2.2 Funding History and Investors

The investor base spans Silicon Valley VCs, Chinese strategic investors, US hedge funds, and Australian institutions — which is itself a source of both strength (capital depth) and scrutiny (see §2.7):

- **Venture / growth:** Sequoia Capital China (now HongShan), DST Global, Greenoaks, G Squared, Hermitage Capital, Square Peg (Australian VC), 1835i (ANZ's fintech fund), ANZi (ANZ's venture arm), Addition, Baillie Gifford, Hummingbird, QED Investors, T. Rowe Price.
- **Strategic / corporate:** Tencent (a long-standing strategic investor and a signal of the China corridor strategy), Salesforce Ventures.
- **Institutional / pension:** Hostplus (Australian industry superannuation fund), a North American pension fund.

Tencent's presence matters in two ways: it validates Airwallex's China strategy (RMB flows, CIPS access), and it is cited in US national-security scrutiny of the company (see §2.7).

### 2.3 Headquarters: Melbourne → Singapore → San Francisco

- **Founded:** Melbourne, Australia (2015). Melbourne remains a major engineering and APAC hub.
- **Global HQ → Singapore:** Airwallex relocated its global headquarters to Singapore, opening its HQ office in February 2023 in **Guoco Tower, Tanjong Pagar** — the tallest building in Singapore — in the heart of the city-state's financial district. Singapore is the company's largest employee hub and the seat of its MAS licensing.
- **Second global HQ → San Francisco:** In December 2025, alongside the Series G and a US$1B+ US expansion plan (2026–2029), Airwallex designated **San Francisco as a second global headquarters**, making the company dual-headquartered (Singapore + San Francisco). The company is formally domiciled in the Cayman Islands, a standard structure for global VC-backed companies.

This three-city arc — Melbourne (origin/engineering), Singapore (regulated APAC HQ), San Francisco (US growth HQ) — tracks the company's market strategy: APAC-first, then a determined push into the US market.

### 2.4 Scale: Revenue, Volume, Customers, People

As of mid-2026 (all figures from company disclosures, press, and analyst estimates such as Sacra):

- **Annualized revenue:** ~US$720M (March 2025, +90% YoY); **>US$1B ARR** reached by December 2025.
- **Annualized transaction volume (TPV):** exceeded US$100B (August 2024), US$130B (March 2025), and **US$200B+** (mid-2025).
- **Customers:** ~150,000 businesses worldwide (2025), with customer count growing ~50% in 2024; platform/embedded-finance customers drive ~45% of TPV despite being a small share of accounts.
- **People:** ~2,000+ employees across 26 offices (December 2025); ~2,800 by end-2025 (Revelio Labs estimate); the company said it expected headcount to grow >50% through 2026.
- **Reach:** services accessible in ~200 countries; **80+ licenses and permits** held by end-2025; 92% of transfers arrive within the same day.

### 2.5 Profitability

Airwallex historically reinvested essentially every dollar of revenue into infrastructure and growth. The company reported reaching **EBITDA profitability in Q4 2025**, confirmed in its 2025 End-of-Year Mission Update (published February 2026) — framed explicitly as a consequence of the infrastructure strategy paying off ("our execution is strong enough that we reached EBITDA profitability in Q4 2025, while still investing aggressively in growth"). By mid-2026 the company was described as having "surpassed US$1B ARR and achieved profitability" (EBITDA basis). Note the distinction: **EBITDA-positive is not GAAP net income** — Airwallex has not claimed net profitability, and its US$10B ARR ambition implies years of heavy reinvestment. It is now, however, profitable enough at the operating level to fund growth and to credibly approach an IPO.

### 2.6 The Path to IPO

IPO speculation has accompanied Airwallex since ~2023:

- **2024:** CEO Jack Zhang said the company was preparing for an IPO "within the next two years" (i.e., ~2026), fueling reports of a 2026 listing.
- **January 2026:** Zhang walked that back explicitly: **no IPO before 2028**, pushing back against persistent media references to a 2026 listing.
- **June 2026 (Series H):** the company said it aims to be **IPO-ready within a year** (i.e., ~2027), with the Series H framed as the last private round before a potential listing.

A realistic read as of August 2026: IPO readiness target ~2027, with an actual listing more likely 2027–2028. The listing venue is unconfirmed (Hong Kong and the US are the frequently speculated options; a Singapore listing has also been floated given the HQ). The company's profitability, >US$1B ARR, and US$11B valuation make it one of the most anticipated fintech listings in the pipeline.

### 2.7 The 2025–2026 Regulatory and Political Scrutiny

The single most consequential company-level risk in the current period is US political scrutiny over China ties:

- **December 2025:** prominent US VC **Keith Rabois publicly alleged a "Chinese backdoor" to US customer data**, a claim Airwallex strongly denies. The episode triggered a crisis-communications response and press reports that Airwallex relocated some staff and international roles out of China.
- **June 2026:** **US Senator Tom Cotton (R-Ark.) wrote to Treasury Secretary Scott Bessent urging an investigation** of Airwallex's ties to the Chinese government and potential divestment pressure, citing Tencent's stake, China-based operations, and the Chinese payment license.
- Airwallex's public response emphasizes that it is a Cayman-domiciled, globally licensed company, that customer funds and data are held regionally with regulated partners (including in the US via partner banks), and that it has moved international-facing roles out of China to remove any ambiguity.

This is a live, evolving issue and should be tracked by any architect or procurement function evaluating Airwallex as a vendor (see §15.5 on risk). It parallels the broader 2025–2026 US scrutiny of Chinese-linked financial technology (TikTok, and payment firms with China exposure) and is the main overhang on the stock's IPO narrative.

---

## 3. The Product Portfolio at a Glance

Airwallex organizes its products into six families, all accessible from a single web/mobile app and a single API surface:

| # | Product family | What it does | Who it is for |
|---|---|---|---|
| 1 | **Global Accounts** | Multi-currency business accounts with local receiving details, virtual accounts, FX conversion, and Yield on balances | SMEs and digital businesses needing a cross-border operating account |
| 2 | **Payments** | International transfers (200+ countries), mass/batch payments (payroll, suppliers), payment links, and online payment acceptance (acquiring) | Finance teams, e-commerce merchants, platforms |
| 3 | **Cards** | Multi-currency Visa corporate cards (physical + virtual) with programmable spend controls and expense management | Businesses managing employee/team spend |
| 4 | **Treasury & FX** | Interbank-based FX conversion, FX API (quotes/orders), multi-currency cash management | CFOs/treasurers, platforms needing embedded FX |
| 5 | **Airwallex for Platforms** | Embedded finance: accounts, payments, cards, FX, and payouts exposed as APIs for SaaS platforms and marketplaces | Marketplaces, payroll platforms, vertical SaaS, enterprises |
| 6 | **Integrations & apps** | Connectors to e-commerce (Shopify, WooCommerce, Magento), accounting (Xero, QuickBooks, NetSuite, Sage), and partner ecosystems | Non-developer businesses and ops teams |

Sections 4–9 go through each family in depth.

---

## 4. Global Accounts: Multi-Currency Business Accounts

### 4.1 The Multi-Currency Wallet

The **Global Account** is Airwallex's core product: a single business account that can hold, receive, convert, and pay in **60+ currencies** (the company markets "hold, receive, and spend in 60+ currencies"; the exact tradeable currency list is a subset of ~60). Funds sit in segregated accounts at partner banks; Airwallex maintains the multi-currency ledger. Key properties:

- Open in minutes with automated KYB onboarding (no branch, no paperwork);
- One login/app/API across all currencies — no per-country bank relationships;
- Balance can be converted between any supported currencies at interbank + margin;
- Businesses can hold balances in currencies they *receive* (USD, EUR, GBP, AUD, SGD, HKD, CNY, JPY, CAD, NZD, and more) to avoid forced conversion.

### 4.2 Local Account Details (Local Receiving Accounts)

For ~20+ major currencies, Airwallex provides **local account details** — "local accounts" — so customers can receive money as if they had a local bank account in that country, at local speed and with no international transfer fees on the receiving side:

- **USD** — US local account (ABA routing + account number) for ACH/domestic wire;
- **EUR** — IBAN (SEPA) via the Dutch EMI entity;
- **GBP** — UK sort code + account number (Faster Payments);
- **AUD** — Australian BSB + account number (NPP/Osko);
- **SGD** — local account for FAST/PayNow;
- **HKD** — local account (CHATS/FPS);
- **CNY** — local CNY receiving via the China license;
- Plus CAD, NZD, JPY and other major corridors.

Receiving locally collapses the traditional correspondent chain: a US customer paying a Singapore business over ACH arrives as a domestic US payment, lands in the Airwallex USD balance, and can be converted to SGD only when needed. The company's marketing states local receiving details are available in 20+ currencies across 60+ countries (some markets cap the number of global accounts per customer; a limit of ~10 global accounts per account has been reported in some markets).

### 4.3 Virtual Accounts for Collections and Reconciliation

Airwallex's local account details function as **virtual accounts** in the sense used throughout [programmable_business_bank_guide.md](programmable_business_bank_guide.md): each collection account is a unique identifier tied to a master account, so incoming payments can be matched to invoices, customers, or orders programmatically.

- Businesses can request multiple local account numbers (e.g., per customer, per invoice batch, per sales channel);
- Incoming payments are credited to the master balance with metadata (payer, reference) enabling automated reconciliation;
- The Airwallex API exposes account/transaction data so reconciliation can be fully automated (see §15.3 and the virtual-account reconciliation patterns in [programmable_business_bank_guide.md](programmable_business_bank_guide.md)).

This is the same architecture banks build with sub-ledgers (master account + virtual account mapping) — Airwallex ships it as a product.

### 4.4 Account Features: Balances, FX, Interest

- **Multi-currency balances:** one wallet, many currency sub-balances; zero or minimal cost to hold;
- **FX conversion:** anytime, at interbank + margin (see §7.1), including "FX at the point of payment" (convert and pay in one step);
- **Interest/Yield:** via the separate Airwallex Yield product (§4.5) — the account itself is a transaction account, not an interest-bearing deposit;
- **Free local transfers** in 20+ currencies (domestic rails), and cheap international transfers (92% same-day);
- **Sub-accounts / team access** with roles and approvals (up to multi-conditional approvals on higher tiers).

### 4.5 Airwallex Yield

**Airwallex Yield** (launched November 2023 in Australia, then Hong Kong and Singapore) lets businesses earn returns on idle **AUD, USD, GBP, EUR, and HKD** balances by investing them in **money market funds** (initially Goldman Sachs and Fullerton funds) managed by licensed custodians. Key facts:

- Funds are swept from the account into money-market instruments; returns passed to the customer (~4% APY quoted in 2026 marketing; ~US$200M attracted within months of launch);
- It is an investment product, not a deposit: no deposit insurance, capital not guaranteed, and subject to fund rules — an important distinction from a bank savings account;
- Strategically, Yield is Airwallex's answer to the "interest on balances" question: rather than pay interest on float (which a non-bank cannot do in most jurisdictions), it routes idle cash to regulated money funds and keeps the platform sticky.

---

## 5. Payments: Transfers, Mass Payments, Payment Links, and Acceptance

### 5.1 International Transfers (Global Transfers)

Airwallex's flagship payout capability:

- **Coverage:** transfers in **60+ currencies to 200+ countries**;
- **Speed:** 92% of funds arrive the same day — via local rails where possible rather than SWIFT;
- **Cost:** FX at interbank + margin (0.5% above interbank is the standard quoted rate on the free tier; lower on volume plans), no hidden correspondent fees, and **free local transfers** in 20+ currencies;
- **Rail routing:** the platform chooses the optimal local rail (FPS, SEPA, NPP, FAST, ACH, CIPS, etc.) or falls back to SWIFT/correspondent where no local option exists (see §11.3);
- **API-first:** transfers can be created individually or in bulk, with full status tracking (see §16).

The classic use case: a Singapore business pays a Chinese supplier in CNY at interbank + margin in minutes, where a bank would charge 2–4% FX margin plus SWIFT fees and take 2–5 days.

### 5.2 Mass Payments (Batch Payments)

The **Mass Payments** product handles high-volume payout runs:

- **Payroll:** pay employees in multiple countries and currencies from one batch file (many global payroll platforms use Airwallex as the underlying rail);
- **Supplier payments:** batch AP runs to hundreds of suppliers, each in its own currency;
- **Platform payouts:** marketplace seller settlements, affiliate/creator payouts, insurance claims, lending disbursements;
- **API:** the mass payout API accepts batch payloads, validates each payment, and reports per-payment status via webhooks — the same pattern as the batch/collections APIs discussed in [payments_hub_guide.md](payments_hub_guide.md);
- **No per-payment minimums on most plans**, transparent per-payment pricing.

### 5.3 Payment Links (Request Payments)

**Payment links** are hosted payment pages for requesting money without building a checkout:

- Create a link in the web app, via the API, or from within Xero invoices;
- The customer pays in their own currency and preferred payment method (cards, wallets, APMs);
- Funds settle directly into the business's Global Account in the merchant's currency;
- Use cases: invoices, deposits, manual sales, proforma payments, event/registration fees.

Payment links are Airwallex's version of "request to pay" — the payout/collection pair is symmetric, which is what makes the platform a full *payments* company rather than a transfer-only company.

### 5.4 Payment Acceptance (Acquiring)

**Airwallex Online Payments** is the acquiring/gateway product:

- **Card acceptance:** Visa, Mastercard, and Amex;
- **Digital wallets:** Apple Pay, Google Pay, Alipay, WeChat Pay (critical for APAC merchants);
- **Local payment methods (APMs):** 160+ — bank transfers, BNPL (e.g., Klarna/Afterpay in supported markets), local wallets, and scheme-specific methods by region;
- **Checkout options:** hosted checkout page, Drop-in/Embeddable components (frontend SDKs), and full REST API (PaymentIntents model — see §15.1);
- **Settlement:** multi-currency — settle card revenue directly into the Global Account in one of 8+ settlement currencies, avoiding double FX conversion (a stated differentiator vs single-currency gateways);
- **Plugins:** Shopify, WooCommerce, Magento, and others (§9);
- **Fees:** transparent card-processing pricing — e.g., ~1.3% + 20p (UK), 1.65% + US$0.30 domestic / 2.8–3.4% + US$0.30 international in some markets — with surcharging configuration available;
- **POS:** in-person payments capability launched in selected markets (2025–2026), including via the Japan entity (§12.1).

Acceptance + accounts + payouts in one platform is the "full loop" that distinguishes Airwallex from pure gateways (Stripe-style) and pure transfer companies (Wise-style).

---

## 6. Cards: Corporate Spend and Programmable Issuing

### 6.1 The Corporate Card

The **Airwallex Corporate Card** is a multi-currency **Visa** business debit card (physical and virtual), funded directly from Global Account balances:

- **Multi-currency:** spend directly in the currency held (no FX fee when spending from a matching balance; competitive FX otherwise);
- **Virtual + physical:** instant virtual cards on creation; physical cards for in-person spend;
- **Free to create** (up to a plan-dependent number; unlimited virtual cards on higher tiers / via API);
- **Expense management:** a built-in module for card expenses, receipts, reimbursements, and approval workflows;
- **Network and issuing:** Visa network; issuing via Airwallex's licensed entities and banking partners (e.g., in the US via partner banks, in Singapore via the MPI-licensed entity and card partners).

### 6.2 Spend Controls

Cards are programmable at the individual card level:

- **Per-card limits** (transaction amount, daily/monthly caps);
- **Merchant controls** (allowed/blocked MCCs, online-only, domestic-only);
- **Lifecycle controls:** freeze/unfreeze, auto-expiry, auto-destruction;
- **Approval flows:** multi-conditional approvals on advanced tiers (e.g., >S$5,000 requires two approvers);
- **Real-time visibility:** every transaction streams to the dashboard/API for reconciliation with accounting integrations (Xero, NetSuite, etc.).

This is the same "programmable card" pattern described in [programmable_business_bank_guide.md](programmable_business_bank_guide.md) — spend controls as code, with auto-creation/destruction of cards for contractors, campaigns, or per-employee pods.

### 6.3 Programmable Cards for Platforms

Through the Issuing API, platforms can **create, fund, limit, and destroy cards programmatically** for their own users:

- Neo-banks and expense platforms issuing cards to their customers on Airwallex's license/partner rails;
- Marketplaces issuing supplier/contractor cards with per-project limits;
- Marketing teams spinning up campaign cards that auto-expire.

Card issuing is a core pillar of the embedded-finance strategy (§8) and one of the fastest-growing revenue lines (cards + payments now drive >50% of gross profit per Sacra's 2025 analysis).

---

## 7. Treasury and FX

### 7.1 FX at Interbank Plus a Margin

Airwallex's FX engine is the heart of its economics and its brand:

- **Pricing model:** real-time **interbank rate + a transparent margin** — the standard published rate is **0.5% above interbank** (free/Explore tier), with volume-based discounts (reported down to ~0.2–0.3% on higher tiers); the task brief's "interbank + 0.2–0.6%" range is broadly accurate, with 0.5% the anchor figure;
- **No hidden fees:** no separate correspondent charges, no "rate spread inside the rate" games — the margin is the price;
- **The FX API:** quote and lock rates programmatically (GET/POST `/fx/quote`, execute via `/fx/order`) — platforms can embed transactional FX (see §16);
- **The FX engine:** a proprietary, real-time engine that aggregates liquidity (from banking partners and market makers) and manages the book, rather than passing every conversion through a single market maker at retail markup.

### 7.2 FX Risk Management

Airwallex's treasury products are deliberately lighter than a full bank treasury suite (compare [nasdaq_calypso_guide.md](nasdaq_calypso_guide.md) for the enterprise-treasury context):

- **Core tools:** hold natural offsets (multi-currency balances), convert at the point of payment, and use the FX API for just-in-time conversion;
- **Limited hedging:** Airwallex does not position itself as a derivatives house; full forward/option hedging remains the domain of bank FX desks (the company has explored forward products in some markets but they are not a headline offering);
- **Practical implication for treasurers:** Airwallex replaces the *execution* layer of cross-border FX (cheaper, faster, API-driven) but not the *risk management* layer (bank treasury systems still run exposure, hedging, and cash forecasting).

### 7.3 FX at the Point of Payment

The signature workflow is **"FX at the point of payment"**: instead of converting USD → SGD → CNY across two legs with two margins, the customer pays a CNY supplier directly from USD balance — Airwallex converts USD → CNY once, at interbank + margin, and delivers CNY via the local rail. One conversion, one margin, one day. This single-leg design is the economic core of the value proposition and the primary reason customers report 90%+ savings on TT/FX fees versus banks.

---

## 8. Airwallex for Platforms: Embedded Finance and BaaS

### 8.1 The Platform Value Proposition

**Airwallex for Platforms** is the embedded-finance business: SaaS platforms, marketplaces, payroll platforms, and enterprises embed Airwallex's licensed financial infrastructure into their own products. Reported platform customers include **Shein** (cross-border e-commerce) and **Papaya Global** (global payroll). By 2025, platform customers drove **~45% of total TPV** despite being a small share of accounts — the fastest-growing segment.

Typical embedded flows:

- **Marketplaces:** sellers are onboarded to Airwallex-connected wallets; the marketplace splits settlement (fees, escrow-like holds, shipping) and pays out sellers globally — *"Airwallex as Acquirer: used by marketplaces to compliantly accept payments via Airwallex's payment gateway and automatically split proceeds, without ever having to touch merchant funds"*;
- **Payroll platforms:** multi-country payroll runs via the mass payout API;
- **Vertical SaaS:** embedded accounts, cards, and FX inside the software their users already run;
- **Escrow-like flows:** hold funds in a wallet until conditions are met, then release — a programmatic version of the conditional payment patterns in [programmable_business_bank_guide.md](programmable_business_bank_guide.md).

### 8.2 The API Suite

The developer surface is a unified REST API family (all under one platform, one credential model):

| API group | Capabilities |
|---|---|
| **Accounts API** | Create/query multi-currency accounts and balances; request local account details; sub-accounts for platforms (connected accounts) |
| **Payments API** | PaymentIntents (acceptance), payment consents, refunds, disputes; webhooks for status changes |
| **Transfers API** | Single and mass payouts; beneficiary management; status tracking |
| **FX API** | Live rates, quotes (with lock), orders; batch conversions |
| **Cards API (Issuing)** | Create/fund/limit/destroy cards; transaction feed; spend-control rules |
| **Webhooks** | Event notifications (`payment_intent.succeeded`, `payment_attempt.paid`, `transfer.status_changed`, `refund.settled`, `payment_dispute.requires_response`, etc.) with signature verification |
| **SDKs** | Frontend SDKs (Drop-in checkout, embeddable components), mobile SDKs (iOS/Android) |
| **OAuth2 (Partner API)** | Authorization-code + PKCE flow to act on behalf of other Airwallex clients |

The API model deliberately resembles the modern payments API canon (Stripe-style PaymentIntents, idempotency keys, webhooks) — see [spec_driven_development_frameworks_guide.md](../technology/spec_driven_development_frameworks_guide.md) for how such APIs are typically specified and tested, and [payments_hub_guide.md](payments_hub_guide.md) for the orchestration context.

### 8.3 The Banking-as-a-Service Angle

Airwallex is frequently grouped with BaaS providers, with an important nuance:

- **What it shares with BaaS:** licensed entities, a partner-bank network, and APIs that let third parties offer financial products under their own brand;
- **What differs:** Airwallex is not a bank core provider or a neutral "bank-in-a-box" (like Solaris/ClearBank); it is a payments company whose own products (accounts, cards, FX) are the same ones it exposes to platforms. Platforms use Airwallex *as* the payments provider, not as the core behind their own bank-like licenses;
- **Practically:** this makes Airwallex a strong embedded-payments choice and a weak choice if the goal is full deposit-taking/lending — for the latter, see the BaaS paths in [programmable_business_bank_guide.md](programmable_business_bank_guide.md).

### 8.4 The Developer Platform

- **Docs:** comprehensive API reference and integration guides (airwallex.com/docs);
- **Sandbox:** full test environment with test cards, simulated rails, and webhook testing;
- **Dashboard:** the merchant/ops console (also used for non-developer administration);
- **Support model:** developer support plus, on enterprise tiers, dedicated solutions engineers.

---

## 9. Integrations and Ecosystem

### 9.1 E-Commerce Platforms

- **Shopify:** official Airwallex Online Payments app — merchants accept cards/wallets/APMs and settle in multiple currencies directly into their Global Account (reportedly settling in up to 8 currencies), avoiding double conversion; the app is a flagship channel for acquiring;
- **WooCommerce and Magento:** official plugins with the same acceptance + settlement model;
- **Marketplaces:** payments-for-platforms integration pattern (§8.1).

### 9.2 ERP and Accounting Software

- **Xero:** two-way sync of transactions and receipts, plus payment links creatable inside Xero invoices;
- **QuickBooks (Intuit):** a formal partnership for multi-currency reporting and reconciliation;
- **NetSuite:** integration for mid-market finance teams (transaction sync, reconciliation);
- **Sage:** supported in the integration catalog;
- Sync cadence is automatic (reported hourly) for transactions and receipts, which is the reconciliation bridge described in §15.3.

### 9.3 Payments Platforms and the Shopify Partnership

- **Stripe / Adyen / Checkout.com:** generally *competitors* in acceptance and embedded payments (see §13), though platforms often run Airwallex alongside a primary acquirer for cross-border/multi-currency payouts — e.g., Stripe Connect for card acceptance + Airwallex for global payouts to sellers;
- **The Shopify relationship** deserves the "partnership" label with a caveat: it is primarily a distribution/integration relationship (Airwallex as a payment method in Shopify checkout) rather than an exclusive strategic alliance; Shopify merchants can choose among gateways, and Airwallex competes for that slot against Shopify Payments, Stripe, PayPal, and Adyen;
- **Payroll platforms:** integrations/serving of global payroll providers (e.g., Papaya Global) as the payout rail.

---

## 10. Business Model and Customer Segments

### 10.1 Revenue Model

| Revenue line | Mechanics | Notes |
|---|---|---|
| **FX spread** | Margin above interbank on every conversion (0.5% standard) | Historically the primary revenue line; still core, now complemented by others |
| **Payment fees** | Card acceptance (interchange + processor markup, ~1.3–3.4% + fixed), transfer fees (mostly on international, not local, transfers) | Acquiring is a large and fast-growing line |
| **Card fees** | Interchange share on corporate card spend; card program fees on higher tiers | Cards + payments together >50% of gross profit (Sacra, 2025) |
| **Subscription / SaaS** | Tiered plans (Free / Advanced / Enterprise; e.g., SG top tier S$399/month) with features (cards included, approvals, API limits) | A small but recurring line; keeps SMBs in the funnel |
| **Platform / volume pricing** | Custom pricing for platform and enterprise volume; embedded-finance contracts often per-transaction | Platform segment = ~45% of TPV |
| **Interest income / float** | Interest earned on safeguarded customer float held at partner banks; Yield product passes returns to customers while generating fee/revenue share | Not a primary disclosed line; exists at the margin |

Airwallex's model is a **take-rate business** (FX + payments + cards), not a net-interest-margin business: it monetizes the *flow* of money, not the *stock* (deposits), which is exactly why it can operate as a licensed payments institution rather than a bank.

### 10.2 Customer Segments

- **SMEs and digital businesses** — the founding segment: e-commerce merchants, SaaS companies, import/export traders, freelancer-heavy businesses;
- **Mid-market corporates / enterprises** — cross-border payables/receivables, subsidiary funding, global payroll, treasury-lite needs;
- **Platforms** — marketplaces, vertical SaaS, payroll/HR platforms embedding finance (§8);
- **Institutional-ish flows** — global fundraising proceeds, large supplier networks (the company cites customers saving "at least 5% per dollar transferred versus the big banks").

### 10.3 Geographic Focus

- **APAC (core):** Australia (origin), Singapore (HQ), Hong Kong (major hub), mainland China (licensed, the RMB corridor), Japan (new license + acquiring), New Zealand (entered March 2025), Vietnam (acquired IPS license holder CTIN Pay);
- **UK/EU:** via the FCA EMI (UK) and DNB EMI (Netherlands, passported across the EEA single market);
- **US:** the 2025–2029 strategic priority — US$1B+ investment in people and brand, San Francisco second HQ, state MTLs + partner bank model;
- **Global:** services accessible in ~200 countries through the network of rails and partners.

### 10.4 Pricing and Plans

Airwallex uses a freemium-ish tiered model:

- **Free (Explore/Starter):** global accounts, local receiving details, transfers, 0.5% FX margin, limited cards — free if the customer maintains qualifying deposit/volume thresholds;
- **Advanced:** more cards (e.g., 50), advanced spend controls, multi-conditional approvals, lower FX margins;
- **Enterprise:** unlimited cards, dedicated account manager, custom volume pricing, API/SLA terms, custom integration support;
- **Platform pricing:** per-transaction, negotiated at volume.

---

## 11. Technology and Architecture

### 11.1 One Platform, One API

Airwallex's architecture is organized around a single claim: **one platform, one API**. A unified API gateway and ledger sit above a global network of licensed entities, banking partners, and local rails, so that a customer (or a platform) integrates once and can move money in any supported market. The design mirrors the "unified API over multi-rail infrastructure" pattern described in [payments_hub_guide.md](payments_hub_guide.md).

### 11.2 The Proprietary Core

- **Payments engine:** routing, validation, retries, and settlement orchestration across rails;
- **Core ledger:** a multi-currency, double-entry ledger maintaining customer balances and sub-accounts (the master-account + virtual-account model of [programmable_business_bank_guide.md](programmable_business_bank_guide.md));
- **FX engine:** real-time rate aggregation, quote locking, and netting across the customer book;
- **Card issuing platform:** card lifecycle, authorization handling, spend controls;
- **Compliance platform:** KYB/KYC, transaction monitoring, sanctions screening (§11.5);
- **Clearing/settlement layer:** connections to banking partners and local schemes, with reconciliation of partner-bank statements to the internal ledger;
- **Infrastructure:** cloud-native microservices (built on public cloud), with regional deployment for data-residency requirements.

### 11.3 The Global Network: Local Rails and Clearing

Where a traditional bank moves cross-border money over SWIFT correspondent chains, Airwallex's strategy is **local clearing membership / local-rail access** via its licensed entities and banking partners:

| Market | Local rails |
|---|---|
| Singapore | **FAST / PayNow** (via MAS MPI entity and SG banking partners) |
| Australia | **NPP (New Payments Platform / Osko)** and BSB network |
| UK | **FPS (Faster Payments Service)**, CHAPS |
| Europe (EEA) | **SEPA** (Credit Transfer / Instant) via the DNB EMI |
| US | **ACH, Fedwire, RTP** via banking partners |
| Hong Kong | **CHATS / FPS** |
| Mainland China | **CIPS and local CNY rails** via the China license (RMB corridor) |
| Japan | Local rails via the registered funds-transfer entity |

This is the same rail taxonomy covered in [financial_infrastructure_guide.md](financial_infrastructure_guide.md): the winning design is to be *in* the local scheme (or behind a partner that is), so that an "international" payment becomes a domestic payment at both ends.

### 11.4 Banking Partners and Correspondent Network

Airwallex's banking partners (publicly cited) include **DBS, J.P. Morgan, Standard Chartered, and ANZ**, plus **Evolve Bank & Trust** in the US (partner bank providing FDIC-insured accounts for some US customers), and others across markets. The partner-bank layer serves several functions:

- Holding customer funds (safeguarding / segregation per e-money rules);
- Providing local account details and clearing access;
- Extending the rail network where Airwallex itself is not a scheme member;
- In the US, providing the FDIC-insured account wrapper for qualifying customers.

The architecture is therefore a **hybrid**: Airwallex's own licenses + banking partners' clearing membership + (increasingly) its own local licenses. The company frames this as moving from "partner-dependent" toward "self-clearing" capability as licenses accumulate (80+ by end-2025).

### 11.5 Compliance Architecture

Compliance is the moat and the cost center:

- **Onboarding:** automated KYB/KYC with document verification, beneficial-ownership checks, and risk scoring — minutes to open, machine-decided where possible;
- **Transaction monitoring:** real-time screening and pattern detection across the ledger;
- **Sanctions screening:** continuous screening against sanctions lists (OFAC, UN, EU, and local lists) across parties and transactions;
- **Segregation/safeguarding:** customer funds held in segregated accounts at partner banks, per e-money and payments regulation;
- **Certifications:** PCI DSS, SOC 1, SOC 2 (and ISO 27001 in relevant entities);
- **Local programs:** each licensed entity runs its own AML/CTF program under its regulator (AUSTRAC, MAS, FCA, DNB, HKMA/C&E, FinCEN, PBOC, etc.).

For the architecture of these engines, see [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md).

---

## 12. Licensing and Regulatory Footprint

### 12.1 The License Map

Airwallex's regulatory model is **license acquisition plus bank partnership** — it holds or operates under more than 80 licenses and permits worldwide (end-2025). The principal ones:

| Jurisdiction | Entity | License / registration | Regulator |
|---|---|---|---|
| **Australia** | Airwallex Pty Ltd | AFSL **487221** (financial services); AUSTRAC remittance-registration (AML/CTF Act 2006) | ASIC / AUSTRAC |
| **Singapore** | Airwallex (Singapore) Pte. Ltd. | **Major Payment Institution (MPI)** license **PS20200541** | MAS |
| **Hong Kong** | Airwallex (Hong Kong) Ltd | Money Service Operator **16-09-01929**; group member UniCard Solution Ltd holds **SVF0009** (stored value facility) | HK Customs & Excise / HKMA |
| **United Kingdom** | Airwallex UK Ltd | **EMI** authorization **900876** | FCA |
| **European Union** | Airwallex Europe (Netherlands) | **EMI license** granted 30 Apr 2021 by **DNB**, passported across the EEA single market | DNB |
| **United States** | Airwallex US, LLC | **Money transmitter licenses** in most states (**NMLS #1928093**); **MSB registration** with FinCEN; partner-bank model with **Evolve Bank & Trust** (FDIC) for some customers | State regulators / FinCEN |
| **Mainland China** | Airwallex (China) entity | **Third-party online payment license** (announced Mar 2023) — makes Airwallex a licensed third-party payment provider in China | PBOC |
| **Japan** | Airwallex Japan | **Type II funds transfer service provider** registration (第二種資金移動業者) — enables local FX/acquiring in Japan | JFSA / Kanto bureau |
| **New Zealand** | Airwallex NZ | Market entry March 2025 (registered/licensed for payments) | FMA / RBNZ (registration) |
| **Vietnam** | CTIN Pay (acquired 2025) | Intermediary payment service (IPS) license | SBV |

*Check current entity/license details in the Airwallex Help Centre ("How is Airwallex licensed and regulated?") and regulator registers before relying on them — licenses evolve.*

### 12.2 The Licensing Strategy: License Acquisition vs Bank Partnership

Airwallex's approach is the **licensed payments infrastructure** model:

1. **Acquire payments licenses** (EMI, MPI, MSO, MTL, remittance) in every market of strategic importance — this gives it direct local-rail access, local-currency accounts, and the right to onboard customers locally without a banking partner for each product;
2. **Use bank partners** where a full banking license is required (deposit-taking, FDIC-insured US accounts, scheme membership that only banks hold);
3. **Avoid becoming a bank:** no deposit-taking ambition, no lending, no deposit-insurance obligations — this keeps the regulatory perimeter lighter (payments regulation instead of banking regulation) while still safeguarding customer funds.

The trade-off is precisely what §14.1 discusses: Airwallex gets speed and reach but not the trust wrapper, capital markets access, or deposit franchise of a bank.

---

## 13. Competitive Landscape

### 13.1 vs Traditional Banks

The incumbent alternative for cross-border B2B payments is **SWIFT correspondent banking**:

| Dimension | Bank (SWIFT) | Airwallex |
|---|---|---|
| FX pricing | 2–4% margin typical, opaque | Interbank + 0.5% (transparent) |
| Speed | 2–5 business days, multiple hops | 92% same-day |
| Fees | Correspondent + intermediary charges | One transparent fee |
| Local receiving | Requires local accounts per country | Local details in 20+ currencies |
| API | Often absent or legacy file-based | Full REST + webhooks |
| Trust | Deposit-insured, regulated, balance sheet | Licensed payments institution, safeguarded funds |

Banks retain the advantages of credit (loans, overdrafts), deposits, treasury products, and relationship banking — Airwallex competes on the *payments rail* layer, not the balance-sheet layer.

### 13.2 vs Payments Fintechs

- **Wise (Wise Business)** — the closest direct competitor in cross-border transfers and multi-currency accounts: similar interbank + margin pricing, local account details, and a London-listed, profitable model (FY2025: £1.2B revenue, 15.6M customers, £21.5B customer holdings). Wise is stronger in consumer/remittance and price leadership; Airwallex is stronger in APAC, platform/embedded finance, acceptance, and issuing. Wise has no significant acquiring product; Airwallex does.
- **Payoneer** — US-listed, strong in marketplace payouts (Amazon sellers), freelance, and China trade corridors; revenue ~US$900M (12 months to mid-2024). Payoneer is more marketplace-led and US-centric; Airwallex is more platform/API-led and APAC-centric.
- **WorldFirst (Ant Group)** — the China-corridor specialist (CNY receiving/paying for cross-border e-commerce), owned by Ant Group; a direct competitor in the China corridor at lower prices, but narrower (no broad acquiring, no EU/US account stack).
- **Currencycloud (Visa)** — B2B FX + payments API for platforms; overlaps the embedded-FX layer, but with Visa ownership and a narrower product set (no cards/accounts stack of its own).
- **Checkout.com / Adyen** — enterprise acquiring competitors in acceptance; they win on enterprise processing scale, while Airwallex wins where multi-currency *accounts + FX + payouts* matter alongside acceptance.
- **Stripe (Treasury, Issuing, Connect)** — the biggest platform overlap: Stripe Connect + Issuing + Treasury versus Airwallex for Platforms. Stripe is stronger in developer mindshare, acceptance breadth, and US dominance; Airwallex is stronger in cross-border multi-currency payouts, APAC rails (CIPS, FPS, FAST), and licensed local entities in APAC.

### 13.3 vs Business Banking Fintechs

Mercury, Brex, Ramp (and APAC peers Aspire, Statrys, Arival) focus on the US/SG domestic business-banking stack: deposits, spend management, and accounting sync — with cross-border as an add-on. Airwallex inverts the priority: cross-border + multi-currency + FX is the core, with spend management as a supporting layer (see the player analysis in [programmable_business_bank_guide.md](programmable_business_bank_guide.md)). Mercury's cross-border and Wise/Revolut Business's card products overlap at the edges, but no US business-banking fintech offers Airwallex's local-account + rail coverage across APAC.

### 13.4 The Comparison Table

| Company | Focus | Model | Coverage | Fees (indicative) | Strengths |
|---|---|---|---|---|---|
| **Airwallex** | Cross-border payments + financial infrastructure (accounts, cards, FX, acceptance, embedded finance) | Licensed payments institution, take-rate (FX + payments + cards) | 60+ currencies, 200+ countries, local details 20+ currencies; APAC-strong | FX interbank + 0.5%; acquiring 1.3–3.4% + fixed | APAC rails, one API, platform/embedded finance, local accounts, China corridor |
| **Wise** | Cross-border transfers + multi-currency accounts (consumer + business) | Licensed EMI, take-rate; London-listed, profitable | 40+ currencies, ~160 countries | Interbank + 0.3–1%+ | Price transparency, brand, consumer base, profitability |
| **Payoneer** | Marketplace payouts, freelance, cross-border e-commerce | Licensed payments (US-listed) | 150+ countries, 150+ currencies | Transaction + FX fees | Marketplace/China trade corridors, scale |
| **WorldFirst (Ant)** | China-corridor cross-border payments (CNY) | Licensed (Ant Group) | China-centric + major corridors | Low, aggressive CNY pricing | China rails, e-commerce sellers |
| **Currencycloud (Visa)** | B2B FX + payments API for platforms | EMI/MTL via partners | ~35 currencies | Interbank + spread | Pure embedded-FX API, Visa backing |
| **Stripe** | Global acceptance + Connect/Treasury/Issuing | Aggregator + partner banks | 100+ countries (acceptance) | 2.9% + 30¢ typical | Developer ecosystem, acceptance breadth, US strength |
| **Adyen** | Enterprise acquiring | Licensed PSP (direct scheme membership) | Global, single platform | Negotiated | Enterprise processing, direct acquiring |
| **Checkout.com** | Enterprise acquiring + payouts | Licensed PSP | Global | Negotiated | Enterprise scale, APAC processing |
| **Mercury** | US business banking (startups) | Partner banks (FDIC) | US-centric | Free tiers | US startup banking UX |
| **Brex / Ramp** | US corporate cards + spend management | Partner banks + card programs | US-centric | Card/interchange model | Spend controls, accounting sync |
| **Traditional banks** | Full-service banking incl. SWIFT cross-border | Bank (balance sheet) | Global | 2–4% FX + SWIFT fees | Trust, credit, deposits, treasury |

### 13.5 Airwallex Differentiation

1. **Cross-border-first design** — every product assumes multi-currency operation; no domestic-first architecture with international bolted on;
2. **APAC strength** — the China corridor (RMB, CIPS), Southeast Asia rails (FAST/PayNow, FPS), and licensed entities in AU/SG/HK/CN/JP — the region where most global B2B payment pain lives;
3. **One API over the whole stack** — accounts + FX + transfers + cards + acceptance + webhooks, unique among cross-border players;
4. **Local-account coverage** — 20+ currencies of local receiving details;
5. **Embedded finance for platforms** — ~45% of TPV from platforms;
6. **Transparent interbank pricing** — 0.5% margin, no hidden correspondent fees.

---

## 14. Banking Context

### 14.1 A Bank-Adjacent Fintech, Not a Bank

Airwallex occupies the **bank-adjacent** space: a licensed payments institution that behaves like a business bank for cross-border operations but is structurally not a bank:

- **No deposit-taking:** customer funds are safeguarded (segregated at partner banks / e-money safeguarding), not deposited with Airwallex's balance sheet;
- **No lending / no credit products:** no overdrafts, loans, or credit cards (cards are debit-funded from balances);
- **No deposit insurance:** funds are not covered by FSCS/DIIC/SDIC-type schemes (US customers with partner-bank accounts may have FDIC pass-through where offered);
- **Regulatory perimeter:** payments/e-money regulation (EMI, MPI, MSO, MTL) instead of banking regulation — lighter capital requirements, faster product iteration, less systemic trust.

For architects this matters: Airwallex is a *rail and platform* vendor, not a *bank counterparty* in the deposit/credit sense. Treasurers should keep their primary bank relationship and use Airwallex as the payments/FX execution layer (see §14.4).

### 14.2 Airwallex vs Digital Banks

Digital banks (see [green_link_digital_bank_guide.md](green_link_digital_bank_guide.md)) hold banking licenses, take deposits, and lend; Airwallex does none of these. The two are complements more than competitors:

- A digital bank provides the regulated deposit/credit relationship; Airwallex provides the global payments/FX/cards plumbing;
- Several digital banks and neobanks are effectively Airwallex *customers* (embedding its payouts), while others compete in SME cross-border;
- For a Singapore SME: DBS/OCBC/UOB digital banking for domestic banking + Airwallex for cross-border is a common stack; Airwallex's own account is not a substitute for a full banking relationship (no loans, no deposit insurance).

### 14.3 Airwallex in the Singapore Ecosystem

Singapore is Airwallex's regulatory and operational center of gravity:

- **MAS MPI license PS20200541** — one of the largest MPI-licensed payment firms in Singapore, subject to the full MAS payments regulatory framework (safeguarding, AML/CFT, technology risk, notification obligations);
- **Global HQ** in Guoco Tower (Tanjong Pagar), the largest employee hub (~160+ staff locally, per public data);
- **SG products:** SGD local accounts, FAST/PayNow receiving, SGD/global transfers, cards, acquiring;
- **SG banking partnerships:** relationships with DBS and other major local banks for clearing and safeguarding;
- **Ecosystem role:** a flagship of Singapore's fintech scene, frequently cited alongside MAS's payments modernization efforts (see the payments regulatory context in [financial_infrastructure_guide.md](financial_infrastructure_guide.md)).

### 14.4 Airwallex for Corporate Treasury

For corporate treasurers (the world of [nasdaq_calypso_guide.md](nasdaq_calypso_guide.md)), Airwallex is best understood as a **modern execution layer** for cross-border payments and FX:

- **What it replaces:** the slow/expensive SWIFT + bank FX desk execution for routine cross-border payables/receivables;
- **What it does not replace:** the bank treasury stack — cash forecasting, liquidity management, hedging/derivatives, in-house banking, and bank relationship infrastructure;
- **Integration pattern:** run Airwallex alongside the core bank (multi-bank setup), push payments via API from the treasury system, reconcile via webhooks/statements;
- **Typical corporate use:** subsidiary funding, global payroll, supplier payments, e-commerce settlement, and emergency FX execution — at a fraction of bank cost, with full API automation.

---

## 15. The Architect's View: Integrating Airwallex

### 15.1 Integration Patterns

- **REST APIs** (`/api/v1/...`) with JSON; versioned; documented at airwallex.com/docs. The canonical resources: `accounts`, `transfers`, `fx/quote`, `fx/order`, `payment_intents`, `payment_consents`, `cards`, `beneficiaries`, `webhooks`;
- **PaymentIntents model** for acceptance: create an intent, attach a payment method, confirm; **always listen for the success webhook rather than trusting the synchronous response** (the browser may close mid-flow) — the same guidance as [payments_hub_guide.md](payments_hub_guide.md);
- **Frontend SDKs:** Drop-in and embeddable components for checkout; mobile SDKs for iOS/Android;
- **API-first platform flows:** open accounts, issue cards, and run payouts programmatically (no dashboard dependency);
- **Sandbox:** test in the sandbox environment with test data before production keys.

### 15.2 Authentication and Security

- **Standard auth:** API keys + Client ID → `POST /api/v1/authentication/login` → **bearer access token** (short-lived, refreshed automatically); tokens authorize all other endpoints;
- **Partner API (on behalf of others):** **OAuth2** authorization-code flow with **PKCE** — used by platforms acting for multiple Airwallex clients;
- **Webhook verification:** every webhook carries `x-signature` / `x-timestamp` headers; verify the HMAC signature and replay-window before processing (see [spec_driven_development_frameworks_guide.md](../technology/spec_driven_development_frameworks_guide.md) for contract-testing such integrations);
- **Transport/at-rest:** TLS in transit; encryption at rest; PCI DSS scope for card data (hosted fields/checkout keep card data out of your servers);
- **No mTLS required** for standard APIs (TLS 1.2+); secrets are API keys/credentials — store them in a vault, rotate per policy.

### 15.3 Webhooks, Idempotency, and Reconciliation

- **Webhooks:** event types include `payment_intent.succeeded`, `payment_attempt.paid`, `payment_consent.verified`, `refund.settled`, `payment_dispute.requires_response`, `transfer.status_changed`, and account/balance events. Design handlers to be **idempotent** (dedupe by event ID) and to retry with backoff on failure;
- **Idempotency:** create-type endpoints accept an **Idempotency-Key** header — retrying the same request with the same key returns the original result instead of double-executing (the payment idempotency pattern of [payments_hub_guide.md](payments_hub_guide.md));
- **Reconciliation:** match incoming payments to virtual accounts (§4.3) and reconcile outbound transfers via status webhooks + daily statements/transaction exports; the accounting integrations (Xero/NetSuite/QB) automate the books side (see virtual-account reconciliation in [programmable_business_bank_guide.md](programmable_business_bank_guide.md));
- **Operational pattern:** webhook → write to your ledger → reconcile against Airwallex statements daily → escalate on mismatches.

### 15.4 Data Residency

- Customer and transaction data are processed and stored **regionally** aligned to the contracting entity: EEA data under the Dutch EMI (GDPR), Singapore data under the MPI entity, US data under Airwallex US, China data under the China entity (subject to Chinese data-localization requirements);
- For platforms with global users, the pattern is **entity-per-region contracting** — your contract and data residency follow the Airwallex entity your customer signs with;
- Verify the current data-center and sub-processor list in the privacy/DPA documentation during vendor due diligence, especially given the 2025–2026 scrutiny (§2.7).

### 15.5 Risk and Concentration

- **Platform dependency:** embedding Airwallex deeply (accounts + cards + payouts + acceptance) creates switching costs and single-vendor concentration; design a thin abstraction layer over the Airwallex APIs so a migration (e.g., to Wise/Currencycloud/Stripe or a bank) is feasible;
- **Counterparty/regulatory risk:** Airwallex is a non-bank payments institution — no deposit insurance, funds safeguarded (not guaranteed); monitor its licenses, safeguarding arrangements, and any regulatory actions;
- **Geo-political risk (2025–2026):** the US scrutiny over China ties (§2.7) is a live vendor risk for US-facing deployments; track Treasury/CFIUS developments, and consider entity/regional contracting (e.g., US entity for US data) as mitigation;
- **Operational risk:** multi-rail settlement depends on partner banks and local schemes — the standard clearing/settlement risk of the rails described in [financial_infrastructure_guide.md](financial_infrastructure_guide.md);
- **Fraud/compliance risk:** acceptance products bring card fraud and disputes; Airwallex provides dispute tooling (`payment_dispute.requires_response`), but you own your fraud strategy at the checkout layer.

### 15.6 Vendor Selection Framework

For a cross-border payments provider decision (Airwallex vs bank vs other fintechs):

1. **Define the flows:** inbound collection (local details? virtual accounts?), outbound payouts (countries/currencies/volume), FX needs, card needs, acceptance needs;
2. **Score the rail coverage:** local account details in your currencies; local payout rails in your destination markets (CIPS for China, FPS for UK, etc.);
3. **Price the take-rate:** FX margin + transfer fees + acquiring fees + card fees for *your* volume mix, not the headline rate;
4. **Assess the platform fit:** API quality, webhooks/idempotency, sandbox, SDKs, accounting integrations, support model;
5. **Evaluate the trust wrapper:** licenses in your markets, safeguarding, data residency, certifications (PCI DSS/SOC), financial health, and (in 2026) geo-political exposure;
6. **Check concentration and exit:** can you run multi-vendor (bank + fintech) with an abstraction layer? What does offboarding look like?
7. **Decision heuristic:** *bank* if you need credit/deposits/treasury depth; *Airwallex-like fintech* if the requirement is cross-border flow at low cost with API automation and APAC rails; *specialist* (Wise/Currencycloud/Payoneer/WorldFirst) if your flows are narrow (consumer remittance, China corridor, marketplace payouts).

---

## 16. Worked Example: A Singapore E-Commerce Business Pays a Chinese Supplier

**Scenario:** A Singapore e-commerce company sells to US customers (receives USD) and pays a supplier in mainland China (CNY). The flow uses Airwallex as the entire cross-border layer.

**Step 1 — USD receiving (US local account).** The business opens a Global Account and requests a **USD local account** (ABA routing + account number). US customers pay via ACH/domestic wire as if paying a US company. Funds land in the USD balance — no SWIFT, no correspondent fees, no FX.

**Step 2 — FX conversion (USD → CNY).** When the supplier invoice is due, the business converts USD to CNY at **interbank + 0.5%** via the FX API. (Alternative: skip the explicit conversion and let the transfer itself convert — "FX at the point of payment.")

**Step 3 — Payment to the supplier (CNY via local rails).** The business creates a CNY transfer to the supplier's Chinese bank account. Airwallex routes via **CIPS/local CNY rails** (supported use case: cross-border e-commerce sellers paying third-party suppliers in mainland China). The supplier receives CNY domestically — typically same-day.

**Step 4 — API calls (illustrative sequence).**

```
POST /api/v1/authentication/login
     {client_id, api_key}  →  {token}

# 1. Create beneficiary (once)
POST /api/v1/beneficiaries/create
     {name: "Supplier Co.", account_details: {bank_name, bank_account_no, ...}}

# 2. Quote FX (USD → CNY)
POST /api/v1/fx/quote
     {source_currency: "USD", target_currency: "CNY",
      amount: 10000, conversion_action: "sell"}  →  {quote_id, rate, valid_until}

# 3. Create transfer (with point-of-payment conversion)
POST /api/v1/transfers/create
     {source_currency: "USD", source_amount: 10000,
      target_currency: "CNY", beneficiary_id: "...",
      quote_id: "...",           # optional: convert at quoted rate
      idempotency_key: "inv-2026-08-001"}   →  {transfer_id, status: "processing"}

# 4. Webhook confirms completion
POST /your-webhook   x-signature: ...   body:
     {event: "transfer.status_changed", data: {transfer_id, status: "completed"}}
```

**Step 5 — Reconciliation.** Incoming USD customer payments arrive against **virtual account numbers** (one per customer/channel), so each payment auto-matches to an order. The outbound CNY transfer reconciles via the `transfer.status_changed` webhook and daily statements. Xero/NetSuite sync automates the ledger. The whole loop — receive USD, convert, pay CNY, reconcile — runs with zero manual bank work.

**Step 6 — Card and acceptance tail (optional).** The business can additionally accept card payments via Airwallex Online Payments (settling in USD directly into the same wallet) and issue Visa cards to its team for operating expenses — all on the same API/account.

**Cost comparison (illustrative):** bank SWIFT + bank FX would charge ~2–4% FX margin + correspondent fees + 2–5 days; the Airwallex path costs ~0.5% FX + a small transfer fee (often free on local legs) and lands same-day.

---

## 17. Airwallex in Singapore and Asia

### 17.1 Singapore Operations

- **MAS MPI license PS20200541** (Airwallex (Singapore) Pte. Ltd., incorporated 2016) — full suite of regulated payment services: account issuance, domestic and cross-border transfers, merchant acquisition, and e-money;
- **HQ office** at Guoco Tower, Tanjong Pagar (opened February 2023); Singapore is the largest employee hub and the seat of the group's global operations;
- **Local products:** SGD local account details (FAST/PayNow receiving), SGD↔foreign transfers, SGD corporate cards, local acquiring;
- **Local positioning:** Airwallex is a reference case in MAS-regulated payments infrastructure — see the payments-regulation and rails context in [financial_infrastructure_guide.md](financial_infrastructure_guide.md).

### 17.2 The APAC Strategy and the China Corridor

- **China corridor:** RMB flows are the strategic core — the China online-payment license (March 2023), CNY local receiving/paying (CIPS/local rails), and support for cross-border e-commerce repatriation into mainland China;
- **ASEAN expansion:** Singapore hub + Vietnam (CTIN Pay acquisition, 2025) + New Zealand entry (March 2025) + Japan (funds-transfer registration + acquiring launch);
- **Hong Kong:** MSO + SVF licenses; a major treasury and trade node;
- **Australia:** origin market with AFSL + AUSTRAC registration; Melbourne remains an engineering hub;
- **The thesis:** the fastest-growing global B2B payment flows are intra-Asia (China manufacturing ↔ SEA/US/Europe demand); local-rail access in each node is the moat.

### 17.3 Competition in Asia

- **Wise** — strong in SG/AU/UK transfers and accounts; competes head-on in FX and local details, weaker in acceptance and platform depth;
- **WorldFirst (Ant Group)** — aggressive on the China e-commerce corridor (CNY pricing), narrower product scope;
- **Local/China specialists:** XTransfer, PingPong, LianLian — procurement-focused China-supplier-payment platforms that compete on the same corridor;
- **Regional neobanks:** Aspire (SG), Statrys, Arival — SG/HK business accounts with cross-border features, competing for SME primary-account mindshare;
- **Incumbent banks:** DBS/OCBC/UOB and HK/CN banks retain the deposit and credit relationship but lose payments flow to fintechs on cost and speed.

### 17.4 The Regulatory Environment

MAS's payments framework (Payment Services Act 2019, with the 2024–2025 amendments expanding the regulated activity scope, e-money safeguards, and user-protection duties) is the primary SG constraint and enabler: the MPI regime gave Airwallex a clear, scalable license — the model the company replicated across APAC. For the full rails/regulatory map, see [financial_infrastructure_guide.md](financial_infrastructure_guide.md).

---

## 18. The Future: 2026 and Beyond

### 18.1 IPO Path and Profitability

- **IPO:** IPO-ready within a year (June 2026 statement); Jack Zhang has said no listing before 2028; expect filing speculation through 2027. Venue unconfirmed (HK/US most cited);
- **Profitability:** EBITDA-positive since Q4 2025; net profitability still ahead, but the operating leverage story is now credible (>US$1B ARR, ~US$200B+ TPV, 80+ licenses);
- **Valuation trajectory:** US$5.6B (2022) → US$6.2B (May 2025) → US$8B (Dec 2025) → US$11B (Jun 2026) — a steep re-rating on the embedded-finance + AI narrative.

### 18.2 Product Expansion: Toward a Full-Stack Financial Platform

- **Stated vision:** US$10B ARR ("empower businesses to operate anywhere, anytime");
- **Likely expansion vector:** deepen the "financial platform" stack — Yield/investments (already live), lending/working capital (reportedly explored; would require partners or new licenses), expense management and spend analytics, invoicing, and treasury-lite tools;
- **"Airwallex bank"?** No bank-license ambition announced; the model remains licensed payments + partner banks. A full "Airwallex bank" would be a strategic departure (deposit-taking, capital requirements) — more likely is continued bank-like *products* (Yield, cards, credit via partners) without a banking license;
- **Cards and acceptance:** continued geographic rollout of acquiring and issuing (Japan, LatAm/ME as licenses land).

### 18.3 Embedded Finance Growth

Platforms already drive ~45% of TPV; expect the embedded-finance business to outgrow the direct business: marketplaces, payroll, vertical SaaS, and (increasingly) AI-agent operators embedding the same API surface. This is the wedge into the "financial infrastructure for the AI economy" narrative.

### 18.4 AI in Airwallex

- **AI-native strategy (2025–2026):** the company frames itself as building "financial infrastructure for a world where AI agents handle business payments and finance" — the Series H was explicitly raised to fund this;
- **Shipped:** an AI assistant embedded in the web app (automates onboarding, multi-currency setup, card issuance, workflow configuration via conversational commands);
- **Pipeline:** AI-powered compliance (automated onboarding/transaction monitoring — see [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md)), AI-assisted FX/treasury decisions, agentic payments (agents initiating and reconciling payments through the APIs);
- **Architect's note:** agentic payments make idempotency, webhook reliability, and reconciliation *more* critical — the §15 patterns become table stakes.

### 18.5 Global Expansion

- **US:** the priority market — US$1B+ investment 2026–2029, San Francisco second HQ, deepening MTL coverage and partner-bank relationships;
- **Japan:** new license + acquiring rollout;
- **Latin America, Middle East, Africa:** expected next wave (licenses and partnerships), consistent with the "global financial infrastructure" thesis — as of mid-2026 these are directional rather than announced;
- **The constraint:** each new market means licenses, banking partners, rails, and compliance — the moat and the cost structure of the business.

---

## 19. Glossary

| Term | Definition |
|---|---|
| **FX** | Foreign exchange — converting one currency to another; Airwallex's core revenue line (margin over interbank) |
| **Interbank rate** | The wholesale rate at which banks trade currencies among themselves; the transparent baseline Airwallex prices from (plus a margin, typically 0.5%) |
| **Local account / local details** | Account details (e.g., US ABA + account number, UK sort code + account, IBAN, BSB) that let a business receive money as a domestic payment in that country without a local entity |
| **Virtual account** | A unique sub-identifier (often an account number) mapped to a master account, used to tag and reconcile incoming payments; see [programmable_business_bank_guide.md](programmable_business_bank_guide.md) |
| **Mass payments** | Batch payouts (payroll, suppliers, sellers) executed in one run via file or API |
| **Payment link** | A hosted payment page shared with a customer to request payment (invoice-style), no checkout build required |
| **Acquiring** | Accepting card/wallet payments on behalf of a merchant (merchant acquiring); Airwallex is the acquirer/gateway in its Online Payments product |
| **APM** | Alternative payment method — local methods beyond cards (wallets, bank transfer schemes, BNPL); Airwallex supports 160+ |
| **MPI license** | Major Payment Institution — a MAS license (Singapore) for payment service providers above volume thresholds, with comprehensive regulation; Airwallex holds PS20200541 |
| **EMI license** | Electronic Money Institution — an EU/UK license to issue e-money and provide payment services; Airwallex holds UK (FCA 900876) and Netherlands (DNB) EMIs |
| **MTL** | Money Transmitter License — US state licenses for money transmission; Airwallex US holds them in most states (NMLS #1928093) |
| **MSB** | Money Services Business — the FinCEN registration category for US money transmitters |
| **CIPS** | Cross-Border Interbank Payment System — China's RMB cross-border payment system; the modern alternative to SWIFT for CNY |
| **NPP** | New Payments Platform — Australia's real-time payment rail (Osko); Airwallex receives/pays via BSB-based local accounts |
| **FPS** | Faster Payments Service — the UK's near-real-time rail (sort code + account); CHAPS for high value |
| **Correspondent banking** | The legacy network where banks hold accounts with each other (Nostro/Vostro) to move cross-border money, often via SWIFT — slow and costly; Airwallex avoids it where local rails exist |
| **SWIFT** | The messaging network (and its MT/ISO 20022 message standards) used by banks for cross-border payments; see [iso_20022_core_processes_guide.md](iso_20022_core_processes_guide.md) |
| **Embedded finance** | Financial products (accounts, cards, payments, FX) embedded into a non-financial platform's product via APIs; Airwallex for Platforms |
| **Platform API** | The API surface (accounts, payments, transfers, FX, cards, webhooks) platforms integrate to offer finance to their users |
| **Webhook** | An HTTP callback Airwallex sends on events (payment succeeded, transfer status changed); signature-verified with `x-signature`/`x-timestamp` |
| **Idempotency** | Retrying an API call with the same key yields the same result, preventing duplicate payments; Airwallex supports `Idempotency-Key` |
| **Reconciliation** | Matching payments/ledger entries to orders/invoices — via virtual accounts, webhooks, and statements; see [programmable_business_bank_guide.md](programmable_business_bank_guide.md) |
| **KYB** | Know Your Business — corporate onboarding due diligence (beneficial ownership, entity verification) automated by Airwallex |
| **Sanction screening** | Checking parties/transactions against sanctions lists (OFAC, UN, EU, local) in real time |
| **Treasury** | Managing a company's cash, FX, and liquidity; Airwallex is the execution layer for cross-border treasury flows; see [nasdaq_calypso_guide.md](nasdaq_calypso_guide.md) |
| **BaaS** | Banking-as-a-Service — offering banking products through APIs over licensed entities; Airwallex is a partial BaaS (payments stack, not core banking) |
| **TPV / ARR** | Total Payment Volume (annualized transaction volume) / Annual Recurring Revenue — Airwallex reported ~US$200B+ TPV and >US$1B ARR in 2025 |

---

## 20. Sources and Notes

**Primary sources:** Airwallex newsroom and press releases (Series E2 2022, China license 2023, Yield launches, Series F/G/H 2025–2026, 2025 End-of-Year Mission Update), Airwallex Help Centre (licensing/regulation), Airwallex Docs (API, OAuth2, webhooks, China payouts), airwallex.com product pages.

**Analyst and media:** Sacra research ("Airwallex at $700M revenue", 2025; "Airwallex reached EBITDA profitability in Q4 2025"), The Business Times Singapore (Series G/H coverage), Australian Financial Review (Sen. Cotton letter, June 2026), Axios (Cotton–Bessent letter), Reuters/CNBC profiles (co-founders, Series G), SMH/The Age (HQ relocation, Latitude 37, 2026), Wise FY2025 Annual Report (comparison data), Fintech Futures, Tech Funding News, Reuters, Wikipedia (Airwallex — for the co-founder names and license/valuation chronology).

**Verification notes:**
- The co-founder list is **Jack Zhang, Lucy Liu, Max Li, Xijing (Jacob) Dai, and Ki-lok Wong**. Several early sources and the task brief render the fourth name as "Xijing Wang"; the documented spelling is **Xijing Dai** (also styled Jacob Dai).
- Valuation: Series E (Oct 2021) US$5.5B; **Series E2 (Oct 2022) US$5.6B**; Series F (May 2025) US$6.2B; Series G (Dec 2025) US$8B; **Series H (Jun 2026) US$11B**. Total raised ≈ **US$1.8B**.
- HQ: **global HQ Singapore (Feb 2023, Guoco Tower)**; **San Francisco designated second global HQ (Dec 2025)**; domiciled Cayman Islands.
- Profitability: **EBITDA-profitable Q4 2025**; >US$1B ARR by Dec 2025; IPO guidance "not before 2028" (Jan 2026) and "IPO-ready within a year" (Jun 2026).
- FX pricing: **interbank + 0.5%** standard on the free tier (range ~0.2–0.6% depending on plan/volume).
- Coverage: 60+ currencies held; **local receiving details in 20+ currencies**; transfers to **200+ countries**; **160+ local payment methods** (acceptance).
- Licenses verified: AU AFSL 487221 + AUSTRAC; SG MAS MPI PS20200541; HK MSO 16-09-01929 (+SVF via UniCard SVF0009); UK FCA EMI 900876; NL DNB EMI (Apr 2021); US MTLs NMLS #1928093 + FinCEN MSB (+ Evolve Bank & Trust partner); CN third-party payment license (Mar 2023); JP Type II funds-transfer registration.
- **Figures are as of August 2026** and should be re-verified against Airwallex's current disclosures and regulator registers before use in procurement or architecture decisions. The US political scrutiny of Airwallex's China ties (2025–2026) is ongoing and evolving.

**Sibling guides in this repository:** [programmable_business_bank_guide.md](programmable_business_bank_guide.md) · [payments_hub_guide.md](payments_hub_guide.md) · [financial_infrastructure_guide.md](financial_infrastructure_guide.md) · [iso_20022_core_processes_guide.md](iso_20022_core_processes_guide.md) · [green_link_digital_bank_guide.md](green_link_digital_bank_guide.md) · [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md) · [nasdaq_calypso_guide.md](nasdaq_calypso_guide.md) · [core_banking_systems_guide.md](core_banking_systems_guide.md) · [supply_chain_finance_guide.md](supply_chain_finance_guide.md) · [spec_driven_development_frameworks_guide.md](../technology/spec_driven_development_frameworks_guide.md)
