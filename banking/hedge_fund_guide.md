# Hedge Funds: A Comprehensive Guide

**The Mechanics of the Absolute-Return Fund — the Definition and Structure, the 1949–2026 History, the Strategy Taxonomy, Fund Economics (Fees, High-Water Marks, Hurdles, Gates), the Service Stack and the Lifecycle, the Global Industry (HFR, AUM, the Largest Firms), the Banking Relationships, Risk and Performance, a Comparison Table, and a Cymbal Bank Worked Example of an Asia Long/Short Fund**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Banking Domain / Institutional Investment & Capital Markets — the general hedge-fund deep-dive: what a hedge fund is (a pooled, private, periodically-liquid vehicle with an absolute-return mandate), the verified history from A.W. Jones's 1949 partnership through the 1966 Fortune article that coined the term, the 1992 ERM crisis, LTCM 1998, the 2008 crisis and Madoff, to the record $5.15 trillion industry of end-2025; the strategy taxonomy (long/short equity, market-neutral, event-driven, global macro, relative value, CTA/managed futures, quant/systematic, multi-strategy); the fund economics (management and performance fees, the 2-and-20 convention and its compression, high-water marks, hard and soft hurdles, lock-ups, redemption terms, gates, side pockets); the structure (GP/LP, the service stack, master-feeder, UCITS) and the lifecycle (launch, seeding, growth, wind-down); the global industry (HFR/HFRI indices, the largest firms, geography, the Singapore hub); the banking relationships (prime brokerage, custody, fund administration, fund financing, KYC/AML); risk (drawdown, VaR, stress testing, the 2008 redemption spiral) and performance (measurement, benchmarks, GIPS, the alpha/beta debate, persistence); a hedge-fund-vs-PE-vs-mutual-fund-vs-CTA comparison; and a Cymbal Bank worked example of a fictional Asia-focused long/short fund as a prime-brokerage, custody, fund-finance and FX client. The companion Singapore-landscape guide is [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) — this guide is its general sibling: the fund itself, not the jurisdiction.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** hfr.com (HFR Global Hedge Fund Industry Report press releases — Year-End 2025, 22 January 2026; 3Q25, 23 October 2025; 2Q26, 23 July 2026; July 2026 flash), eur-lex.europa.eu (Council Directive 85/611/EEC of 20 December 1985 — UCITS), efama.org (EFAMA Fact Book 2025 and the February-2025 Fact Sheet), sec.gov (SEC Litigation Release No. 20889 and Press Release 2008-293 — Madoff), Wikipedia (Alfred Winslow Jones; Carol Loomis; Hedge fund; Long-Term Capital Management; Black Wednesday; Madoff investment scandal; Drawdown; Value at risk) with its primary citations, InvestmentNews/Pensions & Investments (largest-firms ranking, February 2026, June-2025 data), Investopedia and Databento (gate provisions), Wiley/Springer academic literature (performance persistence), and the sibling repo guides [Hedge Funds in Singapore](hedge_funds_singapore_guide.md), [Citadel LLC](citadel_llc_guide.md), [Private Equity](private_equity_guide.md), [Cayman & BVI Master-Feeder Structures](cayman_bvi_master_feeder_guide.md), [Market Making in Singapore](market_making_singapore_guide.md), [Investment Portfolio Operations](investment_portfolio_operations_guide.md), [Enterprise Risk Management](enterprise_risk_management_guide.md), [FircoSoft](fircosoft_guide.md)
> **Last Updated:** August 2026
> **Companion guides:** [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) (the SG landscape and the MAS regime — the sibling to this guide) · [Citadel LLC](citadel_llc_guide.md) (the multi-strategy archetype; §10 the prime-brokerage worked example) · [Private Equity](private_equity_guide.md) (carry mechanics; §9 the subscription-line worked example) · [Cayman & BVI Master-Feeder Structures](cayman_bvi_master_feeder_guide.md) (offshore structures) · [Market Making in Singapore](market_making_singapore_guide.md) (§10 the PB worked example) · [Investment Portfolio Operations](investment_portfolio_operations_guide.md) (§7.1 GIPS) · [Enterprise Risk Management](enterprise_risk_management_guide.md) (stress testing) · [FircoSoft](fircosoft_guide.md) (sanctions screening) · [Private Banking](private_banking_guide.md)

---

**How to use this guide:** Section 1 is the definition — the short answer, the four structural features, the key-facts table, and why a hedge fund matters to a bank. Section 2 is the history — Jones 1949, the 1966 Fortune article, the 1968–1970 shakeout, the global-macro era and Black Wednesday, LTCM 1998, the 2008 crisis and Madoff, and the 2020s. Section 3 is the strategy taxonomy — each strategy's definition, return driver, and a comparison table. Section 4 is the fund economics — fees, the 2-and-20 convention and its compression, high-water marks, hurdles, lock-ups, redemptions, gates, side pockets. Section 5 is the structure and lifecycle — the service stack, master-feeder, UCITS, and the launch-to-wind-down arc. Section 6 is the global industry — the verified HFR AUM figures, the HFRI indices, the largest firms, geography, and the Singapore hub. Section 7 is the banking relationships — prime brokerage, custody and administration, fund financing, KYC/AML. Section 8 is risk; Section 9 is performance. Section 10 is the comparison table. Section 11 is the Cymbal Bank worked example. Section 12 is the claims audit (✅/⚠/❌) and "What Could Not Be Verified"; Section 13 is the glossary, references, and the closing line. Cross-references follow the repository convention: sibling guides in `banking/` are plain filenames. **Integrity convention:** ✅ = verified this pass against a primary or cited source (source named in §12); ⚠ = flagged/unverified — press estimate, contested, or not re-verified live; ❌ = refuted or rejected. Nothing in this guide was invented; figures that could not be re-verified are marked ⚠ and listed again in §12.

---

## Table of Contents

1. [What a Hedge Fund Is — the Pooled, Private, Liquid Absolute-Return Vehicle](#1-what-a-hedge-fund-is--the-pooled-private-liquid-absolute-return-vehicle)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Four Structural Features](#12-the-four-structural-features)
   - 1.3 [The Key-Facts Table](#13-the-key-facts-table)
   - 1.4 [Why a Hedge Fund Matters to a Bank](#14-why-a-hedge-fund-matters-to-a-bank)
2. [The History — 1949 to the 2020s](#2-the-history--1949-to-the-2020s)
   - 2.1 [A.W. Jones, 1949 — the First Hedged Fund](#21-aw-jones-1949--the-first-hedged-fund)
   - 2.2 [1966 — Carol Loomis Names the Industry; 1968–1970 — the First Shakeout](#22-1966--carol-loomis-names-the-industry-19681970--the-first-shakeout)
   - 2.3 [The 1980s–90s — Global Macro and Black Wednesday](#23-the-1980s90s--global-macro-and-black-wednesday)
   - 2.4 [LTCM, 1998 — the Fed-Brokered Bailout](#24-ltcm-1998--the-fed-brokered-bailout)
   - 2.5 [The 2000s and the 2008 Crisis — Redemptions, Gates, and Madoff](#25-the-2000s-and-the-2008-crisis--redemptions-gates-and-madoff)
   - 2.6 [The 2010s–2020s — Institutionalisation and the $5 Trillion Industry](#26-the-2010s2020s--institutionalisation-and-the-5-trillion-industry)
3. [The Strategy Taxonomy](#3-the-strategy-taxonomy)
   - 3.1 [The Taxonomy Map — and Why It Varies by Provider](#31-the-taxonomy-map--and-why-it-varies-by-provider)
   - 3.2 [Long/Short Equity](#32-longshort-equity)
   - 3.3 [Market-Neutral](#33-market-neutral)
   - 3.4 [Event-Driven — Merger Arbitrage, Distressed, Special Situations](#34-event-driven--merger-arbitrage-distressed-special-situations)
   - 3.5 [Global Macro](#35-global-macro)
   - 3.6 [Relative Value](#36-relative-value)
   - 3.7 [CTA / Managed Futures](#37-cta--managed-futures)
   - 3.8 [Quantitative / Systematic](#38-quantitative--systematic)
   - 3.9 [Multi-Strategy](#39-multi-strategy)
   - 3.10 [The Strategy Comparison Table](#310-the-strategy-comparison-table)
4. [The Fund Economics — Fees, High-Water Marks, Hurdles, Liquidity](#4-the-fund-economics--fees-high-water-marks-hurdles-liquidity)
   - 4.1 [The Management Fee — on NAV, Not Commitments](#41-the-management-fee--on-nav-not-commitments)
   - 4.2 [The Performance Fee and the High-Water Mark](#42-the-performance-fee-and-the-high-water-mark)
   - 4.3 [The Hurdle — Hard vs Soft](#43-the-hurdle--hard-vs-soft)
   - 4.4 [The 2-and-20 Convention — and Its Compression (Cross-Referenced)](#44-the-2-and-20-convention--and-its-compression-cross-referenced)
   - 4.5 [Lock-Ups, Redemption Terms, and Notice Periods](#45-lock-ups-redemption-terms-and-notice-periods)
   - 4.6 [Gates](#46-gates)
   - 4.7 [Side Pockets](#47-side-pockets)
5. [The Structure and the Lifecycle](#5-the-structure-and-the-lifecycle)
   - 5.1 [The Service Stack — GP, Manager, Fund, Administrator, Auditor, Custodian, Prime Broker](#51-the-service-stack--gp-manager-fund-administrator-auditor-custodian-prime-broker)
   - 5.2 [Master-Feeder and Onshore/Offshore Choice (Cross-Referenced)](#52-master-feeder-and-onshoreoffshore-choice-cross-referenced)
   - 5.3 [The UCITS Wrapper — the 1985 Directive and the European Fund Market](#53-the-ucits-wrapper--the-1985-directive-and-the-european-fund-market)
   - 5.4 [The Lifecycle — Launch, Seeding, Growth, Wind-Down](#54-the-lifecycle--launch-seeding-growth-wind-down)
6. [The Global Industry](#6-the-global-industry)
   - 6.1 [Global AUM — the Verified HFR Figures and the Press Estimates](#61-global-aum--the-verified-hfr-figures-and-the-press-estimates)
   - 6.2 [AUM by Strategy (⚠)](#62-aum-by-strategy-)
   - 6.3 [The Indices — HFR, the HFRI Fund Weighted Composite, and the Benchmark Layer](#63-the-indices--hfr-the-hfri-fund-weighted-composite-and-the-benchmark-layer)
   - 6.4 [The Largest Firms by AUM (Cross-Referenced)](#64-the-largest-firms-by-aum-cross-referenced)
   - 6.5 [The Geographic Split — US, Europe, Asia](#65-the-geographic-split--us-europe-asia)
   - 6.6 [The Singapore Hub (Cross-Referenced)](#66-the-singapore-hub-cross-referenced)
7. [The Banking Relationships](#7-the-banking-relationships)
   - 7.1 [Prime Brokerage — the Core Product](#71-prime-brokerage--the-core-product)
   - 7.2 [Custody and Fund Administration (Cross-Referenced)](#72-custody-and-fund-administration-cross-referenced)
   - 7.3 [Fund Financing — Subscription Lines, NAV Facilities, GP Financing (Cross-Referenced)](#73-fund-financing--subscription-lines-nav-facilities-gp-financing-cross-referenced)
   - 7.4 [KYC/AML and Fund-Client Onboarding (Cross-Referenced)](#74-kycaml-and-fund-client-onboarding-cross-referenced)
8. [The Risk](#8-the-risk)
   - 8.1 [The Risk Profile of Hedge-Fund Strategies](#81-the-risk-profile-of-hedge-fund-strategies)
   - 8.2 [Drawdown — the Peak-to-Valley Measure](#82-drawdown--the-peak-to-valley-measure)
   - 8.3 [Value at Risk — and Its Limits](#83-value-at-risk--and-its-limits)
   - 8.4 [Stress Testing (Cross-Referenced)](#84-stress-testing-cross-referenced)
   - 8.5 [The 2008 Redemption Spiral](#85-the-2008-redemption-spiral)
   - 8.6 [Risk Management as a Bank's Concern](#86-risk-management-as-a-banks-concern)
9. [The Performance](#9-the-performance)
   - 9.1 [Measuring Returns — the Fund's Own Calculation, TWR, and IRR](#91-measuring-returns--the-funds-own-calculation-twr-and-irr)
   - 9.2 [Benchmarks — the HFRI Indices and Peer Groups](#92-benchmarks--the-hfri-indices-and-peer-groups)
   - 9.3 [GIPS and Composite Presentation (Cross-Referenced)](#93-gips-and-composite-presentation-cross-referenced)
   - 9.4 [The Alpha/Beta Debate](#94-the-alphabeta-debate)
   - 9.5 [Persistence of Performance — the Weak-Evidence Finding (⚠)](#95-persistence-of-performance--the-weak-evidence-finding-)
10. [The Comparison Table — Hedge Fund vs Private Equity vs Mutual Fund vs CTA](#10-the-comparison-table--hedge-fund-vs-private-equity-vs-mutual-fund-vs-cta)
11. [The Cymbal Bank Worked Example — Merlion Long/Short Fund](#11-the-cymbal-bank-worked-example--merlion-longshort-fund)
    - 11.1 [The Scenario](#111-the-scenario)
    - 11.2 [The Facility Terms](#112-the-facility-terms)
    - 11.3 [The Prime-Brokerage and Financing Arithmetic](#113-the-prime-brokerage-and-financing-arithmetic)
    - 11.4 [The Fee and Performance Arithmetic](#114-the-fee-and-performance-arithmetic)
    - 11.5 [The KYC/AML Overlay and the Relationship Logic](#115-the-kycaml-overlay-and-the-relationship-logic)
12. [The Claims Audit and What Could Not Be Verified](#12-the-claims-audit-and-what-could-not-be-verified)
    - 12.1 [The Claims Audit — Verified, Flagged, Rejected](#121-the-claims-audit--verified-flagged-rejected)
    - 12.2 [What Could Not Be Verified](#122-what-could-not-be-verified)
13. [Glossary, References, and the Closing Line](#13-glossary-references-and-the-closing-line)
    - 13.1 [The Glossary](#131-the-glossary)
    - 13.2 [References and Further Reading](#132-references-and-further-reading)

---

## 1. What a Hedge Fund Is — the Pooled, Private, Liquid Absolute-Return Vehicle

### 1.1 The Short Answer

A **hedge fund** is a pooled investment fund that holds liquid assets and uses complex trading and risk-management techniques — short selling, leverage, derivatives — to aim for a positive return in any market environment, rather than to beat an index ✅ (Wikipedia, "Hedge fund", with its primary citations). Three adjectives in that sentence do the structural work:

- **Pooled** — investors' money is commingled into one vehicle, and every investor's return is the vehicle's return (net of fees), not a bespoke portfolio.
- **Private** — the fund is not offered to the public. In the United States, regulations require that hedge funds be marketed only to institutional investors and high-net-worth individuals ✅ (same source); the classic legal architecture is a limited partnership with a 99-investor cap that sidesteps the Investment Company Act of 1940 ✅ (Wikipedia, "Alfred Winslow Jones" — Jones built his fund exactly this way).
- **Liquid** — unlike a private-equity fund, a hedge fund is open-ended: it invests in relatively liquid assets and typically allows investors to subscribe and redeem periodically at net asset value (NAV), subject to lock-ups, notice periods and gates (§4.5–§4.6) ✅ (Wikipedia, "Hedge fund").

The mandate that completes the definition is **absolute return**: hedge-fund strategies aim to produce positive returns regardless of whether markets rise or fall — the "hedge" being the insulation of returns from market risk ✅ (same source). Note the honesty clause in the same source: "there are no formal or fixed definitions of fund types, and so there are different views of what can constitute a 'hedge fund'" ✅ — which is why §3 flags the strategy-taxonomy variance and §12 flags the AUM-definition variance.

### 1.2 The Four Structural Features

Every hedge fund, from Jones's 1949 partnership to a 2026 multi-strategy platform, is a combination of four features:

| Feature | The convention | Why it matters |
|---|---|---|
| **Private placement** | Sold only to accredited/qualified investors and institutions; no public prospectus; an offering memorandum (the "PPM") sets the terms | Regulatory lightness is the point — the fund can short, lever and concentrate in ways a registered mutual fund cannot |
| **Open-ended liquidity** | Periodic dealing — typically monthly or quarterly NAVs; subscriptions and redemptions at NAV; lock-ups (often 1 year) and redemption notices (30–90 days) gate the exits | The fund is *periodically* liquid, not daily; the mismatch between daily-marked assets and quarterly dealing is the source of the 2008 redemption spiral (§8.5) |
| **Absolute-return mandate** | Success = positive risk-adjusted return in up and down markets, not outperformance of a benchmark; market-neutral or directional by strategy (§3) | Fees are earned on *profits* (performance fee), which aligns the manager with gains but — as Buffett has argued — not with losses (§4.2) |
| **Performance-linked fees** | Management fee (typically 1–4% p.a. on NAV, 2% the standard) plus a performance fee (typically 20% of profits, 10–50% range) with a high-water mark ✅ (Wikipedia, "Hedge fund") | The fee engine is the fund's economics — see §4 for the full machinery |

The consequence of the combination is an asset class that sits between a mutual fund (daily dealing, no leverage, index-relative) and private equity (closed-end, illiquid, decade-long): a hedge fund is *levered and flexible, but liquid enough to be redeemed* — which is precisely what makes it both a prime-brokerage client and a liquidity-risk problem in a crisis.

### 1.3 The Key-Facts Table

| Aspect | Fact | Status |
| --- | --- | --- |
| Origin | First modern hedge fund: A.W. Jones & Co., 1949, a long/short equity limited partnership; Jones coined "hedged fund" | ✅ Wikipedia (Alfred Winslow Jones) |
| The name | The term "hedge fund" was popularised by Carol Loomis's 1966 *Fortune* article on Jones | ✅ Wikipedia (Carol Loomis); issue dating ⚠ (§12) |
| Global AUM | **US$5.15 trillion** at end-2025 — the first time above $5 trillion; +$642.8B in 2025 (record); strongest inflow year since 2007 | ✅ HFR press release, 22 Jan 2026 |
| 2025 performance | HFRI Fund Weighted Composite +12.5% — best year since 2009 | ✅ HFR, 22 Jan 2026 |
| Index provider | HFR (Hedge Fund Research, Inc.) — publisher of the HFRI indices; index history since 1993 per hfr.com | ✅ hfr.com (homepage + releases) |
| Fee convention | "2 and 20" — 2% management fee, 20% performance fee; Jones's 1949 partnership charged 20% of profits and later added 2% on assets | ✅ Wikipedia (Jones); compression trend ⚠ (§4.4) |
| Fee mark | Performance fees apply above the high-water mark; hurdles (hard/soft) optional | ✅ Wikipedia ("Hedge fund") |
| Investor base | Institutions only in the US by regulation; ~86% of industry assets sit with billion-dollar-plus managers | ✅ InvestmentNews/P&I, Feb 2026 |
| Largest firm | Bridgewater Associates, ~US$78B (June 2025 data) | ✅ InvestmentNews/P&I; ⚠ ranking vintage |
| Dominant domicile | Cayman Islands ~34% of global fund count (2011 data); half of funds offshore | ✅ Wikipedia (TheCityUK citation); vintage ⚠ |
| EU wrapper | UCITS — Council Directive 85/611/EEC of 20 December 1985 | ✅ EUR-Lex |
| Prime brokerage | The bank product at the centre of the fund: clearing, leverage/financing, securities lending, custody, execution | ✅ Wikipedia ("Hedge fund", structure) |
| Regulation | Post-2008: SEC registration of advisers (2010) and EU AIFMD (2011) added oversight | ✅ Wikipedia; AIFMD cross-ref [Private Equity](private_equity_guide.md) §7 |

### 1.4 Why a Hedge Fund Matters to a Bank

For a bank like Cymbal Bank, a hedge fund is not one client but five revenue streams wrapped in one relationship:

1. **Prime brokerage** — margin lending, securities financing, clearing, custody, execution, capital introduction and risk reporting (§7.1); the single deepest product a bank sells a hedge fund, and the anchor of the worked example in §11.
2. **Custody and fund administration** — the fund's assets sit with a custodian; its NAV is produced by an administrator; both are bank or bank-adjacent businesses (§7.2, cross-ref [Investment Portfolio Operations](investment_portfolio_operations_guide.md)).
3. **Fund financing** — subscription lines, NAV facilities and GP financing, adapted from the private-equity template (§7.3, cross-ref [Private Equity](private_equity_guide.md) §9).
4. **Treasury and FX** — a fund trading across Asia needs multi-currency accounts, FX forwards and cash sweeps; the worked example prices these (§11).
5. **The compliance overlay** — onboarding a fund client is a KYC/AML and sanctions exercise (cross-ref [FircoSoft](fircosoft_guide.md) and [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §9.3), and the bank's PB credit risk is a first-order risk-management problem (§8.6).

The bank's risk view is the mirror image of the fund's: the fund sees the prime broker as a counterparty and lender; the bank sees the fund as a leveraged borrower whose collateral is marked to market daily. That mutual dependence — and the 2008 lesson that it can amplify a crisis in both directions — is the thread running through §7–§8.

---

## 2. The History — 1949 to the 2020s

### 2.1 A.W. Jones, 1949 — the First Hedged Fund

**Alfred Winslow Jones** (1900–1989) was an unlikely founder: a sociologist with a Columbia PhD who worked for *Fortune* magazine in the 1940s ✅ (Wikipedia, "Alfred Winslow Jones"). While researching a March 1949 *Fortune* article, "Fashions in Forecasting", on stock-market technicians, he conceived the investment partnership that bears his name. Two months before the article went to press he established **A.W. Jones & Co.**, raising **US$100,000 — $40,000 of it his own**; the partnership gained **17.3% in its first year** ✅ (same source).

Jones's innovation was not a trade but an operating system, and every element survives in the modern industry:

- **Long/short equity** — he bought as many stocks as he sold, so market-wide moves were neutralised and the portfolio's value depended on *stock selection* rather than market direction ✅ (same source).
- **Leverage** — he used leverage to buy more shares, which is what made the hedge (shorts) affordable and the returns interesting ✅ (same source).
- **The 99-investor limited partnership** — structured to avoid the Investment Company Act of 1940, the template for every private fund since ✅ (same source).
- **The fee model** — Jones took **20% of profits** as compensation, invoking the Phoenician sea captains who kept a fifth of a successful voyage's profits; he charged no fee unless he made a profit, and **later added a 2% management fee on all assets** ✅ (same source). This is the ancestor of "2 and 20" (§4.4).
- **Manager capital and restricted withdrawals** — Jones kept his own money in the fund and limited redemptions, the two governance features that institutional LPs still demand ✅ (secondary: sharemaestro.com profile; Jones-era practice as widely described ⚠ for the exact withdrawal terms).

### 2.2 1966 — Carol Loomis Names the Industry; 1968–1970 — the First Shakeout

For seventeen years Jones's model stayed obscure. Then, in **1966, Carol Loomis of *Fortune*** wrote the article that **coined the term "hedge fund"** and popularised the strategy: **"The Jones Nobody Keeps Up With"** — the title under which Wikipedia records it; the task brief and much of the literature cite **June 1966** as the issue date, while Wikipedia's Jones bibliography dates the article to the **April 1966** issue — ⚠ variant dating, flagged in §12 ✅ (Wikipedia, "Carol Loomis" and "Alfred Winslow Jones" bibliography) ✅ (article substance). The article's opening line lionised Jones — "the best professional money manager of investors' money these days" — and reported that **Jones's fund had outperformed the best mutual fund over the previous five years by 44%, and had beaten the top-performing Dreyfus Fund by 87% over ten years**, despite its incentive fee ✅ (Wikipedia, "Carol Loomis").

The publicity detonated the industry: **within the next three years at least 130 hedge funds were started**, including **George Soros's Quantum Fund** and **Michael Steinhardt's Steinhardt Partners** ✅ (same source). Then came the first shakeout: **many hedge funds closed during the recession of 1969–1970 and the 1973–1974 stock market crash due to heavy losses** ✅ (Wikipedia, "Hedge fund"). The pattern — a boom fed by publicity and leverage, followed by a bear-market cull — would repeat in 1998 and 2008.

### 2.3 The 1980s–90s — Global Macro and Black Wednesday

The industry "received renewed attention in the late 1980s" ✅ (Wikipedia, "Hedge fund") — and the strategy that dominated the era was **global macro**: large directional positions in share, bond and currency markets driven by macroeconomic analysis (§3.5). The emblematic trade was Soros's against the pound. **Black Wednesday — 16 September 1992** — saw the UK forced to withdraw sterling from the European Exchange Rate Mechanism after failing to hold the currency above its ERM floor; **Soros, who had built a huge short position in sterling, made over £1 billion in profit** ✅ (Wikipedia, "Black Wednesday"). The episode established the global-macro archetype: a single manager's thesis, expressed at a scale that moved a central bank's policy, and a profit that dwarfed the fund's own capital base.

The 1990s also brought the structural growth spurt: the number of hedge funds increased significantly with the 1990s bull market, and strategies expanded beyond long/short equity into credit arbitrage, distressed debt, fixed income, quantitative and multi-strategy ✅ (Wikipedia, "Hedge fund"). US institutional investors — pensions and endowments — began allocating serious money ✅ (same source), turning hedge funds from a rich-man's hobby into an institutional asset class.

### 2.4 LTCM, 1998 — the Fed-Brokered Bailout

**Long-Term Capital Management L.P.** was founded in **1994** by **John Meriwether**, the former Salomon Brothers bond-arbitrage chief, with a board that included **Myron Scholes and Robert Merton** — who won the 1997 Nobel Prize for the Black–Scholes model ✅ (Wikipedia, "Long-Term Capital Management"). LTCM ran **convergence trading**: quantitative fixed-income relative-value positions (on-the-run/off-the-run Treasuries, swap spreads, sovereign spreads) at extreme leverage. The early record was spectacular — annualised returns after fees of roughly **21%, 43% and 41%** in its first three years ✅ (same source).

The collapse is the canonical leverage lesson: in **1998 LTCM lost $4.6 billion in less than four months**, a combination of high leverage and exposure to the 1997 Asian crisis and the **1998 Russian financial crisis** (the Russian default and the flight to quality blew out every spread the fund was short) ✅ (same source). With the fund on the verge of failure and its counterparty web touching every major dealer, the **Federal Reserve Bank of New York brokered a $3.6 billion bailout by a consortium of 14 banks** — the agreement of **23 September 1998** recapitalised the fund under Fed supervision; the fund was **liquidated and dissolved in early 2000** ✅ (same source). The investor losses were severe, but the systemic lesson — that a levered fund's counterparty network can turn a private failure into a public-policy problem — is what made 1998 the dress rehearsal for 2008.

### 2.5 The 2000s and the 2008 Crisis — Redemptions, Gates, and Madoff

By **2008 the worldwide hedge-fund industry held an estimated US$1.93 trillion** in AUM ✅ (Wikipedia, "Hedge fund"). The 2008 financial crisis then produced the industry's defining stress: **many hedge funds restricted investor withdrawals**, and AUM and popularity declined ✅ (same source). The mechanism was the **redemption spiral** (§8.5): investors redeemed, funds sold assets into falling markets, marks fell, more investors redeemed, and funds responded with **gates and suspensions** (§4.6) — the liquidity tools that had existed in the fine print of offering memoranda for years suddenly became the industry's headline.

The crisis also exposed the fraud risk embedded in the private, opaque model. **Bernard Madoff** was arrested on **11 December 2008** after confessing to his sons; the **SEC charged him and Bernard L. Madoff Investment Securities LLC** that same day, alleging a **multi-billion-dollar Ponzi scheme** — the SEC's complaint put the fraud at **$50 billion** ✅ (SEC Press Release 2008-293; SEC Litigation Release No. 20889). He **pleaded guilty on 12 March 2009** to 11 federal crimes — **the largest known Ponzi scheme in history**, worth an estimated **$65 billion** — and was **sentenced on 29 June 2009 to 150 years** ✅ (Wikipedia, "Madoff investment scandal"). The scandal's postscript for the fund-services industry was legal as well as reputational: *Anwar v. Fairfield Greenwich* (SDNY 2015), the major case on **fund-administrator liability** for NAV-related failures — the administrator and other defendants **settled in 2016 for $235 million** ✅ (Wikipedia, "Hedge fund", structure section). Madoff's funds had used an administrator that never independently verified the assets; the case is why modern administrators, and the banks that vet them, treat NAV independence as a first-order control.

### 2.6 The 2010s–2020s — Institutionalisation and the $5 Trillion Industry

The post-crisis decade institutionalised the industry: AUM rebounded to almost **$2 trillion by April 2011**, and by **July 2017 reached a record $3.1 trillion** ✅ (Wikipedia, "Hedge fund"). The mid-2010s brought the "hedge fund killing field" — Dan Loeb's phrase for the classic long/short squeeze as central-bank easing lifted correlations — and a consolidation "around the larger, more established firms such as Citadel, Elliott, Millennium, Bridgewater" ✅ (same source). The 2020s delivered the scale: the industry crossed **$4 trillion** in 2023–2024 territory and then, per HFR, hit **$4.98 trillion in 3Q25** (eighth consecutive quarterly record, largest quarterly net inflow since 2007) and **surged past $5 trillion to a record $5.15 trillion at end-2025**, with **2025 net inflows of $115.8 billion — the strongest calendar year of investor inflows since 2007** ✅ (HFR press releases, 23 Oct 2025 and 22 Jan 2026). The 2026 tape shows the new regime's texture: **2Q26 industry capital rose $409.3 billion — the largest quarterly increase in history** (FT: "Hedge funds grow at fastest rate in history as AI boom lifts markets") — and then **July 2026's AI-led rout knocked the HFRI Technology Index down 7.0%, its worst decline since 2008** ✅ (HFR releases, 23 Jul 2026 and 7 Aug 2026). Multi-strategy and quant platforms (Citadel, Millennium, Point72, Balyasny, D.E. Shaw) and the largest macro houses (Bridgewater) now dominate flows — the concentration story of §6.4 — and the 2020s industry is, structurally, the Jones model at institutional scale: leverage, hedging, performance fees and periodic liquidity, now wrapped in regulation, gates, and a $5-trillion balance sheet.

---

## 3. The Strategy Taxonomy

### 3.1 The Taxonomy Map — and Why It Varies by Provider

Hedge-fund strategies are "generally classified among four major categories: **global macro, directional, event-driven, and relative value (arbitrage)**" ✅ (Wikipedia, "Hedge fund", citing Hedge Fund Tools) — with a further division into **discretionary/qualitative** (manager selection) and **systematic/quantitative** (computerised selection) ✅ (same source). The industry's index providers run their own, overlapping taxonomies: **HFR's four families are Equity Hedge, Event-Driven, Relative Value Arbitrage and Macro** ✅ (HFR Year-End 2025 release — the strategy capital figures of §6.2), while **BarclayHedge** uses a finer-grained classification (the Wikipedia citations reference BarclayHedge's strategy-definition library) ⚠. **The taxonomy varies by provider** — a "long/short" fund may sit in Equity Hedge (HFR) or be its own category (BarclayHedge); a CTA may be a Macro sub-strategy (HFR) or a separate asset class — so every comparison table in this guide names its source and flags the variance. The map below follows the common-denominator scheme: the eight families a banker actually hears in a fund's pitch.

### 3.2 Long/Short Equity

**Definition.** The Jones original: the manager buys undervalued stocks and shorts overvalued ones, typically with a net bias — 130/30, 150/50, or plain 100/50 (long/short as percentages of NAV) are common shapes. **Return driver.** Stock selection ("alpha") on both legs, with the short book funding and hedging the long book; the market (beta) contributes only the *net* exposure. **Profile.** The classic institutional hedge-fund allocation — high capacity, moderate fees, and a return stream that historically beat long-only in flat and falling markets while lagging in straight-up bulls ⚠ (directional historical generalisation). The 2010s "killing field" (§2.6) was precisely this strategy losing its edge to central-bank-driven correlation ✅ (Wikipedia, "Hedge fund").

### 3.3 Market-Neutral

**Definition.** A long/short variant engineered for **zero net market exposure**: longs and shorts are beta-matched so the portfolio's return is independent of market direction ✅ (Wikipedia, "Hedge fund" — market-neutral funds "neutralize the effect of market swings"). **Return driver.** Pure relative-value alpha — the spread between the long book and the short book — plus carry. **Profile.** Low correlation to equities, low volatility, but lower absolute returns and acute capacity limits: the strategy's alpha does not scale. The statistical-arbitrage quant funds (§3.8) are the systematic version of the same idea.

### 3.4 Event-Driven — Merger Arbitrage, Distressed, Special Situations

**Definition.** Positions driven by corporate events whose *outcomes*, not market direction, determine the payoff ✅ (Wikipedia strategy categories; BarclayHedge definitions ⚠). The three canonical sub-strategies:

- **Merger arbitrage (risk arbitrage).** After a deal is announced, buy the target and short the acquirer (or the deal-currency), capturing the **spread** between the market price and the deal price; the risk is the deal breaking ✅ (BarclayHedge definition as cited by Wikipedia ⚠ primary page not re-extracted). *Return driver:* deal-spread convergence; *risk:* deal failure (the "termination spread" blows out).
- **Distressed.** Buy the debt (and sometimes equity) of companies in or near bankruptcy at deep discounts; returns come from restructuring outcomes and recovery rates ✅ (Wikipedia, "Hedge fund" — distressed debt listed among the 1990s strategy expansions).
- **Special situations.** Anything event-shaped that is not a clean merger or bankruptcy — spin-offs, asset sales, regulatory outcomes, activism (Elliott's home turf, §6.4).

**Return driver.** Event resolution, not market direction; **Profile.** Moderate volatility, positive skew in calm markets, but event risk is binary — the 2008 spike in merger terminations and the 2020 COVID deal-break wave hit the strategy hard ⚠ (directional, widely reported).

### 3.5 Global Macro

**Definition.** Large directional positions in share, bond or currency markets taken on the back of macroeconomic analysis — "big picture" bets on rates, FX, inflation, policy ✅ (Wikipedia, "Hedge fund"). **Return driver.** The manager's macro thesis expressed through leveraged, concentrated positions; the Soros/Quantum Black Wednesday trade of §2.3 is the archetype — £1 billion of profit on a thesis that the pound's ERM rate was unsustainable ✅ (Wikipedia, "Black Wednesday"). **Profile.** The most flexible and the most volatile category; returns cluster in crisis years (macro funds thrived in 2008 and 2022 ⚠); capacity is effectively unlimited because the positions are in the deepest markets; fees at the top end. Discretionary macro (Soros, Tudor, Brevan Howard) and systematic macro (the trend-following CTAs of §3.7) split the category ✅ (Wikipedia — discretionary vs systematic divide).

### 3.6 Relative Value

**Definition.** Long and short positions in *related* instruments whose prices should converge — the LTCM playbook: on-the-run vs off-the-run Treasuries, swap spreads, convertible arbitrage, volatility arbitrage ✅ (Wikipedia strategy categories; LTCM article for the trade list). **Return driver.** Spread convergence plus carry; the manager is paid for identifying mispriced *relationships*, not directions. **Profile.** Low volatility and low correlation in normal markets — HFR's Relative Value Arbitrage family posted positive performance in 27 of the last 30 months as of 3Q25 ✅ (HFR release, 23 Oct 2025) — but the strategy carries the leverage and the liquidity risk that killed LTCM: spreads *widen* in stress, and the arbitrage becomes a margin call ✅ (Wikipedia, "Long-Term Capital Management").

### 3.7 CTA / Managed Futures

**Definition.** **Commodity trading advisors** — funds that trade futures and options in commodity and financial markets, taking both long and short positions so they can profit in upswings and downswings; also known as managed-futures funds ✅ (Wikipedia, "Hedge fund"). **Return driver.** Mostly **systematic trend-following** — momentum signals across dozens of markets; some systematic macro and carry. **Profile.** The crisis-diversifier: trend followers historically make their best years in violent markets (2008, 2014's oil crash, 2022's rates repricing ⚠ directional), and their worst in choppy, trendless years; regulated as CTAs under CFTC rules; daily liquidity is common, which separates them from the rest of the hedge-fund complex.

### 3.8 Quantitative / Systematic

**Definition.** Strategies selected by computerised models rather than discretionary judgment ✅ (Wikipedia, "Hedge fund" — the discretionary/systematic divide). The quant universe spans **statistical arbitrage** (market-neutral, high-turnover mean-reversion), **factor investing** (value, momentum, carry, quality — AQR's home), **machine-learning signals** (Two Sigma, Renaissance), and systematic versions of macro, credit and equity strategies. **Return driver.** Signal edge — data, research, and execution technology; capacity is the binding constraint (a signal decays as capital crowds it). **Profile.** Low headline volatility, high Sharpe ambitions, but crowded-trade tail risk — the August 2007 "quant quake" and the March 2020 dash-for-cash are the canonical drawdowns ⚠ (directional, widely documented; not re-verified this pass).

### 3.9 Multi-Strategy

**Definition.** One fund (or platform) running several strategies in parallel — typically equities, credit, macro, commodities and quant — under a single risk, capital and technology framework; the **pod architecture** of Citadel, Millennium, Point72 and Balyasny, where semi-autonomous teams ("pods") trade with centrally allocated capital and risk limits ✅ (cross-ref [Citadel LLC](citadel_llc_guide.md) §3 — pod architecture and central risk; [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §5 for the global firms' SG presence). **Return driver.** Diversification of uncorrelated strategy streams, plus the platform's ability to redeploy capital to whatever is working; leverage is high but centrally risk-managed ✅ (Citadel guide — the 2008 ~7:1 episode). **Profile.** The industry's growth engine of the 2020s — the biggest firms by AUM (§6.4) are multi-strategy platforms; fees at the top end with long lock-ups; the flows data show investors concentrating here ✅ (HFR release, 22 Jan 2026 — managers over $5B took $101.4B of the 2025 inflows).

### 3.10 The Strategy Comparison Table

| Strategy | Return driver | Volatility | Equity correlation | Capacity | Typical fee level |
| --- | --- | --- | --- | --- | --- |
| Long/short equity | Stock selection alpha + net beta | Medium | Medium–high | High | 1.5% / 20% |
| Market-neutral | Relative-value alpha, low beta | Low | Low | Low | 1.5% / 20% |
| Event-driven (merger arb / distressed) | Event resolution, deal spreads | Low–medium | Low–medium | Medium | 1.5% / 20% |
| Global macro (discretionary) | Macro thesis, leveraged | High | Low | Very high | 1.5–2% / 20–25% |
| Relative value | Spread convergence + carry | Low–medium | Low | Medium | 1.5% / 20% |
| CTA / managed futures | Trend-following, systematic | Medium | Low (crisis-positive) | High | 1–1.5% / 15–20% |
| Quant / systematic | Signal edge, factor premia | Low–medium | Low | Low–medium | 1.5–2% / 20–30% |
| Multi-strategy | Diversified streams, capital redeployment | Low–medium | Low | Platform-constrained | 2–3% / 25–30% ⚠ top-end |

*Taxonomy note:* the table is a common-denominator synthesis; HFR's four families (Equity Hedge, Event-Driven, Relative Value Arbitrage, Macro) and BarclayHedge's finer scheme do not map one-to-one ⚠ (§3.1, §12). Fee levels are directional market practice, not a verified schedule ⚠.

---

## 4. The Fund Economics — Fees, High-Water Marks, Hurdles, Liquidity

### 4.1 The Management Fee — on NAV, Not Commitments

The **management fee** is the fund's running cost line: a percentage of the fund's **net asset value** — typically **1% to 4% per annum, with 2% being standard** — expressed annually but calculated and paid monthly or quarterly ✅ (Wikipedia, "Hedge fund"). The hedge-fund framing differs from private equity in a way that matters to every investor and every banker:

- **PE charges on committed capital** during the investment period — the fee is earned whether or not the money is deployed (cross-ref [Private Equity](private_equity_guide.md) §3.1 — do not re-derive).
- **Hedge funds charge on NAV** — the fee floats with the fund's mark, so a fund in drawdown earns a shrinking fee, and a growing fund earns a fat one. This is why the hedge-fund management fee is described as "designed to cover the operating costs of the manager" — the *profits* come from the performance fee — yet at scale the management fee alone can make a manager rich ✅ (Wikipedia, "Hedge fund" — economies of scale, CalPERS criticism).

### 4.2 The Performance Fee and the High-Water Mark

The **performance fee** (incentive fee) is "typically **20% of the fund's profits during any year**, though performance fees range between 10% and 50%" ✅ (Wikipedia, "Hedge fund"). Almost all performance fees include a **high-water mark** (the "loss carryforward provision"): **the fee applies only to net profits — profits after losses in previous years have been recovered** ✅ (same source). The mechanism in numbers:

- Year 1: fund starts at NAV 100, rises to 120. Fee = 20% × 20 = 4. High-water mark is now 120.
- Year 2: fund falls to 100. No performance fee (below the mark).
- Year 3: fund rises to 130. Fee = 20% × (130 − 120) = 2 — the 20-point loss from Year 2 had to be recovered first ✅ (arithmetic follows the verified definition).

The high-water mark is the industry's answer to "heads I win, tails you lose": without it, a manager could harvest fees on every up-year and ignore the down-years. Its dark side is real and documented: **a manager will sometimes close a fund that has suffered serious losses and start a new fund** rather than work for years below the mark ✅ (Wikipedia, "Hedge fund"). Warren Buffett's criticism of the fee structure — hedge funds share the profits but not the losses, creating an incentive for high-risk management — is recorded in the same source ✅.

### 4.3 The Hurdle — Hard vs Soft

A **hurdle** is a performance threshold below which no performance fee is paid, typically tied to a benchmark rate such as LIBOR (historically) or the one-year Treasury bill rate plus a spread ✅ (Wikipedia, "Hedge fund"). The two variants, verified:

- **Soft hurdle:** the fee is calculated on **all** of the fund's returns once the hurdle is cleared — e.g. hurdle 5%, fund returns 15% → fee on the full 15%.
- **Hard hurdle:** the fee is calculated **only on returns above the hurdle** — hurdle 5%, fund returns 15% → fee on the 10% above ✅ (Wikipedia, "Hedge fund", with the worked example).

The hurdle exists "to ensure that a manager is only rewarded if the fund generates returns in excess of the returns that the investor would have received if they had invested their money elsewhere" ✅ (same source). Hard hurdles are the LP-friendly variant; soft hurdles are more common in practice ⚠ (market practice, not re-verified).

### 4.4 The 2-and-20 Convention — and Its Compression (Cross-Referenced)

**"2 and 20"** — a 2% management fee plus a 20% performance fee — is the industry's canonical package: Jones's 1949 partnership charged 20% of profits and later added 2% on assets ✅ (Wikipedia, "Alfred Winslow Jones"), and "Jones also developed the popular 2-and-20 structure" ✅ (Wikipedia, "Hedge fund"). The exact moment the package became canonical is not pinnable to a single primary document — the same caveat the sibling guide records for carried interest ⚠ (cross-ref [Private Equity](private_equity_guide.md) §12: "The precise origin of '2 and 20'… no primary document establishes when the package became canonical").

**The carry mechanics** — the 20% performance fee is the hedge fund's cousin of private-equity carried interest: a share of profits above an investor threshold, governed by the fund agreement and the high-water mark (the hedge-fund analogue of the PE waterfall's hurdle-and-catch-up). For the full PE framing — the waterfall, the catch-up, the GP commitment, the clawback — cross-ref [Private Equity](private_equity_guide.md) §3 (do not re-derive). The hedge-fund-specific differences: fees run on **NAV not commitments** (§4.1); the "waterfall" is a per-investor high-water mark rather than a whole-fund distribution order; and there is no clawback because there is no distribution of carry before the fund's final accounting — the HWM performs that function ✅ (structure follows the verified HWM definition; PE cross-ref).

**Fee compression** is the industry's pricing story of the 2010s–2020s: the once-standard 2-and-20 is "history", replaced by a raft of innovative fee structures as competition for assets intensifies ⚠ (With Intelligence, "Pricing performance" — secondary); average management fees for established funds are "closer to 1.3–1.5%" ⚠ (globalinvestments.net, 2024–25 strategy review — secondary). The direction — down — is corroborated by the verified statement that performance-fee rates "have fallen since the start of the credit crunch" ✅ (Wikipedia, "Hedge fund"). The large multi-strategy platforms are the exception that proves the rule: their fees sit *above* 2-and-20 (the §3.10 top-end row) because the pod architecture and central risk infrastructure are expensive to run ⚠.

### 4.5 Lock-Ups, Redemption Terms, and Notice Periods

A hedge fund's liquidity is *periodic, not daily* — and the fine print is where the fund's liquidity risk lives ✅ (Wikipedia, "Hedge fund" — open-ended funds allow periodic subscription/redemption at NAV; lock-ups are the qualification). The standard toolkit:

- **Lock-up:** a period — typically one year, longer for multi-strategy and private-credit sleeves — during which an investor cannot remove money ✅ (Wikipedia, "Hedge fund" — "similar to private-equity funds, hedge funds employ a lock-up period during which an investor cannot remove money").
- **Redemption frequency:** dealing typically monthly or quarterly (some funds annually); the investor redeems at the next NAV after the dealing date.
- **Redemption notice:** 30–90 days' notice is the norm — the fund needs the notice to liquidate positions in an orderly way (the worked example in §11 assumes 60 days).
- **Redemption fees:** some funds charge a fee for early withdrawals during a specified period (typically a year) or when withdrawals exceed a threshold — unlike management and performance fees, redemption fees are usually kept by the fund and redistributed to all investors ✅ (Wikipedia, "Hedge fund").

### 4.6 Gates

A **gate** is a provision in the fund documents that **limits the percentage of the fund's assets that can be redeemed on any redemption date** — a cap (commonly 10–25% of NAV ⚠ market convention) on aggregate redemptions per dealing date ✅ (Databento compliance guide: "Redemption gates are provisions in hedge fund documents that limit the percentage of a fund's assets that can be redeemed on any redemption date… protect the fund and remaining investors from liquidity pressures caused by large-scale redemptions"; Investopedia: a gate "limits hedge fund withdrawals during redemption periods to prevent fund runs"). When the gate is triggered, requested redemptions are **scaled down pro rata** and typically paid at the next dealing date (or deferred). Gates "protect the fund and remaining investors" — and, in 2008, they protected the fund at the cost of the redeeming investor's liquidity: the gate is precisely the clause that turned the 2008 redemption requests into a queue ✅ (the 2008 restriction of withdrawals is verified in §2.5; the gate mechanics per Databento/Investopedia). A **suspension** is the stronger cousin — a temporary pause on processing redemptions entirely, allowed when the fund cannot value or liquidate its assets in an orderly way ⚠ (Greenman Open guidance, secondary).

### 4.7 Side Pockets

A **side pocket** is "a mechanism whereby a fund compartmentalizes assets that are relatively illiquid" ✅ (Wikipedia, "Hedge fund"): the illiquid investment is carved out of the main portfolio into a segregated account. The mechanics, verified:

- **No standard redemption rights** attach to the side-pocket interest — the investor's redemption right applies to the main portfolio, not the side pocket ✅ (same source).
- Side-pocket interests are allocated **only to investors present when the investment is made** — new investors do not share them ✅ (same source).
- Funds typically carry side-pocket assets **at cost** for fee and NAV purposes, avoiding the valuation problem of assets with no market price ✅ (same source).

Side pockets came to prominence in the **2008-era crisis**: funds holding assets that had ceased to trade (structured credit, private loans, suspended equities) side-pocketed them so the main portfolio could keep dealing — and the SEC subsequently **expressed concern about aggressive use and sanctioned managers for inappropriate use** ✅ (same source; the 2010 SEC probe is cited to Reuters via Wikipedia). For a bank, side pockets matter twice: in **valuation** (the NAV a PB lends against and an administrator certifies) and in **redemptions** (a redeeming investor gets a side-pocket certificate, not cash — a fact every fund-financing credit analysis must model ⚠ analysis).

---

## 5. The Structure and the Lifecycle

### 5.1 The Service Stack — GP, Manager, Fund, Administrator, Auditor, Custodian, Prime Broker

A hedge fund is a legal shell surrounded by a service stack; the shell and the stack are what a banker actually onboards ✅ (Wikipedia, "Hedge fund", structure section — the service-provider list):

| Layer | Who | What they do | Verified anchor |
| --- | --- | --- | --- |
| The fund | A limited partnership (Cayman, Delaware, or a VCC in Singapore — §5.2) | The legal vehicle that holds the assets and issues interests | ✅ Wikipedia |
| The GP / manager | The fund-management firm (the "investment manager"), legally and financially distinct from the fund | Runs the portfolio, decides trades, exercises discretion | ✅ Wikipedia |
| The prime broker | A division of a large investment bank | Clears trades; provides leverage and short-term financing; counterparty to derivatives; lends securities; custody; execution and clearing | ✅ Wikipedia |
| The administrator | A fund-administration firm | **NAV calculation** (the price at which investors buy and sell), valuation, accounting, subscription/redemption processing | ✅ Wikipedia; *Anwar v. Fairfield Greenwich* liability case |
| The auditor | An independent accounting firm | Year-end audit of the financial statements under US GAAP or IFRS; may verify NAV and AUM | ✅ Wikipedia |
| The custodian | A bank (often the PB's own custody arm, or a separate global custodian) | Safe-keeping, settlement, corporate actions, income collection | ✅ Wikipedia (PB "can provide custodial services"); cross-ref [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §3 |
| The distributor / placement agent | A broker-dealer or placement agent | Marketing the fund to investors (many funds self-distribute) | ✅ Wikipedia |

The governance point that runs through the stack: **the US does not require a fund to appoint an administrator** — all functions can be done in-house, creating the conflict-of-interest and fraud surface that Madoff exploited and *Anwar* litigated; most funds appoint external auditors and administrators "thereby arguably offering a greater degree of transparency" ✅ (Wikipedia, "Hedge fund").

### 5.2 Master-Feeder and Onshore/Offshore Choice (Cross-Referenced)

The canonical global structure is the **master-feeder**: one master fund (typically a **Cayman Islands** vehicle — an SPC for multi-class structures) holds all the assets and executes all the trades, while parallel **feeder funds** (an offshore feeder for non-US and tax-exempt investors, an onshore feeder for US taxable investors) collect the money and subscribe into the master ✅ (cross-ref [Cayman & BVI Master-Feeder Structures](cayman_bvi_master_feeder_guide.md) — the master-feeder mechanics, the feeder tax logic (ERISA, PFIC), and the Cayman SPC/BVI structure options are that guide's subject; do not re-derive). The domicile facts worth carrying into this guide: as of 2011, **half of existing hedge funds were registered offshore and half onshore; the Cayman Islands was the leading location, accounting for 34% of the total number of global hedge funds** (US 24%, Luxembourg 10%, Ireland 7%, BVI 6%, Bermuda 3%) ✅ (Wikipedia, "Hedge fund", citing TheCityUK — vintage ⚠). The tax logic: offshore funds investing in the US pay withholding tax on certain investment income but not US capital gains, and US tax-exempt investors go offshore to preserve their exempt status ✅ (same source). In Asia, the same architecture now runs with a Singapore twist — the VCC umbrella as the onshore alternative to the Cayman shell, per [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §3 (cross-ref).

### 5.3 The UCITS Wrapper — the 1985 Directive and the European Fund Market

**UCITS** — Undertakings for Collective Investment in Transferable Securities — is the European retail fund wrapper, born of the **Council Directive 85/611/EEC of 20 December 1985** "on the coordination of laws, regulations and administrative provisions relating to undertakings for collective investment in transferable securities (UCITS)" ✅ (EUR-Lex, CELEX 31985L0611). Its significance for the hedge-fund world is the *regulated* counter-model: a UCITS fund can be marketed to retail investors across the EU under a single passport, in exchange for hard constraints — eligible assets limited to transferable securities and prescribed liquid instruments, leverage limits, and a depositary — that classic hedge funds reject ✅ (directive's object as stated in its title and recitals; the constraint-vs-flexibility framing is standard ⚠).

The UCITS wrapper matters to hedge funds in two ways. First, **managers build "UCITS hedge funds"** — constrained long/short, market-neutral, and macro funds inside the wrapper — to reach European retail and institutional money that cannot buy a Cayman fund (Man GLG's and many others' European distribution runs this way ⚠ product-level detail not re-verified). Second, the wrapper is a **scale fact of European fund markets**: EFAMA's data show **UCITS net assets of €15,765 billion against €8,281 billion of AIF net assets at end-February 2025** (€15,295 billion UCITS at end-2024) ✅ (EFAMA Investment Fund Industry Fact Sheet, February-2025 data) — i.e. **UCITS are roughly two-thirds of the European fund market by net assets**. The frequently-cited "UCITS = 40% of the EU fund market" stat was **not verified this pass and is inconsistent with the EFAMA figures** ⚠ (§12) — the verified EFAMA numbers are used here instead.

### 5.4 The Lifecycle — Launch, Seeding, Growth, Wind-Down

**Launch (incubation).** A manager raises a first fund — often small (tens of millions), often from friends, family and a few seed investors — and runs it for one to three years to build a track record. The launch economics are brutal: a $50M fund at 1.5% management fee earns $750k a year, less than the cost of the service stack (§5.1) — which is why the incubation period is where most funds die ⚠ (industry-standard logic; no aggregate survival statistic verified this pass).

**Seeding.** **Seed capital** — an anchor allocation (a bank's PB capital-introduction desk, a seed fund, a strategic investor) that lets a young fund reach critical mass — is often structured with **first-loss** or revenue-share economics: the seed investor takes a stake in the manager or a share of fees in exchange for the anchor cheque ✅ (the product exists; cross-ref [Citadel LLC](citadel_llc_guide.md) §10.2 — capital introduction as a PB product; the seeding economics ⚠ market practice). Jones's own $40,000 of personal capital in the 1949 fund is the archetype of the manager's own skin in the game ✅ (§2.1).

**Growth (institutionalisation).** Success brings the institutional layer: the fund adds a professional administrator and auditor (if it has not), a prime-brokerage relationship with negotiated terms, a compliance programme, and — the inflection point — the **capacity question**: every strategy has a size beyond which alpha decays (§3.8, §3.10), so growing funds either close to new money (the classic long/short move) or convert into multi-strategy platforms that can redeploy capital across strategies (§3.9). The 2020s flows data show the outcome: investors concentrate in the largest platforms ✅ (HFR, 22 Jan 2026 — $101.4B of 2025 inflows went to managers over $5B).

**Wind-down.** The failure mode is the **redemption spiral**: performance disappoints → investors give notice → the fund sells assets to pay redemptions → marks fall → more investors redeem → gates and suspensions go up → the fund liquidates (§8.5). The documented examples run from the 1969–70 cull (many funds closed) through LTCM's 1998 liquidation and the 2008 restrictions, to the modern norm that "the rate of new fund start-ups is now outpaced by fund closings" ✅ (Wikipedia, "Hedge fund"). Liquidation itself is a service-stack exercise — the administrator computes final NAVs, the auditor signs off, the custodian delivers assets, the PB unwinds the financing — and a bank's wind-down competence is part of the relationship pitch (§11.5) ⚠ analysis.

---

## 6. The Global Industry

### 6.1 Global AUM — the Verified HFR Figures and the Press Estimates

The industry's size is measured, primarily, by **HFR** (Hedge Fund Research, Inc.), whose quarterly Global Hedge Fund Industry Report is the reference series ✅ (hfr.com). The verified numbers, this pass:

| Period | Global hedge-fund capital | Note | Source |
| --- | --- | --- | --- |
| 3Q25 | **US$4.98 trillion** | Record; 8th consecutive quarterly rise; largest quarterly net inflow since 2007 | ✅ HFR release, 23 Oct 2025 |
| 4Q25 / end-2025 | **US$5.15 trillion** | **First time above $5 trillion**; 9th consecutive quarterly rise; +$178.9B in 4Q ($134.1B performance + $44.8B inflows) | ✅ HFR release, 22 Jan 2026 |
| FY2025 | +**$642.8B** | Record calendar-year growth: $527.0B performance gains + $115.8B net inflows — **strongest inflow year since 2007** | ✅ HFR release, 22 Jan 2026 |
| 2Q26 | +**$409.3B** in the quarter | Largest quarterly increase in history; $364B of performance gains; FT: "fastest rate in history" | ✅ HFR release, 23 Jul 2026 |

Press and regulator estimates orbit these figures with definitional caveats: the **SEC** and **BarclayHedge** publish their own aggregates that differ from HFR's because "AUM" can mean regulatory assets, managed capital, or fee-paying assets — the variance is flagged in §12 ⚠ (e.g. the Citadel guide's note that quoted AUM often includes the founder's own capital — cross-ref [Citadel LLC](citadel_llc_guide.md) §3). The directional story is not in dispute: the industry is above $5 trillion for the first time, and the 2025–26 growth is the fastest on record ✅ (HFR).

### 6.2 AUM by Strategy (⚠)

The HFR Year-End 2025 release gives verified strategy-capital figures:

| HFR strategy family | Capital at end-2025 | 2025 change | Verified |
| --- | --- | --- | --- |
| Equity Hedge | **US$1.57 trillion** | +$260.5B | ✅ HFR, 22 Jan 2026 |
| Event-Driven | **US$1.45 trillion** | +$169.6B | ✅ HFR |
| Relative Value Arbitrage | **US$1.35 trillion** | +$137.3B | ✅ HFR |
| Macro | **US$786.6 billion** | +$75.3B | ✅ HFR |

⚠ **Caveat:** these four HFR families do not map one-to-one onto the §3 taxonomy (CTAs sit inside Macro; merger arbitrage sits inside Event-Driven; market-neutral funds are spread across Equity Hedge and Relative Value) — any finer breakdown by strategy (e.g. "CTA share of global AUM") is a provider-specific estimate, not a verified figure ⚠ (§12).

### 6.3 The Indices — HFR, the HFRI Fund Weighted Composite, and the Benchmark Layer

The benchmark layer is dominated by **HFR — Hedge Fund Research, Inc.**, the Chicago-based index, database and research firm that publishes the **HFRI** index family, including the **HFRI Fund Weighted Composite Index® (FWC)** — the industry's headline aggregate ✅ (HFR releases, which report the FWC's returns; hfr.com homepage: "HFR®… the established global leader in indexation, analysis and research of the hedge fund industry", index history "Since 1993 — 30+ years of index history", IOSCO-compliant with an independent Index Advisory Board ✅). Verified index facts from this pass:

- The **HFRI FWC advanced +12.5% in 2025** — the best calendar year since 2009 ✅ (HFR, 22 Jan 2026).
- Strategy-level 2025: **HFRI Equity Hedge (Total) +17.1%**, **HFRI Event-Driven (Total) +10.9%**; sub-strategy: **HFRI EH: Healthcare +33.9%**, **HFRI EH: Energy/Basic Materials +21.4%** ✅ (same release).
- July 2026: the **HFRI Technology Index fell −7.0%** on AI weakness — worst since 2008 ✅ (HFR flash, 7 Aug 2026).

⚠ On the corporate side: no 2022–2024 restructuring or rebranding of HFR could be **verified** this pass (searches returned empty; see §12). The 2026 site branding is "HFR®" with a "Since 1993" index-history marker, and HFR also operates an asset-management arm (HFR Asset Management) and the ManagerLink reporting platform ✅ (hfr.com homepage) — but no formal rebrand event is asserted here. Rival index/database providers (BarclayHedge, the Eurekahedge SG/HK-focused databases, and the defunct HFRX investable index family) exist; none was re-verified this pass ⚠.

### 6.4 The Largest Firms by AUM (Cross-Referenced)

The verified ranking anchor for this guide is the **Pensions & Investments largest-hedge-funds ranking as carried by InvestmentNews (2 February 2026), figures from June-2025 financial reports** ✅:

| Rank | Firm | AUM (June 2025 data) | Strategy identity |
| --- | --- | --- | --- |
| 1 | Bridgewater Associates | US$78.0B | Global macro / risk parity (Pure Alpha, All Weather) |
| 2 | Millennium Management | US$77.5B | Multi-strategy (founded 1989) |
| 3 | Elliott Management | US$76.1B | Event-driven / activist / distressed |
| 4 | Citadel | US$67.6B | Multi-strategy (five strategy groups, one platform) |
| 5 | Man Group | US$66.5B | Quant + discretionary (AHL, GLG, Numeric) |
| 6 | D.E. Shaw Group | US$60.4B | Multi-strategy / quant |
| 7 | AQR Capital Management | US$51.0B | Quant / factor |
| 8 | Two Sigma | US$50.7B | Quant / ML |
| 9 | Goldman Sachs Asset Management | US$48.0B | Multi-strategy platform |
| 10 | Renaissance Technologies | US$46.0B | Quant (Medallion + institutional) |

Supporting facts, cross-referenced rather than re-derived: **Citadel** — the firm this repository deep-dives — entered 2026 at ~$67B AUM ✅ and was reported above **$77B** by December 2025 per Bloomberg ✅, with **$90.4 billion of cumulative net gains since inception** (most in history, per LCH Investments, December 2025) ✅ and the record 2022 (~$16B returned, Wellington +38%) ✅ — all cross-ref [Citadel LLC](citadel_llc_guide.md) §3–§4 (do not re-derive). Concentration is the industry's structural fact: **billion-dollar-plus managers control about $3.6 trillion, roughly 86% of industry assets** ✅ (InvestmentNews/P&I), and in 2025 the largest managers took $101.4B of inflows against $6.6B for sub-$1B managers ✅ (HFR). ⚠ Caveats: the P&I figures are a June-2025 vintage ranking; firm AUM definitions differ (managed capital vs regulatory assets); Man Group's figure is its hedge-fund complex, not its €170B+ group AUM ⚠ (group-level figure per press, not re-verified).

### 6.5 The Geographic Split — US, Europe, Asia

The geography of the industry is US-dominated, European-mature, Asian-growing — but the precise split is **not** a verified HFR statistic this pass ⚠. The verified anchors are directional:

- **Domicile:** the 2011 snapshot — Cayman 34% of fund count, US 24%, Luxembourg 10%, Ireland 7%, BVI 6%, Bermuda 3% ✅ (Wikipedia/TheCityUK, vintage ⚠, §5.2) — shows the offshore/US axis that still dominates the legal structure.
- **The Asia hub:** Singapore's asset-management industry held **S$6.7 trillion** at end-2025 (all asset classes, MAS survey) with a hedge-fund sleeve estimated at **S$327 billion at end-2024** ⚠ — cross-ref [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §1.3 (do not re-derive).
- **Manager geography:** the largest firms are US-headquartered (8 of the top 10 in §6.4), Man Group (London) is the largest non-US manager, and the Asian centres (Singapore, Hong Kong) host both the global platforms' regional offices and a local manager layer (Dymon Asia, Quantedge, APS — cross-ref [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §4).

⚠ Any specific "US 70% / Europe 15% / Asia 15%" style split is a press estimate and is deliberately not stated here (§12).

### 6.6 The Singapore Hub (Cross-Referenced)

Singapore's role in the global industry is covered in full by the sibling guide [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) — this section is a two-paragraph condensation, per the repository convention. Singapore is Asia's leading hedge-fund and asset-management hub: a licensed-manager regime (the LFMC under the SFA, post-RFMC-repeal), the VCC onshore vehicle (1,406 VCCs and 3,443 sub-funds at end-2025), the 13O/13U tax-incentive architecture, a family-office sector of 2,000+ single family offices, and the full service ecosystem — administrators (Alter Domus, IQ-EQ, Vistra), custodians and prime brokers, and the EDB/MAS promotion machinery ✅ (all cross-ref the sibling guide's verified claims; do not re-derive).

For the global industry, the hub's function is threefold: **the Asia seat** for the global multi-strategy platforms (Citadel's shared Singapore office announced 2020; Millennium's SG entity 2006; Point72 2009; Balyasny 2016 — cross-ref [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §5), **the domicile competitor** to the Caymans for new Asia funds (the VCC), and **the capital gateway** — roughly 76% of Singapore's AUM is sourced from outside Singapore and ~88% invested outside it, most into Asia-Pacific ✅ (MAS survey via the sibling guide). The 19 August 2026 MAS/MOF package — performance-profit tax exemption, ONE Pass expansion, a hedge-fund anchoring programme — answered Hong Kong's tax competition and keeps the hub's growth optionality live ⚠ (announced direction; mechanics at Budget 2027 — cross-ref the sibling guide §8.4).

---

## 7. The Banking Relationships

### 7.1 Prime Brokerage — the Core Product

**Prime brokerage (PB)** is the bundled package a bank sells a hedge fund: the prime broker "clears trades and provides leverage and short-term financing… acts as a counterparty to derivative contracts, and lends securities for particular investment strategies, such as long/short equities and convertible bond arbitrage. It can provide custodial services for the fund's assets, and trade execution and clearing services" ✅ (Wikipedia, "Hedge fund"). The core service set, with the worked-example conventions cross-referenced:

| Service | What it is | Why the fund needs it |
| --- | --- | --- |
| **Execution and clearing** | Access to venues (equities, futures, FX, rates) via the bank's broker-dealer | One connection to the world's markets |
| **Financing / margin** | Collateralised, marked-to-market credit against the portfolio — initial margin by product and haircut schedule, variation margin called daily | Leverage is the fund's fuel (§8.1) |
| **Securities lending** | Stock borrow/lend and locates for the short book | A long/short fund's shorts must be borrowed |
| **Custody and asset servicing** | Safe-keeping, settlement, corporate actions, income | The assets must sit somewhere; often the same bank's custody arm (§7.2) |
| **Clearing and settlement** | DvP settlement at the CSDs, fail management | The trade lifecycle (cross-ref [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §2) |
| **Capital introduction** | Introductions to institutional investors | Fundraising is the fund's growth engine |
| **Risk reporting** | Daily P&L, margin, exposure and stress reporting | The fund's own risk room — and the bank's — needs daily marks |

The mechanics — margin lending, securities financing, **rehypothecation** (the PB's re-use of client margin collateral), settlement, and the FIX-based trade lifecycle — are worked through in the repository's two PB examples: [Market Making in Singapore](market_making_singapore_guide.md) §10 (a market maker as PB client) and [Citadel LLC](citadel_llc_guide.md) §10 (a multi-strategy fund as PB client, with the margin-call arithmetic and rehypothecation discussion). **Do not re-derive** — this guide's worked example (§11) follows those conventions and prices the same product set for an Asia long/short fund.

### 7.2 Custody and Fund Administration (Cross-Referenced)

Behind the PB sits the operational layer: the **custodian** holds the assets, settles trades and services corporate actions; the **administrator** produces the NAV, processes subscriptions/redemptions and keeps the books; the **auditor** signs the statements (§5.1). For the bank, the custody-and-administration relationship is both a fee line and a risk control — the independent NAV is what the PB's margin lending and the fund's financing facilities are secured against. The lifecycle mechanics — trade capture through to NAV production, IBOR vs ABOR, valuation (ASC 820/IFRS 13 fair-value hierarchy), reconciliation — are the subject of [Investment Portfolio Operations](investment_portfolio_operations_guide.md) (cross-ref §2–§6; do not re-derive). The control lesson is *Anwar v. Fairfield Greenwich*: the administrator's NAV duties are a real liability surface ✅ (Wikipedia; §2.5).

### 7.3 Fund Financing — Subscription Lines, NAV Facilities, GP Financing (Cross-Referenced)

Hedge funds finance themselves primarily through **PB margin** (§7.1) — but the fund-finance product shelf adds three facilities, adapted from the private-equity template:

- **Subscription lines (capital-call facilities):** revolving credit secured on investors' uncalled commitments. For a hedge fund this is less standardised than for PE — funds with soft commitments or quarterly dealing draw less — but umbrella and seeding structures use it (cross-ref [Private Equity](private_equity_guide.md) §9 for the borrowing-base mechanics, advance rates ~85–95% of eligible commitments, and the worked example — do not re-derive; and [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §9.4 for the hedge-fund framing).
- **NAV facilities:** credit secured on the fund's portfolio NAV — the leverage that sits *on top of* PB margin, used by multi-strategy platforms to lever the whole book ⚠ (market practice; the PE NAV-facility mechanics cross-ref [Private Equity](private_equity_guide.md) §9).
- **GP financing:** credit to the manager/partners — often to fund the GP's own commitment to the fund or to buy out a partner ⚠ (market practice).

The credit logic is the same as PE's: the facility is secured on *something the fund can prove* — uncalled commitments, or a marked NAV produced by an independent administrator. What makes hedge funds harder to finance than PE funds is the asset side: PB margin debt ranks ahead, the NAV moves daily, and the liquidity of the portfolio is the real collateral (cross-ref the credit-risk discussion of [Private Equity](private_equity_guide.md) §9.2 — do not re-derive).

### 7.4 KYC/AML and Fund-Client Onboarding (Cross-Referenced)

Onboarding a hedge fund is a layered KYC exercise, condensed from the siblings:

- **KYC on the structure:** fund constitutive documents (the LPA/PPM, the Cayman or VCC registration), the manager's licence, and the beneficial-ownership chain — manager → fund → investors, with material holders (≥10–25%) and controlling persons identified — the fund-client KYC logic of [Private Equity](private_equity_guide.md) §9.3 applies verbatim ✅ (cross-ref).
- **AML/CFT and sanctions:** transaction monitoring across the fund's flow, **sanctions screening** of counterparties and venues — the FircoSoft screening tooling is documented in [FircoSoft](fircosoft_guide.md) (cross-ref; do not re-derive) — plus **FATCA/CRS** reporting on the investor base ✅ (cross-ref [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §9.3).
- **The MAS overlay** (for a Singapore relationship): the bank's obligations under **Notices 626/626A**, the fund manager's own AML duties (SFA 04-N02), and the VCC's VCC-N01 notice family — the full obligations map is in [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) and [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §9.3 ✅ (cross-ref; do not re-derive).
- **Reputational overlay:** a manager with an enforcement history (insider-trading settlements, auditor qualifications, a Madoff-style custody gap) drives enhanced due diligence — the *Anwar* and Madoff episodes of §2.5 are the cautionary canon ✅.

---

## 8. The Risk

### 8.1 The Risk Profile of Hedge-Fund Strategies

The risk taxonomy, per the verified sources and the strategy map of §3:

- **Leverage risk.** Hedge funds borrow, trade on margin, and use derivatives to obtain exposure beyond their capital ✅ (Wikipedia, "Hedge fund"). The scale is lower than the banks' own: an NBER working paper puts **average investment-bank leverage at 14.2 versus between 1.5 and 2.5 for hedge funds** ✅ (Wikipedia citing Ang/Gorovyy, NBER) — but multi-strategy platforms ran far higher in stress (Citadel ~7:1 in 2008 — cross-ref [Citadel LLC](citadel_llc_guide.md) §2) ⚠.
- **Liquidity mismatch.** The fund holds assets that are marked daily but deals periodically — the lock-up, notice and gate machinery of §4.5–§4.6 exists precisely because the assets are less liquid than the promise ✅ (Wikipedia — liquidity risk is shared with other investment types; the lock-up discussion).
- **Concentration risk.** Too much exposure to one investment, sector, strategy, or group of correlated funds ✅ (Wikipedia — concentration risk definition).
- **Counterparty risk.** The PB, the derivatives counterparties, the securities lenders — LTCM 1998 and 2008 both showed the web can fail (cross-ref §2.4–§2.5; the Wikipedia discussion of prime-broker dependence ✅).
- **Operational risk.** Valuation risk (an inaccurate NAV), style drift, fraud — the *operational due diligence* layer investors run ✅ (Wikipedia — operational due diligence, valuation risk, manager risk).
- **Manager risk.** "Style drift, valuation risk, capacity risk, concentration risk, and leverage risk" ✅ (Wikipedia — the enumerated manager risks).

### 8.2 Drawdown — the Peak-to-Valley Measure

A **drawdown** is "the measure of the decline from a historical peak" in the fund's value; the **maximum drawdown (MDD)** is the worst peak-to-valley loss since inception ✅ (Wikipedia, "Drawdown (economics)"). Formally, if the NAV peaks at P and later stands at V, the drawdown is (P − V)/P; the MDD is the largest such decline over the fund's history ✅ (the article's pseudocode computes DD = 100 × (peak − NAV)/peak). Drawdown matters to hedge funds for three reasons: it is the **investor's real experience** (a 50% drawdown needs a 100% recovery); it is the **high-water mark's shadow** (§4.2 — a drawdown suspends performance fees until recovery); and it is the **funding trigger** — PB margin calls and financing facilities respond to drawdowns in the NAV (cross-ref §8.6). The drawdown-based ratios — Calmar, Sterling, Burke — replace the Sharpe denominator (standard deviation) with drawdown functions ✅ (same source).

### 8.3 Value at Risk — and Its Limits

**Value at Risk (VaR)** "estimates how much a set of investments might lose (with a given probability), given normal market conditions, in a set time period such as a day" ✅ (Wikipedia, "Value at risk"). The canonical reading: **a one-day 5% VaR of $1 million means there is a 0.05 probability the portfolio falls by $1 million or more over one day — a loss of that size or worse is expected on 1 day out of 20** ✅ (same source). Common parameters are 1% and 5% tail probabilities over one-day and two-week horizons ✅ (same source).

VaR's limits are as verified as its definition: it assumes normal markets and no trading; it does **not** assess the magnitude of loss beyond the threshold (a "VaR breach" can be much worse than VaR — the article's coin-flip example shows a 1% VaR of $0 alongside a possible $12,700 loss) ✅ (same source); and hedge-fund returns are non-normal, so funds "in addition to value at risk and similar measurements… may use integrated measures such as drawdowns" ✅ (Wikipedia, "Hedge fund" — the risk-measurement discussion). The tail-risk supplement is **expected shortfall** (CVaR) ✅ (Wikipedia, "Value at risk" — related measures). For the bank's PB desk, VaR and drawdown limits are the fund's *credit limits in disguise* (§8.6) — and LTCM was the empirical proof that normal-market VaR misses the tail ✅ (§2.4).

### 8.4 Stress Testing (Cross-Referenced)

Stress testing — the exercise of revaluing the book under prescribed shocks (rate moves, spread blowouts, liquidity freezes, counterparty defaults) — is the tool that sees around VaR's normal-market blind spot. The framework, the scenarios, and the governance (risk appetite, ICAAP, the CRO) are the subject of [Enterprise Risk Management](enterprise_risk_management_guide.md) (cross-ref; do not re-derive). The hedge-fund-specific application: the fund's own risk room runs daily stress scenarios (the Citadel-style platforms run hundreds per day — cross-ref [Citadel LLC](citadel_llc_guide.md) §10.2 ✅), and the PB runs its own on the *fund as a borrower* — stressing the portfolio's value, the haircuts, and the fund's ability to post variation margin in a crisis. The 2008 margin-call cascade (a fund's stress becomes the PB's loss) is the scenario every PB stress test is built around ✅ (the 2008 restriction-of-withdrawals record; analysis).

### 8.5 The 2008 Redemption Spiral

The 2008 crisis gave the industry its canonical risk narrative, verified in §2.5: **by 2008 AUM was ~$1.93 trillion; the crisis caused many funds to restrict investor withdrawals; AUM and popularity declined** ✅ (Wikipedia, "Hedge fund"). The spiral mechanics: redemptions forced asset sales into falling markets → marks fell → NAV fell → more redemptions (and PB margin calls on top) → funds invoked **gates and suspensions** (§4.6) → the funds that survived were the ones whose liquidity matched their liabilities. The systemic echo ran through the prime brokers: a hedge fund's failure is its PB's credit loss, and the PB's distress is the fund's frozen financing — the two-sided dependence of §1.4 ✅ (Wikipedia — hedge funds "closely connected to their prime brokers… which could contribute to their instability in a crisis, though this works both ways" ✅).

### 8.6 Risk Management as a Bank's Concern

For Cymbal Bank, the fund's risk is the bank's risk in four places:

1. **PB credit risk.** The margin loan is secured by the portfolio and marked daily; the credit decision rests on concentration, liquidity and leverage — the same three factors that killed LTCM and stressed Citadel in 2008 (cross-ref [Citadel LLC](citadel_llc_guide.md) §10.3 — do not re-derive). Limits, haircuts, and margin-call rights are the bank's risk controls.
2. **Custody and admin risk.** The independent NAV (administrator + auditor) is the collateral's *value*; a Madoff-style custody gap means the collateral does not exist (§2.5, §7.2).
3. **Fund-finance credit risk.** Subscription-line and NAV-facility credit depends on the borrowing base and the NAV quality (§7.3).
4. **Concentration and correlation risk.** The bank's own book of fund clients is correlated — in a 2008-style event, all funds redeem, all margin calls hit together, and the PB's stress is the bank's stress (cross-ref [Enterprise Risk Management](enterprise_risk_management_guide.md) for the bank-wide view).

---

## 9. The Performance

### 9.1 Measuring Returns — the Fund's Own Calculation, TWR, and IRR

A hedge fund's return is computed **on its own NAV series**: (NAV_t − NAV_{t−1})/NAV_{t−1}, net of fees, typically monthly ✅ (the NAV-based dealing of §5.1 makes this the fund's own definition — the administrator's NAV is the price at which investors buy and sell ✅ Wikipedia). Beyond that simple number, three measurement conventions matter:

- **Time-weighted return (TWR):** links sub-period returns geometrically, eliminating the effect of cash flows — the standard for comparing managers (GIPS-compliant firms report TWRR) ✅ (cross-ref [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §7.1 and [Wealth Management](wealth_management_guide.md) — the TWRR/MWRR discussion).
- **Money-weighted return (IRR/XIRR):** reflects the investor's actual experience including the timing of subscriptions and redemptions — the two diverge when investors add or withdraw at good or bad times ✅ (same cross-refs).
- **The fund's own reported return:** usually the TWR-style NAV return, net of all fees — which is why "net vs gross" and "fee drag" are the first questions in any due-diligence review ⚠ (analysis).

For the LP, the honest question is *their* IRR, not the fund's TWR — the same cash-flow-weighting discipline the PE guide applies to DPI/IRR math (cross-ref [Private Equity](private_equity_guide.md) §5.6 — do not re-derive).

### 9.2 Benchmarks — the HFRI Indices and Peer Groups

The benchmark layer is the **HFRI family** (§6.3): the **HFRI Fund Weighted Composite (FWC)** for the industry, and the strategy indices (HFRI Equity Hedge, HFRI Event-Driven, HFRI Relative Value Arbitrage, HFRI Macro, plus sub-strategy indices like HFRI EH: Healthcare) for peer comparison ✅ (HFR — the index family is that source's product; the 2025 strategy returns are verified in §6.3). The benchmark caveats: the FWC is an **equal-weighted average of constituent funds** (it is not investable, and it is survivor/selection-biased by database construction ⚠ — the non-investable vs investable index distinction is documented in the Wikipedia "Hedge fund" treatment of indices ✅), and "beating the FWC" is a peer-relative statement, not an absolute-return statement — which sits awkwardly with the industry's absolute-return branding ✅ (the absolute-return mandate of §1.1; the index-bias caveat ⚠).

### 9.3 GIPS and Composite Presentation (Cross-Referenced)

Fund managers that present track records to institutions increasingly do so under the **Global Investment Performance Standards (GIPS)** — created by the CFA Institute, whose lineage runs AIMR committee 1995 → first edition 1999 → the 2020 edition effective 1 January 2020 ✅ (cross-ref [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §7.1 — the verified GIPS facts and the composite-presentation rules are that guide's content; do not re-derive). The hedge-fund specifics: GIPS requires **composites** (all portfolios with the same strategy, not cherry-picked winners), **portability** rules for track records, and disclosure of fee schedules — the discipline that counters the hedge-fund industry's historical habit of presenting only the flagship fund's returns ✅ (cross-ref; the cherry-picking critique ⚠ directional).

### 9.4 The Alpha/Beta Debate

Hedge funds sell **alpha** — manager skill, return uncorrelated with markets — but pay for a blend of alpha and beta: long/short funds carry net market beta; macro funds carry factor and carry premia; quant funds harvest **alternative betas** (value, momentum, carry, volatility) that are systematic, not skilled ✅ (the factor framing is standard and cross-referenced in the repo — e.g. the strategy descriptions of §3 and the factor discussion in [Asset Management & Alternatives](asset_management_alternatives_guide.md); the alpha/beta decomposition itself is a modelling choice ⚠). The debate's empirical spine: if hedge-fund returns were mostly skilled alpha, performance would persist — and the persistence evidence (§9.5) says otherwise; if they are mostly beta, investors could replicate them more cheaply — which is the "hedge fund replication" school ✅ (Wikipedia, "Hedge fund" — the replication category in its indices treatment). The 2-and-20 fee is only rational if a meaningful part of the return is genuine alpha ✅ (the fee-vs-value debate as documented in Wikipedia's fees discussion — the CalPERS criticism and Buffett's critique).

### 9.5 Persistence of Performance — the Weak-Evidence Finding (⚠)

The general academic finding is that **performance persistence is weak and short-lived** ⚠ — the caveat is real because the literature is genuinely mixed ✅:

- **Hentati-Kaffel & de Peretti (2015):** "nearly 80% of all hedge fund returns are random, with evidence of performance persistence concentrated in hedge funds that follow event-driven and relative value strategies" ✅ (European Financial Management, via the Wiley survey "Persistent Doubt: An Examination of Hedge Fund Performance").
- **Equity long/short specifically:** "only limited evidence of persistence is found for raw returns… there is no persistence in returns beyond 1 year" ✅ (Journal of Derivatives & Hedge Funds, 2015 — Springer).
- **Business-cycle dependence:** persistence appears for up to a year in growth/bull regimes, driven by top performers, and "weakens dramatically during recession periods and bear stock markets" ✅ (Journal of International Financial Markets, Institutions & Money, 2022 — ScienceDirect).
- **The counter-finding:** persistence is documented "for managers who perform well when hedge fund markets are generally weak" ⚠ (Alpha Architect summary of the academic literature).

The practical reading for allocators and banks: **last year's winner is a weak predictor of next year's winner** — which is why due diligence (process, risk management, the service stack) matters more than the return column, and why the industry's own flows data show institutions concentrating in the large platforms (§6.4) ⚠ (analysis).

---

## 10. The Comparison Table — Hedge Fund vs Private Equity vs Mutual Fund vs CTA

| Attribute | Hedge fund | Private equity | Mutual fund (UCITS/long-only) | CTA / managed futures |
| --- | --- | --- | --- | --- |
| **Vehicle** | Open-ended private fund (LP, VCC, offshore shell) | Closed-end private fund (fixed ~10-year life) | Open-ended regulated fund (public) | Open-ended private fund (CFTC-regulated advisor) |
| **Investors** | Accredited/qualified + institutions only | Institutional LPs (accredited) | Retail + institutional | Institutions + qualified individuals |
| **Liquidity** | Periodic — monthly/quarterly dealing, lock-ups, notice, gates | None during the life; exits via distributions or secondaries | Daily dealing at NAV | Often daily/weekly dealing |
| **Assets** | Liquid (equities, bonds, FX, derivatives); some illiquid sleeves | Illiquid private companies | Transferable securities (UCITS constraints) | Exchange-traded futures and options |
| **Leverage** | Standard — PB margin, derivatives; NBER average 1.5–2.5x ✅ | Acquisition debt at the company level (LBO) | Prohibited or tightly capped (UCITS) | Standard — margin on futures |
| **Shorting** | Core tool | Not used (long-only) | Prohibited in long-only wrappers | Core tool (long and short futures) |
| **Fees** | 2-and-20 convention (compressing); on NAV | ~2% + 20% carry on commitments; on committed capital | ~0.1–1% management only | ~1–1.5% + 15–20% incentive |
| **Fee trigger** | High-water mark (+ optional hurdle) | Hurdle (typically 8%) + catch-up + waterfall | None | High-water mark |
| **Benchmark** | Absolute return; HFRI FWC / strategy index | IRR, MOIC, DPI, TVPI vs vintage peers | Index-relative (e.g. S&P 500) | Absolute return; SG CTA Index ⚠ |
| **Regulation** | Light: private-placement exemptions; adviser registration (US 2010), AIFMD (EU) | AIFMD, SEC private-fund rules (vacated 2024), MAS | Full: UCITS directive, SEC registration | CFTC/CTA registration |
| **Transparency** | Low — quarterly letters, position data negotiated | Low — annual reports, quarterly updates | High — daily NAV, full holdings | Low-medium — daily NAV, no positions |
| **Return objective** | Positive absolute return in all markets | Outperform on exits (IRR/MOIC) | Beat the benchmark | Positive in trending markets; crisis diversification |

*Sources per row:* the hedge-fund column is verified in §1–§4; the PE column cross-refs [Private Equity](private_equity_guide.md) §1–§3 (do not re-derive); the UCITS constraints cross-ref §5.3 ✅ (EUR-Lex/EFAMA); the CTA column cross-refs §3.7 ✅ (Wikipedia); the fee and transparency rows are directional market practice ⚠. The point of the table: hedge funds are the *only* column that combines periodic liquidity, leverage, shorting and performance fees — which is exactly why they are both the most flexible institutional product and the most operationally demanding client (§7–§8).

---

## 11. The Cymbal Bank Worked Example — Merlion Long/Short Fund

### 11.1 The Scenario

**Cymbal Bank** (Singapore) onboards **Merlion Long/Short Fund Ltd** — a *fictional* Asia-focused long/short equity fund. The profile: a Cayman master-feeder structure (Cayman master fund + feeder funds; cross-ref [Cayman & BVI Master-Feeder Structures](cayman_bvi_master_feeder_guide.md)) run by **Merlion Capital Pte. Ltd.**, an MAS-licensed A/I LFMC in Singapore (cross-ref [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §2 — do not re-derive); US$250M of committed capital; monthly dealing with a one-year initial lock-up and 60-day redemption notice; a 1.5% management fee and 20% performance fee with a high-water mark and a soft hurdle of SOFR + 3%; a $25M subscription line from the bank; and a mandate to run 200–300% gross exposure across Asia-Pacific equities, hedged with single-stock and index shorts. The example follows the repository's worked-example conventions — see [Market Making in Singapore](market_making_singapore_guide.md) §10 and [Citadel LLC](citadel_llc_guide.md) §10 (do not re-derive) — and illustrates *what a bank sells such a client*; it is not a description of any real fund.

### 11.2 The Facility Terms

| Facility / product | Terms | Notes |
| --- | --- | --- |
| PB margin facility | US$150M revolving; SOFR + 120bps; initial margin 10–50% by asset class (equities 10–15%, single-stock shorts 50% cash collateral) | The fund's leverage engine; variation margin called daily |
| Securities lending | US$150M short-book notional; weighted borrow fee 0.35% p.a.; locates and recall management | The short book's fuel |
| Subscription line | US$25M revolving; SOFR + 150bps; 35% undrawn fee; borrowing base = eligible uncalled commitments | Hedge-fund-style adaptation of the PE product (cross-ref [Private Equity](private_equity_guide.md) §9) |
| Custody | 4bps p.a. on AUM | Plus corporate actions, income collection, DvP settlement |
| FX / treasury | USD/SGD/JPY/HKD forwards at 1–2 pip spread; multi-currency accounts; cash sweeps | The fund's Asia flow settles in four currencies |
| Execution and clearing | 10bps on gross notional; FIX connectivity (cross-ref the repo's FIX guide) | The fund's OMS → Cymbal PB → venues |
| Management fee | 1.5% p.a. on NAV, paid quarterly | Below the 2% convention — fee compression in action (§4.4) |
| Performance fee | 20% above high-water mark; soft hurdle SOFR + 3% | §4.2–§4.3 mechanics |

### 11.3 The Prime-Brokerage and Financing Arithmetic

The fund deploys its US$250M as follows:

| Line | Amount | Derivation |
| --- | --- | --- |
| AUM (committed capital, fully drawn) | US$250.0M | The scenario |
| Long book | US$400.0M | 160% gross long |
| Short book | US$150.0M | 60% short |
| **Gross exposure** | **US$550.0M** | 400 + 150 = **2.20x NAV** |
| **Net exposure** | **US$250.0M** | 400 − 150 = **1.00x NAV** |
| PB margin loan drawn | US$150.0M | Funds the long book beyond equity; secured by the portfolio, marked daily |

**The bank's financing revenue (the fund's financing cost), year 1:**

| Item | Arithmetic | Amount |
| --- | --- | --- |
| Margin-loan interest | $150.0M × (SOFR 3.80% + 1.20%) = $150.0M × 5.00% | US$7.500M |
| Borrow fees on shorts | $150.0M × 0.35% | US$0.525M |
| Execution and clearing | $550.0M gross × 10bps | US$0.550M |
| Custody | $250.0M × 4bps | US$0.100M |
| **Subtotal — PB/custody revenue** | | **US$8.675M** |
| Subscription line, drawn | $10.0M × 5.30% × 0.5 yr (drawn six months) | US$0.265M |
| Subscription line, undrawn | $15.0M × 0.35% × 0.5 yr | US$0.026M |
| **Total bank revenue, year 1** | | **US$8.966M** |

*Check: the relationship's revenue is ~3.6% of the fund's AUM — the arithmetic of why hedge funds are a bank's most profitable institutional client segment ⚠ (relationship economics; the repo's PB examples show the same shape).*

### 11.4 The Fee and Performance Arithmetic

**Year 1 performance, the fund's own books:**

| Line | Arithmetic | Amount |
| --- | --- | --- |
| Long-book P&L | $400.0M × +9.0% | +US$36.000M |
| Short-book P&L | $150.0M × +6.0% (shorted names *fell* 6% → the fund gains) | +US$9.000M |
| **Gross trading profit** | | **+US$45.000M** |
| Financing costs (§11.3) | $7.500M + $0.525M | −US$8.025M |
| Management fee | $250.0M × 1.5% | −US$3.750M |
| Other costs (admin, audit, legal) | $0.150M + $0.250M + $0.200M | −US$0.600M |
| **Net profit before performance fee** | | **+US$32.625M** |
| Performance fee | 20% × $32.625M (above HWM; soft hurdle SOFR+3% = 6.8% cleared) | −US$6.525M |
| **Investor net profit** | | **+US$26.100M** |
| **NAV at year-end** | $250.0M + $26.1M | **US$276.1M** |
| **Net return** | $26.1M / $250.0M | **+10.4%** |

*Checks:* gross return on NAV = 45.0/250 = 18.0%; net of financing and fees = 26.1/250 = 10.4% — a **7.6-point fee-and-financing drag** (management fee 1.5%, performance fee 2.6%, financing 3.2%, other 0.2%) ⚠ (internally consistent by construction, per the repository's worked-example convention). The high-water mark resets to 110.4; if year 2 returns −5%, no performance fee is earned until the fund recovers past 110.4 (§4.2 mechanics).

### 11.5 The KYC/AML Overlay and the Relationship Logic

- **KYC on the structure:** Cayman master-feeder documents → Merlion Capital (A/I LFMC, MAS Financial Institutions Directory) → the fund's investor base, with material holders (≥10%) and controlling persons identified — the fund-client KYC logic of [Private Equity](private_equity_guide.md) §9.3 applies verbatim ✅ (cross-ref).
- **AML/CFT and sanctions:** transaction monitoring across the fund's four-currency Asia flow; sanctions screening of counterparties and venues (cross-ref [FircoSoft](fircosoft_guide.md)); FATCA/CRS on the investor base; the bank's Notice 626/626A obligations — the MAS map in [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §9.3 (cross-ref; do not re-derive).
- **The bank's credit view:** the lending case rests on the *manager's licence and track record*, the *fund's NAV quality* (independent administrator + auditor — the *Anwar* lesson, §2.5), and the *portfolio's liquidity* against its 60-day redemption notice. PB limits: gross leverage cap 300%; single-name concentration cap 10%; 95% one-day VaR limit 2.5% of NAV; a margin call at 10% NAV drawdown (§8.6). The subscription line is secured on the borrowing base of eligible uncalled commitments — the PE template, cross-ref [Private Equity](private_equity_guide.md) §9.1–§9.2.
- **The relationship logic:** the fund gets one bank for financing, custody, clearing, FX and compliance; the bank gets ~$9M of revenue per year, a marquee institutional client, and a natural cross-sell into the manager's future products (a multi-strategy fund, a private-credit sleeve) and the family offices behind the fund (cross-ref [Private Banking](private_banking_guide.md) for the private-banking adjacency). The risk is managed, not eliminated: in a 2008-style event, the fund's redemptions, the margin calls and the bank's own stress all arrive together — which is why the relationship is priced, limited and stress-tested from day one (§8.4–§8.6).

---

## 12. The Claims Audit and What Could Not Be Verified

### 12.1 The Claims Audit — Verified, Flagged, Rejected

| Claim | Status | Source |
| --- | --- | --- |
| A.W. Jones founded the first modern hedge fund, A.W. Jones & Co., in 1949; US$100,000 raised ($40,000 his own); first-year gain 17.3%; coined "hedged fund" | ✅ Verified | Wikipedia, "Alfred Winslow Jones" |
| Jones's model: long/short equity, leverage, 99-investor LP (Investment Company Act 1940 avoidance), 20% of profits + later 2% management fee | ✅ Verified | Wikipedia, "Alfred Winslow Jones" |
| Carol Loomis's 1966 *Fortune* article coined/popularised the term "hedge fund"; Jones outperformed the best mutual fund by 44% over 5 years and Dreyfus by 87% over 10 years; ≥130 funds started within 3 years, incl. Soros's Quantum and Steinhardt Partners | ✅ Verified | Wikipedia, "Carol Loomis" |
| The article's issue date (June 1966) and exact title ("The Jones Nobody Keeps Pace With") | ⚠ Variant — Wikipedia records "The Jones Nobody Keeps Up With", 1966 (bibliography: April 1966 issue) | Wikipedia bibliography; task-brief convention |
| Many hedge funds closed during the 1969–70 recession and the 1973–74 crash | ✅ Verified | Wikipedia, "Hedge fund" |
| Black Wednesday 16 September 1992; UK forced out of the ERM; Soros's short made over £1 billion | ✅ Verified | Wikipedia, "Black Wednesday" |
| LTCM founded 1994 by Meriwether; Scholes/Merton (Nobel 1997); ~21%/43%/41% first three years; lost $4.6B in <4 months in 1998; 23 Sep 1998: 14 banks' $3.65B recapitalisation brokered by the NY Fed; dissolved early 2000 | ✅ Verified | Wikipedia, "Long-Term Capital Management" |
| Global hedge-fund AUM ~US$1.93 trillion by 2008; crisis caused funds to restrict withdrawals; ~$2T by April 2011; record $3.1T July 2017 | ✅ Verified | Wikipedia, "Hedge fund" (2008/2011/2017 figures) |
| Madoff: arrested 11 Dec 2008; SEC charged him the same day (complaint: $50B fraud); guilty plea 12 Mar 2009; 150-year sentence 29 Jun 2009; largest Ponzi in history, ~$65B | ✅ Verified | SEC Press Release 2008-293; SEC Lit. Rel. 20889; Wikipedia, "Madoff investment scandal" |
| *Anwar v. Fairfield Greenwich* (SDNY 2015) — administrator-liability case; $235M settlement 2016 | ✅ Verified | Wikipedia, "Hedge fund" (structure section) |
| UCITS = Council Directive 85/611/EEC of 20 December 1985 | ✅ Verified | EUR-Lex (CELEX 31985L0611) |
| Global industry capital: $4.98T at 3Q25; $5.15T at end-2025 (first time >$5T); FY25 +$642.8B; 2025 net inflows $115.8B (strongest since 2007); HFRI FWC +12.5% in 2025 (best since 2009); EH +17.1%, ED +10.9%; EH capital $1.57T, ED $1.45T, RVA $1.35T, Macro $786.6B | ✅ Verified | HFR press releases (hfr.com), 23 Oct 2025 and 22 Jan 2026 |
| 2Q26: industry capital +$409.3B — largest quarterly rise in history; July 2026: HFRI Technology Index −7.0% (worst since 2008) | ✅ Verified | HFR releases, 23 Jul 2026 and 7 Aug 2026 |
| HFR = Hedge Fund Research, Inc. — publisher of the HFRI indices incl. the HFRI Fund Weighted Composite; index history since 1993; IOSCO-compliant; Kenneth J. Heinz President | ✅ Verified | hfr.com homepage + releases |
| HFR 2022–2024 restructuring/rebranding | ⚠ Not verified — no evidence found this pass | §12.2 |
| Largest firms (June-2025 data): Bridgewater $78.0B; Millennium $77.5B; Elliott $76.1B; Citadel $67.6B; Man Group $66.5B; D.E. Shaw $60.4B; AQR $51.0B; Two Sigma $50.7B; GSAM $48.0B; Renaissance $46.0B; billion-dollar+ managers ≈ $3.6T ≈ 86% of assets | ✅ Verified (ranking vintage ⚠) | InvestmentNews/P&I, 2 Feb 2026 |
| Citadel: ~$67B at start-2026; >$77B Dec 2025 (Bloomberg); $90.4B cumulative gains (LCH, Dec 2025); 2022 record ~$16B | ✅ Cross-referenced (not re-derived) | [Citadel LLC](citadel_llc_guide.md) §3–§4 |
| Management fees 1–4% p.a. (2% standard); performance fees typically 20% (10–50% range); high-water mark = fee only on net profits after loss recovery; hard vs soft hurdle definitions; redemption fees | ✅ Verified | Wikipedia, "Hedge fund" (fees section) |
| Fee compression: average management fees ~1.3–1.5% for established funds; "2-and-20 is history" | ⚠ Secondary/directional | globalinvestments.net; With Intelligence |
| NBER: average investment-bank leverage 14.2 vs hedge funds 1.5–2.5 | ✅ Verified | Wikipedia citing Ang/Gorovyy NBER working paper |
| Gates limit the % of fund assets redeemable on a redemption date; protect against runs | ✅ Verified | Databento compliance guide; Investopedia |
| Side pockets: illiquid-asset compartments; no standard redemption rights; carried at cost; SEC sanctions for misuse; 2008-era prominence | ✅ Verified (definition); ⚠ (2008 prominence detail) | Wikipedia, "Hedge fund" (side pockets) |
| Prime broker: clears trades, provides leverage/financing, counterparty to derivatives, lends securities, custody, execution, clearing | ✅ Verified | Wikipedia, "Hedge fund" (structure) |
| Domicile split 2011: Cayman 34% of fund count; US 24%; Luxembourg 10%; Ireland 7%; BVI 6%; Bermuda 3%; half offshore/onshore | ✅ Verified (vintage ⚠) | Wikipedia citing TheCityUK |
| Drawdown = decline from historical peak; MDD = worst peak-to-valley loss | ✅ Verified | Wikipedia, "Drawdown (economics)" |
| VaR: loss estimate at a given probability over a horizon (e.g. one-day 5% VaR of $1M = ≥$1M loss expected 1 day in 20); normal-market assumptions; tail limits | ✅ Verified | Wikipedia, "Value at risk" |
| Performance persistence is weak/limited; ~80% of hedge-fund returns random; persistence concentrated in event-driven/RV; none beyond 1 year; weaker in recessions | ✅ Verified (literature mixed — ⚠ nuance) | Wiley/Springer/ScienceDirect academic summaries |
| UCITS ≈ 66% of European fund net assets (€15,765B UCITS vs €8,281B AIF, end-Feb 2025); the "UCITS = 40% of EU fund market" figure | ✅ EFAMA figures; ❌ as stated — the 40% claim is unverified and inconsistent with EFAMA data | EFAMA Fact Sheet, February-2025 data |
| GIPS: AIMR committee 1995; first edition 1999; CFA Institute 2020 edition effective 1 Jan 2020 | ✅ Cross-referenced | [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §7.1 |
| Singapore: S$6.7T industry AUM (end-2025); VCC 1,406 vehicles; hedge-fund sleeve ~S$327B (end-2024); 19 Aug 2026 MAS/MOF package | ✅/⚠ Cross-referenced | [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) §1.3, §8.4 |
| Worked-example arithmetic (Merlion: gross 2.2x, net 1.0x; net return +10.4%; bank revenue ~$8.97M) | ✅ Internally consistent | This guide's own arithmetic (§11) |

### 12.2 What Could Not Be Verified

Facts and claims that this pass could not pin to a primary source, flagged ⚠ in the text:

- **The exact issue date and title of Loomis's 1966 article.** The substance (1966, *Fortune*, Loomis, "hedge fund" coinage, the 44%/87% outperformance figures, the 130-fund wave) is verified via Wikipedia's Carol Loomis article; the *date* (task brief: June 1966; Wikipedia's Jones bibliography: April 1966) and *title* ("The Jones Nobody Keeps Pace With" vs Wikipedia's "The Jones Nobody Keeps Up With") vary by source. The Fortune archive itself was not accessible this pass (search returned empty).
- **HFR's founding year and any 2022–2024 restructuring/rebranding.** hfr.com's homepage verifies "Since 1993" for index history and the "HFR®" brand; the commonly-cited 1992 founding year and any corporate restructuring/rebranding in 2022–2024 could not be verified — several searches returned empty, and the Wikipedia "Hedge Fund Research" page does not exist. No rebrand event is asserted in this guide.
- **The "UCITS = 40% of the EU fund market" figure.** Not verified; the EFAMA Fact Sheet (Feb 2025) implies UCITS are ~66% of European fund net assets, so the 40% claim is flagged as unverified (and inconsistent with EFAMA's numbers) rather than repeated.
- **A strategy-level AUM breakdown beyond HFR's four families** (Equity Hedge, Event-Driven, Relative Value Arbitrage, Macro). Any finer split (CTA share, market-neutral share) is provider-specific; the taxonomy itself varies between HFR and BarclayHedge (⚠ §3.1).
- **The geographic split of global AUM** (US/EU/Asia percentages). Only directional anchors were verified (2011 domicile counts; Singapore's MAS numbers); no current regional AUM split was verified this pass.
- **Fee-compression statistics** (average management fees 1.3–1.5%; performance-fee averages). Directionally corroborated by the verified statement that performance-fee rates fell post-credit-crunch (Wikipedia), but the specific modern averages are secondary sources.
- **2008-era side-pocket usage specifics** (which funds side-pocketed what). The side-pocket definition and the SEC's concern/sanctions are verified; the crisis-year inventory is not.
- **Performance-persistence findings.** The direction (weak, short-lived persistence) is verified across the cited academic summaries, but the literature is genuinely mixed — the Alpha Architect counter-finding (persistence for managers who do well in weak hedge-fund markets) is flagged.
- **The "at least 130 hedge funds" post-1966 count** — verified to Wikipedia's Carol Loomis article, which itself relies on a secondary account; not re-extracted from the original article.
- **The 2011 domicile split** (Cayman 34% etc.) — verified to Wikipedia's citation of TheCityUK; vintage is 2011, and the current split (e.g. the VCC's rise) is not re-verified.
- **Market-practice parameters** — gate caps (10–25%), seeding economics, NAV-facility conventions, multi-strategy fee levels (2–3%/25–30%), hedge-fund subscription-line usage — are market conventions flagged ⚠, not verified standards.
- **The worked example's numbers** are fictional and internally consistent by construction; no real fund, mandate or facility is described.

**Method note:** web_search degraded intermittently during this pass (several queries — Loomis's article, LTCM, Soros/Black Wednesday, HFR's corporate history, the "2-and-20" origin — returned empty on first attempts); verification was completed via direct extraction of hfr.com (press releases and homepage), eur-lex.europa.eu, efama.org (Fact Sheet PDF), sec.gov press records, Wikipedia (with primary citations noted), InvestmentNews/P&I, Databento/Investopedia, and the sibling repo guides. The HFR Year-End-2025 release body was recovered from the saved full-text cache of the hfr.com page. No number, date or quote in this guide was invented; everything not directly verified carries ⚠ and a source note.

---

## 13. Glossary, References, and the Closing Line

### 13.1 The Glossary

| Term | Definition |
| --- | --- |
| **Hedge fund** | A pooled, private, open-ended fund using leverage, shorting and derivatives to aim for positive absolute returns (✅ §1) |
| **AUM** | Assets under management — the fund's (or industry's) capital; definitions vary by provider ⚠ (§6.1) |
| **Long/short equity** | Buying undervalued and shorting overvalued stocks; the Jones original (§3.2) |
| **Market-neutral** | A long/short book with net exposure engineered to ~zero, so returns are market-independent (§3.3) |
| **Event-driven** | Positions whose payoff depends on corporate events — mergers, bankruptcies, special situations (§3.4) |
| **Merger arbitrage** | Long target / short acquirer after an announced deal, capturing the deal spread (§3.4) |
| **Distressed** | Buying the debt/equity of companies in or near bankruptcy at discounts (§3.4) |
| **Global macro** | Large directional positions in rates, FX, equities and commodities driven by macro analysis (§3.5) |
| **Relative value** | Long/short positions in related instruments that should converge — the LTCM playbook (§3.6) |
| **CTA** | Commodity trading advisor — a futures-trading fund, mostly systematic trend-following (§3.7) |
| **Managed futures** | The CTA style of trading futures and options across markets, long and short (§3.7) |
| **Quant / systematic** | Strategies selected by computerised models rather than discretion (§3.8) |
| **Multi-strategy** | One platform running several strategies under central risk — the pod architecture (§3.9) |
| **Prime brokerage** | The bank package: clearing, margin financing, securities lending, custody, execution, capital introduction, risk reporting (§7.1) |
| **Margin** | Collateralised, marked-to-market credit against a portfolio; initial margin by product, variation margin called daily (§7.1) |
| **Management fee** | The running fee on NAV (typically 1–4%, 2% standard) that covers the manager's costs (§4.1) |
| **Performance fee** | The incentive fee on profits (typically 20%, 10–50% range), paid above the high-water mark (§4.2) |
| **2-and-20** | The canonical package — 2% management fee + 20% performance fee; Jones's 1949 fee structure ancestor (§4.4) |
| **High-water mark** | The NAV peak a fund must surpass before performance fees resume — fees apply only to net profits after losses are recovered (§4.2) |
| **Hurdle** | The threshold return below which no performance fee is paid; hard = fee on excess only, soft = fee on all returns once cleared (§4.3) |
| **Lock-up** | The initial period during which an investor cannot redeem (typically 1 year) (§4.5) |
| **Redemption** | An investor's withdrawal at the next dealing NAV, subject to notice (typically 30–90 days) (§4.5) |
| **Gate** | A cap on the % of fund assets redeemable on any redemption date; scales down excess requests (§4.6) |
| **Side pocket** | A segregated account for illiquid assets; no standard redemption rights; carried at cost (§4.7) |
| **Master-feeder** | The two-tier structure: a master fund holds the assets; parallel feeders collect investors (cross-ref [Cayman & BVI Master-Feeder Structures](cayman_bvi_master_feeder_guide.md)) (§5.2) |
| **UCITS** | Undertakings for Collective Investment in Transferable Securities — the EU retail wrapper, born of Directive 85/611/EEC (20 Dec 1985) (§5.3) |
| **Drawdown** | The decline from a historical NAV peak; maximum drawdown = worst peak-to-valley loss (§8.2) |
| **VaR** | Value at risk — the loss a portfolio might exceed at a given probability over a horizon, under normal markets (§8.3) |
| **HFRI** | HFR's index family, including the HFRI Fund Weighted Composite (FWC) — the industry's headline benchmark (§6.3) |
| **Capital introduction** | The PB service that introduces funds to institutional investors (§7.1) |
| **Seeding** | Anchor capital (often first-loss or fee-sharing) that launches a young fund (§5.4) |
| **Capacity** | The AUM beyond which a strategy's alpha decays (§5.4, §3.10) |
| **Redemption spiral** | Redemptions → asset sales → falling marks → more redemptions → gates; the 2008 dynamic (§8.5) |
| **GIPS** | Global Investment Performance Standards — the CFA Institute's composite-presentation standards (cross-ref [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §7.1) (§9.3) |
| **Rehypothecation** | A prime broker's re-use of client margin collateral (cross-ref [Citadel LLC](citadel_llc_guide.md) §10.4) |
| **Cymbal Bank** | The bank persona of this research series — the hedge fund's banking partner in §11 |

### 13.2 References and Further Reading

**Primary and verified sources used this pass:**
- **HFR (hfr.com)** — "Global Hedge Fund Industry Capital Surges Past Historic $5 Trillion Milestone" (Year-End 2025 release, 22 January 2026; the Q4-2025 GIR text); "Global Hedge Fund Industry Capital Surges, Nears Historic $5 Trillion Milestone" (3Q25, 23 October 2025); "Hedge Fund Industry Asset Growth Shatters Records" (2Q26, 23 July 2026); "Technology Hedge Funds Suffer Worst Decline Since 2008" (July-2026 flash, 7 August 2026); hfr.com homepage (index history "Since 1993", IOSCO/SOC-2 methodology, Kenneth J. Heinz)
- **EUR-Lex** — Council Directive 85/611/EEC of 20 December 1985 (UCITS), CELEX 31985L0611 (eur-lex.europa.eu)
- **EFAMA** — Investment Fund Industry Fact Sheet, February-2025 data (UCITS net assets €15,765B; AIF €8,281B); Fact Book 2025
- **SEC** — Press Release 2008-293 (11 December 2008, Madoff charges); Litigation Release No. 20889 (complaint alleging a $50B fraud)
- **Wikipedia** (with primary citations noted) — Alfred Winslow Jones; Carol Loomis; Hedge fund; Long-Term Capital Management; Black Wednesday; Madoff investment scandal; Drawdown (economics); Value at risk
- **InvestmentNews / Pensions & Investments** — "Top 10 largest hedge funds by AUM" (Mark Rosanes, 2 February 2026; June-2025 financial-report figures)
- **Databento Trading Compliance Guide** — "What are redemption gates?"; **Investopedia** — "How Gate Provisions Work"
- **Academic literature (via publisher summaries)** — Hentati-Kaffel & de Peretti (2015) and the Wiley "Persistent Doubt" survey (*European Financial Management*); "The performance persistence of equity long/short hedge funds" (*Journal of Derivatives & Hedge Funds*, Springer, 2015); "Hedge fund performance persistence under different business cycles" (*J. Int. Financial Markets, Institutions & Money*, ScienceDirect, 2022); Ang/Gorovyy NBER working paper (leverage comparison, via Wikipedia)
- **Sibling repo guides (cross-referenced, not re-derived)** — [Hedge Funds in Singapore](hedge_funds_singapore_guide.md), [Citadel LLC](citadel_llc_guide.md), [Private Equity](private_equity_guide.md), [Cayman & BVI Master-Feeder Structures](cayman_bvi_master_feeder_guide.md), [Market Making in Singapore](market_making_singapore_guide.md), [Investment Portfolio Operations](investment_portfolio_operations_guide.md), [Enterprise Risk Management](enterprise_risk_management_guide.md), [FircoSoft](fircosoft_guide.md), [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md), [Private Banking](private_banking_guide.md)
- **Secondary/directional (⚠)** — With Intelligence ("Pricing performance" — fee compression); globalinvestments.net (2024–25 fee levels); Alpha Architect (persistence counter-finding); Greenman Open (redemption-suspension guidance); sharemaestro.com (Jones profile)

---

*End of guide — Hedge Funds: The Absolute-Return Craft. The money is pooled, the mandate is absolute, the fees are performance-linked, and the liquidity is periodic — the whole industry still runs on the machinery Alfred Winslow Jones assembled in 1949: the hedge, the leverage, the aligned fee, and the discipline that earns the hedged return.*
