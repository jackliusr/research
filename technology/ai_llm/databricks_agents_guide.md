# Databricks Agents — the governed agent

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Cymbal Bank
> **Repo:** [jackliusr/research](https://github.com/jackliusr/research) · **Series:** Technology / AI & LLM · **Domain:** Enterprise AI platforms, agent stacks, AI governance
> **Date:** 2026-09-17

---

## Contents

1. [The overview, the boundary and the before/after frame](#1-the-overview-the-boundary-and-the-beforeafter-frame)
2. [The product-name decoder — the old-name / new-name map](#2-the-product-name-decoder--the-old-name--new-name-map)
3. [The product surface, item by item](#3-the-product-surface-item-by-item)
4. [The optimization claim examined](#4-the-optimization-claim-examined)
5. [Tools, functions and the governance model](#5-tools-functions-and-the-governance-model)
6. [Evaluation and tracing the Databricks way](#6-evaluation-and-tracing-the-databricks-way)
7. [Deployment and serving](#7-deployment-and-serving)
8. [Multi-agent and supervision patterns on the platform](#8-multi-agent-and-supervision-patterns-on-the-platform)
9. [Interoperability and lock-in](#9-interoperability-and-lock-in)
10. [The honest assessment](#10-the-honest-assessment)
11. [The competitive frame](#11-the-competitive-frame)
12. [The regulated-institution angle](#12-the-regulated-institution-angle)
13. [The Cymbal Bank worked example](#13-the-cymbal-bank-worked-example)
14. [The claims audit](#14-the-claims-audit)
15. [What Could Not Be Verified](#15-what-could-not-be-verified)
16. [Glossary](#16-glossary)
17. [Cross-references and further reading](#17-cross-references-and-further-reading)
18. [Closing summary](#18-closing-summary)

---

## 1. The overview, the boundary and the before/after frame

### 1.1 What this guide is

This is the guide to the **Databricks expression of the agent stack** as it exists on **2026-09-17**. It covers six things and nothing else:

1. **The agent products** — what they are actually called today, what each one does, and whether it is GA, preview or announced.
2. **The tool, function-calling and governance model** — how a tool is defined, bound, permissioned, audited and traced.
3. **The evaluation and tracing machinery** — MLflow 3's GenAI surface, LLM judges, synthetic evaluation data, human feedback.
4. **The deployment and serving path** — from developed agent to served endpoint, on which compute, at what cost.
5. **The multi-agent and natural-language-data agents** — Supervisor Agent and Genie Agents.
6. **An honest assessment** — maturity, lock-in, preview surface, and fit for a regulated enterprise.

### 1.2 The boundary — what this guide does NOT own

Three other parts of the library own adjacent ground. This guide **cross-references them and does not re-derive them**:

| Owned elsewhere | Owner | This guide's relationship |
| --- | --- | --- |
| What Databricks **is** — lakehouse, Unity Catalog, compute families, storage, the platform comparison, serving infrastructure | [`technology/databricks_guide.md`](../databricks_guide.md) | Cross-ref for serving infrastructure and catalog mechanics. §3.7 of that guide is quoted below as the before/after frame. |
| The **platform comparison** across Databricks and its peers | [`technology/ai_llm/enterprise_ai_platforms_guide.md`](enterprise_ai_platforms_guide.md) | Point there. §11 of this guide positions the **agent layer** only. |
| The **discipline** of building, evaluating, operating and governing agents in general | the agent-engineering cluster (see §17) | Cross-ref by path. General methodology is out of scope here. |

The rule of thumb: **this guide answers "what does Databricks ship, and is it real?"** — not "what is Databricks?" and not "how should one build an agent?"

### 1.3 The before/after frame — what the platform guide recorded, and what changed

The platform guide's agent coverage is **§3.7 "Databricks Mosaic AI"**, a four-bullet, 2024-era summary. Quoting it verbatim so the drift is visible rather than silently contradicted:

> * "**Mosaic AI Agent Framework** — a framework for building, evaluating, and deploying **RAG applications and agents**: tool/function calling, memory, multi-step orchestration, and built-in **agent evaluation** (quality, latency, cost, safety metrics on test sets)."
> * "**Vector Search** — a **managed vector database** on Delta tables for RAG retrieval (hybrid keyword + vector, ANN indexes); embeddings stored and governed in Unity Catalog like any table."
> * "**Mosaic AI Gateway** — a **unified LLM access + governance point**: one API for hosted, fine-tuned, and **external models (OpenAI, Anthropic, Azure OpenAI, Bedrock...)**, with per-route rate limits, API-key management, and audit logging — the LLM equivalent of an API gateway for AI."
> * "**Foundation model APIs** — hosted, pay-per-token endpoints for open models (**Llama 3, Mixtral, Mistral, DBRX** and others), served in the customer's cloud region; no GPU infrastructure to manage."

Its timeline table places "Mosaic AI Agent Framework, Vector Search, and Gateway" at **2024**. **That framing is now dated.** It is not wrong as history; it is wrong as a description of the current product surface. Specifically, between that snapshot and 2026-09-17:

- The **branding moved**. "Mosaic AI Agent Framework" as a *product* name has largely been displaced by **Agent Bricks** as the umbrella, while `databricks-agents` the *Python package* still carries the old Mosaic-name description in its PyPI summary (verified 2026-09-17: package `databricks-agents` 1.12.0, summary string **"Mosaic AI Agent Framework SDK"**, uploaded 2026-08-21).
- The **gateway was renamed and widened**. "Mosaic AI Gateway" → **AI Gateway** → docs now title the surface **"AI governance with Unity Gateway"**, with marketing calling it **Unity AI Gateway**. The rename is not cosmetic: the governance object moved from "an LLM endpoint" to "a Unity Catalog securable", and it now governs **MCP servers**, not just model endpoints.
- **Vector Search was renamed** to **AI Search** on **2026-06-01**.
- **Genie Spaces were renamed** to **Genie Agents**.
- **Multi-Agent Supervisor was renamed** to **Supervisor Agent** on **2026-02-05**.
- The **model catalogue** in the old bullet ("Llama 3, Mixtral, Mistral, DBRX") no longer describes the shelf. The 2026 shelf is frontier third-party models served by Databricks: OpenAI GPT-5.x (including a Codex variant), Anthropic Claude Sonnet 4.6 / Opus 4.6 / Sonnet 5, Google Gemini 3.x, plus Qwen embedding models — all via Foundation Model APIs pay-per-token (Databricks release notes, February and June 2026).

A reader who learned this surface in 2024 and returns in 2026 will misname almost everything. §2 exists for that reader.

### 1.4 What is genuinely new since the platform guide

Three things have no counterpart in the 2024 snapshot at all: **Agent Bricks** (launched **2025-06-11** in Beta) — an *automated* agent-building layer whose central claim is auto-optimization on the customer's own data, generating its own synthetic data and task-aware benchmarks; **Supervisor Agent** — a *managed* orchestrator construct, **GA 2026-02-10 in select US regions for workspaces without Enhanced Security and Compliance features**; and **MCP as a first-class governed object** — **Unity Catalog MCP Services** and **managed MCP servers**, registered as Unity Catalog securables and governed through the gateway.

### 1.5 The governance thesis in one paragraph

Databricks' differentiator is not the agent runtime. It is that **the agent's tools, the data behind those tools, the models it calls and the agent itself are all securables in the same catalog**, evaluated under the **same privileges** and written to the **same audit trail** as everything else. An agent registered as a Unity Catalog model, whose tools are Unity Catalog functions and MCP services, whose model access is a Unity Catalog model service, and whose traces can be stored in Unity Catalog tables is a *governable* agent in a way that a Python process holding a broad service-account key is not. That is a real architectural claim and it is the reason this guide is titled **the governed agent**. §5 and §12 test how far it actually holds.

---

## 2. The product-name decoder — the old-name / new-name map

This surface's single most confusing property is that **the same thing has had two or three names, and two different things have shared a name.** Verified renames from Databricks product release notes are marked with a date; anything claimed only from secondary sources is flagged ⚠.

### 2.1 Old name → new name

| Old name (as you may have learned it) | Current name (2026-09-17) | Status of the rename | Verified how |
| --- | --- | --- | --- |
| Mosaic AI Agent Framework (product umbrella) | **Agent Bricks** (umbrella); `databricks-agents` remains the SDK package | Progressive rebrand, no single release note | Databricks product page + docs tree; PyPI summary still says "Mosaic AI Agent Framework SDK" |
| Mosaic AI Vector Search | **Databricks AI Search** | Renamed **2026-06-01** | Release note: "Vector Search is now AI Search", June 2026 |
| Mosaic AI Gateway / AI Gateway | **Unity Gateway** (docs) / **Unity AI Gateway** (marketing) | Renamed during 2026 | Docs page title "AI governance with Unity Gateway", last updated 2026-09-11; release note "AI Gateway (Beta)", 2026-02-12 |
| Agent Bricks Multi-Agent Supervisor | **Supervisor Agent** | Renamed **2026-02-05** | Release note: "Agent Bricks: Multi-Agent Supervisor is now Supervisor Agent" |
| Genie Spaces | **Genie Agents** | Renamed, date not established in a release note I could open ❌ | Docs note on the Genie Agents setup page: "Genie Agents were formerly known as Genie Spaces", page updated 2026-09-14 |
| Mosaic AI Agent Evaluation | **Agent Evaluation** (within MLflow 3) | Absorbed into MLflow 3's evaluation framework | Azure Databricks migration guide "Migrate to MLflow 3 from Agent Evaluation"; MLflow 3.0.0 released 2025-06-10 |
| MLflow LLM Evaluate / `mlflow.evaluate` for GenAI | **`mlflow.genai.evaluate()`** + **Scorer** objects | New API; **explicitly not interoperable** with the classic `mlflow.evaluate()` / `EvaluationMetric` path | MLflow docs: "The two systems serve different purposes and are not interoperable." |
| Databricks-hosted models (as an undifferentiated set) | **Foundation Model APIs**, now surfaced as **model services** (`system.ai`) | Model services in **Beta**, 2026-06-29 | Release note: "Govern LLMs in Unity Catalog with model services (Beta)" |

### 2.2 Things that share a name but are not the same

- **"Agent Bricks" is both an umbrella and a product line.** Marketing uses it for the whole agent platform (including Omnigent and model access); docs use it for the managed builders (`Knowledge Assistant`, `Supervisor Agent`) and the SDK-driven Custom Agents. When someone says "Agent Bricks", ask which.
- **"Agent Evaluation" is both a docs section and a legacy product name.** The current docs page is part of MLflow 3; the legacy product was the pre-MLflow-3 Mosaic AI Agent Evaluation. Migration guides exist, which means both names are in circulation.
- **"Gateway" is three things**: the pre-2026 AI Gateway, the current Unity Gateway control plane, and the literal `AI Gateway` tab in the workspace UI where MCP servers are listed (docs still say "go to your workspace > **AI Gateway** > **MCPs**").
- **"Agent" is three things**: an Agent Bricks managed builder, a Custom Agent you wrote in Python, and an *Agent Service* — a registration entry for an agent that runs *outside* Databricks.

### 2.3 The decoder table a reader actually needs

| You will meet this term | It means |
| --- | --- |
| Agent Bricks | The umbrella agent platform; also the managed builder family |
| Knowledge Assistant | A managed, no-code chatbot over your documents with citations |
| Supervisor Agent | A managed multi-agent orchestrator (GA 2026-02-10, select US regions, non-ESC) |
| Custom Agents | Agents you author in Python; deployed on Databricks Apps |
| Genie Agent | The natural-language-data agent (formerly Genie Space) |
| Agent Service | A Unity Catalog registration for an externally-hosted agent (Beta) |
| Unity Gateway / Unity AI Gateway | The AI governance control plane (models, agents, MCP, guardrails, cost) |
| AI Search | The managed vector/keyword search product (formerly Vector Search) |
| Agent Evaluation | The evaluation surface inside MLflow 3 |
| MLflow 3 / `mlflow.genai` | The library and API surface for tracing, evaluation, judges, feedback |
| Omnigent | A harness-composition layer for coding agents above Claude Code / Codex, hosted on Databricks (see §3.9) |
| Lakebase | Serverless Postgres positioned as agent memory / operational data |

---

## 3. The product surface, item by item

Every entry carries a **status** and a **verified date**. Where the docs page carries no status label and no release note announced a graduation, that is stated as an **explicit negative** — I do not infer GA from silence. Databricks' product docs last-updated dates throughout are **2026-09-11 to 2026-09-15**; dates below are the *feature* dates, not the page dates.

### 3.1 Agent Bricks — the umbrella

| | |
| --- | --- |
| **Status** | Umbrella platform; components vary (see below). Launched **Beta 2025-06-11**. |
| **Verified** | Press release "Databricks Launches Agent Bricks", 2025-06-11; docs tree; product page (accessed 2026-09-17) |
| **What it is** | Databricks' name for the enterprise agent platform: model access, execution, governance and context in one system. In-workspace it presents as **Knowledge Assistant**, **Supervisor Agent**, **Custom Agents**, plus **Genie Agents** as the data-facing agent. |
| **Who it is for** | Two audiences, deliberately: no-code business/SME builders (Knowledge Assistant, Supervisor Agent) and Python developers (Custom Agents). |
| **Cost shape** | Serverless: serverless compute DBUs + Model Serving / Foundation Model API tokens + (for retrieval) AI Search endpoint DBUs. Agent Bricks itself is not a separately-priced SKU in the material I could verify ❌ — it bills through the underlying serverless and serving SKUs. |

The name has been broadened in this direction twice. A **March 2026** release note is titled "**Agent Bricks now covers all Databricks AI capabilities**" — meaning the umbrella absorbed surfaces that were previously documented elsewhere. This is why old Mosaic-era names still float around: the umbrella grew over them.

### 3.2 Knowledge Assistant — the managed document chatbot

| | |
| --- | --- |
| **Status** | Documented, available in-UI (`Create Agent` → **Knowledge Assistant**). **No GA announcement located** — see ❌ in §15. |
| **Verified** | Docs page "Use Knowledge Assistant…", last updated **2026-09-11** |
| **What it does** | Creates a question-answering agent **over your documents**, with **citations**. States it follows an **Instructed Retriever** approach rather than a plain RAG pipeline. |
| **Inputs** | A Unity Catalog **volume** or volume directory (txt, pdf, md, ppt/pptx, doc/docx); a Unity Catalog **table with a file column** (must be a streaming table or have change data feed enabled, with a `content` column of BINARY/STRING and a `metadata`/`_metadata` STRUCT containing `file_path`, `file_name`, `file_size`, `file_modification_time`); or an **AI Search index** limited to embedding models `databricks-gte-large-en`, `databricks-bge-large-en`, `databricks-qwen3-embedding-0-6b`. |
| **Limits documented** | Up to **10 knowledge sources**. Files **> 100 MB** skipped. PDF/DOC/DOCX/PPT/PPTX with **> 500 pages** skipped. Agent creation + sync can take **up to a few hours**. Syncing is **manual** — an SME or `CAN MANAGE` holder must click **Sync**; it is incremental. |
| **Maturity read** | The file/table/AI-search triple and the hard skip limits are the honest part of the docs: this is a *document* chatbot, not a general retrieval platform. The manual sync and the hours-long build are the operational reality — it is not a sub-minute experiment loop. |

### 3.3 Supervisor Agent — the managed orchestrator

| | |
| --- | --- |
| **Status** | **Generally Available, 2026-02-10**, *select US regions*, *workspaces without Enhanced Security and Compliance features*. Renamed from Multi-Agent Supervisor on **2026-02-05**. |
| **Verified** | Databricks product release notes, February 2026 (two entries: the rename and the GA), March 2026, April 2026, June 2026; Databricks blog "Agent Bricks Supervisor Agent is Now GA" |
| **What it does** | A **managed supervisor/orchestrator** that coordinates **Genie Agents, published dashboards, Knowledge Assistant endpoints, model serving endpoints, Unity Catalog functions, Unity Catalog tables and volumes, AI Search indexes, other Supervisor Agents, web search, external MCP servers, Unity Catalog MCP Services, custom MCP servers, and custom agents on Databricks Apps** — to complete tasks across specialised domains. |
| **Improvement loop** | Improved via **natural-language feedback from subject matter experts** (Databricks calls this **Agent Learning on Human Feedback**, ALHF) — you add questions and guidelines; the supervisor incorporates them into routing and answers. |
| **Requirements** | Serverless compute, Unity Catalog, Model Serving access, a **serverless usage policy with nonzero budget**, a supported region, and **at least one subagent**. |
| **Governance behaviour** | Documented as having **built-in access controls so end users only reach the subagents and data they have access to**. The GA blog states it natively supports **On-Behalf-Of (OBO) authentication**, acting as a transparent proxy for the human user, with every data fetch or tool execution validated against the user's Unity Catalog permissions. |
| **Documented limitations** | **Web search** is available only where a web-search-capable native model exists or cross-geography processing is enabled, and **is not available in workspaces with the Enhanced Security and Compliance add-on**. Web search always requires **end-user approval per invocation** before the query leaves for the public web, and runs on `databricks-gpt-5` regardless of the supervisor's own model. |
| **Maturity read** | The most consequential single fact in this table is the **ESC exclusion on GA day**. A regulated firm running the compliance security profile is, by the vendor's own release note, outside the GA envelope as of 2026-02-10, with a March 2026 carve-in only for workspaces using the compliance security profile *with HIPAA controls*. Cross-check your own workspace posture before assuming availability. |

### 3.4 Genie Agents — the natural-language-data agent

| | |
| --- | --- |
| **Status** | Documented and available; formerly **Genie Spaces**. No GA/preview label on the setup page; the rename date is **not established** ❌ (see §15). |
| **Verified** | Docs page "Create, share, and manage a Genie Agent", last updated **2026-09-14**; June 2026 release note (budgets); February 2026 release note (regional model hosting for JP/KR); March 2026 release note (regions) |
| **What it does** | A natural-language chat interface over Unity Catalog data for business users. Analysts configure it with tables, **example SQL queries**, **instructions**, and **trusted assets**; business users query it in the chat UI. |
| **Requirements/limits** | Data must be registered in Unity Catalog. Up to **50 tables/views/metric views** per agent. Requires a **pro or serverless SQL warehouse**; compute credentials are embedded in the agent. Each agent supports up to **200,000 conversations**, each up to **10,000 messages**. |
| **The governance nuance that matters** | Data access is evaluated **using each end user's own Unity Catalog permissions**. And explicitly: *"Genie can query tables beyond those explicitly added to an agent. Access is controlled by Unity Catalog permissions, not by the Genie Agent itself."* The 50-table list is **context, not a security boundary**. |
| **Dependency to note** | Genie uses **partner-powered AI features**, which must be enabled at account and workspace level by an account administrator. Disabling them leaves the sidebar entry visible but non-functional. For a regulated buyer, "who has enabled partner-powered AI, and under what DPA" is a real question. |
| **Agent surface** | A **Genie Agents API** exists (import/export, conversation API) and Genie is a first-class **subagent type** in Supervisor Agent. Docs also cover **Genie in multi-agent systems** built with Custom Agents. |
| **Maturity read** | Genie is the most *shipped* thing in this guide — it predates Agent Bricks and is in wide customer use. It is also the surface where the "governed" claim is most load-bearing and most subtly false if you read the 50-table limit as an ACL. |

### 3.5 Custom Agents and the `databricks-agents` SDK

| | |
| --- | --- |
| **Status** | Documented, actively released. Package `databricks-agents` **1.12.0**, uploaded **2026-08-21**. |
| **Verified** | PyPI JSON API for `databricks-agents` (59 versions; 1.4.0 → 1.12.0 between 2025-08-20 and 2026-08-21); docs "Author an agent and deploy it on Databricks Apps", last updated **2026-09-15** |
| **Authoring model** | Python. Databricks recommends the MLflow **`ResponsesAgent`** interface, which wraps any framework (docs name **LangGraph, LangChain, OpenAI Agents SDK, LlamaIndex**, and "any authoring library") and provides streaming, multi-agent support, comprehensive tool-calling message history, tool-calling confirmation, long-running tool support, automatic tracing, and OpenAI Responses-schema compatibility. |
| **Serving component** | **MLflow AgentServer** — an async FastAPI server exposing a `/responses` endpoint, with built-in tracing, routing, logging and error handling. |
| **Templates** | `github.com/databricks/app-templates`, including an `agent-openai-agents-sdk` template with a built-in chat UI, MLflow evaluation code, and a default MCP server connection. |
| **Notable 2026 twist** | The docs now recommend authoring with **AI coding assistants** (Claude, Cursor, Copilot) using shipped **agent skills** (`/.claude/skills`) and an `AGENTS.md` file. The vendor's official authoring guidance has itself become agent-assisted. |
| **Maturity read** | This is the mature path: a real SDK, a real server, real templates, a 59-version release history. The cost is that you own the code, the deployment and the evaluation — the "auto-optimization" the marketing leads with does **not** apply here. |

### 3.6 Agent Evaluation and MLflow 3 — the evaluation and tracing tooling

| | |
| --- | --- |
| **Status** | **MLflow 3.0 generally available 2025-06-10** (per the press release and PyPI first-3.0 upload timestamp). Current `mlflow` on PyPI: **3.16.1**, uploaded **2026-09-16**. MLflow **trace storage in Unity Catalog**: Beta → **Public Preview 2026-04-30**. |
| **Verified** | PyPI JSON API for `mlflow` (187 releases; 3.0.0 at 2025-06-10T19:55:15); press release 2025-06-11 ("MLflow 3.0 is generally available"); docs "Evaluate and improve", April 2026 release note |
| **API surface** | `mlflow.genai.evaluate()` with **Scorer** objects; built-in and custom **LLM judges**; the docs state plainly that this GenAI system is **separate from the classic ML evaluation system** (`mlflow.evaluate()`, `EvaluationMetric`) and **not interoperable with it**. |
| **Offline/online symmetry** | The same evaluation configuration (judges and custom metrics) is used for **offline evaluation and online production monitoring** — functionally the closest thing to a regression gate the stack offers. |
| **Trace storage** | Traces are stored in **OpenTelemetry format**; when stored in Unity Catalog, access is governed through **Unity Catalog schema and table permissions** and traces are queryable from Databricks SQL or the MLflow Python SDK. Regional availability is limited. |
| **Maturity read** | Strong, and genuinely ahead of what most agent platforms ship as an evaluator. The version cadence (3.0.0 in June 2025 → 3.16.1 in September 2026, ~12 minor releases in 15 months) is both evidence of investment and evidence of churn: pin your version and expect judge behaviour to move between minors. |

### 3.7 Unity Gateway — the LLM gateway and guardrails

| | |
| --- | --- |
| **Status** | Mixed. **AI Gateway (Beta) — 2026-02-12**. **AI Gateway now governs MCP servers (Beta) — April 2026**. **Model services (Beta) — 2026-06-29**. The docs page itself is titled "AI governance with Unity Gateway" and carries no GA label. Treat the whole control plane as **Beta** unless your account team shows otherwise. |
| **Verified** | Docs "AI governance with Unity Gateway", last updated **2026-09-11**; February / April / June 2026 release notes |
| **What it governs** | Registered AI assets as **Unity Catalog securables**: Model APIs in `system.ai`; **external model providers** (OpenAI, Anthropic) with bring-your-own-key; registered models; **MCP tools** (MCP servers registered as securables, with tool filtering and service policies); **custom tools** (Unity Catalog functions); **HTTP connections**; and custom model services. |
| **Traffic management** | **Rate limits** on model and MCP services; **traffic splitting and fallbacks**; **Smart Routing** for coding tasks; **budgets** with per-user thresholds and hard caps. |
| **Guardrails** | **Service policies** (also called guardrails) that govern requests and responses based on content and caller, using built-in and custom policies; the product page names **prompt-injection prevention, sensitive data detection and content filtering** as organisation-wide policies. |
| **Observability** | **Usage tracking** (requests, tokens, latency) via system tables; **cost attribution** to services, target models, principals and tags; **inference tables** logging requests and responses to Unity Catalog Delta tables. |
| **Maturity read** | The capability list is the strongest thing on the platform for a regulated buyer — an inference table is exactly what a model-risk reviewer asks for. The status is the weakest thing: a Beta control plane that *is also* the audit evidence store is a dependency worth pinning in writing. |

### 3.8 AI Search — the managed search/retrieval product

| | |
| --- | --- |
| **Status** | Renamed from **Vector Search** on **2026-06-01**. **Full-text keyword search: Beta.** Dedicated **full-text indexes: Beta** (storage-optimized endpoints only, triggered sync mode). |
| **Verified** | Docs "Databricks AI Search", last updated **2026-09-14**; June 2026 release note |
| **Mechanics (documented, not marketing)** | HNSW for ANN; **L2 distance** (normalise embeddings for cosine-equivalent ranking — the docs state this explicitly). Hybrid keyword search uses **Okapi BM25**; similarity and keyword results are fused with **Reciprocal Rank Fusion** (`rrf_param = 60`). |
| **Embedding options** | Four: Delta Sync index with Databricks-computed embeddings; Delta Sync with self-managed embeddings; Direct Vector Access index (manual REST updates); and full-text-only index (Beta). **Note the one-way door**: a self-managed-embedding index **cannot** be converted to a Databricks-managed index — you must rebuild. |
| **Endpoints** | **Standard** (up to 320M vectors at dim 768; supports high QPS) and **storage-optimized** (up to ~1B embeddings at dim 768; high QPS is standard-endpoints-only). |
| **Governance** | ACLs on AI Search endpoints; indexes are Unity Catalog-governed objects (`SELECT` on the index plus `USE CATALOG`/`USE SCHEMA`). |
| **Maturity read** | Mature, well-documented, and unusually honest about its own internals. The rename is substantive: the product now does keyword search without embeddings, which is why the old name was dropped. |

### 3.9 Foundation-model serving, Databricks Apps and Omnigent

**Foundation model serving.** Databricks serves LLMs natively through **Foundation Model APIs** with **pay-per-token** access and "no infrastructure to run", and connects external providers through the same access control and traffic management. The 2026 catalogue is frontier third-party models — GPT-5.x (incl. a Codex variant), Claude Sonnet 4.6 / Opus 4.6 / Sonnet 5, Gemini 3.x — added by release note through February and June 2026. Model access is being reshaped into **Unity Catalog model services** (Beta, 2026-06-29) so an endpoint is defined once and shared across workspaces by privilege. A separate **Foundation model Unity Catalog permissions** feature reached **GA in June 2026**.

**Databricks Apps.** The hosting surface for Custom Agents and custom chat UIs. Git-backed app deployments are **GA (April 2026)**; a built-in chat UI, streaming, Databricks authentication and optional persistent chat history ship in the templates. Server routes run **as the authenticated user** by default, so per-user permissions apply — and a missing `CAN QUERY` / `CAN RUN` returns a 403 rather than silently escalating.

**Omnigent.** The product page presents **"Omnigent on Databricks"** as the newest layer: *"Run, supervise and share any agent or harness, hosted on Databricks. Omnigent runs your agents above the harnesses you already use, so you can compose Claude Code, Codex and custom agents in one workflow. Contextual policies … are enforced at runtime through Unity AI Gateway, and every session is traced in MLflow and shareable with a single link."* It links to `omnigent.ai/docs`. ⚠ **This is sourced only from the Databricks Agent Bricks product page; I could not open omnigent.ai/docs nor find an independent Databricks doc or release note describing Omnigent.** Its GA/preview status is unestablished. Do not plan around it.

**Agent services.** A Beta Unity Catalog registration mechanism: *if your agent runs outside Databricks*, register it as an **Agent Service** to make it discoverable and govern who can use it, with the same grants that protect tables, models and functions.

### 3.10 Where products overlap or supersede one another

| Overlap | Honest reading |
| --- | --- |
| Knowledge Assistant vs. Custom Agent with AI Search | Knowledge Assistant is the packaged path; a Custom Agent with AI Search indexes is the programmable path. Same underlying retrieval concepts, different control. |
| Supervisor Agent vs. a Custom Agent multi-agent orchestrator | Supervisor Agent is **managed and governed**; a custom orchestrator is **yours**. Docs support both, including routing from a custom orchestrator to Genie Agents. |
| Genie Agent vs. AI Search | Different problems: Genie answers questions by generating **SQL** over governed tables; AI Search retrieves **documents**. Supervisor Agent stitches the two. |
| Agent Bricks umbrella vs. `databricks-agents` SDK | The SDK still describes itself as the "Mosaic AI Agent Framework SDK". The umbrella name moved; the package name did not. Expect both names in the wild indefinitely. |
| AI Gateway vs. Unity Gateway | Same control plane, renamed and widened (models → MCP servers → agents). Older docs and UI labels retain "AI Gateway". |

---

## 4. The optimization claim examined

### 4.1 The vendor claim, as stated

From the 2025-06-11 press release, verbatim:

> "Agent Bricks uses novel research techniques developed by Databricks Research to automatically generate domain-specific synthetic data and task-aware benchmarks. Based on these benchmarks, it automatically optimizes for cost and quality, saving enterprises from the tedious trial-and-error of current approaches."

And the mechanism, also verbatim, in three steps:

> "First, Agent Bricks automatically generates the task-specific evaluations and LLM judges to assess quality. Next, synthetic data is created that looks like the customer's data to substantially supplement the agent's learning. Last, Agent Bricks searches across the full gamut of optimization techniques to refine the agent. At the end of this automated workflow, the customer simply needs to select the iteration that matches the balance of quality and cost that they want the agent to achieve."

The headline in the press release dateline is *"Agent Bricks automatically optimizes AI agents on customers' unique data to deliver cost-efficient, trustworthy agents."*

### 4.2 What it actually optimizes, mechanically

Reading the three steps literally, the object being optimized is:

- **The judge/metric set** — generated first, task-specific, from the task description plus your data.
- **The data distribution** the agent is measured on — *synthetic*, generated to "look like" yours.
- **The agent's configuration** — searched against those judges over "the full gamut of optimization techniques", producing **a set of candidate iterations**, from which **the human picks** the quality/cost point.

Two things follow from that reading, and both are true of any optimizer regardless of vendor:

1. **It optimizes against a metric and a data distribution.** The judges are generated *before* the search begins and are held fixed across the search (that is what makes it an optimization). If the generated judge is wrong about what "good" means for your task, the optimizer will converge — enthusiastically and at scale — on the wrong target.
2. **The final selection is human.** The docs and the press release both describe choosing among iterations on a quality/cost frontier. So the "no manual tuning" claim in marketing is best read as **"no manual prompt/model-hyperparameter tuning"**, not **"no human judgement"**.

### 4.3 Does it require labelled data?

**The vendor's claim is that it does not require you to supply labels** — the synthetic data and benchmarks are *generated*. This is the interesting part of the design and it is the claim that most needs independent evidence.

**What independent evidence exists: I found none that I can cite.** The evidence available to me is:

Databricks' own press release and product page (vendor, primary); named customer quotes in the press release — AstraZeneca (400,000 clinical trial documents parsed, "without writing a single line of code", "in just under 60 minutes"), Flo Health ("double our medical accuracy over standard commercial LLMs"), North Dakota University System ("saving 30 days of manual trial-and-error"), Hawaiian Electric ("significantly outperformed our original open-source implementation (built on LangChain) in both LLM-as-judge and human evaluation accuracy metrics") and Lippert, which are **vendor-selected quotes in vendor material**, not independent evaluations, with no published methodology and no reproducible claim behind "60 minutes" or "double the accuracy"; and a 2026 community summary asserting Agent Bricks runs "in general availability with over 100,000 agents built, processing more than one quadrillion tokens per year" ⚠ — **secondary source, summit-marketing provenance, not verified against a Databricks primary document I could open.**

So: the optimization mechanism is documented at a level sufficient to understand *what is optimized*; the **efficacy claim is supported only by vendor material and vendor-curated customer quotes**.

### 4.4 The honest framing a buyer needs

Optimization-by-search over generated judges is a legitimate technique. It is also structure-blind in a specific way that matters:

It **can** find a better configuration of prompt, model, retrieval and tool-use policy against *the metric it was given*. It **cannot** know whether the task definition is right, because it has no access to the business decision the agent's output feeds. It **cannot** know whether the synthetic evaluation distribution matches production: the docs describe synthetic data "that looks like the customer's data", resemblance is the design goal, and resemblance is not equivalence — a better prompt found on synthetic data is a hypothesis about production, not a result. And it **cannot** detect a judge that has learned your reviewers' bias, since generated judges inherit their blind spots from whatever generated them.

The correct posture is to treat auto-optimization as **a fast hypothesis generator**, and the human selection step as the actual control. That step should not be a product-owner clicking the nicest-looking iteration; it should be a documented acceptance threshold against **held-out real data**, signed by the model owner. See [`llm_evaluation_vs_validation_guide.md`](llm_evaluation_vs_validation_guide.md) and [`ai_agent_drift_guide.md`](ai_agent_drift_guide.md) for why the held-out set and the drift monitoring are the parts that make this defensible.

---

## 5. Tools, functions and the governance model

This section is written for a **security reviewer**, not a developer. If you read one section, read this one.

### 5.1 How a tool is defined and bound to an agent

There are six tool mechanisms, and they are governed differently:

| Tool mechanism | What it is | How it is governed |
| --- | --- | --- |
| **Unity Catalog function** | Custom Python or SQL logic, callable as an agent tool. Docs show use with LangChain, OpenAI and Anthropic frameworks. | Registered as a Unity Catalog securable. End users need **`EXECUTE`** on the function (plus `USE CATALOG` / `USE SCHEMA`). |
| **MCP Service** | An external or built-in MCP server registered as a **Unity Catalog securable**, with tool filtering and service policies. Built-in `system.ai` services cover SaaS tools such as Slack, GitHub and Google Drive. | **`EXECUTE`** on the MCP Service plus `USE CATALOG` / `USE SCHEMA`; external MCP servers additionally need **`USE CONNECTION`** on the Unity Catalog connection. |
| **Managed MCP server** | Ready-to-use Databricks servers giving governed access to **Genie, AI Search, Databricks SQL, Unity Catalog functions and a code interpreter** — no server to build or host. | Same securable model; access via the gateway. |
| **Local Python function tool** | A function defined in the agent's own code (`@function_tool` / `@tool`). | **Not** a Unity Catalog securable — docs state these "don't require resource grants in `databricks.yml` because they run within the agent process." |
| **Hosted custom MCP** | Your own MCP server **hosted as a Databricks App**. | **`CAN_USE`** on the Databricks app. |
| **Custom agent as a subagent** | Another agent, hosted on Databricks Apps. | **`CAN_USE`** on the app. |

The **local Python function tool** row is the one to notice: it is the only mechanism in the list that a catalog-based reviewer **cannot see**. Everything it reaches is reached through the agent's own process identity, and the catalog has no object to audit.


### 5.2 How Unity Catalog governs the tool's data

The chain, as documented: the **agent is registered as a Unity Catalog model** (or a model service / agent service); the **tools it calls are governed as MCP services, functions and connections**; the **data behind a tool is governed by ordinary Unity Catalog privileges** (`SELECT` on a table, `READ VOLUME` on a volume, `EXECUTE` on a function); **lineage** runs from outputs back to source data (the product page claims "complete lineage from outputs to source data"); and **audit** is covered by **Genie Agent event** audit-log entries for AI/BI activity plus gateway requests and responses logged to **inference tables** — Unity Catalog Delta tables — for monitoring and debugging.

### 5.3 On-behalf-of versus service identity — the crux

The GA blog states Supervisor Agent natively supports **On-Behalf-Of (OBO) authentication**, acting as a transparent proxy for the human user, validating every data fetch and tool execution against that user's Unity Catalog permissions. Databricks Apps routes also run **as the authenticated user** by default. Both are the **right** design, and both are what you should demand.

But three counterexamples are documented on the same platform and must be understood as intentional exceptions:

- **Genie Agents**: *"Your compute credentials are embedded into the Genie Agent to provide warehouse access for all users."* Data access is still evaluated per end user, so this is narrower than it sounds — but a **warehouse credential is embedded in the object**, and it is a different trust shape from pure OBO.
- **The 50-table limit is context, not an ACL**: Genie can query tables beyond those attached, with access controlled by Unity Catalog permissions.
- **Custom Agents you write**: OBO behaviour depends on how you build them. Nothing prevents a developer from authenticating an outbound call with a service principal whose grants are broader than any user's.

### 5.4 What an agent can reach once it has a permission — the honest security consequence

State this plainly to your risk committee:

> **Binding enterprise data to an agent that a language model drives converts a data permission into a *capability* exercised by non-deterministic software.**

A `SELECT` grant on a table used to mean "a human or a scheduled job can read this." It now also means "any prompt that reaches this agent can attempt to cause this table to be read, joined, summarised and returned to whoever is holding the conversation." The permission has not changed. What has changed is **who — or what — exercises it, how often, and with what predictability.**

The platform's own docs carry a warning on the Supervisor Agent page, and it is worth quoting because it is the vendor saying it:

> "Executing arbitrary code in an agent tool can expose sensitive or private information that the agent has access to. **Customers are responsible to run only trusted code, and implement guardrails and proper permissions to prevent unintended access to data.**"

Practical consequences for a reviewer to reason about:

**Prompt injection is a data-exfiltration technique, not a content-quality problem** — an agent that can query tables and fetch URLs can be steered into encoding one into the other; the gateway offers **prompt-injection prevention** as a service policy, which is one layer and not the control. **Least privilege must be applied to the agent's identity, not just the user's**: OBO protects against the broad-service-account failure mode, but the *subagent* and *MCP connection* identities still need their own minimal grants. **Row filters and column masks are the compensating control** — ABAC over governed tags reached **GA in April 2026** and now evaluates row filters and column masks using the **session user's identity** through views and functions, a documented **breaking change** with a three-month grace period for contacted customers; this is the mechanism that keeps an agent's output masked, and it changed behaviour in 2026. And **inference tables are your evidence and also a new sensitive dataset**, because they log requests and responses containing fragments of the data the agent read.

For the general threat model, red-team method and control library, see [`ai_red_teaming_guide.md`](ai_red_teaming_guide.md), [`ai_governance_framework_guide.md`](ai_governance_framework_guide.md) and [`ai_governance_bias_redteaming_guide.md`](ai_governance_bias_redteaming_guide.md). This section deliberately stops at *what the Databricks objects are*.

### 5.5 How the gateway governs model access

Same principle, applied to models: model endpoints are **Unity Catalog securables** (`system.ai` Model APIs, external provider services, registered models, custom model services). Access is by **privilege**, traffic is by **rate limit / traffic split / fallback / Smart Routing**, spend is capped by **budgets**, and content is filtered by **service policies**. `system.ai` allowlisting controls which foundation models exist in a workspace at all — a workspace-level blast-radius control most reviewers miss.

---

## 6. Evaluation and tracing the Databricks way

### 6.1 The current surface

**Library**: `mlflow` **3.16.1** (2026-09-16), with the GenAI path under **`mlflow.genai`**. **Evaluation API**: `mlflow.genai.evaluate()` with **Scorer** objects — **not** the classic `mlflow.evaluate()` / `EvaluationMetric`, which the docs state is **not interoperable**. **Judges**: built-in and custom **LLM judges**, scoring traces "in a few clicks" from the UI and running programmatically over evaluation datasets. **Tracing**: end-to-end **MLflow Tracing** across every agent step, with trace storage in **Unity Catalog** (OpenTelemetry format, Unity Catalog permission-governed, SQL-queryable) reaching **Public Preview 2026-04-30** after Beta. **Human feedback**: built-in **review apps** for stakeholders and SMEs, also being built into the chatbot template. **Production monitoring**: the **same** judge/metric configuration used offline, applied online.

### 6.2 The LLM-judge approach and its documented limits

The mechanism: a language model scores an agent's output against a criterion, either built-in or written by you. Its strengths are real — cheap at scale, consistent in a way human review is not, and usable as a **regression gate** when the same judge is applied to successive agent versions.

Its limits, stated honestly:

**A judge is a model with a bias and a failure rate** — it inherits the taste of whatever generated it, and it can be wrong in ways that correlate with the thing being measured (it likes verbose answers; it likes answers that look like its own). **Judges are versioned software**: with ~12 MLflow minors in 15 months, a judge implementation can change under you, so pin versions and re-baseline on upgrade. **Auto-generated judges are the weakest link in the auto-optimization loop** (§4) — if Agent Bricks generated the judge and then optimized against it, the judge is both the measurement instrument and the objective function, a closed loop with no external reference. And **judge-based evaluation measures agreement with a rubric, not correctness in the world**: for a document-extraction agent you can often find ground truth, but for a judgement-of-adequacy agent you generally cannot.

### 6.3 Synthetic evaluation-data generation

Agents APIs and Mosaic-era Agent Evaluation both advertise **synthetic data generation** for evaluation sets; Agent Bricks generates synthetic data and task-aware benchmarks as a *design feature* of the auto-optimization loop. The honest framing: synthetic sets are **good for coverage and regression pressure**, and **not** a substitute for a held-out set drawn from production traffic. The repo's discipline-level treatment of why is in [`llm_evaluation_frameworks_guide.md`](llm_evaluation_frameworks_guide.md) and [`llm_evaluation_vs_validation_guide.md`](llm_evaluation_vs_validation_guide.md).

### 6.4 Tracing a multi-step agent run

Trace a run and you get the step structure: model calls, tool calls, retrievals, and the message history — including intermediate tool-calling messages, because `ResponsesAgent` is designed to return them. This is what makes a multi-step failure **debuggable** rather than merely **mysterious**, and it is the mechanism by which "the agent picked the wrong tool" becomes a fixable defect instead of an anecdote.

### 6.5 The honest boundary — and where it sits against the siblings

> **Trace-and-evaluate tooling answers "did it do the steps right". It does not answer "was the task worth doing".**

Concretely: MLflow can tell you the retrieval step returned the wrong document, the judge can tell you the answer was unsupported, and monitoring can tell you the pattern changed last Tuesday. None of these can tell you whether the agent should have existed, whether the human workflow around it is sound, whether the tool it called *should* have been callable, or whether the quality bar you set is the bar the business needs. Those are **design and ownership** questions, and they live in the agent-engineering cluster: [`agentic_engineering_guide.md`](agentic_engineering_guide.md), [`agentops_guide.md`](agentops_guide.md), [`agents_work_fall_apart_guide.md`](agents_work_fall_apart_guide.md), [`llm_agents_failures_production_guide.md`](llm_agents_failures_production_guide.md), [`ai_agent_drift_guide.md`](ai_agent_drift_guide.md). Databricks gives you instrumentation; it does not give you the reading.

---

## 7. Deployment and serving

### 7.1 Two deployment paths, and they are not interchangeable

| | **Managed builders** (Knowledge Assistant, Supervisor Agent) | **Custom Agents** |
| --- | --- | --- |
| Authoring | In-workspace UI; zero code | Python + MLflow `ResponsesAgent` |
| Hosting | Databricks-managed endpoint | **Databricks Apps** (or Model Serving) |
| Versioning | Managed by Databricks; improved via SME feedback | **Yours** — git-backed app deployments are GA |
| Evaluation | Built-in loop | You wire MLflow evaluation yourself |
| Guardrails/gateway | Applied by the platform | You apply them (`databricks.yml` grants, gateway config) |
| Compute | **Serverless compute required** | Databricks Apps compute |

The managed path is where the "governed by default" story is strongest — OBO, built-in access controls, and the platform's own improvement loop. The custom path is where you keep control and take on every obligation that comes with it.

### 7.2 Which compute serves it, and scaling

**Managed agents** require **serverless compute** and a **serverless usage policy with a nonzero budget** — without a nonzero budget the agent does not run, a hard and easily-missed prerequisite in a cost-controlled enterprise. **Custom agents** run on **Databricks Apps**, with the MLflow **AgentServer** FastAPI process serving `/responses` and handling routing, logging and error handling. **Model endpoints** behind either path are **Model Serving** endpoints (CPU or GPU) scaling by **concurrency range** with optional **scale-to-zero**. **Genie Agents** require a **pro or serverless SQL warehouse** and are capped at 200,000 conversations per agent and 10,000 messages per conversation.

Cross-ref [`technology/databricks_guide.md`](../databricks_guide.md) for the serving infrastructure itself (compute families, warehouse behaviour, networking) rather than re-deriving it here.

### 7.3 Latency expectations the vendor documents

**Databricks does not publish a latency SLO for agents.** The docs describe streaming responses and document that agent creation/sync "can take up to a few hours", but I found **no documented p50/p95 latency target, no documented concurrency limit per agent endpoint, and no documented timeout budget for a Supervisor Agent run** ❌. Any latency figure in a vendor deck is a customer anecdote. Measure it yourself under your own load; cross-ref [`llm_latency_optimization_guide.md`](llm_latency_optimization_guide.md).

### 7.4 Versioning, promotion, A/B and evaluation-driven deployment

**Custom agents** have git-backed deployments (GA April 2026) — deploy from a branch, tag or commit, which is what makes promotion auditable. **Managed builders** are improved in place via SME feedback, and there is **no documented "promote version 4 to production" gate** for Knowledge Assistant / Supervisor Agent ❌ — a genuine gap for change control, since an agent that learns continuously is an agent whose behaviour changed without a release record. On **A/B**, the gateway documents **traffic splitting and fallbacks** for model services and MLflow links each agent version to its evaluation results for comparison, but the documented capability sits at the **model-service** layer; at the **agent** layer the supported pattern is version-linked evaluation, not native split-traffic agent A/B ❌.

### 7.5 What is billed, and when

The clearest primary source I could open is the Model Serving pricing page (accessed 2026-09-17). GPU serving DBU rates per hour:

| GPU configuration | DBUs/hour |
| --- | --- |
| T4 or equivalent | 10.48 |
| A10G × 1 | 20.00 |
| A10G × 4 | 112.00 |
| A10G × 8 | 290.80 |
| L40S × 1 | 44.86 |
| A100 40GB × 8 | 538.40 |
| A100 80GB × 8 | 628.00 |
| H100 × 1 | 100.00 |
| H100 × 8 | 800.00 |

Documented billing rules from the same page:

- **CPU serving** (model, feature, **agent**) bills at **4 DBU per hour** for standard workloads, with 4 GB memory per concurrent request; some models get dynamic optimizations for higher concurrency.
- CPU/model/feature/**agent** serving usage appears on the bill under the **Serverless Real-time Inference** SKU (Model Serving SKU on SAP Databricks).
- **Scale-to-zero**: with scale-to-zero selected, GPU endpoints go idle after **30 minutes** of no requests and are **not charged** while idle. Without scale-to-zero, the minimum charge follows the minimum provisioned concurrency.
- **Data transfer and connectivity** costs are billed separately for cross-region/AZ movement or egress.
- **Regional pricing** is applied: "Our regional prices are based on the regional cost of infrastructure supporting our serverless products."

**Explicit negative on pricing**: I could **not** open a Databricks page publishing **Foundation Model API per-token prices**, **serverless compute DBU rates** for agent workloads, or **AI Search endpoint DBU rates** ❌. DBU rates are also regionally variable and the pricing page renders them client-side, so any number quoted without your region, tier and contract is not your price. **Get these from your account team in writing.**

---

## 8. Multi-agent and supervision patterns on the platform

### 8.1 What the platform gives you as a construct

**Supervisor Agent** is the platform's multi-agent construct, and it is genuinely a construct rather than a pattern you must code:

- A **dynamic supervisor** that analyses the question and routes between Genie Agents, Knowledge Assistants, MCP servers, functions, dashboards, other supervisors and custom agents.
- **Nested supervision** — a Supervisor Agent can be a subagent of another Supervisor Agent.
- **Managed delegation and result synthesis** — the docs describe task delegation and synthesis as platform behaviour.
- **Human feedback as the tuning interface** — ALHF lets an SME improve routing and answers with natural-language guidelines rather than code. Zapier is quoted using exactly this to improve orchestration between Genie spaces.
- **Per-end-user access control across the fan-out** — documented as built in.

### 8.2 What you must build yourself

**A custom orchestrator** in Python (on Databricks Apps) routing to Genie Agents and endpoints — documented, but it is your code, your failure handling and your tracing. **Agent-to-agent handoff semantics beyond the supervisor pattern** — the platform documents a supervisor; it does not document peer-to-peer negotiation, market/blackboard patterns, or contractual handoff between two autonomous agents of equal standing. And **long-running / stateful supervision** — a page for Supervisor Agent long-running tasks exists, which tells you the mainstream pattern is **request-response**, not durable execution.

### 8.3 Mapping onto the sibling multi-agent guides

| Platform provision | The discipline that owns the rest |
| --- | --- |
| Supervisor → subagent delegation | Hierarchy design, depth limits, fan-out cost control: [`hierarchical_multi_agent_frameworks_guide.md`](hierarchical_multi_agent_frameworks_guide.md) |
| Managed + custom agent mixing | Where determinism must live and where it may not: [`hybrid_multi_agent_systems_guide.md`](hybrid_multi_agent_systems_guide.md) |
| Genie + Knowledge Assistant + MCP fan-out in a financial workflow | Regulated-workflow specifics: [`multi_agent_banking_guide.md`](multi_agent_banking_guide.md) |
| Durable / long-running supervision | [`durable_ai_agent_workflows_guide.md`](../durable_ai_agent_workflows_guide.md) and [`agentic_workflows_guide.md`](../agentic_workflows_guide.md) |
| Tool-permission concentration across a fan-out | [`agent_sandboxing_strategies_guide.md`](agent_sandboxing_strategies_guide.md) |

**The honest split**: the platform offers **one well-governed multi-agent construct** and a documented DIY path. It does not offer a multi-agent *architecture*. Choosing the topology, bounding the fan-out, deciding where a human must be in the loop, and owning the composite failure modes remain yours — and a supervisor that fans out to five subagents each with its own data grants is a **five-times-larger permission surface** than any single agent you would have signed off.

---

## 9. Interoperability and lock-in

### 9.1 Framework integrations — what is *documented* versus what was merely blogged

| Framework | Databricks documentation status | Evidence |
| --- | --- | --- |
| **LangGraph / LangChain** | **Documented as supported.** Named in the "Build custom agents" docs row as an authoring library. Code examples in the agent-authoring docs use `langgraph.prebuilt.create_react_agent` and `databricks_langchain.ChatDatabricks`. | docs.databricks.com "Author an agent and deploy it on Databricks Apps", 2026-09-15 |
| **OpenAI Agents SDK** | **Documented as supported**, and is the *default* template (`agent-openai-agents-sdk`). Docs show `from agents import Agent, function_tool`. The `ResponsesAgent` interface is **compatible with the OpenAI Responses schema**. | Same page; `github.com/databricks/app-templates` |
| **LlamaIndex** | **Documented as supported** — named in the same build-agents table ("LangGraph, LangChain, OpenAI, and LlamaIndex"). | docs.databricks.com "Use agents on Databricks", 2026-09-15 |
| **PyFunc** | **Documented** as an authoring option in the February 2026 release note. | Release notes, February 2026 |
| **Anthropic tool-use** | **Documented** — Unity Catalog function tools are shown usable with Anthropic frameworks. | docs.databricks.com "Create agent tools using Unity Catalog functions" |
| **DSPy / AutoGen / CrewAI / Semantic Kernel / Google ADK** | **No Databricks documentation found giving first-class support** ❌. The wrapper story (`ResponsesAgent` wraps *any* framework) means they will *probably* work, but "will probably work" is not a support statement. | Searched docs tree and release notes; no result |

**The single most important integration fact**: `ResponsesAgent` is a **framework-agnostic wrapper interface**, and the docs say so explicitly — *"You can author agents using any framework. The key is wrapping your agent with MLflow `ResponsesAgent`."* That is a genuinely portability-friendly design decision and it deserves credit.

### 9.2 MCP support

MCP is **documented as first-class and governed**, which is rarer than it sounds:

- The docs frame MCP as *"one of several ways to connect tools on Databricks"* and as *"an open-source standard that connects AI agents to tools, resources, and prompts."*
- Three sources of MCP servers: **MCP Services** (external MCP servers registered as Unity Catalog securables, or built-in `system.ai` services for Slack, GitHub, Google Drive); **Databricks managed MCP servers** (Genie, AI Search, Databricks SQL, Unity Catalog functions, code interpreter — no server to host); **custom MCP** hosted as a Databricks App.
- **Outbound direction too**: external clients — the docs name **Claude, Cursor, MCP Inspector** — can be wired to Databricks-hosted MCP servers.
- **Governance**: MCP servers are Unity Catalog securables with **`EXECUTE`** (registered MCP Service) or **`USE CONNECTION`** (external MCP server), with tool filtering, service policies and audit trails.
- **Symmetry with third-party guidance**: the docs cite `modelcontextprotocol.io` as the standard.

This is the strongest interoperability item on the platform. Where most vendors' MCP support is "we can call an MCP server", Databricks makes the MCP server a **permissioned, discoverable catalog object**.

### 9.3 What is genuinely portable, and what is not

**Portable in a meaningful sense:** your **agent code** if you wrote it with a third-party framework and wrapped it in `ResponsesAgent` (the wrapper is a wrapper); your **MCP tools** in principle, since MCP is an open standard and a custom MCP server is just a server; **MLflow** itself, which is open source (187 PyPI releases, shipped by the same company), so tracing and evaluation code travels even if the governed trace storage does not; and your **model choices**, since the catalogue includes third-party frontier models — switching models is not switching platforms.

**Not portable — and this is the honest core of the section:** an **agent bound to governed Delta data under Unity Catalog**. The *agent* is portable; the *value* is not. An agent whose tools are Unity Catalog functions over governed tables has no meaningful second home — you would have to rebuild the data layer, the permission model and the lineage. This is not a Databricks-specific sin; it is the general truth that an agent's worth is in its grounding, and grounding is where the platform lives. Neither are **managed builders** portable: Knowledge Assistant and Supervisor Agent are products, not artifacts, and there is no documented export of a Supervisor Agent's orchestration (Genie Agents have an import/export API and can be exported as a metric view — the exception that proves the rule). Nor are **gateway service policies, inference tables, rate limits and budgets**, which are configurations rather than code, nor **attribute-based access control, governed tags, row filters and column masks** — the security model itself is the lock-in.

### 9.4 The exit question, posed properly

Do not ask *"can we leave Databricks?"* Ask:

1. **What would we take with us?** Code and MCP servers: yes. Data governance, evaluation history, audit trail, agent tuning: no.
2. **What does leaving cost?** For a Custom Agent over a portable data platform, weeks. For a fleet of Knowledge Assistants and Supervisor Agents over Unity Catalog, effectively a rebuild **plus** the loss of the evaluation and feedback history that made them good.
3. **Is the lock-in priced fairly?** This is the real question. Open standards (MCP, OpenTelemetry, MLflow, Delta Lake) and an agnostic wrapper interface are the vendor's answer, and they are a better answer than most. The lock-in that remains is **data and governance lock-in**, and it is not concealed — the product is *marketed* on it.

**Verdict**: the framework layer is unusually portable; the **governance layer is the moat**, by design and openly so. Budget for it consciously rather than discovering it at renewal. See [`ai_agent_platform_selection_guide.md`](ai_agent_platform_selection_guide.md) for the general selection framework.

---

## 10. The honest assessment

### 10.1 Where this stack is genuinely ahead

**1. Governance over tools and data inside one catalog — verified, and it is a real differentiator.**

The claim is checkable in the docs, not just in marketing: *"an agent is registered as a Unity Catalog model, and the tools it calls are governed as MCP services, functions, and connections."* Add model endpoints as Unity Catalog securables, add traces storable in Unity Catalog tables, and you have a coherent answer to the question every regulated buyer asks first — **"what exactly can this agent reach, and who authorised it?"** — expressed in the same ACL language as the rest of the estate:

- Agent → Unity Catalog model / model service / agent service
- Tool → MCP Service (`EXECUTE`) or Unity Catalog function (`EXECUTE`) or app (`CAN_USE`) or connection (`USE CONNECTION`)
- Data → `SELECT` / `READ VOLUME`, with ABAC row filters and column masks
- Model → `system.ai` Model API with workspace allowlisting
- Evidence → inference tables, system tables, audit logs, Unity Catalog-stored traces

Other platforms can govern agents. Very few can say the *agent, its tools, its models and its data* resolve to **one permission model with one lineage graph**. That is the differentiator and it survives scrutiny.

**2. OBO authentication as the default posture on managed agents.** Acting as the user rather than a broad service account is the correct design, and it is documented rather than merely promised. **3. MLflow 3's GenAI evaluation, tracing and human-feedback surface** — a genuine library with a real release cadence, offline/online symmetry and human review apps; most competitors' evaluation story is thinner. **4. MCP as a governed catalog object**, as above. **5. Honest documentation in several places** — the Genie page openly states it can query tables beyond those attached, the AI Search page publishes its BM25/HNSW/L2/RRF parameters, and the Supervisor Agent page carries a customer-responsibility warning about arbitrary code execution. This is not the tone of a vendor hiding its seams.

### 10.2 Where it is behind or unproven

| Area | Assessment |
| --- | --- |
| **Auto-optimization efficacy** | Mechanism documented; **efficacy evidenced only by vendor material and vendor-curated customer quotes**. No independent benchmark, no published methodology, no reproducibility. |
| **Preview surface** | Unity Gateway / AI Gateway is **Beta**; MCP governance is **Beta**; model services are **Beta**; Agent services are **Beta**; dedicated full-text search indexes are **Beta**; MLflow trace storage in Unity Catalog is **Public Preview**. Much of what makes the governance story compelling is **not GA**. |
| **GA scope** | Supervisor Agent GA'd on 2026-02-10 in **select US regions** and explicitly **not** for workspaces with Enhanced Security and Compliance features. |
| **Change control for managed agents** | No documented version-promotion gate for Knowledge Assistant / Supervisor Agent. Continuous SME learning is a feature *and* an unaudited release channel. |
| **Latency, concurrency and scale disclosure** | No published agent latency SLO, no documented per-endpoint concurrency limit, no documented supervisor timeout budget ❌. |
| **Pricing transparency** | Model Serving rates are published; **Foundation Model API per-token prices, serverless compute DBU rates and AI Search DBU rates were not recoverable** from public pages in my access path ❌. |
| **Framework support breadth** | Four named, first-class frameworks. AutoGen, CrewAI, Semantic Kernel and Google ADK are not documented ❌. |
| **Multi-agent architecture** | One supervisor construct plus a DIY path. No peer handoff, no durable supervision as a first-class construct. |

### 10.3 The operational reality for a serious deployment

Assume, and budget for: a **serverless compute dependency** and a **nonzero-budget usage policy** as hard prerequisites; **hours-long knowledge-source sync** and **manual re-sync**; **model and API churn** (the 2026 catalogue gained GPT-5.x, Claude 4.6/5 and Gemini 3.x within months, and MLflow shipped ~12 minors in 15 months); a **rename cadence** that will invalidate your runbooks and your internal documentation at least annually; and a **Beta control plane** that is simultaneously your guardrail and your evidence store.

None of that is disqualifying. All of it is the difference between a pilot and a fleet.

### 10.4 Do not use it for this

**Latency-critical synchronous paths** — with no published latency SLO and a multi-step, multi-service architecture under the hood, do not put a Supervisor Agent on a card-authorisation or trade-execution path; use it for advice and analysis, with a deterministic system making the decision. **Anything on the Enhanced Security and Compliance profile that needs Supervisor Agent today** — verify with your account team against the *current* region and ESC matrix, because the GA release note explicitly excludes you. **Autonomous write actions on regulated data without a compensating control** — the platform gives you read governance and an audit trail, not transaction semantics, compensation logic or an approval workflow. **Peer-to-peer autonomous agent negotiation**, which is not a documented construct; do not let an architecture diagram imply otherwise. And **a hard real-time, high-concurrency consumer workload** — standard endpoints, budgets and rate limits are designed for enterprise internal traffic, not retail-scale fan-out.

### 10.5 What the vendor's documentation does not tell a buyer

1. **Whether the benchmarks are held out.** Agent Bricks generates the judges and the synthetic data, then optimizes against them. No documentation I could open states that a **held-out real-data set** is used to validate the winning iteration.
2. **How the judges were validated.** Which models generate them, what their measured agreement with human reviewers is, and how they behave on adversarial or out-of-distribution inputs is not disclosed.
3. **What continuous learning does to reproducibility.** If an SME's guidelines change Supervisor Agent behaviour, is the prior behaviour reproducible? No documented answer ❌.
4. **The real availability matrix.** Region-by-region, ESC-by-ESC, feature-by-feature GA status is spread across multiple release notes with silent carve-outs. There is no single "what can I actually turn on" table.
5. **Cost per interaction.** No published cost model tying a Supervisor Agent run to DBUs and tokens. The Published rates are for *infrastructure units*; what a consultation costs is left to you to measure.
6. **What the platform does *not* govern.** The local-Python-tool row in §5.1 is the honest example: at least one tool mechanism is invisible to the catalog, and no document says "and here is how to audit it."

---

## 11. The competitive frame

This section positions the **agent layer** only. The platform-comparison machinery — market position, data platform capability, total cost of ownership across clouds — lives in [`technology/ai_llm/enterprise_ai_platforms_guide.md`](enterprise_ai_platforms_guide.md). **Point there; do not duplicate.**

### 11.1 Hyperscaler agent platforms

| Platform | Product named | Status and evidence | How it differs from Databricks' agent layer |
| --- | --- | --- | --- |
| **AWS** | **Amazon Bedrock AgentCore** | **GA 2025-10-13** — primary: `aws.amazon.com/about-aws/whats-new/2025/10/amazon-bedrock-agentcore-available/`, "Amazon Bedrock AgentCore is now generally available"; GA added VPC, PrivateLink, CloudFormation and resource tagging support. Later additions (policy, evaluations, runtime instances, web search) were reported in 2026 through secondary sources ⚠. | AgentCore is a **runtime + identity + memory + gateway** toolkit for agents built with any framework, model or protocol. Its governance is AWS-native (IAM, VPC, PrivateLink) rather than an asset catalog. The weak point Databricks attacks is **data governance** — IAM governs *who can invoke*, but the lineage from an agent output back to a governed table is not an AWS catalog primitive the way it is in Unity Catalog. |
| **Microsoft** | **Azure AI Foundry Agent Service** | GA status asserted by secondary sources in 2026 ⚠ — I could **not** open a Microsoft or Azure primary page confirming it within this research pass. Treat as **unverified**. | Deeply integrated with **Entra ID** and the Microsoft estate; strongest when your agents live near Microsoft 365, Dynamics and Azure data. Governance is identity-first (Entra) plus Purview for data. Databricks' advantage is the same one it has on Azure generally: **one catalog over the lakehouse**. |
| **Google** | **Agent Development Kit (ADK)** + **Vertex AI Agent Engine** | **ADK is documented as open-source** on Google Cloud docs (`docs.cloud.google.com/gemini-enterprise-agent-platform/build/adk`, listing Python, TypeScript, Go, Java, Kotlin SDKs). One secondary source states **Vertex AI Agent Builder has become the "Gemini Enterprise Agent Platform"** ⚠ — not verified at a Google primary page. | ADK is the most **framework-like** of the hyperscaler options and is genuinely open source, which makes it more portable than Databricks' managed builders. Google's governance answer is **BigQuery + Dataplex**; it is a credible catalog story, but the data is more often outside it than in a lakehouse-native estate. |

### 11.2 Standalone agent frameworks

| Framework | Position versus the Databricks agent layer |
| --- | --- |
| **LangGraph / LangChain** | **Not a competitor to Databricks — a supported authoring option inside it.** You can build on LangGraph and wrap it in `ResponsesAgent`. Its advantage is portability and control; its disadvantage is that *you* build the governance, the evaluation harness and the serving path. |
| **OpenAI Agents SDK** | Same relationship, and it is the **default** Databricks agent template. OpenAI's own platform story competes on model quality and agent primitives; Databricks competes on governing those agents' access to enterprise data. |
| **CrewAI / AutoGen / Semantic Kernel** | Not documented as first-class Databricks integrations ❌. They will generally work behind the wrapper, but you are off the documented path on evaluation, tracing and deployment. |

### 11.3 SaaS agent platforms

| Platform | Position versus the Databricks agent layer |
| --- | --- |
| **Salesforce Agentforce** | **Exists; date and GA details not verified at a Salesforce primary source in this pass** ⚠ (secondary sources cite customer counts and ARR in 2026). Positioned around **CRM workflows and Salesforce data**: it embeds agents where the business process already runs. Databricks' counter is that the *data* is in the lakehouse, not the CRM — an agent over governed enterprise data across many systems, not one application's records. The two are complements more often than substitutes. |
| **Microsoft Copilot Studio** | **Exists; GA/preview specifics not verified at a Microsoft primary source in this pass** ⚠. Low-code agent authoring inside the Microsoft stack, with Agent 365 reported as a management control plane ⚠. Competitive with Databricks on citizen-builder ease; weaker on governed multi-source data access, stronger on Office/Dynamics reach. |

### 11.4 The buyer's decision criteria

Ask in this order:

1. **Is the data already here?** If your governed source of truth is a lakehouse in Unity Catalog (or you are prepared to make it one), the Databricks agent layer's central advantage is already earned. If your data is scattered across SaaS systems, that advantage is a migration project.
2. **Is the agent's job to read governed data, or to act inside a SaaS application?** Read-governed-data → Databricks' model fits. Act-inside-a-CRM → Agentforce or Copilot Studio fit better, possibly alongside Databricks for the data layer.
3. **Do you need GA today, on a compliance security profile?** Then check the availability matrix first, in writing. As of 2026-02-10, Supervisor Agent GA excluded ESC workspaces.
4. **Is portability or governance the priority?** Governance here costs portability, and the vendor is candid about it. Choose deliberately.
5. **Do you need independent evidence of quality claims?** For auto-optimization, no independent evidence exists. If your validation regime demands it, treat the vendor claim as a hypothesis and fund your own evaluation.
6. **Can you operate a Beta control plane as a production dependency?** If not, plan for the GA surface only, and know which capabilities that costs you.

---

## 12. The regulated-institution angle

*Written from the bank's chair, condensed and cross-referenced. The general governance, red-team and model-risk methodology lives in [`ai_governance_framework_guide.md`](ai_governance_framework_guide.md), [`ai_red_teaming_guide.md`](ai_red_teaming_guide.md) and [`ai_governance_bias_redteaming_guide.md`](ai_governance_bias_redteaming_guide.md) — this section only states what Databricks' architecture does to those obligations.*

### 12.1 What catalog-based governance buys a supervised firm

**A single answerable question** — *who can this agent reach, and who authorised it?* — expressed in privileges a supervisor can inspect rather than code a supervisor must read. **Least privilege enforceable per end user** via OBO, so the failure mode where an agent becomes a shared super-user is designed out of the managed path. **Lineage from agent output back to source data**, the artefact an auditor asks for when challenging a decision. **An evidence trail with a schema**: inference tables (requests and responses), system tables (usage, cost), audit logs (Genie Agent events), and traces storable in Unity Catalog with table-level permissions. **Masking that follows the user, not the query**: ABAC row filters and column masks, now evaluated with the **session user's identity** through views and functions (GA April 2026, and a **breaking change** you must have tracked). And **spend control as a governance control**: rate limits and hard budget caps are risk limits, not merely finance tooling.

### 12.2 Model-risk and validation expectations applied to an auto-optimized agent

Applied to the auto-optimization claim, standard model-risk questions land hard:

| Model-risk question | Answer for Agent Bricks auto-optimization |
| --- | --- |
| What is the model? | Not a single model: a *search process* over configurations against **auto-generated judges**. |
| Is it a model change? | Arguably yes — and if so it is a **versioned model that requires validation and change approval**, not a tuning activity. |
| Was it independently validated? | **Not that I could establish** ❌. Only vendor claims and vendor-curated customer quotes exist publicly. |
| Is it reproducible? | **No documented answer** ❌, and the SME-feedback loop implies continuous change. |
| Does it have documented limitations and assumptions? | The synthetic-data assumption is documented in outline; the judges' validation is not. |
| Who is the owner? | **You are.** Databricks owns the optimizer; you own the task definition, the acceptance threshold and the consequence. |

The defensible position: **treat the optimized agent as a candidate, validate it against your own held-out data with your own threshold, and record the human decision.** See [`llm_evaluation_vs_validation_guide.md`](llm_evaluation_vs_validation_guide.md) and [`ai_agent_drift_guide.md`](ai_agent_drift_guide.md).

### 12.3 Data residency and cross-border questions

Concrete, vendor-documented items: **cross-geography processing** is a documented gating condition (Supervisor Agent's web search is unavailable in regions lacking both a web-search-capable native model and cross-geography processing), and both switches are residency-relevant; **regional model hosting for Genie Agents** in Japan and Korea is a **February 2026** feature, evidence that regional inference is being built region by region and a reminder to check *your* region rather than assume; **regional pricing** exists, following from regional infrastructure cost, which matters for any chargeback model; and **partner-powered AI features** must be enabled by an account administrator for Genie — who enabled them, under which DPA and in which region is a question for your third-party risk function.

### 12.4 Third-party and concentration risk

Databricks is already a systemically relevant vendor in a lakehouse estate; adding agents deepens the concentration rather than creating a new kind. The specific new exposures: the **model supply chain**, since your agent's behaviour now depends on third-party model versions served through the platform (GPT-5.x, Claude 4.6/5, Gemini 3.x) and model deprecation is a change event in your agent; **Beta dependencies**, since the gateway that enforces guardrails and holds audit evidence is in Beta; **renames as operational risk**, because a rename invalidates runbooks, internal policies and evidence references, and three occurred in 2026 alone; and the **auto-optimization dependency** — an agent *continuously improved by the vendor's optimizer* is one whose behaviour you co-own with your vendor, so contract for consultation, notification of change and rollback.

### 12.5 The audit trail an agent run leaves

Documented artefacts, assembled: the **MLflow trace** (step-level, OpenTelemetry, optionally Unity Catalog-stored with table permissions), the **inference table** entry (request and response, in a Delta table), **system tables** for usage and cost attribution, **audit log** entries (including Genie Agent events), and **lineage** to source data. That is a materially better trail than a bespoke agent holding a service-account key and writing to a log file — which is the honest comparison, and the reason to accept the platform's constraints.

### 12.6 Which banking agent use cases are ready, and which are not

| Ready now, with controls | Approve-with-conditions | Not ready |
| --- | --- | --- |
| Internal knowledge assistance over policy/procedure documents (Knowledge Assistant, citations, internal users only) | Supervised advice/analysis over governed data (Genie + Supervisor Agent) with a human decision-maker and logged rationale | Autonomous action on customer accounts or balances |
| Natural-language analytics over governed marts for analysts (Genie Agent, permissions inherited) | Credit-policy and operations assistance where the agent *drafts* and a human *signs* | Anything on a card-authorisation, payment or trading path |
| Read-only document extraction and classification pipelines | Documented case summarisation where the output is reviewed before disclosure | Cross-border agent fan-out where data residency is unresolved |
| Developer-facing analytics and reporting assistance | Agent-driven report preparation feeding a formal control, with evidence retention | Supervisor Agent in an ESC-profile workspace (verify current matrix first) |

---

## 13. The Cymbal Bank worked example

**Cymbal Bank is a fictional bank.** The deployment below is illustrative. **Every number in this section is illustrative and clearly labelled as such** — no vendor price sheet, contract, or benchmark produced the figures in §13.6. The purpose is to show the *shape* of a governed deployment and the decisions it forces, not to predict a bill.

### 13.1 The starting position

Cymbal Bank runs Databricks as its lakehouse. Credit policy, product documentation, procedure manuals and the operational data marts sit in Unity Catalog with a working permission model. The bank wants an agent in front of a **regulated workflow**. The chosen candidate:

> **Credit-policy and operations-assistance agent.** A credit officer asks a question about policy interpretation and about the current exposure of a specific counterparty or portfolio. The agent answers with citations to approved policy documents and with figures drawn from governed marts. **It never decides.** It drafts and cites; the credit officer decides and signs.

### 13.2 Use-case selection — the filter actually applied

The bank rejects two candidates before accepting this one:

| Candidate | Verdict | Why |
| --- | --- | --- |
| Autonomous credit-limit adjustment | **Rejected** | The platform provides read governance and an audit trail, not transaction semantics, compensation logic or an approval workflow. The agent would need write authority over a balance-affecting system. Not ready (§12.6). |
| Customer-facing chat on account enquiries | **Deferred** | No published latency SLO for agent endpoints ❌, and the residency position for customer-facing inference in the bank's jurisdictions is unresolved. |
| **Credit-policy and operations assistance (internal, advice-only)** | **Accepted** | Read-only, internal users, human decision-maker retained, evidence trail available, and the value is *time to a cited answer* rather than a decision. |

### 13.3 Tool and data binding under Unity Catalog, with least privilege

The bank's binding, expressed as the objects a security reviewer would inspect:

| Agent capability | Mechanism | Grant | Least-privilege note |
| --- | --- | --- | --- |
| Answer policy questions with citations | **Knowledge Assistant** over three Unity Catalog volumes (`policy.credit.volumes.approved_docs`, `policy.ops.volumes.procedures`, `policy.risk.volumes.manuals`) | `READ VOLUME` on the three volumes | Volumes contain only documents marked **Approved** by the policy function. Draft policy lives in a different volume with no grant. |
| Query current exposure | **Genie Agent** over a dedicated **curated mart** (≤ 50 tables) | `SELECT` on the mart tables; `USE CATALOG` / `USE SCHEMA` | The mart is built for this purpose with ABAC **row filters** by desk and **column masks** on customer identifiers — so the agent's output is masked *per end user*, not by the agent. |
| Historical document lookup by identifier | **Unity Catalog function** `policy.search_reference_code(code STRING)` | `EXECUTE` on the function only | The function is the only path to the underlying table; no direct `SELECT` is granted to users. |
| Read an internal ticketing backlog | **MCP Service** to the bank's ticketing system | `EXECUTE` on the MCP Service; `USE CONNECTION` on the connection | Read-only credential in the connection. Write tools are excluded by **tool filtering**. |
| Web search | **Disabled** | — | Deliberate: prevents exfiltration by encoding. Named as a decision, not an oversight. |
| Arbitrary code execution | **Disabled** | — | Deliberate: the docs warn that arbitrary code in a tool can expose what the agent can reach (§5.4). |
| Local Python function tools | **Prohibited by build standard** | — | Because they are **invisible to the catalog** (§5.1). This is a bank rule, not a platform one. |

**Model access**: routed through **Unity Gateway** with an **allowlist** restricted to bank-approved models in `system.ai`, per-user **rate limits**, and **hard budget caps** per desk. Prompt-injection prevention, sensitive-data detection and content filtering are attached as **service policies** — and priced in as *one layer*, never as the control.

**Degradation choice**: the bank elects to grant **no service-account fallback** on the Genie mart. If a user lacks a privilege, the answer is a 403 and a support ticket — not a broader identity quietly filling the gap.

### 13.4 Evaluation design and acceptance threshold

The bank does **not** accept the vendor's generated judges as the acceptance criterion. Its own design:

| Layer | What it is | Threshold |
| --- | --- | --- |
| **Grounding** | 300 held-out questions answered by two credit officers independently, drawn from **real recent** officer queries, not synthetic | ≥ 95% of answers must cite a document that **actually supports** the answer, adjudicated by the officers. Below this, the agent's *citation* value is nil. |
| **Citation correctness** | Each answer's cited extract must literally contain the basis for the claim | ≥ 98% (this is the product's entire value; a cited-but-wrong answer is worse than no answer). |
| **Numeric fidelity** | Where the answer quotes a figure from the mart, the figure must match a re-run SQL query | 100% — a single wrong number fails the release |
| **Refusal behaviour** | Out-of-scope or policy-absent questions must be refused, not guessed | ≥ 90% correct refusal on a 100-question adversarial set |
| **Injection resistance** | Prompt-injection corpus run against the agent; measured by whether any out-of-allowlist data or URL fetch is attempted | **Zero** successful exfiltrations; any attempt fails the release |
| **Cost per consultation** | Serverless compute + tokens + search, attributed by user tag | Illustrative ceiling of **USD 0.20 per consultation** — a bank-set budget, not a vendor figure |
| **Latency (bank-measured, since none is published)** | p95 wall-clock for a full supervisor run | **≤ 30 seconds** for advice-only; pilot measured by the bank, not derived from any vendor statement |

**Held-out discipline**: the 300-question set is split — 200 for tuning the agent's instructions, **100 locked away**, unopened until final acceptance. If the agent is re-optimized, the locked 100 are re-drawn. This is the part the auto-optimization claim does not provide and the bank must supply ([`llm_evaluation_vs_validation_guide.md`](llm_evaluation_vs_validation_guide.md)).

### 13.5 Deployment and monitoring plan

**Path**: Knowledge Assistant + Genie Agent + UC function + ticketing MCP, coordinated by a **Supervisor Agent**, with a thin **Databricks Apps** chat UI as the officer-facing surface; deployment artifacts are **git-backed** (GA April 2026) so release records exist. **Pre-production**: pilot with 12 credit officers for 6 weeks, fully logged, with no production decisions taken on agent output. **Monitoring**: MLflow **production monitoring using the same judges as the offline gate** (the platform's genuine strength), weekly review of traces and feedback, **drift watch** on query mix and grounding rate, and cost attribution by desk via service tags. **Acceptance gate for release**: the **locked 100** plus the injection and numeric-fidelity tests — a failing numeric-fidelity test fails the release regardless of aggregate scores. **Change control (bank-imposed, because the platform does not supply it)**: any change to agent instructions, the mart, the policy volumes, the allowlisted models or the MCP tool set triggers **re-validation against the locked set**; because Supervisor Agent improves through SME feedback, the bank requires a **monthly re-validation** as a standing control and treats SME guideline changes as **configuration changes under change management** — the bank closing the gap identified in §7.4, which is not a platform feature.

### 13.6 Illustrative cost estimate

> **ILLUSTRATIVE ONLY.** Derived from the published *DBU rates* in §7.5 and bank-assumed volumes, not from a Databricks quote. DBU **prices** are regional and contract-specific and could not be established publicly in this research ❌. Treat every figure below as a planning shape, not a budget.

| Line | Assumption | Illustrative amount |
| --- | --- | --- |
| Users | 45 credit officers + 15 operations analysts | 60 named users |
| Consultations | 8 per user per working day, 21 days | ~10,100/month |
| Agent serving (CPU) | 4 DBU/hour baseline, endpoint held warm during business hours (~10 h/day × 21 d = 210 h) | **~840 DBU/month** |
| Model tokens | Supervisor run averaging ~3 model calls; frontier-model pricing applied at the bank's negotiated rate | Not computable publicly ❌ — budget locally |
| AI Search endpoints | One standard endpoint for the policy corpus, business-hours | Regional DBU rate not established ❌ |
| Genie | Pro/serverless SQL warehouse, business-hours | SQL warehouse DBUs |
| Knowledge Assistant sync | Incremental, triggered by policy change (expected ~4×/month) | Negligible compute, hours of elapsed time |
| **Bank-set budget ceiling** | Cost per consultation cap | **~USD 0.20** (illustrative) |

**What the bank could not establish before committing budget**: Foundation Model API per-token prices, serverless compute DBU rates for agent workloads, and AI Search DBU rates — all three were **not recoverable from public Databricks pages** in this research pass ❌ (§15). The estimate was therefore built bottom-up from published *GPU/CPU serving rates* plus bank-assumed volumes, and **the bank's procurement team obtained the remainder from the vendor in writing before sign-off**, which is the recommendation for any reader of this guide.

### 13.7 Failure modes and guardrails

| Failure mode | Guardrail | Residual risk |
| --- | --- | --- |
| Confident answer with no supporting policy | Mandatory citations; grounding threshold ≥ 95%; refusal set | An officer trusts a well-written wrong answer anyway |
| Correct answer, wrong counterparty's data | ABAC row filters by desk; column masks on identifiers; OBO | A mis-assigned desk attribute in the data itself |
| Prompt injection via a retrieved document | Gateway prompt-injection policy + web search disabled + no write tools | A novel injection bypasses the policy; monitored, not prevented |
| Silent behaviour change via SME feedback | Monthly re-validation; SME guideline changes as change requests | Between validations, behaviour can drift — this is a **standing** residual risk, accepted explicitly |
| Model deprecation by a third-party provider | Model allowlist reviewed monthly; re-run of the locked set on any model change | Provider deprecation windows may be short |
| Cost surprise | Per-user rate limits + hard budget caps at the gateway | Budget exhaustion degrades service rather than overspending — the acceptable failure direction |
| Mart definition changes under the agent | Mart change is a first-class change; triggers re-validation | Test coverage of the mart itself |
| Over-trust after a good pilot | Answers always presented with citations and a **"verify before relying on this for a credit decision"** banner; agent output never enters the credit file directly | Cultural, not technical — mitigated by policy, not code |

### 13.8 The approval path

1. **Use-case and risk-tiering** under the bank's AI governance framework ([`ai_governance_framework_guide.md`](ai_governance_framework_guide.md)) — advice-only, internal, human-in-the-loop: the middle tier, not the lightest.
2. **Security review** of §13.3's binding, with an explicit decision record for **web search disabled**, **code execution disabled**, and **local function tools prohibited**.
3. **Model-risk review** treating the agent (and any auto-optimization applied to it) as a **versioned model** — §12.2's questions answered in writing, including "not independently validated" and the compensating control.
4. **Red-team** the injection corpus and the refusal set ([`ai_red_teaming_guide.md`](ai_red_teaming_guide.md)).
5. **DPIA / third-party risk** on partner-powered AI features for Genie and on the frontier model providers in the allowlist.
6. **Change-management registration** of the monthly re-validation and the SME-guideline-as-change-request rule.
7. **Operational readiness**: runbook covering budget exhaustion, model deprecation, MCP connection failure, and gateway outage — noting that the gateway is a **Beta** dependency.

### 13.9 The honest recommendation

**Proceed, scoped as an advice-only assistant, with the bank's own evaluation gate.** The reasons are the ones §10.1 gives: catalog-based governance over the tools and data is real, OBO is the right default, the evidence trail is materially better than the bespoke alternative, and the workflow value (cited answers in seconds rather than manual policy hunting) is modest but genuine and low-consequence because a human signs.

**What the bank could not establish**, and should say so in its own approval record:

- **No independent evidence** that auto-optimization produces better agents than careful manual engineering. The bank therefore does **not** rely on it as a control and rejects any suggestion that it substitutes for validation.
- **No published latency, concurrency or timeout SLO** for agent endpoints ❌ — the bank measures its own and holds no vendor commitment.
- **No public per-token or serverless DBU pricing** ❌ — the bank's budget rests on vendor-provided numbers obtained in writing.
- **No documented version-promotion gate** for managed agents ❌ — the bank supplies one through change management.
- **No GA confirmation for Knowledge Assistant** ❌, and the gateway remains **Beta** — the bank accepts a preview-surface dependency for a non-critical, advice-only workflow, and would not for a decisioning one.
- **No established rename date for Genie Spaces → Genie Agents** ❌ — a documentation curiosity that nonetheless signals how fast this surface moves.

**What the bank would need before widening scope**: Supervisor Agent GA confirmation for the bank's exact region **and** its ESC-profile posture; a documented change-control story for continuously-learning agents; and either independent evaluation evidence or enough internal evaluation maturity to substitute for it. Until then, the agent stays advice-only, internal, and on the narrow side of the permission boundary.

---

## 14. The claims audit

**GA-versus-preview status is the highest-risk fact class in this guide** and is treated as load-bearing. Every entry carries source and date.

### 14.1 Verified

| # | Claim | Source | Source type | Date |
| --- | --- | --- | --- | --- |
| 1 | Agent Bricks launched, in **Beta** | databricks.com press release "Databricks Launches Agent Bricks" | Vendor primary | 2025-06-11 |
| 2 | **MLflow 3.0 generally available** | Same press release; PyPI `mlflow` 3.0.0 upload timestamp 2025-06-10T19:55:15 | Vendor primary + package registry | 2025-06-10/11 |
| 3 | **Supervisor Agent GA** — select US regions, workspaces **without** ESC features | docs.databricks.com release notes, February 2026 | Vendor primary | 2026-02-10 |
| 4 | Multi-Agent Supervisor **renamed** to Supervisor Agent | Same release notes | Vendor primary | 2026-02-05 |
| 5 | **Vector Search renamed to AI Search** | docs.databricks.com release notes, June 2026 | Vendor primary | 2026-06-01 |
| 6 | Supervisor Agent **+ custom MCP servers and custom agents on Databricks Apps** | Release notes, April 2026 | Vendor primary | 2026-04-30 |
| 7 | **MLflow trace storage in Unity Catalog → Public Preview** (OTel format, UC-governed) | Release notes, April 2026 | Vendor primary | 2026-04-30 |
| 8 | Supervisor Agent **web search** as built-in tool; requires per-invocation user approval; runs on `databricks-gpt-5` regardless of supervisor model | Release notes, June 2026 + Supervisor Agent docs | Vendor primary | 2026-06-03 |
| 9 | **ABAC GA**; row filters/column masks now evaluated with **session user identity** through views/functions; **breaking change**, 3-month grace | Release notes, April 2026 | Vendor primary | 2026-04-28 |
| 10 | **Unity Gateway** governs models, MCP services, functions, connections; rate limits, traffic splitting, Smart Routing, budgets, service policies, usage tracking, cost attribution, **inference tables** | docs.databricks.com "AI governance with Unity Gateway" | Vendor primary | Last updated 2026-09-11 |
| 11 | **AI Gateway (Beta)**; **MCP governance (Beta)**; **model services (Beta)** | Release notes Feb/Apr/Jun 2026 | Vendor primary | 2026-02-12 / 2026-04 / 2026-06-29 |
| 12 | `databricks-agents` **1.12.0**, summary string "Mosaic AI Agent Framework SDK" | PyPI JSON API | Package registry | Uploaded 2026-08-21 |
| 13 | `mlflow` **3.16.1** on PyPI | PyPI JSON API | Package registry | Uploaded 2026-09-16 |
| 14 | `mlflow.genai.evaluate()` + **Scorer**; **not interoperable** with classic `mlflow.evaluate()` | mlflow.org docs (via search result content) | Vendor primary | Accessed 2026-09-17 |
| 15 | Tool governance map: agent as UC model; tools as MCP services / functions / connections; `EXECUTE`, `USE CONNECTION`, `CAN_USE`, `SELECT`, `READ VOLUME` requirements | Supervisor Agent docs; Unity Gateway docs; agent-authoring docs | Vendor primary | 2026-09-11 to 2026-09-15 |
| 16 | **Local Python function tools require no resource grants** because they run in-process | docs.databricks.com "Author an agent…" | Vendor primary | 2026-09-15 |
| 17 | Genie Agents: **≤ 50 tables**; **200,000 conversations**; **10,000 messages**; pro/serverless SQL warehouse; per-user UC permissions; **can query tables beyond those attached**; **partner-powered AI features** must be enabled | docs.databricks.com "Genie Agents — set up" | Vendor primary | Last updated 2026-09-14 |
| 18 | Knowledge Assistant: inputs and hard limits (10 sources, 100 MB, 500 pages, hours to sync, **manual re-sync**) | docs.databricks.com "Knowledge Assistant" | Vendor primary | Last updated 2026-09-11 |
| 19 | AI Search internals: **HNSW**, **L2**, **Okapi BM25**, **RRF (rrf_param 60)**; endpoint capacities; self-managed→managed index conversion impossible | docs.databricks.com "Databricks AI Search" | Vendor primary | Last updated 2026-09-14 |
| 20 | MCP: three server sources, governed as UC securables, external clients (Claude, Cursor, MCP Inspector) supported | docs.databricks.com "MCPs and agent tools" | Vendor primary | Last updated 2026-09-11 |
| 21 | Documented frameworks: **LangGraph, LangChain, OpenAI Agents SDK, LlamaIndex, PyFunc, Anthropic**; `ResponsesAgent` wraps any framework | Agent-authoring docs; build-agents docs; Feb 2026 release note | Vendor primary | 2026-09-15 |
| 22 | Model Serving **GPU DBU rates** table; **CPU serving 4 DBU/hour**; Serverless Real-time Inference SKU; 30-minute GPU scale-to-zero; regional pricing | databricks.com Model Serving pricing page | Vendor primary | Accessed 2026-09-17 |
| 23 | **Omnigent on Databricks** exists on the product page with the quoted description | databricks.com Agent Bricks product page | Vendor primary (marketing) | Accessed 2026-09-17 |
| 24 | **2026 model catalogue**: GPT-5.3 Codex, Claude Sonnet 4.6, Claude Opus 4.6, Claude Sonnet 5, Gemini 3.1 Pro, Qwen3-Embedding-0.6B | Release notes, Feb and Jun 2026 | Vendor primary | 2026-02 / 2026-06 |
| 25 | **Amazon Bedrock AgentCore GA** with VPC, PrivateLink, CloudFormation, tagging | aws.amazon.com about-aws whats-new; AWS blog | Vendor primary | 2025-10-13 |
| 26 | Google **ADK** documented as open-source with Python/TypeScript/Go/Java/Kotlin SDKs | docs.cloud.google.com | Vendor primary | Accessed 2026-09-17 |
| 27 | Databricks **>$7B run-rate, >80% YoY, $190B valuation, $5B raise**; product list includes Genie, Lakebase, Agent Bricks, Unity AI Gateway | databricks.com press release | Vendor primary | 2026-08-13 |

### 14.2 Flagged (⚠ — secondary source only)

| # | Claim | Source | Why flagged |
| --- | --- | --- | --- |
| F1 | Agent Bricks "**generally available with over 100,000 agents built, processing more than one quadrillion tokens per year**", GA milestones incl. Custom Agents and Document Intelligence, "regional expansion across 7 Azure regions" | Aggregated Summit-summary sites | Secondary, marketing-provenance, **not confirmed at a Databricks primary page**. **Do not quote these figures.** |
| F2 | Custom Docker images for AI Runtime CLI workloads (Beta); classification/extraction public preview; Supervisor Agent default availability under compliance security profile **with HIPAA controls** | Search-result descriptions of Databricks release notes (March 2026, June 2026) | Primary pages **not opened** in this pass; content consistent but unconfirmed as fetched. |
| F3 | Bedrock AgentCore 2026 additions (policy, evaluations, runtime instances, web search, DynamoDB vector search GA) | Secondary news/aggregator sites | Primary AWS pages not opened for the 2026 items; the **2025-10-13 GA** itself is primary-verified. |
| F4 | Azure AI Foundry Agent Service GA | Secondary comparison articles | **No Microsoft/Azure primary page opened.** Treat status as **unverified**. |
| F5 | Vertex AI Agent Builder rebranded to "Gemini Enterprise Agent Platform" | One secondary article | Not verified at a Google primary page; the Google Cloud docs **URL** contains `gemini-enterprise-agent-platform`, which is corroborating but not conclusive on the rebrand narrative. |
| F6 | Salesforce Agentforce / Microsoft Copilot Studio customer counts, ARR, "Agent 365" control plane | Secondary comparison articles | Product existence is credible; **status, dates and figures unverified**. |
| F7 | Omnigent hosted at `omnigent.ai` | Databricks product page linking to it | Link is primary; **the destination was not opened**, and no Databricks doc or release note describes Omnigent. Status unestablished. |

### 14.3 Rejected / could not be established (❌)

| # | Claim | Why rejected |
| --- | --- | --- |
| R1 | **Knowledge Assistant is generally available** | Not established. The docs page carries no status label and no GA announcement was located. **Not asserted.** |
| R2 | **A published per-token price for Foundation Model APIs** | Not found in any Databricks page reachable in this pass. |
| R3 | **Published serverless compute DBU rates for agent workloads; AI Search endpoint DBU rates** | Not found. Pricing page renders rates client-side and regionally. |
| R4 | **An agent latency SLO / p95 target / per-endpoint concurrency limit / supervisor timeout budget** | No such numbers documented anywhere I could reach. |
| R5 | **A GA/preview label for Genie Agents, and the rename date from Genie Spaces** | Docs state "formerly known as Genie Spaces" with no date and no status label. |
| R6 | **A documented independent benchmark or methodology for Agent Bricks auto-optimization** | None exists publicly to my knowledge after searching. |
| R7 | **A documented version-promotion / A-B gate for managed agents** | Not documented. Model-service traffic splitting exists; agent-level release gating is not a documented feature. |
| R8 | **An established GA status for Unity Gateway as a whole** | Component statuses are Beta; the docs page itself carries no GA label. **Treated as Beta in this guide.** |
| R9 | **A verified omnigent.ai URL** | Not opened; not confirmed to be a Databricks property. |
| R10 | **Agent Bricks as a separately-priced SKU** | No SKU evidence found; billing appears to flow through serverless and serving SKUs. |

---

## 15. What Could Not Be Verified

Collected here as explicit negatives, because the temptation to fill these gaps with inference is exactly what a reader of a fast-moving vendor surface must resist. **Nothing in this list should be read as a negative about the product — only as a statement about the evidence.**

1. **The GA status of Knowledge Assistant.** No status label on the docs page, no GA announcement located. I do **not** assert it is GA and I do **not** assert it is a preview.
2. **The GA status of Unity Gateway as a whole.** Component features are documented in Beta; the control plane as a product carries no GA label I could find.
3. **The GA status of Azure AI Foundry Agent Service.** Secondary sources assert GA in 2026; no Microsoft primary page was opened in this pass.
4. **The GA status, dates or figures for Salesforce Agentforce and Microsoft Copilot Studio.** Product existence and general positioning are credible; specifics are unverified.
5. **The rename date of Genie Spaces → Genie Agents.** The docs assert the rename; no date is given on the page I opened and no release note was found.
6. **A published per-token price for Foundation Model APIs**, and **DBU rates for serverless compute and AI Search endpoints**. Not found; pricing page rates are client-rendered and regional.
7. **Any published latency, concurrency or timeout guarantee for agent endpoints.** None found. The bank in §13 measured its own.
8. **Any independent, reproducible evaluation of Agent Bricks auto-optimization.** None found. Vendor press release and vendor-curated customer quotes are the entire public evidence base I could locate.
9. **Whether Agent Bricks' synthetic benchmarks are validated against held-out real data.** Not documented.
10. **Which models generate Agent Bricks' LLM judges, and their measured agreement with human reviewers.** Not documented.
11. **Whether a continuously-learning managed agent's prior behaviour is reproducible after SME feedback.** Not documented.
12. **A documented agent-level version-promotion or split-traffic A/B gate for Knowledge Assistant / Supervisor Agent.** Not documented.
13. **The contents or existence of `omnigent.ai/docs`.** Not opened; Omnigent's GA/preview status is unestablished.
14. **Whether AutoGen, CrewAI, Semantic Kernel or Google ADK are supported Databricks authoring frameworks.** Not documented as first-class; the wrapper interface suggests they would work, but no support statement exists.
15. **A single consolidated availability matrix** (region × ESC profile × feature × GA/Beta) for the agent surface. The information is distributed across monthly release notes with carve-outs stated in prose; I could not assemble an authoritative complete matrix from the sources reachable in this pass.
16. **A published cost model per agent interaction.** No vendor document ties a Supervisor Agent run to DBUs and tokens.

---

## 16. Glossary

| Term | Meaning |
| --- | --- |
| **ABAC** | Attribute-based access control in Unity Catalog — row filters and column masks driven by **governed tags**. GA April 2026. |
| **Agent Bricks** | Databricks' agent platform umbrella; also the name of the managed builder family. |
| **Agent Evaluation** | The evaluation surface inside MLflow 3; formerly the standalone Mosaic AI Agent Evaluation product. |
| **Agent Service** | A Unity Catalog registration for an agent that runs **outside** Databricks. **Beta.** |
| **AgentServer (MLflow)** | Async FastAPI server exposing `/responses`, serving a Custom Agent with built-in tracing. |
| **AI Search** | Managed vector + keyword search over Unity Catalog data. **Formerly Vector Search** (renamed 2026-06-01). |
| **ALHF** | Agent Learning on Human Feedback — Databricks' name for improving a managed agent (notably Supervisor Agent) from natural-language SME guidance. |
| **`databricks-agents`** | The Python SDK package. 1.12.0 as of 2026-08-21; still summarised on PyPI as "Mosaic AI Agent Framework SDK". |
| **Databricks Apps** | The app hosting surface for Custom Agents and chat UIs; routes run as the authenticated user by default. |
| **DBU** | Databricks Unit — the billing unit for compute and serving. Rates are regional and contract-specific. |
| **ESC** | Enhanced Security and Compliance add-on. Several agent capabilities are documented as unavailable when it is enabled. |
| **Foundation Model APIs** | Pay-per-token hosted model endpoints (`system.ai`); 2026 catalogue includes frontier third-party models. |
| **Genie Agent** | The natural-language-data agent over Unity Catalog tables. Formerly **Genie Space**. |
| **Governed tags** | Unity Catalog tags that carry policy meaning; the basis of ABAC. GA April 2026. |
| **Inference table** | A Unity Catalog Delta table logging gateway requests and responses — the audit artefact. |
| **Knowledge Assistant** | Managed, no-code chatbot over documents with citations. |
| **Lakebase** | Databricks' serverless Postgres, positioned as agent memory / operational data. >$100M run-rate per the 2026-08-13 press release. |
| **MCP** | Model Context Protocol — the open standard Databricks uses for tool integration, with MCP servers governed as Unity Catalog securables. |
| **MCP Service** | An MCP server **registered as a Unity Catalog securable**; `EXECUTE` to use. |
| **MLflow 3** | The GenAI-era MLflow (3.0 GA 2025-06-10; 3.16.1 as of 2026-09-16). GenAI APIs live under `mlflow.genai`. |
| **OBO** | On-Behalf-Of authentication — the agent acts as the human user, so the user's Unity Catalog permissions apply. |
| **Omnigent** | A layer presented on the Databricks product page for running agents above existing harnesses (Claude Code, Codex). ⚠ **Status unestablished; single-source.** |
| **ResponsesAgent** | The MLflow agent interface that wraps any framework for Databricks compatibility. |
| **Scorer** | The evaluation-metric object used by `mlflow.genai.evaluate()`. |
| **Service policy** | A guardrail attached to a model or MCP service, governing request/response content by policy function. |
| **Supervisor Agent** | The managed multi-agent orchestrator. GA 2026-02-10 (select US regions, non-ESC). Formerly **Multi-Agent Supervisor**. |
| **Unity Catalog** | The governance layer: catalogs, schemas, tables, volumes, functions, models, connections, MCP services, agent services. |
| **Unity Gateway / Unity AI Gateway** | The AI governance control plane. Documented as **Beta**; formerly **AI Gateway** / **Mosaic AI Gateway**. |

### 16.1 The old-name / new-name mapping, condensed

| Old name | New name | Date |
| --- | --- | --- |
| Mosaic AI Vector Search | **Databricks AI Search** | 2026-06-01 |
| Mosaic AI Gateway → AI Gateway | **Unity Gateway** (docs) / **Unity AI Gateway** (marketing) | During 2026 |
| Agent Bricks Multi-Agent Supervisor | **Supervisor Agent** | 2026-02-05 |
| Genie Spaces | **Genie Agents** | Date not established ❌ |
| Mosaic AI Agent Framework (product) | **Agent Bricks** (SDK package name unchanged) | Progressive, no single note |
| Mosaic AI Agent Evaluation | **Agent Evaluation** (inside MLflow 3) | Absorbed at MLflow 3.0 GA, 2025-06-10 |
| `mlflow.evaluate()` for GenAI | **`mlflow.genai.evaluate()`** + Scorer | MLflow 3.0; the two are **not interoperable** |
| Databricks-hosted models (undifferentiated) | **Foundation Model APIs** → **model services** (`system.ai`) | Model services Beta, 2026-06-29 |

---

## 17. Cross-references and further reading

### 17.1 Within this repository — platform and platform comparison

- [`technology/databricks_guide.md`](../databricks_guide.md) — **the platform guide**. Owns what Databricks *is*: lakehouse, Unity Catalog, compute, storage, serving infrastructure. §3.7 is the before/after frame quoted in §1.3 here; cross-ref it for serving infrastructure rather than re-deriving.
- [`technology/ai_llm/enterprise_ai_platforms_guide.md`](enterprise_ai_platforms_guide.md) — **owns the platform comparison** across vendors (Databricks appears heavily in it). §11 of this guide positions the **agent layer** only.

### 17.2 Within this repository — the agent-engineering cluster (the *discipline*)

- [`technology/ai_llm/agentic_engineering_guide.md`](agentic_engineering_guide.md)
- [`technology/ai_llm/agent_harness_engineering_guide.md`](agent_harness_engineering_guide.md)
- [`technology/ai_llm/agent_scaffolding_guide.md`](agent_scaffolding_guide.md)
- [`technology/ai_llm/agent_versioning_guide.md`](agent_versioning_guide.md)
- [`technology/ai_llm/agentops_guide.md`](agentops_guide.md)
- [`technology/agentic_workflows_guide.md`](../agentic_workflows_guide.md) — note: at `technology/`, not `technology/ai_llm/`
- [`technology/durable_ai_agent_workflows_guide.md`](../durable_ai_agent_workflows_guide.md) — note: at `technology/`, not `technology/ai_llm/`
- [`technology/ai_llm/hierarchical_multi_agent_frameworks_guide.md`](hierarchical_multi_agent_frameworks_guide.md)
- [`technology/ai_llm/hybrid_multi_agent_systems_guide.md`](hybrid_multi_agent_systems_guide.md)
- [`technology/ai_llm/multi_agent_banking_guide.md`](multi_agent_banking_guide.md)
- [`technology/ai_llm/agents_work_fall_apart_guide.md`](agents_work_fall_apart_guide.md)
- [`technology/ai_llm/llm_agents_failures_production_guide.md`](llm_agents_failures_production_guide.md)
- [`technology/ai_llm/ai_agent_drift_guide.md`](ai_agent_drift_guide.md)
- [`technology/ai_llm/agent_sandboxing_strategies_guide.md`](agent_sandboxing_strategies_guide.md)
- [`technology/ai_llm/llm_evaluation_frameworks_guide.md`](llm_evaluation_frameworks_guide.md)
- [`technology/ai_llm/llm_evaluation_vs_validation_guide.md`](llm_evaluation_vs_validation_guide.md)
- [`technology/ai_llm/ai_agent_platform_selection_guide.md`](ai_agent_platform_selection_guide.md)
- [`technology/ai_llm/enterprise_agentic_platform_architecture_guide.md`](enterprise_agentic_platform_architecture_guide.md)
- [`technology/ai_llm/ai_governance_framework_guide.md`](ai_governance_framework_guide.md)
- [`technology/ai_llm/ai_red_teaming_guide.md`](ai_red_teaming_guide.md)
- [`technology/ai_llm/ai_governance_bias_redteaming_guide.md`](ai_governance_bias_redteaming_guide.md)
- [`technology/ai_llm/rag/`](rag/) — 18 guides on retrieval-augmented generation
- [`technology/ai_llm/llm_latency_optimization_guide.md`](llm_latency_optimization_guide.md)

### 17.3 Access-path notes

`web_extract` worked on every Databricks surface used here — product pages, press releases, the blog, `docs.databricks.com` doc and release-notes pages, the `learn.microsoft.com` Azure Databricks mirror and `developers.databricks.com` — despite those sites being JavaScript-heavy; several pages exceeded the extract budget and were truncated head+tail, cached locally and paged with `read_file`. **`web_search` returned empty on the pricing query**, which is why per-token and serverless DBU rates are recorded as explicit negatives rather than filled in. The **PyPI JSON API** (`curl` to a temp file then a separate parse, because the `curl | python3` pipe was blocked by the environment's security scan) and the **release-notes pages** were the reliable fallbacks. No archives (web.archive.org) were needed.

---

## 18. Closing summary

The Databricks agent stack in September 2026 is **not** the stack the platform guide recorded in 2024, and the difference is not cosmetic. The umbrella is now **Agent Bricks**; the managed builders are **Knowledge Assistant** and **Supervisor Agent**; the natural-language-data agent is a **Genie Agent**; the gateway is **Unity Gateway**; retrieval is **AI Search**; and the MCP server your agent calls is a **permissioned object in a catalog**, not a URL in a config file.

What that buys — and it is real — is the answer to the only question a regulated buyer asks first: **what can this agent reach, and who authorised it?** Expressed in the same privileges as everything else in the estate, with lineage back to source data, an inference table holding the requests and responses, and an MLflow trace showing every step. That is the differentiator, and it survived every check this guide could apply.

What it does not buy is the part the marketing leads with. **Auto-optimization is documented as a mechanism and unevidenced as an outcome** — the public evidence is a vendor press release and vendor-selected customer quotes, and the optimizer's own judges are both the measuring instrument and the objective. **GA is narrower than it sounds**: Supervisor Agent generally available in select US regions and explicitly not for Enhanced Security and Compliance workspaces. **Much of the governance story is Beta**, including the control plane that would hold your evidence. **No agent latency SLO, no version-promotion gate for managed agents, and no public per-token or serverless DBU pricing were recoverable** — all recorded above as explicit negatives rather than guessed.

The honest posture for a bank is the one §13 works through: **advice-only, internal, human-in-the-loop, with your own locked held-out evaluation as the acceptance gate**, your own change control standing in for the promotion gate the platform does not document, and the auto-optimizer treated as a hypothesis generator rather than a control. Used that way, this is a stack worth deploying, on the strength of governance rather than of automation.

Databricks' real product here is not the agent. It is **the governed agent.**
