# The Machine Learning System Design Interview: A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Professional Development / Career Series — ML Engineering, MLOps, Interview Preparation, Career Progression
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** technology/ (career-skills / ML-engineering series)
> **Last Updated:** August 2026

---
> A comprehensive guide to the **Machine Learning System Design (MLSD) interview** — the open-ended, product-flavored interview round used by FAANG and most ML-heavy companies to test whether a candidate can design a *complete ML system*, not just fit a model. It covers the interview format and the roles it targets, the evaluation criteria, the published frameworks (Chip Huyen 2022, Alex Xu & Ali Aminian 2023), the eight components of every design (clarify → data → features → model → training → serving → evaluation → monitoring), the classic case studies (recommendation, search ranking, feed ranking, ad prediction, fraud detection), the trade-offs and common mistakes, a 4–6 week practice plan, a full worked example (fraud detection for a bank), and a one-page summary.

**Audience**: ML engineers, applied scientists, and data scientists preparing for ML system design interviews; solution architects in banking (the Cymbal Bank context) who evaluate or build ML platforms and want to speak the ML design language; hiring managers designing ML interview loops. The worked example deliberately uses **fraud detection for a bank** — a familiar banking context — so the framework is exercised on a domain the reader already knows.

**How to use this guide:**

- **Interview candidates**: read Sections 1–3 (format, criteria, framework) and 11 (practice plan) first, then drill Sections 4–8 (components) and 9 (case studies), and finish with Section 12 (worked example) as a rehearsal script.
- **Architects / hiring managers**: Sections 1–2 (what the round tests), Section 9 (what good answers look like across domains), Section 10 (the failure modes to screen for).
- **Everyone**: the glossary (Section 14) doubles as an ML-design vocabulary checklist — if a term is unfamiliar, that is a learning item, not trivia.

**Cross-references in this series** (this guide links to, rather than duplicates, the deep-dives): `ml_platforms_comparison_guide.md` (feature stores, ML platforms, Feast/Tecton), `mlops_lifecycle_frameworks_guide.md` (the MLOps lifecycle the design must feed into), `mathematics_for_ml_and_ds_study_notes.md` (the math prep: metrics, probability, loss functions), `quantitative_developer_skillset_guide.md` (the career-skills framing and interview lens for a banking technologist), `personalization_engines_guide.md` (recommendation/feed systems in production), `llm_evaluation_frameworks_guide.md` and `llm_evaluation_vs_validation_guide.md` (evaluation methodology, LLM-era), `ai_agent_drift_guide.md` and `drift_detection_methods_guide.md` (monitoring and drift), `knowledge_distillation_guide.md` (model compression for latency), and the `banking/` guides (e.g. `financial_risk_compliance_systems_guide.md`, `full_stack_banking_guide.md`) for the fraud-detection domain context.

**Verification notes (honesty policy):** facts in this guide were checked against primary or near-primary sources where possible (the ByteByteGo book listing, Chip Huyen's published materials, the two-tower/YouTube literature). Where a claim is common knowledge but not cleanly verifiable — interview *frequency* statistics, company-specific rubrics, compensation numbers — it is explicitly flagged as **anecdotal/unverified** rather than asserted as fact.

---

## Table of Contents

1. [The Interview Format](#1-the-interview-format)
2. [The Evaluation Criteria](#2-the-evaluation-criteria)
3. [The Framework](#3-the-framework)
4. [The Data](#4-the-data)
5. [The Features](#5-the-features)
6. [The Model](#6-the-model)
7. [The Training and the Serving](#7-the-training-and-the-serving)
8. [The Evaluation](#8-the-evaluation)
9. [The Case Studies](#9-the-case-studies)
10. [The Trade-offs and the Mistakes](#10-the-trade-offs-and-the-mistakes)
11. [The Practice Plan](#11-the-practice-plan)
12. [The Worked Example: Fraud Detection for a Bank](#12-the-worked-example-fraud-detection-for-a-bank)
13. [The One-Page Summary](#13-the-one-page-summary)
14. [Glossary](#14-glossary)
15. [Conclusion](#15-conclusion)
16. [Appendix: The Prompt Bank and Resources](#16-appendix-the-prompt-bank-and-resources)

---

## 1. The Interview Format

### 1.1 What Is the ML Design Interview?

The **ML system design interview** is a ~45–60 minute, open-ended conversation in which the interviewer describes a product problem — "design a recommendation system for an e-commerce app", "design fraud detection for a credit card issuer", "design a news feed ranking system" — and the candidate designs the end-to-end ML system that would solve it. It is deliberately vague, deliberately product-flavored, and deliberately collaborative: the interviewer plays the role of a product manager or engineering lead who keeps adding constraints, and the candidate must *ask* for requirements rather than assume them.

It is the ML analogue of the classic software **system design interview** ("design Twitter"), with one crucial difference: the artifact being designed is not just a set of services and databases, but a **learning system** — data pipelines, features, a model, training, serving, evaluation, and monitoring that keeps the model correct as the world changes. A candidate who designs "a model with good accuracy" has failed the round even if the model choice is perfect; the round rewards candidates who design *the whole system around the model*.

Key properties of the round (all verified against published accounts of the FAANG loops):

- **Open-ended and under-specified.** The prompt is one or two sentences. The first test is whether you recognize what is missing (users, latency, scale, cost, success metric) and ask.
- **Product-first.** Every design decision must trace back to a business goal and a user experience. "Why does this metric matter?" is a question you should be able to answer for every metric you propose.
- **No single right answer.** Interviewers score the *process* — structure, trade-offs, depth, communication — not the final architecture. Two candidates can produce different valid designs.
- **Collaborative.** The interviewer will push back, add scale, change constraints ("now the catalog is 10× bigger", "now fraudsters adapt"). How you react to new constraints is part of the signal.
- **Prevalent at FAANG and ML-heavy companies.** Google (MLE rounds), Meta, Amazon, Apple, Netflix, Uber, Airbnb, and most applied-ML shops run an ML design round for ML engineer and applied scientist roles. *(Verified: Google MLE interview experience write-ups and Meta's published 4-phase ML design framework confirm the round exists at both; the exact frequency across the industry is anecdotal.)*

### 1.2 Why It Matters: The Roles

The round exists because the two roles it screens for spend most of their working life doing exactly this activity.

| Role | What they build | Why the ML design round fits | Typical interview mix |
|---|---|---|---|
| **ML engineer** | Production ML systems: pipelines, feature infra, serving, monitoring, retraining | The job *is* system design around models; coding and infrastructure depth are expected | Coding + ML design + (sometimes) ML fundamentals |
| **Applied scientist / research scientist (applied)** | Models that move product metrics: ranking, detection, prediction, personalization | The job is model choice + evaluation under product constraints; depth of ML theory is expected | ML fundamentals + ML design + coding (lighter) |

The two roles are scored slightly differently on the same rubric: the **ML engineer** is expected to go deeper on serving, scale, and infrastructure (can you actually ship this?); the **applied scientist** is expected to go deeper on modeling, loss functions, and evaluation (is this the right model, and how do you *know*?). A strong candidate calibrates depth to the role they are interviewing for.

### 1.3 The Structure of the Interview

Across companies the round follows the same five-phase arc. (Verified against published Google/Meta interview accounts and the ByteByteGo framework; naming varies by company but the arc is stable.)

1. **Problem → clarification.** The interviewer states the prompt; you ask scoping questions: what does the product do, who are the users, what are the success metrics, what are the constraints (latency, scale, cost, hardware), what data is available, is this new or improving an existing system?
2. **Requirements → restatement.** You restate the problem in your own words, convert business goals into ML objectives, define the prediction task (regression/classification/ranking/retrieval), and agree on the evaluation metric *before* touching models.
3. **Design → the meat.** You walk the framework: data → features → model → training → serving → evaluation → monitoring, with the interviewer probing each.
4. **Deep-dive.** The interviewer picks one component and pushes for depth: "you mentioned in-batch softmax — what's wrong with it?", "how would you compute this feature at serving time?", "walk me through your A/B test and its pitfalls."
5. **Follow-ups.** Scaling questions ("10× users"), edge cases ("new users have no history", "fraudsters adapt"), and robustness questions ("how do you detect the model has gone stale?").

### 1.4 The Format Table

| Aspect | Description |
|---|---|
| Duration | 45–60 minutes (some companies: 30-minute variant for junior roles) |
| Format | One interviewer (sometimes two, one scoring depth, one scoring communication); whiteboard or shared doc |
| Prompt style | One-sentence product problem: "design X for Y with Z users" |
| Deliverable | A structured verbal design with a diagram, metrics, and explicit trade-offs — not code |
| Candidate activity mix | ~30–40% asking questions, ~60–70% designing and explaining |
| Interviewer role | Constraint-giver and stress-tester: adds scale, changes requirements, asks "why" |
| Scoring | A rubric across ML depth, system thinking, communication, trade-offs (Section 2) |
| What kills a candidate | Jumping to the model, ignoring serving, no metrics, no clarifying questions, one-way monologue |

### 1.5 The Prompt Bank (what you'll actually be asked)

Every ML design prompt in the wild is a variation of one of the families below. Before the interview, practice at least one design per family so the pattern-match is instant:

| Prompt family | Example prompts | Metric to anchor | Model core |
|---|---|---|---|
| Recommendation | "Design movie/YouTube-style recommendations" | Engagement, retention, catalog coverage | Two-tower retrieval + ranking funnel |
| Search | "Design search ranking for an e-commerce site" | Query success, NDCG, CTR | LTR (pairwise → listwise) |
| Feed | "Design a news feed" | Watch time, session value | Retrieval → ranker → diversity policy |
| Ads | "Design a CTR prediction system" | Revenue, ROAS, calibration | Wide & deep / DCN pCTR |
| Fraud/risk | "Design fraud detection for payments" | Net loss reduction, recall@fixed-FPR | GBDT + rules hybrid, real-time |
| Content moderation | "Design a system to detect harmful content" | Precision@recall target, takedown SLA | Classifier + LLM reviewer + queue |
| Forecasting | "Design demand forecasting for retail" | WAPE, inventory cost | GBDT with seasonality features |
| Personalization | "Design push-notification optimization" | CTR uplift vs annoyance | Response prediction + throttling |
| Churn/LTV | "Design churn prediction" | Retention lift, LTV | GBDT / survival-style |
| LLM-era | "Design an LLM support assistant" / "RAG for internal docs" | Answer quality, cost, latency, deflection rate | Retrieval + LLM (see Section 9.7) |

### 1.6 Company Flavors (anecdotal, unverified)

The same round is dressed differently by company, and calibrating to it matters: **Meta** runs a highly collaborative 45-minute design with a published 4-phase structure (scoping → data/features → model → serving → eval/monitoring) and scores heavily on asking questions; **Google** MLE rounds blend ML design with systems design and expect both to a high bar, with the rubric split across ML depth, systems, and communication; **Amazon** adds leadership-principle flavor (bias for action — "what would you ship first?") and often includes a follow-up on cost; **startups** tend to run it as a design review of a real problem their team faces, where pragmatic right-sizing beats breadth. None of this is officially published in detail — treat it as interview folklore to verify with current employees — but the framework in Section 3 works for all of them.

## 2. The Evaluation Criteria

### 2.1 The Four Criteria

Interviewers (and the published rubrics from ByteByteGo, Exponent, and company loops) consistently evaluate four dimensions. (Verified: these four — ML depth, system thinking, communication, trade-offs — appear in the widely circulated Meta/Google ML-design scoring guides; the exact weights are company-specific and unverified.)

**ML depth** — Do you actually understand the models you propose? Not just names: loss functions, failure modes, why this model fits this data, and the alternatives you rejected. Name-dropping "two-tower" without explaining the in-batch softmax and its bias is a negative, not a positive. Depth is demonstrated through *specificity*: exact loss, exact metric formula, exact failure mode.

**System thinking** — Do you design the whole loop: where data comes from, how features are computed online *and* offline, how the model is served within the latency budget, how you know it's degrading, how you retrain? The signal is *connections*: every component you mention should link to at least one other (features → serving latency, labels → retraining cadence, drift → alerting → retraining).

**Communication** — Do you drive the conversation with structure ("let me cover data, then features, then the model") instead of free-associating? Do you use the interviewer's constraints and restate them? Do you explain the *why* behind every choice? The interview is collaborative — a candidate who talks at the interviewer for 45 minutes and a candidate who asks nothing both fail.

**Trade-offs** — Do you recognize that every design decision is a compromise, and can you articulate the cost of your choice? "I chose X over Y because … at this scale, and the cost is …" is the sentence pattern interviewers are listening for. Trade-offs are scored on *identification* (seeing the tension), *articulation* (naming the cost), and *resolution* (picking with justification).

### 2.2 The Criteria Table

| Criterion | What they look for | How to demo it |
|---|---|---|
| ML depth | Correct model choice, exact losses/metrics, failure modes, awareness of alternatives | Say "I'd use logistic regression on engineered features as a baseline, then GBDT; for the deep model I'd train with focal loss because of the 99:1 imbalance — and here's the label noise problem I'd have to handle" |
| System thinking | End-to-end loop: data → features → model → serving → eval → monitoring, with the components connected | Draw the full diagram before going deep; say "this feature is computed at request time from the online store, and the same definition must be used in training to avoid train/serve skew" |
| Communication | Structure, restatement, collaboration, "why" for every choice | Open with "let me clarify requirements first, then I'll walk data → features → model → serving, and we can deep-dive anywhere"; check in with the interviewer every 3–5 minutes |
| Trade-offs | Tensions named, costs stated, choices justified | "GBDT gives better offline AUC but 5× the p99 latency of logistic regression — at our 100ms budget I'd take the regression to start, with a plan to move to a distilled GBDT" |

A useful self-test: after the interview, a friend should be able to reconstruct (a) the problem, (b) the metric, (c) the model and why, (d) the serving architecture, and (e) at least three trade-offs you named — from your side of the conversation alone.

### 2.3 How the Round Is Scored in Practice

Interviewers do not add points per component; they form a *holistic* signal per criterion and then look for the weakest one. Three practical consequences: (1) **A strong framework run with modest depth beats a deep model discussion with no structure** — the framework is what makes depth *usable*; (2) **one glaring miss can sink the round** — a candidate who designs a beautiful recommendation system but never mentions serving latency has failed the system-thinking criterion regardless of everything else; (3) **the deep-dive is where the score separates** — two candidates can both say "two-tower"; the one who can explain the in-batch softmax bias, the negative-sampling choice, and the cold-start fallback is the one who gets the senior rating. The rubric is symmetric with the criteria table above: interviewers *listen for* ML depth, *watch for* system thinking, *feel* communication, and *probe for* trade-offs.

## 3. The Framework

### 3.1 The Published Frameworks (Verified)

There is no single official framework, but two published frameworks dominate interview-prep material, and they agree on the same eight components in slightly different orders:

- **Chip Huyen — "Introduction to Machine Learning Interviews" (2022) and "Designing Machine Learning Systems" (O'Reilly, June 2022).** The interview book's ML systems design chapter lays out a scoping-and-design process; the O'Reilly book (and her Stanford CS 329S course, *Machine Learning Systems Design*) provides the deep reference on data, features, training, serving, and monitoring — including the "data-centric" argument that most ML failure is data failure, and the train/serve skew taxonomy. *Verified: both books exist with these titles and dates; CS 329S materials are public.*
- **Alex Xu & Ali Aminian — "Machine Learning System Design Interview" (ByteByteGo, published January 2023, 284 pages).** *Verification flag: the brief for this guide said "2024"; the verified publication date is January 2023 (ISBN 1736049127, Byte Code LLC).* The book is the closest thing to a standard playbook: a **7-step framework** (1 understand the problem and requirements, 2 prepare the data, 3 feature engineering, 4 model selection, 5 model training, 6 model evaluation, 7 deployment and serving) plus a bank of ~15 case studies (recommendation, search, feed, ads, fraud, etc.) with concrete numbers (latency budgets, model families, metric choices). The book's own blurb — "machine learning system design interviews are the most difficult to tackle of all technical interview questions" — is a reasonable summary of its reputation.
- **Company frameworks.** Meta publishes (via its interview guides and coach material) a 4–5 phase structure — problem scoping and requirements; data pipeline and features; model selection and training; serving, inference, and scaling; evaluation, monitoring, and iteration — and stresses that the round is *collaborative*. Google's MLE rounds follow the same arc. *(Verified: the Meta 4-phase framing appears in public interview guides; exact internal rubrics are unverified.)*

### 3.2 The Framework Steps

The eight-component framework used in this guide is the union of the above — it is what every published framework reduces to:

1. **Clarify the requirements** — users, business goal, constraints, scale, latency, success metric. (3–5 minutes.)
2. **Data** — what data exists, what's missing, labels, volume, quality, leakage risks. (5–7 minutes.)
3. **Features** — raw signals → features; offline vs online computation; feature store; skew. (5–7 minutes.)
4. **Model** — task type, baseline, candidate model families, loss, why this model. (5–7 minutes.)
5. **Training** — data split, sampling, training pipeline, cadence, experiment infra. (3–5 minutes.)
6. **Serving** — batch vs online vs real-time; latency budget; infrastructure; scaling. (5–7 minutes.)
7. **Evaluation** — offline metrics, baselines, error analysis; online A/B, guardrail metrics. (3–5 minutes.)
8. **Monitoring** — data/concept drift, model decay, alerting, retraining triggers, feedback loops. (3–5 minutes.)

**The two golden rules.** (1) *Order is signal* — going straight to the model is the #1 failure mode; the framework exists to force data-first, metric-first thinking. (2) *Time-box it* — you have ~40 minutes of design time; the table below is the budget that keeps you from over-investing in step 4 at the expense of steps 7–8, which is where seniors separate from juniors.

### 3.3 The Framework Table

| Step | Activities | Time budget |
|---|---|---|
| 1. Clarify | Scope users/goal/metrics/constraints; restate; agree the metric | 3–5 min |
| 2. Data | Sources, volume, labels, quality, leakage, class balance | 5–7 min |
| 3. Features | Signal list, transformations, online vs offline, feature store | 5–7 min |
| 4. Model | Task type, baseline, model family, loss, why | 5–7 min |
| 5. Training | Splits, sampling, pipeline, cadence, experimentation | 3–5 min |
| 6. Serving | Batch/online/real-time, latency budget, infra, scaling | 5–7 min |
| 7. Evaluation | Offline metrics + baselines + error analysis; A/B + guardrails | 3–5 min |
| 8. Monitoring | Drift, decay, alerts, retraining, feedback loops | 3–5 min |

### 3.4 The Published Frameworks Compared

| Source | Year (verified) | Shape | Strengths | Best use in prep |
|---|---|---|---|---|
| Chip Huyen, *Designing Machine Learning Systems* (O'Reilly) | June 2022 | Book, lifecycle-organized: data → features → training → serving → monitoring | Deepest on data-centric design, train/serve skew, and production failure modes | Weeks 1–2 depth reading; the "why" behind each step |
| Chip Huyen, *Introduction to Machine Learning Interviews* | 2022 | Book with an ML-systems-design chapter and question bank | Interview-focused scoping; ties design to interview answers | Framework vocabulary and practice questions |
| Alex Xu & Ali Aminian, *Machine Learning System Design Interview* (ByteByteGo) | January 2023 | 7-step framework + ~15 worked case studies with concrete numbers | Interview-shaped: step order, latency budgets, metric choices per case | The primary interview playbook; case-study rehearsal |
| Meta's published ML-design guidance | 2024–2025 era | 4–5 phase structure, collaborative framing | Company-specific: scoping first, data second, model third | Calibrating to one company's style |

Takeaway: the sources disagree on packaging (7 steps vs 8 components vs 4 phases) and agree on substance — clarify, data, features, model, training, serving, evaluation, monitoring. Pick one framework as your spine (this guide's eight steps), and use the others to check you missed nothing.

---

## 4. The Data

### 4.1 The Data Requirements

Data is where ML designs live or die, and interviewers know it. Chip Huyen's central argument — *most ML failures are data failures, not model failures* — has become interview orthodoxy: candidates who treat data as a given ("we'll just collect clicks") fail; candidates who treat it as the riskiest component score. *(Verified: Huyen's "Designing Machine Learning Systems" (2022) makes the data-centric case explicitly.)*

The questions to answer in this step:

- **Task and label definition.** What exactly are we predicting? For fraud: "is this transaction fraudulent" needs a label definition — confirmed fraud (chargeback, investigation outcome) vs suspicious-then-cleared. Label *definitions* and label *timing* (fraud is only confirmed weeks later) are design decisions, not trivia.
- **Volume and coverage.** How many examples per day/week? How long is history? Cold-start entities (new users, new items, new merchants) — do they have any data at all?
- **Quality.** Missing values, noisy labels, duplicate events, inconsistent schemas across sources. A data-quality failure mode named specifically beats a generic "clean the data".
- **Legal and regulatory constraints.** In banking: GDPR/MAS data protection, retention limits, explainability requirements (the EU AI Act's risk tiers), cross-border data flow. For a banking audience this is a differentiator — name it early.
- **Feedback loops.** Model outputs change future data (recommenders influence what users click; fraud models influence what fraudsters attempt). This is a *data* problem as much as a monitoring problem.

### 4.2 The Data Sources

| Source type | Examples | Typical use in the design |
|---|---|---|
| User/entity profile data | Demographics, account age, KYC tier, merchant category | Static features; entity resolution |
| Behavior/event data | Clicks, views, transactions, logins, searches | The label source and the richest signal; usually event-streamed (Kafka/Kinesis) |
| Historical/label data | Past outcomes: chargebacks, conversions, purchases | Training labels; requires careful label timing |
| Third-party/enrichment | Fraud consortiums, credit bureaus, device fingerprints | Cold-start and risk signals; cost and latency per lookup |
| Model-output data | Scores, recommendations, decisions from prior models | Features for later models; feedback loops |

The design should name *which* sources feed training (offline, batch, historical) versus serving (online, low-latency lookups), and flag which sources are hard to get at request time.

### 4.3 The Data Challenges

**Class imbalance** — fraud is the canonical case: 0.01–1% of transactions are fraudulent, so accuracy is a useless metric (99.99% accuracy by predicting "not fraud"). The design must switch to precision/recall/PR-AUC, and handle the imbalance honestly: resampling (undersample majority, oversample minority with care — SMOTE synthetic examples are a red flag in fraud because they don't resemble real fraud), cost-sensitive learning (weight the loss), or just keeping the imbalance and thresholding the score. The *interviewer's* follow-up is almost always "accuracy is 99.9% — is that good?" — and the answer is "no, because the base rate is 0.1%; I care about recall at a fixed precision, or expected loss".

**Label leakage** — the highest-value data insight in the round. Leakage is information in the training features that would not be available at prediction time, or that encodes the label. Classic examples: (1) *temporal leakage* — using "number of transactions in the past 30 days" computed including the transaction being scored; (2) *target leakage* — a feature like "fraud flag added by the risk team" that exists in the warehouse but not at request time; (3) *ID leakage* — an account/device ID that the model memorizes because each ID appears once with one label. Demo it by saying: "I'd validate with a time-based split — train on months 1–11, test on month 12 — and I'd audit every feature for whether it would exist in the request-time context."

**Other challenges worth naming**: label delay (fraud confirmed at T+30 days → labels for today's training set are incomplete), non-stationarity (fraud patterns, user tastes, and ad markets drift), duplicate/contradictory labels (transaction flagged fraud by one system, cleared by another), and data availability at cold start (new users, new devices).

### 4.4 The Data Table

| Aspect | Design questions | Common failure |
|---|---|---|
| Task & labels | What's the prediction target? How are labels defined, and when are they available? | Vague label ("fraud") with no operational definition or timing |
| Volume & coverage | Examples per day, history length, cold-start entities | Ignoring new users/new items entirely |
| Quality | Missing values, noise, schema drift, duplicate events | "We'll clean it" with no specifics |
| Balance | Base rate of the positive class; how to handle | Optimizing accuracy on a 0.1% base-rate problem |
| Leakage | Is every feature available at request time? Time-based split? | Temporal or target leakage inflating offline metrics |
| Governance | Retention, privacy, explainability, cross-border rules | Ignoring regulation in a regulated domain (banking!) |

### 4.5 The Leakage Audit Checklist

Leakage is the single highest-value data insight in the round, so make the audit concrete — run this ten-point checklist aloud and interviewers will visibly upgrade their score:

1. **Request-time availability** — for every feature, "would this value exist in the serving context at scoring time?" If it comes from a post-hoc warehouse job, it leaks.
2. **Window exclusion** — "transactions in the last 30 days" must exclude the transaction being scored; off-by-one window bugs are the most common temporal leak.
3. **Label-derived fields** — any column added by the risk/investigation team after the event (e.g. a fraud flag, an account status update) is target leakage unless it is provably available at request time.
4. **High-cardinality IDs** — raw account/device IDs can memorize one-label-per-ID; use hashed/embedded versions only after verifying no ID appears with contradictory labels.
5. **Time-based split** — train on months 1–11, test on month 12; random splits hide temporal leakage and overstate performance on drifting problems.
6. **Attack-window backtest** — hold out a known fraud attack period and check the model would have caught it; this is the fraud-domain equivalent of a robustness test.
7. **Point-in-time joins** — historical features must be joined on the state *at event time*; the feature store's point-in-time correctness exists precisely to prevent "today's value leaked into yesterday's training row".
8. **Whole-period aggregations** — "average over the entire dataset" features (including the future) are leakage; aggregate per event-time window only.
9. **Imputation leaks** — filling missing values with global/column statistics computed on the full dataset leaks future information; fit imputers on the training window only.
10. **Duplicates** — the same event duplicated across sources can appear in both train and test; dedupe by event ID before splitting.

## 5. The Features

### 5.1 Feature Engineering

Feature engineering converts raw data into model inputs, and the interview tests whether you can do it under the constraint that **the same feature must be computable offline (training) and online (serving)**.

The signal categories to brainstorm per entity (user, item, transaction, context):

- **Identity/static** — account age, country, merchant category, device type.
- **Behavioral/historical** — counts, rates, and recency-weighted aggregates: "transactions in last 24h/7d/30d", "average ticket size", "velocity" (fraud's favorite: count of distinct merchants in an hour).
- **Interaction/contextual** — time of day, channel, IP reputation, session features.
- **Cross-entity** — user×merchant history ("has this user ever transacted with this merchant?"), user×device.
- **Model-derived** — embeddings from a prior model, scores from a rules engine, graph features (degree in the fraud network).

Interview-worthy transformation points: **recency weighting** (exponential decay over time windows beats raw counts), **log transforms** for heavy-tailed amounts, **binning/one-hot vs embeddings** for high-cardinality categoricals (merchant ID is high-cardinality → embed or hash, don't one-hot), and **temporal windows that match the serving context** (a 30-day count feature must be computable at request time from the online store).

The deep-dive follow-up is usually **train/serve skew**: if training computes "user's spend in last 30 days" from a nightly batch job and serving computes it from a different code path at request time, the two values drift apart — the classic fix is a **feature store** that computes features once with a shared definition and serves both paths.

### 5.2 The Feature Stores

A **feature store** is the platform component that makes train/serve consistency tractable: a feature registry (definitions as code), an offline store (historical features for training, with point-in-time correctness so features join on the *state at event time*, not today's state), an online store (low-latency lookups for serving), and monitoring per feature (drift, staleness, missing rates). Tools: Feast, Tecton, Databricks Feature Store, SageMaker Feature Store, and cloud-native equivalents.

Interview framing: you don't need to design a feature store from scratch — you need to *invoke* it correctly: "I'd register features in the feature store with one definition; training reads historical values from the offline store with point-in-time joins; serving reads the online store within the latency budget; and staleness/drift per feature is monitored." Mentioning point-in-time correctness is a strong senior signal — it's the #1 silent bug in real feature pipelines. *Cross-ref: `ml_platforms_comparison_guide.md` for the platform comparison (Feast/Tecton/cloud feature stores, ML platforms end-to-end); `schema_evolution_data_drift_guide.md` for feature-level drift monitoring.*

### 5.3 The Feature Table

| Feature class | Examples | Serving note |
|---|---|---|
| Static/identity | Account age, KYC tier, merchant category | Read from profile DB at request time |
| Behavioral/velocity | Tx count last 24h/7d, distinct merchants/hour, avg ticket | Precomputed in the online store; window must match request context |
| Contextual | Time of day, channel, IP reputation | Computed at request time; some need external lookups (latency!) |
| Cross-entity | User×merchant history, user×device | Join at request time or precomputed aggregates |
| Model-derived | Embeddings, rules-engine scores, graph degrees | Must be available *before* scoring; versioning matters |

## 6. The Model

### 6.1 Model Selection: Start with the Baseline

The model step has a strict order that interviewers reward: **task type → baseline → candidate families → loss → justification → what you'd actually ship.**

- **Task type first.** Is this classification (fraud: binary), regression (LTV: continuous), ranking (search: list of docs), retrieval (recommendation: top-K from a huge corpus), or a hybrid funnel (retrieval → ranking)? The task type dictates the metrics and the loss, and it's the first thing to state out loud.
- **The baseline is non-negotiable.** Every design should open the model discussion with a baseline: a **heuristic/rules baseline** (fraud: amount > threshold + velocity > N; recommendation: most popular items; search: BM25) and/or a **simple ML baseline** (logistic regression on the engineered features). Baselines are how you *prove* your fancy model adds value — you cannot claim "two-tower beats everything" without a bar to beat. This also buys you signal: "our rules baseline already catches 60% of fraud at 1% FPR; the ML model needs to beat *that*."
- **Match the model family to the data shape.** Small structured feature sets with tabular data → logistic regression, then GBDT (XGBoost/LightGBM) — still the workhorse for tabular fraud/ad CTR problems. High-cardinality sparse categoricals + large data → deep models (wide & deep, DCN, Transformers for sequences). Sequences → LSTM/Transformer encoders. Massive corpora with retrieval → two-tower embeddings (Section 9.1). LLM-era: for text-heavy products, consider whether an LLM component is justified at all — usually it's not for ranking/detection at latency budgets, and saying so is a senior answer.
- **Name the loss.** Logistic loss (binary), focal loss (imbalance), pairwise/listwise ranking losses (LTR), softmax/in-batch softmax with temperature (two-tower retrieval), weighted MSE (regression with heterogeneous noise).
- **Justify with the trade-off.** "I'd start with logistic regression because it's interpretable, cheap, and debuggable; move to GBDT when we need interactions; go deep only if data volume justifies it — the cost is latency and interpretability, and the benefit is interaction modeling."

### 6.2 The Model Choices

The families an ML-design candidate must be able to place correctly: **linear models** (LR, ridge — baselines, interpretability, calibration), **tree ensembles** (GBDT, Random Forest — tabular SOTA, feature interactions, no normalization needed, hard to serve at extreme QPS but fine with ONNX/compiled runtimes), **deep models** (MLPs, wide & deep, DCN — sparse categoricals; Transformer/attention — sequences; two-tower — retrieval), **graph models** (GNNs — fraud networks, social graphs; expensive, need graph infra), **nearest-neighbor/embedding retrieval** (ANN: FAISS/ScaNN/HNSW — candidate generation), and **LLMs** (text understanding, agents; latency/cost heavy — justify before using).

### 6.3 The Model Table

| Model | Use case | Trade-off |
|---|---|---|
| Logistic regression | Baseline for fraud/CTR; interpretable, calibratable | No feature interactions; linear only |
| GBDT (XGBoost/LightGBM) | Tabular classification/regression (fraud, churn, LTV) | SOTA on tabular; slower inference than LR; needs care with categoricals |
| Wide & deep / DCN | Ad CTR, recommendations with sparse features | Strong on sparse categoricals; more infra than GBDT |
| Two-tower (dual encoders) | Candidate retrieval over millions of items | Fast ANN serving; retrieval quality limits ranking ceiling |
| Transformer / sequence models | User sequences, text, session data | Expressive; heavy compute; needs lots of data |
| GNN | Fraud networks, social/similarity graphs | Captures relational signal; graph infra complexity |
| LLM (fine-tuned/prompted) | Text products, content understanding | Powerful; latency/cost; hallucination risk; overkill for tabular |

### 6.4 The Loss Function Cheat Sheet

Naming the loss is the fastest way to demonstrate ML depth — the loss encodes the *objective*, and the objective encodes the business problem:

| Problem shape | Loss | Why it fits |
|---|---|---|
| Binary classification, balanced | Binary cross-entropy (log loss) | Standard; well-calibrated probabilities with enough data |
| Binary classification, imbalanced | Weighted BCE / focal loss | Down-weights the easy majority class; focal loss is the standard for extreme imbalance (fraud, rare events) |
| Multi-class | Softmax cross-entropy | Standard; pair with temperature for retrieval-style softmax |
| Retrieval (two-tower) | In-batch softmax with log-Q correction | Treats batch negatives as a sampled softmax; the correction fixes popularity bias |
| Ranking (pairwise) | RankNet/LambdaRank hinge-style losses | Optimizes pairwise preference; LambdaMART on GBDTs was the pre-deep SOTA |
| Ranking (listwise) | ListNet/ListMLE, softmax over positions | Optimizes the list metric directly; modern neural rankers |
| Regression | MAE / RMSE / Huber | MAE robust to outliers; RMSE penalizes large errors; Huber is the hybrid |
| Time-to-event / churn | Survival loss (Cox partial likelihood) | Uses censored data (customer hasn't churned *yet* but might) instead of discarding it |
| Quantile targets | Pinball loss | Learns conditional quantiles for interval forecasts and risk-style decisions |

The interview move: pick the loss *and* say what it costs — "BCE with class weights is simple and debuggable; focal loss squeezes more recall on the minority class but adds a hyperparameter to tune" — then move on.

## 7. The Training and the Serving

### 7.1 The Training Pipeline

The training step covers how the model is *built and refreshed*, and interviewers use it to test production thinking:

- **Data split.** Random vs time-based. Random splits overstate performance when the world drifts — for fraud, recommendation, and ads, use **temporal splits** (train on months 1–11, validate/test on month 12) and be ready to explain why. For ranking, split on *queries*, not rows, to avoid the same query leaking across train/test.
- **Sampling.** Class imbalance handling (Section 4.3): subsample negatives, weight the loss, or stratify. For retrieval training, negative sampling strategy (random, in-batch, hard negatives) is a design decision with real consequences.
- **Pipeline machinery.** Feature computation → join → validation (schema checks, drift checks) → training → experiment tracking (MLflow/W&B) → model registry → evaluation gates → promotion. *Cross-ref: `mlops_lifecycle_frameworks_guide.md` for the full lifecycle framing; `ml_platforms_comparison_guide.md` for the platforms that automate it.*
- **Training cadence and retraining strategy.** Batch retraining nightly/weekly (cheap, lags drift), continuous/streaming retraining (fast, complex), or online learning (per-example updates — good for fraud where patterns shift in hours; needs strong monitoring). The interview answer should tie cadence to *drift speed*: "fraud patterns shift in days, so I'd retrain nightly, and trigger ad-hoc retraining when drift monitoring fires."
- **Experimentation infrastructure.** How do you know a new model is better? Offline eval gates, shadow deployment, then A/B (Section 8.2). The training step is where "how do you improve this later" gets answered.

### 7.2 The Serving

Serving is the step juniors skip and seniors lead with — the model must produce a prediction within a latency budget at production scale. Three patterns:

- **Batch (offline) serving.** Precompute scores on a schedule — e.g., nightly recommendations per user written to a cache/DB. Pros: cheap, simple, can use heavy models. Cons: stale by definition (a fraud model scored yesterday can't catch a pattern from this morning; recommendations miss real-time context).
- **Online (request-time) serving.** Score at request time with a model server (TensorFlow Serving, TorchServe, Triton, ONNX Runtime) behind the API. Pros: fresh features, fresh model. Cons: latency budget (typically p50 < 30–50ms, p99 < 100–200ms for interactive products), feature lookups must be fast (online feature store, caching), model must fit the budget (compression: quantization, pruning, distillation — *cross-ref: `knowledge_distillation_guide.md`*).
- **Real-time / streaming.** Near-real-time scoring on event streams (Kafka → Flink → model) — the fraud pattern: score every transaction in milliseconds as it happens, with the model server embedded in the stream processor or called over RPC. Pros: catches the current attack. Cons: most complex infra, hardest to test.

The canonical interview answer shows the **latency budget breakdown**: 100ms total → 20ms feature lookups (cached), 5ms preprocessing, 40ms model inference, 15ms post-processing/rules, 20ms headroom. Then: "at this QPS I'd need N replicas with autoscaling; the feature store is the bottleneck so I'd precompute and cache; if the model doesn't fit the budget I'd quantize or distill." That *specificity* is what scores.

### 7.3 The Serving Patterns (architecture detail worth naming)

- **Funnel / two-stage serving** — retrieval (ANN over embeddings, ~100–1000 candidates in ms) → ranking (heavier model on the candidates) → policy/post-processing (diversity, business rules, fairness constraints). Standard for recommendation, search, feed.
- **Rules + model hybrid** — hard business rules (regulatory blocks, minimum-risk overrides) around the model; model scores then threshold + rules. Very common in banking fraud/credit (explainability, audit).
- **Shadow deployment** — new model scores live traffic in parallel without affecting decisions; compare against the champion before cutover.
- **Caching layers** — precomputed scores for popular entities; cache invalidation on feature/model update.

### 7.4 The Training/Serving Table

| Aspect | Batch | Online (request-time) | Real-time (streaming) |
|---|---|---|---|
| When | Scheduled (nightly/hourly) | On request | On event (per transaction) |
| Latency | Minutes–hours | ms (p50/p99 budgets) | ms, embedded in the stream |
| Freshness | Stale by schedule | Feature-fresh at request | Freshest possible |
| Cost | Low per prediction | Medium (servers, scaling) | High (stream infra, ops) |
| Use cases | Recommendations, daily risk scores | Search/feed ranking, ad CTR | Fraud detection, real-time risk |
| Failure mode | Stale decisions | Latency spikes, feature misses | Infra complexity, hard testing |

### 7.5 The Scale Math (say numbers, don't wave hands)

Interviewers reward concrete scale reasoning. The two formulas that cover 90% of serving questions:

- **Concurrency = QPS × latency.** At 10K TPS and a 50ms p99 budget, in-flight requests ≈ 10,000 × 0.05 = 500. A 16-core replica handling ~1ms of CPU per request at 60% utilization sustains roughly 16 × 0.6 / 0.001 ≈ 9,600 QPS — so ~2 replicas cover peak with headroom, 3–4 for failover. Saying "I'd need a handful of replicas, here's the math" beats "I'd scale horizontally".
- **Latency budget = feature fetch + preprocessing + inference + post-processing.** A 50ms budget might split as 15ms cached feature lookups + 5ms preprocessing + 20ms inference + 10ms rules/response. If the model misses its slice, the fix is compression (quantization, distillation — *cross-ref: `knowledge_distillation_guide.md`*), a cheaper model family, or moving work off the hot path (precomputed scores, cached embeddings).
- **Cost check.** Batch scoring 100M transactions nightly on CPU is cheap; real-time scoring at 10K TPS with GPU inference is not. Name the cost class of your choice ("CPU is fine for GBDT at this QPS") — it signals you have run these systems.

## 8. The Evaluation

### 8.1 The Offline Evaluation

Offline evaluation answers "is this model better than the baseline on historical data?" — and the interview tests whether you pick the metric that matches the *business* problem, not the metric that matches the library default.

- **Classification** — precision, recall, F1, PR-AUC (imbalanced: fraud), ROC-AUC (balanced), calibration error (ECE — does 0.9 mean 90%?). For fraud: **recall at a fixed precision** (or FPR) is the operating metric, because the business constraint is "no more than X% of alerts may be false positives" (alert fatigue, investigation cost).
- **Regression** — MAE, RMSE (penalizes large errors), MAPE, quantile losses (pinball loss for quantile regression).
- **Ranking/retrieval** — NDCG (position-weighted relevance), MAP, MRR, recall@K/precision@K (retrieval), plus business-aligned surrogates (CTR). For retrieval: recall@100 matters more than NDCG@10 — the ranking stage fixes the top-10.
- **The evaluation stack** — holdout with the *right split* (temporal!), baseline comparison (Section 6.1), **error analysis** (slice the errors: by merchant category, by channel, by time-of-day — which slice is the model worst on? this is where seniors shine), and calibration checks. Mention that offline gains must survive an A/B to count.
- **The trick question:** "offline AUC improved 0.01 — should we ship?" Answer: "not by itself; AUC is insensitive to operating points and doesn't capture business cost. I'd check PR-AUC/recall-at-fixed-precision, run error analysis, then A/B." *(Cross-ref: `llm_evaluation_frameworks_guide.md` and `llm_evaluation_vs_validation_guide.md` for the evaluation methodology lineage; `mathematics_for_ml_and_ds_study_notes.md` for the metric math.)*

### 8.2 The Online Evaluation (A/B Testing)

Online evaluation is where the design closes the loop: **A/B testing** the new model against the champion on live traffic, with the business metric as the decision criterion.

The interview must show you know A/B's ML-specific pitfalls:

- **The metric.** Primary = business metric (fraud loss in $, conversion, revenue per session, retention). Guardrail metrics = everything that must not regress (latency, error rate, non-target engagement). Secondary = model metrics (CTR, precision@k).
- **Sample ratio mismatch / SRM** — if the split is 50/50 but traffic isn't, something is broken (usually a serving bug — very common in practice; check it early).
- **Interference and network effects.** In feed/search/ads, one user's treatment changes *other* users' experiences (position of an ad changes who sees it; a recommender's output changes the ecosystem). If treatment bleeds into control, the A/B is invalid. Mentioning this is a senior signal. Mitigations: cluster-randomization, interleaving (for ranking: present interleaved results from both models to the same user — faster and lower variance).
- **Novelty effects & carryover** — new recommenders get a temporary engagement boost (novelty) that decays; users need time to adapt. Run the experiment long enough, or analyze by day.
- **Multiple comparisons** — running 20 A/Bs and shipping the one that wins by chance; correct with sequential testing or FDR control.
- **Statistical validity** — power analysis before launch (effect size you care about, sample size, duration), and always "the difference must clear the guardrails".

### 8.3 The Metrics: business metrics vs model metrics

The distinction interviewers probe: **model metrics** (AUC, NDCG, recall) measure the model; **business metrics** (fraud loss in $, revenue, retention, investigation cost per alert, user complaints) measure the product. Every model metric in the design should map to a business metric: "recall@0.1% FPR for fraud ↔ $ loss prevented minus investigation cost; NDCG@10 for search ↔ engagement and query success; CTR for ads ↔ revenue." A design that never names a business metric has not designed a product — it has fit a model. (This maps directly to the banking context: a fraud model is approved by the business on *expected loss reduction and false-positive cost*, not AUC — cross-ref `financial_risk_compliance_systems_guide.md`.)

### 8.4 The Evaluation Table

| Phase | Metrics | Use |
|---|---|---|
| Offline — classification | Precision, recall, F1, PR-AUC, ROC-AUC, ECE | Model selection, threshold setting, baseline comparison |
| Offline — regression | MAE, RMSE, MAPE, quantile loss | Model selection for continuous targets |
| Offline — ranking/retrieval | NDCG, MAP, MRR, recall@K, precision@K | Ranking model selection; retrieval quality |
| Offline — slices/error analysis | Per-segment metrics (channel, category, time) | Find where the model fails; prioritize data/features |
| Online — A/B | Business primary (fraud $, conversion, revenue), guardrails (latency, error rate) | Ship/rollback decision |
| Online — advanced | Interleaving, bandits, sequential testing | Faster/lower-variance decisions; adaptive exploration |
| Monitoring (ongoing) | Drift stats (PSI, KS), feature staleness, business KPI deltas | Detect decay, trigger retraining (Section 8.4 → Section 3.2 step 8) |

### 8.5 The Evaluation Pitfalls Checklist

Run this list when you finish any design; each item is a question interviewers love to ask because most candidates miss it:

1. **Metric mismatch** — AUC is not an operating-point metric; for fraud/risk use recall@fixed-FPR, for ranking use NDCG/recall@K, for probability-feeding-auctions use calibration error.
2. **Random split on temporal data** — always justify the split; temporal data wants temporal splits.
3. **No baseline comparison** — "improved 0.01 AUC" is meaningless without "vs the rules engine / LR / champion".
4. **No calibration check** — if scores feed decisions or auctions, ECE matters as much as ranking quality.
5. **SRM unchecked** — verify the treatment/control split actually holds on traffic logs before reading any result.
6. **Multiple comparisons** — every additional metric you peek at increases the chance of a false win; pre-register the primary metric.
7. **Novelty-short experiments** — engagement uplifts decay; run long enough or analyze by week.
8. **No guardrails** — a model that wins on the primary metric but regresses latency/error-rate/fairness is not shippable; guardrails are non-negotiable.
9. **Offline-only decisions** — offline gains are hypotheses; the A/B is the decision.
10. **Forgetting the business metric** — if the evaluation cannot be expressed in dollars/retention/conversion, it is not a product evaluation (Section 8.3).

---

## 9. The Case Studies

The five canonical case studies — recommendation, search ranking, feed ranking, ad prediction, fraud detection — cover the interview space between them (every prompt is a variation or combination). *(Verified: the ByteByteGo book covers all five as its core chapters; the "ML system design: 500 case studies" community repo and Exponent/DataInterview libraries confirm these are the standard set.)* Learn the *design logic* of each, not the answer: the two-tower story transfers to search retrieval, the LTR story transfers to feed, the calibration story transfers from ads to any probability-scoring product.

### 9.1 Recommendation (the Two-Tower)

The canonical design: a **two-stage funnel**. Stage 1 — **retrieval**: from a catalog of millions of items, generate a few hundred candidates per user. Stage 2 — **ranking**: score the candidates with a richer model and return the top-N.

The retrieval model is the **two-tower architecture** (verified as the dominant published pattern): a *user tower* encodes user features (history, profile, context) into an embedding; an *item tower* encodes item features into an embedding; relevance = dot product (or cosine) of the two. The towers are trained jointly with **in-batch softmax** — for each (user, item) positive pair, the other items in the batch act as negatives — the architecture popularized by YouTube's 2016 candidate-generation paper (*Deep Neural Networks for YouTube Recommendations*, Covington et al.) and formalized by Google's *Sampling-Bias-Corrected Neural Modeling for Large Corpus Item Recommendations* (Yi et al., 2019), which corrects the popularity bias in-batch softmax introduces. At serving time, item embeddings are precomputed and indexed in an **ANN store** (FAISS, ScaNN, HNSW), so retrieval is a fast nearest-neighbor lookup; the user embedding is computed live. *(Verified: this two-stage, two-tower + ANN pattern is the consensus architecture described across the published literature and practitioner write-ups; it powers YouTube, Spotify, and TikTok-class systems.)*

Interview depth points that score: in-batch softmax's popularity bias and the log-Q correction; negative sampling strategy (random vs hard negatives); two-tower recall@K as the retrieval metric (the ranking stage can only reorder what retrieval finds — retrieval quality caps the whole system); cold-start (popular-item fallback, content-based priors); and the feedback loop (the recommender shapes the user history that trains the next model — exploration is not optional).

### 9.2 Search Ranking (Learning-to-Rank)

Search is "given a query and a document set, return the best-ordered list" — a **learning-to-rank (LTR)** problem. The three classic LTR paradigms (verified, standard ML background):

- **Pointwise** — predict relevance score per (query, doc) independently, sort by score. Simple; ignores list context.
- **Pairwise** — learn to prefer the correct document over an incorrect one for the same query (RankNet, LambdaRank; LambdaMART = LambdaRank on GBDTs was the pre-deep-learning SOTA). Optimizes pairwise accuracy, a proxy for the list metric.
- **Listwise** — optimize the list metric directly (ListNet, ListMLE, softmax over positions; modern neural rankers with listwise losses).

The design: features (query-doc textual similarity via BM25/embeddings, doc quality/authority, click features, user context), a two-stage funnel (candidate retrieval via BM25 + embedding retrieval → neural ranker), offline metric NDCG/MRR (position-weighted), and the evaluation trap: **position bias** — clicks depend on position, so naive click-derived labels over-reward whatever was ranked first. Fixes: inverse propensity weighting (IPW) from the *Unbiased LTR* literature, or interleaved A/B testing online. The classic interview follow-up: "your offline NDCG went up but CTR didn't — why?" — answer: metric mismatch, position bias, or the novelty effect.

### 9.3 Feed Ranking

The feed (news feed / short video) is recommendation's close cousin with three twists: (1) **freshness** matters — the feed must surface new content, not just the globally-best items; (2) **diversity** — the top-10 by score alone is homogeneous (all sports, all news), so ranking includes a diversity/post-processing stage (MMR — maximal marginal relevance — or constraint-based re-ranking); (3) the objective is often **engagement (watch time, session value)**, not clicks — a *value model* (e.g., predicted watch time) that can trade off short clicks against long sessions. Architecture: retrieval (two-tower, per-user) → ranking (deep model on candidate features) → policy layer (diversity, freshness boost, hard rules). The design discussion should cover position bias in the labels, exploration, and the *guardrail metrics* (retention, content-type balance) that keep the engagement objective from gaming itself.

### 9.4 Fraud Detection (the Class Imbalance)

Fraud is the banking/risk canonical case (and the worked example in Section 12), characterized by **extreme class imbalance** (0.01–1% positive rate), **adversarial adaptation** (fraudsters change tactics when the model catches them), **label delay** (confirmed only after investigation/chargeback, days–weeks later), and **hard latency** (score the transaction in milliseconds, at the payment gateway, before approval). Design essentials: temporal validation, precision/recall operating point set by *cost* (investigation cost per alert vs loss per missed fraud), features built for velocity (Section 5.1), rules + model hybrid (Section 7.3) for auditability, and **concept drift monitoring + fast retraining** because the adversary moves. The metric answer that wins: "I optimize recall at a fixed false-positive budget set by investigation capacity, and I report expected loss reduction in dollars."

### 9.5 Ad Prediction

Ads = **pCTR (click-through) and pCVR (conversion) prediction** for auction bidding: the model predicts the probability of click/conversion, and the auction combines the probability with the bid to pick the ad. Design points: sparse high-cardinality features (user ID, ad ID, context) → wide & deep / DCN-style models; **calibration** is critical (pCTR feeds the auction economics — a miscalibrated 0.9 is a disaster even if AUC is great; measure ECE, recalibrate with Platt scaling or isotonic regression); *position bias* in click labels; *selection bias* (we only see clicks for ads that won the auction — the training distribution is not the serving distribution); and the feedback loop (winning ads get more exposure and more labels). Business metric: revenue/ROAS, with guardrails on user experience.

### 9.6 The Case Study Table

| Case | Model | Key design point |
|---|---|---|
| Recommendation | Two-tower retrieval + ranking funnel | In-batch softmax training; ANN serving; retrieval quality caps the system |
| Search ranking | LTR: pairwise (LambdaMART) → listwise neural | NDCG offline; position bias in labels; interleaving online |
| Feed ranking | Retrieval → deep ranker → diversity policy | Freshness + diversity post-processing; value model (watch time) not clicks |
| Fraud detection | GBDT/deep + rules hybrid | Class imbalance → recall@fixed-FPR; label delay; adversarial drift → fast retrain |
| Ad prediction | Wide & deep / DCN for pCTR/pCVR | Calibration (ECE) because scores feed auctions; selection bias; revenue metric |

### 9.7 LLM-Era Design Variants

2025+ prompts increasingly blend classic ML design with LLM components, and the framework absorbs them cleanly — the LLM is a *model choice*, not a new paradigm. The three recurring variants:

- **RAG-based assistant** ("design an LLM support assistant over internal documents") — clarify (deflection rate, answer quality, cost per conversation); data (documents, permissions, freshness); retrieval (embeddings + BM25 hybrid — *cross-ref: the `rag/` guides, e.g. `rag_optimization_techniques_guide.md`*); the LLM as the generator; evaluation (offline: retrieval recall, groundedness, LLM-as-judge; online: deflection rate, CSAT, cost); monitoring (retrieval drift, answer-quality sampling, cost per query). *Cross-ref: `llm_evaluation_frameworks_guide.md` and `llm_evaluation_vs_validation_guide.md` for the eval methodology.*
- **Content moderation** ("design harmful-content detection") — the classic classifier + queue pattern, upgraded with an LLM triage layer: cheap classifier screens, LLM reasons about borderline cases, humans review the queue; precision@recall target set by takedown policy; adversarial drift is the core problem.
- **The "when NOT to use an LLM" test** — the senior answer. For tabular fraud/ads/ranking at ms latency, an LLM is the wrong tool; saying "I'd use a GBDT here because the LLM buys nothing at 10× the latency and cost, and it can't explain a denial to a regulator" scores higher than force-fitting an LLM into every design. The framework's trade-off discipline (Section 10) is exactly what makes this call.

## 10. The Trade-offs and the Mistakes

### 10.1 The Accuracy vs Latency Trade-off

The most frequently probed tension: **better models cost more compute**. A GBDT that lifts AUC by 0.02 but blows the p99 latency budget is not an improvement. The interview answer pattern: name the budget (Section 7.2), pick the best model that fits, and keep an *upgrade path* — "start with the cheap model, and if we need the accuracy, we buy it back with distillation, quantization, or a two-stage funnel where the heavy model only scores a shortlist." Also name the *cost* side: GPU inference vs CPU inference changes the economics; a nightly batch job can afford a model a request-time path cannot. The senior signal is treating accuracy as one input to a constrained optimization (latency, cost, interpretability, data), not the objective itself.

### 10.2 The Complexity vs Benefit Trade-off

Every component you add — streaming pipeline, feature store, online learning, microservices — has a cost: engineering time, ops burden, failure modes. The rule interviewers reward: **add complexity only when the benefit is demonstrable** ("do we have evidence the nightly batch is losing money? if not, don't build streaming yet"). Concretely: rules baseline first, then a simple model, then complexity as *measured* need (the "measure → improve → measure" loop). Over-engineering (Kafka + Flink + online learning + GNN for a problem with 10K transactions/day) reads as inexperience; right-sizing reads as seniority. Related tensions worth naming: model complexity vs interpretability (regulated banking: explainability may be mandatory), personalization vs privacy, freshness vs stability (retrain too often → noisy models), and exploration vs exploitation.

### 10.3 The Common Mistakes

The failure modes are consistent across interview debriefs and prep material (verified thematically across ByteByteGo, Exponent, and interviewer write-ups; the exact rankings are anecdotal):

1. **Jumping straight to the model.** "I'd use a two-tower model" in the first minute, with no requirements, no metric, no data discussion. *Fix: the framework (Section 3) — clarify → data → features → model.*
2. **Missing the data discussion entirely.** No labels, no leakage, no imbalance, no data sources. *Fix: always spend the first third of the design on data; name leakage and the split explicitly.*
3. **Ignoring serving.** Designing a model that cannot meet the latency budget or the scale. *Fix: state the latency budget and serving pattern for every design.*
4. **No evaluation plan.** Never says how to know it works. *Fix: offline metric + A/B with guardrails, always.*
5. **No baseline.** Claims "deep learning" without a bar to beat. *Fix: rules + LR baseline first (Section 6.1).*
6. **No trade-offs.** Every choice presented as free. *Fix: use the "X over Y because …, at the cost of …" pattern.*
7. **No business metric.** Metrics stop at AUC/accuracy. *Fix: map every model metric to a business metric (Section 8.3).*
8. **Monologue.** Talks for 45 minutes, ignores the interviewer's constraints. *Fix: restate constraints, check in every 3–5 minutes, invite deep-dives.*
9. **Ignoring monitoring and retraining.** The model is a one-time artifact. *Fix: close the loop — drift → alert → retrain (Section 3.2 step 8).*
10. **Unstructured rambling.** Jumps between components. *Fix: announce the structure ("data first, then features, then model"), keep the time budget.*

### 10.4 The Mistakes Table

| Mistake | Why it fails | The fix |
|---|---|---|
| Jumping to the model | Shows no product/system thinking; skips the metric | Run the framework in order; clarify first |
| Missing the data | Data failure is the #1 real-world ML failure mode | Data step: sources, labels, leakage, imbalance, split |
| Ignoring the serving | The model can't run at the required latency/scale | Latency budget + serving pattern for every design |
| No evaluation plan | Cannot prove value; cannot ship decisions | Offline metrics + baselines + A/B with guardrails |
| No baseline | Cannot attribute gains to the model | Rules/LR baseline before any fancy model |
| No trade-offs | Looks like the candidate doesn't understand costs | "X over Y because …, at the cost of …" |
| No business metric | Designing a model, not a product | Map model metrics → business metrics ($, retention, conversion) |
| Monologue | Fails the collaborative test | Restate constraints; check in; ask what to deep-dive |
| No monitoring/retraining | The system decays and no one knows | Drift monitoring → alerting → retraining loop |
| Unstructured rambling | Cannot follow the reasoning | Announce structure; time-box each step |

### 10.5 The Trade-off Table

The tensions below cover nearly every follow-up question in the round. For each, the interview answer is the same shape: *name the tension, pick a side for this problem, state the cost*:

| Trade-off | The tension | The senior resolution pattern |
|---|---|---|
| Accuracy vs latency | Better models cost more compute | Budget first; pick the best model that fits; buy accuracy back with distillation/quantization or a two-stage funnel |
| Complexity vs benefit | Every component adds ops cost | Rules → simple model → complexity only when the benefit is *measured* (right-sizing) |
| Model complexity vs interpretability | Deep models are opaque; regulated domains need answers | Hybrid: simple model + SHAP where explanations are mandatory; deep only where they aren't |
| Freshness vs stability | Retrain too often → noisy models; too rarely → stale | Tie cadence to measured drift speed; gate retraining on eval-vs-champion |
| Personalization vs privacy | More user data personalizes better | Data minimization, differential privacy, on-device options; name the regulation |
| Exploration vs exploitation | Pure exploitation converges to local optima | ε-greedy/bandits on a traffic slice; guardrail the exploration |
| Batch vs real-time | Freshness vs cost and complexity | Choose by the decision's time-sensitivity; upgrade when the batch measurably loses money |
| Precision vs recall (risk) | Blocking too much costs customers; blocking too little costs losses | Set the operating point from *costs* (investigation/friction vs fraud loss), not from the curve |
| Single model vs ensemble | Ensembles win accuracy, cost ops | Ensemble only where the accuracy is worth the serving cost; distill afterwards |
| Own infra vs buy platform | Control vs speed | Start on the platform, extract when the constraint (latency, cost, customization) bites |

## 11. The Practice Plan

### 11.1 The Plan: 4–6 Weeks

The consensus prep duration is **4–6 weeks** alongside a job search (verified as the standard recommendation in prep guides; exact weeks vary by background — a practicing ML engineer needs less, a generalist needs more). Structure: weeks 1–2 build the toolkit, weeks 3–4 drill cases, weeks 5–6 simulate the real thing.

- **Week 1 — The framework and the vocabulary.** Internalize the eight steps and the time budget (Section 3.3). Learn the metric formulas cold (precision/recall/F1/AUC/PR-AUC/NDCG/MRR/ECE) and the loss functions (logistic, focal, softmax, pairwise). *Cross-ref: `mathematics_for_ml_and_ds_study_notes.md` for the metric math.* Read one book: Chip Huyen's *Designing Machine Learning Systems* (depth) or the ByteByteGo *Machine Learning System Design Interview* (interview-shaped). Learn the two-tower and LTR mechanics until you can explain in-batch softmax and LambdaMART without notes.
- **Week 2 — The components.** One component per day: data/leakage, features/feature store, model selection/baselines, training/retraining, serving/latency, evaluation/A-B, monitoring/drift. For each, prepare your *standard answer* — the 2-minute version and the 5-minute deep version. *Cross-ref: `mlops_lifecycle_frameworks_guide.md` (lifecycle), `ml_platforms_comparison_guide.md` (feature stores/serving platforms), `drift_detection_methods_guide.md` (drift).*
- **Week 3 — The five case studies.** One per day, designed from scratch in 45 minutes each: recommendation, search, feed, ads, fraud (Section 9). Write a one-page design per case (metric, data, features, model, serving, eval, monitoring, trade-offs).
- **Week 4 — Mocks and case library.** 2–3 mock interviews per week (see 11.2). Build your **case library**: a table of 10–15 cases with the one-line design for each (problem → metric → model → key trade-off), so any prompt maps to a known pattern. Add LLM-era variants (RAG-based assistants, content moderation) — interviewers increasingly blend classic ML design with LLM components. *Cross-ref: the `ai_llm/` guides (e.g. `llm_evaluation_frameworks_guide.md`, `production_ready_llm_agents_guide.md`) for LLM-era design patterns.*
- **Weeks 5–6 — Full simulation.** Timed 60-minute sessions with a real partner (friend, coach, or AI mock interviewer). Practice the *follow-ups* (scale 10×, cold start, adversarial shift). Record yourself; score against the Section 2 rubric; fix the weakest criterion. Keep going until you can run the framework in your sleep.

### 11.2 The Practice Methods

- **Mock interviews** — the highest-value hour of prep. A partner plays the constraint-giver; you run the framework under time pressure. Debrief against the four criteria (Section 2). Without mocks, you are rehearsing alone for a collaborative exercise — the communication criterion cannot be practiced solo. *(Anecdotal but near-universal: candidates who did 5+ mocks outperform their offline-only preparation by a wide margin.)*
- **Case library** — 10–15 canonical cases with one-page designs; the interview is pattern-matching, and the library is what you match against. Keep a running "trade-offs" column so the tensions are memorized with the designs.
- **The 2-minute/5-minute answer drill** — for every component, prepare a tight 2-minute version (the default) and a 5-minute deep version (for the deep-dive). This prevents both rambling and shallowness.
- **Framework-only drills** — take a random product ("design a spam filter", "design churn prediction") and run only the *structure*: clarify → metric → data → features → model → serving → eval → monitoring, in 15 minutes. This builds the skeleton reflex.
- **Review against the mistakes list** — after every mock, check the Section 10.4 table: which of the ten mistakes did you commit? Track the count across mocks; it should converge to zero.

### 11.3 The Plan Table

| Week | Focus | Activities |
|---|---|---|
| 1 | Framework + vocabulary | Eight steps + time budget; metrics/losses cold; read Huyen or ByteByteGo; two-tower & LTR mechanics |
| 2 | Components | One component/day: data, features, model, training, serving, evaluation, monitoring — with 2-min/5-min standard answers |
| 3 | Case studies | One case/day: recommendation, search, feed, ads, fraud — full 45-min design + one-page write-up |
| 4 | Mocks + case library | 2–3 mocks; build 10–15 case library; add LLM-era variants |
| 5–6 | Full simulation | Timed 60-min mocks; follow-up drills (scale, cold start, adversarial); self-score vs Section 2 rubric |

## 12. The Worked Example: Fraud Detection for a Bank

### 12.1 The Scenario

> **Prompt:** "Design a fraud detection system for a retail bank. Customers use debit and credit cards for online and in-store payments. The bank wants to reduce fraud losses while keeping good customers from being blocked."

This is the classic banking ML-design prompt — deliberately chosen because the banking context (Cymbal Bank, Singapore/MAS environment) is familiar territory. *(Cross-ref: `financial_risk_compliance_systems_guide.md` for the risk/compliance framing, `full_stack_banking_guide.md` for the payment/transaction context.)* The following walkthrough is the *full framework run* — what a strong answer sounds like, minute by minute.

### 12.2 The Walkthrough (the framework steps, full)

**Step 1 — Clarify (3–4 min).** "Before designing, a few questions: (a) Is this for *real-time authorization* (score each transaction before approval — I'll assume yes, since that's where losses happen, with a batch path for post-hoc review), or alerting only? (b) What's the scale? I'll assume ~10M active cards, ~100M transactions/day peak, ~10K TPS — I'll confirm my assumptions. (c) What's the success metric? I'd propose **fraud loss in $ prevented minus investigation cost**, with **false-positive rate** (good customers blocked) as the guardrail — a bank that blocks too many good transactions loses customers, which is worse than the fraud. (d) What data is available? Card, merchant, device, IP, location, transaction history, and past confirmed-fraud labels from chargebacks and investigations. (e) Regulatory constraints? In this environment: explainability/auditability of decisions, data protection, and the obligation to investigate rather than silently block. Let me restate: *a real-time scoring service that assigns a fraud probability to every card transaction, triggers rules/blocking above a threshold set by cost, feeds a review queue, and retrains as fraud shifts — measured on net loss reduction.*"

**Step 2 — Data (5–6 min).** "Labels: 'fraud' = confirmed via chargeback or investigation outcome, which arrives **T+7 to T+30 days** — so today's training set has incomplete labels for the most recent weeks; I'd train on data with a label maturity cutoff. Base rate: ~0.1% of transactions are fraud (say 0.05–0.5%) — a fundamentally imbalanced problem, so accuracy is meaningless; I'd optimize **recall at a fixed FPR/FP budget**. Leakage risks I'd audit: any feature that only exists in the warehouse post-investigation (e.g. a risk-team flag), and temporal leakage in window features (the current transaction must not be inside its own 'past 30 days' window). Validation: **time-based split** (train months 1–11, test month 12) plus a *backtest on the fraud attack of last quarter* to sanity-check robustness. Data sources: transaction events (streamed, the core signal), card/account profile, merchant attributes, device/IP/intelligence feeds (latency and cost per lookup), and the investigation outcomes (labels)."

**Step 3 — Features (5 min).** "Feature groups: (1) **velocity** — tx count, distinct merchants, distinct countries, and total amount over 1h/24h/7d windows (the fraud signature); (2) **card behavior** — typical ticket size, typical merchant categories, time-of-day distribution (deviation features: 'this amount is 5× the card's usual'); (3) **context** — channel (CNP vs in-store), device/IP reputation, card-present vs not, distance from home location; (4) **cross-entity** — has this card ever used this merchant/device?; (5) **model-derived** — a card-risk score from a slower batch model, graph features from the fraud network. Serving note: velocity features must be computed in real time from the event stream and read from the online feature store at request time, with **exactly the same definitions as training** (point-in-time joins) to avoid train/serve skew. I'd put features in the feature store with one definition, monitored per-feature for drift."

**Step 4 — Model (5 min).** "Task: binary classification with a decision threshold set by cost. Baseline: the bank's current **rules engine** (amount thresholds, velocity rules, country blocks) — I'd measure it first; it likely catches 40–60% of fraud at a tolerable FP rate, and the ML model must beat *that*, not beat a random classifier. First ML model: **gradient-boosted trees (LightGBM/XGBoost)** on the engineered features — strong on tabular data, handles the imbalance via class weights, interpretable via SHAP (which the compliance team will ask for). If data volume and infrastructure justify it later, a **deep model** (DCN or a small Transformer over the transaction sequence) can capture per-card sequence patterns; the trade-off is latency, cost, and explainability. Loss: logistic loss with **class weighting** (or focal loss) for the imbalance; I'd avoid SMOTE — synthetic fraud doesn't look like real fraud. Output: a calibrated probability (Platt scaling) — the threshold then comes from the cost model: FP cost (investigation, customer friction) vs FN cost (fraud loss)."

**Step 5 — Training (3–4 min).** "Training pipeline: nightly batch on labeled data with label maturity cutoff; temporal split; schema/quality gates before training; experiment tracking and model registry; evaluation gate (must beat the champion on recall@fixed-FPR and on the last-attack backtest) before promotion. Retraining: **nightly** to start — fraud patterns shift in days — plus **ad-hoc retraining triggered by drift monitoring** (e.g., if the fraud attack shifts, waiting for tonight is too slow). Consider online learning later for the highest-velocity attacks, but only when the batch cadence measurably loses money — complexity discipline."

**Step 6 — Serving (5 min).** "Real-time scoring at the payment gateway: every transaction gets a fraud score **before** authorization, within a hard latency budget (~50ms p99 at ~10K TPS; every millisecond of delay in authorization hurts the checkout experience). Architecture: transaction event → feature computation (stream processor updating velocity features) → online feature store lookup → model inference (GBDT via ONNX Runtime / a compiled runtime on CPU — fast enough at this QPS; GPU not needed) → decision service: **rules + model hybrid** — regulatory and hard-risk rules (sanctions, card-not-present limits) take precedence; the model score sets the operating path: approve / block / route to review queue. Review queue: scored cases below the hard-block threshold but above a lower threshold go to human investigation (the FP budget is really an *investigation capacity* budget). Shadow-deploy new models against the champion, compare, then cut over. Scale: horizontal replicas with autoscaling; the feature store and model cache are the hot path — precompute and cache aggressively."

**Step 7 — Evaluation (3–4 min).** "Offline: **recall at fixed FPR** (e.g. recall@0.1% FPR) and **PR-AUC** as the headline metrics; error analysis by merchant category, channel, and time — the model usually fails on new-merchant and cross-border slices, which tells us where to add data. Compare against the rules baseline and the champion model on both the holdout month and the historical attack periods. Online: **A/B or interleaved** deployment on a traffic slice (e.g. new cards only, or shadow scoring with simulated decisions first), primary metric = **net loss reduction in $** (fraud prevented − investigation cost − friction cost of false blocks), guardrails = FP rate, authorization latency, customer complaint rate. Because fraud is adversarial and rare, I'd also run **synthetic attack replay** — replay a known fraud pattern against the new model offline — since live A/B on fraud is slow (few positives)."

**Step 8 — Monitoring (3 min).** "Monitor three levels: (1) **data drift** — feature distributions (PSI/KS per feature; *cross-ref: `drift_detection_methods_guide.md`*); (2) **concept drift** — the model's score distribution vs realized fraud rate per segment; the model decays when fraudsters shift tactics; (3) **business KPI deltas** — fraud loss, FP rate, investigation queue depth. Alerting on drift + decay triggers retraining (step 5) and a review of features. Also monitor **feedback loops**: if the model blocks more, fraudsters move to other channels — the drift on one channel is often caused by the model's own success on another. In banking, keep an **audit trail** of every decision (features, score, threshold, rule fired) for regulators and for label feedback from investigators — investigation outcomes flow back as labels, closing the loop."

### 12.3 The Interviewer Notes (what was being tested)

| Candidate move | What it demonstrated |
|---|---|
| Asked about real-time vs batch, scale, success metric, regulation *before* any model talk | Requirement clarification; product thinking; banking domain fluency |
| Restated the problem and the metric (net loss reduction) | Structure and communication; business alignment |
| Opened the model step with the *rules baseline* | Baseline discipline; honest attribution of value |
| Named label delay (T+30) and temporal leakage | Real production data awareness — the #1 differentiator |
| Gave the full serving path with a latency budget and a rules+model hybrid | System thinking; regulatory reality (auditability) |
| Picked recall@fixed-FPR and justified it with cost (investigation capacity) | Metric-business alignment; imbalance mastery |
| Closed with drift → retrain → feedback loops | Lifecycle thinking (the loop, not the artifact) |
| Named the trade-offs as it went (GBDT vs deep, nightly vs online learning) | Trade-off articulation at every step |

### 12.4 The Lessons

- **Design the system, not just the model.** The winning answer spent ~60% of the time on data, serving, evaluation, and monitoring — the model discussion was one component among eight, and it opened with a baseline. The candidate who spends 30 minutes on model architectures fails; the candidate who spends 5 minutes on the model and 35 minutes on the system passes.
- **The metric is the contract.** Everything — data decisions, model choice, threshold, serving, monitoring — hangs off the metric (net loss reduction, recall@fixed-FPR). A candidate who cannot state the metric in the first five minutes has not started.
- **Domain fluency is a force multiplier.** The banking context (label delay, investigation capacity, regulators, audit trails, friction cost of false blocks) made every generic ML component concrete. For a banking audience, this is the edge that separates a generic FAANG-style answer from a *credible* one.
- **Trade-offs, said out loud, are free points.** Every choice in the walkthrough names its cost. Interviewers cannot score a trade-off you did not articulate.

### 12.5 A Second Mini-Example: Churn Prediction (transfer check)

To prove the framework transfers, here is the same eight-step run compressed for a different banking-adjacent problem, *customer churn prediction* (e.g. a retail bank's deposit/current-account attrition):

1. **Clarify** — target: customers likely to close or go dormant in the next 90 days; success metric: **retention lift** (churners prevented × LTV) minus offer cost; guardrail: don't annoy retained customers (contact-rate cap); the product is a CRM campaign feed, not real-time scoring.
2. **Data** — labels: account closure/dormancy events (clean, but "dormant" needs an operational definition, e.g. no login and no balance movement for 90 days); base rate maybe 2–5% churn per quarter; leakage: the account-status fields updated *after* the churn decision.
3. **Features** — usage velocity (logins/transactions per week, decaying), balance trajectory (declining balance is the classic leading signal), product-holding diversity (single-product customers churn more), support-ticket sentiment, tenure, channel shift (app → none).
4. **Model** — baseline: "churn rate by tenure band" heuristic; then GBDT with survival-style framing (time-to-event) or plain binary classification on the 90-day window; interpretability via SHAP matters because the CRM team must act on "why".
5. **Training** — temporal split by quarter; retrain monthly (churn drivers drift with the economy and interest rates); class imbalance handled by weighting, not oversampling.
6. **Serving** — batch nightly: score all customers, output a ranked contact list with reasons; integrate with the CRM and the contact-frequency capper; no request-time latency constraint — the cheap path is correct.
7. **Evaluation** — offline: recall@fixed-precision (offer budget = precision constraint) and lift vs random targeting; online: A/B the campaign on matched segments, primary metric = retention lift net of offer cost, guardrails = complaint rate, contact rate.
8. **Monitoring** — churn-rate drift by segment, feature drift (balance trajectory), and the campaign feedback loop (did the offered customers actually stay? — investigation/label feedback, exactly like fraud).

Note what *changed* vs fraud: batch serving instead of real-time, a marketing metric instead of a loss metric, a precision constraint from the offer budget instead of an FPR constraint from investigation capacity — and what *didn't*: the same eight steps, the same baseline-first model logic, the same leakage audit, the same drift-closed loop. That invariance is the framework's entire point.

## 13. The One-Page Summary

**The ML design interview in one page.** It is a 45–60 minute collaborative design conversation: the interviewer gives you a one-sentence product problem, and you design the complete ML system around it — judged on **ML depth, system thinking, communication, and trade-offs** (Section 2), not on the "right answer".

**The framework (memorize this order):** 1 **Clarify** (users, goal, metric, constraints — restate them) → 2 **Data** (sources, labels, leakage, imbalance, split) → 3 **Features** (signals, online vs offline, feature store, skew) → 4 **Model** (task type, *baseline first*, family, loss, why) → 5 **Training** (temporal split, sampling, cadence tied to drift) → 6 **Serving** (batch/online/real-time, latency budget, rules+model hybrid) → 7 **Evaluation** (offline metric matched to the problem + error analysis; A/B with business metric and guardrails) → 8 **Monitoring** (drift → alert → retrain → feedback loop). Time-box: ~5 min each, model ~5–7 min, never more.

**The components cheat-sheet:** data fails first (leakage, labels, imbalance); features must be identical offline and online (feature store, point-in-time joins); models need baselines and named losses; serving must fit a stated latency budget; evaluation must map every model metric to a business metric; monitoring closes the loop.

**The five case studies:** recommendation (two-tower + ANN funnel), search (LTR: pointwise/pairwise/listwise, position bias), feed (freshness + diversity + value model), ads (pCTR/pCVR + calibration), fraud (imbalance + adversarial drift + real-time).

**The three golden rules:** (1) *Never jump to the model* — the framework order is the answer structure. (2) *Never skip serving and monitoring* — juniors stop at evaluation; seniors finish at the feedback loop. (3) *Say the trade-offs out loud* — "X over Y because …, at the cost of …".

**The final word:** the ML design interview is a proxy for the job — designing learning systems that earn their keep in production. Practice the framework until it is reflexive (4–6 weeks, Section 11), build a case library, do real mocks, and walk into the room with the one-sentence mission: **design the system, not just the model.**

## 14. Glossary

- **ML design interview** — the open-ended, ~45–60 minute interview round where a candidate designs a complete ML system for a product problem; scored on process (depth, systems thinking, communication, trade-offs), not a single answer.
- **System design** — the practice of designing the architecture of a software/ML system — components, data flow, constraints, trade-offs — before implementation; the ML design interview is its ML-flavored variant.
- **ML engineer** — the engineer who builds and operates production ML systems: pipelines, feature infra, serving, monitoring; interviewed on coding + ML design depth in infrastructure.
- **Applied scientist** — the scientist who develops models that move product metrics; interviewed on ML fundamentals + ML design depth in modeling and evaluation.
- **Framework** — the ordered set of design steps (clarify → data → features → model → training → serving → evaluation → monitoring) that structures an ML design answer; the published versions are Huyen's (2022) and Xu & Aminian's 7-step (2023).
- **Clarify** — the first framework step: asking scoping questions (users, business goal, metric, constraints, scale, latency) and restating the problem before designing.
- **Data** — the framework step covering sources, labels, volume, quality, imbalance, leakage, and validation splits; the most common source of real ML failure.
- **Features** — the engineered model inputs derived from raw data (velocity, context, cross-entity, model-derived); must be computable identically offline and online.
- **Feature store** — the platform serving both training (offline store with point-in-time correctness) and serving (online low-latency store) from one registered feature definition; e.g. Feast, Tecton.
- **Model selection** — choosing the model family by task type, data shape, and constraints; always anchored to a baseline.
- **Baseline** — the bar a new model must beat: a heuristic/rules system or a simple model (logistic regression); without one, model gains cannot be attributed.
- **Training** — the pipeline step: splits, sampling, loss, training cadence, retraining strategy, experiment tracking.
- **Serving** — running the model in production: batch (scheduled), online (request-time), or real-time (streaming); constrained by latency budget and scale.
- **Online** — (in serving) request-time inference; (in evaluation) evaluation on live traffic (A/B testing). Context decides which meaning applies.
- **Offline** — (in serving) batch/scheduled inference; (in evaluation) evaluation on historical data. Context decides which meaning applies.
- **Real-time** — streaming/event-time inference, e.g. scoring every transaction as it happens; freshest but most complex.
- **Evaluation** — measuring whether the model works: offline on historical data, online via A/B, and continuously via monitoring.
- **Offline metrics** — metrics computed on historical data: precision, recall, PR-AUC, ROC-AUC, NDCG, MAP, MRR, MAE/RMSE, calibration error.
- **Online metrics** — metrics from live experiments: business metrics (conversion, revenue, fraud $), guardrails (latency, error rate), and experiment-validity checks (SRM).
- **A/B test** — the live experiment comparing a treatment (new model) against control (champion) on the business metric with guardrails; ML-specific pitfalls: interference, novelty, multiple comparisons.
- **Recommendation** — the case study of predicting what a user will engage with, typically via a two-stage funnel: retrieval then ranking.
- **Two-tower** — the dominant retrieval architecture: a user tower and an item tower producing embeddings whose dot product scores relevance; trained with in-batch softmax; served via ANN search.
- **Search ranking** — the case study of ordering documents for a query; a learning-to-rank problem.
- **Learning-to-rank (LTR)** — training a model to order items: pointwise, pairwise (RankNet/LambdaMART), or listwise losses; evaluated with NDCG/MRR.
- **Feed ranking** — the case study of ordering a personalized feed; adds freshness, diversity, and engagement-value objectives to ranking.
- **Ad prediction** — the case study of predicting click/conversion probability (pCTR/pCVR) for auction bidding; dominated by calibration and selection-bias concerns.
- **Fraud detection** — the case study of flagging fraudulent transactions/accounts under extreme class imbalance, adversarial drift, label delay, and hard real-time latency.
- **Class imbalance** — the condition where the positive class is extremely rare (fraud ~0.1%); makes accuracy meaningless and drives metric choice (recall@fixed-FPR, PR-AUC) and loss choice (class weights, focal loss).
- **Trade-off** — the explicit cost of a design choice; the interview rewards naming "X over Y because …, at the cost of …" for every decision.
- **Latency** — the time budget for a prediction (p50/p99); the constraint that pits model quality against serving feasibility.
- **Complexity** — engineering/ops cost of a component; the discipline is adding complexity only when the benefit is measured (right-sizing).
- **Monitoring** — the ongoing observation of data drift, concept drift, and business KPIs; the trigger for retraining and the closure of the ML lifecycle loop.
- **Drift** — the decay of model validity as the data distribution changes: data drift (features), concept drift (the relationship between features and labels).
- **Practice plan** — the 4–6 week structured prep: framework (wks 1–2), case studies (wk 3), mocks + case library (wk 4), full simulation (wks 5–6).
- **Mock interview** — a timed practice session with a partner playing the constraint-giver; the only reliable way to train the communication and collaboration criteria.
- **Case library** — the candidate's personal bank of 10–15 canonical designs (problem → metric → model → trade-off) used for pattern-matching any prompt.

## 15. Conclusion

The Machine Learning System Design interview is the FAANG/ML-industry's test of whether you can build *learning systems* rather than models — and for a solution architect in banking, it is also the vocabulary of the ML platform conversations you already have every day: data pipelines, feature stores, model serving, drift, retraining, evaluation. The framework in this guide (clarify → data → features → model → training → serving → evaluation → monitoring) is the same loop your production platforms run, and the case studies (recommendation, search, feed, ads, fraud) are the same problems the `ml_platforms_comparison_guide.md`, `mlops_lifecycle_frameworks_guide.md`, and `banking/` guides describe from the platform side. Prepare with the 4–6 week plan, practice with real mocks, drill the worked example until it is fluent, and remember the one sentence that wins the round: **design the system, not just the model.**

## 16. Appendix: The Prompt Bank and Resources

### 16.1 The Practice Prompt Bank

Drill these in order of difficulty (verified as the standard canon across the ByteByteGo book, Exponent, and community case libraries):

- **Easy:** movie recommendation; churn prediction; spam detection; demand forecasting.
- **Medium:** search ranking for e-commerce; news-feed ranking; CTR prediction; fraud detection; content moderation.
- **Hard:** video recommendation at YouTube scale; real-time bidding ads; ride-hailing ETA/price prediction; LLM support assistant with RAG; credit scoring with fairness constraints; multi-task feed (recommend + ads + organic mix).

### 16.2 Recommended Resources

- **Books:** Alex Xu & Ali Aminian, *Machine Learning System Design Interview* (ByteByteGo, 2023) — the interview playbook; Chip Huyen, *Designing Machine Learning Systems* (O'Reilly, 2022) — the production depth reference; Chip Huyen, *Introduction to Machine Learning Interviews* (2022) — question bank and scoping.
- **Free material:** Chip Huyen's CS 329S course materials (Stanford); Valerii Babushkin & Arseny Kravchenko, *Machine Learning System Design* (free ebook) — a strong community framework; the community `ml-system-design` GitHub repos with hundreds of case-study outlines. *(Verify current links at prep time — community repos move.)*
- **Mock platforms:** Exponent, DataInterview, Hello Interview, and AI mock-interview tools — use them for reps, not as the source of truth; nothing replaces a human constraint-giver.
- **This series' cross-refs:** `ml_platforms_comparison_guide.md` (platforms/feature stores), `mlops_lifecycle_frameworks_guide.md` (lifecycle), `mathematics_for_ml_and_ds_study_notes.md` (metric math), `llm_evaluation_frameworks_guide.md` (LLM eval), `drift_detection_methods_guide.md` (monitoring), `personalization_engines_guide.md` (recommendation systems), and the `banking/` guides (domain context).


