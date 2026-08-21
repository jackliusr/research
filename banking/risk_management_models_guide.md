# Risk Management Models in Banking: A Comprehensive Guide

*The quantitative risk-model deep-dive — from Merton's structural model to FRTB's expected shortfall, from Basel IRB scorecards to machine-learning PD models.*

**Series:** Banking & Financial Technology Guides — this is the dedicated companion to the risk-and-compliance **systems** landscape (financial_risk_compliance_systems_guide.md). Where that guide maps the *platforms, vendors, and data flows* that run risk functions, this guide maps the *mathematics, methodologies, and regulatory model frameworks* themselves. Where that guide had ~38 passing model mentions across §3–6, this guide is the full model-level treatment.

**Audience:** Solution architects, quants, risk technologists, and engineers building or integrating risk platforms (e.g., at a global investment bank like Crédit Agricole CIB, or a Singapore wholesale/corporate bank) who need to know *what the models actually compute* and *why the regulators demand what they demand*.

**How this guide is organised:** a risk taxonomy (the four families of risk and the models that measure them); then one deep-dive per risk family (credit, market, operational, liquidity); then the capital models that consume all of them; then stress testing; then the governance layer (model risk management); then the machine-learning frontier; then a worked example tracing a PD model through its full lifecycle; a one-page summary; and a glossary. Every section ends with a reference table.

**Scope note on verification:** dates and attributions below were checked against primary sources and reputable references (Basel Committee documents, the 1974/1995/1994 academic canon, regulator publications, industry press) in August 2026. Where a fact could not be fully verified, it is explicitly flagged **[verify]** rather than asserted. The April 2026 OCC model-risk guidance refresh is flagged in §9.

**Cross-references used throughout:** financial_risk_compliance_systems_guide.md (the systems landscape — the complement to this guide), banking_limits_domain_guide.md (how model outputs feed limit setting), asset_backed_trading_guide.md (securitisation risk models), interest_engines_core_banking_guide.md and core_banking_processes_guide.md (loan products whose risk these models price), murex_mx3_platform_guide.md (market-risk modules), universal_banking_model_guide.md (bank-wide operating model), insurance_open_source_commercial_guide.md (insurance risk parallels), ../technology/distributed_auth_guide.md (identity risk), ../technology/adversarial_ml_attacks_guide.md (ML-risk-model security), ../technology/ai_llm/ai_governance_bias_redteaming_guide.md (model governance for AI), ../technology/ai_llm/llm_instruction_tuning_guide.md, ../technology/defi_guide.md (crypto risk models).

### Reading paths

Different readers will want different routes through this guide:

- **Solution architects and engineers** — read §1.1 (the model ecosystem) and §11 (the worked example's design section) for the architecture pattern; §9 (model risk management) for the governance workflow that every model system must encode; and the per-risk tables (§3.3, §4.3, §5.1, §6.1) for the model inventory you are building systems around. Pair with financial_risk_compliance_systems_guide.md for the vendor landscape.
- **Quants and modellers** — the core is §3 (credit families and the ASRF formula), §4 (VaR/ES and the volatility/dependence machinery), and §5 (LDA mechanics); the worked examples in §4.5, §5.3, §6.3, and §7.3 show the arithmetic end-to-end.
- **Risk managers and business stakeholders** — §2 (the taxonomy), §8 (stress testing), and §12 (the one-page summary) give the governance-level picture; the glossary in §13 covers the vocabulary.
- **ML/data-science teams** — §10 (ML in risk) plus §9 (the SR 11-7 discipline your models must survive); cross-reference ../technology/adversarial_ml_attacks_guide.md and ../technology/ai_llm/ai_governance_bias_redteaming_guide.md for the security and AI-governance layers.

Each section is written to stand alone: the tables at the end of each section are the quick-reference, and the cross-references let readers jump between this guide and its siblings without losing context.

---

## Table of Contents

1. Introduction: Why Risk Models Are the Bank's Nervous System
2. The Risk Taxonomy: Credit, Market, Operational, Liquidity
3. Credit Risk Models: PD, LGD, EAD and the Model Families
4. Market Risk Models: VaR, RiskMetrics, and Expected Shortfall
5. Operational Risk Models: From AMA/LDA to the SMA
6. Liquidity Risk: LCR, NSFR, and Cashflow Modelling
7. Capital Models: Basel I, II, III and the RWA Machine
8. Stress Testing: CCAR, EBA, MAS and Scenario Design
9. Model Risk Management: SR 11-7 and the Validation Discipline
10. Machine Learning in Risk: The New Model Frontier
11. Worked Example: A PD Model Lifecycle at a Singapore Bank
12. Summary: The Mathematics of Prudence (One Page)
13. Glossary

---

## 1. Introduction: Why Risk Models Are the Bank's Nervous System

A bank is, at bottom, a machine for taking calculated risk: it borrows short and lends long (liquidity and interest-rate risk), it lends to borrowers who may not repay (credit risk), it holds trading books exposed to prices and rates (market risk), and it runs people and processes that can fail (operational risk). Every one of those exposures is measured with a *model* — a quantitative method that maps observable data into a number: a probability of default, a value-at-risk, a capital charge, a stressed loss.

Three forces made these models not just useful but *mandatory*:

- **Regulation.** The Basel accords (1988, 2004, 2010–2017) hard-wired models into the capital framework. Banks that can prove their models are better than the regulator's standardised weights are rewarded with lower capital — the "internal ratings-based" (IRB) bargain that turned model quality into a P&L matter.
- **Crisis.** Every major banking crisis of the last 30 years was, in part, a model failure: LTCM (1998, VaR underestimation), the 2007–2009 crisis (correlation and rating models), the 2023 US regional-bank failures (interest-rate and liquidity modelling under stress). Regulators responded with heavier stress testing (CCAR, EBA) and heavier model governance (SR 11-7).
- **Mathematics.** The academic canon — Merton (1974), Jarrow–Turnbull (1995), RiskMetrics (1994), Artzner et al. on coherent risk measures (1999) — supplied the tools: option-pricing theory for default, quantile-based risk measures, and loss distributions.

**The central equation of the whole discipline** is the expected-loss identity:

```
EL = PD × LGD × EAD
```

Expected loss = probability of default × loss given default × exposure at default. Credit risk models are, at heart, three sub-models producing these three inputs; market risk models produce a *distribution* of P&L and read off a quantile; operational risk models build a loss distribution from frequency and severity; liquidity models simulate cashflows. Capital is then the difference between the *unexpected* loss (the tail) and the *expected* loss (provisioned for).

**The architecture view.** For a solution architect, the model layer sits between the data layer and the decision layer: data warehouses and risk data marts (BCBS 239-grade lineage) feed model engines (scorecards, VaR engines, stress platforms), which feed consumers — regulatory capital (RWA engines), limit systems (see banking_limits_domain_guide.md for how model outputs become limits), pricing, provisioning (IFRS 9 ECL), and management information. The systems themselves, vendor by vendor, are covered in financial_risk_compliance_systems_guide.md; this guide covers what runs *inside* them.

A note on tone: formulas are kept to their canonical, attributable forms; everything else is explained in plain language. Where a formula's attribution is disputed or a regulatory detail varies by jurisdiction, the text says so.

### 1.1 The model ecosystem: from data to decision

A risk model never lives alone — it sits inside a pipeline that runs from source systems to board decisions, and understanding that pipeline is half the battle for anyone building risk technology. The canonical five-layer stack:

1. **Source systems** — the systems of record that generate the raw material: core banking (loans, deposits — see core_banking_processes_guide.md and interest_engines_core_banking_guide.md), trading and market data (prices, curves, vols — see murex_mx3_platform_guide.md), payments, collateral (asset_backed_trading_guide.md), and external feeds (bureau, ratings, macro).
2. **Risk data platform** — the integration, storage, and lineage layer. BCBS 239 (2013) made this a regulatory deliverable in its own right: risk data must be accurate, complete, timely, and *reconcilable* to the books and records, with documented lineage from source field to model input (see financial_risk_compliance_systems_guide.md §2.2 and §10).
3. **Model engines** — the calculations themselves: scorecard and PD/LGD/EAD engines, VaR/ES engines, op-risk capital calculators, LCR/NSFR engines, stress-test satellite models, ECL calculators. Each engine in this layer is an SR 11-7 "model" with an inventory entry, a validator, and a monitoring regime (§9).
4. **Decision consumers** — limit systems (banking_limits_domain_guide.md), pricing and RAROC, provisioning (IFRS 9), capital and RWA reporting, regulatory submissions (CCAR/DFAST, EBA, MAS returns), and management MI.
5. **Governance and oversight** — model inventory, validation workflow, backtesting, issue tracking, and the committees (model risk committee, capital committee) that sign off changes.

Two architectural truths follow. First, **model outputs are only as good as the data lineage and the engine's reproducibility**: the same PD must be reproducible from the same data at any later date — which is why versioned code, versioned data, and calculation logs are audit staples. Second, **models are consumed by other models**: a PD feeds an ECL engine and a RAROC calculator and a stress-test satellite, so a model change ripples through the whole estate — which is why model change management (impact analysis across consumers) is a first-class process, not an afterthought.

### 1.2 How models fail: the crisis casebook

Every major model family in this guide has a documented failure mode, and the history of the industry is written in them. Knowing the casebook is what makes the governance sections (§9–10) feel necessary rather than bureaucratic:

- **LTCM (1998).** Long-Term Capital Management's convergence-arbitrage book used historical correlations that had never been observed at crisis stress levels; when Russia defaulted, correlations went to 1, the model said "diversified," the market said otherwise, and a $4.6bn bailout was required. The lesson: historical correlation is not a promise.
- **VaR and the 2008 crisis.** Pre-crisis, banks ran 99%/10-day VaR under benign volatility; the Gaussian assumption priced the tails at near-zero probability, and model risk (the models themselves were wrong, not just unlucky) produced losses that regulators later quantified as a major contributor to capital shortfalls. The regulatory response was the 2009 Basel 2.5 stressed-VaR patch and then the FRTB (§4.2) — the direct lineage from crisis to this guide's ES formulas.
- **Rating and correlation models (2007–2008).** The Gaussian-copula models that priced CDO tranches assumed stable, low asset correlations; when housing correlations jumped, senior tranches — rated AAA by models that assumed they could not both default — defaulted en masse. David X. Li's 2000 copula paper became the most infamous formula in finance. The lesson: dependence is the hardest thing to model, and stress tests with correlation breaks (§8) exist because of it.
- **The 2023 US regional-bank failures.** Silicon Valley Bank and Signature Bank failed after rapid rate rises crushed the market value of their long-duration bond books — a *banking-book* interest-rate and liquidity problem that no VaR number had flagged, because the accounting model (held-to-maturity at amortised cost) and the liquidity model (stable deposits assumed to be sticky) both failed simultaneously. The lesson: models that ignore the *interaction* of risk types — rate risk, liquidity risk, depositor behaviour — miss the failure mode that actually kills banks.
- **The AMA (2000s–2016).** Operational-risk internal models accumulated complexity while failing to predict crisis losses, ending in the AMA's removal (§5). The lesson: a model whose complexity exceeds its data will eventually be retired by regulators.
- **Model risk itself (2011–).** SR 11-7 codified what all of the above had in common: the *model* was the risk. From then on, "did the model work?" became a supervised question with an inventory, validators, and consequences.

The casebook has a single moral, which this guide repeats in every section: **models are conditional statements — conditional on data, on assumptions, on regime — and the discipline of risk management is making those conditions explicit, tested, and governed.**

---

## 2. The Risk Taxonomy: Credit, Market, Operational, Liquidity

The Basel Committee's taxonomy groups a bank's risks into four pillar risk families — **credit, market, operational, and liquidity** — plus a penumbra of secondary risks (interest-rate risk in the banking book, concentration risk, model risk, climate risk, conduct risk) that are managed *through* the four families. The taxonomy is not academic: each family has its own model families, its own capital treatment, and (in most banks) its own front office of quants and risk managers.

**Credit risk** — the risk that a borrower or counterparty fails to meet its obligations. It is the largest risk on a typical bank balance sheet and the most model-mature. Instruments: loans, bonds, derivatives counterparty exposure, trade finance (see trade finance guides in this series), securitisations (asset_backed_trading_guide.md). Models: PD/LGD/EAD components, structural and reduced-form default models, scorecards, portfolio credit models (CreditMetrics, CreditRisk+, KMV), CVA models for counterparty credit risk.

**Market risk** — the risk of loss from adverse movements in market prices: interest rates, FX, equities, commodities, credit spreads, implied volatilities. It arises in the trading book (marked-to-market) and, in a different form, in the banking book (interest-rate risk in the banking book, IRRBB). Models: VaR (parametric, historical, Monte Carlo), expected shortfall (ES), stress scenarios, sensitivities (delta/vega), and the FRTB machinery that now governs trading-book capital.

**Operational risk** — the risk of loss from inadequate or failed internal processes, people, and systems, or from external events; includes legal risk and (in some definitions) model risk, and excludes strategic and reputational risk. Basel's definition (BCBS, 2004) is the canonical one. Models: the advanced measurement approach's loss distribution approach (AMA/LDA — now superseded by the standardised measurement approach, SMA), scenario analysis, key risk indicators (KRIs), and increasingly machine learning on loss-event data.

**Liquidity risk** — the risk that the bank cannot meet its obligations as they fall due without incurring unacceptable losses. Two faces: *funding liquidity risk* (cannot raise cash) and *market liquidity risk* (cannot sell assets without moving the price). Models: the LCR and NSFR ratio frameworks, cashflow-mismatch (maturity ladder) models, funding concentration analysis, and intraday liquidity monitoring. Liquidity was the great blind spot of Basel I/II — Basel III fixed it.

**The secondary risks** are worth naming because they route into the four families: *concentration risk* (a credit-risk overlay), *IRRBB* (market/ALM), *model risk* (governed via SR 11-7, §9), *climate risk* (transmission via credit and market risk — now in EBA stress tests), *conduct and compliance risk* (operational-adjacent; the systems in §7–8 of financial_risk_compliance_systems_guide.md), and *reputational risk* (mostly unmodellable; managed qualitatively).

### 2.1 The taxonomy table

| Risk family | Definition (canonical source) | Typical models | Capital/measurement home |
|---|---|---|---|
| Credit risk | Risk of loss from obligor or counterparty failure to meet obligations (Basel II, 2004) | PD/LGD/EAD models, Merton structural, Jarrow–Turnbull reduced-form, scorecards, CreditMetrics, CreditRisk+, CVA | Basel Pillar 1 — Standardised or IRB (F-IRB/A-IRB); IFRS 9 ECL for provisioning |
| Market risk | Risk of loss from adverse price/rate/volatility movements (Basel II 2004; FRTB 2019) | VaR (parametric/historical/Monte Carlo), ES, stress scenarios, sensitivity measures | Basel Pillar 1 — Standardised Approach (SBM) or IMA under FRTB; internal capital for IRRBB |
| Operational risk | Risk of loss from inadequate/failed internal processes, people, systems, or external events; incl. legal risk (Basel II, 2004) | LDA (frequency × severity), scenario analysis, KRIs, SMA capital formula | Basel Pillar 1 — SMA (replaced AMA per Basel III 2016–17 revisions) |
| Liquidity risk | Risk of inability to meet obligations as they fall due without unacceptable losses (BCBS 2010/2013) | LCR (HQLA vs 30-day net outflows), NSFR (1-year stable funding), maturity-ladder / cashflow models, concentration metrics | Basel III liquidity standards (not Pillar 1 capital — a separate quantitative regime) |
| Secondary (credit-concentration, IRRBB, model risk, climate) | Overlays on the four families | Concentration limits (banking_limits_domain_guide.md), IRRBB EVE/NII sensitivity, SR 11-7 model inventory, climate scenarios | Pillar 2 (ICAAP/SREP), supervisory guidance |

**Why the taxonomy matters to architects:** risk systems are organised exactly along these lines — credit risk systems, market risk systems, operational risk systems, treasury/liquidity systems (the §3–6 of financial_risk_compliance_systems_guide.md map) — and the data models, calculation engines, and reporting feeds are all partitioned by risk family. When someone asks "which model feeds this number?", the first question is always "which risk family?".

### 2.2 The taxonomy in practice: from risk appetite to limits

The taxonomy is not a filing system — it is the skeleton of the bank's risk governance, and it connects to everything else in this series:

- **Risk appetite.** The board's risk appetite statement is written *per risk family*: "we will not lose more than X of CET1 in a severe recession" (credit), "VaR at 99%/1-day will not exceed Y" (market), "op-risk losses will stay under Z" (operational), "LCR will be held above 130%, not 100%" (liquidity). Each of those statements is only enforceable because the corresponding model quantifies it.
- **Limits.** Model outputs become limits: PD/rating grids set credit limits per counterparty, VaR/ES limits per desk and per book, LCR/NSFR floors for the treasury, loss tolerance for op risk. The limit lifecycle — design, approval, monitoring, enforcement — is the subject of banking_limits_domain_guide.md, and it is the *execution layer* of every model in this guide.
- **Pricing and profitability.** RAROC = risk-adjusted return ÷ economic capital, where economic capital is itself a model output per risk family; a trade or loan is priced so that its RAROC clears the hurdle. The models therefore sit inside the front-to-back pricing chain (universal_banking_model_guide.md).
- **Stress testing.** Each family has its own stress: credit stress (macro → PD/LGD shocks), market stress (scenario shocks to prices/vols), liquidity stress (funding run-off), op-risk stress (tail scenarios). The taxonomy dictates which scenarios each stress programme must cover (§8).
- **Organisation.** Banks run separate front-office risk functions per family (credit risk, market risk, operational risk, treasury risk), each with its own model owners, validators, and committees — the taxonomy is literally the org chart. That matters for systems: "risk" at a bank is not one platform but four (or more) federated estates, which is exactly what financial_risk_compliance_systems_guide.md documents vendor by vendor.

---

## 3. Credit Risk Models: PD, LGD, EAD and the Model Families

Credit risk modelling is the oldest and deepest quantitative discipline in banking. Its core decomposition — expected loss as the product of three components — comes from the Basel framework's treatment of credit risk and is the organising principle of every credit model suite:

```
EL = PD × LGD × EAD
```

- **PD — probability of default**: the likelihood that the obligor defaults within a given horizon (Basel's regulatory horizon is 12 months; IFRS 9 adds lifetime horizons). A *default event* is defined consistently (Basel: 90+ days past due, or "unlikeliness to pay" — bankruptcy, restructuring, etc.) so that PDs are comparable across portfolios.
- **LGD — loss given default**: the share of exposure lost once default occurs, after recoveries, collateral realisation, and workout costs: `LGD = 1 − recovery rate`. Ranges from a few percent on secured residential mortgages to 60–100% on unsecured wholesale exposure.
- **EAD — exposure at default**: the amount outstanding at the time of default, including undrawn commitments drawn before default: `EAD = drawn + (undrawn × credit conversion factor)`. For derivatives, EAD is a *potential future exposure* measure (add-on or simulated).

These three are never modelled in isolation: they share data (default histories), they share model governance (one validation, one model inventory entry, one backtesting regime — §9), and they feed the same downstream consumers (RWA, ECL, pricing, limits). The regulatory link between them is the Basel IRB capital formula, which converts PD, LGD, EAD and maturity (M) into a risk weight via an asymptotic single-risk-factor (ASRF) model — the Vasicek-style one-factor model that Basel II adopted.

**The other key identity** is the relationship between expected and unexpected loss: a bank *provisions* EL (it is a statistical certainty) and *holds capital* for UL (the tail beyond EL, typically the 99.9% VaR of the loss distribution under IRB). This EL/UL split is the heart of the capital models in §7.

### 3.1 The three model families

Credit models come in three canonical families, each answering "when and why does a firm default?" differently.

**Structural models (the Merton family).** Robert Merton, "On the Pricing of Corporate Debt: The Risk Structure of Interest Rates", *Journal of Finance* 29(2), 1974, pp. 449–470 — verified. The insight: a firm's equity is a **European call option on the firm's assets**, with strike equal to the face value of the debt and maturity equal to the debt's maturity. Default occurs at maturity *T* if asset value *V_T* falls below the debt face value *D*:

```
Equity value  E = V·N(d1) − D·e^(−rT)·N(d2)     (Black–Scholes with V, D as parameters)
Default       V_T < D  ⟹  PD = N(−d2)  ("distance to default" = d2)
```

The model turns a balance-sheet statement into a market-implied default probability: firm value dynamics (asset volatility, leverage) imply how far the firm is from the default boundary — the *distance to default*. Moody's KMV commercialised it (expected default frequencies, EDFs). Strengths: economically grounded, forward-looking via equity prices. Weaknesses: default can only happen *at* debt maturity in the basic model; it notoriously understates short-term credit spreads (empirical literature from the 1980s–90s found structural models price short-dated spreads far below observed ones) — the "credit spread puzzle". Extensions (Black–Cox first-passage with a barrier, Longstaff–Schwartz, Leland–Toft) relax the at-maturity assumption.

**Reduced-form models (the Jarrow–Turnbull family).** Robert Jarrow and Stuart Turnbull, "Pricing Derivatives on Financial Securities Subject to Credit Risk", *Journal of Finance* 50(1), 1995, pp. 53–85 — verified. Default is not derived from firm value but modelled directly as an exogenous, unpredictable event: the first jump of a Poisson process with a **default intensity** λ(t). The survival probability to time T is:

```
P(survive to T) = exp(−∫₀ᵀ λ(s) ds)
```

The intensity is *calibrated to observable market prices* — credit spreads on bonds and CDS — rather than to balance sheets. This makes reduced-form models the workhorse for pricing credit derivatives (CDS, CLNs) and for CVA, because they fit the market term structure of credit and can handle default *before* maturity naturally. Duffie–Singleton (1999) extended the framework to risky discounting. Strengths: market-consistent, flexible. Weaknesses: less economically interpretable than structural models; default is a "black box" jump.

**Scorecards (logistic regression).** The workhorse of retail and SME lending. A scorecard is a logistic-regression model mapping borrower characteristics (application data, bureau data, behavioural data) to a PD:

```
PD(x) = 1 / (1 + e^−(β₀ + β₁x₁ + … + βₖxₖ))
Score  = offset + scale × ln(odds)   (odds = PD/(1−PD)); higher score ⇒ lower PD
```

Standard practice bins continuous variables into *weight-of-evidence* (WoE) categories before regression, which gives the monotonic, interpretable, regulator-friendly form that rating agencies and IRB supervisors expect. Scorecards come in three flavours: **application** (new business, uses application + bureau data), **behavioural** (existing customers, uses account behaviour — the "scorecard = logistic regression + scaling" formulation is standard industry practice, e.g., SAS's credit-scorecard methodology), and **collection** (delinquency management). Discriminatory power is measured with Gini/AUROC and KS; calibration is checked with binomial tests and the Hosmer–Lemeshow test.

### 3.2 The IRB approach: Foundation vs Advanced

Basel II (2004) introduced the **internal ratings-based (IRB) approach** — verified: banks may, *subject to supervisor approval*, use their own models for credit risk capital instead of the standardised risk weights. The bargain: better risk sensitivity in exchange for model discipline (the "use test" — the model must genuinely drive day-to-day risk management, not just capital). Basel II 2004 is the canonical source; the approach was carried into Basel III and then significantly *restricted* by the Basel III finalisation (December 2017, "Basel III: Finalising post-crisis reforms", colloquially "Basel IV"/Basel 3.1).

**Foundation IRB (F-IRB)** — the bank models **PD**; the regulator prescribes **LGD, EAD, and maturity (M)** through fixed supervisory values per exposure class (e.g., senior unsecured corporate LGD of 45% — the Basel II supervisory LGD). **Advanced IRB (A-IRB)** — the bank models **PD, LGD, and EAD** (and M), using its own data and models throughout. A-IRB is the prestige tier: lower capital for well-modelled portfolios, but far heavier data and validation requirements (the bank must demonstrate it can estimate LGD and EAD from its own loss and exposure history — a decade-scale data requirement).

**Basel 3.1 (2017) reined IRB in** — verified in outline: equity exposures lose IRB entirely (simple risk-weight approach only, per the Bank of England's implementation text cited in research), and A-IRB is removed for certain asset classes (large corporates, financial institutions, unlisted equities — banks must use F-IRB or standardised), with *input floors* on PD and LGD (e.g., a 5bp PD floor for corporate and bank exposures — [verify: exact floor schedule per the December 2017 text]) and a 72.5% *output floor* (RWA under internal models cannot fall below 72.5% of standardised RWA). The direction of travel is unambiguous: after 2008, regulators decided internal models were too generous and began dialling them back.

### 3.3 The credit model table

| Model | Family | What it computes | Typical use |
|---|---|---|---|
| Merton (1974) | Structural | Distance to default → PD from equity price & balance sheet | Public corporates, market-implied PDs; ancestor of KMV/EDF |
| Black–Cox / Longstaff–Schwartz | Structural (first-passage) | PD with default before maturity (barrier) | Corporate debt pricing, ratings migration |
| Jarrow–Turnbull (1995) / Duffie–Singleton | Reduced-form (intensity) | Survival/default probabilities from credit spreads; risky discounting | CDS pricing, credit derivatives, CVA |
| Application/behavioural scorecards | Logistic regression | PD per obligor from characteristics | Retail/SME origination, account management |
| Rating-migration models (Moody's/KMV, CreditMetrics) | Portfolio | Joint default/migration distribution | Portfolio credit VaR, economic capital |
| CreditRisk+ (actuarial) | Portfolio (Poisson) | Loss distribution from default rates | Portfolio capital, low-data portfolios |
| LGD models (workout, market, implied) | Statistical/structural | Recovery rate per facility | IRB capital, ECL, pricing |
| EAD/CCF models | Statistical | Exposure at default incl. commitments | IRB capital, facility limits |
| CVA models | Reduced-form + simulation | Counterparty credit valuation adjustment | Derivative pricing, XVA desk (see murex_mx3_platform_guide.md) |
| IFRS 9 ECL models | PD × LGD × EAD, 12m & lifetime | Expected credit loss for provisioning | Financial statements (see financial_risk_compliance_systems_guide.md §3.5) |

### 3.4 The IRB capital formula: the asymptotic single-risk-factor model

The IRB risk weights that turn PD/LGD/EAD into capital are themselves a model — the **asymptotic single-risk-factor (ASRF) model**, the Vasicek one-factor framework that Basel II adopted and Basel III retained (with parameter floors). Its logic: each obligor's asset value is driven by one *systematic* factor (the state of the economy) plus an *idiosyncratic* factor; default occurs when assets fall below the default point; in an infinitely granular portfolio, the idiosyncratic noise diversifies away and the loss distribution is driven purely by the systematic factor. The capital requirement for a corporate exposure (the canonical form, as published in the Basel II text) is:

```
K = LGD × [ N( ( N⁻¹(PD) + √ρ · N⁻¹(0.999) ) / √(1−ρ) ) − PD ] × maturity adjustment
RWA = K × 12.5 × EAD
asset correlation ρ = 0.12 × (1 − e^(−50·PD))/(1 − e^(−50)) + 0.24 × (1 − (1 − e^(−50·PD))/(1 − e^(−50)))
```

Read it piece by piece: N⁻¹(PD) is the obligor's default point on the standard-normal asset-value scale; N⁻¹(0.999) is the supervisor-set 99.9% confidence level (the "unexpected loss" target); the ratio inside N(·) is the *conditional* default probability given the systematic factor at its 99.9% worst; subtract PD (the expected loss, which is provisioned) and multiply by LGD to get the unexpected-loss capital K; the maturity adjustment scales up for long-dated exposures; and the ×12.5 converts capital to risk-weighted assets (the inverse of the 8% minimum — K × 12.5 = K/0.08). The correlation function is the subtle part: PD-dependent and asset-class-dependent, so that *lower-rated obligors get lower assumed correlation* (they are more idiosyncratic — their defaults are less synchronised with the economy), which is the model's mechanism for keeping capital sensitive to portfolio composition.

The ASRF's strengths and its critics are both famous. Strengths: a closed-form capital number from three inputs, portfolio-granularity justification, and a transparent confidence level. Criticisms: it assumes infinite granularity (single-name concentration must be handled separately, via Pillar 2), a single systematic factor (no industry/region structure), and *static* correlations — the assumption that asset correlations are stable proved disastrously wrong in 2008, when systematic correlation jumped toward 1 and "diversified" portfolios defaulted together. This is why §8's stress testing exists: the ASRF models the normal crisis; stress tests model the correlation-breakdown crisis.

### 3.5 IFRS 9 ECL: the accounting twin of the regulatory models

Since IFRS 9 (effective 2018, with CECL as the US analogue), the same PD/LGD/EAD machinery produces **provisioning**, not just capital — and the accounting version differs from the regulatory version in ways that matter to model builders:

- **Horizons.** IFRS 9 requires *12-month* ECL for performing assets (Stage 1) and *lifetime* ECL once credit risk has increased significantly since origination (Stage 2, "SICR" — significant increase in credit risk) or once defaulted (Stage 3). Lifetime ECL needs a **PD term structure** (1-year PDs rolled forward) — a component many banks had to build from scratch, often cohort-based or Merton-style.
- **Point-in-time vs through-the-cycle.** Regulatory IRB PDs are TTC (calibrated to the long-run average default rate); IFRS 9 demands *forward-looking, point-in-time* PDs that respond to the current and expected economic state — so banks run the *same* scorecard through a macro overlay that shifts PDs with GDP/unemployment forecasts, and weight **multiple economic scenarios** (base + upside + downside, probability-weighted) rather than a single path.
- **Where they meet.** The EL identity is identical (ECL = PD × LGD × EAD), but the numbers differ: the same loan can have an IRB PD of 2% and an IFRS 9 Stage-2 lifetime PD of 6% at the same date, and both are "correct" for their purpose. Reconciliation between the two — regulatory capital and accounting provisions — is a standing headache for risk-and-finance data integration (financial_risk_compliance_systems_guide.md §11.2 covers the systems side).

The practical lesson for model governance: most large banks now maintain a *unified* credit-model platform where one scorecard feeds both the IRB calibration chain and the IFRS 9 staging/ECL chain, with the TTC/PIT distinction applied as an overlay. That is exactly the kind of architecture decision a solution architect owns — and it is why this guide's worked example (§11) treats the PD model as a multi-consumer asset.

### 3.6 The structural/reduced-form divide in practice: pricing, CVA, and the spread puzzle

The choice between structural and reduced-form families is not academic — it determines which problems each model is used for, and the boundary is worth making precise:

- **Structural models are balance-sheet models.** They shine where you have equity prices and a balance sheet (public corporates), and they produce *economically interpretable* outputs — distance to default, expected default frequency, the leverage that drives the PD. Their classic extensions: **Black–Cox** (default before maturity when asset value hits a barrier), **Longstaff–Schwartz** (stochastic interest rates plus default barrier), **Leland–Toft** (endogenous default when equity value makes it optimal to stop servicing debt), and the **KMV approach** (empirical mapping from distance-to-default to historical default frequencies rather than a closed-form normal assumption).
- **Reduced-form models are market models.** They shine where the market prices credit directly — corporate bonds, CDS, credit-linked notes, and the whole **CVA/XVA** complex. A bank's CVA desk prices counterparty risk on derivatives by simulating counterparty exposures (Monte Carlo, §4.1) and discounting them with *risk-adjusted* curves built from CDS spreads — which is precisely the Jarrow–Turnbull/Duffie–Singleton machinery: default intensity calibrated to the CDS market, survival probabilities from the intensity, and expected exposure weighted by survival. The systems side of this — the XVA engines inside trading platforms — is covered in murex_mx3_platform_guide.md.
- **The credit spread puzzle.** Structural models, calibrated honestly, systematically *underprice* short-dated credit spreads — the market charges more for near-term default risk than balance-sheet fundamentals imply. The puzzle has driven a half-century of research (taxes, liquidity premia, jump-to-default, rare disasters) and is the main reason the industry uses reduced-form models for anything market-priced: when the market is the counterparty, you calibrate to the market.
- **Where they meet.** Modern credit portfolios blend them: scorecards for origination (retail/SME), structural/KMV for public-corporate monitoring, reduced-form for pricing and CVA, and portfolio models (CreditMetrics-style migration models, CreditRisk+ actuarial models) for the correlation-driven capital and economic-capital numbers. The "one true model" does not exist; the art is matching family to use — which is what the credit model table (§3.3) is for.

---

## 4. Market Risk Models: VaR, RiskMetrics, and Expected Shortfall

Market risk is the risk of loss from adverse moves in prices, rates, volatilities, and spreads. Its canonical measure is **Value at Risk (VaR)**: the loss that will not be exceeded at a given confidence level over a given horizon. Formally, for loss L over horizon h:

```
VaR_α = inf{ x : P(L ≤ x) ≥ α }     (the α-quantile of the loss distribution)
```

Standard conventions: **99% confidence, 1-day horizon** for internal risk management; **99%, 10-day horizon** for regulatory market-risk capital (Basel II's internal models approach); **97.5%, one-tailed, 10-day** for the FRTB internal models approach (§4.2). VaR is a single number from a *distribution* — its three estimation methods differ only in how that distribution is built.

**RiskMetrics (1994).** J.P. Morgan published the first RiskMetrics Technical Document in **October 1994** (fourth edition December 1996) — verified. It was the first public, bank-grade VaR methodology, and it defined the **parametric (variance–covariance) approach**: assume daily P&L is multivariate normal, then

```
VaR = α × σ_p × √h     where σ_p² = w′ Σ w  (portfolio variance from the covariance matrix)
```

with volatilities estimated by **exponentially weighted moving averages (EWMA)**, λ = 0.94 for daily data (the famous RiskMetrics decay factor). EWMA gives fast-adapting volatility that spikes after shocks — a deliberate response to the fat tails and volatility clustering that plain historical variance misses. RiskMetrics' gift to the industry was *standardisation*: a published covariance-matrix methodology that every bank could implement, which is why "RiskMetrics-style VaR" still describes the parametric norm. Its limitation is the normality assumption — Gaussian VaR understates tail risk, which the 2008 crisis demonstrated brutally.

### 4.1 The three VaR methods

**Parametric (variance–covariance).** As above: assume a distribution (normal, or Student-t for fatter tails), estimate the covariance matrix (EWMA or GARCH), compute portfolio variance, read off the quantile. Fast, closed-form, cheap — ideal for large books of linear instruments. Weaknesses: normality assumptions, poor for options (non-linear payoffs need delta-gamma approximations), poor for fat tails.

**Historical simulation.** Revalue the current portfolio under *actual historical return scenarios* — take the last N trading days' (e.g., 250–500) realised moves in every risk factor, apply them to today's positions, sort the resulting P&L, and read off the quantile. No distributional assumption, no covariance estimation — it uses the empirical joint distribution of the risk factors directly, which captures fat tails and correlations as they actually occurred. Weaknesses: the past may not contain today's risks (the 2008 CDO shock wasn't in the 2007 history), and it needs full revaluation per scenario, which is computationally heavy for exotic books (mitigated by delta-approximation "historical VaR with sensitivities").

**Monte Carlo simulation.** Generate thousands of synthetic risk-factor scenarios by simulating from estimated stochastic processes (geometric Brownian motion for equities, Hull–White or similar for rates, with copulas or correlated Brownians for dependence), fully revalue the portfolio under each scenario, and read the quantile from the simulated P&L distribution. Most flexible and accurate for options and path-dependent instruments; also the standard for counterparty-credit exposure simulation (potential future exposure) and for the FRTB's expected shortfall. Weaknesses: model risk in the chosen processes, computational cost, and dependence on estimated parameters.

**Backtesting** is the mandatory quality check: compare daily VaR against actual P&L and count exceptions (days the loss exceeded VaR). Basel's traffic-light approach (green/yellow/red zones, 1–4/5–9/10+ exceptions in 250 days) scales a multiplicative capital multiplier ("plus factor") — a piece of regulatory design where the model's own performance adjusts the capital it generates. Kupiec's POF test and Christoffersen's independence test are the statistical workhorses.

### 4.2 Expected Shortfall and the FRTB

**Expected shortfall (ES)** — the average loss *beyond* the VaR quantile:

```
ES_α = E[ L | L > VaR_α ]
```

ES answers VaR's blind spot: VaR says nothing about *how bad* the tail is (two portfolios with identical 99% VaR can have wildly different 99.5%+ losses). ES also fixes VaR's mathematical flaw — VaR is *not subadditive* (portfolio VaR can exceed the sum of its parts' VaRs, breaking diversification logic), while ES is a **coherent risk measure** (Artzner, Delbaen, Eber, Heath, 1999), including subadditivity. ES is thus the theoretically respectable tail measure, and it is now the regulatory standard.

**FRTB — the Fundamental Review of the Trading Book.** The Basel Committee's post-crisis rebuild of market-risk capital, published as "Minimum capital requirements for market risk" (revised January 2016; final text January 2019) — verified in outline. Key changes: (1) **ES at 97.5%, one-tailed, 10-day horizon replaces the 99% VaR** for the internal models approach — calibrated so ES ≈ old VaR in normal markets but captures the tail properly; (2) a **stressed calibration** (ES computed on a 12-month period of maximum stress — the "stressed ES") replaces Basel 2.5's add-on stressed VaR; (3) **non-modellable risk factors** (NRFs) — factors lacking enough observed data — cannot go through the internal model; they attract a *capital add-on* computed by scenario stress, which forces banks to hold data to prove modellability; (4) a new **standardised approach (SA)** with a sensitivities-based method (SBM) — risk-factor sensitivities × prescribed risk weights, with correlation buckets — plus default-risk charges; (5) strict **P&L attribution tests** between the desk's daily P&L and the model's hypothetical P&L, gating IMA eligibility; (6) a clearer **boundary between the trading book and banking book** (the "trading book boundary" with a presumption of tradability), ending the arbitrage of parking risky assets in the banking book. FRTB also revised the **CVA framework** (SA-CVA and BA-CVA approaches). Implementation has been staggered and repeatedly delayed — the EU applied it via CRR3 from **January 2025**; other jurisdictions (US, UK, Japan, Singapore) moved on their own timelines [verify: current implementation status per jurisdiction as of 2026]. The direction is unmistakable: internal models for market risk are smaller, more expensive, and more strictly gated than ever.

### 4.3 The market risk table

| Model / approach | Method family | Computes | Strengths | Weaknesses |
|---|---|---|---|---|
| Parametric VaR (RiskMetrics 1994) | Variance–covariance, EWMA | VaR from covariance matrix | Fast, standardised, cheap | Normality, linear-only, tail-blind |
| Historical simulation VaR | Empirical scenarios | VaR from past P&L scenarios | No distributional assumption, captures empirical tails | Past ≠ future, data-hungry, revaluation cost |
| Monte Carlo VaR | Simulated scenarios | VaR (and full loss distribution) | Handles options/path-dependence, flexible | Model risk, computational cost |
| Stressed VaR (Basel 2.5, 2009) | Historical on stressed window | SVaR add-on | Captures crisis regimes | Add-on mentality, double counting |
| Expected shortfall (FRTB IMA, 2019) | ES at 97.5%, 10-day, stressed | Tail-average loss | Coherent, tail-correct | Needs more data than VaR, harder backtest |
| Sensitivities-based approach (FRTB SA) | Risk-weight × sensitivities | Standardised capital | No internal model needed, comparable | Conservative, correlation buckets crude |
| IRRBB models (EVE/NII) | Re-pricing gap / simulation | Banking-book rate sensitivity | Captures the non-trading book | Not in FRTB; Pillar 2 / BCBS 368 |
| Scenario stress (bespoke) | Deterministic shocks | Loss under defined scenarios | Transparent, executable | Not probabilistic |

### 4.4 Volatility, dependence, and the tails: the machinery behind the methods

All three VaR methods and ES stand on two pieces of statistical machinery that deserve their own section, because they are where models live or die:

**Volatility modelling.** The parametric method's covariance matrix is only as good as its volatility forecasts. The canonical approaches: (1) **EWMA** (RiskMetrics): σ²_t = λ·σ²_(t−1) + (1−λ)·r²_(t−1), with λ = 0.94 for daily data — a recursive smoothing that weights recent shocks heavily and needs no estimation beyond λ; (2) **GARCH(1,1)**: σ²_t = ω + α·r²_(t−1) + β·σ²_(t−1) — the workhorse econometric model that adds a long-run mean level (ω) to EWMA and is estimated by maximum likelihood; (3) implied-volatility-based forecasts from options markets, which are forward-looking where the others are backward-looking. The empirical facts that drive all of this — volatility *clusters* (calm periods and stormy periods persist), has *fat tails* (daily returns are not normal — kurtosis far above 3), and is *asymmetric* (down-moves raise volatility more than up-moves) — are exactly what the 2008 and 2020 episodes demonstrated, and exactly what the move from VaR to ES and from normal to Student-t assumptions is responding to.

**Dependence and tail estimation.** Portfolio risk is a correlation story: the whole is riskier than the sum when correlations rise. The modern toolkit: (1) **copulas** — separate the marginal distributions from the dependence structure, so you can have Gaussian margins with *t-copula* tail dependence (joint extreme moves) or *Clayton* lower-tail dependence; copulas are how Monte Carlo engines generate correlated scenarios from arbitrary marginals, and how AMA/LDA op-risk models correlate units of measure; (2) **extreme value theory (EVT)** — model the tail separately from the body using the generalised Pareto distribution fitted to exceedances over a high threshold (peaks-over-threshold), giving principled estimates of the 99.9th percentile that raw historical simulation cannot reach because the data simply doesn't contain enough 1-in-1000 events; (3) the sobering lesson of 2008 — **correlations are regime-dependent** and tend to 1 exactly when you need diversification most — which is why regulators now demand stress tests with correlation breaks rather than relying on estimated correlations (see §8).

For an architect the practical takeaway: the choice between the three VaR methods and ES is not a religious one — production market-risk systems (e.g., the risk modules in murex_mx3_platform_guide.md) typically run *all three in parallel*: parametric VaR for speed and daily limits, historical simulation for the default regulatory metric, Monte Carlo/ES for the trading book and exotic products, and stress scenarios on top. The governance question is which number is *the* limit number and which is the *capital* number — the answer being set by the FRTB for capital and by the bank's own risk appetite for limits.

### 4.5 A worked VaR example: the three methods on one book

A small concrete example makes the methods concrete. Take a SGD 100m book of Singapore government bonds and SGD 50m of USD/SGD FX forwards — a typical treasury desk book at a Singapore bank. Say the daily portfolio volatility is estimated at σ = 0.8% of notional (≈ SGD 1.2m daily P&L standard deviation), and the portfolio's 250-day historical P&L distribution has a 99th-percentile daily loss of SGD 2.9m, while Monte Carlo simulation (10,000 scenarios, t-distributed returns with 5 degrees of freedom) produces a 99th-percentile loss of SGD 3.6m.

- **Parametric VaR (99%, 1-day)** = 2.33 × 1.2m ≈ **SGD 2.8m**. Fast, closed-form, and — under the normal assumption — the *smallest* of the three numbers: the normal distribution cannot see the fat tails.
- **Historical simulation VaR (99%, 1-day)** = **SGD 2.9m**. Empirically anchored: whatever actually happened on the worst 2–3 days of the last 250 is the number — no distribution assumed, but only what the past contained.
- **Monte Carlo VaR (99%, 1-day)** = **SGD 3.6m**. The t-distribution's heavier tails generate more extreme scenarios than history actually delivered — and the 10-day scaling makes the gap bigger: √10 ≈ 3.16 multiplies all three, so the 10-day numbers are ≈ SGD 8.8m / 9.2m / 11.4m.
- **ES (97.5%, 10-day, the FRTB metric)** averages everything *beyond* the 97.5% quantile of the simulated 10-day distribution — say ≈ **SGD 13m** — a number no VaR formulation produces, because VaR by construction ignores the size of the tail beyond the quantile.

The demonstration is the whole point: for the *same book at the same moment*, the four numbers differ by a factor of ~1.5. None is "wrong" — they answer different questions (normal-world quantile, empirical quantile, tail-heavy quantile, average tail loss). This is why market-risk governance is fundamentally about *which question the limit is set on*, and why the FRTB's move to ES was a change of question, not just of parameter. It is also why a risk system that shows "the VaR" without stating method, confidence, and horizon is committing the field's cardinal sin — ambiguity is the enemy of risk reporting.

---

## 5. Operational Risk Models: From AMA/LDA to the SMA

Operational risk is the orphan of the model world: it lacks the elegant theory of Merton, yet it has produced some of the biggest single losses in banking history (rogue trading, fraud, processing failures — the systems angle is in financial_risk_compliance_systems_guide.md §5). Basel II (2004) defined it as *the risk of loss resulting from inadequate or failed internal processes, people and systems, or from external events; this definition includes legal risk, and excludes strategic and reputational risk* — verified. Basel II offered three approaches of escalating sophistication: the **Basic Indicator Approach (BIA)** (15% of gross income), the **Standardised Approach (TSA)** (12–18% of gross income per business line), and the **Advanced Measurement Approach (AMA)**.

**AMA and the LDA.** The AMA let banks use internal models for operational-risk capital, subject to supervisory approval — verified. The canonical AMA model is the **loss distribution approach (LDA)**: segment the bank into *units of measure* (UoMs — business line × event-type cells, using Basel's eight business lines and seven event types), then for each UoM estimate (a) a *frequency distribution* (how often losses occur — Poisson or negative binomial) and (b) a *severity distribution* (how big they are — lognormal, Weibull, or generalised Pareto from extreme value theory for the tail), convolve the two into an annual aggregate loss distribution, and take the **99.9th percentile of the one-year loss** as capital:

```
OpRisk capital (AMA/LDA) = VaR_99.9% of the 1-year aggregate loss distribution
   Aggregate loss = Σ over events of severity, with number of events ~ Frequency
```

LDA inputs combine four mandated data elements: **internal loss data** (a five-year history minimum), **external loss data** (industry consortia such as ORX), **scenario analysis** (expert judgements of tail events — "what if a payments outage lasted 3 days?"), and **business environment and internal control factors** (BEICFs). Correlations between UoMs are typically modelled with copulas so the total is less than the naive sum. Weaknesses that eventually killed the AMA: data scarcity (five years of internal data is nothing for a 1-in-1000-year capital number), scenario subjectivity, model complexity that even regulators found opaque (Risk.net's famous headline: "Even for me, AMA models are too complicated" — a Basel Committee member), and a demonstrated inability to predict the 2008 losses.

**The SMA takeover.** The Basel III revisions (consultative 2016, finalised December 2017) **removed the AMA** and replaced it with the **Standardised Measurement Approach (SMA)** — verified. The SMA is deliberately formulaic, not model-based: capital = the **Business Indicator (BI)** (a financial-statement proxy built from interest income, services income, and financial components) × a marginal coefficient schedule (12% up to a BI of €1bn, 15% from €1–30bn, 18% above €30bn), with a **Loss Component (LC)** add-on that scales with the bank's own 10-year average operational losses. The message was explicit: after a decade of AMA, the Committee concluded internal op-risk models added complexity without adding safety, and capital should be a transparent function of size and realised losses. This is the single clearest example in the whole Basel canon of *regulatory retreat from internal models* — and a warning to any bank still investing in heavyweight LDA infrastructure.

### 5.1 The op-risk table

| Model / approach | Family | Computes | Status |
|---|---|---|---|
| Basic Indicator Approach (BIA, Basel II 2004) | Simple formula | 15% × gross income | Replaced by SMA |
| Standardised Approach (TSA, Basel II 2004) | Business-line formula | 12–18% × gross income per line | Replaced by SMA |
| AMA — Loss Distribution Approach (Basel II 2004) | Frequency × severity convolution | 99.9% 1-year VaR of loss distribution | Removed by Basel III (2016–17); legacy systems still in production at some banks |
| Extreme value theory / GPD severity tails | Statistical tail modelling | Tail severity parameters for LDA | Now mainly scenario/ORX analytics |
| Standardised Measurement Approach (SMA, Basel III 2017) | BI × marginal coefficients + loss component | Formulaic op-risk capital | Current Basel standard; EU CRR3 applies from 2025 |
| Scenario analysis (expert) | Structured judgement | Tail-event loss estimates | Survives as risk-management tool, no longer capital-direct |
| KRI dashboards / early warning | Indicator monitoring | Loss-proxy trends | Management tool, feeds Pillar 2 |

### 5.2 The op-risk classification engine: business lines and event types

Every op-risk model — LDA, SMA, scenario analysis, KRI — depends on a consistent classification of losses, and Basel's taxonomy is the universal one. **Eight business lines** (corporate finance; trading & sales; retail banking; commercial banking; payment & settlement; agency services; asset management; retail brokerage) and **seven event types** (internal fraud; external fraud; employment practices and workplace safety; clients, products & business practices; damage to physical assets; business disruption and system failures; execution, delivery & process management). A loss event is classified into a business-line × event-type cell, which becomes the *unit of measure* in an LDA — the cell where frequency and severity are estimated and convolved. This classification is deceptively important:

- **Consistency across time and entities** is what makes five years of internal loss data usable at all — a loss misclassified between "execution, delivery & process management" and "clients, products & business practices" lands in the wrong distribution and distorts capital. Banks run loss-data governance programmes (loss-data collection standards, per BCBS guidance) whose only job is to keep the classification honest.
- **The big-tail problem.** Operational loss data is dominated by a few giant events (the "heavy tail"): a single rogue-trading or fraud event can exceed the sum of thousands of small losses. This is why severity modelling uses EVT (generalised Pareto tails, §4.4), why scenario analysis exists (expert judgements about tail events the data cannot contain), and why the SMA's loss component uses a 10-year average — to smooth the tail rather than let one year's monster loss dominate.
- **The 2008 lesson, formalised.** The AMA's capital proved unresponsive to crisis losses — partly because internal loss histories had no 2008-like events, and partly because the model's complexity defeated its own scrutiny. The SMA's replacement formula (BI × coefficients + loss component) is the regulators' answer: op-risk capital should be a *transparent function of size and realised losses*, not an act of statistical imagination. The classification machinery survives, though — the SMA still needs clean, consistently classified loss data for the loss component, and the seven event types still structure the risk-and-control self-assessments (RCSAs) and scenario workshops that feed Pillar 2.

### 5.3 A worked LDA example: the payments outage

To make the LDA concrete before it is retired to history: suppose Meridian's "payment & settlement" business line has, per year, an average of 3 loss events above the collection threshold (frequency ~ Poisson(3)), with severities distributed lognormally (median SGD 0.5m, heavy tail — occasionally a settlement failure or fraud event lands at SGD 20–50m). Monte Carlo convolution: draw 10,000 simulated years, each with a Poisson number of events and lognormal severities, sum them, and read the 99.9th percentile of the resulting annual-loss distribution — say **SGD 38m**. That number is the AMA capital for that unit of measure; add the other UoMs with a copula correlation structure, and the bank gets its total AMA op-risk capital.

The exercise exposes both the LDA's appeal and its fatal weakness in one breath: the 99.9th percentile is being estimated from a distribution whose shape rests on ~3 events/year and a handful of scenarios — the tail is *imagination, not data*. (Contrast: the SMA computes the same bank's op-risk capital from the Business Indicator and a 10-year realised-loss average — no tail estimation at all.) For architects with legacy LDA implementations still in production: the calculation engines, loss-data warehouses, and scenario-capture tools survive as *management* infrastructure — they now feed Pillar 2 ICAAP and scenario analysis rather than Pillar 1 capital — so the systems investment is not wasted, just re-purposed.

---

## 6. Liquidity Risk: LCR, NSFR, and Cashflow Modelling

Liquidity risk is the risk that the bank cannot meet its obligations as they fall due without incurring unacceptable losses. Basel I and II said almost nothing about it — the 2007–2009 crisis showed why that mattered: Northern Rock, Bear Stearns, and Lehman died not of insolvency but of *funding*. Basel III responded with a dedicated liquidity framework — verified: "Basel III: International framework for liquidity risk measurement, standards and monitoring" (December 2010), with the LCR standard finalised in January 2013 and the NSFR standard finalised in October 2014.

**LCR — Liquidity Coverage Ratio.** LCR = stock of **High-Quality Liquid Assets (HQLA)** ÷ **total net cash outflows over 30 calendar days**, required ≥ **100%** (phased in from 60% in 2015 to 100% in 2019):

```
LCR = HQLA / net cash outflows (30 days)  ≥  100%
```

HQLA is tiered: **Level 1** (cash, central-bank reserves, high-grade government bonds — counted at full value, no haircut) and **Level 2** (2A: high-grade covered bonds/Agency MBS; 2B: lower-grade corporate debt and equities — with haircuts and a 40% cap on Level 2 within HQLA). Net cash outflows are computed by applying *run-off factors* to liabilities and *inflow factors* to assets — retail deposits run off at 3–10% in 30 days, wholesale funding at higher rates, committed credit lines at 5–10% — which makes the LCR a *stress-scenario ratio*: "could this bank survive 30 days of the run-off factors Basel prescribes?" The LCR forced banks to hold real liquidity buffers for the first time; it is the reason modern treasury desks run daily LCR optimisation (see financial_risk_compliance_systems_guide.md §6 for the treasury systems that do this).

**NSFR — Net Stable Funding Ratio.** NSFR = **Available Stable Funding (ASF)** ÷ **Required Stable Funding (RSF)** over a one-year horizon, required ≥ **100%**:

```
NSFR = ASF / RSF  ≥  100%
```

ASF weights funding by stability (Tier 1/2 capital and long-term debt at 100%, retail deposits at 90–95%, wholesale funding at 0–50% by tenor); RSF weights assets by how much stable funding they *lock up* (cash at 0%, loans at 50–85% by type, long-dated illiquid assets at 100%). The NSFR is the structural, anti-maturity-mismatch ratio: it penalises the "fund long assets with short funding" model that killed 2023's failed US regional banks [verify: NSFR applicability in the US — the US finalised its NSFR rule in 2021 but there were later proposals to modify it]. Where the LCR is a 30-day stress gauge, the NSFR is a one-year structural balance-sheet constraint.

**Beyond the two ratios** sits the modelling undergrowth: *maturity-ladder / cashflow-mismatch models* (contractual cashflows vs behavioural assumptions — see interest_engines_core_banking_guide.md for how loan cashflows are computed), *behavioural modelling of non-maturing deposits* (core vs volatile portions — a genuine model family with the same governance needs as credit models), *funding concentration limits* (by counterparty, product, currency), *intraday liquidity* monitoring (BCBS 248), and *collateral management* (asset_backed_trading_guide.md). Liquidity risk is also where **stress testing meets the balance sheet**: liquidity stress tests simulate the run-off of funding under scenarios that credit and market stress tests feed.

### 6.1 The liquidity table

| Model / metric | Framework | Computes | Horizon | Status |
|---|---|---|---|---|
| LCR | Basel III (finalised Jan 2013) | HQLA ÷ 30-day net outflows ≥ 100% | 30 days | Global standard; fully phased since 2019 |
| NSFR | Basel III (finalised Oct 2014) | ASF ÷ RSF ≥ 100% | 1 year | Global standard; EU applied 2021, US 2021 [verify per-jurisdiction dates] |
| Maturity ladder / cashflow mismatch | Internal + supervisory (BCBS 2010 monitoring tools) | Contractual net positions per bucket | Daily–1y+ | Management + supervisory monitoring |
| Non-maturing deposit behaviour model | Internal (statistical) | Core vs volatile deposit split | Rolling | Feeds LCR/NSFR and ALM |
| Funding concentration metrics | Basel III monitoring tools | Top-counterparty/product/currency shares | Point-in-time | Supervisory reporting |
| Intraday liquidity monitoring | BCBS 248 | Settlement flows, buffers, delays | Intraday | Larger banks |
| Liquidity stress tests | CCAR/DFAST, EBA, MAS, internal | Cash shortfall under run scenarios | 30 days–1 year | Pillar 2 / supervisory |

### 6.2 Inside the ratios: HQLA, run-off factors, and the ALM connection

The LCR's apparent simplicity hides a detailed parameterisation that is itself a model landscape:

- **HQLA tiers and haircuts.** Level 1 assets (cash, central-bank reserves, sovereign bonds at the top credit quality) count at 100% of market value with no haircut. Level 2A (high-quality corporate bonds, covered bonds, agency securities) counts at 85% (15% haircut); Level 2B (lower-rated corporate bonds, unencumbered equities, RMBS) at 50–70% (30–50% haircuts), and Level 2 is capped at 40% of total HQLA. The haircuts are the model: they encode the regulator's estimate of how much value the asset loses if it must be sold into a 30-day fire sale.
- **Run-off and inflow factors.** Net cash outflows = outflows − min(inflows, 75% of outflows) (the 75% cap enforces the prudential assumption that you cannot fund your way out of a crisis with new inflows). Outflow factors encode *behavioural assumptions about depositors and lenders under stress*: stable retail deposits run off at 3–5% (depending on deposit-insurance coverage), less-stable retail at 10%, unsecured wholesale from non-financial corporates at 40% (or 20% if fully covered by deposit insurance), financial-institution funding at 100%, and committed-but-undrawn credit and liquidity lines at 5–10% (higher for financial counterparties). These factors are updated by the Committee when the evidence changes — they are, in effect, a *supervisory behavioural model*.
- **The ALM connection.** The LCR and NSFR sit on top of the asset-liability management (ALM) engine: the same cashflow projections that drive gap analysis and interest-rate risk in the banking book (IRRBB — EVE/NII sensitivity under rate shocks, per BCBS 368) also produce the contractual cashflow base for the liquidity ratios, with behavioural overlays (non-maturing deposits, early redemptions) that are genuinely bank-owned models. And the funding-cost side feeds **fund transfer pricing (FTP)** — the internal charge that allocates liquidity cost to business lines, which is how the LCR/NSFR discipline reaches loan pricing. interest_engines_core_banking_guide.md covers the cashflow computation machinery; the point here is that liquidity "models" are a continuum from supervisory ratios (LCR/NSFR) through bank-owned behavioural models (deposit cores, drawdown behaviour on committed lines) to ALM/FTP models — all governed by the same validation discipline as credit models, because a wrong deposit-behaviour assumption is a liquidity crisis in miniature.

### 6.3 A worked LCR example: the 30-day survival question

A concrete LCR computation makes the ratio tangible. Meridian's balance sheet: SGD 6bn of deposits (4bn stable retail at a 5% run-off factor, 2bn wholesale from non-financial corporates at a 40% factor), SGD 1bn of committed-but-undrawn credit lines to corporates (10% drawdown factor), and SGD 0.5bn of other outflows (20% factor). Outflows = 4bn×5% + 2bn×40% + 1bn×10% + 0.5bn×20% = 0.2 + 0.8 + 0.1 + 0.1 = **SGD 1.2bn**. Inflows from maturing assets are capped at 75% of outflows — say SGD 0.3bn counted → net outflows = **SGD 0.9bn**. HQLA stock: SGD 1.0bn of cash and SGS bonds (Level 1, 100%) + SGD 0.5bn of high-grade corporate bonds (Level 2A, 85%) = 1.0 + 0.425 = **SGD 1.425bn**.

LCR = 1.425 / 0.9 = **158%** — comfortably above the 100% minimum (and above the bank's own 130% appetite floor, §2.2). Now stress it: shift 1bn of the retail deposits to "less stable" (10% factor) and 0.5bn of the corporate deposits to financial-institution funding (100% factor): outflows rise by 1bn×5% + 0.5bn×60% = 0.35bn → LCR = 1.425/1.25 = **114%** — still above minimum, but the headroom has halved. That sensitivity — "what does the LCR do if our deposit mix shifts?" — is exactly the question treasury runs daily, and why the LCR engine is not a static ratio calculator but a scenario-capable cashflow platform (financial_risk_compliance_systems_guide.md §6).

---

## 7. Capital Models: Basel I, II, III and the RWA Machine

Capital is the bank's own cushion against unexpected loss, and the Basel accords are the rulebook that says how much. The history is a 35-year arc of ever-more-sophisticated models — and, after 2008, of regulators pulling some of that sophistication back.

**Basel I (1988).** The first accord, prompted by the Herstatt failure (1974) and the Latin American debt crisis. A flat **8% minimum capital ÷ risk-weighted assets (RWA)**, with credit exposures bucketed into five risk weights (0%, 10%, 20%, 50%, 100% — cash/government at 0%, interbank at 20%, residential mortgages at 50%, corporate loans at 100%) — verified in outline. Crude but effective: it created the concept of *risk-weighted* capital that still governs. Basel I's fatal flaw was regulatory arbitrage: risk weights were so coarse that banks could hold the same capital against AAA bonds and junk loans (and off-balance-sheet vehicles, which is exactly what happened pre-2008).

**Basel II (2004).** The three-pillar architecture — verified: **Pillar 1** minimum capital (credit risk via Standardised *or* IRB; market risk via the internal models approach on top of the 1996 market-risk amendment; operational risk via BIA/TSA/AMA); **Pillar 2** supervisory review (the ICAAP — internal capital adequacy assessment process — where banks must prove they hold capital for *all* risks, modelled or not, and supervisors add Pillar 2 add-ons); **Pillar 3** market discipline (mandatory disclosure). Basel II's gift was risk sensitivity — the IRB approach tied capital to measured PD/LGD/EAD — and its curse was *model dependence*: capital was only as good as the models, and 2008 showed models at their worst.

**Basel III (2010, revised 2017).** The crisis response. First wave (December 2010): **quality** — the core measure became **Common Equity Tier 1 (CET1) at 4.5% of RWA** (was effectively ~2%), Tier 1 at 6%, total at 8%, plus a **2.5% capital conservation buffer**, a **countercyclical buffer** (0–2.5%), and a **G-SIB surcharge** (up to 3.5%) — so a global systemically important bank realistically needs ~11.5–13%+ CET1 *including buffers* — plus a **3% leverage ratio** (non-risk-weighted — the first serious check on RWA gamesmanship) and the **LCR/NSFR** liquidity standards (§6). Second wave — the December 2017 "Basel III finalisation" (colloquially Basel IV/3.1): the **72.5% output floor** (internal-model RWA cannot fall below 72.5% of standardised RWA), tighter credit-risk standardised weights, the IRB input floors and restrictions (§3.2), the SMA replacing AMA (§5), and a revised CVA framework.

**How the machine fits together.** RWA = Σ (exposure × risk weight), where the risk weight comes from either the standardised grid or an IRB formula. For IRB corporate exposures the risk weight is a function of PD, LGD, M via the Basel ASRF formula (the Vasicek one-factor model), which embeds a 99.9% confidence, one-year horizon and a supervisor-set asset correlation. Capital = CET1 + AT1 + Tier 2 against RWA. Expected loss (PD × LGD × EAD) is *not* capitalised — it is provisioned (Basel II: compare EL to provisions; shortfall comes out of capital, surplus goes to Tier 2 — verified via the IRB Wikipedia treatment). Capital covers **unexpected loss** — the tail beyond EL. Economic capital (internal, modelled at the bank's own confidence level, often 99.95–99.97%) is the internal parallel to regulatory capital and feeds ICAAP and RAROC (risk-adjusted return on capital), which is how models ultimately drive *pricing and limit decisions* (banking_limits_domain_guide.md).

### 7.1 The capital table

| Regime | Year(s) | Core innovation | Capital requirement | Legacy |
|---|---|---|---|---|
| Basel I | 1988 | Risk-weighted assets; 8% minimum | 8% of RWA | Coarse weights; arbitrage-prone |
| Market-risk amendment | 1996 | VaR internal models for trading book | 99%/10-day VaR × multiplier | First internal-model capital |
| Basel II | 2004 | Three pillars; IRB (F/A); op-risk approaches; ICAAP | PD/LGD/EAD-driven RWA | Model dependence exposed in 2008 |
| Basel 2.5 | 2009 | Stressed VaR, incremental risk charge | SVaR add-ons | Interim patch |
| Basel III | 2010 | CET1 4.5% + buffers; leverage ratio 3%; LCR/NSFR | ~11.5–13%+ CET1 for G-SIBs incl. buffers | Current framework's core |
| Basel III finalisation ("Basel IV"/3.1) | 2017 | Output floor 72.5%; IRB input floors; SMA; CVA rewrite | Floored, less model-dependent RWA | Phasing 2023–2030 by jurisdiction (EU CRR3 2025) [verify: national timelines] |

### 7.2 Capital instruments: what counts, and why it matters to model builders

"Capital" is not one number but a hierarchy of instruments with different loss-absorbing qualities — and the distinction matters because model outputs (RWA, stressed losses) interact with each tier differently:

- **CET1 (Common Equity Tier 1)** — ordinary shares, retained earnings, and disclosed reserves: the *going-concern* cushion that absorbs losses while the bank still operates. Minimum 4.5% of RWA, plus the 2.5% conservation buffer, plus countercyclical and G-SIB add-ons. Everything in this guide that says "capital" most importantly means CET1.
- **AT1 (Additional Tier 1)** — perpetual, loss-absorbing instruments with no maturity date, the most famous being **contingent convertibles (CoCos)**: bonds that convert to equity (or write down) when CET1 breaches a trigger (typically 5.125%). AT1 is *going-concern* capital of a harder kind — it absorbs losses at the point of non-viability. (The 2023 Credit Suisse AT1 wipeout, where CHF 16bn of AT1 was written to zero ahead of shareholders, made the instrument's risk profile famous — and, notably for the models theme, the wipeout was triggered by a supervisory determination of *viability*, not a model output.)
- **Tier 2** — subordinated debt with ≥5-year original maturity: *gone-concern* capital that absorbs losses in resolution, after CET1 and AT1 are exhausted. Total capital = CET1 + AT1 + Tier 2 ≥ 8% of RWA.
- **Deductions and filters.** Goodwill, deferred-tax assets, and other intangible-dependent items are deducted from CET1 — the accounting-vs-regulatory boundary that risk-and-finance reconciliation (financial_risk_compliance_systems_guide.md §11.2) spends its life on.

For the model builder the practical points are: (1) **buffers sit on top of minimums** — a bank that reports "CET1 13%" is above the 4.5% minimum but may be below its fully-loaded requirement once the conservation buffer, countercyclical buffer, G-SIB surcharge, and Pillar 2 add-ons are stacked; the *usable* headroom is what stress tests (CCAR/DFAST, EBA) measure, and what management capital planning optimises; (2) **the leverage ratio (≥3% of total exposure)** is the non-risk-weighted backstop — it means RWA-reducing models can only take you so far, because the leverage denominator ignores risk weights entirely; this is the single biggest structural limit on "model arbitrage" of capital; (3) **the output floor (72.5%)** means internal-model RWA can never go below 72.5% of what the standardised approach would produce — the ultimate expression of the post-2008 thesis that internal models are a *discount*, not a blank cheque.

### 7.3 A worked RWA example: standardised vs IRB on one loan

To see the whole machine in miniature, price the capital on a single SGD 100m, 5-year senior unsecured corporate loan to a mid-rated obligor, PD = 1.5%, LGD = 45% (senior unsecured corporate), EAD = 100m:

- **Standardised approach:** a corporate exposure to an unrated/mid-rated obligor carries a 100% risk weight → RWA = 100m × 100% = 100m → capital at 8% = **SGD 8m** (more with buffers).
- **IRB (F-IRB):** plug PD = 1.5%, LGD = 45% into the ASRF formula (§3.4) with the corporate correlation function (ρ ≈ 0.17 at PD 1.5%) and a maturity adjustment for 5 years (≈ 1.07): the conditional tail default probability comes out near 14%, so K = 45% × (14% − 1.5%) × 1.07 ≈ 6.0% → RWA = 6.0% × 12.5 × 100m = 75m → capital = **SGD 6m** at 8%.
- **With the 72.5% output floor:** the floor caps the discount at the *portfolio* level: total internal-model RWA cannot fall below 72.5% of total standardised RWA. For this loan in isolation, the IRB RWA of 75m already exceeds the floored value (72.5% × 100m = 72.5m), so it is unaffected; the floor binds for the exposures where internal models are most aggressive (e.g., a mortgage book where model RWA might otherwise be 30% of standardised). The mechanism is the same either way: the model discount survives, but it is bounded.

The numbers — 8m standardised, 6m IRB — are the whole history of Basel in one row: Basel I said 8m and stopped; Basel II said "trust your model, pay 6m"; Basel 3.1 says "trust your model, but not below the floor." For a portfolio of thousands of loans, these per-loan differences compound into the billions — which is why IRB approvals, model validation, and the output floor are fought over so fiercely, and why the RWA engine is the most heavily governed calculation in the bank.

---

## 8. Stress Testing: CCAR, EBA, MAS and Scenario Design

Where VaR and PD models ask "what is the distribution?", stress testing asks "what happens *if*?" — it is the discipline that caught what the models missed in 2008, and it has grown from a back-office exercise into a fully-fledged regulatory regime with its own models, data pipelines, and infrastructure (the systems are in financial_risk_compliance_systems_guide.md §3.4 and §10).

**CCAR — the US regime.** The **Comprehensive Capital Analysis and Review** (Federal Reserve, from **2011**; predecessor SCAP 2009, run on the 19 largest BHCs — verified) is the annual test in which large US bank holding companies project their capital ratios under **supervisory scenarios** (baseline, adverse, severely adverse) over **nine quarters**, using their own models under Fed-specified macro paths (unemployment, GDP, equity prices, house prices, etc.). The Fed runs its *own* models in parallel (the "supervisory stress test", DFAST — the Dodd-Frank Act stress tests, mandated by the 2010 Act, running since 2013), and the outcomes set the **stress capital buffer (SCB)** — the capital a bank must hold above its minimums *just for being stress-resilient*. Until 2020 the CCAR also carried a *qualitative* objection power over capital plans (a governance review); the Fed dropped the qualitative objection in 2020 and folded the exercise into the SCB framework. Critically, CCAR is **model-on-model**: the scenarios are macro paths, and every bank's internal credit, market, and operational models must be re-run under them — which is why stress-testing infrastructure (scenario servers, satellite models, data marts) is a multi-hundred-million-dollar line item at large US banks.

**EBA — the EU regime.** The **European Banking Authority's EU-wide stress tests** (biennial since 2011: 2011, 2014, 2016, 2018, 2021, 2023 — verified in outline) use a **bottom-up** design with *common* methodologies, scenarios, and templates developed with the ESRB, ECB, and European Commission: banks compute the scenario impacts themselves within tight rules ("constrained bottom-up"), and the EBA publishes bank-level CET1 impacts. The EBA test feeds the **SREP** (supervisory review and evaluation process) through the ECB for significant institutions. The 2023 round covered ~70 banks with a stress scenario including a severe recession, and the EBA has announced climate-risk integration into the methodology (first for the 2025–2027 rounds [verify: exact 2027 timeline per the EBA announcement]).

**MAS — the Singapore regime.** For the Singapore angle (relevant to this guide's audience): MAS embeds stress testing in its supervisory framework — banks in Singapore must run robust, forward-looking stress tests covering credit, market, liquidity, and operational risk, and MAS conducts its own supervisory and industry-wide stress exercises. MAS's approach follows the BCBS "Principles for sound stress testing practices and supervision" (2009) and is applied through MAS risk-management guidelines and notices (e.g., the credit-risk notices) [verify: the exact current MAS guidelines/notices numbering and the schedule of MAS industry-wide stress tests as of 2026]. Practically, a Singapore bank's stress-testing programme mirrors the global pattern: ICAAP stress tests for Pillar 2, annual supervisory submissions, and internal early-warning scenarios.

**Scenario design is itself a craft.** The three pillars of scenario construction: *historical* scenarios (replay 2008, 1998, 2020), *hypothetical* scenarios (a designed shock to a specific vulnerability — a China slowdown, a rates shock, a cyber outage), and *reverse stress tests* (start from a business-threatening loss and work backwards to the scenario that causes it — a BCBS requirement that catches risks forward-looking scenarios miss). The model content of stress testing is the *satellite models*: macro-to-PD mapping (regressions of default rates on GDP/unemployment), macro-to-LGD overlays, market-risk scenario shocks, and the aggregation logic that turns macro paths into P&L and capital ratios. Stress test results feed capital planning (the CCAR capital plan), contingency funding plans (liquidity), and — increasingly — climate scenarios (transition and physical risk paths).

### 8.1 The stress table

| Regime | Authority | Design | Horizon | Output use |
|---|---|---|---|---|
| SCAP (2009) | US Federal Reserve | Top-down, 19 BHCs | 2 years | Crisis capital recapitalisation |
| CCAR (2011–2020) | US Federal Reserve | Bottom-up (bank models) + Fed models; qualitative review | 9 quarters | Capital plan approval, objections |
| DFAST (2013–) | US Federal Reserve (mandated by Dodd-Frank Act 2010) | Supervisory + company-run | 9 quarters | Public disclosure; SCB |
| SCB framework (2020–) | US Federal Reserve | Supervisory stress test sets stress capital buffer | 9 quarters | Capital requirements (replaces qualitative objection) |
| EBA EU-wide stress test (2011–) | EBA with ESRB/ECB/EC | Constrained bottom-up, common templates | 3 years | Public disclosure; SREP/ECB follow-up |
| MAS supervisory stress tests | MAS | Industry-wide + firm-specific | 1–3 years | Supervisory review; ICAAP expectations |
| ICAAP stress tests | Bank internal (Pillar 2) | Bank-designed scenarios | 1–5 years | Internal capital adequacy; SREP input |
| Reverse stress tests | Bank internal (BCBS 2009) | Loss→scenario inversion | Point-in-time | Early warning, contingency plans |

### 8.2 The satellite model stack: how a macro scenario becomes a capital ratio

The most under-appreciated part of stress testing is the chain of models that turns a macro path into a projected capital ratio. For an architect, this chain *is* the system design problem of stress testing:

1. **Scenario engine.** The regulator (or the bank) publishes paths for the macro variables — GDP, unemployment, equity indices, house prices, rates, credit spreads — for each scenario and each quarter of the horizon. These are the *shocks*; everything downstream is model.
2. **Satellite models (macro→risk).** The bank's PD/LGD/EAD models are re-run under the scenario through *satellite regressions*: default rates modelled as a function of GDP growth and unemployment (a simple, standard form: Δdefault rate = β₀ + β₁·ΔGDP + β₂·Δunemployment + ε, estimated on history and shocked with the scenario path); LGD adjusted for house-price and recovery-market stress; EAD adjusted for utilisation behaviour under stress. These satellites are SR 11-7 models in their own right — they have development, validation, and monitoring just like the base PD models.
3. **P&L and balance-sheet projection.** Credit losses (PD/LGD/EAD × exposure) combine with net interest income (rate paths × the asset-liability book — see interest_engines_core_banking_guide.md), market losses (scenario shocks to the trading book — the FRTB/ES machinery of §4), operational losses (op-risk stress), and fees/expenses to produce the projected P&L; the balance sheet evolves under the scenario (loan growth assumptions, funding behaviour), which determines the capital denominators.
4. **Capital ratio projection.** Starting CET1, minus after-tax losses, plus capital actions (dividends, issuance — constrained by supervisory assumptions about what a stressed bank may actually do), divided by projected RWA (itself re-computed under stressed parameters) = projected CET1 ratio per quarter. The minimum across the horizon is the number the regulator publishes and acts on.
5. **Aggregation and governance.** Results flow through the risk data platform (BCBS 239 lineage from scenario input to ratio output), are reviewed in model-risk governance (each satellite model's assumptions and validation rating documented), and are compared against the regulator's own independent estimates — the "supervisory vs company-run" gap analysis that is a feature of CCAR/DFAST and a standing source of tension between banks and supervisors.

The same stack, with lighter machinery, runs at every ICAAP: a Singapore bank's MAS-driven ICAAP stress tests use the identical logic — macro scenarios → satellite models → P&L/capital projection — at whatever granularity the bank's data supports. Climate stress testing is the newest layer on this stack: transition-risk scenarios (carbon prices, policy shocks) and physical-risk scenarios (weather events) are converted into credit and market losses through the same satellite architecture, which is why the EBA's climate integration and the BCBS's climate-risk guidance are model-governance events, not just policy events.

### 8.3 The CCAR story in detail: from SCAP to the stress capital buffer

The US regime is worth its own close-up because it invented modern supervisory stress testing, and its history is a model-governance case study:

- **SCAP (2009)** — the Supervisory Capital Assessment Program: the first public, top-down stress test of the 19 largest BHCs, run by the Fed with a 2-year adverse scenario at the depths of the crisis. Its output — the capital shortfall each bank had to raise — was the first time a supervisor's *scenario model* directly dictated bank balance sheets. Ten of the 19 banks were required to raise capital.
- **CCAR (2011–2019)** — the annualised successor: banks submit capital plans with their own nine-quarter projections under Fed scenarios (baseline/adverse/severely adverse), *and* the Fed runs its own supervisory models (DFAST, from 2013, mandated by the Dodd–Frank Act) on the same scenarios. The Fed could *object* to a capital plan on quantitative grounds (projected capital ratios below minimums) or qualitative grounds (weak capital planning processes) — the qualitative objection was a governance lever no other regime had.
- **SCB (2020–)** — the Fed retired the qualitative objection and converted the quantitative test into the **Stress Capital Buffer**: each bank's capital requirement is now explicitly "minimums + buffers + its own stress-test-determined SCB," published annually. The CCAR brand lives on as the historical exercise series (CCAR Historical Exercises 2011–2021, per the Fed's archive), but the *model* — supervisory scenarios, satellite models, nine-quarter projection — is now embedded in the SCB framework rather than a standalone approval ritual.

Three design features are worth lifting for any stress-testing programme: (1) **scenario severity is calibrated to history-plus** — the severely-adverse scenario is engineered to be worse than anything in the sample period, so the test is genuinely forward-looking; (2) **company-run vs supervisory-run** — the gap between what the bank's models produce and what the Fed's models produce is itself a supervisory signal about model quality; (3) **the tests are public** — publishing bank-by-bank results creates market discipline that no internal exercise can match. These three features — severity, dual computation, publication — are the pattern the EBA, MAS, and every ICAAP framework have copied in their own style.

---

## 9. Model Risk Management: SR 11-7 and the Validation Discipline

Every model in this guide is itself a source of risk. **Model risk** is the risk of adverse consequences from decisions based on models that are incorrect or misused — from a flawed PD that underprices lending, to a VaR that misses the tail, to a stress model that underestimates a crisis. The governing document is **SR 11-7 / OCC 2011-12, "Supervisory Guidance on Model Risk Management"**, issued jointly by the Federal Reserve Board and the OCC on **April 4, 2011** — verified (the OCC published it as Bulletin 2011-12; it replaced the 2000 Model Validation bulletin). SR 11-7's canonical definition: *a model is a quantitative method, system, or approach that applies statistical, economic, financial, or mathematical theories, techniques, and assumptions to process input data into quantitative estimates*. Anything that converts data into numbers used for decisions — a PD scorecard, a VaR engine, an ECL calculator, a stress-test satellite model, even a pricing formula — is in scope.

SR 11-7's core demands, which every bank's model-risk management (MRM) function now institutionalises:

- **Model inventory.** Every model registered, classified by risk tier (high/medium/low materiality), with an owner, a developer, and a validator. The inventory is the backbone of MRM — you cannot govern what you cannot list.
- **Development and documentation.** Models must be built to documented standards: data rationale, assumptions stated and challenged, conceptual soundness argued, and full documentation that a third party can reconstruct the model from.
- **Independent validation.** The heart of SR 11-7. Validation must be performed by *qualified, independent* parties (not the developers, not the users — a genuine third line), and must cover three elements: (1) **evaluate conceptual soundness** — is the theory right, are the assumptions defensible?; (2) **ongoing monitoring** — is the model performing as expected in production (backtesting, benchmarking)?; (3) **outcomes analysis** — do realised outcomes match model predictions (default rates vs PDs, losses vs VaR)? Validation findings are rated (e.g., satisfactory / needs attention / material weakness) and tracked to remediation.
- **Effective challenge.** The intellectual spine of the whole regime: decisions must withstand genuine adversarial review, from the model owner up to the board. If challenge is ceremonial, the model risk is unmanaged regardless of how good the mathematics is.
- **Governance and use.** Models must be used for the purposes they were validated for; a change of use (e.g., a PD model repurposed for pricing) is itself a model-risk event requiring re-validation. The "use test" (from Basel IRB) and SR 11-7's use requirements are the same idea from two directions.

**The regime has matured and is still moving.** BCBS 239 (2013) added the data-governance half (risk data aggregation and reporting — the data the models eat must be lineage-clean; see financial_risk_compliance_systems_guide.md §2.2). IFRS 9 ECL models and stress-test models inherited the same validation discipline. And the industry press reports that the **OCC issued new model-risk guidance in April 2026** replacing the 2011 letter — a 15-year reign ending; the new guidance reportedly steps back from agentic AI while extending expectations for AI/ML models [verify: full content and status of the 2026 OCC guidance — proposed vs final — as of this writing]. For practitioners: SR 11-7's vocabulary (conceptual soundness, effective challenge, outcomes analysis) is still the lingua franca of every model-validation meeting in the world, and it is the template MAS and other non-US supervisors apply to their own examinations of model risk at internationally active banks.

### 9.1 The MRM table

| Instrument | Year | Scope | Key mechanism |
|---|---|---|---|
| SR 11-7 / OCC 2011-12 | 2011 | Model risk at US banks (Fed + OCC) | Model inventory, independent validation, effective challenge; defines "model" |
| OCC 2000-16 (predecessor) | 2000 | Model validation | Replaced by SR 11-7 |
| BCBS 239 | 2013 | Risk data aggregation & reporting | Data lineage, reconciliation, architecture for risk data |
| Basel IRB "use test" | 2004 (Basel II) | IRB model usage | Models must drive real risk decisions, not just capital |
| IFRS 9 / CECL governance | 2018 (IFRS 9 effective) | ECL model oversight | Model-based provisioning under audit scrutiny |
| MAS model risk expectations | Ongoing | Singapore banks | Applies international MRM expectations via supervision [verify: specific MAS issuance] |
| OCC guidance refresh | 2026 (reported) | Model risk, incl. AI/ML | Reported replacement/update of SR 11-7 letter [verify: status] |
| EU AI Act | 2024 (in force), 2026+ obligations | AI systems incl. credit scoring | High-risk classification; risk-management systems for AI (see ../technology/ai_llm/ai_governance_bias_redteaming_guide.md) |

### 9.2 The three lines of defense and the validation organisation

SR 11-7's independence requirements slot into the bank's wider **three lines of defense** structure, and understanding the slotting is essential for anyone building model systems (the workflow features, user roles, and access controls in a model-governance platform all encode it):

- **First line — the business.** Model owners and developers sit in the business lines (credit, market risk, treasury, finance). They own the model, its data, its use, and the P&L consequences of its use. They are the ones who must satisfy the "use test."
- **Second line — risk management.** The model-risk management (MRM) function — typically reporting to the CRO — sets the standards (model development policy, validation policy, inventory requirements) and *owns the independent validation function*. Validators must be independent of developers: separate reporting lines, no involvement in model development, compensation not tied to model outcomes. This is the "effective challenge" made organisational. In many banks validation sits as a distinct unit *within* second line (sometimes a dedicated model validation team of quants), precisely so the challenge is real.
- **Third line — internal audit.** Audit reviews the MRM framework itself: did validation actually happen, are findings remediated, is the inventory complete? Audit is the check on the checkers.

The working machinery inside this structure: **model inventory** (the master register — metadata: model ID, name, owner, developer, validator, risk tier, status, version, validation rating, findings, next review date); **risk tiering** (high/medium/low materiality — a high-tier model like an IRB PD engine gets full validation, ongoing monitoring, and annual review; a low-tier reporting formula gets light-touch checks); **validation tiers** (full independent validation for new material models; targeted re-validation for changes — a data change, a recalibration, a change of use; ongoing monitoring between validations); and **findings management** (validation findings rated — e.g., satisfactory, satisfactory with conditions, material weakness — with remediation owners and deadlines tracked to closure, because a material weakness on a capital-direct model is a regulatory disclosure event). The systems that support all this — inventory databases, validation workflow, backtesting automation — are the same class of platform covered in financial_risk_compliance_systems_guide.md §10–12; what matters here is that the *data model of the inventory* (who validates what, when, with what findings) is a regulatory artefact in its own right, and it is the first thing an examiner asks for.

### 9.3 Documentation standards: the model package

SR 11-7's practical heart is the **model package** — the documentation set that makes a model independently reconstructable. The canonical contents, which every validation meeting assumes:

1. **The model development document** — business purpose and use, population definition, data description (sources, windows, exclusions, quality issues), methodology and its rationale (why logistic vs alternatives), variable selection, parameter estimates, performance results (in-sample and holdout), and stated limitations.
2. **The data appendix** — definitions (including the default definition), sample construction, data lineage, and the versioned dataset used. An examiner's first test is usually "can I reproduce the development sample from the data warehouse?" — the answer lives here.
3. **The validation report** — the independent assessment: conceptual soundness, data adequacy, outcomes analysis, and the validator's findings with ratings.
4. **The implementation specification** — how the model is coded and deployed: the scoring equations, the interface contracts (inputs, outputs, reason codes), the calculation schedule, and the change history. This is the document the engineers live in, and it must match the development document *exactly* — model implementation drift (the deployed code diverging from the validated design) is a classic finding.
5. **The monitoring plan** — backtest design, stability thresholds, trigger definitions (when does the model enter "for review"?), and the monitoring reports' formats and owners.

The model package is not one document produced once — it is a living artefact versioned alongside the model itself, and "documentation current?" is a standing validation finding category. In systems terms, the package is the *metadata spine* of the model inventory: every artefact linked to the model ID, every version retained, every change logged — the model equivalent of audit-ready source control.

---

## 10. Machine Learning in Risk: The New Model Frontier

The model canon above is largely 1974–2019 mathematics. The frontier is machine learning — and it has arrived in risk with measurable results: gradient-boosted trees (XGBoost, LightGBM) now routinely beat classical logistic-regression scorecards by 5–10 AUC points on default prediction (industry benchmarks, e.g., "XGBoost Beats Merton by 5–10 AUC on Default Prediction" — verified as a published benchmark claim), random forests and neural nets are used for LGD and EAD, ML drives transaction-monitoring and fraud detection at scale (see financial_fraud_detection_at_scale_guide.md in this series), graph analytics catch organised fraud rings, and NLP/LLMs read news, contracts, and surveillance communications (see financial_risk_compliance_systems_guide.md §12 and ../technology/ai_llm/llm_instruction_tuning_guide.md).

Where ML earns its keep in risk:

- **Credit:** ML scorecards and ensemble models for PD at origination and account management; ML for LGD/EAD where recovery data is rich; alternative-data underwriting (bureau + behavioural + external data); early-warning systems flagging deteriorating names from unstructured data.
- **Market risk:** ML for volatility forecasting (GARCH-family hybrids), regime detection, and scenario generation; reinforcement-learning and deep hedgers on the trading side; ML-assisted VaR/ES model risk mitigation via challenger models.
- **Operational risk:** ML on internal loss data for LDA-style severity/frequency estimates; NLP for scenario extraction; anomaly detection for fraud and rogue-trading patterns; KRI prediction.
- **Liquidity:** ML for deposit behavioural modelling (core vs volatile splits) and intraday liquidity forecasting.
- **AML/fraud:** the biggest production win — supervised models on transaction streams, graph-based network analytics, and LLM-assisted SAR narrative drafting (systems in financial_risk_compliance_systems_guide.md §7).

**The governance problem is the story.** Every ML model is still a *model* in the SR 11-7 sense, and the classical validation discipline applies — but ML strains it in five documented ways:

1. **Explainability.** Logistic scorecards are intrinsically interpretable (coefficients, WoE); a 500-tree gradient boost is not. Practice has converged on post-hoc explanation (SHAP values, partial dependence plots, LIME) plus inherently interpretable challenger models — and on regulators accepting "explainability at the level of the decision, not the weights."
2. **Bias and fairness.** Credit models touch protected characteristics indirectly; regulators (and MAS's FEAT principles — Fairness, Ethics, Accountability, Transparency, issued November 2018, with the Veritas industry framework for fairness metrics in credit scoring) demand bias testing and mitigation (see ../technology/ai_llm/ai_governance_bias_redteaming_guide.md).
3. **Overfitting and data drift.** ML models fit the training past and decay as populations and economies move; risk models need monitored drift, periodic retraining governance, and stability safeguards — the model-risk analogue of the model-monitoring discipline in SR 11-7.
4. **Adversarial robustness.** Risk models sit inside the bank's security perimeter with the rest of the estate; models trained on public or contaminated data can be poisoned, and fraudsters actively probe ML detectors (see ../technology/adversarial_ml_attacks_guide.md).
5. **Regulatory acceptance.** IRB approval of ML PD models remains rare and hard-won (supervisors still default to logistic-style interpretability); most ML deployment to date is in *management* models (early warning, monitoring, fraud) rather than *capital-direct* models — though the 2026 OCC guidance refresh reportedly addresses AI/ML model risk explicitly [verify].

**The pragmatic architecture pattern** (increasingly standard): keep the regulatory capital engine classical and interpretable (logistic scorecard → IRB); run ML as challenger, early-warning, and enrichment layers alongside; use ML outputs to *inform* (reviews, overrides, monitoring triggers) rather than *determine* capital; and put every ML model through the full SR 11-7 inventory–validation–monitoring lifecycle. That is the design that gets machine learning into production without failing the next examination.

### 10.1 The ML table

| Risk area | ML model family | Typical use | Governance emphasis |
|---|---|---|---|
| Credit — origination | Gradient boosting / NN scorecards | PD at application, alternative data | Bias testing, explainability (SHAP), challenger logistic model |
| Credit — portfolio | Early-warning systems (gradient boosting + NLP) | Name deterioration flags | Drift monitoring, human-in-the-loop review |
| Credit — provisioning | ML LGD/EAD | Recovery and exposure estimates | Data lineage, outcomes analysis |
| Market risk | ML volatility/regime models | Vol forecasting, scenario generation | Backtesting, challenger to VaR/ES |
| Operational risk | NLP + anomaly detection | Loss-data mining, scenario extraction, fraud patterns | Data quality, false-positive management |
| Liquidity | ML deposit behaviour | Core/volatile splits, intraday forecasts | Behavioural-model validation, stress consistency |
| AML/fraud | Supervised + graph + LLM | Transaction monitoring, network analytics, SAR drafting | Fairness, adversarial robustness, auditability |
| Model risk itself | ML challengers | Benchmarking classical models | Standard SR 11-7 lifecycle |

### 10.2 The ML model lifecycle in risk: MLOps meets SR 11-7

The practical question for every risk-technology team is not "can ML beat logistic regression?" (it often can, on raw discrimination) but "can an ML model survive the SR 11-7 lifecycle?" The production pattern that answers yes has converged across the industry:

- **Development with governance by design.** Data versioning (every training snapshot reproducible and lineage-documented), experiment tracking (every candidate model logged with its hyperparameters, features, and metrics), and a formal development report in the classical style — problem definition, data description, methodology choice, results on holdout, limitations. The ML report is longer, not shorter, than the logistic one: feature engineering must be documented, hyperparameter search must be reproducible, and the chosen model must be argued for against simpler alternatives.
- **Validation, adapted.** The three SR 11-7 elements map onto ML practice: *conceptual soundness* becomes architecture and feature audit (are the features economically sensible and stable? is the target definition right? — the same default-definition discipline as §11); *ongoing monitoring* becomes performance and drift monitoring (population stability, feature drift, PSI/CSI-style indices, calibration tracking); *outcomes analysis* becomes realised-vs-predicted backtesting (the AUC/Gini on the validation sample is a snapshot — the monitored calibration curve is the truth). Explainability artefacts — SHAP summary plots, partial dependence plots, top-feature reason codes per decision — are now standard validation deliverables, because the validator (and the examiner) will ask "why did the model decline this borrower?" and "does the model's behaviour match business logic at the extremes?"
- **Champion–challenger in production.** The approved ML model runs alongside a challenger (often the incumbent logistic scorecard or a simpler ML model); the pair is monitored together, and promotion/rollback decisions are governed by the model risk committee. This is how risk teams get the AUC uplift without betting the franchise on a black box.
- **Retraining as a controlled event.** ML models decay; retraining is a model *change*, which under SR 11-7 triggers re-validation (scoped by materiality — a periodic retraining on the same population with the same features may get targeted validation; a new population or new features gets full validation). The governance question "when does a retrained model become a *new* model?" is answered by change-management policy: material changes to population, features, or methodology ⇒ new inventory entry and full validation.
- **Human oversight and the override economy.** Every ML risk model ships with human-in-the-loop controls: overrides by relationship managers (monitored, because overrides that systematically beat the model reveal model weakness), decision logs, and escalation thresholds. The override rate is itself a model-monitoring metric — a spike in overrides is often the first signal of model drift.

The consistent industry conclusion: ML's risk-model value is real but conditional — conditional on the governance machinery being built *before* the model ships. That is why every serious risk-ML programme I know looks less like a data-science project and more like a controlled software-delivery pipeline with a validation gate: the model registry and the validation workflow are the CI/CD of risk modelling (see also ../technology/ai_llm/ai_governance_bias_redteaming_guide.md for the AI-governance layer, and ../technology/adversarial_ml_attacks_guide.md for the security layer).

### 10.3 Feature engineering and alternative data: where the AUC comes from

The honest secret of ML's uplift over classical models is less the algorithm and more the *features*: machine learning lets risk teams use data that logistic scorecards were never designed for. The modern risk-data feature landscape:

- **Behavioural and transactional data.** Payment histories, account-usage patterns, cashflow volatility, and utilisation dynamics — at daily granularity, not the monthly snapshots of classical scorecards. A corporate borrower's payments behaviour inside the bank's own transaction data (see financial_fraud_detection_at_scale_guide.md for the analytics platform angle) is the single richest untapped signal for SME and mid-corporate PD.
- **Alternative and unstructured data.** News and social sentiment (NLP), legal and registry records (court filings, ownership structures — graph analytics), satellite and geospatial data (inventory and activity proxies for commodity/trade finance), and supply-chain graphs (is a key supplier distressed? — the supply_chain_finance guides in this series touch the same data). Each of these is a *feature factory*: raw source → cleaned, versioned, lineage-tagged feature → model input, with the same BCBS 239 discipline as any risk data.
- **The feature lifecycle.** Features decay like models do: a feature's predictive power must be monitored (drift per feature, missingness, distribution shift), and the feature store — the versioned, shared repository of curated features — is the data-platform counterpart of the model registry. A feature that quietly stopped updating (a data feed that broke) is the classic silent-model-failure: the model still scores, but on stale inputs.
- **The boundary that matters.** Alternative data raises the bias question more sharply than classical data: models trained on behaviour can *proxy* protected characteristics without ever seeing them (the "redlining via postal code" problem), and MAS's FEAT principles and the EU AI Act's high-risk rules for credit scoring both bite here. The governance response is standardised: fairness metrics computed per protected group, bias testing in validation, and documented business rationale for every alternative-data feature — the same discipline as §10.2, applied upstream at the feature layer.

---

## 11. Worked Example: A PD Model Lifecycle at a Singapore Bank

To make the whole guide concrete, here is the full lifecycle of a single, familiar model: an **IRB application scorecard for mid-corporate lending** at a fictional Singapore-headquartered wholesale bank ("Meridian Bank" — think the corporate-lending profile of a Crédit Agricole CIB Singapore franchise: trade finance, commodity finance, structured lending to Asian corporates). The context is deliberately familiar — the same lifecycle runs at every bank with an IRB licence, and the *systems* it touches are those in financial_risk_compliance_systems_guide.md §3 and banking_limits_domain_guide.md.

**The scenario.** Meridian holds a SGD 8bn mid-corporate loan book. Under Basel II/III it uses F-IRB for this portfolio: it must model **PD** itself (supervisor sets LGD/EAD for the capital calculation), so the PD scorecard is capital-direct, pricing-direct (RAROC), and ECL-direct (IFRS 9). The regulator — for a Singapore bank, MAS — requires IRB approval, the use test, and ongoing validation. The stakes: a 50bp error in average PD moves the loan-loss provision and the RWA by tens of millions.

**Stage 1 — Scope and default definition.** The project charter defines the population (mid-corporate obligors, SGD 5m–100m exposure, excluding financial institutions and property developers — which have their own models), the horizon (12-month regulatory PD), and — critically — the **default definition**, harmonised with Basel: 90+ days past due *or* unlikeliness to pay (bankruptcy, distressed restructuring, MAS/supervisory declaration). The default flag is the model's target variable; everything downstream depends on its consistency, so this stage is where data governance (BCBS 239 lineage) is locked in.

**Stage 2 — Data assembly.** Five-plus years of obligor histories from the loan system (see core_banking_processes_guide.md and interest_engines_core_banking_guide.md for where this data lives), financial statements, bureau data (where available for corporates — more relevant for SME sub-segments), and internal watchlist/restructuring records. The developer builds the development sample (~25,000 obligor-years, ~600 defaults), defines the observation window and the performance window, and handles the classic biases: right-censoring (live loans that haven't had time to default), survivorship (written-off loans must stay in the sample), and seasoning (new-vintage underrepresentation).

**Stage 3 — Development.** The model is a **logistic regression scorecard** in the classic mould: candidate characteristics (leverage, interest cover, current ratio, industry, tenor, collateral share, relationship tenure, behavioural delinquency flags) are binned into WoE categories, screened for monotonicity and business logic, and entered into stepwise logistic regression. The final model — typically 8–12 characteristics — is converted to a points score (score = offset + scale × ln(odds)), with the scale set so a doubling of odds costs ~20 points. Performance is assessed on a holdout sample: **AUROC ~0.78** (vs 0.60 for the old expert matrix — the classic "the model beats the judgement matrix" result), plus KS, and calibration curves. A **term-structure component** (Merton-style or cohort-based) converts the 1-year PD into multi-year PDs for IFRS 9 lifetime ECL.

**Stage 4 — Calibration to the target rate.** Raw model probabilities are rescaled so the portfolio-average predicted PD matches the observed long-run default rate (the *target rate* — a supervisory concept: PDs must be calibrated to the through-the-cycle average, not the point-in-time peak or trough). The bank also computes a **downturn adjustment** for conservatism. The final PD grid is bucketed into rating grades (e.g., 1–20) with monotonic PDs — the master scale that the whole bank uses for limits (banking_limits_domain_guide.md), pricing, and reporting.

**Stage 5 — Independent validation (SR 11-7).** The validation unit — separate from the developers, reporting to the CRO — performs the three SR 11-7 elements: (1) *conceptual soundness*: is logistic/WoE appropriate, are the characteristics economically sensible (a characteristic that is statistically strong but economically backwards — e.g., "borrowers with lawsuits are safer" — is rejected or reversed); (2) *ongoing monitoring*: design of the quarterly backtest (predicted vs actual default rates per grade, binomial and Hosmer–Lemeshow tests) and the annual recalibration triggers; (3) *outcomes analysis*: comparison of realised defaults against PD bands. The validator also *benchmarks*: a challenger gradient-boosted model is built to check the logistic model is not leaving obvious signal on the table (the ML challenger gets 5+ AUC points higher — documented and accepted, since the capital-direct model must be interpretable; the ML variant becomes an early-warning overlay per §10). Validation rating: satisfactory with conditions. Findings go into the model-risk inventory.

**Stage 6 — Regulatory submission.** The model, its documentation, validation report, and use-test evidence (the bank must show the scorecard drives limit setting, pricing, and watchlisting — not just capital) are submitted to MAS for IRB approval — the F-IRB application for the mid-corporate PD model. The process involves supervisory dialogue, possibly a challenge to the default definition or the target rate, and takes on the order of a year. [Note: MAS's specific IRB application mechanics and timelines are per-supervisor practice — verify against MAS's current supervisory expectations.]

**Stage 7 — Deployment.** The scorecard is implemented in the origination and limit systems (see banking_limits_domain_guide.md for the limit-consumer side): a mid-corporate borrower's application data is scored, the PD feeds pricing (RAROC: if return on capital < hurdle, the deal is repriced or declined), limit setting, and IFRS 9 ECL at each reporting date. Model outputs are written to the risk data mart with full lineage (BCBS 239), feeding RWA and provision calculations that were previously produced by the legacy expert matrix.

**Stage 8 — Ongoing monitoring and review.** Quarterly: backtest exception reports per grade, population-stability reports (are new borrowers different from the development sample?), characteristic-stability checks, and override monitoring (how often do relationship managers override the score, and do overrides default more than the model predicts?). Annual: recalibration of the target rate; model re-development is triggered by material drift, portfolio change, or a supervisor request — the model enters the inventory's "for review" state, and the cycle begins again.

**The design, in architectural terms.** For a solution architect, the Meridian PD model lifecycle is a spec for a platform, and the interfaces are the interesting part:

- **Data interfaces.** The model consumes four source streams — loan/master data (from the core banking system, via the feeds described in core_banking_processes_guide.md), financial-statement data (external provider + treasury/relationship data), behavioural data (payment history from the payments engine), and macro data (GDP/unemployment for the IFRS 9 overlay and stress satellites). Each stream must arrive *as-of a consistent observation date*, with lineage tags (BCBS 239) and a versioned snapshot per model run — "the model was scored on data as of 31-Mar" must be reproducible forever.
- **Calculation services.** The scorecard runs as a batch service on the risk data mart: input (obligor characteristics) → WoE transforms → logistic score → calibrated PD → rating grade. The service must be *deterministic* (same input, same output, every run — no floating-point or ordering nondeterminism), versioned (a model version tag in every output row), and *explainable* (reason codes — the top contributing characteristics — generated for every decision, for the front office, the customer, and the regulator).
- **Consumption interfaces.** The PD output feeds four consumers: the RWA engine (F-IRB capital via the ASRF formula), the IFRS 9 ECL engine (staging + lifetime PD term structures), the pricing/RAROC layer (deal-level profitability), and the limit system (rating-based credit limits — banking_limits_domain_guide.md). Each consumer has its own data contract, and a model change triggers impact analysis across all four before deployment.
- **Governance workflow.** Around the calculation sits the SR 11-7 workflow: model inventory entry, validation task (with evidence attachments), findings and remediation tracking, approval gates (model risk committee), and the monitoring dashboards (backtest exceptions, stability indices) that feed the quarterly review. The workflow is not a document — it is a system, with roles (developer, validator, approver, monitor), SLAs, and an audit trail.
- **The ML overlay.** In parallel, the ML challenger (gradient boosting) runs the same scoring job with its own versioning and monitoring; its outputs feed the early-warning dashboard, not capital. The architecture keeps the two pipelines separate but co-located — same data platform, separate model services, shared monitoring — which is the production pattern argued for in §10.

That is the design pattern: a deterministic, versioned, explainable scoring service, wrapped in a governed lifecycle, feeding multi-consumer contracts, with an ML challenger alongside. It is, in miniature, the architecture of every risk platform in financial_risk_compliance_systems_guide.md.

**The lessons.** (1) *Data quality is the binding constraint* — every modelling choice downstream is limited by the default flag and the history; Meridian's real work was data archaeology, not statistics. (2) *The model lifecycle is a governance lifecycle* — the validation and monitoring stages consume more effort than development, and that is the SR 11-7 world working as designed. (3) *The use test is the soul of IRB* — a beautiful model that doesn't change decisions is both worthless and non-compliant. (4) *ML complements but does not replace* — the interpretable capital-direct model plus the ML challenger/overlay is the production pattern of the decade. (5) *Models decay* — the 0.78 AUC is a snapshot; without drift monitoring and disciplined rebuilds, every model in this guide silently becomes a liability. That last lesson is the bridge to the summary: models are not artifacts but *processes*, and risk management is the process of keeping them honest.

---

## 12. Summary: The Mathematics of Prudence (One Page)

Risk management models in banking are the mathematics of prudence — the quantified form of a bank's oldest duty: know what you can lose before you lend. The discipline, in one page:

- **One identity rules credit.** EL = PD × LGD × EAD. Credit models are three sub-models (probability of default, loss given default, exposure at default) built with three families: structural (Merton 1974 — equity as an option on assets), reduced-form (Jarrow–Turnbull 1995 — default as an intensity process), and scorecards (logistic regression — the retail/SME workhorse). Basel II's IRB approach (F-IRB: bank models PD; A-IRB: bank models PD/LGD/EAD) made model quality a capital question — and Basel 3.1 (2017) has been pulling internal-model scope back ever since.
- **Market risk is a distribution problem.** VaR reads a quantile (99%, 1- or 10-day) from a loss distribution built three ways — parametric (RiskMetrics 1994, EWMA), historical simulation, Monte Carlo. The crisis taught that quantiles miss the tail, so the FRTB (final 2019) replaced VaR with expected shortfall at 97.5% — the coherent, tail-correct measure — and gated internal models with stressed calibration, non-modellable-factor add-ons, and P&L-attribution tests.
- **Operational risk lost its models.** AMA/LDA (frequency × severity → 99.9% VaR) was removed by Basel III (2017) in favour of the formulaic SMA — the clearest regulatory retreat from internal models in the canon.
- **Liquidity risk got ratios.** Basel III's LCR (30-day HQLA coverage ≥ 100%) and NSFR (one-year stable funding ≥ 100%) turned the lessons of 2008 into binding balance-sheet constraints.
- **Capital is the point of it all.** Basel I (8% of RWA, 1988) → Basel II (three pillars, 2004) → Basel III (CET1 4.5% + buffers, leverage ratio, LCR/NSFR, 2010–2017, with the 72.5% output floor). Capital covers unexpected loss; expected loss is provisioned.
- **Stress testing is the model of last resort.** CCAR (US, 2011–), DFAST, the EBA's EU-wide tests, and MAS's supervisory expectations run the whole model stack under adverse scenarios — because every model's past is a bad guide to the next crisis.
- **Models need models.** SR 11-7 / OCC 2011-12 made model risk a managed risk: inventory, independent validation (conceptual soundness, ongoing monitoring, outcomes analysis), effective challenge. Machine learning is now being absorbed into that discipline — explainability, bias, drift, and adversarial robustness are the new validation agenda, and the interpretable-capital-model-plus-ML-overlay pattern is how it ships.

The through-line of 50 years of risk modelling: every generation of models is built to fix the last crisis, every generation of models is gamed and outgrown, and the response is not less mathematics but better-governed mathematics. That is why the field's real product is not a number — a PD, a VaR, a capital ratio — but a *process* that keeps numbers honest: the mathematics of prudence.

---

## 13. Glossary

- **Risk management** — the discipline of identifying, measuring, monitoring, and controlling the risks a bank takes; in this guide, the quantitative core of that discipline.
- **Credit risk** — risk of loss from an obligor's or counterparty's failure to meet obligations (Basel II definition).
- **Market risk** — risk of loss from adverse movements in market prices, rates, volatilities, and spreads.
- **Operational risk** — risk of loss from inadequate or failed internal processes, people, or systems, or external events; includes legal risk (Basel II definition).
- **Liquidity risk** — risk of inability to meet obligations as they fall due without unacceptable losses (funding and market liquidity faces).
- **PD (probability of default)** — likelihood an obligor defaults within a horizon (regulatory: 12 months), given a defined default event (90+ days past due or unlikeliness to pay).
- **LGD (loss given default)** — share of exposure lost at default after recoveries and costs; LGD = 1 − recovery rate.
- **EAD (exposure at default)** — amount outstanding at default, including drawn-down of undrawn commitments; for derivatives, potential future exposure.
- **EL (expected loss)** — PD × LGD × EAD; the statistically expected loss, provisioned rather than capitalised.
- **Merton (structural) model** — Merton (1974): firm equity as a European call option on firm assets; default when asset value falls below debt at maturity; yields distance-to-default and PD.
- **Structural model** — credit model family deriving default from firm-value dynamics (Merton, Black–Cox, Longstaff–Schwartz, Leland–Toft).
- **Reduced-form model** — credit model family modelling default as an exogenous intensity process, calibrated to market spreads (Jarrow–Turnbull 1995, Duffie–Singleton 1999).
- **Jarrow–Turnbull model** — Jarrow & Turnbull (1995), *Journal of Finance* 50(1): default as first jump of a Poisson process with intensity λ; survival probability exp(−∫λ).
- **Scorecard** — logistic-regression credit model (application/behavioural/collection) mapping characteristics to a score; higher score ⇒ lower PD.
- **Logistic regression** — PD(x) = 1/(1+e^−(β₀+βx)); the canonical interpretable classification model behind scorecards.
- **IRB (internal ratings-based approach)** — Basel II approach letting banks use internal models for credit capital, subject to approval; Foundation (bank models PD; supervisory LGD/EAD) vs Advanced (bank models PD, LGD, EAD).
- **Basel** — the Basel Committee on Banking Supervision's capital framework: Basel I (1988), Basel II (2004, three pillars), Basel III (2010–2017), Basel 3.1/"IV" finalisation (2017).
- **VaR (value at risk)** — the loss not exceeded at confidence α over horizon h; the α-quantile of the loss distribution (99%, 1-day/10-day conventions).
- **RiskMetrics** — J.P. Morgan's 1994 published VaR methodology: parametric variance–covariance with EWMA volatility (λ = 0.94 daily).
- **Parametric (variance–covariance) VaR** — VaR from assumed distribution and estimated covariance matrix; closed-form.
- **Historical simulation** — VaR from applying historical risk-factor moves to the current portfolio; empirical, assumption-light.
- **Monte Carlo simulation** — VaR from thousands of simulated risk-factor scenarios with full revaluation; handles non-linearity.
- **ES (expected shortfall)** — E[L | L > VaR_α], the average loss beyond the VaR quantile; coherent (subadditive) tail measure.
- **FRTB (Fundamental Review of the Trading Book)** — Basel's market-risk reform (final 2019): ES at 97.5% replaces VaR, stressed calibration, non-modellable-factor add-ons, new standardised approach, P&L-attribution gating; EU application via CRR3 from 2025.
- **AMA (advanced measurement approach)** — Basel II's internal-model approach for operational risk; the LDA is its canonical implementation. Removed by Basel III (2017).
- **LDA (loss distribution approach)** — op-risk model: frequency × severity per unit of measure, convolved to an annual loss distribution; capital = 99.9% VaR.
- **SMA (standardised measurement approach)** — Basel III's formulaic op-risk capital: Business Indicator × marginal coefficients + loss component.
- **LCR (liquidity coverage ratio)** — Basel III: HQLA ÷ 30-day net cash outflows ≥ 100%.
- **NSFR (net stable funding ratio)** — Basel III: available stable funding ÷ required stable funding ≥ 100% over one year.
- **CCAR (Comprehensive Capital Analysis and Review)** — the Federal Reserve's annual stress test of large US bank holding companies (2011–; with SCAP 2009 predecessor and DFAST since 2013).
- **EBA (European Banking Authority)** — EU authority running biennial EU-wide stress tests with common methodology and templates.
- **MAS (Monetary Authority of Singapore)** — Singapore's central bank and financial regulator; supervisory stress-testing expectations and IRB approvals for Singapore banks.
- **Stress test** — projection of losses/capital under adverse scenarios (historical, hypothetical, reverse), at the heart of CCAR/DFAST, EBA, MAS, and ICAAP.
- **SR 11-7 (OCC 2011-12)** — the 2011 Federal Reserve/OCC supervisory guidance on model risk management: inventory, independent validation, effective challenge. (Reported replacement guidance from the OCC in 2026 [verify].)
- **Model risk** — risk of adverse consequences from decisions based on models that are incorrect or misused (SR 11-7).
- **ML (machine learning)** — data-driven model families (gradient boosting, random forests, neural networks, NLP/LLMs) increasingly applied in risk; governed under the classical validation discipline extended for explainability, bias, drift, and adversarial robustness.
- **Expected loss (EL)** — see EL above; the provisioning-relevant component of credit loss (IFRS 9 ECL = 12-month or lifetime PD × LGD × EAD).

*End of guide. Companion reading: financial_risk_compliance_systems_guide.md (the systems that run all of this), banking_limits_domain_guide.md (where model outputs become limits), asset_backed_trading_guide.md (securitisation risk), murex_mx3_platform_guide.md (market-risk modules), ../technology/adversarial_ml_attacks_guide.md and ../technology/ai_llm/ai_governance_bias_redteaming_guide.md (the ML-risk-model security and governance frontier).*

