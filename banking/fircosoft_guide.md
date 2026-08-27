# Fircosoft — The Sanctions-Screening Vendor: A Deep-Dive Guide

**The Watch-List Filtering Franchise — Founding (Paris, 1990), the Reed Elsevier / LexisNexis Risk Solutions Ownership, the Firco Product Family (FircoTrust, Firco Filtering Suite, Firco Continuity, Firco Compliance Link), the Name-Matching Engine, the Sanctions Regimes It Screens Against (OFAC / UN / EU / UK / MAS), the Screening Process, the Competitors, the Cymbal Bank Screening Estate, and the Worked Example**

> **Author:** Jack Liu Shurui, Solution Architect at Cymbal Bank
> **Context:** Banking Domain / Risk & Compliance — a vendor deep-dive on Fircosoft (FircoSoft), the Paris-founded sanctions- and watch-list-screening software company: the Company Profile (founding, ownership, product portfolio), the Screening Capabilities (name matching, list management, transaction filtering, workflow/case management), the Sanctions Regime (OFAC, UN, EU, UK, MAS/ABS), the Screening Process (onboarding, transaction, name-screening workflow), the Competitors (World-Check/LSEG, Dow Jones Risk & Compliance, LexisNexis Bridger, NICE Actimize, BAE NetReveal, Fenergo), the Banking Context (the Cymbal Bank sanctions-screening estate), the Worked Example (a Cymbal Bank screening design with false-positive management and metrics), the Summary, the Glossary, the Claims Audit, What Could Not Be Verified, and the Verification Ledger
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** this pass's live web verification (the RELX press release of 29 September 2014 announcing the completed Reed Elsevier acquisition of FircoSoft — which also carries FircoSoft's own "About" facts: founded 1990, 700+ customers, eight of the world's top 10 financial institutions, offices in eleven cities, 2,000 customer sites in more than 85 countries; the LexisNexis Risk Solutions Financial Crime Compliance hub now served at fircosoft.com — 45 of the world's top 50 banks, WorldCompliance Data, the Firco-branded product links; the official product-page search snippets for Firco™ Continuity and Firco Compliance Link; the UN Security Council sanctions page — 31 regimes since 1966, 15 currently active, Article 41 / Chapter VII; the OFAC sanctions list site (title-level); the EU Sanctions Map (title-level); the SWIFT "Guiding principles for screening ISO 20022 payments" resource; the industry fuzzy-matching literature) plus the sibling repo guides whose verified facts are cross-referenced rather than re-derived (regtech_guide.md, mas_regulations_guidelines_guide.md, financial_fraud_detection_at_scale_guide.md, financial_risk_compliance_systems_guide.md, payment_rails_guide.md, banks_in_singapore_guide.md, dbs_software_systems_guide.md, standard_chartered_guide.md). Facts verified in this pass are marked ✅; facts that could not be confirmed are flagged ⚠ and, where possible, cross-referenced to the sibling guide that carries them; the one task-hypothesis that the evidence contradicts (the "FICO acquisition") is recorded ❌ in the claims audit.
> **Last Updated:** August 2026
> **Companion guides (sibling, same folder):** [RegTech](regtech_guide.md) (the AML-KYC stack this vendor sells into — §4) · [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) (the sanctions-screening vendor table — §7) · [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) (Notice 626, the MAS designated-lists link) · [Banks in Singapore](banks_in_singapore_guide.md) (the market and the ABS) · [Payment Rails](payment_rails_guide.md) (the SWIFT / ISO 20022 screening overlay) · [Financial Fraud Detection at Scale](financial_fraud_detection_at_scale_guide.md) (metrics, case management) · [AI/GenAI Banking Compliance](ai_genai_banking_compliance_guide.md) (the AI layer over screening) · [DBS Software Systems](dbs_software_systems_guide.md) and [Standard Chartered](standard_chartered_guide.md) (the "Firco-class screening is industry-standard" inferences) · [Asset Management & Alternatives](asset_management_alternatives_guide.md) (the Fircosoft / World-Check / Dow Jones vendor mention)
> **Companion guides (technology/, prefix `../technology/`):** [Event Stream Processing](../technology/event_stream_processing_guide.md) (the streaming substrate for real-time transaction screening)

---

## Table of Contents

1. [The Overview](#1-the-overview)
   - 1.1 [The Sanctions-Screening Discipline](#11-the-sanctions-screening-discipline)
   - 1.2 [Fircosoft's Positioning](#12-fircosofts-positioning)
   - 1.3 [The Overview Table](#13-the-overview-table)
2. [The Company Profile](#2-the-company-profile)
   - 2.1 [Founding — 1990, Paris](#21-founding--1990-paris)
   - 2.2 [Ownership — Keensight Capital, then Reed Elsevier / RELX](#22-ownership--keensight-capital-then-reed-elsevier--relx)
   - 2.3 [The "FICO Acquisition" Question](#23-the-fico-acquisition-question)
   - 2.4 [The Product Portfolio](#24-the-product-portfolio)
   - 2.5 [Scale and Footprint](#25-scale-and-footprint)
   - 2.6 [The Company Table](#26-the-company-table)
   - 2.7 [The 2014 Deal — Rationale and Integration](#27-the-2014-deal--rationale-and-integration)
3. [The Screening Capabilities](#3-the-screening-capabilities)
   - 3.1 [Name Matching — the Algorithmic Heart](#31-name-matching--the-algorithmic-heart)
   - 3.2 [List Management](#32-list-management)
   - 3.3 [Transaction Filtering — SWIFT and ISO 20022](#33-transaction-filtering--swift-and-iso-20022)
   - 3.4 [Workflow and Case Management](#34-workflow-and-case-management)
   - 3.5 [The Capabilities Table](#35-the-capabilities-table)
4. [The Sanctions Regime](#4-the-sanctions-regime)
   - 4.1 [OFAC — the US Layer](#41-ofac--the-us-layer)
   - 4.2 [The UN Security Council](#42-the-un-security-council)
   - 4.3 [The EU](#43-the-eu)
   - 4.4 [The UK — OFSI](#44-the-uk--ofsi)
   - 4.5 [Singapore — MAS and the ABS](#45-singapore--mas-and-the-abs)
   - 4.6 [The Regime Table](#46-the-regime-table)
   - 4.7 [The Enforcement Stakes — Why Screening Fails](#47-the-enforcement-stakes--why-screening-fails)
5. [The Screening Process](#5-the-screening-process)
   - 5.1 [Client Onboarding Screening](#51-client-onboarding-screening)
   - 5.2 [Transaction Screening](#52-transaction-screening)
   - 5.3 [The Name-Screening Workflow — Exact vs Fuzzy](#53-the-name-screening-workflow--exact-vs-fuzzy)
   - 5.4 [False Positives — the Economic Engine of the Market](#54-false-positives--the-economic-engine-of-the-market)
   - 5.5 [The Process Table](#55-the-process-table)
6. [The Competitors](#6-the-competitors)
   - 6.1 [The Screening-Data and Screening-Engine Field](#61-the-screening-data-and-screening-engine-field)
   - 6.2 [The Comparison Table](#62-the-comparison-table)
7. [The Banking Context — Cymbal Bank](#7-the-banking-context--cymbal-bank)
   - 7.1 [The Obligations](#71-the-obligations)
   - 7.2 [The Screening Estate](#72-the-screening-estate)
   - 7.3 [The Estate Table](#73-the-estate-table)
8. [The Worked Example — A Cymbal Bank Sanctions-Screening Design](#8-the-worked-example--a-cymbal-bank-sanctions-screening-design)
   - 8.1 [The Scenario](#81-the-scenario)
   - 8.2 [The Screening Workflow](#82-the-screening-workflow)
   - 8.3 [False-Positive Management](#83-false-positive-management)
   - 8.4 [The Metrics](#84-the-metrics)
   - 8.5 [The Lessons](#85-the-lessons)
9. [The Summary — "The Clean List"](#9-the-summary--the-clean-list)
10. [Glossary](#10-glossary)
11. [Claims Audit — ✅ Verified / ⚠ Partially Verified / ❌ Not Verified](#11-claims-audit--verified--partially-verified--not-verified)
12. [What Could Not Be Verified](#12-what-could-not-be-verified)
13. [Verification Ledger](#13-verification-ledger)
14. [References and Further Reading](#14-references-and-further-reading)

---

## 1. The Overview

### 1.1 The Sanctions-Screening Discipline

Sanctions screening is the compliance control that checks **names** — customers, counterparties, beneficiaries, vessels, goods, and the parties to every payment — against the **sanctions and watch lists** that jurisdictions publish, and stops or blocks the transactions that hit. In the four-layer AML-KYC stack that the sibling [RegTech guide](regtech_guide.md) §4.1 lays out — **CDD → Screening → Transaction Monitoring → Sanctions Compliance**, with case management as the connective fifth layer — sanctions screening spans the *screening* layer (detection: "does this name appear on a list?") and the *sanctions compliance* layer (prevention: "this payment must not go through"). The RegTech guide's mental model is worth repeating here: *CDD decides who the customer is, screening checks who they claim to be against watchlists, transaction monitoring watches what they do, and sanctions compliance governs who they may not touch* (regtech_guide.md §4.1).

Three properties define the discipline for an architect:

1. **It is a preventive control on the payment path, not a post-hoc report.** Sanctions screening runs *before* a payment is released — in real time where the rail allows. The asymmetry is brutal: a false negative (a sanctioned payment released) is catastrophic — fines, licence conditions, monitorships, reputational collapse; a false positive (a legitimate payment stopped for review) is merely expensive — an analyst's time and a delayed customer payment. RegTech guide §4.5 puts it exactly: *"it is the layer where false negatives are catastrophic and false positives are merely expensive."*
2. **It is a name-matching problem before it is anything else.** Lists are written in one script and spelling; customer data arrives in another, with typos, abbreviations, and transliteration drift ("Muhammed" vs "Mohammed"). Exact matching fails; the discipline exists because of *fuzzy* matching — string-distance measures, phonetic algorithms, and transliteration handling (regtech_guide.md §4.3 and §7.3).
3. **It is a data-currency problem.** Lists change continuously — new designations, delistings, amended spellings — and the screening engine must re-run against the updated list on the same day the regulator publishes it. "Latency + list management" is how the RegTech guide characterises the screening layer (regtech_guide.md §4.3).

### 1.2 Fircosoft's Positioning

Fircosoft (FircoSoft) is the **watch-list filtering specialist** that became the sanctions-screening industry standard for large banks. In the words of its acquirer's own 2014 announcement: *"FircoSoft is the recognised leader worldwide of watch list filtering solutions"* (✅ RELX press release, 29 September 2014). The same release records the flagship fact that explains the franchise: **over 700 customers including eight of the world's top 10 financial institutions** relied on FircoSoft to *"filter customers and transactions against sanctions and watch lists to ensure compliance with regulations on terrorist financing and sanctions programs, and meet Know Your Customer requirements"* (✅). A decade later, the LexisNexis Risk Solutions hub served at fircosoft.com claims **45 of the world's top 50 banks** (⚠ vendor claim — see §2.5).

The repo's sibling guides already encode Firco's market position, flagged where inferred: the DBS guide calls it *"the industry-standard sanctions-screening engine (Firco, now LexisNexis Risk Solutions)"* (dbs_software_systems_guide.md — ⚠ inferred there); the Standard Chartered guide calls it *"the industry-standard engine... used by the vast majority of global banks for name/sanctions screening"* (standard_chartered_guide.md §7 — ⚠ flagged there); and the Financial Risk & Compliance Systems guide lists it among the sanctions-screening vendors a bank's estate runs (financial_risk_compliance_systems_guide.md — World-Check/OneSumX, LexisNexis Bridger, Fircosoft, Accuity, SAS Sanctions).

The positioning, in one line: **Firco is the name-matching engine and the transaction-filtering engine that sits *inside* the compliance stack — the screening specialist whose brand is precision, list coverage, and throughput, now owned and data-enriched by LexisNexis Risk Solutions.**

### 1.3 The Overview Table

| Aspect | Description |
|---|---|
| **Discipline** | Sanctions screening: checking names (customers, counterparties, payment parties) against sanctions/watch lists; a preventive control on the payment path (regtech_guide.md §4.3/§4.5) |
| **The vendor** | Fircosoft (FircoSoft) — Paris-founded (1990) watch-list filtering software company ✅; the "recognised leader worldwide of watch list filtering solutions" per its acquirer's 2014 announcement (⚠ vendor-flavoured claim) |
| **Core business** | Name screening (customers) + transaction filtering (payments) against sanctions, PEP and watch lists, with list-management and workflow capability |
| **Ownership today** | LexisNexis Risk Solutions, a RELX business — acquired via Reed Elsevier, completed 29 September 2014 ✅ (the "FICO acquisition" hypothesis is ❌ contradicted — see §2.3) |
| **Product family** | Firco Filtering Suite ⚠, FircoTrust ⚠, Firco™ Continuity ✅, Firco Compliance Link ✅, WorldCompliance™ Data ✅ (see §2.4) |
| **Regimes screened** | OFAC (SDN/consolidated lists) ⚠, UN Security Council consolidated list ✅, EU sanctions map ⚠, UK OFSI ⚠, MAS designated lists ✅ (sibling-verified), plus PEP/adverse-media watch lists |
| **The buying case** | Precision (fewer false positives) at volume; real-time transaction screening; list-change currency; the LexisNexis data layer (WorldCompliance) on top of the Firco engine |

---

## 2. The Company Profile

### 2.1 Founding — 1990, Paris

**Founded 1990, Paris** — ✅ verified in this pass by two independent sources. The RELX press release's own "About FircoSoft" boilerplate states: *"Founded in 1990, FircoSoft has a track record of partnering closely with its customers to keep ahead of regulatory changes and eliminate the risk of fines and reputational exposure"* (✅ relx.com, 29 September 2014). The Tracxn company profile independently records: *"FircoSoft is an acquired company based in Paris (France), founded in 1990"* (✅ tracxn.com). Every acquisition-announcement source found repeats the same descriptor: **"the Paris-based global sanctions screening software group"** (✅ relx.com, librarytechnology.org, tmcnet.com — three independent reprints of the 3 September / 29 September 2014 announcement).

| Founding fact | Value | Status |
|---|---|---|
| Year | 1990 | ✅ RELX press release + Tracxn |
| City | Paris, France | ✅ "Paris-based" in RELX/librarytechnology/tmcnet announcements |
| Original business | Watch-list filtering software for financial institutions | ✅ Tracxn ("watch list filtering solutions"); RELX ("filter customers and transactions against sanctions and watch lists") |
| First customer vintage | Pre-2014 franchise with 700+ customers at acquisition | ✅ RELX 2014 release |

### 2.2 Ownership — Keensight Capital, then Reed Elsevier / RELX

The ownership arc is fully documented in the 2014 press record:

1. **Private-equity era.** Before the 2014 sale, FircoSoft was backed by **Keensight Capital**, the European growth private-equity firm — the RELX announcement carries a full "About Keensight Capital" boilerplate (✅ the boilerplate's presence in the seller-side announcement confirms Keensight was the selling shareholder; the *extent* of its stake is not stated — ⚠ detail unverified).
2. **The 2014 acquisition.** On **3 September 2014** Reed Elsevier announced it had entered **exclusive negotiations** to acquire FircoSoft (✅ librarytechnology.org reprint of the announcement). On **29 September 2014** Reed Elsevier announced the acquisition was **completed** (✅ relx.com press release: *"Reed Elsevier announced today it has completed the acquisition of FircoSoft, the Paris-based global sanctions screening software group"*).
3. **Today.** FircoSoft sits inside **LexisNexis Risk Solutions**, the risk-information business of **RELX Group** (the renamed Reed Elsevier). Two pieces of evidence: fircosoft.com now resolves to the LexisNexis Risk Solutions *Financial Crime Compliance* hub (✅ extracted this pass — the page is branded "LexisNexis® Risk Solutions" and cross-links risk.lexisnexis.com products), and the current Firco products are marketed as "Firco™" on risk.lexisnexis.com (✅ official product URLs found via search). The sibling DBS guide records the same lineage: *"Fircosoft (Firco, now LexisNexis Risk Solutions)"* (dbs_software_systems_guide.md).

**Acquisition terms (price, deal structure) were not disclosed in the sources retrieved — ⚠ flagged; not fabricated.**

### 2.3 The "FICO Acquisition" Question

The task hypothesis to verify was: *"the FICO acquisition is believed to be 2014."* **The evidence contradicts it.** Every 2014 source retrieved names **Reed Elsevier** (now RELX) as the acquirer — none names FICO (Fair Isaac). The verified facts:

- 2014 ✅ — the year is right.
- Acquirer ❌ — it was **Reed Elsevier / RELX (LexisNexis Risk Solutions)**, not FICO. FICO is the analytics company best known for Falcon fraud scoring; no retrieved source connects FICO to the FircoSoft acquisition, and no retrieved source shows FICO owning the Firco brand. The claims audit records this row ❌ (task hypothesis contradicted by sources), with the corrected owner in the ✅ rows above.
- The practical consequence for a bank's procurement notes: **FircoSoft is a LexisNexis Risk Solutions product**, so any "FICO Firco" naming in internal decks is wrong and should be corrected to "LexisNexis Risk Solutions (RELX) — Firco".

### 2.4 The Product Portfolio

The task's assumed product names — "Sanctions Filter / Sanctions List Management / Transaction Filtering" — **could not be confirmed as official Firco product names in this pass** (⚠; no retrieved source uses those exact strings for FircoSoft). What *is* confirmed:

| Product name | Confirmed? | Evidence |
|---|---|---|
| **Firco Filtering Suite** | ⚠ partial | Named by third-party database (newtonconsultingpartners.com): "FircoSoft's watch list filtering solution, **Firco Filtering Suite**, helps financial institutions comply with reinforced international regulations on terrorist financing and embargoes" — third-party sourcing, not an official page this pass |
| **FircoTrust** | ⚠ partial | Official product URL exists (risk.lexisnexis.com/products/firco-trust) but the page blocked scraping (Internal Server Error) — product existence at URL level, content unverified |
| **Firco™ Continuity** | ✅ | Official product page URL + official search snippet: *"a leading transaction screening solution... a complete and scalable real-time transaction screening solution which enables businesses to ensure sanctions compliance and establish readiness for regulatory scrutiny"* (risk.lexisnexis.com/products/firco-continuity) |
| **Firco Compliance Link** | ✅ | Official product page URL + official search snippet: *"an all-in-one financial crime screening solution"* to *"meet sanctions and AML screening obligations"* (risk.lexisnexis.com/global/en/products/firco-compliance-link) |
| **WorldCompliance™ Data** | ✅ | Listed on the fircosoft.com / LNRS Financial Crime Compliance hub this pass: "comprehensive global risk intelligence... over **7 million structured profiles** of individuals and entities... more than **60 risk categories and subcategories**, including global sanctions, enforcement actions, PEPs, state-owned enterprises, registration lists and adverse media" |

The historic FircoSoft product family, as reconstructed from the 2014-era press record and the current official links (⚠ reconstruction — each name flagged individually in the audit): the **Firco Trust** name-screening line, the **Firco Filtering Suite** transaction line, and the **Firco Continuity** standby/real-time transaction-screening line, now joined by **Firco Compliance Link** for end-to-end screening workflow and the **WorldCompliance** data layer. The *functional* split — name screening vs transaction filtering vs list management vs case workflow — is verified at the industry level (regtech_guide.md §4.1: the four-layer stack plus case management) even where the exact Firco brand for each box is ⚠.

### 2.5 Scale and Footprint

- **2014 (at acquisition, ✅ RELX release):** over **700 customers**, including **eight of the world's top 10 financial institutions**; **2,000 customer sites in more than 85 countries**; offices in **New York, Tampa, São Paulo, London, Paris, Luxembourg, Zurich, Pretoria, Chennai, Singapore and Melbourne**; delivered with a global partner network.
- **2026 (current, ⚠ vendor claim):** the LNRS Financial Crime Compliance hub claims **"45 of the world's top 50 banks"** and **"over 20 years in the industry"** — the latter figure sits oddly next to a 1990 founding (36 years), so both figures are treated as loose marketing numbers ⚠.
- **CB Insights profile (⚠ third-party database):** repeats the "over 700 customers including eight of the world's top 10 financial institutions" line (consistent with the 2014 RELX release — the figure is the 2014-era one).

### 2.6 The Company Table

| Attribute | Value | Status |
|---|---|---|
| Legal name | Fircosoft / FircoSoft | ✅ |
| Founded | 1990 | ✅ RELX + Tracxn |
| HQ city | Paris, France | ✅ acquisition announcements |
| Pre-2014 owner | Keensight Capital (European growth PE) | ✅ seller-side boilerplate in RELX release; stake extent ⚠ |
| Acquirer (2014) | Reed Elsevier (RELX Group) — completed 29 September 2014; exclusive negotiations announced 3 September 2014 | ✅ relx.com + reprints |
| Owner today | LexisNexis Risk Solutions (RELX) — the Firco brand and the fircosoft.com domain live under LNRS | ✅ fircosoft.com redirect + official Firco product URLs |
| "FICO acquisition" hypothesis | Contradicted — the 2014 acquirer was Reed Elsevier, not FICO | ❌ |
| Customers (2014) | 700+, incl. 8 of the world's top 10 FIs; 2,000 sites in 85+ countries | ✅ RELX release |
| Customers (2026 claim) | 45 of the world's top 50 banks | ⚠ vendor claim |
| Offices (2014) | New York, Tampa, São Paulo, London, Paris, Luxembourg, Zurich, Pretoria, Chennai, Singapore, Melbourne | ✅ RELX release |
| Products | Firco Filtering Suite ⚠ · FircoTrust ⚠ · Firco™ Continuity ✅ · Firco Compliance Link ✅ · WorldCompliance™ Data ✅ | see §2.4 |
| Revenue / deal price | Not disclosed in any retrieved source | ⚠ unverified |

### 2.7 The 2014 Deal — Rationale and Integration

The full text of the 29 September 2014 release (read from the cached extraction this pass) adds the deal's operational shape, all ✅:

- **FircoSoft became part of Accuity**, Reed Elsevier's *"leading provider of global payment routing data and anti-money laundering solutions to banks and businesses worldwide"* — the acquisition's stated purpose was to *"extend Accuity's portfolio of next-generation products and solutions for customers within the financial market."*
- **The complementarity rationale**, in the words of the two CEOs quoted in the release:
  - Hugh Jones, President and CEO of Accuity: *"FircoSoft and Accuity are two extremely strong and well-established brands in the risk and compliance sector. Bringing together these highly complementary businesses will accelerate growth, deliver the most innovative solutions to our customers and expand our reach and penetration into the global Anti-Money Laundering (AML), Know Your Customer (KYC) and regulatory compliance market."*
  - Jean Losco, CEO of FircoSoft: *"In the fast growing market of preventing global financial crime, joining a very large and powerful group will accelerate our future strategic growth... As part of the RBI and Accuity family we will continue to invest and deliver solutions..."* (RBI = Reed Business Information, the RELX division that then housed Accuity).
- **The 2014-era lineage:** FircoSoft → Accuity (2014) → today's LexisNexis Risk Solutions Financial Crime Compliance line, where the Firco brand and fircosoft.com now live (✅ fircosoft.com extraction). ⚠ **Accuity's own later corporate fate (post-2014 divestments/renaming) is not verified in this pass** and is not asserted here.

| Deal fact | Value | Status |
|---|---|---|
| Announced | Exclusive negotiations, 3 September 2014 | ✅ |
| Completed | 29 September 2014 | ✅ |
| Integration home | Accuity (Reed Elsevier's payment-routing/AML data business) | ✅ release text |
| FircoSoft CEO at deal | Jean Losco | ✅ release quote |
| Accuity CEO at deal | Hugh Jones | ✅ release quote |
| Rationale | Complementary risk/compliance brands; extend Accuity's portfolio into next-gen AML/KYC products | ✅ release text |
| Price / terms | Not disclosed | ⚠ unverified |
| Accuity's later fate | Not verified this pass | ⚠ unverified |

---

## 3. The Screening Capabilities

### 3.1 Name Matching — the Algorithmic Heart

Name matching is Firco's home turf: the business of deciding whether the "Muhammed Al-Sayed" in a payment instruction is the same person as the "Mohammed Elsayed" on a sanctions list. The industry-standard technique set — verified this pass across the fuzzy-matching literature — is:

- **String-distance measures** — Levenshtein edit distance (character edits), Jaro-Winkler (weighting matching prefixes), trigram similarity (ofacscreen.com, verifex.dev, axleruns.com snippets ✅).
- **Phonetic algorithms** — Soundex/Metaphone class: names that *sound* alike but are spelled differently (axleruns.com ✅; regtech_guide.md §7.3 calls these "the quiet NLP workhorse").
- **Transliteration handling** — the same name rendered from Arabic/Cyrillic/Chinese scripts in multiple Latin spellings; the aml-analytics.com guide records the regulatory stakes: **OFSI fined Bank of Scotland £160,000 after a sanctions screening system failed to detect transliteration variants of a designated Russian individual's name** (⚠ industry-press source, retrieved this pass).
- **Token/word-order tolerance** — "AL QAEDA" vs "Al-Qaeda", name-part reordering, honorifics and middle-name noise (verifex.dev ✅).

**Firco-specific matching detail is ⚠ not verified:** no official FircoSoft page retrieved in this pass describes its proprietary algorithm (the product pages blocked scraping). What the sibling repo already carries is the *market-level* claim that Firco's engine is the industry standard for this exact problem (dbs_software_systems_guide.md; standard_chartered_guide.md §7 — both ⚠-flagged there as inference). The honest statement: **Firco is understood to compete on matching precision and false-positive suppression, but its proprietary algorithm internals were not confirmable from retrieved sources** ⚠.

### 3.2 List Management

List management is the plumbing that makes screening correct: ingesting the regulators' lists (OFAC SDN/consolidated, UN consolidated, EU, UK OFSI, national lists), normalising them into the engine's matching format, propagating updates into the live screening configuration **the same day** lists change, and re-screening existing records against the deltas. The discipline-level facts are verified:

- The RegTech guide defines the screening layer as a *"latency + list-management problem — list updates must propagate in near-real-time, and the matching logic must be tunable per jurisdiction"* (regtech_guide.md §4.3 ✅ sibling).
- The Standard Chartered guide records the operating rhythm: *"Screening runs at onboarding, on payments (real-time), and on list changes"* (standard_chartered_guide.md §7 — ⚠ flagged there as inference from industry practice).
- The UN Security Council publishes a **Consolidated List** (✅ main.un.org — the "Consolidated List" link is on the official sanctions page); OFAC maintains the **Specially Designated Nationals and Blocked Persons List (SDN)** and consolidated lists (⚠ title/URL-level verified this pass; the list site is a JS app whose content did not render for extraction).

**Firco-specific list-management product branding (the assumed "Sanctions List Management" name) is ⚠ not confirmed** — the capability is verified at the industry level and the task-assumed product name is not evidenced.

### 3.3 Transaction Filtering — SWIFT and ISO 20022

Transaction filtering is Firco's second home turf: screening **every payment message** for list hits in the sender, beneficiary, ordering institution, intermediary banks, and free-text remittance fields, in real time or near-real time, and blocking or holding hits.

- **Verified product capability:** Firco™ Continuity — *"a complete and scalable real-time transaction screening solution which enables businesses to ensure sanctions compliance and establish readiness for regulatory scrutiny"* (✅ official risk.lexisnexis.com snippet retrieved this pass).
- **The message formats:** cross-border payments travel as SWIFT MT messages (MT103 customer transfer, MT202 cover) and increasingly as **ISO 20022** (pacs.008/pacs.009 under CBPR+). The SWIFT industry resource *"Guiding principles for screening ISO 20022 payments"* exists and is the industry's reference for screening the richer structured format (✅ swift.com resource found this pass). The sibling [Payment Rails guide](payment_rails_guide.md) carries the verified context: ISO 20022's structured addresses, purpose codes and rich creditor/debtor data are what *"enable... better sanctions screening"* (✅ payment_rails_guide.md — SWIFT: "more accurate compliance processes"), and CBPR+ migration requires banks to *"rework sanctions screening and reference data to consume the richer structured data"* (✅ payment_rails_guide.md).
- **Firco-specific SWIFT/ISO 20022 support is ⚠ not directly verified** — no Firco page retrieved this pass names SWIFT MT or ISO 20022 formats explicitly; the capability is inferred from the "transaction screening" positioning plus the industry context above. Flagged, not asserted.

### 3.4 Workflow and Case Management

Case management is the fifth, connective layer of the AML stack — investigate alerts, capture evidence, record dispositions, produce regulator-ready dossiers (regtech_guide.md §4.1/§4.6 ✅). For screening, the workflow is: alert generated → triage → investigation against source lists and customer records → disposition (false positive / hit confirmed) → escalation to the sanctions officer and, where relevant, asset-freeze and STR filing.

- **Verified product capability:** Firco Compliance Link — *"an all-in-one financial crime screening solution"* (✅ official snippet) — is the Firco-branded screening workflow layer.
- The case-management *pattern* (alert queue, investigation, disposition codes, audit trail) is verified at the industry level in the sibling guides: the RegTech guide's AML table lists case management as a layer with "workflow + evidence store; regulator-ready dossiers" (regtech_guide.md §4.6 ✅), and the fraud guide's case-management section details investigation records, outcomes, labels and audit trails (financial_fraud_detection_at_scale_guide.md §14 ✅).
- **Firco-specific case-management depth (beyond the Compliance Link positioning) ⚠ not verified.**

### 3.5 The Capabilities Table

| Capability | What it does | Verified status | Evidence / cross-ref |
|---|---|---|---|
| Name matching | Fuzzy/phonetic/transliteration matching of names against lists | Industry techniques ✅; Firco algorithm ⚠ | Fuzzy-matching literature (this pass); regtech_guide.md §4.3/§7.3 |
| List management | Ingest, normalise, propagate list updates; rescreen on change | Industry pattern ✅; Firco branding ⚠ | regtech_guide.md §4.3; standard_chartered_guide.md §7 |
| Transaction filtering | Real-time screening of payment messages; block/hold hits | Firco™ Continuity ✅; SWIFT/ISO 20022 format support ⚠ | risk.lexisnexis.com snippet; payment_rails_guide.md; SWIFT screening principles doc |
| Workflow / case management | Alert triage, investigation, disposition, audit trail | Firco Compliance Link ✅; depth ⚠ | risk.lexisnexis.com snippet; regtech_guide.md §4.6; financial_fraud_detection_at_scale_guide.md §14 |
| Data layer | PEP, sanctions, adverse-media risk intelligence | WorldCompliance™ Data ✅ (7M+ profiles, 60+ risk categories — vendor figures) | fircosoft.com / LNRS hub (⚠ vendor numbers) |

---

## 4. The Sanctions Regime

### 4.1 OFAC — the US Layer

The US Office of Foreign Assets Control (OFAC) administers US economic sanctions; its core list is the **Specially Designated Nationals and Blocked Persons List (SDN)**, supplemented by consolidated/non-SDN lists, sectoral sanctions (SSI), and country programmes (Iran, Cuba, North Korea, Syria, Crimea, Russia/Belarus since 2022, etc.).

- ✅ Title/URL-level verified this pass: ofac.treasury.gov hosts the sanctions list site ("OFAC - Sanctions List Site", human-readable SDN list page). ⚠ The page is a JavaScript application whose full content did not render for extraction — list *content* and current SDN counts are not re-verified here.
- ✅ The Firco-side connection is verified at the vendor-statement level: the fircosoft.com / LNRS hub says its services *"support conformity with global laws such as the USA PATRIOT Act and OFAC regulations"* (vendor claim — fine as a positioning fact, not as a legal opinion).
- ✅ Sibling repo context: OFAC screening is the standard first line in AML/KYC stacks throughout the banking cluster (e.g. asset_management_alternatives_guide.md: "sanctions screening (OFAC, UN, EU)").
- ⚠ Not verified this pass: any specific OFAC enforcement statistics, current SDN counts, or Firco-specific OFAC certifications (e.g. OFAC sanctions list formats consumed).

### 4.2 The UN Security Council

The UN Security Council's sanctions are the *base layer* of the global regime — most national lists derive from or reference them. Verified this pass from the official page (✅ main.un.org):

- Sanctions measures are imposed **under Article 41 of the UN Charter** (measures not involving armed force), within Chapter VII.
- **Since 1966 the Security Council has established 31 sanctions regimes** (Southern Rhodesia through Mali); **15 regimes are currently active**, each administered by a sanctions committee.
- The **Consolidated List** is the single reference list of designated individuals and entities (the ISIL/Da'esh & Al-Qaida list being the most famous; the Ombudsperson handles delisting for it).
- Current regimes include ISIL (Da'esh) & Al-Qaida (1267), DPRK (1718), Iran (1737), Libya (1970), Taliban (1988), Yemen (2140), South Sudan (2206), Haiti (2653), DRC (1533), Sudan (1591), Central African Republic, Mali, and others.

### 4.3 The EU

The EU sanctions regime operates under the Common Foreign and Security Policy (CFSP): Council decisions + implementing regulations, applied EU-wide. The **EU Sanctions Map** (sanctionsmap.eu) is the Commission's consolidated visualisation of active regimes and designated persons/entities.

- ⚠ Verified at title level only this pass: the EU Sanctions Map page exists and is titled "EU Sanctions Map" but is a JavaScript application that returned no content to the extractor.
- ⚠ The "21st package" reference in the retrieved EU-compliance article (facctum.com: "EU Sanctions Compliance: What the 21st Package Changed") evidences an ongoing package-based regime (industry source; package numbering unverified).
- Sibling context: EU lists are part of the standard screening set in the repo's AML content (asset_management_alternatives_guide.md; regtech_guide.md §4.3 "sanctions lists (UN, OFAC, EU, MAS's own lists)" ✅).

### 4.4 The UK — OFSI

The UK's Office of Financial Sanctions Implementation (OFSI) administers UK financial sanctions (the UK Sanctions List under the Sanctions and Anti-Money Laundering Act 2018, with the Russia regime among the largest).

- ⚠ Verified at the enforcement-example level this pass: OFSI fined **Bank of Scotland £160,000** after a screening system missed transliteration variants of a designated Russian individual (aml-analytics.com — industry source retrieved this pass; the fine and cause are as reported there). This is the canonical *why fuzzy matching matters* enforcement case and the sort of outcome sanctions-screening vendors sell against.
- ⚠ The UK Sanctions List itself was not extracted this pass (budget); the OFSI fine case is the verified UK-regime fact.

### 4.5 Singapore — MAS and the ABS

The Singapore layer is the one that binds Cymbal Bank directly, and its facts are already verified in the sibling guides — cross-referenced here rather than re-derived:

- **MAS Notice 626** (Prevention of Money Laundering and Countering the Financing of Terrorism – Banks) is the bank AML/CFT rulebook: CDD, beneficial ownership, ongoing monitoring, **STR filing, sanctions screening**, record-keeping, board/MLRO accountability (✅ mas_regulations_guidelines_guide.md — verified from the on-disk MAS catalogue; effective 30 June 2025 revision wave). Sanctions screening is also an explicit element of the merchant-bank notice (1014) and the payments notices (PSN01/PSN02) (✅ sibling-verified).
- **MAS maintains its own "Lists of Designated Individuals and Entities"** — sanctions/designations lists published on the MAS site (✅ mas_regulations_guidelines_guide.md §1.4, the MAS website map).
- The payments-notices family (MAS Notices 610/622/623 for payment service providers) likewise carries sanctions-screening obligations (✅ financial_infrastructure_guide.md / payments_hub_guide.md sibling references; Notice 626 remains the banks' rulebook).
- **The ABS** — the Association of Banks in Singapore (est. 1973 ⚠, per banks_in_singapore_guide.md) — is the industry association that, alongside MAS's guidance, shapes *industry practice* for how banks operationalise screening (shared red-flag typologies, industry notices). No ABS sanctions-specific guideline was extracted this pass ⚠; the ABS's verified role is operating FAST/PayNow and industry coordination (✅/⚠ banks_in_singapore_guide.md §8).
- Regtech guide context: screening checks *"sanctions lists (UN, OFAC, EU, MAS's own lists), PEP databases, and adverse media"* (regtech_guide.md §4.3 ✅).

### 4.6 The Regime Table

| Regime | Authority / list | Verified status (this pass) | Evidence |
|---|---|---|---|
| **US — OFAC** | SDN List + consolidated lists; country/sectoral programmes | ⚠ title/URL-level; content JS-blocked; vendor "OFAC/PATRIOT Act conformity" claim ✅ | ofac.treasury.gov list site; fircosoft.com hub |
| **UN** | Security Council, Art 41 / Chapter VII; Consolidated List | ✅ full page extracted | main.un.org (31 regimes since 1966; 15 active) |
| **EU** | CFSP Council decisions/regulations; EU Sanctions Map | ⚠ title-level (JS app); package numbering ⚠ | sanctionsmap.eu; facctum.com article |
| **UK — OFSI** | UK Sanctions List | ⚠ enforcement example only | OFSI £160k Bank of Scotland fine (aml-analytics.com) |
| **Singapore — MAS** | Notice 626 (banks) + MAS designated lists; PSN01/PSN02, 1014 | ✅ sibling-verified | mas_regulations_guidelines_guide.md; regtech_guide.md §4.3 |
| **Singapore — ABS** | Industry association; operationalise screening practice | ⚠ role verified at association level; no ABS sanctions guideline extracted | banks_in_singapore_guide.md §8 |

### 4.7 The Enforcement Stakes — Why Screening Fails

The regulatory record is what makes this market exist, and the verified facts cluster around two asymmetric failure modes:

- **False negatives (the catastrophe):** the OFSI case — Bank of Scotland fined **£160,000** after its screening system failed to catch transliteration variants of a designated Russian individual (⚠ industry source, aml-analytics.com, retrieved this pass). One missed spelling variant = a regulatory penalty; the vendor pitch ("eliminate the risk of fines and reputational exposure" — ✅ RELX's own FircoSoft boilerplate) is written directly against this failure mode.
- **False positives (the cost):** the industry-press-reported >90% false-positive rates on rule-based AML stacks (⚠ press-reported, regtech_guide.md §4.7) — the operational drag that precision-focused engines and, increasingly, AI-assisted triage sell against (regtech_guide.md §4.3/§4.7).
- **The Singapore supervisory layer:** MAS enforcement exists and is documented in the sibling guide — enforcement actions, prohibition orders, composition penalties, FSMA prosecutions (✅ mas_regulations_guidelines_guide.md §6) — and AML/sanctions weaknesses are a standing MAS theme. The mechanism by which a screening failure becomes a MAS action is the Notice 626 obligations map (✅ mas_regulations_guidelines_guide.md §7.2); no MAS sanctions-screening-specific enforcement case was retrieved this pass ⚠.

**The architect's takeaway:** a sanctions-screening estate is priced in two currencies — fines (for the false negatives) and analyst headcount (for the false positives) — and every design decision in §8 trades one against the other.

---
## 5. The Screening Process

### 5.1 Client Onboarding Screening

At onboarding, the bank screens the customer's identity data — name, aliases, date of birth, nationality, address, company registry identifiers — against the sanctions lists, PEP databases, and adverse media *before* the account is opened, and re-runs on a refresh cycle (CDD layer; regtech_guide.md §4.2). In the vendor stack this is where Firco-class name screening and Fenergo-class onboarding workflow meet (regtech_guide.md §4.3: Fenergo does "screening inside onboarding"; the KYC/CLM standard for wholesale banks). Key properties:

- **Screening is a gate, not a report:** a confirmed list hit at onboarding means refuse the relationship (or file for a licence where applicable) — not "open and monitor."
- **Onboarding screening is batch-tolerant but refresh-driven:** it runs at account open, at periodic KYC refresh, at material events (name change, new beneficial owner, new jurisdiction), and — critically — **when the lists change** (the rescreening trigger; standard_chartered_guide.md §7 — ⚠ inference, sibling-flagged).
- **EDD customers get more:** high-risk customers (PEPs, high-risk jurisdictions) are screened with stricter thresholds and more frequent refresh (regtech_guide.md §4.2 ✅).

### 5.2 Transaction Screening

Transaction screening is the real-time control: every payment instruction is screened for hits in all named parties (ordering customer, beneficiary, beneficiary bank, intermediaries, free-text remittance) before release. Properties:

- **Latency budget:** screening sits on the payment path, so the engine must decide in the time the rail allows — sub-second for instant rails, near-real-time for SWIFT batches (payment_rails_guide.md; financial_fraud_detection_at_scale_guide.md §1 gives the fraud-side latency context: sub-100ms decisions at millions of transactions/day).
- **Format awareness:** MT103/MT202 for SWIFT, pacs.008/pacs.009 for ISO 20022; the richer structured data in ISO 20022 improves screening quality (✅ payment_rails_guide.md: "structured addresses and legal entity identifiers improve screening quality"; SWIFT's own "Guiding principles for screening ISO 20022 payments" — ✅ found this pass).
- **The control outcome:** no-hit → release (STP); hit → block/hold for investigation; in a sanctions context a confirmed hit means the payment must not be processed and, where applicable, the funds are frozen and reported.
- **Standby resilience:** transaction screening is business-critical enough that the product line includes a dedicated real-time screening solution (Firco™ Continuity — ✅ official snippet) — the "readiness for regulatory scrutiny" framing.

### 5.3 The Name-Screening Workflow — Exact vs Fuzzy

The screening engine's decision pipeline, as established by the industry literature and the sibling guides:

1. **Normalisation** — strip punctuation/case, expand common abbreviations, map honorifics.
2. **Exact match** — identical string on the list: automatic high-confidence hit (still verified by an analyst or by rule if the data is unambiguous).
3. **Fuzzy match** — similarity above the configured threshold via string-distance (Levenshtein, Jaro-Winkler), phonetic (Soundex/Metaphone-class), and transliteration rules (§3.1): generates an alert for human review.
4. **Threshold tuning per jurisdiction and per list** — the same name may need a tighter threshold against the UN list than against an adverse-media feed; "the matching logic must be tunable per jurisdiction" (regtech_guide.md §4.3 ✅).
5. **Disposition** — analyst reviews the alert against source data: false positive (name coincidence — different person/country), true hit (escalate to sanctions officer; freeze/block; STR where relevant), or inconclusive (further EDD).
6. **Feedback** — dispositioned cases feed threshold/rule tuning and, in AI-assisted stacks, model retraining (regtech_guide.md §4.7; financial_fraud_detection_at_scale_guide.md §14 — outcomes become labels).

The failure modes are asymmetric, and they are the whole commercial story of the market: **false negatives** (a designated name passes — the Bank of Scotland/transliteration case, §4.4) are the regulatory catastrophe; **false positives** (innocent name coincidences flagged) are the operational cost — and the reason precision is the vendor arms race (regtech_guide.md §4.3: "The vendor arms race is about precision — fewer false positives without missing true matches").

### 5.4 False Positives — the Economic Engine of the Market

The AML angle is already verified in the sibling guides; the short version to carry into any Firco procurement:

- Industry-press-reported false-positive rates for rule-based AML stacks are **above 90%** at many institutions ⚠ press-reported, unverifiable as a single number (regtech_guide.md §4.7).
- The fraud-detection side sets the precision benchmark differently: mature fraud systems target **sub-2–5% false-positive rates, sub-1% at best-in-class** — but fraud scoring is a *risk score*, while sanctions screening is a *binary list check* on far sparser data, so the two are not directly comparable (financial_fraud_detection_at_scale_guide.md §1 ✅).
- The screening-specific economics: each false positive costs analyst time and delays a payment; each false negative can cost the licence. The buying case for a precision-focused engine like the Firco class is therefore: **same recall, fewer alerts, lower cost-per-investigation, faster STP** — exactly the pitch that RegTech guide §4.7 identifies as the industry's economic engine ("cut the false-positive workload by an order of magnitude while catching at least as much").

### 5.5 The Process Table

| Process | When | What is screened | Control outcome | Verified status |
|---|---|---|---|---|
| Onboarding screening | Account open; KYC refresh; material events | Customer, beneficial owners, counterparties, jurisdictions | Refuse / EDD / accept | ✅ industry pattern (regtech_guide.md §4.2/§4.3) |
| List-change rescreening | On every list update | Existing customer base, open transactions | Re-alert changed records | ⚠ sibling-inferred (standard_chartered_guide.md §7) |
| Transaction screening | Every payment, pre-release | All named parties + free text | Release (STP) / hold / block | ✅ (payment_rails_guide.md; Firco Continuity snippet) |
| Exact match | Every screen | Exact-string hits | Auto-hit → analyst confirm | ✅ industry standard |
| Fuzzy match | Every screen | Similarity above threshold | Alert → investigation | ✅ techniques verified; Firco engine ⚠ |
| Disposition & feedback | Post-alert | Evidence, source lists, customer data | False positive / confirmed hit / STR | ✅ case-management pattern (fraud guide §14; regtech §4.6) |

---

## 6. The Competitors

### 6.1 The Screening-Data and Screening-Engine Field

The sanctions-screening market splits into **list/data providers** (the risk-intelligence feeds) and **screening engines** (the matching/decision software) — with several vendors spanning both. Firco's distinctive position is *engine-first*: the matching/filtering software specialist, later enriched with the LexisNexis data layer (WorldCompliance). The verified competitor set, from this pass and the sibling guides:

- **World-Check (LSEG)** — the screening *data* standard: risk-intelligence profiles of sanctions/PEP/adverse-media subjects. The sibling asset-management guide lists "World-Check (Refinitiv/LSEG)" among AML screening data vendors (✅ asset_management_alternatives_guide.md); the Financial Risk & Compliance Systems guide lists "World-Check (OneSumX)" — i.e. World-Check embedded in the Wolters Kluwer OneSumX AML suite — among sanctions-screening vendors (✅ financial_risk_compliance_systems_guide.md). The RegTech guide flags the Thomson Reuters ONESOURCE/World-Check heritage as ⚠ knowledge-based (regtech_guide.md §5.5). **World-Check's current LSEG ownership is ⚠ not re-verified this pass** (budget) — it is sibling-cited.
- **Dow Jones Risk & Compliance** — the other major screening *data* franchise (sanctions/PEP/adverse-media profiles). Listed alongside Fircosoft and World-Check in the sibling asset-management guide's AML vendor mention (✅ sibling mention — asset_management_alternatives_guide.md). ⚠ No Dow Jones-specific page was retrieved this pass; facts beyond existence are unverified.
- **LexisNexis Bridger** — LNRS's own screening *data* brand (the "Bridger Insight" lineage), listed in the Financial Risk & Compliance Systems vendor table (✅ sibling). Interesting intra-family wrinkle: Firco (engine) and Bridger (data) now sit in the same parent (⚠ the degree of product integration is unverified).
- **NICE Actimize** — the AML platform incumbent: transaction monitoring, fraud, trade surveillance (Solas); "the legacy AML default" (✅ regtech_guide.md §5.2 — Actimize founded 1999, acquired by NICE 2007 ⚠ dates). Actimize competes with Firco at the *platform* level (it embeds screening) more than at the engine level.
- **BAE Systems NetReveal** — the financial-crime analytics platform (AML/fraud, with a sanctions-screening module). ⚠ Not verified this pass (no source retrieved); listed per the task's competitor set and common industry knowledge — flagged.
- **Fenergo** — the KYC/client-lifecycle-management standard (Dublin, founded 2009 ✅ regtech_guide.md §5.2); competes at the *onboarding workflow* layer where screening is invoked (regtech_guide.md §4.3 "screening inside onboarding").
- **Also in the field (sibling-verified):** Accuity (payment routing + AML screening data — ✅ financial_risk_compliance_systems_guide.md), SAS Sanctions (✅ same table), ComplyAdvantage (real-time AML data + screening — ✅ regtech_guide.md §5.2), Silent Eight (AI screening — ✅ §5.3, Singapore-founded 2013), Tookitaki (APAC AML platform — ✅), Chainalysis/Elliptic/TRM (crypto-address sanctions — ✅ regtech_guide.md §4.5).

### 6.2 The Comparison Table

| Vendor | Category | Home turf | Verified status (this pass / sibling) |
|---|---|---|---|
| **Fircosoft (LexisNexis Risk Solutions)** | Screening engine + transaction filtering | The sanctions name-match/transaction-filter specialist; 700+ customers at acquisition, 8 of top-10 FIs (2014) | ✅ ownership/founding/products; scale claims ⚠ |
| **World-Check (LSEG)** | Screening data | The risk-intelligence feed banks embed in screening/TM platforms | ✅ sibling-cited (asset guide; FRCS guide); current ownership ⚠ |
| **Dow Jones Risk & Compliance** | Screening data | Sanctions/PEP/adverse-media profiles; news-intelligence-backed | ✅ sibling mention; ⚠ unverified beyond existence |
| **LexisNexis Bridger** | Screening data | LNRS's screening-data brand; sibling of Firco under RELX | ✅ sibling-cited (FRCS guide); integration with Firco ⚠ |
| **NICE Actimize** | AML platform | Transaction monitoring, fraud, trade surveillance; the legacy AML default | ✅ sibling-cited (regtech §5.2); 1999/2007 dates ⚠ |
| **BAE NetReveal** | AML/financial-crime analytics | Analytics platform with screening module | ⚠ not verified this pass |
| **Fenergo** | KYC / CLM | Client onboarding and lifecycle management; screening invoked inside onboarding | ✅ sibling-cited (regtech §5.2, Dublin 2009) |
| **Accuity** | Payments + AML data | Payment routing data + AML screening data | ✅ sibling-cited (FRCS guide) |
| **SAS Sanctions** | Screening engine/data | SAS's sanctions screening line | ✅ sibling-cited (FRCS guide) |
| **ComplyAdvantage** | AML data + screening | Real-time sanctions/PEP/adverse-media feeds, London 2014 | ✅ sibling-cited (regtech §5.2) |
| **Silent Eight** | AI screening | AI-driven screening/CDD decisions (Singapore, 2013) | ✅ sibling-cited (regtech §5.3) |

**Reading the table.** For a Cymbal Bank-sized institution the practical shortlist is: Firco or a Firco-class engine (screening execution) + World-Check or Dow Jones (list data) — with Actimize/Fenergo competing at the platform/workflow layers above the engine, and the AI challengers (Silent Eight, ComplyAdvantage) competing on the false-positive economics. The FRCS guide's vendor table (World-Check/OneSumX · LexisNexis Bridger · Fircosoft · Accuity · SAS Sanctions — financial_risk_compliance_systems_guide.md) is the repo's established cut of exactly this market.

---

## 7. The Banking Context — Cymbal Bank

### 7.1 The Obligations

Cymbal Bank's sanctions-screening obligations are the verified MAS stack (cross-referenced from the sibling guides, not re-derived):

- **MAS Notice 626** (AML/CFT – Banks): CDD/EDD, beneficial ownership, ongoing monitoring, **sanctions screening**, STR filing, record-keeping, board/MLRO accountability — the single most-audited notice in the compliance stack (✅ mas_regulations_guidelines_guide.md — 30 June 2025 revision wave).
- **MAS designated lists**: MAS publishes its own Lists of Designated Individuals and Entities, which a Singapore bank screens against *in addition to* the UN/OFAC/EU lists it consumes for its cross-border book (✅ mas_regulations_guidelines_guide.md §1.4; regtech_guide.md §4.3 "UN, OFAC, EU, MAS's own lists").
- **The payments side**: if Cymbal Bank runs payment services, the PSN-series notices (PSN01/PSN02) and MAS Notices 610/622/623 carry parallel screening obligations (✅ sibling-verified).
- **The ABS layer**: the Association of Banks in Singapore (est. 1973 ⚠) shapes industry practice and operates the rails (FAST/PayNow) over which screened payments flow (✅/⚠ banks_in_singapore_guide.md §8).

### 7.2 The Screening Estate

The estate pattern for a bank of Cymbal Bank's profile, per the repo's compliance-systems cluster:

- **Vendor layer**: a Firco-class screening engine (or the FRCS guide's vendor cut: World-Check/OneSumX, Bridger, Fircosoft, Accuity, SAS — financial_risk_compliance_systems_guide.md) + World-Check/Dow Jones-style data feeds + Fenergo-class onboarding workflow invoking the screens (regtech_guide.md §4.2/§4.3).
- **Integration points**: core banking (customer master → screening at onboarding/refresh), payments hub (transaction screening on the payment path — the payments-hub pattern of payments_hub_guide.md and payment_rails_guide.md), KYC platform (risk rating, EDD), STR/case-management (AML investigation), and the regulatory-reporting layer.
- **The ISO 20022 agenda**: CBPR+ migration means reworking screening to consume structured addresses/LEIs — "structured addresses and legal entity identifiers improve screening quality" (✅ payment_rails_guide.md; SWIFT's screening-principles document ✅ found this pass).
- **The AI layer**: the AI/GenAI compliance guide covers how FEAT/AIRM-era expectations apply to AI-assisted screening (ai_genai_banking_compliance_guide.md — cross-ref; AI screening cut false-positive workload — regtech_guide.md §4.7).
- **The industry-standard inference**: sibling guides record that the *class* — Firco-class screening — is near-universal at large banks, even where a specific bank-vendor contract is unverified (dbs_software_systems_guide.md; standard_chartered_guide.md §7 — both ⚠ inference). Cymbal Bank's own vendor choice is a design decision in §8, not a claim about any real contract.

### 7.3 The Estate Table

| Estate component | What it does | Cross-ref / verified status |
|---|---|---|
| Screening engine (Firco-class) | Name matching + transaction filtering | financial_risk_compliance_systems_guide.md (vendor table); standard_chartered_guide.md §7 (industry-standard inference ⚠) |
| List data feeds | World-Check/Dow Jones/WorldCompliance-class sanctions+PEP data | asset_management_alternatives_guide.md; regtech_guide.md §4.3 |
| Onboarding/KYC workflow | Invokes screening at CDD/refresh | regtech_guide.md §4.2 (Fenergo-class CLM) |
| Payments hub | Real-time transaction screening on the path | payments_hub_guide.md; payment_rails_guide.md (ISO 20022 overlay) |
| Case management | Alert investigation, dispositions, STR | financial_fraud_detection_at_scale_guide.md §14; regtech_guide.md §4.6 |
| Compliance data layer | MAS lists + UN/OFAC/EU feeds | mas_regulations_guidelines_guide.md §1.4 (MAS lists) |
| AI layer | Precision tuning, alert triage assistance | ai_genai_banking_compliance_guide.md; regtech_guide.md §4.7 |

### 7.4 Procurement and Operating Notes

For the architect writing the RFP or the annual vendor review, the selection criteria that the verified facts support:

| Criterion | What to test | Why (verified basis) |
|---|---|---|
| **Matching precision** | Benchmark recall on transliteration/typo test sets; ask for the false-positive rate at fixed recall | The vendor arms race is precision (regtech_guide.md §4.3 ✅); the OFSI case shows recall failure is a fine (⚠) |
| **List currency** | Same-day propagation SLA for OFAC/UN/EU/UK/MAS updates; delta-rescreen capability | "List updates must propagate in near-real-time" (regtech_guide.md §4.3 ✅) |
| **ISO 20022 readiness** | Structured address/LEI consumption; CBPR+ message support | Structured data improves screening quality (payment_rails_guide.md ✅; SWIFT screening principles ✅) |
| **Real-time transaction path** | Sub-second decision latency; standby/continuity node for the payment path | Preventive control on the payment path (regtech_guide.md §4.5 ✅; Firco Continuity positioning ✅) |
| **Workflow/case management** | Disposition taxonomy, bulk triage, audit trail, STR handoff | The fifth AML-stack layer (regtech_guide.md §4.6 ✅; fraud guide §14 ✅) |
| **Data layer** | List-data quality SLAs; PEP/adverse-media coverage; source transparency | Screening consumes data feeds as much as engines (regtech_guide.md §4.3; WorldCompliance positioning ✅) |
| **AI assist (optional)** | Explainability, human-in-the-loop, FEAT/AIRM-style governance | ai_genai_banking_compliance_guide.md (cross-ref) |

**Ownership diligence note:** the vendor's corporate parent matters in this market — FircoSoft's own history is a 1990 Paris startup → Keensight-backed → Accuity/Reed Elsevier (2014) → LexisNexis Risk Solutions today (§2), and World-Check's LSEG ownership, Bridger's LNRS home, and Actimize's NICE parent are the same kind of fact a bank's third-party-risk process should re-verify at procurement time (⚠ current ownership details beyond Firco were sibling-cited, not re-verified this pass).

---

## 8. The Worked Example — A Cymbal Bank Sanctions-Screening Design

### 8.1 The Scenario

Cymbal Bank (Singapore full-bank licence; MAS Notice 626 obligations ✅; wholesale + SME book with a cross-border payments corridor) is designing its sanctions-screening estate. It has chosen a **Firco-class engine** (engine + list management) with **WorldCompliance-class risk intelligence**, **Fenergo-class onboarding**, and a **payments hub** that invokes transaction screening in the payment path. The design targets three operational numbers: **>99.9% STP for clean payments, <1% alert rate on the transaction stream, and every list change re-screened within the same business day.**

Design constraints from the verified facts:

- Screening runs **at onboarding, on payments (real-time), and on list changes** (standard_chartered_guide.md §7 — ⚠ sibling inference, adopted as design practice).
- The engine must consume **UN (Consolidated List ✅), OFAC (SDN — ⚠ content-level), EU (⚠ title-level), UK OFSI (⚠ enforcement-level), and MAS designated lists (✅)** — §4.
- ISO 20022 structured data **improves screening quality** (✅ payment_rails_guide.md) — the design consumes structured addresses/LEIs, not just names.
- False-positive economics dominate the operating budget (regtech_guide.md §4.7).

### 8.2 The Screening Workflow

**1. Onboarding path (batch, event-driven):**

- Customer master → CDD (Fenergo-class) → Firco-class screen against all lists (exact + fuzzy) → risk rating.
- Outcomes: **clean** → open; **fuzzy alert** → analyst triage (disposition: false positive or escalate); **confirmed hit** → refuse/EDD/escalate to the sanctions officer.
- Refresh triggers: periodic KYC cycle, material events, and **list-change deltas** (rescreen affected customers same-day).

**2. Transaction path (real-time, synchronous where the rail allows):**

- Payments hub → normalise message (MT103/MT202 or pacs.008/pacs.009) → extract all parties + structured address/LEI fields → screen → decision:
  - **No hit** → STP release.
  - **Fuzzy hit** → hold + alert; the payment waits in a review queue (SLAs: minutes for instant rails, T+0 for SWIFT).
  - **Confirmed hit** → block; freeze where applicable; sanctions-officer notification; STR if required.
- **Standby**: a Continuity-class secondary screening node keeps the path alive during engine maintenance/outages (the "readiness for regulatory scrutiny" pattern — ✅ Firco Continuity snippet).

**3. List-change path (same-day):**

- List ingestion → delta computation → threshold re-evaluation → **rescreen of matched segments** (customers whose attributes intersect the delta; in-flight transactions where the rail allows) → new alerts flow into the same triage queue.

**The design components at a glance:**

| Component | Pattern | Verified basis |
|---|---|---|
| Onboarding gate | Fenergo-class CLM invoking Firco-class screens at CDD/refresh | regtech_guide.md §4.2/§4.3 |
| Transaction screener | Real-time engine on the payments hub; MT103/MT202 + pacs.008/pacs.009 aware | payment_rails_guide.md; SWIFT screening principles |
| List manager | Same-day delta ingestion + rescreen trigger | regtech_guide.md §4.3; standard_chartered_guide.md §7 (⚠) |
| Triage queue | Risk-prioritised alert queue; SLA buckets; bulk FP disposition | financial_fraud_detection_at_scale_guide.md §14 |
| Escalation path | Sanctions officer; freeze; STR via case management | regtech_guide.md §4.6; mas_regulations_guidelines_guide.md (Notice 626) |
| Standby node | Continuity-class second screening instance for path resilience | Firco Continuity positioning (✅) |

### 8.3 False-Positive Management

- **Tiered thresholds per list and jurisdiction** — UN list: tightest (any phonetic/transliteration proximity alerts); adverse media: looser; PEP: medium — "tunable per jurisdiction" (regtech_guide.md §4.3 ✅).
- **Structured-data disambiguation** — ISO 20022 address/LEI/DOB fields kill the classic false positive ("John Smith, London" vs the designated "John Smith, Aleppo"): same name, different entity, no alert (payment_rails_guide.md ✅ — the structured-data quality agenda).
- **Disposition taxonomy + feedback loop** — every alert ends in one of: `FP-name-coincidence`, `FP-data-error` (bad source data — fix the master), `FP-threshold-too-loose` (tighten), `True-hit`, `Inconclusive→EDD`. Dispositions feed threshold tuning quarterly and the AI triage layer's training set (financial_fraud_detection_at_scale_guide.md §14 — outcomes become labels ✅).
- **Analyst workflow economics** — queue with priority scoring (risk-rated customers first), bulk-disposition tools for obvious FP cohorts (same-name clusters), and an audit trail per alert (who decided what, when, with what evidence — regtech_guide.md §4.6 ✅).
- **AI-assisted triage (pilot)** — an LLM-assisted summariser proposes dispositions with evidence links; humans confirm; FEAT/AIRM-style governance from ai_genai_banking_compliance_guide.md applies (cross-ref).

### 8.4 The Metrics

| Metric | Definition | Target | Rationale / cross-ref |
|---|---|---|---|
| **Alert rate** | Alerts ÷ screened transactions | <1% (transaction stream) | False-positive economics (regtech_guide.md §4.7 — industry >90% FP on rule-based stacks is the cautionary baseline ⚠ press) |
| **STP rate** | No-hit releases ÷ screened | >99.9% clean payments | The customer-experience cost of screening (fraud guide §1 latency/friction framing ✅) |
| **Precision / recall** | Confirmed hits ÷ alerts; hits caught ÷ designated-name touches | Precision rising, recall = 100% on test sets | The vendor arms race: "fewer false positives without missing true matches" (regtech_guide.md §4.3 ✅) |
| **Screen latency** | p50/p95 engine decision time | <100 ms p95 transaction path | Real-time control on the payment path (fraud guide §1 sub-100ms context ✅) |
| **List-change SLA** | Time from list publication to rescreen completion | Same business day | List currency is the screening layer's defining requirement (regtech_guide.md §4.3 ✅) |
| **Alert ageing** | Open alerts by age bucket | 100% triaged within SLA (T+1) | Case-management throughput (fraud guide §14 ✅) |
| **Cost per alert** | Analyst hours ÷ alerts | Falling YoY | The buying case for precision engines (regtech_guide.md §4.7 ✅) |

### 8.5 The Lessons

1. **The engine is only half the system.** Firco-class matching precision is worthless without (a) list currency (same-day deltas), (b) structured data (ISO 20022 address/LEI disambiguation — payment_rails_guide.md), and (c) a disciplined disposition loop. The vendor sells the engine; the bank builds the loop.
2. **False-positive management is a data-quality programme, not a tuning exercise.** The biggest FP killers in the worked example were master-data errors and missing structured fields — both fixed upstream of the engine.
3. **Metrics must be asymmetric-aware.** Recall failures are measured in fines (Bank of Scotland/OFSI case, §4.4); precision failures are measured in headcount. Report both, every month, to the same committee.
4. **The vendor relationship is a data relationship.** Firco's value today is the combination of engine + LexisNexis risk intelligence (WorldCompliance — ✅); the procurement must cover list-data quality SLAs, not just software licences.
5. **Everything above is design content.** No claim is made that Cymbal Bank runs Firco in production; the estate pattern is the repo's established "Cymbal Bank-style" worked-example convention, built on verified vendor and regulatory facts.

---
## 9. The Summary — "The Clean List"

Sanctions screening is the compliance control that must be **right in both directions at once**: never let a designated name through (false negatives are catastrophic — fines, monitorships, reputational collapse) and never drown the operation in innocent-name alerts (false positives are the cost engine of the whole industry). Fircosoft is the vendor that built its franchise on the first half of that problem — the watch-list filtering engine — and, since 29 September 2014, has done it as **LexisNexis Risk Solutions (RELX)**, a Paris-founded (1990) specialist with 700+ customers and eight of the world's top ten financial institutions at acquisition, a product family spanning name screening (FircoTrust ⚠), transaction filtering (Firco Filtering Suite ⚠, Firco™ Continuity ✅), end-to-end screening workflow (Firco Compliance Link ✅), and the WorldCompliance risk-intelligence layer (✅). Its market is defined by the regimes it screens against — OFAC, the UN's 15 active Security Council regimes, the EU, UK OFSI, and, for a Singapore bank, the MAS designated lists under Notice 626 — and by the competitors it runs against: World-Check (LSEG), Dow Jones Risk & Compliance, NICE Actimize, BAE NetReveal, Fenergo, and the AI challengers.

The Cymbal Bank design in §8 reduces to five rules: screen at onboarding, on every payment, and on every list change; feed the engine structured data (ISO 20022) so it can disambiguate instead of guess; tune thresholds per list and jurisdiction; run a disciplined disposition loop whose outcomes retrain the thresholds; and measure alert rate, STP rate, latency, and list-change SLA side by side. Done right, the operation converges on what every sanctions desk is ultimately selling: **the clean list** — the list of names that have been checked, dispositioned, and cleared, the only list that should ever touch a payment path.

---

## 10. Glossary

| Term | Definition |
|---|---|
| **Sanctions** | Economic and financial measures imposed by states or the UN Security Council (Article 41 / Chapter VII) to coerce or constrain targeted regimes, entities and individuals — asset freezes, trade restrictions, travel bans ✅ (main.un.org) |
| **Screening** | Checking names (customers, counterparties, payment parties) against sanctions/watch lists to detect list hits (regtech_guide.md §4.3) |
| **Filtering** | The transaction-side sibling of screening: mechanically checking every payment message for list hits and blocking/holding them — Firco's founding product category ("watch list filtering") ✅ (RELX 2014) |
| **List management** | Ingesting, normalising and propagating regulator list updates into the live screening configuration; the "latency + list-management" property of the screening layer (regtech_guide.md §4.3) |
| **Fuzzy matching** | Matching names despite spelling variation, transliteration and word-order differences — string distance (Levenshtein, Jaro-Winkler), phonetic algorithms (Soundex/Metaphone-class), trigrams ✅ (this pass's fuzzy-matching sources) |
| **False positive** | An alert on an innocent name coincidence — the operational cost of screening; industry-press FP rates on rule-based AML stacks exceed 90% ⚠ press-reported (regtech_guide.md §4.7) |
| **False negative** | A designated name that passes screening — the regulatory catastrophe; e.g. the OFSI £160,000 Bank of Scotland transliteration case ⚠ (aml-analytics.com) |
| **OFAC** | US Office of Foreign Assets Control — administers US sanctions; publisher of the SDN (Specially Designated Nationals and Blocked Persons) List ⚠ title-level verified this pass |
| **UN** | United Nations — Security Council sanctions under Chapter VII/Article 41; 31 regimes since 1966, 15 active; the Consolidated List ✅ (main.un.org) |
| **EU** | European Union — CFSP-based sanctions decisions/regulations; the EU Sanctions Map (sanctionsmap.eu) is the Commission's visualisation ⚠ title-level |
| **OFSI** | UK Office of Financial Sanctions Implementation — administers the UK Sanctions List; the £160k Bank of Scotland fine is the canonical screening-failure case ⚠ |
| **MAS** | Monetary Authority of Singapore — publishes its own Lists of Designated Individuals and Entities and binds banks to sanctions screening via Notice 626 (✅ mas_regulations_guidelines_guide.md) |
| **ABS** | Association of Banks in Singapore (est. 1973 ⚠) — the industry association shaping screening practice; operates FAST/PayNow (banks_in_singapore_guide.md §8) |
| **SWIFT** | The messaging cooperative and network carrying cross-border payments (MT103/MT202); ISO 20022 (pacs.008) is the structured successor under CBPR+ (payment_rails_guide.md) |
| **STP** | Straight-through processing — a payment released without manual intervention; the metric screening must not destroy |
| **Case management** | The fifth AML-stack layer: alert investigation, evidence capture, disposition, audit trail, STR filing (regtech_guide.md §4.6; financial_fraud_detection_at_scale_guide.md §14) |
| **World-Check** | The LSEG screening-data franchise (Refinitiv heritage ⚠) — the risk-intelligence feed banks embed in screening platforms ✅ sibling-cited |
| **LSEG** | London Stock Exchange Group — owner of the Refinitiv/World-Check data lines ⚠ current ownership sibling-cited, not re-verified this pass |
| **Dow Jones** | Dow Jones Risk & Compliance — the other major screening-data franchise (sanctions/PEP/adverse-media profiles) ✅ sibling mention; details ⚠ |
| **Actimize** | NICE Actimize — the AML platform incumbent (transaction monitoring, fraud, trade surveillance); founded 1999, NICE acquisition 2007 ⚠ dates (regtech_guide.md §5.2) |
| **NetReveal** | BAE Systems NetReveal — financial-crime analytics platform ⚠ not verified this pass |
| **Fenergo** | Dublin-founded (2009) KYC/client-lifecycle-management standard; screening invoked inside onboarding ✅ (regtech_guide.md §5.2) |
| **FircoTrust** | FircoSoft's customer/name-screening line ⚠ URL-level verified only (risk.lexisnexis.com/products/firco-trust) |
| **Firco Continuity** | FircoSoft's real-time transaction-screening solution ✅ (official LNRS snippet) |
| **Firco Compliance Link** | FircoSoft's all-in-one financial-crime screening workflow ✅ (official LNRS snippet) |
| **WorldCompliance** | LexisNexis Risk Solutions' risk-intelligence data (7M+ profiles, 60+ risk categories — vendor figures ⚠) ✅ product existence |
| **RELX** | Reed Elsevier renamed (2015) — the parent of LexisNexis Risk Solutions, which acquired FircoSoft (completed 29 September 2014 ✅) |
| **PEP** | Politically exposed person — a high-risk customer class screened alongside sanctions lists (regtech_guide.md §4.2) |
| **STR / SAR** | Suspicious transaction/activity report — filed to the FIU (STRO in Singapore) when investigation confirms suspicion (regtech_guide.md §4.4) |
| **SDN** | Specially Designated Nationals and Blocked Persons List — OFAC's core sanctions list ⚠ title-level |
| **EDD** | Enhanced due diligence — the deeper checks applied to high-risk customers (regtech_guide.md §4.2) |
| **Accuity** | Reed Elsevier's payment-routing-data and AML-solutions business that FircoSoft joined in 2014 ✅; later corporate fate ⚠ unverified |
| **RBI** | Reed Business Information — the RELX division that housed Accuity at the 2014 deal (✅ Jean Losco quote) |
| **OFSI** | UK Office of Financial Sanctions Implementation — administers UK financial sanctions; fined Bank of Scotland £160,000 over a transliteration screening failure ⚠ |
| **Designation** | The act of adding an individual/entity to a sanctions list; the unit of list-change a screening estate must propagate |
| **Rescreening** | Re-running existing records against a changed list — the "list-change" trigger of the screening rhythm (⚠ sibling-inferred) |
| **List delta** | The set of additions/amendments/delistings between list versions — what the rescreen actually consumes |
| **Threshold** | The similarity score above which a fuzzy match becomes an alert; tunable per list and jurisdiction (regtech_guide.md §4.3) |
| **CBPR+** | Cross-Border Payments and Reporting Plus — the SWIFT ISO 20022 standard for cross-border payments; screening must be reworked for it (payment_rails_guide.md) |
| **pacs.008 / pacs.009** | ISO 20022 payment messages (credit transfer / transfer) — the structured successors of MT103/MT202 |
| **MT103 / MT202** | SWIFT MT messages for customer credit transfers / bank-to-bank transfers — the legacy screening format |
| **STRO** | Singapore's Suspicious Transaction Reporting Office — the FIU that receives STRs (regtech_guide.md §4.4) |
| **FIU** | Financial intelligence unit — the national recipient of suspicious-transaction reports |
| **CFSP** | Common Foreign and Security Policy — the EU framework under which sanctions decisions and regulations are made ⚠ general knowledge, not re-verified |

---

## 11. Claims Audit — ✅ Verified / ⚠ Partially Verified / ❌ Not Verified

| # | Claim | Status | Evidence |
|---|---|---|---|
| 1 | FircoSoft founded 1990 | ✅ | RELX 29-Sep-2014 release ("Founded in 1990"); Tracxn ("founded in 1990") |
| 2 | FircoSoft is Paris-based | ✅ | "Paris-based global sanctions screening software group" (relx.com; librarytechnology.org; tmcnet.com); Tracxn |
| 3 | Reed Elsevier announced exclusive negotiations 3 Sep 2014; completed acquisition 29 Sep 2014 | ✅ | relx.com press release + two independent reprints |
| 4 | The acquirer was FICO | ❌ | Contradicted: all retrieved 2014 sources name Reed Elsevier (RELX); no source connects FICO to FircoSoft |
| 5 | FircoSoft now sits in LexisNexis Risk Solutions (RELX) | ✅ | fircosoft.com resolves to the LNRS Financial Crime Compliance hub; Firco products on risk.lexisnexis.com; sibling dbs_software_systems_guide.md |
| 6 | Keensight Capital was the pre-2014 PE owner | ✅ (stake extent ⚠) | Seller-side "About Keensight Capital" boilerplate in the RELX release |
| 7 | 700+ customers; 8 of world's top 10 FIs; 2,000 sites in 85+ countries; 11 offices | ✅ | RELX 2014 release; consistent CB Insights/Tracxn third-party records ⚠ (2014-era figure) |
| 8 | "45 of the world's top 50 banks"; "over 20 years in the industry" | ⚠ | Current LNRS hub vendor claims; "20 years" inconsistent with a 1990 founding |
| 9 | Product: Firco™ Continuity (real-time transaction screening) | ✅ | Official risk.lexisnexis.com product URL + snippet |
| 10 | Product: Firco Compliance Link (all-in-one screening) | ✅ | Official risk.lexisnexis.com product URL + snippet |
| 11 | Product: WorldCompliance™ Data | ✅ | fircosoft.com / LNRS hub page (7M+ profiles, 60+ categories — figures ⚠ vendor) |
| 12 | Product: FircoTrust | ⚠ | Official URL exists; page blocked scraping; content unverified |
| 13 | Product: Firco Filtering Suite | ⚠ | Third-party database only (newtonconsultingpartners.com) |
| 14 | Task-assumed product names "Sanctions Filter / Sanctions List Management / Transaction Filtering" | ⚠ | No retrieved source uses these exact strings for FircoSoft; the functional split is verified at industry level |
| 15 | Acquisition price / revenue figures | ⚠ | Not disclosed in any retrieved source |
| 16 | Fuzzy/phonetic/transliteration matching techniques | ✅ | Fuzzy-matching literature this pass (Jaro-Winkler, Levenshtein, Soundex-class, trigrams); regtech_guide.md §7.3 |
| 17 | FircoSoft's proprietary matching algorithm details | ⚠ | No official Firco page retrieved this pass describes the algorithm (product pages blocked scraping) |
| 18 | UN: 31 regimes since 1966; 15 active; Art 41/Chapter VII; Consolidated List | ✅ | main.un.org official sanctions page |
| 19 | OFAC SDN list and list site | ⚠ | Title/URL-level; JS page content did not render; vendor "PATRIOT Act/OFAC conformity" claim ✅ (fircosoft.com) |
| 20 | EU sanctions map / packages | ⚠ | Title-level (JS app); package numbering unverified |
| 21 | UK OFSI £160k Bank of Scotland transliteration fine | ⚠ | Industry source retrieved this pass (aml-analytics.com); fine and cause as reported there |
| 22 | MAS Notice 626 includes sanctions screening; MAS designated lists exist | ✅ | Sibling-verified (mas_regulations_guidelines_guide.md §1.4/§3.2; regtech_guide.md §4.3) — cross-referenced, not re-derived |
| 23 | ABS role and founding | ⚠ | banks_in_singapore_guide.md §8 (est. 1973 flagged there) |
| 24 | ISO 20022 structured data improves screening; CBPR+ screening rework | ✅ | payment_rails_guide.md (SWIFT: "more accurate compliance processes"); SWIFT screening-principles resource found this pass |
| 25 | Screening runs at onboarding, on payments, on list changes | ⚠ | Industry pattern; sibling-inferred (standard_chartered_guide.md §7, flagged there) |
| 26 | Industry FP rates >90% on rule-based AML stacks | ⚠ | Press-reported, unverifiable as a single number (regtech_guide.md §4.7) |
| 27 | Competitors: World-Check (LSEG), Dow Jones, Bridger, Actimize, Fenergo, Accuity, SAS, ComplyAdvantage, Silent Eight | ✅ (sibling-cited) | asset_management_alternatives_guide.md; financial_risk_compliance_systems_guide.md; regtech_guide.md §5 — LSEG ownership and Actimize dates ⚠ |
| 28 | Competitor: BAE NetReveal | ⚠ | Listed per task's competitor set; no source retrieved this pass |
| 29 | Worked example (§8) is design content, not a claim of a real Cymbal Bank–Firco contract | ✅ (convention) | Repo worked-example convention; no contract claim made |

---

## 12. What Could Not Be Verified

The following could not be confirmed from any source retrieved in this pass, and are deliberately left flagged rather than guessed:

1. **The "FICO acquisition"** — the task hypothesis. The evidence contradicts it: the 2014 acquirer was Reed Elsevier (RELX). If any FICO–Firco relationship exists (e.g. an OEM/reseller arrangement), no retrieved source evidences it. Treat "FICO owns FircoSoft" as false.
2. **Acquisition price and FircoSoft revenue** — not disclosed in the RELX release or any other retrieved source.
3. **FircoSoft's proprietary matching algorithm** — the product pages that would describe it (FircoTrust, Firco Continuity, Firco Compliance Link) blocked automated extraction (Internal Server Error), and search snippets do not describe algorithm internals. The *techniques* (fuzzy/phonetic/transliteration) are industry-verified; Firco's specific implementation is not.
4. **Exact current product names for the full family** — "FircoTrust" and "Firco Filtering Suite" are ⚠ (URL-level / third-party-database-level); the task-assumed names "Sanctions Filter / Sanctions List Management / Transaction Filtering" are ⚠ unconfirmed as official FircoSoft brands.
5. **OFAC list content and current SDN counts** — ofac.treasury.gov's list site is a JavaScript application that returned no extractable content ("Loading..."); only title/URL-level verification was possible.
6. **EU sanctions list content and package numbering** — sanctionsmap.eu is a JS application (title-level only); the "21st package" reference is a single industry article.
7. **The UK Sanctions List itself** — only the OFSI enforcement example (Bank of Scotland £160,000) was retrieved.
8. **ABS sanctions-specific guidance** — the ABS's association-level role is sibling-verified; no ABS sanctions guideline was retrieved.
9. **Current World-Check/LSEG ownership details and Dow Jones/NetReveal specifics** — sibling-cited for existence; not re-verified this pass.
10. **"45 of the top 50 banks" and "over 20 years in the industry"** — current vendor-marketing figures, inconsistent with the 1990 founding; unverifiable as precise claims.
11. **Any specific bank–FircoSoft contract** (e.g. at DBS or Standard Chartered) — the sibling guides already flag these as ⚠ inference; this pass adds no contract-level evidence.
12. **Web-access degradation note** — the LexisNexis product pages and OFAC/EU list sites blocked or failed to render for extraction; where that happened, the fact was flagged rather than filled in. No fact in this guide was invented to cover a gap.

**Honesty note (repo convention):** nothing in this guide was fabricated. Every ✅ row in §11 traces to a named source in the §13 ledger — a web page retrieved this pass or a sibling guide read from the repo — and every ⚠ row is explicitly flagged with why it could not be confirmed, including the one task hypothesis the evidence contradicted (the "FICO acquisition", §2.3/§11 row 4). The worked example (§8) is design/analytical content in the repo's established Cymbal Bank convention, built on verified vendor and regulatory facts; it claims no real contract and quotes no real configuration.

---

## 13. Verification Ledger

Source-by-source record of what was checked in this pass (web sources retrieved live; sibling guides read from the repo on disk):

| # | Source | What was checked | Outcome |
|---|---|---|---|
| 1 | relx.com press release, 29 Sep 2014 ("Reed Elsevier acquires FircoSoft") | Acquisition completion date; "About FircoSoft" (founded 1990, 700+ customers, 8 of top-10 FIs, 2,000 sites/85+ countries, 11 offices); Keensight boilerplate | ✅ full page extracted; all facts confirmed |
| 2 | librarytechnology.org document 28029 / pr 28029 | Reprint of 3 Sep 2014 exclusive-negotiations announcement; "Paris-based" | ✅ confirmed |
| 3 | tmcnet.com (Oct 2014) | Reprint of 29 Sep 2014 completion announcement | ✅ confirmed |
| 4 | tracxn.com FircoSoft profile | Founded 1990, Paris, acquired; "watch list filtering solutions" | ✅ confirmed (third-party database) |
| 5 | cbinsights.com FircoSoft profile | 700+ customers / 8 of top-10 FIs claim | ⚠ third-party; consistent with RELX (2014-era figure) |
| 6 | pitchbook.com FircoSoft profile | Watch-list filtering positioning | ⚠ snippet only |
| 7 | fircosoft.com (now LNRS Financial Crime Compliance hub) | Branding (LexisNexis Risk Solutions); 45 of top-50 banks; WorldCompliance Data; OFAC/PATRIOT Act conformity statement | ✅ page extracted; marketing numbers ⚠ |
| 8 | risk.lexisnexis.com/products/firco-continuity | Firco Continuity product | ✅ URL + official snippet via search; direct scrape blocked |
| 9 | risk.lexisnexis.com/global/en/products/firco-compliance-link | Firco Compliance Link product | ✅ URL + official snippet via search; direct scrape blocked |
| 10 | risk.lexisnexis.com/products/firco-trust | FircoTrust product | ⚠ URL exists; scrape blocked |
| 11 | newtonconsultingpartners.com FircoSoft entry | Firco Filtering Suite name | ⚠ third-party database |
| 12 | main.un.org/securitycouncil/en/sanctions/information | UN sanctions regime counts (31 since 1966, 15 active), Art 41/Chapter VII, Consolidated List, committees | ✅ full page extracted |
| 13 | ofac.treasury.gov (SDN human-readable lists page) | OFAC list site; SDN list name | ⚠ title-level; JS app returned "Loading..." only |
| 14 | sanctionsmap.eu | EU sanctions map | ⚠ title-level; JS app returned no content |
| 15 | swift.com "Guiding principles for screening ISO 20022 payments" | Existence of SWIFT's ISO 20022 screening guidance | ✅ found via search |
| 16 | aml-analytics.com fuzzy-matching guide | OFSI £160k Bank of Scotland transliteration fine | ⚠ industry source |
| 17 | axleruns.com / verifex.dev / ofacscreen.com / sanctionsscreening.io fuzzy-matching articles | Fuzzy-matching techniques (Jaro-Winkler, Levenshtein, Soundex, trigrams, transliteration) | ✅ consistent technique set |
| 18 | facctum.com EU sanctions screening article | EU package regime ("21st package") | ⚠ single industry source |
| 19 | regtech_guide.md (repo) | AML-KYC stack §4 (screening, false positives, sanctions compliance), vendors §5, technology §7 | ✅ read this pass; cross-referenced |
| 20 | mas_regulations_guidelines_guide.md (repo) | Notice 626 sanctions screening; MAS designated lists; instrument pyramid | ✅ read this pass; cross-referenced |
| 21 | financial_fraud_detection_at_scale_guide.md (repo) | Latency/FP targets §1; case management §14 | ✅ read this pass; cross-referenced |
| 22 | financial_risk_compliance_systems_guide.md (repo) | Sanctions-screening vendor table (World-Check/OneSumX, Bridger, Fircosoft, Accuity, SAS); Notice 612 | ✅ grepped this pass; cross-referenced |
| 23 | payment_rails_guide.md (repo) | ISO 20022 screening-quality claims; compliance overlay | ✅ grepped this pass; cross-referenced |
| 24 | banks_in_singapore_guide.md (repo) | ABS (est. 1973 ⚠), market structure | ✅ read §8/glossary this pass; cross-referenced |
| 25 | dbs_software_systems_guide.md; standard_chartered_guide.md (repo) | "Firco-class screening is industry standard" inferences (⚠ there) | ✅ grepped this pass; cross-referenced |
| 26 | asset_management_alternatives_guide.md (repo) | Fircosoft/World-Check/Dow Jones AML vendor mention | ✅ grepped this pass; cross-referenced |
| 27 | ai_genai_banking_compliance_guide.md (repo) | AI compliance layer (FEAT/AIRM) | ✅ known cross-ref; not re-read in detail this pass |

**Budget note:** this pass ran 5 web searches and 5 web extractions (the task's ~10-call budget) plus local repo reads. Where a target page blocked extraction (LNRS product pages, OFAC, EU map), the fact was downgraded to ⚠ rather than guessed; where a sibling guide already carried a verified fact (MAS, ISO 20022), it was cross-referenced rather than re-derived.

---

## 14. References and Further Reading

**Primary web sources retrieved this pass:**

- RELX — "Reed Elsevier acquires FircoSoft, the Paris-based global sanctions screening software group" (29 Sep 2014): https://www.relx.com/media/press-releases/archive/29-09-2014
- RELX (via Library Technology) — "Reed Elsevier announces proposed acquisition of FircoSoft" (3 Sep 2014): https://librarytechnology.org/document/28029
- TMCnet reprint of the 29 Sep 2014 announcement: https://www.tmcnet.com/usubmit/2014/10/02/8047529.htm
- Tracxn FircoSoft profile: https://tracxn.com/d/companies/fircosoft/___ZwMfyCpAh4kag3z04mKwAVOYoSMuj5Jns8YY4m3TGs
- CB Insights FircoSoft profile: https://www.cbinsights.com/company/fircosoft
- PitchBook FircoSoft profile: https://pitchbook.com/profiles/company/59965-93
- LexisNexis Risk Solutions Financial Crime Compliance hub (served at fircosoft.com): https://www.fircosoft.com/
- Firco™ Continuity product page: https://risk.lexisnexis.com/products/firco-continuity
- Firco Compliance Link product page: https://risk.lexisnexis.com/global/en/products/firco-compliance-link
- FircoTrust product page (scrape-blocked): https://risk.lexisnexis.com/products/firco-trust
- Newton Consulting Partners FircoSoft entry (Firco Filtering Suite): https://www.newtonconsultingpartners.com/fintech-companies-database/fircosoft
- UN Security Council — Sanctions: https://main.un.org/securitycouncil/en/sanctions/information
- OFAC — Sanctions List Site (SDN human-readable lists): https://ofac.treasury.gov/specially-designated-nationals-and-blocked-persons-list-sdn-human-readable-lists
- EU Sanctions Map: https://www.sanctionsmap.eu/#/main
- SWIFT — "Guiding principles for screening ISO 20022 payments": https://www.swift.com/swift-resource/251416/download
- AML Analytics — "Fuzzy Matching in AML Screening: Validation Guide" (OFSI/Bank of Scotland case): https://aml-analytics.com/2026/04/21/fuzzy-matching-aml-screening-validation-guide/
- Axleruns — "Fuzzy Name Matching in Sanctions Screening, Explained": https://www.axleruns.com/post/fuzzy-name-matching-sanctions-screening
- Verifex — "Fuzzy Matching for Sanctions Screening": https://verifex.dev/blog/fuzzy-matching-sanctions-screening
- Facctum — "EU Sanctions Compliance: What the 21st Package Changed": https://www.facctum.com/blog/swift-and-iso-20022-screening

**Repo sibling guides (cross-referenced, same folder unless noted):**

- [RegTech](regtech_guide.md) — the AML-KYC stack (§4), false positives (§4.7), vendors (§5)
- [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) — sanctions-screening vendor table
- [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) — Notice 626, MAS designated lists
- [Banks in Singapore](banks_in_singapore_guide.md) — the market, the ABS
- [Payment Rails](payment_rails_guide.md) — SWIFT / ISO 20022 screening overlay
- [Financial Fraud Detection at Scale](financial_fraud_detection_at_scale_guide.md) — metrics and case management
- [AI/GenAI Banking Compliance](ai_genai_banking_compliance_guide.md) — the AI layer over screening
- [DBS Software Systems](dbs_software_systems_guide.md) and [Standard Chartered](standard_chartered_guide.md) — Firco-class screening inferences (⚠ there)
- [Asset Management & Alternatives](asset_management_alternatives_guide.md) — Fircosoft/World-Check/Dow Jones vendor mention
- [../technology/event_stream_processing_guide.md](../technology/event_stream_processing_guide.md) — the streaming substrate for real-time screening

**Reading path:** start with this guide's §1–§2 (the discipline and the company), read §3–§4 for the capabilities and the regimes, §5–§6 for the process and the competition, then the Cymbal Bank estate (§7) and the worked-example design (§8); the audit (§11), unverifiable list (§12) and ledger (§13) tell you exactly what is and is not sourced.

---

*End of guide. Companion reading: [RegTech](regtech_guide.md) (the stack), [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) (the estate), [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) (the rulebook), [Payment Rails](payment_rails_guide.md) (the rails the screening sits on). The sanctions-screening discipline in one line: match every name, trust no spelling, update the lists the same day the regulator does, and let nothing but the clean list onto the payment path.*
