# The Finance Concepts Case Study: The Concepts, the Canon, and the Worked Cases

> A deep-dive on the finance concepts that underwrite every capital-allocation decision — the time value of money, NPV and IRR, the cost of capital and CAPM, the Modigliani–Miller propositions, valuation, working capital, portfolio theory, options, and real options — each stated in its standard form and verified back to its canon source, then turned into a short worked case, and closed with a full Cymbal Bank platform-investment decision case.

**Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
**Context:** Professional Development / Banking & Finance Series
**Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
**Last Updated:** September 2026

> **What this guide is.** A concept-by-concept reference on the finance canon as it is actually used in a capital-allocation decision — the formulas in their standard forms, the assumptions behind them, the places where textbooks genuinely disagree, and a worked case per concept cluster. Part A is the concepts; Part B is the worked cases; Part C is the banking adaptations (a bank's cost of capital is not a normal firm's, and its "working capital" is a trade-finance book); Part D is a complete Cymbal Bank decision case in the case-study form.

> **How to use this guide.** Read §1–§2 for the machinery (the time value of money, and the NPV/IRR decision rules); §3–§4 for the cost of capital and capital structure (WACC, CAPM, the Modigliani–Miller propositions); §5–§6 for valuation and the working-capital/ratio toolkit; §7 for risk, return, and options; §8 for the worked cases; §9–§10 for the banking applications and the full Cymbal Bank case. The claims-audit table in §11 and the "What Could Not Be Verified" section in §12 are the verification layer — consult them before quoting a citation or a number.

> **What this guide is not.** It does not re-teach the case-study *method* — the Harvard Business School method, case anatomy, the issue→analysis→alternatives→recommendation discipline, and the teaching note are the subject of [management/management_case_study_guide.md](../management/management_case_study_guide.md), and this guide cross-references it rather than restating it. It does not re-derive the MBA finance core, which is summarised in [management/mba_body_of_knowledge_guide.md](../management/mba_body_of_knowledge_guide.md) §6, nor the CFA curriculum's Corporate Issuers and Portfolio Management topics, which are covered in [cfa_program_guide.md](cfa_program_guide.md) §3. It does not teach how to write an internal investment *business case* — a different artefact — which is [management/business_case_development_guide.md](../management/business_case_development_guide.md). It does not re-derive the banking-domain deep dives it leans on: [basel_regulatory_capital_guide.md](basel_regulatory_capital_guide.md), [treasury_alm_guide.md](treasury_alm_guide.md), [risk_management_models_guide.md](risk_management_models_guide.md), [trade_finance_guide.md](trade_finance_guide.md), [supply_chain_finance_guide.md](supply_chain_finance_guide.md), [capital_markets_architecture_guide.md](capital_markets_architecture_guide.md), [hedge_fund_guide.md](hedge_fund_guide.md), and [investment_portfolio_operations_guide.md](investment_portfolio_operations_guide.md). For the cost-modelling cousins it cross-refs [../technology/tco_modeling_guide.md](../technology/tco_modeling_guide.md) and [../technology/finops_guide.md](../technology/finops_guide.md) lightly.

> **A note on verification.** Claims are tagged ✅ (verified at a primary or authoritative source), ⚠ (flagged — contested, practitioner-sourced, or a convention that varies between texts), or ❌ (could not be verified). Finance is unusually prone to tidy attribution stories: one name gets attached to a model that several people built in the same decade, and one convention gets taught as "the" formula when the textbook next door uses another. Where the record is genuinely contested, this guide says so rather than choosing the tidy version. Every worked example is arithmetic — computed, not asserted — and every figure is invented.

---

## Table of Contents

1. [Time Value of Money](#1-time-value-of-money)
2. [Capital Budgeting: NPV, IRR, and the Incremental Cash-Flow Discipline](#2-capital-budgeting-npv-irr-and-the-incremental-cash-flow-discipline)
3. [The Cost of Capital: WACC, CAPM, and the Pricing of Risk](#3-the-cost-of-capital-wacc-capm-and-the-pricing-of-risk)
4. [Capital Structure: The Modigliani–Miller Propositions and After](#4-capital-structure-the-modiglianimiller-propositions-and-after)
5. [Valuation Methods](#5-valuation-methods)
6. [Working Capital and the Ratio Families](#6-working-capital-and-the-ratio-families)
7. [Risk, Return, and Optionality: Portfolio Theory, Options, and Real Options](#7-risk-return-and-optionality-portfolio-theory-options-and-real-options)
8. [The Worked Cases: Applying the Concepts](#8-the-worked-cases-applying-the-concepts)
9. [Banking Applications: Cost of Capital, RAROC/EVA, and Business-Line Valuation](#9-banking-applications-cost-of-capital-raroceva-and-business-line-valuation)
10. [The Cymbal Bank Worked Case: The Trade-Finance Platform Investment](#10-the-cymbal-bank-worked-case-the-trade-finance-platform-investment)
11. [Claims Audit](#11-claims-audit)
12. [What Could Not Be Verified](#12-what-could-not-be-verified)
13. [Glossary](#13-glossary)
14. [Sources and Further Reading](#14-sources-and-further-reading)

---

## 1. Time Value of Money

### 1.1 The Master Concept (standard)

Almost everything downstream in this guide is one idea wearing different clothes: **a dollar today is worth more than a dollar tomorrow, because today's dollar can be invested and earn a return.** Discounting reverses compounding. The discount rate is the price of time plus the price of risk. Get this wrong and every NPV, every valuation, and every cost-of-capital number inherits the error.

The concept dates to the earliest actuarial and compound-interest literature, but its modern *managerial* form — discounting a project's cash flows at an opportunity cost — is the twentieth-century capital-budgeting tradition. The MBA core states it plainly ([management/mba_body_of_knowledge_guide.md](../management/mba_body_of_knowledge_guide.md) §6); this section states the formulas precisely and flags where texts differ.

### 1.2 The Core Formulas (standard forms; ✅ textbook-standard — ⚠ convention variants flagged in §1.3)

| Concept | Standard form | Notes |
|---|---|---|
| Future value (single sum) | `FV = PV × (1 + r)^n` | Compounding at rate `r` for `n` periods |
| Present value (single sum) | `PV = FV / (1 + r)^n` | The discounting operation |
| Continuous compounding | `FV = PV × e^(r·n)` | Limit of `(1 + r/m)^(m·n)` as `m → ∞` |
| FV, **ordinary** annuity | `FVA = C × [ ((1 + r)^n − 1) / r ]` | Payments at **end** of each period |
| PV, **ordinary** annuity | `PVA = C × [ (1 − (1 + r)^−n) / r ]` | The annuity factor |
| FV / PV, **annuity due** | multiply the ordinary-annuity result by `(1 + r)` | Payments at **beginning** of each period |
| Level perpetuity | `PV = C / r` | `n → ∞`; requires `r > 0` |
| Growing perpetuity | `PV = C₁ / (r − g)` | First cash flow one period ahead; requires `r > g` |
| Gordon growth (perpetuity-with-growth) | `P₀ = D₁ / (r − g) = D₀(1 + g) / (r − g)` | Perpetuity-of-dividends special case; see §5 |

**Worked check (arithmetic).** Let `C = 100`, `r = 8%`, `n = 5`, `g = 3%`, `D₀ = 2.00`.

- Ordinary-annuity PV factor: `[1 − 1.08^−5] / 0.08 = 3.992710…` → `PVA = 399.2710`.
- Annuity-due PV factor: `3.992710 × 1.08 = 4.312127…` → `PV = 431.2127`.
- Growing perpetuity with `C₁ = D₀(1+g) = 2.06`: `2.06 / (0.08 − 0.03) = 41.20`.

**Zero sign convention matters.** A dollar paid *out* is negative, a dollar received *in* is positive, and cash-flow tables must hold one convention consistently or NPV will double-count.

### 1.3 Timing Conventions and the Flags (⚠)

Textbooks genuinely disagree on surface details, and a diligent reader should expect it:

- ⚠ **Annuity-due vs ordinary annuity.** The finance-standard treatment (ordinary annuity, payments in arrears) is dominant in the corporate-finance canon (Brealey–Myers–Allen, Ross–Westerfield–Jaffe); actuarial and lease texts more often lead with the annuity-due (payments in advance). The *math* is identical — the due version is the ordinary version scaled by `(1 + r)` — but the naming is not universal, and one must always ask *when* the first payment lands.
- ⚠ **Discounting conventions.** Some texts discount annual cash flows at the *end* of year `t` (the usual convention); project-finance and mid-year-convention textbooks discount at the *middle* of the year (`t − 0.5`), which raises NPV modestly and is defensible for smoothly-occurring flows. Neither is "wrong"; they are different assumptions about timing.
- ⚠ **Rate quotation.** An annual percentage rate (APR) is a nominal rate that ignores intra-year compounding; the effective annual rate (EAR) is `(1 + APR/m)^m − 1`. Stating "8%" without saying APR or EAR is an incomplete statement, and mortgage/consumer-finance disclosure regimes define them differently across jurisdictions.
- ⚠ **The growth-model numerator.** The Gordon growth model is written both as `D₁/(r − g)` (next period's dividend, the cleaner form) and as `D₀(1 + g)/(r − g)`. The two are algebraically identical, but the shorthand "the Gordon growth model" is also used loosely for the whole dividend-discount model, which is a broader object (§5.4).

### 1.4 Why It Anchors Everything

Every later section is this section applied to a different cash-flow pattern: NPV discounts an uneven project stream (§2); WACC supplies the rate (§3); DCF adds a terminal value built from a growing perpetuity (§5); the cash-conversion cycle is a working-capital timing problem (§6); and option pricing discounts a *probability-weighted* payoff at a risk-free rate (§7). If the reader takes one thing from Part A, it is that `(1 + r)^−t` is the unit of account for every financial decision in the rest of the guide.

---

## 2. Capital Budgeting: NPV, IRR, and the Incremental Cash-Flow Discipline

### 2.1 The Four Decision Rules (standard; ✅ in the corporate-finance canon)

| Rule | Definition | Decision rule | Status |
|---|---|---|---|
| **Net present value (NPV)** | `NPV = Σ_{t=1..n} CFₜ / (1 + r)ᵗ − I₀` (equivalently `Σ_{t=0..n}` with `CF₀ = −I₀`) | Accept if `NPV > 0` (mutually exclusive: pick highest NPV) | ✅ theoretically correct criterion |
| **Internal rate of return (IRR)** | The rate `r*` that sets `NPV = 0` | Accept if `r* >` cost of capital | ✅ definition; ⚠ decision use |
| **Payback period** | Years until cumulative cash flows recover `I₀` | Accept if payback ≤ a cutoff | ✅ definition; ⚠ ignores time value and cash flows after cutoff |
| **Discounted payback** | Years until cumulative *discounted* cash flows recover `I₀` | Accept if ≤ cutoff | ✅ definition; ⚠ still ignores flows after cutoff |
| **Profitability index (PI)** | `PI = PV(future cash flows) / I₀` | Accept if `PI > 1` | ✅ definition; useful under capital rationing |

### 2.2 Why NPV Is the Correct Criterion (✅)

NPV is the only one of the five that measures the change in the owners' wealth in money terms, and it does so consistently across scale, timing, and risk: it adds. Because it is expressed in currency, it aggregates across projects without a weighting convention; because it discounts at the opportunity cost of capital, it prices timing consistently; and because that rate reflects the project's risk, it prices risk consistently. The corporate-finance canon (Brealey–Myers–Allen, *Principles of Corporate Finance*, in its successive editions) states the NPV rule as the master criterion and treats IRR, payback, and PI as screens or communication tools. The MBA core reaches the same conclusion ([management/mba_body_of_knowledge_guide.md](../management/mba_body_of_knowledge_guide.md) §6.2). ✅

### 2.3 The Known Pitfalls of IRR (✅ standard; ⚠ where practitioner treatments differ)

IRR is the most-used and most-abused rule in practice. Its pathologies are standard curriculum content:

1. **Multiple IRRs with non-conventional cash flows.** When a project's cash flows change sign more than once (e.g. `−100, +250, −160`), the NPV function can cross zero more than once, so more than one IRR exists. **Descartes' rule of signs** (René Descartes, *La Géométrie*, 1637) bounds the number of positive real roots by the number of sign changes — directly relevant, since the IRR equation is a polynomial in `1/(1+r)`. ✅ (Note: the *rule* is 1637; its application to IRR is textbook-standard and the "Descartes" naming is routine. The claim that Descartes himself had finance in mind is obviously false.)
2. **The no-IRR case.** Some cash-flow patterns produce *no* real IRR — NPV never crosses zero for any admissible rate. The rule then says nothing, and the analyst must fall back on NPV.
3. **The reinvestment-rate assumption (⚠).** IRR implicitly assumes interim cash flows are reinvested *at the IRR itself*; NPV implicitly assumes reinvestment at the cost of capital. NPV's assumption is the more defensible one (the opportunity cost), which is the standard textbook reason to prefer NPV. Some practitioner texts present the *modified IRR (MIRR)* — which sets an explicit reinvestment and a financing rate — as the remedy; the treatment and naming of MIRR varies, so flag it ⚠.
4. **The scale problem.** IRR is a percentage and ignores size: a 60% return on a SGD 1,000 project beats a 20% return on a SGD 100 million project, even though the second creates vastly more value. NPV does not have this failure mode.
5. **The timing problem.** A long-duration project with a lower IRR can have a higher NPV than a short-duration project with a higher IRR.
6. **The mutual-exclusivity conflict.** For mutually exclusive projects, IRR and NPV can rank projects differently because of scale and timing. **When they conflict, follow NPV** — this is the unambiguous textbook rule (⚠ some practitioner scorecards still rank on IRR, which is exactly the error the canon warns against).
7. **Borrowing projects and sign flips.** For a project that is a net *borrowing* (initial inflow, later outflows), a *low* IRR is good — the sign convention inverts the rule. Using "higher IRR is better" mechanically fails here.

### 2.4 The Incremental Cash-Flow Discipline (✅; a different concept from the discount rate)

The discount rate gets the attention; the cash-flow definition causes the expensive mistakes. Capital budgeting is **incremental** — it asks what changes *because* of the decision:

| Item | Treatment | Why |
|---|---|---|
| **Sunk costs** | **Exclude** | Already spent; cannot be changed by the decision; including them is the classic error |
| **Opportunity costs** | **Include** | A resource used by the project forgoes its next-best use; the forgone value is a real cost |
| **Allocated overhead** | **Careful** | Include only the *incremental* overhead the project causes; pure allocation is a transfer, not a cash flow, unless it displaces something |
| **Working capital** | **Include the investment; release it at the end** | Inventory, receivables, and payables are cash tied up during the project's life and recovered at its end (§6) |
| **Capitalised interest / financing flows** | **Exclude from project cash flow** | Financing is captured in the discount rate (the separation principle, §2.5); double-counting it is an error |
| **Taxes** | **Include, on the incremental pre-tax cash flow** | Tax is a real cash flow; the depreciation *tax shield* is a separate benefit |
| **Depreciation** | **Not a cash flow, but a tax shield** | The shield is `Depreciation × tax rate`; add it back when computing cash flow via the tax route |
| **Cannibalisation / synergy** | **Include the net effect on existing business** | Inflows stolen from an existing product are a cost; incremental sales to new customers are not |
| **Terminal / salvage value** | **Include, net of tax on the gain** | Disposal proceeds and released working capital are terminal inflows |

**The depreciation tax shield.** If depreciation is `D`, the tax rate is `T`, and depreciation is tax-deductible, the shield is `D × T` per period. Its present value, for straight-line depreciation of an asset costing `I` over `n` years with no salvage, is `(I/n)·T × annuity-factor(r, n)`. Textbooks flag that depreciation *methods* (straight-line, declining balance, sum-of-years-digits, and the US MACRS system with its half-year convention) change the shield's timing and therefore its present value, even when total depreciation is equal. ✅

### 2.5 The Separation Principle (✅, via Modigliani–Miller)

The **separation principle** — the investment decision is made independently of how the investment is financed — is a consequence of the Modigliani–Miller irrelevance result when capital markets are perfect (§4.2). Its practical form: value the project's cash flows at a discount rate reflecting the project's risk, and keep the financing decision in the capital-structure and treasury conversation. Mixing them (e.g. subtracting interest payments *and* discounting at WACC) double-counts financing. The MBA core and the CFA Corporate Issuers topic both teach the separation in this form ([management/mba_body_of_knowledge_guide.md](../management/mba_body_of_knowledge_guide.md) §6; [cfa_program_guide.md](cfa_program_guide.md) §3).

### 2.6 The Capital-Rationing Coda (⚠)

When capital is *rationed* (a hard budget cap), NPV alone can be the wrong ranking if projects are divisible: the **profitability index** ranks value per unit of scarce capital and is the standard tool for the divisible case. For *indivisible* projects under a single constraint, the correct answer is a knapsack (integer-programming) problem, and the simple PI ranking can fail. ⚠ Most corporate-finance texts mention this; few corporate scorecards implement it. It matters most to banks, whose capital is a regulated scarce resource — which is why §9 returns to it.

---

## 3. The Cost of Capital: WACC, CAPM, and the Pricing of Risk

### 3.1 WACC — The Formula (standard; ✅ in the canon)

The weighted average cost of capital is the after-tax blended cost of the firm's permanent capital:

```
WACC = (E/V) × Re  +  (D/V) × Rd × (1 − Tc)  [+ (P/V) × Rp if preferred stock exists]
```

where `E` is the market value of equity, `D` the market value of debt, `P` preferred, `V = E + D + P`, `Re` the cost of equity, `Rd` the *pre-tax* cost of debt, and `Tc` the marginal corporate tax rate. The `(1 − Tc)` factor reflects that interest is tax-deductible — the **interest tax shield**.

The cost of equity is usually estimated with the CAPM (§3.2): `Re = Rf + β × (ERP)`, where `ERP = E(Rm) − Rf` is the market risk premium.

### 3.2 The Origin: the Modigliani–Miller Weighted-Average-Cost Proposition (✅)

WACC is not a convenience formula; it descends from Modigliani and Miller's 1958 paper (§4.2), which showed that *under their assumptions the weighted average cost of capital of a firm is independent of its capital structure* — the **weighted-average-cost-of-capital proposition**: `WACC = R₀ ≡ ρ`, the cost of capital of an unlevered firm in the same risk class. Any claim that "debt is cheap so more debt lowers WACC" is, in the no-tax world of the 1958 paper, an illusion: the equity holders demand `Re = R₀ + (D/E)(R₀ − Rd)` (MM Proposition II), which rises exactly to offset the cheaper debt. The 1963 correction restores the tax benefit (§4.3). ✅

### 3.3 The Beta Estimation Problem (✅; ⚠ conventions)

Beta is the CAPM's only firm-specific input and the noisiest:

- **Levered vs unlevered beta.** The observed (equity) beta reflects both business risk and financial leverage. The **Hamada relation** (Robert Hamada, 1972 — ✅ standard attribution; ⚠ page details not re-verified here) unlevers it: `β_L = β_U × [1 + (1 − Tc)(D/E)]`, with the inverse giving `β_U` (asset beta) from `β_L`. One re-levers the comparable's asset beta at the target's capital structure.
- **Estimation window and frequency.** Betas estimated on daily vs monthly returns, over 2 vs 5 years, and with or without adjustment (Bloomberg's and BARRA's adjusted betas shrink toward 1.0) differ materially. ⚠ There is no single "correct" window; the choice is a convention, and it should be disclosed.
- **Comparable-set selection.** Peer betas inherit the peers' business mix and leverage; the analyst's job is to net out the differences, and the residual is judgment, not arithmetic.

### 3.4 Weights: Book, Market, Target, Current (⚠ all four are conventions)

- ⚠ **Market vs book weights.** The canon prefers **market-value** weights because they reflect the opportunity cost of capital actually at risk. Banks and regulated utilities are sometimes valued on book weights because regulatory capital is defined on book values — a legitimate exception, but an exception nonetheless (§9).
- ⚠ **Target vs current weights.** A firm moving toward a target capital structure should weight at the **target** (management's stated or long-run policy); weighting at the current year's market values imports a temporary distortion. The two differ, sometimes a lot, and the choice changes WACC by tens of basis points.
- ⚠ **Rounding, and the "which V" question.** Whether to include operating leases, minority interests, or contingent obligations in `D` varies across houses and accounting regimes.

### 3.5 The Country-Risk-Premium Debate (⚠)

For valuations where cash flows are exposed to a country whose sovereign risk is not already in the risk-free rate, the standard treatment (associated with Aswath Damodaran's NYU Stern teaching and datasets) adds a **country risk premium (CRP)** to the mature-market equity risk premium:

```
CRP ≈ sovereign default spread × (σ_equity / σ_sovereign bond)
```

⚠ The method is standard in emerging-market valuation *practice*, but the "right" way to add it — to the discount rate for all cash flows, or only to cash flows genuinely exposed to the country, or via a country-adjusted beta — is debated; Damodaran's own approach has evolved across editions. Damodaran publishes free datasets and lecture notes at `pages.stern.nyu.edu/~adamodar` (the page was not retrievable at extraction time for this guide — see §12). ⚠ Treat any single CRP number as a convention, not a fact.

### 3.6 The Deliberate Frictions the Canon Ignores (⚠)

WACC assumes taxes, but not the frictions that make it a *lower bound* on the true hurdle: issuance costs, financial-distress costs, agency costs, and the option value of waiting (§7.4). A project that clears a WACC hurdle can still destroy value once real options and distress costs are priced. The canon says this; practice often forgets.

---

## 4. Capital Structure: The Modigliani–Miller Propositions and After

### 4.1 The Two Questions

Capital structure asks: (i) does the *mix* of debt and equity change the firm's value? and (ii) if so, how do we choose the mix? The answers run through two papers, two corrections, and three surviving theories.

### 4.2 Modigliani–Miller 1958 — Irrelevance Without Taxes (✅)

Franco Modigliani and Merton Miller, "The Cost of Capital, Corporation Finance and the Investment," *American Economic Review* 48(3): 261–297 (1958). Under the assumptions of perfect capital markets (no taxes, no transaction costs, no bankruptcy costs, symmetric information, fixed investment policy), the value of the firm is **independent of its capital structure**: `V_L = V_U`. Two propositions:

- **MM Proposition I (no taxes):** `V_L = V_U`. Value is set by the assets' cash flows and their risk, not by the slicing of the claims.
- **MM Proposition II (no taxes):** `Re = R₀ + (D/E)(R₀ − Rd)`. The cost of equity rises linearly with leverage, exactly offsetting the cheaper debt, so WACC stays at `R₀`.

The mechanism is arbitrage: if a levered firm were cheap, an investor could replicate the leverage personally (homemade leverage) and pocket a riskless profit, so prices must settle at irrelevance. The 1990 Nobel summary credits Miller's capital-structure contribution alongside Markowitz's portfolio theory and Sharpe's CAPM (✅, NobelPrize.org, 1990). Miller's own prize share is 1/3; Modigliani had received his in 1985 (✅, NobelPrize.org, 1985).

### 4.3 Modigliani–Miller 1963 — The Interest Tax Shield (✅)

Franco Modigliani and Merton Miller, "Corporate Income Taxes and the Cost of Capital: A Correction," *American Economic Review* 53(3): 433–443 (1963). Once corporate interest deductibility is admitted, the irrelevance result *breaks*: debt adds value through the **interest tax shield**:

```
V_L = V_U + Tc × D
```

Equivalently (MM Proposition II *with* taxes): `Re = R₀ + (1 − Tc)(D/E)(R₀ − Rd)`. Because the shield is public and formulaic, the tax-corrected model implies a corner solution — *all* debt — since value rises monotonically in `D` at `Tc` per dollar. Real firms do not sit at the corner, which is the empirical starting point for the trade-off theory (§4.4). ✅

### 4.4 The Trade-off Theory (✅ standard; ⚠ contested against the evidence)

The **trade-off theory** says the optimal capital structure balances the marginal tax benefit of debt (`Tc` per marginal dollar) against the marginal **financial-distress costs** — direct costs (legal, advisory) plus the larger indirect costs (lost customers, lost employees, foregone investment, supplier terms). As `D` rises, the present value of expected distress costs rises convexly, so a firm chooses the debt level where the two marginals cross. This explains why profitable, asset-heavy, low-volatility firms carry more debt. ⚠ The trade-off theory fits the cross-section only loosely; it struggles to explain why the most profitable firms are often the least levered — the fact that most embarrassed the theory and motivated the pecking order. ✅ standard statement of the theory; ⚠ the theory-versus-evidence record is genuinely contested across the empirical literature.

### 4.5 The Pecking-Order Theory (✅ Myers 1984; ⚠ contested as a general explanation)

Stephen A. Myers, "The Capital Structure Puzzle," *Journal of Finance* 39(3) (1984) — ✅ the paper is canonical and the citation is standard; ⚠ the exact page range was not re-verified at source for this guide (§12). The pecking order abandons an optimising target and instead follows an information-asymmetry ordering: firms finance with **internal funds first, then debt, then equity last**, because management issues securities only when it believes they are overpriced, and investors rationally read an equity issue as bad news (the adverse-selection logic of Myers and Majluf, 1984, "Corporate Financing and Investment Decisions When Firms Have Information That Investors Do Not Have," *Journal of Financial Economics* 13(2): 187–221 — ✅ standard citation). The pecking order explains the profitability–leverage inverse relationship and the reluctance to issue equity, but ⚠ it makes few sharp predictions and its empirical support is mixed — some tests support it, some do not.

### 4.6 Miller 1977, Signalling, and Market Timing (⚠)

- **Miller 1977.** Merton H. Miller, "Debt and Taxes," *Journal of Finance* 32(2): 261–275 — ✅ standard citation. Miller showed that once *personal* taxes are added (interest taxed at the personal rate, equity returns taxed more lightly or deferred), the net tax advantage of debt is smaller than the corporate-rate model implies — in the extreme "Miller equilibrium," the advantage can vanish. ✅ concept; ⚠ the empirical magnitude of the personal-tax offset is contested.
- **Signalling (Ross 1977).** Stephen Ross, "The Determination of Financial Structure: The Incentive-Signalling Approach," *Bell Journal of Economics* 8(1): 23–40 — ✅ standard citation. Debt signals management's confidence (it hardens the commitment to service claims); leverage is a costly signal.
- **Market timing (Baker & Wurgler 2002).** Firms issue equity when their stock is high relative to book and debt when low, so observed capital structure partly reflects the cumulative history of past market windows — ⚠ a "theory" of the *process*, not of an optimum.

### 4.7 The Honest Scorecard (⚠)

The canon offers three families — trade-off, pecking order, signalling — and no single one explains the observed cross-section. The intellectually honest statement is that capital structure is a *response to frictions*, and which friction dominates is firm- and market-specific. ⚠ Any confident claim of "the optimal debt ratio" should be treated as a modelling artefact until tested against the firm's own distress economics. This is exactly the kind of contested claim the case-study discipline is built to surface (see the analysis discipline in [management/management_case_study_guide.md](../management/management_case_study_guide.md) §7).

### 4.8 The Cymbal Bank Angle (⚠)

Banks invert the problem. For a non-financial firm, debt is a financing choice with a tax advantage; for a bank, deposits and wholesale borrowings are *raw material* — the balance sheet is the production function — so "leverage" is not a capital-structure decision but a regulatory and business necessity. A bank does not choose between debt and equity to minimise WACC; it is *required* to hold a minimum of equity against risk-weighted assets, and the binding constraint is the **cost of that equity** versus the return it can generate. §9 develops this. The Modigliani–Miller framework still applies, but the interesting variable becomes the *equity* slice and its return (ROE/RAROC), not the debt slice.

---

## 5. Valuation Methods

### 5.1 The DCF Machinery (standard; ✅ in the canon)

Discounted cash flow values an asset as the sum of its expected future cash flows discounted at a rate reflecting risk. The two routes differ in *whose* cash flow they discount:

- **Free cash flow to the firm (FCFF)** — cash available to all capital providers:
  `FCFF = EBIT × (1 − T) + Depreciation & amortisation − Capital expenditure − ΔNWC`.
  Discounted at **WACC** → **enterprise value**.
- **Free cash flow to equity (FCFE)** — cash available to equity after debt service:
  `FCFE = Net income + D&A − Capex − ΔNWC + Net borrowing`.
  Discounted at the **cost of equity** → **equity value** directly.

The two are consistent if the debt policy assumed in FCFE matches the weights in WACC. ⚠ Mixing an FCFF cash flow with a cost-of-equity rate (a frequent spreadsheet error) produces a number that means nothing.

### 5.2 Terminal Value: Two Conventions (⚠)

An explicit forecast rarely exceeds ten years, so most of the value sits in the terminal value. Two standard conventions:

```
Gordon growth (perpetuity-with-growth):   TV_n = FCF_{n+1} / (WACC − g) = FCF_n × (1 + g) / (WACC − g)
Exit multiple:                            TV_n = EBITDA_n × (selected multiple)
```

⚠ The choice is a convention, and the two can differ by a wide margin. The Gordon form is sensitive to `g` and to the spread `(WACC − g)` — a small change in either moves the value enormously. The exit-multiple form imports the market's current pricing and sidesteps the `g` debate but anchors on a comparables set that may be unrepresentative. **The terminal value usually dominates**, which is why a DCF debate is really a debate about the terminal assumption — a point the MBA core states explicitly ([management/mba_body_of_knowledge_guide.md](../management/mba_body_of_knowledge_guide.md) §6.6).

**Worked check (arithmetic).** `FCF₀ = 100`, `g = 3%`, `WACC = 9%`, five-year horizon:
`TV₅ = 100 × 1.03 / (0.09 − 0.03) = 103 / 0.06 = 1,716.67`; discounted five years: `1,716.67 / 1.09⁵ = 1,115.72`. Note how large the terminal component is relative to any plausible explicit-period flows.

### 5.3 The Enterprise-to-Equity Bridge (standard)

```
Equity value = Enterprise value − Net debt  (+ non-operating assets − minority interest − preferred ...)
Net debt     = Total debt − Cash and cash equivalents
```

⚠ The bridge is where four-fifths of valuation errors live: whether to net operating cash, lease liabilities (post-IFRS 16), pension deficits, or associates differs by house and by accounting regime, and each inclusion changes the equity number. For a bank the bridge is *inverted*: debt is funding, not a claim to net off, so equity value is usually more directly obtained from FCFE or from the dividend-discount model (§5.4, §9).

### 5.4 The Dividend-Discount Model and the Gordon Growth Attribution (✅ verified)

The **dividend-discount model (DDM)** values equity as the present value of expected dividends: `P₀ = Σ_{t≥1} Dₜ / (1 + r)ᵗ`. When dividends grow at a constant rate `g`, this collapses to the **perpetuity-with-growth** form:

```
Gordon growth model:   P₀ = D₁ / (r − g) = D₀ × (1 + g) / (r − g),   r > g
```

**Attribution (✅ verified).** The constant-growth result traces to **Myron J. Gordon and Eli Shapiro, "Capital Equipment Analysis: The Required Rate of Profit," *Management Science*, October 1956, 3(1): 102–110** — the citation appears in Modigliani and Miller's own 1958 reference list, which is where this guide confirmed it. ✅ ⚠ **Naming flag:** the formula is universally called the *"Gordon growth model"* in the textbook canon, though its original framing was a *required-rate-of-profit* result, and "Gordon growth model" is often used loosely to mean the whole DDM. ⚠ The textbook's `D₁` (next period) vs `D₀(1 + g)` (last period compounded) forms are algebraically identical but reported inconsistently; always check which dividend the source used.

### 5.5 Comparable-Company Multiples (⚠ convention-driven)

Multiples value a firm relative to a peer set. The common set:

| Multiple | Numerator / denominator | Typical use |
|---|---|---|
| **EV/EBITDA** | Enterprise value / EBITDA | Capital-structure-neutral; the workhorse for control valuations |
| **EV/EBIT** | Enterprise value / EBIT | When depreciation differs across peers |
| **P/E** | Market capitalisation / net income (or price / EPS) | Equity value; sensitive to leverage and accounting |
| **P/B** (P/BV) | Market capitalisation / book equity | Banks and financials (where book ≈ economic capital, §9) |
| **P/S**, **EV/Sales** | Market cap or EV / revenue | Early-stage or margin-insensitive comparison |

⚠ **Multiple selection and definition are conventions, not facts.** Houses differ on whether EBITDA is adjusted (one-off items added back), on the treatment of leases, on forward vs trailing periods, and on whether the median or the mean is the anchor. "The peer multiple" is not a number; it is a range plus a justification. The controlling-valuation literature treats the method as a *cross-check* on DCF, not a substitute for it (a view the case discipline would insist on). ✅ method; ⚠ definitional variance.

### 5.6 Precedent Transactions (⚠)

**Precedent-transaction analysis** applies multiples paid in *actual control deals* rather than trading levels. Because control transactions embed a control premium (frequently 20–40%), precedent multiples usually sit above trading multiples. ⚠ The method is standard and widely used in M&A, but the empirical dispersion is wide, the samples are small and non-contemporaneous, and the control premium itself varies with the cycle.

### 5.7 Residual Income and EVA (cross-ref)

Two equity-side relatives of DCF that share its logic:

- **Residual income (RI):** `RI_t = Net income_t − (Cost of equity × Beginning book equity_{t-1})`. Value = `Book equity₀ + Σ PV(RI_t)`.
- **Economic value added (EVA):** `EVA = NOPAT − WACC × Capital employed`. Value = `Invested capital + Σ PV(EVA_t)`.

EVA is the Stern Stewart–popularised label (⚠ attributed to a consultancy — see §11); the underlying concept is the same residual-income logic. Cross-ref §9, where RAROC is the banking-flavoured version of exactly this idea.

### 5.8 Damodaran's Public Resources (✅ the site exists; ⚠ not retrievable here)

Aswath Damodaran (NYU Stern) publishes, free, an unusually complete valuation dataset: equity risk premiums, country risk premiums, industry betas and multiples, and full lecture notes and spreadsheets, alongside his valuation texts (the *Damodaran on Valuation* line and the *Investment Valuation* text). The resources at `pages.stern.nyu.edu/~adamodar` are a de facto standard reference in the field. ✅ the resource is real and well known; ⚠ **it could not be retrieved at extraction time for this guide** (see §12), so no specific figure from it is quoted here.

---

## 6. Working Capital and the Ratio Families

### 6.1 The Operating Cycle and the Cash-Conversion Cycle (standard; ✅ textbook-standard)

```
Operating cycle        = DIO + DSO
Cash-conversion cycle  = DIO + DSO − DPO
```

where, using *average* balances:

| Metric | Standard form | Reads as |
|---|---|---|
| **DIO** (days inventory outstanding) | `Avg inventory / COGS × 365` | Days inventory sits before sale |
| **DSO** (days sales outstanding) | `Avg receivables / Revenue × 365` | Days before customers pay |
| **DPO** (days payables outstanding) | `Avg payables / COGS × 365` | Days before the firm pays suppliers |

`NWC` (net working capital) = current assets − current liabilities; **operating** NWC excludes cash and short-term borrowings, because the question of interest is the *operating* cash cycle, not the financing mix.

**Worked check (arithmetic).** Revenue 36,500; COGS 25,550; average inventory 4,200; average receivables 6,000; average payables 3,500:
`DIO = 4,200/25,550 × 365 = 60.0 d`; `DSO = 6,000/36,500 × 365 = 60.0 d`; `DPO = 3,500/25,550 × 365 = 50.0 d`.
So `operating cycle = 120.0 d` and `CCC = 70.0 d`. A shorter CCC frees cash — the whole point of §6.3 and of §9.3's trade-finance treatment.

### 6.2 The Definitional Variants (⚠ — the flags that matter most here)

The formulas above are the standard ones, but each has a live convention variant, and the variants can change the answer materially:

- ⚠ **DSO basis: revenue vs credit sales.** Using total revenue understates DSO for a firm with cash sales; using credit sales overstates it if cash sales are ignored. The worked example shows the swing: on an 80%-credit-sales basis, the same 6,000 of receivables gives `DSO = 6,000/(36,500 × 0.8) × 365 = 75.0 d` instead of 60.0 d.
- ⚠ **DIO basis: COGS vs revenue.** The COGS basis is the canon; some houses use revenue, which shrinks DIO (here `4,200/36,500 × 365 = 42.0 d` instead of 60.0 d).
- ⚠ **DPO denominator: COGS vs purchases.** COGS is a proxy for purchases; if purchases are known, they are the more accurate denominator. Payables related to capex should be excluded from operating DPO.
- ⚠ **Average vs closing balances.** Analysts differ; averages smooth seasonality (the right choice for a seasonal book), closing balances are simpler.
- ⚠ **365 vs 360 days.** Banking convention often uses 360 days for money-market products; corporate analysis usually uses 365. State it.

### 6.3 Funding the Cycle (standard; ✅)

The CCC defines a **funding requirement**: each day of CCC must be financed, either by internal cash or by borrowing, and the cost of that carry is `(CCC/365) × revenue (or COGS) × funding rate`. Shortening DIO or DSO, or extending DPO, releases cash. Supplier terms are financing: accepting 60-day terms instead of 30 releases a month of payables, but may carry a hidden price if the supplier offers a discount for early payment. This is the corporate side of the same economics that trade finance sells as a product (§9.3).

### 6.4 The Ratio Families (standard; ⚠ definitional variance)

| Family | Ratios | Notes |
|---|---|---|
| **Liquidity** | Current ratio = CA/CL; Quick ratio = (CA − inventory)/CL; Cash ratio = (cash + marketable securities)/CL | Rule-of-thumb thresholds (e.g. current ≥ 2) are conventions, not laws; industry matters |
| **Leverage** | D/E = total debt/equity; Net debt/EBITDA; Interest coverage = EBIT/interest | ⚠ Gross vs net debt, book vs market equity, EBIT vs EBITDA coverage |
| **Efficiency / activity** | Inventory turnover = COGS/avg inventory; Receivables turnover = revenue/avg AR; Asset turnover = revenue/avg assets | The reciprocals × 365 give DIO/DSO |
| **Profitability** | Gross margin; Operating margin = EBIT/revenue; Net margin = NI/revenue; ROA = NI/avg assets; ROE = NI/avg equity | ⚠ Net income vs NOPAT variants |
| **Market** | EPS = NI/shares; P/E = price/EPS; Dividend yield = DPS/price; Payout ratio = DPS/EPS | ⚠ Basic vs diluted EPS; trailing vs forward |

### 6.5 The DuPont Decomposition (✅ origin: DuPont; ⚠ attribution depth)

The three-step **DuPont identity** splits ROE into drivers:

```
ROE = Net margin × Asset turnover × Equity multiplier
    = (NI / Revenue) × (Revenue / Total assets) × (Total assets / Equity)
```

Worked check: `NI = 3,650`, `Revenue = 36,500`, `Assets = 30,000`, `Equity = 15,000`:
`ROE = (3,650/36,500) × (36,500/30,000) × (30,000/15,000) = 0.10 × 1.2167 × 2.00 = 24.33%`. ✅ arithmetic; the product equals `3,650/15,000` exactly.

**Origin (⚠).** The decomposition is attributed to the **DuPont Corporation** (E.I. du Pont de Nemours) in the early twentieth century — commonly credited to the executive **Donaldson Brown** around 1919 — and the attribution is standard in the literature. ⚠ The depth of the attribution is contested at the margin: the *technique* is certain, the exact authorship and date are reported with variance, and the five-step extension (adding tax and interest burdens) has no single canonical attribution. Treat "DuPont: Donaldson Brown, c. 1919" as the textbook line, not a documented primary-source fact.

### 6.6 Common-Size and Trend Analysis (standard)

- **Common-size statements** express each line as a percentage of a base — revenue for the income statement, total assets for the balance sheet — so firms of different sizes, and the same firm across years, compare directly.
- **Trend (index) analysis** rebases a base year to 100 and follows the index, exposing drift a single-year snapshot hides.
Both are descriptive techniques, not decision rules; they supply the "what changed" that the case discipline then interrogates for "why" ([management/management_case_study_guide.md](../management/management_case_study_guide.md) §7).

---

## 7. Risk, Return, and Optionality: Portfolio Theory, Options, and Real Options

### 7.1 Markowitz and Mean-Variance Optimisation (✅)

**Harry M. Markowitz, "Portfolio Selection," *Journal of Finance* 7(1): 77–91 (1952)** — the founding paper of modern portfolio theory. Its move was to treat *risk* as the variance of returns and *return* as the expected return, so a portfolio becomes a mean–variance problem: for a given expected return, minimise variance; for a given variance, maximise expected return. The set of such portfolios is the **efficient frontier**. ⚠ The exact page range is the standard citation; it was not re-verified at a source for this guide (JSTOR was not retrievable — §12), though the paper's existence, year, and journal are beyond doubt and Markowitz shared the 1990 Nobel "for their pioneering work in the theory of financial economics" (✅ NobelPrize.org, 1990).

### 7.2 The Portfolio Variance and Diversification (standard; ✅)

For a two-asset portfolio with weights `w₁, w₂`, standard deviations `σ₁, σ₂`, and correlation `ρ`:

```
σ_p² = w₁²σ₁² + w₂²σ₂² + 2·w₁·w₂·ρ·σ₁·σ₂        σ_p² = ΣᵢΣⱼ wᵢwⱼσᵢⱼ (general form)
```

**Worked check.** `w₁ = 0.6`, `σ₁ = 20%`; `w₂ = 0.4`, `σ₂ = 12%`; `ρ = 0.3`:
`σ_p² = 0.36×0.04 + 0.16×0.0144 + 2×0.6×0.4×0.3×0.2×0.12 = 0.0144 + 0.002304 + 0.003456 = 0.02016` → `σ_p = 14.20%`. Compare the perfectly-correlated case (`ρ = 1`): `0.6×0.20 + 0.4×0.12 = 16.80%`. The 2.6-point gap is the **diversification benefit**: combining imperfectly correlated assets lowers portfolio risk below the weighted average. ✅

### 7.3 Beta, the Security Market Line, and Systematic Risk (✅)

Diversification leaves **systematic** (market) risk, which cannot be diversified away, and idiosyncratic risk, which can. Beta measures the systematic exposure: `βᵢ = Cov(Rᵢ, R_m) / Var(R_m)`. The **security market line** is the CAPM (§3.2): `E(Rᵢ) = Rf + βᵢ × ERP`. This is the bridge from portfolio theory (§7) to the cost of equity (§3) — the same machinery, read from the other end.

### 7.4 The Critiques (⚠ all live debates)

- ⚠ **Estimation error and "error maximisation."** Mean-variance optimisation takes the sample means, variances, and covariances as if they were known; they are noisy estimates, and the optimiser maximises the *impact of the errors*, loading on assets with overstated expected returns. Richard Michaud's critique (Michaud, 1989, "The Markowitz Optimization Enigma: Is 'Optimized' Optimal?", *Financial Analysts Journal* — ✅ standard reference; ⚠ page details not re-verified) is the canonical statement.
- ⚠ **Roll's critique (1977).** Richard Roll, "A critique of the asset pricing theory's tests," *Journal of Financial Economics* 4(2): 129–176 — ✅ standard citation — argues that testing the CAPM requires the true market portfolio, which is unobservable, so CAPM tests are joint tests of the model *and* of the proxy. This is the strongest theoretical objection to the model's empirics.
- ⚠ **Fama–French and the factor zoo.** Eugene Fama and Kenneth French (1992, 1993 and after) showed that size and book-to-market (and later profitability and investment) explain the cross-section of returns better than beta alone, spawning multi-factor models. The debate over whether these are risk factors or mispricings is unresolved.
- ⚠ **Fat tails and non-normality.** Mean-variance assumes (or at least leans on) elliptical return distributions; realised returns have fatter tails and skew, which matters most in a crisis — the regime that motivates the whole exercise.

### 7.5 Options and the Black–Scholes–Merton Model (✅)

**Fischer Black and Myron Scholes, "The Pricing of Options and Corporate Liabilities," *Journal of Political Economy* 81(3): 637–654 (1973)**, and **Robert C. Merton, "Theory of Rational Option Pricing," *Bell Journal of Economics and Management Science* 4(1): 141–183 (1973)**. The joint result — the **Black–Scholes–Merton (BSM) model** — values an option by constructing a continuously rebalanced replicating portfolio whose payoff matches the option, so the option's price follows from no-arbitrage. **Attribution flags (⚠):** the model is jointly named because Merton extended and generalised it and coined the term "Black–Scholes"; the **1997 Nobel Memorial Prize went to Scholes and Merton, not Black**, who had died in 1995 (the Nobel is not awarded posthumously) — ✅ NobelPrize.org, 1997, "for a new method to determine the value of derivatives." Naming it the "Black–Scholes–Merton model" is the honest shorthand; naming it "Black–Scholes" only, the market's habit, under-credits Merton (and the joint naming can itself be read as under-crediting several other contributors).

### 7.6 The BSM Formula and Its Inputs (standard; ✅)

```
Call:  C = S·N(d₁) − K·e^(−rT)·N(d₂)
Put :  P = K·e^(−rT)·N(−d₂) − S·N(−d₁)

d₁ = [ ln(S/K) + (r + σ²/2)·T ] / (σ·√T)        d₂ = d₁ − σ·√T
```

Inputs: `S` spot price, `K` strike, `r` risk-free rate (continuously compounded), `σ` volatility, `T` time to expiry, `N(·)` the standard-normal CDF.

**Worked check.** `S = K = 100`, `r = 3%`, `σ = 25%`, `T = 1`:
`d₁ = 0.2450`, `d₂ = −0.0050`, `N(d₁) = 0.5968`, `N(d₂) = 0.4980`.
`C = 100×0.5968 − 100×e^(−0.03)×0.4980 = 11.35`; `P = 8.39`.
Parity check: `C − P = 2.9554 = S − K·e^(−rT) = 100 − 97.0446 = 2.9554`. ✅ The parity identity holds to four decimals — a reader can verify the model against put–call parity (§7.8) without any market data.

### 7.7 The Binomial Model (✅ Cox–Ross–Rubinstein 1979)

**John C. Cox, Stephen A. Ross, and Mark Rubinstein, "Option Pricing: A Simplified Approach," *Journal of Financial Economics* 7(3): 229–263 (1979)** — the **CRR binomial model**, which builds the option value by discrete up/down branches and converges to BSM as the number of steps grows. ✅ standard citation; ⚠ page details not re-verified at source here. Its practical value is that it prices **American** options and accommodates dividends and early exercise, which the base BSM does not.

### 7.8 Put–Call Parity and the Greeks (standard; ✅)

**Put–call parity** (European options, non-dividend-paying stock): `C − P = S − K·e^(−rT)`. It is model-free — an arbitrage identity — which is why §7.6 uses it as an arithmetic check.

The **Greeks** are the sensitivities: **delta** `∂C/∂S` (call delta `= N(d₁)`; put delta `= N(d₁) − 1`; the worked example gives 0.5968 and −0.4032), **gamma** `∂²C/∂S²`, **vega** `∂C/∂σ`, **theta** `∂C/∂t`, **rho** `∂C/∂r`. ✅ standard definitions; the convention on theta's sign (per day vs per year, passed vs remaining) varies ⚠.

### 7.9 The Assumptions and the Volatility Smile (⚠)

The base BSM assumes: geometric Brownian motion (lognormal prices) with **constant volatility and drift**, a constant risk-free rate, **no dividends** (in the base case), continuous frictionless trading with no transaction costs, unlimited short-selling borrowing, and continuous hedging. Each is false to some degree. The most instructive failure is the **volatility smile / skew**: if the model were exactly right, the volatility implied by market option prices would be the same across strikes and maturities. In practice implied volatility varies systematically with strike (a "smile" in currencies, a downward "skew" in equity indices post-1987), which is the market telling the model its lognormal, constant-volatility assumption is wrong. ⚠ This is a genuine, widely documented empirical contradiction *drawn from options theory itself* — a good example of a framework whose assumptions are false yet whose outputs remain useful as a quote convention.

### 7.10 Real Options (⚠ — widely taught, less widely practised)

**Real options** extend option logic to *physical and managerial* decisions: the option to **defer** an investment, to **expand**, to **abandon**, or to **switch** inputs/outputs. The framing:

```
Strategic NPV = Static (DCF) NPV + Value of the embedded real options
```

Canonical references: Lenos Trigeorgis, *Real Options* (MIT Press, 1996), and Tom Copeland and Vladimir Antikarov, *Real Options: A Practitioner's Guide* (2001) — ✅ both are standard texts; ⚠ page/edition details not re-verified here. The logic is sound and the pedagogy is now standard in corporate-finance courses.

⚠ **The honest flag:** real options are far more widely *taught* than reliably *practised*. Estimating the volatility of a non-traded project cash flow, specifying the option's strike and exercise terms, and avoiding double-counting the flexibility already embedded in the DCF assumptions are all hard, and the valuation can be sensitive to arbitrary choices. Many practitioners use real-options thinking qualitatively — as a reason to stage investment and preserve the ability to stop — without producing a defensible option value. Treat any single "real options value" as a modelling choice, not a measurement.

---

## 8. The Worked Cases: Applying the Concepts

Each case below is **illustrative — every figure is invented** and arithmetically consistent, computed rather than asserted. Each follows the case-analysis discipline condensed to **Issue → Analysis → Alternatives → Recommendation**; for the full discipline (the issue tree, the alternatives framing, the recommendation with implementation, the teaching note) see [management/management_case_study_guide.md](../management/management_case_study_guide.md) §7, which this guide deliberately does not restate.

### 8.1 Time Value of Money: Pay Now or in Instalments? *(illustrative — all figures invented)*

**Issue.** A vendor offers a Cymbal Bank infrastructure purchase on two terms: **SGD 100,000** now, or **SGD 22,000 a year for five years**. The bank's opportunity cost of funds is **8%**. Which term is cheaper in present-value terms?

**Analysis.** The instalment stream is an annuity. Ordinary (payments in arrears): `PV = 22,000 × [1 − 1.08⁻⁵]/0.08 = 22,000 × 3.992710 = 87,839.62`. If the first payment is due immediately (annuity due): `PV = 87,839.62 × 1.08 = 94,866.79`. Both are below 100,000.

**Alternatives.** (A) Pay now: cost 100,000. (B) Instalments in arrears: cost 87,839.62. (C) Instalments in advance: cost 94,866.79.

**Recommendation.** Take the instalments, but **check the timing clause first** — it is worth SGD 7,027.17 of present value to the counterparty. On arrears terms the saving is `100,000 − 87,839.62 = 12,160.38`; on advance terms it is only `100,000 − 94,866.79 = 5,133.21`. This is the annuity-due/ordinary distinction of §1.3 with money attached: the convention is not academic.

### 8.2 Capital Budgeting: When IRR and NPV Disagree *(illustrative — all figures invented)*

**Issue.** Two mutually exclusive projects, discount rate **10%**: **Project A** (`−1,000` then `+1,400`) and **Project B** (`−10,000` then `+12,500`). A second question: a project with cash flows `(−100, +250, −150)` is proposed. Which do you fund, and what does the second project's IRR tell you?

**Analysis.** A: `NPV = 272.73`, `IRR = 40.0%`. B: `NPV = 1,363.64`, `IRR = 25.0%`. **IRR ranks A first; NPV ranks B first.** Because B creates five times the value, the correct answer is **B** — NPV is the criterion (§2.2); IRR is a screen. The second project has sign changes `−,+,−`, so it can have multiple IRRs: solving `−100 + 250/(1+r) − 150/(1+r)² = 0` gives **IRR = 0% and IRR = 50%**, and `NPV > 0` for every rate strictly between 0% and 50%. An IRR rule of "accept if IRR > 10%" would accept it on the 50% root and reject it on the 0% root — the rule is undefined here.

**Alternatives.** (A) Fund A on its higher IRR. (B) Fund B on its higher NPV. (C) Fund the `(−100, +250, −150)` project on its "50% IRR".

**Recommendation.** Fund **B**. Reject the third project on a *conditional* basis — not because IRR says so, but because its NPV is only `+3.31` at 10% (thin margin), it has non-conventional cash flows, and it is a borrowing-shaped exposure whose IRR sign convention inverts (§2.3). The transferable lesson: *when IRR and NPV conflict, NPV wins; when IRR is ambiguous, only NPV speaks.*

### 8.3 Cost of Capital: Building a WACC and Testing the Weights *(illustrative — all figures invented)*

**Issue.** A Cymbal Bank subsidiary must set a discount rate for a non-bank subsidiary's investment appraisal. The inputs: `Rf = 2.75%`, `ERP = 5.5%`, levered `β = 1.15`, pre-tax `Rd = 4.2%`, tax `= 17%`. What is WACC, and how much does the weighting convention move it?

**Analysis.** `Re = 2.75% + 1.15 × 5.5% = 9.075%`. After-tax `Rd = 4.2% × (1 − 0.17) = 3.486%`.
At **70/30** market weights: `WACC = 0.70 × 9.075% + 0.30 × 3.486% = 7.3983%`.
At **60/40**: `6.8394%`. At **85/15** (a low-leverage posture): `8.2366%`. The weight convention moves WACC by **1.4 percentage points**, enough to flip a marginal project.

**Alternatives.** (A) Use current market weights (70/30). (B) Use the stated target weights (85/15 for a bank-like capital floor). (C) Use book weights to match the regulatory definition of capital.

**Recommendation.** Use **target market weights**, disclose them, and run the appraisal at both the 70/30 and 85/15 rates as a sensitivity. ⚠ The choice is a convention (§3.4); the right discipline is to *fix it, state it, and test it*, not to present one WACC as fact.

### 8.4 Capital Structure: The Trade-off Optimum *(illustrative — all figures invented)*

**Issue.** A firm's unlevered value is **SGD 5,000**; the tax rate is **17%**; financial-distress costs rise with leverage. How much debt should it carry?

**Analysis.** MM with taxes says `V_L = V_U + Tc × D`, which rises without limit at 17 cents per dollar of `D`. Adding illustrative expected distress costs:

| Debt `D` | Tax shield `Tc·D` | PV of distress costs | Firm value |
|---|---|---|---|
| 0 | 0 | 0 | 5,000 |
| 1,000 | 170 | 20 | 5,150 |
| **2,000** | **340** | **120** | **5,220** |
| 3,000 | 510 | 420 | 5,090 |
| 4,000 | 680 | 1,100 | 4,580 |

**Alternatives.** (A) Zero debt (MM no-tax corner). (B) Maximum debt (MM with-tax corner). (C) The trade-off optimum at `D = 2,000`.

**Recommendation.** `D = 2,000`, value 5,220 — the point where the marginal tax shield meets the marginal distress cost (§4.4). Note the shape: the MM model alone would choose (B) — a corner no real firm occupies; the trade-off theory is what makes the model usable, and ⚠ its empirical fit is imperfect (§4.7). Presenting (B) as "the model's answer" is the textbook error this case is designed to expose.

### 8.5 Valuation: The Terminal-Value Convention Swamp *(illustrative — all figures invented)*

**Issue.** A Cymbal Bank unit projects `FCF = 100` per year for five years, `WACC = 9%`. Value it two ways: a Gordon terminal value at `g = 3%`, and an exit multiple of **8× EBITDA** on year-5 EBITDA of **130**.

**Analysis.** Explicit-period PV `= 100 × [1 − 1.09⁻⁵]/0.09 = 388.97`.
Gordon: `TV₅ = 100 × 1.03 / (0.09 − 0.03) = 1,716.67`; `PV = 1,716.67/1.09⁵ = 1,115.72`; **EV = 1,504.68**.
Multiple: `TV₅ = 8 × 130 = 1,040`; `PV = 675.93`; **EV = 1,064.89**.
The two conventions differ by **41.3%** on the same cash flows.

**Alternatives.** (A) Gordon terminal value. (B) Exit multiple. (C) Both, as a range.

**Recommendation.** Report **(C) a range**, with the terminal assumption named and stress-tested — the terminal value is ~74% of the Gordon EV. ⚠ A single DCF number is a convention dressed as a measurement (§5.2); the honest output is sensitivity, and the cross-check is a multiples view.

### 8.6 Working Capital: The Cash the Cycle Eats *(illustrative — all figures invented)*

**Issue.** A distributor has revenue **36,500**, COGS **25,550**, average inventory **4,200**, receivables **6,000**, payables **3,500**. It funds the cycle at **6%**. What does the cycle cost, and what does tightening it release?

**Analysis.** `DIO = 60.0 d`; `DSO = 60.0 d`; `DPO = 50.0 d`; `CCC = 70.0 d`. Daily revenue `= 100`. The cycle's funding cost `≈ 70 × 100 × 6% = 420/year`. Cutting DSO by **10 days** releases `10 × 100 = 1,000` of cash and saves `60/year` — a permanent one-off cash release plus a recurring saving. ⚠ On a credit-sales (80%) basis, DSO would read **75 days**, not 60 (§6.2) — the release estimate moves with the definition.

**Alternatives.** (A) Do nothing. (B) Tighten DSO by 10 days (credit control). (C) Extend DPO by 10 days (supplier terms) — the mirror image, with the risk of supplier pricing or rationing.

**Recommendation.** **(B)**, with the definition of DSO fixed and disclosed before the target is set. The cash release is real; the target is only meaningful once the measurement convention is pinned down.

### 8.7 Risk and Return: The Diversification Arithmetic *(illustrative — all figures invented)*

**Issue.** A portfolio holds **60%** in an asset with `σ = 20%` and **40%** in one with `σ = 12%`, correlation `ρ = 0.3`. What is portfolio volatility, and what does the market pay for its systematic risk?

**Analysis.** `σ_p² = 0.6²×0.20² + 0.4²×0.12² + 2×0.6×0.4×0.3×0.20×0.12 = 0.02016` → `σ_p = 14.20%`. The naive weighted average is `0.6×20% + 0.4×12% = 16.80%`; diversification removed **2.6 points** of risk with no change in expected return. For the SML (`Rf = 3%`, `ERP = 5.5%`), an asset with `β = 1.2` earns `E(R) = 3% + 1.2 × 5.5% = 9.6%`.

**Alternatives.** (A) Hold the single high-σ asset. (B) Hold the diversified 60/40 portfolio. (C) Hold the portfolio and add factor exposures (size, value).

**Recommendation.** **(B)** on the diversification arithmetic (it dominates (A) on a risk-adjusted basis if the expected returns are equal); treat **(C)** cautiously — ⚠ the CAPM factor debate (§7.4) is unresolved, and the estimation error in optimisers is real. The transferable point: diversification is the only "free lunch" the theory grants, and it is arithmetic, not opinion.

### 8.8 Options and Real Options: The Value of Not Committing *(illustrative — all figures invented)*

**Issue.** A Cymbal Bank platform option: commit **SGD 12,000** now, or pilot for **SGD 3,000** and hold the right to invest the 12,000 in a year. The project's present value today is a **10,000** asset with annual up/down factors of **1.40 / 0.70**; the risk-free rate is **3%**. Is the option to defer worth anything?

**Analysis.** Static NPV of committing now `= 10,000 − 12,000 = −2,000` → reject. But with the right to wait, the payoff is a call: up-state `max(14,000 − 12,000, 0) = 2,000`; down-state `max(7,000 − 12,000, 0) = 0`. Risk-neutral `p = (1.03 − 0.70)/(1.40 − 0.70) = 0.4714`. `Option value = (0.4714 × 2,000 + 0.5286 × 0) / 1.03 = 915.4`. **The right to defer is worth ~915**, so the staged structure beats both "commit now" and "do nothing".

**Alternatives.** (A) Commit now (NPV −2,000). (B) Do nothing (value 0). (C) Pilot and defer (static NPV of the staged cost plus the option value ≈ 915 minus the pilot cost, subject to whether the pilot is itself staged).

**Recommendation.** **(C)** — stage the investment and preserve the abandon/defer option; the point is not the precise 915 but that on a static DCF the project is dead and on a real-options view it is alive. ⚠ Real options are widely taught and less reliably practised (§7.10); use the *thinking* to justify staging, and be sceptical of any single "real-options value" presented as a measurement.

---

## 9. Banking Applications: Cost of Capital, RAROC/EVA, and Business-Line Valuation

### 9.1 Why a Bank's Cost of Capital Is a Different Animal (⚗ framed; ✅ in substance)

For a non-financial firm, debt is a financing choice; for a bank, **debt — deposits and wholesale borrowings — is the raw material of the business**. The balance sheet *is* the production function. Three consequences:

1. **The regulatory capital floor binds.** A bank cannot choose its leverage to minimise WACC; it must hold a minimum of equity against risk-weighted assets ([basel_regulatory_capital_guide.md](basel_regulatory_capital_guide.md)). The capital structure is a *constraint*, not a dial.
2. **Deposit funding is the product, not just the liability.** The spread between asset yield and deposit cost is the business model; "cheap debt" is not a free lunch but the franchise.
3. **The binding constraint is the cost of *equity*.** Because equity must be held and is the scarcest, most expensive claim, the decisive question is whether a business earns enough to cover the cost of the equity capital it consumes — the ROE/RAROC question, not the WACC question. Cross-ref the ALM and funding machinery in [treasury_alm_guide.md](treasury_alm_guide.md).

### 9.2 RAROC and EVA (⚠ — practitioner/vendor-attributed concepts)

- **RAROC (Risk-Adjusted Return on Capital):** broadly, `RAROC = risk-adjusted return / economic capital`, where the numerator nets expected loss from revenue and the denominator is the economic (often VaR- or stress-based) capital allocated to the activity. **⚠ Vendors and risk systems define it differently** — RAROC, RORAC (return on risk-adjusted capital), RARORAC and their relatives differ in the numerator (pre- or post-tax, expected loss treatment, cost allocation) and in the denominator (regulatory vs economic capital). The label is a family, not a formula.
- **EVA (Economic Value Added):** `EVA = NOPAT − WACC × Capital employed`. Positive EVA means the business earned more than the cost of the capital it tied up. **⚠ EVA is attributed to Stern Stewart & Co. as a trademarked practitioner product**, though the underlying residual-income concept predates it and has an academic lineage. Treat "EVA" as a *branded implementation of residual income*.
- The banking-specific point: because capital is regulated and scarce, **capital allocation is the scarce-resource problem of §2.6** — the PI/knapsack logic applies with regulatory capital as the rationed resource. Cross-ref [risk_management_models_guide.md](risk_management_models_guide.md) for the capital-allocation modelling.

### 9.3 Valuation of a Business Line: A Trade-Finance DCF *(illustrative — all figures invented)*

**Issue.** Cymbal Bank wants to know what its trade-finance line is worth and whether it earns its keep. The line's illustrative figures: `NOPAT = 42` (SGD m), capital employed (economic capital) `= 420`, `WACC = 9%`, projected free cash flow `35, 38, 41, 43, 45` over five years, terminal growth `2%`.

**Analysis.** `RAROC = 42 / 420 = 10.0%` — above the 9% WACC, so value-creating. `EVA = 42 − 0.09 × 420 = 42 − 37.8 = 4.2`. DCF: `TV₅ = 45 × 1.02/(0.09 − 0.02) = 655.7`; discounted (`/1.09⁵ = 426.2`) plus the explicit-period PV (`155.5`) gives a **line value ≈ 581.6**. Note that the terminal value is again the majority of the value.

**The funds-transfer-pricing (FTP) problem (⚠).** A business line's reported profit depends on the *internal* price at which it is credited for deposits it raises or charged for funding it consumes. **FTP is an internal-pricing convention, vendor- and bank-specific** — there is no single "correct" FTP curve, and the choice can move a line's reported pre-tax profit by a wide margin. Value the line on the same FTP basis you manage it on, and disclose it.

**Alternatives.** (A) Value the line as a standalone DCF (above). (B) Value it on ROE versus the group's cost of equity. (C) Value it on RAROC versus the group's hurdle. (D) Value it as a *component* of a relationship (the deposits and payments it pulls in).

**Recommendation.** Use **RAROC/EVA as the operating test and DCF as the strategic value**, with **(D) as a sanity check** for relationship value that the standalone line P&L omits. ⚠ The enterprise-vs-equity bridge is *inverted* for a bank (§5.3): the line's "enterprise value" is not a meaningful object when debt is raw material; the equity-side metrics (ROE, RAROC, EVA) are the primary ones.

### 9.4 Working Capital in a Trade-Finance Book *(illustrative — all figures invented)*

**Issue.** A trade-finance book (letters of credit, guarantees, receivables discounting, forfaiting) has an illustrative cycle: revenue **240**, receivables **60**, collateral inventory **15**, payables **40** (COGS 180). What is its cash-conversion cycle, and what is the economics of funded versus unfunded exposure?

**Analysis.** `DSO = 60/240 × 365 = 91.2 d`; `DIO = 15/180 × 365 = 30.4 d`; `DPO = 40/180 × 365 = 81.1 d`; **`CCC = 40.6 d`**. Funding the cycle at 5% costs `40.6/365 × 240 × 5% ≈ 1.33`. The trade-finance twist: **unfunded exposures** — LCs and guarantees — carry **no funding cost** but earn **commitment and utilisation fees**, while **funded exposures** — discounting and forfaiting — consume balance sheet and capital and earn a spread. The economics of the book is therefore a portfolio question: optimum fee income per unit of capital and per unit of funded exposure, not merely a spread. Cross-ref [trade_finance_guide.md](trade_finance_guide.md) and [supply_chain_finance_guide.md](supply_chain_finance_guide.md) — this guide condenses; those derive.

**Alternatives.** (A) Grow funded volume. (B) Grow unfunded (fee) volume. (C) Shift mix toward guarantee/LC fee income and hold funded exposure flat.

**Recommendation.** **(C)** where capital is the binding constraint: unfunded fee income consumes less regulatory capital per unit of revenue than funded discounting, so mix is the first lever. ⚠ All the definitions carry the §6.2 flags — the DSO/DPO basis materially changes the cycle here, and trade books are seasonal, so averages matter more than closing balances.

---

## 10. The Cymbal Bank Worked Case: The Trade-Finance Platform Investment

### 10.1 The Case

It was 7:15 on a Tuesday morning in March 2026, and **Mei Lin Tan**, Head of Trade Finance at Cymbal Bank, had one line left to change in a paper she was due to present to the investment committee at two o'clock. The paper asked for **SGD 12 million** to build a trade-finance origination and servicing platform in-house. Six months of feasibility work had gone into it. The question the committee would actually ask was the one she still could not answer cleanly:

> **"Is this worth doing at all — or are we building a platform because we are losing?"**

Cymbal Bank was the repository's fictional mid-sized Singapore bank: a retail deposit base, a respectable wholesale book, and a trade-finance line that had been a quiet earner for a decade. The line's problem was not profitability but *trajectory*: the line earned an illustrative **RAROC of 10%** against a **9%** economic cost of capital — value-creating, but by a thin margin — while customers were migrating to digital platforms that issued letters of credit and tracked guarantees in hours rather than days. The line was growing, and its cost-to-serve was not falling with it.

Mei Lin had a fourth fact on her desk that she had deliberately left out of the first draft: Cymbal had already spent **SGD 1.5 million** on the feasibility study and a vendor bake-off. It was a real cost, and a real number, and the Chief Financial Officer had put it in the summary "so the committee sees we've done the work." Mei Lin was not sure it belonged in the decision at all.

### 10.2 Exhibit 1 — Incremental Cash Flows (SGD '000) *(illustrative — all figures invented)*

| | Year 0 | Year 1 | Year 2 | Year 3 | Year 4 | Year 5 |
|---|---|---|---|---|---|---|
| Incremental EBITDA (fee & spread income) | — | 2,650 | 3,550 | 4,350 | 4,750 | 5,050 |
| Less: opportunity cost of redeployed staff | — | (450) | (450) | (450) | (450) | (450) |
| **Net pre-tax operating cash flow** | — | **2,200** | **3,100** | **3,900** | **4,300** | **4,600** |
| Depreciation (straight-line, on the 10,000 depreciable base, over 5 yrs) | — | 2,000 | 2,000 | 2,000 | 2,000 | 2,000 |
| Taxable income | — | 200 | 1,100 | 1,900 | 2,300 | 2,600 |
| Tax @ 17% | — | (34) | (187) | (323) | (391) | (442) |
| NOPAT | — | 166 | 913 | 1,577 | 1,909 | 2,158 |
| Add back depreciation (non-cash) | — | 2,000 | 2,000 | 2,000 | 2,000 | 2,000 |
| **Free cash flow (operating)** | — | **2,166** | **2,913** | **3,577** | **3,909** | **4,158** |
| Capital expenditure — platform build | (12,000) | | | | | |
| Working-capital investment / (release) | (900) | | | | | 900 |
| **Net cash flow to the firm** | **(12,900)** | **2,166** | **2,913** | **3,577** | **3,909** | **5,058** |

*Excluded by design: the SGD 1.5 m feasibility study (a sunk cost — §2.4); any allocation of group overhead (not incremental); and all financing flows (captured in the discount rate — the separation principle, §2.5).*

### 10.3 Exhibit 2 — WACC Build-Up *(illustrative — all figures invented)*

| Input | Value | Source / note |
|---|---|---|
| Risk-free rate `Rf` | 2.75% | Long-dated Singapore government proxy (invented) |
| Equity risk premium `ERP` | 5.5% | Mature-market premium (invented) |
| Levered beta `β` | 1.15 | Notional peer-set beta (invented) |
| **Cost of equity `Re = Rf + β·ERP`** | **9.075%** | `2.75% + 1.15 × 5.5%` |
| Pre-tax cost of debt `Rd` | 4.20% | Illustrative funding cost |
| Tax rate `Tc` | 17% | Singapore corporate rate |
| **After-tax cost of debt** | **3.486%** | `4.20% × (1 − 0.17)` |
| Weights `E/V` / `D/V` | 70% / 30% | **Target market weights** (the §3.4 convention) |
| **WACC** | **7.3983% ≈ 7.40%** | `0.70 × 9.075% + 0.30 × 3.486%` |

*⚠ Bank caveat: for the bank itself, the binding constraint is the cost of **equity** and the regulatory capital floor (§9.1); the 7.40% here is the discount rate for a capital-budgeting appraisal of a platform project, applied on a market-weight convention that is disclosed, not discovered.*

### 10.4 Exhibit 3 — The Decision Metrics (base case, no terminal value)

| Metric | Value | Reads as |
|---|---|---|
| **NPV @ 7.40%** | **+SGD 1,008k** | Positive — but thin (≈ 8% of the investment) |
| **IRR** | **9.93%** | Above the 7.40% WACC by ≈ 2.5 points |
| **Payback** | **4.07 years** | Inside the five-year plan, barely |
| **Discounted payback** | **4.72 years** | Almost the whole horizon consumed |
| **Profitability index** | **1.078** | $1.078 of PV per $1 invested |

### 10.5 Exhibit 4 — Sensitivity and Scenarios *(illustrative — all figures invented)*

**NPV vs WACC (base cash flows):**

| WACC | 6.4% | 6.9% | **7.4%** | 7.9% | 8.4% |
|---|---|---|---|---|---|
| NPV (SGD '000) | 1,438 | 1,220 | **1,008** | 799 | 596 |

**Ramp scenarios (year 4–5 revenue at 70% / 100% / 130% of plan):**

| Scenario | Slow ramp | **Base** | Fast ramp |
|---|---|---|---|
| NPV (SGD '000) | **(747)** | **1,008** | 2,762 |

**Terminal-value conventions (the decision's real hinge):**

| Convention | No terminal value | Exit multiple 3× EBITDA₅ | Gordon growth `g = 2%` |
|---|---|---|---|
| Block value (SGD '000) | — | 15,150 | 78,565 |
| NPV (SGD '000) | **1,008** | 11,611 | 55,992 |

**The partner alternative:** licence plus integration of **SGD 4,000k** now and **SGD 1,600k a year** for five years → **NPV +2,491k, IRR 28.6%** — faster, cheaper, less controlled.

### 10.6 Analysis

1. **The incremental discipline is the case.** The SGD 1.5 m feasibility study is **excluded** — it is sunk (§2.4). The **opportunity cost of redeployed staff (450/year) is included** — those people have a next-best use. The **working capital (900) is invested and released in year 5**. The **depreciation tax shield (340/year at 17%)** is embedded in the NOPAT-to-FCF bridge. Financing flows are excluded; the cost of capital carries them (§2.5).
2. **The base case is genuinely marginal.** NPV +1,008 on a 12,900 outlay is a PI of 1.078; the discounted payback is 4.72 years against a five-year horizon. This is not a "build it and the numbers speak" project — it is a project whose **decision hinges on assumptions**.
3. **The terminal assumption dominates.** The same operating plan gives NPV 1,008 with no terminal value, 11,611 on a 3× EBITDA exit, and 55,992 on a 2% Gordon terminal value. The case is *really* a debate about whether the platform has durable terminal economics — the §5.2 point, now with the committee in the room.
4. **The ramp is the second hinge.** A slow ramp turns NPV negative (−747). The near-term origination volume, not the terminal value, is what the pilot must prove.
5. **The banking angle.** On the bank's own metrics the platform must lift the line's **RAROC above its 10%-vs-9% margin (§9.3)**; a project that clears a 7.40% WACC hurdle but does not move RAROC has not answered the bank's actual question. ⚠ FTP convention (§9.3) will determine much of the reported lift.

### 10.7 Alternatives

| Option | Cost | Time | Control | Base NPV | Key risk |
|---|---|---|---|---|---|
| **A. Build in-house** | 12,000 + WC | 24–30 months | Full | +1,008 | Thin margin; benefits late |
| **B. Partner / licence** | 4,000 + 1,600 p.a. | 9–12 months | Shared | +2,491 | Dependency; leakage; exit terms |
| **C. Do nothing** | 0 | — | — | 0 (incremental) | Erosion of the book; the cost of *not* acting is not zero |
| **D. Phase (pilot then build)** | 3,000 pilot, then option on 9,000 | staged | Full if completed | Positive expected, preserves abandon option | Slower; two-stage governance |

The option to defer is not decoration: the §8.8 logic puts a positive value (~915 in the worked analogue) on the right to stop after the pilot — and on a base NPV of only 1,008, **the option is worth about as much as the project**.

### 10.8 Recommendation

**Proceed, conditionally — choose D (phase), with a hard stage gate.** Concretely: fund a **SGD 3.0 m nine-month pilot** of the build, gated on three measurable outcomes — origination volume, straight-through-processing rate, and unit cost-to-serve — and release the remaining **SGD 9.0 m** only if the gates are met. The decision rule, stated in the house form:

> **Proceed if NPV > 0 at the target WACC *and* the line's RAROC clears its hurdle *and* the pilot gates are met; abandon at the gate if they are not.**

The rationale is not that the base NPV is impressive — it is not. It is that the base NPV is thin, the terminal value is doing most of the work, and the ramp is the swing factor. A project whose value rests on assumptions that only a pilot can test should be *staged* to buy the right to stop. Option B is the fallback if the pilot misses; Option C is the fallback if both are rejected, and it should be priced honestly — doing nothing is not free when a book is eroding.

### 10.9 A Note on Cymbal Bank

**Cymbal Bank is this repository's explicitly fictional illustrative persona.** Every figure in §10 — the cash flows, the WACC inputs, the RAROC, the scenarios — is **invented**, arithmetically consistent, and computed for teaching purposes only. Nothing here describes a real institution or a real transaction. The conventions used across the repo's Cymbal Bank examples are set out in the sibling Cymbal case in [management/management_case_study_guide.md](../management/management_case_study_guide.md) §10 and applied across the banking guides; this guide follows the same convention, and the case is written in the case-analysis discipline that guide documents ([management/management_case_study_guide.md](../management/management_case_study_guide.md) §7).

---

## 11. Claims Audit

| Claim | Status | Source / note |
|---|---|---|
| Modigliani & Miller (1958), "The Cost of Capital, Corporation Finance and the Theory of Investment," *AER* 48(3): 261–297 | ✅ | Full text retrieved at aeaweb.org; volume XLVIII, June 1958, No. 3; page range confirmed |
| Modigliani & Miller (1963), "Corporate Income Taxes and the Cost of Capital: A Correction," *AER* 53(3): 433–443 | ⚠ | Standard citation; page range **not re-verified at source** here (the AER PDF URL returned 404) |
| MM Proposition I / II (no taxes): `V_L = V_U`; `Re = R₀ + (D/E)(R₀ − Rd)` | ✅ | Standard canon; derived in the 1958 paper |
| MM with taxes: `V_L = V_U + Tc·D` | ✅ | Standard canon; the 1963 correction |
| Modigliani Nobel Prize 1985 | ✅ | NobelPrize.org, 1985 — "for his pioneering analyses of saving and of financial markets" |
| Markowitz, Miller, Sharpe Nobel Prize 1990 | ✅ | NobelPrize.org, 1990 — "for their pioneering work in the theory of financial economics" (1/3 each) |
| Merton & Scholes Nobel Prize 1997 | ✅ | NobelPrize.org, 1997 — "for a new method to determine the value of derivatives" (1/2 each); Black not included (died 1995) |
| Gordon & Shapiro (1956), "Capital Equipment Analysis: The Required Rate of Profit," *Management Science* 3: 102–110 | ✅ | Cited in MM (1958) reference list: Manag. Sci., Oct. 1956, 3, 102–110 |
| Sharpe (1964), "Capital Asset Prices…," *Journal of Finance* 19: 425–442 | ✅ | Verified in the sibling [management/mba_body_of_knowledge_guide.md](../management/mba_body_of_knowledge_guide.md) §6.4 |
| CAPM developed independently by Treynor (1961/62), Sharpe (1964), Lintner (1965), Mossin (1966) | ✅ | Verified in the sibling MBA guide §6.4 ("Treynor–Sharpe–Lintner–Mossin, 1961–66") |
| Markowitz (1952), "Portfolio Selection," *Journal of Finance* 7(1): 77–91 | ⚠ | Paper, journal, year and author beyond doubt; **exact page range not re-verified at source** (JSTOR blocked) |
| Black & Scholes (1973), *JPE* 81(3): 637–654; Merton (1973), *Bell Journal* 4(1): 141–183 | ⚠ | Standard citations; **not re-verified at source** here (publisher pages blocked) |
| Cox, Ross & Rubinstein (1979), *JFE* 7(3): 229–263 | ⚠ | Standard citation; not re-verified at source here |
| Myers (1984), "The Capital Structure Puzzle," *Journal of Finance* 39(3) | ⚠ | Canonical paper; **exact page range not re-verified at source** |
| Miller (1977), "Debt and Taxes," *JF* 32(2): 261–275; Ross (1977), *Bell J.* 8(1): 23–40; Roll (1977), *JFE* 4(2): 129–176 | ⚠ | Standard citations; not re-verified at source here |
| NPV is the theoretically correct capital-budgeting criterion | ✅ | Standard corporate-finance canon (Brealey–Myers–Allen and peers); stated in the sibling MBA guide §6.2 |
| IRR pathologies (multiple IRRs; Descartes' rule of signs; no-IRR; scale/timing; reinvestment assumption; mutual-exclusivity conflict) | ✅ standard | Standard curriculum content; the Descartes rule dates to Descartes, *La Géométrie*, 1637 |
| The annuity-due vs ordinary-annuity, book-vs-market weights, target-vs-current weights, Rf/ERP and DSO/DIO/DPO conventions vary across texts | ⚠ | Convention-dependent; flagged throughout |
| DuPont decomposition origin: DuPont Corp. / Donaldson Brown, c. 1919 | ⚠ | Standard attribution; **authorship depth and exact date contested** |
| EVA attributed to Stern Stewart & Co. | ⚠ | Practitioner/vendor-attributed; residual-income concept predates it |
| RAROC / RORAC / RARORAC definitions vary by vendor and risk system | ⚠ | Practitioner-sourced; a family of definitions |
| Funds-transfer-pricing (FTP) is an internal-pricing convention | ⚠ | Vendor- and bank-specific; no single canonical curve |
| Damodaran's resources at `pages.stern.nyu.edu/~adamodar` | ✅ exists / ⚠ not retrieved | The resource is real and well known; **not retrievable at extraction time here**, so no figure is quoted |
| CFA curriculum topic statements and weights | ⚠ | Cross-referenced to [cfa_program_guide.md](cfa_program_guide.md) §3; not re-verified this edition |
| All worked-example arithmetic (§1, §5–§10) | ✅ | Computed with the terminal (`python3`); every discount factor, WACC, NPV, IRR, ratio, and day-count reproduced |
| The Cymbal Bank case and every figure in it | ✅ fictional | Constructed illustrative case; all figures invented |

---

## 12. What Could Not Be Verified

- ⚠ **The page range and DOI of Modigliani & Miller (1963).** The AER "top-20" PDF URL for the 1963 correction returned 404; the citation is given in its standard form but was not re-verified at a source for this guide.
- ⚠ **JSTOR and publisher records for several papers** (Markowitz 1952; the Sharpe 1964 record; Black–Scholes 1973; Merton 1973; Cox–Ross–Rubinstein 1979; Myers 1984). JSTOR served a "Client Challenge" and the Wiley/Chicago/ScienceDirect pages failed to retrieve; those citations are stated in their standard textbook forms but their exact page ranges and DOIs were **not re-verified here**.
- ⚠ **Damodaran's public NYU Stern resources.** `pages.stern.nyu.edu/~adamodar` failed to retrieve (scraping-engine error). The resource is real and widely used, but **no figure or dataset from it is quoted** in this guide.
- ⚠ **Brealey, Myers & Allen, *Principles of Corporate Finance* — the precise edition and publisher-page details.** Referenced as the canon; the edition line was not verified at a publisher page here.
- ⚠ **The CFA Institute curriculum's exact current topic weights/statements.** The CFA programme page loaded but the detailed curriculum statements were not extracted; this guide cross-references [cfa_program_guide.md](cfa_program_guide.md) §3 instead of re-deriving them.
- ⚠ **The depth of the DuPont/Donaldson Brown attribution.** The technique is certain; the authorship and the c. 1919 date are reported with variance and were not verified to a primary source.
- ⚠ **Hamada (1972) page details, Michaud (1989) page details, and the Trigeorgis (1996) / Copeland–Antikarov (2001) edition details.** All are standard references; their bibliographic precision was not re-verified here.
- ⚠ **EVA, RAROC, RORAC, and FTP definitions.** These are practitioner- and vendor-specific; the guide states them as families of conventions, not as verified single definitions.
- ⚠ **`web_search` was unavailable (rate-limited) during this guide's preparation**, so verification was performed by direct extraction of primary/publisher URLs. Several primary sources (JSTOR, Wiley, Chicago, ScienceDirect) block automated retrieval — a limitation of the tooling, not of the citations' existence.
- ⚠ **Any claim about the Cymbal Bank case being real.** It is constructed; every exhibit is invented.

---

## 13. Glossary

| Term | Meaning |
|---|---|
| **Annuity** | A level stream of cash flows; **ordinary** if payments fall at period end, **due** if at the start |
| **APR / EAR** | Annual percentage rate (nominal, ignores intra-year compounding) / effective annual rate (`(1 + APR/m)^m − 1`) |
| **Beta (β)** | Sensitivity of an asset's return to the market: `Cov(Rᵢ, R_m)/Var(R_m)` |
| **BSM (Black–Scholes–Merton)** | The option-pricing model of Black–Scholes (1973) and Merton (1973) |
| **CAPM** | Capital Asset Pricing Model: `E(Rᵢ) = Rf + βᵢ·ERP`; developed by Treynor, Sharpe, Lintner, and Mossin |
| **CCC** | Cash-conversion cycle = `DIO + DSO − DPO` |
| **Country risk premium (CRP)** | An increment to the ERP for exposure to a sovereign's risk (Damodaran-style treatment) |
| **DCF** | Discounted cash flow valuation |
| **DDM** | Dividend-discount model; the constant-growth special case is the "Gordon growth model" |
| **Descartes' rule of signs** | The count of positive real roots of a polynomial is bounded by its sign changes — the source of multiple-IRR cases |
| **DIO / DSO / DPO** | Days inventory outstanding / days sales outstanding / days payables outstanding |
| **DuPont identity** | `ROE = net margin × asset turnover × equity multiplier` (DuPont Corp., c. 1919) |
| **EVA** | Economic value added = `NOPAT − WACC × capital employed`; a Stern Stewart–branded residual income |
| **Enterprise value (EV)** | Value of operations to all capital providers; `Equity + Net debt` |
| **ERP** | Equity risk premium = `E(R_m) − Rf` |
| **FCFE / FCFF** | Free cash flow to equity / to the firm |
| **FTP** | Funds-transfer pricing — the internal price at which a bank's units are charged/credited for funds |
| **Hamada relation** | Levering/unlevering beta: `β_L = β_U[1 + (1 − Tc)(D/E)]` |
| **IRR** | Internal rate of return — the discount rate setting NPV to zero |
| **MIRR** | Modified IRR — an IRR with explicit reinvestment/financing rates (⚠ naming/treatment varies) |
| **MM propositions** | Modigliani–Miller results on capital structure irrelevance (1958) and the tax correction (1963) |
| **NPV** | Net present value = PV(inflows) − PV(outflows) at the cost of capital |
| **NWC** | Net working capital = current assets − current liabilities |
| **PI** | Profitability index = PV(future cash flows)/initial investment |
| **Put–call parity** | `C − P = S − Ke^(−rT)` — a model-free arbitrage identity |
| **RAROC** | Risk-adjusted return on capital; ⚠ vendor-specific variants (RORAC, RARORAC) |
| **Real options** | The options embedded in real investments: defer, expand, abandon, switch |
| **Residual income** | `Net income − cost of equity × beginning book equity` |
| **SML** | Security market line: the CAPM relationship plotted in `E(R)`–`β` space |
| **Sunk cost** | A cost already incurred and unrecoverable; excluded from incremental analysis |
| **WACC** | Weighted average cost of capital; the FCFF discount rate |
| **Cymbal Bank** | This repository's fictional Singapore bank persona, used throughout Part D |

---

## 14. Sources and Further Reading

**Primary / verified for this guide:** American Economic Association — Modigliani & Miller, "The Cost of Capital, Corporation Finance and the Theory of Investment," *AER* 48(3), June 1958, pp. 261–297 (`aeaweb.org`, full text). NobelPrize.org — the 1985, 1990, and 1997 Economic Sciences prize summaries (Modigliani; Markowitz–Miller–Sharpe; Merton–Scholes). Gordon & Shapiro (1956) as cited in MM's 1958 reference list. Sharpe (1964) as verified in the sibling MBA guide §6.4.

**Standard corporate-finance canon:** Richard Brealey, Stewart Myers & Franklin Allen, *Principles of Corporate Finance* (McGraw-Hill) — the NPV/IRR, capital-budgeting, and WACC treatments. Aswath Damodaran, *Investment Valuation* and the *Damodaran on Valuation* line (Wiley), plus his free NYU Stern resources at `pages.stern.nyu.edu/~adamodar` (risk premiums, betas, multiples, lecture notes). Stephen Ross, Randolph Westerfield & Jeffrey Jaffe, *Corporate Finance*.

**Journal canon (cited, standard forms):** Modigliani & Miller (1958, 1963); Miller (1977), "Debt and Taxes," *JF* 32(2); Myers (1984), "The Capital Structure Puzzle," *JF* 39(3); Myers & Majluf (1984), *JFE* 13(2): 187–221; Ross (1977), *Bell J.* 8(1): 23–40; Markowitz (1952), "Portfolio Selection," *JF* 7(1): 77–91; Sharpe (1964), *JF* 19(3): 425–442; Lintner (1965), *RES* 47(1): 13–37; Mossin (1966), *Econometrica* 34(4): 768–783; Black & Scholes (1973), *JPE* 81(3): 637–654; Merton (1973), *Bell J.* 4(1): 141–183; Cox, Ross & Rubinstein (1979), *JFE* 7(3): 229–263; Roll (1977), *JFE* 4(2): 129–176; Hamada (1972), *JF* 27(2); Michaud (1989), *FAJ*.

**Texts cited for options and capital structure framing:** Lenos Trigeorgis, *Real Options* (MIT Press, 1996); Tom Copeland & Vladimir Antikarov, *Real Options: A Practitioner's Guide* (2001).

**Currency of the concepts:** For the CFA curriculum's Corporate Issuers and Portfolio Management topics (the analyst's framing of the same material), see [cfa_program_guide.md](cfa_program_guide.md) §3; for the MBA finance core (the manager's framing), see [management/mba_body_of_knowledge_guide.md](../management/mba_body_of_knowledge_guide.md) §6; for the case-analysis discipline applied throughout Part B and Part D, see [management/management_case_study_guide.md](../management/management_case_study_guide.md) §7 and its Cymbal case in §10.

**Internal cross-references (banking and technology):** [basel_regulatory_capital_guide.md](basel_regulatory_capital_guide.md); [treasury_alm_guide.md](treasury_alm_guide.md); [risk_management_models_guide.md](risk_management_models_guide.md); [trade_finance_guide.md](trade_finance_guide.md); [supply_chain_finance_guide.md](supply_chain_finance_guide.md); [capital_markets_architecture_guide.md](capital_markets_architecture_guide.md); [hedge_fund_guide.md](hedge_fund_guide.md); [investment_portfolio_operations_guide.md](investment_portfolio_operations_guide.md); [management/business_case_development_guide.md](../management/business_case_development_guide.md); [../technology/tco_modeling_guide.md](../technology/tco_modeling_guide.md); [../technology/finops_guide.md](../technology/finops_guide.md).

---

*The concepts are the grammar and the cases are the sentences; verify the formula, price the convention, and decide on the incremental cash flow — because that is what separates a number from a decision, and it is why the canon still earns its keep — the finance case.*
