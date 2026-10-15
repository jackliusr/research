# GRC in Retail Banking versus Corporate and Investment Banking: One Framework, Two Businesses — A Comparison Guide

> **Author:** Jack Liu Shurui, Solution Architect
> **Series:** Banking & Financial Technology Guides — the governance, risk and compliance cluster
> **Audience:** Solution architects, risk and compliance technologists, and governance professionals who must make one GRC framework work across a retail bank and a corporate and investment bank at the same time.

*The dedicated comparison of how the same governance, risk and compliance discipline meets two structurally different businesses — retail banking and corporate and investment banking (CIB). It does not re-explain GRC. The shared machinery — the three lines, the compliance operating model, regulatory change, control design and testing, the assurance map, issue and remediation, the KRI framework, the platform layer, board reporting, the adjacent governance domains, and individual accountability — is owned by the sibling [Governance, Risk and Compliance (GRC) in Banking: The Integrated Discipline](grc_in_banking_guide.md), and this guide cross-references it throughout. What this guide owns is exactly one subject: **where the difference between the two businesses is real, where it is only apparent, and why.***

**Series / boundary declared by name.** The risk cluster already has an integrated GRC guide, and that guide's worked example is a CIB. [Governance, Risk and Compliance (GRC) in Banking: The Integrated Discipline](grc_in_banking_guide.md) (967 lines) owns the discipline end to end: §2 the three lines as a working relationship, §3 the compliance operating model, §4 regulatory change management, §5 control design and the control framework, §6 control testing, RCSA and the assurance map, §7 the third line, §8 issue and remediation management, §9 the KRI framework, §10 the GRC platform layer, §11 board and committee reporting, §12 the adjacent governance domains, §13 individual accountability, §14 a worked example, §15 the anti-patterns, §16 the claims audit. **Critically, its §14 worked example is a CIB:** the institution it calls Cymbal Bank "operates a corporate and investment banking business across four lines: global markets, structured finance, trade finance, and corporate banking." Its §12 covers adjacent governance **domains** — data, AI, resilience, third-party — which are *domains* of the discipline, not *business lines*; the domains cut across retail and CIB alike, and the sibling guide treats them that way. So the existing guide describes GRC as practised on the investment-banking side, and the retail-versus-wholesale comparison does not exist anywhere in this repository.

That gap is not only a missing file. It is a missing *question*. Every bank of scale that runs both a retail franchise and a wholesale business runs **one** GRC framework across **two** businesses, and the most consequential design failures in that arrangement are not framework failures — they are application failures: a KRI set built for one population read against the other, a conduct programme designed for one customer type deployed at the other, a control test that works on millions of small interactions applied to a handful of enormous bespoke deals. This guide is about those failures and the design decisions that avoid them.

**What this guide does NOT do.** It does not re-derive the three lines (see [Enterprise Risk Management: The ERM Discipline](enterprise_risk_management_guide.md) §5, which owns the three-lines model itself, and [grc_in_banking_guide.md](grc_in_banking_guide.md) §2), nor the control framework, RCSA, the assurance map, the KRI *framework*, the control library, the issue lifecycle, the GRC platform layer, the adjacent governance domains, or individual accountability. Each is referenced in a clause with an explicit cross-reference and then left alone.

**Jurisdiction and sourcing constraint.** This guide describes the regulatory relationship **structurally** — who supervises what kind of business, and why the relationship differs — and prints **no** penalty amount, fine figure, conduct statistic, market-share or cost number, and no unsourced rule, deadline or threshold. A named regime, rule or authority appears below **only** where this research pass verified it at the issuing body's own material with a date; where verification was not obtained, the text describes the structure instead and does not name the regime. The verified framework anchors used are the IIA three-lines position papers (2013, 2020), COSO's internal-control and ERM guidance (1992/2013, 2004/2017), ISO 31000:2018 (published 2018-02, reviewed and confirmed 2023), and the two BCBS guidelines that define compliance risk and bank corporate governance (2005, 2015), plus BCBS 239 (January 2013) via the risk-data guide. Everything else is labelled practice description or argument. **Every bank persona below is Cymbal Bank, a fictional institution; no real bank's internal arrangements are described anywhere in this guide.**

**Cross-references used throughout:** [grc_in_banking_guide.md](grc_in_banking_guide.md) (the shared discipline), [enterprise_risk_management_guide.md](enterprise_risk_management_guide.md) (frameworks, taxonomy, appetite, and §5 the three-lines model), [risk_management_models_guide.md](risk_management_models_guide.md), [risk_data_aggregation_guide.md](risk_data_aggregation_guide.md) (BCBS 239), [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md) (the GRC-platform and compliance-systems estate), [universal_banking_model_guide.md](universal_banking_model_guide.md) (universal versus specialised banking; §3 economics and §6 the regulatory framework), [basel_regulatory_capital_guide.md](basel_regulatory_capital_guide.md), and [aml_certifications_exam_content_guide.md](aml_certifications_exam_content_guide.md) and the repo's financial-crime material.

### Reading paths

- **Solution architects and GRC technologists** — §3 (the taxonomy comparison as a data-model problem), §8 (the KRI question, the most concrete section), §10 (what the platform must represent on each side), §14 (the worked example's two answers side by side).
- **Risk and compliance professionals** — §2 (the economics that drive the governance difference), §4 (conduct, the sharpest difference), §5–§7 (the three lines, the compliance operating model, the control framework in each), §9 (the regulatory relationship).
- **Governance professionals and group-function designers** — §11 (the same job title, two jobs), §12 (where the two converge, and what that means for a group function), §13 (the failure modes), §15 (the anti-patterns checklist).
- **General readers** — §1, §3, §8, §14, §16 (the worked example, the glossary and the closing summary).

Each section stands alone; the tables are the quick reference and the cross-references carry the reader to the sibling guide that owns the underlying detail.

---

### Table of Contents

1. [Overview, Decoder, Thesis, and the Boundary Declared by Name](#1-overview-decoder-thesis-and-the-boundary-declared-by-name)
2. [The Two Businesses Structurally](#2-the-two-businesses-structurally)
3. [The Risk Taxonomy on Each Side](#3-the-risk-taxonomy-on-each-side)
4. [Conduct — The Sharpest Difference](#4-conduct--the-sharpest-difference)
5. [The Three Lines in Each](#5-the-three-lines-in-each)
6. [The Compliance Operating Model in Each](#6-the-compliance-operating-model-in-each)
7. [The Control Framework in Each](#7-the-control-framework-in-each)
8. [The KRI Question](#8-the-kri-question)
9. [The Regulatory Relationship in Each](#9-the-regulatory-relationship-in-each)
10. [The Platform and the Data in Each](#10-the-platform-and-the-data-in-each)
11. [The Organisation and the Skills in Each](#11-the-organisation-and-the-skills-in-each)
12. [Where the Two Converge](#12-where-the-two-converge)
13. [The Failure Modes](#13-the-failure-modes)
14. [The Cymbal Bank Worked Example](#14-the-cymbal-bank-worked-example)
15. [The Anti-Patterns, then the Claims Audit](#15-the-anti-patterns-then-the-claims-audit)
16. [What Could Not Be Verified, the Glossary, the Cross-References, and the Closing Summary](#16-what-could-not-be-verified-the-glossary-the-cross-references-and-the-closing-summary)

---

## 1. Overview, Decoder, Thesis, and the Boundary Declared by Name

### 1.1 The decoder: what "retail versus CIB" actually compares

The phrase "GRC in retail versus CIB" is ambiguous in a way that matters, so the decoder fixes the terms before anything else.

**Retail banking** here means a business of **many, small, homogeneous exposures sold through distribution to a mass customer base**: current accounts and deposits, cards, personal loans, mortgages, small-business lending, payments, wealth and insurance distribution. Its unit of account is the customer relationship and the transaction; its economics are scale and cost-to-serve; its risk management is heavily statistical, because with millions of exposures the law of large numbers does real work.

**Corporate and investment banking (CIB)** here means a business of **few, large, heterogeneous exposures negotiated bilaterally and booked in capital markets**: global markets (trading, sales, financing), structured finance, trade finance, and corporate/lending coverage of large firms and institutions. Its unit of account is the deal, the mandate, the trade, and the counterparty relationship; its economics are spread, fee and capital efficiency on low-volume/high-value activity; its risk management is heavily judgemental, because a portfolio of tens of large bespoke positions does not give the law of large numbers anything to work with.

**GRC** here means the integrated discipline: the three lines as a working relationship, the compliance operating model, control design and testing, the assurance map, issue and remediation, the KRI framework, the platform layer, board reporting, the adjacent governance domains, and individual accountability. It is owned by the sibling guide and not re-explained here.

So the comparison is **not** "two frameworks". It is: **one framework, applied to two business models whose risk-generating physics differ.** That framing is the whole guide.

### 1.2 The thesis

**A risk type does not change name between the two businesses; it changes character.** The categories in a bank's taxonomy — credit, operational, conduct, market, liquidity, financial crime, technology and resilience, model, third-party — are the same on both sides, and a group GRC function rightly maintains them as one taxonomy (see [enterprise_risk_management_guide.md](enterprise_risk_management_guide.md) on taxonomy). What differs is the **shape of the risk inside each category**:

- **Operational risk is a volume phenomenon in retail and a transaction phenomenon in CIB.** A retail operational risk is a control that fails on a *fraction* of millions of interactions — a percentage-point defect in a payment flow, a mis-posted fee at scale, an outage that touches a large customer cohort at once. It is detected by rates, trends and sampling and it is felt through accumulation. A CIB operational risk is a control that fails *once*, on a large bespoke deal or a single settlement — a mis-instructed payment on a nine-figure trade, a documentation defect in a structured transaction, a manual step on the only loan of its kind that quarter. It is detected by exception and judgement and it is felt through severity, not frequency.
- **Conduct risk is product and process risk in retail and suitability and market-conduct risk in CIB.** Retail conduct is about *design* — was the product designed to deliver fair value, was the disclosure clear, was the distribution channel incentivised to sell what the customer needed, was the vulnerable customer treated fairly at scale (see §4). CIB conduct is about *suitability and market integrity* — was the structure appropriate for a professional counterparty, was the information asymmetry managed, was the market treated fairly, were conflicts disclosed and managed.

Everything else in this guide develops that thesis. Where the two sides converge — and they do genuinely converge on doctrine — the guide says so plainly: the honest claim is that the **application** differs, not the **doctrine**.

### 1.3 The one-line boundary

| | Retail banking | Corporate and investment banking |
|---|---|---|
| **Unit of risk** | The customer interaction, at volume | The deal / mandate / trade, at size |
| **Risk physics** | Accumulation — control failure as a rate over a large population | Severity — control failure as an event of consequence |
| **Detection mode** | Statistics: rates, trends, sampling, complaint analytics | Judgement: exception, escalation, committee, deal review |
| **Conduct shape** | Product and process (design, disclosure, fair treatment at scale) | Suitability and market conduct (professional counterparty, integrity, conflicts) |
| **Supervisory stance** | Conduct supervision at distribution and product level, addressed to the whole customer base | Supervisory interest in footprint, licensing, perimeter and cross-border booking |
| **The GRC doctrine** | **Identical — the shared discipline.** Only the application differs | **Identical — the shared discipline.** Only the application differs |
### 1.4 The boundary of this guide, restated as a division of labour

| Subject | Owner | This guide's treatment |
|---|---|---|
| The three-lines model itself | [enterprise_risk_management_guide.md](enterprise_risk_management_guide.md) §5 | §5 covers only the *shape* difference in how the lines are populated |
| The three lines as a working relationship | [grc_in_banking_guide.md](grc_in_banking_guide.md) §2 | Cross-referenced; not re-explained |
| Compliance operating model | [grc_in_banking_guide.md](grc_in_banking_guide.md) §3 | §6 covers only the difference in emphasis and load |
| Regulatory change management | [grc_in_banking_guide.md](grc_in_banking_guide.md) §4 | Not re-explained |
| Control design and the control framework | [grc_in_banking_guide.md](grc_in_banking_guide.md) §5 | §7 covers only preventive-vs-detective emphasis and testing reach |
| Control testing, RCSA, assurance map | [grc_in_banking_guide.md](grc_in_banking_guide.md) §6 | §7 and §13 reference the doctrine; this guide adds only the volume-vs-deal testing difference |
| The third line | [grc_in_banking_guide.md](grc_in_banking_guide.md) §7 | Referenced |
| Issue and remediation management | [grc_in_banking_guide.md](grc_in_banking_guide.md) §8 | §12 notes that the lifecycle is genuinely common |
| The KRI framework | [grc_in_banking_guide.md](grc_in_banking_guide.md) §9 | §8 owns the *comparison*: why an indicator meaningful on one side misleads on the other |
| The GRC platform layer | [grc_in_banking_guide.md](grc_in_banking_guide.md) §10; [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md) | §10 covers only what the platform must *represent* differently |
| Board and committee reporting | [grc_in_banking_guide.md](grc_in_banking_guide.md) §11 | Referenced |
| Adjacent governance **domains** (data, AI, resilience, third-party) | [grc_in_banking_guide.md](grc_in_banking_guide.md) §12 | Named there as domains; they cut across both businesses |
| Individual accountability | [grc_in_banking_guide.md](grc_in_banking_guide.md) §13 | §12 notes the accountability structure is common |
| Economics of the bank model | [universal_banking_model_guide.md](universal_banking_model_guide.md) §3, §6 | §2 of this guide uses it as the driver of the governance difference |
| BCBS 239 / risk data | [risk_data_aggregation_guide.md](risk_data_aggregation_guide.md) | §10 cross-references |
| The comparison itself | **this guide** | Where the difference is real, where it is only apparent, and why |

The table is the contract between this guide and its siblings. If a reader wants the three lines *defined*, go to ERM §5. If a reader wants the three lines *run* as an operating relationship, go to the sibling's §2. If a reader wants to know *what changes when the business under those lines is a branch network rather than a trading floor*, that is this guide, §5.

### 1.5 What the reader should take away

Five claims, stated up front so they can be tested against the rest of the guide:

1. **The framework does not fork.** A group that ran two separate GRC frameworks for retail and CIB would lose the very things the framework exists to give it: comparability, one taxonomy, one accountability structure, one obligation inventory.
2. **The application must fork, in named places.** The KRI set (§8), the conduct programme (§4), the control-testing method (§7), the platform data model (§10) and the skill profile of the compliance officer (§11) are the places where one-size-fits-all reliably fails.
3. **The difference is mostly magnitude, distribution and detection mode, not category.** The taxonomy is the same; the shape inside each category differs (§3).
4. **The sharpest genuine difference is conduct** (§4) — product and process versus suitability and market conduct — because it is the one category where the *counterparty's* nature (mass-market consumer versus professional) changes the substance of the obligation, not just its measurement.
5. **A group function serving both should hold one framework with an explicit business-line dimension, not two private GRC stacks** (§12) — the same reason a bank keeps one general ledger rather than one per business line.

---

## 2. The Two Businesses Structurally

This section establishes the *cause*. Everything in §3–§11 is an effect of the economics described here. The treatment deliberately mirrors [universal_banking_model_guide.md](universal_banking_model_guide.md) §3 (economics) and §6 (regulatory framework), which own the bank-model analysis; this section extracts only the parts that drive GRC design.

### 2.1 Retail: many small homogeneous exposures

The retail bank's risk-generating machine has four characteristics that together determine its governance:

- **Exposure count is very large and each exposure is small.** Deposits, cards, loans and payments are issued and renewed in the tens of thousands to millions. No single customer outcome moves the institution; the aggregate does.
- **Exposures are homogeneous and standardised.** A mortgage is a mortgage; a card product has a template. Products are designed once, approved once, then instantiated many times. That is what makes them cheap to originate and service.
- **Funding is deposit-led and relationship-based.** Retail deposits are the bank's structural funding; the relationship is long-lived and the customer's switching cost is real, so the governance question is often about *retention and fair treatment over time*, not about a single transaction.
- **Cost-to-serve is the binding constraint.** Because margins per account are thin, the business cannot afford bespoke judgement on every interaction. It automates, it standardises, it straight-through-processes — and it accepts statistical control because per-item review is uneconomic.

The governance consequence is direct: **risk management can be statistical, and control design must be compatible with automation.** A control that requires human judgement per item is unaffordable at retail scale; a control that enforces a rule per item is both affordable and, if designed well, more reliable than a human. Retail GRC is therefore biased toward **preventive, system-enforced, product-templated** controls and toward **rate-based detection** of what slips through.

### 2.2 CIB: few large heterogeneous exposures

The wholesale and investment bank's machine has the mirror-image characteristics:

- **Exposure count is small and each exposure is large.** A structured-finance facility, a large corporate loan, a derivatives position, a trade-finance line — these are few relative to the retail book and each is material on its own.
- **Exposures are heterogeneous and bespoke.** Each large deal is negotiated. Documentation is bilateral. The structure of one transaction is not the structure of another, which is precisely the value the client pays for and precisely the reason a template control cannot cover it.
- **Revenue is capital-markets and advisory.** Spread and fee income on large, negotiated, often cross-border activity; the mix is lumpy and the economics are highly sensitive to a small number of transactions.
- **Concentration and people dependency are structural.** Exposure concentrates in a few names, sectors and jurisdictions; and the capability sits in a comparatively small number of skilled people whose departure or error is itself a risk.

The governance consequence is the mirror image: **risk management must be judgemental and case-specific, and controls must be compatible with negotiation.** A rule that says "auto-decline if X" is fine for a card; it cannot adjudicate whether a bespoke structure is appropriate for a professional counterparty. CIB GRC is therefore biased toward **detective and review-based** controls — deal committee approvals, mandatory second-line sign-off, documentation checklists, mandate and limit attestation — and toward **exception-based detection** of what those reviews miss.

### 2.3 The structural comparison

| Dimension | Retail banking | Corporate & investment banking |
|---|---|---|
| **Exposure count** | Very large (many) | Small (few) |
| **Exposure size** | Small each | Large each |
| **Homogeneity** | High — standardised products on templates | Low — bespoke, negotiated structures |
| **Revenue driver** | Net interest margin + fees, at volume | Spread, fee and advisory income, at size |
| **Funding** | Deposit-led, relationship-based | Wholesale and capital-markets funding |
| **Binding constraint** | Cost-to-serve | Capital efficiency and expertise |
| **Risk-generation physics** | Accumulation (volume) | Severity / concentration (transaction) |
| **Statistical leverage** | High — law of large numbers works | Low — tens of large positions is not a distribution worth averaging |
| **Control design bias** | Preventive, automated, templated | Detective, reviewed, bespoke |
| **Detector of a failure** | A rate that moves | An exception that is raised |
| **Where judgement lives** | In product design and change, not in per-item handling | In per-item handling, at the deal |
| **Where the money is lost** | Gradually, across a population | Suddenly, on one transaction |

### 2.4 Where this section's logic breaks down (and why we say so)

Two honest caveats, because a comparison that overstates its own case is worse than none:

- **The split is a spectrum, not a binary.** A large trading desk runs some volume processes that behave like retail (millions of market-data or settlement messages); a retail bank runs some bespoke processes that behave like CIB (a bespoke commercial restructuring, a private-banking mandate). Small-business lending sits deliberately in the middle, and the governance answer there is a hybrid, not one pole. [universal_banking_model_guide.md](universal_banking_model_guide.md) treats universal banking as a *portfolio* of businesses for exactly this reason.
- **Legal-entity structure cuts across the business split.** A bank may run retail and CIB in one entity, or in separate entities under a group, or in branches and subsidiaries across jurisdictions. That choice is itself a governance variable (§9), and it does not track neatly onto "retail vs CIB".

---

## 3. The Risk Taxonomy on Each Side

This is the analytical heart of the guide. The taxonomy is one (see [enterprise_risk_management_guide.md](enterprise_risk_management_guide.md)); the **character** of each risk inside its category changes between the businesses. Read the table first, then the prose, which does the volume-versus-transaction and product-versus-suitability work.

### 3.1 The comparison table

| Risk type | Retail character | CIB character | Same or different doctrine? |
|---|---|---|---|
| **Credit risk** | Statistical: a portfolio of many small homogeneous exposures; loss is modelled as a distribution (PD × LGD × EAD over a large book); concentration is managed by product and segment limits | Judgemental: few large heterogeneous exposures; loss is driven by single-name and sector concentration; modelled but heavily overlaid by deal-level judgement and collateral/structural analysis | Same concepts (PD/LGD/EAD, concentration, provisioning). Different **data science**: population modelling vs name-level analysis |
| **Operational risk** | Volume phenomenon: a control failing on a fraction of millions of interactions; detected by rates and trends; felt through accumulation | Transaction phenomenon: a control failing once on a large bespoke deal; detected by exception; felt through severity | Same definition, same control taxonomy. Different **detection method** — rate vs event |
| **Conduct risk** | Product and process: design, disclosure, fair treatment at scale, distribution incentives, vulnerable customers | Suitability and market conduct: appropriateness for professional counterparties, information asymmetry, market integrity, conflicts of interest | Same doctrine (treat the customer fairly / protect market integrity). Different **obligation's object** — the product at scale vs the transaction and the market |
| **Market & liquidity risk** | Minimal direct market risk (a banking book); liquidity risk is deposit-run and funding-mix risk | Core: trading-book market risk, counterparty credit risk, funding-liquidity and intraday liquidity risk across capital markets | Shared frameworks (limits, VaR/ES, LCR/NSFR in the Basel framework — see [basel_regulatory_capital_guide.md](basel_regulatory_capital_guide.md)). Radically different **materiality** |
| **Financial crime** | Volume: identity at onboarding, sanctions screening and fraud at millions-of-transactions scale; typologies skew to fraud, mule accounts, consumer scams | Relationship and transaction: complex corporate structures, trade-based typologies, correspondent and cross-border flows, PEP and sanctions complexity on large counterparties | Same three regimes (AML, sanctions, fraud). Different **typology mix** and different **screening population** |
| **Technology & resilience** | Single points of failure touch a mass customer base at once; an outage is a customer-outcome event at scale; recovery is measured against a baseline of millions of customers depending on the same system | Failures concentrate in complex, lower-volume, bespoke and often third-party-dependent platforms (trading systems, settlement, clearing); an outage is a market event with a small number of very consequential users | Same resilience doctrine (impact tolerances, important business services). Different **blast radius and dependency map** |
| **Model risk** | High-volume statistical models: credit scorecards, collections propensities, fraud models, pricing; risk is drift and fairness at population scale | Valuation, risk and capital models: derivatives pricing, XVA, bespoke credit-rating models; risk is model error on large positions and on illiquid instruments where the model *is* the mark | Same model-risk doctrine (validation, governance, limits). Different **model population** and different **failure mode** (population drift vs valuation error on a big position) |
| **Third-party & outsourcing** | Vendors of scale: payment rails, card networks, contact-centre and collections outsourcing; concentration in a handful of critical suppliers serving the whole base | Venue, clearing, data and platform dependencies; a concentration in market infrastructure and in a small number of specialist providers | Same third-party framework (see the sibling guide §12 domain treatment). Different **criticality logic** |

### 3.2 The analytical work: volume vs transaction

The table's operational-risk row is where the guide's thesis is most precise. Consider two control failures that a taxonomy would both call "operational risk — process execution":

- **Retail.** A fee is applied to the wrong segment of accounts. The control that should prevent this operates millions of times. The failure rate might be small, and yet the count of affected customers is large, and the *detection* mechanism is a **rate comparison** — a quality-assurance sample, an exception report, a complaint pattern. A single instance is invisible and irrelevant; the *proportion* is everything. The remediation is a product-template fix and a customer-remediation exercise, and it runs at population scale.
- **CIB.** A settlement instruction on a large transaction is mis-keyed. The control operated **once**. There is no rate; there is a count of one, and the count of one is the whole story. Detection comes from an **exception** — a failed settlement, a reconciliation break, a counterparty query — and the response is a bespoke investigation of one transaction. The remediation may be a one-off, and whether a *control fix* is warranted depends on judgement, not arithmetic.

The same taxonomy entry, two different failure mechanics, two different detection methods, two different remediation shapes. A GRC programme that describes "operational risk" once and applies it to both will build the wrong detector on at least one side.

### 3.3 The analytical work: product vs suitability

The conduct row is the second analytical hinge. Retail conduct attaches to the **product and the process**: a product approved once and sold many times is where the fair-value question, the disclosure question and the distribution-incentive question all live, and the customer who bears the outcome is a mass-market consumer (see §4). CIB conduct attaches to the **transaction and the market**: the specific suitability of a specific structure for a specific professional counterparty, the integrity of the market on which the transaction is done, and the conflicts the firm faces in arranging it (see §4). The doctrine — act with integrity, treat customers fairly — is the same; the *object* of the obligation is not.

### 3.4 Where the categories genuinely converge

Three categories are close enough in character between the two sides that a group framework should treat them as substantially common, and say so:

- **Financial crime** is common at the *regime* level and *screening machinery* level (customer due diligence, sanctions screening, monitoring, reporting) even though typologies and populations differ. A group AML programme is genuinely one programme with two typology libraries.
- **Third-party and outsourcing** is common in its governance logic, because the escalation of vendor concentration and resilience is a group-level problem on both sides.
- **Model risk** is common in its *discipline* — validate, govern, limit, monitor — even though the models themselves share almost nothing.

The convergence matters for §12: a group function should hold these three as genuinely common programmes with a business-line dimension, not duplicate them.

---
## 4. Conduct — The Sharpest Difference

Conduct is the sharpest difference between the two businesses, and it is the one category where the *nature of the counterparty* changes the substance of the obligation rather than only its measurement. This section treats the difference structurally. **No specific rule, deadline, threshold or supervisory expectation is named here unless it is a verified anchor with a date; the rest is structure.**

### 4.1 Retail conduct is product and process risk

Retail conduct risk attaches to four things, all of which are properties of the *product portfolio and its distribution*, not of a single transaction:

- **Product design and value.** Was the product designed to deliver fair value to the customer it is sold to? Is the fee structure comprehensible? Does the product behave as the customer would reasonably expect across its whole life — including at the points where circumstances change (arrears, redemption, renewal, death of a policyholder)? Because the product is approved once and sold many times, a design defect is a defect *at scale*: the harm is not one customer, it is a proportion of the book.
- **Disclosure and communications.** Were the terms clear, prominent and not misleading, at the point of sale and throughout the relationship? Retail disclosure is a *volume* obligation: the same communication is delivered to a population and must be fair to the least sophisticated reader, not the modal one, because the firm cannot assume the customer's expertise.
- **Fair treatment at scale, including vulnerable customers.** The obligation is to treat the *customer base* fairly, with particular attention to customers in circumstances of vulnerability (financial difficulty, health, bereavement, low financial capability). Vulnerability is a population-level concept in retail — the firm must design processes that identify and handle it systematically, because it cannot rely on per-relationship judgement at volume.
- **Distribution incentives and sales practice.** How is the product sold, by whom, and what does the incentive structure reward? The classic retail conduct failure is a design that pays the distribution channel for volume rather than for the customer's outcome, so the governance control sits partly in *remuneration and oversight of the channel* — a structural control, not a transaction control.

The consistent theme: **retail conduct is upstream.** It is governed in product approval, change management, campaign and disclosure sign-off, distribution oversight and complaint analytics — the machinery that stands between the product design and the millions of customers who will meet it.

### 4.2 CIB conduct is suitability and market conduct

CIB conduct risk attaches to the transaction and the market, across three strands:

- **Suitability and appropriateness against a professional counterparty.** The counterparty here is not a mass-market consumer but a professional, often sophisticated, client. The obligation is not "was the least sophisticated reader misled" but "was this specific structure appropriate and suitable for this specific counterparty, given its mandate, its sophistication, its objectives and its ability to bear the position". That is a **judgement applied to a bespoke object** — the opposite of the retail template.
- **Information asymmetry and conflicts of interest.** In wholesale markets the firm frequently stands between a client and a market, or acting for itself, or arranging for one client against another. Conduct risk lives in the management of the resulting information asymmetry, the Chinese walls and information barriers, the allocation of trades, the priority of client orders, and the disclosure and management of conflicts. The obligation is to *manage the conflict*, not to avoid every situation in which one exists.
- **Market integrity.** The firm is an actor *in* the market, not only a distributor *to* customers. Market-conduct risk therefore concerns the integrity of the market itself — manipulation, misinformation, abusive trading patterns, and the proper conduct of the firm's own market activity. The object of the obligation is the market, not only the client.

The consistent theme: **CIB conduct is transactional and judgemental.** It is governed in deal review and committee approval, mandate and suitability assessment, information-barrier policy and surveillance, conflicts registers (see the sibling's §12 domain treatment of the surrounding governance), and the supervisory judgement of the second line on a live transaction under time pressure (§11).

### 4.3 The comparison

| Dimension | Retail conduct | CIB conduct |
|---|---|---|
| **Object of the obligation** | The product and the process, applied to a population | The transaction and the market, applied to a counterparty |
| **Counterparty's nature** | Mass-market consumer; vulnerability is a design input | Professional counterparty; suitability is assessed per client |
| **Design question** | Was the product designed for fair value and fair treatment? | Was the structure appropriate, and were conflicts managed? |
| **Disclosure question** | Fair, clear, not misleading — to the least sophisticated reader | Adequate and timely for a professional, with conflicts disclosed |
| **Where the harm surfaces** | Distributed across a proportion of the book, over time | Concentrated in transactions and episodes in the market |
| **Detection** | Complaint analytics, product performance, QA at scale, mystery shopping, channel-vigilance data | Surveillance alerts, deal review, trade reconstruction, escalation and whistleblowing |
| **Governing artefact** | Product approval and change governance, disclosure inventory, distribution-oversight controls | Mandate and suitability records, conflicts register, information-barrier controls, surveillance rule set |
| **Tempo of a supervisory interaction** | Continuous, product- and campaign-timed | Event- and transaction-timed |
| **Dominant failure** | A fair product sold unfairly at scale, or a fair product that was never fair | A bespoke deal on which suitability or a conflict was not managed |

### 4.4 The one place the difference is *apparent*, not real

There is a real trap here, and the guide should name it. It is tempting to conclude that because the counterparties differ, the *doctrine* differs — that "treat the customer fairly" is a retail idea and "market integrity" a wholesale one. **That is the apparent, not real, difference.** The underlying doctrine is one: act with integrity, deal fairly, manage conflicts, do not mislead, protect the integrity of the market you operate in. The sibling guide's integrated discipline covers that doctrine once. What changes between retail and CIB is *how the doctrine is operationalised* — the artefact, the detector, the tempo, the object — not the doctrine.

### 4.5 The conduct bridge across the group

A group running both businesses will typically hold **one conduct-risk appetite statement** at group level (see [enterprise_risk_management_guide.md](enterprise_risk_management_guide.md) on appetite), and then **two application manuals** beneath it: the retail conduct manual (product, disclosure, distribution, vulnerability) and the CIB conduct manual (suitability, conflicts, market integrity). §14 shows what that looks like in practice, and §15's anti-patterns include the failure mode of writing one manual and applying it to both populations.

---

## 5. The Three Lines in Each

The three-lines model is owned by [enterprise_risk_management_guide.md](enterprise_risk_management_guide.md) §5, and the three lines as a working relationship — the handoffs, the shared narrative, the evidence discipline — are owned by [grc_in_banking_guide.md](grc_in_banking_guide.md) §2. **Neither is re-explained here.** This section covers only the difference in the **shape** of each line between the two businesses, because the shape determines who the first line actually is, how many people it is, and whether the second line can reach it.

### 5.1 The first line: a distributed floor versus a desk

| | Retail first line | CIB first line |
|---|---|---|
| **Who owns the risk** | Branch and contact-centre leadership, product owners, payments and operations process owners, collections and recoveries teams, mortgage and loan origination | Trading-desk heads, deal teams, relationship managers, transaction-management and trade-finance operations, capital-markets product heads |
| **Population shape** | Large and geographically distributed; many small units; high staff turnover in front-line roles | Small and concentrated; senior, specialised staff; low turnover in the core roles |
| **Where the control sits** | In a system or a standardised procedure followed by many people, plus supervisory review of front-line behaviour | In the judgement of a small number of skilled people, plus committee approval and documented review |
| **The first-line's grip on its own control** | Often weak per unit, strong in aggregate: a local manager can change behaviour but not the system; the system change is a central product-engineering act | Often strong per transaction but fragile to a single departure: the control depends on the person |
| **The evidence the first line produces** | Volume evidence: system logs, exception reports, QA samples, complaint records | Deal evidence: approvals, mandates, checklists, committee minutes, correspondence |
| **The governance risk** | The floor is too far from the control's design to be accountable for it (the sibling's §13.3 point, at retail scale) | The desk is so close to the control that independence of the second line can erode in practice |

The retail first line's structural problem is **distance**: the process owner who is accountable is often not the person who can change the control, because the control is a system feature set by a central product and engineering function. The CIB first line's structural problem is the opposite — **proximity**: the desk and the deal team may be the only people who understand the transaction, and the second line must have enough of the same sophistication to challenge them without doing their job.

### 5.2 The second line: the compliance and risk function

The second line's *charter* is the same on both sides — advise, monitor, test, report, under the BCBS guidelines of 29 April 2005 (*Compliance and the compliance function in banks*), which define compliance risk and the compliance function's mandate, independence, resources and reporting, and of 08 July 2015 (*Corporate governance principles for banks*), which place the control functions and the three lines of defence within bank governance. What differs is the **shape of the coverage problem**:

- **Retail.** A comparatively small second-line team must cover a vast, standardised, automated estate. It cannot review transactions; it must **design controls into the product and monitor the population**. Its leverage is analytics, sampling, complaint trend analysis, and product/change review. Its risk is that it becomes an oversight of *statistics* and loses contact with the actual customer experience.
- **CIB.** A second-line team with specialised expertise must cover few but complex, bespoke, fast-moving transactions. It must **be in the deal flow** — advisory at the point of structuring, review at the committee, surveillance of the market activity. Its leverage is judgement and presence. Its risk is that it becomes structurally captured by the business it advises, and that its expertise walks out of the door.

### 5.3 The third line: internal audit

Internal audit's charter is common (see [grc_in_banking_guide.md](grc_in_banking_guide.md) §7). The shape difference is in **what auditing each side requires**:

- **Retail.** Audit must be comfortable with **data-driven testing at population scale** — full-population analysis rather than sampling where the data allows, and testing of automated controls and their configuration. The audit *skill* is data and systems, and the audit *risk* is that it tests the process description rather than the real customer outcome.
- **CIB.** Audit must be comfortable with **transaction and judgement testing** — reconstructing a specific deal, assessing the quality of a committee's judgement, and auditing in a domain where the population is tiny and sampling is not the tool. The audit *skill* is market and product expertise, and the audit *risk* is that the small population means audit coverage of any one area is thin and episodic.

### 5.4 The common shape (so the guide does not over-claim)

Despite the differences above, the **structural logic is identical**: the business owns the risk, the second line assures it, the third line opines independently, and the same independence rules apply. There are not "retail three lines" and "CIB three lines"; there is one three-lines model populated differently. This is the point §12 makes at length: the convergence of doctrine is genuine, and the fork is in population, expertise and detection — not in the model.

---
## 6. The Compliance Operating Model in Each

The compliance operating model — the mandate to identify, advise, monitor, test and report, and the advisory-versus-enforcement tension within it — is owned by [grc_in_banking_guide.md](grc_in_banking_guide.md) §3 and is not re-explained here. This section covers only **where the emphasis differs** between the two businesses.

### 6.1 Retail: analytics, product gatekeeping and complaint-driven signal

The retail compliance operating model leans on four mechanisms:

- **Product and change gatekeeping.** Because the product is approved once and sold many times, the highest-leverage compliance act is *at the gate*: reviewing a new product, a pricing change, a campaign or a disclosure before it reaches the customer base. In a mature retail model this is a formal approval step with documented conditions, not an informal review.
- **Monitoring by analytics.** Retail compliance monitors the *population*: complaint volumes and themes, product performance against design intent, sales-quality and suitability metrics for distributed products, and unfair-treatment indicators. The unit of observation is a rate or a trend.
- **Complaint and remediation as a signal source.** The retail complaint is a first-class compliance input precisely because it is the one place a mass-market customer can surface what a process did to them individually. Complaint analytics is how a retail firm finds a defect that its own QA did not.
- **Distribution oversight.** Where products are sold through branches, brokers, agents or digital channels, compliance must monitor the *channel* — sales-practice reviews, incentive-design oversight, and controls over who is allowed to sell what to whom.

### 6.2 CIB: advisory depth, deal dependence and perimeter mapping

The CIB compliance operating model leans differently:

- **Advisory-heavy at the point of the deal.** The dominant activity is advising the business, in real time, on live transactions: is this structure permissible, does this client mandate support it, what must be disclosed, what conflicts exist. The value of the function is judgement under time pressure, and the model is staffed accordingly.
- **Deal-dependent monitoring.** Monitoring is built around transactions, mandates and markets — pre-trade and post-trade checks, suitability and appropriateness records, allocations, information-barrier controls, and surveillance of the firm's own market activity.
- **Licensing, registration and regulatory-perimeter load.** A wholesale business must map where it is licensed or registered to do which activities in which jurisdictions, and must answer constant questions of regulatory *perimeter* — is this activity permitted here, is this client eligible, is this booking location consistent with the licence. This load is heavy and jurisdiction-dependent on the CIB side and comparatively light on the retail side, where the question is usually "may we sell this product in this market to this type of customer", not "are we permitted to book this bespoke transaction from this entity".
- **Jurisdiction mapping as an operating model.** Because CIB activity is cross-border and booked across entities, the compliance model must maintain a live map of activity-to-entity-to-jurisdiction (see also §9), and a change in that map is a governance event.

### 6.3 The comparison

| Operating-model element | Retail emphasis | CIB emphasis |
|---|---|---|
| **Where the high-leverage act sits** | At the product/change gate, before the customer base is reached | At the deal, in real time, under time pressure |
| **Monitoring instrument** | Analytics, sampling, complaint and QA trend data | Pre/post-trade checks, suitability records, surveillance, deal review |
| **Signal that finds a defect** | A complaint or a population statistic that moves | An exception, an alert, or a challenge from a counterparty or regulator |
| **Advisory load** | Structured and scheduled (product, campaign, disclosure review) | Ad hoc and urgent (live transactions) |
| **Regulatory-perimeter load** | Moderate — product and distribution permissions | Heavy — activity/entity/jurisdiction licensing and booking questions |
| **Jurisdiction dimension** | Usually a small number of retail markets with common obligations | A live map of many jurisdictions and cross-border booking |
| **Staffing shape** | Few specialists, strong analytics and product skills | Specialists with market and product depth, present in the deal flow |
| **Native failure mode** | Compliance becomes a statistical overseer with no line of sight to the real customer outcome | Compliance becomes captured by, or dependent on, the desk it advises |

### 6.4 The shared mandate underneath

Both models run on the same professionally defined mandate: identify the compliance risk, advise the business on it, monitor the business's compliance, test the controls, and report to the board *(BCBS, Compliance and the compliance function in banks, Guidelines, 29 April 2005 — cited here as a framework-level anchor for the mandate, independence and reporting expectations; the specific five-component phrasing is the practice description in the sibling guide §3)*. As with the three lines, the mandate is one; the emphasis that implements it is two.

---

## 7. The Control Framework in Each

The control-design doctrine — preventive versus detective, manual versus automated versus IT-dependent-manual, design versus operating effectiveness, and the testing approach that follows — is owned by [grc_in_banking_guide.md](grc_in_banking_guide.md) §5 and §6 and is not re-explained here. This section covers only the difference in **which controls carry the load** on each side, and the **testing reach** that follows.

### 7.1 Retail: controls that run at volume

Retail control architecture is **preventive and system-enforced by design**, because that is the only architecture that scales economically:

- **System-enforced limits and rules on a small number of product templates.** A card limit, a payment-velocity rule, an eligibility rule, an arrears-stage rule: each is coded once and enforced on every one of millions of interactions. The control is not a person checking each case; it is the system refusing to do the non-conforming thing.
- **Straight-through processing (STP) as a control.** Where a process is fully automated with no manual touch, the *absence of manual handling* is itself a control characteristic, because manual handling is where retail process defects enter at volume.
- **Automated monitoring and post-event detection for what the preventive controls miss.** Every preventive control is imperfect; retail complements it with **exception reporting and quality assurance done by sampling or, where the data allows, over the full population**.
- **Product and change control as a control.** Because a defect propagates to the whole book, the *change* control (approval, testing, staged rollout, rollback) is a first-class preventive control in retail.
- **Customer-remediation as a control response.** When a retail defect is found, the remedy usually includes a **customer-remediation exercise** — finding everyone affected and making them whole — which is itself a controlled, population-scale process.

The testing implication: retail controls are tested by **configuration review of the automated control, plus sampling or full-population testing of the outcome**. The sibling's §6 doctrine applies unchanged; the *reach* differs — retail can often test the whole population, and where it can, sampling is a weaker substitute chosen only for cost.

### 7.2 CIB: controls that run once

CIB control architecture is **detective and review-based by design**, because the object of the control is bespoke and the population is too small to enforce a template:

- **Deal committee and approval as the control.** Significant transactions are approved by committee, with conditions recorded. The control *is* the review — a documented judgement by named people — and it is detective by nature because the deal is constructed before it is approved.
- **Mandatory second-line sign-off on defined transaction types.** Where the risk or the perimeter question is high, the second line is a required signature before execution. This is a control in the deal flow, and its strength depends entirely on the second line's expertise and independence (§5.2).
- **Pre-trade and post-trade checks.** Limit checks, eligibility checks, collateral and documentation checks around a specific trade. These can be automated for certain standard flows (see §7.3), but the bespoke remainder is reviewed.
- **Documentation and evidence discipline.** The CIB control record is the *deal file*: the mandate, the term sheet, the approvals, the suitability assessment, the conflicts assessment, the correspondence. The control is as strong as the file, and the file is bespoke.
- **Bespoke escalation.** Anything outside the standard requires escalation, and the escalation path is itself a control — but it is a path taken a small number of times, with judgement at the end of it, not a rule applied thousands of times.

The testing implication: CIB controls are tested by **re-performance of the review on sampled transactions, assessment of the quality of the committee's judgement, and reconstruction of a specific deal against its approvals**. Where the transaction population is in the tens, "sampling" and "all of them" converge, and the test is a deep examination of a small number of files rather than a statistical inference.

### 7.3 The comparison

| Control dimension | Retail | CIB |
|---|---|---|
| **Dominant control type** | Preventive, automated, system-enforced | Detective, reviewed, judgement-based |
| **Unit the control protects** | The transaction/account at volume | The deal/trade at size |
| **Where the control lives** | In code, on a product template | In a committee, a signature, a file |
| **Standard flow** | STP end-to-end where possible | Automated for high-volume standard flows (payments, confirmations); reviewed for the rest |
| **Bespoke flow** | Rare (a commercial restructure); handled by exception and escalation | The norm; handled by review and committee |
| **Testing method** | Configuration review + sampling or full-population testing of the outcome | Re-performance of the review + file reconstruction on a small population |
| **Evidence type** | System logs, exception reports, QA samples, complaint records | Approvals, minutes, mandates, suitability and conflicts records, correspondence |
| **Native blind spot** | The automated control is mis-configured and the failure is invisible because it is uniform | The reviewed control is approved by people too close to the deal to challenge it |

### 7.4 The one-line summary

**Retail controls prevent at volume and are tested statistically; CIB controls review at the deal and are tested by reconstruction.** The design-and-testing *doctrine* underneath — that design effectiveness and operating effectiveness are independent, that a control must be testable as specified, that evidence must exist — is one doctrine, owned by the sibling guide §5–§6, and it applies identically on both sides.

---
## 8. The KRI Question

This is the most concrete section in the guide, and the one most likely to change what a reader actually builds. The KRI *framework* — the characteristics of a good indicator, ownership, thresholds, the action that a breach triggers, and the pruning discipline — is owned by [grc_in_banking_guide.md](grc_in_banking_guide.md) §9. **This section owns a different question: why an indicator that is meaningful on one side of the retail/CIB split misleads on the other.** No threshold below is presented as an industry norm; every threshold is bank-, risk- and metric-specific and must be set by the bank, for its own appetite, with the sibling guide's discipline applied.

### 8.1 The mechanism: population size, denominator, lag and skew

An indicator is a *ratio* or a *count*, and both behave differently depending on the population behind them. Four properties drive every retail/CIB indicator mismatch:

- **Population size and the denominator.** A retail metric is almost always a **rate over a very large population** — complaints per thousand accounts, defects per million payment instructions, arrears-stage transition rates. The denominator is large and stable, so the rate is meaningful and small changes are interpretable. A CIB metric is often a **count over tens of transactions** — exceptions on structured deals this quarter, late approvals on trade-finance facilities. There is no stable denominator worth dividing by, and a rate computed over a tiny population is dominated by noise.
- **A single event's effect.** In retail, one additional failure vanishes: it moves a numerator of thousands by one part and the indicator does not blink. In CIB, one event *is* the indicator: it can move a count of three to four, a forty-percent change, while the underlying risk is unchanged. The same event, therefore, is invisible on one side and alarming on the other, and an indicator threshold that ignores this will produce false calm on one side and false alarm on the other.
- **Lag and the tempo of the signal.** Retail indicators are typically **lagging population statistics** — they tell you a defect existed across a period, after the fact, because that is how a rate accumulates. CIB indicators are often **contemporaneous transactional signals** — an exception raised on a live deal, a limit breach flagged at pre-trade. An indicator designed for one side's tempo will be either too stale or too jittery on the other.
- **Distribution skew and the tail.** A retail population has a long tail — a small proportion of customers, cases or complaints carries a disproportionate share of the harm or the value. A **retail average hides the tail**, so an average-based indicator can look healthy while the tail (a vulnerable-customer cohort, a single product segment, a particular branch cluster) is burning. A CIB population is so small that **there is no meaningful distribution to average**: the average of ten deals is not a description of the deals, and the "tail" is simply one of ten items. An indicator that reports a CIB average has discarded the only information that mattered — which deal.

The single most useful sentence in this section: **in retail a single event vanishes in the rate and the average hides the tail; in CIB a single event dominates the count and the "average" is noise.**

### 8.2 The retail worked indicator set

A coherent retail indicator set, with what each detects, the denominator trap, and why copying a CIB indicator across would fail.

| Indicator (illustrative construction) | What it detects | The denominator trap | Why copying a CIB indicator here fails |
|---|---|---|---|
| **Complaint rate per 1,000 accounts, by product and theme** | A product or process defect surfacing through the customer's own voice; theme clustering points at a specific control | The denominator must be the *relevant* account base, not the whole book; a whole-book denominator hides a defect in one product | A CIB "number of client escalations" has no denominator and no theme distribution — imported to retail it becomes a count of complaints that says nothing about rate or spread |
| **QA defect rate on a defined sample of a standard process** | Whether a standardised process is executing as designed | Sample size and construction must be stable, or the rate moves with the sampling rather than the process | A CIB "exceptions on reviewed deals" is a count of a bespoke review; as a retail rate it would need a sample design that does not exist |
| **Proportion of a cohort experiencing a defined harm (e.g. failed direct debit followed by a charge)** | A customer-outcome defect that is a *rate* through the customer journey | Cohort definition is the trap: the denominator must be the affected cohort, not total customers | A CIB "number of client complaints upheld" is a count; imported, it loses the cohort and the rate |
| **Tail indicator: harm concentrated in a defined vulnerable segment** | The tail the average hides — a segment or cluster doing far worse than the mean | The denominator is the segment, deliberately; the whole-base average is the trap this indicator exists to defeat | A CIB count has no distribution to slice, so a "tail" indicator is meaningless when imported |
| **Stale-signal guard: indicator computed on a lagged window with a freshness check** | Whether the population statistic has gone stale because the feed or the product changed | The trap is comparing windows of different lengths; the freshness check forces the window to be explicit | A CIB transactional signal is contemporaneous; imported as a lagging retail statistic it either double-counts or misses the live defect |

The reasons a copy-across fails, stated plainly: **the retail indicators are rates whose whole meaning depends on a large, stable denominator and a distribution that can be sliced; imported CIB counts have neither. A rate computed on a CIB-sized population is noise; a count computed on a retail-sized population is unreadable.**

### 8.3 The CIB worked indicator set

A coherent CIB indicator set on the same principles.

| Indicator (illustrative construction) | What it detects | The small-n trap | Why copying a retail indicator here fails |
|---|---|---|---|
| **Count of transactions approved with conditions outstanding past their due date** | Whether the deal-committee control is actually closing its conditions — the CIB version of "did the control run" | With tens of deals, the count is the finding; turning it into a percentage implies a denominator that is too small to stabilise it | A retail "past-due rate per 1,000" has no CIB denominator; imported, it would be a percentage of a handful — statistically meaningless |
| **Count of deals escalated outside standard mandate, with disposition** | Whether the perimeter/mandate discipline is being tested and how judgements are being made | The count must be read *with* the dispositions, not alone — the number without the judgements is uninterpretable | A retail "escalation rate" presumes volume; imported, it encourages interpreting single-digit numbers as trends |
| **Information-barrier and conflicts breaches raised, and time to resolution** | Whether the market-integrity and conflicts controls are detecting and handling issues | The count is small; trends require multiple periods, and a single breach is itself the signal, not a rate | A retail "breach rate" hides the event in a denominator; imported to CIB it would bury the one thing that mattered |
| **Surveillance alerts: dispositions and false-positive ratio, watched for pattern not rate** | Whether the surveillance rule set is calibrated and whether patterns of concern exist | The trap is reading the ratio as a rate over a small alert population; the *pattern* across alerts is the signal | A retail "alert precision" metric assumes alert volume; imported, it over-weights a change in a handful of alerts |
| **Concentration and name-level exposure movement (e.g. movement in the largest names)** | Whether concentration is building where a single name could dominate the loss distribution | The trap is averaging across names; the indicator must preserve the name, not the mean | A retail "concentration by product segment" averages across a large book; imported, it would average away the single-name risk that is the point |

The reasons a copy-across fails, stated plainly: **CIB indicators are counts and name-level facts over a small population, where the event is the information. Retail indicators are rates over a large population, where the event is noise. Importing a rate into CIB buries the signal; importing a count into retail produces an unreadable number.**

### 8.4 The bridge indicators — the few that legitimately travel

Not every indicator is side-specific. A small number are genuinely **group-level** because they measure the *machinery*, not the population:

- **Control-testing coverage and pass rate**, per business line, as a *coverage* metric — read on a business-line dimension, not pooled.
- **Issue ageing and repeat-failure counts**, which are properties of the issue lifecycle (sibling §8) and are meaningful on both sides as long as the business-line dimension is preserved.
- **Regulatory-obligation-to-control coverage**, which is a property of the obligation inventory (sibling §4).
- **Personnel and licence attestation completeness** — that the required attestations were collected, on both sides.

The rule for the bridge set: **keep the group-level indicator and always carry a business-line dimension**, so a reader can see that a group number is a blend of two populations with different physics. A group KRI reported *without* the business-line dimension is the single most common way a KRI set lies.

### 8.5 The design rule the section produces

> **Build one KRI framework (sibling §9) with two indicator sets, one per population physics. Keep a small bridge set at group level, but never report a group KRI without its business-line dimension, and never compare a rate to a count — or an average to a name.**

### 8.6 What this section does NOT do

It does **not** say a retail indicator is "better" or a CIB indicator "worse", and it does not assert that one business is harder to govern. It says the two populations have different statistical physics, that indicators are built for a population's physics, and that a set built for one physics will mislead when read against the other — which is precisely the failure mode §13(d) and the anti-pattern in §15 record. **No threshold appears in this section as a norm; thresholds are bank- and metric-specific and are set with the sibling guide's KRI discipline (owner, threshold, action, review date), not imported from this document.**

---
## 9. The Regulatory Relationship in Each

This section describes the regulatory relationship **structurally**: who supervises what kind of business and why the relationship differs between retail and CIB. It names a regime or authority only where this pass verified it against the issuing body's own material with a date. The framework-level anchors used are the two BCBS guidelines — *Compliance and the compliance function in banks* (**29 April 2005**) and *Corporate governance principles for banks* (**08 July 2015**) — which are framework-level, not jurisdiction-specific, and are cited as such. **Everything else is structural description; no specific rule, deadline, threshold or penalty figure is named.**

### 9.1 Retail: conduct supervision at distribution and product level

The retail regulatory relationship is characterised by three features:

- **Customer-behaviour supervision.** Because the retail firm's risk is product and process conduct (§4), the supervisor's interest is in how the firm's products, pricing, disclosures and distribution behave toward the whole customer base. The supervisory interaction is continuous and market-wide, addressed to the firm's entire retail book, not to individual transactions that the supervisor picks out.
- **Product and distribution as the object of supervision.** Supervisory attention lands on the product approval process, the fairness and clarity of the distribution and disclosure, the treatment of vulnerable customers, and the incentive design of the sales channel. This is a supervision of *systems and outcomes across a population*, which mirrors the retail control architecture (§7).
- **Market-wide conduct and consumer-protection expectations.** A retail supervisor will typically set or enforce expectations that apply across the whole market — so a change in expectations is felt by the firm as a *change in the terms on which it may deal with its entire retail base*, and the firm's response is a product/process programme, not a transaction response.

The structural consequence for GRC: the retail regulatory relationship makes the **product/change gate** and the **customer-outcome evidence** the central artefacts. A retail firm demonstrates compliance by showing that its products were designed fairly and that its customer outcomes are fair *in aggregate* — evidence of a population, not of a deal.

### 9.2 CIB: footprint, licensing, perimeter and cross-border booking

The CIB regulatory relationship is characterised by different features:

- **Supervisory interest in the entity and activity footprint.** A wholesale firm does business across many jurisdictions, through branches and subsidiaries, and the supervisor's interest includes the *perimeter*: which entity may do which activity where. The relationship is therefore partly a relationship about the firm's **legal-entity and booking structure**, not only about its conduct.
- **Home-host tension.** Where the firm operates through a branch or subsidiary abroad, it has a home supervisor and one or more host supervisors, whose interests can differ — the home supervisor sees the group's risk; the host supervisor sees the local entity's risk and its local-market conduct. Managing that tension is a governance matter as much as a compliance one, and it is a CIB-heavy problem because the footprint is driven by the wholesale business.
- **Cross-border booking and equivalence-type questions.** A wholesale transaction may be booked in one jurisdiction, arranged in another, and risk-managed in a third, and the firm must be confident that the booking is permitted and that its permissions travel. The compliance operating model carries a live activity/entity/jurisdiction map for this reason (§6.2).
- **Transaction- and market-level supervision.** The CIB interaction with supervisors is more transaction-, market- and event-shaped: examinations of specific activities, requests relating to specific transactions, and supervision of the firm *as a market participant* rather than only as a distributor to customers.

The structural consequence for GRC: the CIB regulatory relationship makes the **obligation inventory with a jurisdiction dimension**, the **licensing and perimeter map**, and the **transaction/deal evidence** the central artefacts. A CIB firm demonstrates compliance partly by showing that each transaction was permitted, appropriate and correctly booked in the right entity and jurisdiction.

### 9.3 The comparison

| Dimension | Retail regulatory relationship | CIB regulatory relationship |
|---|---|---|
| **Object of supervision** | Conduct at product/distribution level; customer outcomes across the base | Footprint, licensing, perimeter, transaction and market conduct |
| **Tempo** | Continuous, market-wide | Event-, transaction- and examination-shaped |
| **Unit** | The population and its outcomes | The entity, the activity, the transaction |
| **Where the firm's evidence points** | Product approval, disclosure, complaints, outcome metrics | Obligation inventory (jurisdictional), licences, deal files, booking records |
| **Home-host dimension** | Present but lighter (local retail markets with common obligations) | Central (branch/subsidiary footprint across many jurisdictions) |
| **The compliance artefact that carries the load** | The product/change gate and the outcome dataset | The perimeter/licensing map and the deal file |
| **Native failure mode** | A product or process that treats the base unfairly | An activity booked where it is not permitted, or a deal not suitable for the client |

### 9.4 What is deliberately not named here

This section does **not** name a specific conduct regime, equivalence provision, consumer-protection rule, cross-border booking rule, deadline or threshold, because none was verified at a primary source in this pass. The framework-level BCBS guidelines of 2005 and 2015 are cited because they are framework-level and were verified; anything below that level is described structurally. A reader who needs the jurisdiction-specific rules should go to the repo's jurisdiction material (see [mas_regulations_guidelines_guide.md](mas_regulations_guidelines_guide.md) and [basel_regulatory_capital_guide.md](basel_regulatory_capital_guide.md) for the capital and prudential side). The omission is deliberate and is recorded in §16.

---

## 10. The Platform and the Data in Each

The GRC platform layer — what it structurally does (registers, control library, assessment workflow, issue workflow, reporting) and the taxonomy/ownership data problem it inherits — is owned by [grc_in_banking_guide.md](grc_in_banking_guide.md) §10, and the compliance-systems estate and the GRC-platform table are owned by [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md). BCBS 239 (Principles for effective risk data aggregation and risk reporting, **January 2013**) and its risk-data implications are owned by [risk_data_aggregation_guide.md](risk_data_aggregation_guide.md). **None of that is re-explained here.** This section covers only the difference in **what the platform must represent** on each side.

### 10.1 What retail forces the platform to represent

- **Millions of low-value customer interactions.** The platform (and the systems it draws on) must be able to hold, aggregate and evidence *customer-level* facts at population scale: account-level events, transaction-level defects, product-level performance, complaint-level detail. The data model is inherently **wide**: many records, small values each.
- **Product and process versioning.** Because a retail conduct or operational defect is a property of a product/process *version*, the platform must be able to say "this control belonged to product version X, sold between dates A and B" — so that when a defect is found, the affected population can be defined precisely. **Version-effective-dated product and process metadata** is a retail-first requirement.
- **Customer-level obligations and outcomes.** The obligation inventory on the retail side maps to *obligations about the treatment of a customer class* (disclosure, fair value, vulnerability), and the evidence is customer- and outcome-level. The platform must therefore represent the customer population as the unit of the obligation.
- **Aggregation for conduct analytics.** The platform (or the analytics estate it feeds) must support the retail indicator set of §8.2 — rates over populations, tail segments, cohort definitions — which means the data model must retain enough dimensions to *slice* the population (product, segment, region, channel, vulnerability flag), not just to total it.

### 10.2 What CIB forces the platform to represent

- **Deal-level artefacts.** The platform must hold the *deal file* as a first-class object: mandates, term sheets, approvals and conditions, suitability and appropriateness assessments, conflicts assessments, correspondence. The data model is inherently **deep**: a few records, each rich and bespoke.
- **Bespoke documentation and negotiated terms.** Unlike a product template, a deal's terms are negotiated, so the platform cannot assume a fixed schema — it must accommodate structured fields for the common attributes and *document-linked* free-form artefacts for the rest.
- **Counterparty and licence records.** The platform must hold counterparty records (entity, jurisdiction, eligibility, mandate) and **licence and permission records** (which entity may do which activity where), because the CIB compliance artefacts of §9.2 point at exactly these.
- **Jurisdictional obligations.** The obligation inventory on the CIB side is inherently *multi-jurisdictional*: the same activity may attract different obligations depending on the entity and the booking location. The platform must represent obligations with a **jurisdiction and entity dimension**, not a single global list.
- **Deal-scoped evidence.** Evidence on the CIB side attaches to a specific deal or trade, so the platform's evidence model must be able to *link* an obligation to a control to a specific transaction's artefacts — the reconstruction that §7.2's testing depends on.

### 10.3 The comparison

| Platform/data requirement | Retail | CIB |
|---|---|---|
| **Unit of record** | The customer interaction/account at volume | The deal / trade / mandate at size |
| **Data model shape** | Wide — many small records | Deep — few rich records |
| **Metadata that matters most** | Product/process version, effective dates, population dimensions | Deal attributes, entity, jurisdiction, counterparty, licence |
| **Documentation** | Templated, standardised, versioned | Bespoke, negotiated, linked to the deal |
| **Obligation inventory shape** | Obligations about customer classes, mapped to product/process | Obligations with a jurisdiction and entity dimension, mapped to activities |
| **Evidence type** | Population statistics, logs, outcome datasets, complaint records | Deal files, approvals, suitability and conflicts records, booking records |
| **Aggregation need** | Rates and cohorts — needs sliceable dimensions and tails | Name-level facts — needs to preserve the individual, not average it |
| **BCBS 239 relevance** | Risk-data aggregation across a large retail book (see [risk_data_aggregation_guide.md](risk_data_aggregation_guide.md)) | Risk-data aggregation across a concentrated wholesale book and across entities/jurisdictions |
| **Native platform failure** | The model cannot define the affected population when a defect is found (no version-effective data) | The model cannot reconstruct a deal (artefacts held in the wrong system, or not held at all) |

### 10.4 The single-model conclusion

The platform question has a clean answer that §12 develops: **one platform, one taxonomy, two data models expressed as a business-line dimension and two artefact profiles — not two private GRC stacks.** The reason is the same one that makes the whole comparison tractable: the framework is one, the mechanisms (registers, control library, assessment and issue workflows, reporting) are one, and only what the model must *represent* differs. A group that runs two GRC stacks gets two obligation inventories, two control libraries and no way to compare coverage across its businesses — the exact fragmentation the sibling guide's §10 and §15 warn against. The alternative — one central model with an explicit business-line dimension — is strictly better precisely because the two businesses share a framework.

---
## 11. The Organisation and the Skills in Each

The same job title covers two different jobs. A "compliance officer" in a retail bank and a "compliance officer" in a CIB do not do the same work, and a group function that treats them as interchangeable — transferring staff without retooling, or writing one role description — will produce a function that is competent on one side and dangerous on the other. This section describes the difference **structurally**; it deliberately does not invent any figure for pay, headcount, grade or career length, because those vary by institution and market and none was verifiable.

### 11.1 The retail compliance officer's craft

The retail craft is **product, process and population**:

- **Product design review.** Reading a new product or a pricing change and asking whether it delivers fair value and treats the customer base fairly across its whole life.
- **Campaign and disclosure approval.** Reviewing mass-market communications for clarity, fairness and non-misleading effect on the least sophisticated reader.
- **Complaints and remediation.** Working the complaint-analytics signal, defining affected populations, and running or assuring customer-remediation exercises.
- **Distribution oversight.** Monitoring branches, brokers, agents and digital channels for sales-practice and incentive problems.
- **Sampling and trend analysis.** Designing and reading the QA samples and the rate-based indicators of §8.2, and knowing when a rate has moved for a real reason and when it has moved because the population or the sampling changed.

The enabling skills are **analytics and data literacy, product knowledge, and process/control design** — a compliance professional who can read a dataset and reason about a population.

### 11.2 The CIB compliance officer's craft

The CIB craft is **transaction, judgement and perimeter**:

- **Deal advice under time pressure.** Answering, on a live transaction, whether a structure is permissible, what the client's mandate supports, what must be disclosed.
- **Regulatory-perimeter questions.** Determining whether an activity is permitted in a given entity and jurisdiction, and how a cross-border transaction may be booked.
- **Licensing and registration.** Maintaining the map of what the firm is licensed or registered to do, where, through which entity.
- **Market-conduct surveillance judgement.** Reading surveillance alerts and deciding which are real, and what a pattern across alerts means.
- **Cross-border structuring.** Advising on how a transaction is arranged and booked so that it is permissible across the jurisdictions it touches.

The enabling skills are **market and product depth, cross-jurisdiction regulatory knowledge, and the confidence to challenge a senior desk under time pressure** — a compliance professional who can hold their own in a deal conversation.

### 11.3 The comparison

| Dimension | Retail compliance | CIB compliance |
|---|---|---|
| **Core object of the work** | Product, process and population | Transaction, judgement and perimeter |
| **Tempo** | Structured and scheduled (product, campaign, disclosure cycles) | Ad hoc and urgent (live deals) |
| **The craft's pivot** | Reading a population correctly | Making and defending a judgement on one object |
| **Key instrument** | Analytics, sampling, trend and complaint data | Deal review, surveillance, licensing/perimeter analysis |
| **Stakeholder style** | Many distributed process owners; a channel to oversee | A few senior desks; a relationship to hold without being captured |
| **Enabling skill** | Data literacy + product/process design | Market/product depth + cross-border regulatory knowledge |
| **Source of authority** | The rigour of the analysis and the process | The depth of the judgement and the independence to make it |
| **Failure if mis-staffed** | A former CIB officer reviews products by asking "is this deal appropriate?" and misses population harm | A former retail officer reviews deals by asking "does this product comply?" and misses transaction-specific judgement |
| **Career path (structural)** | Generally built within a business line or a centre of expertise, with moves across products and channels | Generally built from market/product experience, often with a move from the first line into compliance |
| **Enablement (structural)** | Data tooling, product-change tooling, complaint and QA systems | Deal-flow access, perimeter/licensing tools, surveillance platforms, fast advice channels |

### 11.4 Why the difference is structural, not a matter of seniority

It is tempting to treat retail and CIB compliance as the same role at different seniority, or as one role that can be reallocated when the business mix shifts. Both are wrong. The difference is in the **kind of judgement** the role makes — a population judgement versus an object judgement — and in the **kind of independence** it must hold — independence from a distributed process it does not run, versus independence from a desk it sits beside. A person strong in one kind of judgement is not automatically strong in the other, and the enablement and career structures that build each are different. This is the reasoning behind the anti-pattern in §15 ("the group function staffed to the larger business") and the failure mode in §13(e).

### 11.5 The group-staffing implication

A group compliance function serving both businesses must therefore hold **two skill pools and two enablement stacks under one leadership and one framework**. Concretely: shared standards, taxonomy and reporting; separate teams, tooling and development paths for the retail craft and the CIB craft; and a deliberate rotation or bridging policy where a person does move between them. The group function does not need two frameworks — §12 — but it does need two skill pools, and it must not let the larger business by headcount quietly set the standards for both.

---

## 12. Where the Two Converge

The honest counterpoint. This guide has spent eleven sections on difference; it would be dishonest to stop without stating clearly where the retail and CIB application are **the same thing**, because the convergence is genuine and it is the reason the split should be handled *inside* one framework rather than by building two.

### 12.1 The genuine commonalities

- **The same three lines.** The business owns the risk, the second line assures it, the third line opines independently — identically on both sides. The model is one (ERM §5; sibling §2), and neither business gets a different model.
- **The same control-design doctrine.** Preventive vs detective, manual vs automated vs IT-dependent-manual, design vs operating effectiveness, testable-as-specified: the doctrine is one (sibling §5–§6). Only which controls carry the load differs (§7).
- **The same issue lifecycle.** Raise → assess → assign → remediate → validate → close, with root cause required and independent validation before closure: one lifecycle (sibling §8). A defect on a credit-card fee and a defect on a structured deal travel the same workflow.
- **The same accountability structure.** Individual accountability requiring information, authority and reach, else it is a liability assignment: one structure (sibling §13). The CIB desk head and the retail product owner are held accountable by the same logic.
- **The same obligation-inventory discipline.** Every obligation inventoried, mapped to a control, with a named owner and a refresh cadence: one discipline (sibling §4). The retail obligation and the CIB obligation sit in the same inventory with the same attributes.
- **The same assurance-map logic.** One map of every control and who assures it, by what method, at what depth, at what frequency, to reveal overlap and gaps: one artefact (sibling §6.4). The map is where the two businesses are finally comparable.
- **The same group-function problem.** Both sides need a second line that is close enough to be useful and independent enough to challenge; both sides can suffer a captured function or a stranded control; both sides need the same independence rules.

### 12.2 What the convergence means for a group function

The convergence has a direct design consequence, and it is the guide's constructive recommendation:

| Group-function question | The wrong answer | The right answer |
|---|---|---|
| How many frameworks? | Two — one per business | **One framework.** The doctrine is genuinely common (§12.1) |
| How many taxonomies? | Two, or one that pretends the risks are identical | **One taxonomy** with a business-line dimension, so risks are comparable and can be sliced |
| How many indicator sets? | One set, copied to both sides | **Two indicator sets** under one KRI framework (§8), plus a small business-line-dimensioned bridge set |
| How many data models? | Two private GRC stacks | **One platform, one model, two artefact profiles** expressed as a business-line dimension (§10.4) |
| How many obligation inventories? | One per business, or one that ignores jurisdiction | **One inventory** with entity/jurisdiction attributes where the CIB side needs them |
| How many skill pools? | One pool expected to cover both | **Two skill pools and two enablement stacks** under one leadership (§11.5) |
| How is the group pack reported? | Pooled numbers that hide the two physics | **Business-line-dimensioned numbers**, so a group total is never mistaken for one population |

**The governing sentence: one framework with two application manuals — one taxonomy with two indicator sets, one platform with two data models, one accountability structure with two skill pools.** A single central model with an explicit business-line dimension beats two private GRC stacks because it is the only arrangement that keeps the doctrine common while letting the application differ — which is exactly the design problem the retail/CIB split poses.

### 12.3 The convergence caveat

Convergence on doctrine does not license convergence on **application artefacts**. The places where one-size-fits-all reliably fails are enumerated and are the guide's named fork points: the KRI set (§8), the conduct programme (§4), the control-testing method (§7), the platform data model (§10.4) and the compliance officer's skill profile (§11). A group that holds the doctrine common but lets those five fork is doing this correctly. A group that pools them is committing the failure modes of §13.

---
## 13. The Failure Modes

Five failure modes, each as symptom / cause / guardrail. These are the *mechanisms*; §15 restates the programme-level anti-patterns as a checklist and is distinct from this table.

| # | Failure mode | Symptom | Cause | Guardrail |
|---|---|---|---|---|
| **(a)** | **A retail control fails silently at volume** | A defect is present across a large population for a long period before anyone notices; the rate was never computed, or the aggregation hid the affected segment | The preventive automated control is mis-configured and the failure is *uniform*, so nothing looks anomalous; the detector (a rate or a sample) either does not exist or is averaged across a population that hides the tail | Build a rate-based detector for every preventive retail control (§8.2), test the automated control's configuration *and* its outcome, and add a tail/segment indicator so the average cannot hide concentrated harm |
| **(b)** | **A CIB control fails once and expensively** | A single large transaction goes wrong — a suitability gap, a documentation defect, a limit or perimeter breach on a bespoke deal — with material consequences from one event | The control is a reviewed judgement made by people close to the deal; the review was performed but the challenge was insufficient, and there was no second detector because the population is tiny | Independent second-line sign-off on defined high-risk transaction types (§7.2), documented conditions and their closure, and reconstruction testing on a small population (§7.2) rather than a design review alone |
| **(c)** | **A conduct programme designed for one side is applied to the other** | A retail-style disclosure campaign is deployed on professional counterparties (and misses suitability); or a CIB-style per-client suitability review is imposed on a mass-market product (and cannot be executed) | One conduct manual written for one counterparty type, then applied across the firm | Two conduct application manuals under one group conduct-risk appetite (§4.5): retail product/process manuals and CIB suitability/market-conduct manuals |
| **(d)** | **A KRI set is copied across the split** | False calm on one side (a retail count read as a rate) and false alarm on the other (a single CIB event read as a trend); the group pack reports a blend the reader mistakes for one population | Indicators are built for one population's statistical physics and read against the other (§8.1) | Two indicator sets under one framework (§8.2–§8.3), a small business-line-dimensioned bridge set (§8.4), and the standing rule that a group KRI is never reported without its business-line dimension |
| **(e)** | **A group compliance function is optimised for the larger business by headcount** | The larger business's standards, tooling and skill assumptions become the group standard; the smaller business is served by staff who lack its craft, and its risks are mis-measured | The function is resourced proportional to headcount/volume, so the dominant population sets the norms; the two skill pools are not maintained separately (§11.5) | Explicit two-skill-pool design under one leadership (§11.5); standards set at group level *independently* of either business's size; business-line-dimensioned reporting so the smaller business's risks stay visible |

### 13.1 The common thread

Every one of the five is the same underlying error: **applying one population's physics to another's.** (a) and (b) are that error in control testing; (c) in conduct; (d) in indicators; (e) in organisation. The guardrails are all versions of the same fix: keep the framework and doctrine common, and carry an explicit business-line dimension through every artefact where the two businesses actually differ.

---

## 14. The Cymbal Bank Worked Example

**Cymbal Bank is a fictional institution used here for illustration only.** It is the same fictional bank named in the sibling [GRC in Banking](grc_in_banking_guide.md) guide, where its CIB business was the subject of the worked review; here it is given a retail arm alongside that CIB business so the *same* governance question can be answered twice. It is fictional from end to end, the figures and artefacts are pedagogical constructions, and nothing in this section should be read as a statement about any real institution, any real vendor, or any real consultancy. **No real bank's internal arrangements are described.**

### 14.1 The scenario

Cymbal Bank runs:

- **A corporate and investment banking (CIB) business across four lines** — global markets, structured finance, trade finance, and corporate banking (the businesses the sibling guide's §14 worked example described), and
- **a retail arm** — current accounts and deposits, cards, personal loans, mortgages, payments, and a small-business lending portfolio.

It runs the shared discipline: the three lines of the sibling's §2, the compliance function of its §3, a control library (§5), an RCSA cycle and second-line testing (§6), internal audit (§7), an issue workflow (§8), a KRI framework (§9), a GRC platform (§10), and board reporting (§11).

The governance question this year is concrete: **the compliance function must produce, for the coming year, one monitoring plan and one KRI set that the board can read — and the board has asked how the function will report the two businesses without either one's risks disappearing into the other's.**

### 14.2 The retail answer

The retail compliance team proposes:

- **Monitoring plan** — weighted toward analytics and the product/change gate: quarterly population analysis of the complaint set by product and theme (the §8.2 complaint-rate indicator); a defined QA sample on the standard origination and servicing processes; a design review at every product, pricing, campaign and disclosure change; and a distribution-oversight review of the branch, broker and digital channels each period.
- **KRI set** — the rate-based and tail indicators of §8.2: complaint rate per thousand accounts by product and theme; the standard-process QA defect rate; the cohort-harm indicator; and a tail indicator for a defined vulnerable-customer segment, explicitly constructed so the population average cannot hide the segment.
- **What it is designed to catch** — a fair product sold unfairly at scale; a process defect distributed across a population; harm concentrated in a segment the average would conceal.
- **The artefact that carries the load** — the product/change gate and the customer-outcome dataset.

### 14.3 The CIB answer

The CIB compliance team proposes, for the same year:

- **Monitoring plan** — weighted toward deal review and transaction surveillance: mandatory second-line sign-off on defined high-risk transaction types across the four lines; a quarterly review of deal-committee approvals and their outstanding conditions; monitoring of the conflicts and information-barrier controls; surveillance of the firm's own market activity, read for *pattern* rather than rate; and upkeep of the licensing and perimeter map for the cross-border booking footprint.
- **KRI set** — the count-based and name-level indicators of §8.3: transactions approved with conditions outstanding past due date; deals escalated outside standard mandate with disposition; information-barrier and conflicts breaches with time to resolution; surveillance alerts read for pattern; and movement in the largest names (concentration preserved by name, not averaged).
- **What it is designed to catch** — a bespoke deal on which suitability or a conflict was not managed; a control that ran but did not challenge; an activity booked where it is not permitted.
- **The artefact that carries the load** — the deal file and the perimeter/licensing map.

### 14.4 The two answers side by side

| Element | Retail answer | CIB answer |
|---|---|---|
| **Monitoring plan centre of gravity** | Analytics + product/change gate | Deal review + transaction surveillance |
| **KRI construction** | Rates over a large population + a tail/segment indicator | Counts and name-level facts + pattern, not rate |
| **What the pack is designed to reveal** | A fair-product-sold-unfairly defect or a population-scale process defect | A single high-consequence adverse judgement or a perimeter breach |
| **Detector** | A rate that moved, or a complaint theme | An exception raised, a condition outstanding, a breach count |
| **The artefact carrying the load** | Product/change approval + outcome dataset | Deal file + licensing/perimeter map |
| **Group pack treatment** | Reported on a business-line dimension so it is never blended into the CIB numbers | Reported on a business-line dimension so it is never blended into the retail numbers |

### 14.5 THE decision the business-line difference actually changed

The two proposals were walked to the group compliance committee, and the committee made **one** decision that the business-line difference genuinely forced, and would have been made wrongly if only one business's template had been followed:

> **The KRI set would be two sets, not one — and the group pack would report each KRI with its business-line dimension, never pooled — *because the two populations have different statistical physics*.**

Concretely, the committee rejected the superficially cleaner option of "one KRI set for the whole bank", on the ground that any single set would be wrong for at least one population: a set of retail rates would bury the single high-consequence CIB event (§8.1, §8.3), and a set of CIB counts would be unreadable at retail scale and would let a distributed defect accumulate invisibly (§8.2, §13(a)). It also rejected the superficially responsive option of "two separate GRC stacks", on the ground that two stacks mean two taxonomies, two obligation inventories and no comparability — the fragmentation the sibling guide warns against (§12.2). The committee's decision was therefore the guide's constructive recommendation arrived at from the evidence: **one framework, one taxonomy, one platform with a business-line dimension, and two indicator sets and two application manuals beneath it.**

The change was not cosmetic. Before the decision, the draft group pack had a single "conduct indicator" line that blended the two businesses; the retail team's population rate and the CIB team's transaction count were being added together in the same cell. After the decision, that line was split, and the very first period revealed what the blend had hidden: the retail line showed a *rate* moving in one product theme, while the CIB line showed a *count* whose movement was a single deal. Neither fact was visible in the blended number, and neither would have survived a naive copy-across of the other side's indicator type. The business-line difference did not change the framework; it changed one reporting decision and one indicator design — and that was enough to keep both sets of risks on the board's page.

### 14.6 What the example is and is not

It is an illustration of the guide's single subject — where the difference between the two businesses is real and where it is only apparent — expressed as a worked decision. It is **not** a description of any real bank, and every artefact in it is a pedagogical construction. The mechanism it shows (a blended group indicator hiding two different population physics) is the argument of §8, not a measured finding about any institution. Its fictional bank is the same fictional bank the sibling guide uses; keeping the persona is deliberate, so the two guides' worked examples are visibly about one (fictional) institution with two businesses.

---
## 15. The Anti-Patterns, then the Claims Audit

### 15.1 The anti-patterns checklist

These are the programme-level anti-patterns — distinct from §13's *failure modes*, which describe mechanisms. Each row is a design decision a group can make wrongly, with the symptom you can observe and the guardrail that prevents it.

| # | Anti-pattern | Symptom | Guardrail |
|---|---|---|---|
| **(a)** | **One indicator set for two populations** | A group pack blends a retail rate with a CIB count; the reader mistakes the blend for one population; neither side's risks are legible | Two indicator sets under one framework (§8.2–§8.3); a small business-line-dimensioned bridge set (§8.4); never report a group KRI without its business-line dimension |
| **(b)** | **The retail control tested by sampling that never reaches the failing case** | The QA sample passes quarter after quarter while the defect persists in an unsampled segment; the control's configuration was never examined | Test the automated control's *configuration* as well as a sample of outcomes (§7.1); where the data allows, test the full population; add a tail/segment indicator so the sample cannot hide concentrated harm |
| **(c)** | **The CIB control tested by design review with no deal-level test** | The control is documented as designed and "tested", but no transaction was reconstructed; the review's quality was never actually examined | Test by re-performing the review on a small population and reconstructing a specific deal against its approvals (§7.2); assess the quality of the committee's judgement, not the existence of the minute |
| **(d)** | **The group function staffed to the larger business** | The dominant business's standards, tooling and craft become the group norm; the smaller business is served by staff without its craft and its risks are mis-measured | Two skill pools and two enablement stacks under one leadership (§11.5); standards set at group level independently of either business's size; business-line-dimensioned reporting |
| **(e)** | **The conduct policy written for one customer type** | A retail-style disclosure programme is applied to professional counterparties, or a per-client suitability process is imposed on a mass-market product | Two conduct application manuals under one group conduct-risk appetite (§4.5): retail product/process, and CIB suitability/market-conduct |
| **(f)** | **The platform that cannot express business-line context** | The register cannot say which business a risk, control or issue belongs to; coverage cannot be compared across the two; a defect's affected population cannot be defined | One platform with an explicit business-line dimension (§10.4); version-effective product/process metadata for retail; deal/entity/jurisdiction attributes for CIB |
| **(g)** | **The framework forked into two private GRC stacks** | Two taxonomies, two obligation inventories, no comparability, duplicated effort; the group cannot see its own coverage | Keep one framework and one platform (§12.2); fork only the named application artefacts — KRI set, conduct manual, testing method, data-model profiles, skill pools |
| **(h)** | **The doctrine treated as forkable** | The firm concludes that because the counterparties differ, the *doctrine* differs, and writes separate three-lines or accountability models per business | Hold the doctrine common (it is genuinely one — §12.1); fork only the application; the three lines, the issue lifecycle and the accountability structure are one model |

### 15.2 The claims audit

Every claim of consequence in this guide, with its status and source. **Kind** ∈ {Verified, Practice description, Argument, Illustrative, Cross-reference}. "Practice description" means a description of how GRC work is done that is not defined by a single authoritative standard; "argument" means a claim of reasoning rather than fact; "illustrative" belongs to the Cymbal Bank example; "cross-reference" points to the sibling guide that owns the underlying claim. Sources carry both the **date** and the **kind**.

| Claim | Status / Kind | Source, date and kind |
|---|---|---|
| The three-lines model itself (and the shape of the model) is owned by the ERM guide, not re-explained here | Cross-reference | [enterprise_risk_management_guide.md](enterprise_risk_management_guide.md) §5 — kind: Cross-reference (no date; internal cross-reference) |
| The three lines as a working relationship, the compliance operating model, regulatory change, control design, testing/RCSA/assurance map, the third line, issue and remediation, the KRI framework, the platform layer, board reporting, the adjacent governance domains, and individual accountability are owned by the sibling GRC guide | Cross-reference | [grc_in_banking_guide.md](grc_in_banking_guide.md) §§2–13 — kind: Cross-reference (no date; internal cross-reference) |
| The sibling GRC guide's §14 worked example is a CIB business (four lines: global markets, structured finance, trade finance, corporate banking) | Verified (as a statement about this repository) | [grc_in_banking_guide.md](grc_in_banking_guide.md) §14.1 — kind: Verified (repository text, checked this pass) |
| The sibling GRC guide's §12 covers adjacent governance **domains** (data, AI, resilience, third-party), not business lines | Verified (as a statement about this repository) | [grc_in_banking_guide.md](grc_in_banking_guide.md) §12 — kind: Verified (repository text, checked this pass) |
| No retail-versus-CIB GRC comparison exists elsewhere in the repository | Verified (as a statement about this repository) | Repository enumeration under `banking/` this pass — kind: Verified (no file name matches retail/CIB GRC comparison) |
| The IIA's three-lines update is a Position Paper dated 08 September 2020, renaming "Three Lines of Defense" to the Three Lines Model | Verified | IIA, *The IIA's Three Lines Model: An update of the Three Lines of Defense*, Position Paper, **08 September 2020** — kind: Verified (issuing body; carried from the repo's earlier verified-anchors pass) |
| The same IIA page records a **September 2024** update reflecting the Global Internal Audit Standards glossary; the **month** of the 2020 paper is uncertain (a sibling guide says July 2020) | Verified (2024 update) / flagged (month) | IIA content page — kind: Verified with a flagged month; see §16 |
| The original three-lines formulation is the IIA 2013 position paper *The Three Lines of Defense in Effective Risk Management and Control* | Verified | IIA, position paper, **2013** — kind: Verified (issuing body; carried from the earlier pass) |
| COSO *Internal Control — Integrated Framework* was issued **1992** and refreshed **2013** (ICIF-2013); COSO ERM comprises **2004** and **2017** | Verified | COSO guidance — kind: Verified (issuing body; carried from the earlier pass) |
| ISO 31000:2018 *Risk management — Guidelines*, Edition 2, published **2018-02**, last reviewed and confirmed **2023** | Verified | ISO 31000 page — kind: Verified (issuing body; carried from the earlier pass) |
| BCBS *Compliance and the compliance function in banks* is a Guidelines publication of **29 April 2005** defining compliance risk and the compliance function's mandate, independence, resources and reporting | Verified | BCBS, Guidelines, **29 April 2005** — kind: Verified (issuing body; carried from the earlier pass) |
| BCBS *Corporate governance principles for banks* is a Guidelines publication of **08 July 2015** covering the control functions and the three lines of defence in bank governance | Verified | BCBS, Guidelines, **08 July 2015** — kind: Verified (issuing body; carried from the earlier pass) |
| BCBS 239 (Principles for effective risk data aggregation and risk reporting) is dated **January 2013**; risk data aggregation is owned by the risk-data guide | Verified / Cross-reference | BCBS 239, **January 2013**; [risk_data_aggregation_guide.md](risk_data_aggregation_guide.md) — kind: Verified (issuing body) + Cross-reference |
| Retail banking is a business of many small homogeneous exposures with deposit funding, distribution economics, statistical risk management, standardised products and scale-driven cost-to-serve | Practice description | [universal_banking_model_guide.md](universal_banking_model_guide.md) §3, §6 — kind: Practice description (bank-model economics; no single standard) |
| CIB is a business of few large heterogeneous exposures with capital-markets and advisory revenue, bespoke structures, negotiated documentation, concentration and skilled-people dependency | Practice description | [universal_banking_model_guide.md](universal_banking_model_guide.md) §3, §6 — kind: Practice description |
| A risk type does not change name between the two businesses but changes character; operational risk is a volume phenomenon in retail and a transaction phenomenon in CIB | Argument (the guide's thesis) | Reasoning in §1, §3; kind: Argument |
| Conduct risk is product and process risk in retail and suitability/market-conduct risk in CIB | Argument | Reasoning in §1, §4; kind: Argument |
| The regulatory relationship differs structurally: customer-behaviour and product/distribution conduct supervision in retail against footprint, licensing, perimeter and cross-border booking interest in CIB | Practice description / Argument | Structural description (§9); framework anchors: BCBS 2005 and 2015 — kind: Practice description + Argument |
| Home-host tension and cross-border booking are CIB-heavy structural features | Practice description | Structural description (§9.2) — kind: Practice description (not a named rule) |
| An indicator meaningful on one side misleads on the other because of population size, denominator, event effect, lag and skew; a retail rate buries a single event and a CIB count is unreadable at retail scale | Argument (a statistical-mechanics claim) | Reasoning in §8.1; kind: Argument |
| A retail average hides the tail and a CIB small-n average is noise; CIB indicators must preserve the name, retail indicators must slice the population | Argument | §8.1, §8.2, §8.3; kind: Argument |
| KRI thresholds are bank-, risk- and metric-specific and none is presented as an industry norm | Practice description (a deliberate constraint) | Stated throughout §8; kind: Practice description |
| Preventive/system-enforced controls carry the load in retail; detective/review-based controls carry the load in CIB | Practice description | §7; consistent with the sibling guide's control doctrine ([grc_in_banking_guide.md](grc_in_banking_guide.md) §5–§6) — kind: Practice description |
| Retail controls are tested by configuration review plus sampling/full-population testing; CIB controls by re-performance and file reconstruction | Practice description | §7.1–§7.2; kind: Practice description |
| The same job title covers two different crafts (population judgement vs object judgement), requiring two skill pools and two enablement stacks | Argument / Practice description | §11; kind: Argument (the divergence) + Practice description (the skill content) |
| The doctrine genuinely converges (three lines, control design, issue lifecycle, accountability, obligation inventory, assurance map, group-function problem) and only the application forks | Argument | §12.1; kind: Argument |
| A single central GRC model with an explicit business-line dimension beats two private stacks | Argument | §12.2; kind: Argument |
| The five failure modes and eight anti-patterns are mechanisms and design decisions, not measured findings | Argument | §13, §15.1; kind: Argument |
| **No penalty amount, fine figure, conduct statistic, market-share or cost number, and no unsourced rule, deadline or threshold was printed in this guide** | Verified (a constraint about this document) | Self-audit of the file this pass; the constraint is stated in §1 and §9.4 and reiterated in §16 — kind: Verified (document constraint) |
| Cymbal Bank and all its artefacts, decisions and figures in §14 | Illustrative | Fictional worked example; no real institution, vendor or consultancy is asserted — kind: Illustrative |

### 15.3 The claims audit conclusion

The audit contains two categories of claim and no third: claims about the **shared framework**, which are cross-referenced to the sibling guide that owns them or verified against a dated issuing body; and claims about the **application difference**, which are labelled Argument or Practice description because the retail/CIB application split is not defined by any single standard the way COSO's components or ISO 31000's principles are. No claim in this guide rests on an unbounded statistic, and one row of the audit is itself a claim about the document: **that no penalty figure, conduct statistic or unsourced regime was printed, and why** — because the sourcing constraint of §1 forbids printing what was not personally verified at a primary source this pass, and none of the jurisdiction-specific rules below the framework level could be so verified (see §16).

---

## 16. What Could Not Be Verified, the Glossary, the Cross-References, and the Closing Summary

### 16.1 What Could Not Be Verified

Listed so a reader can see exactly where this guide's confidence ends.

- **Jurisdiction-specific conduct, consumer-protection, suitability, licensing and cross-border booking rules.** Named in this guide: **none.** §4 and §9 describe the *structure* of the retail conduct relationship and the CIB footprint/perimeter relationship and deliberately do not name a specific rule, regime or threshold. **Why:** this pass attempted to verify two candidate jurisdiction-level anchors by extracting the issuing body's own pages — a conduct/consumer-duty page on a national conduct regulator's site and a wholesale-markets directive page on a legislative database — and **both extractions failed** (the extraction tool returned an error for each; the hosts were unreachable or blocked to the tool). Under the sourcing constraint of §1, an unverified rule must not be printed, so the text is structural only. This is the **tool limitation** entry for this guide: `web_extract` succeeded for nothing this pass beyond what these pages returned, `web_search` was attempted in the earlier repository pass and is treated the same way, and an empty or failed result was treated as a tool limitation rather than as absence of material. A reader needing the named rules should go to the repo's jurisdiction material, principally [mas_regulations_guidelines_guide.md](mas_regulations_guidelines_guide.md) for Singapore and [basel_regulatory_capital_guide.md](basel_regulatory_capital_guide.md) for the prudential/capital side.
- **No penalty figure, conduct statistic, market-share or cost number is printed anywhere in this guide, and this is deliberate.** No fine, no enforcement outcome, no complaint statistic, no cost-of-compliance figure, no headcount or pay figure, and no KRI threshold presented as an industry norm. The reason is twofold: such figures were not personally verified at a primary source this pass, and even when attributable they would date the guide and invite the reader to treat a firm-specific number as a norm. Where a design question needs a number, the guide states that the number is bank-, risk- and metric-specific (see §8).
- **The month of the IIA's 2020 three-lines update.** The IIA's own material dates the position paper **08 September 2020**; a sibling repo guide dates the update **July 2020**. This guide states the **year (2020)** with confidence and flags the **month as uncertain**; treat "2020" as the safe form.
- **Whether the IIA 2020 paper remains the current three-lines reference.** The IIA's material carries notice of a successor statement, and of a **September 2024** update to the 2020 paper reflecting the Global Internal Audit Standards glossary. The successor statement's content and date were **not** verified this pass. This guide uses the 2020 formulation because it is the one the sibling ERM and GRC guides reference; a reader relying on the model for a live compliance purpose should check the IIA's successor statement directly.
- **The regulatory status of practice artefacts.** The KRI *sets* in §8, the monitoring plans in §14, and the bridge-indicator concept of §8.4 are **practice constructions**, not definitions from a standard. The KRI *framework* is owned by the sibling guide §9 and is itself practice-based (as that guide records). Nothing in §8 is attributed to a standard that does not contain it.
- **Whether any particular real institution runs the split this guide describes, and how.** Not asserted and not knowable from outside. Cymbal Bank is fictional; no real bank's internal GRC arrangements are described anywhere in this guide.
- **All figures, thresholds and artefacts in the Cymbal Bank example (§14).** Illustrative and fictional; none is a measurement.

### 16.2 Glossary

| Term | Meaning in this guide |
|---|---|
| **Retail banking** | Banking to a mass customer base in many small homogeneous exposures — deposits, cards, loans, mortgages, payments, distribution — managed statistically at scale |
| **Corporate and investment banking (CIB)** | Banking to large corporate and institutional clients in few large heterogeneous exposures — global markets, structured finance, trade finance, corporate banking — managed judgementally at the deal |
| **Volume phenomenon** | A risk whose behaviour is driven by accumulation across a large population (retail operational risk) |
| **Transaction phenomenon** | A risk whose behaviour is driven by severity on a single large object (CIB operational risk) |
| **Product and process conduct** | Retail conduct risk attaching to the design, disclosure, distribution and fair treatment of a product across a population |
| **Suitability and market conduct** | CIB conduct risk attaching to the appropriateness of a structure for a professional counterparty and the integrity of the market |
| **Denominator trap** | The error of constructing an indicator against the wrong base (a whole book instead of a cohort; a rate over a population too small to stabilise) |
| **Small-n trap** | The error of reading a count or rate over a very small population as though it were statistically stable |
| **Bridge indicator** | A group-level indicator that is genuinely common to both businesses (e.g. issue ageing), carried at group level with a business-line dimension |
| **Perimeter** | The boundary of what a legal entity is licensed/registered to do, and where — a CIB-heavy compliance concern |
| **Home-host tension** | The potential divergence between a group's home supervisor's view of risk and a host supervisor's view of the local entity |
| **Cross-border booking** | Bookkeeping a transaction in an entity/jurisdiction other than where it was arranged or risk-managed |
| **Business-line dimension** | The attribute in a taxonomy, indicator, or data model that records which business (retail or CIB) an item belongs to |
| **Application manual** | The business-specific implementation of the common framework (the retail manual and the CIB manual) |
| **Cymbal Bank** | The fictional institution used for the worked example in this guide and in the sibling GRC guide |

### 16.3 Cross-references

- [grc_in_banking_guide.md](grc_in_banking_guide.md) — **the shared GRC discipline this guide takes as given**: §2 the three lines as a working relationship, §3 the compliance operating model, §4 regulatory change, §5 control design, §6 testing/RCSA/assurance map, §7 the third line, §8 issue and remediation, §9 the KRI framework, §10 the platform layer, §11 board reporting, §12 adjacent governance **domains**, §13 individual accountability, §14 the CIB worked example this guide reuses, §15 anti-patterns, §16 claims audit.
- [enterprise_risk_management_guide.md](enterprise_risk_management_guide.md) — the frameworks, taxonomy, appetite, and **§5 the three-lines model itself**.
- [universal_banking_model_guide.md](universal_banking_model_guide.md) — §3 the economics and §6 the regulatory framework that drive §2 of this guide.
- [risk_data_aggregation_guide.md](risk_data_aggregation_guide.md) — BCBS 239 and the risk-data layer referenced in §10.
- [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md) — the compliance-systems estate and the GRC-platform table referenced in §10.
- [risk_management_models_guide.md](risk_management_models_guide.md) — the quantitative models behind the model-risk row of §3.
- [basel_regulatory_capital_guide.md](basel_regulatory_capital_guide.md) — the Pillar 1/2 and capital machinery behind the market/liquidity and credit rows of §3.
- [mas_regulations_guidelines_guide.md](mas_regulations_guidelines_guide.md) — the jurisdiction-specific regulatory material §9 deliberately does not duplicate.
- [aml_certifications_exam_content_guide.md](aml_certifications_exam_content_guide.md) and the repo's financial-crime material — the financial-crime depth behind §3's financial-crime row.

### 16.4 The closing summary

Retail banking and corporate and investment banking are governed by **one discipline applied to two businesses.** The doctrine does not fork: it is one three-lines model, one control-design doctrine, one issue lifecycle, one accountability structure, one obligation-inventory discipline, one assurance-map logic, and one group-function problem — which is why the honest design answer is one framework with two application manuals, one taxonomy with two indicator sets, and one platform with two data models. What genuinely differs is the **shape of the risk inside each category**: operational risk is a volume phenomenon in retail and a transaction phenomenon in CIB; conduct risk is product and process risk in retail and suitability and market-conduct risk in CIB; the detector is a rate on one side and an exception on the other; the control is preventive at volume on one side and reviewed at the deal on the other; and the indicator that is meaningful on one side is misleading on the other because the two populations have different statistical physics. The places where one-size-fits-all reliably fails are named and few — the KRI set, the conduct manual, the control-testing method, the platform data model and the compliance officer's skill profile — and the guardrail everywhere else is the same: keep the framework common, carry an explicit business-line dimension through every artefact where the two businesses actually differ, and never let either business's application template silently become the group's standard. The existing GRC guide in this repository worked the discipline on the CIB side; this guide is the missing half — the retail side, and the seam between them.

**retail banking is governed by the risk of doing something to a customer; investment banking by the risk of doing something with a counterparty.**
