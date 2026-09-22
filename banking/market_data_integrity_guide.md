# Market Data Integrity: What You Knew, and When You Knew It — Cross-Vendor Reconciliation, Corporate Actions and Point-in-Time Correctness — A Comprehensive Guide

**The market- and reference-data *integrity* discipline of the repo's capital-markets cluster: the identifier stack verified at source (ISIN/ISO 6166 and the ANNA Service Bureau, CUSIP and CUSIP Global Services under FactSet since 1 March 2022, SEDOL and the SEDOL Masterfile at LSEG, the FIGI open standard at OMG, the LEI under ISO 17442-1:2020 and GLEIF, the OTC-derivative UPI and UTI under the DSB and CPMI-IOSCO, the ISO 10383 Market Identifier Code under SWIFT, the ISO 10962 CFI at ANNA), the effective-dated crosswalk that makes two vendors' answers joinable, cross-vendor reconciliation as a *control* with per-field tolerance classes and a break workflow with ownership and aging, the corporate-action *lifecycle* as a data-model driver rather than a downstream adjustment, the two time axes (valid/effective time and transaction/system time) and what the FRTB risk-factor eligibility test now demands of the price record, the minimum viable schema for a small team, the architecture and operating model, the regulatory frame, a Cymbal Bank worked example that exercises all three problems in one narrative, the anti-patterns, the claims audit, and the honest list of what could not be verified**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Banking / Capital Markets — the data-integrity deep-dive of the repository's market-data material. The repo names the vendors everywhere and addresses their *integration* problems nowhere: an unbounded, case-insensitive search for `isin` across this repository matches 356 markdown files — because it matches the `-ising` substring in *advertising*, *raising*, *organising* and *supervising* — while the word-bounded search matches 17; `symbolog` matches 0 files, `FIGI` 1, word-bounded `RIC` 1, `SEDOL` 4, the phrase "corporate action" 4, "security master" 7 and word-bounded `LEI` 17 — against 99 files mentioning "market data" and 60 mentioning Bloomberg. This guide fills exactly that gap: not what market data *is* (the sibling guides do that), but what breaks when two sources disagree about the same instrument, when the instrument's own history is rewritten by a corporate action, and when the price you are reading today is not the price anyone saw on the date it claims to describe. The neighbouring content is cross-referenced by section, never re-derived: the market-data taxonomy and feed architecture of [Capital Markets Architecture](capital_markets_architecture_guide.md) §5, the producer-to-consumer value chain and vendor comparison of [Market Data Consumption](market_data_consumption_guide.md), the corporate-action *operations* and balance-reconciliation universe of [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §4/§5/§8, BCBS 239 itself in [Risk Data Aggregation](risk_data_aggregation_guide.md), the wire protocol in [FIX Protocol](fix_protocol_guide.md), and the storage and lineage engineering under `../technology/data/`. Facts verified this pass are marked ✅ with the source named in §14; anything that could not be re-verified at source is flagged ⚠ honestly, listed again in §15. No vendor product name, standard number, issuing body, ownership fact, tolerance or regulatory requirement in this guide was invented.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** the FactSet press release "FactSet Completes Acquisition of CUSIP Global Services" (investor.factset.com, 1 March 2022; the same text filed as exhibit 99.1, sec.gov/Archives/edgar/data/1013237/000095015722000210/ex99-1.htm — completion 1 March 2022, approximately US$1.925bn, announced 27 December 2021, CUSIP system stewarded "in close partnership with the American Bankers Association"), the ANNA Service Bureau page and ANNA's own service site (anna-web.org/about-the-anna-service-bureau — ASB built and operated by ANNA partners CUSIP Global Services and SIX Financial Information, data from 120+ National Numbering Agencies since 2001, 200+ jurisdictions, 1M+ new ISINs monthly; annaservice.com/anna/whats.jsp — ANNA as Registration Authority for ISIN and CFI under appointment by ISO), the Derivatives Service Bureau (anna-dsb.com — DSB designated sole service provider for the UPI system, operator of the UPI reference data library; nomination by the Financial Stability Board), ISO 4914:2021 (iso.org/standard/80506.html — published 2021-11-16, ISO/TC 68/SC 8), the CPMI-IOSCO *Harmonisation of the Unique Transaction Identifier — Technical Guidance* (bis.org, published February 2017), the ISO 10383 Market Identifier Codes pages (iso20022.org/market-identifier-codes — S.W.I.F.T. SC as Registration Authority, monthly publication on the second Monday and modifications effective the fourth Monday, list published 14 September 2026; and the MIC Registration Procedures PDF at iso20022.org), the SWIFT data-standards page (swift.com/standards/data-standards — SWIFT as Registration Authority for ISO 9362 BIC, ISO 13616 IBAN and ISO 10383 MIC), GLEIF's "This is GLEIF" and "The Legal Entity Identifier (LEI)" pages (gleif.org — GLEIF established 2014 by the FSB, overseen by the Regulatory Oversight Committee, LEI issuers as Local Operating Units; 20-character code) and ISO 17442-1:2020 (iso.org/standard/78829.html — Part 1: Assignment, published 2020-08, last reviewed and confirmed 2026, superseding ISO 17442:2019), the OMG FIGI material (omg.org/intro/OpenFIGI_and_OMG.pdf — FIGI as an OMG standard issued and distributed by Bloomberg L.P. as Registration Authority under OMG auspices, fee-free open data; omg.org/spec/FIGI — the OMG specification page) and Bloomberg's own press releases (bloomberg.com/company/press — Bloomberg and Kaiko as Certified Providers; the FIGI award release describing the standard as issued by Bloomberg L.P. and Kaiko as Certified Providers, distributed by Bloomberg L.P. as Registration Authority under the auspices of OMG and X9), the Basel Framework's *Internal models approach: model requirements* chapter MAR31 and *general provisions* MAR30 (bis.org/committees/bcbs/basel-framework/standard/mar/31/... and .../mar/30/... — read directly, MAR31.12–MAR31.16 on the risk factor eligibility test), BCBS publication d457 *Minimum capital requirements for market risk* (bis.org/bcbs/publ/d457.htm — published 14 January 2019, corrected 25 February 2019, effective 1 January 2022, integrated into the consolidated Basel Framework), LSEG's SEDOL Masterfile pages and brochure (lseg.com/en/data-analytics/market-data/data-analytics-pricing/data-symbology/sedol; the LSEG SEDOL Masterfile brochure PDF; londonstockexchange.com/contact/symbology), the LSEG Data Catalogue reference-data/identifiers page (lseg.com), **LSEG's own Symbology page (lseg.com/en/data-analytics/market-data/data-analytics-pricing/data-symbology — read this pass: "The RIC is a market-level identifier for instruments and pricing sources issued by LSEG"; PermID "issued by LSEG"; LEI "based on ISO 17442" with LSEG acting as an LOU under GLEIF requirements and ROC-agreed principles; SEDOL codes "issued at the country level for most asset classes ... in operation for over 40 years", over 22 million active securities and "allocated at a more granular level than ISIN codes"; and London Stock Exchange Plc as "the sole issue[r]" of ISIN, FISN and CFI codes for Great Britain, Jersey, Guernsey and the Isle of Man)**, ICE's Fixed Income & Data Services site (ice.com/data-services — ICE Consolidated Feed, ICE Consolidated History, ICE Data API, ICE Global Network, and ICE Data Pricing & Reference Data, LLC as the US evaluations entity), and the ISO/IEC 9075-1:2011 catalogue entry (iso.org/standard/53681.html — the SQL framework part, withdrawn, revised by 9075-1:2016 and 9075-1:2023). Every ✅ in §14 carries its source and the date checked.
> **Last Updated:** September 2026
> **Companion guides (sibling, same folder — the capital-markets cluster):** [Capital Markets Architecture](capital_markets_architecture_guide.md) (§5 — the market-data summary: what market data is, the real-time feed architecture, the reference-data/golden-copy idea, the data-type table, the identifier stack; cite it for the taxonomy, do not re-derive it) · [Market Data Consumption](market_data_consumption_guide.md) (the value chain venue → feed → vendor → platform → decision, the consolidated-tape and consolidated-tape-provider material, the vendor comparison, the consumption patterns, the data-management layer — cite it for the producer/consumer map and the feed taxonomy) · [Investment Portfolio Operations](investment_portfolio_operations_guide.md) (§4 corporate-action *operations* — types, mandatory vs voluntary, the announcement/record/ex-date/payment lifecycle, MT564/MT566 and ISO 20022 seev.xxx flows, elections/claims/tax; §5 accounting/balance reconciliation — positions, cash, NAV; §8 reference and market data — the security master §8.1, identifier basics §8.2, feeds §8.3, governance §8.4) · [Risk Data Aggregation](risk_data_aggregation_guide.md) (BCBS 239 itself, risk aggregation, lineage — this guide cross-references it and notes one numbering discrepancy, §11.2) · [FIX Protocol](fix_protocol_guide.md) (the wire protocol and its market-data messages) · [Online Investment Trading Platforms](online_investment_trading_platforms_guide.md) (broker platforms — the retail-side consumer, lightly) · [Transaction Foundation Model](transaction_foundation_model_guide.md) (a model built on transaction data where the point-in-time question recurs) · [Enterprise Risk Management](enterprise_risk_management_guide.md) (model and market-risk governance)
> **Companion guides (technology/, prefix `../technology/`):** [Deterministic Engineering](../technology/deterministic_engineering_guide.md) (reproducibility — the backtest-determinism angle) · [Senior Data Platform Architect](../technology/senior_data_platform_architect_guide.md) (the data-platform mandate; its BCBS 239 section-grouping correction is the authority this guide cites for §11.2) · the `data/` shelf: [Data Governance Framework](../technology/data/data_governance_framework.md) · [Data Lineage Tools](../technology/data/data_lineage_tools.md) · [Data Fabric](../technology/data/data_fabric_guide.md) · [Enterprise Data Platforms](../technology/data/enterprise_data_platforms_guide.md) · [Data Pipeline Versioning](../technology/data/data_pipeline_versioning.md) · [Backfill Data Engineering](../technology/data/backfill_data_engineering.md) · [Cloud Object Storage Lakehouse](../technology/data/cloud_object_storage_lakehouse_guide.md) · [On-Prem Lakehouse](../technology/data/on_prem_lakehouse_guide.md) · [Delta Lake vs Iceberg](../technology/data/delta_lake_vs_iceberg.md) · [Data Vault 2 Modelling](../technology/data/data_vault_2_modeling.md) · [Types of Dimensions in Data Warehousing](../technology/data/types_of_dimensions_data_warehousing.md) (SCD Types 0–7 — the effective-dating pattern, §3)

---

**How to use this guide:** Section 1 is the overview, the one-line thesis and the decoder — read it first, because the guide uses a specific vocabulary and deliberately *does not* re-explain the market-data landscape that the sibling guides already cover. Section 2 is the spine: the four structural causes that generate the three problems this guide exists to solve. Sections 3 to 6 take the three problems one at a time and in depth — the identifier problem (§3), cross-vendor reconciliation (§4), corporate actions as a lifecycle and a data-model driver (§5), and point-in-time correctness (§6). Section 7 is the synthesis: they are one problem. Sections 8 and 9 are the engineering core — the minimum viable schema (§8) and the architecture, flow and build-vs-buy question (§9). Section 10 is the operating model, which is where integrity actually lives or dies. Section 11 is the regulatory frame — FRTB's market-data requirement verified at the BCBS text, BCBS 239's grouping used as the standard itself uses it, and the prudential and conduct regimes that depend on correct reference and corporate-action data; it is cross-reference and pointer, not compliance advice. Section 12 is the Cymbal Bank worked example (fictional, illustrative, figures labelled as such): two vendors, one static attribute, a corporate action one vendor applied and the other had not, and a backtest that had been quietly consuming restated prices. Section 13 is the anti-patterns in symptom/cause/guardrail form. Section 14 is the claims audit; §15 is the explicit list of what could not be verified; §16 closes with the glossary, the cross-references and the final formulation. **Integrity convention:** ✅ = verified this pass against the source named in §14; ⚠ = flagged — either secondary-sourced or not re-verifiable this pass; ❌ = rejected — a claim encountered in circulation that the evidence contradicts. Cross-references follow the repository convention: sibling guides in `banking/` are plain filenames; guides in `technology/` are prefixed `../technology/`.

---

## Table of Contents

1. [The Overview, the Identity and the Decoder](#1-the-overview-the-identity-and-the-decoder)
   - 1.1 [The Thesis in One Line](#11-the-thesis-in-one-line)
   - 1.2 [What This Guide Covers, and Who It Is For](#12-what-this-guide-covers-and-who-it-is-for)
   - 1.3 [The Decoder — The Vocabulary This Guide Uses](#13-the-decoder--the-vocabulary-this-guide-uses)
   - 1.4 [The Boundary — What This Guide Does Not Do](#14-the-boundary--what-this-guide-does-not-do)
2. [Why This Is Hard — The Four Structural Causes](#2-why-this-is-hard--the-four-structural-causes)
   - 2.1 [Cause One — The Instrument's Identity Is Contested](#21-cause-one--the-instruments-identity-is-contested)
   - 2.2 [Cause Two — The Data Is Not Fixed: Vendors Restate History](#22-cause-two--the-data-is-not-fixed-vendors-restate-history)
   - 2.3 [Cause Three — The Instrument Is Not Fixed: Corporate Actions Change What the Identifier Refers To](#23-cause-three--the-instrument-is-not-fixed-corporate-actions-change-what-the-identifier-refers-to)
   - 2.4 [Cause Four — The Market Is Global: Calendars, Conventions, Regimes, Settlement Practice](#24-cause-four--the-market-is-global-calendars-conventions-regimes-settlement-practice)
   - 2.5 [How the Four Causes Generate Exactly Three Problems](#25-how-the-four-causes-generate-exactly-three-problems)
3. [The Identifier Problem — The Symbology Stack](#3-the-identifier-problem--the-symbology-stack)
   - 3.1 [The Stack at a Glance](#31-the-stack-at-a-glance)
   - 3.2 [Identifier by Identifier: Purpose, Issuer, Scope, Limits](#32-identifier-by-identifier-purpose-issuer-scope-limits)
   - 3.3 [The Effective-Dated Crosswalk — A Mapping Table Without Dates Is a Time Bomb](#33-the-effective-dated-crosswalk--a-mapping-table-without-dates-is-a-time-bomb)
   - 3.4 [The Identifier Lifecycle — What Causes an Identifier to Change](#34-the-identifier-lifecycle--what-causes-an-identifier-to-change)
   - 3.5 [The Rule: The Identifier You Standardise On Determines What You Can Ever Join On](#35-the-rule-the-identifier-you-standardise-on-determines-what-you-can-ever-join-on)
4. [Cross-Vendor Reconciliation](#4-cross-vendor-reconciliation)
   - 4.1 [Why Two Vendors Disagree About the Same Instrument](#41-why-two-vendors-disagree-about-the-same-instrument)
   - 4.2 [Reconciliation as a Control, Not a Comparison](#42-reconciliation-as-a-control-not-a-comparison)
   - 4.3 [The Tolerance Categories](#43-the-tolerance-categories)
   - 4.4 [The Break Workflow](#44-the-break-workflow)
   - 4.5 [The Golden-Source Pattern and Its Consistency Problem](#45-the-golden-source-pattern-and-its-consistency-problem)
   - 4.6 [The Storage Requirement That Makes Reconciliation Possible](#46-the-storage-requirement-that-makes-reconciliation-possible)
5. [Corporate Actions — The Lifecycle, Not a List](#5-corporate-actions--the-lifecycle-not-a-list)
   - 5.1 [The Lifecycle and the Control Point at Each Stage](#51-the-lifecycle-and-the-control-point-at-each-stage)
   - 5.2 [Mandatory vs Voluntary and the Control Consequences](#52-mandatory-vs-voluntary-and-the-control-consequences)
   - 5.3 [The Principal Event Classes and What Each Does to the Instrument's History](#53-the-principal-event-classes-and-what-each-does-to-the-instruments-history)
   - 5.4 [The Message Standards in Use, and the Migration Under Way](#54-the-message-standards-in-use-and-the-migration-under-way)
   - 5.5 [Reconciling Corporate-Action Data — Across Vendors and Against Custodians](#55-reconciling-corporate-action-data--across-vendors-and-against-custodians)
   - 5.6 [The Adjustment Question Is a Data-Model Problem](#56-the-adjustment-question-is-a-data-model-problem)
   - 5.7 [What a Control Cannot Settle](#57-what-a-control-cannot-settle)
6. [Point-in-Time Correctness](#6-point-in-time-correctness)
   - 6.1 [The Two Axes and Their Standard Vocabulary](#61-the-two-axes-and-their-standard-vocabulary)
   - 6.2 [Why the Latest Value Is Not the Truth for a Past Date](#62-why-the-latest-value-is-not-the-truth-for-a-past-date)
   - 6.3 [Silent Restatement](#63-silent-restatement)
   - 6.4 [Survivorship Bias](#64-survivorship-bias)
   - 6.5 [Look-Ahead Bias in Backtests and Model Development](#65-look-ahead-bias-in-backtests-and-model-development)
   - 6.6 [As-Of Queries and the Vintage](#66-as-of-queries-and-the-vintage)
   - 6.7 [The Hard Limit, Stated Plainly](#67-the-hard-limit-stated-plainly)
7. [The Three Problems Are One Problem](#7-the-three-problems-are-one-problem)
8. [The Data Model](#8-the-data-model)
   - 8.1 [The Effective-Dated Identifier Crosswalk](#81-the-effective-dated-identifier-crosswalk)
   - 8.2 [The Bitemporal Record — Valid Time and Knowledge Time](#82-the-bitemporal-record--valid-time-and-knowledge-time)
   - 8.3 [The Immutable Vendor-Raw Store](#83-the-immutable-vendor-raw-store)
   - 8.4 [The Corporate-Action Ledger That Drives Adjustment](#84-the-corporate-action-ledger-that-drives-adjustment)
   - 8.5 [The Minimum Viable Schema for a Small Team](#85-the-minimum-viable-schema-for-a-small-team)
9. [The Architecture and the Flow](#9-the-architecture-and-the-flow)
   - 9.1 [The Five Layers](#91-the-five-layers)
   - 9.2 [Where Each Control Lives](#92-where-each-control-lives)
   - 9.3 [The Data Contract at Each Interface](#93-the-data-contract-at-each-interface)
   - 9.4 [The Lineage Requirement](#94-the-lineage-requirement)
   - 9.5 [Build vs Buy for a Security Master](#95-build-vs-buy-for-a-security-master)
10. [The Operating Model](#10-the-operating-model)
    - 10.1 [The Daily and Periodic Cycles](#101-the-daily-and-periodic-cycles)
    - 10.2 [The Break Queue and Its Ownership](#102-the-break-queue-and-its-ownership)
    - 10.3 [Escalation](#103-escalation)
    - 10.4 [The Vendor-Query Process](#104-the-vendor-query-process)
    - 10.5 [Change Control Over the Crosswalk and the Adjustment Conventions](#105-change-control-over-the-crosswalk-and-the-adjustment-conventions)
    - 10.6 [The Audit Trail a Regulator Would Ask For](#106-the-audit-trail-a-regulator-would-ask-for)
    - 10.7 [The Honest Point: No Tolerance Policy and No Owner Means Nobody Acts](#107-the-honest-point-no-tolerance-policy-and-no-owner-means-nobody-acts)
11. [The Regulatory Frame](#11-the-regulatory-frame)
    - 11.1 [FRTB — What the Market-Risk Framework Demands of the Price Record](#111-frtb--what-the-market-risk-framework-demands-of-the-price-record)
    - 11.2 [BCBS 239 — Accuracy, Completeness, Lineage, Using the Standard's Own Grouping](#112-bcbs-239--accuracy-completeness-lineage-using-the-standards-own-grouping)
    - 11.3 [The Prudential and Reporting Regimes That Consume Market Data](#113-the-prudential-and-reporting-regimes-that-consume-market-data)
    - 11.4 [The Conduct and Reporting Regimes That Depend on Reference and Corporate-Action Data](#114-the-conduct-and-reporting-regimes-that-depend-on-reference-and-corporate-action-data)
    - 11.5 [Where to Read the Detail in This Repository](#115-where-to-read-the-detail-in-this-repository)
12. [The Cymbal Bank Worked Example](#12-the-cymbal-bank-worked-example)
    - 12.1 [The Scenario](#121-the-scenario)
    - 12.2 [The Break: One Static Attribute, Two Vendors](#122-the-break-one-static-attribute-two-vendors)
    - 12.3 [Investigating: The Identifier Crosswalk and Its Effective Dates](#123-investigating-the-identifier-crosswalk-and-its-effective-dates)
    - 12.4 [The Tolerance Question and the Break Workflow](#124-the-tolerance-question-and-the-break-workflow)
    - 12.5 [The Adjustment Decision and Its Effect on the Historical Series](#125-the-adjustment-decision-and-its-effect-on-the-historical-series)
    - 12.6 [The Discovery: A Backtest Consuming Restated Prices](#126-the-discovery-a-backtest-consuming-restated-prices)
    - 12.7 [The Point-in-Time Fix and What It Costs to Adopt](#127-the-point-in-time-fix-and-what-it-costs-to-adopt)
    - 12.8 [The Recommendation, Including What Cymbal Decides to Stop Doing](#128-the-recommendation-including-what-cymbal-decides-to-stop-doing)
13. [The Anti-Patterns](#13-the-anti-patterns)
14. [The Claims Audit](#14-the-claims-audit)
    - 14.1 [The Convention](#141-the-convention)
    - 14.2 [The Claims Audit — Verified, Flagged, Rejected](#142-the-claims-audit--verified-flagged-rejected)
15. [What Could Not Be Verified](#15-what-could-not-be-verified)
16. [The Glossary, the Cross-References and the Closing Summary](#16-the-glossary-the-cross-references-and-the-closing-summary)
    - 16.1 [The Glossary](#161-the-glossary)
    - 16.2 [Cross-References](#162-cross-references)
    - 16.3 [The Closing Summary](#163-the-closing-summary)

---

## 1. The Overview, the Identity and the Decoder

### 1.1 The Thesis in One Line

**The three problems — identifier ambiguity, cross-vendor disagreement, and point-in-time erosion — are one problem: what did we know, and when did we know it.**

Everything that follows is an elaboration of that sentence. An identifier is the claim *which instrument*; a vendor record is the claim *who said so*; a timestamp is the claim *as of when*. Integrity is the discipline of being able to answer all three questions about any number you publish, for any date you published it, months or years later — and of knowing, honestly, where you cannot.

### 1.2 What This Guide Covers, and Who It Is For

**The audience.** A bank's data-management and markets-technology practitioners: the data office that owns the security master, the market-data operations team that runs the vendor feeds, the middle-office and risk-technology teams that consume prices and static data, and the quant or model-development function that consumes history. It is written as a working how-to. There are no academic surveys here; every section ends in a decision somebody has to make.

**The three problems, stated once.** (1) *Identity*: the same instrument is called three things by three sources, and the identifier you standardise on determines which of them you can ever join. (2) *Disagreement*: two vendors, asked the same question about the same instrument, answer differently — and whether that is a defect or a legitimate difference of methodology is itself a decision the firm has to make and record. (3) *Time*: the answer you get today for last March may not be the answer anyone had last March, because vendors restate history, corporate actions rewrite series, and instruments that die disappear from current snapshots.

**What the guide does that nobody else in the repository does.** The repo is rich on what market data *is* and how it *flows*; it is empty on what happens when the answers conflict. Word-bounded, the repository mentions `ISIN` in 17 files, `LEI` in 17, `SEDOL` in 4, the phrase "corporate action" in 4, "security master" in 7, word-bounded `RIC` in 1 and `FIGI` in 1 — and `symbolog` in none — while 99 files mention "market data" and 60 mention Bloomberg. The vendors are named everywhere and their *integration* problems are addressed nowhere. **The substring trap is worth understanding as a research lesson in its own right:** an unbounded, case-insensitive search for `isin` across this repository matches 356 markdown files, and almost none of them are about securities identification — they are matched by *advertising*, *raising*, *organising* and *supervising*. Word-bounded, the same search matches 17. This is the same class of trap the repository has recorded before (`rust`/`trust`, `CAMS`/`scams`, `iLO`/`pilot`, `exam`/`example`): a naive match count is evidence about your search pattern, not about the corpus. The same discipline applies to the matching this guide is *about* — a crosswalk that matches on a substring rather than on a validated identifier is the identifier equivalent of the 356-file answer.

### 1.3 The Decoder — The Vocabulary This Guide Uses

These terms are used precisely throughout. Where a term has a standard anchor, the anchor is named.

| Term | What it means here | Standard anchor |
|---|---|---|
| **Identifier stack (symbology)** | The set of codes a firm stores side by side for one instrument: a global identifier (ISIN), national identifiers (CUSIP, SEDOL), vendor-proprietary codes (RIC and vendor-internal keys), a venue code (MIC) and a classification code (CFI) | ISO 6166 (ISIN), ISO 10383 (MIC), ISO 10962 (CFI) — §3 |
| **Golden copy** | The single designated authoritative record for a data element or domain, from which other systems are fed rather than each sourcing independently | discussed in [Capital Markets Architecture](capital_markets_architecture_guide.md) §5.3 and [Market Data Consumption](market_data_consumption_guide.md) §9.1 — this guide adds the *pattern* (§4.5), not the definition |
| **Cross-vendor reconciliation** | Comparing two independent providers' answers for the same instrument and field, to detect where your record of the instrument is wrong or contested | this guide's §4; distinct from the accounting reconciliation of [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §5 |
| **A break** | A specific, identified, item-level disagreement between two sources or between a source and the golden copy, with an owner and an age | §4.4 |
| **A tolerance** | The pre-agreed, documented threshold and rule set that determines whether a difference is a break at all — per field type, per asset class, agreed with the business, not with the engineer | §4.3 |
| **Corporate-action lifecycle** | The staged chain announcement → entitlement → payment/effective, each stage with its own control point and its own failure mode | the operational processing is [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §4; the *data-model* consequences are §5 of this guide |
| **Mandatory vs voluntary** | Mandatory actions apply to all holders with no election (a cash dividend, a split); voluntary actions require an election with a deadline (a tender offer, a rights subscription); mandatory-with-options sits between | same, [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §4.2 |
| **The adjustment** | The act of restating a historical price or quantity series so that it is continuous across a corporate action — the choice of *whether* and *how* to adjust is a data-model decision that changes every downstream number | §5.6 |
| **The two time axes** | *Valid time* (also effective time, business time, application time): the period in the real world that a fact describes. *Transaction time* (also system time, knowledge time): when the database recorded that fact | SQL temporal vocabulary — §6.1, flagged for the exact standard term where not verified at source |
| **As-of** | A query or a publication rendered as of a specified point on *both* axes — "what did the system know as of 30 June about the position on 15 April" | §6.1, §6.6 |
| **The vintage** | The version of a dataset as it stood on a given knowledge date; the set of values a consumer would have seen had they asked then | §6.6 |
| **The restatement** | A provider's (or a firm's) replacement of a previously published historical value with a different one — sometimes flagged, often not | §6.3 |

Two more, used as controls rather than as vocabulary. **The break queue** is the operating artefact where reconciliation becomes work: a queue with owners, ages and escalation thresholds (§10.2). **The override** is the deliberate, recorded decision to accept a non-default source for a field; an override without an audit trail is an anti-pattern, not a control (§13).

### 1.4 The Boundary — What This Guide Does Not Do

**The boundary, declared by name.** This guide does not re-derive what the repository already covers, and it deliberately stops at the edge of four neighbouring subjects.

- **The market-data taxonomy and the feed architecture** — what market data is, the real-time feed landscape, the golden-copy idea and the data-type taxonomy — live in [Capital Markets Architecture](capital_markets_architecture_guide.md) §5 (its §5.1 what market data is, §5.2 the real-time feeds, §5.3 reference data and golden copy, §5.4 the data-type table, §5.5 data governance including the identifier stack). That section is a summary; this guide goes where it does not — the *mechanism* of reconciliation (§4), the corporate-action *lifecycle* (§5) and the *temporal model* (§6). Cite §5 for the taxonomy; do not expect it here.
- **The producer-to-consumer map** — venues, consolidated tapes and their providers, the vendor comparison and the platform-class consumption patterns — is [Market Data Consumption](market_data_consumption_guide.md): its §2 (the consolidated tape, the SIPs, the plan transition and the EU consolidated tape providers), §3 (the vendors, with a comparison table), §4.6 (corporate-action and reference-data feeds), §8.4 (historical data and time-series stores) and §9 (the data-management layer: golden source §9.1, entitlements §9.2, governance §9.3). This guide assumes that chain and does not reproduce it.
- **Corporate-action *operations*** — the event types, mandatory vs voluntary elections, the announcement/record/ex-date/payment lifecycle, the MT564/MT566 and ISO 20022 seev.xxx flows, and the elections/claims/tax workload — is [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §4. That guide's **§5 is accounting and balance reconciliation** — positions, cash, NAV against custodian and administrator. **This guide's reconciliation is a different animal: cross-vendor market and reference data — two providers, the same instrument, the same field, and no custodian anywhere in the loop.** Where the two meet (corporate-action entitlements against the vendor's announcement record) the boundary is stated explicitly at §5.5. The security master and the identifier basics are that guide's §8.1 and §8.2; this guide takes the identifiers further — issuing body, current ownership, scope and *limits* — at §3.
- **BCBS 239 itself, risk aggregation and lineage** — the standard, its fourteen principles and the implementation record — is [Risk Data Aggregation](risk_data_aggregation_guide.md), and the deep data-governance treatments sit in [Data Governance Framework](../technology/data/data_governance_framework.md) and [Senior Data Platform Architect](../technology/senior_data_platform_architect_guide.md). This guide cross-references them for the accuracy, completeness and lineage expectations (§11.2) and **records one numbering discrepancy** in the risk-data guide without editing it.
- **The wire protocol, the storage platforms and the reproducibility discipline** — FIX's market-data messages are [FIX Protocol](fix_protocol_guide.md); the storage and pipeline engineering is the `../technology/data/` shelf; backtest determinism is [Deterministic Engineering](../technology/deterministic_engineering_guide.md). All are cross-referenced by filename and section, never re-derived.

**One boundary worth stating at the outset, because it is where practitioners most often look for the wrong answer:** integrity problems are almost never solved by buying a better feed. A second feed buys you a *second opinion*, which is useful only if you record both opinions, date them, decide which is authoritative for which field, and keep the decision under change control. That is a data-model and operating-model problem, and it is what the rest of this guide is about.

---

## 2. Why This Is Hard — The Four Structural Causes

### 2.1 Cause One — The Instrument's Identity Is Contested

**There is no single identifier that means "the instrument" everywhere, and the ones that exist were designed for different purposes by different bodies with different scopes.** A global identifier (ISIN) is national in its allocation; national identifiers (CUSIP, SEDOL) are national in their scope; a vendor's code (a RIC, a vendor-internal key) is proprietary and encodes the vendor's own view of the market; an open identifier (FIGI) is defined at multiple levels of granularity by design; a legal-entity identifier (LEI) identifies the issuer and not the security; and the OTC-derivative identifiers identify a *product* and a *transaction* rather than a listed line. Each of these is correct, and none of them is a superset of the others.

**The consequence is structural, not accidental.** The moment a firm stores more than one identifier — which every firm does — it has taken on the obligation to maintain the mapping, and the mapping is a claim about time ("this CUSIP became this ISIN on this date"). A mapping table without dates is not a mapping; it is a snapshot of somebody's belief at an unrecorded moment (§3.3). This is the first cause, and it is the one most often mistaken for a data-entry problem. It is not: it is a design problem, because the identifier you choose as the join key determines, permanently, which questions you can ask (§3.5).

### 2.2 Cause Two — The Data Is Not Fixed: Vendors Restate History

**A vendor's record of the past is a live opinion, not a fixed fact.** Providers correct prices, revise static attributes, amend corporate-action terms, and backfill coverage. Sometimes the correction is announced; often it simply appears in the next load, and the previous value is gone from the surface a consumer sees. The reasons are ordinary: a late trade is reported, an exchange corrects a closing print, a terms sheet is amended, a mapping to an underlying instrument was wrong and is fixed, coverage is extended backwards to an earlier date.

**This is precisely the property the market-risk framework has now regulated.** The Basel Framework's model-eligibility chapter, MAR31, requires a bank to test whether a risk factor is *modellable* by counting **real prices** — and it explicitly contemplates that those prices arrive from a third-party vendor, at a lag, and defines what the vendor must be able to demonstrate: the number of corresponding real prices observed **and the dates at which they have been observed**; a minimum necessary set of identifier information so the bank can map a real price to a risk factor; and an audit of the validity of the vendor's pricing information whose results must be available on request to the supervisor (MAR31.14, verified at source ✅). A regulator has, in other words, written into the capital framework the requirement that a vendor be able to say *what it knew and when* — and that a bank be able to map that answer onto its own risk factors. The restatement problem is therefore not a technicality of quant research; it is a capital-relevant property of the price record.

**The observable consequence inside a bank.** Two teams ask for the same instrument's history on the same day and get different numbers, because one pulled it last year and cached it and the other pulled it this morning. Both are "right" relative to when they asked. Neither can prove which is which, because neither recorded when it asked. §6.3 treats this as *silent restatement*; §8.3 treats the remedy — an immutable vendor-raw store — as a storage decision.

### 2.3 Cause Three — The Instrument Is Not Fixed: Corporate Actions Change What the Identifier Refers To

**A security is not a durable object; it is a series of states, and a corporate action is a state transition.** A two-for-one split changes the number of shares outstanding and halves the price, so the price series that was continuous before the event is discontinuous after it unless somebody restates it. A merger retires the identifier and issues a new one — sometimes with a new ISIN for the new line, sometimes with the surviving line keeping its number and the surviving *issuer* changing identity. A spin-off creates a new instrument that has no history. A rights issue creates a new line that exists for weeks and then vanishes. An ISIN that means one thing on Monday can mean something materially different on Tuesday, and it will still be the same ISIN.

**This is the cause that breaks the other two.** The identifier crosswalk (cause one's mitigation) has to be effective-dated *because* of cause three: the mapping changed when the action took effect. The vendor disagreement (cause two) is most acute *because* of cause three: the two providers may be at different stages of applying the same event to their history — one has adjusted, one has not, and both are describing the same instrument on the same date. And the temporal model (cause two's mitigation) must be able to say *which state of the instrument* a price belongs to, not merely *which instrument*. §5 is about this cause; §7 shows how it propagates.

**The honest framing for the practitioner.** The corporate-action *processing* problem — did we elect in time, did we claim the entitlement, did the custodian pay — is an operations problem, and the repository covers it in [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §4. The corporate-action *data* problem is different: it is that every corporate action is, from the data model's point of view, a *retroactive edit to history that must be recorded as an event rather than applied as an overwrite*. A firm can be flawless at the first and structurally incapable of the second, and that is a very common state of affairs.

### 2.4 Cause Four — The Market Is Global: Calendars, Conventions, Regimes, Settlement Practice

**The same economic fact is recorded differently in different jurisdictions, and the differences are not errors.** Trading calendars differ (a price that is missing because the venue was closed is not a missing price); settlement cycles differ; market conventions differ (day-count, quoting convention, ex-date offset conventions, dividend withholding practice); the *regulatory* regimes differ (which identifier is mandatory for which report); and the *naming* of the same organisation differs across providers. A firm that runs one global process assumes one of these differences away, and the assumption eventually produces a break that nobody can explain because the explanation is "the convention is different there and we never wrote it down."

**Two identifiers in §3 illustrate the point concretely.** The Market Identifier Code (MIC, ISO 10383) is published monthly with modifications effective on the fourth Monday, and the registration authority is explicit that only representatives of the market organisation that operates the venue are entitled to request changes — so a firm's view of "the venue" is a view of a list that is itself monthly, versioned and free (✅ verified at source). And the LEI standard itself was republished as a *part* structure — ISO 17442-1:2020 *Part 1: Assignment*, reviewed and confirmed in 2026 — which means a firm's documentation citing "ISO 17442" is citing a superseded edition (✅ verified at source; see §14 for the discrepancy note against the sibling guide's citation).

### 2.5 How the Four Causes Generate Exactly Three Problems

The causes are structural properties of the market. The problems are the operational shapes those properties take when a firm tries to publish numbers.

| Structural cause | The problem it generates | The section that treats it |
|---|---|---|
| The identity is contested (§2.1) | **The identifier problem** — you cannot join two sources on a key nobody agrees on, and the crosswalk you build is only as good as its effective dates | §3 |
| The data is not fixed (§2.2) | **Point-in-time erosion** — the number you report for a past date can legitimately change, and you may have no record of having used the old one | §6 (remedy §8.3) |
| The instrument is not fixed (§2.3) | **Both, at once** — the crosswalk needs dates *because* the instrument changes state, and the two vendors disagree *because* they are at different states of applying the same event | §5, and the synthesis at §7 |
| The market is global (§2.4) | **Cross-vendor disagreement** — two providers with different calendars, conventions, sourcing and coverage cannot be made to agree by assertion | §4 |

The asymmetry in that table is the guide's point. Causes one and two each generate one problem; cause three generates both simultaneously, which is why corporate actions are the hinge (§7); and cause four is the reason reconciliation is a *control discipline with agreed tolerances* rather than a comparison that someone eyeballs (§4.3).

**The practical corollary, stated once so it can be referenced later.** Any firm can pick two of the following three and get them cheaply: *coverage* (we have every instrument), *correctness* (every field is right by some definition), and *provability* (we can demonstrate what we knew and when). Integrity work is the recognition that the third is not optional, because without it the first two are unfalsifiable — and the third is the one that requires the storage, the effective dates and the audit trail that nobody's roadmap asks for.

---

## 3. The Identifier Problem — The Symbology Stack

### 3.1 The Stack at a Glance

**A "symbology" is not a product or a table; it is the firm's position on how it will name things.** The stack below is the set of identifier *types* a securities-capable firm carries, with the issuing authority verified at source this pass. The ownership column is the highest-risk class in this whole guide, because this space changes hands: an assertion made from memory about who issues an identifier today is exactly the kind of claim that is wrong by the time it is read.

| Identifier | Standard | Issuer / allocating body | Ownership and current status (verified this pass) | Type of thing it names |
|---|---|---|---|---|
| **ISIN** | ISO 6166 | The National Numbering Agencies (NNAs); **ANNA is the Registration Authority appointed by ISO**; NNAs allocate within their national prefix | ANNA is a member association of NNAs; the **ANNA Service Bureau (ASB)** is *built and operated by ANNA partners CUSIP Global Services and SIX Financial Information*, collecting from **120+ NNAs** since **2001**, covering **200+ jurisdictions** ✅ — with the ISO 6166 number itself cited by ANNA and **not opened at iso.org this pass** ⚠; LSEG's Symbology page states that **London Stock Exchange Plc is the sole issuer of ISIN, FISN and CFI codes for Great Britain, Jersey, Guernsey and the Isle of Man** ✅ — a concrete illustration that ISIN allocation is national, and that the FISN and CFI codes sit alongside it at the numbering-agency level | A specific security (global scope, national allocation) |
| **CUSIP** | The US/Canada national numbering system (CUSIP Global Services is the US NNA) | **CUSIP Global Services (CGS)**; the system is stewarded "in close partnership with the American Bankers Association" | **CGS is owned by FactSet** — acquisition **completed 1 March 2022, approximately US$1.925bn**, announced 27 December 2021, acquired from S&P Global ✅ | A security issued in the US or Canadian markets |
| **SEDOL** | The UK/national numbering system operated by the exchange | **SEDOL identifiers are assigned by the London Stock Exchange (LSEG)**, on the request of market participants; delivered through the **SEDOL Masterfile** | LSEG operates the SEDOL Masterfile as a near-real-time reference dataset; LSEG is the London Stock Exchange Group, which acquired Refinitiv (completed 29 January 2021 — verified in [Market Data Consumption](market_data_consumption_guide.md) §3) ✅ / LSEG's SEDOL Masterfile page says in operation over 30 years, while LSEG's Symbology page says over 40 years — both LSEG's own words, recorded as an unresolved inconsistency ⚠. The Symbology page adds: issued at the country level for most asset classes (equities, fixed income, funds, exchange traded products, indices, structured products, warrants, certificates, exchange traded derivatives and digital assets), covering **over 22 million active securities**, and allocated **at a more granular level than ISIN codes** ✅ | A security, at country level, on request |
| **RIC** | Proprietary vendor symbology (no open standard) | **LSEG** — LSEG's own symbology page states that the RIC is a market-level identifier for instruments and pricing sources issued by LSEG | **Verified at source this pass ✅** (lseg.com Symbology page, September 2026); LSEG's acquisition of Refinitiv completed 29 January 2021 per [Market Data Consumption](market_data_consumption_guide.md) §3 | A quote or instrument *as the vendor's platform names it*, including chain/venue context |
| **FIGI** | The **OMG FIGI** specification (Object Management Group) | **Bloomberg L.P. as Registration Authority under OMG auspices**; **Bloomberg and Kaiko as Certified Providers** | The standard is published by OMG; the FIGI is **fee-free open data** — free to use, free to issue, free to redistribute ✅. A press description also refers to OMG **and X9** auspices ⚠ | Any financial instrument, across asset classes, at multiple levels of granularity |
| **LEI** | **ISO 17442-1:2020** *Part 1: Assignment* (published 2020-08; confirmed 2026; supersedes ISO 17442:2019) | **Local Operating Units (LOUs)** issue; **GLEIF** administers the global system | GLEIF is a not-for-profit foundation **established 2014 by the FSB**, overseen by the **Regulatory Oversight Committee (ROC)**, G20-endorsed; GLEIF makes the Global LEI Index open and free ✅ | A **legal entity** — not an instrument, and not a natural person |
| **UPI** | **ISO 4914:2021** (published 2021-11-16; ISO/TC 68/SC 8) | The **Derivatives Service Bureau (DSB)** — designated **sole service provider for the UPI system** and operator of the UPI reference data library | The DSB's designation includes a **nomination by the Financial Stability Board**; the reference data library is the DSB's ✅ | An **OTC derivative product** reportable to a trade repository |
| **UTI** | ISO 23897 ⚠ (number seen in circulation but not re-verified at ISO this pass) | **No central issuing body**: the UTI is generated by the reporting party per harmonised rules | The primary anchor is **CPMI-IOSCO, *Harmonisation of the Unique Transaction Identifier — Technical Guidance*, February 2017** ✅ | An **OTC derivative transaction** (the event, not the product) |
| **MIC** | **ISO 10383** | **SWIFT (S.W.I.F.T. SC, La Hulpe, Belgium) is the Registration Authority** | Published **monthly** — second Monday, modifications effective the fourth Monday; free of charge; the list seen this pass was published **14 September 2026** ✅ | An **exchange, trading venue or trade-reporting facility** |
| **CFI** | ISO 10962 | **ANNA states that it is the Registration Authority for ISIN and CFI, under appointment by ISO** | ANNA's own service site states the appointment ✅ for the RA role; the **ISO 10962 number and the standard's current edition were not opened at iso.org this pass** ⚠ | A **classification** of an instrument by category and attributes |
| **ISO 15022 / ISO 20022 corporate-action messages** | ISO 15022 (the SWIFT MT securities message set) and ISO 20022 (the seev family) | SWIFT for ISO 15022; the ISO 20022 registration and maintenance structure for the seev messages | The message *flows* and the migration milestones are verified in [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §4.4 ✅ — cite it, do not re-derive | A **corporate-action event, its instruction and its confirmation** |

### 3.2 Identifier by Identifier: Purpose, Issuer, Scope, Limits

**The limits column is the part that matters, because it is what determines what you can join on.** Each identifier below gets its purpose, its handler of record, its scope, and what it cannot do.

- **ISIN (ISO 6166).** *Purpose:* one code per security, globally agreed, so that a cross-border instruction can name a security unambiguously. *Issuer:* the NNA for the instrument's home market, with ANNA as ISO's Registration Authority; the consolidated view is the ANNA Service Bureau, operated by CUSIP Global Services and SIX Financial Information ✅. *Scope:* the widest of the securities identifiers — equities, debt, derivatives, indices, in 200+ jurisdictions according to ANNA ✅. ***Limits:*** (i) it identifies a *security*, not a *listing* — the same ISIN trades on many venues, so an ISIN alone cannot tell you which venue a price came from (that is what a MIC is for); (ii) it is not issued for every instrument — OTC derivatives, syndicated loans and some private and structured instruments are outside it, which is exactly the gap the UPI and FIGI exist to address; (iii) it is *not stable across certain events* — a reorganisation can retire an ISIN and issue a new one for what an investor experiences as the same holding; (iv) the national prefix tells you where it was allocated, not where the risk is; and (v) the ANNA Service Bureau is an aggregation of national authorities' data, so its completeness inherits each NNA's coverage decisions. **The join consequence:** an ISIN-keyed crosswalk is the only one that will join across vendors at all reliably — and it still will not tell you the venue, the entitlement, or the state of the instrument on a given date.
- **CUSIP.** *Purpose:* the North American national numbering system, and the identifier embedded in US settlement and clearing practice. *Issuer:* CUSIP Global Services, which is also the US NNA and therefore also allocates US ISINs; CUSIP Global Services has been owned by FactSet since the acquisition completed 1 March 2022 ✅. *Scope:* US and Canadian securities. ***Limits:*** (i) it is *national* — a firm with a global book cannot make CUSIP the primary key without a crosswalk for everything else; (ii) the same economic exposure can carry different CUSIPs in different markets or currencies; (iii) CUSIP's role as the US NNA means the CUSIP and ISIN for a US security are issued by the same body, which makes them *look* interchangeable in a US-only dataset and hides the mapping problem until the firm goes global; and (iv) licensing — a national numbering agency's data is a licensed product, not a public good, which constrains redistribution inside the firm (§9.3). **The join consequence:** CUSIP is a strong key inside North America and a liability as a global primary key.
- **SEDOL.** *Purpose:* the UK market's national identifier, widely used for reference and reporting, delivered through the SEDOL Masterfile ✅. *Issuer:* the London Stock Exchange, part of LSEG; SEDOLs are assigned on the request of market participants ✅. ***Limits:*** (i) **allocation on request** means coverage follows demand, not the universe — an instrument nobody asked about may simply not have one, which is the opposite of a registry's behaviour; (ii) it is a national system used globally, so the "which jurisdiction issued it" intuition does not hold; (iii) LSEG describes the Masterfile as a near-real-time dataset under continuous change ✅, so the SEDOL *universe* is itself versioned; and (iv) the historical claim that SEDOL has been issued "since 1979" is **⚠ secondary** — the sibling guide flags it and this guide does not assert it, and **LSEG's own two pages disagree with each other on the age of the system** (over 30 years on the SEDOL Masterfile page, over 40 years on the Symbology page — both LSEG, recorded rather than smoothed); and (v) it is allocated **at a more granular level than ISIN** ✅, so a SEDOL and an ISIN for the same economic exposure are not one-to-one by construction. **The join consequence:** SEDOL is excellent for UK-centric joins and for instruments a market participant asked about, and is not a completeness guarantee.
- **RIC.** *Purpose:* the vendor platform's own instrument-and-context naming, which is why it threads through the vendor's terminal, its APIs and its data licences. *Issuer:* the vendor. ***Limits, stated bluntly:*** a RIC is **proprietary** — it has no meaning outside its vendor's ecosystem, its structure encodes the vendor's view of the listing chain and venue, and **it can change** when the vendor's view changes or when the underlying line changes. It is, in the most practical sense, the identifier a trading desk will say out loud and the worst possible primary key for a firm's data model. **The issuing body is verified at source:** LSEG's own symbology page describes the RIC as a market-level identifier for instruments and pricing sources **issued by LSEG** ✅ (read this pass). What is *not* published is the RIC's internal structure and port convention — LSEG's public material describes the identifier and its uses, not its syntax — so a firm documenting a RIC-based mapping must treat the structure as vendor-documented in the vendor's own product material rather than as a published standard, and must version it, because a proprietary code can change with the vendor's view.
- **FIGI (OMG specification).** *Purpose:* an explicitly *open* identifier — free to use, issue and redistribute — with **uniqueness at multiple levels of granularity**, precisely so that instruments which traditional symbologies struggle with (loans, OTC derivatives, commodities, crypto) can be identified, and so that a firm can tie fragmented symbologies together ✅. *Issuer:* issued and distributed by **Bloomberg L.P. as Registration Authority under the auspices of OMG**, with **Bloomberg and Kaiko as Certified Providers** ✅. ***Limits:*** (i) it is an **OMG standard, not an ISO/TC 68 financial-services standard** — so where a regime mandates an identifier, it does not mandate a FIGI; (ii) granularity is a *feature and a trap*: because FIGIs are assigned at multiple levels (share-class, venue-and-instrument, and so on), a join that mixes levels will silently double-count, and choosing the right level is a design decision the firm must make and document; (iii) coverage, while broad and asserted as spanning all asset classes, is provider-driven — the standard does not create the identifiers, the providers do; and (iv) an "open data" identifier is free to *use* but is not automatically free to *replace* a licensed national identifier in a downstream licence obligation. **The join consequence:** FIGI is the most attractive *cross-asset-class* join key of the set and the one most likely to be mis-joined by level.
- **LEI (ISO 17442-1:2020).** *Purpose:* to identify the *legal entity* — the party — unambiguously, so that exposures, counterparties and issuers can be aggregated across systems and across borders ✅. *Issuer:* LOUs (LEI Issuers) accredited under GLEIF's framework; GLEIF administers the system, publishes the open Global LEI Index, was established in 2014 by the FSB and is overseen by the ROC ✅. ***Limits:*** (i) it identifies an entity, **not an instrument** — an LEI will never be a security key; (ii) it excludes **natural persons** (the standard's own scope excludes them, while including individuals acting in a business capacity) ✅; (iii) it is stable for a given legal entity, but **legal entities restructure** — mergers, name changes and successor entities are events the firm must track, and an entity's identifier being stable does not mean the *entity* is the same entity it was; (iv) umbrella and sub-fund structures require care — the standard contemplates collective investment funds at umbrella and sub-fund level ✅, and getting the level wrong corrupts look-through exposure; and (v) the current edition is a *part* — ISO 17442-1:2020 — so documentation citing bare "ISO 17442" is citing a withdrawn edition ✅. **The join consequence:** the LEI is how you join instruments to *issuers* and is indispensable for issuer-level aggregation and for the regulatory regimes of §11.3; it is never a substitute for an instrument identifier.
- **UPI (ISO 4914:2021).** *Purpose:* unambiguous identification of OTC derivative **products** reportable to trade repositories, so that authorities can aggregate OTC derivative transactions by product and by reference-data element ✅. *Issuer:* the **DSB**, designated sole service provider for the UPI system and operator of the UPI reference data library, with the FSB's nomination behind it ✅. ***Limits:*** (i) it identifies a **product definition**, not a trade and not a listed instrument; (ii) it is scoped to the instrument categories the standard names — at minimum swaps, forwards, non-listed and complex listed options, and the "others/miscellaneous" category, which the standard cross-refers to the CFI classification ✅ — so it is not a universal derivative identifier by construction; and (iii) a bank's internal product taxonomy will not map one-to-one onto it, so the mapping is itself an effective-dated artefact. **The join consequence:** UPI is for regulatory aggregation and counterparty/trade-repository alignment; it will not join your trading book's instrument table to anything, and should not be asked to.
- **UTI.** *Purpose:* to identify a **transaction** for OTC-derivative reporting so that the same trade reported to different trade repositories can be recognised as one trade ✅. *Issuer:* **no central authority issues UTIs** — the harmonised guidance in CPMI-IOSCO's February 2017 *Technical Guidance* defines how the identifier is to be generated and by whom, and jurisdictions implement it ✅. ***Limits:*** (i) it carries no product content at all — product identity is the UPI's job, and conflating the two is a common modelling error; (ii) generation rules and allocation responsibility differ by regime, so the same economic trade can be represented differently across jurisdictions; and (iii) **⚠ the ISO 23897 standard number for the UTI was not re-verified at ISO this pass** — the number is in wide circulation but this guide does not present it as source-verified. **The join consequence:** UTI joins *reports about the same trade*; it never joins a trade to a security.
- **MIC (ISO 10383).** *Purpose:* to identify the **venue** that is the source of a price ✅. *Issuer:* SWIFT as Registration Authority ✅. *Scope and mechanics:* registered at operating/exchange level and at market-segment level, with segment MICs linked to their operating MIC; published monthly (second Monday), free of charge, modifications effective the fourth Monday, and only the market organisation that operates the venue may request a change ✅. ***Limits:*** (i) a MIC identifies a *place*, not an instrument on it — a MIC plus an ISIN plus a timestamp is the minimum triple for "where did this price come from"; (ii) MICs are **created and deactivated**, so a historical price's venue code may no longer be active, and a join against today's MIC list will silently drop history; (iii) segment-vs-operating ambiguity means "which MIC" is a modelling decision — the same venue can be identified at two levels; and (iv) a MIC says nothing about *who carries the venue's data* or on what licence terms. **The join consequence:** MIC is what makes a price record attributable, and it must be stored with the effective-dating of §3.3 or history becomes unattributable.
- **CFI (ISO 10962).** *Purpose:* to classify an instrument by category and attributes rather than to identify it; ANNA is ISO's Registration Authority for it, alongside ISIN ✅. ***Limits:*** a classification code does not identify anything uniquely — the identical CFI is shared by thousands of instruments, so it is a filter and a taxonomy key, never a join key in the sense of uniqueness. Its value is that it gives a *standard* vocabulary for "what kind of instrument is this," which is what makes cross-vendor consistency checks on instrument type possible at all (§4.3).

**A note on what this list implies for a design.** Reading the limits column end-to-end makes one thing obvious: **there is no single identifier that solves the join problem, so the firm's actual decision is which identifier is the *anchor* of the crosswalk**, and the answer is nearly always ISIN as the anchor with a set of *level* identifiers around it (MIC for venue, CFI for classification, LEI for issuer, UPI/UTI for the derivative reporting perimeter), plus vendor codes carried as attributes with their own validity windows rather than as keys. Anything else — using a vendor code as the anchor, or a national identifier as the global key — is a decision to be able to answer only the questions that identifier can answer, forever.

### 3.3 The Effective-Dated Crosswalk — A Mapping Table Without Dates Is a Time Bomb

**The crosswalk is the table that says which identifiers refer to the same instrument.** It is the single most consequential table in the security master, and the one most commonly built wrong, because the wrong version is easy to build: two columns, `isin` and `cusip`, loaded from a vendor file, refreshed daily. It works in testing. It fails in production in a specific, diagnosable way.

**Why it fails.** Vendor files are snapshots of *today's* mapping. Overwrite them daily, and the mapping you had in March is gone — so when an audit asks "what did you think this CUSIP mapped to when you generated the March risk report," the honest answer is "we cannot tell you," and the dishonest answer is to reconstruct it from today's file and hope. Worse, the daily overwrite *destroys the evidence of a change*: if the vendor changed the mapping last week, the firm cannot prove that the change happened, cannot quantify the impact on reports produced before it, and cannot answer the regulator's question about which reports were affected.

**The correct shape, and the minimum fields.** A crosswalk row is a statement about a period, not a fact:

```
identifier_crosswalk (
  crosswalk_id            -- surrogate key
  isin                    -- the anchor
  id_type                 -- 'CUSIP' | 'SEDOL' | 'RIC' | 'FIGI' | 'MIC' | 'LEI' | 'UPI' | ...
  id_value                -- the value of that identifier
  valid_from              -- the date this mapping became true (valid time)
  valid_to                -- the date it ceased to be true, NULL if current (valid time)
  source_vendor           -- who told us, and under which product
  knowledge_from          -- when we first knew it (transaction time)
  knowledge_to            -- when we stopped knowing it (transaction time)
  confidence              -- asserted-by-vendor | confirmed-by-two-vendors | overridden
  override_reason         -- required when confidence = overridden
)
```

Three properties make this a crosswalk rather than a snapshot. **Effective dates on both axes** (§6.1, §8.2): valid time says when the mapping was true in the world, knowledge time says when the firm learned it. **Non-overwriting**: a change inserts a row and closes the previous one; it never updates in place (§13, anti-pattern three). **Provenance**: every mapping carries who asserted it, because a mapping asserted by one vendor and contradicted by another is a break with an owner (§4.4), not a row to be silently trusted.

**The failure modes the dates expose.** With a properly dated crosswalk, four questions that were previously unanswerable become routine: *which instruments had a mapping change last quarter, and which reports consumed the old mapping*; *whether a vendor's mapping change preceded or followed the report we filed*; *which crosswalk rows are asserted by a single source with no confirmation*; and *which rows have a gap* — a period covered by no mapping at all, which is where joins silently drop rows and totals silently under-count. That last one deserves emphasis: **a gap in a crosswalk does not produce an error message; it produces a smaller number.** The position was there, the price was there, the join found nothing, the aggregate came out lower, and nobody noticed until a reconciliation with an external party disagreed by an amount nobody could attribute.

### 3.4 The Identifier Lifecycle — What Causes an Identifier to Change

An identifier changes for reasons that fall into five families, each with a distinct detection problem. The list is the checklist a security-master team should be able to answer for its own book.

| Change family | What happens | The detection problem | The data-model response |
|---|---|---|---|
| **Corporate-action-driven** | A split, merger, reorganisation, redenomination or change of listing retires an identifier and issues a new one, or keeps the number but changes what it refers to | The vendor supplies a new row; nothing in the data itself says "this replaces that" — the link is the event, not the row | The corporate-action ledger must carry the identifier linkage as a first-class field (§8.4) |
| **Venue-driven** | The instrument moves venue, a segment MIC is created or deactivated, a market restructures | A MIC join against today's list silently drops history (§3.2) | Effective-date the MIC and keep deactivated codes resolvable |
| **Legal-entity-driven** | The issuer merges, renames, or restructures; umbrella and sub-fund relationships change | The LEI is stable for the entity but the entity's *identity as you care about it* is not | Entity history tracked separately from instrument history; LEI for the party, instrument identifiers for the security |
| **Provider-driven** | A vendor changes its own codes, renames a line, changes the level at which it assigns a code (FIGI levels are the canonical example) | The change appears as a diff in a file with no event behind it | Vendor codes carried as dated attributes, never as keys; provider change notices processed as change-controlled events (§10.5) |
| **Correction-driven** | The identifier was wrong: a mis-mapping, a duplicate, an issue that was never allocated one | Only detectable by reconciliation against a second source or against the numbering agency | The reconciliation control of §4 is the only systematic detector |

**What "identifier lifecycle management" means in practice**, therefore, is not a data-quality dashboard. It is the combination of the effective-dated crosswalk (§3.3), the corporate-action ledger (§8.4), the vendor change-notice process (§10.5) and the reconciliation control (§4) — four mechanisms that only work together, which is the first appearance of the guide's recurring pattern.

### 3.5 The Rule: The Identifier You Standardise On Determines What You Can Ever Join On

**The rule, stated as a rule.** Choose the anchor identifier on the basis of the *questions you will need to answer*, not the convenience of the file in front of you. The choice is close to irreversible in practice, because every downstream system, every report and every model inherits it.

| If you anchor on… | You can join | You cannot join (or join only by building a crosswalk you now own forever) |
|---|---|---|
| **ISIN** | Cross-vendor instrument records; regulatory reports that use ISIN; the ANNA-aggregated universe; most third-party data | Venue-level price attribution without a MIC; entity-level aggregation without an LEI; OTC derivatives without a UPI; anything outside the ISIN allocation perimeter |
| **CUSIP** | US/Canada settlement and clearing; the US NNA's own ISIN allocation | Non-North-American instruments (arbitrarily); global regulatory reports; anything requiring an unambiguous global key |
| **A vendor code (e.g. RIC)** | That vendor's platform, end to end, cheaply | Every other vendor; the firm's own history if the vendor changes its codes; the firm's own independence from the vendor's commercial decisions |
| **FIGI** | Across asset classes including loans, OTC derivatives, crypto and other instruments traditional symbologies struggle with ✅ | Regulatory mandates that name another identifier; joins that require a single granularity level without an explicit level rule |
| **LEI** | Issuer-level and counterparty-level aggregation, look-through, entity risk | Any instrument-level question at all — an LEI is not an instrument key |

**The corollary that catches firms out.** Whichever identifier you anchor on, the *second* identifier in your stack is the one that will generate the most mapping work, because it is the one you will be tempted to treat as equivalent. For a globally oriented firm that is usually ISIN plus CUSIP (or ISIN plus SEDOL) — near-equivalent-looking, differently scoped, differently licensed, differently versioned. The guide returns to this in §12.3, where a Cymbal Bank break turns out to be exactly this problem wearing a corporate action as a disguise.

---

## 4. Cross-Vendor Reconciliation

### 4.1 Why Two Vendors Disagree About the Same Instrument

**Two providers disagree because they are answering a slightly different question at a slightly different time with a different method — and none of that is a defect.** The productive move is to categorise the cause, because the category determines who can resolve it and whether it should be resolved at all. The seven families below cover the great majority of real breaks; the eighth row is the one that is not a data difference at all.

| Cause family | What it looks like | Who resolves it |
|---|---|---|
| **Timing / as-of** | Prices for the same date differ because the providers cut off at different times, incorporate late-reported trades on different cycles, or stamp a value with a different effective timestamp | Data operations, with the vendor's own documentation of its cut-off |
| **Sourcing** | One provider takes the value from the primary venue's own feed; the other takes it from a consolidator, a contributor panel, or an evaluated model | Data management (this is a *policy* decision, not an error) |
| **Methodology** | Evaluated pricing models, curve construction, rounding and FX-rate selection differ; the providers may each be internally consistent and mutually inconsistent | Risk/finance sign-off — methodology differences are owned by the consumer of the number, not by the feed team |
| **Correction state** | One provider has applied a correction or an amendment that the other has not yet applied (or will never apply) | Reconciliation process — detect, date, decide (§4.4) |
| **Commercial coverage** | One provider simply does not carry the instrument, the venue or the field (an entitlement or coverage boundary, not a wrong answer) | Data management plus licensing — a coverage gap must be *declared*, not treated as a break |
| **Definitional / semantic** | The field means different things — "issue date" as first issue versus first settlement; "coupon" as current versus original; quantity as shares outstanding versus free float | Data governance (business glossary) — and the field-level mapping in the data contract (§9.3) |
| **Identification** | The two records are not about the same instrument at all: a crosswalk row differs, or one vendor has already applied an identifier change the other has not (§3.4) | Security master — this is the break that *looks* like a data defect and is actually a mapping defect |
| **Genuine, unresolvable market practice** | Providers hold different but defensible views of a date or a term, with no authoritative arbiter | Nobody. Record both, choose one under change control, document the choice (§5.7) |

**The most important line in that table is the last one.** A reconciliation programme that assumes every difference has a right answer will burn its investigators on the 5–10% of breaks that have no right answer, and will therefore never build the discipline to handle the rest. The control's job is to *classify and decide*, not to force agreement.

### 4.2 Reconciliation as a Control, Not a Comparison

**A comparison is a report. A control has an objective, a tolerance, an owner, evidence, and a consequence.** The distinction is the whole difference between a reconciliation that improves data and one that produces a monthly PDF nobody opens.

| Property | A comparison | A control |
|---|---|---|
| **Objective** | Show differences | Assure that the firm's published instrument record is right, or that it knows where it is wrong |
| **Threshold** | Whatever the report happens to highlight | Pre-agreed tolerances per field type, documented and approved by the consuming business (§4.3) |
| **Ownership** | Whoever ran the report | A named owner per break class, with an escalation path (§10.2–10.3) |
| **Evidence** | The current report | The retained input answers *and* the report, so the break can be re-investigated as of the date it was raised (§4.6) |
| **Consequence** | None | Breaks age, escalate, and eventually breach an SLA with a defined management action |
| **Coverage** | The fields someone thought of | A declared scope: which fields, which asset classes, which universe, and what is *outside* the tolerance regime |

**Where reconciliation sits among the controls.** In control terms, cross-vendor reconciliation is a **detective** control: it finds problems after they exist. The **preventative** controls in the same chain are validation on ingest (rejecting structurally impossible records), the effective-dated crosswalk (refusing to join on an undated mapping, §3.3), and the data contract at the interface (§9.3). A firm with only detective controls discovers its problems in the reconciliation queue, which is expensive; a firm with only preventative controls discovers its problems when a consumer complains, which is worse. The pattern that works is a thin preventative layer plus a detective layer with teeth — and the detective layer needs the tolerances that make "with teeth" definable.

**One control-design point that is easy to miss.** The reconciliation must be **independent of the process it controls**. If the same team that loads vendor A's file also decides whether the difference between A and B is acceptable, the control has an incentive problem, and it will be visible in the audit trail as a queue where every break is closed within a day and almost none carries a root cause. Ownership of *investigation* and ownership of *tolerance-setting* should not sit with the same person.

### 4.3 The Tolerance Categories

**Different field types cannot share a tolerance, because "the same value" means something different for each.** A price is the same within a basis-point band; an identifier is the same or it is not; a corporate-action date is the same day or it is not, but a corporate-action *term* can differ by rounding. The table below is the categories a firm should be able to point at in a written policy; the specific numbers are **firm-specific and must be agreed with the consumers**, not copied from any guide, including this one.

| Category | Matching rule | Why it needs its own tolerance | What a break here implies |
|---|---|---|---|
| **Identifier fields** (ISIN, CUSIP, SEDOL, MIC, LEI) | **Exact** | An identifier is either right or wrong; a "near miss" is a different instrument | A security-master or crosswalk defect — high severity, usually affects many records |
| **Price and valuation fields** | Numeric, within a per-asset-class basis-point band (wider for illiquid and evaluated instruments, tighter for liquid exchange-traded ones) | The band depends on how the price is produced: an exchange close is deterministic, an evaluated bond price is a model output | Either a timing/correction difference (benign) or a methodology difference (a policy decision) |
| **Static / descriptive attributes** (name, currency, coupon, maturity, issue date, country) | Exact for enumerable fields (currency, country, maturity date); normalised-match for free text (name, description) | Free text will never match character-for-character across providers, so an exact-match rule generates 100% noise and the control is abandoned | A definitional difference (glossary), a correction, or an identification problem |
| **Classification / taxonomy fields** (instrument type, CFI, sector, asset class) | Exact on the standard code; **mapped** where the providers use their own taxonomies | Providers legitimately use proprietary taxonomies, so the comparison is between *mappings*, not codes | A mapping table defect, or a coverage difference |
| **Corporate-action fields** (event type, ex-date, record date, pay date, rate/ratio, mandatory flag) | Exact on dates and flags; ratio/rate within a rounding tolerance; text normalised | Dates and mandatory-flags drive downstream entitlement logic, so a one-day difference is not a rounding issue; ratios are quoted to differing precision | The most consequential break class — see §5.5 |
| **Entity / counterparty fields** (LEI, legal name, domicile) | Exact on LEI; normalised on name; **level-checked** for fund structures | LEI level errors (umbrella vs sub-fund) are common and silently wrong | An entity-reference defect (§3.2, LEI limits) |
| **Timestamps** | Within a clock-skew tolerance | Providers use different clocks, time zones and stamping conventions | Usually benign; matters only for intraday sequencing |
| **Coverage / missing records** | Presence test — a record expected and absent | Absence is not a "difference"; it needs its own rule and its own break class | A coverage, entitlement or crosswalk-gap problem (the dangerous one: missing rows shrink aggregates, §3.3) |

**Three rules that make tolerances real rather than decorative.** First, **per-field and per-asset-class** — a single firm-wide percentage is a way of not having a policy. Second, **asymmetric where the consequence is asymmetric** — for corporate-action dates, the tolerance should differ between "we are a day late" and "we are a day early," because the downstream payment logic does. Third, **agreed with the consumer and reviewed on a schedule**, because a tolerance set by the data team alone has no authority when a break is contested; the escalation path (§10.3) needs a prior agreement to escalate *to*.

**A regulatory reason why the price tolerance is not purely a matter of taste.** Under the Basel Framework's model-eligibility requirements, a price counts as a *real* observation for the risk factor eligibility test only if it is a transaction price, a verifiable price from an arm's-length transaction, a committed quote collected and verified through a third-party vendor, trading platform or exchange, or a vendor price meeting those conditions — and collateral reconciliations and valuations explicitly cannot be used to meet the test (MAR31.12, verified at source ✅). A bank therefore cannot treat a reconciliation agreement *about* a price as equivalent to having a real price; the two questions — "do our two feeds agree?" and "is this a real price for capital purposes?" — are different, and only the second is capital-relevant.

### 4.4 The Break Workflow

**A break is not a difference; it is a difference that crossed a tolerance, was raised as an item, and acquired an owner and an age.** The workflow below is the minimum viable version; the state names are deliberately plain so that a firm can map its own tooling onto them.

| Stage | What happens | The artefact that proves it happened |
|---|---|---|
| **Detection** | The reconciliation run compares source A and source B (or source and golden copy) and emits differences | The run log, with the inputs identified by version and receipt time |
| **Classification** | The difference is bucket-matched against the tolerance table and given a cause family (§4.1) | The break record's `cause_family` field |
| **Investigation** | An investigator reconstructs what each source said, and when | The retained vendor answers (§4.6) — without them this step is impossible |
| **Ownership** | The break is routed to the owner of the field or the source, per a written RACI | The break's `owner` field, with the routing rule referenced |
| **Aging** | The break ages against an SLA expressed in age buckets (e.g. same-day, week, month, quarter), tiered by severity | The break's `raised_at` and `age_bucket` |
| **Escalation** | Breaks exceeding an age or value threshold escalate to a named forum (§10.3) | The escalation record, and the forum's minutes |
| **Resolution** | One of: correct our record; obtain a correction from a vendor; accept one source as authoritative for this field; or classify as unresolvable market practice and record the choice | The resolution record, with the evidence attached |
| **Override with audit trail** | When the accepted value is not the default source's value, the override is recorded with reason, approver, and expiry | The override record — **an override without this is an anti-pattern (§13), not a resolution** |

**Aging is the control's teeth.** A break queue without age buckets is a queue that will fill and stay full; the practical failure mode is a large population of breaks aged "months" that everyone has decided is normal, which is functionally identical to having no reconciliation. The design goal is a *small, current* queue: most differences auto-classified as within tolerance or as known methodology differences, a small number of real breaks with real owners, and a monthly number that the risk or finance forum actually challenges.

**Two workflow traps worth naming.** The **auto-close trap**: an automation that closes breaks when they disappear from the next run's output. The break did not get resolved; the vendor's new answer happened to match, which is exactly the restatement case (§6.3) and is the moment the firm should have recorded a knowledge-time event. The **override-exception trap**: overrides proliferate until the "golden copy" is a patchwork nobody can explain, which is §4.5's consistency problem arriving through the break queue.

### 4.5 The Golden-Source Pattern and Its Consistency Problem

**The pattern.** For each data field (or family of fields), designate one authoritative source — the golden source — and feed every consumer from the golden record rather than from each consumer's own vendor feed. This is the pattern described from the taxonomy side in [Capital Markets Architecture](capital_markets_architecture_guide.md) §5.3 and [Market Data Consumption](market_data_consumption_guide.md) §9.1; what follows is the *design consequence* those summaries do not reach.

**Best-of-breed per field is attractive and creates a specific defect.** The obvious refinement — take prices from the vendor strongest in equities, corporate actions from the specialist, terms and conditions from a third, entity data from the LEI system — produces what practitioners typically call a Frankenstein record: a single instrument row whose fields came from four providers, three of which have never seen each other's data. The defect is **internal inconsistency**: the price series is adjusted according to vendor A's corporate-action convention while the quantity and terms come from vendor B, so the arithmetic that should tie (price × quantity, adjustment factors against the event ledger) provably does not. The failure is silent, because every individual field is within its tolerance and every individual feed is working.

**Four mitigations that actually hold.**

- **Field-level provenance is mandatory.** Every field in the golden record carries its source and its knowledge date (§8.2). Without this, an inconsistency investigation has no starting point.
- **Arithmetic ties are controls, not tests.** Price × quantity against a market value, adjustment factors against the event ledger, currency conversions against the stated FX source — these cross-field ties catch exactly the inconsistencies per-field reconciliation cannot see, because they compare fields to *each other* rather than each to a second vendor.
- **Anchor coherence at the event.** Where a corporate action drives both a price series and an identifier (the common case), take *both* from the source that owns the event, or re-derive both from the firm's own event ledger (§8.4) rather than mixing sources across the event boundary.
- **One reference vendor for whole-record comparison, on a schedule.** Best-of-breed sourcing is fine day to day; a periodic full-record comparison against a single reference provider is what detects the drift that per-field tolerances were designed not to see.

**The honest trade-off.** Best-of-breed raises per-field quality and lowers whole-record coherence; single-vendor sourcing does the reverse, and also concentrates commercial and coverage risk. Neither is right in the abstract. What is *always* wrong is choosing best-of-breed per field without the four mitigations above — because the firm then has the costs of multi-vendor sourcing and none of the evidence that would let it manage them.

### 4.6 The Storage Requirement That Makes Reconciliation Possible

**You cannot investigate a disagreement you did not record.** This is the single most frequently violated requirement in the whole discipline, and it is a storage decision made once and lived with for years.

**What must be retained, at minimum.**

| Artefact | Why | Retention design point |
|---|---|---|
| **The vendor's raw answer, as received** — the file or message, unmodified, with its own timestamps | The vendor's answer is the *evidence*; once transformed, it is the firm's opinion instead | Append-only landing zone; never overwrite; store the file identity and checksum |
| **The receipt timestamp and sequence** | Distinguishes "the vendor changed the value" from "we loaded it at a different time" | Server-side receipt time, immutable, on the record not in a log file |
| **The vendor's effective/as-of dates** | A vendor's answer is for a date; a firm that stores it without the vendor's own date cannot reconcile anything as-of | Separate the two: `vendor_as_of_date` and `received_at` |
| **The reconciliation run and its inputs** | To re-investigate a closed break, the run's inputs must still exist | Store the run's input version identifiers, not a copy of the inputs |
| **The prior golden values** | Detects restatement and computes impact (§6.3) | Effective-dated, non-overwriting golden record (§8.2) |
| **The override records** | The audit trail the regulator asks for (§10.6) | Immutable, with approver and reason |

**Two design choices that cost almost nothing and save enormous effort later.** First, **never transform in the landing zone**: the raw answer is loaded as received, and all normalisation happens downstream where it is a *derived*, reproducible step ([../technology/data/data_pipeline_versioning.md](../technology/data/data_pipeline_versioning.md) and [../technology/data/backfill_data_engineering.md](../technology/data/backfill_data_engineering.md) cover the mechanics — cross-reference, do not re-derive). Second, **make the retention period an explicit, written decision** with a date and an owner, because the default in most organisations is "as long as the disk is cheap, until a cost-reduction programme deletes it," and the deletion is invisible until the first audit or the first break that needs history older than the retention window.

**The regulatory frame that makes this concrete rather than philosophical.** Under MAR31.14 (✅ verified at source) a bank relying on vendor real prices for its risk-factor eligibility test may do so only if the vendor communicates the **number of corresponding real prices observed and the dates at which they have been observed**, provides a minimum necessary set of identifier information to map those prices to risk factors, and is subject to an audit of its pricing information whose results are available to the supervisor. A firm that does not retain what its own vendors said, with dates, cannot demonstrate any of that on demand — which is what turns "we should keep the raw files" from an engineering preference into a supervisory expectation.

---

## 5. Corporate Actions — The Lifecycle, Not a List

### 5.1 The Lifecycle and the Control Point at Each Stage

**Corporate actions are where the data model and the operations function meet, and this section treats only the data side.** The event *types*, the operations workload (elections, claims, withholding tax), and the custodian message handling are [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §4 and are not re-derived here. What this section adds is the lifecycle read as a *data pipeline with a control point at every stage*, and what each stage does to the instrument's stored history.

| Stage | What the data side must capture | The control point | The failure mode if it is missed |
|---|---|---|---|
| **Announcement** | The event as first published: type, terms, announced dates, the issuer and the affected identifier(s), the source and the *announcement timestamp* | Record the announcement as an immutable, dated fact — an announcement is an *early, often incomplete* statement, not the truth | The firm has no record of what it knew before the terms changed — the point-in-time hole of §6.3 |
| **Terms confirmation and amendments** | Every version of the terms, with the version's own validity period | **Track amendments as separate versions.** Terms genuinely change | The firm holds one row for a multi-version event and cannot reconstruct what it acted on |
| **Ex-date and record date** | Both dates, and the convention that links them (which depends on the settlement cycle in the market) | Verify the ex-date against the market's convention, not against a single feed | Entitlement is determined on the wrong date; downstream claims and reversals |
| **Election window** (voluntary events) | The election options, the deadline, and the *firm's* election and the time it was made | Deadline management — the control is a clock, not a data check | Missed deadlines produce claims, and the missed-election record is itself audit evidence |
| **Allocation / entitlement** | What the firm was entitled to, per legal entity and per account | Reconcile entitlement to the position on record date | The accounting-side break treated in [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §5 |
| **Payment / delivery / effective** | The cash or securities movement, and the date it became effective | Confirm the event *completed*, not merely that it was announced | The instrument's record reflects a state change that did not happen (or happened on a different date) |
| **Post-event identifier and quantity reconciliation** | The resulting identifier(s), the quantity ratio, the new terms, and the linkage between the pre- and post-event identifiers | Reconcile the *instrument record* — not just the money — against a second source | The firm's instrument record silently keeps the pre-event identifier and quantity, and every subsequent price is wrong by a factor (§5.6) |

**Two facts about the dates make the table operationally sharp.** First, the **record date** is the date on which holders of record are entitled; trades settling after it do not carry the entitlement, and the **ex-date** is the date from which the security trades without it — the relationship between the two is a *market convention* that depends on the settlement cycle ([Investment Portfolio Operations](investment_portfolio_operations_guide.md) §4.3 for the lifecycle, §3.1 for the settlement cycle, including the US move to T+1 from 28 May 2024 verified there ✅). Second, **T+1 compresses the corporate-actions window**: a shorter settlement cycle shrinks the gap between trade and entitlement, which reduces some risk and removes the slack that operations teams used to absorb date mismatches — so a date break that was previously absorbable now lands on the payment.

### 5.2 Mandatory vs Voluntary and the Control Consequences

**The distinction that matters for data is not "does the holder choose" but "what does the event do to the instrument's history."** Mandatory actions ([Investment Portfolio Operations](investment_portfolio_operations_guide.md) §4.2) apply to all holders with no election and are, from the data model's point of view, **bulk restatement events**: a split rescales everyone's quantity and price; a cash dividend adjusts the price series under most research conventions. Voluntary actions require an election and are, from the data model's point of view, **conditional state changes**: the instrument may or may not be replaced depending on what holders elect and whether the event completes, and the firm's own election is itself a recorded fact that belongs in the audit trail.

| Dimension | Mandatory | Voluntary (and mandatory-with-options) |
|---|---|---|
| Volume | High — every instrument, every period | Lower, but time-critical |
| Data-model effect | Restatement of the historical series (price, quantity, or both) | Conditional replacement of the instrument, plus a recorded election |
| Primary risk | Silent under-processing: the event applies and the firm's series does not get updated, so history is wrong by a factor | Missing a deadline, or holding a terms version that was subsequently amended |
| Control emphasis | Completeness: did we see *every* event for *every* instrument on the book? | Timeliness and version-tracking: did we act on the right terms in time? |
| Where it lives | The event ledger drives adjustment (§8.4) | The ledger plus the election record, owned with the operations workflow ([Investment Portfolio Operations](investment_portfolio_operations_guide.md) §4.5) |

**Completeness is the mandatory-event control that most firms under-build.** Because mandatory events are numerous and individually unremarkable, the systematic risk is a *missed* event rather than a mishandled one — and a missed split is not visible as an error, because the resulting price series looks perfectly plausible; it is simply wrong by a factor of two. The detector is the reconciliation of §4 run against a second vendor's event feed, with a tolerance regime that treats "one vendor shows an event the other does not" as a high-severity break class in its own right (§4.3, coverage row).

### 5.3 The Principal Event Classes and What Each Does to the Instrument's History

| Event class | Effect on quantity | Effect on the price series | Effect on identifier / terms | The record that must survive |
|---|---|---|---|---|
| **Cash dividend** | None | Price drops by the distribution on the ex-date; the series is only continuous if *someone* decides how to treat the dividend | None | The rate, the ex/record/pay dates, and **the adjustment convention applied** (§5.6) |
| **Stock split / reverse split** | Multiplied / divided by the ratio | Rescaled inversely to the ratio; an unadjusted series has a step discontinuity | Usually none; the *line* is the same line | The ratio, the effective date, and the pre/post quantity |
| **Stock / scrip dividend** | Increases | As above, pro rata to the distribution | None | Whether cash or stock was the default, and what was elected |
| **Rights issue** | Increases if subscribed | The theoretical ex-rights adjustment; the rights themselves may trade separately | May create a **temporary separate line** for the rights | The subscription ratio and price, the adjustment factor, and the fate of the rights line |
| **Spin-off** | May be restated by the parent adjustment | Parent series restated to reflect the distribution of value | **Creates a new instrument with no history** | The new instrument's identifiers and start date, and the parent's adjustment |
| **Merger / acquisition** | Replaced by consideration | The series *ends*; it does not continue | The identifier is retired; a new line may be issued with a **new ISIN** | The consideration structure, the effective date, and the identifier linkage (§3.4) |
| **Redenomination** | Usually unchanged | Rescaled by the redenomination ratio | Currency of the line changes | The ratio, the effective date, and the currency history |
| **Capital return / tender / buyback** | Decreases (cancelled shares) | Price adjusts by the capital returned | May be accompanied by a reorganisation | The terms, the acceptance ratio, and the effective date |
| **Name / domicile / listing change** | None | None | Descriptive attributes change; the identifier may or may not | The attribute's validity window — a name change with no dates corrupts every historical reference |
| **Delisting / insolvency** | Position ceases to exist | Series terminates; **the instrument disappears from current snapshots** | Identifiers may be retired or reused | That the instrument *existed on these dates* — the survivorship-bias record (§6.4) |

**The single sentence that ties this table to the rest of the guide.** Every row in it is an instruction to *edit the past* — and the difference between a firm that can edit the past correctly and one that cannot is whether the edit was recorded as an event with dates (§8.4) or applied as an overwrite to a current-state table (§13, anti-pattern three). The event classes are ordinary; the discipline is in the recording.

### 5.4 The Message Standards in Use, and the Migration Under Way

**The standards.** Corporate actions move between issuers, custodians, CSDs and holders over **ISO 15022** — the SWIFT MT securities message set, of which MT564 is the corporate action notification and MT566 the confirmation — and increasingly over **ISO 20022**, whose seev message family carries the equivalent content. Both the message flows and the migration milestones are verified in [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §4.4 ✅, including DTCC's use of the ISO 20022 corporate-action format and its conversion to the SWIFT SR2025 version dated 23 November 2025, and the end of SWIFT's MT-to-ISO-20022 coexistence for cross-border FI-to-FI payments on 22 November 2025. **Cite §4.4; do not re-derive it here.** What this guide adds is the data-model reading of those messages:

- **An announcement is a versioned statement, not a row.** The message set carries announcements, amendments and cancellations. A firm whose model has one row per event cannot represent the amendment, which means it cannot answer "which terms did we act on" — the exact question an audit asks.
- **One economic event arrives as many messages, from many parties.** The issuer's announcement, the custodian's notification and each intermediary's version are separate messages about one event. Deduplicating them into a single *event* with a single *firm's view of terms* is a modelling task that sits above the message parser; a firm that stores messages as its event table has no event table.
- **The message is not the event's terms authority for the firm's data.** A custodian's notification reflects the custodian's processing chain; a data vendor's event feed reflects the vendor's sourcing policy. Where they differ, the firm is in §5.5 and §5.7, not in a parser bug.

**The direction of travel** is unambiguous — corporate-action messaging is consolidating on ISO 20022, as [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §4.4 states and evidences ✅. The data-model consequence is that a firm's event schema should be *standard-neutral*: build the internal event model from the business content (type, terms, dates, versions, affected identifiers), not from one wire format's field layout, so that a format migration is a parser change rather than a schema change and a historical backfill.

### 5.5 Reconciling Corporate-Action Data — Across Vendors and Against Custodians

**Corporate-action data is the least reconcilable data in the reference-data estate, and it needs a three-way structure rather than a two-way one.**

| Comparison | What it detects | Owner | Boundary note |
|---|---|---|---|
| **Vendor A vs vendor B** (the event feed, field by field) | Coverage gaps (one vendor has no event), date disagreements, term-version disagreements, methodology differences in derived fields | Market-data management or security master | **This guide's territory** — a data comparison, no positions involved |
| **Vendor event record vs the firm's own instrument record** | That the event was captured but the identifier, quantity or terms were not updated | Security master | This guide's territory — the §5.1 post-event reconciliation |
| **Vendor event terms vs the custodian's notification** | Which version of the terms the firm's processing chain is acting on | Investment operations | Boundary with [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §4 — the custodian comparison belongs to the operational control |
| **Entitlement vs position on record date** | Missed or mis-dated entitlements | Investment operations | **[Investment Portfolio Operations](investment_portfolio_operations_guide.md) §5 — accounting/balance reconciliation, explicitly not this guide's cross-vendor data comparison** |

**The three-way structure is the point.** A firm comparing only its two vendors will find term disagreements it cannot adjudicate; a firm comparing its vendors against the custodian's notification finds that the custodian has an *operational* stake in the answer (it is computing an entitlement) and is therefore a different kind of witness from a data vendor. Neither comparison is sufficient. The practical design is: the vendors' feeds drive the *reference* record (what the instrument is), the custodian's message drives the *operational* record (what we are owed), and where they differ the firm applies the market's convention (usually, the depository's processing) while recording both — because the one thing a firm must never do is silently overwrite one with the other and lose the ability to explain a payment that did not arrive.

### 5.6 The Adjustment Question Is a Data-Model Problem

**Adjustment is a view over history; it is not a mutation of history.** This is the sentence to take away from the section, and it is where most firms' data models are structurally wrong rather than merely incomplete.

**The mechanics, stated exactly.** A corporate action changes the units and/or the price scale of an instrument. To compare a pre-event price with a post-event price — for a return, a chart, a backtest, a VaR input or a performance calculation — the historical series must be expressed on a *common* basis. The standard way to do that is to store the *raw* (as-traded) price and the *adjustment factors* derived from the event ledger, and to compute an adjusted series on demand: adjusted price for date *t* = raw price for *t* × (the cumulative factor for the events after *t*). The alternative — writing the adjusted value into the historical price row — destroys the raw observation, and with it the ability to answer three questions: what did the market actually trade at, what factor was applied, and what would the series look like under a different convention.

| Convention choice | What it produces | Where it is right | What it costs |
|---|---|---|---|
| **Unadjusted (raw)** | The price the market actually traded | Attribution of a *specific* trade; evaluation of an executed price against a historical print; anything requiring the as-traded value | Non-continuous series; naive returns across an event are meaningless |
| **Split/ratio-adjusted** | A continuous series for quantity-changing events | Almost universally used for equity research and backtesting | Requires the event ledger to be complete (§5.2) — a missed split silently doubles your returns |
| **Total-return adjusted** (cash distributions reinvested) | A series comparable to a total-return index | Performance measurement and long-horizon research | Embeds a reinvestment assumption the business must endorse |
| **Price-return adjusted** (cash distributions excluded from the series, dividends handled separately) | A series comparable to a price index | Risk measures where the distribution is modelled separately | Requires the distribution data to be complete and separate |

**Market practice here is genuinely divided, and the guide says so rather than picking a winner.** Whether cash dividends are adjusted into the price series for a given purpose depends on the purpose and on the firm's own convention; both conventions are defensible, and providers differ in what they apply by default. What is *not* defensible is applying one convention in the price store and another in the risk engine, or changing conventions without an effective date — because the adjustment convention is not a display setting, it changes every downstream number (§12.5).

**Three consequences that make this a data-model decision and not a calculation.** First, **the event ledger must be complete before any adjustment is meaningful** — which makes the corporate-action feed's completeness the precondition for the price series' correctness, and makes §5.2's completeness control a *price* control. Second, **the factor must be stored, not inferred**: a firm that computes adjustment factors on the fly from a current event table cannot reproduce a series as it was computed last year. Third, **the raw price must never be destroyed**, because the adjusted series is a derived view whose underlying observation is the only evidence of what the market did.

### 5.7 What a Control Cannot Settle

**The honest list.** A reconciliation control is powerful and bounded, and the bound arrives in these shapes:

- **Terms that are genuinely ambiguous.** Where an issuer's announcement permits more than one reading and the market has not converged, two providers can be simultaneously defensible. The control's output is a documented choice under change control (§10.5), not a resolution.
- **Amendments after the firm has acted.** The firm processed version 1 and the issuer published version 2; no reconciliation can un-act what happened. What it *can* do is prove the sequence — which requires the versioning of §5.1.
- **Voluntary-event outcomes.** Whether a tender is accepted, and at what ratio, is not knowable at announcement; the record must distinguish *announced* from *completed*.
- **Overlapping events.** Two actions with interleaved dates make the *order* of application material; if the providers disagree on order, the resulting factors differ even when the terms agree. The control must compare factors *and* order, which means the event ledger, not the price series, is the thing to reconcile.
- **Absence of an authoritative source.** For some instruments and markets there is no depository or registry to appeal to. Both vendors' answers are opinions; the firm's choice is a policy, and its only protection is having recorded that it made one.
- **Anything about a date the firm did not record.** No control reconstructs the past; §6.7 is the hard limit, and it applies here in its most concrete form: the adjustment applied to a series in March, once overwritten, cannot be recovered from the adjusted values alone.

---

## 6. Point-in-Time Correctness

### 6.1 The Two Axes and Their Standard Vocabulary

**A fact in a market-data system has two times, and a system that stores only one of them cannot answer questions about the past.** The pair of concepts is standard in temporal data management:

- **Valid time** (also called effective time, business time, or application time): the period **in the world** that the fact describes. A price for 15 April has `valid_from = 15 April`; a crosswalk mapping that was true until a merger has a `valid_to`.
- **Transaction time** (also called system time or knowledge time): the period during which **the database held** the fact. A price for 15 April that the firm loaded on 16 April and replaced on 20 June has a knowledge window that starts on 16 April and ends on 20 June.

**The standards anchor, stated honestly.** The SQL standard's temporal features are the closest thing to a standards-grade vocabulary: **SQL:2011** is where system-versioned tables (transaction time) and application-time period tables (valid time) were introduced, and the ISO/IEC 9075 family catalogue confirms the parts and their lifecycle — ISO/IEC 9075-1:2011 (the framework part) was **withdrawn**, revised by 9075-1:2016, with a further version 9075-1:2023 in force today (✅ verified at the ISO catalogue). **⚠ What this pass did not verify at source is the exact wording of the SQL:2011 feature names themselves**, so this guide uses the *concepts* — valid time and transaction time — as its working vocabulary and names SQL:2011 as the standards anchor for the feature family rather than quoting its clause text. Where a firm needs a contractual standard citation, that is the item to re-verify (§15). The temporal-database literature and the data-warehousing pattern language are the other recognised anchors: this repository's own treatment of slowly changing dimensions — SCD Types 0–7, with Type 2 as *add a new row and keep the full history* ([../technology/data/types_of_dimensions_data_warehousing.md](../technology/data/types_of_dimensions_data_warehousing.md) §3) — is the same idea in the Kimball idiom, and Type 2 is what an effective-dated crosswalk or a valid-time price record looks like in a warehouse.

**Why the vocabulary matters more than it looks.** Firms routinely call both axes "as-of," which makes the two requirements indistinguishable in a conversation and therefore in a design. "As-of 15 April" can mean *the price for 15 April* (valid time) or *the price we had on 15 April* (transaction time) — and those are different numbers whenever the vendor restated in between. §6.6 makes the disambiguation concrete.

### 6.2 Why the Latest Value Is Not the Truth for a Past Date

**The latest value is the truth for *today's question about the past*, under *today's* conventions. It is not what anyone knew at the time** — and it is routinely presented as though it were. Three mechanisms produce the divergence, and they are the three problems of this guide seen from the time axis:

| Mechanism | What changes between then and now | Which problem it is |
|---|---|---|
| **Restatement** | The provider replaced the value it previously published for that date | The data-is-not-fixed cause (§2.2), treated at §6.3 |
| **Retroactive adjustment** | A corporate action has been applied to the historical series, changing values for dates *before* the event was known | The instrument-is-not-fixed cause (§2.3), treated at §5.6 |
| **Coverage and survival** | The universe itself differs: instruments that existed then are absent from today's snapshot, and instruments covered today were not covered then | The identity and global-market causes (§2.1, §2.4), treated at §6.4 |

**The practical test a firm can run on itself.** Take a report produced six months ago, and try to reproduce it exactly from today's data. If the numbers come out the same, either the data genuinely did not change or — far more likely — the firm never recorded enough to notice that it did. A reproduction that succeeds *silently* is not evidence of stability; it is evidence of a missing knowledge-time record.

### 6.3 Silent Restatement

**Silent restatement is the replacement of a published historical value with no change flag, no notification and no record on the firm's side.** It is the most damaging of the three mechanisms because it is the least visible: nothing errors, nothing is missing, the run completes, and the number quietly changes.

**How it presents, in the order a team usually encounters it.**

1. **The cached-versus-refreshed discrepancy** — a quant team that cached a history extract last quarter and a risk team that pulled fresh data this quarter disagree, and neither can prove the sequence.
2. **The unexplained report-to-report movement** — a month-end number moves more than the market can explain, because the *inputs* moved retroactively.
3. **The irreproducible backtest** — a model that reproduced its published results in April does not in September, with no code change. (The reproducibility discipline itself is [Deterministic Engineering](../technology/deterministic_engineering_guide.md) — cross-reference; the market-data cause is this section.)
4. **The audit finding** — a regulator asks what the firm reported in March and what it knew in March, and the firm can answer the first question and not the second.

**Detection requires the knowledge axis, and nothing else works.** Retain each load as a version, diff each new load against its predecessor by instrument and field, and raise a *data change notice* for differences on **historical** dates — that is, differences whose valid date is in the past. A difference on today's value is ordinary market movement; a difference on a date 40 days ago is a restatement, and it deserves a record, an impact assessment and — where material and the firm has already reported on it — a decision. Three pragmatic points: the diff is cheap (it is a comparison of two stored versions, §4.6); the *signal* is the valid-date-age, not the size of the change (large legitimate corrections and small material ones both occur); and the output is not a queue to be cleared but a **ledger** — a permanent record of what changed, when the firm learned it, and what was done.

**What a restatement means for work already done.** A report filed, a risk number sent to the board, a model validated: all of these were accurate as of the knowledge they used. The firm's obligation is not to un-file them but to be able to *explain* the movement — which requires the knowledge-time record and nothing less. This is also the point where the regulatory frame bites: the FRTB vendor-provenance requirements (§4.6, §11.1) exist precisely so that a bank can interrogate the *count and dates* of the price observations behind a risk factor, which is a restatement question asked by a supervisor.

### 6.4 Survivorship Bias

**A current snapshot of the instrument universe is a list of survivors.** Instruments that merged, delisted, failed, or were acquired are absent or reduced to a stub — and any analysis built on today's universe silently excludes them. The bias is not primarily a data-quality defect; it is a *selection* effect, and it is introduced by the choice of universe, not by any individual value being wrong.

**Why it is so easy to create accidentally.** The most common construction in a bank is the one that produces it: take today's security master, join the price history for those instruments over the last ten years, run the analysis. Every instrument that did not survive the period is excluded from *the whole history*, including the years when it was alive and its performance was contributing to the portfolio's actual outcome. The resulting series is systematically flattering in exactly the situations where accuracy matters most — stressed periods, sector rotations, credit events.

**What a point-in-time universe requires, concretely.**

- **Dated membership, not current membership.** For an index or a benchmark, the constituent list *as of each rebalance date*; for the firm's own book, the instruments held *as of each date*. Both are valid-time records, and both are the same pattern as the effective-dated crosswalk (§3.3).
- **Listing and delisting dates on the instrument record.** The record must say when an instrument came into existence and when it ceased — and must retain the record after it ceases (the anti-pattern is deleting the delisted instrument, §13).
- **A termination value, not a hole.** An instrument that delisted at a price, was acquired for consideration, or was written down must carry the terminal value and the terminal date, so that a history can be closed rather than truncated.
- **Identifier linkage across the death of a line** (§3.4, §5.3), so that "the position in instrument X ended on 3 March and became a position in instrument Y" is representable.

**The reconciliation connection, stated once.** Survivorship bias is a *universe* defect, so per-field reconciliation will never detect it: there is no counterpart record to disagree with, because the instrument is simply not in the file. The detector is a **completeness control** — the count of instruments with a price on a given historical date, compared against a source that has no survivorship filter (an exchange's own listing history, a depository's records, or the vendor's *historical* universe file where one is licensed). This is the case that justifies the coverage break class of §4.3 having its own severity rather than being treated as an empty-diff result.

### 6.5 Look-Ahead Bias in Backtests and Model Development

**Look-ahead bias is using a value that was not knowable at the moment the model pretends to decide.** In market data it arrives through three doors, all of them data-model failures rather than modelling errors:

- **The adjustment door.** Because corporate-action adjustment is computed with hindsight (§5.6), an adjusted price series *knows about the future*: the factor applied to a 2019 price embeds an event announced in 2021. Any model trained on adjusted data has, formally, been told something about the future. The magnitude varies by strategy and by instrument; the *existence* of the contamination is structural and should be stated in model documentation rather than assumed away.
- **The restatement door.** A model that trains on today's history is training on values that, in many cases, were not the values available on the date they are attributed to (§6.3). Fundamental and estimate data are worse in this respect than prices, because revisions are normal in those datasets.
- **The universe door.** A model trained on today's constituents (§6.4) selected itself a sample of survivors.

**Why this is a data-management problem rather than a quant problem.** The quant team cannot fix it; it can only *report* whether its inputs were point-in-time. The data platform either has the knowledge-time record and the point-in-time universe, or it does not, and if it does not, the honest position is that every backtest is an approximation with an unquantified, non-zero bias — which is a statement a firm can put in writing, manage, and progressively reduce by starting to record. That last point is §6.7, and it is the reason this guide treats point-in-time not as an analytics nicety but as the storage decision the whole guide builds toward.

**The reproducibility link.** Reproducing a backtest requires the inputs to be *addressable*, which means the code needs to be able to ask for a dataset as of a knowledge date and get the same bytes. That is the discipline of [Deterministic Engineering](../technology/deterministic_engineering_guide.md) applied to market data, and the platform mechanics of snapshotting and time-travel over immutable versions are covered in [../technology/data/data_pipeline_versioning.md](../technology/data/data_pipeline_versioning.md) and [../technology/data/delta_lake_vs_iceberg.md](../technology/data/delta_lake_vs_iceberg.md) — cross-reference for the mechanics; the requirement this guide asserts is simply that the *market-data* layer must expose a knowledge-dated read.

### 6.6 As-Of Queries and the Vintage

**An as-of query that specifies one axis is ambiguous and will eventually be wrong.** The disambiguated form is a pair: *render the value that was valid at date V, as known at knowledge date K.* In practice three query shapes cover almost everything:

| Query shape | Meaning | What it needs | Typical use |
|---|---|---|---|
| **"The value for date V, as we know it today"** | Latest knowledge, past valid time | Valid-time record only | Mark-to-market, current analytics, charts |
| **"The value for date V, as we knew it on date K"** | Restated-excluding; the historical view | Both axes | Backtests, model validation, reproducibility |
| **"The vintage as of K"** | The whole dataset as known on K | Both axes, plus a whole-dataset version identity | Regulatory reconstruction, dispute investigation, post-incident review |

**The vintage is the second and third shapes generalised to a dataset: the set of values a consumer would have seen had they asked on the knowledge date.** Firms usually acquire the capability in one of two ways. The lighter route is a **daily snapshot of the golden copy**, published as an immutable, dated version — cheap, blunt, and effective for the "what did we publish in March" question. The heavier, more precise route is a **bitemporal store** (§8.2) in which every record carries both axes and the vintage is a predicate rather than a copy. Both are legitimate; the choice is a cost and latency trade-off, and the important thing is that *one* of them exists, because without either the third query shape is unanswerable no matter how good the engineering is.

**A word on the lakehouse mechanics, by reference rather than by assertion.** Modern table formats provide snapshot and time-travel semantics that make retained versions queryable without a full physical copy; the comparison and the operational caveats are the subject of [../technology/data/delta_lake_vs_iceberg.md](../technology/data/delta_lake_vs_iceberg.md), and versioning strategy over pipelines is [../technology/data/data_pipeline_versioning.md](../technology/data/data_pipeline_versioning.md), with retention and backfill practice in [../technology/data/backfill_data_engineering.md](../technology/data/backfill_data_engineering.md). This guide does not evaluate those products; it states the requirement — a knowledge-dated read — and routes the reader to the mechanism.

### 6.7 The Hard Limit, Stated Plainly

**History you did not record cannot be reconstructed.** Not by better engineering, not by a smarter schema, not by a vendor's historical file, and not by an inference from the values that remain. If the firm overwrote March's crosswalk, it does not have March's crosswalk; if it discarded the vendor file it loaded on 3 April, it cannot prove what the vendor said on 3 April; if it never recorded a knowledge date, it cannot distinguish a restatement from a stable history. This is the guide's most important sentence after the thesis, and it is deliberately placed early in the reader's path through the material rather than buried in a closing caveat.

**Four corollaries that make the limit actionable.**

- **It is a storage decision, not a technical one.** The remedy is to begin recording: retain the raw vendor answers with receipt timestamps (§4.6), effective-date the crosswalk (§3.3), carry both axes on the golden record (§8.2), and snapshot or time-travel the result (§6.6). Each is a decision with a cost, an owner, and a date — and none of them is hard engineering.
- **The cost is storage and process discipline; the cost of *not* doing it is unbounded and arrives as a finding.** Retention has a budget line and a number; the inability to explain a number the firm published has no ceiling, no schedule, and no good moment.
- **Vendor historical datasets are a partial substitute and inherit the vendor's conventions.** Buying history means buying *that provider's* adjustment convention, restatement behaviour and coverage decisions — which is why the FRTB vendor requirements (counts and dates of real price observations, auditable pricing information; MAR31.14, ✅ verified at source) are framed as things a bank must be able to *interrogate*, not merely receive.
- **The clock starts when you start.** A firm that begins recording today has a complete, auditable knowledge history from today, and an acknowledged gap behind it — which is a vastly better position than a firm that has a five-year archive whose knowledge dates are unknown. The first year's record is worth more than the previous five years' reconstruction.

**And the limit has a quiet corollary that this guide uses to close.** Every mechanism in §8 exists to make the *next* restatement, the *next* corporate action and the *next* identifier change a recorded event rather than a silent loss of the past. That is what integrity engineering is: not omniscience about history, but custody of the record of what was known.

---

## 7. The Three Problems Are One Problem

**Identity that changes, sources that disagree, time that erodes — and the same design decision sits under all three.** Sections 3 to 6 took them apart because they are worked on by different teams with different tools. This section puts them back together, because in production they never arrive separately.

**How the problems generate each other.** The chain runs in one direction, and the direction is what makes the ordering of this guide's sections a design statement rather than a table of contents.

| Step | What happens | Why it is not separable |
|---|---|---|
| **1. Identity is contested** (§3) | The firm must pick an anchor identifier and maintain a crosswalk | The anchor determines every join the firm will ever be able to make, including the joins its reconciliation will run on |
| **2. The instrument changes state** (§5) | A corporate action edits the past: identifiers, quantities, price scales, terms | The crosswalk needs effective dates *because* of this — the mapping changed when the action took effect; and the two vendors start to disagree *because* they are at different states of applying the same event |
| **3. The sources disagree** (§4) | Reconciliation finds breaks, most of them explicable, some of them not | Whether a break is explicable depends on the corporate-action state (step 2) and on whether the two records are even about the same instrument (step 1) — so a reconciliation built without the first two steps produces breaks nobody can classify |
| **4. Time erodes the record** (§6) | Values are restated, universes change, adjustments are retroactive | The only way to keep steps 1–3 answerable about the past is to *record* the knowledge axis — which is a decision taken at step 1, not a feature added at step 4 |

**How a decision for one problem constrains the others, stated as four hard couplings.**

- **The identifier choice constrains the join.** Anchoring on a vendor code makes the firm's history dependent on that vendor's code stability and on that vendor's continued commercial relationship; anchoring on ISIN makes venue-level attribution require a MIC and makes OTC derivatives require a separate path (§3.5). This is decided once and inherited by the reconciliation design, the corporate-action linkage and the historical store.
- **The corporate-action convention constrains the history.** The adjustment convention (§5.6) determines what the stored price series *means*, and therefore what any model trained on it learned. A firm that changes its convention without an effective date has made its back history non-reproducible by construction, no matter how good its storage is.
- **The storage decision constrains what can be reconciled.** No retention, no reconciliation (§4.6); no knowledge axis, no restatement detection (§6.3); no point-in-time universe, no survivorship control (§6.4). Every one of these is decided by how the first table was designed, not by a later project.
- **The tolerance decision constrains what a break means.** A break tolerance agreed per field type (§4.3) is what converts a difference into an item with an owner; without it, the reconciliation produces numbers, and numbers are not a control (§4.2).

**The single design question underneath.** Strip the three problems of their vocabulary and each one resolves to the same question: **for this fact about this instrument, what was known, by whom, and when — and is that recorded?** The instrument's identity is "what was known about which instrument"; the cross-vendor break is "what was known by two parties, and when each knew it"; the point-in-time question is "what was known at the time, as opposed to what is known now." The engineering answer to all three is one pattern: **record the fact with both its times, its source, and its identity — and never overwrite.**

**What this means for the practitioner's next step.** It means a firm should not sequence these as three projects. The cheap, high-leverage first move is deliberately unglamorous: begin retaining the vendor answers with receipt timestamps (§4.6), and begin effective-dating the crosswalk (§3.3). Those two changes cost almost nothing, make everything else possible later, and — unlike a reconciliation dashboard, a new vendor or a data-quality programme — they get *more* valuable the longer they have been running.

---

## 8. The Data Model

**The engineering core: five structures, each doing one job, together making the three problems answerable.** The sketches below are deliberately minimal — the point is the shape, not the product. They are written as plain table definitions with the columns that carry the semantics, because the semantics is what gets omitted.

### 8.1 The Effective-Dated Identifier Crosswalk

**Job: answer "what identifiers refer to the same instrument, and as of when did that become true?"** The full design rationale is §3.3; the structure is repeated here so the model reads as a whole.

```
-- the anchor: one row per instrument lineage, not per identifier
instrument (
  instrument_sk          -- surrogate key, stable forever
  instrument_lineage_id  -- groups the states of one economic instrument across corporate actions
  asset_class
  cfi_code               -- ISO 10962 classification (§3.2)
  created_knowledge_ts
)

-- every identifier ever attached to an instrument, effective-dated
instrument_identifier (
  instrument_sk
  id_type                -- 'ISIN' | 'CUSIP' | 'SEDOL' | 'RIC' | 'FIGI' | 'MIC' | 'UPI' | 'UTI'
  id_value
  valid_from             -- business/valid time: when this identifier began denoting this instrument
  valid_to               -- NULL = still current
  knowledge_from         -- transaction time: when we first held this assertion
  knowledge_to           -- when we stopped holding it (a change of belief closes the row)
  source_id              -- FK to data_source (vendor/product/licence)
  confidence             -- 'vendor_asserted' | 'two_vendor_confirmed' | 'overridden'
  override_id            -- FK when confidence = 'overridden' (§8.5)
  PRIMARY KEY (instrument_sk, id_type, id_value, knowledge_from)
)
```

Three properties matter. **The lineage key** (`instrument_lineage_id`) is what makes "the instrument that became this instrument" representable without pretending the identifiers are equal (§3.4, §5.3). **The four dates** are the entire point: valid time answers business questions, knowledge time answers audit questions (§6.1). **The natural key includes `knowledge_from`**, so a change of belief is an insert — and the table can never be "refreshed" into amnesia.

### 8.2 The Bitemporal Record — Valid Time and Knowledge Time

**Job: hold the firm's asserted values, with both times and per-field provenance, without ever destroying a prior belief.** This is the golden record's shape; the *content* is one row per instrument per field per validity window, which is verbose by design — the verbosity is the audit trail.

```
-- the golden record, bitemporal and field-attributed
golden_instrument_field (
  instrument_sk
  field_code             -- 'PRICE_CLOSE' | 'SHARES_OUTSTANDING' | 'NAME' | 'COUPON' | ...
  currency               -- where applicable, part of the fact, not an afterthought
  valid_from             -- valid time window this value describes
  valid_to               -- NULL = open-ended
  knowledge_from         -- when this value became the firm's held belief
  knowledge_to           -- when it ceased to be held (closed by a later row)
  value_num              -- numeric facts
  value_txt              -- text facts (exactly one of value_num/value_txt is set)
  source_id              -- who asserted this value (the field-level provenance, §4.5)
  vendor_as_of_date      -- the vendor's own as-of date, which may differ from valid_from
  received_at            -- when the firm received the vendor's assertion
  quality_flag           -- 'validated' | 'within_tolerance' | 'break_open' | 'override'
  override_id
  PRIMARY KEY (instrument_sk, field_code, currency, valid_from, knowledge_from)
)
```

**The read patterns this shape supports, stated as the queries the business actually asks.**

| Business question | Predicate | Axis used |
|---|---|---|
| "Mark to market today" | `valid_to IS NULL AND knowledge_to IS NULL` (current belief, current validity) | neither axis in the past |
| "What was the closing price for 15 April?" | `valid_from <= '…-04-15' < valid_to AND knowledge_to IS NULL` | valid time |
| "What price did we use on 20 April for 15 April?" | `valid_from <= '…-04-15' < valid_to AND knowledge_from <= '…-04-20' < knowledge_to` | both |
| "What changed since 1 April, for historical dates?" | diff on `knowledge_from >= '…-04-01'` where `valid_to < knowledge_from` | both — the restatement detector (§6.3) |

**A performance note that is not an aside.** Bitemporal field-level storage multiplies rows, and the honest answer is that it is unnecessary for every field: **price and valuation fields, corporate-action fields and identifier fields** carry the highest restatement and audit value and deserve the full treatment; a descriptive field that has never changed in twenty years is fine in a simpler table. The choice should be recorded as a documented scope decision (the §4.2 "declared scope" property), not left to whoever builds the table.

### 8.3 The Immutable Vendor-Raw Store

**Job: preserve the evidence.** Requirements are §4.6; the shape is a landing zone that nothing rewrites.

```
vendor_raw_delivery (
  delivery_id            -- surrogate
  source_id              -- which vendor, which product, which licence class
  file_or_message_id     -- the vendor's own identity for the delivery
  content_checksum       -- detect tampering and de-duplicate re-deliveries
  received_at            -- server-side, immutable
  vendor_as_of_date      -- the vendor's stated as-of for the file's content
  vendor_extract_run_id  -- many vendors version their extract runs; keep it
  storage_uri            -- pointer to the immutable object (object store / retained volume)
  processing_status      -- 'landed' | 'parsed' | 'failed' — status, never content
)
```

**Two rules.** The store is **append-only**: a re-delivered file is a new `delivery_id`, never a replacement — because a re-delivery that differs is itself the restatement evidence (§6.3). And the store is **content-addressed where possible** (checksum-addressed objects), which makes retention cheap, de-duplication automatic, and the "did the vendor actually change the value?" question a one-line comparison rather than an investigation. The downstream mechanics of parsing, versioning and backfill belong to [../technology/data/data_pipeline_versioning.md](../technology/data/data_pipeline_versioning.md) and [../technology/data/backfill_data_engineering.md](../technology/data/backfill_data_engineering.md); the object-storage and retention considerations to [../technology/data/cloud_object_storage_lakehouse_guide.md](../technology/data/cloud_object_storage_lakehouse_guide.md) — cross-reference, do not re-derive.

### 8.4 The Corporate-Action Ledger That Drives Adjustment

**Job: make every adjustment traceable to a recorded event, and every event a versioned, dated fact.** The ledger is the *driver*; the adjusted price series is a derived artefact. Firms that invert this — adjusting prices in the price pipeline and keeping no ledger — cannot reproduce a series, cannot answer which factors were applied, and cannot reconcile their events against a vendor's (§5.6).

```
corporate_action_event (
  event_id                 -- one row per economic event, deduplicated across many messages (§5.4)
  instrument_lineage_id    -- the instrument or lineage affected
  event_type_code          -- 'CASH_DIV' | 'SPLIT' | 'MERGER' | 'RIGHTS' | 'SPINOFF' | ...
  mandatory_flag           -- mandatory | mandatory_with_options | voluntary
  announcement_ts          -- when first announced (valid time, not receipt)
  announced_by             -- issuer | depository | vendor_feeds
  first_knowledge_ts       -- when the firm learned of it (transaction time)
  source_id                -- the vendor/custodian feed that carried it
  status                   -- 'announced' | 'confirmed' | 'completed' | 'cancelled' | 'lapsed'
  PRIMARY KEY (event_id)
)

corporate_action_version (
  event_id
  version_no               -- announcements get amended; versions are the audit trail (§5.1)
  knowledge_from           -- when this version became the firm's belief
  knowledge_to
  ex_date                  -- valid time
  record_date
  payment_date
  terms_json               -- the terms as stated, in a standard-neutral shape (§5.4)
  source_id
  PRIMARY KEY (event_id, version_no, knowledge_from)
)

corporate_action_effect (
  event_id
  version_no
  instrument_sk_before
  instrument_sk_after      -- may equal _before for quantity-only events (a split)
  effective_date           -- valid time of the state change
  quantity_factor          -- multiply pre-event quantity by this
  price_factor             -- multiply pre-event price by this to place it on the post basis
  factor_basis             -- 'RATIO' | 'THEORETICAL_EX' | 'TOTAL_RETURN' | 'PRICE_RETURN'
  PRIMARY KEY (event_id, version_no, instrument_sk_before)
)
```

**Why the factors live in the ledger and not in the price table.** Three reasons, all of them §5.6 restated as schema: the raw price survives (so an adjusted series can be *recomputed* under a different convention without losing what the market traded); the factor is *dated and attributable* (so "what adjustment did we apply to this series in March" has an answer); and the convention is *explicit* (`factor_basis`) rather than implicit in a pipeline's behaviour — which is what lets a firm change convention under change control (§10.5) instead of discovering the change in a model's output.

### 8.5 The Minimum Viable Schema for a Small Team

**The thesis of this subsection: a five-person team cannot build an enterprise security master, and does not need to.** What it needs is the minimum set of structures that makes the three problems answerable, deployed in priority order, with the cheap and durable parts first. The table below is the priority order, not a wish list.

| Priority | Structure | Why it goes first | Effort profile |
|---|---|---|---|
| **1** | **Immutable vendor-raw store** (§8.3) | Nothing else can be retrofitted if the evidence is thrown away; it is the only structure whose *late* adoption has an unrecoverable cost | Low build; storage cost; process discipline |
| **2** | **Effective-dated identifier crosswalk** (§8.1) | Every join, every reconciliation and every historical question depends on it; it is small and can be maintained by a named owner | Low–medium build; ongoing stewardship |
| **3** | **Corporate-action ledger with versions and effects** (§8.4) | Makes adjustment reproducible and makes the event feeds reconcilable; without it the price series is unfalsifiable | Medium build; depends on feed quality |
| **4** | **Bitemporal golden record for the high-value fields** (§8.2) | The audit and restatement answers; scope it deliberately, not universally | Medium–high build; the storage bill is real |
| **5** | **Break queue, tolerance policy and override register** (§4.3, §4.4, §10.2) | The operating layer that turns structures 1–4 into a control | Mostly process; tooling can start as a table |

**A deliberately small end-state.** One database, these tables (plus reference tables for `data_source`, `tolerance_policy`, `break`, `override_record`, `data_change_notice`), one nightly cycle that lands raw, updates identifiers and events, recomputes the golden view, runs the reconciliation and posts breaks to the queue. That is achievable by a small team, it is auditable, and — critically — **it degrades gracefully**: if the team never gets to the bitemporal store, structures 1–3 still answer the questions that matter most, and the vendor-raw store means the bitemporal history can be *derived retrospectively from the retained files*. That last property is the strongest argument for building the raw store first, whatever the roadmap says.

**What to buy instead of build, and why, is §9.5** — the honest trade-off is that the *structures* above are cheap and specific to your firm, while a commercial security master is expensive and general, and the two are complements in a specific order.

---

## 9. The Architecture and the Flow

### 9.1 The Five Layers

**The flow, drawn as layers, with the control that lives in each.** The value chain venue → feed → vendor → platform → decision is [Market Data Consumption](market_data_consumption_guide.md)'s territory (its §1 and §3); this is the *integrity pipeline* that sits inside the firm's boundary.

| Layer | What it does | The integrity property it must have |
|---|---|---|
| **1. Ingestion** | Receives vendor deliveries and messages (prices, statics, corporate actions, entity data) | **Nothing is transformed here.** Raw is landed with receipt timestamps and checksums (§8.3); parsing is a downstream, reproducible step |
| **2. Normalisation** | Maps each source's fields to the firm's standard-neutral model; applies the crosswalk; converts codes to the firm's vocabulary | **Mapping is versioned and effective-dated.** The mapping is a first-class artefact (§3.3, §9.3), not code |
| **3. Reconciliation** | Runs the comparisons of §4 across sources, and the event/price arithmetic ties of §4.5 | **Every break is retained with its inputs** so it can be re-investigated as of its raise date |
| **4. Golden publication** | Publishes the firm's asserted record — with both time axes, per-field provenance and an immutable version identity | **Publication is versioned and immutable**; the published version is the firm's position for that date (§6.6) |
| **5. Distribution** | Feeds consumers (risk, finance, quant, trading, reporting) | **Consumption is entitled and attributable** — which consumer got which version at which time (the entitlement layer is [Market Data Consumption](market_data_consumption_guide.md) §9.2) |

**Two properties of the layering that make it a design rather than a diagram.** First, **each layer is independently re-runnable from the one above it** — which is what makes a restatement a *recomputation* rather than a crisis, and what makes backfills safe ([../technology/data/backfill_data_engineering.md](../technology/data/backfill_data_engineering.md)). Second, **the boundary between layer 1 and layer 2 is the retention boundary** — everything above the line is evidence, everything below is opinion, and a firm that blurs the two loses the ability to say what it was told (§4.6).

### 9.2 Where Each Control Lives

| Control | Type | Layer | What it catches | What it cannot catch |
|---|---|---|---|---|
| Schema and plausibility validation | Preventative | 2 | Structurally impossible records (bad dates, unknown codes, negative prices) | A plausible but wrong value |
| Effective-dated crosswalk enforcement | Preventative | 2 | Joins on an undated or ambiguous mapping; ambiguous matches across a corporate-action boundary | A mapping that is dated but wrong |
| Corporate-action completeness check | Preventative/detective | 2–3 | A mandatory event present at one source and absent at another (§5.2) | An event no source carried |
| Cross-vendor reconciliation | Detective | 3 | Disagreements, coverage gaps, staleness | A shared error in both vendors |
| Arithmetic ties (price × quantity, factor vs event) | Detective | 3–4 | The Franken-record inconsistency that per-field tolerances miss (§4.5) | An error consistent across the tied fields |
| Restatement detection (knowledge-time diff) | Detective | 4 | Silent restatement of historical values (§6.3) | A restatement the firm never re-loaded |
| Universe completeness (historical counts) | Detective | 4 | Survivorship bias in the published universe (§6.4) | Missing instruments that no source ever carried |
| Publication versioning | Preventative | 4 | The loss of "what we published" | — |

**The pattern to notice: there is no control on that list that catches a shared error in both vendors.** That is not a gap in the design; it is a property of two-source reconciliation, and it is the reason the alternatives matter — a third source, an exchange or depository record, the issuer's own filing, or (for capital purposes) the audit-of-pricing-information expectation that MAR31.14 places on the vendor itself (✅ verified at source). A firm that believes two vendors agreeing proves correctness has confused agreement with truth.

### 9.3 The Data Contract at Each Interface

**Every layer boundary is a contract, and every contract needs the same five things.** The general treatment of data contracts and interface governance is the repository's data-platform material — [../technology/data/data_governance_framework.md](../technology/data/data_governance_framework.md), [../technology/data/enterprise_data_platforms_guide.md](../technology/data/enterprise_data_platforms_guide.md) and [../technology/data/data_fabric_guide.md](../technology/data/data_fabric_guide.md) — and this guide does not re-derive it. What is specific to market and reference data is *what the contract must contain*:

| Contract element | The market-data-specific requirement |
|---|---|
| **Identity** | Which instrument, by which identifier and at which granularity level (§3.2) — including the FIGI-level trap and the MIC-for-venue rule |
| **Field semantics** | The definitional clarity of §4.1's semantic row: what "coupon," "issue date" and "quantity" mean *at this interface*, written down once (the glossary, enforced at the boundary) |
| **Time semantics** | Which of the two axes each timestamp is, and what the vendor's `as_of` means relative to receipt (§8.2) |
| **Quality expectations** | The tolerance class applicable (§4.3) and the expected completeness — because a contract with no tolerance is a contract that cannot fail |
| **Change protocol** | Notice periods for schema, code-list and coverage changes — the mechanism that makes §10.5's change control possible |

**The one contract that most firms omit, and the one that matters most internally: the contract between the data platform and the model-development function.** It is where point-in-time expectations must be stated explicitly — *this dataset is point-in-time as of knowledge date K; this dataset is restated-current* — because a model team that assumes point-in-time when it is not gets an unquantified look-ahead bias (§6.5) and no error message.

### 9.4 The Lineage Requirement

**Lineage is not a diagram; it is the ability to answer a question under time pressure.** The regulatory weight of lineage is BCBS 239's, and its treatment belongs to [Risk Data Aggregation](risk_data_aggregation_guide.md) and [../technology/data/data_lineage_tools.md](../technology/data/data_lineage_tools.md) — cross-reference. The market-data-specific requirement adds one dimension the general lineage discussion does not carry: **lineage across restatements.**

A conventional lineage graph answers "where did this number come from?" by traversing transformations. For market data the question that actually arises is: *the number we published for 15 April was X, and the number we would publish now is Y — show me why.* Answering it requires lineage *through time*: which vendor delivery, which crosswalk version, which corporate-action version, which tolerance decision, which override. Three practical consequences:

- **Lineage must include the identifier mapping and the event versions**, not just the code (`etl_job_7 → table_z`). A pipeline-level graph that stops at the table cannot explain a corporate-action-driven change.
- **Column-level lineage is the bar** for any field whose value is published externally, which is the same conclusion the repository records for BCBS 239 purposes ([Risk Data Aggregation](risk_data_aggregation_guide.md) §11, and its cross-reference to the data governance treatment).
- **Lineage must survive the retirement of an instrument.** Delisted and merged lines are precisely the ones whose numbers get questioned years later (§6.4), so lineage cannot be pruned with the instrument.

### 9.5 Build vs Buy for a Security Master

**The honest trade-off, stated as a decision rule rather than a preference.** A commercial security master brings coverage, vendor relationships, corporate-action processing at scale, and a support organisation; it also brings cost, a data model you must accept, an integration project, and a set of conventions you inherit rather than choose. A built security master brings exactly the semantics this guide has described — your anchor identifier, your effective dates, your adjustment convention, your tolerance policy — and brings with it an ongoing stewardship obligation that firms consistently underestimate.

| Dimension | Buy | Build |
|---|---|---|
| **Time to first value** | Fast for coverage and vendor connectivity | Slow — the identifier and event model takes real design work |
| **Coverage and corporate-action breadth** | The vendor's coverage is the product; global breadth is hard to match | Only as broad as your feeds and your team's capacity |
| **Semantic fit** | You adapt your conventions to the product's model | The model is exactly your semantics — anchor, axes, conventions, tolerances |
| **Cost profile** | Licence plus implementation plus annual uplift | Build plus permanent stewardship; the stewardship is the part that gets cut |
| **Evidence and reversibility** | The retailer model is the vendor's; extracting your own history later is a project | Your raw store and lineage are yours from day one; the *unrecoverable* asset is yours |
| **Where it fails** | When the product cannot express your adjustment convention, your entity levels, or your effective-dated lineage | When the team that built it leaves and the conventions are undocumented |

**The decision rule that the rest of this guide implies.** Buy the *coverage* and the *connectivity*; build the *evidence* and the *conventions*. Concretely, and in this order: build the immutable vendor-raw store (§8.3), build the effective-dated crosswalk (§8.1) and the corporate-action ledger (§8.4) even if a vendor supplies the same content, and buy the breadth of instrument and event coverage you cannot economically assemble. The reason is asymmetric and it is the whole argument of §6.7: if a purchased security master is later replaced, the firm can replace it *if it retained its own evidence and its own dated history* — and cannot recover the years in which it did not. **The structures that are cheap to build and impossible to rebuild later must never be outsourced**, whatever the roadmap pressure says.

---

## 10. The Operating Model

**Integrity structures that nobody operates are documentation, not controls.** This section is the part of the guide that determines whether §8's tables matter.

### 10.1 The Daily and Periodic Cycles

| Cadence | What runs | The check that proves it ran |
|---|---|---|
| **Intraday / continuous** | Ingestion of streaming and delivered vendor content into the raw store; validation on ingest | Delivery counts and checksums; ingest exception log |
| **Daily (end-of-day)** | Crosswalk updates from identifier and event feeds; corporate-action capture; golden record recomputation with `knowledge_from = today`; the cross-vendor reconciliation run; the restatement diff; publication of the day's immutable golden version; break posting | The day's run log with input versions; the published version identifier; the day's break count by class |
| **Weekly** | Break-queue review by class, with aging and owner follow-up; vendor query status | The review's minutes and the queue's age distribution |
| **Monthly** | Tolerance-policy review against actual break populations; coverage and universe-completeness counts; corporate-action completeness sampling; the reconciliation result reported to a named forum | The forum's pack — and the questions it asks |
| **Quarterly / annual** | Change control over conventions and tolerance policy (§10.5); retention-period review; evidence of the audit trail's completeness (§10.6); re-validation of vendor SLA and coverage | The signed change records and the retention decision's review date |

**The design principle behind the cadence.** Everything that must be *decided* — tolerances, conventions, retention — is on a periodic cycle with a named forum; everything that must be *executed* — landing, computing, comparing — is daily and automated. A firm that decides conventions daily (in someone's head, per break) has no conventions; a firm that only executes periodically has stale data.

### 10.2 The Break Queue and Its Ownership

**Ownership is the control.** A break has exactly one accountable owner — the owner of the *field* (the data steward) or the owner of the *source* (the vendor relationship), per a written RACI — and the queue's value comes from the fact that names are attached to items with ages.

| Break class | Typical owner | Why that owner |
|---|---|---|
| Identifier / crosswalk | Security-master data steward | They own the mapping and the lifecycle rules (§3.4) |
| Price and valuation | Market-data operations, with risk/finance as the tolerance authority | Operations can investigate the source; risk/finance own the acceptance threshold (§4.3) |
| Static attribute | Security-master data steward | Field-level authority and glossary |
| Corporate-action field | Market-data operations, escalating to investment operations where entitlement is affected | The data fix is a feed issue; the consequence is an operational one (§5.5) |
| Coverage / missing instrument | Market-data management plus the vendor relationship owner | Usually a licensing or coverage gap, not a data error |
| Methodology difference | Risk/finance (policy owner) | Not fixable by data work; needs a documented decision |

**The queue's health metrics, in the order they should be reported.** Age distribution by class (not total count), the proportion of breaks closed with a recorded root cause, the proportion closed as "within tolerance after reclassification," and the number of overrides outstanding with an expiry date. A suite of metrics that reports only the *total* break count will be managed by suppressing the total.

### 10.3 Escalation

**Escalation needs three things to exist before the break does: a threshold, a forum, and a prior agreement.** The threshold is expressed in both dimensions that matter — **age** (how long it has been open) and **materiality** (how many instruments, how much value, which reports it touches). The forum is named (a data governance or risk-data committee with market-data on its agenda, not a project meeting). And the agreement is that the forum *owns the decision* when the break cannot be resolved by the data team, which is the only way a methodology disagreement ever gets closed: a decision-making body, not a better query.

**The escalation ladder that works in practice:** owner → team lead within the SLA bucket → data governance forum at the materiality threshold → vendor account escalation (where the vendor is the cause) → and, where the exposure is reportable or the break affects a filed return, into the reporting or risk function's own escalation, because at that point it is not a data issue any more.

### 10.4 The Vendor-Query Process

**When the vendor is the source of the discrepancy, the firm needs a process that produces evidence, not correspondence.** The distinction matters: a query that asks "why is your price different?" gets a polite restatement of the methodology; a query that says "on 15 April, for instrument X, your file (delivery_id, checksum, received_at) showed Y and your previous delivery for the same as-of date showed Z — please confirm which is current and whether a correction notice was issued" gets an answer that can be acted on and filed.

**The process, in five steps.** (1) **Verify the break from retained evidence** — which requires §4.6, and is why the query process cannot be built before the raw store exists. (2) **Quantify and scope it** — the number of instruments and dates affected, and which of the firm's published numbers consumed them. (3) **Query with the evidence attached**, in the vendor's own identifiers and delivery references, through the contracted support channel rather than a personal relationship. (4) **Record the vendor's answer as a knowledge-time event** on the affected records, whatever it says — including "no error," because "the vendor confirmed the value" is itself part of the audit trail. (5) **Track the resolution to a change** — a correction in a later delivery, a documented methodology confirmation, or an accepted policy difference recorded under change control.

**The one structural observation that surprises firms:** several vendors can tell a client how many price observations they have and on what dates, and are contractually used to being asked — the Basel Framework's model-eligibility requirements make precisely that information a condition of a bank's use of vendor prices for the risk factor eligibility test (MAR31.14, ✅ verified at source), including auditability of the vendor's pricing information. A firm's vendor-management function that is not asking for observation counts and dates is leaving a supervisory-grade capability on the table.

### 10.5 Change Control Over the Crosswalk and the Adjustment Conventions

**Two artefacts change rarely and matter enormously, and both need the same discipline: a request, an impact assessment, an approval, an effective date and a rollback.**

| Change | Who must approve | The impact assessment that must exist |
|---|---|---|
| **A crosswalk mapping change** (§3.3) | Security-master owner, with the affected consumers informed | Which reports, models and reconciliations consumed the old mapping, and whether any published number is affected |
| **An adjustment-convention change** (§5.6) | Risk/finance (the consumer), with the data owner executing | The before/after effect on the historical series, the effect on model inputs and on any published performance or risk number, and the effective date from which the new convention applies |
| **A tolerance change** (§4.3) | The consuming business, not the data team | The change in break population it produces, and an explicit statement of what it will now stop detecting |
| **A retention-period change** (§4.6) | Data owner plus risk/audit | What evidence will no longer be available, and which future questions become unanswerable |
| **A vendor or product change** (coverage, feed, licence class) | Data management plus licensing and the consumers | Coverage gained and lost, the mapping work, and the point-in-time consequence of switching the *source* of a field mid-history |

**The last row is the one that catches firms out.** Changing a data source looks like a procurement event; it is really a *history event*, because the new source's restatement behaviour, adjustment convention and coverage behind the switch-over date are different. The impact assessment must therefore state *from which valid date* the new source becomes the firm's assertion, and what the record will say about the dates before it — which is only answerable if both sources' raw answers were retained.

### 10.6 The Audit Trail a Regulator Would Ask For

**The questions, in the order they are actually asked, and the artefact that answers each.** This list is the practical test of §8's design; if the firm cannot answer any row, that row is the roadmap.

| The question | The artefact |
|---|---|
| "Show me the record for instrument X on 15 April." | The published golden version for 15 April, with per-field source (§8.2, §9.1) |
| "What did you know on 20 April about 15 April, and what do you know now?" | Both axes on the golden record; the two rows (§6.6) |
| "Why did the value change between those dates?" | The data change notice, the retained vendor deliveries, and the lineage across the change (§6.3, §9.4) |
| "Who approved the value you used instead of the default source?" | The override register, with reason, approver and expiry (§4.4) |
| "Show me the supplier's original file." | The immutable vendor-raw store, checksum-addressed (§8.3) |
| "What evidence do you have that this price was a real price observation?" | The vendor's observation counts and dates, and its auditable pricing information (MAR31.14 ✅); the firm's own mapping of real prices to risk factors, which MAR31.15 requires the bank to be able to evidence through documented policies and procedures ✅ |
| "Which instruments were in the universe on that date, including the ones that have since delisted?" | The point-in-time universe record (§6.4) |
| "What is your tolerance for this field, and who agreed it?" | The tolerance policy, with the approving business owner and the date (§4.3) |

**The tone of this list is the point.** None of these questions is exotic; all of them are answerable *only* by structures designed before the question was asked. That asymmetry — cheap to build early, impossible to retrofit — is the reason this guide puts the storage decisions in §8 and the governance decisions in §10.5 rather than treating them as implementation details.

### 10.7 The Honest Point: No Tolerance Policy and No Owner Means Nobody Acts

**A reconciliation with no agreed tolerance and no named owner produces a report nobody acts on, and it does so while consuming real effort.** This is not a cynical observation about human nature; it is a structural property. Without a tolerance, every difference is arguably a problem, so nothing is a priority. Without an owner, every break is somebody else's, so nothing is actioned. Without an age SLA, nothing is urgent, so the queue grows. The result is the familiar artefact: a reconciler whose daily output is a large spreadsheet that is reviewed by nobody, maintained faithfully, and cited in audit as evidence of a control that does not exist.

**The three-part fix, in order of impact.** (1) **Name owners for break classes**, even if the classes are crude to begin with — ownership is what converts a report into a process (§10.2). (2) **Agree tolerances with the consuming business** and write them down, even if the first version is a placeholder that the business signs off as provisional (§4.3). (3) **Give the queue an age SLA and an escalation forum** so that the population has a reason to be small (§10.3). None of these is engineering work, all three are prerequisites for the engineering work being useful, and the sequencing is deliberate: the guide has put them last because they are the ones most often deferred, and the ones without which nothing earlier in the guide pays off.

---

## 11. The Regulatory Frame

**Cross-reference and pointer, not compliance advice.** This section states which regulatory expectations touch market-data integrity, verifies the one requirement that is specifically about market-data quality at its source, applies the standard's own grouping to BCBS 239 while recording a discrepancy in a sibling guide, and routes the reader to the repository's regulatory material rather than restating it. It contains no advice on compliance, no interpretation of how any supervisor will apply a rule to a particular firm, and no claims about any institution's arrangements.

### 11.1 FRTB — What the Market-Risk Framework Demands of the Price Record

**The framework, verified at source.** The Basel Committee's *Minimum capital requirements for market risk* — publication **d457** — was **published 14 January 2019**, with a corrected version on **25 February 2019**, **effective 1 January 2022**, and has been **integrated into the consolidated Basel Framework** (✅ verified at bis.org/bcbs/publ/d457.htm). Its core features include an internal models approach that relies on expected shortfall models and **sets out separate capital requirements for risk factors that are deemed non-modellable** (✅ the same page). In the consolidated framework, that content lives in the **MAR** standard: MAR30 (general provisions), MAR31 (model requirements, which is where risk-factor eligibility is defined), MAR32 (backtesting and P&L attribution), MAR33 (capital requirements calculation) — the chapters were read directly this pass (✅).

**The market-data requirement, in the standard's own terms.** For a risk factor to be classified as modellable, a necessary condition is that it **passes the risk factor eligibility test (RFET)**, which "requires identification of a sufficient number of real prices that are representative of the risk factor"; **collateral reconciliations or valuations cannot be considered real prices** to meet the test; and a price is real only if it meets at least one of four criteria — a price at which the institution transacted; a verifiable price for an actual transaction between other arm's-length parties; a committed quote collected and verified through a third-party vendor, trading platform or exchange; or a vendor price meeting those conditions (MAR31.12, ✅ verified at source). The test's numeric thresholds are explicit: **at least 24 real price observations per year** (measured over the period used to calibrate the current ES model, with no more than one observation per day counted), **with no 90-day period in which fewer than four real price observations are identified**, monitored monthly; **or at least 100 real price observations over the previous 12 months** (MAR31.13, ✅ verified at source).

**The three demands this places on a bank's data management, stated as consequences.**

- **Provenance, not just value.** A bank must be able to say *why* a price is real, which means its own price record must retain the observation type and the source, per price — a data-model requirement, and the reason §4.3's price tolerance is not the whole story.
- **Vendor evidence, including counts and dates.** Where a bank counts real prices collected from a third-party vendor, the vendor must communicate **the number of corresponding real prices observed and the dates at which they have been observed**, provide a minimum necessary set of identifier information so the bank can map prices to risk factors, and be subject to an audit of the validity of its pricing information whose results are available to the supervisor on request (MAR31.14, ✅ verified at source) — which is precisely the "what did you know, and when" requirement this guide is about, written as a condition of using the data.
- **Documented mapping, and a bucketing choice.** A real price is representative for a risk factor where the bank can extract the factor's value from it, and the bank must have **policies and procedures describing its mapping of real price observations to risk factors**, with sufficient information to supervisors to judge the methodologies; where a risk factor is a point on a curve or surface, the standard sets out a **bucketing approach** for counting observations (MAR31.15 and MAR31.16, ✅ verified at source).

**Implementation status, flagged with the date checked.** The standard's own effective date is **1 January 2022**, and the consolidated framework chapters carry "effective as of" dates of **1 January 2023** with publication dates in 2020 (✅ both read from the BIS pages on the date of this pass, September 2026). **⚠ The jurisdictional application dates — which supervisors applied the framework when, and with what national modifications — were not verified this pass**, and this guide makes no statement about them; the repository's [Basel Regulatory Capital](basel_regulatory_capital_guide.md) and [Enterprise Risk Management](enterprise_risk_management_guide.md) guides are where the capital-framework context belongs.

### 11.2 BCBS 239 — Accuracy, Completeness, Lineage, Using the Standard's Own Grouping

**The standard's own structure, which is the only grouping this guide uses.** BCBS 239, *Principles for effective risk data aggregation and risk reporting* (January 2013), numbers fourteen principles in a single sequence grouped under four sections: **Section I — Overarching governance and infrastructure = P1–P2; Section II — Risk data aggregation capabilities = P3–P6; Section III — Risk reporting practices = P7–P11; Section IV — Supervisory review, tools and cooperation = P12–P14** (✅ established from the standard's own published text and recorded at [Senior Data Platform Architect](../technology/senior_data_platform_architect_guide.md), whose claims audit carries the verification row and whose BCBS 239 section records the correction).

**⚠ The discrepancy, recorded not inherited.** [Risk Data Aggregation](risk_data_aggregation_guide.md) **§2 is headed "The Governance Principles (P1–P5)"** (line 123 of that file, and the matching entry in its table of contents). That grouping is not the standard's: Section I covers **P1–P2 only**, and P3–P6 are Section II. In fairness to that guide, its §2 body text and its §1 numbering note *do* state the standard's own mapping and explicitly describe P1–P5 as the industry's treatment rather than the standard's — so the issue is the heading's label, which asserts a grouping the standard does not use. **Do not edit that guide**; cite it for BCBS 239's substance and use the standard's grouping when the grouping itself is the point. (The same correction is recorded in [Senior Data Platform Architect](../technology/senior_data_platform_architect_guide.md) — the claims-audit row verifying "Section I = P1–P2; Sections II/III/IV = P3–P6 / P7–P11 / P12–P14" against the published PDF.)

**What BCBS 239 asks that market-data integrity answers.** The principle-by-principle content is [Risk Data Aggregation](risk_data_aggregation_guide.md)'s and is not re-derived here; what matters for this guide is the direct mapping from the standard's expectations onto the structures of §8:

| BCBS 239 expectation | The market-data integrity structure that evidences it |
|---|---|
| **P3 Accuracy and integrity** — accurate, reliable data, aggregated largely automatically | The reconciliation controls of §4 (detective) and the validation and crosswalk enforcement of §9.2 (preventative); the break queue's root-cause discipline of §10.2 |
| **P4 Completeness** — all material risk data, cut by entity, line, asset type, industry, region | The point-in-time universe records of §6.4 and the coverage break class of §4.3; the LEI and entity-level handling of §3.2 |
| **P5 Timeliness** | The publication cycle and versioned golden record of §9.1 / §8.2 (the timeliness discipline itself is the subject of the repository's late-arriving-data material — cross-reference, do not re-derive) |
| **P2 Data architecture and IT infrastructure** — supporting aggregation in normal times *and* in stress | The layered pipeline of §9.1, with the raw store as the evidence layer and independently re-runnable stages |
| **P1 Governance**, and the lineage expectations supervisors test | The ownership, change control and audit trail of §10 — and lineage across restatements (§9.4) |
| **P6 Adaptability** | The bitemporal record's ability to answer a *new* question about an old date without rebuilding the past (§8.2) |

**The line to draw, stated once.** BCBS 239 governs **risk** data aggregation; market and reference data are *inputs* to it, and the standard's applicability to a given firm's market-data estate depends on whether that data is material to the bank's risk management and reporting. This guide does not adjudicate that scope question; it notes that a break in the instrument record propagates into every principle above, which is why market-data integrity work is invariably *found* by a BCBS 239 programme rather than initiated by one.

### 11.3 The Prudential and Reporting Regimes That Consume Market Data

**Named at the level of what the regime consumes, rather than what it requires — and the detail is routed, not restated.** Four families matter, and each has a repository home:

- **Market-risk capital (the FRTB family).** Consumes prices, risk factors and the evidence of their realness — §11.1 above, verified at source; the capital-framework context is [Basel Regulatory Capital](basel_regulatory_capital_guide.md), with the model-governance angle in [Enterprise Risk Management](enterprise_risk_management_guide.md).
- **Risk data aggregation and reporting (BCBS 239).** Consumes every instrument, entity and price in the aggregation universe — §11.2; the standard itself is [Risk Data Aggregation](risk_data_aggregation_guide.md).
- **Prudential reporting and treasury returns.** Consume instrument-level positions, classifications and valuations, which is where a mis-identified instrument becomes a mis-classified exposure; the repository's [Treasury & ALM](treasury_alm_guide.md) and [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) guides cover the reporting chains and their technology.
- **OTC-derivatives transaction reporting.** Consumes the derivative identifiers: the **UPI (ISO 4914:2021)** exists to identify OTC derivative products **reportable to trade repositories** so that authorities can aggregate reported transactions by product or by reference-data element (✅ verified at source — the standard's abstract and the DSB's designation), and the **UTI** exists to identify the transaction (✅ the concept is anchored in CPMI-IOSCO's February 2017 *Technical Guidance*). A bank whose internal derivative taxonomy does not map to the UPI has a reporting-data problem that no amount of downstream validation fixes (§3.2).

**The one requirement worth naming explicitly in the reporting family**, because it is where market-data integrity is not a preference: **the obligation to report the correct instrument**. Every regime in this list carries instrument identity in its returns — by ISIN, by national identifier, by UPI/UTI or by an entity identifier — and a wrong mapping is a wrong return regardless of how good the valuation was.

### 11.4 The Conduct and Reporting Regimes That Depend on Reference and Corporate-Action Data

**The family, stated at the level of dependency rather than of requirement.** Beyond the prudential frame, reference and corporate-action data are load-bearing for three conduct- and disclosure-shaped activities, and the repository covers each in depth:

- **Fund valuation, NAV and investor reporting.** A corporate action applied wrongly, or an adjusted-versus-raw convention applied inconsistently, changes a NAV — the calculation, the trade-date-versus-settlement-date treatment, the fair-value hierarchy and the fund-administrator relationship are [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §6, and this guide's §5.6 is the data-side precondition for all of it.
- **Performance, attribution and benchmark reporting.** Requires a **continuous** series across events and a **point-in-time** benchmark composition — §5.6's adjustment conventions and §6.4's point-in-time universes respectively; the measurement frameworks themselves are [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §7.
- **Client reporting and mandate compliance.** Depends on correct instrument characteristics (classification, restrictions, eligibility) — the reference-data foundation is that guide's §8, and the compliance machinery its §9.

**Also worth naming, without asserting a requirement:** entitlement and conduct regimes turn on *who knew what when* in precisely the sense this guide's thesis uses — a firm's ability to show the sequence of what it was told, when it acted, and which version of the terms it acted on (the versioning of §5.1 and the audit trail of §10.6) is the same evidence in a conduct context as in a prudential one.

### 11.5 Where to Read the Detail in This Repository

| Topic | Read |
|---|---|
| BCBS 239 in full, with its implementation history | [Risk Data Aggregation](risk_data_aggregation_guide.md) (use the standard's own section grouping — §11.2 above) |
| Basel capital framework and FRTB context | [Basel Regulatory Capital](basel_regulatory_capital_guide.md) |
| Risk-management and model-governance context | [Enterprise Risk Management](enterprise_risk_management_guide.md) |
| Regulatory-reporting systems and reconciliation technology | [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) |
| Treasury and ALM reporting chains | [Treasury & ALM](treasury_alm_guide.md) |
| MAS regulations, notices and outsourcing expectations | [MAS Regulations & Guidelines](mas_regulations_guidelines_guide.md) |
| Data governance, lineage and data-quality frameworks | [Data Governance Framework](../technology/data/data_governance_framework.md), [Data Lineage Tools](../technology/data/data_lineage_tools.md) |
| The data-platform mandate and its BCBS 239 correction | [Senior Data Platform Architect](../technology/senior_data_platform_architect_guide.md) |

**No compliance advice is given here, and nothing in this section should be read as an interpretation of how any supervisor will apply any requirement to any firm.**

---

## 12. The Cymbal Bank Worked Example

> **Everything in this section is fictional and illustrative.** Cymbal Bank is a fictional institution used throughout this repository as its worked-example persona. The two market-data vendors are referred to only as **Vendor A** and **Vendor B**; no real vendor is asserted to be a supplier to Cymbal, and no real institution is asserted to use any vendor or product. **All figures, dates, instrument names, quantities and costs in this section are illustrative and invented for the narrative** — they are not market data, not vendor terms, and not benchmarks of any kind. The mechanisms and the structures, however, are exactly those of §§3–10.

### 12.1 The Scenario

**Cymbal Bank runs two market-data vendors, both licensed, both feeding the same security master.** Vendor A supplies the firm's primary evaluation and end-of-day service; Vendor B supplies a second opinion and is the source of record for one segment of the book. Cymbal's design is the common one: the security master takes static and reference data from Vendor A, prices from Vendor A with Vendor B as a cross-check, and corporate actions from both, with the reconciliation run nightly against a tolerance table that a previous programme produced and that nobody has reviewed since.

**The instrument at the centre of the story** is a listed equity — called **XYZ Ltd** here — held in an illustrative long position of **SGD 12.0 million** in the equity book and in a smaller position inside one **illustrative research portfolio** used by the quant team. Both figures are invented for the narrative.

### 12.2 The Break: One Static Attribute, Two Vendors

On a Tuesday in September, the nightly reconciliation posts **a single break** on XYZ Ltd. It is not a price break; it is a **static-attribute break** on the field `shares_outstanding`, and it is enormous in magnitude:

| | Vendor A (illustrative) | Vendor B (illustrative) |
|---|---|---|
| `shares_outstanding` | 250,000,000 | 25,000,000 |
| `price_close` for the same date | SGD 1.20 | SGD 12.00 |
| Implied market capitalisation | SGD 300 million | SGD 300 million |

**The first observation is the one that prevents a week of wasted work: the two vendors agree completely on the economics and disagree entirely on the units.** A ten-for-one stock split took effect in the market; Vendor A has applied it to both quantity and price, and Vendor B had not yet applied it at the time of the run. The accounting for the break, in the cause families of §4.1, is **correction state** — and, more precisely, **corporate-action state**: two providers at different stages of applying the same event.

**Why the tolerance regime flagged it correctly and could easily not have.** The break fired because identifier and enumerable static fields carry an **exact-match** tolerance (§4.3). Had `shares_outstanding` been compared with a percentage band, a 10× difference would still have fired — but had the split been a **1.1-for-1** event of the sort that occurs routinely, a percentage-based rule applied to a quantity field would have produced a "within tolerance" result and a **silent 10% error in every per-share and market-cap number derived from it**. This is the concrete form of §4.3's rule that each field type needs its own tolerance: the *tolerance category*, not the size of the difference, is what decides whether a break is seen.

### 12.3 Investigating: The Identifier Crosswalk and Its Effective Dates

**The investigation starts where §3 says it should: with the identifier.** XYZ Ltd's ISIN is unchanged — the split was a quantity-and-price event, not a reorganisation — and both vendors map the same ISIN. The two records are about the same instrument, so the break is not an identification break (§4.1). What the investigation does surface, however, is **two defects in the crosswalk that the split exposed**.

- **Defect one: the crosswalk's mapping rows carry no effective dates.** Cymbal's crosswalk was built as a two-column daily refresh: `isin → vendor_b_instrument_key`. Because Vendor B's instrument key for XYZ Ltd also changed with the split (an illustrative artefact of that vendor's naming of lines), the daily refresh had already overwritten the previous key. The firm therefore had **no record of what Vendor B's key had been before the split**, which meant it could not simply re-point its mapping — it had to ask the vendor to confirm the key history in writing. With the effective-dated crosswalk of §3.3, the prior row would still have been there with its `valid_to`.
- **Defect two: the corporate-action linkage between the pre- and post-event states existed only inside a spreadsheet.** The quant team was maintaining its own mapping of "which price series belongs to which instrument" for the research universe — a second crosswalk, undocumented, unreconciled, and in this case **stale by one split**. This is the §4.5 consistency problem arriving from outside the platform: the golden record said one thing and the research cache said another.

**The data-model lesson, stated in the guide's own vocabulary.** The instrument's identity did not change; the instrument's *state* did, and the firm had no place to record the state transition (§5.3, §8.4). The crosswalk needed dates *because of the corporate action* — which is §2.5's claim that cause three generates both of the other problems, appearing in a real break.

### 12.4 The Tolerance Question and the Break Workflow

**The break ran the workflow of §4.4, and the workflow's quality — not the fix's speed — is what the example is about.**

| Stage | What happened at Cymbal (illustrative) | What it demonstrates |
|---|---|---|
| Detection | The nightly run posted one break: `shares_outstanding`, exact-match class | Detection is the cheap part |
| Classification | Cause family tagged `correction_state` / corporate action; severity high, because a quantity error propagates to per-share metrics | The taxonomy of §4.1 is what makes routing possible |
| Investigation | The investigator needed the prior mapping row (gone), the vendor's pre-split key (had to be requested), and the quant cache (found, stale) | §4.6 in one sentence: two of the three artefacts should have been retained by the firm |
| Ownership | Routed to the security-master steward, with risk and the quant team as affected consumers | The correct owner is the field's owner (§10.2) |
| Aging | Open nine business days, two of them waiting on a vendor correspondence | Age is the control's teeth — and the loop that only closes on a vendor's reply is a *latency defect in the firm*, not in the vendor |
| Escalation | Escalated at day ten to the data governance forum on materiality — the research portfolio's exposure to XYZ Ltd was small, but the *backtest history* was not | Materiality must be measured in **consumers and history**, not only in position size |
| Resolution | Vendor A's post-split values accepted as the golden record from the split's effective date; Vendor B's key history recorded; the quant cache retired | Resolution is a *recorded decision*, not a data edit |
| Override | The split's effective date in the golden record was set from Vendor A's event feed rather than from the default static-data source, and the override was recorded with a reason, an approver and an expiry | The anti-pattern §13 rejects is an override with no audit trail — here it existed and was the reason the fix was explainable |

**The tolerance question the forum actually asked, and could not answer in the meeting:** *what tolerance did we agree for a quantity field, with whom, and when?* The tolerance table had a row for `shares_outstanding` reading "exact match" with **no approving business owner recorded** and no review date. The break had therefore been detected correctly and *accepted* on no authority. The forum's action was not to change the tolerance — exact match was right — but to record who owned it (§10.7 in practice).

### 12.5 The Adjustment Decision and Its Effect on the Historical Series

**The split raised the question the guide flags as a data-model decision, not a calculation (§5.6): which series do we store, and which do we derive?**

Cymbal's position before the break was, in effect, two positions: the **risk engine** consumed *raw* prices from Vendor A, and the **quant research cache** consumed *split- and dividend-adjusted* prices from Vendor B. Both were defensible choices for their purpose. The defect was that neither was *stated*, and nothing captured the **factor** that related them — so when the split arrived, the risk engine's series had a 10× discontinuity on the effective date, and the research cache's series had a 10× discontinuity *before* it, because Vendor B had not yet applied the event when the cache was built.

| Consequence (illustrative) | What it looked like |
|---|---|
| **Risk measures** | Returns computed across the split's effective date in the raw series produced an illustrative one-day return far outside any plausible distribution — a data artefact that, if it survived validation, would inflate a volatility measure and any expected-shortfall or VaR calculation consuming it |
| **Position and exposure reporting** | Per-share metrics derived from a pre-split quantity against a post-split price (or the reverse) were wrong by a factor of ten, with no error raised anywhere |
| **Research history** | The cache's series was discontinuous in the other direction: unadjusted for the new event, and — as §12.6 shows — already stale in ways nobody had measured |
| **Reconciliation** | The nightly price comparison kept matching, because both vendors' prices were internally consistent with their own quantities: **the break was invisible in the price field and visible only in the quantity field** |

**The decision Cymbal took**, and the point of the example: **store raw, store the factor, derive the view.** Prices are stored as published; the split's factors live in the corporate-action ledger with effective dates (§8.4); adjusted series are computed on request from a *named* convention, recorded per series; and the convention itself is now a change-controlled artefact with a consumer sign-off (§10.5). The immediate effect is that the risk engine's series and the research series are both reproducible and clearly different — and that the difference between them is now a documented fact rather than a discovery.

### 12.6 The Discovery: A Backtest Consuming Restated Prices

**The investigation into the split turned up something larger, and it is the reason this example exists.** While reconciling the research cache against the golden record, the team found that the cache had been built **fourteen months earlier** and refreshed by appending only *new* dates — never re-reading history. Comparing the cache's historical values against what the firm's vendor files actually contained at the time revealed two classes of discrepancy (all figures illustrative):

- **Coverage extension.** Vendor B had, in the intervening period, extended its history backwards for a segment of the research universe; the cache had no rows for that period, so the backtest had silently run on a **smaller universe** in those months than the firm's current data implies.
- **Restatement.** For a subset of instruments, the vendor's historical values had changed — corrections loaded with no change flag — and the cache held the **superseded** values from fourteen months earlier.

**So the research portfolio's published results had been produced from a dataset that was neither the current record nor a recorded vintage.** It was a third thing: a stale snapshot with no knowledge date. Nobody could say whether the results would be better or worse on corrected data; what could be established was that they were **not reproducible**, which is the point at which the discussion stopped being about market data and became about model validation.

**Mapping the discovery onto the guide's sections, because the mapping is the lesson:**

| Discovery | The mechanism | The section |
|---|---|---|
| Coverage extended backwards; cache had holes | Restatement and backfill with no knowledge axis; a "current snapshot" treated as history | §6.2, §6.3 |
| Historical values changed with no notice | Silent restatement; detectable only by diffing retained versions | §6.3, §4.6 |
| A smaller universe in early months | Survivorship and coverage bias in the *universe*, invisible to per-field reconciliation | §6.4 |
| Results not reproducible | No knowledge-dated read; the dataset was undated | §6.5, §6.6, §6.7 |
| Nobody could quantify the impact | No retained prior golden values against which to measure | §8.2, §4.6 |

### 12.7 The Point-in-Time Fix and What It Costs to Adopt

**Cymbal's fix was deliberately the cheap, durable end of the options in §6.6 and §8.5, in priority order.**

| Move | What it involved (illustrative) | What it cost | What it bought |
|---|---|---|---|
| **1. Retain the raw vendor deliveries, immutably** | An append-only landing area with receipt timestamps and checksums for both vendors' daily files and event feeds | Storage volume and a written retention decision — cheap per terabyte, and the retention decision was the hard part | The evidence layer: every future break becomes investigable (§4.6) |
| **2. Effective-date the crosswalk** | Adding `valid_from` / `valid_to` / knowledge columns and closing rows on change instead of refreshing | A small schema change plus a reconciliation between the two vendors' keys | The map stops being overwritten; the next split needs no vendor correspondence (§3.3) |
| **3. Build the corporate-action ledger with versions and factors** | An event table driven by both vendors' event feeds, with factors and effective dates; the split was the first event loaded | Integration work with both feeds, and a decision about what "completed" means for each event type | Adjustment becomes reproducible and reconcilable at the *event* level (§8.4, §5.5) |
| **4. Extend the bitemporal treatment to prices, statics and corporate-action fields** | Both axes on the highest-value fields; other fields left on the existing model | The significant cost — a data-model change touching the golden record's consumers | The audit and restatement answers (§8.2, §10.6) |
| **5. Publish a daily immutable golden version** | A dated snapshot (the lighter route of §6.6) rather than a full time-travel regime | Small, if the golden record is already versioned | "What did we publish in March" becomes a lookup |
| **6. Add point-in-time universe counts to the daily checks** | Counts of instruments with a price per historical date, trended and alerted | A query and an alert | The survivorship control that per-field reconciliation can never provide (§6.4) |
| **7. Retire the research cache and require knowledge-dated reads** | The quant team queries the platform's versioned read instead of maintaining a parallel store | Migration effort for the research function, and some performance cost | One source of truth for history, and reproducibility by construction (§6.5) |

**What adoption did not fix, stated honestly because the guide's §15 promises it.** The **fourteen months of history that were not recorded** could not be reconstructed: the cache's knowledge dates are unknowable, the superseded values that the cache held are recoverable only if the platform's *own* prior loads happened to retain them, and the affected backtest results will carry an unquantified uncertainty forever. That is §6.7 with a date stamp on it, and it is the argument that carried the funding.

### 12.8 The Recommendation, Including What Cymbal Decides to Stop Doing

**The recommendation to Cymbal's data governance forum (illustrative), in the form the guide's sections imply.**

1. **Approve the five structures of §8.5 in priority order**, with the raw store and the effective-dated crosswalk funded first — because they are the two whose late adoption is unrecoverable.
2. **Name owners and record approving authorities for every tolerance row** in the reconciliation policy, and put the policy on a review cycle (§4.3, §10.7).
3. **Make the adjustment convention a documented, change-controlled, consumer-approved artefact** — one convention per *published series*, effective-dated (§5.6, §10.5).
4. **Add the restatement diff and the universe-completeness count to the daily cycle** as first-class controls with break classes of their own (§6.3, §6.4).
5. **Adopt the MAR31.14-grade vendor evidence expectation as a contractual query habit** — observation counts and dates on request (§10.4, §11.1).

**And the things Cymbal decides to STOP doing** — because a recommendation that only adds is a recommendation that will not be executed:

- **Stop consuming the vendor's adjusted price series as the platform's raw price input.** The adjusted series is a *view* produced under a vendor convention the firm does not control and may not know; the platform stores the as-published value and derives the view under its own recorded convention (§5.6). This is the single change that removes the whole class of §12.5's defects.
- **Stop maintaining a second crosswalk outside the platform.** The research function's private mapping is retired; mappings are the security master's, with dates (§3.3, §12.3). Two mappings is not redundancy; it is two versions of the truth with no reconciliation between them.
- **Stop closing reconciliation breaks by "they match now."** A break that disappears because the vendor's numbers moved is a restatement event and is recorded as one, with a knowledge-time entry and an impact note (§4.4, §6.3). This is the practice that had been hiding §12.6.
- **Stop accepting overrides without an approver and an expiry.** Every override in the register is reviewed; those without an authorising record are re-decided or reversed (§4.4, §10.6).

**The one-line summary of the example, which is also the guide's argument compressed.** A break on a single static field turned out to be a corporate action in mid-application, which turned out to be a crosswalk with no dates, which turned out to be a fourteen-month-old unrecorded vintage feeding a model — and every one of those links was discoverable *because* somebody had kept something; the missing links were exactly the things nobody had thought to keep.

---

## 13. The Anti-Patterns

**Seven failure modes that account for most market-data integrity incidents, in symptom / cause / guardrail form.** Each one has appeared in this guide as a positive recommendation; here they are as what the defect looks like from the inside.

| # | Anti-pattern | Symptom | Cause | Guardrail |
|---|---|---|---|---|
| 1 | **Crosswalk with no effective dates** | Nobody can say what an instrument was mapped to last quarter; a mapping change destroys the prior mapping; vendor correspondence is needed to reconstruct a key | A two-column mapping table refreshed daily, designed as a lookup rather than a statement about time | Effective dates on both axes; close rows instead of overwriting (§3.3, §8.1) |
| 2 | **Golden source chosen per field with no consistency check** | Every field is within tolerance and the record is internally contradictory; arithmetic that should tie does not; investigations have no starting point | Best-of-breed sourcing adopted for per-field quality, without whole-record coherence | Field-level provenance; arithmetic tie controls; anchor coherence at the event; periodic whole-record comparison (§4.5, §9.2) |
| 3 | **History adjusted in place with no record of the pre-adjustment value** | The as-traded price is unrecoverable; the adjusted series cannot be recomputed under a different convention; "what factor did we apply in March" is unanswerable | Adjustment implemented as a transformation on the price pipeline rather than as a derived view over a recorded event ledger | Store raw, store factors in the ledger, derive the view; never mutate the observation (§5.6, §8.4) |
| 4 | **A backtest built on a current snapshot** | Results that a reviewer cannot reproduce; a universe that flatters the strategy; restated historical values consumed unknowingly | History pulled from today's record with no knowledge date, refreshed by appending new dates only | Knowledge-dated reads; point-in-time universes; retained prior golden versions (§6.4, §6.5, §6.6) |
| 5 | **Corporate actions applied downstream instead of driving the data model** | Each consumer applies its own convention; the risk series and the research series differ with nobody knowing why; a missed event propagates as a plausible wrong number | Corporate actions treated as an operations feed for cash and custody, not as a state transition in the instrument's record | The event ledger drives identifiers, quantities and factors; completeness control with the event feed as a break class (§5.2, §5.3, §8.4) |
| 6 | **An override with no audit trail** | The golden record contains values nobody can authorise; the "source of record" is a patchwork; audit cannot attribute a value | Overrides made as a convenience in a load process or a spreadsheet, with no register | An override register with reason, approver and expiry; overrides as data, not as code (§4.4, §10.6) |
| 7 | **A reconciliation whose tolerances were never agreed with anyone** | A large queue that nobody actions; every difference is arguable; the report is cited in audit as evidence of a control that does not exist | Tolerances set by the data team, or by nobody, and never reviewed with the consuming business | Tolerances per field type, approved by the consumer, reviewed on a schedule, with the approving authority recorded (§4.3, §10.7) |

**Three secondary patterns worth naming because they masquerade as good practice.** The **auto-close automation** that closes breaks when the next run happens to match — it hides restatement and manufactures a clean queue (§4.4). The **total-break-count metric** that is managed by suppressing the total rather than by resolving breaks (§10.2). And the **second crosswalk** maintained by a research or risk team "for speed" — the §12.3 defect, which is not redundancy but a competing version of the truth.

**What all seven have in common, in one sentence.** Every one of them is a *decision taken by default* rather than by design: the table refreshed instead of versioned, the field sourced rather than reconciled, the adjustment applied rather than recorded, the snapshot treated as history, the event processed rather than modelled, the override granted rather than registered, the tolerance assumed rather than agreed. The remedy in every case is the same shape — make the decision explicit, date it, and write down who made it.

---

## 14. The Claims Audit

### 14.1 The Convention

**Every factual claim in this guide that carries a standard number, an issuing body, an ownership state, a vendor or product name, a date or a threshold is listed below with its status and its source, and the date the source was checked.** The convention:

- **✅** — verified this pass against the source named, on the date shown. Where the source is a sibling guide's verified claim rather than a page opened this pass, the row says so explicitly.
- **⚠** — flagged: secondary-sourced, snippet-level, not re-verifiable this pass, or verified only in part. Each flagged item is listed again in §15.
- **❌** — rejected: a claim in circulation that the evidence contradicts, or an inference that the evidence does not support.

**Two research conditions that affect how these rows should be read.** First, the **web search backend returned empty result sets on several attempts during this pass** (recorded as a tool limitation in §15, not as evidence of absence), so much of the verification here was done by direct extraction of primary pages rather than by search — which is the stronger method, but it also means some items were reached by URL rather than discovered. Second, **several pages were read in their extracted form and one page (DTCC's corporate-actions section) returned a 404**, so the corporate-action message standards are cited from the sibling guide that verified them rather than re-verified here.

### 14.2 The Claims Audit — Verified, Flagged, Rejected

| # | Claim | Status | Source | Date checked | Quality / note |
|---|---|---|---|---|---|
| 1 | **ISIN** is defined by **ISO 6166**; ISINs are 12-character; **ANNA** is the ISO-appointed **Registration Authority** for ISIN (and CFI); **NNAs** allocate within national prefixes | ✅ / ⚠ | ANNA's own service site (annaservice.com — "ANNA serves as the Registration Authority for ISIN and CFI numbering standards, under appointment by ISO"); ANNA homepage (anna-web.org) | Sept 2026 | The RA role and the 12-character format read from ANNA's own material; the ISO 6166 number is ANNA's own citation and was **not** opened at iso.org this pass ⚠ |
| 2 | The **ANNA Service Bureau (ASB)** aggregates identifier data from **120+ NNAs** across **200+ jurisdictions**, with **1 million+ new ISINs monthly**, collecting "since 2001"; the ASB is **developed and operated by ANNA partners CUSIP Global Services and SIX Financial Information** | ✅ | ANNA, "Global ISIN Hub — The ANNA Service Bureau" (anna-web.org/about-the-anna-service-bureau/) | Sept 2026 | Read directly from ANNA's own page; the operator statement is the ownership-critical fact — **the ASB is run by CGS (FactSet) and SIX**, which is the current ownership state, not the historical one |
| 3 | **CUSIP Global Services (CGS)** is the US national numbering agency; **FactSet completed its acquisition of CGS on 1 March 2022 for approximately US$1.925bn**, announced 27 December 2021, acquired **from S&P Global**; CGS continues "in close partnership with the **American Bankers Association**" | ✅ | FactSet press release, "FactSet Completes Acquisition of CUSIP Global Services" (investor.factset.com, 1 March 2022); the same text filed as SEC exhibit 99.1 (sec.gov/Archives/edgar/data/1013237/000095015722000210/ex99-1.htm) | Sept 2026 | **Verified at the acquirer's own press release and its SEC filing.** This **confirms** the figure and date stated in [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §8.2 — no discrepancy to report; the deal value and completion date are as that guide states |
| 4 | **SEDOL** identifiers are assigned by the **London Stock Exchange** on the request of market participants, delivered through the **SEDOL Masterfile**, described by LSEG as a near-real-time global reference dataset "in operation for over 30 years" | ✅ / ⚠ | LSEG SEDOL Masterfile page (lseg.com/en/data-analytics/market-data/data-analytics-pricing/data-symbology/sedol), the LSEG SEDOL Masterfile brochure PDF, and the London Stock Exchange symbology contact page | Sept 2026 | The issuing body and the on-request allocation model are verified from LSEG's own material; **the age of the system is marketing copy — and LSEG's own two pages disagree (over 30 years here, over 40 years on the Symbology page, see row 32)** ⚠ — and the historical claim that SEDOL has been issued "since 1979" is **not asserted** by this guide (the sibling guide also flags it) |
| 5 | **RIC**: proprietary vendor symbology; LSEG's own Symbology page describes the RIC as a market-level identifier for instruments and pricing sources **issued by LSEG**; the page also states that PermID is issued by LSEG | ✅ | **LSEG Symbology page** (lseg.com/en/data-analytics/market-data/data-analytics-pricing/data-symbology — read this pass) | 22 Sept 2026 | **Upgraded from ⚠ to ✅ in the parent's verification pass.** The issuing body is now source-verified; what LSEG does not publish is the RIC's internal structure and port convention, so the guide treats the syntax as vendor-documented rather than standardised (§3.2) |
| 6 | **FIGI** is a standard of the **Object Management Group (OMG)**, issued and distributed by **Bloomberg L.P. as Registration Authority under OMG auspices**, with **Bloomberg and Kaiko as Certified Providers**; it is **fee-free open data** (free to use, issue and redistribute), covering all asset classes and offering uniqueness at multiple levels of granularity | ✅ / ⚠ | OMG's own FIGI material (omg.org/intro/OpenFIGI_and_OMG.pdf) and the OMG FIGI specification page (omg.org/spec/FIGI) | Sept 2026 | Read from OMG's material. **⚠ A Bloomberg press description of 2023 refers to the standard's auspices as "OMG and X9"**; this guide does not treat the X9 role as verified and does not state it as fact |
| 7 | The **LEI** is a 20-character code; the standard is **ISO 17442-1:2020** *Part 1: Assignment* (published 2020-08, last reviewed and confirmed **2026**), superseding **ISO 17442:2019**; **GLEIF** administers the global system, was **established in 2014 by the FSB**, is **overseen by the Regulatory Oversight Committee**, and LEIs are issued by **Local Operating Units**; the Global LEI Index is open and free | ✅ | GLEIF, "The Legal Entity Identifier (LEI)" and "This is GLEIF" (gleif.org); ISO catalogue entry for ISO 17442-1:2020 (iso.org/standard/78829.html) | Sept 2026 | Verified from GLEIF's own pages **and** the ISO catalogue entry, which also shows 17442:2019 withdrawn and the "-1:2020" part structure. **See row 8 for the discrepancy this creates with a sibling guide's citation** |
| 8 | The sibling [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §8.2 cites the LEI standard as "**ISO 17442**" | ⚠ discrepancy (minor) | ISO catalogue (iso.org/standard/78829.html — ISO **17442-1:2020**, and ISO 17442:2019 shown as withdrawn) | Sept 2026 | Not an error of substance — the ISO 17442 *family* is correct — but the **current edition is the Part 1:2020 assignment standard**, so documentation should cite ISO 17442-1:2020. Recorded, not edited (§15) |
| 9 | The **UPI** is **ISO 4914:2021**, published **2021-11-16**, developed under **ISO/TC 68/SC 8**, specifying an unambiguous scheme to identify **OTC derivative products reportable to trade repositories**, applicable at minimum to the CFI categories swaps, forwards, non-listed and complex listed options, and "others (miscellaneous)" | ✅ | ISO catalogue entry (iso.org/standard/80506.html) | Sept 2026 | Read directly from the ISO catalogue entry, including the abstract's category list |
| 10 | The **Derivatives Service Bureau (DSB)** has been designated the **sole service provider for the UPI system**, performing the role of issuer of UPI codes and operator of the UPI reference data library, with a **nomination by the Financial Stability Board** | ✅ | DSB (anna-dsb.com, including its UPI launch announcement of October 2023) | Sept 2026 | Read from the DSB's own site; the DSB describes itself as operating under ANNA ISO mandates, and ANNA's site cross-links the DSB |
| 11 | The **UTI** concept — a uniform global unique transaction identifier for OTC derivatives — is anchored in **CPMI-IOSCO's *Harmonisation of the Unique Transaction Identifier — Technical Guidance*, published February 2017**; the ISO standard number **ISO 23897** is widely cited for the UTI but **was not verified at ISO this pass** | ✅ / ⚠ | CPMI-IOSCO Technical Guidance (bis.org publication page); FSB publication notice (fsb.org, February 2017) | Sept 2026 | The **guidance and its date are verified**; the **standard number is flagged** ⚠ and is not asserted as verified in this guide (§15) |
| 12 | **MIC** is **ISO 10383**; the **Registration Authority is SWIFT (S.W.I.F.T. SC, La Hulpe, Belgium)**; MICs are registered at operating/exchange and market-segment level; the list is **published monthly on the second Monday** with modifications effective the **fourth Monday**; creation, maintenance and deactivation are **free of charge**; only representatives of the market organisation operating the venue may request changes; the list seen this pass was **published 14 September 2026** with modifications effective **28 September 2026** | ✅ | ISO 10383 MIC pages (iso20022.org/market-identifier-codes) and the MIC Registration Procedures PDF | Sept 2026 | Read directly; this is a **current-state verification with a publication date attached**, which is the standard this guide applies to any statement about an identifier's governance |
| 13 | **SWIFT** acts as Registration Authority for **ISO 9362 (BIC)**, **ISO 13616 (IBAN)** and **ISO 10383 (MIC)** | ✅ | SWIFT data standards page (swift.com/standards/data-standards) | Sept 2026 | SWIFT's own statement of its RA roles; used here only to corroborate the MIC RA |
| 14 | **ISO 15022** (the SWIFT MT securities message set: **MT564** corporate action notification, **MT566** confirmation) and **ISO 20022** (the **seev** family) carry corporate-action content; **DTCC** uses the ISO 20022 corporate-action format and converted to the SWIFT **SR2025** version on **23 November 2025**; SWIFT's MT→ISO 20022 coexistence for cross-border FI-to-FI payments ended **22 November 2025** | ✅ via sibling | [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §4.4 and its claims audit (dtcc.com ISO 20022 messaging specifications and SR2025 notice; swift.com FAQ) | Sept 2026 (sibling's verification pass; not re-opened here) | **Cross-referenced, not re-derived.** The seev message *numbers* (seev.031 / seev.036) are marked ⚠-secondary in the sibling guide and this guide does not assert them |
| 15 | **FRTB**: the BCBS *Minimum capital requirements for market risk* (**d457**) was **published 14 January 2019**, corrected **25 February 2019**, **effective 1 January 2022**, and is **integrated into the consolidated Basel Framework**; its internal models approach sets **separate capital requirements for non-modellable risk factors** | ✅ | BCBS publication page (bis.org/bcbs/publ/d457.htm) | Sept 2026 | Read directly from the BCBS page, including the effective-into-force date and the consolidated-framework note |
| 16 | The **risk factor eligibility test (RFET)**: a risk factor must pass the RFET to be classified modellable; **collateral reconciliations or valuations cannot be real prices**; a price is real only if it meets one of four criteria (own transaction; verifiable arm's-length transaction price; committed quote collected and verified through a third-party vendor, trading platform or exchange; or a vendor price meeting those conditions) (**MAR31.12**) | ✅ | Basel Framework, *Internal models approach: model requirements*, MAR31 (bis.org — chapter read directly, including its FAQs) | Sept 2026 | Quoted from the standard's consolidated text |
| 17 | The RFET thresholds: **at least 24 real price observations per year** (no more than one per day counted) with **no 90-day period with fewer than four** such observations, monitored monthly, **or at least 100 real price observations over the previous 12 months** (**MAR31.13**) | ✅ | Basel Framework MAR31.13, read directly | Sept 2026 | Numeric thresholds verified verbatim at source, including the footnotes on ES-calibration periods and the LIBOR-transition FAQ |
| 18 | Where a bank counts vendor real prices, the **vendor must communicate the number of corresponding real prices observed and the dates at which they have been observed**, provide identifier information to map real prices to risk factors, and be **subject to an audit of the validity of its pricing information** available to the supervisor on request (**MAR31.14**) | ✅ | Basel Framework MAR31.14, read directly | Sept 2026 | The most consequential market-data requirement in the guide; quoted from source |
| 19 | A bank must have **policies and procedures describing its mapping of real price observations to risk factors** and give supervisors sufficient information to judge its methodologies (**MAR31.15**); the standard sets out a **bucketing approach** to counting observations for curve/surface risk factors (**MAR31.16**) | ✅ | Basel Framework MAR31.15–31.16, read directly | Sept 2026 | Read at source; the bucketing detail is not reproduced here |
| 20 | **BCBS 239** numbers **fourteen principles in four sections**: **Section I = P1–P2**; **Section II = P3–P6**; **Section III = P7–P11**; **Section IV = P12–P14** | ✅ via repo-recorded verification | [Senior Data Platform Architect](../technology/senior_data_platform_architect_guide.md) — its BCBS 239 section records the correction and its claims audit carries the verification row against the standard's published PDF (bis.org) | Standard published January 2013; verification recorded in this repository | This guide uses the standard's grouping throughout |
| 21 | [Risk Data Aggregation](risk_data_aggregation_guide.md) **§2 is headed "The Governance Principles (P1–P5)"** (line 123) and its ToC repeats that label | ✅ as a description of that file; ❌ as a description of the standard | The file itself (heading and ToC read this pass); the standard's own sectioning per row 20 | Sept 2026 | **Recorded as a discrepancy, not inherited and not edited.** That guide's §2 body and its §1 numbering note do state the standard's own mapping, so the issue is the heading's label; see §11.2 |
| 22 | **ISO/IEC 9075-1:2011** (the SQL framework part) is **withdrawn**, revised by **9075-1:2016**, with **9075-1:2023** available as the new version; the family is developed by **ISO/IEC JTC 1/SC 32** | ✅ | ISO catalogue entry (iso.org/standard/53681.html) | Sept 2026 | Used only to place the SQL standard family's temporal-feature generation and its lifecycle; the **exact SQL:2011 feature names and clause text were not verified at source** ⚠ |
| 23 | **SQL:2011** introduced **system-versioned tables** (transaction time) and **application-time period tables** (valid time) | ⚠ | Stated as the recognised anchor; **not verified at ISO or in the standard's text this pass** (the ISO page opened was for the framework part, not Part 2 where the features are defined) | Sept 2026 | **Flagged.** This guide therefore uses the *concepts* (valid time / transaction time) as its working vocabulary and names SQL:2011 as the feature family's origin rather than quoting it as a citation (§6.1, §15) |
| 24 | **SCD Type 2** ("add a new row, keep the full history") is the data-warehousing pattern equivalent to a valid-time record; the repository's dimension guide covers **SCD Types 0–7** | ✅ | [Types of Dimensions in Data Warehousing](../technology/data/types_of_dimensions_data_warehousing.md) §3 (read this pass) | Sept 2026 | Used as the repo-internal anchor for the effective-dating pattern |
| 25 | **ICE**'s data business presents itself as **Fixed Income & Data Services**, with products including **ICE Consolidated Feed**, **ICE Consolidated History** (a cloud-based archive of historical tick data), **ICE Data API** and **ICE Global Network**, and US fixed-income evaluations provided through **ICE Data Pricing & Reference Data, LLC** | ✅ | ICE (ice.com/data-services) | Sept 2026 | Read from ICE's own site; **only the product names actually seen on that page are used** — no other ICE product name is asserted |
| 26 | **LSEG** presents a data catalogue spanning **1,000+ datasets**, **500+ global exchanges**, and lists a **Corporate Actions** data category, with delivery via direct feeds, APIs, cloud platforms and **LSEG Workspace** | ✅ | LSEG Data Catalogue (lseg.com/en/data-analytics/financial-data/reference-data/identifiers) | Sept 2026 | Read from LSEG's own site; used only as evidence of the current corporate identity and product naming |
| 27 | **LSEG**'s acquisition of **Refinitiv** completed **29 January 2021** at approximately **US$27bn** | ✅ via sibling | [Market Data Consumption](market_data_consumption_guide.md) §3 and its Primary Sources (LSEG press release) | Sibling's verification pass; not re-opened here | Cross-referenced; this guide states it only as the successor-entity fact behind RIC/SEDOL ownership and flags the RIC attribution separately (row 5) |
| 28 | The repository's coverage pattern: unbounded case-insensitive `isin` matches **356** markdown files; word-bounded it is **17**; `symbolog` **0**; `FIGI` **1**; word-bounded `RIC` **1**; "corporate action" **4**; "security master" **7**; `SEDOL` **4**; word-bounded `LEI` **17**; "market data" **99**; Bloomberg **60** | ✅ (provided as established fact in the task's dedup record; used as framing, not re-measured here) | The repository-wide coverage check recorded by the commissioning work (grep-based) | Sept 2026 | Used as the guide's motivation in §1.2; the **inference that 356 files are "about" securities identification is ❌ rejected** — the matches are dominated by the `-ising` substring, which is the point |
| 29 | ANNA was established in **1992** by 22 founding numbering agencies | ⚠ | An ANNA service-site page (annaservice.com/anna/whats.jsp) as returned by the search backend; the year was **not confirmed on a page opened directly** | Sept 2026 | **Flagged** and not relied upon; the guide cites only the ASB's "since 2001" collection date from ANNA's own page (row 2) |
| 30 | **ISO 10962** is the **CFI** classification standard, for which **ANNA** is the ISO-appointed **Registration Authority** | ⚠ / ✅ | ANNA's own service site statement of RA roles (as in row 1) | Sept 2026 | ANNA's RA role is read from ANNA's material; the **ISO 10962 number itself was not opened at iso.org this pass** ⚠ |
| 31 | The **Corporate Action Notification / Confirmation** operation-lifecycle facts, the **record date / ex-date** relationship, **mandatory vs voluntary**, and the fact that **US T+1 applies from 28 May 2024** | ✅ via sibling | [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §4.2, §4.3, §3.1 and its claims audit (SEC Release 34-96930 for T+1) | Sibling's verification pass; not re-opened here | Cross-referenced, not re-derived, per the boundary declared in §1.4 |
| 32 | **LSEG's Symbology page** states: the **RIC** is a market-level identifier for instruments and pricing sources **issued by LSEG**; **PermID** is issued by LSEG; the **LEI** is based on **ISO 17442**, with LSEG acting as an **LOU** under GLEIF requirements and LEI ROC-agreed principles; **SEDOL** codes are **issued at the country level for most asset classes** (equities, fixed income, funds, ETPs, indices, structured products, warrants, certificates, exchange traded derivatives, digital assets), **in operation for over 40 years**, covering **over 22 million active securities**, and **allocated at a more granular level than ISIN codes**; and **London Stock Exchange Plc is the sole issuer of ISIN, FISN and CFI codes for Great Britain, Jersey, Guernsey and the Isle of Man** | ✅ (with one internal ⚠) | LSEG Symbology page (lseg.com/en/data-analytics/market-data/data-analytics-pricing/data-symbology) | 22 Sept 2026 | Read directly from LSEG's own page. **⚠ LSEG's two pages disagree on the age of the SEDOL system** — over 30 years on the SEDOL Masterfile page versus over 40 years here — recorded as an unresolved inconsistency rather than smoothed over (§3.1, row 4) |

---

## 15. What Could Not Be Verified

**Every ⚠ in §14, restated plainly, with what specifically could not be confirmed.** This list is deliberately explicit: it is the difference between a guide that states its limits and one that launders uncertainty into assertion.

- **The RIC — flagged in the draft, resolved in the parent's verification pass, and kept here as the record of the flag.** The draft flagged the RIC's issuing body because no vendor page covering it had been opened. On re-check, **LSEG's own Symbology page states that the RIC is a market-level identifier for instruments and pricing sources issued by LSEG** ✅ (§14 row 5, checked 22 September 2026). What remains unverified is narrower: the RIC's **internal structure and port convention**, which LSEG's public material does not publish, so the guide treats the syntax as vendor-documented rather than standardised. Nothing else about the RIC is outstanding.
- **The ISO standard number for the UTI (ISO 23897).** The number is in wide circulation and is consistent across secondary sources, but it **was not opened at the ISO catalogue** in this pass. The *concept* and its governance are verified from CPMI-IOSCO's February 2017 Technical Guidance (§3.1, §11.3, row 11); the number is flagged.
- **The exact SQL:2011 feature names and clause text.** The ISO page opened was for the **framework** part of the SQL standard (ISO/IEC 9075-1:2011), not Part 2, where system-versioned and application-time period tables are defined. The feature *concepts* are used as this guide's vocabulary and SQL:2011 is named as their origin; **a firm needing a contractual citation must verify the feature names and clauses in the published standard** (§6.1, row 23).
- **The ISO number and edition for the CFI standard (ISO 10962).** ANNA's Registration Authority role for CFI is taken from ANNA's own material; the standard's number and current edition were not opened at ISO (§3.1, row 30).
- **FRTB's jurisdictional implementation dates and any national modifications.** The BCBS text, its publication and correction dates, its effective date, the consolidated-framework chapter dates and the RFET requirements are all verified at source. **Which supervisors applied the framework when, and with what local modifications, was not verified** (§11.1, row 15).
- **ANNA's founding year (1992).** Returned by the search backend from ANNA's service site but not confirmed on a page opened directly; not relied on anywhere in the guide (§14, row 29).
- **The "OMG and X9" auspices for FIGI.** The OMG material and the OMG specification page were read; a press description additionally names X9 as a co-auspice. This guide states the OMG role as verified and **does not assert the X9 role** (§3.1, row 6).
- **SEDOL's "issued since 1979" history.** The sibling guide flags it as secondary and this guide repeats the flag rather than the claim; LSEG's own "over 30 years" is marketing copy (§14, row 4).
- **The corporate-action message standards (ISO 15022 / ISO 20022 seev).** Cited from [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §4.4, which verified them at source (including the DTCC SR2025 conversion date of 23 November 2025 and the end of payment-message coexistence on 22 November 2025). **This guide did not re-open those sources**; the specific seev message numbers remain ⚠-secondary in the sibling guide and are not asserted here (§5.4, row 14).
- **The ISO 20022 Registration Authority.** An attempt to open the ISO 20022 site's landing page this pass failed at the extraction layer, so the identity of the ISO 20022 registration authority is **not stated in this guide at all** — no claim is made.
- **DTCC's corporate-actions content.** The DTCC corporate-actions URL attempted this pass returned a **404**; the DTCC-related claims in §5.4 come from the sibling guide's verified citation rather than from a page opened here.
- **Vendor product names beyond those read at source.** Only product names actually seen on a vendor's own page this pass are used: ICE's named products (§14, row 25), LSEG's data-catalogue framing and **LSEG Workspace** (§14, row 26), and the SEDOL Masterfile (§14, row 4). **No Bloomberg product name, no FactSet product name and no additional LSEG product name is asserted** — deliberately, because a vendor product named from memory is exactly the class of error this guide's source discipline exists to prevent.
- **A tool limitation, recorded honestly.** The **web search backend returned empty result sets on multiple attempts** during this pass (both in batches and singly), which is recorded as a **limitation of the tooling, not as evidence of absence** of the material searched for. The verification work was therefore done predominantly by **direct extraction of primary pages** (standards bodies, registration authorities, the DSB, GLEIF, the BCBS/Basel Framework, LSEG, ICE, the acquirer's press release and its SEC filing), and the guide says so where it matters.
- **Not verified because it is not verifiable:** any figure, date, quantity or cost in §12. Those are **fictional and illustrative by construction** and are not claims about anything.

---

## 16. The Glossary, the Cross-References and the Closing Summary

### 16.1 The Glossary

**Adjustment** — restating a historical price or quantity series so that it is continuous across a corporate action; a *view* computed from stored raw values and stored factors, never a mutation of the observation (§5.6).
**ANNA** — the Association of National Numbering Agencies; the ISO-appointed Registration Authority for ISIN (and CFI); the ANNA Service Bureau aggregates from 120+ NNAs and is operated by CUSIP Global Services and SIX Financial Information (§3.1).
**Announcement version** — the state of a corporate action's terms at a point in time; terms are amended, so an event has versions, each with its own knowledge window (§5.1, §8.4).
**As-of** — a reference to a point in time that, to be unambiguous, must specify *which* axis (valid time or knowledge time) — and ideally both (§6.1, §6.6).
**Bitemporal** — recording a fact with both its valid time and its transaction time; the shape of the golden record (§8.2).
**Break** — a difference that crossed an agreed tolerance and was raised as an item with an owner and an age (§4.4).
**Break queue** — the operating artefact where breaks live with owners, ages and escalation thresholds; its health is measured by age distribution and root-cause recording, not by total count (§10.2).
**CFI** — Classification of Financial Instruments; a classification code (not an identifier), for which ANNA is the ISO-appointed Registration Authority (§3.2).
**Corporate-action lifecycle** — announcement → terms confirmation/amendments → ex-date and record date → election window → allocation/entitlement → payment/delivery → post-event reconciliation, each with a control point (§5.1); the operational processing is [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §4.
**Corporate-action ledger** — the event store that *drives* adjustment: events, versions and effects (quantity factor, price factor, basis), with effective dates (§8.4).
**Coverage break** — a difference of *presence* (an instrument, field or event one source has and another lacks); its own break class, because absence is not a "difference" (§4.3) and because missing rows silently shrink aggregates (§3.3).
**Crosswalk** — the mapping between an instrument's identifiers; correct only when effective-dated on both axes and never overwritten (§3.3, §8.1).
**Cross-vendor reconciliation** — the control comparing two providers' answers for the same instrument and field; a *data* control, distinct from accounting/balance reconciliation ([Investment Portfolio Operations](investment_portfolio_operations_guide.md) §5) (§4, §1.4).
**CUSIP** — the US/Canada national numbering system; issued by CUSIP Global Services, owned by FactSet since 1 March 2022, stewarded in partnership with the American Bankers Association (§3.1).
**Data contract** — the agreement at each interface covering identity, field semantics, time semantics, quality expectations and the change protocol (§9.3).
**Effective-dated** — carrying validity dates; applied here to crosswalk rows, identifiers, tolerances, conventions and universes.
**Ex-date / record date** — the date from which a security trades without an entitlement, and the date on which holders of record are entitled; their relationship is a market convention tied to the settlement cycle (§5.1).
**FIGI** — Financial Instrument Global Identifier; an OMG standard, fee-free open data, issued/distributed by Bloomberg L.P. as Registration Authority, with Bloomberg and Kaiko as Certified Providers; unique at multiple levels of granularity (§3.2).
**FRTB** — the Basel Committee's market-risk capital framework (d457, published 14 January 2019, effective 1 January 2022), including the internal models approach, the risk factor eligibility test and non-modellable risk factors (§11.1).
**Golden copy / golden source** — the firm's designated authoritative record for a field or domain; per-field sourcing raises per-field quality and lowers whole-record coherence unless the §4.5 mitigations are in place (§4.5).
**Identifier stack (symbology)** — the set of identifiers a firm carries for one instrument; the anchor choice determines what the firm can ever join on (§3.5).
**ISIN** — ISO 6166; a 12-character global securities identifier allocated by NNAs under ANNA's Registration Authority; identifies the security, not the venue, and is not issued for every instrument (§3.2).
**ISO 10383 (MIC)** — the Market Identifier Code standard; Registration Authority SWIFT; monthly publication (§3.2).
**ISO 15022 / ISO 20022** — the securities message standards carrying corporate-action content (MT564/MT566 and the seev family); cross-referenced to [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §4.4 (§5.4).
**ISO 17442-1:2020** — the LEI assignment standard (Part 1), superseding ISO 17442:2019 (§3.2).
**ISO 4914:2021** — the UPI standard for OTC derivative products reportable to trade repositories (§3.2).
**Knowledge time (transaction time)** — when the firm held a fact; the axis that makes restatement detectable (§6.1).
**LEI** — the 20-character legal-entity identifier under the GLEIF-administered global system, overseen by the Regulatory Oversight Committee, issued by LOUs; identifies the entity, never the instrument (§3.2).
**Mandatory / voluntary** — whether an event applies to all holders with no election or requires one; the data-model distinction is bulk restatement versus conditional state change (§5.2).
**NNA** — National Numbering Agency; the body that allocates identifiers within a national prefix under ANNA's framework (§3.3).
**Override** — the recorded decision to accept a value other than the default source's; valid only with reason, approver and expiry (§4.4, §13).
**Point-in-time (PIT)** — a value or universe as it was knowable on a given knowledge date, rather than as it is today (§6.2).
**Raw store** — the immutable, append-only landing of vendor deliveries as received, with receipt timestamps and checksums; the evidence layer (§8.3).
**RIC** — a market-level vendor identifier for instruments and pricing sources **issued by LSEG** ✅; proprietary, encoding the vendor's view of the listing chain and venue, and therefore the worst possible primary key for a firm's own data model (§3.2).
**Restatement** — the replacement of a previously published historical value; *silent* restatement carries no flag and is detected only by diffing retained versions (§6.3).
**RFET (risk factor eligibility test)** — the test in MAR31 determining whether a risk factor is modellable, based on counts of *real price* observations (24 per year with a 90-day/four-observation floor, or 100 over twelve months) (§11.1).
**Real price** — a price meeting one of MAR31.12's four criteria; collateral reconciliations and valuations explicitly do not qualify (§11.1).
**Survivorship bias** — the selection effect created by using a current instrument universe for historical analysis, excluding instruments that did not survive (§6.4).
**Tolerance** — the pre-agreed, per-field-type rule determining whether a difference is a break; approved by the consuming business, not the data team alone (§4.3).
**Two axes** — valid time and knowledge time; storing only one makes the other class of question unanswerable (§6.1).
**UPI / UTI** — the OTC derivative *product* identifier (ISO 4914:2021, issued by the DSB) and the *transaction* identifier (anchored in CPMI-IOSCO's February 2017 guidance) (§3.2).
**Valid time (effective time)** — the period in the world that a fact describes (§6.1).
**Vintage** — the version of a dataset as known on a knowledge date; the third query shape of §6.6.
**Look-ahead bias** — using a value not knowable at the modelled decision time; in market data, introduced by adjustment with hindsight, by restatement, and by universe selection (§6.5).

### 16.2 Cross-References

- **[Capital Markets Architecture](capital_markets_architecture_guide.md) §5** — the market-data taxonomy, the feed architecture, the reference-data/golden-copy idea, the data-type table and the governance summary. Cited for the taxonomy; this guide covers the reconciliation mechanism, the corporate-action lifecycle and the temporal model that §5 does not reach.
- **[Market Data Consumption](market_data_consumption_guide.md)** — the value chain venue → feed → vendor → platform → decision; §2 the consolidated tape and its providers; §3 the vendors; §4.6 corporate-action and reference-data feeds; §8.4 historical data and time-series stores; §9 the data-management layer (golden source §9.1, entitlements §9.2, governance §9.3). Cited for the producer/consumer map and the feed taxonomy.
- **[Investment Portfolio Operations](investment_portfolio_operations_guide.md)** — §4 corporate-action *operations* (types, mandatory vs voluntary, lifecycle, MT564/MT566 and ISO 20022 seev flows, elections/claims/tax); §5 accounting/balance reconciliation (positions, cash, NAV) — **explicitly a different control from this guide's cross-vendor data reconciliation**; §8 reference and market data (security master §8.1, identifiers §8.2, feeds §8.3, governance §8.4).
- **[Risk Data Aggregation](risk_data_aggregation_guide.md)** — BCBS 239 itself, risk aggregation, lineage; cited with the standard's own section grouping and with the §2 heading discrepancy recorded (not edited) at §11.2.
- **[FIX Protocol](fix_protocol_guide.md)** — the wire protocol and its market-data messages; the transport layer beneath the ingestion stage of §9.1.
- **[Online Investment Trading Platforms](online_investment_trading_platforms_guide.md)** — broker platforms as a consumer class (light cross-reference).
- **[Transaction Foundation Model](transaction_foundation_model_guide.md)** — a model built on transaction data where the point-in-time question recurs.
- **[Basel Regulatory Capital](basel_regulatory_capital_guide.md)** and **[Enterprise Risk Management](enterprise_risk_management_guide.md)** — the capital-framework and model-governance homes for §11.1.
- **[Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md)**, **[Treasury & ALM](treasury_alm_guide.md)**, **[MAS Regulations & Guidelines](mas_regulations_guidelines_guide.md)** — the reporting chains and supervisory context for §11.3–§11.4.
- **[Deterministic Engineering](../technology/deterministic_engineering_guide.md)** — reproducibility, which backtest determinism depends on; §6.5's companion.
- **[Senior Data Platform Architect](../technology/senior_data_platform_architect_guide.md)** — the platform mandate and the recorded BCBS 239 grouping correction this guide cites.
- **The `technology/data/` shelf** — [Data Governance Framework](../technology/data/data_governance_framework.md), [Data Lineage Tools](../technology/data/data_lineage_tools.md), [Data Fabric](../technology/data/data_fabric_guide.md), [Enterprise Data Platforms](../technology/data/enterprise_data_platforms_guide.md), [Data Pipeline Versioning](../technology/data/data_pipeline_versioning.md), [Backfill Data Engineering](../technology/data/backfill_data_engineering.md), [Cloud Object Storage Lakehouse](../technology/data/cloud_object_storage_lakehouse_guide.md), [On-Prem Lakehouse](../technology/data/on_prem_lakehouse_guide.md), [Delta Lake vs Iceberg](../technology/data/delta_lake_vs_iceberg.md), [Data Vault 2 Modelling](../technology/data/data_vault_2_modeling.md), [Types of Dimensions in Data Warehousing](../technology/data/types_of_dimensions_data_warehousing.md) — cited for the storage, lineage, versioning and dimension-modelling mechanics that §8 and §9 require but do not re-derive.

### 16.3 The Closing Summary

**The guide opened with an identity, a decoder and a boundary, and it ends with a single question.** Along the way it verified who issues each identifier today and what each one cannot join on (§3); treated reconciliation as a control with per-field tolerances, an owned break queue and a storage precondition (§4); read corporate actions as a lifecycle whose data-model consequence is a retroactive edit that must be recorded as an event rather than applied as an overwrite (§5); and used the two time axes to show why the latest value is not the truth for a past date, and why history you did not record cannot be reconstructed at all (§6). It put the three problems back together as one (§7), sketched the five structures that make them answerable (§8), layered the flow and named the controls (§9), described the operating model without which none of it is a control (§10), and set the regulatory frame — with FRTB's risk factor eligibility test verified at the Basel text as the one place a supervisor has written "what did you know, and when" into the capital rules (§11). The worked example exercised all three problems in one narrative: a static-attribute break that was a corporate action mid-application, exposed by a crosswalk with no dates, which in turn exposed a fourteen-month-old unrecorded vintage feeding a model (§12).

**What a practitioner should take away, in four sentences.** Choose your anchor identifier for the questions you will need to answer, and effective-date everything that maps or changes. Record the vendor's raw answer, its receipt time and its own as-of date, because you cannot investigate a disagreement you did not record. Store raw values and store the factors, and derive the adjusted series under a named, approved, change-controlled convention — never in place. And accept the hard limit honestly: the only remedy for history you did not record is to begin recording, which makes this a storage and governance decision rather than a technical one.

**The three problems are one problem.** Identity that changes, sources that disagree, and time that erodes are three faces of a single question — for this fact about this instrument, what was known, by whom, and when — and the discipline of answering it is what determines whether a number the firm publishes today can still be explained a year from now. Everything in this guide reduces to that, and so does the title.

**what you knew, and when you knew it.**

