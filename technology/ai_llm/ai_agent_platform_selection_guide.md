# AI Agent Platform Selection: The "How to Choose an Agent Platform" Playbook

> **A comprehensive guide to selecting an AI agent platform — the agent-native platform landscape (LangGraph Platform, OpenAI Agents SDK + AgentKit, Microsoft Copilot Studio, Salesforce Agentforce, AWS Bedrock Agents, Google ADK + Vertex Agent Builder, CrewAI, AutoGen/AG2, smolagents, Semantic Kernel), the platform categories, the evaluation criteria, the weighted-scoring framework, the decision matrix, the selection process, the pitfalls, and a worked selection for a bank choosing an agent platform.**

**Author:** Jack Liu Shurui
**Role:** Solution Architect, Crédit Agricole CIB
**Date:** August 2026
**Version:** 1.0
**Repository:** github.com/jackliusr/research
**Series:** AI/LLM Engineering Guides — Agent & Platform track

**Reading time:** ~60 minutes

---

## Table of Contents
1. [The Agent Platform Landscape](#1-the-agent-platform-landscape)
2. [Platform Categories](#2-platform-categories)
3. [The Evaluation Criteria](#3-the-evaluation-criteria)
4. [The Weighted-Scoring Framework](#4-the-weighted-scoring-framework)
5. [The Decision Matrix](#5-the-decision-matrix)
6. [The Selection Process](#6-the-selection-process)
7. [The Pitfalls](#7-the-pitfalls)
8. [Worked Example: A Bank Chooses an Agent Platform](#8-worked-example-a-bank-chooses-an-agent-platform)
9. [The Future (2026+)](#9-the-future-2026)
10. [Summary: Agent Platform Selection in One Page](#10-summary-agent-platform-selection-in-one-page)
11. [Glossary](#11-glossary)

---

## How This Guide Fits the Series

This guide is the **dedicated deep-dive on agent platform selection** — the *decision playbook*: how to survey the agent-native platform landscape, categorise the candidates, weigh the evaluation criteria, score them, and choose. The series' other guides are the context this one operates on:

| Guide | Role relative to this guide |
|---|---|
| [enterprise_ai_platforms_guide.md](enterprise_ai_platforms_guide.md) | THE general AI vendor survey (Azure AI Foundry / Vertex / Bedrock categories, vendor comparison, evaluation criteria §5, decision framework §11). **This guide is the agent-platform-specific companion**: the same evaluation discipline applied to the agent-NATIVE platforms (the agent builders), rather than the full AI platform suites |
| [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) | The build-side reference architecture (gateways, observability layers, security, governance, FinOps) — what you build *on top of* whatever platform you choose |
| [autonomous_agents_guide.md](autonomous_agents_guide.md) | The agents umbrella — its §7 covers the major frameworks and §7.3 the platform/marketplace angle; this guide turns that survey into a selection method |
| [agent_scaffolding_guide.md](agent_scaffolding_guide.md) | The code scaffold (loop, tools, config, prompt versioning) — the artifact the platform hosts |
| [agentops_guide.md](agentops_guide.md) | The ops discipline — observability, evals, monitoring. **Platform-native observability (§3.4 here) is scored against that discipline** |
| [mcp_framework_tools_guide.md](mcp_framework_tools_guide.md) | The tool layer — MCP. **MCP support is a first-class selection criterion (§3.3 here)** |
| [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) and [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) | The build playbook and the failure catalogue — the engineering bar the chosen platform must meet |
| [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md) and [hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md) | Multi-agent orchestration patterns — a workload that separates platform candidates |
| [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md) and [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) | Eval design — the evals capability you score the platform on |
| [implementing-responsible-ai.md](implementing-responsible-ai.md) | Governance: RBAC, audit, approval workflows, HITL — the governance criteria (§3.5 here) |
| [ai_agent_drift_guide.md](ai_agent_drift_guide.md) | Drift and behavioral monitoring — a capability to check in the platform's monitoring surface |
| [llm_agent_use_cases.md](llm_agent_use_cases.md) and [coding_agents_research.md](coding_agents_research.md) | Use-case catalogue — the input to the use-case→platform matrix (§5.1 here) |
| [china_ai_agent_frameworks.md](china_ai_agent_frameworks.md) | The China ecosystem (Qwen agents, Dify, Coze) — a parallel landscape if you operate in APAC/China markets |
| [financial_risk_compliance_systems_guide.md](../../banking/financial_risk_compliance_systems_guide.md) | Banking risk/compliance context for the worked example (§8 here) |
| [finops_guide.md](../finops_guide.md) | FinOps practice — the cost-attribution discipline behind the cost criterion (§3.7 here) |

**Verification note.** This guide was written against the 2025–2026 state of the agent-platform market. Claims are flagged inline as **(verified)** where confirmed against vendor documentation or reputable industry reporting as of August 2026, **(approximate)** where the fact is directionally right but numbers/pricing are vendor-controlled and move fast, and **(opinion)** where it is my synthesis as a solution architect. The agent-platform market is the fastest-moving segment of the AI stack; several platforms below were renamed, rebranded, or superseded within a single 12-month window (LangGraph Cloud → LangGraph Platform → LangSmith Deployment; Bedrock Agents → Bedrock AgentCore; AutoGen → AG2 fork). Where a fact could not be verified, I say so honestly rather than presenting marketing copy as consensus.

---

## 1. The Agent Platform Landscape

### 1.1 What an "Agent Platform" Is

An **agent platform** is a product that lets you **build, deploy, run, and operate AI agents without owning the runtime infrastructure** — it packages the agent runtime (the loop: model calls, tool calls, state, memory), the orchestration primitives, the tool-integration layer, the observability, and the governance controls into one managed or self-hostable offering.

The mental model for this guide:

> **A framework gives you code. A platform gives you the running system.** The framework is a library you import; the platform is the environment that runs the thing you build with it — plus the operational surface (monitoring, evals, access control, audit, scaling, billing) that makes it production-safe.

The platforms in scope here are the **agent-native** ones — the products purpose-built for the *agent* lifecycle (as opposed to the general enterprise AI platforms in [enterprise_ai_platforms_guide.md](enterprise_ai_platforms_guide.md), which cover the wider ML/LLM lifecycle). Every major AI vendor now ships one, because the 2024–2026 market consensus is that agents — not chatbots — are the enterprise AI workload of record. *(Verified — by 2025–2026, all four hyperscalers plus OpenAI, Anthropic-ecosystem players, LangChain, CrewAI, and Salesforce ship dedicated agent-builder products; the Gartner-style prediction that the majority of enterprises will deploy agents by 2026–2027 is widely cited, though the exact percentage is contested.)*

### 1.2 The Agent-Native Platforms — The Candidates

The candidate set for any serious selection exercise, with the facts verified as of August 2026:

#### 1.2.1 LangGraph Platform (LangChain) — the hosted agent platform *(verified)*

- **What it is:** LangChain's managed platform for deploying, scaling, and operating **LangGraph** agents — long-running, stateful, human-in-the-loop-capable agents. Formerly **LangGraph Cloud**; rebranded to **LangGraph Platform** in October 2025, when LangChain also rebranded/expanded its deployment service under the **LangSmith Deployment** brand — the naming has churned, so check the current product page. *(Verified — the LangChain changelog of 13 Oct 2025 documents "formerly known as LangGraph Cloud" and the new deployment options; the LangGraph Platform GA announcement and the LangSmith-Deployment-as-successor-brand both appear in LangChain's own materials.)*
- **Deployment options:** Developer tier (cloud-hosted), **Self-Hosted Lite** (free, limited to ~1M nodes executed, runs locally or self-hosted), and enterprise self-hosted/BYOC options for regulated environments. *(Verified — the deployment-options announcement lists exactly these; the 1M-node figure is the vendor's published cap.)*
- **Position:** the reference "agent-framework platform" — if you build with LangGraph (the most widely adopted agent *framework*), this is the native path to production. Deep integration with LangSmith (tracing, evals) and LangGraph's checkpointing/memory model.

#### 1.2.2 OpenAI Agents SDK + AgentKit — the code-first API platforms *(verified)*

- **Agents SDK (March 2025):** OpenAI's lightweight, open-source SDK for building multi-agent workflows — agents, handoffs, guardrails, sessions. Provider-agnostic: supports the OpenAI Responses and Chat Completions APIs **plus 100+ other LLMs**, so it is *not* locked to OpenAI models. Python and JavaScript/TypeScript. *(Verified — OpenAI's SDK docs and GitHub state provider-agnosticism and the 100+ model figure explicitly.)*
- **AgentKit (6 October 2025):** OpenAI's "complete set of tools for developers and enterprises to build, deploy, and optimize agents" — orchestration with versioning, connectors, eval pipelines, and frontend components, layered around the Agents SDK. This is OpenAI's move from *framework* to *platform-ish* offering. *(Verified — OpenAI's launch announcement of 6 Oct 2025.)*
- **Position:** the code-first end of the spectrum. Maximum developer control, minimal managed runtime; you own the infrastructure, orchestration, and ops. AgentKit narrows that gap but is still developer-centric rather than a no-code enterprise product.

#### 1.2.3 Microsoft Copilot Studio — the Microsoft agent builder *(verified)*

- **What it is:** Microsoft's agent-builder product: a low-code studio for building conversational and **autonomous agents** — agents that "perceive events, make decisions, and execute tasks independently using triggers, instructions, and guardrails" rather than waiting for prompts. Sits inside the Microsoft stack: publishes to Microsoft 365 Copilot, connects to Power Platform connectors, and shares governance with **Azure AI Foundry** (the broader AI platform in [enterprise_ai_platforms_guide.md](enterprise_ai_platforms_guide.md)). The studio has been progressively rebranded toward **"Agent Builder"** in Microsoft's 2026 messaging. *(Verified — Microsoft Learn's autonomous-agents guidance and the Copilot Studio blog both confirm the autonomous-agent capability and the Agent Builder branding.)*
- **Pricing:** seat-based (Copilot Credits packs) **and** a pay-as-you-go metered option for standalone agents. *(Verified — Microsoft's product page lists both.)*
- **Position:** the enterprise low-code platform. If your estate is Microsoft 365 + Power Platform + Azure, this is the path of least resistance — at the price of being the most Microsoft-centric option on this list.

#### 1.2.4 Salesforce Agentforce — the CRM-native digital labor platform *(verified)*

- **What it is:** Salesforce's agent platform, launched September 2024 and upgraded to **Agentforce 2.0** (announced 17 December 2024, generally available early 2025). Positioned as a "**digital labor**" platform: agents that act on CRM data — customer service, sales, marketing — composed with **Agent Builder** (natural-language composition of topics, instructions, and skills from the existing action library) and powered by the **Atlas reasoning engine** over Salesforce Data Cloud. *(Verified — Salesforce's own press materials and launch timeline.)*
- **Position:** the enterprise *application* platform. Unmatched if your agents must live inside Salesforce data and flows; an awkward fit for general-purpose engineering workloads outside the CRM.

#### 1.2.5 AWS Bedrock Agents — and the AgentCore evolution *(verified)*

- **What it is:** AWS's managed agent service on **Amazon Bedrock**: define an agent (instructions, model, action groups/tools, knowledge bases, memory, guardrails), and AWS runs the orchestration loop at scale. Long-running support, multi-agent collaboration, and Bedrock Guardrails integration are all GA.
- **The 2025–2026 evolution:** AWS has been transitioning the runtime to **Amazon Bedrock AgentCore** — the next-generation managed agent runtime — with "**runtime instances**": persistent, managed compute (EC2-based, GPU-capable) for production agents, multi-agent collaboration, and sessions lasting up to **14 days** (announced August 2026). **Bedrock Agents (Classic) is in maintenance mode** — new work targets AgentCore. *(Verified — AWS docs explicitly note "Bedrock Agents Classic maintenance mode" and the AgentCore runtime-instances announcement is an AWS blog of Aug 2026.)*
- **Position:** the cloud-agent platform for AWS estates, with the deepest enterprise plumbing (IAM, KMS, VPC, CloudTrail) and the widest model choice of any managed platform (Bedrock hosts models from Anthropic, Meta, Mistral, Cohere, Amazon, and more). Also notable: AWS now ships its own **Strands Agents SDK** for code-first multi-agent development on top.

#### 1.2.6 Google ADK + Vertex AI Agent Builder — the Google agent stack *(verified)*

- **ADK (Agent Development Kit, April 2025):** Google's open-source agent framework — Python, TypeScript, Go, Java, Kotlin — for building, evaluating, and deploying agents. The "developer's toolkit" layer of Google's stack.
- **Vertex AI Agent Builder:** the managed layer — **Agent Garden** (no-code agent creation) plus **Agent Engine** (the managed runtime that deploys and scales ADK-built agents on Vertex, with VPC, IAM, and Vertex observability). *(Verified — Google's ADK materials and the Vertex agent documentation describe the Agent Garden / ADK / Agent Engine stack.)*
- **Position:** the Google-cloud agent platform. Strong Gemini-native experience (multi-modal, long context), reasonable open-source ethos at the framework layer (ADK), and the same "managed runtime on your VPC" pattern as AgentCore.

#### 1.2.7 CrewAI (Enterprise / AMP) — framework + platform *(verified)*

- **What it is:** CrewAI began as the popular role-based multi-agent **framework** (crews of agents with roles, goals, and tasks). It now ships a platform: **CrewAI AMP** — a cloud multi-agent platform with a no-code studio, unified control plane, real-time observability, secure integrations, and enterprise support — plus a dedicated **CrewAI Enterprise** offering, with cloud and on-premise deployment options. *(Verified — CrewAI's enterprise site and GitHub FAQ describe AMP's control plane/observability and the on-premise option.)*
- **Position:** the framework→platform maturation case *par excellence* — see §9.3. Good for role-based business-process crews (research, back-office ops); lighter enterprise plumbing than the hyperscaler platforms.

#### 1.2.8 AutoGen (Microsoft, open-source) — the research-origin multi-agent framework *(verified)*

- **What it is:** Microsoft's open-source multi-agent framework (conversation-driven multi-agent collaboration, ~55K GitHub stars), one of the most-cited academic-origin agent frameworks. **Governance reality:** after the November 2024 governance split, the original maintainers forked the project (see AG2 below); Microsoft's own direction moved on to the **Microsoft Agent Framework** (the enterprise successor that unifies AutoGen and Semantic Kernel — at v1.0 as of 2026). Classic AutoGen is effectively in maintenance mode, and the community's migration paths are AG2 or the Microsoft Agent Framework. *(Verified — the fork timeline and the Microsoft Agent Framework v1.0 are documented in the Microsoft/Semantic-Kernel GitHub repo and multiple 2025–2026 comparisons.)*
- **Position:** open-source framework for research and self-hosted multi-agent systems; **not** a platform — there is no managed AutoGen runtime. Choose it for research/experimentation and self-hosting, not for turnkey production.

#### 1.2.9 AG2 (the AutoGen fork) — the community continuation *(verified)*

- **What it is:** the community fork of AutoGen created after the November 2024 governance split, renamed from "AutoGen" to **AG2** to avoid trademark collision with Microsoft's continued use of the name. Continues active development of the conversation-driven multi-agent model with a community governance model. *(Verified — the fork timeline is documented across the project's own materials and multiple 2025–2026 framework comparisons.)*
- **Position:** for teams already invested in the AutoGen programming model who want the community-governed continuation. Self-hosted only.

#### 1.2.10 smolagents (HuggingFace) — the minimal code-agent library *(verified)*

- **What it is:** HuggingFace's open-source agent library (December 2024) built on an unusual premise: **code agents** — the model writes *Python code* (not JSON tool calls) and the library executes it. The core logic fits in ~1,000 lines; abstractions are deliberately minimal. First-class support for many model providers and local models. *(Verified — HuggingFace's launch blog and the GitHub repo state the ~1,000-line core and the code-agent design.)*
- **Position:** the simplicity end of the framework spectrum — ideal for prototyping, research, and teams that want to understand every line. Not a platform: no managed runtime, no enterprise governance.

#### 1.2.11 Semantic Kernel (Microsoft) — the embeddable orchestration SDK *(verified)*

- **What it is:** Microsoft's lightweight, open-source AI orchestration SDK (C#, Python, Java) that "sits inside existing applications and services rather than as a standalone runtime" — you embed agent capabilities into your own services. **Important 2026 update:** Microsoft's **Agent Framework (MAF)** is the enterprise-ready successor, v1.0 with stable APIs and long-term support. *(Verified — the Microsoft Learn overview and the semantic-kernel GitHub repo.)* **Security note:** Semantic Kernel has had two confirmed critical RCE-class vulnerabilities (a confused-deputy pattern) — a reminder that self-hosted agent frameworks put the security burden on you. *(Verified — the advisories are documented in public security write-ups.)*
- **Position:** for .NET/C#/Java enterprise estates embedding agent logic into existing services — the "agent as library" philosophy.

### 1.3 Framework vs Platform — The Distinction

The single most important conceptual tool in this guide. The **frameworks** are libraries: **LangGraph, CrewAI, AutoGen, AG2, smolagents, Semantic Kernel, ADK, OpenAI Agents SDK**. The **platforms** are managed environments: **LangGraph Platform, Bedrock Agents/AgentCore, Vertex Agent Builder (Agent Engine), Copilot Studio, Agentforce, CrewAI AMP/Enterprise, AgentKit** (partially).

| Axis | Framework (library) | Platform (managed) |
|---|---|---|
| What you get | Code: loop, tools, state abstractions | Running system: runtime + infra + ops surface |
| Who runs it | You | The vendor (or your ops on self-hosted platform builds) |
| Ops burden | You build tracing, evals, scaling, auth | Bundled (quality varies by vendor) |
| Control | Total | Bounded by platform abstractions |
| Lock-in | Code-level (portable if you keep abstractions thin) | Service-level (harder to leave) |
| Time to production | Longest (you assemble everything) | Shortest (batteries included) |
| Skill required | Agent engineers | Makers/analysts up to agent engineers |

*(Verified as a general market structure — every major vendor now sells "framework + managed platform" as a ladder: LangGraph → LangGraph Platform, CrewAI → CrewAI AMP, ADK → Agent Engine, Agents SDK → AgentKit. The framework-vs-platform boundary is the one most selection exercises get wrong; see §7.1.)*

### 1.4 The Landscape Table

| Platform | Vendor | Type | Hosted / Self-hosted | Key features (verified as of Aug 2026) |
|---|---|---|---|---|
| LangGraph Platform (a.k.a. LangSmith Deployment) | LangChain | Agent-framework platform | Both — cloud Developer tier + Self-Hosted Lite + enterprise self-hosted | LangGraph runtime, checkpointing/memory, HITL, LangSmith tracing + evals, MCP support |
| OpenAI Agents SDK | OpenAI | Open-source framework (API-agent) | Self-hosted (library) | Agents, handoffs, guardrails, sessions; provider-agnostic (100+ LLMs); Python/TS |
| AgentKit | OpenAI | API-agent platform | Cloud (managed services) + SDK | Orchestration w/ versioning, connectors, evals, frontend components; built on Agents SDK |
| Microsoft Copilot Studio | Microsoft | Cloud-agent platform (low-code) | Hosted | Autonomous agents, M365 Copilot integration, Power Platform connectors, MCP support, PAYG + credits pricing |
| Salesforce Agentforce | Salesforce | Enterprise application platform | Hosted (Salesforce cloud) | CRM-native digital labor, Agent Builder, Atlas reasoning, Data Cloud grounding |
| AWS Bedrock Agents / AgentCore | AWS | Cloud-agent platform | Hosted (VPC-integrated) | Managed orchestration, multi-agent collaboration, runtime instances (persistent, GPU, 14-day sessions), Guardrails, MCP support, widest model choice |
| Google ADK | Google | Open-source framework | Self-hosted (library) | Python/TS/Go/Java/Kotlin, built-in eval, deploys to Agent Engine |
| Google Vertex Agent Builder | Google | Cloud-agent platform | Hosted (VPC-integrated) | Agent Garden (no-code) + Agent Engine (managed runtime), Gemini-native, MCP support |
| CrewAI Enterprise / AMP | CrewAI | Agent-framework platform | Both — cloud + on-premise | Crews (roles/tasks), no-code studio, control plane, observability, enterprise support |
| AutoGen | Microsoft | Open-source framework | Self-hosted | Conversation-driven multi-agent; maintenance mode → Microsoft Agent Framework |
| AG2 | AG2 community | Open-source framework | Self-hosted | AutoGen continuation, community governance |
| smolagents | HuggingFace | Open-source framework | Self-hosted | Code agents (~1,000 LOC core), minimal abstractions, local-model friendly |
| Semantic Kernel | Microsoft | Open-source framework | Self-hosted (embeddable) | C#/Python/Java, embeds in apps; superseded by Microsoft Agent Framework (v1.0) |

---

## 2. Platform Categories

Five categories organise the landscape into decision-relevant buckets. Category membership tells you more about a platform's *nature* than any single feature does — a no-code CRM platform and a code-first SDK are not competitors for the same workload.

### 2.1 Cloud-Agent Platforms (Managed)

**The players:** AWS Bedrock Agents/AgentCore, Google Vertex Agent Builder (Agent Engine), Microsoft Copilot Studio, (and in the general-AI-platform sense, Azure AI Foundry's agent tooling — see [enterprise_ai_platforms_guide.md](enterprise_ai_platforms_guide.md)).

The hyperscaler play: the agent runtime is a **managed service on your cloud account**, integrated with the cloud's identity (IAM/Entra ID), networking (VPC/VNet), audit (CloudTrail/diagnostic logs), and model marketplace. You define the agent declaratively (instructions + tools + knowledge + guardrails) and the cloud runs the loop.

- **Strengths:** deepest enterprise plumbing; compliance and data-residency story tied to your existing cloud trust boundary; scales and patches for you; broad model access through the cloud's model catalog.
- **Weaknesses:** least portable (you are committing to a cloud); orchestration expressiveness can lag code-first frameworks for exotic control flows; cloud pricing surprises on high-volume agent loops.
- **Best for:** enterprises already standardised on a hyperscaler that need production-grade governance and don't need framework-level control.

### 2.2 Agent-Framework Platforms (Framework + Managed Runtime)

**The players:** LangGraph Platform, CrewAI AMP/Enterprise.

The "framework vendor grows up" play: start with the framework's programming model, then deploy the same code to the vendor's managed runtime. You keep developer expressiveness *and* get a platform's ops surface (tracing, evals, scaling, HITL).

- **Strengths:** developer velocity (code in, production out); framework-native observability; the framework skills you already hired for transfer; self-hosted options for regulated estates.
- **Weaknesses:** smaller compliance/enterprise surface than hyperscaler platforms; vendor is a startup-scale company (CrewAI, LangChain) — platform longevity is a real diligence item; less cloud-native integration (you integrate your cloud yourself).
- **Best for:** teams that have committed to a framework (LangGraph, CrewAI) and want a fast, framework-native path to production without building their own runtime.

### 2.3 API-Agent Platforms (Code-First)

**The players:** OpenAI Agents SDK + AgentKit; arguably the AWS Strands Agents SDK.

The "SDK-first" play: lightweight, open-source, provider-agnostic building blocks plus (in AgentKit's case) managed companion services. You own the runtime; the vendor supplies the primitives and the managed extras you opt into.

- **Strengths:** maximum control; no platform lock-in at the runtime level; excellent for engineering teams; provider-agnostic (SDK runs against 100+ models).
- **Weaknesses:** you assemble the ops stack (observability, evals, governance) yourself; no enterprise UI; everything is code.
- **Best for:** strong agent-engineering teams that treat the platform question as "which primitives" rather than "which environment".

### 2.4 Enterprise Application Platforms (Application-Native)

**The players:** Salesforce Agentforce (CRM-native); adjacent: ServiceNow AI Agents, SAP Joule agents, Microsoft Dynamics agents.

The "agents as a feature of the application" play: the platform is not an agent runtime you build on — it is the business application (CRM, service desk, ERP) with agents as a native capability operating on *its* data and workflows.

- **Strengths:** fastest time-to-value inside the application; data access without integration (the agent is born inside the data); business-user composition (Agent Builder).
- **Weaknesses:** confined to the application's data model and flows; general engineering workloads are a poor fit; the platform is the vendor's cloud — portability is minimal.
- **Best for:** enterprises whose agent use cases are dominated by one business application (CRM service/sales, service management, ERP).

### 2.5 Open-Source Frameworks (Self-Hosted)

**The players:** AutoGen, AG2, smolagents, Semantic Kernel, ADK (as a library), LangGraph (as a library), CrewAI (as a library).

The "bring your own everything" play: open-source libraries, self-hosted, full control, full responsibility.

- **Strengths:** zero license cost; total control and portability; auditability (you can read the code — a real compliance argument); no vendor dependency; runs anywhere, including air-gapped.
- **Weaknesses:** you build the platform — runtime, scaling, tracing, evals, auth, audit, on-call — yourself; security burden is yours (Semantic Kernel's RCE history is the cautionary tale); team skill requirements are the highest.
- **Best for:** research, regulated estates with strict self-hosting mandates, teams with strong agent-engineering depth, and cost-sensitive deployments at scale.

### 2.6 The Category Comparison

| Category | Strengths | Weaknesses | Best for |
|---|---|---|---|
| Cloud-agent platforms (Bedrock Agents, Vertex Agent Builder, Copilot Studio) | Enterprise plumbing, compliance, managed ops, model catalogs | Portability, cloud lock-in, control-flow expressiveness | Hyperscaler-standardised enterprises, regulated workloads |
| Agent-framework platforms (LangGraph Platform, CrewAI AMP) | Developer velocity, framework-native ops, self-host options | Startup-vendor longevity risk, thinner enterprise surface | Framework-committed teams going to production fast |
| API-agent platforms (OpenAI Agents SDK, AgentKit) | Control, provider-agnostic, engineering-friendly | You assemble ops; code-only | Strong engineering teams, custom runtimes |
| Enterprise application platforms (Agentforce) | Application-native data, fastest in-app value, no-code | Application-bound, vendor-cloud-only | CRM/service-desk/ERP-centric use cases |
| Open-source frameworks (AutoGen, AG2, smolagents, Semantic Kernel) | Free, portable, auditable, air-gap capable | You build and run everything; security burden yours | Research, self-hosting mandates, deep-engineering teams |

---

## 3. The Evaluation Criteria

Nine criteria cover everything a selection exercise actually needs. They are deliberately orthogonal — each asks a different question, and each maps to a weight in the scoring framework (§4). The criteria are ordered roughly by how often they decide real enterprise selections; the weight suggestions in §3.10 reflect that.

### 3.1 Deployment Model — Hosted vs Self-Hosted

**The question: where does the agent runtime run, and who controls it?**

- **Hosted:** the vendor runs the runtime (Copilot Studio, Agentforce, LangGraph Platform Developer tier). Fastest to start; zero infra. The control/compliance question is: *whose trust boundary is the agent inside?*
- **Self-hosted:** you run the runtime in your environment — either the platform's self-hosted edition (LangGraph Platform Self-Hosted, CrewAI on-premise, Bedrock/Vertex running inside your VPC) or a bare framework (AutoGen, smolagents). Full control, data stays in your boundary — at the cost of running and operating the runtime yourself.

For regulated industries (banking, healthcare), the practical question is rarely "hosted or not" but "**can the runtime run inside our VPC / on-prem boundary, and does the vendor's control plane touch our data?**" Hyperscaler platforms (Bedrock Agents, Vertex Agent Builder) run inside your cloud account and are therefore "hosted *and* in-your-boundary" — a distinction that matters more than the binary label. *(Verified as the standard enterprise pattern — see the deployment-models treatment in [enterprise_ai_platforms_guide.md](enterprise_ai_platforms_guide.md) §9.)*

Score high when: the platform offers the deployment mode you need (hosted for speed, in-VPC/self-hosted for compliance) *and* the vendor's architecture keeps data inside that boundary.

### 3.2 Model Access — Flexibility and BYO-Key

**The question: which models can the agent call, and who pays for them?**

- **Model catalog breadth:** how many models does the platform offer? Bedrock hosts Anthropic, Meta, Mistral, Cohere, Amazon, and more; Vertex hosts Gemini + open models + Anthropic; Copilot Studio is Azure OpenAI-centric; Agentforce is Atlas/Salesforce-model-centric; OpenAI AgentKit is OpenAI-centric (though the SDK underneath is provider-agnostic).
- **BYO-key:** can you bring your own model API keys (e.g. call your own Anthropic/OpenAI account from inside the platform)? Critical for pricing control and for using models the platform doesn't host.
- **Model freedom matters for:** negotiating leverage (no single-model hostage), resilience (failover across providers), and capability fit (coding vs reasoning vs multilingual).

*(Approximate — model catalogs change monthly; the direction of each platform's model policy is verified, the exact lists are not.)*

### 3.3 Tool Integration — MCP and Custom Tools

**The question: how do agents reach your systems?**

- **MCP (Model Context Protocol) support** is now the default expectation: every serious platform either supports MCP servers natively or via connectors (Copilot Studio has MCP support; Bedrock Agents supports MCP servers; Vertex agents support MCP; LangGraph/LangSmith supports MCP; Agentforce supports MCP-style external actions). MCP is the interoperability layer that prevents your tools from becoming platform-specific — see [mcp_framework_tools_guide.md](mcp_framework_tools_guide.md) for the protocol deep-dive and its security/permission model. *(Verified — MCP, introduced by Anthropic in November 2024, was adopted by OpenAI (March 2025), Google, and Microsoft, and is supported across the major agent platforms; the specific connector mechanics vary and are approximate.)*
- **Custom tools:** can you register arbitrary functions/APIs (OpenAPI, Python, connectors)? Is there a no-code tool builder (Copilot Studio, Agentforce, Agent Garden) or code-only?
- **The security angle:** tool permissions, allow-lists, and credential handling are where agent platforms get dangerous — score the *permission model*, not just connectivity. Cross-ref [mcp_framework_tools_guide.md](mcp_framework_tools_guide.md) §security and [prompt_injection_guide.md](prompt_injection_guide.md).

### 3.4 Observability — Tracing, Evals, Monitoring

**The question: can you see what the agent did, and can you prove it?**

The AgentOps discipline in [agentops_guide.md](agentops_guide.md) defines what production agents need: span-level tracing (agent → step → tool → LLM), online and offline evals, dashboards (success rate, latency, cost), and alerting. Platforms differ enormously here:

- **Native vs bolt-on:** LangGraph Platform ships LangSmith (tracing + evals); AgentCore/Vertex ship cloud-native logging + CloudWatch/Cloud Logging (you assemble evals yourself); Copilot Studio ships Copilot Studio analytics + Azure Application Insights; Agentforce ships its own analytics. 
- **The critical question:** can you export traces and run *your own* eval harnesses against production traffic, or are you confined to the vendor's dashboards? For regulated estates, the audit trail must be exportable and immutable.

Score high when: tracing is automatic, evals are first-class (not just dashboards), and data is exportable (OTel-friendly — see §9.4).

### 3.5 Governance — RBAC, Audit, Approval Workflows

**The question: who may build, deploy, and let agents act — and is that recorded?**

- **RBAC:** per-user/per-team permissions for building agents, adding tools, and promoting to production.
- **Audit:** immutable logs of who changed what, and what agents did (tool calls, data accessed). For banking this is the regulatory face — cross-ref [implementing-responsible-ai.md](implementing-responsible-ai.md) and [financial_risk_compliance_systems_guide.md](../../banking/financial_risk_compliance_systems_guide.md).
- **Approval workflows / HITL:** human-in-the-loop checkpoints before high-impact actions (payments, external comms, privileged tool use). Some platforms have first-class HITL (LangGraph Platform, Copilot Studio, Agentforce); with others you build it yourself.
- **Agent registry & lifecycle:** a governed catalogue of production agents with owners, risk ratings, and version control.

### 3.6 Security — SSO, Data Residency, Compliance

**The question: does the platform meet your security and regulatory bar?**

- **SSO / IdP integration:** SAML/OIDC/Entra ID — table stakes for enterprise; check which identity providers are supported.
- **Data residency:** where does agent data (prompts, traces, tool payloads) physically reside? Can you pin a region? Can you pin *no-exfiltration* (data never leaves your boundary)?
- **Compliance certifications:** SOC 2 (baseline), ISO 27001, PCI DSS, HIPAA, and region-specific (MAS TRM for Singapore, EU AI Act readiness). Note that a hyperscaler platform inherits the cloud's certifications; a startup platform (LangChain, CrewAI) may have SOC 2 but a thinner compliance surface — verify the actual scope, don't assume.
- **Model data-handling:** are prompts/traces used for training? Can you opt out? (Enterprise tiers are typically no-training, but read the terms.)

### 3.7 Cost — The Pricing Model

**The question: how are you billed, and what does that mean at scale?**

| Pricing model | Who uses it | Cost behaviour | 
|---|---|---|
| Per-seat | Copilot Studio (credits), Agentforce (Agentforce licenses) | Predictable per user; can be expensive for high-volume automation (a per-seat license for a bot that replaces 100 agents is cheap; per-seat for internal-use agents across 10,000 staff adds up) |
| Per-token / per-message | Copilot Studio PAYG, Bedrock (model + runtime), Vertex (model + runtime), LangGraph Platform (usage tiers) | Scales with actual usage; needs budget caps and loop-cost control (see [agentops_guide.md](agentops_guide.md) §6 and [finops_guide.md](../finops_guide.md)) |
| Per-agent / per-action | Agentforce (per conversation/action), some SaaS platforms | Aligns to business value ("digital labor per action") but the per-action price is the least transparent |

*(Approximate — all pricing is vendor-controlled and changes frequently; the *models* are verified, the *numbers* are not. Treat any quoted unit price as a snapshot to re-verify during the PoC.)*

The scoring question is not "cheapest" but "**predictable and controllable at our expected volume**" — including the hidden multipliers: tokens per task (agent loops multiply token spend), storage of traces/sessions, and egress.

### 3.8 Lock-In — Portability and Export

**The question: if we leave, what do we carry, and what do we lose?**

- **Code-level portability:** can you take the agent logic with you? Frameworks (LangGraph, CrewAI, ADK, Agents SDK) are code — portable by construction. Platform abstractions (Bedrock agent definitions, Copilot Studio flows, Agentforce topics) are declarative artifacts bound to the platform.
- **Data export:** can you export traces, evals, session history, and audit logs in open formats (OTel, JSON, Parquet)?
- **The honest assessment:** every platform has *some* lock-in; the question is where it sits. Code-first = lock-in at the integration layer only. Managed platforms = lock-in at the runtime and tooling layer. The mitigation is the same in all cases: keep tool definitions standard (MCP — see [mcp_framework_tools_guide.md](mcp_framework_tools_guide.md)), keep agent logic in version-controlled code, and export everything continuously. Cross-ref the lock-in analysis in [enterprise_ai_platforms_guide.md](enterprise_ai_platforms_guide.md) §8.

### 3.9 Ecosystem — Integrations and Community

**The question: what can the agent reach, and who can help you build?**

- **Prebuilt connectors:** Copilot Studio (hundreds of Power Platform connectors + MCP), Agentforce (Salesforce actions), Bedrock/Vertex (cloud service integrations), LangGraph Platform (LangChain ecosystem), CrewAI (tool plugins).
- **Community & hiring:** LangGraph/CrewAI/AutoGen have the largest developer communities and the deepest talent pools; smolagents is small but growing; the managed platforms' communities are enterprise-practitioner rather than open-source.
- **Vendor viability:** for startup-vendor platforms (LangChain, CrewAI), diligence on funding, roadmap, and the product's churn (LangGraph Cloud → Platform → LangSmith Deployment; AutoGen → fork) is part of ecosystem scoring.

### 3.10 The Criteria Table

| # | Criterion | The question to ask | Weight suggestion |
|---|---|---|---|
| 1 | Deployment model | Does it run where we need it (hosted / in-VPC / self-hosted), with data in our boundary? | 10–15% |
| 2 | Model access | What models can it call, and can we BYO-key? | 8–12% |
| 3 | Tool integration | MCP support? Custom tools? Permission model? | 10–15% |
| 4 | Observability | Native tracing/evals/monitoring, exportable? | 10–15% |
| 5 | Governance | RBAC, audit, approval workflows, agent registry? | 8–12% |
| 6 | Security | SSO, data residency, compliance certs (SOC 2 +)? | 10–15% |
| 7 | Cost | Predictable pricing at our volume, controllable? | 8–12% |
| 8 | Lock-in | Portable code/tools, exportable data? | 5–10% |
| 9 | Ecosystem | Connectors, community, vendor viability? | 5–10% |

The weights are *suggestions* — they encode a typical regulated-enterprise posture (deployment, security, observability, tools heavy). Your weights come from your requirements (§4.1), not from this table. Non-negotiable requirements (e.g. "must run in our VPC", "must pass MAS audit") are not weights — they are **elimination filters** applied before scoring.

---

## 4. The Weighted-Scoring Framework

### 4.1 The Framework — Requirements → Weights → Scores → Decision

The whole method in four steps:

1. **Requirements:** write down what the agents must do, where they must run, and what constraints are non-negotiable (regulatory, security, integration). The requirements gathering process is in §6.1.
2. **Weights:** turn the requirements into weights over the nine criteria (§3.10). Weights must sum to 100%.
3. **Scores:** score each shortlisted platform 1–5 per criterion using the rubric (§4.2), based on evidence — vendor docs, the PoC (§6.3), and reference customers, not marketing.
4. **Decision:** compute weighted totals (§4.3), sanity-check against the decision matrix (§5) and the pitfalls (§7), and decide — build, buy, or framework (§6.5).

The discipline that makes this work: **weights before scores**. If you score first and tune weights to match your favourite, the framework is astrology. Decide what matters *before* you look at the candidates.

### 4.2 The Scoring Rubric — the 1–5 Scale

| Score | Meaning | What evidence looks like |
|---|---|---|
| 5 — Excellent | Exceeds requirements; no material gaps | Proven in the PoC, documented, reference customers in your industry |
| 4 — Good | Meets requirements fully | Documented capability, confirmed in PoC |
| 3 — Adequate | Meets core requirements; gaps on the edges | Documented, but not exercised in PoC; or partial coverage |
| 2 — Weak | Material gaps on important aspects | Workarounds required; feature immature or roadmap-only |
| 1 — Poor | Fails the requirement | Documented absence, or PoC failure |

Two rubric rules: (a) **a 3 is "adequate", not "fine"** — in a competitive selection, 3s and below are where the decision is lost; (b) **score what you verified, not what you read** — a feature demoed in vendor slides but never exercised in the PoC is a 3 at most.

### 4.3 The Weighted Score — Calculation

For each platform *P* and criterion *i*:

> **WeightedScore(P) = Σ (weightᵢ × scoreᵢ) / Σ weightᵢ**

with weights as percentages (summing to 100) and scores on the 1–5 scale, the weighted total lands on the same 1–5 scale. A worked row:

- Deployment: weight 15%, score 4 → 0.15 × 4 = 0.60
- Model access: weight 10%, score 5 → 0.10 × 5 = 0.50
- …and so on; sum all nine contributions.

A platform scoring **≥ 4.0** is a clear fit; **3.5–4.0** is a viable but conditional fit (look at where the 2s and 3s sit — a 2 on *governance* is disqualifying for a bank even if the total is 3.8); **< 3.5** is a non-fit absent a compelling strategic reason.

### 4.4 The Worked Scoring — Example

A generic regulated-enterprise profile (weights from §3.10 defaults) scoring three candidates:

| Criterion | Weight | Platform A (hyperscaler managed) | Platform B (framework platform) | Platform C (open-source framework) |
|---|---|---|---|---|
| Deployment model | 15% | 5 (0.75) | 4 (0.60) | 3 (0.45) |
| Model access | 10% | 5 (0.50) | 3 (0.30) | 5 (0.50) |
| Tool integration | 12% | 4 (0.48) | 5 (0.60) | 4 (0.48) |
| Observability | 12% | 4 (0.48) | 5 (0.60) | 2 (0.24) |
| Governance | 10% | 5 (0.50) | 3 (0.30) | 1 (0.10) |
| Security | 12% | 5 (0.60) | 3 (0.36) | 2 (0.24) |
| Cost | 10% | 3 (0.30) | 4 (0.40) | 5 (0.50) |
| Lock-in | 8% | 2 (0.16) | 4 (0.32) | 5 (0.40) |
| Ecosystem | 11% | 4 (0.44) | 5 (0.55) | 3 (0.33) |
| **Weighted total** | **100%** | **4.21** | **4.03** | **3.24** |

*(Illustrative numbers — designed to show the pattern, not a real evaluation.)* The verdict: A and B are both viable; A wins on enterprise surface, B on developer experience. C is eliminated not by any single disaster but by the governance/observability zeros that a regulated estate cannot paper over — exactly the pattern to look for.

### 4.5 The Scoring Caveats

- **Weight inflation:** people over-weight what they can measure (cost) and under-weight what they can't (lock-in, governance). If your weights don't reflect your written requirements, the framework is rationalising.
- **Incomplete data:** you cannot score what you haven't verified. Mark unverified criteria as "not scored" rather than guessing a 3 — an honest gap forces the PoC to close it.
- **Bias:** the "shiny object" bias (§7.4) inflates scores for the newest platform; the "safe choice" bias inflates the incumbent cloud. Mitigation: score against the rubric with evidence notes, score blind to vendor names in a second pass, and have a colleague independently score the same evidence.
- **The rubric trap:** scores are only as honest as the evidence. A table full of 4s and 5s with no evidence column is a wishlist.

---

## 5. The Decision Matrix

### 5.1 Use-Case Mapping — Use Case → Platform

Different agent workloads belong on different platform categories. The mapping below encodes the experience of the 2025–2026 deployments (and the use-case catalogue in [llm_agent_use_cases.md](llm_agent_use_cases.md)):

| Use case | Natural platform category | Representative platform | Why |
|---|---|---|---|
| Customer support (conversational) | Cloud-agent platform / enterprise app platform | Copilot Studio, Agentforce, Bedrock Agents | Needs channels, CRM/ticketing data, HITL, and enterprise auth out of the box |
| Research / analysis agents (internal) | Framework platform / API-agent | LangGraph Platform, CrewAI, OpenAI Agents SDK | Developer-heavy, tool-rich, evolving logic — code-first wins |
| Coding agents | Framework platform / API-agent | LangGraph Platform, AgentKit, open-source (smolagents) | Deep tool integration (IDE, repos), constant iteration, expert builders |
| Workflow / back-office automation | Cloud-agent platform / low-code | Copilot Studio, Vertex Agent Builder (Agent Garden), CrewAI AMP | Connectors to enterprise apps; makers can compose; needs audit trails |
| CRM-native automation (sales/service) | Enterprise application platform | Agentforce | The agent must live inside the CRM's data and flows |
| Regulated core processes (payments, AML) | Cloud-agent platform in-VPC, or self-hosted framework | Bedrock Agents/AgentCore in VPC, LangGraph Platform self-hosted | Data residency + immutable audit + approval workflows are non-negotiable |
| Research / experimentation / air-gapped | Open-source framework | AutoGen, AG2, smolagents, Semantic Kernel | Zero cost, full control, auditable code |

### 5.2 The Shortlist — Quick Filters

Before scoring, cut the field with cheap filters. Three filters eliminate 80% of candidates in minutes:

1. **Hosted vs self-hosted:** if your estate mandates in-VPC/self-hosted, every hosted-only platform (Copilot Studio, Agentforce, AgentKit managed services) is out. If you have no ops team, every self-host-only option (AutoGen, AG2, smolagents, Semantic Kernel) is out.
2. **Budget:** per-seat platforms price out of high-volume internal automation; per-token platforms need budget caps you may not have. If the pricing model can't match your volume profile (§3.7), filter out.
3. **Integration needs:** if the agents must reach Salesforce, filter to platforms with a Salesforce connector or MCP path; if they must reach M365, Copilot Studio jumps the queue; if they must reach your core banking APIs, MCP + custom-tool support becomes the filter.

Result: **3–4 candidates** for the scored PoC (§6). The decision matrix §5.1 informs the filters — a support-agent selection and a research-agent selection will legitimately shortlist different platforms.

### 5.3 The Decision Tree

```
Do your agents have to run inside your boundary (VPC / on-prem / air-gap)?
├─ YES → Can a managed platform run in-boundary (Bedrock/Vertex in-VPC, LangGraph self-hosted)?
│        ├─ YES → Are you cloud-standardised? 
│        │        ├─ YES → Bedrock Agents/AgentCore or Vertex Agent Builder
│        │        └─ NO  → LangGraph Platform self-hosted or CrewAI on-prem
│        └─ NO  → Do you have agent-engineering depth?
│                 ├─ YES → Open-source framework (LangGraph, CrewAI, AutoGen/AG2, smolagents)
│                 └─ NO  → You need a platform, not a framework — revisit the managed options
└─ NO (hosted is acceptable)
   ├─ Do you need model freedom (multi-model, BYO-key)?
   │  ├─ YES → Bedrock Agents/AgentCore, Vertex Agent Builder, or OpenAI Agents SDK (+AgentKit)
   │  └─ NO  → Is the workload inside a business application (CRM / M365)?
   │          ├─ YES → Agentforce (CRM) or Copilot Studio (M365) — fastest in-app value
   │          └─ NO  → Developer-led workload?
   │                  ├─ YES → LangGraph Platform or CrewAI AMP (framework velocity)
   │                  └─ NO  → Copilot Studio / Agent Garden (low-code, makers)
```

*(Opinion — my synthesis of the category logic in §2 and the use-case mapping in §5.1; the tree is a heuristic, not a law. It compresses the §4 scoring into a decision path for the common cases, and it is deliberately biased toward the platform answer: for most enterprises, "managed platform in the right category" beats "framework we assemble ourselves".)*

---

## 6. The Selection Process

### 6.1 Discovery — Landscape Scan and Requirements Gathering

- **Landscape scan:** maintain the candidate list (§1.4) as a living table — this market renames itself quarterly (LangGraph Cloud → LangGraph Platform → LangSmith Deployment; Bedrock Agents → AgentCore). Re-scan at the start of every selection; don't reuse last year's shortlist.
- **Requirements gathering:** interview the stakeholders (business, engineering, risk/compliance, ops) and produce a written requirements doc: use cases and volumes, deployment boundary, model policy, tool inventory (what must the agents reach), observability expectations, governance/audit obligations, budget envelope, and the non-negotiables. The requirements doc is the input to §4.1's weights. Templates exist in the [llm_agent_use_cases.md](llm_agent_use_cases.md) and the governance practice in [implementing-responsible-ai.md](implementing-responsible-ai.md).
- **Output:** requirements doc + candidate landscape table.

### 6.2 Shortlist — Filter to 3–4

Apply the quick filters (§5.2) to cut the field, then sanity-check the survivors against the decision matrix (§5.1). Keep **3–4 candidates**: enough for a real comparison, few enough that each gets a genuine PoC. Add one "wildcard" if the landscape is shifting fast — but commit to eliminating it on evidence, not hope.

### 6.3 Proof-of-Concept — Build the Pilot Agent

The PoC is where selection is actually decided. Rules:

- **Build the real pilot agent** — the actual first production use case, with real tools and real data, not a "hello world" agent. An unrealistic PoC is pitfall §7.5.
- **Same agent, every platform:** the pilot spec is fixed (same tools, same eval set, same volume target); platforms differ only in how they implement it. This is what makes scores comparable.
- **Include the ops surface:** deploy, trace, evaluate, and (for enterprise candidates) exercise RBAC, audit, and HITL in the PoC — the demo of the happy path is worth little.
- **Duration:** 2–4 weeks per platform for a realistic pilot (see §6.6).

### 6.4 Evaluation — Score the Pilot

Score the PoC evidence against the rubric (§4.2) with the weights from §4.1. Each criterion's score cites pilot evidence ("MCP server connected in day 2; eval harness ran 200 golden cases; trace export worked via OTel"). Run the caveats (§4.5): independent second scoring, honest gaps marked unscored, vendor-bias check.

### 6.5 Decision — Build vs Buy vs Framework

Three distinct answers, and the selection produces one:

- **Buy (managed platform):** the platform meets the weighted bar, the PoC passed, and the ops/gov surface is real. You accept its abstractions and pay for the runtime. — The default for most enterprises.
- **Framework (assemble):** the workload needs control the managed platforms can't give (exotic orchestration, air-gap, cost at extreme scale), and you have the engineering depth. You accept building the ops stack ([agentops_guide.md](agentops_guide.md) is your roadmap).
- **Build (in-house platform):** rare and expensive — you're building an agent *product* for others, or no vendor fits a hard requirement (e.g. air-gapped core-banking agent infrastructure at national scale). If you're a bank, "build" should mean "framework + in-house ops layer", never "write our own agent runtime". Cross-ref [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) for what that looks like.

The decision is recorded with the scoring table, the PoC evidence, and the chosen platform's risk register (lock-in, vendor viability, migration path) — a selection without a written record is not a selection, it's an opinion.

### 6.6 The Process Table

| Stage | Activities | Outputs | Duration |
|---|---|---|---|
| Discovery | Landscape scan; stakeholder interviews; requirements doc | Requirements doc; candidate landscape table | 1–2 weeks |
| Shortlist | Quick filters; decision-matrix sanity check | 3–4 candidates with rationale | 3–5 days |
| Proof-of-concept | Build the same pilot agent on each candidate; exercise ops surface | PoC build notes; evidence per platform | 2–4 weeks per platform (parallelise) |
| Evaluation | Rubric scoring with evidence; independent re-score | Weighted scoring tables; gap list | 1 week |
| Decision | Build/buy/framework call; risk register; written record | Decision memo; procurement input | 2–3 days |

Total: **4–8 weeks** for a rigorous selection. Faster is possible for low-stakes choices; slower is normal when procurement, security review, and vendor contracting run in parallel. *(Opinion — durations are my planning synthesis for a mid-size enterprise, not vendor guidance.)*

---

## 7. The Pitfalls

### 7.1 Framework-vs-Platform Confusion — "We Need a Framework" vs "We Need a Platform"

**The symptom:** the team debates LangGraph vs CrewAI vs AutoGen for a month, when the actual decision is *who runs the runtime*. Most enterprises that "need a framework" actually need a platform and are about to spend 6–12 months building the ops surface the platform bundles ([agentops_guide.md](agentops_guide.md) documents exactly that surface: tracing, evals, monitoring, guardrails, cost).

**The test:** if your team can't name who runs tracing, evals, scaling, auth, and audit for the chosen framework — you need a platform, not a framework. Conversely, if your requirement is an air-gapped runtime with bespoke orchestration, a managed platform is the wrong question entirely.

### 7.2 Lock-In — The Proprietary Trap

**The symptom:** six months in, the agents are tangled in platform-specific abstractions (Bedrock agent definitions, Copilot Studio flows, Agentforce topics); the export path nobody checked is a dead end; the "we can always move" assumption was never tested.

**The prevention:** treat lock-in as a scored criterion (§3.8) *before* signing, and engineer against it from day one: tools via MCP (portable by construction — [mcp_framework_tools_guide.md](mcp_framework_tools_guide.md)), agent logic in version-controlled code with thin platform bindings, continuous export of traces/evals/audit logs in open formats (OTel). A platform you can *leave* is a platform you can *negotiate with*.

### 7.3 Over-Engineering — A Platform for the Simple

**The symptom:** a full multi-agent platform, three vendors, and a governance committee for an agent that answers five FAQs. The mirror-image of §7.1: the platform question is real, but the *workload* doesn't need one. 

**The prevention:** start simple. The series' consistent guidance applies — "workflows for the core, agents for the edges" ([production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md)) and "most queries don't need an agent". Selection should be *scaled to the workload*: a single low-risk internal agent can start on the cheapest viable option (even a framework) and be migrated when the portfolio justifies a platform. The platform is a *portfolio* decision, not a *demo* decision.

### 7.4 The Shiny Object — Hype and the Newest Platform

**The symptom:** the selection starts because someone saw a demo of the newest platform; scores inflate for whatever launched last quarter; the incumbent is scored harshly for being boring. This market manufactures shiny objects quarterly (the §1.4 landscape changes names, not just features).

**The prevention:** evaluation discipline (§4): weights before scores, evidence-column scoring, independent re-scoring, and a standing rule that "new" is a *risk* item (immature tooling, unproven vendor) until the PoC proves otherwise. Ask of every candidate: *what has this platform actually run in production, in our industry, at our volume?* — and require a reference, not a case-study PDF.

### 7.5 Pilot Failure — The Unrealistic PoC

**The symptom:** the PoC is a demo agent ("book a meeting with GPT-4o on two platforms") that never touches real tools, real data, real volumes, or real governance — so the scores are fiction and the production agent fails in week one with a tool-permission problem the PoC never exercised.

**The prevention:** the PoC spec (§6.3) must be the *actual first production use case*, with the real tool inventory, a real eval set, and real (or representative) data, and must exercise the ops surface — tracing, evals, RBAC, audit, HITL. If a platform can't complete the real pilot, that is the finding; don't soften the pilot to fit the platform.

### 7.6 The Pitfalls Table

| Pitfall | Symptom | Prevention |
|---|---|---|
| Framework-vs-platform confusion | Long framework debates; nobody owns the runtime/ops; "we'll assemble it" with no ops team | Run the §7.1 test; decide platform-vs-framework from requirements, not taste |
| Lock-in | Platform-specific abstractions everywhere; no export path; "we can always move" untested | Score portability (§3.8) pre-signing; MCP for tools; code-first agent logic; continuous OTel export |
| Over-engineering | Full platform + governance for trivial workloads; multi-vendor stack for one FAQ agent | Scale selection to workload; start simple; treat platforms as portfolio decisions |
| The shiny object | Scores favour the newest platform; incumbent scored down for boringness; demo-driven selection | Weights before scores; evidence columns; "new" scored as risk until PoC proves it |
| Pilot failure | Demo PoC, fictional scores, production failures in week one | Real pilot spec (§6.3): real tools, real data, real eval set, ops surface exercised |

---

## 8. Worked Example: A Bank Chooses an Agent Platform

### 8.1 The Scenario

A mid-size bank (the same scenario as the agents series — the customer-support agent and the internal research agent of [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md), [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md), and [agentops_guide.md](agentops_guide.md)) is selecting an agent platform to take those two pilots to production, with more agent use cases (compliance review, trade-matching support, internal knowledge) already in the pipeline. The bank is **AWS-primary** (a legacy VPC estate) with a small but competent agent-engineering team (4 engineers) and a strict risk/compliance function.

### 8.2 The Requirements

Gathered in discovery (§6.1), the non-negotiables and priorities:

- **Deployment:** runtime **must run inside the bank's VPC** or a self-hosted boundary; no agent data may leave the boundary (MAS TRM-aligned, [financial_risk_compliance_systems_guide.md](../../banking/financial_risk_compliance_systems_guide.md)). → Hosted-only platforms eliminated (Copilot Studio, Agentforce, AgentKit managed).
- **Model access:** multi-model with BYO-key; the bank already contracts Anthropic and OpenAI directly and wants negotiating leverage. 
- **Tool integration:** **MCP support is mandatory** — the bank's tool inventory (core-banking APIs, CRM, document store) is being wrapped as MCP servers ([mcp_framework_tools_guide.md](mcp_framework_tools_guide.md)).
- **Observability:** full tracing + evals + monitoring, with exportable traces (the AgentOps discipline in [agentops_guide.md](agentops_guide.md)).
- **Governance/security:** RBAC, immutable audit, approval workflows for high-impact actions; SSO (Entra ID); data residency in-region (Singapore); SOC 2 + ISO 27001 evidence.
- **Budget:** predictable, usage-based pricing; no per-seat model (agents are automation, not seats).
- **Ecosystem:** AWS-native integrations preferred; community/talent availability for the framework layer.

### 8.3 The Weights

From the requirements (sum = 100%):

| Criterion | Weight | Rationale |
|---|---|---|
| Deployment model | 15% | VPC mandate is the hardest constraint |
| Model access | 10% | BYO-key + multi-model is a stated priority |
| Tool integration (MCP) | 15% | The tool inventory is the agent's entire value |
| Observability | 12% | Production ops per the AgentOps bar |
| Governance | 12% | Regulatory audit + approval workflows |
| Security | 13% | Residency, SSO, certs |
| Cost | 8% | Usage-based, controllable |
| Lock-in | 8% | Portability engineered, but scored |
| Ecosystem | 7% | AWS-native + community |

### 8.4 The Scores

Candidates shortlisted (§5.2: in-VPC filter, MCP filter, budget filter): **Amazon Bedrock Agents/AgentCore**, **LangGraph Platform (self-hosted)**, and **Vertex AI Agent Builder** (Google is the bank's secondary cloud for analytics — included for genuine comparison). Scores from the PoC evidence (the support agent + research agent pilots, real MCP tools, real eval set):

| Criterion | Weight | Bedrock Agents/AgentCore | LangGraph Platform (self-hosted) | Vertex Agent Builder |
|---|---|---|---|---|
| Deployment model | 15% | 5 (0.75) — native in-VPC | 5 (0.75) — self-hosted in boundary | 5 (0.75) — in-VPC on GCP, but new region/network for the bank |
| Model access | 10% | 5 (0.50) — widest catalog + BYO-key | 4 (0.40) — BYO-key, no catalog | 4 (0.40) — good catalog, BYO-key |
| Tool integration (MCP) | 15% | 5 (0.75) — MCP servers native, PoC-verified | 5 (0.75) — MCP first-class, PoC-verified | 4 (0.60) — MCP supported; PoC hit connector friction |
| Observability | 12% | 4 (0.48) — CloudWatch + X-Ray; evals assembled ourselves | 5 (0.60) — LangSmith tracing/evals native | 4 (0.48) — Cloud Logging + Vertex evals; export OK |
| Governance | 12% | 5 (0.60) — IAM, CloudTrail, Bedrock guardrails, approval patterns | 3 (0.36) — RBAC/HITL good; audit export needs our build | 4 (0.48) — IAM/audit solid |
| Security | 13% | 5 (0.65) — in-region, SOC 2/ISO, KMS | 3 (0.39) — we operate it; certs are ours to provide | 4 (0.52) — in-region (singapore), SOC 2/ISO |
| Cost | 8% | 4 (0.32) — usage-based, predictable with caps | 3 (0.24) — self-hosted ops + platform license; 1M-node Lite cap noted | 3 (0.24) — usage-based, GCP egress/new-region costs |
| Lock-in | 8% | 2 (0.16) — AWS-runtime-bound; MCP keeps tools portable | 4 (0.32) — code + MCP portable; LangGraph itself open | 3 (0.24) — GCP-bound |
| Ecosystem | 7% | 5 (0.35) — AWS-native, big community | 4 (0.28) — largest agent-framework community | 3 (0.21) — ADK growing, smaller in-region talent pool |
| **Weighted total** | **100%** | **4.56** | **4.09** | **3.92** |

*(Illustrative scores — designed to be realistic for the scenario, not a real vendor evaluation; the pattern, not the numbers, is the lesson.)*

### 8.5 The Decision

**Chosen: Amazon Bedrock Agents / AgentCore.** The weighted totals (4.56 vs 4.09 vs 3.92) confirm the qualitative read: for a bank that is AWS-primary with a VPC mandate, AgentCore's in-VPC managed runtime, native MCP support (PoC-verified), widest model catalog with BYO-key, CloudTrail audit, in-region residency, and AWS-native ecosystem beat LangGraph Platform's superior developer experience and observability. LangGraph Platform was the runner-up and remains the fallback if the team's custom-orchestration needs outgrow AgentCore's declarative model — and because agent logic was kept code-first and tools MCP-based, that fallback is genuinely exercisable. Vertex was eliminated on ecosystem fit (GCP is the bank's secondary cloud, not its agent home) despite scoring respectably.

Two explicit risk items were recorded with the decision: **(a)** AgentCore is the *newer* runtime (Bedrock Agents Classic is in maintenance mode) — the bank mitigates by pinning the pilot to the GA feature set and re-verifying at each release; **(b)** runtime-level lock-in — mitigated by MCP tools, code-first agent definitions, and continuous trace/audit export.

### 8.6 The Lesson — Choose the Platform, Not the Hype

The bank did not choose "the best agent platform" — it chose **the platform that fit its requirements**, and the scoring made that visible: the winner won on the weighted criteria (deployment, tools, security, ecosystem) and lost on lock-in, and the bank *bought the lock-in knowingly* because the scoreboard said the fit was worth it. The discipline — weights before scores, real PoC, evidence columns, recorded risks — is what separates this outcome from the pitfall outcomes of §7. A bank that had picked "the newest platform" or "the framework our engineers love" would have gotten a worse fit at higher risk. **Fit, not features** (§10).

---

## 9. The Future (2026+)

### 9.1 Platform Consolidation — The Hyperscaler Platforms

The hyperscalers are converging on the same product shape — a managed agent runtime on your cloud (Bedrock AgentCore, Vertex Agent Engine, Azure AI Foundry agent tooling — see [enterprise_ai_platforms_guide.md](enterprise_ai_platforms_guide.md) for the platform survey) — and the market is consolidating around them for enterprise workloads. The consequence for selection: **the cloud you already trust increasingly determines the platform you should choose**, which makes the platform decision *easier* (fewer credible options) but *more consequential* (you're doubling down on the cloud). The counterweight is the open layer: MCP and OTel keep tools, traces, and agent logic portable even as runtimes consolidate.

### 9.2 MCP as the Standard

MCP has become the USB-C of agent tooling: adopted by OpenAI, Google, Microsoft, AWS, and the framework ecosystem, and now a first-class selection criterion (§3.3). The trend for selection is profound — **the tool layer is the portable part of your agent stack**. A bank that wraps its core-banking APIs as MCP servers can move agents across platforms without rebuilding integrations ([mcp_framework_tools_guide.md](mcp_framework_tools_guide.md)). The remaining risk is MCP *governance* — permissions, auth, and injection defense at the protocol layer — which is the next battleground.

### 9.3 The Framework-to-Platform Maturation

Every major framework is growing a platform: LangGraph → LangGraph Platform (→ LangSmith Deployment), CrewAI → CrewAI AMP/Enterprise, ADK → Agent Engine, OpenAI Agents SDK → AgentKit, AutoGen → Microsoft Agent Framework (unifying AutoGen + Semantic Kernel at v1.0). The selection implication: **the framework-vs-platform boundary (§1.3) is shifting under everyone's feet** — today's framework choice is tomorrow's platform commitment, and vice versa. Choose with the maturation trajectory in view: a framework with a credible platform path (LangGraph, CrewAI, ADK) de-risks the "we started with a framework and now need a platform" migration that dominated 2025 horror stories.

### 9.4 The Open Source — OTel and Open Agents

The open-source counterweight to consolidation: **OpenTelemetry's GenAI conventions** are becoming the standard for agent telemetry export (flagged in [agentops_guide.md](agentops_guide.md) as still settling — the spec's stability is work-in-progress), and open agent *runtimes* are emerging (Strands Agents SDK from AWS, Microsoft Agent Framework, the LangGraph runtime, open-source AgentCore-competitors). The selection trend: **insist on OTel-compatible export** even when buying managed — it is the cheapest lock-in insurance available, and it's what makes a future migration (or a future multi-platform estate) affordable.

### 9.5 Trends Summary

| Trend | Direction | Selection impact |
|---|---|---|
| Hyperscaler consolidation | Managed agent runtimes converge on the big three clouds | Cloud trust increasingly decides platform choice; fewer credible candidates |
| MCP as the standard | MCP becomes the universal tool layer | Tool portability neutralises a chunk of lock-in; MCP governance is the new risk |
| Framework → platform maturation | Every framework grows a managed runtime | Framework choices are forward-platform bets; pick frameworks with credible platform paths |
| Open source & OTel | Open runtimes + standard telemetry export | Cheapest lock-in insurance; insist on OTel export even in managed platforms |
| Agent marketplace/registry | Enterprises catalogue and govern agents as products | Governance criteria (§3.5) keep rising in weight |

---

## 10. Summary: Agent Platform Selection in One Page

**The landscape.** The candidates are the agent-native platforms: LangGraph Platform (LangChain), OpenAI Agents SDK + AgentKit, Microsoft Copilot Studio, Salesforce Agentforce, AWS Bedrock Agents/AgentCore, Google ADK + Vertex Agent Builder, CrewAI Enterprise/AMP, and the open-source frameworks (AutoGen, AG2, smolagents, Semantic Kernel). They sort into five categories — cloud-agent platforms, agent-framework platforms, API-agent platforms, enterprise application platforms, and open-source frameworks — and each category has a natural workload.

**The criteria.** Nine criteria decide the choice: deployment model, model access, tool integration (MCP), observability, governance, security, cost, lock-in, ecosystem. Non-negotiables are elimination filters, not weights.

**The scoring.** Requirements → weights → scores → decision. Weights before scores. Rubric-scored (1–5) with evidence. Weighted total = Σ(weight × score). ≥4.0 clear fit; 3.5–4.0 conditional; <3.5 non-fit. Beware weight inflation, incomplete data, and bias.

**The matrix and the process.** Use-case → platform mapping shortlists candidates; quick filters (hosted/self-hosted, budget, integrations) cut the field to 3–4; a *real* pilot on each candidate, scored with evidence, produces the decision: build, buy, or framework — in 4–8 weeks.

**The pitfalls.** Framework-vs-platform confusion, lock-in, over-engineering, the shiny object, and the unrealistic PoC — each with a known symptom and prevention.

**The worked example.** A bank with a VPC mandate, MCP tooling, and AWS-primary estate scored Bedrock Agents/AgentCore (4.56) over LangGraph Platform self-hosted (4.09) and Vertex Agent Builder (3.92) — and bought the lock-in knowingly, because the fit was the point.

> **The final word: fit, not features.** The best agent platform is not the one with the most impressive demo — it is the one whose deployment model, tool story, ops surface, and governance match *your* requirements, weights, and constraints. Score with discipline, pilot with reality, decide with evidence — and when the hype arrives (it will, quarterly), the scoreboard is your defence. Choose the platform, not the hype.

---

## 11. Glossary

| Term | Definition |
|---|---|
| **Agent platform** | A product that provides the runtime, orchestration, tool-integration, observability, and governance to build, deploy, and operate AI agents — managed or self-hosted |
| **Agent framework** | A library (SDK) providing the agent programming model — loop, tools, state — that you embed in your own code and run yourself (LangGraph, CrewAI, AutoGen, smolagents, Semantic Kernel, ADK, OpenAI Agents SDK) |
| **LangGraph Platform** | LangChain's platform for deploying and operating LangGraph agents (formerly LangGraph Cloud; deployment branding now under LangSmith Deployment); cloud, Self-Hosted Lite, and enterprise self-hosted editions |
| **LangChain** | The company (and its open-source LLM application framework ecosystem) behind LangGraph, LangSmith, and LangGraph Platform |
| **OpenAI Agents SDK** | OpenAI's open-source, provider-agnostic agent framework (Python/TS; agents, handoffs, guardrails, sessions; 100+ LLMs) |
| **AgentKit** | OpenAI's 2025 toolkit for building, deploying, and optimizing agents — orchestration, connectors, evals, frontend components — layered on the Agents SDK |
| **Copilot Studio** | Microsoft's low-code agent-builder (conversational + autonomous agents) integrated with Microsoft 365 Copilot, Power Platform, and Azure AI Foundry; pay-as-you-go or credits |
| **Agentforce** | Salesforce's CRM-native agent ("digital labor") platform — Agent Builder, Atlas reasoning engine, Data Cloud grounding; 2.0 GA in early 2025 |
| **Bedrock Agents** | AWS's managed agent service on Amazon Bedrock — declarative agents, action groups, knowledge bases, memory, guardrails, multi-agent collaboration |
| **AgentCore** | AWS's next-generation managed agent runtime (successor to Bedrock Agents Classic, now in maintenance mode); runtime instances = persistent managed compute, GPU-capable, sessions up to 14 days |
| **Vertex Agent Builder** | Google's managed agent stack on Vertex AI: Agent Garden (no-code) + Agent Engine (managed runtime for ADK agents), Gemini-native, VPC-integrated |
| **ADK (Agent Development Kit)** | Google's open-source agent framework (April 2025; Python/TS/Go/Java/Kotlin) that deploys to Vertex Agent Engine |
| **CrewAI** | Role-based multi-agent framework, plus CrewAI AMP/Enterprise platform (no-code studio, control plane, observability; cloud or on-premise) |
| **AutoGen** | Microsoft's open-source conversation-driven multi-agent framework (~55K stars); after the Nov 2024 governance split, in maintenance mode, direction moved to Microsoft Agent Framework |
| **AG2** | The community fork of AutoGen (renamed post-split); continues the AutoGen programming model under community governance |
| **smolagents** | HuggingFace's minimal open-source agent library — code agents (model writes Python, ~1,000-LOC core) |
| **Semantic Kernel** | Microsoft's open-source, embeddable AI orchestration SDK (C#/Python/Java); enterprise successor is the Microsoft Agent Framework (v1.0) |
| **Hosted** | The vendor runs the agent runtime and ops surface; you consume it as a service |
| **Self-hosted** | You run the runtime in your own environment (on-prem, VPC, air-gap); you operate it |
| **MCP (Model Context Protocol)** | Open standard (Anthropic, Nov 2024; adopted by OpenAI/Google/Microsoft/AWS) connecting agents to tools/data servers — the universal tool-integration layer |
| **BYO-key** | Bring-your-own-key: calling your own model-provider accounts/keys from inside a platform, for pricing control and model flexibility |
| **RBAC** | Role-based access control — per-user/per-team permissions over building, deploying, and operating agents |
| **SSO** | Single sign-on — identity-provider integration (SAML/OIDC/Entra ID) for platform access |
| **Data residency** | Where agent data (prompts, traces, tool payloads) physically resides; regional pinning and no-exfiltration guarantees |
| **SOC 2** | The baseline security/compliance certification for SaaS platforms (trust-services criteria); often the minimum bar for enterprise agent platforms |
| **Lock-in** | The cost/difficulty of leaving a platform — runtime abstractions, data export, tool bindings |
| **Portability** | The degree to which agent logic, tools, and data can move across platforms (MCP tools + code-first logic + OTel export maximise it) |
| **Pricing model** | How the platform bills: per-seat, per-token/per-message, per-agent/per-action — each with different cost behaviour at scale |
| **Weighted scoring** | The evaluation method: criterion weights (summing 100%) × rubric scores (1–5) → weighted total per candidate |
| **Rubric** | The score definition table (1–5 with evidence anchors) that makes scores comparable and defensible |
| **Decision matrix** | The use-case → platform mapping used to sanity-check shortlists and route workloads to platform categories |
| **Decision tree** | The heuristic question flow (hosted? model freedom? enterprise?) that compresses the scoring into a fast routing path |
| **Shortlist** | The 3–4 candidates surviving the quick filters, taken forward to the scored PoC |
| **Proof-of-concept** | The pilot build: the real first production agent, built identically on each candidate, exercising tools, data, and the ops surface |
| **Pilot** | The bounded production-realistic trial of the platform — the evidence source for scoring |
| **Build-vs-buy** | The final decision: buy a managed platform, assemble with a framework, or build an in-house agent platform layer |
| **Over-engineering** | Selecting a heavyweight platform/governance for a workload that doesn't need it; the antidote is start-simple and scale selection to the workload |
| **Fit-not-features** | The guide's thesis: choose the platform that matches your requirements, weights, and constraints — not the one with the best demo |
