# Temenos: A Comprehensive Guide

> **Scope.** This guide is the dedicated deep-dive on Temenos — the company, its product
> portfolio (Transact/T24, Infinity, Banking Cloud, Payments, Data Hub, Financial Crime
> Mitigation, Exchange), the Transact architecture, deployment and implementation models,
> competitive positioning, and how banks actually use Temenos. It is one of a series of
> sibling guides in this repository; where a topic already has a dedicated guide, this
> document cross-references it by filename instead of duplicating it:
>
> - Vendor landscape, core-banking eras, composable debate → `core_banking_systems_guide.md`
> - Oracle FLEXCUBE / OBMA competitor deep-dive → `oracle_banking_microservices_architecture_guide.md`
> - Open-source competitor → `apache_fineract_guide.md`
> - Singapore digital bank build on Temenos (ANEXT/GLDB on Huawei Cloud, 11-month implementation) → `green_link_digital_bank_guide.md`
> - Payments hub architecture, vendor options, Singapore context → `payments_hub_guide.md`
> - ISO 20022 payments processes (pain/pacs/camt, CBPR+, instant) → `iso_20022_core_processes_guide.md`
> - AML/financial-crime systems and regulatory context → `financial_risk_compliance_systems_guide.md`

**Facts note.** Where a number is verified against a public source (Temenos press
releases, Euromoney, Hindenburg report, Reuters) it is stated plainly. Where the figure is
approximate or contested, it is flagged as such. **Important correction to a common
misconception: Vista Equity Partners' 2022 bid for Temenos did NOT complete** — see §2.6.

---

## Table of Contents

1. [What Is Temenos?](#1-what-is-temenos)
2. [Company History](#2-company-history)
3. [Product Portfolio at a Glance](#3-product-portfolio-at-a-glance)
4. [Temenos Transact (Core Banking)](#4-temenos-transact-core-banking)
5. [Temenos Infinity (Digital Banking)](#5-temenos-infinity-digital-banking)
6. [Temenos Payments (Payments Hub)](#6-temenos-payments-payments-hub)
7. [Temenos Data Hub (Data Layer)](#7-temenos-data-hub-data-layer)
8. [Temenos Financial Crime Mitigation (FCM)](#8-temenos-financial-crime-mitigation-fcm)
9. [Temenos Analytics, Wealth, Fund Management, Insurance](#9-temenos-analytics-wealth-fund-management-insurance)
10. [Temenos Banking Cloud (SaaS)](#10-temenos-banking-cloud-saas)
11. [Temenos Exchange (Marketplace)](#11-temenos-exchange-marketplace)
12. [The Transact Architecture in Depth](#12-the-transact-architecture-in-depth)
13. [Deployment Models](#13-deployment-models)
14. [The Implementation Model](#14-the-implementation-model)
15. [Competitive Landscape](#15-competitive-landscape)
16. [Strengths and Criticisms](#16-strengths-and-criticisms)
17. [Banking Context: How Banks Use Temenos](#17-banking-context-how-banks-use-temenos)
18. [Implementation Reality: Timelines, Cost, Upgrades](#18-implementation-reality-timelines-cost-upgrades)
19. [Integration Patterns](#19-integration-patterns)
20. [Temenos Skills and Careers](#20-temenos-skills-and-careers)
21. [Temenos in Singapore and Asia](#21-temenos-in-singapore-and-asia)
22. [The Future: 2026 and Beyond](#22-the-future-2026-and-beyond)
23. [Glossary](#23-glossary)
24. [References and Related Guides](#24-references-and-related-guides)

---

## 1. What Is Temenos?

Temenos AG (SIX: TEMN) is a Swiss banking-software company headquartered in
Grand-Lancy, Geneva, founded in 1993. It is, by market share and installed base, the
**world's largest core-banking software vendor**: the industry sales-league tables
(IBS Intelligence, Forrester, IDC) have consistently ranked it #1 in core banking, and
Euromoney named its core solution "the world's best core banking solution" in 2025.
See the vendor-landscape section of `core_banking_systems_guide.md` (§5.2) for where
Temenos sits in the market structure.

### 1.1 Scale in numbers (verified)

| Metric | Figure | Source / as of |
|---|---|---|
| Banks / financial institutions served | 1,000+ across 150+ countries | Euromoney, 2025 |
| Total customers (broader definition incl. smaller institutions) | ~3,000 | Hindenburg report, Feb 2024 |
| Banks on Temenos Banking Cloud (SaaS) | 700+ | Temenos Banking Cloud factsheet, 2022 |
| Digital-banking (Infinity) clients | 850+ | Temenos press release, Nov 2022 |
| Revenue FY2024 | US$1.04B (+4.4% YoY) | FY2024 results, Feb 2025 |
| ARR FY2024 | US$804M (+12% constant currency, top of guidance) | FY2024 results, Feb 2025 |
| FY2025 EPS | US$4.06 (vs US$2.46 FY2024) | FY2025 results, Feb 2026 |
| Q2 2026 | ARR +11% YoY; deal slippage noted | Q2 2026 results, Jul 2026 |
| HQ | Grand-Lancy, Geneva, Switzerland | — |
| Listing | SIX Swiss Exchange, ticker TEMN (still listed as of mid-2026) | — |

The name "Temenos" comes from the Greek *temenos* (τέμενος) — a sacred precinct cut
off from ordinary ground. The company's founding product, T24, was built to run
"24 hours a day", which is where the "24" comes from.

### 1.2 What Temenos sells

Temenos is a **"full-stack" banking vendor**: it sells everything a bank needs to run
its system of record and its customer channels, rather than a single point solution:

- **Core banking** — Temenos Transact (formerly T24), the flagship core.
- **Digital banking** — Temenos Infinity (omnichannel front-end, portal + mobile).
- **Payments** — Temenos Payments (ISO 20022-native payments hub).
- **Data** — Temenos Data Hub (reporting, analytics, golden-source data).
- **Financial crime** — Temenos Financial Crime Mitigation (AML, sanctions, KYC).
- **Wealth & fund administration** — Temenos Wealth, Temenos Multifonds.
- **Insurance** — Temenos Insurance (core insurance processing).
- **Delivery** — Temenos Banking Cloud (SaaS/platform), Temenos Exchange (partner
  marketplace), the Model Bank (pre-configured reference model), professional services.

This breadth — core + channels + payments + compliance + data on one platform — is the
company's core differentiator and the reason it wins "platform" deals rather than
point-product deals.

### 1.3 The three "flavors" of Temenos you will meet in the wild

1. **The traditional T24/Transact bank** — running Transact (or an old T24 release)
   on-premise, often customized, upgraded every few years with pain.
2. **The cloud-transition bank** — moving Transact to Temenos Banking Cloud (SaaS)
   or to a private cloud (AWS/Azure/GCP) run by the bank or an SI.
3. **The digital-bank build** — a greenfield build on Transact + Infinity + Banking
   Cloud in 6–18 months, using the Model Bank as the starting point (the GLDB/ANEXT
   case in `green_link_digital_bank_guide.md` is the canonical example).

---

## 2. Company History

### 2.1 Founding and the T24 era (1993–2001)

- **1993** — Temenos is founded in Geneva by **Jean-Pierre Brulé** (who remained CEO
  for three decades) with a small team of banking technologists. The founding insight:
  build a single, parameterized core-banking system that runs 24/7 across retail,
  corporate, and treasury, instead of the bespoke COBOL systems banks wrote themselves.
- **1990s** — The core system, **T24** ("Temenos 24", a reference to 24-hour
  availability), gains traction with mid-size banks and microfinance institutions in
  Europe, Africa, and the Middle East. T24's differentiator from day one: it is
  **configured, not coded** — products, rates, and charts of accounts are parameters,
  which made implementation dramatically faster than custom builds.
- **2001** — Temenos IPOs on the SIX Swiss Exchange (ticker TEMN), funding
  international expansion.

### 2.2 The consolidation years (2001–2018)

Temenos grew both organically and through acquisition, assembling the "full stack":

- **2007** — Acquires **jBASE** (the MultiValue database that underpins T24),
  bringing the database in-house.
- **2011–2012** — The **Misys episode**: Temenos agrees an all-share merger with UK
  rival Misys plc. **Vista Equity Partners** — then, as now, an aggressive enterprise-
  software PE firm — torpedoes the merger with a £1.27B rival bid for Misys, taking
  Misys private. (Misys later merged with D+H to become **Finastra**; see §15.)
  The irony: eleven years later Vista would try to buy Temenos itself.
- **2013** — Acquires **Sandstone Technology** (lending origination, Australia).
- **2015** — Acquires **FRSGlobal** (risk & regulatory reporting).
- **2016** — Launches **Temenos Infinity**, the omnichannel digital-banking platform.
- **2017** — Acquires **Multifonds** (fund administration), which becomes the Temenos
  fund-management franchise.
- **2018** — **T24 is rebranded Temenos Transact**, and the company publicly commits
  to open APIs and cloud delivery. (The T24 name lives on in the architecture and in
  the skills market — see §20.)

### 2.3 The cloud push and product expansion (2019–2022)

- **2019** — Acquires **Kony** (US low-code digital-experience platform) to bolster
  the digital front-end; announces **Temenos Financial Crime Mitigation** (FCM).
- **2020** — Launches **Temenos Banking Cloud** (the SaaS/platform offering) and
  acquires **Avoka** (digital account opening/onboarding, Australia).
- **2021** — Launches **Temenos Exchange** (partner marketplace) and **Temenos Data
  Hub** (the data/BI layer). The "Temenos platform" story — one platform, all
  products — becomes the marketing spine.
- **2022** — Deepens cloud partnerships: AWS (core-as-a-service), Microsoft Azure
  (banking on Azure), and regional clouds including **Huawei Cloud** for China and
  Southeast Asia deployments (the ANEXT/GLDB build — see `green_link_digital_bank_guide.md`).

### 2.4 The Vista bid that failed (Nov 2022 – 2023)

**Verified correction to the common narrative:** Temenos was *not* taken private by
Vista Equity Partners. The facts:

- **November 2022** — Vista announces an agreed cash offer of **CHF 130 per share**,
  valuing Temenos at roughly **CHF 5.1B (~US$5.3–5.5B)** — the figure sometimes quoted
  as "~$4.4B" is an incorrect or stale estimate. The deal was recommended by the
  Temenos board.
- **2023** — The offer is extended repeatedly while regulatory conditions fail to
  clear by the long-stop date, and **the bid lapses without completing**.
- **Evidence of non-completion:** Temenos remained listed on SIX (SWX:TEMN); in
  February 2024 Hindenburg Research shorted it as a *public* company at a ~$7.5B
  market cap; Temenos has published quarterly results continuously as SIX:TEMN
  through Q2 2026 (July 2026).
- Media reports in 2025 suggested Vista again explored a (lower-priced) approach;
  nothing completed. As of mid-2026 Temenos is still an independent, listed company.

Why it matters for this guide: several otherwise-reputable profiles of Temenos still
describe it as "a Vista portfolio company." That is wrong, and it changes how you
should read the "PE ownership concerns" debate in §16 — those concerns were live
during the 2022–23 bid process, but the company's strategy since 2024 has been set
by its own board, not by Vista.

### 2.5 The Hindenburg crisis and the CEO succession (2024)

- **15 Feb 2024** — Hindenburg Research publishes *"Temenos: Major Accounting
  Irregularities, Failed Products And An Aggressive Accounting Culture"* after a
  4-month investigation (25 former employees interviewed). The share price falls
  ~25–35% in a day. Hindenburg alleged, among other things, revenue backdating,
  questionable Indian deals, and executives cashing out ~$1.1B of stock over a decade.
- **Feb–Mar 2024** — Temenos rejects the report as "false and misleading," and the
  board removes founder-CEO **Jean-Pierre Brulé** (who had led the company since
  1993). **Andreas Andreades** (then chairman) serves as interim CEO.
- **Apr 2024** — An independent examination commissioned by the board concludes there
  was no evidence of improper revenue recognition; Hindenburg disputes the
  examination's scope and independence in a follow-up report.
- **1 May 2024** — **Jean-Pierre Brulard** (ex-Kepler-Cheuvreux CEO, ex-Oracle sales
  executive) takes over as CEO. Brulard pushes "cloud + AI" and subscription
  conversion (term licences → SaaS), visible in the 2024–25 financials: SaaS revenue
  grew ~30%+ while term licensing shrank.
- **4 Sep 2025** — Brulard steps down "with immediate effect" (the company cited the
  need for "new leadership to execute the next phase of the strategic plan"). CFO
  **Takis Spiliopoulos** is appointed interim CEO; the board (chaired by Thibault de
  Tersant) starts a permanent-successor search. As of mid-2026 Spiliopoulos remains
  in the role.

**CEO timeline summary:** Jean-Pierre Brulé (founder, 1993–2024) → Andreas Andreades
(interim, early 2024) → Jean-Pierre Brulard (May 2024–Sep 2025) → Takis Spiliopoulos
(CFO/interim CEO, Sep 2025–present).

### 2.6 Key milestones timeline

| Year | Milestone |
|---|---|
| 1993 | Founded in Geneva by Jean-Pierre Brulé |
| 1990s | T24 core banking system built and deployed |
| 2001 | IPO on SIX Swiss Exchange (TEMN) |
| 2007 | Acquires jBASE (the MultiValue database under T24) |
| 2011–12 | Misys merger collapses; Vista takes Misys instead (→ Finastra) |
| 2013 | Acquires Sandstone Technology (lending origination) |
| 2015 | Acquires FRSGlobal (regulatory reporting/risk) |
| 2016 | Launches Temenos Infinity (digital banking) |
| 2017 | Acquires Multifonds (fund administration) |
| 2018 | T24 rebranded **Temenos Transact**; open-API commitment |
| 2019 | Acquires Kony (low-code digital); launches FCM line |
| 2020 | Launches **Temenos Banking Cloud**; acquires Avoka (onboarding) |
| 2021 | Launches **Temenos Exchange** and **Temenos Data Hub** |
| 2022 | AWS/Azure/Huawei cloud partnerships; Vista bid announced (Nov) |
| 2023 | Vista bid lapses — Temenos stays listed |
| 2024 | Hindenburg report (Feb); Brulé exits; Brulard CEO (May) |
| 2025 | Brulard departs (Sep); CFO Spiliopoulos interim CEO; Transact R25 (AI-driven core) |
| 2026 | Q2 results: ARR +11%; continued SaaS/subscription push |

---

## 3. Product Portfolio at a Glance

| Product | What it is | Lineage / launched | Covered in |
|---|---|---|---|
| **Temenos Transact** | Core banking system (accounts, deposits, loans, payments, ledger) | ex-T24; rebranded 2018 | §4, §12 |
| **Temenos Infinity** | Omnichannel digital banking (portal, mobile, onboarding) | launched 2016; +Kony 2019, +Avoka 2020 | §5 |
| **Temenos Payments** | Payments hub — ISO 20022, real-time, SWIFT | T24 payments engine + modern hub | §6 |
| **Temenos Data Hub** | Data platform — golden source, reporting, BI, regulatory | launched 2021 | §7 |
| **Temenos FCM** | AML transaction monitoring, sanctions, KYC | launched 2019 | §8 |
| **Temenos Analytics** | BI/analytics front-end over banking data | pre-Data Hub analytics line | §9 |
| **Temenos Wealth** | Wealth management (front-to-back, private banking) | — | §9 |
| **Temenos Multifonds** | Fund administration (transfer agency, NAV) | acquired 2017 | §9 |
| **Temenos Insurance** | Core insurance processing | T24 insurance modules + suite | §9 |
| **Temenos Banking Cloud** | SaaS/PaaS delivery platform (containers, K8s, multi-cloud) | launched 2020 | §10 |
| **Temenos Exchange** | Partner marketplace / "app store" for banking apps | launched 2021 | §11 |
| **Model Bank** | Pre-configured reference bank model (industry + country) | T24 heritage | §12, §14 |
| **Temenos Open API** | REST API catalogue over the platform | 2018+ | §12, §19 |
| **Temenos Continuous Deployment** | DevOps/automated-upgrade tooling | cloud era | §14, §18 |

### 3.1 The "one platform" vision

Since the 2018 rebrand, Temenos has marketed **"one platform, all products"**: Transact,
Infinity, Payments, Data Hub, and FCM all share the Temenos platform layer (identity,
integration, data, API, deployment). The honest reading: they share *a* platform
(common tooling, common APIs, common cloud runtime), but Transact remains the
architecture with real depth, and the satellite products have varying degrees of
integration maturity — see the criticisms in §16.

### 3.2 Release naming and cadence (verified)

- Temenos products are released **annually** with a year-based R-number:
  **Transact R20, R21, R22, R23, R24, R25** (2020–2025). R25 (2025) is positioned as
  the "AI-driven core" release.
- There is no "Temenos 2026" product name per se; the scheme is `R<year>` for the
  core platform, with service packs in between. Temenos also markets integrated
  "release waves" across products.
- Banks on Banking Cloud get **continuous releases** (multiple per year) via Temenos
  Continuous Deployment; on-premise banks typically do one major upgrade per year.

---

## 4. Temenos Transact (Core Banking)

Transact is the flagship — the evolution of T24 — and the reason most banks buy
Temenos. In `core_banking_systems_guide.md` terminology (§3–§4), it is a **traditional
client-server-era core** that has been progressively modernized (open APIs,
containers, event streaming) rather than a from-scratch cloud-native core.

### 4.1 Functional scope: the "G20 core"

Transact covers the full retail + corporate banking book in one application family
(sharing one customer and one ledger):

- **Accounts & deposits** — current/savings/deposit products, term deposits, interest
  conditionality (tiered rates, penalty rules), account sweeps.
- **Loans** — consumer, mortgage, corporate, syndicated; amortization engines;
  NPL/collections support; Islamic finance (Murabaha, Ijara, Mudaraba) — Temenos has a
  substantial Islamic-banking franchise in the Gulf and Southeast Asia.
- **Payments** — domestic, cross-border, real-time, ISO 20022-native payment
  processing inside the core (and via Temenos Payments for hub-style deployments).
- **Ledger** — the multi-currency, multi-company general ledger with full
  double-entry; the "golden source" of the balance sheet.
- **Treasury & markets** — FX, money market, derivatives back-office for corporate
  treasury (this is where it overlaps the trading domain covered in
  `nasdaq_calypso_guide.md` — but Calypso and Transact coexist in real banks: Calypso
  for the trading floor, Transact for the retail/commercial book).
- **Customer** — the CUSTOMER application as the master-data hub (KYC-adjacent),
  plus limits, collateral, guarantees.
- **Channels/back-office** — Teller, COB (Close of Business), statements, collections.

### 4.2 The Transact architecture: heritage and modernization

Two layers of story here — the classic T24 architecture (still what most of the
installed base runs) and the modernized "cloud-native-adjacent" Transact. Both are
detailed in §12. In one paragraph:

- **Classic T24** = multi-tier: (a) presentation (T24 Browser / Designer clients, or
  any channel via OFS/Web Services), (b) the **application server** running the T24
  runtime, (c) the **database** — historically **jBASE**, a **MultiValue** (Pick-style)
  database acquired in-house in 2007, with Oracle/DB2/SQL Server supported as
  alternatives on larger installs, and (d) the **COB scheduler** for batch (Close of
  Business). All business logic is written in **T24 BASIC** (a BASIC dialect with
  MultiValue string handling) and stored in the database as part of the application
  definitions.
- **Modern Transact** (R2x, cloud era) = the same core wrapped in: a **REST API layer**
  (Temenos Open API), an event/streaming layer (Kafka-based integration in recent
  releases), **PostgreSQL** as a first-class supported database (announced for R23,
  aligning the core with cloud economics), and **containerized deployment** on
  Kubernetes via Banking Cloud. It is "microservices-adjacent": the platform is
  decomposed into deployable services (banking services, APIs, data services), but
  the core engine itself remains a shared, stateful application — not a set of
  independent microservices in the Thought Machine Vault sense (§15).

### 4.3 Transact vs T24: what actually changed in 2018

The 2018 rebrand was more than a name change:

| Aspect | T24 (pre-2018) | Transact (R18–R25) |
|---|---|---|
| Integration | OFS files/XML, SOAP web services | + full REST (Open API), events |
| Deployment | On-premise, monolithic app server | + containers, Kubernetes, SaaS |
| DB | jBASE (default), Oracle/DB2 | + PostgreSQL (R23+) |
| UI | T24 Browser (classic) | + Infinity as the channel; T24 Browser retained for ops |
| Upgrades | Big-bang, painful | + Temenos Continuous Deployment (automated, SaaS) |
| Development | Designer (T24 BASIC) | + Workbench (DevOps-oriented), CI/CD pipelines |

### 4.4 Who runs Transact

Representative (publicly-known) Transact/T24 anchors across regions: **Standard
Chartered** (multi-country retail), **Emirates NBD** and **Mashreq** (UAE), **Al Rajhi
Bank** (Saudi Arabia — one of the largest Islamic-banking implementations), **UnionBank
of the Philippines**, **ANEXT Bank** (Singapore digital bank, on Huawei Cloud — see
`green_link_digital_bank_guide.md`), plus hundreds of mid-size banks, microfinance
institutions, and Islamic banks across Africa, the Middle East, and Asia. (These are
drawn from Temenos press announcements over the years; a full client list is not
not publicly maintained).

---

## 5. Temenos Infinity (Digital Banking)

Infinity is Temenos's **omnichannel digital-banking platform** — the portal/mobile
front-end that sits in front of Transact (or, in practice, in front of other cores
too, since Temenos sells it standalone to non-Temenos banks). Launched 2016, it has
850+ clients (Temenos, Nov 2022) and is one of the most-installed digital banking
platforms globally.

### 5.1 What Infinity does

- **Digital channels** — retail and corporate internet banking, mobile banking,
  tablets, APIs for third parties; one omnichannel session model (start on mobile,
  finish on web).
- **Customer journeys / onboarding** — account opening, KYC capture, product
  origination; the **Avoka** acquisition (2020) added best-in-class digital account
  opening; the **Kony** acquisition (2019) added low-code app development.
- **Product configuration** — non-technical staff configure journeys, widgets,
  content, and channel availability ("config over code" carried into the front-end).
- **Lifecycle** — marketing/engagement, offers, and cross-sell journeys on top of
  the banking APIs.

### 5.2 Infinity architecture

- **Micro-frontends and composable UI**: Infinity is built on a component model —
  journey widgets that can be composed per channel and per segment (retail vs SME vs
  corporate). This is the "composable" play on the channel side; see the composable
  debate in `core_banking_systems_guide.md` §6.
- **API-driven**: Infinity consumes banking APIs (Temenos Open API, or any bank API)
  rather than owning business logic; it is deliberately core-agnostic.
- **Design system + low-code**: theming/white-labelling per brand; the Kony heritage
  gives it a low-code development surface for custom journeys.
- **Infinity Marketplace** — an in-product storefront of pre-built journeys/widgets
  (part of the broader Exchange ecosystem, §11).

### 5.3 Infinity vs "core-led" digital

A common architecture question: *should the digital layer be the core's own UI
(T24 Browser/Transact screens) or a separate platform like Infinity?* The Temenos
answer is both: **Infinity for customers** (and increasingly for staff portals),
**T24 Browser/Transact screens for back-office operations**. Real Temenos banks
almost always end up running Infinity (customer-facing) alongside Transact ops
screens (staff-facing), which is exactly the pattern described in
`core_banking_systems_guide.md` §2.6 (front office vs back office).

---

## 6. Temenos Payments (Payments Hub)

Temenos Payments is the **payments hub** product — for banks that want payments
decoupled from the core (the classic hub pattern in `payments_hub_guide.md`) or that
need a modern ISO 20022-native payments layer even where the core is not Temenos.

### 6.1 Capabilities

- **ISO 20022-native** — pain/pacs/camt message handling end-to-end (see
  `iso_20022_core_processes_guide.md` for the message model); CBPR+ cross-border
  support; UETR/GPII tracking.
- **Real-time payments** — connectivity to instant schemes: SEPA Instant (SCT Inst),
  Singapore FAST, India UPI (via partner rails), US RTP/FedNow (via partners); fraud
  screening on the payment path.
- **SWIFT connectivity** — SWIFT Alliance/cloud integrations, GPI, cover payments.
- **Payment orchestration** — routing, format transformation, exceptions
  (camt.056/camt.029), returns (pacs.004), investigations, reconciliation.
- **Hub or embedded** — deployable as a standalone hub in front of any core, or as
  the payment engine inside Transact. Temenos often sells it as the "payments
  modernization" entry point to non-Temenos banks.

### 6.2 Relationship to Transact

Historically, payments were a module of T24 (the payment orders, FT/MT messages, and
the "CUSTOMER+ACCOUNT+payment" flows). Modern Temenos positions **Temenos Payments as
the hub** with Transact as the ledger — the hub owns the ISO 20022 lifecycle, the core
owns the postings. For the payments-hub build-vs-buy analysis and the Singapore
payments context, see `payments_hub_guide.md` (§12 vendors, §14 Singapore).

### 6.3 2025 product news

At Sibos 2025 Temenos launched **Temenos Money Movement & Management** — an
AI-powered, pre-integrated platform combining payments and account services,
targeting the mid-market/corporate space. It signals where Temenos is heading:
payments + accounts as a packaged, AI-assisted SaaS product rather than a
multi-year core project.

---

## 7. Temenos Data Hub (Data Layer)

Data Hub (launched 2021) is Temenos's **data management and BI layer** — the
"golden source" data platform that sits alongside (not inside) the core.

### 7.1 What it does

- **Data aggregation** — pulls from Transact and non-Temenos systems into a
  unified banking data model; the "single version of the truth" for reporting.
- **Reporting & regulatory** — financial statements (IFRS/GAAP), regulatory returns,
  and (via the FRSGlobal lineage, acquired 2015) risk and regulatory reporting
  automation.
- **BI/analytics** — dashboards and analytics over banking data; historically this
  was the separate **Temenos Analytics** product (see §9), now folded into the
  Data Hub story.
- **Data services** — APIs to serve downstream consumers (data warehouse, risk
  systems, data science/AI workloads), plus data-quality tooling.

### 7.2 Architectural position

Data Hub is a **sidecar, not a replacement**: the core remains the system of record;
Data Hub is the reporting/analytics substrate. In a typical bank landscape it sits
where `core_banking_systems_guide.md` §2.5 puts the "data platform" — between the core
and the BI/regulatory consumers — and it competes at the margin with bank data
warehouses (Snowflake, Teradata) and BI stacks. Its differentiator: the Temenos data
model and pre-built content for Transact customers (faster to stand up than a
custom warehouse, less flexible than a general-purpose one).

---

## 8. Temenos Financial Crime Mitigation (FCM)

FCM (launched 2019) is Temenos's **financial-crime suite**: AML transaction
monitoring, sanctions screening, KYC/CDD, and case management — sold standalone or
bundled with Transact.

### 8.1 Modules

- **AML transaction monitoring** — rules + scenario-based monitoring of Transact (and
  non-Temenos) payment/account flows; alert generation, case management, SAR/STR
  workflows.
- **Sanctions screening** — name/entity screening against sanctions lists (UN, OFAC,
  EU, MAS), with fuzzy matching and list management.
- **KYC / CDD** — customer due diligence, risk scoring, periodic review workflows,
  integrated with the core's CUSTOMER data.
- **FCM on the payment path** — real-time screening inside Temenos Payments for
  instant-scheme transactions (FAST, SEPA Instant), where screening latency is
  critical (see `payments_hub_guide.md` §9 on fraud/risk in the hub).

### 8.2 Positioning

For a bank building its AML estate, FCM competes with specialist vendors (FICO
Falcon, NICE Actimize, SAS, Oracle OFS) that are deeper in detection science — but it
wins on **integration economics**: no separate data pipeline from the core, no
duplicate customer master. The regulatory context (MAS, and the broader APAC AML
regime) is covered in `financial_risk_compliance_systems_guide.md`; FCM is the
Temenos-native answer to that problem space. Banks with serious, regulator-driven AML
programs still typically run a best-of-breed specialist — the "full-stack" pitch
wins more often at mid-size banks than at tier-1s.

---

## 9. Temenos Analytics, Wealth, Fund Management, Insurance

### 9.1 Temenos Analytics

The BI/analytics product line that predates Data Hub: dashboards, KPIs, and
analytics content (customer profitability, product performance, channel analytics)
over banking data. For Transact banks it is the out-of-the-box analytics layer; for
large banks it tends to lose out to dedicated BI platforms (PowerBI, Tableau,
Looker) fed from the data warehouse. In current marketing it is largely absorbed
into the Data Hub story (§7).

### 9.2 Temenos Wealth

Temenos Wealth is the **wealth-management front-to-back suite**: portfolio
management, advisory, order management, and client reporting — aimed at private banks
and wealth managers, often on top of Transact accounts. It competes with Avaloq
(the dominant Swiss/European wealth platform — see the vendor table in
`core_banking_systems_guide.md` §5.8) and with best-of-breed wealth tech. Temenos
positioning: wealth + core + digital on one platform, especially attractive in
Asia/Middle-East private-banking builds.

### 9.3 Temenos Multifonds (Fund Management)

From the 2017 Multifonds acquisition: **fund administration** — NAV calculation,
transfer agency, investor accounting, fund order routing — for asset managers and
fund administrators, delivered increasingly as SaaS. It is Temenos's answer in the
fund-services domain and is a genuinely separate franchise from the core-banking
business (competes with SS&C, FIS Investran, and similar).

### 9.4 Temenos Insurance

Temenos Insurance provides **core insurance processing** (policy admin, claims) built
on the same platform family, aimed at bancassurance and general insurers. It is the
smallest and least prominent line of the portfolio — in practice most Temenos banks
buy the banking stack and keep insurance on specialist systems (Duck Creek, Guidewire,
SAP). Mentioned here for completeness; treat it as a niche extension rather than a
strategic pillar.

---

## 10. Temenos Banking Cloud (SaaS)

**Temenos Banking Cloud** is the SaaS/PaaS delivery platform — the vehicle for
Temenos's strategic pivot from licence+implementation to subscription+service. It
turns the whole portfolio (Transact, Infinity, Payments, Data Hub, FCM) into a
managed, multi-tenant-able banking platform. Launched 2020; 700+ banks on it by 2022
(Temenos factsheet); the default delivery model for all new greenfield deals.

### 10.1 What it provides

- **Managed SaaS** — Temenos (or a certified partner) runs the stack: patching,
  upgrades, monitoring, 24/7 operations, security, resilience. Banks consume
  "banking services" rather than administer servers.
- **Self-service provisioning** — sandboxes, environments, and banking services
  provisioned on demand (the "click-of-a-button" positioning at Temenos Community
  Forum events).
- **Continuous delivery** — Temenos Continuous Deployment automates upgrades,
  including regression testing, which is the strategic answer to the historic
  "Temenos upgrades are painful" criticism (§16).
- **Marketplace access** — Banking Cloud subscribers get one-click integration with
  Exchange partner apps (§11).

### 10.2 Cloud substrate (verified)

Temenos Banking Cloud is **multi-cloud**:

- **AWS** — flagship partnership; Temenos core-banking-as-a-service on AWS (2022),
  with ESG/carbon-footprint claims vs on-premise.
- **Microsoft Azure** — supported; Temenos published performance benchmarks on Azure
  (with MongoDB Atlas for some data services).
- **Google Cloud** — supported for Banking Cloud deployments.
- **Huawei Cloud** — used for China and select Southeast Asia deployments; the ANEXT/
  GLDB digital-bank build in Singapore runs Transact on Huawei Cloud (see
  `green_link_digital_bank_guide.md`).

### 10.3 Reference architecture (high level)

`Browser/Infinity → API gateway (Open API) → banking services (Transact, Payments,
Data Hub, FCM as containers) → Kubernetes cluster → managed Postgres/Oracle (or
jBASE in transition) on cloud → cloud-native services (Kafka event bus, object
storage, secrets, monitoring)`. The platform is containerized and orchestrated by
Kubernetes, with databases as managed services. It is "cloud-native-adjacent":
containerized and API-first, but the core remains the shared stateful Transact
engine — not a decomposed microservices core (contrast with OBMA in
`oracle_banking_microservices_architecture_guide.md` §4, and with Vault in §15).

### 10.4 Economics

The financials show the pivot in action: FY2024 SaaS revenue grew ~30%+ while term
licensing declined ~48% (Q2 2024), ARR grew 12% cc to $804M (FY2024), and the
company now guides on **ARR** as the headline metric. Under the Brulard/Spiliopoulos
leadership the message is consistent: subscription conversion is the growth engine,
legacy maintenance (~$250M/yr, ~18% of revenue per FY2025 analyses) is the cash cow
being deliberately converted.

---

## 11. Temenos Exchange (Marketplace)

**Temenos Exchange** (launched 2021) is the partner marketplace — the "app store"
for banking apps: third-party fintech solutions pre-integrated with the Temenos
platform, available to banks via Banking Cloud or on-premise deployments.

### 11.1 What it is

- **Partner ecosystem** — hundreds of fintech partners (payments, lending, credit
  scoring, fraud, KYC/identity, analytics, FX, treasury) with certified
  integrations to Temenos Open API.
- **Certification** — partners are vetted and their integrations tested against
  Temenos APIs, reducing bank-side integration risk (vs. buying from a fintech and
  doing the integration yourself).
- **Distribution** — banks subscribe to apps through the marketplace; billing and
  provisioning are platform-managed.
- **Monetization** — Temenos takes a revenue share; partners get distribution to
  1,000+ banks. For Temenos it is the **composability play**: the answer to
  "Temenos is a monolith" is "you can compose best-of-breed apps from Exchange
  without leaving the platform."

### 11.2 Exchange and the composable-banking debate

See the composable debate in `core_banking_systems_guide.md` §6. The two camps:
cloud-native cores (Vault, Mambu) are composable *by construction*; traditional
vendors (Temenos, Oracle, Finastra) are composable *by extension* — the core stays
whole, and everything around it becomes pluggable via APIs and marketplaces.
Exchange is Temenos's bet that banks want the latter: a proven, deep core plus a
curated ecosystem, rather than assembling a core from parts. The critique: Exchange
apps are mostly channel/edge capabilities, not core capabilities — the parts of the
stack a bank might genuinely want to replace (ledger, product engine) are exactly
the parts Exchange does not offer.

---

## 12. The Transact Architecture in Depth

This is the section that matters most to architects: what Transact *is* under the
hood, and what has changed (and not changed) in the cloud era.

### 12.1 The MultiValue heritage: jBASE and the T24 data model

T24 was born in the MultiValue (Pick) tradition. **jBASE** — a MultiValue database
whose origins lie in the Pick operating system lineage — was T24's native data store,
and Temenos bought the company in 2007 to own it. What "MultiValue" means in
practice:

- Records are stored in **dynamic files**; a field can hold a **single value or a
  repeated list of values** (a customer's multiple phone numbers, an account's
  multiple signatories, a product's multiple rate tiers). This makes the data model
  compact and flexible — and it is the root of both T24's agility and its
  "not like a normal database" learning curve.
- There is **no fixed schema** in the relational sense: the schema *is* the
  application definitions (see below). This is why T24 customizations so rarely
  require database migrations — you add a field to an application definition, not
  an `ALTER TABLE`.
- Relational databases (Oracle, DB2, SQL Server) were later made supported
  alternatives on larger installs (the "T24 on Oracle" deployments), and
  **PostgreSQL became a first-class supported database in R23+** — the cloud-era
  default, because it is the database you can run cheaply and managed on AWS/Azure/
  GCP/Huawei. jBASE remains supported and still underpins many existing installs,
  especially smaller ones.

### 12.2 The T24 building blocks

Everything in T24/Transact is defined in terms of a small set of primitives. If you
understand these five, you understand T24:

| Building block | What it is | Analogy |
|---|---|---|
| **Application** | A file definition: record structure + field dictionary + processing rules (e.g., `CUSTOMER`, `ACCOUNT`, `ACCOUNT.CLASS`, `TELLER`, `FT` payments). Applications are the "tables". | Schema + stored procedures |
| **Version** | A screen layout over an application (e.g., `CUSTOMER,INPUT.DEFAULT`); defines which fields show, in what order, with what validation, and what happens on input (the "I" processing — deal slips, authorization flows). | Form/view + input handler |
| **Enquiry** | A query/report definition over applications; the T24 query language with selection criteria, sorting, and output formatting. Enquiries are the standard way to get data out (and power many of the screens). | SQL view / report |
| **Routine** | A subroutine written in **T24 BASIC** (the BASIC dialect with MultiValue string handling); the unit of custom business logic. | Stored procedure / service |
| **Table / reference data** | System-level and bank-level reference data (products, interest conditions, rates, chart of accounts, company parameters, holiday calendars). | Configuration database |

Around these sit the operational machinery:

- **COB (Close of Business)** — the batch/scheduler that runs the overnight cycle
  (interest accrual, posting, statement generation, regulatory files). In classic
  T24 this is a *daily* batch window; cloud-era Transact pushes more of this to
  real-time and to the event bus.
- **OFS (Open Financial Service)** — the classic file/XML-based integration channel
  used to push transactions in and out (OFS messages). Still widely used; now
  supplemented by REST and events.
- **Web services / IRIS** — the SOAP-to-modern integration layer. **IRIS** is
  Temenos's integration framework that wraps core services for external consumption
  (and is the ancestor of parts of the current API layer). In R2x releases the
  integration surface is REST-first (Open API) with **Kafka** event streaming for
  asynchronous integration.
- **Multi-company, multi-currency** — a single Transact instance hosts many
  "companies" (legal entities/branches) and all currencies; the ledger consolidates
  across them. This is a core architectural fact that makes Transact a natural fit
  for banks with subsidiaries (and a complexity driver for consolidation reporting).

### 12.3 The Designer and the Workbench (development environments)

- **Designer** — the classic T24 development environment: browse applications,
  create versions/enquiries/routines, edit T24 BASIC. For two decades, "T24
  development" meant Designer.
- **Workbench** — the modern (R2x) developer tooling: a single entry point for
  configuring Transact modules with SDLC/DevOps support (source control, builds,
  CI/CD pipelines). Workbench is the tool for teams that treat Transact
  customization as software engineering rather than configuration.

### 12.4 The Model Bank

The **Model Bank** is Temenos's pre-configured reference implementation: a complete,
parameterized bank — products, rates, charts of accounts, processes, and
localizations — shipped with every Transact release. Implementation teams **start
from the Model Bank and configure, rather than build from a blank system**. There
are industry models (retail, corporate, wealth, Islamic, microfinance) and country
models (local regulatory content, tax, reporting formats — Temenos markets country
models across 100+ jurisdictions). The Model Bank is the single biggest reason
Temenos implementations are faster than bespoke builds — and the source of the
"config over code" philosophy in §14.

### 12.5 The classic T24 runtime topology

```
┌──────────────────────────────────────────────────────────────┐
│ Channels: T24 Browser │ Infinity │ ATM/ISO │ OFS │ APIs      │
└──────────────────────────────┬───────────────────────────────┘
                               ▼
┌──────────────────────────────────────────────────────────────┐
│ Application server (T24/Transact runtime)                     │
│  - T24 BASIC routines (business logic)                        │
│  - Version/input processing, enquiries, COB scheduler         │
│  - OFS / Web Services / IRIS / REST / Kafka adapters          │
└──────────────────────────────┬───────────────────────────────┘
                               ▼
┌──────────────────────────────────────────────────────────────┐
│ Database: jBASE (MultiValue) │ Oracle │ DB2 │ PostgreSQL      │
└──────────────────────────────────────────────────────────────┘
```

### 12.6 Modern Transact: what "cloud-native" really means here

The honest label is **containerized and API-first, not microservices-native**:

- **Containers/Kubernetes** — Transact and its satellite products run as containers
  orchestrated by Kubernetes (the Banking Cloud substrate, §10.3).
- **Open API layer** — the **Temenos Open API** is a REST catalogue over the core:
  accounts, customers, payments, limits, transactions, origination. Temenos markets
  it as hundreds of APIs across dozens of API groups (the marketing language is
  "40+ API groups" — treat the exact count as marketing, the direction as fact).
  The Open API is also the basis of the open-banking/PSD2 story (§19).
- **Event-driven** — Kafka-based event streaming (R23+) for real-time integration:
  postings, payments, customer events published to the bus for downstream systems
  (fraud, data hub, channels). This is the modernization that lets Transact behave
  like a modern event source without rebuilding the engine.
- **What hasn't changed** — the core engine is still a shared stateful application
  with a centralized ledger; "microservices-adjacent" means the *platform* around
  the core is decomposed, not the core itself. Compare OBMA
  (`oracle_banking_microservices_architecture_guide.md` §4), which decomposed the
  banking services themselves, and Vault (§15), which is decomposed by design.

### 12.7 Performance and scale notes

- Transact is proven at very large scale (top-tier banks in the Gulf and Asia run
  tens of millions of accounts per instance); benchmarks published for Banking
  Cloud (e.g., Azure/MongoDB Atlas, AWS) claim throughput in the thousands of
  TPS with sub-second response for core operations.
- The traditional bottlenecks are the COB window and the database; modern
  deployments mitigate with event-driven postings (post to the bus in real time,
  settle in the ledger), parallel COB, and managed databases.
- Multi-tenancy: Banking Cloud supports logical multi-tenancy (isolated schemas/
  environments per bank) rather than shared-table multi-tenancy — banks keep
  physical/logical isolation, which matters for regulators.

---

## 13. Deployment Models

| Model | Who runs it | Typical buyer | Notes |
|---|---|---|---|
| **On-premise** | The bank (with SI support) | Existing T24 banks, regulated jurisdictions requiring local hosting | Classic multi-tier install; full control; upgrade burden on the bank |
| **Private cloud (bank-hosted)** | The bank on AWS/Azure/GCP/Huawei (or SI-managed) | Banks wanting cloud economics with control; data-residency-driven | Containerized Transact, managed DB, bank's own DevOps |
| **Managed/cloud (Temenos SaaS)** | Temenos (or certified partner) on Banking Cloud | New deals, digital banks, banks exiting data-center operations | Subscription pricing; continuous upgrades; 700+ banks |
| **Hybrid** | Mixed | Groups with subsidiaries at different maturity | Core on SaaS + on-prem satellites, or vice versa |

Deployment-model decisions in practice:

- **Regulatory/data residency** is the dominant constraint in Asia (MAS guidelines,
  India RBI, China) — see §21.
- Temenos's commercial direction is unambiguous: **Banking Cloud is the default
  offer**; on-premise is priced/positioned as the legacy or special-case option.
- For an architect, the meaningful split is *who owns the upgrade* — SaaS (Temenos
  upgrades continuously, bank tests against sandboxes) vs. self-hosted (bank runs
  its own release cycles, §18).

---

## 14. The Implementation Model

### 14.1 Config-over-code: the philosophy

T24/Transact is **parameterized by design**: products, pricing, interest conditions,
charts of accounts, authorization matrices, and even many processes are
*configured*, not coded. The ideal implementation is: take the Model Bank, switch on
the products and countries you need, configure your rates and chart of accounts,
localize, integrate, and go live. Custom code (T24 BASIC routines, custom versions/
enquiries) is the exception that should be minimized — every line of customization
is a line you must re-test and re-migrate on every upgrade.

### 14.2 The upgrade challenge and the "regression-free" promise

The historic criticism is real: **Temenos upgrades are painful** because the
installed base is heavily customized, and every upgrade is a regression-testing and
re-migration exercise. Temenos's strategic answer:

- **On Banking Cloud**: **Temenos Continuous Deployment** automates upgrades —
  build, regression-test, and deploy new releases continuously, with the vendor
  absorbing the upgrade engineering. The promise: banks on SaaS are always on the
  latest release ("banks using Temenos Core are always able to upgrade to the
  latest version" — Temenos product page).
- **On-premise**: annual major releases (R2x) + service packs; the bank owns the
  regression burden. The "no custom code" ideal exists precisely because
  customization is the cost driver of upgrades.
- **Reality**: the "regression-free upgrade" promise holds best for banks that
  stayed close to vanilla; heavily customized T24 shops still face multi-month
  upgrade programs. This is the single most important commercial/technical tension
  in the Temenos story, and it is the main reason the vendor pushes so hard toward
  SaaS (where it controls the customization surface).

### 14.3 Methodology and partners

- Temenos delivers through its own professional services plus a deep **SI
  ecosystem**: Accenture, Deloitte, Capgemini, IBM, KPMG, PwC, and regional SIs
  (TCS, Wipro, Cognizant, plus local specialists in each market). Most tier-1
  implementations are led by Accenture or Deloitte with Temenos in a product role.
- The delivery methodology is Model Bank-driven: *fit-gap analysis against the
  Model Bank → configure → prototype (accelerated by the pre-built model) →
  integrate → migrate → test → cutover*. Temenos markets significantly compressed
  timelines vs. custom builds (the ANEXT/GLDB digital bank went live in **11
  months** — see `green_link_digital_bank_guide.md`).
- Sizing reality: the *licence* is the headline price, but the *implementation
  services* (often 2–5x the licence) and the *SI ecosystem* are where the money
  goes. Total cost of ownership is discussed in §18.

---

## 15. Competitive Landscape

The core-banking vendor market and the full vendor table are covered in
`core_banking_systems_guide.md` §5; this section is the Temenos-specific comparison.

### 15.1 Head-to-head map

| Competitor | Type | Temenos's edge | Their edge over Temenos |
|---|---|---|---|
| **Oracle (FLEXCUBE / OBMA)** | Traditional core + modernized microservices suite | Breadth, Model Bank, Europe/MEA installed base, cloud momentum | Asia installed base (FLEXCUBE), OBMA's genuinely decomposed microservices architecture (`oracle_banking_microservices_architecture_guide.md`) |
| **Finastra (Misys lineage)** | Traditional core (Fusion Essence) + lending/payments | Transact depth, one-platform story, banking-cloud maturity | Pricing aggression, lending/payments point strengths, Fusion Essence cloud-native variant |
| **Thought Machine Vault** | Cloud-native core (UK) | Functionality depth, 30 years of banking content, global footprint | True cloud-native/microservices design, "core as code", developer experience, no upgrade problem |
| **Mambu** | SaaS core (Netherlands) | Enterprise scale, corporate/treasury breadth, on-prem option | Composable SaaS model, speed to market for digital banks, API-first DNA |
| **FIS (Profile/Modern Banking Platform)** | US-centric cores | Global footprint, retail+corporate depth | US market dominance, banking-as-a-service |
| **Fiserv (DNA, Signature)** | US-centric cores | Global, multi-country multi-company | US credit union/regional bank dominance |
| **Infosys Finacle** | Traditional core (India) | Product depth, Europe/MEA/Islamic | India/EMEA price-performance, local ecosystem |
| **TCS BaNCS** | Traditional core (India) | Same | Same — India/EMEA, cost-effective scale |
| **Avaloq** | Wealth/private-banking core (Swiss) | Full-stack retail+corporate+wealth | Wealth depth, Swiss private-banking dominance |
| **Apache Fineract** | Open source | Support, depth, compliance content | Zero licence cost, open data model (`apache_fineract_guide.md`) |

### 15.2 The classic rivalry: Temenos vs Oracle

The most storied rivalry in core banking (detailed in
`oracle_banking_microservices_architecture_guide.md` §15). Roughly: **Temenos is
stronger in Europe, the Middle East, and Africa; FLEXCUBE is stronger in Asia** —
the two split most large core-replacement deals globally. The rivalry now has a
second act: Oracle's OBMA decomposed its suite into real microservices, while
Temenos modernized in place (containers + APIs + events around the same engine).
Both are converging on the same message (SaaS, AI, ISO 20022, cloud), which is
exactly why the "era" framing in `core_banking_systems_guide.md` §3 matters: the
incumbent-vs-incumbent fight is increasingly about modernization credibility, and
both vendors are vulnerable to the cloud-native challengers below.

### 15.3 The cloud-native challengers: Vault and Mambu

- **Thought Machine Vault** — built from scratch as a cloud-native core (SQL/Python
  product logic, no batch, no legacy). It wins digital-bank and greenfield deals
  (e.g., several UK/EU digital banks) and increasingly tier-2 conversions. Its
  ceiling is functional depth: Vault does not have 30 years of banking content, so
  complex corporate/treasury/Islamic requirements still favor Transact.
- **Mambu** — the SaaS composable core, strongest in Europe/EMEA digital banking and
  lending; wins where speed and API-first matter more than depth.
- The pattern to watch: challengers win the *agility* narrative and the first
  greenfield wave; Temenos wins the *depth and global footprint* argument and the
  conversion deals. Every serious RFP now includes at least one of Vault/Mambu,
  which was unthinkable a decade ago. See the vendor table and the "picking a
  vendor archetype by bank profile" section in `core_banking_systems_guide.md` §5.15.

### 15.4 The US gap

Temenos's historical weakness is **North America**, where FIS/Fiserv dominate and
where Temenos's retail-core share is small (its US presence leans on Infinity
digital, FCM, and wealth). The IDC MarketScape North America digital-core
assessment (2024) named Temenos a Leader, but the US core market remains FIS/Fiserv/
Oracle territory. This matters for any global strategy discussion: Temenos is the
global #1 by installed base, but not the US #1.

### 15.5 The open-source flank

Apache Fineract (see `apache_fineract_guide.md`) attacks the low end: free core
banking for microfinance and digital banks, which is precisely the segment where a
smaller Temenos deals used to win. Temenos's answer is the Model Bank + SaaS
economics + the microfinance industry model; Fineract's answer is zero licence cost
and an open data model. In practice they rarely meet in the same RFP (Fineract
shops cannot pay for Temenos; Temenos shops cannot tolerate Fineract's support
profile) — but Fineract keeps pricing pressure on Temenos's smallest deals.

---

## 16. Strengths and Criticisms

### 16.1 Strengths

1. **Market share and scale** — the largest core-banking installed base in the
   world: 1,000+ FIs, 150+ countries, and the reference list that comes with it
   (Euromoney "world's best core banking solution" 2025).
2. **Full-stack breadth** — core + digital + payments + data + FCM + wealth on one
   platform. No other vendor offers the same span with a single commercial
   relationship; this wins platform deals and makes point-solution competitors
   fight uphill.
3. **The Model Bank** — the pre-configured reference model is genuinely
   differentiated: it compresses implementation from years to months and is the
   reason Temenos can credibly sell "go live in 11 months" digital-bank builds.
4. **Global footprint + localization content** — country models, regulatory
   reporting, Islamic banking, multi-company/multi-currency — a bank with
   subsidiaries in 20 countries can run them on one platform.
5. **Banking Cloud momentum** — 700+ banks on SaaS, continuous upgrades, multi-cloud
   (AWS/Azure/GCP/Huawei); the subscription pivot is working financially (ARR +12%
   FY24, +11% Q2 2026).
6. **The Exchange ecosystem** — the marketplace gives banks a curated composability
   story and gives Temenos an ecosystem moat that point-product vendors lack.

### 16.2 Criticisms

1. **"Not truly cloud-native"** — the core is a 1990s MultiValue application
   containerized and API-wrapped, not designed as microservices. Architects who
   value clean decomposition (and who read the OBMA guide or work with Vault) find
   the "cloud-native" marketing overstated. This is a fair criticism of the *core*;
   the *platform* around it is genuinely cloud-native.
2. **Upgrade pain** — the annual release + regression burden on customized,
   self-hosted installs is real and well documented; the "always on latest version"
   promise applies mainly to SaaS customers. Customization is the tax.
3. **Cost** — licence + implementation services (often 2–5x licence) + annual
   maintenance make Temenos one of the most expensive options, especially for
   mid-size banks; SaaS conversion changes the shape (subscription) but not the
   level of spend.
4. **Legacy tech perception** — T24 BASIC, jBASE, COB and the Designer toolset are
   "old technology" in hiring and boardroom conversations; talent pools skew
   toward veteran T24 consultants rather than new graduates (see §20).
5. **Governance history** — the Hindenburg report (2024) and the chaotic 2022–23
   Vista bid period damaged the governance narrative; the two CEO changes in 18
   months (2024–2025) plus interim leadership through 2026 keep strategy questions
   live. (Note: the "PE ownership" criticism is often mis-stated — Vista's bid
   failed, so Temenos is *not* a PE-owned company; the accurate criticism is about
   activist/board turmoil and short-seller allegations, plus the *threat* of PE
   re-approaches at depressed valuations.)
6. **Challenger pressure** — Vault/Mambu win the "modern" narrative; every Temenos
   renewal now faces a cloud-native alternative in the shortlist.

### 16.3 The balanced read

Temenos is best understood as **the deep, proven, expensive incumbent that is
modernizing seriously**: the product breadth and reference list are unmatched, the
SaaS/AI pivot is real and financially visible, and the architecture is being
modernized in place rather than rebuilt. Its risk profile is execution (upgrades,
customization tax, talent) and narrative (cloud-native challengers, governance
history) rather than product survival — Temenos will be a top-3 core vendor for the
foreseeable future, but it will increasingly share the market with the cloud-native
wave (see `core_banking_systems_guide.md` §3 and §6 for the era and composability
framing).

---

## 17. Banking Context: How Banks Use Temenos

### 17.1 The three bank archetypes

1. **The traditional T24 bank** — a long-standing on-premise T24/Transact customer,
   often heavily customized, upgraded every few years by an SI. This is the bulk of
   the 1,000+ installed base and the source of maintenance revenue. Their problems:
   upgrade cost, technical debt, talent retention.
2. **The cloud-transition bank** — moving to Banking Cloud (or private cloud) to
   convert capex to opex, get continuous upgrades, and shed data-center operations.
   This is Temenos's strategic growth engine and the commercial reason for the
   SaaS pivot.
3. **The digital-bank build** — greenfield: Model Bank + Transact + Infinity +
   Banking Cloud, live in 6–18 months. The ANEXT/GLDB case (Temenos on Huawei
   Cloud, 11 months) is the canonical example in `green_link_digital_bank_guide.md`;
   the same playbook has been repeated across Asia, Africa, and Latin America.

### 17.2 The "system of record" pattern

In most Temenos banks, Transact is the **system of record** — the authoritative
ledger and customer/account master — while everything else (channels, risk, fraud,
data warehouse, payments hub, digital) surrounds it:

```
Channels (Infinity, ATM, mobile, corporate portals)
        │
        ▼
API layer / integration (Open API, OFS, Kafka events)
        │
        ▼
┌───────────────┐     ┌──────────────┐     ┌───────────────┐
│  Transact     │◄───►│ Temenos       │     │ Third-party   │
│  (system of   │     │ Payments /    │     │ systems:      │
│   record)     │     │ Data Hub /    │     │ risk, fraud,  │
│               │     │ FCM           │     │ DW, channels  │
└───────────────┘     └──────────────┘     └───────────────┘
```

The recurring architecture questions in this pattern:
- **What stays in the core vs. moves out?** (payments → hub; reporting → Data Hub;
  AML → FCM or specialist; origination → Infinity/Avoka journeys).
- **How much customization is acceptable?** — the upgrade-tax tradeoff of §14.2.
- **Single-core vs multi-core** — groups with subsidiaries on different systems
  consolidate onto Transact for the subsidiaries while the parent may keep its own
  core (the multi-core estate discussion in `core_banking_systems_guide.md` §2.6).

### 17.3 Where Temenos wins and loses deals

Wins: platform deals, multi-country banks, Islamic banking, digital-bank builds with
a real product agenda, banks exiting mainframes that want proven depth rather than
startup risk, wealth + core combos in Asia/MEA. Loses: pure-agility digital banks
(Vault/Mambu), US regional banks (FIS/Fiserv), India price-sensitive deals
(Finacle/BaNCS), teams that demand true microservices decomposition (OBMA/Vault).

---

## 18. Implementation Reality: Timelines, Cost, Upgrades

### 18.1 Timelines

- **Digital-bank build (greenfield, Model Bank-based)**: 6–18 months to live; the
  ANEXT/GLDB case did 11 months (`green_link_digital_bank_guide.md`).
- **Full core replacement at an existing bank**: typically **12–24 months** for a
  mid-size bank, **24–36+ months** for a tier-1 multi-country programme. The
  variance is driven by customization scope, data migration (legacy cleanup), and
  integration surface — not by the product itself.
- **Phased/parallel runs**: most tier-1 conversions run legacy and Transact in
  parallel with a big-bang cutover per country/entity; Temenos's multi-company
  model supports entity-by-entity rollout.

### 18.2 Total cost of ownership

Components (order of magnitude for a mid-size bank):

| Component | Typical share of 5-year TCO | Notes |
|---|---|---|
| Licence/subscription | 20–35% | SaaS converts to recurring; on-prem licence + ~20%/yr maintenance |
| Implementation services | 30–45% | Often 2–5x the licence; SI-led |
| Integration | 10–20% | Channels, payments, risk, data warehouse |
| Data migration | 5–15% | Legacy cleanup dominates |
| Ongoing operations/upgrades | 10–20% | Regression testing is the hidden cost; SaaS shifts this to the vendor |

### 18.3 The upgrade cycle

- On-premise: **one major release per year** (R##) + service packs; each upgrade
  is a programme (test, regression, re-migrate customizations). Budget 3–6 months
  of effort per major upgrade for a customized shop.
- SaaS (Banking Cloud): continuous releases via Temenos Continuous Deployment;
  banks test against sandboxes before promotion. The regression burden moves from
  the bank to the vendor — which is the entire point of the SaaS pitch.

### 18.4 Migration realities

- Data migration from legacy cores is the biggest schedule risk in every Temenos
  programme: account/customer history, product re-parameterization, and the
  "what does the balance actually mean" reconciliation problem.
- The Model Bank reduces *configuration* risk but not *data* risk; banks
  consistently underestimate data cleanup. Budget accordingly — this is the 
  lesson every SI will repeat.

---

## 19. Integration Patterns

### 19.1 The integration surface (what you connect to)

| Surface | Protocol/format | Use case |
|---|---|---|
| **Temenos Open API** | REST/JSON | Channels, mobile, open banking, third-party apps (the default for anything new) |
| **OFS (Open Financial Service)** | File/XML/message | Legacy batch interfaces, ATM/switch feeds, high-volume input — still everywhere in the installed base |
| **Web Services / IRIS** | SOAP/XML | Pre-REST integrations; IRIS wraps core services for enterprise service bus (ESB) consumers |
| **Events** | Kafka (JSON/AVRO) | Real-time downstream: fraud, data hub, channels, reconciliation, streaming analytics |
| **Direct DB access** | JDBC/ODBC to jBASE/Oracle/Postgres | Reporting/read-only consumers (discouraged for writes) |
| **Files/batch** | CSV, ISO 8583, SWIFT MT, ISO 20022 | Regulatory reporting, payments, statement delivery |

### 19.2 Payments integration patterns

- **ISO 20022 end-to-end**: Temenos Payments handles pain/pacs/camt natively —
  see `iso_20022_core_processes_guide.md` for the message lifecycle (initiation →
  clearing → settlement → reporting) and `payments_hub_guide.md` for hub
  architecture. Transact itself is ISO 20022-capable for its payment flows, but
  hub-style deployments push message handling to Temenos Payments.
- **SWIFT**: connectivity via SWIFT Alliance / SWIFT cloud integrations; GPI
  tracking; CBPR+ compliance for cross-border (see `iso_20022_core_processes_guide.md`
  §13).
- **Real-time schemes**: SEPA Instant (SCT Inst) and Singapore FAST are the
  canonical integrations — instant-scheme transactions enter via the hub, get
  real-time fraud/sanctions screening (FCM on the path), post to Transact, and
  return acks in seconds. The same pattern applies to US RTP/FedNow and India UPI
  via regional partners.
- **The "core + hub" split**: the ledger owns postings; the hub owns the message
  lifecycle, routing, and scheme connectivity. Temenos sells both sides, but the
  pattern is vendor-agnostic — see `payments_hub_guide.md` §4 for the state machine.

### 19.3 Open banking / API patterns

- Temenos Open API is the PSD2/open-banking surface: account information (AISP)
  and payment initiation (PISP) APIs, consent management, and the ISO 20022-flavored
  payloads regulators expect.
- The same APIs power Infinity journeys and Exchange apps — one API layer serving
  first-party channels and third-party access alike (the "API-first bank" pattern).
- Practical advice for architects: **never let channels call Transact directly**.
  Put everything behind the Open API / integration layer, reserve OFS for legacy
  batch, and use events for anything real-time downstream. This is how Temenos
  banks keep the core swappable in principle and testable in practice.

---

## 20. Temenos Skills and Careers

### 20.1 The T24/Transact skill set

The Temenos talent market is a world of its own, and it maps directly to the
architecture in §12:

| Skill | What it is | Who needs it |
|---|---|---|
| **T24 BASIC development** | Writing routines in the T24 BASIC dialect | Customization teams at banks and SIs |
| **Designer / Workbench** | Configuring applications, versions, enquiries | Functional consultants, developers |
| **Temenos functional consulting** | Products, rates, Model Bank configuration, fit-gap | The largest and most portable skill group — "Temenos consultants" are a recognized market |
| **T24/Transact administration** | jBASE/Oracle/Postgres administration, COB tuning, environment management, upgrades | Bank ops teams |
| **Transact architecture** | Deployment topology, integration (OFS/IRIS/API/events), performance, upgrade strategy | Solution architects — the rarest and best-paid layer |
| **Banking Cloud / DevOps** | Kubernetes, CI/CD (Temenos Continuous Deployment), cloud ops | Growing fast with the SaaS pivot |

### 20.2 The consultant market

- Demand is structurally high: the 1,000+ installed base needs upgrade and
  migration skills continuously, and every new digital-bank deal creates a
  multi-year consulting wave. T24/Transact skills are the classic "accidental
  career" — a niche with high rates and low supply.
- Supply is aging: the deepest expertise sits with veterans who learned T24 in the
  1990s–2000s; new graduates gravitate to cloud-native stacks, which is a real
  long-term risk for both Temenos and its customers.
- SIs (Accenture, Deloitte, Capgemini, IBM, and the regional specialists) run
  large Temenos practices; for a solution architect in banking, Temenos skills
  are a durable differentiator — especially combined with the modern stack
  (Kubernetes, Kafka, APIs) that the Banking Cloud era requires.

### 20.3 Architect-relevant takeaways

- Know the **five building blocks** (§12.2) cold — they are the vocabulary of every
  Temenos conversation.
- Know the **upgrade economics** (§18) — they drive every commercial decision a
  Temenos bank makes.
- Know the **era position** (`core_banking_systems_guide.md` §3): Temenos is the
  archetypal "modernized traditional core," and your job as architect is usually to
  integrate around it cleanly rather than to change it.

---

## 21. Temenos in Singapore and Asia

### 21.1 The Asia footprint

- Temenos has had an **Asia-Pacific regional presence in Singapore** for decades
  (regional HQ functions, sales, services, and partners); Singapore is also a hub
  for its Southeast Asia banking business and for the digital-bank wave.
- Representative Temenos anchors in Asia (from Temenos announcements over the
  years): **ANEXT Bank** (Singapore digital bank — Transact on Huawei Cloud, 11
  months; see `green_link_digital_bank_guide.md`), **UnionBank of the Philippines**,
  plus Islamic-banking franchises across Malaysia/Indonesia/Brunei, and retail
  banks across Vietnam, Sri Lanka, and the subcontinent. (Temenos does not publish
  a full client list; treat specifics as illustrative.)
- The competitive reality in Asia: **FLEXCUBE's home turf** (Oracle has a huge
  Asian installed base — see `oracle_banking_microservices_architecture_guide.md`),
  with Finacle and TCS BaNCS strong in India and price-competitive EMEA/Asia deals.
  Temenos wins the premium/multi-country and Islamic segments; the challengers
  (Vault/Mambu) are winning the new digital-bank licences.

### 21.2 The Singapore digital-bank context

- MAS granted four digital-bank licences in Dec 2020 (two digital full banks, two
  digital wholesale banks). ANEXT (Ant Group) and GLDB (Green Link — the Grab-Singtel
  consortium, now branded **GXS Bank** and **Trust Bank** respectively in their
  different forms) are covered in `green_link_digital_bank_guide.md`, including the
  verification of what "Green Link Digital Bank" actually is.
- **ANEXT's Temenos build is the case that matters here**: a digital wholesale bank
  on Transact + Banking Cloud on **Huawei Cloud**, live in **11 months** — the
  proof point for the Model Bank + SaaS playbook in Asia. The guide details the
  architecture choices and the lessons for banking architects.
- The GLDB/GXS and Trust builds took different core paths (this repo's
  `green_link_digital_bank_guide.md` and the vendor table in
  `core_banking_systems_guide.md` §5 give the comparison) — a useful real-world
  illustration of the "challenger core vs traditional core" decision.

### 21.3 MAS regulatory context

- MAS's technology-risk (TRM), AML/CFT (MAS Notice 626 etc.), and data-residency
  expectations shape every core and payments decision in Singapore — see
  `financial_risk_compliance_systems_guide.md` for the compliance-systems
  landscape and `payments_hub_guide.md` §14 for the Singapore payments context
  (FAST, PayNow, and the ISO 20022 migration timeline).
- Temenos addresses these with: FCM (AML/sanctions screening), Data Hub
  (regulatory reporting), ISO 20022-native payments (FAST/PayNow connectivity),
  and Banking Cloud's data-residency posture (including regional clouds like
  Huawei Cloud when that is the residency answer).

### 21.4 Temenos's Asia strategy

- The commercial priority is the **SaaS conversion and digital-bank land-grab**:
  Asia is where greenfield digital banks are densest, and where Temenos can win
  net-new platform deals without converting a legacy base.
- The Huawei Cloud relationship matters specifically for China and China-linked
  builds (ANEXT is Ant Group-owned; Huawei Cloud was the residency/sovereignty
  choice). For non-China Asian banks, AWS/Azure are the default Banking Cloud
  substrates.
- Watch for: Vault/Mambu taking the *next* wave of digital-bank licences (they
  already have several globally), and Temenos countering with depth + the
  Exchange ecosystem in the mid-market.

---

## 22. The Future: 2026 and Beyond

### 22.1 Strategy under the current (non-PE) ownership

- **Cloud-first as default**: Banking Cloud is the default offer for every new
  deal; the ARR/SaaS pivot (ARR +11–12% through FY24–Q2 2026) is the board's
  headline metric. Expect continued aggressive subscription conversion and
  "always on latest release" positioning.
- **Leadership**: with interim CEO Takis Spiliopoulos (CFO) still in place as of
  mid-2026 and a permanent-successor search that has dragged, the strategy risk
  is *continuity*, not direction — the cloud+AI direction is set and visible in
  the financials. The Hindenburg aftermath (independent examination, governance
  changes) has receded but remains the backdrop to any equity story.
- **The PE question**: Vista's failed 2022–23 bid and 2025 reports of renewed
  interest mean a take-private at a depressed valuation is a live (if
  unconfirmed) scenario — the accurate framing is "repeated PE interest, no PE
  ownership," not "PE-owned."

### 22.2 AI in Temenos

- **Temenos AI** is the umbrella for AI-assisted banking: AI copilots for
  operations (account servicing, payments exceptions, KYC/AML case triage),
  AI-assisted product configuration, and AI in the customer journey (Infinity
  personalization).
- **Transact R25 is the "AI-driven core"** release: AI embedded in core operations
  (anomaly detection on postings, natural-language enquiry against banking data,
  AI-assisted COB/exception handling). The positioning is the **"autonomous
  bank"**: AI handling routine operations while humans handle exceptions.
- **Sibos 2025: Temenos Money Movement & Management** — the AI-powered payments/
  accounts platform — is the product manifestation of the AI push.
- Architect's read: Temenos AI is mostly *assistive* (copilots, screening, NL
  interfaces) rather than *autonomous core logic*; the safe integration pattern is
  AI on the event bus and APIs (the modern surfaces of §19), not inside T24 BASIC.
  For the broader AI-in-banking context see the LLM/AI guides in this repo's
  `technology/ai_llm/` directory.

### 22.3 Composable direction and the platform vision

- Temenos's strategic answer to composability is **Exchange + Banking Services +
  Open API**: keep the core whole, make everything around it composable, and
  monetize the ecosystem. See the composable debate in
  `core_banking_systems_guide.md` §6 for where this lands in the market spectrum.
- The **"one platform, all products"** vision (Transact + Infinity + Payments +
  Data Hub + FCM as services on one cloud platform) is the long-term bet: Temenos
  wants to be the *platform* a bank builds on, not just the core it buys.
- **The honest caveat**: the vision is only as strong as the weakest integration.
  Banks repeatedly report that the satellite products integrate *well enough*,
  not *seamlessly* — the platform story is real but partial, and the depth of the
  core remains the actual moat.

### 22.4 Competitive pressure and the 2026+ outlook

- The cloud-native challengers (Vault, Mambu) will keep winning greenfield and
  tier-2 deals; Oracle's OBMA keeps the incumbent-vs-incumbent pressure up; the
  open-source flank (Fineract) keeps the low end cheap.
- Temenos's defensible position: **depth + footprint + ecosystem + the
  installed-base conversion machine** (1,000+ banks that must modernize, most of
  them into Banking Cloud). The risk is that "modernize in place" is a decade-long
  story in a market that is moving to "born cloud-native."
- Most likely 2026–2030 path: Temenos remains the global #1 by share and the
  default "safe choice" for serious core replacement, while ceding the pure
  greenfield-agility segment to challengers — and the AI + SaaS story determines
  whether it grows or defends. For the architect, the practical conclusion is the
  same as in §16.3: treat Temenos as a deep, proven platform with real
  modernization momentum, plan around its upgrade and customization economics,
  and integrate through its open surfaces.

---

## 23. Glossary

| Term | Meaning |
|---|---|
| **T24** | The original Temenos core banking system ("Temenos 24", 24-hour banking); the product name until the 2018 rebrand; still used loosely for the architecture and the skills market |
| **Transact** | The current name of the flagship core banking system (T24's evolution); releases are `R<year>` (R22, R23, R24, R25…) |
| **Application (T24)** | A file definition in the T24 data model — the "table" (e.g., `CUSTOMER`, `ACCOUNT`) with fields, dictionary, and processing rules |
| **Version (T24)** | A screen layout over an application (e.g., `CUSTOMER,INPUT.DEFAULT`) defining fields, validation, and input processing |
| **Enquiry (T24)** | A query/report definition over applications — the T24 query language for selecting, sorting, and outputting data |
| **Routine / T24 BASIC** | A subroutine in the BASIC dialect with MultiValue string handling; the unit of custom business logic |
| **jBASE** | The MultiValue (Pick-lineage) database that was T24's native data store; acquired by Temenos in 2007; still used, with Oracle/DB2/PostgreSQL as alternatives |
| **MultiValue** | The data model (Pick tradition) where fields hold repeated values; the foundation of T24's flexible, schema-light design |
| **Designer** | The classic T24 development environment (applications, versions, enquiries, routines) |
| **Workbench** | The modern, DevOps-oriented configuration/development tooling for Transact (R2x) |
| **Model Bank** | Temenos's pre-configured reference bank shipped with each release — the starting point for implementations (industry + country models) |
| **COB** | Close of Business — the batch/scheduler running the overnight processing cycle |
| **OFS** | Open Financial Service — the classic file/XML integration channel for T24 |
| **IRIS** | Temenos's integration framework wrapping core services for external consumption (pre-REST era; superseded at the API edge by Open API) |
| **Temenos Open API** | The REST API catalogue over the platform (accounts, payments, customers, limits…) — the open-banking and integration surface |
| **Infinity** | The omnichannel digital-banking platform (portal/mobile), launched 2016 |
| **Banking Cloud** | Temenos's SaaS/PaaS delivery platform (containers, Kubernetes, multi-cloud: AWS/Azure/GCP/Huawei), launched 2020 |
| **Temenos Continuous Deployment** | The automated build/regression-test/deploy tooling that powers SaaS upgrades |
| **Temenos Payments** | The ISO 20022-native payments hub product (real-time, SWIFT, orchestration) |
| **Data Hub** | The data/BI layer — golden-source data platform for reporting, regulatory, analytics (launched 2021) |
| **FCM** | Temenos Financial Crime Mitigation — AML monitoring, sanctions screening, KYC (launched 2019) |
| **Exchange** | The partner marketplace/"app store" for pre-integrated third-party banking apps (launched 2021) |
| **Multifonds** | Fund-administration platform acquired 2017 (NAV, transfer agency) |
| **Model Bank / country model** | Pre-configured localizations (regulatory, tax, reporting) shipped for 100+ jurisdictions |
| **R22/R23/R24/R25** | Annual Transact releases (2022/2023/2024/2025); R25 is the "AI-driven core" release |
| **Vista Equity Partners** | US PE firm (founded 2000, Robert F. Smith) that bid for Temenos in Nov 2022 at CHF 130/share — **the bid lapsed in 2023; Temenos was never taken private**; Vista earlier acquired Misys (2012), which became Finastra |
| **Hindenburg Research** | US short-seller whose Feb 2024 report on Temenos alleged accounting irregularities; Temenos denied the claims and commissioned an independent examination |
| **ANEXT / GLDB** | Ant Group's Singapore digital wholesale bank, built on Temenos on Huawei Cloud in 11 months — see `green_link_digital_bank_guide.md` |
| **T24 Browser** | The classic T24/Transact operations UI (staff-facing screens) |

---

## 24. References and Related Guides

### Sibling guides in this repository

- `core_banking_systems_guide.md` — vendor landscape (§5), core-banking eras (§3), the composable debate (§6), the multi-core estate (§2.6)
- `oracle_banking_microservices_architecture_guide.md` — the FLEXCUBE→OBMA story and Temenos-vs-Oracle comparison (§15)
- `apache_fineract_guide.md` — the open-source competitor
- `green_link_digital_bank_guide.md` — the ANEXT/GLDB Singapore digital-bank build on Temenos (Huawei Cloud, 11 months)
- `payments_hub_guide.md` — payments hub architecture, vendors, and the Singapore context (§12, §14)
- `iso_20022_core_processes_guide.md` — the ISO 20022 message lifecycle that Temenos Payments implements
- `financial_risk_compliance_systems_guide.md` — AML/financial-crime and regulatory systems (FCM context)
- `financial_infrastructure_guide.md`, `bian_banking_architecture_guide.md`, `data_models_banking_insurance_guide.md` — adjacent architecture references
- `nasdaq_calypso_guide.md` — the trading-domain counterpart (Temenos treasury vs Calypso)

### External sources consulted (verified facts)

- Temenos FY2024 results (Feb 2025) — revenue US$1.04B, ARR US$804M (+12% cc)
- Temenos FY2025 results (Feb 2026) — EPS US$4.06; Q2 2026 results (Jul 2026) — ARR +11%
- Euromoney Awards for Excellence 2025 — "world's best core banking solution: Temenos"; 1,000+ FIs, 150+ countries
- Temenos Banking Cloud factsheet (2022) — 700+ banks on SaaS
- Temenos press (Nov 2022) — 850+ Infinity clients
- Hindenburg Research, "Temenos: Major Accounting Irregularities…" (Feb 15, 2024) and follow-up (Apr 2024); CNBC/Bloomberg/Yahoo coverage of the share-price reaction
- Temenos press (Sep 4, 2025) — CEO transition (Brulard out, Spiliopoulos interim); fintech media coverage
- Temenos press (Feb 2024–May 2024) — Hindenburg response and Brulard appointment
- Sibos 2025 coverage — Temenos Money Movement & Management launch
- IDC MarketScape North America Digital Core Banking Platforms 2024 — Temenos named a Leader

*Guide compiled for the research repository of a Singapore-based solution architect;
cross-referenced against the sibling banking guides above. Figures as of mid-2026.*


