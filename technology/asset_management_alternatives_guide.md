# Asset Management & Alternatives Management: A Comprehensive Guide

> **Author**: Jack Liu Shurui  
> **Repository**: https://github.com/jackliusr/research  
> **Last Updated**: July 2026  
> **Scope**: Global industry overview with Singapore-specific focus

---

## Table of Contents

1. [Asset Management Overview](#1-asset-management-overview)
2. [Investment Process: Front to Back](#2-investment-process-front-to-back)
3. [Alternatives Management](#3-alternatives-management)
4. [Key Differences: Traditional AM vs Alternatives](#4-key-differences-traditional-am-vs-alternatives)
5. [Key Operational Processes](#5-key-operational-processes)
6. [Technology & Architecture](#6-technology--architecture)
7. [Singapore Context](#7-singapore-context)
8. [Glossary](#8-glossary)

---

## 1. Asset Management Overview

### 1.1 What Is Asset Management?

Asset management is the professional management of investment portfolios on behalf of clients — including institutions (pension funds, insurance companies, sovereign wealth funds, endowments), high-net-worth individuals (HNWIs), and retail investors. Asset managers deploy capital across financial markets to achieve specific risk-return objectives, earning fees for their services.

The global asset management industry reached approximately **$147 trillion in AUM by mid-2025**, according to McKinsey and BCG, up 11% year-over-year. PwC projects global AUM will reach **$200 trillion by 2030** at a CAGR of 6.2%. Over 80% of gross revenue growth in 2025 was driven by market appreciation rather than organic inflows, highlighting the industry's continued reliance on market tailwinds.

### 1.2 Key Players (Top Global Asset Managers)

| Rank | Firm | AUM (approximate) | Headquarters |
|------|------|-------------------|--------------|
| 1 | BlackRock | ~$11.5T | New York |
| 2 | Vanguard | ~$9.0T | Valley Forge, PA |
| 3 | State Street Global Advisors | ~$4.3T | Boston |
| 4 | Fidelity Investments | ~$4.2T | Boston |
| 5 | Capital Group | ~$2.6T | Los Angeles |
| 6 | J.P. Morgan Asset Management | ~$2.5T | New York |
| 7 | Amundi | ~$2.2T | Paris |
| 8 | PIMCO | ~$1.9T | Newport Beach, CA |
| 9 | UBS Asset Management | ~$1.7T | Zurich |
| 10 | DWS Group | ~$1.0T | Frankfurt |

Additional notable players include **Invesco**, **Franklin Templeton**, **T. Rowe Price**, **Credit Agricole Asset Management**, and **BNP Paribas Asset Management**.

### 1.3 Asset Classes

**Public Equity (Stocks)**: The largest asset class. Managers invest in listed company shares — geographically (US, Europe, EM, Asia ex-Japan) and by style (growth, value, blend, GARP).

**Fixed Income (Bonds)**: Government bonds (USTs, JGBs, Bunds, Gilt), corporate bonds (IG, HY), emerging market debt, agency/MBS, municipal bonds. Duration and credit quality define risk profiles.

**Cash & Equivalents**: Money market instruments, T-bills, commercial paper, repos. Used for liquidity management and as a defensive allocation.

**Multi-Asset / Balanced**: Combined equity + fixed income + alternatives in a single managed portfolio. Includes target-date funds (TDFs), risk-parity, and balanced mandates.

### 1.4 Business Models: Active vs Passive

**Active Management**: Portfolio managers make discretionary investment decisions to outperform a benchmark index. Requires intensive research, analytical staff, and trading. Average active equity fees for institutional mandates: 0.35–0.70% ER; retail active funds: 0.50–1.20%.

**Passive / Index Management**: Tracks a benchmark (S&P 500, FTSE 100, MSCI World) at minimal cost. Dominated by Vanguard, BlackRock (iShares), and State Street (SPDR). Fees as low as 0.01–0.10%. Indexing surpassed active US equity AUM for the first time in 2023 and continues to grow share.

**Smart Beta / Factor Investing**: Rules-based strategies targeting specific factors (value, momentum, quality, low volatility, size) — a hybrid between active and passive.

### 1.5 Fee Structures

- **Management Fee**: A% of AUM, paid annual (typically accrued daily, paid monthly/quarterly). Ranges 0.01% (passive) to 2.0%+ (specialist active/alternatives).
- **Performance Fee**: % of outperformance (20% of returns above a hurdle, common in hedge funds).
- **Carried Interest**: Profit share for private equity/real estate GPs (typically 20% of profits after returning LP capital and hurdle).
- **Fee Compression**: Ongoing industry trend. Active fund fees dropped to ~0.36% average by 2025 as passive competition intensifies. Vanguard and Schwab led fee cuts in 2025.

---

## 2. Investment Process: Front to Back

The investment lifecycle is organized into three functional lines: Front Office, Middle Office, and Back Office.

### 2.1 Front Office

**Research → Idea Generation → Portfolio Construction → Order Generation**

The Front Office generates alpha. It comprises portfolio managers (PMs), analysts (equity, credit, macro, quant), and traders.

#### Research & Idea Generation
- **Fundamental Research**: Bottom-up company analysis (financial statements, management meetings, channel checks, industry calls). Analysts produce investment theses, target prices, and rating changes.
- **Quantitative Research**: Systematic models using factors, machine learning, and alternative data (satellite imagery, credit card transactions, web scraping) to generate signals.
- **Macro Research**: Top-down analysis of interest rates, FX, geopolitics, and economic cycles.

#### Portfolio Construction
- **Strategic Asset Allocation (SAA)**: Long-term policy weights based on investor risk/return objectives.
- **Tactical Asset Allocation (TAA)**: Short-term deviations from SAA based on market views.
- **Optimization**: Mean-variance optimization, risk-parity, Black-Litterman model, factor tilting.
- **Constraints**: Liquidity requirements, regulatory restrictions, ESG mandates, concentration limits.

#### Order Generation
PMs and analysts generate investment decisions that flow to traders as orders. Orders specify instrument, side (buy/sell), quantity, limit/stop conditions, and execution instructions.

#### Key Front Office Systems
- **Portfolio Management Systems (PMS)**: Aladdin (BlackRock), Bloomberg AIM, MSCI BarraOne, FactSet, SimCorp Dimension.
- **Research Platforms**: Bloomberg Terminal, FactSet, Refinitiv Eikon (now LSEG Workspace), AlphaSense, Visible Alpha.
- **Order Management Systems (OMS)**: Bloomberg AIM, Charles River (CRD), ThinkFolio (SS&C), Moxy (for fixed income).
- **Execution Management Systems (EMS)**: FlexTrade, Portware, Eze EMS (SS&C), Bloomberg EMSX — used by traders for multi-asset execution.

### 2.2 Middle Office

**Pre-Trade Compliance → Risk Monitoring → TCA (Transaction Cost Analysis)**

The Middle Office sits between FO and BO, ensuring investment decisions are executed within defined boundaries.

#### Pre-Trade Compliance
- **Rule-Based Restrictions**: Position limits (max 5% in any single issuer), sector concentration limits, liquidity constraints, leverage caps.
- **Regulatory Limits**: UCITS 10% diversification rule, AIFMD leverage limits, SEC 1940 Act requirements.
- **ESG Restrictions**: Exclusion lists (controversial weapons, thermal coal), norms-based screening, engagement tracking.
- **Guideline Monitoring**: Real-time checking every order against fund mandate rules. Systems: Charles River Compliance, Bloomberg AIM Compliance, LPA (Horn Norum), Aladdin.

#### Risk Monitoring
- **Market Risk**: Value-at-Risk (VaR), tracking error, factor exposure (Barra), Greeks (options).
- **Credit Risk**: Spread duration, default probability, recovery rates, counterparty risk (CSA, ISDA agreements).
- **Liquidity Risk**: Bid-ask spreads, position size vs ADV, redemption gates.
- **Operational Risk**: Trade failure rates, settlement breaks, NAV errors.
- **Stress Testing**: Scenario analysis (rate shock, equity crash, credit event, FX crisis), reverse stress testing.
- **Systems**: MSCI RiskManager (BarraOne), RiskMetrics, Bloomberg PORT, Aladdin Risk, Axioma.

#### Transaction Cost Analysis (TCA)
- **Pre-Trade TCA**: Estimated market impact, timing risk, spread cost before execution.
- **Post-Trade TCA**: Implementation shortfall vs arrival price, VWAP/ TWAP slippage, broker comparison.
- **Systems**: Bloomberg TCA, ITG ACE, Abel Noser, BEST Execution (UBS), FlexTrade TCA.

### 2.3 Back Office

**Trade Confirmation/Affirmation → Settlement → Custody → Performance Reporting**

The Back Office ensures trades settle correctly, assets are safeguarded, and investors receive accurate reporting.

#### Trade Confirmation & Affirmation
- **Trade Capture**: Trades flow from OMS/EMS to BO systems (often via FIX or proprietary API).
- **Affirmation**: Institutional trades are affirmed electronically through **Omgeo CTM** (now part of DTCC) or **MarkitServ** (now IHS Markit) for OTC derivatives. T+0/T+1 affirmation is increasingly automated.
- **Confirmation**: Matching trade details between counterparties. Mismatches (unmatched trades) require manual resolution before settlement.

#### Settlement
- **Lifecycle**: Trade date (T) → Settlement date (T+1 for equities in most markets, T+2 for some fixed income, T+0 for FX swaps). US equities moved to T+1 in May 2024.
- **Mechanisms**: Central Securities Depositories (CSDs) — DTCC (US), Euroclear (EU), Clearstream (EU). Payments via SWIFT MT messages (MT202, MT103).
- **SWIFT Standards**: Transitioning from legacy MT to **ISO 20022** (structured XML) across payments, securities, and reporting. CBPR+ went live for cross-border payments in 2025.
- **Fails**: When settlement fails for insufficient securities/cash. Managed via buy-in procedures (CSDR in EU, SEC Rule 204).

#### Custody
- **Global Custodians**: BNY Mellon, State Street, J.P. Morgan, Citibank, HSBC, BNP Paribas, CACEIS, Societe Generale Securities Services (SGSS).
- **Sub-Custody**: Local banks in each market used by global custodians for local settlement and safekeeping.
- **Services**: Safekeeping of assets, income collection, corporate actions processing, tax reclamation, securities lending, FX, collateral management.

#### Performance Reporting
- **Returns**: Time-Weighted Return (TWR) — standard for fund performance, removes cash flow timing. Money-Weighted Return (MWR / IRR) — used for private assets.
- **Attribution**: Brinson performance attribution (allocation + selection + interaction effects), factor attribution (Carhart 4-factor, Fama-French).
- **Benchmarking**: Comparison vs standard indices (S&P 500, Bloomberg Agg, MSCI ACWI) or custom benchmarks.
- **Reporting Pack**: Monthly/quarterly fact sheets, portfolio holdings, performance commentary, ESG metrics, risk statistics.

---

## 3. Alternatives Management

### 3.1 What Are Alternatives?

Alternative investments are asset classes outside traditional public equity, fixed income, and cash. They are characterized by:

- **Illiquidity Premium**: Higher expected returns in exchange for locking up capital for extended periods.
- **Longer Lock-Up Periods**: Quarterly, annual, or multi-year redemption restrictions.
- **Less Regulation**: Not subject to UCITS rules in the EU; governed by AIFMD, SEC private placement rules, or local equivalents.
- **Higher Fees**: Management fees of 1.0–2.0% + performance/carry (typically 20% of profits).
- **Lower Transparency**: Portfolio holdings disclosed less frequently (quarterly or annually).

**Global Alternatives AUM**: Estimated at ~$20+ trillion as of 2025, the fastest growing segment of asset management. Private markets revenues projected to reach $432 billion by 2030, delivering over half of the global asset management industry's revenues (PwC 2025).

### 3.2 Private Equity

Private equity involves investing in private companies (or taking public companies private) with the goal of improving operations and exiting at a profit.

#### The PE Lifecycle

1. **Fundraising**: GP raises capital from LPs (pension funds, endowments, insurance, sovereign wealth funds, family offices). Typical fund life: 10 years (5-year investment period + 5-year harvesting period).
2. **Deal Sourcing**: Proprietary network, investment banks, M&A advisors, industry contacts. Auctions vs bilateral deals.
3. **Due Diligence**: Financial, commercial, operational, legal, regulatory, ESG, and IT due diligence. 3–6 months typically.
4. **Acquisition**: Structured via SPV, financed with equity (30–40%) and debt (60–70%). LBO (leveraged buyout) mechanics, syndicated loans, high-yield bonds.
5. **Value Creation**: Operational improvement, margin expansion, buy-and-build (add-on acquisitions), management change, digital transformation, cost optimization.
6. **Exit**: IPO (primary/secondary), trade sale (strategic acquirer), secondary buyout (another PE firm), dividend recapitalization.

#### Waterfall Distribution Mechanics

The waterfall determines how profits flow between GP and LPs:

1. **Return of Capital**: 100% of distributions go to LPs until they have received their total capital contribution.
2. **Preferred Return (Hurdle)**: Next distributions go to LPs until they achieve a preferred return (typically 7–8% IRR).
3. **GP Catch-Up**: Once LPs receive their preferred return, 100% of distributions go to the GP until the GP has received 20% of total profits (or the catch-up level).
4. **Carried Interest Split**: Remaining profits split 80/20 (LP/GP) in the standard "European waterfall" (whole-fund basis).

Two styles exist:
- **European (Whole-Fund) Waterfall**: Preferred by ILPA. Carry calculated across the entire portfolio. Early winners subsidize later losses.
- **American (Deal-by-Deal) Waterfall**: Carry calculated per deal. More favorable to GPs but less common in institutional PE.

#### GP / LP Arrangements

- **General Partner (GP)**: The fund manager. Contributes 1–5% of fund capital. Receives management fee (1.5–2.0% of committed capital) and carried interest (20% of profits).
- **Limited Partners (LPs)**: Institutional investors. Capital commitments called over time via capital calls. No management control over day-to-day operations.
- **LP Advisory Committee (LPAC)**: Advisory body that consents to conflicts of interest, fund extensions, and key-person changes.

#### Key PE Systems
- **Dynamo Software**: Cloud-based CRM, investor relations, and fund management.
- **Investran** (SS&C): End-to-end accounting, waterfall, and reporting for private markets.
- **iLevel** (SS&C): Portfolio monitoring, valuation, and analytics platform.
- **Cobalt** (now part of FIS): Private capital markets data and analytics.
- **eFront** (BlackRock): Alternative investment management platform (PE, real estate, infrastructure).
- **Allvue Systems** (formerly FIS Private Capital Suite): Waterfall, accounting, and LP reporting.

### 3.3 Private Credit / Direct Lending

**Scale**: The private credit market has grown rapidly, driven by banks retreating from corporate lending post-GFC and post-SVB. Estimated at $1.5–$2.0 trillion globally as of 2025.

**Structure**: Floating rate, senior secured loans to middle-market companies (typically $10M–$500M EBITDA). Includes:
- **Direct Lending**: Bilateral or club loans to mid-market companies.
- **Mezzanine**: Subordinated debt with equity warrants/kickers.
- **Distressed / Special Situations**: Debt of companies in financial difficulty.
- **Asset-Based Lending (ABL)**: Loans secured against receivables, inventory, equipment.
- **Private CLOs**: Bespoke collateralized loan obligations managed by credit specialists.

**Process**: Origination (direct sourcing or through sponsor relationships) → Underwriting (credit analysis, covenant structuring, collateral evaluation) → Syndication (club deals, participate to other lenders) → Servicing (covenant monitoring, amendments, payment collection).

**Key Players**: Ares Management, Blue Owl Capital, Golub Capital, HPS Investment Partners, Oaktree, Middle Market Direct Lending funds.

### 3.4 Real Estate & Infrastructure

#### Real Estate
- **Property Types**: Office (challenged post-COVID), retail (omnichannel adaptation), residential / multifamily (resilient), industrial/logistics (strong demand), data centers (booming from AI/cloud), life sciences, hotels.
- **Strategies**: Core (stabilized, low leverage, income focus), Core-Plus (moderate value-add), Value-Add (repositioning, renovations), Opportunistic (development, land, highest risk/return).
- **REITs**: Real Estate Investment Trusts — publicly traded (listed on exchanges) or private. Must distribute 90%+ of taxable income. Provide liquid exposure to real estate.
- **Valuation Approaches**: DCF (discounting NOI), comparable cap rates (NOI / purchase price), replacement cost.

#### Infrastructure
- **Sub-Sectors**: Transport (tolls, airports, ports, rail), Energy (renewables: wind, solar, hydro; conventional: pipelines, storage), Digital (data centers, fiber, towers), Social (hospitals, schools, PPPs).
- **Characteristics**: Long-dated, inflation-linked cash flows, high barriers to entry, essential services, low correlation to public markets.
- **Brownfield vs Greenfield**: Brownfield = operating assets (lower risk, stable yield). Greenfield = construction/development (higher risk, higher return).

**Systems for Real Estate & Infra**: Yardi, MRI Software, Altus Group, eFront (BlackRock), Investran, JD Edwards (Oracle).

### 3.5 Hedge Funds

**Structure**: Private investment pools using a broad range of strategies, typically with performance fees (2 and 20 model — 2% management fee, 20% performance fee — is now less common; many have moved to 1.5/15 or lower). Lock-up periods: quarterly to annual redemptions, often with gates and side pockets.

**Major Strategies**:

| Strategy | Description | Key Examples |
|----------|-------------|--------------|
| Long/Short Equity | Buy undervalued, short overvalued stocks | Citadel, D.E. Shaw, Millennium, Point72 |
| Global Macro | Directional bets on FX, rates, commodities | Bridgewater, Brevan Howard, Caxton |
| Event-Driven | M&A arbitrage, distressed, special sits | Paulson, Elliott, York Capital |
| Relative Value | Fixed income arb, convertible arb, Vol arb | PDT (Morgan Stanley), Capula |
| Quant / Systematic | High-frequency, statistical arb, trend-following | Renaissance (Medallion), Two Sigma, AQR |
| Multi-Strategy | Diversified across pods/desks within one fund | Citadel, Millennium, D.E. Shaw, Balyasny |

**Hedge Fund Operations**:
- **Prime Brokerage**: Services from investment banks (Goldman Sachs, Morgan Stanley, J.P. Morgan, Credit Suisse/now UBS) — trade execution, financing (margin lending), securities lending for short sales, custody, capital introduction.
- **Margin Financing**: Funds borrow against portfolio assets for leverage (typical 1.5x–3x for multi-strat).
- **Short Selling**: Borrow stock from prime broker, sell, buy back later at lower price. Hard-to-borrow fees and locate requirements.
- **Collateral Management**: Variation margin cash flows on derivatives (cleared and OTC), initial margin posting (ISDA SIMM).

**Key Hedge Fund Systems**:
- **Paladyne** (Broadridge / Itiviti): EMS/OMS for hedge funds, multi-asset.
- **Eze EMS** (SS&C): Leading EMS for long/short equity, integrates with prime brokers.
- **Advent Geneva** (SS&C): Portfolio accounting and reporting, multi-currency, multi-entity.
- **Kubero** (Broadridge): Fund accounting and NAV for alternatives.
- **Bloomberg Enterprise** (Trading, AIM, TOMS).

### 3.6 Digital Assets / Crypto Funds

**Growth**: Institutional adoption has increased significantly. Spot Bitcoin ETFs approved in the US (Jan 2024), Ethereum ETFs (mid-2024), and growing Asian and Middle Eastern regulatory clarity. Crypto hedge funds and venture funds have re-emerged after the 2022 downturn.

**Challenges**:
- **Custody**: Self-custody/qualified custodians (Coinbase Custody, BitGo, Fidelity Digital Assets, BNY Mellon digital custody). Hot/cold wallet segregation, multi-signature security.
- **Regulation**: SEC (US) — evolving; MAS (Singapore) — Payment Services Act for Digital Payment Tokens; EU — MiCA (Markets in Crypto-Assets) effective 2024–2025.
- **Valuation**: 24/7 markets, extreme volatility, limited reliable pricing sources.
- **Fund Structures**: Often set up as standalone VCC (Singapore), Cayman Islands exempted companies, or Delaware LLCs.

---

## 4. Key Differences: Traditional AM vs Alternatives

| Dimension | Traditional AM | Alternatives |
|-----------|---------------|--------------|
| **Liquidity** | Daily dealing (T+1/2 settlement) | Quarterly, annual, or multi-year lock-ups; side pockets |
| **Valuation** | Mark-to-market — observable prices | Mark-to-model — appraisals, DCF, broker quotes (quarterly) |
| **Fee Structure** | Management fee only (0.01–1.20%) | Management fee (1.0–2.0%) + Performance/Carry (15–20%) |
| **Regulation** | UCITS, 40 Act, SFA (retail/wholesale) | AIFMD, SEC D Rules, private placement, QIB accreditation |
| **Operations** | High volume, low complexity; automated | Low volume, high complexity; manual-heavy, bespoke |
| **Reporting** | Standardized periodic packs | Waterfall calc, IRR, TVPI, DPI, RVPI, capital call notices |
| **Systems** | Bloomberg AIM, Charles River, Aladdin, SimCorp | Dynamo, Investran, iLevel, eFront, Advent Geneva |
| **Leverage** | Limited (UCITS max 2x gross/10% derivatives) | Often significant (PE: 2–4x debt/EBITDA; HF: 2–5x NAV) |
| **Transparency** | Holdings disclosed monthly/quarterly | Holdings disclosed annually (PE), quarterly (HF) |
| **Data** | High-frequency pricing, benchmark indices | Stale valuations, deal-level financials, LP letters |

---

## 5. Key Operational Processes

### 5.1 NAV Calculation

- **Traditional Funds**: Daily NAV calculated by fund administrators or in-house teams. Process: price all portfolio securities at market close → accrue income (dividends, coupons) → accrue expenses (management fees, admin fees, custody fees) → add/subtract capital flows (subscriptions, redemptions) → calculate NAV per share.
- **Alternative Funds**: NAV calculated quarterly (most PE/RE) or monthly (some HF). Requires valuation of illiquid holdings via independent valuation agents (IVAs), appraisals, or DCF models. Valuations are subject to IPEV guidelines (International Private Equity and Venture Capital Valuation guidelines).

### 5.2 Investor Reporting

- **Traditional Funds**: Monthly fact sheets (performance, top holdings, sector breakdown, fees) typically within 10 business days of month-end.
- **Alternative Funds**: Quarterly or annual reporting. Includes:
  - **PE/RE**: Capital account statements, NAV bridge, IRR since inception, TVPI (Total Value to Paid-In), DPI (Distributions to Paid-In), RVPI (Residual Value to Paid-In).
  - **Hedge Funds**: Performance summary, risk metrics, exposure report, attribution, liquidity profile.
  - **ESG Reporting**: Increasingly required — SFDR (EU), Article 8/9 disclosures, TCFD, SASB.

### 5.3 Compliance Monitoring

- **Pre-Trade Compliance**: Position/sector/duration limits, mandate rule checks, restricted list screening.
- **Post-Trade Compliance**: Regulatory filings (SEC Form 13F, Form PF, Form ADV; FCA; AIFMD Annex IV). Best execution reporting (MiFID II RTS 28).
- **AML/KYC**: Onboarding investors — CDD (Customer Due Diligence), EDD (Enhanced Due Diligence) for PEPs, sanctions screening (OFAC, UN, EU), source of wealth/verification.
- **Systems**: Fircosoft, World-Check (Refinitiv/LSEG), Dow Jones Risk & Compliance, LexisNexis Bridger.

### 5.4 Corporate Actions

Corporate actions are events initiated by issuers that affect the securities held in a portfolio.

**Mandatory Events** (no shareholder choice):
- Cash dividends
- Stock splits / reverse splits
- Bonus issues
- Maturity / redemption

**Voluntary Events** (shareholder choice required):
- Rights issues (entitlement)
- Tender offers (buybacks)
- Merger / acquisition elections (cash/stock/mixed)
- Warrants conversion
- Proxy votes (voting instruction)

**Operational Process**:
1. **Announcement**: Custodian receives details via SWIFT (MT564/ISO 20022 Seeval).
2. **Validation**: Reconciliation against issuer/agent terms.
3. **Election Deadline**: For voluntary events, fund managers must elect by stated deadline.
4. **Entitlement Calculation**: How many rights/cash each position is entitled to.
5. **Settlement**: Income collection (dividends, coupons) or delivery of new securities.
6. **Tax**: Withholding tax treatment, reclaim filing, treaty benefits.

**Key Challenge in Alternatives**: Unlisted assets (PE, RE) require manual corporate actions tracking — no automated SWIFT flow — making it operationally intense.

### 5.5 Performance & Attribution

- **Time-Weighted Return (TWR)**: Eliminates effect of cash flows. Standard for public funds (GIPS compliant). Links sub-period returns geometrically.
- **Money-Weighted Return (MWR / IRR)**: Accounts for size and timing of cash flows. Standard for private assets (PE, RE, infrastructure). Modified Dietz for approximate calculations.
- **Brinson Attribution**: Decomposes portfolio returns vs benchmark into Allocation Effect (sector/region weight differences) + Selection Effect (security selection within sectors) + Interaction Effect.
- **Factor Decomposition**: Barra risk model factors (Value, Momentum, Size, Volatility, Quality, Yield, Growth).
- **GIPS (Global Investment Performance Standards)**: Created by CFA Institute. Required standard for all firms presenting compliant track records.

### 5.6 Tax & Legal

- **FATCA (US)**: Foreign Account Tax Compliance Act. Non-US funds must report US investors or face 30% withholding.
- **CRS (Common Reporting Standard)**: OECD-driven automatic exchange of financial account information. Fund must classify each investor and report to local tax authority.
- **Treaty Reclaims**: Non-resident investors may reclaim excess withholding tax using double tax treaties. Managed by custodians or specialized reclaim agents.
- **Fund Domiciles**:
  - **Luxembourg**: UCITS and AIF/RIF (SICAV, SIF, RAIF). Most popular EU fund domicile.
  - **Ireland**: ICAV (Irish Collective Asset-Management Vehicle). QIAIF / RIAIF.
  - **Cayman Islands**: Exempted Company / LLC. Dominant for hedge funds and PE.
  - **Singapore VCC**: Variable Capital Company — increasingly popular (see Section 7).
  - **Delaware / BVI / Jersey / Guernsey**: Also common for specific structures.

---

## 6. Technology & Architecture

### 6.1 System Landscape

| Function | Systems |
|----------|---------|
| **OMS** | Bloomberg AIM, Charles River, ThinkFolio (SS&C), Moxy, LongView |
| **EMS** | FlexTrade, Portware, Eze EMS, Bloomberg EMSX, Flextrade |
| **PMS** | Aladdin (BlackRock), Bloomberg AIM, FactSet, MSCI BarraOne |
| **Risk** | MSCI RiskManager (BarraOne), Bloomberg PORT/RISK, RiskMetrics, Axioma |
| **Fund Accounting** | SS&C Advent (APX, Geneva), Investran (PE), eFront, Dynamo |
| **TCA** | Bloomberg TCA, ITG ACE, Abel Noser, FlexTrade TCA |
| **Compliance** | Charles River Compliance, Bloomberg AIM Compliance, LPA (Horn Norum) |
| **Investor Portal** | Dynamo, Allvue, Intralinks, Waterhouse, IHS Markit |
| **Data Management** | Bloomberg Data License, FactSet, Refinitiv, ICE Data Services, Xignite |

### 6.2 Integration Protocols

- **SWIFT**: The global standard for interbank financial messaging. MT (legacy) and ISO 20022 (next-gen, MX format) message types. Used for settlement, custody, corporate actions, FX. Key types: MT300 (FX confirmations), MT500 series (securities settlement), MT564/566 (corporate actions), MT536 (triparty collateral).
- **FIX Protocol**: Financial Information eXchange. The standard for electronic trading — pre-trade (orders, quotes), trade (execution reports), post-trade (allocations, confirmations). Used between funds, brokers, and exchanges. FIX 5.0 SP2 / FIX Latest is current.
- **ISO 20022**: Sweeping global migration (SWIFT CBPR+ 2025). Structured XML envelopes. Covers: pain. (payment initiation), pacs. (clearing/settlement), camt. (cash management), seeval/seev. (securities settlement/corporate actions).
- **SFTP / API**: Custodian file delivery — daily transaction reports, valuation files, trial balances. Increasingly RESTful APIs replacing flat-file SFTP.

### 6.3 Reference Data

- **Security Master**: Central repository of instrument static data — ISIN, CUSIP, SEDOL; asset class, maturity, coupon; corporate actions history; pricing sources.
- **Counterparty Master**: Legal entity identifiers (LEIs), counterparty credit ratings, CSA/ISDA terms, settlement instructions (SSIs).
- **Pricing Feeds**: Bloomberg B-Pipe/PRICING, Refinitiv/BENCHMARK, ICE Data, SuperDerivatives (derivatives), consensus pricing platforms (MarkIt, Total Derivatives).
- **Market Data**: Bloomberg, Refinitiv Eikon, FactSet, Dow Jones, MSCI ESG Research.

### 6.4 Architecture Trends

- **Cloud Migration**: Firms moving from on-prem to cloud (AWS, Azure, GCP). SaaS offerings (Dynamo cloud-native, eFront cloud, Charles River SaaS).
- **Data Lakes & Data Fabric**: Centralizing portfolio, reference, market, and ESG data. Snowflake, Databricks, AWS S3 + Glue for data engineering.
- **AI / ML**: Alpha generation (sentiment analysis, alternative data integration), automation (smart document processing for corporate actions, NAV reconciliation), fraud detection, TCA pattern analysis.
- **API-First Architecture**: Microservices replacing monolithic OMS/PMS. OpenAPI/REST and FIX APIs for linking best-of-breed components.
- **Real-Time Processing**: T+1 settlement in US/Canada (2024), T+0 ambition in some markets (India, China) requires real-time trade matching, risk checks, and settlement workflows.

---

## 7. Singapore Context

### 7.1 Singapore as an Asset Management Hub

Singapore has established itself as Asia's leading asset management center alongside Hong Kong. As of the latest MAS surveys:

- **Total AUM**: ~S$5.4+ trillion (2024), growing ~10% annually. Growth led by alternatives — PE, VC, hedge funds, and private credit driving 14% increase in alternative AUM.
- **Asset Mix**: Approximately 80% sourced from outside Singapore (cross-border), confirming Singapore's role as an international gateway.
- **Workforce**: Over 15,000 asset management professionals employed.

### 7.2 Variable Capital Company (VCC)

Introduced by the Variable Capital Companies Act in 2020, the VCC is Singapore's flagship fund vehicle:

**Key Features**:
- **Variable Capital**: Shares can be created/redeemed without shareholder approval — simplifies subscription/redemption processing.
- **Umbrella Structure**: Single VCC can house multiple sub-funds with segregated assets and liabilities (protected cell company concept). Enables cost sharing across strategies.
- **Redomiciliation**: Foreign funds can migrate to Singapore VCC structure (Cayman, BVI, Luxembourg VCCs have migrated).
- **No Public Filing of Holdings**: Unlike Irish/UCITS equivalents, VCC holdings are not publicly filed — appealing to institutional/private investors.

**2025 Regulatory Updates**:
- Lifetime tax exemption introduced for qualifying VCC funds.
- Greater flexibility for umbrella VCCs to launch and terminate sub-funds.
- Enhanced anti-money laundering requirements for fund managers.

### 7.3 MAS Regulation

- **Securities and Futures Act (SFA, Cap. 289)**: Primary legislation for fund management activities in Singapore.
- **Capital Markets Services (CMS) License**: Required for fund management. Types:
  - **RFMC (Registered Fund Management Company)**: For up to 30 qualified investors, AUM < S$250M. No longer accepting new applications after phased licensing changes.
  - **LFMC (Licensed Fund Management Company)**: For retail/Accredited/Institutional investors. Stricter capital requirements (S$500K–S$1M base capital, S$50M–S$200M+ regulatory capital depending on AUM).
  - **VCFM (Venture Capital Fund Manager)**: Exempt from certain SFA requirements. AUM < S$250M, only in VC.
- **Variable Capital Companies Act (VCCA)**: Governs VCC setup, operation, and dissolution.
- **Anti-Money Laundering / Countering Financing of Terrorism (AML/CFT)**: MAS Notice SFA 04-N02. Source of wealth/verification, ongoing transaction monitoring, suspicious transaction reporting.

### 7.4 Tax Incentives

Singapore offers attractive tax schemes for fund managers:

- **Section 13O (formerly 13R)**: Tax exemption for onshore Singapore-managed funds (VCC or trust structure). Conditions: minimum AUM S$50M (as of 2025), minimum annual local business spending S$200K, minimum 2 investment professionals in Singapore.
- **Section 13U (formerly 13X)**: Tax exemption for funds of any form (onshore, offshore, VCC, LP, trust). Conditions: minimum AUM S$50M, minimum local business spend S$200K annually, minimum 3 investment professionals in Singapore (at least 1 non-executive director). Must be approved by MAS — "enhanced tier" fund.
- **Section 13D**: For resident funds managed outside Singapore (non-Singapore managed).
- **Section 13OA**: Introduced 2024–2025 — tax incentive for single-family offices under the Variable Capital Company regime.

### 7.5 Major Singapore-Based Asset Managers

| Firm | Type | AUM (approx) | Notes |
|------|------|--------------|-------|
| **GIC** | Sovereign Wealth Fund | ~S$800B+ | Manages Singapore's foreign reserves |
| **Temasek Holdings** | Sovereign Investment Co | ~S$500B+ | Owns Mphasis, Singtel, DBS, SMRT stakes |
| **Fullerton Fund Mgmt** | Active Multi-Asset | ~S$50B+ | Temasek-linked, income-focused |
| **UOB Asset Management** | Bank-owned | ~S$40B+ | UOB retail + institutional funds |
| **DBS Asset Management** | Bank-owned | ~S$35B+ | DBS retail mandates + ETF range |
| **Eastspring Investments** | Prudential plc subsidiary | ~S$250B+ | Asia-focused, largest foreign-owned manager in SG |
| **Nikko Asset Mgmt (Asia)** | Regional HQ | ~S$200B+ | Japan parent, Asia ex-Japan focus |
| **Seviora Group** | Multi-boutique | ~S$100B+ | Temasek's asset management group (Fullerton, Seviora AM, Griffin) |
| **APS Asset Mgmt** | Independent | ~S$5B+ | Asia focused credit and equity |
| **Novo Tellus** | PE | ~S$3B+ | Southeast Asian tech buyouts |

### 7.6 Singapore as an Alternatives Hub

Singapore is rapidly growing as a hub for alternative fund management, particularly for:

- **Private Credit**: Southeast Asian direct lending — infrastructure financings, mid-market sponsor loans, trade finance. Several global private credit managers have set up Singapore booking centers.
- **Real Estate**: Major RE market (CapitaLand, GIC Real Estate, Mapletree) and growing data center / logistics investments.
- **Infrastructure**: SG as base for ASEAN infrastructure funds (renewables, toll roads, ports). Keppel, Sembcorp, GIC Infra.
- **Family Offices**: MAS reported over 1,400 single-family offices by 2025, driving demand for alternatives exposure, co-investment platforms, and multi-family office services.
- **Fintech / Crypto**: MAS issued digital payment token (DPT) licenses under Payment Services Act. Several crypto fund managers and tokenization platforms based in SG.

---

## 8. Glossary

| Term | Definition |
|------|-----------|
| **AUM** | Assets Under Management — total market value of assets managed by a firm |
| **Alpha** | Excess return relative to a benchmark, attributed to manager skill |
| **Carried Interest (Carry)** | GP's share of profits, typically 20% after LP return of capital + hurdle |
| **DPI** | Distributions to Paid-In — realized returns in PE |
| **EMS** | Execution Management System — used by traders to execute orders |
| **ESG** | Environmental, Social, Governance — sustainable investing criteria |
| **FIX** | Financial Information eXchange — electronic trading protocol |
| **GP** | General Partner — fund manager |
| **IRR** | Internal Rate of Return — money-weighted return metric |
| **LP** | Limited Partner — fund investor |
| **NAV** | Net Asset Value — fund's assets minus liabilities, per share |
| **OMS** | Order Management System — for order generation and routing |
| **PE / VC** | Private Equity / Venture Capital |
| **PMS** | Portfolio Management System — for portfolio construction and analytics |
| **REIT** | Real Estate Investment Trust — publicly/privately traded property vehicle |
| **RVPI** | Residual Value to Paid-In — unrealized returns in PE |
| **SWIFT** | Society for Worldwide Interbank Financial Telecommunication — messaging network |
| **TCA** | Transaction Cost Analysis — measuring execution quality |
| **TVPI** | Total Value to Paid-In (DPI + RVPI) — total return multiple in PE |
| **TWR** | Time-Weighted Return — standardized performance measure |
| **UCITS** | Undertakings for Collective Investment in Transferable Securities — EU retail fund regime |
| **VaR** | Value at Risk — statistical risk measure |
| **VCC** | Variable Capital Company — Singapore fund vehicle |

---

## References & Further Reading

1. **McKinsey (2025)**: *Asset Management 2025: The Great Convergence*
2. **BCG (2026)**: *Global Asset Management Report: An Imperative for Growth*
3. **PwC (2025)**: *Global Asset & Wealth Management Report — Private Markets to Account for More Than Half of Industry Revenues by 2030*
4. **EY (2024)**: *Global Alternative Fund Survey — Shaping New Horizons*
5. **MAS (2024)**: *Singapore Asset Management Survey* — Monetary Authority of Singapore
6. **Callan (2025)**: *Investment Management Fee Study*
7. **ILPA (2024)**: *Private Equity Principles* — Institutional Limited Partners Association
8. **IPEV (2022)**: *International Private Equity and Venture Capital Valuation Guidelines*
9. **CFA Institute**: *Global Investment Performance Standards (GIPS)*
10. **Broadridge (2024)**: *US & European Fund Fee Trends*

---

> **Maintained by Jack Liu Shurui** — Singapore-based finance and technology professional.  
> This guide is part of a personal research repository. Corrections, updates, and contributions welcome via GitHub issues or PRs.
