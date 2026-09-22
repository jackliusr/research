# Mirae Asset Securities (Singapore) Pte. Ltd.: The Entity, Its Licence and Its Place in the Group — A Comprehensive Guide to a Korean Securities Firm's Singapore Subsidiary

*Companion deep-dive in the Singapore financial-institutions series of the [jackliusr/research](https://github.com/jackliusr/research) repository — the Mirae Asset Securities (Singapore) entry alongside the two securities-firm peers [TD Securities](td_securities_software_systems_guide.md) and [Morgan Stanley](morgan_stanley_software_systems_guide.md), the Singapore-banking guides ([DBS](dbs_bank_guide.md), [Standard Chartered](standard_chartered_guide.md), [Banks in Singapore](banks_in_singapore_guide.md)), the market-infrastructure and regulation guides ([MAS Regulations & Guidelines](mas_regulations_guidelines_guide.md), [Capital Markets Architecture](capital_markets_architecture_guide.md), [Singapore Security Clearance](../technology/singapore_security_clearance_guide.md)), and the wider Asian-security-firm set ([Hedge Funds Singapore](hedge_funds_singapore_guide.md), [Private Equity Singapore](private_equity_singapore_guide.md)). This guide is about **one legal entity** — Mirae Asset Securities (Singapore) Pte. Ltd., UEN 201106132Z, incorporated in Singapore on 14 March 2011 — and deliberately **not** about the Korean listed parent, not about Mirae Asset Financial Group, and not about Mirae Asset Global Investments. It covers what the regulator's register actually says, what the entity's own website says, what the group says about itself, the corporate history in name-by-period form, the regulatory position, the technology a firm of this kind runs, and an explicit audit of what remains unknown.*

**Verification convention used throughout: ✅ = verified in this research pass (primary or secondary sources); ⚠ = flagged (inferred, approximate, single-source, or structural inference); ❌ = disputed; unmarked = structural/industry knowledge presented as such. The consolidated [Claims-Status table is in §14](#14-the-claims-audit), and the non-public specifics are collected in [§15](#15-what-could-not-be-verified).**

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Banking Domain / Singapore Financial Institutions — the entity profile, licensing, group position and operating context of Mirae Asset Securities (Singapore) Pte. Ltd. (UEN 201106132Z), a Singapore-incorporated Capital Markets Services licensee
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026
> **Companion guides:** [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md), [Market Data Consumption Guide](market_data_consumption_guide.md), [Market Data Integrity Guide](market_data_integrity_guide.md), [Market Making Singapore Guide](market_making_singapore_guide.md), [FINOS CDM Guide](finos_cdm_guide.md), [FIX Protocol Guide](fix_protocol_guide.md), [Capital Markets Architecture Guide](capital_markets_architecture_guide.md), [Financial Trading & Order Infrastructure](financial_trading_order_infrastructure.md), [Online Investment Trading Platforms Guide](online_investment_trading_platforms_guide.md), [Nasdaq Calypso Guide](nasdaq_calypso_guide.md), [Banks in Singapore Guide](banks_in_singapore_guide.md), [TD Securities Software Systems Guide](td_securities_software_systems_guide.md), [Morgan Stanley Software Systems Guide](morgan_stanley_software_systems_guide.md), [Standard Chartered Guide](standard_chartered_guide.md), [DBS Bank Guide](dbs_bank_guide.md), [SignalPlus Guide](signalplus_guide.md), [Singapore Security Clearance Guide](../technology/singapore_security_clearance_guide.md), [Univers Guide](../technology/univers_guide.md), [Hedge Funds Singapore Guide](hedge_funds_singapore_guide.md), [Private Equity Singapore Guide](private_equity_singapore_guide.md)

---

## Table of Contents

1. [The Overview, the Identity and the Decoder](#1-the-overview-the-identity-and-the-decoder)
2. [The Entity](#2-the-entity)
3. [The Group It Belongs To](#3-the-group-it-belongs-to)
4. [The Corporate History](#4-the-corporate-history)
5. [The Securities Business, and Where This Firm Sits In It](#5-the-securities-business-and-where-this-firm-sits-in-it)
6. [The Global Footprint](#6-the-global-footprint)
7. [What The Singapore Entity Is For](#7-what-the-singapore-entity-is-for)
8. [The Regulatory Position](#8-the-regulatory-position)
9. [The Technology A Firm Of This Kind Runs](#9-the-technology-a-firm-of-this-kind-runs)
10. [The Bank Interface](#10-the-bank-interface)
11. [The Honest Assessment](#11-the-honest-assessment)
12. [Worked Example: Cymbal Bank Onboards A Singapore Securities Firm](#12-worked-example-cymbal-bank-onboards-a-singapore-securities-firm)
13. [The Anti-Patterns](#13-the-anti-patterns)
14. [The Claims Audit](#14-the-claims-audit)
15. [What Could Not Be Verified](#15-what-could-not-be-verified)
16. [The Glossary, Cross-References and Closing Summary](#16-the-glossary-cross-references-and-closing-summary)

---

## 1. The Overview, the Identity and the Decoder

### 1.1 The thesis in one line

A securities firm earns its living standing between the client's order and the market, and **Mirae Asset Securities (Singapore) Pte. Ltd. is that role performed in one jurisdiction, under one regulator, inside one legal entity** — a Singapore-incorporated Capital Markets Services licensee whose registered name is recorded by the Monetary Authority of Singapore as a dealer in securities and collective investment schemes, an adviser on corporate finance, and an exempt financial adviser. Everything else in this guide is elaboration of that sentence, plus an honest account of what the sentence does *not* tell you.

### 1.2 The single most important distinction in this guide

Four things share the words "Mirae Asset" and they are not interchangeable. Confusing them is the central error this guide exists to prevent:

| # | What it is | What it is not | Identifier |
|---|---|---|---|
| (i) | **Mirae Asset Securities (Singapore) Pte. Ltd.** — the subject of this guide | not the parent, not the group, not the asset manager | Singapore UEN **201106132Z** ✅ |
| (ii) | **Mirae Asset Securities Co., Ltd.** — the Korean listed securities firm | not the Singapore entity | KRX: **006800** ⚠ (Wikipedia/IR-sourced) |
| (iii) | **Mirae Asset Financial Group** — the Korean parent group | not a regulated Singapore entity | private group ⚠ |
| (iv) | **Mirae Asset Global Investments** — the asset-management arm | not the securities firm, not the Singapore entity | Korea-based asset manager ⚠ |

There is also a fifth, easy-to-miss collision: **Global X Investments Pte Ltd**, a separate Singapore-incorporated group affiliate at 61 Robinson Road, Singapore 068893, which markets ETFs to institutional investors in Southeast Asia ✅. It is in the group, it is in Singapore, and it is **not** the entity this guide is about. Section 3 keeps the four apart; section 13 turns the confusion into guardrails.

### 1.3 What this guide covers

- **§2 The Entity** — registered name, UEN, incorporation date, registered office, licence type, regulated activities, the documented relationship to the Korean parent, and the discovery that *two* Singapore registrations bear this exact name.
- **§3–§4 The Group and the History** — the parent group, its founding, and a date-by-date timeline with the name-by-period mapping table.
- **§5–§7 The Business and the Footprint** — what a securities firm does, which of those lines the Singapore entity is licensed for, what the group's global network looks like, and what a Singapore subsidiary of a foreign securities firm is generically *for*.
- **§8–§10 The Regulatory Position, the Technology, the Bank Interface** — the Singapore regime, the generic technology stack of a firm of this type, and the mechanism-level interfaces between such a firm and a bank.
- **§11–§13 The Honest Assessment, a Worked Example, the Anti-Patterns** — what an outsider can and cannot know, an entirely fictional onboarding scenario, and the recurring failure modes.
- **§14–§16 The Claims Audit, the Gap List, the Glossary** — every claim with its source, date and quality; the explicit list of what could not be established; the vocabulary and cross-references.

### 1.4 The decoder

The vocabulary below is used throughout. Where a term has a Singapore-regulatory meaning, that meaning is the one intended.

- **Broker-dealer** — an intermediary that both executes client orders as agent (*broking*) and trades for its own account (*dealing*). In Singapore the regulatory hook is "dealing in capital markets products".
- **A securities firm's business lines** — typically: cash equities and fixed income broking; derivatives; investment banking (advisory, equity capital markets, debt capital markets); securities financing (margin, repo, securities lending); market-making and liquidity provision; wealth management; principal investment. Not every firm has every line, and no firm's licence is a list of its revenue.
- **The cash/derivatives split** — *cash* products are the underlying instruments themselves (shares, bonds); *derivatives* are contracts whose value derives from them (futures, options, swaps, CFDs). The two carry different clearing, margin, and reporting consequences, and in Singapore they can sit under different regulatory hooks.
- **Cash equities and institutional broking** — execution and coverage for institutional clients (asset managers, insurers, sovereign funds, banks, hedge funds), as opposed to retail brokerage. The unit of value is the commission and the quality of the flow, research and execution.
- **Market-making** — quoting continuous two-way prices in a product to provide liquidity; in a Singapore context the obligation-facing framework is described in the [Market Making Singapore Guide](market_making_singapore_guide.md).
- **The group-versus-entity distinction** — the group's AUM, capital, employees and rankings are the *group's*; a subsidiary's licence, capital and conduct are the *subsidiary's*. A subsidiary is a separate legal person; it is not a division.
- **The regulator and the licence** — in Singapore, the Monetary Authority of Singapore (MAS) both supervises and lists; a firm conducting regulated capital-markets activity holds a **Capital Markets Services (CMS) licence** under the Securities and Futures Act (SFA). The regime is covered by name in the [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md) rather than re-derived here.
- **Exempt Financial Adviser** — a status, not a licence: an entity that would otherwise need a financial-adviser licence is exempted for specified activities, and is recorded as such in the MAS register.
- **Parent-and-subsidiary structure** — the parent holds shares in the subsidiary; it does not hold the subsidiary's licence. Ownership percentage is a register fact, not an inference from branding.

### 1.5 The boundary of this guide (declared, not implied)

This guide sits between three neighbours and does not restate them.

- Against the **peers** [TD Securities](td_securities_software_systems_guide.md) and [Morgan Stanley](morgan_stanley_software_systems_guide.md): both are *software-systems* deep-dives of large North American bank-owned dealers, and both are able to name platforms because those platforms are publicly marketed. This guide is an *entity and identity* deep-dive of a subsidiary whose platform choices are not publicly documented, and §9 says so plainly instead of importing the parent's technology story.
- Against the **Singapore-banking guides** [DBS](dbs_bank_guide.md), [Standard Chartered](standard_chartered_guide.md) and [Banks in Singapore](banks_in_singapore_guide.md): those cover banks — deposit-taking, licensed under the Banking Act, with a balance-sheet and payments franchise. This entity takes no deposits, and its licence is a capital-markets licence. Where a bank appears in this guide, it appears as a *counterparty type* (§10), never as this firm's client.
- Against the **regulation guides** [MAS Regulations & Guidelines](mas_regulations_guidelines_guide.md) and [Capital Markets Architecture](capital_markets_architecture_guide.md): the SFA/CMS regime, the conduct rules and the market-infrastructure map are theirs. This guide cross-references them and records only what the register says about *this* entity.

### 1.6 What was verifiable, in one paragraph

The entity is real, live, and current on the regulator's directory as at September 2026: a Capital Markets Services licensee dealing in securities and collective investment schemes and advising on corporate finance, also an exempt financial adviser, with a named chief executive officer and a Battery Road address ✅. Its Singapore incorporation in March 2011, its former name Daewoo Securities (Singapore), its SGD 50.41m paid-up capital and its Korean-equity brokerage activity description come from an ACRA-derived aggregator, not from a paid ACRA BizFile+ search ⚠ ✅. Its ownership, its shareholder percentages, its headcount, its financial statements, its platform stack and the date of its own renaming are **not** established by anything consulted. The names of the group and the parent are established; the *shareholding chain between them and this entity* is not. Sections 2 and 15 carry those gaps explicitly.

---

## 2. The Entity

This is the guide's most important section. Everything it states carries a source; everything it cannot state is named as a gap.

### 2.1 The regulator's register entry (primary source)

The MAS **Financial Institutions Directory**, the regulator's own public register, lists the entity at entry 2448, at `eservices.mas.gov.sg/fid/institution/detail/2448-MIRAE-ASSET-SECURITIES-SINGAPORE-PTE-LTD`, with the page footer showing "Last Updated: 11 Sep 2026" (retrieved in this research pass, September 2026). The entry records ✅:

| Field | Value on the MAS register |
|---|---|
| Name | **MIRAE ASSET SECURITIES (SINGAPORE) PTE. LTD.** |
| Status line | "Incorporated in Singapore" |
| Licence type/status | **Capital Markets Services Licensee** |
| Regulated activity 1 | **Dealing In Capital Markets Products** — *Securities*; *Collective Investment Schemes* |
| Regulated activity 2 | **Advising on Corporate Finance** |
| Additional status | **Exempt Financial Adviser** |
| Exempt activity 1 | **Advising on Investment Products** — *Collective Investment Schemes*; *Securities* |
| Exempt activity 2 | **Issuing or Promulgating Analyses/Reports on Investment Products** — *Securities* |
| Key personnel | **CHOI HYUNSUK**, Chief Executive Officer |
| Contact | **+65 66719846** |
| Address | **6 BATTERY ROAD #29-04 049909** |

Three things about that table matter more than the rest.

1. **The register does not record a capital figure, an incorporation date, or an ownership chain.** MAS publishes licence, activity, key personnel and contact details for a CMS licensee; the company-registry facts come from ACRA. Any guide that quotes a MAS "capital" figure for this entity is quoting something MAS did not publish.
2. **"Incorporated in Singapore" is a real distinction, not boilerplate.** MAS marks locally incorporated entities separately from foreign-entity branches on the same directory. This entity is the *locally incorporated* kind — a subsidiary, not a branch ✅.
3. **The licence is a list of permissions, not a business plan.** "Dealing in capital markets products — securities" says what may be done; it does not say how much of it is done, or with whom.

### 2.2 The company-registry record (aggregator, ACRA-derived)

The company-record facts below come from an **ACRA-derived aggregator** page at `companies.sg/business/201106132Z/...` (retrieved September 2026). It is a secondary rendering of registry data, not the registry itself. **A paid ACRA BizFile+ search / business profile was NOT performed in this research pass**, and only ACRA BizFile+ is authoritative ⚠ ✅.

| Field | Value in the aggregator record | Confidence |
|---|---|---|
| Entity name | MIRAE ASSET SECURITIES (SINGAPORE) PTE. LTD. | ✅ (matches MAS) |
| UEN | **201106132Z** | ✅ |
| Incorporation date | **14 March 2011** | ✅ (aggregator; registry-source, not independently re-pulled) |
| Company type | Private Company Limited by Shares | ✅ (aggregator) |
| Status | **Live Company** | ✅ (aggregator; consistent with MAS listing) |
| Former entity name | **DAEWOO SECURITIES (SINGAPORE)** | ✅ (aggregator) |
| Paid-up capital | **SGD 50,410,000** | ✅ (aggregator) |
| Registered office | **6 Battery Road, #37-01A, Six Battery Road, Singapore 049909** | ✅ (aggregator) |
| Principal SSIC activity | *Brokerage of Korean equity for Singapore institutional investors* | ✅ (aggregator) |
| Secondary SSIC activity | *Brokerage of Korean equity for Singapore institutional investors* (identical text) | ✅ (aggregator) |

Two observations on the aggregator's data:

- **The registered office and the MAS contact address differ by floor.** The register of companies shows the registered office at **#37-01A**; the MAS directory and the entity's own website show **#29-04** ✅ ✅. Both are 6 Battery Road. A registered office is a legal address; the operating floor may differ, and in Singapore it frequently does. Do not treat the mismatch as an inconsistency — treat it as two different questions.
- **The aggregator's own "Trustworthiness Analysis" and "Risk Level: Medium" are the aggregator's editorial opinion**, not a regulatory rating, and should not be cited as either ✅ ⚠. The aggregator itself flags two gaps: it cannot explain the name change, and it reads the CBD address as not conveying scale. Those are legitimate observations; they are not regulatory findings.

### 2.3 The identification trap: two Singapore registrations bear this exact name

The same aggregator page carries a "Companies with similar names" list. Buried in it — between `MIRAE MARINE` and `MIRAE PTE. LTD.` — is a second entity with the identical registered name ✅:

| | The live entity | The dissolved entity |
|---|---|---|
| Name | MIRAE ASSET SECURITIES (SINGAPORE) PTE. LTD. | MIRAE ASSET SECURITIES (SINGAPORE) PTE. LTD. |
| UEN | **201106132Z** | **201013438N** |
| Registered address shown | 6 Battery Road | **6 Shenton Way** (OUE Downtown area) |
| Status | **Live Company** | **Dissolved — Members Voluntary Winding Up** ✅ |
| Activity shown | Brokerage of Korean equity for Singapore institutional investors | Management consultancy services ⚠ |
| Incorporated | 14 March 2011 | 25 June 2010 |

This is a genuine, register-level identification trap and it has an inversion that is worth pausing on: **the older registration (2010) bears the name the group uses today and is dead; the younger registration (2011) carries the Daewoo heritage and is alive.** A counterparty screening system that matches on name alone — or that dedupes by "most recent registration wins" — can select the wrong legal person, or flag a live firm as dissolved.

Two honest cautions about this finding:

- **The aggregator's "similar names" list may display the *current* name for every entry rather than the name at dissolution.** If so, `201013438N` may have been renamed at some point and the point-in-time naming cannot be read off the list. What *is* clear from the aggregator is that the two records are distinct legal persons with distinct UENs, distinct addresses, distinct activity descriptions and different statuses ✅.
- **Only ACRA BizFile+ and the MAS register are authoritative.** This guide therefore states the *existence* of the two records as a finding and does **not** assert the purpose, ownership, shareholder composition, or wind-up rationale of `201013438N` ❌⚠.

### 2.4 The entity's own account of itself

The entity operates a Singapore website at `securities.miraeasset.com.sg` (retrieved September 2026), linked from the parent's official "Family Websites of Mirae Asset Financial Group" list ✅. It describes itself in its own words:

> "**Mirae Asset Securities (Singapore) Pte. Ltd.** is into diversified & well-integrated financial services, offering world class sales & trading services to a diverse range of institutional investors through top quality services including Investment Banking, Advisory Services & Institutional Business Services." ✅ (entity's own website, retrieved Sep 2026)

The site's "Location" page gives the operating contact details ✅:

- **Address:** 6 Battery Road, #29-04, Singapore 049909 *(matches the MAS address)*
- **Phone:** +65 6671 9845 *(note: the MAS register shows +65 66719846 — two slightly different numbers, each in a source dated to this pass; treat the MAS entry as the regulator-recorded contact and the website as the operating contact)*
- **Email:** adminsg@miraeasset.com.sg

**A content-quality observation, not a criticism of the firm's business:** the site's "Institutional Business" blurb states the firm is "a preferred brokerage for domestic and global institutional investors **in India**, offering fundamental analysis of various sectors and companies" — on the *Singapore* site ✅⚠. That text appears to be inherited from a sibling entity's website copy (the group also operates an Indian brokerage). It is evidence that the site is partly template-driven, and it is a concrete reason to prefer the register over the marketing site for identity questions.

### 2.5 The documented relationship to the Korean parent — and the undocumented part

What is **documented**:

- The parent's investor-relations site lists the Singapore operation in its global network, describing **Singapore** as "Engaging in IB, Global FI Sales and Cross-border deals" ✅ (english.securities.miraeasset.com, retrieved Sep 2026). This is the parent's own characterisation of what its Singapore securities business does.
- The same parent site lists `securities.miraeasset.com.sg` under **Singapore → Mirae Asset Securities** in its family-websites directory ✅, which is a public, first-party statement of affiliation.
- The entity's own site presents group-level financials under the label "(Mirae Asset Group Financial, Estimated as of December 2025)" ✅ — group figures, on the subsidiary's page, explicitly labelled as group figures.
- The entity's own "Who We Are" page states that Singapore's "Subsidiary Company Opened" date is **2012** ✅⚠ — i.e. the group's own timeline dates the Singapore *subsidiary* to 2012, while the Singapore register dates the *incorporation* to March 2011.

What is **NOT documented** in any source consulted:

- **The identity of the shareholder(s) of UEN 201106132Z.** No source consulted names the holding company, and no source states a shareholding percentage. Any statement of the form "X% owned by Y" in this guide would be fabricated. **The ownership table of this entity is UNVERIFIED in this pass** ❌.
- **The intermediate holding structure** between the Korean listed securities firm and the Singapore company (e.g. whether shares are held directly or through a regional holding company). **UNDOCUMENTED.**
- **Whether the Singapore entity is a subsidiary of Mirae Asset Securities Co., Ltd. (the listed Korean firm), or of another group company.** The branding and the parent's website imply the securities-firm line; the corporate-register chain is not public in the sources consulted. Mark ⚠ and do not assert.
- **Any regulatory licence, capital figure or ranking of the Korean parent as if it were the Singapore entity's.** The Korean firm's licences are Korean; this entity's licence is Singaporean. Where §5 or §6 quotes parent-level figures, they are labelled as the parent's.

### 2.6 What could NOT be established about this entity from public sources — bluntly

Stated once, here, and again in §15:

1. Its **ownership**: shareholder identity and percentages — unknown.
2. Its **financial statements**: the paid-up capital (SGD 50.41m) is a registry fact; revenue, profit, assets and headcount are not public.
3. The **date of its own renaming** from Daewoo Securities (Singapore) to Mirae Asset Securities (Singapore) — the register-derived source shows the former name but not the change date.
4. Whether it is a **direct or indirect** subsidiary of the Korean listed firm.
5. Its **clients, counterparties, trading venues, clearing members and vendors** — none asserted anywhere in this guide.
6. Its **technology stack** — not publicly documented (§9).
7. Whether the **2010 registration `201013438N`** was ever related to the live entity in ownership or personnel terms ❌.

---

## 3. The Group It Belongs To

### 3.1 The distinction, restated because it is the whole job

**Mirae Asset Securities (Singapore) Pte. Ltd. (UEN 201106132Z) is a Singapore-incorporated company.** It belongs to a Korean financial group. It is *not* that group, it is *not* the Korean listed securities firm, and it is *not* the asset-management arm. Every figure in this section is a **group or parent figure**, and is labelled as such. None of it is this entity's capital, licence, headcount or revenue.

### 3.2 The parent group, and the founder

| Claim | Value | Source, date | Quality |
|---|---|---|---|
| Group name | **Mirae Asset Financial Group** (미래에셋금융그룹) | Wikipedia "Mirae Asset Financial Group", retrieved Sep 2026 | ✅ secondary |
| Founded | **1997** | Wikipedia; Mirae Asset US "Our Story" timeline ("1997 — Mirae Asset Venture Capital Founded; Mirae Asset Investment Advisory Founded (current Mirae Asset Global Investments)") | ✅ two independent-ish sources agree |
| Founder | **Park Hyeon-joo** (also rendered **Hyeon Joo Park**) | Wikipedia; Mirae Asset US timeline references "the Park Hyeon Joo Fund #1" | ✅ |
| First retail mutual fund | **December 1998** — "Park Hyeon Joo No.1", described as Korea's first retail mutual fund; the US site calls it "Korea's first close-ended mutual fund" | Wikipedia; Mirae Asset US timeline | ✅ (labels differ slightly between sources) |
| Headline group scale | Group client assets **>US$550bn (Dec 2020)**; **US$729.5bn (2025)**; **$845bn+ and 67 offices, 16,000+ employees (source dated March 2026)**; **$800.3bn total client assets "estimated as of December 2025"** on the Singapore entity's own page | Wikipedia; [globalxetfs.com/about](https://www.globalxetfs.com/about) (retrieved Sep 2026, citing Mirae Asset March 2026); securities.miraeasset.com.sg | ✅ multiple dated, but **mutually inconsistent in magnitude and date — do not average them** ⚠ |

**The AUM figures are a cautionary example, not a dataset.** Within this single research pass, group client assets were quoted as US$550bn (Dec 2020), US$729.5bn (2025), US$800.3bn (Dec 2025 estimate) and US$845bn+ (Mar 2026). The Global X Canada press release of 1 May 2024 manages both **"$710 billion"** and **"$800 billion"** in the same document ✅⚠. Different bases (client assets vs AUM), different dates and different perimeters explain some of the spread; none of it would be defensible as "the group's AUM" without a date and a definition. **Where this guide quotes a group figure it quotes it with its source and its date, and never as a bare number.**

### 3.3 The group's principal businesses

From the parent's own investor site and the group's stated affiliate structure ✅⚠:

- **Asset management** — **Mirae Asset Global Investments**, described as Korea's asset-management business and the entity under which the ETF platforms sit. Wikipedia states Global X ETFs and Global X Investments Canada are its subsidiaries ✅.
- **Securities** — **Mirae Asset Securities** (the Korean listed firm): "brokerage services, wealth management, investment banking, and sales and trading services", listed on KOSPI ✅.
- **Life insurance** — **Mirae Asset Life Insurance**, KOSPI-listed since July 2015 ✅.
- **Venture investment** — **Mirae Asset Venture Investment**, KOSDAQ-listed since March 2019 ✅.
- **Capital and other affiliates** — the parent's family-website directory lists, among others, Mirae Asset Capital, Mirae Asset Financial Service, Mirae Asset Consulting, Energy Infra Asset Management, and the Mirae Asset Park Hyeon Joo Foundation ✅.
- **The securities firm's own five stated business pillars** — Global Business, Investment Banking, Trading, Wealth Management, Principal Investment ✅ (parent's IR site, retrieved Sep 2026).

### 3.4 Documented fact vs the group's own account

The house discipline for this section:

| Statement type | Example | How this guide treats it |
|---|---|---|
| **Registry/regulatory fact** | UEN 201106132Z is a live CMS licensee named Mirae Asset Securities (Singapore) Pte. Ltd. | ✅ asserted, primary source, dated |
| **First-party factual claim** | "Mirae Asset Group Financial, Estimated as of December 2025: Total Client Assets 800.3 Billion USD" | ✅ as *the group's own stated figure*, dated, not re-derived |
| **First-party narrative** | "Mirae Asset Securities has built the largest global network in Korea, in 11 countries" | ⚠ as the group's own account — marketing-flavoured, not audited |
| **Secondary press** | "South Korea's largest brokerage firm" (Yonhap, 4 Nov 2016); "largest investment banking and stock brokerage company by market capitalisation in South Korea" (Wikipedia, undated) | ⚠ dated claims by others, quoted as such |
| **Unsourced claim inside a secondary source** | Wikipedia's "founded in 1999 — *citation needed*" and its "2011.11 Acquired Betashares" entry | ❌⚠ flagged, not carried forward |

**A specific correction worth recording:** Wikipedia's "Mirae Asset Financial Group" M&A list includes "**2011.11 Acquired Betashares, a major ETF player in Australia**". No source consulted in this pass supports Mirae owning BetaShares, and the group's own Australia timeline describes acquisitions of **Global X Australia** (2022, from ETF Securities Australia) and a stake in **Stockspot**, not BetaShares. Treat that Wikipedia line as **❌ unverified**, likely a confusion with the 2011 Horizons ETFs (Canada) acquisition, which *is* independently documented ✅.

### 3.5 Where the Singapore entity sits, in one honest paragraph

The Singapore company is presented by its group as part of **the securities-firm line** — the parent's IR directory links it under "Singapore → Mirae Asset Securities", and the group's own expansion timeline marks Singapore as a "Subsidiary Company Opened" in 2012 ✅⚠. Its registered former name (Daewoo Securities (Singapore)) and its registry activity description (brokerage of Korean equity for Singapore institutional investors) point the same way ✅. What no consulted source provides is the **legal chain** — which group company holds the shares, or in what proportion. The structural expectation (a Singapore subsidiary of a Korean securities group operating a regional institutional franchise) is coherent with all the evidence and is **UNDOCUMENTED at the level of the ownership register**. Say that out loud rather than inferring a percentage.

---

## 4. The Corporate History

Every date below carries its source. Where sources disagree, both are shown. The history is presented for **two different things**, kept visually separate: the history of the **securities business** (Korean, group-level) and the history of **this Singapore entity**. Do not merge them — the Singapore company is fifteen years old, not fifty-six.

### 4.1 The Korean securities business, date by date

| Date | Event | Source | Quality |
|---|---|---|---|
| **1970** | **Daewoo Securities** founded within the Daewoo conglomerate | Wikipedia "Mirae Asset Securities" | ✅ secondary |
| **1991** | Group network: **London** subsidiary opened | securities.miraeasset.com.sg, "Who We Are" expansion timeline | ✅ first-party |
| **1992** | Group network: **New York** subsidiary opened | same | ✅ first-party |
| **1994** | Group network: **Hong Kong SAR** subsidiary opened | same | ✅ first-party |
| **1997** | **Mirae Asset Financial Group** founded by Park Hyeon-joo; Mirae Asset Venture Capital and Mirae Asset Investment Advisory (now Mirae Asset Global Investments) founded | Wikipedia; Mirae Asset US "Our Story" | ✅ |
| **1998** | **Mirae Asset Global Investments** founded (Wikipedia states "a year before Mirae Asset Securities"); Korea's first close-ended retail mutual fund launched in December | Wikipedia; Mirae Asset US timeline | ✅ |
| **1999** | **Mirae Asset Securities founded** (Wikipedia attaches *citation needed*); **Daewoo Securities spun off** from the Daewoo conglomerate in the same year | Wikipedia | ⚠ the 1999 founding of Mirae Asset Securities is asserted in Wikipedia *and* by the group's own US timeline ("1999 — Mirae Asset Securities Founded") — two sources, one of them first-party, so treat as ✅ for the group's claim, but note Wikipedia's own flag |
| **2006** | Mirae Asset Securities **lists on KOSPI** (006800.KS) | Mirae Asset US timeline | ✅ first-party |
| **2007** | Group establishes **Mirae Asset Securities (Hong Kong)** and **(Vietnam)**; Beijing representative office opens | Mirae Asset US timeline; group expansion timeline (Vietnam 2007, Beijing 2007) | ✅ |
| **2008** | **Mirae Asset Securities (USA)** established | Mirae Asset US timeline | ✅ |
| **2010** | Mirae Asset Securities (Brazil) established; Shanghai representative office opened | Mirae Asset US timeline; group expansion timeline | ✅ |
| **2010** | **Korea Development Bank** takes control of Daewoo Securities (the firm becomes KDB Daewoo Securities) | Not verified in this pass; the "KDB Daewoo Securities" name is documented from 2016 sources | ⚠ **treat the 2010 date as unverified** — the *name* KDB Daewoo Securities is documented; the timing of the KDB takeover is not established by any source consulted |
| **Jan 2016** | Mirae Asset Securities **wins the bid** for a **43% stake in Daewoo Securities** — reported at **₩2.39tn (~US$2bn)** | Bloomberg, 25 Jan 2016 (via Wikipedia citation) | ✅ dated secondary |
| **April 2016** | **Acquisition completed**: 43% of **KDB Daewoo Securities** bought from the state-run Korea Development Bank for **₩2.32tn (US$2bn)**; the group **renames the company Mirae Asset Daewoo** later that month | Yonhap, 4 Nov 2016 | ✅ primary-quality news agency |
| **4 Nov 2016** | Extraordinary shareholders' meetings of both Mirae Asset Securities and Mirae Asset Daewoo **approve the merger** of Mirae Asset Securities *by* Mirae Asset Daewoo; Yonhap reports the combined firm would hold **₩6.7tn in capital** and register as an integrated entity the day after completion, listing on KOSPI on **20 January** | Yonhap, 4 Nov 2016 | ✅ (note: Yonhap's article calls them "the two asset management firms" — a loose description of two *securities* firms; flagged as source imprecision ⚠) |
| **29 Dec 2016** | **Merger completed** (Wikipedia: "merged with Daewoo Securities as the end of December 2016") | Yonhap; Wikipedia | ✅ |
| **24 March 2021** | Shareholders' meeting **approves the renaming of Mirae Asset Daewoo to Mirae Asset Securities**; the company had used the Mirae Asset Daewoo name since the 2016 merger | Business Korea, 25 March 2021 (article dated 2021.03.25, reporting the 24 March meeting) | ✅ |
| **7 July 2018 / July 2018** | Mirae Asset Global Investments **acquires Global X** (US ETF provider) for **US$488m** | Wikipedia "Global X ETFs" (sources dated 2018); Mirae Asset US timeline ("2018 — Acquires Global X") | ✅ |
| **2022** | Mirae Asset Global Investments and Global X **acquire ETF Securities Australia** (now **Global X Australia**); Mirae Asset Securities **acquires GHCO**, a British ETF market maker and liquidity provider | Mirae Asset US timeline | ✅ first-party |
| **2023** | Mirae Asset Securities acquires Indian brokerage **Sharekhan** (Share Purchase Agreement) | Mirae Asset US timeline | ✅ first-party |
| **2024 (Nov)** | Wikipedia's M&A list dates the **Sharekhan acquisition completion to 2024.11**, citing a Sharekhan blog post; the group's own timeline describes the 2023 SPA | Wikipedia; Mirae Asset US timeline | ⚠ **date discrepancy — SPA 2023, completion 2024; both statements can be true; flagged rather than resolved** |
| **2025** | Group reports reaching **KRW 1,000 trillion in client assets**; **Mirae Asset Securities obtained IMA business authorization** | Mirae Asset US timeline | ⚠ first-party, unaudited |
| **2026** | Mirae Asset Securities **U.S. joined a U.S. tokenization working group**; **Mirae Asset Securities' Hong Kong SAR received final approval from the SFC to offer digital asset trading services to retail investors** | Mirae Asset US timeline (2026 entries) | ✅ first-party, dated within the group's own timeline |

### 4.2 This Singapore entity, date by date

| Date | Event | Source | Quality |
|---|---|---|---|
| **25 June 2010** | A separate Singapore company, **UEN 201013438N**, bearing the same name as today's live entity, is incorporated at 6 Shenton Way | ACRA-derived aggregator, retrieved Sep 2026 | ✅ (register-level; see §2.3) |
| **14 March 2011** | **UEN 201106132Z incorporated** in Singapore, as a Private Company Limited by Shares, under the name **DAEWOO SECURITIES (SINGAPORE)** | ACRA-derived aggregator | ✅ |
| **2012** | The group's own expansion timeline records **Singapore as a "Subsidiary Company Opened"** | securities.miraeasset.com.sg "Who We Are" | ✅ first-party ⚠ (one-year gap against the March 2011 incorporation — incorporation and operational opening are different events; the date of the *name change* is still unknown) |
| **April 2016 → 24 March 2021** | The parent goes Daewoo → KDB Daewoo → Mirae Asset Daewoo → Mirae Asset Securities. The Singapore entity's **own** renaming from Daewoo Securities (Singapore) to Mirae Asset Securities (Singapore) falls somewhere in this window | Register shows the former name; **the change date is not shown by any source consulted** | ⚠ **DATE NOT ESTABLISHED — flagged explicitly** |
| **Dissolved — date not shown** | UEN **201013438N** recorded as "Dissolved — Members Voluntary Winding Up" | ACRA-derived aggregator, retrieved Sep 2026 | ✅ status; ⚠ wind-up date not shown |
| **11 Sep 2026** | MAS Financial Institutions Directory page for the live entity last updated; shows CMS licence, exempt FA status, CEO CHOI HYUNSUK | MAS FID entry 2448 | ✅ primary |
| **Sep 2026** | Entity operates `securities.miraeasset.com.sg`, presents itself as offering sales & trading, investment banking, advisory and institutional business to institutional investors | Entity's own website, retrieved Sep 2026 | ✅ first-party |

### 4.3 The name-by-period mapping (the table to keep)

| Period | The Korean securities business is called | The Singapore company is called | Note |
|---|---|---|---|
| 1970 – 1999 | **Daewoo Securities** (within the Daewoo conglomerate) | — (not yet incorporated) | spun off 1999 |
| 1999 – 2016 | **Daewoo Securities**, then **KDB Daewoo Securities** after KDB takes control | **Daewoo Securities (Singapore)** from 14 Mar 2011 | the Singapore entity is born into the Daewoo-name period |
| April 2016 – 24 Mar 2021 | **Mirae Asset Daewoo** (renamed April 2016 after the 43% acquisition; the listed securities business merged into it Dec 2016) | **Mirae Asset Securities (Singapore)** today *(the rename date is not public)* | the Singapore entity's former-name field records only the Daewoo name |
| 24 Mar 2021 – present | **Mirae Asset Securities Co., Ltd.** (KRX 006800) | **Mirae Asset Securities (Singapore) Pte. Ltd.** (UEN 201106132Z) | both names now share the "Mirae Asset Securities" string — the single largest source of confusion in this guide |

**Read the table this way:** the *same string* "Mirae Asset Securities" today denotes (a) a KOSPI-listed Korean securities firm and (b) a Singapore-incorporated CMS licensee. They are different legal persons with different regulators, different capital and different liabilities. Secondary sources use "Daewoo", "Mirae Asset Daewoo", "Mirae Asset Securities" and "Mirae Asset" interchangeably, sometimes within one article ⚠.

### 4.4 The history trap, stated plainly

A reader who finds a 2016 article about "Mirae Asset Securities acquiring a 43% stake in Daewoo Securities" is reading about **the Korean parent's acquisition**, not about the Singapore entity (which had already been incorporated for five years and was *inside* the acquired Daewoo group at the time). Conversely, a reader who finds that the Singapore entity was "formerly Daewoo Securities (Singapore)" is reading the *consolidated* consequence of that same acquisition, five years later. **The Singapore entity was not acquired by itself; it was, on the evidence of its name and its registry description, part of the acquired perimeter.** The precise mechanics — which corporate step renamed it, and when — remain UNDOCUMENTED.

---

## 5. The Securities Business, and Where This Firm Sits In It

### 5.1 What a securities firm does

Stripped of branding, the business lines of a securities firm are:

- **Broking / execution** — receiving client orders and executing them in the market, as agent. Institutional broking (coverage of asset managers, insurers, sovereign funds, banks) and retail broking are different businesses with different economics, different licensing consequences and different technology.
- **Dealing** — trading as principal against the client or in the market. This is where "market-making" lives: quoting continuous two-way prices to provide liquidity, typically in ETFs, listed derivatives, or cash equities.
- **Underwriting and advisory** — investment banking: equity capital markets (IPOs, placements), debt capital markets (bond issuance), mergers and acquisitions advisory, corporate finance. Feeds on a licence to *advise on corporate finance*.
- **Financing and securities lending** — margin financing to clients, securities lending and borrowing, repurchase agreements. Balance-sheet-intensive and risk-intensive; the source of much of the difference between a securities firm and a pure agency broker.
- **Wealth management** — product distribution, discretionary and advisory portfolios, typically to individuals and family offices.
- **Principal investment** — the firm invests its own capital in assets (real estate, infrastructure, private equity) rather than merely intermediating.
- **Research** — published analysis. In Singapore, *issuing or promulgating analyses/reports on investment products* is itself a regulated (or here, exempt) activity, which is why it appears on this entity's MAS entry.

### 5.2 The Korean parent's documented position — labelled as the source's own claim

These figures and rankings belong to **Mirae Asset Securities Co., Ltd. (KRX 006800)**, the Korean listed firm. They are quoted here because they are the context a reader will look for; they are **not** this Singapore entity's.

| Claim | Attribution | Quality |
|---|---|---|
| "South Korea's largest brokerage firm" (in the context of the 2016 merger creating it) | **Yonhap, 4 November 2016** — a news agency's characterisation at that date | ⚠ dated claim by a third party |
| "the largest investment banking and stock brokerage company **by market capitalisation** in South Korea" | Wikipedia "Mirae Asset Securities", undated in the article text | ⚠ single-source, undated — do not restate as current |
| Combined capital **₩6.7 trillion** expected from the December 2016 merger | Yonhap, 4 Nov 2016 | ⚠ forward-looking figure from 2016, not a current capital position |
| Five business pillars: Global Business, Investment Banking, Trading, Wealth Management, Principal Investment | Parent's IR site, retrieved Sep 2026 | ✅ first-party description |
| "Provides liquidity for ETFs to domestic and international investors, securities lending, credit offering, and trades in over-the-counter derivatives for risk hedging" | Parent's IR site, Trading description, retrieved Sep 2026 | ✅ first-party description of the *parent's* trading business |
| Mirae Asset Securities Hong Kong acted as joint book-running co-manager for **BioNTech**'s IPO — reported as the first South Korean securities company to be a NASDAQ IPO underwriter | Korea Herald, 4 Nov 2019 (via Wikipedia citation) | ✅ dated secondary — and note it attributes the act to the **Hong Kong** affiliate, not to Singapore |
| Nine consecutive years on the **Dow Jones Sustainability Index World** as of November 2020 | Wikipedia, citing Korea Herald/Korea Times 15 Nov 2020 | ⚠ dated, and a **parent-level** ESG listing, not this entity's |

### 5.3 Which of these lines the Singapore entity is licensed for

This is the part that is not inference. From the MAS register ✅:

| Business line | Is the Singapore entity permitted? | Evidence |
|---|---|---|
| **Dealing in capital markets products — Securities** | **Yes** | MAS FID entry 2448 |
| **Dealing in capital markets products — Collective Investment Schemes** | **Yes** | MAS FID entry 2448 |
| **Advising on Corporate Finance** | **Yes** | MAS FID entry 2448 |
| **Advising on Investment Products — CIS and Securities** | **Yes, as an Exempt Financial Adviser** | MAS FID entry 2448 |
| **Issuing or promulgating analyses/reports on investment products — Securities** | **Yes, as an Exempt Financial Adviser** (i.e. research publication) | MAS FID entry 2448 |
| **Brokerage of Korean equity for Singapore institutional investors** | Stated as the company's registered *activity description* (SSIC) — a registry description of the business, not a licence | ACRA-derived aggregator |
| **Sales & trading, investment banking, advisory and institutional business services to institutional investors** | Stated by the entity about itself | Entity's own website, retrieved Sep 2026 |
| **Deposit-taking** | **No** — nothing on the register suggests it, and a CMS licence is not a banking licence | MAS FID entry 2448 (absence) |
| **Retail fund management / asset management** | **Not evidenced** — "advising on investment products" and "dealing in CIS" are not asset management. The group's asset-management business is a **different entity** (Mirae Asset Global Investments) | MAS FID entry 2448; §3 |

**Note the deliberate blank in that table:** nothing on the MAS register says this entity makes markets, underwrites IPOs, lends securities, or takes margin balances. Those are things *securities firms* do; the register says what *this* one may do. §15 records the distinction as a gap.

### 5.4 The one-line positioning

The register plus the firm's own description give a coherent picture of a **wholesale, institutional-facing securities subsidiary**: it may deal in securities and CIS, advise on corporate finance, advise on investment products and publish research — and it presents itself as a sales-and-trading, IB and advisory shop aimed at institutional investors, with Korean-equity brokerage as its registered activity description ✅⚠. What it is *not* on any evidence consulted is a retail broker, a bank, an asset manager, or a market maker by licence.

---

## 6. The Global Footprint

Each item below was verified individually in this pass, with a date and a source. **Nothing in this section is carried over from memory**, and where a holding has or may have changed hands, that is stated.

### 6.1 The securities firm's own network (first-party, dated 2026)

The parent's IR site lists its global network, and its Singapore page states the local role: **"Singapore — Engaging in IB, Global FI Sales and Cross-border deals"** ✅ (english.securities.miraeasset.com, retrieved Sep 2026).

The group's expansion timeline (on the Singapore entity's own "Who We Are" page) records these openings ✅:

| Market | Year shown | Type |
|---|---|---|
| London, UK | 1991 | Subsidiary |
| New York, US | 1992 | Subsidiary (US Holdings and US Investments opened 2018) |
| Hong Kong SAR | 1994 | Subsidiary |
| Beijing, China | 2007 | Representative office |
| Vietnam | 2007 | Subsidiary |
| Brazil | 2010 | Subsidiary |
| Shanghai, China | 2010 | Representative office |
| **Singapore** | **2012** | **Subsidiary** |
| Indonesia | 2013 | Subsidiary |
| Mongolia | 2013 | Subsidiary |
| India | 2017 | Subsidiary (Mirae Asset Sharekhan launched 2024) |
| London, UK (further) | 2023: "Acquired GTX"; 2024: "UK Investments Opened" | Acquisition / new entity |

The same page claims the firm "has built the largest global network in Korea, in **11 countries**" ✅ first-party ⚠. **Note the discrepancy with Wikipedia**, which says Mirae Asset Securities "engages in global brokerage, investment banking, and trading businesses in **ten** foreign countries" and separately that it operates in "ten foreign countries" — Wikipedia's count is undated and unevidenced in the article text ⚠. Treat the 11-country figure as the group's own, dated to 2026.

### 6.2 The asset-management line and the ETF platforms

These are **Mirae Asset Global Investments** entities (the asset-management arm), not securities-firm entities and not the Singapore securities entity:

| Affiliate | Verified position | Source and date |
|---|---|---|
| **Global X ETFs (US)** | Acquired by Mirae Asset Global Investments in **July 2018 for US$488m**; **New York-headquartered**; as of the site retrieved Sep 2026 it states "Global X ETFs is a member of Mirae Asset Financial Group", with a source line dated **March 2026** (16,000+ employees; $845bn+ in AUM; 67 offices) and **~$169bn of AUM in Global X ETFs listed worldwide as of June 2026**; CEO **Ryan O'Connor** (appointed effective 8 April 2024) | Wikipedia; [globalxetfs.com/about](https://www.globalxetfs.com/about); KED Global 21 Feb 2024 |
| **Global X Investments Canada Inc.** | Rebrand completed from **Horizons ETFs** effective **1 May 2024** (announced 6 March 2024); described as **"a wholly owned subsidiary of Mirae Asset Global Investments Co., Ltd."**, "> $30 billion of AUM and 108 ETFs listed on major Canadian stock exchanges"; Horizons ETFs was acquired by the group in **2011** (BusinessWire 14 Nov 2011 refers to an **85% stake**) | Global X Canada press release 1 May 2024; Mirae Asset US 18 Apr 2024; Wikipedia citations |
| **Global X Australia** | Group timeline: **2022** — Mirae Asset Global Investments and Global X **acquire ETF Securities Australia**, now branded Global X Australia | Mirae Asset US timeline |
| **Global X Brazil** | Group timeline: **2022** — Mirae Asset Global Investment Brazil renamed **Global X Brazil** | Mirae Asset US timeline |
| **Global X Japan** | **2019** — established as a JV between Global X, **Daiwa Securities** and **Daiwa Asset Management** | Mirae Asset US timeline; Wikipedia |
| **Global X Investments Pte Ltd (Singapore)** | Present and marketing to institutional investors in Southeast Asia; address **61 Robinson Road, Level 07-01 Suite 758, Singapore 068893**; contact listed as **Richard Siaw, CFA, Director, Southeast Asia**; site footer data "~$115bn of AUM in Global X ETFs listed worldwide as of August 2025" and group figures sourced **June 2025** | [globalxetfs.sg](https://www.globalxetfs.sg/), retrieved Sep 2026 |
| **GHCO** | British ETF market maker / liquidity provider; acquired by **Mirae Asset Securities** (the securities line, not the asset manager) in **2022**; still listed under Greece and the UK in the parent's family directory, retrieved Sep 2026 | Mirae Asset US timeline; parent IR directory |

### 6.3 Have any holdings changed hands? What was checked

The brief for this guide required an explicit answer to whether any group ETF holding has been **divested** since the rebrand, so that no divested holding is presented as current.

**What was checked in this pass:**

1. **Global X US** — searched for a 2024–2026 sale, divestment or partial-stake transaction in Global X Management Company LLC / Mirae Asset Global Investments. **Result: nothing found.** `web_search` returned **empty result sets** for every query attempted in this pass (see §15.3 — a tool limitation, not evidence of absence). Direct extraction of `globalxetfs.com/about` in September 2026 shows Global X **still presenting itself as a member of Mirae Asset Financial Group**, with group figures sourced to **March 2026** ✅. The most defensible statement is therefore: **no divestment was found, and Global X US continued to present itself as a group member as at the March 2026 source date; a formal ownership confirmation for September 2026 was NOT obtained.**
2. **Global X Canada** — the 1 May 2024 press release states the company "is a wholly owned subsidiary of Mirae Asset Global Investments Co., Ltd." as at that date ✅. **Current ownership not confirmed as at Sep 2026** beyond that press release and the parent directory still listing `globalx.ca` under Canada ✅; the site is live and copyright-dated **© 2026** ✅.
3. **Global X Australia / Europe** — the parent's family directory, retrieved Sep 2026, still lists Global X (Australia) and Global X (Ireland/Europe) ✅; `globalx.com.au` extracted in this pass turned out to be **Dye & Durham's legal-information business, formerly GlobalX** — an entirely unrelated Australian company that happens to share the name ⚠✅. The group's Australian ETF site is `globalxetfs.com.au`, listed in the parent directory; it was **not** independently re-extracted in this pass. **Flagged as not independently verified.**
4. **Global X ETFs Europe** — `globalxetfs.eu/about-us` returned a 404 page in this pass while the site's navigation (Our ETFs / About / News / Contact / Manco information) is live ✅⚠. **Site live; ownership statement not independently re-verified in this pass.**

**Honest conclusion for §6.3:** no source consulted records a change of hands in any group ETF holding. That is a *negative result from a limited search*, not a positive verification. Every claim of continuity in this section carries the date of the source that supports it, and §15 lists the two continuity questions that remain open.

### 6.4 What the footprint does and does not say about Singapore

The footprint establishes that the group is genuinely global, that it has run a securities subsidiary in Singapore since the early 2010s, and that a *separate* Singapore entity (Global X Investments Pte Ltd) markets group ETFs in the region ✅. It establishes **nothing** about the Singapore securities entity's own clients, revenues, product lines or headcount, and it must not be used as a proxy for them.

---

## 7. What The Singapore Entity Is For

This section separates **what such an entity generically does** from **what is documented about this one**. Read the two halves as strictly different knowledge claims.

### 7.1 The generic answer: what a foreign securities subsidiary in Singapore is *for*

Singapore's role as a regional financial centre, and its regulatory and tax architecture, make it a standard location for a foreign securities group's regional hub. In the generic case, a subsidiary of a foreign securities firm in Singapore exists to do some combination of the following:

- **Institutional sales and coverage** — a locally licensed, locally present sales team covering Southeast Asian and global institutional clients (asset managers, insurers, sovereign wealth funds, banks, family offices) for the group's research, execution and product shelf. Locally licensed sales staff can be compensated and conduct business in a way a fly-in team cannot.
- **Execution into Asian venues** — routing and executing client orders into Singapore Exchange, other ASEAN venues, and — via connections — Japan, Korea, Hong Kong, Greater China and India. The subsidiary is frequently the group's access point to the region's liquidity.
- **Client servicing and regional operations** — onboarding, settlement instruction handling, confirmations, client reporting, and the operational layer that makes cross-border flow work inside Asian business hours.
- **Group treasury and financing** — intra-group lending and borrowing, collateral management, funding vehicles, and the treasury seat that lets a group fund its regional book in a currency and legal regime it chooses.
- **Regional coverage and relationship management** — the coordination point for the group's other Asian entities, and the seat from which the group originates regional investment-banking mandates and cross-border deals.
- **Research distribution** — publishing or distributing analysis into the region under a licence that permits it (in this entity's case, as an exempt financial adviser for issuing or promulgating analyses and reports on investment products in securities).
- **Balance-sheet light, licence heavy** — the subsidiary exists partly because a *licence* is territorial. A Singapore CMS licence is what allows the group to book regulated activity in Singapore at all.

None of that is a claim about this firm. It is the class description.

### 7.2 The documented answer: what the sources say about this one

What is actually documented about Mirae Asset Securities (Singapore) Pte. Ltd.'s purpose:

| Documented statement | Source, date | Reading |
|---|---|---|
| Registered activity description: **"Brokerage of Korean equity for Singapore institutional investors"** | ACRA-derived aggregator, retrieved Sep 2026 | The single most concrete statement of purpose available: a cross-border brokerage channel between Korean equity and Singapore institutional money. This is a *registry activity description*, not a business plan ⚠ |
| **"Engaging in IB, Global FI Sales and Cross-border deals"** | Parent's IR site (Singapore card), retrieved Sep 2026 | The parent's own one-line summary of the Singapore securities operation: investment banking, global fixed-income sales, cross-border deals ✅ |
| **"Diversified & well-integrated financial services, offering world class sales & trading services to a diverse range of institutional investors … including Investment Banking, Advisory Services & Institutional Business Services"** | Entity's own website, retrieved Sep 2026 | The firm's self-description. Consistent with the register's licence set (dealing, corporate-finance advice, research), and **not** evidenced beyond marketing copy ⚠ |
| Licence permitting **dealing in securities and CIS, corporate-finance advice, product advice and research publication** | MAS FID entry 2448, last updated 11 Sep 2026 | What the entity is *permitted* to do ✅ |
| Group **equity capital US$17.2bn**, **total client assets US$800.3bn** ("Mirae Asset Group Financial, Estimated as of December 2025") shown on the Singapore site | Entity's own website | **Group** figures, not this entity's; shown on a Singapore page, which is exactly how the entity/group confusion starts ⚠ |
| Singapore listed as a **subsidiary opened 2012** in the group's network | Entity's own website | Group's own account of the entity's founding as a subsidiary ✅⚠ |

### 7.3 The gap between the two halves

The generic half says: institutional sales, execution, servicing, treasury, regional coverage, research. The documented half says: Korean-equity brokerage for Singapore institutional investors, IB, global FI sales, cross-border deals — with a licence set that matches.

**The overlap is real but the precision is not.** No source consulted states this entity's client list, its revenue split, its headcount, its trading venues, its clearing arrangements, its balance sheet, or which of the generic functions it actually performs. §7.1 tells you what to *expect* of a Singapore securities subsidiary; §7.2 tells you what is *documented* about this one; §15 records the difference as a gap. **Do not let the generic half do duty for the documented half** — that is anti-pattern 1 in §13.

---

## 8. The Regulatory Position

The regime itself is not re-derived here. The [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md) is the repository's reference for the Securities and Futures Act (SFA), the Financial Advisers Act (FAA), MAS notices and guidelines, and the conduct framework. This section records only **what the regime implies for this entity**, and **what the regulator's register says about this entity**.

### 8.1 The regime, in the minimum form needed

- **The regulator.** The Monetary Authority of Singapore is Singapore's central bank and integrated financial regulator; it supervises capital markets intermediaries and maintains the public Financial Institutions Directory used as the primary source in this guide ✅.
- **The statute.** Regulated capital-markets activity in Singapore is conducted under the **Securities and Futures Act**. A person carrying on business in a regulated activity without authorisation commits an offence; the authorisation is a **Capital Markets Services licence**.
- **The licence class.** This entity holds a **Capital Markets Services Licensee** status ✅ — the licence class for intermediaries that **deal in capital markets products**, **advise on corporate finance**, and similar regulated activities.
- **Financial advisory activity.** Advice on investment products and the issuing or promulgating of analyses/reports are financial-advisory activities under the **Financial Advisers Act**. An entity that would otherwise need a financial adviser's licence can be recorded as an **Exempt Financial Adviser** for specified activities, which is exactly what this entity's register entry shows ✅.
- **The activity taxonomy matters.** "Dealing in capital markets products" is a category with sub-categories — the register names **Securities** and **Collective Investment Schemes** for this entity. A licence scoped to securities and CIS is not a licence scoped to, say, exchange-traded derivatives, spot foreign exchange, or digital payment tokens. **Read the sub-categories, not just the headline** ✅.
- **The register is not the whole obligation set.** A CMS licensee is also subject to business-conduct requirements (minimum base capital or risk-based capital, compliance arrangements, record-keeping, client-asset segregation, reporting and notification obligations), anti-money-laundering and counter-terrorism-financing obligations, and the applicable MAS Notices. Those obligations attach to the licence class, and are covered in the [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md); they are **not** evidenced on the directory entry, which is a listing rather than a compliance statement ⚠.

### 8.2 This entity's position on the register, dated

| Item | Register value | Date of the register page |
|---|---|---|
| Name | MIRAE ASSET SECURITIES (SINGAPORE) PTE. LTD. | Page "Last Updated: 11 Sep 2026" |
| Locally incorporated? | Yes — "Incorporated in Singapore" | same |
| Licence | Capital Markets Services Licensee | same |
| Dealing in capital markets products | Securities; Collective Investment Schemes | same |
| Advising on corporate finance | Yes | same |
| Exempt financial adviser | Yes | same |
| Advising on investment products | Collective Investment Schemes; Securities | same |
| Issuing/promulgating analyses or reports on investment products | Securities | same |
| Key personnel (CEO) | CHOI HYUNSUK | same |
| Contact | +65 66719846; 6 Battery Road #29-04, Singapore 049909 | same |

**What the register does not show, and what that means:** no licence conditions are published, no enforcement history appears on the entry, no capital figure is published, and no branch/subsidiary parent is named. A reader must not read the *absence* of any of these as a positive statement. **Absence of a disclosed condition is not evidence of the absence of a condition** ⚠ ✅.

### 8.3 Obligations that attach to this licence class (structural, not entity-specific)

Presented as the shape of the obligation set, because the entity-specific detail is not public:

- **Capital and financial-resource requirements** — a CMS licensee must maintain base capital or risk-based capital above the applicable threshold; the Singapore company's registry **paid-up capital of SGD 50,410,000** is a *different* concept from regulatory capital and cannot be read as satisfying (or failing) any regulatory test ✅⚠.
- **Conduct and disclosure** — client-asset handling, order-handling and best-execution expectations, conflicts management, disclosure of remuneration and interests, and marketing restrictions. This is the layer that determines whether "institutional-only" activity can be conducted at all without retail-facing protections.
- **Suitability and the institutional-client carve-out** — much of the retail-facing conduct machinery has a lighter counterpart for **accredited investors** and **institutional investors**; the entity's stated institutional focus and its registered activity description both point at that wholesale posture ⚠.
- **Research and advisory conduct** — publishing analysis and advising on investment products imports independence, disclosure and record-keeping expectations that pure dealing does not.
- **AML/CFT** — enhanced due diligence, screening, transaction monitoring and reporting; for a cross-border brokerage with Korean-market exposure, this is a substantive control domain rather than a paper exercise.
- **Books, records and regulatory reporting** — a licensed intermediary must be able to produce its books, and must report to MAS on stated cycles and on stated events (including changes in key personnel and control).
- **Cross-border conduct** — dealing with clients and counterparties outside Singapore, or routing orders into foreign venues, brings the host-state rules of those venues into play alongside Singapore's.

### 8.4 The cross-border dimension: host state, home state, and subsidiary versus branch

This is the structural point that determines who supervises what, and it is worth stating precisely:

- **A branch is an extension of a foreign legal person.** Its obligations sit on the foreign entity; the host regulator's grip runs through registration and conduct requirements aimed at the foreign body.
- **A subsidiary is a separate legal person incorporated locally.** It is supervised by the **host-state regulator as a domestic institution** — MAS supervises Mirae Asset Securities (Singapore) Pte. Ltd. directly, as a Singapore-incorporated CMS licensee ✅ — while its parent is supervised by its own home-state regulator.
- **Consequences of the subsidiary form** (structural, applies to this entity by virtue of its registered form ✅):
  1. The subsidiary's capital and licence are its own; the parent's capital is not available to the subsidiary's regulator as a direct matter.
  2. The subsidiary's regulator can impose conditions, require its own capital, inspect its books, and act against it without reaching the parent.
  3. Group-level support is real in practice but is a **commercial and reputational expectation**, not a regulatory guarantee, unless supported by documented arrangements.
  4. Group-wide risk, IT, compliance and audit functions typically operate as global programmes applied locally — which is why group policy and local regulation can conflict, and why the local entity must be able to evidence local compliance in its own name ⚠.
- **The Korean side of the chain.** The listed Korean parent is regulated in Korea and listed on the KOSPI ✅. Its disclosures are the primary public window onto the group — and they are Korean-language and group-consolidated, which is the reader's problem described in §11.

### 8.5 The regulatory summary, in one paragraph

A Singapore-incorporated Capital Markets Services licensee dealing in securities and collective investment schemes, advising on corporate finance, and acting as an exempt financial adviser for product advice and research publication, supervised directly by MAS ✅. The licence class imports a substantial obligation set whose detail is not public; the register does not publish conditions, capital or enforcement history; and the entity's subsidiary form means MAS supervises it as a domestic institution rather than as a foreign branch — with the parent's regulator and the parent's capital sitting in a different jurisdiction and a different legal person. The [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md) carries the regime detail.

---

## 9. The Technology A Firm Of This Kind Runs

### 9.1 The rule for this section

**No platform, vendor or system is attributed to Mirae Asset Securities (Singapore) Pte. Ltd. anywhere in this guide.** No source consulted documents this entity's technology estate, and this research pass found none. What follows is (a) the generic technology shape of an institutional securities firm, stated as industry knowledge, and (b) the honest statement that the firm's specifics are not public. This is a deliberate contrast with the [TD Securities](td_securities_software_systems_guide.md) and [Morgan Stanley](morgan_stanley_software_systems_guide.md) guides, which *can* name platforms because those firms market them publicly.

### 9.2 Order and execution management (generic)

An institutional securities firm runs two closely related tiers:

- **Order management** — capturing client orders, validating them against limits, credit and entitlements, allocating them, and maintaining the order's state through its life. The order record is also a regulatory record.
- **Execution management** — routing to venues and brokers, managing algos, handling market data, and tracking fills. For a firm whose flow is cross-border (e.g. Singapore-based institutional clients trading Korean or Asian cash equity), the interesting parts are the **gateways to foreign brokers and venues**, the session handling across time zones, and the reconciliation of a trade executed in one market and confirmed in another.
- **Connectivity standard** — venue and counterparty connectivity in this industry conventionally uses the **FIX protocol**, covered by name in the [FIX Protocol Guide](fix_protocol_guide.md), and order/execution semantics increasingly modelled against the **FINOS Common Domain Model**, covered in the [FINOS CDM Guide](finos_cdm_guide.md). The general architecture of order handling is set out in [Financial Trading & Order Infrastructure](financial_trading_order_infrastructure.md) and the [Capital Markets Architecture Guide](capital_markets_architecture_guide.md).

### 9.3 Market-data consumption (generic, and cross-referenced)

An institutional brokerage consumes market data in at least four distinct ways, and each carries its own licensing and control consequences:

- **Real-time pre-trade data** — quotes, order books and reference prices for the instruments it quotes or executes. Licensed per user, per device, per use; redistributed only under entitlement.
- **Post-trade and reference data** — end-of-day prices, corporate actions, security masters, static data used for valuation, confirmations and client reporting.
- **Derived and index data** — index levels, analytics and benchmarks used in products and research.
- **Regulatory and reporting feeds** — data feeding best-execution and transaction-reporting obligations.

The two repository guides that govern this domain by name are the [Market Data Consumption Guide](market_data_consumption_guide.md) (how data is licensed, consumed and controlled) and the [Market Data Integrity Guide](market_data_integrity_guide.md) (how data quality and lineage are assured). Both apply to a firm of this type **as a class**; neither is asserted here as this entity's implementation.

### 9.4 Post-trade processing (generic)

Post-trade is where a securities firm's operational risk concentrates, and it is conventionally decomposed into:

- **Trade enrichment and validation** — completing missing fields, checking against standing settlement instructions, detecting breaks.
- **Confirmation and affirmation** — matching the firm's record against the counterparty's, increasingly via central matching platforms and industry affirmation services.
- **Allocation** — splitting a block execution across the underlying client accounts that gave the order.
- **Settlement instruction and status tracking** — instructing custodians or settlement agents and tracking the instruction to settlement.
- **Fails management and claims** — detecting unsettled positions, managing buy-ins and compensation claims.

The domain model for these objects is exactly what the [FINOS CDM Guide](finos_cdm_guide.md) addresses, and cross-border settlement mechanics are covered in the [Singapore Security Clearance Guide](../technology/singapore_security_clearance_guide.md).

### 9.5 Clearing and settlement connectivity (generic)

A firm of this type connects to the post-trade plumbing of each market it touches. In the Singapore/Asian context this conventionally means connectivity to the local clearing and settlement infrastructure, to a **clearing member** where it is not itself one, and to the **custodians** holding its and its clients' assets; in cross-border flow it means correspondents and global custodians for the markets where it has no direct membership. The repository's [Singapore Security Clearance Guide](../technology/singapore_security_clearance_guide.md) and [Financial Infrastructure Guide](financial_infrastructure_guide.md) cover the plumbing by name. **Nothing in this paragraph is a statement about which venues or custodians this entity uses** — that is not public.

### 9.6 Books and records, and the regulatory reporting layer (generic)

- **Books and records** — the ledger, the position and trade repository, the client and counterparty master, and the retention regime that makes all of it auditable. For a licensed intermediary this is a compliance obligation before it is a technology choice.
- **Regulatory reporting** — the layer that turns internal data into regulatory submissions: transaction reporting to the relevant regime, position and large-exposure reporting, capital and financial-resource reporting, AML-related reporting, and periodic returns to MAS. This layer is notoriously the least standardised part of a securities firm's estate, because each jurisdiction's schema is its own.
- **The group-dimension** — a foreign-owned subsidiary conventionally inherits group reporting and data standards, which must then be reconciled with local regulatory schemas. That reconciliation work is generic to the structure, not a documented fact about this firm ⚠.

### 9.7 The honest statement

**Mirae Asset Securities (Singapore) Pte. Ltd.'s specific platform choices, vendor relationships, in-house systems, cloud posture and data architecture are NOT publicly documented in any source consulted in this research pass.** No order management system, execution management system, market-data vendor, clearing gateway, books-and-records platform, reporting tool or infrastructure provider is attributed to it here. Anything a reader has seen elsewhere claiming a specific stack for this entity should be checked against a first-party disclosure. The reason this is stated rather than glossed: a securities subsidiary's technology is a *regulatory* and *operational* control surface, and inferring it from the parent's — or from a peer's — is precisely anti-pattern 4 in §13.

---

## 10. The Bank Interface

### 10.1 Why this section exists

A securities firm and a bank meet at specific, enumerable points. This section describes **the mechanisms** — the interfaces, the documents and the risk questions — at the level an architect or a counterparty-onboarding analyst needs. It does not describe this entity's actual arrangements, because none are public. And in accordance with the governing rule of this guide, **no real bank or institution is asserted anywhere in this section as a client, counterparty or partner of this firm**.

### 10.2 The interface points

**(a) Execution counterparty.** A securities firm trading as principal, or seeking liquidity in a market where it is not a member, deals with banks as market-makers and liquidity providers. The interface is: trading documentation (a master agreement governing the trading relationship), credit lines and limits, settlement instructions, and the operational plumbing behind both. In Asia, a firm's bank relationships on the execution side often double as its access to markets where it has no direct membership.

**(b) Clearing and custody client relationship — in either direction.** Two distinct relationships exist and are frequently confused:

- **The firm as a client of a bank** — using a bank as clearing broker, prime broker, custodian, or settlement agent. The bank then holds the firm's (or the firm's clients') assets, extends settlement credit, and provides the operational interface to the market's clearing house.
- **The firm as a provider to a bank's clients** — the bank's wealth-management or private-banking arm distributing products, or the bank's asset-management arm trading through the firm. Same plumbing, different direction, and entirely different conduct obligations.

Which of these applies to this entity is **not public** ⚠.

**(c) Securities lending and financing.** Repo, securities lending and borrowing, margin financing. The mechanisms: a securities lending agreement or a global master securities lending agreement, collateral schedules, eligibility haircuts, recall and substitution mechanics, and the income-distribution waterfall. A firm with client long positions can lend; a firm needing inventory borrows. For a cross-border brokerage, the interesting cases are **borrow across markets** (inventory in one market, demand in another) and the collateral-currency mismatch.

**(d) Collateral and margin.** Mechanism-level: initial and variation margin under the applicable margin regime (with different rules for cleared vs uncleared derivatives), collateral eligibility and haircut schedules, daily (or intraday) margin calls, dispute resolution on valuation, and the legal enforceability of the collateral arrangement in each relevant jurisdiction. The data-model dimension of these objects is what the [FINOS CDM Guide](finos_cdm_guide.md) and the [Collateral and Margin domain models](financial_trading_order_infrastructure.md) address by name; the operational schedule is typically governed by a collateral annex to the master agreement.

**(e) Cross-border settlement.** The mechanisms: correspondent banking for cash movement in currencies the firm does not settle domestically, global custodian networks for markets where it holds no account, **FX** for the currency conversion between trade currency and funding currency, and the settlement-cycle mismatch between markets. The repository's [Singapore Security Clearance Guide](../technology/singapore_security_clearance_guide.md) covers the local settlement infrastructure and the [Financial Infrastructure Guide](financial_infrastructure_guide.md) covers the wider map.

**(f) Nostro/vostro and intraday liquidity.** A securities firm settles in multiple currencies. The bank interface here is account structure, intraday liquidity provision, and the payment-rails connectivity that moves value between them — the domain of the [Payment Rails Guide](payment_rails_guide.md).

### 10.3 The documentation a bank would require (mechanism checklist)

Stated as the conventional set, not as anything verified about this entity:

| Interface | Typical documentation |
|---|---|
| Principal trading | Master agreement for trading in securities/derivatives, product annexes, credit-support annex |
| Clearing/custody | Clearing or custody agreement, account-opening pack, standing settlement instructions, market-specific addenda |
| Securities lending | Securities lending agreement (bilateral or via a global master), collateral schedule |
| Repo/financing | Global master repurchase agreement, annexes, margin provisions |
| FX | FX master agreement, confirmations process, netting provisions |
| Payments | Account documentation, mandate and signature authority, screening information |
| Regulatory | Entity constitutional documents, licence evidence, AML/KYC file, tax forms, sanctions screening |

### 10.4 The interface question, restated

The bank–securities-firm interface is not a single relationship; it is a **portfolio of contracts with different legal entities, different governing laws and different collateral pools**. The most common serious error is treating "we trade with Mirae Asset" as one relationship when it is several, each contracted with a specific legal person (this guide's §2 entity, or its Korean parent, or a different group affiliate) under a specific agreement. §12 turns that into a worked exercise.

---

## 11. The Honest Assessment

### 11.1 What an outside reader can know about a foreign securities subsidiary

The public record for a Singapore subsidiary of a foreign securities group is thin by construction, and it is thin in a patterned way:

| Layer | What is public | What is not |
|---|---|---|
| **Regulatory status** | Licence type, regulated activities, exempt status, key personnel, address and contact, updated by the regulator ✅ | Licence conditions, capital adequacy, inspection findings, correspondence with the regulator |
| **Corporate registry** | Name, UEN, incorporation date, type, status, former name, paid-up capital, registered office, activity codes, the existence of related-name entities ✅ | Shareholders and holdings (unless a paid profile is bought), directors and officers, financial statements for a private company, charges and encumbrances |
| **The firm's own website** | Self-description, business lines, address, a group-financials panel ✅ | Clients, volumes, revenues, headcount, systems, anything operational |
| **The parent's disclosures** | Group strategy, group network, group financials, group filings in Korea ✅ | The subsidiary's standalone financials; the subsidiary's risk profile; the legal ownership chain |
| **Press** | Milestones, M&A, leadership changes ⚠ | Anything the firm chooses not to speak about |

### 11.2 The three specific thinnesses for this entity

1. **Ownership is not public.** No source consulted in this pass names the shareholder of UEN 201106132Z or a percentage. Every guide that states an ownership figure for this entity is stating something this research could not verify. The parentage is *implied* by branding, by the parent's own family-website directory, and by the registry former name — and *undocumented* at the level of the register.
2. **Financials are not public.** The one financial-register fact available is paid-up capital (SGD 50,410,000) ⚠. That is a capital-at-incorporation-and-issuance figure, not revenue, not profit, not assets, not regulatory capital.
3. **The name-change date is not public.** The register records the former name Daewoo Securities (Singapore) and says nothing about when the change happened. The parent's name changes bracket the window (April 2016 → 24 March 2021); the Singapore change date falls inside it and is **not established** ⚠.

### 11.3 The Korean-language-source problem

The group's most authoritative public disclosures — KOSPI filings, DART disclosures, the Korean-language IR site, Korean business press — are **in Korean**, and they are the place where a serious analyst would resolve most of the gaps in §15. This guide uses:

- the **English-language section of the parent's own IR site** (first-party, dated, reliable for the parent's stated network and business descriptions) ✅;
- the **group's English "Our Story" timeline** on the US asset-management site ✅;
- **English-language Korean business press** (Yonhap, Business Korea) for dated corporate events ✅;
- **Wikipedia**, used with care and flagged where it carries its own quality problems (the Mirae Asset Securities article attaches *citation needed* to its founding claim; the group article carries conflict-of-interest and list-format maintenance tags) ⚠.

**What this guide did not do:** it did not read Korean-language primary filings. A reader who needs the group's audited consolidated accounts, or the DART filings that would evidence the Korean entity's subsidiaries, must go to the Korean primary sources. **Where a claim here rests on a secondary summary of a Korean-language source, it is marked as such** (e.g. the shareholding-change timeline in §4, and the Dow Jones Sustainability Index listing in §5.2) ⚠.

### 11.4 A finding worth surfacing because it is about the *sources*, not the firm

- The **Global X Canada press release of 1 May 2024** quotes the group's AUM as both **"more than $710 billion"** and, in its own About paragraph, **"more than $800 billion"** ✅⚠.
- The Singapore entity's own site shows group total client assets of **US$800.3bn** "estimated as of December 2025" while the Global X Singapore site shows **US$729bn+** sourced to **June 2025** and the Global X US site shows **$845bn+** sourced to **March 2026** ✅⚠.
- Wikipedia's group article lists an acquisition ("Betashares") that the group's own timeline does not support ❌.

These are not scandals; they are normal artefacts of marketing copy, differing definitions and differing dates. They are recorded here because the **habit a reader should take from this guide** is: quote a group number only with its source and its date, and never treat a group number as a subsidiary's.

### 11.5 The confidence statement for this guide's own claims

| Section | Basis | Confidence |
|---|---|---|
| §2 The Entity | MAS register (primary) + ACRA-derived aggregator (secondary) + the entity's own site | **High** on name, UEN, licence, activities, CEO, address; **explicitly nil** on ownership |
| §3 The Group | First-party group sites, Wikipedia with flags, dated press | **Medium** — group facts reliable when dated; ownership chain undocumented |
| §4 The Corporate History | Yonhap, Business Korea, first-party timelines, Wikipedia with flags | **High** on the major dated events (2016 acquisition and merger, 2021 rename); **flagged** on 1999 founding and the Singapore rename date |
| §5 The Securities Business | Generic (unmarked) + parent's dated claims + MAS register | **High** on what the licence permits; **no** claim about volumes, clients or revenue |
| §6 The Global Footprint | Affiliate-by-affiliate, first-party sites, dated | **Medium-high** with dated sources; **two continuity questions left open** (§6.3) |
| §7 What The Singapore Entity Is For | Generic (clearly separated) + four dated documented statements | **High** on the separation; **the documented half is thin by design** |
| §8 The Regulatory Position | MAS register (primary) + regime description cross-referenced out | **High** on the entity's status; obligations structural rather than entity-specific |
| §9 The Technology | Entirely generic; **no attribution** | **Nil** on this firm's stack — stated as such |
| §10 The Bank Interface | Mechanism-level, generic; no real institution asserted | **Nil** on this firm's arrangements — stated as such |
| §12 The Worked Example | Fictional | **Illustrative only** — no factual claim |

**The single most important sentence in this section:** the entity is real, live, licensed and identifiable at the regulator's register with high confidence ✅; almost everything about how it operates commercially is not public, and this guide says so rather than filling the space.

---

## 12. Worked Example: Cymbal Bank Onboards A Singapore Securities Firm

**This section is entirely fictional and illustrative.** Cymbal Bank is a fictional bank persona used across this repository as the only bank persona. **No real institution is asserted, anywhere in this section, as a client, counterparty, clearer, custodian or partner of Mirae Asset Securities (Singapore) Pte. Ltd., or of any other firm.** Nothing below is a statement about the subject entity's actual business arrangements. The scenario exists to make the identity work concrete.

### 12.1 The scenario

Cymbal Bank's markets division wants to trade with a Singapore securities firm that presents itself as "Mirae Asset Securities Singapore". Cymbal's onboarding analyst has a business card, a website, and a name that appears in a Korean group's global network. The onboarding pack requires the analyst to answer five questions before any line is opened.

### 12.2 Question 1 — Which legal entity is the counterparty?

The analyst must not accept "Mirae Asset Securities Singapore" as an answer. The correct form of the answer is a **legal entity name plus a registration number plus a jurisdiction**. Working through it:

1. **Search the regulator's register, not the website.** The MAS Financial Institutions Directory returns a listed entity: **MIRAE ASSET SECURITIES (SINGAPORE) PTE. LTD.**, marked "Incorporated in Singapore", licence status Capital Markets Services Licensee, with a named CEO ✅.
2. **Go to the company registry for the identifier.** The Singapore company record gives **UEN 201106132Z**, incorporated **14 March 2011**, status **Live Company**, former name **DAEWOO SECURITIES (SINGAPORE)**, paid-up capital SGD 50,410,000 ⚠ (ACRA-derived aggregator).
3. **Sweep for name collisions — this is the step everyone skips.** The same registry listing reveals a **second** company with the identical name, **UEN 201013438N**, status **Dissolved — Members Voluntary Winding Up**, registered at a different address (6 Shenton Way) and incorporated 25 June 2010 ⚠✅. If Cymbal's screening system keys on name, it may hit the dead one; if it keys on "latest", it may pick the wrong record; if it keys on the website, it learns nothing about either.
4. **Record the answer in the file** as: *Mirae Asset Securities (Singapore) Pte. Ltd., UEN 201106132Z, Singapore, CMS licensee, live* — and record the *second* UEN as a screened-and-excluded near match, with the reason.

**Outcome:** Cymbal has one entity, one identifier, and one documented near-miss. Had the analyst stopped at the website, the file would contain a name and a brand and no legal person.

### 12.3 Question 2 — Which regulator supervises it?

- The entity is **Singapore-incorporated**, which makes it a **subsidiary** rather than a branch ✅.
- It is supervised by **MAS** directly, as a domestic institution, under the **Securities and Futures Act / Capital Markets Services** framework ✅.
- Its licence covers **dealing in securities and collective investment schemes, advising on corporate finance, and (as an exempt financial adviser) advising on investment products and publishing research on securities** ✅.
- **What the analyst must not assume:** that the Korean parent's home-state licence covers the Singapore entity (it does not — different legal person, different regulator), and that the group's global reputation is a supervisory fact (it is not — MAS supervises this entity's own capital and conduct).

**Outcome:** the supervision answer is specific and citable, and the parent's status is recorded separately as *group information*, not as this entity's regulatory standing.

### 12.4 Question 3 — What does the parent relationship mean for risk assessment?

This is the question where good analysis and lazy analysis diverge most sharply.

**Legitimate uses of the parent relationship:**
- Understanding the **business model** — a cross-border brokerage channel for Korean and Asian equity, run by a group with a large global network, is a coherent story and helps the analyst ask the right questions ✅⚠.
- Understanding **where to look for group-level financial information** — the Korean listed parent's public disclosures, with the caveat that they are group-consolidated and Korean-language ⚠.
- Understanding **why the entity exists** — territorial licensing, regional sales coverage, cross-border deal originations ✅⚠.
- Assessing **reputational and franchise dependence** — an institutional client's willingness to trade with the subsidiary often turns on the group's standing, which is a real and legitimate risk factor.

**What the analyst must NOT do:**
- **Substitute the parent's capital for the subsidiary's** in any credit or capital assessment. The subsidiary's paid-up capital is a registry fact ⚠; the group's equity capital figure (US$17.2bn, "estimated as of December 2025") is a *group* figure and belongs in the file as context, not as the counterparty's resources ✅⚠.
- **Assume a guarantee or support agreement exists.** Group support is a commercial expectation; a legal obligation to support the subsidiary must be documented if it is to be relied upon. **The ownership chain and any support arrangements are not public** ❌ — so this goes in the file as an open item requiring the counterparty's own documentation.
- **Treat the group's rankings as the subsidiary's** ("South Korea's largest brokerage firm") — those are the Korean listed firm's, dated to 2016 reporting ✅⚠.

**Outcome:** the parent goes in the file as *group context, unverified ownership chain, no documented support arrangement*.

### 12.5 Question 4 — What documentation must Cymbal obtain?

The onboarding pack, as it would be assembled for a wholesale securities counterparty:

| Item | Purpose | What Cymbal should accept as evidence |
|---|---|---|
| Constitutional documents (constitution / memorandum) and certificate of incorporation | Establish the legal person, jurisdiction, and its powers | Registry-extracted documents in the entity's own name — not a group document |
| UEN confirmation | Match the contracting entity to the register | The UEN **201106132Z** on the contract must match the register entry |
| Licence evidence | Confirm regulated status and permitted activities | MAS register entry showing CMS licensee and the activity set |
| Board resolution / authority to enter the agreement | Confirm the signatory binds the entity | Executed resolution naming the agreement and the signatories |
| AML/KYC pack — ownership and control | Screen the entity, its owners and its controllers | Beneficial-ownership declaration, **which is the counterparty's burden to satisfy**; Cymbal should not infer ownership from branding ❌ |
| Sanctions and adverse-media screening | Identify legal, regulatory and reputational risk | Screen the entity, near-name matches (including UEN 201013438N) and the parent separately |
| Tax forms and FATCA/CRS status | Tax reporting | Signed forms from the entity |
| Standing settlement instructions | Settle correctly | SSIs verified by callback through a known channel — **never** from an email signature |
| Trading / clearing / lending / repo master agreements | Govern each interface separately | One agreement per interface per legal entity — see §10.4 |
| Financial information | Assess the counterparty | Ask for the entity's own financials; treat group financials as context only |

### 12.6 Question 5 — What is still open after onboarding?

A complete file should end with an explicit open-item list rather than a clean bill of health. For this scenario:

1. **Ownership chain unverified** — Cymbal could not confirm, from public sources, who owns UEN 201106132Z, or the percentages. The counterparty's own beneficial-ownership declaration is required.
2. **No documented group support** — the group context is understood; support is not documented.
3. **Technology and operational control unknown** — no public information establishes the entity's platforms, its cybersecurity posture, or its outsourcing arrangements; the counterparty's own operational-risk questionnaire is the only route.
4. **Near-name entity unresolved in substance** — UEN 201013438N is dissolved; its relationship (if any) to the live entity is unknown, and the file records the near match with the exclusion reason.
5. **Legal-name-in-use versus marketing-name** — the group operates a *separate* Singapore affiliate for ETFs (Global X Investments Pte Ltd, 61 Robinson Road) ✅; if any part of Cymbal's business is with that entity rather than the securities one, it is a different counterparty, a different set of agreements and a different screening file.
6. **The 2012-versus-2011 date gap** — the group's timeline dates the Singapore *subsidiary* to 2012 while the registry dates incorporation to March 2011 ✅⚠; the file records both and prefers the register.

**Outcome:** Cymbal has a defensible onboarding: correct legal entity, correct identifier, correct regulator, parent relationship characterised with its limits stated, documents specified per interface, and open items recorded rather than assumed away. That is the entire lesson of this guide expressed as a procedure.

---

## 13. The Anti-Patterns

Each entry is given as **symptom → cause → guardrail**, in the repository's house form.

### Anti-pattern 1: Confusing the group with the entity

- **Symptom.** A counterparty file, a pitch, or an article describes "Mirae Asset Securities (Singapore)" using the group's AUM, the group's employee count, the group's office count, or the group's ranking — and never names the UEN or the licence. Group financials appear on the subsidiary's own website, which makes the confusion feel sourced.
- **Cause.** Brand-driven reading. The group's brands are consistent ("Mirae Asset"), the group's marketing pages are the top search results, and the subsidiary's page *displays group figures* under a group label that is easy to miss. The two also share a name string with the Korean listed firm.
- **Guardrail.** **Anchor every statement to a legal entity.** If the sentence cannot carry a UEN (or a KRX ticker, for the parent) next to its subject, it is not yet a fact about anyone. Group figures go in the "group context" box with their date, never in the entity's columns.

### Anti-pattern 2: Treating the Korean parent's licence or capital as the Singapore entity's

- **Symptom.** A credit memo or a capability statement recites the parent's capital (or the group's equity capital and client assets) as if it were the Singapore company's financial resources, or implies the parent's Korean licences extend to Singapore activity.
- **Cause.** Read-across from a familiar group name; lazy substitution of "the group's" for "the entity's"; and the fact that the group does publish impressive figures while the subsidiary publishes almost none.
- **Guardrail.** Separate columns: **the entity** (UEN 201106132Z — CMS licence, registered activities, paid-up capital SGD 50.41m ⚠, CEO, address ✅) and **the group** (AUM/client assets, equity capital, network, rankings — with source and date ✅⚠). Never let a figure cross the columns. Where the entity's own financial resources are needed and are not public, **request them** rather than borrowing the parent's.

### Anti-pattern 3: Onboarding a counterparty without identifying the contracting legal entity

- **Symptom.** The trade documentation names a brand ("Mirae Asset Securities Singapore"), or names a group company that is not the licensed Singapore entity, or names the entity whose near-identical twin was dissolved. Legal-name mismatches surface at settlement, at credit-event time, or at close-out — the worst possible moments.
- **Cause.** Onboarding run from a business card and a website; screening keyed on name strings without a registry check; no name-collision sweep; no check that the signing entity is the licensed entity.
- **Guardrail.** Make "**legal entity name + identifier + jurisdiction**" a mandatory field and a hard stop. Run the register check (MAS directory **and** the company registry) before the KYC file is opened. Sweep for name collisions and record near matches *with reasons* — including the dissolved UEN 201013438N in this instance. Verify SSIs by callback, never from the paperwork.

### Anti-pattern 4: Assuming a foreign subsidiary's technology mirrors its parent's

- **Symptom.** A risk or architecture document asserts that the Singapore entity runs the parent's trading platform, the group's data stack, or the group's compliance tooling — often because the group publishes something about its own technology.
- **Cause.** Structural inference treated as fact. Subsidiaries genuinely do often inherit group platforms, which makes the inference feel safe; but they also frequently run local systems for local regulatory reasons, and the *inheritance is not documented* for this entity in any source consulted.
- **Guardrail.** Describe the **class** (an order/execution tier, market-data consumption under entitlement, post-trade processing, clearing connectivity, books and records, a regulatory reporting layer — see §9) and mark anything entity-specific as unverified. Attribute no platform to this firm. If the answer matters operationally, obtain it from the counterparty's operational-risk questionnaire or the group's own disclosure — not from analogy.

### Anti-pattern 5: Carrying a group-holdings claim forward after it has changed hands

- **Symptom.** A document states that the group owns a particular business as though the position were permanent — typically a name that was rebranded, sold, or partially divested since the source was written. The 2024 rebrand of Horizons ETFs to Global X Investments Canada is a live example: a rebrand is **not** a sale, and a naive reader can mistake the new name for a new owner.
- **Cause.** Undated sources; treating a rebrand as a transaction or a transaction as a rebrand; no re-verification against the parent's current disclosure; a search engine returning old pages first.
- **Guardrail.** For every holding: state the **date of the source**, the **direction of change** (acquired / rebranded / merged / sold / stake reduced / unknown), and a **current-status sentence**. If current status cannot be confirmed, write exactly that. In this guide the honest formulation is: **no divestment was found in this pass, Global X US still presented itself as a group member at a March 2026 source date, and current ownership of the Canadian and European platforms was not independently re-verified for September 2026** (see §6.3).

### Anti-pattern 6 (a sixth, specific to this name): assuming "Mirae Asset" anywhere means this entity

- **Symptom.** A record, a news item or a venture-round announcement involving a company with "Mirae Asset" in its name is attributed to this Singapore entity.
- **Cause.** The string "Mirae Asset" appears across a group with dozens of affiliates, across separate legal entities in the same city (the securities company at 6 Battery Road ✅ and the ETF distributor at 61 Robinson Road ✅), and across unrelated commercial names. The *name* is a weak identifier.
- **Guardrail.** Resolve to an identifier before attributing anything. UEN 201106132Z for this entity; a different UEN for the ETF affiliate; a Korean registration for the parent; and for anything else, no attribution at all.

---

## 14. The Claims Audit

Every substantive claim in this guide, with its source, its date, and its quality. ✅ = verified in this pass; ⚠ = flagged; ❌ = disputed or contradicted.

### 14.1 Entity identity claims

| Claim | Source | Date of source | Quality |
|---|---|---|---|
| The registered name is MIRAE ASSET SECURITIES (SINGAPORE) PTE. LTD. | MAS Financial Institutions Directory, entry 2448 | page last updated **11 Sep 2026**; retrieved Sep 2026 | ✅ primary |
| It is "Incorporated in Singapore" (i.e. a subsidiary, not a branch) | MAS FID entry 2448 | Sep 2026 | ✅ primary |
| Licence: Capital Markets Services Licensee | MAS FID entry 2448 | Sep 2026 | ✅ primary |
| Dealing in capital markets products — Securities and Collective Investment Schemes | MAS FID entry 2448 | Sep 2026 | ✅ primary |
| Advising on Corporate Finance | MAS FID entry 2448 | Sep 2026 | ✅ primary |
| Exempt Financial Adviser — advising on investment products (CIS, securities) | MAS FID entry 2448 | Sep 2026 | ✅ primary |
| Exempt Financial Adviser — issuing or promulgating analyses/reports on investment products (securities) | MAS FID entry 2448 | Sep 2026 | ✅ primary |
| CEO / key personnel: CHOI HYUNSUK | MAS FID entry 2448 | Sep 2026 | ✅ primary |
| Contact: +65 66719846; 6 Battery Road #29-04, Singapore 049909 | MAS FID entry 2448 | Sep 2026 | ✅ primary |
| UEN 201106132Z | ACRA-derived aggregator (companies.sg) | retrieved Sep 2026 | ✅ secondary (registry-derived) |
| Incorporated 14 March 2011 | ACRA-derived aggregator | retrieved Sep 2026 | ✅ secondary |
| Private Company Limited by Shares; status Live Company | ACRA-derived aggregator | retrieved Sep 2026 | ✅ secondary |
| Former entity name: DAEWOO SECURITIES (SINGAPORE) | ACRA-derived aggregator | retrieved Sep 2026 | ✅ secondary |
| Paid-up capital SGD 50,410,000 | ACRA-derived aggregator | retrieved Sep 2026 | ⚠ secondary, single source |
| Registered office 6 Battery Road #37-01A, Six Battery Road, Singapore 049909 | ACRA-derived aggregator | retrieved Sep 2026 | ⚠ secondary (note the floor differs from MAS and the website — both are genuine, different questions) |
| Principal/secondary activity: "Brokerage of Korean equity for Singapore institutional investors" | ACRA-derived aggregator (SSIC descriptions) | retrieved Sep 2026 | ⚠ secondary, aggregator-rendered |
| **A second Singapore registration with this identical name exists: UEN 201013438N, status "Dissolved — Members Voluntary Winding Up", address 6 Shenton Way, incorporated 25 June 2010** | ACRA-derived aggregator, "Companies with similar names" list | retrieved Sep 2026 | ✅ the *existence* of the second record; ⚠ its displayed name and activity (aggregator may render current names in the list) |
| The entity's shareholder(s) and shareholding percentages | — | — | ❌ **NOT ESTABLISHED — no source consulted names any shareholder or percentage** |
| The date of the Singapore entity's own renaming from Daewoo Securities (Singapore) | — | — | ⚠ **NOT ESTABLISHED** — register shows the former name only |
| The operational start date of the Singapore subsidiary | Entity's own website ("Singapore — 2012, Subsidiary Company Opened") | retrieved Sep 2026 | ⚠ first-party, one year after the registered incorporation (14 Mar 2011) |

### 14.2 Group, parent and history claims

| Claim | Source | Date of source | Quality |
|---|---|---|---|
| Mirae Asset Financial Group founded 1997 by Park Hyeon-joo | Wikipedia; Mirae Asset US "Our Story" timeline | Wikipedia retrieved Sep 2026; US timeline retrieved Sep 2026 | ✅ two sources |
| Mirae Asset Global Investments founded 1998; Mirae Asset Securities founded 1999 | Wikipedia (Securities founding carries *citation needed*); Mirae Asset US timeline | retrieved Sep 2026 | ✅ for the group's own claim; ⚠ for Wikipedia's uncited version |
| Daewoo Securities founded 1970 within the Daewoo conglomerate; spun off 1999 | Wikipedia | retrieved Sep 2026 | ✅ secondary |
| January 2016: Mirae Asset won the bid for a 43% stake in Daewoo Securities, ₩2.39tn (≈US$2bn) | Bloomberg, 25 Jan 2016 (via Wikipedia citation) | 25 Jan 2016 | ✅ dated secondary |
| April 2016: acquisition completed — 43% of KDB Daewoo Securities from Korea Development Bank for ₩2.32tn (~US$2bn); renamed Mirae Asset Daewoo later that month | Yonhap | 4 Nov 2016 | ✅ news agency |
| 4 Nov 2016: shareholders approve the merger; combined capital ₩6.7tn; listing slated 20 Jan | Yonhap | 4 Nov 2016 | ✅ (⚠ Yonhap's phrase "the two asset management firms" is imprecise) |
| 29 Dec 2016: merger completed | Yonhap; Wikipedia | 4 Nov 2016; retrieved Sep 2026 | ✅ |
| 24 March 2021: renamed Mirae Asset Daewoo → Mirae Asset Securities at the shareholders' meeting | Business Korea | article dated 25 Mar 2021, reporting the 24 Mar meeting | ✅ |
| 2020 dividends: 200 won/common share, 220 won/preferred; 2017–2019 dividends ₩124.7bn/₩153.9bn/₩182.1bn; 2020 dividends plus cancelled shares ≈ ₩280.5bn = 34.1% of net profit | Business Korea | 25 Mar 2021 | ✅ dated secondary (these are **parent-level** figures) |
| "South Korea's largest brokerage firm" (2016 merger context) | Yonhap | 4 Nov 2016 | ⚠ third-party claim, dated |
| "Largest investment banking and stock brokerage company by market capitalisation in South Korea" | Wikipedia | undated in text; retrieved Sep 2026 | ⚠ single-source, undated |
| Group total client assets US$800.3bn ("estimated as of December 2025") | Entity's own Singapore website | retrieved Sep 2026 | ✅ as a first-party group figure ⚠ (conflicts in magnitude/date with other first-party figures) |
| Group: 16,000+ employees, $845bn+ AUM, 67 offices | globalxetfs.com/about, citing Mirae Asset | source dated **March 2026**; retrieved Sep 2026 | ✅ first-party ⚠ (figures dated and inconsistent with other first-party figures) |
| Group: US$729.5bn AUM and 15,997 employees (2025) | Wikipedia, citing am.miraeasset.com/about-us | retrieved 4 Oct 2025 | ⚠ secondary rendering of a first-party page |
| "2011.11 Acquired Betashares, a major ETF player in Australia" | Wikipedia M&A list | retrieved Sep 2026 | ❌ **NOT SUPPORTED by any other source consulted; treat as erroneous** |
| Sharekhan acquisition: SPA in 2023 (group timeline) vs completion 2024.11 (Wikipedia) | Mirae Asset US timeline; Wikipedia citing a Sharekhan post | retrieved Sep 2026 | ⚠ date discrepancy recorded, not resolved |

### 14.3 Global footprint claims

| Claim | Source | Date of source | Quality |
|---|---|---|---|
| Parent's Singapore operation described as "Engaging in IB, Global FI Sales and Cross-border deals" | Parent IR site, global network cards | retrieved Sep 2026 | ✅ first-party |
| Parent IR family directory lists `securities.miraeasset.com.sg` under Singapore → Mirae Asset Securities, and `globalxetfs.sg` under Singapore → Global X | Parent IR site | retrieved Sep 2026 | ✅ first-party |
| Group network openings: London 1991, New York 1992, Hong Kong 1994, Beijing 2007, Vietnam 2007, Brazil 2010, Shanghai 2010, Singapore 2012, Indonesia 2013, Mongolia 2013, India 2017 | Entity's own website, "Who We Are" timeline | retrieved Sep 2026 | ✅ first-party ⚠ (Singapore 2012 conflicts with the 2011 incorporation) |
| Parent claims "the largest global network in Korea, in 11 countries" | Entity's own website | retrieved Sep 2026 | ⚠ first-party marketing claim |
| Wikipedia states the firm operates "in ten foreign countries" | Wikipedia | retrieved Sep 2026 | ⚠ single-source, undated, conflicts with the 11-country claim |
| Global X (US) acquired by Mirae Asset Global Investments July 2018 for US$488m | Wikipedia (Global X ETFs); Mirae Asset US timeline ("2018 — Acquires Global X") | retrieved Sep 2026 | ✅ |
| Global X US: HQ New York; ~$169bn AUM in Global X ETFs listed worldwide (June 2026); 200+ professionals; CEO Ryan O'Connor; **"member of Mirae Asset Financial Group"** | globalxetfs.com/about | AUM as of **June 2026**; group figures sourced **March 2026**; retrieved Sep 2026 | ✅ first-party — **this is the evidence that Global X US still presented itself as a group member in 2026** |
| Horizons ETFs (Canada) acquired by the group in 2011 (85% stake reported) | BusinessWire 14 Nov 2011 (via Wikipedia citation); Mirae Asset US timeline | 14 Nov 2011 | ✅ dated secondary |
| Horizons ETFs Management (Canada) Inc. rebranded to Global X Investments Canada Inc. effective 1 May 2024 (announced 6 March 2024) | Global X Canada press release; Mirae Asset US site 18 Apr 2024 | 1 May 2024 / 6 Mar 2024 / 18 Apr 2024 | ✅ — **a rebrand, not a sale** |
| Global X Canada: "> $30 billion of AUM and 108 ETFs listed on major Canadian stock exchanges"; "wholly owned subsidiary of Mirae Asset Global Investments Co., Ltd." | Global X Canada press release | 1 May 2024 | ✅ as at that date; ⚠ **current ownership not confirmed as at Sep 2026** |
| Global X Canada site live, © 2026 | globalx.ca (press release page) | retrieved Sep 2026 | ✅ |
| ETF Securities Australia acquired 2022, now Global X Australia; Brazil renamed Global X Brazil 2022; Global X Japan JV with Daiwa 2019; GHCO acquired by Mirae Asset Securities 2022 | Mirae Asset US timeline; Wikipedia | retrieved Sep 2026 | ✅ first-party / dated |
| **Global X Investments Pte Ltd**, 61 Robinson Road Level 07-01 Suite 758, Singapore 068893; contact Richard Siaw, Director Southeast Asia; site data ~$115bn global ETF AUM (Aug 2025), group figures sourced June 2025 | globalxetfs.sg | retrieved Sep 2026 | ✅ first-party — **a separate Singapore legal entity from this guide's subject** |
| No divestment of any group ETF holding was found in this pass | — (negative result) | Sep 2026 | ⚠ **negative result from limited search; Python/web_search returned empty sets (see §15.3)** |
| `globalx.com.au` is Dye & Durham's legal-information business, formerly GlobalX — unrelated to the group | web extraction of globalx.com.au | retrieved Sep 2026 | ✅ — a name-collision warning, not a group claim |

### 14.4 Regulatory, technology and interface claims

| Claim | Source | Date of source | Quality |
|---|---|---|---|
| The entity is supervised by MAS as a Singapore-incorporated CMS licensee (subsidiary, not branch) | MAS FID entry 2448 (status line) + SFA/CMS regime | Sep 2026 | ✅ |
| A CMS licence imports business-conduct, capital, AML/CFT, record-keeping and reporting obligations in the applicable form | Regime knowledge; cross-referenced to [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md) | — | structural/unmarked — obligation detail **not** on the register |
| Licence conditions, regulatory capital position, inspection findings and enforcement history for this entity | — | — | ❌ **not published on the register; not established** |
| The entity's OMS/EMS, market-data vendors, clearing connectivity, books-and-records platform, reporting tools, cloud posture | — | — | ❌ **NOT publicly documented — no attribution made anywhere in this guide** |
| The technology description in §9 (order/execution tiers, market-data consumption categories, post-trade stages, clearing connectivity, reporting layer) | Industry knowledge; cross-referenced to [Market Data Consumption](market_data_consumption_guide.md), [Market Data Integrity](market_data_integrity_guide.md), [FINOS CDM](finos_cdm_guide.md), [FIX Protocol](fix_protocol_guide.md), [Singapore Security Clearance](../technology/singapore_security_clearance_guide.md) | — | unmarked **class description**, explicitly not an entity claim |
| The bank-interface mechanisms in §10 (execution counterparty, clearing/custody either direction, securities lending, collateral and margin, cross-border settlement, nostro/vostro) | Industry knowledge; cross-referenced to [FINOS CDM Guide](finos_cdm_guide.md), [Singapore Security Clearance Guide](../technology/singapore_security_clearance_guide.md), [Financial Infrastructure Guide](financial_infrastructure_guide.md) | — | unmarked **mechanism description**; **no real institution asserted as this firm's counterparty** |
| Any institution as a client, counterparty, clearer, custodian or partner of this firm | — | — | ❌ **none asserted — the governing rule of this guide** |
| The §12 worked example | Fictional | — | **illustrative only; no factual claim about any real party** |

---

## 15. What Could Not Be Verified

### 15.1 The entity-specific gap list

The following were sought and **could not be established** from public sources in this research pass. Each is a genuine gap, not an omission of effort.

1. **Ownership and shareholding.** The identity of the shareholder(s) of UEN 201106132Z, and any percentage holding. Consequently: **no ownership percentage is asserted anywhere in this guide**, and no parent-subsidiary legal link is asserted beyond what the group's own website and the registry former name imply. Marked **UNVERIFIED** in §2.5 and §3.5.
2. **The legal chain to the Korean parent.** Whether shares are held directly by Mirae Asset Securities Co., Ltd. or through one or more intermediate holdings. **UNDOCUMENTED.**
3. **The date of the Singapore entity's own renaming** from Daewoo Securities (Singapore) to Mirae Asset Securities (Singapore). Bounded by the parent's name changes (April 2016 → 24 March 2021); the specific date is **NOT ESTABLISHED**.
4. **Financial statements.** Revenue, profit, total assets, regulatory capital position, and audited accounts. Only the registry paid-up capital figure (SGD 50,410,000) is available ⚠. No paid ACRA BizFile+ profile was purchased.
5. **Headcount and organisational structure.** Not published.
6. **Licence conditions and any regulatory action.** The MAS directory entry publishes licence type and activities, not conditions, capital requirements or enforcement history. **Nothing was found, and nothing is asserted about compliance or enforcement in either direction.**
7. **Clients, counterparties, custodians, clearing members and trading venues.** None identified, and **none asserted**.
8. **Products and volumes.** No product list, no trade volumes, no revenue split by business line.
9. **The relationship, if any, between the live entity (201106132Z) and the dissolved same-name entity (201013438N).** Their relationship is **UNKNOWN**; only the existence of both records is established.
10. **The operational start date** of the Singapore business (the group says 2012; the register says incorporated March 2011 — the gap is unexplained).
11. **A first-party statement of ownership** from the entity or its parent naming the shareholder, at any date.
12. **The technology estate** — platforms, vendors, in-house builds, cloud, data architecture. **Not publicly documented** (§9.7).

### 15.2 The group-level gaps carried into this guide

13. **Group AUM and client-asset figures are mutually inconsistent across first-party sources** (US$550bn Dec 2020; US$729bn+ June 2025; US$800.3bn Dec 2025 estimate; US$845bn+ March 2026; and "$710bn"/"$800bn" in one 2024 press release). The definitional basis of each is not stated. **Not resolved.**
14. **Current ownership status of Global X Canada, Global X Europe and Global X Australia** as at September 2026. Verified as of the dates of the sources cited (§6.2); **not independently re-verified for September 2026**. No divestment was found; that is a negative result, not a confirmation.
15. **The correct count of the parent's foreign markets** — "11 countries" (first-party, 2026) versus "ten foreign countries" (Wikipedia, undated). Not resolved.
16. **The KDB takeover date of Daewoo Securities** (the point at which the firm became "KDB Daewoo Securities"). The *name* is documented from 2016 sources; the timing of the takeover is **not established** by anything consulted.
17. **The Sharekhan acquisition date** — 2023 SPA (first-party) versus 2024.11 completion (Wikipedia). Both statements may be true of different events; **not resolved**.
18. **The "Betashares" acquisition claim** in Wikipedia's group M&A list. **Rejected as unsupported** in this pass, not merely flagged.

### 15.3 Tool and access limitations encountered (recorded honestly)

This is a record of what this research pass could *not do*, so that no reader mistakes a limitation for a finding:

1. **Web search returned empty result sets for every query attempted**, including the two gap queries requested for the brief — a query about a possible **Global X US divestment/sale in 2025** and a query about the **group's 1997 founding** — and also for control queries that must have had results (e.g. "Mirae Asset Securities"). **The tool returned `data.web: []` reproducibly. This is a tool limitation and is NOT evidence of absence.** Compensating action: direct URL extraction (web_extract) was used instead, which is how the entity's own website, the parent's IR directory, the Global X US about page and the group's "Our Story" timeline were obtained.
2. **Direct search-engine endpoints were unreachable** — extraction of DuckDuckGo HTML and Lite search URLs failed with engine errors. No general web discovery was therefore available; only known-URL extraction worked.
3. **No paid ACRA BizFile+ search or business profile was retrieved.** All registry-derived facts rest on a third-party aggregator that renders ACRA data. Only BizFile+ is authoritative.
4. **No Korean-language primary filings were read** (KOSPI/DART disclosures, Korean-language IR pages). Group claims resting on Korean-language sources are marked as resting on secondary summaries.
5. **Some affiliate pages returned 404s** during extraction (the Global X Europe about-us page; a parent IR business sub-page), so those pages' contents could not be confirmed beyond the live navigation. Flagged rather than guessed.
6. **The MAS register's search interface is JavaScript-driven**, so entities were verified by direct URL to the known detail page rather than by exhaustive directory search. This means **other Singapore-registered "Mirae Asset" entities may exist that this pass did not enumerate** — the two Singapore entities named in this guide (the securities company and the ETF distributor) are those surfaced by the sources consulted, not a certified complete list.

### 15.4 What is *not* a gap (so the reader does not over-correct)

The following **were** verified and should not be treated as unknown: the entity's registered name, its UEN, its Singapore incorporation, its live status, its licence type including each regulated activity, its exempt financial-adviser activities, its named CEO, its MAS contact details, its registered office, its paid-up capital (secondary), its former name, its registered activity description, the existence of the second same-name registration, and the parent's own description of the Singapore operation.

---

## 16. The Glossary, Cross-References and Closing Summary

### 16.1 Glossary

| Term | Meaning as used in this guide |
|---|---|
| **ACRA** | Accounting and Corporate Regulatory Authority, Singapore's company registrar. BizFile+ is its authoritative public search; only a BizFile+ extract is definitive. |
| **Advised on corporate finance** | A regulated activity covering advice to corporations on capital structure, takeovers, fund-raising and similar; appears on this entity's MAS entry. |
| **Branch vs subsidiary** | A branch is an extension of a foreign legal person; a subsidiary is a locally incorporated separate legal person regulated as a domestic institution. This entity is a **subsidiary**. |
| **Capital Markets Services (CMS) licence** | The Singapore authorisation under the Securities and Futures Act to carry on regulated capital-markets activities such as dealing in capital markets products. |
| **Collective Investment Scheme (CIS)** | A pooled investment arrangement (fund); named on this entity's register entry both under dealing and under exempt financial advice. |
| **CMS licensee** | The holder of a Capital Markets Services licence. |
| **Counterparty** | The legal person on the other side of a contract. In this guide, always a specific entity, never a brand. |
| **Exempt Financial Adviser** | A status recorded by MAS for an entity exempted from holding a financial adviser's licence for specified activities; not a licence substitute for dealing. |
| **Dealing in capital markets products** | Carrying on business of buying or selling capital markets products, as principal or agent; the core regulated activity on this entity's entry. |
| **Financial Advisers Act (FAA)** | The Singapore statute governing financial advice and the issuing/promulgation of analyses and reports on investment products. |
| **Institutional investor** | A wholesale client class (funds, insurers, banks, sovereign entities) treated differently from retail clients under Singapore conduct rules. |
| **Key personnel** | On the MAS directory, the senior officers the register lists for a licensee — here the Chief Executive Officer. |
| **Market-making** | Quoting continuous two-way prices to provide liquidity; a business activity, not a licence name. See [Market Making Singapore Guide](market_making_singapore_guide.md). |
| **MAS** | Monetary Authority of Singapore — central bank and integrated financial regulator; publisher of the Financial Institutions Directory. |
| **Near-name match** | A screening hit on a similar-but-distinct legal entity (e.g. a dissolved company with an identical name, or an unrelated company sharing a brand string). Recorded with an exclusion reason. |
| **OMS / EMS** | Order Management System / Execution Management System — the capture/validate/route tiers of a trading estate. |
| **Paid-up capital** | The amount of capital actually paid in on issued shares; a registry fact, **not** regulatory capital. |
| **Principal (dealing as)** | Trading for one's own account, as opposed to as agent for a client. |
| **Securities and Futures Act (SFA)** | The Singapore statute governing capital-markets activity, licensing and market conduct. |
| **SSIC** | Singapore Standard Industrial Classification — the activity codes used in the company registry; the source of this entity's "brokerage of Korean equity…" activity description. |
| **UEN** | Unique Entity Number — the Singapore identifier for a registered entity. The subject of this guide is **201106132Z**. |
| **Uncleared / cleared margin** | The margin regimes applicable to derivative exposures depending on whether the trade is centrally cleared; drives collateral and documentation design. |

### 16.2 Cross-references

**Regulation and market structure:** [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md) · [Capital Markets Architecture Guide](capital_markets_architecture_guide.md) · [Financial Infrastructure Guide](financial_infrastructure_guide.md) · [Singapore Security Clearance Guide](../technology/singapore_security_clearance_guide.md) · [Market Making Singapore Guide](market_making_singapore_guide.md)

**Trading and technology:** [FIX Protocol Guide](fix_protocol_guide.md) · [Financial Trading & Order Infrastructure](financial_trading_order_infrastructure.md) · [FINOS CDM Guide](finos_cdm_guide.md) · [Market Data Consumption Guide](market_data_consumption_guide.md) · [Market Data Integrity Guide](market_data_integrity_guide.md) · [Nasdaq Calypso Guide](nasdaq_calypso_guide.md) · [Online Investment Trading Platforms Guide](online_investment_trading_platforms_guide.md)

**Securities-firm peers and Singapore institutions:** [TD Securities Software Systems Guide](td_securities_software_systems_guide.md) · [Morgan Stanley Software Systems Guide](morgan_stanley_software_systems_guide.md) · [Banks in Singapore Guide](banks_in_singapore_guide.md) · [DBS Bank Guide](dbs_bank_guide.md) · [Standard Chartered Guide](standard_chartered_guide.md) · [Hedge Funds Singapore Guide](hedge_funds_singapore_guide.md) · [Private Equity Singapore Guide](private_equity_singapore_guide.md)

**Other company guides in the house shape:** [Univers Guide](../technology/univers_guide.md) · [SignalPlus Guide](signalplus_guide.md)

### 16.3 Closing summary

This guide set out to do one thing: describe **one Singapore legal entity** — Mirae Asset Securities (Singapore) Pte. Ltd., UEN 201106132Z — accurately, and to keep it strictly apart from the Korean listed securities firm (KRX 006800), the Mirae Asset Financial Group, and the group's asset-management and ETF businesses.

What the record supports, and what it does not:

- **The entity exists, is live, and is licensed.** MAS lists it as a Singapore-incorporated **Capital Markets Services licensee** dealing in securities and collective investment schemes, advising on corporate finance, and acting as an **exempt financial adviser** for product advice and for issuing or promulgating analyses on securities, with a named CEO (CHOI HYUNSUK) and a Battery Road address ✅.
- **Its Singapore identity is documented twice over**, in the register of companies (incorporated 14 March 2011, former name Daewoo Securities (Singapore), paid-up capital SGD 50,410,000) and on the regulator's directory — **with a genuine identification trap**: a second, dissolved Singapore registration bears the identical name (UEN 201013438N) ⚠✅.
- **Its role is documented at the level of description, not at the level of fact.** The registry calls it Korean-equity brokerage for Singapore institutional investors; the parent calls the Singapore operation "IB, Global FI Sales and Cross-border deals"; the entity calls itself a sales-and-trading, IB and advisory business for institutional investors ✅⚠. Its clients, flows, financials and systems are **not public** ❌.
- **The group is documented; the ownership chain is not.** Group founding (1997), the 2016 Daewoo acquisition and merger, and the March 2021 renaming are all dated and sourced ✅; **the shareholder of the Singapore entity and any percentage holding are not established** ❌, and are therefore asserted nowhere in this guide.
- **The technology is described only as a class.** No platform, vendor or system is attributed to this firm, and §9 says so explicitly rather than importing a parent's or a peer's estate.
- **No real institution is presented as this firm's client or counterparty**, and the fictional worked example (§12) exists purely to make the identity discipline concrete — identify the contracting legal entity, resolve its identifier, find its regulator, characterise the parent relationship with its limits stated, document each interface separately, and record the open items instead of assuming them away.

The honest bottom line for a reader using this guide as reference material: **you can be confident about who this firm is and what it is licensed to do, and you should be sceptical of anyone — including this guide's own industry-knowledge sections — who tells you how it operates.** The register tells you the role; the market tells you the value; and the entity stands — as every securities firm does — between the client and the venue, where the order meets the market.
