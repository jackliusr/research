# Nutanix Enterprise AI vs Red Hat OpenShift AI — Comprehensive Head-to-Head Comparison

> **Author:** Jack Liu Shurui — Solution Architect at Crédit Agricole CIB, Singapore
> **Context:** Technology Research — Infrastructure / AI-platform series; the dedicated head-to-head between the two enterprise AI platform poles: Nutanix's turnkey AI-infrastructure stack (Nutanix Enterprise AI) and Red Hat's Kubernetes-native MLOps platform (OpenShift AI)
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** Nutanix press releases and product blog; Red Hat press releases and OpenShift AI documentation; The Nutanix Bible (nutanixbible.com); release and CVE coverage (IBM Community, endoflife.date, security advisories); vendor news coverage
> **Last Updated:** August 2026

---

## Table of Contents

1. [Overview — The Two Platforms](#1-overview--the-two-platforms)
2. [Nutanix Enterprise AI — Deep Dive](#2-nutanix-enterprise-ai--deep-dive)
3. [Red Hat OpenShift AI — Deep Dive](#3-red-hat-openshift-ai--deep-dive)
4. [Head-to-Head — Table, Architecture, Ecosystem, Economics](#4-head-to-head--table-architecture-ecosystem-economics)
5. [Selection Guidance — Which for Which Need](#5-selection-guidance--which-for-which-need)
6. [Worked Example — A Mid-Size Bank's AI Platform Selection](#6-worked-example--a-mid-size-banks-ai-platform-selection)
7. [Summary — One Page](#7-summary--one-page)
8. [Glossary](#8-glossary)
9. [Claims Status, References and Disclaimer](#9-claims-status-references-and-disclaimer)

### How to Read This Guide

This is the dedicated head-to-head on **Nutanix Enterprise AI (NAI)** versus **Red Hat OpenShift AI (RHOAI)** — the infrastructure-first AI platform against the Kubernetes-first MLOps platform. It mirrors the structure and rigour of the sibling head-to-head [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md) (Charmed Kubeflow vs OpenShift AI — the other "open Kubeflow" comparison), and it is the product-level counterpart of the portfolio survey in [nutanix_products_guide.md](nutanix_products_guide.md) — which covers the full Nutanix Cloud Platform and its AI products (§8, GPT-in-a-Box). Several sibling guides carry adjacent depth and are cross-referenced inline:

- **The Nutanix portfolio** — [nutanix_products_guide.md](nutanix_products_guide.md) §8 covers GPT-in-a-Box (announced 15 Aug 2023) and the AI portfolio beyond the box; this guide is the dedicated deep-dive on its 2024 successor product, **Nutanix Enterprise AI**, against the Red Hat alternative. Cross-reference heavily — this guide assumes that portfolio context and does not re-derive it.
- **The OpenShift AI landscape** — [openshift_ai_alternatives_guide.md](openshift_ai_alternatives_guide.md) is the whole-alternatives landscape (Kubeflow, Databricks, Anyscale, etc.), where Nutanix appears only as a commercial Kubeflow support vendor; this guide is the focused two-way match-up. Do not confuse the two scopes.
- **The open-Kubeflow head-to-head** — [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md) is the pattern this guide mirrors; it also contains the deep OpenShift AI component tables (Workbenches, DSP, KServe, Model Registry, TrustyAI) that §3 summarises.
- **The on-prem AI playbook** — [on_prem_llm_deployment_guide.md](ai_llm/on_prem_llm_deployment_guide.md) is the practical playbook for the kind of on-prem GPU estate both platforms deliver; [gpu_optimization_guide.md](gpu_optimization_guide.md) covers the GPU-compute layer (NVIDIA GPU Operator, MIG, time-slicing) both platforms ride on; [llm_instruction_tuning_guide.md](ai_llm/llm_instruction_tuning_guide.md) covers the fine-tuning layer neither platform owns out of the box.
- **The sovereignty and cost angles** — [htx_ngine_guide.md](htx_ngine_guide.md) is the sovereign-AI reference where Nutanix is the platform layer (ST Engineering + NVIDIA + Nutanix + Google); [cloud_providers_guide.md](cloud_providers_guide.md) is the cloud-comparison context; [finops_guide.md](finops_guide.md) is the cost lens §4.4's economics mirror; [ml_platforms_comparison_guide.md](ml_platforms_comparison_guide.md) is the broader ML-platforms survey this pair belongs to.

**Note on verification.** This guide was researched in August 2026. Claims are marked **Verified** (confirmed against vendor primary sources or reliable secondary coverage during research), **Reported** (widely reported but not independently confirmed), or **flagged** inline where specifics are approximate or could not be pinned down. The full claims-status table is in §9. Where this guide inherits a fact from a sibling guide (e.g., RHOAI component behaviour from [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md)), it cites the sibling rather than re-verifying.

---

## 1. Overview — The Two Platforms

### 1.1 The Two Platforms

**Nutanix Enterprise AI (NAI)** is Nutanix's enterprise AI platform, launched in **November 2024** (Verified — Nutanix's own May 2025 blog references "the launch of NAI last November"; TechAfrica News reported the launch on 13 Nov 2024). It is the 2024 evolution of **GPT-in-a-Box**, Nutanix's turnkey on-premises generative-AI stack announced **15 August 2023** (Verified — cross-ref [nutanix_products_guide.md](nutanix_products_guide.md) §8.1), and it ships as the AI control plane of **GPT-in-a-Box 2.0**, running on the **Nutanix Kubernetes Platform (NKP)** — the company's Kubernetes distribution, formerly Karbon — over the AHV hypervisor and AOS storage core. NAI's job is to make LLM inference endpoints deployable "in minutes" through a UI, with models and data staying inside the enterprise (or the chosen cloud), and with NVIDIA's inference stack (NIM microservices, AI Enterprise) baked in via the May 2024 Nutanix–NVIDIA collaboration (Verified — press release, 21 May 2024, .NEXT Barcelona).

**Red Hat OpenShift AI (RHOAI)** is Red Hat's enterprise MLOps platform, renamed from **Red Hat OpenShift Data Science (RHODS)** and launched under its current name in **May 2023** (Verified — Red Hat press release, 23 May 2023: "Red Hat OpenShift AI Accelerates Generative AI Adoption Across Hybrid Cloud"). It is the commercial, enterprise-hardened downstream of the **Open Data Hub (ODH)** project — Red Hat's open-source AI/ML platform initiative on OpenShift that began around **2018** (Verified — Red Hat Emerging Technology blog, Sept 2018, describing ODH as a proof-of-concept on the Mass Open Cloud, running on OpenShift and OpenStack). RHOAI packages the Kubeflow-family components — notebooks, pipelines, KServe model serving, plus Red Hat additions (Model Registry, TrustyAI explainability, Kueue scheduling) — as OpenShift Operators managed through a single `DataScienceCluster` custom resource.

**The one-sentence difference:** NAI is an *AI-infrastructure platform* — a turnkey, validated, GPU-plus-software bundle that extends the Nutanix HCI stack upward into model serving; RHOAI is an *MLOps application platform* — a Kubernetes-native, component-rich layer that sits on top of whatever OpenShift cluster you already run (or will buy).

**At a glance:**

| | Nutanix Enterprise AI | OpenShift AI |
|---|---|---|
| **What it is** | Turnkey AI-infrastructure platform (Nov 2024) | Kubernetes-native MLOps platform (May 2023) |
| **Core promise** | LLM endpoints in minutes, sovereign, on your Nutanix estate | Full MLOps lifecycle on OpenShift, enterprise-hardened |
| **Who it's for** | Infra teams, regulated/sovereign shops, Nutanix HCI estates | Platform/data-science teams, OpenShift-standardised enterprises |
| **Watch-out** | Thin MLOps lifecycle; opaque pricing (Flagged) | OpenShift-only; subscription + ops cost (Flagged) |
| **In one line** | The AI layer of the HCI stack | The AI tenant of the K8s platform |

### 1.2 The Comparison Scope

This guide compares the two platforms across five dimensions, mirroring the sibling head-to-head's scope:

| Dimension | What it covers |
|---|---|
| **Positioning** | What problem each platform claims to solve; who it is sold to (IT infrastructure vs data science/platform teams); turnkey appliance vs composable platform |
| **Architecture** | The substrate (Nutanix HCI: AOS/AHV/NKP vs Red Hat OpenShift Kubernetes), control plane, tenancy, and where the AI layer sits |
| **Stack** | The concrete component stack: GPUs, container platform, model serving, RAG, pipelines, notebooks, registry, guardrails |
| **Features** | Model serving, RAG, guardrails, fine-tuning, monitoring, governance — what is built in vs left to the customer |
| **Economics** | Licensing and packaging (Nutanix subscription vs OpenShift + AI add-on), hardware costs, and the approximate all-in cost of a production cluster |

**What this guide is not:** it is not the whole OpenShift AI alternatives landscape (that is [openshift_ai_alternatives_guide.md](openshift_ai_alternatives_guide.md)), and it is not the full Nutanix portfolio survey (that is [nutanix_products_guide.md](nutanix_products_guide.md)). It is the two-way match-up, plus the worked selection example.

### 1.3 The Overview Table

| Aspect | Nutanix Enterprise AI | Red Hat OpenShift AI |
|---|---|---|
| **Vendor** | Nutanix (NASDAQ: NTNX), San Jose, CA | Red Hat (IBM subsidiary), Raleigh, NC |
| **Launched** | November 2024 (Verified); evolves GPT-in-a-Box (Aug 2023) | Renamed/positioned May 2023 (Verified); lineage from OpenShift Data Science and Open Data Hub (2018) |
| **Product type** | AI-infrastructure platform (turnkey, validated stack) | MLOps application platform (Kubernetes-native component layer) |
| **Substrate** | Nutanix Cloud Platform: AOS + AHV + NKP (Kubernetes), NVIDIA GPUs | Red Hat OpenShift (any supported infra: bare metal, vSphere, OpenStack, public cloud, even Nutanix AHV) |
| **Core AI layer** | NVIDIA NIM microservices + NVIDIA AI Enterprise; curated validated models (Hugging Face + NVIDIA) | Kubeflow-family components: Workbenches, Data Science Pipelines, KServe, Training Operator, Model Registry, TrustyAI |
| **Control plane** | NAI UI/APIs on NKP (model endpoints, RAG blueprints, guardrails) | OpenShift Operators + single `DataScienceCluster` CR |
| **Data/GPU story** | Data already on the HCI cluster (Files/Objects/NDB); GPU scheduling on AHV via NKP | Bring-your-own storage (PVC/ODF); NVIDIA GPU Operator, Kueue scheduling |
| **Key partnerships** | NVIDIA (NIM, NeMo, AI Enterprise), Hugging Face | Kubeflow/ODH community, IBM (Granite, RHEL AI/InstructLab), ISV OperatorHub |
| **Typical buyer** | Infrastructure teams in regulated/sovereign enterprises wanting turnkey on-prem AI | Platform/data-science teams in OpenShift-standardised enterprises wanting full MLOps |
| **Economics** | No public list price; Nutanix subscription packaging (Flagged) | OpenShift subscription + AI add-on; ~$12K–$80K+/yr for production clusters (Flagged, approximate) |

**Read the table as a fork:** if your starting point is *infrastructure* ("I need GPU compute + model serving, quickly, on-prem"), NAI is the fast path. If your starting point is *platform* ("I need an MLOps lifecycle for many teams, on OpenShift"), RHOAI is the full path. §4 makes the fork explicit.

---

## 2. Nutanix Enterprise AI — Deep Dive

### 2.1 The Product — AI-Infrastructure Platform, GPT-in-a-Box Evolution

Nutanix Enterprise AI is Nutanix's 2024 AI platform product. It is best understood as the second act of **GPT-in-a-Box**:

| Milestone | Date | What happened | Verification |
|---|---|---|---|
| GPT-in-a-Box 1.0 | 15 Aug 2023 | Turnkey on-prem GenAI stack: NVIDIA GPUs + Nutanix platform + model serving + RAG tooling; the sovereignty play | Verified (press release; cross-ref [nutanix_products_guide.md](nutanix_products_guide.md) §8.1) |
| Nutanix × NVIDIA collaboration | 21 May 2024 | NVIDIA NIM inference microservices integrated with GPT-in-a-Box 2.0; announced at .NEXT Barcelona | Verified (joint press release / Business Wire) |
| **Nutanix Enterprise AI** | **November 2024** | The AI control plane product: UI-driven LLM endpoint deployment, curated validated models, NIM built in; a component of GPT-in-a-Box 2.0 | Verified (Nutanix blog; TechAfrica News 13 Nov 2024) |
| NAI agentic release | May 2025 | .NEXT release enabling agentic AI workloads; deeper NVIDIA AI Enterprise integration; RAG "blueprints" with reranking and safety guardrails | Verified (StorageNewsletter 8 May 2025; Nutanix blog) |

**Positioning (Verified):** Nutanix positions NAI as "a simple way to deploy LLMs" — an intuitive, UI-driven path to LLM inference endpoints "in minutes," with explicit control over *where models and data reside* (edge, on-premises, or public cloud) so that regulated and sovereign workloads never touch a public AI API. The Nutanix FAQ is admirably honest about scope: **Nutanix provides validated third-party LLMs and secure APIs to create endpoints for the generative-AI applications you provide — Nutanix does not provide GenAI applications** (Verified — nutanix.com/en_sg/products/nutanix-enterprise-ai/faq). In other words: NAI is the *platform and the model-inference layer*, not the finished chatbot or copilot; the application layer remains the customer's (or a partner's) job — the same honest boundary flagged for GPT-in-a-Box in [nutanix_products_guide.md](nutanix_products_guide.md) §8.3.

**Evolution vs GPT-in-a-Box 1.0 (Verified, with nuance):** GPT-in-a-Box 1.0 was a validated *reference architecture* — a bill of materials and deployment pattern (GPUs + AOS/AHV + Files/Objects + model software). GPT-in-a-Box 2.0 formalises this as a productised stack: **Nutanix Enterprise AI** (the AI software layer) + **Nutanix Kubernetes Platform** (the container substrate) + the Nutanix Cloud Platform underneath. NAI adds what 1.0 lacked: a proper model-serving control plane (endpoint deployment UI/APIs), a curated model catalog, built-in NIM integration, and RAG/guardrail tooling — i.e., it moves from "validated stack" toward "operational AI platform," while still stopping short of a full MLOps lifecycle (no first-class notebooks/pipelines/experiment tracking of its own — Flagged: check current datasheet, as NAI releases have been rapid).

### 2.2 The Nutanix AI Stack — AOS/AHV + GPU + NVIDIA NIM Integration

The NAI stack is Nutanix's HCI core with three AI layers on top (Verified for layers; the exact component list evolves with releases — Flagged):

```
┌──────────────────────────────────────────────────────────┐
│  NUTANIX ENTERPRISE AI (AI control plane, Nov 2024)      │
│  • UI/APIs: deploy LLM inference endpoints in minutes    │
│  • Curated validated models (Hugging Face + NVIDIA)      │
│  • RAG blueprints: embeddings + vector DB + reranking +  │
│    safety guardrails                                     │
├──────────────────────────────────────────────────────────┤
│  NVIDIA stack (May 2024 collaboration — Verified)        │
│  • NVIDIA NIM microservices (inference)                  │
│  • NVIDIA NeMo (model tooling), NVIDIA AI Enterprise     │
├──────────────────────────────────────────────────────────┤
│  NUTANIX KUBERNETES PLATFORM (NKP, ex-Karbon)            │
│  • Managed Kubernetes on AHV; orchestrates model-serving │
│    pods, GPU scheduling, lifecycle                       │
├──────────────────────────────────────────────────────────┤
│  NUTANIX CLOUD PLATFORM (the HCI core)                   │
│  • AOS (distributed storage: Files, Objects, Volumes)    │
│  • AHV (hypervisor) + Prism (management)                 │
│  • NVIDIA GPU nodes (A100/L40S-class and newer)          │
└──────────────────────────────────────────────────────────┘
```

Key structural facts:

- **The container layer is NKP, not OpenShift.** NAI's model-serving workloads run on Nutanix's own Kubernetes platform on AHV (Reported — Nutanix blog and NCP-AI training material describe NKP orchestrating model-serving pods and GPU scheduling; this is consistent with the product's positioning). If your organisation standardises on OpenShift, NAI is not the natural control plane — see §4.2.
- **The GPU layer is NVIDIA-first.** NIM inference microservices (from the NVIDIA API catalog), NeMo tooling, and NVIDIA AI Enterprise are the documented acceleration/inference stack (Verified — 21 May 2024 collaboration announcement; May 2025 release notes). AMD-GPU support is not part of the documented NAI story (Flagged: check current support matrix).
- **The data layer is the differentiator.** The same cluster that serves VMs serves the AI: Files for datasets/checkpoints, Objects for corpora and model artifacts, NDB for operational data feeding RAG — "your data is already here, under the same protection and governance" (Verified positioning; cross-ref [nutanix_products_guide.md](nutanix_products_guide.md) §8.3). RAG pipelines can pull directly from Nutanix Objects without egress.
- **Public-cloud deployment is supported.** NAI can be deployed in any Kubernetes environment, including public cloud, "operationally consistent" with on-premises deployments (Reported — tech news coverage of the launch; flag: confirm current NC2/cloud deployment mechanics in the datasheet).
- **Partner validation.** Nutanix runs an "AI Ready" validation program so partners (Fujitsu, Cisco, Lenovo, Dell, HPE OEM paths) can ship NAI-validated appliances — the same appliance-ish go-to-market as GPT-in-a-Box (Reported — siliconcanals coverage of the partner program).

### 2.2.1 The Deployment Walkthrough — From Cluster to Endpoint

The NAI deployment path is designed to be an infrastructure-team workflow, not a data-engineering one (steps verified in outline from Nutanix's product blog and NCP-AI training material; exact wizard names and options evolve with releases — Flagged):

| Step | What happens | Skills required |
|---|---|---|
| 1. **Estate sizing** | Choose the HCI cluster (existing or new), add GPU nodes (A100/L40S-class and newer), size storage for models + vector data (Files/Objects) | Infrastructure architect; reference [gpu_optimization_guide.md](gpu_optimization_guide.md) for GPU sizing and [nutanix_products_guide.md](nutanix_products_guide.md) §2–3 for the storage layer |
| 2. **Cluster provisioning** | AOS/AHV cluster up on Prism (or NC2 for cloud); GPU nodes attached and verified | Existing Nutanix runbooks |
| 3. **NKP enablement** | Nutanix Kubernetes Platform deployed on AHV; GPU device plugin/tolerations configured so NKP pods can schedule on GPU nodes | Moderate K8s concepts (namespaces, pods, tolerations) |
| 4. **NAI install** | Nutanix Enterprise AI enabled from the platform; connects to NKP and the NVIDIA stack (NIM microservices from the catalog, NVIDIA AI Enterprise validation) | Following the NAI admin guide |
| 5. **Model selection** | Pick from the curated validated catalog ("small/medium/large" — Hugging Face + NVIDIA validated) | None — the catalog does the choosing |
| 6. **Endpoint deployment** | Deploy an LLM inference endpoint via the NAI UI/API — "in minutes" (Verified); endpoint exposed as a secure API for your application | None |
| 7. **RAG blueprint** | Apply a preset blueprint: embedding model + vector DB + reranking + safety guardrails over your corpus (Verified, May 2025) | Corpus preparation (document ingestion, chunking) |
| 8. **Application layer** | Build/commission the GenAI application (copilot, summariser) against the NAI secure API — Nutanix explicitly does not provide applications (Verified — FAQ) | Application developers or an AI Ready partner (Reported) |

The takeaway: steps 1–7 are measured in days for a team that already runs Nutanix; step 8 is the part that takes quarters and is the bank's own work in the §6 example. This is the turnkey promise — and its honest boundary — in one walkthrough.

### 2.3 The Nutanix Features — Model Serving, RAG, Guardrails

**Model serving (Verified):** NAI's core feature is deployment of LLM inference endpoints through a UI/API "in minutes." Nutanix offers a **curated list of validated models** through partnerships with **Hugging Face and NVIDIA**, presented as a "small, medium, and large" selection so teams pick a proven model rather than assembling one (Verified — Tech Field Day presentation; Nutanix blog). NVIDIA NIM microservices (Llama-class, Nemotron-class and other catalog models) provide the inference runtime, with the option to run the same models on-prem or in the cloud. Endpoints are exposed as **secure APIs** for the customer's own GenAI applications (Verified — NAI FAQ).

**RAG (Verified):** RAG is a first-class NAI workflow, delivered as preset **"blueprints"** — a RAG flow can send context to a reasoning LLM, use **reranking** to pick the best answer, and integrate **embedding models** with **vector databases** (Verified — Nutanix blog, May 2025; reference architectures run vector DBs such as Milvus/Qdrant-class on the platform, cross-ref [nutanix_products_guide.md](nutanix_products_guide.md) §8.3). Because the vector store and the source documents can both live on Nutanix storage, the RAG loop stays inside the cluster — the sovereignty argument in practice. (For the general RAG pattern, see the `ai_llm/rag/` guides in this repo.)

**Guardrails (Verified):** the May 2025 agentic release explicitly includes **safety guardrails** in the inference path — reranking and guardrail steps that "determine the best answer and ensure the safety of the response" before it is returned (Verified — Nutanix blog "Nutanix Enterprise AI Makes Agents..."). Scope note (Flagged): these are inference-path guardrails (input/output filtering, safety checks) — NAI does not ship a full AI-governance suite (no built-in explainability/bias-detection layer comparable to TrustyAI; those concerns fall to the application layer and to the governance guides in this repo, e.g. [ai_governance_bias_redteaming_guide.md](ai_llm/ai_governance_bias_redteaming_guide.md)).

**What NAI does not ship out of the box (Flagged — honest boundary):** no notebook/workbench experience, no pipeline engine, no experiment tracking, no model registry of its own (the curated catalog is a deploy-from-catalog, not a team model registry), no hyperparameter tuning. Teams needing the full MLOps lifecycle must assemble it on NKP or look at the RHOAI/Kubeflow side of this comparison. This is the single most important scoping fact in the whole comparison.

### 2.3.1 NAI Feature Detail — Model Catalog, Endpoint Lifecycle, RAG Anatomy

**The curated model catalog (Verified in outline — Flagged for specifics):**

| Catalog dimension | What Nutanix provides | Notes |
|---|---|---|
| **Sourcing** | Validated models via Hugging Face + NVIDIA partnerships | "Small, medium, and large" selection (Verified — Tech Field Day) |
| **Sizes** | Small (fast/cheap, e.g., 7–8B-class), medium, large (best quality) | Sizing guidance built into the catalog |
| **Runtime** | NVIDIA NIM microservices from the NVIDIA API catalog | NIM integration announced 21 May 2024 (Verified) |
| **Fit check** | Nutanix validates model + GPU + platform combinations | Removes the "will it run" guesswork |
| **Custom models** | Not the primary path — the catalog is curated, not a BYO registry (Flagged) | Teams needing custom/registry workflows should look at RHOAI or an MLOps layer |

**The endpoint lifecycle (Verified in outline — the lifecycle NAI does manage):**

```
Model selected from catalog → NIM runtime pulled → endpoint deployed (UI/API, minutes)
  → secure API exposed to application → guardrails applied per endpoint
  → monitor/serve (platform-level metrics via Prism/NKP) → scale with GPUs
```

| Lifecycle stage | NAI behaviour | Gap to flag |
|---|---|---|
| Deploy | UI/API, minutes, validated stack | — |
| Version/promote | Catalog versions; no team model registry (Flagged) | Registry is application-layer work |
| Observe | Prism/NKP platform metrics (Reported) | No built-in model-quality monitoring (drift, hallucination rates) — Flagged |
| Guard | Inference-path safety guardrails + reranking (Verified, May 2025) | No explainability/bias-detection suite (Flagged) |
| Decommission | Endpoint teardown via UI/API | — |

**RAG blueprint anatomy (Verified — Nutanix blog, May 2025):** a NAI RAG flow chains: (1) **embedding model** (from the catalog) → (2) **vector database** (on-platform; Milvus/Qdrant-class per reference architectures, cross-ref [nutanix_products_guide.md](nutanix_products_guide.md) §8.3) → (3) **retrieval + context assembly** → (4) **reasoning LLM** (from the catalog) → (5) **reranking** of candidate answers → (6) **safety guardrails** on the response → (7) return. Delivered as preset **"blueprints"** so the bank in §6 applies the pattern, not a research project. The full RAG pattern beyond the blueprint is the subject of the `ai_llm/rag/` guides in this repo.

### 2.4 The Nutanix Table — Aspect / Description

| Aspect | Description |
|---|---|
| **Product** | Nutanix Enterprise AI (NAI) — AI control plane for LLM inference on the Nutanix platform; component of GPT-in-a-Box 2.0 |
| **Launch** | November 2024 (Verified); GPT-in-a-Box lineage from 15 Aug 2023 (Verified) |
| **Positioning** | Turnkey AI-infrastructure platform: validated models + NIM + storage under one UI; sovereignty-first |
| **Container platform** | Nutanix Kubernetes Platform (NKP, ex-Karbon) on AHV (Reported) |
| **GPU stack** | NVIDIA GPUs (A100/L40S-class and newer); NVIDIA NIM, NeMo, AI Enterprise (Verified) |
| **Model catalog** | Curated, validated "small/medium/large" models via Hugging Face + NVIDIA (Verified) |
| **Model serving** | UI/API endpoint deployment in minutes; secure APIs for customer apps (Verified) |
| **RAG** | Preset blueprints: embeddings + vector DB + reranking; data local on Nutanix storage (Verified) |
| **Guardrails** | Inference-path safety guardrails (input/output safety, reranking) (Verified, May 2025) |
| **MLOps lifecycle** | Not included out of the box (no notebooks/pipelines/experiment tracking) (Flagged) |
| **Governance** | Sovereignty/data-residency by design; no built-in explainability suite (Flagged) |
| **Cloud** | Deployable on any Kubernetes environment incl. public cloud (Reported) |
| **Ecosystem** | NVIDIA + Hugging Face + Nutanix AI Ready partner program (Reported) |
| **Pricing** | Not published; Nutanix subscription packaging (Flagged) |
---

## 3. Red Hat OpenShift AI — Deep Dive

### 3.1 The Red Hat MLOps Platform — Open Data Hub Lineage

Red Hat OpenShift AI (RHOAI) is Red Hat's enterprise MLOps platform for the full AI/ML lifecycle — training, serving, monitoring, and managing models and AI-enabled applications — delivered as an Operator on Red Hat OpenShift. Its lineage is the clearest way to understand it:

| Milestone | Date | What happened | Verification |
|---|---|---|---|
| Open Data Hub (ODH) project | 2018 | Red Hat-led open-source AI/ML platform project on OpenShift; began as a proof-of-concept on the Mass Open Cloud (runs on OpenShift and OpenStack); grew into the community reference architecture for AI on OpenShift | Verified (Red Hat Emerging Technology blog, 19 Sept 2018; opendatahub.io/GitHub) |
| Red Hat OpenShift Data Science (RHODS) | 2021 | Red Hat's commercial managed/self-managed data-science platform built from ODH components with enterprise hardening | Verified (Red Hat product history; press coverage) |
| **Red Hat OpenShift AI** | **23 May 2023** | Renamed and relaunched: "building and expanding upon the proven capabilities of Red Hat OpenShift and Red Hat OpenShift Data Science," positioning RHOAI as the unified platform to train, serve, monitor, and manage AI/ML models from experiments to production | Verified (Red Hat press release, 23 May 2023) |
| Versioning shift | 2025–2026 | RHOAI releases moved through 2.18/2.19 (mid-2025), 2.24, 2.25, then to 3.x (3.3/3.4/3.5 seen in 2026 security advisories) | Verified numbering via release/CVE coverage; exact GA dates flagged |
| ODH docs archived | March 2026 | opendatahub.io documentation archived in favour of the OpenShift AI Self-Managed documentation — ODH's stewardship role is effectively consolidated into RHOAI | Verified (opendatahub.io notice) |

**The lineage point that matters:** RHOAI is *Kubeflow/ODH in enterprise clothing*. The open-source components are upstream ODH/Kubeflow projects (notebooks, KFP-compatible pipelines, KServe, Katib); Red Hat's value-add is the integration, hardening, support, and the proprietary management layer (the RHOAI Operator, dashboards, Model Registry, TrustyAI). The same architecture detail — including the deprecations (ModelMesh in 2.19, CodeFlare in 2.24) — is documented component-by-component in [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md) §§2–6 and is not re-derived here.

### 3.2 The OpenShift AI Stack — OpenShift + Kubeflow Components

RHOAI is a layer on OpenShift, installed via Operator Lifecycle Manager (OLM) and reconciled through a single `DataScienceCluster` custom resource — one YAML turns every component on or off (`managementState: Managed | Removed`):

```
┌──────────────────────────────────────────────────────────┐
│  RHOAI COMPONENTS (OpenShift AI Dashboard)               │
│  • Workbenches (JupyterLab / RStudio / VS Code)          │
│  • Data Science Pipelines (Tekton, KFP-compatible)       │
│  • KServe model serving (via Serverless + Service Mesh)  │
│  • Training Operator (PyTorch/TF/XGBoost/MPI) + Ray      │
│  • Kueue (job queueing) + Model Registry + TrustyAI      │
├──────────────────────────────────────────────────────────┤
│  OPENSHIFT NATIVE SERVICES                               │
│  • OpenShift Pipelines (Tekton) • Serverless (Knative)   │
│  • Service Mesh (Istio) • Monitoring (Prometheus)        │
│  • GitOps (Argo CD) • Logging (Loki) • SCC security      │
├──────────────────────────────────────────────────────────┤
│  OLM + DataScienceCluster CR (single control plane)      │
├──────────────────────────────────────────────────────────┤
│  RED HAT OPENSHIFT (any supported infrastructure)        │
│  • Bare metal, vSphere, OpenStack, Nutanix AHV, AWS,     │
│    Azure, GCP, IBM Z/Power (Tech Preview)                │
└──────────────────────────────────────────────────────────┘
```

Structural facts:

- **The container platform is OpenShift, full stop.** RHOAI does not run on plain Kubernetes — it requires OpenShift, which brings the platform's native Pipelines (Tekton), Serverless (Knative), Service Mesh (Istio), Monitoring (Prometheus/Alertmanager), GitOps (Argo CD), Logging, and SCC pod-security. OpenShift itself runs on a wide range of infrastructure — bare metal, vSphere, OpenStack, public clouds (ROSA/ARO/IPI), **Nutanix AHV** (OpenShift supports Nutanix as an infrastructure provider — Reported, cross-ref [charmed_kubernetes_vs_openshift_guide.md](charmed_kubernetes_vs_openshift_guide.md)), and IBM Z/Power (Tech Preview for RHOAI).
- **Notebooks → Workbenches.** JupyterLab is the default; RStudio and VS Code images are available; per-project (OpenShift namespace) isolation, idle culling, GPU tolerations, custom images (Verified — cross-ref charmed guide §6.1).
- **Pipelines → Data Science Pipelines (DSP).** Tekton-based, KFP-compatible API and SDK, so upstream Kubeflow Pipelines code largely ports over; step caching, recurring runs, artifact tracking (Verified — cross-ref charmed guide §6.2).
- **Model serving → KServe.** Single-model serving via KServe on OpenShift Serverless + Service Mesh (scale-to-zero, canary, autoscaling); vLLM first-class for LLMs; RawDeployment mode for always-on low-latency; ModelMesh multi-model serving deprecated in 2.19 (Verified — cross-ref charmed guide §6.4).
- **Distributed training.** Training Operator (PyTorch, TensorFlow, XGBoost, MPI) + Ray for scaling; Kueue for cluster queues and GPU scheduling; CodeFlare deprecated in 2.24 (Verified — cross-ref charmed guide §6.3).
- **Management layer.** The RHOAI Operator + Dashboard are Red Hat proprietary; everything else traces to ODH/Kubeflow open source. Upgrades ride OLM channels with version-compatibility constraints across OpenShift, Service Mesh, Serverless, and RHOAI (Verified — cross-ref charmed guide §4).

### 3.2.1 The Deployment Walkthrough — From Cluster to DataScienceCluster

The RHOAI deployment path is a platform-engineering workflow (steps verified against Red Hat documentation via the sibling head-to-head, [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md) §4; exact operator versions and prerequisites shift with releases — Flagged):

| Step | What happens | Skills required |
|---|---|---|
| 1. **OpenShift cluster** | Stand up OpenShift (bare metal, vSphere, OpenStack, cloud, or Nutanix AHV as platform provider — Reported) | OpenShift administration (or a managed ROSA/ARO subscription) |
| 2. **Prerequisite operators** | Install OpenShift Service Mesh (Istio), Serverless (Knative), and Pipelines (Tekton) operators — required by RHOAI | Operator Lifecycle Manager familiarity |
| 3. **RHOAI operator** | Install the OpenShift AI operator from OperatorHub (OLM) | OLM concepts |
| 4. **DataScienceCluster CR** | Apply one YAML declaring which components are `Managed` (Workbenches, DSP, KServe, Training Operator, Model Registry, TrustyAI, Kueue…) | Understanding the component matrix (§3.2) |
| 5. **GPU enablement** | Install NVIDIA GPU Operator + NFD via OperatorHub; configure via `ClusterPolicy` CR; set up Kueue cluster queues and resource flavours | GPU operator + scheduling configuration (cross-ref [gpu_optimization_guide.md](gpu_optimization_guide.md)) |
| 6. **Projects and users** | Create OpenShift projects per team; wire OIDC/LDAP auth; assign RHOAI roles (admin vs data scientist) | OpenShift RBAC + SCC awareness |
| 7. **Model serving** | Data scientists deploy models as KServe `InferenceService` CRs (vLLM recommended for LLMs); scale-to-zero via Serverless; canary via Service Mesh | Kubernetes + KServe practice |
| 8. **Lifecycle use** | Notebooks → DSP pipelines → Training Operator jobs → Model Registry → TrustyAI monitoring | The data-science team's daily tools |

The takeaway: steps 1–6 are a **platform project** (weeks for a team that does not already run OpenShift; days if the platform exists), and steps 7–8 are the *point* — the lifecycle that NAI deliberately leaves out. The deployment cost is the price of the lifecycle.

### 3.3 The OpenShift AI Features

**Full MLOps lifecycle (Verified):** RHOAI covers the lifecycle NAI deliberately leaves out — notebooks for experimentation, DSP for pipeline automation, Training Operator/Katib for distributed training and hyperparameter tuning, KServe for serving, Model Registry for model versioning, TrustyAI for explainability and bias detection, Kueue for GPU-aware scheduling, and OpenShift Monitoring for observability. It is the complete package, at the price of complexity.

**LLM-era additions (Verified — cross-ref charmed guide §6.4, §10):** vLLM as the recommended LLM serving runtime; **RHEL AI / InstructLab** integration for Granite-model fine-tuning (the sibling to NAI's NIM integration); Ray for scaling data prep and serving; first-class scale-to-zero serving economics.

**Security and governance (Verified):** OpenShift SCC (the most granular pod-security model in the Kubernetes ecosystem), FIPS mode with validated cryptographic modules, comprehensive audit logging, OAuth (OIDC/LDAP/HTPasswd) — the features that make RHOAI the default answer for regulated banking/insurance/government estates (cross-ref charmed guide §11). TrustyAI adds model explainability and bias detection — the governance layer NAI flags as out of scope.

**Open-source posture (Verified):** the components are open source; the platform (OpenShift + RHOAI operator layer) is commercial subscription software. Red Hat is a founding-era contributor to ODH and Kubeflow — one of the major vendors (with AWS, Bloomberg, Google, IBM, NVIDIA, Nutanix, Arrikto) listed by Kubeflow itself (Verified — Kubeflow Wikipedia/contributors).

### 3.3.1 RHOAI Component Detail — Notebooks, Pipelines, Serving, Training

Condensed from the sibling head-to-head's component-by-component tables ([charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md) §6), where each row is Verified against Red Hat documentation:

**Notebooks (Workbenches):**

| Feature | RHOAI behaviour |
|---|---|
| IDEs | JupyterLab default; RStudio, VS Code images available |
| Isolation | Per-OpenShift-project (namespace) isolation; RHOAI admin/data-scientist roles |
| GPU | GPU tolerations + accelerator profiles |
| Lifecycle | Idle culling (configurable), PVC-backed home storage, custom images via ImageStreams |

**Pipelines (Data Science Pipelines):**

| Feature | RHOAI behaviour |
|---|---|
| Engine | Tekton (same engine as OpenShift CI/CD); KFP-compatible API + SDK |
| Porting | Upstream KFP v1/v2 pipelines largely port as-is; subtle behavioural deltas (caching, conditionals) |
| Automation | Recurring runs, step caching, artifact tracking to MLMD |

**Model serving (KServe):**

| Feature | RHOAI behaviour |
|---|---|
| Serving model | KServe `InferenceService` on OpenShift Serverless + Service Mesh |
| LLM runtime | vLLM first-class (recommended); Triton, TF/PyTorch runtimes, caikit |
| Scaling | Scale-to-zero (Knative KPA), HPA, canary/traffic-split, RawDeployment for always-on |
| Multi-model | ModelMesh deprecated in 2.19 — single-model serving is the supported path |

**Training and scheduling:**

| Feature | RHOAI behaviour |
|---|---|
| Distributed training | Training Operator (PyTorch, TensorFlow, XGBoost, MPI); Ray for scaling |
| Tuning | Katib (Bayesian/random/grid/NAS) |
| Scheduling | Kueue cluster queues + resource flavours; GPU quotas per project; CodeFlare deprecated in 2.24 |

**Governance and registry:**

| Feature | RHOAI behaviour |
|---|---|
| Model Registry | ODH Model Registry API (v1beta1) + dashboard UI; versioned artifacts, registration workflow |
| Explainability | TrustyAI — bias detection, explainability, model monitoring |
| Security | OpenShift SCC, FIPS mode, audit logging, OAuth (OIDC/LDAP/HTPasswd) |

### 3.3.2 Release Cadence and Managed Options

- **Version cadence (Verified numbering — exact dates flagged):** RHOAI shipped 2.5 in the 2023 rename era and moved through 2.9 → 2.11 → 2.13 → 2.16 → 2.19 (mid-2025) → 2.24 → 2.25, then renumbered to **3.x during 2026** (3.3/3.4/3.5 seen in security advisories). Releases trail upstream ODH/Kubeflow; Red Hat publishes a support matrix constraining OpenShift × Service Mesh × Serverless × RHOAI combinations — version-pinning is a real operational discipline.
- **Managed options (Verified):** RHOAI is available self-managed (on any OpenShift) and as a managed cloud service on Red Hat's managed platforms (ROSA on AWS, ARO on Azure) — the "consume it without running OpenShift yourself" path that NAI answers with the turnkey box. Managed RHOAI trades infrastructure ops for Red Hat-managed control planes and per-vCPU pricing.
- **Deprecation churn (Verified — cross-ref charmed guide §2.2):** ModelMesh deprecated in 2.19, CodeFlare in 2.24, v1alpha1 Model Registry API deprecated in 2.24 — the lifecycle of an actively evolving platform, and the price of the broad component set.

### 3.4 The OpenShift Table — Aspect / Description

| Aspect | Description |
|---|---|
| **Product** | Red Hat OpenShift AI (RHOAI) — enterprise MLOps platform on OpenShift; ex-RHODS, ex-OpenShift Data Science |
| **Launch / lineage** | Renamed May 2023 (Verified); RHODS (2021) → Open Data Hub (2018) lineage (Verified) |
| **Positioning** | Full-lifecycle MLOps: train, serve, monitor, manage — on the enterprise Kubernetes platform |
| **Container platform** | Red Hat OpenShift only (OLM Operators; `DataScienceCluster` CR) |
| **GPU stack** | NVIDIA GPU Operator via OperatorHub; NFD node discovery; Kueue queueing; MIG/time-slicing (cross-ref [gpu_optimization_guide.md](gpu_optimization_guide.md)) |
| **Notebooks** | Workbenches: JupyterLab, RStudio, VS Code; per-project isolation; idle culling (Verified) |
| **Pipelines** | Data Science Pipelines (Tekton, KFP-compatible API/SDK) (Verified) |
| **Model serving** | KServe + OpenShift Serverless/Service Mesh; vLLM first-class; RawDeployment mode (Verified) |
| **Training** | Training Operator (PyTorch/TF/XGBoost/MPI) + Ray; Katib tuning; Kueue scheduling (Verified) |
| **Registry & governance** | Model Registry (ODH API v1beta1) + TrustyAI explainability/bias detection (Verified) |
| **LLM integration** | RHEL AI / InstructLab (Granite), vLLM, Ray (Verified) |
| **Security** | SCC, FIPS mode, audit logging, OAuth OIDC/LDAP (Verified) |
| **Cloud** | OpenShift everywhere: ROSA, ARO, GCP IPI, on-prem, edge (MicroShift — RHOAI not supported there) |
| **Ecosystem** | Red Hat + IBM (Granite/RHEL AI) + Kubeflow/ODH community + OperatorHub ISVs |
| **Pricing** | OpenShift subscription + AI add-on; ~$12K–$80K+/yr production (Flagged, approximate) |

---

## 4. Head-to-Head — Table, Architecture, Ecosystem, Economics

### 4.1 The Comparison Table — Dimension by Dimension

| Dimension | Nutanix Enterprise AI | Red Hat OpenShift AI |
|---|---|---|
| **Category** | AI-infrastructure platform (turnkey validated stack) | MLOps application platform (component-rich K8s layer) |
| **Launch** | November 2024 (Verified) | May 2023 rename; ODH lineage 2018 (Verified) |
| **Underlying stack** | AOS + AHV + NKP (Nutanix HCI) + NVIDIA GPUs | OpenShift (any infra) + OLM operators + Kubeflow/ODH components |
| **Control plane** | NAI UI/APIs for model endpoints, RAG blueprints, guardrails | OpenShift AI Dashboard + `DataScienceCluster` CR |
| **Model deployment** | Curated validated catalog (HF + NVIDIA), "deploy in minutes" via UI | Bring-your-own via KServe InferenceService; vLLM recommended |
| **Notebooks** | Not included (Flagged) | Workbenches: JupyterLab/RStudio/VS Code (Verified) |
| **Pipelines** | Not included (Flagged) | DSP — Tekton, KFP-compatible (Verified) |
| **Training / tuning** | Not included (Flagged); NVIDIA NeMo tooling adjacent | Training Operator + Ray + Katib + Kueue (Verified) |
| **Model serving** | NIM microservices; secure APIs; sovereign endpoints | KServe + Serverless (scale-to-zero); vLLM; RawDeployment |
| **RAG** | First-class blueprints: embeddings + vector DB + reranking (Verified) | Build-your-own on Ray/vector DBs; no preset blueprints |
| **Guardrails** | Inference-path safety guardrails (Verified, May 2025) | TrustyAI bias/explainability; guardrails via app layer (no preset) |
| **Model registry** | Curated catalog only (no team registry) (Flagged) | Model Registry with API + UI (Verified) |
| **Explainability** | Not included (Flagged) | TrustyAI built-in (Verified) |
| **Data locality** | Native — data on the same HCI cluster (Files/Objects/NDB) | Bring-your-own storage; OpenShift Data Foundation optional |
| **GPU management** | NKP on AHV; NVIDIA stack (NIM/AI Enterprise) | NVIDIA GPU Operator + NFD + Kueue queues/flavours |
| **Security posture** | Data sovereignty by design; Prism/AHV controls | SCC + FIPS + audit — the regulated-industry benchmark |
| **Kubernetes skills needed** | Low (UI-driven; NKP managed) | High (OpenShift administration required) |
| **Infrastructure flexibility** | Nutanix-centric (any K8s for cloud, but the story is HCI) | OpenShift-centric (OpenShift runs on almost anything) |
| **Ecosystem anchor** | NVIDIA + Hugging Face | Red Hat + IBM + Kubeflow/ODH community |
| **Pricing model** | Nutanix subscription; not public (Flagged) | OpenShift sub + AI add-on; ~$12K–$80K+/yr (Flagged) |
| **Best first question** | "Do you run (or want) Nutanix HCI?" | "Do you run (or want) OpenShift?" |

**Feature-parity matrix (the quick scan):** ✅ built-in / first-class · ⚠️ possible but assembled or limited · ❌ not shipped

| Feature | NAI | RHOAI |
|---|---|---|
| LLM endpoint deployment (UI/API) | ✅ minutes, validated stack (Verified) | ✅ KServe `InferenceService` (CR-driven) |
| Curated validated model catalog | ✅ Hugging Face + NVIDIA, small/medium/large (Verified) | ⚠️ BYO models + community images; no vendor-curated catalog |
| Custom / BYO models | ⚠️ Not the primary path (Flagged) | ✅ Any model image; vLLM/Triton runtimes |
| Notebooks (JupyterLab etc.) | ❌ | ✅ Workbenches (JupyterLab/RStudio/VS Code) |
| Pipelines | ❌ | ✅ DSP (Tekton, KFP-compatible) |
| Distributed training | ❌ (NeMo tooling adjacent — Flagged) | ✅ Training Operator + Ray + Katib |
| GPU scheduling/queues | ⚠️ NKP scheduling on AHV (Reported) | ✅ Kueue cluster queues + flavours |
| Model registry | ❌ (catalog, not a team registry — Flagged) | ✅ Model Registry API + UI |
| RAG blueprints | ✅ preset: embeddings + vector DB + reranking (Verified) | ⚠️ assemble on Ray/vector DBs |
| Inference-path guardrails | ✅ safety guardrails (Verified, May 2025) | ⚠️ app layer + TrustyAI monitoring |
| Explainability / bias detection | ❌ | ✅ TrustyAI |
| Scale-to-zero serving | ❌ (not documented — Flagged) | ✅ OpenShift Serverless/KServe |
| Model monitoring | ⚠️ platform metrics only (Reported) | ✅ OpenShift Monitoring + TrustyAI |
| Multi-tenancy | ⚠️ Prism roles + NKP namespaces (Reported) | ✅ OpenShift projects + RHOAI roles |
| SSO / OIDC / LDAP | ⚠️ (Reported) | ✅ OpenShift OAuth end-to-end |
| FIPS / audit / SCC-class security | ⚠️ sovereignty-by-design instead | ✅ SCC, FIPS mode, audit logging |
| Managed cloud option | ⚠️ NAI on any K8s in cloud (Reported) | ✅ Self-managed + managed ROSA/ARO |

The matrix is the honest summary of the whole comparison: **NAI wins where the answer is "curated and quick," RHOAI wins where the answer is "complete and composable."** There are no surprises past this table.

### 4.2 Architecture Comparison — HCI-Based vs Kubernetes-Based

**Nutanix Enterprise AI is HCI-based (Verified).** The architecture is a vertical extension of the Nutanix Cloud Platform: AOS provides the distributed storage (block/file/object) and data services, AHV is the hypervisor, NKP provides Kubernetes on top of AHV, and NAI sits on NKP as the AI control plane. Everything — GPUs, model weights, vector stores, source documents — lives on the same cluster, managed by Prism. The architectural *bet* is data gravity and simplicity: the platform is where the data already is, and the AI layer is a feature of the infrastructure rather than a separate platform. The corollary: NAI's capabilities are bounded by what NKP/Nutanix ship — there is no plug-in OpenShift-native service mesh, and the MLOps lifecycle is thin by design.

**OpenShift AI is Kubernetes-based (Verified).** The architecture is a horizontal platform on top of OpenShift: OpenShift provides the cluster (on whatever infrastructure you choose — bare metal, vSphere, OpenStack, public cloud, even Nutanix AHV as a platform provider), and RHOAI installs as operators. The AI layer is a *tenant* of the platform: notebooks, pipelines, and inference services are all ordinary OpenShift workloads governed by OpenShift's security (SCC), networking (Service Mesh), and observability (Monitoring). The architectural *bet* is composability and lifecycle completeness: the platform is generic and reusable, and the AI layer is a rich set of interchangeable components. The corollary: it is a heavier, more skill-demanding architecture (OpenShift administration is a speciality), and it does not give you the turnkey "box" — you assemble the estate (or buy OpenShift on pre-validated hardware separately).

| Architectural aspect | Nutanix Enterprise AI | OpenShift AI |
|---|---|---|
| **Substrate model** | HCI: compute + storage + hypervisor + K8s in one product family | K8s platform: cluster + optional storage (BYO/ODF) |
| **Control-plane location** | NAI on NKP on AHV | RHOAI operators on OpenShift |
| **Data path** | Native — storage and AI on the same cluster | External/attached storage; PVCs |
| **Component model** | Curated, validated, few knobs | Composable, many components, per-component `managementState` |
| **Scale-to-zero serving** | Not documented (Flagged) | Yes — OpenShift Serverless (Verified) |
| **Multi-tenancy** | Prism/AHV role model + NKP namespaces (Reported) | OpenShift projects + RHOAI roles (Verified) |
| **Portability** | NAI on any K8s for cloud; core story is HCI-bound | OpenShift-bound; OpenShift itself is infra-portable |
| **Operational model** | Infrastructure team runs it like storage/VM estate | Platform team runs it like a K8s platform |
| **Failure domain** | Single-vendor stack (Nutanix + NVIDIA) | Multi-project stack (Red Hat + ODH community) |

**Where the AI workloads actually run — a closer look:**

| Workload | Nutanix Enterprise AI | OpenShift AI |
|---|---|---|
| Inference endpoint | NIM microservice pod on NKP, GPU scheduled on AHV node | KServe `InferenceService` pod on OpenShift worker, GPU via device plugin |
| Vector database (RAG) | On-platform (Milvus/Qdrant-class) beside the data | BYO vector DB in-cluster (or external); you assemble |
| Embedding/reasoning models | From the validated catalog, NIM runtime | BYO model images; vLLM/Triton runtimes |
| Notebook | Not shipped (Flagged) | Workbench pod per project, PVC-backed |
| Pipeline execution | Not shipped (Flagged) | Tekton `PipelineRun` per DSP run |
| Model artifacts | Nutanix Objects (model store) | PVC / Object storage; Model Registry metadata |
| Monitoring | Prism/NKP platform metrics (Reported) | OpenShift Monitoring (Prometheus) discovers KServe metrics automatically |
| Identity | Prism roles + NKP (Reported) | OpenShift OAuth (OIDC/LDAP) end-to-end |

**The nuance worth naming:** the two are *not* mutually exclusive at the infrastructure layer — OpenShift runs on Nutanix AHV as a supported platform provider (Reported), and Nutanix itself is a commercial Kubeflow-support vendor in the alternatives landscape ([openshift_ai_alternatives_guide.md](openshift_ai_alternatives_guide.md) §2.1). A shop can run RHOAI *on* Nutanix infrastructure — but then the AI control plane is RHOAI, not NAI. The fork is about who owns the AI layer, not about whether Nutanix hardware is involved.

### 4.3 Ecosystem Comparison — NVIDIA vs Red Hat

**Nutanix anchors on NVIDIA (Verified).** The NAI ecosystem is deliberately narrow and deep: NVIDIA NIM microservices and NeMo (the inference and model-tooling runtime), NVIDIA AI Enterprise certification, a curated Hugging Face + NVIDIA model catalog, vector databases for RAG, and the Nutanix AI Ready partner program for OEM appliances (Fujitsu, Cisco, Lenovo, Dell, HPE paths — Reported). This is an *infrastructure ecosystem*: it makes the box work well with GPUs and models, and it is single-threaded on NVIDIA for acceleration (AMD-GPU support not documented — Flagged). For sovereign builds, the same partnership pattern appears at scale in HTX NGINE, where Nutanix is the platform layer with NVIDIA and Google ([htx_ngine_guide.md](htx_ngine_guide.md) §6.3).

**Red Hat anchors on the open-source ML community (Verified).** RHOAI's ecosystem is the ODH/Kubeflow project ecosystem (KServe, KFP/DSP, Katib, Training Operator), extended with Red Hat and IBM assets (RHEL AI/InstructLab for Granite fine-tuning, OpenShift-native Pipelines/Serverless/Mesh/GitOps/Monitoring, OperatorHub ISV integrations) and the broader ML tooling market (Ray, vLLM, MLflow deployable in-cluster). It is a *platform ecosystem*: broad, composable, multi-vendor — at the cost of integration work and version-skew management that Red Hat's support contract absorbs but does not eliminate.

| Ecosystem aspect | Nutanix Enterprise AI | OpenShift AI |
|---|---|---|
| **Anchor partner** | NVIDIA (NIM, NeMo, AI Enterprise) | Red Hat + IBM (Granite, RHEL AI); ODH/Kubeflow community |
| **Model supply** | Curated catalog (Hugging Face + NVIDIA), validated | BYO models; vLLM; RHEL AI/InstructLab fine-tuning |
| **Vector/RAG** | Vector DBs on-platform, RAG blueprints | Ray + any vector DB; you assemble the pattern |
| **MLOps tools** | Not shipped (Flagged) | Full set: registry, TrustyAI, Kueue, DSP |
| **Hardware partners** | OEM appliance paths (AI Ready program) | OpenShift-certified hardware broadly (incl. Nutanix AHV) |
| **Lock-in profile** | High on Nutanix platform + NVIDIA; low model lock-in | High on OpenShift; low component lock-in (open source) |
| **Support model** | Single vendor (Nutanix) for the stack | Red Hat for platform; community for components |

**Who you actually integrate with — the partner/ISV surface:**

| Integration surface | Nutanix Enterprise AI | OpenShift AI |
|---|---|---|
| Inference software | NVIDIA NIM (catalog models), NVIDIA AI Enterprise | KServe runtimes: vLLM, Triton, TF/PyTorch, caikit; NIM deployable in-cluster |
| Model supply | Hugging Face + NVIDIA curated catalog | Any model image; Hugging Face; IBM Granite via RHEL AI/InstructLab |
| Fine-tuning | NVIDIA NeMo tooling (adjacent; not a NAI UI feature — Flagged) | RHEL AI / InstructLab integration (Verified) |
| Vector databases | On-platform reference architectures (Milvus/Qdrant-class) | BYO — Milvus, Qdrant, pgvector, Elasticsearch all deployable |
| Observability | Prism + NKP metrics (Reported) | OpenShift Monitoring + TrustyAI; Grafana, Loki optional |
| CI/CD | Application layer only | OpenShift Pipelines + GitOps (Argo CD) — natural integration |
| ISV breadth | Nutanix AI Ready partners (appliances/apps) (Reported) | OperatorHub certified operators; Red Hat partner ecosystem |
| Sovereign builds | HTX NGINE pattern (Nutanix + NVIDIA + Google) ([htx_ngine_guide.md](htx_ngine_guide.md) §6.3) | Red Hat/IBM sovereign engagements; RHOAI in regulated estates |

### 4.4 The Economics — Pricing (Flagged)

Neither vendor publishes a clean, comparable price list; the figures below are **estimates for planning, not quotes** (Flagged). The structure of the two cost models is the reliable part; the numbers are the sketchy part.

**Nutanix Enterprise AI economics (Flagged):** Nutanix is a 100% software, subscription-first company (cross-ref [nutanix_products_guide.md](nutanix_products_guide.md) §9) — NAI has **no public list price** and is delivered through Nutanix subscription packaging (platform subscription, typically with GPU-node sizing) plus server hardware from OEM partners (Dell/HPE/Lenovo/Cisco/Fujitsu). Practical cost drivers: (1) HCI cluster + GPU nodes (A100/L40S-class — the GPU capex dominates); (2) the Nutanix software subscription covering AOS/AHV/Prism/NKP + NAI; (3) support. Budget guidance: expect the GPU hardware to be 60–80% of the initial outlay and the software subscription to be a per-node/term charge sized by cluster — but **negotiate and confirm via Nutanix sales**; do not plan from published figures that do not exist.

**OpenShift AI economics (Flagged — approximate):** RHOAI costs = OpenShift subscription + AI add-on. Planning figures carried over from the sibling head-to-head ([charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md) §12): OpenShift Platform Plus roughly **$1,000–$2,500 per 2-core/year**, with an **AI add-on estimated at $500–$1,500 per worker node/year**; a minimum viable 3-node cluster lands around **$12K–$25K/year** and a production 10-node GPU cluster around **$40K–$80K+/year** (plus the same GPU hardware capex as any platform). RHOAI also carries an operational cost NAI avoids: OpenShift administration skills and prerequisite operators (Service Mesh, Serverless, Pipelines).

| Cost driver | Nutanix Enterprise AI | OpenShift AI |
|---|---|---|
| **Software pricing** | Not published; Nutanix subscription (Flagged) | OpenShift sub + AI add-on (Flagged, approx) |
| **Hardware** | OEM servers incl. NVIDIA GPUs; capex-dominant | Your choice: bare metal/cloud; same GPU capex |
| **Minimum viable estate** | 3–4 node HCI + 1–2 GPU nodes | 3-node OpenShift + GPU workers; ~$12K–$25K/yr software |
| **Production (10 nodes, GPUs)** | Subscription sized per cluster (Flagged) | ~$40K–$80K+/yr software (Flagged) |
| **Hidden/ops costs** | NVIDIA AI Enterprise licensing; storage sizing | OpenShift admin skills; prerequisite operators |
| **Support** | Included in Nutanix subscription | Included in Red Hat subscription |
| **Cost model verdict** | All-in turnkey; predictable-ish, opaque list price | Transparent-ish tiers; adds up fast at scale |

**Economics takeaway (Flagged):** NAI's total cost is dominated by the physical estate and is opaque to list-price comparison; RHOAI's total cost is dominated by per-core subscription economics that scale with every worker node. For the [finops_guide.md](finops_guide.md) lens: NAI is a "buy the factory" model, RHOAI is a "buy capacity continuously" model — the former wins for steady-state sovereign estates, the latter can win for elastic GPU usage where scale-to-zero serving (Serverless) actually saves license-bearing capacity.

**TCO scenario sketch — 36 months, one platform, two GPU nodes (all figures planning-grade, Flagged):**

| Cost line | Nutanix Enterprise AI (est.) | OpenShift AI (est.) |
|---|---|---|
| GPU nodes (2 × 8-GPU A100/L40S-class) | ~$250K–$400K capex (hardware-dominant) | Same hardware — no platform avoids GPU capex |
| Software subscription (36 mo) | Nutanix sub sized per cluster — not public; budget 20–40% of hardware (Flagged) | OpenShift (~$1K–$2.5K/2-core/yr) + AI add-on (~$500–$1.5K/node/yr): ~$40K–$80K/yr for a 10-node estate (Flagged) |
| Platform operations (36 mo) | Existing infra team; low incremental headcount | +1–2 OpenShift platform engineers or managed ROSA/ARO premium |
| Data science tooling | Application-layer budget (copilot app, vector ops) | Included in subscription (DSP, registry, TrustyAI) |
| NVIDIA software | NVIDIA AI Enterprise licensing (Flagged) | GPU Operator included; NIM optional |
| **Rough 36-month total** | **~$350K–$600K**, most of it hardware | **~$450K–$800K+**, software + people recurring |
| Break-even intuition | Wins if the estate exists and stays steady-state | Wins if GPU usage is elastic or the platform is shared across many teams |

The honest reading: NAI's *recurring* cost is the smaller part (hardware is sunk), while RHOAI's *recurring* cost compounds with every node and every year. Neither is cheap; both are defensible — the economics follow the same fork as everything else in this comparison.
---

## 5. Selection Guidance — Which for Which Need

### 5.1 The Which-for-Which-Need Table

| Need / situation | Nutanix Enterprise AI | Red Hat OpenShift AI |
|---|---|---|
| "We need on-prem LLM serving fast, with minimal team" | ✅ **Strong** — deploy validated models via UI in minutes; NKP managed | ⚠️ Workable, but you must stand up and run OpenShift first |
| "We already run Nutanix HCI and want AI on it" | ✅ **Strong** — AI rides the estate where the data already lives | ⚠️ Possible (OpenShift on AHV), but you add a second platform |
| "We already run OpenShift / are standardising on it" | ❌ Wrong layer — NAI replaces rather than extends OpenShift | ✅ **Strong** — natural extension; single platform, single vendor |
| "We need the full MLOps lifecycle (notebooks, pipelines, registry)" | ❌ Not shipped out of the box (Flagged) | ✅ **Strong** — the complete lifecycle, integrated |
| "We are a regulated bank / insurer / government agency" | ✅ Strong for sovereignty + turnkey; guardrails included | ✅ **Strong** — SCC + FIPS + audit + TrustyAI; the compliance benchmark |
| "We have small IT team, no Kubernetes platform engineers" | ✅ **Strong** — infrastructure-team-shaped product | ⚠️ Requires OpenShift admin skills (or managed ROSA/ARO) |
| "We have a large data-science organisation (50–500 people)" | ⚠️ Thin lifecycle; teams will outgrow the catalog | ✅ **Strong** — multi-project tenancy, Kueue scheduling, registry |
| "We want a team model registry + explainability/governance" | ❌ Not included (Flagged) | ✅ **Strong** — Model Registry + TrustyAI built-in |
| "We need scale-to-zero serving economics for bursty inference" | ❌ Not documented (Flagged) | ✅ **Strong** — OpenShift Serverless/KServe |
| "We are an NVIDIA-centric shop (DGX, NIM, AI Enterprise)" | ✅ **Strong** — NIM-native by design | ✅ Strong — GPU Operator + NIM also supported in-cluster |
| "We are an IBM/RHEL shop (Granite, InstructLab, Z-series)" | ❌ Not the fit | ✅ **Strong** — RHEL AI/InstructLab integration, IBM Z Tech Preview |
| "We want maximum multi-cloud portability of the AI layer" | ⚠️ NAI on any K8s for cloud (Reported); core is HCI | ⚠️ OpenShift-bound, but OpenShift runs on all clouds |
| "We want to avoid vendor lock-in" | ⚠️ Single-vendor stack (Nutanix + NVIDIA) | ⚠️ OpenShift-locked; components are open source |
| "We want a sovereign-AI build (national/industry AI factory)" | ✅ **Strong** — the HTX NGINE pattern ([htx_ngine_guide.md](htx_ngine_guide.md)) | ✅ Strong — RHOAI in sovereign/regulated estates, with Red Hat/IBM backing |

### 5.2 Selection Criteria — The Verified Checklist

Run these criteria in this order; the first two are filters, the rest are weightings.

1. **The stack filter (the decisive question).** Which substrate do you already run or intend to run? NAI extends the **Nutanix HCI stack**; RHOAI extends the **OpenShift stack**. Choosing the platform against the grain of your estate (NAI into an OpenShift shop, RHOAI into a pure-AHV shop with no K8s skills) is the most common and most expensive mistake. *Verified framing: platform follows stack — see §7.*
2. **The lifecycle requirement.** Do you need notebooks, pipelines, training, registry, and explainability as product features? If yes, NAI is disqualified as the *only* platform (its own FAQ confirms it provides endpoints, not applications; the catalog is curated models, not team MLOps) — Verified. If your need is "serving + RAG + guardrails on our own data," NAI is fully in scope.
3. **Team shape and skills.** Infrastructure teams that run storage/VM estates comfortably → NAI's operational model matches (Prism-centric, UI-driven, NKP managed). Platform/data-engineering teams with Kubernetes fluency → RHOAI matches. Hiring or training OpenShift administrators is a real, budgeted cost — verified as the top operational delta in §4.4.
4. **Compliance and governance.** For MAS-TRM/PCI-DSS/SOC-2-class estates: RHOAI's SCC + FIPS + audit + TrustyAI is the documented benchmark (Verified — cross-ref charmed guide §11); NAI's sovereignty-by-design (data never leaves the estate) is its complementary strength. Map your specific control set to both before deciding.
5. **Model and GPU strategy.** NVIDIA NIM-centric, want validated small/medium/large models with guardrails built into the inference path → NAI. Want to serve any model (vLLM, custom, Granite) and fine-tune in-cluster → RHOAI.
6. **Economics.** NAI: opaque list price, capex-dominant, predictable subscription; get a quote (Flagged). RHOAI: transparent per-core/per-node tiers, scales linearly with estate; ~$12K–$80K+/yr software for production (Flagged — cross-ref charmed guide §12). Add the ops-cost delta either way.
7. **Ecosystem patience.** NAI = narrow, deep, NVIDIA-anchored, turnkey. RHOAI = broad, composable, community-anchored, integration work absorbed by Red Hat support. Choose based on whether you want fewer decisions (NAI) or more options (RHOAI).

**Decision shortcut:** if question 1 answers "Nutanix HCI" and question 2 answers "serving/RAG only" → **NAI**. If question 1 answers "OpenShift (or any-K8s-with-OpenShift-flavour)" and question 2 answers "full MLOps" → **RHOAI**. Mixed answers → score both with the weighted template pattern from [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md) §15, then let §6's worked example show how the scoring plays out in practice.

**The weighted scoring template (blank — copy and score):**

| # | Criterion | Weight (1–5) | NAI (1–5) | RHOAI (1–5) | Notes |
|---|---|---|---|---|---|
| 1 | Existing substrate (Nutanix HCI vs OpenShift) | | | | Weight 5 — the decisive filter |
| 2 | Lifecycle need (serving/RAG only vs full MLOps) | | | | Weight 5 — NAI disqualified on full lifecycle |
| 3 | Team skills (infra vs platform engineers) | | | | Weight by hiring budget |
| 4 | Compliance controls (sovereignty vs audit/FIPS) | | | | Weight 5 for regulated estates |
| 5 | Time-to-first-value | | | | Weight high if business pressure |
| 6 | GPU/model strategy (NIM catalog vs BYO/vLLM) | | | | NVIDIA-centric → NAI; model freedom → RHOAI |
| 7 | Scale of data-science org | | | | >50 data scientists tilts RHOAI |
| 8 | Cost model (capex vs recurring) | | | | FinOps lens ([finops_guide.md](finops_guide.md)) |
| 9 | Vendor lock-in tolerance | | | | Both are lock-in-heavy, differently |
| 10 | Managed-service appetite (NC2/NAI vs ROSA/ARO) | | | | Consume-vs-operate preference |
| | **Total** | **50 max** | **50 max** | **50 max** | **Higher = recommended** |

§6 fills this template in for a real (fictional) bank and shows how the weighting — not the capability lists — decides the outcome.

---

## 6. Worked Example — A Mid-Size Bank's AI Platform Selection

### 6.1 The Scenario (Familiar Context)

A **mid-size regional bank in Singapore** (~2,500–4,000 employees; MAS-regulated; retail + SME banking) needs an on-premises generative-AI platform. The bank already runs **Nutanix HCI** (AOS/AHV/Prism) for its core application estate — the same starting point as the worked example in [nutanix_products_guide.md](nutanix_products_guide.md) §11 — and has a **small infrastructure team** (no dedicated Kubernetes platform engineers) plus a **nascent data-science team of ~8** that currently builds in Jupyter notebooks on laptops. The business sponsors two use cases, both data-sovereignty-constrained (customer data may not leave the estate):

- **Use case A — Customer-service copilot:** RAG over the bank's product terms, policies, and FAQ corpus; agents grounded in the bank's documents; answers must be safe and non-hallucinated (guardrails).
- **Use case B — Document intelligence:** summarisation and extraction over internal documents (KYC packs, loan files) — batch-ish, internal users only.

The CIO has asked for one platform decision that covers both use cases for the next 36 months. Two vendor shortlists have emerged from the [ml_platforms_comparison_guide.md](ml_platforms_comparison_guide.md) survey: **Nutanix Enterprise AI** (the incumbent infrastructure vendor's AI platform) and **Red Hat OpenShift AI** (the MLOps-standard candidate, cross-ref [openshift_ai_alternatives_guide.md](openshift_ai_alternatives_guide.md)).

### 6.2 The Requirements

| # | Requirement | Detail | Weight (1–5) |
|---|---|---|---|
| R1 | Data sovereignty | Models + data stay on-prem; no public AI APIs | 5 |
| R2 | Time-to-first-value | First copilot in production within one quarter | 4 |
| R3 | Team fit | Operable by the existing infra team; no new K8s platform team | 4 |
| R4 | Compliance | MAS TRM, audit trail, model-answer safety (guardrails) | 5 |
| R5 | RAG on bank documents | Grounded answers over internal corpora; vector store in estate | 5 |
| R6 | MLOps growth path | Notebooks/pipelines/registry for the growing DS team | 3 |
| R7 | Existing estate leverage | Reuse the Nutanix HCI investment; avoid a second infrastructure platform | 4 |
| R8 | Cost predictability | Software cost visible and stable for 36 months | 3 |

### 6.3 Applying the Comparison

| Requirement | Nutanix Enterprise AI — evidence | OpenShift AI — evidence |
|---|---|---|
| R1 Sovereignty | ✅ Native: models/NIM + data on the HCI estate; the core positioning (Verified) | ✅ Strong: on-prem OpenShift, no cloud dependency; BYO storage |
| R2 Time-to-value | ✅ UI-driven validated-model deployment "in minutes" (Verified) | ⚠️ Requires OpenShift estate + operators first; weeks not minutes |
| R3 Team fit | ✅ Infrastructure-team-shaped; NKP managed, Prism-centric (Reported) | ⚠️ OpenShift administration is a new speciality; hiring needed |
| R4 Compliance | ⚠️ Sovereignty + inference guardrails (Verified); no audit/explainability suite | ✅ SCC + FIPS + audit + TrustyAI — the regulated benchmark (Verified) |
| R5 RAG | ✅ Preset RAG blueprints: embeddings + vector DB + reranking (Verified) | ⚠️ Build-your-own on Ray/vector DBs; more assembly |
| R6 MLOps path | ❌ Not shipped out of the box (Flagged) | ✅ Full lifecycle: DSP, Training Operator, Model Registry |
| R7 Estate leverage | ✅ AI rides the existing AOS/AHV estate; data already local (Verified) | ⚠️ Adds a second platform layer (OpenShift) on top of AHV |
| R8 Cost | ⚠️ Opaque list price; subscription quote needed (Flagged) | ⚠️ Transparent tiers but ~$12K–$80K+/yr + ops cost (Flagged) |

**Scoring (1–5, weight × score):** NAI — R1:5×5=25, R2:4×5=20, R3:4×5=20, R4:5×3=15, R5:5×5=25, R6:3×1=3, R7:4×5=20, R8:3×3=9 → **137/170**. RHOAI — R1:5×5=25, R2:4×2=8, R3:4×2=8, R4:5×5=25, R5:5×3=15, R6:3×5=15, R7:4×2=8, R8:3×3=9 → **113/170**. (Scores are illustrative of the method, not a substitute for vendor quotes and PoCs — Flagged.)

### 6.4 The Choice — Verified

**The bank chooses Nutanix Enterprise AI** — and the decision is defensible on the verified evidence:

1. **The stack filter decides it (§5.2 criterion 1).** The bank is a Nutanix HCI estate with no OpenShift; choosing RHOAI means standing up and staffing a second infrastructure platform. NAI extends what the bank already runs and already knows how to operate (Verified positioning; Reported for NKP-managed operations).
2. **The lifecycle requirement does not disqualify NAI here (§5.2 criterion 2).** The bank's R6 (MLOps growth path) is weighted 3/5 and is genuinely secondary — the DS team of 8 gets notebooks and experiment tracking from the application layer in year 2 while NAI covers serving/RAG/guardrails. The bank explicitly budgets for the application layer (per the honest NAI FAQ boundary — Verified: Nutanix provides endpoints, not applications; the copilot app itself is built by the bank's vendor partner).
3. **R4 compliance is met differently but adequately.** MAS TRM here is satisfied by sovereignty-by-design (data never leaves the estate — the bank's dominant control), inference-path safety guardrails (Verified, May 2025), and the bank's own application-layer logging — rather than RHOAI's platform-level audit/FIPS. A bank whose dominant controls were platform-level (FIPS mandates, deep audit requirements) should flip this conclusion toward RHOAI — the scoring would flip too, because R4 is the heaviest weight.
4. **The RAG requirement is native.** Preset blueprints with embedding models, vector DBs, and reranking on-platform (Verified) match use case A directly, and the corpus already sits on Nutanix Files/Objects.

**Post-decision verifications (the "choice-verified" step):** (a) obtain a Nutanix subscription quote sized for 2 GPU nodes (A100/L40S-class) + the existing cluster, and confirm NAI's exact current component list and pricing — both flagged as unverifiable from public sources in §2.4/§4.4; (b) confirm with Nutanix that the specific curated models required (e.g., a Llama-class model for the copilot) are in the validated catalog at the right size for 2 GPUs; (c) run a 4-week PoC on one GPU node with the bank's actual FAQ corpus before committing; (d) engage a Nutanix AI Ready partner for the application layer (Reported — partner program).

### 6.4.1 The Implementation Roadmap (Post-Choice)

| Phase | Timeline | Activities | Gate |
|---|---|---|---|
| 0. **PoC** | Weeks 1–4 | One GPU node; deploy a small catalog model; run the bank's FAQ corpus through a RAG blueprint; measure answer quality and guardrail behaviour | Business sign-off on demo |
| 1. **Foundation** | Months 2–3 | GPU nodes in production estate; NKP + NAI install; endpoint + RAG blueprint for use case A; security review (MAS TRM mapping) | Infosec approval |
| 2. **Use case A live** | Months 4–6 | Customer-service copilot application (partner-built) on NAI secure APIs; guardrails tuned; drift/hallucination logging added at app layer | Business go-live |
| 3. **Use case B** | Months 6–9 | Document-intelligence endpoints; internal user rollout; access control via Prism/NKP identity | Internal rollout |
| 4. **Growth path** | Months 9–36 | Evaluate MLOps additions on NKP (notebooks, experiment tracking) or the RHOAI/Kubeflow route for the DS team; revisit NAI catalog for new models | Architecture review |

### 6.4.2 The Risk Register (What Could Derail the Choice)

| Risk | Likelihood | Mitigation |
|---|---|---|
| NAI pricing differs materially from the quote-free estimates (Flagged) | Medium | Lock a 36-month subscription quote before PO; model worst case in §4.4's TCO sketch |
| Curated catalog lacks the needed model class/size at rollout | Medium | PoC phase 0 verifies catalog fit on the actual GPUs; keep a BYO path on NKP as fallback |
| Guardrails insufficient for regulatory scrutiny of LLM answers | Medium | App-layer logging + human-in-the-loop review in phase 2; MAS TRM mapping as a gate |
| DS team outgrows NAI's thin lifecycle (Flagged) | Medium-high by year 3 | Plan the phase-4 review explicitly; the estate (NKP/GPUs) is reusable by a later RHOAI/Kubeflow layer — the hardware choice is not stranded |
| NVIDIA licensing/stack changes (NIM versioning) | Low | Nutanix validates NIM combinations; pin versions with Nutanix support |
| RHOAI alternative later wins internal comparison | Low | The §6.5 lessons keep the method (not the answer) as the durable artefact |

### 6.5 The Lessons

1. **The substrate question comes first, always.** In this example the decisive fact was not "which platform is more capable" but "which platform extends the stack we already run." RHOAI was the more complete MLOps product (verified) and still lost on fit.
2. **Scope honesty is a selection asset.** NAI's own FAQ says it ships endpoints, not applications — the bank planned the application layer instead of discovering it. The same honesty saved the bank from assuming NAI included notebooks/registry it does not (Flagged).
3. **Weight the compliance control set, not the compliance checkbox.** Sovereignty controls → NAI; platform-level audit/FIPS/explainability controls → RHOAI. Same bank, different dominant controls, different answer.
4. **Flagged facts must be closed before purchase.** NAI pricing, the exact catalog, and the guardrail behaviour in production are all either unpublicised or release-dependent; the PoC and the quote are part of the decision, not afterthoughts.
5. **The pattern generalises.** The same method (stack filter → lifecycle check → weighted scoring → close the flagged facts) selects RHOAI for a different profile: an OpenShift-standardised bank with 100 data scientists and a platform team. The guide's §5 table is the lookup; §6 is the method in motion.

---

## 7. Summary — One Page

**Nutanix Enterprise AI (Nov 2024, Verified)** is the AI-infrastructure platform: validated NVIDIA NIM-based model serving, curated Hugging Face/NVIDIA models, RAG blueprints, and inference-path guardrails — delivered as the AI layer of the Nutanix HCI stack (AOS/AHV/NKP), evolving GPT-in-a-Box (2023). **Red Hat OpenShift AI (renamed May 2023, Verified; Open Data Hub lineage 2018)** is the MLOps application platform: Workbenches, Data Science Pipelines, KServe + vLLM, Training Operator, Model Registry, TrustyAI, Kueue — delivered as operators on OpenShift.

| | Nutanix Enterprise AI | OpenShift AI |
|---|---|---|
| **Category** | Turnkey AI infrastructure | Full MLOps platform |
| **Substrate** | Nutanix HCI (AOS/AHV/NKP) + NVIDIA | OpenShift + Kubeflow/ODH components |
| **Sweet spot** | Serving + RAG + guardrails, fast, on your estate, small team | Lifecycle MLOps at scale, regulated estates, OpenShift shops |
| **Watch-outs** | No MLOps lifecycle; opaque pricing (Flagged) | OpenShift-only; subscription + ops cost (Flagged) |
| **Ecosystem** | NVIDIA + Hugging Face, deep and narrow | Red Hat + IBM + ODH community, broad and composable |
| **Final word** | The platform follows the stack | The platform follows the stack |

**The final word: the platform follows the stack.** Both products are excellent at what they are, and the choice between them is rarely about capability — it is about which stack your enterprise already runs and which problem you are buying for. If your stack is Nutanix HCI and your need is sovereign LLM serving with RAG and guardrails, Nutanix Enterprise AI is the platform that follows your stack. If your stack is OpenShift and your need is the full MLOps lifecycle across many teams, OpenShift AI is the platform that follows your stack. Choose the platform for the stack you have and the layer you need — verify the flagged economics and the current component lists — and treat the application layer as your own (Nutanix's own FAQ says so, and Red Hat's ecosystem assumes it). Whichever you pick, the deeper playbooks in this repo ([on_prem_llm_deployment_guide.md](ai_llm/on_prem_llm_deployment_guide.md), [gpu_optimization_guide.md](gpu_optimization_guide.md), [htx_ngine_guide.md](htx_ngine_guide.md)) are where the real work continues.

---

## 8. Glossary

| Term | Definition |
|---|---|
| **Nutanix Enterprise AI (NAI)** | Nutanix's AI platform (launched Nov 2024, Verified): UI/API-driven LLM inference endpoint deployment, curated validated models, NVIDIA NIM integration, RAG blueprints and guardrails; the AI layer of GPT-in-a-Box 2.0 on the Nutanix Cloud Platform |
| **OpenShift AI (RHOAI)** | Red Hat's enterprise MLOps platform (renamed May 2023, Verified): notebooks, pipelines, KServe serving, Model Registry, TrustyAI on OpenShift; the commercial downstream of Open Data Hub |
| **Red Hat** | The open-source enterprise software company (IBM subsidiary since 2019) behind RHEL, OpenShift, and OpenShift AI |
| **Open Data Hub (ODH)** | Red Hat-led open-source AI/ML platform project on OpenShift (2018, Verified); upstream community for OpenShift AI components; docs archived March 2026 in favour of OpenShift AI Self-Managed docs |
| **Kubeflow** | The open-source ML toolkit for Kubernetes (initially Google, April 2018) whose components (notebooks, pipelines, Katib, KServe) underpin both RHOAI and the Charmed Kubeflow comparison |
| **KServe** | Kubernetes-native model-serving project (formerly KFServing): InferenceService CRs, canary, autoscaling; the serving engine in both RHOAI and Kubeflow stacks |
| **Notebooks** | Interactive coding environments (JupyterLab etc.) for experimentation; RHOAI's are Workbenches; NAI does not ship them |
| **Pipelines** | Orchestrated ML workflow engines; RHOAI's Data Science Pipelines (Tekton, KFP-compatible API); NAI does not ship them |
| **Model serving** | Running trained models behind inference APIs; NAI via NIM microservices; RHOAI via KServe (+ vLLM) |
| **GPT-in-a-Box** | Nutanix's turnkey on-prem GenAI stack, announced 15 Aug 2023 (Verified); GPT-in-a-Box 2.0 (2024) productises it with Nutanix Enterprise AI + NKP |
| **NVIDIA** | The GPU/AI-hardware and software company (CUDA, AI Enterprise); the acceleration anchor of NAI's ecosystem |
| **NIM** | NVIDIA Inference Microservices — containerised, optimised inference runtimes for catalog models; integrated with NAI via the May 2024 collaboration (Verified) |
| **AOS** | **Nutanix AOS**: the Nutanix Cloud Platform's distributed operating system (storage, services) — the "OS" layer of HCI |
| **AHV** | **Nutanix AHV**: Nutanix's enterprise hypervisor (KVM-based), the compute-virtualisation layer of HCI |
| **HCI** | Hyperconverged infrastructure: compute + storage + virtualisation as software on commodity servers; the category Nutanix invented and NAI extends |
| **MLOps** | Machine-learning operations: the lifecycle practice (experiment, train, serve, monitor, govern) that RHOAI productises and NAI partially covers |
| **RAG** | Retrieval-Augmented Generation: grounding LLM answers in retrieved documents (embeddings + vector DB + rerank); first-class in NAI blueprints, assemble-yourself on RHOAI |
| **Guardrails** | Safety controls on model input/output; NAI ships inference-path safety guardrails (Verified, May 2025); RHOAI's governance comes via TrustyAI and the application layer |
| **Platform** | In this guide, the product layer that hosts AI workloads — the thing being compared (NAI vs RHOAI) |
| **Comparison** | The head-to-head method used here: positioning, architecture, stack, features, economics, selection, worked example |

---

## 9. Claims Status, References and Disclaimer

### 9.1 Claims Status

| Claim | Status |
|---|---|
| NAI launched November 2024; a component of GPT-in-a-Box 2.0 | **Verified** (Nutanix blog "Introducing Nutanix Enterprise AI"; TechAfrica News 13 Nov 2024) |
| GPT-in-a-Box announced 15 Aug 2023 | **Verified** (cross-ref [nutanix_products_guide.md](nutanix_products_guide.md) §8.1) |
| Nutanix × NVIDIA collaboration (NIM with GPT-in-a-Box 2.0) announced 21 May 2024, .NEXT Barcelona | **Verified** (Nutanix/Business Wire press release) |
| NAI features: UI endpoint deployment, curated models (HF + NVIDIA), NIM, RAG blueprints, safety guardrails (May 2025 release) | **Verified** (Nutanix blog, May 2025; StorageNewsletter 8 May 2025; Tech Field Day) |
| NAI FAQ: Nutanix provides validated LLMs + secure APIs, not GenAI applications | **Verified** (Nutanix FAQ page) |
| NAI runs on NKP (ex-Karbon) on AHV; any-K8s cloud deployment; AI Ready partner program | **Reported** (Nutanix blog/NCP-AI training material; tech news coverage) |
| NAI lacks notebooks/pipelines/registry/explainability out of the box; AMD-GPU support; NAI pricing | **Flagged** — not verifiable from public sources; check current datasheet/quotes |
| RHOAI renamed from RHODS, announced 23 May 2023 | **Verified** (Red Hat press release) |
| ODH origins 2018 (Mass Open Cloud PoC, OpenShift/OpenStack); ODH docs archived March 2026 | **Verified** (Red Hat Emerging Technology blog, 19 Sept 2018; opendatahub.io) |
| RHOAI components: Workbenches, DSP (Tekton), KServe, Training Operator, Model Registry, TrustyAI, Kueue; ModelMesh deprecation 2.19, CodeFlare 2.24 | **Verified** (inherited from [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md), itself verified against Red Hat docs) |
| RHOAI version numbering (2.5→2.19→2.25→3.x) | **Verified** numbering via release/CVE coverage; exact 3.x GA dates **flagged** |
| OpenShift runs on Nutanix AHV as infrastructure provider | **Reported** (OpenShift platform-provider support; cross-ref k8s guides) |
| RHOAI pricing ~$12K–$80K+/yr production; NAI no public list price | **Flagged** — estimates for planning, not quotes (cross-ref charmed guide §12) |
| NAI adoption/customer counts | **Flagged** — no verified public figures |

### 9.2 References

1. **Nutanix — Introducing Nutanix Enterprise AI** — https://www.nutanix.com/blog/introducing-nutanix-enterprise-ai
2. **Nutanix — Nutanix Enterprise AI FAQs** — https://www.nutanix.com/en_sg/products/nutanix-enterprise-ai/faq
3. **Nutanix + NVIDIA collaboration press release (21 May 2024)** — https://www.nutanix.com/press-releases/2024/nutanix-and-nvidia-collaborate-to-accelerate-enterprise-ai-adoption (also Business Wire)
4. **Nutanix — Nutanix Enterprise AI Makes Agents... (May 2025)** — https://www.nutanix.com/blog/nutanix-enterprise-ai-makes-agents
5. **StorageNewsletter — NEXT 2025: Nutanix Enables Agentic AI Anywhere** — https://www.storagenewsletter.com/2025/05/08/next-conference-nutanix-enables-agentic-ai-anywhere-with-latest-release-of-nutanix-enterprise-ai/
6. **TechAfrica News — Nutanix Enterprise AI for Hybrid Multicloud (13 Nov 2024)** — https://techafricanews.com/2024/11/13/nutanix-expands-ai-capabilities-with-new-enterprise-ai-solution-for-hybrid-multicloud/
7. **Red Hat — OpenShift AI Accelerates Generative AI Adoption (23 May 2023)** — https://www.redhat.com/en/about/press-releases/red-hat-openshift-ai-accelerates-generative-ai-adoption-across-hybrid-cloud
8. **Red Hat Emerging Technology — A Hub for Open Data at Mass Open Cloud (19 Sept 2018)** — https://next.redhat.com/2018/09/19/a-hub-for-open-data-at-mass-open-cloud/
9. **Red Hat — Building an open ML platform with OpenShift and Open Data Hub (2019)** — https://www.redhat.com/en/blog/building-an-open-ml-platform-with-red-hat-openshift-and-open-data-hub-project
10. **Open Data Hub** — https://opendatahub.io/docs (archived March 2026) and https://github.com/opendatahub-io
11. **Tech Field Day — Nutanix at AI Infrastructure Field Day 2** — https://techfieldday.com/appearance/nutanix-presents-at-ai-infrastructure-field-day-2/
12. **The Nutanix Bible — Nutanix Enterprise AI** — https://nutanixbible.com
13. **IBM Community — OpenShift AI 2.19 on IBM Cloud RHOAI** — https://community.ibm.com/community/user/blogs/sugandha-agrawal/2025/07/24/rhoai219
14. **CVE coverage confirming RHOAI 2.19–2.21 and 2.25/3.3–3.5 versions** — dbugs.ptsecurity.com PT-2025-40011; kkm-mako.com openshift-ai-cve; app.opencve.io CVE-2026-18620
15. **Kubeflow (Wikipedia — contributors incl. NVIDIA and Nutanix; initial release Apr 2018)** — https://en.wikipedia.org/wiki/Kubeflow
16. **Sibling head-to-head (pattern + RHOAI component detail)** — [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md)
17. **Nutanix portfolio survey (GPT-in-a-Box §8, licensing §9, bank worked example §11)** — [nutanix_products_guide.md](nutanix_products_guide.md)
18. **OpenShift AI alternatives landscape (Nutanix as Kubeflow-support vendor)** — [openshift_ai_alternatives_guide.md](openshift_ai_alternatives_guide.md)
19. **Sovereign AI (HTX NGINE, Nutanix as platform layer)** — [htx_ngine_guide.md](htx_ngine_guide.md)
20. **On-prem LLM deployment playbook** — [on_prem_llm_deployment_guide.md](ai_llm/on_prem_llm_deployment_guide.md); **GPU compute** — [gpu_optimization_guide.md](gpu_optimization_guide.md); **cost lens** — [finops_guide.md](finops_guide.md)

### 9.3 Disclaimer

Pricing figures are estimates based on publicly available information and may vary with licensing agreements, geography, support tiers, and negotiating position — verify with Nutanix and Red Hat sales teams (both products' pricing is flagged approximate in §4.4). Product features referenced are those documented as of August 2026 research; NAI's releases and RHOAI's versioning have been moving quickly (NAI agentic release May 2025; RHOAI 3.x during 2026) — confirm current versions and component lists before architecting. Where this guide inherits verified facts from sibling guides, it cites them rather than re-verifying; where claims are Reported or Flagged, they are marked as such in §9.1.
