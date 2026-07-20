# Full MLOps Lifecycle Frameworks — A Comprehensive Guide

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Crédit Agricole CIB  
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Companion Guides:** [Enterprise AI Platforms](enterprise_ai_platforms_guide.md) · [AI Adoption Strategies](ai_adoption_strategies_guide.md)  
> **Last Updated:** July 2026

---

## Table of Contents

1. [What Is MLOps?](#1-what-is-mlops)
2. [Why MLOps Matters](#2-why-mlops-matters)
3. [MLOps Maturity Model](#3-mlops-maturity-model)
4. [The ML Lifecycle Stages in Detail](#4-the-ml-lifecycle-stages-in-detail)
5. [CI/CD for ML — The Three Pipelines](#5-cicd-for-ml--the-three-pipelines)
6. [MLOps Frameworks and Tools Comparison](#6-mlops-frameworks-and-tools-comparison)
7. [MLOps Architecture Patterns](#7-mlops-architecture-patterns)
8. [MLOps for Generative AI / LLMOps](#8-mlops-for-generative-ai--llmops)
9. [Governance and Compliance in MLOps](#9-governance-and-compliance-in-mlops)
10. [MLOps in Banking and Finance](#10-mlops-in-banking-and-finance)
11. [Implementation Roadmap](#11-implementation-roadmap)
12. [Common Challenges and Failure Modes](#12-common-challenges-and-failure-modes)
13. [Best Practices](#13-best-practices)
14. [Comparison Matrix — Tools Across Lifecycle Stages](#14-comparison-matrix--tools-across-lifecycle-stages)
15. [Visual MLOps Lifecycle](#15-visual-mlops-lifecycle)

---

## 1. What Is MLOps?

**MLOps** (Machine Learning Operations) is the set of practices, processes, and tools that operationalise ML models from ideation through production to retirement. It applies DevOps principles to ML while incorporating data engineering and ML-specific testing, monitoring, and governance.

### 1.1 The Three Disciplines

| Discipline | Role in MLOps |
|---|---|
| **ML / Data Science** | Model development, feature engineering, experimentation, evaluation |
| **DevOps** | CI/CD pipelines, infrastructure as code, containerisation, orchestration |
| **Data Engineering** | Data pipelines, feature stores, data validation, lineage, quality |

### 1.2 Core Principles

- **Repeatability** — Every run reproducible from versioned code, data, and environment.
- **Automation** — Manual handoffs replaced by automated training, testing, deployment pipelines.
- **Continuous Delivery** — Models treated as software artifacts, deployed through staged pipelines.
- **Observability** — Models monitored for drift, degradation, and operational health.
- **Governance** — Every version, deployment, and approval tracked with audit trails.
- **Collaboration** — Shared infrastructure and code practices across data science, engineering, platform teams.

### 1.3 CI/CD/CT for ML

MLOps extends DevOps with a third pillar — Continuous Training:

- **CI (Continuous Integration)** — Validates code, data schemas, and training pipelines. Runs unit tests on data code, integration tests on pipelines, trains a candidate model on a data slice.
- **CD (Continuous Delivery/Deployment)** — Automates model promotion through staging environments using shadow, canary, or blue-green strategies.
- **CT (Continuous Training)** — Automatically retriggers training pipelines on new data, drift detection, or schedule. The ML-specific feedback loop.

### 1.4 MLOps vs. DevOps — Key Differences

| Aspect | DevOps | MLOps |
|---|---|---|
| **Artifact** | Code + config | Code + data + model + hyperparams + environment |
| **Testing** | Unit, integration, E2E | Plus data validation, model eval, bias tests, drift tests |
| **Versioning** | Git | Data versioning (DVC) + model registry + Conda/Docker |
| **Deployment** | Deploy code to servers | Deploy model to batch/real-time/edge endpoints |
| **Monitoring** | CPU, mem, latency | Plus drift (data/concept/feature), model accuracy, bias |
| **Rollback** | Revert commit | Revert model version + data snapshot |

---

## 2. Why MLOps Matters

### 2.1 The Reproducibility Crisis
Without MLOps, training runs use ad hoc data snapshots, notebook environments vary, and results cannot be reproduced. Studies show the majority of ML research and many production models cannot be replicated.

### 2.2 Model Decay (Concept + Data Drift)
Models degrade over time. A fraud model trained on 2023 patterns fails on 2026 fraud. Accuracy drops 2–5%/month in volatile domains (credit risk, fraud, demand forecasting). Without monitoring, decay goes undetected until business impact occurs.

### 2.3 Deployment Friction
Industry surveys find 50–90% of ML models never reach production. Root causes: Jupyter-to-production handoff gaps, no standardised deployment, incomplete documentation, inability to serve at required latency.

### 2.4 Governance and Regulatory Scrutiny

| Regulation | Jurisdiction | Key Requirements |
|---|---|---|
| **MAS FEAT** | Singapore | Fairness, Ethics, Accountability, Transparency for AI in finance |
| **EU AI Act** | EU (2024–2027) | Risk-based categories: unacceptable (prohibited), high-risk (risk mgmt, data gov, transparency, human oversight), limited-risk, minimal-risk |
| **BCBS 239** | International | Risk data aggregation, data lineage, data quality controls, auditability |
| **SR 11-7 / OCC 2011-12** | US Federal Reserve | Model risk management: independent validation, ongoing monitoring, documentation |
| **ECAF** | EU Banking | Explainable credit decisions — adverse action reason codes required |

### 2.5 Operational Benefits
Faster time-to-market (months → days), higher model velocity, reduced operational risk, cost efficiency (right-sized infra, no idle GPU), audit readiness.

---

## 3. MLOps Maturity Model

### Level 0 — Manual (No MLOps)
- Jupyter notebooks, no experiment tracking, no data versioning.
- Model saved as .pkl, deployed by email to engineering.
- No monitoring.
- **Risk:** High — models unreproducible, knowledge siloed, regulatory non-compliance.

### Level 1 — Pipeline Automation (CI for Training)
- Experiment tracking (MLflow, W&B), training code versioned, data versioned (DVC).
- Automated training pipeline, data validation checks, model registry.
- **But:** Deployment is manual, no monitoring, no retraining.
- **Risk:** Medium — reproducible but deployment friction remains.

### Level 2 — Automated Deployment (CD for Models)
- Automated CD pipelines (shadow/canary/blue-green), standardised serving infra (KServe, Seldon).
- Basic monitoring (latency, throughput, accuracy dashboards).
- Rollback automated.
- **But:** No automated retraining, governance ad hoc.
- **Risk:** Low-Medium — fast deployment but manual upkeep.

### Level 3 — Full Automated MLOps
- Continuous training (CT) auto-triggered by drift, schedule, or new data.
- Zero-touch model promotion through automated gates.
- Full observability: drift, bias, data quality, system metrics.
- Governance embedded: model cards auto-generated, audit trails complete, approval gates enforced.
- **Risk:** Low — full lifecycle automation with governance-by-design.

| Level | Exp Tracking | CI Training | CD Models | CT Auto-Retrain | Monitoring | Governance |
|---|---|---|---|---|---|---|
| **0** | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **1** | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| **2** | ✅ | ✅ | ✅ | ❌ | Basic | ❌ |
| **3** | ✅ | ✅ | ✅ | ✅ | Full | ✅ |

---

## 4. The ML Lifecycle Stages in Detail

### 4.1 Business Understanding & Problem Framing

Define business problem, assess feasibility, set measurable success criteria before coding.

| Activity | Output |
|---|---|
| **KPI Definition** | Business metric the model impacts (approval rate, fraud loss, churn) |
| **Success Criteria** | Minimum precision/recall/business-impact threshold |
| **Feasibility Assessment** | Does data exist? Is ML the right solution? |
| **Risk Tiering** | High/Medium/Low model risk classification |
| **Problem Framing** | Translate business → ML problem (classification, regression, ranking, etc.) |
| **Stakeholder Alignment** | Signed-off scope from business, DS, engineering, risk, compliance |

**Key feasibility questions:** Is problem well-defined? Is sufficient labelled data available? Can we measure model impact on KPI in a closed loop? What is the cost of false positive vs. false negative?

### 4.2 Data Acquisition & Exploration

| Activity | Output |
|---|---|
| **Data Source Catalogue** | Internal + external sources (DBs, APIs, data lakes, vendors) |
| **Data Contracts** | Schema, freshness, SLAs, ownership, semantics between producers and consumers |
| **EDA** | Distributions, correlations, missing values, outliers |
| **Data Quality Assessment** | Completeness, accuracy, consistency, timeliness, validity, uniqueness |
| **Data Lineage** | Full path from source → model input, including transforms and joins |
| **Sensitivity Scan** | PII, financial data, regulated data identification |

### 4.3 Data Preparation & Feature Engineering

| Activity | Description |
|---|---|
| **Feature Pipelines** | Automated, repeatable pipelines — same logic for training and inference (prevents training-serving skew) |
| **Feature Store** | Centralised curated features with online + offline serving, point-in-time correctness, sharing across teams |
| **Data Validation** | Schema checks, distributional tests, anomaly detection (Great Expectations, TFDV) |
| **Transformation Pipelines** | Scaling, encoding, imputation — reusable components (Spark, dbt, TF Transform) |
| **Feature Versioning** | Each feature computation versioned alongside data and model versions |

### 4.4 Model Development & Experimentation

| Activity | Description |
|---|---|
| **Experiment Tracking** | Log all runs: hyperparams, metrics, data snapshot, code commit, environment |
| **Hyperparameter Tuning** | Systematic search (grid, random, Bayesian, PBT) |
| **Model Versioning** | Model registry with unique version, lineage, metadata |
| **Code Review for ML** | Check for data leakage, correct train/test split, pipeline correctness, metric validity |
| **Reproducibility** | Full environment spec (Docker + pinned deps + data hash) |
| **Baseline Model** | Simple heuristic/classical model as minimum performance bar |

**Reproducibility checklist:** Code commit hash ✅ · Data snapshot hash ✅ · Environment definition ✅ · Hyperparameters logged ✅ · Random seed fixed and logged ✅ · Training script (not notebook) ✅ · Evaluation script ✅

### 4.5 Model Evaluation & Validation

| Activity | Description |
|---|---|
| **Offline Metrics** | Accuracy, P/R/F1, AUC-ROC, RMSE/MAE, log loss, MAP@K, NDCG |
| **Cross-Validation** | K-fold, time-series (purging/embargo), grouped CV (prevents data leakage) |
| **Backtesting** | Historical simulation as if deployed at that time — critical for credit, trading, fraud |
| **Fairness/Bias Testing** | Demographic parity, equal opportunity, equalised odds across protected groups |
| **Explainability** | SHAP, LIME, partial dependence, counterfactual explanations |
| **Robustness** | Adversarial testing, edge cases, OOD stress tests |
| **Model Cards** | Standardised doc: intended use, performance (by slice), fairness, caveats |

**Caveats:** Offline metrics ≠ online performance. Imbalanced datasets need precision-recall curves. Time-series requires temporal CV. Evaluation uses same feature pipeline as production.

### 4.6 Model Deployment

| Aspect | Options |
|---|---|
| **Strategy** | Shadow (replay traffic, no user impact) → Canary (5–10% traffic) → Blue-Green (instant switch) → A/B Testing (controlled split) |
| **Serving** | Batch (scheduled scoring), Real-time (API endpoint, <100ms), Streaming (Kafka + Flink + model server) |
| **Optimisation** | Quantisation (FP16, INT8), pruning, distillation, ONNX, TensorRT |
| **Containerisation** | Model + deps + serving framework → Docker image |
| **Infra** | GPU/CPU nodes, auto-scaling, load balancing, API gateway (Terraform, K8s) |
| **Approval Gates** | Sign-off from DS, risk, compliance, platform before production |

| Strategy | Risk | Traffic | Rollback |
|---|---|---|---|
| Shadow | None | 0% users | Instant |
| Canary | Low | 5–10% → gradual | Near-instant |
| Blue-Green | Low | 100% switch | Instant (DNS/LB) |
| A/B Testing | Low | Controlled split | Instant |
| Big Bang | High | 100% immediately | Slow |

### 4.7 Model Monitoring & Observability

| Metric | What It Detects | Method |
|---|---|---|
| **Prediction Drift** | Shift in output distribution | PSI, K-S test |
| **Feature Drift** | Per-feature distribution shift | PSI, JSD, Wasserstein distance |
| **Concept Drift** | Features→target relationship changes | Requires ground truth labels |
| **Data Quality** | Missing values, schema violations, volume anomalies | Great Expectations, whylogs |
| **Performance Degradation** | Actual accuracy when ground truth arrives | Online metrics (lagged) |
| **Bias Drift** | Fairness degrades over time | Fairness metric monitoring |

**Drift thresholds:** PSI < 0.1 (none), 0.1–0.25 (moderate), > 0.25 (severe). K-S p < 0.05 = significant drift.

**Ground truth latency challenge:** Labels often arrive days/weeks later (credit risk, fraud). Threshold-based drift on predictions serves as a leading indicator while awaiting ground truth.

### 4.8 Model Governance & Compliance

| Activity | Description |
|---|---|
| **Model Risk Tiering** | High/Medium/Low based on materiality, regulatory impact, complexity |
| **Approval Gates** | Dev review → Independent validation → Business approval → Deploy sign-off |
| **Documentation** | Model cards, datasheets, technical spec, validation report |
| **Audit Trails** | Immutable log of all versions, deployments, retraining events, approvals |
| **Regulatory Reporting** | Automated reports on usage, performance, bias, changes |
| **Bias Monitoring** | Ongoing fairness metrics across model lifecycle, quarterly reports |
| **Model Retirement** | Formal decommission: notify stakeholders, archive artifacts, document retirement |

**Risk tiers for financial institutions:**

| Tier | Examples | Validation |
|---|---|---|
| **High** | Credit scoring, capital calculation, AML | Independent validation, quarterly monitoring |
| **Medium** | Churn, propensity, operational risk | Self-validation + peer review, semi-annual |
| **Low** | Recommendations, internal dashboards | Light docs, annual monitoring |

### 4.9 Model Retraining & Lifecycle Management

| Activity | Description |
|---|---|
| **Retraining Triggers** | Schedule, drift threshold exceeded, new data available, performance degradation |
| **Data Refresh** | Updated training dataset, point-in-time correct, no lookahead bias |
| **Continuous Training Pipeline** | Data validation → features → training → eval → comparison → conditional deploy |
| **Model Comparison Gate** | Candidate must beat current production model by minimum lift |
| **Model Registry** | Source of truth for versions, metadata, lineage, deployment status |
| **Lifecycle Stages** | Experimentation → dev → staging → production → deprecated → retired |
| **Model Retirement** | Archive artifacts, remove serving, notify consumers, document successor |

**Retraining frequency:** Credit risk: monthly–quarterly · Fraud: weekly–daily · Recommendations: daily–real-time · NLP/LLMs: quarterly–yearly · CV: monthly–quarterly · Forecasting: weekly–monthly.

---

## 5. CI/CD for ML — The Three Pipelines

### 5.1 Code Pipeline (Traditional CI/CD)
Validates and builds ML application code. Triggers: PR, push to main. Steps: lint → unit tests → build Docker → push registry.

### 5.2 Model Pipeline (Training + Evaluation)
Trains, evaluates, registers models. Triggers: manual, schedule, CI completion. Steps: data validation → features → training → evaluation → model registry.

### 5.3 Data Pipeline (Feature Engineering + Validation)
Transforms raw data into validated features. Triggers: new data, schedule. Steps: ingestion → validation → transformation → feature store.

### 5.4 CI for Data + Model Training

```
[PR Opened] → [Data Contract Validation] → [Data Quality Tests]
→ [Training on Data Slice] → [Eval vs Baseline] → [Merge Decision]
```

Every PR changing training code or feature logic triggers: data contract checks, Great Expectations suite, mini-training on data slice, baseline comparison, ML code review (check for data leakage, correct splitting).

### 5.5 CD for Model Deployment

```
[Model Registry (staging)] → [Shadow Deploy] → [Eval vs Production]
→ [Canary (5%)] → [Auto-Rollback if Degraded] → [Full Production]
```

Design principle: deployment is a separate gate from training. A model can pass training but fail shadow because real-world data differs from test set.

### 5.6 CT — Continuous Training

```
[Trigger: Schedule / Drift / New Data] → [Data Refresh] → [Feature Recompute]
→ [Retrain] → [Eval vs Current Production Model]
→ [IF better → deploy via CD] / [IF worse → alert DS team]
```

The comparison gate is critical: never blindly deploy a retrained model. It must demonstrate statistically significant improvement on recent test data.

---

## 6. MLOps Frameworks and Tools Comparison

### 6.1 End-to-End ML Platforms

| Tool | Description | Strengths | Weaknesses | Best For |
|---|---|---|---|---|
| **Kubeflow** | Full ML platform on K8s: pipelines, notebooks, Katib (HP tuning), KServe (serving) | K8s-native, portable, full lifecycle | High complexity, K8s expertise required | K8s-native teams needing self-hosted ML platform |
| **MLflow** | Experiment tracking, model registry, deployment | Simple, broad integration, excellent registry | Not a full orchestration platform | Teams starting MLOps, tracking-first |
| **TFX** | Google's production ML pipelines: TFDV, Transform, TFMA, TF Serving | Battle-tested at Google, strong validation | TF-centric, steep learning curve | TF shops needing production-grade pipelines |
| **Metaflow** | Netflix/Outerbounds: human-centric ML orchestration | Excellent UX, Pythonic, versioned snapshots | AWS-centric | DS teams wanting ergonomic path to production |
| **SageMaker** | AWS-native full lifecycle: labeling→training→deployment→monitoring | Deep AWS integration, broadest feature set | AWS lock-in, expensive at scale | AWS-native orgs wanting managed platform |
| **Vertex AI** | GCP-native: AutoML, custom training, registry, endpoints | GCP integration, unified platform | GCP lock-in, less mature than SageMaker | GCP-native teams |
| **Azure ML** | Azure-native: designer, AutoML, pipelines, endpoints | Azure integration, strong enterprise features | Azure lock-in | Microsoft/Azure shops |
| **ZenML** | Open-source, pipeline-centric, stack abstraction | Framework-agnostic, clean API, infra abstraction | Smaller ecosystem | Teams wanting opinionated MLOps framework |
| **DVC** | Data version control, pipeline versioning, exp tracking | Git-native, lightweight, strong data versioning | No built-in deployment or monitoring | Git-native teams needing data versioning |

### 6.2 Experiment Tracking & Model Registry

- **Weights & Biases** — Beautiful UI, collaboration, sweeps (HPO), LLM monitoring. [Pricing at scale]
- **Comet** — Self-hosted option, strong monitoring. [Smaller community]
- **Neptune** — Excellent metadata organisation, flexible. [Smaller ecosystem]
- **MLflow Tracking** — Free, simple, broad integration. [Less polished UI]

### 6.3 Workflow Orchestration

- **Airflow** — Largest ecosystem, battle-tested. [Heavy, not ML-native, verbose DAG syntax]
- **Prefect** — Pythonic, event-driven, automatic retries, good UI. [Smaller ecosystem]
- **Dagster** — Asset-based (not task-centric), excellent lineage, rich UI. [Newer, steeper learning curve]
- **Flyte** — ML-native, strong typing, re-entrant checkpoints, K8s-native. [K8s-only, smaller community]

### 6.4 Model Serving

- **KServe** — K8s-native, auto-scaling, canary/blue-green, multi-framework.
- **Seldon Core** — K8s-native, built-in monitoring + explainability + drift detection.
- **TorchServe** — Optimised for PyTorch.
- **Triton Inference Server** — NVIDIA GPU optimisation, highest GPU throughput, model ensembles.
- **BentoML** — Python-native, cloud-agnostic, simple API.

### 6.5 Monitoring & Observability

- **Evidently AI** — Open-source drift/data quality/performance monitoring. Excellent visualisations, Great Expectations integration.
- **Arize AI** — Full ML observability: drift, performance, bias, LLM monitoring, explainability.
- **WhyLabs** — Open-source whylogs + managed dashboard, cost-effective.
- **Seldon Alibi Detect** — Drift and outlier detection library.

---

## 7. MLOps Architecture Patterns

### 7.1 The Three-Pillar Architecture

```
┌──────────────────────┐   ┌──────────────────────┐
│    Feature Store     │   │    Model Registry     │
│  (Feast / Tecton /   │   │  (MLflow / SageMaker /│
│   SageMaker FS)      │   │   W&B / Custom)       │
│  ─ Offline features  │   │  ─ Model versions     │
│  ─ Online features   │   │  ─ Metadata + lineage │
│  ─ Point-in-time     │   │  ─ Approval status    │
│  ─ Feature lineage   │   │  ─ Deployment history │
└──────────┬───────────┘   └──────────┬───────────┘
           │                          │
           └──────────┬───────────────┘
                      │
             ┌────────▼────────┐
             │  Serving Layer   │
             │  (KServe / Triton│
             │  / SageMaker /   │
             │  FastAPI)        │
             │  ─ Real-time API │
             │  ─ Batch Scorer  │
             │  ─ Streaming     │
             └─────────────────┘
```

### 7.2 Training (Offline) vs. Inference (Online) Pipeline

| Aspect | Training | Inference |
|---|---|---|
| **Frequency** | Hours–Days | Milliseconds–Seconds |
| **Data Volume** | TBs (full history) | KBs (single record) |
| **Infra** | GPU cluster, Spark | CPU/GPU API servers, auto-scaling |
| **Latency** | None | Strict SLA (e.g., <100ms) |
| **Orchestration** | Airflow, Kubeflow | K8s, serverless |

### 7.3 Batch Inference
`[Data Lake] → [Batch Feature Pipeline] → [Scorer] → [Predictions Table] → [CRM/Reports]`
Use cases: nightly churn scoring, weekly credit limits.

### 7.4 Real-Time Inference
`[Client] → [API Gateway] → [Model Server] → [Online Feature Store] → [Prediction]`
Use cases: fraud detection, real-time credit decisions, recommendation APIs.

### 7.5 Streaming Inference
`[Kafka/Kinesis] → [Flink/Spark Streaming] → [Model Server gRPC] → [Prediction Stream]`
Use cases: real-time fraud on transaction streams, ad bidding, IoT anomaly detection.

### 7.6 Edge Inference
`[Edge Device / Mobile / POS] → [On-Device Model (TFLite / CoreML / ONNX)] → [Sync to Cloud]`
Use cases: POS fraud, mobile keyboard, autonomous vehicles, industrial IoT.

### 7.7 Hybrid On-Prem / Cloud (Regulated Industries)

```
[On-Prem: Sensitive Data Lake] → [Feature Store (On-Prem)] → [Training (GPU)] → [Compliance Gate]
     │ Encrypted, Audited Transfer
[Cloud: Model Registry Replica] → [Model Serving (Auto-scale)] → [Monitoring]
```

Design principles: sensitive data never leaves on-prem · Feature store sync (on-prem writes, cloud shadow copy) · Model artifacts encrypted, transfer audited · Inference on-cloud (anonymised features) or on-prem (sensitive predictions).

---

## 8. MLOps for Generative AI / LLMOps

### 8.1 LLMOps Lifecycle

```
[Prompt Engineering] → [RAG Pipeline Dev] → [Fine-Tuning / RLHF]
→ [Evaluation] → [Guardrails] → [Deployment] → [Monitoring + Content Safety]
→ [Feedback Loop → Retrain / Refine Prompts]
```

| Stage | Key Tools |
|---|---|
| **Prompt Engineering** | LangChain, OpenAI Playground, Anthropic Console |
| **RAG Pipeline** | LangChain, LlamaIndex, Chroma, Pinecone, Weaviate |
| **Fine-Tuning / RLHF** | Axolotl, Unsloth, TRL |
| **Evaluation** | RAGAS, LangSmith, DeepEval, MLflow Evaluation |
| **Guardrails** | Guardrails AI, Nemo Guardrails |
| **Deployment** | vLLM, TGI, Triton, BentoML, SageMaker JumpStart |
| **Monitoring** | Arize AI, Helicone, W&B Prompts, LangFuse |
| **Content Safety** | Llama Guard, Azure AI Content Safety |

### 8.2 LLMOps vs. Traditional MLOps

| Aspect | Traditional MLOps | LLMOps |
|---|---|---|
| **Model** | Train or fine-tune open-source | Mostly API-based foundation models + prompt/RAG |
| **Evaluation** | Clear metrics (accuracy, F1) | Subjective: hallucination, helpfulness, safety |
| **Versioning** | Model artifacts | Prompts + model + RAG config |
| **Cost** | GPU hours | API tokens (input + output) |
| **Latency** | Typically <100ms | 200ms–10s (output length dependent) |
| **Privacy** | Training data stays on-prem | Prompt data leaves to API unless self-hosted |
| **Monitoring** | Drift, model decay | Hallucination rate, toxicity, token spend, latency |
| **Rollback** | Previous model version | Previous prompt / RAG config |

### 8.3 Key LLMOps Challenges
- **Hallucination** — Reference-free detection via LLM-as-judge, consistency checks, factual grounding.
- **Prompt versioning** — Minor wording changes dramatically alter outputs. Version prompts like code.
- **Cost management** — Every API call costs money. Track per-query spend.
- **Evaluation difficulty** — BLEU/ROUGE poorly correlate with human judgment. Need LLM-as-judge + human eval.
- **Model deprecation** — Foundation model providers deprecate versions; prompts may break.
- **Latency vs. quality** — Longer outputs = better quality but higher latency and cost.

---

## 9. Governance and Compliance in MLOps

### 9.1 Model Risk Management Frameworks

**SR 11-7 / OCC 2011-12** (US Federal Reserve — gold standard for MRM):
- Three Lines of Defence: Model Development (1st) → Independent Validation (2nd) → Internal Audit (3rd)
- Components: Governance policies · Dev/implementation/use · Independent validation · Ongoing monitoring · Documentation

**MAS FEAT** (Singapore): Fairness (no systematic bias) · Ethics (transparency about AI use) · Accountability (clear ownership) · Transparency (explainable decisions)

**EU AI Act** (EU 2024–2027):
- **Unacceptable Risk** — Social scoring, real-time biometric surveillance → Prohibited
- **High Risk** — Credit scoring, employment, essential services → Risk mgmt, data gov, transparency, human oversight, accuracy/robustness
- **Limited Risk** — Chatbots → Transparency obligation
- **Minimal Risk** — Spam filters → Voluntary codes

### 9.2 Model Documentation

| Artifact | Content | Origin |
|---|---|---|
| **Model Card** | Model details, intended use, performance by slice, fairness, caveats | Mitchell et al., 2019 |
| **Datasheet for Datasets** | Motivation, composition, collection, preprocessing, uses | Gebru et al., 2018 |
| **Technical Spec** | Architecture, hyperparameters, training procedure, eval methodology | Internal standard |
| **Validation Report** | Independent findings, limitations, recommendations, conditional approvals | MRM tier-based |
| **Bias Audit Report** | Fairness metrics across demographics, mitigations | FEAT / EU AI Act |

### 9.3 Model Approval Gates

```
[Dev Complete] → [Peer Review] → [Independent Validation] (high-risk)
→ [Business Owner Approval] → [Risk/Compliance Review]
→ [Model Registry → Staging] → [Shadow Period 2–4 weeks]
→ [Model Registry → Production] → [Ongoing Monitoring]
```

**By risk tier:** Low = peer review + business sign-off · Medium = peer review + independent validation + business sign-off · High = peer review + independent validation + risk/compliance + executive sign-off + shadow observation.

---

## 10. MLOps in Banking and Finance

### 10.1 Unique Constraints

1. **Mandatory validation before production** — High-risk models require formal independent validation before deployment (weeks–months).
2. **Full auditability** — Every decision, version change, and parameter update must be auditable.
3. **Explainability for credit decisions** — ECAF requires adverse action reason codes. Black-box models need surrogate explainers.
4. **Model risk tiers** — Different governance per tier: credit risk (full validation) vs. marketing propensity (peer review).
5. **BCBS 239 data lineage** — Every model input/output traceable to source.
6. **Legacy risk system integration** — Must integrate with Murex, Calypso, AXIOM, Moody's, SAS — often via batch CSV/JDBC.

### 10.2 Typical Banking MLOps Stack

```
┌────────────────────────────────────────────────────────────┐
│              Model Governance Dashboard (Custom)            │
│  Model inventory · Risk tiers · Approval status · Audit    │
└─────────────────────┬──────────────────────────────────────┘
                      │
┌─────────────────────▼──────────────────────────────────────┐
│          Orchestration (Airflow / Control-M)                │
│  Training DAGs · Feature Pipeline DAGs · Inference DAGs    │
└──────┬──────────────────────────────┬──────────────────────┘
       │                              │
┌──────▼─────────────┐   ┌───────────▼──────────────────────┐
│  Training Platform  │   │  Model Registry (Custom)         │
│  Kubeflow /         │   │  Versions + metadata             │
│  SageMaker (on-prem)│   │  Approval workflow + audit trail │
└──────┬─────────────┘   └───────────┬──────────────────────┘
       │                              │
┌──────▼──────────────────────────────▼──────────────────────┐
│       Feature Store (Feast / Custom) — Point-in-time corr. │
│       Online serving · Data lineage tracking               │
└─────────────────────────┬──────────────────────────────────┘
                          │
┌─────────────────────────▼──────────────────────────────────┐
│  Monitoring: Evidently AI / Seldon Alibi / Custom          │
│  Drift detection · Model performance · Bias · Alerting     │
└────────────────────────────────────────────────────────────┘
```

**Key characteristics:** Hybrid on-prem/cloud · Custom registry for banking approval workflows · Airflow/Control-M (ubiquitous in banking) · Batch-dominant inference · SAS model integration.

### 10.3 Lifecycle in Banking

**Pre-Production:** Model in secured sandbox → Peer review → Independent validation → Registry registration with risk tier → Approval chain (DS Lead → Model Risk Officer → Business → Compliance)

**Production:** Shadow/Canary/Batch deployment → Every prediction logged with feature values + model version → Weekly drift reports → Monthly performance vs. realised outcomes → Quarterly model risk review

**Retirement:** Successor validated → Retirement request with rationale → Downstream systems migrated → Artifact archived → Model marked "retired" in registry

---

## 11. Implementation Roadmap

### Phase 1 — Foundation (Months 1–3)
- Experiment tracking (all runs logged with params, metrics, artifacts)
- Model registry (source of truth for versions, metadata)
- Code + data + environment versioning (Git + DVC + Docker)
- Basic training pipeline (reproducible with one command)
- Data validation (Great Expectations on key sources)
- Model cards for all production models

**Exit:** Every experiment tracked. Every model in registry. One-command reproducible training.

### Phase 2 — Automation (Months 4–8)
- ML CI pipeline (PR triggers slice training, eval, blocks regression)
- ML CD pipeline (shadow → canary → production automated)
- Standardised inference endpoints with auto-scaling
- Basic monitoring (drift detection, prediction distribution, dashboards)
- Alerting (Slack/PagerDuty for drift and quality issues)

**Exit:** Code commit → production deployment with no manual steps. All production models monitored.

### Phase 3 — Governance (Months 9–12)
- Model risk tier classification for all models
- Formal approval workflows integrated into model registry
- Automated model documentation (model cards from pipeline metadata)
- Bias monitoring (fairness metrics on all production models)
- Audit trails (immutable lifecycle event log)
- Automated regulatory reporting

**Exit:** Full audit trail · Automated reports · Consistent risk tier enforcement.

### Phase 4 — Optimization (Months 13–18)
- Continuous Training (auto-retraining on drift/schedule/new data)
- Advanced observability (concept drift, online metrics, root cause analysis)
- Auto-rollback on performance degradation
- LLMOps platform (prompt management, RAG eval, guardrails)
- Self-service MLOps (DS deploy without engineering handoff)
- Cost optimisation (right-sized inference, model compression)

**Exit:** Zero-touch model lifecycle. Full observability. LLMs governed through LLMOps platform.

---

## 12. Common Challenges and Failure Modes

### 12.1 ML Not Treated as Engineering
**Symptom:** Notebook-only development, no version control, no testing.
**Impact:** Models work on laptop but fail in production. Unreproducible. Knowledge lost when DS leaves.
**Fix:** Enforce engineering practices: version control, code review, testing, reproducible environments.

### 12.2 Skipping Monitoring
**Symptom:** "Model is working fine" — but nobody actually measures.
**Impact:** Silent decay for weeks/months. Cumulative business damage.
**Fix:** Deploy monitoring with every model. Non-negotiable: drift + performance monitoring from Day 1.

### 12.3 Model Decay Undetected
**Symptom:** Monitoring exists but only for system metrics (CPU, latency).
**Impact:** Accuracy degrades gradually; poor decisions for weeks before anyone notices.
**Fix:** Implement both drift (leading indicator) and performance (lagging indicator) with threshold alerts.

### 12.4 Data Scientist vs. Engineer Divide
**Symptom:** "DS builds, engineers deploy" — manual handoff, incomplete docs, training-serving skew.
**Impact:** Deployed model ≠ built model. Finger-pointing.
**Fix:** Cross-functional teams. Shared pipeline ownership. Automated pipelines eliminate handoff.

### 12.5 Tool Overload
**Symptom:** 10+ tools (MLflow + W&B + DVC + Airflow + Feast +...) with no platform strategy.
**Impact:** Integration complexity > value. Teams maintain connectors instead of building models.
**Fix:** Start with 2–3 core tools. Add only when there's a clear gap. Prefer integrated platforms.

### 12.6 Compliance as Afterthought
**Symptom:** Governance requirements discovered after model is in production.
**Impact:** Retrospective documentation, emergency shutdowns, regulatory penalties.
**Fix:** Build compliance into pipelines from Day 1. Auto-generate documentation. Enforce approval gates.

### 12.7 Insufficient Production Testing
**Symptom:** Model passes offline evaluation but fails in production.
**Impact:** Surprise incidents. Stakeholder trust erosion.
**Fix:** Shadow deploy. Test with production data patterns. Stress test endpoints. Test latency under load.

### 12.8 Last-Mile Integration
**Symptom:** Model produces predictions but integrating into business workflows is manual.
**Impact:** Outputs sit in a database; business processes don't use them.
**Fix:** Design downstream integration before building the model. Build the pipeline from prediction → action.

### 12.9 Drift Type Confusion
**Symptom:** Team detects "drift" but doesn't distinguish feature vs. prediction vs. concept drift.
**Impact:** Wrong corrective action. Retraining when data quality fix was needed.
**Fix:** Educate team on drift types. Different detection + response strategies for each.

---

## 13. Best Practices

1. **Start with experiment tracking** — Highest-ROI MLOps investment. Log every run before any pipeline automation.

2. **Version everything:**
   - Code (Git) · Data (DVC/LakeFS) · Models (MLflow Registry) · Environment (Docker/Conda)
   - Every training run must be reproducible from a single commit + data hash.

3. **Automate testing early:**
   - **Data tests:** Schema, distributions, missing rates — run on every data pipeline.
   - **Model tests:** Eval on held-out set, baseline comparison, bias metrics — run on every training.
   - **Infra tests:** Endpoint health, latency SLA, GPU availability — run on every deployment.

4. **Monitor both data and model drift:**
   - Data drift (leading indicator — fires before performance degrades)
   - Concept drift (lagging indicator — fires after degradation)
   - Both are essential; data drift alone misses concept drift without distribution change.

5. **Stage deployments:** Shadow → Canary (5–10%) → Gradual (25%→50%→75%→100%) — each stage with auto-verification gates and automatic rollback.

6. **Document with model cards:** Include intended use, performance by slice, fairness eval, ethical considerations, maintenance schedule.

7. **Establish model risk tiers:** 3–4 tiers with clear criteria. Not every model needs full independent validation. Review tier assignments annually.

8. **Treat pipelines as code:** Version-controlled, testable, executable from CLI. Pipeline definitions = code files, not notebooks. Parameters = config files.

9. **Design for reproducibility:** Log commit hash + data hash + Docker tag + hyperparams + random seed. A run is reproducible if you can get identical results by restoring all five.

10. **Choose frameworks that integrate, not replace:** Prefer modular, open-API tools you can adopt incrementally. If a framework requires rewriting existing code to fit its model, it's too opinionated.

---

## 14. Comparison Matrix — Tools Across Lifecycle Stages

| Tool | Data Mgmt | Feature Eng | Exp Track | Model Registry | Training Pipeline | HP Tuning | Serving | Monitoring | Governance | LLMOps |
|---|---|---|---|---|---|---|---|---|---|---|
| **Kubeflow** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅(Katib) | ✅(KServe) | ❌ | ❌ | ❌ |
| **MLflow** | ❌ | ❌ | ★★★ | ★★★ | ❌ | ❌ | ✅Basic | ❌ | ❌ | ✅Eval |
| **TFX** | ✅(TFDV) | ✅(TFT) | ❌ | ❌ | ✅ | ❌ | ✅(TFS) | ✅(TFMA) | ❌ | ❌ |
| **Metaflow** | ✅Snap | ❌ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **SageMaker** | ✅FS | ✅Proc | ★★★ | ★★★ | ✅ | ✅Auto | ✅Inf | ✅Mon | ✅ | ✅JS |
| **Vertex AI** | ✅FS | ✅Pipe | ★★★ | ★★★ | ✅ | ✅Viz | ✅EP | ✅Mon | ❌ | ✅Gen |
| **Azure ML** | ✅FS | ✅Pipe | ★★★ | ★★★ | ✅ | ✅HD | ✅EP | ✅Drift | ✅ | ✅ |
| **ZenML** | ✅ | ✅ | ✅Int | ✅Int | ✅ | ❌ | ✅Int | ✅Int | ❌ | ❌ |
| **DVC** | ✅Ver | ✅Pipe | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **W&B** | ❌ | ❌ | ★★★ | ★★★ | ❌ | ✅Sweep | ❌ | ✅Prompts | ❌ | ✅ |
| **Comet** | ❌ | ❌ | ★★★ | ★★★ | ❌ | ✅ | ❌ | ✅ | ❌ | ✅ |
| **Neptune** | ❌ | ❌ | ★★★ | ★★★ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **Airflow** | ❌ | ✅Orc | ❌ | ❌ | ✅Orc | ❌ | ❌ | ❌ | ❌ | ❌ |
| **Prefect** | ❌ | ✅Orc | ❌ | ❌ | ✅Orc | ❌ | ❌ | ❌ | ❌ | ❌ |
| **Dagster** | ✅Asset | ✅Asset | ❌ | ❌ | ✅Orc | ❌ | ❌ | ❌ | ❌ | ❌ |
| **Flyte** | ❌ | ✅ | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **Evidently** | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ★★★ | ❌ | ❌ |
| **Arize AI** | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ★★★ | ❌ | ✅ |
| **Helicone** | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅LLM | ❌ | ✅ |
| **LangSmith** | ❌ | ❌ | ✅Prompts | ❌ | ❌ | ❌ | ❌ | ✅LLM | ❌ | ★★★ |

**Legend:** ✅ = Built-in · ★★★ = Best-in-class · ❌ = Not purpose-built · FS=Feature Store, Proc=Processing, Orc=Orchestration, Int=Integration, Snap=Snapshots, HD=HyperDrive, Viz=Vizier, TFS=TF Serving, JS=JumpStart, EP=Endpoints, Mon=Monitor

---

## 15. Visual MLOps Lifecycle

```
                         ┌─────────────────────┐
                         │  1. Business        │
                         │  Understanding &    │
                         │  Problem Framing    │
                         │  ─ KPIs / Success   │
                         └──────────┬──────────┘
                                    ▼
                         ┌─────────────────────┐
                         │  2. Data            │
                         │  Acquisition &      │
                         │  Exploration        │
                         │  ─ Contracts / EDA  │
                         └──────────┬──────────┘
                                    ▼
                         ┌─────────────────────┐
                         │  3. Data Prep &     │
                         │  Feature Engineering │
                         │  ─ Feature Store     │
                         │  ─ Data Validation   │
                         └──────────┬──────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              │                     │                     │
              ▼                     ▼                     ▼
    ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
    │ 4. Model         │ │ 5. Model         │ │ 6. Model         │
    │ Development &    │◄┤ Evaluation &     │◄┤ Deployment       │
    │ Experimentation  │ │ Validation       │ │ ─ Shadow/Canary  │
    │ ─ Exp Tracking   │ │ ─ Metrics/Bias   │ │ ─ Blue-Green/A/B │
    │ ─ HPO / Ver.     │ │ ─ Explainability │ │ ─ Serving (B/R/S)│
    └──────────────────┘ └──────────────────┘ └────────┬─────────┘
              │                     │                   │
              └─────────────────────┼───────────────────┘
                                    ▼
                         ┌─────────────────────┐
                         │  7. Model           │
                         │  Monitoring &       │
                         │  Observability      │
                         │  ─ Drift / Quality  │
                         │  ─ Performance      │
                         └──────────┬──────────┘
                                    │
        ┌───────────────────────────┼───────────────────────────┐
        │                           │                           │
        ▼                           ▼                           ▼
┌────────────────┐     ┌────────────────┐     ┌────────────────────┐
│ 8. Governance  │     │ 9. Model       │     │  CI / CD / CT      │
│ & Compliance   │◄────┤ Retraining &   │◄────┤  Pipelines         │
│ ─ MRM Tiers    │     │ Lifecycle Mgmt │     │                    │
│ ─ Approvals    │     │ ─ Auto-retrain │     │  ─ CI: Data + Code │
│ ─ Audit Trails │     │ ─ Model Reg.   │     │  ─ CD: Deploy      │
│ ─ Reports      │     │ ─ Retirement   │     │  ─ CT: Train       │
└────────────────┘     └────────────────┘     └────────────────────┘


            CROSS-CUTTING CAPABILITIES
 ┌──────────┬──────────┬────────────┬─────────────┐
 │ Feature  │  Model   │  Pipeline  │  Infra as   │
 │  Store   │ Registry │Orchestration│   Code      │
 ├──────────┼──────────┼────────────┼─────────────┤
 │ ML Ops   │ Data &   │ Security & │ Cost &     │
 │ Platform │ Model    │  Privacy   │ Resource   │
 │          │ Lineage  │            │ Management │
 └──────────┴──────────┴────────────┴─────────────┘


     STAKEHOLDERS                    REGULATIONS
 ┌─────────────────┐         ┌─────────────────────────┐
 │ Data Scientist  │         │ SR 11-7 (US) · MAS FEAT │
 │ ML Engineer     │         │ EU AI Act · BCBS 239    │
 │ Data Engineer   │         │ ECAF (EU) · PRA SS (UK)│
 │ Platform Eng.   │         └─────────────────────────┘
 │ Model Risk Offr │
 │ Compliance      │
 └─────────────────┘
```

---

## References

1. "MLOps: Continuous Delivery and Automation Pipelines in ML" — Google Cloud
2. "Hidden Technical Debt in ML Systems" — Sculley et al., NeurIPS 2015
3. "Model Cards for Model Reporting" — Mitchell et al., FAT* 2019
4. "Datasheets for Datasets" — Gebru et al., arXiv 2018
5. SR 11-7 — Federal Reserve / OCC, 2011
6. MAS FEAT Principles — Monetary Authority of Singapore, 2018
7. EU AI Act — European Parliament, 2024
8. BCBS 239 — Basel Committee, 2013
9. "The ML Test Score" — Breck et al., NeurIPS 2019
10. Kubeflow · MLflow · ZenML · Feast · Evidently AI · LangSmith docs

---

> **Disclaimer:** This guide is for educational reference. Regulatory interpretations vary by jurisdiction. Consult your model risk management and compliance teams for specific requirements. Tool capabilities as of July 2026.
