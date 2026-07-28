# Financial Risk & Compliance Systems: A Comprehensive Guide

## Table of Contents

1. [Introduction: What Are Financial Risk & Compliance Systems?](#1-introduction-what-are-financial-risk--compliance-systems)
2. [Regulatory Context and Key Regulations](#2-regulatory-context-and-key-regulations)
   - 2.1 [Basel III / Basel IV — Capital Adequacy Framework](#21-basel-iii--basel-iv--capital-adequacy-framework)
   - 2.2 [BCBS 239 — Risk Data Aggregation and Risk Reporting](#22-bcbs-239--risk-data-aggregation-and-risk-reporting)
   - 2.3 [Local Regulations: MAS (Singapore)](#23-local-regulations-mas-singapore)
   - 2.4 [European Regulations](#24-european-regulations)
   - 2.5 [US Regulations](#25-us-regulations)
   - 2.6 [Cross-Border Frameworks](#26-cross-border-frameworks)
3. [Credit Risk Systems](#3-credit-risk-systems)
   - 3.1 [Credit Risk Measurement Fundamentals](#31-credit-risk-measurement-fundamentals)
   - 3.2 [Standardized vs. Internal Ratings-Based Approaches](#32-standardized-vs-internal-ratings-based-approaches)
   - 3.3 [Credit Risk Systems Landscape](#33-credit-risk-systems-landscape)
   - 3.4 [Stress Testing for Credit Risk](#34-stress-testing-for-credit-risk)
   - 3.5 [Expected Credit Loss (ECL) — IFRS 9 / CECL](#35-expected-credit-loss-ecl--ifrs-9--cecl)
4. [Market Risk Systems](#4-market-risk-systems)
   - 4.1 [Market Risk Measurement Fundamentals](#41-market-risk-measurement-fundamentals)
   - 4.2 [VaR Methodologies](#42-var-methodologies)
   - 4.3 [Expected Shortfall and Stress VaR](#43-expected-shortfall-and-stress-var)
   - 4.4 [Basel FRTB — Fundamental Review of the Trading Book](#44-basel-frtb--fundamental-review-of-the-trading-book)
   - 4.5 [Market Risk Systems and Data](#45-market-risk-systems-and-data)
5. [Operational Risk Systems](#5-operational-risk-systems)
   - 5.1 [Operational Risk Definition and Basel Approaches](#51-operational-risk-definition-and-basel-approaches)
   - 5.2 [The Standardised Measurement Approach (SMA)](#52-the-standardised-measurement-approach-sma)
   - 5.3 [Operational Risk Systems and Data](#53-operational-risk-systems-and-data)
   - 5.4 [Banking-Specific OpRisk Types](#54-banking-specific-oprisk-types)
6. [Liquidity Risk and Treasury Systems](#6-liquidity-risk-and-treasury-systems)
   - 6.1 [Liquidity Risk and Basel Metrics](#61-liquidity-risk-and-basel-metrics)
   - 6.2 [ALM, FTP, and Liquidity Management](#62-alm-ftp-and-liquidity-management)
   - 6.3 [Treasury Systems Landscape](#63-treasury-systems-landscape)
7. [AML and Financial Crime Compliance](#7-aml-and-financial-crime-compliance)
   - 7.1 [AML/CFT Regulatory Framework](#71-amlcft-regulatory-framework)
   - 7.2 [Customer Due Diligence and KYC](#72-customer-due-diligence-and-kyc)
   - 7.3 [Transaction Monitoring](#73-transaction-monitoring)
   - 7.4 [Sanctions Screening](#74-sanctions-screening)
   - 7.5 [AML Systems Landscape](#75-aml-systems-landscape)
8. [Trade Surveillance and Conduct Risk](#8-trade-surveillance-and-conduct-risk)
   - 8.1 [Trade Surveillance Scope and Regulations](#81-trade-surveillance-scope-and-regulations)
   - 8.2 [Market Abuse Typologies](#82-market-abuse-typologies)
   - 8.3 [Surveillance Systems and Capabilities](#83-surveillance-systems-and-capabilities)
   - 8.4 [Conduct Risk and Behavioral Analytics](#84-conduct-risk-and-behavioral-analytics)
9. [Regulatory Reporting Systems](#9-regulatory-reporting-systems)
   - 9.1 [Regulatory Returns Landscape](#91-regulatory-returns-landscape)
   - 9.2 [Data Lineage and Governance (BCBS 239)](#92-data-lineage-and-governance-bcbs-239)
   - 9.3 [Regulatory Reporting Systems](#93-regulatory-reporting-systems)
   - 9.4 [The Regulatory Hub Concept](#94-the-regulatory-hub-concept)
10. [Technology Architecture Patterns](#10-technology-architecture-patterns)
    - 10.1 [Integration Patterns and System Topology](#101-integration-patterns-and-system-topology)
    - 10.2 [System Classification Map](#102-system-classification-map)
    - 10.3 [Data Integration and Challenges](#103-data-integration-and-challenges)
11. [Cloud Adoption and Data Management](#11-cloud-adoption-and-data-management)
    - 11.1 [Cloud Adoption in Risk and Compliance](#111-cloud-adoption-in-risk-and-compliance)
    - 11.2 [Risk and Finance Data Integration](#112-risk-and-finance-data-integration)
12. [Advanced Analytics and AI/ML](#12-advanced-analytics-and-aiml)
    - 12.1 [Machine Learning Use Cases](#121-machine-learning-use-cases)
    - 12.2 [NLP and Graph Analytics for Compliance](#122-nlp-and-graph-analytics-for-compliance)
    - 12.3 [LLM Applications in Compliance](#123-llm-applications-in-compliance)
    - 12.4 [AI Governance and Model Risk Management](#124-ai-governance-and-model-risk-management)
13. [Future Trends](#13-future-trends)
    - 13.1 [RegTech and SupTech](#131-regtech-and-suptech)
    - 13.2 [ESG Risk Integration](#132-esg-risk-integration)
    - 13.3 [Real-Time Risk](#133-real-time-risk)
    - 13.4 [Convergence of Risk, Finance, and Treasury](#134-convergence-of-risk-finance-and-treasury)
    - 13.5 [Modern Data Architecture Transitions](#135-modern-data-architecture-transitions)
14. [References and Further Reading](#14-references-and-further-reading)

---

## 1. Introduction: What Are Financial Risk & Compliance Systems?

Financial risk and compliance systems are the integrated suite of software platforms, data pipelines, analytics engines, and reporting tools that financial institutions use to measure, monitor, manage, and report on financial risks — credit risk, market risk, operational risk, and liquidity risk — and to comply with a dense web of regulatory requirements covering capital adequacy, anti-money laundering (AML), know-your-customer (KYC), trade surveillance, conduct risk, and regulatory reporting.

These systems are critical infrastructure in modern banking. Unlike general-purpose enterprise software, risk and compliance systems must handle extreme data volumes (millions of trades, positions, and transactions daily), complex calculations (Monte Carlo simulations, credit valuation adjustments, expected shortfall computations), strict timeliness requirements (intraday margin calls, T+0 trade reporting), and unforgiving auditability standards (full data lineage, versioned regulatory submissions, documented model governance). Failures can result in regulatory penalties (fines running into billions of dollars), increased capital charges, reputational damage, and in extreme cases, systemic risk — as illustrated by the 2008 financial crisis where risk systems failed to aggregate and report exposures across counterparties (AIG, Lehman) in real time.

For a Singapore-based bank operating across Asia-Pacific and Europe — such as Crédit Agricole CIB — the compliance burden is compounded by the need to satisfy multiple regulatory regimes simultaneously: MAS (Singapore), HKMA (Hong Kong), PRA/FCA (UK), ACPR/AMF (France/EU), and local regulators across Asia. Each jurisdiction has unique reporting templates, taxonomy differences, submission timelines, and language requirements, making multi-jurisdiction regulatory reporting one of the most complex technology challenges in banking.

The bank's risk and compliance technology estate typically comprises 20–50 distinct systems, spanning front-office risk analytics, middle-office risk management, back-office operations, and enterprise-wide compliance. These systems are interconnected via complex data pipelines spanning batch, real-time, and file-based integration. The discipline of Risk and Finance Convergence (R&F) has emerged as a major architectural initiative at most large banks, aiming to create a unified data foundation that serves both regulatory reporting and risk management. At many European and Asian banks, the Convergence programme is the single largest technology transformation initiative, often surpassing core banking modernisation in scope and budget.

---

## 2. Regulatory Context and Key Regulations

### 2.1 Basel III / Basel IV — Capital Adequacy Framework

The Basel Accords, published by the Basel Committee on Banking Supervision (BCBS), form the global regulatory standard for bank capital adequacy. Basel III (finalised post-2008 financial crisis) and Basel IV (the 2017 finalisation of Basel III reforms) rest on three pillars:

**Pillar 1 — Minimum Capital Requirements**

Pillar 1 prescribes the minimum amount of capital a bank must hold against three risk types:

- **Credit Risk**: Calculated via the Standardised Approach (SA) or Internal Ratings-Based (IRB) approach. The SA uses regulator-set risk weights by asset class; the IRB approach allows banks to use internal estimates of probability of default (PD), loss given default (LGD), and exposure at default (EAD). Foundation IRB (F-IRB) lets banks estimate PD only, while Advanced IRB (A-IRB) allows estimation of all parameters. Under Basel IV, the IRB approach is restricted — certain asset classes (e.g., large corporate exposures, bank exposures, and equity exposures) lose IRB eligibility, and an "input floor" constrains model parameters.

- **Market Risk**: Governed by the Fundamental Review of the Trading Book (FRTB), which replaces Value-at-Risk (VaR) with Expected Shortfall (ES) at a 97.5% confidence level, introduces desk-level approval, and mandates a strict boundary between the banking book and trading book.

- **Operational Risk**: The Standardised Measurement Approach (SMA) replaces all previous approaches (BIA, TSA, AMA). SMA calculates capital as a Business Indicator Component (BIC) multiplied by an Internal Loss Multiplier (ILM) derived from historical operational loss experience.

Additional Pillar 1 requirements include:
- **Credit Valuation Adjustment (CVA)** capital charge for OTC derivatives counterparty credit risk
- **Leverage Ratio**: Tier 1 capital / total exposure ≥ 3%
- **Liquidity Coverage Ratio (LCR)**: High-Quality Liquid Assets (HQLA) / total net cash outflows over 30 days ≥ 100%
- **Net Stable Funding Ratio (NSFR)**: Available Stable Funding / Required Stable Funding ≥ 100% over a one-year horizon

**Pillar 2 — Supervisory Review**

Pillar 2 requires banks to conduct their own Internal Capital Adequacy Assessment Process (ICAAP) and internal liquidity adequacy assessment (ILAAP). Supervisors review these and may impose additional capital charges (Pillar 2A for current risks, Pillar 2B for stress testing). Stress testing is a core Pillar 2 tool — banks run severe but plausible scenarios to assess capital adequacy under adverse conditions.

**Pillar 3 — Market Discipline**

Pillar 3 mandates public disclosure of risk exposures, risk-weighted assets (RWA), capital composition, liquidity metrics, and leverage ratios. Disclosures follow standardised templates to enable comparability across institutions. Capital conservation buffers (2.5% of RWA), countercyclical buffers (0–2.5%), and systemic risk buffers (for global systemically important banks, G-SIBs) are additional layers above the minimum.

### 2.2 BCBS 239 — Risk Data Aggregation and Risk Reporting

BCBS 239 ("Principles for Effective Risk Data Aggregation and Risk Reporting") is one of the most impactful regulations for risk technology. Its 14 principles require:

- **Data governance and architecture** (Principles 1–3): Banks must establish robust data governance frameworks, data taxonomies, and data architecture that supports risk aggregation.
- **Accuracy and integrity** (Principles 4–5): Risk data must be accurate, complete, and reconciled to source systems. Data quality must be measured and certified.
- **Timeliness** (Principle 6): Risk data must be aggregated within prescribed timeframes — intraday for certain exposures, T+1 for most regulatory reporting.
- **Adaptability** (Principle 7): Banks must maintain the ability to produce ad-hoc risk reports under stressed conditions.
- **Risk reporting** (Principles 8–10): Reports must be accurate, comprehensive, clear, and relevant to decision-makers.
- **Supervisory review and implementation** (Principles 11–14): Banks must demonstrate risk data aggregation capabilities to supervisors; non-compliance can result in capital add-ons.

BCBS 239 has been a primary driver of risk data modernisation. Many banks have invested heavily in data lineage tools, data quality dashboards, and enterprise data warehouses to meet its requirements. The FSB (Financial Stability Board) has repeatedly pushed back compliance deadlines for lagging banks.

### 2.3 Local Regulations: MAS (Singapore)

For a Singapore-based institution, the Monetary Authority of Singapore (MAS) regulations are paramount:

- **MAS Notice 637** (Risk Management): Sets requirements for credit risk, market risk, liquidity risk, and operational risk management. Mandates risk limit frameworks, stress testing, and independent risk oversight.
- **MAS Notice 612** (AML/CFT): Comprehensive AML/CFT requirements covering customer due diligence, transaction monitoring, suspicious transaction reporting (STR), name screening, and record-keeping. Requires risk-based approach to AML.
- **MAS Technology Risk Management (TRM) Guidelines**: Addresses technology resilience, cybersecurity, cloud outsourcing, and business continuity. Requires notification of significant IT incidents.
- **MAS Notice 653** (Outsourcing): Governs outsourcing arrangements, including cloud outsourcing, requiring risk assessment, notification, and oversight of material service providers.

### 2.4 European Regulations

- **CRD IV / CRD V** (Capital Requirements Directive): Implements Basel III/IV in EU law, covering capital adequacy, governance, and supervision.
- **EMIR** (European Market Infrastructure Regulation): Mandates central clearing and trade reporting for OTC derivatives.
- **MiFID II / MiFIR** (Markets in Financial Instruments Directive): Governs trading venues, algorithmic trading, best execution, transaction reporting, and trade transparency.
- **MAR** (Market Abuse Regulation): Prohibits insider dealing and market manipulation, requires suspicious transaction and order reports (STORs).
- **SFTR** (Securities Financing Transactions Regulation): Mandates reporting of securities lending, repo, and margin lending transactions.
- **CSDR** (Central Securities Depositories Regulation): Settlement discipline and CSD requirements.
- **GDPR** (General Data Protection Regulation): Data privacy requirements that constrain how personal data is used in risk and compliance processes.
- **DORA** (Digital Operational Resilience Act): ICT risk management, incident reporting, digital operational resilience testing, and third-party risk management.

### 2.5 US Regulations

- **CCAR / DFAST** (Comprehensive Capital Analysis and Review / Dodd-Frank Act Stress Testing): The Federal Reserve's annual stress testing exercise for large banks (≥$250B assets). Banks project capital ratios over a 9-quarter horizon under severely adverse, adverse, and baseline scenarios. Results determine capital distribution capacity.
- **FR Y-14** (FR Y-14A/M/Q): Quarterly regulatory reports collecting granular data on capital, risk, and PPNR (Pre-Provision Net Revenue).
- **FR Y-9C**: Consolidated financial statements for bank holding companies (US equivalent of FINREP).
- **Call Reports** (FFIEC 031/041): Quarterly condition and income reports for US banks.
- **FINRA Rules**: Conduct rules for broker-dealers covering trade reporting, supervision, and market integrity.
- **SEC Rules**: Includes Rule 15c3-1 (Net Capital), Rule 17a-3/4 (Recordkeeping), and market access rules.
- **OFAC Sanctions**: US sanctions programs enforced by the Office of Foreign Assets Control.

### 2.6 Cross-Border Frameworks

- **FATCA** (Foreign Account Tax Compliance Act): US law requiring foreign financial institutions to report US account holders.
- **CRS** (Common Reporting Standard): OECD standard for automatic exchange of financial account information between tax authorities.
- **Wolfsberg Group**: Industry guidance on correspondent banking due diligence, AML, and trade finance.
- **FATF Recommendations**: The Financial Action Task Force sets global AML/CFT standards adopted by 200+ jurisdictions.

---

## 3. Credit Risk Systems

### 3.1 Credit Risk Measurement Fundamentals

Credit risk — the risk of loss from a borrower's failure to meet contractual obligations — is quantified through four core parameters:

| Parameter | Description | Formula / Notes |
|-----------|-------------|-------------|
| **PD** | Probability of Default | Likelihood that a counterparty defaults within a given horizon (typically 1 year) |
| **LGD** | Loss Given Default | Percentage of exposure lost given default (1 — recovery rate) |
| **EAD** | Exposure at Default | Outstanding exposure at the time of default |
| **EL** | Expected Loss | EL = PD × LGD × EAD |
| **RWA** | Risk-Weighted Assets | RWA = 12.5 × EAD × PD × LGD risk-weight (Basel formula) |
| **CVA** | Credit Valuation Adjustment | Market value of counterparty credit risk in derivatives |
| **DVA** | Debit Valuation Adjustment | Own credit risk adjustment on liability valuations |

Expected loss (EL) is the loss the bank expects on average; it is provisioned for and affects P&L. Unexpected loss (UL), representing the volatility of losses around EL, is the basis for economic and regulatory capital.

### 3.2 Standardized vs. Internal Ratings-Based Approaches

**Standardized Approach (SA):**
- Regulator-set risk weights by asset class (sovereign, bank, corporate, retail, etc.)
- Simpler implementation — requires external credit ratings or fixed percentages
- No internal models required; lower data demands
- Under Basel IV, SA risk weights have become more granular and risk-sensitive

**Internal Ratings-Based (IRB) Approach:**
- **Foundation IRB (F-IRB)**: Bank estimates PD; supervisor provides LGD (45% for senior, 75% for subordinated) and EAD
- **Advanced IRB (A-IRB)**: Bank estimates PD, LGD, and EAD using internal models
- Requires extensive historical data (5–7+ years), model validation, and supervisory approval
- Under Basel IV: IRB constrained — loss given default floor at 25% for senior corporate exposures, parameter floors applied, and certain exposures removed from IRB eligibility

### 3.3 Credit Risk Systems Landscape

**Core Credit Risk Engines:**
- **Algorithmics** (now IBM): Multi-asset class risk analytics for credit risk, market risk, and ALM
- **Moody's Analytics RiskFrontier**: Economic capital, RWA calculation, portfolio credit risk, IFRS 9/CECL
- **SAS Risk Management**: Credit risk modelling, stress testing, IFRS 9, Basel RWA calculation
- **FICO**: Credit scoring, origination decisioning, credit risk management

**Supporting Systems:**
- **Loan Origination Systems**: nCino, Finastra Loan IQ, FIS ACBS, Moody's CreditLens
- **Credit Scoring Engines**: FICO, Experian, Equifax, VantageScore, custom internal scorecards using gradient boosting or logistic regression
- **Limit Management Systems**: Track and enforce counterparty credit limits across the institution in real time
- **Collateral Management Systems**: ACMS (Finastra), Calypso Collateral, CloudMargin, triResolve — manage margin calls, collateral optimisation, and segregation
- **Credit Derivatives Processing**: DTCC DerivServ, MarkitWire, TriOptima

**Data Requirements:**
Credit risk systems consume large volumes of data: historical default and recovery rates by asset class and geography, exposure data at counterparty and facility level, credit ratings (internal and external from S&P, Moody's, Fitch), macroeconomic variables (GDP, unemployment, interest rates, property prices), and market data for CVA/DVA calculations.

### 3.4 Stress Testing for Credit Risk

Credit stress testing involves:

1. **Scenario Generation**: Adverse macroeconomic scenarios (recession, interest rate shock, sector-specific downturn). Under CCAR, banks receive Fed-defined scenarios; under ICAAP, banks design their own.
2. **Portfolio Impact Analysis**: Projecting PD, LGD, and EAD under stressed conditions using satellite models linking macro variables to credit parameters.
3. **Reverse Stress Testing**: Starting from a capital breach point and working backward to identify the scenario that would cause it — used to identify hidden vulnerabilities.
4. **Concentration Risk Analysis**: Single-name exposure, sectoral, geographical, and name concentration.

### 3.5 Expected Credit Loss (ECL) — IFRS 9 / CECL

The transition from incurred loss (IAS 39) to expected loss (IFRS 9) accounting was one of the largest data and system change programmes in banking history.

**IFRS 9 (International Financial Reporting Standard 9):**
- Applies to all financial assets measured at amortised cost or fair value through OCI
- **Three-stage model**:
  - **Stage 1** (Performing): 12-month ECL recognised; interest revenue on gross carrying amount
  - **Stage 2** (Underperforming — significant increase in credit risk, SICR): Lifetime ECL recognised; interest revenue on gross carrying amount
  - **Stage 3** (Non-performing / credit-impaired): Lifetime ECL recognised; interest revenue on net carrying amount (after ECL)
- Requires forward-looking information, macroeconomic scenarios (at least three: base, upside, downside), and probability-weighting
- SICR assessment uses quantitative criteria (PD deterioration thresholds) and qualitative criteria (forbearance, 30+ days past due, watchlist status)

**US CECL (Current Expected Credit Loss):**
- ASC 326 issued by FASB
- Lifetime ECL for all assets at origination (no Stage 1/2 distinction)
- Broader than IFRS 9 — applies to all financial assets, including held-to-maturity debt securities, net investments in leases, and off-balance-sheet credit exposures
- Requires historical loss data, macroeconomic forecasts, probability-weighted scenarios (or a single best estimate), vintage analysis, and model governance

**ECL Data and Systems:**
- **Data requirements**: Historical loss data (default, recovery, prepayment), macroeconomic forecasts (GDP, unemployment, interest rates, house prices, FX), probability-weighted scenario parameters, vintage/snapshot data, product and behavioural characteristics
- **Model governance**: SR 11-7 framework for model validation, outcomes analysis, independent review
- **Key systems**: Moody's ImpairmentStudio, SAS Expected Credit Loss, Oracle Financial Services ECL, SAP, and cloud-based solutions
- **Integration**: ECL systems feed accounting entries (journal vouchers) to the General Ledger (GL) — dual-capture for Stage transitions, scenario changes, and model updates

---

## 4. Market Risk Systems

### 4.1 Market Risk Measurement Fundamentals

Market risk is the risk of losses in on- and off-balance-sheet positions arising from movements in market prices. The four standard market risk classes are:

- **Interest Rate Risk**: Yield curves, basis risk, optionality
- **FX Risk**: Spot and forward foreign exchange rates
- **Equity Risk**: Equity prices and indices
- **Commodity Risk**: Energy, metals, agricultural commodities

### 4.2 VaR Methodologies

Value-at-Risk (VaR) answers: "What is the maximum loss I can expect over N days with X% confidence?"

Three standard methodologies:

**1. Parametric (Variance-Covariance):**
- Assumes normal distribution of returns
- VaR = portfolio standard deviation × z-score × √(holding period)
- Fast to compute, but fails to capture fat tails and non-linear risks (options)

**2. Historical Simulation:**
- Uses actual historical returns over a lookback window (e.g., 250–500 days)
- No distribution assumption — captures actual correlations and fat tails
- Slow to react to changing volatility (equal-weighting issue)
- Often enhanced with filtered historical simulation (GARCH-based scaling)

**3. Monte Carlo Simulation:**
- Generates thousands/millions of random scenarios from assumed distributions
- Most flexible — handles non-linear instruments (options, structured products)
- Computationally intensive (GPU clusters and grid computing common)
- Allows user-defined risk factor distributions and correlations

### 4.3 Expected Shortfall and Stress VaR

**Expected Shortfall (ES / CVaR):**
- Average loss beyond the VaR threshold (e.g., average of worst 2.5% of outcomes)
- Coherent risk measure — satisfies sub-additivity (diversification benefit)
- More sensitive to tail risk than VaR
- Basel FRTB mandates ES at 97.5% (equivalent conservatism to VaR at 99%)

**Stress VaR:**
- VaR calculated under a historical stress period (e.g., 2008 financial crisis, 2020 COVID)
- Captures risks that are invisible in benign market conditions

**Incremental Risk Capital and CRM:**
- **Incremental Risk Capital (IRC)**: Captures default and migration risk for positions held in the trading book over a one-year liquidity horizon
- **Comprehensive Risk Measure (CRM)**: Applied to correlation trading portfolios (CDO tranches, nth-to-default swaps), capturing both market risk and default risk

### 4.4 Basel FRTB — Fundamental Review of the Trading Book

FRTB represents the most significant overhaul of market risk capital since the 1996 Market Risk Amendment. Key changes:

| Aspect | Pre-FRTB (Basel 2.5) | FRTB |
|--------|----------------------|------|
| Risk measure | VaR (99%, 10-day) | Expected Shortfall (97.5%) |
| Confidence level | 99% VaR | 97.5% ES (equivalent conservatism) |
| Lookback period | 1 year (equal weight) | 1 year (equal weight for ES) |
| Liquidity horizons | Uniform 10-day | Graded: 10d–120d per risk factor |
| Model approval | Firm-level | Desk-level |
| NMRF treatment | Not separately defined | Separate capital add-on |
| Banking/trading book boundary | Subjective | Prescriptive boundary |
| P&L Attribution test | Not required | Desk must pass PLA test |

**Key FRTB Concepts:**

- **Internal Models Approach (IMA)**: Desks with approved models use ES; subject to desk-level P&L Attribution (PLA) test and backtesting. Desks failing PLA revert to Standardised Approach (SA).
- **Standardised Approach (SA)**: Simplified sensitivity-based method (delta risk charges, vega risk charges, curvature risk charges, default risk charge)
- **Non-Modellable Risk Factors (NMRF)**: Risk factors with insufficient observable data — subject to a stress scenario capital add-on (not eligible for ES modelling)
- **Default Risk Charge (DRC)**: Replaces IRC, capturing jump-to-default risk in the trading book
- **Profit & Loss Attribution (PLA) Test**: Compares risk-theoretical P&L (from the pricing model) to actual desk P&L — desks must explain 75%+ of P&L variation to retain IMA approval
- **Strict Banking/Trading Book Boundary**: Prescriptive list of instruments eligible for trading book; presumption that debt instruments held to maturity are banking book

### 4.5 Market Risk Systems and Data

**Market Risk Systems:**
- **Murex MxG**: Front-to-back risk platform for trading, treasury, and risk management. Handles market risk, credit risk, treasury, and collateral. Strong in FX, interest rate, and derivatives.
- **Calypso**: Cross-asset trading, risk, and treasury platform. Used for market risk analytics, position keeping, and post-trade processing.
- **Bloomberg AIM**: Portfolio management and order management system with risk analytics for fixed income, equities, and derivatives.
- **Sophis Risque** (now SS&C): Multi-asset portfolio and risk management.
- **Imagine Software**: Real-time trading and risk management for derivatives.
- **OpenGamma**: Risk analytics platform specialising in derivatives — sensitivity, scenario, and margin analytics.
- **activePivot**: Real-time in-memory aggregation for market risk dashboards and P&L attribution (extremely fast for large position sets).

**Risk Factor Data:**
Market risk engines consume hundreds of risk factors: yield curves (by currency and tenor), FX spot and forward rates, equity indices and individual stock prices, volatility surfaces (equity, FX, interest rate), credit spreads (CDS curves, bond spreads), and correlation matrices.

**Market Data Vendors:**
- **Bloomberg**: B-PIPE (real-time), Bloomberg data license (historical)
- **Refinitiv** (now LSEG): Elektron, Tick History
- **SIX Financial**: Real-time and reference data
- **ICE Data Services**: Evaluated bond prices, CDS, derivatives pricing
- **IDC** (Interactive Data Corporation): Evaluated pricing

**Data Management Challenges:**
- Feed handlers must manage 500K+ individual instruments' prices daily
- Data quality checks: tolerance-based price moves, stale price detection, cross-vendor comparison
- Fallback procedures: when primary feed fails, substitute from secondary vendor, last available price, or modelled price
- Vendor reconciliation: resolving discrepancies between Bloomberg and Refinitiv for the same instrument

---

## 5. Operational Risk Systems

### 5.1 Operational Risk Definition and Basel Approaches

Operational risk is the risk of loss resulting from inadequate or failed internal processes, people, systems, or external events. This includes legal risk but excludes strategic and reputational risk.

The Basel framework has evolved through several measurement approaches:

| Approach | Description | Status |
|----------|-------------|--------|
| BIA (Basic Indicator Approach) | 15% of gross income | Superseded by Basel IV |
| TSA (Standardised Approach) | % of gross income per business line | Superseded by Basel IV |
| ASA (Alternative Standardised Approach) | Variant of TSA for retail/commercial banking | Superseded by Basel IV |
| AMA (Advanced Measurement Approach) | Banks' internal loss distribution models | Superseded by Basel IV |
| **SMA** (Standardised Measurement Approach) | BIC + ILM formula | **Final Basel III/Basel IV** |

The AMA (Advanced Measurement Approach) — which allowed banks to use their own internal models for operational risk capital — has been removed in the final Basel III framework. This was driven by the inability of AMA models to capture tail risk (banks rarely lost more than AMA capital before 2008, then lost multiples of it during the crisis) and lack of cross-bank comparability.

### 5.2 The Standardised Measurement Approach (SMA)

SMA computes operational risk capital as:

**Operational Risk Capital = BIC × ILM**

Where:

- **BIC (Business Indicator Component)** = BI × α
  - BI (Business Indicator) = sum of three components:
    - Interest, Lease, and Dividend Component (ILDC)
    - Services Component (SC)
    - Financial Component (FC)
  - α (regulatory factor): 11% for low BI, 15% for mid BI, 18% for high BI
- **ILM (Internal Loss Multiplier)**: Based on the bank's historical operational losses relative to BIC
  - ILM = ln(exp(1) − 1 + (Loss Component / BIC))^0.8
  - Loss Component = average annual operational loss × 7 (for the most recent 10 years of loss data)
  - ILM ranges from approximately 0.8 (low losses relative to BIC) to 1.5+ (high losses)

Banks must have at least **5 years of internal loss data** for SMA calculation (10 years preferred). The ILM can provide capital relief if the bank demonstrates a strong loss record, or increase capital if losses are high.

### 5.3 Operational Risk Systems and Data

**Systems Categories:**

- **Loss Data Collection Systems**: Capture and classify operational loss events (internal fraud, external fraud, execution failures, etc.). Examples: SAS OpRisk, IBM OpenPages, proprietary databases.
- **Risk and Control Self-Assessment (RCSA) Platforms**: Frameworks for business lines to assess inherent risk, control effectiveness, and residual risk. Often part of GRC platforms.
- **Key Risk Indicator (KRI) Dashboards**: Track leading and lagging indicators — system downtime, failed trades, staff turnover, operational losses, audit findings.
- **Scenario Analysis Tools**: Structured workshops producing expert-estimated loss distributions for severe events not well-captured by internal data.
- **GRC (Governance, Risk & Compliance) Platforms**: SAS OpRisk, IBM OpenPages, MetricStream, RSA Archer, ServiceNow GRC.

**Operational Risk Data:**

| Data Type | Description | Source |
|-----------|-------------|--------|
| Internal loss data | Actual operational loss events (date, amount, Basel category, business line) | Loss data collection system |
| External loss data | Loss events from other banks (industry consortia) | ORX, SAS OpRisk Global Data, Fitch |
| Scenario data | Expert estimates of loss frequency/severity | RCSA workshops |
| RCSA data | Control self-assessment scores | Business line submissions |
| KRI data | Key risk indicator values | Operations, IT, HR systems |
| External data | Benchmarking, industry trends | Regulators, industry bodies |

### 5.4 Banking-Specific OpRisk Types

Basel categorises operational risk into seven event types:

1. **Internal Fraud**: Misappropriation of assets, tax evasion, bribery, insider trading
2. **External Fraud**: Theft, forgery, cheque fraud, cyber attacks
3. **Employment Practices & Workplace Safety**: Discrimination, union disputes, health/safety
4. **Clients, Products & Business Practices**: Market manipulation, product defects, fiduciary breaches, mis-selling
5. **Damage to Physical Assets**: Natural disasters, terrorism, vandalism
6. **Business Disruption & System Failures**: Hardware/software failures, telecoms disruption, power outages
7. **Execution, Delivery & Process Management**: Failed transactions, data entry errors, accounting errors, missing legal documents

**Modern banking-specific OpRisk types (not explicit in Basel taxonomy but key in practice):**

- **Cyber Risk**: Data breaches, ransomware, DDoS attacks, supply chain attacks
- **Vendor / Third-Party Risk**: Outsourcing failures, vendor data breaches, concentration risk in critical vendors
- **Conduct Risk**: Mis-selling, unsuitable advice, poor customer outcomes (overlaps with Type 4)
- **Model Risk**: Incorrect model output, model misuse, model validation failures
- **Regulatory Risk**: Regulatory breaches, fines, enforcement actions, licence restrictions
- **Legal Risk**: Litigation, contract disputes, regulatory investigations

---

## 6. Liquidity Risk and Treasury Systems

### 6.1 Liquidity Risk and Basel Metrics

Liquidity risk is the risk that a bank cannot meet its financial obligations as they fall due without incurring unacceptable losses. It arises from mismatches between cash inflows and outflows, sudden withdrawal of funding, or market dislocation making it impossible to sell assets.

**Key Basel Metrics:**

**LCR (Liquidity Coverage Ratio) — Short-term (30 days):**
- Formula: Stock of HQLA / Total Net Cash Outflows over 30 days ≥ 100%
- HQLA (High-Quality Liquid Assets): Cash, central bank reserves, government bonds (Level 1), select corporate/covered bonds (Level 2A/2B) — with haircuts
- Net cash outflows: Stressed scenario assumptions (retail deposit run-off rates, wholesale funding roll-off, collateral calls, committed credit lines drawdown)
- Banks must maintain LCR ≥ 100% at all times (actual requirements may be higher due to local regulators)

**NSFR (Net Stable Funding Ratio) — Long-term (1 year):**
- Formula: Available Stable Funding (ASF) / Required Stable Funding (RSF) ≥ 100%
- ASF: Portion of capital and liabilities expected to be reliable over one year (tiered by stability)
- RSF: Stable funding required for each asset type and off-balance-sheet exposure
- Encourages banks to fund long-term assets with stable, long-term liabilities

**Additional Liquidity Metrics:**
- **Liquidity Gap Analysis**: Contractual and behavioural maturity mismatch (time buckets from overnight to 5+ years)
- **Concentration of Funding**: By counterparty, product, currency, and geography
- **Liquidity Stress Testing**: Firm-specific and market-wide scenarios (credit rating downgrade, loss of wholesale funding, market dislocation)
- **Intraday Liquidity Monitoring**: Real-time tracking of payment flows, settlement balances, and daylight overdrafts

### 6.2 ALM, FTP, and Liquidity Management

**Asset-Liability Management (ALM):**
- Strategic management of the balance sheet to achieve risk-return objectives
- Manages interest rate risk in the banking book (IRRBB — net interest income sensitivity and economic value of equity)
- Manages liquidity risk through structural liquidity planning
- Requires simulation of net interest income (NII) and economic value of equity (EVE) under parallel and non-parallel rate shocks

**Funds Transfer Pricing (FTP):**
- Internal mechanism charging business units for funding usage and crediting them for funding provision
- Two main components: liquidity FTP (term premium) and credit FTP (default risk)
- Behaviouralised FTP: prepayment assumptions, core deposit modelling, non-maturing deposit behaviour
- Critical for profitability measurement — distortions in FTP lead to mispriced products

**Intraday Liquidity Monitoring:**
- Growing regulatory focus (BCBS 248, MAS TRM guidelines)
- Systems must track: payment flows (SWIFT, RTGS), settlement queues, intraday credit usage, collateral availability
- Real-time dashboards showing current liquidity position vs. internal limits and regulatory thresholds

### 6.3 Treasury Systems Landscape

**Major Treasury Systems:**
- **Murex**: Front-to-back for treasury (FX, MM, derivatives, fixed income). Integrated credit and liquidity risk modules.
- **Calypso**: Cross-asset treasury and risk management. Handles FX, money markets, derivatives, and collateral.
- **Bloomberg AIM**: Portfolio management, order management, and treasury analytics. Heavy presence in asset management and hedge fund treasury.
- **Finastra Kondor Global Risk** (formerly Reuters Kondor+): FX, money market, and derivatives trading and risk. Broad instal base in commercial banks.
- **Wallstreet Suite** (now GTreasury): Treasury management, cash forecasting, debt and investment management.
- **Finastra Summit** (formerly Summit Systems): Derivatives and treasury processing for capital markets.
- **FIS Ambit**: Treasury and risk for mid-tier banks.
- **SAP Treasury**: Integrated with SAP ERP for cash management and bank account management.

**Data Requirements for Treasury Systems:**
- Cash flow data: Contractual (loan/deposit repayments, bond coupons, derivative cash flows) and behavioural (prepayment models, core deposit models)
- Collateral data: Collateral pool composition, eligibility, haircuts, rehypothecation rights
- Funding data: Wholesale funding maturities, retail deposit balances, secured/unsecured funding split, committed credit lines
- Securities portfolios: Bond holdings by type, maturity, currency, HQLA classification
- Market data: Yield curves, FX rates, funding spreads

---

## 7. AML and Financial Crime Compliance

### 7.1 AML/CFT Regulatory Framework

Anti-Money Laundering and Countering the Financing of Terrorism (AML/CFT) is one of the most operationally intensive compliance domains. Key regulatory frameworks:

- **FATF Recommendations**: 40 recommendations setting the global standard for AML/CFT — risk-based approach, customer due diligence, record-keeping, suspicious transaction reporting, and international cooperation
- **Wolfsberg Group**: Industry guidance on correspondent banking, trade finance, and beneficial ownership
- **MAS Notice 612** (Singapore): CDD, ongoing monitoring, STR filing, sanctions screening, record-keeping
- **US Bank Secrecy Act (BSA)**: Currency transaction reports (CTRs), suspicious activity reports (SARs), and CDD rules
- **EU Anti-Money Laundering Directive (AMLD)**: 6th AMLD (AMLD6) entered into force 2021 — expanded criminal liability, harmonised sanctions, beneficial ownership registers
- **Singapore CDSA** (Corruption, Drug Trafficking and Other Serious Crimes (Confiscation of Benefits) Act): Primary domestic AML legislation

### 7.2 Customer Due Diligence and KYC

CDD/KYC is the foundation of AML compliance — a bank cannot effectively monitor transactions if it does not know its customer.

**Core Functions:**

| Component | Description |
|-----------|-------------|
| **Identity Verification** | Government-issued ID verification, biometric matching, document authentication |
| **Beneficial Ownership Identification** | Identifying natural persons owning >25% of legal entities |
| **PEP Screening** | Politically Exposed Persons — higher risk category requiring enhanced due diligence (EDD) |
| **Sanctions Screening** | Real-time and batch screening against OFAC, UN, EU, MAS consolidated sanctions lists |
| **Adverse Media Screening** | Negative news screening for criminal activity, corruption, regulatory actions |
| **Risk Rating** | Low / Medium / High based on customer type, geography, occupation, product usage, transaction behaviour |
| **EDD (Enhanced Due Diligence)** | For high-risk customers: source of wealth documentation, ongoing transaction monitoring intensification |

**KYC Systems:**
- **Fenergo**: End-to-end CLM (Client Lifecycle Management) — onboarding, KYC, regulatory data, tax documentation (FATCA/CRS)
- **Moody's Analytics KYC**: Customer onboarding, screening, due diligence
- **LexisNexis Risk Solutions**: Identity verification, risk scoring, adverse media
- **World-Check / OneSumX**: Sanctions, PEP, and adverse media screening (Refinitiv/LSEG)
- **WorkFusion**: AI-powered KYC automation for document processing and data extraction

### 7.3 Transaction Monitoring

Transaction monitoring is the continuous surveillance of customer transactions to detect potentially suspicious activity.

**Rule-Based Scenarios (Traditional Approach):**

| Scenario | Description |
|----------|-------------|
| Structuring Detection | Transactions just below reporting thresholds |
| Rapid Movement | Funds flowing through multiple accounts in short periods |
| High-Risk Jurisdiction | Transactions involving countries with weak AML controls |
| Velocity Checks | Unusual transaction frequency within a time window |
| Large Cash Transactions | Cash deposits/withdrawals exceeding thresholds |
| Round Amount Patterns | Transactions in round numbers inconsistent with normal activity |
| Unusual Counterparty | Transactions with entities outside normal business relationships |
| High-Risk Product Usage | Use of complex products (trusts, shell companies, private investment vehicles) |

**Advanced Analytics (Modern Approach):**

- **Behaviour Detection**: Machine learning models that learn each customer's normal transaction profile (amounts, frequency, counterparties, channels) and flag deviations
- **Network Analysis**: Graph-based analysis identifying suspicious patterns — circular flows, funnel accounts, shared addresses/phones across unrelated customers
- **Anomaly Detection**: Unsupervised learning identifying transactions statistically far from expected patterns
- **Watchlist Filtering**: Real-time matching of transactions against sanctions, PEP, and internal blacklists
- **Case Management**: Workflow for investigators to review alerts, gather evidence, document decisions, and file SARs/STRs
- **Rule Tuning**: Regular optimisation of thresholds and scenarios to balance detection rate vs. false positive ratio

**AI/ML-Based Detection:**
- Neural networks for transaction classification (reducing false positive rates from 95%+ to 80%+ in production systems)
- Graph Neural Networks (GNNs) for suspicious entity identification
- Boosting models (XGBoost, LightGBM) for risk scoring
- Time-series models for behavioural baseline drift detection
- Agent-based simulation for typology discovery

### 7.4 Sanctions Screening

Sanctions screening compares customer names, counterparty names, transaction counterparties, and beneficiary/ordering entities against sanctions lists.

**Key Aspects:**

- **Lists**: OFAC (SDN list), UN Consolidated List, EU Consolidated List, MAS Sanctions List, UK OFSI list, internal blacklists
- **Batch Screening**: New customers against entire list; periodic rescreening of existing customers
- **Real-Time Screening**: Payment messages (SWIFT MT103, MT202) and trade transactions screened before processing
- **Fuzzy Matching Algorithms**: Levenshtein distance, Soundex, Double Metaphone, phonetic matching — manages transliteration variations, name variations, abbreviations
- **False Positive Reduction**: Filtering by jurisdiction, transaction context, name commonness; name scoring; suppression lists for false positives
- **List Management**: Automated list updates from sanctions authorities; manual overrides and audit trail for list changes

### 7.5 AML Systems Landscape

| Category | Systems |
|----------|---------|
| Transaction Monitoring | **Actimize** (NICE) — the dominant platform, with full suite for AML, trade surveillance, fraud; **Fiserv AML Manager**; **SAS AML**; **Oracle Financial Services AML**; **Quantexa** (network analytics, entity resolution); **NetReveal** (BAE Systems) |
| KYC/CLM | **Fenergo**; **Moody's Analytics KYC**; **LexisNexis Bridger**; **ComplyAdvantage** |
| Sanctions Screening | **World-Check (OneSumX)** ; **LexisNexis Bridger**; **Fircosoft**; **Accuity**; **SAS Sanctions** |
| Fraud Detection | **SAS Fraud Management**; **IBM Safer Payments**; **Feedzai**; **Featurespace** |
| Network/Graph Analytics | Custom solutions on **Neo4j**, **Memgraph**, **TigerGraph**; **Quantexa** |
| Regulatory Filing | **Filing.gov** (FinCEN); **Suspicious Activity Report (SAR)** filing systems; **goAML** (UNODC) |

---

## 8. Trade Surveillance and Conduct Risk

### 8.1 Trade Surveillance Scope and Regulations

Trade surveillance systems monitor trading activity to detect market abuse — insider dealing, market manipulation, and benchmark manipulation. The regulatory backdrop includes MiFID II, MAR, Dodd-Frank, and MAS guidelines on market conduct.

**What Trade Surveillance Monitors:**
- **Algo Trading Controls**: Pre-trade risk checks (order size, price collars, execution limits); algo code changes approval and testing; kill switches
- **Best Execution**: Monitoring whether client trades achieve best possible outcome under MiFID II (price, cost, speed, likelihood of execution)
- **Order Book Analysis**: Patterns of order placement and cancellation across trading venues
- **Trade Reconstruction**: Full sequencing of orders, amendments, cancellations, and executions for a given instrument
- **Communication Surveillance**: Voice recording, emails, Bloomberg chats, WhatsApp — cross-referenced against trading activity

### 8.2 Market Abuse Typologies

**Insider Dealing:**
- Trading while in possession of inside information
- Front-running: Trading ahead of a large client order
- Tipping: Passing inside information to others who then trade
- Trading patterns: Unusual profitability, timing correlation with corporate events

**Market Manipulation:**

| Type | Description |
|------|-------------|
| Wash Trading | Simultaneous buy and sell of same instrument to create false volume |
| Spoofing | Placing orders visible to the market with intent to cancel before execution |
| Layering | Multiple orders at different price levels creating false depth |
| Pump and Dump | Buying to inflate price, then selling at the inflated level |
| Cross-Product Manipulation | Manipulating a derivative (CDS) to profit in the underlying (bond) |
| Marking the Close | Trading at market close to influence settlement prices |
| Quote Stuffing | Rapid order entry/withdrawal to slow competitors' systems |

**Benchmark Manipulation:**
- LIBOR manipulation (pre-2012)
- FX Fix manipulation (WM/Reuters 4pm fix)
- ISDAFix manipulation
- Manipulation of commodity benchmarks

**Dissemination:**
- Spread false/misleading information via social media, chat rooms, or research
- "Hack and dump": Compromised accounts used to spread false information

**Improper Disclosure:**
- Delayed disclosure of inside information
- Selective disclosure to analysts before public release

### 8.3 Surveillance Systems and Capabilities

**Major Trade Surveillance Systems:**

- **NICE Actimize Surveillance**: Full-spectrum trade surveillance for equities, fixed income, derivatives, FX — includes cross-asset, cross-market correlation
- **Nasdaq SMARTS** (formerly SMARTS Group): Leading market surveillance platform — used by 80% of exchanges and many sell-side firms
- **FIS Protegent**: Real-time surveillance for equities and derivatives
- **Bloomberg Surveillance**: Bloomberg Terminal-integrated surveillance for trade reconstruction and order book analysis
- **OneMarketData (OMD)**: Cross-asset market data for surveillance analytics
- **IPC Systems**: Voice recording and communication surveillance integration
- **Cinnober IRIS** (now Nasdaq): Real-time surveillance and risk monitoring for exchanges and clearing houses

**Capabilities:**
- **Cross-market surveillance**: Correlating trading patterns across multiple venues (lit, dark pools, systematic internalisers)
- **Alert visualisation**: Network graphs showing connected accounts, traders, and instruments
- **Replay and reconstruction**: Full order book replay for investigation
- **Scenario backtesting**: Testing new surveillance typologies against historical data

### 8.4 Conduct Risk and Behavioral Analytics

Conduct risk is broader than market abuse — it encompasses all behaviours that could lead to poor customer outcomes, regulatory breaches, or reputational damage.

**Conduct Risk Areas:**
- Sales practices: Mis-selling, unsuitable advice, pressure selling
- Product suitability: Products sold to wrong customer segments
- Customer outcomes: Fair treatment, transparency, complaint handling
- Whistleblowing: Internal reporting of misconduct
- Conflicts of interest: Personal account dealing, gifts and entertainment, external business interests

**Behavioral Analytics (E-Comms Surveillance):**
Surveillance of electronic communications has become a major focus, driven by the proliferation of messaging apps (WhatsApp, WeChat, Signal) in financial services.

| Channel | Surveillance Approach |
|---------|----------------------|
| Voice | Voice-to-text transcription, keyword detection, sentiment analysis, speaker identification |
| Email | Keyword scanning, language model analysis, attachment inspection, thread analysis |
| Bloomberg Chat | Native Bloomberg API integration, keyword/phrase detection |
| WhatsApp | Mobile device management (MDM) integration, screenshot/document capture |
| WeChat / Signal | MDM capture (limited), proof-of-retention, policy enforcement |
| Zoom / Teams | Recording and transcription, meeting analytics |

**Conduct Risk Systems:**
- **Global Relay**: Message archiving and e-communications surveillance
- **Smarsh**: Digital communications capture and archiving — email, social media, WhatsApp, WeChat
- **NICE**: Voice recording, voice analytics, and e-communications surveillance
- **Verint**: Voice, video, and text analytics for compliance
- **SAI Global** (now Riskonnect): Conduct risk management and incident reporting

---

## 9. Regulatory Reporting Systems

### 9.1 Regulatory Returns Landscape

Regulatory reporting is the cornerstone of compliance — the systematic submission of financial and risk data to regulators on prescribed templates and schedules. Key returns by jurisdiction:

**EU / EEA:**
- **COREP** (Common Reporting): Capital adequacy — own funds, capital requirements, RWA by risk type, large exposures, leverage ratio, LCR, NSFR
- **FINREP** (Financial Reporting): IFRS-based financial statements — balance sheet, P&L, impairment, asset quality
- **AE**: Asset encumbrance reporting
- **ESMA reporting**: MiFID II transaction reporting, EMIR/SFTR trade reporting
- **AnaCredit**: Granular credit data reporting to the ECB

**US:**
- **FFIEC 101** (Regulatory Capital Reporting) and **FFIEC 102** (Market Risk RWA)
- **FR Y-9C**: Consolidated financial statements for bank holding companies
- **FR Y-14A/M/Q**: Capital planning and stress testing data
- **Call Reports** (FFIEC 031/041): Quarterly condition and income
- **Regulatory Capital** (FR 2052a): Liquidity monitoring

**Singapore:**
- **MAS 610** (Notices and returns): Capital adequacy, liquidity, large exposures, connected exposures
- **MAS 635**: Regulatory returns for securities-based activities
- **FATCA / CRS reporting** to IRAS

**Global:**
- **EMIR Trade Reporting**: OTC derivatives to trade repositories (DTCC, CME, ICE, UnaVista, Bloomberg)
- **SFTR**: Securities financing transactions to trade repositories
- **Dodd-Frank Swap Reporting**: Swap transactions to SDRs (swap data repositories)
- **Short Selling Reporting**: Net short positions to regulators

### 9.2 Data Lineage and Governance (BCBS 239)

BCBS 239 has been the primary driver of data lineage and governance requirements for regulatory reporting:

- **End-to-End Data Lineage**: Every number in a regulatory return must be traceable to its source system. Regulators expect to see full lineage maps showing data flow from trade capture / GL → data warehouse → validation → report → submission
- **Data Quality Dashboards**: Real-time dashboards measuring completeness, accuracy, timeliness, and validity of regulatory data. Certified by the Chief Data Officer (CDO)
- **Certification of Regulatory Returns**: Senior management attestation that returns are accurate and complete. Requires documented reconciliation between return values and general ledger / risk systems
- **Audit Trails**: Every change to regulatory data — manual override, reclassification, restatement — must be logged with user, time, reason, and previous value
- **Explanation of Variations**: Periodic variations in reported numbers must be documented with a business explanation, distinguishing between true economic changes, data quality changes, and methodology changes

### 9.3 Regulatory Reporting Systems

| System | Coverage | Notes |
|--------|----------|-------|
| **Oracle Financial Services Reg Reporting** | COREP, FINREP, US, local APAC; multi-jurisdiction | Part of OFSAA suite; strong data lineage |
| **AxiomSL** (now Adenza/SS&C) | Global regulatory reporting (COREP, FINREP, G-SIB, LCR, NSFR, CCAR, MAS, HKMA) | Market leader in regulatory reporting; strong transformation engine, data lineage |
| **Wolters Kluwer OneSumX** | COREP, FINREP, US, MAS, APAC; risk and compliance | Reg reporting, compliance, risk management |
| **Moody's RiskAuthority** | Credit risk RWA, capital adequacy | Strong credit risk RWA calculation engine |
| **IBM OpenPages** | Regulatory compliance management, not report generation | Platform for managing regulatory obligations, not return computations |
| **Vizor Regulatory Reporting** | XBRL regulatory returns | Used by central banks and regulators as submission platform |
| **FRS (Fidelity Regulatory Solutions)** | Market-leading in US Call Reports, FR Y-9C | US-specific regulatory reporting |
| **Bloomberg REGCOMP** | Regulatory submission platform | Used by asset managers primarily |

**Feeds Into Regulatory Reporting Systems:**
- Finance GL (general ledger): Balance sheet, income statement, retained earnings
- Risk systems: Credit RWA, market risk capital, operational risk capital
- Trade capture / position keeping: Derivatives exposures, trading book positions
- Reference data: Legal entities, counterparty data, product taxonomy, ratings

### 9.4 The Regulatory Hub Concept

A growing architectural pattern is the **regulatory hub** — a central data store for all regulatory reporting:

**Characteristics:**
- Single source of truth for regulatory data across all returns
- Central repository of data validation and transformation rules
- Unified data lineage for all regulatory numbers
- Standardised reconciliation engine (risk vs. finance, returns vs. source systems)
- Automated report generation and submission
- Audit trail for all data movements and transformations

**Benefits:**
- Eliminates siloed regulatory reporting (different teams using different systems for different returns, producing inconsistent numbers)
- Reduces reconciliation burden between returns
- Enables cross-return traceability
- Simplifies regulatory audit response (one system to trace, not 15)

**Challenges:**
- Building a single data model that serves COREP, FINREP, MAS, CCAR, US Call Reports, and EMIR simultaneously is extremely complex — each return uses different taxonomies, thresholds, consolidation scopes, and valuation bases
- Differences in taxonomy (e.g., European NACE vs. US NAICS industry codes), consolidation scope (prudential vs. accounting), and valuation methodology (IFRS vs. Basel) lead to inherent inconsistencies that the hub must manage without masking genuine differences
- Data ownership: Different business divisions own different source data — the regulatory hub aggregates but does not control upstream quality
- Performance: Running all regulatory returns through a single computation engine requires significant infrastructure (in-memory grids, parallel processing)

**Implementation Approaches:**

Banks typically take one of three approaches to regulatory hub buildout:

1. **Greenfield Unified Platform**: Acquire a single vendor platform (e.g., AxiomSL, OneSumX) to replace all existing regulatory reporting systems. High cost, high risk, but maximum consistency. Typically takes 3–5 years for a full implementation at a large bank.

2. **Hub-and-Spoke**: Build a central data repository and lineage engine, but retain specialised computation engines for specific returns (e.g., Credit RWA in Moody's RiskAuthority, Liquidity in a dedicated treasury system). The hub handles data consolidation, transformation, validation, and lineage; computation is delegated. Most common approach at large banks.

3. **Incremental Convergence**: Start with one domain (e.g., COREP) as the hub, then progressively add other returns (FINREP, liquidity, large exposures) over successive releases. Each release adds another set of templates and rules to the same platform. Lower risk, but full convergence may take 7+ years.

**Single Source of Truth — The Reality:**

The concept of a single source of truth for risk and compliance data is widely discussed but rarely achieved in practice. The reality is typically a **controlled federation** where:

- The regulatory hub holds the authoritative version of regulatory numbers (the numbers submitted to regulators)
- Source systems remain the authoritative systems for underlying positions, trades, and counterparties
- The hub validates, transforms, and reconciles data, but does not replace source systems
- Permanent difference documentation explains why risk numbers and finance numbers differ and why this is acceptable
- Data lineage connects every regulatory number back to source systems, with transformation logic documented and version-controlled

**Submission Distribution:**
- Direct to regulator portals: XBRL (EU), XML (MAS and other regulators), PDF/CSV (various)
- Regulatory submission platforms: Bloomberg REGCOMP, Surecomp, Gresham Technology

---

## 10. Technology Architecture Patterns

### 10.1 Integration Patterns and System Topology

Risk and compliance systems do not exist in isolation — they form a complex web of integrations across the bank's technology estate.

**Source Systems That Feed Risk and Compliance:**

| Source System | Data Provided |
|---------------|--------------|
| Trade Capture Systems (Murex, Calypso, Summit, Front Arena, Kondor+, Bloomberg AIM) | Trade details, positions, valuations, sensitivities |
| General Ledger (Oracle EBS, SAP, custom mainframe) | Balance sheet, P&L, impairment, accounting entries |
| Settlement Systems | Confirmed trades, failed trades, settlement status |
| SWIFT Messaging | Payment instructions, trade confirmations, corporate actions |
| Market Data Feeds (Bloomberg, Reuters, ICE, SIX) | Prices, yield curves, volatilities, ratings |
| Core Banking Systems (Finacle, Silverlake, T24, FIS Profile) | Customer accounts, deposits, loans, transaction history |
| KYC/Onboarding Systems | Customer risk ratings, documentation, screening results |

**Typical Architecture (Layered):**

```
Source Systems → ETL/CDC → Data Lake (Raw) → Data Warehouse (Curated) → Risk Engines, Reporting, Dashboards
```

- **Data Lake (Raw)**: Landing zone for all source data in native format — Kafka topics, files on HDFS/S3, streaming tables. Preserves data for lineage, audit, and reprocessing.
- **Data Warehouse (Curated)**: Quality-controlled, reconciled, transformed data in risk and finance data models (BCBS 239 compliant). Often uses star/snowflake schemas for aggregation and reporting.
- **Risk Engines**: Run calculations on curated data — Monte Carlo for market risk, PD models for credit risk, SMA formula for operational risk.
- **Dashboards and Reports**: Consume risk engine output and warehouse aggregates for regulatory returns, risk dashboards, and management information.

### 10.2 System Classification Map

| Category | Primary Systems | Function |
|----------|----------------|----------|
| **Regulatory Reporting** | AxiomSL (Adenza/SS&C), Wolters Kluwer OneSumX, Oracle Financial Services RCS | Generate and submit regulatory returns |
| **Risk Engines** | Moody's RiskFrontier, SAS Risk, Murex MxG, Bloomberg AIM | Compute credit/market/operational risk capital |
| **ECL / IFRS 9 / CECL** | Moody's ImpairmentStudio, SAS Expected Credit Loss, Oracle Financial Services ECL | Expected credit loss calculation and accounting |
| **AML / KYC** | Actimize, Fenergo, LexisNexis Bridger, Quantexa, ComplyAdvantage | Transaction monitoring, screening, KYC |
| **Trade Surveillance** | NICE Actimize Surveillance, Nasdaq SMARTS, Bloomberg Surveillance | Market abuse detection |
| **GRC Platforms** | SAS OpRisk, IBM OpenPages, MetricStream, RSA Archer, ServiceNow GRC | Operational risk, compliance management |
| **Treasury** | Murex, Calypso, Bloomberg AIM, Finastra Kondor, Wallstreet Suite | ALM, FTP, liquidity, cash management |
| **Risk Data Management** | OFSAA, SAS Risk Management, IBM OpenPages, Moody's RiskAuthority, FIS Adaptiv | Data integration, lineage, quality for risk |

### 10.3 Data Integration and Challenges

**Integration Patterns:**

| Pattern | Description | Latency | Typical Use |
|---------|-------------|---------|-------------|
| Real-time (Kafka, MQ) | Streaming event processing | Sub-second | Trade capture, market data, limit checking |
| Near-real-time | Micro-batch (5–15 min intervals) | Minutes | Position keeping, P&L attribution |
| Batch (overnight) | Daily bulk loads | T+1 | GL feeds, regulatory reporting, risk calculations |
| File-based (SFTP) | Periodic flat file exchange | Variable | External vendor data, regulatory submissions |
| API-based | REST/SOAP web services | On-demand | Reference data lookup, STP |

**Data Quality and Governance (BCBS 239 Deep Dive):**

Under BCBS 239 principles 4–5, banks must demonstrate that risk data is accurate and complete. Common data quality dimensions implemented in risk platforms include:

| Dimension | Definition | Typical Metrics |
|-----------|------------|-----------------|
| Completeness | All required data fields are populated | % populated fields, % missing critical data |
| Accuracy | Data correctly reflects the source/real-world value | Reconciliation variance %, error rate |
| Timeliness | Data available within required timeframes | % data loaded by T+1, intraday latency |
| Validity | Data conforms to defined formats, ranges, and rules | % valid ISINs/LEIs, date format compliance |
| Consistency | Data is coherent across systems and time | Cross-system variance %, period-over-period stability |

BCBS 239 implementation typically involves:
- **Data Ownership and Stewardship**: Business data owners appointed for each data domain (counterparty, product, trade, position). Data stewards responsible for quality measurement and remediation.
- **Data Quality Dashboard**: Enterprise data quality tool (Informatica DQ, Collibra, Talend, SAS Data Management) visualising quality metrics against thresholds, with escalation workflows for degraded quality.
- **Data Certification**: Periodic (quarterly) sign-off by data owners that quality meets defined standards. Certification recorded in governance system with sign-off audit trail.
- **Issue Management**: Data quality issues logged, triaged, assigned, tracked to resolution. Materiality assessment: does the issue affect regulatory numbers?
- **Remediation Plans**: For failing quality dimensions, documented remediation plans with target dates, ownership, and progress tracking.

**Risk and Finance Reconciliation (R&F Convergence Technical Detail):**

The reconciliation between risk-engine outputs and finance (GL) numbers is one of the most persistent operational challenges in banking. Differences arise from:

| Source of Difference | Risk System | Finance System | Resolution |
|----------------------|-------------|----------------|------------|
| Valuation methodology | Basel EPE (effective expected positive exposure) | IFRS 13 fair value | Recognised as permanent difference |
| Consolidation scope | Regulatory consolidation (prudential) | Accounting consolidation (IFRS 10) | Reconciliation schedule |
| Accrual vs. mark-to-market | MTM for all trading book positions | Accrual accounting for some | Adjusting entries |
| Provisioning | Basel EL vs. IFRS 9 ECL | IFRS 9 ECL (different methodology) | Provision mapping table |
| Day count conventions | Act/360 on trades | Act/365 for accounting | System transformation layer |
| Timing differences | Trade date accounting | Settlement date accounting | Reconciled daily |

Many banks now run automated R&F reconciliation engines that take risk outputs and GL outputs, compare at granular level, and automatically classify differences as: timing, permanent, data error, or methodology — with drill-down to individual trades or positions for errors. This is a core regulatory reporting control, tested by auditors and regulators during examinations.

---

## 11. Cloud Adoption and Data Management

### 11.1 Cloud Adoption in Risk and Compliance

Risk and compliance systems have traditionally been on-premise due to regulatory concerns about data residency, confidentiality, and supervisory access. However, cloud adoption is accelerating.

**Current State:**
- **On-premise dominant**: Most core risk engines, regulatory reporting, and AML systems run on-premise or in co-location data centres
- **Hybrid deployments**: Non-critical workloads (data lakes, analytics sandboxes, dashboards) moving to cloud
- **Cloud for stress testing**: Elastic compute for CCAR/ICAAP stress testing (massive parallel scenario computation) — AWS, Azure, GCP

**Cloud Advantages:**
- Elasticity for stress testing (thousands of cores on-demand, then release)
- Advanced analytics: Cloud-native ML services (SageMaker, Vertex AI, Azure ML) for credit scoring, fraud detection
- Data platforms: Snowflake, Databricks, BigQuery for risk data aggregation
- Reduced infrastructure management, faster deployments

**Regulatory Cloud Services:**
- **AWS Risk & Compliance**: GuardDuty, Config, Security Hub, Macie, Audit Manager
- **Azure Policy**: Regulatory compliance built-in (SOC2, PCI, FedRAMP, MAS TRM)
- **GCP Assured Workloads**: Sovereign controls, CMEK (customer-managed encryption keys), Access Transparency

**Challenges:**
- **Data residency**: Regulators require data to stay within jurisdiction. Cloud providers offer region-specific data centres but multi-jurisdiction banks must manage data partitioning
- **Regulatory approval**: Outsourcing to cloud requires notification/approval (MAS Notice 653, EBA Guidelines, PRA SS 2/21). Some regulators prohibit cloud for certain tier-1 applications
- **Vendor lock-in**: Proprietary cloud data platforms make multi-cloud difficult; data gravity reinforces single-vendor dependency
- **Managed services**: Increasing availability of managed risk and compliance services on cloud (SAS Cloud, Moody's Analytics Cloud, AxiomSL Cloud)

### 11.2 Risk and Finance Data Integration

Risk and Finance Data Integration (RFI) is the technical and business discipline of harmonising data used by risk management and financial reporting.

**Key Integration Dimensions:**
- **Data Model**: Unified chart of accounts, risk taxonomy, product hierarchy
- **Data Lineage**: End-to-end traceability from source to risk calculation to regulatory return
- **Data Quality**: Common quality dimensions for risk and finance — completeness, accuracy, timeliness, consistency
- **Reconciliation**: Automated reconciliation between risk-engine outputs and GL entries, with drill-down to position level

**BCBS 239-Driven Capabilities:**
- **Data Quality Dashboards**: Real-time metrics on completeness, accuracy, and timeliness of risk data. Green/amber/red thresholds with escalation. Certified by business data owners.
- **Data Certification**: Periodic sign-off on data quality by data owners and data stewards. Certification recorded in audit trail.
- **Reference Data Management (RDM)**: Golden copy of counterparties, legal entities, instrument masters, product taxonomies, rating scales, and collateral types
- **Market Data Management**: Vendor management (Bloomberg, Refinitiv, ICE, SIX), feed handler resilience, price validation, fallback procedures, data quality reporting

---

## 12. Advanced Analytics and AI/ML

### 12.1 Machine Learning Use Cases

**Credit Risk Scoring:**
- Gradient boosting models (XGBoost, LightGBM, CatBoost) for PD estimation — outperforming traditional logistic regression
- Neural networks for complex interactions in corporate credit risk
- Alternative data inclusion: Utility payments, transaction metadata, social media signals, satellite imagery (for agricultural loans)
- Explainability requirement: SHAP, LIME, and custom rule extraction for model interpretability under SR 11-7

**AML Transaction Monitoring:**
- ML-based anomaly detection reducing false positive ratios from 95%+ to 80%+ 
- Graph Neural Networks (GNNs) detecting complex money laundering networks (circular flows, layering patterns)
- Unsupervised learning for detecting unknown typologies
- Time-series models for behavioural drift detection

**Market Abuse Detection:**
- Anomaly detection on trading patterns: Isolation Forest, autoencoders, LSTM-based sequence models
- Order book manipulation detection: Convolutional neural networks on L2 order book snapshots
- Cross-market correlation analysis: Graph-based alerts for correlated trading across instruments

**Fraud Detection:**
- Real-time scoring of payments and transactions using ensemble models
- Deep learning for card fraud detection (transaction sequences)
- Entity resolution linking fraud accounts across different banking platforms

### 12.2 NLP and Graph Analytics for Compliance

**NLP for Compliance:**
- **Contract Analysis**: NLP extraction of key terms (covenants, triggers, material adverse change clauses) from legal documents
- **E-Communications Surveillance**: Language models for detecting market abuse and conduct risk in voice transcripts, chats, and emails
- **Regulatory Text Mining**: Parsing new regulations, identifying obligations applicable to the bank, mapping to existing controls and policies
- **SAR Narrative Generation**: Semi-automated drafting of suspicious activity report narratives from case management data

**Network Analysis (Graph Analytics):**
- **Graph Databases**: Neo4j, Memgraph, TigerGraph for financial crime detection
- **Centrality Measures**: Identifying central nodes (accounts) in suspicious transaction networks using betweenness, closeness, PageRank
- **Community Detection**: Louvain, label propagation, and other algorithms to find clusters of related accounts
- **Entity Resolution**: Deduplicating customer records across systems using graph-based matching (identity resolution)
- **Link Analysis**: Visualising relationships between accounts, transactions, counterparties, devices, and IP addresses

### 12.3 LLM Applications in Compliance

Large Language Models (LLMs) are finding increasing application in compliance:

- **Regulatory Q&A**: LLM-powered chatbots answering compliance questions from regulatory texts — "What is the MAS reporting deadline for STRs?"
- **Policy Extraction**: Extracting specific obligations, thresholds, and deadlines from dense regulatory documents
- **Document Summarisation**: Summarising regulatory guidance, policy documents, and investigation reports
- **SAR Narrative Generation**: Drafting structured narratives for suspicious activity reports from case evidence
- **Chatbot for KYC**: Guiding relationship managers through KYC documentation requirements

**Caveats and Guardrails:**
- LLMs hallucinate — regulatory content requires human-in-the-loop validation
- Data privacy: Customer data must not be sent to third-party LLM APIs
- Model risk: LLMs used in compliance processes fall under SR 11-7 and similar model risk frameworks

### 12.4 AI Governance and Model Risk Management

AI/ML models in risk and compliance are subject to rigorous model risk management (MRM) frameworks:

**SR 11-7 (Fed/OCC) — Supervisory Guidance on Model Risk Management:**
- **Model Definition**: Any quantitative method, system, or approach that applies statistical, economic, financial, or mathematical theories, techniques, and assumptions
- **Three Lines of Defence**: Model development (1st), model validation (2nd), internal audit (3rd)
- **Key Elements**:
  - Model development: Sound design, theory, data quality, implementation
  - Model validation: Conceptual soundness, outcomes analysis, ongoing monitoring
  - Governance: Policies, procedures, roles, model inventory
  - Documentation: Comprehensive, clear, audit-ready
- **Model Risk Tiering**: Low/medium/high risk based on materiality, complexity, and usage

**AI/ML-Specific MRM Considerations:**
- **Explainable AI**: SHAP/LIME/PDP for credit decisions. Regulators increasingly expect individual-level explanations for adverse decisions.
- **Fairness / Bias Monitoring**: Disparate impact testing for credit scoring and KYC risk rating. Protected attribute analysis (race, gender, age as prohibited by local law).
- **Model Risk for LLMs**: Extended MRM covering prompt injection, hallucination, output toxicity, data leakage, and reproducibility
- **Regulatory Expectations for AI Governance**: EU AI Act (risk-based classification for financial AI — credit scoring and insurance pricing classified as high-risk, requiring conformity assessments, human oversight, and documentation), MAS FEAT (Fairness, Ethics, Accountability, Transparency for credit scoring — originally co-developed by MAS with ABS (Association of Banks in Singapore), now a de facto standard for AI in ASEAN credit decisions), NY DFS Part 504, Bank of England/PRA expectations for AI governance in model risk frameworks.

**Model Inventory and Lifecycle Management (ModelOps):**
Banks typically maintain a model inventory of 500–3000+ models across credit risk (PD, LGD, EAD, scorecards), market risk (VaR models, pricing models), operational risk (fraud detection, RCSA quantification), AML (transaction monitoring, screening), and finance (ECL, FTP). ModelOps platforms (H2O AI, ModelOp, SAS Model Manager, MLOps tools on cloud) manage the lifecycle:

- **Development**: Version-controlled model code, training data versioning, experiment tracking (MLflow, DVC, Weights & Biases)
- **Validation**: Independent model validation team (separate from development), documented challenge of conceptual soundness, outcomes analysis (backtesting, benchmarking), and implementation assessment
- **Deployment**: Model deployment pipeline with automated testing, champion/challenger model testing (running new model in parallel with incumbent)
- **Monitoring**: Ongoing performance monitoring (PSI — Population Stability Index, CSI — Characteristic Stability Index, AUC decay, calibration drift), data drift detection, model refresh triggers
- **Retirement**: Model decommissioning process with documented justification, impact assessment, and archival

**Model Risk for LLMs — Emerging Framework Elements:**
- Prompt injection resistance testing
- Hallucination rate measurement and guardrail validation
- Output consistency / reproducibility testing
- Data leakage detection (training data memorisation)
- Toxicity and bias evaluation on financial domain content
- Evidence-attribution mechanisms (retrieval-augmented generation, citation grounding)

**Example ML Use Cases Summary:**

| Domain | Use Case | ML Approach | Regulatory Context |
|--------|----------|-------------|--------------------|
| AML | Transaction monitoring false positive reduction | Ensemble learning, anomaly detection | MAS 612, BSA |
| IFRS 9 | ECL macroeconomic scenario integration | Satellite models, Markov switching | IFRS 9, CECL |
| Regulatory Change | Regulatory text mining and obligation extraction | NLP, LLMs | Cross-regulatory |
| Market Abuse | Anomaly detection on trading patterns | Autoencoders, GNNs | MAR, MiFID II |
| Credit Risk | Alternative data credit scoring | Gradient boosting, neural networks | IRB, SR 11-7 |

---

## 13. Future Trends

### 13.1 RegTech and SupTech

**RegTech (Regulatory Technology):**
The RegTech ecosystem has grown rapidly, driven by the cost of compliance (estimated at 4–10% of non-interest expenses for major banks). Key trends:

- **API-Based Regulatory Reporting**: XBRL and XML-based direct filing APIs. Some regulators (MAS, HKMA, ECB) are adopting structured digital reporting.
- **Machine-Readable Regulation**: Digitising regulations into machine-executable formats (taxonomies, rules engines). The UK FCA is experimenting with digital sandboxes.
- **NLP for Regulatory Interpretation**: Automated monitoring of regulatory publications, extracting obligations, deadlines, and penalties, mapping to existing controls.
- **Automated Compliance Monitoring**: Continuous control monitoring rather than point-in-time attestation.
- **Streamlined KYC Utilities**: Shared KYC utilities (e.g., SWIFT KYC Registry, banks' shared utilities like the Singapore KYC marketplace) reducing redundant onboarding.
- **Digital Identity Verification**: e-KYC using biometrics, digital identity systems (Singpass, eIDAS, Aadhaar), video verification, and blockchain-based identity.

**SupTech (Supervisory Technology):**
Regulators themselves are adopting technology — data analytics for market surveillance, NLP for regulatory filings analysis, and machine learning for risk assessment. Examples include MAS' SupTech initiatives for AML data analytics and the ECB's off-site supervision data platform.

### 13.2 ESG Risk Integration

Environmental, Social, and Governance (ESG) risk is rapidly being integrated into the risk management framework:

- **ESG Risk Taxonomy**: Categorising climate risk (physical risk: floods, fires, storms; transition risk: carbon pricing, policy changes, technology shifts), social risk (labour practices, community impact), and governance risk (board composition, executive compensation, corruption)
- **Climate Stress Testing**: Regulators are running climate scenario analysis exercises (NGFS scenarios — orderly, disorderly, hot house world). Banks must model impact of climate scenarios on credit risk (PD/LGD shifts in carbon-intensive sectors), market risk (repricing of carbon-exposed assets), and operational risk (physical damage to branches, data centres)
- **Disclosure Requirements**: SFDR (EU Sustainable Finance Disclosure Regulation), CSRD (Corporate Sustainability Reporting Directive), TCFD (Task Force on Climate-related Financial Disclosures), ISSB (International Sustainability Standards Board)
- **Carbon Pricing Risk**: Impact of carbon taxes on loan portfolios and trading books
- **Physical Risk Models**: Catastrophe modelling for property portfolios, supply chain disruption
- **Data Challenges**: ESG data is sparse, inconsistent, and lacks standardisation. Climate risk data (scenario parameters, physical risk maps) is new for most banks

### 13.3 Real-Time Risk

Regulatory and business pressures are driving toward real-time risk measurement:

- **Intraday Risk Measurement**: FRTB pushes toward intraday market risk reporting (desk-level ES must be monitored intraday). Real-time credit limit monitoring for counterparties.
- **Real-Time Position Keeping**: Continuous position updates from trade capture, rather than end-of-day batch.
- **Intraday Liquidity Monitoring**: Real-time tracking of payment flows, settlement queues, and collateral positions (BCBS 248 requirements).
- **Real-Time AML Screening**: Payments screened against sanctions lists in milliseconds.
- **Technological Enablers**: In-memory databases (SAP HANA, activePivot), streaming analytics (Kafka Streams, Flink, Spark Streaming), real-time data pipelines

### 13.4 Convergence of Risk, Finance, and Treasury

The "three pillars" of the finance function — risk, finance (CFO), and treasury — are converging:

- **Unified Data Model**: A single data model covering risk (credit, market, operational, liquidity), finance (GL, P&L, balance sheet), and treasury (cash flows, funding). Aspirational but increasingly practical with modern data platforms.
- **Integrated Stress Testing (CST)**: Concurrent stress testing — running credit, market, operational, and liquidity stress tests on a unified dataset. Replaces siloed CCAR / ICAAP / liquidity stress testing.
- **Basel III / FRTB Integration Demand**: FRTB creates more integration demand by requiring consistent data across trading and banking books, consistent valuation methodologies, and consolidated risk reporting.
- **Organisational Convergence**: Many banks are creating "Financial Risk Management" functions combining market risk, credit risk, and treasury risk under a single CRO-reporting structure.
- **Technology Convergence**: Cloud data platforms (Snowflake, Databricks, BigQuery) that serve risk, finance, and treasury from a single data foundation.

### 13.5 Modern Data Architecture Transitions

Risk and compliance systems are gradually moving from SOA/monolithic architectures toward modern data architectures:

- **Data Mesh**: Domain-driven data ownership — credit risk owns credit data, market risk owns market data, treasury owns liquidity data — with central data infrastructure for sharing and governance. Each domain publishes data products consumed by other domains.
- **Data Fabric**: Automated data integration, quality, and lineage across on-premise and cloud data sources. Metadata-driven data management with AI-powered data cataloguing.
- **Cloud Data Platforms**: Snowflake, Databricks, and Google BigQuery for risk data aggregation and analytics. Allows separation of compute and storage, enabling elastic scaling for stress testing.
- **Real-Time Data Streaming**: Kafka, Flink, and Spark Streaming replacing batch ETL for time-sensitive data flows (market data, trade capture, AML screening).
- **Microservices for Risk**: Decoupling monolithic risk engines into independent services (pricing service, VaR service, limit check service) communicating via APIs and event streams.
- **Open Source Adoption**: Growing use of open-source tools: Apache Spark for risk calculations, Airflow for workflow orchestration, Presto/Trino for ad-hoc querying, dbt for data transformation — alongside traditional vendor systems.

---

## 14. References and Further Reading

**Regulatory Documents:**
- Basel Committee on Banking Supervision. (2017). *Basel III: Finalising Post-Crisis Reforms.*
- Basel Committee on Banking Supervision. (2013). *Principles for Effective Risk Data Aggregation and Risk Reporting (BCBS 239).*
- Monetary Authority of Singapore. *MAS Notice 637 — Risk Management.*
- Monetary Authority of Singapore. *MAS Notice 612 — Anti-Money Laundering and Countering the Financing of Terrorism.*
- Federal Reserve. (2011). *SR 11-7: Supervisory Guidance on Model Risk Management.*
- European Securities and Markets Authority (ESMA). *MiFID II / MiFIR Regulatory Framework.*

**Industry Frameworks:**
- Financial Stability Board. *Thematic Review on Implementation of BCBS 239.*
- Wolfsberg Group. *Correspondent Banking Due Diligence Questionnaire.*
- FATF. *International Standards on Combating Money Laundering and the Financing of Terrorism & Proliferation.*
- NGFS. *Climate Scenarios for Central Banks and Supervisors.*

**Vendor Documentation:**
- Moody's Analytics. *RiskFrontier, ImpairmentStudio Documentation.*
- SAS Institute. *SAS Risk Management, SAS AML Documentation.*
- NICE Actimize. *Actimize AML, Surveillance Documentation.*
- SS&C Adenza. *AxiomSL Regulatory Reporting Documentation.*

---

*Written for the [research](https://github.com/jackliusr/research) repository — technology/ subdirectory. This guide is a living document; updates for regulatory changes, new vendor products, and evolving technology trends should be contributed via the usual PR process.*
