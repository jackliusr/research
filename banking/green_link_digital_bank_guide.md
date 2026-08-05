# Green Link Digital Bank (GLDB): A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Crédit Agricole CIB, Singapore  
> **Context:** Digital Banking / Banking Architecture — GLDB Verification, Singapore MAS Digital Bank Licences, Digital Wholesale Banking, MSME Supply Chain Finance, Cloud-Native Core Banking, Temenos + Huawei Cloud 技术栈  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** August 2026

---

## Table of Contents

1. [Verification: What "Green Link Digital Bank" Actually Is](#1-verification-what-green-link-digital-bank-actually-is)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [Verified Status at a Glance](#12-verified-status-at-a-glance)
   - 1.3 [Evidence Trail: Primary and Secondary Sources](#13-evidence-trail-primary-and-secondary-sources)
   - 1.4 [Correcting Two Common Errors](#14-correcting-two-common-errors)
   - 1.5 [What Could NOT Be Verified](#15-what-could-not-be-verified)
2. [The GLDB Profile as Verified](#2-the-gldb-profile-as-verified)
   - 2.1 [Consortium and Shareholders](#21-consortium-and-shareholders)
   - 2.2 [Name, Motto, and Brand](#22-name-motto-and-brand)
   - 2.3 [Licence Application History (2019–2022)](#23-licence-application-history-20192022)
   - 2.4 [Intended Business Model](#24-intended-business-model)
   - 2.5 [Technology Stack: Temenos Core on Huawei Cloud](#25-technology-stack-temenos-core-on-huawei-cloud)
   - 2.6 [Leadership](#26-leadership)
   - 2.7 [Products and Services](#27-products-and-services)
   - 2.8 [Current Status (as of August 2026)](#28-current-status-as-of-august-2026)
3. [The Broader Verified Context: Singapore's Digital Banking Wave](#3-the-broader-verified-context-singapores-digital-banking-wave)
   - 3.1 [The MAS Digital Bank Framework (2019–2020)](#31-the-mas-digital-bank-framework-20192020)
   - 3.2 [The Licence Awards: Four Winners (Dec 2020) and a Fifth Bank (Trust)](#32-the-licence-awards-four-winners-dec-2020-and-a-fifth-bank-trust)
   - 3.3 [The Digital Banks at a Glance](#33-the-digital-banks-at-a-glance)
   - 3.4 [The Journeys Since Launch (2022–2026)](#34-the-journeys-since-launch-20222026)
   - 3.5 [Profitability: The Hard Reality](#35-profitability-the-hard-reality)
   - 3.6 [Significance for the Banking Ecosystem](#36-significance-for-the-banking-ecosystem)
4. [Lessons for Banking Architects](#4-lessons-for-banking-architects)
   - 4.1 [Licensing Strategy: Consortium Design and Licence Type](#41-licensing-strategy-consortium-design-and-licence-type)
   - 4.2 [Cloud-Native Core Selection: The GLDB Pattern](#42-cloud-native-core-selection-the-gldb-pattern)
   - 4.3 [Build vs. Buy vs. BaaS](#43-build-vs-buy-vs-baas)
   - 4.4 [Regulatory Expectations: MAS TRM, FEAT, Outsourcing, Data Residency](#44-regulatory-expectations-mas-trm-feat-outsourcing-data-residency)
   - 4.5 [The SME Banking Opportunity](#45-the-sme-banking-opportunity)
   - 4.6 [Counter-Lessons: What GLDB Also Teaches](#46-counter-lessons-what-gldb-also-teaches)
5. [Honest Limitations](#5-honest-limitations)
   - 5.1 [Claims-Status Table](#51-claims-status-table)
   - 5.2 [How to Read This Guide](#52-how-to-read-this-guide)
6. [References](#6-references)

---

## 1. Verification: What "Green Link Digital Bank" Actually Is

### 1.1 The Short Answer

**Green Link Digital Bank (GLDB) is a real, MAS-licensed, operating digital wholesale bank in Singapore.** It was one of the **four successful applicants** in the Monetary Authority of Singapore's (MAS) December 2020 digital bank licence round — *not* an unsuccessful applicant, and *not* a phantom or unverifiable entity.

The verified facts, in brief:

- **Legal/operating status:** Licensed digital wholesale bank (DWB), regulated by MAS under the Banking Act; commenced banking business on **3 June 2022** and continues to operate (corporate website, internet-banking portal, and social presence active through 2026).
- **Consortium:** Awarded to the **Greenland Group – Linklogis Consortium** — comprising Greenland Financial Holdings Group Co. Ltd, Linklogis Hong Kong Ltd, and Beijing Co-operative Equity Investment Fund Management Co. Ltd — per the MAS press release of 4 December 2020.
- **Focus:** Wholesale banking for micro, small and medium enterprises (MSMEs), built around **supply chain finance**, working capital, payments and deposits; a deliberate Singapore–China trade-corridor play.
- **Technology:** **Temenos core banking** (core, payments, data hub) running on **Huawei Cloud**; implemented by Greenland Digitech in ~11 months; go-live June 2022.
- **The licence narrative matters:** GLDB is frequently omitted from English-language "Singapore digital bank" summaries, which typically list GXS, MariBank, Trust and ANEXT. That list is **incorrect for the December 2020 round**: Trust Bank was *not* among the December 2020 awardees (it received a full bank licence separately and commenced banking business on 15 December 2021), while **GLDB was** the fourth winner. This guide was written precisely to fix that error, which also appears in earlier guides in this repository (see [1.4](#14-correcting-two-common-errors)).

### 1.2 Verified Status at a Glance

| Attribute | Verified status |
|---|---|
| Is GLDB a real entity? | **Yes** — Green Link Digital Bank Pte. Ltd., Singapore UEN **202115950G** (per its internet-banking portal footer) |
| Operating bank, failed applicant, or consortium? | **Operating bank** (digital wholesale bank licence, MAS-regulated) |
| Commencement | **3 June 2022** (bank's own announcement; Straits Times; Huawei Cloud case study) |
| Was it an unsuccessful applicant? | **No** — it was one of the **four successful applicants** announced 4 December 2020 |
| Licence type | **Digital Wholesale Bank (DWB)** — one of two DWB licences awarded in Dec 2020 (the other: ANEXT Bank) |
| Backers | Greenland Group (Chinese state-linked Fortune Global 500 developer) + Linklogis (HK-listed supply chain fintech) + Beijing Co-operative Equity Investment Fund Management |
| Operating as of Aug 2026? | **Yes** — website (glbank.com), ib.glbank.com portal (© 2026), LinkedIn (~30k followers) all live |
| Profitable? | **Not verified as profitable** — reported loss-making in 2023 (Straits Times); no profitability announcement found |

### 1.3 Evidence Trail: Primary and Secondary Sources

Verification was conducted in August 2026 via web search and direct retrieval of primary/authoritative pages. The evidence hierarchy:

1. **MAS press release, 4 December 2020** — "MAS Announces Successful Applicants of Licences to Operate New Digital Banks in Singapore" (full text retrieved via The Asian Banker's republish). This is the *primary regulatory source* establishing the four winners and GLDB's place among them. It names, under Digital Wholesale Bank: *"A consortium comprising Greenland Financial Holdings Group Co. Ltd, Linklogis Hong Kong Ltd, and Beijing Co-operative Equity Investment Fund Management Co. Ltd."* It also states MAS had earlier announced it would award **up to two DFB and up to three DWB licences**, and that there were **14 eligible applications** (the DWB count; 7 DFB applications are widely reported, giving 21 in total — see [5.1](#51-claims-status-table)).
2. **GLDB's own corporate pages** (glbank.com): "Our Heritage" page states GLDB "received a Digital Wholesale Bank licence from the Monetary Authority of Singapore in December 2020"; the launch announcement (June 2022) states commencement of banking business on 3 June 2022 to serve MSMEs.
3. **Temenos success story** (vendor case study): confirms Temenos core banking + Temenos Payments + Temenos Data Hub on Huawei Cloud; ~11-month implementation by Greenland Digitech; "first digital-only bank in Singapore" claim.
4. **Huawei Cloud customer case study**: confirms Huawei Cloud as the platform (ECS, CCE/CCI containers, RDS for MySQL/PostgreSQL, MapReduce, active-active Tier-3 data centres, data-residency controls) and MAS compliance certification.
5. **Contemporary and current press**: Straits Times ("Green Link launches as Singapore's first digital bank", June 2022; ANEXT/SME article confirming GLDB loss-making in 2023), CNA IN FOCUS (5 Feb 2025, five digital banks, FY2023 losses, profitability timelines), Fintech News Singapore, Yahoo Finance/Reuters wire, Proxtera (ANEXT soft-launch date), NTUC (Trust launch), Trust Bank Corporate Governance report (Trust's own licensing timeline).
6. **Secondary/aggregator sources (flagged as lower confidence):** Statrys (ownership 80/20 split; five-bank taxonomy), The Kopinotes, Aspire blog reviews (product details), Singapore Business Review/Asian Banking & Finance interview with GLDB founding CEO (strategy detail).

Where sources disagree or a claim is unverified, this guide says so explicitly and records it in the claims-status table ([5.1](#51-claims-status-table)).

### 1.4 Correcting Two Common Errors

**Error 1 — "The four December 2020 winners were GXS, MariBank, Trust and ANEXT."** This widely repeated list is wrong. The MAS press release of 4 December 2020 names:

| Licence | Awardee | Backers |
|---|---|---|
| Digital Full Bank (DFB) | GXS Bank | Grab Holdings + Singapore Telecommunications (Singtel) |
| Digital Full Bank (DFB) | MariBank | Sea Limited (wholly-owned entity) |
| Digital Wholesale Bank (DWB) | Green Link Digital Bank (GLDB) | Greenland Financial Holdings + Linklogis HK + Beijing Co-operative Equity Investment Fund Management |
| Digital Wholesale Bank (DWB) | ANEXT Bank | Ant Group (wholly-owned entity) |

**Trust Bank was not in this batch.** Trust Bank Singapore Limited was incorporated on 8 December 2020 (four days after the awards), holds a **full bank licence** (not a digital-bank-framework licence), and — per its own corporate governance report — **commenced banking business on 15 December 2021**, launching publicly on 1 September 2022. It is a fifth digital bank in the market, but its licence did not come through the December 2020 digital bank framework round. (Sources: MAS PR via The Asian Banker; Trust Bank CG report; NTUC launch release; CNA.)

**Error 2 — repository guides miscount the licencees.** The earlier repository guides contain the same misconception:

- [core_banking_systems_guide.md](core_banking_systems_guide.md) §8.5/§9 (Singapore context) states the December 2020 four were "GXS Bank (Grab + Singtel) and MariBank (Sea/Shopee) as digital full banks; **Trust Bank** (Standard Chartered + FairPrice Group) and **ANEXT Bank** (Ant Group) as digital wholesale banks". This omits GLDB and wrongly places Trust in the round.
- [programmable_business_bank_guide.md](programmable_business_bank_guide.md) §13 states MAS awarded digital full-bank licences to "**Trust** (Standard Chartered-led consortium) and **GXS** (Grab/Singtel), and digital wholesale-bank licences to **MariBank** (Sea/Shopee) and **ANEXT** (Ant Group)" — both halves are wrong relative to the MAS press release (Trust was never a Dec-2020 digital-bank-framework awardee; MariBank is a full bank, not wholesale).

Readers should treat this guide's Section 1 as the correction, backed by the MAS press release. The correct five-bank landscape (per CNA, Statrys, and the banks themselves, as of 2025–2026):

1. **GXS Bank** — DFB (Dec 2020), Grab + Singtel, consumer launch Aug 2022
2. **MariBank** — DFB (Dec 2020), Sea Limited, launched 2022
3. **Green Link Digital Bank (GLDB)** — DWB (Dec 2020), Greenland + Linklogis, commenced 3 Jun 2022
4. **ANEXT Bank** — DWB (Dec 2020), Ant Group, commenced 2 Jun 2022
5. **Trust Bank** — Full bank licence (commenced business 15 Dec 2021), Standard Chartered + FairPrice Group, public launch 1 Sep 2022

### 1.5 What Could NOT Be Verified

Honest negatives — things this guide could not confirm despite searching:

- **Exact shareholding percentages.** Statrys reports "80% Greenland Group / 20% Linklogis", but no primary source (ACRA filing, GLDB annual report) was retrieved. The MAS press release lists *three* consortium members, so the treatment of Beijing Co-operative Equity Investment Fund Management Co. Ltd within the capital structure is unknown.
- **GLDB's financial statements.** FY2023/FY2024 loss or balance-sheet figures for GLDB specifically were not found in retrievable sources (Straits Times confirms it was loss-making in 2023 but gives no figures; its annual reports are not freely accessible).
- **Customer and loan book scale.** No reliable public figures for GLDB deposits, loans, or customer counts were found.
- **Impact of Greenland Group's real-estate distress.** Greenland Holdings (Chinese property developer) has faced severe debt stress in the 2023–2026 period in public reporting, but **no source was found linking this to GLDB's operations, capital, or MAS standing**. Any speculation on this is out of scope and unverified.
- **CEO succession timeline.** The founding CEO at launch was Goh Soon Hong (2022 interviews); later news (ST School Pocket Money Fund partnership) names Melvin Teo as CEO. The exact transition date and current board composition were not verified.
- **The "first digital bank in Singapore" claim.** GLDB's and Huawei's marketing claim GLDB was "the first digital-only bank in Singapore to launch services" (3 June 2022); ANEXT Bank's soft launch and MAS approval to commence business was 2 June 2022. Both claims circulate; the one-day gap means the "first" title is contested and immaterial.
- **Formal licence-grant dates.** MAS announced the four successful applicants on 4 Dec 2020; the formal grant of each licence followed the applicants meeting prudential pre-conditions. The precise licence-issue date per bank was not verified (a secondary source claims the first DFB licences were awarded on 4 January 2022; not corroborated).
- **Anything about the *other* unsuccessful applicants.** The ~17 remaining applicants (out of 21) are not covered; GLDB's status as a winner makes the "who didn't win" question secondary.

---

## 2. The GLDB Profile as Verified

### 2.1 Consortium and Shareholders

GLDB's licence was awarded to the **Greenland Group – Linklogis Consortium**. The MAS press release names three members:

1. **Greenland Financial Holdings Group Co. Ltd** — financial arm of **Greenland Group** (Greenland Holdings), the Shanghai state-linked conglomerate best known as a property developer, described by GLDB as "a Fortune Global 500 enterprise" with institutional scale in real estate, infrastructure and large-scale operations.
2. **Linklogis Hong Kong Ltd** — the Hong Kong entity of **Linklogis** (联易融), a Hong Kong-listed supply chain finance technology company (founded by Song Qun). Linklogis is the supply-chain-tech brain of the consortium: its platform digitises trade/supply chain finance, invoice financing and accounts-payable/receivable workflows.
3. **Beijing Co-operative Equity Investment Fund Management Co. Ltd** — a Beijing-based investment fund manager. Its specific role and stake in GLDB could not be verified (see [1.5](#15-what-could-not-be-verified)).

Reported (not primary-verified) shareholding: **80% Greenland Group / 20% Linklogis** per Statrys' 2026 guide. The bank's name itself fuses the two lead shareholders — **Green**land + **Link**logis = **Green Link** — which the founding CEO confirmed in an interview.

The combination was strategic: Greenland brings balance-sheet scale, China-enterprise relationships and a Singapore base; Linklogis brings proven MSME supply chain finance technology and a book of China/ASEAN trade-finance clients. This is a classic *"anchor-capital + fintech-platform"* consortium design (see [4.1](#41-licensing-strategy-consortium-design-and-licence-type)).

### 2.2 Name, Motto, and Brand

- **Name:** Green Link Digital Bank — "Green" from Greenland Group, "Link" from Linklogis (confirmed by founding CEO Goh Soon Hong in a 2022 Asian Banking & Finance interview).
- **Motto:** GLDB adopted the Chinese principle **勤为善小、行无止境** ("Be good. Be diligent. Be persistent.") as its corporate principle — a stated commitment that even the smallest client matters.
- **Brand positioning:** "Business banking for all, made seamless" (LinkedIn); "a regulated digital wholesale bank... bringing banking, payments, financing and working capital together within a single digital platform" (glbank.com).
- **Identity markers:** UEN 202115950G (i.e., the Singapore entity was incorporated in **2021** — consistent with: selection Dec 2020 → incorporation 2021 → licence conditions met → operations June 2022). Wholesale-only: it does **not** take retail deposits and is **not** an SDIC-insured deposit-taking institution for consumers (SDIC coverage applies to the retail DFBs; see [3.1](#31-the-mas-digital-bank-framework-20192020)).

### 2.3 Licence Application History (2019–2022)

The verified timeline:

| Date | Event |
|---|---|
| **Jun 2019** | MAS announces its digital bank framework: eligible non-bank institutions may apply for up to **two digital full bank (DFB)** and up to **three digital wholesale bank (DWB)** licences (5 total; per the Dec 2020 press release's restatement of the plan). Business Times: "Singapore to issue up to five new licences to digital banks". |
| **2019–2020 (window)** | Applications received; widely reported total of **21 applications — 7 for DFB, 14 for DWB** (the MAS release copy retrieved cites "14 eligible applications", consistent with the DWB count). The Greenland–Linklogis consortium applied for a DWB licence. |
| **4 Dec 2020** | MAS announces the **four successful applicants**: GXS Bank and MariBank (DFB); **Green Link Digital Bank (GLDB)** and ANEXT Bank (DWB). MAS notes successful applicants "must meet all relevant prudential requirements and licensing pre-conditions before MAS grants them their respective banking licences", and expects commencement "from early 2022". The third DWB slot is left unfilled. |
| **2021** | GLDB entity incorporated (UEN 202115950G). Implementation of the core platform proceeds (~11 months total, led by Greenland Digitech with Temenos and Huawei). |
| **3 Jun 2022** | GLDB **commences banking business** in Singapore. |
| **2022–present** | Operating; product expansion (accounts, deposits, loans, supply chain finance); open-banking API pilots; still loss-making as of 2023 reporting (see [2.8](#28-current-status-as-of-august-2026)). |

Key point for the repo's earlier guides: GLDB is **not** a footnote to the Singapore digital bank story — it is one of the four founding licencees, the **first (or tied-for-first) wholesale digital bank to actually launch**, and the only one whose consortium is China-state-linked (Greenland) plus a listed fintech (Linklogis).

### 2.4 Intended Business Model

GLDB's model, as stated by the bank, its founders, and its vendor case studies:

- **Segment:** Micro, small and medium enterprises (**MSMEs**) — including "technology innovation enterprises" — in high-growth sectors such as technology and sustainable construction (Temenos case). Focus on businesses with cross-border (Singapore–China corridor) activity.
- **Core proposition:** Supply chain finance and working capital. Per founding CEO Goh Soon Hong: financing that lets buyers extend payment terms while suppliers get paid earlier; cash and payments; loans; trade/supply chain financing; and future **open banking** services for MSMEs (piloted via Temenos core APIs).
- **Why Singapore:** a leading trade and finance hub with high digital penetration, political stability, and a position as ASEAN gateway; the consortium wanted to bring its China MSME-financing experience to Southeast Asia and "facilitate trade along the Singapore–China corridor".
- **Differentiation vs incumbents:** specialist supply chain finance products, plus a lean cloud-native cost base — "implement a lean, agile, and cost-efficient operating model" (Temenos case) — and ABCD+ technology (AI, blockchain, cloud, big data) for credit assessment and trade finance, per founder statements.
- **Wholesale constraint:** as a DWB, GLDB cannot take retail (SGD) deposits or serve consumers; its market is corporates, MSMEs and financial institutions. This is why it is absent from consumer "best digital bank" lists — a recurring source of the misperception that it is a failed or minor applicant.

### 2.5 Technology Stack: Temenos Core on Huawei Cloud

This is the best-documented part of GLDB's profile, thanks to two vendor case studies. Verified stack elements:

**Core banking (Temenos):**
- **Temenos core banking** (the Transact-based core) — system of record for accounts, deposits, loans, and general-ledger/back-office processing.
- **Temenos Payments** — payments processing (relevant to the payments layer; see [payments_hub_guide.md](payments_hub_guide.md)).
- **Temenos Data Hub** — batch scheduling, end-of-day position visibility, back-office workflow efficiency.
- Stated outcomes: product launch cycle reduced from ~6 months to ~1 month (**83% faster**); platform operated with **50% fewer resources** than a typical core; **11-month** implementation; open banking API pilots run on Temenos core APIs.

**Cloud platform (Huawei Cloud):**
- Cloud-native compute: **ECS** (elastic cloud servers) and **CCE/CCI** (Kubernetes-based containers) for rapid cluster expansion and service surges.
- Data: **RDS for MySQL and RDS for PostgreSQL** (open-source-compatible), primary + up to 5 read replicas per instance; **MapReduce** big data platform for analytics (credit decisions, trade data).
- Resilience: Tier-3 data centres, **intra-city active-active**, third-party backups — "failure of a single data centre does not affect system availability".
- Security/governance: E2E HTTPS, **LandingZone** IT governance, Cloud Connect + Cloud Bastion Host for network isolation, **IAM** for identity; **data residency** controls such that "data in Singapore never leaves the country" (cross-border access via network access points only).
- Compliance: Huawei Cloud states GLDB achieved **MAS security compliance certification quickly**; the vendor's compliance tooling (baselines, solution packages, monitoring) was cited as a key reason for selection.
- Ops: 24/7 bilingual (Chinese/English) support; Direct Connect / Cloud Connect for global VPC interconnection (China–Singapore cross-border O&M).

**Stack rationale (as documented in the Huawei case):** GLDB's stated selection criteria were (1) meeting MAS compliance quickly, (2) flexible, scalable, highly available cloud-native architecture, (3) **technological neutrality / avoiding vendor lock-in** and open-source compatibility for an open banking ecosystem, and (4) bilingual cross-border service. Temenos was selected after "assessing core banking systems from international and local vendors".

**Interpretation for architects:** GLDB is a textbook *commercial-off-the-shelf core (Temenos) on a hyperscaler (Huawei Cloud), with the fintech partner (Linklogis) supplying the differentiated supply-chain layer* — i.e., buy the core, differentiate in the product layer. The "ABCD+" (AI, Blockchain, Cloud, Big Data) framing from the founders sits on top of this: Linklogis-type supply chain finance logic, data-driven credit, and (eventually) blockchain for trade documents. See [4.2](#42-cloud-native-core-selection-the-gldb-pattern) and the vendor landscape in [core_banking_systems_guide.md](core_banking_systems_guide.md) (§5).

### 2.6 Leadership

| Person | Role (verified) |
|---|---|
| **Goh Soon Hong** | Founding Executive Director & CEO (2022). Former DBS banker — Head of Sales & Distribution and Country Head of Medium Enterprise, SME Banking at DBS. |
| **Song Qun (宋群)** | Linklogis founder, chairman & CEO; **Vice-Chairman of GLDB**. Stated GLDB's ABCD+ technology and ESG ambitions at launch. |
| **Kuang Mi** | CEO of Lianqi Cloud and Head of IT at **Greenland Digitech** (described by Temenos as parent company of GLDB); led the core-platform implementation. |
| **Melvin Teo** | CEO per later news coverage (ST School Pocket Money Fund partnership announcement). Succession date unverified. |

The leadership mix mirrors the consortium: DBS SME-banking experience (Goh), Linklogis supply-chain fintech founders (Song), and Greenland's technology arm (Kuang Mi / Greenland Digitech).

### 2.7 Products and Services

Verified from GLDB's site, its launch materials, and 2025–2026 review coverage (product details from third-party reviews are flagged as lower-confidence in [5.1](#51-claims-status-table)):

- **Business current accounts** (multi-currency, incl. SGD and foreign-currency accounts).
- **Fixed deposits** — SGD and USD.
- **Business loans** — SME financing, with 2026 review coverage citing term loans up to ~S$300,000.
- **Supply chain finance** — the flagship: receivables/payables finance, invoice-based working capital, digital supply chain financing for MSMEs (leveraging Linklogis technology).
- **Payments** — domestic and cross-border payment capabilities (via Temenos Payments; wholesale/corporate focus).
- **Open banking APIs** — piloted with clients for MSME open banking services (per Temenos case study).
- No retail/consumer products by design (DWB licence constraint).

### 2.8 Current Status (as of August 2026)

- **Operating:** glbank.com live; internet banking portal (ib.glbank.com) live with © 2026 branding; LinkedIn presence ~30k+ followers; active product marketing (current-account promos, fixed deposit rates, SME loans) through 2025–2026.
- **Financials:** **Loss-making in FY2023** (Straits Times, via its ANEXT/SME ecosystem article: "like Anext, it was also loss-making in 2023", while "growing its operating income rapidly"). No FY2024/FY2025 results or profitability announcement were found; GLDB-specific loss figures are unverified.
- **Positioning:** One of two wholesale digital banks (with ANEXT) serving commercial customers; the CNA (Feb 2025) landscape piece lists GLDB and ANEXT as the two DWBs "serving commercial customers in Singapore", with GXS/MariBank/Trust moving into SME segments from the retail side.
- **Community:** Partnership/donation with ST School Pocket Money Fund (S$50,000) — signal of ongoing local operations and brand-building.
- **Watch items (unverified, flagged):** Greenland Group's property-sector distress and any capital-support implications for GLDB; CEO-transition details; scale metrics. See [1.5](#15-what-could-not-be-verified).

---

## 3. The Broader Verified Context: Singapore's Digital Banking Wave

### 3.1 The MAS Digital Bank Framework (2019–2020)

Singapore's digital bank experiment was announced by MAS in **June 2019** as part of its push to keep the banking system competitive and tech-forward. Verified framework elements:

- **Two licence classes:**
  - **Digital Full Bank (DFB):** can take retail deposits and offer consumer banking. Starts as a **Restricted DFB** with minimum paid-up capital of **S$15 million**, with restrictions (e.g., deposit caps) while it builds out; must progress to the full **S$1.5 billion** paid-up capital requirement within **3–5 years** of commencing business, plus full liquidity requirements (100% NSFR, 100% LCR). DFBs are members of the **SDIC Deposit Insurance Scheme** (S$100,000 coverage per depositor).
  - **Digital Wholesale Bank (DWB):** serves corporates, MSMEs and financial institutions; **cannot accept retail SGD deposits** (no SDIC-insured retail deposits). Minimum paid-up capital **S$100 million** (consistent with the Banking Act wholesale licence threshold).
- **Allocation plan:** up to **two DFBs and three DWBs** (five licences in total) per the December 2020 press release's restatement.
- **Application pool:** **21 applications** (7 DFB, 14 DWB) from a wide field of tech companies, incumbents' affiliates, and consortia.
- **Selection criteria (as publicly framed by MAS):** business model strength, value proposition to the market, ability to meet prudential standards, technological capability, and growth plans — MAS explicitly wanted applicants that would raise the bar on innovation and financial inclusion, with a credible path to profitability.

The framework deliberately created a *permissioned lab* for cloud-native banking: new entrants without legacy cores, competing with the big three (DBS, OCBC, UOB) and the international banks. For the banking-architecture context, see the Singapore section of [core_banking_systems_guide.md](core_banking_systems_guide.md) and the programmable-banking discussion in [programmable_business_bank_guide.md](programmable_business_bank_guide.md).

### 3.2 The Licence Awards: Four Winners (Dec 2020) and a Fifth Bank (Trust)

**4 December 2020** — MAS announced the four successful applicants (see the table in [1.4](#14-correcting-two-common-errors)). The third DWB slot was left unfilled; MAS has not issued further digital bank licences since (all current guides and 2025–2026 reporting count five digital banks in the market).

**The fifth digital bank, Trust Bank**, came through a different door: incorporated **8 December 2020**; granted a **full bank licence**; commenced banking business **15 December 2021**; launched publicly **1 September 2022**. Trust is 60% Standard Chartered Bank (Singapore) Limited / 40% FairPrice Group (NTUC's enterprise arm) — a consumer-focused bank leveraging the FairPrice ecosystem. Because it holds a full bank licence rather than a digital-bank-framework licence, it is exempt from the DFB deposit cap and is the only digital bank that can offer a full suite of retail products out of the box (SDIC-insured, full credit card, etc.). Its licence history is frequently conflated with the Dec-2020 round (see [1.4](#14-correcting-two-common-errors)).

**Why the conflation happens:** press coverage in late 2022 ("Singapore's first digital bank" race) centred on consumer-facing Trust, GXS and MariBank; GLDB and ANEXT, being wholesale and business-focused, got far less coverage. The "four winners" list then drifted to the four most visible consumer names — but the *licensed* four are GXS, MariBank, GLDB and ANEXT.

### 3.3 The Digital Banks at a Glance

| Bank | Licence (award) | Backers | Commencement | Segment focus | SDIC retail deposits |
|---|---|---|---|---|---|
| **GXS Bank** | DFB (Dec 2020) | Grab + Singtel | Consumer launch Aug 2022 | Retail (gig-economy, ecosystem data); expanding to SME | Yes |
| **MariBank** | DFB (Dec 2020) | Sea Limited | Launched 2022 | Retail (Shopee ecosystem); SME loans; Mari Invest wealth | Yes |
| **Trust Bank** | Full bank (commenced 15 Dec 2021) | Standard Chartered (60%) + FairPrice Group (40%) | Public launch 1 Sep 2022 | Consumer everyday banking; FairPrice ecosystem | Yes |
| **ANEXT Bank** | DWB (Dec 2020) | Ant Group (wholly-owned) | 2 Jun 2022 (soft launch) | SMEs, cross-border trade, MSME financing | No |
| **Green Link Digital Bank (GLDB)** | DWB (Dec 2020) | Greenland + Linklogis + Beijing Co-op Fund | **3 Jun 2022** | MSMEs, supply chain finance, SG–China corridor | No |

(Trust's public launch: NTUC release, 1 Sep 2022. ANEXT's MAS approval to commence: 2 June 2022 per Proxtera. GLDB: 3 June 2022 per the bank and Straits Times.)

### 3.4 The Journeys Since Launch (2022–2026)

**GXS Bank (Grab + Singtel).** Launched consumer savings in Singapore in August 2022 (the first DFB to launch), leveraging Grab and Singtel ecosystems ("one in two sole proprietorships has a relationship with Grab, Singtel, or both", per CNA). Later expanded into the commercial segment: deposit accounts and loan products for small businesses (starting with sole proprietors), using ecosystem cash-flow data for credit assessment. Also launched Malaysia's first digital bank (GXBank) via a separate consortium in 2023. FY2023 loss: **S$152.1 million**; target: break even by **end-2026**.

**MariBank (Sea).** Sea's digital bank launched in 2022, initially for Shopee ecosystem users and sellers. First mover into wealth: **Mari Invest** (Oct 2023), a fee-free, no-lock-in investment account with instant cash-out — AUM surpassed **S$953 million** at end-2024. Offers SME deposit accounts and short/long-term business loans. FY2023 loss: **S$52.2 million**. Plans for 2025–2026: wealth expansion, payments, personal loans (invite-only rollout).

**Trust Bank (SCB + FairPrice).** Public launch 1 Sep 2022 as "Singapore's first digital bank" (marketing; technically the first consumer digital bank with a full licence). Consumer products: savings, credit card (including the numberless card), insurance. Leveraged FairPrice Group's member base for distribution. FY2023 loss: **S$128.4 million** (smallest increase, +3%). **Expects profitability around end-2025** (stated in early 2025); rolling out TrustInvest (wealth) in 2025.

**ANEXT Bank (Ant Group).** Wholly-owned Ant entity; commenced 2 June 2022 (soft launch) — the first digital wholesale bank to start operations, one day before GLDB. Focus: SMEs, cross-border/MSME financing, trade finance for Asia. Received additional capital injection of ~**S$200 million (≈US$148.5M)** from Ant Group (reported 2023–2024). Loss-making in 2023 but growing operating income rapidly (Straits Times).

**Green Link Digital Bank (GLDB).** Commenced 3 June 2022; MSME/supply-chain-focus wholesale bank (Section 2). Loss-making in 2023, growing operating income (Straits Times); no public profitability date.

### 3.5 Profitability: The Hard Reality

Verified landscape numbers (CNA IN FOCUS, 5 Feb 2025):

- **All three retail-facing digital banks were loss-making in FY2023:** GXS **S$152.1M** loss (double-digit % increase YoY), MariBank **S$52.2M** loss, Trust **S$128.4M** loss (+3% YoY).
- Positive signals: net interest income growing at all three (GXS and Trust "multiple folds"; MariBank flipped to positive NII).
- The structural problem: **Singapore has <2% unbanked adults** — digital banks cannot grow by banking the unbanked; they must win deposits from entrenched incumbents (DBS/OCBC/UOB at record profits) via rates, UX, and ecosystems.
- Benchmarks: Hong Kong's ZA Bank reported its **first monthly net profit in July 2024, ~4 years after launch** — the first digital bank in Hong Kong to do so. Analysts (Bain, FIS, Quinlan & Associates) expect Singapore digital banks to take "a few more years".
- Public targets: Trust ~end-2025; GXS end-2026; MariBank "sustainable growth" without a date; GLDB/ANEXT no public targets found.
- Deposit strategy is the battleground: payroll capture, ecosystem deposits (FairPrice for Trust, Shopee for MariBank, Grab/Singtel for GXS), and sticky retail/wealth products.

### 3.6 Significance for the Banking Ecosystem

- **The licensed beachhead of cloud-native banking in a mature market.** Singapore's five digital banks (four under the 2020 framework + Trust) proved that a bank can be built from scratch on a cloud-native core in under a year of implementation and launch in ~18 months from selection (GLDB: Dec 2020 → Jun 2022; ANEXT and Trust similar). That timeline benchmark resets expectations for incumbents' modernization (see [core_banking_systems_guide.md](core_banking_systems_guide.md) §8 on modernization and §8.5 on the digital bank case).
- **Supply chain finance for MSMEs moved to the banking mainstream.** GLDB (Linklogis tech) and ANEXT (Ant tech) made digital supply chain finance a licensed-bank product in Singapore, not just a fintech offering — directly relevant to the trade-finance and payments discussion in [payments_hub_guide.md](payments_hub_guide.md).
- **Regulatory architecture as product strategy.** MAS's two-tier licence design (retail vs wholesale) plus the later full-bank route (Trust) shows how licence choice determines the entire business model and tech perimeter (SDIC membership, deposit caps, capital trajectory S$15M→S$1.5B).
- **Data residency and cross-border banking.** GLDB's China–Singapore design (Huawei Cloud with in-country data residency, cross-border access via network endpoints) is an early, working template for banks serving the SG–China corridor — the same corridor that drives much of Singapore's trade-finance volume.
- **The "unbanked" lesson for Asia.** Singapore's <2% unbanked rate means digital banks there must *win* customers; the same playbook (ecosystem data, SME supply chain) is being exported to Malaysia, Indonesia, Thailand as those regulators issue digital bank licences — GLDB's founding CEO explicitly cited ASEAN expansion as the growth horizon.

---

## 4. Lessons for Banking Architects

GLDB's trajectory — selection (Dec 2020), eleven-month implementation, June 2022 go-live, and five years of operation as a licensed wholesale digital bank — is a compact, well-documented case study in how a digital bank is actually built. This section distils the architect-relevant lessons, cross-referenced to the sibling guides in this repository.

### 4.1 Licensing Strategy: Consortium Design and Licence Type

The licence decision is the *first* architecture decision — it determines the business model, the customer perimeter, the regulatory capital trajectory, and therefore the entire technology boundary.

- **Consortium design — "anchor capital + fintech platform".** GLDB paired Greenland Group (balance sheet, China-enterprise relationships, Singapore presence) with Linklogis (supply chain finance technology, MSME trade-finance clients). The architecture consequence: the core is generic (Temenos), and the differentiation lives in the partner's platform layer. Compare this with the single-sponsor model of ANEXT (Ant wholly-owned) and MariBank (Sea wholly-owned) — simpler governance, but the consortium brought GLDB a ready-made trade corridor.
- **Licence type sets the tech perimeter.** As a Digital Wholesale Bank, GLDB:
  - cannot accept retail SGD deposits → no SDIC-insured retail deposit infrastructure, no consumer channels (contrast the DFBs: S$15M restricted capital rising to S$1.5B, SDIC membership, deposit caps);
  - targets corporates/MSMEs → the platform is built around account, payments, and lending APIs for businesses, not consumer onboarding.
  - **Architect lesson:** define the licence perimeter *before* the target architecture — retail vs wholesale changes channels, onboarding/KYC, deposit systems, and reporting obligations more than any technology choice does.
- **The timeline benchmark.** Selection Dec 2020 → incorporation 2021 → commencement Jun 2022 (with ANEXT one day earlier). An ~18-month selection-to-launch, with an ~11-month core implementation, is now the demonstrated benchmark for a from-scratch digital bank in Singapore. See [core_banking_systems_guide.md](core_banking_systems_guide.md) §8 (modernization) for how this benchmark resets expectations for incumbents.

### 4.2 Cloud-Native Core Selection: The GLDB Pattern

GLDB's stack is a textbook **COTS-core-on-hyperscaler** pattern:

- **Core:** Temenos core banking (Transact-based) + Temenos Payments + Temenos Data Hub — the system of record for accounts, deposits, loans, and GL/back-office processing.
- **Cloud:** Huawei Cloud (ECS, CCE/CCI Kubernetes containers, RDS MySQL/PostgreSQL with up to 5 read replicas, MapReduce analytics), Tier-3 data centres, intra-city active-active, third-party backups.
- **Differentiation layer:** Linklogis supply chain finance logic, data-driven credit, ABCD+ (AI, Blockchain, Cloud, Big Data) framing — the product layer that the COTS core does not provide.

**Why this pattern won the selection (per the documented criteria):**
1. meeting MAS compliance quickly (vendor compliance tooling/baselines);
2. flexible, scalable, cloud-native architecture;
3. technological neutrality / avoiding vendor lock-in and open-source compatibility for the future open-banking ecosystem;
4. bilingual (Chinese/English) cross-border service and support.

**Architect lessons:**
- **Buy the core, differentiate in the product layer.** The GLDB pattern confirms the composable-banking thesis (see [core_banking_systems_guide.md](core_banking_systems_guide.md) §6): a modern core plus a domain-specific fintech layer beats building a core in-house for a new entrant.
- **Implementation speed is a vendor-selection criterion.** An 11-month go-live (via Greenland Digitech as SI) was only possible with a configurable COTS core on a cloud platform with managed services. Custom cores or heavy on-prem estates cannot reproduce this timeline.
- **Data residency is a first-class requirement.** "Data in Singapore never leaves the country" with cross-border access via network access points only — a working template for the SG–China corridor and for MAS expectations (see [financial_infrastructure_guide.md](financial_infrastructure_guide.md) and the data-residency discussion in the on-premises deployment guide).
- **Avoiding lock-in was explicit.** Open-source-compatible data layers (RDS MySQL/PostgreSQL) and container-native compute were deliberate choices to keep the option of portability — a useful counterpoint to the "just use the vendor's managed everything" reflex.

### 4.3 Build vs. Buy vs. BaaS

GLDB is a **buy + partner** case: COTS core (Temenos) on hyperscaler (Huawei Cloud) with a fintech partner (Linklogis) supplying the differentiated layer, implemented by the consortium's own SI (Greenland Digitech).

| Option | GLDB-equivalent | Trade-off (per the GLDB evidence) |
|---|---|---|
| **Build in-house** | Not chosen | Highest cost/risk; ~11-month go-live impossible; core banking is a solved problem for a new entrant |
| **Buy (COTS core + cloud)** | GLDB's choice | Proven, fast (11 months), compliance accelerators; cost of licensing + SI; configuration over code |
| **BaaS (partner bank licence)** | Not chosen (they pursued their own licence) | Fastest to market, least control; GLDB's consortium had both the capital and the regulator-appetite to go licensed |

**Architect lesson:** the build-vs-buy-vs-BaaS decision (see [programmable_business_bank_guide.md](programmable_business_bank_guide.md) §9 for the full framework) is dominated by *licensing intent*: if the consortium intends to be a licensed bank (as GLDB did), BaaS is off the table for the core; the real decision is build vs buy, and the evidence strongly favours buy for new entrants.

### 4.4 Regulatory Expectations: MAS TRM, FEAT, Outsourcing, Data Residency

GLDB's documented compliance journey maps directly onto the MAS framework covered elsewhere in this repository:

- **MAS TRM (Technology Risk Management).** The security-testing, availability, and cyber-resilience expectations (see the PTES guide's MAS TRM discussion and [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md)) were met with cloud-provider compliance tooling — baselines, solution packages, monitoring — which GLDB explicitly cited as a selection driver. The lesson: *regulatory compliance is a procurement criterion, not an afterthought.*
- **Outsourcing.** The Temenos + Huawei Cloud + Greenland Digitech stack is a multi-party outsourcing structure (core vendor, cloud provider, SI). MAS outsourcing guidelines require oversight of material outsourcing arrangements — a governance burden the consortium had to stand up from day one (see the vendor-management discussion in the risk & compliance guide).
- **Data residency.** Huawei Cloud's in-country data-residency controls ("data in Singapore never leaves the country") directly address MAS expectations on data localisation for customer data.
- **FEAT (Fair Dealing).** As a wholesale bank, GLDB's fair-dealing obligations centre on business customers (transparency of terms, fair treatment of MSME borrowers) rather than retail conduct.

**Architect lesson:** for a new digital bank, compliance architecture (audit trails, data residency, vendor oversight, security baselines) must be designed in the first sprint, not bolted on — GLDB's 11-month timeline only worked because compliance tooling came with the platform choices.

### 4.5 The SME Banking Opportunity

GLDB is the purest expression in Singapore of the **SME/supply-chain-finance thesis**:

- The segment is structurally underserved: MSMEs get less attractive pricing and slower service from incumbents than large corporates; trade finance remains paper-heavy.
- GLDB's stack (Linklogis invoice/receivables digitisation + Temenos core + open-banking API pilots) targets the working-capital gap: let buyers extend payment terms while suppliers get paid earlier.
- The Singapore–China corridor is the wedge: a trade corridor where the consortium's relationships and bilingual operations are a genuine moat.

**Architect lesson:** SME banking is won in the *data layer* — ecosystem and supply-chain data for credit assessment (the same playbook as GXS's Grab/Singtel cash-flow data and MariBank's Shopee seller data). The core is table stakes; the credit-decisioning data platform is the differentiator (see the data-model and analytics guides for the data-platform patterns).

### 4.6 Counter-Lessons: What GLDB Also Teaches

The case is not uniformly positive, and the caveats are as instructive as the successes:

- **Profitability is the unresolved chapter.** Loss-making in FY2023; no public profitability date. In a market with <2% unbanked adults, digital banks must *win* customers from entrenched incumbents. Wholesale digital banks face the additional challenge of a smaller addressable market (no retail deposits) and longer enterprise sales cycles.
- **Consortium risk.** The Greenland Group parent has faced severe real-estate debt distress in 2023–2026 public reporting. No source links this to GLDB's operations or MAS standing, but the *structural* lesson stands: a consortium anchored on a distressed parent carries reputational and capital-support questions that a bank architect should factor into vendor/partner due diligence.
- **Marketing-claim drift.** GLDB and Huawei both claimed "first digital bank in Singapore" (3 Jun 2022) while ANEXT soft-launched 2 Jun 2022 — a one-day gap that makes the claim contested and immaterial. For architects: when evaluating vendor case studies, date-stamp and cross-check "first/only" claims against primary sources.
- **Visibility asymmetry.** GLDB is absent from most consumer "Singapore digital bank" lists precisely because it is wholesale — and that absence feeds the recurring misperception that it is a failed or minor applicant (Section 1.4). The lesson for research discipline: absence from consumer-facing coverage is not evidence of failure for a wholesale bank.

---

## 5. Honest Limitations

### 5.1 Claims-Status Table

| Claim | Status | Source / Basis |
|---|---|---|
| GLDB is a real, MAS-licensed, operating digital wholesale bank | **Verified** | MAS press release 4 Dec 2020; glbank.com; Temenos & Huawei case studies |
| GLDB was one of the four successful Dec-2020 applicants | **Verified** | MAS press release 4 Dec 2020 (primary) |
| Licence type = Digital Wholesale Bank (DWB) | **Verified** | MAS press release 4 Dec 2020 |
| Commenced banking business 3 June 2022 | **Verified** | GLDB launch announcement; Straits Times; Huawei case study |
| Consortium = Greenland Financial Holdings + Linklogis HK + Beijing Co-operative Equity Investment Fund Management | **Verified** | MAS press release 4 Dec 2020 |
| Trust Bank was NOT a Dec-2020 digital-bank-framework awardee; licensed separately, commenced 15 Dec 2021 | **Verified** | Trust Bank corporate governance report; NTUC launch release; CNA |
| Tech stack: Temenos core + Temenos Payments + Temenos Data Hub on Huawei Cloud | **Verified** | Temenos success story; Huawei Cloud case study |
| ~11-month implementation by Greenland Digitech | **Verified** | Temenos success story |
| MSME / supply chain finance focus; Singapore–China corridor | **Verified** | GLDB corporate pages; founding CEO interviews |
| Loss-making in FY2023 | **Verified (no figures)** | Straits Times; CNA IN FOCUS (5 Feb 2025) |
| 80% Greenland / 20% Linklogis shareholding | **Lower confidence** | Statrys (aggregator); no primary source (ACRA filing/annual report) retrieved |
| GXS FY2023 loss S$152.1M; MariBank S$52.2M; Trust S$128.4M | **Verified** | CNA IN FOCUS, 5 Feb 2025 |
| Mari Invest AUM > S$953M end-2024 | **Verified** | MariBank disclosures via press coverage |
| ANEXT capital injection ~S$200M | **Reported** | Press (2023–2024); not cross-checked to a primary filing |
| GLDB-specific financial statements, customer/loan-book scale | **Unverified** | Not publicly retrievable |
| GLDB product details (term loans to ~S$300k etc.) | **Lower confidence** | 2025–2026 third-party reviews (Aspire blog etc.) |
| CEO succession timeline (Goh Soon Hong → Melvin Teo) | **Unverified** | Succession date not found |
| Greenland distress impact on GLDB | **No evidence found** | Absence of sources; explicitly out of scope |
| "First digital bank in Singapore" (GLDB) | **Contested** | ANEXT soft-launch 2 Jun 2022 precedes GLDB's 3 Jun 2022 |
| Formal licence-grant dates per bank (vs award announcement) | **Unverified** | Secondary claim (4 Jan 2022) not corroborated |
| FY2024/FY2025 results or profitability announcement for GLDB | **Unverified** | Not found |

### 5.2 How to Read This Guide

- **Section 1 is the correction.** If you read only one section, read 1.1–1.4: it fixes the recurring "four winners = GXS, MariBank, Trust, ANEXT" error and establishes the verified five-bank landscape.
- **Sections 2–3 are the verified record**, built on primary sources (MAS, the bank itself, vendor case studies) with secondary sources flagged.
- **Section 4 is interpretation** — architect lessons drawn from the verified record, cross-referenced to the sibling guides. Treat the lessons as analysis, not as GLDB-stated facts.
- **Section 5.1 is the confidence map.** Where a claim matters for a decision (vendor selection, market entry, research citation), check the claim's status before relying on it.
- **Verification date:** August 2026. Licence counts, ownership structures, and profitability status evolve; re-verify against MAS and the banks' own disclosures before use in formal work.

---

## 6. References

**Primary / authoritative:**
1. Monetary Authority of Singapore — press release, "MAS Announces Successful Applicants of Licences to Operate New Digital Banks in Singapore", 4 December 2020 (full text via The Asian Banker's republish).
2. Green Link Digital Bank — corporate website (glbank.com): "Our Heritage"; launch announcement (June 2022); internet banking portal (ib.glbank.com).
3. Temenos — success story: Green Link Digital Bank (core banking, payments, data hub on Huawei Cloud; 11-month implementation; product-launch cycle 6→1 month; 50% fewer resources).
4. Huawei Cloud — customer case study: Green Link Digital Bank (ECS, CCE/CCI, RDS MySQL/PostgreSQL, MapReduce, Tier-3 DCs, active-active, data residency, MAS compliance certification).
5. Trust Bank Singapore Limited — Corporate Governance Report (licensing timeline: incorporated 8 Dec 2020; commenced banking business 15 Dec 2021).
6. NTUC / FairPrice Group — launch release (Trust Bank public launch 1 Sep 2022).

**News / analysis:**
7. The Straits Times — "Green Link launches as Singapore's first digital bank" (June 2022); SME/ANEXT ecosystem article (GLDB loss-making in 2023).
8. CNA IN FOCUS — "Five digital banks, one city-state: Singapore's digital banking experiment" (5 Feb 2025) — FY2023 losses (GXS S$152.1M, MariBank S$52.2M, Trust S$128.4M), profitability targets.
9. Asian Banking & Finance / Singapore Business Review — interviews with GLDB founding CEO Goh Soon Hong (2022): name origin, ABCD+ strategy, ASEAN ambitions.
10. Proxtera — ANEXT Bank soft-launch date (2 June 2022).
11. Fintech News Singapore; Yahoo Finance/Reuters wire — digital bank licence round coverage.
12. Statrys (2026 guide) — ownership split (80/20), five-bank taxonomy (lower-confidence aggregator).

**Repository cross-references:**
- [core_banking_systems_guide.md](core_banking_systems_guide.md) — §6 composable banking, §8 modernization, §8.5/§9 Singapore digital bank case (note: §8.5/§9 licence list is corrected by Section 1.4 of this guide).
- [programmable_business_bank_guide.md](programmable_business_bank_guide.md) — §9 build-vs-buy-vs-BaaS, §13 MAS digital bank licences (note: §13 licence list is corrected by Section 1.4 of this guide).
- [financial_infrastructure_guide.md](financial_infrastructure_guide.md) — Singapore FMI context (MEPS+, FAST/PayNow, MAS oversight).
- [payments_hub_guide.md](payments_hub_guide.md) — payments processing and trade-finance context.

---

> **Honesty footer**: This guide was produced with verification-first discipline. Claims are graded in the Section 5.1 claims-status table (verified / reported / lower confidence / unverified / contested). Primary sources (MAS press release, GLDB corporate pages, vendor case studies, Trust Bank governance report) were retrieved via web search in August 2026; where a source was only reachable through an aggregator republish (e.g., the MAS release via The Asian Banker), that is stated. Financial figures and licence facts are as of August 2026 and evolve — re-verify before use.
