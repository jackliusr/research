# The Transaction Foundation Model (TFM)

**The Model Class Pre-Trained on Financial Transaction Sequences — What the Label Actually Denotes, Who Promotes It, Why Transactions Are a Sequence Problem, the Data and Architecture, Pre-Training and Downstream Tasks, the Published Evidence (and What It Does Not Establish), the Vendor and Implementation Landscape, Build-versus-Buy-versus-Extend, the Hard Regulatory Constraints, the Evaluation Problem, and a Cymbal Bank Worked Example**

> **Author:** Jack Liu Shurui, Solution Architect
> **Last Updated:** September 2026
> **Primary Sources:** arXiv:2511.19693 (TREASURE, Visa Research, v1 24 Nov 2025 / v3 8 Apr 2026); arXiv:2511.08939 (TransactionGPT, v1 12 Nov 2025 / v2 2 Mar 2026); arXiv:2604.08649 (PRAGMA, Revolut, v1 9 Apr 2026 / v2 24 Aug 2026); arXiv:2507.23267 (nuFormer / "Your Spending Needs Attention", Nubank, v1 31 Jul 2025 / v2 10 Aug 2026); arXiv:2401.01641 and the ICAIF '23 paper DOI 10.1145/3604237.3626850 (Skalski et al., "Towards a Foundation Purchasing Model", 2023); the NVIDIA Technical Blog "Build Your Own Transaction Foundation Model for Financial Intelligence" (Benjamin Wu, 16 Jun 2026) and the NVIDIA-AI-Blueprints/transaction-foundation-model repository; the Plaid engineering blog "Learning the grammar of money with a sequential foundation model" (Raghu Chetlapalli, Wen Yao, 25 Jun 2026) and Plaid's earlier transaction-foundation-model post; the Mastercard news story "Inside Mastercard's new gen AI engine" (2026); the Stripe Sessions 2025 newsroom announcement (7 May 2025); F-Industries, "The Rise of Transaction Foundation Models in Banks" (16 Jun 2026); Regulation (EU) 2016/679 (GDPR) Articles 5, 6 and 22 as reproduced at gdpr-info.eu, with recital 71 in the linked recital set; Regulation (EU) 2024/1689 (AI Act) Article 6(2) and Annex III point 5(b) via artificialintelligenceact.eu Annex III; Regulation B, 12 CFR § 1002.9 at consumerfinance.gov; the MAS FEAT Principles (published 12 November 2018, verified via secondary compilations because the primary MAS page could not be retrieved). **Evidence class is labelled on every claim** — (1) paper, (2) vendor/consultancy publication, (3) vendor benchmark, (4) press. Claims that could not be re-verified live are flagged ⚠ and listed in §15.

---

**How to use this guide:** Section 1 is the identity gate — what the term denotes, who promotes it, and the definitional verdict on whether the named systems are foundation models in the standard sense; read that first, because almost every later claim depends on it. Section 2 is the technical argument (why sequence, and why not). Section 3 is the data and its governance. Section 4 is architecture at design level, deliberately without re-deriving transformer mechanics. Section 5 is pre-training and the downstream task inventory with an evidence grade per task. **Section 6 is the section the guide exists for** — what the published results actually report, and the baseline question. Sections 7–8 are the vendor landscape and the build/buy/extend decision. Section 9 is the regulated-banking constraints, and Section 10 the evaluation problem. Section 11 is the honest state of the field. Section 12 is the Cymbal Bank worked example — explicitly fictional and illustrative. Sections 13–16 are anti-patterns, the claims audit, what could not be verified, and the glossary.

**Boundary.** This guide owns the TFM **as a model class**: its identity, the sequence argument, its data, its architecture at design level, its pre-training and downstream tasks, its published evidence, its implementation landscape, its build-versus-buy economics, the constraints that bind it in a regulated bank, and the problem of evaluating it. It does **not** own: fraud-detection-at-scale engineering discipline (see [financial_fraud_detection_at_scale_guide.md](financial_fraud_detection_at_scale_guide.md)); the AML/KYC and regtech vendor landscape (see [regtech_guide.md](regtech_guide.md)); the payment rails and the transaction engine (see [payment_rails_guide.md](payment_rails_guide.md), [payments_hub_guide.md](payments_hub_guide.md), [bian_transaction_engine_guide.md](bian_transaction_engine_guide.md)); AI regulation and model-risk integration for banking (see [ai_genai_banking_compliance_guide.md](ai_genai_banking_compliance_guide.md)); and transformer, pre-training and fine-tuning mechanics as techniques (see the repository's AI/LLM guides under `../technology/ai_llm/`, e.g. [enterprise_ai_platforms_guide.md](../technology/ai_llm/enterprise_ai_platforms_guide.md), [ai_governance_bias_redteaming_guide.md](../technology/ai_llm/ai_governance_bias_redteaming_guide.md), [llm_evaluation_vs_validation_guide.md](../technology/ai_llm/llm_evaluation_vs_validation_guide.md)). Where this guide touches a topic a sibling owns, it cross-references by name and stops.

**One clarification of vocabulary that this guide holds throughout.** A **TFM is a model class, not a detection system, not a product and not a monitoring platform.** A bank that buys a fraud platform has not thereby acquired a TFM, and a bank that trains a transaction model has not thereby acquired a fraud operating model. Keeping those two apart is the difference between a useful architecture decision and a procurement mistake.

---

**Table of Contents** — numbered section links; each section's own subheadings are listed at its head.

1. [Overview, Identity Gate and Decoder](#1-overview-identity-gate-and-decoder)
2. [Why Transactions Are a Sequence Problem](#2-why-transactions-are-a-sequence-problem)
3. [The Data Landscape and Its Governance](#3-the-data-landscape-and-its-governance)
4. [Architecture at the Design Level](#4-architecture-at-the-design-level)
5. [Pre-Training and Downstream Tasks](#5-pre-training-and-downstream-tasks)
6. [The Published Evidence](#6-the-published-evidence)
7. [The Implementation and Vendor Landscape](#7-the-implementation-and-vendor-landscape)
8. [Build Versus Buy Versus Extend](#8-build-versus-buy-versus-extend)
9. [Regulated-Banking Constraints](#9-regulated-banking-constraints)
10. [The Evaluation Problem](#10-the-evaluation-problem)
11. [The Honest State of the Field](#11-the-honest-state-of-the-field)
12. [The Cymbal Bank Worked Example](#12-the-cymbal-bank-worked-example)
13. [Anti-Patterns](#13-anti-patterns)
14. [The Claims Audit](#14-the-claims-audit)
15. [What Could Not Be Verified](#15-what-could-not-be-verified)
16. [Glossary, Cross-References and Closing Summary](#16-glossary-cross-references-and-closing-summary)

---

## 1. Overview, Identity Gate and Decoder

### 1.1 The Short Answer

"Transaction Foundation Model" (TFM) is a **label applied to transformer-based (or, historically, recurrent) neural models pre-trained on sequences of financial transactions and then adapted to downstream tasks** — fraud and anomaly detection, recommendation and personalisation, credit and lifetime-value prediction, and increasingly reconciliation and network-side analytics.

The label is **not a settled standard**. As of this pass it is used by three non-overlapping constituencies: payment network operators describing their own internal models (Visa Research, Mastercard), fintechs and challenger banks with proprietary transaction corpora (Stripe, Plaid, Nubank, Revolut), and consultancies and infrastructure vendors who are naming a market (F-Industries, NVIDIA, Thoughtworks). Each uses "foundation model" to mean something slightly different, and only some of the uses survive the standard definition — see 1.3.

The problem the class addresses, in one line: **a customer's financial behaviour lives in the order, spacing and mutual relationship of their transactions, and a per-transaction tabular model discards that structure by construction.**

### 1.2 Who Promotes the Term, and How Consistently

Four classes of promoter, with material differences in what they mean:

- **Network operators (payment networks).** Visa Research published TREASURE as "a multipurpose transformer-based foundation model specifically designed for transaction data" (class 1, arXiv:2511.19693v3) and TransactionGPT as "a foundation model for consumer transaction data within one of the world's largest payment networks" (class 1, arXiv:2511.08939v2). Mastercard, by contrast, publicly describes its generative model as a **"large tabular model"** rather than a sequence or foundation model, explicitly drawing the contrast with LLMs built on unstructured data (class 2/4, Mastercard news story 2026, corroborated by press). So even between the two largest networks the framing diverges — and Mastercard's "tabular" self-description sits awkwardly against the sequence thesis this guide examines.
- **Fintechs with transaction corpora.** Stripe announced "the world's first AI foundation model for payments" at Sessions on 7 May 2025, describing a model "trained on tens of billions of transactions, with unsupervised learning to generate nuanced embeddings" (class 2, Stripe newsroom; class 4, TechCrunch 7 May 2025). Plaid published a "transaction foundation model" for event-level meaning, then a **"sequential foundation model"** in June 2026 with self-supervised objectives (class 2, Plaid engineering blog, 25 Jun 2026). Revolut published PRAGMA as "a family of foundation models for banking event sequences" (class 1, arXiv:2604.08649v2). Nubank's nuFormer is described as "transformer-based representation learning models for transaction data" using self-supervised learning (class 1, arXiv:2507.23267v2) — note the more careful wording, which stops short of "foundation model" in the abstract.
- **Consultancies and opinion sources.** F-Industries defines a TFM as "an AI model trained specifically on financial transaction data: payments, wire transfers, ACH batches, trade settlements, interbank messages, and the metadata that surrounds them" and frames the question as one of "infrastructure" rather than product (class 2, findustries.co, 16 Jun 2026). That definition is materially looser than the academic one: **it requires only that the model be trained on transaction data — not that it be pre-trained, self-supervised, at scale, or transferable.** A supervised gradient-boosted model on transaction features would satisfy it. F-Industries publishes no datasets, metrics or evaluations; it is a named party's claim. Thoughtworks is named in the same breath by the dispatcher's reconnaissance, but its article body could not be retrieved (see §15) — a tool limitation, not evidence of absence.
- **Infrastructure vendors enabling the build.** NVIDIA's technical blog teaches readers to "Build Your Own Transaction Foundation Model" and lists Stripe, Nubank/NuFormer, Visa/TransactionGPT, Mastercard's large tabular model, Revolut/PRAGMA and Plaid as "industry signal" (class 2, developer.nvidia.com, 16 Jun 2026).

**Consistency verdict:** the term is used in at least three senses — (i) a self-supervised pre-trained event-sequence encoder that transfers to many tasks (the academic sense); (ii) any model trained on transaction data (the consultancy sense); (iii) a marketing frame for a vendor's payments ML. The three do not coincide, and a reader who assumes they do will over-credit the evidence.

### 1.3 The Definitional Verdict — Foundation Model or Fashionable Name?

Apply the standard definition: **pre-train on broad, largely unlabelled data at scale, then adapt or transfer to many downstream tasks, including tasks the developer did not select.** Judged against that:

- **Structurally genuine where self-supervised pre-training on unlabelled event sequences is followed by demonstrated multi-task transfer.** TREASURE, TransactionGPT, PRAGMA and the NVIDIA blueprint each pre-train on unlabelled sequences and then demonstrate transfer. That is the real thing in form.
- **Structurally genuine but narrow.** nuFormer and the NVIDIA blueprint demonstrate transfer into essentially **one task family** (recommendation at Nubank; fraud classification in the blueprint). Pre-training plus embedding reuse into an existing tabular model is a real transfer mechanism, but a single demonstrated downstream task is not the "many tasks" clause.
- **Not a foundation model in any meaningful sense.** Mastercard's "large tabular model" is a vendor's own framing of a structured-data neural network; nothing in the public material establishes self-supervised pre-training on unlabelled sequences or transfer to developer-unselected tasks. F-Industries' definition is satisfied by models that are not pre-trained at all.
- **The unresolvable qualification.** Even for the genuine cases, the *foundation* property that matters commercially — **transfer to a bank other than the one that built it** — is asserted far more often than it is demonstrated. Skalski et al. is the strongest published case for cross-institution transfer (≤pretrained on data from 180 issuing banks, then evaluated on held-out issuers); TREASURE, TransactionGPT and PRAGMA demonstrate transfer **within the publishing institution's own data and tasks**, on non-public corpora. In the open literature, no third party has independently reproduced any of these results.

**The verdict the guide adopts:** *the leading named systems are foundation models in architecture and pre-training method, and largely unproven as foundation models in the transfer sense that matters to a buying bank.* The label is defensible for TREASURE, TransactionGPT, PRAGMA, Skalski et al. and (in form) Stripe/Plaid/NVIDIA based on what they publish; it is **not** earned by the marketing-forward uses of the term, and there is no evidence that any of these models transfers usefully into a *different* institution's transaction distribution.

### 1.4 Where Each Named System Falls

| Named system | Publisher | Pre-trained, self-supervised, at scale? | Multi-task transfer demonstrated? | Data | Verdict per the standard definition |
|---|---|---|---|---|---|
| TREASURE (arXiv:2511.19693) | Visa Research | Yes — ~6B transactions, 30M cardholders, next-transaction + network-signal objectives | Yes, but in-house only: abnormal-behaviour detection and recommendation embeddings | Non-public "industry-grade" | Genuine in form; transfer demonstrated only inside Visa |
| TransactionGPT (arXiv:2511.08939) | Visa Research | Yes — "billion-scale real-world transactions", 3D-Transformer | Yes, in-house: anomaly detection, future-transaction generation | Non-public, company datasets | Genuine in form; in-house transfer only |
| PRAGMA (arXiv:2604.08649) | Revolut | Yes — masked modelling on a large banking event corpus | Yes: credit scoring, fraud detection, lifetime value, via linear probe then light fine-tuning | Non-public | Closest to the textbook definition in its stated protocol; no external replication |
| nuFormer (arXiv:2507.23267) | Nubank | Yes — SSL on transaction data | Narrow: large-scale recommendation; embeddings integrated with existing tabular features | Non-public | Pre-trained encoder, one downstream domain — not yet a foundation model on the "many tasks" clause |
| Skalski et al. (ICAIF '23 / arXiv:2401.01641) | FeatureSpace/others | Yes — generative pre-training on 5.1B transactions | Fraud detection plus public-dataset self-supervised benchmarks; out-of-domain issuer transfer | 180 issuing banks (non-public) **plus public datasets** | The strongest transfer evidence in the published record; title itself hedges: "Towards a Foundation Purchasing Model" |
| NVIDIA blueprint | NVIDIA | Yes — ~29M-parameter Llama decoder pre-trained from scratch | Narrow: fraud AP lift; repo claims swappable tasks | **IBM TabFormer — 24.4M synthetic transactions** | Genuine pre-training, tiny model, synthetic data, one demonstrated task |
| Mastercard "large tabular model" | Mastercard | Not established publicly | Not established publicly | Anonymised transaction data | A vendor's structured-data neural model; nothing published supports the sequence or transfer claim |
| Stripe payments foundation model | Stripe | Described by the vendor as unsupervised pre-training on tens of billions of transactions, then a classifier | Not established publicly | Non-public | Genuine in form as described; no paper, no metrics, no replication |
| Plaid transaction / sequential foundation models | Plaid | Yes, per the vendor — CPC, RTD and TCL self-supervised objectives | Stated as "in testing" on two use cases; no numbers published | Plaid's network | Genuine in form as described; evidence class 2 only |
| F-Industries' "TFM" | Consultancy | Not required by their definition | Not required | n/a | A market label, not a technical class |

### 1.5 The Decoder — Vocabulary

- **Foundation model** — a model pre-trained at scale on broad data and adaptable to many downstream tasks. Coined for language and vision; extended to tabular, time-series and, here, transaction data. The load-bearing word is *transfer*, not *large*.
- **Pre-training / fine-tuning** — pre-training is the large-scale, usually self-supervised phase that learns general representations; fine-tuning is the smaller supervised phase that adapts them to a task. Cross-ref the repository's `../technology/ai_llm/` guides for the mechanics; this guide does not re-derive them.
- **The event sequence** — a customer's transactions as an ordered series of timestamped events, each with attributes that may be static (constant across the history) or dynamic (changing per event).
- **Sequence representation (embedding)** — a fixed-length vector summarising a variable-length history. TREASURE, nuFormer, PRAGMA, the NVIDIA blueprint and Plaid all produce one; the embedding is what gets sold or reused.
- **Masked modelling** — hide part of the input and predict it, forcing the model to learn context. PRAGMA uses masked modelling; masked and next-event prediction are the two dominant objectives in this literature.
- **Transfer** — the reuse of pre-trained representations on a task other than the pre-training objective. This is the claim that carries the label, and the claim that is hardest to verify.
- **The downstream task** — the actual decision: is this transaction fraudulent, is this applicant creditworthy, what will this customer buy next, will they churn.
- **The tabular baseline** — the feature-engineering plus gradient-boosted-decision-tree pipeline (XGBoost/LightGBM/CatBoost class) that dominates production fraud and credit. It is the incumbent, not a straw man; the honest question in this guide is always whether a TFM beats *it*, not whether it beats logistic regression.

### 1.6 The Four Evidence Classes

Every substantive claim in this guide carries one of four labels, and the labels are not decorative: the topic is consultancy-led and vendor-led, and its value depends on keeping the strata separate.

1. **Paper** — peer-reviewed or archival (arXiv, ACM/KDD/ICAIF).
2. **Vendor or consultancy publication** — a named party describing its own product, market or method.
3. **Vendor benchmark or case study** — a performance number published by the party that also built the thing being measured.
4. **Press** — third-party reporting, which usually relays a class-2 claim without independent testing.

A claim appearing in NVIDIA, Thoughtworks or F-Industries material is a **named party's claim**, never a finding. A performance number published by the model's own builder is class 3 and is not transferable to another institution's data.

**Verification note for §1.** Verified live: the title, author list, submission dates, subject class and licence of arXiv:2511.19693, 2511.08939, 2507.23267, 2604.08649 and 2401.01641; the NVIDIA blog's date, author, all six technical specifics and the identity of its "industry signal" list; the Plaid sequential-foundation-model post's date, authors and objective set; the Mastercard news story and its "large tabular model" phrasing; the Stripe Sessions 2025 newsroom headline and the TechCrunch coverage date. Flagged ⚠: the F-Industries definition (URL verified as existing; page body not read in full during this pass); the Thoughtworks article (not retrievable, §15). Note that the "world's first" element of Stripe's claim is **contradicted by the published record** — Skalski et al. appeared at ICAIF in November 2023, and nuFormer predates Sessions 2025.

---

## 2. Why Transactions Are a Sequence Problem

### 2.1 What a Sequence Carries That a Row Cannot

The published framing, attributed: TREASURE's authors observe that "raw transaction data is logged as individual entries whenever transactions occur, with all attributes potentially changing between transactions", and that "since our goal is to model cardholder behavior, it is more meaningful to group transactions from the same card together and order them chronologically" (class 1, arXiv:2511.19693v3 §1). Plaid makes the same argument in customer terms: two consumers with "nearly identical 90 day summaries — same monthly income, same average balance, same rent amount, the same number of overdraft fees, and similar category-level spending" can be in completely different financial situations, because "the sequence tells a different story: one user experienced a temporary shock and recovered; the other is caught in a recurring liquidity squeeze" (class 2, Plaid engineering blog, 25 Jun 2026). NVIDIA's version: most production tabular pipelines depend on "hand-engineered features and rule sets that are brittle, expensive to maintain, and blind to the sequential structure inside a customer history" (class 2, NVIDIA technical blog, 16 Jun 2026).

The specific information at stake:

- **Order and causality.** A €900 deposit arriving *before* the €900 debit is a funded account; the same two amounts reversed is a return, a fee and possibly a closed account. Aggregates over a month are identical.
- **Cadence and periodicity.** Salary on the 25th, rent on the 1st, a subscription on the 12th. Regularity is itself a signal; its interruption is often the earliest observable sign of a life event, an account takeover or a liquidity crisis.
- **Interval and velocity.** The gap between transactions, and the compression of that gap, carry information that a count of transactions in a window destroys.
- **Joint evolution across attributes.** The merchant, the amount, the channel, the country and the response code co-evolve. A deviation is only a deviation relative to the customer's own trajectory.
- **Mutual relationship between events.** Two transactions may be individually unremarkable and jointly diagnostic — the classic fraud pattern of a small test charge followed shortly by a large one.

### 2.2 The Event-Stream View of Financial Behaviour

Under the sequence view, a customer is a stream, not a table. PRAGMA describes the resulting model as a "general-purpose representation layer" trained on "the discrete, variable-length nature of financial records" (class 1, arXiv:2604.08649v2); Plaid describes the model as learning "how financial events combine, in what order, and over what cadence" (class 2, 25 Jun 2026); TREASURE models a cardholder by grouping transactions per card, ordering them, and separating attributes that stay constant from those that vary (class 1, §1 and §3.1). The practical consequence is architectural: the unit of prediction becomes *the history*, not *the row*, and the model's output is a representation of a customer that many different heads can consume.

### 2.3 What the Tabular Formulation Discards by Construction

A tabular model is a function of a feature vector. To build that vector from an event stream you must aggregate: count, sum, mean, max, time-since-last, and rolling windows. Aggregation is **many-to-one and order-destroying**. Once "number of transactions in the last 24 hours" has been computed, the sequence of those transactions is gone; once "mean ticket size over 30 days" exists, the fact that the last three tickets were each three standard deviations above the customer's own distribution is gone unless an engineer thought to add a feature for it.

This is not a failure of tabular modelling; it is a statement about where the information goes. Practitioners compensate by engineering order-sensitive features (time-since-last-transaction, sequences of MCC codes, recency-weighted counts), which is precisely the point: **the sequence model internalises the feature engineering that a tabular pipeline must externalise, and it does so without a human choosing in advance which temporal patterns matter.** Whether that is a benefit or a vendor talking point depends entirely on §6.

### 2.4 The Counter-Argument — Why Tabular Won

The tabular pipeline is the incumbent in production fraud and credit for defensible reasons, and a guide that omits them is propaganda:

- **Interpretability.** A gradient-boosted tree yields per-decision attributions that a model-validation function, an auditor and an adverse-action disclosure can all use. This is not a soft preference; in credit and in disputed fraud blocks it is a legal requirement (§9).
- **Latency and cost.** A tree ensemble scores in single-digit milliseconds on CPU. A transformer over a 512-event history does not, without serious engineering.
- **Small-data and cold-start robustness.** Trees do well with modest labelled data, mixed types and missing values. A pre-trained sequence model's advantage is largest exactly when you have scale and where the pre-training distribution matches the customer.
- **The extreme-sparsity problem.** Fraud is rare — often well under 0.2% of events — and the events carrying signal are rarer still. From the model's perspective, most of a transaction sequence is a near-duplicate of the customer's routine, and the informative deviations are a handful of tokens. Learning to attend to those, without over-fitting to noise, is genuinely hard; TREASURE's own sequence-length cap of 512 events (class 1, §4.1) exists partly to bound that cost.
- **Maturity of tooling, monitoring and governance.** The tabular model has an established drift-detection, retraining, back-testing and documentation practice. The sequence model does not (see [financial_fraud_detection_at_scale_guide.md](financial_fraud_detection_at_scale_guide.md) for the production discipline and §10 here for the evaluation gap).

### 2.5 When the Sequence Formulation Actually Pays

The honest question is not "is sequence modelling better" but "under what conditions does the extra machinery pay for itself". Stripped of promotion, the conditions are:

1. **You hold long, dense, first-party histories for the same customers.** Short or sparse histories give the sequence model nothing to model.
2. **Your existing feature pipeline has hit a ceiling that is traceable to temporal structure.** If the incumbent's residual errors are not order-related, a sequence model will not fix them.
3. **The task tolerates the latency and the explainability cost.** Ranking a next-best merchant offer is not the same as declining a payment or refusing credit.
4. **You have an evaluation capability that can detect a small real lift.** Without it you cannot tell a genuine improvement from a favourable test split (see §10).
5. **You can train, version and retrain it.** A pre-trained encoder is a model with a lifecycle, not an installation.

Where any of the five is missing, the sequence formulation is a research project wearing a product's clothes. Section 8 turns this into a decision procedure, and Section 12 works a concrete case.

---

## 3. The Data Landscape and Its Governance

### 3.1 What Transaction Data Exists and Who Holds It

The raw material is heterogeneous, and the heterogeneity matters because it determines what each holder can train:

- **Card authorisation and clearing records** — held by the card networks and, in parallel, by issuers and acquirers. Rich in network signals: response codes, approval/decline flags, stand-in-processing decisions, system flags. TREASURE's whole differentiator rests on this signal class (class 1, §1, §3.1). Visa describes itself in the same paper as processing "over 300B transactions or 15T dollars annually between more than 4B credentials and 150M+ merchants across 200+ countries" citing its own 2024 fact sheet (class 1 citing a class-2 corporate document).
- **Account-to-account and internal ledger movements** — held by the account-servicing institution. Plaid's model operates on this view of a consumer across institutions (class 2, 25 Jun 2026).
- **Wire transfers and interbank messages** — including SWIFT-format message traffic; held by the correspondent and messaging layers. F-Industries names "wire transfers, ACH batches, trade settlements, interbank messages" explicitly in its definition and claims a correspondent-banking use case around "SWIFT message patterns" (class 2, findustries.co, 16 Jun 2026). Cross-ref [payment_rails_guide.md](payment_rails_guide.md) and [payments_hub_guide.md](payments_hub_guide.md) for the rails themselves.
- **Batch clearing and settlement files** — ACH, RTP, card settlement; lower event rate per file, much higher value per event.
- **Surrounding metadata** — device, channel, geography, merchant descriptors, timestamps, and derived context. Plaid's model explicitly fuses "the meaning of the transaction itself", "timing, including the order of events, the spacing between them, and recurring calendar patterns" and "account and transaction attributes such as amounts, account characteristics, and other contextual signals" (class 2, 25 Jun 2026).

### 3.2 Schema and Semantic Heterogeneity

The same economic event appears differently across sources. A card purchase carries an authorisation code and a merchant category; the same purchase seen from the issuer's ledger carries a posting date and a running balance; seen through an aggregator it carries a normalised descriptor and a cleaned merchant identity. Plaid's stated motivation for building on its network is exactly this: "Plaid's network spans thousands of financial institutions and millions of consumers, giving the model exposure to a wide variety of financial formats, income levels, and economic behaviors" (class 2, 25 Jun 2026). Two consequences for a builder: (i) a tokenizer and vocabulary are institution-specific artefacts, not a standard (see §4.1); (ii) an embedding learned on one institution's schema does not port to another's without a mapping layer that is itself a modelled object.

### 3.3 The Labels Problem

Labels are the binding constraint, and they are worse than ordinary supervised learning problems:

- **Fraud labels are sparse.** Even at network scale, the positive rate is a fraction of a percent. TREASURE's abnormal-behaviour task is reported through an undisclosed in-house metric precisely because the underlying figures are sensitive and presumed small (class 1, §4.1).
- **Fraud labels are delayed.** A fraud determination arrives after disputes, chargebacks and investigation — weeks to months after the event. Any model trained on confirmed fraud is training on a decision made with information the model never had, and any offline metric computed before labels mature is optimistic.
- **Fraud labels are adversarially contaminated.** Confirmed-fraud labels reflect what the *current* system caught. Fraud the incumbent filters out never becomes a label, so the training set is a biased sample of the fraud population — a selection effect that penalises any challenger model evaluated on it.
- **Credit labels are delayed by years.** A default label matures over the life of the loan, and the population is shift-prone across economic regimes.
- **Genuine "normal" is unlabelled.** The largest useful supervision signal in the sequence setting is self-supervision on unlabelled events — which is why pre-training is attractive here at all (class 1, PRAGMA and Skalski et al.; class 2, Plaid).

### 3.4 The Volume Asymmetry

The models in this class are pre-trained on corpora of a size that only a handful of parties possess: ~6B transactions from 30M cardholders (TREASURE, class 1), "billion-scale real-world transactions" (TransactionGPT, class 1), 5.1B transactions from 180 issuing banks (Skalski et al., class 1), and "tens of billions of transactions" (Stripe, class 2). A single mid-size bank's own transaction history is orders of magnitude smaller and covers a different, narrower population.

This is the structural asymmetric fact of the field: **whoever sits on the network holds the pre-training corpus.** It is also the reason "transfer across institutions" is the claim that matters and the claim that is least demonstrated — the parties with the data have no incentive to publish a cross-institution transfer result, and the parties without it cannot produce one.

### 3.5 Governance — Who May Train on What

The secondary-use question is the practical block for a bank, and it is jurisdiction-specific. In the EU:

- **Purpose limitation, GDPR Article 5(1)(b)** — personal data must be "collected for specified, explicit and legitimate purposes and not further processed in a manner that is incompatible with those purposes", with an express carve-out for archiving, scientific/historical research and statistical purposes under Article 89(1). Verified against the reproduced Regulation text at gdpr-info.eu. Whether large-scale pre-training of a commercial model is "incompatible" with the purpose for which the transaction data was collected is a **judgement a controller must document**, not a settled question — and a general-purpose encoder trained on the whole customer base is exactly the case where the "specified, explicit purpose" is hardest to articulate in advance.
- **Lawful basis, GDPR Article 6** — each processing operation needs a basis. Fraud prevention has a specific hook (Article 6(1)(d) vital interests and 6(1)(f) legitimate interests, with recital 47 recognising fraud prevention as a legitimate interest), whereas training a transferable representation for unspecified future uses sits much less comfortably on any single basis and is very difficult to base on consent without the consent being demonstrably specific and freely given. This guide flags the Article 6 reasoning as an assessment, not a verified provision-by-provision conclusion: only Articles 5 and 22 were read at primary text during this pass.
- **Automated decision-making, GDPR Article 22** — verified: the data subject has "the right not to be subject to a decision based solely on automated processing, including profiling, which produces legal effects concerning him or her or similarly significantly affects him or her", with exceptions including contractual necessity, and — where those exceptions apply — a right "to obtain human intervention on the part of the controller, to express his or her point of view and to contest the decision". Recital 71 is the recital the Article points to for profiling safeguards.
- **The EU open-finance instruments.** PSD2 and its successor PSD3, together with the proposed Financial Data Access (FIDA) framework, govern access to and reuse of customer payment-account data (this guide's summary ⚠ — the EUR-Lex primary text could not be retrieved during this pass; see §15). The relevant point for a builder is directional and reliable even so: the EU regime is moving toward *customer-controlled sharing of financial data for specified purposes*, which is a narrower permission than *the platform holder trains a general model on everything it sees*.

Elsewhere:

- **United States.** GLBA governs the privacy of non-public personal information held by financial institutions and constrains disclosure to non-affiliated third parties; FCRA governs the use of consumer report information in credit decisions and imposes obligations on users of such information. This guide flags both as **not verified at primary text during this pass** — the precise operative restrictions on internal secondary use of first-party transaction data differ from the restrictions on sharing it, and an architect should have counsel resolve the specific question rather than rely on this summary.
- **Singapore.** The PDPA's consent-and-purpose framework governs collection and use of personal data, with the PDPC's legitimate-interests regime available in defined circumstances. MAS has separately published the **FEAT principles** (Fairness, Ethics, Accountability, Transparency) for the use of AI and data analytics in the financial sector — published 12 November 2018, non-binding, four principles, with the Veritas initiative as the accompanying assessment toolkit. Verified here via secondary compilations (OECD AI policy database, FinTech News Singapore, and specialist compilations) because the primary MAS page could not be retrieved; see §15. Cross-ref [ai_genai_banking_compliance_guide.md](ai_genai_banking_compliance_guide.md) for the supervisory material and [banks_in_singapore_guide.md](banks_in_singapore_guide.md) for the regime.

### 3.6 Jurisdictional Differences

Three differences matter to this model class specifically:

- **Purpose limitation is enforced against modelling, not just against sharing.** A regime can permit an institution to *hold* transaction data and still restrict *pre-training a general encoder* on it, because the purpose is not the one consented to or the one the basis covered.
- **Fraud detection is treated differently from credit.** The EU AI Act expressly carves fraud detection out of its high-risk credit-scoring entry (verified, §9.4), while GDPR Article 22 protection is strongest precisely where an automated decision significantly affects a person — so a sequence model used to *block a payment* and the same model used to *refuse credit* face materially different obligations even when the code is identical.
- **Group-wide reuse is restricted differently from internal reuse.** A model pre-trained by a group entity on one jurisdiction's data and fine-tuned in another raises cross-border transfer questions on top of purpose-limitation questions.

The practical governance conclusion: **before any modelling work, the data-rights question must be answered in writing for (a) the jurisdiction of collection, (b) the jurisdiction of training, (c) the jurisdiction of deployment, and (d) whether embeddings derived from the data are themselves personal data.** That last point is unresolved in this guide and is discussed in §9.5.

**Verification note for §3.** Verified at primary/reproduced text: GDPR Article 5(1)(b) and Article 22, both read at gdpr-info.eu, which reproduces the Regulation. Verified at a regulator page: the structure of Regulation B § 1002.9's adverse-action notice requirements at consumerfinance.gov (used in §9). Verified via secondary compilations only: MAS FEAT's date, non-binding status and four principles. Flagged ⚠ and moved to §15: the Article 6 lawful-basis reasoning; the GLBA and FCRA summaries; the PSD2/PSD3/FIDA summary; the PDPA summary. Company-reported volumes (Visa's 300B/15T figures; Stripe's "tens of billions") are quoted as the companies' own statements, not as verified measurements.

---

## 4. Architecture at the Design Level

> **[A note on scope.]** This section describes the *design decisions* these systems make. Transformer mechanics — self-attention, decoder-only versus encoder-only, positional encoding, fine-tuning protocols — are owned by the repository's AI/LLM guides under `../technology/ai_llm/` and are not re-derived here.

### 4.1 The Tokenisation-of-Transactions Problem

A transaction is not a token. It is a record with a dozen-plus fields of mixed type, several of which are categorical with enormous vocabularies. Every design in this class has to answer the same question — *what is the unit that goes into the transformer?* — and the published answers differ:

- **NVIDIA's blueprint** uses a "custom domain tokenizer that converts each transaction into roughly 12 semantic tokens with a 6,251-symbol vocabulary" (class 2, NVIDIA technical blog, 16 Jun 2026). The framing is deliberately language-model-like: the transaction becomes a short sentence, and the customer's history becomes a document. Vocabulary size is small and deliberate — a fixed 6,251 symbols is a bounded, maintainable artefact that a bank could audit.
- **TREASURE** does not tokenise the transaction into a flat symbol sequence at all. It carries two structurally different inputs — a **static attribute vector** and, per transaction, a **dynamic attribute vector** — each containing numerical and categorical attributes. Numerical values are log-scaled (because "all numerical features in our dataset... exhibit long-tail distributions") and passed through a linear layer; categorical values are mapped to per-attribute category indices and looked up in **attribute-specific embedding tables**. The static representation is placed first in the transformer input so that, under causal masking, every dynamic position can attend to it (class 1, arXiv:2511.19693v3 §3.1–§3.3).
- **Plaid** describes combining three complementary sources — meaning learned from large-scale financial data, timing (order, spacing, calendar patterns), and account/transaction attributes — into "a unified representation of financial activity over time" (class 2, Plaid, 25 Jun 2026). It does not publish the encoding scheme.
- **PRAGMA** is described as addressing "the discrete, variable-length nature of financial records" with masked modelling over a heterogeneous banking event corpus (class 1, arXiv:2604.08649v2), again without a published tokenizer specification.

**The design consequence.** Vocabulary and tokenization are the least transferable part of the whole stack. NVIDIA's is published and swappable; TREASURE's categorical mappings are explicitly withheld — the paper states that mechanisms exist "to handle new category values... that may appear during inference but were not present during training" but that "the specific implementation details are omitted to protect sensitive information about the raw category values" (class 1, §3.3). A bank evaluating a vendor model should therefore ask the tokenizer question first: *what are the symbols, who maintains them, and what happens on an unseen merchant?*

### 4.2 Static Versus Dynamic Attributes

TREASURE's first named capability is "an input module with dedicated sub-modules for static and dynamic attributes, enabling more efficient training and inference" (class 1, abstract). The paper's own motivation is that after grouping and sorting a card's transactions, "certain attributes remain constant throughout a card's transaction history, while others vary with each transaction", and it cites prior work on field-and-time-aware transformers for sequential tabular data to justify specialised sub-modules (class 1, §1, citing Zhang et al. 2023). In TREASURE's sampled dataset each transaction comprises **five static and sixteen dynamic attributes** (class 1, §4.1).

Why the split matters architecturally: a static attribute (account type, card product, tenure band) need not be re-encoded at every timestep, which reduces compute and, more importantly, stops the model from re-learning a constant as if it were a signal. This is a cheap design win and, notably, one of the few that a bank could copy without any of the vendor's data.

### 4.3 Encoder Choice and Context Length

- **TREASURE** uses a **3-layer Transformer decoder with 4 attention heads and hidden dimension 256**, an input module of 3 layers, causal masked self-attention, and **no explicit positional encoding** — on the argument that a decoder-only architecture with causal masking "inherently captures the relative ordering of transactions through its autoregressive nature" (class 1, §3.2). The model is small by LLM standards. It was trained for 20 epochs with AdamW at learning rate 10⁻⁴, batch size 256 (class 1, §4.1).
- **TransactionGPT** introduces "a novel 3D-Transformer architecture specifically tailored for capturing the complex dynamics in payment transaction data", with design innovations for "modality fusion and computational efficiency" and joint optimisation with downstream objectives (class 1, arXiv:2511.08939v2).
- **NVIDIA's blueprint** uses "a compact Llama decoder with ~29M parameters... pretrained from scratch using NVIDIA NeMo AutoModel" (class 2, 16 Jun 2026). 29M parameters is roughly four orders of magnitude below a frontier LLM and is a deliberate engineering choice for a reproducible teaching artefact.
- **PRAGMA** is "a Transformer-based architecture" pre-trained with masked modelling (class 1, abstract).

**Context length.** TREASURE caps sequence length at **512 transactions**, which it states "covers over two years of transaction history for most cardholders and provides a deterministic computational budget", and asserts that under this configuration "the model's inference latency remains operationally feasible for real-time transaction processing" (class 1, §4.1). That is the single most useful engineering number in the published record for a bank: it tells you the frontier of the class as published is *a few hundred events*, not an unbounded history — and it is a design constraint chosen for latency, not a modelling limitation.

### 4.4 High-Cardinality Outputs

This is the genuinely novel technical work in the class, and TREASURE states the problem precisely: predicting the next merchant with standard cross-entropy "would require computing over 150M+ logits for each prediction", which is "significantly more computationally expensive" than language models whose vocabularies are "typically under one million" (class 1, §1). The paper's solution set:

- Attributes are split by a **cardinality threshold of 1,024** into low- and high-cardinality groups (class 1, §3.5). In the dataset, "categorical attributes range in cardinality from 2 to over 100M".
- Low-cardinality attributes use **standard cross-entropy**; numerical attributes are modelled as **log-normal distributions** via negative log-likelihood over predicted mean and standard deviation, so the model emits a distribution rather than a point estimate.
- High-cardinality attributes use an **InfoNCE-style contrastive loss** over the positive category plus a sampled set of negatives, with negatives **shared across all timesteps and samples within a batch**. The paper reports the memory effect directly: the shared strategy kept forward-pass memory below 3GB and backward-pass below 6GB, whereas independent sampling hit out-of-memory above 64 negatives per positive, measured under PyTorch 2.6 (class 1, §3.5).
- Rare and new entities are mapped to **shared aggregated identifiers** to control vocabulary size (class 1, §3.5).

The overall objective is not a plain sum: the abnormal-behaviour loss is used as a **reference to scale the other attribute losses**, so that the primary detection task dominates the gradient direction while auxiliary tasks still contribute (class 1, §3.5, Eq. 4). The paper's ablation reports that this scoring is materially better than both a simple sum and an equal-contribution weighting, and that the equal-contribution strategy degraded abnormal-behaviour detection severely (class 1, §4.2, Table 1).

### 4.5 Customer and Merchant Hierarchies, and Network Signals

- **The customer as the sequence unit.** TREASURE groups by card and orders chronologically; the sequence is the cardholder's history (class 1, §1, §3.1). Plaid's unit is the consumer across institutions (class 2, 25 Jun 2026).
- **Merchant hierarchy.** TREASURE predicts merchant, merchant country, merchant city and merchant category as separate attributes (class 1, §4.1) — a coarse-to-fine hierarchy in effect. Its embedding visualisations show merchants of the same country clustering into regional super-clusters and businesses of similar type clustering together (class 1, §4.6), which the authors read as evidence that the learned geometry reflects real market structure.
- **Network signals as first-class inputs to the objective.** This is TREASURE's distinguishing design claim: the model has **two prediction heads per timestep** — one for the attributes of the *next* transaction, one for the *current* transaction's **payment network signals (response codes and system flags)**. The paper is explicit that network signals are used exclusively as outputs "as they represent outcomes of a transaction... only known after it has been processed" (class 1, §3.1). The authors' stated contrast with prior work is that an earlier transaction foundation model "does not incorporate signals from the payment network itself" and therefore "cannot leverage direct indicators from the payment network, such as issuer decline codes, which provide powerful, real-time evidence" (class 1, §2).
- **Graph signals.** This guide found no published design in this class that incorporates an explicit transaction-graph or entity-resolution layer as part of the pre-training architecture. Where graph analysis appears in bank practice it generally does so as a separate analytical layer; cross-ref [financial_fraud_detection_at_scale_guide.md](financial_fraud_detection_at_scale_guide.md) for graph-based detection and [regtech_guide.md](regtech_guide.md) for the vendor tooling around it. Nothing in the sources below establishes that TFMs absorb that layer.

### 4.6 Pre-Training Objectives

| Objective | Where it appears | What it trains the model to do |
|---|---|---|
| Next-transaction attribute prediction | TREASURE (class 1) — "the model learns cardholder behavior through next-transaction prediction tasks" | Predict the next event's attributes from the history |
| Current-transaction network-signal prediction | TREASURE (class 1) | Predict response codes and system flags for the current event |
| Masked modelling | PRAGMA (class 1) | Reconstruct masked events from surrounding context |
| Generative autoregression on transaction sequences | Skalski et al. (class 1) | Learn contextualised embeddings by generative pre-training |
| Causal / next-token language modelling over a transaction token stream | NVIDIA blueprint (class 2) — a Llama decoder pre-trained from scratch | Learn transaction "language" from a domain tokenizer |
| Contrastive Predictive Coding (CPC) | Plaid (class 2) | Make future financial activity more predictable |
| Replaced Token Detection (RTD) | Plaid (class 2) | Recognise events plausible in isolation but inconsistent with the sequence |
| Temporal Contrastive Learning (TCL) | Plaid (class 2) | Align representations from different periods of the same user |
| Joint self-supervised + downstream optimisation | TransactionGPT (class 1) | Fuse pre-training with task objectives |
| Auxiliary attribute-prediction tasks with loss rescaling | TREASURE (class 1) | Preserve detection performance while multi-task learning helps generalisation |

Two things are worth noticing. First, **the objective menu is standard** — it is metric learning, masked modelling and contrastive prediction, transplanted. The novelty in this class is in the input encoding (§4.1–4.4), not in the loss functions. Second, **the most distinctive objective in the published record (TREASURE's network-signal head) is the one least available to anyone who is not a network operator**, because it consumes data only the network holds.

### 4.7 Design Attribution Table

| Design choice | Adopted by | Source class | Transferable to a non-network bank? |
|---|---|---|---|
| ~12 semantic tokens per transaction, 6,251-symbol vocabulary | NVIDIA | 2 | Yes — published and described as swappable |
| Static/dynamic sub-modules | TREASURE | 1 | Yes — reimplementable from the paper |
| Input module 3 layers, decoder 3 layers / 4 heads / 256 hidden | TREASURE | 1 | Yes |
| No explicit positional encoding (causal masking implies order) | TREASURE | 1 | Yes, but the justification is architecture-specific |
| Log-scaled numerics with log-normal output heads | TREASURE | 1 | Yes |
| InfoNCE with batch-shared negatives; 1,024-cardinality threshold | TREASURE | 1 | Yes |
| 512-transaction context cap | TREASURE | 1 | Yes, as a starting budget |
| Payment-network signal head (response codes, system flags) | TREASURE | 1 | **No** — requires network data |
| 3D-Transformer with joint downstream optimisation | TransactionGPT | 1 | Not reimplementable from the abstract alone |
| CPC + RTD + TCL objective combination | Plaid | 2 | Conceptually yes; implementation unpublished |
| ~29M-parameter Llama decoder on synthetic data | NVIDIA | 2 | Yes — open repository |
| Linear probe then light fine-tuning on embeddings | PRAGMA | 1 | Yes — this is the cheapest adaptation recipe published |

**Verification note for §4.** All TREASURE architectural specifics are quoted or paraphrased from the verified HTML body of arXiv:2511.19693v3 (§3.1–§3.5, §4.1–§4.6, Tables 1–3), including the 3-layer/4-head/256-dimensional configuration, the 20-epoch/AdamW/10⁻⁴/batch-256 training setup, the 5-static/16-dynamic attribute split, the 1,024-cardinality threshold, the 2-to-100M cardinality range, the 150M-logit figure, the memory measurements under PyTorch 2.6, and the 512-event cap. NVIDIA's tokenizer, vocabulary and 29M-parameter figures are from the verified 16 Jun 2026 blog. Plaid's three objectives are from the verified 25 Jun 2026 post. TransactionGPT's 3D-Transformer and PRAGMA's masked-modelling architecture are quoted from their verified arXiv abstracts — **not** from their full bodies, which were not read during this pass; any finer architectural claim about those two papers should be treated as unverified (see §15).

---

## 5. Pre-Training and Downstream Tasks

### 5.1 The Objective Inventory

The distinction that matters operationally is between **pre-training objectives** (self-supervised, no labels, run once at scale) and **downstream adaptation** (supervised, labelled, run per task per institution). The published adaptation recipes form a ladder of increasing cost:

1. **Embedding extraction, frozen encoder, shallow head.** TREASURE provides embeddings to a downstream recommender (class 1, §4.5). PRAGMA reports that "strong performance can be achieved by training a simple linear model on top of the extracted embeddings" (class 1, abstract). NVIDIA compresses 512-dimensional per-user embeddings to 64 dimensions with PCA before use (class 2, 16 Jun 2026).
2. **Embeddings concatenated with existing tabular features.** This is the NVIDIA blueprint's actual recipe: "combining raw tabular features with foundation-model embeddings in an XGBoost classifier" (class 2, 16 Jun 2026). Nubank's nuFormer takes the same shape — "an end-to-end fine-tuning method that integrates user embeddings with existing tabular features" (class 1, abstract).
3. **Light fine-tuning of the encoder.** PRAGMA states performance "can be further improved with lightweight fine-tuning" (class 1, abstract).
4. **Full joint training of encoder and task head.** TransactionGPT reports joint optimisation with downstream objectives (class 1, abstract).

For an architect, step 1 and step 2 are the ones that matter, because they are the ones that leave the incumbent model in place (see §8.4).

### 5.2 Task Inventory with Evidence Grades

Every row below states what the task requires, who has published, and — the load-bearing column — whether the published work shows a gain over the **incumbent** (a well-tuned tabular or rules-plus-tabular production system) or **merely over a naive or internal baseline**.

| Downstream task | What it requires | Who has published | Published comparison against | Grade |
|---|---|---|---|---|
| **Fraud / abnormal-behaviour detection** | Low-latency scoring, extreme class imbalance, tolerance of delayed labels | TREASURE (class 1): 111% RI vs **Visa's own production systems**; TransactionGPT (class 1): "significantly improves downstream anomaly transaction detection performance against a competitive production model"; Skalski et al. (class 1): improves value-detection rate at high precision thresholds on hold-out issuer data; NVIDIA (class 2/3): AP lift over an XGBoost baseline **on synthetic data, with the TFM augmenting the XGBoost model** | **Incumbent production models, but internal and non-public**, plus one XGBoost-augmentation result on synthetic data | **Strong on paper, weak on reproducibility.** No published comparison against an externally tuned boosted-tree model on a public, real transaction corpus. See §6.6 |
| **Transaction monitoring / financial-crime screening** | Alert generation under regulatory obligation, auditability, documented tuning | **No published TFM paper found in this pass.** F-Industries asserts the benefit (class 2, no evidence). The vendor landscape is owned by [regtech_guide.md](regtech_guide.md) | n/a | **Un-evidenced for this class.** Named-party claim only |
| **Credit and underwriting** | Long label maturation, documented adverse-action reasoning, fair-lending testing | PRAGMA (class 1) lists credit scoring among its downstream tasks and reports "superior performance across multiple domains directly from raw event sequences"; Plaid (class 2) argues the credit-risk case qualitatively — "the overall composition of a borrower's financial life often matters more than their current balance" — with no published numbers | PRAGMA reports superiority across domains but on its own corpus; Plaid publishes no metric at all | **Weak.** No published adverse-action-compatible comparison; none of the sources addresses the explainability requirement this task carries |
| **Customer personalisation and recommendation** | Offline ranking metrics that map to online engagement | TREASURE (class 1): embeddings outperform supervised task-specific embeddings by an average of 104% across HR@K and NDCG@K settings; nuFormer (class 1): "improvements for large-scale recommendation problems at Nubank... achieved solely through enhanced representation learning rather than incorporating new data sources" | **Supervised task-specific embeddings** trained for the same task — a genuine same-task baseline, though internal | **Moderate-to-good for the recommendation task**, and the best-defended claim in the class: both TREASURE and nuFormer compare against a supervised model actually built for that task, not against a naive one. Still internal and non-reproducible |
| **Retention and churn** | Multi-month label windows, intervention cost accounting | Skalski et al. (class 1) names churn prediction as a motivating use case; NVIDIA (class 2) lists churn prediction among swappable downstream tasks; no published churn result found in this pass | n/a | **Asserted, not evidenced** |
| **Merchant / network-side applications** | Merchant-level aggregation, network-level risk | TREASURE (class 1) prediction of next merchant, country, city and category with Prec@1 reported; F-Industries (class 2) claims correspondent-banking and cross-border use cases around SWIFT message patterns with no evidence | TREASURE reports Prec@1 against its own variants and production systems; the merchant ranking task has no external benchmark | **Merchant-side: internal only.** Network-side (F-Industries): **claim only** |
| **Lifetime value prediction** | Long horizons, heavy censoring | PRAGMA (class 1) lists it; NVIDIA (class 2) lists it | Own corpus | **Asserted** |
| **Reconciliation and exception handling** | Operational data beyond transactions, human-in-the-loop | F-Industries (class 2) asserts it, with no metric, dataset or case study | n/a | **Claim only.** Note also that this is an operations task, not a financial-decision task, which weakens the "high-risk decision" objection but does not supply evidence |

### 5.3 Reading the Grades

Three patterns are worth stating plainly:

- **Where the task is internal to a network operator, the evidence is strongest and least reproducible.** Fraud detection and next-merchant prediction are exactly where TREASURE has production comparators and exactly where no outsider can check the result.
- **Where the task is a ranking problem, the baseline is honest.** Both TREASURE and nuFormer compare against a supervised model built specifically for the task. That is the right comparison and it is to the field's credit that they published it. It is still not a public benchmark.
- **Where the task carries a legal obligation, the evidence is absent.** Credit, adverse action and financial-crime screening are the tasks with the heaviest documentation requirements and the thinnest published TFM evidence — and no source in this pass demonstrates a TFM meeting an adverse-action standard. That asymmetry is the single most important thing an architect can take from this section.

**Verification note for §5.** TREASURE's embedding-frozen and recommender results, the 104% average improvement and the HR@K/NDCG@K metric names are from the verified HTML body (arXiv:2511.19693v3 §4.5). PRAGMA's linear-probe-then-fine-tune statement is quoted from its verified abstract. nuFormer's end-to-end fine-tuning description and its "enhanced representation learning rather than... new data sources" claim are quoted from its verified abstract. NVIDIA's PCA compression and XGBoost augmentation are from the verified 16 Jun 2026 blog. The absence of published TFM work on financial-crime screening and churn is **an absence of evidence in this pass's searches, not proof that none exists** — recorded as such in §15.

---

## 6. The Published Evidence

> **This is the section the guide exists for.** Its purpose is to establish what the published results actually show — datasets, task definitions, baselines, metrics — and to state plainly where comparison against a well-tuned tabular baseline is absent, weak, or favourable only on a vendor-selected benchmark.

### 6.1 TREASURE — What the Paper Actually Reports

**Identity (verified).** arXiv:2511.19693, "TREASURE: The Visa Payment Foundation Model for High-Volume Transaction Understanding", 15 authors under Visa Research, Foster City, CA. v1 submitted 24 Nov 2025, v2 26 Nov 2025, v3 8 Apr 2026. Primary class cs.LG, cross-listed cs.AI, CC-BY-4.0. The HTML v3 header records acceptance at the **Proceedings of the 32nd ACM SIGKDD Conference on Knowledge Discovery and Data Mining**, DOI 10.1145/3770854.3783942 — so this is archival/peer-reviewed-class work (evidence class 1), not merely an arXiv preprint. The name is an acronym: **TRansformer Engine As Scalable Universal transaction Representation Encoder**.

**Datasets (verified from the paper body).** The pre-training corpus was "approximately six billion transactions from 30 million distinct cardholders, recorded between September 1, 2020, and November 30, 2022" — a 26-month window split as 24 months training, the 25th month for validation, the final month for testing (class 1, §4.1). Each transaction comprises five static and sixteen dynamic attributes. **The attribute names are withheld**: "The specific names of these attributes cannot be disclosed due to the sensitive nature of the project." The paper describes this as "industry-grade datasets" (abstract). **This is not a public corpus and cannot be obtained, audited or re-used by any reader.**

**Tasks and metrics (verified).**
- **Next-transaction prediction**, measured by **Prec@1** for merchant, merchant country, merchant city and merchant category, and by **sMAPE** for transaction amount (class 1, §4.1).
- **Abnormal-behaviour detection**, measured by an **undisclosed in-house metric** whose absolute values are not published. The paper states it "measure[s] the model's performance in abnormal behavior detection using an in-house performance metric. Detailed performance figures cannot be disclosed due to the sensitive nature of this task." What *is* published is a **ratio** to the deployed system, which the authors term **Relative Improvement (RI)**, defined so that "a ratio of 1.5 signifies that the evaluated model is 50% better than the currently deployed system" (class 1, §4.1).
- **Merchant recommendation**, measured by **HR@K and NDCG@K** in a two-tower architecture, with training/validation/test drawn from transactions after 1 September 2022 so that the pre-training corpus did not contain those interactions (class 1, §4.5).

**Reported results (verified).**
- TREASURE achieved an **RI of 2.1171** on abnormal-behaviour detection against the currently deployed system, which the paper headlines as **111% improvement**. The comparable figures in the same table are **1.9606** for a single-purpose internal model and **1.8768** for the simple loss-aggregation strategy, with the equal-contribution strategy at **0.1768** (i.e. materially worse than the incumbent) (class 1, §4.2, Table 1). Note that 2.1171 rounds to 112%, not 111%; the paper states 111%.
- Against architecture variants **of its own model** — TREASURE+LSTM at RI 1.4427 and TREASURE+GRU at RI 1.3979 — the Transformer configuration wins (class 1, §4.3, Table 2).
- Recommendation embeddings outperform supervised task-specific embeddings by an **average of 104%** across HR@K and NDCG@K settings (class 1, §4.5).
- Sharing negatives across a batch, rather than sampling per timestep, removes a substantial memory ceiling: forward pass below 3GB, backward below 6GB, versus out-of-memory above 64 negatives per positive under independent sampling (class 1, §3.5).
- The paper reports ablation studies and a scaling analysis showing performance improves with both data volume and model size (class 1, §4.7), but the paper's own claim of "scaling law for transaction data" is asserted in the contributions list and this pass did not read the §4.7 body in detail — treat the specific scaling-law shape as unverified.

**Deployment status (verified).** The paper does **not** claim production deployment. It states that deployment "follows a rigorous multi-stage process involving extensive offline evaluation, parallel validation phases, and regulatory review" and that "the comprehensive offline evaluation results presented in this paper represent a significant milestone in this process, and we are actively working toward full production deployment" (class 1, §4.1). Visa separately states it "deploys 100+ machine learning models" on payment network data (class 1 citing a class-2 corporate source). **The headline 111% therefore rests on offline evaluation against internal production comparators, not on a live A/B result.**

**One internal inconsistency worth flagging.** The abstract and §4.5 report the recommendation improvement as **104%**; the introduction's own summary of the same result states **90%** ("enhance recommendation system performance by 90%") (class 1, §1 versus abstract and §4.5). The 104% figure is defined in §4.5 as an average across settings, which may explain the divergence — but the paper as published carries two different numbers for the same claimed result, and a reader should note it. Flagged accordingly in §14.

### 6.2 The 111% Figure, Read Precisely

The number is vendor-internal and must be read with four qualifiers, all of them from the paper itself:

1. **The baseline is the vendor's own production system**, not a published, well-tuned boosted-tree benchmark. It is described as "the currently deployed system" (class 1, §4.1, §4.2). How that system is architected, how it was tuned, and how it compares to a state-of-the-art gradient-boosted model are **not stated**.
2. **The metric is undisclosed.** RI is a ratio on "an in-house performance metric" whose definition and absolute values the paper explicitly withholds. A reader cannot tell whether the underlying measure is precision, recall, average precision, a detection rate at a fixed alert budget, or a composite business KPI — nor at which operating point the comparison is made. RI of 2.1171 is therefore not interpretable in absolute terms.
3. **The dataset is non-public and unauditable.** No external party can reproduce, extend or contest the number.
4. **The evaluation is offline.** The paper states the model is not yet in full production.

None of this makes the result false. It makes it **a class-3 vendor benchmark carried in a class-1 publication** — a distinction this guide holds throughout, and the reason the claims audit in §14 lists the 111% as *flagged*, not *verified*.

### 6.3 The Skalski et al. Prior Work and TREASURE's Distinction

**Verified identity.** The earlier transaction foundation model TREASURE cites as "(Skalski et al., 2023)" is: **P. Skalski, D. Sutton, S. Burrell, I. Perez, J. Wong, "Towards a Foundation Purchasing Model: Pretrained Generative Autoregression on Transaction Sequences"**, in *Proceedings of the Fourth ACM International Conference on AI in Finance* (ICAIF '23), November 27–29, 2023, Brooklyn, NY, pages 141–149, DOI 10.1145/3604237.3626850, also at arXiv:2401.01641 (submitted 3 Jan 2024, v2 4 Jan 2024), licence CC-BY-NC-SA-4.0.

**Verified claims from the abstract.** It presents "a generative pretraining method that can be used to obtain contextualised embeddings of financial transactions"; benchmarks "on public datasets demonstrate that it outperforms state-of-the-art self-supervised methods on a range of downstream tasks"; and it performs "large-scale pretraining of an embedding model using a corpus of data from 180 issuing banks containing 5.1 billion transactions and appl[ies] it to the card fraud detection problem on hold-out datasets", reporting that "the embedding model significantly improves value detection rate at high precision thresholds and transfers well to out-of-domain distributions."

**Why this paper carries unusual weight in the guide.** It is the only work in this pass that combines (a) large-scale pre-training on real, multi-institution transaction data, (b) **benchmarking against other self-supervised methods on public datasets**, and (c) an explicit **out-of-domain transfer** evaluation on held-out issuers. Those are the three things the rest of the field mostly omits. Caveats remain: the pre-training corpus is non-public, and the abstract claims improvement over self-supervised baselines — which is a different and weaker comparison than improvement over a well-tuned supervised boosted-tree incumbent on real data. Note also that the paper's own title hedges — *Towards* a Foundation Purchasing Model.

**TREASURE's stated distinction.** TREASURE acknowledges the shared "treats transactions as a sequence of events" premise and then distinguishes itself on two grounds: (i) Skalski et al. "does not incorporate signals from the payment network itself" and therefore "cannot leverage direct indicators from the payment network, such as issuer decline codes, which provide powerful, real-time evidence that significantly enhances tasks like abnormal behavior detection"; (ii) Skalski et al. "relies on Gated Recurrent Units (GRUs)", an architecture that TREASURE's §4.3 experiments "demonstrate to be inferior to the Transformer for modeling transaction sequences" (class 1, §2). The paper then claims: "To the best of our knowledge, TREASURE is the first Transformer-based foundation model designed to holistically model payment transactions by simultaneously capturing both consumer behavior and payment network signals, validated at an industrial scale."

**The honest reading of that comparison.** Point (i) is well-taken and structural — network signals are a genuinely different input class, available only to a network operator. Point (ii) is **not a head-to-head comparison**: TREASURE's evidence against GRUs is TREASURE-with-a-GRU-backbone (RI 1.3979) versus TREASURE-with-a-Transformer-backbone (RI 2.1171), both trained on Visa's own data with Visa's own objective and loss function (class 1, §4.3, Table 2). **The paper does not run Skalski et al.'s model on its data, and Skalski et al.'s public-dataset benchmarks are not revisited.** So the claimed architectural superiority of transformers over GRUs is demonstrated *for this architecture and this corpus*, and the comparison the citation implies — "our model beats theirs" — is not made.

### 6.4 TransactionGPT, PRAGMA and nuFormer

**TransactionGPT (verified identity).** arXiv:2511.08939, title "TransactionGPT", 29 authors, v1 12 Nov 2025, v2 2 Mar 2026, cs.LG and cs.CL, marked "Technical Report" in arXiv comments (not stated as accepted at a venue), CC-BY-4.0. Verified abstract claims: it is "a foundation model for consumer transaction data within one of the world's largest payment networks"; designed "to understand and generate transaction trajectories while simultaneously supporting a variety of downstream prediction and classification tasks"; a "novel 3D-Transformer architecture"; "trained on billion-scale real-world transactions"; it "significantly improves downstream anomaly transaction detection performance against a competitive production model and exhibits advantages over baselines in generating future transactions"; evaluations use "a diverse collection of company transaction datasets spanning multiple downstream tasks"; and it benchmarks LLM-derived embeddings and fine-tuned LLMs, "demonstrating that TGPT achieves superior predictive accuracy as well as faster training and inference". **Note the shared authorship with TREASURE** — several authors appear on both papers, and both are Visa Research. The two are sibling efforts, not independent corroborations, and a reader counting "two papers" should count them as one programme. No quantitative results are stated in the abstract; the body was not read in this pass.

**PRAGMA (verified identity).** arXiv:2604.08649, "PRAGMA: Revolut Foundation Model", 14 authors, v1 9 Apr 2026, v2 24 Aug 2026, cs.LG with cs.CE/cs.CL/cs.IR/q-fin.CP cross-listings, CC-BY-4.0. Verified abstract claims: "a family of foundation models for banking event sequences"; pre-trains "a Transformer-based architecture with masked modelling on a large-scale, heterogeneous banking event corpus using a self-supervised objective tailored to the discrete, variable-length nature of financial records"; supports "credit scoring, fraud detection, and lifetime value prediction"; "strong performance can be achieved by training a simple linear model on top of the extracted embeddings and can be further improved with lightweight fine-tuning"; and "PRAGMA achieves superior performance across multiple domains directly from raw event sequences". **This is the cleanest statement of the textbook foundation-model protocol in the pass** — self-supervised pre-training, frozen-embedding probe, light fine-tuning, multiple named downstream tasks. The "superior performance" claim is against unspecified baselines on a non-public corpus; the abstract does not name them. Not evaluated here beyond the abstract.

**nuFormer (verified identity).** arXiv:2507.23267, "Your Spending Needs Attention: Modeling Financial Habits with Transformers", 12 authors, v1 31 Jul 2025, v2 10 Aug 2026, cs.IR, CC-BY-4.0. Note the name: NVIDIA's blog calls it "Nubank's NuFormer"; the paper itself denotes the approach **"nuFormer"**. Verified abstract claims: it investigates "transformer-based representation learning models for transaction data"; the motivation is that "in many financial institutions, most production models follow traditional machine learning (ML) approaches by converting unstructured data into manually engineered tabular features"; it proposes "a new method enabling the use of SSL with transaction data by adapting transformer-based models to handle both textual and structured attributes"; it includes "an end-to-end fine-tuning method that integrates user embeddings with existing tabular features"; and "our experiments demonstrate improvements for large-scale recommendation problems at Nubank. Notably, these gains are achieved solely through enhanced representation learning rather than incorporating new data sources." **Two things matter here for the honest reading: nuFormer targets ONE downstream domain (recommendation), and it does not replace the tabular model — it integrates user embeddings with existing tabular features.** It is the same architectural posture as NVIDIA's blueprint, published a year earlier, by a bank.

**Also verified, at headline level only:** Stripe's Sessions 2025 announcement (7 May 2025) describes "the world's first AI foundation model for payments"; the company's own accompanying description states a model "trained on tens of billions of transactions, with unsupervised learning to generate nuanced embeddings", used with a classifier (class 2/4). No paper, no task definitions, no metrics. The "world's first" claim is **contradicted by the published record** given Skalski et al. (November 2023) and nuFormer (July 2025). Mastercard's 2026 news story describes a generative model using anonymised transaction data, framed as an insights engine rather than a chatbot and described as a **large tabular model** trained on structured data such as transactions, fraud patterns and chargebacks, with early testing claimed to distinguish legitimate purchases from suspicious activity (class 2, plus class-4 press corroboration). No paper, no metrics, and — importantly for this guide — **no sequence claim**.

### 6.5 The NVIDIA Blueprint — the One Open Artefact

**Verified identity.** "Build Your Own Transaction Foundation Model for Financial Intelligence", NVIDIA Technical Blog, dated **16 Jun 2026**, author **Benjamin Wu**, with the repository at `github.com/NVIDIA-AI-Blueprints/transaction-foundation-model`.

**Verified specifics.** A custom domain tokenizer converts each transaction into **roughly 12 semantic tokens with a 6,251-symbol vocabulary**. A compact **Llama decoder of ~29M parameters** is pre-trained from scratch using **NVIDIA NeMo AutoModel** on the **IBM TabFormer dataset of 24.4M synthetic card transactions**. **Last-token pooling** extracts **512-dimensional embeddings per user history**, compressed to **64 dimensions with PCA**. GPU data processing uses **cuDF and cuML**. The downstream step is "augmenting a downstream fraud classifier with embeddings", and the reported result is an **Average Precision lift of 41.76% over a strong baseline** on the temporal test split — which the blog's own narrative elsewhere describes as "a near-50% lift in Average Precision... over a strong XGBoost baseline on the IBM TabFormer fraud dataset".

**Five things a careful reader must hold at once:**

1. **It trains on synthetic data.** IBM TabFormer is a synthetically generated transaction dataset. Any absolute performance claim is a claim about synthetic data.
2. **The TFM does not replace the tabular model — it augments it.** The classifier is XGBoost, and the TFM supplies features. NVIDIA's own summary bullet says "combining raw tabular features with foundation-model embeddings in an XGBoost classifier". This is the single most important structural fact in the section: **the only openly reproducible artefact in this class uses the sequence model as a feature source for the incumbent gradient-boosted model, not as its replacement.**
3. **The blog's two headline numbers differ.** The AI-generated summary states 41.76%; the body text states "a near-50% lift". Both are the same result described at different precision, or the page is internally inconsistent; either way, a reader quoting "nearly 50%" is quoting the more flattering of the vendor's own two renderings.
4. **The baseline is characterised, not named.** "A strong baseline" and "a strong XGBoost baseline" — the blog does not publish the baseline's feature set, tuning, or metric value, so the lift cannot be contextualised.
5. **The model is small and the data synthetic, and the vendor says so.** That honesty is creditable, and it also bounds what the artefact proves.

**Evidence classification:** the blog is a **class-2 vendor publication**; its performance number is a **class-3 vendor benchmark on synthetic data**. The repository is a genuine open artefact — the only one found in this pass whose claims a bank could actually attempt to reproduce.

### 6.6 The Baseline Question — the Guide's Most Important Finding

Set out the comparisons that the published record actually contains:

| Claim | What it is compared against | Is the comparator a well-tuned tabular model? | Public? | Reproducible by a third party? |
|---|---|---|---|---|
| TREASURE +111% abnormal-behaviour detection | "the currently deployed system" — Visa's own production models, on an undisclosed metric | **Unknown** — the paper does not describe the production system's architecture or tuning | No | No |
| TREASURE +104% recommendation | Supervised embeddings trained specifically for the recommendation task | No — embeddings vs embeddings | No | No |
| TREASURE Transformer > GRU | TREASURE retrained with GRU/LSTM backbones | No — same model, different backbone | No | No |
| TransactionGPT anomaly detection | "a competitive production model" | **Unknown** — described only as production | No | No |
| PRAGMA credit/fraud/LTV | Unnamed baselines, own corpus | **Unknown** from the abstract | No | No |
| nuFormer recommendation | Existing tabular production features at Nubank, with embeddings added | **Not replaced — augmented** | No | No |
| Skalski et al. fraud detection | Public-dataset self-supervised methods; hold-out issuers | No for the abstract's headline claim | Partly (public datasets named; 180-bank corpus is not) | Partially |
| NVIDIA +41.76% AP (or "near-50%") | "a strong XGBoost baseline" on synthetic data, with embeddings **added to** that XGBoost | The comparator is XGBoost — but on **synthetic** data, and the TFM is a **feature source inside it** | Yes (dataset and repo) | Yes, in principle |

**The finding, stated plainly: in no work found in this pass does a transaction foundation model beat a well-tuned, publicly benchmarked gradient-boosted-tree incumbent on real transaction data, on a public dataset, with the comparator fully specified.** The comparisons that exist are of four kinds:

1. **Against the vendor's own prior production system** (TREASURE, TransactionGPT) — real, but internal to the vendor, on an undisclosed metric, on non-public data.
2. **Against a task-specific supervised model of the same family** (TREASURE and nuFormer recommendation) — a genuinely fair comparison, and the best-defended result in the field, but again internal.
3. **Against the model's own architecture variants** (TREASURE's GRU/LSTM ablation) — informative about architecture, silent about whether the class beats the incumbent.
4. **On synthetic data, with the sequence model augmenting the tabular model** (NVIDIA) — the only open, reproducible result, and it makes the *opposite* of the replacement argument.

**The field's benchmark practices permit exactly one honest conclusion for a buying bank: the published evidence establishes that this model class is competitive with, and in vendor-internal evaluations better than, the vendor's own incumbents — and it does not establish that it beats your incumbent on your data.** A bank's evaluation must therefore be internal and champion-challenger, or the decision rests on someone else's benchmark. This is developed into the anti-patterns in §13 and the evaluation design in §12.6.

### 6.7 What the Results Do and Do Not Establish

**Established (with the caveats stated):**
- Pre-training transformer (and, in earlier work, recurrent) sequence models on large transaction corpora is technically feasible, and the published systems improve multi-task efficiency over training one model per task (class 1, TREASURE §4.2).
- Predictions of next-transaction attributes are learnable at meaningful precision on a network-scale corpus — TREASURE reports Prec@1 of 0.1421 for merchant, 0.5634 for country, 0.1892 for city and 0.4335 for category, with the merchant figure meaningfully above its own variants (class 1, §4.2, Table 1).
- High-cardinality categorical prediction at 100M+ category scale is tractable with contrastive loss and shared negative sampling (class 1, §3.5).
- Embeddings from a pre-trained transaction model outperform embeddings trained supervised for a specific recommendation task, by the publisher's own measurement (class 1, TREASURE §4.5; class 1, nuFormer abstract).
- Cross-institution and out-of-domain transfer is demonstrable in at least one published setting (class 1, Skalski et al.).
- The whole approach can be reproduced at teaching scale on synthetic data, and used to augment an XGBoost classifier (class 2/3, NVIDIA).

**Not established:**
- That any of these models beats a well-tuned boosted-tree incumbent on real transaction data on a public, third-party benchmark. **No such comparison was found in this pass.**
- That embeddings or models transfer usefully from the publisher's population to a different bank's population. Skalski et al. shows transfer across issuers within a shared consortium corpus; nothing shows transfer from a network operator to an unrelated bank.
- That the reported improvements survive a champion-challenger test with label delay respected (see §10.2).
- That a TFM can satisfy an adverse-action or automated-decision explanation requirement in practice (see §9.2–9.3).
- That the training data for any of these systems was lawfully available for the purpose used (see §3.5) — no source in this pass addresses this.
- That the models are robust to adversarial adaptation (see §10.4; cross-ref [adversarial_ml_attacks_guide.md](../technology/adversarial_ml_attacks_guide.md)).

**Verification note for §6.** Verified at arXiv abstract/listing pages: identifiers, titles, full author lists, submission and revision dates, subject classifications and licences for arXiv:2511.19693, 2511.08939, 2604.08649, 2507.23267 and 2401.01641. Verified at the TREASURE HTML v3 body: the KDD 2026 venue and DOI, the Visa Research affiliations, the ~6B/30M/26-month dataset description and split, the 5-static/16-dynamic attribute structure, attribute-name withholding, the training configuration, the Prec@1/sMAPE metric definitions, the RI definition and the 2.1171/1.9606/1.8768/0.1768/1.4427/1.3979 figures, the 104% recommender figure, the 90% figure in §1, the 111% headline, the 3GB/6GB memory result under PyTorch 2.6, the 512-event cap, the "actively working toward full production deployment" statement, and the Skalski et al. bibliography entry with pages 141–149. Verified at the NVIDIA blog: date, author, all five technical specifics, the repository URL, the "augmenting a downstream fraud classifier" step, and both the 41.76% and "near-50%" renderings. Not read in full during this pass, and therefore flagged in §15: the TransactionGPT, PRAGMA, nuFormer and Skalski et al. paper **bodies**; the F-Industries page body; the four named open-finance and US privacy instruments.

---

## 7. The Implementation and Vendor Landscape

### 7.1 Four Layers of Supply

What exists to be obtained falls into four layers, and only one of them is a product a bank can buy as a TFM:

1. **Network operators' own models** — internal systems, not for sale.
2. **Fintech platforms' models** — largely internal, occasionally surfaced as an API or productised capability.
3. **Fraud, fincrime and decisioning vendors** — platforms that may embed sequence modelling, sold as a platform rather than a model. **The vendor rows are owned by [regtech_guide.md](regtech_guide.md); this guide cross-refers rather than re-assembling that landscape.**
4. **Infrastructure vendors and open artefacts** — the tooling and blueprints that let a bank build its own.

### 7.2 Network Operators' Own Models

| Offering | What it actually is | Does the vendor describe it as pre-trained and transferable? | Evidence status |
|---|---|---|---|
| Visa TREASURE | A transformer encoder pre-trained on ~6B Visa transactions with next-transaction and network-signal objectives; evaluated standalone and as an embedding provider | **Yes** — "multipurpose transformer-based foundation model"; transfer demonstrated in-house | Class 1 paper; results class 3 (internal benchmark); no public model, no public data, not deployed at time of writing |
| Visa TransactionGPT | A 3D-Transformer foundation model trained on billion-scale transactions for anomaly detection and future-transaction generation | **Yes** — "a foundation model for consumer transaction data" | Class 1 technical report; internally benchmarked |
| Mastercard large tabular model | A structured-data neural model on anonymised transaction data, positioned as an insights engine for commerce rather than a chatbot | **No** — Mastercard describes it as a **large tabular model**, explicitly contrasted with LLM-based generative systems. It is a generative-AI model on tabular data, **not** described as a pre-trained transferable transaction-sequence model | Class 2 vendor story + class 4 press; no paper, no metrics |

The architecture decision this implies is uncomfortable and worth stating: **the two largest networks do not agree on the shape of the answer.** One publishes sequence-transformer foundation models; the other publicly frames its effort as a large *tabular* model. A bank citing "the networks are doing it" as justification for building a sequence model is citing half the network landscape.

### 7.3 Platform and Financial-Crime Vendors

The relevant vendor population — fraud platforms, transaction-monitoring systems, screening and case-management tools — is catalogued in [regtech_guide.md](regtech_guide.md) (global platforms, Singapore regtechs, and the AML/KYC stack) and the production fraud discipline is owned by [financial_fraud_detection_at_scale_guide.md](financial_fraud_detection_at_scale_guide.md).

What this guide adds, and only this: **when a fraud or fincrime vendor claims "foundation model" capability, the question to ask is the one from hazard (c) — is the vendor describing a model it pre-trained on unlabelled transaction sequences and then transferred, or a supervised model with sequence-derived features?** In this pass, **no fraud or financial-crime platform vendor was identified that publishes a TFM paper, a pre-training corpus description, or a transfer evaluation.** That is an absence of evidence in this pass's searches; see §15. It is also the correct default posture: a platform vendor applying ML to transaction data is doing something valuable and established, and calling it a foundation model does not change what it is.

### 7.4 Infrastructure Vendors and Open Artefacts

| Artefact | What it is | Availability | Evidence status |
|---|---|---|---|
| NVIDIA Build-Your-Own-Transaction-Foundation-Model developer example | End-to-end pipeline: cuDF/cuML tokenization → ~29M Llama decoder pre-trained with NeMo AutoModel → 512-d embeddings → 64-d PCA → XGBoost augmentation | **Open** — GitHub repository and a `build.nvidia.com` example; described as having independently swappable components (tokenizer, HuggingFace-compatible decoder, downstream task) | Class 2 publication + class 3 benchmark on synthetic data |
| IBM TabFormer dataset | 24.4M synthetic card transactions, used as the blueprint's pre-training corpus | Public | Synthetic — not a substitute for real transaction data |
| NeMo AutoModel / CUDA-X (cuDF, cuML) | Pre-training library and GPU data-processing libraries used in the blueprint | Commercial/open tooling | Vendor documentation |
| PRAGMA-style linear-probe recipe | The cheapest published adaptation pattern: frozen embeddings + linear model, then light fine-tuning | Method published; no model released | Class 1 method description |

**This layer is where the genuine build option lives.** A bank that wants to experiment can today, in principle, reproduce NVIDIA's pipeline on its own data — swapping the synthetic corpus for its own histories, the tokenizer for one matching its schema, and the downstream head for its incumbent's task. That is a concrete, bounded experiment rather than a research programme, and it is the option §8 argues for first.

### 7.5 The Availability Reality

Summarising the supply side without euphemism:

- **No TFM is available off the shelf as a model.** No network operator or fintech in this pass offers its transaction foundation model as a licensable artefact with weights a bank could fine-tune.
- **No TFM has a published, obtainable pre-training corpus.** Every real-data corpus in the published record is proprietary.
- **The only reproducible artefact is a synthetic-data blueprint that augments, rather than replaces, an XGBoost model.**
- **What is genuinely for sale is tooling and platforms**, plus — at most — a vendor-hosted embedding API whose provenance, training data and evaluation a customer cannot inspect.
- **"Founder-led" claims of a TFM product should be verified against hazard (c)**: does the vendor state that the model was pre-trained on unlabelled transaction sequences and transferred, or is "foundation model" being used to mean "our ML is modern"?

**Verification note for §7.** The NVIDIA artefact's components, repository and swappability claim are from the verified 16 Jun 2026 blog; the blog also links the `build.nvidia.com` example and the IBM TabFormer repository, both of which are named in the verified page text. Mastercard's "large tabular model" framing and insights-engine positioning are from the verified Mastercard news story and corroborating press. Visa's "100+ machine learning models" and the scale figures are quoted from TREASURE's introduction, which cites Visa's own corporate documents. **Flagged in §15:** no evidence was located during this pass that any named fraud/financial-crime platform vendor publishes a TFM; that is an absence of evidence and is recorded as such rather than asserted as a fact about the market.

---

## 8. Build Versus Buy Versus Extend

### 8.1 The Three Options, Stated Honestly

Most treatments of this decision offer two options. There are three, and the third is the one the published evidence actually supports:

1. **Build** a transaction foundation model (pre-train an encoder on the bank's own transaction sequences, adapt it per task).
2. **Buy** — acquire a platform, an embedding API, or a vendor model and integrate it into decisioning.
3. **Extend the incumbent** — keep the existing feature-engineering plus gradient-boosted-tree pipeline in production and add sequence-derived features or embeddings to it as an additional input channel. **This is what NVIDIA's example does, what nuFormer does, and what neither presents as a replacement.**

Option 3 is not a compromise; it is the option that preserves the incumbent's explainability, latency profile and validation history while testing whether sequence structure carries incremental signal. Any of the three can be right, but option 3 is the correct *default*, and the burden of proof sits with the other two.

### 8.2 What Building Requires

- **Data rights, in writing.** The lawful basis and purpose analysis for pre-training on the bank's full transaction history, covering the collection jurisdiction, the training jurisdiction and the deployment jurisdiction, plus a position on whether embeddings derived from that data are themselves personal data (§3.5, §9.5). This is the gating item and it is a legal deliverable, not a data-engineering one.
- **Compute.** Material but not exotic: TREASURE's model is a 3-layer decoder with hidden dimension 256 and a 512-event context (class 1, §3.2, §4.1); NVIDIA's is 29M parameters (class 2). A bank does not need frontier-scale infrastructure — it needs a reproducible training environment and a GPU data-processing path (NVIDIA uses cuDF and cuML).
- **Sequence-engineering skill.** Tokenization design, vocabulary maintenance, masking and negative-sampling strategy, and the handling of unseen categories. The published record shows that high-cardinality handling is where the real difficulty sits (§4.4) and that the tokenizer is the least portable component (§4.1). This skill set is scarcer in most banks than tabular ML skill.
- **Evaluation capability.** A champion-challenger harness that respects label delay, handles cost asymmetry, and can detect a small increment against a strong incumbent (§10). Without this, the exercise produces a number nobody can act on.
- **Ongoing retraining and lifecycle.** Vocabulary drift (new merchants, new MCCs), distribution drift, adversarial adaptation, and a retraining cadence with validation gates. Cross-ref the repository's MLOps material at [../technology/mlops_lifecycle_frameworks_guide.md](../technology/mlops_lifecycle_frameworks_guide.md) for the machinery rather than re-deriving it here.
- **Model-risk documentation from day one.** If the model influences a credit or fraud decision, the validation function will want the pre-training corpus described, the tokenizer specified, the adaptation recipe documented and the performance attribution reproducible (§9.1).

### 8.3 What Buying Gives and Costs

**What buying gives:** time-to-capability, an evaluation that someone else has already run (on someone else's data), support and maintenance, and a liability counterparty. Where the vendor already processes the customer's transactions, the vendor may also hold data the bank does not have — which is the only genuinely structural advantage in the buy path, and it is a data advantage, not a modelling one.

**What buying costs:**
- **Provenance opacity.** A vendor-hosted embedding API gives the bank a vector without the corpus, the tokenizer or the evaluation. That is precisely the input a validation function needs (§9.1) and precisely what is usually not supplied.
- **Explainability hand-off.** If the vendor's embedding feeds a decision the bank makes, the bank still owns the adverse-action obligation (Regulation B § 1002.9 in the US; GDPR Article 22 in the EU). The vendor cannot discharge it.
- **Concentration and exit risk.** A model trained on the vendor's population is a dependency, and switching costs rise with integration depth.
- **Distribution mismatch.** The vendor's population is not yours. Every published transfer claim in this class is intra-organisation (§6.6), which means the vendor's model meets your distribution as an unmeasured out-of-distribution transfer until you test it.

### 8.4 The Third Option — Extending the Incumbent

The recipe, assembled from what the sources actually describe:

1. Keep the production gradient-boosted model exactly as it is. It is the champion. Its monitoring, back-testing and documentation continue unchanged.
2. Pre-train (or obtain) a sequence encoder on the bank's own histories — self-supervised, no labels required, which sidesteps the label-scarcity problem entirely for this phase.
3. Extract a per-customer embedding at the decision point, under the same latency and point-in-time discipline as the incumbent's features. **This is the hard engineering part and the sources do not solve it for you:** a per-history embedding must be computable at decision time from data available at decision time, or the increment is look-ahead leakage.
4. Feed the embedding to the incumbent as an additional feature channel alongside the existing tabular features — NVIDIA's "combining raw tabular features with foundation-model embeddings in an XGBoost classifier" (class 2, 16 Jun 2026); nuFormer's "end-to-end fine-tuning method that integrates user embeddings with existing tabular features" (class 1, abstract).
5. Measure the increment against the champion on the champion's own metric, on matured labels, with the champion fully tuned. If the increment is not there, stop — and that is a successful outcome, because it was cheap.
6. Only if the increment survives, consider whether the encoder should also serve as a standalone scorer — which is a separate decision with separate explainability consequences.

**Why this is the defensible default:** it holds the incumbent accountable, it produces a decision either way, it preserves the ability to explain a decision (the tree still attributes), and it matches the only two published production-grade examples in the field (NVIDIA's and Nubank's), both of which stop at augmentation.

### 8.5 Illustrative Cost Structures

> **These figures are illustrative and invented for this guide's decision framework. They are not benchmarks, not vendor quotes, and not derived from any source. They are order-of-magnitude shapes for a mid-size retail bank, presented in clearly-labelled units so they can be scaled or discarded.**

| Cost element | Build | Buy (embedding API or platform module) | Extend (embedding as a feature channel) |
|---|---|---|---|
| Legal / data-rights analysis | 1–2 FTE-months, external counsel likely | Light — vendor DPA plus an internal purpose review | 1 FTE-month — same analysis, scoped to internal training |
| Data engineering (sequence assembly, point-in-time correctness) | 3–6 FTE-months | 1–2 FTE-months (integration only) | 2–4 FTE-months — the hardest part of this path |
| Pre-training compute | Modest by modern standards — a small decoder on a few hundred GPU-hours, illustrative | Included in licence | Same as build, or reused from a bought encoder |
| Sequence-ML skill | 2–3 senior ML engineers, sustained | Not required in-house | 1–2 engineers, sustained |
| Evaluation harness | 2–3 FTE-months, plus ongoing | 1–2 FTE-months | 2–3 FTE-months — you must build this regardless |
| Model-risk documentation and validation | 2–4 FTE-months first time | 2–4 FTE-months first time (harder without provenance) | 2–4 FTE-months first time |
| Ongoing run cost | Retraining cadence, vocabulary maintenance, drift monitoring | Licence plus integration maintenance | Retraining cadence for the encoder plus existing incumbent cost |
| Elapsed time to a defensible answer | 9–18 months | 3–9 months to integration, longer to a defensible answer | 6–12 months |

### 8.6 Where the Real Cost Sits

The GPU line is the smallest line in every column. **The real cost is data assembly and evaluation.** Specifically:

- **Point-in-time correctness.** Rebuilding historical per-customer sequences as they existed at each decision moment, without leakage, is a data-engineering problem that dominates the project and is invisible in vendor demos.
- **Label maturation.** Waiting for dispute and default outcomes to settle before the comparison can be made converts a modelling timeline into a calendar timeline (§10.2).
- **Validation and documentation.** A model that feeds a credit or fraud decision carries the same documentation and validation burden as any other model in the inventory — and for a pre-trained encoder with an opaque corpus, more.
- **Maintaining the tokenizer and vocabulary.** TREASURE withholds its categorical mappings for sensitivity reasons (class 1, §3.3); a bank cannot withhold its own from its own maintainers, so it inherits an artefact with a lifecycle nobody has prior experience of.

A bank that budgets for GPUs and not for these four has budgeted for the wrong project.

**Verification note for §8.** The augmentation recipes in 8.4 are quoted or paraphrased from the verified NVIDIA blog (16 Jun 2026) and the verified nuFormer abstract. The architectural scale figures used to bound compute expectations (3 layers, hidden 256, 512-event context, 29M parameters) are from the verified TREASURE HTML body and NVIDIA blog respectively. **Every figure in §8.5 is illustrative and invented for this guide** — none is sourced, and the table label says so at the point of use.

---

## 9. Regulated-Banking Constraints

> The regulatory material specific to AI in banking — the supervisory expectations, the model-risk integration, the EU AI Act machinery, the governance frameworks — is owned by [ai_genai_banking_compliance_guide.md](ai_genai_banking_compliance_guide.md) and by the repository's model-risk content. This section addresses only the constraints that bind **this model class** specifically, and cross-refers rather than restates.

### 9.1 What a Validation Function Would Demand

A model validation function examining a TFM would apply the existing model-risk framework — the SR 11-7 style guidance that is covered elsewhere in this repository and should be cross-referenced there rather than restated here — and would immediately hit four properties of the class that the framework was not written for:

1. **The corpus is the model.** A pre-trained encoder's behaviour is a function of a corpus that, in every published case, is non-public and often undescribed (TREASURE withholds attribute names; the exact difficulty is set out in §6.1). Validation normally asks "what data was this developed on, and is it representative?" — for the leading systems the honest answer is "we cannot tell you."
2. **The vocabulary is an unversioned dependency.** New merchants, new MCCs, new countries arrive continuously. The model's input space changes without a retrain, which is a form of silent drift that standard monitoring does not capture.
3. **Performance is a ratio to an undisclosed baseline.** The published headline results are RI-style ratios against internal production systems (§6.2). A validator cannot independently assess a number that has neither absolute value nor a specified comparator.
4. **Explainability is not additive.** A gradient-boosted incumbent explains itself; a sequence encoder requires a post-hoc technique bolted on, and the bolted-on explanation is a different object from the model's actual computation (§9.2).

What a validation function would therefore reasonably demand: a documented pre-training corpus profile and its representativeness relative to the deployed population; the tokenizer specification and its drift-monitoring plan; the adaptation recipe with a frozen reference implementation; out-of-time and out-of-population performance with confidence intervals; a stated champion-challenger protocol; and an explanation mechanism that has itself been validated. **Note that this is a demand list, not a description of current practice** — no source in this pass describes a validation function that has completed an SR 11-7-style review of a transaction foundation model.

### 9.2 Explainability and the Adverse-Action Problem

This is a structural constraint, not a caveat, and it is where the class is most awkward.

**What the regimes require.** In the United States, a creditor taking adverse action on a credit application must notify the applicant and provide a statement of specific reasons — Regulation B, 12 CFR § 1002.9, verified at consumerfinance.gov: the creditor must notify the applicant of action taken and, where credit is denied, disclose the specific reasons for the denial or the applicant's right to a statement of specific reasons, with the section addressing timing, the ECOA notice, and the content of the statement. In the EU, GDPR **Article 22** (verified at reproduced text) gives a data subject the right not to be subject to a decision based **solely** on automated processing that produces legal effects or similarly significantly affects them, subject to exceptions including contractual necessity — and where those exceptions apply, the controller must implement safeguards including "at least the right to obtain human intervention on the part of the controller, to express his or her point of view and to contest the decision." Recital 71 is the profiling safeguard the Article points to. In Singapore, the **MAS FEAT principles** (Fairness, Ethics, Accountability, Transparency) set supervisory expectations for AI-driven decisions in financial services, including explainability expectations, verified here via secondary compilations rather than the primary MAS page (§3.5, §15).

**Why a large pre-trained sequence model is structurally awkward here.** The obligation is not "produce *an* explanation" — it is "produce *the* reasons", specific and contestable, and where the decision significantly affects the person, a contestable mechanism. A sequence model's decision is a function of hundreds of attended events through millions of parameters. Three specific mismatches:

- **The explanation is a different object from the decision.** Post-hoc attribution over an embedding explains a surrogate's behaviour on a proxy input; it is not the model's own computation. Producing it and calling it "the reason" is, at minimum, a documentation risk.
- **Reasons must be consumer-legible.** "Your embedding's cosine similarity to a fraud-cluster centroid exceeded a threshold" satisfies no adverse-action standard in any jurisdiction reviewed here. The output has to map back to something a person can understand and dispute.
- **The contest mechanism has to change the outcome.** GDPR Article 22's safeguards include human intervention and the right to contest. If the human reviewer cannot interrogate the model — because the reviewer has a vector, not reasons — the safeguard is procedural rather than substantive.

**The genuine tension worth naming:** the tasks where sequence structure is most valuable (fine-grained behavioural anomaly detection) are adjacent to the decisions where explainability is most legally demanding (credit, payment blocking). A bank can sensibly deploy a sequence model for *ranking* and *prioritising* while keeping the actual declination on an explainable model — and that is a defensible architecture, not a cop-out.

### 9.3 Proposed Resolutions, and Whether Any Is Demonstrated

| Approach | How it is supposed to work | Demonstrated in this class? |
|---|---|---|
| Post-hoc attribution (SHAP or similar) on input features and on embeddings | Attribute the score to features the reviewer recognises | **Not demonstrated for a TFM in any source found.** SHAP on a gradient-boosted incumbent is standard practice; SHAP over a 512-event transformer embedding is not evidenced here |
| Surrogate models | Train an interpretable model to approximate the sequence model locally, and explain the surrogate | **Not demonstrated.** The surrogate's fidelity to the original is the open question and no source in this pass measures it |
| Attention inspection | Read the attention weights to see which historical events drove the output | **Not demonstrated as an explanation mechanism.** TREASURE uses attention and visualises embeddings (class 1, §3.2, §4.6), but neither is presented as an adverse-action explanation, and attention-as-explanation is not established as legally sufficient anywhere in this pass |
| Retrieval of similar historical cases | Show the customer the closest historical cases that resemble theirs — "here are comparable accounts and how they behaved" | **Not demonstrated in this class, and conceptually the most promising:** it produces a legible artefact (cases) rather than an attribution, and it aligns with the embedding structure the models already learn. No implementation or validation found |
| Human-in-the-loop review | Route automated adverse decisions to a human with discretion to reverse | **Required in substance by GDPR Article 22(3) where its exceptions apply**, and standard in bank fraud operations — but its adequacy depends on the human having reasons to work with, which loops back to the rows above |
| Using the model only for ranking, with the declination on an explainable model | The sequence model orders the queue; an explainable model or a rule makes the actual decision | **This is the resolution that requires no new evidence** — it is a systems design that avoids the problem rather than solving it, and it is the resolution this guide can recommend on the evidence available |

**Honest summary: the resolution of the explainability problem for transaction foundation models is aspirational, not demonstrated.** The field proposes post-hoc attribution and attention inspection; no source found in this pass shows either meeting an adverse-action or Article 22 standard in practice. The only demonstrated-safe pattern is architectural avoidance — keep the sequence model in the ranking and prioritisation layer, and keep the declination on a model that can state specific reasons.

### 9.4 The AI-Regulation Position for Credit and Fraud

**The EU AI Act.** Verified at Annex III via artificialintelligenceact.eu: high-risk classification under Article 6(2) attaches to the systems listed in Annex III, and Annex III point 5(b) covers "AI systems intended to be used to evaluate the creditworthiness of natural persons or establish their credit score, **with the exception of AI systems used for the purpose of detecting financial fraud**."

That single sentence is the most consequential regulatory fact for this model class, and it cuts two ways:

- **Fraud detection is expressly excluded** from the creditworthiness high-risk entry. A TFM used for financial-fraud detection is not swept into Annex III by point 5(b) on its face.
- **The same model used for credit scoring is in scope.** PRAGMA names credit scoring among its downstream tasks (class 1, abstract); TREASURE and nuFormer target recommendation and detection rather than credit. **One model, two tasks, two regulatory postures** — which means the classification question is about the *use*, not the *model*, and a bank must classify each deployment separately.
- Exclusion from Annex III is not exemption from the regulation: other obligations (transparency, and the general prohibitions and other provisions) can still apply, and the AI Act's interaction with existing financial-services model-risk and consumer-protection rules is a question owned by [ai_genai_banking_compliance_guide.md](ai_genai_banking_compliance_guide.md). The Article 6(3) filtering mechanism and the rest of the Act's architecture are not restated here and were not all verified at primary text in this pass.

**The United States.** Adverse-action reasoning obligations under ECOA/Regulation B apply where the model contributes to a credit decision, verified at § 1002.9. Whether — and how — those obligations extend to a fraud-related declination is jurisdiction- and product-specific and is not resolved by this guide. FCRA's role in credit decisions is flagged as unverified in this pass (§15).

**Singapore.** MAS FEAT's fairness, ethics, accountability and transparency expectations apply to AI-driven decisions in the financial sector; the principles are non-binding, and the Veritas initiative supplies assessment tooling. Cross-ref [ai_genai_banking_compliance_guide.md](ai_genai_banking_compliance_guide.md) for how these integrate into supervisory practice.

### 9.5 Data Protection and the Training-Data Question

Three questions, none of which this guide can close:

1. **Was the pre-training use compatible with the purpose for which the data was collected?** GDPR Article 5(1)(b) (verified) requires specified, explicit and legitimate purposes and prohibits incompatible further processing, with a research/statistical carve-out under Article 89(1). For a network operator training on its own network traffic, the analysis differs from a bank training on customer account data held under a banking relationship. Both need it documented.
2. **Are embeddings personal data?** A vector derived from a customer's transaction history and used to make a decision about that customer is, on any reading, personal data in that use. Whether an *anonymised* or aggregated embedding serves as a lawful basis for training or sharing is unresolved here for lack of a verified source, and a bank should treat it as personal data until counsel says otherwise.
3. **What happens on erasure or objection?** GDPR Article 21's right to object and Article 17's erasure interact badly with a pre-trained model that has absorbed the data into its weights. Retraining from scratch on request is not operationally plausible at scale; suppressing the customer at inference while leaving the weights untouched is the pragmatic answer, and it is not a settled legal position. **No source in this pass addresses deletion-from-a-transaction-foundation-model, and that gap is recorded in §15.**

The practical governance instruction: treat the pre-training corpus as a data asset requiring the same disposal, retention and access discipline as the raw data, and record the purpose analysis before the first training run — not after.

### 9.6 Audit and Reproducibility

- **Reproducibility of a training run** requires the corpus, the tokenizer, the vocabulary version, the objective weights and the random seeds. TREASURE withholds the attribute names and the OOV mechanism for sensitivity (class 1, §3.3); a bank cannot withhold these from its own auditors.
- **Auditability of decisions** requires the ability to reconstruct, for a specific historical decision, the input sequence, the embedding, the model version and the threshold in force. This is a data-retention design problem, and a long-history sequence model makes it heavier than a tabular model because the reconstructable input is large.
- **Change control** must cover the vocabulary and the tokenizer as versioned artefacts, not just the weights. A vocabulary update is a model change.
- **Third-party models** raise the same questions with less access: an embedding API used in a decision the bank makes puts the bank's audit obligation on an artefact the bank cannot inspect, which is a control gap a model-risk function should record explicitly.

### 9.7 Hard Constraints Versus Supervisory Expectation

| Requirement | Status |
|---|---|
| Adverse-action notice with specific reasons for a credit denial (US, Regulation B § 1002.9) | **HARD CONSTRAINT** — but its application to a given model depends on the product and jurisdiction |
| No decision based solely on automated processing where the data subject has not been given the Article 22(2) exception plus Article 22(3) safeguards (EU) | **HARD CONSTRAINT** where Article 22 applies |
| Creditworthiness/credit-score AI classified high-risk (EU AI Act Annex III 5(b)); financial-fraud detection expressly excepted | **HARD CONSTRAINT** in classification terms — an obligation to classify correctly and comply accordingly |
| Purpose limitation on pre-training (GDPR Article 5(1)(b)) | **HARD CONSTRAINT** — but whether a given use is compatible is an assessment, not a lookup |
| Explanation of an automated decision to the person affected | **HARD CONSTRAINT** in the credit context; **supervisory expectation** in the fraud context in the jurisdictions reviewed |
| Fairness, ethics, accountability, transparency for AI-driven decisions (MAS FEAT) | **SUPERVISORY EXPECTATION** — the principles are non-binding |
| Model-risk documentation and validation (SR 11-7 style framework) | **SUPERVISORY EXPECTATION** that functions as a hard constraint in practice for a regulated bank |
| Vocabulary and tokenizer change control, decision reconstruction, corpus profiling | **GOOD PRACTICE** — not named in any instrument reviewed here, but implied by auditability obligations |

**Verification note for §9.** Verified at reproduced primary text: GDPR Article 22 (including the Article 22(2) exceptions, the Article 22(3) safeguards and the recital 71/72/91 pointers) and Article 5(1)(b), both at gdpr-info.eu. Verified at a regulator page: the adverse-action notification structure of Regulation B, 12 CFR § 1002.9, at consumerfinance.gov, including the statement-of-specific-reasons requirement. Verified at artificialintelligenceact.eu: Annex III point 5(b)'s creditworthiness entry and its express exception for "AI systems used for the purpose of detecting financial fraud". Verified via secondary compilations: MAS FEAT's 12 November 2018 publication date, non-binding character and four principles. **Flagged and moved to §15:** GDPR Article 21 and Article 17 summaries and the deletion-from-a-model question; Article 6 lawful-basis reasoning; FCRA's role; the AI Act's Article 6(3) filtering and non-Annex obligations; the entire PSD2/PSD3/FIDA summary. The model-risk guidance itself (SR 11-7 and its analogues) is referenced by name only, consistent with the repository convention that this material lives elsewhere.

---

## 10. The Evaluation Problem

> Techniques for building evaluation, monitoring and lifecycle machinery are owned by the repository's MLOps and applied-ML guides — see [../technology/mlops_lifecycle_frameworks_guide.md](../technology/mlops_lifecycle_frameworks_guide.md) and the AI/LLM guides under [../technology/ai_llm/](../technology/ai_llm/), e.g. [llm_evaluation_vs_validation_guide.md](../technology/ai_llm/llm_evaluation_vs_validation_guide.md). This section states *why this model class is hard to evaluate* and does not re-derive the tooling.

### 10.1 Why There Is No Shared Benchmark

There is no transaction-foundation-model equivalent of ImageNet or GLUE, and the reason is structural rather than an oversight:

- **The data cannot be shared.** Transaction histories are the most commercially sensitive and personally identifying dataset a financial institution holds. TREASURE withholds even its attribute *names* (class 1, §3.3). A shared benchmark would require a shared corpus, which no network operator or bank has an incentive to create.
- **The synthetic substitute is not the task.** The IBM TabFormer dataset used by NVIDIA's blueprint is synthetic and public; the blueprint's own result is therefore a result about synthetic data (class 3).
- **Task definitions are not standardised.** TREASURE measures abnormal-behaviour detection with an undisclosed in-house metric reported as a ratio (§6.2). Another institution's "detection" is a different construct. A metric you cannot name is a metric you cannot compare.
- **Public datasets in the literature are the exception.** Skalski et al. is notable precisely because it benchmarks on public datasets *and* pre-trains on a private 180-bank, 5.1-billion-transaction corpus (class 1) — and it can therefore report only the first half of its evidence to a third party.
- **Performance depends on the population.** A model tuned to one institution's merchant mix, geography and customer behaviour is not portable in a measurable way, which means a benchmark result would not transfer even if the benchmark existed.

**Consequence:** every meaningful evaluation of a TFM is internal, and every externally reported number is a benchmark on somebody else's data. That is not a temporary state of affairs in this field; it follows from the ownership structure of the data (§3.4).

### 10.2 Label Delay and Contamination

- **Delay.** A fraud determination matures through disputes and chargebacks over weeks to months. An evaluation run at day 14 sees a partially labelled outcome and will systematically over-credit whichever model flags the fastest-maturing cases. The discipline is to fix a maturation window, evaluate on labels matured to that horizon, and re-evaluate when the window rolls — and to state the window in the result.
- **Contamination.** Labels come from the incumbent's own decisions. Fraud the incumbent filters out is never investigated and never labelled, so a challenger is scored partly on the incumbent's blind spots rather than on fraud as it exists. This is a selection effect built into the label source, and it biases *against* a genuinely different model — the challenger's correct detections of incumbent-missed fraud cannot be counted because they were never labelled positive.
- **Feedback loops.** A model that changes which transactions get reviewed changes the future label distribution. Any evaluation that runs long enough to matter is also an intervention.
- **The REJECTED pattern to watch for:** a comparison run before labels mature, on the incumbent's label set, at a single operating point, reported as a single number.

### 10.3 Imbalance and Cost Asymmetry

At a fraud rate well below one percent, accuracy is meaningless and the operating point dominates everything. Two consequences for evaluating this class:

- **Average Precision and precision-recall curves, not ROC-AUC.** With extreme imbalance, ROC-AUC flatters models that do nothing at the operating point that matters; the precision-recall view exposes the trade-off the business actually faces. It is notable that NVIDIA reports an **Average Precision** lift (class 2/3) — the right metric family — while TREASURE reports a ratio against an undisclosed metric (§6.2), which is not interpretable in these terms.
- **The cost ratio is the missing parameter.** A false positive in fraud costs a customer friction and a contact-centre call; a false negative costs the loss. Neither is published by any source in this pass, and without the ratio, a model that improves recall at 3× the false-positive rate cannot be judged. **A vendor metric without a cost ratio is not a decision input.**

### 10.4 Drift and Adversarial Adaptation

- **Covariate drift.** Merchant mix, payment instrument mix, seasonality and macro conditions shift the input distribution continuously. A model pre-trained on a 2020–2022 window (TREASURE's corpus, class 1, §4.1) is a model of that period's behaviour as much as of behaviour in general.
- **Concept drift.** What constitutes abnormal behaviour changes as customer habits and fraud tactics evolve.
- **Adversarial adaptation.** Fraud is an actively adversarial problem: the population that generates the labels is learning the model. A sequence model that has learned a customer's routine is also, indirectly, a manual for imitating it. Offline metrics measured on historical attacks systematically overstate performance against future ones, and pre-training on a fixed corpus does not confer robustness to a target that moves. Cross-ref [../technology/adversarial_ml_attacks_guide.md](../technology/adversarial_ml_attacks_guide.md) for the adversarial-ML discipline; this guide's point is only that offline evaluation cannot see this effect at all.
- **Vocabulary drift.** The unversioned-input problem from §9.1 — new merchants and categories arrive without a retrain and silently change the input space.

### 10.5 The Offline-Versus-Online Gap

Offline metrics decide nothing; the online result decides everything. The specific gaps between them in this class:

| Offline | Online reality |
|---|---|
| Evaluation on a temporal test split | Interleaved traffic, partial exposure, interactions with other rules and models |
| Point-in-time features assembled retrospectively (risk of leakage) | Features computed under latency budget with whatever data is actually available |
| Fixed operating point chosen for the report | Threshold constrained by analyst capacity; the alert budget, not the model, sets the operating point |
| Labels matured for the evaluation | Labels arriving over months, with the model already learning from its own reviews |
| No customer reaction | Customer friction, complaints, dispute behaviour and regulatory attention |
| Model served from a research environment | Latency in the authorisation path, where milliseconds are a business metric |

TREASURE's own paper is the clearest illustration of the gap: it explicitly presents offline evaluation as "a significant milestone" and states the team is "actively working toward full production deployment" (class 1, §4.1). **The published state of the art in this class is a pre-deployment offline result. A buyer should read every published number in this field as offline by default unless the source says otherwise.**

### 10.6 Why a Vendor Metric Is Not Transferable Evidence

Assembling the reasoning into one statement, because it is the practical conclusion of the whole guide:

A vendor benchmark has **five properties** that prevent it from being evidence about *your* deployment, and the reader can check each: (1) it was measured on the vendor's data, not yours; (2) the comparator is the vendor's own prior system or an unspecified "strong baseline", not your tuned incumbent; (3) the metric may be undisclosed or defined differently from your operating measure; (4) the evaluation was offline, without your latency and alert-budget constraints; (5) it cannot be reproduced or audited from outside. **Every performance number reported in this guide's sources has at least three of those five properties.** The correct use of a vendor number is as a *signal that the experiment is worth running*, never as an input to a deployment decision.

**Verification note for §10.** TREASURE's corpus dates, metrics and deployment-status language are from the verified HTML body (§4.1). NVIDIA's use of Average Precision is from the verified 16 Jun 2026 blog, which defines it as "the area under the precision-recall curve". The IBM TabFormer dataset's synthetic character is stated in the verified NVIDIA blog text. The evaluation-technique cross-references were selected from the repository's existing content, and each cited path was confirmed to exist. The claims in 10.2–10.5 about label delay, contamination bias and the offline-online gap are the guide's own synthesis of standard fraud-analytics practice, consistent with [financial_fraud_detection_at_scale_guide.md](financial_fraud_detection_at_scale_guide.md), and are not attributed to the TFM sources — none of which discusses evaluation difficulty in these terms.

---

## 11. The Honest State of the Field

### 11.1 Documented, Claimed, Unresolved

**Documented (evidence class 1, verified at source):**
- Pre-training sequence models on very large real transaction corpora is feasible, and the published systems are architecturally small (TREASURE: 3-layer decoder, hidden 256; NVIDIA: 29M parameters).
- High-cardinality categorical prediction — merchant vocabularies in the hundreds of millions — is tractable using contrastive objectives with batch-shared negatives.
- Transformer backbones outperform recurrent backbones *within the same model and corpus* on next-transaction prediction and abnormal-behaviour detection (TREASURE §4.3).
- Pre-trained transaction embeddings can outperform supervised task-specific embeddings on a recommendation task (TREASURE §4.5; nuFormer).
- Cross-issuer and out-of-domain transfer of pre-trained transaction embeddings to card-fraud detection has been demonstrated in at least one published multicentric setting (Skalski et al.).
- A minimal version of the whole pipeline is reproducible on synthetic data at negligible scale (NVIDIA), and its demonstrated use is **augmenting**, not replacing, an XGBoost classifier.

**Claimed (evidence class 2, 3 or 4):**
- That a TFM improves fraud detection by 111% (Visa, class 3 in a class-1 paper, undisclosed metric, non-public baseline).
- That a transaction foundation model exists as a licensable product anywhere (no vendor source located).
- That TFMs benefit reconciliation, exception handling and correspondent banking (F-Industries, class 2, no evidence).
- That a "world's first" payments foundation model was announced in May 2025 (Stripe, class 2, contradicted by the 2023–2025 published record).
- That Mastercard's large tabular model improves fraud distinction (class 2/4, no metrics).
- That Plaid's sequential foundation model captures behavioural signals traditional models miss (class 2, no numbers published).

**Unresolved:**
- Whether any TFM beats a well-tuned boosted-tree incumbent on real data on a public benchmark. **No such comparison exists in this pass.**
- Whether a model pre-trained on one institution's population transfers usefully to an unrelated bank.
- Whether a pre-trained transaction embedding is a lawful basis-compatible use of the underlying personal data in each jurisdiction.
- Whether any explanation technique satisfies an adverse-action or Article 22 standard for a sequence model's decision.
- What deletion or objection means for data absorbed into pre-trained weights.
- How such a model behaves under adversarial adaptation over time.

### 11.2 The Strongest Genuine Argument For

State it as its proponents would, and it survives scrutiny:

**The sequence carries information the incumbent provably discards, and the discard is mechanical rather than incidental.** Every aggregation step in a feature pipeline is many-to-one: once "count in last 24 hours" exists, the order is gone. The published evidence shows that order and cadence are learnable — TREASURE's Prec@1 results and Plaid's two-consumer example are both statements that the same aggregates can correspond to different situations — and it shows that the resulting representations are reusable, since two independent production teams (NVIDIA, Nubank) and at least four research groups converged on using embeddings to feed an existing model rather than to replace it. Self-supervised pre-training also sidesteps the worst feature of this domain — sparse, delayed, adversarially contaminated labels — by requiring no labels at all for the pre-training phase. And the field has published its engineering constraints honestly: context caps, memory ceilings, tokenizer vocabularies, sample counts. **That is a real technical case, made with real numbers, and it does not depend on the 111%.**

### 11.3 The Strongest Genuine Argument Against

State it as a sceptical validator would, and it also survives scrutiny:

**The published evidence, read strictly, does not answer the question a buyer is asking.** Every headline performance number in this class is either a ratio to an undisclosed internal baseline on non-public data, a comparison against the model's own architectural variants, or a result on synthetic data — and the one artefact anyone can reproduce (NVIDIA's) uses the sequence model to *augment* the gradient-boosted incumbent, not to replace it. The strongest vendor-independent comparative evidence for a TFM beating a well-tuned tabular model on real data on a public benchmark **does not exist**. Meanwhile the costs are concentrated exactly where a regulated bank is weakest: an explainability mechanism that is proposed but not demonstrated; a training-data purpose question no source addresses; a tokenizer and vocabulary that are unversioned inputs; a deletion problem with no settled answer; and an evaluation problem that makes it impossible to know whether a modest offline lift will survive contact with label delay, the alert budget and a moving adversary. **The honest formulation is not "this does not work" — it is "this is unproven against the incumbent, and unproven is not the same as false, but it is also not a reason to migrate production credit or fraud decisioning."**

### 11.4 Watching Rather Than Building — the Conditions

A bank should be **watching rather than building** when any of the following holds, and most banks are in at least two:

1. **Its incumbent has not been shown to be at its ceiling.** If the gradient-boosted model has unexploited feature headroom, latency headroom, or an untuned operating point, the cheaper work is there. Diagnose the residual errors before replacing the model.
2. **Its evaluation harness cannot detect a small increment.** Without matured-label champion-challenger infrastructure, the project cannot produce a decision (see §10).
3. **Its data-rights position on pre-training is not documented.** Building before resolving §9.5 means building something that may not be deployable.
4. **The target task is credit, adverse action or financial-crime screening.** Those are the tasks with the heaviest explanation obligations and the thinnest published TFM evidence (§5.2, §9.2).
5. **The bank's transaction histories are short, sparse or fragmented across systems.** Sequence pre-training has little to work with, and the point-in-time assembly cost is disproportionate.
6. **The decisioning path is latency-critical and the bank has no serving story for a per-history encoder.** The offline-online gap (§10.5) is where this class most often fails silently.
7. **The intent is to be seen to have a foundation model.** This is not a technical condition, but it is the most common one, and it is what §13.1 is about.

A bank should move from watching to experimenting only when it can answer §11.4's first three conditions, and to extend rather than replace — which is the posture the evidence supports, and the one §12 reaches for a concrete case.

**Verification note for §11.** Everything in 11.1's "documented" list is traceable to a verified source and is cited in the sections above. Everything in the "claimed" list is labelled with its evidence class. 11.2 and 11.3 are the guide's own synthesis on the verified evidence and are explicitly arguments, not findings. **No forecast is offered and no side is taken beyond the evidence**, consistent with the task's constraint: the guide does not assert that TFMs are the future, and it does not assert that they are hype.

---

## 12. The Cymbal Bank Worked Example

> **Cymbal Bank is a fictional institution used throughout this repository as a worked-example persona. Nothing in this section describes a real bank, a real vendor engagement, a real deployment or a real financial result. Every figure below is illustrative and invented for teaching purposes. Cymbal Bank is the only bank persona in this guide, and no real bank is asserted anywhere in it as a customer or user of any model.**

### 12.1 The Scenario and the Disclaimer

**Cymbal Bank** is a mid-size retail and small-business bank. Its payments estate handles card authorisations, account-to-account transfers including instant payments, and batch outgoing payments. It has run a **gradient-boosted fraud model in production for four years**, retrained monthly on its own labelled outcomes, with a decisioning layer that routes alerts to analysts and blocks the highest-scoring transactions.

A vendor presents a proposal: adopt a **transaction foundation model** — in the vendor's framing, a pre-trained model that understands transaction sequences and "improves fraud detection by over 100% over production systems". Cymbal's head of financial crime asks the architecture team to evaluate it. The rest of this section is that evaluation, worked in the order the evidence requires.

### 12.2 The Incumbent's Actual Performance

The evaluation begins — as it should — by measuring the incumbent, because a challenger cannot be assessed without a champion's real numbers.

| Incumbent metric | Illustrative value | Why it matters |
|---|---|---|
| Daily card transactions | ~10 million | Determines alert volume and analyst capacity |
| Fraud rate (labelled, matured) | ~0.12% | Sets the imbalance regime; ROC-AUC is uninformative at this rate |
| Model | Gradient-boosted trees over ~850 engineered features, monthly retrain | The comparator. **Any challenger must be measured against this, fully tuned** |
| Average Precision (PR-AUC), matured labels | ~0.71 | The headline ranking metric |
| Recall at the operating point (analyst capacity: ~3× review ratio) | ~62% | The business-constrained operating point, not a free parameter |
| False-positive rate at that point | ~0.9% of transactions | The customer-friction cost |
| Fraud losses prevented (annual, illustrative) | a nine-figure sum | The baseline economics the proposal claims to improve on |
| Median score latency | single-digit milliseconds on CPU | The constraint any challenger must meet in the authorisation path |

Two observations from this step frame everything that follows. First, **Cymbal's operating point is set by analyst capacity and false-positive cost, not by the model's optimal threshold** — so a challenger that improves ranking at the low-false-positive end of the curve may produce no business change at all. Second, **Average Precision on matured labels at a fixed review ratio is a high bar**: a well-tuned boosted model with 850 features over four years of retraining is a strong incumbent, and the honest framing of the vendor's claim is not "is the TFM better than nothing" but "does the TFM beat 0.71 AP at 0.9% FPR".

### 12.3 The Case for the Sequence Approach

The architecture team concedes the genuine part of the pitch. Cymbal's 850 features include order-sensitive aggregates — recency-weighted counts, time-since-last-transaction, velocity windows — but they were all chosen by engineers, and the specific pattern that most resembles the fraud the incumbent misses at 62% recall is, by construction, one nobody thought to encode. On a sample audit of false negatives, one recurring shape appears: a small transaction to an unusual merchant, followed within minutes by a larger one, in a session pattern that aggregation-level features render as "one normal and one slightly unusual transaction". **That is a sequential pattern, and Cymbal's features genuinely cannot see it.** The case is real; the question is whether the model class, as available, monetises it.

### 12.4 Data Rights and Privacy Analysis

Before any modelling, the legal position:

- **Purpose.** Cymbal's customers provided transaction data to operate their accounts and to prevent fraud. Pre-training an encoder on the **entire** history of **every** customer, for a representation intended to serve multiple future tasks, is a broader purpose than account operation. Under Article 5(1)(b) (verified, §3.5) that must be assessed for compatibility, and Cymbal's counsel concludes that **a general-purpose pre-training run on the full customer base is not one they can support on the account-operation purpose alone**, while a scoped run whose sole documented purpose is fraud prevention on the same data the fraud model already uses is defensible.
- **Scope.** Cymbal therefore scopes the corpus: transaction histories of customers, restricted to the fields the incumbent fraud model already uses, with the documented purpose "improving fraud detection on Cymbal's own transactions". No cross-border training, no group-entity reuse without a separate assessment.
- **Embeddings.** Treated as personal data. The embedding is stored per customer, is subject to the same retention and access discipline as the feature store, and is deleted on account closure.
- **Erasure and objection.** Cymbal's position: on objection, suppress the customer's embedding at inference and retain the model weights; a full retrain on request is not operationally feasible and Cymbal documents that position explicitly rather than pretending the problem does not exist. **This position is recorded as an open legal exposure, not as compliance.**
- **Explainability.** Because the experiment is a feature-augmentation exercise (§12.5), the decisioning layer is unchanged: the decision is made by the tree, which attributes. No customer-facing decision is made by the encoder.

**Outcome of this step:** the rights analysis passes *only* for the narrow, purpose-scoped, internal, non-decisioning configuration. It would fail for a general-purpose pre-training run, and it would fail for a vendor-hosted embedding API unless the data-protection terms were resolved — which is a concrete reason the buy option scores badly here.

### 12.5 Build, Buy or Extend

| Option | Assessment for Cymbal |
|---|---|
| **Buy** | The vendor cannot supply the corpus, the tokenizer or an auditable evaluation; Cymbal would be embedding an opaque model into a decision path with a Regulation B / Article 22 obligation attached, and would be sending customer transaction data (or accepting embeddings derived from it) outside its own control. **Rejected.** A second reason specific and decisive: the vendor's "100%+ improvement over production systems" is *its own* production system, on *its own* population (§6.2, §12.6) |
| **Build** | Cymbal has the data and the engineers, but not the sequence-engineering bench, and the full build path carries 9–18 months and the validation burden of a standalone model. **Not the first move** — premature, given that the smallest experiment has not yet answered the only question that matters |
| **Extend** | Pre-train a small encoder on Cymbal's scoped corpus (self-supervised, no labels), extract a per-customer embedding under point-in-time discipline, feed it into the existing gradient-boosted model as an additional feature block, and measure the increment against the champion. **Selected.** This is the NVIDIA blueprint's pattern and nuFormer's pattern (§8.4) |

### 12.6 The Evaluation Design, and Why the Vendor Numbers Cannot Be Reused

**Why the vendor's numbers cannot be reused.** For the same five reasons set out in full in §10.6 — the vendor's data, the vendor's comparator, an undisclosed metric, an offline evaluation, and no audit trail — plus one that is specific to Cymbal: the vendor's headline improvement is measured against **the vendor's own production system**, and Cymbal's incumbent is not the vendor's incumbent (§6.2). **The evaluation Cymbal actually designs:**

- **Champion.** The production gradient-boosted model, fully tuned, frozen for the duration. It is not re-tuned to make the comparison flattering — and it is not handicapped either.
- **Challenger.** The same gradient-boosted architecture, with the sequence embedding appended as an additional feature block. **The only difference between champion and challenger is the presence of the embedding**, which isolates the increment to sequence information rather than to a different model family.
- **Split.** Strictly out-of-time: training on periods ending before the evaluation window, evaluation on a window after it, so the encoder sees no future information. The encoder is pre-trained on a period ending before the earliest evaluation label.
- **Point-in-time discipline.** Embeddings for the evaluation window are computed from data available at the decision timestamp only. A leakage test is run deliberately: if the challenger's lift vanishes when a one-day embargo is applied between feature computation and label window, the lift was leakage — and that test is treated as mandatory.
- **Labels.** Matured to a fixed horizon, with a second read at a longer horizon to test maturity sensitivity. All fraud labels are re-derived from the case-management record rather than from the incumbent's alerts, so the incumbent's blind spots do not silently define the target.
- **Metrics.** Average Precision and precision-at-fixed-false-positive-rate on matured labels, with confidence intervals; recall at Cymbal's actual review ratio; and a cost-weighted comparison using Cymbal's own false-positive and false-negative costs. **The decision rule is stated in advance:** the challenger is adopted only if it improves the cost-weighted metric at the operating point **and** the improvement is distinguishable from noise on the evaluation window.
- **Out-of-population probe.** The encoder is additionally pre-trained on a **held-out segment of Cymbal's own customers** and evaluated on a second segment, to test within-institution transfer before trusting anything about cross-institution transfer. If it fails at home, the vendor's transfer claims are moot.
- **Operational constraints checked in parallel:** embedding serving latency in the authorisation path, CPU/GPU cost per decision, vocabulary update procedure, and reproducibility of the training run.

**Expectation, stated honestly before the result is seen:** the plausible outcome, given that the incumbent already contains order-sensitive engineered features, is a small-to-absent increment. **A well-run experiment that returns no increment is the most likely good outcome here, and Cymbal's sponsors are told that in advance so that a null result is not read as failure.**

### 12.7 Explainability for the Decisions Involved

Cymbal distinguishes three decisions and their obligations:

| Decision | Who decides | Obligation | Effect on the design |
|---|---|---|---|
| **Credit decision** (e.g. a small-business overdraft) | Credit policy model | **Adverse-action statement of specific reasons** (US) / **GDPR Article 22 safeguards** (EU) where applicable | The encoder **must not** make this decision. No experiment touches it |
| **Payment blocking** | Fraud model plus rules, with analyst review | Customer notification and a dispute route; in the EU, Article 22 exposure if solely automated and significantly affecting | The experiment keeps the decision with the tree and the analyst. The encoder only contributes a feature |
| **Alert prioritisation** | Fraud operations | Lower obligation, but fairness monitoring still applies | The encoder may influence ranking here; this is where the increment is measured |

**The resolution Cymbal adopts is architectural avoidance, and it is the only one the evidence supports (§9.3):** the sequence model stays in the feature and ranking layer; the declination stays on a model that can state specific reasons. Cymbal records explicitly that it has **not** validated any post-hoc attribution technique over the embedding and does not claim one.

### 12.8 Illustrative Cost Comparison

> **Illustrative and fictional. These are invented planning shapes for teaching the decision, not vendor quotes, not benchmarks, and not derived from any source.**

| Line | Buy (as proposed) | Build (full) | Extend (selected) |
|---|---|---|---|
| Licence or platform fee | Recurring annual fee, illustrative seven figures | — | — |
| Legal / data-rights analysis | 0.5 FTE-month (external terms unresolved) | 2 FTE-months | 1 FTE-month |
| Sequence data assembly and point-in-time pipeline | 1 FTE-month (integration) | 5 FTE-months | 3 FTE-months |
| Encoder pre-training compute | Included | Illustrative low-hundreds of GPU-hours | Same as build, scoped |
| Sequence-ML engineering | Vendor-supplied | 2–3 engineers, 6+ months | 1–2 engineers, 4 months |
| Evaluation harness (champion-challenger, leakage tests, cost weighting) | 1 FTE-month | 3 FTE-months | 3 FTE-months — unavoidable on any path |
| Model-risk documentation and validation | 3 FTE-months, harder without provenance | 4 FTE-months | 3 FTE-months |
| Analyst and operations change | Material — new tooling | Material | Minimal — the operating procedure is unchanged |
| Time to a decision Cymbal can defend | 3–6 months to integration; longer to a defensible answer | 12–18 months | **6–9 months** |

**The decisive line is not compute; it is that Extend is the only path where the operating procedure, the explainability position and the validation file remain intact.**

### 12.9 The Recommendation

**Cymbal approves a limited, time-boxed extension experiment alongside the incumbent. It does not adopt the vendor's proposal. It does not build a standalone TFM.**

The reasoning, in the order the evidence supports it:

1. **The incumbent is strong and its operating point is constrained by analyst capacity and false-positive cost, not by model quality.** Any gain must appear at that operating point. This is the first thing to establish, and it is established before the vendor is judged (§12.2).
2. **The case for sequence information is real and specific at Cymbal** — the false-negative audit found a sequential pattern the feature set cannot express (§12.3). That justifies one experiment, not a migration.
3. **The vendor's evidence is unusable for a Cymbal decision** — a ratio against the vendor's own production system, on the vendor's population, on a metric Cymbal cannot map (§12.6). It is a reason to run the experiment; it is not a reason to deploy.
4. **Data rights permit only a narrow configuration**, and that configuration happens to be the one the evidence actually supports: internal, purpose-scoped, non-decisioning (§12.4).
5. **Explainability is resolved by architecture, not by technique** — the encoder contributes a feature; the tree and the analyst keep the decision (§12.7). No unvalidated attribution method is claimed.
6. **Extend is the option that is cheap, reversible and informative either way** (§12.5, §12.8). A null increment closes the question at low cost and revalidates the incumbent — which is itself a valuable outcome, because it converts "we think our model is good" into "we have measured that sequence information adds nothing further."

**Cymbal's explicit decision rule and its honest conclusion:**

- **Adopt a standalone TFM** in production decisioning: **not supported by the evidence** available in this pass, and refused.
- **Adopt a limited extension experiment alongside the incumbent:** **supported**, and approved, with a decision gate at the end of the evaluation window.
- **No change:** **the correct default if — and this is the honest fallback — the data-rights analysis fails, the point-in-time pipeline cannot be built without leakage, or the challenger's increment is indistinguishable from noise at the cost-weighted operating point.** Any of those three is a plausible outcome, and the team commits in advance to accepting it.

The most defensible outcome the evidence supports, stated without dressing it up: **Cymbal does not adopt a transaction foundation model for decisions that affect customers; it tests whether sequence embeddings add measurable signal to the incumbent it already trusts, under a charter that permits a null result.**

---

## 13. Anti-Patterns

Each entry gives symptom, cause and guardrail.

### 13.1 Adopting a sequence model because the incumbent is unfashionable

- **Symptom:** The sponsor's framing is comparative and aesthetic — "we're still on gradient boosting while the industry is moving to foundation models". No specific residual error in the incumbent has been identified. The business case cites the category's momentum rather than a measured shortfall.
- **Cause:** A foundation-model narrative in the market plus the normal institutional pressure to appear current. The comparison being made is between the bank's production reality and the vendor's marketing material, not between two models on the same data.
- **Guardrail:** Require a **named, quantified residual error in the incumbent** before any challenger work is authorised — a specific pattern in the false negatives, with a count and an estimated loss. If nobody can produce one, the answer is no. Second guardrail: make "revalidate and retune the incumbent" an explicitly acceptable outcome of the same budget cycle, so the option to conclude "the incumbent is fine" carries no career penalty.

### 13.2 Benchmarking against a naive baseline rather than the production model

- **Symptom:** A vendor or internal evaluation reports a large relative lift over "a baseline" — logistic regression, a rules engine, a single decision tree, or "a strong baseline" that is never specified. The production model is absent from the comparison.
- **Cause:** A naive baseline is easy to beat; a well-tuned boosted-tree incumbent with years of feature engineering is not. The published record shows this pattern throughout — the NVIDIA blog compares against "a strong XGBoost baseline" on synthetic data without publishing its tuning, and TREASURE compares against its own architecture variants and an undisclosed production system (§6.6).
- **Guardrail:** **The comparator is the production model, fully tuned, at the production operating point, and nothing else counts as a result.** Write the comparator into the experiment charter before the first training run. Where a vendor supplies a baseline, demand the architecture, the feature count, the tuning and the metric value.

### 13.3 Assuming transfer across institutions when the data distribution is proprietary

- **Symptom:** "The model was trained on billions of transactions, so it will generalise to ours." The vendor's corpus is orders of magnitude larger than the bank's, and the size is offered as a proxy for transferability.
- **Cause:** Scale is confused with coverage. The published transfer evidence is almost entirely intra-organisation — TREASURE's transfer is inside Visa, TransactionGPT's inside its network, nuFormer's inside Nubank, and the NVIDIA blueprint's on synthetic data. Skalski et al. demonstrates cross-issuer transfer, but within a shared consortium corpus (§6.3, §6.6).
- **Guardrail:** **Test within-institution transfer first, then cross-segment transfer inside the bank, before believing anything about cross-institution transfer.** Make the out-of-population probe a mandatory experiment (§12.6). Treat "we pre-trained on more data than you have" as a data-advantage claim, not a transfer claim.

### 13.4 Treating a vendor's metric as evidence

- **Symptom:** A slide with a percentage on it enters the business case, and the number is later quoted in a board paper or a regulator conversation without its provenance.
- **Cause:** Class-3 vendor benchmarks are the most quotable, least verifiable artefacts in the field, and their caveats live in footnotes that do not survive the journey to a slide. The published record contains at least two numbers — TREASURE's 111% and NVIDIA's 41.76%-versus-"near-50%" — that are either against undisclosed comparators or internally inconsistent (§6.1, §6.5).
- **Guardrail:** **Every externally sourced performance figure must be recorded with its evidence class, its dataset, its comparator and its date before it may enter any decision document.** If the comparator or metric is unknown, the figure may be quoted as "the vendor states X" and may not be used as an input to a sizing or approval decision. The claims audit in §14 is the artefact this guardrail produces.

### 13.5 Ignoring label delay when measuring fraud performance

- **Symptom:** The challenger looks better in a comparison run shortly after deployment, and the gap narrows or reverses when the same period is re-scored months later once disputes have settled. Alternatively, the comparison is never re-run, and an immature-label result becomes the accepted truth.
- **Cause:** Fraud labels mature through chargebacks and investigation over weeks to months. An early read systematically over-credits whichever model flags the fastest-maturing cases, and the incumbent's label set does not contain fraud the incumbent missed (§10.2).
- **Guardrail:** **Fix a label-maturation window in the experiment charter, evaluate only on labels matured to that horizon, and re-evaluate when the window rolls.** Report the window in every result. Where possible, re-derive labels from the case-management record rather than from the incumbent's alerts, so contamination bias does not define the target.

### 13.6 Deploying a model whose decisions cannot be explained to the party affected

- **Symptom:** A declined applicant receives reasons generated by a post-hoc attribution method that nobody has validated against the model's actual behaviour, or receives a reason that maps to a feature the customer cannot act on. Legal or compliance first learns about the explainability position after launch.
- **Cause:** The explanation requirement is treated as an afterthought bolted onto a model chosen on performance grounds. No source in this pass demonstrates a post-hoc attribution or attention-inspection method meeting an adverse-action or Article 22 standard for a sequence model's decision (§9.3).
- **Guardrail:** **Classify every deployment by decision type before the model is chosen, and let the classification drive the architecture.** If the decision is a credit decision or a solely-automated decision that significantly affects the person, keep the declination on a model that states specific reasons and confine the sequence model to ranking and prioritisation. Record explicitly which explanation techniques have **not** been validated. Cross-ref [ai_genai_banking_compliance_guide.md](ai_genai_banking_compliance_guide.md) for the regulatory machinery and [../technology/ai_llm/ai_governance_bias_redteaming_guide.md](../technology/ai_llm/ai_governance_bias_redteaming_guide.md) for the governance practices.

### 13.7 The training-data rights question discovered after the model is built

- **Symptom:** A working model exists, and the purpose-limitation, lawful-basis or cross-border analysis begins afterwards. The conclusion is either that the corpus cannot lawfully be used for pre-training, or that some customers' data must be excluded — which in a pre-trained model means retraining from scratch, discarding months of work.
- **Cause:** Pre-training feels like a modelling task rather than a processing operation. GDPR Article 5(1)(b) and its analogues apply to the pre-training use, not only to the deployment (§3.5, §9.5), and the "specified, explicit and legitimate purpose" is hardest to articulate for a general-purpose encoder trained on the whole customer base.
- **Guardrail:** **The purpose and lawful-basis analysis is a gate, not a workstream: no training run starts before it is written, scoped and signed off**, covering the collection, training and deployment jurisdictions and a position on embeddings as personal data. Scope the corpus to the narrowest configuration that answers the business question — which, in Cymbal's case, was also the configuration the evidence supported (§12.4). Treat corpus scope as a design input on day one, because it constrains what the model can learn.

---

## 14. The Claims Audit

**Evidence classes:** (1) paper / (2) vendor or consultancy publication / (3) vendor benchmark or case study / (4) press. **Verdicts:** VERIFIED — confirmed against a primary or reproduced-primary source during this pass; FLAGGED — sourced but with a material caveat, an undisclosed component, an internal inconsistency, or confirmed only via a secondary source; REJECTED — contradicted or unsupported. Paper identifiers and claims, all performance figures, all product claims and all regulatory provisions are the highest-risk classes and are audited individually below.

| # | Claim | Source | Date | Class | Verdict |
|---|---|---|---|---|---|
| 1 | arXiv:2511.19693 = "TREASURE: The Visa Payment Foundation Model for High-Volume Transaction Understanding", 15 authors, Visa Research | arXiv abs + HTML v3 | 24 Nov 2025 v1 / 8 Apr 2026 v3 | 1 | **VERIFIED** (identifier, title, authors, dates, affiliations) |
| 2 | TREASURE accepted at the 32nd ACM SIGKDD Conference; DOI 10.1145/3770854.3783942 | TREASURE HTML v3 header | Aug 2026 proceedings | 1 | **VERIFIED** at the paper's own header (note: the header's Jeju Island date string reads "August 9–13, 2025" while the venue is KDD 2026 — an internal inconsistency in the paper's own front matter) |
| 3 | TREASURE corpus: ~6 billion transactions, 30 million cardholders, 1 Sep 2020 – 30 Nov 2022; 24 months train / 25th validation / final month test | TREASURE §4.1 | as above | 1 (dataset) | **VERIFIED as transcribed**, but non-public and unauditable |
| 4 | TREASURE architecture: 3-layer decoder, 4 heads, hidden 256, 3-layer input module, no explicit positional encoding, 20 epochs, AdamW 10⁻⁴, batch 256, 512-event context cap; 5 static + 16 dynamic attributes, names withheld; InfoNCE with batch-shared negatives and a 1,024-cardinality threshold (3GB/6GB memory vs OOM above 64 negatives) | TREASURE §1, §3.2, §3.5, §4.1 | as above | 1 | **VERIFIED** (memory measurements reported under PyTorch 2.6) |
| 5 | TREASURE: +111% abnormal-behaviour detection over production systems | TREASURE abstract, §4.2 | as above | 3 carried in class 1 | **FLAGGED** — RI = 2.1171 against "the currently deployed system" on an **undisclosed in-house metric** with withheld absolute values, on non-public data, offline. Also note 2.1171 rounds to 112%, not 111% |
| 6 | TREASURE: +104% recommendation improvement from embeddings | TREASURE abstract, §4.5 | as above | 3 carried in class 1 | **FLAGGED** — the same result is stated as **90%** in §1 of the same paper. Internally inconsistent; baseline is task-specific supervised embeddings, internal only |
| 7 | TREASURE: Transformer outperforms GRU and LSTM backbones | TREASURE §4.3, Table 2 | as above | 1 | **VERIFIED as reported** — but the comparison is TREASURE-with-a-GRU (RI 1.3979) vs TREASURE-with-a-Transformer (RI 2.1171); **not** a head-to-head against Skalski et al.'s GRU model |
| 8 | TREASURE was not in full production deployment at the time of writing | TREASURE §4.1 | as above | 1 | **VERIFIED** — "actively working toward full production deployment" |
| 9 | Skalski et al. 2023 = "Towards a Foundation Purchasing Model: Pretrained Generative Autoregression on Transaction Sequences", ICAIF '23, pp. 141–149, DOI 10.1145/3604237.3626850 | arXiv:2401.01641 abs + TREASURE bibliography | Nov 2023 / Jan 2024 | 1 | **VERIFIED** (title, five authors, venue, pages, DOI, arXiv ID, CC-BY-NC-SA licence) |
| 10 | Skalski et al.: pre-trained on 180 issuing banks, 5.1 billion transactions; improves value detection rate at high precision thresholds; transfers out-of-domain; benchmarks on public datasets | Skalski et al. abstract | Jan 2024 | 1 | **VERIFIED as abstract claims**; the paper **body** was not read |
| 11 | arXiv:2507.23267 = "Your Spending Needs Attention: Modeling Financial Habits with Transformers" (nuFormer), 12 authors, Nubank | arXiv abs | 31 Jul 2025 v1 / 10 Aug 2026 v2 | 1 | **VERIFIED** (identifier, title, authors, dates, cs.IR) |
| 12 | nuFormer: SSL on transaction data; integrates user embeddings with existing tabular features; improvements on large-scale recommendation at Nubank; no new data sources | nuFormer abstract | as above | 1 | **VERIFIED as abstract claims.** Note the paper says **one** domain (recommendation) and **augments** the tabular model |
| 13 | arXiv:2511.08939 = "TransactionGPT", 29 authors, Visa Research, "Technical Report" | arXiv abs | 12 Nov 2025 v1 / 2 Mar 2026 v2 | 1 | **VERIFIED** (identifier, title, authors, dates, cs.LG/cs.CL, Technical Report comment) |
| 14 | TransactionGPT: 3D-Transformer, billion-scale real-world transactions, improves anomaly detection vs a competitive production model, benchmarks LLM embeddings | TransactionGPT abstract | as above | 1 (claims) | **FLAGGED** — abstract-level only; no quantitative results stated in the abstract, and the paper body was not read in this pass. Comparator described only as "a competitive production model" |
| 15 | arXiv:2604.08649 = "PRAGMA: Revolut Foundation Model", 14 authors | arXiv abs | 9 Apr 2026 v1 / 24 Aug 2026 v2 | 1 | **VERIFIED** (identifier, title, authors, dates) |
| 16 | PRAGMA: masked modelling pre-training on a large banking event corpus; linear probe then light fine-tuning; credit scoring, fraud detection, LTV supported | PRAGMA abstract | as above | 1 (claims) | **FLAGGED** — abstract-level only; "superior performance" against **unnamed** baselines on a non-public corpus; body not read |
| 17 | Stripe announced "the world's first AI foundation model for payments" at Sessions, trained on tens of billions of transactions with unsupervised learning | Stripe newsroom; TechCrunch; company LinkedIn post | 7 May 2025 | 2 + 4 | **FLAGGED** — the model's existence and the announcement are verified; the "world's first" claim is **contradicted by the published record** (Skalski et al., Nov 2023; nuFormer, Jul 2025). No paper, no metrics |
| 18 | Plaid operates a "transaction foundation model" and a "sequential foundation model" with CPC, RTD and TCL objectives | Plaid engineering blog (two posts) | 25 Jun 2026 (sequential); earlier 2026 (transaction) | 2 | **VERIFIED as vendor statements** — dates, authors, objective names and the "in testing with two of our most consequential use cases" status confirmed. No performance numbers published |
| 19 | Mastercard is building a generative "large tabular model" on anonymised transaction data, as an insights engine rather than a chatbot | Mastercard news story; corroborating press | 2026 | 2 + 4 | **VERIFIED as vendor framing** — including that Mastercard calls it a **tabular**, not sequence, model. No metrics, no paper |
| 20 | NVIDIA blog "Build Your Own Transaction Foundation Model for Financial Intelligence", author Benjamin Wu | developer.nvidia.com | 16 Jun 2026 | 2 | **VERIFIED** (date, author, title, URL) |
| 21 | NVIDIA blueprint: ~12 semantic tokens per transaction, 6,251-symbol vocabulary, ~29M-parameter Llama decoder, NeMo AutoModel, IBM TabFormer 24.4M synthetic transactions, 512-d embeddings via last-token pooling, 64-d PCA, cuDF/cuML | NVIDIA blog | 16 Jun 2026 | 2 | **VERIFIED** — all six specifics confirmed in the blog text, and the repository at github.com/NVIDIA-AI-Blueprints/transaction-foundation-model is linked from it |
| 22 | NVIDIA: Average Precision lift of 41.76% over a strong baseline on the temporal test split | NVIDIA blog (AI-generated summary) | 16 Jun 2026 | 3 | **FLAGGED** — the blog's own body states "a near-50% lift... over a strong XGBoost baseline"; two different figures for the same result. Baseline characterised but not specified; data is **synthetic** |
| 23 | NVIDIA's example uses the TFM to **augment** XGBoost rather than replace it | NVIDIA blog | 16 Jun 2026 | 2 | **VERIFIED** — "combining raw tabular features with foundation-model embeddings in an XGBoost classifier"; "augmenting a downstream fraud classifier with embeddings" |
| 24 | F-Industries definition of a TFM and its claimed benefits in fraud, reconciliation and correspondent banking | findustries.co | 16 Jun 2026 | 2 | **FLAGGED** — the URL and the definition string are verified as present; the page **body was not read in full** in this pass. Zero published evidence, metrics or datasets: a named party's claim, not a finding. Its definition is materially looser than the academic one (no pre-training or transfer requirement) |
| 25 | Thoughtworks published "The power of Transaction Foundation Models: Building the unified intelligence layer for payments" | thoughtworks.com | undated in this pass | 2 | **UNVERIFIED** — the article body could not be retrieved on two attempts (cookie wall / landing-page redirect). **Tool limitation, not evidence of absence** (§15) |
| 26 | GDPR Article 22: right not to be subject to solely automated decisions with legal or similarly significant effects; exceptions; Article 22(3) safeguards; recital 71 pointer. Article 5(1)(b): purpose limitation with the Article 89(1) research/statistical carve-out | GDPR Arts. 22 and 5, reproduced at gdpr-info.eu | Regulation (EU) 2016/679 | — | **VERIFIED** at reproduced text (not at EUR-Lex directly) |
| 27 | GDPR Article 6 lawful basis analysis for pre-training; Articles 17 and 21 interaction with model weights | This guide's analysis | Sep 2026 | — | **FLAGGED** — Article 6 reasoning is an assessment, not a verified provision-by-provision conclusion; Articles 17 and 21 were not read at primary text |
| 28 | EU AI Act Annex III point 5(b): creditworthiness/credit-score AI is high-risk, **"with the exception of AI systems used for the purpose of detecting financial fraud"** | artificialintelligenceact.eu Annex III | Regulation (EU) 2024/1689 | — | **VERIFIED** — the wording and the fraud exception confirmed. (Article 6(3) filtering and non-Annex obligations were not read and are not asserted in this guide.) |
| 29 | US: creditor must provide adverse-action notice and statement of specific reasons | Regulation B, 12 CFR § 1002.9, consumerfinance.gov | current version | — | **VERIFIED** at the CFPB regulation page (notification, ECOA notice, statement of specific reasons, timing). (GLBA and FCRA were not read at primary text and are flagged in §15.) |
| 30 | Singapore: MAS FEAT — four principles, published 12 Nov 2018, non-binding, with Veritas assessment tooling | OECD AI policy database, FinTech News Singapore, specialist compilations | 12 Nov 2018 | — | **FLAGGED** — confirmed via secondary compilations; the primary MAS page returned 404 on this pass. (EU PSD2/PSD3/FIDA were not retrievable at primary text and are flagged in §15.) |
| 31 | Visa processes 300B+ transactions / 15T dollars annually across 4B+ credentials and 150M+ merchants | TREASURE §1 citing Visa's own 2024 fact sheet | 2024 | 2 (quoted in 1) | **FLAGGED** — a company's self-reported figure quoted by its own researchers; not independently verified |

---

## 15. What Could Not Be Verified

Recorded honestly, in the form the task requires — and in each case as **an absence of evidence in this pass, not evidence of absence**. Some of these are tool limitations and some are genuine gaps in the public record; the two are labelled distinctly.

### Tool limitations (retrieval failed; the source may well exist)

1. **Thoughtworks, "The power of Transaction Foundation Models: Building the unified intelligence layer for payments".** Retrieval was attempted and returned the Thoughtworks blogs landing page behind a cookie wall / client-side redirect, with no article body. **The article's existence, date, author and claims are therefore unverified, and no claim from it appears anywhere in this guide.** This is a tool limitation, not evidence of absence.
2. **The primary MAS FEAT page.** `mas.gov.sg/schemes-and-initiatives/feat-principles` returned HTTP 404 during this pass. MAS FEAT's substance (four principles, 12 November 2018, non-binding, Veritas toolkit) was verified instead via the OECD AI policy database, FinTech News Singapore and specialist compilations — secondary sources, and labelled as such in §3.5 and §9.4.
3. **EUR-Lex retrieval for the EU open-finance instruments.** The FIDA proposal URL (COM(2023) 360) failed to scrape, and the AI Act Article 6 page timed out. Annex III point 5(b) was verified through artificialintelligenceact.eu instead, which is why §9.4 quotes that provision specifically and characterises FIDA/PSD3 only directionally. The F-Industries page body likewise could not be read in full, so its claims are treated throughout as a named party's claim rather than as findings.

### Genuine gaps in the public record

4. **The TransactionGPT, PRAGMA, nuFormer and Skalski et al. paper bodies.** Only their abstracts, listings and metadata were read. Every claim attributed to them in this guide is an abstract claim, and finer architectural or experimental detail — baselines, dataset sizes, metric definitions, ablations — is unverified.
5. **The NVIDIA blog's true effect size.** The page carries 41.76% in its AI-generated summary and "near-50%" in its body for the same result. Which is correct, and what the baseline's actual Average Precision is, could not be determined from the page.
6. **A published comparison of any TFM against a well-tuned, publicly benchmarked gradient-boosted-tree incumbent on real transaction data.** No such comparison was located in this pass. This is the guide's principal finding and it is stated as an absence of a published comparison, not as proof that no unpublished comparison exists.
7. **Any vendor product offered as a transaction foundation model.** No vendor source was located offering a licensable TFM with weights, a documented pre-training corpus, or a transfer evaluation. Nor was any fraud or financial-crime platform vendor located that publishes a TFM paper or pre-training corpus description.
8. **Any quantitative result from Stripe, Plaid or Mastercard.** All three describe models but none publishes metrics in the material reviewed; there is no paper for any of the three in this pass.
9. **TREASURE's actual abnormal-behaviour metric.** The metric's definition and absolute values are withheld by the paper by design. RI = 2.1171 cannot be converted into a precision, recall or Average Precision figure.
10. **TREASURE's scaling-law result.** The contributions list asserts a scaling law for transaction data; the §4.7 body was not read in detail, so the shape and claims of that analysis are unverified.
11. **TransactionGPT's relationship to TREASURE beyond shared authorship.** Several authors appear on both papers and both are Visa Research. The papers are treated here as one programme rather than two independent results; any formal relationship (companion papers, shared corpus, shared code) is unverified.
12. **Whether embeddings derived from transaction data are personal data, what deletion from a pre-trained model means, and whether any explanation technique meets an adverse-action or Article 22 standard for a sequence model's decision.** No verified source found for any of the three. Recorded as open governance gaps in §9.3 and §9.5.
13. **A published, independent evaluation of the lawfulness of these systems' pre-training data**, and any cross-border or group-wide reuse assessment for a transaction foundation model. No source in this pass addresses either for TREASURE, TransactionGPT, PRAGMA, nuFormer, Skalski et al. or any vendor model.
14. **Adversarial-robustness results for any transaction foundation model.** No source found. Nor was the BIS, FSB, BCBS or IOSCO position on this model class searched in this pass; the supervisory material is owned by [ai_genai_banking_compliance_guide.md](ai_genai_banking_compliance_guide.md) in any case.

**Deliberately not asserted:** no real bank is named anywhere in this guide as a customer or user of any model — the named institutions are the publishers of the work under discussion. And no forecast is offered: the guide does not assert that TFMs are the future of transaction modelling, nor that they are hype.

---

## 16. Glossary, Cross-References and Closing Summary

### 16.1 Glossary

| Term | Definition | Where it is used here |
|---|---|---|
| **Transaction Foundation Model (TFM)** | A label used, inconsistently, for transformer- or recurrent-based models pre-trained on sequences of financial transactions. In the strict sense: self-supervised pre-training on unlabelled event sequences at scale, then transfer to many downstream tasks | §1 |
| **Foundation model** | A model pre-trained at scale on broad data and adaptable to many downstream tasks. The load-bearing criterion is transfer, not size | §1.3, §1.5 |
| **Pre-training** | The large-scale, usually self-supervised phase that learns general representations before any task-specific training | §1.5, §5.1 |
| **Fine-tuning / adaptation** | The smaller supervised phase that adapts pre-trained representations to a task. The ladder runs from frozen-embedding probes to full joint training | §5.1 |
| **Event sequence** | A customer's transactions as an ordered series of timestamped events | §2.1 |
| **Static / dynamic attribute** | An attribute constant across a customer's transaction history versus one that varies per event | §4.2 |
| **Tokenization of transactions** | Encoding a transaction record into model inputs. NVIDIA uses ~12 semantic tokens per transaction with a 6,251-symbol vocabulary; TREASURE uses static and dynamic attribute vectors with per-attribute embedding tables | §4.1 |
| **Vocabulary** | The set of symbols the model's input and output spaces support. Maintained and versioned; an unversioned input space is a monitoring gap | §4.1, §9.6 |
| **Embedding** | A fixed-length vector representing a variable-length history or an event, reusable as a feature or as a representation for a shallow head | §1.5, §5.1 |
| **Masked modelling** | An objective that hides part of the input and predicts it from context. Used by PRAGMA; the sibling of next-event prediction | §4.6 |
| **Network signals** | Payment-network response codes and system flags, known only after a transaction is processed. TREASURE predicts them as auxiliary outputs, and they are available only to a network operator | §4.5 |
| **Contrastive loss / InfoNCE** | An objective that pulls a positive pair together and pushes sampled negatives apart, used to make 100M-way categorical prediction tractable | §4.4 |
| **High-cardinality categorical** | A categorical attribute with an enormous number of distinct values — TREASURE reports a range from 2 to over 100M | §4.4 |
| **Relative Improvement (RI)** | TREASURE's reported ratio of challenger to incumbent performance on an undisclosed in-house metric; RI 1.5 means 50% better | §6.1, §6.2 |
| **Average Precision (AP)** | Area under the precision-recall curve; the appropriate ranking metric under extreme class imbalance | §10.3 |
| **Champion / challenger** | The production model versus the candidate model, compared on the production metric at the production operating point | §10, §12.6 |
| **Tabular baseline** | The feature-engineering plus gradient-boosted-tree pipeline that dominates production fraud and credit — the incumbent that any TFM must beat | §1.5, §6.6 |
| **Point-in-time correctness** | Computing features (and embeddings) from only the data available at the decision timestamp, without leakage | §8.6, §12.6 |
| **Label maturation** | The delay before a fraud or default outcome is final, during which any evaluation is optimistic; the derived bias from labels generated by the incumbent's own decisions is **label contamination** | §10.2 |
| **Offline-versus-online gap** | The difference between a retrospective evaluation and behaviour in production, where latency, alert budget, customer reaction and interleaved systems apply | §10.5 |
| **Adverse action** | An unfavourable credit decision that triggers notification and statement-of-reasons obligations under US Regulation B § 1002.9 | §9.2, §9.7 |
| **GDPR Article 22** | The right not to be subject to a solely-automated decision with legal or similarly significant effects, subject to exceptions and safeguards including human intervention and the right to contest | §9.2, §9.7 |
| **MAS FEAT** | Fairness, Ethics, Accountability, Transparency — Singapore's non-binding principles for AI and data analytics in the financial sector, published 12 November 2018 | §3.5, §9.4, §9.7 |
| **High-risk AI system (EU AI Act)** | A system listed in Annex III or a safety component under Article 6(2); Annex III 5(b) covers creditworthiness and credit scoring, expressly excepting financial-fraud detection | §9.4, §9.7 |
| **Evidence class (1–4)** | The classification used throughout: paper / vendor or consultancy publication / vendor benchmark / press | §1.6 |

### 16.2 Cross-References

**Sibling guides in `banking/` (plain filenames):**
- [financial_fraud_detection_at_scale_guide.md](financial_fraud_detection_at_scale_guide.md) — fraud-detection-at-scale discipline: architecture, stream processing, latency, feature engineering, model lifecycle. **A TFM is a model class, not a detection system**; the detection system is owned there.
- [regtech_guide.md](regtech_guide.md) — the AML/KYC and regtech vendor landscape, including named screening and monitoring vendors. This guide cross-refers its vendor rows rather than re-assembling them.
- [ai_genai_banking_compliance_guide.md](ai_genai_banking_compliance_guide.md) — AI regulation and model-risk material for banking: EU AI Act machinery, supervisory expectations, model-risk integration. Cited by name throughout §9.
- [risk_management_models_guide.md](risk_management_models_guide.md) — the models behind the risk tooling.
- [payment_rails_guide.md](payment_rails_guide.md), [payments_hub_guide.md](payments_hub_guide.md), [bian_transaction_engine_guide.md](bian_transaction_engine_guide.md) — the rails that generate the transaction data, the hub that routes it, and the engine that processes it.
- [banks_in_singapore_guide.md](banks_in_singapore_guide.md) — the Singapore regime that MAS FEAT sits within.

**Technology guides (prefix `../technology/`; `../` = repository root):**
- [../technology/mlops_lifecycle_frameworks_guide.md](../technology/mlops_lifecycle_frameworks_guide.md) — MLOps lifecycle: retraining, monitoring, rollout. §8 and §10 cross-refer its techniques rather than re-deriving them.
- [../technology/adversarial_ml_attacks_guide.md](../technology/adversarial_ml_attacks_guide.md) — the adversarial-ML discipline behind §10.4.
- [../technology/ai_llm/ai_governance_bias_redteaming_guide.md](../technology/ai_llm/ai_governance_bias_redteaming_guide.md) and [../technology/ai_llm/llm_evaluation_vs_validation_guide.md](../technology/ai_llm/llm_evaluation_vs_validation_guide.md) — governance, bias and red-teaming practice behind §13.6, and the evaluation-versus-validation distinction relevant to §10.
- Transformer, pre-training and fine-tuning mechanics, and AI platforming: the repository's AI/LLM guides under [../technology/ai_llm/](../technology/ai_llm/).

### 16.3 Closing Summary

The label "Transaction Foundation Model" describes something real and something overstated at the same time, and the two are separable.

**Real:** pre-training self-supervised representation models on orders of magnitude of transaction events works. The architecture is small by modern standards — a three-layer decoder with hidden dimension 256, a 512-event context, or a 29-million-parameter Llama decoder — and the engineering constraints are published: a 6,251-symbol vocabulary, contrastive loss with batch-shared negatives, a 1,024-cardinality threshold, memory ceilings measured under PyTorch 2.6. Two independent production teams and at least four research groups converged on the same posture: use the pre-trained encoder to produce embeddings that feed an existing model. The underlying argument is mechanical and sound — aggregation is many-to-one, so a per-row formulation discards ordering and cadence by construction.

**Overstated:** the transfer claim. TREASURE, TransactionGPT, PRAGMA and nuFormer demonstrate transfer inside the organisation that built them, on non-public data. Skalski et al., the earliest work in the class and the only one with public-dataset benchmarking, titles itself *Towards* a foundation model and demonstrates cross-issuer transfer within a shared consortium corpus. The headline number everyone quotes — 111% over "production systems" — is a ratio on an undisclosed metric, against a baseline that is the publisher's own prior system, on data no reader can obtain, in an evaluation that has not yet reached production. The repeatable, openly reproducible artefact in the whole field is a synthetic-data blueprint whose demonstrated use is to **augment** XGBoost, not replace it. And in no work found here does a TFM beat a well-tuned, publicly benchmarked gradient-boosted-tree incumbent on real transaction data.

**Unresolved, and the reason a bank should watch rather than migrate:** how to explain a sequence model's decision to the person it affects, when the obligation is to state specific reasons and the model computes a vector; whether pre-training is a compatible use of the data under purpose limitation, and what deletion means for weights that have absorbed it; whether a model trained on one population transfers to another's; and whether any offline lift survives label delay, the alert budget and an adversary that learns.

**The practical conclusion this guide can defend:** extend rather than replace. Pre-train or obtain an encoder, extract embeddings under point-in-time discipline, append them to the incumbent as a feature channel, measure the increment against the incumbent on matured labels at the operating point the business actually runs, and accept a null result as a good outcome. Keep the declination on a model that can state its reasons. Resolve the data-rights question before the first training run, not after. And when the vendor's deck reports a lift over "production systems", ask whose production system, on whose metric, on whose data — because in this field, that question is the entire difference between a benchmark and a decision.

The information was never in the row. It was always the sequence, not the row.
