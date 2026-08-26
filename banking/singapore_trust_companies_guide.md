# Trust Companies in Singapore: The Quiet Custodians — A Comprehensive Guide

**The SG Trust-Industry Deep-Dive — Trust Basics, the Legal Regime, Trust Types, Private Trust Companies, Market Players, the Family-Office Interplay, and a Worked PTC/VCC Structure**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Banking Domain / Wealth & Estate-Planning Infrastructure — Trust Companies in Singapore: Trust Basics (Settlor–Trustee–Beneficiary), the Trust Companies Act 2005 Regime, MAS Licensing, Trust Types, Private Trust Companies (PTCs), Market Players (Bank-Owned Trustees & Independents), the Family-Office (13O/13D) Interplay, Worked Example, One-Page Summary
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** sso.agc.gov.sg (Trust Companies Act 2005; Trust Companies (Exemption) Regulations; Business Trusts Act 2004), mas.gov.sg (Financial Institutions Directory, licensing framework, TCA-N03), the trust companies' own sites (DBS Trustee, BOS Trustee/OCBC, HSBC Trustee, Standard Chartered, Vistra, TMF, CSC/Intertrust, Asiaciti, Tricor, Apex, Ocorian, JTC), Singapore Trustees Association (sta.org.sg), STEP, Law Gazette (2007 TCA update), industry commentary (Anlian Group, Kensington Trust, CapitalMarkets.SG), press (The Business Times, WealthBriefingAsia, Hubbis)
> **Last Updated:** August 2026
> **Companion guides:** [Singapore Private Markets](singapore_private_markets_guide.md) (the VCC/13O/13U fund regime — THE sibling), [Bond Financial Group](bond_financial_group_company_guide.md), [Bond Capital Group](bond_capital_group_company_guide.md) (SG holding/wealth structures), [Banks in Singapore](banks_in_singapore_guide.md) (the bank-owned trustees' parents), [Trust Bank](trust_bank_guide.md) (the DIGITAL BANK — see the name-collision note below), [Asset-Backed Trading](asset_backed_trading_guide.md), [Financial Infrastructure](financial_infrastructure_guide.md), [Risk Management Models](risk_management_models_guide.md), [Supply Chain Finance](supply_chain_finance_guide.md), [Universal Banking Model](universal_banking_model_guide.md), [MBA Body of Knowledge](../management/mba_body_of_knowledge_guide.md), [CFA Program](cfa_program_guide.md)

---

## 0. The Name-Collision Warning — Trust Companies ≠ Trust Bank

Before anything else, the naming trap. **"Trust Bank" in this repository is the DIGITAL BANK** — the Standard Chartered × FairPrice Group (NTUC) joint-venture consumer digital bank, covered in [trust_bank_guide.md](trust_bank_guide.md). It holds a full bank licence and has nothing to do with trusts in the fiduciary sense.

**This guide is about TRUST COMPANIES — the MAS-licensed trustees** that hold and administer trust assets for families and institutions under the Trust Companies Act 2005. The word "trust" in "Trust Bank" is a brand name; the word "trust" in "trust company" is a legal relationship (and a licence category). They share a syllable, not a business:

| | Trust Bank (the digital bank) | Trust companies (this guide) |
|---|---|---|
| What it is | Consumer digital bank (SC × FairPrice JV) | Fiduciaries licensed to act as trustee |
| Regulator / regime | Full bank licence (MAS, Banking Act) | Trust Companies Act 2005 + MAS licensing |
| Core activity | Deposits, payments, mass-market retail | Settlor→trustee→beneficiary structures, estate planning, CIS trusteeship |
| Sibling guide | [trust_bank_guide.md](trust_bank_guide.md) | This file |

If a reader picks up [trust_bank_guide.md](trust_bank_guide.md) expecting DBS Trustee, they will be lost. This guide is the other side of that coin.

### 0.2 How to Read This Guide — the Companion Map

This guide is the **trust side of the SG wealth stack**. It is designed to be read alongside its siblings in the repository — the map:

| Sibling guide | What it adds to this guide |
|---|---|
| [singapore_private_markets_guide.md](singapore_private_markets_guide.md) | THE sibling — the VCC/13O/13U fund regime, the 2,000+ SFO story, the fund-manager licensing routes. The trust's fund-side mirror: trusts own the vehicles; the fund guide covers the vehicles |
| [bond_financial_group_company_guide.md](bond_financial_group_company_guide.md) / [bond_capital_group_company_guide.md](bond_capital_group_company_guide.md) | SG holding/wealth structure design — the corporate layer adjacent to the trustee layer |
| [banks_in_singapore_guide.md](banks_in_singapore_guide.md) | The parent groups of the bank-owned trustees (DBS, OCBC, UOB, HSBC) |
| [trust_bank_guide.md](trust_bank_guide.md) | The name-collision — the digital bank, NOT a trust company (§0.1) |
| [asset_backed_trading_guide.md](asset_backed_trading_guide.md) / [financial_infrastructure_guide.md](financial_infrastructure_guide.md) / [risk_management_models_guide.md](risk_management_models_guide.md) / [supply_chain_finance_guide.md](supply_chain_finance_guide.md) | The passing-mention adjacencies: trust assets get invested, custodied and financed through the machinery these guides cover |
| [universal_banking_model_guide.md](universal_banking_model_guide.md) | The bank model the trust companies plug into (and compete with) |
| [../management/mba_body_of_knowledge_guide.md](../management/mba_body_of_knowledge_guide.md) / [cfa_program_guide.md](cfa_program_guide.md) | The legal/entity and wealth-planning vocabulary this guide assumes |

**Reading convention:** every section carries its verification posture inline — **(Verified)** for claims confirmed against primary or strong secondary sources in this research pass; **(reported / flagged)** for claims that are widely stated but were not independently re-confirmed here. The claims-status table in §10 is the master list. Nothing here is investment, legal, or tax advice.

---

## Table of Contents

1. [The Trust Basics](#1-the-trust-basics)
   - 1.1 [What a Trust Is — the Definition](#11-what-a-trust-is--the-definition-verified)
   - 1.2 [The Three Parties — Settlor, Trustee, Beneficiary](#12-the-three-parties--settlor-trustee-beneficiary-verified)
   - 1.3 [The Basics Table — Role / Function / Notes](#13-the-basics-table--role--function--notes)
   - 1.4 [Supporting Cast — Protector, Letter of Wishes, Trust Deed](#14-supporting-cast--protector-letter-of-wishes-trust-deed)
   - 1.5 [Why Singapore — the Jurisdiction's Trust Appeal](#15-why-singapore--the-jurisdictions-trust-appeal)
   - 1.6 [The Milestones Timeline](#16-the-milestones-timeline)
2. [The Legal Regime](#2-the-legal-regime)
   - 2.1 [The Trust Companies Act 2005](#21-the-trust-companies-act-2005-verified)
   - 2.2 [The Prohibition and the Definition of Trust Business](#22-the-prohibition-and-the-definition-of-trust-business-verified)
   - 2.3 [The MAS Licensing Regime — Licensed vs Registered](#23-the-mas-licensing-regime--licensed-vs-registered-verified)
   - 2.4 [The Regime Table — Category / Who / Notes](#24-the-regime-table--category--who--notes)
   - 2.5 [Compliance Machinery — Capital, AML/CFT, Notices](#25-compliance-machinery--capital-amlcft-notices)
   - 2.6 [Reading the Regime — a Two-Tier Gate](#26-reading-the-regime--a-two-tier-gate)
   - 2.7 [The Regime in Practice — What a Licence Costs and Requires](#27-the-regime-in-practice--what-a-licence-costs-and-requires)
3. [The Trust Types](#3-the-trust-types)
   - 3.1 [Discretionary Trusts](#31-discretionary-trusts-verified)
   - 3.2 [Revocable vs Irrevocable Trusts](#32-revocable-vs-irrevocable-trusts-verified)
   - 3.3 [Unit Trusts — the Collective-Investment-Scheme Trustee](#33-unit-trusts--the-collective-investment-scheme-trustee-verified)
   - 3.4 [Purpose Trusts](#34-purpose-trusts-verified-flagged)
   - 3.5 [Charitable Trusts](#35-charitable-trusts-verified)
   - 3.6 [Business Trusts](#36-business-trusts-verified)
   - 3.7 [The Types Table — Type / Use / Notes](#37-the-types-table--type--use--notes)
   - 3.8 [The Tax Angle — How Trusts Are Taxed in Singapore](#38-the-tax-angle--how-trusts-are-taxed-in-singapore)
4. [The Private Trust Companies](#4-the-private-trust-companies)
   - 4.1 [What a PTC Is — the Family's Own Trustee](#41-what-a-ptc-is--the-familys-own-trustee-verified)
   - 4.2 [The MAS Exemption and Its Conditions](#42-the-mas-exemption-and-its-conditions-verified)
   - 4.3 [Why Families Use PTCs](#43-why-families-use-ptcs)
   - 4.4 [The PTC Table — Feature / Detail / Notes](#44-the-ptc-table--feature--detail--notes)
   - 4.5 [PTC vs Licensed Trust Company](#45-ptc-vs-licensed-trust-company)
5. [The Market Players](#5-the-market-players)
   - 5.1 [The Bank-Owned Trustees](#51-the-bank-owned-trustees-verified)
   - 5.2 [The Independents — Global Trust & Corporate Services](#52-the-independents--global-trust--corporate-services-verified-flagged)
   - 5.3 [The Players Table — Firm / Type / Notes](#53-the-players-table--firm--type--notes)
   - 5.4 [Reading the Market — Consolidation and the Two Tiers](#54-reading-the-market--consolidation-and-the-two-tiers)
   - 5.5 [The Economics — How Trust Companies Make Money](#55-the-economics--how-trust-companies-make-money)
6. [The Family-Office Interplay](#6-the-family-office-interplay)
   - 6.1 [The 13O/13D/13U Regime in One Paragraph](#61-the-13o13d13u-regime-in-one-paragraph-cross-ref)
   - 6.2 [How the Trust and the Fund Scheme Fit Together](#62-how-the-trust-and-the-fund-scheme-fit-together-verified)
   - 6.3 [The Interplay Table — Layer / Structure / Notes](#63-the-interplay-table--layer--structure--notes)
   - 6.4 [The AML/CFT Discipline Across Both Layers](#64-the-amlcft-discipline-across-both-layers)
   - 6.5 [The Regulatory Timeline — Trusts and FOs Converging](#65-the-regulatory-timeline--trusts-and-fos-converging)
7. [The Worked Example — A Family-Office Trust Structure](#7-the-worked-example--a-family-office-trust-structure)
   - 7.1 [The Scenario — the Lim Family](#71-the-scenario--the-lim-family)
   - 7.2 [The Structure — PTC + VCC + 13O Design](#72-the-structure--ptc--vcc--13o-design)
   - 7.3 [The Structure Diagram (textual)](#73-the-structure-diagram-textual)
   - 7.4 [The Design Decisions](#74-the-design-decisions)
   - 7.5 [The Lessons](#75-the-lessons)
   - 7.6 [The Implementation Plan — Three Phases](#76-the-implementation-plan--three-phases)
   - 7.7 [The Risk Register](#77-the-risk-register)
8. [The Summary — Trust Companies in One Page](#8-the-summary--trust-companies-in-one-page)
9. [Glossary](#9-glossary)
10. [Claims Status, References and Further Reading](#10-claims-status-references-and-further-reading)

---

## 1. The Trust Basics

### 1.1 What a Trust Is — the Definition (Verified)

A **trust** is a fiduciary legal relationship in which one party (the **settlor**) transfers legal ownership of assets to another party (the **trustee**), who holds and manages those assets for the benefit of a third party or parties (the **beneficiaries**). The trustee owns the assets **at law**; the beneficiaries own them **in equity**. This split of legal and beneficial ownership is the defining feature — it is why trusts work for estate planning, asset protection, and succession.

The Singapore-law frame (verified — industry commentary, e.g., Anlian Group, and the statutes):

- An **express trust** is constituted when a settlor transfers property to a trustee to hold for beneficiaries under a trust deed satisfying the **"three certainties"**: (1) certainty of intention (the settlor clearly intends to create a trust), (2) certainty of subject matter (the assets are identifiable), and (3) certainty of objects (the beneficiaries are identifiable). Without the three certainties, the arrangement is not a valid trust.
- The trustee owes **fiduciary duties** (loyalty, no conflict, no profit from the position) and statutory duties under the **Trustees Act 1967** — the Singapore trust statute that codifies trustee powers and duties (investment powers, delegation, maintenance and advancement, and — since the 2017 amendments — transparency and AML/CFT obligations under Part 7 and the Trustees Regulations 2017).
- The trust is **not a separate legal person** — it is a relationship. The trustee (a company or individual) is the legal person. This is the single most important structural fact: everything downstream — accounts, custody, contracts — sits in the trustee's name.
- A trust takes effect either during the settlor's lifetime (an **inter vivos** trust, settled on a trust deed) or on death (a **testamentary** trust, created by a will). Both are express trusts.

**Terminology caution (verified):** "trust company" is a regulated-industry term in Singapore, not a generic one. Under the Trust Companies Act 2005, carrying on **trust business** in or from Singapore is prohibited without a licence or exemption (§2). An ordinary company with "Trust" in its name that does not hold a trust licence is not a trust company in the regulated sense — which brings us back to the Trust Bank name-collision in §0.

### 1.2 The Three Parties — Settlor, Trustee, Beneficiary (Verified)

| Party | Role | Key points |
|---|---|---|
| **Settlor** | The person who creates the trust by transferring assets into it | Settles the trust deed; decides the terms; can reserve limited powers (e.g., to appoint/remove trustees — subject to the terms); in an irrevocable trust, gives up legal ownership and usually the right to vary the trust |
| **Trustee** | The legal owner and manager of the trust assets | Holds assets at law; owes fiduciary and statutory duties (Trustees Act 1967); must act in the beneficiaries' interests; personally liable on contracts entered as trustee (with indemnity out of trust assets); in Singapore, a *professional* trustee must be a licensed trust company (§2) or a permitted individual |
| **Beneficiary** | The person(s) for whose benefit the trust exists | Owns the assets in equity; has standing to enforce the trust against the trustee; may be a class ("my children", "charities") in a discretionary trust rather than named individuals; a beneficiary with a vested interest can compel proper administration |

The three-party structure is the whole game. The settlor *starts* the trust and steps back; the trustee *runs* it; the beneficiary *receives* it. Problems arise when roles blur — e.g., a settlor who keeps effective control may have the trust attacked as a **sham** (the trust looks real but is a facade for the settlor's own continued control) — a key reason professional trustees and clean governance matter (see §7).

### 1.3 The Basics Table — Role / Function / Notes

| Role | Function | Notes |
|---|---|---|
| Settlor | Creates the trust; transfers assets; sets the deed's terms | Can be an individual, a couple, or a corporate entity; in Singapore family structures often the patriarch/matriarch; must be careful not to retain de facto control (sham risk) |
| Trustee | Legal owner; administers and invests the assets; distributes per the deed | Licensed trust company (institutional) or PTC (family-owned) or individual; owes fiduciary + statutory duties; the regulated party under the TCA |
| Beneficiary | The economic owner; receives income/capital per the deed | Can be a class, unborn, or charitable; in discretionary trusts has no fixed entitlement until the trustee exercises discretion |
| Protector (supporting) | Oversight role appointed by the settlor in the deed | Vetoes certain trustee actions (e.g., removing a trustee, varying the trust); not a statutory concept in Singapore law but widely used in trust deeds (reported — standard practice) |
| Trust deed (supporting) | The constitutional document | Sets the three certainties in writing; governs powers, beneficiaries, governing law, amendment |
| Letter of wishes (supporting) | Non-binding guidance from settlor to trustee | Tells the trustee how the settlor *wishes* discretion to be exercised; deliberately not binding so the trust is not a sham and retains flexibility |
| Enforcer/beneficiary (supporting) | The party with standing to enforce | In charitable and purpose-trust contexts, enforcement may sit with a statutory body or named enforcer (§3.4, §3.5) |

### 1.4 Supporting Cast — Protector, Letter of Wishes, Trust Deed

Professional trust practice adds three instruments around the bare three-party skeleton:

- **The trust deed** — the constitution: names the parties, the trust property, the beneficiary class, the trustee's powers (investment, distribution, advancement, delegation), the governing law, the jurisdiction of administration, and whether the trust is revocable or irrevocable (§3.2). Singapore trusts commonly specify **Singapore law as governing law**, which brings the Trustees Act 1967 and the TCA regime into play.
- **The letter of wishes** — a settlor's non-binding memo of intent. Its deliberate non-binding character is what preserves the trustee's genuine discretion (and the trust's validity against sham attacks). In a discretionary family trust, the letter often covers *who* the settlor hopes will benefit, in what proportions, and for what purposes (education, housing, business succession, philanthropy).
- **The protector** — a family confidant, adviser, or committee with reserved powers (often: appoint/remove trustees, approve distributions to certain classes, amend the deed in defined ways). Singapore law has no statutory protector regime; protectors exist by contract in the deed (reported — standard industry practice, flagged as not statute-codified).

### 1.5 Why Singapore — the Jurisdiction's Trust Appeal

Singapore's trust franchise rests on a verified, mutually reinforcing stack:

- **A modern, English-derived trust law** — Trustees Act 1967, plus a steady stream of judicial decisions and the 2017 transparency reforms, gives families a predictable common-law environment with statutory clarity.
- **A regulated, credible trustee industry** — compulsory MAS licensing since 1 February 2006 (§2) means trust business is done by supervised, capitalised firms, not unregulated individuals.
- **The wealth stack** — the world's leading single-family-office jurisdiction (2,000+ SFOs as of end-2024 — cross-ref [singapore_private_markets_guide.md](singapore_private_markets_guide.md)), a S$6.7-trillion asset-management industry (2025, reported), no capital-gains tax, and the 13D/13O/13U fund incentives (§6) that make Singapore the natural home for the *trust plus fund* architecture Asian families now standardise on.
- **Fiscal neutrality for trusts** — Singapore does not tax trusts as entities; income is taxed at the beneficiary level on distribution (and the 13O/13U schemes exempt qualifying fund investment income entirely — §6). Estate duty was abolished in 2008 (reported — widely known, flagged as not re-confirmed here).
- **International standards** — FATF-aligned AML/CFT rules (Trustees Regulations 2017 Part 7, MAS Notice TCA-N03), a stable political and legal system, and the region's deepest pool of private banks, law firms and trust administrators.

For the banker's view of where this sits in the wealth stack: the trust layer sits *above* the private-banking and custody layer (the bank holds assets for the trustee) and *beside* the fund layer (the trust owns the fund vehicle's shares). [banks_in_singapore_guide.md](banks_in_singapore_guide.md) covers the banks; [singapore_private_markets_guide.md](singapore_private_markets_guide.md) covers the fund regime; this guide covers the trustee layer that connects them.

### 1.6 The Milestones Timeline

| Year | Singapore trust-landmark | Notes |
|---|---|---|
| 1885 | Trust Companies Act era begins | The pre-1967 trust-company statutes; voluntary registration under the Companies Act governs the industry for most of the 20th century |
| 1938 | OCBC Trustee (today's BOS Trustee) incorporated | One of the oldest trust companies in Singapore — 20 May 1938 (verified) |
| 1967 | Trustees Act 1967 enacted | The modern codification of trustee powers and duties — Singapore's core trust statute |
| 1985 | Trust Companies Act (Cap 336, 1985 Rev Ed) | The pre-reform statute, consolidated under voluntary registration |
| 2004 | Business Trusts Act 2004 passed | The registered-business-trust regime with trustee-managers (verified) |
| 28 Feb 2005 | Trust Companies Act 2005 assented | Repeal-and-re-enactment moving to compulsory licensing (verified — Gazette) |
| 1 Feb 2006 | TCA regime in force | Voluntary registration replaced by MAS compulsory licensing; 21 licences issued by mid-Dec 2006 (verified — Law Gazette) |
| 2017 | Trustees (Transparency and Effective Control) Regulations | Part 7 AML/CFT transparency regime: verification, 5-year records, S$20,000 notification threshold (verified) |
| 2020 | Variable Capital Companies Act in force | The VCC becomes the standard fund/family-office vehicle (cross-ref private-markets guide) |
| 2024–2025 | MAS SFO class-exemption framework; sanctions posture tightened | The 13O/13U application path reworked; Prince Holdings precedent (reported — flagged) |
| 2026 | FO tax rules eased, AML checks widened (Aug 2026) | Verified — The Business Times, cross-ref private-markets guide |

---

## 2. The Legal Regime

### 2.1 The Trust Companies Act 2005 (Verified)

The **Trust Companies Act 2005** (TCA, Cap 336, 2006 Revised Edition) is the statute that regulates the trust industry in Singapore. Verified facts:

- **Enactment:** the Act was passed in 2005 (assented 28 February 2005 per the Government Gazette) to "repeal and re-enact with amendments the Trust Companies Act (Chapter 336 of the 1985 Revised Edition) so as to provide for the licensing and regulation of trust companies and for matters connected therewith."
- **Commencement:** the new regulatory regime **became effective on 1 February 2006** (verified — Law Gazette, TCA 2020 Revised Edition text showing "[1 February 2006]").
- **The reform:** the TCA replaced the *voluntary registration* of trust companies under the Companies Act with a **compulsory licensing system administered by MAS**. The change was explicitly aimed not at altering the law of trusts (which remains the Trustees Act 1967 and case law) but at regulating the *financial institutions* that provide trust services for investment and wealth management — ensuring they are fit and proper and that money laundering is curtailed.
- **Structure:** the Act (82 sections plus schedules, per the 2020 Revised Edition) covers the licence itself (Part II), conduct and duties (Part III), the regulator's powers (including investigative powers, Part IV), and administrative provisions — plus the subsidiary **Trust Companies Regulations**, the **Trust Companies (Exemption) Regulations**, and the **Trust Companies (Appeals) Regulations**.
- **Key definitions and directions:** MAS issues Notices and Guidelines under the Act — the most important being **Notice TCA-N03** (AML/CFT obligations for licensed trust companies) and the Guidelines on Standards of Conduct and on Criteria for the Grant of a Trust Business Licence.

### 2.2 The Prohibition and the Definition of Trust Business (Verified)

The Act's core rule is a **prohibition with exceptions** (verified — Law Gazette 2007):

> No person other than a licensed trust company may carry on (or hold itself out as carrying on) **trust business** in or from Singapore, unless that person falls within a statutory exception or is exempted by the Act.

**"Trust business"** is broadly defined as the provision of services with respect to:
1. the **creation of an express trust**;
2. **acting as trustee** of, or **arranging for any person to act as trustee** of, an express trust; or
3. the provision of **trust administration services** in relation to an express trust.

**"Carrying on business"** is not defined in the Act, but MAS has indicated that any activity conducted with **system, repetition and continuity** counts — a profitable purpose is not critical. Everyday, non-commercial arrangements (an employee holding a car on trust for an employer, a friend declaring a trust over a share of an apartment) are clearly outside the Act's purpose.

**Statutory exceptions** (people who are *not* carrying on trust business at all, so the prohibition does not touch them — verified): a **bare trustee**; an **administrator or trustee of a business trust**; the **trustee-manager of a registered business trust** (under the Business Trusts Act 2004); a person **preparing or advising on a will**; and a person acting as **executor or administrator of a deceased person's estate** (including the consequential role as trustee of the estate).

**Exempt persons** (people who *are* carrying on trust business but are exempted from licensing for limited activities — verified): banks licensed under the Banking Act; approved merchant banks under the MAS Act; direct life insurers (added June 2006) — all exempt in respect of creating express trusts, arranging trustees, and providing "procedural and non-discretionary" trust administration services. Also exempt with conditions: **lawyers** (per-client settled financial assets ≤ S$2 million, ≤ 30 trust clients, Form 8 notification to MAS within one month), accountants, trustees of debentures, trustees of collective investment schemes, the Central Depository (CDP), custodians of collateral, and — critically for this guide — **private trust companies** under the Trust Companies (Exemption) Regulations (§4).

### 2.3 The MAS Licensing Regime — Licensed vs Registered (Verified)

The regime produces two broad categories, mirroring the Act's two routes:

**1. Licensed Trust Companies (LTCs)** — companies that hold a **trust business licence** from MAS under the TCA. Licensing criteria (verified — Act s 5(1) and MAS Guidelines): the applicant must be **fit and proper** (owners, directors, and management), have minimum financial resources (regulatory capital computed under the Trust Companies Regulations with reference to **qualifying assets** and the MAS Notice on Qualifying Assets and Reduction Percentages), have the organisational capability to run trust business, and be able to comply with MAS standards of conduct. Today's MAS Financial Institutions Directory lists licensed trust companies alongside two adjacent categories — **Exempt Trust Company** and **Approved CIS Trustee** (a trustee approved under the Securities and Futures Act for collective investment schemes — the category under which the bank-owned trustees hold their CIS mandates) — per the CapitalMarkets.SG registry summary. **The current number of licensed trust companies is not re-confirmed here** (flagged): the historical anchor is 21 trust business licences issued as of mid-December 2006 (verified — Law Gazette), and the count has grown since, but no single authoritative current figure is cited in this guide rather than guessed.

**2. Registered / exempt trust companies (the "registered" tier)** — persons who carry on (limited) trust business **without a licence** because they are statutorily exempt: the banks, merchant banks, life insurers, lawyers, and PTCs above. The historical record: as of mid-December 2006 there were **29 registered exempt persons — mainly banks, including four law firms** (verified — Law Gazette). Exempt persons must **notify MAS in Form 8 within one month** of commencing trust business and remain subject to MAS's powers (including AML/CFT directions) in respect of the exempt activities.

The two-tier design is deliberate: **institutional trust services for the public require a licence; captive and incidental trust services (a bank servicing its own customers, a family's PTC serving only its own family) are exempt but still notified and supervised.** That is the whole architecture of the Singapore trust industry in one sentence.

### 2.4 The Regime Table — Category / Who / Notes

| Category | Who | Notes |
|---|---|---|
| Licensed trust company (LTC) | Companies licensed by MAS under the TCA to carry on trust business for the public | Fit-and-proper test; regulatory capital against qualifying assets; subject to TCA-N03 AML/CFT and standards-of-conduct guidelines; e.g., DBS Trustee, BOS Trustee, HSBC Trustee (Singapore), and the independents (§5) |
| Approved CIS trustee | Trust companies approved as trustees of collective investment schemes under the SFA | The unit-trust trustee category (§3.3); bank-owned trustees typically hold this alongside their LTC licence (DBS Trustee, BOS Trustee — verified) |
| Exempt trust company | Statutorily exempt persons carrying on limited trust business: banks (Banking Act), approved merchant banks, direct life insurers | Exempt only for creating trusts, arranging trustees, and procedural/non-discretionary administration for their own customers; Form 8 notification to MAS within one month (verified) |
| Registered exempt persons | Lawyers, accountants, debenture trustees, CIS trustees, CDP, collateral custodians | Subject to conditions (lawyers: ≤ S$2m settled per client, ≤ 30 clients, Form 8); historical count 29 such persons (mainly banks + 4 law firms) as of Dec 2006 (verified) |
| Private trust company (PTC) | A family's own company acting as trustee only of connected trusts | Exempt under the Trust Companies (Exemption) Regulations reg 4; must engage an LTC for AML/CFT-related trust administration under reg 4(2) (§4) |
| Excepted persons (no licence needed) | Bare trustees; business-trust trustees; trustee-managers of registered business trusts; will preparers; executors/administrators | Not carrying on regulated trust business at all — outside the prohibition (verified) |

### 2.5 Compliance Machinery — Capital, AML/CFT, Notices

Beyond the licence itself, the regime's operating machinery (all verified as existing components; parameters flagged where date-sensitive):

- **Capital:** licensed trust companies must maintain financial resources at levels prescribed by the Trust Companies Regulations, computed with reference to their **qualifying assets** (the assets under trust administration) using the reduction percentages in the MAS Notice on Qualifying Assets and Reduction Percentages. In plain terms: **the bigger the trust book, the more capital the trustee must hold** — a direct incentive for prudent growth.
- **AML/CFT:** licensed and exempt trust companies are subject to **MAS Notice TCA-N03** (AML/CFT for trust companies) and, for the trustees themselves, the **Trustees Act 1967 Part 7** and the **Trustees (Transparency and Effective Control) Regulations 2017**: trustees of Singapore-law trusts (and trusts administered in Singapore, or with any Singapore-resident trustee) must verify and retain information on all trust parties and effective controllers, keep records **at least 5 years after ceasing to be trustee**, and **notify specified persons (financial institutions, lawyers, licensed estate agents, property developers) on business relationships or transactions above S$20,000** (verified — Anlian Group summary of the statutory requirements).
- **Reporting and supervision:** annual returns to MAS, audit requirements, MAS investigative powers, and the appeals regime (Trust Companies (Appeals) Regulations). MAS can impose conditions on licences and directions on exempt persons.
- **Standards of conduct:** the MAS Guidelines on Standards of Conduct for Licensed Trust Companies cover conflicts of interest, client-asset handling, and the duty to act in beneficiaries' interests — the fiduciary layer translated into supervisory rules.

### 2.6 Reading the Regime — a Two-Tier Gate

For a banker or family adviser, the regime reads as a **two-tier gate**:

- **Tier 1 (licensed)** — the public, commercial trust industry: bank-owned trustees and global independents that serve the market at scale, hold CIS trustee mandates, and absorb regulatory capital. These are the firms in the MAS Financial Institutions Directory under "Licensed Trust Company".
- **Tier 2 (exempt/registered)** — captive and incidental trust activity: banks serving their own customers, lawyers doing incidental trust work, and the family PTCs. These are *regulated but not licensed* — they must notify, comply with AML/CFT directions, and (for PTCs) outsource the compliance-critical administration to a licensed firm.

The gate matters commercially: **licence counts are a scarce-resource signal.** MAS does not hand out trust licences liberally — the historical run-rate (21 licences two years into the regime; a modest count today, flagged) reflects deliberate selectivity. That scarcity is why a family that wants trustee *control* without a licence builds a PTC (§4), and why the licensed firms that do exist are effectively franchise-holders of a regulated oligopoly — a structural fact that shows up in the market map of §5.

### 2.7 The Regime in Practice — What a Licence Costs and Requires

For a firm considering a Singapore trust licence (or a family evaluating whether its LTC partner is sound), the operational picture (components verified; parameters flagged as date-sensitive):

- **The application:** a written application to MAS with the corporate structure, ownership, directors and senior management (fit-and-proper scrutiny), a business plan, and the compliance programme. MAS Guidelines on Criteria for the Grant of a Trust Business Licence set the expectations; approval is discretionary and conditional.
- **Ongoing capital:** licensed trust companies must maintain financial resources tied to qualifying assets under administration (Trust Companies Regulations + the MAS Notice on Qualifying Assets and Reduction Percentages) — the "bigger book, bigger capital" rule of §2.5. This is the moat that keeps small, weakly capitalised trustees out of the public business.
- **Conduct obligations:** the MAS Guidelines on Standards of Conduct for Licensed Trust Companies (client-asset handling, conflicts, beneficiary-first conduct), annual returns and audits, and notification of material changes (shareholders, directors, business scope).
- **The AML stack:** TCA-N03-aligned policies, screening of settlors/beneficiaries/effective controllers, record retention, and the S$20,000 transaction-notification duty under the Trustees Regulations 2017 (verified).
- **What it costs (reported — directional):** industry commentary puts a Singapore LTC's annual compliance and capital burden in the six-figure range once professional staffing, audit, and capital are counted (flagged — figures vary widely by book size and are not official). This cost base is precisely why PTCs (§4) and the reg-4(2) engagement model exist: the family gets fiduciary control without carrying a licence's fixed cost, and the LTC monetises its licence by servicing many PTCs.

---

## 3. The Trust Types

### 3.1 Discretionary Trusts (Verified)

A **discretionary trust** is one in which the trustee has **discretion** over *which* beneficiaries receive distributions and *how much*, within a defined beneficiary class (often "my children and remoter issue", or a class including charities). No beneficiary has a vested entitlement until the trustee exercises its discretion; each beneficiary holds only a **mere expectancy** (a right to be considered). The settlor's **letter of wishes** (§1.4) guides — but does not bind — the trustee's exercise of discretion.

Why it dominates Singapore family practice: flexibility across generations (the trustee can adapt to marriages, divorces, births, business fortunes), **asset protection** (no beneficiary can point to a fixed entitlement that a creditor could attach), tax-planning latitude (income can be routed to the beneficiaries in the lowest tax position), and **privacy** (no beneficiary needs to be named publicly). The classic structure in this guide's worked example (§7) is discretionary and irrevocable.

### 3.2 Revocable vs Irrevocable Trusts (Verified)

The **revocable/irrevocable** distinction is the second axis of trust design:

- **Revocable trust:** the settlor reserves the power to revoke, vary, or amend the trust during their lifetime. Common in *US-style* estate planning (and in Singapore for certain holding structures where the settlor wants flexibility). Consequence: the settlor retains a degree of control — which weakens asset protection (creditors can generally reach assets the settlor can revoke back) and can expose the trust to taxation issues in the settlor's hands.
- **Irrevocable trust:** the settlor permanently parts with the assets; the trust cannot be amended or revoked except in narrowly defined circumstances (often with beneficiary consent and a protector's concurrence). This is the **default for serious asset protection and succession planning** in Singapore: the settlor genuinely steps away, the trust is robust against future claims (divorce, insolvency, estate disputes), and the legal-ownership split is clean.

The tax and estate consequences differ materially: assets in an irrevocable trust are generally outside the settlor's estate (relevant since Singapore abolished estate duty in 2008 — reported), while revocable-trust assets may remain attributable to the settlor. Trust deeds specify the axis explicitly; the choice is one of the first questions a family adviser asks.

### 3.3 Unit Trusts — the Collective-Investment-Scheme Trustee (Verified)

A **unit trust** is a form of **collective investment scheme (CIS)** under the Securities and Futures Act: investors pool money into a fund; a **trustee** holds the fund's assets while a **manager** runs the investments; investors hold **units** representing their share of the fund's net asset value. The trustee is the investor-protection linchpin: it holds the assets separately from the manager, oversees NAV calculations, and acts in unitholders' interests.

This is where trust companies and the fund industry meet directly. The **Approved CIS Trustee** category (MAS approval under the SFA) is held by the bank-owned trustees — DBS Trustee and BOS Trustee are both licensed trust companies *and* approved CIS trustees (verified — firm statements and the MAS FID registry). In the unit-trust chain: the fund manager (CMS-licensed, per [singapore_private_markets_guide.md](singapore_private_markets_guide.md) §5.7), the trustee (this guide's subject), the custodian (often the bank group itself), and the administrator. For a banker, the CIS-trustee business is a natural extension of custody and fund-services — see [financial_infrastructure_guide.md](financial_infrastructure_guide.md) and [asset_management_alternatives_guide.md](asset_management_alternatives_guide.md).

### 3.4 Purpose Trusts (Verified, Flagged)

A **purpose trust** is a trust established for a **purpose rather than for beneficiaries** — e.g., holding shares in a family business to keep control consolidated, or holding an asset for a specific non-charitable objective. English-derived trust law (which Singapore follows) has a general **rule against non-charitable purpose trusts**: a trust needs identifiable beneficiaries to be enforceable (the beneficiary principle), so *pure* purpose trusts are not generally valid without statutory backing. Verified/flagged position for Singapore:

- **No general statutory purpose-trust regime** exists (unlike the BVI VISTA trust or Labuan foundations) — flagged as the widely understood position, not re-verified against case law here.
- **Statutory carve-outs do exist:** business trusts are recognised under the **Business Trusts Act 2004** (§3.6) — a registered business trust is a purpose-like vehicle with unitholders; and the **PTC exemption** expressly contemplates a PTC acting as trustee of purpose trusts in defined circumstances (the Trust Companies (Exemption) Regulations framework — verified via the exemption-regulations text, which defines "charity" in the PTC context and contemplates non-beneficiary trusts, flagged for detail).
- **Practice:** families wanting offshore-style purpose vehicles typically combine a Singapore trust *with* an offshore purpose trust/foundation (BVI, Labuan, Jersey) for the shareholding layer, or use the Singapore PTC to achieve the *governance* outcome (family control of a trustee) that purpose trusts are often used for offshore. The distinction matters for advisers: Singapore is a *beneficiary-trust* jurisdiction with targeted statutory exceptions, not a purpose-trust jurisdiction.

### 3.5 Charitable Trusts (Verified)

A **charitable trust** is a trust for purposes recognised as charitable in law (relief of poverty, advancement of education, advancement of religion, and other purposes beneficial to the community). Verified anchors:

- Charitable trusts are **enforceable without private beneficiaries** — the beneficiary principle is relaxed because the Attorney-General (and the **Commissioner of Charities**, under the Charities Act) polices them. They are the standard vehicle for structured philanthropy.
- **Registration:** charities in Singapore (including charitable trusts) must register with the Commissioner of Charities if their income exceeds thresholds (reported — registration requirements are date-sensitive, flagged); the Charities Act regime sits alongside the trust law.
- **Tax:** donations to registered charities enjoy tax deductions (reported — IRAS-administered); the charitable trust itself is not taxed on its income (reported, flagged).
- **PTC relevance:** the PTC exemption (reg 4) expressly permits a PTC to act as trustee of **charitable trusts** settled in connection with the family — so a family's philanthropy arm can sit inside its PTC structure without needing a separate licensed trustee (§4, §7).
- **Cross-ref:** the family-office ecosystem routinely pairs a 13O fund (wealth) with a charitable trust (giving) — the two halves of a family's capital. See [singapore_private_markets_guide.md](singapore_private_markets_guide.md) for the fund side.

### 3.6 Business Trusts (Verified)

A **business trust** is a trust that carries on a **business** (as opposed to holding passive assets), with **units** issued to investors — regulated in Singapore by the **Business Trusts Act 2004** (BTA). Verified anchors:

- The BTA governs **registered business trusts**: a trust that meets statutory criteria and is registered under the Act. The regulated party is the **trustee-manager** — a company that both holds the trust property *and* manages the business, responsible for governance, disclosure, and compliance.
- Business trusts are a **listed-infrastructure staple** in Singapore: they let operating assets (utilities, real estate, infrastructure) be financed by unit-holders while the sponsor retains management via the trustee-manager. The best-known examples are listed on the SGX (e.g., utilities and infrastructure trusts — reported, names flagged as illustrative).
- **Trust-company angle:** the trustee-manager of a registered business trust is **excepted** from the TCA trust-business prohibition (verified — statutory exception), so a business trust does not require a licensed trust company as trustee; the trustee-manager itself is the regulated entity under the BTA and the Securities and Futures Act. This is why "business trust" lives in the capital-markets guide family rather than this one — see [financial_infrastructure_guide.md](financial_infrastructure_guide.md) for the market infrastructure it plugs into.
- **Distinction to keep clean:** a *unit trust* (§3.3) is a CIS whose trustee holds assets for passive pooled investment; a *business trust* is an operating-business trust with a trustee-manager under the BTA. Both issue units; neither is a family wealth vehicle — family wealth uses the discretionary trust (§3.1) and the PTC (§4).

### 3.7 The Types Table — Type / Use / Notes

| Type | Use | Notes |
|---|---|---|
| Discretionary trust | Family wealth, succession, asset protection | Trustee chooses beneficiaries/amounts within a class; letter of wishes guides; the default Singapore family vehicle (verified concept) |
| Revocable trust | Flexibility; holding structures where the settlor wants to retain the power to amend/revoke | Weaker asset protection; assets may remain attributable to the settlor; uncommon for hard succession planning |
| Irrevocable trust | Hard asset protection; estate planning; multi-generational succession | Settlor parts with assets permanently; the default for the worked example (§7) |
| Unit trust (CIS) | Pooled retail/institutional investment funds | Trustee = investor-protection linchpin under the SFA; Approved CIS Trustee category; bank-owned trustees hold these mandates (verified) |
| Purpose trust | Holding for a non-beneficiary purpose (e.g., shareholding consolidation) | Not generally valid under SG common law; statutory carve-outs (business trusts; PTC contexts); offshore purpose vehicles commonly paired with SG structures (flagged) |
| Charitable trust | Structured philanthropy; family foundations | Enforceable without private beneficiaries (Commissioner of Charities); PTC may serve as trustee of connected charitable trusts (verified) |
| Business trust | Operating-business financing via units (infrastructure, utilities) | Business Trusts Act 2004; trustee-manager regulated under the BTA/SFA; trustee-manager excepted from TCA licensing (verified) |

### 3.8 The Tax Angle — How Trusts Are Taxed in Singapore

The tax treatment of trusts is what makes Singapore's structure attractive (reported — IRAS-administered rules; flagged for date-sensitivity, cross-ref the private-markets guide's 13O/13U treatment):

- **No trust-level tax on distribution:** Singapore taxes trusts on a **conduit/beneficiary basis** — income is generally taxed when it flows to beneficiaries (at their rates), not at the trust as an entity. Accumulated income retained in the trust can be taxed at the trustee level in defined cases (reported — flagged).
- **No capital-gains tax and no estate duty:** Singapore does not tax capital gains generally, and **estate duty was abolished in 2008** (reported — widely documented). For an irrevocable discretionary trust, this means assets settled into the trust can grow and pass across generations without the estate-duty drag that exists in many Western jurisdictions.
- **The 13O/13D/13U schemes:** for the *investment-fund* layer (the VCC holding the family's portfolio), qualifying investment income is **exempt under the fund schemes** of §6 — the tax engine that runs alongside the trust chassis.
- **Foreign-source and offshore nuances:** foreign-source income remitted by non-resident trustees, and the treatment of trusts with offshore settlors/beneficiaries, carry their own IRAS rules (reported — flagged; engage a tax adviser, which is precisely the professional ecosystem this guide's readers serve).
- **The adviser's takeaway:** the Singapore trust stack is *tax-neutral by design* — the value is in succession, protection and governance, not in aggressive avoidance. The incentives (13O etc.) are administered, conditioned, and policed — which is why the compliance discipline of §6.4 is inseparable from the structure.

---

## 4. The Private Trust Companies

### 4.1 What a PTC Is — the Family's Own Trustee (Verified)

A **private trust company (PTC)** is a company — typically a Singapore private company limited by shares — that acts as **trustee only of a defined set of trusts connected to a family or group**, rather than offering trust services to the public. In the Singapore structure, the family (through a holding arrangement) owns the PTC; the PTC is the trustee of the family's trusts; and the family's principals sit on the PTC's board (often alongside professional directors). The PTC gives the family **trustee control without a trust licence**.

Verified anchors:

- The PTC's status is defined by the **Trust Companies (Exemption) Regulations** under the TCA: a private trust company is **exempt from holding a trust business licence** where it acts as trustee only of trusts settled in connection with the company's owners/their families (the exemption's scope: trusts for related parties, employees, charitable trusts, and purpose-trust contexts — flagged for the precise reg-4 drafting, which is conditions-heavy).
- **Regulation 4(2)** (verified — Singapore Statutes Online): the PTC **must engage a licensed trust company to carry out trust administration services** for the purposes of conducting the necessary checks to comply with any written direction issued by MAS on the prevention of money laundering or countering the financing of terrorism. In plain terms: **the PTC runs the trust; a licensed trustee runs the AML/compliance plumbing.** This is the single most important design fact in the PTC world — the exemption is not a licence-free-for-all.
- PTCs must also comply with the TCA's exemption conditions (notification/registration with MAS — the exempt-person framework of §2.3), the Trustees Act 1967 Part 7 transparency rules (Trustees Regulations 2017), and any MAS directions on AML/CFT.

### 4.2 The MAS Exemption and Its Conditions (Verified)

The exemption logic (verified via the exemption-regulations text and industry commentary — Kensington Trust, Anlian Group):

1. **Why exempt at all:** the TCA regulates *public* trust business. A PTC serves only its own family's trusts — there is no consumer to protect, so full licensing is disproportionate. The exemption keeps captive trustees out of the licence queue while still pulling them into the AML/CFT net.
2. **The conditions in substance:**
   - The PTC is a **company** (typically Singapore-incorporated, private, limited by shares).
   - It acts as trustee **only of trusts connected to the family/group** — i.e., settled by the owners or their connected persons, for the benefit of family members, employees, charitable purposes, or the defined purpose-trust cases (flagged for the reg's exact list).
   - It does **not** hold itself out as carrying on trust business for the public.
   - It **engages a licensed trust company** for the trust-administration services needed to comply with MAS AML/CFT directions (reg 4(2)) — the mandated outsourcing.
   - It **notifies MAS** as an exempt person (Form 8 process, §2.3) and remains subject to MAS powers and directions.
3. **Why the engagement requirement matters commercially:** every PTC structure generates a **recurring revenue stream for the licensed trust companies** — the "shadow trustee" role. This is a deliberate regulatory design: families get control; licensed firms get a mandated seat at the table (usually as administrator/co-trustee or AML-services provider); MAS gets a supervised chain with no unregulated gap. It also means the *licensed* players' client lists are much broader than the trusts they formally trustee.

### 4.3 Why Families Use PTCs

- **Governance and continuity:** the PTC board (family members + trusted advisers) is the permanent decision-making body for the family's trusts — no dependence on the personnel, policies, or risk appetite of an outside trustee. Succession of *trustee control* is built into the family's own governance.
- **A unified family trustee:** one PTC can serve many trusts (a main discretionary trust, a charitable trust, a purpose-style holding trust, an employee-trust for the family business) — one board, one policy, one compliance posture.
- **Privacy:** the PTC's trust relationships are not publicly searchable the way a licensed trustee's client book might be; the family's affairs stay within the family's company.
- **Control without sham:** done properly, the PTC lets the family *govern* without the settlor *controlling* the assets — the settlor's powers are reserved in the deed (if at all), the PTC board acts as fiduciary, and the licensed trustee's involvement evidences genuine third-party oversight. This is what keeps the structure defensible against sham challenges and creditor attack (§1.2, §7.4).
- **Cost at scale:** for a family with several trusts and meaningful AUM, a PTC's fixed costs (company, board, compliance services from the LTC partner) are lower than paying a licensed trustee's percentage-based fees on every trust.

### 4.4 The PTC Table — Feature / Detail / Notes

| Feature | Detail | Notes |
|---|---|---|
| What it is | A private company acting as trustee only of connected (family) trusts | Singapore-incorporated, limited by shares (verified concept) |
| Legal basis | Exemption under the Trust Companies (Exemption) Regulations, reg 4 | Not a licence — an exemption from the TCA licensing requirement (verified) |
| Scope of trusteeship | Trusts settled by the owners/connected persons: family, employees, charitable, defined purpose contexts | Conditions-heavy drafting — flagged for the reg's exact list |
| Mandated LTC engagement | Must engage a licensed trust company for trust administration re MAS AML/CFT directions (reg 4(2)) | The compliance backbone; a recurring revenue line for LTCs (verified) |
| Notification | Exempt-person notification to MAS (Form 8 within one month of commencement) | Same discipline as banks' exempt trust activity (§2.3) |
| AML/CFT | Trustees Act 1967 Part 7 + Trustees Regulations 2017; MAS directions; TCA-N03 alignment | 5-year record retention; S$20,000 notification threshold (verified) |
| Who owns it | The family (via holding structure); family members + advisers on the board | Often a protector/family-council layer above the board (reported practice) |
| Why | Trustee control, governance continuity, privacy, cost at scale | The "control without a licence" vehicle |
| Cost profile | Setup + annual (company, board, compliance, LTC engagement) | Fixed-cost model vs percentage fees of a full-service LTC (reported — directional) |

### 4.5 PTC vs Licensed Trust Company

| | PTC | Licensed trust company |
|---|---|---|
| Licence | None — exemption under reg 4 | MAS trust business licence under the TCA |
| Client scope | Only connected family/group trusts | The public: any settlor, any trust book |
| Regulator relationship | Notifies MAS; subject to directions; must engage an LTC for AML administration | Directly licensed, capitalised, supervised; MAS FID listing |
| Governance | Family board + advisers | Independent professional board/management |
| Public trust business | No | Yes — the franchise |
| Fees | Cost-recovery, family-owned | Market fees (often ad valorem) |
| Best for | A family with multiple trusts and long-horizon governance | Commercial trusteeship, CIS mandates, third-party settlors, the reg-4(2) engagement role |

The synthesis: **PTC and LTC are complements, not rivals.** The LTC industry's PTC-engagement business (reg 4(2)) is a structural feature of the regime, and the worked example in §7 uses both layers by design.

---

## 5. The Market Players

### 5.1 The Bank-Owned Trustees (Verified)

The most visible tier of the Singapore trust industry is owned by the banking groups — each major bank runs a trustee subsidiary that is a licensed trust company (and usually an approved CIS trustee), serving the bank's private-banking clients and its fund business:

- **DBS Trustee Limited** (verified — dbs.com.sg and MAS FID): a Singapore-incorporated company **wholly owned by DBS Group Holdings**, one of the largest banking groups in the region; a **licensed trust company regulated by MAS** and an **approved trustee under the Securities and Futures Act**. Serves DBS Treasures Private Client and DBS Private Bank clients with trust structures for legacy/succession planning. Parent context: [dbs_bank_guide.md](dbs_bank_guide.md), [banks_in_singapore_guide.md](banks_in_singapore_guide.md).
- **BOS Trustee Limited** (verified — Bank of Singapore site, Companies.sg, MAS FID): **formerly known as OCBC Trustee**; incorporated **20 May 1938** — one of the **oldest trust companies in Singapore**; a wholly-owned subsidiary of **Bank of Singapore** within the **OCBC Group**; a **licensed trust company and approved CIS trustee**. It is the OCBC group's dedicated trustee (the task's "OCBC Trust" is today's BOS Trustee after the Bank of Singapore rebrand — noted for the name-mapping). Parent context: [ocbc_software_systems_guide.md](ocbc_software_systems_guide.md).
- **UOB** — the United Overseas Bank group runs its trust business through its private-banking/wealth arm; the specific trustee entity name (reported as a UOB-group trustee company in MAS records) is **flagged — not independently re-confirmed here**. Treat the UOB trust franchise as real but the entity detail as unverified in this guide.
- **HSBC Trustee (Singapore) Limited** (verified — MAS FID): a **licensed trust company** based at Marina Bay Financial Centre Tower 2; the HSBC group's Singapore trustee. Parent context: [hsbc_software_systems_guide.md](hsbc_software_systems_guide.md).
- **Standard Chartered Trust (Singapore) Limited** — the Standard Chartered group's Singapore trustee arm (reported; **flagged** — not independently re-confirmed in this research pass). Note the amusing adjacency: Standard Chartered is the majority owner of *Trust Bank the digital bank* ([trust_bank_guide.md](trust_bank_guide.md)) *and* runs a trust company — two different "Trust" businesses under one group.

Also present in the bank-owned tier (reported, flagged): **Cititrust (Singapore)**, **BNP Paribas Trust Corporation Singapore**, and other international banks' local trustee subsidiaries serving their Asian private-banking clients — the pattern being that every serious private bank in Singapore needs a local licensed trustee to offer trust services.

### 5.2 The Independents — Global Trust & Corporate Services (Verified, Flagged)

The independent tier is dominated by global corporate-services and fund-services groups with licensed trust businesses in Singapore:

- **Vistra** (verified — vistra.com, EQT release): one of the world's leading fund and corporate services providers; in 2023–2024 completed its **merger with Tricor Group** (both owned by Baring Private Equity Asia / EQT) to become a global industry leader — the flagship consolidation of the sector. Vistra's Singapore trust business is part of its global trust network.
- **Tricor** — the Asia business-expansion specialist; **now part of Vistra** (verified). Included separately in this guide only because legacy references still list it as an independent (the merger is the canonical market event for the sector).
- **TMF Group** (reported, flagged for detail): global corporate/trust services with a large Singapore operation.
- **Intertrust (now CSC)** — Intertrust was a leading global corporate/trust and fund-services group with a Singapore trust licence; it was **acquired by CSC** (the US corporate-services giant) — the transaction completed in 2023 (reported; exact completion date flagged). Today's entity is the combined CSC/Intertrust Singapore business.
- **Asiaciti** — Asia-Pacific independent trust company with a Singapore presence (reported, flagged for detail).
- **Apex Group** — global fund administration and corporate services with a Singapore trust/fiduciary offering (reported, flagged).
- **Ocorian** — global trust, fund and corporate services group (post-2021 Elian merger) with Singapore operations (reported, flagged).
- **JTC Group** — Jersey-headquartered trust/corporate services group with Singapore operations (reported, flagged).
- **Portcullis Trust (Singapore) Ltd** (verified — MAS FID registry entry surfaced in this research): a licensed trust company **and** approved CIS trustee at Hong Leong Building, Raffles Quay — a representative of the mid-tier independent trustees.

**Counting caveat (flagged):** the *current* number of licensed trust companies in Singapore is not quoted in this guide — no single authoritative figure was re-confirmed in this research pass. The verified historical anchor is **21 trust business licences as of mid-December 2006** (Law Gazette). Industry coverage (CapitalMarkets.SG and MAS FID) confirms the LTC population is larger today and includes the names above, but a precise count is left to the reader's check of the MAS Financial Institutions Directory (eservices.mas.gov.sg → FID → "Licensed Trust Company").

### 5.3 The Players Table — Firm / Type / Notes

| Firm | Type | Notes |
|---|---|---|
| DBS Trustee Limited | Bank-owned (DBS Group Holdings) | LTC + SFA-approved CIS trustee; serves DBS Private Bank/TPC clients (verified) |
| BOS Trustee Limited | Bank-owned (Bank of Singapore / OCBC Group) | Formerly OCBC Trustee; incorporated 1938; LTC + approved CIS trustee; one of the oldest trust companies in SG (verified) |
| UOB group trustee | Bank-owned (UOB) | Entity name flagged — not independently re-confirmed |
| HSBC Trustee (Singapore) Limited | Bank-owned (HSBC) | LTC at MBFC Tower 2 (verified — MAS FID) |
| Standard Chartered Trust (Singapore) Limited | Bank-owned (Standard Chartered) | Reported — flagged; same group as the Trust Bank digital-bank JV (name-collision §0) |
| Cititrust / BNP Paribas Trust Corp (SG) | Bank-owned (internationals) | Reported — flagged; the international-bank trustee pattern |
| Vistra | Independent (global corporate/trust services) | Merged with Tricor (BPEA EQT deal) — the sector's flagship consolidation (verified) |
| Tricor | Independent (Asia corporate services) | Now part of Vistra (verified) |
| TMF Group | Independent | Global corporate/trust services; SG operation (reported — flagged) |
| Intertrust (now CSC) | Independent | Acquired by CSC, completed 2023 (reported — completion date flagged) |
| Asiaciti | Independent (Asia-Pacific) | SG presence (reported — flagged) |
| Apex Group | Independent | Fund admin + trust/fiduciary; SG offering (reported — flagged) |
| Ocorian | Independent | Global trust/fund/corporate services; SG operation (reported — flagged) |
| JTC Group | Independent | Jersey-headquartered; SG operation (reported — flagged) |
| Portcullis Trust (Singapore) Ltd | Independent (mid-tier) | LTC + approved CIS trustee (verified — MAS FID) |
| Singapore Trustees Association (STA) | Industry body | Represents trust companies in SG; advocacy on legislation; conferences (verified — sta.org.sg) |
| STEP | Professional body | Society of Trust and Estate Practitioners — global body with an active Singapore branch; the trust profession's credentialing body |

### 5.4 Reading the Market — Consolidation and the Two Tiers

Two structural facts organise the market (both flagged as *directional* but well evidenced):

1. **Consolidation is the story of the independent tier.** Vistra+Tricor (BPEA EQT) and CSC+Intertrust are the marquee deals; the global corporate-services industry has been rolling up relentlessly (a trend running in parallel with the banking consolidation in [banks_in_singapore_guide.md](banks_in_singapore_guide.md)). For families, this cuts both ways: bigger platforms mean deeper compliance and global reach; they also mean the *independent, family-responsive* trust shop is a scarcer species — which is one more reason PTCs (§4) exist.
2. **The bank-owned trustees are distribution-led, the independents are relationship-led.** The bank trustees exist to serve the group's private-banking clients and CIS/fund mandates (they are part of the wealth stack that [singapore_private_markets_guide.md](singapore_private_markets_guide.md) and [banks_in_singapore_guide.md](banks_in_singapore_guide.md) describe); the independents win business on fiduciary expertise, global multi-jurisdiction structuring, and the reg-4(2) PTC-engagement business. A family's choice between them is usually a choice between *banking relationship* and *independent specialisation* — or, increasingly, both (bank for custody and credit, independent for trusteeship).

For the banker's lens: the trust companies are simultaneously **clients** (they need custody, banking, FX, lending against trust assets) and **competitors** (their CIS-trustee and wealth-structuring businesses sit next to the bank's own). The interplay is a classic universal-banking adjacency — see [universal_banking_model_guide.md](universal_banking_model_guide.md).

### 5.5 The Economics — How Trust Companies Make Money

Understanding the trust industry's revenue model explains its behaviour (reported — standard industry practice; fee scales flagged as directional):

- **Trustee/administration fees:** usually **ad valorem** (a percentage of assets under administration, commonly in the low-basis-points range for large books, higher for bespoke structures) or fixed annual fees per trust. The PTC-engagement business (reg 4(2)) is billed as annual administration/AML services per trust — the LTC's recurring, low-touch revenue line.
- **CIS trustee fees:** the bank-owned trustees earn trustee fees on the unit trusts and funds they oversee — a volume business tied to the fund industry's growth (cross-ref the private-markets guide's AUM trajectory: S$5.4T in 2023 → S$6.7T in 2025, reported).
- **Setup and structuring fees:** one-off fees for settling trusts, drafting deeds, and designing PTC/governance layers — the entry ticket for every new family structure.
- **Adjacent revenue:** referrals into the owning bank's private-banking, custody, treasury and lending businesses (the bank-owned trustees' real strategic value) or, for independents, into corporate services, fund administration and global entity management (Vistra/TMF/Intertrust-style platforms).
- **The strategic logic:** for banks, the trustee subsidiary is a **client-retention and cross-sell weapon**, not primarily a profit centre; for independents, trust fees are core revenue within a broader corporate-services bundle. This difference explains why bank trustees rarely compete aggressively on price (they want the whole banking relationship) while independents compete on fiduciary depth and multi-jurisdiction capability (§5.4).

---

## 6. The Family-Office Interplay

### 6.1 The 13O/13D/13U Regime in One Paragraph (Cross-Ref)

The Singapore family-office story is fundamentally a **tax-incentive story wrapped in a fund-vehicle story**, and the definitive treatment is [singapore_private_markets_guide.md](singapore_private_markets_guide.md) §5–§6. The one-paragraph version for this guide:

- **Section 13O** (Income Tax Act) — the onshore fund tax-exemption scheme: a Singapore-incorporated fund vehicle (typically a **VCC**) managed by a Singapore-based fund manager gets tax exemption on qualifying investment income. Successor to 13R; schemes extended to **31 December 2029** (verified — cross-ref). Parameters (minimum fund size, local-spend and hiring conditions, the 2024–2025 class-exemption framework for SFOs) are date-sensitive — flagged.
- **Section 13D** — the dedicated single-family-office (SFO) scheme: a fund vehicle owned and controlled by one family, managed by the family's own Singapore-based manager, no third-party investors. The scheme most of the **2,000+ SFOs** (end-2024 count, verified — cross-ref) associate with (scheme-usage mix not published — flagged).
- **Section 13U** — the enhanced tier for vehicles that stay offshore (existing Cayman/BVI structures, non-SG co-investors): exemption without SG incorporation/manager requirements, **S$50 million minimum** (reported — flagged), local substance conditions.
- **The scale:** S$5.4 trillion SG asset-management industry (2023 MAS survey — reported) → S$6.7 trillion (2025 — reported, cross-ref); 2,000+ SFOs (end-2024, verified; 2,000+ end-2025 reported). Singapore is the world's leading SFO jurisdiction, and MAS has been tightening the SFO framework (class exemption 2024/2025, eased FO tax rules + widened AML checks August 2026 — verified via cross-ref).

### 6.2 How the Trust and the Fund Scheme Fit Together (Verified)

The trust and the 13O/13D fund scheme are **complementary, not substitutes** (verified — Anlian Group's framing, echoed across the industry):

- **Different jobs:** the **trust** solves *succession, asset protection, privacy and governance* (who owns the family's wealth across generations, and how); the **13O/13D scheme** solves *tax efficiency* (what happens to the qualifying investment income the family's money earns). One is a legal-ownership architecture; the other is a tax wrapper.
- **The canonical combination:** the trust **owns the equity in the family-office fund manager** (and/or the fund vehicle). The next generation inherits **trust interests** rather than direct shares in the family company — so the ownership of the operating wealth structure passes under the trust's governance, while the fund vehicle earns tax-exempt income under 13O. Structure: *Settlor → Family trust (trustee: PTC) → owns SFO manager (13D) / fund vehicle (VCC + 13O) → investments, private-bank custody*.
- **Why this is the standard:** it keeps the family business/fund ownership out of probate, shields it from matrimonial and creditor claims on individual family members, keeps the tax-exempt vehicle owned by a stable fiduciary rather than by a changing cast of individual shareholders, and lets the family's governance (PTC board + family council) outlive any individual.
- **The regulatory weave:** the trustee side carries its own AML/CFT duties (Trustees Act Part 7, TCA-N03, S$20,000 notification threshold); the SFO side carries MAS's scheme conditions and the 2025 sanctions-exposure posture (after the Prince Holdings precedent — reported, flagged). A structure that is tax-optimised but governance-sloppy fails both regulators' tests.

### 6.3 The Interplay Table — Layer / Structure / Notes

| Layer | Structure | Notes |
|---|---|---|
| Beneficial ownership | The family (beneficiaries of the trust) | Settlor steps back; beneficiaries hold equitable interests only |
| Trust layer | Discretionary, irrevocable family trust; trustee = PTC (family board) + engaged LTC for reg 4(2) AML administration | Succession, asset protection, privacy; the PTC/LTC partnership of §4 |
| Company layer | PTC (trustee company) and SFO manager company owned by the trust | Trust holds equity in the manager — the canonical combination (verified) |
| Fund layer | VCC (or LP) fund vehicle; 13D/13O/13U tax scheme | The fund regime of [singapore_private_markets_guide.md](singapore_private_markets_guide.md) §5; 13O for onshore, 13U for offshore holdings |
| Management layer | CMS-licensed / exempt / VCFM manager (often the family's own) | Licensing routes per the private-markets guide §5.7 |
| Banking layer | Private bank: custody, treasury, lending; the trustee's bank accounts | [banks_in_singapore_guide.md](banks_in_singapore_guide.md); the bank-owned trustees of §5.1 are both service providers and competition |
| Philanthropy layer | Charitable trust (often under the same PTC) | Tax-deductible giving; PTC exemption covers connected charitable trusts (verified) |
| Compliance layer | Trustees Act Part 7 transparency; TCA-N03; SFO scheme conditions; sanctions due diligence | The discipline that holds the whole stack together |

### 6.4 The AML/CFT Discipline Across Both Layers

The trust+13O stack is only as good as its compliance backbone, and MAS has been explicit that the two regimes are policed together (verified threads):

- **Trustees** (licensed and PTCs) must verify and retain information on all trust parties and effective controllers, keep records 5 years post-trusteeship, and notify specified persons on relationships/transactions above **S$20,000** (Trustees Regulations 2017 — verified).
- **LTCs and PTCs** carry the equivalent obligations under **MAS Notice TCA-N03** (verified — the notice exists and is the operational AML instrument for the sector).
- **SFO scheme-holders** face MAS's 2024–2025 class-exemption framework (refreshed legal opinions, calibrated substance/minimum-spend conditions — reported) and the **2025 sanctions-exposure posture** for tax-incentive recipients (reported — flagged): ongoing due diligence on settlors, beneficiaries and effective controllers is expected, not optional.
- The identity/compliance infrastructure this runs on is covered in [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md) and [../technology/distributed_auth_guide.md](../technology/distributed_auth_guide.md); the corporate-service-provider perimeter that adjoins the trust regime is the Corporate Service Providers Act 2024 regime (reported — flagged).

### 6.5 The Regulatory Timeline — Trusts and FOs Converging

| Period | Regulatory development | Notes |
|---|---|---|
| Pre-2006 | Trust business under voluntary Companies Act registration | The regime this guide's §2 replaced |
| 1 Feb 2006 | TCA compulsory licensing in force | Trust companies become regulated fiduciaries (verified) |
| 2017 | Trustees Act Part 7 + Trustees Regulations 2017 | Trustee-level AML/CFT transparency (verified) |
| 2019–2020 | VCC Act; 13R/13X era matures | The fund-vehicle layer arrives; SFO scheme usage accelerates (cross-ref) |
| 2023 | MAS SFO consultation; response published | Framework for SFOs under formalisation (reported) |
| 2024 | New Sections 13D/13O/13OA/13U effective (successors to 13R/13X); 13OA for SG LPs | Verified via cross-ref (DLA Piper) |
| 2024–2025 | SFO class-exemption framework; refreshed legal opinions; substance conditions calibrated | Reported — flagged |
| 2025 | Sanctions-exposure posture after Prince Holdings parliamentary reply; MinLaw guidance on Trustees Regulations | Reported — flagged |
| Aug 2026 | FO tax rules eased; AML checks widened | Verified — The Business Times (cross-ref) |
| To 31 Dec 2029 | 13D/13O/13U schemes extended | Verified — cross-ref private-markets guide |

The converging trajectories are the story: trust law (1967–2017) and the FO/fund incentives (2004–2026) started as separate regimes and have **merged into one supervisory posture** — the trustee and the SFO are now two regulated faces of the same family structure, policed by the same MAS muscle.

---

## 7. The Worked Example — A Family-Office Trust Structure

### 7.1 The Scenario — the Lim Family

The Lim family is the familiar context from the SG wealth cluster: a **Singapore-based UHNW family** — third-generation patriarch *Mr Lim*, his wife, two adult children (one running the family's regional distribution business, one a Singapore-based investment professional), and three grandchildren. The family wealth: a **S$300 million liquid portfolio** (global equities, bonds, private-credit funds, two private-market co-investments), a **family distribution business** worth ~S$150 million, and a **private collection** of real estate. Mr Lim is 68 and wants three things:

1. **Succession without probate drama** — the next generation should inherit through governance, not a will contest.
2. **Asset protection** — a divorce or insolvency of either child should not fracture the family capital.
3. **Tax efficiency** — the family wants a Singapore 13O structure on the liquid portfolio (cross-ref [singapore_private_markets_guide.md](singapore_private_markets_guide.md) §6.4), plus a philanthropic arm.

The family has been a private-banking client of a large Singapore bank for 20 years and is now being advised on the standard Singapore architecture: **trust + PTC + VCC + 13O**, with the bank-owned or independent trustee in the reg-4(2) engagement seat.

### 7.2 The Structure — PTC + VCC + 13O Design

The design (the industry-standard Singapore family-office trust architecture, assembled from the verified components of §1–§6):

**Layer 1 — The trust.** Mr Lim settles a **Singapore discretionary, irrevocable trust** (governing law: Singapore; Trustees Act 1967 applies). The beneficiary class: his wife, his children, remoter issue, and (for the philanthropy arm) registered charities. A **letter of wishes** guides distributions (income to the children's households; capital for grandchildren's education and business ventures). A **protector** (the family's long-time family-office counsel) holds defined veto powers.

**Layer 2 — The PTC.** A Singapore **private trust company**, Lim Family Trustees Pte. Ltd., is the **trustee**. Ownership: held by a family holding company owned by Mr Lim and (post-succession) the children. Board: Mr Lim (chair), the two children, the family counsel, and a professional independent director. The PTC relies on the **reg-4 exemption** and, per **reg 4(2)**, engages a **licensed trust company** — the family's private bank's trustee arm (e.g., a DBS/OCBC-group trustee) — to run the AML/CFT checks and prescribed trust-administration services.

**Layer 3 — The manager company.** The trust (via the PTC as trustee) owns 100% of **Lim Family Office Pte. Ltd.** — the SFO fund manager (13D-aligned; the family's own CMS-exempt or VCFM manager per the private-markets guide §5.7). The next generation's inheritance is thus *interests in the trust*, not shares in the family office.

**Layer 4 — The fund vehicle.** The liquid portfolio moves into **Lim Family VCC (a VCC umbrella, 13O scheme)**: sub-fund A (public markets), sub-fund B (private credit — the bank-adjacent sleeve from [asset_backed_trading_guide.md](asset_backed_trading_guide.md) and the private-markets guide §7.1), sub-fund C (co-investment SPVs). The VCC's qualifying investment income is **tax-exempt under 13O** (scheme conditions per §6.1; SFO class-exemption compliance in place).

**Layer 5 — The banking layer.** The private bank provides custody, treasury and lending across the trust, manager and fund accounts; the bank-owned trustee (or an independent LTC) holds the reg-4(2) engagement; the bank's own CMS/trading infrastructure services the portfolio.

**Layer 6 — The philanthropy arm.** A **charitable trust** (Lim Family Foundation Trust) settled alongside, trustee also the PTC (the exemption covers connected charitable trusts — §3.5), with the Commissioner of Charities registration handled by the family office.

### 7.3 The Structure Diagram (textual)

```
Mr Lim (settlor) ── settles ──► Singapore Discretionary Irrevocable Trust
                                     │
                                     │ trustee
                                     ▼
                        Lim Family Trustees Pte. Ltd. (PTC)
                        │  (reg 4 exemption; reg 4(2) engagement)
                        │
                        ├── engages ──► Licensed Trust Company (AML/CFT administration)
                        │
                        └── owns 100% ──► Lim Family Office Pte. Ltd. (SFO manager, 13D-aligned)
                                             │
                                             └── manages ──► Lim Family VCC (13O scheme)
                                                                 ├─ Sub-fund A: public markets
                                                                 ├─ Sub-fund B: private credit
                                                                 └─ Sub-fund C: co-investment SPVs
                        └── trustee of ──► Lim Family Foundation Trust (charitable)

Beneficiaries: Mrs Lim, children, grandchildren, charities
Protector: family counsel (veto powers)
Letter of wishes: guides distributions, deliberately non-binding
Private bank: custody, treasury, lending (banks_in_singapore_guide.md)
```

### 7.4 The Design Decisions

| Decision | Choice | Why |
|---|---|---|
| Discretionary + irrevocable | Yes on both | Flexibility across generations; genuine parting with assets → asset protection, no sham risk, clean estate treatment (§3.1, §3.2) |
| PTC as trustee | Lim Family Trustees Pte. Ltd. | Family governance and continuity; one trustee for all family trusts; privacy; cost at scale (§4.3) |
| LTC engagement | Bank-owned trustee (or independent LTC) | Mandated by reg 4(2); brings independent AML oversight; keeps the structure defensible (verified) |
| Trust owns the manager | 100% of Lim Family Office Pte. Ltd. | The canonical trust+13O combination — next gen inherits trust interests, not shares (§6.2, verified) |
| VCC + 13O for the portfolio | Umbrella VCC, 13O scheme | Tax-exempt qualifying income; umbrella flexibility; the fund regime of the private-markets guide §5 |
| Charitable trust under the PTC | Foundation Trust | Philanthropy within the same governance; PTC exemption covers it (verified) |
| Protector + letter of wishes | Family counsel as protector | Non-binding guidance preserves trustee discretion; protector blocks hostile changes |
| Governing law | Singapore | The TCA + Trustees Act stack; SFO incentives; no estate duty (reported) |

### 7.5 The Lessons

1. **The trust is the chassis, the 13O is the engine.** Families that build a 13O VCC without a trust get tax efficiency and a probate problem; families that build a trust without the 13O get governance and a tax problem. The standard architecture needs both — the trust owns the manager, the fund earns exempt income (§6.2).
2. **The PTC is a governance product, not a tax product.** Its value is a permanent family board and one trustee for all trusts; its price is the mandated LTC engagement and the full AML discipline. Treating the PTC as a way to "avoid regulation" is how structures get attacked.
3. **Control must be fiduciary, not personal.** The settlor's powers stay out of the day-to-day; the protector's powers are defined; the LTC's oversight is real. A structure the settlor visibly still runs is a sham in the making — the single most common way Singapore trust structures fail (reported — consistent adviser commentary).
4. **The banking relationship is part of the structure.** Custody, lending and the trustee engagement usually sit with the same banking group (bank-owned trustee) or split (independent trustee + bank for custody). Both patterns are legitimate; the choice should be explicit (§5.4).
5. **Compliance is the price of the incentive.** TCA-N03, Trustees Act Part 7 transparency, SFO scheme conditions and sanctions due diligence are not afterthoughts — they are the conditions under which the tax and governance benefits exist at all (§6.4).

### 7.6 The Implementation Plan — Three Phases

| Phase | Steps | Key parties | Duration (indicative) |
|---|---|---|---|
| Phase 1 — Design (months 0–2) | Family governance review; settle the beneficiary class and letter of wishes; choose PTC board; select the LTC engagement partner (bank-owned vs independent — §5.4); structure the VCC umbrella and sub-funds | Family counsel; LTC; tax adviser; private bank | ~8 weeks |
| Phase 2 — Execution (months 2–5) | Incorporate PTC + manager company; settle the trust deed; MAS exempt-person notification (Form 8); engage LTC (reg 4(2)); incorporate the VCC; file the 13O application (SFO class-exemption path with refreshed legal opinion); open banking/custody accounts | PTC directors; LTC; MAS application advisers; corporate services firm | ~12–16 weeks (MAS timelines variable — flagged) |
| Phase 3 — Funding and running (months 5–9) | Transfer assets into the VCC sub-funds; settle the charitable trust; register with Commissioner of Charities; AML files built for all layers (TCA-N03, Trustees Regs); annual reporting calendar set | Private bank (custody); LTC (AML administration); PTC board; auditors | Ongoing |

### 7.7 The Risk Register

| Risk | Mitigation | Notes |
|---|---|---|
| Sham challenge (settlor retains control) | Deed limits settlor powers; protector vetoes defined; LTC engagement evidences independent oversight | The classic failure mode (§7.5, lesson 3) |
| Beneficiary dispute / family conflict | Discretionary class + letter of wishes; family council above the PTC board; independent director on the board | Governance is the product |
| LTC partner failure or conflict | Two-LTC review; termination/replacement rights in the engagement agreement; bank custody separate from trustee | The reg-4(2) seat must not become a single point of failure |
| 13O condition breach (spending, hiring, private-banking account) | Dedicated SFO-compliance calendar; class-exemption legal-opinion refreshes | Cross-ref private-markets guide §5 |
| AML/CFT breach (TCA-N03, Trustees Regs) | Centralised CDD file for settlor/beneficiaries/controllers; S$20,000 notification process; 5-year retention | Verified obligations (§6.4) |
| Sanctions exposure (settlor/beneficiary screening) | Ongoing screening; the 2025 posture makes this non-optional (reported — flagged) | Prince Holdings precedent |
| Cross-border tax leakage (offshore assets, foreign beneficiaries) | IRAS/foreign-counsel review before settlement; 13U route if offshore vehicle must stay | §3.8 |
| Trustee liability events (contracts, litigation) | Trustee indemnity clauses; adequate trust assets; D&O cover for PTC board | Standard practice (reported) |

---

## 8. The Summary — Trust Companies in One Page

**What they are.** Trust companies are the MAS-regulated institutions that act as trustees — holding legal ownership of assets for the benefit of others under the settlor–trustee–beneficiary structure. They are the professional custodians of Singapore's wealth architecture: the firms that make succession, asset protection and philanthropy legally real.

**The regime.** The Trust Companies Act 2005 (in force 1 February 2006) replaced voluntary registration with **compulsory MAS licensing**: carrying on trust business in or from Singapore requires a licence, a statutory exception, or an exemption. The regime's genius is its two tiers — **licensed trust companies** (the public, capitalised, supervised franchise) and **exempt/registered players** (banks serving their own customers, lawyers, and the family **PTCs** that must engage a licensed trustee for AML administration under reg 4(2)). Every structure in this guide runs through that two-tier gate.

**The vehicles.** Discretionary irrevocable trusts are the default family vehicle; unit trusts make trust companies the investor-protection linchpin of the fund industry (Approved CIS Trustee); charitable trusts carry philanthropy; business trusts (BTA 2004) and purpose structures occupy their statutory corners.

**The market.** Bank-owned trustees (DBS Trustee, BOS Trustee/OCBC, HSBC Trustee, the flagged UOB and Standard Chartered arms) serve the private-banking and CIS books; global independents (Vistra–Tricor, TMF, CSC/Intertrust, Asiaciti, Apex, Ocorian, JTC, Portcullis) serve relationship-led, multi-jurisdictional structuring — in a sector defined by consolidation (Vistra+Tricor; CSC+Intertrust).

**The family-office interplay.** The trust and the 13O/13D/13U fund schemes are complements: the trust owns the family-office manager; the VCC+13O earns tax-exempt income; the 2,000+ SFOs are the demand side. The worked example — the Lim family's PTC + VCC + 13O stack — is the standard Singapore design, held together by the AML/CFT discipline both regulators enforce.

**The final word — the quiet custodians.** Banks get the headlines; fund managers get the AUM; but the trust companies are the **quiet custodians** of Singapore's wealth — the licensed fiduciaries and family-owned PTCs that hold the legal title, run the governance, and carry the compliance, generation after generation. In a city-state built on intermediated trust — from its banking secrecy-era origins to its 2,000-family-office present — the trust companies are where the trust in "trust" actually lives.

---

## 9. Glossary

| Term | Definition |
|---|---|
| Trust | A fiduciary relationship in which a settlor transfers legal ownership of assets to a trustee, who holds and manages them for beneficiaries (legal vs equitable ownership split) |
| Settlor | The person who creates a trust by transferring assets into it and settling the trust deed |
| Trustee | The legal owner and administrator of trust assets, owing fiduciary and statutory duties (Trustees Act 1967); in Singapore, professional trustees must be licensed trust companies |
| Beneficiary | The person(s) for whose benefit a trust exists; owner of the equitable interest; in discretionary trusts, a member of a class with no fixed entitlement until the trustee's discretion is exercised |
| Trust Companies Act | The Trust Companies Act 2005 (Cap 336) — the Singapore statute establishing compulsory MAS licensing for trust business (in force 1 Feb 2006) |
| MAS | Monetary Authority of Singapore — central bank and financial regulator; licences and supervises trust companies under the TCA |
| Licence | A trust business licence granted by MAS under the TCA; required to carry on trust business in/from Singapore unless excepted or exempt |
| Licensed trust company (LTC) | A company holding a MAS trust business licence; fit-and-proper, capitalised, supervised; e.g., DBS Trustee, BOS Trustee, HSBC Trustee (Singapore) |
| Discretionary trust | A trust where the trustee decides which beneficiaries receive distributions and how much, within a defined class |
| Revocable | A trust the settlor can revoke/amend during their lifetime; weaker asset protection |
| Irrevocable | A trust the settlor permanently parts with; the default for asset protection and succession |
| Unit trust | A collective investment scheme in which a trustee holds the fund's assets and investors hold units (Approved CIS Trustee category) |
| Purpose trust | A trust for a purpose rather than beneficiaries; not generally valid under SG common law absent statutory backing (business trusts; PTC contexts) |
| Charitable trust | A trust for legally charitable purposes; enforceable without private beneficiaries (Commissioner of Charities) |
| Business trust | A trust carrying on a business with units, regulated under the Business Trusts Act 2004; run by a trustee-manager |
| Private trust company (PTC) | A family-owned company acting as trustee only of connected trusts; exempt from licensing under the Trust Companies (Exemption) Regulations reg 4; must engage an LTC for AML administration (reg 4(2)) |
| PTC | See private trust company |
| Family office | A structure managing one family's wealth (single FO) or several (multi-FO); 2,000+ SFOs in Singapore as of end-2024 (verified — cross-ref) |
| 13O | Income Tax Act Section 13O — onshore fund tax-exemption scheme for SG-incorporated vehicles with SG-based managers; extended to 31 Dec 2029 (verified — cross-ref) |
| VCC | Variable Capital Company — Singapore corporate fund vehicle under the Variable Capital Companies Act (in force 14 Jan 2020); umbrella sub-funds; the 13O carrier |
| Estate planning | Arranging ownership and succession of wealth (wills, trusts, structures) to pass assets efficiently and avoid probate disputes |
| Asset protection | Structuring so family wealth is shielded from creditors, matrimonial and insolvency claims — typically via irrevocable discretionary trusts |
| STEP | Society of Trust and Estate Practitioners — the global professional body for trust and estate professionals, with an active Singapore branch |
| Custodianship | The role of holding and safeguarding assets on behalf of others — the trust company's core function; also the bank custody layer beneath it |
| Trust Bank | The Standard Chartered × FairPrice digital bank — NOT a trust company (name-collision, §0; see trust_bank_guide.md) |

---

## 10. Claims Status, References and Further Reading

### Claims Status Table

| Claim | Status | Source |
|---|---|---|
| Trust Companies Act 2005 (Cap 336); compulsory MAS licensing; in force 1 Feb 2006; replaced voluntary Companies Act registration | Verified | Singapore Statutes Online (sso.agc.gov.sg, TCA 2005 + 2020 Rev Ed); Law Gazette (2007); Government Gazette PDF |
| Prohibition: no person other than an LTC may carry on trust business in/from SG unless excepted/exempt; broad definition of trust business; "system, repetition and continuity" test | Verified | Law Gazette (2007) |
| Statutory exceptions (bare trustee, business-trust trustee, trustee-manager, will preparers, executors) and exempt persons (banks, merchant banks, life insurers, lawyers with ≤S$2m/≤30-client/Form-8 conditions) | Verified | Law Gazette (2007) |
| 21 trust business licences and 29 registered exempt persons as of mid-Dec 2006 | Verified (historical) | Law Gazette (2007) |
| Current number of licensed trust companies | Flagged — not re-confirmed; check MAS FID | eservices.mas.gov.sg FID (Licensed Trust Company category) |
| PTC exemption under Trust Companies (Exemption) Regulations reg 4; reg 4(2) requires engaging an LTC for AML/CFT-related trust administration | Verified | Singapore Statutes Online (TCA2005-RG1) |
| Trustees Act 1967 Part 7 + Trustees Regulations 2017: verification, 5-year records, S$20,000 notification threshold; MAS Notice TCA-N03 | Verified | Trustees Regulations text; Anlian Group summary; MAS notices |
| Business Trusts Act 2004; registered business trusts; trustee-manager | Verified | Singapore Statutes Online (BTA2004) |
| DBS Trustee: wholly owned by DBS Group Holdings; LTC + SFA-approved trustee | Verified | dbs.com.sg; MAS FID; CapitalMarkets.SG |
| BOS Trustee: formerly OCBC Trustee; incorporated 1938; Bank of Singapore/OCBC subsidiary; LTC + approved CIS trustee | Verified | bankofsingapore.com; Companies.sg; MAS FID |
| HSBC Trustee (Singapore) Limited: LTC at MBFC | Verified | MAS FID |
| UOB trust entity; Standard Chartered Trust (Singapore); Cititrust; BNP Paribas Trust Corp SG | Flagged — reported, not re-confirmed | Industry directories; MAS FID (partial) |
| Vistra + Tricor merger completed (BPEA EQT) | Verified | vistra.com; EQT release; law.asia |
| Intertrust acquired by CSC (completed 2023) | Reported — completion date flagged | Press; industry commentary |
| Singapore Trustees Association exists and represents trust companies | Verified | sta.org.sg |
| Trust + 13O/13U complementarity; trust owns SFO manager equity; SFO class-exemption framework 2024/2025; sanctions posture 2025 | Verified (complementarity) / reported (framework details flagged) | Anlian Group; cross-ref private-markets guide |
| 13O/13U schemes extended to 31 Dec 2029; 2,000+ SFOs end-2024; S$6.7T industry (2025); S$5.4T (2023) | Verified / reported (cross-ref) | singapore_private_markets_guide.md claims table; MAS survey |
| Singapore abolished estate duty (2008); trust taxation at beneficiary level | Reported — flagged | General tax commentary (IRAS) |
| Purpose trusts not generally valid under SG common law; no general statutory regime | Flagged — widely understood position; statutory carve-outs verified | Comparative trust commentary; exemption-regulations context |

### References and Further Reading

- **Statutes (Singapore Statutes Online, sso.agc.gov.sg)** — Trust Companies Act 2005 (Cap 336); Trust Companies (Exemption) Regulations (TCA2005-RG1); Trust Companies Regulations; Business Trusts Act 2004; Trustees Act 1967; Trustees (Transparency and Effective Control) Regulations 2017; Income Tax Act (Sections 13D/13O/13OA/13U)
- **MAS** — mas.gov.sg: Financial Institutions Directory (Licensed Trust Company / Exempt Trust Company / Approved CIS Trustee categories); Notice TCA-N03; Guidelines on Standards of Conduct and Criteria for Grant of Trust Business Licence; SFO framework statements (2023 consultation response, 2024–2025 class exemption, Aug 2026 FO rule easing)
- **Industry bodies** — Singapore Trustees Association (sta.org.sg); STEP Singapore (step.org)
- **Trust companies' sites** — DBS Trustee (dbs.com.sg), Bank of Singapore / BOS Trustee (bankofsingapore.com), HSBC Trustee, Standard Chartered, Vistra (vistra.com — Tricor merger), TMF Group, CSC (Intertrust), Asiaciti, Apex Group, Ocorian, JTC, Portcullis
- **Commentary** — Law Gazette "Update of Trust Companies Act 2005" (2007); Anlian Group (Singapore family trust and 13O/13U complementarity); Kensington Trust (Singapore PTCs); CapitalMarkets.SG (MAS licensed trust company registry); Raffles Corporate Services (PTC setup walkthrough)
- **Press** — The Business Times (SFO numbers, FO rule easing); WealthBriefingAsia; Hubbis; Reuters (SFO count, 14 Jan 2025)
- **Sibling guides** — [singapore_private_markets_guide.md](singapore_private_markets_guide.md) (VCC/13O/13U/FO regime — the trust's fund-side sibling); [bond_financial_group_company_guide.md](bond_financial_group_company_guide.md) and [bond_capital_group_company_guide.md](bond_capital_group_company_guide.md) (SG holding/wealth structure design); [banks_in_singapore_guide.md](banks_in_singapore_guide.md) (bank parents); [trust_bank_guide.md](trust_bank_guide.md) (the name-collision); [asset_backed_trading_guide.md](asset_backed_trading_guide.md) (the private-credit sleeve); [financial_infrastructure_guide.md](financial_infrastructure_guide.md) (market infrastructure); [risk_management_models_guide.md](risk_management_models_guide.md); [supply_chain_finance_guide.md](supply_chain_finance_guide.md); [universal_banking_model_guide.md](universal_banking_model_guide.md) (the bank adjacency); [../management/mba_body_of_knowledge_guide.md](../management/mba_body_of_knowledge_guide.md) and [cfa_program_guide.md](cfa_program_guide.md) (the vocabulary)

*Nothing in this guide constitutes investment, legal, or tax advice. All figures are as-of the dates stated and subject to revision. Claims marked "flagged" or "reported" were not independently re-confirmed in this research pass; the MAS Financial Institutions Directory is the authoritative source for current licence status.*
