# AI Platform Engineering: The Platform Discipline for the AI Era — A Comprehensive Guide

> **The AI-platform deep-dive: what an AI platform is, why platform engineering is the discipline that builds it, what the layered AI-platform stack looks like, which LLMOps components plug into it, how it evolves from MLOps, how self-service and golden paths make it usable, what it means for a bank, and how to design one — closing the repository's standing Platform-Engineering skill gap with the AI angle.**

**Author:** Jack Liu Shurui  \
**Role:** Solution Architect, Cymbal Bank  \
**Audience:** Enterprise Architects, AI/ML Platform Teams, Platform Engineers, Data Architects, CTO Office  \
**Version:** 1.0  \
**Last Updated:** August 2026

---

## Series Context: Where This Guide Sits

This guide closes **Gap #4 — Platform Engineering for data** in `data_architect_skillgaps_guide.md` (the last open standing priority; Zero Trust was closed by `zero_trust_network_architecture_guide.md`). The skill-gap scan named exactly the gap this guide fills: *"the self-serve data platform layer (IDP for data, golden paths for pipelines, data-product platform, mesh enablement) — the missing operational half"* — but with the **AI angle** the user requested: the internal developer platform for AI.

The repository already owns the AI-infrastructure pieces. This guide is the **AI-platform discipline deep-dive** that binds them together:

| The repo owns… | The guide that owns it | This guide's relationship to it |
|---|---|---|
| The model gateway (routing, security, cost, observability) | `enterprise_ai_gateway_guide.md` | The gateway is **one component of the platform** — cross-ref'd in §4.1, not re-derived |
| The model registry / model hub | `huggingface_vs_csghub_guide.md` | The registry is **one component of the model layer** — cross-ref'd in §3.3 |
| The serving layer (vLLM, TGI, TensorRT-LLM, latency levers) | `low_latency_genai_patterns_guide.md` | The serving **layer of the stack** — cross-ref'd in §3.4 |
| The MLOps lifecycle (CRISP-DM, phases, pipelines) | `mlops_lifecycle_frameworks_guide.md` | The **evolutionary ancestor** — cross-ref'd in §5 |
| The ML platform tooling (MLflow, Kubeflow, SageMaker, Vertex AI, Azure ML) | `ml_platforms_comparison_guide.md` | The **model-layer tooling landscape** — cross-ref'd in §3.3 |
| The agentic workflows, MCP, agent-runtime guides | `agentic_workflows_guide.md`, `ai_llm/mcp_framework_tools_guide.md`, `ai_llm/production_ready_llm_agents_guide.md` | The **orchestration layer** — cross-ref'd in §3.5 |
| The eval frameworks (DeepEval, RAGAS, TruLens) | `ai_llm/llm_evaluation_frameworks_guide.md`, `ai_llm/rag/ragas_guide.md`, `ai_llm/rag/trulens_guide.md` | The **eval component** — cross-ref'd in §4.3 |
| The guardrails and LLM security | `llm_development_risks_security_guide.md` | The **guardrails component** — cross-ref'd in §4.4 |
| The governance layer (RAI, trust) | `responsible_ai_frameworks_guide.md`, `ai_trust_assessments_guide.md` | The **governance layer** — cross-ref'd in §3.6 |
| The banking AI requirements | `../banking/ai_genai_banking_compliance_guide.md` | The **banking context** — cross-ref'd in §7 |
| The platform-team/topology angle | `../management/high_performance_team_guide.md` | The **platform-team canon** — cross-ref'd in §2.5 |

**What this guide owns that no sibling does:** (1) the AI-platform concept — the internal developer platform for AI; (2) the platform-engineering canon — IDP, Backstage, golden paths, platform-as-a-product, platform teams; (3) the six-layer AI-platform stack as a *platform* (not a tool survey); (4) the LLMOps components as *platform capabilities*; (5) the MLOps → LLMOps → AI-platform taxonomy; (6) self-service and the prompt-to-production golden path; (7) the banking context for the AI platform; (8) a complete worked design.

## How to Use This Guide

The guide is written to be read two ways:

- **Top-to-bottom** — the canonical order: concept (§1) → canon (§2) → anatomy (§3) → components (§4) → evolution (§5) → self-service (§6) → banking (§7) → design (§8) → summary (§9). This is the recommended read for an architect building the mental model.
- **Question-first** — for the working architect, each section answers a specific question:

| If you are asking… | Read |
|---|---|
| "What exactly is an AI platform, and how is it different from an AI gateway / MLOps?" | §1, §5 |
| "What is platform engineering, and what are IDP, Backstage, golden paths?" | §2 |
| "What layers does an AI platform have, and what goes in each?" | §3, §3.8 (the service catalog) |
| "How do the gateway, prompts, evals, and guardrails fit together?" | §4, §4.6 (the traced request) |
| "Where does my bank sit on the platform maturity ladder?" | §5.3 |
| "How do developers actually use it — what is prompt-to-production?" | §6 |
| "What do the EU AI Act, MAS, and model-risk mean for the platform?" | §7, `../banking/ai_genai_banking_compliance_guide.md` |
| "Show me a concrete design I can adapt" | §8 |
| "Give me the one-page version" | §9 |

**Conventions:** cross-refs are relative links to sibling guides (the repository's technology guides live in `technology/`, with `ai_llm/` and `../banking/`, `../management/` prefixes for the other directories); verified claims are marked in the body and audited in the Appendix: Verification Notes, where anything not re-verified this session is flagged honestly.

---

## Table of Contents

- [How to Use This Guide](#how-to-use-this-guide)
1. [The Platform Overview](#1-the-platform-overview)
2. [The Platform-Engineering Canon](#2-the-platform-engineering-canon)
3. [The Platform Stack](#3-the-platform-stack)
4. [The LLMOps Components](#4-the-llmops-components)
5. [The Evolution](#5-the-evolution)
6. [The Self-Service](#6-the-self-service)
7. [The Banking Context](#7-the-banking-context)
8. [The Worked Example — An AI-Platform Design](#8-the-worked-example--an-ai-platform-design)
9. [The Summary — One Page](#9-the-summary--one-page)
10. [Glossary](#10-glossary)
- [References (Primary Sources)](#references-primary-sources)
- [Appendix: Verification Notes](#appendix-verification-notes)

---

## 1. The Platform Overview

### 1.1 The AI-Platform Definition

An **AI platform** is the **internal developer platform (IDP) for AI** — the sum of all the technology, tooling, and services that a platform team binds together to **pave golden paths for AI developers**: from prompt and dataset, through experiment and evaluation, to a governed, served, observable model or agent in production. It is platform engineering applied to the AI lifecycle: the same discipline that gave enterprises self-service infrastructure for software now gives them self-service AI.

The definition unpacks into three claims, each of which structures this guide:

1. **It is a *platform*, not a tool.** An AI platform is not a gateway, not a registry, not an eval suite, not an MLOps package — it is the *binding* of all of them into one productized, self-service whole. The platform-engineering canon (§2) supplies the binding logic: golden paths, platform-as-a-product, platform teams.
2. **It is for *AI developers*, with the AI-specific lifecycle.** The "developer" is the AI engineer, ML engineer, data scientist, or business analyst building LLM applications, agents, or predictive models. The lifecycle is the *prompt-to-production* path (§6): sandbox → data → model → eval → gateway → deploy → observe, with governance in the path, not bolted on.
3. **It is *internal*, an IDP.** Like any IDP, it exists to reduce cognitive load and enable self-service — but for workloads that are probabilistic, token-billed, GPU-bound, and regulatorily sensitive. That is what makes the AI platform a distinct discipline from both classic platform engineering and classic MLOps (§5).

### 1.2 The AI Platform vs the Adjacent Concepts

The fastest way to see the AI platform is to say what it is *not*:

| Concept | What it is | How it relates to the AI platform |
|---|---|---|
| **AI gateway** | The centralized control plane for model access — routing, security, cost, observability (see `enterprise_ai_gateway_guide.md`) | A **component inside** the platform's serving/governance layers — the platform's enforcement point |
| **Model registry / hub** | The versioned store of models, datasets, prompts (see `huggingface_vs_csghub_guide.md`) | A **component of the model layer** — the platform's artifact store |
| **MLOps platform** (MLflow, Kubeflow, SageMaker…) | Model-lifecycle tooling for training, tracking, deployment (see `ml_platforms_comparison_guide.md`) | The **model layer's tooling**, absorbed into the platform |
| **PaaS** (Heroku-class) | Application runtime with managed infrastructure | The **infrastructure layer's ancestor** — the AI platform adds AI-specific layers on top |
| **LLMOps stack** | The operational practice + tooling for LLM apps (prompts, evals, guardrails, tokens) | The **operational layer** of the platform — see §4 |
| **AI platform** | The IDP for AI: all of the above, bound together, self-served, golden-pathed | **This guide** |

### 1.3 The Overview Table

| Aspect | Description |
|---|---|
| **Definition** | The internal developer platform for AI — the productized, self-service binding of the AI infrastructure, data, model, serving, orchestration, and governance layers into golden paths for AI developers |
| **Core promise** | From prompt to production on a paved road: any AI team ships a governed, evaluated, observable AI capability without becoming an expert in GPUs, gateways, guardrails, or compliance |
| **Primary users** | AI engineers, ML engineers, data scientists, prompt engineers, business analysts (low-code), and the stream-aligned product teams they sit in |
| **Primary artifact** | The golden path (prompt-to-production) plus the platform layers beneath it — the portal is the shop window, not the product |
| **Key layers** | Infrastructure → data → model → serving → orchestration → governance (§3) |
| **LLMOps components** | Model gateway, prompt management, eval platforms, guardrails (§4) |
| **Relationship to the IDP** | The AI platform *is* an IDP — the canonical IDP extended with AI-specific layers and golden paths (§2.1, §3) |
| **Relationship to the AI gateway** | The gateway is the platform's control plane for model traffic — the platform owns it, teams consume it (`enterprise_ai_gateway_guide.md` §1) |
| **Relationship to MLOps** | MLOps is the model-lifecycle ancestor; the AI platform is the productized, self-service successor (§5) |
| **Success metric** | Adoption and time-to-production: golden-path usage, self-service rate, eval pass rate, time from prompt to production |
| **Banking driver** | Regulatory requirements (EU AI Act, MAS, model-risk) become *platform features* — the paved road is the only road that is also compliant (§7) |

### 1.4 Why Now — The Four Drivers

The AI platform is not a vendor category that happened; it is the answer to four simultaneous pressures that no earlier construct absorbs:

1. **The GPU-economics pressure.** Inference and fine-tuning are the most expensive compute an enterprise has ever run on demand — token-billed, capacity-scarce, and wasted when every team provisions its own. The platform is the only place where GPU capacity, token spend, and model choice can be metered, routed, and optimized together (the infrastructure meter of §3.1 meeting the gateway's token meter of §4.1). Without a platform, AI cost is unbudgetable.
2. **The shadow-AI pressure.** When the golden path does not exist, developers use the workaround: public LLM APIs, personal keys, ungoverned prompts — the exact risk profile a bank cannot tolerate (cross-ref `../banking/ai_genai_banking_compliance_guide.md` and `llm_development_risks_security_guide.md`). The platform's self-service is the sanctioned alternative that makes shadow AI unnecessary.
3. **The agent wave.** Agents (§3.5) multiply the governance surface: every tool call is an action, every action is a potential control breach. A lone team cannot build the sandboxing, entitlements, and tracing an agent runtime requires — those are platform-scale concerns (`ai_llm/production_ready_llm_agents_guide.md`, `ai_llm/llm_agents_failures_production_guide.md`).
4. **The regulatory pressure.** The EU AI Act, MAS, and model-risk expectations make *evidence* the production bottleneck: every model, prompt, and agent needs documented lineage, evaluation, and oversight. The platform generates that evidence as a by-product of the golden path (§7) — the only scalable way to be auditable.

The four drivers explain the taxonomy of §5: MLOps answered the first AI wave (models), LLMOps answered the second (LLM systems), and the AI platform answers the third — when AI stops being a special project and becomes a general enterprise capability, it needs a paved road like every other capability.

---

## 2. The Platform-Engineering Canon

The AI platform inherits its entire operating model from platform engineering. This section is the canon — the five ideas that define the discipline — verified against primary sources.

### 2.1 The IDP — The Internal Developer Platform

**Verified definition (Humanitec / Kaspar von Grünberg, who coined the term):** *"An Internal Developer Platform (IDP) is the sum of all the tech and tools that a platform engineering team binds together to pave golden paths and enable self-service for developers. IDPs reduce ticket ops, standardize config setups, cut lead time and time to market. Well-designed IDPs follow a platform-as-a-product approach."*

The IDP concept answers a specific failure. DevOps ("you build it, you run it" — Amazon CTO Werner Vogels, 2006; the term coined by Patrick Debois at DevOpsDays Ghent, 2009) shifted operational load onto developers at exactly the moment the toolchain exploded — Kubernetes, Terraform, the CNCF landscape. The result was **cognitive load crisis**: developers drowning in infrastructure detail for tasks as simple as changing an environment variable. The platform-engineering answer, per the canon:

- A **platform team** owns the shared substrate so product teams don't have to (Team Topologies, 2019 — §2.5).
- The substrate is built **as a product** for its users, the developers (Thoughtworks Technology Radar first flagged platform product teams in 2017; Netflix launched its platform-engineering initiative in 2017 — both verified via Humanitec's canon article).
- The product is the **IDP**: a coherent layer over the tooling, delivered as self-service.

**IDP ≠ portal.** A portal (Backstage, §2.2) is the *interface* to an IDP — a catalog, a UI, a set of templates. The IDP is the *binding* of the underlying tech: infrastructure orchestration, application configuration, deployment management, environment management, and RBAC (Humanitec's five functional areas). Many "IDP projects" fail because they build a portal and call it a platform — the portal without the paved automation behind it is a catalog of work, not a removal of work (§2.3, §8.5).

**For the AI platform:** the IDP's five functional areas map directly — infrastructure orchestration (GPU clusters), application configuration (model/prompt config), deployment management (serving rollouts), environment management (sandboxes), RBAC (model-access entitlements). The AI platform is the IDP with the AI layers added (§3).

### 2.2 Backstage — The Spotify 2020 Open-Source, the CNCF

**Verified:** Backstage is *"an open source framework for building internal developer portals (IDPs)"* — created by Spotify, open-sourced in **2020**, and a **CNCF Incubating Project** (Spotify Engineering, verified 2026). GitHub: 34.3k+ stars, Apache-2.0 license, ~75k commits, backed by the CNCF/Linux Foundation governance.

What Backstage contributed to the canon:

1. **The software catalog** — a centralized, plugin-extensible registry of every software component (services, libraries, infrastructure), which "restores order to your microservices and infrastructure." At Spotify itself: **2,700+ engineers managing 14,000+ software components** through it (verified via Spotify Engineering).
2. **The portal as an aggregation layer** — Backstage unifies the tooling sprawl (CI, cloud, monitoring, docs) behind one UI with plugins, instead of N dashboards.
3. **The template pattern** — "software templates" encode golden paths: a developer clicks "create a new service" and gets a scaffold with the organization's defaults baked in.

**For the AI platform:** Backstage (or a Backstage-class portal — Portal for Backstage, or commercial IDP portals) becomes the **front door** of the AI platform: the catalog registers models, prompts, agents, and AI services as first-class entities; templates scaffold *AI services* ("new RAG service", "new agent") with the bank's guardrails pre-wired; the portal surfaces the golden path. The portal is the shop window — the paved road behind it is the platform (§6).

### 2.3 The Golden Paths — The Paved Roads

**Verified definition (platformengineering.org):** *"A golden path is a preconfigured, paved road that provides an end-to-end workflow for developers, enabled via an Internal Developer Platform (IDP)… a predefined route that guides developers through common tasks, designed to reduce cognitive load and ensure that they can operate safely and in compliance."* Red Hat's framing: an *"opinionated, well-documented, and supported way of building and deploying software"* — explicitly including "AI/MLOps teams" among its users.

The **paved-road** metaphor is the older sibling: Google SRE culture long described the *"paved road"* — the standardized, supported, default-safe path (the path where "if you follow it, everything works and is supported") — and Evan Bottcher's widely-read essay *"Paved Road in the Jungle"* (2018) brought it into the platform discourse: internal platforms should make the *desired* way of doing things the *easiest* way, not the enforced way. The nuance (per the Levelop primer): a **paved road** is the underlying supported automation; a **golden path** is the opinionated end-to-end route built on it; **guardrails** are the automated constraints that keep users on it.

The strategic logic of golden paths: **prohibition fails, attraction works.** Developers route around anything slower than their workaround — so the golden path must be the fastest, easiest, best-supported route, and the alternatives must be *more* expensive (slower reviews, no support, no access). For a bank, this is the mechanism that makes compliance the path of least resistance: the compliant way is the paved way (§7).

**For the AI platform:** the golden path is the **prompt-to-production** route (§6.1) — the end-to-end route from "I have an idea" to "my model/agent is live, evaluated, governed, observable" — with each stage's complexity absorbed by the platform.

### 2.4 The Platform-as-a-Product

Platform-as-a-product is the operating model of platform engineering: the platform is **a product**, its users (internal developers) are **customers**, and the platform team runs it with product-management discipline — discovery, roadmap, feedback loops, metrics — rather than as a "shared service" that is funded once and ignored.

The canon's key moves:

- **Product thinking** (Marty Cagan's *Inspired* is the standard reference for the product discipline, applied *lightly* to internal platforms — the same discovery/iteration muscle): talk to customers (developers), find the jobs-to-be-done, ship incrementally.
- **The platform has a roadmap and a backlog**, prioritized by user impact, not by internal politics.
- **Adoption is the success metric.** A platform nobody uses is infrastructure theater. DORA metrics (deployment frequency, lead time, change failure rate, MTTR) and developer-experience (DX) surveys are the platform's KPIs.
- **Internal SLAs and support** — the platform team runs the paved road 24/7 and owns its reliability (cross-ref `zero_downtime_system_design_guide.md` for the availability discipline applied to the platform itself).

**For the AI platform:** the "customers" are AI teams with distinct personas (AI engineer, ML engineer, data scientist, business analyst). The product is the prompt-to-production path plus the layer services (model gateway as a service, eval as a service, sandbox as a service). The roadmap is driven by the AI use-case pipeline of the business — in a bank, the regulatory and business teams are stakeholders of the product too (§7, §8.4).

### 2.5 The Platform Teams — Cross-Ref the High-Performance Team Guide

**Verified:** Matthew Skelton and Manuel Pais's *Team Topologies* (2019) — "often hailed as the definitive guide to platform engineering" (Humanitec) — defines the **platform team** as one of four fundamental team types: the platform team *"provides internal services to reduce the cognitive load of stream-aligned teams"* — where **stream-aligned teams** own a slice of business value end-to-end. The platform team is the *enabling* layer that lets stream-aligned teams move fast without absorbing infrastructure complexity.

The team-topology angle is cross-referenced in depth by `../management/high_performance_team_guide.md` (the HPT guide's §7 engineering-teams and §8 team-design sections — including the platform-team type and its effectiveness conditions). For this guide's purposes, the platform-team pattern is:

- **One platform team (or a small set of platform teams), many stream-aligned AI teams.** The AI platform team owns the layers and the golden path; the stream-aligned squads (e.g., the trade-surveillance AI squad, the client-onboarding agent squad) own their use cases.
- **Clear team APIs** — the platform team's interface is the golden path and the platform services, not a ticket queue (or the ticket queue is the smell that the platform failed).
- **Product ownership inside the platform team** — a product manager for the platform, engineering leads per layer, and an SRE/availability arm (cross-ref `zero_downtime_system_design_guide.md`).
- **Thinnest viable platform** — the platform team builds the thinnest layer that removes the pain, then iterates; a platform team that builds for a year without users is the canonical failure mode (Team Topologies' "platform-first trap").

The AI-platform team composition and its effectiveness conditions are worked concretely in §8.4.

### 2.6 The Canon Table

| Canon element | Origin (verified) | Definition | Role in the AI platform |
|---|---|---|---|
| **Internal Developer Platform (IDP)** | Humanitec (Kaspar von Grünberg, 2021 — term coined); platformengineering.org | The sum of tech and tools a platform team binds together to pave golden paths and enable self-service | The AI platform *is* an IDP — the IDP extended with AI layers (§3) |
| **Backstage** | Spotify, open-sourced 2020; CNCF Incubating (verified) | Open-source framework for internal developer portals: catalog, templates, plugins | The portal/front door; catalog of models, prompts, agents; AI service templates |
| **Golden path** | platformengineering.org; Red Hat; Spotify's "Golden Path" practice | Preconfigured, paved, end-to-end, supported workflow; reduces cognitive load; safe and compliant by default | The prompt-to-production route (§6.1) |
| **Paved road** | Google SRE culture; Evan Bottcher, *Paved Road in the Jungle* (2018) | The standardized, supported default path — the desired way is the easiest way | The platform's underlying automation; attraction over prohibition |
| **Platform-as-a-product** | Thoughtworks Technology Radar (2017); product-management canon (Cagan) | The platform is a product; developers are customers; adoption is the metric | The AI platform's operating model: roadmap, DX surveys, SLAs (§8.4) |
| **Platform team** | *Team Topologies* (Skelton & Pais, 2019) | A team type that provides internal services to reduce stream-aligned teams' cognitive load | The AI platform team + stream-aligned AI squads (§8.4); cross-ref `../management/high_performance_team_guide.md` |

### 2.7 The Canon in Practice — How the Pieces Interlock

The five canon elements are not five separate ideas; they are one mechanism with five moving parts. The mechanism, in causal order:

1. **Cognitive load** is the problem the whole canon exists to solve (the developer who must master Kubernetes + the CNCF landscape + model serving + compliance to ship one AI feature).
2. **The platform team** (Team Topologies) is the organizational answer: a team whose entire job is to absorb that load for others.
3. **Platform-as-a-product** is the operating discipline that keeps that team honest: it must discover, ship, and measure like a product team, because a platform team without customers becomes a bureaucracy.
4. **The IDP** is the artifact that team builds: the bound-together substrate of tooling and services (§2.1's five functional areas).
5. **Golden paths and the portal** are the delivery surfaces: the golden path is the paved route through the IDP, and the portal (Backstage-class, §2.2) is the shop window that makes the path discoverable.

The classic failure sequence is the canon inverted: an organization buys (or builds) a portal first, calls it "the platform," skips the product discipline, and wonders why adoption stalls at the pilot. The canon's order matters: **team → product → substrate → path → portal**. Every subsequent section of this guide assumes that order — the AI platform is the substrate (§3), its components are the services (§4), its path is prompt-to-production (§6), and its portal is the front door (§8.2).

---

## 3. The Platform Stack

The AI platform is a **layered architecture** — six layers, from silicon to policy. Each layer is owned by the platform, consumed by golden paths, and cross-referenced to the repository guide that already deep-dives it. The rule of this guide: **name the layer, place the components, cross-ref the depth.**

### 3.1 The Infrastructure Layer — The GPU and the Cluster

The foundation: GPU clusters, Kubernetes (or GPU-aware container orchestration), high-speed interconnect (NVLink/NVSwitch fabrics, RDMA), object/parallel storage for checkpoints and datasets, and the capacity-management machinery that makes GPUs a schedulable, allocatable platform resource rather than a scarce fighting item.

This layer is deep-dived by `low_latency_genai_patterns_guide.md` (its §4 serving frameworks and the hardware/quantization levers — KV cache, speculative decoding, continuous batching, tensor/pipeline parallelism — presuppose the infrastructure layer this guide names). The platform-specific concerns *this* guide adds:

- **GPU as a service** — quota-driven, project-scoped GPU pools; preemption policies; the platform's capacity planner.
- **Environment sprawl control** — sandboxed dev/experiment environments with the same governance as prod (banking non-negotiable).
- **Cost as a first-class resource** — GPU-hours are the platform's most expensive commodity; the platform meters them (cross-ref `finops_guide.md`'s cost discipline and `enterprise_ai_gateway_guide.md`'s token-cost tracking — the infrastructure meter and the gateway meter meet in the platform's showback).

### 3.2 The Data Layer — Feature Stores and the Lakehouse

The data layer supplies what models and agents consume: curated datasets, **feature stores**, vector stores, eval datasets, and the pipelines that keep them fresh. The lakehouse (cross-ref `cloud_object_storage_lakehouse_guide.md` — the storage-and-architecture deep-dive) is the data estate the AI platform sits on; the AI platform adds the AI-specific serving of that estate.

**Feature stores — verified:** a feature store is the platform component that defines, manages, and serves ML features consistently between training and serving — solving the classic "training/serving skew" problem. **Feast** is the canonical open-source feature store: *"created at Gojek in 2018 and later co-developed with Google Cloud… now a standalone open-source project under the Linux Foundation (LF AI & Data)"* (verified via feast.dev/GitHub). Feast's contribution: a registry of feature definitions, online (low-latency) and offline (batch) serving from the same definitions, and point-in-time-correct retrieval for training datasets.

**For the AI platform:** the data layer is where RAG and agentic workloads meet the bank's data governance — curated corpora for retrieval, vector stores (cross-ref `ai_llm/rag/vector_databases_guide.md`), and the eval/ground-truth datasets that make eval-driven development possible (§4.3). The platform's data-layer services: dataset-as-a-service, features-as-a-service, embeddings/vector-as-a-service — each governed by the same lineage and access controls the lakehouse already enforces.

**The training-data service** — the skill-gap scan's Gap #10 (ML training-data lifecycle: labeling, curation, versioning — see `data_architect_skillgaps_guide.md` §4) is exactly a data-layer service the AI platform should provide: curated, versioned, licensed datasets for fine-tuning, with the same governance as features (dataset license checks before training, PII classification before prompt-building, version pinning in the experiment record so a training run is reproducible — cross-ref `mlops_lifecycle_frameworks_guide.md` for the lifecycle framing). In a bank, the training-data service is also the *audit* service: "what was this model trained on, and who approved that data?" is answered by the data layer, not by a search through notebooks.

### 3.3 The Model Layer — Registry and Experiment Tracking

The model layer is the home of the AI artifacts: models, checkpoints, prompts, and the experiments that produce them.

- **The model registry** — the versioned, governed store of models and their metadata (license, lineage, eval results, approval status). Deep-dived by `huggingface_vs_csghub_guide.md` (Hugging Face Hub vs CSGHub — the community-powerhouse vs enterprise-fortress comparison, including the registry context and self-hosting considerations). For the platform: the registry is where the governance layer's approvals attach — a model cannot be served unless its registry record says "approved."
- **Experiment tracking** — the experiment record for training and fine-tuning runs. **MLflow** (open-sourced by Databricks in 2018, now under the Linux Foundation's LF AI & Data) is the de-facto open-source tracking/registry standard; **Weights & Biases (W&B)** (founded 2017) is the commercial leader for experiment visualization and, since 2023, LLM-era prompt/trace tracking. Both are profiled in `ml_platforms_comparison_guide.md` (§2 MLflow, plus Kubeflow, SageMaker, Vertex AI, Azure ML, OpenShift AI) — cross-ref for the tool-level depth; this guide places them: tracking = the model layer's memory, registry = the model layer's shelf.
- **Training/fine-tuning platforms** — Kubeflow (Google-originated, CNCF project — per `ml_platforms_comparison_guide.md` §3) for Kubernetes-native pipelines; managed services (SageMaker, Vertex AI, Azure ML — `ml_platforms_comparison_guide.md` §5–§7) where the bank's cloud strategy allows. In a bank, fine-tuning is a governed, gated activity — the platform wraps the training tooling with approval flows.

### 3.4 The Serving Layer — vLLM, TGI, and Friends

The serving layer turns models into endpoints: inference engines, autoscaling, batching, quantization, and the latency levers. This is the deepest-dived sibling territory — `low_latency_genai_patterns_guide.md` covers vLLM (§4.1), Hugging Face TGI (§4.2), TensorRT-LLM, the latency anatomy (TTFT/TPOT), continuous batching and PagedAttention, speculative decoding, and quantization — and `on_prem_llm_deployment_guide.md` covers the deployment path.

The platform-specific concerns this guide adds: **serving-as-a-service** — the platform exposes "deploy this approved model" as a golden-path step; the developer chooses the engine (vLLM for throughput, TGI for HF ecosystem, TensorRT-LLM for max perf on NVIDIA), and the platform handles autoscaling, replica policy, and the front-door **gateway** (which routes, meters, and protects the endpoints — `enterprise_ai_gateway_guide.md`).

### 3.5 The Orchestration Layer — The Agent Runtime

The orchestration layer is the AI-platform's newest and most distinctive layer: the **agent runtime** — the execution substrate for LLM applications and agents — covering prompt pipelines, tool calling, multi-step workflows, memory/context management, MCP servers, and the sandboxing of agent actions.

Cross-refs (the cluster this guide binds): `agentic_workflows_guide.md` (the canonical design-pattern catalog — workflows as patterns), `ai_llm/mcp_framework_tools_guide.md` (the Model Context Protocol — the standard for tool/context wiring), `ai_llm/production_ready_llm_agents_guide.md` (the production-operations playbook), `ai_llm/enterprise_agentic_platform_architecture_guide.md` (the enterprise agent-platform architecture), and `ai_llm/llm_agents_failures_production_guide.md` (why agents fail — the failure modes the platform must design out). `beyond_zero_enterprise_security_guide.md` contributes the agent-governance angle (identity, least-privilege tool access).

The platform-specific concerns: **agents as deployable, governable artifacts** — an agent registered in the catalog, versioned, evaluated, and served like any other AI capability; the runtime enforces tool-access policy (an agent can only call the tools its registry record permits) and provides the observability contract for multi-step traces.

### 3.6 The Governance Layer — The RAI and Trust Layer

The governance layer is the horizontal policy plane: responsible-AI frameworks, trust/risk assessments, security guardrails, and the audit trail — embedded in every layer rather than a seventh vertical layer. Cross-refs: `responsible_ai_frameworks_guide.md` (the frameworks — EU AI Act, NIST AI RMF, MAS — as design inputs), `ai_trust_assessments_guide.md` (the trust-assessment discipline), `llm_development_risks_security_guide.md` (the security-threat catalog the guardrails answer — §4.4).

The platform-specific mechanism: **governance-as-code-in-the-path.** The golden path enforces policy automatically — a prompt cannot reach a production model without passing the guardrail pipeline; a model cannot be served without a registry approval; an agent cannot act without tool entitlements. In a bank this is not a nice-to-have: it is how the platform operationalizes the EU AI Act and MAS obligations (§7).

### 3.7 The Stack Table

| Layer | Components (platform services) | Cross-ref (the sibling that owns the depth) |
|---|---|---|
| **Infrastructure** | GPU clusters, Kubernetes, interconnect, storage, GPU-as-a-service, quotas/cost metering | `low_latency_genai_patterns_guide.md` (hardware + inference levers), `finops_guide.md` (cost discipline) |
| **Data** | Lakehouse, feature stores (Feast), vector stores, curated corpora, eval datasets, data pipelines | `cloud_object_storage_lakehouse_guide.md`, `ai_llm/rag/vector_databases_guide.md` |
| **Model** | Model registry, experiment tracking (MLflow/W&B), fine-tuning platforms (Kubeflow/SageMaker/Vertex AI), model approvals | `huggingface_vs_csghub_guide.md`, `ml_platforms_comparison_guide.md`, `mlops_lifecycle_frameworks_guide.md` |
| **Serving** | Inference engines (vLLM, TGI, TensorRT-LLM), autoscaling, quantization, endpoint management | `low_latency_genai_patterns_guide.md` (§4), `on_prem_llm_deployment_guide.md` |
| **Orchestration** | Agent runtime, workflow engine, MCP servers, tool sandboxing, memory/context, agent registry | `agentic_workflows_guide.md`, `ai_llm/mcp_framework_tools_guide.md`, `ai_llm/production_ready_llm_agents_guide.md`, `ai_llm/enterprise_agentic_platform_architecture_guide.md` |
| **Governance** | RAI frameworks, trust assessments, guardrail pipeline, audit trail, model/prompt/agent approvals | `responsible_ai_frameworks_guide.md`, `ai_trust_assessments_guide.md`, `llm_development_risks_security_guide.md`, `beyond_zero_enterprise_security_guide.md` |

### 3.8 The Platform Service Catalog — What the Layers Expose

The layers are the anatomy; the **service catalog** is what the developer actually consumes. Each layer exposes its capability as a service with a contract, an SLA, and a cost — the platform's version of an API catalog. This is the table an architect should actually design against:

| Platform service | Layer | What the developer gets | Key contract points |
|---|---|---|---|
| **GPU-as-a-service** | Infrastructure | Quota-scoped GPU pools (dev/prod), pre-approved images, capacity planning | Quota per project, preemption policy, cost per GPU-hour |
| **Sandbox-as-a-service** | Infrastructure | One-click governed experiment environments with pre-approved models and synthetic data | Data isolation, model allowlist, auto-expiry |
| **Data/features-as-a-service** | Data | Dataset entitlements, feature-store access (Feast), vector-store ingestion | Lineage, PII classification, freshness SLA |
| **Tracking-as-a-service** | Model | Managed experiment tracking (MLflow/W&B-class) with shared lineage | Experiment retention, artifact storage limits |
| **Registry-as-a-service** | Model | Model/prompt/agent registration with lineage, eval, and approval state | Approval gate, license check, immutability of approved versions |
| **Serving-as-a-service** | Serving | "Deploy this approved model" — engine choice (vLLM/TGI/TensorRT-LLM), autoscaling | TTFT/TPOT SLOs (cross-ref `low_latency_genai_patterns_guide.md`), engine limits |
| **Gateway-as-a-service** | Serving/Governance | Model access: API keys scoped to approved models, budgets, regions; routing policy | Token budget, fallback policy, residency rules (cross-ref `enterprise_ai_gateway_guide.md`) |
| **Eval-as-a-service** | Model/Governance | Shared eval suites, judges, datasets; CI-integrated eval gates | Eval definitions versioned; gate thresholds per channel |
| **Guardrail-as-a-service** | Governance | Input/output/agent-action policy enforcement on every call | Policy per channel; block/redact/quarantine actions; audit of decisions |
| **Agent-runtime-as-a-service** | Orchestration | Managed agent execution: workflows, MCP tools, sandboxing, entitlements | Tool allowlist, step limits, trace retention |
| **Portal/catalog** | All | The front door: register, discover, template, approve (Backstage-class, §2.2) | Catalog completeness, template freshness |

The service catalog is also the **chargeback/showback ledger**: every service has a meter (GPU-hours, tokens, storage, eval runs), so the platform's cost story closes the loop between the infrastructure meter (§3.1) and the gateway meter (§4.1) — the `finops_guide.md` discipline applied to AI.

---

## 4. The LLMOps Components

LLMOps is the operational discipline for LLM-based systems — and its four signature components are *platform capabilities* in the AI platform: the **model gateway**, **prompt management**, **eval platforms**, and **guardrails**. The platform's job is to productize each one: a component is "platformized" when a developer consumes it through the golden path without building it. The 2026 LLMOps tooling landscape (LangSmith, W&B, Braintrust, Arize Phoenix, Promptfoo, Guardrails AI — see the sibling `mlops_lifecycle_frameworks_guide.md` and the eval guides for tool-level depth) is exactly the component market this layer abstracts.

### 4.1 The Model Gateway — Cross-Ref the Enterprise AI Gateway Guide

The **model gateway** is the centralized control plane between AI consumers and AI providers: routing (by task, cost, quality, latency), security (auth, injection detection, PII redaction, moderation), cost management (per-token, per-team), observability (TTFT, TPS, drift), semantic caching, provider fallback, and compliance enforcement. The full architecture, platform profiles, and banking-specific requirements are deep-dived in `enterprise_ai_gateway_guide.md` — this guide does not re-derive them.

The platform-engineering view of the gateway:

- The gateway is the **serving layer's front door and the governance layer's enforcement point** — the single place where every production model call passes policy (§3.4, §3.6).
- The gateway is **platformized** — "model access" is a golden-path entitlement: a team gets an API key scoped to approved models, budgets, and regions; the gateway enforces what the registry approved.
- The gateway makes **model choice a developer decision, not an infrastructure decision**: the developer declares the task, the gateway routes to the right model at the right cost — the platform's own routing policy lives here (cross-ref `enterprise_ai_gateway_guide.md` §6).

### 4.2 The Prompt Management

Prompt management is the LLMOps component that version-controls the platform's most peculiar artifact: the prompt. A prompt is code-adjacent (it determines behavior) but not code (it is natural language with its own failure modes), so it needs its own management layer:

- **Prompt versioning and lineage** — every prompt is versioned, associated with the model it targets and the eval results that qualified it; production prompts are traceable to the experiment that produced them.
- **Prompt-as-code** — prompts live in repositories, reviewed like code, deployed through the pipeline; templates parameterize them (system prompts, few-shot examples, retrieval instructions).
- **The prompt catalog** — the governed store of approved prompt templates (the enterprise's "prompt IP"), searchable and reusable, with per-domain approval (a client-facing prompt has a different bar than an internal one).
- **Tooling** — LangSmith (LangChain's tracing/eval/prompt platform, launched 2023) is the category's reference implementation; W&B's LLM tooling (prompt-tracking/Weave, 2023–) covers the experiment side; Promptfoo-style tools bring prompts into CI. The platform wraps these: the catalog's approval state feeds the gateway's enforcement.

The banking nuance: in a regulated firm a prompt is a *controlled artifact* — versioned, approved, auditable — because the prompt determines what the model says to clients and staff. Prompt management is how the platform makes that control operational (§7).

### 4.3 The Eval Platforms — Cross-Ref the Eval Guides

Evaluation is the LLMOps component that makes AI engineering a *discipline* instead of a lottery: the platform's eval capability is **eval-as-a-service** — suites, harnesses, judges, and datasets that teams run continuously. The depth lives in the sibling guides: `ai_llm/llm_evaluation_frameworks_guide.md` (DeepEval and the LLM-evaluation framework landscape), `ai_llm/rag/ragas_guide.md` (RAG evaluation), `ai_llm/rag/trulens_guide.md` (TruLens), plus the RAG-eval methodology and tools comparisons in `ai_llm/rag/`.

The platform-engineering view:

- **Eval is the CI for AI.** The golden path runs evals at every stage — a prompt change, a model upgrade, a RAG corpus refresh, an agent-tool change — before anything reaches the gateway. "Did it pass eval?" becomes the platform's version of "did CI pass?"
- **Eval assets are platform assets** — the eval datasets (golden sets, adversarial sets), the judge models (LLM-as-judge with calibrated human agreement), and the metric definitions are shared, versioned platform services — not per-team notebooks.
- **Offline vs online** — offline evals in the pipeline; online monitoring (drift, quality scores, hallucination rates) on the serving side, with the gateway's observability feeding the online loop (cross-ref `enterprise_ai_gateway_guide.md` §9).
- **The eval gate is a governance gate** — in a bank, "passed the eval suite" is part of the model-risk record that an internal validation function reviews (§7, and `../banking/ai_genai_banking_compliance_guide.md`).

### 4.4 The Guardrails — Cross-Ref the LLM-Security Guide

Guardrails are the automated safety constraints on model inputs and outputs: prompt-injection defense, PII detection/redaction, content moderation, jailbreak resistance, output-format validation, and domain-policy checks (a bank's guardrail layer also enforces suitability and conduct rules). The threat catalog they answer is deep-dived in `llm_development_risks_security_guide.md`; the responsible-AI framing in `responsible_ai_frameworks_guide.md` and `ai_trust_assessments_guide.md`.

The platform-engineering view:

- **Guardrails are a pipeline, not a plugin** — layered: input guardrails (before the model), output guardrails (after the model), and agent guardrails (on tool actions), with the **gateway as the enforcement point** for all three (cross-ref `enterprise_ai_gateway_guide.md` §3/§7).
- **Guardrail policy is configuration, not code** — domain teams tune policies (a client-communication channel is stricter than an internal-coding channel) through the platform's policy console; the platform's guardrail service runs the policies consistently everywhere.
- **Guardrail testing is part of eval** — adversarial eval sets (injection, jailbreak) are the test suite for the guardrail pipeline itself; the platform ships the red-team eval assets (cross-ref `ai_llm/ai_governance_bias_redteaming_guide.md` and `adversarial_ml_attacks_guide.md`).

### 4.5 The Components Table

| Component | Purpose | Representative tooling (verified category) | Cross-ref |
|---|---|---|---|
| **Model gateway** | Unified, governed access to models: routing, security, cost, observability, fallback | LiteLLM-class/enterprise gateways; managed gateway services | `enterprise_ai_gateway_guide.md` (the full deep-dive) |
| **Prompt management** | Versioning, lineage, cataloging, approval of prompts | LangSmith (2023–), W&B LLM tooling (Weave/Prompts, 2023–), Promptfoo | This guide §4.2 (component placement) |
| **Eval platforms** | Offline eval suites, LLM-as-judge, eval datasets, online monitoring | DeepEval, RAGAS, TruLens, LangSmith, Braintrust, Arize Phoenix | `ai_llm/llm_evaluation_frameworks_guide.md`, `ai_llm/rag/ragas_guide.md`, `ai_llm/rag/trulens_guide.md` |
| **Guardrails** | Input/output/tool-action safety: injection, PII, moderation, policy | Guardrails AI, NeMo Guardrails-class, gateway-embedded policies | `llm_development_risks_security_guide.md`, `enterprise_ai_gateway_guide.md` §7 |

### 4.6 The Components in the Golden Path — One Request, Full Trace

The four components are not parallel tools; they are the stages of a single governed request. Following one production call through the platform shows how they compose — and why the gateway is the enforcement point:

| Stage of the request | Component at work | What happens | Evidence produced |
|---|---|---|---|
| **1. Client application calls** | Gateway | AuthN/Z, budget check, routing decision (which approved model for this task/channel) | Call record: requester, model, cost, route |
| **2. Input guardrails** | Guardrails | Injection scan, PII detection/redaction, domain policy check (e.g., suitability rules for client channels) | Guardrail decision record |
| **3. Prompt assembly** | Prompt management | The exact prompt version from the catalog (system prompt + retrieved context + user input) is locked | Prompt version ID in the trace |
| **4. Model inference** | Gateway + serving | The approved model version serves (vLLM/TGI); tokens metered | Token usage, latency, model version |
| **5. Output guardrails** | Guardrails | Moderation, PII, format validation, refusal-of-harm checks | Guardrail decision record |
| **6. Eval/monitoring** | Eval platform | Online quality signals (drift, hallucination rate, feedback) update the artifact's score | Online-eval metrics |

One request = one full trace: prompt version, model version, guardrail decisions, eval signals, cost, latency. That trace *is* the audit evidence a bank must produce on demand (§7) — the components compose into the platform's answer to "show me what this AI did and why it was allowed."

### 4.7 The Component-Selection Questions

When the platform team chooses *which* tool implements each component, four questions decide the answer — and they are platform questions, not tool questions:

1. **Where does the policy live?** The component must not become a second source of truth. If the gateway already enforces routing and guardrail policies (`enterprise_ai_gateway_guide.md` §3), the guardrail tool must be *policy-rendering* behind it — not a parallel enforcer with its own rules that can drift.
2. **Is it a component or a platform?** LangSmith-class tools and MLflow-class platforms are *components* of the model/eval layers — selecting them is selecting a layer's implementation, not "buying an AI platform." The IDP is still the binding (§2.1). Conversely, a vendor that claims to sell "the AI platform" whole is selling a portal plus components — the binding still has to happen in-house.
3. **Does it fit the golden path's evidence model?** Every component must emit the trace fields §4.6 requires (prompt version, model version, guardrail decisions, cost). A brilliant eval tool whose results cannot be attached to the registry record and the audit trail fails the platform's test.
4. **Does it respect the bank's constraints?** European hosting and data residency (§7.1), self-hostability where needed (`huggingface_vs_csghub_guide.md`), and license/governance fit — the platform's component choices are procurement decisions with compliance surface, not developer preferences.

The discipline: **choose components for their contracts, not their demos** — the contract is the trace, the policy point, and the residency box the component must tick.

---

## 5. The Evolution

### 5.1 The Taxonomy — MLOps vs LLMOps vs AI Platform

The AI platform is the third rung of an evolution — each rung inherits the previous one and adds a layer of abstraction:

**MLOps (the model lifecycle).** The discipline of operationalizing predictive ML: experiment tracking, training pipelines, model registries, deployment, monitoring. The lifecycle is deep-dived by `mlops_lifecycle_frameworks_guide.md`; the tooling by `ml_platforms_comparison_guide.md`. MLOps' artifact is the *model* (a deterministic function fitted to data); its hard problems are reproducibility, data drift, and deployment.

**LLMOps (the GenAI operations).** The discipline of operating LLM-based systems: prompts, tokens, contexts, evals, guardrails, agent traces. LLMOps' artifact is the *prompt-plus-model-plus-context* (a probabilistic system whose behavior is set as much by the prompt and the retrieved context as by the weights); its hard problems are nondeterminism, evaluation, cost-per-token, and safety. Most of what the industry calls "LLMOps platforms" (§4) are *component* tools — the LLMOps layer of a platform, not the platform itself.

**AI platform (the IDP for AI).** The productized, self-service binding of MLOps + LLMOps + infrastructure into golden paths. The AI platform's artifact is the *path*: the route from idea to governed production AI. Its hard problem is not any single layer — it is the *composition*: making the enterprise's AI lifecycle (a) fast (self-service), (b) safe (governance in the path), and (c) economic (metered, routed, cached) at the same time. Where MLOps asked "how do we run models?", the AI platform asks "how do we let every team run models, agents, and prompts on a paved road?"

### 5.2 The Evolution Table

| Era | Core artifact | Primary concerns | Signature tooling | The platform answer |
|---|---|---|---|---|
| **MLOps** (2015–) | The model | Reproducibility, drift, deployment | MLflow, Kubeflow, SageMaker, Vertex AI (`ml_platforms_comparison_guide.md`) | Model layer services (registry, tracking, training) |
| **LLMOps** (2022–) | The prompt+model+context system | Nondeterminism, eval, tokens, safety | LangSmith, W&B, DeepEval, RAGAS, guardrails (§4) | LLMOps components as platform capabilities (gateway, prompts, eval, guardrails) |
| **AI platform** (2024–) | The golden path (prompt-to-production) | Composition: self-service + governance + economics at once | IDP portals (Backstage-class), platform orchestrators, the six-layer stack (§3) | This guide — the IDP for AI |

*Note: the era dates are structural markers, not precise boundaries — most enterprises run all three rungs simultaneously today. The taxonomy matters for the interview and the design: name which rung a tool or practice belongs to, and the platform story writes itself.*

### 5.3 The Platform Maturity Model — From Ad Hoc to Federated

The evolution is not only historical; it is the **maturity ladder** every organization climbs. Assessing where a bank sits on this ladder is a standard architecture question — and the ladder doubles as the platform's roadmap:

| Level | Name | What exists | The failure mode at this level | The next step |
|---|---|---|---|---|
| **0** | Ad hoc | Every team runs its own notebooks, keys, prompts, and shadow models | Shadow AI, unbudgeted GPU spend, no evidence trail | Name the problem; fund a platform team (§2.5) |
| **1** | Shared services | A gateway exists; some models served centrally; registries begin | Governance is policy-on-paper; evals are per-team; the "platform" is a set of APIs, not a path | Build the golden path for one use case end-to-end (§8.5 lesson 1) |
| **2** | Productized | The prompt-to-production path works; portal, templates, eval gates, guardrails are services | The platform is a single team's product; stream-aligned teams are consumers, not contributors | Productize metrics: DX survey, adoption KPIs, SLOs (§2.4) |
| **3** | Federated | Multiple platform teams per domain (AI, data, security) with defined team APIs; the golden paths compose | Boundary friction: "which platform do I go to for a vector store?" | Publish the service catalog (§3.8) and the team APIs (cross-ref `../management/high_performance_team_guide.md` §8) |
| **4** | Ecosystem | The platform is the default way AI happens; off-path is the exception; suppliers (models, tools) are pluggable | Complacency: the paved road stops being repaved as the AI tooling wave moves | Keep the product loop running — the road is never finished (§2.4) |

Most banks in 2026 sit between **1 and 2**: the gateway and some central services exist, but the *path* does not — which is exactly the gap the golden path (§6.1) and the worked design (§8) close.

---

## 6. The Self-Service

### 6.1 The Golden Paths for AI — The Prompt-to-Production Path

The AI platform's golden path is **prompt-to-production**: the paved, end-to-end route every AI use case travels. The platform's design goal is that a team's *default* journey is exactly this path, and that every stage's complexity is absorbed by a platform service:

1. **Ideate & sandbox** — a governed sandbox environment with pre-approved models, a notebook/IDE, sample datasets, and the gateway's dev keys. The developer experiments without touching production infrastructure or production data (banking: synthetic data first).
2. **Data & context** — curated datasets, feature-store access, vector-store ingestion for RAG; the platform provides data entitlements, lineage, and PII classification automatically.
3. **Build & evaluate** — prompts versioned in the catalog; evals run against shared eval assets; the eval gate reports "passes for channel X."
4. **Register & approve** — the model/prompt/agent is registered in the catalog with lineage, eval results, and risk classification; the governance workflow (model-risk, compliance, security) approves in-platform.
5. **Serve** — "deploy" publishes the approved artifact behind the gateway: routing policies, budgets, guardrails, and observability are attached automatically from the registry record.
6. **Observe & iterate** — production traces, quality and drift metrics feed back; the next iteration starts the path again (cross-ref `mlops_lifecycle_frameworks_guide.md` for the lifecycle framing and `enterprise_ai_gateway_guide.md` §9 for the observability contract).

The path is the **attraction mechanism** (§2.3): it is the fastest, easiest, and the *only fully supported and fully compliant* route. Off-path options exist but are deliberately more expensive (no support, manual security review, no budget).

### 6.2 The AI-Developer Experience (AI-DX)

The AI platform's user experience is the **AI-developer experience** — the platform-engineering translation of developer experience (DX) to the AI persona set:

- **Personas** — the AI engineer (builds agents/apps), the ML engineer (fine-tunes and serves models), the data scientist (experiments), the business analyst (low-code/no-code prompt apps), and the *consumer* teams (stream-aligned squads) — each with a tailored self-service surface.
- **Surfaces** — the portal (catalog, templates, approvals — Backstage-class, §2.2), the CLI/SDK (the golden path as `ai-platform new rag-service`), and the APIs (layer services consumed programmatically). The portal is for discovery and governance; the CLI/SDK is for velocity.
- **Templates** — "new RAG service", "new agent", "new eval suite", "new prompt" scaffolds with the bank's defaults (guardrails, logging, registry wiring) pre-baked — the Backstage software-template pattern applied to AI artifacts.
- **The DX loop** — platform teams measure the AI-DX (time-to-first-eval, time-to-production, self-service rate, support-ticket rate) exactly as product teams measure DX; the DX survey is the platform's customer-feedback channel (§2.4).

### 6.3 The Self-Service Table

| Golden-path stage | Self-service capability | Platform layer/service |
|---|---|---|
| Ideate & sandbox | One-click governed sandbox, pre-approved models, dev gateway keys | Infrastructure (GPU-as-a-service), gateway |
| Data & context | Dataset/feature/vector entitlements, synthetic-data service, PII auto-classification | Data layer, governance |
| Build & evaluate | Prompt catalog, eval suites as a service, CI-integrated eval gates | Model layer, eval platform |
| Register & approve | Catalog registration, automated risk classification, in-platform approval workflow | Model registry, governance |
| Serve | One-command deploy behind the gateway; routing, budgets, guardrails auto-attached | Serving layer, gateway, guardrails |
| Observe & iterate | Production traces, quality/drift dashboards, feedback to the path | Gateway observability, orchestration |

### 6.4 The Self-Service Anti-Patterns

Self-service fails in predictable ways — the platform-engineering literature and the golden-path discussions (platformengineering.org; the "paved roads, not portals" critique) converge on the same anti-pattern set. An architect should recognize all four immediately:

1. **The portal with no road** — a beautiful catalog of forms and links that surfaces work instead of removing it; developers route around anything slower than their workaround. The IDP is the paved automation; the portal is its window (§2.1, §2.2).
2. **The paved road to nowhere** — the golden path exists but ends at the registry: deployment, monitoring, and iteration are still manual. A path is only a path if it reaches production *and* back (§6.1's observe-and-iterate step).
3. **Self-service without guardrails** — self-service is a governance surrender unless the self-served capabilities are policy-constrained by construction. The golden path's gates (eval, approvals, guardrails) are what make self-service safe in a bank (§4.4, §7).
4. **The platform that never ships** — a platform team that builds for a year without users is the canonical failure (Team Topologies' platform-first trap). The product loop — first path, first users, then breadth — is the only cure (§2.4, §8.5).

### 6.5 The AI-DX Metrics — How the Platform Knows It Works

Platform-as-a-product needs metrics (§2.4); AI platforms need AI-specific ones on top of the standard DX set:

| Metric | What it measures | The signal |
|---|---|---|
| **Time-to-first-eval** | How fast a new idea gets its first evaluation result | The platform's "hello world" latency — target minutes, not weeks |
| **Time-to-production** | Sandbox → governed production on the golden path | The headline value metric — quarters → days is the transformation (§8.3) |
| **Golden-path adoption rate** | % of AI workloads that shipped on the path vs off-path | The platform's market share; the anti-shadow-AI metric |
| **Self-service rate** | % of platform actions completed without tickets | The IDP's core promise (§2.1) |
| **Eval-gate pass rate / catch rate** | How often the eval gate blocks a change, and what it catches | Proof the gates are real (and calibrated, not ceremonial) |
| **Model call cost per use case** | Token + GPU cost per business outcome | The economics story (§1.4 driver 1) |
| **Evidence completeness** | % of production artifacts with full lineage + eval + approval records | The audit-readiness metric — the one the regulators will ask about (§7) |

The metrics are the platform team's product dashboard: published, reviewed on the product cadence, and driving the roadmap — the operational translation of §2.4.

### 6.6 The Template Gallery — The Path as Scaffolds

The golden path becomes concrete in the **templates** developers start from (the Backstage software-template pattern, §2.2, applied to AI artifacts). A bank's template gallery is a miniature of the platform's governance: each template bakes the defaults in so the paved way is the only way:

| Template | What it scaffolds | Baked-in defaults (the governance, pre-wired) |
|---|---|---|
| **RAG service** | Retrieval app: corpus connection, embedding pipeline, vector store, prompt, gateway wiring | Approved embedding models, PII-filtered corpora, guardrail profile, eval suite stub |
| **Agent** | Agent runtime: tool manifest, MCP wiring, memory, step policy, tracing | Tool allowlist placeholder, entitlement review step, sandboxed execution, trace retention |
| **Prompt app** | Prompt + channel binding: catalog entry, versioning, approval workflow | Channel risk tier, human-oversight checkpoint if client-facing, eval gate required |
| **Fine-tune** | Training run: dataset, hyperparameters, tracking, registry promotion | Approved base models, dataset license check, approval before registry promotion |
| **Eval suite** | Evaluation harness for an artifact: datasets, judges, thresholds | Shared golden/adversarial sets, LLM-as-judge config, gate thresholds per channel |
| **Batch inference** | Scheduled scoring: feature-store input, model version, output sink | Freshness SLA, drift check on output, cost budget |

Every template is versioned itself — when the bank's guardrail policy changes, the templates change, and the next scaffolded project inherits the new defaults. **Templates are how the platform ships governance at the speed of self-service.**

---

## 7. The Banking Context

### 7.1 The Cymbal Bank AI Platform — Cross-Ref the Bank Series and the Compliance Guide

The banking context for the AI platform is set by two cross-refs: `../banking/ai_genai_banking_compliance_guide.md` (the requirements map — EU AI Act, MAS, US regulators, model-risk, privacy, security — the binding constraints the platform operationalizes) and `../banking/credit_agricole_software_systems_guide.md` (the bank-series deep-dive — the group's AI estate and the "transformation IA" programme).

**The Crédit Agricole AI programme — verified (2026-08):** the group's documented AI marker is the press release *"Le Crédit Agricole accélère sa transformation IA"* (10 June 2026, presse.credit-agricole.com): **~€500 million invested over 3 years**; creation of the group **"Entreprise IA"** — an AI company that *"develops and operates the group's industrial AI platforms"* and favours European solutions; and an **"AI For All"** transformation programme embedding AI across teams. The earlier "Casa IA" internal-assistant nickname is unverified as the official programme name (flagged in the bank guide §8). The Microsoft partnership (June 2023) is verified as an AI-skills/training partnership (Microsoft France × Simplon × Crédit Agricole Consumer Finance); the group's cloud posture is Azure-anchored. Cymbal Bank's Singapore operation is a wholesale/markets/trade hub — not a retail bank — which shapes the platform's use cases (trade, markets, corporate flows) in this region.

**What this means for the AI platform:** the "Entreprise IA" mandate — *industrial AI platforms* — is precisely the platform-engineering object this guide describes: the group-level AI platform on which the federation's use cases run, with European-solution preference (a constraint on the model layer: European hosting, European model options — cross-ref `huggingface_vs_csghub_guide.md`'s self-hosting/registry discussion and `on_prem_llm_deployment_guide.md`).

**The banking requirements the platform must absorb** (from `../banking/ai_genai_banking_compliance_guide.md`): the EU AI Act's risk tiers (a bank's client-facing GenAI is high-risk territory; the platform must classify and gate accordingly), MAS's AI governance principles and the FEAT/verifiable-AI expectations in Singapore, model-risk management (the platform's eval and approval records *are* the model-risk evidence), data residency and third-party risk (the gateway's provider governance), and the auditability of every prompt, call, and decision.

### 7.2 The Banking Table

| Banking requirement (source: `../banking/ai_genai_banking_compliance_guide.md`) | The AI platform's response | Where it lands in the stack |
|---|---|---|
| EU AI Act risk classification and documentation | Risk classification at registration; automated documentation from the registry record (eval, lineage, guardrails) | Model layer, governance |
| MAS AI governance / verifiable AI (Singapore) | Eval-as-a-service as the evidence trail; human-oversight checkpoints in the golden path | Eval platform, governance |
| Model-risk management (SR 11-7-style validation) | Registry approvals gated by validation; eval reports as the model-risk record | Model registry, eval platform |
| Data residency & privacy | Regional data services, PII guardrails, synthetic-data sandboxes, residency-aware routing | Data layer, gateway, guardrails |
| Third-party & provider risk | Provider governance in the gateway: approved providers, contracts, exit paths | Gateway (`enterprise_ai_gateway_guide.md` §10) |
| Auditability | Full trace: prompt version, model version, eval results, guardrail decisions, cost, for every production call | Governance + gateway observability |
| Group platform mandate ("Entreprise IA", ~€500M/3yrs, European preference — verified 2026) | The group-level AI platform this guide designs; European hosting/models as a model-layer constraint | The whole platform (§8) |

### 7.3 The Bank Series — Platform Patterns Across the Banks

The bank series (`../banking/`) shows three distinct patterns for banking AI estates — useful as a contrast set for the platform design (§8):

- **The federation pattern (Crédit Agricole)** — the platform must serve 39 regional banks plus CASA's lines; the "transformation IA"/"Entreprise IA" programme is the industrial-platform answer to federal rollout (§7.1). Cross-ref `../banking/credit_agricole_software_systems_guide.md` §8–§9.
- **The journey pattern (DBS)** — DBS organised delivery around ~40+ end-to-end customer journeys, with a data/AI estate feeding real-time decisions (fraud, credit, personalisation) and platform engineering lifting release velocity — the journey organisation *is* the stream-aligned structure that an AI platform serves (§2.5). Cross-ref `../banking/dbs_bank_guide.md`.
- **The legacy-consolidation pattern (Deutsche)** — the AI programme rides on a legacy-consolidation wave; the platform's first job is wrapping and instrumenting the old estate before greenfield paths make sense. Cross-ref `../banking/deutsche_bank_software_systems_guide.md` §5.

The architect's takeaway: **the AI platform is not a technology choice, it is a response to the bank's organisational shape** — federation, journeys, or consolidation each dictate a different rollout order (§8.6) and a different set of first golden paths.

### 7.4 The Singapore Angle — The Hub Platform

Cymbal Bank's Singapore operation is a wholesale/markets/trade hub (§7.1), and the AI platform's Singapore face differs from the Paris face in three ways:

- **MAS as the lead regulator for the hub workloads** — MAS's AI governance principles and the FEAT (Fairness, Ethics, Accountability, Transparency) expectations apply to the hub's AI use cases; the platform's eval and evidence trail is what demonstrates verifiable AI to MAS (cross-ref `../banking/ai_genai_banking_compliance_guide.md` §2 for the MAS section).
- **Data residency is a *regional* design constraint** — Singapore-hub data (trade flows, markets data) must be served from the region; the platform's residency-aware routing (gateway policy, §4.1) is not a Paris-only concern. The hybrid estate of §8.2 gets a Singapore-region leg.
- **The hub is the platform's speed-test** — a wholesale hub with leaner hierarchies is the natural first-adopter population for the golden path (the pilot squad of §8.3 is exactly the hub profile: markets technology, fast iteration, heavy compliance surface). The bank-series lesson (§7.3) applies: start where the organisational shape is thinnest.

The Singapore angle is the reminder that an AI platform for a global bank is a **multi-region product**: the layers are the same, the policies are regional, and the golden path must be paved in every region that runs AI.

---

## 8. The Worked Example — An AI-Platform Design

### 8.1 The Scenario — A Cymbal Bank

Design an AI platform for a familiar client: **"Caisse Fédérale" (CF)** — a Cymbal Bank European wholesale bank with a Singapore hub (cross-ref `../banking/credit_agricole_software_systems_guide.md` §10 for the Cymbal Bank Singapore shape: markets, trade finance, corporate flows). The constraints are the familiar ones:

- **Regulated and audited**: EU AI Act territory, MAS AI governance in Singapore, model-risk validation on everything that touches a decision (cross-ref `../banking/ai_genai_banking_compliance_guide.md`).
- **Hybrid estate**: Azure-anchored cloud plus on-prem GPU capacity; European data residency mandatory; a European-solution preference for the AI stack (the verified "Entreprise IA" posture, §7.1).
- **A real use-case pipeline**: client-onboarding agent, trade-surveillance assistant, market-abuse detection, RAG over the policy/legal corpus, code-assist for developers, and a dozen more — currently 20 teams, each building their own "AI stack" (the sprawl the platform exists to end).
- **The mandate**: one industrial AI platform for the group; time-to-production and governance are the two non-negotiable success criteria; the platform budget is funded (cross-ref `../management/business_case_development_guide.md` for the investment case framing — the platform's business case is the sum of the use cases it unblocks).

### 8.2 The Design — The Layered Stack

**The six layers (§3), instantiated for CF:**

| Layer | CF instantiation | Why this choice |
|---|---|---|
| **Infrastructure** | Azure (EU regions) + an on-prem GPU pod (for data-residency-sensitive workloads); Kubernetes as the substrate; GPU quotas per domain | Hybrid per residency rules; one control plane across both |
| **Data** | The existing lakehouse (cross-ref `cloud_object_storage_lakehouse_guide.md`) + **Feast** feature store for the predictive models; a governed vector store for the RAG corpora; eval datasets curated per domain | Features/context served consistently; RAG corpus under data governance |
| **Model** | **MLflow** (tracking + registry) or a **CSGHub-class** self-hosted registry (cross-ref `huggingface_vs_csghub_guide.md`) given the European-preference and residency constraints; fine-tuning on the GPU pod (Kubeflow) or managed (Azure ML) | The registry record carries the approval state; European hosting respected |
| **Serving** | **vLLM** for the throughput workloads, **TGI** for the HF-ecosystem models (cross-ref `low_latency_genai_patterns_guide.md` §4); autoscaling; all endpoints behind the gateway | The latency levers are already deep-dived; the platform just exposes them |
| **Orchestration** | The **agent runtime**: MCP-based tool wiring (cross-ref `ai_llm/mcp_framework_tools_guide.md`), workflow engine (cross-ref `agentic_workflows_guide.md`), agent sandboxing per the production-agents playbook (`ai_llm/production_ready_llm_agents_guide.md`) | Agents are first-class platform artifacts with tool entitlements |
| **Governance** | The **gateway** as enforcement point (cross-ref `enterprise_ai_gateway_guide.md`), guardrail pipeline (cross-ref `llm_development_risks_security_guide.md`), RAI/trust frameworks wired in (cross-ref `responsible_ai_frameworks_guide.md`, `ai_trust_assessments_guide.md`) | Policy is configuration in the path, not a manual review queue |

**The portal:** Backstage (CNCF, §2.2) as the front door — the catalog registers CF's models, prompts, agents, and AI services; templates scaffold the golden path.

### 8.3 The Golden Path — The Prompt-to-Production Walk-Through

The trade-surveillance squad's journey (the §6.1 path, instantiated):

1. **Sandbox** — the squad clicks "new AI workspace" in the portal; a governed sandbox spins up with pre-approved models (the European-hosted set), synthetic market data, and dev gateway keys. No production data, no tickets.
2. **Data & context** — the squad connects the feature store (market-abuse features) and the vector store (the surveillance policy corpus); entitlements and PII classification apply automatically.
3. **Build & evaluate** — prompts live in the catalog; the squad's eval suite (precision/recall on a labeled surveillance golden set — cross-ref the eval guides, §4.3) runs in CI; the eval gate reports "passes for internal-surveillance channel."
4. **Register & approve** — the agent is registered with lineage, eval results, and an auto-classified risk tier (EU AI Act high-risk territory for market surveillance); model-risk and compliance approve in-platform; the evidence trail is generated automatically.
5. **Serve** — "deploy" publishes behind the gateway: routing to the approved model, budget attached to the squad, guardrails on (injection, PII, output moderation), full traceability on.
6. **Observe** — drift and quality dashboards feed the squad's iteration loop; a model upgrade re-runs the eval suite before it touches the gateway.

Every step that *used* to be a manual, ungoverned, two-month slog (GPU access, data approval, model validation, security review, deployment) is now a paved, self-service, *recorded* step. That is the platform's entire value proposition.

### 8.4 The Platform Team

**The topology** (Team Topologies, §2.5; cross-ref `../management/high_performance_team_guide.md` §7–§8 for the effectiveness conditions):

- **One AI platform team** (~10–14 people, product-shaped): a platform product manager (the "platform-as-a-product" owner, §2.4), layer leads (infrastructure, data, model, serving, orchestration, governance), an AI-DX engineer (portal/templates/CLI), and an SRE arm for the platform itself (availability per `zero_downtime_system_design_guide.md`).
- **Stream-aligned AI squads** — the use-case teams (surveillance, onboarding, policy-RAG, code-assist) consume the platform; they own outcomes, not infrastructure.
- **Partners, not passengers** — model-risk, compliance, and security sit *inside* the platform's design (they define the golden path's gates) while staying independent reviewers; the platform team's interface to them is the automated evidence the path generates (§7).
- **The operating rhythm** — the DX survey and path-usage metrics drive the roadmap; platform SLOs (eval-gate latency, sandbox spin-up time, gateway availability) are published like a product.

### 8.5 The Lessons

1. **Start with the golden path, not the portal.** A portal with no paved automation behind it is a catalog of work (§2.1). CF's first milestone was the *path* — one use case, end-to-end, on the platform — with the portal grown from what the path needed.
2. **The gateway is the control plane; everything else plugs into it.** Routing, cost, security, and audit all meet at one point (`enterprise_ai_gateway_guide.md`) — build it early, and every later layer (guardrails, registry approvals, budgets) becomes a policy attached to it.
3. **Eval is the CI for AI.** The eval gate is what makes "governed" and "fast" compatible: the evidence is produced by the path itself, not by a manual review queue (§4.3).
4. **Governance must be inside the path, or it will be bypassed.** Prohibition fails; the compliant route must be the fastest route (§2.3, §7). CF's regulators are served by the platform's automatic evidence trail — which is why the compliance team is the platform's ally, not its reviewer.
5. **Adoption is the only real KPI.** GPU utilization, catalog entries, and self-service rate matter; the number of teams using the golden path is the number that matters most (§2.4).
6. **The platform is a product, funded like one.** The business case is the portfolio of use cases it unblocks (cross-ref `../management/business_case_development_guide.md`); the platform team runs on product cadence, not project cadence (§8.4).

### 8.6 The Rollout — Phases 0–3

The design becomes real through the maturity ladder of §5.3, one phase at a time — each phase ends with a working golden path, not a milestone deck:

| Phase | Scope | Exit criteria | Risks to manage |
|---|---|---|---|
| **0 — Foundation (weeks 1–8)** | Platform team staffed (§8.4); the gateway stood up with two approved European-hosted models; registry seeded; one sandbox template | A developer can get a governed sandbox + gateway keys in under an hour | Scope creep into "build everything" — the thinnest viable platform (§2.5) |
| **1 — First path (weeks 9–20)** | The trade-surveillance squad's full prompt-to-production journey (§8.3) on the path — including the eval gate and the approval workflow | Squad time-to-production measured in days; evidence trail complete for one artifact | The path is built for one squad and becomes a bespoke pipeline — the path must be templated from day one |
| **2 — Productize (months 6–12)** | Portal (Backstage-class), templates for RAG/agent/prompt artifacts, service catalog published (§3.8), AI-DX metrics live (§6.5) | Golden-path adoption > 50% of new AI workloads; shadow-AI requests measurably down | Adoption theater — metrics that look good without changing behavior; the DX survey is the honest check |
| **3 — Federate (months 12–24)** | Domain platform teams (AI, data, security) with published team APIs; the bank-series patterns served (§7.3); suppliers pluggable | The platform is the default way AI happens in the group; off-path is the exception | Boundary friction between platforms — the service catalog and team APIs are the interface contract (§5.3 level 3) |

The phase plan embodies the guide's central lesson (§8.5): **path first, portal second, scale last** — and at every phase, the regulated way is the paved way.

---

## 9. The Summary — One Page

**The problem.** Every AI team in the enterprise builds its own stack: its own GPU access, its own prompts, its own evals, its own guardrails, its own compliance fight. The result is sprawl, shadow AI, ungoverned models, and a time-to-production measured in months. The tooling is not the problem — the *composition* is.

**The canon.** Platform engineering supplies the answer. The **IDP** is the sum of tech a platform team binds together to pave **golden paths** and enable self-service (§2.1). **Backstage** — Spotify's 2020 open-source, now CNCF Incubating — is the portal pattern (catalog, templates, plugins) (§2.2). The **golden path** is the preconfigured paved road that reduces cognitive load and makes the safe way the easy way (§2.3). **Platform-as-a-product** makes adoption the metric (§2.4). **Platform teams** (Team Topologies) reduce stream-aligned teams' cognitive load (§2.5).

**The stack.** The AI platform is the IDP for AI: six layers — **infrastructure** (GPU/cluster), **data** (lakehouse, feature stores like Feast, vector stores), **model** (registry, experiment tracking — MLflow/W&B — training platforms), **serving** (vLLM, TGI, TensorRT-LLM), **orchestration** (the agent runtime, MCP, workflows), and **governance** (RAI, trust, security) — each layer deep-dived by a sibling guide and bound here into one product (§3). Its LLMOps components — the **model gateway**, **prompt management**, **eval platforms**, **guardrails** — are platform capabilities, not point tools (§4). The evolution is clear: **MLOps** ran the model, **LLMOps** runs the prompt-plus-model-plus-context, the **AI platform** paves the road from prompt to production (§5).

**The path.** Self-service is the point: the **prompt-to-production golden path** — sandbox → data → build & eval → register & approve → serve → observe — with governance, cost, and security baked into every step (§6). For a bank, this is not a convenience: the EU AI Act, MAS, and model-risk requirements become platform features, and the evidence trail is generated by the path itself (§7). The worked design shows it concretely: a Cymbal Bank runs one platform team, six layers, one portal, one golden path — and the trade-surveillance squad goes from idea to governed production in days, not quarters (§8).

**The final word.** Platform engineering gave software a paved road; AI platform engineering gives AI the same road — from prompt to production, safe by construction, self-service by design. **The paved road for AI is the whole discipline in one image: make the governed way the fastest way, and the platform has won.**

---

## 10. Glossary

| Term | Definition |
|---|---|
| **AI platform** | The internal developer platform (IDP) for AI: the productized, self-service binding of the AI infrastructure, data, model, serving, orchestration, and governance layers into golden paths for AI developers (§1) |
| **Platform engineering** | The discipline of designing, building, and operating internal developer platforms as products, to reduce developers' cognitive load and enable self-service (§2) |
| **IDP / Internal Developer Platform** | The sum of all the tech and tools a platform engineering team binds together to pave golden paths and enable self-service for developers (Humanitec definition, verified) |
| **Backstage** | The open-source framework for building internal developer portals — created by Spotify, open-sourced 2020, CNCF Incubating project (verified); catalog + templates + plugins |
| **Golden path** | A preconfigured, paved, end-to-end workflow for developers, enabled via an IDP, designed to reduce cognitive load and ensure safe, compliant operation (platformengineering.org definition, verified) |
| **Paved road** | The standardized, supported default route metaphor from Google SRE culture and Evan Bottcher's *Paved Road in the Jungle* (2018): the desired way is the easiest, best-supported way |
| **Platform-as-a-product** | The operating model where the internal platform is treated as a product, its users as customers, and adoption as the success metric (§2.4) |
| **Platform team** | The Team Topologies team type that provides internal services to reduce the cognitive load of stream-aligned teams (Skelton & Pais, 2019) |
| **Feature store** | The platform component that defines, manages, and serves ML features consistently between training and serving (§3.2) |
| **Feast** | The open-source feature store, created at Gojek in 2018 and co-developed with Google Cloud; now under the Linux Foundation's LF AI & Data (verified) |
| **Model registry** | The versioned, governed store of models with lineage, license, eval, and approval metadata — the model layer's shelf (§3.3) |
| **MLflow** | The open-source experiment-tracking and model-registry platform, open-sourced by Databricks in 2018, now under LF AI & Data (§3.3) |
| **Weights and Biases (W&B)** | The commercial experiment-tracking platform (founded 2017), extended since 2023 into LLM prompt/trace tracking (§3.3) |
| **Kubeflow** | The open-source Kubernetes-native ML platform, originally developed by Google, a CNCF project (per the repo's comparison guide) |
| **SageMaker** | Amazon's managed ML platform: training, tuning, deployment, registry (§3.3; `ml_platforms_comparison_guide.md` §5) |
| **Vertex AI** | Google Cloud's managed ML/GenAI platform (§3.3; `ml_platforms_comparison_guide.md` §6) |
| **Serving** | The layer that turns models into endpoints: inference engines, batching, autoscaling, quantization (§3.4) |
| **vLLM** | The open-source high-throughput inference engine (PagedAttention/continuous batching), a serving-layer staple (`low_latency_genai_patterns_guide.md` §4.1) |
| **TGI** | Hugging Face Text Generation Inference — the HF-ecosystem serving engine (`low_latency_genai_patterns_guide.md` §4.2) |
| **Orchestration** | The layer that runs multi-step AI workloads: workflows, agent runtimes, tool/MCP wiring (§3.5) |
| **Agent runtime** | The execution substrate for agents: tool calling, memory/context, sandboxing, entitlements (§3.5) |
| **MLOps** | The discipline of operationalizing predictive ML — the model lifecycle (tracking, training, registry, deployment) (§5) |
| **LLMOps** | The discipline of operating LLM-based systems: prompts, tokens, evals, guardrails, traces (§4, §5) |
| **Gateway (model gateway)** | The centralized control plane between AI consumers and providers: routing, security, cost, observability, fallback (`enterprise_ai_gateway_guide.md`) |
| **Prompt management** | The versioning, lineage, cataloging, and approval of prompts as controlled artifacts (§4.2) |
| **Eval (evaluation)** | The discipline and tooling of measuring AI outputs — eval suites, LLM-as-judge, datasets; "the CI for AI" (§4.3) |
| **Guardrails** | The automated safety constraints on inputs, outputs, and agent actions — injection defense, PII, moderation, policy (§4.4) |
| **Self-service** | The platform property where developers obtain capabilities (environments, models, approvals, deployments) without tickets (§6) |
| **Prompt-to-production** | The AI platform's golden path: sandbox → data → build & eval → register & approve → serve → observe (§6.1) |

---

## Appendix: Verification Notes

**Verified via web research (2026-08-25, Firecrawl backend):**

- **Backstage** — open-sourced by Spotify in **2020**; a **CNCF Incubating Project** (spotify.engineering "Open Source" page); GitHub 34.3k+ stars, Apache-2.0, ~75k commits; Spotify: 2,700+ engineers, 14,000+ software components (verified).
- **IDP definition** — Humanitec / Kaspar von Grünberg ("What Is an Internal Developer Platform?", 2021): the IDP as "the sum of all the tech and tools that a platform engineering team binds together to pave golden paths and enable self-service"; IDP ≠ portal; five functional areas (infrastructure orchestration, application configuration, deployment management, environment management, RBAC); canon dates (Vogels "you build it, you run it" 2006; Debois DevOpsDays 2009; Thoughtworks radar 2017; Team Topologies 2019; Netflix platform engineering 2017) (verified via the Humanitec article).
- **Golden paths** — platformengineering.org definition (preconfigured paved road, end-to-end workflow, cognitive load, safe/compliant operation); Red Hat framing (opinionated, well-documented, supported route, explicitly including AI/MLOps teams) (verified).
- **Feast** — created at Gojek 2018, co-developed with Google Cloud, now standalone under the Linux Foundation (LF AI & Data) (verified via feast.dev/GitHub).
- **platformengineering.org** — maintains dedicated tracks including **AI platform engineering** (verified via site navigation).
- **Crédit Agricole "transformation IA"** — press release 10 Jun 2026: ~€500M/3 years, "Entreprise IA" (develops and operates the group's industrial AI platforms, European-solution preference), "AI For All" (verified via the repo's own `../banking/credit_agricole_software_systems_guide.md` §8, re-verified 2026-08-22).
- **LLMOps component landscape** — LangSmith, W&B (LLMOps/prompt-tracking), Braintrust, Arize Phoenix, Promptfoo, Guardrails AI confirmed as the category's tooling (2026 LLMOps platform comparisons).

**Flagged / not fully verifiable during this session (search backend rate-limited):**

- **Backstage CNCF donation date** — widely reported as 2022; the *incubating status* is verified, the exact donation date is flagged.
- **Platform-engineering adoption figures** — the commonly cited "83% of organizations adopting platform engineering" (Humanitec surveys) could not be re-verified this session; treat any single adoption percentage as reported-and-approximate, not audited.
- **Evan Bottcher, *Paved Road in the Jungle* (2018)** — the attribution is solid canon, but the martinfowler.com URL returned 404 during extraction; the essay's hosting has moved — the *concept* is verified via the golden-path literature, the *URL* is not.
- **Kubeflow CNCF status** — stated as "CNCF project" per the repo's own `ml_platforms_comparison_guide.md`; the exact maturity level (sandbox/incubating) is not pinned here.
- **Tool launch dates** (MLflow 2018, W&B 2017, LangSmith 2023, Feast 2018, vLLM/TGI 2023) — consistent with the repo's sibling guides and widely documented, but not re-verified against primary sources this session; they are structural dates, not load-bearing claims.
- **The worked example ("Caisse Fédérale")** — a fictional design vehicle in the familiar Cymbal Bank context; no real bank is described by it.

**Cross-reference map (how this guide binds the repo):** gateway → `enterprise_ai_gateway_guide.md`; registry → `huggingface_vs_csghub_guide.md`; serving → `low_latency_genai_patterns_guide.md`; MLOps lifecycle → `mlops_lifecycle_frameworks_guide.md`; ML platform tooling → `ml_platforms_comparison_guide.md`; lakehouse → `cloud_object_storage_lakehouse_guide.md`; agent runtime/MCP → `agentic_workflows_guide.md`, `ai_llm/mcp_framework_tools_guide.md`, `ai_llm/production_ready_llm_agents_guide.md`, `ai_llm/enterprise_agentic_platform_architecture_guide.md`; eval → `ai_llm/llm_evaluation_frameworks_guide.md`, `ai_llm/rag/ragas_guide.md`, `ai_llm/rag/trulens_guide.md`; guardrails/security → `llm_development_risks_security_guide.md`; RAI/trust → `responsible_ai_frameworks_guide.md`, `ai_trust_assessments_guide.md`; banking → `../banking/ai_genai_banking_compliance_guide.md`, `../banking/credit_agricole_software_systems_guide.md`; teams → `../management/high_performance_team_guide.md`; investment → `../management/business_case_development_guide.md`; skill gap → `data_architect_skillgaps_guide.md` (Gap #4, closed by this guide).

---

## References (Primary Sources)

**Platform-engineering canon:**

- Humanitec — *"What Is an Internal Developer Platform?"* (Kaspar von Grünberg, 2021) — the IDP definition, IDP ≠ portal, the five functional areas, the canon timeline (Vogels 2006, Debois 2009, Thoughtworks 2017, Netflix 2017, Team Topologies 2019)
- platformengineering.org — the platform-engineering community; golden-path definition; the AI/DATA/DEVEX/SECURITY tracks (including AI platform engineering)
- Skelton & Pais — *Team Topologies* (2019) — the platform team type
- Spotify Engineering — Backstage open-source (2020) and CNCF Incubating status; software catalog and templates
- Red Hat — *"What is a golden path for software development?"* — the opinionated, supported-route framing
- Google SRE literature — the paved-road culture; Evan Bottcher, *Paved Road in the Jungle* (2018)

**ML/AI platform sources:**

- Feast — feast.dev / GitHub — the open-source feature store (Gojek 2018, Google Cloud co-development, Linux Foundation / LF AI & Data)
- MLflow, Kubeflow, W&B, LangSmith, DeepEval, RAGAS, TruLens, Guardrails AI — vendor/docs as cross-referenced in the sibling guides and §4
- CNCF TAG App Delivery — platform engineering whitepaper and maturity model (referenced; exact URL unverified this session)

**Repository cross-references:** the full map is in the Appendix — every sibling guide this guide binds, with the section of this guide that uses it.

---

*End of guide. Verified claims are audited in the Appendix above; anything flagged there remains the reader's own verification task — the repository's house rule: never present an unverified claim as fact.*

