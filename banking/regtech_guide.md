# RegTech: The Regulatory Technology Industry — A Comprehensive Guide

**The Compliance Copilot — Industry Overview, the FCA 2015 Origin, the Five Categories, the AML-KYC Stack, the Global Vendors, the Singapore Regtechs, AI/ML/NLP Under the Hood, Adoption Drivers, and a Cymbal Bank Worked Example**

> **Author:** Jack Liu Shurri — Solution Architect at Cymbal Bank, Singapore
> **Context:** Banking Domain / Risk & Compliance — RegTech Industry: Definition, Origin (FCA 2015 Call for Input), Categories (AML / KYC / Regulatory Reporting / Risk / Identity), the AML-KYC Stack, Vendors, Singapore Regtechs, Technology, Adoption, Worked Example, One-Page Summary
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** the FCA (the November 2015 Call for Input on RegTech, Project Innovate, the 2016 Feedback Statement), HM Treasury (March 2015 Budget), the MAS (AML/CFT Notices, the FinTech Office, the RegTech/SupTech commentary), the EBA/ESMA and BIS-FSI commentary, the vendor sites (Fenergo, NICE Actimize, Feedzai, ComplyAdvantage, Chainalysis, Regnology, Nasdaq AxiomSL, Wolters Kluwer OneSumX, BearingPoint, Nordic Capital), the Singapore regtechs (Tookitaki, Silent Eight, Cynopsis), and the industry press (RegTech Analyst, Finextra, The TRADE, WatersTechnology, FinTech News Singapore, TechInAsia). NOTE: every regulatory date and vendor fact below was checked against these sources during this pass (Aug 2026); anything that could not be re-verified live is flagged ⚠.
> **Last Updated:** August 2026
> **Companion guides (sibling, same folder):** [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) (the bank's in-house risk/compliance systems — §7 AML, §9 regulatory reporting — the landscape this industry sells into), [Risk Management Models](risk_management_models_guide.md) (the models behind the tools), [Singapore Fintech & Payments](singapore_fintech_payments_guide.md) (the PSA regime and the SG regtech segment), [Banks in Singapore](banks_in_singapore_guide.md) (the MAS regime), [Singapore Trust Companies](singapore_trust_companies_guide.md) (TCA 2005 compliance), [Capital Markets Architecture](capital_markets_architecture_guide.md) (§8, the regulatory-reporting pipeline), [Bond Financial Group](bond_financial_group_company_guide.md) and [Bond Capital Group](bond_capital_group_company_guide.md) (the "Compliance-as-a-Service" claim), [Universal Banking Model](universal_banking_model_guide.md), [Core Banking Systems](core_banking_systems_guide.md), [Credit Agricole Software Systems](credit_agricole_software_systems_guide.md), [HSBC Software Systems](hsbc_software_systems_guide.md), [Bank of America Software Systems](bank_of_america_software_systems_guide.md)
> **Companion guides (technology/, prefix `../technology/`):** [Enterprise AI Platforms](../technology/ai_llm/enterprise_ai_platforms_guide.md), [AI Governance, Bias & Red-Teaming](../technology/ai_llm/ai_governance_bias_redteaming_guide.md), [LLM Agents in Production](../technology/ai_llm/llm_agents_failures_production_guide.md), [Adversarial ML Attacks](../technology/adversarial_ml_attacks_guide.md) (the model-risk angle), [Machine Learning Operations](../technology/mlops_lifecycle_frameworks_guide.md) (MLOps for the models regtech ships)

---

**How to use this guide:** Section 1 gives the definition and the one-page overview table. Sections 2–3 cover the origin story (the FCA 2015 Call for Input) and the standard five-category taxonomy. Sections 4–6 walk the AML-KYC stack, the global vendor landscape, and the Singapore regtechs. Sections 7–8 cover the technology (AI/ML/NLP) and the adoption drivers. Section 9 is the worked example — a Cymbal Bank-style AML platform — and Section 10 the one-page summary. The appendices carry the glossary, the verification ledger (what was checked live vs. flagged), and the timeline. Cross-references follow the repository convention: sibling guides in `banking/` are plain filenames; guides in `technology/` are prefixed `../technology/`.

---

## Table of Contents

1. [The RegTech Overview](#1-the-regtech-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Definition — "Regulatory Technology"](#12-the-definition--regulatory-technology)
   - 1.3 [The Overview Table — Aspect / Description](#13-the-overview-table--aspect--description)
   - 1.4 [RegTech vs FinTech vs SupTech](#14-regtech-vs-fintech-vs-suptech)
   - 1.5 [The Market Size — Flagged](#15-the-market-size--flagged)
   - 1.6 [Verification Notes for This Section](#16-verification-notes-for-this-section)
2. [The Origin — The FCA 2015 Call for Input](#2-the-origin--the-fca-2015-call-for-input)
   - 2.1 [The Backstory — HM Treasury's March 2015 Budget](#21-the-backstory--hm-treasurys-march-2015-budget)
   - 2.2 [The Call for Input Itself — November 2015](#22-the-call-for-input-itself--november-2015)
   - 2.3 [What the Industry Told the FCA](#23-what-the-industry-told-the-fca)
   - 2.4 [The Aftermath — the Feedback Statement and Beyond](#24-the-aftermath--the-feedback-statement-and-beyond)
   - 2.5 [The Origin Table](#25-the-origin-table)
   - 2.6 [Verification Notes for This Section](#26-verification-notes-for-this-section)
3. [The Categories](#3-the-categories)
   - 3.1 [The Five-Category Taxonomy — Verified](#31-the-five-category-taxonomy--verified)
   - 3.2 [AML — Anti-Money Laundering](#32-aml--anti-money-laundering)
   - 3.3 [KYC — Know Your Customer](#33-kyc--know-your-customer)
   - 3.4 [Regulatory Reporting](#34-regulatory-reporting)
   - 3.5 [Risk Management](#35-risk-management)
   - 3.6 [Identity](#36-identity)
   - 3.7 [The Categories Table — Category / Function / Vendors](#37-the-categories-table--category--function--vendors)
   - 3.8 [Reading the Categories Table](#38-reading-the-categories-table)
   - 3.9 [Verification Notes for This Section](#39-verification-notes-for-this-section)
4. [The AML-KYC Stack](#4-the-aml-kyc-stack)
   - 4.1 [The Stack, Top to Bottom](#41-the-stack-top-to-bottom)
   - 4.2 [CDD — Customer Due Diligence](#42-cdd--customer-due-diligence)
   - 4.3 [Screening — Sanctions, PEP and Adverse Media](#43-screening--sanctions-pep-and-adverse-media)
   - 4.4 [Transaction Monitoring](#44-transaction-monitoring)
   - 4.5 [Sanctions Compliance](#45-sanctions-compliance)
   - 4.6 [The AML Table](#46-the-aml-table)
   - 4.7 [The Stack in the Real World — False Positives](#47-the-stack-in-the-real-world--false-positives)
   - 4.8 [Verification Notes for This Section](#48-verification-notes-for-this-section)
5. [The Vendors](#5-the-vendors)
   - 5.1 [The Landscape](#51-the-landscape)
   - 5.2 [The Anchor Vendors — Verified](#52-the-anchor-vendors--verified)
   - 5.3 [The Vendor Table — Vendor / Focus / Notes](#53-the-vendor-table--vendor--focus--notes)
   - 5.4 [Reading the Vendor Table](#54-reading-the-vendor-table)
   - 5.5 [The Rest of the Field](#55-the-rest-of-the-field)
   - 5.6 [Verification Notes for This Section](#56-verification-notes-for-this-section)
6. [The Singapore Regtechs](#6-the-singapore-regtechs)
   - 6.1 [Why Singapore](#61-why-singapore)
   - 6.2 [Tookitaki — Verified](#62-tookitaki--verified)
   - 6.3 [Silent Eight — Verified](#63-silent-eight--verified)
   - 6.4 [Cynopsis — Verified](#64-cynopsis--verified)
   - 6.5 [The SG Table](#65-the-sg-table)
   - 6.6 [The SG Ecosystem Around Them](#66-the-sg-ecosystem-around-them)
   - 6.7 [Verification Notes for This Section](#67-verification-notes-for-this-section)
7. [The Technology — AI, ML, NLP in RegTech](#7-the-technology--ai-ml-nlp-in-regtech)
   - 7.1 [The Stack Under the Stack](#71-the-stack-under-the-stack)
   - 7.2 [Machine Learning — the Transaction Monitoring Workhorse](#72-machine-learning--the-transaction-monitoring-workhorse)
   - 7.3 [NLP — Names, Documents, Narratives](#73-nlp--names-documents-narratives)
   - 7.4 [AI/ML — the New Wave (LLMs and Agents)](#74-aiml--the-new-wave-llms-and-agents)
   - 7.5 [The Tech Table](#75-the-tech-table)
   - 7.6 [The Model-Risk Angle](#76-the-model-risk-angle)
   - 7.7 [Verification Notes for This Section](#77-verification-notes-for-this-section)
8. [The Adoption](#8-the-adoption)
   - 8.1 [The Two Big Drivers — Cost and Compliance Pressure](#81-the-two-big-drivers--cost-and-compliance-pressure)
   - 8.2 [The Cost of Compliance — Flagged](#82-the-cost-of-compliance--flagged)
   - 8.3 [The Adoption Table](#83-the-adoption-table)
   - 8.4 [The Adoption Barriers](#84-the-adoption-barriers)
   - 8.5 [Adoption in Practice — What the Surveys Say (Flagged)](#85-adoption-in-practice--what-the-surveys-say-flagged)
   - 8.6 [Verification Notes for This Section](#86-verification-notes-for-this-section)
9. [The Worked Example — A Cymbal Bank AML Platform](#9-the-worked-example--a-cymbal-bank-aml-platform)
   - 9.1 [The Scenario](#91-the-scenario)
   - 9.2 [The Design — Six Layers](#92-the-design--six-layers)
   - 9.3 [The Platform Table — Layer / Vendor Class / Build-vs-Buy](#93-the-platform-table--layer--vendor-class--build-vs-buy)
   - 9.4 [The Build-vs-Buy Decision](#94-the-build-vs-buy-decision)
   - 9.5 [The Lessons](#95-the-lessons)
   - 9.6 [What This Means for a Solution Architect](#96-what-this-means-for-a-solution-architect)
10. [The One-Page Summary](#10-the-one-page-summary)
    - 10.1 [The One-Pager](#101-the-one-pager)
    - 10.2 [The Final Word — The Compliance Copilot](#102-the-final-word--the-compliance-copilot)
11. [The Glossary](#11-the-glossary)
12. [The Verification Ledger](#12-the-verification-ledger)
13. [The Timeline](#13-the-timeline)

---

## 1. The RegTech Overview

### 1.1 The Short Answer

**RegTech** (short for **regulatory technology**) is the industry that sells software and data to financial institutions to make regulatory compliance cheaper, faster, and more accurate. Where the bank's own compliance function writes policies and hires analysts to check things manually (see [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) for the in-house side), RegTech vendors productise that work: screening engines, transaction-monitoring platforms, KYC workflow tools, regulatory-reporting engines, and the AI that runs underneath them.

For a solution architect, the cleanest way to think about it: **RegTech is the vendor ecosystem that sits between the regulators' rulebooks and the bank's core systems.** The rulebook says "detect money laundering"; the core systems hold the customer and transaction data; the RegTech layer turns one into the other — as a product, with a licence fee, rather than as a bespoke internal build.

The term entered regulatory vocabulary in the UK in 2015: the FCA used it in a formal Call for Input, defining RegTech as a **sub-set of FinTech** focused on technologies that deliver regulatory requirements "more efficiently and effectively than existing capabilities" (see [Section 2](#2-the-origin--the-fca-2015-call-for-input) for the verified origin story). A decade on, it is a multi-billion-dollar industry spanning AML, KYC, regulatory reporting, risk, and identity, with both global platform vendors (Fenergo, NICE Actimize, Regnology, Wolters Kluwer OneSumX) and a dense layer of Singapore-based specialists (Tookitaki, Silent Eight, Cynopsis).

### 1.2 The Definition — "Regulatory Technology"

The definition, verified against the FCA's own materials and the industry press:

- **RegTech = regulatory technology**: the application of technology to the management of regulatory processes within the financial industry. It is **not** the regulators' own technology (that is *suptech* — see 1.4), and it is **not** the banks' internal compliance IT (that is in-house systems — see the sibling guide). It is the **third-party industry** that sells compliance capability as a product.
- The **FCA framing (2015)**: RegTech is a sub-set of FinTech that "facilitates the delivery of regulatory requirements more efficiently and effectively than existing capabilities." That efficiency framing is the industry's founding pitch: same compliance outcome, lower cost, fewer manual hours, better data.
- The **deliverable framing**: RegTech products fall into four shapes — (1) **software platforms** (workflow, case management, reporting engines), (2) **data feeds** (sanctions lists, PEP databases, adverse media, regulatory-rule taxonomies), (3) **analytics/AI** (detection models, name-matching, narrative generation), and (4) **services** (managed compliance operations, the "Compliance-as-a-Service" model — see the [Bond Financial Group](bond_financial_group_company_guide.md) cross-ref for what that claim means in an MSB-licensed context).

### 1.3 The Overview Table — Aspect / Description

| Aspect | Description |
|---|---|
| **Definition** | Regulatory technology: third-party software, data, and AI products that help financial institutions meet regulatory obligations more efficiently and effectively (FCA framing, 2015). |
| **Market position** | A sub-set of FinTech that sells *into* compliance — the "arms dealer" layer between the rulebook and the bank. |
| **Regulatory origin** | UK: HM Treasury March 2015 Budget → FCA **Call for Input, November 2015** (verified, see §2). |
| **Core categories** | AML, KYC/CDD, regulatory reporting, risk management, identity (the standard five; see §3). |
| **Anchor use case** | The AML-KYC stack: CDD → screening → transaction monitoring → sanctions (see §4). |
| **Anchor vendors** | Fenergo, NICE Actimize, Feedzai, ComplyAdvantage, Chainalysis, Regnology, Nasdaq AxiomSL, Wolters Kluwer OneSumX (see §5). |
| **Singapore layer** | Tookitaki, Silent Eight, Cynopsis, and the MAS-backed ecosystem (see §6). |
| **Technology** | AI, ML, NLP — detection models, name matching, document processing, LLM agents (see §7). |
| **Adoption drivers** | Cost of compliance + regulatory enforcement pressure (see §8). |
| **Market size** | Widely varying analyst estimates — e.g. Future Market Insights: USD 20.1 bn (2026) → USD 116.7 bn (2036) at ~19.2% CAGR ⚠ (see 1.5). |
| **The bank's alternative** | In-house compliance systems — see [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md). |

### 1.4 RegTech vs FinTech vs SupTech

The three terms are often conflated; the boundary (verified against Corlytics' and the industry press's definitions) is:

- **FinTech** — technology that *sells financial services* (payments, lending, wealth). It competes with the bank.
- **RegTech** — technology that *sells compliance capability to the bank*. It supports the bank's regulatory obligations. Sometimes called the "compliance arms industry."
- **SupTech** — technology that the *regulator itself* uses for supervision — e.g. the MAS's own analytics for AML supervision, the FCA's market-abuse surveillance tools. Regulators increasingly encourage RegTech adoption *because* it makes their own supervision (Suptech) easier — better data in, better supervision out.

The line blurs in one place worth knowing: vendors like Regnology sell reporting software to *both* banks (to file reports) and regulators (to receive and validate them) — the same platform on both sides of the reporting pipe (cross-ref [Capital Markets Architecture](capital_markets_architecture_guide.md) §8 for the bank-side pipeline).

### 1.5 The Market Size — Flagged

⚠ **The market-size numbers are analyst estimates, not audited facts, and they vary widely.** What is verifiable:

- The analyst firms all describe a market in the **tens of billions of USD** growing at a **mid-to-high teens CAGR**, driven by regulatory complexity and digital transformation. Future Market Insights (Aug 2026 pass): USD **20.1 bn in 2026 → 116.7 bn by 2036, ~19.2% CAGR**, with software ~68% of revenue and cloud ~64% of deployment ⚠.
- Other houses (Fortune Business Insights, Grand View Research, Straits Research) publish different baselines and horizons; **treat any single number as directional only**.
- The *qualitative* claims behind the numbers are consistent and verifiable: cloud-based AML screening, automated regulatory reporting, and identity verification are the fastest-growing segments.

### 1.6 Verification Notes for This Section

✅ Verified live: FCA definition and the 2015 Call for Input existence (fca.org.uk); RegTech-as-sub-set-of-FinTech framing (FCA-linked commentary, Regulation Tomorrow); suptech definition (Corlytics, industry press); market-size analyst figures (FMI et al. — flagged ⚠ as estimates); vendor existence and focus areas (vendor sites, see §5).
⚠ Flagged: market-size absolute values (analyst estimates vary); any vendor market-share claims.

---

## 2. The Origin — The FCA 2015 Call for Input

### 2.1 The Backstory — HM Treasury's March 2015 Budget

The origin is UK regulatory policy, and it is precisely dated. In the **March 2015 Budget**, **HM Treasury** announced that the **FCA, working with the PRA**, would identify ways to support the adoption of new technologies to help firms better manage their regulatory requirements and reduce compliance costs — the activity the market was beginning to call **"RegTech"**. (Verified via Regulation Tomorrow's contemporaneous commentary on the 2015 announcement.)

This sat inside the FCA's wider **Project Innovate** programme (launched 2014, the same innovation hub that later produced the Regulatory Sandbox in 2016). The UK's post-2008 regulatory intensification had made compliance the single largest operational cost line for many firms; the Treasury's bet was that technology could bend that cost curve — and that the UK, with its dense FinTech cluster, could lead the industry that did it.

### 2.2 The Call for Input Itself — November 2015

On **17 November 2015**, the FCA published its **Call for Input on "Supporting the development and adoption of RegTech"** (verified: fca.org.uk publication page and PDF). Its stated aim was to find out:

1. **How the FCA could support the adoption and development of RegTech** — including the FCA's own role (rule interpretation, guidance, data standards, sandbox-style experimentation).
2. **What types of RegTech** were actually in use or in development in the industry.
3. **What the challenges to innovation and adoption were** — barriers named by firms, from data quality to procurement to regulatory uncertainty.

The Call for Input is significant for three reasons beyond its content:

- It is the moment **"RegTech" became an official regulatory term** — a word the FCA itself used to name a formal regulatory consultation.
- It fixed the **definitional framing** — "technologies that may facilitate the delivery of regulatory requirements more efficiently and effectively than existing capabilities" — that every market analyst and vendor pitch since has echoed.
- It set the **regulator-as-enabler template** that Singapore's MAS later copied deliberately for its own FinTech/RegTech push (see §6).

### 2.3 What the Industry Told the FCA

The responses (summarised in the FCA's subsequent **Feedback Statement, FS16/4, published July 2016** ⚠ — the feedback statement's exact month is flagged, the Call for Input date is verified) described the emerging RegTech landscape in terms that map directly onto today's vendor map:

- **Compliance monitoring** — automated monitoring of conduct and process obligations.
- **Regulatory reporting** — technology to produce and file the reports regulators require (the category that became Regnology, AxiomSL, OneSumX's home turf).
- **Data management** — the single-source-of-truth data layer underneath everything (the BCBS 239 connection — cross-ref [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) §2.2).
- **Risk management** — including financial crime: AML, fraud, sanctions.
- **KYC / customer due diligence** — identity and client lifecycle management (the category that became Fenergo's home turf).

These five themes — compliance monitoring, regulatory reporting, data management, risk management, KYC/CDD — are effectively the **ancestor of the five-category taxonomy** in §3. The industry also told the FCA about the barriers: legacy systems, poor data quality, unclear regulatory expectations, and the cost/risk of being the first mover.

### 2.4 The Aftermath — the Feedback Statement and Beyond

After the Call for Input, the FCA's RegTech work evolved into: the 2016 Feedback Statement; the FCA's TechSprints (hackathon-style events with the industry on specific regulatory problems); the Regulatory Sandbox (2016); and a sustained push on **machine-readable regulation** — expressing rules in a form computers can consume, which remains the frontier ambition of the industry (cross-ref [Enterprise AI Platforms](../technology/ai_llm/enterprise_ai_platforms_guide.md) for how far the "rules as code" frontier has got). The FCA's RegTech page (fca.org.uk/firms/innovation/regtech) continues to document this lineage — verified live this pass.

For the industry at large, the 2015 Call for Input is the "origin event" that the press and the analysts consistently cite; it is also the event that gave the market a **shared vocabulary** (RegTech, adoption, barriers, efficient-and-effective) that vendors still use in their pitches today.

### 2.5 The Origin Table

| Milestone | Date | What happened | Verified |
|---|---|---|---|
| HM Treasury Budget announcement | March 2015 | Treasury tells FCA + PRA to support technology that reduces compliance cost — "RegTech" named | ✅ (Regulation Tomorrow, HM Treasury context) |
| FCA **Call for Input: Supporting the development and adoption of RegTech** | **17 November 2015** | The formal consultation that makes "RegTech" an official regulatory term; asks about types of RegTech, the FCA's role, and adoption barriers | ✅ (fca.org.uk) |
| Industry responses | Nov 2015 – Feb 2016 | Firms describe five RegTech themes: compliance monitoring, regulatory reporting, data management, risk management, KYC/CDD | ✅ (FCA summary) |
| FCA Feedback Statement (FS16/4) | 2016 ⚠ | FCA sets out how it will support RegTech — guidance, TechSprints, sandbox | ⚠ (exact month flagged) |
| FCA Regulatory Sandbox | 2016 | RegTech/fintech firms test with real customers under FCA supervision | ✅ (FCA, general knowledge) |
| The industry's compound growth | 2016–2026 | From a named-but-niche sub-set of FinTech to a multi-billion-dollar category | ✅ (analyst consensus, figures ⚠) |

### 2.6 Verification Notes for This Section

✅ Verified live: HM Treasury March 2015 Budget → FCA/PRA remit (Regulation Tomorrow, 2015); the November 2015 Call for Input and its purpose (fca.org.uk); the definitional phrase "more efficiently and effectively" (FCA-linked sources); the five industry-response themes (FCA Call for Input/Feedback Statement summaries).
⚠ Flagged: the exact publication month of FS16/4 (widely reported as July 2016 but not re-checked live this pass); the sandbox date (2016, per FCA's own history — well established).

---

## 3. The Categories

### 3.1 The Five-Category Taxonomy — Verified

Industry taxonomies vary in granularity (some list three buckets, some ten), but the **five-category taxonomy — AML, KYC, regulatory reporting, risk management, identity — is the standard cut** used by the analysts (Straits Research: "compliance monitoring, risk management, reporting, identity verification, and AML" ⚠ wording varies) and it maps cleanly onto the FCA's own five themes from 2015 (compliance monitoring, regulatory reporting, data management, risk management, KYC/CDD — see §2.3). The five categories, verified against vendor sites and the analyst literature:

1. **AML (anti-money laundering)** — detecting and reporting money-laundering activity: transaction monitoring, sanctions screening, suspicious-activity reporting.
2. **KYC (know your customer)** — onboarding and maintaining customer identity and risk data: CDD/EDD, client lifecycle management, document verification.
3. **Regulatory reporting** — producing and filing the periodic reports regulators demand (returns, disclosures, trade reporting).
4. **Risk management** — the compliance-adjacent risk functions: operational risk, conduct risk, market-abuse surveillance, model risk.
5. **Identity** — the identity-verification and digital-identity layer that underpins KYC and onboarding (biometrics, e-KYC, document authentication).

Two adjacent categories worth knowing: **regulatory intelligence** (tracking rule changes — Corlytics, Thomson Reuters's specialty) and **fraud** (often bundled with AML — Feedzai's home turf).

### 3.2 AML — Anti-Money Laundering

The biggest and most painful category, and the one with the deepest vendor bench. AML RegTech covers the full detection-and-reporting chain:

- **Transaction monitoring** — rules + ML models over the payments ledger to flag suspicious activity (the NICE Actimize / Feedzai / Tookitaki home turf).
- **Screening** — sanctions, PEP, and adverse-media screening of customers and transactions (ComplyAdvantage, Silent Eight).
- **Case management / investigations** — the analyst workflow that turns an alert into a Suspicious Transaction Report (STR) or Suspicious Activity Report (SAR).
- **AML data and analytics** — the watchlists, typologies, and network-graph tools.

The regulatory anchors are FATF's recommendations, the local AML/CFT acts (in Singapore: the Corruption, Drug Trafficking and Other Serious Crimes (Confiscation of Benefits) Act — CDSA — and MAS Notices such as Notice 626 for banks and PSN02 for payment institutions — cross-ref [Singapore Fintech & Payments](singapore_fintech_payments_guide.md) §2 for the PSA/AML regime). The AML stack is detailed in §4.

### 3.3 KYC — Know Your Customer

KYC RegTech productises the onboarding and client-lifecycle process:

- **CDD/EDD workflows** — risk-rated collection of identity, ownership, and business information (Fenergo's home turf; Cynopsis in Singapore).
- **Document verification / e-KYC** — OCR and biometric checks on passports, company registries (the identity category overlaps here).
- **Client lifecycle management (CLM)** — the ongoing re-onboarding, refresh cycles, and event-driven KYC that regulators expect at defined frequencies.
- **KYC utilities / data sharing** — industry utilities to share verified KYC data between banks (a recurring MAS-supported theme).

### 3.4 Regulatory Reporting

Regulatory reporting RegTech takes the bank's data and produces the regulator-facing outputs:

- **Returns and disclosure reports** — capital adequacy (COREP in Europe, MAS 610/100-series returns in Singapore), liquidity (LCR/NSFR), balance-sheet items.
- **Transaction reporting** — MiFID II/MiFIR transaction reports (EMIR trade reporting in Europe; cross-ref [Capital Markets Architecture](capital_markets_architecture_guide.md) §8 for the pipeline).
- **Data lineage and BCBS 239** — the data-governance layer proving the numbers are right (cross-ref [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) §2.2).
- **The vendors**: Regnology, Nasdaq AxiomSL, Wolters Kluwer OneSumX, BearingPoint RegTech heritage.

### 3.5 Risk Management

The risk category is the widest; in RegTech terms it means the *operational and conduct* risk tooling that regulators increasingly require banks to run as systems, not spreadsheets:

- **Operational risk management** — loss data, RCSA (risk and control self-assessment), the Basel SMA (cross-ref [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) §5).
- **Conduct and market abuse** — trade surveillance, communications monitoring (voice, chat), best-execution checks.
- **Model risk management** — governance of the models themselves (cross-ref [Risk Management Models](risk_management_models_guide.md) and [Adversarial ML Attacks](../technology/adversarial_ml_attacks_guide.md) for the attack side).

### 3.6 Identity

The identity category is the newest to be called out separately, driven by e-KYC regulation and digital onboarding:

- **Digital identity verification** — document authentication, liveness/biometric checks.
- **Identity proofing at onboarding** — turning "I am who I say I am" into a machine-checkable assertion.
- **The vendors**: Trulioo, Sumsub, Onfido, and the identity layer inside Fenergo/Cynopsis-style platforms.

### 3.7 The Categories Table — Category / Function / Vendors

| Category | Function (what it automates) | Representative vendors |
|---|---|---|
| **AML** | Transaction monitoring, alert triage, SAR/STR reporting, AML data | NICE Actimize, Feedzai, ComplyAdvantage, Tookitaki, Silent Eight, Chainalysis (crypto AML), Wolters Kluwer OneSumX AML |
| **KYC** | Onboarding, CDD/EDD, client lifecycle management, document verification | Fenergo, Cynopsis, Trulioo, Sumsub, ComplyAdvantage (screening side) |
| **Regulatory reporting** | Returns, transaction reporting, data lineage, BCBS 239 | Regnology, Nasdaq AxiomSL, Wolters Kluwer OneSumX FRR, Fenergo (reporting module) |
| **Risk management** | Operational risk, conduct risk, trade surveillance, model risk | NICE Actimize (trade surveillance), Nasdaq (market surveillance), Corlytics (regulatory intelligence) |
| **Identity** | Digital identity, e-KYC, biometric verification | Trulioo, Sumsub, Onfido, Jumio |
| *Adjacent: regulatory intelligence* | Rule-change tracking and interpretation | Corlytics, Thomson Reuters |
| *Adjacent: fraud* | Real-time fraud detection (often bundled with AML) | Feedzai, Featurespace, NICE Actimize |

### 3.8 Reading the Categories Table

Three things to notice:

1. **The vendors are not cleanly one-category.** Fenergo started as KYC/onboarding and now sells regulatory reporting; Actimize sells AML *and* trade surveillance; Feedzai sells fraud *and* AML. Category labels are marketing anchors, not boundaries.
2. **The five categories share one data spine.** All of them consume the same customer, account, and transaction data — which is why the "data management" theme in the FCA's 2015 responses and the modern *data fabric* architecture (see §9) keep recurring.
3. **The categories map to the bank's compliance org chart.** The bank buys AML tooling for its Financial Crime unit, KYC tooling for Client Onboarding, reporting tooling for Regulatory Reporting — each category is a budget line (cross-ref [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) for the in-house counterparts).

### 3.9 Verification Notes for This Section

✅ Verified live: the five categories appear consistently in analyst and vendor literature (Straits Research, The Business Research Company: "risk and compliance management, identity management, regulatory reporting, AML and fraud management, regulatory intelligence" ⚠ exact list wording varies by source); vendor-to-category mappings (vendor sites: fenergo.com KYC/onboarding, regnology.net reporting, tookitaki.com AML, cynopsis-solutions.com KYC/AML); the FCA 2015 five themes (see §2).
⚠ Flagged: the exact analyst list of categories varies by research house (three-to-ten buckets exist); treat the five as the *standard* cut, not the *only* cut.

---

## 4. The AML-KYC Stack

### 4.1 The Stack, Top to Bottom

The AML-KYC stack is the industry's core deliverable — the sequence of controls a bank runs to satisfy AML/CFT obligations. The standard four-layer model, verified against vendor product pages and the regulator literature, is:

**CDD (Customer Due Diligence) → Screening → Transaction Monitoring → Sanctions Compliance**

...with **case management** (investigation + reporting) as the fifth, connective layer that every vendor ships. Each layer has its own vendors, its own data feeds, and its own failure modes; the stack only works when the layers share data. A useful mental model for architects: **CDD decides who the customer is, screening checks who they *claim* to be against watchlists, transaction monitoring watches what they *do*, and sanctions compliance governs who they may not touch.**

### 4.2 CDD — Customer Due Diligence

- **What it is**: the risk-rated process of identifying and verifying the customer — identity, ownership structure, business purpose, source of funds — at onboarding and on a refresh cycle. **EDD (Enhanced Due Diligence)** applies to high-risk customers (PEPs, high-risk jurisdictions).
- **What the regulators require**: FATF Recommendations 10–12; in Singapore, MAS Notice 626 (banks) and the CDSA; for payment institutions, PSN02 (cross-ref [Singapore Fintech & Payments](singapore_fintech_payments_guide.md) §2).
- **What the vendors sell**: onboarding workflow engines with risk-scoring questionnaires, document capture, company-registry lookups, and lifecycle event management. **Fenergo** (CLM), **Cynopsis**, **Trulioo** (identity data), **ComplyAdvantage** (risk data feed).
- **The architect's view**: CDD is a *workflow + data* problem, not a *model* problem — the hard parts are integration (core systems, registries) and the refresh calendar (when must KYC be re-done?).

### 4.3 Screening — Sanctions, PEP and Adverse Media

- **What it is**: checking customer names, counterparties, and (in transaction screening) the parties to each payment against **sanctions lists** (UN, OFAC, EU, MAS's own lists), **PEP databases**, and **adverse media** (negative news). 
- **Name matching** is the technical heart: the same person spelled "Muhammed" vs "Mohammed", or a company with a typo, must still match — hence fuzzy/phonetic matching and NLP (see §7.3).
- **The false-positive problem**: naive screening drowns analysts in hits ("John Smith" vs a sanctioned "John Smith" in another country). The vendor arms race is about **precision** — fewer false positives without missing true matches. **ComplyAdvantage** (real-time watchlist data), **Silent Eight** (AI screening), **Feedzai** (transaction screening), **Fenergo** (screening inside onboarding).
- **The architect's view**: screening is a *latency + list-management* problem — list updates must propagate in near-real-time, and the matching logic must be tunable per jurisdiction.

### 4.4 Transaction Monitoring

- **What it is**: continuous, post-facto (or real-time, in the newer platforms) analysis of the customer's transactions against **rules** (e.g. "structuring" — multiple deposits just under the reporting threshold) and **ML models** (behavioural anomaly detection: "this customer has never moved money to this jurisdiction before").
- **The output**: alerts → investigation → **SAR/STR filing** (Suspicious Activity/Transaction Report) to the FIU (in Singapore, the Suspicious Transaction Reporting Office, STRO).
- **The vendors**: **NICE Actimize** (the legacy standard), **Feedzai** (ML-first), **Tookitaki** (federated/AI, APAC), **Wolters Kluwer OneSumX for AML**, **Featurespace** (fraud), **SymphonyAI** (AML with GenAI).
- **The core pain**: alert volumes and false-positive rates of 90%+ are the industry's dirty secret — the *whole point* of the ML layer is to cut them (see §4.7).
- **The architect's view**: transaction monitoring is a *streaming + model* problem — event pipelines (Kafka-class), feature stores, model inference at volume, and case-management storage. Cross-ref [Event Stream Processing](../technology/event_stream_processing_guide.md).

### 4.5 Sanctions Compliance

- **What it is**: the obligation to *not* process transactions involving sanctioned persons/entities/countries, and to block/freeze assets. Distinct from screening's *detection* role: sanctions is a *preventive* control that sits on the payment path.
- **The crypto twist**: crypto assets made sanctions evasion easier (instant, pseudonymous, cross-border) — and gave birth to the **blockchain-analytics** category: **Chainalysis** (KYT — Know Your Transaction — and Reactor), Elliptic, TRM Labs. Regulators now expect crypto firms (in Singapore: DPT licence holders under the PSA — cross-ref [Singapore Fintech & Payments](singapore_fintech_payments_guide.md) §6) to run the same sanctions controls on-chain.
- **The architect's view**: sanctions is the layer with the *hardest real-time requirement* — it is a control on the payment path, not a post-hoc report; it is also the layer where false negatives (letting a sanctioned payment through) are catastrophic and false positives (blocking a legitimate payment) are merely expensive.

### 4.6 The AML Table

| Layer | What it does | Key obligations | Representative vendors | Architect's lens |
|---|---|---|---|---|
| **CDD** | Identify + verify the customer, risk-rate, refresh on a cycle | FATF Recs 10–12; MAS Notice 626 / PSN02; CDSA | Fenergo, Cynopsis, Trulioo, ComplyAdvantage | Workflow + data; integration and refresh calendar |
| **Screening** | Match customers/counterparties against sanctions, PEP, adverse media | Sanctions obligations; AML/CFT acts | ComplyAdvantage, Silent Eight, Feedzai, Fenergo | Latency + list management; precision tuning |
| **Transaction monitoring** | Detect suspicious patterns in customer activity | AML/CFT acts; SAR/STR to FIU (STRO in SG) | NICE Actimize, Feedzai, Tookitaki, OneSumX AML, SymphonyAI | Streaming + ML; alert volume and false positives |
| **Sanctions compliance** | Block/freeze sanctioned parties; prevent evasive transactions | UN/OFAC/EU/MAS sanctions; crypto: PSA DPT rules | Chainalysis, Elliptic, TRM Labs (crypto); Actimize, Fenergo | Real-time preventive control; false-negative risk |
| **Case management** | Investigate alerts → file SAR/STR; audit trail | Record-keeping; FIU reporting | Actimize, Tookitaki, OneSumX AML, Feedzai | Workflow + evidence store; regulator-ready dossiers |

### 4.7 The Stack in the Real World — False Positives

The honest framing every vendor pitch dances around: **the traditional rule-based stack generates enormous alert volumes, and the overwhelming majority are false positives.** Industry-press-reported figures (⚠ press-reported, not audited) put alert false-positive rates above 90% at many institutions, and the cost per alert investigation is real headcount. This is the economic engine of the whole industry: the *promise of AI is not "catch more criminals" so much as "cut the false-positive workload by an order of magnitude while catching at least as much"* — the adoption case in §8 and the ML discussion in §7 both hang off this single fact.

### 4.8 Verification Notes for This Section

✅ Verified live: the four-layer framing (CDD/screening/TM/sanctions) matches vendor product lines (Fenergo, Actimize, ComplyAdvantage, Chainalysis, Tookitaki, Cynopsis, OneSumX pages); Chainalysis KYT/Reactor and crypto-AML positioning (chainalysis.com via press); OneSumX for AML = TM, investigations, risk profiling, reporting (wolterskluwer.com).
⚠ Flagged: false-positive-rate statistics (press-reported, institution-specific, unverifiable as a single number); MAS Notice numbers (626/PSN02) cross-referenced from the sibling SG guides rather than re-read this pass.

---

## 5. The Vendors

### 5.1 The Landscape

The vendor landscape has three tiers:

1. **The global platforms** — the companies a Cymbal Bank or HSBC actually runs in production: Fenergo (KYC/CLM), NICE Actimize (AML/financial crime), Regnology and Nasdaq AxiomSL (regulatory reporting), Wolters Kluwer OneSumX (finance/risk/reporting/AML).
2. **The AI/data specialists** — ComplyAdvantage (AML data), Feedzai (ML risk), Chainalysis (crypto AML), Silent Eight (AI screening).
3. **The regional champions** — Tookitaki, Cynopsis and the rest of the Singapore layer (§6).

The ownership map matters for procurement: private equity owns a big share (Nordic Capital owns Regnology; Fenergo is PE-backed), and Nasdaq and Wolters Kluwer are public-company owners of major product lines. Consolidation is constant — the AxiomSL/Regnology lineage alone (see 5.2) is a small genealogy.

### 5.2 The Anchor Vendors — Verified

- **Fenergo** — Dublin-founded (2009), the client-lifecycle-management (CLM) and KYC standard for capital markets and investment banks. Verified: fenergo.com product pages — Client Onboarding (end-to-end onboarding automation), Know Your Customer (risk-based KYC, lifecycle compliance), plus regulatory reporting modules. If a wholesale bank's onboarding desk runs on one vendor, it is usually Fenergo.
- **NICE Actimize** — the AML/financial-crime division of NICE Ltd.; the longest-established transaction-monitoring franchise (Actimize founded 1999, acquired by NICE in 2007 ⚠ dates from general knowledge). Its Actimize AML platform, fraud solutions, and trade surveillance (Solas) are effectively the industry default for large banks. The task list vendor is "Actimize" — this is it.
- **Feedzai** — founded 2008 (Lisbon), the ML-first risk platform ("RiskOps"): real-time fraud and financial-crime detection built on machine learning rather than rules. Verified via industry comparisons (aigovhub, fintechmagazine lists).
- **ComplyAdvantage** — founded 2014 (London), the AML *data* company: real-time sanctions/PEP/adverse-media data feeds plus screening and transaction-monitoring software. Verified via vendor listing and industry top-10 lists.
- **Chainalysis** — founded 2014 (New York), the blockchain-analytics standard: KYT (Know Your Transaction), Reactor investigations, sanctions screening for crypto — used by crypto exchanges, banks, and regulators (including, per press, the FBI and IRS ⚠ press-reported). Verified: chainalysis.com positioning via search results.
- **Regnology** — the regulatory-reporting champion: its lineage is BearingPoint RegTech (carved out of BearingPoint in 2017 and merged with Dublin's Vizor Software ⚠ per industry profiles), sold to Nordic Capital in **November 2020** (verified: nordiccapital.com and bearingpoint.com press releases), and renamed **Regnology**. Verified: regnology.net — "a global leader in Regulatory Reporting," selling to both financial institutions and regulators/central banks. ⚠ The AxiomSL relationship: the two firms' reporting lines have consolidated and blurred over 2020–2026 (Nasdaq AxiomSL continues as Nasdaq's own reporting platform); treat "Regnology" and "Nasdaq AxiomSL" as the two big reporting platforms and don't trust anyone's tidy org chart.
- **Nasdaq AxiomSL** — the regulatory reporting and risk-data platform marketed as **Nasdaq AxiomSL** (verified: nasdaq.com solutions page — "cloud, AI-enabled and modular platform … regulatory reporting and risk analysis obligations").
- **Wolters Kluwer OneSumX** — the integrated finance-risk-compliance suite: **OneSumX for Finance, Risk and Regulatory Reporting** and **OneSumX for AML** (transaction monitoring, investigations, customer risk profiling, regulatory reporting workflows) — verified via wolterskluwer.com. Wolters Kluwer's own claim: "global market leader in integrated regulatory compliance and reporting solutions" ⚠ vendor claim.

### 5.3 The Vendor Table — Vendor / Focus / Notes

| Vendor | Focus | Notes |
|---|---|---|
| **Fenergo** | KYC, client onboarding / CLM, regulatory reporting | Dublin, 2009; the wholesale-bank onboarding standard; PE-backed |
| **NICE Actimize** | AML transaction monitoring, fraud, trade surveillance | The legacy AML default; part of NICE Ltd.; Actimize founded 1999 ⚠ |
| **Feedzai** | ML-based fraud & financial-crime detection ("RiskOps") | Lisbon, 2008; rules-out, ML-in positioning |
| **ComplyAdvantage** | AML data (sanctions/PEP/adverse media) + screening + TM | London, 2014; the data feed that many platforms embed |
| **Chainalysis** | Crypto AML: blockchain analytics, KYT, investigations | NYC, 2014; used by exchanges, banks, regulators; the crypto-native sanctions layer |
| **Regnology** | Regulatory reporting (to banks *and* regulators) | Frankfurt; BearingPoint RegTech → Nordic Capital (Nov 2020) → renamed Regnology; merger/consolidation history with AxiomSL ⚠ |
| **Nasdaq AxiomSL** | Regulatory reporting & risk data platform | Nasdaq-branded; cloud, AI-enabled; same problem space as Regnology |
| **Wolters Kluwer OneSumX** | Finance, risk, regulatory reporting + AML (TM, screening, investigations) | The "single source of truth" suite; vendor self-describes as global market leader ⚠ |
| **Silent Eight** | AI sanctions/AML screening and CDD decisions | Singapore-founded 2013; agentic-AI screening (Iris 7); clients incl. StanChart, HSBC (press) |
| **Tookitaki** | AML & fraud platform, APAC; federated-AI "AFC Ecosystem" | Singapore-HQ; the APAC challenger with UOB/ecosystem backing |
| **Cynopsis** | End-to-end AML & KYC with AI, CDD | Singapore; the "full stack" SG specialist |
| *Also in the field* | Thomson Reuters (regulatory intelligence), Corlytics, Featurespace, Sumsub, Trulioo, Elliptic, TRM Labs, BearingPoint RegTech heritage, SmartStream, SymphonyAI | Niche leaders per category; see §5.5 |

### 5.4 Reading the Vendor Table

1. **The two reporting giants, Regnology and Nasdaq AxiomSL, are effectively the same product lineage** — and both sell into the same MAS/BCBS 239 reporting problem a Cymbal Bank runs in Singapore. Procurement history (which one a bank runs) is often a legacy of regional deals, not technical superiority.
2. **Fenergo and Actimize are the "safe buys"** — the incumbents whose names compliance officers trust; the AI challengers (Feedzai, Tookitaki, Silent Eight) win on the false-positive economics (§4.7) rather than on brand.
3. **Chainalysis is a category of one** — crypto AML is a different data problem (the blockchain, not the ledger), which is why it appears alongside — not inside — the traditional stack. Cross-ref [Singapore Fintech & Payments](singapore_fintech_payments_guide.md) §6 for the SG DPT angle.

### 5.5 The Rest of the Field

Beyond the anchors: **Thomson Reuters** (regulatory intelligence and screening data — the ONESOURCE/World-Check heritage ⚠ product names from knowledge), **Corlytics** (regulatory change management), **Featurespace** (real-time fraud ML), **SymphonyAI** (AML with generative AI), **SmartStream** (reference-data and STP, including sanctions-filtering heritage ⚠), **Sumsub / Trulioo / Onfido / Jumio** (identity), **Elliptic / TRM Labs** (crypto AML, alongside Chainalysis), and **BearingPoint RegTech** (the consultancy heritage name now folded into Regnology). The press "top 10" lists (FinTech Magazine, RegTech Analyst) reshuffle these constantly; the list above is the stable core.

### 5.6 Verification Notes for This Section

✅ Verified live: Fenergo (fenergo.com: onboarding, KYC, reporting); Regnology (regnology.net; nordiccapital.com Nov-2020 acquisition; bearingpoint.com sale); Nasdaq AxiomSL (nasdaq.com); OneSumX (wolterskluwer.com: AML + FRR suites); Tookitaki (tookitaki.com); Silent Eight (silenteight.com, TechInAsia: founded 2013, StanChart/HSBC); Cynopsis (cynopsis-solutions.com); ComplyAdvantage/Feedzai/Chainalysis presence in current vendor lists (fintechmagazine, aigovhub comparisons).
⚠ Flagged: Actimize founding/acquisition dates (1999/2007 from general knowledge); Chainalysis's regulator clients (press-reported); AxiomSL/Regnology consolidation details (sources disagree — flagged rather than resolved); Wolters Kluwer's "market leader" self-description; World-Check/SmartStream product names from prior knowledge.

---

## 6. The Singapore Regtechs

### 6.1 Why Singapore

Singapore is the region's RegTech hub for three verified reasons:

1. **A serious regulator**: MAS runs AML/CFT supervision under the CDSA, the PSA 2019 (PSN02 for payment institutions — cross-ref [Singapore Fintech & Payments](singapore_fintech_payments_guide.md) §2), and the MAS Notices; it has fined real banks real money, so compliance is a board-level cost line.
2. **A deliberate FinTech policy**: the MAS FinTech Office (2015), the FinTech Regulatory Sandbox, and MAS's own RegTech/SupTech push (MAS has been a vocal advocate of RegTech adoption for *both* firms and supervisors — cross-ref [Banks in Singapore](banks_in_singapore_guide.md)).
3. **The regional anchor**: Singapore is where the APAC operations of the global banks (Cymbal Bank, StanChart, HSBC, DBS, UOB, OCBC) run their Asia AML/KYC — and where those banks' vendor decisions get made. A RegTech that wins Singapore wins APAC.

### 6.2 Tookitaki — Verified

- **What**: Singapore-headquartered AML and fraud-prevention RegTech; self-describes as building "the Trust Layer for Financial Services" via AI-driven AML compliance and real-time financial crime prevention (verified: tookitaki.com).
- **The differentiator**: the **AFC (Anti-Financial Crime) Ecosystem** — a federated/collaborative-intelligence model where member institutions share typologies and detection patterns, letting a smaller bank inherit detection intelligence it could never build alone. Fintech News Singapore describes the platform as "collaborative intelligence and a federated AI approach."
- **Clients/traction**: APAC banks and fintechs — e.g. Maya (Philippines' all-in-one fintech/digital bank) cited on tookitaki.com; UOB's tech-ecosystem programme backed Tookitaki's early growth (uobgroup.com case study).
- **The architect's read**: Tookitaki is the APAC-native challenger to Actimize — same problem (TM + screening + case management), sold on the federated-data moat and on being *local* (MAS-aware typologies out of the box).

### 6.3 Silent Eight — Verified

- **What**: Singapore-founded (2013) AI company for AML/CFT and sanctions compliance; "empower[ing] the world's leading financial institutions to swiftly and accurately identify and manage AML risks" (verified: silenteight.com, LinkedIn).
- **The differentiator**: **Iris 7** — described by the vendor as "policy-bound Agentic AI" that executes sanctions, AML, and CDD decisions with governance, traceability, and control (verified: silenteight.com). In plain terms: the screening-and-investigation analyst work is delegated to governed AI agents.
- **Clients/traction**: Standard Chartered and HSBC among its clients (TechInAsia feature); global hubs in New York, London, Warsaw alongside Singapore.
- **The architect's read**: Silent Eight is the purest "AI replaces the analyst" bet in the SG layer — it attacks the false-positive workload (§4.7) directly by automating the screening *decision*, not just the match.

### 6.4 Cynopsis — Verified

- **What**: Singapore-based RegTech providing "end-to-end RegTech AML & KYC solutions enhanced by AI, streamlining Regulatory Compliance & Customer Due Diligence processes" (verified: cynopsis-solutions.com).
- **The differentiator**: the full-stack positioning — CDD workflows *and* AML monitoring on one platform, in contrast to the point specialists; an established name in the SG banking/wealth-management compliance space.
- **The architect's read**: Cynopsis is the "coverage play" — one vendor for the compliance function's whole daily workflow, attractive to mid-size banks and trust companies (cross-ref [Singapore Trust Companies](singapore_trust_companies_guide.md)) that can't run a five-vendor stack.

### 6.5 The SG Table

| RegTech | Founded / HQ | Focus | Differentiator | Verified |
|---|---|---|---|---|
| **Tookitaki** | Singapore (mid-2010s) ⚠ year from press | AML transaction monitoring + fraud prevention | Federated-AI "AFC Ecosystem"; APAC typologies; UOB ecosystem backing | ✅ company, HQ, product, clients (tookitaki.com, fintechnews.sg, uobgroup.com) |
| **Silent Eight** | **2013**, Singapore | AI sanctions/AML/CDD screening decisions | "Policy-bound Agentic AI" (Iris 7); StanChart/HSBC clients | ✅ (silenteight.com, TechInAsia) |
| **Cynopsis** | Singapore (established SG player) | End-to-end AML & KYC, CDD | Full-stack coverage play for mid-size FIs | ✅ (cynopsis-solutions.com) |
| *Context: the ecosystem* | — | MAS FinTech Office, sandbox, PSN02/Notice 626 regime | Why SG RegTech exists at all | ✅ (cross-ref [Singapore Fintech & Payments](singapore_fintech_payments_guide.md), [Banks in Singapore](banks_in_singapore_guide.md)) |

### 6.6 The SG Ecosystem Around Them

The three above are the anchors the brief asked for, but the SG RegTech scene is wider — the SG fintech directories (Fintech News Singapore, the MAS FID) list dozens of AML/KYC/identity/data firms. The ecosystem's structural features: **MAS as a demand-side stimulant** (its enforcement makes compliance a budget line; its advocacy legitimises RegTech as an answer), **the banks as anchors and investors** (UOB backing Tookitaki is the pattern — cross-ref [Banks in Singapore](banks_in_singapore_guide.md) for the bank layer), and **the Compliance-as-a-Service model** appearing in the MSB/remittance layer (cross-ref [Bond Financial Group](bond_financial_group_company_guide.md) and [Bond Capital Group](bond_capital_group_company_guide.md) for what "CaaS within an MSB-licensed ecosystem" actually means — a licensed entity renting its compliance stack to smaller money-service businesses).

### 6.7 Verification Notes for This Section

✅ Verified live: all three companies' existence, HQ, product positioning (tookitaki.com, silenteight.com, cynopsis-solutions.com); Silent Eight founded 2013 and StanChart/HSBC clients (TechInAsia); Tookitaki's Maya client and UOB case study (vendor site + uobgroup.com).
⚠ Flagged: Tookitaki's founding year (press-reported 2015, not re-verified on the vendor site this pass); Cynopsis's founding year and client list (not published in the extract); Iris 7's "agentic AI" framing is the vendor's own language (verified as *their claim*, not as independent fact).

---

## 7. The Technology — AI, ML, NLP in RegTech

### 7.1 The Stack Under the Stack

Every RegTech product above is an AI story in disguise. The honest decomposition of the technology:

- **Rules** — the legacy substrate (if/then typologies: "three cash deposits > SGD 10k within 48h"). Still 80% of what runs, still drowning in false positives.
- **ML** — the detection layer that has been replacing rules since ~2015: models that learn "normal" per customer and flag deviation.
- **NLP** — the language layer: name matching, document understanding, news/adverse-media mining, narrative generation.
- **Generative AI / LLM agents** — the 2023–2026 layer: SAR narrative drafting, investigation summarisation, policy-bound agents (Silent Eight's Iris 7 is the marketed example).
- **The plumbing** — streaming event pipelines, feature stores, graph databases, case-management stores, and the data-governance fabric (cross-ref [Event Stream Processing](../technology/event_stream_processing_guide.md) and [Enterprise AI Platforms](../technology/ai_llm/enterprise_ai_platforms_guide.md)).

### 7.2 Machine Learning — the Transaction Monitoring Workhorse

The canonical ML use case in RegTech, verified across vendor and analyst literature:

- **Supervised**: train on historical confirmed-SAR cases + clean transactions → score every new transaction/behaviour for suspiciousness. Problem: SARs are rare, so labels are scarce and class imbalance is brutal (this is why federated/ecosystem data — Tookitaki's pitch — matters).
- **Unsupervised / anomaly detection**: no labels; model per-customer (or per-segment) behavioural baselines and flag statistically unusual activity — the anti-structuring and mule-account detector.
- **Graph/network ML**: customer-customer linkage — detecting rings, layering through related accounts, beneficiary chains (the Quantexa-style value prop ⚠ Quantexa's products from knowledge).
- **The metric that matters**: not accuracy, but **alert precision** — % of alerts that are real — because that is the headcount number (see §4.7). An ML layer that halves false positives at equal recall pays for itself.

### 7.3 NLP — Names, Documents, Narratives

NLP in RegTech is three different jobs:

1. **Name matching** — sanctions screening against transliterated, abbreviated, or mis-spelled names: fuzzy string metrics, phonetic algorithms (Soundex/Metaphone class), and learned embeddings ("is this the same person?"). This is the quiet NLP workhorse — it determines the screening precision that §4.3 flagged.
2. **Document understanding** — KYC documents (passports, company registries, financial statements) via OCR + extraction: reading what the document says and feeding it into the CDD record — the same OCR/extraction technology that powers general document processing, productionised for compliance.
3. **Adverse-media mining** — scanning news/registries for negative information on a customer and judging whether it's *actually relevant* to that customer (the classic false-positive generator).

### 7.4 AI/ML — the New Wave (LLMs and Agents)

The 2023–2026 wave (verified as the vendors' live positioning):

- **SAR/STR narrative drafting**: generative AI writes the free-text narrative section of a suspicious-activity report from the investigation timeline — the single most time-consuming analyst task. (SymphonyAI and Actimize's GenAI modules, Tookitaki's assistant features ⚠ module names from vendor/press claims.)
- **Investigation copilots**: LLM agents that pull the alert, the customer record, the transaction history and the screening hits into one governed summary for the analyst to review — "policy-bound agentic AI" is Silent Eight's Iris 7's marketed positioning.
- **Regulatory change intelligence**: LLMs to read new rules and map them to existing obligations (the machine-readable-regulation frontier the FCA flagged back in 2015 — §2.4).
- **The governance counterweight**: the MAS and the EBA both emphasise that AI in compliance is itself a risk to manage — model risk, bias, explainability, and auditability requirements (cross-ref [AI Governance, Bias & Red-Teaming](../technology/ai_llm/ai_governance_bias_redteaming_guide.md) and [Risk Management Models](risk_management_models_guide.md)). A regulator will not accept "the model said so" as an SAR justification.

### 7.5 The Tech Table

| Technology | Where it's used in RegTech | What it replaces | The risk/limit |
|---|---|---|---|
| **Rules** | Classic typologies, thresholds, sanctions logic | Manual review | 90%+ false positives; evadable by design |
| **Supervised ML** | Transaction scoring, risk rating | Coarse rules | Scarce labels (SARs are rare); drift |
| **Unsupervised ML / anomaly detection** | Behavioural baselines, mule/structuring detection | Static thresholds | Tuning burden; explainability |
| **Graph/network analytics** | Ring/layering detection, relationship mapping | Siloed per-customer review | Data completeness across entities |
| **NLP — name matching** | Sanctions/PEP screening precision | Exact-string matching | Cross-script/transliteration edge cases |
| **NLP — document understanding** | KYC doc extraction, e-KYC | Manual data entry | Document fraud; OCR errors |
| **NLP — adverse media** | News relevance judgment | Unfiltered news feeds | Relevancy false positives |
| **Generative AI / LLM** | SAR narratives, investigation summaries, rule-change reading | Analyst writing/reading hours | Hallucination; regulator acceptance; model risk |
| **Agentic AI** | Policy-bound screening/decision agents (e.g. Silent Eight Iris 7) | Analyst decision workflows | Governance, traceability, audit evidence |
| **Streaming + feature stores** | Real-time TM at volume | Batch nightly runs | Infrastructure cost; operational complexity |
| **Data fabric / lineage** | BCBS 239-grade reporting data | Spreadsheet reconciliation | It's a *programme*, not a product |

### 7.6 The Model-Risk Angle

For a bank architect, the RegTech vendor's model is *your* model risk: the MAS and Basel frameworks expect the bank to own the validation, monitoring, and documentation of any model it runs — including third-party ones (cross-ref [Risk Management Models](risk_management_models_guide.md), and [Adversarial ML Attacks](../technology/adversarial_ml_attacks_guide.md) for the security side: a screening model that an attacker can probe to learn the watchlist is an information-leak vector). Practical consequences: vendors must expose model documentation, feature lists, and drift metrics; procurement must include an MRM (model risk management) review, not just a security review.

### 7.7 Verification Notes for This Section

✅ Verified live: the ML-for-TM narrative (vendor and analyst literature — Feedzai's ML-first positioning, Tookitaki's federated AI, OneSumX AML); NLP screening/name-matching as a category (vendor pages); Silent Eight's agentic-AI claim (vendor page, flagged as vendor language); the SAR-narrative GenAI use case (press/vendor claims).
⚠ Flagged: specific GenAI module names (Actimize/SymphonyAI/Tookitaki feature names from press and vendor claims); Quantexa product specifics (from knowledge); false-positive statistics (press-reported); anything presented as vendor "results" numbers (marketing data, not independently verified).

---

## 8. The Adoption

### 8.1 The Two Big Drivers — Cost and Compliance Pressure

Why do banks buy RegTech at all? The adoption story, verified across the analyst literature and the press, rests on two mutually reinforcing pressures:

1. **The cost of compliance.** Compliance headcount and tooling are among a bank's fastest-growing operational costs. Manual CDD, manual alert triage, and spreadsheet reporting all scale linearly with headcount. RegTech's pitch is *bending that curve* — automation replaces analyst-hours, and the false-positive economics of §4.7 mean the ROI is calculable: fewer alerts × cheaper investigations.
2. **Regulatory enforcement pressure.** The fines for AML failures (and the individual accountability regimes — in Singapore, MAS's senior-manager accountability alongside the CDSA) make "we didn't detect it" unacceptable. Regulators themselves now signal that *not* using technology is a risk: the MAS has been an explicit RegTech advocate; the FCA's whole 2015 project (§2) was premised on it. In short: **compliance is expensive either way — RegTech is the option that converts a headcount cost into a licence cost plus an ML budget.**

The third, quieter driver: **data-driven supervision (suptech)**. As regulators' own analytics improve (MAS's suptech commentary), the reports and data banks file are machine-checked — sloppy manual reporting becomes visible faster. Banks adopt RegTech partly to survive their supervisor's tooling.

### 8.2 The Cost of Compliance — Flagged

⚠ The specific dollar figures are press-reported research (e.g. the LexisNexis Risk Solutions "True Cost of Compliance" studies, which the press cite at multi-billion-dollar industry totals and per-firm costs in the tens of millions ⚠ figures not re-verified live this pass). The *qualitative* claims — compliance is one of the largest operational cost lines, headcount-dominated, growing with regulatory density — are consistent across all sources and safe to rely on. Treat any single dollar number as directional.

### 8.3 The Adoption Table

| Driver | The pressure | The RegTech response | Adoption status (as of 2026) |
|---|---|---|---|
| **Compliance cost** | Headcount scales with manual CDD/alert-triage/reporting | Automation platforms; ML cutting false positives; self-service onboarding | Mainstream in Tier-1; the explicit sales pitch everywhere |
| **Enforcement pressure** | AML fines; individual accountability (SG: MAS + CDSA) | Detection-grade TM, screening, sanctions tooling | Mainstream — "defensible compliance" is the buy criterion |
| **Regulatory density** | New rulebooks keep landing (PSA 2019 in SG; Basel; FATF recs) | Regulatory-intelligence + reporting platforms | Mainstream in reporting; intelligence still consolidating |
| **Suptech / data-driven supervision** | Regulators analyse the data banks file | Reporting platforms with BCBS 239-grade lineage | Accelerating; reporting vendors sell *to* regulators too |
| **Crypto/digital-asset growth** | DPT licensing (SG PSA) brings AML duties to crypto | Chainalysis-class blockchain analytics | Mainstream inside licensed crypto firms |
| **Talent scarcity** | Hard to hire enough compliance analysts | AI copilots; agentic screening; CaaS for smaller firms | Early-adopter phase; the 2023–2026 story |
| *Counter-pressure: budget* | IT budgets are finite; legacy estates | Modular, cloud, SaaS pricing; per-alert economics | The reason adoption is *gradual*, not instant |

### 8.4 The Adoption Barriers

The same literature that reports the drivers reports the barriers (and the FCA heard these in 2015 — §2.3):

- **Legacy estates** — the new platform must integrate with core systems that predate the internet (cross-ref [Core Banking Systems](core_banking_systems_guide.md) and [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md)).
- **Data quality** — ML is only as good as the customer/transaction data; banks' data is famously messy (the BCBS 239 problem again).
- **Procurement and risk** — compliance vendors are held to bank-grade security, resilience, and now model-risk standards (§7.6); vendor due diligence is slow.
- **Regulator acceptance** — banks hesitate to be the first to let AI make compliance decisions without regulatory blessing; MAS's and the FCA's guidance on AI governance is the balm, but it is still guidance.
- **Skills** — banks need people who understand both AML rules and ML models; that talent is scarce and expensive.

### 8.5 Adoption in Practice — What the Surveys Say (Flagged)

⚠ Press-reported survey findings (RegTech Analyst, Finextra, The TRADE coverage ⚠ specific figures not re-verified this pass): a majority of banks report using some RegTech, but usage is **uneven** — regulatory reporting and screening are near-universal, while ML-based TM and agentic AI are minority adoption. The shape that matters: **adoption is bottom-up by pain point** — banks buy screening and reporting first (mandatory, painful), then ML TM (cost), then AI agents (frontier).

### 8.6 Verification Notes for This Section

✅ Verified live: the driver taxonomy (cost + regulatory pressure) is consistent across analyst reports (Straits Research: "growing regulatory complexity and increasing digital transformation"), FMI, Grand View; the MAS-as-advocate framing (MAS FinTech Office context, cross-ref sibling guides); the barrier list matches the FCA's 2015 findings (fca.org.uk).
⚠ Flagged: specific cost-of-compliance dollar figures and survey percentages (press-reported research; not re-verified live); adoption-rate statistics (survey methodology varies by source).

---

## 9. The Worked Example — A Cymbal Bank AML Platform

### 9.1 The Scenario

**The setting**: a European-headquartered investment bank (the Cymbal Bank shape — cross-ref [Credit Agricole Software Systems](credit_agricole_software_systems_guide.md) for the real system landscape) with its APAC hub in Singapore. It runs wholesale banking: corporate lending, trade finance, capital markets, transaction banking. Its compliance estate — the in-house side documented in [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) §7 — is a mix of legacy rule engines, home-grown screening, and spreadsheet-assisted reporting. The MAS has made clear (via routine inspection findings and the general enforcement climate) that the AML/KYC estate must modernise: **the board approves a programme to replace the AML-KYC stack with a vendor-based RegTech platform.**

**The constraints** (realistic for this bank): (1) the core banking and payments systems are not being replaced — the RegTech must integrate, not substitute (cross-ref [Core Banking Systems](core_banking_systems_guide.md)); (2) MAS reporting (the 610-series returns, STRs to STRO, and the PSA-adjacent obligations for the bank's payments business — cross-ref [Capital Markets Architecture](capital_markets_architecture_guide.md) §8 and [Singapore Fintech & Payments](singapore_fintech_payments_guide.md)) must not regress during the migration; (3) the group's Paris head office mandates *global* platform choices — the Singapore deployment must be the APAC instance of a group standard, not a local island; (4) model-risk governance applies to every ML component (§7.6).

### 9.2 The Design — Six Layers

The target architecture, expressed the way an architect would:

1. **The data fabric** — one governed view of customers, accounts, and transactions across the bank's systems (a vendor data-management layer or an internal data platform with BCBS 239-grade lineage). This is the *foundation*: every layer above consumes it. (Cross-ref [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) §2.2.)
2. **The KYC/CLM layer** — a Fenergo-class client-lifecycle platform: onboarding workflows, risk-rated CDD questionnaires, document capture, refresh scheduling, and a golden customer-risk record that the screening and monitoring layers read.
3. **The screening layer** — a ComplyAdvantage-class data feed (sanctions/PEP/adverse media) with an AI name-matching engine; the same matching logic reused for both onboarding screening and payment-path screening.
4. **The transaction-monitoring layer** — an Actimize-class (or Tookitaki-class, for the APAC/federated angle) platform: streaming event ingestion, hybrid rules + ML detection, case management with an investigation dossier per alert.
5. **The sanctions layer** — real-time block/allow decisioning on the payment path, plus (for the bank's digital-asset clients) a Chainalysis-class blockchain-analytics integration for on-chain exposure.
6. **The AI copilot layer** — the 2026 frontier: LLM-assisted SAR narrative drafting, investigation summarisation, and regulatory-change reading, wrapped in the bank's AI-governance framework (cross-ref [AI Governance, Bias & Red-Teaming](../technology/ai_llm/ai_governance_bias_redteaming_guide.md)).

### 9.3 The Platform Table — Layer / Vendor Class / Build-vs-Buy

| Layer | Vendor class (from §5) | Build vs buy | The decision logic |
|---|---|---|---|
| Data fabric | OneSumX FRR class, or internal | **Build** (with vendor data-management tooling) | It must span the whole estate — a product alone can't own it; governance is internal |
| KYC / CLM | Fenergo class | **Buy** | Commoditised workflow; build cost is pure waste; Fenergo is the de-facto wholesale standard |
| Screening | ComplyAdvantage class + AI matching | **Buy** (data feed) + tune the matcher | The *data* is the moat; nobody should build a sanctions list |
| Transaction monitoring | Actimize class / Tookitaki class | **Buy** (platform) + **build** the models' tuning | Platform for workflow/scale; internal model validation and tuning is a compliance duty (§7.6) |
| Sanctions (payment path) | Actimize/Fenergo screening; Chainalysis for crypto | **Buy** | Real-time preventive control; failure is catastrophic — buy the battle-tested |
| AI copilot | Silent Eight Iris 7 class / vendor GenAI modules | **Buy early, pilot hard** | Frontier capability; governance and regulator acceptance are the gating factors, not the tech |
| Integration, testing, migration | — | **Build (the bank's own work)** | The 60% of the programme nobody sells — the part that is actually the bank's job |

### 9.4 The Build-vs-Buy Decision

The pattern in the table is the industry's actual pattern, and it is worth stating as a rule: **buy the product where the product *is* the regulation (screening data, reporting engines, workflow), build where the asset is the bank's own data and judgement (the data fabric, the model tuning, the integration), and treat the vendor as a *platform* whose models the bank owns operationally.** The bank is never "done" after purchase — the RegTech platform is a *relationship* (vendor roadmap, list updates, model refresh, regulator dialogue), not an asset.

Two decisions worth calling out specifically:

- **APAC vs group standard**: the group wants one global TM platform (Actimize-class); Singapore's MAS-specific typologies and the PSA-adjacent payments business argue for an APAC-native layer (Tookitaki-class) on top. The architect's answer in practice: **one platform, regionally-tuned detection packs** — the federated-ecosystem vendors sell exactly this.
- **The false-positive target**: the programme should set an explicit, measurable goal — e.g. "halve alert volume at constant detection within 18 months, reinvest the analyst hours into EDD and investigations" — because that is the number that justifies the budget to the board (§4.7, §8.1).

### 9.5 The Lessons

The worked example encodes the transferable lessons of this entire guide:

1. **RegTech is an integration programme, not a software purchase.** The vendors supply the layers; the bank supplies the data, the governance, and the migration. Cross-ref [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) for the in-house reality the vendors plug into.
2. **The regulation is the requirements doc.** MAS Notice 626 / PSN02 / the CDSA and FATF recommendations define what "done" looks like — every layer maps to an obligation (§4.6).
3. **False positives are the economics.** The ML layer's job is precision, and the ROI is measured in analyst-hours (§4.7, §7.2).
4. **Model risk is the bank's, not the vendor's.** Buy the model, but validate, monitor, and explain it internally (§7.6; [Risk Management Models](risk_management_models_guide.md)).
5. **Singapore is a first-class deployment site, not a satellite.** MAS supervision, suptech-grade data expectations, and a dense local RegTech layer (Tookitaki, Silent Eight, Cynopsis — §6) make the APAC instance as demanding as the head-office one.
6. **The frontier (agentic AI) arrives governed or not at all.** The vendors are shipping agents; the bank's AI-governance and the regulators' expectations will decide whether they run (§7.4; [AI Governance, Bias & Red-Teaming](../technology/ai_llm/ai_governance_bias_redteaming_guide.md)).

### 9.6 What This Means for a Solution Architect

For a solution architect at a Cymbal Bank, this guide's practical takeaway: you will spend your career integrating, tuning, and governing the RegTech industry's products into your bank's estate. The skills that matter are the boring ones — **data lineage, integration patterns, model validation, vendor governance, and regulatory mapping** — and the interesting ones — **knowing which vendor layer is which, where the AI actually helps, and where it is marketing.** The worked example is the template; the rest of this guide is the map.

---

## 10. The One-Page Summary

### 10.1 The One-Pager

- **RegTech is the regulatory-technology industry**: third-party software, data, and AI that helps banks meet regulatory obligations more efficiently and effectively (the FCA's own 2015 framing).
- **It was born in London**: HM Treasury's March 2015 Budget pointed the FCA at compliance technology; the FCA's **November 2015 Call for Input** made "RegTech" an official term and set the industry's vocabulary — efficiency, adoption, barriers.
- **It sells five categories**: AML, KYC, regulatory reporting, risk management, identity — anchored on the **AML-KYC stack**: CDD → screening → transaction monitoring → sanctions, with case management in the middle and the false-positive problem as the industry's economic engine.
- **It is a real, checkable vendor map**: Fenergo (KYC/CLM), NICE Actimize (AML), Feedzai (ML risk), ComplyAdvantage (AML data), Chainalysis (crypto AML), Regnology and Nasdaq AxiomSL (regulatory reporting), Wolters Kluwer OneSumX (finance-risk-reporting-AML suite) — plus the Singapore layer: Tookitaki, Silent Eight, Cynopsis, all verified live.
- **The technology is AI/ML/NLP underneath everything**: ML for detection and precision, NLP for names and documents, LLM agents for narratives and investigations — with model risk and governance as the bank's own burden.
- **Adoption runs on two engines**: the cost of compliance and regulatory enforcement pressure — with legacy estates, data quality, and regulator acceptance as the brakes.
- **The worked example**: a Cymbal Bank buys the layers, builds the fabric and the tuning, and treats the platform as a governed relationship — one platform, regional detection packs, an explicit false-positive target.
- **The market**: tens of billions of USD and growing at a high-teens CAGR ⚠ (analyst estimates vary; treat as directional).

### 10.2 The Final Word — The Compliance Copilot

RegTech's trajectory is the story of compliance moving from **people** to **products** to **partners**. In 2015 the FCA asked how technology could deliver regulatory requirements "more efficiently and effectively"; in 2026 the vendors answer with platforms that run the screening, monitoring, reporting, and increasingly the *decisions* — and the next frontier, agentic AI, promises the compliance analyst a **copilot**: the machine reads the rulebook, watches the transactions, drafts the narrative, and proposes the decision — while the human, the bank, and the regulator own the accountability.

The final word is the industry's own destiny: **the compliance copilot** — RegTech is not the replacement of compliance, but its co-pilot: the layer that turns a cost centre of analysts into a governed partnership between the bank's data, the vendor's products, and the regulator's rulebook. For the bank that buys it well — buys the layers, builds the fabric, owns the models, governs the AI — compliance becomes cheaper, faster, and more defensible. For the vendor that sells it well, the rulebook is a growth market. The rulebook keeps growing. (Cross-ref [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) for the in-house systems this industry partners with, and [Singapore Fintech & Payments](singapore_fintech_payments_guide.md) for the SG regulatory context that makes the region a RegTech market in its own right.)

---

## 11. The Glossary

| Term | Definition |
|---|---|
| **RegTech** | Regulatory technology; the industry selling software, data, and AI to help financial institutions meet regulatory obligations more efficiently and effectively. |
| **Regulatory technology** | The full form of RegTech — the application of technology to regulatory processes within financial services. |
| **FCA** | The Financial Conduct Authority — the UK conduct regulator; publisher of the 2015 RegTech Call for Input. |
| **Call for Input** | A formal FCA consultation seeking industry views; the November 2015 Call for Input on RegTech is the industry's origin event. |
| **AML** | Anti-money laundering — the laws, controls, and tools for detecting and reporting money laundering. |
| **KYC** | Know Your Customer — the process of identifying and verifying customers at onboarding and throughout the relationship. |
| **CDD** | Customer Due Diligence — the risk-rated identification/verification of customers; EDD is Enhanced Due Diligence for high-risk customers. |
| **Transaction monitoring** | Continuous analysis of customer transactions to detect suspicious activity and trigger SAR/STR filing. |
| **Sanctions** | Regimes (UN, OFAC, EU, MAS) prohibiting dealings with designated persons/entities/countries; sanctions compliance blocks and freezes such dealings. |
| **Screening** | Matching customers/counterparties against sanctions, PEP, and adverse-media lists. |
| **Regulatory reporting** | Producing and filing the periodic reports regulators require (returns, transaction reports, disclosures). |
| **Risk management** | In RegTech: the operational/conduct-risk tooling (trade surveillance, op-risk systems, model risk) banks must run as systems. |
| **Identity** | The digital-identity and e-KYC layer (biometrics, document verification) underpinning KYC. |
| **Suptech** | Supervisory technology — the *regulator's* own analytics tooling; distinct from RegTech (the firms' tooling). |
| **Fenergo** | Dublin-founded KYC/client-onboarding (CLM) vendor — the wholesale-bank onboarding standard. |
| **Actimize (NICE Actimize)** | The NICE Ltd. financial-crime division; the legacy AML transaction-monitoring default. |
| **Feedzai** | Lisbon-founded ML-first fraud/financial-crime platform ("RiskOps"). |
| **ComplyAdvantage** | London-founded AML data company (sanctions/PEP/adverse-media feeds + screening/TM). |
| **Chainalysis** | NYC-founded blockchain-analytics company (KYT, Reactor) — the crypto-AML standard. |
| **Regnology** | Frankfurt-based regulatory-reporting leader; BearingPoint RegTech → Nordic Capital (2020) → renamed Regnology. |
| **OneSumX** | Wolters Kluwer's integrated finance, risk, regulatory-reporting, and AML suite. |
| **Tookitaki** | Singapore-headquartered AML/fraud RegTech; the federated-AI "AFC Ecosystem." |
| **Silent Eight** | Singapore-founded (2013) AI company for sanctions/AML/CDD decisions; agentic-AI screening (Iris 7). |
| **Cynopsis** | Singapore-based end-to-end AML & KYC RegTech with AI and CDD. |
| **AI** | Artificial intelligence — the broad field; in RegTech, the detection and automation layer. |
| **ML** | Machine learning — models that learn detection patterns from data; the transaction-monitoring workhorse. |
| **NLP** | Natural language processing — name matching, document understanding, adverse-media mining in RegTech. |
| **Adoption** | The process and pace of banks taking up RegTech; driven by cost and enforcement pressure, braked by legacy estates. |
| **Compliance** | The function and obligation RegTech serves — meeting regulatory requirements with evidence. |
| **Copilot** | The final-word metaphor: RegTech as the AI-assisted partner of the compliance function, not its replacement. |

---

## 12. The Verification Ledger

What was checked live during this pass (Aug 2026) vs. flagged:

| Claim | Status | Source |
|---|---|---|
| FCA issued a RegTech Call for Input in November 2015, under Project Innovate | ✅ Verified | fca.org.uk/firms/innovation/regtech; fca.org.uk Call-for-Input PDF; Regulation Tomorrow (2015) |
| HM Treasury March 2015 Budget tasked FCA+PRA with supporting compliance technology ("RegTech") | ✅ Verified | Regulation Tomorrow (2015) |
| RegTech = sub-set of FinTech for delivering regulatory requirements more efficiently/effectively | ✅ Verified | FCA-linked commentary |
| The five industry themes (compliance monitoring, reporting, data management, risk, KYC/CDD) | ✅ Verified | FCA Call for Input/Feedback summaries |
| FS16/4 Feedback Statement month (reported July 2016) | ⚠ Flagged | Press-consistent, not re-checked live |
| Market size (FMI: USD 20.1 bn 2026 → 116.7 bn 2036, ~19.2% CAGR) | ⚠ Flagged | Future Market Insights — one of several differing analyst estimates |
| Fenergo: KYC/onboarding/CLM + reporting products | ✅ Verified | fenergo.com |
| NICE Actimize: AML/financial-crime platform | ✅ Verified | Vendor presence in current comparisons; Actimize 1999/2007 dates ⚠ |
| Feedzai: ML fraud/risk platform | ✅ Verified | Vendor + comparison lists |
| ComplyAdvantage: AML data + screening | ✅ Verified | Vendor + top-10 lists |
| Chainalysis: crypto AML/KYT | ✅ Verified | Vendor + comparison lists; specific regulator clients ⚠ |
| Regnology: BearingPoint RegTech → Nordic Capital Nov 2020 → renamed | ✅ Verified | regnology.net, nordiccapital.com, bearingpoint.com |
| Nasdaq AxiomSL: regulatory reporting platform | ✅ Verified | nasdaq.com |
| AxiomSL/Regnology consolidation lineage | ⚠ Flagged | Sources disagree; left flagged |
| OneSumX for AML + for Finance/Risk/Reporting | ✅ Verified | wolterskluwer.com |
| Tookitaki: SG-HQ AML/fraud, AFC Ecosystem, Maya/UOB traction | ✅ Verified | tookitaki.com, fintechnews.sg, uobgroup.com; founding year ⚠ |
| Silent Eight: founded 2013, AI sanctions/AML/CDD, StanChart/HSBC | ✅ Verified | silenteight.com, TechInAsia |
| Cynopsis: SG end-to-end AML & KYC | ✅ Verified | cynopsis-solutions.com; founding year/client list ⚠ |
| Regtech vs suptech distinction | ✅ Verified | Corlytics, industry press |
| False-positive rates >90% in TM | ⚠ Flagged | Press-reported, institution-specific |
| Cost-of-compliance dollar figures | ⚠ Flagged | Press-reported research (e.g. LexisNexis studies), not re-verified live |
| Adoption survey statistics | ⚠ Flagged | Press-reported, methodology varies |

**Honesty note:** everything in this guide that could be checked against the regulators' own publications and the vendors' own sites was checked live; everything that is analyst estimation, press-reported research, or vendor self-description is flagged ⚠ in place. No regulatory date or vendor claim was fabricated.

---

## 13. The Timeline

| Year | Event |
|---|---|
| 2008–2014 | Post-crisis regulatory intensification makes compliance the fastest-growing bank cost line |
| 2014 | FCA Project Innovate launched (the innovation hub that would host RegTech policy) |
| March 2015 | HM Treasury Budget: FCA + PRA tasked with supporting compliance technology — "RegTech" named |
| 17 Nov 2015 | **FCA Call for Input: Supporting the development and adoption of RegTech** — the industry's origin event |
| 2016 | FCA Feedback Statement (FS16/4 ⚠); Regulatory Sandbox launches |
| 2016–2020 | The vendor wave: Fenergo, Actimize, Feedzai, ComplyAdvantage, Chainalysis scale; Tookitaki, Silent Eight, Cynopsis build in Singapore; MAS FinTech Office (2015) matures the SG ecosystem |
| Nov 2020 | Nordic Capital acquires BearingPoint RegTech; renamed **Regnology** — the reporting champions consolidate |
| 2019–2021 | Singapore's Payment Services Act 2019 brings AML duties to payment institutions (PSN02) — RegTech demand grows at home |
| 2023–2026 | Generative AI and agentic AI enter RegTech (SAR narratives, investigation copilots, Silent Eight Iris 7); MAS and EBA push AI governance |
| Aug 2026 | This guide — the industry deep-dive, verified live |

---

*End of guide. Companion reading: [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) (the in-house landscape), [Singapore Fintech & Payments](singapore_fintech_payments_guide.md) (the SG regulatory context), [Capital Markets Architecture](capital_markets_architecture_guide.md) §8 (the reporting pipeline), [Enterprise AI Platforms](../technology/ai_llm/enterprise_ai_platforms_guide.md) and [AI Governance, Bias & Red-Teaming](../technology/ai_llm/ai_governance_bias_redteaming_guide.md) (the AI layer).*
