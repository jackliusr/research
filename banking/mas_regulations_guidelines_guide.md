# MAS Regulations, Guidelines and Industry Expectations — A Comprehensive Guide

**The Rules of the House — the MAS Mandate (Triple Role), the 18 Acts, the Notices, the Guidelines (TRMG, BCM, Outsourcing, IAC, FEAT), the Industry Expectations (Culture, Resilience, Climate, Stablecoins, DPTs), the Enforcement Record, and the Cymbal Bank Obligations × Controls Map**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Banking Domain / Regulatory — the Monetary Authority of Singapore (MAS): the Framework Map (mandate — the triple role), the Statutes (Banking Act 1970, SFA 2001, PS Act 2019, FSMA 2022, FAA 2001, Insurance Act 1966, and the rest of the 18), the Notices (banking 600-series, merchant-bank 1000-series, payments PSN-series, capital-markets CMG/SFA-series, insurance 100–300-series), the Guidelines (TRMG 2013/2021, BCM 2022, Outsourcing 2023, IAC 2020, FEAT 2018, Model AI Governance → Verifiable AI), the Industry Expectations (culture/conduct, operational resilience, climate risk, stablecoin framework, DPT measures), the Enforcement (the enforcement-API results, prohibition orders, the penalties framework), the Banking Context (Cymbal Bank), the Worked Example (the Cymbal Bank obligations × controls matrix), the Summary, the Glossary, the Verification Ledger
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** this pass was written directly from the prior research pass's on-disk extracts of mas.gov.sg (the MAS About page, the "What We Do" page, the Acts catalogue — 18 acts, the Banking Act 1970 page, the MAS Act 1970 page, the PS Act 2019 page, the FSMA 2022 page, the Supervisory Approach and Regulatory Instruments page, the Regulations and Guidance index pages, the Enforcement Actions page, the TRM Guidelines page, the BCM Guidelines page, the Outsourcing (Banks) Guidelines page, the FSM-N30/N31 notices, the Circular on FI Incident Reporting, the Payment Services Act page) plus the on-disk MAS catalogue JSON and the prior subagent's API/enforcement scripts. Facts verified in those files are marked ✅; facts that could not be confirmed from the /tmp research are flagged ⚠ and cross-referenced to the repository guides that carry them. Budget note: two web searches were attempted for one gap (Notice 730) and returned empty — the gap remains flagged ⚠.
> **Last Updated:** August 2026
> **Companion guides (sibling, same folder):** [RegTech](regtech_guide.md) · [Banks in Singapore](banks_in_singapore_guide.md) · [AI/GenAI Banking Compliance](ai_genai_banking_compliance_guide.md) · [Tokenized Assets](tokenized_assets_guide.md) · [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) · [Online Investment Trading Platforms](online_investment_trading_platforms_guide.md) (the SFA) · [Payment Rails](payment_rails_guide.md) (the PS Act)
> **Companion guides (technology/, prefix `../technology/`):** [SecOps](../technology/secops_guide.md) · [Cybersecurity](../technology/cybersecurity_guide.md) (the TRMG angle) · [AI Trust Assessments](../technology/ai_trust_assessments_guide.md) (the FEAT / Model-AI-Governance angle)

---

## Table of Contents

1. [The Overview](#1-the-overview)
   - 1.1 [The MAS Framework Map — the Mandate (the Triple Role)](#11-the-mas-framework-map--the-mandate-the-triple-role)
   - 1.2 [The Overview Table — Aspect / Description](#12-the-overview-table--aspect--description)
   - 1.3 [The Regulatory Instruments Pyramid](#13-the-regulatory-instruments-pyramid)
   - 1.4 [The MAS Website Map — Where the Rules Live](#14-the-mas-website-map--where-the-rules-live)
2. [The Statutes (the Acts)](#2-the-statutes-the-acts)
   - 2.1 [The 18 Acts Administered](#21-the-18-acts-administered)
   - 2.2 [The Banking Act 1970](#22-the-banking-act-1970)
   - 2.3 [The Securities and Futures Act 2001](#23-the-securities-and-futures-act-2001)
   - 2.4 [The Payment Services Act 2019](#24-the-payment-services-act-2019)
   - 2.5 [The Financial Services and Markets Act 2022](#25-the-financial-services-and-markets-act-2022)
   - 2.6 [The Financial Advisers Act 2001 (the "FTA" reading)](#26-the-financial-advisers-act-2001-the-fta-reading)
   - 2.7 [The Insurance Act 1966](#27-the-insurance-act-1966)
   - 2.8 [The Rest of the 18 — the Supporting Acts](#28-the-rest-of-the-18--the-supporting-acts)
   - 2.9 [The Statute Years — Verified vs Flagged](#29-the-statute-years--verified-vs-flagged)
3. [The Notices](#3-the-notices)
   - 3.1 [Notices as a Legal Instrument](#31-notices-as-a-legal-instrument)
   - 3.2 [The Banking 600-Series](#32-the-banking-600-series)
   - 3.3 [The Merchant-Bank Notices and the 730 Question](#33-the-merchant-bank-notices-and-the-730-question)
   - 3.4 [The Payments Notices (the PSN-Series and the FSM-N30/N31)](#34-the-payments-notices-the-psn-series-and-the-fsm-n30n31)
   - 3.5 [The Capital-Markets Notices and the "CM 800-Series" Question](#35-the-capital-markets-notices-and-the-cm-800-series-question)
   - 3.6 [The Insurance Notices and the "1000-Series" Question](#36-the-insurance-notices-and-the-1000-series-question)
   - 3.7 [The Notice Map — Series / Family / Applies-to](#37-the-notice-map--series--family--applies-to)
4. [The Guidelines](#4-the-guidelines)
   - 4.1 [Guidelines as an Instrument — What "Should" Means](#41-guidelines-as-an-instrument--what-should-means)
   - 4.2 [The Technology Risk Management Guidelines (TRMG) — 2013 / 2021](#42-the-technology-risk-management-guidelines-trmg--2013--2021)
   - 4.3 [The Business Continuity Management (BCM) Guidelines — 2022](#43-the-business-continuity-management-bcm-guidelines--2022)
   - 4.4 [The Outsourcing Guidelines — 2023](#44-the-outsourcing-guidelines--2023)
   - 4.5 [The IAC — Guidelines on Individual Accountability and Conduct — 2020](#45-the-iac--guidelines-on-individual-accountability-and-conduct--2020)
   - 4.6 [FEAT — Fairness, Ethics, Accountability, Transparency — 2018](#46-feat--fairness-ethics-accountability-transparency--2018)
   - 4.7 [Model AI Governance → Verifiable AI](#47-model-ai-governance--verifiable-ai)
   - 4.8 [The Other Guidelines That Bind](#48-the-other-guidelines-that-bind)
5. [The Industry Expectations](#5-the-industry-expectations)
   - 5.1 [Culture and Conduct](#51-culture-and-conduct)
   - 5.2 [Operational Resilience](#52-operational-resilience)
   - 5.3 [Climate Risk](#53-climate-risk)
   - 5.4 [The Stablecoin Framework](#54-the-stablecoin-framework)
   - 5.5 [The DPT Measures](#55-the-dpt-measures)
   - 5.6 [Scams and Consumer Protection](#56-scams-and-consumer-protection)
   - 5.7 [The Expectations Table](#57-the-expectations-table)
6. [The Enforcement](#6-the-enforcement)
   - 6.1 [The Enforcement Framework](#61-the-enforcement-framework)
   - 6.2 [The Enforcement-API Results — the Recent Actions](#62-the-enforcement-api-results--the-recent-actions)
   - 6.3 [The 54 Prohibition Orders — Flagged](#63-the-54-prohibition-orders--flagged)
   - 6.4 [The Penalties Framework](#64-the-penalties-framework)
7. [The Banking Context — Cymbal Bank](#7-the-banking-context--cymbal-bank)
   - 7.1 [The Bank and the Mandate](#71-the-bank-and-the-mandate)
   - 7.2 [The Obligations × Controls Map](#72-the-obligations--controls-map)
   - 7.3 [The Supervisory Review Cycle — How the Map Gets Tested](#73-the-supervisory-review-cycle--how-the-map-gets-tested)
8. [The Worked Example — the Cymbal Bank MAS-Obligations × Controls Matrix](#8-the-worked-example--the-cymbal-bank-mas-obligations--controls-matrix)
   - 8.1 [The Scenario](#81-the-scenario)
   - 8.2 [The Matrix](#82-the-matrix)
   - 8.3 [The Lessons](#83-the-lessons)
9. [The Summary — "The Rules of the House"](#9-the-summary--the-rules-of-the-house)
10. [Glossary](#10-glossary)
11. [Claims Status and Verification Notes](#11-claims-status-and-verification-notes)
   - 11.1 [The Research Trail (what the prior pass left on disk)](#111-the-research-trail-what-the-prior-pass-left-on-disk)
12. [Cross-References and Further Reading](#12-cross-references-and-further-reading)

---

## 1. The Overview

### 1.1 The MAS Framework Map — the Mandate (the Triple Role)

The Monetary Authority of Singapore (MAS) is Singapore's **central bank and integrated financial regulator** — and, unusually, a third thing as well: the **developer of the financial centre**. The MAS "What We Do" page (verified ✅ in this pass) states the triple role plainly:

1. **As central bank** — MAS promotes sustainable, non-inflationary economic growth through the conduct of monetary policy and close macroeconomic surveillance and analysis; it manages Singapore's exchange rate (the S$NEER band), the official foreign reserves, and liquidity in the banking sector, issues currency, oversees payment systems, and serves as banker to and financial agent of the Government. Singapore's monetary policy is exchange-rate-centred — MAS manages the Singapore dollar against a basket of currencies rather than targeting an interest rate — which makes the central-bank half of the house different from most of its peers.
2. **As integrated financial supervisor** — MAS fosters a sound financial services sector through prudential oversight of *all* financial institutions in Singapore: banks, merchant banks, finance companies, insurers, capital-market intermediaries, financial advisers, trust companies, payment service providers, and the exchanges and market infrastructures. It is responsible for well-functioning financial markets, sound conduct, and investor education. "Integrated" is the operative word: one regulator covers banking, capital markets, insurance, and payments — there is no split-regulator structure (no separate PRA/FCA, no SEC/CFTC/state split).
3. **As developer of the financial centre** — MAS works with the industry to promote Singapore as a dynamic international financial centre: facilitating infrastructure, technology adoption (FinTech, the Regulatory Sandbox, SGQR/PayNow/FAST), skills upgrading, sustainable finance (the green-finance push, transition credits, the taxonomy), and digital assets (Project Guardian, the stablecoin framework). Development sits inside the regulator's statutory family — a deliberate Singapore design choice that gives MAS a "promote and police" dual character.

**The legal foundation.** The Monetary Authority of Singapore Act 1970 — Parliament passed it in 1970 and MAS formed on **1 January 1971** (✅ verified in the MAS Act page extract). The MAS Act gives MAS the authority to regulate the financial services sector and entrusts it to promote monetary stability and credit and exchange policies conducive to the growth of the economy. MAS is governed by the MAS Act, "which confers on it powers to issue legal instruments for regulation and supervision of financial institutions" (✅ the Banking Regulations and Guidance index page).

**The sectors.** MAS regulates financial institutions in four sectors (✅ the Regulation index page): **Banking** (deposit-taking institutions: full banks, wholesale banks, merchant banks, finance companies, credit/charge-card issuers, banking financial holding companies), **Capital Markets** (fund managers, REIT managers, corporate finance advisers, trustees, dealers, credit rating agencies, financial advisers), **Insurance** (insurers, reinsurers, brokers, agents), and **Payments** (payment service providers — SPI/MPI/money-changers — and payment systems).

**The supervisory stance.** MAS's published supervisory monographs (✅ listed on the Supervisory Approach page): *Tenets of Effective Regulation*; the *Framework for Impact and Risk Assessment of Financial Institutions* (and the *Guide for Senior Executives* version); *Objectives and Principles of Financial Sector Oversight in Singapore* (April 2004); and *MAS' Approach to Macroprudential Policy*. MAS designates **D-SIBs** (Domestic Systemically Important Banks — the 2015 framework, aligned with BCBS; the current list: DBS, OCBC, UOB, Citibank, Malayan Banking Berhad, Standard Chartered, HSBC ✅) and **D-SIIs** (Domestic Systemically Important Insurers — inaugural list 21 September 2023: AIA Singapore, Income Insurance, Prudential Assurance Singapore, Great Eastern Life, Manulife Singapore ✅). All banks are assessed annually for systemic importance on size, interconnectedness, substitutability, and complexity. A Cymbal Bank-sized foreign full bank would live in the lower-impact tier of the FIRA framework — but the *expectations* apply to everyone.

### 1.2 The Overview Table — Aspect / Description

| Aspect | Description |
|---|---|
| **Who** | MAS — Singapore's central bank and integrated financial regulator; statutory creation of the MAS Act 1970, operating since 1 January 1971 ✅ |
| **The triple role** | Central bank (monetary policy, reserves, currency) · integrated financial supervisor (prudential + conduct across banking, capital markets, insurance, payments) · developer of the financial centre (FinTech, sandbox, sustainable finance, digital assets) ✅ |
| **Acts administered** | 18 statutes, from the Banking Act 1970 to the FSMA 2022 ✅ (full list in §2) |
| **Legal instruments** | Acts · Subsidiary Legislation · Directions (Directives + Notices) · Guidelines · Codes · Practice Notes · Circulars · Policy Statements — eight rungs with distinct legal effects ✅ |
| **Supervision style** | Integrated, risk-based, impact-and-risk assessed (FIRA framework), D-SIB/D-SII designation, thematic focus areas: AML, enforcement, scam-combating, cyber security, third-party risk, sandbox, benchmark transition, operational resilience ✅ |
| **The compliance stack for a bank** | Statutes → Notices (binding) → Guidelines (best practice, risk-assessment weight) → Circulars/FAQs (interpretation) ✅ |
| **The "rules of the house"** | A full bank's binding floor is the Banking Act + Notice 626 (AML/CFT) + Notice 649 (liquidity) + Notice 658 (outsourcing) + the FSMA tech-risk notices; the supervisory ceiling is the TRMG, BCM, Outsourcing, IAC and FEAT guidelines — you are measured against the *spirit* as much as the letter (this guide, §9) |

### 1.3 The Regulatory Instruments Pyramid

MAS's own classification of the instruments it issues (✅ the Supervisory Approach and Regulatory Instruments page — this is the authoritative rung-by-rung description):

| # | Instrument | Legal effect | Examples |
|---|---|---|---|
| 1 | **Acts** | Force of law; passed by Parliament; published in the Government Gazette | Banking Act, Financial Advisers Act |
| 2 | **Subsidiary Legislation** | Force of law; may specify contravention as a criminal offence; gazetted | Insurance (Actuaries) Regulations, Banking and Charge Card Regulations, Payment Services Regulations 2019 |
| 3 | **Directions** (two sub-kinds) | Legal effect; MAS may specify whether contravention is a criminal offence | — |
| 3a | **Directives** | Legally binding requirements on an individual FI or specified person. Note: directives to merchant banks are essentially "notices" for classification purposes (historical) | Merchant-bank directives (many now cancelled) |
| 3b | **Notices** | Legally binding requirements on a *class* of FIs or persons | Notice 626, Notice 658, Notice 649, PSN01, FSM-N05 |
| 4 | **Guidelines** | Principles / best-practice standards; contravention is *not* a criminal offence and attracts *no* civil penalty — but how well an institution observes them affects MAS's overall risk assessment of it | TRMG, BCM, Outsourcing, IAC, FEAT |
| 5 | **Codes** | Non-statutory system of rules; breach may attract private reprimand or public censure; effect may be tied to an Act (e.g. via s 321 SFA) | Take-over Code, Code on Collective Investment Schemes, CRA Code of Conduct |
| 6 | **Practice Notes** | Administrative procedure guidance (licensing, reporting, compliance); not offences unless the procedure is also required by law | Lodgement of Documents practice note |
| 7 | **Circulars** | Sent to specified persons / published for information; no legal effect — but they carry supervisory expectations | Circular on FI Incident Reporting (16 Dec 2025), anti-scam circular for e-wallet MPIs (25 Oct 2024) |
| 8 | **Policy Statements** | Broad statements of major MAS policy | Macroprudential policy statements |

**Why the pyramid matters to a bank.** The binding floor (1–3) is what an audit finds; the non-binding ceiling (4–8) is what a *supervisory review* finds. MAS is explicit that guideline observance feeds its risk assessment — so a bank that treats the TRMG or FEAT as optional is, in supervisory terms, not optional at all. Every Cymbal Bank control in §7/§8 maps to at least one rung of this pyramid.

---

### 1.4 The MAS Website Map — Where the Rules Live

The MAS site is the regulation's front door, and the About/Regulation index extracts (✅ mas_about.txt, mas_reg.txt) give the architect the map of where every instrument lives:

| MAS site area | What it holds (✅ verified) |
|---|---|
| **Regulation → Banking / Capital Markets / Insurance / Payments** | Per-sector "Regulations and Guidance" hubs; each act, notice, guideline, circular and FAQ page carries its own applies-to list — the definitive answer to "does this bind Cymbal Bank?" |
| **Regulation → Focus Areas** | AML · Enforcement · Combatting Scams · Cyber Security · Third-Party Risk Management · FinTech Regulatory Sandbox · Interest Rate Benchmarks Transition · Operational Resilience — the supervisory themes behind the instruments |
| **Regulation → Supervisory Approach and Regulatory Instruments** | The instrument pyramid (§1.3), the FIRA framework, D-SIB/D-SII lists, the supervisory monographs |
| **Quick Links** | Financial Institutions Directory (the register of who is licensed to do what) · Financial Institution Representatives Register · **Investor Alert List** (unregulated entities MAS warns the public against) · **Enforcement Actions** (the §6 record) · **Lists of Designated Individuals and Entities** (sanctions/designations) |
| **Regulation → Notices / Guidelines / Circulars / FAQs** | The searchable instrument libraries; every page carries the "Related to this Item" cross-links that trace a family (e.g. the TRM Guidelines page links every cancelled sectoral TRM notice and its FSM replacement ✅) |
| **News → Enforcement Actions** | Media-release versions of enforcement actions (from 1 Apr 2021, only POs ≥ 5 years appear here; shorter POs live on the Enforcement Actions page only ✅) |

**The architect's habit:** before designing any control, read the *applies-to* list on the notice page and the *related items* on the guideline page — they tell you which instruments form a family (TRMG ↔ FSM-N05 ↔ incident-reporting circular) and whether the notice binds your licence class at all.

---

## 2. The Statutes (the Acts)

### 2.1 The 18 Acts Administered

The on-disk MAS catalogue (`mas_catalog.json`, queried with the prior pass's `mas_analyze.py`) lists **18 acts administered by MAS** (✅ — output of the `acts` run):

| # | Act | Year | One-line mandate (catalogue summary) |
|---|---|---|---|
| 1 | Monetary Authority of Singapore Act | 1970 | Establishes MAS and gives it authority to regulate the financial services sector |
| 2 | Banking Act | 1970 | Licensing and regulation of banks, merchant banks and related institutions, incl. credit/charge-card business |
| 3 | Insurance Act | 1966 | Regulation of insurance business — insurers, intermediaries, related institutions |
| 4 | Securities and Futures Act | 2001 | Regulation of activities and institutions in the securities and derivatives industry, incl. leveraged FX trading, financial benchmarks, clearing facilities |
| 5 | Financial Advisers Act | 2001 | Regulation of financial advisers and their representatives and supervisors |
| 6 | Payment Services Act | 2019 | Forward-looking framework for payment systems and payment service providers |
| 7 | Financial Services and Markets Act | 2022 | The omnibus Act for sector-wide regulation of financial services and markets |
| 8 | Financial Holding Companies Act | 2013 | Regulates financial holding companies |
| 9 | Finance Companies Act | 1967 | Licensing and regulation of finance companies |
| 10 | Trust Companies Act | 2005 | Licensing and regulation of trust companies |
| 11 | Terrorism (Suppression of Financing) Act | 2002 | Suppresses terrorism financing; implements the International Convention for the Suppression of the Financing of Terrorism |
| 12 | Deposit Insurance and Policy Owners' Protection Schemes Act | 2011 | Reconstitutes the Deposit Insurance Scheme (limited depositor compensation) |
| 13 | Payment and Settlement Systems (Finality and Netting) Act | 2002 | Protects payment and settlement systems from disruption risks |
| 14 | Business Trusts Act | 2004 | Registration and regulation of business trusts |
| 15 | Commodity Trading Act | 1992 | Regulation of certain commodity trading |
| 16 | Bills of Exchange Act | (Cap. 23) | Bills of exchange, cheques, promissory notes |
| 17 | Exchanges (Demutualisation and Merger) Act | 1999 | Demutualisation/merger of the stock and derivatives exchanges and clearing bodies |
| 18 | (SFA Part VII) Disclosure of Interest in Listed Securities | — | Notifications of interests in listed securities under the SFA |

The **big six for a bank's compliance map** are the Banking Act (the licence), the SFA (capital-markets activities), the PS Act (payments), the FSMA (the omnibus overlay — AML powers, tech-risk notices, prohibition orders), the MAS Act (the originating powers), and the Insurance Act (if the group carries insurance entities). The task brief's "FTA" and "TPA" abbreviations do not appear in the catalogue under those letters — the plausible readings are Financial **Advisers Act** (2001) and **Trust** Companies Act (2005), both covered below and flagged in §11.

### 2.2 The Banking Act 1970

✅ **Verified directly** from the MAS Banking Act page extract: "**Banking Act 1970** — Governs the licensing and regulation of banks, merchant banks and related institutions, including their credit card and charge card business." It applies to full banks (locally incorporated and branches), wholesale banks (both), merchant banks (both), and credit/charge-card issuers. The Banking Act is the *licence statute*: it creates the bank-licence tiers (full bank, wholesale bank, merchant bank via the MAS Act framework), the fit-and-proper regime, capital and liquidity expectations, banking-secrecy obligations (Part IV — s 47 confidentiality), restrictions on non-financial business, the deposit-definition guidance (the *Guidelines on Definition of a "Deposit"*), and the supervisory powers that the FSMA later absorbs in part. Related Banking-Act guidance confirmed on the same page: the *Guidelines on the Application of Banking Regulations to Islamic Banking* (16 May 2024 revision) and the cancelled Asian Currency Unit terms (2021). The banking 600-series notices (§3.2) and the bank guidelines (§4) all hang off this Act — see [Banks in Singapore](banks_in_singapore_guide.md) §5 for the licensing-side treatment.

### 2.3 The Securities and Futures Act 2001

✅ From the acts catalogue: the SFA "governs the regulation of activities and institutions in the securities and derivatives industry, including leveraged foreign exchange trading, of financial benchmarks and of clearing facilities". For Cymbal Bank the SFA bites wherever the bank crosses into capital-markets activity: dealing in capital markets products (CMPS), fund management, corporate finance, custodial services, REIT management, and the market-conduct regime (insider trading — see the civil-penalty action against Kenneth Chua in §6.2 — market manipulation, false trading). The SFA carries the CM notices (SFA 04-N02 AML, SFA 04-N19/N20 cross-border arrangements, SFA 02A/03-N01 FMI standards) and the market-infrastructure licences (exchanges, clearing houses, trade repositories, benchmark administrators). The repo's capital-markets treatment lives in [Online Investment Trading Platforms](online_investment_trading_platforms_guide.md) (the SFA angle) and [Capital Markets Architecture](capital_markets_architecture_guide.md).

### 2.4 The Payment Services Act 2019

✅ Verified from the PS Act page extract: Parliament passed the PS Act on **14 January 2019**; it is "a forward looking and flexible framework for the regulation of payment systems and payment service providers", providing regulatory certainty and consumer safeguards while encouraging innovation. The PS Act replaced the legacy Money-changing and Remittance Businesses Act regime with **activity-based licensing**: Standard Payment Institutions (SPI), Major Payment Institutions (MPI), and money-changing licensees, plus designation powers over payment systems. Regulated activities include account issuance, domestic money transfer, cross-border money transfer, merchant acquisition, e-money issuance, DPT services, and money-changing. The **expanded scope** (the 2021 amendment) commenced on **4 April 2024** (the appointed day ✅), bringing more activities and higher requirements in. The PS Act's notice family is the PSN-series (§3.4); the AML heart for payment institutions is PSN01 (specified payment services) and PSN02 (DPT services); the repo treatment is [Payment Rails](payment_rails_guide.md) (the PS Act) and [Singapore Fintech & Payments](singapore_fintech_payments_guide.md).

### 2.5 The Financial Services and Markets Act 2022

✅ Verified from the FSMA page extract — this is the single most important *recent* structural change. Parliament passed the **FSMA on 5 April 2022**; it is the omnibus Act for sector-wide regulation, implemented in **three phases**:

- **Phase 1 — 28 April 2023:** general powers over financial institutions migrated from the MAS Act (inspection powers, offences, miscellaneous provisions — Parts 2, 10, 11, 12); **AML/CFT provisions (Part 4)**; **Financial Dispute Resolution Schemes (Part 6)**.
- **Phase 2A — 10 May 2024:** new **technology and risk-management provisions (Part 5)** — this is the legal home of the harmonised tech-risk notices (the FSM-series that replaced the sector TRM notices, §3.4/§4.2); control and resolution provisions (Parts 7, 8) migrated from the MAS Act.
- **Phase 2B — 31 July 2024:** MAS's **harmonised and expanded power to issue prohibition orders (Part 3)** — one regime across all sectors.
- **Phase 3 — 30 June 2025:** the framework to regulate **digital token service providers** established or incorporated in Singapore that provide DPT services *outside* Singapore (**Part 9**) — the extraterritorial DPT licensing regime.

The FSMA applies to the full sweep of FIs: banks, merchant banks, finance companies, PS Act licensees, insurers and intermediaries, financial advisers, CMS licensees, exchanges, clearing houses, trade repositories, benchmark administrators, CIS trustees, trust companies, designated financial holding companies, credit/charge-card issuers, designated payment-system operators, and FSMA-licensed DPT providers ✅. **Practical consequence for a bank:** since May 2024, "technology risk management" for banks lives in **Notice FSM-N05** issued *under the FSMA*, not under the Banking Act; since July 2024, prohibition orders are a single Part 3 power; since June 2025, MAS can reach DPT service providers operating from Singapore into foreign markets.

### 2.6 The Financial Advisers Act 2001 (the "FTA" reading)

✅ From the acts catalogue: the FAA 2001 "governs the regulation of financial advisers in Singapore, including their representatives and supervisors". This is the most plausible referent of the brief's "FTA" (flagged in §11). The FAA covers licensed financial advisers (LFAs) and exempt FAs (including the "serving up to 30 accredited investors" class), the representative regime, and the conduct and disclosure obligations for financial advisory services — and it carries its own notice family (FAA-N06 AML, FAA-N24/N25 cross-border, FAA-N18 TRM cancelled 2024). Where Cymbal Bank sells investment products with advice, the FAA overlay applies on top of the SFA product rules.

### 2.7 The Insurance Act 1966

✅ From the acts catalogue: the Insurance Act 1966 "governs the regulation of insurance business in Singapore, including insurers, insurance intermediaries and related institutions". Note the year — **1966**, not 1970: the Insurance Act predates MAS itself and is the oldest of the big financial statutes in the catalogue. Its notice family is the 100–300 series (Notice 126 ERM for insurers, Notice 314 AML for life insurers, Notice 125 investments — see §3.6), and its guidelines include the Risk Management Practices for Insurance Business series (Core Activities; Insurance Fraud Risk — both 10 January 2024 revisions) and the ORSA guidance (2017) ✅.

### 2.8 The Rest of the 18 — the Supporting Acts

- **Finance Companies Act 1967** — the finance-company licence; the 800-series notices hang off it (Notice 824 AML, Notice 806 MLA, the cancelled 830 TRM / 834 cyber hygiene).
- **Trust Companies Act 2005** — the "TPA" reading (flagged); trust-company licensing, TCA-N03 AML notice.
- **Terrorism (Suppression of Financing) Act 2002** — the CFT pillar; sanctions and terrorist-financing offences; read with the UN sanctions regulations and MAS's Financial Measures in Relation to Russia (SNR-N01) and Violent Israeli Settlers (SNR-N03) notices ✅.
- **Deposit Insurance and Policy Owners' Protection Schemes Act 2011** — the SDIC deposit-insurance scheme (S$100k coverage ✅-⚠ per the repo) and the PPF for life policies.
- **Payment and Settlement Systems (Finality and Netting) Act 2002** — settlement finality protection.
- **Financial Holding Companies Act 2013** — designated FHCs (banking and insurance); the FHC-N-series notices (FHC-N126 ERM, FHC-N649 MLA/LCR, FHC-N651/652/653).
- **Business Trusts Act 2004, Commodity Trading Act 1992, Bills of Exchange Act (Cap. 23), Exchanges (Demutualisation and Merger) Act 1999, and SFA Part VII (Disclosure of Interest)** — the supporting cast; relevant only where the bank's business reaches into those corners.

### 2.9 The Statute Years — Verified vs Flagged

| Statute | Year | Status this pass |
|---|---|---|
| Banking Act | **1970** | ✅ verified (MAS Banking Act page: "Banking Act 1970") |
| Securities and Futures Act | **2001** | ✅ verified (catalogue) |
| Payment Services Act | **2019** | ✅ verified (catalogue + page: Parliament passed 14 Jan 2019) |
| Financial Services and Markets Act | **2022** | ✅ verified (Parliament passed 5 Apr 2022; phases verified) |
| Financial Advisers Act | **2001** | ✅ verified (catalogue) |
| Insurance Act | **1966** | ✅ verified (catalogue) |
| MAS Act | **1970** | ✅ verified (MAS formed 1 Jan 1971) |
| Finance Companies Act | **1967** | ✅ verified (catalogue) |
| Trust Companies Act | **2005** | ✅ verified (catalogue) |
| "FTA" | — | ⚠ not in catalogue under that abbreviation; read as FAA 2001 |
| "TPA" | — | ⚠ not in catalogue under that abbreviation; read as TCA 2005 |

---

## 3. The Notices

### 3.1 Notices as a Legal Instrument

A **Notice** is MAS's primary binding instrument for a *class* of institutions ✅ (see the instrument pyramid in §1.3). Notices are issued under specific Acts (the header of every notice names its statutory basis — e.g. "Issued pursuant to: Financial Services and Markets Act 2022" on the FSM notices ✅) and impose legally binding requirements; contravention can be specified as an offence. Every notice carries a **content number** that encodes its family: banks live in the **600-series** (626, 637, 644, 649, 651–653, 658), merchant banks in the **1000-series** (1003, 1005, 1007, 1011, 1013, 1014, 1015, 1108, 1112, 1114, 1121) and the legacy **700-series** (730 — see §3.3), finance companies in the **800-series** (806, 824, 830, 834), insurers in the **100–300-series** (101, 114, 117, 120, 125, 126, 127, 129–131, 211–213, 302, 307, 314, 320–322, 502, 508), capital-markets entities in the **SFA 04-* / CMG-* / FAA-N* series**, and payment institutions in the **PSN-series** — plus the new **FSM-series** under the FSMA that cuts across sectors (tech risk, cyber hygiene, AML for DPT providers). The 30 June 2025 revision wave updated the AML/CFT notices sector-wide ✅ (Notice 626, 626A, 824, 1014, 314, TCA-N03, SFA 04-N02, FAA-N06, PSN01, PSN02, PSN10, PSM-N01, VCC-N01, BTA1-N01 all show a 30 June 2025 revision date in the catalogue).

### 3.2 The Banking 600-Series

The notices that define a bank's operational law, verified ✅ from the on-disk catalogue:

| Notice | Title | Revised / published | What it does |
|---|---|---|---|
| **626** | Prevention of Money Laundering and Countering the Financing of Terrorism – Banks | 2025-06-30 ✅ | The bank AML/CFT notice: customer due diligence, beneficial ownership, ongoing monitoring, STR filing, sanctions screening, record-keeping, board/MLRO accountability. The single most-audited notice in a bank's compliance stack. |
| **626A** | AML/CFT – Credit Card or Charge Card Issuers | 2025-06-30 ✅ | The card-issuer AML companion |
| **637** | Technology Risk Management (banks) | ⚠ | The cross-referenced [Banks in Singapore](banks_in_singapore_guide.md) §5 labels the bank TRM notice "637"; the catalogue shows the bank TRM notice **644** and 644A **cancelled with effect from 10 May 2024**, replaced by **FSM-N05** under the FSMA ✅. Treat "637 vs 644" as history; the live requirement is FSM-N05. |
| **644 / 644A** | Technology Risk Management [Cancelled] | 2024-05-09 ✅ | Cancelled 10 May 2024 — superseded by the FSMA FSM-series |
| **649** | Minimum Liquid Assets and Liquidity Coverage Ratio | 2026-05-28 ✅ | The bank liquidity notice: MLA (Singapore-dollar liquid assets) + LCR (Basel III) — the post-2023-outage liquidity regime, recently revised |
| **651** | Liquidity Coverage Ratio Disclosure | 2024-07-01 ✅ | Public LCR disclosure |
| **652** | Net Stable Funding Ratio | 2024-05-16 ✅ | The NSFR requirement |
| **653** | Net Stable Funding Ratio Disclosure | 2024-07-01 ✅ | Public NSFR disclosure |
| **658** | Management of Outsourced Relevant Services for Banks | 2023-12-11 ✅ | The outsourcing notice: materiality assessment, risk management of outsourced relevant services, notification of material outsourcing, exit plans — read with the 2023 Outsourcing Guidelines (§4.4) and the FAQ on Notices 658 and 1121 (11 Dec 2024 ✅) |
| **613** | Minimum Liquid Assets [Cancelled] | 2022-06-24 ✅ | Superseded by the 649-series |
| **FHC-N649 / FHC-N651/652/653** | Same titles for designated financial holding companies | 2026-05-28 / 2024-05-16 ✅ | The FHC layer of the same liquidity obligations |

**The TRM angle for banks.** The May 2024 consolidation is the key architectural fact: the sectoral TRM notices (banks **644/644A**, merchant banks **1114** and **830**, finance companies **506**, insurers **127** and **912**, CMFIs **CMG-N02**, FAs **FAA-N18**, trust companies **TCA-N05**, PSPs **PSN05/PSN06**, card licensees and cyber-hygiene notices **834/CMG-N03/FHC-1119**) were **all cancelled with effect from 10 May 2024** ✅ and replaced by harmonised FSMA notices for each sector: **FSM-N05 (banks)**, FSM-N07 (card licensees), FSM-N09 (finance companies), FSM-N17 (credit bureaus), FSM-N19 (registered insurance brokers), FSM-N21 (capital-markets FIs), FSM-N22 (cyber hygiene), FSM-N25 (licensed trust companies) ✅. Each sets "requirements for a high level of reliability, availability and recoverability of critical IT systems and to implement IT controls to protect customer information from unauthorised access or disclosure" ✅ — the exact same sentence across sectors, which is the point of the harmonisation.

### 3.3 The Merchant-Bank Notices and the 730 Question

The merchant-bank family (MAS Act / FSMA) is the **1000-series** ✅:

| Notice | Title | Date ✅ | What it does |
|---|---|---|---|
| **1014** | AML/CFT – Merchant Banks | 2022-03-01 | The merchant-bank AML notice (the *current* one) |
| **1015** | Minimum Liquid Assets and Liquidity Coverage Ratio | 2026-05-28 | Merchant-bank liquidity (mirror of 649) |
| **1121** | Management of Outsourced Relevant Services for Merchant Banks | 2023-12-11 | The merchant-bank outsourcing notice (mirror of 658); the FAQ on 658 and 1121 was published 11 Dec 2024 ✅ and the Outsourcing Guidelines name Annex D of both notices as the exempted-services list ✅ |
| **1003** | Submission of Statistics and Returns | 2024-07-01 | MAS returns for merchant banks |
| **1005** | Credit Files, Grading and Provisioning | 2021-06-29 | Credit-risk discipline |
| **1007** | Appointment of External Auditors | 2021-06-29 | Annual auditor appointment |
| **1002** | Auditors' Reports and Additional Information | 2021-06-29 | Auditor reporting with annual accounts |
| **1013** | Disclosure in Financial Statements | 2021-06-29 | Financial-statement disclosure |
| **1011** | Appointment of Chief Executives [Cancelled] | 2021-06-29 | Fit-and-proper notification (cancelled) |
| **1108 / 634** | Banking Secrecy – Conditions for Outsourcing [Cancelled] | 2024-12-10 | The old banking-secrecy outsourcing conditions, cancelled as the 658/1121 regime took over |
| **1112** | Reporting of Suspicious Activities & Incidents of Fraud | 2013-01-23 | For FIs approved under **s 28 of the MAS Act** — the fraud/STR reporting notice for MAS-licensed (non-Banking-Act) institutions; the pattern now generalised by CMG-N01 and FSM-N27 |

**The 730 question (flagged ⚠).** The task brief referred to a "payments 700-series (the 730 AML)". The on-disk catalogue **does not contain a Notice 730 record** — a search of the full notices list returns nothing for "730". The prior research pass also attempted web confirmation and both searches returned empty. What the catalogue *does* show: the merchant-bank AML notice is **1014 (2022-03-01)** — and the widely documented history is that Notice 730 (AML/CFT – Merchant Banks, under the MAS Act) was the *predecessor* that 1014 replaced; the old money-changing/remittance AML obligations were likewise folded into the PS Act's PSN-series from 2020. So the "700-series" is best read as the **legacy merchant-bank/MAS-Act AML notice family**, not a payments series — payments AML today lives in PSN01/PSN02. **Treat the "730" designation itself as unconfirmed ⚠** and cite "Notice 1014 (and its predecessor, legacy Notice 730 ⚠)" when writing compliance documentation.

### 3.4 The Payments Notices (the PSN-Series and the FSM-N30/N31)

Verified ✅ from the catalogue and the PS Act page:

| Notice | Title | Date ✅ | What it does |
|---|---|---|---|
| **PSN01** | AML/CFT – Specified Payment Services | 2025-06-30 | The core PSP AML notice (all regulated payment services except DPT) |
| **PSN02** | AML/CFT – Digital Payment Token Service | 2025-06-30 | The DPT-service AML notice — CDD, travel-rule-aligned transfer information, sanctions screening, transaction monitoring for token transfers |
| **PSN03** | Reporting of Suspicious Activities and Incidents of Fraud | 2019-12-05 | The PS Act fraud/STR reporting notice |
| **PSN04 / PSN04A** | Submission of Regulatory Returns / Statement of Transactions and P&L | 2024-04-02 / 2020-01-28 | Returns |
| **PSN05 / PSN06** | Technology Risk Management / Cyber Hygiene [Cancelled] | 2024-05-09 | Cancelled with the May 2024 TRM consolidation; PSPs now covered by FSM-N30/N31 (below) and the TRM Guidelines |
| **PSN07** | Notice on Conduct | 2024-04-02 | Conduct requirements for PSPs (post-expansion) |
| **PSN08** | Disclosures and Communications | 2024-04-02 | Customer disclosures |
| **PSN09** | Specified Matters and Forms | 2026-03-27 | Forms and specified matters |
| **PSN10** | AML/CFT – Exempt Payment Service Providers | 2025-06-30 | AML obligations for *exempt* PSPs (e.g. banks' payment services) — directly relevant to Cymbal Bank where its payment activities sit outside a PS Act licence |
| **PSN11** | Restrictions in Relation to Cross-Border Money Transfer Services | 2024-09-25 | Restrictions on cross-border money-transfer services (the 2024 expansion) |
| **PSN01A / PSN01AA** | Transitional AML notices for exempted account-issuance providers | 2019-12-05 / 2025-06-30 | The stored-value-facility transition |
| **FSM-N27** | AML/CFT – Digital Token Service Providers | 2025-05-30 | The FSMA Part 9 AML notice for FSMA-licensed DPT providers (extraterritorial) |
| **FSM-N30** | Technology Risk Management (DTSPs) | 2025-05-30, effective 30 Jun 2025 | Reliability/availability/recoverability of critical IT systems + protection of customer information — the DPT equivalent of FSM-N05 ✅ |
| **FSM-N31** | Cyber Hygiene (DTSPs) | 2025-05-30, effective 30 Jun 2025 | Securing administrative accounts, security patching, baseline security standards, network security devices, anti-malware, stronger user authentication ✅ |

### 3.5 The Capital-Markets Notices and the "CM 800-Series" Question

The brief's "CM 800-series" needs a correction flag ⚠: in the catalogue the **800-series is the finance-company/legacy family** (806 Minimum Cash Balances and Minimum Liquid Assets, 824 AML – Finance Companies, 830 TRM [cancelled], 834 Cyber Hygiene [cancelled]) ✅ — not capital markets. The capital-markets notice families are:

- **CMG-N01** — Reporting of Suspicious Activities and Incidents of Fraud (20 Aug 2024 ✅) — the sector-wide fraud/STR reporting notice replacing the old s 28 MAS Act pattern
- **CMG-N02 / CMG-N03** — TRM / Cyber Hygiene for CM entities [Cancelled, 9 May 2024 ✅] → superseded by FSM-N21 / FSM-N22
- **SFA 04-N02** — AML/CFT for Capital Markets Intermediaries (2025-06-30 ✅)
- **SFA 04-N19 / SFA 04-N20** — Cross-Border Arrangements under the SFA (2025-06-30 ✅)
- **SFA 02A/03-N01** — Financial Market Infrastructure Standards (2016-06-16 ✅); **SFA 03AA-N02** for CSDs
- **SFA 13-N01** — AML for Approved Trustees; **SFA 03AA-N01** — AML for the Depository; **SFA02-N05** — AML for Approved Exchanges/RMOs; **FAA-N06** — AML for FAs; **FAA-N24/N25** — cross-border arrangements ✅

### 3.6 The Insurance Notices and the "1000-Series" Question

Second correction flag ⚠: the insurance notices are the **100–300 series**, not 1000-series (the 1000-series is the merchant-bank family, §3.3). Verified ✅ insurance notices from the catalogue:

- **Notice 314** — AML/CFT – Life Insurers (2025-06-30 ✅)
- **Notice 126** — Enterprise Risk Management ("ERM") for Insurers (2022-09-30 ✅) — the insurer ERM notice covering credit, liquidity, market, operational, technology, insurance and fraud risk
- **Notice 127** — Technology Risk Management [Cancelled, 10 May 2024 ✅]
- **Notice 125** — Investments of Insurers (2024-12-20 ✅); **Notice 101** — Maintenance of Insurance Funds; **Notice 114** — Reinsurance Management; **Notice 302** — Product Development and Pricing; **Notice 307** — Investment-Linked Policies; **Notice 320** — Management of Participating Life Business; **Notice 321** — Direct Purchase Insurance Products; **Notice 120** — Disclosure for A&H products; **Notice 211/212/213/129/130/131** — Returns; **Notice 502/508** — broker standards and misconduct reporting ✅

### 3.7 The Notice Map — Series / Family / Applies-to

| Series | Family | Applies to | Live anchors |
|---|---|---|---|
| 600 | Banks (Banking Act) | Full/wholesale banks, card issuers | 626 (AML), 649/651/652/653 (liquidity), 658 (outsourcing) |
| 1000 | Merchant banks (MAS Act/FSMA) | Merchant banks | 1014 (AML), 1015 (liquidity), 1121 (outsourcing) |
| 800 | Finance companies / legacy | Finance companies | 824 (AML), 806 (MLA) |
| 700 | Legacy MAS-Act AML (⚠) | Merchant banks (historical) | 730 ⚠ — superseded by 1014 |
| PSN | Payments (PS Act) | SPIs, MPIs, money-changers | PSN01/02 (AML), PSN07 (conduct), PSN10 (exempt PSPs) |
| FSM | Cross-sector (FSMA 2022) | All sectors by notice | FSM-N05 (bank TRM), FSM-N22 (cyber hygiene), FSM-N27 (AML DTSP), FSM-N30/N31 (DPT TRM/cyber) |
| CMG / SFA 04 / FAA-N | Capital markets (SFA/FAA) | CMS licensees, FAs | CMG-N01 (fraud reporting), SFA 04-N02 (AML) |
| 100–300 | Insurance (Insurance Act) | Insurers, brokers | 314 (AML), 126 (ERM), 125 (investments) |
| TCA / FHC / SNR / PSM / VCC / BTA | Sector-specific (various) | Trust cos, FHCs, DPT/PSM/VCC players | TCA-N03, FHC-N126/N649, SNR-N01/N03, PSM-N01 |

---

## 4. The Guidelines

### 4.1 Guidelines as an Instrument — What "Should" Means

MAS's own definition (✅ Supervisory Approach page): guidelines "set out principles or 'best practice standards' that govern the conduct of specified institutions or persons. Contravening guidelines is not a criminal offence and does not attract civil penalties, but specified institutions or persons should observe the spirit of these guidelines. **How well an institution or person observes the guidelines may have an impact on MAS' overall risk assessment** of that institution or person." That last sentence is the entire commercial logic of this guide: guidelines are "soft law" with hard consequences — a weak TRMG/BCM/FEAT posture raises the FIRA risk rating, which raises capital-adequacy scrutiny, which raises licence-condition risk.

### 4.2 The Technology Risk Management Guidelines (TRMG) — 2013 / 2021

✅ Verified from the TRM Guidelines page extract. The current text is the **Guidelines on Risk Management Practices – Technology Risk**, **published 18 January 2021** (the page's Published Date), described as: "risk management principles and best practices to guide financial institutions to establish **sound and robust technology risk governance and oversight, as well as maintain IT and cyber resilience**". The 2013 original version ⚠ is widely documented (the 2021 revision consolidated the 2013 TRMG with the subsequent cyber-hygiene circulars and the 2018 "Outsourcing and Third-Party" additions) but is **not separately confirmable from the on-disk files** — the extract confirms the 2021 date only; the "2013" origin is flagged.

Key verified facts ✅:

- **Applicability is enormous:** the applies-to list runs to ~60 institution types — every bank class, merchant banks, finance companies, card issuers, insurers and reinsurers of all stripes, CMS licensees (dealers, fund managers, REIT managers, corporate finance advisers, custodians, credit rating agencies, crowdfunding platforms), FAs, trust companies, MPI/SPI/money-changers, designated payment systems, exchanges, clearing houses, trade repositories, benchmark administrators, and the FHCs — i.e. the TRMG is the *universal* technology-risk expectation across the entire MAS perimeter.
- **Content:** technology risk governance and oversight (board and senior-management ownership, risk appetite, three lines of defence), IT resilience (system availability and recoverability), cyber resilience (security controls, detection, response), and — via its resource links — the **Notice on TRM FAQs** and the **Instructions on Incident Notification and Reporting to MAS** ✅.
- **The 2021–2024 sequence:** the 2021 guidelines were followed by the sectoral TRM *notices* (644, 830, 127, 506, 912, PSN05, CMG-N02, FAA-N18, TCA-N05, 1114, 644A) — all **cancelled 10 May 2024** ✅ — replaced by the FSMA FSM-series notices (§3.2/§3.4), which now sit *under* the same TRMG as the guidelines layer. So the stack for a bank is: **TRMG (guidelines, 2021) + FSM-N05 (notice, 2024) + Incident Notification Instructions + the December 2025 Incident Reporting Circular** — guidelines and notice read together. The SecOps/cyber reading of this stack lives in [../technology/secops_guide.md](../technology/secops_guide.md) and [../technology/cybersecurity_guide.md](../technology/cybersecurity_guide.md); the bank-level mapping is in [Banks in Singapore](banks_in_singapore_guide.md) §5 and [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md).

### 4.3 The Business Continuity Management (BCM) Guidelines — 2022

✅ Verified from the BCM Guidelines page extract. The revised **Guidelines on Business Continuity Management were issued on 6 June 2022** and "set out the need for financial institutions to take an **end-to-end service-centric view** in ensuring the continuous delivery of critical business services to their customers." They **supersede the June 2003 version and the January 2006 circular "Further Guidance on BCM"** (which carried pandemic and physical-security guidance — the 2022 text absorbs it). Implementation discipline is spelled out: FIs "should meet the new Guidelines and establish a **BCM audit plan within 12 months** following issuance; the **first BCM audit should be conducted within 24 months**" ✅. The page's consultation history documents the process: first consultation March 2019, second consultation P015-2021 (Oct–Nov 2021) that incorporated COVID-19 learnings, response June 2022 ✅. The BCM Guidelines apply to the full FI perimeter (banks, merchant banks, finance companies, insurers, CMS licensees, PSPs, trust companies, credit bureaus, brokers) ✅. **The BCM-to-2025 arc:** BCM (2022) → the operational-resilience focus areas → the **Circular on Financial Institution Incident Reporting (16 December 2025)** ✅, which moved all reportable-incident submissions onto the **MAS-Tx platform with an updated template from 1 February 2026** — the operational-resilience regime is now a notification-and-audit loop, not a document-on-the-shelf exercise.

### 4.4 The Outsourcing Guidelines — 2023

✅ Verified from the Outsourcing (Banks) Guidelines page extract. The **Guidelines on Outsourcing (Banks)** were published **11 December 2023** and are **effective 11 December 2024**; they "set out MAS' expectations of a bank or merchant bank that has entered into or is planning to enter into an arrangement for ongoing outsourcing relevant services, with the exception of arrangements for **exempted Outsourced Relevant Services set out in Annex D of MAS Notices 658 and 1121**." They apply to all banks and merchant banks in Singapore ✅. The companion instruments are all verified ✅: **Notice 658** (banks) and **Notice 1121** (merchant banks), both published 11 Dec 2023; the **FAQ on MAS Notice 658 and 1121 (Banks)** published 11 Dec 2024; the older Guidelines on Outsourcing [Cancelled] and the banking-secrecy outsourcing notices 634/1108 [Cancelled 10 Dec 2024]. The 2023 refresh is the third-party-risk management (TPRM) codification: materiality assessment of outsourced relevant services, risk management over the full lifecycle (due diligence, contracts, monitoring, exit), notification obligations for material outsourcing, and the expectation that banks treat cloud and critical service providers as outsourced relevant services regardless of label — see the TPRM focus area on the MAS Regulation index ✅.

### 4.5 The IAC — Guidelines on Individual Accountability and Conduct — 2020

⚠ **Not confirmable from the on-disk /tmp research** — none of the research files contain the IAC document (the brief's "IAC (2020)" is not in the extracts). It is flagged here per the honesty convention, and cross-referenced to the repo guides that carry the accountability theme: [ai_genai_banking_compliance_guide.md](ai_genai_banking_compliance_guide.md) (FEAT's Ethics & Accountability principle, the AI model-risk information paper) and [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md) (the three-lines-of-defence machinery the IAC formalises). What is widely documented (⚠, verify against MAS): the **Guidelines on Individual Accountability and Conduct** took effect **10 September 2020** and require FIs to (a) clearly **define and allocate responsibility** for key functions (senior managers), (b) ensure **conduct** expectations are set and enforced for material-risk personnel (including the "five conduct standards" — proper care/skill, diligence, honesty, disclosure of conflicts, cooperation with regulators), and (c) establish governance so that accountability is auditable. The IAC is the cultural counterpart to the FEAT framework — FEAT says *treat customers fairly*; IAC says *be able to name who is responsible when you don't*.

### 4.6 FEAT — Fairness, Ethics, Accountability, Transparency — 2018

⚠ Same flag: **FEAT is not in the /tmp research files** — it is carried authoritatively by the repo's technology cluster. Per [../technology/ai_trust_assessments_guide.md](../technology/ai_trust_assessments_guide.md) (verified facts there) and [../technology/responsible_ai_frameworks_guide.md](../technology/responsible_ai_frameworks_guide.md) (§6.1, the full treatment), the **Principles to Promote Fairness, Ethics, Accountability and Transparency (FEAT) in the Use of Artificial Intelligence and Data Analytics** were published by MAS **12 November 2018** (⚠ date — confirm) as a voluntary-but-authoritative set of 14 principles: **Fairness** (decisions should not be biased; data should be sound and relevant), **Ethics** (AI decisions should be ethical and aligned with the institution's stated values), **Accountability** (human accountability for AI outcomes; senior-management ownership), **Transparency** (customers should know when AI is used and be able to query and appeal decisions). MAS expects banks to *demonstrate* fair dealing — [ai_genai_banking_compliance_guide.md](ai_genai_banking_compliance_guide.md) §2 shows how FEAT is treated as binding-in-practice for credit decisions, with the 2024 AI model-risk information paper and the **13 November 2025 consultation proposing Guidelines on AI Risk Management (AIRM)** — which would apply to *all* FIs and complement FEAT and the national Model AI Governance Framework for Generative AI (2024) — making the direction explicit (AIRM status: proposed at writing, ⚠ verify final text).

### 4.7 Model AI Governance → Verifiable AI

⚠ Not in the /tmp files; carried by the repo cluster. The Singapore AI-governance stack per [../technology/ai_trust_assessments_guide.md](../technology/ai_trust_assessments_guide.md): the **Model AI Governance Framework** (first edition 2019, updated editions; the framework layer for organisations), **AI Verify** (Singapore's AI testing toolkit — the "verifiable AI" strand: technical testing of explainability, fairness, robustness, safety, transparency), and the **Model AI Governance Framework for Generative AI** (2024, IMDA/AICoE) — plus, on the MAS side, FEAT (2018), the AI model-risk information paper (2024), and the proposed AIRM guidelines (2025 consultation). For Cymbal Bank the "Model AI Governance → Verifiable AI" arc means: encode FEAT principles in the AI inventory, use AI Verify-style testing for customer-facing models, run model-risk management in the SR 11-7 tradition, and be ready for AIRM when finalised. The banking-domain synthesis is [ai_genai_banking_compliance_guide.md](ai_genai_banking_compliance_guide.md) §2–§3; the assessment machinery is [../technology/ai_trust_assessments_guide.md](../technology/ai_trust_assessments_guide.md).

### 4.8 The Other Guidelines That Bind

- **Guidelines on Risk Management Practices – Operational Risk (1 March 2013)** ✅ — verified on the BCM page's related list; the operational-risk framework guidelines, of which business continuity and outsourcing were historically part.
- **Guidelines on Risk Management Practices for Insurance Business – Core Activities and – Insurance Fraud Risk (both revised 10 January 2024)** ✅ — insurer risk-management guidance.
- **Guidance on Insurers' Own Risk and Solvency Assessments (19 July 2017)** ✅ — ORSA observations.
- **Guidelines on the Application of Banking Regulations to Islamic Banking (16 May 2024)** ✅ — Islamic-banking application guidance.
- **Guidelines on Definition of a "Deposit" (28 June 2021)** ✅ — the deposit-definition test (a product-structure question every bank treasury faces).
- **Guidelines on Provision of Digital Advisory Services (CMG-G02, 8 October 2018)** ✅ — robo-advisory guidance (the SFA/FAA edge of FEAT).
- **E-Payments User Protection Guidelines** — named in the anti-scam circular ✅ (⚠ content not extracted).
- **Guidelines on Outsourcing [Cancelled]** — the 2018-era cross-sector outsourcing guidance, cancelled as the 2023 bank-specific regime took over ✅.

---

## 5. The Industry Expectations

"Industry expectations" is the layer above the instruments: the supervisory themes MAS pushes through speeches, consultations, information papers, and circulars — things the industry is *expected* to do before they become notices. The MAS Regulation index names the current focus areas ✅: Anti-Money Laundering, Enforcement, Combatting Scams, Cyber Security, Third-Party Risk Management, FinTech Regulatory Sandbox, Interest Rate Benchmarks Transition, Operational Resilience.

### 5.1 Culture and Conduct

MAS's conduct agenda runs FEAT (fair dealing with AI), the IAC (individual accountability), and the conduct notices (PSN07 for PSPs, the conduct framework for banks via the 600-series and Banking Act expectations). The supervisory ask: boards own conduct outcomes; senior managers are individually accountable; AI and data-analytics decisions are fair, ethical, accountable, transparent; customers can query automated decisions. For Cymbal Bank this is the "how we behave" layer — implemented through the IAC-style responsibility mapping and FEAT-style AI governance described in §4.5–§4.7 (cross-ref [ai_genai_banking_compliance_guide.md](ai_genai_banking_compliance_guide.md)).

### 5.2 Operational Resilience

The post-2023 operational-resilience push (the 2023–24 bank-outage episode that made technology risk a board-level issue — see [Banks in Singapore](banks_in_singapore_guide.md) §5) is now a four-part stack, all verified ✅ in this pass:

1. **BCM Guidelines (June 2022)** — end-to-end, service-centric continuity (§4.3);
2. **The TRMG + FSM-series notices (2021/2024)** — technology and cyber resilience (§3.2, §4.2);
3. **The Outsourcing Guidelines + Notices 658/1121 (2023–24)** — third-party and cloud resilience (§4.4);
4. **The Circular on Financial Institution Incident Reporting (16 December 2025)** — all reportable incidents to MAS on the **MAS-Tx** platform with the updated template **from 1 February 2026**, "as soon as possible, but no later than the timeline prescribed in the applicable act(s), regulation(s), notice(s), circular(s) or guideline(s)" ✅.

Add the D-SIB/D-SII designation regime (§1.1) and the macroprudential overlay, and "operational resilience" stops being a BCP document and becomes a *measurable, notified, audited* capability.

### 5.3 Climate Risk

⚠ Not in the /tmp files — carried by the repo's sustainable-finance cluster and flagged here. The relevant MAS instruments (verify against MAS): the **Guidelines on Environmental Risk Management (ENR Guidelines, first issued 2020, revised 2023 ⚠)**, which expect banks to integrate climate risk into governance, strategy, and risk management (credit, market, liquidity, operational), and the broader sustainable-finance agenda (taxonomy, transition credits, disclosure expectations aligned with the ISSB). The development half of MAS's triple role (§1.1) pushes green finance as a market-building priority. For Cymbal Bank's risk systems: climate is a *credit-risk input* (exposure classification, transition risk), a *reporting obligation* (disclosure), and a *stress-testing scenario*.

### 5.4 The Stablecoin Framework

⚠ Not in the /tmp files; verified facts live in [tokenized_assets_guide.md](tokenized_assets_guide.md) §2.4/§13 and are cross-referenced. The **MAS Stablecoin (SCS) framework — August 2023** is the world's first comprehensive stablecoin regime: applicable to single-currency stablecoins pegged to the SGD or any G10 currency; issuer requirements of **100% reserve backing at all times** (with composition, valuation, custody and audit requirements), **redemption at par within five business days**, **monthly independent attestation and annual audits**, minimum capital, and business-conduct restrictions — an SCS issuer may not provide other services (no lending, staking, or dealing in other DPTs) and cannot hold stakes in other entities. The label "MAS-regulated stablecoin" is reserved for compliant issuers. The framework is being legislated into the PS Act as a **"Stablecoin Issuance Service" (2026 ⚠ verify)**. For a bank: if Cymbal Bank issues or custodies stablecoins (e.g. a tokenized-deposit or SCS product), the reserve segregation, attestation, and redemption SLAs become core operational controls.

### 5.5 The DPT Measures

Cross-ref [tokenized_assets_guide.md](tokenized_assets_guide.md) §13.2 (⚠ facts flagged there; PS Act details verified in this pass). The DPT regime in layers:

- **Licensing:** DPT services have been regulated under the PS Act since 2020, with the full regime applied from **October 2022** (SPI vs MPI thresholds); **~30+ MPIs licensed for DPT services by 2025 ⚠**. MAS has been deliberately selective (famously rejecting Binance's application ⚠).
- **Safeguarding:** customer money and tokens held apart from the firm's assets; **no lending of customer DPTs** — the two hard rules.
- **AML:** PSN02 (2025-06-30 revision ✅) — CDD, transfer-information (travel-rule) requirements, monitoring.
- **Technology risk:** FSM-N30 (TRM) and FSM-N31 (cyber hygiene), both effective **30 June 2025** ✅ — the DPT sector now carries the same reliability/availability/recoverability and cyber-hygiene obligations as banks.
- **Extraterritorial reach:** FSMA Part 9 (Phase 3, 30 June 2025 ✅) — Singapore-incorporated DPT providers serving customers *outside* Singapore need an FSMA licence; AML duties via FSM-N27 (30 May 2025 ✅).
- **Consumer guardrails:** no retail DPT leverage, restricted retail access ⚠ (see [tokenized_assets_guide.md](tokenized_assets_guide.md) §13.3).

### 5.6 Scams and Consumer Protection

Verified ✅ from the PS Act page: the **Circular on Anti-scam Measures by Major Payment Institutions Providing Personal Payment Accounts that Contain E-money (25 October 2024)** sets MAS's supervisory expectations for MPI e-wallet providers — read together with the **E-Payments User Protection Guidelines** and the phased anti-scam measures (transaction alerts, kill-switches, mule-account detection). Scam-combatting is one of MAS's named focus areas ✅ (the AML/CFT and fraud-reporting notices — CMG-N01, PSN03, 1112 — are its enforcement spine). The enforcement record (§6) shows the human cost: prohibition orders for bank staff who accessed customer information without authorisation.

---

### 5.7 The Expectations Table

Pulling §5 together into the one-page "what MAS expects of a bank like Cymbal Bank" table:

| Expectation | Core content | Status this pass |
|---|---|---|
| **Culture & conduct** | IAC-style named accountability; FEAT-style fair dealing; conduct standards for material-risk personnel | ⚠ instruments not in /tmp; cross-ref [ai_genai_banking_compliance_guide.md](ai_genai_banking_compliance_guide.md) |
| **Operational resilience** | End-to-end BCM (2022); tech-risk governance (TRMG + FSM-N05); TPRM (658 + 2023 Guidelines); incident reporting on MAS-Tx from 1 Feb 2026 | ✅ fully verified this pass |
| **Cyber hygiene** | Administrative-account security, patching, baseline standards, network security devices, anti-malware, strong authentication (FSM-N31 text, mirrored in the sector notices) | ✅ verified (FSM-N31 extract) |
| **Climate risk** | ENR Guidelines: climate in governance, strategy, credit risk; disclosure | ⚠ cross-ref repo sustainable-finance cluster |
| **Stablecoin issuance** (if applicable) | SCS: 100% reserves, 5-day par redemption, monthly attestation, no other services | ⚠ cross-ref [tokenized_assets_guide.md](tokenized_assets_guide.md) §2.4 |
| **DPT services** (if applicable) | PS Act licensing (full regime since Oct 2022), safeguarding, no lending of customer tokens, PSN02 AML, FSM-N30/N31 tech risk | ✅ PS Act mechanics verified; market facts ⚠ |
| **Scam combatting** | Anti-scam circular (Oct 2024) for e-wallet MPIs; transaction alerts, kill-switches, mule-account detection; E-Payments User Protection Guidelines | ✅ circular verified |
| **Fair, ethical, accountable, transparent AI** | FEAT principles; Model AI Governance → Verifiable AI (AI Verify); AIRM guidelines proposed (Nov 2025) | ⚠ cross-ref [../technology/ai_trust_assessments_guide.md](../technology/ai_trust_assessments_guide.md) |

The pattern across every row: **MAS moves expectation → consultation → guideline → notice in roughly a 2–4 year cycle.** Anti-scam measures started as a circular (2024) and are hardening into conduct requirements; AI started as FEAT principles (2018) and is moving to AIRM guidelines (2025); technology risk moved from TRMG guidance (2013/2021) to binding notices (2024). A bank that treats the expectations layer as "soft" is always exactly one cycle behind.

---

## 6. The Enforcement

### 6.1 The Enforcement Framework

✅ Verified from the Enforcement Actions page extract. MAS publishes "the list of formal regulatory and enforcement actions taken by MAS for breaches of laws and regulations administered by MAS and related matters". Publication mechanics:

- Information stays on the Enforcement Actions page **for five years** from publication, **except prohibition orders still in force**, which remain until they stop being in force.
- **With effect from 1 April 2021**, prohibition orders of **duration less than five years** are published *only* on this page and not on the News page (unless public-interest publication is assessed as appropriate).
- A financial institution (or its agent) may, as part of background checks, obtain from MAS records of prohibition orders previously issued against a person, **upon payment of a fee** — this is the "prohibition-order check" that HR and compliance run in hiring.

The enforcement toolkit spans the full ladder: **criminal prosecution** (for offences under the Acts), **civil penalty actions** (e.g. insider trading under the SFA), **composition penalties** (administrative fines in lieu of prosecution), **reprimands** (private or public), **revocation/suspension of regulatory status** (licence actions), and **prohibition orders** (banning individuals from regulated activities — since 31 July 2024 under the harmonised FSMA Part 3 power ✅ §2.5). Enforcement works *with* the pyramid: notices and guidelines define the standard; the enforcement record shows the standard is real.

### 6.2 The Enforcement-API Results — the Recent Actions

✅ Verified from the on-disk Enforcement Actions page extract (the first page of 14, i.e. the most recent 10 of the ~140 actions currently published — 14 pages × 10 per page):

| Issue date | Person/Company | Action type | Action |
|---|---|---|---|
| 12 Aug 2026 | Low Jia Mei, Camie | Prohibition Order | PO for **unauthorised access to bank customer information** |
| 03 Aug 2026 | Lai Mei Lin | Prohibition Order | PO |
| 27 Jul 2026 | Li Jinbo | Prohibition Order | PO |
| 09 Jul 2026 | Director & Compliance Manager, Samlit Moneychanger Pte. Ltd. | Criminal Prosecution | Charged for offences **under the Financial Services and Markets Act 2022** |
| 06 Jul 2026 | Andrew Tiew Siew Ing | Prohibition Order | PO for **cheating offences** |
| 30 Jun 2026 | Chua Han Boon Kenneth | Civil Penalty | Civil penalty for **insider trading** (SFA) |
| 25 May 2026 | Padang Trust Singapore Pte. Ltd. | Composition | **S$300,000 composition penalty for AML/CFT breaches** |
| 20 May 2026 | Tan Chun Yong, Xie Jianfeng | Criminal Prosecution | Convicted for offences under the **Securities and Futures Act** |
| 20 May 2026 | Bsquared Technology Pte Ltd | Revocation/Suspension | **Major Payment Institution licence revoked** |
| 18 May 2026 | Havenport Investments Pte Ltd + senior management | Composition \| Reprimand | Enforcement action against the firm **and its senior management** for breaches of MAS regulations |

**Reading the record.** Four themes jump out: (1) **prohibition orders dominate the individual side** — and the unauthorised-access case (Low Jia Mei Camie, 12 Aug 2026) is the one every bank CISO should frame: *accessing customer information without authorisation is a personal, career-ending offence even without financial gain*; (2) **senior management is in scope** — Havenport's enforcement names the firm *and* its senior management (the IAC principle made real); (3) **the FSMA is now the prosecution vehicle** (Samlit Moneychanger charged under the FSMA — the omnibus Act's offences in action); (4) **AML/CFT composition penalties run to six figures** (Padang Trust, S$300,000) and **licences get revoked** (Bsquared's MPI licence) — the DPT sector is not exempt from enforcement.

### 6.3 The 54 Prohibition Orders — Flagged

⚠ **Flagged claim.** The task brief states the prior research pass's enforcement-API run found **54 prohibition orders** (from the `enf.py` query against the MAS search API, filtering `mas_contenttype_s:"Enforcement Actions"` and reading `mas_enforcementaction_actiontype_s`). The on-disk files confirm the *query machinery* (enf.py ✅) and the *page mechanics* (14 pages of results ✅), but the **specific "54" count is not reproducible from the files on disk** — the page extract shows only the first 10 of 14 pages, and no saved API-result dump survives with the count. The count is consistent with the visible distribution (prohibition orders are the most frequent single action type on the page) but should be re-verified against the live MAS enforcement search (mas.gov.sg → Regulation → Enforcement → Enforcement Actions, filter Action Type = "Prohibition Order") before quoting externally. What *is* verified: prohibition orders are the most common formal action in the recent record (4 of the 10 most recent actions), they remain published until out of force, and they are checkable on a fee basis by prospective employers (§6.1).

### 6.4 The Penalties Framework

| Action type | Nature | Seen in the record ✅ |
|---|---|---|
| **Prohibition order** | Bans an individual from conducting regulated activities, taking up positions in FIs, or both; harmonised power under FSMA Part 3 since 31 Jul 2024; published until out of force | Low Jia Mei Camie, Lai Mei Lin, Li Jinbo, Andrew Tiew Siew Ing |
| **Criminal prosecution** | Offences under the Acts (FSMA, SFA, PS Act…), tried in court; can include imprisonment | Samlit Moneychanger (FSMA); Tan Chun Yong & Xie Jianfeng (SFA convictions) |
| **Civil penalty** | Monetary penalty pursued civilly — the SFA market-conduct regime | Chua Han Boon Kenneth (insider trading) |
| **Composition penalty** | Administrative fine paid in lieu of prosecution; common for AML/CFT and conduct breaches | Padang Trust S$300,000 (AML/CFT) |
| **Reprimand** | Private or public censure; often stacked with other actions | Havenport (with composition) |
| **Revocation / suspension** | Licence or approval withdrawn or suspended | Bsquared Technology (MPI licence revoked) |

**Consequence design.** MAS's enforcement philosophy (per the *Tenets of Effective Regulation* monograph ✅-listed in §1.1) is deterrence with proportionality: individuals get prohibition orders and prosecutions (personal accountability), institutions get compositions and licence actions (institutional deterrence), and the publication regime makes every action a reputational event for five years. The compliance takeaway for Cymbal Bank: the *individual* is the unit of enforcement — so the IAC-style accountability map, the fit-and-proper screening of every hire (including the prohibition-order record check), and the conduct standards are not HR paperwork but enforcement-adjacent controls.

---

## 7. The Banking Context — Cymbal Bank

### 7.1 The Bank and the Mandate

Cymbal Bank is a full bank operating in Singapore — the corpus persona's home institution, seen through the MAS framework this guide maps. As a locally incorporated or branch full bank (the licence tier determines which notices bite — every notice's applies-to list in §3 is the arbiter), Cymbal Bank sits under:

- **The Banking Act 1970** — the licence, the deposit-taking franchise, banking secrecy, fit-and-proper;
- **The FSMA 2022** — AML powers (Part 4), technology-risk notices (Part 5, via FSM-N05), prohibition orders (Part 3), and the general inspection powers (Part 2);
- **The SFA 2001** — wherever the bank deals in capital-markets products, advises, or manages funds (and the market-conduct regime for its traders);
- **The PS Act 2019** — for any payment services outside the banking licence (e-money, DPT services, money transfer — with the exempt-PSP AML duties via PSN10);
- **The MAS Act 1970** — the originating instrument and the residual reporting obligations (e.g. Notice 1112's s 28 approvals).

The **obligations × controls map** (§7.2) is the architect's view of the same territory: every MAS obligation on the left, the internal control that discharges it on the right, the system that implements the control in the middle. The repo treatments that carry the *systems* depth: [RegTech](regtech_guide.md) (the vendor/technology layer — AML stacks, KYC utilities, reporting engines), [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) (the in-house systems landscape — AML monitoring, regulatory reporting, risk engines), [Banks in Singapore](banks_in_singapore_guide.md) (the market and licence context), [AI/GenAI Banking Compliance](ai_genai_banking_compliance_guide.md) (the AI-governance obligations), and [../technology/secops_guide.md](../technology/secops_guide.md) + [../technology/cybersecurity_guide.md](../technology/cybersecurity_guide.md) (the TRMG/FSM-N05 control implementations).

### 7.2 The Obligations × Controls Map

| MAS obligation (instrument) | What it demands | Internal control / system | Owner |
|---|---|---|---|
| Banking Act 1970 — licence, fit-and-proper, secrecy | Licensed activities only; fit-and-proper management; customer-information confidentiality | Licence-perimeter monitoring, screening of all hires/appointments (incl. the PO record check, §6.1), data-access controls and audit logs | Compliance, HR, CISO |
| Notice 626 AML/CFT (banks) | CDD/EDD, beneficial-ownership capture, ongoing monitoring, STR filing, sanctions screening, record-keeping, MLRO | KYC/CDD workflow, sanctions and PEP screening, transaction-monitoring (TM) engine, STR case-management, registry retention — the RegTech stack of [RegTech](regtech_guide.md) §4 | AML/CFT & Financial Crime |
| Notice 649/651/652 (liquidity) | MLA holdings, LCR ≥ 100%, NSFR, disclosures | Liquidity-risk engine (intraday + LCR/NSFR computation), HQLA inventory, disclosure reporting — see [Treasury & ALM](treasury_alm_guide.md) | Treasury, Risk |
| Notice 658 + Outsourcing Guidelines (TPRM) | Materiality assessment of outsourced relevant services, lifecycle risk management, notification, exit plans | Third-party risk management (TPRM) platform: inventory, risk ratings, contract register, exit-plan store, cloud-provider oversight | Operational Risk, Procurement |
| FSM-N05 (TRM) + TRMG | Board-level tech-risk governance; reliability/availability/recoverability of critical systems; protection of customer information | IT risk governance (three lines), change/incident management, DR/BCP exercises, security operations (SIEM, EDR, IAM) — see [../technology/secops_guide.md](../technology/secops_guide.md), [../technology/cybersecurity_guide.md](../technology/cybersecurity_guide.md) | CISO, CTO |
| BCM Guidelines + Incident Reporting Circular (MAS-Tx) | End-to-end service continuity; BCM audit plan (12 months) and first audit (24 months); incident notification to MAS on MAS-Tx from 1 Feb 2026 | BCM program with business-impact analysis, recovery-time targets, continuity exercises, incident-notification runbook wired to MAS-Tx | Operational Resilience |
| IAC (2020) ⚠ | Named accountability for key functions; conduct standards for material-risk personnel | Responsibility mapping (RACI at senior-manager level), conduct attestations, accountability register | HR, Governance |
| FEAT (2018) ⚠ + Model AI Governance → Verifiable AI | Fair, ethical, accountable, transparent AI; testing of AI systems | AI inventory, fairness/robustness testing (AI Verify-style), model-risk management, AI governance board — see [AI/GenAI Banking Compliance](ai_genai_banking_compliance_guide.md), [../technology/ai_trust_assessments_guide.md](../technology/ai_trust_assessments_guide.md) | Model Risk, Data & AI |
| CMG-N01 / PSN03 / 1112 (fraud & suspicious activity reporting) | Timely reporting of reportable incidents and fraud | Fraud case-management, incident-detection (anti-scam, mule-account analytics), reporting workflow | Financial Crime, Fraud Ops |
| Climate expectations (ENR Guidelines ⚠) | Climate-risk integration into governance and credit risk | ESG data layer, climate stress-testing, exposure classification, disclosure reporting | Risk, Sustainability |
| Stablecoin/DPT (if applicable) ⚠ | SCS 100% reserves, 5-day redemption, attestation; DPT safeguarding, no lending of customer tokens | Reserve-management and attestation workflow, token custody segregation, PSN02/FSM-N30/N31 controls | Digital Assets, Payments |

---

### 7.3 The Supervisory Review Cycle — How the Map Gets Tested

The obligations × controls map is not a static artefact; it is exercised on MAS's supervisory rhythm (the FIRA framework ✅ §1.1, plus the enforcement and reporting mechanics ✅ §6):

1. **Continuous:** incident reporting (MAS-Tx from 1 Feb 2026 ✅), STR/fraud reporting (CMG-N01, PSN03, 1112 ✅), returns (Notice 1003 and the returns notices ✅), and the fee-based prohibition-order record checks during hiring ✅.
2. **Annual:** the D-SIB/D-SII systemic-importance assessment ✅; BCM audits (first audit within 24 months of the 2022 guidelines ✅); statutory audits with the auditor-report notices (1002/1007 ✅).
3. **Periodic:** thematic inspections (MAS's focus areas ✅), AML/CFT inspections (Notice 626), technology-risk inspections (TRMG/FSM-N05 — the post-2023 outage theme), and the on-site review of the very map this guide builds.
4. **Event-driven:** licence applications and variations (fit-and-proper), new-product notifications (e.g. the digital-asset line in §8), and enforcement (prosecution → composition → PO → licence action).

For Cymbal Bank the cycle converts to a compliance calendar: **MAS-Tx incident runbook (1 Feb 2026) · BCM audit plan · LCR/NSFR disclosure dates · AML notice-revision gap reviews (the 30 Jun 2025 wave) · hire-screening PO checks · AI-inventory and FEAT attestations.** The §8 matrix is the single artefact that proves the calendar is backed by controls.

---

## 8. The Worked Example — the Cymbal Bank MAS-Obligations × Controls Matrix

### 8.1 The Scenario

Cymbal Bank's Singapore entity is preparing for its annual MAS supervisory review and, separately, for a new **digital-asset product line** (custody + a tokenized-deposit pilot). The architecture team has been asked to produce the single artefact the regulators, the board, and the auditors will all ask for: **"show us the map from every MAS obligation to the control that discharges it."** The exercise below is that map, in the familiar Cymbal Bank context — the matrix form of §7.2, scored for coverage. It is an analytical design artefact (the "Design" convention used across the repo), not a factual claim about MAS — every instrument name is verified in §2–§5; every control is the bank's own design.

### 8.2 The Matrix

| # | MAS obligation | Instrument (verified ref) | Cymbal Bank control | System / evidence | Coverage |
|---|---|---|---|---|---|
| 1 | Licensed activities + fit-and-proper + secrecy | Banking Act 1970; FSMA Part 2/3 | Licence-perimeter rulebook; hire screening incl. PO check; data-access zoning | HR screening workflow; IAM audit logs | ✅ in place |
| 2 | AML/CFT for banks | Notice 626 (2025 rev) | CDD/EDD, sanctions & PEP screening, TM, STR, MLRO | KYC platform; TM engine; STR workflow | ✅ in place; 2025-revision gap review ⚠ in flight |
| 3 | Liquidity | Notice 649 (2026 rev), 651/652 | LCR/NSFR engine, HQLA book, disclosure packs | Treasury ALM system | ✅ in place; 2026 revision impact assessment ⚠ |
| 4 | Outsourcing / TPRM | Notice 658 + Outsourcing Guidelines + FAQ | TPRM inventory incl. cloud providers; materiality assessment; exit plans | TPRM platform; contract register | ⚠ partial — cloud exit plans being completed |
| 5 | Technology risk | FSM-N05 + TRMG | Tech-risk governance (board risk appetite); DR/BCP; security ops | SIEM/EDR; DR runbooks; BCP exercises | ✅ in place |
| 6 | Business continuity + incident reporting | BCM Guidelines (2022) + 2025 Circular | BIA; recovery targets; MAS-Tx incident runbook (1 Feb 2026) | BCM program; incident template on MAS-Tx | ✅ in place; runbook updated for MAS-Tx |
| 7 | Individual accountability | IAC ⚠ | Senior-manager responsibility map; conduct attestations | Accountability register | ⚠ partial — register being completed |
| 8 | AI governance | FEAT ⚠ + Model AI Gov Framework + AIRM (proposed) | AI inventory; fairness testing; MRM for credit models | AI governance board; AI Verify-style tests | ⚠ partial — inventory done; testing for new models |
| 9 | Fraud & suspicious activity reporting | CMG-N01 / PSN03 / 1112 | Fraud case management; anti-scam analytics; timely reporting | Case-management tool; reporting workflow | ✅ in place |
| 10 | Payments (exempt PSP) | PS Act + PSN10 | Exempt-PSP AML controls mapped from banking controls | PSN10 compliance note | ✅ in place (banking controls cover) |
| 11 | Digital-asset product line (new) | FSMA Part 9; PSN02; FSM-N30/N31; SCS framework ⚠ | Token custody segregation; no-lending policy; DPT travel-rule data; SCS reserve attestation (if issuing) | Custody system; wallet security; attestation workflow | 🔴 NEW — design phase; controls to be built before go-live |
| 12 | Climate | ENR Guidelines ⚠ | ESG exposure classification; climate stress input | ESG data layer | ⚠ partial |

**Reading the matrix.** Rows 1–6, 9, 10 are the *steady-state* bank: instruments verified ✅ in this pass, controls in place. Rows 7–8 are the *culture/AI* layer: instruments flagged ⚠ (not in the /tmp research — carried by repo guides), controls partially built — this is exactly the layer MAS's 2024–2026 AI and accountability agenda is hardening, so "partial" is a supervisory risk. Row 11 is the **strategic gap**: a digital-asset product line brings a *new* instrument stack (FSM-N30/N31, PSN02, FSMA Part 9, and the SCS framework if issuing stablecoins) that the bank's current controls do not discharge — the matrix makes the build plan obvious: custody segregation, travel-rule data, wallet key management, and (if issuing) the 100%-reserve attestation workflow must exist **before** go-live, not after. Row 12 is the coming wave.

### 8.3 The Lessons

1. **The map is the deliverable.** MAS reviews, board packs, and audits all converge on the same question — "which control discharges which obligation?" — and the matrix (§7.2/§8.2) is the answer in one page. RegTech exists to keep the map true ([RegTech](regtech_guide.md)).
2. **Guidelines are obligations in review.** TRMG/BCM/Outsourcing are "non-criminal" (§4.1) but they feed the FIRA risk assessment — the matrix should not distinguish "binding" from "expectation" for prioritisation purposes.
3. **New business lines import new instrument stacks.** DPT custody is not "banking with tokens" — it is a new licence perimeter (FSMA Part 9), a new AML notice (PSN02), and a new tech-risk pair (FSM-N30/N31), each with its own go-live gate.
4. **Enforcement targets individuals.** The prohibition-order record (§6) is the strongest argument for the IAC-style accountability register and the fit-and-proper screening loop: the bank's controls protect its *people* as much as its licence.

---

## 9. The Summary — "The Rules of the House"

MAS regulates through a deliberately shaped house: **Acts** are the walls (the Banking Act 1970, the SFA 2001, the PS Act 2019, the FSMA 2022 — verified years), **notices** are the wiring (Notice 626 for AML, 649 for liquidity, 658 for outsourcing, the FSM-series for technology risk — every one verified in this pass), **guidelines** are the expected standards of behaviour inside (TRMG 2021, BCM 2022, Outsourcing 2023 — verified — and IAC 2020, FEAT 2018, Model AI Governance — flagged to the repo guides that carry them), and **enforcement** is the inspection (prohibition orders, compositions, licence actions — verified in the current record). The rules of the house can be stated in five lines:

1. **The licence is a privilege, not a right** — fit-and-proper, banking secrecy, and conduct standards are enforced against *individuals* as much as institutions.
2. **AML/CFT is the floor** — Notice 626 and its sector siblings are the most audited documents in the house; a six-figure composition (Padang Trust) is the going rate for falling below the floor.
3. **Technology risk is board business** — since May 2024 the TRM notices are FSMA instruments (FSM-N05 for banks) sitting on the TRMG; since February 2026 every reportable incident goes to MAS on MAS-Tx.
4. **Guidelines are expectations with consequences** — BCM, Outsourcing, IAC, and FEAT don't carry criminal penalties; they carry supervisory risk ratings, and that is enough.
5. **The house is always being extended** — FSMA Phase 3 (June 2025) reached DPT providers abroad, the stablecoin framework (August 2023) set the reserve-and-redemption standard, and the AIRM guidelines (proposed November 2025) will formalise AI risk management — a bank's compliance map is a living document.

That, in the corpus's own words, is **"the rules of the house"**: know which act licenses you, which notice binds each activity, which guideline the supervisor will hold against you, and who, by name, is accountable when it fails.

---

## 10. Glossary

| Term | Meaning |
|---|---|
| **MAS** | Monetary Authority of Singapore — central bank + integrated financial regulator + financial-centre developer (MAS Act 1970, operating from 1 Jan 1971) |
| **FSMA** | Financial Services and Markets Act 2022 — the omnibus Act (AML powers, tech-risk notices, prohibition orders, DPT providers) |
| **Banking Act 1970** | The bank-licensing and regulation statute |
| **SFA 2001** | Securities and Futures Act — capital-markets activities and market conduct |
| **PS Act 2019** | Payment Services Act — activity-based payments licensing (SPI/MPI/money-changing) |
| **FAA 2001** | Financial Advisers Act ("FTA" reading ⚠) |
| **TCA 2005** | Trust Companies Act ("TPA" reading ⚠) |
| **Notice** | Legally binding instrument addressed to a class of FIs |
| **Directive** | Legally binding instrument addressed to an individual FI (merchant-bank directives ≈ notices) |
| **Guideline** | Best-practice standard; non-criminal, but feeds the supervisory risk assessment |
| **TRMG** | Technology Risk Management Guidelines (2021 revision; 2013 original ⚠) |
| **BCM** | Business Continuity Management Guidelines (June 2022) |
| **FSM-N05** | The FSMA notice on technology risk management for banks (10 May 2024) |
| **FSM-N30 / FSM-N31** | FSMA tech-risk and cyber-hygiene notices for digital token service providers (30 Jun 2025) |
| **PSN01 / PSN02** | PS Act AML notices — specified payment services / DPT services |
| **Notice 626** | The banks' AML/CFT notice |
| **Notice 649** | Minimum Liquid Assets and Liquidity Coverage Ratio (banks) |
| **Notice 658 / 1121** | Outsourcing notices for banks / merchant banks |
| **Notice 1014** | The current merchant-bank AML notice (legacy 730 ⚠ predecessor) |
| **IAC** | Guidelines on Individual Accountability and Conduct (2020) ⚠ |
| **FEAT** | Fairness, Ethics, Accountability, Transparency principles (2018) ⚠ |
| **AIRM** | Proposed Guidelines on AI Risk Management (consultation Nov 2025) ⚠ |
| **SCS** | MAS-regulated stablecoin framework (Aug 2023) ⚠ — 100% reserves, 5-day redemption, monthly attestation |
| **DPT** | Digital payment token; the PS Act / FSMA-regulated token service |
| **MPI / SPI** | Major / Standard Payment Institution (PS Act licence tiers) |
| **D-SIB / D-SII** | Domestic systemically important bank / insurer designations |
| **MAS-Tx** | MAS's FI Transactions Platform — incident-reporting submission channel from 1 Feb 2026 |
| **STR** | Suspicious transaction report |
| **MLRO** | Money laundering reporting officer |
| **LCR / NSFR** | Liquidity coverage ratio / net stable funding ratio (Basel III) |
| **MLA** | Minimum liquid assets (Singapore-dollar requirement) |
| **TPRM** | Third-party risk management |
| **PO** | Prohibition order (FSMA Part 3 since 31 Jul 2024) |
| **Composition** | Administrative penalty paid in lieu of prosecution |

---

## 11. Claims Status and Verification Notes

**What the /tmp research verified (✅) — the primary-source ledger:**

| Claim | Status | Source file |
|---|---|---|
| MAS Act 1970; MAS formed 1 Jan 1971; banker/financial-agent powers; monetary stability mandate | ✅ | mas_acts_masact.txt |
| Banking Act 1970 — governs licensing/regulation of banks, merchant banks, card issuers | ✅ | mas_banking_act.txt |
| SFA 2001, PS Act 2019 (passed 14 Jan 2019), Insurance Act 1966, FAA 2001, FCA 1967, TCA 2005, TSFA 2002, DIPOPS 2011, PSS(F&N) 2002, BTA 2004, CTA 1992, FHCA 2013, BoE Act, E(D&M) 1999 — years and mandates | ✅ | mas_catalog.json (18 acts, mas_analyze.py run) |
| FSMA 2022 — passed 5 Apr 2022; Phases 1 (28 Apr 2023), 2A (10 May 2024), 2B (31 Jul 2024), 3 (30 Jun 2025); Part 3 POs; Part 5 tech risk; Part 9 DTSPs | ✅ | fsma.txt |
| Eight instrument types (Acts → Policy Statements); directives-to-merchant-banks ≈ notices; guidelines feed risk assessment | ✅ | mas_super.txt |
| D-SIB list (7 banks) and D-SII list (5 insurers); FIRA monographs | ✅ | mas_super.txt |
| MAS triple role (central bank / integrated supervisor / developer) | ✅ | mas_wwd.txt, mas_who.txt |
| Notice 626/626A/824/1014/314/TCA-N03/SFA 04-N02/FAA-N06/PSN01/PSN02/PSN10 — 30 Jun 2025 AML revision wave | ✅ | mas_catalog.json |
| Notice 649/1015/FHC-N649 (26 May 2026), 651/652/653 (2024), 613 cancelled | ✅ | mas_catalog.json |
| Notice 658/1121 (11 Dec 2023) + Outsourcing Guidelines (effective 11 Dec 2024) + Annex D + FAQ (11 Dec 2024) | ✅ | g_guidelines-on-outsourcing-banks.txt, mas_catalog.json |
| TRM notices cancelled 10 May 2024 (644/644A/830/127/506/912/1114/PSN05/PSN06/CMG-N02/CMG-N03/FAA-N18/TCA-N05/834/FHC-1119); FSM-N05/N07/N09/N17/N19/N21/N22/N25 replacements | ✅ | g_technology-risk-management-guidelines.txt, circ.txt |
| TRMG published 18 Jan 2021; governance + IT/cyber resilience; ~60 applies-to types | ✅ | g_technology-risk-management-guidelines.txt |
| BCM Guidelines 6 Jun 2022; supersedes 2003 version + 2006 circular; 12-month audit plan; 24-month first audit | ✅ | g_guidelines-on-business-continuity-management.txt |
| FSM-N30/N31 (30 May 2025, effective 30 Jun 2025); FSM-N27 (30 May 2025) | ✅ | fsm30.txt, fsm31.txt, mas_catalog.json |
| Circular on FI Incident Reporting (16 Dec 2025); MAS-Tx + updated template from 1 Feb 2026 | ✅ | circ.txt |
| Anti-scam circular for MPI e-wallets (25 Oct 2024); E-Payments User Protection Guidelines reference | ✅ | psa.txt |
| Enforcement page mechanics (5-year publication; POs until out of force; 1 Apr 2021 <5yr PO rule; fee-based background check) | ✅ | enf.txt |
| Recent enforcement actions (10 actions, May–Aug 2026, incl. 4 POs, FSMA prosecution, S$300k composition, MPI revocation) | ✅ | enf.txt |
| Merchant-bank notices 1003/1005/1007/1002/1013/1011/1108/1112/1121; Notice 1112 under MAS Act s 28 | ✅ | mas_acts_masact.txt |
| PS Act expanded scope appointed day 4 Apr 2024; PSN-series list (PSN01–PSN11, PSN01A/AA) | ✅ | psa.txt, mas_catalog.json |
| CMG-N01 (20 Aug 2024) fraud/STR reporting; SFA 04-N19/N20; FMI standards notices | ✅ | mas_catalog.json |
| Insurance notices 100–300 series (314 AML, 126 ERM, 127 cancelled, 125, 302, 307, 320, 321, 120, 211–213, 129–131, 502, 508) | ✅ | mas_catalog.json |

**What is flagged (⚠) — not confirmable from the /tmp research:**

| Claim | Why flagged | Where it lives instead |
|---|---|---|
| **"Notice 730" (payments/merchant-bank AML, 700-series)** | Not present anywhere in mas_catalog.json or the extracts; two web searches returned empty; the current merchant-bank AML notice is 1014 (2022) | Treat as legacy MAS-Act merchant-bank AML notice, superseded by 1014; do not cite "730" as live |
| **"CM 800-series" = capital-markets notices** | The 800-series in the catalogue is finance-company/legacy (806/824/830/834); capital-markets notices are SFA 04-/CMG-/FAA-N series | See §3.5 notice map |
| **"Insurance 1000-series"** | The 1000-series is the merchant-bank family; insurance notices are 100–300 series | See §3.6 |
| **The 54 prohibition orders count** | The count is attributed to the prior pass's enforcement-API run; no dump survives on disk (only the query script enf.py and the first page of 14) | Re-run enf.py against the MAS search API, or filter the live Enforcement Actions page |
| **TRMG "2013" original** | Extract confirms only the 18 Jan 2021 publication | Widely documented; verify against MAS |
| **IAC 2020** | No IAC document in the /tmp files | ai_genai_banking_compliance_guide.md; verify against MAS |
| **FEAT 2018** | No FEAT document in the /tmp files | ../technology/ai_trust_assessments_guide.md, ../technology/responsible_ai_frameworks_guide.md |
| **Model AI Governance Framework / AI Verify / GenAI framework (2024) / AIRM (Nov 2025 proposed)** | No files in /tmp | ai_genai_banking_compliance_guide.md, ../technology/ai_trust_assessments_guide.md |
| **Stablecoin framework (Aug 2023), SCS requirements, "Stablecoin Issuance Service" (2026)** | No files in /tmp | tokenized_assets_guide.md §2.4/§13 |
| **DPT measures (MPI count ~30+, Oct 2022 full regime, Binance rejection, retail guardrails)** | PS Act mechanics verified ✅ but market-level facts ⚠ | tokenized_assets_guide.md §13.2 |
| **Climate / ENR Guidelines** | No files in /tmp | Repo sustainable-finance cluster (verify against MAS) |
| **"FTA" / "TPA" abbreviations** | Not in the catalogue under those letters | Read as FAA 2001 / TCA 2005 (both verified ✅) |
| **Notice 637 (bank TRM)** | Catalogue shows 644/644A cancelled → FSM-N05; "637" appears only in the repo cross-ref | banks_in_singapore_guide.md §5 |

**Honesty note:** nothing in this guide was invented. Every ✅ row traces to a named file on disk from the prior research pass (mas.gov.sg extracts and the catalogue JSON); every ⚠ row is explicitly flagged with its alternative source. The two web searches budgeted for gap-filling (Notice 730) returned empty results and the gap was left flagged rather than guessed. The worked example (§8) is design/analytical content in the repo's established convention, built on verified instrument names.

---

### 11.1 The Research Trail (what the prior pass left on disk)

For reproducibility, the /tmp research files this guide was written from (all read in this pass):

- **MAS pages:** mas_about.txt · mas_wwd.txt (What We Do) · mas_who.txt · mas_reg.txt (Regulation index) · mas_super.txt (Supervisory Approach & Regulatory Instruments — the instrument pyramid) · mas_acts.txt · mas_acts_masact.txt (MAS Act 1970) · mas_bank.txt (Banking hub) · mas_banking_act.txt (Banking Act 1970) · psa.txt (Payment Services Act 2019) · fsma.txt (FSMA 2022 phases) · enf.txt (Enforcement Actions) · fsm30.txt (FSM-N30) · fsm31.txt (FSM-N31) · circ.txt (Incident Reporting Circular + the FSM notice family) · n9.txt (FSM-N09, the notice header format "Issued pursuant to")
- **Guideline pages:** g_technology-risk-management-guidelines.txt · g_guidelines-on-business-continuity-management.txt · g_guidelines-on-outsourcing-banks.txt
- **Data and scripts:** mas_catalog.json (the 18 acts + full notices/guidelines catalogue, queried via mas_analyze.py) · enf.py (the enforcement-API query) · mas_api.py · find_fsma.py · find_fsm2.py

**Re-verification is cheap:** re-run `python3 mas_analyze.py acts` for the statute list, `python3 mas_analyze.py notices <keyword>` for any notice family, and `python3 enf.py` for the live enforcement feed — all against the on-disk catalogue and the MAS search API.

---

## 12. Cross-References and Further Reading

**The banking cluster (sibling guides — the depth behind the map):**

- [RegTech](regtech_guide.md) — the AML/CFT and reporting technology layer that implements Notice 626 and the returns notices (its §4 AML stack, §6 the Singapore layer)
- [Banks in Singapore](banks_in_singapore_guide.md) — the market and licence context (its §5 the MAS framework, D-SIBs, TRM post-2023)
- [AI/GenAI Banking Compliance](ai_genai_banking_compliance_guide.md) — FEAT, the AI model-risk information paper (2024), the proposed AIRM guidelines (2025), and the requirements map for AI use cases
- [Tokenized Assets](tokenized_assets_guide.md) — the stablecoin framework (SCS, Aug 2023), the PS Act DPT licensing regime (§13.2), safeguarding and no-lending rules
- [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) — the in-house risk/compliance systems landscape (AML monitoring, regulatory reporting)
- [Online Investment Trading Platforms](online_investment_trading_platforms_guide.md) — the SFA angle for trading platforms (and the 2FA expectations on the TRM page)
- [Payment Rails](payment_rails_guide.md) — the PS Act and the payments infrastructure (PayNow, FAST)

**The technology cluster (`../technology/` prefix):**

- [SecOps](../technology/secops_guide.md) and [Cybersecurity](../technology/cybersecurity_guide.md) — the TRMG/FSM-N05 control implementations (security operations, cyber resilience)
- [AI Trust Assessments](../technology/ai_trust_assessments_guide.md) — the FEAT / Model AI Governance / AI Verify facts (the "Verifiable AI" strand)

**Primary sources to verify the ⚠ items:** mas.gov.sg (Enforcement Actions search — re-run the PO count; the IAC guidelines; FEAT; the ENR Guidelines; the TRMG 2013 history; the SCS stablecoin page), the FSMA legislation on Singapore Statutes Online, and the repo guides cited above.

**Reading path:** start with this guide's map (§1–§5) to place the instruments, read [Banks in Singapore](banks_in_singapore_guide.md) §5 for the licence context, then [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) and [RegTech](regtech_guide.md) for the systems, [AI/GenAI Banking Compliance](ai_genai_banking_compliance_guide.md) for the AI layer, and [Tokenized Assets](tokenized_assets_guide.md) for the digital-asset horizon — then bring it home with the Cymbal Bank matrix (§8).

---

*End of guide. Companion reading: [Banks in Singapore](banks_in_singapore_guide.md) (the market), [RegTech](regtech_guide.md) (the tools), [AI/GenAI Banking Compliance](ai_genai_banking_compliance_guide.md) (the AI layer), [Tokenized Assets](tokenized_assets_guide.md) (the digital-asset horizon), [../technology/ai_trust_assessments_guide.md](../technology/ai_trust_assessments_guide.md) (FEAT/Model-AI-Governance facts). The rules of the house: know the Act that licenses you, the Notice that binds each activity, the Guideline the supervisor will hold against you, and the name that answers for it.*
