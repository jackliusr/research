# ExodusPoint Capital Management: A Comprehensive Guide

**The Entities, Founding, Launch, Strategy Mix, Capital, Funds, Performance Record, Technology, Asia Footprint and Bank Interface of Michael Gelband's Multi-Manager Hedge-Fund Platform — with the Identity Gate on the ExodusPoint Name Cluster, the Six Private Funds Disclosed in Its Form ADV, the Point-in-Time Capital Record, the Performance Question Answered Honestly, the Singapore CMS-Licensed Entity, and a Cymbal Bank Worked Example**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Banking Domain / Institutional Investment & Capital Markets — the multi-manager ("multi-strat") hedge-fund platform as a distinct firm type, the founding-versus-launch distinction, regulatory-register evidence (SEC/IAPD Form ADV, MAS Financial Institutions Directory, ACRA, UK Companies House), the capital-reporting problem for a private fund manager, the bank-interface question for a fund that publicly names almost no counterparty, the Singapore/Asia angle, and the Cymbal Bank institutional lens
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026
> **Companion guides (sibling, same folder — the hedge-fund cluster):** [Citadel LLC](citadel_llc_guide.md) (the archetype firm guide and the structural template this guide follows — its §3 and §7 own the multi-manager platform and the trading-technology stack, cross-referenced here, not re-derived) · [Hedge Fund Guide](hedge_fund_guide.md) (its §3.9 owns the general multi-strategy/multi-manager pod architecture and the platform model; its §4 owns fund economics; its §7 owns the banking relationships — all cross-referenced, not repeated) · [Hedge Funds Singapore](hedge_funds_singapore_guide.md) (the Singapore manager landscape and the RFMC/LFMC/CMS regime — §10 of this guide settles the outstanding ExodusPoint claim at its line 360) · [Hudson River Trading](hudson_river_trading_guide.md) (the prop-trading firm type, which this firm is not) · [Crane Capital](crane_capital_guide.md) (the model for identity discipline and for handling capital claims honestly — §2 and §6 of that guide are the pattern for §2 and §6 here) · [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) (the Singapore regulatory overlay and the Cymbal Bank persona conventions) · [Cayman BVI Master-Feeder](cayman_bvi_master_feeder_guide.md) (the onshore/offshore master-feeder structure this firm's funds actually use — cross-ref §5) · [Asset Management & Alternatives](asset_management_alternatives_guide.md) and [Private Equity](private_equity_guide.md) (the alternatives context)
> **Companion guides (technology/, prefix `../technology/`):** [Quantitative Developer Skillset](../technology/quantitative_developer_skillset_guide.md) (the quant-research skill stack behind a platform of this type — cross-ref §9) · [Low-Latency C++ Development](../technology/low_latency_cpp_development_guide.md) (the latency-engineering discipline — cross-ref §9)

---

**How to use this guide:** Section 1 is the overview — the short answer, the key-facts table, why a bank should care, and the evidence base. Section 2 is the identity gate — "which ExodusPoint?", the name cluster, the verified legal-entity chain across four registries, the dated registry walk-through, the founding-versus-launch distinction (2017 versus 2018), the founders' documented prior affiliations, and what the gate does **not** establish. Section 3 is the history — the 2017 departure from Millennium, the record 2018 launch, the build-out and the poaching litigation, the 2022 stumble, and the 2023–2025 rewrite. Section 4 is the platform and the strategy mix — the multi-manager model cross-referenced to the [Hedge Fund Guide](hedge_fund_guide.md) §3.9 rather than re-derived, the fixed-income lean, risk management, and the fee-and-hurdle structure. Section 5 is the funds — the six private funds disclosed in the firm's Form ADV, their jurisdictions, the master-feeder structure, the internal investor vehicles, and the investor base as disclosed. Section 6 is the capital — the point-in-time AUM record, the $159.4 billion regulatory figure explained, the raise history, and headcount as a capital proxy. Section 7 is the performance record — the firm discloses nothing, and this section reports exactly what the press has reported, dated, with the caveats attached. Section 8 is the people and the culture. Section 9 is the technology question, cross-referenced to [Citadel LLC](citadel_llc_guide.md) §7 with the firm's own public traces kept separate from inference. Section 10 is the Singapore and Asia angle — the MAS-licensed Singapore entity, the two Singapore addresses, and the repository claim this guide settles. Section 11 is the peer positioning, deliberately dated and attributed. Section 12 is the bank interface — what the firm's own filings disclose and what nothing discloses. Section 13 is the Cymbal Bank worked example — a multi-manager fund group as a banking client. Section 14 is the claims audit (✅/⚠/❌), with §14.4 "What Could Not Be Verified". Section 15 is the glossary. Section 16 is cross-references, primary sources and the closing summary. **Integrity convention:** ✅ = verified this pass against a primary or named source (the source and its date are named in §14); ⚠ = flagged/unverified — press estimate, contested, secondary-sourced, or not re-verified live; ❌ = refuted or rejected. Nothing in this guide was invented: where a private firm discloses nothing, this guide says so, and every figure that could not be re-verified at a primary source is marked ⚠ and listed in §14.4.

---

## Table of Contents

1. [The Overview](#1-the-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Key-Facts Table](#12-the-key-facts-table)
   - 1.3 [Why This Matters to a Bank](#13-why-this-matters-to-a-bank)
   - 1.4 [The Evidence Base at a Glance](#14-the-evidence-base-at-a-glance)
2. [The Identity — Which ExodusPoint?](#2-the-identity--which-exoduspoint)
   - 2.1 [The Name Cluster — Many Firms, One Subject](#21-the-name-cluster--many-firms-one-subject)
   - 2.2 [The Verified Legal-Entity Chain](#22-the-verified-legal-entity-chain)
   - 2.3 [The Registry Evidence Walk-Through](#23-the-registry-evidence-walk-through)
   - 2.4 [Founding Versus Launch — the 2017/2018 Distinction](#24-founding-versus-launch--the-20172018-distinction)
   - 2.5 [The Founders and Their Documented Prior Affiliations](#25-the-founders-and-their-documented-prior-affiliations)
   - 2.6 [What the Identity Gate Does Not Establish](#26-what-the-identity-gate-does-not-establish)
3. [The History — 2017 to 2026](#3-the-history--2017-to-2026)
   - 3.1 [The Departure From Millennium (2017)](#31-the-departure-from-millennium-2017)
   - 3.2 [The Launch (2018)](#32-the-launch-2018)
   - 3.3 [The Build-Out and the Poaching Suits (2018–2021)](#33-the-build-out-and-the-poaching-suits-20182021)
   - 3.4 [The 2022 Stumble and the Reset](#34-the-2022-stumble-and-the-reset)
   - 3.5 [2023–2025: the Rewrite](#35-20232025-the-rewrite)
   - 3.6 [The History Table](#36-the-history-table)
4. [The Platform and the Strategy Mix](#4-the-platform-and-the-strategy-mix)
   - 4.1 [The Multi-Manager Model (Cross-Referenced)](#41-the-multi-manager-model-cross-referenced)
   - 4.2 [What the Firm Says About Its Own Approach](#42-what-the-firm-says-about-its-own-approach)
   - 4.3 [The Fixed-Income Lean](#43-the-fixed-income-lean)
   - 4.4 [The Pod Question — What This Guide Does Not Re-Derive](#44-the-pod-question--what-this-guide-does-not-re-derive)
   - 4.5 [Risk Management](#45-risk-management)
   - 4.6 [Fees, Hurdles and the Pass-Through](#46-fees-hurdles-and-the-pass-through)
5. [The Funds — Six Vehicles and the Master-Feeder Structure](#5-the-funds--six-vehicles-and-the-master-feeder-structure)
   - 5.1 [The Fund Table](#51-the-fund-table)
   - 5.2 [The Master-Feeder Structure](#52-the-master-feeder-structure)
   - 5.3 [The Internal Investor Vehicles](#53-the-internal-investor-vehicles)
   - 5.4 [The Jurisdictions and the 3(c) Elections](#54-the-jurisdictions-and-the-3c-elections)
   - 5.5 [The Investor Base as Disclosed](#55-the-investor-base-as-disclosed)
6. [The Capital — What Is Publicly Known](#6-the-capital--what-is-publicly-known)
   - 6.1 [The AUM Record — Point-in-Time Claims](#61-the-aum-record--point-in-time-claims)
   - 6.2 [The $159.4 Billion Regulatory Figure, Explained](#62-the-1594-billion-regulatory-figure-explained)
   - 6.3 [The Raise History](#63-the-raise-history)
   - 6.4 [Headcount as a Capital Proxy](#64-headcount-as-a-capital-proxy)
   - 6.5 [What the Firm Itself Says About Its Size](#65-what-the-firm-itself-says-about-its-size)
7. [The Performance Record — What Is Reported and What Is Not](#7-the-performance-record--what-is-reported-and-what-is-not)
   - 7.1 [The Firm Discloses No Returns](#71-the-firm-discloses-no-returns)
   - 7.2 [The Press-Reported Return Record](#72-the-press-reported-return-record)
   - 7.3 [The One Investor Datapoint With a Named Source](#73-the-one-investor-datapoint-with-a-named-source)
   - 7.4 [The Peer Comparison, Dated](#74-the-peer-comparison-dated)
   - 7.5 [The Honesty Note](#75-the-honesty-note)
8. [The People, the Leadership and the Culture](#8-the-people-the-leadership-and-the-culture)
   - 8.1 [The Principals](#81-the-principals)
   - 8.2 [The Second Tier](#82-the-second-tier)
   - 8.3 [Churn and the Advisory Bench](#83-churn-and-the-advisory-bench)
   - 8.4 [Culture Markers and the Technology Complaints](#84-culture-markers-and-the-technology-complaints)
   - 8.5 [The People Table](#85-the-people-table)
9. [The Technology Question — Cross-Referenced](#9-the-technology-question--cross-referenced)
   - 9.1 [The Class of Stack (Cross-Referenced)](#91-the-class-of-stack-cross-referenced)
   - 9.2 [The Only Public Technology Traces](#92-the-only-public-technology-traces)
   - 9.3 [The 2022 Criticism, Attributed](#93-the-2022-criticism-attributed)
   - 9.4 [What Is Not Public](#94-what-is-not-public)
10. [The Singapore and the Asia Angle](#10-the-singapore-and-the-asia-angle)
    - 10.1 [The Singapore Entity — a Verified Licence, Not an Office Rumour](#101-the-singapore-entity--a-verified-licence-not-an-office-rumour)
    - 10.2 [The Two Singapore Addresses, Explained](#102-the-two-singapore-addresses-explained)
    - 10.3 [The MAS Licensing Context (Cross-Referenced)](#103-the-mas-licensing-context-cross-referenced)
    - 10.4 [The Rest of Asia — Hong Kong, Tokyo, Dubai, Jersey](#104-the-rest-of-asia--hong-kong-tokyo-dubai-jersey)
    - 10.5 [The Repository Claim This Guide Settles](#105-the-repository-claim-this-guide-settles)
11. [The Peer Positioning — One Platform Among Several](#11-the-peer-positioning--one-platform-among-several)
    - 11.1 [The Peer Set](#111-the-peer-set)
    - 11.2 [The 2025 Scorecard, Dated](#112-the-2025-scorecard-dated)
    - 11.3 [The Fee and Hurdle Convergence](#113-the-fee-and-hurdle-convergence)
    - 11.4 [The Comparison Caveat](#114-the-comparison-caveat)
12. [The Bank Interface — What Is Disclosed and What Is Not](#12-the-bank-interface--what-is-disclosed-and-what-is-not)
    - 12.1 [The Covenant This Guide Follows](#121-the-covenant-this-guide-follows)
    - 12.2 [What the Firm's Own Filings Disclose](#122-what-the-firms-own-filings-disclose)
    - 12.3 [The One Provider the Firm Names Itself](#123-the-one-provider-the-firm-names-itself)
    - 12.4 [Why Prime Brokerage Is Confidential by Default](#124-why-prime-brokerage-is-confidential-by-default)
    - 12.5 [What a Bank Actually Sees](#125-what-a-bank-actually-sees)
13. [The Cymbal Bank Worked Example — A Multi-Manager Fund as a Banking Client](#13-the-cymbal-bank-worked-example--a-multi-manager-fund-as-a-banking-client)
    - 13.1 [The Scenario](#131-the-scenario)
    - 13.2 [Onboarding a Cayman Master-Feeder Group](#132-onboarding-a-cayman-master-feeder-group)
    - 13.3 [The Financing and Margin Conversation](#133-the-financing-and-margin-conversation)
    - 13.4 [The Reconciliation Question — NAV, RAUM and GAV](#134-the-reconciliation-question--nav-raum-and-gav)
    - 13.5 [The Regulatory Overlay — MAS, CRS/FATCA and the Singapore Entity](#135-the-regulatory-overlay--mas-crsfatca-and-the-singapore-entity)
    - 13.6 [The Monitoring and the Re-Onboarding Triggers](#136-the-monitoring-and-the-re-onboarding-triggers)
    - 13.7 [The Recommendation — and the Condition the Client Resists](#137-the-recommendation--and-the-condition-the-client-resists)
    - 13.8 [The Lessons](#138-the-lessons)
14. [The Claims Audit — Verified, Flagged, Rejected](#14-the-claims-audit--verified-flagged-rejected)
    - 14.1 [The Verified Claims (✅)](#141-the-verified-claims-)
    - 14.2 [The Flagged Claims (⚠)](#142-the-flagged-claims-)
    - 14.3 [The Rejected or Not-Found Claims (❌)](#143-the-rejected-or-not-found-claims-)
    - 14.4 [What Could Not Be Verified](#144-what-could-not-be-verified)
    - 14.5 [The Anti-Patterns — Symptom, Cause, Guardrail](#145-the-anti-patterns--symptom-cause-guardrail)
15. [The Glossary](#15-the-glossary)
16. [Cross-References, Primary Sources and the Closing Summary](#16-cross-references-primary-sources-and-the-closing-summary)
    - 16.1 [Cross-References](#161-cross-references-repository-convention-sibling-banking-guides-by-plain-filename)
    - 16.2 [Primary Sources Used This Pass](#162-primary-sources-used-this-pass)
    - 16.3 [Register-Query Snapshot](#163-register-query-snapshot-as-at-23-september-2026)
    - 16.4 [The Closing Summary](#164-the-closing-summary)

---

## 1. The Overview

### 1.1 The Short Answer

**ExodusPoint Capital Management, LP** is a **New York-headquartered multi-manager hedge-fund platform**, founded in **April 2017** by **Michael Gelband** and **Hyung Lee** — two senior Millennium Management executives — and launched to external capital in **2018** with a reported **$8.5 billion**, a figure the press has repeatedly described as the **largest hedge-fund launch in history**. Its SEC registration is verified at the regulator's own register: Form ADV, **CRD 294156**, SEC file number **801-112901**, CIK **1736225**, legal entity identifier **549300X6BK8XW1RP4G25**, principal office **65 East 55th Street, New York, NY 10022**, telephone (646) 940-9600; the most recent filing in the cached record is an **Other-Than-Annual Amendment dated 19 August 2026**. ✅ (SEC/IAPD Form ADV for CRD 294156, filed 19 August 2026 — read this pass from the cached PDF text extraction; §2.3 walks the evidence.)

The firm's own website (exoduspoint.com, extracted 23 September 2026) describes it in one sentence: *"ExodusPoint was founded in 2017 and began managing investor capital in 2018. The Firm is led by Michael Gelband and employs a global multi-strategy investment approach, seeking to deliver compelling asymmetric returns by combining complementary liquid strategies managed by experienced portfolio managers within a robust risk framework."* ✅ (firm's own site, 23 Sep 2026.) That sentence contains the **founding-versus-launch distinction** this guide keeps throughout: **2017 is the founding**, **2018 is the launch** — the year external capital arrived. The distinction is not pedantry: the Delaware limited partnership behind the adviser is reported as formed in 2017 (⚠ secondary source), the UK limited-liability partnership at 20 St James's Street was incorporated on **22 February 2018** ✅ (UK Companies House), the Singapore company was incorporated on **19 July 2018** ✅ (ACRA-derived record), and the funds did not begin managing outside money until 2018 ✅ (the firm's own words).

In one line each, the answers a banker most often wants: **What is it?** A multi-manager platform running a fixed-income-led, market-neutral book through six disclosed private funds. **How big is it?** The firm itself publishes no AUM. Press-reported capital has moved from a reported **$13.9 billion peak at mid-2022** and **$13.1 billion at December 2022**, down to **$12.02 billion at end-2023** and **$11.04 billion at 30 June 2024** (Hedgeweek, 19 August 2024, citing a Reuters report and the firm's 13F filing), with Wikipedia's infobox reporting **US$12 billion and 646 employees as of 1 January 2024** (⚠ secondary) and a December 2025 figure of approximately **$12 billion** (⚠). **Is it regulated?** Yes, in three places that this guide verified: an SEC-registered investment adviser in the United States ✅, a **Singapore-incorporated Capital Markets Services licensee** for fund management and dealing ✅, and three Active UK entities at Companies House ✅. **Who owns it?** Not publicly established — the ownership chain and ultimate beneficial owners of the group are not disclosed in any source examined (see §2.6). **Does it disclose its performance?** No. A private fund manager discloses returns to its investors, not to the public (§7).

### 1.2 The Key-Facts Table

| Aspect | Fact | Status |
| --- | --- | --- |
| Legal name (adviser) | EXODUSPOINT CAPITAL MANAGEMENT, LP | ✅ (Form ADV, 19 Aug 2026) |
| Regulators' identifiers | CRD 294156 · SEC file 801-112901 · CIK 1736225 · LEI 549300X6BK8XW1RP4G25 | ✅ (Form ADV) |
| Founded | April 2017 (firm's own "founded in 2017"; Wikipedia infobox "April 2017") | ✅ firm statement / ⚠ infobox |
| Began managing external capital | 2018 (firm's own words) | ✅ (firm's site) |
| Launch size | $8.5 billion, reported June 2018 | ⚠ press-reported (FT, 21 May 2018; Business Insider, 21 Apr 2022) |
| "Largest launch in history" | The framing used by FT, Reuters/Bloomberg-derived coverage and Business Insider | ⚠ attributed, not asserted |
| Founders | Michael Gelband and Hyung Lee | ✅ (firm's site, ADV officers/fund directors, press) |
| Founders' prior affiliation | Millennium Management — Gelband head of fixed income, Lee head of equities | ⚠ press/secondary (Bloomberg 21 May 2018; Forbes 25 Oct 2017; Business Insider 21 Apr 2022) |
| Headquarters | 65 East 55th Street, New York, NY 10022 | ✅ (Form ADV principal office; firm's site) |
| Other US offices | 15 offices, all US, per Item 1.F(5); Stamford CT carries 12 advisory employees, Austin TX 4, San Francisco CA 3, Bala Cynwyd PA 3, San Jose CA 2, ten others 1 or 0 | ✅ (Form ADV Schedule D §1.F) |
| Non-US locations (firm's own site) | London · Jersey · Dubai (DIFC) · Hong Kong · Singapore · Tokyo | ✅ (firm's site, 23 Sep 2026) |
| UK entities | EXODUSPOINT CAPITAL MANAGEMENT UK, LLP (OC421170); EXODUSPOINT SERVICES UK, LTD (11220419); EXODUSPOINT CAPITAL MANAGEMENT UK TECHNOLOGIES, LTD (12329778) — all Active at 20 St James's Street, London SW1A 1ES | ✅ (UK Companies House, 23 Sep 2026) |
| Singapore entity | EXODUSPOINT CAPITAL MANAGEMENT SINGAPORE, PTE. LTD., UEN 201824688K, incorporated 19 July 2018, Live | ✅ (ACRA-derived record, 23 Sep 2026) |
| Singapore licence | MAS **Capital Markets Services Licensee** — Fund Management **and** Dealing in Capital Markets Products (securities, collective investment schemes, exchange-traded derivatives, OTC derivatives, spot FX for leveraged FX trading); CEO **XINYU LIU** | ✅ (MAS Financial Institutions Directory, 23 Sep 2026) |
| Private funds disclosed | Six: EXODUSPOINT PARTNERS MASTER FUND, LP (Cayman) · EXODUSPOINT PARTNERS INTERNATIONAL FUND, LTD (Cayman) · EXODUSPOINT PARTNERS INTERMEDIATE FUND, LP (Cayman) · EXODUSPOINT PARTNERS FUND, LP (Delaware) · EXODUSPOINT MANAGEMENT INVESTORS, LLC (Delaware) · EXODUSPOINT MANAGEMENT INVESTORS INTERNATIONAL, LTD. (Cayman) | ✅ (Form ADV Schedule D §7.B) |
| Fund type as filed | "hedge fund" for the funds reported | ✅ (Form ADV §7.B question 10) |
| Regulatory assets under management (adviser) | **$159,413,741,487** discretionary, across **6 accounts**, as filed 19 Aug 2026; $115,064,838,605 attributable to non-US persons | ✅ (Form ADV Item 5.F) |
| Employees (adviser) | **707** total; **332** performing investment advisory functions | ✅ (Form ADV Item 5.A/5.B) |
| Employees (press/infobox) | 646 as of 1 Jan 2024 (Wikipedia infobox); ~650 in 2025 and 688 in 2022 per Old Well Labs data cited by Business Insider, 28 Jul 2025 | ⚠ secondary |
| Published AUM | **None.** The firm publishes no AUM figure on its website or in its filings' Item 5 narrative | ✅ (verified absence) |
| Published performance | **None.** No return figure is disclosed publicly by the firm | ✅ (verified absence) |
| Disciplinary disclosures | Criminal, regulatory-action and civil-judicial disclosure reporting pages: "No Information Filed" | ✅ (Form ADV, DRP pages) |
| Named service providers | The firm itself names exactly one: its Cayman fund administrator, in its own website privacy notice | ✅ (site privacy notice, effective 3 Apr 2023) |
| Prime brokers | The firm files a multi-dealer panel fund by fund in its Form ADV; this guide does not reproduce provider names and asserts no relationship | see §12 |

### 1.3 Why This Matters to a Bank

For a bank like Cymbal Bank, an ExodusPoint-type client matters on five fronts.

**First, as a counterparty type rather than a story.** A multi-manager platform of this size consumes an enormous amount of bank product — financing and margin, settlement and custody-adjacent services, FX, cash management across a dozen currencies and a dozen offices, listed and OTC derivative clearing, and the operational plumbing that turns thousands of daily trades into a single monthly NAV. The general shape of that relationship — fund economics, prime-brokerage mechanics, the pod architecture behind the risk — is owned by the [Hedge Fund Guide](hedge_fund_guide.md) (§3.9, §4, §7) and by [Citadel LLC](citadel_llc_guide.md) §10, and this guide does not re-derive it. What this guide adds is the **specifics of this firm's disclosed footprint**: which entities exist, in which jurisdictions, under which licences, with which funds and which offices — the facts a bank's onboarding and periodic-review files actually have to cite.

**Second, as an identity problem.** "ExodusPoint" sounds singular but resolves to a group of entities across at least four registries — a Delaware limited partnership, three English entities, a Singapore private company, a Cayman master fund and its feeders — and the adviser's own Form ADV files a second, entirely different business name: **ATLAS CAPITAL** ✅ (Form ADV Schedule D §1.B). A name-screen that returns "ExodusPoint" and stops is not a completed screen; §2 and §13.2 show what completing it looks like.

**Third, as a capital-reporting problem.** This is a firm whose capital is reported *only* by third parties — a $159 billion regulatory figure in a public filing, a $12 billion press figure, a $13 billion investor-document figure, a 646-or-650-or-707 headcount — and a bank that treats any one of them as "the AUM" without its date and its basis has already made an error. §6 is written to make that error hard to make.

**Fourth, as a performance-reporting problem.** The firm publishes no returns. Every return number in circulation comes from investor documents seen by journalists, from people described as close to the firm, or from performance databases. §7 sets out exactly what is reported, by whom, on what date and on what basis — and states plainly what cannot be known.

**Fifth, as a Singapore client.** ExodusPoint is not a foreign fund manager with a marketing office in Singapore; it is a **Singapore-incorporated, MAS-licensed Capital Markets Services licensee** ✅ (§10). That changes the supervisory questions a Singapore bank asks, and it settles a claim that has been sitting open in this repository's [Hedge Funds Singapore](hedge_funds_singapore_guide.md) guide (§10.5).

### 1.4 The Evidence Base at a Glance

Every factual claim in this guide traces to one of six evidence classes. The claims audit (§14) records which class supports each claim.

| Evidence class | What it is used for here | How it is treated |
| --- | --- | --- |
| Regulatory filings (Form ADV, CRD 294156, signed 19 Aug 2026) | The adviser's identity, identifiers, offices, employee counts, regulatory AUM, the six private funds, their jurisdictions and structure, audit/custody/prime-broker records, disciplinary negatives | ✅ for what the filing states; quoted as filed, with the filer named |
| Regulators' registers (MAS Financial Institutions Directory; UK Companies House; ACRA-derived corporate records) | Licences, entity status, incorporation dates, registered offices | ✅ as at the query date, 23 Sep 2026 |
| The firm's own website (exoduspoint.com, extracted 23 Sep 2026) | The founding/launch sentence, strategy language, office list, privacy notice naming its Cayman administrator, investor-portal cookies | ✅ for what the page says; ⚠ where the page would be reporting a metric (it reports none) |
| Named, dated press | Launch size and framing, capital history, performance claims, personnel moves, peer comparisons | ⚠ throughout — every figure carries outlet, date and basis; several rest on anonymous sources |
| Encyclopedic secondary compilation (Wikipedia, retrieved this pass) | The infobox AUM and headcount; the founding narrative with its own citations; return figures cited to Bloomberg and Business Insider | ⚠ — treated as a pointer to the underlying article, never as the article |
| Secondary aggregator (radientanalytics.com, read this pass) | The 2017 Delaware formation date, the GP and its owners, a Dec-2025 AUM figure | ⚠ — one of these figures was independently confirmed in the primary filing (§6.2) and is marked ✅ there |

Two conventions apply throughout. **Dates are part of the fact**: no capital or performance figure appears in this guide without the outlet and the date that produced it, and no such figure is presented as current. **Absence is a finding**: where the firm discloses nothing, this guide says "not publicly disclosed" rather than inferring, and §14.4 collects those absences in one place.

---

## 2. The Identity — Which ExodusPoint?

*This section follows the identity-gate pattern of [Crane Capital](crane_capital_guide.md) §2 and its §2.3 registry walk-through: establish the legal entities at the regulators' own registers, dated, before any other claim.*

### 2.1 The Name Cluster — Many Firms, One Subject

"ExodusPoint" is not a name cluster of the Crane Capital kind — there is no family of unrelated US advisers and property firms trading under it — but it is a **multi-entity group**, and writing about it accurately requires holding six kinds of entity apart:

- **The adviser** — EXODUSPOINT CAPITAL MANAGEMENT, LP, a US (Delaware) limited partnership registered with the SEC as an investment adviser ✅.
- **The general partner of the funds** — EXODUSPOINT PARTNERS GP, LLC, listed in the adviser's Schedule D as the general partner of the private funds ✅.
- **The funds themselves** — six private funds, four of them Cayman companies or partnerships and two of them Delaware vehicles ✅ (§5.1).
- **The UK group** — three Active English entities at 20 St James's Street ✅ (§2.2).
- **The Singapore licensed entity** — EXODUSPOINT CAPITAL MANAGEMENT SINGAPORE, PTE. LTD. ✅ (§10).
- **The second business name** — **ATLAS CAPITAL**, filed by the adviser as an *"other business name"* in its own Form ADV, together with a website address, http://www.atlas-capital.com, listed in the same filing's website section ✅ (Form ADV Schedule D §1.B and §1.I). This is a genuine identity artefact and it is explained here rather than left dangling: "Atlas Capital" is not a separate firm the guide has confused with its subject — it is a name the subject itself files. What sits behind it (a brand, a legacy vehicle, a marketing name) is **not established** by the filing and could not be established this pass: the domain did not resolve for extraction ⚠.

### 2.2 The Verified Legal-Entity Chain

| # | Entity | Registry | Identifier | Status / date | Source (all read 23 Sep 2026 unless dated) |
| --- | --- | --- | --- | --- | --- |
| 1 | EXODUSPOINT CAPITAL MANAGEMENT, LP | SEC / IAPD (US) | CRD 294156 · SEC 801-112901 · CIK 1736225 · LEI 549300X6BK8XW1RP4G25 | Registered investment adviser; current amendment dated 19 Aug 2026 | ✅ Form ADV |
| 2 | EXODUSPOINT PARTNERS GP, LLC | Listed in the adviser's Schedule D | (no separate registration shown) | General partner of the private funds | ✅ Form ADV Schedule D |
| 3 | EXODUSPOINT PARTNERS MASTER FUND, LP | Cayman Islands; 3(c)(7)/3(c)(1); CIMA-registered | Private fund ID 805-1823187184 | Hedge fund; master fund in a master-feeder arrangement | ✅ Form ADV §7.B |
| 4 | EXODUSPOINT PARTNERS INTERNATIONAL FUND, LTD | Cayman Islands; CIMA-registered | ADV §7.B record | Hedge fund; feeder into #3 | ✅ Form ADV §7.B |
| 5 | EXODUSPOINT PARTNERS INTERMEDIATE FUND, LP | Cayman Islands; CIMA-registered | ADV §7.B record | Hedge fund; feeder into #3 | ✅ Form ADV §7.B |
| 6 | EXODUSPOINT PARTNERS FUND, LP | Delaware (US) | ADV §7.B record | Hedge fund; feeder into #3 | ✅ Form ADV §7.B |
| 7 | EXODUSPOINT MANAGEMENT INVESTORS, LLC | Delaware (US) | ADV §7.B record | Hedge fund; internal investor vehicle; $100,000 minimum | ✅ Form ADV §7.B |
| 8 | EXODUSPOINT MANAGEMENT INVESTORS INTERNATIONAL, LTD. | Cayman Islands; CIMA-registered | ADV §7.B record | Hedge fund; internal investor vehicle; $100,000 minimum | ✅ Form ADV §7.B |
| 9 | EXODUSPOINT CAPITAL MANAGEMENT SINGAPORE, PTE. LTD. | ACRA (Singapore) + MAS | UEN 201824688K | Incorporated 19 July 2018; Live Company; MAS CMS licensee | ✅ ACRA-derived record; ✅ MAS FID |
| 10 | EXODUSPOINT CAPITAL MANAGEMENT UK, LLP | UK Companies House | OC421170 | Incorporated 22 Feb 2018; Active; limited liability partnership | ✅ Companies House |
| 11 | EXODUSPOINT SERVICES UK, LTD | UK Companies House | 11220419 | Incorporated 22 Feb 2018; Active; SIC 64205 (financial services holding companies) | ✅ Companies House |
| 12 | EXODUSPOINT CAPITAL MANAGEMENT UK TECHNOLOGIES, LTD | UK Companies House | 12329778 | Incorporated 22 Nov 2019; Active; SIC 62090 (other IT service activities) | ✅ Companies House |

Two structural observations follow directly from the table. The **adviser is a US limited partnership**, not a Cayman or Channel-Islands holding structure — the offshore layer is at the *fund* level (Cayman master and two Cayman feeders) plus the internal investor vehicles, which is the conventional arrangement for a US multi-manager platform with a predominantly non-US client base. And the **UK group is tripartite by function**: an LLP (the operating/advisory vehicle), a services company classified as a financial-services holding company, and a technology company incorporated in late 2019 — a shape that says the London operation was planned from February 2018 onwards and was given its own technology entity within two years ✅ (Companies House dates).

### 2.3 The Registry Evidence Walk-Through

**SEC / IAPD (the primary register for the adviser).** The firm's Form ADV for CRD 294156 is the single richest public document about it, and this pass read it in full from the cached PDF text extraction (the live IAPD firm-summary page is JavaScript-driven and was not usable; the PDF at reports.adviserinfo.sec.gov/reports/ADV/294156/PDF/294156.pdf is the artefact). It establishes, dated **19 August 2026**: the legal name and identifiers (§1.2); the principal office and business hours; **15** other US offices with per-office advisory employee counts; **707** employees of which **332** perform investment advisory functions; **67%** of clients are non-US persons; **6** pooled investment vehicles with **$159,413,741,487** of discretionary regulatory assets under management and none non-discretionary; compensation by a percentage of assets, performance-based fees and — as the filing's own "other" entry — **"PASS THROUGH REIMBURSEMENT"**; the six private funds with jurisdictions, minimum investments, beneficial-owner counts, auditors, custodians and prime brokers recorded per fund; five external locations at which books and records are kept; the websites **exoduspoint.com**, a LinkedIn company page, a Glassdoor page and **http://www.atlas-capital.com**; the other business name **ATLAS CAPITAL**; and the three disciplinary disclosure reporting pages, all of which read **"No Information Filed"**. One caution about the filing as a source for *people*: **its text layer carries no personal names at all** — the Chief Compliance Officer field of Item 1.J extracts blank, and a name-by-name test of this guide's personnel record returned zero hits in the extracted text — so **no person is asserted in this guide on the strength of the filing**, and the leadership names used in §8 come from the firm's own website instead (§8.2) ⚠.

**A note on the cached extraction, because it matters for accuracy.** The text extraction of the filing is highly repetitive — page headers recur on every page and Schedule D records interleave — so a reader can easily mis-attribute a value to the wrong fund or to the wrong address. Two concrete examples, both of which this guide avoided: the address *1 Microsoft Way, Redmond, Washington* appears in the extraction many times but belongs to **Schedule D §1.L (location of books and records)** — a cloud provider's address — not to Schedule D §1.F (offices); and the *"Dallas, TX 33401"* entry carries a ZIP code (33401) that is a West Palm Beach ZIP, an artefact of the filing or of the extraction that is recorded here as an anomaly rather than reconciled ⚠. The rule applied throughout: an address claimed as an office was accepted only from a §1.F block, and a value claimed as a fund's was accepted only where the record's own field sequence supported it.

**MAS Financial Institutions Directory (the Singapore register).** Queried 23 September 2026: **EXODUSPOINT CAPITAL MANAGEMENT SINGAPORE, PTE. LTD.**, shown as **"Incorporated in Singapore"**, licence type **Capital Markets Services Licensee**, activities **Fund Management** and **Dealing In Capital Markets Products** (securities; collective investment schemes; exchange-traded derivatives contracts; over-the-counter derivatives contracts; spot foreign exchange contracts for the purposes of leveraged foreign exchange trading), key personnel **CEO XINYU LIU**, website exoduspoint.com, telephone +65 65766110, address Ocean Financial Centre, 10 Collyer Quay #19-06/08, Singapore 049315 ✅. This is a **positive licence**, not an office rumour, and it is one of the two or three most important facts in this guide (§10).

**ACRA-derived corporate records (the Singapore company register).** UEN **201824688K**: entity type *Local Company*, company type *Private Company Limited by Shares*, **incorporation 19 July 2018**, status **Live Company**, UEN issuing agency ACRA, SSIC primary code **66309**, registered office **77 Robinson Road, #13-00, Robinson 77, Singapore 068896**, **15 officers**, annual return dated 16 June 2026, accounts due 31 July 2027 ✅ (opengovsg.com corporate record for UEN 201824688K, read 23 Sep 2026).

**UK Companies House (the UK register).** A name search for "ExodusPoint" on 23 September 2026 returned four entries, three of them distinct companies ✅: **EXODUSPOINT CAPITAL MANAGEMENT UK, LLP (OC421170)**, incorporated 22 February 2018, Active, limited liability partnership, registered office 20 St James's Street, London SW1A 1ES, last accounts to 31 December 2024 with the next due 30 September 2026; **EXODUSPOINT SERVICES UK, LTD (11220419)**, incorporated 22 February 2018, Active, private limited company, SIC 64205; **EXODUSPOINT CAPITAL MANAGEMENT UK TECHNOLOGIES, LTD (12329778)**, incorporated 22 November 2019, Active, private limited company, SIC 62090. The **FCA Financial Services Register** could **not** be queried successfully this pass — the register's search interface returned a malformed result set to automated extraction — so **no claim is made here about the UK entities' FCA authorisation status in either direction** ⚠ (see §14.4). That is a real gap, and it is stated rather than papered over: the fact that a UK LLP is *incorporated* says nothing about whether any UK entity is *authorised*.

### 2.4 Founding Versus Launch — the 2017/2018 Distinction

The single most confusing thing about this firm's dates is that they are all defensible, because they refer to different events. Laid out in one place:

| Date | Event | Source |
| --- | --- | --- |
| April 2017 | Founding — the firm's own site says "founded in 2017"; Wikipedia's infobox says "April 2017" | ✅ firm statement / ⚠ infobox |
| 2017 | Delaware limited partnership formation of the adviser, reported by a secondary aggregator; GP reported as ExodusPoint Capital Partners GP, LLC, owned by Gelband and Lee | ⚠ secondary |
| 25 October 2017 | Forbes reports the Englander–Gelband dispute (the departure narrative's dated press anchor) | ⚠ press, via Wikipedia's citation |
| 4 January 2018 | Dealbreaker reports it "looks like" Gelband won the arbitration against Millennium | ⚠ press, via Wikipedia's citation |
| 22 February 2018 | The two original UK entities incorporated (LLP OC421170; Services Ltd 11220419) | ✅ Companies House |
| 21 May 2018 | FT and Bloomberg report the launch and its size | ⚠ press |
| June 2018 | The firm "launched with $8.5 billion in capital" / "started trading with a record $8.5 billion" | ⚠ press (FT 21 May 2018; Business Insider 21 Apr 2022 and 28 Jul 2025; Wikipedia) |
| 19 July 2018 | The Singapore company incorporated (UEN 201824688K) | ✅ ACRA-derived |
| 2018 | "Began managing investor capital" | ✅ firm's own words |

The guide's rule, applied everywhere below: **"founded" means 2017; "launched" means 2018**; and no sentence in this guide uses "founded in 2018" or "founded in 2017 and launched in 2017". Where a source is inconsistent, the source is characterised rather than smoothed.

### 2.5 The Founders and Their Documented Prior Affiliations

**Michael Gelband** (founder; the firm's site says "The Firm is led by Michael Gelband"; Business Insider, 28 July 2025, describes him as **sole chief investment officer** after Hyung Lee stepped down in 2024) and **Hyung Lee** (co-founder; the ADV lists him nowhere by name, but the press consistently names both as co-founders) ✅/⚠.

Their prior affiliation is reported the same way by every dated source examined: both came from **Millennium Management**, where **Gelband ran the fixed-income business** and **Lee ran equities**, and where Gelband was seen by some as heir to **Israel Englander** until he was declined an ownership stake and left ⚠ (Business Insider, 21 April 2022, which says Gelband "spent close to a decade running the fixed-income side of the business" and had been a Lehman Brothers executive; Forbes, 25 October 2017; Bloomberg, 21 May 2018; Wikipedia, retrieved this pass, which cites those articles). Business Insider (28 July 2025) adds the figure that is most often repeated about him — that he **told former colleagues his Millennium unit generated $7 billion in trading revenue over his eight years there** ⚠ (a self-reported figure repeated by a journalist, not a filing). The same article notes that Gelband **does not manage money himself** at ExodusPoint ⚠.

Three further details are dated and attributed, and none is asserted as adjudicated fact: **Millennium filed an arbitration case against Gelband** over staff poaching shortly after his departure, with the outcome not made public although the press believed Gelband won ⚠ (Forbes, 25 Oct 2017; Dealbreaker, 4 Jan 2018; Wikipedia); **Schonfeld Strategic Advisors sued ExodusPoint in October 2018** to stop it poaching employees, and ExodusPoint ultimately won ⚠ (Bloomberg Law, 2 Oct 2018 and 9 Apr 2019 reports, via Wikipedia's citations); and several Millennium staff followed the founders to the new firm ⚠ (same sources). This guide does not assert an ownership structure, a control relationship or the founders' former employer as fact beyond what these named sources say — and states explicitly at §2.6 what remains unestablished.

### 2.6 What the Identity Gate Does Not Establish

- **The ownership chain.** Who owns the adviser, the GP and the non-US entities; the percentage interests of Gelband and Lee; whether any outside shareholder exists — none of this is established by any source read this pass. A secondary aggregator reports the GP as "ExodusPoint Capital Partners GP, LLC, owned by Michael Gelband and Hyung Soon Lee" ⚠ and the primary filing's Schedule A/B ownership table lies beyond the cached portion of the extraction, so it could not be confirmed there ⚠.
- **The group structure between entities.** The ADV records *that* the funds and the GP exist; the UK, Jersey, Dubai, Hong Kong, Tokyo and Singapore operations are established by the website and, for Singapore and the UK, by registers. **How the group is wired** — which entity employs whom, which entity holds which licence, where profits sit — is not public in any single document examined.
- **FCA status.** Unresolved (§2.3, §14.4): the FCA register could not be searched to a valid result this pass, so no claim is made.
- **The "Atlas Capital" business name.** Filed by the firm itself ✅, but its purpose, history and any vehicle behind it are not established ⚠.
- **Jersey, Dubai, Hong Kong and Tokyo entities.** The *offices* are on the firm's own site ✅; the *entities* behind them were not researched to register level here (out of scope for this pass), and no claim is made about their licences.

---

## 3. The History — 2017 to 2026

### 3.1 The Departure From Millennium (2017)

The founding story is a well-reported one and it is told here with its sources attached, because it is the kind of story that hardens into folklore. In 2017 a senior Millennium executive — **Michael Gelband**, who had run the firm's fixed-income business after arriving from Lehman Brothers and who was seen by some as the eventual successor to founder **Israel Englander** — left after being declined an ownership stake, and set out to build a competing multi-manager platform with Millennium's former head of equities, **Hyung Lee** ⚠ (Business Insider 21 Apr 2022; Forbes 25 Oct 2017; Bloomberg 21 May 2018, the latter two via Wikipedia's citations). Several Millennium employees followed, and Millennium **filed an arbitration case** to stop the recruiting; the outcome was not made public, though the press concluded Gelband had won ⚠. What is not in doubt, because the firm says it itself, is the founding year and the model: *"founded in 2017"*, a *"global multi-strategy investment approach"*, *"complementary liquid strategies managed by experienced portfolio managers within a robust risk framework"* ✅ (exoduspoint.com, 23 Sep 2026).

### 3.2 The Launch (2018)

The launch is the firm's defining public fact. Two dated press accounts frame it identically: the **Financial Times** on **21 May 2018** published "Gelband finalises largest hedge fund launch ever", and **Bloomberg** the same day published "Michael Gelband to Launch Biggest Hedge Fund Startup Ever" ⚠ (both cited by Wikipedia, retrieved this pass; the FT page itself returned an error to automated extraction this pass ⚠, so the FT headline and date are reported as cited rather than re-read). The size is reported as **$8.5 billion**, with the previous record launch described as **Convexity Capital's $6.3 billion in 2006** ⚠ (FT, 21 May 2018, as reported in the research brief underpinning this guide and repeated in subsequent coverage).

Two discipline points. **First**, the phrase *"largest launch in history"* is a **framing used by the press**, not a statistical fact this guide asserts; it is attributed in every use. **Second**, the $8.5 billion is **launch capital as reported**, not a figure the firm has ever published: the firm publishes no AUM at all (§6.5). Business Insider has repeated the $8.5 billion figure in three separate dated pieces (21 April 2022, 2 February 2023, 28 July 2025) ⚠, which makes it the most consistently reported number in this firm's public record — and still a press figure.

A fee detail belongs to the launch story because it is unusual and because the primary filing corroborates its direction. Wikipedia, citing the 2022 Business Insider reporting, says that rather than charging the standard management fee the firm **passes on unlimited costs to investors** and that these are expected to be "substantial" over time, making it more expensive than peers ⚠. The primary filing corroborates that **pass-through reimbursement is a compensation arrangement of the adviser**: Item 5.E records compensation by a percentage of assets, performance-based fees and *"Other (specify): PASS THROUGH REIMBURSEMENT"* ✅ (Form ADV Item 5.E, 19 Aug 2026). The filing does not quantify it; the "unlimited costs" characterisation is press-attributed ⚠. See §4.6.

### 3.3 The Build-Out and the Poaching Suits (2018–2021)

The four years after launch were a build-out of the kind the [Hedge Fund Guide](hedge_fund_guide.md) §3.9 describes generically for platform firms — capital, pods, infrastructure — executed at unusual speed. Dated markers:

- **October 2018** — Schonfeld Strategic Advisors sues ExodusPoint to stop it poaching employees; the injunction bid fails in April 2019 and ExodusPoint ultimately wins ⚠ (Bloomberg Law, 2 Oct 2018; 9 Apr 2019).
- **2019** — the fund returns **6.8%**, against a hedge-fund industry average of 9% ⚠ (Bloomberg, 8 Jan 2020, via Wikipedia's citation).
- **October 2019** — a senior executive leaves after less than two years ⚠ (WSJ, 24 Oct 2019, via Wikipedia).
- **Spring 2020** — the firm raises **$3 billion** in what coverage describes as its only fundraising since launch, oversubscribed, with **80% of the capital coming from existing investors** ⚠ (Business Insider, 21 Apr 2022, quoting a source close to the firm; Wikipedia). That same article reports the fund's 2020 return as **13.5%** ⚠ (via Wikipedia citing Bloomberg).
- **Early 2022 data as reported** — the firm was managing **$13.5 billion** with offices "around the world — including London, Paris, Singapore, and Hong Kong" and a headcount "close to 700, including 109 portfolio managers", per an investor update sent at the end of January ⚠ (Business Insider, 21 Apr 2022). The same article reports investor-base composition (pensions, endowments, foundations, sovereign wealth funds) ⚠ and criticism of the firm's **technology build-out** and of management style from more than a dozen people interviewed ⚠ (§9.3).

### 3.4 The 2022 Stumble and the Reset

2022 is the year the firm's growth curve broke, and the reporting is unusually granular because it rests on *investor documents* seen by journalists — which is a better class of evidence than rumour, but still not a disclosure.

Per Business Insider's analysis of **2 February 2023** ⚠: the firm returned **5.5% to 6% in 2022, depending on share class** (investor documents and people familiar), an improvement on **4.9% in 2021** but far behind peers it is benchmarked against — **Citadel 38.1%, Millennium 12.4%, Point72 10.25%, Balyasny 9.7%**. It **started 2022 with $13.5 billion and 674 employees, including 108 portfolio managers**; by June it was at **$13.9 billion, nearly 700 employees and 106 PMs**, with the fund up 3.3%; it **ended the year with $13.1 billion**, headcount down **11%** from June and the **number of portfolio managers down nearly 25% to 81**, after a November shake-up. Assets fell **$800 million between June and December** even as performance was positive — i.e., the decline was redemptions, not losses. The same reporting describes **where the money was made and lost**: rates and macro trading (the fixed-income side Gelband oversaw) produced the bulk of profits, while **equity quant lost 0.77% and event arbitrage 1.82%**, and the firm's exposure shifted from an even fixed-income/equities split at the end of 2021 to a majority in rates and macro by the end of 2022. A former fixed-income employee is quoted saying it was "surprising to me how much equity and capital was being put toward equities when fixed-income was clearly the breadwinner" ⚠.

This is also the point at which the **$1 billion of 2023 withdrawals** later reported by Bloomberg (**27 March 2024**, "ExodusPoint Clients Yanked $1 Billion From Hedge Fund Last Year") and the associated Reddit/Reuters-derived $1 billion 2023 outflow figure in Hedgeweek (**19 August 2024**) begin to make sense as a single narrative: performance that was acceptable but not competitive, a fee base that investors questioned, and a firm closed to new capital and therefore unable to replace redemptions ⚠.

### 3.5 2023–2025: the Rewrite

The last three reported years are the recovery. Dated markers, all ⚠ press unless noted:

- **2023** — the firm adds **$1 billion in new cash** and makes a series of hires, including **Adam Galeon** (long-short equities), **Michael Lapsa** (systematic strategies) and **Peter McConnon** (formerly head of London macro for Balyasny, as a senior managing director of fixed income and macro) (Business Insider, 28 Jul 2025).
- **2024** — the firm **adopts a cash hurdle** for performance fees, collecting performance fees only when it beats a Treasury bill — a change Business Insider describes as lowering the overall fee rate and boosting net returns (Business Insider, 28 Jul 2025; the same outlet reported the industry-wide trend on 20 Sep 2024). Reported **2024 return: 11.3%**, its best since 2020 ⚠ (Bloomberg, via Wikipedia's citation).
- **H1 2024** — AUM falls **$1 billion** to **$11.04 billion at 30 June 2024**, from **$12.02 billion at end-2023**, with the drop attributed by unnamed sources primarily to client withdrawals; the firm had been **closed to new investments since 2023** and was not replacing withdrawn capital; through July 2024 the fund was **up 3.6%** against a **6.3%** multi-strategy average (PivotalPath) and **Citadel's 8.82%** ✅ fetched (Hedgeweek, 19 Aug 2024, citing a Reuters report and the firm's 13F filing).
- **Mid-2025** — the firm is described as **"$11 billion"**, up more than 9% for the year through June and **more than 18% over the twelve months from July 2024**; roughly **75% of the firm's risk is in fixed-income books**; headcount stands at **roughly 650**, down from **688 in 2022**, per regulatory filings compiled by Old Well Labs; the firm is **closed to new capital** and is not expanding into commodities or private credit ⚠ (Business Insider, 28 Jul 2025).
- **2025 full year** — ExodusPoint returns **18%** (including 2.1% in December), described as **its best year on record** by "a person close to the New York-based manager", in a year when Millennium made **10.5%**, Citadel Wellington **10.2%**, Balyasny **16.7%**, Point72 **17.5%** and D.E. Shaw's Composite **18.5%** ✅ fetched (Business Insider, 6 Jan 2026; the story was originally published 2 Jan 2026).
- **8 December 2025** — the firm **hires Russel (of Eisler Capital) to help run stock trading** ⚠ (Bloomberg, 8 Dec 2025, as recorded in the research brief; not re-read this pass).

### 3.6 The History Table

| Year | Event | Evidence class |
| --- | --- | --- |
| 2017 (April) | Founded by Michael Gelband and Hyung Lee after Gelband's departure from Millennium | ✅ firm statement / ⚠ press |
| 2017 (25 Oct) | Forbes reports the Englander–Gelband dispute | ⚠ press |
| 2018 (Jan) | Dealbreaker reports the arbitration appears to have gone Gelband's way | ⚠ press |
| 2018 (22 Feb) | EXODUSPOINT CAPITAL MANAGEMENT UK, LLP and EXODUSPOINT SERVICES UK, LTD incorporated in England | ✅ Companies House |
| 2018 (21 May / June) | Launch to external capital with a reported $8.5 billion — the "largest launch ever" framing | ⚠ press (FT; Bloomberg; Business Insider) |
| 2018 (19 Jul) | Singapore company incorporated (UEN 201824688K) | ✅ ACRA-derived |
| 2018 (Oct) | Schonfeld sues over poaching; injunction denied April 2019 | ⚠ press |
| 2019 | Return of 6.8% vs a 9% industry average | ⚠ press |
| 2019 (22 Nov) | EXODUSPOINT CAPITAL MANAGEMENT UK TECHNOLOGIES, LTD incorporated | ✅ Companies House |
| 2020 (spring) | $3 billion raised — described as the firm's only fundraising since launch; 80% from existing investors | ⚠ press |
| 2021 | Return of 4.9%; fundamental equities unit "WestWind" launched under Robert Bovo | ⚠ press |
| 2022 | Return of 5.5–6%; assets peak mid-year at a reported $13.9bn and end the year at $13.1bn; PM count −25% | ⚠ press (investor documents) |
| 2023 | $1bn of reported net withdrawals; $1bn of new capital raised; senior hires; closed to new investments | ⚠ press |
| 2024 | Cash hurdle adopted; AUM reported at $11.04bn (30 Jun) from $12.02bn (end-2023); return of 11.3% | ⚠ press |
| 2025 | Return of 18% — reported as the firm's best year on record; ~$11bn; headcount ~650 | ⚠ press |
| 2026 (19 Aug) | Form ADV filed: 707 employees, 332 advisory, 6 funds, $159.4bn regulatory AUM, no disciplinary disclosures | ✅ primary |

---

## 4. The Platform and the Strategy Mix

### 4.1 The Multi-Manager Model (Cross-Referenced)

ExodusPoint is a **multi-manager platform** — in market shorthand a "multi-strat" — the firm type whose architecture, economics and risk logic are already set out in this repository: the **pod architecture** and the platform model are owned by [Hedge Fund Guide](hedge_fund_guide.md) **§3.9**, fund economics (management and performance fees, high-water marks, pass-through expense arrangements) by that guide's **§4**, and the banking relationships that flow from the model by its **§7**; the archetype firm treatment is [Citadel LLC](citadel_llc_guide.md) §3. This guide deliberately **does not re-derive** any of it. What follows is only what is specific to *this* firm: what it says about its own approach, where its risk actually sits, and how its fee structure has changed.

### 4.2 What the Firm Says About Its Own Approach

The firm's own website (exoduspoint.com, extracted 23 September 2026) is the only place it describes its method, and the description is worth quoting precisely because so little else is published ✅:

- **Global multi-strategy**, founded 2017, managing capital since 2018, led by Michael Gelband.
- **Capital allocation across managers** described as "equally important", determined by "criteria including investment strategy, expected return profile, capacity and liquidity, and overall portfolio impact" — i.e., a central allocation function, the defining trait of the platform model.
- **Strategy span**: "major asset classes across global markets, predominantly falling within the fixed income and equity asset classes"; the strategies are "generally market neutral" and "typically exhibit low levels of correlation to broad market indices".
- **Risk**: a framework implemented by an experienced risk team "utiliz[ing] both portfolio manager level and fund level elements", with a philosophy centred on "diversification across liquid strategies and capital preservation".
- **People**: "Human capital is an important element of the multi-manager model"; a platform "designed to attract and support experienced investment professionals across all liquid systematic and discretionary equity and fixed income strategies", providing "the capital, infrastructure, and intellectual resources" for a "frictionless environment".

Note what the firm does **not** say: it publishes no AUM, no return, no fee detail, no fund list, no investor names and no counterparties on its website ✅ (verified absence — the extract contains none of these).

### 4.3 The Fixed-Income Lean

The most consistently reported structural fact about the firm's risk is its **fixed-income tilt**, and it comes from two independent places. Business Insider reported on **21 April 2022** that the firm's profits had "largely stemmed from its fixed-income trading, the side of the business overseen by Gelband", and that **fixed income accounted for 75% of the firm's risk** ⚠; the same outlet reported on **28 July 2025** that **roughly 75% of the firm's risk was in fixed-income books** — the specialty of a co-founder who, by the same account, **does not manage money himself** ⚠. The 2022 reporting adds the counterpoint: the **equities** side produced the recurring headaches, with equity quant and event arbitrage posting the worst results of the year ⚠ (Business Insider, 2 Feb 2023). That combination — a fixed-income-led, market-neutral book, with a smaller and later-built equity franchise — is what distinguishes this firm from the equity- and quant-led platforms it is compared against (§11.4).

### 4.4 The Pod Question — What This Guide Does Not Re-Derive

How the pods work — how PM teams are capitalised, how risk limits and drawdown triggers are set per pod, how a PM's P&L is separated and how payouts are computed — is the subject of [Hedge Fund Guide](hedge_fund_guide.md) §3.9 and is **not restated here**. Two firm-specific observations only, both attributed: the firm's **portfolio-manager count is reported to have fallen nearly 25% in 2022** (from 106 at June to **81** at year-end, after a November shake-up) while headcount fell 11% ⚠ (Business Insider, 2 Feb 2023) — a datapoint about pod turnover that is otherwise private; and the firm is reported to have been **strategic about expansion and hiring**, reducing staff slightly since 2022 rather than exploding headcount the way some peers did ⚠ (Business Insider, 28 Jul 2025).

### 4.5 Risk Management

Two things are verifiable here and one is not. Verified: the firm describes a risk framework with **portfolio-manager-level and fund-level elements** implemented by an in-house risk team, oriented to **diversification across liquid strategies and capital preservation** ✅ (exoduspoint.com). Reported: the firm's **Chief Risk Officer is Kunal Kumar**, named January 2025 (previously Co-Head of Global Macro Risk at Balyasny) ✅ (firm's leadership page, 23 Sep 2026), with the founding risk head **Dev Joneja** now titled **Chairman of Risk** ✅ (same source) rather than having left. Not public: any risk number — gross or net exposure, leverage, VaR, drawdown limits, liquidity terms — is **not disclosed anywhere** this pass examined. The only indirect, and highly imperfect, public signal is the adviser's filing, where the funds' combined current gross asset value is on the order of **$180 billion** against a press-reported ~$11–12 billion of net investor capital — which would imply a leveraged book, but the guide flags this as an **arithmetic observation from two different measures, not a leverage figure** (§6.2, §13.4).

### 4.6 Fees, Hurdles and the Pass-Through

Fee detail for this firm is thin, and what exists is worth separating by evidence class.

| Element | What is reported | Status |
| --- | --- | --- |
| Compensation arrangements of the adviser | "A percentage of assets under your management"; "Performance-based fees"; "Other (specify): **PASS THROUGH REIMBURSEMENT**" | ✅ Form ADV Item 5.E, 19 Aug 2026 |
| Expense pass-through characterisation | Rather than a standard management fee, the firm "will pass on unlimited costs to investors", expected to be "substantial" over time | ⚠ press (Business Insider reporting cited by Wikipedia) |
| Cash hurdle on performance fees | Adopted 2024 — performance fees only when the firm beats a Treasury bill; described as lowering the overall fee rate and boosting net returns | ⚠ press (Business Insider, 28 Jul 2025) |
| Management fee rate, performance fee rate, high-water-mark mechanics, liquidity terms | **Not publicly disclosed** | ❌ not found |

The general economics of these arrangements — what a pass-through actually covers, how a cash hurdle interacts with a high-water mark, why platform firms converged on hurdles — belong to [Hedge Fund Guide](hedge_fund_guide.md) §4 and are not restated here. The firm-specific point is narrower and material to a bank: **the adviser's own filing confirms that expense pass-through is one of its compensation arrangements** ✅, which means a bank analysing the client's cost base should expect operating expenses to flow to the funds rather than sit with the manager.

---

## 5. The Funds — Six Vehicles and the Master-Feeder Structure

### 5.1 The Fund Table

The firm's Form ADV Schedule D §7.B discloses six private funds, reported under the adviser as "Total Funds: 6". Everything in this table is from that filing (signed 19 August 2026) ✅, with figure-attribution caveats noted below.

| Fund | Jurisdiction | Type as filed | Structure | Minimum investment (as filed) | Current gross asset value (as filed) | Beneficial owners (as filed) |
| --- | --- | --- | --- | --- | --- | --- |
| **EXODUSPOINT PARTNERS MASTER FUND, LP** | Cayman Islands | hedge fund | Master fund (3(c)(7)/(3(c)(1)); CIMA-registered; private fund ID 805-1823187184) | $5,000,000 | $158,566,481,481 | 103 |
| **EXODUSPOINT PARTNERS INTERNATIONAL FUND, LTD** | Cayman Islands | hedge fund | Feeder into the master fund | $5,000,000 | $9,801,947,780 | 77 |
| **EXODUSPOINT PARTNERS INTERMEDIATE FUND, LP** | Cayman Islands | hedge fund | Feeder into the master fund | $5,000,000 | $9,295,446,241 | 2 |
| **EXODUSPOINT PARTNERS FUND, LP** | Delaware, US | hedge fund | Feeder into the master fund | $5,000,000 | $3,652,520,629 | 26 |
| **EXODUSPOINT MANAGEMENT INVESTORS, LLC** | Delaware, US | hedge fund | Internal investor vehicle | $100,000 | $428,574,642 | 16 |
| **EXODUSPOINT MANAGEMENT INVESTORS INTERNATIONAL, LTD.** | Cayman Islands | hedge fund | Internal investor vehicle | $100,000 | $47,295,936 | 14 |

**Figure-attribution caveat.** The cached extraction of the filing interleaves fund records across page boundaries, so fund-to-figure attribution had to be established from each record's own field sequence rather than by proximity alone. The master fund's **$158,566,481,481** figure is independently corroborated: Item 5.F of the same filing reports total regulatory assets under management of **$159,413,741,487** across six accounts, and the master fund sits at the centre of the structure, so the two are consistent by construction ✅. The remaining five gross-asset-value figures and all six beneficial-owner counts are recorded here as **as-filed but attribution-checked only by field sequence** ⚠ — they should be read as the filing's numbers, not re-derivable from this guide.

### 5.2 The Master-Feeder Structure

The structure is the conventional one, and it is worth naming plainly because it drives the entire banking relationship: **one Cayman master fund (EXODUSPOINT PARTNERS MASTER FUND, LP) into which the other funds invest**, with a **US onshore feeder** (EXODUSPOINT PARTNERS FUND, LP, Delaware), an **offshore feeder** (EXODUSPOINT PARTNERS INTERNATIONAL FUND, LTD, Cayman), a second Cayman vehicle (EXODUSPOINT PARTNERS INTERMEDIATE FUND, LP) and two small internal vehicles (§5.3) ⚠. For the mechanics of this arrangement — why masters sit offshore, how feeders handle US-taxable versus non-US investors, how subscriptions and redemptions flow through, and why the arrangement suits a predominantly non-US investor base — see [Cayman BVI Master-Feeder](cayman_bvi_master_feeder_guide.md); it is cross-referenced, not re-derived.

The filing's own numbers tell the story of *where the money sits*: the master fund's gross asset value is roughly **13 times** the press-reported net investor capital of ~$11–12 billion, and the master fund alone roughly equals the adviser's entire regulatory-AUM figure (§6.2). Both statements are arithmetic on public numbers, not disclosures, and both are central to the reconciliation trap described in §13.4.

### 5.3 The Internal Investor Vehicles

Two of the six funds are different in kind and the filing shows it: **EXODUSPOINT MANAGEMENT INVESTORS, LLC** (Delaware) and **EXODUSPOINT MANAGEMENT INVESTORS INTERNATIONAL, LTD.** (Cayman), each with a **$100,000 minimum investment** against **$5,000,000** for the four main funds ✅. The names and the minimums are the filing's; the inference is reasonable and is flagged as inference — these are **internal vehicles for the firm's own people** (principals and staff investing alongside external capital), the standard "GP/founder vehicle" pattern on a multi-manager platform.

### 5.4 The Jurisdictions and the 3(c) Elections

Every fund is registered as a **hedge fund** in answer to the fund-type question ✅, and the funds record exclusions from the definition of an investment company under **section 3(c)(1) and/or section 3(c)(7)** of the Investment Company Act of 1940 ✅ — the private-fund exemptions that make an unregistered, private placement possible, described in [Hedge Fund Guide](hedge_fund_guide.md) §2 and [Cayman BVI Master-Feeder](cayman_bvi_master_feeder_guide.md). The **Cayman** funds are additionally registered with the **Cayman Islands Monetary Authority** ✅, which is disclosed in the filing. One fund's record carries a **Form D file number (021-313831)** ✅ — the US private-placement notice that accompanies a Regulation D offering. The pattern (Cayman master, US and Cayman feeders, CIMA registration, Reg D where relevant) is the standard offshore spine of a US multi-manager platform and is exactly the structure a bank's onboarding team will see in the client's constitutional documents.

### 5.5 The Investor Base as Disclosed

Four disclosure routes give a partial picture of who the money belongs to, and none of them complete it:

1. **The filing's client statistics** ✅: **six** pooled investment vehicles as clients; **67%** of the adviser's clients are non-US persons; **$115,064,838,605** of the $159.4bn regulatory AUM is attributable to non-US persons (≈72%); **zero** clients for whom the adviser does not have regulatory AUM. This tells a bank that the investor base is **predominantly non-US** without naming a single investor.
2. **The filing's beneficial-owner counts** ✅/⚠: 103 for the master fund, 77 for the offshore feeder, 26 for the onshore feeder, 16 and 14 for the two internal vehicles, and 2 for the intermediate Cayman vehicle — the count of beneficial owners, not of investors' identity or size.
3. **Press-reported investor composition** ⚠: "pensions, endowments, foundations, sovereign wealth funds" (Business Insider, 21 Apr 2022), and one named public pension — the **Teacher Retirement System of Texas**, whose document showed a **$500 million investment in August 2018 had become a little more than $607 million three years later, an average annual return of 7.86%** ⚠ (same article) — the single most concrete investor-level datapoint in this firm's public record (§7.3).
4. **An absence** ❌: no investor list, no subscription amounts, no side-letter terms and no fee schedules are published anywhere examined.

--- 

## 6. The Capital — What Is Publicly Known

*This section follows the capital discipline of [Crane Capital](crane_capital_guide.md) §6: every figure is a point-in-time claim, every claim carries its source and date, and the difference between different kinds of "AUM" is stated rather than smoothed.*

### 6.1 The AUM Record — Point-in-Time Claims

There is no such thing as "ExodusPoint's AUM" in the public record — there are only dated, differently-sourced claims, and they disagree because they measure different things on different dates. Assembled in one place, which is the only honest way to present them:

| Date of the claim | Figure | Basis | Source |
| --- | --- | --- | --- |
| August 2018 | $500 million invested by a single public pension | Investor document | ⚠ Business Insider, 21 Apr 2022 (Teacher Retirement System of Texas) |
| June 2018 | **$8.5 billion** at launch | Press reporting of launch capital | ⚠ FT, 21 May 2018; Bloomberg, 21 May 2018; repeated by Business Insider 2022/2023/2025 |
| Spring 2020 | **$3 billion** raised in a single oversubscribed round | Press, sourced to a person close to the firm | ⚠ Business Insider, 21 Apr 2022 |
| Start of 2022 | **$13.5 billion** | Investor update (end-January) | ⚠ Business Insider, 21 Apr 2022 |
| June 2022 | **$13.9 billion** | Investor documents | ⚠ Business Insider, 2 Feb 2023 |
| December 2022 | **$13.1 billion** — the reported peak | Investor documents | ⚠ Business Insider, 2 Feb 2023; Hedgeweek, 19 Aug 2024 (citing Reuters) |
| 2023 | **$1 billion** of net client withdrawals | Press, and the firm's 13F filing per a Reuters report | ⚠ Bloomberg, 27 Mar 2024; Hedgeweek, 19 Aug 2024 |
| 31 December 2023 | **$12.02 billion** | Reuters report on the firm's 13F filing | ⚠ Hedgeweek, 19 Aug 2024 |
| 1 January 2024 | **US$12 billion** and **646 employees** | Encyclopedic infobox | ⚠ Wikipedia infobox, retrieved this pass |
| 30 June 2024 | **$11.04 billion** | Reuters report on the firm's 13F filing | ✅ as published (Hedgeweek, 19 Aug 2024) / ⚠ for the underlying figure |
| July 2025 | **"$11 billion"** | Press, sourced to a person with direct knowledge | ⚠ Business Insider, 28 Jul 2025 |
| December 2025 | approximately **$12 billion** | Press | ⚠ Wikipedia (citing a Bloomberg article), retrieved this pass |
| 31 December 2025 | approximately **$11.7 billion** of AUM, with approximately **$159.4 billion** of regulatory AUM | Secondary aggregator reading of Form ADV | ⚠ aggregator for the $11.7bn; **✅ for the $159.4bn, confirmed inside the filing itself (§6.2)** |
| 19 August 2026 | **$159,413,741,487** of regulatory assets under management, 6 accounts | The adviser's own filing | ✅ Form ADV Item 5.F |

**How to read this table.** The $11–14 billion band is *net investor capital as reported by third parties* — sometimes from investor documents, sometimes from a 13F-based calculation, sometimes from an unnamed source — and it is the number the industry means when it says the firm is "an $11 billion manager". The $159.4 billion is the *adviser's own regulatory-assets-under-management figure*, which is a different measure entirely (§6.2). Neither is a disclosure of the firm's equity, its leverage or its net asset value per share, and the firm has never published any of the three.

### 6.2 The $159.4 Billion Regulatory Figure, Explained

The research brief for this guide flagged the $159.4 billion regulatory-AUM figure as needing verification inside the filing itself before use. It verified ✅. Form ADV Item 5.F for CRD 294156, filed 19 August 2026, reports **discretionary regulatory assets under management of $159,413,741,487 across 6 accounts**, non-discretionary **$0**, total **$159,413,741,487**, with **$115,064,838,605** attributable to non-US persons; the corresponding Item 5.D table reports the same $159.4 billion against the single client category "pooled investment vehicles (other than investment companies and business development companies)", against **6** clients ✅.

What the figure is and is not:

- **It is a gross, portfolio-level measure.** Regulatory AUM counts the market value of the securities portfolios the adviser manages — including positions financed with borrowed money. On a multi-manager platform, that means the figure scales with the funds' gross exposure, not with investors' equity.
- **It is not net investor capital.** The same filing's Schedule D shows the master fund's own current gross asset value at **$158,566,481,481** ✅ (attribution corroborated by the matching total), while press reporting puts net investor capital in the **$11–12 billion** range ⚠. The ratio implied by those two public numbers — roughly thirteen to one — is an **arithmetic observation about two different measures**, not a disclosed leverage ratio, and this guide does not present it as one. What it does say, plainly, is that **any bank or reader who treats the $159.4 billion as "the fund's size" has misread a regulatory definition**.
- **It is not a market-share statement.** Nothing in it is comparable to the AUM figures of Millennium, Citadel or Point72 as those firms report them.

This is also the single best illustration in this guide of the capital-reporting discipline the repository asks for: the same firm carries a "$12 billion" number, an "$11.04 billion" number and a "$159.4 billion" number, all defensible, all measuring different things on different dates, and all useless without their label.

### 6.3 The Raise History

Reported fundraising events, in order: the **2018 launch** ($8.5 billion as reported) ⚠; a single subsequent round in **spring 2020** of **$3 billion**, described as oversubscribed and as the firm's only fundraising since launch, with **80% of the capital from existing investors** ⚠ (Business Insider, 21 Apr 2022; Wikipedia repeats "only performed fundraising once in the spring of 2020 when it raised $3 billion"); **$1 billion of new cash raised in 2023** ⚠ (Business Insider, 28 Jul 2025); and the state described thereafter as **closed to new capital** — "closed to new investments since last year [2023]" ✅ as published (Hedgeweek, 19 Aug 2024, citing a source with knowledge of operations) and "closed to new capital, like many of its peers" ⚠ (Business Insider, 28 Jul 2025). The two descriptions of 2023 (a $1 billion raise and a $1 billion withdrawal) are both press-reported and both sit in the record; this guide reports them side by side rather than choosing one, because they are not necessarily contradictory — a firm can raise into one vehicle while investors redeem from another.

### 6.4 Headcount as a Capital Proxy

Headcount is the only capacity measure this firm reports to a regulator, and it is revealing:

| Date | Employees | Basis | Source |
| --- | --- | --- | --- |
| Start of 2022 | 674 (including 108 portfolio managers) | Investor update | ⚠ Business Insider, 21 Apr 2022 / 2 Feb 2023 |
| June 2022 | nearly 700 (106 PMs) | Investor documents | ⚠ Business Insider, 2 Feb 2023 |
| End of 2022 | down 11% from June; PMs down to **81** | Investor documents | ⚠ Business Insider, 2 Feb 2023 |
| 2022 (annual) | **688** | Regulatory filings compiled by Old Well Labs | ⚠ Business Insider, 28 Jul 2025 |
| 1 January 2024 | **646** | Encyclopedic infobox | ⚠ Wikipedia |
| Mid-2025 | **roughly 650** | Old Well Labs data | ⚠ Business Insider, 28 Jul 2025 |
| 19 August 2026 | **707** total, **332** performing investment advisory functions, 2 state-registered adviser representatives, 0 broker-dealer representatives | The adviser's own filing | ✅ Form ADV Item 5.A/5.B |

Two readings follow. The press and infobox series (674 → 688 → 646 → ~650) shows a firm that **stopped growing headcount after 2022** — which is unusual for a multi-manager platform in that period, and which Business Insider attributes to a deliberate "strategic" approach to expansion ⚠. The filing's **707 / 332** is the more precise and more current figure ✅, and it is worth separating the two numbers: 332 people performing investment advisory functions (portfolio managers, analysts and research) inside a 707-person firm is a ratio a bank can sanity-check against the reported ~81–109 portfolio managers of 2022–2025.

### 6.5 What the Firm Itself Says About Its Size

**Nothing.** The firm's website, extracted 23 September 2026, publishes no AUM figure, no return, no employee count and no fund list ✅. Its Form ADV reports regulatory AUM because the SEC requires it ✅, and reports employee counts because the SEC requires it ✅ — but the adviser publishes no marketing AUM anywhere examined. The practical consequence for a bank: **the firm's size must be obtained from the client directly** (through its audited financial statements and fund documents), because the public record can only ever supply a dated third-party estimate.

---

## 7. The Performance Record — What Is Reported and What Is Not

### 7.1 The Firm Discloses No Returns

A private fund does not disclose its returns, and this firm does not. It publishes **no** performance figure on its website ✅ (verified: the extracted site pages contain none), no performance in its Form ADV — Form ADV asks for AUM and fees, not returns ✅ — and no public investor letters or factsheets are in the record examined ❌. Every return number about ExodusPoint in circulation reaches the public through one of three routes: **investor documents seen by journalists**, **people described as familiar with the firm's returns**, or **performance databases** (PivotalPath, HFRI, BarclayHedge) quoted in press coverage. This section reports those figures as exactly that, and asserts none of them as fact.

### 7.2 The Press-Reported Return Record

| Year | Reported return | Attribution and basis |
| --- | --- | --- |
| 2018 (part-year) | Not reported as a figure this pass | — |
| 2019 | **6.8%**, against a hedge-fund average of 9% | ⚠ Bloomberg, 8 Jan 2020, via Wikipedia's citation |
| 2020 | **13.5%** | ⚠ via Wikipedia's citation of press; the same source that gives 2024's 11.3% |
| 2021 | **4.9%** | ⚠ Business Insider, 2 Feb 2023 (investor documents) |
| 2022 | **5.5% to 6%**, depending on share class; up 3.3% at June | ⚠ Business Insider, 2 Feb 2023 (investor documents seen by Insider and people familiar with the matter) |
| 2023 | Not reported as a full-year figure this pass; the year is characterised by the $1bn of withdrawals | ⚠ |
| 2024 | **11.3%** — described as its best result since 2020 | ⚠ via Wikipedia's citation of Bloomberg |
| 2024 (partial) | **up 3.6% through July 2024** | ✅ as published (Hedgeweek, 19 Aug 2024, citing PivotalPath) |
| 2025 (partial) | **up more than 9%** through June; **more than 18%** over the twelve months to July 2025 | ⚠ Business Insider, 28 Jul 2025 |
| 2025 (full year) | **18%**, including **2.1% in December** — reported as **the firm's best year on record** | ✅ as published (Business Insider, 6 Jan 2026, "a person close to the New York-based manager told Business Insider") |
| Aggregate, 2018–2022 | 7.86% average annual return on one pension's investment (see §7.3) | ⚠ Business Insider, 21 Apr 2022, on a Teacher Retirement System of Texas document |

Three caveats apply to the whole table. **The figures are press-reported**, often resting on a single unnamed source or on documents a journalist saw but a reader cannot; the outlet and date are given in every row for exactly that reason. **Some rows are undated at source** — a number that appears in a 2025 article and is described only as "this year" is recorded here with the article's date, not with a claimed as-of date. **No figure in this table is verified by the firm**, and where the honest answer is that only the firm and its investors know, this guide says so rather than implying the press consensus is the truth.

### 7.3 The One Investor Datapoint With a Named Source

The most useful performance datapoint in the public record is not a return table — it is a **named public pension's own document**. Business Insider reported on 21 April 2022 that a document from the **Teacher Retirement System of Texas** showed **a $500 million investment in ExodusPoint made in August 2018 had turned into a little more than $607 million three years later — an average annual return of 7.86%** ⚠. It is a single investor's experience over a single three-year window, and it is not a fund return (it reflects the investor's own share class and timing), but it is the only investor-level figure in this guide's record that comes from a *named, institutional, publicly accountable* source rather than from an unnamed person. The same article reported that **80% of the $3 billion raised in spring 2020 came from existing investors** and quoted a source close to the firm describing the investor base as wanting "up 8, 10, 12 every year" rather than "up 20, up 20, down 30" ⚠.

### 7.4 The Peer Comparison, Dated

Performance claims about this firm are meaningless without the peer set of the year, because the firm's own reported numbers have ranged from comfortably ahead of the industry (2025) to well behind its direct rivals (2022). The two dated comparisons:

| Fund | 2022 | 2024 (through July) | 2025 |
| --- | --- | --- | --- |
| **ExodusPoint** | **5.5–6%** ⚠ | **3.6%** ✅ as published | **18%** ✅ as published |
| Citadel (Wellington) | 38.1% ⚠ | 8.82% ✅ as published | 10.2% ✅ as published |
| Millennium | 12.4% ⚠ | — | 10.5% ✅ as published |
| Point72 | 10.25% ⚠ | — | 17.5% ✅ as published |
| Balyasny | 9.7% ⚠ | — | 16.7% ✅ as published |
| D.E. Shaw (Composite) | — | — | 18.5% ✅ as published |
| Schonfeld (Partners) | — | — | 12.5% ✅ as published |
| Multi-strategy average / HFRI | 4.25% industry decline (HFRI Fund Weighted Composite) ⚠ | 6.3% multi-strategy average (PivotalPath) ✅ as published | — |

Sources: 2022 figures — Business Insider, 2 Feb 2023 (investor documents and people familiar); 2024 — Hedgeweek, 19 Aug 2024 (PivotalPath and firm-reported media figures); 2025 — Business Insider, 6 Jan 2026 (people familiar with each firm's returns). The 2025 table is reproduced in full in §11.2.

### 7.5 The Honesty Note

What a bank can honestly say about this firm's performance, and what it cannot:

- **Can say:** that the firm publicly discloses no return; that the press has reported double-digit returns in two of the last three reported years (11.3% in 2024 ⚠, 18% in 2025 ✅ as published); that it reported mid-single-digit returns in 2021 and 2022 ⚠; and that it trailed its direct peers badly in 2022 and led several of them in 2025.
- **Cannot say:** whether any reported return is gross or net, in which share class, after what fees, over what precisely defined period, or whether it applies to the master fund's investors as a whole.
- **Should not do:** present a headline number — "the fund made 18% in 2025" — without the outlet, the date and the words "reported". Four sentences of qualification are cheaper than one mis-stated fact in a credit file.

---

## 8. The People, the Leadership and the Culture

### 8.1 The Principals

**Michael Gelband** is the firm's public face: the website says "The Firm is led by Michael Gelband" ✅ (23 Sep 2026); the press describes him as **founder and, since 2024, sole chief investment officer**, a former Lehman Brothers executive who ran Millennium's fixed-income division, who **does not manage money at ExodusPoint**, and who reported to former colleagues that his Millennium unit generated **$7 billion of trading revenue over eight years** ⚠ (Business Insider, 28 Jul 2025, 21 Apr 2022; Forbes, 25 Oct 2017). He is the personalisation of the firm's fixed-income identity (§4.3).

**Hyung Lee** is the co-founder whose public profile is deliberately low and whose role has changed: co-founder with Gelband, previously Millennium's head of equities ⚠ (Business Insider; Bloomberg); **stepped down in 2024** and is now described as a **senior advisor**, having relocated to Puerto Rico ⚠ (Business Insider, 28 Jul 2025). The Asia-Pacific hiring coverage of 2022 treats the firm's Asian build-out as part of the multi-strategy hiring wave of that year ⚠ (Bloomberg, 30 Nov 2022, via Wikipedia's citation).

### 8.2 The Second Tier

The following names are reported, dated and attributed; none is confirmed by the firm itself, and none should be treated as a register of officers ✅/⚠:

| Name | Reported role | Source |
| --- | --- | --- |
| **Garrett Berg** | **President and Chief Operating Officer**, with the firm since launch; oversees all non-investment activities; previously Global Head of Marketing and Investor Relations at Hoplite Capital and, before that, at BlueCrest | ✅ firm's leadership page, 23 Sep 2026 (role also reported by Business Insider, 28 Jul 2025) |
| **Kunal Kumar** | **Chief Risk Officer**, named January 2025 (joined 2023 from Balyasny, where he was Co-Head of Global Macro Risk) | ✅ firm's leadership page, 23 Sep 2026 |
| **Dev Joneja** | **Chairman of Risk**, with the firm since launch; previously Global Head of Risk at Millennium and 13 years at Lehman Brothers | ✅ firm's leadership page, 23 Sep 2026 |
| **Adam Galeon** | Runs **long-short equities**; a 2023 hire after Lee's units were reallocated | ⚠ Business Insider, 28 Jul 2025 |
| **Michael Lapsa** | Runs **systematic strategies**; a 2023 hire | ⚠ Business Insider, 28 Jul 2025 |
| **Peter McConnon** | **Senior managing director, fixed income and macro**; formerly head of London macro for Balyasny | ⚠ Business Insider, 28 Jul 2025 |
| **Jon Hoffman** | A former Lehman Brothers trader who worked with Gelband at the bank; described as one of the firm's best-known portfolio managers, running a **basis-trade strategy** | ⚠ Business Insider, 28 Jul 2025 |
| **Robert Bovo** | Head of the **WestWind** fundamental equities unit launched in 2021; a former Millennium PM who joined in 2019 | ⚠ Business Insider, 2 Feb 2023 |
| **Erik Addington** | **Chief Financial Officer** (joined November 2018); previously ten years at Nomura as CFO Americas and CFO Asia ex-Japan, and Lehman Brothers 1998–2008 | ✅ firm's leadership page, 23 Sep 2026 |
| **Tim Cruise** | **Global Chief Compliance Officer**, named January 2025 (joined the firm in 2020; previously Two Sigma and Morgan Stanley) | ✅ firm's leadership page, 23 Sep 2026 |
| **Emily Locher** | **General Counsel** (joined 2026; previously General Counsel and CCO at Capstone Investment Advisors) | ✅ firm's leadership page, 23 Sep 2026 |
| **Arram Han** | **Head of Infrastructure** (joined 2021; previously 24 years at Société Générale, including CTO of the Americas) — the only public evidence that infrastructure leadership sits at executive level (§9.2) | ✅ firm's leadership page, 23 Sep 2026 |
| **Xinyu Liu** | **CEO of the Singapore entity**, per the MAS directory | ✅ MAS FID, 23 Sep 2026 |

The table's strongest entry is **Garrett Berg**: named on the firm's own leadership page as President and COO, with the firm since launch, and also the firm's published **press contact** ✅ (exoduspoint.com, 23 Sep 2026) — a firm-side, dated corroboration of a role the press also reports. The table also carries an explicit warning about provenance: the personnel rows sourced to **Business Insider's 28 July 2025** feature rest on unnamed sources and cannot be cross-checked, and one class of attribution was **withdrawn during verification** — earlier drafting read three of these names (Berg, Addington, and a "chief compliance officer" signature) out of the firm's Form ADV, but the filing's extracted text layer contains **no personal names at all**, so each of those attributions was re-sourced to the firm's own website (where it exists) or dropped ⚠.

### 8.3 Churn and the Advisory Bench

The firm's early years were marked by senior departures, and the pattern persisted into the reset: Business Insider's 2 February 2023 analysis reports a November 2022 shake-up that "jettisoned a string of PMs", a nearly 25% fall in portfolio managers over the second half of 2022, and an 11% fall in headcount from June ⚠; a WSJ piece of 24 October 2019 reports an executive leaving after less than two years ⚠. By 2025 the firm is described as having moved several founding figures "into advisory roles" — Lee among them ⚠ (Business Insider, 28 Jul 2025) — although the firm's own leadership page still lists the founding risk head, **Dev Joneja, as Chairman of Risk** ✅ (exoduspoint.com, 23 Sep 2026), which is a title change rather than a departure. The pattern a bank should read into this is the ordinary one for a platform seven years past launch: **founder-era roles giving way to operating executives**, with the investment-side leadership now drawn substantially from outside hires (Galeon, Lapsa, McConnon, Kumar) ⚠.

### 8.4 Culture Markers and the Technology Complaints

The firm's own website describes its culture in three words — **"integrity and entrepreneurship"** — and promises a "professional, results-driven, and collegial environment" ✅ (exoduspoint.com, 23 Sep 2026). The press record is less admiring and is reported here with its provenance: Business Insider's 21 April 2022 piece, based on conversations with "more than a dozen employees, industry insiders, and people close to ExodusPoint", describes a firm that had **struggled with its technology build-out**, suffered **"a rash of high-profile exits"**, and attracted **"mixed reviews" of Gelband's management style** ⚠. The same reporting describes recruiting pressure — a market where "arbitrageurs" rather than swing-for-the-fences investors are wanted ⚠. None of this is a finding about the firm; it is what named journalism reported on a dated basis, and it is included because a bank's reputational-diligence file should know what the press has said, not only what the firm says.

### 8.5 The People Table

| Aspect | Fact | Status |
| --- | --- | --- |
| Leader | Michael Gelband, founder; sole CIO since 2024 | ✅ (firm's site: "led by Michael Gelband") / ⚠ (CIO detail) |
| Co-founder | Hyung Lee; stepped down 2024, now senior advisor, relocated to Puerto Rico | ⚠ press |
| President and COO | Garrett Berg (firm's leadership page; also the firm's press contact) | ✅ firm's site, 23 Sep 2026 |
| Chief risk officer | Kunal Kumar (named January 2025; previously Balyasny) | ✅ firm's leadership page |
| Chairman of Risk | Dev Joneja (with the firm since launch; previously Millennium's Global Head of Risk) | ✅ firm's leadership page |
| Chief compliance officer | Tim Cruise (Global CCO, named January 2025; joined 2020) | ✅ firm's leadership page |
| Chief financial officer | Erik Addington (joined November 2018; previously Nomura) | ✅ firm's leadership page |
| General counsel | Emily Locher (joined 2026) | ✅ firm's leadership page |
| Head of infrastructure | Arram Han (joined 2021; previously Société Générale) | ✅ firm's leadership page |
| Singapore CEO | Xinyu Liu | ✅ MAS FID, 23 Sep 2026 |
| Employees | 707 total; 332 in investment advisory functions (19 Aug 2026) | ✅ Form ADV |
| Portfolio managers | 81 at end-2022 (down from 106 in June); 108–109 reported in early 2022 | ⚠ press |
| Named senior hires 2023 | Adam Galeon (long-short equities), Michael Lapsa (systematic), Peter McConnon (fixed income and macro) | ⚠ press |
| Ownership of the firm by named individuals | **Not established** — see §2.6 | ❌ not found |

---

## 9. The Technology Question — Cross-Referenced

### 9.1 The Class of Stack (Cross-Referenced)

A multi-manager platform of this size runs a technology estate of a recognisable class: a central risk and position-keeping system across all pods, an order-management layer with FIX connectivity to dozens of venues and dealers, market-data capture and storage, a market-neutral portfolio-financing and margin engine, a data platform for quant research, and an internal developer estate. That class of stack is described in [Citadel LLC](citadel_llc_guide.md) **§7** (platforms, low-latency infrastructure, market data and FIX connectivity, quant research and AI/ML), with the deeper engineering references in [Quantitative Developer Skillset](../technology/quantitative_developer_skillset_guide.md) and [Low-Latency C++ Development](../technology/low_latency_cpp_development_guide.md). **This guide does not re-derive any of it, and it attributes no platform, vendor or system to ExodusPoint.** Its specific stack is not publicly documented ✅ (verified absence across the sources examined) — and any guide that names one would be inventing it.

### 9.2 The Only Public Technology Traces

Five traces exist in the public record, and they are traces, not an architecture:

1. **Cloud-hosted books and records, disclosed to the regulator.** Form ADV Schedule D **§1.L** (locations of books and records other than the principal office) lists five external entities: a **cloud/computing provider** (the entity whose Redmond, Washington headquarters address is used as the section-attribution example in §2.3), a **software-as-a-service provider**, a **document-management vendor**, an **HR/finance SaaS vendor** and a **financial-institution group** ✅ (Form ADV §1.L, 19 Aug 2026). §12.2 explains why this guide describes these by category rather than reproducing the counterparty names in full. The inference — flagged as inference — is that the firm's corporate back-office runs on commercial cloud and SaaS rather than on-premise only ⚠.
2. **A UK technology company.** **EXODUSPOINT CAPITAL MANAGEMENT UK TECHNOLOGIES, LTD** (12329778), incorporated 22 November 2019 and Active, SIC **62090** ("other information technology service activities") ✅ (Companies House, 23 Sep 2026). A platform that incorporates a dedicated UK technology entity nineteen months after launch is a platform building in-house engineering — a reasonable inference from the register, and one flagged as inference ⚠.
3. **An investor portal with a Microsoft-stack session cookie.** The firm's own privacy notice lists the cookies used on its site, including **`.ASPXFORMSAUTH`** — described by the firm as the session cookie that keeps users logged in "on our investor portal" — and **`incap_ses_*`**, described as session cookies for "DDoS protection and web application firewall protection" ✅ (exoduspoint.com privacy notice, effective 3 April 2023). From that a reader can infer, as an inference ⚠, an ASP.NET-based investor portal behind a commercial WAF/CDN — a normal, unremarkable and non-proprietary stack for investor reporting.
4. **A London equities build-out.** Hedgeweek has reported the appointment of a **Pictet trader to the London equities team** ✅ as published (hedgeweek.com, related coverage listed alongside its 19 Aug 2024 piece) — evidence of a live London trading desk, not of any technology.
5. **A named head of infrastructure.** The firm's own leadership page lists **Arram Han as Head of Infrastructure**, with the firm since 2021 and previously 24 years at Société Générale, including as Chief Technology Officer of the Americas ✅ (exoduspoint.com leadership page, 23 Sep 2026). It is the only public evidence that infrastructure is represented at executive level — and it is placed last here deliberately, because it answers a bank's most obvious technology question ("who owns your platform?") with a name and a title and nothing whatever about the stack.

### 9.3 The 2022 Criticism, Attributed

The only substantive reporting on the firm's technology is critical and dated: Business Insider's **21 April 2022** feature, on the basis of more than a dozen interviews, describes a firm that **"struggled with its tech build-out"** in its early years, with infrastructure described as a factor in senior departures, and quotes insiders saying performance would follow "as more infrastructure is put in place and teams are built out" ⚠. That is a *reported* characterisation from 2022 about a 2018–2021 period, and it should be read with the same scepticism as any other anonymous-source claim — but it is also the only public evidence about the firm's technology, and a bank's diligence file is better for knowing it.

### 9.4 What Is Not Public

Not disclosed anywhere examined: the firm's OMS/EMS, its risk platform, its market-data vendors, its cloud provider by contract (only the books-and-records address is disclosed, which is indicative but not dispositive), its FIX connectivity, its data platform, its colocation arrangements, its research infrastructure, its headcount split between engineering and investment, and any technology spend figure ❌. A bank that needs any of this for a diligence or credit purpose must **ask the client and document the answer**, because the public record does not contain it.

---

## 10. The Singapore and the Asia Angle

### 10.1 The Singapore Entity — a Verified Licence, Not an Office Rumour

The most important Singapore fact in this guide is that ExodusPoint's Singapore presence is **not a marketing office of a foreign manager**: it is a **Singapore-incorporated company holding a Capital Markets Services licence**. As at **23 September 2026**, the **MAS Financial Institutions Directory** (queried this pass — the primary register) shows ✅:

| Field | Value |
| --- | --- |
| Name | EXODUSPOINT CAPITAL MANAGEMENT SINGAPORE, PTE. LTD. |
| Status | Incorporated in Singapore |
| Licence | **Capital Markets Services Licensee** |
| Regulated activities | **Fund Management**; **Dealing In Capital Markets Products** |
| Products covered by the dealing activity | Securities · Collective Investment Schemes · Exchange-Traded Derivatives Contracts · Over-The-Counter Derivatives Contracts · Spot Foreign Exchange Contracts for the Purposes of Leveraged Foreign Exchange Trading |
| Key personnel | CEO — **XINYU LIU** |
| Business address | Ocean Financial Centre, 10 Collyer Quay, #19-06/08, Singapore 049315 |
| Telephone | +65 65766110 |
| Website | exoduspoint.com |

The **ACRA-derived corporate record** for the same company (UEN **201824688K**) adds the corporate facts ✅: incorporated **19 July 2018**, status **Live Company**, entity type **Local Company**, company type **Private Company Limited by Shares**, SSIC primary code **66309**, registered office **77 Robinson Road, #13-00, Robinson 77, Singapore 068896**, **15 officers**, annual return dated **16 June 2026**, accounts due **31 July 2027**. The firm's own website lists the same **Ocean Financial Centre #19-06/08** address as its Singapore office ✅ (exoduspoint.com, 23 Sep 2026).

Two observations a bank can use. **First**, the licensing breadth matters: fund management **plus** dealing in capital markets products — including OTC derivatives and leveraged FX — means the Singapore entity is licensed to conduct market-facing dealing activity, not merely to market the group's funds. **Second**, the incorporation date (**19 July 2018**) sits inside the launch year and **before** the firm's first reported London/Paris/Asia footprint (2022), which places Singapore among the **founding wave** of the firm's international expansion rather than among its later additions ⚠ (the 2022 Business Insider reference to London, Paris, Singapore and Hong Kong offices is the first press mention in this record).

### 10.2 The Two Singapore Addresses, Explained

Two Singapore addresses appear in this guide and they are both correct, because they serve different statutory purposes:

| Address | What it is | Source |
| --- | --- | --- |
| **Ocean Financial Centre, 10 Collyer Quay, #19-06/08, Singapore 049315** | The **business address** on the MAS licence record and the **office address** on the firm's own website | ✅ MAS FID; ✅ firm's site |
| **77 Robinson Road, #13-00, Robinson 77, Singapore 068896** | The **ACRA registered office** — the statutory address at which legal service and official correspondence are received | ✅ ACRA-derived record |

A **registered office is not necessarily a place of business**: it is the company's legal address for service, and Singapore companies — particularly subsidiaries of foreign groups and fund managers — commonly appoint a corporate-services provider's address for the purpose. The public records show Robinson 77 to be exactly that kind of building: the ACRA-derived record lists numerous unrelated entities at **77 Robinson Road, #13-00** in the same period ✅ (opengovsg.com corporate record, 23 Sep 2026). That is consistent with a corporate-secretarial or registered-office-provider address, and the guide states it as a **reasonable reading of the register, not as a documented agency relationship** ⚠. The operational conclusion for a bank is practical: **the address to use for notices under the MAS licence and for the client's own business is the Collyer Quay address; the address to use for legal service is Robinson Road**, and a CDD file should record both with their different purposes.

### 10.3 The MAS Licensing Context (Cross-Referenced)

The Singapore regime — the distinction between a **Capital Markets Services (CMS) licence** under the Securities and Futures Act, the **RFMC and LFMC** fund-management tiers, and the exemptions that let a foreign manager operate without one — is set out in [Hedge Funds Singapore](hedge_funds_singapore_guide.md) §2 and in [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md); it is not re-derived here. What matters for this firm is one line of classification: **ExodusPoint Singapore is a full CMS licensee, not an RFMC or LFMC**, and it holds a dealing licence in addition to fund management ✅ (MAS FID). For a Singapore bank, that has three practical consequences: the entity is a **direct MAS supervisee** with its own CEO (Xinyu Liu) accountable to the regulator ✅; it is a **licensed counterparty for the dealing activities listed**, which bears on product-eligibility and onboarding questions; and the bank's own MAS Notice 626 AML/CFT obligations attach to the relationship on the basis of a licensed financial institution, not an exempt foreign manager ✅ (the MAS overlay is cross-referenced to [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md)).

### 10.4 The Rest of Asia — Hong Kong, Tokyo, Dubai, Jersey

The firm's own website lists eight locations, and the non-US half of the list is the Asian and offshore footprint ✅ (exoduspoint.com, 23 Sep 2026):

| Location | Address on the firm's site |
| --- | --- |
| New York (headquarters) | 65 East 55th Street, 6th Floor, New York, NY 10022 |
| Stamford | The Metro Center, One Station Place, Suite 430, Stamford, CT 06902 |
| London | 20 St James's Street, London SW1A 1ES |
| Jersey | International Finance Centre 5, First Floor, The Esplanade, St Helier JE2 3BY |
| Dubai | Unit R2-02B, Level 2, ICD Brookfield Place, Dubai International Financial Centre |
| **Hong Kong** | 18th Floor, Prosperity Tower, 39 Queens Road Central, Hong Kong SAR |
| **Singapore** | Ocean Financial Centre, 10 Collyer Quay, #19-06/08, Singapore 049315 |
| **Tokyo** | Marunouchi Nijubashi Building, Level 2, 3-2-2 Marunouchi, Chiyoda-ku, Tokyo 100-0005 |

Three notes. **London's address matches the registered office of all three UK entities exactly** (20 St James's Street) ✅ — the strongest corroboration in this guide between a website and a register. **Jersey and Dubai** are the classic offshore-financing and DIFC-marketing locations, and the firm's own privacy notice treats **DIFC** data-protection law as applicable to it, which independently corroborates a Dubai establishment ✅ (privacy notice, effective 3 Apr 2023). **No register-level check** was performed this pass for Hong Kong (SFC), Tokyo (JFSA), Dubai (DFSA) or Jersey (JFSC) entities — the offices are established by the firm's own site, and any licence claim about those locations is **out of scope and not asserted** ❌/⚠ (§14.4). Press coverage of the firm's Asia hiring (Bloomberg, 30 Nov 2022, "Hedge Fund Hiring Spree Sweeps Big Multistrategy Firms in Asia") places it within the region's multi-strategy build-out ⚠.

### 10.5 The Repository Claim This Guide Settles

The [Hedge Funds Singapore](hedge_funds_singapore_guide.md) guide records at its **line 360** that ExodusPoint is among the managers **"commonly reported with Singapore offices (⚠ none re-verified this pass)"**. **That claim is now settled, and it resolves in the affirmative with a stronger finding than the original claim:** ExodusPoint's Singapore presence is not merely a "Singapore office" of a foreign manager but a **Singapore-incorporated private company (UEN 201824688K, incorporated 19 July 2018, Live) holding an MAS Capital Markets Services licence for Fund Management and Dealing in Capital Markets Products** ✅ (ACRA-derived record and MAS Financial Institutions Directory, both read 23 September 2026). The correct upgrade is from *"commonly reported with Singapore offices (⚠ unverified)"* to *"a locally incorporated, MAS-licensed CMS entity, verified at the regulator's own directory"* ✅. Any future pass on [Hedge Funds Singapore](hedge_funds_singapore_guide.md) line 360 can replace the ⚠ with a ✅ and cite this guide's §10.1, whose evidence is dated 23 September 2026.

---

## 11. The Peer Positioning — One Platform Among Several

### 11.1 The Peer Set

ExodusPoint is one multi-manager platform among several, and this guide does not present it as representative of the type. The comparison set that the press and the firm's own investors actually use is: **Millennium Management** (Englander — the founders' former employer and the closest structural analogue), **Citadel** (Griffin — the archetype firm guide in this repository), **Point72** (Cohen), **Balyasny Asset Management**, **Schonfeld Strategic Advisors**, **D.E. Shaw** (whose Composite fund is the multi-strategy flagship), **Jain Global** (the newest entrant), and — for the Asia context — **Dymon Asia** and **Pinpoint Asset Management** ⚠ (all as used in the dated coverage below). Each differs from ExodusPoint in scale, vintage, strategy mix and risk appetite; the [Hedge Fund Guide](hedge_fund_guide.md) §3.9 owns the general comparative anatomy of the type, and [Citadel LLC](citadel_llc_guide.md) §3 owns the archetype's specifics.

### 11.2 The 2025 Scorecard, Dated

The most complete dated peer table available this pass is Business Insider's, published **6 January 2026** (originally **2 January 2026**) ✅ as published, sourced to people familiar with each firm's returns:

| Rank (by 2025 return) | Fund | December 2025 | Full-year 2025 |
| --- | --- | --- | --- |
| 1 | AQR Apex | 3.0% | 19.6% |
| 2 | D.E. Shaw Composite | n/a | 18.5% |
| 3 | Dymon Asia | 1.9% | 18.0% |
| **4** | **ExodusPoint** | **2.1%** | **18.0%** |
| 5 | Boothbay | 1.2% | 17.9% |
| 6 | Point72 | 1.7% | 17.5% |
| 7 | Balyasny | 1.2% | 16.7% |
| 8 | Walleye | 2.0% | 15.5% |
| 9 | LMR | 3.4% | 12.7% |
| 10 | Schonfeld Partners | 2.3% | 12.5% |
| 11 | Pinpoint Asset Management | 1.1% | 11.6% |
| 12 | Millennium | 1.9% | 10.5% |
| 13 | Citadel Wellington | 1.8% | 10.2% |
| 14 | Verition | 1.6% | 7.5% |
| 15 | Jain Global | 1.5% | 3.7% |

The same article records the scale context in which those returns were earned: **Millennium $83.5 billion**, **Citadel $72 billion**, **Balyasny $31 billion** ✅ as published. On that scale ladder ExodusPoint — reported at **~$11 billion** in July 2025 ⚠ and approximately **$12 billion** in December 2025 ⚠ — sits in the second rank of platforms by size while, in 2025, outperforming most of the largest ones. In 2022 the relationship was inverted: it made **5.5–6%** while Citadel made **38.1%** and Millennium **12.4%** ⚠ (§7.4). Both statements are dated and attributed; the pattern they describe is exactly why the repository insists on dated figures for this firm.

### 11.3 The Fee and Hurdle Convergence

One genuinely informative comparison is structural rather than numerical: the **cash hurdle**. ExodusPoint adopted a cash hurdle on its performance fees in **2024** — collecting performance fees only when it beats a Treasury bill — and Business Insider reported the same change across multi-strategy peers, with the effect of lowering headline fees and raising net returns ⚠ (Business Insider, 28 Jul 2025; the industry piece dated 20 Sep 2024). Combined with the firm's **pass-through reimbursement** arrangement (✅ Form ADV Item 5.E) and the Wikipedia-cited characterisation of unlimited cost pass-through since launch ⚠, the picture is a firm that has been under fee pressure from its investors and has responded with hurdle mechanics rather than with a headline management-fee cut. The general economics are cross-referenced to [Hedge Fund Guide](hedge_fund_guide.md) §4.

### 11.4 The Comparison Caveat

Three cautions, all of which the repository's honesty convention requires:

- **Not representative.** A multi-manager platform's mid-teens returns in a good year and mid-single-digit returns in a bad one are a property of the type, not of this firm alone; the pod architecture that produces them is described generically in [Hedge Fund Guide](hedge_fund_guide.md) §3.9 and should not be re-derived from one firm's record.
- **Not apples to apples.** ExodusPoint's risk sits predominantly (reported ~75%) in fixed income ⚠, whereas several peers of similar size are equity- or quant-led — so a year-by-year return ranking compares genuinely different risk profiles, which is precisely why the peer set is reported in a table with dates rather than as a claim that the firm is "better" or "worse" than its rivals.
- **Not verified by the firms.** Every number in §11.2 comes from Business Insider's sourcing, not from a filing. Firms in the private-fund world settle returns by share class, after different fee arrangements and with different reporting lags; treating a press table as a league table of truth is the error this section exists to prevent.

---

## 12. The Bank Interface — What Is Disclosed and What Is Not

### 12.1 The Covenant This Guide Follows

This guide follows the repository's rule for firms of this type: **no bank, prime broker, administrator, auditor or service provider is asserted as this firm's counterparty or provider unless the firm or the provider has publicly said so**, and prime brokerage in particular is **confidential by market convention**. Two consequences apply throughout, and the second is the one that needs care. First, **no bank is presented as this firm's client or counterparty** anywhere in this guide, and **Cymbal Bank is the only bank persona used in the worked example** (§13), which is fictional by construction. Second, **this guide does not reproduce the provider names the firm files** — but the reason matters, and it is *not* that the relationships are secret. The firm's own Form ADV files a prime-broker and custodian panel fund by fund, with registration numbers and office cities, and that filing is public; what this guide declines to do is launder that disclosure into an assertion. A name copied out of a filing into a narrative guide loses its provenance, its date and its context, and the repository's house rule is that no real institution is presented here as this firm's counterparty. The honest formulation is therefore: **the primes are named in the filing, and they are not named here** — a choice, not a gap (§12.2, §12.4).

### 12.2 What the Firm's Own Filings Disclose

The one public channel in which this firm's provider relationships surface is its **own regulatory filings and its own website** — and it is worth being precise about what those documents contain, because it is more than a reader might expect:

| Disclosure channel | What it contains | Treatment in this guide |
| --- | --- | --- |
| Form ADV **Schedule D §7.B.24** | For each private fund, whether it uses prime brokers and, for each one, the **name, SEC registration number, CRD number, principal office city/country and whether it acts as custodian** — a multi-dealer panel recorded fund by fund | ✅ the disclosure exists; **names not reproduced** here (§12.4) |
| Form ADV **Schedule D §7.B.25** | Custodian records per fund, including legal and business names and the location of the responsible office | ✅ exists; names not reproduced |
| Form ADV **Schedule D §7.B.23** | The auditor for each fund, including the firm's name, the office responsible, independence, PCAOB registration number and whether opinions were unqualified | ✅ exists; names not reproduced |
| Form ADV **Schedule D §1.L** | The five external entities at which books and records are kept — one cloud provider, one software-as-a-service provider, one document-management vendor, one HR/finance SaaS provider and one financial-institution group | ✅ exists; **described by category** in §9.2, with the cloud provider's Redmond address used as the §2.3 example; **names not reproduced** |
| The firm's website privacy notice | The firm names **one** service provider expressly: its Cayman fund administrator (§12.3) | ✅ named here, because the firm itself published it |

The honest summary is therefore **not** "the firm discloses nothing" — the firm's filings disclose a great deal to the SEC, and that material is public — but rather: **the firm publishes almost nothing about its counterparties in its marketing materials, and this guide does not reproduce the counterparty names it does file**. Any bank, journalist or researcher who wants those names can read them in the filing; what this guide will not do is assert them as relationships in the firm's voice or reproduce them as if the firm had announced them.

### 12.3 The One Provider the Firm Names Itself

There is exactly one exception in this guide's record, and it is a firm-side, public, dated statement: the firm's **Global Privacy Notice** (effective **3 April 2023**), published on its own website, states that it may disclose investor personal information to "our affiliates and service providers, such as **fund administrators (including, but not limited to, Northern Trust Global Fund Services Cayman Limited and its affiliates)**, custodians, broker-dealers, accountants, and lawyers, as necessary to effect and administer the fund" ✅ (exoduspoint.com privacy notice, extracted 23 Sep 2026). That entity also appears in the firm's Form ADV Schedule D §1.L as a location at which books and records are kept ✅ — an independent corroboration of the same relationship from the regulatory filing. It is named here for exactly one reason: **the firm published it itself**, on its own website, in a notice addressed to the public, and the repository's rule permits naming a provider that the firm or the provider has publicly identified. It is the only service provider this guide names, and the only one the firm names outside its regulatory filings.

The precise way to use this fact is: **the firm itself publicly identifies a Cayman fund administrator for its funds**, which is a legitimate, firm-attributed disclosure. Everything beyond it — the identity of the custodians, the auditing firm, the prime brokers, the bank relationships — is **not** stated by the firm outside its filings, and is therefore **not asserted here**. This is the single most useful piece of bank-interface information in this guide, because the administrator is the counterparty through which a bank's own fund-services, custody and financing conversations with the client's funds actually flow.

### 12.4 Why Prime Brokerage Is Confidential by Default

A prime-brokerage relationship is confidential by market convention: dealer panels are negotiated privately, financing spreads and margin terms are commercially sensitive, and both sides treat the panel as non-public absent a disclosure obligation. This firm is no exception, and this guide infers nothing from size or strategy — a platform does not "obviously" use any particular dealer, and the appropriate response to any claim that it does is to ask for the filing, or the client, or both. Two practical notes for a bank:

- **The panel is wide, and the filing shows it.** Form ADV Schedule D §7.B.24 records prime-broker relationships fund by fund, and the extracted filing shows **a multi-dealer panel across the six funds rather than a single-prime arrangement** ✅ — the ordinary arrangement for a platform of this type, and the reason operational due diligence on such a client is a *multi-prime* exercise (multiple margin statements, multiple collateral schedules, multiple middle-office reconciliations).
- **The filing also shows custody split across prime brokers and additional custodians** ✅ (§7.B.25 custodian records), which is the arrangement a bank should expect when it discusses custody-adjacent services with the client's funds.

### 12.5 What a Bank Actually Sees

Putting the disclosure question to one side, the interface a bank actually has with a firm like this is best described by **where the money and the paper flow**, generically and without naming any provider:

- **Fund-level financing and margin** — the funds' portfolios are financed through dealer counterparties; a bank's exposure-assessment conversation is with the fund vehicles (the Cayman master and the feeders of §5.1), not with the adviser.
- **Cash and treasury** — the operating entities (the US LP, the UK LLP and companies, and the Singapore private company of §10.1) need multi-currency operating accounts; the Singapore entity's CMS licence means a Singapore bank onboards **a locally licensed financial institution**, not an exempt foreign manager.
- **Fund flows** — subscriptions and redemptions move through the feeder funds, typically via the administrator; the adviser's own accounts are cost-centre accounts.
- **FX and dealing** — the Singapore entity is licensed, among other things, for **dealing in OTC derivatives and leveraged FX** ✅ (MAS FID), which is directly relevant to any FX or derivatives conversation with that entity.
- **Cross-border structure** — the group spans the US, the UK, Jersey, the DIFC, Singapore, Hong Kong and Tokyo ✅ (firm's site; UK and Singapore registers), which means CRS/FATCA classification, transfer-pricing documentation and local-entity onboarding all sit inside a single client relationship.

The general mechanics of these relationships — margin lending, securities financing, rehypothecation, settlement and the trade lifecycle, the KYC/AML overlay — are owned by [Citadel LLC](citadel_llc_guide.md) §10 and [Hedge Fund Guide](hedge_fund_guide.md) §7, and are not re-derived here.

---

## 13. The Cymbal Bank Worked Example — A Multi-Manager Fund as a Banking Client

### 13.1 The Scenario

*A hypothetical, and explicitly so.* **Cymbal Bank** (the repository's fictional Singapore full-bank persona) has been asked to onboard the Singapore licensed entity of a global multi-manager platform — call the client "the Group", mirroring the public facts of this guide — and to extend it an operating-account and FX relationship, with a financing conversation to follow at the fund level. The bank's client is **the Singapore CMS licensee**; the bank's potential financing counterparties are **the Cayman-domiciled funds**; and the bank's information problem is **the one §6 describes**: no published AUM, no published performance, and a public record full of dated third-party estimates. Nothing in this section describes an actual relationship between any real bank and this firm; no relationship is asserted, and no real institution is named as a client or counterparty.

### 13.2 Onboarding a Cayman Master-Feeder Group

The KYB (know-your-business) file for this client looks nothing like a corporate loan file, and the disambiguation work is the reason. The steps a competent file records:

1. **Establish the legal-entity chain first** — the same chain this guide verified in §2.2: the US adviser (CRD 294156 ✅), the GP, the six funds and their jurisdictions ✅, the three UK entities ✅, and the Singapore company with its UEN and MAS licence ✅. Each entity in the group is a separate onboarded or recorded party, with the register extract dated.
2. **Resolve the names.** Screen **"EXODUSPOINT"** and screen **"ATLAS CAPITAL"** — the second because the firm's own Form ADV files it as an additional business name ✅ (§2.1). Record a disposition for every hit, including the deliberate ones: a screening file that contains only "exoduspoint" is incomplete on the firm's own filing.
3. **Fix the addresses correctly** (a step that trips up real files): use **Ocean Financial Centre, 10 Collyer Quay #19-06/08** for the Singapore business address, and **77 Robinson Road #13-00** as the ACRA registered office ✅, noting that the second is a service address rather than a place of business (§10.2).
4. **Evidence the licence at the regulator**, not from the client's own marketing: print the MAS Financial Institutions Directory entry showing the CMS licence, the activities (fund management and dealing) and the **CEO Xinyu Liu** ✅, and date the extract.
5. **Establish the ownership and control picture, and record what is unestablished.** The public record does **not** disclose the group's ownership chain or ultimate beneficial owners (§2.6, §14.4). The file therefore records the client's own constitutional documents and ownership declarations as the primary evidence, notes the absence of a public ownership record, and **does not invent a UBO from press coverage of the founders** — a point worth stating in the file precisely because it is tempting to write "owned by Michael Gelband and Hyung Lee" on the strength of journalism.
6. **Record the regulatory perimeter across the group**: SEC-registered adviser in the US ✅, CMS licensee in Singapore ✅, UK entities incorporated and Active with their FCA status **unresolved** ⚠ (the register could not be queried this pass, §2.3) — and treat the unresolved item as **open**, to be closed by the client providing its FCA reference or permission notice, rather than assumed either way.

### 13.3 The Financing and Margin Conversation

If the relationship extends to fund-level financing, the conversation is with the **fund entities** — the Cayman master and feeders of §5.1 — and the analytic questions are those of [Citadel LLC](citadel_llc_guide.md) §10 and [Hedge Fund Guide](hedge_fund_guide.md) §7, applied to a fund whose public record is silent. Three firm-specific handling points:

- **Assume a multi-prime panel** ✅ (the filing records prime-broker relationships fund by fund, §12.2/§12.4) and design the operational due diligence accordingly: the bank's exposure sits alongside other financers, and collateral, margin and rehypothecation terms must be read against the fund's other facilities, not in isolation.
- **Do not size the client from the press.** The "$11 billion" figure and the "$159.4 billion" figure are different measures on different dates (§6.1, §6.2). The financing analysis needs the **fund's audited financial statements and its actual exposure and NAV**, obtained from the client or its administrator — the public record cannot support a limit.
- **Recognise the fixed-income skew.** The firm's risk is reported at roughly **75% in fixed income** ⚠ (§4.3). For a financing bank that means the collateral pool is weighted toward rates, macro and basis positions with their own haircut, duration and correlation profiles — a materially different book from an equity-focused platform of the same reported size, and one that should shape margin schedules rather than being treated as generic "hedge fund" risk.

### 13.4 The Reconciliation Question — NAV, RAUM and GAV

This is the worked example's most useful lesson, and it is a public-record lesson before it is a banking one. The same firm generates four different "size" numbers, and a bank that reconciles them wrongly will set the wrong limit:

| Measure | This firm's illustration | What it is |
| --- | --- | --- |
| **Net investor capital (reported AUM)** | ~$11–12 billion ⚠ (Business Insider, Hedgeweek) | Investors' money in the funds, as reported by third parties |
| **Net asset value of the master fund** | Not publicly disclosed ❌ | Fund equity after liabilities — the number that actually matters for credit |
| **Current gross asset value (as filed)** | $158.6 billion for the master fund ✅ (Form ADV §7.B) | Gross value of positions, including financed exposure |
| **Regulatory assets under management (as filed)** | $159,413,741,487 across 6 accounts ✅ (Form ADV Item 5.F) | The adviser's filing measure of the portfolios it manages |

The disciplined procedure: **reconcile from the audited NAV outward** — that is, from the fund's own statements, to gross exposure and to the financing stack — and treat the regulatory and gross figures as **corroborating the presence of significant leverage**, not as the size of the client. The arithmetic that a ~$159 billion gross figure sits alongside a ~$11–12 billion reported investor-capital figure is a public observation (§6.2) and it is exactly the kind of observation that should trigger a financing conversation about margin, gross and net exposure limits — **not** a conclusion about the firm's leverage ratio, which is not disclosed anywhere.

### 13.5 The Regulatory Overlay — MAS, CRS/FATCA and the Singapore Entity

Onboarding a **Singapore CMS licensee** changes the overlay in three concrete ways:

- **The bank's own MAS obligations attach directly.** MAS Notices 626 and 626A (AML/CFT for banks) apply to the relationship; the MAS regime and the persona conventions are cross-referenced to [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) rather than restated here.
- **Tax transparency is a structural feature, not an afterthought.** With a predominantly non-US client base (the adviser's filing reports **67%** of clients as non-US persons and **$115.06 billion** of its regulatory AUM attributable to non-US persons ✅) and a Cayman master with US and Cayman feeders ✅, the CRS and FATCA classification of each entity is a first-week task, and the Singapore entity's own status has to be documented separately from the funds'.
- **Reporting is duplicated by design.** The Singapore entity reports to MAS; the adviser reports to the SEC; the funds report to CIMA ✅. A bank's periodic review should treat these as **independent evidence sources** — the thing that makes this client, for all its opacity about capital and performance, unusually *documented* on the structural side.

### 13.6 The Monitoring and the Re-Onboarding Triggers

Ongoing monitoring for this client class is event-driven as much as flow-driven, and the public record supplies the trigger list. Re-onboard or refresh on:

- **A change of licence status** at MAS for the Singapore entity (a CMS licence is reviewed against the entity, its CEO and its activities) ✅.
- **A new fund or a new jurisdiction** — the firm's Form ADV discloses six funds today ✅; a seventh, or a new feeder, changes the financing counterparties.
- **A change in the reported capital regime** — the firm has moved from raising (2020, 2023) to closed ✅/⚠ (§6.3); a re-opening, or a large redemption run, materially changes the counterparty.
- **A leadership or control change** — the departure of a co-founder in 2024 is a worked example of exactly the event a periodic review should pick up ⚠ (§8.1).
- **A public-information change** — a new regulatory registration, a new licence in a new jurisdiction, or the appearance of an adverse press item, each of which should be checked against the file's own record rather than absorbed from the internet. The repository's screening conventions (see [Fircosoft](fircosoft_guide.md) for sanctions/AML screening themes) apply to the name-cluster problem of §2.1 in particular.

### 13.7 The Recommendation — and the Condition the Client Resists

**The recommendation splits in two, and that split is the point.** The onboarding and operating-account part is approved; the financing part is not, and it is not approved on the client's timetable.

| Recommendation | Decision | Reasoning |
| --- | --- | --- |
| Operating accounts, multi-currency cash and FX for **the Singapore CMS licensee** | **Approve**, on standard terms, no credit exposure | The entity is locally incorporated, MAS-licensed for fund management and dealing, verified at the regulator's directory, with a named CEO and two dated addresses (§10.1–§10.2). This is ordinary institutional onboarding of a supervised financial institution ✅ |
| **Fund-level financing** against the Cayman master and feeders | **Not yet approved.** Conditions attached, none waivable at this stage | The client's own public record cannot support a limit: no published AUM, no published performance, no disclosed gross or net exposure, no disclosed NAV, and no disclosed financing stack (§6, §7, §12.5) |
| A **support or keepwell undertaking** from the adviser, or an explicit statement that none will be given | **Required before any fund-level exposure is booked** | The group is structurally separated — adviser, GP, funds and operating companies are different legal persons, and no guarantee relationship is disclosed anywhere (§2.2, §13.4) |

**The conditions Cymbal attaches.** Before a fund-level facility is priced: (1) audited financial statements for the master fund and each feeder, plus the administrator's monthly NAV pack; (2) monthly reporting of **gross exposure, net exposure and the aged financing stack, facility by facility** — the multi-prime view that a single-prime-assumption limit would miss (§13.3); (3) an initial-margin and haircut schedule agreed against the actual collateral mix, not against a generic "hedge fund" bucket, because the firm's risk is reported at roughly 75% fixed income and rates ⚠ (§13.3); (4) a documented statement of every other prime broker, custodian and financing counterparty, or confirmation that the client will not provide one — either answer is workable, silence is not; and (5) an entity-level representation that Cymbal's exposure ranks as it expects within the financing stack.

**The condition the fund resists — and where the file lands.** The client accepts (1) to (3) in principle. It **resists** the rest, and the resistance is structural rather than adversarial. It resists **(2) to the pod level**: platform managers treat pod-level exposure and position data as the platform's own intellectual property, and disclosing it to a financing bank is for them the same category of act as disclosing strategy. It resists **(4)**: naming the other financers means putting its own panel in a bank's file, which is precisely what this guide's §12 does not do either — the client's instinct and the guide's editorial rule converge on the same point. And it resists **(5)** most of all, because a multi-manager platform keeps the adviser and the funds legally apart on purpose, and an undertaking from the adviser to a fund-level facility is the one concession that would cross the architecture the platform exists to maintain. **Cymbal's position, put plainly in the file:** the bank does not waive (4) or (5); the operating relationship proceeds regardless; fund-level financing stays **unapproved and at zero limit** until the documentation arrives, and if the client will never provide an adviser-level undertaking, then Cymbal's structure is **fully collateralised at the fund level with daily margin and no reliance on group support** — a softer facility at a lower size, which Cymbal prices accordingly and the client is free to decline. There is no version of this recommendation in which the bank sets a limit from a reported number; the reconciliation exercise of §13.4 exists so that nobody in the credit chain is tempted to try.

### 13.8 The Lessons

1. **Structure is verifiable; capital is not.** The entities, licences, addresses and funds of this group are all established at registers ✅. Its AUM, performance, investors and counterparties are not established anywhere ❌/⚠. The onboarding file should look exactly like that: strong on structure, explicit about the gaps.
2. **Read the filing, not the press, for structure — and read the press, not the filing, for nothing.** The press supplies dates, colour and personnel; the filing supplies the counterparty categories, the funds and the numbers. Mixing them up is how a file acquires a prime broker it never had.
3. **Name the second business name.** "Atlas Capital" appears in the firm's own filing ✅; a screening file that misses it has a documented gap.
4. **Distinguish the addresses, the entities and the measures** — registered office from place of business, adviser from funds, NAV from regulatory AUM. This client's public record punishes every conflation, which is precisely why it is a good worked example.

---

## 14. The Claims Audit — Verified, Flagged, Rejected

### 14.1 The Verified Claims (✅)

| Claim | Evidence | Date |
| --- | --- | --- |
| Legal name EXODUSPOINT CAPITAL MANAGEMENT, LP; CRD 294156; SEC file 801-112901; CIK 1736225; LEI 549300X6BK8XW1RP4G25 | Form ADV (SEC/IAPD) for CRD 294156, read from the cached PDF text extraction and re-fetched this pass | Filed 19 Aug 2026 |
| Principal office 65 East 55th Street, New York, NY 10022; phone (646) 940-9600; business hours 8:00–17:00 | Form ADV Item 1.F; firm's own site | 19 Aug 2026 / 23 Sep 2026 |
| 15 other US offices with per-office advisory employee counts (Stamford 12, Austin 4, San Francisco 3, Bala Cynwyd 3, San Jose 2, ten at 1 or 0) | Form ADV Item 1.F(5) and Schedule D §1.F | 19 Aug 2026 |
| 707 employees, of whom 332 perform investment advisory functions; 67% of clients non-US persons | Form ADV Item 5.A/5.B/5.C | 19 Aug 2026 |
| Six private funds, names and jurisdictions as listed in §5.1; all filed as "hedge fund"; master-feeder with EXODUSPOINT PARTNERS MASTER FUND, LP (ID 805-1823187184) as master | Form ADV Schedule D §7.B | 19 Aug 2026 |
| Regulatory AUM $159,413,741,487 discretionary across 6 accounts; $0 non-discretionary; $115,064,838,605 attributable to non-US persons | Form ADV Items 5.D and 5.F | 19 Aug 2026 |
| Master fund current gross asset value $158,566,481,481 | Form ADV Schedule D §7.B question 11 | 19 Aug 2026 |
| Compensation includes a percentage of assets, performance-based fees, and "Other: PASS THROUGH REIMBURSEMENT" | Form ADV Item 5.E | 19 Aug 2026 |
| Criminal, regulatory-action and civil-judicial disclosure reporting pages: "No Information Filed" | Form ADV DRP pages | 19 Aug 2026 |
| Leadership named on the firm's own leadership page: Michael Gelband (CEO, CIO, Co-Founder), Garrett Berg (President and COO), Dev Joneja (Chairman of Risk), Kunal Kumar (CRO, named Jan 2025), Tim Cruise (Global CCO, named Jan 2025), Erik Addington (CFO), Emily Locher (General Counsel, 2026), Arram Han (Head of Infrastructure) | exoduspoint.com leadership page, extracted this pass | 23 Sep 2026 |
| The Form ADV's text layer carries no personal names (CCO field blank; zero hits on every name tested) — the reason no person is asserted here from the filing | Form ADV PDF text extraction, tested this pass | 19 Aug 2026 filing / tested 23 Sep 2026 |
| Additional business name "ATLAS CAPITAL"; websites exoduspoint.com, a LinkedIn page, a Glassdoor page and http://www.atlas-capital.com | Form ADV Schedule D §1.B and §1.I | 19 Aug 2026 |
| Five external books-and-records locations, including a cloud provider, a SaaS provider, a document-management vendor, an HR/finance SaaS provider and a financial-institution group | Form ADV Schedule D §1.L | 19 Aug 2026 |
| Fund-director records for the Cayman funds (names withdrawn — see §14.2) | Form ADV Schedule D §7.B question 3(a) — disclosure exists; names not readable in the filing's text layer | 19 Aug 2026 |
| Singapore entity EXODUSPOINT CAPITAL MANAGEMENT SINGAPORE, PTE. LTD. is a MAS **Capital Markets Services licensee** for Fund Management and Dealing in Capital Markets Products, with CEO Xinyu Liu, at Ocean Financial Centre #19-06/08 | MAS Financial Institutions Directory, queried this pass | 23 Sep 2026 |
| Singapore entity UEN 201824688K, incorporated 19 July 2018, Live Company, private company limited by shares, SSIC 66309, registered office 77 Robinson Road #13-00, 15 officers | ACRA-derived corporate record (opengovsg.com) | 23 Sep 2026 |
| Three UK entities, all Active at 20 St James's Street, London SW1A 1ES: UK, LLP (OC421170, 22 Feb 2018); SERVICES UK, LTD (11220419, 22 Feb 2018, SIC 64205); UK TECHNOLOGIES, LTD (12329778, 22 Nov 2019, SIC 62090) | UK Companies House company records | 23 Sep 2026 |
| The firm's own site's founding/launch sentence; eight office locations; strategy, risk and culture language | exoduspoint.com, extracted this pass | 23 Sep 2026 |
| The firm publicly names a Cayman fund administrator in its own privacy notice (effective 3 April 2023) | exoduspoint.com privacy notice, extracted this pass | 23 Sep 2026 |
| The firm publishes no AUM and no performance figure on its website | exoduspoint.com, extracted this pass (verified absence) | 23 Sep 2026 |
| 18% return for 2025, including 2.1% in December, reported as the firm's best year on record | Business Insider, as published | 6 Jan 2026 |
| AUM of $11.04bn at 30 Jun 2024; $12.02bn at end-2023; $13.1bn at Dec-2022 peak; up 3.6% through July 2024 vs a 6.3% peer average; closed to new investments since 2023 | Hedgeweek, citing a Reuters report and the firm's 13F | 19 Aug 2024 |

### 14.2 The Flagged Claims (⚠)

| Claim | Why flagged | Source and date |
| --- | --- | --- |
| Fund-director and signatory attributions previously read out of the Form ADV ("Erik Addington and Garrett Berg as Cayman fund directors"; a named chief compliance officer on the execution page) | **Withdrawn during verification**: the filing's extracted text layer contains no personal names at all (the CCO field of Item 1.J extracts blank; zero hits for every name tested), so the attributions were unverifiable as stated. Where the same people are confirmed, it is from the firm's own leadership page — Berg (President/COO), Addington (CFO), Tim Cruise (Global CCO, named Jan 2025) | Form ADV PDF text extraction, tested 23 Sep 2026; exoduspoint.com leadership page, 23 Sep 2026 |
| Founded April 2017; Delaware LP formed 2017; GP = ExodusPoint Capital Partners GP, LLC owned by Michael Gelband and Hyung Soon Lee | Secondary/aggregator; the ownership element was not confirmed in the primary filing (Schedule A/B lies beyond the cached portion) | radientanalytics.com firm page, read this pass |
| ~$11.7bn AUM as at 31 Dec 2025 | Secondary aggregator only | same |
| $8.5bn launch size, and the "largest hedge fund launch ever" framing | Press-reported; the FT article itself was unreachable this pass (scrape error), so the FT headline and date are cited at second hand | FT 21 May 2018; Bloomberg 21 May 2018; Business Insider 2022/2023/2025 |
| Founders' prior roles at Millennium (Gelband fixed income, Lee equities); Gelband declined an ownership stake and left; Millennium's arbitration; Schonfeld's 2018 suit and its outcome | Press and secondary, resting on unnamed sources or court reporting not re-read this pass | Forbes 25 Oct 2017; Dealbreaker 4 Jan 2018; Bloomberg Law 2 Oct 2018 / 9 Apr 2019; Business Insider 21 Apr 2022 |
| "Passes on unlimited costs to investors" and "more expensive than peers" | Press characterisation; the filing confirms pass-through reimbursement but not the "unlimited" framing | Business Insider, cited via Wikipedia |
| Return figures other than 2025 and the 2024 partial: 6.8% (2019), 13.5% (2020), 4.9% (2021), 5.5–6% (2022), 11.3% (2024) | Press/encyclopedic, several cited via Wikipedia to Bloomberg; none re-read at the outlet this pass | as listed in §7.2 |
| AUM series 2022–2025 ($13.5bn, $13.9bn, $13.1bn, $11bn, ~$12bn); 646 employees at 1 Jan 2024; 688 (2022) and ~650 (2025) headcount | Press, investor documents, infobox and third-party data (Old Well Labs) | Business Insider 21 Apr 2022 / 2 Feb 2023 / 28 Jul 2025; Wikipedia |
| Reported leverage implied by comparing $159bn regulatory AUM with ~$11–12bn of reported investor capital | Arithmetic across two different measures on different dates — presented as an observation, not a leverage figure | §6.2 |
| All personnel statements except the eight leadership-page biographies and the Singapore CEO, and except the four investment-side names (Galeon, Lapsa, McConnon, Hoffman) set out in §8.2 | Press only, mostly one unnamed source per statement | Business Insider 28 Jul 2025, 2 Feb 2023 |
| Cash hurdle adopted 2024 | Press only | Business Insider 28 Jul 2025 |
| ~75% of risk in fixed income | Press only, attributed to a person with direct knowledge | Business Insider 21 Apr 2022; 28 Jul 2025 |
| 2023 events: $1bn of withdrawals and $1bn of new capital | Press/aggregator; the two sit side by side in the record | Bloomberg 27 Mar 2024; Business Insider 28 Jul 2025 |
| UK entities' FCA authorisation status | The FCA register could not be queried to a valid result this pass; no claim made either way | register.fca.org.uk, attempted 23 Sep 2026 |
| The "Atlas Capital" business name beyond the fact of the filing | The filing records the name and a website; the domain did not resolve for extraction and its purpose is unknown | Form ADV §1.B/§1.I |
| Dallas TX office ZIP code 33401 as extracted | Anomalous (33401 is a West Palm Beach ZIP); recorded as filed and flagged rather than reconciled | Form ADV Schedule D §1.F |
| Five of the six funds' gross-asset-value figures and all six beneficial-owner counts | As filed, but attribution checked only by field sequence because the cached extraction interleaves records | Form ADV Schedule D §7.B |

### 14.3 The Rejected or Not-Found Claims (❌)

| Claim | Status | Evidence |
| --- | --- | --- |
| Any published AUM figure by the firm | ❌ | The firm's website publishes none; its filing reports regulatory AUM only because the SEC requires it |
| Any published performance figure by the firm | ❌ | No return appears on the website or in the filing |
| A named investor list, subscription amounts, side letters or fee schedules | ❌ | None found in any source examined |
| The firm's ownership chain, UBOs or shareholdings by named individuals | ❌ | Not disclosed in any source examined; the ADV Schedule A/B table was outside the cached portion |
| The identity of the firm's prime brokers, custodians or auditor as asserted relationships | ❌ | The filing discloses them; this guide does not assert or reproduce them (§12) |
| A Cymbal Bank relationship of any kind with this firm | ❌ | The worked example (§13) is explicitly hypothetical; no such relationship exists or is implied |
| The firm's specific technology stack (OMS/EMS, risk platform, market data, cloud contract, colocation) | ❌ | Not publicly documented |
| An entity or licence check for Hong Kong, Tokyo, Dubai or Jersey | ❌ | Out of scope this pass; only offices are established, by the firm's own site |
| "Founded in 2018" | ❌ | Contradicted by the firm's own statement ("founded in 2017 … began managing investor capital in 2018") — 2018 is the launch year |

### 14.4 What Could Not Be Verified

The repository's honesty convention requires a dedicated statement of the gaps. Despite the register queries, filing reads and press work of this pass, the following **could not be verified**, and none of them is asserted anywhere in this guide:

- **The firm's own AUM, in any period.** The firm publishes none; every figure in §6.1 is a third-party claim. There is no primary or firm-sourced AUM number in existence as far as this pass could establish, other than the regulatory measure of §6.2 — which is not the same thing.
- **The firm's performance, in any period, from the firm.** The 2025 figure (18%) and the 2024 partial (3.6% through July) are verified **as published** by named outlets on named dates; every other return is press-reported at second hand, and several were not re-read at the original outlet this pass (the FT page returned a scrape error; Bloomberg pages were cited via Wikipedia rather than read).
- **The ownership and control structure of the group.** Who owns the adviser, the GP, the Singapore and UK entities; the founders' percentage interests; whether there are outside shareholders. The primary filing's ownership schedule lay beyond the cached portion of the extraction, so even that route was unavailable ⚠.
- **The UK entities' FCA authorisation status.** The FCA Financial Services Register could not be searched to a valid result by automated extraction this pass (the interface returned a malformed result set), so no claim is made that any UK entity is or is not authorised. **This is an open item, not a negative finding.**
- **The identity and status of the Jersey, DIFC, Hong Kong and Tokyo entities.** Their offices are on the firm's own site; their legal entities, licences and statuses were not researched at register level this pass.
- **The prime-broker, custodian and audit arrangements as relationships.** They are disclosed in the firm's filing; this guide deliberately does not reproduce or assert them (§12.2, §12.4), so the reader should treat "what is the firm's prime broker?" as **answered in the filing and not answered here**, by choice rather than by absence of evidence.
- **"Atlas Capital."** Filed as the firm's own additional business name ✅, but what it is for, whether a vehicle sits behind it, and whether the associated domain is operated by the firm could not be established (the domain did not resolve for extraction) ⚠.
- **The precise meaning of the fund-level gross-asset-value figures beyond the master fund.** The master fund's figure is corroborated against the filing's own regulatory-AUM total; the other five figures and the beneficial-owner counts are as-filed but attribution-checked only by field sequence, because the cached extraction of the filing interleaves fund records across page boundaries — a genuine methodological limit on this pass, recorded rather than hidden.
- **The firm's risk numbers.** Gross exposure, net exposure, leverage, VaR, drawdown limits and fund liquidity terms are not disclosed anywhere examined; the only public trace is the arithmetic of §6.2.
- **The firm's investor base beyond counts and the one named pension.** No investor list exists publicly; the 67%-non-US and $115.06bn figures are the filing's aggregates, not names.
- **Tool limitations encountered this pass (recorded as limitations, never as evidence of absence).** Two `web_search` queries returned **empty result sets** (one on the firm's UK/FCA entities, one on the May 2018 Bloomberg launch article) — an empty search is a tool failure here, not a finding. The Financial Times article on the 2018 launch returned a scrape error. The FCA register search returned a malformed result set. The live SEC IAPD firm-summary page is JavaScript-driven and returned no usable content, which is why this guide relies on the PDF filing rather than the HTML summary. The cached Form ADV text extraction is truncated at roughly 2 MB of a 3.7 MB document, so Items 6–11 and the ownership schedules were not readable locally, and the document's page-level repetition means per-record values must be attribution-checked by field sequence.

### 14.5 The Anti-Patterns — Symptom, Cause, Guardrail

Seven failure modes recur whenever a firm of this type is written up, onboarded or credit-assessed. Each is stated as symptom, cause and guardrail, because the guardrail is the only part that survives contact with a real file.

| # | Anti-pattern | Symptom in the file | Cause | Guardrail |
| --- | --- | --- | --- | --- |
| 1 | **Treating a reported AUM as current** | "Client AUM: $12 billion" with no date, no outlet, no basis, sitting in a field that looks like a fact | Press figures are point-in-time and reported inconsistently; the client publishes none at all, so a dated number is the only defensible form | Every capital figure carries **outlet + date + basis**, or it does not go in the field. For this firm the internal-reconciliation default is the **regulatory AUM of §6.2** (filed, dated 19 Aug 2026) *and* the reported investor-capital estimate, kept visibly apart |
| 2 | **Treating a press-reported return as evidence** | "Fund returned 18% in 2025, so the pod model works" — a performance statement in a credit or marketing memo | A private fund discloses no returns; the numbers come from investor documents journalists saw or unnamed sources, and are often gross-or-net-unspecified | Never assert a return; record it as **reported**, with outlet and date, in a section explicitly headed as reported (§7). Four sentences of qualification are cheaper than one mis-stated fact |
| 3 | **Onboarding the wrong legal entity** | The adviser is onboarded for a fund-level facility; the Singapore office is onboarded as if it were the manager; a brand name is onboarded as a counterparty | A group of this kind has 12+ distinct legal persons (§2.2), and name-screening on the brand does not identify the contracting entity | Establish the chain first (§2.2), identify **which entity contracts for what**, date every register extract, and record the addresses' different statutory purposes (§10.2) |
| 4 | **Assuming a prime-brokerage relationship from size or strategy** | "A platform this size obviously uses [named dealer] as prime broker" | Prime-brokerage panels are confidentially negotiated; size is not a disclosure, and a plausible name is not evidence | Say only what is disclosed: the panel is **filed** and not reproduced here (§12.2, §12.4); ask the client, and document the answer |
| 5 | **Assuming a foreign office implies a local entity — or the reverse** | Marking an office as a subsidiary (or a licence as marketing) without a register extract | A website office list and a corporate register answer different questions; a registered office is often a service address, not a place of business | Get the entity **and** its licence at its own register, and where none exists **say so as a verified negative rather than assuming omission means exemption**. This guide's own case is the counter-example: Singapore is a locally incorporated CMS licensee (§10.1), while Jersey, DIFC, Hong Kong and Tokyo are established as *offices* only and no licence is claimed (§14.4) |
| 6 | **Repeating a founder's prior affiliation from memory** | "Founder X, formerly head of Y at Bank Z" — a sentence with no source and often the wrong title | Founder narratives are the most repeated and least re-verified claims in the genre, and they propagate through secondary sources | Verify at a **named outlet with a date** (§2.5), name the outlet inline, and where the only support is an aggregator's summary of a filing, mark it ⚠ — as this guide does for the GP ownership line (§14.2) |
| 7 | **Confusing regulatory AUM with the client's size** | A limit, a ranking or a "we manage $159 billion" claim built on the adviser's filing figure | Form ADV asks for a portfolio-level measure, not net investor capital, and the gap here is a factor of roughly ten (§6.2, §13.4) | Use the measure for what it is, and reconcile from **audited NAV outward** (§13.4) |

The unifying failure is a **provenance collapse**: a number, a name or a relationship migrating from a dated, attributed source into a file as an unattributed fact. Every guardrail above is the same instruction in different clothes — keep the outlet, keep the date, keep the register extract, and where a private firm discloses nothing, write that down and stop.

---

## 15. The Glossary

| Term | Meaning |
| --- | --- |
| **3(c)(1) / 3(c)(7)** | The exclusions from the definition of "investment company" in the Investment Company Act of 1940 under which a private fund can be offered without registration; the election is recorded for each of this firm's funds in its Form ADV ✅. |
| **ACRA** | The Accounting and Corporate Regulatory Authority of Singapore — the registry behind UEN 201824688K ✅. |
| **ADV (Form ADV)** | The SEC's registration and reporting form for investment advisers; for this firm, CRD 294156, most recently amended 19 August 2026 — the single richest public document about it ✅. |
| **Atlas Capital** | An additional business name filed by ExodusPoint Capital Management, LP in its own Form ADV ✅; purpose and any entity behind it not established ⚠ (§2.1). |
| **Beneficial owner count** | The number of beneficial owners a private fund reports in its ADV record (e.g., 103 for the master fund) ✅/⚠ — a count, not an investor list. |
| **Cash hurdle** | A performance-fee condition under which the manager earns performance fees only after returning more than a Treasury bill; this firm adopted one in 2024 ⚠. |
| **Cayman Islands Monetary Authority (CIMA)** | The Cayman regulator with which the firm's Cayman funds are registered, as disclosed in the filing ✅. |
| **CIK** | The SEC's Central Index Key — 1736225 for this adviser ✅. |
| **Cost pass-through** | An arrangement in which fund expenses are borne by the fund rather than the manager; the filing records "PASS THROUGH REIMBURSEMENT" as a compensation arrangement ✅. |
| **CRD number** | The FINRA/IARD identifier for an adviser or broker-dealer — 294156 for this firm ✅. |
| **CRS / FATCA** | The automatic-exchange and US withholding regimes that make entity-by-entity tax classification a first-week banking task for a group with a Cayman master and US/Cayman feeders (§13.5). |
| **CMS licence** | Singapore's Capital Markets Services licence under the Securities and Futures Act; the firm's Singapore entity holds one for fund management **and** dealing ✅ (MAS FID, 23 Sep 2026). |
| **DRP (Disclosure Reporting Page)** | The Form ADV pages reporting criminal, regulatory and civil-judicial disclosures; all three read "No Information Filed" for this adviser ✅. |
| **Existing-business-name screen** | The screening step against names a client files as its own — "Atlas Capital" here — as distinct from the brand name (§13.2). |
| **Feeder fund / master fund** | The two-layer fund structure in which feeders of different tax profiles invest into one master; this firm's structure has three feeders plus two internal vehicles into one Cayman master ✅ (§5.2). |
| **FCA** | The UK Financial Conduct Authority; its Financial Services Register could not be queried to a valid result this pass, so no claim is made about the firm's UK entities' authorisation ⚠. |
| **13F filing** | The quarterly US equity-holdings filing by institutional managers; press reports have used it to infer this firm's AUM movements ✅ as published (Hedgeweek, 19 Aug 2024). |
| **Fund administrator** | The service provider that computes NAV and processes subscriptions and redemptions; the firm names a Cayman fund administrator in its own privacy notice ✅ (§12.3). |
| **GAV (gross asset value)** | The value of a fund's positions before liabilities, disclosed per fund in the ADV (§5.1, §13.4); distinct from NAV and from regulatory AUM. |
| **GP (general partner)** | The entity that manages a fund and carries its liability — here EXODUSPOINT PARTNERS GP, LLC, per the filing ✅. |
| **JPY / JFSA, DFSA, JFSC** | The Japanese, Dubai and Jersey regulators — named here only to record that **no** register check was performed with them this pass ❌. |
| **LEI** | Legal Entity Identifier — 549300X6BK8XW1RP4G25 for this adviser ✅ (Form ADV Item 1.P). |
| **LFMC / RFMC** | Licensed / Registered Fund Management Company — the Singapore fund-manager tiers; ExodusPoint Singapore is neither, being a full CMS licensee ✅ (cross-ref [Hedge Funds Singapore](hedge_funds_singapore_guide.md) §2). |
| **MAS FID** | The Monetary Authority of Singapore's Financial Institutions Directory — the register on which this firm's Singapore CMS licence was verified ✅ (23 Sep 2026). |
| **MAS Notice 626 / 626A** | The MAS notices imposing AML/CFT obligations on Singapore banks; the overlay for §13.5 (cross-ref [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md)). |
| **Multi-manager platform ("multi-strat")** | The firm type this subject belongs to: a central risk and allocation layer funding autonomous trading teams; its architecture is owned by [Hedge Fund Guide](hedge_fund_guide.md) §3.9 and not re-derived here. |
| **NAV** | Net asset value — the fund's equity after liabilities; the measure that matters for credit, and the one **not** publicly disclosed for this firm's funds ❌. |
| **Old Well Labs** | The industry data tracker whose compilation of regulatory filings supplied the 688 (2022) and ~650 (2025) headcount figures cited by Business Insider ⚠. |
| **Pass-through reimbursement** | The formal name, in the firm's own filing, of the cost-pass-through arrangement of §4.6 ✅. |
| **PCAOB** | The US audit oversight body; the filing records the auditors' PCAOB registration numbers per fund ✅ (names not reproduced here). |
| **PivotalPath** | The hedge-fund performance database cited by Hedgeweek for the 2024 peer comparison ✅ as published. |
| **Pod** | A manager/team unit within a platform, with its own capital allocation and risk limits; see [Hedge Fund Guide](hedge_fund_guide.md) §3.9 — this guide does not define it further. |
| **Private fund** | An unregistered pooled investment vehicle relying on a 3(c) exclusion; six are disclosed for this adviser ✅. |
| **RAUM (regulatory assets under management)** | The SEC's filing measure of managed portfolio value — $159,413,741,487 here ✅ — a gross, portfolio-level measure, **not** net investor capital (§6.2). |
| **Registered office** | A company's statutory address for service — 77 Robinson Road #13-00, Singapore, for the Singapore entity ✅ — distinct from its place of business (§10.2). |
| **Regulation D / Form D** | The US private-placement safe harbour and its notice; one of the firm's funds reports Form D file number 021-313831 ✅. |
| **SSIC 66309** | The Singapore industrial classification "fund management activities n.e.c." applicable to the Singapore entity ✅. |
| **Teacher Retirement System of Texas** | The US public pension whose own document supplied the only investor-level return datapoint in this guide's record: $500m to ~$607m over three years, 7.86% annualised ⚠ (Business Insider, 21 Apr 2022). |
| **UEN** | Unique Entity Number — Singapore's entity identifier; 201824688K for the Singapore company ✅. |
| **UBO** | Ultimate beneficial owner — **not** established for this group from public sources ⚠ (§2.6, §13.2). |
| **13O / 13U** | Singapore tax-incentive schemes for funds and family offices (cross-ref [Hedge Funds Singapore](hedge_funds_singapore_guide.md)); no award to any ExodusPoint entity is public ❌. |
| **At-cost / AUM band** | The "$11–14 billion" band in which third-party reporting places the firm's net investor capital ⚠ — never a firm disclosure. |
| **Cymbal Bank** | The repository's fictional Singapore full-bank persona — the only bank persona used across this repository's worked examples, and the counterparty of §13. |

---

## 16. Cross-References, Primary Sources and the Closing Summary

### 16.1 Cross-References (repository convention: sibling `banking/` guides by plain filename)

- [Citadel LLC](citadel_llc_guide.md) — the archetype firm guide and the structural template this guide follows. Its §3 owns the multi-strategy hedge-fund franchise, its §4 the performance-discipline conventions mirrored in §7 here, its §7 the trading-technology estate cross-referenced in §9, and its §10 the prime-brokerage and trade-lifecycle mechanics cross-referenced in §12–§13. **Not re-derived.**
- [Hedge Fund Guide](hedge_fund_guide.md) — the general reference: its **§3.9** owns the multi-strategy/multi-manager pod architecture and the platform model (cross-referenced at §4.1 and §11.4, deliberately not repeated); its **§4** owns fund economics (fees, hurdles, high-water marks — cross-referenced at §4.6 and §11.3); its **§7** owns the banking relationships (cross-referenced at §12.5).
- [Hedge Funds Singapore](hedge_funds_singapore_guide.md) — the Singapore manager landscape and the RFMC/LFMC/CMS regime (§10.3 here). **This guide settles the ExodusPoint item at that guide's line 360**: verified as a locally incorporated, MAS-licensed CMS entity as at 23 September 2026 (§10.5 here).
- [Crane Capital](crane_capital_guide.md) — the model for identity discipline (§2 of that guide → §2 here) and for capital-claim handling (§6 of that guide → §6 here).
- [Cayman BVI Master-Feeder](cayman_bvi_master_feeder_guide.md) — the master-feeder structure this firm's funds actually use (§5.2, §13.5); cross-referenced, not re-derived.
- [Hudson River Trading](hudson_river_trading_guide.md) — the proprietary-trading firm type; included here to mark the contrast, since ExodusPoint is an external-capital manager, not a prop firm.
- [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) — the Singapore regulatory regime, MAS Notice 626/626A and the Cymbal Bank persona conventions (§10.3, §13.5).
- [Asset Management & Alternatives](asset_management_alternatives_guide.md), [Private Equity](private_equity_guide.md), [Private Equity Singapore](private_equity_singapore_guide.md) — the alternatives context (§11).
- [Investment Portfolio Operations](investment_portfolio_operations_guide.md) — custody, settlement and reporting mechanics behind §12.5 and §13.3.
- [Fircosoft](fircosoft_guide.md) — screening themes applied to the name-cluster and second-business-name problem (§2.1, §13.6).
- [Quantitative Developer Skillset](../technology/quantitative_developer_skillset_guide.md) and [Low-Latency C++ Development](../technology/low_latency_cpp_development_guide.md) — the engineering disciplines behind a platform of this class (§9.1).

### 16.2 Primary Sources Used This Pass

- **SEC / IAPD — Form ADV for CRD 294156** (SEC file 801-112901; CIK 1736225; Other-Than-Annual Amendment dated 19 August 2026), read from the cached text extraction at `/home/ubuntu/.hermes/cache/web/reports.adviserinfo.sec.gov-a9abb92c98.md` and re-fetched from `reports.adviserinfo.sec.gov/reports/ADV/294156/PDF/294156.pdf` on 23 September 2026. Cited throughout for identity, offices, employee counts, clients, compensation, regulatory AUM, the six funds, service-provider categories and the disciplinary negatives. **Not** cited for any personal name: the filing's text layer carries none (§2.3, §14.2).
- **MAS Financial Institutions Directory** — entry for EXODUSPOINT CAPITAL MANAGEMENT SINGAPORE, PTE. LTD. (institution detail 221826), read 23 September 2026: licence type, activities, CEO, business address, telephone.
- **ACRA-derived corporate record** — opengovsg.com corporate record for UEN 201824688K, read 23 September 2026: incorporation date, entity/company type, status, SSIC, registered office, officer count, annual-return and accounts dates.
- **UK Companies House** — company records for OC421170 (EXODUSPOINT CAPITAL MANAGEMENT UK, LLP), 11220419 (EXODUSPOINT SERVICES UK, LTD) and 12329778 (EXODUSPOINT CAPITAL MANAGEMENT UK TECHNOLOGIES, LTD), plus the name search returning them, read 23 September 2026.
- **exoduspoint.com** — home, terms, privacy notice (effective 3 April 2023), careers and contact content, extracted 23 September 2026: the founding/launch sentence, strategy and risk language, the eight office locations, the press contact and the named Cayman fund administrator.
- **Hedgeweek**, "ExodusPoint's assets shrink by $1bn in H1", 19 August 2024 — AUM $11.04bn at 30 June 2024 and $12.02bn at end-2023 (citing a Reuters report and the firm's 13F), $13.1bn December 2022 peak, 3.6% through July 2024 vs a 6.3% peer average and Citadel's 8.82%, and closed-to-new-investments status.
- **Business Insider**, "Hedge funds' 2025 report card…", 6 January 2026 (originally 2 January 2026) — ExodusPoint 18% for 2025 including 2.1% in December, described as its best year on record; the full peer table of §11.2; scale context for Millennium, Citadel and Balyasny.
- **Business Insider**, "Behind Michael Gelband and $11 billion hedge fund ExodusPoint's strong year", 28 July 2025 — "$11 billion" size, >9% through June 2025, >18% over twelve months to July 2024→2025, ~75% of risk in fixed income, Lee's 2024 step-down and advisory role, the second-tier personnel list, the cash hurdle, the 2023 $1bn raise, closed-to-new-capital status, and the 688 (2022) to ~650 (2025) headcount series attributed to Old Well Labs.
- **Business Insider**, "ExodusPoint… shed assets and employees amid mixed performance last year", 2 February 2023 — 2022 returns of 5.5–6% and 2021's 4.9%, the $13.5bn/$674/108-PM start-of-2022 position, the $13.9bn June position, the $13.1bn year-end position, the PM and headcount declines, the $800m second-half asset fall, the strategy-level equity losses, and the WestWind unit.
- **Business Insider**, "How ExodusPoint — the biggest hedge fund launch in history — continues to amass billions despite lackluster returns", 21 April 2022 — the $13.5bn position and 109 PMs, the London/Paris/Singapore/Hong Kong office list, the Teacher Retirement System of Texas datapoint (7.86% annualised), the $3bn 2020 raise with 80% from existing investors, and the technology and management-style criticisms.
- **Wikipedia**, "ExodusPoint Capital Management", retrieved this pass from the cached article — used as a pointer to its cited articles (FT 21 May 2018; Bloomberg 21 May 2018; Forbes 25 Oct 2017; Dealbreaker 4 Jan 2018; WSJ 24 Oct 2019; Bloomberg 8 Jan 2020; Business Insider 2022/2023/2025/2026) and for the infobox AUM and headcount, each marked ⚠ here.
- **The parent-verified research brief for this guide** (23 September 2026) — the fact base and the hazards this guide was written against; every item was re-checked against the sources above where cheaply possible, and the one figure it asked to be verified (the $159.4bn regulatory AUM) was confirmed inside the filing ✅ (§6.2).

### 16.3 Register-Query Snapshot (as at 23 September 2026)

| Register | What was queried | Result | Meaning |
| --- | --- | --- | --- |
| SEC / IAPD (via the public Form ADV PDF) | CRD 294156 | Full filing obtained; most recent amendment 19 Aug 2026 | Adviser registered and reporting ✅ |
| MAS Financial Institutions Directory | EXODUSPOINT CAPITAL MANAGEMENT SINGAPORE, PTE. LTD. | Capital Markets Services licensee — fund management and dealing; CEO Xinyu Liu | Locally licensed entity ✅ |
| ACRA (via the corporate record) | UEN 201824688K | Incorporated 19 Jul 2018; Live Company; private company limited by shares | Locally incorporated ✅ |
| UK Companies House | Name search "ExodusPoint" | Three Active entities at 20 St James's Street, London SW1A 1ES | UK group established ✅ |
| FCA Financial Services Register | "ExodusPoint" | Search interface returned a malformed result set | **No claim made either way** ⚠ |
| Register checks for Jersey, DIFC, Hong Kong, Tokyo | — | Not performed this pass | Offices established by the firm's own site only ❌ |

### 16.4 The Closing Summary

ExodusPoint Capital Management is a study in a particular kind of asymmetry: a firm whose **structure** is unusually well documented and whose **economics** are almost entirely undisclosed. On the structural side, this pass verified at the regulators' own registers that ExodusPoint is an SEC-registered investment adviser (CRD 294156, filed 19 August 2026) with 15 US offices and 707 employees; that it runs six disclosed private funds — a Cayman master with a Cayman feeder, an intermediate Cayman vehicle and a Delaware feeder, plus two internal investor vehicles with $100,000 minimums — all filed as hedge funds, with a named general partner and a Cayman administrator the firm identifies itself; that it has a three-entity UK group at 20 St James's Street (an LLP and two companies, all Active, one of them a technology company incorporated in 2019); and that its Singapore presence is not an office rumour but a **Singapore-incorporated private company (UEN 201824688K, incorporated 19 July 2018, Live) holding an MAS Capital Markets Services licence for fund management and dealing** — the finding that settles the outstanding ⚠ at [Hedge Funds Singapore](hedge_funds_singapore_guide.md) line 360 and upgrades it to a dated ✅. On the economic side, the honest position is thinner: the firm publishes no AUM and no return; the reported capital record runs from a reported $8.5 billion launch in 2018 through a reported $13–14 billion mid-2022 peak to about $11–12 billion in 2024–2025, all of it third-party and most of it sourced to unnamed people or to journalist-seen investor documents; and the one figure the firm itself files — $159,413,741,487 of regulatory assets under management — is a gross, portfolio-level measure that must never be read as the firm's size, a distinction this guide's §6.2 and §13.4 exist to make unmissable. The performance record, reported here strictly as reported, runs from 4.9% in 2021 and 5.5–6% in 2022 (against Citadel's 38.1% and Millennium's 12.4%) to 11.3% in 2024 and 18% in 2025 — described as the firm's best year on record — and none of it is a disclosure.

For a bank, the lessons are practical ones. Verify the group at registers and date the extracts. Name-screen the second business name the client itself files, not only the brand. Keep the registered office apart from the place of business, the adviser apart from the funds, and regulatory AUM apart from net investor capital. Treat every capital and performance figure as a point-in-time claim with an outlet attached, and refuse to set a limit on any of them. Say out loud that the ownership chain is not public, rather than assembling a UBO from journalism. And recognise what this kind of firm is: not an exotic client but a structural one — a platform of pods and feeders and licensed entities held together by one thing the public record cannot show. The funding, the margin, the collateral and the counterparty relationships all run through a single group whose public filings describe six funds, four jurisdictions and a dozen offices while disclosing neither its performance nor a single bank it deals with. That is what a multi-manager platform looks like from the outside: many pods, one balance sheet.
