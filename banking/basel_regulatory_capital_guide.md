# Basel III / Basel IV Regulatory Capital: The Capital-Framework Deep-Dive — A Comprehensive Guide

*The dedicated deep-dive on regulatory capital: how the Basel framework defines capital, sizes it against risk, and anchors the banking system — from the 1988 Accord to the Basel III finalisation ("Basel IV") and its implementation in the EU, UK, US and Singapore. This is the regulatory-capital anchor of the bank-systems series: it cross-references the [Treasury & ALM Guide](treasury_alm_guide.md) (the *liquidity* side of Basel III — the LCR/NSFR — the two guides together read as the full Basel III/IV picture), the [Risk Management Models Guide](risk_management_models_guide.md) (the VaR/FRTB and IRB model machinery), the [Capital Markets Architecture Guide](capital_markets_architecture_guide.md) (the FRTB/market-risk angle), the [Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md) (§2, the regulatory-context frame), the [Core Banking Systems Guide](core_banking_systems_guide.md) (the RWA-data angle), the [Banks in Singapore Guide](banks_in_singapore_guide.md) (the MAS implementation angle), the [Crédit Agricole Software Systems Guide](credit_agricole_software_systems_guide.md) (the Cymbal Bank context used in the worked example), the [RegTech Guide](regtech_guide.md) (the reporting angle), the [Universal Banking Model Guide](universal_banking_model_guide.md), the [Singapore Fintech & Payments Guide](singapore_fintech_payments_guide.md), the [Risk Data Aggregation Guide](../technology/risk_data_aggregation_guide.md) (the BCBS 239 data side), the [Business Case Development Guide](../management/business_case_development_guide.md), and the [MBA Body of Knowledge Guide](../management/mba_body_of_knowledge_guide.md) (the finance vocabulary). Every ratio and date in this guide was verified against primary or authoritative secondary sources in §11; anything that could not be verified is explicitly flagged — the verification convention is enforced line by line.*

> **Context:** Banking / Risk & Regulatory — Basel History (I 1988, II 2004, III 2010–2017), Capital Instruments (CET1, AT1/CoCos, Tier 2), Minimums and Buffers (4.5% CET1, 2.5% CCB, CCyB, G-SIB/D-SIB), RWA (credit SA/IRB, market FRTB, operational SMA, CVA), Leverage Ratio (3%), Basel IV (Dec 2017 finalisation, 72.5% output floor), Implementation (EU CRR3 2025, UK Basel 3.1, US endgame, MAS Notice 637), Pillar 2 (ICAAP/SREP, TLAC/MREL), Cymbal Bank worked example.

**How to read this guide.** The ten sections form a complete arc: the history (§1), the instruments (§2), the minimums and buffers (§3), the risk weighting (§4), the leverage backstop (§5), the Basel IV finalisation (§6), the implementation (§7), the Pillar 2 overlay (§8), a worked Cymbal Bank capital calculation (§9), and a one-page summary (§10), closing with the verification record (§11), the glossary (§12), and the series cross-references (§13). Readers who want the *rulebook* should read §1–§3 and §6; readers who want the *calculation* should read §4, §5, and §9; readers who want the *implementation reality* should read §7 and §8; readers who want the *architecture* should read §4 and §13. Every section stands alone, with cross-references where the series has more depth.

## Table of Contents

1. [The Basel History](#1-the-basel-history)
2. [The Capital Instruments](#2-the-capital-instruments)
3. [The Minimums and the Buffers](#3-the-minimums-and-the-buffers)
4. [The RWA: Risk-Weighted Assets](#4-the-rwa-risk-weighted-assets)
5. [The Leverage Ratio](#5-the-leverage-ratio)
6. [The Basel IV: The 2017 Finalisation](#6-the-basel-iv-the-2017-finalisation)
7. [The Implementation](#7-the-implementation)
8. [The Pillar 2](#8-the-pillar-2)
9. [The Worked Example](#9-the-worked-example-a-capital-calculation-for-a-cymbal-bank)
10. [The Summary: One Page](#10-the-summary-one-page)
11. [Verification Notes and Sources](#11-verification-notes-and-sources)
12. [Glossary](#12-glossary)
13. [Cross-References in This Series](#13-cross-references-in-this-series)

---

## 1. The Basel History

### 1.1 Why Capital, and Why Basel

A bank's **regulatory capital** is the equity and quasi-equity cushion that absorbs losses before depositors, creditors, and — in a crisis — taxpayers do. The Basel Committee on Banking Supervision (BCBS), the standard-setting body of the G10/global supervisors hosted at the Bank for International Settlements (BIS) in Basel, has written the rulebook that every major jurisdiction transposes into law. The framework's core arithmetic is simple: **capital ÷ risk-weighted assets (RWA) ≥ minimum ratio**, where the sophistication lies entirely in how "capital" and "risk-weighted assets" are defined. Every reform since 1988 has been a battle over those two definitions.

### 1.2 Basel I (1988): The 8 Percent

The first Basel Accord — *International Convergence of Capital Measurement and Capital Standards* — was published in **July 1988** and implemented in G10 countries through the early 1990s. Verified: it set the first global minimum capital requirement of **8% of risk-weighted assets** (the "Cooke ratio", named after the Committee's chairman Peter Cooke). Its design was deliberately crude:

- **Risk weights were flat and few**: 0% (cash, OECD-government debt), 20% (interbank), 50% (residential mortgages), 100% (everything else, notably corporate loans). The entire risk spectrum of a commercial loan book collapsed into one bucket.
- **Capital had two tiers**: Tier 1 (equity and disclosed reserves, "core" capital) and Tier 2 (undisclosed reserves, subordinated debt, hybrid instruments). Tier 1 had to be at least 50% of total capital — i.e., an effective minimum Tier 1 of 4%.
- **Only credit risk was in scope.** The 1996 Amendment (*to Incorporate Market Risks*) added a market-risk charge (the standardised measurement method or internal models — the origin of the 99%/10-day VaR framework), but operational risk was not charged at all.

Basel I's achievement was a common language and a floor; its failure was that the floor was not risk-sensitive. A loan to a triple-A sovereign and a loan to a distressed start-up both attracted 100%. The arbitrage was obvious and banks exploited it — securitisation was the most notorious channel (moving 100%-weighted assets off the balance sheet). Basel I also had no mechanism for supervisory review and no disclosure discipline.

### 1.3 Basel II (2004): The Three Pillars

Basel II — *International Convergence of Capital Measurement and Capital Standards: A Revised Framework* — was published in **June 2004** (verified, BIS document bcbs128; a compilation including the 1988 Accord's unrevised elements, the 1996 Amendment and the November 2005 revisions exists as bcbs128's companion). It restructured the whole edifice around **three pillars** (verified):

- **Pillar 1 — Minimum capital requirements.** Risk weights became genuinely risk-sensitive. For **credit risk**: the **Standardised Approach (SA)** (external-ratings-driven, more granular weights) or the **Internal Ratings-Based (IRB) approach** (banks' own PD/LGD/EAD models, subject to supervisory approval — the "use test"). For **market risk**: standardised or internal-models approaches carried over from the 1996 amendment. For the first time, **operational risk** attracted a capital charge (Basic Indicator Approach, Standardised Approach, or the Advanced Measurement Approach — AMA, i.e. internal loss-distribution models).
- **Pillar 2 — Supervisory review.** The supervisor reviews the bank's own risk assessment (which became the **ICAAP**) and can impose additional capital — the **SREP** family that still governs today (§8).
- **Pillar 3 — Market discipline.** Public disclosure of risk, capital, and RWA so that markets could price banks on their true riskiness.

Basel II was the intellectual high-water mark of internal models: the IRB approach let the largest banks size capital from their own validated models. But the 2007–2009 crisis exposed the flaws — model optimism, procyclicality, the off-balance-sheet blind spot (conduits and SIVs sat outside RWA), and the total absence of a liquidity requirement. The crisis also revealed that Pillar 2 had not been used forcefully enough. Basel II was effectively superseded before it was fully implemented.

### 1.4 Basel III (2010): The GFC Response

Basel III — *Basel III: A global regulatory framework for more resilient banks and banking systems* — was published by the BCBS in **December 2010** (verified, BIS bcbs189; supplemented through 2017). It was the direct response to the Global Financial Crisis (GFC), and it attacked every failure Basel II had left standing:

- **More and better capital.** The minimum **CET1 ratio was raised from 2% to 4.5%** (verified), Tier 1 to 6%, and total capital stayed at 8%. Capital quality was redefined: deductions (goodwill, intangibles, DTAs, MSRs) now bite into CET1; hybrid instruments were reclassified; the **capital conservation buffer (CCB) of 2.5%** of CET1 (verified) sits on top of the minimums, so the *combined* CET1 requirement is 7.0% before any distribution restrictions kick in.
- **Macroprudential buffers.** The **countercyclical capital buffer (CCyB)** of 0–2.5% (verified; jurisdictions can set it higher in stress) and the **G-SIB buffer** (1–3.5% by bucket — verified) plus domestic D-SIB buffers (§3).
- **The leverage ratio.** A simple, non-risk-based backstop — Tier 1 ÷ leverage exposure ≥ **3%** (verified, finalised January 2014, BIS bcbs270) — so that RWA games could no longer hide true balance-sheet size (§5).
- **Liquidity.** For the first time, quantitative liquidity standards: the **LCR** (30-day survival, 100% minimum, finalised 2013, bcbs238) and the **NSFR** (one-year stable funding, 100% minimum, finalised 2014, bcbs295). The liquidity side is the subject of the dedicated sibling [Treasury & ALM Guide](treasury_alm_guide.md) — this guide covers the capital side, and the two together are the full Basel III picture.
- **G-SIB designation** (from 2011) and the resolution agenda (TLAC, §8).

Phase-in ran from 2013 to 2019 (CET1 3.5% from 2013, 4.5% from 2015; buffers fully loaded from 2019), which is why "fully loaded Basel III" was the disclosure language of the late 2010s.

### 1.5 "Basel IV" (2017): The Finalisation

The BCBS does not use the term "Basel IV". What the market calls Basel IV is the package titled *Basel III: Finalising post-crisis reforms*, published on **7 December 2017** (verified, BIS bcbs424) — the completion of the post-crisis agenda. Its three engines: the **output floor** (72.5% of standardised RWA — verified), the **revision of the credit-risk frameworks** (IRB restrictions and parameter floors), and the **replacement of the operational-risk AMA with the Standardised Measurement Approach (SMA)** — with the **CVA framework** revised alongside. The **market-risk piece, the FRTB**, was finalised separately in **January 2019** (verified, BIS d457; effective 1 January 2022). §6 is the dedicated treatment; the implementation calendar is §7.

### 1.6 The History Table

| Accord | Year | Essence |
|---|---|---|
| **Basel I** | **1988** (July) | First global standard: **8% minimum capital ÷ RWA**; flat risk weights (0/20/50/100%); Tier 1 + Tier 2; credit risk only (1996 amendment: market risk) |
| **Basel II** | **2004** (June) | **Three pillars**: Pillar 1 risk-sensitive minimums (SA and IRB for credit; op-risk charge introduced), Pillar 2 supervisory review (ICAAP/SREP), Pillar 3 market discipline (disclosure) |
| **Basel III** | **2010** (December) | **GFC response**: CET1 minimum 4.5%, Tier 1 6%, total 8%; CCB 2.5%; CCyB 0–2.5%; G-SIB buffers; leverage ratio 3%; LCR and NSFR; phase-in 2013–2019 |
| **Basel IV (Basel III finalisation)** | **2017** (7 December) | **Output floor 72.5% of standardised RWA**; revised credit-risk SA and IRB restrictions; CVA revision; SMA replaces AMA for operational risk; FRTB finalised January 2019 |
| **Implementation wave** | **2023–2028** | Jurisdictional lawmaking: EU CRR3 (Jan 2025), UK Basel 3.1 (Jan 2027 after delays), US re-proposal (2026, pending), MAS Notice 637 (Jul 2024) — §7 |

### 1.7 The Basel III Phase-In (2013–2019)

Basel III did not arrive at once: the BCBS set a phase-in so that banks could build capital without a credit crunch (verified, the phase-in arrangements in bcbs189 and the liquidity standards). The milestones that matter:

| Requirement | 2013 | 2014 | 2015 | 2016 | 2017 | 2018 | 2019 |
|---|---|---|---|---|---|---|---|
| **CET1 minimum** | 3.5% | 4.0% | **4.5%** | 4.5% | 4.5% | 4.5% | 4.5% |
| **Capital conservation buffer** | — | — | — | 0.625% | 1.25% | 1.875% | **2.5%** |
| **CCyB** | — | — | — | phase-in to 2.5% | | | fully loaded |
| **Leverage ratio** | monitoring | monitoring | disclosure | | | **binding (3%)** | |
| **LCR** | 60% | 100% from 2015–2019 ramp | | | | | **100%** |
| **NSFR** | monitoring | | | | | **binding (100%)** | |
| **G-SIB buffer** | — | — | — | **from 2016**, by bucket | | | fully loaded |

The practical effect: "fully loaded Basel III" became the disclosure language of 2019, and the ratios banks quote today (CET1 12–16%) are measured against a *fully-loaded* stack — minimums plus buffers — not the 2010 starting point.

### 1.8 The Failure-and-Fix Map

Each GFC failure maps to a specific Basel III/IV reform — the fastest way to remember the framework is to remember what it was fixing (verified against the bcbs189 objectives):

| GFC failure | Reform that fixed it |
|---|---|
| Too little capital, and much of it low-quality hybrids | CET1 4.5% minimum + strict CET1 definition and deductions (§2) |
| Capital too small to be *usable* in a downturn | Capital conservation buffer 2.5% (§3) |
| Procyclicality — capital vanished exactly when credit was needed | Countercyclical buffer 0–2.5% (§3) |
| RWA games — models and securitisation compressed risk weights | Leverage ratio 3% (2014); output floor 72.5% (2017, §6) |
| Model optimism and RWA variability | IRB restrictions and parameter floors (§6) |
| Liquidity freeze (2008's true killer) | LCR and NSFR — see the [Treasury & ALM Guide](treasury_alm_guide.md) |
| Too-big-to-fail | G-SIB buffers, TLAC, resolution regimes (§3, §8) |
| Opacity — nobody could compare banks | Pillar 3 disclosure revisions (§6) |

### 1.9 The Basel Committee Itself

The BCBS is the standard-setter, not a regulator: a forum of ~45 central banks and supervisory authorities (its membership expanded from the original G10), hosted by the BIS in Basel, deciding by consensus. It has no treaty power — every standard becomes binding only when a jurisdiction transposes it, which is why the same Basel III finalisation yields CRR3 in the EU, Basel 3.1 in the UK, the (unfinished) US endgame, and MAS Notice 637 in Singapore (§7). The Committee monitors implementation through its Regulatory Consistency Assessment Programme (RCAP); the gaps between member jurisdictions' rules are the daily reality of a global bank's capital team.

### 1.10 The Road Between the Accords

Three way-stations matter for the story. **1996**: the *Amendment to the Capital Accord to Incorporate Market Risks* added the first market-risk charge (standardised or internal-models — the 99%/10-day VaR), the first time internal models were allowed to size regulatory capital. **1999–2004**: the Basel II consultations ran through the Asian crisis and the LTCM collapse, which taught the Committee that off-balance-sheet and market risks could kill a bank faster than its credit book. **2005–2008**: the Basel II framework was finalised and transposed (the EU's CRD in 2006–07; the US held out for large banks), only to be overtaken by the GFC — a crisis whose proximate cause was not too little *regulatory* capital on the books but too little *real* capital, too much leverage hidden off-balance-sheet, and zero liquidity backstop. Basel III was written between 2010 and 2017 with those three gaps as its table of contents — which is why the framework is best remembered as a *map of the crisis* (§1.8).

### 1.11 Reading a Bank's Capital Disclosure

Every quarter, a bank's results release and its **Pillar 3 report** present the capital picture in a predictable shape — worth learning to read, because the same skeleton appears in the [DBS Bank Guide](dbs_bank_guide.md), the [UOB](uob_software_systems_guide.md)/[OCBC](ocbc_software_systems_guide.md) guides, the [HSBC Guide](hsbc_software_systems_guide.md), the [Bank of America Guide](bank_of_america_software_systems_guide.md), and the [Crédit Agricole Guide](credit_agricole_software_systems_guide.md):

1. **The headline ratios** — CET1, Tier 1, total capital, leverage ratio — quoted *fully loaded* (all phase-ins complete) and often also *transitional*.
2. **The RWA bridge** — RWA by risk type (credit, market, operational, CVA) and the movement from the prior quarter (book growth, model changes, methodology changes, FX).
3. **The buffer position** — CET1 ratio vs the *combined buffer requirement* (minimum + CCB + CCyB + G-SIB/D-SIB), with the "MDA distance" — how much buffer headroom exists before distributions are constrained.
4. **The leverage ratio** and its exposure-measure build-up (especially telling for CIBs).
5. **Pillar 2 (EU/UK)** — P2R and P2G, plus TLAC/MREL position for G-SIBs (or, in SG, the MAS minimums of §7.4).

The analyst's trick: the *difference* between a bank's CET1 and its combined requirement plus P2 is its true distribution and resilience capacity — a 12.6% CET1 at a 9.5% combined-plus-P2 line is "comfortable"; the same 12.6% against an 11.5% line is "tight". The framework of this guide is precisely the vocabulary for that sentence.

---

## 2. The Capital Instruments

### 2.1 The Three-Tier Hierarchy

Basel III defines three tiers of capital, ranked by loss-absorption quality. The ranking is not cosmetic: the tier determines (a) what the instrument can absorb (going-concern losses vs resolution losses), (b) the conditions attached (perpetuity, coupons, triggers), and (c) the ratio it feeds (CET1, Tier 1, or total capital). The Basel III definition is in bcbs189 (Dec 2010) and the eligibility criteria in its annexes; the EU transposition is CRR articles 26–62.

**CET1 — Common Equity Tier 1** (verified). The highest-quality capital: ordinary shares, share premium, retained earnings, other disclosed reserves, and (subject to limits) minority interests — *minus* a long list of deductions: goodwill and other intangibles, deferred tax assets that rely on future profitability, mortgage-servicing rights above thresholds, and certain holdings of financial-sector capital. CET1 is permanent, fully subordinated, absorbs losses on a going-concern basis (it is the first line), has no trigger and no coupon obligation. This is the capital that keeps the bank alive.

**AT1 — Additional Tier 1** (verified). Perpetual, deeply subordinated instruments that absorb losses on a going-concern basis at a pre-specified trigger. The workhorses are **contingent convertibles — CoCos** (verified): bonds that *convert to equity* or are *written down* when the issuer's CET1 ratio falls below a trigger (the common EU trigger is 5.125% CET1, and always at the point of non-viability — PONV), plus a statutory write-down mechanism so that a failing bank recapitalises itself without taxpayer money. Features: no maturity date (perpetual), callable by the issuer only after ≥5 years (and only with supervisory approval), non-cumulative coupons payable at the issuer's discretion (cancellable without triggering default — the hallmark of the March 2023 Credit Suisse write-down, where $17bn of AT1 was written to zero while equity retained some value). AT1 feeds the Tier 1 ratio.

**Tier 2** (verified). The gone-concern layer: subordinated debt with a minimum original maturity of 5 years (amortising in the final five years), plus eligible revaluation reserves and (in some jurisdictions) subordinated loan capital. Tier 2 absorbs losses *in resolution* — after CET1 and AT1 are exhausted — and is the cushion that protects senior creditors and depositors. Tier 2 feeds the total capital ratio but not Tier 1.

### 2.2 The Instruments Table

| Instrument | Tier | Perpetuity | Coupon | Loss absorption | Trigger / write-down | Feeds ratio |
|---|---|---|---|---|---|---|
| **Common equity** (ordinary shares, retained earnings, reserves) | CET1 | Perpetual | Discretionary (dividend) | First loss, going-concern, always | None — always loss-absorbing | CET1 (and Tier 1, total) |
| **AT1 / CoCo bonds** (perpetual subordinated) | AT1 | Perpetual | Non-cumulative, cancellable at issuer discretion | Going-concern at trigger | **CET1 < 5.125%** (typical EU trigger) and/or PONV: conversion to equity or write-down | Tier 1 |
| **Subordinated term debt** (≥5y, amortising) | Tier 2 | Dated | Cumulative obligation while solvent | Gone-concern, in resolution | PONV write-down in resolution | Total capital |
| *(Ineligible hybrids)* | — | — | — | Reclassified out under Basel III (2013–18 transition) | — | — |

The 2010 reforms' bluntest message: **"capital" is only as good as its loss absorption**. Instruments that carried Tier 1/2 status under Basel II but lacked the new features (cumulative coupons, early call options, missing PONV clauses) were phased out by 2023 at the latest, and any instrument that "provides an incentive to redeem" is ineligible. For the systems angle — how these instruments are issued, serviced, and reported — see the [Core Banking Systems Guide](core_banking_systems_guide.md) and the [RegTech Guide](regtech_guide.md).

### 2.3 The CET1 Reconciliation

CET1 is not "book equity" — it is book equity *minus a long list of regulatory deductions*, which is why two banks with identical accounting equity can report very different CET1. The reconciliation (illustrative; the deduction rules are in bcbs189 and CRR articles 36–56):

| Item | €bn (illustrative) |
|---|---|
| Ordinary share capital + share premium | 9.5 |
| Retained earnings and reserves | 16.0 |
| Eligible minority interests | 0.5 |
| **Gross CET1** | **26.0** |
| − Goodwill and other intangibles | −3.0 |
| − Deferred tax assets relying on future profitability | −1.0 |
| − Other deductions (MSRs above threshold, financial-sector holdings, shortfall of provisions) | −0.5 |
| **CET1** | **21.5** |

The pattern to internalise: goodwill and intangibles are *always* deducted (they cannot absorb loss), DTAs are deducted unless they can be offset against tax liabilities, and holdings of *other* banks' capital instruments are deducted to prevent the financial system from capitalising itself with itself. The same reconciliation is computed for AT1 (its own deduction set) and Tier 2, giving the three ratio numerators.

### 2.4 CoCo Mechanics and the AT1 Market

The CoCo's two design parameters are the **trigger** and the **loss-absorption mechanism**:

- **Trigger** — a *regulatory* trigger (the common EU calibration: automatic conversion/write-down when the issuer's CET1 ratio falls below **5.125%** of RWA) and always the **point of non-viability (PONV)** trigger, where the supervisor determines the bank is failing and would otherwise require public support. The 5.125% figure is the calibration that made the market: high enough to convert while the bank is still alive, low enough that conversion is rare.
- **Mechanism** — **conversion** (the CoCo turns into ordinary shares at a pre-set price) or **principal write-down**, which may be *temporary* (written down, and re-written-up if the bank recovers — the Swiss variant) or *permanent* (the EU norm). Both convert the debt into loss absorption at the moment it is needed, i.e. exactly when the equity market would refuse to raise capital.

The AT1 market is Europe's answer to a US gap: US rules do not recognise CoCos as regulatory capital for large banks (flagged — US AT1-equivalent issuance is negligible; the US stack is CET1 + Tier 2 + preferred stock that does not meet Basel AT1 criteria), so the ~€200–300bn outstanding AT1 market (flagged estimate) is predominantly European and Asian. Two events define its behaviour: the 2017–2018 "cocos panic" (when a Deutsche Bank AT1 call decision and accounting-driven fears briefly crushed the asset class) and the **March 2023 Credit Suisse write-down** — FINMA ordered ~US$17bn of AT1 written to zero in the UBS takeover (flagged figure), an outcome that surprised equity holders (who retained some value) and re-priced the whole asset class. The lesson for capital management: AT1 is cheap funding with a *tail risk* — coupons are discretionary, but the trigger is mechanical, and in a resolution the instrument's investors rank *below* equity holders in practice, whatever the marketing said. For the modelling side of AT1 pricing (trigger probability, conversion value) see the [Risk Management Models Guide](risk_management_models_guide.md).

### 2.5 The Capital Stack in a Group Structure

For a group like Crédit Agricole (with Cymbal Bank as the CIB subsidiary), the capital stack is built at two levels: **CET1 is held where the risk is** — the CIB subsidiary is capitalised with equity from the parent, so Cymbal Bank's own balance sheet carries the CET1 that supports its RWA — while **AT1 and Tier 2 are typically issued at the group parent** (Crédit Agricole S.A. level), where the market values the consolidated credit, and downstreamed or counted within the group's consolidated stack. The regulatory consequence is that the *group* ratio is the binding one (consolidated supervision under CRR3; in Singapore, MAS Notice 637 applies to locally incorporated banks and Notice 638 to branches, with the branch's capital allocated from the group). The practical consequence is that a CIB's capital management is really two jobs: optimising the *subsidiary's* CET1 allocation (the scarce resource — group equity), and participating in the *group's* AT1/Tier 2 issuance calendar (the cheaper layers). This split — where the equity sits vs where the debt is issued — is also what resolution planning formalises (§8.4): the holding company issues the bail-inable debt precisely so that resolution can happen at one point of entry.

---

## 3. The Minimums and the Buffers

### 3.1 The Minimums

Basel III's Pillar 1 minimums (verified, bcbs189 and the consolidated Basel Framework RBC20): **CET1 ≥ 4.5% of RWA**, **Tier 1 ≥ 6%**, **total capital ≥ 8%**. These are the *minimums* — the floor below which a bank is insolvent in regulatory terms. In practice every internationally active bank runs far above them, because the **buffers** sit on top and because markets and supervisors punish anyone near the floor.

On top of the minimums sit the **capital buffers** (verified, BCBS framework RBC30/RBC40 and the FSI summary *The capital buffers in Basel III*):

- **Capital conservation buffer (CCB)** — a fixed **2.5% of CET1** (verified). It is always "on". A bank whose CET1 falls *into* the buffer (between 4.5% and 7.0% total CET1, i.e. minimum + buffer) is not insolvent but is **restricted in its distributions**: the **Maximum Distributable Amount (MDA)** framework scales back dividends, share buy-backs, and discretionary bonus payments as capital approaches the 4.5% minimum. The buffer's purpose is to make capital *usable* in a downturn without the bank having to be wound down — it converts the minimum into a "speed limit" on payouts rather than a cliff.
- **Countercyclical capital buffer (CCyB)** — **0–2.5% of RWA**, CET1-eligible (verified; jurisdictions may raise it above 2.5% in periods of excess credit growth). Set by each jurisdiction based on credit-to-GDP gaps and other signals, it rises when credit is overheating and is released when the cycle turns, so that capital is released into the economy precisely when banks need to keep lending. Reciprocity rules make a bank's home jurisdiction's buffer apply to its credit exposures in that jurisdiction wherever the bank is domiciled. Jurisdictions must pre-announce increases by up to 12 months (verified, BIS FSI summary). Post-COVID, many jurisdictions set it at 0.5–1.0%; France's CCyB, relevant for the Cymbal Bank example, has been positive since 2019 — the exact level is a moving target, flagged in §11.
- **G-SIB buffer** — **1.0% to 3.5% of CET1/RWA** depending on the bank's bucket (verified, BCBS RBC40: five buckets with 100bp ranges; the top bucket's 3.5% reflects the 2017 recalibration). Applied to **global systemically important banks** designated by the FSB from the BCBS scoring methodology (size, interconnectedness, cross-jurisdictional activity, substitutability, complexity — 130bp cutoff, verified). The buffer is pure **higher loss absorbency (HLA)** — extra CET1 so that a G-SIB's failure would not cascade. The FSB publishes the list annually: Crédit Agricole (the Cymbal Bank parent group) sits in the 1.5% bucket as of the November 2024 list (flagged — bucket levels move annually).
- **D-SIB / O-SII buffer** — up to **2.5%** (verified, BCBS RBC40; EU O-SII buffers follow the EBA methodology), set by national supervisors for **domestic systemically important banks** (D-SIBs; O-SII in EU parlance — Other Systemically Important Institutions). Same logic as the G-SIB buffer, applied to national champions.
- **Systemic risk buffer (SRB)** — an EU-specific buffer for long-term non-cyclical systemic risk (CRD allows up to 5%; flagged — this is a CRD/CRR construct, not a Basel standard per se).

The **combined buffer requirement** is the sum of the CCB, CCyB, G-SIB, and D-SIB/SRB buffers. Breaching the combined buffer triggers MDA restrictions; breaching *minimums* triggers recovery and, ultimately, resolution. The US implements the CCB family differently for its largest banks — the **Stress Capital Buffer (SCB)** replaces the fixed CCB with a bank-specific amount derived from the annual CCAR stress test (flagged, US-specific — see §7).

### 3.2 The Buffers Table

| Buffer | Size (CET1 % RWA) | Nature | Trigger / consequence of breach |
|---|---|---|---|
| **Capital conservation buffer (CCB)** | **2.5%** | Always on, fixed | Capital in buffer zone (4.5–7.0% CET1) → **MDA**: distributions (dividends, buybacks, bonuses) scaled back progressively |
| **Countercyclical buffer (CCyB)** | **0–2.5%** (jurisdiction-set; higher possible in stress) | Macroprudential, time-varying | Raised when credit growth is excessive; released in downturns; reciprocity across jurisdictions; breach adds to MDA constraint |
| **G-SIB buffer** | **1.0–3.5%** by bucket (1.0 / 1.5 / 2.0 / 2.5 / 3.5) | Higher loss absorbency for global systemically important banks (FSB/BCBS list) | Breach → MDA constraint (same distribution limits) |
| **D-SIB / O-SII buffer** | **0–2.5%** (national calibration) | Domestic systemic importance (EU: O-SII; SG: MAS designates D-SIBs) | Breach → MDA constraint |
| **Systemic risk buffer (SRB)** | Up to 5% (EU CRD; flagged) | EU-specific, non-cyclical systemic risk | Breach → MDA constraint |
| **Combined buffer requirement** | Sum of all above (typical G-SIB: 2.5 + 0.5 + 1.5 = 4.5%) | The operative distribution constraint | Capital below combined → **MDA** restrictions; below minimums → recovery/resolution |

The practical read: a G-SIB's *effective* CET1 requirement is 4.5% (minimum) + 2.5% (CCB) + CCyB + surcharge ≈ **9.5–10.5% before any distribution** — which is why the banks one reads about (DBS, HSBC, Crédit Agricole) all report CET1 ratios in the 12–16% band. For the DBS/MAS capital-add-on episode that made this vivid in Singapore, see the [Banks in Singapore Guide](banks_in_singapore_guide.md) and the [DBS Bank Guide](dbs_bank_guide.md).

### 3.3 The MDA Mechanics

The **Maximum Distributable Amount (MDA)** is how the buffers bite. When a bank's CET1 falls inside its *combined buffer requirement* (the sum of CCB + CCyB + G-SIB + D-SIB), it is not in breach of minimums — but its **distributions are capped as a percentage of its earnings**. The Basel Framework's conservation table (verified, Basel Framework RBC30, Table 1) divides the 2.5% conservation buffer into quartiles:

| CET1 ratio (% RWA) | Maximum payout (% of distributable earnings) |
|---|---|
| ≥ 7.0 (4.5% minimum + full 2.5% buffer) | No restriction (100%) |
| 6.375 – 7.0 | 60% |
| 5.75 – 6.375 | 40% |
| 5.125 – 5.75 | 20% |
| 4.5 – 5.125 | 0% |

(Distributions = dividends, share buy-backs, and discretionary bonus payments; the constraint is recalculated at each distribution date. The EU transposition in CRD IV Article 141 uses slightly different band boundaries — flagged.) When the CCyB and G-SIB/D-SIB buffers are added, the *combined* buffer widens the zone: with a 5.0% combined buffer, a G-SIB's distribution freedom only fully returns at CET1 ≥ 9.5%. The design intent: a bank in trouble **conserves automatically** — no supervisor needs to order it — and markets see the MDA trigger as the first amber light, ahead of the red light of minimums breach.

### 3.4 The Full Capital Stack, One View

Pulling §2 and §3 together, the complete requirement ladder for a typical G-SIB-style bank:

| Layer | Requirement (CET1 terms) | Behaviour |
|---|---|---|
| Pillar 1 minimum | **4.5%** | Insolvency line; breach → recovery/resolution |
| Capital conservation buffer | **+2.5%** | Distribution restriction zone (MDA) |
| CCyB + G-SIB + D-SIB | **+0.5–6.5%** (bank-specific) | Same MDA zone, widened |
| Combined buffer requirement | ≈ **9.5–11%** | Full distribution freedom returns above this |
| Pillar 2 (P2R) | +1–3% (supervisor-set) | Binding but *usable* (losses may consume it) — §8 |
| Pillar 2 guidance (P2G) | +0–1% | Supervisory expectation, non-binding |
| **Real-world CET1** | **12–16%** | What well-run large banks actually hold |

Every percentage point in that ladder is expressed in CET1 terms but binds the whole stack: a bank that cannot meet the combined buffer with CET1 must still hold Tier 1 and total capital above 6% and 8% — the ratios interact.

### 3.5 The Buffers in Action: The COVID Release

The buffers' design was tested within a decade of their introduction. In **March 2020**, as the COVID-19 shock hit, the GHOS (the BCBS's governing body) and national authorities moved in the intended direction: the **countercyclical buffer was cut to 0%** across most jurisdictions (it had been 0–0.5% in most places anyway, so the real release was the *signal*), and supervisors — the ECB most explicitly — announced that banks could **temporarily operate inside the conservation buffer and Pillar 2 guidance** without triggering automatic supervisory action (the minimums, however, were never waived). Banks were expected to *use* the buffers to absorb losses and keep lending, which is precisely the behaviour the framework was designed to permit: buffers are meant to be run down in stress and rebuilt in the recovery — the CCyB's whole logic is that it is released exactly when credit is needed. The episode also showed the limits: because most banks entered 2020 with CET1 ratios far above the combined requirement, few ever came close to the MDA zone, and the buffer release was more reassurance than arithmetic. (Verified directionally from the BCBS/GHOS March 2020 statements and ECB announcements; the specific ECB "operate below buffer/P2G" language is flagged.)

### 3.6 How a Bank Becomes a G-SIB: The Score and the Buckets

G-SIB designation is mechanistic but judgment-touched (verified, BCBS RBC40 and the G-SIB assessment methodology): the BCBS computes a **score** from five weighted categories — **size** (20%), **interconnectedness** (20%), **cross-jurisdictional activity** (20%), **substitutability** (20%), and **complexity** (20%) — using publicly disclosed balance-sheet data; banks above a **130 basis-point cutoff** are designated G-SIBs by the FSB, and the score maps to a buffer bucket (each bucket spans 100bp of score):

| Score bucket | Higher loss absorbency (CET1 % RWA) |
|---|---|
| Bucket 1 (lowest) | **1.0%** |
| Bucket 2 | **1.5%** |
| Bucket 3 | **2.0%** |
| Bucket 4 | **2.5%** |
| Bucket 5 (highest) | **3.5%** (raised from 2.5% in the 2017 recalibration) |

Two subtleties: the bucket determines the buffer *until the next annual review* (the FSB list is republished each November, so a bank's surcharge moves year to year — Crédit Agricole Group has sat in the 1.5% bucket in recent lists, flagged); and the score is *reported* by the bank itself under the disclosure framework, so G-SIB status is partly self-declared and then verified. D-SIB buffers follow the same logic at national scale, with national methodologies (the EU's O-SII identification uses EBA guidelines; MAS designates D-SIBs among Singapore-incorporated banks — see the [Banks in Singapore Guide](banks_in_singapore_guide.md)).

---

## 4. The RWA: Risk-Weighted Assets

### 4.1 What RWA Is

**Risk-weighted assets** are the denominator of every capital ratio: the bank's exposures, each multiplied by a **risk weight** that reflects the exposure's riskiness, summed across the book. The entire regulatory-sophistication battle — Basel I's flat weights, Basel II's models, Basel IV's floor — is a battle over *whose risk weights count*: the regulator's (standardised) or the bank's (internal models). RWA covers four risk families: **credit risk** (the dominant share for a commercial bank), **market risk** (the trading book), **operational risk**, and (since Basel III) **CVA risk** on derivatives, with **counterparty credit risk** (CCR) sitting inside the credit/market boundary.

### 4.2 Credit Risk: The SA vs the IRB

**The Standardised Approach (SA)** (verified): the supervisor prescribes the risk weights, mostly from external credit ratings and exposure class. Under the 2017-revised SA (bcbs424), the grid is far more granular than Basel I: investment-grade corporates 65% (down from 100%), unrated corporates 100% (or 85% for investment-grade SMEs, 75% retail and SME retail), residential mortgages 20–45% depending on LTV, commercial real estate 60–110% (LTV-driven), and — the headline change — **equity exposures up to 400%** and real-estate-dependent equity 250–400%. Banks with no ratings can use the **standardised approach for credit risk assessment (SCRA)**. The SA's virtue is comparability and simplicity; its vice is blunter risk sensitivity.

**The Internal Ratings-Based approach (IRB)** (verified): banks use *their own* models for the capital drivers — **PD** (probability of default), **LGD** (loss given default), **EAD** (exposure at default) — under supervisory approval, mapping them through the Basel II asymptotic single-risk-factor (Vasicek) formula into capital. Two flavours: **Foundation IRB (F-IRB)** — bank supplies PD, supervisor supplies LGD/EAD — and **Advanced IRB (A-IRB)** — bank supplies all three. IRB is the reason large banks' credit RWA can be 20–50% lower than the SA's — and the reason the BCBS spent 2014–2017 reining it in (parameter floors, and the IRB *restrictions* in §6: no IRB for large financial-sector entities or equities, PD floors, LGD floors). The model machinery — PD/LGD/EAD estimation, the Vasicek formula, validation — is the subject of the [Risk Management Models Guide](risk_management_models_guide.md), which this guide cross-references rather than duplicates.

### 4.3 Market Risk: The FRTB

Market-risk capital is the trading book's charge, and since the **Fundamental Review of the Trading Book (FRTB)** it has its own architecture: a **revised boundary** between the trading book and the banking book (fewer assets may be arbitrarily parked in either), and two approaches — the **Standardised Approach (SA)**: sensitivities-based method (delta, vega, curvature risk) plus a default-risk charge; and the **Internal Models Approach (IMA)**: **Expected Shortfall at 97.5%/10-day** (replacing 99% VaR), with the **non-modellable risk factors (NMRF)** charged separately, a **Default Risk Charge (DRC)**, and **P&L attribution tests (PLA)** as the entry ticket. The revised FRTB standard is dated **January 2019** (verified, BIS d457), effective 1 January 2022. FRTB is covered in depth from the markets/architecture side in the [Capital Markets Architecture Guide](capital_markets_architecture_guide.md) and from the modelling side in the [Risk Management Models Guide](risk_management_models_guide.md) — cross-reference both rather than re-deriving the formulas here.

### 4.4 Operational Risk: The SMA

Operational risk capital had, under Basel II, four approaches (BIA, TSA, AMA) of which the **AMA** let large banks use internal loss-distribution models. The December 2017 finalisation **removed the AMA** and replaced all approaches with the **Standardised Measurement Approach (SMA)** (verified, bcbs424 and the BIS *Standardised Measurement Approach for operational risk* note d355). The SMA is deliberately *formulaic*: capital = **Business Indicator (BI)** × marginal coefficients (12% up to €1bn, 15% €1–30bn, 18% above €30bn) × an **Internal Loss Multiplier (ILM)** (1 when a bank's 10-year average operational losses equal the BI-component benchmark; 0.8–1.4 range otherwise). The BI is a financial-statement proxy (interest income + services income + financial components). In the consolidated Basel Framework the approach is presented as the single Standardised Approach for operational risk (chapter OPE25; the loss-multiplier treatment was adjusted during framework consolidation — flagged in §11). The SMA's critics note it is only weakly risk-sensitive (the ILM is bounded), which is exactly what the Committee wanted: comparability over model sophistication.

### 4.5 Counterparty Credit Risk and CVA

Derivatives exposures attract **counterparty credit risk (CCR)** capital under the **SA-CCR** (the 2014 standardised approach for counterparty credit risk, replacing the old CEM add-on approach; internal-model-method (IMM) remains for the largest houses) and a separate **CVA charge** (the risk that the counterparty's credit spread moves against the bank on its derivatives book). The CVA framework was rewritten in the December 2017 package (§6): the **SA-CVA** (sensitivities-based) and **BA-CVA** (basic approach), with a simplified alternative for small books.

### 4.6 The RWA Table

| Risk | Approaches | Notes |
|---|---|---|
| **Credit risk** | **SA** (external-ratings/class-based weights: IG corporate 65%, unrated 100%, retail 75%, mortgages 20–45% by LTV, equity up to 400%) vs **IRB** (F-IRB: bank PD, supervisor LGD/EAD; A-IRB: all bank-supplied) | IRB restricted by Basel IV (no IRB for large financials/equities; PD/LGD floors); Vasicek ASRF formula converts PD/LGD/EAD to capital — see [Risk Management Models Guide](risk_management_models_guide.md) |
| **Counterparty credit risk (CCR)** | **SA-CCR** (2014, replaces CEM), internal model method (IMM) | Derivatives EAD via SA-CCR; margin and netting recognised |
| **Market risk** | **FRTB**: SA (sensitivities-based: delta/vega/curvature + DRC) vs **IMA** (ES 97.5%/10-day + NMRF + DRC + PLA) | Revised standard January 2019, effective 2022; trading/banking book boundary revised — see [Capital Markets Architecture Guide](capital_markets_architecture_guide.md) |
| **Operational risk** | **SMA** (BI × marginal coefficients 12/15/18% × ILM) | Replaced BIA/TSA/**AMA** (Dec 2017); formulaic, low model risk |
| **CVA risk** | **SA-CVA** / **BA-CVA** / simplified | Revised December 2017; hedges recognised under strict criteria |

### 4.7 The Standardised Risk-Weight Grid (2017 Revision)

The revised SA (bcbs424) is the floor's reference basis, so its grid is worth knowing in outline (simplified — the full standard is finer, with rating-grade steps and LTV bands; verified directionally from bcbs424 and secondary summaries):

| Exposure class | Risk weight (simplified) |
|---|---|
| Sovereigns (rated, domestic-currency) | 0% (up to 150% for weak ratings) |
| Banks (rated) | 20–150% by rating grade |
| Corporates — investment grade | **65%** (was 100% under the old SA) |
| Corporates — unrated / sub-investment-grade | 100% (85% for SME corporates) |
| Retail — residential mortgages | 20–45% (LTV-driven, risk-insensitive 20% for low-LTV) |
| Retail — other | 75% |
| Commercial real estate | 60–110% (LTV-driven) |
| Equity (listed / all other) | 250% / 400% (real-estate dependent: 150–400%) |
| Past-due exposures | 150% (with credit-risk-mitigation reductions) |

The pattern: the 2017 SA *lowered* the weights of good-quality exposures (IG corporate 100→65%) while *raising* the weights of risky ones (equity up to 400%, past-due 150%) — a deliberate shift of the capital burden onto risk. Because the output floor is computed against this grid, the SA's calibration now sets a *global minimum cost of risk* for every bank, whether it uses the SA or not.

### 4.8 The IRB Formula in Words

The IRB capital formula is the Basel II asymptotic single-risk-factor (ASRF) model — the Vasicek one-factor framework (carried into Basel III/IV with parameter floors). In words: capital for a loan = **EAD × LGD × (unexpected-loss factor − expected loss)** where the unexpected-loss factor is the probability that the obligor's asset value falls below the default point in a severe (99.9%) downturn, conditional on one common economic factor. The correlation parameter ρ (12–24% for corporates, size-adjusted) embeds the idea that large firms' defaults are more driven by the common factor. The 2017 floors (PD ≥ 5bp corporates / 10bp financials; LGD floors on unsecured exposures) deliberately push the formula's *inputs* up so the formula cannot be optimised into meaninglessness. The full derivation, the estimation of PD/LGD/EAD, and the validation regime (the use test, SR 11-7-style model risk management) are the subject of the [Risk Management Models Guide](risk_management_models_guide.md) — this guide stops at the regulatory shape.

### 4.9 RWA in the Data Estate

For the architect's eye, RWA is not a number — it is a *data product* assembled every quarter from: exposure data (facilities, drawn/undrawn, collateral, seniority), rating data (external and internal PD grades), model parameters (LGD, EAD conversion factors, correlations), market data (positions, sensitivities, risk factors for the FRTB/ES engine), loss data (the op-risk loss history feeding the SMA's ILM), and reference data (the instrument ledger behind the capital deductions). Each input has an owner, a system of record, and a quality bar — and BCBS 239 (*Principles for effective risk data aggregation and risk reporting*) exists precisely because the 2008 crisis showed that banks could not produce credible RWA and risk reports under stress. The production cycle (month-end positions → risk engines → RWA by risk type → COREP/Pillar 3 templates → board and regulator) is the subject of the [Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md), the [Core Banking Systems Guide](core_banking_systems_guide.md), and the [Risk Data Aggregation Guide](../technology/risk_data_aggregation_guide.md). The capital framework this guide describes is the *specification*; that data pipeline is the *implementation*.

---

## 5. The Leverage Ratio

### 5.1 The 3 Percent Backstop

The leverage ratio is the *non-risk-based* backstop that Basel III added so that clever RWA compression could never disguise true balance-sheet size (verified: Basel III introduced a minimum leverage ratio of **3%**; the final framework *Revised Basel III leverage ratio framework and disclosure requirements* is dated January 2014, BIS bcbs270):

**Leverage ratio = Tier 1 capital ÷ leverage exposure ≥ 3%**

The **leverage exposure measure** is deliberately broad: on-balance-sheet assets (no risk weighting), plus derivative add-ons (under SA-CCR), plus securities-financing (SFT) exposures, plus off-balance-sheet items at credit conversion factors. It is the *accounting-size* denominator that the RWA denominator can no longer shrink. For every $100 of exposure, at least $3 of Tier 1 (verified).

Two calibrations matter on top of the 3%:

- **G-SIB leverage buffer**: G-SIBs must hold an additional leverage buffer equal to **50% of their G-SIB surcharge** (verified; in the US rule this is the "leverage buffer requirement" of 50% of the surcharge — 12 CFR 217.11; the Basel framework carries the same 50% calibration).
- **US supplementary leverage ratio (SLR)**: US G-SIBs are subject to a tighter, US-specific calibration — 5% at the top-tier bank holding company and 6% at the insured depository institution (flagged — US-specific, not a Basel standard).

The leverage ratio is where investment banks and CIBs feel the squeeze: derivative and SFT books inflate the exposure measure without adding RWA, so a markets-heavy bank can have a *comfortable* CET1 ratio and a *tight* leverage ratio simultaneously — a theme the worked example in §9 exploits. Singapore's MAS implements the 3% minimum with the same exposure definition for banks incorporated in Singapore (MAS Notice 637, §7).

### 5.2 The Leverage Table

| Element | Calibration | Notes |
|---|---|---|
| **Minimum leverage ratio** | **3%** (Tier 1 ÷ leverage exposure) | Finalised January 2014 (bcbs270); binding backstop for all banks |
| **Exposure measure** | On-balance-sheet assets + derivative add-ons (SA-CCR) + SFT exposures + off-balance-sheet at CCFs | No risk weighting — the point |
| **G-SIB leverage buffer** | **50% of the G-SIB surcharge** (e.g. 1.5% surcharge → +0.75%) | Basel standard and US rule (12 CFR 217.11) |
| **US SLR** | 5% (top-tier BHC) / 6% (insured depository) for US G-SIBs | US-specific; flagged |
| **EU (CRR3)** | 3% minimum + G-SII leverage buffer (50% of G-SII buffer) | CRR3 keeps the Basel calibration |
| **MAS (Singapore)** | 3% minimum under MAS Notice 637 | Applies to Singapore-incorporated banks |

### 5.3 Why Derivatives and SFTs Inflate the Exposure Measure

The leverage exposure measure is where CIBs differ from retail banks. Three mechanics inflate it far beyond balance-sheet size:

- **Derivative add-ons**: derivatives enter at their positive fair value *plus* a potential-future-exposure add-on (under SA-CCR) — a large derivatives book contributes exposure even when netted to near-zero economic risk.
- **SFT gross-up**: securities-financing transactions (repo, reverse repo, securities lending) count at their full exposure — a repo book that nets out in risk terms still adds the gross lent amount to the denominator.
- **Off-balance-sheet at CCFs**: commitments and guarantees enter at credit conversion factors.

The consequence: a markets-heavy bank can show *comfortable* CET1 and *tight* leverage simultaneously. Illustratively, a €650bn-asset CIB with a €120bn SFT book and €90bn gross derivatives can carry a leverage exposure of €840bn+ — nearly 30% above total assets — while a retail bank's exposure tracks its balance sheet closely (see the §9 worked example, where the ratio lands at 3.10%). This is why the leverage ratio, not CET1, is the operative constraint for the trading-book businesses, and why the US G-SIB SLR debate of 2020–21 (reserve and Treasury growth during COVID stress-testing the US-specific 5%/6% calibrations — flagged, US-specific) mattered: when the denominator is defined gross, balance-sheet growth is expensive even at zero risk weight.

### 5.4 A Mini Leverage Calculation

The arithmetic is worth doing once by hand. Take a stylised bank: **€100 on-balance-sheet assets, €30 of undrawn committed facilities (50% CCF → +15), €20 of derivative add-ons (+20), €10 of SFT gross-up (+10)** → leverage exposure **€145**. With Tier 1 of €4.6 (i.e. CET1 3.6 + AT1 1.0), the ratio is 4.6/145 = **3.17%** — passing, but notice what happened: the bank's *risk-based* Tier 1 ratio might be 15% (RWA ≈ €31) while its leverage ratio is 3.2%. The two denominators tell different stories: RWA says "this bank is conservative"; the exposure measure says "this bank is big". Regulatory capital is the *intersection* of both stories — the binding constraint is whichever bites, and for markets-heavy banks it is usually the leverage one.

---

## 6. The Basel IV: The 2017 Finalisation

### 6.1 What "Basel IV" Actually Is

"Basel IV" is the market's name for **Basel III: Finalising post-crisis reforms**, published by the BCBS on **7 December 2017** (verified). The Committee itself calls it the "finalisation of Basel III" — it closes the agenda opened in 2010. Its central purpose was to **reduce unacceptable variability in RWA** — the BCBS's own studies had shown RWA for identical portfolios differing by 40–70% across banks depending on model choices, which made capital ratios not comparable and models arbitrageable. The package has five engines (verified across bcbs424 and the PwC/RegPulse summaries):

1. **The output floor** — the headline.
2. **Credit-risk revisions** — a more granular SA and heavy IRB restrictions.
3. **The CVA framework revision** — SA-CVA / BA-CVA.
4. **Operational risk** — the SMA replacing the AMA (and BIA/TSA).
5. **Market risk** — handed to the FRTB, finalised January 2019.

### 6.2 The Output Floor: 72.5 Percent

The **output floor** (verified) caps how much capital relief internal models can deliver: a bank using IRB/IMA models must hold capital of at least **72.5% of the capital its exposures would attract under the standardised approaches** — equivalently, its modelled RWA cannot fall below 72.5% of standardised RWA. A bank whose models produce RWA of 60% of the standardised number must therefore compute capital on 72.5% of the standardised RWA, not on its own 60%. The floor was phased in from **50%** (2022) rising by 5pp per year to **72.5%** (2027) in the original calendar; the implementation delays (§7) shifted the phase-in to 2023–2028 for G-SIBs and 2024–2029 for others (EU: 50% → 72.5% by 2030 — verified via EBA/EC sources). The floor bites hardest on precisely the banks the models helped most: A-IRB corporates and mortgage books, and markets-heavy CIBs whose IMA desks compress market RWA.

### 6.3 The IRB Restrictions

The 2017 package restricted the IRB approach in four ways (verified):

- **Scope**: the IRB approach may no longer be used for **large financial-sector entities** (banks, insurers above size thresholds) and **equity exposures** — these must be standardised.
- **Parameter floors**: hard floors on **PD** (5bp for corporate/exposure-class; 10bp for financials) and on **LGD** (unsecured corporate/retail exposures floored, e.g. 25–50% bands by collateral type) and EAD conversion factors.
- **Loss-given-default model restrictions**: LGD estimation for real-estate collateral tightened to the "reduced form" for exposures without sufficient loss data.
- **The effect**: banks that built Basel II-era A-IRB empires must now either re-estimate under floors or move exposures to the SA — the RWA uplift is the whole point.

### 6.4 The CVA Revision

The CVA framework (verified) was rewritten in the same package: a **sensitivities-based SA-CVA** (with supervisory and internal-model variants) and a **BA-CVA** (basic approach using regulatory or internal default-probability inputs and the SA-CCR exposure framework), plus a simplified approach for small derivative books. The revision tightened the recognition of hedges (only CVA-relevant hedges reduce the charge) and re-based the exposure calculation on SA-CCR.

### 6.5 The Basel IV Table

| Reform | What changed | Notes / status |
|---|---|---|
| **Output floor** | Modelled RWA cannot fall below **72.5% of standardised RWA** | Phased 50% → 72.5% (EU by 2030, UK from 2027, MAS from 2024 — §7) |
| **Credit risk — SA** | Recalibrated, more granular risk weights (IG corporate 65%, unrated 100%, mortgages LTV-based, equity up to 400%); SCRA for unrated banks | Applies to all banks; the floor's reference basis |
| **Credit risk — IRB** | **No IRB for large financials/equities**; PD floors (5bp/10bp), LGD floors, EAD floors; reduced-form LGD for real estate | Biggest RWA uplift for A-IRB corporate banks |
| **CVA risk** | **SA-CVA / BA-CVA** replace the 2010 CVA charge; stricter hedge recognition | Applies to derivatives books |
| **Operational risk** | **SMA** (BI × coefficients × ILM) replaces BIA/TSA/**AMA** | Formulaic; removes internal models for op risk |
| **Market risk** | **FRTB** finalised January 2019 (d457): ES 97.5%, NMRF, DRC, PLA; revised boundary | Effective 2022; the trading-book half of Basel IV — see [Capital Markets Architecture Guide](capital_markets_architecture_guide.md) |
| **Pillar 3** | Revised disclosure framework (2018–2021 iterations) | The market-discipline leg updated alongside |

### 6.6 The RWA-Variability Evidence

The output floor is a response to measurement, not theory. The BCBS's own RWA studies (2013 for credit risk, 2016 for market risk — flagged as reported findings) showed that banks holding *identical* hypothetical portfolios produced RWA that differed by **40–70% or more** depending on model choices, supervisory practices, and definitional gaps. Two implications: (a) capital ratios were not comparable across banks, which broke Pillar 3's market-discipline premise; and (b) models had become a *choice variable* — the bank with the most aggressive (validated!) models paid the least capital for the same risk. The Committee's answer was not to abolish models but to bound them: the floor guarantees that modelled RWA cannot fall below 72.5% of the standardised number, so model sophistication can *reduce* capital only within a 27.5% band. The floor is thus the closing act of the Basel II bargain — models earned their keep in 2004; 2017 set the price.

### 6.7 The Basel IV Phase-In Calendar

| Milestone | Date |
|---|---|
| bcbs424 finalisation published | **7 December 2017** |
| FRTB revised standard | **January 2019** (effective 1 January 2022) |
| Original BCBS target (G-SIBs / others) | 1 January 2023 / 1 January 2024 |
| COVID-19 postponement (BCBS announcement) | March 2020: +1 year |
| Revised G-SIB target | 1 January 2028 (flagged — confirm the latest GHOS statement) |
| EU CRR3 applies / output floor fully loaded | 1 January 2025 / **2030** (50% → 72.5%) |
| UK Basel 3.1 applies | **1 January 2027** (delayed from 2026) |
| MAS Notice 637 applies (phased) | **1 July 2024** |
| US | No date — re-proposed 19 March 2026 (flagged) |

The pattern: the *standard* is final, but its *law* is a staggered, jurisdiction-by-jurisdiction build-out — the subject of the next section.

### 6.8 What Basel IV Did Not Change

For balance, the finalisation left the *architecture* untouched. The **minimums stayed at 4.5% / 6% / 8%**; the **buffers stayed** (CCB 2.5%, CCyB 0–2.5%, G-SIB 1–3.5%, D-SIB 0–2.5%); the **leverage ratio stayed at 3%** (with the G-SIB leverage buffer at 50% of the surcharge); the **LCR and NSFR stayed at 100%**; and the three-pillar structure survived intact. What changed was *inside* Pillar 1: the calibration of the standardised risk weights, the scope and floors of the internal models, the op-risk and CVA formulas, and — through the output floor — the maximum benefit any model can deliver. The design message is worth stating: Basel IV is not a new religion; it is the re-engineering of the existing one — "same cathedral, new flying buttresses". The liquidity half of the cathedral is unchanged too — see the [Treasury & ALM Guide](treasury_alm_guide.md).

---

## 7. The Implementation

Basel standards are not law: each jurisdiction transposes them. The G-SIB implementation date moved from 2023 to 2028 as jurisdictions slipped; as of writing (August 2026) the picture is: **EU in force, UK finalised but delayed, US re-proposed and unresolved, MAS in force with a phased build-up.** The detail below is verified to the dates shown; anything uncertain is flagged.

### 7.1 The European Union: CRR3, from 1 January 2025

The EU transposed the finalisation as **CRR3 — Regulation (EU) 2024/1623** (amending the CRR, Regulation 575/2013) plus **CRD VI — Directive (EU) 2024/1619**. Verified: **CRR3 applies from 1 January 2025** (EBA's CRR3/CRD6 dashboard; European Commission: "The EU is implementing the Basel III framework beginning 1 January 2025"); CRD VI must be transposed into national law by **11 January 2026**. Key EU choices: the **output floor phases from 50% (2025) to 72.5% by 2030** (verified); FRTB market risk applies from 2025 with the **internal models approach deferred to 1 January 2026** (the "market risk deferral" — verified via betterregulation.com and EBA); operational risk moves to the new standardised approach (AMA removed) from 2025. The European Commission's impact assessment estimated a weighted-average increase in minimum requirements of roughly **+6.4% to +8.4% by 2030** after transition (verified via EY's summary of the EC assessment). The EU applies the finalisation to *all* ~4,500 EU banks (solo and consolidated), wider than the Basel "internationally active" scope.

### 7.2 The United Kingdom: Basel 3.1, Delayed to 1 January 2027

The PRA's implementation — "Basel 3.1" — progressed in stages: **PS17/23** (September 2023, near-final part 1), **PS9/24** (12 September 2024, final rules for credit risk and the output floor, at that point effective **1 January 2026**), **PS7/25** (SME and infrastructure-lending adjustments), and **PS1/26** (published 20 January 2026, finalising market risk, CVA, operational risk, and reporting/disclosure). Verified: in **January 2025 the PRA announced a one-year delay to 1 January 2027**, explicitly "to allow more time for greater clarity to emerge about plans for its implementation in the United States" (Bank of England news release, January 2025). So the UK's final rules are written but the effective date is **1 January 2027** — the UK is waiting on the US.

### 7.3 The United States: The Basel III Endgame — Re-Proposed, Not Finalised (FLAGGED)

The US story is the open item. Verified: the federal banking agencies (FRB, OCC, FDIC) proposed the "Basel III endgame" package in **July 2023** for banks with ≥$100bn in assets (a G-SIB-heavy scope, with the agencies' own estimates of meaningful CET1/RWA increases; the proposal also folded in the US's signature differences — the stress capital buffer, the SLR). The proposal was **never finalised** — it drew heavy industry and political opposition, and the agencies announced in late 2024 that they would **re-propose**. Verified: on **19 March 2026** the three agencies issued **three re-proposals** ("Basel III Endgame, Take Two") that together would comprehensively rewrite the US capital framework with materially reduced stringency — press reporting cites roughly **$87.7bn of system-wide CET1 relief** relative to the 2023 proposal (Fox Rothschild summary of the agencies' announcement; Freshfields and EY analyses of the 19 March 2026 package). Status as of writing: **proposal, comment period open, no final rule, no effective date — flagged.** Any US implementation remains years away, which is why the UK delayed and why EU banks watch the US calendar.

### 7.4 Singapore: MAS Notice 637, Effective 1 July 2024 (Phased)

MAS implements the final Basel III reforms for banks incorporated in Singapore through the revised **MAS Notice 637** ("Risk Based Capital Adequacy Requirements"). Verified: the revised Notice was published **20 September 2023** (circular PPD 10/2023) and is **effective 1 July 2024**, with a phased build-up per MAS's finalised implementation timeline (circular PPD 08/2023) — including the output floor transitioning toward 72.5% over the implementation window (the specific annual phase-in percentages are MAS-calibrated; flagged in §11). Two MAS-specific notes, both flagged: (a) MAS's *minimum* ratios under Notice 637 — CET1 6.5%, Tier 1 8%, total 10% — sit **above** the Basel III 4.5/6/8 minimums (per MAS Notice 637 reporting guidance; verify current Notice wording); with the 2.5% conservation buffer the combined total requirement is 12.5% before buffers. (b) Foreign banks in Singapore (including Cymbal Bank's branch) are supervised under MAS Notice 638 (branch capital / "own funds" requirements — flagged; the dedicated SG angle is the [Banks in Singapore Guide](banks_in_singapore_guide.md)). MAS also pairs capital with conduct muscle: the 2023–24 DBS outage episode brought a supervisory capital add-on (~S$1.6bn per public reporting — flagged figure) — a reminder that Pillar 2 add-ons are a live tool in Singapore.

### 7.5 The Implementation Table

| Jurisdiction | Instrument | Effective | Notes |
|---|---|---|---|
| **EU** | **CRR3** (Reg (EU) 2024/1623) + **CRD VI** (Dir (EU) 2024/1619) | **1 January 2025** (CRD VI transposed by 11 Jan 2026) | Output floor 50% → **72.5% by 2030**; FRTB SA from 2025, IMA from Jan 2026; SMA for op risk from 2025; EC estimate +6.4–8.4% minimums by 2030 |
| **UK** | Basel 3.1 (PS17/23, PS9/24, PS7/25, **PS1/26** final) | **1 January 2027** (delayed from 1 Jan 2026, announced Jan 2025) | Delay explicitly to align with the US; final rules published 20 Jan 2026 |
| **US** | "Basel III endgame" | **No final rule** — re-proposed **19 March 2026** | July 2023 proposal never finalised; re-proposal materially lighter (~$87.7bn CET1 relief vs 2023 — press-reported, flagged); SCB and SLR remain US-specific features |
| **Singapore (MAS)** | Revised **MAS Notice 637** (PPD 10/2023) | **1 July 2024**, phased | Output floor phases toward 72.5%; MAS minimums above Basel (CET1 6.5% / T1 8% / total 10% — flagged); Notice 638 for foreign-bank branches (flagged) |
| *Canada (OSFI), Japan, etc.* | National rules | 2023–2025 phasing | Outside this guide's scope; flagged as partial |

### 7.6 What Is Actually In Each Package

| Package | Headline content |
|---|---|
| **EU CRR3** | Revised SA credit risk and IRB restrictions; op-risk standardised approach; FRTB (SA from 2025, IMA from 2026); CVA SA-CVA/BA-CVA; output floor phased 50%→72.5% by 2030; leverage ratio with G-SII buffer; new third-country-branch (TCB) regime under CRD VI; reporting (COREP 3.0) and disclosure revisions |
| **UK Basel 3.1** | Same Basel core, UK-calibrated: PS9/24 (credit risk + output floor), PS1/26 (market risk, CVA, operational risk, reporting/disclosure); SME and infrastructure-lending adjustments (PS7/25) retained as UK divergences; applies 1 January 2027 |
| **US re-proposal (Mar 2026)** | Three coordinated proposals reworking the large-bank capital framework with materially lower impact than the 2023 proposal (~$87.7bn CET1 relief vs 2023 per press reporting — flagged); preserves US-specific machinery (stress capital buffer, SLR); exact scoping per the 19 March 2026 releases — flagged |
| **MAS Notice 637** | Final Basel III reforms for Singapore-incorporated banks: revised credit/op-risk/CVA approaches, output floor phase-in, leverage ratio; MAS minimums above the Basel floor (CET1 6.5% / T1 8% / total 10% — flagged); Notice 638 for foreign-bank branches (flagged) |

The operational reality for a global CIB like Cymbal Bank: the *same* underlying portfolio is re-calculated multiple times — group consolidation under CRR3, branch returns under MAS Notice 638 in Singapore, and (where relevant) local calibrations elsewhere — and each jurisdiction's phase-in means the *same* quarter can produce different ratios in different books. Capital management systems (the COREP/returns engines, the RWA data estate) are discussed in the [Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md), the [RegTech Guide](regtech_guide.md), and the [Risk Data Aggregation Guide](../technology/risk_data_aggregation_guide.md).

### 7.7 The Implementation Signals for Architects

For a solution architect in a bank, "Basel IV implementation" is not a compliance memo — it is a multi-year programme touching every system that touches risk: (a) **the RWA engine** — new SA grids, IRB floors, FRTB SA/IMA, SMA, SA-CVA all re-parameterised; (b) **the reporting layer** — new COREP 3.0 templates (EU), MAS returns, Pillar 3 disclosure formats; (c) **the data layer** — new data requirements (loss history for the ILM, sensitivities for FRTB, LTV and collateral data for the revised SA) that did not exist under Basel II; (d) **parallel runs** — most jurisdictions require the old and new frameworks to run side-by-side for a period, so the engine must support dual calibration; and (e) **the phase-in calendar** — the output floor's annual 5pp steps (EU: 50% in 2025 → 72.5% in 2030) mean the system must compute *and report* the transition, not just the end state. The common failure mode is treating it as a parameter change when it is a data-model change: the revised SA's LTV-driven mortgage weights, for instance, require loan-level collateral data that many legacy core-banking estates never stored — see the [Core Banking Systems Guide](core_banking_systems_guide.md) and the [Temenos Guide](temenos_guide.md) for where that data lives.

---

## 8. The Pillar 2

### 8.1 ICAAP and SREP

Pillar 2 is the supervisory-review pillar that Basel II created and the crisis made central. Two mirror-image processes (verified):

- **ICAAP — Internal Capital Adequacy Assessment Process**: the *bank's* own assessment — it must demonstrate that it holds capital adequate for all material risks, *including those Pillar 1 does not capture*: concentration risk, interest-rate risk in the banking book (IRRBB), model risk, business risk, and stress scenarios. The ICAAP output is the bank's own view of required capital (often an internal "economic capital" target well above the regulatory minimums).
- **SREP — Supervisory Review and Evaluation Process**: the *supervisor's* assessment of the bank against the same risks, concluding in **Pillar 2 requirements**. In EU/UK practice the SREP splits into **P2R** (the binding Pillar 2 *requirement*, expressed in % RWA, usually 1–3% for a large bank; it can be consumed by losses without triggering distribution restrictions, unlike buffers) and **P2G** (Pillar 2 *guidance*, non-binding, a supervisory expectation). Breaching P2G triggers supervisory dialogue; breaching P2R triggers supervisory measures. In the US the analogue is the **stress capital buffer** and the supervisory capital add-ons from CCAR (flagged — different machinery, same spirit).

Pillar 2 is how supervisors make capital *bank-specific*: two identical balance sheets can face different P2R if one is judged riskier. It is also the tool behind real-world add-ons — the MAS capital add-on imposed on DBS after the 2023–24 outages is the Singapore instance (§7.4; [Banks in Singapore Guide](banks_in_singapore_guide.md)).

### 8.2 TLAC and MREL

Resolution capital is the *other* half of the post-crisis answer to "too big to fail". Verified:

- **TLAC — Total Loss-Absorbing Capacity** (FSB *Principles on Loss-absorbing and Recapitalisation Capacity of G-SIBs in Resolution* and the **TLAC Term Sheet**, issued **9 November 2015**): G-SIBs must hold loss-absorbing instruments and liabilities (equity, AT1, Tier 2, and *eligible senior* debt) sufficient that, in resolution, the bail-in can absorb losses and recapitalise the bank. The term sheet's minimums: **16% of RWA and 6% of the leverage exposure measure**, rising for the fully-loaded period (publicly quoted at **18% RWA / 6.75% leverage exposure** from 2022 — flagged, see §11), with subordination requirements so that bail-inable debt sits below excluded liabilities.
- **MREL — Minimum Requirement for Own Funds and Eligible Liabilities** (EU, under the BRRD; the UK equivalent under the Banking Act): the resolution-authority-set equivalent for *all* banks in resolution scope, generally calibrated as loss-absorption (Pillar 1 + Pillar 2R) plus a recapitalisation amount (e.g. P2R on top), with subordination requirements for G-SIIs.

The relationship: TLAC is the FSB's G-SIB standard; MREL is the EU/UK legal instrument implementing the same idea for a wider population. Both share the insight that capital regulation alone cannot end bailouts — you need a pre-built cushion of *bail-inable* liabilities on top of the CET1/AT1/Tier 2 stack.

### 8.3 The Pillar 2 Table

| Element | Who runs it | What it is | Calibration / notes |
|---|---|---|---|
| **ICAAP** | The bank | Own assessment of capital adequacy vs all material risks (incl. Pillar-1-blind spots: concentration, IRRBB, model, business risk) | No fixed formula; drives internal economic-capital targets; feeds SREP |
| **SREP** | The supervisor | Evaluation of the bank's risk, governance, capital, and liquidity | Concludes in P2R (binding) + P2G (guidance); typically 1–3% RWA for large banks |
| **P2R / P2G** | Supervisor (EU/UK) | Pillar 2 requirement vs guidance | P2R binding but *usable* (not a buffer); P2G non-binding expectation |
| **TLAC** | FSB standard → national law | Bail-inable capacity for G-SIBs: equity + AT1 + T2 + eligible senior debt | **16% RWA / 6% leverage exposure** (2015 term sheet; 18%/6.75% fully-loaded — flagged); subordination rules |
| **MREL** | Resolution authorities (EU BRRD; UK) | Loss absorption + recapitalisation capacity in resolution | Calibrated per bank: P1 + P2R (+ recap amount); G-SII subordination requirement |
| **US analogue** | FRB (CCAR) | Stress capital buffer + supervisory add-ons | Different machinery, same purpose (flagged) |

### 8.4 Resolution Planning: The Gone-Concern Stack

Pillar 2's quiet partner is the **resolution** regime. Capital regulation answers "how much can the bank lose and still live?"; resolution planning answers "if it cannot live, how does it die without taking the system with it?" The instruments line up by going-concern/gone-concern:

- **Going-concern capital** (CET1, AT1): absorbs losses while the bank operates — the subject of §2–§6.
- **Gone-concern capital** (Tier 2) and **bail-inable liabilities** (TLAC/MREL): absorb losses *in resolution*, when the resolution authority imposes losses on shareholders and creditors and recapitalises the bank so that critical functions continue.
- **Resolution tools** (under the FSB Key Attributes and, in the EU, the BRRD): bail-in, sale of business, bridge institution, asset separation; operationalised through recovery plans (the bank's) and resolution plans or "living wills" (the authority's).

The strategic point for a CIB: the *debt structure* of a G-SIB is now a regulatory instrument. Issuing eligible senior debt at the holding company (rather than at operating subsidiaries) is what makes **single-point-of-entry** resolution work, and the subordination requirements embedded in TLAC/MREL mean a bank cannot simply issue whatever debt is cheapest — it must maintain a pre-positioned, bail-inable layer sized against RWA and the leverage exposure. Capital and resolution together are why the post-2010 system's answer to "too big to fail" is: *more equity, more buffers, more bail-inable debt, and a plan* — no single pillar is the answer.

### 8.5 Pillar 2 in Practice: What the Disclosures Show

Reading a large EU bank's disclosure, the Pillar 2 picture appears as the **Total SREP Capital Requirement (TSCR)** — the sum of Pillar 1 (4.5% CET1) plus the P2R (typically ~1.0–2.5% for a diversified large bank — typical range, flagged) — plus the **P2G** (often ~0.5–1.5%, flagged) and the combined buffers. Two recurring subtleties: (a) P2R is usually expressed *as a total-capital requirement* in the EU (the binding CET1 sub-amount is published separately), so a bank's "9.5% CET1 minimum-plus-P2R" headline decomposes into 4.5% P1 + ~1.5% P2R-CET1 + 2.5% CCB + CCyB + surcharges; and (b) because P2R is *usable* while buffers are not, supervisors deliberately keep the two apart — a bank that consumes its P2R has not breached a buffer and faces no automatic distribution cap, but it has consumed the supervisor's margin of comfort, and the SREP discussion the following year will reflect it. In the US the same function is performed by the **stress capital buffer** (the CCAR-determined add-on that *replaces* the fixed CCB for the largest banks — flagged, US-specific), which is why US G-SIBs quote "SCB" language that European readers must translate.

---
## 9. The Worked Example: A Capital Calculation for a Cymbal Bank

### 9.1 The Scenario

To make the framework concrete, we build a capital calculation for a **Cymbal Bank-style bank** — the corporate-and-investment-banking arm of a European G-SIB group (Crédit Agricole Group is an FSB-designated G-SIB; Cymbal Bank is its CIB subsidiary, regulated in France by the ACPR/ECB and, in its Singapore branch, by MAS). The bank's shape is the familiar CIB one: a corporate-loan book, a large trading book, a big derivatives and SFT (repo/securities-lending) footprint, and a markets-heavy operational profile.

> **All figures below are illustrative and simplified for pedagogy — they are NOT Cymbal Bank's published numbers.** Real ratios come from the group's public disclosures; the systems context is in the [Crédit Agricole Software Systems Guide](credit_agricole_software_systems_guide.md). What matters is the *method*: every step below is the one a real capital team runs each quarter.

**The illustrative balance sheet (€bn):** corporate loans 180 · trading assets 110 · derivatives (gross positive fair value) 90 · reverse repos / SFTs 120 · financial investments 60 · cash and central-bank balances 40 · other assets 50 → **total assets ≈ €650bn**.

### 9.2 Step 1 — The RWA Build-Up

Each exposure family is risk-weighted under its approach:

| Risk family | Approach | Exposure (€bn) | RWA (€bn) | Notes |
|---|---|---|---|---|
| Corporate credit — IRB (A-IRB) | IRB (PD/LGD/EAD) | 120 | **52** | Large-corporate book; PD floors bind at the margin |
| Corporate credit — SA | SA | 60 | **46** | Unrated/IG mix: 65–100% weights |
| Counterparty credit risk | SA-CCR | 90 (gross derivatives) | **15** | Netting + margin recognised |
| Market risk | FRTB (SA + IMA) | 110 (trading book) | **28** | IMA on the liquid desks; SA + DRC on the rest |
| Operational risk | SMA (BI × coefficients × ILM) | BI ≈ €9bn | **19** | BI in the 15% band (€1–30bn) |
| CVA risk | BA-CVA | 90 (derivatives) | **10** | |
| **Total RWA** | | | **170** | |

RWA composition: credit 57%, market 16%, op risk 11%, CCR 9%, CVA 6% — a CIB-shaped book (a retail bank would show credit at 75–85%).

### 9.3 Step 2 — The Capital Stack

| Capital | Amount (€bn) | % RWA (170) | Minimum | Verdict |
|---|---|---|---|---|
| **CET1** | 21.5 | **12.6%** | 4.5% | ✅ well above |
| **AT1** (CoCos, 5.125% trigger) | 4.5 | 2.6% | — | ✅ |
| **Tier 1** | 26.0 | **15.3%** | 6.0% | ✅ |
| **Tier 2** (5y subordinated) | 3.4 | 2.0% | — | ✅ |
| **Total capital** | 29.4 | **17.3%** | 8.0% | ✅ |

### 9.4 Step 3 — The Buffer Check (Group Context)

The *combined buffer requirement* for the group (G-SIB in the 1.5% bucket): CCB **2.5%** + CCyB **0.5%** (France applies a positive CCyB — level flagged as illustrative) + G-SIB surcharge **1.5%** + O-SII **0.5%** (illustrative) = **5.0%**. The CET1 stack is therefore:

- **4.5%** minimum + **5.0%** buffers = **9.5%** CET1 needed before distributions are fully free;
- Bank at **12.6%** → **3.1pp of MDA-free headroom** → dividends and bonuses are unrestricted (but see §9.6 — Pillar 2 consumes part of this).

### 9.5 Step 4 — The Leverage Ratio and the Output Floor

**Leverage ratio.** Leverage exposure = on-balance-sheet €650bn + derivative add-ons (SA-CCR) + SFT gross-up ≈ **€840bn** (illustrative). Ratio = Tier 1 €26.0bn ÷ 840 = **3.10%** — *barely above the 3% minimum*. The lesson writes itself: a CIB with a €650bn balance sheet and €170bn of RWA has an *ample* risk-based ratio but lives centimetres from the leverage floor. If the SFT book grows €30bn, the ratio drops to 2.99% and the bank must either raise AT1 or shrink — the leverage ratio, not CET1, is the binding constraint.

**Output floor.** Re-run the whole book under standardised approaches: SA credit + FRTB-SA + SMA + BA-CVA ≈ **€225bn** (illustrative). Floor = **72.5% × 225 = €163.1bn**. Internal-model RWA (€170bn) > floor (€163.1bn) → **not binding, but only just** (4% margin). Stress the book: if IRB parameter floors push modelled corporate RWA down to €155bn, the floor lifts RWA back to **€163.1bn**, and CET1 falls from 21.5/155 = 13.9% to 21.5/163.1 = **13.2%**. The floor caps the model benefit — exactly what the Committee designed it to do.

### 9.6 Step 5 — The Pillar 2 Overlay

SREP concludes (illustrative): **P2R 1.5%** (concentration + IRRBB + model risk on the IRB/IMA books) and **P2G 1.0%**. The supervisor's CET1 picture: 4.5% minimum + 5.0% buffers + 1.5% P2R = **11.0% binding-ish CET1**, plus 1.0% guidance = 12.0% expected. The bank's 12.6% is now "adequate but not abundant" — the apparent 3.1pp of buffer headroom shrinks to ~0.6pp once P2G is considered. Note that P2R is *usable* (losses can consume it without MDA) while buffers are not (consuming them triggers MDA) — the two behave differently under stress.

### 9.7 The CET1 Reconciliation Behind the Number

The €21.5bn CET1 is not a single line item — it is the reconciliation of §2.3 in miniature (all figures illustrative):

| Item | €bn |
|---|---|
| Share capital + premium + retained earnings + reserves | 26.0 |
| − Goodwill/intangibles, DTAs, other deductions | −4.5 |
| **CET1** | **21.5** |

The deduction discipline is exactly why two banks with the same accounting equity report different CET1 — and why the *data* behind the reconciliation (which entities hold which instruments, which assets are intangibles, where DTAs sit) is a systems problem as much as a finance problem: the [Core Banking Systems Guide](core_banking_systems_guide.md) (chart-of-accounts and instrument ledgers), the [Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md) (the regulatory-reporting chain), and the [Risk Data Aggregation Guide](../technology/risk_data_aggregation_guide.md) (BCBS 239 data quality) are the machinery that makes the number credible quarter after quarter.

### 9.8 The Lessons

1. **The binding constraint is rarely the minimum ratio.** For a CIB it is usually the leverage ratio (3%) or the combined-buffer/MDA line — check all of them, not just CET1.
2. **The output floor caps the model dividend.** 72.5% of standardised RWA is the real ceiling on IRB/IMA relief; RWA comparability is the regulatory goal.
3. **Buffers are speed limits, not solvency lines.** Falling *into* the conservation buffer restricts distributions (MDA); it does not trigger resolution. Breaching *minimums* is the life-or-death event.
4. **Pillar 2 is where capital becomes bank-specific.** P2R/P2G can add 2–3pp on top of the buffers; identical balance sheets get different requirements.
5. **Capital is a data problem.** CET1 is a legal-instruments ledger; RWA is a product of the data estate (PD/LGD/EAD histories, trade-level positions, loss data). RWA quality is data quality — the [Core Banking Systems Guide](core_banking_systems_guide.md), the [Risk Management Models Guide](risk_management_models_guide.md), and the [Risk Data Aggregation Guide](../technology/risk_data_aggregation_guide.md) (BCBS 239) are the machinery underneath §4.
6. **Jurisdiction is a calculation.** The same group computes CRR3 (EU consolidation) and MAS Notice 637/638 (Singapore branch) in parallel, with different effective dates and phase-ins (§7) — a global CIB runs the model suite multiple times per quarter.

---

## 10. The Summary: One Page

Regulatory capital is the answer to a single question: *if the bank's risks turn out worse than modelled, who absorbs the loss?* Basel's answer, built over four decades: the shareholders first (CET1), then the AT1/CoCo layer, then Tier 2, then — in resolution, not bailout — the TLAC/MREL bail-inable debt, and only then the taxpayer.

- **1988** — Basel I's 8% floor made capital a global quantity; its flat risk weights made it a blunt one.
- **2004** — Basel II's three pillars made capital risk-sensitive (SA vs IRB) and added supervision (Pillar 2) and disclosure (Pillar 3) — but trusted models too far.
- **2010** — Basel III's GFC response rebuilt the *quality* of capital (CET1 4.5%, Tier 1 6%, total 8%), stacked buffers on top (CCB 2.5%, CCyB 0–2.5%, G-SIB 1–3.5%, D-SIB 0–2.5%), added the 3% leverage backstop, and — via the LCR/NSFR, the subject of the [Treasury & ALM Guide](treasury_alm_guide.md) — regulated liquidity for the first time.
- **2017–2019** — "Basel IV" capped model discretion with the **72.5% output floor**, restricted IRB, replaced the AMA with the SMA, rewrote CVA, and finished the FRTB.
- **Today** — the EU is live (CRR3, Jan 2025), the UK is finalised but delayed (Jan 2027, waiting on the US), the US is re-proposed and unresolved (March 2026), and MAS is phased in (Notice 637, Jul 2024). Pillar 2 (ICAAP/SREP, P2R/P2G) and the resolution stack (TLAC/MREL) sit on top of the Pillar 1 machine.

The practical arithmetic for a G-SIB-like bank: roughly **9.5–11% CET1** before buffers-plus-P2 are satisfied, **12–16%** where real banks actually run, a leverage ratio that is often the true binding constraint for markets-heavy firms, and an output floor that quietly sets the ceiling on model relief. The worked example in §9 shows the whole calculation in one pass. At the end of the architecture — above the minimums, inside the buffers, behind the floor, under the supervisor's Pillar 2 lens — what every ratio, buffer, and trigger is ultimately describing is **the capital that anchors** the balance sheet: the equity cushion that lets a bank take risk in the first place, absorb the unexpected without breaking its promises, and keep lending when the economy needs it most. Everything else in the framework exists to define, size, and protect it.

---

## 11. Verification Notes and Sources

**Verified against primary/authoritative sources (line-by-line convention):**

- **Basel I** — July 1988 Accord, *International Convergence of Capital Measurement and Capital Standards*; **8% minimum** and Tier 1 ≥ 50% of total capital; 1996 market-risk amendment (BIS; Bundesbank Basel-framework overview; Investopedia "Basel Accords").
- **Basel II** — **June 2004** framework (BIS bcbs128, *International Convergence of Capital Measurement and Capital Standards: A Revised Framework*); **three pillars** (minimum capital / supervisory review / market discipline) (BIS bcbs128; Investopedia).
- **Basel III** — **December 2010** (BIS bcbs189); **CET1 minimum 4.5%**, Tier 1 6%, total 8% (bcbs189; multiple secondary confirmations); **CCB 2.5%** and **CCyB 0–2.5%** (BIS FSI summary *The capital buffers in Basel III*; BIS d541 buffers paper); **leverage ratio 3%**, final framework January 2014 (BIS bcbs270; Wikipedia Basel III); **LCR** (bcbs238, 2013) and **NSFR** (bcbs295, 2014) — liquidity side cross-referenced to [Treasury & ALM Guide](treasury_alm_guide.md).
- **G-SIB / D-SIB buffers** — BCBS framework RBC40: **buckets 1.0–3.5%** higher loss absorbency, 130bp designation cutoff, 100bp bucket ranges; D-SIB up to 2.5%; G-SIB **leverage buffer = 50% of the surcharge** (RBC40; US 12 CFR 217.11).
- **Basel IV** — **7 December 2017** finalisation (BIS bcbs424); **output floor 72.5%** of standardised RWA, phased from 50% (PwC; RegPulse; pomegra.io summaries of bcbs424); **IRB restrictions** (no IRB for large financials/equities; PD floors 5bp/10bp; LGD floors); **CVA revision** (SA-CVA/BA-CVA); **SMA** replacing AMA (BIS d355; arxiv 2502.00962; risk.net SMA topic).
- **FRTB** — **January 2019** revised market-risk standard (BIS d457, *Minimum capital requirements for market risk*), effective 1 January 2022 (BIS; Sia Partners).
- **EU CRR3** — **Regulation (EU) 2024/1623**, applies from **1 January 2025** (EBA CRR3/CRD6 dashboard; European Commission Basel III page); output floor 50% → 72.5% **by 2030**; FRTB IMA deferred to 1 January 2026 (betterregulation.com CRR3 summary); EC impact estimate +6.4–8.4% minimums by 2030 (EY summary of the EC assessment).
- **UK Basel 3.1** — PS17/23 (Sep 2023), PS9/24 (Sep 2024, originally effective 1 Jan 2026), PS7/25 (SME/infrastructure adjustments), **PS1/26** (20 January 2026, finalising the framework); **delay to 1 January 2027 announced January 2025** explicitly pending US clarity (Bank of England news release; PwC UK summaries).
- **US** — July 2023 "Basel III endgame" proposal; never finalised; **re-proposed 19 March 2026** ("Take Two", three FRB/OCC/FDIC proposals, ~$87.7bn CET1 relief vs 2023 per press summaries — Fox Rothschild; Freshfields; EY; Federal Reserve press release bcreg20260319a). **Status flagged: no final rule, no effective date.**
- **MAS** — revised **MAS Notice 637** published 20 September 2023 (PPD 10/2023), **effective 1 July 2024** with phased implementation (PPD 08/2023; Allen & Gledhill; Rajah & Tann); **MAS minimums CET1 6.5% / Tier 1 8% / total 10%** (per MAS Notice 637 guidance summaries — flagged, verify current Notice wording); DBS capital add-on ~S$1.6bn per public reporting (flagged figure; [Banks in Singapore Guide](banks_in_singapore_guide.md)).
- **TLAC** — FSB *Principles on Loss-absorbing and Recapitalisation Capacity of G-SIBs in Resolution* and **TLAC Term Sheet, 9 November 2015**; minimums **16% RWA / 6% leverage exposure**, fully-loaded 18% RWA / 6.75% (FSB; BIS FSI TLAC summary; Regnology — the 18%/6.75% calibration is flagged).
- **AT1/CoCos** — perpetual, callable ≥5y, non-cumulative cancellable coupons, loss absorption at trigger (5.125% typical EU) and PONV (Euromoney; Investopedia; Segal Marco).

**Flagged / not fully verifiable from the sources used:** the exact current French CCyB level and Crédit Agricole's G-SIB bucket (bucket lists move annually); the annual phase-in percentages of the MAS output floor and the Notice 638 branch-capital detail; the US re-proposal's precise capital effects (press-reported relief figure); the SMA's loss-multiplier treatment as consolidated in the Basel Framework (OPE25); TLAC 18%/6.75% phase-in specifics; and all worked-example figures (§9), which are illustrative by design. Where a fact could not be confirmed, the text says so inline.

**Method.** Facts were verified against primary sources (BIS/BCBS publications and framework chapters, FSB standards, EBA/EC pages, Bank of England publications, Federal Reserve releases, MAS-focused law-firm circulars) and authoritative secondary summaries, using targeted web searches conducted for this guide (December 2017 publication date, 72.5% floor, FRTB January 2019, SMA replacement of the AMA, CRR3 1 January 2025 application, UK delay to 1 January 2027, US 19 March 2026 re-proposal, MAS Notice 637 1 July 2024 effectiveness, leverage ratio 3%, G-SIB buffer buckets, TLAC term sheet, AT1/CoCo features, MDA conservation grid). Where two sources disagreed or a primary text was not reachable, the claim is flagged rather than asserted. This guide deliberately does not cite specific figures from any bank's public disclosures; the worked example's numbers are pedagogical constructions.

---

## 12. Glossary

| Term | Definition |
|---|---|
| **Basel I** | The 1988 Accord: first global minimum capital standard — 8% of risk-weighted assets, flat risk weights, credit risk only |
| **Basel II** | The 2004 Revised Framework: three pillars (minimum capital, supervisory review, market discipline); SA and IRB credit approaches; first op-risk charge |
| **Basel III** | The December 2010 GFC-response package: higher-quality capital (CET1 4.5%), buffers, leverage ratio 3%, LCR/NSFR; phased in 2013–2019 |
| **Basel IV** | Market name for the December 2017 *finalisation of Basel III* (bcbs424): output floor 72.5%, IRB restrictions, CVA revision, SMA; FRTB finalised Jan 2019 |
| **Basel III endgame** | US term for the agencies' implementation of the finalisation (July 2023 proposal; re-proposed March 2026 — pending) |
| **CET1** | Common Equity Tier 1 — ordinary shares, retained earnings, reserves minus deductions; the highest-quality, first-loss capital |
| **AT1** | Additional Tier 1 — perpetual, going-concern loss-absorbing instruments (e.g. CoCos); feeds Tier 1 |
| **CoCo** | Contingent convertible — AT1 bond that converts to equity or is written down at a trigger (typically CET1 < 5.125% and/or PONV) |
| **Tier 2** | Gone-concern capital: ≥5-year subordinated debt, amortising, absorbing losses in resolution |
| **Buffer** | Capital held above the minimums (CCB, CCyB, G-SIB, D-SIB); breach restricts distributions (MDA) rather than triggering insolvency |
| **Conservation buffer** | The fixed 2.5% CET1 buffer (CCB); capital within it is usable but distributions are scaled back |
| **Countercyclical** | The 0–2.5% (CCyB) buffer set by jurisdictions against credit growth; released in downturns |
| **G-SIB** | Global systemically important bank — FSB/BCBS-designated; 1.0–3.5% higher-loss-absorbency buffer |
| **D-SIB** | Domestic systemically important bank — national designation; 0–2.5% buffer (EU: O-SII) |
| **RWA** | Risk-weighted assets — exposures × risk weights, the denominator of all capital ratios |
| **Credit risk** | Risk of loss from borrower default; the dominant RWA driver for commercial banks |
| **SA** | Standardised Approach — supervisor-prescribed risk weights (for credit, and the FRTB-SA for market risk) |
| **IRB** | Internal Ratings-Based approach — bank models for PD/LGD/EAD (F-IRB/A-IRB), restricted by Basel IV |
| **Market risk** | Risk of loss from market-price movements (trading book); charged under FRTB |
| **FRTB** | Fundamental Review of the Trading Book — the revised market-risk framework (Jan 2019 standard): ES 97.5%, NMRF, DRC, PLA, revised boundary |
| **Operational risk** | Risk of loss from failed processes, people, systems, or external events |
| **AMA** | Advanced Measurement Approach — Basel II's internal-models op-risk approach; removed by Basel IV |
| **SMA** | Standardised Measurement Approach — the formulaic op-risk capital method (BI × coefficients × ILM) replacing the AMA |
| **Leverage ratio** | Tier 1 ÷ leverage exposure ≥ 3% — the non-risk-based backstop (finalised 2014) |
| **Output floor** | Basel IV constraint: modelled RWA cannot fall below 72.5% of standardised RWA |
| **CVA** | Credit Valuation Adjustment risk — the charge for counterparty-credit-spread moves on derivatives (SA-CVA/BA-CVA) |
| **ICAAP** | Internal Capital Adequacy Assessment Process — the bank's own Pillar 2 capital assessment |
| **SREP** | Supervisory Review and Evaluation Process — the supervisor's assessment concluding in P2R/P2G |
| **TLAC** | Total Loss-Absorbing Capacity — FSB minimum bail-inable capacity for G-SIBs (16% RWA / 6% leverage exposure, 18%/6.75% fully-loaded) |
| **MREL** | Minimum Requirement for Own Funds and Eligible Liabilities — the EU/UK resolution equivalent for banks in resolution scope |
| **CRR3** | Regulation (EU) 2024/1623 — the EU's transposition of the Basel III finalisation, applying from 1 January 2025 |
| **Pillar 1** | Minimum capital requirements — the ratio machinery (RWA, minimums, buffers) |
| **Pillar 2** | Supervisory review — ICAAP/SREP, P2R/P2G, bank-specific capital |
| **LCR** | Liquidity Coverage Ratio — HQLA ÷ 30-day net outflows ≥ 100% (see [Treasury & ALM Guide](treasury_alm_guide.md)) |
| **NSFR** | Net Stable Funding Ratio — available stable funding ÷ required stable funding ≥ 100% (see [Treasury & ALM Guide](treasury_alm_guide.md)) |
| **GFC** | Global Financial Crisis (2007–2009) — the event that produced Basel III |

---

## 13. Cross-References in This Series

| Guide | Why it matters here |
|---|---|
| [Treasury & ALM Guide](treasury_alm_guide.md) | The **liquidity half of Basel III** — LCR/NSFR/IRRBB; read with this guide for the full Basel III/IV picture |
| [Risk Management Models Guide](risk_management_models_guide.md) | The **model machinery** under RWA: PD/LGD/EAD and IRB, VaR/ES and FRTB, SMA formula, model risk |
| [Capital Markets Architecture Guide](capital_markets_architecture_guide.md) | The **FRTB/market-risk angle** on the trading book |
| [Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md) | §2 regulatory context; the compliance-systems overlay on capital reporting |
| [Core Banking Systems Guide](core_banking_systems_guide.md) | The **RWA-data angle** — exposures and the data estate behind the ratios |
| [Banks in Singapore Guide](banks_in_singapore_guide.md) | The **MAS implementation angle** (Notice 637/638, the DBS capital add-on) |
| [Crédit Agricole Software Systems Guide](credit_agricole_software_systems_guide.md) | The **Cymbal Bank context** behind the §9 worked example |
| [RegTech Guide](regtech_guide.md) | The **reporting angle** — COREP, MAS returns, disclosure |
| [Universal Banking Model Guide](universal_banking_model_guide.md) | Where capital sits in the universal-bank business model |
| [Singapore Fintech & Payments Guide](singapore_fintech_payments_guide.md) | The payments-liquidity interplay with capital (light) |
| [Risk Data Aggregation Guide](../technology/risk_data_aggregation_guide.md) | **BCBS 239** — the data-aggregation discipline behind credible RWA |
| [Business Case Development Guide](../management/business_case_development_guide.md) | The cost-of-capital lens on system investments (light) |
| [MBA Body of Knowledge Guide](../management/mba_body_of_knowledge_guide.md) | The finance vocabulary (light) |
