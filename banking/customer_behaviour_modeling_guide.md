# Customer Behaviour Modeling: Churn, Propensity, CLV, Next-Best-Action, Segmentation, Recommendation and Uplift in Banking

**The Customer-Analytics Deep-Dive — Modeling What Customers Will Do Next — the Data Foundations (Transaction, Behavioural, Demographic, Digital-Trace, CRM, Bureau), the Verified Model-Family Canon (Survival-Analysis and Classifier Churn Models,
Direct-Marketing Response Models, the Pareto/NBD → BG/NBD + Gamma-Gamma CLV Lineage, Segmentation, Next-Best-Action Decisioning, Collaborative-Filtering Recommendation, Causal Uplift Modeling), the Engineering and Governance Overlay (Pipeline, SR 11-7
Model Risk, PDPA/GDPR), and a Cymbal Bank Churn-and-Next-Best-Action Worked Example — Verified Against Primary Sources**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
>
> **Context:** Banking Domain / Customer Analytics — the customer-behaviour modeling guide of the repository's banking cluster: the analytics discipline of predicting and influencing what customers do next — churn, propensity to buy, customer lifetime
> value (CLV), next-best-action (NBA) decisioning, segmentation, product recommendation and uplift — with a banking application lens. The guide covers the data foundations (transaction, behavioural, demographic and digital-trace data, and the derived
> analytics constructs — recency/frequency/ monetary value, tenure, engagement, product holdings), the model families one by one with the canonical academic anchors verified directly this pass (Kaplan–Meier and Cox survival analysis; the
> direct-marketing response-model tradition; the CLV lineage from Berger & Nasr 1998 through Jain & Singh 2002 and Gupta & Lehmann 2003 to Schmittlein–Morrison–Colombo's Pareto/NBD 1987 and Fader–Hardie–Lee's BG/NBD 2005; segmentation's clustering and
> latent-class canon; next-best-action decisioning, whose coinage is vendor-driven and flagged ⚠; collaborative filtering from GroupLens 1994 to matrix factorization 2009; and uplift modeling, whose contested origins — Radcliffe & Surry 1999, Lo 2002,
> Radcliffe 2007 — are presented with the evidence and flagged ⚠ where attribution is uncertain), the engineering and governance overlay (the modeling pipeline and classic ML evaluation, model risk management under SR 11-7 with the MAS overlay
> cross-referenced to the ERM sibling, and PDPA/GDPR privacy cross-referenced to the data-governance sibling), and a Cymbal Bank worked example — a churn-early-warning and next-best-action program for the retail book, from data to deployment.
> Cross-references to the private-banking/ wealth guides ([Private Banking](private_banking_guide.md), [Wealth Management](wealth_management_guide.md), [Investment Portfolio Operations](investment_portfolio_operations_guide.md)) supply the value
> context for CLV; the marketing-analytics sibling supplies the response-modeling management overlay; and the guide's verification conventions — ✅ verified, ⚠ flagged, ❌ rejected, with the residue in §12.4 "What Could Not Be Verified" — are applied
> claim by claim, so the reader always knows which facts were checked against primary sources this pass and which rest on industry convention or secondary attribution.
>
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
>
> **Primary Sources:** Crossref metadata records (api.crossref.org) for the academic canon — Schmittlein, Morrison & Colombo 1987 (*Management Science* 33(1), 1–24, DOI 10.1287/mnsc.33.1.1), Fader, Hardie & Lee 2005 (*Marketing Science* 24(2), 275–284,
> DOI 10.1287/mksc.1040.0098), Fader, Hardie & Lee 2005 (*Journal of Marketing Research* 42(4), 415–430, DOI 10.1509/jmkr.2005.42.4.415), Colombo & Jiang 1999 (*Journal of Interactive Marketing* 13(3), 2–12), Berger & Nasr 1998 (*Journal of Interactive
> Marketing* 12(1), 17–30), Jain & Singh 2002 (*Journal of Interactive Marketing* 16(2), 34–46), Gupta & Lehmann 2003 (*Journal of Interactive Marketing* 17(1), 9–24), Gupta, Lehmann & Stuart 2004 (*Journal of Marketing Research* 41(1), 7–18), Sarwar
> et al. 2001 (WWW10, 285–295, DOI 10.1145/371920.372071), Koren, Bell & Volinsky 2009 (*IEEE Computer* 42(8), 30–37, DOI 10.1109/MC.2009.263); the Resnick et al. 1994 GroupLens paper (ACM CSCW'94, 175–186, plus the authors' own page at
> presnick.people.si.umich.edu); brucehardie.com (Hardie's publication list, incl. Fader & Hardie 2001 CDNOW case in *Interfaces*); the Wikipedia articles on the Kaplan–Meier estimator (1958 JASA product-limit estimator; the Kaplan/Meier manuscripts
> combined under editor John Tukey), proportional hazards (the Cox model), collaborative filtering, customer lifetime value and uplift modelling (the last carrying the full citation chain for the uplift lineage: Radcliffe & Surry 1999, Edinburgh Credit
> Scoring and Credit Control VI; Lo 2002, ACM SIGKDD Explorations 4(2), 78–86; Radcliffe 2007, Direct Marketing Analytics Journal; Rzepakowski & Jaroszewicz 2010/2011; Kane, Lo & Zheng 2014; Lo & Pachamanova 2015); gartner.com (the Gartner "Anatomy of
> a Next Best Action for Sales" research document exists — the NBA glossary definition itself was not extractable this pass and is flagged ⚠); and the sibling guides in this repository (cross-referenced, not re-derived). Facts verified this pass are
> marked ✅ with the verification recorded in the claims audit (§12); conventions and approximations are flagged ⚠ where the evidence is thinner.
>
> **Last Updated:** September 2026
>
> **Companion guides (banking siblings, same folder — the wealth and risk clusters):** [Private Banking](private_banking_guide.md) (the HNW/UHNW segment definitions, the RM model, the relationship economics behind per-customer value — cross-ref §5) ·
> [Wealth Management](wealth_management_guide.md) (the whole wealth industry, client-segment ladder and revenue pools — cross-ref §5) · [Investment Portfolio Operations](investment_portfolio_operations_guide.md) (the investops lifecycle that generates
> the transaction/behavioural data trails — cross-ref §2) · [Market Data Consumption](market_data_consumption_guide.md) (external data feeds into the analytics estate — cross-ref §2) · [Enterprise Risk Management](enterprise_risk_management_guide.md)
> (the three-lines model and model risk under SR 11-7 — cross-ref §10, do not re-derive) · [Risk Management Models](risk_management_models_guide.md) (the model-risk-management discipline and validation conventions — cross-ref §10) · [MAS Regulations,
> Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) (the Singapore overlay and the Cymbal Bank persona conventions — cross-ref §10) · [AI/GenAI Banking Compliance](ai_genai_banking_compliance_guide.md) (the AI-risk requirements
> map for model estates — cross-ref §10)
>
> **Companion guides (management/, prefix `../management/`):** [Analytics Management](../management/analytics_management_guide.md) (the marketing-analytics management sibling — response modeling in direct marketing, experiment design, privacy
> governance — cross-ref §4, do not duplicate) · [Ancillary Revenue Products](../management/ancillary_revenue_products_guide.md) (the product/cross-sell shelf and its revenue mechanics — cross-ref §8)
>
> **Companion guides (technology/, prefix `../technology/`):** [Customer Lifetime Value Prediction](../technology/customer_lifetime_value_prediction.md) (the repository's technology-side CLV notebook — supplementary, cross-ref §5) · [CRM Data Warehouse
> Modelling](../technology/data/crm_data_warehouse_modelling.md) (the CRM/star-schema layer beneath customer analytics — cross-ref §2) · [Data Governance Guide](../technology/data_governance_guide.md) (PDPA/GDPR controls, retention, data protection —
> cross-ref §10) · [Large-Scale Web Data Acquisition](../technology/large_scale_web_data_acquisition_guide.md) (event/clickstream acquisition at scale — cross-ref §2) · [LLM Agent Use Cases in
> Finance](../technology/ai_llm/llm_agent_use_cases_finance_guide.md) (agentic marketing and customer-service overlays on the decision estate — cross-ref §10) · [Hybrid Multi-Agent Systems](../technology/ai_llm/hybrid_multi_agent_systems_guide.md) and
> [Multi-Agent Banking](../technology/ai_llm/multi_agent_banking_guide.md) (the customer-service chapters — cross-ref §10) · [MLOps Lifecycle Frameworks](../technology/mlops_lifecycle_frameworks_guide.md) (the pipeline/MLOps layer — cross-ref §10) ·
> [Responsible AI Frameworks](../technology/responsible_ai_frameworks_guide.md) (fairness and explainability for customer models — cross-ref §10)

---

**How to use this guide:** Section 1 is the overview — the short answer, the key-facts table, and how this discipline sits against its siblings (cross-referenced, not re-derived). Section 2 is the data foundations — the data types (transaction,
behavioural, demographic, digital-trace, CRM, bureau) and the classic analytics constructs (recency/frequency/monetary value, tenure, engagement, product holdings) derived from them. Sections 3–9 are the model-family deep-dives, each verified against
canonical academic or industry anchors: churn models (§3 — Kaplan–Meier and Cox survival analysis, ML classifiers, banking applications, time-based evaluation), propensity/response models (§4 — the direct-marketing response-model tradition,
cross-referencing the marketing-analytics sibling), customer lifetime value (§5 — historical and predictive approaches and the verified Pareto/NBD → BG/NBD + gamma-gamma lineage, cross-referencing the wealth guides for value context), segmentation (§6 —
RFM cells, k-means, hierarchical and latent-class approaches), next-best-action (§7 — the decisioning framing, with the contested, vendor-driven coinage flagged ⚠), recommendation models (§8 — collaborative filtering, matrix factorization, content-based
and hybrid, with the canonical references verified), and uplift modeling (§9 — the causal framing, with the contested origins presented on the evidence and flagged ⚠). Section 10 is engineering, governance and privacy — the pipeline (features, training,
validation, classic ML evaluation, not LLM evaluation), deployment, SR 11-7 model-risk management (cross-referenced to the ERM sibling), the MAS overlay, and PDPA/GDPR (cross-referenced to the data-governance sibling) — all condensed, with the deep dives
in the siblings. Section 11 is the Cymbal Bank worked example — a churn-early-warning and next-best-action program for the retail book, from data to deployment. Section 12 is the claims audit (✅/⚠/❌), with §12.4 "What Could Not Be Verified". Section 13 is
the glossary, Section 14 the cross-references and further reading, and Section 15 the closing summary. Each section is self-contained enough to read alone; the worked example (§11) is the best second read after §1 because it shows the canon assembled.

---

## Table of Contents

1. [The Overview](#1-the-overview)
2. [The Data Foundations](#2-the-data-foundations)
3. [Churn Models](#3-churn-models)
4. [Propensity and Response Models](#4-propensity-and-response-models)
5. [Customer Lifetime Value](#5-customer-lifetime-value)
6. [Segmentation](#6-segmentation)
7. [Next-Best-Action Models](#7-next-best-action-models)
8. [Recommendation Models](#8-recommendation-models)
9. [Uplift Modeling](#9-uplift-modeling)
10. [Engineering, Governance and Privacy](#10-engineering-governance-and-privacy)
11. [The Cymbal Bank Worked Example — A Churn and Next-Best-Action Program](#11-the-cymbal-bank-worked-example--a-churn-and-next-best-action-program)
12. [The Claims Audit — Verified, Flagged, Rejected](#12-the-claims-audit--verified-flagged-rejected)
13. [Glossary](#13-glossary)
14. [Cross-References and Further Reading](#14-cross-references-and-further-reading)
15. [Closing Summary](#15-closing-summary)

---

## 1. The Overview

### 1.1 The Short Answer

Customer behaviour modeling is the analytics discipline of building statistical and machine-learning models of what customers have done and will do next — churn, propensity, value, and response to an action — and of wiring those models into decisions that
change the customer journey. The model families that make up the discipline are the subject of this guide, each with its verified canon: **churn models** (survival analysis and classifiers predicting who leaves and when), **propensity/response models**
(who will respond to an offer or contact), **customer lifetime value** (what a customer is worth now and in the future), **segmentation** (who is alike and how the book decomposes), **next-best-action** (what to do with a given customer at a given
moment), **recommendation** (what product or content the customer should be offered next), and **uplift modeling** (whose behaviour an action actually changes — the incremental, causal question).

In banking the discipline sits at the junction of the customer book and the data estate. A retail or private bank runs it on the same raw materials that every other banking analytics domain uses — the transaction ledger, the product holdings, the
interaction logs — but asks a different question from credit or market risk: not "will the customer repay / how much is this position worth," but "what will this customer do next, what is the customer worth, and which action changes the customer's
behaviour?" The value is in the linkage: a churn model by itself is a report card; a churn model feeding a next-best-action engine that selects a retention offer under budget and conduct constraints is a profit center. The sibling [Analytics
Management](../management/analytics_management_guide.md) guide covers how such a modeling portfolio is run as a management discipline; this guide covers the models themselves.

The canonical academic anchors were verified directly this pass at Crossref and the primary literature, and each appears in its section with its ✅ mark: the **Kaplan–Meier product-limit estimator** (Kaplan & Meier, *Journal of the American Statistical
Association*, 1958) and the **Cox proportional-hazards model** (Cox, 1972) that anchor survival-style churn analysis; the **Pareto/NBD "counting your customers" model** (Schmittlein, Morrison & Colombo, *Management Science*, 1987) and its **BG/NBD
simplification** (Fader, Hardie & Lee, *Marketing Science*, 2005) with the **gamma-gamma** monetary-value model (lineage anchored by Colombo & Jiang's stochastic RFM model, *Journal of Interactive Marketing*, 1999); the CLV review lineage of **Berger &
Nasr (1998)**, **Jain & Singh (2002)** and **Gupta & Lehmann (2003)** (with Gupta, Lehmann & Stuart's 2004 "Valuing Customers" result that a 1% retention improvement moves firm value by roughly 5%); collaborative filtering from **Resnick et al. (1994)**
GroupLens through **Sarwar et al. (2001)** item-based filtering to **Koren, Bell & Volinsky (2009)** matrix factorization; and the uplift lineage of **Radcliffe & Surry (1999)**, **Lo (2002)** and **Radcliffe (2007)** — whose contested coinage is flagged
⚠ in §9. Two industry coinages in the guide — "next-best-action" and "uplift modeling" — are genuinely vendor- or practitioner-driven rather than academic, and the guide says so plainly rather than pretending to a clean origin story.

The boundaries of the discipline are worth stating, because they are where the banking lens changes the picture. Customer behaviour modeling is **not** credit risk modeling (it predicts customer actions, not obligor default — although the churn of a
deposit book and the prepayment of a loan book are behaviour models that sit *next to* credit models on the same data; cross-ref [Risk Management Models](risk_management_models_guide.md)). It is **not** fraud detection (fraud models score individual
transactions in milliseconds against attack patterns; behaviour models score people and journeys — though both consume the same transaction stream). And it is **not** market risk or ALM modeling (the [Treasury & ALM](treasury_alm_guide.md) and [Market
Data Consumption](market_data_consumption_guide.md) worlds model rates and the balance sheet — behaviour models consume their outputs as context features, e.g. the rate ladder that drives deposit attrition). The distinguishing feature of the discipline is
the unit of analysis — the customer relationship over time — and the decision it feeds: a contact, an offer, a service action, or an explicit decision not to act.

### 1.2 The Key-Facts Table

| Fact | Value | Status / Source |
| --- | --- | --- |
| Kaplan–Meier estimator | Non-parametric product-limit estimator of the survival function, handling right-censoring; Kaplan & Meier's separate manuscripts combined by editor John Tukey | ✅ Wikipedia (Kaplan–Meier estimator); JASA 1958 |
| Cox proportional-hazards model | Semi-parametric model λ(t\|X) = λ₀(t)·exp(Xβ); covariate effects estimated without specifying the baseline hazard | ✅ Wikipedia (Proportional hazards model); Cox 1972 |
| Pareto/NBD | Schmittlein, Morrison & Colombo, "Counting Your Customers: Who-Are They and What Will They Do Next?", *Management Science* 33(1), 1–24, 1987 — probability a customer is still active | ✅ Crossref record 10.1287/mnsc.33.1.1 |
| BG/NBD | Fader, Hardie & Lee, "'Counting Your Customers' the Easy Way: An Alternative to the Pareto/NBD Model", *Marketing Science* 24(2), 275–284, 2005 — vastly easier estimation (even in Excel); similar results to Pareto/NBD | ✅ Crossref record 10.1287/mksc.1040.0098 |
| RFM/CLV iso-value curves | Fader, Hardie & Lee, *Journal of Marketing Research* 42(4), 415–430, 2005 (DOI 10.1509/jmkr.2005.42.4.415) | ✅ Crossref DOI metadata |
| Gamma-gamma monetary-value lineage | Colombo & Jiang, "A stochastic RFM model", *Journal of Interactive Marketing* 13(3), 2–12, 1999 — gamma-distributed monetary value with gamma heterogeneity | ✅ Crossref record |
| CLV reviews | Berger & Nasr 1998, JIM 12(1), 17–30; Jain & Singh 2002, JIM 16(2), 34–46; Gupta & Lehmann 2003, JIM 17(1), 9–24 | ✅ Crossref records |
| Retention economics | Gupta, Lehmann & Stuart 2004, "Valuing Customers", JMR 41(1), 7–18 — a 1% improvement in retention improves firm value by ~5%; retention's effect ~5× the discount-rate effect | ✅ Crossref abstract (10.1509/jmkr.41.1.7.25084) |
| Collaborative filtering canon | Resnick et al. 1994 GroupLens (CSCW'94, 175–186); Sarwar et al. 2001 item-based CF (WWW10, 285–295); Koren, Bell & Volinsky 2009 matrix factorization (IEEE Computer 42(8), 30–37) | ✅ Crossref / ACM / author pages |
| Uplift lineage | Radcliffe & Surry 1999 "differential response analysis" (Edinburgh); Lo 2002 "The True Lift Model" (SIGKDD Explorations 4(2), 78–86); Radcliffe 2007 (Direct Marketing Analytics Journal) | ✅ Wikipedia uplift-modelling citation chain |
| "Uplift modeling" coinage | Practitioner coinage (Radcliffe's work and the Scientific Marketer FAQ, ~2007); earlier names: true response/lift modeling, differential response analysis | ⚠ contested, see §9.2 |
| "Next best action" coinage | Vendor/CRM-industry coinage; Gartner publishes on NBA (e.g. "Anatomy of a Next Best Action for Sales"); no single academic origin | ⚠ vendor-driven, see §7.2 |
| Model risk | US: FRB SR Letter 11-7 / OCC Bulletin 2011-12, "Supervisory Guidance on Model Risk Management" (2011) | ⚠ sibling-verified; primary page not re-extracted this pass (URL moved) |
| Cymbal Bank | The repository's bank persona — the only bank persona in this guide | Repository convention |

### 1.3 Positioning Against the Sibling Guides

Customer behaviour modeling is one of several customer-facing analytics disciplines in the repository, and it is deliberately positioned rather than duplicated:

- **[Analytics Management](../management/analytics_management_guide.md)** is the management sibling: running the analytics function, marketing analytics, experimentation, and the commercial-vs-credit analytics split. This guide covers the customer models themselves and cross-references it for the management overlay (§4 especially).
- **[Private Banking](private_banking_guide.md)**, **[Wealth Management](wealth_management_guide.md)** and **[Investment Portfolio Operations](investment_portfolio_operations_guide.md)** supply the value context: what a client relationship is worth, how segments are defined (the HNW/US$1m–30m bands), and the investops lifecycle whose records become behavioural data. CLV in this guide is cross-referenced to them rather than re-derived.
- **[CRM Data Warehouse Modelling](../technology/data/crm_data_warehouse_modelling.md)** is the data layer beneath customer analytics — the star schemas and CRM mart that this guide's §2 assumes as given.
- **[Ancillary Revenue Products](../management/ancillary_revenue_products_guide.md)** is the product/cross-sell shelf — the "next products to sell" that propensity, recommendation and NBA models feed (§8).
- **[Enterprise Risk Management](enterprise_risk_management_guide.md)** and **[Risk Management Models](risk_management_models_guide.md)** own model-risk governance (SR 11-7, validation, the model risk committee); §10 cross-references them.
- **[Data Governance Guide](../technology/data_governance_guide.md)** owns the PDPA/GDPR overlay; §10.5 cross-references it.
- **[LLM Agent Use Cases in Finance](../technology/ai_llm/llm_agent_use_cases_finance_guide.md)**, **[Hybrid Multi-Agent Systems](../technology/ai_llm/hybrid_multi_agent_systems_guide.md)** and **[Multi-Agent Banking](../technology/ai_llm/multi_agent_banking_guide.md)** cover the agentic layer that increasingly consumes model scores in customer service and marketing journeys (§10.3).

What this guide adds is the model canon itself: the seven families, their verified academic anchors, their banking applications, and their honest ⚠ flags where the industry story is looser than the textbooks admit.

---

## 2. The Data Foundations

### 2.1 The Data Types

Customer behaviour models eat data that banks already hold. The practice names below are the industry-standard categories (structural knowledge; the categories themselves are verified conventions of the CRM/analytics literature cross-referenced in
[Analytics Management](../management/analytics_management_guide.md) and [CRM Data Warehouse Modelling](../technology/data/crm_data_warehouse_modelling.md)):

- **Transaction data** — the payments and ledger records: current-account debits/credits, card transactions (with MCC/merchant category), funds transfers, standing orders, direct debits, loan and mortgage repayments. This is the highest-volume, highest-signal source for retail behaviour; in wealth it is the settlement and custody activity of the investops lifecycle (cross-ref [Investment Portfolio Operations](investment_portfolio_operations_guide.md) §2).
- **Behavioural data** — the interaction and usage records beyond money movement: channel logins (mobile app, internet banking, branch, call centre), session frequency and depth, feature usage (payments, transfers, FX, card controls), complaints, service requests. In digital products these become **clickstream/event data** — the raw event stream (page views, clicks, API calls) collected by analytics SDKs and event pipelines.
- **Demographic and CRM data** — the customer master: age band, gender, income band (often self-declared or bureau-derived), occupation, marital/family status, segment codes, onboarding date, and the full **product-holdings** list (which products, opened when, closed when, current balances/limits). CRM systems hold the interaction history: campaigns contacted, offers made, response records, contact preferences.
- **Digital-trace and alternative data** — app telemetry, web behaviour, geolocation (consented), and external digital footprints. This is the "data acquisition" world of [Large-Scale Web Data Acquisition](../technology/large_scale_web_data_acquisition_guide.md) applied to customers, and the external-feed world of [Market Data Consumption](market_data_consumption_guide.md).
- **Bureau and credit data** — external credit-bureau records (credit accounts, enquiries, delinquencies, scores) used for credit-propensity and some affordability questions; in Singapore the bureau layer sits under MAS-regulated credit-reporting rules, cross-referenced from the [MAS guide](mas_regulations_guidelines_guide.md).
- **Market and macro context** — interest-rate, rate-ladder and competitor-offer context (cross-ref [Market Data Consumption](market_data_consumption_guide.md)); time-varying context features are often the difference between a churn model that works in 2023 and one that stops working in 2025 when the rate cycle turns.

### 2.2 The Classic Analytics Constructs

The raw data is aggregated into a small set of classic customer-level constructs that reappear across every model family in this guide. The first three — the **RFM** triplet — are the oldest and most durable:

- **Recency (R)** — time since the last relevant event (last transaction, last login, last contact). In survival terms it is the customer's current age in the "active" state; the Pareto/NBD literature's key insight (Schmittlein, Morrison & Colombo 1987 ✅) is that recency and frequency together identify the probability that a customer is still active.
- **Frequency (F)** — how often the customer transacts or engages in a period. In the stochastic models of §5 it is the observed purchase/transaction count; in behavioural practice it is often bucketed (monthly active, quarterly active, dormant).
- **Monetary value (M)** — the value of the customer's activity: average transaction value, monthly spend, balance held. The gamma-gamma stream (Colombo & Jiang 1999 ✅) models average monetary value as gamma-distributed across customers.
- **Tenure** — time since onboarding or since product opening; the natural clock for cohort analysis and the survival models' time axis.
- **Engagement** — a composite of behavioural intensity: logins, feature usage, channel breadth. Banking-specific composites (e.g. "primary-bank status", "digital-active") are bank conventions, not standards ⚠.
- **Product holdings and depth** — the count and mix of products (cross-sell depth), balances, limits, and share-of-wallet proxies. Holdings breadth is a classic retention predictor: more products ⇒ more friction to leave (structural industry knowledge; the exact coefficients are per-model).
- **Constructed behavioural aggregates** — spend category mixes, cash-flow volatility, balance trajectories, and lifecycle states (new, active, dormant, at-risk, gone) — engineered features whose definitions are bank conventions.

RFM scoring itself (R/F/M each split into quantile bands, then combined into cells) is a direct-marketing convention decades old; its popularization in database-marketing practice is commonly attributed to Arthur Hughes's writings ⚠ (industry attribution
— the exact coinage could not be verified this pass; see §12.4).

The construct set in tabular form, with the model families that consume each:

| Construct | Definition | Consumed by |
| --- | --- | --- |
| Recency (R) | Time since the last relevant event (transaction, login, contact) | Churn (§3), CLV (§5), segmentation (§6) |
| Frequency (F) | Event count per period (transactions, sessions, contacts) | Churn, CLV (Pareto/NBD & BG/NBD inputs), segmentation |
| Monetary value (M) | Average spend / balance / margin per period | CLV (gamma-gamma input), value segmentation |
| Tenure | Time since onboarding or product opening | Survival models (time axis), cohort analysis |
| Engagement | Composite behavioural intensity (logins, feature use, channel breadth) | Churn, propensity, NBA treatment design |
| Product holdings | Count, mix, depth and recency of product openings/closures | Propensity, recommendation (§8), NBA |
| Lifecycle state | New / active / slipping / dormant / gone (rule-and-model assignment) | Segmentation, churn targeting, NBA |
| Trigger events | Salary credit, maturing deposit, mortgage drawdown, rate event | Propensity spikes, NBA inbound moments |

The table is deliberately short: banks that build a thousand features still find their churn and CLV models dominated by a handful of these constructs plus a few engineered deltas, which is why the canonical models of §5.3 need only (R, F, M) to work.

### 2.3 Where the Data Lives

The analytics estate that assembles these constructs is the sibling's territory, not this guide's: the CRM mart and star schemas of [CRM Data Warehouse Modelling](../technology/data/crm_data_warehouse_modelling.md), the identity-resolution and profile
layer of the customer-data platform (CDP) world, the event pipelines of [Large-Scale Web Data Acquisition](../technology/large_scale_web_data_acquisition_guide.md), and the feature store that serves training and real-time scoring alike. The
architecturally important point for the model families below is the **time discipline**: every construct must be computable *as of a point in time* (no look-ahead), which is what makes the transaction/event history a time-series first and a table second.
The worked example (§11) shows the discipline in practice.

---

## 3. Churn Models

Churn — the customer terminating, abandoning or "going dormant" on a relationship — is the best-developed customer-behaviour modeling problem in banking, and it is where survival analysis met machine learning first. Two canonical statistical anchors
underpin the "time-to-event" view of churn; both were verified this pass.

### 3.1 Survival Analysis — Kaplan–Meier and Cox

**Kaplan–Meier (1958).** The Kaplan–Meier estimator — also known as the product-limit estimator — estimates the survival function S(t) = P(τ > t) non-parametrically from lifetime data: a step function that falls at each observed event time by the ratio of
events to those still at risk, and that correctly handles **right-censoring** (customers observed for a period without the event). Kaplan and Meier each submitted similar manuscripts to the *Journal of the American Statistical Association*; editor John
Tukey persuaded them to combine the work into the single 1958 paper, which has since been cited tens of thousands of times ✅ (Wikipedia, Kaplan–Meier estimator, with the 1958 JASA paper as the primary record). In churn terms the "event" is attrition or
dormancy, censoring is "still active at the observation cut-off", and the KM curve is the standard first picture of a book's attrition: how fast do cohorts of deposit customers, cardholders or borrowers stop being active?

**Cox proportional hazards (1972).** The Cox model writes the hazard as λ(t|X) = λ₀(t)·exp(Xβ): a common baseline hazard λ₀(t) over time, with covariates acting multiplicatively on it. Cox's contribution — the landmark 1972 paper, "Regression Models and
Life-Tables," in the *Journal of the Royal Statistical Society* (Series B) — was to show the covariate effects β can be estimated without specifying the baseline hazard at all (the partial likelihood), which is what makes the model semi-parametric and
robust in practice ✅ (Wikipedia, Proportional hazards model; the Cox-model formulation verified, the 1972 paper itself structural knowledge). For churn, the Cox model gives hazard ratios per feature — "a customer whose app logins dropped to zero has 3×
the attrition hazard" — which is why banks keep it in the toolkit even when the production churn score is a gradient-boosted classifier. Proportional-hazards diagnostics (the assumption that covariate effects are constant multiples over time) matter; when
they fail, analysts fall back to stratified models, time-varying coefficients, or accelerated-failure-time alternatives (structural knowledge).

Reading the survival picture in a banking book: take the cohort of current accounts opened in January 2023. The Kaplan–Meier curve for "primary-account active" falls fastest in the first 90 days (accounts opened for a bonus and never funded, or funded and
drained), then settles into a slower, roughly constant attrition; right-censoring is everywhere because most of the cohort is still active at the cut-off, and the estimator uses their censored observation times correctly rather than dropping them. Fitting
a Cox model on the same cohort yields the feature story — tenure at the bank, salary-credit regularity, app-login frequency and product depth with negative coefficients (protective), balance volatility and complaint count with positive ones — and the
hazard ratios become the explainability file that model validation and the business both read. When the proportional-hazards assumption visibly fails (the effect of a rate change on attrition decays over months), the analyst moves to time-varying
covariates or stratified baselines rather than forcing the simple model (structural practice).

### 3.2 ML Classifiers for Churn

The operational churn model in most banks is a supervised classifier on a binary or multi-class target — "attrites within the next 90 days", "becomes dormant within 180 days", "repays the loan early (runoff)": **logistic regression** remains the
interpretable baseline and the model risk management's favourite (coefficients, standard errors, monotonicity checks); **random forests** capture interactions and non-linearities; **gradient-boosted trees** (XGBoost/LightGBM/CatBoost families) are the
default high-performance choice on tabular transaction data; and survival-style losses on boosted trees give calibrated time-to-event scores. The engineering craft is feature construction from §2.2 — recency/frequency/tenure/engagement deltas ("balance
fell 40% over 8 weeks", "logins stopped 21 days ago") — plus the **observation/performance window discipline**: features are computed as of an observation date; the target is read from a fixed forward window; the model is trained on many observation dates
(structural ML practice, detailed in §10).

Class imbalance is the first engineering fact of churn modeling: in a healthy book only 1–3% of customers churn in any quarter, so a naive accuracy-maximising model predicts "no churn" for everyone and looks ~97% right. The standard handling (structural
practice) is to downsample or weight the training data while leaving the validation and test windows at natural prevalence — never rebalance the split you evaluate on — and then to calibrate final probabilities back to the natural base rate so that score
bands mean what they say ("top band = 8% realised attrition"). Decision metrics are precision-at-k and lift rather than accuracy, because every "at risk" flag triggers a retention action with real cost (§3.4).

### 3.3 Banking Applications

- **Deposit attrition** — the demand-deposit or current-account customer who closes or empties the account (the funding-book cost is visible in [Treasury & ALM](treasury_alm_guide.md) terms). Rate-driven attrition spikes when the competitive rate ladder moves; time-varying context features matter.
- **Card dormancy** — the cardholder who stops transacting: not a closed relationship but a decaying one; card portfolios are managed to activity thresholds, and dormancy models feed "reactivation" campaigns.
- **Loan runoff** — the borrower who repays early or refinances away: a prepayment/runoff model on the mortgage and personal-loan books (economically the mirror image of credit risk's default models — cross-ref [Risk Management Models](risk_management_models_guide.md)).
- **Wealth-relationship attrition** — the private-banking analogue: AUM migration out, RM-change sensitivity, the "assets leaving but account staying" state (value context in [Private Banking](private_banking_guide.md)).

Churn definitions themselves are bank conventions ⚠: "churned" can mean account closed, balance below a floor, zero transactions for N months, or AUM out-flow above a threshold — and the definition, not the model, usually drives the economics.

The definition choice is where the economics live, so the worked example (§11) treats it explicitly. A stylised map of the common definitions:

| Churn definition | Typical horizon | What it costs the bank | Typical trigger for the model |
| --- | --- | --- | --- |
| Account closed | 30–90 days | Full relationship loss (deposits, salary credits, future cross-sell) | Closure request, balance → 0 |
| Balance emptied / below floor | 90 days | Funding-book loss (ALM cost) | Balance trajectory, outbound transfers |
| Zero transactions (dormancy) | 90–180 days | Decaying revenue, reactivation cost later | Transaction recency, login recency |
| Card inactive | 90–180 days | Interchange and interest loss | Transaction recency by MCC |
| Loan repaid early (runoff) | 6–24 months | Net-interest-margin loss, reinvestment risk | Prepayment behaviour, refi signals |

The horizons in the table are illustrative bank conventions ⚠, not standards — but the pattern is universal: the definition determines both the label the model learns and the action economics that justify the retention spend.

### 3.4 Evaluation — Time-Based AUC and Lift

Churn classifiers are evaluated the way all customer models in this guide are: on **time-based** (not random) splits — train on observations from months 1–18, validate on months 19–24, test on months 25–30 — because customer behaviour drifts and a model
tuned on a random split overstates real performance. Metrics: **AUC** (rank quality), **top-decile lift** (how many true churners are captured in the top-score decile vs random), **calibration** (predicted probability vs realised attrition rate per score
band), and the **confusion-matrix economics** — precision matters because every "at risk" flag triggers a retention action with cost. Survival evaluation uses the concordance index (C-index), the survival analogue of AUC (structural practice). The
claims-audit table (§12) records which canonical anchors above were verified this pass.

The two model families in one view — the survival models and the classifiers are complements, not substitutes:

| Family | Representative models | Output | Handles censoring | Bank role |
| --- | --- | --- | --- | --- |
| Survival analysis | Kaplan–Meier; Cox PH | Survival function; hazard ratios | ✅ natively | Attrition picture, explainability, validation artefact |
| ML classifier | Logistic; random forest; GBM | P(churn within window) | ⚠ only via windowed labels | Production targeting and ranking |
| Survival-style ML | Survival losses on boosted trees | Time-to-event distribution per customer | ✅ natively | Calibrated early-warning scores |

Whichever family produces the score, the churn model's product is a ranked list of customers with a horizon and a reason — and the reason (hazard ratio, Shapley attribution, or rule) is what lets retention design an offer rather than a plea. The economics
of acting on that list — tiered retention budgets set from CLV, uplift-gated offers — are built out in §5.4, §9 and the worked example (§11).

---

## 4. Propensity and Response Models

### 4.1 The Direct-Marketing Response Tradition

Propensity models answer "how likely is this customer to do X" — buy a product, take up an offer, click, respond — and they descend directly from **response modeling in direct marketing**, the oldest commercial application of predictive modeling on
customers. The classic setup is a binary classifier trained on past campaigns: the target is "responded to the campaign" (purchased, applied, redeemed), the features are the RFM and demographic constructs of §2, and the output is a score that ranks the
audience so the marketer contacts the top N that the budget allows. The management context — campaign economics, test/control discipline, contact strategy — is the [Analytics Management](../management/analytics_management_guide.md) sibling's territory
(its direct-marketing and experimentation content); this section is the condensed model view, and the sibling's response-modeling chapters are not duplicated here.

Two subtleties distinguish good response modeling from naive classification (structural practice): **selection bias** — the historical data only contains people who were contacted, and contact was itself targeted, so the model learns
response-given-targeting, not response per se; and **the budget constraint** — the model is only as good as the rank ordering at the cut-off the budget allows, which is why evaluation is by gains/lift curves rather than accuracy.

A third, banking-specific subtlety is the **channel response curve**: response decays monotonically down the ranked list, but the shape differs by channel — push and eDM show a steep early drop and a long thin tail, while outbound calls (expensive,
intrusive) flatten slower because the conversation itself sells. Because the curve's shape decides how many deciles are worth contacting in each channel, response models are usually estimated per channel and per product family, then re-read through the
profit table of §4.3 rather than through a single global accuracy number (structural practice).

### 4.2 Banking Uses — Propensity to Buy, Take Up, or Act

- **Product propensity** — probability of buying a specific product in a window: credit card, personal loan, mortgage, investment account, insurance (the cross-sell shelf is [Ancillary Revenue Products](../management/ancillary_revenue_products_guide.md)'s subject). The banking twist is that propensity is usually conditional on an event — salary credit, a maturing fixed deposit, a new mortgage — so "trigger propensity" models score the book for both standing propensity and event-driven spikes.
- **Take-up and activation** — given an offer or a newly opened product, will the customer activate (card activation, deposit funding, digital-banking opt-in)?
- **Contact-response propensity** — will the customer respond to an outbound contact at all (the direct-marketing question, now channel-specific: push, SMS, eDM, call)?
- **Cross-sell sequencing** — which product next (the §8 recommendation overlap; propensity models are product-specific, recommender models are cross-product).

The four uses in one view, with the decision each one feeds:

| Propensity type | Target window | Decisive features | Decision it feeds |
| --- | --- | --- | --- |
| Product propensity | 30–90 days | Trigger events, holdings gaps, bureau signals | Cross-sell offer selection |
| Take-up / activation | 7–30 days post-offer | Offer channel, product, prior response history | Offer execution and follow-up |
| Contact response | Days | Recency, channel preference, contact history | Channel routing, contact caps |
| Trigger propensity | Event-driven spikes | Salary credit, maturing deposit, mortgage drawdown | NBA inbound moments (§7, §11.4) |

### 4.3 Evaluation — Gains and Lift

The canonical evaluation artifacts are the **gains chart** (cumulative responders captured as you descend the score-ranked list) and the **lift curve** (gains ÷ the random baseline): a model whose top decile captures 35% of responders has a top-decile
lift of 3.5×. Expected-profit curves add the offer cost and response value per decile and pick the optimal cut-off — the direct-marketing "profit table". These evaluation habits carry over verbatim to churn targeting and NBA offer selection in the worked
example (§11).

Response rates in banking campaigns run at 0.5–5%, so the class-imbalance discipline of §3.2 applies with extra force, and so does the campaign-cycle discipline: a response model is a perishable asset, retrained on each campaign wave's fresh test/control
data, with the previous champion retained as the benchmark challenger. Response models that are re-estimated with the newest responses and validated on the newest holdout behave noticeably better than models retrained on an annual cycle (structural
practice, and the reason the test/control discipline of the analytics sibling is a prerequisite for §4 rather than a nice-to-have).

---

A worked profit-table sketch makes the economics concrete (figures illustrative). Suppose a personal-loan offer campaign: contact cost per customer S$2.50, average margin on a taken-up loan S$180, and the response model ranks 100,000 eligible customers.
If the top decile shows an 8% take-up against a 1.2% average, the top decile alone delivers 8,000 loans × S$180 ≈ S$1.44m of margin against S$25k of contact cost — while the bottom decile at ~0.1% take-up would lose money on contact cost alone. The profit
table (cumulative margin minus cumulative cost per decile) selects the cut-off: contact the deciles where incremental margin clears contact cost, hold the rest. This is the exact arithmetic the NBA engine of §7 repeats per action, per customer, in real
time.

The offer calendar turns that one-off arithmetic into a recurring engine: campaigns are scheduled waves over the year, each wave with its own response model, profit table and contact budget, and each constrained by the cumulative contact-frequency caps
that protect the customer relationship (and the do-not-contact flags of §10.5). A propensity score that is not attached to a calendar slot, a channel and a budget is an unused number; the management of the calendar is the analytics sibling's
campaign-planning discipline, and the worked example (§11) shows the calendar operating inside an NBA engine.

---

## 5. Customer Lifetime Value

### 5.1 What CLV Is — and the Verified Lineage

Customer lifetime value is the present value of the future cash flows attributed to a customer relationship — the "upper limit" on what a firm should spend to acquire or retain that relationship (the Wikipedia CLV article's framing, itself citing the
marketing-metrics literature). The academic lineage was verified this pass at Crossref, and it is a clean one:

- **Berger & Nasr (1998)**, "Customer lifetime value: Marketing models and applications," *Journal of Interactive Marketing* 12(1), 17–30 — the paper that catalogued the CLV calculation models (simple margin × retention geometric series through to more elaborate formulations) ✅ Crossref.
- **Jain & Singh (2002)**, "Customer lifetime value research in marketing: A review and future directions," JIM 16(2), 34–46 — the review that consolidated the field's definitions and open problems ✅ Crossref.
- **Gupta & Lehmann (2003)**, "Customers as assets," JIM 17(1), 9–24 — made the case that customers are intangible assets that can be valued from publicly available data, linking CLV to firm value ✅ Crossref; extended with **Gupta, Lehmann & Stuart (2004)**, "Valuing Customers," *Journal of Marketing Research* 41(1), 7–18, whose headline quantified result — a 1% improvement in retention improves firm value by about 5%, roughly five times the effect of a 1% change in the discount rate — is in the paper's own abstract ✅ Crossref.
- **Schmittlein, Morrison & Colombo (1987)** and **Fader, Hardie & Lee (2005)** supply the stochastic transaction-model machinery (next subsection) ✅ Crossref.
- Term history: the Wikipedia CLV article traces one of the first accounts of the term "customer lifetime value" to the 1988 book *Database Marketing* ⚠ (secondary-source claim; the book itself was not verified this pass).

### 5.2 Computation Approaches — Historical to Predictive

- **Historical (actual) value** — summed realised revenue/margin per customer per period: the accounting view ("customer profitability"), which looks backwards and is easy to compute but cannot drive forward decisions.
- **Simple predictive formulas** — the classic margin × r/(1 + d − r) geometric-series CLV (margin per period, retention probability r per period, discount rate d), the aggregate "average revenue ÷ churn" heuristic, and cohort-based life tables. These are the Gupta–Lehmann-style public-data estimates and the board-deck versions.
- **Traditional RFM-based** — recency/frequency/monetary value cells scored and mapped to value tiers, the direct-marketing approximation of CLV (RFM itself cross-referenced in §2.2).
- **Stochastic transaction models** — the Pareto/NBD → BG/NBD + gamma-gamma stream of §5.3, which model the *process* generating transactions and dropout, and therefore forecast the future rather than extrapolating the past.
- **Full predictive CLV** — machine-learning forecasts of per-period margin and survival (a churn model × a revenue model × discounting), the modern bank implementation: segment-level or even customer-level cash-flow forecasts discounted back, with retention economics from §3's survival models.

The approaches in one view — the trade is always between data hunger and forward-looking power:

| Approach | Inputs | Output | Best for |
| --- | --- | --- | --- |
| Historical margin | Ledger | Past profitability | Cost-to-serve, relationship reviews |
| Geometric formula | Margin, r, d | Perpetuity CLV | Board decks, steering |
| RFM tiers | R/F/M | Value bands | Campaign seeding |
| Pareto/NBD + gamma-gamma | R/F/M per relationship | P(alive), expected transactions and value | Retention targeting |
| Full predictive CLV | All §2 features | Discounted cash-flow forecast | NBA value inputs |

The geometric-series arithmetic is worth doing once by hand, because it explains every CLV number in this guide. Take a deposit customer with S$240/year of net margin to the bank, a 92% annual retention rate, and a 10% discount rate: CLV = margin × r/(1 +
d − r) = 240 × 0.92/(1.10 − 0.92) = 240 × 0.92/0.18 ≈ S$1,227. Raise retention to 95% and the same customer is worth 240 × 0.95/0.15 ≈ S$1,520 — a 3-point retention gain lifts CLV ~24%, which is the same leverage the Gupta–Lehmann–Stuart firm-value result
captures at the portfolio level ✅ (§5.1). Two structural caveats: the formula assumes constant margin and retention in perpetuity (fine for steering, dangerous for valuation), and the discount rate is a policy choice — the repo's ALM sibling ([Treasury &
ALM](treasury_alm_guide.md)) is where the funding-cost side of that choice lives.

### 5.3 The Pareto/NBD → BG/NBD + Gamma-Gamma Stream

The quantitative backbone of modern customer-base analysis is a verified four-paper lineage:

1. **Schmittlein, Morrison & Colombo (1987)**, "Counting Your Customers: Who-Are They and What Will They Do Next?", *Management Science* 33(1), 1–24 — the **Pareto/NBD** model: a Poisson purchasing process with a memoryless exponential dropout process, gamma heterogeneity across customers, and the headline output of a customer-level probability of being still active (P(alive)) computed from recency and frequency alone ✅ Crossref (record and abstract).
2. **Colombo & Jiang (1999)**, "A stochastic RFM model," *Journal of Interactive Marketing* 13(3), 2–12 — the model of monetary value that pairs with the transaction models: average spend gamma-distributed across customers with a gamma heterogeneity ("**gamma-gamma**") — the RFM paradigm made stochastic ✅ Crossref.
3. **Fader, Hardie & Lee (2005a)**, "'Counting Your Customers' the Easy Way: An Alternative to the Pareto/NBD Model," *Marketing Science* 24(2), 275–284 — the **BG/NBD**: a beta-geometric dropout process in place of the Pareto one, vastly easier estimation (the paper's abstract notes parameters obtainable in Excel), and "very similar results" to the Pareto/NBD across purchasing environments ✅ Crossref (record and abstract).
4. **Fader, Hardie & Lee (2005b)**, "RFM and CLV: Using Iso-Value Curves for Customer Base Analysis," *Journal of Marketing Research* 42(4), 415–430 — links the RFM paradigm to CLV formally, combining the Pareto/NBD transaction flow with the gamma-gamma spend submodel, and estimates total CLV for a CDNOW customer cohort ✅ Crossref (full record and abstract).

⚠ Attribution note: the pairing of "Fader, Hardie & Huang" with the gamma-gamma model (a claim circulating in secondary summaries) could **not** be verified. Crossref's record of Fader, Hardie & Huang (2004) is "A Dynamic Changepoint Model for New Product
Sales Forecasting" (*Marketing Science* 23(1), 50–65) — a new-product-sales paper, not a gamma-gamma CLV paper — and no Fader–Hardie–Huang gamma-gamma 2005 record exists ✅ (absence confirmed against Crossref). The verified gamma-gamma lineage is Colombo &
Jiang (1999) → Fader, Hardie & Lee (2005b). The failed-attribution claim is recorded in §12.3.

The BG/NBD + gamma-gamma stack is attractive to banks because it needs only (recency, frequency, monetary value) per customer — exactly the RFM constructs a bank already has — and it produces the two numbers the business wants: P(alive) (a principled
churn probability) and expected future transaction value (a CLV component). Its limits are the model's assumptions (transactions and dropout independent, no covariates in the base model) — modern practice adds covariates via hierarchical-Bayes extensions
and conditional expectations. The repository's technology-side [Customer Lifetime Value Prediction](../technology/customer_lifetime_value_prediction.md) guide carries the implementation detail.

### 5.4 CLV in the Banking Value Context

CLV is the metric that connects this guide to the wealth cluster. The value context — what a relationship is worth by segment, why HNW books are valued the way they are, the RM economics — lives in [Private Banking](private_banking_guide.md) (§1.3, §2.5
segment economics), [Wealth Management](wealth_management_guide.md) (§1.2 segment ladder, revenue pools), and [Investment Portfolio Operations](investment_portfolio_operations_guide.md) (the fee/cost events that cash-flow a CLV model), and is
cross-referenced rather than re-derived. Two banking-specific CLV notes: CLV must be computed **per relationship, not per product** (a customer's deposit attrition while the mortgage stays is a value shift, not a loss); and **retention economics
dominate** — the Gupta–Lehmann–Stuart result (1% retention ≈ 5% firm value ✅) is why churn models (§3) and CLV models are two halves of one program, as the worked example (§11) builds them.

A third banking note is **margin attribution**: a deposit relationship's value runs partly through the bank's internal funds-transfer-pricing ladder rather than through a visible product fee, so the "margin per customer" that feeds every CLV formula is
itself an internal-model output — which is why CLV tiering in a bank is owned jointly by finance and analytics, and why the tiers, not the raw numbers, are what the retention budget of §11 consumes (structural bank practice; the funding-cost mechanics are
the ALM sibling's).

---

## 6. Segmentation

### 6.1 The Segmentation Purposes — Value vs Behaviour

Segmentation is the discipline's oldest organizing act: partitioning the customer book into groups that share something decision-relevant. The two classic purposes pull in different directions:

- **Value-based segmentation** — groups by what customers are worth (deposit tiers, AUM bands, CLV tiers from §5). Its archetype is the wealth cluster's own segment ladder — mass retail → affluent → HNW → UHNW — verified in [Private Banking](private_banking_guide.md) §2 (Capgemini's US$1m/5m/30m bands) and [Wealth Management](wealth_management_guide.md) §1.2. The Capgemini 2026 finding that ~97% of firms still segment primarily by AUM — and that this misses behavioural signals — is a useful warning from the sibling's verified source ✅ (cross-ref the private-banking guide §1.3).
- **Behaviour-based segmentation** — groups by how customers behave: transaction style (salary-banked steady vs cash-cycler vs investor), channel preference (digital-only vs branch), engagement state (new/active/slipping/dormant), life stage. Behavioural segments are where actionability lives: a segment is useful if it maps to a different treatment.

### 6.2 The Approaches — RFM Cells to Mixture Models

- **RFM cells** — the classic direct-marketing segmentation: R/F/M quantiles crossed into a small grid (e.g. 5×5×5 = 125 cells, often collapsed to a handful of value tiers). Crude but instantly computable and explainable; the constructs are §2.2's, the popularization attribution is Arthur Hughes's database-marketing texts ⚠ (see §2.2).
- **k-means** — partitions observations into k clusters by nearest-centroid assignment; fast, scalable to millions of customers, but assumes spherical clusters and needs k chosen (elbow/silhouette — structural practice). The algorithm is textbook-standard (the 1960s coinage "k-means" is commonly attributed to MacQueen ⚠ not re-verified this pass).
- **Hierarchical clustering** — agglomerative or divisive trees of similarity; gives a dendrogram and any granularity, at higher compute cost; used for small books, product taxonomies, and segment trees.
- **Latent class / finite-mixture models** — model-based clustering: the data is assumed to come from a mixture of unobserved segments each with its own parameter vector (e.g. mixture of Gaussians, or latent-class choice models), and the model estimates both the segments and each customer's membership probabilities. The probabilistic membership (soft assignment) is the differentiator: customers belong fractionally to segments, uncertainty is explicit, and the number of segments is a model-selection question (BIC-style criteria — structural practice). Latent-class thinking is also the conceptual bridge to the heterogeneity modeling of §5.3's stochastic customer models.
- **Constructed behavioural states** — the banking overlay: rule-and-model hybrids that assign lifecycle states (new/active/at-risk/dormant/gone) from the constructs of §2.2.

A note the repo's verification conventions require: clustering "quality" (silhouette, within-cluster sums of squares) measures compactness, not business value — segment usefulness is validated downstream by whether segments differ on behaviour and respond
differently to treatments (structural practice; the experimentation overlay is the analytics sibling's).

The segmentation toolbox in one view:

| Approach | What it does | Strengths | Weaknesses | Typical bank use |
| --- | --- | --- | --- | --- |
| RFM cells | Crosses R/F/M quantiles into a grid | Instant, explainable, no fitting | Crude; ignores everything but R/F/M | Value tiers, campaign seeds |
| k-means | k nearest-centroid clusters on scaled features | Fast at millions of customers | Spherical assumption, k choice, hard assignment | Behavioural clusters at scale |
| Hierarchical clustering | Similarity tree at any granularity | Dendrogram, no k needed up front | Compute cost, instability | Segment trees, small books, product taxonomies |
| Latent class / mixture | Model-based soft assignment | Probabilistic membership, explicit uncertainty, model-selection criteria | Assumptions about component shape | Value × behaviour segments, heterogeneity in §5.3-style models |
| Constructed lifecycle states | Rule-and-model hybrids | Directly actionable, auditable | Bank-specific conventions | Churn/dormancy states feeding §3 and §7 |

No approach is "right" in the abstract: the discipline's test is whether the segments differ on behaviour and respond differently to treatments — measurable only with the experimentation overlay of the analytics sibling.

### 6.3 Banking Usage

Banks use segmentation in three connected ways: **portfolio steering** (value tiers set service levels and RM coverage — the wealth cluster's RM-to-client ratios), **treatment design** (behavioural segments pick the offer set, channel and tone of §7's
next-best-action engine), and **model conditioning** (separate churn/propensity models per segment, or segment indicators as features — often better than one global model, at the cost of more models to govern under §10). The retail-to-wealth handoff is
itself a segmentation decision: which mass-affluent customers should be migrated to the private-banking tier, decided on CLV + behaviour, not AUM alone.

---

## 7. Next-Best-Action Models

### 7.1 The Decisioning Framing

Next-best-action (NBA) is less a single model than a **decisioning pattern**: at any customer touchpoint — inbound (app login, call, branch visit) or outbound (campaign, push) — the system computes the single best next action for that customer at that
moment, from an action universe (offer product X, send retention message Y, do nothing, route to RM), constrained by business rules (contact frequency caps, eligibility, regulatory holds) and scored by expected value. The architecture is: model scores in
(churn risk, propensities per product, CLV, uplift per action), an **optimization/decisioning step** that picks the action maximizing expected value under constraints, and an execution layer that delivers it through the channel estate. The decision
engines that run this — business- rule management systems (BRMS) and decision-management platforms with score integration — are the "decision management" technology layer; James Taylor's *Decision Management Systems* (2011) popularized that framing ⚠
(book not re-verified this pass; the decision-management term is industry usage).

An action universe in miniature makes the framing concrete. At the moment a customer opens the Cymbal app, the engine's candidate actions might be:

| Action | Scores that feed it | Typical constraint rows |
| --- | --- | --- |
| Offer 12-month term deposit | Deposit-churn risk, savings propensity, uplift | Rate eligibility, contact cap, no active retention offer |
| Offer credit-card upgrade | Card propensity, CLV tier | Credit eligibility (bureau), conduct/suitability check |
| Retention message + fee waiver | Churn risk bucket, CLV tier, retention uplift | Do-not-contact flag, waiver budget per tier |
| Recommend savings goal feature | Engagement score, app behaviour | Feature eligibility (app version) |
| Route to RM | CLV tier, life event trigger | RM capacity, segment policy |
| Do nothing | All of the above (expected value ≤ 0) | — |

Each row is an expected-value computation under constraints, re-evaluated at every touchpoint; the "no contact" row is what keeps the engine honest (§11.4).

The computation per row is the incremental arithmetic of §4.3 and §9 in one line: expected incremental value of an action ≈ uplift(action) × expected margin of the outcome − contact cost − nuisance cost, where uplift — not raw propensity — is the
multiplier, because the customer who would act anyway contributes nothing incremental. The engine runs this at three decision levels (structural practice): **real-time inbound** (the app session or call, sub-second), **event-triggered batch** (a maturing
deposit, a missed payment, a rate change — minutes to hours), and **outbound campaign waves** (the offer calendar of §4.3, nightly to weekly). The same action universe and constraint rows serve all three levels; only the freshness of the score vector
differs.

### 7.2 The Contested Coinage (⚠)

"Next best action" is an industry coinage, not an academic one, and the guide flags it as such. What is verifiable: **Gartner** publishes on NBA as a sales and marketing practice — the search pass surfaced Gartner's research document "Anatomy of a Next
Best Action for Sales: An Automation Framework" ✅ (gartner.com document page confirmed by title/abstract in the search results), and the CRM-vendor ecosystem (the telco/CRM decisioning vendors and their successors) has marketed "next best action" for two
decades as the real-time, omnichannel descendant of campaign management ⚠ (vendor marketing history; no authoritative origin record was found). What could not be verified: a single first use of the exact phrase, an academic coinage, or an authoritative
standard definition — Gartner's glossary definition itself could not be extracted this pass (paywalled/blocked) and is recorded in §12.4. The honest summary: NBA is a vendor-popularised, Gartner-endorsed framing of an older decisioning idea — scoring plus
rules plus optimization at the point of contact — and this guide treats it as such rather than manufacturing a clean origin.

### 7.3 NBA and Decision Management

The relationship to decision management: NBA is the customer-facing manifestation of the broader decision- management discipline — the practice of encoding repeatable operational decisions as managed, versioned decision services (rules + analytics +
optimization) rather than scattered application logic. A bank's NBA engine is therefore governed like any decision asset: the decision logic is versioned and tested, the models feeding it go through §10's model-risk process, and the action-universe and
constraint rules are owned by the business (marketing/collections/service) with audit trails. NBA engines increasingly consume uplift scores (§9) rather than raw propensities, because the decision question is inherently incremental — "what action changes
this customer's outcome" — which is the precise link between the two sections.

In governance terms the action universe is a product catalogue of its own: every action has an owner, an eligibility rule set, a cost line, an expected-margin line and a contact-frequency budget, and every change to the universe or the constraint rows is
versioned and reviewed like a model change rather than hot-fixed in application code (structural practice — the model-risk and change conventions of §10 apply to the decision layer as much as to the models). The audit trail of "which action, which scores,
which constraints, at which touchpoint" is what lets the estate learn: it is the raw material for the next uplift re-estimation of §9.4.

---

## 8. Recommendation Models

### 8.1 Collaborative Filtering — the Verified Canon

Recommendation models answer "what should this customer be offered next" from the customer's own and others' histories. The canonical lineage was verified this pass:

- **Resnick, Iacovou, Suchak, Bergstrom & Riedl (1994)**, "GroupLens: An Open Architecture for Collaborative Filtering of Netnews," ACM CSCW'94, 175–186 — the paper that named and demonstrated **collaborative filtering** (CF): predict a user's interest from the ratings of like-minded users ✅ (ACM DL record; the authors' own paper page; the RePEc working-paper record, all 1994).
- **Sarwar, Karypis, Konstan & Riedl (2001)**, "Item-based collaborative filtering recommendation algorithms," Proceedings of WWW10, 285–295 — the **item-item** method (recommend items similar to ones the user already liked, similarity computed from co-occurrence across users) that made CF practical at web scale ✅ Crossref (DOI 10.1145/371920.372071).
- **Koren, Bell & Volinsky (2009)**, "Matrix Factorization Techniques for Recommender Systems," *IEEE Computer* 42(8), 30–37 — **matrix factorization**: represent users and items as latent-factor vectors learned from the user-item interaction matrix (the Netflix-Prize-winning family; the paper is the canonical exposition) ✅ Crossref (DOI 10.1109/MC.2009.263).

The family split: **user-user CF** (find similar users, recommend what they liked), **item-item CF** ("people who bought X also bought Y" — generally more stable and interpretable), and **matrix factorization** (latent factors, best accuracy, the Netflix
Prize's lesson that blending many models wins). CF is the workhorse where interactions are rich and plentiful.

Banking interactions are overwhelmingly **implicit** — viewed, clicked, applied, redeemed, opened — rather than explicit star ratings, so the canon's rating-prediction framing maps onto a ranking task: evaluate with **hit-rate@k**, **precision@k** and
**NDCG@k** (normalized discounted cumulative gain, which rewards placing the relevant product near the top of the ranked rail), alongside AUC-style pairwise metrics over implicit-positive vs implicit-negative pairs (structural practice from the
recommender-systems literature; no new canonical anchor is claimed here). Offline ranking quality is only a proxy: the definitive test is online take-up, which is why recommendation rails in banking are champion-challenged like any other customer model
(§10.3) rather than shipped on offline metrics alone.

### 8.2 Content-Based and Hybrid Approaches

- **Content-based** recommendation scores items against the customer's own profile: product attributes (product type, rate band, risk level, fee structure) matched to the customer's revealed preferences. It needs no other customers — a real advantage for cold-start customers and for the sparse long tail of a product shelf — at the cost of serendipity (it only ever recommends more of the same).
- **Hybrid** systems combine CF + content-based + rules (e.g. weighted blends, or CF with content-based fallback for cold start), which is what production recommender stacks in banking actually deploy.
- **Contextual and sequence models** add the when/where: session context and the order of interactions (the clickstream data of §2.1), including the modern neural sequence models — structural industry knowledge, no canonical anchor claimed this pass.

### 8.3 Banking Applications — Product Next-Purchase and Cross-Sell

Banking's recommendation problem differs from media/retail in three ways that matter (structural practice): the **item space is tiny and heterogeneous** (dozens of products, not millions of movies — so item-item CF is shallow and content features carry
more weight); the **interactions are sparse and high-stakes** (a mortgage is not a movie rating; purchase events are rare, regulatory suitability applies — cross-ref the wealth cluster's suitability content); and **negative space is ambiguous** (not
owning a product can mean "not interested", "not eligible", or "owns it elsewhere"). The realistic banking uses are next-product-to-sell recommendations (deposit + card + loan cross-sell sequences — the shelf in [Ancillary Revenue
Products](../management/ancillary_revenue_products_guide.md)), digital-banking content and feature recommendations, and investment-product suggestions under suitability rules (advisory context in [Wealth Management](wealth_management_guide.md));
recommendation scores feed the NBA engine's action universe (§7) alongside propensities (§4) and uplift (§9).

Two operating rules keep recommendation rails inside conduct lines (structural practice): the rail recommends, it does not advise — suitability-checked products only, with the human RM or adviser holding the advice conversation — and the rail is treated
as a decision input with an explicit "do not show" universe (products held, products the customer is ineligible for, products in a cooling-off or complaints state). A recommendation that cannot be actioned is noise; a recommendation that should not be
actioned is a conduct incident waiting to happen.

---

## 9. Uplift Modeling

### 9.1 The Causal Framing — Individual Treatment Effect

Every model so far predicts an outcome. Uplift modeling predicts a **difference**: the change in outcome caused by treating the customer — P(response | treated) − P(response | control) at the individual level, the conditional average treatment effect
(CATE) in causal-inference language. The marketing question is "who should we contact because the contact itself changes their behaviour?", and the causal framing is what separates uplift from propensity: propensity scores rank the *likely* responders
(many of whom would have responded anyway — the "Sure Things"), while uplift targets the *persuadable*. It requires randomized treatment/control data to learn from — holdout control groups in campaigns, or A/B test strata — which is why the
experimentation discipline of the [Analytics Management](../management/analytics_management_guide.md) sibling is a prerequisite, not an optional extra.

The segment taxonomy makes the distinction operational. Radcliffe's four segments (✅ §9.2) map one-to-one onto contact policy:

| Segment | Meaning | Correct action |
| --- | --- | --- |
| Persuadables | Respond only if treated | Treat — the offer earns its cost |
| Sure Things | Would respond anyway | Do not spend on them — the offer is wasted margin |
| Lost Causes | Will not respond either way | Do not treat |
| Do Not Disturbs / Sleeping Dogs | Treatment actively drives them away | Never contact — the action has negative value |

Propensity targeting concentrates spend on Sure Things and Lost Causes alike; uplift targeting spends only where the treatment changes the outcome — which is the entire economic point of the family.

### 9.2 The Verified (and Contested) Origins

The origins are genuinely contested, and this guide presents the evidence rather than a clean story. The verified record (Wikipedia's uplift-modelling article carries the full citation chain, checked this pass):

- **Radcliffe & Surry (1999)**, "Differential response analysis: Modelling true response by isolating the effect of a single action," Proceedings of Credit Scoring and Credit Control VI, Credit Research Centre, University of Edinburgh — the earliest documented formulation under the name "**differential response analysis**" ✅.
- **Lo (2002)**, "The True Lift Model," ACM SIGKDD Explorations Newsletter 4(2), 78–86 — the "**true lift**" formulation from industry practice (Victor Lo at then-Bank One) ✅.
- **Radcliffe (2007)**, "Using Control Groups to Target on Predicted Lift: Building and Assessing Uplift Models," Direct Marketing Analytics Journal — the work that consolidated the "**uplift model**" name and its evaluation, with Radcliffe's widely-used segment taxonomy — **Persuadables, Sure Things, Lost Causes, and Do Not Disturbs/Sleeping Dogs** — named in his companion retention paper ✅ (Wikipedia cites Radcliffe 2007 for the segment names).
- Later canon: Rzepakowski & Jaroszewicz (2010, 2011) uplift decision trees; Kane, Lo & Zheng (2014), "Mining for the Truly Responsive Customers and Prospects Using True-Lift Modeling," *Journal of Marketing Analytics* 2(4), 218–238 — the marketing-journal outing of true-lift methods ✅; Lo & Pachamanova (2015) prescriptive uplift analytics ✅; Devriendt, Moldovan & Verbeke (2018) survey in *Big Data* 6(1) ✅.

⚠ Coinage verdict: "uplift modeling" is practitioner coinage (Radcliffe's 2007 stream and his Scientific Marketer writings popularised the name), while the same family of ideas ran earlier under "differential response analysis" (Radcliffe & Surry 1999),
"true response modelling" (Radcliffe & Surry), and "true lift" (Lo 2002). No single individual cleanly coined "uplift modeling," and secondary accounts that credit one person or one year are simplifications; the guide also notes the older psychology
concept of *differential prediction* as a conceptual antecedent ✅ (Wikipedia). The task brief's "Radcliffe ~2007" is consistent with the verified record; the claim that Lo 2002 coined "uplift" is not supported — Lo's paper is titled "The True Lift Model."

### 9.3 The Approaches — Two-Model, Class Transformation, Causal Forests

- **Two-model (difference) approach** — build one model on the treated group and one on the control group; the uplift score is the difference of the two predicted probabilities. Simple and model-agnostic, but noisy (the difference of two errors) — best used as a benchmark.
- **Class transformation** — a single-model trick: re-label each observation so that treated responders and untreated non-responders get one class and the other two combinations the other, then train one classifier whose score is a monotone transform of uplift (the transformed-target approach; attribution to the practitioner literature ⚠ — the transformation appears in Lo 2002's industry lineage and is documented in the Devriendt et al. 2018 survey ✅, but no single clean academic coinage was verified).
- **Model-based CATE estimation** — causal forests and related heterogeneous-treatment-effect learners (the modern ML-ified versions of the same idea), and the meta-learner families (S-learner/T-learner/X-learner) from the causal-inference literature (structural knowledge; the named implementations — EconML, CausalML, UpliftML — are listed in the verified Wikipedia article's implementations section ✅).

The class-transformation trick deserves one line of mechanics because it is the workhorse: with treatment T ∈ {0,1} and response Y ∈ {0,1}, re-label Z = 1 when (T=1 and Y=1) or (T=0 and Y=0), and Z = 0 otherwise; a classifier of Z then scores
monotonically with the uplift, and only one model needs training, monitoring and validation instead of two — a real saving in the model-risk inventory of §10 (structural practice; the transformation is documented in the verified survey ✅).

### 9.4 Evaluation and Banking Use

Uplift models are evaluated on **randomized holdout data** with uplift-specific curves: the **Qini curve** (incremental responders captured as the targeted fraction grows, measured against the treatment-control difference) and uplift lift; a model adds
value only if the top of its ranking concentrates the incremental response (structural practice; the Qini-curve convention is standard in the uplift literature surveyed by Devriendt et al. 2018 ✅). Banking uses are the high-contact-cost,
high-annoyance-cost decisions where treating the wrong customer is expensive: **retention offers** (contacting a customer who would stay anyway wastes the offer; contacting a "Do Not Disturb" can actively drive the customer away — the telephony/insurance
churn-retention case that motivated the field), **acquisition and cross-sell offers** (only pay for incremental take-up), and **collections treatments**. The worked example (§11) uses uplift as the final gate in the NBA engine.

The evaluation discipline in practice: every treatment wave carries a randomized holdout (typically 5–10% of the eligible audience, stratified by propensity decile so the strata stay balanced), the Qini curve and its summary statistic — the Qini
coefficient, the area between the model's incremental-response curve and the random baseline — are computed on that holdout, and the model earns its keep only when the top of its ranking concentrates incremental response above the baseline. Sample-size
reality intrudes fast: incremental effects of 1–3 points need large holdouts to be distinguishable from noise, which is why uplift learning is a continuous program (every wave re-estimates) rather than a one-off model build (structural practice).

---

## 10. Engineering, Governance and Privacy

### 10.1 The Pipeline — Features, Training, Validation

The customer-model pipeline is classic supervised ML, and it is deliberately condensed here — the lifecycle mechanics (experiment tracking, registries, CI/CD for models) live in the [MLOps Lifecycle
Frameworks](../technology/mlops_lifecycle_frameworks_guide.md) sibling. The steps, in the order a bank runs them:

1. **Feature assembly** — compute the §2.2 constructs from the CRM/transaction/event layers as-of observation dates; store in a feature store so training and online scoring use identical features (the data layer is [CRM Data Warehouse Modelling](../technology/data/crm_data_warehouse_modelling.md)'s subject).
2. **Target definition and labelling** — define the event and the forward window (e.g. "balance zero for 90 consecutive days within 6 months"), respecting the no-look-ahead rule.
3. **Training/validation/test on time** — chronological splits, not random ones (§3.4); class imbalance handled by sampling or weighting — never by distorting the validation split.
4. **Model selection and tuning** — logistic regression baseline vs tree ensembles; hyper-parameter search on the validation window only.
5. **Calibration and thresholding** — probability calibration (isotonic/Platt), then choose the operating threshold from the economics (expected cost of a false alarm × action cost vs value of a caught event).
6. **Champion-challenger and monitoring** — the challenger runs shadow or on a holdout slice; population stability, score drift and feature drift are monitored in production (§10.3).

Two engineering points decide whether this pipeline stays honest at bank scale. First, the **feature store is the contract**: a feature definition (window, aggregation, currency, as-of date) lives in one versioned place and is served identically to the
nightly batch and the sub-second scorer, so training and inference cannot silently diverge. Second, **drift monitoring is a scheduled model, not a dashboard afterthought**: population-stability indices on score distributions, feature-drift tests on the
inputs, and realised-outcome reconciliation on the targets each have an owner, an alert threshold and a documented response (retrain, revalidate, or investigate the data feed) (structural MLOps practice, detailed in the MLOps sibling).

### 10.2 Evaluation — Classic ML, Not LLM

The evaluation of customer behaviour models is classic ML evaluation, and this guide says so explicitly to avoid any confusion with LLM-era evaluation practices: **AUC / lift / gains / profit tables** for ranking models (§3.4, §4.3), **calibration
curves** for probability quality, **C-index** for survival models, **Qini** for uplift (§9.4), and **backtesting** on time-ordered holdout periods as the final arbiter. Business validation adds the profit test: does the top-decile action set clear its
costs at the campaign level? None of this is generative-output evaluation; there are no LLM-as-judge or rubric scores anywhere in the customer-model canon. The generative layer, where banks do use it, sits *above* these models — drafting the offer copy or
the service-agent summary — and is covered by the AI-guide cluster ([LLM Agent Use Cases in Finance](../technology/ai_llm/llm_agent_use_cases_finance_guide.md), [Hybrid Multi-Agent Systems](../technology/ai_llm/hybrid_multi_agent_systems_guide.md),
[Multi-Agent Banking](../technology/ai_llm/multi_agent_banking_guide.md) — the customer-service chapters), which consumes the classical scores rather than replacing them.

### 10.3 Deployment — Scoring, Champions and Challengers

Two deployment patterns dominate: **batch scoring** (the nightly run that refreshes churn/CLV/propensity scores for the whole book — the campaign and reporting engine) and **real-time scoring** (the sub-second score at the touchpoint — the NBA engine's
input, served from the feature store and a low-latency model server). Both feed the decisioning layer of §7; both must be **monitored** — score distributions, feature drift, and realized-outcome feedback loops (the "did the flagged churner actually churn"
reconciliation that closes the loop) — with **champion-challenger** promotion governed like any model change (§10). The behavioural-feedback trap is real: an NBA engine that offers term deposits to everyone flagged as deposit-at-risk will eventually train
its churn model on a world where the offer changed the outcome — which is precisely why uplift data (randomized holds) must be preserved and why §9's models, not §3's, should score the incremental value of the action.

The two patterns plus the event-driven middle in one view:

| Pattern | Cadence | Latency | Serves |
| --- | --- | --- | --- |
| Batch scoring | Nightly | Hours | Campaigns, CLV refresh, reporting, offer calendar |
| Real-time scoring | Per touchpoint | Sub-second | NBA engine, app rails, call-centre desktop |
| Event-triggered scoring | On trigger (maturing deposit, rate move) | Minutes | Retention offers on life events |

Each pattern shares the same model artefact and the same monitoring, differing only in the freshness of the feature vector and the decision latency allowed — which is why the feature store of §10.1 is the architectural load-bearing wall rather than a
convenience.

---

### 10.4 Model Risk Management — SR 11-7 and the MAS Overlay

Customer behaviour models are **models** in the regulatory sense — simplified representations of reality used in decisions — and banks govern them under the model-risk regime that the [Enterprise Risk Management](enterprise_risk_management_guide.md) and
[Risk Management Models](risk_management_models_guide.md) siblings own in depth. The anchor is the US interagency guidance — the Federal Reserve's **SR Letter 11-7 / OCC Bulletin 2011-12, "Supervisory Guidance on Model Risk Management"** (2011) — whose
three pillars (sound development, sound implementation/use, and effective validation with independent challenge) apply to every model that materially affects the bank ⚠ (the guidance itself is the industry's common reference and the ERM sibling's SR 11-7
content is cross-referenced here; the federalreserve.gov page moved and was not re-extracted this pass — recorded in §12.4). In Singapore the MAS overlay on model risk and on AI/GenAI use is carried by the [MAS guide](mas_regulations_guidelines_guide.md)
and the [AI/GenAI Banking Compliance](ai_genai_banking_compliance_guide.md) sibling ⚠ (MAS's specific model-risk guideline texts were not re-extracted this pass; the sibling chain is cited rather than re-derived). The practical consequence for this
guide's subject: a churn model that feeds retention spend, or an NBA engine that drives millions of offers, is a material model — it gets an owner, a model-development document, independent validation (including the time-based backtest of §10.2), a risk
rating, and a slot on the model-risk inventory; changes go through champion-challenger with validation sign-off before promotion.

### 10.5 Privacy — PDPA and GDPR

Customer behaviour modeling runs on personal data, and the privacy overlay is non-negotiable: **Singapore's PDPA** (the Personal Data Protection Act, with its consent, purpose-limitation and notification obligations) and, for any EU/EEA touch, the
**GDPR** (Regulation (EU) 2016/679, with its lawful-basis, purpose-limitation, data-minimisation and individual-rights regime) constrain what may be modeled, with what data, and for how long it may be kept (structural public record; the implementation
detail — retention schedules, DPIA practice, data-protection-by-design — is the [Data Governance Guide](../technology/data_governance_guide.md) sibling's, cross-referenced rather than duplicated). The modeling-specific obligations in practice: purpose
limitation (a model built for fraud detection is not automatically lawful for marketing propensity), minimisation in feature selection (only the data needed for the stated purpose), transparency (the notice that a customer's data is used for analytics),
and deletion/retention discipline on the event histories of §2. The privacy constraints interact with the modelling directly: contact- preference and do-not-contact flags are both a legal requirement and a hard constraint row in the NBA engine's decision
table (§7.1, §11.4).

Purpose limitation deserves the sharpest modelling attention because it is the obligation most often breached by analytics convenience: the same transaction stream that powers a fraud model and a churn model cannot simply flow to a marketing propensity
model without a lawful basis for that purpose, and a model trained on one legal basis cannot be silently re-purposed when the business question changes. In practice this means the analytics estate tracks the purpose attached to each feature set and each
model — a data-governance artefact, not a model artefact — and the DPIAs and retention schedules of the data-governance sibling are reviewed whenever a model family is extended to new data or new decisions (structural practice, cross-referenced; no new
legal claims are made here).

### 10.6 Fairness and Conduct

Customer models also carry fairness and conduct risk: a churn-retention model that systematically under-serves protected groups, or an NBA engine that pushes unsuitable products, is both a conduct failure and a regulatory event (the suitability overlay
lives in the wealth cluster's guides; the AI-fairness and explainability toolkit is [Responsible AI Frameworks](../technology/responsible_ai_frameworks_guide.md)'s subject, cross-referenced). The engineering responses are structural practice:
protected-attribute monitoring of score distributions, disparate-impact checks on model outputs, explainability artefacts (SHAP-style attributions or monotonic logistic coefficients) for the audit file, and human-in-the-loop for high-value or sensitive
actions. None of this is exotic — it is the standard model-governance file extended by fairness metrics, and it is built into the worked example's governance rhythm (§11.5) rather than bolted on.

---

## 11. The Cymbal Bank Worked Example — A Churn and Next-Best-Action Program

### 11.1 The Scenario

Cymbal Bank (the repository's bank persona — see the [MAS guide](mas_regulations_guidelines_guide.md) §1 for the persona conventions) runs a retail book of ~2 million customers in Singapore: current accounts, savings, credit cards, mortgages and personal
loans. The analytics program described here — the **retention-and-next-best-action program** — is the §3–§9 canon assembled into one production estate: a **deposit-and-card churn early-warning model** (the §3 family), a **CLV tiering layer** (§5),
**product propensities** (§4), a **product-recommendation model** for the app's "offers" rail (§8), an **uplift gate** per action (§9), and an **NBA decisioning engine** that picks the action at every touchpoint (§7). All figures below are Cymbal program
parameters — illustrative, not industry claims (consistent with the repo's worked-example conventions).

The program is steered by a small KPI set, each with a target that is a Cymbal program parameter rather than an industry benchmark:

| KPI | Target (program parameter) | What it measures |
| --- | --- | --- |
| Churn top-decile lift, 90-day horizon | ≥ 4× on the test window | Model rank quality (§11.3) |
| Retention-rate movement vs control | +1–2 points per quarter | Program effect (randomized strata) |
| NBA rail redemption | Per-offer targets | Take-up on recommended actions |
| Contact cap | ≤ 6 touches per customer per month | Nuisance control, conduct guardrail |
| Cost per retained customer | < 20% of tier CLV | Retention economics (§5.4) |

### 11.2 Data and Features

The feature layer is assembled nightly from the §2 estate: transaction data (the core ledger and card-authorisation streams), behavioural data (app logins, session depth, feature usage, branch/call contacts), the CRM holdings and demographic master, and
bureau data for credit-eligible offers. The feature sets, all computed as-of date with no look-ahead:

- **Deposit-churn features** — balance trajectory (level, slope, volatility over 4/12/26-week windows), salary-credit presence and recency, login recency/frequency, product count, tenure, complaints, contact history, rate-context (internal rate vs the market ladder — cross-ref [Market Data Consumption](market_data_consumption_guide.md)).
- **Card-dormancy features** — transaction recency/frequency by MCC category, revolving vs transactor behaviour, limit utilisation, app card-control usage.
- **Cross-sell features** — holdings gaps, trigger events (maturing fixed deposit, new salary credit, mortgage drawdown), bureau signals for credit offers, digital engagement segments from §6.

### 11.3 The Models

- **Churn early warning (deposits + cards).** Targets: "primary current account emptied or closed within 90 days" and "card dormant 90 days within 180 days". A gradient-boosted classifier is the production model; a Cox PH model on the same features supplies hazard ratios for the model-risk explainability file, and Kaplan–Meier curves by cohort are the standard portfolio picture (a quarterly cohort review shows attrition by tenure and channel). Evaluation is the time-based discipline of §10: train on 24 months of observation dates, validate on the next 6, test on the final 6; the champion model's top-decile lift and calibration are tracked monthly. (Illustrative Cymbal acceptance bar: top-decile lift ≥ 4× at the 90-day horizon on the test window — a program parameter, not an industry benchmark.)
- **CLV tiering.** The retail book is scored with the BG/NBD + gamma-gamma stack (§5.3) on (recency, frequency, monetary value) per relationship, giving P(alive) and expected 12-month value; customers roll into value tiers that set the retention budget per customer and the service level (the wealth-cluster guides carry the HNW-tier analogue).
- **Propensity and recommendation.** Per-product propensities (§4) and an item-item/hybrid next-product model (§8) score the "offers" universe; the NBA engine consumes both.

The operating cadence keeps the estate current: features recompute nightly; the churn champion-challenger runs monthly on a rolling 24/6/6-month window; CLV tiers refresh quarterly (the BG/NBD parameters are re-estimated, not just re-scored); and the NBA
constraint rows (contact caps, eligibility, consent flags) are owned by the business and reviewed on the same quarterly rhythm as the model-risk file. Every model on the estate — churn, CLV, propensity, recommendation, uplift — has an owner, a development
document and a validation report in the model-risk inventory of §10.4 (Cymbal program structure, consistent with the repo's governance conventions).

### 11.4 The NBA Decisioning Layer

At every touchpoint the decision engine assembles, per customer: churn-risk bucket (from §11.3), CLV tier, per-action propensities, per-action **uplift scores from the last randomized campaign strata** (§9 — the only scores allowed to justify a contact),
and the constraint rows: contact-frequency caps, do-not-contact and marketing-consent flags (PDPA §10.5), eligibility rules (credit product criteria), and conduct rules (no unsuitable investment offers — the wealth-cluster suitability overlay). The engine
then picks the single action maximising expected incremental value under the constraints — offer, retention message, or no contact — and routes it to the app rail, push, or RM workbench. The "do nothing" action is always in the universe: for the Sure
Things and Do Not Disturbs of §9.2, the expected incremental value of contact is zero or negative, and the engine must be able to choose silence.

The per-touchpoint decision flow, executed in order (Cymbal program structure):

1. **Assemble** the score vector for the customer from the feature store (churn bucket, CLV tier, propensities, uplift per action).
2. **Filter** the action universe through the constraint rows (consent, caps, eligibility, conduct) — anything the customer must not be offered is removed before any value computation.
3. **Score** each surviving action: incremental expected value = uplift(action) × expected margin − contact cost − nuisance cost.
4. **Select** the action with the highest incremental expected value, including "do nothing" when no action clears the bar.
5. **Log** the decision — customer, scores, constraints applied, action, channel — as the audit trail and the training data for the next uplift re-estimation.

The engine is deliberately boring by design: no free-form reasoning at the point of decision, no model improvising an offer outside the versioned action universe — the generative layer, where Cymbal uses it, drafts the *copy* for an already-selected
action, and is governed by the AI-guide cluster rather than by this engine (cross-ref §10.2).

### 11.5 Evaluation, Governance and the Operating Rhythm

The program runs on a quarterly rhythm. **Campaign strata** — every outbound treatment carries a randomized control (typically 5–10% holds) so the uplift models and the engine's incremental-value claims can be re-estimated honestly; the churn model's
realized-outcome reconciliation ("did the flagged customers actually churn?") closes the feedback loop of §10.3. **Model governance** — the churn model, the CLV layer and the NBA engine's decision logic each sit on the model-risk inventory with owners,
development documents, independent validation (time-based backtests, challenger runs), and risk ratings, under the SR 11-7 conventions cross-referenced in §10.4; the AI-governance file (fairness monitoring of §10.6) is reviewed with the model validation.
**The business loop** — retention spend is allocated by CLV tier × churn risk × uplift (never churn risk alone — a low-value Sure Thing is not worth a retention offer), and the quarter's champion-challenger results set the next quarter's budget. The whole
estate — features, models, decisions, outcomes — is one auditable chain from the transaction ledger to the offer the customer sees, which is the definition of customer behaviour modeling done as an engineering discipline rather than a stack of notebooks.

The first-year roadmap is a build-up, not a big bang (Cymbal program structure): **Q1** stands up the feature store and the churn early-warning champion with its 24/6/6 time-based validation; **Q2** adds CLV tiering and the first randomized campaign
strata (the 5–10% holds that make §9 possible); **Q3** ships the NBA engine with the churn, CLV, propensity and uplift scores wired to the constraint rows, "do nothing" included; **Q4** hardens the operating rhythm — monthly champion-challenger, quarterly
model-risk review, fairness monitoring, and the realized-outcome reconciliation loop. Each quarter's gate is the KPI table of §11.1: the program earns the next quarter by moving retention and incremental take-up on the randomized evidence, not by shipping
models.

---

## 12. The Claims Audit — Verified, Flagged, Rejected

### 12.1 The Verified Claims (✅)

| Claim | Verification |
| --- | --- |
| Kaplan–Meier product-limit estimator; Kaplan & Meier's manuscripts combined under editor John Tukey into the 1958 JASA paper | ✅ Wikipedia (Kaplan–Meier estimator), citing the 1958 JASA paper |
| Cox proportional-hazards model: λ(t\|X) = λ₀(t)exp(Xβ), coefficients estimable without specifying the baseline hazard | ✅ Wikipedia (Proportional hazards model) |
| Pareto/NBD "counting your customers": Schmittlein, Morrison & Colombo, *Management Science* 33(1), 1–24, 1987; P(alive) from recency/frequency | ✅ Crossref record 10.1287/mnsc.33.1.1 (record + abstract) |
| BG/NBD: Fader, Hardie & Lee, *Marketing Science* 24(2), 275–284, 2005; Excel-easy estimation; results very similar to Pareto/NBD | ✅ Crossref record 10.1287/mksc.1040.0098 (record + abstract) |
| RFM ↔ CLV formal link with gamma-gamma spend submodel and CDNOW cohort application: Fader, Hardie & Lee, *JMR* 42(4), 415–430, Nov 2005 | ✅ Crossref full record 10.1509/jmkr.2005.42.4.415 (record + abstract) |
| Stochastic RFM/gamma-gamma monetary-value model: Colombo & Jiang, *JIM* 13(3), 2–12, 1999 | ✅ Crossref record |
| Berger & Nasr 1998 (*JIM* 12(1), 17–30); Jain & Singh 2002 (*JIM* 16(2), 34–46); Gupta & Lehmann 2003 (*JIM* 17(1), 9–24); Gupta, Lehmann & Stuart 2004 (*JMR* 41(1), 7–18, incl. the ~5% firm-value-per-1%-retention result in the abstract) | ✅ Crossref records (incl. DOIs 10.1002/dir.10032, 10.1002/dir.10045, 10.1509/jmkr.41.1.7.25084) |
| No "Fader, Hardie & Huang 2005 gamma-gamma" record exists; the verified Fader–Hardie–Huang 2004 paper is the Dynamic Changepoint Model for new-product sales (*Marketing Science* 23(1), 50–65) | ✅ Crossref records (attribution claim itself → §12.3 ❌) |
| GroupLens collaborative filtering: Resnick, Iacovou, Suchak, Bergstrom & Riedl, CSCW'94, 175–186 | ✅ ACM DL; authors' page (presnick.people.si.umich.edu); RePEc 1994 record |
| Item-based CF: Sarwar, Karypis, Konstan & Riedl, WWW10, 285–295 | ✅ Crossref 10.1145/371920.372071 |
| Matrix factorization: Koren, Bell & Volinsky, *IEEE Computer* 42(8), 30–37 | ✅ Crossref 10.1109/MC.2009.263 |
| Uplift lineage: Radcliffe & Surry 1999 (Edinburgh CSCC VI); Lo 2002 "The True Lift Model" (SIGKDD Explorations 4(2), 78–86); Radcliffe 2007 (Direct Marketing Analytics Journal); Radcliffe's Persuadables/Sure Things/Lost Causes/Do Not Disturbs taxonomy; Rzepakowski & Jaroszewicz 2010/2011; Kane, Lo & Zheng 2014 (*J. Marketing Analytics* 2(4), 218–238); Lo & Pachamanova 2015; Devriendt et al. 2018 survey (*Big Data* 6(1)) | ✅ Wikipedia (Uplift modelling) full citation chain, checked this pass |
| Gartner publishes NBA research — "Anatomy of a Next Best Action for Sales: An Automation Framework" document exists | ✅ gartner.com document page (title/abstract via search) |
| Capgemini 2026 finding that ~97% of firms still segment primarily by AUM (missing behavioural signals) | ✅ via the private-banking sibling's verified pass (cross-ref [Private Banking](private_banking_guide.md)) |

### 12.2 The Flagged Claims (⚠)

| Claim | Flag |
| --- | --- |
| "Uplift modeling" coinage (who coined it, and when) | ⚠ practitioner coinage; Radcliffe's 2007 stream popularised the name; earlier names: differential response analysis (1999), true lift (2002); no single clean coinage |
| "Next best action" coinage and origins | ⚠ vendor/CRM-industry framing (telco/CRM decisioning vendors), Gartner-endorsed; no academic origin or authoritative first use found |
| Arthur Hughes as the RFM popularizer | ⚠ industry attribution (Hughes's *Strategic Database Marketing* is cited by Fader, Hardie & Lee 2005 for the RFM paradigm ✅-as-citation, but the coinage itself was not verified) |
| k-means coinage attributed to MacQueen | ⚠ common attribution, not re-verified this pass |
| Term "customer lifetime value" first accounts (1988 *Database Marketing* book) | ⚠ per the Wikipedia CLV article; primary source not checked |
| James Taylor's *Decision Management Systems* (2011) as the decision-management popularizer | ⚠ book not re-verified this pass |
| Churn definitions (closed vs dormant vs emptied) | ⚠ bank conventions, no standard |
| FRB SR 11-7 / OCC Bulletin 2011-12 details | ⚠ sibling-verified (ERM guide cross-ref); the federalreserve.gov page moved and was not re-extracted this pass |
| MAS model-risk guideline texts | ⚠ cross-referenced via the MAS and AI/GenAI compliance siblings; not re-extracted this pass |
| Gartner's NBA glossary definition | ⚠ not extractable this pass (paywalled/blocked) |
| The Fader–Hardie–Lee "iso-value" paper's page range claim in secondary summaries | ⚠ the record itself is verified (415–430); some secondary summaries miscite it |

### 12.3 The Rejected Claims (❌)

| Claim | Verdict |
| --- | --- |
| "Fader, Hardie & Huang (2005) proposed the gamma-gamma CLV model" | ❌ no such record exists in Crossref; the Fader–Hardie–Huang 2004 paper is the Dynamic Changepoint Model for new-product sales; the gamma-gamma lineage is Colombo & Jiang 1999 → Fader, Hardie & Lee 2005b |
| "Victor Lo (2002) coined 'uplift modeling'" | ❌ not supported; Lo's paper is titled "The True Lift Model" and the uplift name is documented from the Radcliffe 2007 stream |

### 12.4 What Could Not Be Verified

1. **A single first use of the phrase "next best action"** — vendor marketing history is not archived authoritatively; no academic coinage exists (see §7.2).
2. **The exact coinage history of "uplift modeling"** — the practice lineage is verified (§9.2); who first wrote the word "uplift" is not pinned down by any authoritative source found this pass.
3. **Gartner's glossary definition of NBA** — the document was not extractable (paywall/bot-block); only the existence of Gartner NBA research was confirmed.
4. **The federalreserve.gov SR 11-7 page text** — the URL has moved; the guidance is cited via the ERM sibling's verified treatment.
5. **MAS's current model-risk guideline texts** — cross-referenced via siblings; not re-extracted this pass.
6. **The 1988 *Database Marketing* book as the first CLV account** — Wikipedia's claim, primary source not checked.
7. **RFM's original coinage** (who first built R/F/M cells, and when) — the Hughes popularization is documented, the origin is not.
8. **MacQueen and the k-means coinage year** — common attribution, not verified this pass.
9. **Churn-definition conventions at any named bank** — bank-internal; the worked example's definitions are Cymbal program parameters.
10. **James Taylor's 2011 book contents** — cited as industry framing only.

---

## 13. Glossary

| Term | Definition |
| --- | --- |
| Churn | Customer attrition or dormancy — account closed, balance emptied, activity ceased (bank-defined ⚠) |
| Survival analysis | Time-to-event statistics: Kaplan–Meier curves (non-parametric) and Cox proportional-hazards regression (semi-parametric) ✅ |
| Censoring | Observations known to have survived to the cut-off without the event — handled by survival estimators |
| AUC | Area under the ROC curve — rank-quality metric for classifiers |
| Lift / gains chart | Cumulative responders captured vs random baseline, ranking a model's practical value |
| C-index | Concordance index — the survival-model analogue of AUC |
| Propensity model | Classifier of P(customer does X in a window) — the response-modeling descendant |
| RFM | Recency, frequency, monetary value — the classic behavioural constructs (and cell-based segments) |
| CLV / LTV | Customer lifetime value — present value of future cash flows from the relationship |
| P(alive) | Probability a customer is still active, from the Pareto/NBD and BG/NBD models ✅ |
| Pareto/NBD | Schmittlein–Morrison–Colombo stochastic model of transactions + dropout (1987) ✅ |
| BG/NBD | Beta-geometric/NBD simplification of the Pareto/NBD (Fader, Hardie & Lee 2005) ✅ |
| Gamma-gamma | Stochastic model of average monetary value with gamma heterogeneity (Colombo & Jiang 1999 lineage) ✅ |
| Segmentation | Partitioning the book into value- or behaviour-based groups (k-means, hierarchical, latent class) |
| Latent class / mixture model | Model-based clustering with probabilistic segment membership |
| Next-best-action (NBA) | Decisioning pattern picking the single best action per customer per touchpoint (vendor coinage ⚠) |
| Decision management | The discipline of managed, versioned operational decision services (rules + analytics) |
| Collaborative filtering | Predicting preferences from like-minded users' behaviour (GroupLens 1994 ✅) |
| Matrix factorization | Latent-factor user/item modeling of the interaction matrix (Koren, Bell & Volinsky 2009 ✅) |
| Uplift modeling | Modeling the incremental (causal) effect of a treatment per customer ⚠ coinage |
| CATE | Conditional average treatment effect — the causal target of uplift models |
| Persuadables / Sure Things / Lost Causes / Do Not Disturbs | Radcliffe's uplift segment taxonomy ✅ |
| Qini curve | Uplift evaluation curve: incremental responders vs targeted fraction |
| Champion-challenger | Running a new model against the incumbent on holdout before promotion |
| SR 11-7 / OCC 2011-12 | The US model-risk guidance (sound development, use, validation) ⚠ sibling-verified |
| PDPA / GDPR | Singapore / EU data-protection regimes constraining customer-data analytics |
| Cymbal Bank | The repository's bank persona — the only bank persona in this guide |

---

## 14. Cross-References and Further Reading

**Repository guides (banking siblings — plain filenames):**
- [Private Banking](private_banking_guide.md) — HNW/UHNW segments, RM economics, relationship value (cross-ref §5.4, §6.1)
- [Wealth Management](wealth_management_guide.md) — the segment ladder, advisory models, revenue pools (cross-ref §5.4, §6.1, §8.3)
- [Investment Portfolio Operations](investment_portfolio_operations_guide.md) — the investops lifecycle behind behavioural data and fees (cross-ref §2.1, §5.4)
- [Market Data Consumption](market_data_consumption_guide.md) — external rate/market context features (cross-ref §2.1, §11.2)
- [Enterprise Risk Management](enterprise_risk_management_guide.md) — three lines of defence and model risk under SR 11-7 (cross-ref §10.4, do not re-derive)
- [Risk Management Models](risk_management_models_guide.md) — the model-risk-management discipline and validation (cross-ref §10.4)
- [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) — the Singapore overlay and Cymbal persona conventions (cross-ref §10.4, §11.1)
- [AI/GenAI Banking Compliance](ai_genai_banking_compliance_guide.md) — the AI-risk requirements map (cross-ref §10.4)
- [Treasury & ALM](treasury_alm_guide.md) — the funding-book economics behind deposit attrition (cross-ref §3.3)

**Repository guides (management — prefix `../management/`):**
- [Analytics Management](../management/analytics_management_guide.md) — the analytics-management sibling: response modeling in direct marketing, experimentation, commercial-vs-credit analytics (cross-ref §4, §9.1)
- [Ancillary Revenue Products](../management/ancillary_revenue_products_guide.md) — the cross-sell product shelf (cross-ref §4.2, §8.3)

**Repository guides (technology — prefix `../technology/`):**
- [Customer Lifetime Value Prediction](../technology/customer_lifetime_value_prediction.md) — the CLV implementation companion (cross-ref §5.3)
- [CRM Data Warehouse Modelling](../technology/data/crm_data_warehouse_modelling.md) — the CRM mart beneath the features (cross-ref §2)
- [Data Governance Guide](../technology/data_governance_guide.md) — PDPA/GDPR controls and retention (cross-ref §10.5)
- [Large-Scale Web Data Acquisition](../technology/large_scale_web_data_acquisition_guide.md) — event/clickstream acquisition at scale (cross-ref §2.1)
- [MLOps Lifecycle Frameworks](../technology/mlops_lifecycle_frameworks_guide.md) — the pipeline/MLOps layer (cross-ref §10)
- [LLM Agent Use Cases in Finance](../technology/ai_llm/llm_agent_use_cases_finance_guide.md) — agentic marketing and customer service above the scores (cross-ref §10.2)
- [Hybrid Multi-Agent Systems](../technology/ai_llm/hybrid_multi_agent_systems_guide.md) and [Multi-Agent Banking](../technology/ai_llm/multi_agent_banking_guide.md) — the customer-service chapters (cross-ref §10.2)
- [Responsible AI Frameworks](../technology/responsible_ai_frameworks_guide.md) — fairness, explainability, AI governance (cross-ref §10.6)

**Primary and reference sources used this pass:**
- api.crossref.org — Crossref metadata records for the verified canon (§12.1): 10.1287/mnsc.33.1.1; 10.1287/mksc.1040.0098; 10.1509/jmkr.2005.42.4.415; the Colombo & Jiang 1999 JIM record; 10.1002/dir.10032; 10.1002/dir.10045; 10.1509/jmkr.41.1.7.25084; 10.1145/371920.372071; 10.1109/MC.2009.263
- dl.acm.org and presnick.people.si.umich.edu — Resnick et al. 1994 GroupLens (CSCW'94, 175–186)
- brucehardie.com — Hardie's publication list (incl. Fader & Hardie 2001 CDNOW case, *Interfaces* 31, S94–S107)
- Wikipedia — Kaplan–Meier estimator; Proportional hazards model; Collaborative filtering; Customer lifetime value; Uplift modelling (each carrying its citation chain into the primary literature)
- gartner.com — "Anatomy of a Next Best Action for Sales: An Automation Framework" (document existence; glossary page not extractable)
- Sibling repository guides — cross-referenced as above (Capgemini WWR figures, SR 11-7 treatment, MAS overlay, suitability content, Cymbal conventions)

---

## 15. Closing Summary

Customer behaviour modeling is the discipline of predicting what customers will do next and deciding what to do about it, and this guide has walked the canon with its evidence attached: survival analysis and classifiers for churn (Kaplan–Meier 1958 and
Cox 1972 verified at the literature), the direct-marketing response tradition for propensity, the clean CLV lineage from Berger & Nasr 1998 and Jain & Singh 2002 through Gupta & Lehmann 2003 to the stochastic machinery of Schmittlein–Morrison–Colombo's
Pareto/NBD and Fader–Hardie–Lee's BG/NBD with the gamma-gamma spend model — every record checked at Crossref this pass, and the phantom "Fader, Hardie & Huang 2005" gamma-gamma attribution honestly rejected in §12.3 rather than repeated.

Segmentation, recommendation and next-best-action filled out the decision surface — collaborative filtering's canon verified from GroupLens 1994 to matrix factorization 2009, NBA's vendor-driven coinage flagged ⚠ for what it is — and uplift modeling
closed the loop with its contested practitioner origins presented on the evidence: differential response analysis in 1999, true lift in 2002, the uplift name consolidated around Radcliffe's 2007 work, and the causal question — who does the action actually
change — made explicit.

The engineering and governance overlay kept the discipline honest: classic time-based ML evaluation, champion-challenger promotion, SR 11-7 model-risk governance cross-referenced to the ERM sibling, PDPA/GDPR privacy cross-referenced to the
data-governance sibling, and the Cymbal Bank worked example binding it into one auditable chain from the transaction ledger to the offer rail.

What could not be verified — the NBA coinage, the uplift coinage, Gartner's glossary text, the moved SR 11-7 page — is listed in §12.4 rather than papered over.

The models in this guide share one object and one discipline: whatever the family, each is a lens on a single person's revealed behaviour, and the value of the whole estate is measured the only way it can be — in the changed behaviour of the modeled
customer.

