# Applied Machine Learning — The Modelling Craft: A Comprehensive Guide

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Cymbal Bank  
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Companion Guides:** [Full MLOps Lifecycle Frameworks](mlops_lifecycle_frameworks_guide.md) · [Feature Stores](feature_store_guide.md) · [ML Platforms Comparison](ml_platforms_comparison_guide.md)  
> **Last Updated:** September 2026

---

## Table of Contents

1. [The Overview and the Three-Way Distinction](#1-the-overview-and-the-three-way-distinction)
2. [Problem Framing: From Business Question to Learning Task](#2-problem-framing-from-business-question-to-learning-task)
3. [Label Definition and Target Construction](#3-label-definition-and-target-construction)
4. [The Data Work and the Dataset You Actually Have](#4-the-data-work-and-the-dataset-you-actually-have)
5. [Feature Engineering and Feature Selection](#5-feature-engineering-and-feature-selection)
6. [Leakage: The Modelling Killer](#6-leakage-the-modelling-killer)
7. [Model Selection and the Baseline Discipline](#7-model-selection-and-the-baseline-discipline)
8. [Honest Evaluation: Splits, Metrics and Uncertainty](#8-honest-evaluation-splits-metrics-and-uncertainty)
9. [Probability Calibration and the Confidence Question](#9-probability-calibration-and-the-confidence-question)
10. [Class Imbalance and the Contested Advice](#10-class-imbalance-and-the-contested-advice)
11. [The Error-Analysis Iteration Loop](#11-the-error-analysis-iteration-loop)
12. [Applied Versus Research Trade-offs](#12-applied-versus-research-trade-offs)
13. [The Production Handoff](#13-the-production-handoff)
14. [Modelling Under Regulation: The Banking Constraints](#14-modelling-under-regulation-the-banking-constraints)
15. [The Cymbal Bank Worked Example: One Project, End to End](#15-the-cymbal-bank-worked-example-one-project-end-to-end)
16. [Explainability, Documentation and Model Review](#16-explainability-documentation-and-model-review)
17. [Anti-Patterns and Failure Modes of the Modelling Craft](#17-anti-patterns-and-failure-modes-of-the-modelling-craft)
18. [The Claims Audit, and What Could Not Be Verified](#18-the-claims-audit-and-what-could-not-be-verified)
19. [Glossary](#glossary)
20. [References and Further Reading](#references-and-further-reading)

---

## 1. The Overview and the Three-Way Distinction

**Applied machine learning** is the practitioner's discipline of getting a model to actually work on a real problem. It is not the study of learning algorithms, and it is not the operation of a training pipeline. It is the sequence of judgement calls that decide what to predict, from what data, at what moment, evaluated how, and declared good enough by whose standard.

This guide is deliberately narrow. Three sibling guides already own the adjacent territory, and re-deriving their content here would produce a longer, worse document:

| Territory | Owner | What it covers |
|---|---|---|
| **The operational pipeline** | [MLOps Lifecycle Frameworks](mlops_lifecycle_frameworks_guide.md) §4, §12 | Lifecycle stages operationally, CI/CD/CT, registries, deployment mechanics (shadow, canary, blue-green, A/B), batch vs. real-time vs. streaming serving, monitoring and drift metrics (PSI, K-S), governance risk tiering, retraining triggers, and the operational failure modes |
| **The feature platform** | [Feature Stores](feature_store_guide.md) | The feature-store product, point-in-time correctness as an architecture concern, online/offline serving patterns, feature governance |
| **The mathematics** | [Mathematics for ML and DS Study Notes](mathematics_for_ml_and_ds_study_notes.md) | Linear algebra, probability, optimisation, the derivations behind the estimators |
| **The tooling market** | [ML Platforms Comparison](ml_platforms_comparison_guide.md) | MLflow vs. Kubeflow vs. SageMaker vs. Vertex vs. Azure ML — "which tool" questions |
| **The adversary** | [Adversarial ML Attacks](adversarial_ml_attacks_guide.md) | Evasion, poisoning, extraction, and defences |

**THIS guide owns the modelling decisions.** Specifically:

- **Problem framing** — turning a business complaint into a well-posed learning task, or refusing to.
- **Label definition** — deciding what the target variable *is*, and what constitutes an observation.
- **Feature construction and selection** — as modelling judgements, not as platform capability.
- **Leakage detection** — the discipline that separates a credible result from a story.
- **Model selection** — how to choose, including choosing the baseline.
- **Honest evaluation** — splits, metrics, uncertainty, and the difference between a number and a decision.
- **The error-analysis loop** — the iteration that actually moves the metric.
- **The good-enough decision** — when to stop, and who signs.

It does **not** own the training pipeline, the model registry, the deployment mechanism, the monitoring stack, or the retraining cadence. Where those appear below, they appear in one line with a pointer. A deployment pattern named here is a constraint on modelling (what latency the model may have, whether it must be explainable at scoring time); the mechanics live in [MLOps Lifecycle Frameworks](mlops_lifecycle_frameworks_guide.md) §4.6. A feature-store concept named here is a constraint on how features are computed and served; the product lives in [Feature Store](feature_store_guide.md) §6 and §7.

> **Credo.** A model that works in production is *made*, not found. The reported metric is a claim, and like any claim it needs a mechanism explaining why it is not an artefact of the evaluation. Modelling is the practice of constructing evidence strong enough to survive a hostile reviewer — and of knowing when the honest answer is that no model will help.

### 1.1 The Four Questions Before Any Code

Every modelling engagement answers four questions in order. Skipping one is the most reliable predictor of a project that ships a model nobody uses.

| # | Question | If unanswered |
|---|---|---|
| 1 | **What decision changes?** | The model has no consumer; it will be trained, evaluated, shelved |
| 2 | **What is the label, and when does it become known?** | The evaluation is unmeasurable or leaks |
| 3 | **What is the cost of being wrong, in both directions?** | The metric cannot be chosen, so the "best" model is arbitrary |
| 4 | **What is the baseline, and who is accountable for beating it?** | Any positive number looks like success |

### 1.3 A Fictional Running Example

Throughout this guide the examples use **Cymbal Bank** — a fictional institution used consistently across this repository's ML guides. Cymbal Bank is an *illustrative fiction*: any resemblance to a real institution's systems is stylistic. The examples exist to make modelling constraints concrete, particularly the banking ones in §14, where the regulator's framing routinely dictates modelling choices that a purely technical reading would not.

Cymbal Bank's modelling portfolio in these examples includes: a retail credit-card application scorecard, a term-deposit propensity model, a transaction-fraud detector, and a customer-attrition (churn) model. Each exercises a different part of the craft — the scorecard the regulated path, the propensity model the label-timing problem, the fraud detector the extreme-imbalance and latency problem, the churn model the drift and actionability problem.

### 1.4 The Tooling Map — Pointers Only

Tool selection is a platform decision and is not compared here — that is [ML Platforms Comparison](ml_platforms_comparison_guide.md)'s subject. What the modeller needs is to know which layer owns which artefact, so that a modelling requirement is expressed as a *requirement on a layer* rather than as a personal preference for a vendor.

| Layer | What it owns | Where the detail lives |
|---|---|---|
| **Experiment tracking** | Runs, parameters, metrics, artefacts, comparison across attempts | MLOps §6.2; [ML Platforms Comparison](ml_platforms_comparison_guide.md) |
| **Feature store** | Feature definitions, offline/online consistency, point-in-time correctness | [Feature Stores](feature_store_guide.md) §6, §7, §11 |
| **Pipeline / orchestration** | Scheduling, dependencies, the training and evaluation runs | MLOps §5, §7 |
| **Model registry** | Versions, lineage, stage transitions, deployment status | MLOps §4.9, §6.2 |
| **Serving** | Endpoints, batch scoring, the latency and throughput budget | MLOps §4.6 |
| **Monitoring** | Drift metrics, performance, alert routing | MLOps §4.7 |
| **Model-risk evidence** | Validation, documentation, audit trail, model inventory | MLOps §4.5, §4.8; this guide §14 |

**The rule:** the modeller states requirements *on* these layers; the platform team implements them. A requirement that no layer can express — "the top twenty features must be auditable for availability at the prediction moment" — is modelling work that no tool performs on your behalf. The minimum tracking discipline the modeller owes is small and non-negotiable: every reported result traceable to a code version, a data snapshot, a split definition, and a metric definition (§8).

---

## 2. Problem Framing: From Business Question to Learning Task

Framing is where the guide's leverage is highest and where it is least technical. A well-framed problem is usually easy to model adequately; a badly framed problem is unfixable by any algorithm.

### 2.1 The Framing Ladder

Move down this ladder until you can go no further. Every step down is a test of whether the modelling is warranted at all.

| Rung | Form | Example (Cymbal Bank) | Modelling required? |
|---|---|---|---|
| **Complaint** | A stated pain | "We lose too many term deposits at maturity" | No — not yet a task |
| **Decision** | An action someone will take | "Decide which depositors to call, and how many, one week before maturity" | No |
| **Target quantity** | What must be estimated to make the decision | "Probability this customer rolls over at maturity" | Maybe |
| **Learning task** | The formal problem | "Binary classification, one row per (customer, maturity date)" | Yes |
| **Operational contract** | Constraints the solution must respect | "Score within 200 ms, features available 7 days before maturity, reasons for the top adverse drivers" | Yes, and these constrain the model class |

**Rule of thumb:** if you cannot name the *decision* and the *actor*, you are not framing a model — you are framing a report. Many complaints are better served by a query, a rule, or a process change. Recommending that is a success, not a failure.

### 2.2 Descriptive, Predictive, Causal

The single most common framing error is answering a causal question with a predictive model. Shmueli's *To Explain or To Predict?* makes the statistical distinction explicit: explanation seeks the true data-generating process and the causal role of each variable; prediction seeks the estimate with the lowest expected error on new cases, and the two objectives can select different models with different variables on the same data. A customer-attrition model optimised for prediction will happily use a variable that is a *consequence* of attrition if it predicts well — which is useless, or worse than useless, for a retention campaign that intends to *change* the outcome.

| Question type | Asks | Correct instrument | Wrong instrument's failure |
|---|---|---|---|
| **Descriptive** | "What happened?" | Aggregation, dashboards | A model gives you a black box where a number was needed |
| **Predictive** | "What will happen to this case?" | Supervised learning | — |
| **Causal** | "What happens if I act?" | Experiment, quasi-experimental design, uplift modelling | A propensity score of 0.9 does not mean the customer responds to being called |

**Note on uplift.** Where the decision is "whom do we act on", the useful quantity is often the *incremental* effect of the action, not the outcome probability. Uplift (incremental-response) modelling is a distinct modelling problem with its own evaluation (the Qini-style uplift curves) and its own data requirement: an outcome that varies independent of the action, typically a holdout. Treating a propensity model as an uplift model systematically over-invests in customers who would have converted anyway.

### 2.3 Framing Questions That Decide the Rest of the Project

| Question | Why it constrains everything downstream |
|---|---|
| **What is the unit of observation?** | Customer, account, application, transaction, or (customer, month) panel — each implies a different population, a different label construction, and a different split |
| **What is the prediction moment?** | Fixes which features are legitimately available; the single most important leakage defence (§6) |
| **How many decisions per period?** | Determines whether a small precision gain has material value, and whether capacity is the binding constraint |
| **Is the outcome time-stamped?** | Without it there is no honest temporal split and no label window |
| **Is there an existing rule or scorecard?** | That is the baseline (§7); if the new model cannot beat it, the project has no case |
| **How often does the world change?** | Sets the appetite for a model that stays valid, hence the validation regime and the retraining expectation handed to the MLOps pipeline |

### 2.4 The Feasibility Verdict

Before committing, answer three feasibility tests honestly:

1. **Is the signal present?** A weak but real relationship beats none. If the best available features are known to be unrelated to the outcome, no model class rescues the project.
2. **Is the label learnable?** Labels that are themselves the product of a biased process (see §3.3) teach the model the process, not the world.
3. **Will anything change?** A model whose output no one can act on is an expensive report.

**Deliverable of this section:** a one-page framing memo — decision, actor, unit, prediction moment, label sketch, cost asymmetry, baseline, and the feasibility verdict. It is the document a reviewer should reject *before* compute is spent, not after.

---

## 3. Label Definition and Target Construction

The label is the most consequential and least examined artefact of a modelling project. Algorithms are commoditised; the label is bespoke, and it is where the project is usually wrong.

### 3.1 The Label Specification

A defensible label has six components. Write them down; most leakage and most evaluation disputes dissolve when they are explicit.

| Component | Question | Cymbal Bank example (card default) |
|---|---|---|
| **Definition** | What event counts as positive? | "The account reaches 90 days past due" |
| **Observation anchor** | From when is the clock started? | Application date |
| **Outcome window** | For how long is the event looked for? | 12 months from account opening |
| **Censoring rule** | What about cases where the outcome is not yet observable? | Excluded until the window closes; never treated as negative |
| **Performance window vs. observation window** | Are features and label drawn from disjoint periods? | Features from application time and earlier; label from the following 12 months |
| **Maturity requirement** | How much history must an account have to enter the sample? | At least one month of account activity |

**Censoring is the most frequently botched component.** Treating a not-yet-observed case as a negative systematically biases the label rate downward for recent cohorts and poisons any temporal evaluation. The clean discipline is to train on matured cohorts only, and to accept that the most recent data is the least usable — an uncomfortable trade that honest evaluation requires.

### 3.2 Label Timing and the Prediction Moment

The label window and the feature window must not overlap in a way that lets the future leak into the inputs. The structure is always:

```
[ feature window: data available at the prediction moment ]
|-- prediction moment --|
[ label window: the outcome is observed after this point ]
```

Kaufman, Rosset, Perlich & Stitelman formalise this in *Leakage in Data Mining: Formulation, Detection, and Avoidance* (KDD 2011; extended in ACM TKDD 6(4), 2012) as a **learn–predict separation**: model construction may use only information legitimately available at the prediction moment. Their taxonomy distinguishes leakage by *source* — including **primary** leakage (a feature that is a proxy for the target within the data), **secondary** leakage (a proxy created by the sampling or selection process), and **external leakage** (information from outside the dataset, as in a data-mining competition). Their historical examples are instructive precisely because they are mundane: an account-number field predicting "will the customer open an account" (accounts are numbered only after opening), and a "session length" field predicting "will the user leave this page" (session length is only known at session end). Both pass a code review and destroy the model's production value.

### 3.3 Label Quality: The Process Is the Data

Labels are usually records of a *process*, not measurements of a *world*. When the process is selective, the model learns the selectivity.

| Label pathology | Mechanism | Symptom | Mitigation |
|---|---|---|---|
| **Selective labels** | Only cases that passed a prior screen have outcomes recorded | Model predicts the prior screen | Reconstruct the full population; model the process explicitly |
| **Proxy labels** | The recorded proxy is a poor stand-in for the concept | Hard ceiling on achievable quality | Validate the proxy against a hand-labelled sample |
| **Delayed labels** | Outcome observed long after the decision | Recent data unusable, drift invisible | Censoring rule; monitor the label pipeline's latency |
| **Rejected-inference bias** | Rejected applicants have no performance data | Scorecard cannot be validated on rejects | Reject inference / augmentation techniques; treat as an explicit assumption |
| **Label noise** | Human or system annotation error | Irreducible error floor; miscalibrated probabilities | Estimate the noise rate on an audited sample |

**Reject inference** deserves emphasis for the banking case. A credit model is trained on applicants who were *approved* under the incumbent policy, so the label exists only for the approved — a textbook selective-labels problem. The model is then asked to rank the rejected. This extrapolation is a modelling assumption (that the relationship seen in the approved population extends to the rejected), and it should be stated as an assumption in the model documentation, not silently assumed. This is the modelling counterpart to the operational data-quality work in [MLOps Lifecycle Frameworks](mlops_lifecycle_frameworks_guide.md) §4.2.

### 3.4 Labels You Cannot get Cheaply

Where a label is expensive (a manual review, a lab test, an investigator's time), the modelling problem includes the *labelling budget*. Active-learning style prioritisation — choosing which cases to label for maximum information — is a legitimate modelling decision, but it changes the sample's distribution and therefore the evaluation. Any sample selected non-uniformly for labelling must have its selection accounted for at evaluation time, or the reported metric is a number about the labelling policy rather than about the deployment population.

---

## 4. The Data Work and the Dataset You Actually Have

The dataset is a *decision*, not a discovery. It is the result of inclusion rules, time windows, joins, and exclusions that someone chose. Modelling starts by reading those choices back out.

### 4.1 The Four Data Risks That Kill Projects

| Risk | What it looks like | Why it kills the model |
|---|---|---|
| **Coverage** | The population that will be scored is not the population in the training data | The model is evaluated on a population it will never see |
| **Temporal validity** | Training period differs systematically from the scoring period | The relationship learned is a historical artefact |
| **Selection** | Sample inclusion depends on the outcome or on the prior policy | Label bias (§3.3) |
| **Integrity** | Duplicates, joins that fan out, units/timezone drift | Inflated or deflated metrics with no modelling cause |

**Join fan-out** is the quiet one. A one-to-many join that was intended to be one-to-one multiplies rows, silently reweights the training distribution, and can inflate a metric by making the same cases appear repeatedly across a split boundary. Row-count assertions at every join are not paranoia; they are the modelling equivalent of a unit test.

### 4.2 Exploratory Data Analysis as a Modelling Instrument

EDA is not a preliminary ritual. It is the instrument that detects leakage (§6), decides the feature transformations (§5), and estimates the achievable ceiling. The specific diagnostics that pay for themselves:

| Diagnostic | Reveals |
|---|---|
| Target rate over time | Regime change, label drift, broken pipeline |
| Target rate by cohort | Population shift; whether the incumbent policy changed |
| Univariate distribution of every feature | Constant features, sentinel values (`-1`, `9999`, `1900-01-01`) masquerading as data |
| Missingness pattern, and missingness *vs. target* | Whether "missing" is informative — often it is, and then it is a feature |
| Feature–target relationship by time slice | A relationship that exists in one slice only is usually leakage or a small-sample artefact |
| Cardinality and identifier-shaped columns | Free-text, IDs, and hashes that should never enter the model |
| **Suspiciously strong single features** | The single best leakage detector there is: if one feature almost determines the label, it is probably a descendant of the label |

### 4.3 The Data You Cannot Get

Three honest conversations to have before modelling:

- **The feature that would matter is unavailable at the prediction moment.** No amount of modelling fixes availability; the answer is a process change or a different framing.
- **The history is shorter than the outcome window.** Then the label is immature, and you must either wait, shorten the window, or accept weaker evidence and say so.
- **The data exists but is not joinable.** Identity resolution errors become modelling errors; quantify the match rate and treat it as a reported metric of the dataset.

### 4.4 Data Quality as a Modelling Contract

The data-quality checks that protect the *model* (as distinct from the pipeline) are: schema conformance, range and unit checks, referential integrity, distributional monitoring against a reference window, and row-count assertions per join. The operational implementation — data contracts, validation gates, lineage — belongs to [MLOps Lifecycle Frameworks](mlops_lifecycle_frameworks_guide.md) §4.2; the point here is that the modeller is the *consumer* of that contract and must specify what it needs, not assume it.

> **Practitioner note.** The often-repeated claim that "most of the gains come from data work rather than modelling" is a widely shared practitioner heuristic, not a measured law. It is stated in this repository's experience as a heuristic. The strongest *documented* support for treating data work as first-class is qualitative: Sambasivan et al., *"Everyone wants to do the model work, not the data work": Data Cascades in High-Stakes AI* (CHI 2021), studies practitioners across multiple high-stakes deployments and finds recurring failures originating in data issues, compounding downstream and eroding trust in AI systems. That is a qualitative research finding about failure patterns, not a quantified attribution of accuracy gains. Treat the "80% of the value" style figure as folklore unless a specific study is cited for it. See §18.

---

## 5. Feature Engineering and Feature Selection

Feature work is where domain knowledge enters the model, and where most applied gains are actually found. It splits into two responsibilities this guide keeps separate: **what the feature means** (modelling) and **how it is computed and served consistently** (platform).

### 5.1 The Division of Labour

| Responsibility | Owner |
|---|---|
| What the feature should represent; its business interpretation | **This guide** |
| Time-window logic, aggregation grain, entity keys as modelling intent | **This guide** |
| Transformations: monotone, binning, splines, target/mean encoding, interactions | **This guide** |
| Feature selection: relevance, redundancy, stability, causality plausibility | **This guide** |
| Point-in-time correctness *as an architectural guarantee* | [Feature Store](feature_store_guide.md) §6 |
| Online/offline consistency, serving patterns, feature registry, lineage | [Feature Store](feature_store_guide.md) §7, §11 |
| Feature engineering *for the store* (the store's own view) | [Feature Store](feature_store_guide.md) §5 |

**Rule:** if the answer to a feature question depends on which platform you are using, it belongs to the feature-store guide. If it depends on the business, it belongs here.

### 5.2 Feature Families and Their Typical Failure Modes

| Family | Example | Characteristic failure |
|---|---|---|
| **Raw / demographic** | Age band, tenure | Categorical blow-up; fairness exposure (§14) |
| **Aggregate over a window** | 30-day transaction count, mean balance | Window mismatch between training and serving |
| **Recency / frequency / monetary** | Days since last transaction | Computed "as of now" instead of "as of the prediction moment" — the classic leak |
| **Ratio / velocity** | Balance ÷ income; transactions per active day | Denominator instability; divide-by-zero sentinels |
| **Trend / difference** | 90-day vs. 30-day mean ratio | Double-counts the same period; overlapping windows |
| **Derived judgement** | "Is the customer overdrawn as of the scoring date" | Ambiguous definition; two implementations diverge |
| **Network / relational** | Number of distinct counterparties | Identity resolution error; leakage from the counterparty's own outcome |
| **External** | Bureau score, macroeconomic index | Availability mismatch; vintage (the external data's as-of date ≠ yours) |
| **Text / embedding** | Free-text notes (no longer exclusively an LLM concern) | Train/serve tokenisation skew; embedding model version drift |

### 5.3 Selection: Relevance, Redundancy, Stability

Feature selection in applied work is a three-filter problem, applied in order:

1. **Relevance** — does it carry information about the label? Measured with mutual information, or with a model-based importance under proper cross-validation.
2. **Redundancy** — is it a duplicate of another feature? Correlation clusters (or variance-inflation factor for linear models) identify the redundant group; keeping one representative is usually better than keeping all.
3. **Stability** — does its importance survive across time slices and resamples? An unstable importance is either noise, or a signal that the world changed, or leakage.

**On importance measures.** Model-internal importance (split-gain in tree ensembles) is biased toward high-cardinality and continuous features and says nothing about direction. Permutation importance is more comparable across feature types but is misleading under correlated features — permuting one of a correlated pair understates both. Where a *consistent, additive* attribution is required (and especially where reasons must be produced for a decision, see §14), Shapley-value based attribution has a principled basis: Lundberg & Lee's SHAP framework (NeurIPS 2017) identifies a class of additive feature-attribution methods and shows a unique solution within it satisfying local accuracy, missingness, and consistency. SHAP values are still attributions *of the model*, not causal statements about the world — the distinction matters legally and commercially.

### 5.4 Encoding and Transformation Judgements

| Technique | Use when | Risk |
|---|---|---|
| One-hot | Low-cardinality nominal | Dimensionality with high cardinality |
| Ordinal / monotone (e.g. WOE binning) | The relationship is monotone and the model must be linear, as in scorecards | Loses non-monotone effects |
| Target / mean encoding | High cardinality, and the encoding can be computed with out-of-fold discipline | **Leakage if computed on the full training set**; unstable for rare categories |
| Binning | Non-linearity in a linear model; business-communicable buckets | Arbitrary boundaries; information loss |
| Splines / GAM terms | Smooth non-linearity, interpretable shape | Less familiar to reviewers; extrapolation behaviour must be checked |
| Interactions | Known domain interaction (e.g. utilisation × limit) | Combinatorial explosion; overfitting |
| Missing-value indicator | Missingness is informative | Multiplies features; correlated indicators |

**Target encoding done wrong is the most common self-inflicted leak in applied work.** If the mean target per category is computed on the same rows used to train, the encoding carries the label directly. It must be computed out-of-fold, and its handling at inference (using the training-fold statistics) must be identical.

### 5.5 The Feature Ceiling

A useful discipline: before tuning anything, estimate the achievable ceiling. Take the best single feature, and the best simple combination, and compute the metric. If the gap between that and the target metric is small, modelling effort will not close it — feature work or reframing will. This estimate costs an hour and routinely saves weeks.

---

## 6. Leakage: The Modelling Killer

Leakage is the one error that makes every subsequent number meaningless, and it is invisible in the metric — it *improves* the metric. That is precisely why it must be hunted deliberately.

### 6.1 What Leakage Is, and Who Named It

The canonical practitioner treatment is **Kaufman, Rosset, Perlich & Stitelman, "Leakage in Data Mining: Formulation, Detection, and Avoidance"**, published at **KDD 2011** (DOI 10.1145/2020408.2020496) and extended in **ACM Transactions on Knowledge Discovery from Data (TKDD) 6(4), 2012** (DOI 10.1145/2382577.2382579, a special issue on the best papers of KDD 2011). Verify which version you are citing: the KDD paper is four authors in the proceedings version and the TKDD version credits Stitelman as the fourth author; both are real, and citing "TKDD 2011" is a common error — the *conference* is 2011, the *journal* is 2012.

Their definition: leakage is the introduction of information about the target that should not be legitimately available to mine from. Their formulation requires an explicit statement of the modelling *goal* (what will be known at prediction time), and their principal remedy is **learn–predict separation** — enforcing, in data management, that no information unavailable at prediction time can influence learning. They also propose legitimacy tagging of observations at collection time, and name three detection approaches for the modeller who inherits data they did not collect: **exploratory data analysis** (especially the "too good to be true" performance signal), **ex-post analysis of modelling results** (inspecting what the model actually used), and **early field testing**. Their taxonomy is by source:

| Leakage type (per Kaufman et al.) | Mechanism | Classic example |
|---|---|---|
| **Primary** | A feature leaks the target within the dataset | An account number predicting account opening |
| **Secondary** | Leakage created by the sampling/selection process, not the feature | A pooled dataset where the sampling label is itself predictive |
| **External** | Information from outside the dataset (including public sources) | A competition where the winning entry de-anonymised the target data |

Earlier descriptions of the same phenomenon exist and are worth attributing correctly: Pyle described such features as **"anachronisms"** — things out of place in time, with "too good to be true" performance as their giveaway; Nisbet, Elder & Miner called them **"leaks from the future"** and listed them among the top ten data-mining mistakes; Kohavi et al. described **giveaway attributes** that are downstream in the data-collection process.

### 6.2 The Leakage Taxonomy in Practice

Organised by *where in the workflow the leak enters*, which is how you actually find them:

| Where | Leak | Detection |
|---|---|---|
| **Label construction** | A feature is a *consequence* of the label | Post-hoc: which features dominate? Are they plausible at the prediction moment? |
| **Sampling** | The sample's inclusion criterion is informative | A model predicting "in-sample" from features would succeed |
| **Joining** | A join pulls post-outcome records | Timestamp audit of every joined table |
| **Aggregation** | A window includes the outcome period | Window arithmetic written on a diagram, not in a dataframe |
| **Preprocessing** | Scaling, imputation, or encoding fit on all data | Fit on training folds only — always |
| **Feature selection** | Selection done on the full dataset before splitting | Wrap selection inside the resampling loop |
| **Tuning** | Hyperparameters chosen on the test set | A nested loop, or a strictly held-out set touched once |
| **Duplicates** | Near-duplicate records across train and test | Hash/neighbour check across the split boundary |
| **Time** | Random split on temporal data | Splits by time (§8) |

**Preprocessing leakage is the most common one in practice**, because every library makes it easy: fit the scaler and the imputer on the whole dataset, then cross-validate. The metric is optimistic and nobody sees it. The correct pattern wraps every data-dependent transformation inside the resampling loop.

### 6.3 Repeated Identities, the Subtlest Case

Two rows for the same customer in different folds is leakage. The model recognises the customer, not the signal. It is especially vicious in medical and financial data where multiple rows per entity are normal. Detect it by asking "what is the entity, and can one entity appear in two folds?" — and then grouping (§8.2). Kaufman et al.'s broadest formulation covering this and other non-i.i.d. cases is what makes the paper still the standard reference.

### 6.4 The Detection Protocol

Run this before believing any result:

1. **Plausibility audit** — for each of the top-20 features by importance, write down *when* it is known and *whether it would be known at the prediction moment*. Do this in writing; the leak is easier to see in a sentence than in a matrix.
2. **The "too good to be true" test** — any metric far above the domain's known achievable range is a leak until proven otherwise, not a breakthrough.
3. **The adversarial shuffle** — deliberately leak: add a feature that is a deterministic function of the label with noise. If the model's performance jumps, the machinery cannot distinguish leak from signal, and the split is suspect. (This is a diagnostic on the *pipeline*; obviously do not ship it.)
4. **Feature ablation** — remove the top-N features and re-evaluate. Performance that collapses to plausible levels identifies them as the leak.
5. **Temporal replay** — retrain on data up to time *t* and evaluate on *t+1*. If the temporal evaluation is much worse than the random split, the random split was leaking (§8.2).
6. **Lineage review** — for each feature, name the source table, the join, and the timestamp. Anything you cannot explain in one sentence is a candidate.

**Expected-value note:** an ablation that removes a *legitimate* strong feature is also informative — it tells you how much of your performance rests on one variable, which is a robustness and monitoring question (hand it to [MLOps Lifecycle Frameworks](mlops_lifecycle_frameworks_guide.md) §4.7).

---

## 7. Model Selection and the Baseline Discipline

Model selection in applied work is not "find the best algorithm". It is "find the simplest model that clears the bar, then justify anything more complex".

### 7.1 The Baseline Is the Minimum Bar

Every model has a baseline it must beat, and the baseline is the *incumbent*, not zero:

| Baseline tier | What it is | Why it must be present |
|---|---|---|
| **Trivial** | Majority class; base rate | Establishes the metric floor |
| **Rule** | The business rule currently in force | Establishes *operational* value |
| **Simple model** | Logistic/linear regression on a small engineered set | Establishes *modelling* value |
| **Tuned simple model** | The above, with hyperparameters and features tuned | Establishes that complexity is needed for *gain*, not for competence |

Comparisons must be on identical splits, identical metrics, and (crucially) with confidence intervals (§8.4). A headline accuracy of 94% against a 93% base rate is a rounding error, not a model. The MLOps sibling treats "baseline as minimum bar" as a lifecycle gate in [MLOps Lifecycle Frameworks](mlops_lifecycle_frameworks_guide.md) §4.4; here it is a modelling obligation.

### 7.2 What the Evidence Actually Says About "Best Algorithms"

Two findings are worth knowing because they contradict the common instinct to reach for the newest model:

- **Fernández-Delgado, Cernadas, Barro & Amorim, "Do we Need Hundreds of Classifiers to Solve Real World Classification Problems?"** (*JMLR* 15(90):3133–3181, 2014) evaluated 179 classifiers from 17 families on 121 datasets. Random forest variants were most often best (the best achieving 94.1% of maximum accuracy, exceeding 90% on 84.3% of datasets), with a Gaussian-kernel SVM implemented in C close behind (92.3%) and the difference between the top two **not statistically significant**. Their conclusion is the practically useful part: a handful of families — random forests, SVM, neural networks, boosting ensembles — dominate, and most of the 179 are not worth the search.
- **Caruana & Niculescu-Mizil, "An empirical comparison of supervised learning algorithms"** (ICML 2006) found that **ensemble selection** (calibrated, weighted ensembles of diverse model families) matched or beat the best single model in almost all of the problems studied, which is the empirical basis for "ensembles are a strong default" rather than "the newest architecture is".

The counterweight is **Wolpert's "No Free Lunch" result** — *The Lack of A Priori Distinctions Between Learning Algorithms* (*Neural Computation* 8(7):1341–1390, 1996) — which formalises that averaged over all possible problems, no learner is better than any other. This is not an argument that algorithm choice is arbitrary; it is an argument that **algorithm choice is only justified relative to a distribution of problems like yours**. The practical reading: benchmark on your data, with your label and your split, and treat published leaderboards as weak priors from a different distribution.

### 7.3 A Selection Procedure That Survives Review

| Step | Action | Guard |
|---|---|---|
| 1 | Define the decision-relevant metric and the cost asymmetry | Before seeing any results |
| 2 | Build the trivial and rule baselines | Reproduction of the incumbent |
| 3 | Fit 2–4 model families of differing inductive bias | Same features, same splits |
| 4 | Compare with uncertainty, not point estimates | Paired tests on the same splits; corrected for multiple comparisons |
| 5 | Calibrate if probabilities matter (§9) | Calibration on a held-out fold, never on training |
| 6 | Check stability across time slices and sub-populations | The model chosen on aggregate may be the worst for a critical segment |
| 7 | Freeze the choice, then document what it cannot do | §15 |

### 7.4 Model Complexity as a Cost

Complexity buys flexibility and charges rent in four currencies:

- **Latency and cost** — an ensemble of hundreds of trees scores differently from a single linear model; this is a modelling constraint when the serving budget is tight ([MLOps Lifecycle Frameworks](mlops_lifecycle_frameworks_guide.md) §4.6).
- **Explainability** — regulated decisions may require the principal reasons at the individual level (§14).
- **Operational surface** — more dependencies, more version skew, more to monitor.
- **Review burden** — a black box is not merely harder to explain; it is harder to *validate*, and validation effort is a modelling cost.

**A defensible default in applied work:** use the simplest model whose evaluation clears the bar by a margin that survives uncertainty; escalate to complexity when a documented, measured gain justifies the rent. Rudin's *Stop explaining black box machine learning models for high stakes decisions and use interpretable models instead* (Nature Machine Intelligence 1, 2019) is the strong form of this argument and is worth reading as an argument, even where an institution's risk appetite (or measured performance gap) leads it to a different balance. It is a position, not a settled consensus; the counter-position is that post-hoc explanation methods are adequate for governance when properly validated.

---

## 8. Honest Evaluation: Splits, Metrics and Uncertainty

Evaluation is the part of the craft most often performed as a ritual. The split is wrong, the metric is convenient, the comparison lacks uncertainty, and the result is a number nobody should act on.

### 8.1 The Split Is a Model of Deployment

Choose the split to mirror the deployment, not to maximise the number.

| Split | Mirrors | Use when | Failure it prevents |
|---|---|---|---|
| **Random holdout** | Exchangeable cases | i.i.d. rows, no entity repetition, no time | Nothing — and it is wrong for everything below |
| **Stratified** | Class proportions preserved | Imbalanced classification | Metric variance across folds |
| **Grouped (by entity)** | Unseen entities arriving | Multiple rows per customer/patient/device | Identity leakage (§6.3) |
| **Temporal (forward-chaining)** | Future cases arriving | Any time-ordered process | Optimism from training on the future |
| **Rolling-origin / walk-forward** | Repeatedly refitting as time passes | Production retraining | Single-period luck |
| **Blocked (spatial/hierarchical)** | Autocorrelated neighbours | Geo, species, network data | Neighbour leakage |
| **Nested** | Honest estimate of a *tuned* model | Model selection + performance in one number | Selection optimism (§8.3) |

**The default for applied business problems is temporal.** Random splits on temporal data answer the question "can the model recognise cases from this period", not "will the model work next month". Random splitting of temporal data is the most common structural error introduced in enterprises that have adopted ML recently.

**Grouping is non-optional when entities repeat.** The canonical guidance is Roberts et al., *Cross-validation strategies for data with temporal, spatial, hierarchical, or phylogenetic structure* (Ecography 40(8):913–929, 2017), which documents that standard cross-validation is invalid under dependence between observations — for time (blocked and forward-chaining schemes), for space (useful to remember that leave-one-group-out is not sufficient when there is autocorrelation beyond the group), and for hierarchical structures (split at the level at which independence is claimed). Their conclusion is blunt and portable to commercial data: **the split must be chosen to leave out entire structures of dependence**, and they explicitly note that a design whose folds are not statistically independent gives an optimistic estimate. This is the section's most transferable citation: the same reasoning applies to customers, accounts, geographic regions, and corporate groups.

### 8.2 Time-Series Splitting and the Financial Case

Two additional safeguards are standard in financial modelling:

- **Purging** — removing training observations whose label windows overlap the test period, so the test set is not partially *in* the training labels.
- **Embargo** — excluding a small buffer of observations immediately after the test period, because serial correlation makes near-boundary cases informative about the test set.

Both are named and developed in Marcos López de Prado's *Advances in Financial Machine Learning* (Wiley, 2018), whose chapter on cross-validation introduces the **purged k-fold** and **combinatorial purged cross-validation** schemes. Note the attribution carefully: purging and embargo are *named* in that book as the mechanism for the leakage created by overlapping label windows; the underlying concern (overlapping information between train and test) is the same one Kaufman et al. formalise (§6.1) and Roberts et al. address structurally. If you cite them, cite them as the book's named techniques rather than as a separate research finding.

### 8.3 Selection Optimism, or Why Your Best Number Is a Lie

If you try twenty configurations and report the best, the reported number is biased upward by the selection. This is not a metaphor; it is measurable. **Varma & Simon, "Bias in error estimation when using cross-validation for model selection"** (*BMC Bioinformatics* 7:91, 2006) shows that using the same cross-validation to select among many candidates and then reporting its error yields a substantially optimistically biased estimate, and that the bias grows with the number of candidates and the degree to which selection and estimation share data. The remedies are the nested split (selection in an inner loop, estimation in an outer loop), a strictly held-out set touched once, or a reported correction. Any claim of the form "we tried many models and this one achieved X" requires one of these, and a reviewer who does not ask for it is not doing their job.

### 8.4 Metrics: Choose Before You Look

| Metric | What it measures | Choose it when | Trap |
|---|---|---|---|
| **Base rate** | Prevalence | Always compute it | Reporting accuracy against it flatters |
| **Accuracy** | Overall correctness | Balanced classes, symmetric costs | Meaningless under imbalance |
| **Precision** | Of the flagged, how many are true | The intervention is costly per case | Ignores the missed positives |
| **Recall / sensitivity** | Of the true, how many are found | A miss is expensive (fraud, disease) | Ignores the false-alarm cost |
| **F1** | Harmonic mean of the two | Single-number summary wanted | Assumes equal value of precision and recall |
| **ROC-AUC** | Rank quality across all thresholds | Comparing rankers, balanced-ish data | Optimal under neither cost model; insensitive to the operating point |
| **PR-AUC** | Rank quality where positives are rare | Rare positives | Not comparable across datasets with different base rates |
| **K-S** | Maximum separation of score distributions | Credit scorecard convention | A single-point, threshold-dependent statistic |
| **Lift / gains at k** | Value of acting on the top k | Targeted campaigns with capacity limits | Depends on the chosen k |
| **Brier score / log loss** | Probability quality (proper scoring rules) | Probabilities feed a decision | Dominated by the majority class under imbalance |
| **Expected cost / value** | Money, using the institution's cost matrix | Any decision with quantified costs | Requires the cost matrix, which is a business artefact |
| **Calibration curve / ECE** | Reliability of the probability | Probabilities are consumed directly | Binned metrics are noisy |

**The right question is never "which metric?" but "what decision does this number feed, and what does that decision cost when it is wrong?"** For Cymbal Bank, an attrition model that ranks customers for a retention offer is judged by incremental value at the contacted population; a card scorecard is judged by its rank ordering and its regulatory fairness behaviour; a fraud detector is judged at a fixed alert capacity, where precision at capacity is the operative metric and AUC is a convenience.

### 8.5 Uncertainty: Report Intervals or Report Nothing

A point estimate without an interval is not evidence. Practical methods, in order of preference:

1. **Paired comparison on identical folds** — the same splits for both models, then a paired test on the fold-level differences. This is the cheapest correct comparison and it removes split variance.
2. **Bootstrap confidence intervals** — resample the evaluation set (respecting the grouping! resample *entities*, not rows) and report the interval of the metric.
3. **Repeated resampling** — repeat the whole evaluation across several splits and report the spread. If the spread exceeds the claimed improvement, there is no improvement.
4. **Correction for multiple comparisons** — report how many configurations were tried, and correct, or pre-register. Demšar's *Statistical Comparisons of Classifiers over Multiple Data Sets* (*JMLR* 7, 2006; page range unverified — see §18.2) is the standard reference for the comparison machinery — the Friedman test with a post-hoc correction for comparing multiple classifiers across multiple datasets — and is the right instrument whenever you are comparing more than two alternatives.

**A useful discipline:** report the metric with its interval, the base rate, the split scheme, and the size of the evaluation set, in one line. If the interval spans the baseline's interval, the honest statement is "no demonstrated difference".

---

## 9. Probability Calibration and the Confidence Question

A classifier's score is not a probability. For many applied decisions — pricing, expected-loss calculation, ranking by expected value, deciding a threshold against a cost matrix — the probability is the product, and an uncalibrated score is the wrong output.

### 9.1 Calibration in One Table

| Concept | Statement | Why it matters operationally |
|---|---|---|
| **Ranking** | The order is right | Enough for a targeting list |
| **Calibration** | The magnitude is right: among cases scored 0.2, about 20% are positive | Required when the score is multiplied by money |
| **Refinement** | The distribution is informative | Distinguishes models with the same calibration |

### 9.2 The Canonical Methods

| Method | Source | Shape | Notes |
|---|---|---|---|
| **Platt scaling** | Platt, "Probabilistic Outputs for Support Vector Machines and Comparisons to Regularized Likelihood Methods", in *Advances in Large Margin Classifiers* (MIT Press, 1999) | Logistic sigmoid fit to the scores | Designed for SVM margins; still the standard parametric default |
| **Isotonic regression** | Zadrozny & Elkan, "Transforming classifier scores into accurate multiclass probability estimates", KDD 2002 | Monotone step function | Non-parametric; needs more data and can overfit with little |
| **Temperature scaling** | Guo, Pleiss, Sun & Weinberger, "On Calibration of Modern Neural Networks", ICML 2017 (PMLR 70:1321–1330) | Single-parameter rescaling of logits | A one-parameter variant of Platt scaling; their headline result is that modern deep networks are *poorly calibrated*, and that temperature scaling is surprisingly effective on most datasets |
| **Beta calibration / histogram binning** | Various | Alternative parametric/histogram forms | Use where the sigmoid assumption is visibly wrong |

*Note on Platt's chapter:* the method and its attribution are unambiguous and are corroborated by a peer-reviewed follow-up — Lin, Lin & Weng, "A note on Platt's probabilistic outputs for support vector machines" (*Machine Learning* 68(3):267–276, 2007), DOI 10.1007/s10994-007-5018-6 — but this guide could not independently retrieve the 1999 book chapter's text or page numbers (see §18).

### 9.3 Which Models Need It

**Niculescu-Mizil & Caruana, "Predicting good probabilities with supervised learning"** (ICML 2005) is the empirical reference here: comparing a range of supervised learners on probability quality, they found that a model's *ranking* performance and its *probability* quality diverge — boosted trees and random forests tend to push scores toward the extremes, while methods that assume a particular shape (naive Bayes, and SVMs without calibration) are systematically miscalibrated — and that Platt scaling and isotonic regression both substantially improved the probability estimates of the learners they studied. Their practical conclusion is the one to carry: **if the decision consumes the probability, calibrate, and evaluate calibration (not just AUC) after calibration.** Guo et al. (2017) later showed the problem is not confined to the classical learners.

### 9.4 The Calibration Protocol

1. **Split off a calibration set** (or a fold) that was not used for fitting the model or the hyperparameters.
2. **Fit the calibrator on that set only.** Fitting the calibrator on training data produces a confidently wrong calibrator.
3. **Evaluate with a proper scoring rule** (Brier, log loss) and a calibration curve, *before and after*.
4. **Check calibration by segment**, not only in aggregate. A model that is well calibrated overall can be badly miscalibrated for a sub-population — which is simultaneously a fairness problem (§14) and a decision problem.
5. **Do not recalibrate after every retrain without re-checking.** The calibrator is a model; it decays too.

> **Note on the "classifier calibration" literature.** A frequently misremembered reference is a two-part "Classifier Calibration" paper, sometimes attributed to Cohen & Goldszmidt. This guide could not verify its existence, and the canonical calibration lineage does not run through it — it runs through Platt (1999), Zadrozny & Elkan (2002), Niculescu-Mizil & Caruana (2005) and Guo et al. (2017). If you have seen that title cited, treat it as unverified until you have the paper in hand. See §18.

---

## 10. Class Imbalance and the Contested Advice

Class imbalance is the topic on which the gap between folklore and evidence is widest. The folklore — "resample, and use SMOTE" — is widely implemented and weakly supported. The evidence says the *decision threshold* is the lever, not the class distribution.

### 10.1 What the Evidence Actually Says

**The strongest verified source is clinical-prediction research.** **van den Goorbergh, van Smeden, Timmerman & Van Calster, "The harm of class imbalance corrections for risk prediction models: illustration and simulation using logistic regression"** (*Journal of the American Medical Informatics Association* 29(9):1525–1534, 2022, DOI 10.1093/jamia/ocac093) examined the effect of popular imbalance corrections — random oversampling, random undersampling, SMOTE and variants, and class weighting — on the calibration of risk models. Their findings, in outline: the imbalance corrections **did not systematically improve** the model's discrimination or its overall performance, but they **did distort the predicted probabilities**, producing miscalibrated risk estimates that require correction before use. Their recommended alternative is to model the problem with the **original class distribution and adjust the decision threshold** to match the intended use.

This matters directly: in banking, the *probability* is often the product, so a technique that improves an AUC-flavoured summary while destroying the probability scale is a net harm.

| Approach | Claimed benefit | What the verified evidence shows | Verdict |
|---|---|---|---|
| **Class weighting** | Corrects imbalance | Does not systematically improve performance; alters the probability scale | Use with a recalibration step if the probability matters |
| **Random undersampling** | Balances classes | Discards data; harms calibration in the van den Goorbergh results | Usually the worst option |
| **Random oversampling** | Balances classes | Duplicates cases; harms calibration; no demonstrated systematic performance gain | Rarely worth it |
| **SMOTE / synthetic minority** | Better minority representation | Chawla et al. (JAIR 2002) introduced it and it is highly cited, but its benefit is **contested** — it is included among the corrections whose systematic benefit the clinical evidence does not support, and it creates synthetic points that may not lie on the data manifold | Treat as unproven; validate on your split against a class-weighted and threshold-tuned baseline before adopting |
| **Threshold adjustment** | Moves the operating point to maximise the chosen utility | Achieves the decision objective without distorting the probability model | **The recommended default** |
| **Cost-sensitive learning** | Encodes the asymmetric cost directly | Equivalent objective to thresholding with the same cost matrix | Correct when the cost matrix is genuinely known |

### 10.2 What Is Contested, Stated Plainly

- **"SMOTE helps"** — contested. It is a real, peer-reviewed method (Chawla, Bowyer, Hall & Kegelmeyer, "SMOTE: Synthetic Minority Over-sampling Technique", *Journal of Artificial Intelligence Research* 16:321–357, 2002, DOI 10.1613/jair.953) with a large following, and there exist datasets where it helps. The claim that it helps *systematically*, or that it should be a default step, is not supported by the evidence reviewed here, and the strongest recent verified evidence points the other way for probability-producing models.
- **"Accuracy is useless under imbalance"** — mostly sound, and the reason is arithmetic, not empirical.
- **"ROC-AUC is fine under extreme imbalance"** — contested. ROC-AUC is insensitive to the base rate, which is exactly why it can look healthy while the model is useless at the operating point you actually use; PR-AUC (or precision at capacity) is the more informative summary, at the cost of comparability across datasets.
- **"Resampling is necessary because the model will ignore the minority class"** — usually false. Most modern learners optimise a loss, not an accuracy count, and many handle imbalance adequately with appropriate thresholds.

### 10.3 The Applied Recipe

1. **Do not resample by reflex.** Start with the natural distribution.
2. **Pick the operating point deliberately** — from the cost matrix, the capacity constraint, or a target precision/recall. This is where the "imbalance handling" actually happens.
3. **If the probabilities feed a calculation, check calibration both before and after any correction.**
4. **If you do apply a correction, evaluate the final pipeline end-to-end at the chosen operating point**, not the resampled model's AUC.
5. **Report the base rate and the chosen threshold with every metric.** A precision without a threshold is not a result.

---

## 11. The Error-Analysis Iteration Loop

Error analysis is the discipline that separates modellers from people who run AutoML. The metric tells you *that* the model is wrong; error analysis tells you *how*, and it is usually the only thing that produces a real improvement after the first week.

### 11.1 The Loop

```
   ┌──────────────────────────────────────────────────────────────┐
   │  1. Establish the honest baseline (split + metric + CI)      │
   │  2. Find the worst-performing SLICES, not the worst average  │
   │  3. Read the actual errors (a sample, by hand)               │
   │  4. Form one hypothesis about the mechanism of failure       │
   │  5. Change ONE of {label, population, features, model}       │
   │  6. Re-measure on the SAME split; keep or discard            │
   └──────────────────────────────────────────────────────────────┘
```

**Hyperparameter search is step 5 with the weakest expected return.** It is measurable, so it is over-used. The larger returns are usually one step earlier: a wrong label, a missing population, or a feature that does not exist yet.

### 11.2 Slicing: The Main Instrument

Aggregate metrics hide failures. Slice systematically, and report the metric *per slice* alongside the aggregate:

| Slice by | Reveals |
|---|---|
| **Time** (month, vintage, cohort) | Decay, regime change, seasonality, a broken upstream feed |
| **Entity attributes** (segment, region, channel, tenure band) | Sub-population failure; fairness exposure (§14) |
| **Score band / confidence** | Where the model is right for the wrong reason; where it is effectively a coin flip |
| **Feature-value extremes** | Extrapolation outside the training support |
| **Missingness flags** | Whether imputation is a silent source of error |
| **Data-source / pipeline origin** | A specific feed's quality problem |
| **Outcome latency** | Whether recent cases are systematically harder (label maturity) |

**Report slices with sample sizes.** A slice with 30 cases and a spectacular failure rate is a hypothesis, not a finding. This is the same discipline as the CI requirement in §8.5, applied within the model.

### 11.3 Reading the Errors: The Audit

The highest-yield activity in applied ML is reading a random sample of misclassified cases and writing down what you see. Do 50 to 100 cases, in a shared document, with columns for: the case, the prediction, the truth, and *your best guess at why*. Patterns emerge that no dashboard shows.

| What you find | Likely mechanism | The fix that follows |
|---|---|---|
| Errors concentrate on cases where a key value is missing | Missingness carries signal; imputation erased it | Missing-indicator feature, or a segmented model |
| Errors concentrate in one recent month | Drift, or a downstream business change | Investigate the process change; consider retraining (hand to MLOps §4.9) |
| The truth label looks wrong for several cases | Label quality problem (§3.3) | Fix the label definition or the extraction |
| The model is confidently wrong on coherent groups | Spurious correlation; leakage from an unmodelled process | Look for a shortcut feature |
| Errors look like the *opposite* of the truth | Sign/target-coding error | Check the label encoding |
| Errors are near-duplicates of training cases classified correctly | Entity leakage across folds | Re-split by group (§8.1) |
| Errors are genuinely hard cases with insufficient information | The ceiling is information, not modelling | Report the ceiling; reframe the problem |

### 11.4 The Prioritisation Matrix

Not every error class is worth fixing. Rank by **frequency × cost × fixability**:

| | Cheap to fix | Expensive to fix |
|---|---|---|
| **Frequent + costly** | Do it now | Justify the investment; this is the project's case |
| **Frequent + cheap** | Do it if it is a config change | Usually ignore |
| **Rare + costly** | Handle by rule outside the model | Model it only if the volume justifies it |
| **Rare + cheap** | Ignore | Ignore |

**A frequent finding:** the highest-value fix is not a model improvement but a **business-rule guardrail** — a deterministic override for a known case class the model cannot learn because the data does not contain the relevant variable. Modellers who propose these, rather than pretending the model should handle everything, are more useful than ones who do not.

### 11.5 Data-Centric Versus Model-Centric Iteration

The "data-centric AI" framing — the argument that, once the architecture is adequate, further effort is better spent improving the *data* (labels, consistency, coverage) than the *model* — was popularised by **Andrew Ng** in a series of talks, essays and a competition around 2021 (associated with Landing AI and DeepLearning.AI), rather than originated in a peer-reviewed paper. This guide could not verify a single canonical written source and therefore attributes it as a **practitioner campaign**, not as a research finding (§18). Its practical content is nonetheless sound and is what this section has described procedurally: systematic error analysis, label quality, and slice-level data improvement. Treat the slogan as a heuristic that is often right, not as a law that is always right.

**Where model-side work genuinely wins:** when the data is already clean and dense, the feature set is rich, and the remaining gap is a representational one — non-linear interactions, sequence structure, or a genuinely different inductive bias. Recognise this only after the data-side options are exhausted; the reverse order wastes quarters.

---

## 12. Applied Versus Research Trade-offs

Research and application optimise different things, and confusing them produces both bad papers and bad systems.

### 12.1 What Each Side Optimises

| Dimension | Research | Applied |
|---|---|---|
| **Objective** | Novelty, state-of-the-art on a shared benchmark, generalisable claim | Decision value under operational, legal and cost constraints |
| **Evidence** | Statistical significance of a delta versus published baselines | Demonstrated improvement over the incumbent on the institution's own data, with uncertainty |
| **Data** | Curated benchmark; often i.i.d.-assumed splits | Messy, drifting, selectively labelled, with a process history |
| **Success** | Accepted and replicated | Used, maintained, and defensible in an audit |
| **Failure mode** | No result | A result the business cannot use, or uses wrongly |
| **Time horizon** | Until the result is solid | Until the value is realised and the model retired |

### 12.2 The Paper-to-Production Gap

The gap is real and structural, and it is worth naming the reasons precisely so they are not mysterious:

1. **Benchmark leakage.** Several well-known benchmark competitions were affected by leakage — this is documented, not folklore: Kaufman et al. (§6.1) discuss leakage in the KDD-Cup 2008, INFORMS 2010 and IJCNN 2011 challenges directly, and note that public leaderboards can be gamed from external sources. A result on a leaky benchmark is not a result about the task.
2. **Split mismatch.** Paper splits are usually random; deployment splits are temporal and grouped.
3. **Selection on the test set.** Much reported tuning is performed against a held-out set repeatedly, which is the optimism Varma & Simon quantify (§8.3).
4. **Baseline weakness.** A paper's baseline is often an untuned or outdated implementation rather than the tuned incumbent.
5. **Missing pipeline.** Papers rarely ship the preprocessing, the feature computation, and the inference-time constraints. The model is a small fraction of the system — the "glue code" and pipeline cost of an ML system is the argument of Sculley et al.'s *Hidden Technical Debt in Machine Learning Systems* (NeurIPS 2015), whose inventory (entanglement and the CACE principle, correction cascades, undeclared consumers, unstable data dependencies, configuration debt) is a catalogue of what a paper does not contain.

**Applied reading of a paper:** extract the *idea* and its inductive bias, then re-establish it on your data with your split and your baseline. Do not transfer the reported number.

### 12.3 When to Reach for Research Deliberately

- **The ceiling is representational.** Tabular modelling is saturated by gradient-boosted trees and regularised linear models; the marginal gain from a novel architecture is usually small. Sequence, image, text and graph problems are different, and architecture matters more there.
- **The label is the bottleneck.** No architecture fixes a bad label; go back to §3.
- **The problem is genuinely new.** Then a literature review is the cheapest work available, and the cost of not doing it is re-deriving a known failure.
- **Regulatory defensibility requires a standard method.** Novel methods are harder to validate and to defend to a reviewer; the model-risk function will ask what the incumbent literature says (§14).

### 12.4 The Time Budget Heuristic

In the applied setting, a defensible allocation of effort across a modelling project is roughly: framing and label definition first (always under-invested), then data and features, then evaluation infrastructure, and *last* model class and hyperparameters. The common failure is an inverted order — a week of tuning on a wrong label, a random split, and a metric that does not feed a decision.

---

## 13. The Production Handoff

The modeller's job ends at a defined boundary, and that boundary is worth drawing explicitly because "throw it over the wall" is where projects die. This section states what the modeller owes and what happens to the model afterwards.

### 13.1 What Happens After the Handoff (Owned Elsewhere)

Named here in one line each; the detail lives in [MLOps Lifecycle Frameworks](mlops_lifecycle_frameworks_guide.md):

| Topic | Where the detail lives |
|---|---|
| Deployment strategies: shadow, canary, blue-green, A/B | MLOps §4.6 |
| Batch, real-time and streaming serving | MLOps §4.6 |
| Model registry, versioning, CI/CD/CT | MLOps §5 |
| Monitoring: PSI, K-S, drift dashboards, alerting | MLOps §4.7 |
| Retraining triggers and cadence | MLOps §4.9 |
| Governance, risk tiering, audit artefact storage | MLOps §4.8, §9 |
| Feature computation, online/offline consistency, point-in-time correctness | [Feature Store](feature_store_guide.md) §6, §7 |
| Operational failure modes (last-mile integration, insufficient production testing, drift-type confusion) | MLOps §12 |

### 13.2 What the Modeller Owes the Pipeline

A model is not a handoff; a **model package** is. The minimum contents, each of which is an artefact with a version:

| Artefact | Why the pipeline needs it |
|---|---|
| **Frozen model artefact** + the exact library versions used to train it | Reproducibility; incompatible serialisation is a classic failure |
| **Preprocessing and calibration objects, frozen alongside** | Fitting these at serving time is train/serve skew |
| **Feature specification** — names, definitions, window semantics, source, and the "as-of" rule | Without it, the serving implementation is a guess |
| **Label definition and observation anchor** | So that monitoring can reconstruct the ground truth as it matures |
| **Evaluation report** — split scheme, metric, interval, base rate, slices | The baseline against which production performance is judged |
| **Assumptions register** — extrapolation claims (e.g. reject inference), population limits, known blind spots | The reviewer's first question; also the monitoring design's input |
| **Monitoring requirements** — which inputs to watch, which slices to track, what "bad" looks like, and the metric whose movement should trigger a review | The modeller knows which features are load-bearing; only they do |
| **Latency and throughput budget** the model was built to | A model that cannot meet serving SLA is not a deployable model |
| **Interpretation notes** — directionality per feature, and how to produce a human-readable reason | Required for adverse-action style communication (§14) |

### 13.3 Train/Serve Skew Is a Modelling Responsibility

Skew — the difference between the feature value as computed in training and as computed at serving — is *implemented* by the platform but *specified* by the modeller. The modeller's contribution:

- **Declare the "as-of" semantics of every feature** ("balance as of end of day before the scoring date", not "current balance").
- **Declare handling of late-arriving and corrected data** — the offline table may be a restatement, while the online value was the original. If so, say which one the model was trained on.
- **Specify default/missing behaviour** for values that will be absent at serving time.
- **Provide a skew test**: a defined comparison of offline and online feature distributions on the same requests. What to alert on is operational (MLOps §4.7); *what to compare* is modelling.

### 13.4 Decay, and Why the Modeller Must Name the Drift Risk

Models decay. The modelling discipline is to say *which* kind of change the model is exposed to, because the right response differs:

| Change | What moves | Reference |
|---|---|---|
| **Covariate shift** (input distribution) | P(X) changes; P(Y\|X) is stable | Moreno-Torres et al., "A unifying view on dataset shift in classification", *Pattern Recognition* 45(1):521–530, 2012; and the edited volume *Dataset Shift in Machine Learning* (Quiñonero-Candela, Sugiyama, Schwaighofer & Lawrence, MIT Press) |
| **Prior probability shift** (label base rate) | P(Y) changes | Same taxonomy |
| **Concept drift** | P(Y\|X) itself changes — the relationship the model learned is no longer true | Gama, Žliobaitė, Bifet, Pechenizkiy & Bouchachia, "A survey on concept drift adaptation", *ACM Computing Surveys* 46(4), 2014 |

The distinction matters operationally: under pure covariate shift a recalibration may suffice; under concept drift the model must be refit or replaced. This is why the modeller, not only the platform team, must state which risk dominates — the MLOps sibling covers the *metrics* (PSI, K-S) and warns against drift-type confusion in its §12.9; the *judgement* about which drift the model is exposed to is modelling.

### 13.5 The Offline–Online Gap

It is a widely repeated practitioner observation that a model's offline improvement does not always translate into a production gain. This guide could **not** verify a published study that measures this gap in general terms; the accounts are typically practitioner reports and blog posts from search, advertising and recommender deployments, and they are best attributed as practitioner claims rather than as research findings (§18). The likely mechanisms are nonetheless well understood and are partly the subject of the verified literature: feedback loops and undeclared consumers (Sculley et al., NeurIPS 2015), definitional differences between the offline metric and the business outcome, and interference between the model's actions and the data it is then evaluated on.

**Practical stance:** treat the offline result as a *filter* (does this candidate deserve a live test?) rather than as a *forecast* (this is the gain we will get). Where a shadow or A/B test is feasible, it is the only measurement that settles the question — the mechanics of which belong to MLOps §4.6.

### 13.6 The Good-Enough Decision

Declaring a model production-ready is a modelling judgement with three parts:

1. **Statistical** — the improvement over the incumbent exceeds uncertainty, on the deployment-like split, across the slices that matter.
2. **Operational** — the model meets the serving contract, and the owning team accepts the monitoring and retraining obligation.
3. **Governance** — the model is validated and documented to the institution's standard (§14, §16).

If any part is missing, the honest status is "not ready", and the correct action is to say so in writing. A modeller who cannot say "not ready" is not providing a judgement; they are providing a delay with extra steps.

---

## 14. Modelling Under Regulation: The Banking Constraints

Regulation is not a review stage bolted on at the end; it is a set of modelling constraints that decides the model class, the output format, and the evidence bar. Because this repository is written from a banking vantage point, this section names the regimes and, where verification succeeded, cites the primary source. Anything unverified is flagged in §18.

### 14.1 The Constraints, and What Each One Does to the Model

| Constraint (jurisdiction) | Source | Effect on modelling |
|---|---|---|
| **Adverse-action reasons** (US) | Equal Credit Opportunity Act; **Regulation B, 12 CFR § 1002.9** (verified: notification must contain, or offer on request, *a statement of specific reasons* for adverse action) | The model must be able to produce a small number of *specific, accurate, individual-level* reasons. This constrains model class, feature set, and the explanation method |
| **CFPB interpretive guidance on complex models (withdrawn)** | CFPB **Circular 2023-03**, "Adverse action notification requirements and the proper use of the CFPB's sample forms provided in Regulation B", 89 FR 27361 (Apr 17, 2024) — **withdrawn by the CFPB on 12 May 2025** (90 FR 20084) | The *interpretive* guidance that creditors must disclose the actual principal reasons rather than relying on the sample checklist no longer stands. The underlying ECOA/Regulation B obligation remains, and supervisory expectations may still be applied. **Do not cite the circular as current law** |
| **Model risk management** (US) | **SR 11-7 / OCC Bulletin 2011-12** (2011) — replaced by the interagency **"Model Risk Management: Revised Guidance"**, announced in **OCC Bulletin 2026-13 (Apr 17, 2026)**, which rescinds OCC 2011-12, OCC 1997-24 and OCC 2021-19 | Expect documented conceptual soundness, an independent validation, outcomes analysis, and ongoing monitoring. The 2026 guidance is principles-based (not enforceable standards), excludes generative/agentic AI from scope, and is aimed at banks over ~$30bn in assets |
| **Credit scoring as high-risk AI** (EU) | **EU AI Act**, **Article 6(2)** with **Annex III(5)(b)**: AI systems to *evaluate the creditworthiness of natural persons or establish their credit score* (except fraud detection) are high-risk (verified) | Triggers the Act's high-risk obligations: risk management, data governance, technical documentation, logging, human oversight, accuracy and robustness |
| **Automated decisions** (EU) | **GDPR Article 22** (verified): right not to be subject to a decision based solely on automated processing with legal or similarly significant effects, subject to exceptions and safeguards including human intervention and the right to contest | Where the decision is fully automated, the modelling must support human review and contestation — an operational and explanatory requirement |
| **IRB model validation** (Basel) | Basel Committee, **"Studies on the Validation of Internal Rating Systems (revised)"**, Working Paper No. 14, Feb 2005 (revised May 2005) | For internal-ratings-based capital models, validation is a supervisory expectation with a defined content: conceptual soundness, data quality, quantitative back-testing, and governance |
| **FEAT principles** (Singapore) | MAS FEAT — Fairness, Ethics, Accountability, Transparency — for AI in the financial sector (2018), with subsequent assessment work under the Veritas banner | Sets an expectation of documented fairness assessment and explainability even where not statutory. **Verified only partially — see §18** |

**The single most important practical point:** the constraint that bites hardest is the **reason-code requirement**. It rules out some model classes for some decisions, it makes explanation quality a *functional* requirement rather than a nice-to-have, and it must be designed in at feature-selection time — not retrofitted.

### 14.2 Implications for the Model Class

| Decision | Regime pressure | Typical modelling consequence |
|---|---|---|
| **Credit approval / pricing** (retail) | Adverse-action reasons; high-risk classification in the EU; MRM validation | Scorecard-shaped models (monotone binning, additive points) or constrained monotone models are the defensible default; complex models require an explanation layer and a stronger validation case |
| **Collections / limit management** | Same as above, plus customer-duty expectations | Reason codes and documentable thresholds |
| **Fraud detection** | Explicitly carved out of the EU high-risk credit-scoring annex entry | More latitude on model class; precision at alert capacity and explainability to investigators are the operative constraints |
| **AML / transaction monitoring** | Model risk expectations extended to BSA/AML systems | Alert volume and false-positive cost are the binding constraints; validation must address tuning |
| **Marketing propensity** | GDPR lawful basis; fairness expectations | Model class is largely free; fairness and consent constraints dominate |

### 14.3 Fairness: A Modelling Decision, Not a Post-Hoc Test

Fairness enters at four points, and only the first is usually discussed:

1. **Feature choice** — proxy variables for protected characteristics (postcode, name-derived features, device) carry disparate-impact risk even when the protected attribute is absent.
2. **Label choice** — labels that encode historical discrimination teach it. A target defined by "was this customer profitable" inherits historical lending patterns.
3. **Evaluation** — report metrics by protected group, with intervals and sample sizes (§11.2), and be explicit about which fairness definition is being used. Definitions conflict; there is no universally correct one, and choosing one is a governance decision, not a statistical one.
4. **Threshold** — a single global threshold applied to differently-distributed groups produces different error rates. Whether that is acceptable is an institutional and legal judgement.

**Discipline:** name the fairness definition and the measurement approach in the model documentation, state who chose them, and state the limitation. A model card that says "we did not assess fairness" is more honest and more useful than one that implies an assessment that was not performed. Reporting format and validation reporting are covered in [MLOps Lifecycle Frameworks](mlops_lifecycle_frameworks_guide.md) §4.5 and §4.8.


---

## 15. The Cymbal Bank Worked Example: One Project, End to End

**Explicitly illustrative.** Cymbal Bank is a fictional institution used as the running example across this repository's ML guides; the project below is an invented composite that exercises every section of this guide in sequence. The numbers are illustrative and are not claims about any real institution or dataset.

### 15.1 The Framing

| Rung (§2.1) | Content |
|---|---|
| **Complaint** | "We lose term deposits at maturity and we do not know whom to call." |
| **Decision** | Which depositors to contact in the seven days before maturity, given a call centre that can make ~2,000 calls a week |
| **Quantity** | Probability of rollover at maturity — *and* for the retention question, the incremental effect of a call |
| **Learning task** | Binary classification, one row per (customer, maturity event) |
| **Operational contract** | Score at T-7 days; features must be available T-7; reasons for the top adverse drivers; batch scoring nightly |

Two framing decisions were made before any modelling. First, the **decision** is a ranked call list under a capacity constraint, so the metric will be precision/value at the contacted volume, not accuracy. Second, the underlying business question is causal ("would a call change the outcome?") while the model is predictive — so the model estimates rollover propensity, and the uplift question is explicitly deferred to a holdout-based incrementality test rather than assumed from the propensity score (§2.2).

### 15.2 The Label, and Its Leakage Risks

The label is "the deposit did not roll over within 14 days of maturity." The specification written down (§3.1): observation anchor = maturity date; outcome window = 14 days; censoring rule = events whose window has not closed are **excluded, never labelled negative**; maturity requirement = the deposit must have been open ≥ 90 days.

Three leakage risks were identified at this stage and each needed a control:

- **Label-proxy features.** "Number of contacts since last maturity" and "balance change in the last week" are consequences of the *decision* to call, not of the outcome. They would leak if the historical call log were joined without a timestamp cut-off — the join is restricted to records strictly before T-7 (§6.2, joining).
- **Temporal.** Product-version fields and rate-change flags are known only after the maturity event and were excluded (§6.2, aggregation).
- **Entity.** A customer with two deposits has two rows; if one rolls and one does not, rows are correlated and the split must group by customer (§8.1).

### 15.3 The Baseline

Three baselines were built before any modelling (§7.1): the **trivial** one (predict the base rollover rate, ~62%), the **rule** actually in force (call any depositor with above-median balance maturing in the next week — the incumbent), and a **simple model** (logistic regression on six variables: balance, tenure, product type, maturity amount, prior rollover count, rate-differential flag). The business case rests on beating the *rule*, not the trivial baseline: the rule is what delivers today's result.

### 15.4 The Features and the Leakage Audit

Feature families drawn from §5.2: balance aggregates over 30/90 days, tenure and product attributes, prior-maturity behaviour counts, rate differential against the current market rate, and a channel-engagement count. Before evaluation, the plausibility audit (§6.4) was run on the top twenty features by importance, each written down with *when it becomes known*. It found one feature that became known after maturity (excluded) and one that was a near-duplicate of the label definition (a "maturity instruction" flag recorded only when the customer actually gave an instruction — a textbook primary leak, removed). The ablation check (§6.4) confirmed the model's performance dropped to a plausible level afterwards.

### 15.5 The Evaluation Design

- **Split.** Forward-chaining by maturity date: train on maturities up to a cut-off, evaluate on the following quarter, grouped by customer within each fold (§8.1) — the deployment question is "will this work on next quarter's maturities", so a random split would answer a different and easier question.
- **Metric.** Precision at the 2,000-call weekly capacity, plus expected value per contact using the institution's contact cost and the deposit's margin — the decision-relevant metric (§8.4). PR-AUC is reported as a secondary summary; ROC-AUC is reported only to show that it flatters the result.
- **Calibration.** The propensity feeds an expected-value calculation, so it must be a probability, not a score: isotonic calibration on a held-out fold, checked with a reliability curve and the Brier score before and after (§9.4).
- **Uncertainty.** Paired comparisons on identical folds with bootstrap intervals over *customers*, not rows (§8.5). A claimed gain inside the interval is not a gain.

### 15.6 The Error-Analysis Pass That Changed the Plan

Reading 80 misclassified cases (§11.3) produced three findings that no dashboard had shown:

1. Errors concentrated on customers whose **contact-preference flag was missing** — missingness carried signal and imputation had erased it. Fix: missing-indicator feature (§11.3).
2. A coherent block of errors were deposits that **matured but were re-opened internally** — the label's own extraction treated an internal transfer as a lapse. This was a **label defect**, not a model defect, and fixing it changed the training data (§3.3).
3. The model was confident and wrong on a small group of **deceased or dormant accounts**, which the variable set could not distinguish. Fix: a **business-rule guardrail** — a deterministic suppression rule outside the model, since no feature in the data could separate the group (§11.4).

The measured model gain after these three changes exceeded the gain from the hyperparameter search that had preceded them — which is the practical form of §11.5's argument.

### 15.7 The Handoff and the Monitoring Design

The model package (§13.2) handed over: the frozen artefact with pinned library versions, the calibration and preprocessing objects, the feature specification with its "as-of T-7" rule for every field, the label definition and anchor (so ground truth can be reconstructed as windows close), the evaluation report with its split scheme and intervals, the assumptions register, and the monitoring requirements. The modeller named **which drift risk dominates** (§13.4): primarily covariate shift from interest-rate movement, which the platform's input-distribution monitoring can lead on, with the probability of concept drift rising in a rate-shock regime. Retraining cadence and deployment mechanics belong to [MLOps Lifecycle Frameworks](mlops_lifecycle_frameworks_guide.md) §4.6, §4.7 and §4.9.

### 15.8 The Governance Gates

The model was tiered as **medium** materiality in the institution's own scheme, which sets the validation depth and the documentation bar. The evidence pack that went to the reviewers was assembled from the phases above — problem statement, label specification, population and exclusions, leakage review, baseline comparison, deployment-like evaluation with intervals, calibration evidence, error analysis with the known blind spots, the assumptions register, the fairness assessment, and the reason-code mechanism. No part of that pack was produced at the last minute; each was an artefact of the phase that created it.

### 15.9 What This Project Still Cannot Establish

Stated plainly, because the honesty here is the deliverable:

- **It cannot claim the calls caused the retention.** The model ranks; only a holdout-based incrementality test measures the effect of contact (§2.2).
- **It cannot speak for the unmatured population.** The most recent maturities were excluded by the censoring rule (§3.1), so nothing is known about the newest product version.
- **It cannot establish stability.** One quarter of forward-chaining evaluation is one observation, not a stability result.
- **It cannot claim fairness-neutrality for protected groups.** Group metrics were reported with sample sizes, and some groups were too small for a conclusion (§14.3).
- **It cannot guarantee the serving implementation matches the evaluation.** The skew test is specified (§13.3) but its first result arrives in production.

### 15.10 The Artefacts of the Craft

The condensed sequence a reviewer should expect to see, in the order the phases produce them — the framing memo, the versioned label specification, the dataset description with exclusions, the feature specification with as-of rules, the leakage review note, the split definition, the baseline comparison table, the pre-registered metric with its operating point, the model selection report, the calibration evidence, the error-analysis log, the explanation mechanism, and the model package with its "cannot do" list. A project that cannot produce these is not at a reviewable stage, whatever its headline metric says.

---

## 16. Explainability, Documentation and Model Review

This section owns one question: **what must the modeller be able to explain about a prediction, and about the model, at the level of an individual decision?** Model card templates, validation report formats and registry metadata belong to [MLOps Lifecycle Frameworks](mlops_lifecycle_frameworks_guide.md) §4.5; here is the modelling content that goes into them.

### 16.1 Three Different Claims, Often Confused

| Claim | Question | Instrument | Legitimate use |
|---|---|---|---|
| **Attribution** | Which inputs moved *this score*? | SHAP-style additive attribution (Lundberg & Lee, NeurIPS 2017) | Reason codes; debugging; investigation |
| **Global importance** | Which features matter across cases? | Permutation importance, SHAP aggregates | Feature selection, monitoring priority |
| **Causation** | What would happen if we changed this input? | Experiment or causal design | Deciding whether to intervene |

**Attribution is not causation**, and adverse-action reason codes are attribution claims ("this factor contributed to the decision"), which is why they are defensible. A marketing decision that assumes a reason code is an actionable lever is making a causal claim on attribution evidence. Keep the three apart in documentation, because a reviewer will test whether you have.

### 16.2 Reason Codes That Survive Scrutiny

- **They must be accurate at the individual level.** A global top-10 importance list is not a reason code.
- **They must be few.** The regulatory expectation is a small number of principal reasons, not a ranked dump.
- **They must be in the applicant's language.** "Variable V37 below threshold" is not a reason.
- **They must be stable.** Non-deterministic explanation methods produce different reasons for the same applicant on repeated calls, which is indefensible in a dispute.
- **They must be consistent with the model.** A reason code that contradicts the sign of the feature's effect is a validation finding waiting to happen.

**Modelling consequence:** exactly representable reason codes are easiest to produce from additive, monotone models. Where a complex model is chosen, the explanation layer becomes a first-class component with its own validation — budget for it in the project, not in the remediation.

### 16.3 Documentation as a Modelling Artefact

Two verified instruments are worth adopting, both independent of platform: **Model Cards for Model Reporting** (Mitchell, Wu, Zaldivar, Barnes et al., FAT\* 2019) — intended use, performance disaggregated by group, limitations — and **Datasheets for Datasets** (Gebru, Morgenstern, Vecchione, Vaughan et al., 2018; *Communications of the ACM*, 2021) — provenance, composition and collection process. Their value here is that disaggregated performance forces the slice analysis of §11.2, and provenance makes §4 and §6 reviewable.

**The modeller's addition:** an explicit **"what this model cannot do"** list. Every model has one. Writing it down converts an unknown risk into a managed one, and it is the single most valued paragraph in a review.

---

## 17. Anti-Patterns and Failure Modes of the Modelling Craft

These are modelling-specific. The *operational* failure modes — ML not treated as engineering, skipped monitoring, undetected decay, the DS-versus-engineer divide, tool overload, compliance as an afterthought, insufficient production testing, last-mile integration, drift-type confusion — are catalogued in [MLOps Lifecycle Frameworks](mlops_lifecycle_frameworks_guide.md) §12 and are not repeated here.

| # | Anti-pattern | Symptom | Fix |
|---|---|---|---|
| 1 | **Metric-first framing** | Work begins with a dataset and a metric; the decision is never named | §2.1; write the framing memo first |
| 2 | **The convenient label** | Label chosen because it exists in a table, not because it defines the event | §3.1; specify the six components |
| 3 | **Censoring blindness** | Recent cohorts scored as negatives | §3.1; matured cohorts only |
| 4 | **Random split on temporal data** | Offline metric excellent, production mediocre | §8.1; temporal or rolling-origin splits |
| 5 | **Preprocessing outside the loop** | Scaler/imputer/encoder fit on all data | §6.2; fit inside resampling |
| 6 | **Entity leakage across folds** | Duplicated customers in train and test | §6.3, §8.1; group the split |
| 7 | **Tuning on the test set** | "Best of 40 configurations" reported as an estimate | §8.3; nested split or a once-touched holdout |
| 8 | **Imbalance cargo-culting** | SMOTE applied by default because it is a default | §10; threshold first, correct only if justified |
| 9 | **Accuracy under 1% prevalence** | 99% accuracy reported with no operating point | §8.4; PR-AUC or precision at capacity |
| 10 | **Point estimate worship** | A 0.002 AUC gain declared a victory | §8.5; intervals, paired folds |
| 11 | **Hyperparameter theatre** | Weeks of search, no error analysis | §11; one change at a time, hypotheses first |
| 12 | **Uncalibrated probabilities in a money calculation** | Expected loss computed from raw scores | §9; calibrate on a held-out fold |
| 13 | **Explanation as an afterthought** | Reason codes retrofitted to a model that cannot produce them | §14.2, §15.2; design in from feature selection |
| 14 | **SOTA-chasing** | A novel architecture chosen over a tuned gradient-boosted baseline with no measured gain | §7.1, §12.2; outperform the tuned baseline or explain why not |
| 15 | **Silent assumptions** | Reject inference, extrapolation, population limits undocumented | §13.2; assumptions register |
| 16 | **The un-refusable model** | Nobody can say "not ready" | §13.6; the three-part good-enough test |

---

## 18. The Claims Audit, and What Could Not Be Verified

### 18.1 The Claims Audit

Every substantive claim in this guide, with its status and the quality of the source behind it. "Verified" means the source was located and read; a claim whose source is a peer-reviewed paper is stronger than one resting on practitioner reporting, and the table says which is which.

| Claim | Status | Source and its quality |
|---|---|---|
| Leakage has a formal formulation with named source categories, and the remedy is learn–predict separation | **Verified** | Kaufman, Rosset, Perlich & Stitelman, KDD 2011, pp. 556–563; journal version *ACM TKDD* 6(4), 2012. Peer-reviewed primary; the taxonomy and definition used in §6.1 are the paper's own |
| Cross-validation is invalid under dependence between observations; the split must leave out whole structures of dependence | **Verified** | Roberts et al., *Ecography* 40(8):913–929, 2017. Peer-reviewed primary |
| Using one cross-validation for selection and estimation yields optimistically biased error | **Verified** | Varma & Simon, *BMC Bioinformatics* 7:91, 2006. Peer-reviewed primary |
| Class-imbalance corrections do not systematically improve performance and distort predicted probabilities | **Verified, scope-limited** | van den Goorbergh et al., *JAMIA* 29(9):1525–1534, 2022. Peer-reviewed primary; the evidence is from a simulation and logistic-regression setting, not universal |
| SMOTE is a real method, but its systematic benefit is contested | **Verified as contested** | Chawla et al., *JAIR* 16:321–357, 2002 (the method); the benefit claim is contradicted by the imbalance evidence above. The literature is mixed, not settled |
| Platt scaling, isotonic regression and temperature scaling are the canonical calibration methods | **Verified** | Zadrozny & Elkan, KDD 2002; Niculescu-Mizil & Caruana, ICML 2005; Guo et al., ICML 2017 (PMLR 70:1321–1330). Platt's 1999 chapter is only partially verified — item 3 below |
| Ranking quality and probability quality diverge across learners | **Verified** | Niculescu-Mizil & Caruana, ICML 2005. Peer-reviewed primary |
| Modern deep networks are poorly calibrated, and temperature scaling is effective | **Verified** | Guo et al., ICML 2017 |
| Dataset-shift and concept-drift taxonomies | **Verified** | Moreno-Torres et al., *Pattern Recognition* 45(1):521–530, 2012; Gama et al., *ACM Computing Surveys* 46(4), 2014 |
| Purged k-fold, purging and embargo | **Verified as named book techniques** | López de Prado, *Advances in Financial Machine Learning*, Wiley 2018. The names are the book's; the underlying concern is the one formalised above |
| Data-related failure cascades dominate high-stakes AI deployment problems | **Verified, qualitative** | Sambasivan et al., CHI 2021. Qualitative research on failure patterns — *not* a quantified attribution of accuracy gains |
| A handful of classifier families dominate real-world tabular problems | **Verified** | Fernández-Delgado et al., *JMLR* 15(90):3133–3181, 2014; Caruana & Niculescu-Mizil, ICML 2006 |
| US model risk management guidance was replaced in April 2026 | **Verified** | OCC Bulletin 2026-13, "Model Risk Management: Revised Guidance", 17 April 2026, issued jointly by the OCC, Federal Reserve Board and FDIC; rescinds OCC 2011-12, 1997-24, 2021-19 and the MRM booklet; generative and agentic AI are out of scope; explicitly not enforceable standards; most relevant above $30bn in assets. Read on occ.gov |
| The CFPB's 2023 adverse-action circular is no longer current | **Verified** | Published at 89 FR 27361 (17 Apr 2024); withdrawn by "Interpretive Rules, Policy Statements, and Advisory Opinions; Withdrawal", 90 FR 20084 (12 May 2025). Verified in the Federal Register |
| Creditworthiness assessment is high-risk under the EU AI Act | **Verified** | Regulation (EU) 2024/1689, Article 6(2) with Annex III(5)(b), which carves out fraud detection |
| A right against solely automated decisions with significant effects | **Verified** | GDPR (Regulation (EU) 2016/679) Article 22 |
| "Most gains come from data work" / the "80% of the value" figure | **Flagged — folklore** | No study supporting a quantified attribution was found. Item 8 below |
| The offline-to-online gap as a measured effect | **Flagged — practitioner claim** | No study located. Item 7 below |
| Data-centric AI as a research finding | **Flagged — practitioner campaign** | No canonical written source located. Item 6 below |
| MAS FEAT: the document title, year and the Veritas assessment-methodology naming | **Flagged — primary source unreachable** | Item 1 below |
| A two-part paper titled "Classifier Calibration" attributed to Cohen & Goldszmidt | **Rejected** | No such paper located; the canonical calibration lineage does not run through it. Item 4 below |
| "…as good as it gets" as a calibration-paper title | **Rejected** | Not located. Item 5 below |
| General benchmark values ("good AUC", "acceptable PSI", "sufficient lift") | **Rejected** | No verifiable general values exist; the correct bar is problem-specific, so none are given here. Item 11 |

### 18.2 What Could Not Be Verified

This guide cites only what it could verify, and flags the rest. The following are treated above as practitioner claims or partial findings, never as established results:

1. **MAS FEAT — document title, publication year and the Veritas assessment-methodology naming.** The Monetary Authority of Singapore's site returned a maintenance page to automated retrieval, so neither the FEAT guidance document nor the Veritas assessment methodology could be confirmed from the primary source. The acronym expansion (Fairness, Ethics, Accountability, Transparency) and the existence of a MAS FEAT initiative for the financial sector are widely reported but were not verified here. Verify against mas.gov.sg before citing.
2. **The Federal Reserve's own SR letter number for the 2026 interagency model risk management guidance.** OCC Bulletin 2026-13 was verified in full, including its rescissions. The companion Federal Reserve letter number could not be verified. Note also that SR 11-7 (4 April 2011) is no longer the sole current reference: the 2011 guidance has been replaced.
3. **Platt scaling — the 1999 book chapter's text and page numbers.** The chapter ("Probabilistic Outputs for Support Vector Machines and Comparisons to Regularized Likelihood Methods", *Advances in Large Margin Classifiers*, MIT Press, 1999) could not be retrieved; the available PDF extracted as garbled binary. The method and its attribution are corroborated by a peer-reviewed follow-up that names it (Lin, Lin & Weng, *Machine Learning* 68(3):267–276, 2007).
4. **A two-part paper titled "Classifier Calibration", sometimes attributed to Cohen & Goldszmidt.** No such paper could be located. The verified calibration lineage is Platt (1999), Zadrozny & Elkan (2002), Niculescu-Mizil & Caruana (2005) and Guo et al. (2017). Treat citations to a "Classifier Calibration" two-part paper as unverified.
5. **"…as good as it gets" as a calibration-paper title.** Could not be located.
6. **Andrew Ng's data-centric AI argument as a peer-reviewed source.** No primary paper was verified. It is attributed here as a practitioner campaign (talks, essays and a competition associated with Landing AI and DeepLearning.AI, c. 2021), and §11.5 treats it as a heuristic rather than a law.
7. **The offline-to-online performance gap as a measured result.** No published study quantifying the gap in general terms was verified. §13.5 attributes it as a practitioner observation, with the verified mechanisms (feedback loops, undeclared consumers) cited separately.
8. **The "most gains come from data work" claim and the "80% of the value" style figure.** No study supporting a quantified attribution was found. The closest documented support is qualitative — Sambasivan et al., CHI 2021 — and the percentage figure is folklore, flagged as such in §4.4.
9. **The imprint year of *Dataset Shift in Machine Learning*.** Crossref registers the DOI under 2008 and Open Library returns both 2008 and 2022 records, while the volume is commonly cited as MIT Press, 2009. The edition year is therefore imprecise here.
10. **Demšar (2006) page range.** The paper ("Statistical Comparisons of Classifiers over Multiple Data Sets", *JMLR* 7, 2006) was verified by title, author, venue and year on the JMLR site; the page range was not confirmed and is omitted rather than guessed.
11. **Ideal or benchmark values for metrics.** No "good AUC", "acceptable PSI" or "sufficient lift" thresholds are given in this guide, deliberately: none could be verified as general, and the correct bar is problem-specific.

Where a number in this guide is illustrative — the framing table in §2.1, the constraint examples in §14, and the whole of §15 — it is marked as coming from the fictional Cymbal Bank setting and is not a claim about any real institution.

---

## Glossary

| Term | Meaning in this guide |
|---|---|
| **Applied ML** | The practitioner discipline of getting a model to work on a real problem: framing, labels, features, honest evaluation, the iteration loop, and the good-enough decision |
| **Baseline** | The incumbent being replaced — the trivial, rule, or simple-model result any candidate must beat on the same split and metric |
| **Brier score / log loss** | Proper scoring rules that measure probability quality rather than ranking |
| **Calibration** | Agreement between predicted probability and observed frequency; required whenever the score is multiplied by money |
| **Concept drift** | A change in P(Y\|X) — the learned relationship itself stops holding |
| **Covariate shift** | A change in the input distribution P(X) with P(Y\|X) stable |
| **Censoring** | An outcome not yet observable because its window has not closed; excluded, never labelled negative |
| **Error analysis** | Reading the actual errors, slicing them, and forming one hypothesis at a time — the main source of real gains |
| **Grouped / entity leakage** | The same entity appearing in training and evaluation, so the model recognises the entity rather than the signal |
| **Label leakage** | Information about the target available in the features that would not be legitimately available at the prediction moment |
| **Learning task** | The formal framing: unit of observation, prediction moment, target, and the decision the output feeds |
| **Prediction moment** | The instant at which scoring happens; it fixes which features are legitimately available |
| **Precision / recall** | Of the flagged, how many are true; of the true, how many are flagged |
| **PR-AUC** | Precision–recall area, informative when positives are rare; not comparable across different base rates |
| **Purging / embargo** | Removing training observations whose label windows overlap the test period, plus a buffer after it |
| **Reason code** | The individual-level attribution given to a declined applicant; a functional modelling requirement in regulated credit |
| **Reject inference** | The assumption that the relationship learned on approved applicants extends to rejected ones |
| **Refinement vs calibration** | How informative the score distribution is, versus whether its magnitudes are right |
| **Selection optimism** | The upward bias in a metric caused by choosing the best of many configurations on the same data |
| **Training-serving skew** | The difference between a feature's value in training and at serving time |
| **Uplift / incremental response** | The causal effect of an action on an outcome, as opposed to the outcome probability |

---

## References and Further Reading

Primary and canonical sources cited above, with the venue as verified:

1. **Kaufman, S., Rosset, S., Perlich, C.** — "Leakage in Data Mining: Formulation, Detection, and Avoidance", *KDD 2011*, pp. 556–563, DOI 10.1145/2020408.2020496
2. **Kaufman, S., Rosset, S., Perlich, C., Stitelman, O.** — same title, *ACM Transactions on Knowledge Discovery from Data* 6(4), 2012, pp. 1–21, DOI 10.1145/2382577.2382579
3. **Platt, J.** — "Probabilistic Outputs for Support Vector Machines and Comparisons to Regularized Likelihood Methods", in *Advances in Large Margin Classifiers*, MIT Press, 1999 *(partially verified — §18.2, item 3)*
4. **Lin, H.-T., Lin, C.-J., Weng, R. C.** — "A note on Platt's probabilistic outputs for support vector machines", *Machine Learning* 68(3):267–276, 2007, DOI 10.1007/s10994-007-5018-6
5. **Zadrozny, B., Elkan, C.** — "Transforming classifier scores into accurate multiclass probability estimates", *KDD 2002*, pp. 694–699, DOI 10.1145/775047.775151
6. **Niculescu-Mizil, A., Caruana, R.** — "Predicting good probabilities with supervised learning", *ICML 2005*, pp. 625–632, DOI 10.1145/1102351.1102430
7. **Guo, C., Pleiss, G., Sun, Y., Weinberger, K. Q.** — "On Calibration of Modern Neural Networks", *ICML 2017*, PMLR 70:1321–1330
8. **van den Goorbergh, R., van Smeden, M., Timmerman, D., Van Calster, B.** — "The harm of class imbalance corrections for risk prediction models: illustration and simulation using logistic regression", *JAMIA* 29(9):1525–1534, 2022, DOI 10.1093/jamia/ocac093
9. **Chawla, N. V., Bowyer, K. W., Hall, L. O., Kegelmeyer, W. P.** — "SMOTE: Synthetic Minority Over-sampling Technique", *JAIR* 16:321–357, 2002, DOI 10.1613/jair.953
10. **Roberts, D. R., Bahn, V., Ciuti, S., Boyce, M. S., Elith, J., et al.** — "Cross-validation strategies for data with temporal, spatial, hierarchical, or phylogenetic structure", *Ecography* 40(8):913–929, 2017, DOI 10.1111/ecog.02881
11. **Varma, S., Simon, R.** — "Bias in error estimation when using cross-validation for model selection", *BMC Bioinformatics* 7:91, 2006, DOI 10.1186/1471-2105-7-91
12. **Demšar, J.** — "Statistical Comparisons of Classifiers over Multiple Data Sets", *JMLR* 7, 2006 *(page range unverified — §18.2, item 10)*
13. **Gama, J., Žliobaitė, I., Bifet, A., Pechenizkiy, M., Bouchachia, A.** — "A survey on concept drift adaptation", *ACM Computing Surveys* 46(4), 2014, DOI 10.1145/2523813
14. **Moreno-Torres, J. G., Raeder, T., Alaiz-Rodríguez, R., Chawla, N. V., Herrera, F.** — "A unifying view on dataset shift in classification", *Pattern Recognition* 45(1):521–530, 2012, DOI 10.1016/j.patcog.2011.06.019
15. **Quiñonero-Candela, J., Sugiyama, M., Schwaighofer, A., Lawrence, N. D. (eds.)** — *Dataset Shift in Machine Learning*, MIT Press *(imprint year stated imprecisely — §18.2, item 9)*
16. **Wolpert, D. H.** — "The Lack of A Priori Distinctions Between Learning Algorithms", *Neural Computation* 8(7):1341–1390, 1996, DOI 10.1162/neco.1996.8.7.1341
17. **Fernández-Delgado, M., Cernadas, E., Barro, S., Amorim, D.** — "Do we Need Hundreds of Classifiers to Solve Real World Classification Problems?", *JMLR* 15(90):3133–3181, 2014
18. **Caruana, R., Niculescu-Mizil, A.** — "An empirical comparison of supervised learning algorithms", *ICML 2006*, pp. 161–168, DOI 10.1145/1143844.1143865
19. **Breiman, L.** — "Statistical Modeling: The Two Cultures", *Statistical Science* 16(3), 2001, DOI 10.1214/ss/1009213726
20. **Shmueli, G.** — "To Explain or To Predict?", *Statistical Science* 25(3), 2010 *(SSRN preprint record dated 2009)*
21. **Sculley, D., Holt, G., Golovin, D., Davydov, E., Phillips, T., et al.** — "Hidden Technical Debt in Machine Learning Systems", *NeurIPS 2015*
22. **Sambasivan, N., Kapania, S., Highfill, H., Akrong, D., et al.** — "'Everyone wants to do the model work, not the data work': Data Cascades in High-Stakes AI", *CHI 2021*, pp. 1–15, DOI 10.1145/3411764.3445518
23. **Rudin, C.** — "Stop explaining black box machine learning models for high stakes decisions and use interpretable models instead", *Nature Machine Intelligence* 1(5):206–215, 2019, DOI 10.1038/s42256-019-0048-x
24. **Lundberg, S. M., Lee, S.-I.** — "A Unified Approach to Interpreting Model Predictions", *NeurIPS 2017*
25. **Mitchell, M., Wu, S., Zaldivar, A., Barnes, P., et al.** — "Model Cards for Model Reporting", *FAT\* 2019*, pp. 220–229, DOI 10.1145/3287560.3287596
26. **Gebru, T., Morgenstern, J., Vecchione, B., Vaughan, J. W., et al.** — "Datasheets for Datasets", 2018; *Communications of the ACM* 64(12), 2021
27. **Breck, E., Cai, S., Nielsen, E., Salib, M., Sculley, D.** — "The ML Test Score: A Rubric for ML Production Readiness and Technical Debt Reduction", *IEEE Big Data 2017*, DOI 10.1109/bigdata.2017.8258038
28. **López de Prado, M.** — *Advances in Financial Machine Learning*, Wiley, 2018 (purged k-fold, purging and embargo)
29. **Basel Committee on Banking Supervision** — "Studies on the Validation of Internal Rating Systems (revised)", *Working Paper No. 14*, February 2005 (revised May 2005)
30. **Bank regulation (US)** — Regulation B, 12 CFR § 1002.9 (adverse-action notification); CFPB Circular 2023-03, 89 FR 27361 (Apr 17, 2024); CFPB withdrawal of interpretive rules, policy statements and advisory opinions, 90 FR 20084 (May 12, 2025); OCC Bulletin 2026-13, "Model Risk Management: Revised Guidance" (Apr 17, 2026); Federal Reserve/OCC SR 11-7 and OCC Bulletin 2011-12 (2011)
31. **EU** — Regulation (EU) 2024/1689 (AI Act), Article 6(2) and Annex III(5)(b); GDPR Article 22
32. **Monetary Authority of Singapore** — FEAT principles and the Veritas initiative *(not verified from the primary source — §18.2, item 1)*
33. **Cross-references in this repository** — [MLOps Lifecycle Frameworks](mlops_lifecycle_frameworks_guide.md) §4, §12 · [Feature Stores](feature_store_guide.md) §5, §6, §7, §14 · [ML Platforms Comparison](ml_platforms_comparison_guide.md) · [Mathematics for ML and DS](mathematics_for_ml_and_ds_study_notes.md) · [Adversarial ML Attacks](adversarial_ml_attacks_guide.md) · [ML System Design Interviews](ml_system_design_interview_guide.md)

---

> **Disclaimer:** This guide is for educational reference and describes modelling practice as of September 2026. Regulatory statements are attributed to their sources and several are explicitly flagged as unverified in §18; the supervisory position can change at short notice — the US model risk management guidance was replaced in April 2026 and the CFPB withdrew its 2023 adverse-action circular in May 2025, both within the period covered here. Cymbal Bank is a fictional illustration. Consult your model risk management, legal and compliance functions before relying on any regime statement in this document. Nothing here is legal advice, and no benchmark, threshold or percentage should be adopted without re-deriving it on your own data.

Judge the model by the decision it improves, the evidence it can defend, and the assumptions it declares — and always establish the baseline first.
