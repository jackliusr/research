# Private Equity: A Comprehensive Guide

**The Mechanics of the Patient Capital — the GP/LP Structure, Fund Economics (Fees, Carry, the Waterfall), Deal Types, LBO Mechanics, the LP Universe, Regulation (AIFMD, the SEC Private Fund Rules, the Singapore Regime), the Asia-Pacific Landscape, the Cymbal Bank Angle, and Two Fully Worked Examples (a Subscription-Line Facility and an LBO with the Arithmetic Shown)**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Banking Domain / Private Markets — the dedicated private-equity mechanics guide: the closed-end fund model, GP/LP economics and the distribution waterfall, deal types (LBO, MBO, growth, venture, distressed, secondaries, continuation funds), the debt stack and value-creation levers, exit routes, IRR/MOIC/DPI/TVPI and the J-curve, the LP universe and ILPA, AIFMD and AIFMD II, the SEC Private Fund Advisers Rules (adopted 2023, vacated 2024), the Singapore fund regime (VCC, 13O/13U/13OA, MAS licensing), the Asia-Pacific firm landscape (Hillhouse, BPEA/EQT, Affinity, PAG, CVC Asia, KKR Asia), the Cymbal Bank fund-finance angle, and worked examples with the arithmetic shown
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** ilpa.org (Private Equity Principles, versions 1.0/2.0/3.0), eur-lex.europa.eu (Directive 2011/61/EU — AIFMD; Directive (EU) 2024/927 — AIFMD II), sec.gov and ca5.uscourts.gov (Private Fund Advisers Rules; *National Association of Private Fund Managers v. SEC*, June 2024), sso.agc.gov.sg (Variable Capital Companies Act 2018; Income Tax Act 1947), RSM Singapore and PwC Singapore bulletins on the MAS fund tax incentive circulars (FDD Cir 10/2024), mas.gov.sg (Singapore Asset Management Survey), pag.com, kkr.com / asiaasset.com, firm and press records (Reuters, FT, WSJ, PEI, Nasdaq) for Hillhouse, BPEA/EQT, Affinity, CVC, and PAG
> **Last Updated:** August 2026
> **Companion guides:** [Singapore Private Markets](singapore_private_markets_guide.md) (the SG private-markets ecosystem — VCC, 13O/13U, sovereigns, firm landscape), [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) (the MAS regime — SFA, notices, AML), [Norges Bank Investment Management](norges_bank_investment_management_guide.md) (the SWF/LP view), [Partners Group](partners_group_company_guide.md) and [Partners Group Competitors](partners_group_competitors_guide.md) (the alternatives manager as GP), [Alternatives Front-to-Back Operating Model](alternatives_front_to_back_operating_model.md) (fund operations — NAV, valuation, investor reporting, AML), [Asset Management & Alternatives](asset_management_alternatives_guide.md), [Asset-Backed Trading](asset_backed_trading_guide.md), [Financial Fraud Detection at Scale](financial_fraud_detection_at_scale_guide.md) (AML/sanctions for fund clients), [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md)

---

## Table of Contents

1. [What Private Equity Is — the Closed-End Fund Model](#1-what-private-equity-is--the-closed-end-fund-model)
   - 1.1 [The Definition — and What "Private" Means](#11-the-definition--and-what-private-means)
   - 1.2 [The Closed-End Structure](#12-the-closed-end-structure)
   - 1.3 [Why Private Equity Exists — the Return Sources](#13-why-private-equity-exists--the-return-sources)
   - 1.4 [The Asset-Class Scoreboard — PE vs the Alternatives](#14-the-asset-class-scoreboard--pe-vs-the-alternatives)
2. [The GP/LP Structure and the Fund Lifecycle](#2-the-gp-lp-structure-and-the-fund-lifecycle)
   - 2.1 [General Partner vs Limited Partner](#21-general-partner-vs-limited-partner)
   - 2.2 [The Fund Lifecycle — Raise, Invest, Manage, Exit](#22-the-fund-lifecycle--raise-invest-manage-exit)
   - 2.3 [The J-Curve](#23-the-j-curve)
3. [Fund Economics — Fees, Carry, and the Distribution Waterfall](#3-fund-economics--fees-carry-and-the-distribution-waterfall)
   - 3.1 [The Management Fee](#31-the-management-fee)
   - 3.2 [Carried Interest — the 20% Convention](#32-carried-interest--the-20-convention)
   - 3.3 [The Catch-Up and the Hurdle](#33-the-catch-up-and-the-hurdle)
   - 3.4 [The Distribution Waterfall — Deal-by-Deal vs Whole-Fund](#34-the-distribution-waterfall--deal-by-deal-vs-whole-fund)
   - 3.5 [The GP Commitment](#35-the-gp-commitment)
4. [Deal Types — LBO, Growth Equity, Venture, Distressed, Secondaries](#4-deal-types--lbo-growth-equity-venture-distressed-secondaries)
   - 4.1 [The Deal-Type Map](#41-the-deal-type-map)
   - 4.2 [Buyout (LBO) and MBO](#42-buyout-lbo-and-mbo)
   - 4.3 [Growth Equity](#43-growth-equity)
   - 4.4 [Venture Capital — the VC-vs-PE Boundary](#44-venture-capital--the-vc-vs-pe-boundary)
   - 4.5 [Distressed and Special Situations](#45-distressed-and-special-situations)
   - 4.6 [Secondaries and Continuation Funds](#46-secondaries-and-continuation-funds)
5. [LBO Mechanics — the Debt Stack, Value Creation, Exits, and Performance](#5-lbo-mechanics--the-debt-stack-value-creation-exits-and-performance)
   - 5.1 [The Debt Stack — Senior, Mezzanine, Unitranche](#51-the-debt-stack--senior-mezzanine-unitranche)
   - 5.2 [Leverage Multiples and Credit Metrics](#52-leverage-multiples-and-credit-metrics)
   - 5.3 [Buy-and-Build](#53-buy-and-build)
   - 5.4 [The Value-Creation Levers](#54-the-value-creation-levers)
   - 5.5 [The Exit Routes](#55-the-exit-routes)
   - 5.6 [Performance Metrics — IRR, MOIC, DPI, TVPI](#56-performance-metrics--irr-moic-dpi-tvpi)
6. [Fundraising and the LP Universe](#6-fundraising-and-the-lp-universe)
   - 6.1 [The LP Segments](#61-the-lp-segments)
   - 6.2 [The ILPA Private Equity Principles](#62-the-ilpa-private-equity-principles)
   - 6.3 [The LPAC and Fund Governance](#63-the-lpac-and-fund-governance)
   - 6.4 [The Fundraising Cycle](#64-the-fundraising-cycle)
7. [Regulation — AIFMD, the SEC Private Fund Rules, and Fund Governance](#7-regulation--aifmd-the-sec-private-fund-rules-and-fund-governance)
   - 7.1 [The EU AIFMD (Directive 2011/61/EU)](#71-the-eu-aifmd-directive-201161eu)
   - 7.2 [AIFMD II (Directive (EU) 2024/927)](#72-aifmd-ii-directive-eu-2024927)
   - 7.3 [The SEC Private Fund Advisers Rules — Adopted 2023, Vacated 2024](#73-the-sec-private-fund-advisers-rules--adopted-2023-vacated-2024)
   - 7.4 [The Governance Layer — Clawbacks, Key Person, Preferential Treatment](#74-the-governance-layer--clawbacks-key-person-preferential-treatment)
8. [The Singapore Regime and the Asia-Pacific Landscape](#8-the-singapore-regime-and-the-asia-pacific-landscape)
   - 8.1 [The VCC — the Variable Capital Company](#81-the-vcc--the-variable-capital-company)
   - 8.2 [The Fund Tax Incentives — 13O, 13U, 13OA, 13D](#82-the-fund-tax-incentives--13o-13u-13oa-13d)
   - 8.3 [MAS Fund-Manager Licensing](#83-mas-fund-manager-licensing)
   - 8.4 [The Singapore Hub in Numbers](#84-the-singapore-hub-in-numbers)
   - 8.5 [The Asia-Pacific Firm Landscape](#85-the-asia-pacific-firm-landscape)
9. [The Cymbal Bank Angle — Banking the PE Sponsor](#9-the-cymbal-bank-angle--banking-the-pe-sponsor)
   - 9.1 [The Product Map](#91-the-product-map)
   - 9.2 [The Credit Risk of Fund Finance](#92-the-credit-risk-of-fund-finance)
   - 9.3 [KYC/AML for Fund Clients](#93-kyc-aml-for-fund-clients)
   - 9.4 [The Relationship Logic](#94-the-relationship-logic)
10. [Worked Examples — Subscription-Line Facility and a Fully Worked LBO](#10-worked-examples--subscription-line-facility-and-a-fully-worked-lbo)
    - 10.1 [Worked Example A — the Subscription-Line Facility](#101-worked-example-a--the-subscription-line-facility)
    - 10.2 [Worked Example B — a Fully Worked LBO](#102-worked-example-b--a-fully-worked-lbo)
    - 10.3 [Worked Example C — the Waterfall in Action](#103-worked-example-c--the-waterfall-in-action)
11. [Claims Audit](#11-claims-audit)
12. [What Could Not Be Verified](#12-what-could-not-be-verified)
13. [References and Further Reading](#13-references-and-further-reading)
14. [Glossary](#14-glossary)

---

## 1. What Private Equity Is — the Closed-End Fund Model

### 1.1 The Definition — and What "Private" Means

**Private equity (PE)** is the business of investing in companies that are *not* publicly listed — through funds that pool institutional capital and hold each investment for a fixed, multi-year period before exiting. Two structural facts define the asset class:

1. **The assets are private.** The portfolio companies are unlisted, so there is no public market price; value is estimated by the manager (the GP) under a valuation policy, audited periodically, and realised only at exit. This is the defining difference from public equities, and it is why the metrics in §5.6 (DPI, TVPI, MOIC) exist — "mark-to-market" is replaced by "mark-to-model with realised cash to settle the score".
2. **The vehicle is a closed-end fund.** Investors commit capital for the fund's full life (typically ~10 years, extendable) and cannot redeem at will. Liquidity comes from the fund's exits, not from the investor's right to withdraw.

The term "private equity" is also used loosely for the *whole* private-markets complex (growth, venture, distressed, secondaries, and sometimes private credit). This guide follows the stricter convention: PE is the equity-investing side — buyouts, growth, venture — and private credit is the lending side (see the [Asset-Backed Trading](asset_backed_trading_guide.md) and [Alternatives Front-to-Back](alternatives_front_to_back_operating_model.md) guides for credit).

### 1.2 The Closed-End Structure

A PE fund is a **pooled vehicle with a fixed life**:

| Feature | Typical convention | Notes |
|---|---|---|
| Fund life | ~10 years | Usually 3–5 year investment period + harvest; 2 one-year extensions common, subject to LP consent or LPAC approval |
| Capital | **Committed, not invested upfront** | LPs sign commitments (e.g. US$100M of a US$1B fund); capital is *called* ("drawn down") as deals are made — this is the engine behind subscription-line finance (§9, §10.1) |
| Redemption | None during the life | Liquidity only via distributions from exits; secondary sale of the LP interest is the only exit |
| Vintage | The fund's first-investment year | The performance cohort by which funds are benchmarked |
| Vehicle | Limited partnership, VCC, or equivalent | The legal shell; in Singapore, the VCC (§8.1) or an LP |

Because capital is drawn over time, **DPI/IRR math must be cash-flow weighted**: a US$100M commitment called in year 0 is not the same as one called in year 3. The J-curve (§2.3) and the IRR examples in §10 all hinge on this timing.

### 1.3 Why Private Equity Exists — the Return Sources

The economic case for the asset class rests on four sources of return, three of which are *earned* rather than market-driven:

1. **Operational improvement** — the GP changes how the company runs (margins, growth, cost, management, M&A); the core of the "buy-and-build" model (§5.3).
2. **Multiple arbitrage** — buying at a lower entry multiple than the exit multiple (e.g. 10.0x → 12.0x in the worked LBO, §10.2), driven by growth, de-risking, or a change of buyer universe (strategic buyers pay more than financial buyers).
3. **Deleveraging** — the company's cash flows repay the acquisition debt, so the equity slice grows even if the enterprise value is flat ("deleveraging returns").
4. **Market beta** — general EBITDA and valuation growth in the economy.

The first three are the "alpha" story that justifies illiquidity and fees: a flat-multiple, no-growth LBO still returns equity value through debt paydown alone (the sensitivity case in §10.2 shows ≈25% IRR on a flat multiple).

### 1.4 The Asset-Class Scoreboard — PE vs the Alternatives

| Dimension | Private equity | Venture capital | Private credit | Public equity |
|---|---|---|---|---|
| Target company stage | Mature / cash-generative | Early / pre-profit | Any, debt side | Listed |
| Control | Majority or significant minority | Minority | Covenants, not control | None |
| Return engine | Leverage + operations + multiple | Growth + multiple | Yield + spread | Beta + alpha |
| Fund life | ~10 years | ~10 years | 5–10 years | Open-ended |
| Fee/carry | ~2% / 20% (convention, §3) | ~2% / 20% (convention) | ~1–1.5% / 10–15% (⚠ varies) | ~0.1–1% |

The boundaries are fuzzy: growth equity sits between VC and buyout (§4.3), and "private equity" as a department inside a bank usually means the buyout/growth complex. For the Singapore ecosystem view — who the LPs are, which firms operate here, the VCC/13O/13U wrapper — see the [Singapore Private Markets](singapore_private_markets_guide.md) guide; this guide is the *mechanics* layer underneath it.

### 1.5 Private Equity in One Page

| Question | Answer |
|---|---|
| What is it? | Closed-end funds investing in unlisted companies (§1.1) |
| Who runs it? | The GP; the investors are LPs (§2.1) |
| How long? | ~10-year fund life; 3–5-year investment period; exits from year 3 (§2.2) |
| What does it cost? | ~2% management fee + 20% carry, with hurdles and catch-ups (§3) |
| What deals? | LBOs, growth, venture, distressed, secondaries, continuation funds (§4) |
| How are returns made? | EBITDA growth, multiple arbitrage, deleveraging (§5.4) |
| How is it measured? | IRR, MOIC, DPI, TVPI (§5.6) |
| Who pays for it? | Pensions, sovereigns, endowments, insurers, FoFs, family offices (§6) |
| Who regulates it? | EU AIFMD/AIFMD II, SEC (vacated 2024 rules), MAS in Singapore (§7–§8) |
| What does a bank sell it? | Subscription lines, NAV facilities, GP financing, custody, admin, AML (§9–§10) |

---

## 2. The GP/LP Structure and the Fund Lifecycle

### 2.1 General Partner vs Limited Partner

Every PE fund is built on the **GP/LP split** — two legally distinct roles with opposite economics:

| Role | Who | What they do | What they get |
|---|---|---|---|
| **General Partner (GP)** | The fund manager — the firm (KKR, Hillhouse, a Cymbal Bank client) | Sources deals, runs the portfolio, decides exits, *manages* the fund | Management fee (§3.1) + carried interest (§3.2) + a share of the economics; **unlimited liability** for the partnership's obligations (mitigated by the corporate GP entity) |
| **Limited Partner (LP)** | The investors — pension funds, sovereign wealth funds, endowments, insurers, fund-of-funds, family offices (§6.1) | Provide the capital; oversight via the LPAC (§6.3); **no** day-to-day management | The fund's returns net of fees and carry; **liability limited** to their commitment |

The GP typically contributes a **GP commitment** of 1–3% of the fund (⚠ convention — see §3.5) so that "we eat our own cooking", and the whole structure is documented in the **limited partnership agreement (LPA)**: the constitutional document that fixes fees, the waterfall (§3.4), the investment period, key-person provisions, and the LPAC's powers. The ILPA Principles (§6.2) exist precisely because the LPA is a negotiated document with a structural power imbalance between GP and LP.

### 2.2 The Fund Lifecycle — Raise, Invest, Manage, Exit

The canonical ~10-year cycle:

1. **Raise (6–18 months).** The GP markets the fund to LPs on the strength of its track record, team, and strategy; LPs sign commitments. A fund is "closed" when the target (e.g. US$1B) is reached; **final close** typically happens within 12–18 months of first close. Fundraising economics: the GP typically earns little carry until exits, so fees fund the firm (§3.1).
2. **Invest (years 0–4/5, the "investment period").** Capital is called as deals close; the fund builds a portfolio of typically 10–25 companies ("concentration" varies by strategy; buyout funds concentrate, venture funds diversify). Fees are charged on **committed** capital during this period.
3. **Manage (years 2–8, overlapping).** The GP drives the value-creation plan (§5.4): board seats, management changes, add-on acquisitions, operational programmes. Portfolio companies are marked at fair value (typically under IFRS 13 / ASC 820 style frameworks or the industry's valuation guidelines) — the source of the "mark-to-model" caveat in §1.1.
4. **Exit (years 3–10, the "harvest period").** Companies are sold via the exit routes of §5.5; proceeds flow through the waterfall (§3.4) to LPs first, then carry to the GP. After the ~10-year term (plus extensions), the fund is wound down; any residual assets may be distributed in kind (stock) or via continuation vehicles (§4.6).

The lifecycle in one table:

| Phase | Years | What happens | Cash-flow sign |
|---|---|---|---|
| Fundraising | −1 to 0 | Marketing, commitments, first/final close | None yet |
| Investment period | 0–4/5 | Capital calls, deal execution, portfolio build | Negative (calls > returns) |
| Management | 2–8 | Value creation (§5.4), add-ons, monitoring | Around zero (fees out, small dividends in) |
| Harvest | 3–10 | Exits, distributions, wind-down | Strongly positive |
| Extensions | 10–12 | LPAC/LP-consented 1–2 year extensions; continuation funds (§4.6) | As negotiated |

The **vintage year** (first investment year) is the fund's performance cohort: LPs benchmark a 2019-vintage Asia buyout fund against other 2019-vintage funds, which is why IRR relative to vintage peers — not absolute IRR — is the allocator's first screen.

### 2.3 The J-Curve

Because fees and deal costs are paid from the start while exits arrive only later, a fund's cumulative net cash flow and its IRR **dip negative in the early years and recover** — the **J-curve**:

| Year | Fees (US$) | Invested (US$) | Distributions (US$) | Cumulative net CF (US$) | Fund NAV (US$, illustrative) |
|---|---|---|---|---|---|
| 0 | 2.0 | 30.0 | 0.0 | **(32.0)** | 28 |
| 1 | 2.0 | 35.0 | 0.0 | **(69.0)** | 62 |
| 2 | 2.0 | 20.0 | 0.0 | **(91.0)** | 78 |
| 3 | 2.0 | 0.0 | 10.0 | **(83.0)** | 88 |
| 4 | 2.0 | 0.0 | 20.0 | **(65.0)** | 85 |
| 5 | 1.5 | 0.0 | 35.0 | **(31.5)** | 75 |
| 6 | 1.5 | 0.0 | 50.0 | **+17.0** | 62 |
| 7 | 1.5 | 0.0 | 55.0 | +70.5 | 48 |
| 8 | 1.0 | 0.0 | 60.0 | +129.5 | 28 |
| 9 | 1.0 | 0.0 | 60.0 | +188.5 | 10 |
| 10 | 0.0 | 0.0 | 30.0 | +218.5 | 0 |

*Illustrative, per US$100 of commitments; fees = 2.0% on committed capital in the investment period stepping down to 1.5%/1.0%; 85% of capital deployed; total distributions US$320; net to LPs US$218.5 (≈2.2x net MOIC). The NAV column is an illustrative mark of the remaining portfolio; the shape — negative for ~5–6 years, then steep recovery — is the point.*

The J-curve matters for three audiences: **LPs** (an early "denominator effect": the fund's low early NAV drags the LP's private-markets allocation); **GPs** (fundraising for Fund II happens while Fund I still shows a negative IRR — track record credibility is everything); and **banks** (the fund's early cash-flow profile is exactly why subscription lines, §9/§10.1, exist: the fund is *committed* capital that has not yet been *called*).

---

## 3. Fund Economics — Fees, Carry, and the Distribution Waterfall

The GP's compensation has two legs — the **management fee** (payable regardless of performance) and **carried interest** ("carry", the performance share) — plus the GP's own commitment (§3.5). The shorthand **"2 and 20"** (2% management fee, 20% carry) is the industry's standard convention, but it is a *convention*, not a law of nature: ⚠ the exact origin of the "2 and 20" package cannot be pinned to a single primary source — it is widely attributed to the hedge-fund industry of the 1960s–80s (Alfred Winslow Jones's 20% profit share from 1949 is the most-cited ancestor) and to the early buyout/venture firms of the 1980s, but no primary document establishes a single moment of origin. The numbers also compress: large LPs routinely negotiate 1.5% or less and 20% with a hurdle (§3.3), and fee compression is an industry-wide trend (⚠ directional).

### 3.1 The Management Fee

The management fee pays the firm's operating costs (people, offices, deal sourcing) and is typically:

- **2.0% per annum on committed capital** during the investment period (the "2" of "2 and 20");
- **Stepping down** after the investment period — commonly to 1.5% then 1.0%, and often charged on **cost or NAV** rather than commitments (a "fee step-down" aligned with the fund's declining need for the GP's services);
- **Reduced** (sometimes waived entirely) on recycling or on capital not yet called.

Worked: a US$1.0B fund at 2.0% on commitments for 5 years = **US$20M/yr, US$100M over the investment period**; with step-downs (1.5% on cost years 6–7, 1.0% years 8–9) total fees over life ≈ **US$145M** — roughly 13–15% of commitments, which is the industry ballpark (⚠ convention; actual terms are LPA-negotiated). Fee **offsets** (the GP rebating deal/transaction fees it earns from portfolio companies against the management fee) and **fee waterfalls** (deal fees shared with LPs) are standard ILPA-aligned terms (§6.2) — and were a central target of the SEC's now-vacated private fund rules (§7.3).

### 3.2 Carried Interest — the 20% Convention

**Carried interest** is the GP's share of the fund's profits — the "20" of "2 and 20": the GP receives **20% of the profits** (above return of capital and, usually, a hurdle) once the LPs have been paid their entitlement. Key properties:

- Carry is **asymmetric**: the GP shares in upside but (absent a GP commitment, §3.5) does not share proportionally in losses — hence the LP-side demand for GP commitments, hurdles, and clawbacks.
- Carry is typically **0% until the LPs have received their capital back** (and often an 8% preferred return) under a whole-fund waterfall (§3.4); under a deal-by-deal waterfall the GP takes carry per deal and the LPs rely on a **clawback** at fund end.
- Tax treatment of carry varies by jurisdiction (capital-gains-like treatment in some, ordinary income in others) — a policy battleground; ⚠ no cross-jurisdiction generalisation is attempted here.

### 3.3 The Catch-Up and the Hurdle

The **hurdle** (preferred return) and the **catch-up** are the mechanisms that make 20% a *fair* 20%:

- **Hurdle / preferred return:** LPs must receive a return on their capital — commonly **8% per annum** — before the GP earns carry. The hurdle can be simple or compounded, and can apply per deal or per fund.
- **Catch-up:** after the LPs have received capital plus hurdle, the GP "catches up" — receiving distributions at an accelerated rate (commonly 100%) until the GP's cumulative share equals the agreed carry percentage (20%) of total profits. A **50% catch-up** (GP and LP share the catch-up zone 50/50) is a common LP-friendly variant.

The arithmetic is in Worked Example C (§10.3) — a 100% catch-up that lands the GP at exactly 20% of profits, and the common LP-negotiated alternatives.

### 3.4 The Distribution Waterfall — Deal-by-Deal vs Whole-Fund

The **waterfall** is the order in which distributions are split between LPs and GP. Two families dominate, with terminological caveats:

| Waterfall | Also called (⚠ convention) | How it works | GP-favourable / LP-favourable |
|---|---|---|---|
| **Deal-by-deal** | "American-style" waterfall | Carry is calculated and paid **per realised investment**, as each deal exits; if the fund's final whole-fund economics leave the GP overpaid, a **clawback** (GP must return excess carry, often gross of tax) restores the LP position | GP-favourable (early carry), mitigated by clawback |
| **Whole-fund** | "European-style" waterfall | Carry is calculated **only at the fund level**: LPs first receive 100% of distributions until their contributed capital is returned, then the hurdle, then the GP's catch-up; only then 80/20 | LP-favourable (no carry before LPs are whole) |

⚠ The "American vs European" naming is *industry usage, not a formal standard* — a fund's LPA defines the actual mechanics, and hybrids (e.g. whole-fund with deal-level true-up) are common. The distinction matters to banks too: under a deal-by-deal waterfall, early distributions to LPs can be followed by clawback obligations, which affects the credit analysis of **GP financing** (§9.1).

### 3.5 The GP Commitment

The **GP commitment** — the GP's own capital invested in the fund — is the single most direct alignment mechanism. ILPA's Principles (§6.2) treat a meaningful GP commitment as core to "alignment of interest". Market practice: commonly **1–3% of fund size** (⚠ convention, LPA-negotiated; some large firms commit more, some emerging managers commit less or finance the commitment via a GP financing facility — see §9.1). The GP commitment is usually funded *pari passu* with LP calls and earns the same economics minus the carry on its own share.

---

## 4. Deal Types — LBO, Growth Equity, Venture, Distressed, Secondaries

### 4.1 The Deal-Type Map

| Deal type | Company stage | Capital role | Control | Return engine |
|---|---|---|---|---|
| **Buyout / LBO** | Mature, cash-generative | Acquisition of control, funded with debt | Control | Leverage + operations + multiple (§5.4) |
| **MBO / MBIs** | Mature | Same as LBO; incumbent management is the buyer | Control | Same as LBO |
| **Growth equity** | Scaling, often profitable | Minority capital for expansion | Minority (board seat) | Growth + multiple |
| **Venture capital** | Early, often pre-revenue | Seed/A/B rounds | Minority | Growth + multiple (binary) |
| **Distressed / special situations** | Financial or operational trouble | Rescue capital / debt-to-equity | Control or influence | Turnaround + entry discount |
| **Secondaries** | Any | Buying existing LP interests or portfolios | n/a (LP or GP-led) | Discount + duration |
| **Continuation funds** | Any (GP-led) | Rolling a company into a new vehicle | Control | Extended value-creation runway |

### 4.2 Buyout (LBO) and MBO

The **leveraged buyout (LBO)** is the archetypal PE deal: a sponsor acquires a company using a mix of **debt and equity**, with the target's cash flows and assets supporting the debt. "Leveraged" is the point — debt amplifies the equity return (and the risk). The mechanics are §5 and the full arithmetic is §10.2. An **MBO (management buyout)** is an LBO where the incumbent management team is (part of) the buyer; an **MBI (management buy-in)** brings in external management. In Asia, true control buyouts are a smaller share of the market than in the US/Europe — growth and minority deals dominate (⚠ directional, see §8.5) — but the LBO machinery is the intellectual core of the asset class and of every bank's PE coverage model.

### 4.3 Growth Equity

**Growth equity** sits between VC and buyout: minority investments (10–40%) in companies that are already scaling — often profitable, with a product-market fit proven — to fund expansion, geographic entry, or M&A. Less leverage, less control, less operational surgery than buyout; more governance and rigour than venture. For Asia-Pacific funds (Hillhouse's flagship funds, §8.5), growth is frequently the default strategy because founder-owners resist ceding control and banks' LBO debt is thinner.

### 4.4 Venture Capital — the VC-vs-PE Boundary

**Venture capital** finances early-stage companies (pre-revenue to early scaling) in exchange for minority equity. The VC-vs-PE boundary is drawn at **company maturity and cash-flow profile**: VC targets companies that may have no EBITDA to lever or even no revenue; PE targets companies with cash flows that can service debt. Consequence: VC returns are **binary** (most investments fail; a few return the fund), so VC funds need larger diversification and different metrics (TVPI matters more than IRR in early years; "mark" discipline differs). The fee convention overlaps ("2 and 20" is common in both — ⚠ convention), but VC funds rarely use LBO-style debt, and the *manager* regulatory regimes differ — in Singapore, the dedicated **Venture Capital Fund Manager (VCFM)** licence (§8.3) exists precisely because MAS treats VC managers as a distinct category.

### 4.5 Distressed and Special Situations

**Distressed PE** buys debt or equity of companies in financial or operational distress — often converting debt to equity in a restructuring — and profits from the entry discount and the turnaround. **Special situations** is the broader umbrella (event-driven, balance-sheet, regulatory-change plays). In Asia the segment has grown around stressed real estate and leveraged balance sheets (⚠ directional). The credit side of this world — the debt instruments themselves — is covered in the [Asset-Backed Trading](asset_backed_trading_guide.md) guide.

### 4.6 Secondaries and Continuation Funds

**Secondaries** are the market for *existing* private-markets positions:

- **LP-led secondaries:** an existing LP sells its fund interest (or a portfolio of interests) to a secondary buyer, usually at a discount to NAV (or premium for sought-after funds). This is the private-markets *liquidity* mechanism — the only way out before distributions. Asia's leading platform is NewQuest (TPG NewQuest) — see the [Singapore Private Markets](singapore_private_markets_guide.md) guide §4.
- **GP-led secondaries / continuation funds:** the GP rolls one or more portfolio companies out of a maturing fund into a new vehicle ("continuation fund"), often with new capital and new LPs, extending the hold period while giving existing LPs the choice to cash out or roll over. Continuation funds boomed in the low-exit-rate 2020s (⚠ directional) and are a growing source of fund-finance demand (NAV facilities, §9.1).

### 4.7 The Deal Process — From Sourcing to the 100-Day Plan

Whatever the deal type, the transaction follows a standard industrial process — the machine a bank's sponsor-coverage team works alongside:

1. **Sourcing (3–12 months before signing).** Proprietary deal flow (owner-operator relationships, industry networks, auction processes run by sell-side advisers). Asia's founder-owned market makes *relationship sourcing* dominant (⚠ directional — fewer formal auctions than the US).
2. **Screening and diligence (2–4 months).** Commercial, financial, legal, tax, and (increasingly) ESG/cyber due diligence; the **quality-of-earnings (QoE)** review is the financial cornerstone — it normalises EBITDA to a sustainable number, and the debt and equity sizing of §5.2/§10.2 is built on the *normalised* EBITDA.
3. **The Information Memorandum and the price.** Valuation on EV/EBITDA multiples (§5.2), discounted cash flow, and comparable transactions; the purchase agreement (SPA) with price-adjustment (locked-box vs completion-accounts) mechanics.
4. **Financing.** The debt commitment letter from the bank syndicate (senior) and mezzanine/unitranche providers (§5.1); the equity cheque from the fund — the moment the subscription line (§9, §10.1) usually funds.
5. **Closing and the 100-day plan.** Ownership transfers; the value-creation plan (§5.4) kicks off: management changes, the add-on pipeline, working-capital and capex programmes, board cadence.

The bank appears at steps 4 (financing) and 5 (treasury, FX, payments for the portfolio company) — and, as the fund's subscription-line lender, at step 1 in the sense that the *fund's* ability to close fast is financed by the line (§10.1).

---

## 5. LBO Mechanics — the Debt Stack, Value Creation, Exits, and Performance

### 5.1 The Debt Stack — Senior, Mezzanine, Unitranche

An LBO's debt is a **stack** — layers with different seniority, pricing, and risk:

| Layer | Seniority | Typical pricing (⚠ indicative) | Features |
|---|---|---|---|
| **Senior secured debt** (term loan + revolving credit facility) | First lien | SOFR/SORA + 300–500 bps | Amortising, asset-backed, covenant-lite in strong markets |
| **Mezzanine debt** | Subordinated (2nd lien / unsecured) | 8–12% coupon, often with warrants/equity kicker | Fixed or floating, bullet or light amortisation, equity sweetener |
| **Unitranche** | Single blended layer (senior + mezz merged) | SOFR + 500–700 bps (⚠) | One lender group (private credit funds); faster execution, higher price |
| **Holdco / PIK notes** | Lowest | High coupon paid in kind (PIK) | Funds the equity cheque or interest; last-out risk |

In Asia, **unitranche and mezzanine** play a larger role than the US-style fully-syndicated senior market because bank term-loan supply is thinner (⚠ directional) — one reason private credit funds (see the [Partners Group Competitors](partners_group_competitors_guide.md) guide) and banks' sponsor-coverage desks cooperate. The **revolving credit facility** inside the senior layer funds working capital and, importantly for the bank angle, is often sized against the target's cash flows rather than the sponsor's balance sheet.

### 5.2 Leverage Multiples and Credit Metrics

Leverage is expressed as **debt/EBITDA multiples**. Convention (⚠ market-dependent, varies by region and cycle):

- **Total leverage** at entry: 3.0–5.5x EBITDA for mid-market buyouts; up to 6–7x in hot markets; lower in Asia (⚠ directional).
- **Senior leverage**: typically 2.5–4.0x; the senior layer is sized so that **senior debt service is covered** — i.e. EBITDA/interest ≈ 2.0–3.0x at entry, rising as debt amortises and EBITDA grows.
- The worked LBO (§10.2) uses 6.5x total (5.0x senior + 1.5x mezzanine) — deliberately mid-market; the interest-coverage check is shown in the arithmetic.

Credit metrics that lenders monitor: **leverage ratio** (net debt/EBITDA), **interest coverage** (EBITDA/interest), **fixed-charge coverage**, and **cash-flow headroom** for the amortisation schedule.

### 5.3 Buy-and-Build

**Buy-and-build (B&B)** — the "platform plus add-ons" strategy — is the dominant value-creation playbook: the fund acquires a **platform** company, then acquires smaller **add-on** companies that are bolted on to create scale, consolidate a fragmented market, expand geography, or add capabilities. The economics: each add-on is typically bought at a lower multiple than the platform's eventual exit multiple, so **multiple arbitrage is harvested on the whole group at exit**, not just on organic growth. B&B is why "bolt-on M&A" is the most common question in a sponsor's board meeting — and why the fund's debt documents need **permitted-acquisition** headroom (the senior facility's "accordion" or incremental facility).

### 5.4 The Value-Creation Levers

The standard decomposition of an LBO's equity return (worked numerically in §10.2):

1. **EBITDA growth** — organic (pricing, volume, margin) and inorganic (add-ons). The biggest lever over a 5-year hold.
2. **Multiple expansion/arbitrage** — exit multiple minus entry multiple, applied to *exit* EBITDA. Driven by scale, growth profile, de-risking, and the strategic-buyer premium.
3. **Deleveraging** — debt repayment from free cash flow; every dollar of debt repaid is a dollar of equity value created (1:1, before exit-multiple effects).
4. **Cash-flow engineering** — working-capital release, tax optimisation, capital-expenditure discipline (the least glamorous and most reliable lever).

### 5.5 The Exit Routes

| Exit route | Description | When it dominates |
|---|---|---|
| **Trade sale** | Sale to a strategic buyer (corporates pay control premia) | Most of the time; the classic "sell to the consolidator" |
| **IPO / public listing** | Listing on a stock exchange; the sponsor sells down over time | Strong public markets; the *brand* exit (Courts Asia by BPEA, 2012 — §8.5) |
| **Secondary sale** | Sale to another PE firm ("sponsor-to-sponsor") | When public markets are weak; the *market's* exit |
| **Dividend recapitalisation** | The company raises new debt to pay the sponsor a dividend — a *partial* exit without selling | When leverage capacity exists but no buyer; converts NAV into cash |
| **Continuation fund** | Rolling the asset into a new vehicle (§4.6) | When the hold period is wrong for the asset's trajectory |

### 5.6 Performance Metrics — IRR, MOIC, DPI, TVPI

The four metrics every PE professional (and every fund-finance banker) uses:

| Metric | Meaning | Formula | Notes |
|---|---|---|---|
| **IRR** | Internal rate of return — the annualised, cash-flow-weighted return | Solve r: Σ CFₜ/(1+r)ᵗ = 0 | The industry headline metric; *timing-sensitive* — a 1.5x MOIC over 2 years beats 2.0x over 6 years on IRR |
| **MOIC** | Multiple on invested capital — the simple gross multiple | (Total distributions + residual value) / invested capital | Timing-blind; the "real money" metric |
| **DPI** | Distributions to paid-in capital — realised returns | Cumulative distributions / cumulative called capital | The honesty metric: cash actually returned |
| **TVPI** | Total value to paid-in capital — realised + unrealised | (Distributions + NAV) / paid-in capital | The headline "paper" multiple; NAV is a mark, not cash |

Worked fund-level example: a US$1.0B-commitment fund that has called US$900M, distributed US$1,800M, and holds a US$300M NAV: **DPI = 1,800/1,000 = 1.8x**, **RVPI = 300/1,000 = 0.3x**, **TVPI = 2.1x** (note the denominators: paid-in *capital* = commitments called, here approximated as the full US$1.0B). IRR is computed on the actual cash-flow dates — the subscription-line timing effect in §10.1 exists precisely because IRR is date-sensitive.

---

## 6. Fundraising and the LP Universe

### 6.1 The LP Segments

The demand side of private equity is institutional capital:

| LP segment | Profile | Why they invest | Repo cross-reference |
|---|---|---|---|
| **Public pension funds** | The largest allocators; multi-decade liabilities | Illiquidity premium, inflation-linked returns, cash-flow matching | [Norges Bank Investment Management](norges_bank_investment_management_guide.md) (the SWF archetype) |
| **Sovereign wealth funds** | State-owned pools; long horizons, low return targets | Strategic exposure, domestic economy, co-investment | [Norges Bank Investment Management](norges_bank_investment_management_guide.md); [Singapore Private Markets](singapore_private_markets_guide.md) §2 (Temasek, GIC) |
| **Endowments and foundations** | Yale-model allocators; pioneered illiquid "alternatives" | The endowment model: alpha from illiquidity | [Asset Management & Alternatives](asset_management_alternatives_guide.md) |
| **Insurers** | Long liabilities; solvency-regulated (risk-based capital) | Matching long-dated cash flows; ⚠ solvency capital charges vary | [Insurance Products, Processes & Compliance](insurance_products_processes_compliance_guide.md) |
| **Fund-of-funds** | Diversified PE portfolios for smaller allocators | Access + diversification; pay a second layer of fees | [Partners Group](partners_group_company_guide.md) (the listed FoF archetype) |
| **Family offices** | The fastest-growing SG segment (2,000+ SFOs, end-2024 — Verified, see [Singapore Private Markets](singapore_private_markets_guide.md) §6) | Direct PE, co-investment, bespoke vehicles (13D/13O/13U) | [Singapore Private Markets](singapore_private_markets_guide.md) §6 |

Two structural trends shape the LP universe (⚠ directional): **co-investment** — large LPs investing directly alongside funds, fee-free, which has made LPs into quasi-GPs — and **concentration** — the largest 20–30 allocators now anchor most mega-funds, so a GP's fundraising is really a relationship with a small number of institutions.

### 6.2 The ILPA Private Equity Principles

The **Institutional Limited Partners Association (ILPA)** — the LP trade body — publishes the **ILPA Private Equity Principles**, the industry's governance canon. Verified timeline (ilpa.org):

- **September 2009** — the Principles are first released, following ILPA member round-tables in **March 2009** and a membership survey over summer 2009.
- **January 2011** — **Principles 2.0**, adding (among other things) the **clawback appendix**.
- **June 2019** — **Principles 3.0**, the current edition, covering GP and fund economics, fund term and structure, key person, fund governance, financial disclosures, notification and policy disclosures, and LP disclosures.

The three guiding principles have been constant across all editions: **alignment of interest, governance, and transparency**. Concretely, ILPA-endorsed terms include: meaningful GP commitments; fee and expense transparency (including fee offsets); LPAC consultation on key-person changes, extensions, and conflicts; and clawback mechanics. For a bank, the ILPA Principles are also the *documentation map*: every term they touch (fees, waterfall, LPAC powers, key person) shows up in fund-finance and custody due diligence.

### 6.3 The LPAC and Fund Governance

The **Limited Partner Advisory Committee (LPAC)** is the LPs' governance organ: a small group of LPs (typically 3–7) elected to consult (not decide, unless the LPA says so) on conflicts, valuations, extensions, key-person events, related-party transactions, and litigation. The LPAC is not a board and has no fiduciary duty to the other LPs (⚠ important legal nuance: its role is defined entirely by the LPA). In practice the LPAC is where fund-governance disputes are resolved *before* they become legal disputes — which is why ILPA's Principles devote a full section to LPAC composition, quorum, and powers.

### 6.4 The Fundraising Cycle

A GP raising Fund N+1 typically: (1) tests the market with a **pre-marketing** period; (2) holds **first close** once anchor LPs (often 30–50% of target) commit; (3) continues to **final close** within 12–18 months; (4) files the necessary regulatory notices in each LP's jurisdiction (AIFMD marketing rules in the EU, §7.1; SEC Form ADV and private-fund reporting in the US, §7.3; MAS rules in Singapore, §8). Fundraising is **pro-cyclical**: vintages raised in strong markets get the most capital and often the worst entry prices — the "denominator effect" and "dry powder" debates live here. **Dry powder** (committed but uncalled capital) is the industry's forward indicator, and it is also the collateral base for subscription-line finance (§9, §10.1) — the direct link between the LP universe and the bank.

The LP's diligence list — the questions every allocator (and every fund-finance bank reviewing the same fund) asks:

| Area | The questions |
|---|---|
| Track record | Vintage-by-vintage IRR/MOIC/DPI; is the *next* fund's thesis the same as the *last* fund's realised returns? |
| People | Key-person clauses (§7.4); team continuity; succession; how much of the economics key people actually own |
| Alignment | GP commitment size (§3.5); fee offsets; how the GP's own capital is funded (GP financing is a yellow flag for LPs) |
| Economics | Fee schedule and step-downs; carry structure and hurdle; waterfall type (§3.4); clawback mechanics (§7.4) |
| Valuations | Valuation policy and marks; auditor; write-down history (the honesty test) |
| Governance | LPAC composition and powers (§6.3); side-letter disclosure; conflicts and related-party policies |
| Operations | Fund administrator; depositary/custodian; AML/KYC infrastructure (the bank's world, §9.3) |

---

## 7. Regulation — AIFMD, the SEC Private Fund Rules, and Fund Governance

### 7.1 The EU AIFMD (Directive 2011/61/EU)

The **Alternative Investment Fund Managers Directive** — **Directive 2011/61/EU** of the European Parliament and of the Council, **adopted 8 June 2011** (OJ L 174 of 1 July 2011, pp. 1–73; entry into force **21 July 2011**; member states required to transpose by **22 July 2013**) — is the EU's regime for managers of alternative investment funds (AIFs): hedge funds, private equity, real estate, and other non-UCITS funds. Verified scope and mechanics (eur-lex.europa.eu):

- **Authorisation:** EU AIFMs managing AIFs above **€100M of AUM (leveraged AIFs)** or **€500M (unleveraged, no redemption rights for 5 years — the PE carve-out)** must be authorised by their home-state regulator; below the thresholds, lighter registration applies.
- **Conduct:** AIFMs need capital, sound organisation, a depositary for each AIF, annual reports, and **Annex IV supervisory reporting** (leverage, liquidity, risk) to the home regulator and ESMA.
- **The PE-specific provisions (Chapter V, Articles 26–30):** notification of **control** (defined as >50% of voting rights, including club deals) to the investee company and its shareholders; disclosure of the acquisition to employees/representatives; an annual report obligation on the *AIFM* regarding controlled companies; and **anti-asset-stripping rules** restricting distributions, capital reductions, and share buy-backs in the first **24 months** after control.
- **Marketing:** an EU-authorised AIFM can market EU AIFs to professional investors EU-wide under the **passport**; non-EU managers/funds rely on national private placement regimes (with reporting, cooperation-agreement, and FATF conditions).

The scope matrix in one table:

| AIFM | AIF | Regime |
|---|---|---|
| EU | EU | Full authorisation + EU marketing passport |
| EU | non-EU | Authorisation; EU marketing of the non-EU AIF via passport (or NPPR during transition) |
| non-EU | EU | Authorisation required to manage; marketing via NPPR |
| non-EU | non-EU | No authorisation to manage; marketing into the EU only via NPPR (subject to reporting and FATF/cooperation conditions) |

The AIFMD does not directly regulate a Singapore-domiciled fund marketed only in Asia — but it bites whenever an EU LP or EU marketing is involved, which is why every global fund's legal structure includes an EU-compliant feeder or an EU AIFM entity.

### 7.2 AIFMD II (Directive (EU) 2024/927)

**AIFMD II** — **Directive (EU) 2024/927 of 13 March 2024**, amending Directives 2011/61/EU and 2009/65/EC — was adopted to modernise the regime. Verified scope (eur-lex; Linklaters summary): **delegation arrangements** (an enhanced regime looking beyond portfolio/risk management, targeting "letterbox entities"), **liquidity risk management** (including liquidity management tools for open-ended AIFs), **supervisory reporting**, **depositary and custody services**, and **loan origination by AIFs**. On loan origination — directly relevant to fund finance (§9) — AIFMD II imposes a **retention requirement: an AIF must retain at least 5% of the notional value of any loan it originates and sells** (transitional provisions apply to loan-originating AIFs established before **15 April 2024**). ⚠ The transposition deadline (widely reported as **16 April 2026**) was not re-extracted from the OJ in this pass — treat as reported.

### 7.3 The SEC Private Fund Advisers Rules — Adopted 2023, Vacated 2024

The most consequential recent US attempt to regulate private-fund economics was short-lived:

- **August 2023:** the SEC adopted the **Private Fund Advisers Rules** (88 Fed. Reg. 63206, 23 August 2023) — quarterly statements, mandatory audits, a ban on certain **preferential treatment** of LPs, restrictions on **fee-and-expense allocations** and **GP indemnification**, and a compliance-review amendment for advisers. Compliance dates were to begin **14 September 2024**.
- **June 2024:** in ***National Association of Private Fund Managers v. SEC*** (No. 23-60471), a unanimous Fifth Circuit panel **vacated the rules in full** on 5 June 2024, holding the SEC exceeded its statutory authority under Sections 211(h) and 206(4) of the Investment Advisers Act 1940. The SEC's own announcement of the decision is on sec.gov.
- **Current state:** the rules are **not in force** (vacated). ⚠ Whether the SEC sought or will seek further review (petition for rehearing/certiorari) was reported but not re-extracted in this pass — treat the litigation status as of June 2024 as the verified anchor.

The episode matters beyond the US: the *substance* of the vacated rules (quarterly reporting, preferential-treatment bans, fee-transparency) tracks what ILPA has pushed for since 2009 (§6.2) and what European and Singapore regulators already require — so the policy direction survives the legal defeat.

### 7.4 The Governance Layer — Clawbacks, Key Person, Preferential Treatment

Regardless of regulator, the LPA's governance machinery is the real rulebook:

- **Clawback:** the GP's obligation to return excess carry (typically gross of tax, sometimes net) if deal-by-deal economics overpay it relative to whole-fund outcomes — the price of the "American-style" waterfall (§3.4).
- **Key-person provisions:** if named individuals (the "key persons") cease to devote sufficient time, the fund typically stops investing (or the LPAC/LPs can suspend the investment period) — the LPA's most important risk switch.
- **Preferential treatment:** side letters granting individual LPs better terms (fee breaks, co-investment rights, most-favoured-nation clauses) are standard practice but must be disclosed; the vacated SEC rule would have banned the most egregious forms. ILPA and LPAC practice require transparency of side letters to the LPAC.
- **Valuation governance:** NAV is a mark under the fund's valuation policy (typically in line with IPEV/IFRS-style guidance), reviewed by the auditor and, for regulated funds, the depositary — the substrate of every TVPI number in §5.6.

---

## 8. The Singapore Regime and the Asia-Pacific Landscape

The Singapore layer is covered *in extenso* in the [Singapore Private Markets](singapore_private_markets_guide.md) guide (§5 — the fund regime) and the [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) guide (the SFA, notices, AML). This section states the verified core and points to those guides rather than re-deriving them.

### 8.1 The VCC — the Variable Capital Company

The **Variable Capital Company (VCC)** is Singapore's bespoke corporate vehicle for investment funds, created by the **Variable Capital Companies Act 2018** (passed by Parliament in 2018; framework launched **14 January 2020**; administered jointly by **MAS and ACRA** — Verified: sso.agc.gov.sg; Clifford Chance briefing). Features: **variable capital** (capital may vary without court/shareholder formalities — a corporate-fund structure rather than a trust or LP), **umbrella structure** (one VCC with multiple sub-funds, each ring-fenced), **redomiciliation** (foreign funds can migrate in), and eligibility for the fund tax incentives (§8.2). The VCC is the natural *fund* shell for a PE manager's onshore Asia vehicle — the LBO fund of §10.2 would, in Singapore, plausibly be a VCC (or a Singapore LP under 13OA) with a CMS-licensed manager.

### 8.2 The Fund Tax Incentives — 13O, 13U, 13OA, 13D

Sections **13D, 13O and 13U of the Income Tax Act 1947** provide tax exemption on specified income from designated investments for funds managed by Singapore-based managers (Verified — RSM Singapore on the MAS circular **FDD Cir 10/2024** of 1 October 2024; sso.agc.gov.sg confirms the current section numbers). Verified parameters:

| Scheme | Profile | Verified key parameters (FDD Cir 10/2024, effective from 1 Jan 2025) |
|---|---|---|
| **13O** (onshore) | SG-incorporated vehicle + SG manager; successor to the former 13R | Min **S$5M** AUM in designated investments (new); ≥ **2 investment professionals** tax-resident in SG; **tiered local business spending** (S$200k / S$300k / S$500k by AUM band) |
| **13U** (enhanced tier) | No SG-incorporation or SG-manager requirement; for larger/offshore funds; successor to the former 13X | Min **S$50M** AUM in designated investments at application and at each FY-end; ≥ **3 investment professionals**; tiered LBS |
| **13OA** | Extends 13O-style treatment to **Singapore limited partnerships** — aimed at small PE/VC funds | New scheme, awards from **1 January 2025** (announced in Budget 2024; the 2024 amendments verified via RSM and DLA Piper commentary) |
| **13D** | The family-office scheme | Successor to 13CA; used by single-family-office vehicles |

All three fund schemes (13D/13O/13U) were **extended to 31 December 2029** (Verified — RSM/PwC on FDD Cir 10/2024). ⚠ The precise year of assessment from which the *renumbering* (13R→13O, 13X→13U) took effect — widely reported as **YA 2020** following the 2019 Budget — could not be re-extracted from primary legislation in this pass (the consolidated ITA on SSO shows only the current numbering); treat as reported. Grace periods apply to funds awarded before 1 Jan 2025 (13O existing funds meet the new AUM/IP/LBS criteria from FY ending 2027 / YA 2028 — Verified, RSM).

### 8.3 MAS Fund-Manager Licensing

Managing a fund from Singapore requires a MAS licence under the **Securities and Futures Act 2001** (SFA) unless an exemption applies (cross-ref [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) §2.3 for the SFA itself). The routes (parameters reported/flagged in the sibling guides; not re-verified here):

- **CMS licence for fund management (LFMC)** — the full route; retail and **Accredited/Institutional (A/I)** sub-categories; base capital, compliance and risk functions, ongoing reporting. The credibility route for third-party money.
- **RFMC (Registered Fund Management Company)** — the lighter former regime for managers serving **fewer than 30 qualified investors** with AUM below a commonly cited **S$250M** cap (⚠ flagged). ⚠ MAS's **2026 streamlined framework** consolidates the RFMC class and the LFMC A/I sub-classes into a single LFMC authorisation with statutory exemptions (reported — Raffles Corporate Services guides; not re-extracted from MAS in this pass).
- **VCFM (Venture Capital Fund Manager)** — a streamlined regime for VC funds investing in unlisted startups (reported, flagged).
- **Exempt manager** — managing only the group's own funds (e.g. a family office's in-house manager).

The AML overlay for fund managers and their banks: **Notice SFA 04-N02** (AML/CFT for capital-markets intermediaries), **Notice 626/626A** (banks), and the **VCC-N01** notice for VCCs — all part of the 30 June 2025 MAS AML revision wave (Verified — see the MAS guide's notice map). The bank-side implications are in §9.3.

### 8.4 The Singapore Hub in Numbers

- **Asset-management industry AUM: S$6.7 trillion** as at end-2025, **+10% YoY**, per the **MAS Singapore Asset Management Survey Report 2025** (Verified — MAS's own release/communications; the sibling guide carries the same figure flagged from press coverage).
- **VCC take-up:** 300+ VCCs by mid-2020 (reported); the VCC + 13O/13U/13OA combination is now the standard onshore Asia fund architecture.
- **Family offices:** 2,000+ single family offices as of end-2024 (+43% YoY — Verified: Reuters; see [Singapore Private Markets](singapore_private_markets_guide.md) §6) — a major LP segment for Asia PE funds (§6.1).

### 8.5 The Asia-Pacific Firm Landscape

The dedicated SG *firm* landscape (Blackstone, KKR, Apollo, Carlyle, TPG, Brookfield, Navis, Tembusu, NewQuest, RRJ — with office years and flags) is in the [Singapore Private Markets](singapore_private_markets_guide.md) guide §3–§4. This section adds the **pan-Asia PE majors** with the facts verified for this pass:

| Firm | Founded / HQ | Verified facts | Flags |
|---|---|---|---|
| **Hillhouse** | **2005**, founded by **Lei Zhang (Zhang Lei)** in China; now Singapore-headquartered | Seed capital **US$20M from the Yale endowment** (via David Swensen); early stakes in Tencent, JD.com, Baidu; **US$18B raised for its fifth flagship fund (Aug 2021) — the largest Asia PE fundraise at the time**, breaking KKR's US$15B pan-Asia fund (2021); GL Ventures VC spin-off 2020 | AUM (~US$100B est. 2026) is an **estimate** ⚠ |
| **BPEA → EQT** | **1997**, Hong Kong (Barings Bank affiliate; independent 2000, Jean Salata) | Fund VIII **US$11.2B (2022)**; acquired by **EQT AB for US$7.5B** (announced March 2022, completed October 2022) → **BPEA EQT** → rebranded **EQT Private Capital Asia (January 2024)** | — |
| **Affinity Equity Partners** | Team roots **1998** as UBS Capital Asia Pacific; spun out **2004**; Hong Kong/Singapore | Pan-Asia buyout/growth; **~US$14B** advised/managed (2023); Affinity Asia Pacific Fund V ~US$6.0B; founder-chairman Tang Kok Yew | 1998 vs 2004 founding nuance ⚠; AUM date-sensitive |
| **PAG** | **2002** as Pacific Alliance Group (hedge-fund/credit origins; co-founders Chris Gradel, Horst Geicke); Hong Kong | **Weijian Shan** (ex-Newbridge/TPG) joined as co-founder/executive chairman and **founded PAG's private equity business in 2010** (pag.com); credit & markets, private equity, real assets | The task-brief characterisation "founded 2002 by Weijian Shan et al." is a simplification: the *firm* is 2002, the *PE business* is 2010 ⚠; AUM not verified |
| **CVC Asia** | CVC founded **1981** as Citicorp Venture Capital; **1993** spinoff; Asia franchise from ~2000 | CVC Asia Fund I **US$750M (2000)**; Asia V US$4.5B (2020); **Asia VI US$6.8B (2023)**; group AUM ~**€205B (2025)**; listed Euronext Amsterdam | Exact Asia-office opening year (1999 vs 2000) ⚠ |
| **KKR Asia** | KKR founded **1976** (Kohlberg Kravis Roberts); first Asia offices **Hong Kong and Tokyo 2005** | **KKR Asian Fund I: 2007 vintage, closed at US$3.98B** (≈US$4B); KKR's SG presence since 2012 (sibling guide, Verified); the US$15B pan-Asia fund (2021) that Hillhouse's US$18B surpassed | KKR 1976 founding is well-documented but was not re-extracted from kkr.com in this pass ⚠ (cross-ref [Singapore Private Markets](singapore_private_markets_guide.md) §3) |

Regional texture: Asia PE skews **growth and minority** over control buyouts (⚠ directional — thinner leverage markets, founder control preferences, public-market exit dependence), which is why the worked LBO in §10.2 is presented as the *mechanics* template rather than the modal Asia deal.

---

## 9. The Cymbal Bank Angle — Banking the PE Sponsor

Cymbal Bank's private-markets coverage treats the PE sponsor as a *relationship*, not a transaction: the fund is the entry point; the sponsor's entire ecosystem (portfolio companies, executives, LPs) is the wallet. The operating-model view of how a bank services alternatives managers end-to-end is in the [Alternatives Front-to-Back Operating Model](alternatives_front_to_back_operating_model.md) guide.

### 9.1 The Product Map

| Product | What it is | Typical structure (⚠ market convention unless noted) |
|---|---|---|
| **Subscription line / capital-call facility** | Revolving credit to the *fund*, secured on LPs' **uncalled commitments** — lets the fund invest immediately and call capital later | Size up to **10–25% of total LP commitments**; drawn to fund investments; repaid from capital calls; the bank's flagship PE product — worked example in §10.1 |
| **NAV facility** | Credit secured on the fund's **portfolio NAV** (senior to the LPs' equity, junior to the assets' own debt) | Size commonly ~10–20% of NAV ⚠; used for follow-on capital, distributions, or continuation-fund creation (§4.6); priced wider than subscription lines |
| **GP financing** | Credit to the GP/individual partners — to fund the **GP commitment** (§3.5) or personal liquidity | Small, bespoke, recourse to partners; credit analysis must model the waterfall (§3.4) and clawback (§7.4) — a deal-by-deal fund can *pay* carry early but owe it back |
| **Custody / depositary** | Safe-keeping of fund assets; for EU AIFs, the **depositary** role is mandatory under AIFMD (§7.1) | Bank as custodian under the SFA/FSMA framework; depositary liability rules differ by jurisdiction |
| **Fund administration** | NAV calculation, investor servicing, capital calls/distributions, financial reporting | Frequently the bank's fund-services subsidiary; NAV quality is the bank's own credit risk (it collateralises NAV facilities) |
| **FX, payments, liquidity** | Multi-currency operations, interest-rate hedges (LBO debt is floating, §5.1) | Treasury overlay on every facility above |

The three credit products compared:

| Dimension | Subscription line | NAV facility | GP financing |
|---|---|---|---|
| Collateral | Uncalled LP commitments | Portfolio NAV | GP/partner assets and fund economics |
| Size convention | 10–25% of commitments (⚠) | ~10–20% of NAV (⚠) | Small, bespoke |
| Pricing (⚠ indicative) | SOFR + 150–250 bps | SOFR + 400–700 bps | Highest; recourse |
| Draw purpose | New investments, fast closes | Follow-ons, distributions, continuation funds | GP commitment, partner liquidity |
| Key risk | LP default, concentration, commitment expiry | Mark discipline, valuation policy | Waterfall timing, clawback |
| The bank's diligence focus | Borrowing-base certificate, LP quality | Auditor, valuation policy, NAV volatility | LPA waterfall + GP track record |

### 9.2 The Credit Risk of Fund Finance

Fund finance is asset-based lending where the *collateral is a promise*:

- **Subscription lines** are secured on **uncalled commitments** — so the credit analysis is: (1) **quality of LPs** (a pension fund's commitment is better collateral than a thinly-capitalised family office's); (2) **borrowing-base mechanics** (eligible commitments exclude called, defaulted, or concentrated exposures; advance rates commonly 85–95% on eligible commitments — ⚠ convention); (3) **covenants** (borrowing-base certificates, commitment-expiry triggers — the facility must mature *before* the fund's investment period ends); and (4) **defaulted-LP provisions** (if an LP fails a call, the bank's collateral base shrinks — the LPA's default provisions and the GP's ability to reallocate are part of the bank's underwriting).
- **NAV facilities** replace the promise with **marks**: the bank lends against a haircut of the fund's NAV, so the *valuation policy, auditor, and mark discipline* of §7.4 become the bank's credit risk. A fund that over-marks its NAV is a fund whose NAV facility is over-advanced.
- **Concentration and FX** are the standing risks: a fund whose commitments are 40% one LP (or 60% one country) concentrates the facility; a US$ fund lending into SGD portfolio companies carries currency mismatch through the treasury overlay.
- **Conduct and regulatory risk:** the 2017-era regulatory scrutiny of subscription lines (whether line-drawings should count as "invested capital" for waterfall/IRR purposes — the SEC's 2017 guidance and ILPA's subsequent considerations) means the bank must know *how the fund treats line drawings in its waterfall*; the vacated SEC rules (§7.3) would have added fee/preferential-treatment disclosure on top. ⚠ The 2017 guidance specifics were not re-extracted in this pass — flagged.

### 9.3 KYC/AML for Fund Clients

Fund clients are legal-entity labyrinths (fund → sub-fund → GP → partners → LPs), so KYC/AML is a structural problem, not a form-filling exercise:

- **Entity and UBO identification** through the fund's constitutional chain: the VCC (ACRA-registered, §8.1), the LP, the GP entity, and the *beneficial owners* behind each LP — for pooled funds, the "look-through" question is which LPs are material (typically the ≥10–25% holders and all controlling persons, per the bank's own AML policy).
- **MAS obligations:** Notice 626/626A for banks and **SFA 04-N02** for capital-markets intermediaries (the manager's side) — both refreshed in the **30 June 2025** AML revision wave (Verified — [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) notice map); FATF-based customer due diligence; sanctions screening of fund investors, counterparties, and portfolio companies (sanctions exposure travels through the portfolio: a fund holding a sanctioned asset is a sanctioned counterparty risk).
- **Source-of-funds and red flags:** capital-call fraud (fake call notices), diversion of distributions, layering through sub-funds — the detection playbook is in the [Financial Fraud Detection at Scale](financial_fraud_detection_at_scale_guide.md) guide; the identity infrastructure is in the [Distributed Auth](../technology/distributed_auth_guide.md) guide (cross-referenced from the sibling guides).

### 9.4 The Relationship Logic

The bank's PE franchise compounds: the **subscription line** opens the relationship (the fund must bank where the facility is); **custody and administration** make the bank the fund's operational spine; **NAV and GP financing** deepen it; and the **portfolio companies** (payments, FX, trade finance, deposits) plus the **partners' personal banking** are the downstream wallet. The [Alternatives Front-to-Back Operating Model](alternatives_front_to_back_operating_model.md) guide covers the operating model; the worked examples below show the product arithmetic.

---

## 10. Worked Examples — Subscription-Line Facility and a Fully Worked LBO

### 10.1 Worked Example A — the Subscription-Line Facility

**Setup.** A Cymbal Bank client, a mid-market Asia buyout GP, is raising **Fund II: US$1.0B of total LP commitments** (25 LPs; the largest LP is a US public pension fund at 20%, the rest are pensions, insurers, and family offices). The fund's investment period is 5 years. The GP asks for a **subscription-line / capital-call facility**.

**Sizing.** Market convention sizes the line at **up to 10–25% of total LP commitments** (⚠ convention). The bank approves a **US$200M facility (20%)**:

| Parameter | Value | Note |
|---|---|---|
| Facility size | US$200M (20% of US$1.0B commitments) | Within the 10–25% convention |
| Borrowing base | **90% × eligible uncalled commitments** | Eligible = commitments of LPs in good standing, minus already-called amounts, minus the top-LP concentration adjustment |
| Pricing | SOFR + 175 bps drawn; ~35–50% of margin on undrawn (commitment fee) | ⚠ Indicative market range, not a quote |
| Tenor | 5 years, maturing before the end of the investment period | Commitment-expiry discipline |
| Repayment | From LP capital calls; drawings outstanding **≤ 90 days** | ⚠ Common market convention |
| Covenants | Monthly borrowing-base certificate; LP-default trigger; no drawings after investment-period end | Standard |

**Why the GP wants it — the IRR effect.** The fund has a signed deal: **US$200M** for a platform company it will hold **3 years** and sell for **US$400M**. Two funding paths:

- **Option A — call LPs now:** LPs fund US$200M at t=0; proceeds US$400M at t=3. IRR = (400/200)^(1/3) − 1 = 2^(1/3) − 1 = **≈ 26.0%**.
- **Option B — draw the subscription line:** the bank funds the US$200M at t=0 at **6% p.a.**; the GP calls LPs at t=1 to repay **US$212M** (200 × 1.06); the investment still exits at t=3 for US$400M. LP cash flows: −212 at t=1, +400 at t=3. IRR: (1+r)² = 400/212 = 1.8868 → r = **≈ 37.4%**.

The line lifts the *reported* LP IRR from ≈26% to ≈37% — the timing effect of §5.6 made concrete. This is exactly why regulators and ILPA pushed for transparency on whether line drawings count as invested capital in the waterfall (§9.2): the *economic* return is unchanged; the *dated* return is not. The bank's documentation therefore confirms how the fund's LPA treats line drawings (market practice: line borrowings are repaid from capital calls *before* waterfall distributions, and many LPAs now neutralise the IRR effect by treating drawings as if called at the draw date — ⚠ LPA-specific).

**The bank's risk check.** Borrowing base at draw: eligible uncalled commitments = US$1.0B − 0 called − concentration adjustment (top LP 20% counted at reduced advance rate) ≈ US$900M+; 90% × that ≫ US$200M drawn — headroom is ample. The covenant that matters: the borrowing-base certificate must show the US$200M drawing is covered by *non-defaulted, non-concentrated* LP commitments at all times.

### 10.2 Worked Example B — a Fully Worked LBO

**Setup.** Cymbal Bank's sponsor client acquires **Acme Manufacturing** at **10.0x EBITDA**:

| Entry | Value |
|---|---|
| EBITDA (t=0) | **US$100.0M** |
| Enterprise value (10.0x) | **US$1,000.0M** |
| Senior debt (5.0x, SOFR + 300 bps ≈ 6.5% all-in) | **US$500.0M** |
| Mezzanine (1.5x, 10% coupon) | **US$150.0M** |
| Total debt (6.5x) | **US$650.0M** |
| **Equity funded by the fund** | **US$350.0M** |

*(Transaction fees and expenses ignored for clarity — in practice they add ~2–4% of EV to the equity cheque.)*

**The hold.** EBITDA grows **8% p.a.**; the company's free cash flow repays **US$250M** of debt over 5 years (US$50M/yr — senior amortisation plus excess cash):

| Year | EBITDA (US$M) | Debt outstanding (US$M) | Interest (US$M) | EBITDA/interest |
|---|---|---|---|---|
| 0 | 100.0 | 650.0 | 47.5 | 2.1x |
| 1 | 108.0 | 600.0 | 45.8 | 2.4x |
| 2 | 116.6 | 550.0 | 44.2 | 2.6x |
| 3 | 126.0 | 500.0 | 42.5 | 3.0x |
| 4 | 136.0 | 450.0 | 40.8 | 3.3x |
| 5 | **146.9** | **400.0** | 39.2 | **3.7x** |

*(EBITDA: 100 × 1.08⁵ = 146.93. Interest = 6.5% × 500 + 10% × 150 = 32.5 + 15.0 = 47.5, declining as senior amortises; mezzanine assumed non-amortising for simplicity. Coverage 100/47.5 = 2.1x rising to 146.93/39.2 ≈ 3.7x.)*

**The exit.** The company is sold at **12.0x** EBITDA after 5 years:

- Exit EV = **12.0 × 146.93 = US$1,763.19M**
- Less exit debt of **US$400.0M**
- **Exit equity proceeds = US$1,363.19M**

**The return:**

| Metric | Calculation | Result |
|---|---|---|
| **MOIC** | 1,363.19 / 350.00 | **≈ 3.89x** |
| **IRR** | (1,363.19/350)^(1/5) − 1 | **≈ 31.2%** |

**Value-creation attribution** (where the US$1,013.19M of equity profit came from):

| Lever | Calculation | US$M | Share |
|---|---|---|---|
| EBITDA growth | 10.0x × (146.93 − 100.0) | 469.33 | 46% |
| Multiple expansion | (12.0 − 10.0) × 146.93 | 293.87 | 29% |
| Deleveraging | 650.0 − 400.0 | 250.00 | 25% |
| **Total** | 469.33 + 293.87 + 250.00 | **1,013.20** | 100% |

*Check: 350.00 (equity in) + 1,013.20 (profit) = 1,363.20 ≈ 1,363.19 exit proceeds (rounding).*

**Sensitivity — the "flat multiple" case.** Suppose the exit multiple is **10.0x**, not 12.0x: exit EV = 10.0 × 146.93 = US$1,469.33M; equity = 1,469.33 − 400 = **US$1,069.33M**; MOIC = 1,069.33/350 = **≈ 3.06x**; IRR = (1,069.33/350)^(1/5) − 1 = **≈ 25.0%**. Even with *zero* multiple expansion, the deal returns ~25% — the deleveraging + growth levers doing the work (the case for the asset class in §1.3). And if EBITDA *also* stalls (100 flat, 10.0x exit): EV = 1,000; equity = 600; MOIC 1.71x; IRR ≈ 11.4% — the downside case that keeps leverage at 6.5x honest.

**The full sensitivity grid** (exit debt fixed at US$400M; equity US$350M; each cell shows MOIC / IRR):

| Exit multiple \ EBITDA growth (5 yrs) | 4% p.a. (EBITDA 121.7) | 8% p.a. (EBITDA 146.9) | 12% p.a. (EBITDA 176.2) |
|---|---|---|---|
| **10.0x** | 2.33x / 18.5% | 3.06x / 25.0% | 3.89x / 31.2% |
| **11.0x** | 2.68x / 21.8% | 3.48x / 28.3% | 4.40x / 34.5% |
| **12.0x** | 3.03x / 24.8% | 3.89x / 31.2% | 4.90x / 37.4% |

*Worked cell (bottom-right): EV = 12.0 × 176.23 = 2,114.81; equity = 2,114.81 − 400 = 1,714.81; MOIC = 1,714.81/350 = 4.90x; IRR = 4.90^(1/5) − 1 ≈ 37.4%. The grid is the deal's whole economics in nine numbers — which is why the first page of every sponsor's IC memo is exactly this table.*

### 10.3 Worked Example C — the Waterfall in Action

**Setup.** Fund gross proceeds of **US$250 on a US$100 commitment** (2.5x gross MOIC). LPA: 20% carry, **8% p.a. simple preferred return over 5 years**, **100% catch-up**, whole-fund waterfall (§3.3–3.4). Distributions flow in four steps:

| Step | Rule | LP (US$) | GP (US$) | Remaining (US$) |
|---|---|---|---|---|
| 1. Return of capital | LP gets capital back first | 100.00 | 0.00 | 150.00 |
| 2. Preferred return | 8% × 5 yrs × 100 = 40 to LP | 40.00 | 0.00 | 110.00 |
| 3. Catch-up | GP takes 100% until GP = 20% of cumulative profit: C = 0.2 × (40 + C) → **C = 10** | 0.00 | 10.00 | 100.00 |
| 4. 80/20 split | Remaining 100 split 80/20 | 80.00 | 20.00 | 0.00 |
| **Totals** | | **220.00** | **30.00** | 250.00 |

*Check: GP carry = 30 = exactly **20% of the US$150 profit** (30/150); LP net MOIC = 2.20x. If the LPA had a **50% catch-up**, the GP's catch-up zone is halved and the LP share rises — the standard LP-negotiated trade. Under a **deal-by-deal** waterfall, step 2–4 are run per deal, and the LP relies on the clawback (§7.4) to restore whole-fund economics if later deals lose money.*

---

## 11. Claims Audit

| Fact | Status | Source |
|---|---|---|
| ILPA Private Equity Principles first released September 2009 (following March 2009 LP round-tables) | ✅ Verified | ilpa.org (Principles history) |
| ILPA Principles 2.0 released January 2011 (clawback appendix added) | ✅ Verified | ilpa.org; Kirkland & Ellis PEN (11 Jan 2011) |
| ILPA Principles 3.0 published June 2019; three guiding principles: alignment of interest, governance, transparency | ✅ Verified | ilpa.org |
| AIFMD = Directive 2011/61/EU, adopted 8 June 2011, OJ L 174 (1 July 2011), in force 21 July 2011; transposition by 22 July 2013 | ✅ Verified | eur-lex.europa.eu (CELEX 32011L0061) |
| AIFMD authorisation thresholds: €100M (leveraged) / €500M (unleveraged) | ✅ Verified | eur-lex; Wikipedia (Directive 2011/61/EU) |
| AIFMD PE provisions: control = >50% voting rights; anti-asset-stripping for 24 months after control (Articles 26–30) | ✅ Verified | eur-lex / directive text summaries |
| AIFMD II = Directive (EU) 2024/927 of 13 March 2024 (delegation, liquidity risk mgmt, supervisory reporting, depositary/custody, loan origination); ≥5% loan-retention for loan-originating AIFs | ✅ Verified | eur-lex (2024/927); Linklaters |
| AIFMD II transposition deadline of 16 April 2026 | ⚠ Reported (not re-extracted from OJ) | Press/law-firm summaries |
| SEC Private Fund Advisers Rules adopted August 2023 (88 Fed. Reg. 63206, 23 Aug 2023) | ✅ Verified | sec.gov; Fifth Circuit opinion |
| Rules vacated in full by the Fifth Circuit on 5 June 2024 (*National Association of Private Fund Managers v. SEC*, No. 23-60471) | ✅ Verified | ca5.uscourts.gov; sec.gov announcement |
| Post-vacatur litigation status (further SEC review) | ⚠ Reported only | Press coverage |
| VCC Act 2018; VCC framework launched 14 January 2020; MAS + ACRA administered | ✅ Verified | sso.agc.gov.sg (VCCA 2018); Clifford Chance briefing |
| Sections 13D/13O/13U fund tax incentives extended to 31 December 2029 | ✅ Verified | RSM Singapore (FDD Cir 10/2024, 1 Oct 2024); PwC |
| 13O revised criteria from 1 Jan 2025: min S$5M AUM in designated investments; ≥2 investment professionals; tiered local business spending | ✅ Verified | RSM Singapore |
| 13U: min S$50M AUM in designated investments at application and each FY-end; ≥3 investment professionals | ✅ Verified | RSM Singapore |
| 13OA scheme for Singapore limited partnerships, awards from 1 January 2025 | ✅ Verified | RSM Singapore (Budget 2024 announcements; DLA Piper on the 2024 amendments) |
| 13O/13U renumbering from former 13R/13X effective YA 2020 | ⚠ Reported (not re-extracted from primary legislation) | Secondary commentary; sibling guide |
| MAS fund-manager licensing routes (CMS/LFMC, RFMC ≤30 QIs, VCFM) | ⚠ Cross-referenced (parameters date-sensitive; RFMC sunset/2026 streamlined framework reported) | [MAS Regulations guide](mas_regulations_guidelines_guide.md); [Singapore Private Markets guide](singapore_private_markets_guide.md) §5.7; Raffles Corporate Services |
| Singapore asset-management AUM S$6.7T (end-2025, +10% YoY) | ✅ MAS-released (breakdowns not re-extracted) | MAS Singapore Asset Management Survey Report 2025 (MAS communications); sibling guide (flagged) |
| "2 and 20" fee convention (2% management fee, 20% carry) | ⚠ Convention; exact origin not pinnable to a single primary source | Industry standard (Investopedia et al.); no primary origin document |
| Deal-by-deal "American" vs whole-fund "European" waterfall terminology | ⚠ Industry usage, not a formal standard | LPA practice; industry literature |
| KKR founded 1976; first Asia offices Hong Kong & Tokyo 2005; KKR Asian Fund I 2007 vintage, closed US$3.98B | ✅ Verified (1976 founding cross-referenced, not re-extracted this pass) | asiaasset.com; PitchBook; institutional investor coverage |
| Hillhouse founded 2005 by Lei Zhang with US$20M Yale-endowment seed; US$18B Fund V (Aug 2021), largest Asia fundraise at the time | ✅ Verified (AUM estimate flagged) | Wikipedia→Reuters/FT/Bloomberg; firm site |
| BPEA founded 1997 (Barings affiliate), independent 2000; EQT acquisition US$7.5B closed Oct 2022; renamed BPEA EQT, then EQT Private Capital Asia (Jan 2024) | ✅ Verified | Wikipedia→firm/press records; EQT |
| Affinity Equity Partners: 1998 roots (UBS Capital Asia Pacific), independent 2004; ~US$14B AUM (2023) | ✅ Verified (founding-year nuance flagged) | Wikipedia→press |
| PAG: firm founded 2002 as Pacific Alliance Group; Weijian Shan executive chairman, PE business founded 2010 | ✅ Verified (task-brief simplification flagged) | pag.com (About); Wikipedia |
| CVC: founded 1981 (Citicorp Venture Capital), spinoff 1993; Asia funds since 2000 (Asia I US$750M); Asia VI US$6.8B (2023); ~€205B AUM (2025) | ✅ Verified (Asia office year flagged) | Wikipedia→FT/WSJ/Nasdaq; firm records |
| Subscription-line sizing convention 10–25% of commitments; borrowing-base and 90-day repayment conventions | ⚠ Market convention | Industry practice; no primary standard |
| Worked-example arithmetic (subscription-line IRR 26.0%→37.4%; LBO MOIC 3.89x / IRR ≈31.2%; waterfall LP 220 / GP 30) | ✅ Internally consistent | This guide's own arithmetic |

## 12. What Could Not Be Verified

Facts and claims that this pass could not pin to a primary source, flagged ⚠ in the text:

- **The precise origin of "2 and 20".** The convention is universally described as the industry standard, but no primary document establishes when the package became canonical. The most-cited ancestor (Alfred Winslow Jones's 20% profit share, 1949) is itself a secondary attribution. No fee, carry, or origin figure was fabricated; the convention is presented as convention.
- **The YA of the 13R/13X → 13O/13U renumbering.** Widely reported as effective from YA 2020 (announced in the 2019 Budget), but the consolidated Income Tax Act on sso.agc.gov.sg shows only current numbering, and this pass did not re-extract the amending Act. The sibling guide reports the successor sections taking effect in the 2023–2024 window (as reported) — the two accounts are not fully reconciled here.
- **AIFMD II transposition deadline (16 April 2026).** Widely reported; not re-extracted from the Official Journal in this pass (eur-lex blocked automated extraction).
- **SEC litigation posture after the Fifth Circuit vacatur (June 2024).** Whether the SEC sought further review is reported in press coverage; not re-verified from sec.gov docket records.
- **MAS fund-manager licensing parameters** (RFMC 30-qualified-investor limit, the commonly cited S$250M AUM cap, the 2026 streamlined framework consolidating RFMC into LFMC). Date-sensitive and evolving; cross-referenced to the sibling guides rather than re-derived. MAS primary pages returned 404s during this pass.
- **MAS AUM survey details** beyond the headline S$6.7T / +10% (e.g. the PE sub-allocation, fund-managed vs discretionary splits). The headline figure is MAS-released; the breakdowns were not re-extracted.
- **Subscription-line market statistics** (aggregate market size, average pricing, average utilisation). Only the mechanics and the 10–25%-of-commitments sizing convention are presented, flagged as market practice; the 2017-era SEC/ILPA guidance on subscription lines and net invested capital was referenced from memory of the regulatory debate and is flagged accordingly.
- **Several firm-level details:** Hillhouse's AUM (~US$100B, 2026 estimate); Affinity's founding-year framing (1998 team roots vs 2004 independence); PAG's "founded 2002 by Weijian Shan" simplification (firm 2002, PE business 2010 — pag.com verified); CVC's exact Asia-office opening year; KKR's 1976 founding (well-documented but not re-extracted from kkr.com this pass). No firm fact, year, or fund size was invented; everything not directly verified carries a ⚠ and a source note.
- **AIFMD II loan-retention mechanics in detail** (5% retention verified; the transitional-provisions arithmetic for pre-15-April-2024 AIFs was not re-extracted).

**Method note:** web_search degraded intermittently during this pass (several queries returned empty); verification was completed via direct extraction of ilpa.org, eur-lex (AIFMD), sso.agc.gov.sg, RSM Singapore, Wikipedia (with primary citations noted), pag.com, asiaasset.com, and the sibling repo guides, plus sec.gov/ca5.uscourts.gov via search-result metadata. The worked-example arithmetic (§10) is original to this guide and internally consistent by construction.

## 13. References and Further Reading

- **ILPA** — ilpa.org: Private Equity Principles (Sept 2009; 2.0, Jan 2011; 3.0, June 2019), Principles FAQ, Standardised Reporting Templates
- **AIFMD** — Directive 2011/61/EU of 8 June 2011, OJ L 174, 1 July 2011 (eur-lex.europa.eu/eli/dir/2011/61/oj); Wikipedia "Alternative Investment Fund Managers Directive 2011" (summary of Articles 26–30)
- **AIFMD II** — Directive (EU) 2024/927 of 13 March 2024 (eur-lex.europa.eu/eli/dir/2024/927/oj/eng); Linklaters Financial Regulation, "AIFMD II takes effect at last"; Lexology, "AIFMD II: The New EU Framework for Loan-Origination Funds"
- **SEC** — sec.gov: "Announcement Regarding the Private Fund Advisers Rules" (June 2024); 88 Fed. Reg. 63206 (23 Aug 2023); *National Association of Private Fund Managers v. SEC*, No. 23-60471 (5th Cir., 5 June 2024, ca5.uscourts.gov); Morgan Lewis and Morrison Foerster client alerts
- **Singapore** — sso.agc.gov.sg: Variable Capital Companies Act 2018; Income Tax Act 1947 (ss. 13D/13O/13U); RSM Singapore, "Changes to Fund Tax Incentive Schemes in Singapore" (Oct 2024, on MAS FDD Cir 10/2024); PwC Singapore tax bulletin (Oct 2024); MAS Singapore Asset Management Survey Report 2025; Clifford Chance briefing on the VCC (Jan 2020); DLA Piper on the 2024 fund-incentive amendments
- **Firms** — pag.com (About — Weijian Shan, PAG history); asiaasset.com ("A new Asia fund for KKR?" — HK/Tokyo 2005, Asian Fund I US$3.98B); PitchBook fund profiles; Wikipedia (Hillhouse Investment, Baring Private Equity Asia, Affinity Equity Partners, CVC Capital Partners, PAG) with primary citations to Reuters, FT, WSJ, Bloomberg, Nasdaq, PEI
- **Sibling repo guides** — [Singapore Private Markets](singapore_private_markets_guide.md), [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md), [Norges Bank Investment Management](norges_bank_investment_management_guide.md), [Partners Group](partners_group_company_guide.md), [Partners Group Competitors](partners_group_competitors_guide.md), [Alternatives Front-to-Back Operating Model](alternatives_front_to_back_operating_model.md), [Asset Management & Alternatives](asset_management_alternatives_guide.md), [Financial Fraud Detection at Scale](financial_fraud_detection_at_scale_guide.md)
- **General** — Investopedia ("Two and Twenty"); industry commentary on fee compression, subscription lines, and continuation funds (⚠ directional)

## 14. Glossary

| Term | Definition |
|---|---|
| **GP** | General partner — the fund manager; runs the fund, earns fees and carry |
| **LP** | Limited partner — the investor; liability limited to its commitment |
| **LPA** | Limited partnership agreement — the fund's constitutional document |
| **LPAC** | Limited partner advisory committee — the LPs' consultative governance organ |
| **ILPA** | Institutional Limited Partners Association — the LP trade body; author of the Private Equity Principles |
| **Carried interest (carry)** | The GP's share of profits (commonly 20%) above the LP entitlement |
| **Management fee** | The GP's running fee (commonly 2% p.a. on commitments during the investment period) |
| **Hurdle / preferred return** | The LP return (commonly 8% p.a.) that must be paid before carry |
| **Catch-up** | The mechanism by which the GP reaches its full carry share after the hurdle |
| **Waterfall** | The ordered distribution of fund proceeds between LPs and GP; deal-by-deal ("American") vs whole-fund ("European") |
| **Clawback** | The GP's obligation to return excess carry if final fund economics so require |
| **Commitment** | The capital an LP agrees to provide when called |
| **Capital call (drawdown)** | The GP's demand for committed capital |
| **Dry powder** | Committed but uncalled capital |
| **Vintage year** | The fund's first investment year; its performance cohort |
| **Investment period** | The years (commonly 3–5) during which the fund may make new investments |
| **LBO** | Leveraged buyout — control acquisition funded with debt and equity |
| **EBITDA** | Earnings before interest, tax, depreciation and amortisation — the LBO's cash-flow proxy |
| **Multiple (EV/EBITDA)** | Enterprise value divided by EBITDA; the LBO's pricing and leverage yardstick |
| **Debt stack** | The layers of acquisition debt — senior, mezzanine, unitranche, holdco |
| **Buy-and-build** | Platform + add-on acquisition strategy |
| **MOIC** | Multiple on invested capital — the timing-blind return multiple |
| **IRR** | Internal rate of return — the annualised, cash-flow-weighted return |
| **DPI** | Distributions to paid-in capital — realised returns |
| **TVPI** | Total value to paid-in capital — realised + unrealised (NAV) |
| **J-curve** | The early-negative, later-positive cash-flow/return profile of a PE fund |
| **Secondaries** | The market for existing fund interests or portfolios (LP-led and GP-led) |
| **Continuation fund** | A GP-led vehicle that rolls assets from a maturing fund into a new fund |
| **Subscription line / capital-call facility** | Bank credit secured on LPs' uncalled commitments |
| **NAV facility** | Bank credit secured on the fund's portfolio NAV |
| **GP financing** | Credit to the GP/partners, often to fund the GP commitment |
| **VCC** | Variable Capital Company — Singapore's corporate fund vehicle (Act 2018, live 14 Jan 2020) |
| **13O / 13U / 13OA / 13D** | Singapore Income Tax Act fund tax incentive sections |
| **AIFMD / AIFMD II** | EU Directive 2011/61/EU and its 2024 amendment (Directive (EU) 2024/927) |
| **RFMC / LFMC / VCFM** | MAS fund-manager licensing classes (registered / licensed / venture-capital fund manager) |
| **Cymbal Bank** | The bank persona of this research series — the PE sponsor's banking partner in §9–§10 |

---

*End of guide — Private Equity: The Patient Capital. The money is committed for a decade, the returns are earned in the boardroom, and the arithmetic — fees, carry, the waterfall, the leverage — is the discipline that keeps the patient capital.*
