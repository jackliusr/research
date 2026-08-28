# Cayman and BVI Master-Feeder Fund Structures: A Comprehensive Guide

**The Offshore Architecture — Master-Feeder Mechanics and Why They Exist, the US Tax Drivers (PFIC, QEF, UBTI, FATCA, ECI, ERISA Plan Assets), the Cayman Jurisdiction (Exempted Companies, SPCs, CIMA, the Private Funds Law, Economic Substance), the BVI Jurisdiction (SIBA Fund Categories, the Private Investment Funds Act, Economic Substance), the Cayman-vs-BVI Comparison, the Marketing and Onshore Alternatives, and a Cymbal Bank Worked Example of a Cayman Master with Delaware and Offshore Feeders**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Banking Domain / Institutional Investment & Capital Markets — the dedicated offshore-structure deep-dive of the repository's funds cluster: the master-feeder architecture (the Delaware LP feeder for US taxable investors, the Cayman/BVI offshore feeder for non-US and tax-exempt investors, the Cayman or BVI master as the single trading entity), the US tax drivers that make the architecture necessary (the PFIC regime and the QEF election — IRC §§1291/1293/1295, UBTI and the blocker question, FATCA, ECI), the ERISA plan-assets look-through and its 25% test (29 CFR §2510.3-101), the two offshore jurisdictions in full (Cayman: exempted companies, SPCs, the Mutual Funds Law and CIMA, the Private Funds Law 2020, the International Tax Co-operation (Economic Substance) Law 2018; BVI: the BVI business company, the Securities and Investment Business Act 2010 fund categories, the Private Investment Funds Act 2019, the Economic Substance (Companies and Limited Partnerships) Act 2018), the comparison table including the current EU-list status of both jurisdictions, and the Cymbal Bank worked example (Cayman master + Delaware feeder + offshore feeder with Cymbal Bank as administrator, custodian and prime broker)
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** August 2026
> **Companion guides (sibling, same folder — the funds cluster):** [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) (§3.1–3.2 — the VCC and "Why Global Managers Still Go Offshore"; §9 — the Cymbal Bank worked-example conventions; do not re-derive) · [Private Equity](private_equity_guide.md) (§7 — AIFMD and AIFMD II; §8 — the Singapore regime; §9 — the subscription-line worked example; do not re-derive) · [Citadel LLC](citadel_llc_guide.md) (§10 — the prime-brokerage worked-example mechanics) · [Fircosoft](fircosoft_guide.md) (the sanctions-screening and KYC/AML themes for the worked example) · [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) (the Singapore regulatory overlay) · [Norges Bank Investment Management](norges_bank_investment_management_guide.md) (the institutional-LP view)

---

**How to use this guide:** Section 1 is the overview — the short answer, the scope, the key-facts table, and why the structure matters to a bank. Section 2 is the master-feeder architecture — the vehicles, the aggregation logic, and the per-investor tax isolation that the whole structure exists to deliver. Section 3 is the US tax drivers — the PFIC regime and the QEF election (IRC §§1291/1293/1295/1296/1297), UBTI and blockers, FATCA, and ECI. Section 4 is the ERISA and benefit-plan-investor angle — the plan-assets regulation (29 CFR §2510.3-101) and its 25% test. Sections 5 and 6 are the two jurisdictions in full — Cayman and BVI respectively. Section 7 is the comparison table, including the current EU-list status of both jurisdictions. Section 8 is the fund-marketing context (AIFMD and the NPPRs, condensed and cross-referenced); Section 9 is the onshore-vs-offshore choice (the Singapore VCC, cross-referenced). Section 10 is the Cymbal Bank worked example — a Cayman master with a Delaware feeder and an offshore feeder, with Cymbal Bank as administrator, custodian and prime broker. Section 11 is the claims audit (✅/⚠/❌); Section 12 is "What Could Not Be Verified"; Section 13 is the glossary; Section 14 is cross-references; Section 15 is the closing summary. Cross-references follow the repository convention: sibling guides in `banking/` are plain filenames. **Integrity convention:** ✅ = verified this pass against a primary or cited source (source named in §11); ⚠ = flagged/unverified — press estimate, contested, or not re-verified live; ❌ = refuted or rejected. Nothing in this guide was invented; figures that could not be re-verified are marked ⚠ and listed again in §12.

---

### Table of Contents

1. [The Overview](#1-the-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [What This Guide Covers](#12-what-this-guide-covers)
   - 1.3 [The Key-Facts Table](#13-the-key-facts-table)
   - 1.4 [Why This Matters to a Bank](#14-why-this-matters-to-a-bank)
2. [The Master-Feeder Architecture — What It Is and Why](#2-the-master-feeder-architecture--what-it-is-and-why)
   - 2.1 [The Two-Tier Structure](#21-the-two-tier-structure)
   - 2.2 [The Vehicles in the Stack](#22-the-vehicles-in-the-stack)
   - 2.3 [Why the Structure Exists — Aggregation and Tax Isolation](#23-why-the-structure-exists--aggregation-and-tax-isolation)
   - 2.4 [How Money Moves — Subscriptions, Master Aggregation, NAV, Redemptions](#24-how-money-moves--subscriptions-master-aggregation-nav-redemptions)
   - 2.5 [Master-Feeder vs Standalone](#25-master-feeder-vs-standalone)
3. [The Tax Drivers — PFIC, QEF, UBTI, FATCA, ECI](#3-the-tax-drivers--pfic-qef-ubti-fatca-eci)
   - 3.1 [The PFIC Regime — IRC §1297, §1291, §1293, §1295](#31-the-pfic-regime--irc-1297-1291-1293-1295)
   - 3.2 [The QEF Election and the Practical Mechanics](#32-the-qef-election-and-the-practical-mechanics)
   - 3.3 [UBTI and the Blockers for Tax-Exempt US Investors](#33-ubti-and-the-blockers-for-tax-exempt-us-investors)
   - 3.4 [FATCA and Withholding](#34-fatca-and-withholding)
   - 3.5 [ECI — Effectively Connected Income](#35-eci--effectively-connected-income)
4. [The ERISA and Benefit-Plan-Investor Angle — the 25% Test](#4-the-erisa-and-benefit-plan-investor-angle--the-25-test)
   - 4.1 [The Plan-Assets Regulation — 29 CFR §2510.3-101](#41-the-plan-assets-regulation--29-cfr-25103-101)
   - 4.2 [What Triggers the Look-Through, and the Consequences](#42-what-triggers-the-look-through-and-the-consequences)
   - 4.3 [The Structuring Answers — Feeders, Caps, and Blockers](#43-the-structuring-answers--feeders-caps-and-blockers)
5. [The Cayman Jurisdiction](#5-the-cayman-jurisdiction)
   - 5.1 [The No-Direct-Tax Profile and the Exempted Company](#51-the-no-direct-tax-profile-and-the-exempted-company)
   - 5.2 [The SPC — Part XIV of the Companies Act](#52-the-spc--part-xiv-of-the-companies-act)
   - 5.3 [CIMA and the Mutual Funds Law — Open-Ended Funds](#53-cima-and-the-mutual-funds-law--open-ended-funds)
   - 5.4 [The Private Funds Law 2020 — Closed-Ended Funds](#54-the-private-funds-law-2020--closed-ended-funds)
   - 5.5 [The Economic Substance Regime](#55-the-economic-substance-regime)
6. [The BVI Jurisdiction](#6-the-bvi-jurisdiction)
   - 6.1 [The BVI Business Company and the FSC](#61-the-bvi-business-company-and-the-fsc)
   - 6.2 [SIBA 2010 — the Five Fund Categories](#62-siba-2010--the-five-fund-categories)
   - 6.3 [The Private Investment Funds Act 2019 — Closed-Ended Funds](#63-the-private-investment-funds-act-2019--closed-ended-funds)
   - 6.4 [The BVI Economic Substance Regime](#64-the-bvi-economic-substance-regime)
   - 6.5 [The Cost and Speed Profile](#65-the-cost-and-speed-profile)
7. [The Comparison Table — Cayman vs BVI](#7-the-comparison-table--cayman-vs-bvi)
   - 7.1 [The Row-by-Row Comparison](#71-the-row-by-row-comparison)
   - 7.2 [Reputational and Regulatory Standing — the EU List in 2026](#72-reputational-and-regulatory-standing--the-eu-list-in-2026)
8. [The Fund-Marketing Context — AIFMD and the NPPRs](#8-the-fund-marketing-context--aifmd-and-the-npprs)
   - 8.1 [AIFMD and Non-EU Managers (Cross-Referenced)](#81-aifmd-and-non-eu-managers-cross-referenced)
   - 8.2 [The NPPRs in Practice](#82-the-npprs-in-practice)
9. [The Onshore-vs-Offshore Choice — the Singapore VCC Alternative](#9-the-onshore-vs-offshore-choice--the-singapore-vcc-alternative)
   - 9.1 [The VCC and the 13O/13U Incentives (Cross-Referenced)](#91-the-vcc-and-the-13o13u-incentives-cross-referenced)
   - 9.2 [When Offshore Wins, and When Onshore Does](#92-when-offshore-wins-and-when-onshore-does)
10. [The Worked Example — A Cayman Master with Delaware and Offshore Feeders as a Cymbal Bank Client](#10-the-worked-example--a-cayman-master-with-delaware-and-offshore-feeders-as-a-cymbal-bank-client)
    - 10.1 [The Scenario and the Structure Diagram](#101-the-scenario-and-the-structure-diagram)
    - 10.2 [The Flow of Funds — Subscriptions to Distributions](#102-the-flow-of-funds--subscriptions-to-distributions)
    - 10.3 [The Cymbal Bank Product Map — Administration, Custody, Prime Brokerage](#103-the-cymbal-bank-product-map--administration-custody-prime-brokerage)
    - 10.4 [NAV, Reporting, and the Redemption Cycle](#104-nav-reporting-and-the-redemption-cycle)
    - 10.5 [The KYC/AML Overlay](#105-the-kycaml-overlay)
11. [The Claims Audit — Verified, Flagged, Rejected](#11-the-claims-audit--verified-flagged-rejected)
    - 11.1 [The Verified Claims (✅)](#111-the-verified-claims-)
    - 11.2 [The Flagged Claims (⚠)](#112-the-flagged-claims-)
    - 11.3 [The Rejected Claims (❌)](#113-the-rejected-claims-)
12. [What Could Not Be Verified](#12-what-could-not-be-verified)
13. [Glossary](#13-glossary)
14. [Cross-References and Further Reading](#14-cross-references-and-further-reading)
15. [Closing Summary](#15-closing-summary)

---

## 1. The Overview

### 1.1 The Short Answer

**The master-feeder structure is the standard architecture of the global hedge-fund and private-markets industry: two (or more) investor-facing "feeder" vehicles — one for US taxable investors, typically a Delaware limited partnership, and one for non-US and tax-exempt investors, typically a Cayman Islands or BVI vehicle — that each subscribe into a single "master" fund, the Cayman or BVI entity that actually holds the portfolio and trades.** The structure exists for one dominant reason: it lets a manager run **one portfolio, one trading book, and one set of service-provider relationships at the master level** while giving **each investor group its own tax and regulatory box at the feeder level** — US taxable investors get PFIC/QEF treatment through the Delaware feeder, US tax-exempt investors are kept out of the UBTI path, ERISA plans are managed against the 25% plan-assets look-through, and non-US investors never touch US tax concepts at all.

The architecture is verified at the level of the underlying statutes and regulations, not merely the marketing material:

- **The US tax drivers are statutory.** A Cayman or BVI fund is a **Passive Foreign Investment Company (PFIC)** for US federal income-tax purposes, and the PFIC regime — **IRC §1291** (default taxation of "excess distributions" with an interest charge), **§1293** (current-year inclusion for a **Qualified Electing Fund**), **§1295** (the QEF election), **§1296** (mark-to-market), **§1297** (the PFIC definition — the 75% passive-income test or the 50% passive-asset test) — is verified at law.cornell.edu/uscode and at irs.gov (Form 8621, "Information Return by a Shareholder of a Passive Foreign Investment Company or Qualified Electing Fund", instructions dated 12/2025) ✅ (Section 3).
- **The ERISA constraint is regulatory.** The DOL plan-assets regulation — **29 CFR §2510.3-101** — provides that a plan's assets include an undivided interest in the underlying assets of an entity unless the entity is an operating company or "equity participation in the entity by benefit plan investors is not significant" — and paragraph (f)(1) defines "significant" as **25% or more of the value of any class of equity interests held by benefit plan investors** ✅ (Section 4, verified at law.cornell.edu/cfr).
- **Both offshore jurisdictions have regulated, tax-neutral fund regimes** — Cayman under CIMA (the Mutual Funds Act and the Private Funds Law 2020, the latter in force **7 February 2020** ✅) and BVI under the FSC (the Securities and Investment Business Act 2010 with its five fund categories, and the Private Investment Funds Act 2019 for closed-ended funds) — plus economic-substance regimes that, as of 2026, do **not** reach a pure investment fund: Cayman's International Tax Co-operation (Economic Substance) Law **45 of 2018**, in force **1 January 2019**, excludes "investment funds" from the definition of relevant entity and excludes "investment fund business" from the relevant activities, while its "fund management business" head reaches only CIMA-licensed or otherwise authorised fund managers under the Securities Investment Business Law ✅ (verified at the statute itself, Section 5.5) — and the BVI's Economic Substance (Companies and Limited Partnerships) Act 2018, in force **1 January 2019** ✅ (Section 6.4).
- **The reputational overlay is live.** In the EU list of non-cooperative jurisdictions as updated **17 February 2026**, **Cayman appears on neither annex**, while the **BVI sits on Annex II — the "grey list" of jurisdictions that have committed to tax good-governance standards but are not yet fully compliant** ✅ (verified at the European Commission's EU-list page; Section 7.2).

The result is an architecture that is simultaneously the most standard thing in offshore funds and the most misunderstood: most of what looks like "Cayman/BVI fund formation" is really the *master-feeder packaging of US tax and ERISA constraints* around a trading entity that could, in principle, live anywhere.

### 1.2 What This Guide Covers

This guide is the repository's **offshore-structure deep-dive**, and it is deliberately positioned against its siblings:

1. **The architecture** — the master-feeder structure itself, the vehicles in the stack, and the aggregation/tax-isolation logic (Section 2).
2. **The tax drivers** — the US PFIC/QEF regime, UBTI and the blocker question, FATCA, ECI (Section 3) — verified at the statutes.
3. **The ERISA angle** — the plan-assets regulation and its 25% test (Section 4) — verified at the regulation.
4. **The jurisdictions** — Cayman in full (Section 5) and BVI in full (Section 6), each verified at primary sources (the statutes, CIMA, the FSC, the Cayman General Registry) and major law-firm guides (Conyers, Ogier, Harneys, Loeb Smith).
5. **The choice** — the Cayman-vs-BVI comparison table including the current EU-list status (Section 7), the fund-marketing context (Section 8, cross-referenced), and the onshore-vs-offshore question with the Singapore VCC as the onshore alternative (Section 9, cross-referenced).
6. **The bank's view** — the Cymbal Bank worked example of a Cayman master with Delaware and offshore feeders (Section 10).

What this guide does **not** re-derive: the fund economics (fees, carry, the J-curve), the Singapore MAS regime, the VCC mechanics, the 13O/13U/13D/13OA parameters, and the prime-brokerage worked-example mechanics all live in sibling guides and are cross-referenced by plain filename, per the repository convention.

### 1.3 The Key-Facts Table

| Aspect | Fact | Status |
| --- | --- | --- |
| The architecture | Investor-facing feeders (US taxable → Delaware LP; non-US/tax-exempt → Cayman or BVI feeder) subscribe into one master fund (Cayman or BVI) that holds the portfolio and trades | ✅ (law-firm guides; §2) |
| PFIC definition | IRC §1297 — foreign corporation meeting the 75% passive-income test or the 50% passive-asset test | ✅ (law.cornell.edu/uscode/text/26/1297) |
| Default PFIC tax | IRC §1291 — "excess distributions" and dispositions taxed with an interest charge on deferral | ✅ (law.cornell.edu/uscode/text/26/1291) |
| QEF current inclusion | IRC §1293 — current-year inclusion of the QEF's earnings for electing shareholders | ✅ (law.cornell.edu/uscode/text/26/1293) |
| QEF election | IRC §1295 — "qualified electing fund" defined; election per taxpayer; reported on IRS Form 8621 | ✅ (law.cornell.edu/uscode/text/26/1295; irs.gov) |
| Mark-to-market alternative | IRC §1296 — PFIC mark-to-market election | ✅ (irs.gov Form 8621 instructions 12/2025) |
| ERISA plan assets | 29 CFR §2510.3-101(a)(2) — look-through unless operating company or benefit-plan participation not "significant"; (f)(1) — significant = 25% or more of the value of any class of equity held by benefit plan investors | ✅ (law.cornell.edu/cfr/text/29/2510.3-101) |
| Cayman SPC | Segregated Portfolio Company — a form of exempted company; Companies Act Part XIV; introduced May 1998 | ✅ (Cayman General Registry; Ogier) |
| Cayman open-ended funds | Mutual Funds Act — CIMA registration/licensing (licensed, administered, registered categories; s4(4) exemptions) | ✅ (cima.ky Investment Funds page) |
| Cayman closed-ended funds | Private Funds Law 2020 — in force 7 February 2020 — CIMA registration for private funds | ✅ (CIMA; Loeb Smith) |
| Cayman economic substance | International Tax Co-operation (Economic Substance) Law 45 of 2018, in force 1 January 2019; "investment funds" excluded from relevant entity; "fund management business" limited to SIBL-authorised managers | ✅ (DITC statute text) |
| Cayman direct taxes | None — no income, corporate, or capital-gains tax on funds (the reason the ES regime exists) | ✅ (statute/law-firm guides; §5.1) |
| BVI open-ended funds | SIBA 2010 (Revised 2020) — five categories: incubator, approved, private, professional, public | ✅ (Harneys; BVI FSC guidelines) |
| BVI closed-ended funds | Private Investment Funds Act 2019 + PIF Regulations (Revised 2020) — PIF registration for closed-ended funds; ⚠ commencement date not re-verified live (widely documented as 1 January 2020) | ✅/⚠ (§6.3) |
| BVI economic substance | Economic Substance (Companies and Limited Partnerships) Act 2018, effective 1 January 2019 | ✅ (Harneys; bvifsc.vg) |
| EU list (17 Feb 2026) | Cayman: on neither annex. BVI: on Annex II (grey list, 9 jurisdictions incl. BVI) | ✅ (EC taxation-customs page) |
| Cymbal Bank persona | The repository's fictional bank; sole bank persona in this guide | ✅ (repo convention) |

### 1.4 Why This Matters to a Bank

For a bank serving the funds industry, the master-feeder structure is where most of the revenue actually sits:

- **Prime brokerage and custody attach to the master.** The master fund is the trading entity — it is the prime-brokerage client, the custody client, the margin borrower, and the FX counterparty. The feeders are cash-and-NAV vehicles by comparison.
- **Administration attaches to both layers.** The administrator computes the master NAV and the feeder NAVs, handles subscriptions/redemptions at the feeder level, and produces the investor reporting; a bank with a fund-administration franchise (or an administrator partnership) monetises both.
- **The KYC/AML burden is layered.** The bank must know the manager, the master, each feeder, and — through the look-through question — the investors; the sanctions-screening overlay (cross-ref [Fircosoft](fircosoft_guide.md)) runs across every layer of the structure.
- **The tax and ERISA constraints are what make the structure sticky.** Once a manager has built a Delaware feeder, a Cayman feeder, and a Cayman master with QEF elections in place and ERISA caps calibrated, switching jurisdictions or structures is expensive — which makes the client relationship durable.

The rest of this guide gives a banker the vocabulary and the verified facts to understand, price, and service that structure.

---

## 2. The Master-Feeder Architecture — What It Is and Why

### 2.1 The Two-Tier Structure

A master-feeder structure is two tiers of vehicles:

- **The feeders** are the investor-facing vehicles. Investors subscribe into a feeder; the feeder's shares/units/partnership interests are what the investor actually owns. Each feeder serves a defined investor population — the most important split being **US taxable investors** (in the **Delaware LP feeder**) versus **everyone else** (in the **offshore feeder**, Cayman or BVI).
- **The master** is the asset-holding and trading entity. The feeders invest their capital into the master; the master holds the portfolio, executes trades (typically through a prime broker), and produces the portfolio-level NAV from which the feeder NAVs are derived.

In the canonical global-manager stack:

```
                    ┌────────────────────────────────────────────┐
                    │              INVESTORS                     │
                    └───────┬────────────────────┬───────────────┘
                            │                    │
              US taxable    │                    │   Non-US / tax-exempt /
              investors     │                    │   ERISA plans (managed)
                            ▼                    ▼
              ┌─────────────────────┐  ┌──────────────────────┐
              │  DELAWARE LP FEEDER │  │  OFFSHORE FEEDER     │
              │  (US taxable box)   │  │  (Cayman/BVI,        │
              │  pass-through       │  │  non-US/tax-exempt)  │
              └──────────┬──────────┘  └──────────┬───────────┘
                         │  subscribes            │  subscribes
                         ▼                        ▼
              ┌───────────────────────────────────────────────┐
              │             MASTER FUND (Cayman/BVI)           │
              │  the trading entity — portfolio, PB, custody,  │
              │  margin, FX, NAV                               │
              └───────────────────────┬───────────────────────┘
                                      │
                                      ▼
                          Prime broker / custodians / venues
```

The conventions of this diagram follow the repository's worked-example style (see [Citadel LLC](citadel_llc_guide.md) §10 and [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §9); the *architecture* itself is the standard structure documented across the offshore law-firm literature — Conyers, Walkers, Ogier, Maples, Harneys, Mourant and Campbells all publish master-feeder guides with the same shape ✅ (§11 sources).

### 2.2 The Vehicles in the Stack

- **The US taxable feeder — a Delaware LP.** A Delaware limited partnership (or, for some managers, a Delaware LLC taxed as a partnership) is the standard US feeder. It is a pass-through for US federal income tax: the feeder's share of master income flows through to the US taxable partners, who report it on their own returns. The Delaware feeder typically makes the **QEF election** with respect to the master fund (Section 3.2), converting the master's PFIC treatment from the punitive §1291 default into current-year inclusion under §1293.
- **The offshore feeder — Cayman or BVI.** The non-US feeder is typically a Cayman exempted company (or SPC), a Cayman exempted limited partnership, or a BVI business company — tax neutral in its home jurisdiction, and the vehicle through which non-US investors, US tax-exempt investors (foundations, endowments, pension funds routed carefully), and US taxable investors who prefer offshore vehicles can invest.
- **The master — Cayman or BVI.** The master is most commonly a **Cayman exempted company** (often an **SPC** with segregated portfolios when the manager runs multiple strategies or investor classes under one umbrella), a Cayman exempted limited partnership, or a **BVI business company**. The master is a **PFIC** from the US perspective — which is precisely why the QEF machinery exists (Section 3).

Two structural refinements matter in practice:

- **A second US feeder.** Many managers run both a **Delaware LP feeder** and a **US master** (a Delaware LP or a US corporation) in addition to the offshore feeder — the "master-feeder" with a US master serving US taxable investors directly and the offshore feeder serving everyone else. The choice between "offshore master with Delaware feeder" and "US master plus offshore feeder" turns on where the manager wants the trading entity to be, the investor mix, and ERISA/UBTI considerations (Sections 3–4).
- **The SPC umbrella.** A Cayman SPC master can create a segregated portfolio per strategy or per investor class — statutory ring-fencing of assets and liabilities between portfolios within a single legal entity (Section 5.2). The BVI offers a comparable but less-used segregated-portfolio company variant under the BVI Business Companies Act ⚠ (BVI SPC details not re-verified this pass — Section 12).

### 2.3 Why the Structure Exists — Aggregation and Tax Isolation

The master-feeder structure solves four problems at once:

1. **Economies of scale.** One master = one portfolio, one trading book, one prime-brokerage relationship, one audit, one custody arrangement. A US$500 million master built from two US$250 million feeders gets institutional execution, lower fee tiers, better borrow, and one set of fixed costs — versus two standalone funds each paying for the same plumbing. This is the aggregation rationale, and it is the reason the structure pays for itself even before tax is considered ✅ (law-firm guides).
2. **Per-investor tax isolation.** Each feeder is a separate entity, so the tax character of one investor group does not leak into another's. The Delaware feeder's US taxable partners get QEF current inclusion on the master; the offshore feeder's non-US investors are outside the US tax net entirely (subject to FATCA reporting and any ECI/US-trade-or-business exposure — Section 3); US tax-exempt investors in the offshore feeder are managed against UBTI and the ERISA look-through (Sections 3.3, 4). The master never needs to know who the underlying investors are for US tax purposes — it just pays income up to the feeders.
3. **ERISA and plan-assets management.** Benefit-plan investors can be concentrated in a dedicated feeder whose plan participation is calibrated below the 25% threshold of 29 CFR §2510.3-101(f), so the master's assets are not plan assets (Section 4).
4. **Marketing and regulatory flexibility.** Different feeders can be marketed under different regimes (US private placement to US persons; EU/AIFMD channels for EU investors — Section 8), and a new investor jurisdiction can be added by creating a new feeder without disturbing the master.

### 2.4 How Money Moves — Subscriptions, Master Aggregation, NAV, Redemptions

- **Subscriptions.** An investor subscribes into a feeder (e.g., a US pension fund into the offshore feeder, a US high-net-worth individual into the Delaware feeder). The feeder issues its own interests; the cash is then subscribed into the master in exchange for master shares, so the master's capital base is the aggregate of all feeders' capital.
- **Aggregation and trading.** The master invests the aggregated capital according to the strategy, executing through the prime broker. All portfolio activity — positions, margin, financing, FX, corporate actions — is recorded at the master level.
- **NAV.** The administrator (Section 10) values the master's portfolio to produce the master NAV; each feeder's NAV is its proportional share of the master NAV, adjusted for feeder-level items (fees charged at the feeder level, feeder expenses, accrued performance fees). An investor's NAV per unit is the feeder NAV divided by the feeder's units outstanding.
- **Distributions and redemptions.** A redeeming investor requests redemption at the feeder; the feeder redeems proportionally from the master (or the master distributes to the feeder, which distributes to the investor). Timing follows the fund documents — monthly/quarterly dealing dates, notice periods, gates, and in-kind provisions are standard.

This cash flow is the same whether the master is Cayman or BVI; the jurisdiction differences (Sections 5–7) sit in the regulatory and substance overlay, not in the architecture.

### 2.5 Master-Feeder vs Standalone

A standalone fund (one vehicle holding the portfolio and taking investors directly) is simpler and cheaper to run — and is the right answer for a single-investor-class book, a small seed fund, or a fund whose investors are all in one tax box. The master-feeder pays for itself once any of the following is true:

- **Mixed US/non-US investor base** — the PFIC/QEF machinery is unmanageable inside a single vehicle serving both groups cleanly.
- **US tax-exempt or ERISA investors are expected** — the UBTI and plan-assets constraints (Sections 3.3, 4) need separate boxes.
- **Scale is the plan** — aggregation economics (Section 2.3) justify the extra layer.
- **Multi-strategy or multi-class complexity** — an SPC master (Section 5.2) ring-fences strategies within the trading entity while keeping one board, one administrator, and one auditor.

The decision is therefore never "Cayman vs BVI" in isolation — it is "which offshore jurisdiction hosts the master, and which feeder configuration serves the investor base", with the US tax and ERISA drivers of Sections 3–4 doing most of the work.

---

## 3. The Tax Drivers — PFIC, QEF, UBTI, FATCA, ECI

### 3.1 The PFIC Regime — IRC §1297, §1291, §1293, §1295

A Cayman or BVI fund is, from the US federal income-tax perspective, a **foreign corporation** — and a foreign corporation that earns passive income is a **Passive Foreign Investment Company (PFIC)**. The PFIC definition is in **IRC §1297**: a foreign corporation is a PFIC for a taxable year if either (i) **75% or more of its gross income is passive income** (the income test) or (ii) **at least 50% of the average value of its assets produces, or is held for the production of, passive income** (the asset test) ✅ (law.cornell.edu/uscode/text/26/1297). An investment fund that holds a securities portfolio is a PFIC as a matter of course — the asset test is met by any long-only or long/short book that is more than half invested in financial assets. This is not a planning accident; it is the premise on which the whole master-feeder architecture is built.

For a US person who owns PFIC stock, the Internal Revenue Code offers three regimes:

- **§1291 — the default.** If no election is made, the PFIC shareholder is taxed under **IRC §1291** on "excess distributions" and on gain from dispositions. An excess distribution is the portion of a distribution (or disposition gain) that exceeds 125% of the average distributions received in the prior three years; the excess is **allocated ratably over the shareholder's holding period** and taxed as if it had been distributed in each prior year, at the highest marginal rate applicable to that year, **plus an interest charge** on the deferred tax ✅ (law.cornell.edu/uscode/text/26/1291; uscode.house.gov — §1291(a)(1): the excess distribution "shall be allocated ratably to each day in the taxpayer's holding period"). The label attached to §1291 in the Code's own heading is "Interest on tax deferral" — the interest charge is the mechanism that strips out the time value of the deferral. For a hedge fund that distributes little and rolls gains, §1291 treatment is punitive — which is why every US taxable investor's tax counsel wants the QEF election (Section 3.2).
- **§1293 — current inclusion for a QEF.** If the PFIC is a **Qualified Electing Fund (QEF)** with respect to the shareholder, the shareholder includes currently in income its pro-rata share of the fund's **ordinary earnings and net capital gain** for each taxable year — current taxation instead of deferred-with-interest ✅ (law.cornell.edu/uscode/text/26/1293). Amounts already included (previously taxed income, "PTI") come out tax-free on later distributions.
- **§1296 — mark-to-market.** A shareholder of a PFIC whose stock is "marketable" may instead elect mark-to-market treatment under **IRC §1296**, including annual gains (and deducting losses, subject to a recapture rule) ✅ (irs.gov Form 8621 instructions 12/2025, which lists the §1296 election as reportable).

The election that makes §1293 apply is defined in **IRC §1295**: a "qualified electing fund" is a PFIC with respect to which an election under §1295(b) applies for the taxable year — an election made by the taxpayer, not by the fund ✅ (law.cornell.edu/uscode/text/26/1295). The fund must cooperate by supplying the information (earnings-and-profits and net-capital-gain schedules) that US shareholders need to compute their inclusions — which is why offshore funds' administrators produce **PFIC/QEF information packets** annually.

### 3.2 The QEF Election and the Practical Mechanics

The practical machinery, verified at the IRS:

- **Form 8621** ("Information Return by a Shareholder of a Passive Foreign Investment Company or Qualified Electing Fund") is the vehicle: a US person files it if they receive distributions from, or recognise gain on dispositions of, PFIC stock; report information with respect to a QEF or §1296 election; make an election reportable in Part II of the form; or are required to file under §1298(f) (the annual information-reporting requirement) ✅ (irs.gov "About Form 8621"; Instructions for Form 8621, 12/2025 revision).
- **The election is per-taxpayer and per-fund.** Each US investor in the Delaware feeder makes (or is deemed to make) the QEF election with respect to the master fund; the feeder itself, as a pass-through, does not absorb the election for the partners — the partners' share of the feeder's QEF inclusions flows through the Delaware LP to each partner's return.
- **The annual rhythm:** the fund's administrator computes the master's ordinary earnings and net capital gain; the feeder allocates its share to partners; each US partner includes its share currently (Form 8621 plus the income on the return) and tracks PTI for future distributions.
- **Late elections** are available in limited circumstances (the IRS publishes **Form 8621-A**, "Return by a Shareholder Making Certain Late Elections To End Treatment as a Passive Foreign Investment Company") ✅ (irs.gov forms list). ⚠ The exact eligibility conditions for late QEF elections were not re-verified this pass.

The QEF election is the hinge of the entire structure: without it, a US taxable investor in an offshore fund faces §1291's interest-charge regime; with it, the investor is taxed currently at ordinary/capital-gain rates on the fund's economics as they are earned — which is exactly what a US taxable investor in a Delaware feeder experiences.

### 3.3 UBTI and the Blockers for Tax-Exempt US Investors

**UBTI — unrelated business taxable income.** US tax-exempt investors (foundations, endowments, IRAs, and most retirement trusts) are generally exempt from income tax — but **IRC §511** imposes a tax, at corporate rates, on the **unrelated business taxable income** of tax-exempt organisations ✅ (law.cornell.edu/uscode/text/26/511; the Code's heading: "Imposition of tax on unrelated business income of charitable, etc., organizations"; §511(a) taxes "unrelated business taxable income (as defined in section 512)"). **§512** defines UBTI — and, critically, a tax-exempt partner's share of a **pass-through entity's** income retains its character as it flows through: if a tax-exempt investor holds a partnership interest in a fund that earns income from a business it regularly carries on, or **debt-financed income** (income from property acquired with borrowed funds — the leveraged-share-of-the-fund problem), that income can be UBTI to the exempt investor. ⚠ The debt-financed-income rules of **IRC §514** were not re-verified live this pass, but the leverage point is the standard one: an exempt investor in a leveraged hedge fund holding its interest through a *partnership* can be taxed on the debt-financed portion of the fund's income.

**The blockers.** The structural answer is the corporation:

- **A corporate fund is itself a blocker.** UBTI does not flow through a corporation. A US tax-exempt investor holding shares of a **Cayman exempted company** (the typical offshore feeder) receives distributions that are dividends, not UBTI — the corporate wrapper blocks the look-through. This is one reason the offshore feeder is a company rather than a partnership.
- **The dedicated C-corp blocker.** Where the fund itself is a partnership (e.g., a Delaware LP master serving tax-exempt investors), a **blocker C-corporation** is interposed: the exempt investors hold shares of the blocker, and the blocker holds the partnership interest, so the exempt investors never touch the fund's flow-through income. The blocker is a US C-corp, taxable on its own income — a real cost — which is why blockers are sized to the investor base and why "master as a corporation" is the cleaner structural answer in the Cayman/BVI context ✅ (standard structuring analysis per the law-firm literature).
- ⚠ The phrase **"25% blocker"** is industry shorthand, not a statutory term: it is used for blocker corporations sized or structured so that benefit-plan participation stays under the 25% plan-assets threshold of 29 CFR §2510.3-101(f) (Section 4), and also loosely for any C-corp blocker. The two ideas — the UBTI blocker and the ERISA 25% cap — are distinct; the guide keeps them separate.

### 3.4 FATCA and Withholding

**FATCA — the Foreign Account Tax Compliance Act — is Chapter 4 of the Code, IRC §§1471–1474** ✅ (law.cornell.edu/uscode/text/26/subtitle-A/chapter-4: "§1471. Withholdable payments to foreign financial institutions; §1472. Withholdable payments to other foreign entities; §1473. Definitions; §1474. Special rules"). The mechanism: a **withholding agent** that makes a **withholdable payment** (US-source interest, dividends, and gross proceeds, broadly) to a **foreign financial institution (FFI)** that has not entered the FATCA compliance regime must withhold **30%** ✅ (§1471(a), verified at law.cornell.edu/uscode/text/26/1471; irs.gov "Withholding under FATCA or 'Chapter 4' — IRC 1471-1474").

For the master-feeder architecture:

- **Offshore funds are FFIs.** A Cayman or BVI fund is a foreign financial institution for FATCA purposes; to avoid 30% withholding on its US-source income, it must register with the IRS and report its US accountholders (or rely on its jurisdiction's **intergovernmental agreement (IGA)** with the United States). Cayman and the BVI each operate under a Model 1 IGA under which funds report to their local tax authority, which exchanges with the IRS ⚠ (the IGA status of both jurisdictions is standard and widely documented, but was not re-extracted from the IRS FATCA page this pass).
- **The CRS parallel.** The OECD Common Reporting Standard — implemented in Cayman and the BVI — requires the same account-holder due diligence and reporting among participating jurisdictions; for a bank, FATCA/CRS classification is part of the investor on-boarding data model, not an afterthought (cross-ref the KYC overlay of Section 10.5).
- **Withholding on distributions.** Beyond FATCA, US-source fixed-or-determinable income paid to foreign persons is generally subject to US withholding under chapter 3 (IRC §1441 et seq.) ⚠ (§1441 not re-extracted this pass) — which is part of why offshore funds route US-market income with care (Section 3.5).

### 3.5 ECI — Effectively Connected Income

A foreign corporation is taxable in the United States on income **effectively connected with the conduct of a US trade or business (ECI)** — **IRC §882** taxes a foreign corporation engaged in a US trade or business on its ECI at the §11 rates ✅ (law.cornell.edu/uscode/text/26/882; irs.gov "Effectively connected income (ECI)"). The definitional machinery is in **IRC §864**: "trade or business within the United States" is defined, and — decisively for funds — **§864(b)(2) excludes "trading in stocks or securities"** from the definition, including **trading for the taxpayer's own account** (and trading through a "resident broker, commission agent, custodian, or other independent agent") ✅ (law.cornell.edu/uscode/text/26/864(b)(2)).

The consequence for the architecture: a Cayman or BVI **master** that trades securities for its own account — even through a US prime broker — is generally **not** engaged in a US trade or business by reason of that trading, so its US-market gains are not ECI ✅ (standard reading of §864(b)(2), verified against the statute). The lines that still need care:

- **Dealer activity** — a fund that acts as a dealer (underwriting, market-making, or trading for customers) loses the own-account shelter; funds are structured to avoid dealer status.
- **Income vs gains** — US-source dividends and interest paid to the fund can still face withholding (Section 3.4) even where gains are not ECI.
- **US real property** — gains from US real property interests (FIRPTA, IRC §897) ⚠ (not re-verified this pass) are a separate trap for funds with real-estate exposure.

---

## 4. The ERISA and Benefit-Plan-Investor Angle — the 25% Test

### 4.1 The Plan-Assets Regulation — 29 CFR §2510.3-101

The US Department of Labor's plan-assets regulation — **29 CFR §2510.3-101** — determines when the assets of an ERISA plan (and, by extension, other "benefit plan investors") include the underlying assets of an entity the plan invests in. The general rule, in **§2510.3-101(a)(2)**: when a plan invests in another entity, "the plan's assets include its investment, but do not, solely by reason of such investment, include any of the underlying assets of the entity" — **unless** the entity is neither a publicly-offered security nor a registered investment company, in which case the plan's assets "include both the equity interest and an undivided interest in each of the underlying assets of the entity" unless it is established that "(i) the entity is an operating company, or (ii) equity participation in the entity by benefit plan investors is not significant" ✅ (law.cornell.edu/cfr/text/29/2510.3-101, verified verbatim this pass).

The threshold that matters is paragraph **(f)**:

> (f)(1) Equity participation in an entity by benefit plan investors is "significant" on any date if, immediately after the most recent acquisition of any equity interest in the entity, **25 percent or more of the value of any class of equity interests in the entity is held by benefit plan investors** (as defined in paragraph (f)(2)). ✅ (29 CFR §2510.3-101(f)(1), verified verbatim at law.cornell.edu/cfr)

"Benefit plan investor" is defined in **(f)(2)** to include any employee benefit plan as defined in section 3(3) of ERISA — **whether or not subject to Title I** — plus the related Code §4975(e)(1) plans (IRAs, Keogh plans) and entities whose underlying assets include plan assets by reason of a plan's investment ✅/⚠ (the first category is verified verbatim; the full (f)(2) enumeration was only partially extracted this pass — Section 12). The regulation's own example (j)(2) is instructive: a plan acquiring 15% of a fund where a governmental plan already owns 15% triggers look-through, because benefit plan investors in the aggregate hold **more than 25%** — and the governmental plan's interest counts ✅ (verified at the example text).

### 4.2 What Triggers the Look-Through, and the Consequences

The trigger is mechanical: count the value of **all benefit plan investors'** holdings in **any class of equity** of the entity, immediately after each acquisition; if the aggregate is **≥25% of the class**, the look-through applies. Two features matter for fund structuring:

- **It is per entity.** The test is applied to each entity in the chain — the feeder, then the master. Plans holding 15% of the offshore feeder and 15% of the Delaware feeder can still trip the 25% test inside *each* feeder if other plan money is present.
- **It cascades.** If the feeder's equity is plan assets, the feeder's own assets — the master shares it holds — are plan assets of those plans; the analysis then repeats at the master level. A fund whose master is majority plan-owned runs the full consequence chain.

The consequences of look-through are the expensive part: any person exercising authority or control over the plan assets (the fund's GP, investment manager, and often the administrator) becomes an **ERISA fiduciary** with respect to those assets (§2510.3-101(a)(2) makes the fiduciary point explicit — "any person who exercises authority or control respecting the management or disposition of such underlying assets... is a fiduciary of the investing plan" ✅), subject to ERISA's fiduciary duties and the prohibited-transaction rules (ERISA §406 and Code §4975, the latter being the provision the regulation itself is promulgated under) ✅/⚠ (the prohibited-transaction citations are referenced within §2510.3-101(a)(1); the operative ERISA §406 text was not re-extracted this pass).

### 4.3 The Structuring Answers — Feeders, Caps, and Blockers

The standard answers, in order of preference:

1. **Cap benefit-plan participation.** Keep aggregate benefit-plan holdings in each feeder **below 25% of every class** — a documented structural cap enforced at subscription (the subscription agreement collects benefit-plan status, and the administrator monitors the cap at each dealing date). This is the cleanest answer and the most common ✅ (the 25% figure is the regulation's own, verified in (f)(1)).
2. **A dedicated benefit-plan feeder.** Concentrate plan investors in their own feeder with its own cap arithmetic, so the rest of the structure is untouched by the counting exercise.
3. **A blocker corporation.** Interpose a C-corp blocker (Section 3.3) so plans hold shares of a corporation; ⚠ note that a corporation is *not* automatically outside the plan-assets rule — the "operating company" and "significant participation" exceptions in (a)(2) apply to any entity — so the blocker only helps if plan participation in *it* is also managed, or if the blocker qualifies as an operating company.
4. **Registered or publicly-offered securities.** A fund whose interests are publicly offered or registered under the 1940 Act avoids look-through under (a)(2) — generally unavailable to offshore private funds.

The master-feeder architecture exists in part precisely to make answer (1) administrable: the manager counts plan participation per feeder rather than across a single shared vehicle.

---

## 5. The Cayman Jurisdiction

### 5.1 The No-Direct-Tax Profile and the Exempted Company

The Cayman Islands has **no income tax, no corporation tax, no capital-gains tax, and no withholding tax** — the profile that makes it the default offshore fund domicile. The position is stated plainly in the law-firm literature: "Exempted companies are not subject to any income, withholding or capital gains taxes in the Cayman Islands. Shareholders will not be subject to any income, withholding or capital gains taxes in the Cayman Islands with respect to their shares and dividends received on those shares, nor will they be subject to any estate or inheritance taxes in the Cayman Islands. There are no exchange controls in the Cayman Islands" ✅ (Ogier, "Cayman Islands exempted companies", June 2026). An exempted company may additionally apply under the **Tax Concessions Act (Revised)** for an undertaking that future Cayman tax legislation will not apply to it — available for up to 30 years, normally granted for 20 ✅ (same source).

The **exempted company** — incorporated under the **Companies Act (Revised)** — is the workhorse vehicle ✅ (Ogier; CIMA). For funds it has the decisive feature that it **may redeem and purchase its own shares**, so it can operate as an **open-ended** corporate fund, and it is a straightforward matter to convert between open- and closed-ended operation ✅ (cima.ky Investment Funds page). Its sibling vehicles — the **exempted limited partnership** (the default for US-style private equity), the **unit trust**, and the **SPC** (Section 5.2) — complete the Cayman fund toolkit ✅ (CIMA).

### 5.2 The SPC — Part XIV of the Companies Act

The **Segregated Portfolio Company (SPC)** is a form of exempted company under **Part XIV of the Companies Act** ✅ (Cayman Islands General Registry, official — "The Companies Law, PART XIV, provides for any exempted company, a company by way of continuation and an exempted limited duration company to re-register as a segregated portfolio company"). It was **first introduced in May 1998** by an amendment to the Companies Act ✅ (Ogier SPC briefing, June 2026; Conyers' "Silver Anniversary" note, July 2023, marking 25 years and the Grand Court's confirmation of strict asset/liability segregation). The current consolidated text of the Companies Act is maintained by the Cayman legislation revision process ⚠ (the precise revision year of the Companies Act as at August 2026 was not re-verified this pass — the 2021 Revision is referenced in the ES Act text; Section 12).

The mechanics, verified at the General Registry and Ogier:

- **One legal entity, many portfolios.** The SPC remains a single legal entity; each **segregated portfolio** is not a separate legal entity, but the assets and liabilities of each portfolio are **legally separate** from every other portfolio and from the SPC's general assets ✅ (General Registry; Ogier).
- **The segregation principle.** A portfolio's assets are available only to that portfolio's creditors and shareholders; a portfolio's liabilities extend first to that portfolio's assets and then, unless the articles prohibit it, to the SPC's general assets — but never to another portfolio's assets ✅ (Ogier, quoting the Companies Act's statutory prescription).
- **Name and administration.** The company's name must include "Segregated Portfolio Company" or "SPC"; an SPC files the standard exempted-company annual return plus a return of portfolio movements ✅ (General Registry). Registry fees verified at the General Registry: registration US$500-equivalent (KYD) plus the normal exempted-company fee; annual fee plus portfolio fees of KYD 300 per portfolio (max KYD 1,500) ✅ (ciregistry.ky fee table).

**Why funds use SPCs:** the master fund as an SPC can run one strategy per portfolio — or one investor class per portfolio — with statutory ring-fencing, under one board, one administrator, and one auditor. The master-feeder diagram of Section 2.1 generalises: a master SPC whose portfolios feed separate strategy books, with the feeders subscribing into the relevant portfolios ✅ (standard use documented across Ogier/Conyers; the CIMA page lists the SPC among the four common fund vehicles — exempted company, SPC, unit trust, exempted limited partnership ✅).

### 5.3 CIMA and the Mutual Funds Law — Open-Ended Funds

**CIMA — the Cayman Islands Monetary Authority — regulates funds and fund administrators** under the **Mutual Funds Act (MFL)** ✅ (cima.ky Investment Funds page, verified this pass). The statutory definition of a **mutual fund**: "any company, trust or partnership either incorporated or established in the Cayman Islands, or if outside the Cayman Islands, managed from the Cayman Islands, which issues equity interest redeemable or repurchaseable at the option of the investor, the purpose of which is the pooling of investors' funds with the aim of spreading investment risk and enabling investors to receive profits or gains from investments" ✅ (cima.ky, quoting the MFL). "Funds commonly referred to as hedge funds fall within the definition" ✅ (same source).

The MFL's regulatory categories, verified at CIMA:

- **Licensed mutual fund** — MFL s4(1): a mutual fund operating in and from Cayman must be licensed unless it is administered, registered, or exempt; the licence route serves large, well-known institutions that do not propose to appoint Cayman service providers ✅.
- **Administered mutual fund** — the fund's principal office is provided by a **CIMA-licensed mutual fund administrator**; the administrator carries most of the regulatory responsibility; this route covers funds with **more than 15 investors** that are neither licensed nor registered ✅.
- **Registered mutual fund** — the default for hedge funds: requires either a **minimum aggregate equity interest of CI$80,000 (US$100,000) purchasable by a prospective investor**, or listing on a CIMA-approved stock exchange; a **master fund** must meet the same US$100,000-aggregate test or be listed ✅ (cima.ky — the page explicitly addresses master funds).
- **Exempt funds** — funds meeting the criteria in **MFL s4(4)** are exempt from regulation ✅ (cima.ky confirms the s4(4) exemption exists; ⚠ the specific s4(4) limbs — the classic "15 or fewer investors" and "all sophisticated investors (min US$100,000 initial investment)" limbs — were not re-extracted from the statute this pass; they are standard and widely documented, Section 12).

The four common fund vehicles under the MFL regime: **exempted company, SPC, unit trust, exempted limited partnership** ✅ (cima.ky). Fund administrators are licensed in two tiers — **full administrators** (unlimited funds) and **restricted administrators** (up to 10 funds, each requiring CIMA approval) ✅ (cima.ky).

### 5.4 The Private Funds Law 2020 — Closed-Ended Funds

The **Private Funds Law, 2020** — in force **7 February 2020** — extended CIMA registration to **closed-ended** funds ("private funds"), closing the gap left by the MFL's open-ended focus ✅ (CIMA Private Funds Law 2020 FAQs; Loeb Smith, 13 February 2020; Pillsbury). Verified detail:

- **Definition.** A private fund is a company, unit trust or partnership whose principal business is offering and issuing investment interests to pool investor funds and spread risk, where holders lack day-to-day control and the investments are managed as a whole by or on behalf of the operator for reward ✅ (Loeb Smith, quoting the Law).
- **Exclusions.** Securitisation SPVs, joint ventures, proprietary vehicles, holding vehicles, preferred-equity financing vehicles, sovereign wealth funds, structured finance vehicles and single family offices are listed as non-fund arrangements; single-investor funds are outside scope ✅ (Loeb Smith).
- **Registration mechanics.** A private fund must submit its registration application to CIMA **within 21 days of accepting capital commitments** and be **registered before accepting capital contributions**; existing funds had until **7 August 2020** (the transitional window) ✅ (Loeb Smith, citing s5 of the Law).
- **Ongoing requirements.** Audit, valuation of assets, safekeeping of fund assets, cash monitoring and identification of securities — the same discipline as the MFL regime ✅ (Loeb Smith). A "restricted scope private fund" category exists for exempted limited partnerships managed by a CIMA-licensed or recognised-overseas-regulated manager with all non-retail investors ✅ (Loeb Smith).

For the master-feeder guide, the PFL matters in two places: a **closed-ended** master (private equity/credit) must be registered under the PFL, and the PFL's audit/valuation/safekeeping requirements are exactly the disciplines a bank's fund-administration franchise sells (Section 10).

### 5.5 The Economic Substance Regime

The Cayman economic substance regime is the **International Tax Co-operation (Economic Substance) Law, 2018 — Law 45 of 2018 — in force 1 January 2019** ✅ (the statute itself, DITC consolidated text: "Originally enacted — Law 45 of 2018 — 17th December, 2018" with commencement "1-Jan-2019" per the legislation endnotes; Conyers, January 2019; Lexology). Note the precise position the brief asked to be checked: **the law is dated 2018 and in force from 1 January 2019** — it is not a 2019 law ✅. The current consolidated text is the 2021 Revision, amended by Laws 7 of 2020 and 56 of 2020 and four sets of Amendment of Schedule Regulations (2019 ×3, 2020) ✅ (DITC PDF).

The scope, verified at the statute:

- **Relevant entity.** The economic substance test applies to a "relevant entity" — and the definition **excludes "(i) an investment fund; or (ii) an entity that is tax resident outside the Islands"** ✅ (Schedule, verified verbatim). An "investment fund" is defined as an entity whose principal business is issuing investment interests to raise or pool investor funds so holders benefit from the entity's investment gains ✅ (Schedule, verified verbatim).
- **Relevant activities.** The nine relevant activities: banking business; distribution and service centre business; financing and leasing business; **fund management business**; headquarters business; holding company business; insurance business; intellectual property business; shipping business — and the definition **"does not include investment fund business"** ✅ (Schedule, verified verbatim).
- **Fund management business — the nuance.** The Schedule defines "fund management business" as "the business of managing securities as set out in paragraph 3 of Schedule 2 to the Securities Investment Business Act (2020 Revision) carried on by a relevant entity **licensed or otherwise authorised to conduct business under that Act for an investment fund**" ✅ (verified verbatim). So: **pure investment funds — including pure-equity funds and private funds — are outside the ES regime entirely** (they are not relevant entities and fund business is not a relevant activity), while **SIBL-licensed or authorised fund managers** carrying on fund management for an investment fund are in scope and must satisfy the test. ⚠ The boundary for managers operating under other authorisations (e.g., mutual fund administrators) is fact-specific — flagged in Section 12.
- **The test.** A relevant entity must conduct its **core income generating activities (CIGAs)** in the Islands, be **directed and managed** in the Islands (board meetings with quorum present, minutes kept in the Islands), and have **adequate** operating expenditure, physical presence and full-time employees in the Islands — with a **reduced test for pure equity holding companies** (compliance with Companies Act filings plus adequate resources and premises) ✅ (ss4(2)–4(5), verified verbatim). For fund management business, the CIGAs are taking decisions on holding and selling investments, calculating risk and reserves, taking hedging decisions, and preparing reports to investors or CIMA ✅ (Schedule, verified).
- **Reporting and penalties.** Annual notification of whether a relevant activity is carried on; an economic substance return within 12 months of the financial year-end for in-scope entities; penalties of **CI$10,000** for a first failure to satisfy the test, **CI$100,000** for a subsequent year, and **CI$5,000 plus CI$500 per day** for late filing ✅ (ss7–8, verified verbatim).

For the master-feeder architecture the practical position is clean: the **master and the feeders, being investment funds, file the annual notification and confirm they are not carrying on a relevant activity** — the ES regime is an administrative layer, not a substance burden, for a pure fund ⚠ (the point that fund *managers* with real Cayman presence must satisfy the test — and that managers who are SIBL-licensed in Cayman face the CIGA requirements — is the live one; see Section 7 for the comparison).

---

## 6. The BVI Jurisdiction

### 6.1 The BVI Business Company and the FSC

The **British Virgin Islands** runs the second-largest offshore fund industry in the world on a deliberately light regulatory scaffold. The corporate vehicle is the **BVI business company**, incorporated under the **BVI Business Companies Act, 2004 (Act 16 of 2004, in force 1 January 2005)** ✅ (bvifsc.vg statute text). Funds and financial-services businesses are regulated by the **BVI Financial Services Commission (FSC)** under the **Securities and Investment Business Act, 2010 (SIBA)** — the current consolidated text being SIBA Revised Edition 2020 ✅ (Harneys BVI funds guides, verified this pass).

Two vehicle features matter for funds:

- **Flexibility of form.** BVI funds can be structured as **companies** (the most common), **limited partnerships**, or **unit trusts** ✅ (Harneys).
- **The BVI SPC.** The BVI Business Companies Act provides for **segregated portfolio companies** under section 159, with the **Segregated Portfolio Companies Regulations, 2005** (gazetted 22 December 2005) ✅ (bvifsc.vg regulations text; Carey Olsen). A BVI SPC can be incorporated as such or registered by an existing company ✅ (Carey Olsen). ⚠ The BVI SPC is much less used than the Cayman SPC for master funds; the comparative take-up is directional, and detailed BVI SPC mechanics were not re-verified this pass (Section 12).

**Tax neutrality** is statutory in effect: "BVI funds are not subject to income tax, capital gains tax, withholding tax or stamp duty in the BVI, allowing returns to flow to investors without an additional layer of jurisdiction-level taxation" ✅ (Harneys, verified verbatim). There is no BVI capital duty on share issues and no VAT on fund services ⚠ (the latter two points are standard service-provider claims, not re-verified this pass).

### 6.2 SIBA 2010 — the Five Fund Categories

Under **SIBA**, investment funds fall into **five recognised categories** ✅ (Harneys; BVI FSC SIBA Incubator and Approved Funds Guidelines). The categories, with the verified parameters:

| Category | Investors / limits | Minimum investment | Validity | Key obligations | Approval model |
| --- | --- | --- | --- | --- | --- |
| **Incubator fund** | ≤ 20 investors; net assets ≤ US$20M | US$20,000 per investor | 2 years (+ up to 12-month extension), then must convert or wind up | Very light; no audit required | FSC approval; can start 2 business days after a complete application; US$1,500 application fee |
| **Approved fund** | ≤ 20 investors; net assets ≤ US$100M | None | Unlimited | Administrator required; no manager/custodian/auditor required | FSC approval; fast-track, 2 business days |
| **Private fund** | ≤ 50 investors; private invitation only | None | Unlimited | Lightest of the main three | Recognition by filing + fee (no prior FSC approval) |
| **Professional fund** | Professional investors only | US$100,000 (unless exempted investor) | Unlimited | Licensed/recognised investment manager + auditor required | Recognition by filing + fee |
| **Public fund** | No investor restrictions (retail) | n/a | n/a | Prospectus per SIBA and the Public Funds Code; FSC registration before business | Full registration |

Verified sources for the table: Harneys ("BVI Open-Ended Funds and Approved Funds", verified this pass — the five categories, the investor caps, the net-asset caps, the US$20,000 incubator minimum, the two-business-day fast track for incubator and approved funds, the unlimited validity of approved funds, the recognition-by-filing model for private and professional funds, and the public-fund prospectus requirement) ✅; the **BVI FSC "SIBA Incubator and Approved Funds Guidelines"** (issued 28 May 2015, published 4 June 2015, commenced 1 June 2015, under the **Securities and Investment Business (Incubator and Approved Funds) Regulations, 2015**, gazetted 18 May 2015 — including the US$1,500 application fee and the two-business-day processing standard) ✅; the Loeb Smith conversion guide (incubator funds converting to private/professional/approved on triggering events) ✅.

The positioning is explicit in the FSC's own guidance: the incubator and approved categories were created "to provide more flexibility to smaller and start-up financial services businesses", letting "smaller funds that may not typically qualify as private or professional funds" operate under a lighter framework ✅ (FSC Guidelines, verified verbatim). For master-feeder purposes, the BVI's natural home is the **professional fund** (the classic BVI hedge fund for institutional money) and the **incubator/approved** ladder for emerging managers — the structure that the Singapore sibling guide's "Why Global Managers Still Go Offshore" section (cross-ref [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §3.2) describes from the manager's side.

### 6.3 The Private Investment Funds Act 2019 — Closed-Ended Funds

Closed-ended funds in the BVI are regulated as **Private Investment Funds (PIFs)**. The governing framework is the **Private Investment Funds Act 2019** together with the **Private Investment Funds Regulations** (Revised Edition 2020); Harneys' 2026 guide names the operative legislation as SIBA (Revised Edition 2020) and the PIF Regulations (Revised Edition 2020) ✅ (Harneys "Establishing a Closed-Ended Fund in the BVI", verified this pass). ⚠ The precise commencement date of the Private Investment Funds Act 2019 (widely documented as **1 January 2020**, with existing closed-ended funds required to register by **7 January 2020** in the FSC's transitional arrangements) was not re-verified live this pass — the Act's year (2019) and the Revised 2020 regulations are verified; the commencement date is flagged in Section 12.

The PIF regime, verified at Harneys:

- **The definition.** An entity is a PIF (and must be regulated by the FSC as such) if it **collects and pools investor funds for collective investment and diversification of portfolio risk** and issues interests entitling holders to amounts calculated by reference to the value of a proportionate interest in the fund's net assets ✅.
- **Exclusions.** Single-investor funds, single-asset funds, joint venture companies and special purpose acquisition companies (SPACs) do not require PIF regulation ✅.
- **Distribution modes.** Interests may be distributed on a **"private" basis** — restricted to **no more than 50 investors** or to private invitations only, with **no minimum investment** — or on a **"professional" basis** — available only to **professional investors** with a **minimum initial investment of US$100,000**, unless the investor is an "exempted investor" (the manager, administrator, promoter, underwriter, or the manager's employees) ✅.
- **Professional investor** means a person whose ordinary business involves acquiring or disposing of property of the same kind as the fund's, or whose net worth (individually or with a spouse) **exceeds US$1,000,000, including the primary residence** ✅.
- **Ongoing requirements.** A PIF must issue an offering document or term sheet (exemption possible), maintain a **clear and comprehensive valuation policy** for its assets, and — per the wider regime — appoint an auditor and comply with the PIF Regulations ✅.

For the master-feeder guide, the PIF route is the BVI answer for **closed-ended masters and feeders** (private equity, credit, real estate) — the direct counterpart of the Cayman Private Funds Law 2020 (Section 5.4), with the same "registration, not licensing" philosophy.

### 6.4 The BVI Economic Substance Regime

The BVI economic substance regime is the **Economic Substance (Companies and Limited Partnerships) Act, 2018 — in force 1 January 2019** ✅ (Harneys ES guide, verified this pass: "The Economic Substance (Companies and Limited Partnerships) Act, 2018 (the Act) was introduced in the BVI, effective 1 January 2019"; bvifsc.vg consolidated text as revised to 1 January 2020). Note again the precise dating: **the Act is a 2018 Act, effective from 1 January 2019** — the same pattern as Cayman ✅. The regime is administered by the **International Tax Authority (ITA)**; its interpretative **Economic Substance Code** (the "Rules") was issued in draft April 2019, published October 2019, and updated February 2020 and February 2023 (version 3) ✅ (Harneys). Reporting runs through the **BOSS (Beneficial Ownership Secure Search) system** via the registered agent, within six months of the financial period end ✅.

The scope, verified at the Harneys guide:

- **Relevant activities.** Nine categories — and the Act **"expressly confirms that investment fund business is not [a] relevant activity"** ✅. "Investment fund business" is the business of operating an **investment fund** (defined, as in Cayman, as an entity whose principal business is issuing investment interests to raise or pool investor funds so holders benefit from the investment gains, including entities through which the fund invests, but not the ultimate investment held) ✅ (Harneys, quoting the Act).
- **The fund-management nuance.** The first three relevant activities — banking, insurance, fund management — "are defined by reference to existing regulatory licensing regimes": an entity carrying on **fund management business** "will (or, to be accurate, should) already hold a licence issued by the [FSC]" under SIBA, and "if the activity is outside the scope of the regulatory regime, then it will not be required to meet the economic substance requirement" ✅ (Harneys, verified verbatim). So, as in Cayman: **pure funds are out of scope; FSC-licensed fund managers carrying on fund management business are in scope**, and their CIGAs are taking decisions on holding and selling investments, calculating risks and reserves, hedging decisions, and preparing regulatory/investor reports ✅ (Harneys Schedule 1, verified).
- **The reduced test for pure equity holding entities** — an entity carrying on only "holding business" faces a much reduced requirement ✅ (Harneys).
- **Tax-resident-outside carve-out.** Entities tax resident outside the BVI (in a jurisdiction not itself on the EU blacklist) are not subject to the substance requirements but must still determine whether they carry on relevant activities and file returns ✅ (Harneys).
- **Grandfathering.** Entities existing before 1 January 2019 had a grandfathering period ending **30 June 2019** ✅ (Harneys).

### 6.5 The Cost and Speed Profile

The BVI's pitch is speed and economy, verified at the FSC's own materials and Harneys:

- **Incorporation:** a BVI business company can be incorporated "within one to two business days" ✅ (Harneys).
- **Fund approval:** incubator and approved funds can commence business **two business days after submitting a complete application** ✅ (FSC Guidelines; Harneys); private and professional funds are deemed recognised on filing plus fee ✅ (Harneys).
- **Application fee:** US$1,500 for incubator/approved applications ✅ (FSC Guidelines form checklist).
- **Ongoing costs:** annual government fees on the BVI business company plus FSC fees and registered-agent fees; the precise 2026 fee schedule was not re-verified this pass ⚠ (Section 12).

---

## 7. The Comparison Table — Cayman vs BVI

### 7.1 The Row-by-Row Comparison

| Dimension | Cayman Islands | BVI | Status |
| --- | --- | --- | --- |
| **Fund regulation (open-ended)** | Mutual Funds Act — CIMA registration/licensing; registered funds need US$100,000 aggregate minimum or an approved-exchange listing; licensed/administered/exempt routes | SIBA 2010 — five categories (incubator, approved, private, professional, public); recognition by filing for private/professional | ✅ both (CIMA; FSC/Harneys) |
| **Fund regulation (closed-ended)** | Private Funds Law 2020 (in force 7 Feb 2020) — CIMA registration within 21 days of commitments | Private Investment Funds Act 2019 + PIF Regulations (Revised 2020) — FSC PIF registration | ✅ Cayman; ⚠ BVI commencement date (widely documented as 1 Jan 2020) |
| **Speed of formation** | Exempted company incorporation typically within a day; mutual-fund registration is a filing exercise | Incorporation 1–2 business days; incubator/approved funds can start 2 business days after a complete application | ✅ BVI numbers verified at FSC/Harneys; ⚠ Cayman day-one claims directional |
| **Cost** | Government fees + CIMA registration/annual fees; SPC portfolio fees KYD 300/portfolio (max KYD 1,500) verified | US$1,500 incubator/approved application fee verified; full 2026 schedules ⚠ | ✅/⚠ |
| **Economic substance** | ES Law 45 of 2018 (in force 1 Jan 2019); investment funds excluded; fund management business = SIBL-authorised managers | ES Act 2018 (in force 1 Jan 2019); investment fund business expressly not a relevant activity; fund management = FSC-licensed managers | ✅ both (statutes/guides) |
| **Vehicle depth** | Exempted company, SPC (Part XIV, 1998), ELP, unit trust — SPC is the global default for master funds | BVI business company, LP, unit trust, SPC (BCA s159, 2005) — SPC use is rare | ✅/⚠ |
| **Administrator availability** | Deep: CIMA-licensed full/restricted administrators; global administrators headquartered/strong in Cayman | Strong but thinner than Cayman; administrator required for approved funds | ✅ Cayman tiers verified at CIMA; ⚠ BVI depth directional |
| **Investor preference** | The default for US-facing hedge funds and most global managers; US taxable investors' counsel comfortable with Cayman masters + QEF | The default for Asian and emerging-manager structures; incubator/approved ladder for sub-US$100M launches; historic PE favourite | ✅ directional per law-firm guides |
| **Fund-type fit** | Everything: hedge (registered mutual fund), PE/credit (PFL), SPC umbrellas, master-feeder | Hedge (professional/approved/incubator), PE/credit (PIF), emerging-manager launches | ✅ |
| **EU-list standing (17 Feb 2026)** | On **neither** annex | On **Annex II** (grey list) — committed, not yet fully compliant | ✅ (EC page; §7.2) |
| **Reputational texture** | Grey-list history (Feb–Oct 2020) resolved; ES regime passed EU/OECD scrutiny | Grey-listed since 2023 with outstanding commitments; EU next review October 2026 | ✅/⚠ (§7.2) |

### 7.2 Reputational and Regulatory Standing — the EU List in 2026

The **EU list of non-cooperative jurisdictions for tax purposes** is updated twice a year by the Council (ECOFIN), with the most recent update on **17 February 2026** ✅ (European Commission "Common EU list of third country jurisdictions for tax purposes", situation on 17 February 2026; Arendt, 18 February 2026; Wikipedia citing Council materials). The verified position as at that update:

- **Annex I (the "blacklist") — 10 jurisdictions:** American Samoa, Anguilla, Guam, Palau, Panama, the Russian Federation, Turks and Caicos, US Virgin Islands, Vanuatu and Viet Nam ✅ (EC page). **Neither the Cayman Islands nor the BVI is on Annex I.**
- **Annex II (the "grey list" — state of play for jurisdictions with outstanding commitments) — 9 jurisdictions:** Belize, **the British Virgin Islands**, Brunei Darussalam, Eswatini, Greenland, Jordan, Montenegro, Morocco and Türkiye ✅ (EC page). **The BVI is therefore currently on the grey list**, while **Cayman is on neither annex** ✅.

The history that produced this: the BVI was **added to Annex I in February 2023** and **moved to the grey list in October 2023** on the strength of commitments (Costa Rica and the Marshall Islands moved in the same cycle) ✅ (Council press release of 14 February 2023, via Wikipedia's citation; Brussels Times, 17 October 2023). Cayman was on the grey list from **February 2020 to October 2020** and has been off the list since, following the passage of its economic substance legislation ✅ (Wikipedia history section). ⚠ Two caveats, flagged honestly: (i) the *reasons* for the BVI's continuing Annex II listing (chiefly outstanding OECD Global Forum and substance-enforcement follow-ups) rest on the EC's deficiency table and press coverage that were not fully re-extracted this pass; and (ii) the list is dynamic — the **next revision is due in October 2026** ✅ (Arendt), after this guide's August 2026 date. The practical consequence for fund structuring: grey-listing is not blacklisting — the defensive measures (non-deductibility, CFC rules, higher withholding) attach to Annex I — but it is a due-diligence data point that Asian and EU allocators increasingly ask about ⚠ (directional).

---

## 8. The Fund-Marketing Context — AIFMD and the NPPRs

### 8.1 AIFMD and Non-EU Managers (Cross-Referenced)

The marketing overlay for a Cayman/BVI fund into the EU is the **Alternative Investment Fund Managers Directive (AIFMD)** — **Directive 2011/61/EU** — and its successor package **AIFMD II (Directive (EU) 2024/927)**. The full mechanics — the AIFM licence, the depositary regime, the reporting, and the AIFMD II changes — are derived in the sibling [Private Equity](private_equity_guide.md) guide §7 and are **not re-derived here** ✅ (cross-ref). For this guide's purposes, the shape is: a Cayman/BVI fund marketed to EU investors is an **AIF**; its **non-EU manager** cannot rely on the EU passport and must either market through the **national private placement regimes (NPPRs)** of each member state or, where the fund is EU-domiciled, appoint an EU AIFM. The offshore fund therefore faces a **per-country marketing patchwork**, which is a large part of why EU-domiciled funds (Luxembourg, Ireland) exist at all — and why the offshore-vs-onshore decision (Section 9) is never purely about the fund vehicle itself.

### 8.2 The NPPRs in Practice

Under **Article 42 of AIFMD**, member states may allow non-EU AIFMs to market non-EU AIFs in their territory under national rules ✅/⚠ (Article 42 is the NPPR provision; the article-level cite is standard and cross-referenced from the PE guide, not re-extracted this pass). In practice:

- **Each member state runs its own regime** — notification, investor-type restrictions (professional investors only in most), and local requirements vary; a pan-EU marketing campaign for an offshore fund means a country-by-country compliance exercise ⚠ (directional; the PE guide §7 holds the detail).
- **The UK sits outside AIFMD post-Brexit** and runs its own private-placement/regulatory regime for offshore funds ⚠ (not re-derived here).
- **AIFMD II** extends the framework's reach (loan-origination funds, delegation oversight) but keeps the NPPR architecture for non-EU managers ✅/⚠ (cross-ref PE guide §7).

For the master-feeder guide the marketing layer matters as a *structuring input*: an EU investor base can justify an EU feeder (Luxembourg SICAV or Irish ICAV) subscribing into the same Cayman/BVI master — a third feeder in the diagram of Section 2.1 — so the architecture's "add a feeder per investor jurisdiction" logic extends naturally to the EU.

---

## 9. The Onshore-vs-Offshore Choice — the Singapore VCC Alternative

### 9.1 The VCC and the 13O/13U Incentives (Cross-Referenced)

The onshore challenger to the Cayman/BVI master-feeder is the **Singapore Variable Capital Company (VCC)** — created by the **Variable Capital Companies Act 2018**, launched 14 January 2020, administered jointly by MAS and ACRA — combined with the **13O/13U/13D/13OA fund tax incentives** of the Income Tax Act 1947. The full mechanics — the VCC umbrella structure, the 13O (S$5M minimum AUM, two investment professionals, tiered local business spending) and 13U (S$50M, three professionals) parameters, the revised criteria effective 1 January 2025, and the extension of the schemes to 31 December 2029 — are derived in the siblings and **not re-derived here** ✅ (cross-ref [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §3.1–3.2 and [Private Equity](private_equity_guide.md) §8.1–8.2). The MAS fund-management overlay (LFMC licensing, base capital, notices) is likewise cross-referenced (the MAS guide).

### 9.2 When Offshore Wins, and When Onshore Does

The honest decision rule, synthesising the siblings' "Why Global Managers Still Go Offshore" analysis (cross-ref hedge guide §3.2):

**Offshore (Cayman/BVI) wins when:**

- **US taxable investors are in the mix.** The Delaware-feeder-plus-offshore-master stack with QEF elections (Sections 2–3) is a US-tax architecture; Singapore vehicles do not replicate the US taxable investor experience (a VCC is itself a PFIC for US purposes, and US taxable investors still need the Delaware feeder — at which point the offshore master is usually already in the diagram).
- **ERISA/UBTI-sensitive investors are expected** — the plan-assets caps and the corporate-blocker logic (Sections 3.3, 4) are built into the offshore stack.
- **Global multi-jurisdiction fundraising is the plan** — EU feeders, US feeders, Asia feeders around one master; investor familiarity with Cayman/BVI documentation.
- **The manager is US-headquartered or US-raise-dependent** — their LPs, counsel and administrators expect the standard offshore architecture.

**Onshore (Singapore VCC) wins when:**

- **Asia is the centre of gravity** — Singapore-managed, Asia-sourced capital; the MAS-regulated ecosystem; the 13O/13U tax incentives make the VCC tax-transparent-ish for qualifying income; the gateway AUM pattern (roughly 76% of Singapore-managed AUM sourced from outside Singapore, ~88% invested outside — cross-ref hedge guide §1.1) ✅ (cross-ref).
- **The investors are non-US tax-exempt or Asian institutions** who prefer an MAS-regulated onshore wrapper over an offshore shell.
- **Family-office and private-wealth capital** is the base — the SFO exemption and 13O route (cross-ref hedge guide §7) are Singapore-specific advantages.
- **The fund is closed-ended and Asia-focused** (private credit, real estate, venture) where the VCC/13U wrapper is now standard.

The pragmatic pattern in the market is **hybrid**: an Asia-focused VCC or a BVI/Cayman fund with a Singapore manager, or a VCC feeder subscribing into an offshore master. The offshore architecture of this guide remains the spine — the VCC question is which feeders and which manager sit around it ✅/⚠ (the hybrid-pattern claim is directional industry observation, flagged).

---

## 10. The Worked Example — A Cayman Master with Delaware and Offshore Feeders as a Cymbal Bank Client

### 10.1 The Scenario and the Structure Diagram

**The scenario.** A global long/short equity manager — the fictional firm "Meridian Point Capital", consistent with the repository's worked-example convention of fictional client names around the Cymbal Bank persona — is launching its first institutional fund with a **US$500 million target**. The pipeline splits three ways: **US taxable investors** (a US family office, US high-net-worth individuals, a US taxable institution) who need the Delaware pass-through; **non-US institutions** (a Swiss pension fund, a Japanese insurance company, a sovereign-wealth window) who must never touch US tax concepts; and **US tax-exempt investors** (a US foundation, a university endowment) who need the corporate wrapper against UBTI. Counsel specifies the canonical stack: a **Cayman exempted company master** (the trading entity), a **Delaware LP feeder** for US taxable investors, and a **Cayman exempted company offshore feeder** for non-US and US tax-exempt investors — the architecture of Section 2.1 in concrete form, and exactly the structure this guide's title promises.

The structure, in text mode (diagram conventions follow §2.1 and the repository's worked-example style in [Citadel LLC](citadel_llc_guide.md) §10 and [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §9):

```
                    ┌──────────────────────────────────────────────────┐
                    │                    INVESTORS                     │
                    │  US taxable · non-US · US tax-exempt (ERISA)     │
                    └───────┬───────────────────────────┬──────────────┘
                            │                           │
              US taxable    │            non-US and US   │  tax-exempt
              investors     │            tax-exempt      │  investors
                            ▼                           ▼
        ┌────────────────────────────┐      ┌──────────────────────────┐
        │     DELAWARE LP FEEDER     │      │  CAYMAN OFFSHORE FEEDER  │
        │  (US taxable box;          │      │  (exempted company;      │
        │   pass-through; QEF        │      │   corporate UBTI blocker │
        │   election on the master)  │      │   for tax-exempt US LPs) │
        └─────────────┬──────────────┘      └────────────┬─────────────┘
                      │  subscribes                     │  subscribes
                      │  (master shares)                │  (master shares)
                      ▼                                 ▼
        ┌──────────────────────────────────────────────────────────────┐
        │            CAYMAN MASTER FUND (exempted company)             │
        │  the trading entity — portfolio, PB margin, custody, FX,     │
        │  financing; one master NAV from which both feeder NAVs       │
        │  derive                                                      │
        └──────────────────────────────┬───────────────────────────────┘
                                       │
                                       ▼
        Cymbal Bank — prime brokerage · custody · fund administration ·
        cash management / FX · the KYC/AML overlay
```

Two structural notes, both already derived earlier in the guide: the **ERISA cap arithmetic of Section 4.3** lives at the two feeder boxes — the manager counts benefit-plan participation per feeder after each acquisition and keeps it below 25% of every class, so the master's assets are never plan assets; and the **registered-mutual-fund route** the master uses requires a US$100,000 aggregate minimum purchasable by a prospective investor (or a listing on an approved exchange), which the master fund must itself satisfy ✅ (cima.ky; §5.3).

### 10.2 The Flow of Funds — Subscriptions to Distributions

Walk the money around the loop (the mechanics of Section 2.4, in the client's numbers):

- **Subscriptions at the feeder level.** Each investor subscribes into its designated feeder: a US$10 million commitment from the US family office into the Delaware LP; a €25 million commitment from the Swiss pension fund and a ¥5 billion commitment from the Japanese insurer into the Cayman offshore feeder. The feeder issues its own interests. The subscription agreement captures the data the whole structure depends on: benefit-plan status (the Section 4.3 cap monitor), US-person status and FATCA/CRS classification (Section 3.4), and the investor's tax box.
- **Capital transfers to the master.** The feeders subscribe the cash into the Cayman master in exchange for master shares, so the master's capital base is the aggregate of all feeders' capital. On the first dealing date the master is capitalised at US$200 million (the first close) and trading begins. (In a closed-ended variant — private equity or credit — this leg runs as capital calls from the feeders to the master against commitments; the PFL registration timing of Section 5.4 is keyed to that rhythm.)
- **The master's trading loop.** The master executes through Cymbal Bank's prime brokerage: margin financing against the portfolio, securities lending and borrow, daily P&L, FX forwards for the non-USD legs, and custody of positions (Section 10.3). All portfolio activity is recorded at the master level; the feeders never trade.
- **NAV at both layers.** The administrator values the master's portfolio to produce the master NAV; each feeder NAV is its proportional share of the master NAV adjusted for feeder-level fees and expenses; an investor's per-unit NAV is the feeder NAV divided by the feeder's units outstanding (Section 10.4).
- **Distributions and redemptions flowing back down.** A redeeming investor requests redemption at its feeder; the feeder redeems proportionally from the master; cash flows master → feeder → investor on the redemption payment date. Distributions follow the same path.
- **The tax isolation, per investor class.** The US taxable partner in the Delaware feeder takes QEF inclusions currently under IRC §1293, with previously taxed income coming out tax-free on later distributions (Section 3.2); the non-US investor in the offshore feeder sits outside the US tax net, subject to FATCA reporting and the ECI lines of Section 3.5; the US tax-exempt investor receives dividends from the corporate feeder rather than UBTI (Section 3.3); and no Cayman tax is withheld anywhere in the loop (Section 5.1). One portfolio, three tax outcomes — the entire reason the architecture exists.

### 10.3 The Cymbal Bank Product Map — Administration, Custody, Prime Brokerage

Which Cymbal Bank desks serve which layer:

| Cymbal Bank desk | Serves | What it does in this structure |
| --- | --- | --- |
| **Fund administration** | Master + both feeders | Computes the master NAV and both feeder NAVs; maintains the investor registers at each feeder; processes subscriptions, redemptions and transfers on the dealing calendar; produces the annual PFIC/QEF information packet the Delaware feeder's partners need for Form 8621 (Section 3.2); runs the PFL disciplines of Section 5.4 — valuation, safekeeping records, cash monitoring, identification of securities |
| **Custody** | Master | Holds the master's positions, verifies title, settles trades; custody attaches to the master because the master owns the assets |
| **Prime brokerage** | Master | Margin financing, securities lending and borrow (with rehypothecation governed by the PB agreement and the master's offering documents), daily P&L and margin reporting, execution connectivity — the mechanics cross-referenced from [Citadel LLC](citadel_llc_guide.md) §10 rather than re-derived; the PB relationship is the master's, not the feeders' |
| **Cash management / FX** | Master (and the payment legs) | Sweeps the master's cash, runs the subscription and redemption payment flows at both layers, executes the strategy's FX; multi-currency subscription proceeds are converted at the master level on the dealing date |
| **Compliance (KYC/AML overlay)** | All four entities | The layered onboarding, sanctions screening and transaction filtering of Section 10.5 |

For the banker's own revenue map: **administration monetises both layers, custody and prime brokerage monetise the master**, and the KYC/AML overlay monetises compliance infrastructure across the manager, the master and both feeders. The structure is sticky for the reasons of Section 1.4 — once the QEF elections, the ERISA caps and the service-provider stack are in place, the client relationship is durable.

### 10.4 NAV, Reporting, and the Redemption Cycle

- **The NAV cascade.** Master NAV (portfolio valuation plus accruals) → feeder NAVs (proportional share of master NAV, less feeder-level fees and expenses) → investor statements (feeder NAV per unit × units held). One valuation date, one cascade, two layers of books.
- **The dealing cycle.** In this scenario the fund deals monthly: subscription and redemption notices on a 30–60 day calendar ⚠ (document terms, not statutory), a dealing date at month-end, NAV struck as of the dealing date, redemption proceeds paid in the settlement window the fund documents define (typically 5–10 business days) ⚠. The administrator's dealing calendar is the spine of the whole cycle.
- **Gates and suspensions context.** The fund documents may provide for **gates** (redemptions capped at a percentage of NAV per dealing date), **side pockets** (illiquid positions segregated so redeeming investors are treated fairly), or **suspension of dealing** in stress. The regulatory layer watches this: the BVI FSC's incubator/approved-fund guidelines require a fund to notify the Commission of any matter with a material impact on its affairs — expressly including **suspension of subscriptions or redemptions** ✅ (FSC Incubator and Approved Funds Guidelines §6.1.3, cached verbatim). The Cayman equivalent notification practice for MFL/PFL funds is not re-verified this pass ⚠ (§12).
- **Valuation discipline.** The Cayman Private Funds Law requires valuations appropriate to the assets held and, in any case, at least annually ✅ (Loeb Smith, §5.4); the BVI PIF regime requires a clear and comprehensive valuation policy with valuations at least annually ✅ (Harneys, §6.3). The administrator sells exactly this discipline, plus the monthly (or daily/weekly) NAV cadence the master-feeder needs for the investor statements to mean anything.

### 10.5 The KYC/AML Overlay

The four-layer AML-KYC stack of the sibling [Fircosoft](fircosoft_guide.md) guide — **CDD → screening → transaction monitoring → sanctions compliance**, with case management as the connective layer (fircosoft_guide.md §1.1) — runs across **every entity** in the structure:

- **The layered KYC problem.** The bank must know the **manager** (Meridian Point's principals and controllers); the **master** and **each feeder** (corporate vehicles: constitutional documents, directors, registered agent, ownership); and — through the look-through question — **the investors**, because for a fund client the ultimate beneficial owners are the investors and the feeder's investor register is the KYC dataset. The subscription agreement is where the bank collects what the overlay needs: legal name, aliases, nationality, address, beneficial-ownership attestations, FATCA/CRS classification (Section 3.4), and benefit-plan status (Section 4.3).
- **Sanctions and name screening.** Onboarding screening checks the manager, the master, each feeder and each investor against the sanctions and watch lists — OFAC's SDN and consolidated lists, the UN Security Council Consolidated List, the EU lists, the UK OFSI list, and MAS's designated lists (fircosoft_guide.md §4) — with the exact-vs-fuzzy matching discipline (string-distance, phonetic and transliteration handling) that the Fircosoft guide derives in full (fircosoft_guide.md §3.1, §5.3). Screening is a gate, not a report: a confirmed hit at onboarding means refuse or licence; list changes trigger rescreening of the existing investor base (fircosoft_guide.md §5.1).
- **Transaction filtering.** Every payment in the flow of funds of Section 10.2 — subscription wires into the feeders, capital transfers to the master, redemption payouts, the master's own payments through the prime broker — is screened for list hits in all named parties before release, with block/hold/release outcomes and the latency budget the rails require (fircosoft_guide.md §3.3, §5.2); the SWIFT MT / ISO 20022 format context is cross-referenced to payment_rails_guide.md and fircosoft_guide.md §3.3.
- **The structure's special features.** The same name is screened at four levels (investor, feeder, master, manager), so the screening estate must handle **entity hierarchies** — a sanctioned investor in the feeder register and a sanctioned counterparty on the master's blotter are the same risk class. The fund-side AML obligations are real: a BVI PIF must appoint a money laundering reporting officer, run investor-onboarding procedures and file under FATCA/CRS — and the Harneys guide notes that funds "particularly" choose administrators for exactly this support ✅ (Harneys, cached; §6.3). The Cayman fund-side AML overlay (the Proceeds of Crime Law family) is the same shape but was not re-verified this pass ⚠ (§12). The bank's own Singapore overlay is the MAS regime — cross-ref [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) (Notice 626 and the MAS designated lists, per fircosoft_guide.md §4.5).

**Cymbal Bank institutional convention.** Per the repository's house style, Cymbal Bank is the only bank persona in this guide and in this worked example: every desk above — fund administration, custody, prime brokerage, cash management/FX, and the compliance overlay — is Cymbal Bank's franchise, consistent with the sibling worked examples ([Citadel LLC](citadel_llc_guide.md) §10, [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §9, [Fircosoft](fircosoft_guide.md) §7–8).

---

## 11. The Claims Audit — Verified, Flagged, Rejected

### 11.1 The Verified Claims (✅)

Per the integrity convention in the preamble: ✅ means verified this pass against a primary or cited source, and the source is named. The table below mines the inline citations already carried in Sections 1–9 and re-checks them against the cached primary-source extracts (law.cornell.edu, the Cayman DITC statute, the BVI FSC, Ogier, Harneys, Loeb Smith, Arendt, Wikipedia) plus one live re-verification (the EU-list page, Section 7.2).

| Claim | Source |
| --- | --- |
| PFIC definition — IRC §1297, the 75% passive-income or 50% passive-asset test | law.cornell.edu/uscode/text/26/1297 (cited §3.1) |
| PFIC default tax — IRC §1291 interest charge on excess distributions; QEF current inclusion — §1293; the QEF election — §1295; mark-to-market — §1296 | law.cornell.edu/uscode; IRS Form 8621 instructions, 12/2025 revision (cited §3.1–3.2) |
| Form 8621 reporting obligations, including §1298(f) annual reporting; Form 8621-A exists for certain late elections | irs.gov (cited §3.2) |
| UBTI — IRC §511 imposes the tax; §512 defines it | law.cornell.edu/uscode/text/26/511 (cited §3.3) |
| FATCA — Code Chapter 4, §§1471–1474; 30% withholding on withholdable payments to non-compliant foreign financial institutions | law.cornell.edu/uscode/text/26/subtitle-A/chapter-4 (cited §3.4) |
| ECI — IRC §882 taxes foreign corporations on ECI; §864(b)(2) excludes trading in stocks or securities (own account, or through resident brokers/independent agents) | law.cornell.edu/uscode/text/26/864 — cached verbatim this pass (cited §3.5) |
| ERISA plan assets — 29 CFR §2510.3-101(a)(2) look-through; (f)(1) the 25% significance test; (f)(2) the full benefit-plan-investor definition (ERISA §3(3) plans whether or not Title I-subject, Code §4975(e)(1) plans, and entities whose underlying assets include plan assets); (j)(2) the 15% + 15% > 25% worked example | law.cornell.edu/cfr/text/29/2510.3-101 — cached verbatim; the (f)(2) enumeration is upgraded from ⚠ to ✅ this pass (cited §4.1) |
| Cayman no-direct-tax profile; Tax Concessions Act undertakings up to 30 years (normally 20); incorporation typically 3–5 business days, 1–2 with the US$610 express fee | Ogier, "Cayman Islands exempted companies", June 2026 — cached (cited §5.1) |
| Cayman SPC — Part XIV of the Companies Act, introduced May 1998; one legal entity with statutorily segregated portfolios; name must include "SPC"/"Segregated Portfolio Company"; annual return plus portfolio notice | Ogier SPC briefing, June 2026; Cayman General Registry — cached (cited §5.2) |
| Mutual Funds Law categories (licensed / administered / registered); the US$100,000 aggregate minimum for registered funds, master funds expressly addressed; full and restricted administrator tiers; the four common fund vehicles | cima.ky Investment Funds page (cited §5.3) |
| Private Funds Law 2020 — in force 7 February 2020; registration within 21 days of commitments and before contributions; transitional window to 7 August 2020; audit, valuation, safekeeping, cash-monitoring and securities-identification requirements; CI$100,000 penalty for non-registration | Loeb Smith, 13 February 2020 — cached verbatim (cited §5.4) |
| Cayman economic substance — Law 45 of 2018, in force 1 January 2019 (commencement per the DITC endnotes); "investment fund" excluded from "relevant entity"; "relevant activity" expressly excludes investment fund business; fund management business = SIBL-licensed/authorised managers; penalties CI$10,000 / CI$100,000 / CI$5,000 + CI$500 per day | DITC, International Tax Co-operation (Economic Substance) Act (2021 Revision) — cached verbatim (cited §5.5) |
| BVI SIBA 2010 (Revised 2020) — the five fund categories with the verified parameters; the US$1,500 incubator/approved application fee; the two-business-day fast track; incubator (≤US$20M net assets, ≤20 investors, US$20,000 minimum, 2-year validity + up to 12 months) and approved (≤US$100M, ≤20 investors, unlimited validity) thresholds | BVI FSC Incubator and Approved Funds Guidelines (issued 28 May 2015, commenced 1 June 2015) — cached verbatim; Harneys, "BVI Open-Ended Funds and Approved Funds" (cited §6.2) |
| BVI PIF regime — Private Investment Funds Act 2019 + PIF Regulations (Revised 2020); the PIF definition and exclusions (single investor, single asset, joint venture, SPAC); private basis ≤50 investors / professional basis US$100,000 minimum; the US$1,000,000 net-worth professional investor; appointed persons (management/valuation/safekeeping); application/recognition/annual fees of US$850/US$1,200/US$1,200 | Harneys, "Establishing a Closed-Ended Fund in the BVI" (2026) — cached verbatim (cited §6.3) |
| BVI economic substance — Economic Substance (Companies and Limited Partnerships) Act 2018, effective 1 January 2019; nine relevant activities; investment fund business expressly not a relevant activity; fund management business defined by reference to SIBA licensing; grandfathering to 30 June 2019; BOSS reporting within six months | Harneys BVI economic-substance guide — cached verbatim (cited §6.4) |
| EU list, situation on 17 February 2026 — Annex I: 10 jurisdictions (American Samoa, Anguilla, Guam, Palau, Panama, Russian Federation, Turks and Caicos, US Virgin Islands, Vanuatu, Viet Nam) — Cayman and BVI on neither; Annex II: 9 jurisdictions (Belize, British Virgin Islands, Brunei Darussalam, Eswatini, Greenland, Jordan, Montenegro, Morocco, Türkiye) — BVI on the grey list, Cayman on neither annex | European Commission, "Common EU list of third country jurisdictions for tax purposes" (situation on 17 February 2026) — re-verified live this pass (cited §7.2). Note: the cached extracts (Arendt; Wikipedia) carried the Annex I list but not Annex II; the Annex II detail was re-verified live against the EC page and stands ✅ |
| BVI added to Annex I in February 2023; removed (moved to Annex II) on 17 October 2023 | Council press release, 14 February 2023, via Wikipedia's citations; Brussels Times, 17 October 2023 (cited §7.2) |
| Next EU-list revision due October 2026 | Arendt, 18 February 2026 — cached verbatim (cited §7.2) |
| Cayman beneficial-ownership regime — Beneficial Ownership Transparency Act in force 31 July 2024, replacing the prior regime; 25% ownership/control threshold | Ogier exempted-companies briefing — cached (supplements §5.1) |
| BVI PIF AML obligations — money laundering reporting officer, investor-onboarding procedures, FATCA/CRS reporting; the administrator as the natural AML home | Harneys, cached (supplements §6.3, §10.5) |

### 11.2 The Flagged Claims (⚠)

- **The BVI PIF commencement date** (§6.3): the Act's year (2019) and the Revised 2020 regulations are ✅; the widely documented "in force 1 January 2020, register by 7 January 2020" commencement was not re-verified live.
- **Cayman's grey-list window (February–October 2020)** (§7.2): the cached Wikipedia history confirms Cayman sat on the 2019-era grey list of committing jurisdictions, but the precise window was not re-extracted.
- **The reasons for the BVI's continuing Annex II listing**: the EC's tax-deficiencies table (17 February 2026) was not fully extracted; the listing itself is ✅, the reasons ⚠.
- **Directional cost and speed figures** (§7.1): "Cayman incorporation within a day" versus Ogier's verified 3–5 business days standard / 1–2 with the express fee; the BVI administrator-market depth comparison; the full 2026 fee schedules for both jurisdictions.
- **The hybrid VCC-plus-offshore pattern** (§9.2): directional industry observation, flagged at the point of use.
- **The MFL s4(4) exemption limbs** (§5.3): the exemption's existence is ✅ at CIMA; the classic limbs (15-or-fewer investors; sophisticated-investor routes) were not re-extracted from the statute.
- **§514 debt-financed income, §1441 chapter-3 withholding, and FIRPTA §897** (§3.3–3.5): standard points, not re-verified live.
- **Late QEF election conditions** — Form 8621-A exists ✅; the eligibility conditions ⚠ (§3.2).
- **FATCA/CRS IGA mechanics** for Cayman and the BVI: Model 1 IGA status is standard and widely documented but was not re-extracted from the IRS FATCA page (§3.4).
- **BVI SPC mechanics and comparative take-up** (§6.1): the BCA s159 SPC and the 2005 regulations are ✅ at bvifsc.vg; the "much less used than the Cayman SPC" observation is directional.
- **The Cayman Companies Act's precise revision year** as at August 2026 (§5.2).
- **Cayman suspension-notification practice** for fund dealing suspensions — the BVI equivalent is ✅ verbatim; the Cayman practice is not (§10.4).
- **Form 8621 instructions (12/2025) content** beyond the form's existence and the elections it covers (§3.1–3.2).
- **Prime-brokerage commercial terms** (haircuts, rehypothecation limits, margin rates) for the worked example — cross-referenced to [Citadel LLC](citadel_llc_guide.md) §10 rather than verified here (§10.3).

### 11.3 The Rejected Claims (❌)

- **"Cayman is on the EU blacklist."** ❌ Refuted by the 17 February 2026 update: Cayman appears on neither annex of the EU list (EC page, verified live; §7.2). Its grey-list history (2020) was a commitments status, not a blacklisting.
- **"The BVI is on the EU blacklist."** ❌ The BVI is on **Annex II** — the grey list of jurisdictions that have committed to tax good-governance standards but are not yet fully compliant — not Annex I. The defensive measures (non-deductibility, CFC rules, higher withholding) attach to Annex I (EC page; Arendt; §7.2). The BVI *was* on Annex I between February 2023 and October 2023 — history, not current status.
- **"Cayman's economic substance law is a 2019 law."** ❌ It is the International Tax Co-operation (Economic Substance) Law, **45 of 2018**, in force **1 January 2019** — the statute's own endnotes record "Originally enacted — Law 45 of 2018 — 17th December, 2018" with commencement 1-Jan-2019 (DITC, cached verbatim; §5.5).
- **"The BVI economic substance act is a 2019 act."** ❌ It is the Economic Substance (Companies and Limited Partnerships) Act, **2018**, effective **1 January 2019** (Harneys, cached verbatim; §6.4).
- **"The economic substance regime reaches pure investment funds."** ❌ In both jurisdictions the fund itself is out of scope: Cayman excludes "investment fund" from "relevant entity" and "investment fund business" from "relevant activity" (DITC, cached verbatim; §5.5), and the BVI Act "expressly confirms that investment fund business is not relevant activity" (Harneys, cached verbatim; §6.4). The regimes reach **fund managers** — SIBL-/SIBA-licensed managers carrying on fund management business — not the funds themselves.
- **"The '25% blocker' is a statutory term."** ❌ It is industry shorthand for a blocker corporation sized so benefit-plan participation stays under the 25% threshold of 29 CFR §2510.3-101(f); the regulation contains no such term (cited §3.3, §4.3).

---

## 12. What Could Not Be Verified

The integrity convention of this guide is that nothing unverifiable is asserted as fact. The following could not be verified this pass and are flagged ⚠ wherever they appear above:

1. The **exact commencement day of the BVI Private Investment Funds Act 2019** — widely documented as 1 January 2020, with FSC transitional arrangements to 7 January 2020; the Act's year and the Revised 2020 regulations are verified, the commencement date is not re-verified live.
2. The **precise Cayman grey-list window (February–October 2020)** — the cache supports the general history, not the exact dates.
3. The **detailed reasons behind the BVI's continuing Annex II listing** — the EC tax-deficiencies table of 17 February 2026 was not fully extracted.
4. **Precise formation-cost and fee ranges** for both jurisdictions as at 2026 — the verified figures are point figures (US$1,500 incubator/approved fee; KYD 300 per SPC portfolio capped at KYD 1,500; PIF application/recognition/annual fees of US$850/US$1,200/US$1,200; Ogier's US$610 express incorporation fee), not full schedules.
5. The **"Cayman incorporation within a day"** claim — Ogier verifies 3–5 business days standard, 1–2 with the express fee.
6. The **specific limbs of Mutual Funds Law s4(4)** — the exemption's existence is verified; its exact criteria were not re-extracted.
7. **IRC §514 (debt-financed income), §1441 (chapter-3 withholding) and §897 (FIRPTA)** — standard points, not re-verified live.
8. The **eligibility conditions for late QEF elections** — Form 8621-A exists; the conditions were not verified.
9. The **FATCA Model 1 IGA mechanics** for Cayman and the BVI — standard and widely documented, but not re-extracted from the IRS page this pass.
10. **BVI SPC mechanics in detail** and its comparative take-up versus the Cayman SPC — directional.
11. The **precise revision year of the Cayman Companies Act** as at August 2026.
12. The **Cayman notification practice for fund dealing suspensions** — the BVI equivalent is verified verbatim; the Cayman practice is not.
13. The **BVI administrator-market depth** versus Cayman — directional.
14. The **prevalence of the hybrid VCC-plus-offshore pattern** — directional industry observation (§9.2).
15. **Prime-brokerage commercial terms** (haircuts, rehypothecation limits, margin rates) for the worked example — cross-referenced to the Citadel guide rather than verified here.
16. **Form 8621 instructions (12/2025) content** beyond the form's existence and the elections it covers.
17. The **manager-side facts of the worked example** (Meridian Point Capital and its investors) — fictional scenario parameters, not real-world facts, and not verified as such.

---

## 13. Glossary

| Term | Meaning (cross-ref) |
| --- | --- |
| **Feeder fund** | The investor-facing vehicle in a master-feeder structure; investors subscribe into the feeder, which subscribes into the master (§2.1). |
| **Master fund** | The asset-holding and trading entity; the feeders invest into it, and it holds the portfolio and runs the prime-brokerage/custody relationships (§2.1). |
| **PFIC** | Passive Foreign Investment Company — a foreign corporation meeting IRC §1297's 75% passive-income or 50% passive-asset test; every Cayman/BVI fund is one for US tax purposes (§3.1). |
| **QEF** | Qualified Electing Fund — a PFIC for which the shareholder elects under IRC §1295, converting the punitive §1291 default into current inclusion of ordinary earnings and net capital gain under §1293 (§3.2). |
| **Form 8621** | The IRS information return filed by US persons who own PFIC stock, report QEF or §1296 elections, or are required to report under §1298(f) (§3.2). |
| **UBTI** | Unrelated business taxable income — income of a tax-exempt organisation (IRC §§511–512) from an unrelated trade or business, including debt-financed income; the reason tax-exempt investors use corporate feeders and blockers (§3.3). |
| **Blocker corporation** | A corporation interposed so tax-exempt (or other) investors hold shares of a corporation rather than a flow-through interest, blocking UBTI and the look-through (§3.3). |
| **ECI** | Effectively Connected Income — US-source business income of a foreign person taxed under IRC §882; §864(b)(2) excludes securities trading, including for the taxpayer's own account (§3.5). |
| **FATCA** | Foreign Account Tax Compliance Act — Code Chapter 4 (IRC §§1471–1474); 30% withholding on withholdable payments to non-compliant foreign financial institutions (§3.4). |
| **ERISA** | The US Employee Retirement Income Security Act of 1974 — the statute whose fiduciary and prohibited-transaction rules bite when plan assets are involved (§4). |
| **Plan assets** | Under 29 CFR §2510.3-101(a)(2), when a plan invests in an entity the plan's assets include an undivided interest in the entity's underlying assets unless the entity is an operating company or benefit-plan participation is not significant (§4.1). |
| **The 25% test** | The significance threshold of 29 CFR §2510.3-101(f)(1): benefit-plan participation of 25% or more of the value of any class of equity interests triggers look-through (§4.1). |
| **SPC** | Segregated Portfolio Company — a Cayman exempted company (Part XIV of the Companies Act, introduced 1998) or a BVI company (BCA s159) whose portfolios are statutorily ring-fenced (§5.2, §6.1). |
| **CIMA** | The Cayman Islands Monetary Authority — regulates mutual funds, private funds and fund administrators (§5.3–5.4). |
| **Mutual Funds Law (MFL)** | The Cayman statute regulating open-ended funds; licensed, administered and registered categories plus the s4(4) exemption (§5.3). |
| **Private Funds Law (PFL)** | The Cayman Private Funds Law 2020 — CIMA registration for closed-ended private funds, in force 7 February 2020 (§5.4). |
| **SIBA** | The BVI Securities and Investment Business Act 2010 (Revised 2020) — the statute under which BVI funds fall into five categories (§6.2). |
| **Incubator fund** | BVI category: ≤20 investors, ≤US$20M net assets, US$20,000 minimum, two-year validity (plus up to 12 months), no audit, two-business-day fast track (§6.2). |
| **Approved fund** | BVI category: ≤20 investors, ≤US$100M net assets, administrator required, unlimited validity, two-business-day fast track (§6.2). |
| **Private fund (BVI)** | BVI category: ≤50 investors or private invitation only, recognition by filing (§6.2). |
| **Professional fund** | BVI category: professional investors only, US$100,000 minimum, licensed/recognised manager and auditor required, recognition by filing (§6.2). |
| **Public fund** | BVI retail category: prospectus under SIBA and the Public Funds Code; FSC registration before business (§6.2). |
| **PIF** | Private Investment Fund — the BVI closed-ended fund regime under the Private Investment Funds Act 2019 and the PIF Regulations (Revised 2020) (§6.3). |
| **NPPR** | National Private Placement Regime — the per-member-state route by which non-EU managers market non-EU AIFs under AIFMD Article 42 (§8.2). |
| **AIFMD** | The Alternative Investment Fund Managers Directive (2011/61/EU) and its successor AIFMD II (Directive (EU) 2024/927) (§8.1). |
| **VCC** | Variable Capital Company — the Singapore fund vehicle under the Variable Capital Companies Act 2018, launched 14 January 2020 (§9.1). |
| **13O/13U** | Singapore Income Tax Act fund-incentive schemes (13O: S$5M minimum AUM; 13U: S$50M), with the revised criteria effective 1 January 2025 and the schemes extended to 31 December 2029 (§9.1). |
| **Economic substance** | The requirement, in both Cayman and BVI, that entities carrying on relevant activities have real direction, management and core income generating activities in the jurisdiction; pure investment funds are excluded in both (§5.5, §6.4). |
| **EU grey list** | Annex II of the EU list — jurisdictions that have committed to tax good-governance standards but are not yet fully compliant (the BVI's current status) (§7.2). |
| **EU blacklist** | Annex I of the EU list — non-cooperative jurisdictions subject to defensive measures; neither Cayman nor the BVI is on it as at 17 February 2026 (§7.2). |
| **Prime brokerage** | The bundled financing/custody/execution service a fund's trading entity uses — margin, securities lending, rehypothecation, daily P&L and margin reporting (§10.3). |
| **Fund administrator** | The service provider that computes NAVs, keeps investor records, processes subscriptions and redemptions, and produces tax and reporting packets (§10.3). |
| **Custodian** | The institution that holds a fund's assets, verifies title and settles trades (§10.3). |
| **NAV** | Net Asset Value — assets less liabilities; the master NAV cascades into feeder NAVs and per-unit investor NAVs (§10.4). |
| **Subscription / redemption** | The purchase of fund interests by an investor / the sale of interests back to the fund at NAV, on the dealing calendar defined in the fund documents (§10.2, §10.4). |

---

## 14. Cross-References and Further Reading

**Sibling guides in `banking/` (plain filenames, per the repository convention):**

- [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) — the VCC and "Why Global Managers Still Go Offshore" (§3.1–3.2); the Cymbal Bank worked-example conventions (§9).
- [Private Equity](private_equity_guide.md) — AIFMD and AIFMD II (§7); the Singapore regime (§8); the subscription-line worked example (§9).
- [Private Equity Singapore](private_equity_singapore_guide.md) — the Singapore closed-ended overlay: the 13U route, the SFO exemption, MAS licensing.
- [Citadel LLC](citadel_llc_guide.md) — the prime-brokerage worked-example mechanics (§10), cross-referenced in Sections 10.3 and 12.
- [Norges Bank Investment Management](norges_bank_investment_management_guide.md) — the institutional-LP view of funds and their service providers.
- [Fircosoft](fircosoft_guide.md) — sanctions screening, name matching, list management and transaction filtering; the KYC/AML overlay of Section 10.5 is built on its framework (§1, §3, §5).
- [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) — Notice 626 and the Singapore AML/CFT overlay for the bank side of the structure.

**Primary sources (verified this pass, or cited in Sections 1–9):**

- law.cornell.edu — 26 U.S.C. §§511–514 (UBTI), 864 (ECI definitions), 882 (ECI tax), 1291–1297 (PFIC/QEF), 1471–1474 (FATCA); 29 CFR §2510.3-101 (plan assets).
- irs.gov — Form 8621 and instructions (12/2025 revision); Form 8621-A (certain late elections).
- cima.ky — the Mutual Funds Act and Private Funds Law pages; fund-administrator licensing.
- ditc.ky — the International Tax Co-operation (Economic Substance) Act (2021 Revision), consolidated text.
- ciregistry.ky — the Companies Act (Part XIV, SPCs) and the registry fee table.
- bvifsc.vg — SIBA (Revised 2020); the Private Investment Funds Act 2019 and PIF Regulations (Revised 2020); the Economic Substance (Companies and Limited Partnerships) Act 2018; the Incubator and Approved Funds Guidelines (2015).
- taxation-customs.ec.europa.eu / consilium.europa.eu — the EU list of non-cooperative jurisdictions (situation on 17 February 2026; next revision due October 2026).
- main.un.org — the UN Security Council consolidated sanctions list (cross-ref fircosoft_guide.md §4.2).

**Law-firm and professional guides (the ones this guide's citations rest on):**

- Ogier — "Cayman Islands exempted companies" (June 2026); "Segregated portfolio companies in the Cayman Islands" (June 2026).
- Harneys — "BVI Open-Ended Funds and Approved Funds"; "Establishing a Closed-Ended Fund in the BVI" (2026); "Economic substance in the BVI: a guide for directors and operators" (April 2023).
- Loeb Smith — "Cayman Islands introduce Private Funds Law to regulate closed-ended investment funds" (13 February 2020).
- Conyers — the SPC "Silver Anniversary" note (July 2023).
- Arendt — "Turks and Caicos Islands and Vietnam added to EU list of non-cooperative jurisdictions for tax purposes" (18 February 2026).
- Carey Olsen — the BVI segregated portfolio company regulations (2005); Pillsbury — the Cayman Private Funds Law 2020.

---

## 15. Closing Summary

**The architecture.** The master-feeder structure is the standard plumbing of the global funds industry: investor-facing feeders — a Delaware LP for US taxable investors, a Cayman or BVI vehicle for non-US and US tax-exempt investors — each subscribing into a single Cayman or BVI master that holds the portfolio and trades. The structure exists to run **one portfolio, one trading book and one set of service-provider relationships** while giving **each investor group its own tax and regulatory box** — aggregation at the master, isolation at the feeders, and the whole apparatus justified by US tax and ERISA constraints rather than by anything the offshore jurisdictions themselves require.

**The jurisdictions.** Cayman and the BVI are both tax-neutral fund domiciles with regulated fund regimes — Cayman's Mutual Funds Law and Private Funds Law under CIMA, the BVI's five SIBA categories and the PIF regime under the FSC — and economic-substance regimes that exclude pure investment funds: the ES layer is an administrative filing for a fund, not a substance burden, while locally licensed fund managers carry the real test. The comparison of Section 7 is a texture difference — speed, cost, categories, investor preference and EU-list standing (Cayman on neither annex, the BVI on the grey list, as at 17 February 2026) — not a difference in the architecture.

**The tax drivers.** The PFIC regime and the QEF election (IRC §§1291/1293/1295/1296/1297), UBTI and the blocker question, FATCA's 30% withholding, ECI and the §864(b)(2) trading shelter, and the ERISA plan-assets look-through with its 25% test (29 CFR §2510.3-101) — each verified at the statute or regulation in this guide — are what make the Delaware-feeder-plus-offshore-master stack the answer for US-facing managers, and what make the structure sticky once built.

**The bank's role.** For Cymbal Bank the structure is a multi-product client: fund administration monetises both layers, custody and prime brokerage monetise the master, cash management and FX sit on the payment loop, and the KYC/AML overlay — layered onboarding, sanctions and name screening, and transaction filtering, built on the Fircosoft guide's framework — spans the manager, the master and every feeder.

**The discipline.** This guide verified what it could — the statutes, the regulations, the regulator pages, the law-firm guides — flagged what it could not, and rejected what the evidence contradicted; the claims audit of Section 11 and the honest list of Section 12 are the record of that. Nothing here was invented.

The master-feeder structure is not exotic; it is the ordinary, durable answer to a genuinely hard problem — many kinds of investors, one portfolio, and a tax system that punishes mixing them. Understand the feeders, the master, the tax drivers and the bank's place in the loop, and the whole edifice becomes legible — and that is the offshore architecture.
