# Core Banking Systems Used by US Banks: A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Core Banking / Banking Architecture — the US banking sector, the tier structure (mega-banks, super-regionals, regionals, community banks, credit unions, neobanks), the mainframe heritage of the big four, the FIS/Fiserv/Jack Henry vendor oligopoly, the cloud-native challengers (Thought Machine, Mambu, nCino), the core-modernization wave, the dual-banking regulatory system, US vs. China/Europe, Singapore relevance
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** August 2026

---

## Table of Contents

1. [The US Banking Sector: Tiers, Scale, and Regulation](#1-the-us-banking-sector-tiers-scale-and-regulation)
   - 1.1 [The Bank Tiers](#11-the-bank-tiers)
   - 1.2 [Scale: ~4,400 Institutions, ~$25 Trillion of Assets](#12-scale-4400-institutions-25-trillion-of-assets)
   - 1.3 [Payment Volumes: ACH, Fedwire, CHIPS, and the Instant Rails](#13-payment-volumes-ach-fedwire-chips-and-the-instant-rails)
   - 1.4 [The Regulatory Environment: Dual Banking, OCC, Fed, FDIC, CFPB](#14-the-regulatory-environment-dual-banking-occ-fed-fdic-cfpb)
2. [The Historical Core Landscape](#2-the-historical-core-landscape)
   - 2.1 [Era 1: The Mainframe Era (1960s–2000s)](#21-era-1-the-mainframe-era-1960s2000s)
   - 2.2 [Era 2: The Client-Server Era (1990s–2000s)](#22-era-2-the-client-server-era-1990s2000s)
   - 2.3 [Era 3: The Off-the-Shelf Vendor Era (2000s–2010s)](#23-era-3-the-off-the-shelf-vendor-era-2000s2010s)
   - 2.4 [Era 4: The Cloud Era (2010s–Present)](#24-era-4-the-cloud-era-2010spresent)
   - 2.5 [The In-House Mainframe Reality of the Mega-Banks](#25-the-in-house-mainframe-reality-of-the-mega-banks)
   - 2.6 [The US vs. the Rest of the World: Why International Cores Never Penetrated](#26-the-us-vs-the-rest-of-the-world-why-international-cores-never-penetrated)
3. [The Vendor Landscape](#3-the-vendor-landscape)
   - 3.1 [The Vendor Map at a Glance](#31-the-vendor-map-at-a-glance)
   - 3.2 [FIS (Fidelity National Information Services)](#32-fis-fidelity-national-information-services)
   - 3.3 [Fiserv](#33-fiserv)
   - 3.4 [Jack Henry & Associates](#34-jack-henry--associates)
   - 3.5 [The International Vendors in the US: Temenos and Oracle FLEXCUBE](#35-the-international-vendors-in-the-us-temenos-and-oracle-flexcube)
   - 3.6 [The Cloud-Native Challengers: nCino, Thought Machine, Mambu](#36-the-cloud-native-challengers-ncino-thought-machine-mambu)
   - 3.7 [The Digital-Layer Vendors: Q2, Alkami, NCR](#37-the-digital-layer-vendors-q2-alkami-ncr)
   - 3.8 [The Core-as-a-Service and BaaS Layer: Synapse, Unit, Column](#38-the-core-as-a-service-and-baas-layer-synapse-unit-column)
   - 3.9 [US Vendor Characteristics](#39-us-vendor-characteristics)
4. [The Mega-Banks' In-House Cores](#4-the-mega-banks-in-house-cores)
   - 4.1 [The Self-Operated Core Pattern](#41-the-self-operated-core-pattern)
   - 4.2 [JPMorgan Chase](#42-jpmorgan-chase)
   - 4.3 [Bank of America](#43-bank-of-america)
   - 4.4 [Citi](#44-citi)
   - 4.5 [Wells Fargo](#45-wells-fargo)
   - 4.6 [Capital One: The Cloud Pioneer](#46-capital-one-the-cloud-pioneer)
   - 4.7 [The Mega-Bank Reality: Moat, Legacy Debt, Run vs. Change](#47-the-mega-bank-reality-moat-legacy-debt-run-vs-change)
5. [Community Banks and Credit Unions](#5-community-banks-and-credit-unions)
   - 5.1 [The Community Bank Core Market](#51-the-community-bank-core-market)
   - 5.2 [In-House vs. Outsourced Processing](#52-in-house-vs-outsourced-processing)
   - 5.3 [The Credit Union Core Market](#53-the-credit-union-core-market)
6. [The Modernization Wave](#6-the-modernization-wave)
   - 6.1 [The Drivers](#61-the-drivers)
   - 6.2 [The Four Approaches: Replace, Augment, Wrap, Migrate](#62-the-four-approaches-replace-augment-wrap-migrate)
   - 6.3 [The Core-Replacement Case Studies](#63-the-core-replacement-case-studies)
   - 6.4 [Core-as-a-Service and BaaS](#64-core-as-a-service-and-baas)
   - 6.5 [The Challenges](#65-the-challenges)
7. [The Technology Stack Details](#7-the-technology-stack-details)
   - 7.1 [The Mainframe Stack: IBM Z, COBOL, CICS, IMS/DB2](#71-the-mainframe-stack-ibm-z-cobol-cics-imsdb2)
   - 7.2 [Open Systems and Databases: Oracle, SQL Server, Db2, PostgreSQL](#72-open-systems-and-databases-oracle-sql-server-db2-postgresql)
   - 7.3 [The Cloud: AWS, Azure, GCP](#73-the-cloud-aws-azure-gcp)
   - 7.4 [APIs and Open Banking: The CFPB 1033 Rule](#74-apis-and-open-banking-the-cfpb-1033-rule)
   - 7.5 [The Digital Layer: Q2, Alkami, NCR, and the Vendor Digital Stacks](#75-the-digital-layer-q2-alkami-ncr-and-the-vendor-digital-stacks)
   - 7.6 [Payments: FedNow, RTP, ACH, and the Card Networks](#76-payments-fednow-rtp-ach-and-the-card-networks)
8. [The US vs. Other Markets](#8-the-us-vs-other-markets)
   - 8.1 [US vs. China: The Comparison Table](#81-us-vs-china-the-comparison-table)
   - 8.2 [US vs. Europe](#82-us-vs-europe)
   - 8.3 [US vs. Emerging Markets: The Leapfrog](#83-us-vs-emerging-markets-the-leapfrog)
   - 8.4 [US-Specific Dynamics and Global Implications](#84-us-specific-dynamics-and-global-implications)
9. [The US Core in Practice: An Architect's View](#9-the-us-core-in-practice-an-architects-view)
   - 9.1 [The US Bank Architecture: Core and Surroundings](#91-the-us-bank-architecture-core-and-surroundings)
   - 9.2 [Integration Patterns: APIs, File Feeds, ETL](#92-integration-patterns-apis-file-feeds-etl)
   - 9.3 [Core Data: Accounts, Transactions, and the Data Model](#93-core-data-accounts-transactions-and-the-data-model)
   - 9.4 [Regulatory Reporting: The Call Report and FFIEC](#94-regulatory-reporting-the-call-report-and-ffiec)
   - 9.5 [Risk: Concentration Risk and Third-Party Risk](#95-risk-concentration-risk-and-third-party-risk)
   - 9.6 [Assessment and Vendor Selection](#96-assessment-and-vendor-selection)
10. [The Future: 2026 and Beyond](#10-the-future-2026-and-beyond)
    - 10.1 [Cloud Core Adoption](#101-cloud-core-adoption)
    - 10.2 [The Core Replacement Wave](#102-the-core-replacement-wave)
    - 10.3 [BaaS and Embedded Finance](#103-baas-and-embedded-finance)
    - 10.4 [AI in the Core](#104-ai-in-the-core)
    - 10.5 [Open Banking and the 1033 Compliance Clock](#105-open-banking-and-the-1033-compliance-clock)
    - 10.6 [Consolidation: Vendors and Banks](#106-consolidation-vendors-and-banks)
    - 10.7 [Trends Summary](#107-trends-summary)
11. [Glossary](#11-glossary)
12. [Claims-Status Table](#12-claims-status-table)
13. [References](#13-references)

---

### How to Read This Guide

This is the dedicated deep-dive on **core banking systems used by US banks** — the companion to the umbrella guide [core_banking_systems_guide.md](core_banking_systems_guide.md) (which covers the global core-banking discipline: what a core is, the vendor landscape, modernization approaches, and the "core replacement is the hardest IT project in banking" thesis). It assumes the reader knows the basics from that guide and focuses on what is *different* about the United States: the fragmented tier structure, the mainframe heritage of the biggest banks, the FIS/Fiserv/Jack Henry oligopoly over the long tail, the dual-banking regulatory system, and an incremental — rather than revolutionary — modernization wave.

Related sibling guides in this series that this guide cross-references:

- [chinese_bank_core_systems_guide.md](chinese_bank_core_systems_guide.md) — the mirror image: China's self-developed, distributed, xinchuang-driven core landscape. Section 8 of this guide compares the two directly.
- [financial_infrastructure_guide.md](financial_infrastructure_guide.md) — the US payment rails: Fedwire, ACH, CHIPS, FedNow, RTP, and the card networks.
- [programmable_business_bank_guide.md](programmable_business_bank_guide.md) — the US neobanks and fintechs (Mercury, Brex, Ramp) and the bank-partner/BaaS model.
- [core_banking_systems_guide.md](core_banking_systems_guide.md) — the global umbrella: vendor landscape, BaaS, modernization.
- [temenos_guide.md](temenos_guide.md) and [oracle_banking_microservices_architecture_guide.md](oracle_banking_microservices_architecture_guide.md) — the international core platforms whose US penetration is limited but growing (Regions Bank on Temenos SaaS).
- [data_models_banking_insurance_guide.md](data_models_banking_insurance_guide.md) — the account/transaction data models that US cores implement.
- [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md) — third-party and concentration risk as they apply to core vendors.

Suggested reading paths:

- **New to the domain** — [core_banking_systems_guide.md](core_banking_systems_guide.md) Sections 1–2 first, then this guide's [Section 1](#1-the-us-banking-sector-tiers-scale-and-regulation).
- **Sizing a US modernization program** — [Section 6](#6-the-modernization-wave) and [Section 7](#7-the-technology-stack-details).
- **Vendor/product research** — [Section 3](#3-the-vendor-landscape), [Section 5](#5-community-banks-and-credit-unions), and the [claims-status table](#12-claims-status-table).
- **US vs. China / global comparison** — [Section 8](#8-the-us-vs-other-markets).
- **Singapore/global relevance** — [Section 8.4](#84-us-specific-dynamics-and-global-implications) and [Section 10](#10-the-future-2026-and-beyond).

A note on verification: this guide was researched with web searches in August 2026. US-specific facts that were **verified** from retrievable sources are marked as such; items that are widely reported in industry literature but could **not** be independently verified in this session (e.g., some internal system names, asset figures as of a specific quarter) are explicitly flagged, and everything US-specific is summarised in the [claims-status table](#12-claims-status-table) at the end.

---

## 1. The US Banking Sector: Tiers, Scale, and Regulation

### 1.1 The Bank Tiers

The US banking system is the world's largest by most measures but is structured very differently from China's or Europe's. There is no state-owned mega-tier: the US has roughly **4,400 FDIC-insured banks** plus **~4,300 federally insured credit unions**, ranging from JPMorgan Chase ($4.4 trillion in assets) down to single-branch community banks with a few hundred million in assets. The single most important fact for core-banking technology is that **core-system strategy follows tier** — and the tiers span an enormous range of scale, regulatory burden, and IT budget.

| Tier | Examples | Approx. count | Core-system profile |
|---|---|---|---|
| **Mega-banks / money-center banks ("Big Four")** | JPMorgan Chase, Bank of America, Citi, Wells Fargo | 4 (plus Goldman Sachs, Morgan Stanley as giant wholesale/investment banks) | In-house mainframe cores; COBOL/IMS/DB2 heritage; decades of M&A-induced core proliferation; tens of billions in annual tech spend |
| **Super-regionals** | US Bancorp, PNC, Truist, TD Bank US, Capital One (+ Goldman Sachs, Morgan Stanley in the wholesale/investment sense) | ~10–20 | Mix: in-house client-server/mainframe cores, vendor cores (FIS/Fiserv legacy), and — for Capital One — a cloud-native path; the "deposit/retail" vs. "investment bank" distinction matters |
| **Regional banks** | Regions, Fifth Third, KeyBank, Huntington, Citizens, M&T, First Horizon, and hundreds more | ~100+ ($10B–$100B+) | Vendor cores (FIS Profile, Fiserv DNA/Premier, Jack Henry SilverLake) or in-house legacy; the current battleground for core replacement (Temenos, Thought Machine, Mambu, nCino) |
| **Community banks** | Thousands of locally chartered banks, e.g., the ~4,000 banks under $10B | ~4,000 (the majority of banks by count) | Jack Henry SilverLake/CIF 20/20, Fiserv Premier/DNA, FIS Profile, CSI NuPoint; increasingly outsourced to the vendor (the "outsourced core") |
| **Credit unions** | Navy Federal, PenFed, State Employees' CU, and ~4,300 others | ~4,300 (federally insured) | Jack Henry Symitar (Episys) is the dominant core; Fiserv CUnify/Portico, and co-op processors like CU*Answers |
| **Neobanks / fintech-with-a-bank-partner** | Chime, Varo, SoFi, Dave, Current | dozens of material players | No core of their own: run on sponsor-bank cores via BaaS (The Bancorp, Stride, Cross River, Choice Financial, etc.); Varo and SoFi later obtained their own charters — see [programmable_business_bank_guide.md](programmable_business_bank_guide.md) |

Notes and clarifications:

- **The "Big Four" and the money-center tradition.** JPMorgan Chase, Bank of America, Citi, and Wells Fargo dominate US retail deposits (roughly 40%+ of industry deposits between them) and each operates a huge in-house core estate. "Money-center" historically meant banks headquartered in money centers (New York, later Charlotte/San Francisco) that ran wholesale, correspondent, and international businesses alongside retail. All four are **global systemically important banks (G-SIBs)** and are subject to the most intense supervisory scrutiny in the world.
- **Super-regionals: deposit/retail vs. investment bank.** The label "super-regional" is used loosely. In the *deposit/retail* sense it means US Bancorp, PNC, Truist, TD Bank US, and Capital One — giant retail franchises with $500B–$700B of assets that behave like big regional banks. In the *wholesale* sense, Goldman Sachs and Morgan Stanley are mega-banks by assets and risk but have no retail deposit networks of scale (Goldman built Marcus then largely unwound it; Morgan Stanley's retail is via brokerage wealth management). Their cores are therefore retail-lite: wealth/transaction-banking platforms, not mass-market deposit engines. The two groups have very different core-technology problems.
- **Regional banks** ($10B–$100B+) number in the low hundreds and are the sweet spot for the modern-core vendors. They have real retail franchises but lack the mega-banks' in-house engineering armies; they are old enough to carry two or three generations of legacy cores (often from M&A) and wealthy enough to contemplate replacement.
- **Community banks** (under $10B; the FDIC's community-bank definition was revised in 2020 to capture institutions up to $10B in assets) are the **majority of US banks by count** but hold only a minority of assets. Their IT is typically outsourced entirely: the core vendor runs the system of record, the digital vendor runs online/mobile, and the bank's own IT staff may be a handful of people.
- **Credit unions** are a parallel, mutually owned system: ~4,300 federally insured credit unions regulated by the **NCUA** (not the OCC/FDIC), with their own core-vendor sub-market (Symitar dominance) and their own field of membership rules.
- **Neobanks** are not a regulatory tier at all: they are fintechs with a **bank partner** (sponsor bank) that provides the charter, the deposit accounts, and the core — the "programmable bank" model detailed in [programmable_business_bank_guide.md](programmable_business_bank_guide.md). Two (Varo, SoFi) later bought their own bank charters and became, technically, small banks running (or acquiring) cores.

### 1.2 Scale: ~4,400 Institutions, ~$25 Trillion of Assets

The headline numbers, as of the latest verified data (2025):

- **~4,400 FDIC-insured institutions.** The FDIC's Quarterly Banking Profile reported 4,421 insured commercial banks and savings institutions in Q1 2025 and **4,379 in Q3 2025** (a decline of 42 in the quarter, driven by mergers). The long-run trend is relentless consolidation: from ~18,000 institutions in the mid-1980s to ~8,000 in 2010 to ~4,400 today.
- **~4,300 federally insured credit unions.** The NCUA reported **4,287 federally insured credit unions at Q4 2025**, down from 4,455 a year earlier — the same consolidation curve as banks.
- **~$25 trillion of banking industry assets.** FDIC-insured institutions hold roughly $25 trillion in assets (Q3 2025 industry total ~$25.4T, an approximation); credit unions add roughly another ~$2.5 trillion.
- **JPMorgan Chase alone: $4.4 trillion in assets and $362 billion in stockholders' equity** as of December 31, 2025 (per its 2025 annual report) — the largest US bank by assets, and larger than the *entire* banking systems of most countries. It serves on the order of 80 million US consumers and processes on the order of 300 million transactions a day on its mainframe estate.
- **Concentration of the top tier.** The four mega-banks hold roughly half of industry assets and an even larger share of deposits and technology capacity. Their combined annual technology spend (JPMorgan alone ~$18B in 2025) exceeds the entire revenue of the core-vendor industry they buy from.

The implication for core banking: the US core market is really **two markets** — a handful of self-operated mega-bank estates (hundreds of cores, mostly mainframe) and a vendor-served long tail of thousands of small institutions (a few core products, mostly outsourced). The vendor oligopoly (Section 3) serves the long tail; the mega-banks are their own vendors (Section 4).

### 1.3 Payment Volumes: ACH, Fedwire, CHIPS, and the Instant Rails

The US payment system is enormous and runs largely outside the core systems: the core keeps the accounts; the payment rails move the money. The dedicated treatment is in [financial_infrastructure_guide.md](financial_infrastructure_guide.md); the summary relevant to cores:

- **ACH (Nacha network):** the workhorse retail/batch rail. Roughly 33+ billion payments worth ~$85T+ per year (2024–2025 era figures) — payroll, bill pay, direct deposit, B2B. ACH is how the core's DDA (demand deposit account) postings arrive in batch.
- **Fedwire:** the Fed's large-value real-time gross settlement (RTPS) rail, settling on the order of a quadrillion dollars per year (roughly $1 quadrillion annually in recent years); the finality backbone for large-value payments.
- **CHIPS:** The Clearing House's privately operated large-value netting system, the other half of US wholesale dollar clearing (~$500T+/year).
- **FedNow (launched July 2023)** and **The Clearing House RTP (launched 2017):** the two US instant rails. Both run 24x7x365 with real-time posting — which puts genuine real-time pressure on cores for the first time (see Section 7.6).
- **Cards:** Visa/Mastercard (and Amex/Discover) networks clear and settle most US consumer non-cash spending; card authorization touches the core (or a real-time shadow balance) at the point of sale.

For the core architect the key fact is: **US cores are batch-first, real-time-second**. ACH and card settlement arrive as batch files; FedNow/RTP post via API in real time. The coexistence of both worlds is one of the central tensions in US core modernization (Section 7).

### 1.4 The Regulatory Environment: Dual Banking, OCC, Fed, FDIC, CFPB

The US regulatory structure is fragmented by design — the **dual banking system** in which banks can choose a **national charter** or a **state charter**, and are supervised by overlapping federal and state agencies:

- **The OCC** (Office of the Comptroller of the Currency, in the Treasury) charters and supervises **national banks** and federal savings associations. The mega-banks and most large regionals are OCC-supervised.
- **The Federal Reserve** supervises state-chartered banks that are Fed members, all **bank holding companies** (so it is the umbrella supervisor for the big four's holding companies), and acts as lender of last resort and operator of Fedwire and FedNow.
- **The FDIC** insures deposits (up to $250,000 per depositor per institution), supervises state-chartered non-member banks, and is the **receiver** for failed banks — the agency that famously took over Silicon Valley Bank in March 2023.
- **The CFPB** (Consumer Financial Protection Bureau, created by Dodd-Frank 2010) writes and enforces consumer-protection rules over deposits, lending, and now **open banking (Section 1033)**.
- **State regulators** charter and supervise state banks and state-chartered credit unions, money transmitters, and fintechs — the "50 states" fragmentation that makes national fintech licensing hard.
- **NCUA** regulates federal credit unions (the credit-union mirror of the FDIC/OCC).

The landmark laws: **Dodd-Frank (2010)** — the post-2008-crisis overhaul that created the CFPB, the Financial Stability Oversight Council (FSOC), enhanced prudential standards for large banks, and stress testing (CCAR/DFAST); **Basel III** — the international capital framework, implemented in the US with modifications; and the **Basel III "Endgame"** proposal (July 2023) to raise capital requirements for the largest banks, which was finalized in a recalibrated form in late 2025 (approved by the Fed in November 2025, with phased implementation into 2028 — reported, not independently verified here).

For core systems the regulatory environment matters in five ways:

1. **The core is the system of record for regulatory reporting** — the quarterly **Call Report** (FFIEC 031/041) is generated from core data (Section 9.4).
2. **Third-party risk management** — regulators (OCC Bulletin 2013-29 and successors, FDIC FILs) require banks to manage **concentration risk** in core providers; the "Big Three" vendor oligopoly is itself a supervisory concern (the Kansas City Fed has published research on core-services market concentration).
3. **Core replacement is a supervisory event** — a change of core platform triggers due diligence, risk-assessment, and often prior notice to examiners; regulators have seen enough failed conversions to treat them as high-risk projects.
4. **Open banking (1033)** — the CFPB's October 2024 final rule on **Personal Financial Data Rights** will force large data providers to expose customer-permissioned APIs (Section 7.4), which reshapes how cores expose data.
5. **Charter choice affects the vendor ecosystem** — a national charter means OCC supervision; a state charter means the state regulator plus (if Fed member) the Fed. Vendor selection and core contracts are reviewed in that context.

The regulator-to-core touchpoints in practice:

| Regulator | Core-system touchpoint |
|---|---|
| **OCC** | National-bank chartering/exam: core as critical system, third-party risk (Bulletin 2013-29), core-change scrutiny for national banks |
| **Federal Reserve** | Holding-company supervision (FR Y-9C from core data), CCAR stress testing, Fedwire/FedNow participation, state member banks |
| **FDIC** | Deposit insurance assessment (core deposit data), state non-member bank exams, failed-bank resolution (core data portability becomes urgent at failure) |
| **CFPB** | Consumer rules (Reg DD, Reg E, overdraft), and **1033** open-banking API mandates |
| **State regulators** | State-charter exams, money-transmitter licensing for fintechs, state consumer laws |
| **NCUA** | Credit-union supervision (the CU mirror of OCC/FDIC) |

Note the resolution angle: when a US bank fails, the FDIC must run or sell it — which means the failed bank's core (often an outsourced SilverLake/Premier/Profile service) must be operable by the acquirer or the FDIC itself. Core-vendor business-continuity and data-portability provisions are therefore scrutinized by examiners and by acquirers in the failed-bank auction process.

---

## 2. The Historical Core Landscape

### 2.1 Era 1: The Mainframe Era (1960s–2000s)

US banking automation began in the late 1950s–1960s on **IBM mainframes** (the IBM 1401, then the System/360 from 1964). The first "core" applications were deposit-accounting systems written in **COBOL** (born 1959, designed for business computing) running under batch processing: accounts, ledgers, and interest were processed overnight in batch cycles. By the 1970s–1980s the canonical US core stack was in place:

- **Hardware:** IBM System/370, then 390, then Z series.
- **Language:** COBOL (and later some PL/I, Assembler).
- **Transaction monitor:** **CICS** (Customer Information Control System, 1969) for online inquiry/update.
- **Databases:** **IMS** (Information Management System, hierarchical, 1966) and later **DB2** (relational, 1983).

This is the era in which the mega-banks' **in-house cores were born**: JPMorgan (and the dozens of banks that merged into it — Chemical, Chase Manhattan, Bank One, First Chicago, etc.), Bank of America (famously early, automating in the 1950s–60s), Citi, and Wells Fargo (which absorbed Norwest, Wachovia, etc.) each built and accreted deposit, loan, and general-ledger systems. The classic in-house system names (e.g., Chase's deposit systems, BofA's "Mickey Mouse" era mainframes) are mostly internal; what matters is the pattern: **every mega-bank ran a portfolio of home-grown COBOL mainframe cores, one per major product line, one per acquired bank, with batch settlement and file-based integration.**

### 2.2 Era 2: The Client-Server Era (1990s–2000s)

From the 1990s, UNIX/Oracle client-server platforms became the "modern" alternative for new builds — especially in **card systems** (VisionPLUS, TSYS, First Data) and **loan origination**, and in the mid-tier banks. Some regionals bought or built client-server cores; the mega-banks layered client-server systems on top of mainframes rather than replacing them. This era also produced the classic **Hogan** products (IBS, Profile, Umbrella — the Profile lineage that survives today at FIS), which ran on both mainframe and open platforms.

### 2.3 Era 3: The Off-the-Shelf Vendor Era (2000s–2010s)

The 2000s–2010s were the era of **vendor consolidation and off-the-shelf domination** for everyone below the mega-bank tier. Three companies — **FIS, Fiserv, and Jack Henry** — consolidated the US core market through M&A:

- **Fiserv** (Brookfield, WI) grew through dozens of acquisitions (including **Open Solutions** in 2013, which brought the modern **DNA** core).
- **FIS** (Jacksonville, FL) absorbed **Systematics** (via ALLTEL, 2003), **Hogan** (Profile/IBS), **Metavante** (2009), and **Worldpay** (2019, later divested).
- **Jack Henry** (Monett, MO) stayed focused on community banks and credit unions (**SilverLake**, **CIF 20/20**, **Symitar/Episys**), acquiring **Banno** (2018) for digital.

By the late 2010s the market structure was essentially frozen: the "**Big Three**" served the overwhelming majority of US banks and credit unions — a concentration that the **Kansas City Fed** explicitly flagged (March 2024 research brief: the core-services market is "highly concentrated," with FIS, Fiserv, and Jack Henry holding sizable market shares). The long tail also industrialized **outsourcing**: community banks increasingly stopped running cores in-house and let the vendor process for them (Section 5.2).

### 2.4 Era 4: The Cloud Era (2010s–Present)

From the mid-2010s the cloud-native/SaaS challengers arrived, and the mega-banks began the slow migration of adjacent workloads to public cloud:

- **Capital One** became the trailblazer: an early AWS adopter (from 2015), it announced it was going **"all in" on AWS and exiting its data centers entirely (announced 2020)** — the first major US bank to fully abandon its own data centers.
- **JPMorgan** named AWS a preferred cloud provider (2020) and runs its "JPMC cloud" on top, while the mainframe deposit cores remain in place (Section 4.2).
- **SaaS cores** — Fiserv DNA as a hosted platform, Temenos Transact as SaaS — began selling "modern core without the data center."
- **Cloud-native engines** — Thought Machine Vault, Mambu, nCino, and later FIS's and Fiserv's own next-gen platforms — entered the US market targeting regionals, community banks, and fintechs/BaaS (Section 3.6).

The result in 2026: **the US is simultaneously the world's most mainframe-heavy large-bank market and the world's most active market for core-modernization pilots** — an unusual combination that Section 6 explores.

### 2.5 The In-House Mainframe Reality of the Mega-Banks

The mega-banks' cores are overwhelmingly **in-house mainframe systems**, and this is unlikely to change in this decade:

- **Scale:** JPMorgan processes on the order of 300 million transactions a day (reported figure); a mega-bank's core estate runs on IBM Z mainframes with thousands of COBOL programs, IMS/DB2 databases, and CICS regions, connected by batch files, MQ queues, and (increasingly) API facades.
- **Regulation:** G-SIB supervision (CCAR, resolution planning, third-party oversight) makes a core replacement a multi-year, multi-billion-dollar, regulator-visible program — and regulators' default stance is "don't break the bank that's running."
- **Cost of replacement:** the core is entangled with every other system (payments, cards, channels, risk, reporting). A tier-one bank may run **thirty or more cores in parallel** from decades of M&A (a figure quoted in industry analyses of the legacy landscape). Replacing one is a 5–10 year program; replacing all is not on any board's agenda.

Hence the mega-bank pattern: **never replace the core; strangler-figure it** — build API layers over the mainframe, migrate product lines one at a time to new platforms, move adjacent workloads (data, analytics, payments) to the cloud, and keep the deposit engine on the mainframe for the foreseeable future. The "run the bank vs. change the bank" tension is covered in Section 4.7.

### 2.6 The US vs. the Rest of the World: Why International Cores Never Penetrated

A striking feature of the US market: the international core packages that dominate most of the world — **Temenos T24/Transact** and **Oracle FLEXCUBE** — never achieved meaningful penetration in the US. (The one notable modern exception: **Regions Bank** selected Temenos SaaS in 2023 to replace its legacy cores — Section 6.3.) The US vendor landscape instead developed **independently** around FIS, Fiserv, and Jack Henry. The reasons:

1. **Market size and self-sufficiency.** The US market was large enough to breed its own vendors early; the mainframe-era in-house tradition at the top and the Hogan/Systematics/OSI/Jack Henry tradition at the mid-market left no vacuum for imports.
2. **Regulatory differences.** US deposit accounting (Regulation D-era reserve rules, FDIC insurance reporting, the Call Report, state-level variations), the US checking-account product model, and the US regulatory reporting regime differ enough from the international model that international cores required heavy US-specific parameterization — and US examiners and auditors are conservative about foreign-vendor core dependency.
3. **Mainframe heritage.** The top tier built in-house on IBM; the mid-tier bought US vendors' mainframe products. Temenos and FLEXCUBE were open-systems-era products that arrived when the US market was already spoken for.
4. **Vendor M&A consolidation.** The US core market consolidated into three domestic champions (FIS, Fiserv, Jack Henry) with huge installed bases, switching costs, and outsourcing relationships — a classic oligopoly with high barriers to entry (Section 3.9).

---

## 3. The Vendor Landscape

### 3.1 The Vendor Map at a Glance

| Vendor | HQ | Core products | Target segment | Technology | Representative customers | Status (2026) |
|---|---|---|---|---|---|---|
| **FIS** | Jacksonville, FL | Profile, IBS, Horizon (legacy); Systematics lineage; next-gen cloud core initiatives | Mega/regional banks (legacy), community banks, international (Profile sold globally) | Mainframe + open systems; cloud push | TISCO Bank (Profile, intl.); many US regionals/community banks on Profile; Citi historically a major FIS processing client | Public co.; divested Worldpay (2023); repositioning around core+digital |
| **Fiserv** | Brookfield, WI | **DNA** (modern, real-time, person-centered), Premier, Signature, CUnify (CU), Portico (legacy CU) | Community banks, mid-size banks, credit unions | DNA is open-architecture (originally Open Solutions); Premier/legacy on mainframe/open | ~211 US companies listed on Fiserv DNA (TheirStack); thousands of community FIs; major card/processing business (First Data, 2019) | Public co.; the largest by revenue of the trio; DNA positioned as the "modern" US core |
| **Jack Henry** | Monett, MO | **SilverLake** (community bank), CIF 20/20 (legacy), **Symitar/Episys** (credit union), Banno (digital) | Community banks, credit unions | Mainframe-era cores + cloud-hosted; modern digital layer | Symitar: top CU platform since 2018 (Callahan), serving 212 billion-dollar CUs; SilverLake: thousands of community banks | Public co.; the community/CU specialist; most stable share of the trio |
| **Temenos** | Geneva, Switzerland | Transact (T24), SaaS core | Regionals (US push), private banks, wealth | Open systems, now cloud/SaaS | **Regions Bank** (2023, replacing legacy cores), Commerce Bank (live 2022); most of the rest of the world | The main international challenger making US regional wins |
| **Oracle** | Austin, TX | FLEXCUBE / Oracle Banking (OBMA) | International subsidiaries, some US mid-market | Open systems, microservices (OBMA) | Limited US core adoption; strong internationally | Niche in the US |
| **nCino** | Wilmington, NC | nCino Bank Operating System (Salesforce-based), loan origination | Community/regional banks, commercial lenders | Salesforce cloud | 1,800+ financial institutions globally (vendor claim) | Public co.; dominant US cloud loan-origination; expanding toward "core-adjacent" |
| **Thought Machine** | London / New York | **Vault** (code-first cloud-native core) | Regionals, digital banks, BaaS | Kubernetes, smart contracts, cloud-native | UK/Europe/Asia wins (Atom, SEB, Standard Chartered, Mox, etc.); US expansion since ~2021 (NYC office) | Unicorn; US bank wins not fully public as of this research |
| **Mambu** | Amsterdam / US offices | Mambu (composable SaaS core) | Fintechs, neobanks, BaaS providers, some banks | SaaS, API-first | Many fintech/BaaS deployments globally | Cloud-native; US traction mostly via BaaS/fintech |
| **Q2** | Austin, TX | Q2 digital banking platform | Community/regional banks (digital, **not** a core) | SaaS digital layer | 1,000+ financial institutions (vendor claim) | Digital-layer leader |
| **Alkami** | Plano, TX | Alkami digital banking | Banks & credit unions (digital, not a core) | SaaS digital layer | 1,000+ financial institutions (vendor claim) | Digital-layer leader |
| **CSI (Computer Services Inc.)** | Paducah, KY | CSI NuPoint (core), digital, payments | Community banks | Open systems + outsourcing | ~1,000+ community banks (vendor claim) | Independent NASDAQ co.; community specialist |

### 3.2 FIS (Fidelity National Information Services)

**Background.** FIS, headquartered in Jacksonville, FL, is one of the world's largest banking-and-payments technology companies (it describes itself as "the world's largest provider of banking and payments technology"). Its US core heritage is a museum of US banking history: **Systematics** (the Arkansas mainframe-processing pioneer, absorbed via ALLTEL in 2003), **Hogan** (the Dallas vendor whose **IBS** and **Profile** products defined 1980s-90s core architecture), and **Metavante** (the former Marshall & Ilsley processing arm, acquired 2009). FIS bought **Worldpay** in 2019 and divested a majority stake in 2023 to refocus on banking software.

**Core products.** The main US-relevant cores are **Profile** (the flagship retail core, still widely installed at US regionals and community banks and sold internationally — e.g., TISCO Bank in Thailand signed for Profile), **IBS** (Integrated Banking System, the Hogan-era mainframe core), and **Horizon** (another legacy line); the **Systematics** name survives mostly as history. On the modern side, FIS has been developing and marketing next-generation cloud-core capabilities (industry coverage refers to FIS's cloud core initiatives under names like "FIS Core"/Atmos — reported, not independently verified). FIS also dominates adjacent processing: card processing (via Worldpay heritage), payments, and digital banking.

**Market position.** FIS's US core installed base is the largest in terms of *bank size mix* — it inherited the processing relationships of the biggest banks (Citi was historically a marquee FIS processing client) and serves thousands of smaller institutions. Its challenge is the same as Fiserv's: monetizing a legacy base while the market's growth is in cloud modernization. Revenue ~$10B (post-Worldpay divestiture), public (NYSE: FIS).

### 3.3 Fiserv

**Background.** Fiserv (Brookfield, WI) is the largest of the US core trio by revenue — a position cemented by the 2019 acquisition of **First Data** (the card-processing giant), which made Fiserv a payments powerhouse alongside its core business. Fiserv grew by serial acquisition, and its core portfolio shows it.

**Core products.** The strategically important product is **DNA** — the account-processing platform developed by **Open Solutions Inc. (OSI)**, which Fiserv acquired in 2013. DNA is deliberately different from the mainframe cores: a **real-time, open-architecture, person-centered** platform (accounts hang off a person record rather than the reverse), running on open systems/cloud, and marketed as the "modern" US core. Industry trackers list ~211 US companies using Fiserv DNA (TheirStack, 2025). The legacy lines remain enormous: **Premier** (the workhorse community-bank core, one of the most-installed in the US), **Signature** (a credit-union core), **CUnify** (a modern credit-union account-processing platform), and **Portico** (a legacy CU core). Fiserv's digital stack (formerly "Fiserv Digital", plus the 2020 acquisition of Ondot for card controls) rounds out the platform.

**Market position.** Fiserv's core business is the community/mid-market: thousands of banks and credit unions, increasingly on **outsourced** processing (Fiserv runs the core). Its DNA platform is its answer to the modern-core question, and it is the platform Fiserv points at when a client wants to move off Premier/Signature/Portico without leaving the family. Public (NYSE: FI); ~$19–20B revenue.

### 3.4 Jack Henry & Associates

**Background.** Jack Henry (Monett, MO) is the community-bank and credit-union specialist — the most focused and, in some ways, the most stable of the trio. It did not chase the card-processing mega-deals; it stayed in core processing for smaller institutions and layered on digital (the **Banno** acquisition, 2018) and payments.

**Core products.** The flagship bank core is **SilverLake** (the dominant community-bank core in the US, with thousands of installations, typically run as an outsourced service), alongside the older **CIF 20/20** and **Core Director** lines. For credit unions, **Symitar** — whose account-processing platform is **Episys** — is the dominant US credit-union core: **Callahan & Associates has ranked Symitar the #1 credit-union platform every year since 2018** (December 2024 press release), and it serves **212 billion-dollar credit unions**. **Banno** is Jack Henry's modern digital-banking layer, sold to both banks and credit unions.

**Market position.** Jack Henry's model — outsourcing for community banks and credit unions — is the purest expression of the "vendor runs the core" pattern (Section 5.2). Its customer base is the long tail: thousands of institutions, each a small annuity. Public (NASDAQ: JKHY); ~$2B revenue but exceptionally high-margin and sticky. Jack Henry is also the most active of the trio in opening its cores via APIs for the fintech/embedded wave (its "Jack Henry Connected" / partner ecosystem).

### 3.5 The International Vendors in the US: Temenos and Oracle FLEXCUBE

- **Temenos** (Geneva; US HQ in New York). T24/Transact is the world's most-installed core platform outside the US. In the US it historically served private banks, wealth managers, and international banks' US operations, with limited retail penetration — the US core trio's grip held. That changed with the regional-bank modernization wave: **Regions Bank (~$155B+ assets) selected Temenos in 2023 to move from its legacy core systems to Temenos SaaS**, and **Commerce Bank** (Kansas City) completed its Temenos core transformation in early 2022; Temenos announced further US regional SaaS wins in 2026 (a "leading US regional bank," July 2026). Temenos is now the international vendor with the most credible US momentum. (For the platform itself, see [temenos_guide.md](temenos_guide.md).)
- **Oracle FLEXCUBE / Oracle Banking.** Dominant in Asia-Pacific, Africa, and parts of Europe, FLEXCUBE's US retail presence is negligible — the same story as Temenos but without the recent US wins. US deployments are mostly international banks' US subsidiaries and some mid-market niches. Oracle's more relevant US products are the database (the substrate of countless US cores) and Oracle Banking Microservices Architecture (OBMA) for modern greenfields — see [oracle_banking_microservices_architecture_guide.md](oracle_banking_microservices_architecture_guide.md).

### 3.6 The Cloud-Native Challengers: nCino, Thought Machine, Mambu

- **nCino** (Wilmington, NC; founded 2011, IPO 2020). Built **on Salesforce**, nCino's Bank Operating System (loan origination, commercial onboarding, and increasingly "core-adjacent" products) is the most successful US cloud-native banking platform by adoption — it claims 1,800+ financial institutions globally (vendor claim). It is not a deposit core; it is the modern front-end/lending layer that regionals and community banks buy instead of replacing their core. In the "augment" quadrant of modernization (Section 6.2), nCino is the default choice.
- **Thought Machine** (London; New York office since ~2021). **Vault** is the flagship code-first, cloud-native core: product behavior as "smart contracts," Kubernetes-native, API-first, with its own Vault Core language. Its wins are concentrated in the UK/Europe/Asia (Atom Bank, SEB, Standard Chartered, Mox, Allica, and others), funded by a well-publicized raise (~$83M round reported alongside its US-expansion plans; later valued above $1B). US-specific bank customers were **not verified in this research** — its US traction is real but largely through BaaS/digital-bank partnerships rather than headline regional-bank conversions (flagged honestly in the claims-status table).
- **Mambu** (Amsterdam; US offices). The composable SaaS core (the "AWS of banking" positioning) is strong with fintechs, neobanks, and BaaS providers globally. US traction is mostly in that fintech/BaaS layer rather than charter banks; no headline US bank conversion was verified in this session.

The pattern: **the cloud-native challengers win the greenfields (new banks, BaaS, digital subsidiaries) and the edges (lending, onboarding) while the legacy trio holds the deposit cores** — for now.

### 3.7 The Digital-Layer Vendors: Q2, Alkami, NCR

Most US banks do **not** buy their online/mobile banking from their core vendor. A separate digital-layer market exists, and it matters to core architecture because the digital layer is the biggest consumer of core APIs:

- **Q2** (Austin, TX) — the largest pure-play digital-banking vendor by installed base (1,000+ financial institutions claimed), serving community/regional banks and credit unions.
- **Alkami** (Plano, TX) — the challenger, growing fast among banks and credit unions (1,000+ institutions claimed).
- **NCR** (Atlanta) — digital banking (formerly D3), ATM/self-service, and retail technology; strong in the mid-market.
- The core vendors' own digital stacks — FIS Digital, Fiserv Digital (and Ondot), Jack Henry Banno — compete with the pure-plays for the same customers.

The architectural consequence: a typical community bank runs **core (vendor A) + digital (vendor B) + payments/ATM (vendor C) + loan origination (vendor D)**, stitched together by file feeds and APIs — which is why integration (Section 9.2) is the real cost center of US community-bank IT.

### 3.8 The Core-as-a-Service and BaaS Layer: Synapse, Unit, Column

Below the core vendors sits the **banking-as-a-service (BaaS)** layer that powers the neobanks — treated in depth in [programmable_business_bank_guide.md](programmable_business_bank_guide.md) and the umbrella guide's BaaS sections:

- **The sponsor-bank model.** A licensed bank (The Bancorp, Stride Bank, Cross River, Choice Financial, Column, and others) runs the core and exposes accounts/payments/cards via APIs to fintechs. The fintech (Chime, Dave, Current, etc.) owns the brand and customers but **no system of record of its own**.
- **Unit** — the leading US BaaS platform (API layer over sponsor banks), powering many US fintech programs.
- **Column NA** — a bank-chartered BaaS provider (an actual national bank built for developers).
- **Synapse** — the cautionary tale: the BaaS middleware company that **collapsed in May 2024**, stranding end-user funds held via partner banks and triggering an FDIC/regulator firestorm. The Synapse failure is the canonical case study for BaaS dependency risk (discussed in the umbrella guide); it has made US sponsors and regulators far more skeptical of "middleware BaaS" that interposes a non-bank between the core and the customer.

The BaaS stack in US terms: **sponsor bank (owns charter + core) → BaaS layer (API platform, KYC/compliance, ledger abstraction) → fintech (brand, customers, UX)**. The core behind a US BaaS program is usually one of the same trio's platforms (Fiserv DNA or Premier, Jack Henry, FIS) run by the sponsor, or a cloud-native engine (Vault/Mambu-class) if the sponsor built for BaaS (Column is the pure example). The Synapse lesson has pushed the market toward **bank-owned** BaaS layers and toward sponsors retaining direct ledger visibility — the fintech's "account balances" must trace to the sponsor's core, not to a middleware shadow ledger.

### 3.9 US Vendor Characteristics

Five structural facts about the US core-vendor market:

1. **Extreme concentration.** The Big Three (FIS, Fiserv, Jack Henry) serve the large majority of US banks and credit unions (KC Fed, March 2024). Switching costs are enormous: core conversion projects routinely take 2–4 years and are the single highest-risk IT event in a community bank's life.
2. **Outsourcing as the default.** Below the regional tier, the vendor increasingly *runs* the core (outsourced/processing services) rather than licensing software — the community bank's "core system" is a service, not an installation (Section 5.2).
3. **M&A-built portfolios.** Every major US vendor is a stack of acquired products (Systematics/Hogan/Metavante at FIS; OSI/First Data at Fiserv; Symitar/Banno at Jack Henry), which means product-line rationalization is a permanent feature — and customers are routinely "migrated within the family."
4. **The modern-core gap.** The trio's modern platforms (Fiserv DNA, FIS's next-gen core, Jack Henry's hosted SilverLake/Banno) compete with cloud-native challengers for the modernization budget, but the installed base (and its data) is the moat.
5. **The data moat.** The core holds the deposits, the transaction history, the GL, and the regulatory reporting. Even a mediocre core is extremely hard to leave, because the data migration is the project (Section 6.5).

---

## 4. The Mega-Banks' In-House Cores

### 4.1 The Self-Operated Core Pattern

The four mega-banks (JPMorgan Chase, Bank of America, Citi, Wells Fargo) are, in effect, their own core vendors. Their core estates were built in-house on IBM mainframes from the 1960s–80s and accreted through M&A — JPMorgan alone absorbed Chemical, Chase Manhattan, Bank One, First Chicago, and Bear Stearns/Wealth management systems; Wells Fargo absorbed Norwest and Wachovia; BofA absorbed NationsBank, Fleet, MBNA, and Merrill Lynch; Citi absorbed countless consumer franchises. The result is a **portfolio of cores**, not a core: retail deposits, credit cards, mortgages, small business, wealth, and every acquired bank each have (or had) their own systems of record, connected by batch files and an increasingly dense API/middleware layer.

The self-operated pattern has three pillars:

- **Scale economics.** At JPMorgan's volume (hundreds of millions of transactions a day), the per-transaction cost of a fully amortized mainframe core is hard for any vendor to beat — the mainframe is expensive but *paid for*.
- **Control and secrecy.** The core is the crown jewels: deposit pricing, fraud rules, real-time balance logic. The mega-banks do not want Fiserv or Temenos between them and their data.
- **Regulatory gravity.** G-SIB supervision, resolution planning (the "living will"), and stress testing all presume the current architecture; changing it is a supervisory event of the first order.

The four mega-bank estates at a glance (approximate 2025 figures unless noted):

| Bank | Assets (approx.) | Core posture | Tech spend | Signature modernization moves |
|---|---|---|---|---|
| **JPMorgan Chase** | $4.4T (verified, Dec 2025) | In-house mainframe; deposit/card engines on IBM Z (COBOL/IMS/DB2/CICS) | ~$18B (2025, verified) | JPMC cloud on public cloud; card-platform mainframe-to-cloud program; AI at scale |
| **Bank of America** | ~$3.3T | In-house mainframe; M&A accretion (NationsBank/Fleet/MBNA/Merrill) | ~$4B+ (reported) | Erica AI assistant (2018, 2B+ interactions reported); incremental modernization |
| **Citi** | ~$2.4T | In-house + significant vendor processing (historic FIS client); global complexity | ~$15B/3yr program (reported) | Application-portfolio simplification; cloud migration; consumer divestitures 2021–22 |
| **Wells Fargo** | ~$1.9T | In-house mainframe (Norwest/Wachovia lineage); most legacy-bound of the four | large but undisclosed | Risk/controls-first agenda post-2016; Google Cloud partnership reported |

All four are G-SIBs, all four run "run the bank" budgets of 60–75% of tech spend, and none has a public plan to replace its retail deposit core in this decade.

### 4.2 JPMorgan Chase

- **Scale.** $4.4 trillion in assets and $362 billion in equity at year-end 2025 — the largest US bank and one of the largest in the world; ~80 million US consumers (reported); roughly 5,000 branches and 14,000+ ATMs (per its own site); on the order of 300 million transactions a day processed on mainframes (reported industry figure).
- **Tech spend.** **~$18 billion in 2025** (up from ~$17 billion in 2024) — more than many countries' entire banking-IT budgets; roughly a third of that goes to "run the bank," the rest to growth/modernization (reported breakdown).
- **The core estate.** In-house mainframe systems running COBOL, IMS/DB2, and CICS, with the deposit and card engines at the center. Public confirmations are rare, but JPMorgan's own engineering blog ("Next at Chase") documents the *strangler* pattern in action — e.g., modernizing Chase Credit Card from mainframe to cloud, and a multi-year program to move the firm's data platform off legacy stores.
- **Modernization.** JPMorgan runs its **JPMC cloud** on public cloud (AWS named a preferred provider in 2020; reported) and has driven a firm-wide cloud-migration program (the "Journey" name appears in industry accounts — reported, not independently verified). The strategy is explicit: move *everything except the deposit cores* to cloud, then wrap the mainframe in APIs, then selectively re-platform product lines. JPMorgan is also the most visible US bank on AI (its LLM-based tools, fraud models, and the "IndexGPT"-adjacent research effort).
- **The lesson.** JPMorgan is the proof that the largest US core estates will **not be replaced this decade** — they will be wrapped, migrated around, and gradually decomposed, with the mainframe as the enduring system of record for deposits.

### 4.3 Bank of America

- **Scale.** ~$3.3 trillion in assets (2025, reported/approximate); ~66–70 million consumer and small-business clients (reported).
- **The core estate.** In-house mainframe systems with a similar M&A accretion (NationsBank, Fleet, MBNA, Merrill). BofA's consumer deposit and card platforms remain largely in-house mainframe; its retail technology strategy is layered on top of them.
- **Erica.** BofA's AI assistant (launched 2018) is the most famous consumer-AI deployment in US banking — over 2 billion interactions cumulatively (reported). Erica is an instructive case: a *channel* innovation that required no core replacement, built on APIs over the legacy estate — the "augment" pattern at mega-bank scale.
- **Tech posture.** BofA spends on the order of $4B+ annually on technology (reported), runs a hybrid of in-house mainframe cores, open systems, and cloud, and is methodical rather than radical: incremental modernization, heavy investment in digital channels, data, and AI.

### 4.4 Citi

- **Scale.** ~$2.4 trillion in assets (2025, reported/approximate).
- **The core estate.** Citi is the most complex of the four: a global network of country subsidiaries plus a US consumer franchise (its own + the former OneMain and the 2021–2022 consumer divestitures). Its US consumer banking was historically **vendor-processed in large part** — Citi has long been a marquee processing client of FIS — alongside in-house systems, which makes it the exception that proves the rule (the mega-banks mostly build; Citi also buys at scale).
- **Modernization.** Citi announced a multi-year technology investment program (on the order of $15B over three years, reported) and has publicly committed to modernizing its infrastructure, including moving applications to the cloud and simplifying its application portfolio (it has publicly discussed decommissioning thousands of applications). Its core-retail position in the US has shrunk after the 2021–22 divestitures (consumer banking in Asia/EMEA sold to regional buyers), focusing its US core estate on cards, wealth, and the US consumer bank it retained.

### 4.5 Wells Fargo

- **Scale.** ~$1.9 trillion in assets (2025, reported/approximate); a top-three US retail deposit franchise by branches.
- **The core estate.** In-house mainframe cores (Norwest/Wachovia lineage) with the same accretion pattern; Wells is often described as the most legacy-bound of the big four, compounded by the 2016 fake-accounts scandal that froze its growth agenda for years and forced a focus on risk and controls (the "run the bank" problem in its purest form).
- **Modernization.** Under its transformation programs, Wells has publicly talked about simplifying its technology estate, retiring legacy applications, and increasing cloud usage; it also partnered with Google Cloud (reported) — but its deposit cores remain in-house mainframe, and its public posture is "modernize around the edges."

### 4.6 Capital One: The Cloud Pioneer

Capital One (McLean, VA; ~$520B assets, 2025 reported/approximate) is the deliberate exception: a mega-scale bank (top-10 US bank) that chose the cloud-native path.

- **The AWS journey.** Capital One began moving to AWS in 2015, made AWS its primary cloud, and **announced in 2020 that it was going "all in" on AWS and exiting its data centers entirely** — the first major US bank to fully abandon its own data centers (widely documented; AWS publishes the case study). It is often described as "the first US bank on AWS."
- **The core estate.** Capital One's consumer banking is comparatively young (the bank was founded in 1994 as a credit-card spin-off) — it never carried the 1960s mainframe deposit burden. Its card platforms were modern (VisionPLUS-based, then re-platformed), and its retail bank (built from ING Direct US and others) was architected for the cloud era.
- **The lesson.** Capital One proved a US bank of scale can run on public cloud — but it is the *exception that proves the rule*: its core estate was young and small relative to the big four, its regulatory posture (large but not a G-SIB until recently; it became a G-SIB in 2024 per FSOC determinations) allowed it, and it still runs a multi-year program, not a big bang.

### 4.7 The Mega-Bank Reality: Moat, Legacy Debt, Run vs. Change

The mega-bank core reality in one paragraph: **the in-house core is a competitive moat and a legacy debt at the same time.** The moat: no vendor offers a deposit engine at JPMorgan's scale; the pricing, fraud, and real-time logic embedded in the mainframe is the bank's secret sauce, and its per-transaction economics are defensible. The debt: COBOL skills are scarce and aging (the famous "COBOL programmers are retiring" problem — a reported 200+ billion lines of COBOL are still in production across global banking, much of it in US mega-banks), batch windows constrain 24/7 real-time ambitions, and every modernization idea must negotiate a 40-year-old architecture.

This is the "**run the bank vs. change the bank**" tension: a mega-bank spends roughly 60–75% of its tech budget keeping the lights on (run), leaving a minority for change — and "change" mostly means channels, data, AI, and payments, *not* core replacement. The industry's honest assessment, echoed in [core_banking_systems_guide.md](core_banking_systems_guide.md): **core replacement is the hardest IT project in banking**, and at mega-bank scale it is not a project at all but a 10–20-year institutional capability.

---

## 5. Community Banks and Credit Unions

### 5.1 The Community Bank Core Market

Community banks (roughly 4,000 institutions under $10B in assets — the majority of US banks by count, holding a minority of assets) buy their cores from the vendor trio:

- **Jack Henry SilverLake** — the single most-installed community-bank core; typically run as an outsourced service. Jack Henry's community franchise is its moat.
- **Fiserv Premier** — the workhorse competitor with a huge installed base; **Fiserv DNA** is the migration target within the Fiserv family for community banks wanting real-time/modern architecture.
- **FIS Profile / IBS** — the third leg, with deep heritage in mid-size and community banks.
- **CSI (Computer Services Inc., Paducah, KY)** — the community-bank specialist (NuPoint core + digital + payments), a NASDAQ-listed independent that punches above its weight in the under-$1B segment (1,000+ institutions claimed).

The community-bank decision is driven by three factors: **price** (annual core fees are the biggest line item in a small bank's IT budget), **outsourcing** (see 5.2), and **integration ecosystem** (which vendors' digital, payments, and lending partners already integrate with the core — the "switch" cost is as much about the ecosystem as the core itself).

### 5.2 In-House vs. Outsourced Processing

The defining structural feature of the community/regional core market is the **outsourced core**: the vendor runs the core in its own data centers (or increasingly, its own cloud) and the bank consumes it — sometimes with a thin on-prem "in-house" license as the alternative. Two delivery models:

- **In-house license.** The bank runs the core on its own servers (mainframe or open systems) with vendor support. Declining model — requires IT staff, DR sites, and security/compliance capability small banks struggle to staff.
- **Outsourced / service bureau / cloud-hosted.** The vendor (Jack Henry, Fiserv, FIS, CSI) operates the core. The bank's "IT department" becomes vendor management + a digital-banking integrator. This is now the default for new community-bank deals and the direction of travel for existing ones.

The economics: a community bank's core-processing budget is typically on the order of **$100K–$1M+ per year** depending on size and products (reported, order-of-magnitude), with the core being the largest single IT vendor contract. The trade-off: outsourcing trades cost and staff for **vendor lock-in and concentration risk** — the FDIC/OCC's third-party-risk guidance (and the KC Fed's concentration research) treat this as a systemic issue: thousands of institutions running on a handful of cores means a single vendor's failure (or a single core conversion bug) is a systemic event.

A worked example of the community-bank stack (a fictional but representative $500M-asset bank in the Midwest):

- **Core:** Jack Henry SilverLake, outsourced (Jack Henry hosts and operates it); nightly batch, file-based statement and ACH settlement.
- **Digital:** Alkami (or Q2) online/mobile, integrated to the core via Jack Henry's API hub and file feeds.
- **Payments:** core-driven ACH origination via Jack Henry; card processing via a card processor (e.g., Fiserv First Data or Elavon) with authorization hitting a real-time balance shadow; wires via correspondent.
- **Lending:** nCino (or the core vendor's origination module) for commercial; a mortgage LOS (e.g., ICE Encompass) for mortgages.
- **Data/reporting:** the vendor's reporting suite generates the Call Report; a small data warehouse (often SQL Server) feeds board reporting and marketing.
- **Staff:** typically 1–3 IT staff + a vendor-management officer; the core vendor's account team is effectively the bank's IT department.

This bank's modernization decision in 2026 is not "which core?" but "which *ecosystem*?" — switching cores means re-integrating digital, payments, lending, and reporting, which is why the Big Three's installed base is so sticky and why the cloud-native challengers attack through lending and digital first (nCino's wedge) rather than head-on core replacement.

### 5.3 The Credit Union Core Market

Credit unions are a separate market with the same shape:

- **Jack Henry Symitar (Episys)** — the dominant credit-union core: **#1 platform every year since 2018** per Callahan & Associates, serving **212 billion-dollar credit unions** (i.e., CUs over $1B in assets — the fastest-growing segment).
- **Fiserv CUnify** — Fiserv's modern credit-union account-processing platform, plus the legacy **Signature** and **Portico** lines with large installed bases being migrated within the family.
- **CU*Answers** (Grand Rapids, MI) — the member-owned cooperative processor (CU*BASE core), the notable "we process ourselves" alternative.
- Others: **Symitar's** main challengers include Fiserv CUnify, **Corelation (Keystone)** (a newer CU core), and **Sharetec** — a fragmented tail.

Credit-union core selection is driven by the same outsourcing economics, plus two CU-specific factors: **field-of-membership** product flexibility (CU product sets differ from banks: share drafts, dividends, etc.) and **cooperative governance** (many CUs prefer processor cooperatives or vendor-neutral consultancies). The consolidation trend is identical: ~4,300 federally insured credit unions at Q4 2025, down from ~4,455 a year earlier, with the billion-dollar CUs (739 and rising) doing the buying and the core conversions.

---

## 6. The Modernization Wave

### 6.1 The Drivers

Why US banks are modernizing cores (or at least planning to) in the 2020s:

1. **Cost.** Mainframe and legacy maintenance costs escalate as COBOL skills shrink and mainframe capacity is priced by IBM at premium rates; the "run" budget crowds out everything else.
2. **Legacy debt.** Cores from the 1970s–80s cannot express modern products (real-time interest, instant account opening, configurable fees) without months of development per product.
3. **Digital expectations.** Consumers expect instant onboarding, real-time balances, and mobile-first UX; the digital layer (Section 7.5) is only as good as the core's APIs behind it.
4. **Competition.** Neobanks and big banks set the bar; community banks lose deposit share when their digital experience lags. Fintechs and BaaS providers threaten the deposit franchise itself.
5. **Regulation.** Supervisors push third-party-risk management (Section 9.5), and the CFPB's 1033 open-banking rule (Section 7.4) will force API-readiness on every covered institution.
6. **Talent.** The COBOL workforce is retiring; new engineers will not work on IMS/DB2 batch jobs — they go where the cloud and modern languages are.

### 6.2 The Four Approaches: Replace, Augment, Wrap, Migrate

US modernization practice falls into four approaches, rarely taken pure:

- **Replace** — full core replacement with a new platform: Temenos Transact SaaS (Regions Bank, Commerce Bank), Thought Machine Vault, Mambu, Fiserv DNA (family migration), or Jack Henry's hosted platforms. The hardest and rarest at scale; the norm for greenfields and small banks.
- **Augment** — keep the core, add a modern layer: digital banking (Q2/Alkami/vendor digital), loan origination (nCino), real-time payments (FedNow/RTP connectivity), fraud and data platforms. **This is where most US modernization money actually goes** — the core stays, the experience changes.
- **Wrap** — put middleware/APIs over the legacy core: API gateways, core-abstraction layers, event buses that expose the mainframe as services (the mega-bank pattern, Section 4; also the "core as a platform" style adopted by community banks with vendor-provided API hubs).
- **Migrate** — move the legacy to cheaper/more modern infrastructure without rewriting: mainframe-to-cloud rehosting (AWS Mainframe Modernization, Micro Focus/COBOL-on-Linux, IBM Z on cloud), database re-platforming. Reduces cost and skills risk without product change.

The pragmatic US answer is a **sequence**: wrap → augment → migrate → (eventually, selectively) replace. The umbrella guide's "hardest IT project in banking" warning applies most to the replace step.

### 6.3 The Core-Replacement Case Studies

Verified and notable US core-replacement programs:

- **Regions Bank** (~$155B+ assets) — selected **Temenos** in 2023 to move from its legacy core systems to a **SaaS-based modern core**, under Chief Transformation Officer Paul Weiss; roughly two years into the program by 2025–26 (industry coverage confirms Temenos is running the modernization with Regions steering the roadmap alongside **Commerce Bank**).
- **Commerce Bank** (Kansas City) — completed its **Temenos** core transformation in **early 2022** — one of the first mid-size US banks to go live on Temenos as a primary core, and now a US reference for Temenos' roadmap.
- **Temenos' 2026 momentum** — July 2026 announcement of another "leading US regional bank" selecting Temenos SaaS for core modernization (name undisclosed).
- **Capital One** — the cloud-transformation case (Section 4.6): not a vendor core replacement but the largest US re-platforming of banking workloads to public cloud.
- **The "family migrations"** — Fiserv clients moving Premier/Portico/Signature to DNA/CUnify, and Jack Henry clients moving CIF 20/20 to SilverLake — the quiet majority of "core replacement" projects, which never make the trade press because the vendor doesn't change.
- **Mega-bank partial replacements** — JPMorgan's card-platform modernization and BofA/Citi/Wells incremental product-line re-platforming (Section 4); all are selective, none is a deposit-core big bang.

Honest caveat: headline *vendor-changing* core replacements at US banks with over ~$50B in assets remain **rare events** (Regions is the marquee case); most replacement volume is inside-vendor migration and sub-$10B bank conversions, plus greenfield BaaS/digital banks on cloud-native cores.

### 6.4 Core-as-a-Service and BaaS

The BaaS layer (Section 3.8) is itself a modernization route: a bank can stop building and instead *rent* a core through a sponsor. US examples: **Column** (a developer-first national bank), **Unit** (BaaS API layer), the sponsor banks (The Bancorp, Stride, Cross River), and the neobank programs they power (Chime et al. — [programmable_business_bank_guide.md](programmable_business_bank_guide.md)). The **Synapse collapse (May 2024)** is the defining risk event: middleware-BaaS interposed a non-bank between the sponsor's core and the customer, and when it failed, funds reconciliation became a regulator-level mess. The industry response has been to prefer **sponsor-bank-native BaaS** (the bank's own core behind APIs, e.g., Column, or bank-owned middleware) over independent middleware layers.

### 6.5 The Challenges

The three canonical challenges of US core replacement:

1. **Risk.** Core conversion failures are catastrophic (failed conversions have caused multi-month outages, regulatory actions, and in extreme cases bank failures). Regulators treat a core change as a high-risk event requiring a project plan, conversion testing, parallel runs, and board oversight. The umbrella guide's thesis — "core replacement is the hardest IT project in banking" — is literally true in the US where the core is also the Call Report source (Section 9.4).
2. **Regulatory approval/notice.** While the OCC/FDIC do not formally "approve" core vendor selection, the change is subject to third-party-risk expectations (due diligence on the vendor, concentration analysis, business-continuity planning), and examiners review conversion programs; a bank switching cores while under a supervisory action faces a very high bar.
3. **Data migration.** The account/transaction data migration is the project: dozens of legacy products, historical transaction archives, GL mapping, interest recalculation, statement history, tax reporting (1099s), and the Call Report mapping — plus the integration ecosystem (digital, payments, cards, fraud) that must be re-pointed. Most conversion schedules are dominated by data mapping, not software.

---

## 7. The Technology Stack Details

### 7.1 The Mainframe Stack: IBM Z, COBOL, CICS, IMS/DB2

The mega-bank core stack, still in production at scale:

- **IBM Z** (System/390 lineage) — the hardware; z/OS the operating system.
- **COBOL** — the dominant language (with Assembler and PL/I in the deep legacy); an estimated 200+ billion lines of COBOL remain in production globally (reported industry figure).
- **CICS** — the online transaction monitor: inquiry, update, and the API facade target for modernization.
- **IMS and DB2** — the databases: IMS (hierarchical, from 1966) for the highest-volume transaction data; DB2 for relational data. Both remain in heavy use at the mega-banks.
- **Batch** — JCL, scheduler (CA-7/Control-M class tools), and the overnight batch window that still defines "end of day" for most US deposit cores.

Community-bank legacy cores (SilverLake, Premier, Profile, CIF 20/20) are also mainframe-lineage products, although increasingly rehosted or outsourced; the "modern" vendor platforms (Fiserv DNA, CUnify) are open-systems.

Why the batch window still matters: the US deposit day is a sequence of **cycles** — morning memo-post, daytime online updates (via CICS), afternoon/evening settlement postings (ACH, card, wires), and the overnight **final-post and close** that computes interest, fees, and the general-ledger interface. Regulators and customers expect specific cut-off times (e.g., ACH origination cut-offs, statement cycles), and the overnight window is when the Call Report data is extracted. Modernization pressures (24/7 instant rails, real-time balances) attack precisely this schedule: the closer a bank gets to true 24/7 posting, the more the "day" becomes a continuous flow with only a reporting close — which is the architectural difference between a legacy core with an overlay and a modern real-time core.

The COBOL skills problem quantified: industry estimates put **200+ billion lines of COBOL** still in production globally (reported), the US mega-banks and their vendor cores among the densest concentrations, while the experienced COBOL workforce retires faster than it is replaced. The standard responses are: (1) COBOL-on-modern-platforms (Micro Focus-style rehosting, AWS Mainframe Modernization), (2) API facades that let new engineers work in modern languages against the mainframe, and (3) AI-assisted code analysis/translation pilots — all of which are "migrate/wrap" strategies rather than replacement.

### 7.2 Open Systems and Databases: Oracle, SQL Server, Db2, PostgreSQL

Below and beside the mainframes:

- **Oracle Database** — the single most common database across US banking IT (countless cores, card systems, data warehouses); see [oracle_database_guide.md](../technology/oracle_database_guide.md) for the database treatment.
- **Microsoft SQL Server** — very common in mid-market and community-bank ecosystems (and in vendor platforms).
- **Db2** — on mainframe and LUW (Linux/Unix/Windows) in the vendor cores and legacy open systems.
- **PostgreSQL** — the open-source challenger, appearing in cloud-native cores (Thought Machine Vault uses PostgreSQL in its standard deployment, for example), data platforms, and fintech stacks.
- **Linux/UNIX** — the application tier for virtually every non-mainframe core and middleware product.

### 7.3 The Cloud: AWS, Azure, GCP

- **AWS** — the US banking cloud leader: Capital One (all-in, Section 4.6), JPMorgan (preferred provider for JPMC cloud, reported), and the majority of fintech/BaaS stacks. AWS Mainframe Modernization (replatforming COBOL workloads) and AWS for Financial Services are the relevant programs.
- **Microsoft Azure** — strong in banking via the Microsoft ecosystem (SQL Server, .NET-based vendor stacks) and large enterprise deals (Wells Fargo–Google Cloud reported; BofA–Azure reported for some workloads; many community-bank vendors host on Azure).
- **Google Cloud** — smaller share but present (Wells Fargo partnership reported; mainframe-modernization offerings).
- The pattern: **banks are multi-cloud by accident** (M&A, vendor choices), and regulators require it not to matter (resilience, exit strategies). Core vendors now host outsourced cores in public cloud; the "core in the cloud" is increasingly the default delivery mode for the long tail.

### 7.4 APIs and Open Banking: The CFPB 1033 Rule

US open banking arrived with the **CFPB's Section 1033 rule** — the *Personal Financial Data Rights* rule:

- **Proposed October 2023; final rule issued October 2024** (verified). It requires covered data providers (deposit accounts, credit cards, payment apps, etc.) to make consumer-permissioned data available via APIs to consumers and authorized third parties — account information, transaction history, payment initiation, and recurring-payment data.
- **Compliance is phased by size**: the largest institutions face the earliest deadlines (from ~2026, with full phased implementation into the 2030s; reported).
- The rule effectively forces **API-readiness of the core**: banks must expose machine-readable customer data, with defined performance and error-handling standards. For legacy cores this means an API facade (the "wrap" approach) or a vendor-provided open-banking layer; for modern cores it is native.
- Industry context: US open banking has been *de facto* screen-scraping-driven (Plaid et al. connecting via credentials); 1033 shifts the default to **permissioned APIs** — a structural change for the core-integration market.

### 7.5 The Digital Layer: Q2, Alkami, NCR, and the Vendor Digital Stacks

The digital-banking layer (online banking, mobile apps, account opening, card controls) is where US banks compete on experience, and it sits **on top of** the core:

- **Q2** and **Alkami** — the leading pure-play platforms (Section 3.7), each serving 1,000+ institutions (vendor claims).
- **NCR** — digital + ATM/self-service; strong mid-market presence.
- **Vendor digital stacks** — FIS Digital, Fiserv Digital (+ Ondot card controls), Jack Henry **Banno** — bundled with the core contract.
- The integration pattern: the digital layer consumes core APIs (balances, transactions, transfers), payment rails (cards, ACH, instant), and data platforms — and is usually the first thing a bank modernizes, because it is replaceable without touching the core (the "augment" quadrant).

### 7.6 Payments: FedNow, RTP, ACH, and the Card Networks

- **FedNow** — the Fed's instant-payment rail, **launched July 2023** (verified; see [financial_infrastructure_guide.md](financial_infrastructure_guide.md)); participation has grown steadily since. Real-time posting pressure on cores.
- **RTP** — The Clearing House's real-time rail, live since **2017** (verified); the private-sector instant network.
- **ACH (Nacha)** — the batch backbone: ~33+ billion payments and ~$85T+ per year (reported, 2024–25 era); same-day ACH (2016+) added intraday settlement windows.
- **Cards** — Visa/Mastercard/Amex/Discover: authorization touches the core or a real-time balance shadow; settlement via batch files. Card systems (Fiserv First Data, TSYS/Global Payments, FIS) are a parallel "core" for most US banks' card books.
- **Wire** — Fedwire and CHIPS for large-value (Section 1.3).
- The core-architecture consequence: **dual-speed processing** — batch (ACH, cards settlement) and real-time (FedNow, RTP, card auth) against the same accounts. Modern cores handle both natively; legacy cores need an "instant-payments overlay" that posts real-time debits/credits into a batch-oriented ledger — one of the most common US core-integration projects of the mid-2020s.

The dual-speed ledger problem in practice: a legacy core's day is a sequence of batch cycles (memo-post during the day, final-post overnight), while FedNow/RTP demand **immediate, irrevocable posting**. The standard US pattern is an **overlay ledger**: the instant-payment hub holds real-time balances and posting intentions, syncs with the core via API during the day, and reconciles into the overnight batch — with float, holds, and overdraft logic mirrored in both places. Getting the reconciliation right (and the fraud controls: instant rails are irrevocable, so pre-posting screening is mandatory) is one of the hardest integration projects in US banking, and it is why "instant-payments readiness" has become a core-selection criterion.

The modern-core advantage is native support: Fiserv DNA, Temenos, Vault, and Mambu-class platforms post in real time by design, with the batch window shrinking to reporting/close-of-day processes. For the legacy majority, the overlay is the only realistic path — another reason the "wrap/augment" quadrant dominates US spend.

---

## 8. The US vs. Other Markets

### 8.1 US vs. China: The Comparison Table

The US and China are the two poles of global core-banking strategy, and the contrast is the most instructive comparison in the domain (the China side is detailed in [chinese_bank_core_systems_guide.md](chinese_bank_core_systems_guide.md)):

| Dimension | US | China |
|---|---|---|
| **Market structure** | ~4,400 banks + ~4,300 credit unions; private ownership; no state-owned mega-tier | ~4,000+ institutions, dominated by state-owned giants (ICBC et al.); Big Four serve billions |
| **Core ownership** | **Vendor-dominated** (FIS/Fiserv/Jack Henry oligopoly) for the long tail; in-house mainframe only at the mega-banks | **Self-development (自主研发)** is the norm for big banks; domestic vendors (Sunline, DCITS, GienTech, Yusys) for the mid/lower tiers |
| **Legacy platform** | Mainframe (IBM Z/COBOL/CICS/IMS-DB2) at the top; mainframe-lineage vendor cores below | Mainframe at the Big Four (IBM heritage) — but being **decommissioned** in the distributed wave |
| **Modernization direction** | **Incremental**: wrap → augment → migrate → selective replace; mainframe stays; cloud for adjacent workloads | **Revolutionary**: 去IOE (de-IOE), distributed cores on commodity hardware + domestic databases (OceanBase, GoldenDB, GaussDB), xinchuang (信创) domestic-stack mandate |
| **Cloud** | Public cloud accepted (Capital One all-in; JPMC cloud) | Domestic cloud (Huawei Cloud, Alibaba Cloud, Tencent Cloud); foreign public cloud largely out of reach for core workloads (data-localization rules) |
| **Database landscape** | Oracle dominant, SQL Server, Db2, rising PostgreSQL | Domestic substitution: OceanBase, GoldenDB, GaussDB, DM, Kingbase — Oracle/DB2 being phased out under 信创 |
| **Digital banks** | Neobanks = fintech + bank partner (Chime, Varo, SoFi); Varo/SoFi later chartered | Digital banks are licensed banks backed by tech giants (WeBank, MYbank) with full cloud-native cores |
| **State direction** | None — market-driven, regulator-skeptical of concentration but no industrial policy | Strong — 信创 policy, data laws, state IT giants; technology policy is national strategy |
| **Core replacement appetite** | High risk-aversion; marquee vendor-changing deals are rare (Regions/Temenos) | Large-scale new-generation core programs at the big banks (ICBC, CCB, CMB et al.) — driven by state policy and scale pressure |
| **Vendor concentration** | Big Three serve most institutions (a supervisory concern) | Fragmented domestic vendor market + in-house; state-owned SI giants |
| **Regulatory style** | Dual banking, multiple agencies, rules-based, 1033 open banking now forcing APIs | Single-party state direction via PBOC/NFRA; data localization; tech-substitution mandates |

**What each side can learn:** the US shows the value of *stability and switching-cost awareness* — decades of running on a proven core, with modernization executed in low-risk increments — and the dangers of *vendor concentration without an exit strategy* (the KC Fed's concern). China shows that *large-scale core transformation is possible* when the state, the vendors, and the banks align — but at a cost (state dependency, vendor ecosystem lock-in to domestic stacks) that the US would not accept.

### 8.2 US vs. Europe

- **Vendor mix.** Europe runs Temenos, SAP (Banking Services), FIS, Fiserv, and the Indian vendors (TCS BaNCS, Infosys Finacle) far more than the US. The "Big Three" US trio barely exists in Europe; conversely, Temenos' European dominance never transferred to the US.
- **Architecture era.** European banks have more open-systems cores and fewer mainframes than US mega-banks (though HSBC, Deutsche, and others carry mainframe heritage); Europe also led on **instant payments (SEPA Instant)** and **PSD2 open banking**, which forced API layers onto European cores a decade before the US 1033 rule.
- **Regulatory contrast.** PSD2 mandated open APIs (2018–19) with a clear legal framework; the US is only now (2024 rule, phased compliance) following with a market-driven version. Europe's GDPR and national supervisors add data-governance layers the US lacks federally.
- **What each can learn:** Europe's PSD2 experience is the playbook (and warning) for US 1033 — API-readiness deadlines, third-party access standards, and the screen-scraping-to-API transition. The US's vendor-out-sourcing model for small banks is ahead of Europe's fragmented mid-market, where many small banks still run in-house cores.

### 8.3 US vs. Emerging Markets: The Leapfrog

- **The leapfrog.** Emerging markets (India, Brazil, Nigeria, parts of SEA) built banking scale **after** the cloud era: UPI in India and PIX in Brazil are cloud-era rails with API-native cores; new banks there routinely start on Finacle, FLEXCUBE, Mambu, or Vault in the cloud. The US cannot leapfrog — its cores are already built and paid for (the mainframe "moat").
- **The consequence.** The US modernizes by *subtraction* (wrapping and retiring legacy) while emerging markets modernize by *addition* (greenfield cloud cores). US community banks are effectively in the same position as emerging-market banks in one respect: their outsourced cores mean they, too, can "leapfrog" by switching to SaaS — which is exactly what the cloud-native challengers are selling.
- **The implication for global banks.** A global bank (e.g., a Crédit Agricole operating in the US) faces three different core regimes: the US (vendor/outsourcing, incremental), Europe (open-banking-era cores), and emerging markets (cloud-native greenfields). Group architecture must therefore be **core-agnostic at the edges**: standard APIs and data contracts at the group level, tolerant of wildly different cores underneath.

### 8.4 US-Specific Dynamics and Global Implications

- **Regulatory fragmentation.** The dual banking system and 50-state regulation make US core deployments harder to standardize than in single-supervisor markets — a foreign bank entering the US faces OCC/Fed/FDIC/state/CFPB layers, and core choices are reviewed through that lens.
- **M&A-driven core consolidation.** US bank M&A is constant (the FDIC count falls by ~100–200 institutions a year through mergers), and every merger is a core-conversion project (usually the acquirer's core wins; sometimes the target's is better and survives). Bank consolidation *is* core consolidation.
- **Vendor consolidation.** FIS/Fiserv/Jack Henry consolidated the US core market; the next round is cloud-native consolidation (which challengers survive, which get acquired — e.g., the perennial speculation that FIS/Fiserv buy their way into the modern-core market).
- **What global banks can learn from the US:** (1) the *outsourced core* model for small entities (a subsidiary or digital bank can run on a vendor SaaS core instead of building); (2) the *strangler-figure* approach for legacy (never replace the core, wrap it); (3) the *regulator-as-stakeholder* discipline — US banks treat core risk as a board-level, examiner-visible issue, which is good practice everywhere.
- **What the US can learn from the world:** from China, that transformation *can* be accelerated when there is an industrial-policy tailwind; from Europe, that open-banking deadlines arrive and API-readiness must be planned years ahead (1033 is the US's PSD2 moment); from emerging markets, that greenfield cloud-native cores are now cheap and proven — the US's own neobanks/BaaS layer is the proof.

---

## 9. The US Core in Practice: An Architect's View

### 9.1 The US Bank Architecture: Core and Surroundings

The canonical US bank architecture (for a mid-size bank) looks like this:

```
                ┌─────────────────────────────────────────────┐
                │  Channels: online/mobile (Q2/Alkami/vendor)  │
                │  Branches/ATM (NCR), call center, fintechs   │
                └──────────────────────┬──────────────────────┘
                                       │ APIs / files
                ┌──────────────────────▼──────────────────────┐
                │  Digital/experience layer: account opening,  │
                │  loan origination (nCino), card controls     │
                └──────────────────────┬──────────────────────┘
                                       │ APIs / files
   ┌─────────────┬─────────────────────▼──────────┬───────────────────┐
   │  CORE       │  Payments hub                  │  Data platform    │
   │  (SilverLake│  (FedNow/RTP/ACH/cards/wires)  │  (warehouse,      │
   │  /Premier/  │  + fraud screening             │  reporting,       │
   │  DNA/Profile│                               │  analytics/AI)    │
   └─────────────┴───────────────────────────────┴───────────────────┘
```

The core is the **system of record for accounts, transactions, interest, and fees**; the payments layer moves money across the rails (Section 7.6); the digital layer is the experience; the data platform feeds the Call Report, risk, marketing, and AI. The integration glue (9.2) is where most cost and risk live.

### 9.2 Integration Patterns: APIs, File Feeds, ETL

- **APIs (REST/JSON)** — modern cores expose them natively (Fiserv DNA, cloud-native cores); legacy cores expose them via vendor API hubs (Jack Henry's open platform, FIS/Fiserv API gateways) or bank-built facades. The API layer is the *de facto* modernization surface: most "core modernization" deals are really API-layer deals.
- **File feeds (batch)** — still the workhorse for ACH settlement files, card settlement, statements, and regulatory reporting. Nacha file formats, BAI/MT940-style statements, and ETL into the data warehouse.
- **ETL/data integration** — the core's data is copied (CDC or batch) into warehouses/lakes for reporting and analytics; real-time needs (fraud, instant payments) increasingly use event streams/CDC.
- **The pattern:** **"API in the front, files in the back"** — customer-facing integration is API-based; settlement and reporting remain batch. Modernizing means pushing the batch boundary outward.

The integration surface in summary:

| Integration | Typical pattern | Examples | Modernization trend |
|---|---|---|---|
| Channels → core | REST APIs (via vendor API hub or bank facade) | Balance inquiry, transfers, account opening | API-first; 1033 will standardize |
| Payments in (ACH/cards settlement) | Batch files (Nacha, card settlement) | Nightly ACH posting files | Moving to API/real-time where rails allow |
| Payments out (FedNow/RTP) | Real-time API + overlay ledger | Instant credit/debit postings | The defining integration project of the 2020s |
| Core → data platform | Batch ETL / CDC | Warehouse, Call Report, AML feeds | CDC/streaming for real-time risk |
| Core → regulatory | Vendor reporting modules / ETL | FFIEC 031/041, FR Y-9C, HMDA | Standardized XBRL/API filing |
| Core → fintech/BaaS | Vendor APIs + partner programs | Jack Henry Connected, FIS/Fiserv partner APIs | Opening the core for embedded finance |

### 9.3 Core Data: Accounts, Transactions, and the Data Model

The US core's data model follows the classic account-centric pattern (detailed in [data_models_banking_insurance_guide.md](data_models_banking_insurance_guide.md)):

- **Party/customer** — person or business; **Fiserv DNA's "person-centered" model** is the notable inversion (party first, accounts attached) that its marketing contrasts with account-centric legacy cores.
- **Accounts** — DDA (checking), savings, CDs, loans, with product/parameter tables driving interest and fees; US specifics: Reg D-era savings-transaction limits (removed in 2020), overdraft/NSF processing (a huge US revenue and regulatory topic), and the general ledger integration.
- **Transactions** — postings with float, memo-post vs. posted, holds, and the batch day-cycle; card authorizations hit real-time shadow balances.
- **Derived data** — the Call Report schedules, 1098/1099 tax reporting, escheatment (unclaimed property), and AML/monitoring feeds all derive from core data.

### 9.4 Regulatory Reporting: The Call Report and FFIEC

- **The Call Report** — the quarterly **FFIEC 031/041** report of condition and income that every bank files, generated from core data: balance sheet, income statement, loan and deposit detail, capital. The core vendor's reporting module (or a data-platform build) produces it; errors are examinable.
- **FFIEC** (Federal Financial Institutions Examination Council) — the interagency body that standardizes the report forms and examiner guidance across OCC/Fed/FDIC/NCUA/state supervisors.
- **Other core-derived reports** — FR Y-9C (holding-company report), deposit-insurance assessment data, HMDA (mortgage disclosure), CRA data, and AML/BSA reporting (via the core's transaction feeds).
- The architect's point: **the core is the source of regulatory truth**, and any core change (including data-model changes like DNA's person-centered model) must re-validate the full reporting chain — a real hidden cost of modernization.

### 9.5 Risk: Concentration Risk and Third-Party Risk

- **Core-system risk** — the OCC/FDIC treat the core as a critical system: operational risk (outages, conversions), third-party risk (vendor viability, concentration), and cyber risk (the core and its data are prime targets). Regulators have published guidance on third-party risk management (OCC Bulletin 2013-29 and the 2023 interagency guidance) and have publicly worried about **concentration** in the core market (the KC Fed's March 2024 research brief is the canonical citation: the Big Three serve the large majority of institutions; a single vendor incident is a systemic event).
- **The 2023 bank failures** (SVB, Signature, First Republic) sharpened the focus: none failed *because of* a core, but the run-on-deposits dynamics (instant withdrawals via digital channels) made **real-time balance/liquidity visibility** a supervisory demand — pushing cores toward real-time data access.
- **BaaS/third-party risk** — the Synapse collapse (May 2024) made sponsor banks and regulators re-examine BaaS middleware; see [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md) for the risk-management treatment.

### 9.6 Assessment and Vendor Selection

A pragmatic framework for assessing a US bank's core and choosing a path (used in practice by consultants and SIs):

1. **Inventory** — map the core estate: systems of record, product lines, age, batch windows, data flows, integration surface (the "thirty cores" problem starts with counting them).
2. **Score the core** — product flexibility (days-to-launch a new product), real-time capability (instant-payments readiness), API coverage, reporting burden, run cost (mainframe MIPS, COBOL staff), and vendor relationship (support quality, roadmap, outsourcing terms).
3. **Choose the trajectory** — Replace (greenfields, small banks, family migrations, marquee deals like Regions), Augment (most banks: digital, nCino, instant-payments overlay), Wrap (mega-banks: API facades), or Migrate (rehost to cloud/COBOL-on-Linux to cut run cost).
4. **Run the selection process** — for a replacement: functional fit to US products (checking/overdraft/CD/loan types), data-migration approach, integration ecosystem (which digital/payments/fraud vendors already integrate), regulatory posture (vendor's examiner track record), outsourcing/cloud delivery model, and total cost over 10 years (fees + migration + parallel run + decommissioning).
5. **Plan the conversion** — the umbrella guide's rules apply: parallel runs, conversion dress rehearsals, data reconciliation, customer communication, and examiner engagement.

The US-specific selection truth: **for banks under ~$10B, the vendor ecosystem (digital, payments, lending partners) usually decides the deal** — a bank does not buy a core, it buys the ecosystem around it.

A scoring skeleton used in US core-selection exercises (weights vary by bank):

| Criterion | Typical weight | What to look for (US specifics) |
|---|---|---|
| US product fit | 25% | Checking/overdraft/NSF, CDs, escrow, Reg CC holds, state-law variations |
| Data migration plan | 20% | Historical archives, GL mapping, interest recalc, 1099s, escheatment |
| Integration ecosystem | 15% | Digital (Q2/Alkami/vendor), payments (FedNow/RTP/ACH), lending (nCino), card processors |
| Real-time/API capability | 10% | Native instant-posting, API coverage, 1033 readiness |
| Regulatory posture | 10% | Vendor examiner track record, SOC/audit, concentration analysis, exit strategy |
| Delivery model | 10% | Outsourced/SaaS vs. in-house; cloud hosting; DR/BCP |
| Total cost (10-yr) | 10% | Fees + migration + parallel run + decommissioning + staffing |

In practice the top two criteria (product fit and data migration) decide most deals; the rest differentiate. For the mega-banks the exercise is inverted: they score *whether to replace at all* (almost never) versus wrap/augment/migrate options.

---

## 10. The Future: 2026 and Beyond

### 10.1 Cloud Core Adoption

The "core in the cloud" is becoming the default delivery mode rather than a novelty:

- **Vendor-hosted SaaS** for the long tail (Fiserv DNA/outsourcing, Jack Henry hosted, FIS cloud, CSI) — most new community-bank deals are cloud-hosted.
- **Mainframe-to-cloud rehosting** at the mid-market (AWS Mainframe Modernization, Micro Focus COBOL-on-Linux) to cut run cost and skills risk without rewriting.
- **Mega-bank cloud** continues around the mainframe (JPMC cloud, Capital One all-in, BofA/Citi/Wells incremental) — the deposit core stays, everything else moves.
- Expect 1033 (Section 10.5) and instant payments to keep forcing **API-first delivery** even from mainframe cores.

The "core in the cloud" by tier in 2026:

| Tier | Typical cloud posture | Examples |
|---|---|---|
| Mega-banks | Public cloud around the mainframe; deposit cores on-prem | JPMC cloud (AWS); Capital One all-in on AWS; BofA/Citi/Wells incremental |
| Super-regionals/regionals | Hybrid: vendor-hosted cores, cloud digital/data; selective rehosting | Regions (Temenos SaaS); Commerce Bank; Capital One (cloud-native) |
| Community banks | Vendor-hosted (outsourced) cores in vendor cloud; SaaS digital | SilverLake/Premier/DNA hosted; Q2/Alkami SaaS |
| Credit unions | Same as community banks; Symitar hosted by Jack Henry | Symitar/Episys hosted; CUnify |
| Neobanks/BaaS | Cloud-native from day one (sponsor core or own) | Chime (sponsor cores), Column, Varo/SoFi post-charter |

### 10.2 The Core Replacement Wave

The 2026+ race is real but narrower than the hype:

- **The modern-core candidates** — Temenos (Regions momentum, SaaS model), Thought Machine Vault (regional/digital wins outside the US; US traction via BaaS/greenfields), Mambu (fintech/BaaS), Fiserv DNA (family migration), Jack Henry hosted (community), nCino (lending/core-adjacent).
- **The realistic wave** — hundreds of sub-$10B banks and credit unions converting in the next decade (vendor-family and SaaS conversions), dozens of regionals doing selective replacement (lending, cards, digital-first subsidiaries), and a handful of headline deals (Regions-class). **Full deposit-core replacement at the mega-banks remains off the table** through 2030.
- **The wildcard** — if a large regional executes a Temenos/Vault conversion cleanly at scale, the confidence effect could trigger a genuine wave; if one fails publicly, the wave stalls. The industry is watching Regions.

Scenario view of the 2026–2030 replacement wave:

| Scenario | Likelihood | Trigger/conditions | Consequence |
|---|---|---|---|
| **SaaS conversions of the long tail** | High | 1033 API deadlines; vendor pricing; examiner push on third-party risk | Hundreds of community banks/CUs move to vendor-hosted modern cores (Fiserv DNA/CUnify, Jack Henry hosted) |
| **Regional selective replacement** | High | Digital-first subsidiaries, lending modernization, instant-payments readiness | nCino/Temenos/Vault win lending and subsidiary deals; deposit core stays |
| **Marquee full replacement (Regions-class)** | Medium | Successful Regions/Temenos execution; board appetite | 2–4 more large regionals commit to full core replacement this decade |
| **Mega-bank deposit-core replacement** | Very low | None foreseeable | Wrapped mainframe remains the system of record through 2030+ |
| **Challenger consolidation** | Medium | Funding environment; Big Three acquisitions | Mambu/Thought Machine/nCino-class consolidation into or alongside the trio |

### 10.3 BaaS and Embedded Finance

- BaaS continues to grow (sponsor-bank-native models preferred post-Synapse; Column-class banks; bank-owned middleware).
- Embedded finance (cards, payments, lending inside non-bank products) keeps pushing core capabilities out via APIs — the "programmable bank" theme of [programmable_business_bank_guide.md](programmable_business_bank_guide.md).
- Expect the 1033 rule and instant payments to make **bank-owned API platforms** a competitive necessity, not an option.

### 10.4 AI in the Core

- **AI on core data** — fraud detection (real-time transaction scoring), credit decisioning, collections, and customer service (Erica-class assistants) — all consuming core data via APIs/streams without touching the core itself.
- **AI in the core** — parameterized product configuration, automated reconciliation, anomaly detection in batch, and (experimentally) COBOL-to-modern-language translation (mainframe-modernization vendors sell AI-assisted re-platforming).
- The realistic 2026+ view: AI modernizes *around* the core (data, channels, risk) long before it replaces the core; but AI-assisted migration tools materially lower the cost of the replace/migrate quadrants.

### 10.5 Open Banking and the 1033 Compliance Clock

- The CFPB's **Personal Financial Data Rights final rule (October 2024)** phases in compliance starting with the largest institutions (~2026) and extending into the 2030s (reported).
- The practical effect: **every covered bank must expose permissioned APIs for account data and payment initiation** — a hard deadline for legacy cores that only speak files, and a tailwind for modern cores and API platforms.
- The open question is the 2025–26 political environment (the CFPB's rule survived legal challenge proceedings; its implementation pace under a new administration is a live risk — flagged, not resolved, here).

### 10.6 Consolidation: Vendors and Banks

- **Bank M&A continues** (the ~4,400 institution count keeps falling), each deal a core-conversion project — the quiet majority of "modernization" spend.
- **Vendor consolidation continues** — expect the Big Three to buy or build cloud-native capabilities (Fiserv's DNA, FIS's next-gen core, Jack Henry's hosted stack are the current answers; acquisitions of challengers remain likely), and expect challenger consolidation (which of Thought Machine/Mambu/nCino and the digital players reach escape velocity).
- **Supervisory attention on concentration** (KC Fed research; FDIC/OCC third-party-risk scrutiny) may force **vendor-neutral interoperability** expectations — APIs, data portability (1033 reinforces this), and realistic exit strategies — which would be the single biggest structural change to the US core market.

### 10.7 Trends Summary

1. **The mainframe stays** — the mega-banks' deposit cores remain in-house mainframe through 2030; modernization is wrap/augment/migrate, not replace.
2. **The long tail moves to SaaS** — community banks and credit unions converge on vendor-hosted/cloud cores; the "outsourced core" is now the default.
3. **Temenos becomes the US regional replacement play** — Regions/Commerce validate the model; the industry watches the execution.
4. **Cloud-native cores win the greenfields** — Vault/Mambu/nCino-style platforms power new banks, digital subsidiaries, and BaaS; FIS/Fiserv respond with modern platforms and family migrations.
5. **1033 forces API-readiness** — open banking arrives on a compliance clock, reshaping core integration and data access.
6. **Instant payments force dual-speed cores** — FedNow/RTP real-time posting against batch-era ledgers is the defining integration project of the mid-2020s.
7. **AI modernizes around the core** — data, risk, channels, and (soon) migration tooling; the core itself is the last thing AI touches.
8. **Concentration risk becomes a strategy issue** — vendor exit strategies and data portability move from technical niceties to board-level and supervisory demands.

---

## 11. Glossary

- **Mega-bank / money-center bank** — the largest US banks (JPMorgan Chase, Bank of America, Citi, Wells Fargo); G-SIBs with in-house mainframe cores and tens of billions in annual tech spend.
- **Super-regional** — banks just below the big four: US Bancorp, PNC, Truist, TD Bank US, Capital One (retail/deposit sense); loosely also the wholesale giants Goldman Sachs and Morgan Stanley.
- **Regional bank** — the ~$10B–$100B+ tier (Regions, Fifth Third, KeyBank, Huntington, Citizens, etc.); the core-replacement battleground.
- **Community bank** — small banks, generally under $10B in assets (FDIC's revised definition), the majority of US banks by count; typically run on outsourced vendor cores.
- **Credit union** — member-owned, tax-exempt depository cooperatives (~4,300 federally insured) regulated by the NCUA; core market dominated by Jack Henry Symitar.
- **Neobank** — app-based fintech without (usually) its own charter; runs on a sponsor bank's core via BaaS (Chime, Varo, SoFi, Dave, Current).
- **FDIC** — Federal Deposit Insurance Corporation: deposit insurer, supervisor of state non-member banks, receiver of failed banks.
- **OCC** — Office of the Comptroller of the Currency: charters/supervises national banks.
- **Federal Reserve** — central bank: supervises state member banks and holding companies; operates Fedwire and FedNow.
- **CFPB** — Consumer Financial Protection Bureau: consumer-protection rulemaker; author of the 1033 open-banking rule.
- **NCUA** — National Credit Union Administration: federal credit-union regulator/insurer.
- **Dual banking** — the US system in which banks may take a national charter (OCC) or a state charter (state regulator), with overlapping federal supervision.
- **National charter / state charter** — the two licensing paths under the dual banking system.
- **Basel III / Basel III Endgame** — international capital standards; the US "Endgame" proposal (2023) and final rule (late 2025, reported) raise large-bank capital requirements.
- **Dodd-Frank** — the 2010 financial-reform law; created the CFPB and FSOC, enhanced prudential standards.
- **Mainframe** — the IBM Z-class computers (z/OS, COBOL, CICS, IMS/DB2) that run the mega-banks' deposit cores.
- **COBOL** — the dominant core-banking language since 1959; an estimated 200+ billion lines still in production (reported).
- **CICS** — IBM's online transaction monitor; the interactive face of the mainframe core.
- **IMS / DB2** — IBM databases: IMS (hierarchical, 1966) and DB2 (relational, 1983); the mega-bank core databases.
- **FIS** — Fidelity National Information Services (Jacksonville, FL): Profile/IBS/Horizon/Systematics heritage; one of the Big Three.
- **Fiserv** — the Brookfield, WI processing giant: DNA, Premier, Signature, CUnify; owns First Data (payments).
- **Jack Henry** — Jack Henry & Associates (Monett, MO): SilverLake, CIF 20/20, Symitar/Episys, Banno; community/CU specialist.
- **SilverLake** — Jack Henry's dominant community-bank core.
- **Symitar / Episys** — Jack Henry's dominant credit-union platform (Episys is the processing platform).
- **Profile / IBS / Horizon** — FIS's mainframe-era cores (Hogan/Systematics lineage).
- **DNA** — Fiserv's modern real-time, person-centered core (ex-Open Solutions).
- **Premier** — Fiserv's workhorse community-bank core.
- **Signature / Portico / CUnify** — Fiserv's credit-union cores (legacy and modern).
- **CSI / NuPoint** — Computer Services Inc. (Paducah, KY) and its community-bank core.
- **CU*Answers / CU*BASE** — the credit-union cooperative processor and its core.
- **nCino** — the Salesforce-based cloud banking platform (loan origination + Bank Operating System), Wilmington, NC.
- **Thought Machine / Vault** — the London cloud-native core vendor and its code-first core engine.
- **Mambu** — the Amsterdam composable SaaS core vendor.
- **Q2 / Alkami** — the leading US pure-play digital-banking (non-core) platforms.
- **FedNow** — the Fed's instant-payment rail (launched July 2023).
- **RTP** — The Clearing House's real-time payments rail (launched 2017).
- **Nacha / ACH** — the batch Automated Clearing House network (~33B+ payments/year, reported).
- **Fedwire / CHIPS** — large-value wholesale rails (Fedwire: Fed RTGS; CHIPS: private netting).
- **Call Report** — the quarterly FFIEC 031/041 regulatory report generated from core data.
- **FFIEC** — Federal Financial Institutions Examination Council; standardizes report forms and examination.
- **1033** — Section 1033 of Dodd-Frank; the CFPB's Personal Financial Data Rights final rule (October 2024) mandating consumer-permissioned data APIs.
- **BaaS** — Banking-as-a-Service: licensed banks exposing core/accounts/payments via APIs to fintechs.
- **Core replacement / core conversion** — replacing a bank's system of record; "the hardest IT project in banking."
- **COBOL skills** — the shrinking pool of mainframe programmers; a key modernization driver.
- **Concentration risk** — the supervisory concern that FIS/Fiserv/Jack Henry serve the majority of US institutions, making a single vendor failure systemic.

---

## 12. Claims-Status Table

Per repo convention (see [green_link_digital_bank_guide.md](green_link_digital_bank_guide.md) Section 5), US-specific facts are graded below. **Verified** = retrieved from a source in this session (web search results); **Reported** = widely cited in industry literature but not independently verified here; **Vendor claim** = company marketing; **Unverified** = could not be checked.

| Claim | Status | Notes / source type |
|---|---|---|
| ~4,400 FDIC-insured banks: 4,421 in Q1 2025; 4,379 in Q3 2025 | **Verified** | FDIC Quarterly Banking Profile press releases (govdelivery; cranedata citing FDIC) |
| ~4,287 federally insured credit unions at Q4 2025 (4,455 at Q4 2024) | **Verified** | NCUA press release (Mar 2026) |
| JPMorgan Chase $4.4T assets, $362B equity at Dec 31, 2025 | **Verified** | JPMorganChase 2025 annual report / Q4 2025 results press release |
| JPMorgan tech spend ~$18B (2025), ~$17B (2024) | **Verified (secondary)** | Industry press citing investor materials (ainvest.com) |
| JPMorgan ~80M US consumers; ~300M transactions/day; ~5,000 branches/14,000+ ATMs | **Reported** | Company-era figures/industry quotes; chase.com cites 14,000+ ATMs and 5,000 branches |
| JPMorgan "Journey" modernization program name; AWS preferred provider (2020) | **Reported** | Industry accounts; JPMC cloud documented but program name not verified this session |
| Chase Credit Card mainframe-to-cloud modernization program | **Verified (secondary)** | JPMorgan engineering blog ("Next at Chase," Medium) |
| BofA ~$3.3T assets; Erica launched 2018, 2B+ interactions | **Reported** | Widely documented; not re-verified this session |
| Citi ~$2.4T assets; ~$15B/3yr tech investment; major FIS processing client historically | **Reported** | Industry reporting; Citi tech-investment announcements |
| Wells Fargo ~$1.9T assets; Google Cloud partnership | **Reported** | Industry reporting |
| Capital One "all in on AWS", exiting data centers (announced 2020); first major US bank fully on AWS | **Reported** (widely documented; AWS publishes the case study) | AWS case study + industry coverage |
| Capital One became a G-SIB in 2024 | **Reported** | FSOC determinations coverage |
| FIS/Fiserv/Jack Henry = "Big Three" with high market concentration | **Verified** | Kansas City Fed research brief (Mar 2024) |
| Legacy platform names: Fiserv DNA & Premier, FIS IBS & Horizon, Jack Henry SilverLake & Core Director; "a tier-one bank might have thirty cores" | **Verified (secondary)** | TechBullion market analysis |
| Fiserv DNA: real-time account processing, person-centered data model; ~211 US companies listed | **Verified** | fiserv.com DNA page; TheirStack tracker |
| Fiserv CUnify exists as CU account-processing platform | **Verified (secondary)** | TheirStack reference |
| Symitar = top credit-union platform every year since 2018; 212 billion-dollar CUs | **Verified** | Jack Henry press release citing Callahan & Associates (Dec 2024) |
| SilverLake dominant community-bank core; Banno digital | **Reported** | Industry consensus; vendor materials |
| FIS Profile used by TISCO Bank (Thailand) | **Verified** | Finextra press release |
| FIS next-gen cloud core ("FIS Core"/Atmos naming) | **Reported** | Industry coverage; names not verified |
| CSI NuPoint: ~1,000+ community banks | **Vendor claim** | CSI marketing; company is real (NASDAQ: CSVI) |
| CU*Answers CU*BASE cooperative processor | **Reported** | Industry knowledge; not verified this session |
| Regions Bank (~$155B+) selected Temenos (2023) to move from legacy to SaaS core; CTO Paul Weiss | **Verified** | Temenos press + Tearsheet interview coverage |
| Commerce Bank completed Temenos core transformation early 2022 | **Verified** | Temenos press (April 2023) |
| Another leading US regional bank selected Temenos SaaS (July 2026) | **Verified** | Temenos press release |
| Thought Machine: $83M raise + US expansion plans; NYC presence | **Verified (secondary)** | learnbonds; Similarweb NYC traffic |
| Thought Machine US bank customers (named) | **Unverified** | Not retrievable this session; US traction reported but unnamed |
| Mambu US bank conversions | **Unverified** | US traction mostly fintech/BaaS; no named bank verified |
| nCino: Salesforce-based, 1,800+ FIs | **Vendor claim** | nCino materials |
| Q2/Alkami: 1,000+ FIs each | **Vendor claim** | Company materials |
| FedNow launched July 2023; RTP launched 2017 | **Verified** | Repo sibling financial_infrastructure_guide.md |
| ACH ~33B+ payments / ~$85T+ per year | **Reported** | Nacha annual figures (approximate) |
| CFPB 1033: proposed Oct 2023; final rule Oct 2024; compliance phased from ~2026 | **Verified** | CFPB personal-financial-data-rights page; KPMG/Axway analyses |
| Basel III Endgame: proposed July 2023; recalibrated final rule approved Nov 2025 | **Reported** | Industry reporting; not verified this session |
| Synapse collapse May 2024 | **Reported** | Widely covered; referenced in core_banking_systems_guide.md |
| Community-bank definition raised to <$10B (FDIC, 2020) | **Reported** | FDIC Community Banking Study definition revision |
| 200+ billion lines of COBOL in production globally | **Reported** | Frequently cited industry estimate |
| US core modernization market ~$20B category | **Verified (secondary)** | TechBullion headline figure |
| "Core replacement is the hardest IT project in banking" | **Reported (assessment)** | Umbrella guide thesis (core_banking_systems_guide.md) |
| ACH/Fedwire/CHIPS volumes (Fedwire ~$1Q/yr; CHIPS ~$500T+/yr) | **Reported** | Order-of-magnitude industry figures; see financial_infrastructure_guide.md |

---

## 13. References

Primary/retrievable sources used in this session (via web search):

1. FDIC — Quarterly Banking Profile press releases (Q1 2025: 4,421 institutions; Q3 2025: 4,379; deposits and earnings commentary), via content.govdelivery.com and cranedata.com.
2. NCUA — "NCUA Releases Fourth Quarter 2025 Credit Union System Performance Data" (Mar 6, 2026): 4,287 federally insured credit unions; 739 >$500M.
3. JPMorganChase — 2025 Annual Report and Q4/FY2025 results press release (jpmorganchaseco.gcs-web.com): $4.4T assets, $362B equity, Dec 31 2025; tech-investment coverage ($18B 2025, $17B 2024) via ainvest.com; "Next at Chase" engineering blog (Medium) on mainframe-to-cloud modernization.
4. Kansas City Fed — "Market Structure of Core Banking Services Providers," Payments System Research Briefing (Mar 27, 2024): Big Three concentration.
5. Jack Henry — press release, "Symitar Solidifies Position as the Leading Platform for Credit Unions" (Dec 19, 2024, citing Callahan & Associates; 212 billion-dollar CUs; #1 since 2018).
6. Fiserv — fiserv.com DNA Platform page (real-time, person-centered); TheirStack "Companies that use Fiserv DNA in United States (211)".
7. FIS — Finextra: "Thailand's TISCO Bank signs for FIS core banking system" (Profile); fisglobal.com.
8. Temenos — press releases: "Top US-Based Bank Selects Temenos for Core Banking Modernization in the Cloud" (Apr 2023, Regions + Commerce Bank); "Leading US Regional Bank Selects Temenos SaaS to Modernize Core Banking" (Jul 2026); Tearsheet interview with Regions Bank (core-modernization journey).
9. CFPB — consumerfinance.gov "Personal Financial Data Rights" (final rule Oct 2024); KPMG and Axway analyses of Section 1033.
10. Thought Machine — learnbonds ($83M raise, US expansion); Similarweb (thoughtmachine.net, New York traffic).
11. TechBullion — "Core Banking Modernization in the US: inside the $20 billion category" (legacy platform names; thirty-cores quote).
12. Chase (consumer site) — "more than 14,000 ATMs and 5,000 branches."
13. Repo sibling guides — [core_banking_systems_guide.md](core_banking_systems_guide.md), [chinese_bank_core_systems_guide.md](chinese_bank_core_systems_guide.md), [financial_infrastructure_guide.md](financial_infrastructure_guide.md), [programmable_business_bank_guide.md](programmable_business_bank_guide.md), [temenos_guide.md](temenos_guide.md), [oracle_banking_microservices_architecture_guide.md](oracle_banking_microservices_architecture_guide.md), [data_models_banking_insurance_guide.md](data_models_banking_insurance_guide.md), [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md).

Where sources disagreed or a claim could not be verified, the guide says so inline and in the [claims-status table](#12-claims-status-table).

---

*End of guide. Companion reading: the umbrella [core_banking_systems_guide.md](core_banking_systems_guide.md) for the global discipline; [chinese_bank_core_systems_guide.md](chinese_bank_core_systems_guide.md) for the mirror-image China landscape; [financial_infrastructure_guide.md](financial_infrastructure_guide.md) for the US payment rails; [programmable_business_bank_guide.md](programmable_business_bank_guide.md) for the US neobank/BaaS layer.*
