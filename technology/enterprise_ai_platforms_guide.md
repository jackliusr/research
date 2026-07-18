# Enterprise AI Platforms: A Comprehensive Guide

> **Author:** Jack Liu Shurui  
> **Role:** Solution Architect, Crédit Agricole CIB  
> **Date:** July 2026  
> **Version:** 1.0  
> **Repository:** github.com/jackliusr/research  

---

## Table of Contents

1. [What Are Enterprise AI Platforms?](#1-what-are-enterprise-ai-platforms)
2. [Platform Categories](#2-platform-categories)
3. [Major Platforms Deep Dive](#3-major-platforms-deep-dive)
4. [Vendor Comparison Table](#4-vendor-comparison-table)
5. [Platform Evaluation Criteria](#5-platform-evaluation-criteria)
6. [Architecture Patterns](#6-architecture-patterns)
7. [Enterprise AI Governance](#7-enterprise-ai-governance)
8. [Vendor Lock-In Risk Analysis](#8-vendor-lock-in-risk-analysis)
9. [Deployment Models for Banking](#9-deployment-models-for-banking)
10. [Key Trends](#10-key-trends)
11. [Decision Framework](#11-decision-framework)
12. [Architecture Diagram](#12-architecture-diagram)
13. [References](#13-references)

---

## 1. What Are Enterprise AI Platforms?

Enterprise AI platforms are **integrated software suites** that enable organizations to build, deploy, govern, monitor, and scale AI/ML applications in production. Unlike **point tools** (individual libraries, notebooks, or model registries), enterprise AI platforms provide a **unified stack** spanning the full AI lifecycle:

| Capability | Point Tool Example | Enterprise Platform Coverage |
|---|---|---|
| Data ingestion & preparation | Pandas, dbt | Built-in pipelines + connectors |
| Experiment tracking | MLflow Tracking alone | Integrated with model registry |
| Model training | Jupyter notebook | Managed training infra + auto-ML |
| Model serving | Flask/FastAPI API | Managed endpoints with auto-scaling |
| Model monitoring | Custom scripts | Drift detection, alerts, dashboards |
| Governance | Spreadsheet registry | RBAC, audit trails, compliance reports |
| LLM support | OpenAI SDK alone | Prompt mgmt, RAG templates, guardrails |
| Cost management | Manual tracking | Token tracking, budget alerts |

### Why Organizations Need an Enterprise AI Platform

- **Scale:** From 1 model to hundreds in production
- **Governance:** Regulatory compliance (EU AI Act, MAS FEAT, SR 11-7)
- **Collaboration:** Data scientists, ML engineers, compliance, business analysts on shared infra
- **Reproducibility:** Trace every model from data to deployment
- **Speed:** Templated pipelines reduce time-to-production from months to weeks
- **Cost control:** Centralized GPU/TPU allocation, token tracking, budget governance

### Enterprise AI Platforms vs. Point Tools — A Decision Guide

| Factor | Point Tools | Enterprise Platform |
|---|---|---|
| Team size | < 10 data scientists | 10+ cross-functional |
| Model count | 1–5 models | 20+ models in production |
| Compliance needs | Low | High (banking, healthcare, insurance) |
| Cloud maturity | Single cloud | Multi-cloud or hybrid |
| Budget | Build custom | Buy integrated |
| Time to market | Flexible | Accelerated via templates |

---

## 2. Platform Categories

Enterprise AI platforms fall into six major categories, each with distinct strengths and trade-offs.

### 2.1 Data Science & ML Platforms

Full lifecycle platforms for data preparation, feature engineering, model training, and deployment. Best for organizations with strong data engineering needs.

| Platform | Key Strength | Open Source | Typical Buyer |
|---|---|---|---|
| **Databricks** | Unified lakehouse + AI; Delta Sharing, Mosaic AI | Partial (Spark, MLflow) | Data-heavy orgs on AWS/Azure/GCP |
| **Dataiku** | Visual data prep, citizen data science | Partial (core available) | Hybrid teams (coders + low-code) |
| **H2O.ai** | AutoML, model interpretability, Driverless AI | Yes (H2O-3, Sparkling Water) | Teams needing fast baseline models |
| **Amazon SageMaker** | Deep AWS integration, managed training | No | AWS-native organizations |
| **Google Vertex AI** | Vertex AI Pipelines, AutoML, Gemini | No | GCP-native organizations |
| **Azure Machine Learning** | Azure integration, responsible AI tools | No | Azure-native organizations |

### 2.2 MLOps Platforms

Platforms focused on the model lifecycle: experiment tracking, model registry, deployment pipelines, and monitoring. Used alongside or on top of compute infrastructure.

| Platform | Key Strength | Open Source |
|---|---|---|
| **MLflow** | Model registry, experiment tracking, serving | Yes (Apache 2.0) |
| **Kubeflow** | Kubernetes-native ML workflows, pipelines | Yes (Apache 2.0) |
| **Weights & Biases** | Experiment tracking, sweeps, report sharing | No (free tier) |
| **Neptune.ai** | Metadata store, experiment comparison, model registry | No (free tier) |
| **ZenML** | Pipeline orchestration, MLOps as code | Yes (Apache 2.0) |
| **Flyte** | Data & ML pipelines, multi-tenant, multi-cloud | Yes (Apache 2.0) |

### 2.3 Purpose-Built AI Platforms

Cloud platforms providing managed access to foundation models, fine-tuning, and deployment. Best for teams that want to consume AI as a service.

| Platform | Key Models | Differentiator |
|---|---|---|
| **OpenAI Platform** | GPT-4o, o-series, DALL-E, Whisper | Best-in-class frontier models |
| **Anthropic Console** | Claude 3.5, Claude 4 | Safety, constitutional AI, long context |
| **Google Vertex AI** | Gemini 2.0, Gemma, Imagen | Gen AI Studio, Agent Builder, Model Garden |
| **Azure AI Studio (Foundry)** | GPT-4o, Llama, Mistral, Cohere | Azure AI Search, Copilot connectors |
| **AWS Bedrock** | Claude, Llama, Mistral, Titan | Knowledge bases, guardrails, agents |
| **IBM watsonx** | Granite, Llama, Mistral | AI governance (watsonx.governance) |

### 2.4 Generative AI Frameworks

Open-source frameworks for building RAG pipelines, agentic workflows, and LLM-powered applications. These are typically **layered on top** of the infrastructure platforms above.

| Framework | Primary Focus | Language | Key Feature |
|---|---|---|---|
| **LangChain** | Agent orchestration, chains, RAG, tool use | Python/JS | Largest ecosystem, LangSmith observability |
| **LlamaIndex** | Data indexing, RAG, structured retrieval | Python | Best-in-class data connectors |
| **Haystack (deepset)** | Document QA, search pipelines, RAG | Python | Production-proven, modular components |
| **Semantic Kernel** | Enterprise AI orchestration for Microsoft | C#/Python/Java | Deep Microsoft ecosystem integration |
| **Dify** | Visual LLM app builder | Python | Low-code RAG + agent workflows |
| **AutoGen (Microsoft)** | Multi-agent conversations | Python | Complex agent-to-agent delegation |

### 2.5 LLMOps Platforms

Purpose-built for LLM-specific operations: prompt management, token cost tracking, LLM evaluation, guardrails, and observability for generative AI.

| Platform | Key Strength | Open Source |
|---|---|---|
| **LangSmith** | Tracing, evaluation, prompt management for LangChain apps | No (free tier) |
| **MLflow (LLM features)** | MLflow Evaluate, tracing, model registry for LLMs | Yes (Apache 2.0) |
| **Arize AI / Phoenix** | LLM observability, drift detection, embedding monitoring | Yes (Phoenix) |
| **Helicone** | LLM proxy, cost tracking, caching, rate limiting | Yes |
| **Langfuse** | Open-source LLM observability, prompt management, evaluation | Yes (MIT) |
| **Weights & Biases Weave** | LLM tracing, evaluation, dataset versioning | No (free tier) |
| **Portkey** | AI gateway, guardrails, cost management, fallbacks | Yes |
| **Guardrails AI** | Output guardrails, structured validation | Yes |

### 2.6 Full-Stack AI Platforms

End-to-end enterprise AI suites that bundle data integration, AI model development, automation, and domain-specific applications.

| Platform | Core Focus | Industry Verticals |
|---|---|---|
| **UiPath AI** | AI-powered automation + document understanding | Finance, healthcare, insurance |
| **C3 AI** | Enterprise AI applications (supply chain, CRM, fraud) | Energy, defense, manufacturing |
| **IBM watsonx** | AI studio + data store + governance | Financial services, healthcare |
| **ServiceNow AI** | AI for IT, HR, customer service workflows | IT, telecom, financial services |
| **Pegasystems AI** | Decision and workflow AI | Insurance, banking, healthcare |
| **SAP AI Core** | AI embedded in SAP enterprise workflows | ERP, supply chain, HCM |

---

## 3. Major Platforms Deep Dive

### 3.1 Databricks (Mosaic AI)

**Core offering:** Lakehouse architecture unifying data engineering, data science, and AI. Acquired Mosaic ML in 2023 to add foundation model capabilities.

**Key components:**
- **Unity Catalog:** Centralized governance for data + models + features
- **MLflow:** Open-source model lifecycle management (created by Databricks)
- **Mosaic AI:** Model training, fine-tuning, serving for foundation models
- **Feature Store:** Integrated with Delta Lake for online/offline features
- **Delta Sharing:** Open protocol for cross-org data and model sharing
- **Serverless SQL + AI:** Auto-scaling compute for both analytics and AI

**Best for:** Data-mature organizations with existing Spark/Delta Lake investments. Strong in financial services for unified analytics + AI on governed data.

**Pricing model:** DBUs (Databricks Units) — compute consumption + metered AI serving. Premium tiers add Unity Catalog, serverless, and Mosaic AI.

### 3.2 Dataiku

**Core offering:** Collaborative data science and ML platform emphasizing "everyday AI" — accessible to both coders and non-coders.

**Key components:**
- **Visual ML Designer:** Drag-and-drop pipelines with code-first option
- **AutoML:** Automated model building with explainability
- **Governance:** Model registry, versioning, approval workflows
- **LLM Mesh:** Centralized access to LLMs with usage governance
- **Feature Store:** Reusable features with lineage tracking
- **Plugin ecosystem:** Pre-built connectors and templates

**Best for:** Hybrid teams where data analysts collaborate with data scientists. Strong in regulated industries needing audit trails.

**Pricing model:** Per-user subscription tiers (Designer, Automate, Govern) + compute infrastructure. Enterprise tier includes LLM Mesh and advanced governance.

### 3.3 Google Vertex AI

**Core offering:** Unified AI platform on GCP, integrating data preparation, AutoML, custom training, and foundation model access.

**Key components:**
- **Model Garden:** 150+ foundation models (Gemini, Gemma, Claude, Llama, etc.)
- **Vertex AI Pipelines:** Kubeflow Pipelines-based orchestration
- **AutoML:** Automated model training for tabular, image, text, video
- **Gen AI Studio:** Prompt design, tuning, and evaluation UI
- **Vertex AI Agent Builder:** No-code agent creation with RAG
- **Vertex AI Search:** Enterprise search with grounding
- **Model Registry + Evaluation:** Centralized model management

**Best for:** GCP-native organizations, teams building RAG applications, and organizations needing integrated AI + data platform.

**Pricing model:** Pay-per-use (training compute, prediction, model hosting). Gen AI tokens billed separately. Enterprise contract options available.

### 3.4 Azure AI Studio (formerly Azure AI Foundry)

**Core offering:** Microsoft's unified AI platform with deep Azure ecosystem integration, enterprise-grade security, and Copilot extensibility.

**Key components:**
- **Model Catalog:** 1,600+ models including GPT-4o, Llama, Mistral, Cohere, Phi
- **Azure AI Search:** Enterprise-grade vector search + hybrid + semantic
- **Prompt Flow:** Visual prompt engineering and evaluation
- **AI Content Safety:** Built-in safety filters and guardrails
- **Azure AI Agent Service:** Managed agent runtime
- **Copilot Studio:** Extend Microsoft Copilot with custom data and actions
- **Responsible AI Dashboard:** Fairness, explainability, error analysis

**Best for:** Microsoft-centric organizations, teams building on Azure, Copilot extensibility scenarios.

**Pricing model:** Pay-per-token for models + Azure infrastructure costs. Content Safety and AI Search billed separately. Enterprise agreements available.

### 3.5 AWS Bedrock

**Core offering:** Managed service for building GenAI applications using foundation models via API, with enterprise security and AWS integration.

**Key components:**
- **Model Catalog:** Claude, Llama, Mistral, Cohere, Stable Diffusion, Titan
- **Knowledge Bases:** Fully managed RAG with vector stores
- **Guardrails:** Content policies, topic filters, PII redaction, hallucination detection
- **Agents:** Multi-step task orchestration with action groups
- **Bedrock Studio:** Collaborative GenAI app development
- **Model Evaluation:** Automated quality and safety evaluations
- **Custom Model Import:** Import and serve custom models

**Best for:** AWS-native organizations, teams needing managed RAG with compliance controls, regulated industries.

**Pricing model:** Per-model token pricing + provisioned throughput option. Knowledge Bases incur additional storage and compute costs.

### 3.6 IBM watsonx

**Core offering:** IBM's enterprise AI platform with three pillars — watsonx.ai (studio), watsonx.data (data store), watsonx.governance (governance).

**Key components:**
- **watsonx.ai:** Model training, fine-tuning, prompt lab, 100+ models
- **watsonx.data:** Open data lakehouse based on Iceberg
- **watsonx.governance:** Full model lifecycle governance, risk management, compliance
- **Granite Models:** IBM's open-source LLM family designed for enterprise
- **Model Risk Management:** Alignment with EU AI Act, SR 11-7, MAS FEAT
- **OpenScale:** Production model monitoring with bias detection

**Best for:** Regulated enterprises (especially financial services), multi-cloud or on-prem deployments, organizations with strong governance requirements.

**Pricing model:** Subscription-based tiers per capability. watsonx.governance priced separately. IBM Cloud infrastructure or bring-your-own-cloud.

---

## 4. Vendor Comparison Table

### 4.1 Major Platform Comparison (15 Dimensions)

| Dimension | Databricks | Dataiku | Vertex AI | Azure AI Studio | AWS Bedrock | IBM watsonx |
|---|---|---|---|---|---|---|
| **Category** | Data+ML Platform | Data Science Platform | Cloud AI Platform | Cloud AI Platform | GenAI Service | Full-Stack AI |
| **Data Integration** | Delta Lake, Unity Catalog, Spark | 200+ connectors, visual prep | BigQuery, GCS, Dataproc | Azure SQL, ADLS, Fabric | S3, Glue, Athena, Redshift | Cloud SQL, Db2, S3, Iceberg |
| **Feature Store** | Built-in (Unity Catalog) | Built-in | Vertex AI Feature Store | Azure Feature Store | No native (use SageMaker) | No native |
| **AutoML** | AutoML (via Mosaic AI) | Yes (core feature) | Yes (AutoML tabular/image) | Yes (AutoML via Azure ML) | Limited (SageMaker Canvas) | AutoAI (watsonx.ai) |
| **Custom Training** | Notebooks, Spark, distributed | Visual notebooks, code | Custom training, distributed | Azure ML compute clusters | SageMaker training (separate) | watsonx.ai training |
| **Model Registry** | MLflow (built-in) | Dataiku Model Registry | Vertex AI Model Registry | Azure ML Model Registry | Bedrock model registry | watsonx.ai model registry |
| **Model Serving** | MLflow serving, Mosaic AI | Dataiku API Deployer | Vertex AI Endpoints | Azure Managed Endpoints | Bedrock inference API | watsonx.ai deployment |
| **LLM Access** | Mosaic AI, model catalog | LLM Mesh | 150+ models (Model Garden) | 1,600+ models (Catalog) | Claude, Llama, Mistral, Titan | Granite, Llama, Mistral |
| **RAG Support** | Databricks RAG Studio | LLM Mesh RAG | Vertex AI Agent Builder | AI Search + Prompt Flow | Knowledge Bases | watsonx.ai RAG |
| **Agent Framework** | Mosaic AI Agents | Plugin framework | Agent Builder | Azure AI Agent Service | Bedrock Agents | watsonx Orchestrate |
| **Governance** | Unity Catalog, lineage | Model gov, audit trails | Vertex AI Governance | Responsible AI Dashboard | Guardrails (content safety) | watsonx.governance |
| **Explainability** | SHAP, feature importance | Built-in ML interpretability | Vertex AI Explainable AI | Responsible AI interpretability | SageMaker Clarify | AI Factsheets, OpenScale bias |
| **Monitoring** | MLflow monitoring + custom | Dataiku monitoring | Model Monitoring (Vertex) | Azure Monitor + ML | CloudWatch + Bedrock | OpenScale drift/bias |
| **Deployment Model** | Multi-cloud (AWS/Azure/GCP) | Cloud, on-prem, hybrid | GCP only | Azure only | AWS only | Multi-cloud + on-prem |
| **Open Source** | MLflow, Spark, Delta | Partial (DSS core) | Kubeflow, KFP | MLflow integration | None | Granite models (open) |
| **Cost Model** | DBU consumption | Per-user subscription | Pay-per-use + infra | Pay-per-use + infra | Per-token + provisioned | Subscription + compute |
| **Ideal For** | Data-intensive orgs with data eng. teams | Hybrid data+ML teams (coders + analysts) | GCP-native, GenAI builders | Microsoft-centric, Copilot | AWS-native, quick GenAI | Regulated enterprise, governance-first |

### 4.2 MLOps & LLMOps Platform Comparison

| Dimension | MLflow | Kubeflow | Weights & Biases | LangSmith | Arize AI | Helicone |
|---|---|---|---|---|---|---|
| **Type** | MLOps platform | MLOps platform | Experiment tracking | LLMOps | LLM Observability | LLM Gateway |
| **Experiment Tracking** | Yes (core) | Via KFP metadata | Yes (core) | Yes (traces) | No | No |
| **Model Registry** | Yes (core) | Via KF Serving | Yes (W&B Registry) | No (model focus) | No | No |
| **LLM Tracing** | Yes (MLflow Tracing) | No | Yes (Weave) | Yes (core) | Yes (core) | Yes (core) |
| **Prompt Mgmt** | Yes (MLflow Prompts) | No | Yes (Weave) | Yes (Hub) | No | No |
| **Evaluation** | MLflow Evaluate | No | W&B Evaluate | LangSmith Eval | Arize Eval | No |
| **Cost Tracking** | No | No | No | Limited | No | Yes (core) |
| **Guardrails** | No | No | No | No | No | Yes |
| **Deploy** | Self-hosted / Databricks | Kubernetes | Cloud / Self-hosted | Cloud only | Cloud / Self-hosted | Cloud / Self-hosted |
| **License** | Apache 2.0 | Apache 2.0 | Proprietary | Proprietary | Proprietary + MIT (Phoenix) | MIT |

### 4.3 Generative AI Framework Comparison

| Dimension | LangChain | LlamaIndex | Haystack | Semantic Kernel |
|---|---|---|---|---|
| **Primary Focus** | Agent/chains/routing | Data indexing & retrieval | Document QA & search pipelines | Enterprise AI for Microsoft |
| **RAG Quality** | Good (many integrations) | Excellent (best data connectors) | Excellent (native RAG) | Good (Azure AI Search) |
| **Agent Support** | Excellent (LangGraph) | Good (agent support added) | Good (agent pipelines) | Good (function calling) |
| **Custom Tools** | Excellent | Good | Good | Excellent (C# + Azure) |
| **Learning Curve** | Steep (flexibility) | Moderate | Moderate | Moderate (MS-focused) |
| **Ecosystem** | Largest (LangSmith, Hub) | Growing (LlamaCloud) | Mature (deepset Cloud) | Microsoft-focused |
| **Production Maturity** | High | Medium-High | High | High |
| **Language Support** | Python, JS | Python, TS | Python, Node | C#, Python, Java |
| **Enterprise Support** | Via LangSmith | Via LlamaCloud | Via deepset Cloud | Via Microsoft |

---

## 5. Platform Evaluation Criteria

When evaluating enterprise AI platforms, organizations should assess across seven key dimensions.

### 5.1 Data Readiness

- **Data volume & variety:** Can the platform handle your data scale (terabytes to petabytes)?
- **Data sources:** Number of pre-built connectors for your data sources (databases, data lakes, APIs, streaming)
- **Data quality:** Built-in profiling, validation, and cleaning tools
- **Feature engineering:** Feature store capabilities, feature versioning, online/offline serving
- **Data lineage:** End-to-end traceability from source to model prediction

### 5.2 ML Ops Maturity

- **Experiment tracking:** Can you compare 100s of experiments with metrics, params, and artifacts?
- **Pipeline automation:** DAG-based pipelines with scheduling and trigger support
- **CI/CD for ML:** Model deployment pipelines with staging, canary, and rollback
- **Model registry:** Versioning, stage transitions (staging → production), metadata search
- **Online serving:** Latency, throughput, autoscaling for real-time inference
- **Batch inference:** Scheduled batch scoring with cost optimization

### 5.3 Security & Compliance

- **RBAC:** Fine-grained access control by workspace, model, experiment, dataset
- **Data encryption:** Encryption at rest and in transit; BYOK support
- **Audit trails:** Immutable logs of all actions for regulatory reporting
- **Data residency:** Region restrictions, sovereign cloud support
- **Compliance certifications:** SOC 2, ISO 27001, HIPAA, PCI DSS, FedRAMP
- **PII detection:** Built-in PII redaction in data and model I/O

### 5.4 Team Skills

| Team Profile | Recommended Platform | Rationale |
|---|---|---|
| Python-native data scientists | Databricks, MLflow + SageMaker | Code-first, notebook workflow |
| Mixed (R, Python, SQL) | Dataiku, Vertex AI | Visual + code flexibility |
| .NET / Microsoft shop | Azure AI Studio, Semantic Kernel | Native C# integration |
| Data engineers focus | Databricks, Spark-based platforms | Delta Lake, ETL strength |
| Business analysts + DS | Dataiku, H2O Driverless AI | Low-code + AutoML |
| ML research team | W&B + MLflow + custom training | Max flexibility |

### 5.5 Existing Cloud Investment

- **AWS-native:** SageMaker + Bedrock is the path of least resistance
- **Azure-native:** Azure AI Studio + ML is the natural choice; Copilot extensibility bonus
- **GCP-native:** Vertex AI with BigQuery and Gemini integration
- **Multi-cloud:** Databricks, Dataiku, or IBM watsonx for abstraction
- **On-prem / hybrid:** IBM watsonx or Databricks on private cloud

### 5.6 Governance Needs

| Governance Need | Required Platform Feature |
|---|---|
| Model risk management (SR 11-7) | Model documentation, validation tracking, approval workflows |
| EU AI Act compliance | Risk classification, transparency docs, human oversight |
| MAS FEAT compliance | Fairness monitoring, explainability, accountability logs |
| Bias/fairness monitoring | Automated bias detection, demographic parity tracking |
| Drift detection | Data drift, concept drift, prediction drift alerts |
| Explainability | SHAP, LIME, partial dependence plots, feature attribution |
| Full audit trail | Immutable activity log, model lineage, data provenance |

### 5.7 Scalability

- **Training scale:** Support for distributed training (multi-node/multi-GPU)
- **Inference scale:** Horizontal autoscaling, GPU/CPU inference, response caching
- **Model count:** Can the platform manage 10, 100, or 1,000+ production models?
- **Concurrent users:** Multi-tenant isolation with workload management
- **Global deployment:** Multi-region, edge inference, CDN-accelerated serving

---

## 6. Architecture Patterns

### 6.1 Core Architecture: Feature Store + Model Registry + Serving Layer

```
                    ┌─────────────────────────────────────────┐
                    │           Enterprise AI Platform         │
                    │                                         │
  ┌─────────┐  ┌───▼────┐  ┌──────────┐  ┌───────────┐  ┌────▼─────┐
  │Data     │  │Feature │  │Model     │  │Serving    │  │Monitoring│
  │Pipeline │──►Store   │──►Registry  │──►Layer     │──►& Alerting│
  │(Batch/  │  │(Online │  │(Versioned│  │(Batch/    │  │(Drift,   │
  │ Stream) │  │+Offline│  │+Metadata)│  │Realtime)  │  │Perf, Bias)│
  └─────────┘  └────────┘  └──────────┘  └───────────┘  └──────────┘
       │            │             │              │              │
       │            │             │              │              │
  ┌────▼────────────▼─────────────▼──────────────▼──────────────▼────┐
  │                    Governance & Security Layer                    │
  │      Unity Catalog / DataHub / Collibra + RBAC + Audit Logs      │
  └───────────────────────────────────────────────────────────────────┘
```

**Integration points:**
- **Feature Store** serves training features (offline) and inference features (online via low-latency DB like Redis, DynamoDB)
- **Model Registry** stores artifacts, metadata, metrics, and stage transitions
- **Serving Layer** pulls the model artifact from the registry, joins with online features, runs inference

### 6.2 RAG Pattern on Enterprise Platforms

```
User Query
    │
    ▼
┌──────────────────────────────────────────────────┐
│              Enterprise AI Gateway                │
│  ┌─────────┐  ┌────────┐  ┌──────────────────┐   │
│  │Auth +   │  │PII     │  │Query             │   │
│  │Rate     │──►Redact  │──►Understanding     │   │
│  │Limiting │  │Gateway │  │(Multi-intent)    │   │
│  └─────────┘  └────────┘  └────────┬─────────┘   │
└──────────────────────────────────────────────────┘
                                      │
    ┌─────────────────────────────────┼─────────────────────┐
    │           Orchestrator          │                     │
    │  ┌──────────────────────────────▼──────────────┐      │
    │  │         Query Router                        │      │
    │  │  ┌────────────────┐  ┌──────────────────┐   │      │
    │  │  │ Hallucination  │  │ Intent           │   │      │
    │  │  │ Detection Gate │  │ Classification   │   │      │
    │  │  └────────────────┘  └────────┬─────────┘   │      │
    │  └───────────────────────────────┼──────────────┘      │
    └──────────────────────────────────┼─────────────────────┘
                                       │
                    ┌──────────────────┼──────────────────┐
                    ▼                  ▼                   ▼
          ┌─────────────────┐  ┌────────────────┐  ┌──────────────┐
          │ Vector Search   │  │ Structured    │  │ Multi-Model  │
          │ (RAG Retrieval) │  │ Data Query    │  │ Reasoning    │
          │                 │  │ (SQL/Tabular)  │  │ (Ensemble)   │
          │ Embedding Model │  │ Enterprise    │  │ LLM-as-Judge │
          │ + Vector DB     │  │ Warehouse     │  │ + Consensus  │
          └────────┬────────┘  └───────┬────────┘  └──────┬───────┘
                   │                   │                   │
                   └───────────────────┼───────────────────┘
                                       │
                              ┌────────▼────────┐
                              │  Context        │
                              │  Assembly +      │
                              │  Re-ranking      │
                              └────────┬────────┘
                                       │
                              ┌────────▼────────┐
                              │  LLM Generation  │
                              │  (Grounded)      │
                              └────────┬────────┘
                                       │
                              ┌────────▼────────┐
                              │  Output Guardrails│
                              │  (PII check,     │
                              │   Toxicity filter,│
                              │   Policy check)  │
                              └────────┬────────┘
                                       │
                                     Response
```

**Enterprise RAG Notes:**
- **Chunking strategy:** Semantic chunking vs. fixed-size; metadata tagging for source tracing
- **Hybrid search:** Keyword (BM25) + vector search + re-ranking for enterprise accuracy
- **Access control:** Document-level permissions enforced at retrieval (Azure AI Search, Pinecone namespaces)
- **Grounding:** Every response must cite source documents (enterprise compliance requirement)
- **Caching:** Response caching for identical queries (cost savings)

### 6.3 Agent Orchestration Pattern

```
                    ┌──────────────────────────────────┐
                    │        Agent Orchestrator         │
                    │  (LangGraph / Semantic Kernel /   │
                    │   Bedrock Agents / Vertex Agents) │
                    └────────────────┬─────────────────┘
                                     │
         ┌───────────────────────────┼───────────────────────────┐
         │                           │                           │
    ┌────▼────┐               ┌──────▼──────┐             ┌─────▼────┐
    │ Planner │               │ Router      │             │ Evaluator│
    │ Agent   │               │ Agent       │             │ Agent    │
    │ (Create │               │ (Direct     │             │ (Judge   │
    │  Plan)  │               │  Sub-tasks) │             │  Output) │
    └────┬────┘               └──────┬──────┘             └────┬─────┘
         │                           │                         │
         ▼                           ▼                         ▼
    ┌──────────────────────────────────────────────────────────────┐
    │                    Worker Agent Pool                           │
    │                                                               │
    │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌───────────────┐ │
    │  │ Research │  │ Code     │  │ Data     │  │ Compliance    │ │
    │  │ Agent    │  │ Agent    │  │ Analysis │  │ Check Agent   │ │
    │  │ (Web+KB) │  │ (Write/  │  │ Agent    │  │ (Policy       │ │
    │  │          │  │  Execute)│  │ (SQL+Vis)│  │  Validation)  │ │
    │  └──────────┘  └──────────┘  └──────────┘  └───────────────┘ │
    │                                                               │
    │  ┌──────────┐  ┌──────────┐  ┌────────────────────────────┐  │
    │  │ Approval │  │ Human-   │  │ Tool Executor              │  │
    │  │ Gate     │  │ In-Loop  │  │ (APIs, DBs, SaaS, Internal)│  │
    │  │ (Stop)   │  │ (Review) │  └────────────────────────────┘  │
    │  └──────────┘  └──────────┘                                 │
    └──────────────────────────────────────────────────────────────┘
```

**Banking-specific agent considerations:**
- No autonomous monetary transactions without human approval
- Every agent action must be logged for audit
- Worker agents cannot access production customer data without explicit PII gateway
- Approval gates required for: fund transfers, account changes, credit decisions
- Compliance check agent validates every output against regulatory policies

### 6.4 Multi-Model Gateway Pattern

```
                            ┌─────────────┐
                            │ API Gateway │
                            │ (Uniform    │
                            │  Endpoint)  │
                            └──────┬──────┘
                                   │
                            ┌──────▼──────┐
                            │ Router:     │
                            │ Model +     │
                            │ Provider    │
                            └──────┬──────┘
                                   │
         ┌─────────────────────────┼─────────────────────────┐
         │                         │                         │
    ┌────▼────┐              ┌─────▼─────┐             ┌─────▼────┐
    │ Primary │              │ Fallback  │             │ Cost-    │
    │ Model   │              │ Model(s)  │             │ Optimized│
    │ (GPT-4o)│              │ (Claude,  │             │ (Llama,  │
    │         │              │  Gemini)  │             │  Mistral)│
    └────┬────┘              └─────┬─────┘             └─────┬────┘
         │                         │                         │
         └─────────────────────────┼─────────────────────────┘
                                   │
                            ┌──────▼──────┐
                            │ Response    │
                            │ Validator   │
                            │ (Format +   │
                            │  Safety)    │
                            └─────────────┘
```

**Enterprise benefits:**
- Provider failover — if one model provider is down, auto-route to another
- Cost optimization — use cheaper open-source models for simple tasks, frontier models for complex
- Vendor negotiation leverage — measurable usage across providers
- Latency-based routing — small models for fast responses, large models for complex reasoning
- Compliance isolation — regulated workloads routed to on-prem models, experimentation routed to cloud

### 6.5 Hybrid On-Prem/Cloud Deployment for Banking

```
┌───────────────────────────────┐    ┌─────────────────────────────────┐
│         On-Prem (Regulated)   │    │        Cloud (Experimentation)   │
│                               │    │                                 │
│  ┌─────────────────────────┐  │    │  ┌───────────────────────────┐  │
│  │   Sensitive Data Zone   │  │    │  │   R&D Sandbox             │  │
│  │   - Customer PII         │  │    │  │   - Notebooks             │  │
│  │   - Account data         │  │    │  │   - Experiment tracking   │  │
│  │   - Transaction history  │  │    │  │   - AutoML               │  │
│  │   - Credit risk models   │  │    │  │   - LLM prototyping       │  │
│  └─────────────────────────┘  │    │  └───────────────────────────┘  │
│                               │    │                                 │
│  ┌─────────────────────────┐  │    │  ┌───────────────────────────┐  │
│  │   Model Training Zone   │  │    │  │   Pre-production          │  │
│  │   - On-prem GPU cluster  │──┼────┼──►   - Validate on cloud    │  │
│  │   - Federated learning   │  │    │  │   - Benchmarking          │  │
│  │   - DP-SGD (privacy)     │  │    │  │   - Stress testing        │  │
│  └─────────────────────────┘  │    │  └───────────────────────────┘  │
│                               │    │                                 │
│  ┌─────────────────────────┐  │    │  ┌───────────────────────────┐  │
│  │   Model Serving Zone    │  │    │  │   LLM Gateway (Cloud)     │  │
│  │   - Regulated inference  │  │    │  │   - Frontier models       │  │
│  │   - Risk model scoring   │  │    │  │   - GPU-on-demand         │  │
│  │   - Compliance-approved  │  │    │  │   - RAG vector DB         │  │
│  └────────────┬────────────┘  │    │  └───────────────────────────┘  │
│               │               │    │                                 │
└───────────────┼───────────────┘    └─────────────────────────────────┘
                │                                            │
                │              ┌─────────────────┐           │
                │              │  PII Redaction  │           │
                └──────────────►    Gateway      │◄──────────┘
                               │  (Data Masking  │
                               │   + Tokenization│
                               │   + Anonymization)│
                               └────────┬────────┘
                                        │
                               ┌────────▼────────┐
                               │  Model Serving   │
                               │  (Production     │
                               │  Inference)      │
                               └─────────────────┘
```

**Key design principles:**
1. **Data never leaves the regulated zone without PII redaction**
2. **Models trained on-prem can be promoted to cloud-only after anonymization validation**
3. **Cloud models handle non-personal workloads; on-prem models handle regulated workloads**
4. **All cross-zone traffic goes through the PII redaction gateway**
5. **Audit logs capture every cross-zone data transfer**

---

## 7. Enterprise AI Governance

### 7.1 Model Risk Management Framework

Enterprise AI governance is not optional — it is increasingly mandated by regulation. A robust model risk management (MRM) framework must span the full model lifecycle.

| Stage | Governance Activity | EU AI Act | MAS FEAT |
|---|---|---|---|
| **Scoping** | Risk tier classification (high/low/minimal) | Required (Art. 6-7) | Expected |
| **Data** | Bias audit, data lineage, consent verification | Required (Art. 10) | FEAT: F1-F2 |
| **Development** | Documentation, reproducibility, explainability | Required (Art. 11-13) | FEAT: T1-T4 |
| **Validation** | Independent validation, stress testing, fairness testing | Expected | FEAT: A1-A2 |
| **Deployment** | Approval gates, rollback plan, monitoring setup | Required (Art. 14-16) | FEAT: A3-A4 |
| **Monitoring** | Drift detection, performance decay, bias emergence | Required (Art. 15) | FEAT: E1-E3 |
| **Retirement** | Model decommissioning, data retention, audit archive | Expected | FEAT: T5 |

### 7.2 MAS FEAT Principles in Practice

The Monetary Authority of Singapore's FEAT principles (Fairness, Ethics, Accountability, Transparency) govern AI use in Singapore's financial sector. Here's how they map to platform capabilities:

**Fairness (F1-F2):**
- **F1:** AI systems should treat customers fairly — **Platform feature:** Bias detection across demographic groups
- **F2:** Decisions should be based on relevant, accurate data — **Platform feature:** Data quality checks, feature validation

**Ethics (E1-E3):**
- **E1:** AI should be used responsibly — **Platform feature:** Usage limits, ethical guidelines in guardrails
- **E2:** Customer data should be protected — **Platform feature:** PII redaction, data encryption, audit logs
- **E3:** AI outcomes should be explainable — **Platform feature:** SHAP/LIME explainability, model cards

**Accountability (A1-A4):**
- **A1:** Clear ownership of AI decisions — **Platform feature:** Model ownership metadata, approval workflows
- **A2:** Internal controls and risk management — **Platform feature:** Stage-gated model promotion, validation tracking
- **A3:** Regular review and validation — **Platform feature:** Scheduled model evaluation jobs
- **A4:** Human oversight for high-impact decisions — **Platform feature:** Human-in-the-loop approval gates

**Transparency (T1-T5):**
- **T1:** Disclosure of AI use to customers — **Platform feature:** Model documentation, transparency reports
- **T2:** Explainability of AI decisions — **Platform feature:** Feature attribution, counterfactual explanations
- **T3:** Documentation of design and development — **Platform feature:** Auto-generated model cards, experiment lineage
- **T4:** Communication of AI limitations — **Platform feature:** Confidence scores, uncertainty estimation
- **T5:** Audit trail for decisions — **Platform feature:** Immutable audit logs, retention policies

### 7.3 Model Monitoring & Drift Detection

| Drift Type | What It Detects | Detection Method | Remediation |
|---|---|---|---|
| **Data drift** | Changes in input distribution | Statistical tests (KS, JS divergence, PSI) | Retrain on new data |
| **Concept drift** | Relationship X→Y changes | Performance metric decay | Retrain, feature engineering |
| **Prediction drift** | Changes in model output distribution | Output statistics monitoring | Investigate root cause |
| **Label drift** | Changes in ground truth distribution | Delayed ground truth analysis | Re-labeling |
| **Embedding drift** | Changes in representation space | Distance metrics on embeddings | Re-embed, update index |

### 7.4 Bias & Fairness Monitoring

| Metric | What It Measures | Threshold (Banking) |
|---|---|---|
| **Demographic parity** | Equal positive rate across groups | < 0.1 difference |
| **Equal opportunity** | Equal TPR across groups | < 0.1 difference |
| **Disparate impact** | Ratio of favorable outcomes | 0.8–1.25 |
| **Individual fairness** | Similar predictions for similar individuals | L1 distance < threshold |
| **Counterfactual fairness** | Prediction unchanged if group attribute changed | Consistent prediction |

### 7.5 Explainability Techniques

| Technique | Scope | Model Type | Enterprise Use |
|---|---|---|---|
| **SHAP** | Global + local | Any (model-agnostic) | Mandatory for credit risk |
| **LIME** | Local | Any (model-agnostic) | Quick per-prediction explanations |
| **Partial dependence** | Global | Any | Feature importance at aggregate |
| **Feature attribution** | Local | Tree-based, linear, NN | Model card generation |
| **Counterfactual** | Local | Any | "What if" analysis for applications |
| **Integrated Gradients** | Local | Neural networks | Deep learning explainability |

### 7.6 Audit Trails

Every enterprise AI platform must produce immutable audit logs covering:

| Event | Required Fields | Retention |
|---|---|---|
| Model creation | User, timestamp, dataset, params | 7 years (banking) |
| Training run | Hyperparameters, metrics, data version | 7 years |
| Model promotion | Approver, stage change, rationale | 7 years |
| Inference | Input hash, output, model version, timestamp | 3–5 years |
| Data access | User, dataset, operation, timestamp | 7 years |
| Governance action | Admin user, action, reason, timestamp | 7 years |
| Model retirement | Approver, archive location, backup | 7 years |

---

## 8. Vendor Lock-In Risk Analysis

### 8.1 Risk Matrix

| Lock-In Vector | High-Risk Vendors | Low-Risk Vendors | Mitigation Strategy |
|---|---|---|---|
| **Proprietary model format** | SageMaker, Azure ML | MLflow, ONNX, HuggingFace | Export to ONNX/MLflow format |
| **Custom pipeline DSL** | Kubeflow (KFP SDK), Dataiku | MLflow, Airflow, ZenML | Use standard orchestrators |
| **Closed model access** | OpenAI, Anthropic | Open-source via open models | Multi-model gateway abstraction |
| **Proprietary feature store** | SageMaker, Vertex AI | Feast (open source), Redis | Feast-based abstraction layer |
| **Vendor-specific telemetry** | W&B, Neptune | MLflow, Prometheus | OpenTelemetry standard |
| **Cloud-specific services** | Bedrock, Vertex, Azure AI | Cross-cloud platforms | Abstraction layer (e.g., Databricks) |

### 8.2 Open Standards to Reduce Lock-In

| Standard | Purpose | Supported By |
|---|---|---|
| **MLflow Model Format** | Model packaging & serving | Databricks, Azure ML, Databricks, OSS |
| **ONNX** | Cross-framework model interchange | All cloud providers, PyTorch, TF |
| **OpenAPI** | API specification | All inference endpoints |
| **OpenTelemetry** | Observability & tracing | MLflow, Arize, Langfuse |
| **Delta Lake / Iceberg** | Open table formats | Databricks, AWS, GCP, Azure |
| **Kubernetes** | Container orchestration | Kubeflow, MLflow, Flyte, ZenML |
| **Feast** | Open-source feature store | Any compute platform |
| **OpenAI API standard** | LLM API compatibility | All major providers (compatible APIs) |
| **Docker + OCI** | Container standardization | All deployment targets |

### 8.3 Vendor Lock-In Risk Assessment by Platform

| Platform | Overall Lock-In Risk | Data Lock-In | Model Lock-In | Infra Lock-In |
|---|---|---|---|---|
| **Databricks** | Medium | Medium (Delta Lake is open, Unity Catalog proprietary) | Low (MLflow standard) | Low (multi-cloud) |
| **Dataiku** | High | High (proprietary project format) | Medium (can export models) | Medium |
| **Vertex AI** | High | High (BigQuery, GCP services) | Low (MLflow/ONNX export) | High (GCP-only) |
| **Azure AI** | High | High (Azure services) | Low (MLflow/ONNX export) | High (Azure-only) |
| **Bedrock** | High | High (S3, AWS services) | Low (ONNX export) | High (AWS-only) |
| **IBM watsonx** | Medium | Medium (Iceberg-based) | Low (open Granite models) | Low (multi-cloud + on-prem) |
| **MLflow** | Low | None | Low (open format) | None (any infra) |

**Key recommendation:** Design your architecture with an **abstraction layer** at every integration point. Use MLflow model format for portability. Use Feast for features. Use OpenTelemetry for monitoring. Use Kubernetes for deployment. This ensures you can switch platform components without rebuilding your entire AI stack.

---

## 9. Deployment Models for Banking

### 9.1 The Three-Zone Banking Deployment Model

| Zone | Characteristics | Examples | Platform Fit |
|---|---|---|---|
| **Zone 1: On-Prem (Regulated)** | Sensitive customer data, risk models, compliance-critical | Credit scoring, AML, fraud detection | IBM watsonx, Databricks on private cloud |
| **Zone 2: Cloud (Experimentation)** | Anonymized data, research, model development | Feature engineering, LLM evaluation, AutoML | Vertex AI, Azure AI, Bedrock, SageMaker |
| **Zone 3: Hybrid (Production Serving)** | PII-redacted inference, monitoring, feedback | Customer service RAG, internal knowledge search | Multi-model gateway + PII gateway |

### 9.2 On-Prem Deployment (Zone 1)

**When to use:**
- Customer personal data processing (PII, financial records)
- Models with direct financial impact (credit decisions, trading)
- Regulatory mandates requiring data sovereignty
- High-frequency trading models requiring ultralow latency

**Platform options:**
- **IBM watsonx:** Designed for regulated on-prem deployment; full governance suite
- **Databricks on private cloud:** Unity Catalog and MLflow on customer-managed infrastructure
- **H2O.ai / Dataiku on-prem:** Self-managed deployment options
- **Custom stack:** MLflow + Kubernetes + on-prem NVIDIA GPUs

**Considerations:**
- Higher upfront infrastructure cost (GPU clusters, storage, networking)
- Team needs on-call operations skills for infrastructure management
- Limited access to frontier models (need on-prem open-source alternatives)

### 9.3 Cloud for Experimentation (Zone 2)

**When to use:**
- Rapid prototyping and model development
- Access to frontier models (GPT-4o, Claude 4, Gemini 2.0)
- AutoML and hyperparameter tuning
- Training large models requiring elastic GPU access

**Platform options:**
- **Vertex AI:** Gen AI Studio for prompt engineering, AutoML for baseline models
- **Azure AI Studio:** Prompt Flow, model catalog, responsible AI dashboard
- **AWS Bedrock + SageMaker:** GenAI + traditional ML on unified AWS
- **Databricks:** Notebook-driven development with Unity Catalog governance

**Data restrictions:**
- Only anonymized/synthetic data allowed in cloud experimentation
- PII must be redacted before any cloud upload
- Model artifacts trained on sensitive data cannot remain on cloud

### 9.4 Hybrid Model Serving with PII Redaction Gateway (Zone 3)

**Architecture:**

```
                    ┌─────────────────────────────────────┐
                    │         API Gateway / Load Balancer │
                    └──────────────┬──────────────────────┘
                                   │
                    ┌──────────────▼──────────────────────┐
                    │       PII Redaction Gateway          │
                    │                                     │
                    │  ┌──────────┐  ┌──────────────────┐ │
                    │  │ Scan:    │  │ Transform:       │ │
                    │  │ Regex    │  │ Mask, Tokenize,  │ │
                    │  │ NER      │  │ Anonymize,       │ │
                    │  │ ML Model │  │ Pseudonymize,    │ │
                    │  │ (PII)    │  │ Redact           │ │
                    │  └──────────┘  └──────────────────┘ │
                    │                                     │
                    │  ┌──────────────────────────────┐   │
                    │  │ Policy Engine:               │   │
                    │  │ Data classification -> action│   │
                    │  └──────────────────────────────┘   │
                    └────────────────┬────────────────────┘
                                     │
         ┌───────────────────────────┼───────────────────────────┐
         │                           │                           │
    ┌────▼────┐               ┌──────▼──────┐             ┌─────▼────┐
    │On-Prem  │               │  Multi-Model │             │  Batch   │
    │Model    │               │  Gateway    │             │  Scorer  │
    │(Risk )  │               │  (Cloud)    │             │          │
    └─────────┘               └─────────────┘             └──────────┘
```

### 9.5 Banking-Specific Platform Requirements

| Requirement | Why It Matters | Platform Must Support |
|---|---|---|
| **Model explainability** | Regulatory compliance (SR 11-7, BCBS 239) | SHAP, LIME, feature attribution |
| **Human-in-the-loop** | Regulatory requirement for credit decisions | Approval gates, manual override |
| **Data sovereignty** | Customer data must stay in Singapore | Local region deployment, sovereign cloud |
| **PII redaction** | Protect customer data in AI pipelines | Built-in PII detection/masking |
| **Audit trails** | Regulatory examinations | Immutable logs, 7-year retention |
| **DR/BCP** | Business continuity requirements | Multi-AZ, multi-region failover |
| **Model validation** | Independent validation required | Stage-gated promotion, validation reports |
| **Bias monitoring** | MAS FEAT Fairness principle | Demographic parity, equal opportunity |
| **Cost attribution** | Chargeback to business units | Model-level cost tracking, API metering |
| **Version control** | Model lineage and reproducibility | Experiment tracking, model registry, dataset versioning |

---

## 10. Key Trends

### 10.1 AI Agents & Platform Engineering for AI

The rise of AI agents is driving a new platform engineering discipline within enterprises. Teams are building internal developer platforms (IDPs) that provide:

- **Agent runtime infrastructure** — managed execution, state persistence, observability
- **Tool registries** — centrally governed, security-reviewed tool definitions
- **Agent templates** — pre-built patterns for common workflows (customer support, coding, data analysis)
- **Approval frameworks** — human-in-the-loop workflows for agent actions
- **MCP (Model Context Protocol)** — standardized tool integration protocol

**Why it matters:** Organizations that invest in platform engineering for AI will ship agentic applications 3-5x faster than those building from scratch per agent.

### 10.2 LLMOps Maturing

LLMOps is evolving from ad-hoc practices into a structured discipline with defined tooling:

| Phase | 2023 | 2024 | 2025–2026 |
|---|---|---|---|
| **Prompt Mgmt** | Hardcoded strings | Prompt versioning | Prompt registries with A/B testing |
| **Evaluation** | Manual spot checks | LLM-as-judge | Automated eval suites, regression tests |
| **Tracing** | Console.log | Basic traces | OpenTelemetry, cost-per-trace attribution |
| **Guardrails** | None | Basic content filters | Enterprise policy-as-code, audit trail |
| **Cost Mgmt** | Manual tracking | Token counters | Budget alerts, provider routing, caching |

**Key platforms to watch:** LangSmith, MLflow (LLM features), Arize Phoenix, Langfuse — these are converging toward a unified LLMOps stack.

### 10.3 Open Source AI Platforms

The open-source ecosystem is maturing rapidly, offering credible alternatives to proprietary platforms:

| Layer | Open Source Alternative | Proprietary Equivalent |
|---|---|---|
| **Foundation models** | Llama 4, Mistral, Gemma, Granite, DeepSeek | GPT-4o, Claude 4, Gemini 2.0 |
| **Fine-tuning** | Unsloth, Axolotl, Lamini | SageMaker, Vertex AI tuning |
| **RAG stack** | Qdrant/Milvus + LangChain + Ollama | Vertex AI Search, Bedrock KB |
| **LLM gateway** | Helicone, Portkey (OSS), MLflow Gateway | Azure API Management |
| **Agent framework** | LangGraph, AutoGen, CrewAI | Vertex Agents, Bedrock Agents |
| **LLMOps** | MLflow, Langfuse, Phoenix | LangSmith, Datadog LLM Obs |

**Why it matters:** Open source reduces vendor lock-in, lowers costs, and enables on-prem deployment for regulated industries. The trade-off is higher operational overhead.

### 10.4 AI Governance Regulation

The regulatory landscape is rapidly evolving and will accelerate further:

| Regulation | Region | Key Requirements | Effective |
|---|---|---|---|
| **EU AI Act** | EU | Risk classification, transparency, human oversight, conformity assessment | 2025–2027 (phased) |
| **MAS FEAT** | Singapore | Fairness, ethics, accountability, transparency | 2018 (updated 2020) |
| **SR 11-7** | US | Model risk management, validation, governance | Ongoing |
| **NYC Local Law 144** | US (NYC) | Bias audit for hiring AI | 2023 |
| **China AI Regulation** | China | Algorithm registration, content control | 2023–2024 |
| **Canada AIDA** | Canada | Impact assessment, transparency, bias mitigation | Proposed |
| **G7 Hiroshima AI Process** | International | Guiding principles for advanced AI systems | 2024 |

**Impact on platform selection:** Governance capabilities are becoming the #1 differentiator in regulated industries. **IBM watsonx.governance** and **Azure AI Studio** currently lead in compliance readiness, but Databricks (Unity Catalog + MLflow) and Vertex AI (Governance Dashboard) are closing the gap quickly.

### 10.5 GenAI Application Platforms

A new category of "GenAI application platforms" is emerging, distinct from traditional MLOps platforms:

- **Dify:** Open-source LLM app builder with visual workflow
- **Flowise:** Low-code LLM orchestration
- **Vertex AI Agent Builder:** Google's managed GenAI app platform
- **Azure AI Studio + Copilot Studio:** Microsoft's GenAI app platform
- **LangSmith + LangGraph:** LangChain's managed GenAI platform

**Convergence trend:** These platforms are absorbing RAG, agent, and monitoring capabilities into a single integrated experience. The distinction between "ML platform" and "GenAI platform" will likely disappear by 2027–2028.

---

## 11. Decision Framework

### 11.1 Organization Maturity Model

| Stage | Characteristics | Recommended Platform Strategy |
|---|---|---|
| **Stage 1: Ad-hoc** | 1–3 data scientists, notebooks, no governance | Start with MLflow + SageMaker/Vertex AI |
| **Stage 2: Centralized** | Dedicated ML team, basic model registry, limited compliance | Databricks or Dataiku + MLflow |
| **Stage 3: Standardized** | Multiple teams, CI/CD for ML, governance committee | Databricks (managed) or Azure AI/Vertex AI |
| **Stage 4: Platform** | AI platform team, self-service, LLMOps, governed gen AI | Multi-cloud: Databricks + Bedrock/Vertex + watsonx.governance |
| **Stage 5: AI-native** | AI everywhere, agentic workflows, composable architecture | Enterprise AI platform + open-source stack + multi-model gateway |

### 11.2 Decision Matrix by Use Case

| Use Case | Team Profile | Recommended Platform | Runner-Up |
|---|---|---|---|
| **Classic ML (tabular)** | Python DS | Databricks + MLflow | SageMaker/Vertex AI |
| **Computer vision** | ML engineers | Vertex AI + AutoML | SageMaker + Rekognition |
| **NLP / text classification** | NLP engineers | Databricks + LlamaIndex | Vertex AI + Gemini |
| **RAG (internal KB)** | Full-stack + DS | Azure AI + AI Search | Vertex AI + Agent Builder |
| **Customer-facing chatbot** | AI engineers | Bedrock + Knowledge Bases | Vertex AI Agent Builder |
| **LLM fine-tuning** | ML engineers | Databricks + Mosaic AI | Azure AI + LoRA |
| **Agentic workflow** | AI + backend | LangChain + Azure AI Agents | Bedrock Agents |
| **Fraud detection** | DS + compliance | IBM watsonx + governance | Databricks + MLflow |
| **Credit scoring** | DS + risk | IBM watsonx (on-prem) + SHAP | Databricks on private cloud |
| **AutoML / baseline** | DS + business analysts | Dataiku (+ H2O) | Vertex AI AutoML |
| **Governance-first** | Compliance + DS | IBM watsonx.governance | Azure AI Studio |
| **Multi-cloud / hybrid** | Platform team | Databricks + watsonx | Custom: MLflow + Kubernetes |

### 11.3 Constraint-Based Decision Tree

```
Starting point: Your organization needs an Enterprise AI Platform
        │
        ├── Are you heavily invested in ONE cloud provider?
        │   ├── Yes, AWS → AWS Bedrock (GenAI) + SageMaker (ML)
        │   ├── Yes, Azure → Azure AI Studio + ML (deep integration)
        │   ├── Yes, GCP → Vertex AI + Gemini (best-in-class)
        │   └── Multi-cloud / no preference → Databricks or IBM watsonx
        │
        ├── Is regulatory compliance your #1 concern?
        │   ├── Yes → IBM watsonx (governance-first) or Azure AI (responsible AI)
        │   └── No → Any, prioritize capabilities over compliance
        │
        ├── Do you need on-premises deployment for regulated data?
        │   ├── Yes → IBM watsonx, Databricks on private cloud, H2O.ai
        │   └── No → Cloud-native options are fine
        │
        ├── What is your team composition?
        │   ├── Mostly coders → Databricks, cloud-native platforms
        │   ├── Mostly analysts → Dataiku (visual + code)
        │   ├── Mix → Dataiku, Databricks (both support)
        │   └── Small team → Dataiku, AutoML-first platforms
        │
        ├── Are you building GenAI or classic ML applications?
        │   ├── GenAI-first → Bedrock, Vertex AI, Azure AI Studio
        │   ├── Classic ML-first → Databricks, SageMaker, Dataiku
        │   └── Both → Databricks (unified), Vertex AI (model garden)
        │
        └── What is your lock-in tolerance?
            ├── Low → MLflow + Kubernetes + open models (long-term investment)
            ├── Medium → Databricks (open formats, multi-cloud)
            └── High → Single-vendor (max speed, accept lock-in)
```

### 11.4 Decision Scorecard Template

| Criterion | Weight | Platform A Score (1-5) | Platform B Score (1-5) | Weighted A | Weighted B |
|---|---|---|---|---|---|
| Data integration | 15% | | | | |
| Model training | 10% | | | | |
| Model serving | 10% | | | | |
| LLM support | 10% | | | | |
| Governance | 15% | | | | |
| Security & compliance | 15% | | | | |
| Ease of use | 5% | | | | |
| Team skills fit | 10% | | | | |
| Cost | 5% | | | | |
| Lock-in risk | 5% | | | | |
| **Total** | **100%** | | | **A** | **B** |

**Scoring guidance:** Use the 15-dimension comparison table in Section 4 as your rubric. Weight governance and compliance higher for regulated industries (banking, insurance, healthcare). Weight LLM support higher if GenAI is your primary use case. Weight data integration higher if you have complex, heterogeneous data sources.

---

## 12. Architecture Diagram

Below is an ASCII architecture diagram of a mature **Enterprise AI Platform** deployment suitable for a financial institution like Crédit Agricole CIB:

```
┌────────────────────────────────────────────────────────────────────────────────────┐
│                       ENTERPRISE AI PLATFORM — REFERENCE ARCHITECTURE                │
│                                  (Banking-Grade)                                      │
├────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                    │
│  ┌──────────────────────────────────────────────────────────────────────────────┐ │
│  │  LAYER 1: DATA & FEATURE INFRASTRUCTURE                                       │ │
│  │                                                                               │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────────────┐ │ │
│  │  │ Data Lake│  │ Data     │  │ Streaming│  │ External │  │ Feature Store  │ │ │
│  │  │ (Delta / │  │ Warehouse│  │ (Kafka / │  │ Data     │  │ (Feast / Unity │ │ │
│  │  │ Iceberg) │  │ (SQL)    │  │ Kinesis) │  │ APIs     │  │ Catalog)       │ │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘  └────────────────┘ │ │
│  │                                                                               │ │
│  │  ┌───────────────────────────────────────────────────────────────────────┐   │ │
│  │  │  Data Governance: Lineage, Quality, PII Detection, Classification     │   │ │
│  │  └───────────────────────────────────────────────────────────────────────┘   │ │
│  └──────────────────────────────────────────────────────────────────────────────┘ │
│                                         │                                          │
│  ┌──────────────────────────────────────────────────────────────────────────────┐ │
│  │  LAYER 2: MODEL DEVELOPMENT & TRAINING                                       │ │
│  │                                                                               │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────────────┐ │ │
│  │  │ Notebook │  │ AutoML   │  │ Custom   │  │ Fine-    │  │ Experiment     │ │ │
│  │  │ Environ. │  │ Pipeline │  │ Training │  │ Tuning   │  │ Tracking (W&B/ │ │ │
│  │  │ (Jupyter)│  │ (Jobs)   │  │ (Distrib)│  │ (LoRA)   │  │ MLflow)        │ │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘  └────────────────┘ │ │
│  │                                                                               │ │
│  │  ┌───────────────────────────────────────────────────────────────────────┐   │ │
│  │  │  ML Orchestration: Airflow / Kubeflow / Vertex Pipelines / Azure ML   │   │ │
│  │  └───────────────────────────────────────────────────────────────────────┘   │ │
│  └──────────────────────────────────────────────────────────────────────────────┘ │
│                                         │                                          │
│  ┌──────────────────────────────────────────────────────────────────────────────┐ │
│  │  LAYER 3: MODEL REGISTRY & GOVERNANCE                                        │ │
│  │                                                                               │ │
│  │  ┌────────────────────────────────────────────────────────────────────────┐  │ │
│  │  │                        Model Registry (MLflow)                          │  │ │
│  │  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐      │  │ │
│  │  │  │ Version │  │ Stage   │  │ Metrics │  │ Artifact│  │ Lineage │      │  │ │
│  │  │  │ Control │  │ Mgmt    │  │ Store   │  │ Store   │  │ Tracking│      │  │ │
│  │  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └─────────┘      │  │ │
│  │  └────────────────────────────────────────────────────────────────────────┘  │ │
│  │                                                                               │ │
│  │  ┌────────────────────────────────────────────────────────────────────────┐  │ │
│  │  │  Governance: Approval Workflows, Compliance Checks, Model Cards,       │  │ │
│  │  │  Risk Tier Classification, Documentation (watsonx.governance / Azure)  │  │ │
│  │  └────────────────────────────────────────────────────────────────────────┘  │ │
│  └──────────────────────────────────────────────────────────────────────────────┘ │
│                                         │                                          │
│  ┌──────────────────────────────────────────────────────────────────────────────┐ │
│  │  LAYER 4: MODEL SERVING & INFERENCE                                          │ │
│  │                                                                               │ │
│  │  ┌───────────────────────────────────────────────────────────────────────┐   │ │
│  │  │                      PII Redaction Gateway                               │ │
│  │  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────────┐  │   │ │
│  │  │  │ Input    │  │ PII Scan │  │ Masking/ │  │ Allow/Deny Decision │  │   │ │
│  │  │  │ Capture  │──►(NER/RE)  │──►Tokenize  │──►(Policy Engine)      │  │   │ │
│  │  │  └──────────┘  └──────────┘  └──────────┘  └──────────────────────┘  │   │ │
│  │  └───────────────────────────────────────────────────────────────────────┘   │ │
│  │                                                                               │ │
│  │  ┌───────────────────────────────────────────────────────────────────────┐   │ │
│  │  │                      Multi-Model Gateway                                 │ │
│  │  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────┐  │   │ │
│  │  │  │ On-Prem  │  │ Cloud    │  │ Provider │  │ Cost-    │  │ A/B    │  │   │ │
│  │  │  │ Models   │  │ Models   │  │ Fallback │  │ Optimized│  │ Testing│  │   │ │
│  │  │  │ (Risk)   │  │ (GenAI)  │  │ Router   │  │ Router   │  │ Router │  │   │ │
│  │  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘  └────────┘  │   │ │
│  │  └───────────────────────────────────────────────────────────────────────┘   │ │
│  │                                                                               │ │
│  │  ┌───────────────────────────────────────────────────────────────────────┐   │ │
│  │  │  Serving Infrastructure:                                               │   │ │
│  │  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────────┐  │   │ │
│  │  │  │ Real-time │  │ Batch    │  │ Streaming│  │ Agent Runtime        │  │   │ │
│  │  │  │ Inference │  │ Scorer   │  │ Inference│  │ (LangGraph / Agent   │  │   │ │
│  │  │  │ (REST/gRPC)│  │ (Scheduler)│  │ (Kafka) │  │  Service)            │  │   │ │
│  │  │  └──────────┘  └──────────┘  └──────────┘  └──────────────────────┘  │   │ │
│  │  └───────────────────────────────────────────────────────────────────────┘   │ │
│  └──────────────────────────────────────────────────────────────────────────────┘ │
│                                         │                                          │
│  ┌──────────────────────────────────────────────────────────────────────────────┐ │
│  │  LAYER 5: OBSERVABILITY & MONITORING                                         │ │
│  │                                                                               │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────────────┐ │ │
│  │  │ Model    │  │ Drift    │  │ Bias &   │  │ Cost     │  │ LLM-specific   │ │ │
│  │  │ Perf.   │──►Detection │──►Fairness  │──►Tracking  │──►Observability   │ │ │
│  │  │ (Latency │  │ (Data/   │  │ Monitor  │  │ (Token/  │  │ (Traces, Eval, │ │ │
│  │  │ /Through)│  │ Concept) │  │          │  │ API Cost)│  │  Prompt Quality)│ │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘  └────────────────┘ │ │
│  │                                                                               │ │
│  │  ┌───────────────────────────────────────────────────────────────────────┐   │ │
│  │  │  Alerting & Dashboard: PagerDuty, Grafana, Custom Dashboards           │   │ │
│  │  └───────────────────────────────────────────────────────────────────────┘   │ │
│  └──────────────────────────────────────────────────────────────────────────────┘ │
│                                         │                                          │
│  ┌──────────────────────────────────────────────────────────────────────────────┐ │
│  │  LAYER 6: SECURITY & COMPLIANCE                                              │ │
│  │                                                                               │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────────────┐ │ │
│  │  │ IAM +    │  │ Network  │  │ Data     │  │ Audit    │  │ Compliance    │ │ │
│  │  │ RBAC     │  │ Security │  │ Encryption│  │ Trail    │  │ Report Gen.   │ │ │
│  │  │ (SSO/SAML)│  │ (VPC/SG)│  │ (BYOK/   │  │ (Immutable│  │ (EU AI Act,   │ │ │
│  │  │          │  │          │  │ KMS)     │  │  Logs)   │  │  MAS FEAT)    │ │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘  └────────────────┘ │ │
│  └──────────────────────────────────────────────────────────────────────────────┘ │
│                                                                                    │
│  ┌──────────────────────────────────────────────────────────────────────────────┐ │
│  │  CROSS-CUTTING: CI/CD for ML + Git Integration + Secrets Mgmt + Backup/DR    │ │
│  └──────────────────────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 13. References

### Industry Reports & Frameworks
1. Gartner Magic Quadrant for Data Science and Machine Learning Platforms (2025)
2. Forrester Wave: MLOps Platforms, Q1 2026
3. EU AI Act — Official Journal of the European Union (2024/1689)
4. MAS FEAT Principles — Monetary Authority of Singapore (2018, updated 2020)
5. SR 11-7 / OCC 2011-12 — Model Risk Management (US Federal Reserve)
6. ThoughtWorks MLOps Platform Comparison — github.com/thoughtworks/mlops-platforms

### Platform Documentation
7. Databricks Mosaic AI — docs.databricks.com/en/mosaic-ai
8. Dataiku Documentation — doc.dataiku.com
9. Google Vertex AI — cloud.google.com/vertex-ai/docs
10. Microsoft Azure AI Foundry — learn.microsoft.com/azure/ai-studio
11. AWS Bedrock — docs.aws.amazon.com/bedrock
12. IBM watsonx — ibm.com/docs/watsonx

### Open Standards
13. MLflow — mlflow.org
14. ONNX — onnx.ai
15. OpenTelemetry — opentelemetry.io
16. Feast (Feature Store) — feast.dev
17. Delta Lake — delta.io
18. Apache Iceberg — iceberg.apache.org

### Architecture & Best Practices
19. Microsoft Azure Architecture Center — RAG patterns, Agent design patterns
20. Google Cloud Architecture Framework — AI/ML design
21. AWS Well-Architected Framework — Machine Learning Lens
22. NIST AI Risk Management Framework (AI RMF 1.0)

---

*This guide was prepared by Jack Liu Shurui, Solution Architect at Crédit Agricole CIB, Singapore. The content reflects industry research and practitioner experience as of July 2026. Platform features, pricing, and capabilities are subject to change. Always validate with current vendor documentation before making procurement decisions.*
