# BNY Software Systems Guide — Custody, Asset Servicing, Payments and Markets at the Bank of New York Mellon

**The per-bank software-systems deep dive for BNY — the custody and asset-servicing bank — from the 1784 Bank of New York to the 2026 platform estate, verified against primary sources.**

> **Author:** Jack Liu Shurui, Solution Architect

**Primary Sources:** SEC filings — FY2025 Form 10-K (filed 2026), Q2 2026 Form 10-Q, Q2 2026 earnings release dated July 15, 2026 (and its financial supplement), mid-2026 8-K cover exhibits; SEC press release 2015-170 (FCPA settlement, August 18, 2015); OFAC civil-penalties enforcement pages 2015–2025; the bny.com corporate sitemap (2,234 URLs) and product-page inventory; sibling-guide cross-references in this repository. No dates, numbers, or product names were invented; unverifiable items are flagged ⚠ inline and collected in [§12](#12-what-could-not-be-verified).

**Verification convention used throughout: ✅ = verified in this research pass against a primary source (SEC filing, SEC or OFAC press, BNY's own corporate pages); ⚠ = flagged (inferred, approximate, single-source, structural, or dated/preliminary); ❌ = disputed (the record contradicts the claim); unmarked = structural or industry knowledge presented as such.** The consolidated claims-status table is in [§11](#11-claims-status-and-verification-notes).

**How to use this guide:** This is the BNY entry of the per-bank software-systems series — the sibling of the [Citibank Software Systems Guide](citibank_software_systems_guide.md). Sections §1–§8 walk the reader from bank profile through the custody, payments, markets, wealth, core, data and risk estates; §9 covers Singapore; §10 is the worked Cymbal Bank example; §11–§12 are the honesty audit. BNY is a *platforms* company before it is a *products* company: most of its revenue is fee income for holding, moving, valuing and financing other institutions' assets, so the "systems" in this guide are mostly client-facing platforms for institutions plus the internal engines behind them. Where BNY discloses a platform name, it is marked ✅ with its source; where the engine identity is not public, the guide says so plainly and flags ⚠.

---

## Table of Contents

1. [Bank Profile: From the Bank of New York to BNY](#1-bank-profile-from-the-bank-of-new-york-to-bny)
2. [Custody and Asset-Servicing Platforms](#2-custody-and-asset-servicing-platforms)
3. [Payments and Treasury Systems](#3-payments-and-treasury-systems)
4. [Markets, Clearance and Collateral Platforms](#4-markets-clearance-and-collateral-platforms)
5. [Wealth, Pershing and the Broker-Dealer Estate](#5-wealth-pershing-and-the-broker-dealer-estate)
6. [Core Systems, Modernization and the Legacy Estate](#6-core-systems-modernization-and-the-legacy-estate)
7. [Data and AI](#7-data-and-ai)
8. [Risk, Financial Crime and the Regulatory Context](#8-risk-financial-crime-and-the-regulatory-context)
9. [The Singapore Angle](#9-the-singapore-angle)
10. [Worked Example: Cymbal Bank as a BNY Custody Client](#10-worked-example-cymbal-bank-as-a-bny-custody-client)
11. [Claims Status and Verification Notes](#11-claims-status-and-verification-notes)
12. [What Could Not Be Verified](#12-what-could-not-be-verified)
13. [Glossary](#13-glossary)
14. [References and Further Reading](#14-references-and-further-reading)

---

## 1. Bank Profile: From the Bank of New York to BNY

### 1.1 Scope and Verification Convention

This guide is the **software-systems deep-dive for BNY** — the corporate brand of **The Bank of New York Mellon Corporation** (✅ Q2 2026 earnings release: "BNY is the corporate brand of The Bank of New York Mellon Corporation"). Where the [Citibank guide](citibank_software_systems_guide.md) describes a universal bank's full stack, BNY is a different animal: a **global financial services platforms company** — the 10-K's own opening description — whose systems exist to *service other financial institutions' assets*: custody and asset servicing, issuance, payments, clearance and collateral, wealth and investment management. Its core "product" is operational infrastructure for the rest of the industry.

The verification discipline is the series standard: **✅ verified** means the claim was confirmed in this pass against a primary source — BNY's own SEC filings (FY2025 Form 10-K, Q2 2026 Form 10-Q, the July 15, 2026 earnings release and its supplement), SEC and OFAC enforcement pages, or BNY's corporate site (its full public sitemap, 2,234 URLs, was inventoried locally). **⚠ flagged** means inferred, approximate, single-source, or structurally reconstructed. **❌ disputed** marks claims the public record contradicts. Every materially non-public item lands in [§12](#12-what-could-not-be-verified).

### 1.2 What Is Public: The BNY Disclosure Reality

BNY sits at an unusual point on the disclosure spectrum this series has documented. **The client-platform layer is unusually public for a services company** — custody, payments, clearance and collateral solutions are *the product*, so they are named and marketed on bny.com (inventoried from the corporate sitemap, §2–§5). **The internal engine layer is nearly opaque** — BNY publishes essentially nothing about its core banking systems, its mainframe estate, its internal ledger and settlement engines, or the names of its risk and data platforms, and this guide does not pretend otherwise (§6–§7 are ⚠-heavy by necessity).

The consequence, enforced throughout: **product-layer pages are ✅-verifiable from bny.com slugs; engine-layer identities are ⚠ not-public**; and the *financial* facts — segments, fee lines, scale, employees, legal entities — are ✅-verifiable from the SEC record, which for a U.S. filer is rich and quarterly.

### 1.3 The Verified History: 1784 → BNY

The corporate story is the oldest continuous banking story in America, and its anchor date is primary-source verified:

- **1784 — "In business since 1784."** The FY2025 10-K states it directly: "With its subsidiaries, BNY has been in business since 1784" ✅ [10-K]. The Bank of New York was founded in New York City in 1784, and the common institutional narrative connects its founding to **Alexander Hamilton**, who wrote its first constitution and was instrumental in its establishment — the "first bank in New York" claim and the Hamilton authorship of the founding constitution are institutional history that this pass did not re-verify from a primary source, so they are flagged ⚠ (institutional history, cross-ref bny.com heritage pages) even though the 1784 lineage itself is ✅ [10-K]. Note: this lineage runs through the **Bank of New York** (the 1784 company and its successors), not through Mellon.
- **2007 — the merger that made "BNY Mellon."** The Bank of New York Company and Mellon Financial Corporation merged in 2007 to form The Bank of New York Mellon Corporation, and the combined entity traded and branded as **BNY Mellon** for nearly two decades. This pass did not find the merger recounted in the FY2025 10-K's own text (the 10-K references the predecessor "The Bank of New York Company, Inc." only in legacy exhibit identifiers), so the 2007 merger and the "BNY Mellon" name are ⚠ (institutional history, corroborated by the current legal name and predecessor references; cross-ref bny.com heritage) — the *result* — the legal name The Bank of New York Mellon Corporation — is ✅ [10-K].
- **2022 — new leadership.** **Robin Vince** became President and Chief Executive Officer in September 2022 ✅ [10-K bio]. He added the Chairman role in September 2025 ✅ [10-K bio]. **Jose Minaya** (age 54) is Global Head of BNY Investments and Wealth, in role since September 2024 ✅ [10-K bio].
- **2025–2026 — the "BNY" brand completes its migration from marketing to the tape.** By the Q2 2026 earnings release (July 15, 2026), the company presents itself simply as **BNY**, and its NYSE listing shows the ticker **BNY** (common stock $0.01 par; preferred "BNY/P" and "BNY PRK") ✅ [Q2 2026 earnings release header; mid-2026 8-K cover exhibits]. The FY2025 10-K cover still showed the legacy ticker **BK** and noted "listed under the ticker symbol BK" as of January 31, 2026 ✅ [10-K]. The ticker therefore changed **BK → BNY at some point in H1 2026; the exact month is ⚠ not pinned in this pass** (see §1.4).

### 1.4 The Rebrand and the BK → BNY Ticker Change

| Claim | Source | Status |
|---|---|---|
| "BNY is the corporate brand of The Bank of New York Mellon Corporation" | Q2 2026 earnings release (July 15, 2026) | ✅ verified |
| FY2025 10-K cover lists common stock under ticker symbol **BK**; ~18,530 holders of record as of Jan 31, 2026 | FY2025 Form 10-K | ✅ verified |
| Q2 2026 earnings release header and mid-2026 8-K covers show **NYSE: BNY** (common), BNY/P and BNY PRK (preferred) | Q2 2026 earnings release; 8-K cover exhibits (d119098d8k.htm … d417322d8k.htm) | ✅ verified |
| Ticker changed BK → BNY during H1 2026 | Inference from the two dated observations above | ✅ verified (direction); ⚠ exact month not pinned |
| The "BNY" short brand was announced/rolled out in a specific calendar year (press-reported as May 2024) | Press only; not in SEC filings checked | ⚠ not verified in this pass |

For a systems guide the ticker matters less than what it signals: by mid-2026 the corporation's public identity is fully collapsed to **BNY**, and this guide follows that usage (the legal names The Bank of New York Mellon Corporation and BNY Mellon, N.A. remain, §1.7).

### 1.5 Business Segments and the Q2 2026 Fee Lines

BNY runs **three principal business segments plus an Other segment** ✅ [10-K, 10-Q, Q2ER]:

1. **Securities Services** — the custody and asset-servicing engine (Asset Servicing, Issuer Services); the largest segment.
2. **Market and Wealth Services** — three sub-lines: **Wealth Solutions, Payments and Trade, Clearance and Collateral Management** ✅ [10-Q segment description].
3. **Investment and Wealth Management** — investment management and wealth.
4. **Other** — corporate treasury activities, including the securities portfolio ✅ [10-K].

The Q2 2026 fee detail (all ✅ Q2 2026 earnings release / supplement, dollars in millions):

| Segment line | Q2 2026 | Q1 2026 | Q2 2025 | Note |
|---|---|---|---|---|
| Asset Servicing (investment services fees) | 1,209 | 1,170 | 1,082 | Securities Services |
| Issuer Services | 463 | 278 | 376 | Securities Services |
| Securities Services total investment services fees | 1,672 | 1,448 | 1,458 | plus FX revenue $203m |
| Wealth Solutions | 551 | 544 | 525 | Market and Wealth Services |
| Payments and Trade | 224 | 220 | 209 | Market and Wealth Services |
| Clearance and Collateral Management | 453 | 430 | 385 | Market and Wealth Services |
| Market and Wealth Services total fee revenue | 1,333 | 1,300 | 1,212 | includes FX revenue $34m |
| Investment management fees | 793 | 785 | 748 | Investment and Wealth Management |
| Performance fees | 3 | 1 | 10 | Investment and Wealth Management |

Whole-company Q2 2026 results (✅ Q2 2026 earnings release): total revenue **$5.7bn (+13% YoY)**, noninterest expense $3.4bn (+7%), net income **$1,792m**, diluted EPS **$2.45** (+27%), pre-tax operating margin **39.8%**, ROTCE **31.3%**, CET1 ratio **11.0%**, Tier 1 leverage ratio 5.9%, average deposits **$314bn**. The earnings narrative credits "continuous investments and strategic actions" and "600 basis points of positive operating leverage" — the recurring modernization language analyzed in §6.

### 1.6 Scale: AUC/A and AUM

BNY's defining metric is **assets under custody and/or administration (AUC/A)** — assets BNY holds and services *on behalf of other institutions* — plus assets under management (AUM):

| As of | AUC/A | AUM | Source |
|---|---|---|---|
| June 30, 2026 | **$62.6T** (preliminary) | **$2.2T** | ✅ Q2 2026 earnings release; Q2 2026 Form 10-Q (preliminary ⚠) |
| Dec 31, 2025 | **$59.3T** | $2.2T | ✅ FY2025 Form 10-K |
| June 30, 2025 | **$55.8T** | — | ✅ Q2 2026 Form 10-Q (prior-year comparative) |

The June 30, 2026 AUC/A figure is flagged ⚠ as **preliminary** (so labeled in the earnings release); the year-end 2025 figure is the audited-adjacent 10-K number. The gap between AUC/A ($62.6T) and AUM ($2.2T) is the whole BNY business model in two numbers: ~96% of what BNY touches it does not manage, it *services* — safekeeping, settlement, accounting, and reporting for the world's asset managers, banks and issuers (§2).

### 1.7 The Legal-Entity Structure: National Bank, Trust Companies, Broker-Dealer

For a systems guide the legal-entity map matters because it determines which regulator oversees which platform:

- **BNY Mellon, N.A.** — the national bank subsidiary (OCC-supervised), which per the 10-K houses **Wealth Management and certain Pershing activities** ✅ [10-K]. This is where the depository, custody and payments balance sheet sits.
- **Trust companies**: The Bank of New York Mellon Trust Company, N.A.; BNY Mellon Trust of Delaware; BNY Mellon Investment (trust company) ✅ [10-K] — the corporate-trust and depositary-activity vehicles.
- **Pershing LLC** — a U.S. broker-dealer subsidiary that holds customer cash and securities in its custody operations ✅ [Q2 2026 10-Q]; full treatment in §5.
- **Employees**: approximately **48,100 full-time employees** worldwide at Dec 31, 2025, ~60% based outside the U.S. (including ~10,100 in Europe) ✅ [10-K]. (An earlier extract in the research pass carried a ~53,000 figure; the 10-K text verified here reads 48,100.)

### 1.8 Milestones Timeline

| Year | Milestone | Status |
|---|---|---|
| 1784 | Bank of New York founded; BNY "in business since 1784" | ✅ [10-K]; Hamilton/first-bank-in-NY detail ⚠ institutional history |
| 2007 | Bank of New York Company + Mellon Financial Corporation merge → The Bank of New York Mellon Corporation ("BNY Mellon") | ⚠ institutional history (resulting legal name ✅ [10-K]) |
| Aug 18, 2015 | SEC 2015-170: BNY Mellon pays $14.8m to settle FCPA charges (internship hiring) | ✅ SEC press release |
| Sep 2022 | Robin Vince becomes President and Chief Executive Officer | ✅ [10-K] |
| Sep 2024 | Jose Minaya becomes Global Head of BNY Investments and Wealth | ✅ [10-K] |
| Sep 2025 | Robin Vince adds Chairman of the Board | ✅ [10-K] |
| Dec 31, 2025 | AUC/A $59.3T; AUM $2.2T; ~48,100 employees | ✅ [10-K] |
| H1 2026 | NYSE ticker changes BK → BNY | ✅ direction [10-K vs 8-K/ER]; ⚠ month |
| Jul 15, 2026 | Q2 2026 results: revenue $5.7bn; AUC/A $62.6T (preliminary) | ✅ earnings release |
---

## 2. Custody and Asset-Servicing Platforms

### 2.1 What Custody Means at BNY

Custody at BNY is the operational core of the bank: BNY holds financial assets on behalf of the world's asset managers, pension funds, insurers, banks and official institutions, and performs the daily operational work around them — safekeeping, transaction settlement, income collection, corporate-action processing, valuation and accounting, and regulatory reporting. The scale context from §1.6 applies: **$62.6T of assets under custody and/or administration** as of June 30, 2026 (preliminary ⚠) against only $2.2T of AUM — BNY is an *operator* of other people's assets, not primarily a manager of its own. The Asset Servicing fee line of $1,209m in Q2 2026 (✅ earnings release) is the revenue expression of this estate.

The *mechanics* of custody — safekeeping and settlement (including the US T+1 settlement cycle), DVP (delivery-versus-payment), the IBOR/ABOR distinction (investment-book-of-record vs accounting-book-of-record), NAV calculation, and corporate-action processing — are series-standard material already derived in the sibling [Investment Portfolio Operations Guide](investment_portfolio_operations_guide.md), and this guide does not re-derive them. What follows is the *platform inventory*: what BNY publicly names and sells in this space, and what it does not disclose.

### 2.2 The Public Custody-Solutions Page Inventory

BNY's corporate sitemap (2,234 URLs, inventoried locally from bny.com) exposes a **platforms/custody-solutions/** tree with 13 URLs whose slugs name the marketed solution set (✅ bny.com corporate sitemap; one-line descriptors are structural ⚠ where the page copy was not fetched):

| Public slug (bny.com/.../solutions/platforms/custody-solutions/) | Marketed capability | Status |
|---|---|---|
| safekeeping-transaction-settlement-services | Core custody: safekeeping plus settlement of securities transactions | ✅ page exists; ⚠ engine internals not disclosed |
| corporate-events-management | Corporate-action processing across asset classes and markets | ✅ page exists; ⚠ internals not disclosed |
| prime-custody-solutions | Custody tailored to prime-brokerage / hedge-fund clients (typically financing-adjacent, daily NAV and margin flows) | ✅ page exists; ⚠ scope not detailed |
| cash-liquidity-solutions | Cash management and liquidity services on custodial cash | ✅ page exists |
| tax-services-global-investors | Tax reclaim and tax-operations services for global investors | ✅ page exists |
| trustee-depositary-services-global-asset-protection | Trustee and depositary services (fund depositary in Europe; global asset-protection structures) | ✅ page exists |
| growth-dynamics | Related solutions page | ✅ page exists |

The **fund-investor-solutions** tree (23 URLs) markets the transfer-agency / fund-services layer: investor record-keeping, order processing and distribution services for fund managers — the classic BNY fund-services franchise where the bank keeps the investor register and processes subscriptions and redemptions (✅ sitemap; ⚠ page copy not fetched). The **middle-office-solutions** and **managed-account-solutions-platform** trees (separate URL families) round out the asset-servicing product set for outsourced middle-office and managed-account operating models (✅ sitemap; ⚠ specifics).

### 2.3 The Global Depositary Receipts Franchise

BNY is one of the world's major depositary banks for **Depositary Receipts (DRs)** — the sponsored and unsponsored American/Global Depositary Receipt programs that let non-US companies list their shares on US and other exchanges. The corporate sitemap carries a dedicated **platforms/global-depositary-receipts/** tree (4 URLs) for the franchise (✅ sitemap). BNY's marketing commonly claims leadership in this business; **this pass did not verify the "leading DR depositary bank" ranking from a primary source**, so that specific claim is ⚠ (marketing; unverified here) — the *existence* of the franchise and its dedicated platform pages are ✅. The Issuer Services fee line ($463m in Q2 2026, +23% YoY — ✅ earnings release) is the segment revenue that includes DR and corporate-trust activity, and the jump versus Q1 2026 ($278m) suggests episodic issuance volumes, consistent with the DR business model (⚠ inference from the fee pattern).

### 2.4 Eagle Investment Systems: The Verified Software Spine

A custody and asset-servicing estate of BNY's kind does not run on the marketed pages alone — it runs on an internal *investment-operations software layer* that provides the IBOR/ABOR, accounting and data infrastructure both for BNY's own Asset Servicing operations and for the outsourced middle-office services it sells. BNY's public record in this layer centers on **Eagle Investment Systems**, which this repository has already treated in depth: the verified Eagle spine — founded 1989, the PACE data-management launch in 1996, the STAR accounting platform in 1999, acquisition by Mellon Financial in 2001 (making it a BNY Mellon company from the 2007 merger), and the **2017 ICE deal that did NOT complete** — is documented in §10.7 of the [Investment Portfolio Operations Guide](investment_portfolio_operations_guide.md). Rather than re-derive that history, this guide adopts it as a cross-reference: Eagle-class software (accounting, data management, performance) is the plausible engine family behind BNY's asset-servicing and data-and-analytics solutions (§7), and the 2017 ICE episode is the repository's canonical example of an announced fintech acquisition that collapsed ✅/⚠ [per the sibling guide's sources].

### 2.5 The Custody Estate: Public vs Not

The honest inventory of the custody estate, in the series' ⚠-discipline:

| Layer | What is public | Status |
|---|---|---|
| Marketed solutions | custody-solutions, fund-investor-solutions, DR, middle-office, managed-account page trees | ✅ bny.com sitemap |
| Scale | $62.6T AUC/A (Jun 30, 2026, preliminary), $59.3T (Dec 31, 2025) | ✅ SEC filings |
| Segment revenue | Asset Servicing fees $1,209m, Issuer Services $463m (Q2 2026) | ✅ earnings release |
| Software spine (Eagle family) | History and capabilities | ✅ [Investment Portfolio Operations Guide](investment_portfolio_operations_guide.md) §10.7 |
| Custody core / ledger engine | Safekeeping ledger, settlement engine, corporate-action engine identities | ⚠ not disclosed |
| Market-by-market sub-custody network | Extent of proprietary branches vs agent-bank sub-custodians per market | ⚠ not disclosed in sources used |
| IBOR/ABOR implementation detail | Whether BNY runs Eagle STAR as the ABOR for client books | ⚠ structural inference |
| "Leading depositary bank" ranking | Marketing claim | ⚠ unverified in this pass |

The pattern repeats what §1.2 predicted: **page-level transparency, engine-level opacity**. For a bank that *sells* custody operations, the page trees are the product catalog; for a systems guide, the engines behind them are the interesting unknown, and they stay in [§12](#12-what-could-not-be-verified).
---

## 3. Payments and Treasury Systems

### 3.1 The Payments Business at a Custody Bank

Payments at BNY are not a consumer business; they are the **institutional money-movement layer** of the custody and asset-servicing franchise. The same clients whose assets BNY holds (§2) also need their cash moved: custody cash sweeps, tri-party and collateral cash, FX settlement, issuer and corporate-trust disbursements (dividends, bond redemptions), and the bank's own treasury flows. BNY is also, through its national bank subsidiary, a **USD clearer** in its own right — a participant in the US payment and settlement infrastructure on behalf of its institutional clients (⚠ structural; BNY does not publish its CHIPS/Fedwire membership in the sources used).

The *mechanics* of institutional payments — SWIFT messaging, the ISO 20022 migration, nostro/vostro accounting, and the US clearing rails (Fedwire, CHIPS, ACH, and the real-time rails) — are derived in the sibling [Payment Rails Guide](payment_rails_guide.md) and are cross-referenced rather than re-derived here. What BNY discloses about its own payments estate is thin but real, and is inventoried below.

### 3.2 The Global Payments and Trade Platform

The corporate sitemap carries a **platforms/global-payments-trade/** tree with 16 URLs (✅ bny.com sitemap) — the public home of BNY's institutional payments and trade offering. From the URL family alone the marketed scope is cross-border and domestic payments for financial institutions and corporates, plus trade services; ⚠ the individual page copy was not fetched in this pass, so product names below this level (channel names, portal names) are not asserted.

The one firm financial anchor is the **Payments and Trade fee line: $224m in Q2 2026**, up 7% YoY and 2% QoQ (✅ Q2 2026 earnings release). Payments and Trade sits inside the **Market and Wealth Services** segment (✅ Q2 2026 Form 10-Q segment description), and its fee trend — slow, steady growth against the double-digit growth of Asset Servicing — is consistent with a mature, volume-driven institutional payments book (⚠ inference).

### 3.3 Liquidity, Cash and Treasury Solutions

The **cash-liquidity-solutions** slug appears both in the custody-solutions tree (§2.2) and conceptually across the platform estate, and the sitemap also shows a **liquidity-financing-solutions/** tree (✅ sitemap). What BNY does *not* publicly confirm in the sources used here is a named, branded liquidity portal of the type other banks market (for example Citi's CitiDirect BE family in the [Citibank guide](citibank_software_systems_guide.md)): the sitemap inventory does include URLs plausibly mapping to a **LiquidityDirect**-style offering, but **"LiquidityDirect" as a product name could not be verified from a fetched primary page in this pass** and is therefore ⚠ not asserted as a current product name — any such name belongs to [§12](#12-what-could-not-be-verified) until a page fetch confirms it.

### 3.4 The Payments Estate: Public vs Not

| Layer | What is public | Status |
|---|---|---|
| Marketed platform | global-payments-trade tree (16 URLs); liquidity-financing-solutions tree | ✅ bny.com sitemap |
| Segment revenue | Payments and Trade fees $224m (Q2 2026) | ✅ earnings release |
| Segment home | Payments and Trade inside Market and Wealth Services | ✅ Q2 2026 Form 10-Q |
| Payments engine | The internal payment-hub / orchestration engine behind the platform | ⚠ not disclosed |
| SWIFT/ISO 20022 posture | BNY as a SWIFT member-community bank; specific gateway and ISO 20022 roadmap | ⚠ not disclosed in sources used |
| USD clearing memberships | Fedwire/CHIPS participation | ⚠ structural (see [Payment Rails Guide](payment_rails_guide.md)) |
| Channel/portal names | E-banking and API channel names for corporate clients | ⚠ not verified (sitemap slugs only) |
| Liquidity portal branding | "LiquidityDirect"-type name | ⚠ unverified in this pass |

The disclosure pattern is the book's §1.2 pattern again: the *business* is public (fee line, segment, marketing tree), the *machinery* is not. For a bank whose payments business is institutional and balance-sheet-light relative to its custody franchise, the interesting systems questions — how custodial cash sweeps into money funds, how tri-party cash is moved, how the USD clearing is plumbed — sit in the ⚠ zone and are collected in [§12](#12-what-could-not-be-verified).


## 4. Markets, Clearance and Collateral Platforms

### 4.1 The Markets Business of a Custody Bank

BNY's markets activities are the trading and financing layer attached to its custody franchise — not a proprietary trading house, but an **execution, FX and financing services** business for institutional clients. The public marketing tree is **platforms/execution-services/** — at 401 URLs the largest single family in the corporate sitemap, dominated by global-markets-trading pages (✅ bny.com sitemap). The structure of the tree names the businesses:

| Public slug (within execution-services/) | Marketed capability | Status |
|---|---|---|
| global-markets-trading-foreign-exchange | Institutional FX: spot, forwards, swaps, options for custody and fund clients | ✅ sitemap; ⚠ engine not disclosed |
| iflow | BNY's named FX analytics/flow-intelligence product family (separate iflow slug) | ✅ sitemap; ⚠ capabilities from marketing only |
| global-markets-trading-fixed-income | Fixed-income sales and trading for institutions | ✅ sitemap |
| global-markets-trading-equity-sales-trading | Equity sales trading | ✅ sitemap |
| buy-side-trading-solutions-global-execution | Multi-asset execution solutions aimed at the buy side | ✅ sitemap |
| portfolio-transition-services | Transition management — moving portfolios between managers/benchmarks | ✅ sitemap |
| bny-pershing-trading-services-for-banks-and-broker-dealers (under capital-markets-execution-services/) | Trading services that BNY's Pershing unit offers to banks and broker-dealers | ✅ sitemap; ⚠ see §5 |

FX deserves one structural note: for a custody bank, FX is **embedded in custody flows** — dividend and coupon conversions, NAV hedging, and the settlement of cross-border trades generate constant natural FX demand, and the FX revenue lines in the segment disclosures ($203m in Securities Services, $34m in Market and Wealth Services, Q2 2026 — ✅ earnings release) show FX monetized inside the servicing segments rather than as a standalone trading desk P&L. iFlow is BNY's public-facing brand in this space, and its existence is ✅ (sitemap slug); what the iFlow engine actually contains is marketing-level only ⚠.

The *mechanics* of market data consumption that feed such an estate — feed architectures, reference data, pricing sources — are derived in the sibling [Market Data Consumption Guide](market_data_consumption_guide.md) and cross-referenced rather than re-derived.

### 4.2 Clearance: Global Clearing Solutions

The sitemap carries a **global-clearing-solutions/** tree (3 URLs) (✅ sitemap) — BNY's marketed clearance offering. In BNY's world, "clearance" spans the settlement and clearing services the bank operates for the US market and for cross-border flows, including its role in **US government securities clearing** (⚠ structural: BNY's clearance franchise historically centers on the clearance of Treasuries and other fixed income; the specific clearing memberships and the Fixed Income Clearing Corporation relationship are not stated in the sources used here). The revenue anchor is the **Clearance and Collateral Management fee line: $453m in Q2 2026**, up 18% YoY and 5% QoQ — the fastest-growing line inside Market and Wealth Services (✅ Q2 2026 earnings release).

### 4.3 Collateral: The Global Collateral Management Platform

The sitemap carries a **collateral-management/** tree (3 URLs) explicitly hosting the **Global Collateral Management Platform** (✅ sitemap). This is the heart of BNY's **tri-party collateral management** franchise — the business where BNY acts as the neutral third party that holds and values collateral between two counterparties, most prominently in the **US tri-party repo market** where BNY Mellon, N.A. is one of the two tri-party clearing banks that run the market's settlement infrastructure (⚠ structural/industry knowledge; BNY's public pages market the platform but the tri-party market role is asserted from industry structure, not from a fetched primary page). The Global Collateral Management Platform is the client-facing system through which repo and derivatives collateral is exchanged, valued and optimized (✅ sitemap for platform existence; ⚠ all functional detail).

### 4.4 The Markets, Clearance and Collateral Estate: Public vs Not

| Layer | What is public | Status |
|---|---|---|
| Execution-services tree | 401 URLs incl. FX (iFlow), fixed income, equity sales, buy-side, transitions, Pershing trading services | ✅ bny.com sitemap |
| Clearance tree | global-clearing-solutions (3 URLs) | ✅ bny.com sitemap |
| Collateral platform | Global Collateral Management Platform pages (collateral-management tree) | ✅ bny.com sitemap |
| Revenue | Clearance and Collateral Management $453m; FX revenue $203m + $34m (Q2 2026) | ✅ earnings release |
| Tri-party repo market role | BNY as a tri-party clearing bank in US repo | ⚠ structural/industry knowledge |
| Trading engines | FX pricing/execution engine, fixed-income inventory systems, iFlow internals | ⚠ not disclosed |
| Collateral optimization engines | Collateral-eligibility, allocation and optimization logic | ⚠ not disclosed |
| Clearing memberships | FICC/DTCC relationships and settlement memberships | ⚠ not disclosed in sources used |

Collateral is where BNY's systems load is arguably heaviest: tri-party collateral management is a real-time, high-volume operational business (every US tri-party repo trade requires the collateral to be valued, allocated and moved intraday), and the Global Collateral Management Platform is the rare BNY system that is both *named in public* and *genuinely load-bearing* — yet its internals (⚠) are exactly what BNY does not publish. The honest position: platform name ✅, architecture ⚠, engine identity in [§12](#12-what-could-not-be-verified).


## 5. Wealth, Pershing and the Broker-Dealer Estate

### 5.1 Wealth at BNY: Inside the National Bank

BNY's wealth business is **Wealth Solutions** — the private-bank style offering for wealthy individuals and families — and, per the FY2025 10-K, it is housed inside **BNY Mellon, N.A.**, the national bank subsidiary: the 10-K states that BNY Mellon, N.A. houses **Wealth Management and certain Pershing activities** ✅ [10-K]. The revenue line is the **Wealth Solutions fee line: $551m in Q2 2026** (up 1% QoQ, 5% YoY — ✅ Q2 2026 earnings release), inside the Market and Wealth Services segment. Wealth Solutions is BNY's *smallest* revenue line of the ones disclosed, consistent with a bank whose core is institutional servicing; the wealth franchise exists partly to serve the individuals behind the institutional clients (⚠ structural: BNY does not publish this strategic framing; it is inferred from the business mix).

The wealth-technology tree exists on bny.com — a **wealth-technology/** URL family (✅ sitemap) — but with a single URL slug in the inventory, its marketed scope could not be characterized beyond existence ⚠. What systems sit behind Wealth Solutions (advisory workstations, portfolio reporting, trust accounting) is not public ⚠.

### 5.2 Pershing: The Broker-Dealer Estate

**Pershing** is BNY's clearing and custody business for the wealth-management and broker-dealer industry — the unit that provides clearing, custody, trading and technology services to registered investment advisers, independent broker-dealers and banks. Its verified anchors:

- **Pershing LLC is a U.S. broker-dealer subsidiary** that "holds customer cash and securities" — the Q2 2026 Form 10-Q describes Pershing LLC in exactly these custody terms ✅ [Q2 2026 10-Q].
- Pershing has its own public web presence with its own sitemap (inventoried locally), including its flagship platform **NetX** at /pershing/us/en/platforms/netx.html plus netx-investor pages ✅ (Pershing sitemap, URL-verified). NetX is the broker-dealer/advisor front end: account opening, trading, clearing and reporting for the advisors and brokers that Pershing serves (⚠ functional description from URL context and industry knowledge; page copy not fetched).
- The **Pershing trading-services-for-banks-and-broker-dealers** pages appear inside BNY's execution-services tree (§4.1) — Pershing sells execution services onward to other banks and broker-dealers ✅ sitemap.
- Pershing's history — acquired by BNY from **CSFB (Credit Suisse First Boston) in 2003** — is the common institutional account, but **this pass did not verify the 2003 acquisition from a primary source**, so it is ⚠ (institutional history; unverified here). What is ✅ is the *current* legal reality from the filings: Pershing LLC is a BNY broker-dealer subsidiary holding customer assets as of June 30, 2026.

### 5.3 The 2025–2026 Sale Question

A press-reported story during the research window concerned a possible sale of Pershing (press reports referenced Apex Group as a buyer). The verification result is precise and worth stating plainly:

- **As of June 30, 2026, Pershing LLC remains a BNY subsidiary** — the Q2 2026 Form 10-Q describes it as a going concern of the group with no sale reflected ✅ [Q2 2026 10-Q: no divestiture of Pershing appears in the filing's subsidiary or segment descriptions].
- Any **2025 press announcement of a sale is ⚠ unverified in this pass** — no fetched primary source (BNY press release or SEC filing) confirmed a signed transaction; the absence from the Q2 2026 10-Q is the operative ✅ fact.

For a systems guide the Pershing question matters because Pershing carries its own platform estate (NetX and the clearing stack behind it), and a divestiture would move that estate out of the BNY systems boundary. On the current ✅ record, it has not.

### 5.4 The Wealth and Pershing Estate: Public vs Not

| Layer | What is public | Status |
|---|---|---|
| Wealth legal home | Wealth Management inside BNY Mellon, N.A. | ✅ FY2025 10-K |
| Wealth revenue | Wealth Solutions fees $551m (Q2 2026) | ✅ earnings release |
| Pershing legal status | Pershing LLC, U.S. broker-dealer, holds customer cash/securities; still a BNY subsidiary Jun 30, 2026 | ✅ Q2 2026 Form 10-Q |
| Pershing platform | NetX (pershing.com platform URLs, incl. netx-investor) | ✅ URL-verified |
| Pershing trading services | Sold to banks and broker-dealers; pages in execution-services tree | ✅ sitemap |
| Pershing 2003 acquisition from CSFB | Institutional history | ⚠ unverified in this pass |
| 2025-26 Pershing sale (press-reported, Apex Group) | No sale in Q2 2026 10-Q; press only | ✅ no-sale-as-of-Jun-2026; ⚠ 2025 press |
| Wealth technology stack | Wealth-technology URL family (single slug); advisory/trust systems | ⚠ not disclosed |
| NetX internals | Clearing engine, custody ledger, advisor workstation architecture | ⚠ not disclosed |
| RIA custody platform depth | Pershing's RIA custody offering details | ⚠ not disclosed in sources used |
---

## 6. Core Systems, Modernization and the Legacy Estate

### 6.1 The Honest Position: The Core Is Not Public

The first fact about BNY's core systems is that **BNY does not publicly detail them**. There is no BNY equivalent of the modernization narratives this series documents for other banks — no public naming of the core banking platform, the custody ledger, the mainframe estate, the payments hub, or the data warehouse. The FY2025 10-K, the Q2 2026 10-Q and the earnings releases describe *businesses and results*, not architecture; the 10-K's technology discussion is confined to risk factors (cybersecurity, operational risk, vendor dependence) and human-capital themes. This guide therefore states the class-level reality and flags every identity-level claim ⚠:

- A bank holding **$62.6T of AUC/A** (Jun 30, 2026, preliminary — ✅ earnings release) necessarily runs a **mainframe-scale, batch-plus-real-time operational estate**: securities ledgers, corporate-action engines, payment hubs, and client accounting that process hundreds of millions of transactions daily (⚠ structural inference; the scale is ✅, the estate description is the industry-standard class for a bank of this size).
- The series' sibling guides document the *classes* of core systems such an estate belongs to — the [Core Banking Systems Guide](core_banking_systems_guide.md) for ledger/deposit cores and the [Investment Portfolio Operations Guide](investment_portfolio_operations_guide.md) for the investment-operations layer — and BNY's custody-ledger and accounting functions plausibly sit in the Eagle-family and vendor-or-in-house categories those guides describe (⚠ structural; cross-referenced, not asserted as fact).
- **No named core system, no named mainframe application suite, and no named custody ledger appears anywhere in the primary sources used in this pass** — that absence is itself the finding, and the identities are listed as unknowns in [§12](#12-what-could-not-be-verified).

### 6.2 What IS Public: The Transformation Language

The public record of BNY's modernization is strategic and financial, not architectural:

- The Q2 2026 earnings release credits "**continuous investments and strategic actions to enable differentiated solutions**" and reports **600 basis points of positive operating leverage** with revenue of $5.7bn and a 39.8% pre-tax margin ✅ [Q2 2026 earnings release]. The phraseology — investing while expanding margin — is the standard signature of a bank running an efficiency/transformation program (⚠ interpretation).
- The Q2 2026 release narrative also references "**growth was broad-based across**" all three segments "**reflecting healthy client activity**" — i.e., the public story is demand-led growth plus technology-enabled efficiency, with no system-level detail (✅ for what the release says; ⚠ for any architectural reading).
- BNY's public communications during the window consistently describe the company as a "**platform**" company and emphasize cloud and AI investment themes ⚠ (theme-level, from corporate positioning in the earnings narrative; no named cloud contracts, no named platforms, no dollar figures specific to technology appeared in the primary sources used).

### 6.3 The Legacy Estate Reality

The honest treatment of BNY's legacy estate, in the series discipline:

| Question | Answer on the record | Status |
|---|---|---|
| Does BNY run a large legacy/mainframe estate? | Class-certain for a 240-year-old custody bank of this scale | ⚠ structural inference |
| Are any legacy system names public? | No — none found in 10-K, 10-Q, or earnings releases | ✅ verified absence |
| Is there a public cloud-migration narrative? | Only theme-level ("continuous investments", platform positioning) | ⚠ |
| Is there a public core-replacement program? | No named program in the sources used | ✅ verified absence |
| Are technology spend figures disclosed? | No separate technology-spend line in the sources used (unlike some peers in this series) | ✅ verified absence |

The closest the series gets to a BNY modernization anchor is **Eagle Investment Systems** (the 2017 ICE deal that did not complete, and Eagle's continuing role in the investment-operations layer — see [Investment Portfolio Operations Guide](investment_portfolio_operations_guide.md) §10.7 and §2.4 of this guide) and the **Global Collateral Management Platform** (§4.3), the two publicly named pieces of software infrastructure in an otherwise unnamed estate. Every other engine — the custody ledger, the transfer-agency system, the corporate-actions engine, the DR depositary system, the trust-accounting platform, the payments hub — is a §12 unknown.

### 6.4 What a Systems Guide Can Say Honestly

For readers who need to reason about BNY's core estate, the defensible statements are: (1) the *scale* of the operational load is verified from AUC/A and fee volumes ✅; (2) the *class* of systems required to carry that load is industry-standard structural knowledge, cross-referenced to the sibling mechanics guides ⚠; and (3) the *identities* of the systems are not public and must be treated as unknown, not guessed ⚠. That is the same honesty discipline the [Citibank guide](citibank_software_systems_guide.md) applies to Citi's engine layer, applied to an estate that discloses even less about its core.


## 7. Data and AI

### 7.1 Why Data Is the Product at BNY

For a custody bank, data is not a support function — it is close to the product. Every asset under custody generates a daily stream of positions, prices, corporate actions, income and FX events, and BNY's clients pay for the *packaging* of that data: accounting books, valuations, performance attribution, regulatory reporting and analytics. The **data-and-analytics** platform tree on bny.com (9 URLs, ✅ sitemap) is therefore the public catalog of BNY's data product line:

| Public slug (platforms/data-and-analytics/) | Marketed capability | Status |
|---|---|---|
| accounting-solutions-investment-management | Investment accounting solutions (the ABOR layer for investment managers) | ✅ sitemap; ⚠ internals not disclosed |
| data-management-solutions | Data management (the Eagle PACE-lineage space, §2.4) | ✅ sitemap; ⚠ internals |
| investment-operations-solutions | Investment operations outsourcing / middle-office data operations | ✅ sitemap |
| performance-solutions-portfolio-performance | Portfolio performance measurement and attribution | ✅ sitemap |
| managed-professional-services-investment-data-management | Managed services running clients' investment data operations | ✅ sitemap |

The **managed professional services** slug is the telling one: BNY will *run* a client's investment-data operations on its own stack — which is precisely the Eagle-family managed-services model documented in the [Investment Portfolio Operations Guide](investment_portfolio_operations_guide.md) §10.7 (⚠ structural mapping; the Eagle lineage of the data-management line is a cross-referenced inference, not a BNY disclosure).

The mechanics of how an institution like BNY *consumes* external market data — vendor feeds, pricing hierarchies, reference-data management, the golden-copy problem — are derived in the sibling [Market Data Consumption Guide](market_data_consumption_guide.md) and are cross-referenced rather than re-derived; BNY's internal consumption patterns are not disclosed and are ⚠.

### 7.2 The AI Position

BNY's public AI posture in the sources used is theme-level: the earnings releases and filings of the window describe investment in technology and AI-enabled efficiency in general terms ("continuous investments... to enable differentiated solutions" — ✅ Q2 2026 earnings release) without naming platforms. Specifics:

- **No named BNY AI platform was verified in this pass.** Press-reported names for BNY's internal generative-AI tooling (reported in industry press under names such as "Elixir") were **not confirmed against a primary source** in this pass and are therefore ⚠ unverified — they are candidates for [§12](#12-what-could-not-be-verified), not asserted facts. (The naming convention in this guide is strict: a product name that cannot be sourced to a fetched primary page is not written as fact.)
- The FY2025 10-K's technology discussion is confined to risk and human-capital language (AI appears in the risk-factor framing of the competitive and operational environment) — ✅ for what the 10-K says; ⚠ for any inference about the internal AI estate.
- For the *class* of AI use, the custody context supplies the obvious candidates: corporate-action interpretation, document and contract processing, client-service copilots, and code modernization — all structural ⚠ inference from industry practice, none verified for BNY specifically.

### 7.3 The Data and AI Estate: Public vs Not

| Layer | What is public | Status |
|---|---|---|
| Data product catalog | data-and-analytics tree (accounting, data management, operations, performance, managed services) | ✅ bny.com sitemap |
| Data lineage | Eagle-family history and managed-services model | ✅ [Investment Portfolio Operations Guide](investment_portfolio_operations_guide.md) §10.7 (cross-ref) |
| Data platform internals | Warehouse/lake architecture, golden-copy implementation, vendor feeds | ⚠ not disclosed |
| AI platform | No named platform verified | ⚠ press-reported names unverified (§12) |
| AI spend/scope | Theme-level only in earnings language | ⚠ |
| Market-data consumption | Mechanics per sibling guide; BNY specifics | ⚠ [Market Data Consumption Guide](market_data_consumption_guide.md) cross-ref |

The summary sentence for §7 is the book's refrain with a data twist: at BNY the *data products sold to clients* are public (the page tree), the *data machinery* that produces them is not, and the AI estate is real at the theme level (⚠) with no verified public name. A future research pass with access to BNY press releases and product documentation could move items out of §12; this pass did not have that budget.


## 8. Risk, Financial Crime and the Regulatory Context

### 8.1 The AML and Financial-Crime Posture

A custody bank is a **conduit** — it holds and moves assets for other institutions, which makes it structurally exposed to financial-crime risk in two directions: the assets and flows it services for clients, and the individuals behind the wealth business (§5). BNY's public posture is the standard top-tier-bank one: the 10-K and 10-Q describe AML/KYC/sanctions compliance programs, board-level risk governance, and regulator-imposed capital and compliance expectations as part of the standard risk-factor and supervisory landscape ✅ [10-K, 10-Q — program *existence* is disclosed in boilerplate; no internal system names are given ⚠].

The *mechanics* of the screening systems a bank of this class runs — sanctions-list screening, name-matching, fuzzy logic, and the FircoSoft-family engines that dominate the market — are derived in the sibling [FircoSoft Guide](fircosoft_guide.md) and are cross-referenced rather than re-derived. **No BNY-specific screening engine name is public** ⚠; the FircoSoft guide documents the industry-standard class that BNY's unnamed engines plausibly belong to, nothing more.

### 8.2 The Verified Enforcement Record

The distinguishing feature of BNY's enforcement record in this pass is that it is **small and old**, and both halves of that statement are verified:

- **FCPA (2015) — ✅ primary source.** SEC press release 2015-170 (August 18, 2015): BNY Mellon agreed to pay **$14.8m** — $8.3m disgorgement, $1.5m prejudgment interest, and a $5m penalty — to settle SEC charges that it violated the Foreign Corrupt Practices Act by providing **student internships to family members of foreign government officials affiliated with a Middle Eastern sovereign wealth fund**, in order to win or retain business from the fund ✅ [SEC press release 2015-170]. This is the single largest verified enforcement event in BNY's modern record and it is a *conduct* case (hiring practices), not a systems/AML case.
- **OFAC (2015–2025) — ✅ verified negative.** Each year's OFAC civil-penalties enforcement page from 2015 through 2025 was downloaded and searched; **no BNY, Bank of New York, or Mellon entity appears in any OFAC civil-penalty entry in that eleven-year window** ✅ [OFAC enforcement pages 2015–2025]. This is a verified negative: as of the research date, the public OFAC record shows no BNY sanctions penalty in the period — stated exactly and no stronger than that (absence from OFAC's published penalty pages is not absence of sanctions *exposure*, which every global bank carries ⚠).

The contrast with the enforcement-heavy profiles of some peers in this series (see the [Citibank guide](citibank_software_systems_guide.md) §8 on the 2020 consent orders) is worth stating plainly: BNY's verified public enforcement record in the sources used is one FCPA matter in 2015 and no OFAC penalties in 2015–2025 — a materially cleaner record than the peer set, on the evidence available ✅/⚠ (clean *record* ✅; "clean *program*" would overreach ⚠).

### 8.3 The US Regulatory Context

The US supervisory map for BNY, from the filings:

- **BNY Mellon, N.A.** is a national bank — **OCC-supervised** as to its safety-and-soundness and trust activities ✅ [10-K entity description: national bank; the OCC as its primary federal regulator is structural ✅]. The trust companies (§1.7) and the broker-dealer Pershing LLC (SEC/FINRA-regulated by class ⚠ structural) complete the regulated-entity set.
- **The Bank of New York Mellon Corporation** is a bank holding company supervised by the **Federal Reserve**, and as one of the largest US banking organizations it sits in the enhanced-supervision tier (the 10-K's capital disclosures — CET1 11.0% at Q2 2026, ✅ earnings release — reflect Fed capital rules) ✅ [10-K holding-company status; enhanced-supervision tier ⚠ structural].
- **Resolution planning**: as a large US BHC, BNY is subject to the Fed/FDIC resolution-plan (living-will) regime, and the 10-K discusses resolution and recovery planning in its standard regulatory discussion ✅ [10-K mentions resolution-related matters in boilerplate]; **no BNY resolution-plan deficiency was found in the sources used** ✅ (verified absence — the pass found no FDIC/Fed public finding against BNY's plan comparable to the peer episodes documented in the [Citibank guide](citibank_software_systems_guide.md)).

### 8.4 The Risk, Financial Crime and Regulatory Estate: Public vs Not

| Layer | What is public | Status |
|---|---|---|
| FCPA settlement (Aug 18, 2015) | $14.8m total ($8.3m disgorgement + $1.5m interest + $5m penalty); internship hiring | ✅ SEC 2015-170 |
| OFAC penalties 2015–2025 | No BNY/Bank of New York/Mellon entries on any year's page | ✅ verified negative |
| Regulators | OCC (BNY Mellon, N.A.), Federal Reserve (holding company), SEC (broker-dealer class) | ✅ 10-K entities; ⚠ class-level for SEC/FINRA |
| Capital | CET1 11.0%, Tier 1 leverage 5.9% (Q2 2026) | ✅ earnings release |
| Resolution planning | Subject to living-will regime; no public deficiency found in this pass | ✅ 10-K boilerplate; ✅ verified absence of findings |
| AML/KYC/sanctions systems | Program existence in filing boilerplate; no engine names | ✅ existence; ⚠ identities (§12) |
| Screening engines | FircoSoft-family class per sibling guide | ⚠ [FircoSoft Guide](fircosoft_guide.md) cross-ref |

The regulatory chapter of the BNY story is, on the verified record, the *quiet* one — which is itself the finding. The systems this guide cannot see (the screening engines, the transaction-monitoring stack, the sanctions-filtering layer) are the same §12 unknowns as the core estate, but the *outcome* evidence — one 2015 FCPA matter, zero OFAC penalties in eleven years — is public, verified, and unusually clean for a bank of BNY's centrality to the world's settlement infrastructure.

---

## 9. The Singapore Angle

Singapore is the city where BNY's Asian institutional franchise is anchored in the public record: bny.com maintains an **asia-pacific locations tree** under about-us/locations with a dedicated Singapore page (✅ bny.com sitemap), and Singapore is the one Asian jurisdiction that surfaces inside the Q2 2026 10-Q's own geography tables (✅ [10-Q], §9.1). For a custody bank, Singapore matters as the regional booking-and-servicing centre for Asian cross-border asset flows — the location from which Asian asset managers' and banks' US and European custody is serviced — though that role statement is ⚠ structural inference from the entity roster below, not a BNY disclosure.

### 9.1 Verified Presence: What the Q2 2026 10-Q Shows

The filing-level evidence of a Singapore balance-sheet presence is precise:

- **Q2 2026 10-Q country-concentration table (foreign covered bonds)**: Singapore is listed at **$529m** as of June 30, 2026, against **$607m** at December 31, 2025, in the same table that rows Canada, the UK, Australia, Germany, Belgium, Austria and the Netherlands ✅ [Q2 2026 Form 10-Q]. ⚠ Precision note: the parent research pass's cheat sheet characterized this line as a "deposit line"; direct reading of the filing text places the line in the **foreign-covered-bonds** country table of the securities portfolio. The substance — a Singapore line inside the 10-Q's country geography — is unchanged ✅.
- **Foreign-currency hedging**: the 10-Q's cash-flow hedge discussion names the **Singapore dollar** among the currencies (Indian rupee, euro, Polish zloty, British pound, Singapore dollar, Hong Kong dollar) used in revenue and expense transactions of entities with USD functional currency ✅ [Q2 2026 10-Q] — corroborating that BNY runs operating entities in Singapore whose local-currency costs it hedges.
- The FY2025 10-K text contains **no Singapore mention** in the passages reviewed ✅ (verified absence), so the 10-Q table plus the corporate location page are the operative evidence.

### 9.2 The Singapore Legal-Entity Roster (Verified This Pass)

A web_extract of the bny.com Singapore location page (one extract, run this pass) verified the current entity set at **7 Straits View, #27-01 Marina One East Tower, Singapore 018936** ✅ [bny.com location page, fetched]:

| Entity on the location page | Reading for this guide | Status |
|---|---|---|
| **BNY, Singapore Branch** | The branch vehicle; which group legal entity (e.g., BNY Mellon, N.A.) operates the branch is not stated on the page | ✅ entity; ⚠ parent entity not stated |
| BNY Mellon Investment Management Singapore Pte. Limited | Investment-management distribution/servicing entity (ties to §1.5 IWM segment) | ✅ entity |
| BNY Mellon Institutional Services (Asia Pacific) Pte. Ltd. | The asset-servicing/custody-services vehicle name (ties to §2) | ✅ entity |
| **Eagle Investment Systems Singapore Pte. Ltd.** | Eagle's Singapore legal entity — the §2.4 software spine has an onshore Singapore presence | ✅ entity |
| **Pershing Securities Singapore Private Limited** | Pershing's Singapore securities entity (ties to §5.2) | ✅ entity |

**⚠ the establishment year could not be re-verified in this pass** — the fetched location page lists offices and entities but no founding date, and the web searches run this pass returned empty result sets; the year (common institutional accounts date Bank of New York's Singapore branch to the late 1970s, but that is unverified here) is routed to [§12](#12-what-could-not-be-verified).

### 9.3 What BNY Does in Singapore: The Source-Supported Picture

Only what a source supports is asserted. Entity names (§9.2) support, at ⚠ structural reading: **custody and asset servicing for Asian clients** (Institutional Services (Asia Pacific) Pte. Ltd.), **investment management** (Investment Management Singapore Pte. Limited), **Eagle software and managed data services** (Eagle Investment Systems Singapore Pte. Ltd., consistent with §2.4 and §7.1), and **Pershing-style securities services** (Pershing Securities Singapore Private Limited, §5). The bny.com sitemap adds URL-level evidence of specific activities, ⚠ un-fetched in this pass: a press-release URL announcing the **launch of an FX pricing and trading engine in Singapore** (a systems-relevant data point for §4.1's FX estate, but the date and content were not fetched), a press-release URL on BNY "stepping up commitment" to Singapore's digital and fintech ecosystem, and a company-news URL on BNY Mellon being appointed trustee for a catastrophe bond domiciled in Singapore (an issuer-services/corporate-trust data point for §2.3's franchise). None of these three pages was fetched; they are ⚠ sitemap-level existence only.

### 9.4 MAS Regulatory Context (Cross-Referenced, Not Re-Derived)

⚠ Structural: a global custody bank's Singapore operations sit inside MAS's licensing and supervisory framework — the branch under MAS banking licensing as a foreign bank branch, the Pte. vehicles under the securities/fund-management licensing families relevant to their activities — and MAS's outsourcing and technology-risk expectations (the outsourcing guidelines and the Technology Risk Management notices) apply squarely to a custody-and-asset-servicing estate that runs client operations onshore. The statutes, notices and licence classes themselves are derived in the sibling [MAS Regulations and Guidelines Guide](mas_regulations_guidelines_guide.md) and are cross-referenced rather than re-stated here; that guide, together with the Singapore-market guides in this repository, forms the series' Singapore thread, of which this section is only BNY's corner. No MAS-specific finding about BNY (licence numbers, enforcement, supervisory actions) was verified in this pass ⚠.

### 9.5 The Singapore Estate: Public vs Not

| Layer | What is public | Status |
|---|---|---|
| Geography | $529m line in 10-Q foreign-covered-bonds country table (Jun 30, 2026) | ✅ Q2 2026 Form 10-Q |
| Entities | Branch + four Pte. entities at Marina One (roster above) | ✅ bny.com location page (fetched) |
| Activities | Entity-name and sitemap-URL level only (custody, IWM, Eagle, Pershing, FX engine launch) | ⚠ structural / un-fetched pages |
| Establishment year | Not on any source used; searches empty | ⚠ not verified (§12) |
| MAS licensing detail | Licence classes per sibling guide; no BNY-specific licence numbers verified | ⚠ cross-ref [MAS Regulations and Guidelines Guide](mas_regulations_guidelines_guide.md) |
| Headcount / office detail | Not public in sources used | ⚠ not disclosed |

### 9.6 Singapore in the Systems Picture

For a systems guide the Singapore story is the book's public/not-public split in miniature. The **verified** layer is the entity roster (§9.2) plus the 10-Q geography (§9.1). The roster itself carries systems significance: **Eagle Investment Systems Singapore Pte. Ltd.** puts the §2.4 software spine onshore in Singapore, and **Pershing Securities Singapore Private Limited** puts a Pershing vehicle in the same market — both ⚠ structural readings from entity names, consistent with the onshore servicing model that a MAS-regulated custody and asset-servicing operation follows (the licensing, outsourcing-notice and technology-risk detail is derived in the [MAS Regulations and Guidelines Guide](mas_regulations_guidelines_guide.md), cross-referenced not re-stated here). What stays unknown is equally clear: the branch's establishment year, headcount, MAS licence numbers, and which group entity operates the branch — each is in [§12](#12-what-could-not-be-verified). Finally, in the repository's wider Singapore thread — the MAS guide and the Singapore-market guides of this series — the worked examples of Asian banks routinely route their US-market and cross-border legs through a global custodian of BNY's class; the Cymbal Bank example of §10 shows that pattern from the custodian's side (⚠ structural framing, not a BNY disclosure).

---

## 10. Worked Example: Cymbal Bank as a BNY Custody Client

### 10.1 Design-Fiction Notice and the Persona

**This worked example is design fiction, constructed to show how the verified public platform layer of §2–§5 would serve a real client archetype — it is not a description of an actual BNY client engagement.** Cymbal Bank is the fictional bank persona of this repository's worked examples: a mid-sized Asian bank (it is also the client in the Citibank guide's worked scenario). Every BNY platform named below is one this guide has already verified from the public layer (✅ sitemap/filings); every process detail that BNY does not publish — contract terms, fee schedules, account structures, operating procedures — is marked ⚠ structural or ⚠ design fiction and invents **no** product name beyond those in §2–§5.

### 10.2 Why Cymbal Needs BNY

Cymbal Bank's wealth-management arm is expanding into **cross-border funds distribution** — distributing foreign (mainly US and European) funds to its wealthy clients — and into **global custody** for those clients' overseas securities. Cymbal is a strong regional bank but it has no proprietary presence in the US market: it cannot safekeep US securities, settle US trades, process US corporate actions, or hold USD clearing balances itself. The structural answer for a bank in Cymbal's position is a **global sub-custodian and asset servicer**: BNY holds the US-market assets in its own books and performs the operational work, while Cymbal keeps the client relationship. This is the classic global-custodian/agent-bank pattern this series documents — the mechanics are in the [Investment Portfolio Operations Guide](investment_portfolio_operations_guide.md) — and BNY is one of the two or three banks big enough to carry it at scale (⚠ structural: BNY's $62.6T AUC/A ✅ [Q2 2026 earnings release] is the scale context, not a claim about Cymbal's mandate).

### 10.3 Onboarding and KYC

Opening the relationship follows the correspondent/agent-bank pattern:

1. **Due diligence on Cymbal** — BNY's onboarding runs standard institutional due diligence: Cymbal's licences and regulatory status in its home jurisdiction, its ownership and management, its AML program, and its expected activity profile (⚠ structural: standard practice; BNY publishes no onboarding procedure).
2. **Correspondent-account and custody documentation** — a custody agreement, an account terms schedule, and the documentation package for a non-US financial institution holding US securities (⚠ design fiction: document names and terms are invented; the *class* of documentation is standard).
3. **Screening** — Cymbal and its signatories are screened against sanctions and watch lists as part of onboarding, and every future payment instruction is screened. The *mechanics* of the name-matching and fuzzy-logic engines that do this work — the FircoSoft-family class that dominates the market — are derived in the [FircoSoft Guide](fircosoft_guide.md) and cross-referenced rather than re-derived. No BNY-specific screening engine name is public ⚠ (§12).

### 10.4 The Accounts and Rails

The account structure the mandate implies (⚠ design fiction, but built only from verified BNY legal entities of §1.7):

- **A USD cash account at BNY Mellon, N.A. (New York)** — the national bank subsidiary that carries the custody balance sheet ✅ [10-K]. This account is Cymbal's US settlement and income account: trades settle against it, dividends and coupons credit to it, and Cymbal's treasury uses it for USD payments. In nostro terms this is Cymbal's USD account on BNY's books — the mirror image of the nostro mechanics derived in the [Payment Rails Guide](payment_rails_guide.md), cross-referenced rather than re-derived.
- **Securities accounts on BNY's custody books** — one or more safekeeping accounts per Cymbal client segment, holding the US equities, bonds and fund interests. Whether positions are held omnibus or segregated per Cymbal's underlying customers is a contractual choice ⚠ design fiction.
- **Messaging rails** — Cymbal connects over SWIFT (and the ISO 20022 message set as the industry migrates) to send settlement instructions, receive statements and confirmations, and move cash; channel specifics (gateway, API or portal) are ⚠ not public for BNY and are not asserted. The SWIFT/ISO 20022 mechanics live in the [Payment Rails Guide](payment_rails_guide.md).

### 10.5 The Service Map: Which Verified BNY Platforms the Mandate Touches

| Cymbal need | BNY public platform layer | Where in this guide | Evidence status |
|---|---|---|---|
| Safekeeping and settlement of US-market securities | custody-solutions: safekeeping-and-transaction-settlement services | §2.2 | ✅ page exists; ⚠ engines |
| Corporate-action processing (dividends, mergers, tenders) | custody-solutions: corporate-events-management | §2.2 | ✅ page exists; ⚠ engines |
| Cash and liquidity on settlement/income balances | custody-solutions: cash-liquidity-solutions | §2.2, §3.3 | ✅ page exists |
| Cross-border USD payments and FX conversion of income | global-payments-trade tree; FX embedded in servicing | §3.2, §4.1 | ✅ sitemap + fee lines |
| Fund distribution support for Cymbal's fund clients | fund-investor-solutions tree (investor record-keeping/order processing) | §2.2 | ✅ sitemap; ⚠ page copy |
| Outsourced investment-operations/middle-office services | middle-office-solutions; data-and-analytics: investment-operations-solutions | §2.2, §7.1 | ✅ sitemap slugs |
| Client reporting and accounting data | data-and-analytics tree (accounting, data management, performance) | §7.1 | ✅ sitemap |
| Tax reclaims on US-source income for Cymbal's wealth clients (if elected) | custody-solutions: tax-services-global-investors | §2.2 | ✅ page exists; ⚠ election is design fiction |
| Depositary receipts — **as applicable** if Cymbal issuer clients seek US listings | global-depositary-receipts tree | §2.3 | ✅ sitemap; ⚠ "leading" claim |
| Tri-party collateral — **as relevant** if Cymbal's treasury runs repo with US counterparties | Global Collateral Management Platform | §4.3 | ✅ platform name |
| Not engaged | Pershing clearing/wealth stack (US advisor market), BNY wealth-tech | §5 | ⚠ out of scope for this persona |

### 10.6 Operating Narrative: A Trade, a Reconciliation, a Corporate Action, the Fees

The step-by-step picture (mechanics cross-referenced; BNY-specific detail ⚠ design fiction):

1. **The trade.** A Cymbal wealth client buys US-listed equity. The client's order executes through Cymbal's own execution channel; Cymbal then needs the US settlement leg done by its sub-custodian.
2. **The instruction.** Cymbal sends BNY a settlement instruction over SWIFT/ISO 20022 — buy X shares, settle against Cymbal's USD account, DVP (delivery-versus-payment) so cash and securities move together.
3. **Settlement.** BNY's safekeeping-and-settlement operation settles the trade in the US market on a **T+1** cycle through the US clearing and settlement infrastructure, and books the position to Cymbal's securities account and the cash to Cymbal's USD account. The T+1, DVP and settlement-mechanics detail is derived in the [Investment Portfolio Operations Guide](investment_portfolio_operations_guide.md) and the [Payment Rails Guide](payment_rails_guide.md); ⚠ the internal settlement engine that does this at BNY is not public (§12).
4. **The daily reconciliation.** Each day BNY reports positions, cash and activity to Cymbal; Cymbal reconciles its own books (its accounting book of record) against BNY's statements, investigates breaks, and resolves them by re-instruction or claim. The IBOR/ABOR mechanics behind "Cymbal's own books" are sibling-guide material ([Investment Portfolio Operations Guide](investment_portfolio_operations_guide.md)); BNY's reporting formats are ⚠ contractual.
5. **The corporate action.** The US issuer announces a cash dividend. BNY's corporate-events-management service captures the event, notifies Cymbal of the record and payment dates, processes any elections, and on payment date credits the dividend to Cymbal's USD account — where, per standing instructions, it may be converted to the client's base currency through BNY's embedded custody FX (the $203m Securities Services FX line of Q2 2026 ✅ [earnings release] is the revenue expression of exactly this kind of flow). ⚠ Process detail is the standard corporate-action pattern, not a BNY disclosure.
6. **The fees.** Cymbal pays the custody norm: basis-point fees on assets under custody for safekeeping, per-instruction settlement charges, FX spreads on income and trade conversions, corporate-action processing charges, and data/reporting subscription fees. ⚠ Design fiction: no schedule is public; the *shape* of the fee stack is the industry-standard custody revenue model that BNY's own fee lines (§1.5) reflect.
7. **The reports.** BNY delivers accounting, valuation and performance data through its data-and-analytics services (§7.1) — the packaging of Cymbal's custody data that is, at BNY, close to the product itself.

**The steady state.** Once the mandate is live, the cadence is the industry-standard custody operating pattern: daily reconciliation of positions and cash against BNY's statements; weekly and monthly income reconciliation (dividends and coupons booked by BNY against Cymbal's entitlement records); corporate-action election windows; and fee-billing reviews. Settlement cash that Cymbal leaves with BNY may be swept under the cash-liquidity-solutions line (§2.2, ✅ page exists) into liquidity products — ⚠ design fiction: sweeps are contractual and BNY publishes no sweep mechanics. All of this cadence is cross-referenced to the [Investment Portfolio Operations Guide](investment_portfolio_operations_guide.md); none of it describes a BNY-published procedure.

The worked example ends where the guide's evidence ends: the **relationship pattern is verified** (global custodian/agent-bank structure, BNY's platform catalog, its legal entities and fee lines, all ✅), and the **specifics are flagged** (⚠ design fiction: contracts, schedules, account structures, engine identities). That split — pattern verified, particular not — is the same honesty boundary the [Citibank guide](citibank_software_systems_guide.md) draws around its worked example.

### 10.7 The Boundary of the Mandate: What Cymbal Does Not Touch

The mandate's boundary is as instructive as its content. **Cymbal does not use Pershing's clearing stack** (§5.2) — Pershing serves US advisors, broker-dealers and their customers, not an Asian bank's cross-border custody book (⚠ structural: BNY publishes no customer segmentation that would confirm or deny this). **Cymbal does not buy BNY's wealth-technology tree** (§5.1), which serves BNY's own Wealth Solutions estate inside BNY Mellon, N.A. And if Cymbal's treasury later runs US tri-party repo, the **Global Collateral Management Platform** (§4.3) would be the named BNY system on the other side of that mandate — a separate onboarding with its own agreements ⚠. The service map of §10.5 is therefore not the whole BNY catalog; it is the slice of the catalog that a mid-sized Asian bank's cross-border wealth-and-funds mandate plausibly engages, and the table marks each slice with the evidence for its existence.

The boundary also frames the risk conversation a Cymbal treasury would run before signing. Custody assets held by BNY sit in BNY's books under the custody agreement, and the protections a client relies on — segregation of custody assets, the insolvency remoteness of client securities, and the capital and resolution-planning context of the custodian (§8.3: CET1 11.0% at Q2 2026 ✅, no public resolution-plan deficiency ✅) — are the standard class-level considerations for choosing a global custodian (⚠ structural framing; the mechanics are sibling-guide material in the [Investment Portfolio Operations Guide](investment_portfolio_operations_guide.md), not BNY disclosures about Cymbal).

---

## 11. Claims Status and Verification Notes

**Verification convention (as defined in §1.1): ✅ = verified in this pass against a primary source; ⚠ = flagged (inferred, approximate, single-source, structural, dated/preliminary, or not re-verified here); ❌ = disputed (the public record contradicts the claim).** This table consolidates the key factual claims of §1–§10.

### 11.1 The Claims-Status Table

| Claim | Source | Status |
|---|---|---|
| "In business since 1784"; Bank of New York founded 1784 | FY2025 Form 10-K | ✅ verified |
| Alexander Hamilton wrote the founding constitution; "first bank in New York" | Institutional history (bny.com heritage class) | ⚠ not re-verified this pass |
| 2007 merger of Bank of New York Company and Mellon Financial → BNY Mellon | Institutional history; resulting legal name The Bank of New York Mellon Corporation | ⚠ history; ✅ legal name [10-K] |
| Robin Vince President and CEO since Sep 2022; Chairman since Sep 2025 | FY2025 Form 10-K bio | ✅ verified |
| Jose Minaya Global Head of BNY Investments and Wealth since Sep 2024 | FY2025 Form 10-K bio | ✅ verified |
| Three principal segments (Securities Services; Market and Wealth Services; Investment and Wealth Management) + Other | 10-K / 10-Q / Q2 earnings release | ✅ verified |
| Market and Wealth Services = Wealth Solutions + Payments and Trade + Clearance and Collateral Management | Q2 2026 Form 10-Q | ✅ verified |
| AUC/A $62.6T (preliminary) and AUM $2.2T at Jun 30, 2026; AUC/A $59.3T at Dec 31, 2025 | Q2 2026 earnings release / 10-Q; FY2025 10-K | ✅ verified (Jun-2026 figure ⚠ preliminary) |
| Q2 2026 fee and result lines (Asset Servicing $1,209m … net income $1,792m, EPS $2.45, CET1 11.0%) | Q2 2026 earnings release + supplement | ✅ verified |
| ~48,100 employees at Dec 31, 2025 | FY2025 Form 10-K | ✅ verified (an earlier extract's ~53,000 reading was superseded by the 10-K text, §1.7) |
| "BNY is the corporate brand of The Bank of New York Mellon Corporation" | Q2 2026 earnings release | ✅ verified |
| Ticker BK → BNY during H1 2026 (BK on FY2025 10-K cover; BNY on Q2 2026 release and 8-K covers) | 10-K vs earnings release / 8-K exhibits | ✅ direction; ⚠ exact month not pinned |
| Press-reported May 2024 announcement of the "BNY" brand | Press only | ⚠ not verified this pass (searches empty) |
| Pershing LLC is a US broker-dealer holding customer cash and securities; still a BNY subsidiary Jun 30, 2026 | Q2 2026 Form 10-Q | ✅ verified |
| Pershing acquired from CSFB in 2003 | Institutional history | ⚠ not verified this pass |
| NetX is Pershing's advisor/broker-dealer platform | Pershing sitemap (URL-verified) | ✅ URL; ⚠ content not fetched |
| FCPA settlement Aug 18, 2015: $14.8m total ($8.3m + $1.5m interest + $5m penalty), internship hiring | SEC press release 2015-170 | ✅ verified |
| No BNY/Bank of New York/Mellon entry on OFAC civil-penalty pages 2015–2025 | OFAC enforcement pages (each year downloaded) | ✅ verified negative |
| No BNY resolution-plan deficiency found in sources used | 10-K boilerplate + verified absence | ✅ verified absence |
| Singapore appears in Q2 2026 10-Q country table at $529m (Jun 30, 2026) | Q2 2026 Form 10-Q (foreign-covered-bonds country table) | ✅ verified (⚠ not a deposit line; see §9.1) |
| Singapore legal-entity roster (BNY Singapore Branch, Eagle Investment Systems Singapore Pte. Ltd., Pershing Securities Singapore Pte. Ltd., etc.) | bny.com Singapore location page | ✅ verified this pass (web_extract) — upgrade from ⚠ |
| BNY Singapore establishment year | No source used states it; searches returned empty | ⚠ not verified (§12) |
| Platform inventories: custody-solutions (13 URLs), execution-services (401), fund-investor-solutions (23), global-payments-trade (16), data-and-analytics (9), DR (4), collateral-management (3), global-clearing-solutions (3) | bny.com corporate sitemap (2,234 URLs) | ✅ sitemap-verified |
| Global Collateral Management Platform; iFlow; NetX; Eagle spine | Sitemap / sibling guide | ✅ names; ⚠ internals |
| Eagle Investment Systems history and the 2017 ICE deal that did not complete | [Investment Portfolio Operations Guide](investment_portfolio_operations_guide.md) §10.7 (cross-ref) | ✅/⚠ per sibling guide's sources |
| Internal engine identities (custody ledger, core, payments hub, screening engines) | No disclosure found | ⚠ not public (§12) |
| "Leading DR depositary bank" market-share claims | BNY marketing | ⚠ unverified this pass |
| "LiquidityDirect", "Elixir" and similar rumored product/AI names | Press/rumor only | ⚠ unverified this pass (§12) |
| FX pricing and trading engine launched in Singapore (press-release URL) | bny.com sitemap URL only | ⚠ existence at URL level; content not fetched |

### 11.2 The Honesty Note: Methodology

1. **Upgrades made in this pass:** the Singapore legal-entity roster moved from ⚠ (cheat-sheet open item) to ✅ via the one permitted web_extract of the bny.com Singapore location page; the §9 geography claim was sharpened — the $529m line sits in the 10-Q's foreign-covered-bonds country table, not a deposit table (§9.1). Three web searches were run (Singapore establishment year, ticker-change month, 2024 rebrand press); **all three returned empty result sets**, so no press-dated item was upgraded and those items remain ⚠ in §12.
2. **Verified anchors are the SEC record and the sitemap.** Financial, legal-entity, segment and scale claims rest on the FY2025 10-K, Q2 2026 10-Q and the July 15, 2026 earnings release; enforcement claims on SEC 2015-170 and the OFAC pages; platform claims on the locally inventoried bny.com sitemap. Nothing in this guide was fabricated; unverifiable items are marked ⚠ inline and collected in §12.
3. **The inference zone is the engine layer and the history layer.** Engine identities (core, ledgers, hubs, screening) are class-certain but identity-unknown; the 1784-era Hamilton detail, the 2007 merger narrative and the 2003 Pershing acquisition are treated as institutional history this pass did not re-verify from primary sources.
4. **Deliberately not verified (out of scope):** BNY press releases beyond the earnings release, per-market regulatory filings, vendor contracts, MAS-specific supervisory records for BNY, and internal roadmaps.
5. **Claims this guide deliberately does not make:** no AI-platform identity, no DR market-share ranking, no ticker-change month, no Pershing sale, no engine name, no Singapore establishment year — each is an ⚠ item collected in §12 rather than a guess dressed as a fact.
6. **Single-pass limitation:** every status reflects the evidence gathered in this one pass — SEC filings through the Q2 2026 reporting window, OFAC enforcement pages 2015–2025, one sitemap inventory (2,234 URLs), one page fetch, three (empty) web searches. A later pass with different sources could move items in either direction; the markers are points-in-time, and the guide says so.

---

## 12. What Could Not Be Verified

The following are the materially non-public specifics of the BNY estate and the open items this pass could not close. Each is flagged ⚠ and should be treated as **unknown, not as "likely X"**:

1. **The custody ledger / safekeeping engine identity** ⚠ — the core system that actually records $62.6T of AUC/A (Jun 30, 2026, ✅ preliminary). No name appears in the 10-K, the 10-Q, the earnings releases, or the sitemap; the class is certain (a mainframe-scale securities ledger), the identity is not.
2. **The core banking / mainframe estate** ⚠ — deposit-ledger and general-ledger platforms, the mainframe application suite, and the technology stack behind BNY Mellon, N.A.'s balance sheet. BNY publishes no core-system narrative (a verified absence, §6.3).
3. **The payments hub** ⚠ — the internal orchestration engine behind the Global Payments and Trade platform (§3), including the SWIFT gateway, ISO 20022 roadmap, and USD clearing memberships (Fedwire/CHIPS participation is structural ⚠, not documented).
4. **The screening and transaction-monitoring engines** ⚠ — the sanctions-screening and AML-monitoring systems behind §8's compliance posture. The FircoSoft-family class is documented in the [FircoSoft Guide](fircosoft_guide.md); no BNY-specific engine name is public.
5. **The exact ticker-change month (BK → BNY, H1 2026)** ⚠ — the direction is ✅ (10-K cover vs Q2 2026 release/8-K covers), but no primary source pinning the month was found in this pass.
6. **The press-reported calendar year of the "BNY" rebrand (reported as May 2024)** ⚠ — press-only; the three web searches run this pass returned empty result sets, so it was not re-verified here.
7. **The Pershing 2003 acquisition from CSFB — terms and date** ⚠ — the current legal reality (Pershing LLC a BNY subsidiary, ✅ Q2 2026 10-Q) is verified; the acquisition history is institutional lore unverified in this pass.
8. **2025–2026 press reports of a Pershing sale (press named Apex Group as a possible buyer)** ⚠ — no fetched primary source confirmed a signed transaction. The operative fact is the ✅ negative: **no sale is reflected in the Q2 2026 Form 10-Q**; Pershing LLC remains a BNY subsidiary as of June 30, 2026.
9. **"Leading DR depositary bank" market-share claims** ⚠ — BNY markets leadership in depositary receipts (§2.3); the franchise and its pages are ✅, the ranking was not verified from a primary source.
10. **"LiquidityDirect" or similar liquidity-portal product branding** ⚠ — sitemap URLs plausibly map to such an offering, but the product name was not confirmed from a fetched primary page (§3.3).
11. **Rumored AI / product names (e.g., press-reported internal GenAI "Elixir")** ⚠ — not confirmed against any primary source in this pass (§7.2); per the naming discipline of this guide they are written as unverified, not as fact.
12. **The BNY Singapore establishment year** ⚠ — "⚠ the establishment year could not be re-verified in this pass": the fetched location page (§9.2) lists offices and entities but no founding date, and the web searches returned empty. Institutional accounts commonly date the Bank of New York's Singapore branch to the late 1970s, but that is unverified here and is not asserted.
13. **The Alexander Hamilton constitution authorship and "first bank in New York" detail** ⚠ — institutional history; the 1784 lineage itself is ✅ [10-K].
14. **Per-market sub-custody network composition** ⚠ — the extent of BNY's proprietary branches versus agent-bank sub-custodians in each market is not disclosed in the sources used (§2.5).
15. **The FX pricing and trading engine launched in Singapore** ⚠ — a bny.com press-release URL in the sitemap evidences it (§9.3); its date, capabilities and current status were not fetched.
16. **Technology spend and vendor contracts** ⚠ — BNY discloses no separate technology-spend line and no named cloud or AI vendors in the sources used (§6.3, §7.2).
17. **The legal parent of the Singapore branch** ⚠ — the bny.com location page labels the vehicle "BNY, Singapore Branch" without stating which group entity (BNY Mellon, N.A. or another) operates it (§9.2).
18. **The Wealth Solutions technology stack** ⚠ — the advisory workstations, trust-accounting and portfolio-reporting systems behind §5.1; the public evidence is a single-URL wealth-technology tree.
19. **The fund-services / transfer-agency engine** ⚠ — the investor-register and order-processing system behind the fund-investor-solutions tree (§2.2).
20. **The DR depositary and corporate-trust platform identities** ⚠ — the systems that run the §2.3 depositary-receipt franchise and the trust companies' paying-agency, trustee and catastrophe-bond books (§9.3).

The pattern across all twenty items is the guide's refrain: what BNY sells is public, what BNY runs on is not. Closing any of them requires either a BNY disclosure (press release, product page, filing) or a regulatory document this pass did not hold; each is marked ⚠ and none is guessed. A future pass with access to BNY press releases, product documentation and regulatory filings could move items out of this section; this pass's budget did not allow it, and the guide says so rather than guessing.

---

## 13. Glossary

Terms marked *cross-ref* are defined in depth in the sibling guide named; this guide uses them without re-deriving them. Terms with no cross-ref are defined here at the level this guide needs.

| Term | Meaning in this guide | See |
|---|---|---|
| AUC/A | Assets under custody and/or administration — assets BNY holds and services *on behalf of other institutions*; BNY's defining metric ($62.6T at Jun 30, 2026, preliminary) | §1.6 |
| AUM | Assets under management — assets BNY manages on a discretionary basis ($2.2T) | §1.6 |
| Custody | Safekeeping of financial assets plus the operational work around them (settlement, income, corporate actions, valuation, reporting) | §2.1; *cross-ref* [Investment Portfolio Operations Guide](investment_portfolio_operations_guide.md) |
| Asset servicing | The operational servicing of client portfolios — accounting, valuation, corporate actions, tax, reporting — distinct from safekeeping alone | §2.1–§2.2 |
| Sub-custody / sub-custodian | A local bank that safekeeps and settles in its own market for a foreign bank's customers; the relationship Cymbal Bank has with BNY in the US market | §10 |
| Global custodian | A bank that safekeeps and services assets across many markets through its own branches and sub-custodians | §2.1, §10.2 |
| Depositary receipt (ADR/DR) | A US-tradable certificate representing shares of a non-US company; BNY runs DR programs (sponsored and unsponsored) | §2.3 |
| Issuer services | Corporate-trust and DR services for issuers — paying agency, bond trustee, DR program administration | §2.3 |
| Corporate trust | Trust-company services (trustee, paying agent, escrow) for debt issuers, run through BNY's trust companies | §1.7, §2.3 |
| Transfer agency / fund services | Keeping the investor register and processing subscriptions/redemptions for fund managers (BNY's fund-investor-solutions tree) | §2.2 |
| Tri-party repo | Repo where a neutral third party (at BNY, the Global Collateral Management Platform) holds and values collateral between the two counterparties | §4.3 |
| Collateral management | The valuation, allocation, substitution and optimization of collateral posted in repo and derivatives | §4.3 |
| Clearance | The post-trade process of confirming, netting and settling obligations; BNY's global-clearing-solutions tree and Clearance fee line | §4.2 |
| Clearing broker | A broker-dealer that clears and settles trades and holds customer assets for other brokers/advisors — Pershing's role | §5.2 |
| Pershing | BNY's clearing, custody and technology business for the wealth-management and broker-dealer industry; Pershing LLC is the US broker-dealer subsidiary | §5 |
| NetX | Pershing's public advisor/broker-dealer platform (account opening, trading, clearing, reporting) ⚠ content-level detail not fetched | §5.2 |
| Nostro / vostro | "Our" account on another bank's books / "your" account on our books; Cymbal's USD account at BNY is a vostro for BNY | §10.4; *cross-ref* [Payment Rails Guide](payment_rails_guide.md) |
| SWIFT | The cooperative messaging network used for cross-border payment and securities messages between banks | §10.4; *cross-ref* [Payment Rails Guide](payment_rails_guide.md) |
| ISO 20022 | The XML-based global message standard replacing legacy SWIFT formats; the securities and payments migration target | §10.4; *cross-ref* [Payment Rails Guide](payment_rails_guide.md) |
| T+1 | Trade date plus one business day — the US standard settlement cycle for securities | §10.6; *cross-ref* [Investment Portfolio Operations Guide](investment_portfolio_operations_guide.md) |
| DVP | Delivery-versus-payment — settlement in which securities and cash move simultaneously, eliminating principal risk | §10.6; *cross-ref* [Investment Portfolio Operations Guide](investment_portfolio_operations_guide.md) |
| Corporate actions | Issuer-driven events affecting held securities (dividends, mergers, tenders, splits) that custodians must capture and process | §2.2, §10.6 |
| IBOR / ABOR | Investment book of record (the front-office book) vs accounting book of record (the legal/financial book); the reconciliation between them is core custody operations | *cross-ref* [Investment Portfolio Operations Guide](investment_portfolio_operations_guide.md) |
| NAV / fund accounting | Net asset value calculation and the accounting books behind pooled funds | *cross-ref* [Investment Portfolio Operations Guide](investment_portfolio_operations_guide.md) |
| KYC | Know-your-customer — the identity and due-diligence process applied to clients and their principals | §8.1, §10.3 |
| AML | Anti-money-laundering — the compliance program around customer due diligence, monitoring and reporting | §8.1 |
| Sanctions screening | Checking names and transactions against sanctions and watch lists; the FircoSoft-family engine class | *cross-ref* [FircoSoft Guide](fircosoft_guide.md) |
| OCC | Office of the Comptroller of the Currency — primary federal regulator of national banks such as BNY Mellon, N.A. | §8.3 |
| BHC / Federal Reserve | Bank holding company — The Bank of New York Mellon Corporation's status; supervised by the Federal Reserve | §8.3 |
| Resolution plan | The "living will" a large BHC must file showing how it could be resolved in bankruptcy; no public BNY deficiency found this pass | §8.3 |
| CET1 | Common Equity Tier 1 capital ratio — 11.0% at Q2 2026 | §1.5, §8.3 |
| ROTCE | Return on tangible common equity — 31.3% at Q2 2026 | §1.5 |
| FX (embedded) | Foreign exchange executed as part of custody flows (income conversion, trade settlement), monetized inside servicing segments | §4.1 |
| Agent bank | A bank that acts for another bank in a market where the second bank has no branch — the sub-custody relationship; BNY is the agent bank in Cymbal Bank's US-market example | §10.2 |
| Safekeeping | The holding of securities in custody, with position-keeping and asset protection — the first half of BNY's "safekeeping and transaction settlement" line | §2.1–§2.2 |
| Settlement | The exchange of securities and cash completing a trade; in the US, on a T+1 cycle | §10.6; *cross-ref* [Investment Portfolio Operations Guide](investment_portfolio_operations_guide.md) |
| Paying agent | An issuer-services role: distributing interest and principal to bondholders on an issuer's behalf — run through BNY's trust companies and Issuer Services line | §2.3, §9.3 |
| Prime custody | Custody tailored to prime-brokerage and hedge-fund clients — financing-adjacent, daily NAV and margin flows (BNY's prime-custody-solutions slug) | §2.2 |
| Middle office | The post-trade operations layer between front office and back office — reconciliation, confirmation, collateral monitoring; BNY sells this as outsourced services | §2.2, §7.1 |
| Managed accounts | Separately managed investment accounts whose operations (accounting, reporting) BNY services via its managed-account-solutions platform tree | §2.2 |
| Depositary vs depository | A *depositary* (BNY sense) runs DR programs or acts as fund depositary; a *depository* is settlement infrastructure (for example DTC) — distinct meanings that are easily confused | §2.3 |
| Eagle Investment Systems | BNY's investment-operations software family (accounting, data management, performance); the guide's verified software spine via the sibling guide | §2.4; *cross-ref* [Investment Portfolio Operations Guide](investment_portfolio_operations_guide.md) §10.7 |

### 13.1 A Note on AUC/A Terminology

Assets under custody and/or administration (AUC/A) is the metric this guide leans on hardest, so its conventions deserve one note. **AUC** (assets under custody) counts assets a custodian safekeeps and settles; the **"/A"** extends the count to assets it *administers* without safekeeping — funds it values, accounts for and reports on but does not hold. BNY reports the combined **AUC/A** figure (✅ $62.6T at June 30, 2026, flagged preliminary in the earnings release itself; $59.3T audited-adjacent at December 31, 2025 from the 10-K), and its segments earn fees on both halves: safekeeping and settlement fees on the custody half, fund-accounting and administration fees on the administration half. Two cautions follow for readers comparing across this repository's bank guides: first, AUC/A is *not* AUM — BNY's AUM is only $2.2T, and conflating the two numbers is the single most common misreading of a custody bank's scale (§1.6); second, custodians define the boundary of "custody and/or administration" with some latitude, so cross-bank AUC/A comparisons are approximate unless the filer's definition is checked ⚠ (industry-standard caution, not a BNY-specific finding).

---

## 14. References and Further Reading

### 14.1 Primary Sources Verified in This Pass

- **SEC EDGAR — The Bank of New York Mellon Corporation, CIK 0001390777** (company filings browse: https://www.sec.gov/cgi-bin/browse-edgar?action=getcompany&CIK=0001390777&type=&dateb=&owner=include&count=40 ⚠ pattern-derived browse link). Documents used (local artifacts were downloaded from EDGAR; deep document links are ⚠ pattern-derived, browse by CIK and filing date to reach them):
  - **FY2025 Form 10-K** (filed 2026): "in business since 1784", segments, employees ~48,100, ticker BK as of Jan 31, 2026, entities (BNY Mellon, N.A.; trust companies), Robin Vince/Jose Minaya bios ✅.
  - **Q2 2026 Form 10-Q** (quarter ended June 30, 2026): segment description (Market and Wealth Services sub-lines), Pershing LLC custody language, Singapore foreign-covered-bonds country table ($529m), SGD hedging ✅.
  - **Q2 2026 earnings release dated July 15, 2026 and its financial supplement**: BNY brand statement, NYSE: BNY header, all Q2 2026 fee lines and results (revenue $5.7bn, net income $1,792m, EPS $2.45, CET1 11.0%, AUC/A $62.6T preliminary) ✅.
  - **Mid-2026 8-K cover exhibits** (d119098d8k.htm … d417322d8k.htm): NYSE: BNY, BNY/P, BNY PRK ticker symbols ✅.
- **SEC press release 2015-170** (August 18, 2015): https://www.sec.gov/news/press-release/2015-170 — FCPA settlement, $14.8m total ✅.
- **OFAC civil penalties and enforcement information**: https://ofac.treasury.gov/civil-penalties-and-enforcement-information — 2015 through 2025 annual enforcement pages downloaded; no BNY/Bank of New York/Mellon entries ✅ (verified negative).
- **bny.com corporate site and sitemap** (2,234 URLs inventoried locally from the corporate sitemap): platform trees used in §2–§5, §7 and §9 — https://www.bny.com/corporate/global/en/solutions/platforms/... (custody-solutions, execution-services, collateral-management, global-clearing-solutions, global-depositary-receipts, global-payments-trade, data-and-analytics, fund-investor-solutions, middle-office-solutions, liquidity-financing-solutions, conventional-trust-solutions). ⚠ Individual deep links are pattern-derived from the sitemap inventory; page copy was fetched for the Singapore location page only.
- **bny.com Singapore location page** (fetched this pass): https://www.bny.com/corporate/global/en/about-us/locations/asia-pacific/singapore.html — Singapore entity roster ✅.
- **Pershing sitemap and NetX platform URLs**: https://www.pershing.com/us/en/platforms/netx.html (+ netx-investor pages) ✅ URL-verified; ⚠ content not fetched.
- **Institutional-history sources** (⚠ secondary/not re-verified): bny.com heritage pages and newsroom URLs for the 1784/Hamilton narrative, the 2007 merger, the 2003 Pershing–CSFB acquisition, and the press-reported 2024 "BNY" rebrand.

### 14.2 Sibling Guides in This Repository (relative links, banking/)

All mechanics and definitions marked *cross-ref* in this guide live in these files:

- [Citibank Software Systems Guide](citibank_software_systems_guide.md) — the series' universal-bank entry; §11–§12 model for this guide's honesty audit; the other worked-example home of the Cymbal Bank persona.
- [Investment Portfolio Operations Guide](investment_portfolio_operations_guide.md) — custody mechanics (T+1, DVP, IBOR/ABOR, NAV/fund accounting) and the Eagle Investment Systems history (§10.7).
- [Payment Rails Guide](payment_rails_guide.md) — SWIFT, ISO 20022, nostro/vostro and US clearing-rail mechanics.
- [Market Data Consumption Guide](market_data_consumption_guide.md) — feed architectures, pricing hierarchies and reference-data management.
- [FircoSoft Guide](fircosoft_guide.md) — sanctions-screening engine class and name-matching mechanics.
- [MAS Regulations and Guidelines Guide](mas_regulations_guidelines_guide.md) — the Singapore regulatory context for §9.4.
- [Core Banking Systems Guide](core_banking_systems_guide.md) — the ledger/deposit-core vendor classes BNY's unnamed core plausibly belongs to (§6.1).

### 14.3 Verification Method and Limits

This pass ran on September 4, 2026 against a locally mirrored artifact set of the primary sources: the FY2025 Form 10-K, the Q2 2026 Form 10-Q, the Q2 2026 earnings release and financial supplement, mid-2026 8-K cover exhibits, SEC press release 2015-170, OFAC civil-penalty pages 2015–2025, the bny.com corporate sitemap (2,234 URLs) and the Pershing sitemap. Fieldwork was deliberately capped: one web_extract (the bny.com Singapore location page, which produced the §9.2 roster) and three web searches (all of which returned empty result sets, so no press-dated item was upgraded). Every ✅ in this guide traces to one of those artifacts; every ⚠ marks an inference, an un-fetched page, or an item the artifacts do not settle; ❌ was not needed because no claim in this guide is contradicted by the record used. §11.2 and §12 are the audit trail, and the inline markers throughout are the evidence map.

To re-run or extend this verification: (1) pull the EDGAR filing index for CIK 0001390777 covering the Q2 2026 reporting window and download the 10-K, 10-Q and 8-K exhibits as HTML; (2) download each OFAC year page under /civil-penalties-and-enforcement-information and search for the entity names used here; (3) crawl bny.com/sitemap.xml and the Pershing sitemap and inventory the solutions/platforms URL families; and (4) fetch individual product pages only where a one-line capability descriptor is needed, marking anything un-fetched ⚠ as this guide does. The gate discipline is the series standard: no dates, numbers or product names were invented, and every upgrade to a previously-⚠ item must be recorded in §11.

**Closing summary.** This guide has walked BNY from its 1784 founding — "in business since 1784," verified in the FY2025 10-K — through the 2007 merger that produced the legal name The Bank of New York Mellon Corporation, the September 2022 arrival of Robin Vince as CEO, and the H1 2026 completion of the migration to the single BNY brand and ticker, to a platform estate that is unusually public at the product layer (custody, payments, execution, collateral, data — all inventoried from the corporate sitemap) and unusually opaque at the engine layer (the custody ledger, core estate, payments hub and screening engines remain unnamed, collected honestly in §12). The verified numbers carry the argument: $62.6T of AUC/A against $2.2T of AUM, ~48,100 employees, three segments, an enforcement record of one 2015 FCPA matter and zero OFAC penalties in eleven years, and a Singapore presence documented in the Q2 2026 10-Q's country tables and the Marina One entity roster. BNY is the operational backbone of the institutional asset world — the infrastructure that holds, moves, values and finances other institutions' assets — which is why this series calls it, simply, the bank of banks.