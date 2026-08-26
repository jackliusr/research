# Treasury & Asset-Liability Management (ALM): The Bank-Treasury Function — A Comprehensive Guide

*The dedicated deep-dive on the bank-treasury function: how a bank funds itself, prices internal funds, manages liquidity and interest-rate risk, and guards the balance sheet. This is the treasury/ALM anchor of the bank-systems series: it cross-references the [Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md) (§6, the liquidity-risk-and-treasury-systems frame), the [Risk Management Models Guide](risk_management_models_guide.md) (the VaR/FRTB and liquidity-model angle), the [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md) and [Nasdaq Calypso Guide](nasdaq_calypso_guide.md) (the treasury-systems layer), the [Core Banking Systems Guide](core_banking_systems_guide.md) (the deposit-funding/NMD angle), the [Universal Banking Model Guide](universal_banking_model_guide.md) (the funding model), the [Capital Markets Architecture Guide](capital_markets_architecture_guide.md) (the treasury-desk angle), the [Banks in Singapore Guide](banks_in_singapore_guide.md) (the SG regulatory context), the [Crédit Agricole Software Systems Guide](credit_agricole_software_systems_guide.md) (the Cymbal Bank context used in the worked example), the [Singapore Fintech & Payments Guide](singapore_fintech_payments_guide.md) (the payments-liquidity angle), the [Event Stream Processing Guide](../technology/event_stream_processing_guide.md) (the real-time treasury angle), the [Monolith to Microservices Guide](../technology/monolith_to_microservices_guide.md) (the modernization angle), the [Business Case Development Guide](../management/business_case_development_guide.md) (the treasury-system-investment angle), and the [MBA Body of Knowledge Guide](../management/mba_body_of_knowledge_guide.md) (the finance vocabulary). Where public evidence runs out, this guide says so — the verification convention in §11 is enforced line by line.*

> **Context:** Banking / Risk & Treasury — Asset-Liability Management, Funds Transfer Pricing (FTP), Liquidity Risk (LCR/NSFR), IRRBB (BCBS 368), ALM Models (gap, duration, EVE, NII, NMD behavioural), Treasury Operations (money markets, FX, cash management), Treasury Systems (Murex MX.3, Nasdaq Calypso, FIS), ALCO governance, Singapore context (MAS Notice 649, SORA), Cymbal Bank worked example.

**How to read this guide.** The ten sections form a complete arc: the function (§1), the discipline (§2), the internal pricing (§3), the two regulated risks (§4–5), the models (§6), the operations (§7), the systems (§8), and a worked ALM design (§9), closing with a one-page summary (§10), the verification record (§11), the glossary (§12), and the series cross-references (§13). Readers who want the *design* should read §1, §3, §6, and §9; readers who want the *regulatory* picture should read §4, §5, and §11; readers who want the *architecture* should read §8 and §13. Every section is written to stand alone, with cross-references where the series has more depth.

## Table of Contents

1. [The Treasury Overview](#1-the-treasury-overview)
2. [The ALM](#2-the-alm-asset-liability-management)
3. [The FTP](#3-the-ftp-funds-transfer-pricing)
4. [The Liquidity Risk](#4-the-liquidity-risk-lcr-and-nsfr)
5. [The IRRBB](#5-the-irrbb-interest-rate-risk-in-the-banking-book)
6. [The ALM Models](#6-the-alm-models)
7. [The Treasury Operations](#7-the-treasury-operations)
8. [The Treasury Systems](#8-the-treasury-systems)
9. [The Worked Example](#9-the-worked-example-an-alm-design-for-a-cymbal-bank)
10. [The Summary: One Page](#10-the-summary-one-page)
11. [Verification Notes and Sources](#11-verification-notes-and-sources)
12. [Glossary](#12-glossary)
13. [Cross-References in This Series](#13-cross-references-in-this-series)

---

## 1. The Treasury Overview

### 1.1 What the Bank Treasury Is

**Treasury** is the bank function that manages the institution's funding, liquidity, cash, and financial risks at the firm-wide level. Where the trading desks manage *customer and market risk positions*, the treasury manages the **balance sheet itself**: how the bank raises money (funding), at what cost, for how long, in which currencies, and how that money is deployed.

Every bank is, at its core, a maturity-transformation machine: it takes short-dated, liquid liabilities (deposits, interbank borrowings) and transforms them into longer-dated, illiquid assets (loans, bonds). That transformation is the source of the bank's net interest margin — and the source of its fragility. The treasury function is the *control room* of that machine: it decides the funding mix, prices internal funds, holds the liquidity buffer, hedges interest-rate and FX risk, and reports the risk picture to the Asset & Liability Committee (ALCO) and to regulators.

The function sits between three constituencies:

- **The business lines** (retail, corporate, markets): they originate assets (loans) and liabilities (deposits) at customer rates. Treasury supplies or absorbs their funding through internal transfer pricing.
- **The markets** (interbank, repo, bond, FX): treasury executes funding and hedging transactions in the wholesale markets on behalf of the whole bank.
- **The regulators** (MAS in Singapore, ACPR/ECB for the Cymbal Bank group, the Basel Committee as standard-setter): treasury is the owner of the bank's liquidity-risk and IRRBB frameworks, the LCR/NSFR metrics, and the liquidity-buffer composition.

### 1.2 The Treasury Mandate

The treasury's mandate decomposes into five interlocking jobs:

1. **Funding the balance sheet** — raising stable, cost-efficient funds: customer deposits (retail and corporate), interbank money-market borrowings, certificates of deposit (CDs), commercial paper (CP), covered bonds, senior unsecured issuance, and (for the group) subordinated capital. The funding plan is sized against the asset pipeline and the maturity profile of the existing book.
2. **Managing liquidity** — maintaining the liquidity buffer (high-quality liquid assets, HQLA), projecting 30-day cash flows for the LCR, structuring longer-dated funding for the NSFR, and running the contingency funding plan (CFP) for stress.
3. **Managing interest-rate risk in the banking book (IRRBB)** — measuring and hedging the effect of rate moves on net interest income (NII) and on the economic value of equity (EVE), under the BCBS 368 framework (§5).
4. **Managing FX and funding-currency risk** — matching the currency of assets with the currency of funding, and hedging residual mismatches (cross-currency swaps are the treasury's core hedging instrument).
5. **Pricing internal funds (FTP)** — crediting deposit-gathering units and charging lending units at a transfer rate that isolates the interest-rate and liquidity content of the funds, so that business-line P&L reflects only customer margin, not the cost of the bank's maturity transformation (§3).

### 1.3 The Treasury in the Bank Organisation

Treasury is a *shared service with a market desk*. In a universal bank it typically reports through the Chief Financial Officer or a dedicated Treasurer, with the following structure:

- **ALCO (Asset & Liability Committee)** — the governance body, usually chaired by the CFO or CEO, that sets the risk appetite for liquidity and IRRBB, approves the FTP curve and the funding plan, and reviews ALCO pack reporting. It meets monthly (or more often in stress).
- **The funding desk** — executes money-market and capital-market funding: deposits, CP/CD, repos, bond issuance.
- **The ALM desk** — runs the interest-rate and FX hedging books, executes swaps, and manages the maturity gap (often under the front-office umbrella but with a balance-sheet mandate, distinct from the customer trading desks).
- **Liquidity management** — runs the cash-flow projections, the HQLA portfolio, and the stress-testing/CFP machinery.
- **FTP / internal pricing** — maintains the transfer-pricing curve and the product FTP rates, arbitrates product-pricing disputes with the business lines.
- **Treasury operations (back office)** — settles money-market, FX, and bond transactions; manages nostro accounts and payments (often shared with a payments function).

The treasury is deliberately kept *separate from the customer-facing businesses*: its P&L is the "residual" — the cost of the maturity, liquidity, and rate transformation — which is exactly the slice of bank economics the businesses must not be allowed to price away. This separation is enforced by FTP (the internal pricing of funds) and by limits (funding concentration limits, gap limits, LCR/NSFR limits).

### 1.4 The Overview Table

| Aspect | Description |
|---|---|
| **Definition** | The bank function that manages funding, liquidity, cash, and balance-sheet financial risk at the firm level |
| **Core economic role** | Maturity transformation: fund long-dated illiquid assets with shorter-dated liabilities, earning net interest margin |
| **Key mandates** | Fund the balance sheet; hold liquidity; manage IRRBB; manage FX/funding-currency risk; price internal funds via FTP |
| **Main governance body** | ALCO (Asset & Liability Committee) — sets risk appetite, approves FTP and funding plan |
| **Internal counterparty** | Business lines (lending, deposits) — treasury is their internal "bank of the bank" |
| **External counterparties** | Interbank money market, repo market, bond investors, central bank, FX market |
| **Regulatory owner of** | LCR, NSFR, MLA (SG), IRRBB metrics (ΔEVE, ΔNII), liquidity buffer composition, CFP |
| **Distinct from** | The trading desks (customer/market positions) and the core banking system (customer accounts — see the [Core Banking Systems Guide](core_banking_systems_guide.md)) |
| **Core systems** | Treasury workstations and ALM platforms: Murex MX.3, Nasdaq Calypso, FIS (see §8) |
| **Key risk it manages** | Liquidity risk (short-term survival) and IRRBB (long-term earnings/value) |
| **Success metric** | Stable, diversified, cost-efficient funding; LCR/NSFR comfortably above minimums; NII/EVE within appetite; zero liquidity incidents |

### 1.5 Treasury in a CIB vs a Retail Bank

The treasury function's shape follows the funding base:

- **Retail/universal banks** (DBS, UOB, OCBC — cross-ref the bank guides) have a large, stable, *cheap* deposit base: NMD behavioural cores are deep, the LCR/NSFR are structurally easy to satisfy, and the ALM problem is mostly *rate* risk (what to do with sticky cheap deposits in a rising-rate world) plus the pricing of the deposit franchise through FTP.
- **Corporate and investment banks** (Cymbal Bank, cross-ref [Universal Banking Model Guide](universal_banking_model_guide.md)) have *no retail deposits*: funding is wholesale (interbank, CP/CD, bonds) and corporate deposits, so the LCR and NSFR are structurally harder, the funding plan is a first-order strategic document, and the ALM problem is mostly *liquidity and term* risk. The HQLA buffer is bigger per unit of assets, and the marginal funding cost is discovered daily in the money markets rather than set by the deposit franchise.

### 1.6 The Treasury P&L

The treasury's P&L is the *residual* of the whole bank: customer-spread income is booked by the business lines; the treasury books the FTP interest it charges/credits them, its actual funding and hedging costs, and the net of the two — the **transformation margin**. Three lines dominate:

1. **FTP net interest** — interest received on FTP charges to assets minus interest paid on FTP credits to liabilities.
2. **Funding and buffer cost** — actual wholesale funding costs (interbank, CP/CD, bond issuance) and the carry cost of the HQLA buffer (the buffer earns less than its funding — the "cost of liquidity" the bank pays for the LCR).
3. **Hedging P&L** — the mark-to-market and accrual of the ALM hedging books (swaps, FRAs, cross-currency), net of hedge accounting.

ALCO reads the treasury P&L not for its absolute size but for its *drivers*: a growing buffer cost means the LCR is being financed expensively; a growing hedging loss means the gap is being hedged late; a growing transformation margin means the businesses are ceding economics to treasury — each is a governance signal, not just a number.

---

## 2. The ALM (Asset-Liability Management)

### 2.1 What ALM Is

**Asset-Liability Management (ALM)** is the discipline of managing the *joint* behaviour of the asset side and the liability side of the balance sheet so that the bank's earnings and capital are protected against adverse moves in interest rates, liquidity conditions, and funding availability. It is the analytical and managerial core of the treasury function — the set of models, limits, hedges, and governance that turns "the balance sheet" from an accounting artifact into a risk-managed portfolio.

ALM answers three questions continuously:

1. **Are we liquid?** — Will cash inflows cover cash outflows over the next 30 days (LCR) and is our funding profile stable over one year (NSFR)?
2. **Are we rate-resilient?** — What happens to NII over the next 12 months, and to EVE over the life of the book, if rates move (parallel, steepener/flattener, short-rate shocks)?
3. **Are we funding ourselves well?** — Is the funding mix (deposits vs wholesale vs capital) diversified, stable, and appropriately priced (FTP)?

ALM is sometimes described as "balance-sheet management" or "structural risk management" — *structural* because it concerns the structure of the balance sheet (maturities, repricing dates, options embedded in products) rather than the mark-to-market trading positions that the markets desks run.

### 2.2 The Balance-Sheet View

A bank balance sheet seen through ALM eyes is a collection of **cash-flow streams with repricing and maturity dates**, not a static snapshot:

- **Assets**: loans (fixed and floating rate, with optionality: prepayment, drawdown), securities (HQLA and others), interbank placements, trading assets. Each has a repricing date, a maturity date, and a rate basis (e.g., SORA for SGD assets, SOFR for USD).
- **Liabilities**: deposits (retail and corporate, with *behavioural* maturities for non-maturity deposits — see §6.5), interbank borrowings, CP/CD, issued bonds, and capital. Each has a repricing/maturity profile and a stickiness (how fast it can leave).
- **Equity**: the residual — no contractual maturity, no contractual rate, which makes it the ultimate buffer for both earnings and value.

The ALM problem is that the two sides never match naturally: loans are longer and stickier than deposits; floating assets reprice off different indices than the liabilities that fund them; customers hold options (prepay, withdraw, redeem) that the bank does not price for fully. ALM's job is to measure the mismatch (gap, duration, EVE, NII sensitivity), price it (FTP), and where it exceeds appetite, hedge it (swaps, FRAs, futures) or re-fund it.

### 2.3 ALCO and the ALM Governance Cycle

The ALCO cycle is the operating rhythm of ALM:

1. **Measure** — monthly (or weekly) production of the ALCO pack: gap report, duration report, ΔEVE/ΔNII under shocks, LCR/NSFR projections, funding-concentration report, FTP margins by product.
2. **Decide** — ALCO sets/confirms limits (gap limits per bucket, LCR/NSFR buffers above minimum, funding concentration limits, issuer limits), approves the FTP curve, and sanctions hedging actions.
3. **Act** — the ALM desk executes the hedges and the funding desk adjusts issuance; the liquidity team re-balances the HQLA portfolio.
4. **Report** — to the board risk committee and to regulators (MAS reporting under Notice 649; IRRBB disclosures; Pillar 3).

The ALM function is also the owner of the **stress-testing framework** for the balance sheet: the liquidity stress tests (idiosyncratic, market-wide, combined scenarios — the Basel "liquidity-monitoring tools" and the SG framework), the IRRBB shock scenarios (the BCBS 368 six shocks), and the combined stress used for ICAAP/ILAAP-type processes.

### 2.4 The ALM Table

| Aspect | Description |
|---|---|
| **Definition** | The discipline of managing assets and liabilities jointly to protect earnings, value, and liquidity |
| **Aliases** | Balance-sheet management; structural risk management; funds management (in some banks) |
| **Object of management** | The whole banking book: loans, deposits, securities, funding, equity |
| **Key metrics** | Gap profile; duration (modified/key-rate); ΔEVE; ΔNII (12-month); LCR; NSFR; funding concentration |
| **Key models** | Gap analysis, duration, EVE, NII simulation, NMD behavioural modelling (§6) |
| **Key hedges** | Interest-rate swaps, FRAs, futures, cross-currency swaps, bond issuance |
| **Governance** | ALCO; board risk committee; risk appetite statement; limit framework |
| **Regulatory anchors** | Basel III LCR/NSFR; BCBS 368 (IRRBB); MAS Notice 649; MAS Notice 637 risk-management expectations |
| **System owners** | ALM platforms and treasury workstations (§8); data feeds from core banking and markets systems |
| **Horizon split** | Short-term survival (liquidity: 30 days) vs medium-term earnings (NII: 12–24 months) vs long-term value (EVE: full maturity profile) |
| **Central tension** | Profitability of maturity transformation vs its risk — priced and governed, never eliminated |

### 2.5 A Short History: From Funds Management to BCBS 368

ALM as a named discipline is young. The line of development:

- **1960s–70s**: "funds management" — banks began explicitly managing the liability side (the liability-management revolution) as CD issuance and the Eurodollar market made wholesale funding a strategic choice. Asset/liability *matching* concepts arrived with the savings-and-loan crisis in the US, where maturity mismatch destroyed institutions.
- **1980s–90s**: duration and gap analysis became standard; the 1988 Basel Accord (Basel I) fixed capital but left liquidity national; the early 1990s produced the first ALM software and the formalisation of FTP as a management-accounting discipline.
- **2000s**: the 2004 BCBS *Principles for the Management and Supervision of Interest Rate Risk* (BCBS 108) set supervisory expectations; ALM remained a Pillar 2 / national-discretion affair, with EVE/NII measurement practised by sophisticated banks.
- **2007–09**: the Global Financial Crisis proved liquidity was an *existential* risk, not a footnote — Northern Rock, Lehman, and the interbank freeze made funding stability the first item on every board agenda.
- **2010s**: Basel III added the LCR (2010, revised 2013, effective 2015) and NSFR (2014, effective 2018); BCBS 368 (April 2016) replaced BCBS 108 with the standardised IRRBB framework; FTP matured into matched-maturity best practice; NMD behavioural modelling became a regulated input.
- **2020s**: IRRBB/CSRBB implementation in the EU (CRD V/CRR2), real-time intraday liquidity monitoring, cloud/SaaS treasury platforms (MXGO, SaaS Calypso), and model-risk discipline applied to ALM models — the modern, regulated, data-intensive ALM of this guide.

---

## 3. The FTP (Funds Transfer Pricing)

### 3.1 What FTP Is

**Funds Transfer Pricing (FTP)** is the internal pricing mechanism through which treasury "buys" funds from liability-producing businesses and "sells" funds to asset-producing businesses. Every asset (loan) is charged an internal funding cost; every liability (deposit) is credited an internal funding return; the difference between the customer rate and the FTP rate is the business's *spread*, and everything else — the cost of the maturity mismatch — accrues to treasury as the residual.

FTP exists to solve a measurement problem: if lending units paid only the *average* cost of funds, they could book long-dated fixed-rate loans whose true funding cost (a long-dated funding rate) is higher than the average, and deposit units could gather hot money whose true value is lower than the average — and nobody would see the mispricing until the P&L surprise. FTP attaches a *term- and liquidity-adjusted* price to each unit of funds so that:

- **Lending units** see the true funding cost of the money they consume, at the right tenor.
- **Deposit units** see the true value of the money they raise, at the right tenor.
- **Treasury** owns the residual P&L of the maturity transformation — which is where interest-rate and liquidity risk live.
- **Product profitability** becomes comparable across products with wildly different maturities (a 3-month trade-finance facility vs a 30-year mortgage).

FTP is therefore not a pricing model for customers — it is a *management accounting* model whose output feeds product pricing, performance measurement, and ALCO decisions. The FTP rate is typically built as: **risk-free transfer curve rate (e.g., SORA OIS for SGD) + liquidity premium + option/behavioural adjustments**, per tenor and per product.

### 3.2 Pooling vs Matched-Maturity FTP

The two poles of FTP methodology are **pooling** and **matched-maturity** (with multiple-pool approaches in between):

**Pooling (single pool / average cost).** All funds are pooled; every asset pays and every liability earns the same blended average funding cost (e.g., weighted average cost of all liabilities). Simple, cheap to run, and easy to understand — but it is economically wrong in three ways: (1) it ignores *term*: a 5-year fixed loan is charged the same rate as a 3-month overdraft, so term risk is invisible; (2) it ignores *liquidity*: the term liquidity premium embedded in long funding is not priced; (3) it creates *cross-subsidies*: short-dated deposit units are over-credited and long-dated lending is under-charged (or vice versa), so the true economics hide in treasury. Multiple-pool FTP (e.g., a handful of tenor buckets: ≤1M, 1–3M, 3–6M, 6–12M, >1Y) is a coarse middle ground that captures some term but still misprices within buckets and ignores the liquidity premium.

**Matched-maturity FTP.** Every transaction is priced off the **transfer-pricing curve at the tenor of the product's (behavioural) maturity**: a 5-year fixed-rate loan is charged the 5-year transfer rate; a 3-month deposit is credited the 3-month transfer rate; a non-maturity deposit (current/savings account) is credited a *blended* rate reflecting its behavioural core and non-core decomposition (§6.5). Matched-maturity FTP is the industry best practice for sophisticated banks (and the approach regulators implicitly expect in IRRBB and pricing frameworks): it makes term and liquidity risk explicit, removes cross-subsidies, and concentrates the maturity-transformation P&L in treasury where it is measured and governed.

The transfer curve itself matters as much as the method: banks typically use the **risk-free / OIS curve of the funding currency** (SORA for SGD, SOFR for USD, €STR for EUR) as the base, plus a **liquidity/term premium** overlay reflecting the bank's own funding spreads at each tenor (derived from its issuance curve and CD/CP levels), plus product-specific adjustments (e.g., for prepayment options on loans, or for the embedded call on non-maturity deposits).

### 3.3 FTP Mechanics: The Two-Sided Flow

The mechanics of matched-maturity FTP, step by step:

1. **The curve is set.** Treasury publishes a daily FTP transfer curve per currency (e.g., SGD: SORA OIS + liquidity premium by tenor: 1M, 3M, 6M, 1Y, 2Y, 3Y, 5Y, 7Y, 10Y).
2. **Assets are charged.** A corporate lending unit books a 5Y fixed SGD loan at 3.40%. The 5Y transfer rate is 2.60%. The unit's FTP spread is **+0.80%** — the customer margin, before operating costs and expected loss. The 2.60% is remitted to treasury as the internal funding charge.
3. **Liabilities are credited.** A deposits unit raises SGD corporate demand deposits. Behavioural modelling says the core (stable) portion behaves like 3Y funding (credited at 2.45% on that portion) and the non-core portion like 1M funding (credited at 1.80%). The blended credit is, say, 2.20%. The unit's FTP spread on a 0.10%-paying account is **+2.10%**.
4. **Treasury takes the residual.** Treasury funds the 5Y loan with a mix of the 3Y-behaving core deposits and wholesale funding, and runs the resulting gap; its P&L = loan FTP income (2.60%) − deposit FTP cost (2.20%) − actual wholesale funding costs and hedge costs. That residual is the price of maturity transformation — monitored by ALCO against the risk it consumes (gap limits, EVE/NII sensitivity).
5. **Product pricing uses the output.** Business units price customers as *FTP + target margin + costs + expected loss*; ALCO can see per-product FTP spreads in the ALCO pack and challenge mispriced products.

### 3.4 The FTP Table

| Method | Mechanics | Strengths | Weaknesses | Used by |
|---|---|---|---|---|
| **Single pool (pooling)** | One blended average funding cost applied to all assets and liabilities | Simple, cheap, easy to communicate | No term or liquidity pricing; hides cross-subsidies; treasury residual absorbs all mismatch silently | Small banks, legacy implementations, early-stage FTP |
| **Multiple pool** | A few tenor buckets; rate per bucket | Captures coarse term structure; simple upgrade path | Mispricing within buckets; no liquidity premium; arbitrary bucket edges | Mid-size banks; transitional states |
| **Matched-maturity** | Every product priced off the transfer curve at its behavioural tenor; curve = RFR (SORA/SOFR/€STR) + liquidity premium + product adjustments | Term and liquidity risk explicit; no cross-subsidy; treasury owns true transformation P&L; enables EVE/NII attribution | Requires a curve infrastructure, behavioural models, data quality; governance-heavy | Sophisticated universal banks (Cymbal Bank), all major CIBs; best practice under IRRBB/LCR discipline |
| **Curve components** | RFR base + term liquidity premium (from own issuance/CD spreads) + optionality/behavioural adjustment | Transparent, market-anchored | Premium estimation is judgemental; curve changes flow to business P&L | Standard practice in matched-maturity banks |
| **FTP governance** | ALCO approves curve, methodology, product adjustments; independent validation of behavioural models | Accountability; consistency with risk appetite | Slow to change; model-risk scrutiny | All banks with a real ALM function |

### 3.5 FTP Pitfalls and Governance

FTP looks mechanical but fails in practice in recurring ways:

- **Curve fights.** Business lines dispute the liquidity premium ("your curve says our 5Y loan costs 3.10% but we can fund at 2.80%"). The answer is governance, not arithmetic: the premium comes from the bank's *own* issuance curve, is set by ALCO, and is reviewed periodically — the dispute is a pricing conversation, not a model bug.
- **Behavioural arbitrage.** If the NMD model is generous (long cores, low pass-through), the deposit franchise books fat FTP spreads that the treasury residual quietly eats. This is why the same behavioural assumptions must feed FTP, LCR/NSFR, and IRRBB (§6.7) and why the model is independently validated.
- **Product optionality uncharged.** Prepayment options on loans and the free option in NMD repricing are real costs; FTP that ignores them subsidises the product and misleads ALCO about the transformation P&L. Matched-maturity FTP should carry an optionality adjustment per product family.
- **Cross-currency gaps.** FTP is set per currency; a branch funding SGD loans with USD swaps must charge the *all-in* SGD funding cost including cross-currency basis, or the FX desk's cost silently lands in the treasury residual.
- **FTP as a pricing oracle.** FTP is management accounting, not customer pricing: the customer rate is FTP + target margin + costs + expected loss, and the *target margin* is a commercial decision — FTP just makes the margin visible. Banks that confuse the two either misprice customers or lose the discipline entirely.

Governance that works: ALCO owns the curve and the methodology; an FTP committee (treasury + finance + business representation) adjudicates product-level rates; model validation signs off on behavioural inputs; and the residual P&L is explained monthly to ALCO line by line.

---

## 4. The Liquidity Risk (LCR and NSFR)

### 4.1 The Liquidity Problem

**Liquidity risk** is the risk that the bank cannot meet its payment obligations as they fall due without incurring unacceptable losses — the risk of *not having cash when it is needed*. It is the bank's existential risk: a solvent bank can fail in days if creditors lose confidence and funding runs, as the 2007–09 crisis demonstrated (Northern Rock's run in September 2007, the interbank freeze of 2008, the collapse of funding-dependent institutions). Because liquidity is about *cash flows and confidence* rather than capital, it is measured and regulated separately from solvency — this is why the Basel III framework contains a *liquidity* pillar in addition to capital.

Basel III introduced two quantitative liquidity standards, both developed by the Basel Committee on Banking Supervision (BCBS) and both now fully phased in:

### 4.2 The LCR — Liquidity Coverage Ratio (30-day survival)

**What it is.** The LCR promotes the short-term resilience of a bank's liquidity risk profile by ensuring it has enough **high-quality liquid assets (HQLA)** to survive a 30-day stress scenario:

```
LCR = HQLA / Total net cash outflows over 30 days  ≥  100%
```

**The history (verified).** The LCR was first published by the BCBS in **December 2010** (BCBS 188, "Basel III: International framework for liquidity risk measurement, standards and monitoring", with the January 2013 revisions in BCBS 238). The requirement became **effective on 1 January 2015** — but with a **transitional phasing**: the minimum started at **60% of the final standard in 2015 and rose by 10 percentage points per year, reaching the full 100% on 1 January 2019**. (Some jurisdictions, including Singapore via MAS Notice 649, applied the full 100% to their significant banks from the 2015 start date.)

**Mechanics.** The numerator, HQLA, is a portfolio of assets that can be converted into cash quickly and with little loss even in stress, divided into:

- **Level 1** (no haircut, unlimited): cash, central-bank reserves, and high-rated government securities (e.g., Singapore Government Securities/SGS for SGD banks).
- **Level 2A** (15% haircut): high-rated sovereign/central-bank securities, and certain covered bonds.
- **Level 2B** (25–50% haircuts): lower-rated corporate debt, RMBS, and equities within caps.
- **Caps**: Level 2 may not exceed 40% of HQLA; Level 2B may not exceed 15% of HQLA.

The denominator, **total net cash outflows over 30 days**, is built from *run-off factors* applied to liabilities and commitments (how much of each funding source is assumed to leave in stress: e.g., stable retail deposits 3–5%, less-stable retail deposits 10%, operational deposits 25% (5% at L1), unsecured wholesale funding 5–100% by counterparty type, contingent commitments 5–10%) minus *cash inflows* (maturing loans, other maturing assets) — with inflows **capped at 75% of gross outflows**, so the bank must hold HQLA for at least 25% of its outflow even in the best case.

### 4.3 The NSFR — Net Stable Funding Ratio (one-year stability)

**What it is.** The NSFR promotes resilience over a **one-year horizon**: it requires the bank to fund its activities with enough **stable funding** so that a year-long funding disruption cannot force it into fire-sale asset liquidation:

```
NSFR = Available Stable Funding (ASF) / Required Stable Funding (RSF)  ≥  100%
```

**The history (verified).** The NSFR was finalised by the BCBS in **October 2014** (BCBS 295, "Basel III: the net stable funding ratio"). It became a **minimum standard on 1 January 2018** — replacing the earlier observation-period treatment — and was phased into national law from that date (in the EU via CRR from June 2021; in Singapore applied to banks via MAS requirements for reporting banks and, with disclosure, to domestic systemically important banks).

**Mechanics.** The numerator, **ASF**, is the amount of each liability class that is expected to remain available over a year, weighted by *ASF factors*: Tier 1/2 capital and liabilities with residual maturity ≥1 year: 100%; stable retail deposits: 95%; less-stable retail deposits: 90%; wholesale funding with maturity <1 year: 50%; short-dated interbank funding: 0%. The denominator, **RSF**, is the amount of stable funding each asset *requires*, weighted by *RSF factors*: cash and central-bank reserves: 0%; HQLA securities: 5%; loans to financial institutions <1 year: 10%; corporate loans: 50–65% by tenor; retail mortgages: 65%; other assets (fixed assets, goodwill, trading inventory): 100%; off-balance-sheet commitments: 5–10%.

The economic message of the NSFR is direct: **deposits are worth more than wholesale funding; long-dated funding is worth more than short-dated; and long-dated illiquid loans are the most expensive things to fund.** The NSFR and the LCR together push banks toward the funding structure ALM would choose anyway: stable customer deposits, term-matched wholesale funding, and a real HQLA buffer.

### 4.4 The Singapore Context (verified)

In Singapore, the **Monetary Authority of Singapore (MAS)** implements the Basel liquidity framework through **MAS Notice 649, "Minimum Liquid Assets (MLA) and Liquidity Coverage Ratio"** — applying to all banks in Singapore (full banks, wholesale banks, and locally incorporated reporting banks), issued under the Banking Act (Cap. 19). Notice 649 sets out:

- The **MLA requirement** — the minimum liquid assets each bank must hold (the Singapore-specific formulation of the liquidity buffer, computed on qualifying liabilities).
- The **LCR requirement** — banks must maintain an LCR of at least 100%; MAS required its significant banks to hold the **all-currency and SGD LCR at 100% from January 2015** (stricter than the BCBS phasing), and Singapore-based D-SIBs are subject to additional expectations (e.g., SGD LCR of 100% and all-currency 100%, plus NSFR disclosure requirements under the Notice 649 framework).
- The **reporting and governance framework** — liquidity-risk management expectations consistent with Basel Principle 11-14 on liquidity risk management, plus stress-testing expectations.

**A verification flag (important):** the widely repeated shorthand "MAS Notice 658 is the liquidity framework" is **incorrect — MAS Notice 658 is "Management of Outsourced Relevant Services for Banks"** (issued 11 December 2023, effective 11 December 2024, replacing Notice 634), the outsourcing-governance notice. The SG liquidity framework lives in **MAS Notice 649** (MLA + LCR). The NSFR is embedded in the MAS framework through the Notice 649 regime's D-SIB disclosure requirements; the standalone MAS NSFR notice number could not be independently confirmed in this research pass and is left unverified here (see §11).

### 4.5 The Liquidity Table

| Aspect | LCR | NSFR |
|---|---|---|
| **Purpose** | Short-term survival: 30-day stress survival | Long-term stability: one-year structural funding |
| **Formula** | HQLA / 30-day net cash outflows | ASF / RSF |
| **Minimum** | ≥ 100% (phased 60% → 100%, 2015–2019) | ≥ 100% (from 1 January 2018) |
| **BCBS source** | Dec 2010 (BCBS 188), revised Jan 2013 (BCBS 238) | Oct 2014 (BCBS 295) |
| **Effective date** | 1 January 2015 (full 100% by 1 January 2019) | 1 January 2018 |
| **Key inputs** | HQLA levels (L1/L2A/L2B, caps); run-off factors; inflow cap 75% | ASF factors by funding stability; RSF factors by asset liquidity |
| **Behaviour it drives** | Hold liquid assets; diversify short-term funding; match 30-day flows | Lengthen funding; favour deposits over wholesale; fund illiquid assets with term funding |
| **Singapore anchor** | MAS Notice 649 (MLA + LCR), 100% from 2015 for significant banks | MAS framework via Notice 649 D-SIB disclosures (notice number for standalone NSFR unverified — see §11) |
| **Who owns it** | Treasury liquidity team; reported to ALCO and MAS | Treasury funding/ALM; annual funding plan |
| **Systems** | LCR engine on treasury platform (Murex/Calypso §8); data from core, payments, markets | NSFR engine; same platforms |
| **Stress-testing link** | 30-day scenario is the regulatory stress; internal stresses are worse | Internal one-year stress + funding plan |

### 4.6 Liquidity Management Beyond the Ratios

The ratios are the *minimum*; a real liquidity framework has three more layers:

- **The Basel liquidity-monitoring tools.** The BCBS framework (introduced with the LCR in BCBS 238) requires banks to report: the contractual maturity mismatch ladder, concentration of funding by counterparty, concentration of funding by product/currency, the LCR by significant currency, market-related monitoring (market-wide stress indicators), and intraday liquidity usage. These are the early-warning instruments: a funding-concentration report that shows 40% of wholesale funding from ten counterparties is a crisis in formation, long before the LCR moves.
- **Stress testing and the contingency funding plan (CFP).** Banks run liquidity stress scenarios — idiosyncratic (a ratings downgrade, a run on one product), market-wide (an interbank freeze, a rate shock), and combined — and quantify the resulting cash-flow gap and the survival horizon (the number of days the bank can pay without new funding). The CFP operationalises the response: early-warning indicators with triggers, the escalation tree, the list of funding actions in order (repo the HQLA, sell assets, draw committed lines, access central-bank facilities), and the communications plan. The CFP is *tested* — a paper plan is a criticism waiting to happen.
- **Intraday liquidity.** The most operational liquidity risk of all: settlement obligations within the day (in Singapore, MEPS+) must be met even if the bank is fine at the close of business. Banks monitor intraday liquidity usage against limits, time their payments, and hold same-day capacity — increasingly a real-time data problem (cross-ref [Event Stream Processing Guide](../technology/event_stream_processing_guide.md)).

The governance frame is ALCO + limits: minimum LCR/NSFR buffers above the regulatory minimums (e.g., 120%/110% internal floors), funding-concentration limits, currency-mismatch limits, and CFP trigger levels — all owned by the treasurer, reviewed by ALCO, and reported to the board and to MAS.

---

## 5. The IRRBB (Interest-Rate Risk in the Banking Book)

### 5.1 What IRRBB Is

**Interest-rate risk in the banking book (IRRBB)** is the risk to the bank's *banking book* (everything that is not the trading book: loans, deposits, and the funding that supports them) arising from adverse movements in interest rates. Unlike the trading desks' market risk, IRRBB is *structural*: it arises from the repricing mismatches, basis differences, and embedded options that ALM manages.

IRRBB has three components:

- **Gap risk (repricing risk)** — assets and liabilities reprice at different times (a 5Y fixed loan funded by 3M deposits).
- **Basis risk** — assets and liabilities reference different rate indices (a loan at 3M SORA funded by deposits priced off the bank's own rate).
- **Option risk** — customers exercise embedded options: prepayments on loans, early withdrawals, the right to reprice demand deposits at will (the NMD "free option").

### 5.2 The Regulatory Framework: BCBS 368 (verified)

The international standard for IRRBB is **BCBS 368, "Interest rate risk in the banking book", published by the Basel Committee in April 2016** (the final standards were released on **21 April 2016**). BCBS 368:

- **Replaces the 2004 Principles** (BCBS 108, "Principles for the management and supervision of interest rate risk") and builds on the 2015 consultation (BCBS 319).
- Keeps IRRBB in **Pillar 2** (supervisory review) — there is no Pillar 1 capital charge in the global standard (some jurisdictions, notably the EU via CRD V/CRR2 from 2021, implemented a Pillar 1-style standardized add-on).
- Introduces a **standardised interest-rate shock framework**: six prescribed shock scenarios, applied to the bank's own cash-flow projections, producing **ΔEVE** (change in economic value of equity) and **ΔNII** (change in net interest income) metrics.
- Sets an **outlier test**: a bank whose maximum ΔEVE (across the six shocks) exceeds **15% of its Tier 1 capital** is an outlier, triggering supervisory dialogue (and in some jurisdictions a capital add-on).
- Requires **disclosure** of IRRBB metrics (Pillar 3 template IRRBB1, per BIS DIS70).

**The six standardised shocks** (per BCBS 368 Annex): (1) parallel up (+200 bp); (2) parallel down (−200 bp); (3) **steepener** (+200 bp at the short end, linearly declining to 0 at the long end); (4) **flattener** (−200 bp at the short end, linearly rising to 0 at the long end); (5) **short rates up** (+200 bp at the short end declining to 0 by 1 year); (6) **short rates down** (−200 bp at the short end rising to 0 by 1 year). Banks also report **ΔNII under a +200 bp parallel shock over a 12-month horizon** (and may report the −200 bp variant). The 200 bp magnitude is a *floor* — banks with larger actual rate volatility are expected to use larger shocks.

BCBS 368 also governs the **boundary** between banking and trading books, the treatment of **NMDs** (behavioural maturity — see §6.5), the treatment of **pass-through rates** (the fraction of a market rate move that is passed to NMD rates), and the **EVE vs NII trade-off** (a bank can hedge EVE while harming NII and vice versa — ALCO must own the trade-off explicitly). In Europe, the **EBA IRRBB Guidelines** (EBA/GL/2018/02, published 2018) extend the framework with detailed expectations for IRRBB and credit-spread risk in the banking book (CSRBB); MAS addresses IRRBB expectations for Singapore banks through MAS Notice 637 (risk management) and its supervisory expectations on interest-rate risk.

### 5.3 The IRRBB Table

| Aspect | Description |
|---|---|
| **Definition** | Risk to banking-book earnings and value from adverse interest-rate movements |
| **Components** | Gap (repricing) risk; basis risk; option risk (prepayments, NMD free option) |
| **Standard** | BCBS 368, April 2016 (published 21 April 2016; replaces BCBS 108 of 2004) |
| **Pillar** | Pillar 2 (supervisory review) in the global standard; EU CRD V/CRR2 added a standardized Pillar 1 add-on from 2021 |
| **Key metrics** | ΔEVE (6 shocks); ΔNII (12-month, +200 bp parallel minimum) |
| **Six shocks** | Parallel up/down (±200 bp); steepener; flattener; short rates up/down |
| **Outlier test** | Max ΔEVE > 15% of Tier 1 capital → outlier, supervisory dialogue |
| **Disclosure** | Pillar 3 template IRRBB1 (per BIS DIS70) |
| **Behavioural inputs** | NMD behavioural maturities; pass-through rates; prepayment models (§6.5) |
| **EVE vs NII** | The two metrics can move oppositely; ALCO must own the trade-off explicitly |
| **Regional extensions** | EBA/GL/2018/02 (EU, IRRBB + CSRBB); MAS expectations via Notice 637 |
| **Hedging instruments** | Interest-rate swaps, FRAs, futures, swaptions; hedge accounting per IFRS 9/hedge-accounting rules |
| **System owners** | ALM platforms (§8): EVE/NII engines, behavioural-model libraries, shock scenarios |

### 5.4 Measuring IRRBB in Practice

The measurement workflow, end to end:

1. **Cash-flow generation.** Every banking-book product is projected into a cash-flow stream: contractual flows (loans, term deposits, bonds) use their terms; NMDs use behavioural maturities; loans with prepayment options use prepayment models; floating-rate products reprice at their next repricing date. The result is a *base-case cash-flow ladder* per currency, with the rate index attached to each cash flow.
2. **Base valuation.** Discount the base-case flows at the risk-free/swap curve: the PV difference between assets and liabilities is the base EVE; the same cash flows drive a 12-month base NII projection.
3. **Shock and re-run.** Apply each of the six BCBS 368 shocks to the curve, re-run the cash flows (holding behaviour constant, per the standard), and recompute EVE: the worst ΔEVE across the six is the headline IRRBB number, compared against the 15%-of-Tier-1 outlier threshold. Separately, shock rates +200 bp (parallel) for the 12-month NII projection: the resulting ΔNII is the earnings-sensitivity number.
4. **Hedge and limit.** ALCO sets ΔEVE and ΔNII limits; the ALM desk transacts swaps/FRAs to bring the exposures inside them; the hedge instruments sit in the banking book's hedge portfolios with the accounting treatment validated (IFRS 9 hedge accounting where the group uses it).
5. **Disclose.** Pillar 3 IRRBB1 disclosure reports the ΔEVE/ΔNII under the six shocks and the risk-management approach.

**Adoption in practice (flag):** internationally active banks disclose ΔEVE/ΔNII under the BCBS 368 framework in their Pillar 3 reports; the major Singapore banks (DBS, UOB, OCBC) publish IRRBB metrics in their annual/Pillar 3 disclosures, consistent with MAS supervisory expectations and the group-level frameworks of their European/US parents. The exact SG supervisory threshold calibrations (whether MAS applies the full 15% outlier framework locally) were not independently verified in this pass — treat local implementation details as jurisdiction-specific.

---

## 6. The ALM Models

The ALM modelling toolkit is the analytical core of the function. Five model families do the work; each answers a different question with a different horizon. The models table (§6.6) summarises mechanics and notes.

### 6.1 Gap Analysis (Repricing Gap)

**Question:** *Where is our repricing mismatch?*

**Mechanics.** The balance sheet is bucketed by repricing/maturity date (e.g., overnight–1M, 1–3M, 3–6M, 6–12M, 1–2Y, 2–3Y, 3–5Y, 5–7Y, 7–10Y, >10Y). For each bucket: *rate-sensitive assets − rate-sensitive liabilities* = the **gap**. A positive gap (more assets reprice than liabilities) means NII *falls* when rates fall (assets reprice down faster); a negative gap means NII falls when rates rise. Static gap ignores customer behaviour; the dynamic/behavioural version uses modelled cash flows (NMD cores, prepayments). Simple one-period sensitivity: ΔNII ≈ gap × Δrate. Gap reports are the ALCO pack's most-read page, but they are a *snapshot*, not a full valuation — that is what EVE/NII simulation add.

### 6.2 Duration

**Question:** *How price-sensitive is the balance sheet to a small parallel rate move?*

**Mechanics.** **Duration** is the weighted-average time to receive the present value of a cash-flow stream (Macaulay); **modified duration** is the approximate percentage price change per 100 bp move (≈ Macaulay/ (1+y)). The bank computes the duration of assets, of liabilities, and the **duration gap**: *D_assets − (Liabilities/Assets) × D_liabilities* — the net exposure of equity value to a parallel shift. Positive duration gap: rates up → equity value down. Duration is linear and assumes parallel shifts — fine for a first cut, wrong for curve reshaping (hence **key-rate durations**: sensitivities to 1 bp moves at specific tenors, giving a curve-risk decomposition).

### 6.3 EVE — Economic Value of Equity

**Question:** *What is the whole book worth today, and how much value would a shock destroy?*

**Mechanics.** **EVE** is the present value of the banking book's cash flows: PV(assets) − PV(liabilities), discounted at the risk-free/swap curve. (Equity is the residual, so EVE ≈ PV of future net cash flows + existing equity.) Because NMDs get behavioural maturities and loans get prepayment assumptions, EVE captures *lifetime* repricing risk that a 12-month NII number misses. Under BCBS 368, the bank projects its cash flows under each of the six shocks and computes **ΔEVE** (the change from the base case); the worst shock drives the 15%-of-Tier-1 outlier test. EVE is the long-horizon, value-based lens: it treats the balance sheet like a bond portfolio. Weakness: it ignores the franchise value of future business and is highly sensitive to behavioural assumptions and discounting choices.

### 6.4 NII Simulation — Net Interest Income at Risk

**Question:** *What will we earn over the next 12–24 months, under base and shocked rates?*

**Mechanics.** The bank builds a **projected balance sheet** (existing book rolling off, new business pipeline, maturities reinvested per assumptions), applies **contractual and behavioural cash flows**, and projects **net interest income month by month** under the base curve and under shocks (the BCBS 368 +200 bp parallel NII shock is the regulatory minimum; banks run ±100/±200/±300 bp and scenario suites). Outputs: ΔNII over 12 months, NII-at-risk, and the earnings sensitivity used in hedging decisions and in ALCO limits. NII simulation is the *earnings* lens — the one the CFO cares about most — and it naturally captures basis risk (different indices on assets vs liabilities) that EVE handles poorly. Weaknesses: it depends on new-business and balance-sheet-growth assumptions, and it is myopic for long-dated fixed-rate books.

### 6.5 NMD Behavioural Modelling — Non-Maturity Deposits

**Question:** *What is the true maturity of money that has no contractual maturity?*

**Mechanics.** **Non-maturity deposits (NMDs)** — current accounts, savings accounts, demand deposits — have *no contractual maturity*: the customer can withdraw on demand, and the bank can reprice at will. Modelling them as overnight would make every bank look catastrophically mismatched (funding 5Y loans with overnight money) and would make the LCR/NSFR and EVE meaningless. Behavioural modelling decomposes each NMD pool into:

- **Core vs non-core**: the **core** is the stable floor that stays even in stress (modelled as term funding, e.g., 2–5Y); the **non-core** is the volatile remainder (modelled short, e.g., 1M).
- **Decay / retention rates**: how the pool declines over time without new deposits (survival analysis on historical balances), and how it behaves in stress (LCR run-off factors; NSFR ASF factors of 90–95% for retail).
- **Pass-through rates**: how much of a market-rate change is transmitted to the deposit rate (retail savings pass-through might be 20–40%; corporate demand deposits 60–80%). This determines the NMD's effective sensitivity — the "free option" the bank holds to reprice deposits.

The outputs feed everything: FTP credits (§3), LCR/NSFR inputs, EVE cash-flow projections, NII simulation, and the IRRBB shock results. NMD modelling is the single most consequential *judgement* in ALM — a 1-year shift in the assumed core tenor can move ΔEVE by multiples of the regulatory threshold — which is why regulators require validation, conservatism, and consistency between the FTP, LCR, and IRRBB uses of the same behavioural assumptions. (The BCBS 368 framework explicitly caps the maximum behavioural maturity for NMDs — e.g., 5 years for retail non-transactional, and shorter for transactional/wholesale — and requires banks to demonstrate stability before claiming long cores.)

### 6.6 The Models Table

| Model | Mechanics | Notes |
|---|---|---|
| **Gap analysis** | Bucket assets − liabilities by repricing date; ΔNII ≈ gap × Δrate | Simplest ALM report; static gap ignores behaviour; dynamic gap uses modelled flows; limit-friendly (bucket limits) |
| **Duration** | Macaulay/modified duration; duration gap on equity; key-rate durations for curve risk | Linear, parallel-shift approximation; good for quick equity sensitivity; key-rate extensions handle curve reshaping |
| **EVE** | PV(assets) − PV(liabilities) at swap curve; ΔEVE under BCBS 368's six shocks; outlier test vs 15% Tier 1 | Lifetime value lens; captures long-dated fixed-rate risk; highly sensitive to behavioural assumptions and discounting |
| **NII simulation** | Project balance sheet + rates over 12–24M; compute NII under base and shocks (+200 bp regulatory floor) | Earnings lens; captures basis risk; depends on new-business assumptions; CFO's metric of record |
| **NMD behavioural modelling** | Core/non-core decomposition; decay rates; pass-through rates; feeds FTP, LCR/NSFR, EVE, NII | The key ALM judgement; must be validated, conservative, consistent across uses; BCBS 368 caps behavioural maturities |
| **Supporting models** | Prepayment models (mortgages/loans); funding-concentration analysis; liquidity stress-test cash-flow models; CFP | Each feeds the five core families; all must be independently validated (model risk discipline per the [Risk Management Models Guide](risk_management_models_guide.md)) |

### 6.7 Model Risk in ALM

ALM models are *risk models* in the regulatory sense: they drive FTP prices, hedging decisions, LCR/NSFR inputs, and IRRBB capital dialogue, so a bad model is not an academic problem — it is mispriced products, wrong hedges, and a false sense of safety. The discipline (cross-ref [Risk Management Models Guide](risk_management_models_guide.md) §9 for the SR 11-7/validation machinery):

- **Independent validation.** NMD behavioural models, prepayment models, and the EVE/NII engines are validated by a model-validation function independent of treasury, before first use and on a periodic cycle.
- **Data and backtesting.** Historical deposit balances and decay are backtested against the model's predictions; pass-through rates are estimated from observed rate-setting behaviour, not assumed.
- **Consistency across uses.** The same behavioural assumptions must feed FTP, LCR/NSFR, EVE, and NII — a bank that models NMD cores one way for FTP and another for IRRBB is arbitraging itself, and regulators (and auditors) now check for exactly this.
- **Conservatism and documentation.** Where evidence is thin (a new product, a post-crisis regime), assumptions are set conservatively and documented; the documentation is what survives the supervisory review and the model-risk audit.

---

## 7. The Treasury Operations

The treasury's *operations* are the day-to-day market and cash activities that execute the funding, liquidity, and hedging strategies — distinct from the ALM analytics that set the strategy. Three desks do the operational work, all feeding the same cash-flow picture.

### 7.1 Money Markets

The **money market** is the market for short-term (≤1 year) borrowing and lending — the treasury's raw material market. Operations include:

- **Interbank deposits** — unsecured placement/borrowing of funds with other banks (overnight to 12 months), the classic liquidity-management instrument; in SGD the overnight benchmark is SORA (Singapore Overnight Rate Average), the risk-free rate that replaced SIBOR after the MAS-led transition (SIBOR was fully discontinued at end-2024).
- **Certificates of deposit (CDs) and commercial paper (CP)** — term funding instruments issued into the market, building the bank's funding curve; CP/CD issuance is the front end of the bank's issuance programme.
- **Repo and reverse repo** — secured funding: the bank lends cash against collateral (SGS and MAS bills are the preferred SGD collateral) or borrows cash against its own securities; repos price below unsecured rates (collateralised) and are the deepest, most reliable funding market in stress.
- **Central-bank operations** — participation in the MAS' money-market operations (e.g., the MAS' daily operations for SGD liquidity) and, in stress, the standing facilities — the lender-of-last-resort backstop that makes liquidity risk manageable rather than fatal.

The money-market desk is where the *marginal* funding cost is discovered every day: the level and shape of the short end of the curve, the bank's own credit spread (its CD/CP levels vs SORA), and the availability of collateralised vs unsecured funding are the market signals that feed the FTP curve and the funding plan.

### 7.2 FX

**FX operations** manage the currency dimension of the balance sheet. Every currency mismatch between assets and funding is a funding risk (a USD asset funded with SGD must be converted — and the conversion must be refinanced when it matures), and every currency position is also a P&L risk. Treasury FX operations include:

- **FX spot and forwards** — converting funding proceeds and hedging currency exposures; forwards dominate treasury use because they also hedge the *rollover* of funding.
- **FX swaps and cross-currency swaps** — the treasury's signature instruments: an FX swap borrows one currency and lends another for a term (the "swap points" price the interest differential); a cross-currency swap exchanges fixed/floating cash flows in two currencies for the full tenor, letting the bank fund USD loans with SGD deposits and swap the economics. Cross-currency basis (the deviation of swap-implied funding from covered-interest parity) is a real funding cost that ALM must track per currency.
- **Nostro management** — the bank's accounts with other banks (nostro accounts) must be funded and monitored so that payment flows (SWIFT traffic, RTGS/MEPS+ in Singapore) never bounce; this is the operational link between FX, cash management, and liquidity.

For a Singapore-based CIB like Cymbal Bank, FX operations are the daily bridge between SGD funding, USD-centric wholesale markets, and the EUR home-group balance sheet — cross-currency swaps between SGD/USD/EUR are structural, not tactical.

### 7.3 Cash Management

**Cash management** is the operational core of liquidity: ensuring cash is *where it is needed, when it is needed, in the right currency*, at minimum cost. Components:

- **Daily cash-flow forecasting** — consolidating expected inflows/outflows from loans, deposits, payments, and market transactions into a day-by-day cash forecast; the LCR's 30-day projection is the regulated version of the same discipline (the Basel liquidity-monitoring tools add contractual-maturity mismatch, concentration, and market-related monitoring metrics).
- **Liquidity-buffer management** — buying, selling, and rebalancing the HQLA portfolio (SGS, MAS bills, central-bank reserves) so the buffer is always above the LCR minimum and composition caps.
- **Payments and settlement** — managing settlement through RTGS (in Singapore, MEPS+) and payment schemes, monitoring nostro balances and intraday liquidity (intraday liquidity is itself a monitored risk — Basel tool #5), and managing the bank's participation in payment rails (cross-ref the [Singapore Fintech & Payments Guide](singapore_fintech_payments_guide.md)).
- **Cash concentration / in-house banking** — sweeping balances across accounts and entities so surplus cash is pooled (in the group context, Cymbal Bank runs group-wide cash concentration for clients and its own entities).

Cash management is the operational discipline that makes the LCR number *real*: a bank can report a healthy ratio and still fail if its buffer sits in the wrong currency or the wrong time zone.

### 7.4 The Operations Table

| Operation | Instruments / Activities | Purpose | Risk it manages | Links to |
|---|---|---|---|---|
| **Money markets** | Interbank deposits, CD/CP, repo/reverse repo, central-bank ops | Fund the balance sheet short term; discover marginal funding cost | Funding liquidity; rollover risk; collateral availability | FTP curve; funding plan; LCR (30-day outflows) |
| **FX** | Spot, forwards, FX swaps, cross-currency swaps, nostro mgmt | Convert and hedge funding across currencies; fund USD assets | Currency funding mismatch; cross-currency basis cost; settlement risk | ALM hedging; liquidity (currency dimension); payments |
| **Cash management** | Cash-flow forecasting, HQLA rebalancing, MEPS+/RTGS settlement, concentration | Have cash where and when needed | Intraday liquidity; buffer shortfall; settlement failure | LCR; NSFR; stress testing; payments rails |
| **Back-office treasury ops** | Confirmations, settlement, reconciliation, collateral management | Settle everything the desks trade | Operational/settlement risk; fails | Markets systems; core banking; [Capital Markets Architecture Guide](capital_markets_architecture_guide.md) |

---

## 8. The Treasury Systems

Treasury runs on two classes of systems: **treasury workstations / ALM platforms** (the analytical and risk-owner layer — the subject of this section) and the surrounding data estate (core banking for deposit/loan cash flows, markets systems for trading, data warehouses for reporting). The two market-leading treasury platforms are Murex MX.3 and Nasdaq Calypso, both deep-dived as sibling guides — this section positions them for the treasury/ALM function and points to the detail.

### 8.1 Murex MX.3 (cross-ref: [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md))

**MX.3** is Murex's front-to-back platform: one data model covering trading, treasury, risk, and post-trade operations, used by the sell side (banks and CIBs — DBS runs Murex in Singapore) and increasingly by corporates via the packaged **MXGO** treasury solution. For the treasury/ALM function, MX.3 provides:

- **Treasury & ALM modules** — cash management, liquidity management, funding, and balance-sheet analytics on the same platform as the FX/money-market/derivatives trading it executes: the ALM desk's hedges (swaps, FRAs, cross-currency) are captured in the same system as the FTP and liquidity analytics that motivate them.
- **Risk analytics** — market risk (VaR/FRTB), credit risk, and the liquidity/IRRBB analytics; the platform computes the LCR/NSFR engines, gap/duration/EVE/NII reports, and shock scenarios from the same cash-flow engine that prices the hedges.
- **One-data-model advantage** — no reconciliation between "the hedge book" and "the ALM report": position, valuation, and risk are the same numbers. This is the architectural argument Murex wins on for complex derivatives-heavy treasuries (see the [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md) §10 for a worked treasury implementation, and the [DBS Software Systems Guide](dbs_software_systems_guide.md) for the anchor deployment).

### 8.2 Nasdaq Calypso (cross-ref: [Nasdaq Calypso Guide](nasdaq_calypso_guide.md))

**Nasdaq Calypso** (built by Calypso Technology, founded 1997, acquired by Nasdaq with Adenza in 2023) is the cross-asset front-to-back platform whose **Calypso Treasury** module is a dedicated treasury franchise: cash management, liquidity, funding, payments, bank-account management, in-house banking, cash forecasting, and IRRBB/FRTB-aligned risk — used by banks, CCPs, central banks, and corporate treasuries. For the treasury function:

- **Calypso Treasury** is a *treasury workstation first*: real-time cash positions, cash-flow forecasting, liquidity and funding management, payments, and nostro management — the operational layer (§7) — with the trading layer (money market, FX, derivatives) for the funding and hedging desks.
- **ALM/IRRBB analytics** sit alongside: gap, EVE, NII, LCR/NSFR computation from the same cash-flow data, with the behavioural-modelling layer for NMDs.
- **Positioning vs Murex**: the practitioner shorthand is Murex for complex derivatives risk/valuation depth, Calypso for treasury + collateral + cross-asset operations breadth (per the [Nasdaq Calypso Guide](nasdaq_calypso_guide.md) §19); banks with a huge derivatives treasury (Cymbal Banks) often land on Murex, while treasury-centric and corporate-treasury franchises often land on Calypso.

### 8.3 The Broader Treasury Systems Landscape

- **FIS** (e.g., FIS Quantum/ALM, formerly Sungard's BancWare and Ambit) — long-standing ALM/asset-liability workstation franchise, strong in mid-size and regional banks, regulatory reporting (LCR/NSFR) and budgeting.
- **Zühlke/Moody's (RiskFront/ALM), Wolters Kluwer OneSumX, Oracle (OFSAA / Oracle Banking Treasury Management), Temenos (Treasury + core-linked)** — the ALM-adjacent analytics and treasury-adjacent modules of the bank-systems vendors (cross-ref [Core Banking Systems Guide](core_banking_systems_guide.md) for where core-vendor treasury modules sit).
- **The data estate** — every treasury platform is only as good as its data feeds: core-banking cash flows, payment data, market curves, and the general ledger; the modern pattern is a treasury data lake feeding the ALM engines (cross-ref [Event Stream Processing Guide](../technology/event_stream_processing_guide.md) for the real-time treasury angle and [Monolith to Microservices Guide](../technology/monolith_to_microservices_guide.md) for the modernisation angle).

### 8.4 The Systems Table

| System | Vendor | Treasury role | Strengths | Cross-ref |
|---|---|---|---|---|
| **Murex MX.3** | Murex | Front-to-back trading + treasury + risk; ALM analytics on the hedge platform | One data model; derivatives/risk depth; FTP + LCR/NSFR + IRRBB engines; FX/MM execution | [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md); [DBS Software Systems Guide](dbs_software_systems_guide.md) |
| **Nasdaq Calypso** | Nasdaq (ex-Calypso Technology) | Treasury workstation: cash, liquidity, funding, payments, forecasting; ALM/IRRBB; trading layer | Treasury-first breadth; collateral; corporate-treasury and CCP footprint; in-house banking | [Nasdaq Calypso Guide](nasdaq_calypso_guide.md) |
| **FIS Quantum/ALM (ex-BancWare/Ambit)** | FIS | ALM analytics, LCR/NSFR, budgeting, FTP | Deep ALM-function pedigree; regional-bank franchise | §6 models; [Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md) §6 |
| **Oracle OFSAA / Oracle Banking Treasury** | Oracle | ALM, FTP, liquidity, regulatory reporting | Bank-vendor breadth; integrated with core | [Core Banking Systems Guide](core_banking_systems_guide.md) |
| **Temenos Treasury** | Temenos | Treasury module alongside core | Retail/universal-bank treasury-lite | [Core Banking Systems Guide](core_banking_systems_guide.md) |
| **In-house / bespoke ALM stacks** | Bank IT | Custom FTP/ALM engines on treasury data lakes | Full control; fit-to-purpose; costly to maintain | [Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md) §6 |

### 8.5 Choosing and Implementing a Treasury Platform

The selection and delivery pattern (cross-ref the [Business Case Development Guide](../management/business_case_development_guide.md) for the investment case):

1. **Scope first, vendor second.** The bank's ALM needs are a function of its *funding structure* (CIB vs retail, §1.5), its hedge complexity, and its regulatory footprint. A derivatives-heavy CIB treasury needs the trading/risk depth of an MX.3-class platform; a treasury-centric franchise (corporate treasury, CCP, central bank) may find Calypso's treasury workstation the better fit; a mid-size bank may need no more than FIS-class ALM analytics fed from its core.
2. **The data model is the decision.** The platform's value is one cash-flow truth: FTP rates, LCR/NSFR, ΔEVE/ΔNII, and hedge P&L from the same engine. Implementation effort is dominated by data feeds (core banking, payments, market curves, GL) and by behavioural-model build/validation — not by the vendor's configuration.
3. **Migration risk.** Treasury systems sit between the trading floor and the ledger; cutover touches every desk. Phased migrations (ALM analytics first, then the funding desk, then the hedge books), parallel runs, and reconciliation to the GL are the proven pattern (the worked implementations in the [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md) §10 and the [Nasdaq Calypso Guide](nasdaq_calypso_guide.md) §18 describe it in detail).
4. **Operate, don't just build.** The ALM function runs on *assumptions that must be refreshed*: curves daily, behavioural models annually, stress scenarios per regulatory cycle. The platform's operating model (who owns the models, the validation calendar, the regulatory reporting) is as important as the software.

---

## 9. The Worked Example: An ALM Design for a Cymbal Bank

### 9.1 The Scenario (the familiar context)

Cymbal Bank — the corporate and investment banking arm of Crédit Agricole S.A., the world's largest cooperative banking group — runs universal-bank funding economics inside a CIB franchise (cross-ref [Universal Banking Model Guide](universal_banking_model_guide.md) and the [Crédit Agricole Software Systems Guide](credit_agricole_software_systems_guide.md)). Its Singapore branch (cross-ref [Banks in Singapore Guide](banks_in_singapore_guide.md)) is a wholesale/corporate bank: corporate loans and trade finance in SGD/USD, corporate deposits, and interbank/bond funding — no retail deposit base, which is the *defining* ALM constraint of a CIB branch (funding must come from wholesale and corporate sources, so the LCR and NSFR are structurally harder than for a retail bank).

We design the ALM framework for a stylised **Cymbal Bank balance sheet** (illustrative, SGD millions):

| Assets | Amount | Behaviour | Liabilities | Amount | Behaviour |
|---|---|---|---|---|---|
| Corporate loans (3Y floating, SORA-linked) | 2,000 | Reprice 3M; no prepayment option | Corporate demand deposits (NMDs) | 800 | Core 60% ≈ 3Y; non-core 40% ≈ 1M |
| Trade finance (≤1Y) | 600 | Reprice 1M–6M | Corporate term deposits (3M–1Y) | 400 | Contractual |
| HQLA buffer (SGS + MAS bills + cash) | 300 | Liquid | Interbank & CD/CP funding (≤1Y) | 1,200 | Rollover every 1M–6M |
| Interbank placements | 150 | O/N–1M | Senior bond issuance (3Y, 5Y) | 700 | Contractual |
| Other assets (fixed, equity) | 150 | — | Other liabilities + equity | 100 | — |
| **Total assets** | **3,200** | | **Total liabilities + equity** | **3,200** | |

Constraints that shape the design: (1) no retail deposits → the NSFR leans on term issuance and the LCR on a real HQLA buffer; (2) SGD funding is scarce in wholesale markets → the branch funds in USD/EUR and swaps into SGD where client demand requires (cross-currency basis is a real cost); (3) the group's ALM is run centrally in Paris, so the branch's IRRBB and liquidity metrics must feed group reporting as well as MAS.

### 9.2 The FTP + NII Design

**The FTP curve (SGD).** The branch builds a matched-maturity transfer curve off the **SORA OIS curve** plus a **liquidity premium** derived from its own issuance levels (illustrative, annualised):

| Tenor | SORA OIS | + Liquidity premium | = FTP transfer rate |
|---|---|---|---|
| 1M | 2.00% | 0.15% | 2.15% |
| 3M | 2.05% | 0.20% | 2.25% |
| 6M | 2.10% | 0.25% | 2.35% |
| 1Y | 2.20% | 0.35% | 2.55% |
| 3Y | 2.50% | 0.60% | 3.10% |
| 5Y | 2.80% | 0.80% | 3.60% |

**FTP credits and charges (matched-maturity):**

- 3Y corporate loan at customer rate 4.00%: FTP charge = 3Y transfer 3.10% → **lending spread +0.90%** (before costs/expected loss).
- Trade finance at 3M SORA + 1.40% (≈3.45%): FTP charge = 3M 2.25% → **spread +1.20%**.
- Corporate demand deposits paying 0.60%: behavioural blend — core 60% at 3Y (3.10%), non-core 40% at 1M (2.15%) → blended credit ≈ 2.72% → **deposit spread +2.12%**.
- Corporate term deposits (3M, 2.90%): credit = 3M 2.25% → **spread +0.35%** (thin — the product pays away most of its value; ALCO challenges its role).

**Treasury residual (the transformation P&L).** Treasury receives 3.10% on the loans and pays 2.72%/2.25% on the deposits and 3.60% on the 5Y bond; the gap between asset FTP income and liability FTP cost plus the *actual* cost of the wholesale funding and swaps is the residual — the price of funding a 3Y loan book with 3Y-behaving cores, 3M term deposits, and 5Y bonds. ALCO monitors the residual against the risk that generates it (gap limits, ΔEVE, ΔNII).

**NII projection (12-month, base case, illustrative):**

| Component | NII contribution |
|---|---|
| Lending spread income (2,600 assets × ~1.0% avg) | 26.0 |
| Deposit spread income (1,200 liabilities × ~1.3% avg) | 15.6 |
| Treasury residual (transformation P&L, net of wholesale funding) | 8.4 |
| HQLA carry + other | 2.0 |
| **Base-case NII (12M)** | **52.0** |

**IRRBB design.** Run the BCBS 368 six shocks: with 3Y floating loans (SORA-linked, low gap risk) funded partly by NMD cores (3Y) and 5Y bonds, the book is *asset-sensitive at the short end* (floating loans reprice with SORA; NMD cores lag via pass-through) and *liability-sensitive at the long end*. Expected results: ΔEVE worst-case (flattener or parallel-up) in the range of 6–9% of Tier 1 — inside the 15% outlier threshold but above the internal limit, so the ALM desk buys receive-fixed swaps in SGD and USD to lengthen the asset side's effective duration; ΔNII under +200 bp ≈ +2–3% of NII (asset-sensitive short book earns in a rising-rate world) — the classic EVE/NII trade-off, owned explicitly by ALCO. NMD pass-through assumptions (corporate demand deposits: 60% pass-through) are validated and filed with the group model-risk function.

**Liquidity design.** LCR: HQLA of 300 against 30-day net outflows ≈ 240 (conservative run-off on wholesale funding and commitments) → **LCR ≈ 125%**, above the 100% MAS minimum and the branch's internal 120% floor; buffer composition: SGS + MAS bills + cash (Level 1, no haircut) with no Level 2 reliance (a CIB can't afford haircut surprises). NSFR: ASF from the 3Y/5Y bonds (100%), NMD cores (95%×core), term deposits (50–90% by type) vs RSF from loans (3Y corporate loans 65%, trade finance 50%) → **NSFR ≈ 110–115%**, above the 100% minimum; the branch's funding plan for the year extends the CD/CP profile and prefunds the 5Y maturities to keep it there. Stress: the CFP assumes a 30-day market-wide freeze; the HQLA buffer covers 30 days of net outflows at the *stress* run-off factors, and the group's Paris ALM stands behind the branch as the backstop funder.

### 9.3 The Lessons

1. **FTP is the design's backbone.** Matched-maturity FTP on a SORA-based curve makes every product's term and liquidity economics visible: the demand-deposit franchise is the value engine (2.12% spread), term deposits are marginal, and the 3Y loan book pays its true funding cost. Without FTP, the branch would misprice all three and discover it in the treasury residual years later.
2. **A CIB branch's ALM is defined by its funding side.** No retail deposits means the LCR and NSFR are structurally harder: the HQLA buffer and term issuance are not choices but constraints. The ALM design leads with funding structure, not loan pricing.
3. **Behavioural assumptions dominate the risk numbers.** The NMD core assumption (60% core ≈ 3Y) and the pass-through rate move ΔEVE and the deposit FTP credit more than any other input — which is exactly why they are validated, conservatively set, and shared consistently between FTP, LCR, and IRRBB uses.
4. **EVE and NII must be owned together.** The design is EVE-hedged but NII-positive in a rising-rate world; ALCO owns the trade-off explicitly rather than letting the desks optimise one metric against the other.
5. **Systems make the design operational.** The same cash-flow engine must produce the FTP rates, the LCR/NSFR, the ΔEVE/ΔNII, and the hedge P&L — Murex or Calypso-class platforms (or a well-governed in-house stack) exist precisely to keep those numbers on one data model (§8).

---

## 10. The Summary: One Page

**Treasury is the bank's internal bank — and ALM is its discipline.** Every bank is a maturity-transformation machine: it takes short-dated, liquid liabilities and funds long-dated, illiquid assets with them. The treasury function is the control room of that machine. It funds the balance sheet (deposits, money markets, bonds), holds the liquidity buffer, manages the interest-rate and currency risk of the banking book, prices internal funds through FTP, and reports the whole picture to ALCO and to regulators.

**FTP is the pricing backbone.** Matched-maturity FTP credits every deposit and charges every loan off a transfer curve (risk-free rate + liquidity premium, per tenor and behaviour). It makes term and liquidity risk explicit, removes cross-subsidies between products, and concentrates the true cost of maturity transformation in the treasury residual where ALCO can see and govern it. Pooling hides all of that; matched-maturity reveals it.

**Liquidity is survival.** The LCR (HQLA ≥ 100% of 30-day net outflows; phased in from 60% in 2015 to 100% by 2019) and the NSFR (ASF ≥ RSF over one year; minimum standard from 1 January 2018) are the Basel III liquidity constitution. In Singapore they land through MAS Notice 649 (MLA + LCR), with the local framework applying the 100% LCR to significant banks from 2015.

**IRRBB is the value risk.** BCBS 368 (April 2016) replaced the 2004 principles with a standardised framework: six shocks, ΔEVE with a 15%-of-Tier-1 outlier test, and a 12-month ΔNII under a +200 bp parallel shock — a Pillar 2 regime that has nonetheless reshaped how banks hedge, disclose, and govern their banking books.

**The models are the eyes.** Gap analysis, duration, EVE, NII simulation, and NMD behavioural modelling each see a different slice: the repricing snapshot, the price sensitivity, the lifetime value, the 12-month earnings, and the true maturity of money with no contractual maturity. Behavioural assumptions are the biggest judgement in ALM and the most scrutinised.

**Systems make it operational.** Murex MX.3 and Nasdaq Calypso — the two front-to-back treasury platforms — keep the FTP rates, the LCR/NSFR, the ΔEVE/ΔNII, and the hedge P&L on one data model, so the numbers ALCO governs are the numbers the desks trade.

**The final word: the balance-sheet guardians.** Commercial banks earn their margin by taking maturity, liquidity, and rate risk that no other institution can take at the same scale — and the treasury function exists to make sure that risk is *priced, measured, limited, and funded*, not stumbled into. The businesses originate; the markets trade; the regulators watch. The treasury — through ALM, FTP, liquidity management, and IRRBB — is what turns the balance sheet from a ledger into a risk-managed franchise. When it works, the bank is boringly liquid, quietly profitable, and unremarkable in stress. That is the job: to be the guardians of the balance sheet.

---

## 11. Verification Notes and Sources

This guide was written under a verify-or-flag discipline: regulatory dates and model mechanics were checked against primary and secondary sources during the research pass (web access restored; searches run against the Basel Committee, regulator, and practitioner literature). What follows is the verification record, including the one significant correction to the task brief.

### 11.1 Verified claims

| Claim | Status | Source |
|---|---|---|
| LCR: BCBS standard Dec 2010 (BCBS 188), revised Jan 2013 (BCBS 238) | ✅ Verified | bis.org/publ/bcbs238.htm (BIS) |
| LCR: effective 1 Jan 2015 with phasing from 60% to 100% by 1 Jan 2019, +10pp/year | ✅ Verified | PwC briefing note on the revised LCR |
| NSFR: final standard Oct 2014 (BCBS 295); minimum standard from 1 Jan 2018 | ✅ Verified | bis.org/bcbs/publ/d295.htm (BIS); lewik.org summary |
| IRRBB: BCBS 368 published April 2016 (21 April 2016); replaces the 2004 Principles (BCBS 108) | ✅ Verified | bis.org/bcbs/publ/d368.htm (BIS); bankinghub.eu |
| IRRBB: six standardised shocks; ΔEVE outlier test at 15% of Tier 1; 12-month ΔNII under +200 bp | ✅ Verified | BCBS 368 (d368.pdf) — standard text; consistent across sources |
| FTP: single-pool vs multiple-pool vs matched-maturity methodologies | ✅ Verified | thealmdesk.com module; passfrm.com FTP explainer |
| ALM models: gap analysis, duration, EVE, NII, NMD behavioural modelling as the core toolkit | ✅ Verified | vector-cfo.com EVE guide; longbridge.com EVE explainer; BIS DIS70 (Pillar 3 IRRBB disclosure) |
| MAS Notice 649 = Minimum Liquid Assets + LCR, applying to all banks in Singapore | ✅ Verified | MAS Notice 649 text (via regalert.today and MAS PDFs); Maybank SG LCR disclosure |
| MAS Notice 658 = Management of Outsourced Relevant Services for Banks (issued 11 Dec 2023, effective 11 Dec 2024) — **not** the liquidity framework | ✅ Verified (correction) | allenandgledhill.com; pwc.com/sg — the task brief's "MAS 658 = SG liquidity framework" is a conflation |
| Murex MX.3 and Nasdaq Calypso treasury coverage and positioning | ✅ Verified in-repo | [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md); [Nasdaq Calypso Guide](nasdaq_calypso_guide.md) |

### 11.2 Flagged / unverified

- **MAS NSFR notice number.** The standalone MAS notice implementing the NSFR for Singapore banks could not be independently confirmed in this research pass (searches for "MAS Notice 650 NSFR" returned no authoritative hits). The guide states only what is verified: NSFR disclosure requirements for Singapore D-SIBs sit inside the Notice 649 regime. **Flagged as unverified.**
- **EBA IRRBB guideline number/date.** EBA/GL/2018/02 (IRRBB and CSRBB guidelines) is standard practitioner knowledge and consistent with the 2016 BCBS 368 timeline, but was not re-verified by a direct source fetch in this pass. Treat as standard knowledge, flagged as not independently fetched.
- **SIBOR→SORA transition mechanics** (SIBOR discontinued end-2024, SORA the SGD RFR): widely documented MAS-led transition; stated from domain knowledge, not re-fetched in this pass.
- **All worked-example numbers (§9)** are illustrative design outputs (balance sheet, FTP curve, NII, LCR/NSFR estimates), internally consistent but not factual claims about Cymbal Bank or any real bank.
- **BCBS 368 shock magnitudes and NMD maturity caps** (e.g., 5-year behavioural-maturity caps): consistent with the standard as widely summarised; the precise annex wording was not re-fetched — treat the *framework* as verified, the *exact caps* as standard knowledge.

### 11.3 Search queries used (this pass)

"Basel III LCR liquidity coverage ratio 2015 full phasing 60% 100% timeline NSFR 2018" · "IRRBB BCBS 368 April 2016 interest rate risk banking book standard" · "NSFR net stable funding ratio Basel III 2014 standard implementation 1 January 2018" · "MAS Notice 649 liquidity coverage ratio Notice 650 minimum liquid assets NSFR Singapore banks" · "MAS Notice 658 liquidity coverage ratio Singapore banks revision December 2024" · "funds transfer pricing matched maturity versus pooling method bank FTP mechanics" · "asset liability management ALM models EVE economic value of equity NII simulation non-maturity deposits behavioural modelling" · "MAS 'Notice 650' NSFR net stable funding Singapore" · "Singapore banks NSFR requirement MAS liquidity regulations 2020" — plus in-repo cross-reference checks of the sibling guides.

---

## 12. Glossary

| Term | Definition |
|---|---|
| **Treasury** | The bank function that manages funding, liquidity, cash, and balance-sheet financial risk at the firm level; the bank's internal bank |
| **ALM** | Asset-Liability Management — the discipline of managing assets and liabilities jointly to protect earnings, value, and liquidity |
| **Asset-liability management** | Same as ALM; balance-sheet management / structural risk management |
| **FTP** | Funds Transfer Pricing — the internal mechanism that charges assets and credits liabilities for the cost/value of funds |
| **Funds transfer pricing** | Same as FTP; the internal pricing of funds between treasury and business lines |
| **Matched-maturity** | FTP method pricing each product off the transfer curve at its (behavioural) tenor, making term and liquidity risk explicit |
| **Pooling** | FTP method applying one blended average funding cost to all assets and liabilities regardless of tenor |
| **Liquidity** | The bank's ability to meet payment obligations as they fall due without unacceptable losses |
| **LCR** | Liquidity Coverage Ratio — HQLA / 30-day net cash outflows ≥ 100%; Basel III short-term liquidity standard (effective 2015, phased to 100% by 2019) |
| **NSFR** | Net Stable Funding Ratio — ASF / RSF ≥ 100% over one year; Basel III structural funding standard (minimum from 1 Jan 2018) |
| **IRRBB** | Interest-Rate Risk in the Banking Book — risk to banking-book earnings and value from adverse rate moves (gap, basis, option risk) |
| **BCBS 368** | The April 2016 Basel standard on IRRBB: six shocks, ΔEVE, ΔNII, 15%-of-Tier-1 outlier test, disclosure |
| **Interest-rate risk** | The risk that rate movements reduce earnings (NII) or value (EVE) |
| **Banking book** | Everything the bank holds that is not the trading book: loans, deposits, funding, and their hedges |
| **Gap analysis** | Bucketing rate-sensitive assets minus liabilities by repricing date to expose maturity/repricing mismatches |
| **Duration** | The weighted-average time to receive the PV of a cash-flow stream; modified duration approximates price sensitivity per 100 bp |
| **EVE** | Economic Value of Equity — PV(assets) − PV(liabilities); the lifetime value lens on the banking book |
| **NII** | Net Interest Income — interest income minus interest expense; the earnings lens of ALM |
| **Net interest income** | Same as NII; the banking book's core earnings stream |
| **NMD** | Non-Maturity Deposits — current/savings/demand accounts with no contractual maturity |
| **Non-maturity deposits** | Same as NMD; deposits repayable on demand with bank-controlled repricing |
| **Behavioural modelling** | Modelling the true maturity/behaviour of products (NMD cores, decay, pass-through, prepayment) for FTP, LCR/NSFR, EVE, NII |
| **Money markets** | The market for short-term (≤1 year) borrowing/lending: interbank deposits, CP/CD, repo |
| **FX** | Foreign exchange — the currency conversion, hedging, and funding-bridge operations of the treasury |
| **Cash management** | Ensuring cash is where and when needed: forecasting, HQLA rebalancing, settlement, concentration |
| **Murex** | Vendor of MX.3, the front-to-back trading/treasury/risk platform (see the [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md)) |
| **Calypso** | Nasdaq Calypso — the cross-asset front-to-back platform with a dedicated treasury workstation franchise (see the [Nasdaq Calypso Guide](nasdaq_calypso_guide.md)) |
| **Basel III** | The post-2008 BCBS reforms: capital (Pillar 1), liquidity (LCR/NSFR), leverage, and supervision |
| **MAS Notice 658** | The Singapore outsourcing-governance notice (issued 11 Dec 2023, effective 11 Dec 2024) — **not** the liquidity framework; see MAS Notice 649 for MLA/LCR |
| **Balance sheet** | The bank's assets (loans, securities, cash) and liabilities (deposits, funding, capital) — the object of ALM |
| **Funding** | The raising of money to finance assets: deposits, interbank, CP/CD, bonds, capital |
| **ALCO** | Asset & Liability Committee — the governance body that sets liquidity/IRRBB appetite and approves FTP |
| **HQLA** | High-Quality Liquid Assets — the LCR numerator: cash, central-bank reserves, SGS, high-rated securities (Levels 1/2A/2B) |
| **ASF / RSF** | Available/Required Stable Funding — the NSFR numerator/denominator, weighted by stability factors |
| **SORA** | Singapore Overnight Rate Average — the SGD risk-free rate; the base of SGD FTP curves (replaced SIBOR) |
| **Pass-through rate** | The fraction of a market-rate change transmitted to deposit rates; a key NMD behavioural input |
| **Contingency funding plan (CFP)** | The documented plan for surviving a funding stress: buffers, triggers, actions, backstops |

---

## 13. Cross-References in This Series

| Sibling guide | What it adds to this topic |
|---|---|
| [Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md) | §6 liquidity-risk-and-treasury-systems; the in-house systems frame; MAS notice landscape |
| [Risk Management Models Guide](risk_management_models_guide.md) | The risk-model discipline: VaR/FRTB, liquidity cash-flow modelling, model risk (SR 11-7) |
| [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md) | The treasury/front-office platform deep-dive; worked treasury implementation |
| [Nasdaq Calypso Guide](nasdaq_calypso_guide.md) | The Calypso treasury platform deep-dive; the treasury workflow |
| [Capital Markets Architecture Guide](capital_markets_architecture_guide.md) | The markets stack; the treasury-desk angle; front/middle/back office |
| [Core Banking Systems Guide](core_banking_systems_guide.md) | Deposit funding and customer accounts; the NMD data source |
| [Universal Banking Model Guide](universal_banking_model_guide.md) | The funding model of universal banks; Cymbal Bank context |
| [Banks in Singapore Guide](banks_in_singapore_guide.md) | The SG regulatory context; MAS framework |
| [Crédit Agricole Software Systems Guide](credit_agricole_software_systems_guide.md) | The Cymbal Bank context used in the worked example |
| [DBS / UOB / OCBC / HSBC / Bank of America guides](dbs_bank_guide.md) | Treasury sections of the bank deep-dives (light cross-refs) |
| [Singapore Fintech & Payments Guide](singapore_fintech_payments_guide.md) | Payments-liquidity angle; MEPS+/payment rails |
| [Event Stream Processing Guide](../technology/event_stream_processing_guide.md) | Real-time treasury; intraday liquidity streaming |
| [Monolith to Microservices Guide](../technology/monolith_to_microservices_guide.md) | Modernising the treasury stack |
| [Business Case Development Guide](../management/business_case_development_guide.md) | The business case for treasury-system investment |
| [MBA Body of Knowledge Guide](../management/mba_body_of_knowledge_guide.md) | The finance vocabulary; balance-sheet literacy |

---

*End of guide. Verified as far as the sources listed in §11 allow; flagged where they do not. The numbers in §9 are illustrative design outputs, not real bank data.*

*Series note: the treasury/ALM deep-dive pairs with the [Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md) (§6) for the systems frame, the [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md) and [Nasdaq Calypso Guide](nasdaq_calypso_guide.md) for the platforms, and the [Risk Management Models Guide](risk_management_models_guide.md) for the model-risk discipline — the four together are the repo's treasury-and-ALM coverage.*
