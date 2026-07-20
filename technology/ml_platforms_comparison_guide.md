# ML Platforms Comparison Guide: MLflow, Kubeflow, OpenShift AI & Alternatives

> **A comprehensive guide for enterprise ML platform selection**
>
> Part of the MLOps Series — Crédit Agricole CIB / Jack Liu Shurui
>
> **Last Updated:** July 2026

---

## Table of Contents

1. [The ML Platform Landscape](#1-the-ml-platform-landscape)
2. [Deep Dive: MLflow](#2-deep-dive-mlflow)
3. [Deep Dive: Kubeflow](#3-deep-dive-kubeflow)
4. [Deep Dive: OpenShift AI](#4-deep-dive-openshift-ai)
5. [Deep Dive: Amazon SageMaker](#5-deep-dive-amazon-sagemaker)
6. [Deep Dive: Vertex AI](#6-deep-dive-vertex-ai)
7. [Deep Dive: Azure Machine Learning](#7-deep-dive-azure-machine-learning)
8. [Alternatives & Ecosystem Tools](#8-alternatives--ecosystem-tools)
9. [Comparison Across Dimensions](#9-comparison-across-dimensions)
10. [Platform Selection Criteria](#10-platform-selection-criteria)
11. [Typical Platform Stacks](#11-typical-platform-stacks)
12. [Platform Comparison Matrix](#12-platform-comparison-matrix)
13. [Decision Tree & Selection Framework](#13-decision-tree--selection-framework)
14. [Recommendations for Crédit Agricole CIB](#14-recommendations-for-crédit-agricole-cib)
15. [References & Further Reading](#15-references--further-reading)

---

## 1. The ML Platform Landscape

### 1.1 No Single Tool Covers Everything

The ML platform ecosystem is fragmented by design. ML platforms operate at different layers of the machine learning lifecycle, and no single product covers all needs end-to-end. Most organizations — from startups to regulated banks — assemble a **stack of tools** rather than adopting a monolithic platform.

| Layer | Purpose | Example Tools |
|-------|---------|---------------|
| **Experiment Tracking** | Log parameters, metrics, artifacts during model development | MLflow Tracking, W&B, Neptune |
| **Pipeline Orchestration** | Define, schedule, and execute multi-step ML workflows | Kubeflow Pipelines, Dagster, Flyte, Airflow |
| **Model Registry** | Version, store, govern, and promote models | MLflow Model Registry, W&B Registry, Hugging Face Hub |
| **Model Serving** | Deploy models to production inference endpoints | KServe, Seldon Core, BentoML, vLLM |
| **Feature Store** | Centralize feature definitions, computation, and serving | Feast, Tecton, SageMaker Feature Store |
| **Full ML Platform** | Integrated lifecycle from data to monitoring | SageMaker, Vertex AI, Azure ML, OpenShift AI |
| **ML Governance** | Model risk management, audit, bias, explainability | Custom, SageMaker Clarify, Azure Responsible AI |

### 1.2 The Stack Reality

- **70%+ of enterprises** use 3+ ML tools in their stack (Algorithmia 2024 State of MLOps)
- **MLflow** is the most widely adopted experiment tracker (45%+ market share)
- **Kubeflow** dominates Kubernetes-native ML platforms
- **Cloud platforms** (SageMaker, Vertex AI, Azure ML) are preferred by cloud-native teams wanting managed services
- **OpenShift AI** is the enterprise choice for regulated industries already on Red Hat OpenShift

### 1.3 The Spectrum of Abstraction

Platforms exist on a spectrum from **lightweight tracking tools** to **full-stack managed platforms**:

```
Lightweight ──────────────────────────────────────────────────── Full-Stack
    │                       │                        │
  MLflow                  Kubeflow              SageMaker
  W&B                   OpenShift AI            Vertex AI
  Neptune              Charmed Kubeflow          Azure ML
```

Lower on the stack = more flexibility, less vendor lock-in, but more integration effort.
Higher on the stack = less ops overhead, tighter integration, but more lock-in and cost.

---

## 2. Deep Dive: MLflow

### 2.1 Overview

**MLflow** is the most widely adopted open-source ML experiment tracking and model registry tool, created by Databricks in 2018. **GitHub:** 20k+ stars | **License:** Apache 2.0

### 2.2 Core Components

| Component | Purpose |
|-----------|---------|
| **MLflow Tracking** | Log parameters, metrics, tags, and artifacts for ML runs. REST API + UI. |
| **MLflow Models** | Standard model packaging format (flavors for sklearn, PyTorch, TensorFlow, ONNX, etc.) |
| **MLflow Model Registry** | Centralized model store with versioning, stage transitions (Staging/Production/Archived) |
| **MLflow Pipelines** | Predefined templates for data ingestion, training, evaluation, deployment |
| **MLflow Deployments** | Serve models as REST endpoints (local, Docker, cloud plugins) |
| **MLflow Evaluate** | Built-in evaluation metrics including LLM quality/hallucination metrics |
| **MLflow AI Gateway** | Centralized endpoint management for LLM providers (OpenAI, Anthropic, etc.) |

### 2.3 LLM Support (MLflow 2.x+)

- **Prompt engineering UI** — version prompts alongside code
- **LLM Evaluation** — built-in metrics (toxicity, relevance, correctness, ROUGE, BLEU) with LLM-as-judge
- **LangChain integration** — auto-log LangChain runs, traces, and models
- **OpenAI integration** — track API calls with request/response logging
- **Hugging Face integration** — log and serve transformer models

### 2.4 Strengths

- **Lightweight** — `pip install mlflow`, minimal dependencies
- **Easy to integrate** — add 3 lines of code to existing training scripts
- **Broad framework support** — TensorFlow, PyTorch, scikit-learn, XGBoost, LightGBM, ONNX, spaCy, fastai
- **Open source (Apache 2.0)** — no vendor lock-in for the core
- **Huge community** — 20k+ GitHub stars, 1k+ contributors, extensive documentation
- **Self-hosted or managed** — run your own server or use Databricks managed MLflow
- **Model format standard** — MLflow Models format is becoming an industry standard

### 2.5 Weaknesses

- **Not a full platform** — no orchestration, serving infrastructure, monitoring, or governance
- **MLflow Pipelines are basic** — use Dagster/Flyte/Airflow for real orchestration
- **No serving infrastructure** — MLflow Deployments is a thin wrapper; need KServe/Seldon/vLLM
- **No governance** — no audit trails, approval workflows, model risk management
- **Model Registry is basic** — no automatic validation gates, no compliance reporting
- **No RBAC/multi-tenancy** — requires plugins or Databricks hosting
- **Scaling limits** — tracking server can be a bottleneck at high scale without Databricks

### 2.5 Banking-Specific Considerations

- MLflow Model Registry can be extended with **custom audit hooks** for regulatory compliance; for **MAS FEAT / SR 11-7** needs augmentation with external governance tools
- **FIPS compliance** depends on the hosting environment, not MLflow itself
- **Best for:** Experiment tracking + model registry as part of a larger MLOps stack; teams wanting lightweight tracking without platform lock-in

---

## 3. Deep Dive: Kubeflow

### 3.1 Overview

**Kubeflow** is an open-source ML platform on Kubernetes (CNCF project), providing a full lifecycle platform from notebooks to production serving. Originally developed by Google. **GitHub:** 14k+ stars | **License:** Apache 2.0

### 3.2 Core Components

| Component | Purpose |
|-----------|---------|
| **Kubeflow Pipelines (KFP)** | DAG-based ML workflow orchestration with drag-and-drop UI and Python DSL |
| **Kubeflow Notebooks** | Jupyter notebook servers on K8s with per-user isolation |
| **Katib** | Kubernetes-native hyperparameter tuning and early stopping |
| **KServe (formerly KFServing)** | Serverless model serving with autoscaling, canary deployments, GPU support |
| **Kubeflow Trainer** | Distributed training operator (PyTorch, TensorFlow, MPI, XGBoost) |
| **Central Dashboard** | Unified UI for managing all Kubeflow components |
| **Multi-Tenancy** | Per-user namespaces with isolation via Istio + KFAM |
| **Charmed Kubeflow** | Canonical's enterprise distribution with simplified deployment |

### 3.3 Key Features

- DAG-based pipelines with caching, retry, and parallel execution
- Reusable components shared across teams
- Distributed training — native operators for PyTorch DDP, Horovod, TF distribution
- AutoML with Katib (Bayesian optimization, random search, grid search, NAS)
- Serverless serving via KServe + Knative (scale-to-zero)
- Canary deployments with traffic splitting
- Model explainability with Alibi Explain integration
- Prometheus/Grafana monitoring integration

### 3.4 Strengths

- **Full lifecycle on Kubernetes** — one platform from notebooks to production
- **Portable across clouds** — works on EKS, GKE, AKS, OpenShift, on-prem
- **CNCF project** — strong upstream alignment, active community, Google backing
- **KServe** — best-in-class K8s-native model serving with autoscaling
- **Katib** — Kubernetes-native hyperparameter tuning
- **Extensible** — plug in any K8s-native tool
- **Charmed Kubeflow** (Canonical) — enterprise deployment, support, and upgrades

### 3.5 Weaknesses

- **Complex deployment** — notoriously difficult to install and maintain (improved with Charmed Kubeflow)
- **Requires K8s expertise** — need a strong platform/K8s team
- **Heavy footprint** — 20+ microservices, substantial resource requirements
- **Steep learning curve** — multiple concepts (Pipelines, Operators, Istio, Knative)
- **Jupyter-centric** — notebook-first experience may not suit all workflows
- **No built-in governance** — no MRM, audit trails, or compliance features out of the box
- **Community fragmentation** — multiple distributions (upstream, Charmed, Argo-based)

### 3.6 Banking-Specific Considerations

- Can run on OpenShift using ODF operator; **SCC and FIPS are not native** — requires OpenShift underneath
- Audit trails need custom integration; regulatory reporting needs additional tooling
- **Charmed Kubeflow** offers enterprise support for regulated environments
- **Best for:** Kubernetes-native teams wanting full ML platform; multi-cloud ML; organizations already on K8s

---

## 4. Deep Dive: OpenShift AI

### 4.1 Overview

**Red Hat OpenShift AI** (formerly OpenShift Data Science) is Red Hat's enterprise ML platform built on OpenShift. It provides a unified, supported platform for AI/ML and GenAI workloads with enterprise-grade security, governance, and hybrid cloud deployment. **Licensing:** Requires Red Hat OpenShift subscription + OpenShift AI add-on.

### 4.2 Architecture & Components

Built on the **Open Data Hub (ODH)** operator, integrating upstream open-source components into a supported enterprise product:

| Component | Role |
|-----------|------|
| **OpenShift Pipelines (Tekton)** | Pipeline orchestration for ML workflows |
| **KServe + ModelMesh** | Model serving with serverless (Knative) + multi-model serving |
| **vLLM** | High-throughput LLM serving engine |
| **Workbench Notebooks** | Jupyter + RStudio + VS Code workbenches with GPU support |
| **Kueue** | Batch job scheduling and resource quota management |
| **Distributed Workloads** | CodeFlare + Ray for distributed training |
| **TrustyAI** | Model explainability, bias detection, and monitoring |
| **Service Mesh (Istio)** | Traffic management, security, observability |
| **Monitoring (Prometheus/Grafana)** | Cluster and model monitoring |
| **Red Hat SSO (Keycloak)** | Authentication, RBAC, multi-tenancy |

### 4.3 Security Differentiators

- **SCC (Security Context Constraints)** — Kubernetes Pod Security Policies at the OpenShift level
- **FIPS 140-2/140-3 compliance** — cryptographic module validation
- **SELinux enabled** — mandatory access control for containers
- **OpenShift RBAC** — unified access control across all platform components
- **Network policies** — fine-grained traffic control between ML components
- **Air-gapped installation** — support for disconnected environments

### 4.4 GenAI/LLM Capabilities

- **vLLM** for high-throughput LLM serving with PagedAttention
- **InstructLab** integration for fine-tuning with taxonomy-driven approach
- **LangChain integration** for RAG applications
- **OCI-compliant model registries** for portable model storage
- **TrustyAI** for LLM monitoring, drift, bias, and explainability
- **NVIDIA GPU Operator** for OpenShift GPU management

### 4.5 Strengths

- **Deeply integrated with OpenShift security** — SCC, FIPS, SELinux out of the box
- **Single platform for DevOps + MLOps** — reduce tool sprawl, one operational model
- **Red Hat enterprise support** — SLA-backed, certified, LTS releases
- **Built-in pipelines** — Tekton-based, native to OpenShift
- **Serverless + Service Mesh** — Knative + Istio integrated for model serving
- **Hybrid cloud** — consistent experience on-prem, AWS, Azure, GCP
- **Air-gapped support** — important for regulated banking environments
- **TrustyAI** — built-in explainability and bias monitoring
- **Operator-driven lifecycle** — automated upgrades and Day-2 operations

### 4.6 Weaknesses

- **Requires OpenShift license** — additional cost on top of K8s
- **OpenShift AI add-on** — extra subscription cost
- **OpenShift-locked** — cannot run on vanilla Kubernetes
- **Expensive** — significantly higher TCO than upstream Kubeflow
- **Heavy footprint** — OpenShift itself is resource-intensive
- **Smaller community** — compared to upstream Kubeflow or MLflow
- **Less portable** — models/pipelines tied to OpenShift ecosystem

### 4.7 Banking-Specific Considerations

- **SCC + FIPS + SELinux** meet the most stringent bank security requirements (MAS TRM, SR 11-7)
- **TrustyAI** supports EU AI Act compliance with bias detection and explainability
- Air-gapped deployment supports strict network isolation; certified containers reduce supply chain risk
- **Best for:** Regulated enterprises already on OpenShift; banks needing SCC and FIPS; integrated DevOps+MLOps

---

## 5. Deep Dive: Amazon SageMaker

### 5.1 Overview

**Amazon SageMaker** is AWS's fully managed ML platform covering the entire lifecycle. Launched in 2017, it is the most mature cloud ML platform with the broadest set of features.

### 5.2 Core Components

| Component | Purpose |
|-----------|---------|
| **SageMaker Studio** | Integrated IDE for ML with JupyterLab and code editor |
| **Canvas** | No-code ML for business analysts |
| **Ground Truth** | Data labeling with human + automated labeling |
| **Data Wrangler** | Data preparation and feature engineering UI |
| **Clarify** | Bias detection, model explainability (SHAP) |
| **Feature Store** | Centralized feature repository (online + offline) |
| **Autopilot** | AutoML for automatic model selection and tuning |
| **Training** | Managed distributed training with spot instances (up to 90% savings) |
| **Model Registry** | Versioned model catalog with approval workflows |
| **Inference** | Real-time, serverless, batch, multi-model endpoints |
| **Model Monitor** | Drift detection, data quality, bias drift |
| **JumpStart** | Pre-trained models and foundation models |
| **Pipelines** | DAG-based ML workflow orchestration |

### 5.3 Strengths & Weaknesses

- **Fully managed** — no infra to manage. Broadest AWS integration (S3, Redshift, Athena).
- **End-to-end coverage** — data labeling (Ground Truth) through monitoring (Model Monitor).
- **Spot instances** — up to 90% cost reduction for training.
- **JumpStart** — one-click deployment of foundation models.
- **Weaknesses:** AWS-locked (significant migration effort), expensive at scale, pipelines are SageMaker-specific, cannot run on-prem or other clouds.
- **Best for:** AWS-native organizations; teams wanting fully managed ML without ops overhead.

---

## 6. Deep Dive: Vertex AI

### 6.1 Overview

**Vertex AI** is Google Cloud's unified ML platform with AutoML, custom training, and MLOps capabilities, deeply integrated with BigQuery and Gemini foundation models.

### 6.2 Core Components

| Component | Purpose |
|-----------|---------|
| **Vertex AI Studio** | IDE for ML development, prompt design, model tuning |
| **AutoML** | No-code ML: image, text, tabular, video |
| **Custom Training** | Managed training with custom containers, distributed training |
| **Vertex AI Pipelines** | KFP-based ML workflow orchestration (portable) |
| **Model Registry** | Centralized model version management |
| **Endpoints** | Managed model serving (online, batch, private) |
| **Feature Store** | Online + offline feature serving (BigQuery integrated) |
| **Model Monitoring** | Drift detection, feature attribution, outlier detection |
| **Vertex AI Search** | Enterprise search with RAG capabilities |
| **Agent Builder** | Build AI agents with Gemini |
| **Model Garden** | 150+ foundation models (Google, Anthropic, Meta, Mistral) |
| **ML Metadata** | Experiment tracking and lineage (MLMD) |

### 6.3 Gemini & GenAI Integration

- **Gemini API** — direct access to Gemini 1.5 Flash, Pro, Ultra models
- **Model Garden** — 150+ foundation models
- **Vertex AI Studio** — prompt design, testing, evaluation UI
- **Grounding with Google Search** — reduce hallucinations
- **RAG support** — Vertex AI Search + Vector Search
- **Reasoning Engine** — orchestration for complex AI workflows

### 6.4 Strengths & Weaknesses

- **Best AutoML** — industry-leading automated ML for tabular, image, text, video.
- **Tight BigQuery integration** — train models directly from BigQuery data.
- **Best GenAI platform** — Gemini API, Model Garden (150+ FMs), Agent Builder, Studio.
- **KFP-native pipelines** — portable pipeline definitions across platforms.
- **Competitive pricing** — often lower than SageMaker. TPU access for petaflop-scale training.
- **Weaknesses:** GCP-locked, less mature than SageMaker, smaller ecosystem, lower GCP market share in banking.
- **Best for:** GCP-native organizations; data/analytics shops using BigQuery; GenAI-focused projects.

---

## 7. Deep Dive: Azure Machine Learning

### 7.1 Overview

**Azure Machine Learning** is Microsoft's cloud ML platform with tight integration into the Microsoft ecosystem (Azure, Office 365, GitHub, Power Platform) and best-in-class Responsible AI tools.

### 7.2 Core Components

| Component | Purpose |
|-----------|---------|
| **Azure ML Studio** | Web-based IDE with drag-and-drop Designer |
| **Designer** | Visual pipeline builder (no-code/low-code) |
| **AutoML** | Automated model selection and tuning |
| **Pipelines** | ML workflow orchestration with Python SDK |
| **Model Registry** | Centralized model management with versioning |
| **Endpoints (Managed Online)** | Real-time and batch inference endpoints |
| **Responsible AI Dashboard** | Explainability, fairness, error analysis, counterfactuals |
| **Model Monitoring** | Drift detection, data quality |
| **Azure OpenAI Service** | Managed OpenAI models (GPT-4o, DALL-E, Whisper) |
| **Prompt Flow** | LLMOps: prompt engineering, evaluation, deployment |
| **MLflow Integration** | Native MLflow support (no compatibility layer) |
| **GitHub Integration** | CI/CD with GitHub Actions for ML workflows |

### 7.3 Strengths & Weaknesses

- **Microsoft ecosystem integration** — Office 365, Dynamics, Power BI, Teams, GitHub.
- **Best Responsible AI tools** — dashboard for fairness, explainability, error analysis, counterfactuals.
- **Broadest compliance portfolio** — SOC, HIPAA, PCI-DSS, FedRAMP.
- **Azure OpenAI + Prompt Flow** — seamless managed OpenAI with enterprise controls and LLMOps.
- **Hybrid capabilities** — Azure Arc for on-prem ML workloads.
- **Native MLflow support** — no compatibility layer needed.
- **Weaknesses:** Azure-locked, Feature Store still in preview, can be expensive, many SKUs to navigate.
- **Best for:** Microsoft-centric enterprises; regulated industries; Office 365/Dynamics/Power Platform integration.

---

## 8. Alternatives & Ecosystem Tools

### 8.1 Pipeline Orchestration

#### Dagster
- **Type:** Data/ML pipeline orchestrator — asset-based, software-defined assets
- **Strengths:** Great UI, testing built in, dbt + Spark integration; fresh approach vs Airflow
- **Weaknesses:** ML-specific features still growing, smaller than Airflow
- **Best for:** Data + ML teams wanting unified orchestration | **License:** Apache 2.0 | **Stars:** 12k+

#### Flyte
- **Type:** K8s-native ML workflow orchestration — type-rich SDK, re-entrant workflows
- **Strengths:** Type safety (mypy), auto-resume on failure, multi-tenant; used by Lyft, Spotify, Stripe
- **Weaknesses:** Requires K8s, complex setup, smaller ecosystem
- **Best for:** Type-safe ML orchestration on K8s | **License:** Apache 2.0 | **Stars:** 6k+

#### ZenML
- **Type:** Open-source MLOps framework — pipeline-centric with stack abstraction
- **Strengths:** Clean abstractions, experiment-to-production pipeline, pluggable backends
- **Weaknesses:** Newer, smaller community, abstraction layer complexity
- **Best for:** Framework to abstract underlying infra | **License:** Apache 2.0 | **Stars:** 4k+

#### Metaflow
- **Type:** Human-centric ML orchestration (Netflix/Outerbounds) — DAG-based, cloud execution
- **Strengths:** Data scientist-friendly, Pythonic API, easy local-to-cloud scaling, Netflix-proven
- **Weaknesses:** Python-only, non-ML gaps, enterprise features via Outerbounds
- **Best for:** Easy scaling from laptop to cloud | **License:** Apache 2.0 | **Stars:** 8k+

### 8.2 Experiment Tracking & Model Registry

#### Weights & Biases (W&B)
- **Features:** Runs (tracking), Sweeps (HPO), Artifacts (versioning), Prompts (LLM tracing)
- **Strengths:** Best experiment tracking UI, excellent LLM support, 1k+ integrations, large community
- **Weaknesses:** Cloud-hosted (self-hosted limited), expensive at scale, data egress concerns
- **Best for:** Experiment tracking + LLM prompt engineering | **License:** Proprietary (free tier) | **Stars:** 9k+

#### Neptune.ai
- **Features:** Experiment tracking, model registry, monitoring dashboards
- **Strengths:** Clean UI, good SDK, model comparison, monitoring integration
- **Weaknesses:** Smaller community than W&B, fewer integrations, cloud-only
- **Best for:** Tracking + registry + monitoring in one tool | **License:** Proprietary (free tier)

#### ClearML
- **Features:** Experiment tracking, pipelines, model serving, data management
- **Strengths:** Free, open source, self-hosted, broad feature set
- **Weaknesses:** UI less polished, smaller community, documentation gaps
- **Best for:** Free self-hosted ML platform | **License:** Apache 2.0 | **Stars:** 5k+

### 8.3 Enterprise ML Platforms

| Platform | Type | Key Differentiator | Best For | License |
|----------|------|-------------------|----------|---------|
| **H2O.ai** | AutoML + ML platform | Strong AutoML, model interpretability | AutoML with strong explainability | Apache 2.0 / Proprietary |
| **Dataiku** | Collaborative DS/ML platform | Visual ML, governance, collaboration | Enterprise governed data science teams | Proprietary |
| **Predibase** | Declarative ML (Ludwig) | Low-code fine-tuning, fast deployment | Fine-tuning LLMs and tabular models | Proprietary |
| **Modal** | Serverless ML compute | Fast GPU serverless, pay-per-use, Pythonic | ML compute — training, batch inference | Proprietary |

---

## 9. Comparison Across Dimensions

### 9.1 Architecture: Managed vs Self-Hosted vs Hybrid

| Platform | Architecture | Deployment Options |
|----------|-------------|-------------------|
| **MLflow** | Hybrid | Self-hosted, managed (Databricks), or any cloud VM |
| **Kubeflow** | Self-hosted | Any K8s cluster (cloud, on-prem, hybrid) |
| **OpenShift AI** | Self-hosted (on OpenShift) | OpenShift clusters (on-prem, cloud, hybrid) |
| **SageMaker** | Managed (AWS) | AWS cloud only |
| **Vertex AI** | Managed (GCP) | GCP cloud only (some on-prem via GDC) |
| **Azure ML** | Managed (Azure) | Azure cloud + hybrid (Azure Arc) |

### 9.2 ML Lifecycle Coverage

```
                     Data    Exp     Train   Tune    Eval    Deploy  Monitor  Registry  Feature
                     Prep    Track                                   /Serve             Store
MLflow              ✗       ✓       ✗       ✓       ✓       ○       ✗       ✓       ✗
Kubeflow            ○       ○       ✓       ✓       ○       ✓       ○       ○       ✗
OpenShift AI        ○       ○       ✓       ✓       ✓       ✓       ✓       ✓       ✗
SageMaker           ✓       ✓       ✓       ✓       ✓       ✓       ✓       ✓       ✓
Vertex AI           ○       ✓       ✓       ✓       ✓       ✓       ✓       ✓       ✓
Azure ML            ○       ✓       ✓       ✓       ✓       ✓       ✓       ✓       ○
```
**Legend:** ✓ = Native / Full  |  ○ = Via integration  |  ✗ = Not supported

### 9.3 MLOps Capabilities

| Capability | MLflow | Kubeflow | OCP AI | SageMaker | Vertex AI | Azure ML |
|-----------|--------|----------|--------|-----------|-----------|----------|
| **CI/CD for ML** | ✗ | ○ (Tekton) | ✓ | ✓ | ✓ | ✓ |
| **Experiment Tracking** | ✓✓✓ | ○ (MLflow) | ○ | ✓ | ✓ | ✓ |
| **Model Registry** | ✓✓ | ○ | ✓ | ✓✓ | ✓✓ | ✓✓ |
| **Pipeline Orchestration** | ○ (Basic) | ✓✓ (KFP) | ✓✓ (Tekton) | ✓ | ✓✓ (KFP) | ✓ |
| **Model Monitoring** | ✗ | ○ | ✓ (TrustyAI) | ✓✓ | ✓ | ✓ |
| **Feature Store** | ✗ | ✗ | ✗ | ✓✓ | ✓ | ○ (Preview) |
| **Hyperparameter Tuning** | ✓ | ✓✓ (Katib) | ✓ | ✓✓ | ✓✓ (Vizier) | ✓✓ |
| **AutoML** | ✗ | ✗ | ✗ | ✓✓ (Autopilot) | ✓✓✓ | ✓✓ |

### 9.4 GenAI / LLM Support

| Capability | MLflow | Kubeflow | OCP AI | SageMaker | Vertex AI | Azure ML | W&B |
|-----------|--------|----------|--------|-----------|-----------|----------|-----|
| **Prompt Engineering** | ✓ (UI) | ✗ | ✗ | ○ | ✓✓ (Studio) | ✓✓ (Prompt Flow) | ✓✓ |
| **RAG Support** | ✗ | ✗ | ✓ (LangChain) | ✓ (KB for Bedrock) | ✓✓ (Search+Vector) | ✓ (AI Search) | ✗ |
| **Fine-tuning** | ✗ | ✓ (Trainer) | ✓ (InstructLab) | ✓ (JumpStart) | ✓ (Model Garden) | ✓ (Azure OpenAI) | ✗ |
| **LLM Serving** | ○ | ✓ (KServe+vLLM) | ✓ (KServe+vLLM) | ✓ (JumpStart) | ✓ (Model Garden) | ✓ (Azure OpenAI) | ✗ |
| **LLM Evaluation** | ✓✓ | ✗ | ✓ (TrustyAI) | ✓ (FM Evaluator) | ✓ (Eval SDK) | ✓ (Prompt Flow) | ✓✓ |

### 9.5 Governance & Security

| Capability | MLflow | Kubeflow | OCP AI | SageMaker | Vertex AI | Azure ML |
|-----------|--------|----------|--------|-----------|-----------|----------|
| **Model Risk Management** | ✗ | ✗ | ○ | ○ | ○ | ✓ |
| **Bias Monitoring** | ✗ | ✗ | ✓ | ✓ | ○ | ✓✓ |
| **Explainability** | ✗ | ○ (Alibi) | ✓ (TrustyAI) | ✓ | ✓ | ✓✓ |
| **Audit Trails** | ○ (Basic) | ○ (K8s audit) | ✓✓ (OpenShift audit) | ✓ (CloudTrail) | ✓ (Audit Logs) | ✓ (Azure Monitor) |
| **RBAC / Multi-tenancy** | ✗ | ○ (KFAM) | ✓✓ (SCC+RBAC) | ✓ (IAM) | ✓ (IAM) | ✓ (Azure RBAC) |
| **FIPS 140-2/3** | ✗ | ✗ | ✓✓ | ○ (Limited) | ○ (Limited) | ✓ |
| **Air-gapped** | ✓ | ✓ | ✓✓ | ✗ | ✗ | ○ |

### 9.6 Pricing Model & Community

| Dimension | MLflow | Kubeflow | OpenShift AI | SageMaker | Vertex AI | Azure ML |
|----------|--------|----------|-------------|-----------|-----------|----------|
| **Pricing Model** | Free (OSS) | Free (OSS) | OpenShift license + AI subscription | Pay per compute | Pay per compute | Pay per compute |
| **Open Source** | ✓ (Apache 2.0) | ✓ (Apache 2.0) | Core: ✓ (ODH), Platform: ✗ | ✗ | ✗ | ✗ |
| **Cloud Lock-in** | Very Low | Low | Medium-High | High | High | High |
| **Learning Curve** | Low | High | Medium | Low-Medium | Medium | Medium |
| **Community Size** | Very Large | Large | Small (product) | Very Large | Large | Very Large |
| **Enterprise Support** | Databricks | Canonical, Arrikto | Red Hat (24/7) | AWS Enterprise | Google Enterprise | Microsoft Enterprise |

### 9.7 Vendor Lock-In Risk

| Platform | Model Portability | Pipeline Portability | Overall Lock-In |
|----------|-----------------|---------------------|-----------------|
| **MLflow** | ✓✓✓ (Standard format) | N/A | Very Low |
| **Kubeflow** | ✓✓ (K8s native) | ✓✓ (KFP standard) | Low |
| **OpenShift AI** | ✓ (K8s/OCI containers) | ✓ (Tekton) | Medium-High |
| **SageMaker** | ○ (SageMaker-specific) | ✗ (SageMaker pipelines) | High |
| **Vertex AI** | ○ (Container-based) | ✓ (KFP, portable) | Medium-High |
| **Azure ML** | ○ (Azure-specific) | ✗ (Azure pipelines) | High |

---

## 10. Platform Selection Criteria

### 10.1 Strategic Factors

#### Cloud Strategy
| Scenario | Recommended |
|----------|------------|
| **AWS-native** | SageMaker (deepest AWS integration) |
| **GCP-native** | Vertex AI (tied to GCP ecosystem) |
| **Azure-centric** | Azure ML (Microsoft ecosystem) |
| **Multi-cloud** | Kubeflow or MLflow + cloud-agnostic stack |
| **On-prem / Hybrid** | OpenShift AI or Kubeflow on-prem |

#### Team Skills
| Team Composition | Recommended |
|-----------------|-------------|
| **Data scientists only** | SageMaker / Vertex AI / Azure ML (managed) |
| **DS + ML engineers** | MLflow + cloud platform + orchestration |
| **MLE + platform engineers** | Kubeflow or OpenShift AI |
| **Platform engineers only (new ML team)** | Cloud platform (SageMaker/Vertex/Azure) |

#### ML Maturity
| Level | Characteristics | Recommended Stack |
|-------|----------------|------------------|
| **Experimental** | Ad-hoc notebooks, single DS | MLflow + local Jupyter |
| **Scaling** | Multiple teams, basic CI/CD | MLflow + cloud platform + Dagster/Flyte |
| **Production at scale** | 100+ models, full MLOps, governance | OpenShift AI or cloud platform + full stack |

### 10.2 Compliance & Regulatory Criteria

| Requirement | Supported Platforms |
|------------|-------------------|
| **SOC 2** | All cloud platforms |
| **HIPAA** | SageMaker, Vertex AI, Azure ML |
| **PCI-DSS** | SageMaker, Azure ML |
| **FedRAMP** | OpenShift AI, Azure ML (limited), SageMaker (limited) |
| **FIPS 140-2/3** | OpenShift AI ✓, Azure ML ✓, others limited |
| **MAS TRM** | All major platforms (regional availability varies) |
| **SR 11-7 (MRM)** | Custom governance on any; OpenShift AI + TrustyAI, SageMaker Clarify |
| **MAS FEAT** | Vertex AI XAI, Azure RAI, OpenShift AI TrustyAI |
| **EU AI Act** | Azure RAI (best), OpenShift AI TrustyAI, SageMaker Clarify |

### 10.3 Budget & Scale

| Budget | Recommended Stack |
|--------|-----------------|
| **$0-2k/month** | MLflow (self-hosted) + Modal (serverless) |
| **$2k-10k/month** | Cloud platform + MLflow |
| **$10k-50k/month** | Full cloud platform + W&B + orchestration |
| **$50k+/month** | OpenShift AI or enterprise cloud + full governance |

### 10.4 Scale Factors

| Factor | Lightweight | Medium | Heavy |
|--------|------------|--------|-------|
| **Models in production** | <10 | 10-100 | 100+ |
| **Data volume** | <1TB | 1-100TB | 100TB+ |
| **Training frequency** | Weekly | Daily | Real-time/streaming |
| **Inference latency** | Batch OK | <100ms | <10ms |

---

## 11. Typical Platform Stacks

### 11.1 Startup / Small Team
```
MLflow (tracking + registry) + Modal (serverless GPU) + Hugging Face (models)
```
- **Cost:** $0-2k/month | **Team:** 1-5 DS | **Models:** 1-10

### 11.2 Mid-Size / Growth
```
MLflow (experiment tracking + model registry)
+ Dagster or Flyte (pipeline orchestration)
+ SageMaker / Vertex AI / Azure ML (training + serving)
+ W&B or Neptune (experiment visualization, optional)
+ Feast (feature store, optional)
```
- **Cost:** $2k-15k/month | **Team:** 5-20 (DS + MLE) | **Models:** 10-50

### 11.3 Enterprise / Regulated
```
OpenShift AI (full platform — pipelines + serving + monitoring)
+ MLflow (experiment tracking, supplementary)
+ TrustyAI (explainability + bias monitoring)
+ Custom governance layer (MRM, audit trails, approval workflows)
```
**Alternative (if not on OpenShift):**
```
SageMaker / Vertex AI / Azure ML (training + serving)
+ MLflow (model registry with audit hooks)
+ Custom model risk management platform
+ On-premises serving (for regulated workloads)
```
- **Cost:** $50k-500k+/year | **Team:** 20+ | **Models:** 50-500+

### 11.4 Bank / Financial Services

**Scenario A — On OpenShift:**
```
OpenShift AI + SCC + FIPS + SELinux + TrustyAI + OpenShift audit
  + KServe + vLLM + Custom governance (SR 11-7 / MAS FEAT)
```

**Scenario B — Multi-Cloud / Hybrid:**
```
SageMaker/Vertex AI (cloud training) + MLflow (registry + lineage)
  + On-prem KServe (regulated serving) + Custom MRM + Prometheus/Grafana
```

**Scenario C — GenAI Focused:**
```
Vertex AI + Gemini (LLM) + LangChain/LlamaIndex (RAG)
  + W&B (prompt tracking + evaluation) + Guardrails AI + Custom prompt registry
```

---

## 12. Platform Comparison Matrix

| Feature | MLflow | Kubeflow | OpenShift AI | SageMaker | Vertex AI | Azure ML |
|---------|--------|----------|-------------|-----------|-----------|----------|
| **Category** | Exp. Tracking + Registry | Full Platform (K8s) | Full Platform (OpenShift) | Full Platform (AWS) | Full Platform (GCP) | Full Platform (Azure) |
| **Deployment** | Self-hosted, Databricks | Any K8s cluster | OpenShift only | AWS cloud only | GCP cloud only | Azure + hybrid (Arc) |
| **Lifecycle Score** | 3/7 stages | 5/7 stages | 6/7 stages | **7/7 stages** ✓✓ | **7/7 stages** ✓✓ | **7/7 stages** ✓✓ |
| **Experiment Tracking** | ✓✓✓ (Best) | ○ (Basic + MLflow) | ○ (MLflow integ.) | ✓ | ✓ | ✓ (MLflow native) |
| **Pipelines** | ○ (Basic) | ✓✓ (KFP) | ✓✓ (Tekton) | ✓ | ✓✓ (KFP) | ✓ |
| **Model Registry** | ✓✓ (Standard) | ○ (Metadata) | ✓ | ✓✓ | ✓✓ | ✓✓ |
| **Model Serving** | ○ (Thin wrapper) | ✓✓ (KServe) | ✓✓ (KServe+vLLM) | ✓✓ (Multi-endpoint) | ✓✓ (Endpoints) | ✓✓ (Managed Online) |
| **Model Monitoring** | ✗ | ○ (Prometheus) | ✓ (TrustyAI) | ✓✓ (Model Monitor) | ✓ | ✓ |
| **Feature Store** | ✗ | ✗ | ✗ | ✓✓ (SageMaker FS) | ✓ | ○ (Preview) |
| **Data Labeling** | ✗ | ✗ | ✗ | ✓ (Ground Truth) | ○ | ✗ |
| **AutoML** | ✗ | ✗ | ✗ | ✓ (Autopilot) | ✓✓✓ (Best) | ✓ |
| **Hyperparameter Tuning** | ✓ (Basic) | ✓✓ (Katib) | ✓ (Katib) | ✓✓ (Bayesian) | ✓✓ (Vizier) | ✓✓ (Hyperdrive) |
| **LLM / GenAI Support** | ✓ (Tracking+Eval) | ○ (KServe+vLLM) | ✓ (InstructLab+vLLM) | ✓ (JumpStart) | ✓✓✓ (Gemini+Studio) | ✓✓ (OpenAI+Prompt Flow) |
| **RAG Support** | ✗ | ✗ | ✓ (LangChain) | ✓ (Bedrock KBs) | ✓✓ (Search+Vector) | ✓ (AI Search) |
| **LLM Evaluation** | ✓✓ | ✗ | ✓ (TrustyAI) | ✓ (FM Evaluator) | ✓ (Eval SDK) | ✓ (Prompt Flow) |
| **Governance** | ✗ | ✗ | ○ (TrustyAI) | ○ (Clarify) | ○ | ✓✓ (Responsible AI) |
| **Bias Detection** | ✗ | ✗ | ✓ (TrustyAI) | ✓ (Clarify) | ○ | ✓✓ |
| **Explainability** | ✗ | ○ (Alibi) | ✓ (TrustyAI) | ✓ (Clarify) | ✓ (XAI) | ✓✓ |
| **Audit Trails** | ○ (Basic) | ○ (K8s audit) | ✓✓ (OpenShift audit) | ✓ (CloudTrail) | ✓ (Audit Logs) | ✓ (Azure Monitor) |
| **RBAC / Multi-tenancy** | ✗ | ○ (KFAM) | ✓✓ (SCC+RBAC) | ✓ (IAM) | ✓ (IAM) | ✓ (Azure RBAC) |
| **SSO / OIDC** | ○ (Plugins) | ✓ (Dex) | ✓ (Keycloak) | ✓ (IAM+SSO) | ✓ (Cloud Identity) | ✓ (Azure AD) |
| **FIPS 140-2/3** | ✗ | ✗ | ✓✓ (Native) | ○ (Limited) | ○ (Limited) | ✓ |
| **SCC / Pod Security** | ✗ | ✗ | ✓✓ (SCC+SELinux) | ✓ (Execution role) | ✓ (VPC-SC) | ✓ (AKS) |
| **Air-gapped Support** | ✓ | ✓ | ✓✓ | ✗ | ✗ | ○ |
| **GPU Support** | Via infra | ✓ (K8s GPU) | ✓ (NVIDIA Op.) | ✓✓ (Managed) | ✓✓ (GPU+TPU) | ✓✓ (Managed) |
| **Distributed Training** | ✗ | ✓ (TF/PyTorch Jobs) | ✓ (CodeFlare+Ray) | ✓ | ✓ | ✓ |
| **Auto-scaling** | ✗ | ✓ (HPA/VPA) | ✓ (HPA+Knative) | ✓✓ | ✓✓ | ✓✓ |
| **Pricing Model** | Free (OSS) | Free (OSS) | OCP license + AI add-on | Pay-per-use | Pay-per-use | Pay-per-use |
| **Open Source** | ✓ (Apache 2.0) | ✓ (Apache 2.0) | Core: ✓ (ODH) | ✗ | ✗ | ✗ |
| **Cloud Lock-in** | Very Low | Low | Medium-High | High | High | High |
| **Learning Curve** | Low | High | Medium | Low-Medium | Medium | Medium |
| **Community Size** | Very Large | Large | Small (product) | Very Large | Large | Very Large |
| **Best For** | Tracking + Registry as stack part | K8s-native teams, full lifecycle | Regulated enterprise on OpenShift | AWS-native orgs | GCP-native + GenAI | Microsoft-centric enterprise |

---

## 13. Decision Tree & Selection Framework

### 13.1 Decision Flow Chart

```
START: Need an ML Platform
         │
         ├── Cloud-native? (committed to one cloud)
         │     ├── AWS → SageMaker
         │     ├── GCP → Vertex AI
         │     ├── Azure → Azure ML
         │     └── Multi-cloud/No preference → Go to K8s question
         │
         ├── K8s-native? (have K8s team/expertise)
         │     ├── On OpenShift?
         │     │     ├── Yes → OpenShift AI (budget allows) or Kubeflow
         │     │     └── No → Kubeflow (if team has K8s skills)
         │     └── No K8s → Go to scope question
         │
         ├── Just need lightweight experiment tracking?
         │     ├── Yes → MLflow (lightweight, easy, no lock-in)
         │     └── No → Go to scope question
         │
         ├── Need full ML lifecycle (data → monitoring)?
         │     ├── Yes → Cloud platform (SageMaker/Vertex/Azure) or Kubeflow
         │     └── No → MLflow + point solutions
         │
         ├── Enterprise / Regulated?
         │     ├── Yes → OpenShift AI or cloud platform + custom governance
         │     └── No → Kubeflow or cloud platform
         │
         ├── GenAI / LLM Focus?
         │     ├── Yes → Vertex AI (+ Gemini) or Azure ML (+ OpenAI) + W&B
         │     └── No → Based on other criteria
         │
         └── Cost Sensitive?
               ├── Yes → MLflow + Dagster + KServe (self-hosted, OSS stack)
               └── No → Managed platform (SageMaker/Vertex/Azure/OpenShift AI)
```

### 13.2 Quick Decision Matrix

| Your Situation | Recommended Platform | Rationale |
|--------------|---------------------|-----------|
| AWS-native, want fully managed | **SageMaker** | Deepest AWS integration, end-to-end |
| GCP-native, GenAI focus | **Vertex AI + Gemini** | Best GenAI, best AutoML |
| Microsoft-centric, compliance needs | **Azure ML** | Microsoft ecosystem, best Responsible AI |
| On OpenShift, regulated industry | **OpenShift AI** | SCC/FIPS, single platform, enterprise support |
| On OpenShift, cost-sensitive | **Kubeflow on OpenShift** | Open source, more flexible, less cost |
| K8s-native, multi-cloud | **Kubeflow** | Portable across any K8s, KServe + Katib |
| Just need tracking + registry | **MLflow** | Lightweight, easy integration, huge community |
| Data + ML pipeline focus | **Dagster + MLflow** | Asset-based orchestration + tracking |
| Type-safe ML orchestration on K8s | **Flyte + MLflow** | Re-entrant, type-rich, scalable |
| Experiment-to-production framework | **ZenML + your infra** | Clean abstractions, stack-agnostic |
| LLM prompt engineering + evaluation | **W&B + MLflow** | Best LLM tracing + evaluation |
| AutoML focus | **Vertex AI (AutoML) or H2O.ai** | Best-in-class automated ML |

### 13.3 Weighted Decision Framework

For formal platform selection, use weighted scoring across these categories:

| Category | Weight (Enterprise) | Weight (Startup) | Description |
|----------|-------------------|-----------------|-------------|
| **Functionality** | 20% | 30% | Does it cover needed ML lifecycle stages? |
| **Security & Compliance** | 25% | 5% | FIPS, SCC, audit trails, certifications |
| **Governance** | 15% | 5% | MRM, bias monitoring, explainability |
| **Operational Overhead** | 10% | 20% | Ease of deployment, Day-2 operations |
| **Learning Curve** | 5% | 15% | Time to productivity for data scientists |
| **Total Cost of Ownership** | 10% | 15% | Licensing + compute + operational cost |
| **Vendor Lock-In** | 10% | 5% | Portability of models and pipelines |
| **Ecosystem & Community** | 5% | 5% | Integrations, support, hiring pool |

---

## 14. Recommendations for Crédit Agricole CIB

### 14.1 Context Summary

**Regulatory Requirements:** SCC, FIPS 140-2/3, audit trails, SR 11-7 (model risk management), MAS FEAT (Fairness/Ethics/Accountability/Transparency), EU AI Act.
**Infrastructure:** Potential OpenShift investment, need for integrated DevOps+MLOps, hybrid cloud (on-prem for regulated, cloud for elasticity), Singapore (MAS) and European (ECB) jurisdictions.
**Companion Guide:** The **Charmed Kubeflow vs OpenShift AI** guide covers that specific comparison in detail.

### 14.2 Recommendation 1: OpenShift AI (If on OpenShift)

**If the bank already has or is investing in Red Hat OpenShift:**

```
✅ RECOMMEND: OpenShift AI
```

**Rationale:** SCC + FIPS + SELinux meet non-negotiable banking security requirements. Single DevOps+MLOps platform reduces complexity. TrustyAI provides explainability, bias detection, and compliance (MAS FEAT, EU AI Act). OpenShift audit logging for regulatory audit trails (SR 11-7, MAS TRM). Air-gapped deployment for regulated workloads. Red Hat 24/7 SLA support.

**Stack:** OpenShift AI (Platform) + MLflow (Experiment Tracking) + TrustyAI (Explainability) + Custom Governance Layer (SR 11-7/audit trails/approvals) + OpenShift Monitoring & Audit (Prometheus/Grafana)

### 14.3 Recommendation 2: Cloud Platform + MLflow + Custom Governance (Multi-Cloud/Hybrid)

**If the bank is pursuing a multi-cloud or hybrid strategy:**

```
✅ RECOMMEND: Cloud platform (SageMaker/Vertex/Azure ML) + MLflow + Custom governance
```

**Rationale:** Use cloud platform for training/development (elastic GPU, managed infra). Keep sensitive model serving on-prem (regulated workloads). MLflow as portable registry — models not locked to any cloud. Custom governance layer for SR 11-7, audit trails, approval workflows.

**Cloud Selection:** Azure ML if Azure-centric (common in European banking) or Vertex AI if GenAI-focused; SageMaker if AWS-centric.

### 14.4 Recommendation 3: Kubeflow (If K8s-Native but NOT OpenShift)

**If the bank is Kubernetes-native but not on OpenShift:**

```
✅ RECOMMEND: Kubeflow (or Charmed Kubeflow for enterprise support)
```

**Rationale:** Full ML lifecycle on vanilla K8s. Charmed Kubeflow (Canonical) for enterprise support. Works on any K8s distribution (EKS, GKE, AKS, on-prem). KServe + vLLM for serving, Katib for tuning. More portable than OpenShift AI.

**Note:** Requires strong K8s platform engineering team. SCC/FIPS need to be implemented at the K8s layer.

### 14.5 Summary Recommendation

```
Priority 1: OpenShift AI    (if on OpenShift — SCC/FIPS/audit make this the best fit)
Priority 2: Cloud + MLflow + custom governance   (multi-cloud/hybrid)
Priority 3: Kubeflow    (K8s-native, not on OpenShift)
```

### 14.6 Recommended Stack Diagram

```
┌──────────────────────────────────────────┐
│         OpenShift AI (Platform)          │  ← Lifecycle, security, governance
├──────────────────────────────────────────┤
│  MLflow (Experiment Tracking)            │  ← Supplementary, portable
├──────────────────────────────────────────┤
│  TrustyAI (Explainability + Bias)        │  ← MAS FEAT, EU AI Act compliance
├──────────────────────────────────────────┤
│  Custom Governance Layer                  │  ← SR 11-7, audit trails, approvals
│   • MRM (inventory, validation, monitor) │
│   • Approval workflows (onboard, promote,│
│     retire models)                       │
│   • Regulatory reporting (MAS, ECB, OCC) │
├──────────────────────────────────────────┤
│  OpenShift Monitoring + Audit            │
│  (Prometheus, Grafana, OpenShift Audit)  │
└──────────────────────────────────────────┘
```

---

## 15. References & Further Reading

### Related Guides
- **Enterprise AI Platforms Guide** — Broader AI platform landscape
- **MLOps Lifecycle Frameworks Guide** — MLOps maturity models and process frameworks
- **Charmed Kubeflow vs OpenShift AI Guide** — Detailed comparison of the two K8s ML platforms

### Official Docs
- MLflow: https://mlflow.org/docs/ | Kubeflow: https://www.kubeflow.org/docs/
- OpenShift AI: https://docs.redhat.com/en/documentation/red_hat_openshift_ai/
- SageMaker: https://docs.aws.amazon.com/sagemaker/ | Vertex AI: https://cloud.google.com/vertex-ai/docs
- Azure ML: https://learn.microsoft.com/en-us/azure/machine-learning/

### Industry Reports
- Gartner Magic Quadrant for Data Science and ML Platforms (2025)
- Forrester Wave: MLOps Platforms (2024)
- Algorithmia State of Enterprise MLOps (2024)
- CNCF MLOps Survey (2025)

### Regulatory References
- SR 11-7 / OCC 2011-12: Model Risk Management
- MAS FEAT Principles: Fairness, Ethics, Accountability, Transparency
- EU AI Act: Regulation (EU) 2024/1689
- MAS Technology Risk Management Guidelines (TRM)
- BCBS 239: Principles for Effective Risk Data Aggregation

---

> **Author:** Jack Liu Shurui | **Role:** Solution Architect | **Bank:** Crédit Agricole CIB | **Office:** Singapore
>
> **Part of the MLOps Series** — For questions or corrections, reach out via the internal research repository.
>
> **License:** Internal use — Crédit Agricole CIB
