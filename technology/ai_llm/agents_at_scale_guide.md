# Agents at Scale: A Fleet Is Not One Agent Multiplied

> **Author:** Jack Liu Shurui
> **Role:** Solution Architect
> **Repository:** github.com/jackliusr/research
> **Series:** AI/LLM Engineering Guides — Agent & Platform Architecture track
> **Date:** 22 September 2026
>
> **Companion guides.** This guide owns exactly four things: the **concurrency model** for many simultaneous agent sessions, the **serving layer** they meet, the **economics** of running them, and the **failure modes that exist only at volume**. It defers and does not re-derive: scaling **requirements and targets** → [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) §5 (NFRs); **observability, evaluation, monitoring, tooling** → [agentops_guide.md](agentops_guide.md) §3–§7; **production readiness and cost *controls*** (token budgets, caching, model routing) → [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) §3–§7; **multi-backend and model routing** → [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md) §6–§7; **agent topology and hierarchy** → [hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md); **durable execution, idempotency, sagas, compensation** → [durable_ai_agent_workflows_guide.md](../durable_ai_agent_workflows_guide.md); the **general failure taxonomy and compounding-error arithmetic** → [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) and [agents_work_fall_apart_guide.md](agents_work_fall_apart_guide.md); **reproducibility and deterministic simulation testing** → [deterministic_engineering_guide.md](../deterministic_engineering_guide.md) §5.
>
> **How to read it.** §1–§2 give the thesis and the shape change with no mechanism. §3–§5 are the machinery: concurrency, serving, quota. §6–§7 are the money. §8 is the section this guide exists for — the failure modes that only exist at volume. §9–§13 are the operational, organisational and regulated-enterprise context. §14 is a fictional worked example; §15 is the anti-pattern catalogue and the claims audit; §16 says what could not be verified. Every figure is labelled. `VERIFIED` means I read it on the primary source on the date shown. `ILLUSTRATIVE` means it is a constructed arithmetic input, not a published fact. `⚠` marks an uncertain claim; `❌` marks one I could not verify at all.

---

### Table of Contents

1. [The Overview, the Boundary and the Decoder](#1-the-overview-the-boundary-and-the-decoder)
2. [What Actually Changes at Scale](#2-what-actually-changes-at-scale)
3. [The Concurrency Model](#3-the-concurrency-model)
4. [The Serving Layer](#4-the-serving-layer)
5. [Rate Limits, Quotas and Backpressure](#5-rate-limits-quotas-and-backpressure)
6. [The Economics](#6-the-economics)
7. [Where the Money Actually Goes](#7-where-the-money-actually-goes)
8. [The Failure Modes That Only Exist at Volume](#8-the-failure-modes-that-only-exist-at-volume)
9. [Concurrency Control Over Shared Resources](#9-concurrency-control-over-shared-resources)
10. [Capacity Planning With an Honest Model](#10-capacity-planning-with-an-honest-model)
11. [Testing and Validating at Scale](#11-testing-and-validating-at-scale)
12. [The Organisational Dimension at Volume](#12-the-organisational-dimension-at-volume)
13. [The Regulated-Enterprise and Banking Angle](#13-the-regulated-enterprise-and-banking-angle)
14. [Worked Example: Cymbal Bank Scales a Fleet](#14-worked-example-cymbal-bank-scales-a-fleet)
15. [The Anti-Patterns and the Claims Audit](#15-the-anti-patterns-and-the-claims-audit)
16. [What Could Not Be Verified, Glossary, Cross-References and Closing](#16-what-could-not-be-verified-glossary-cross-references-and-closing)

---

## 1. The Overview, the Boundary and the Decoder

**Thesis, in one line: a fleet is not one agent multiplied — the cost curve, the failure modes and the binding constraint all change shape when you run many.**

Multiply is the intuitive move and it is wrong in a specific, expensive way. "One agent, times a thousand" assumes that each session is an independent, short-lived, cheap, stateless unit of work — that if one agent costs a dollar a day, a thousand cost a thousand dollars a day, that if one agent occasionally times out, a thousand occasionally time out a thousand times as often, and that the hard part scales linearly so the hard part stays hard in the same place. None of those hold. The cost curve stops being linear because a session's cost is dominated by its **long tail**, not its median. The failure modes stop being the same failures because a fleet can generate failures that no individual session can generate — a retry storm is not "many retries", it is a new phenomenon produced by their interaction. And the binding constraint moves: at ten sessions you are limited by how good the model is, at ten thousand you are limited by rate limits, KV-cache memory, downstream tool endpoints, and the governance question of who is allowed to spend what.

**What changes, and why.** A single agent session is a *stateful, long-running, expensive, and frequently non-idempotent* unit of work. It holds a growing conversation, an accumulating tool-call history, live credentials, and intermediate state across minutes to hours. That is fundamentally unlike a stateless HTTP request, which is short, cheap, re-runnable, and safe to duplicate. A fleet is therefore not a scaled stateless service; it is closer to a fleet of **long-running transactions with external side effects**. When you run one, the queue is empty and every local decision is free. When you run ten thousand, every local decision becomes a global signal: your retry policy becomes a load generator, your context window becomes a memory budget on somebody else's GPU, your per-session token budget becomes a line item on a monthly invoice, and your "just run it again" becomes a duplicated wire transfer.

**The decoder.** The vocabulary in this guide is used precisely, because loose vocabulary in this domain is what produces the wrong architecture.

| Term | What it means here | Why it matters at scale |
|---|---|---|
| **Session** | One logical agent run: a task, its accumulated context, its tool calls and its state, from start to terminal outcome. Long-running and stateful. | The unit of work that consumes a slot, a budget and a lock. |
| **Concurrency** | How many sessions are *in flight at the same moment*. A count, not a rate. | Determines memory (KV cache), downstream pressure and lock contention. |
| **Throughput** | How many sessions *complete per unit time*. A rate. | Determines cost per day and whether the queue drains. |
| **In-flight work** | Everything a session is currently holding: an open model call, an open tool call, its context, its state. | Concurrency is really "how much in-flight work you can hold", not "how many processes you started". |
| **The serving layer** | Whatever turns prompts into tokens: a provider endpoint or your own vLLM/SGLang/TGI/TensorRT-LLM/Triton deployment. | Has its own capacity model, entirely different from the orchestration layer's. |
| **Batching** | Serving-layer technique of processing multiple requests' decode steps together to raise GPU utilisation. | The reason your per-request latency and your aggregate throughput trade against each other. |
| **Prefill / decode split** | Prefill processes the input tokens; decode generates output tokens one step at a time. | Agent workloads are prefill-heavy and long-context; chat workloads are not. This is a serving problem. |
| **Quota** | A provider- or platform-imposed ceiling: requests/tokens per window, spend per window, or provisioned capacity. | The real ceiling on fleet size; usually hit long before CPU does. |
| **Backpressure** | A mechanism that slows *producers* when consumers saturate, instead of letting a queue grow. | The difference between a slow fleet and an outage. |
| **Admission control** | Deciding whether to accept a new session at all, before doing work for it. | The only place where shedding cheap; rejecting early costs nothing. |
| **Fan-out** | One session spawning sub-agents, parallel branches or many tool calls. | The primary multiplier on both cost and failure surface. |
| **Cost per task** | Total spend attributable to one completed task, including retries, tool calls and sub-agents. | The only cost unit a business owner can reason about. |
| **The long tail** | The p95/p99 session — the one that ran 40 steps instead of 6 — as distinct from the median. | At fleet scale the tail, not the median, sets the bill. |

**Boundary declaration.** This guide is the **concurrency, serving, economics and volume-failure guide** and nothing else. It does not own scaling *requirements and targets* — [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) §5 owns those, and where targets appear here they are inputs to arithmetic, not a requirements list. It does not own **observability, evaluation, monitoring or the tooling landscape** — [agentops_guide.md](agentops_guide.md) §3–§7 owns those. It does not own **production readiness or cost *controls*** such as token budgets per task, prompt/response caching and model routing — [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) §7 owns those; §6 of this guide derives the *economics* those controls act on, and §7 hands the attribution question to AgentOps rather than re-deriving it. It does not own **multi-backend or model routing** ([hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md) §6–§7), **agent topology or hierarchy** ([hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md)), **durable execution, idempotency, sagas or compensation** ([durable_ai_agent_workflows_guide.md](../durable_ai_agent_workflows_guide.md)), or the **general failure taxonomy and compounding-error arithmetic** ([llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md), [agents_work_fall_apart_guide.md](agents_work_fall_apart_guide.md)). The test I apply to every paragraph in this guide is: *would this sentence be equally true of a single agent?* If yes, it belongs in one of those guides, and I have tried to leave it there.

---

## 2. What Actually Changes at Scale

Before any mechanism, the shape change. Write down the four properties of an agent session and the consequences follow almost mechanically:

| Property of one session | Consequence for one session | Consequence for a fleet |
|---|---|---|
| **Long-running** (minutes to hours) | A slow step is annoying | A slow step occupies a slot, a connection, a lock — capacity is held by *time*, not by count |
| **Stateful** (context + session state accumulate) | Context grows; the model sees more | Aggregate memory (KV cache, trace storage, retrieval index) grows with *concurrency × context length* |
| **Expensive per unit of work** | A single run costs cents to dollars | The bill scales with *work done*, and work done is variable and unbounded per task |
| **Frequently non-idempotent** (it sends emails, moves money, writes files) | You simply do not re-run it | Retry safety becomes a fleet-wide invariant, or retries are forbidden |

Four things change, and they change in the same direction — non-linearly:

**(i) The cost curve.** Chat is roughly linear in requests: each request has a bounded input and a bounded output, and the average is a good predictor of the sum. An agent fleet is not. Cost per session is a *product* of steps × tokens per step, where steps is itself a random variable with a heavy tail, and each step may fan out into sub-agents and tool calls. A workload whose median session costs $0.04 and whose p99 costs $1.80 does not cost "4 cents times N" — the sum is dominated by the tail, and the tail is populated by exactly the sessions that are hardest to observe (they are the ones that went wrong, retried, and drifted).

**(ii) The failure modes.** Most agent failure literature describes per-session failures: a bad tool call, a hallucinated argument, a loop that never terminates. A fleet introduces failures that are *emergent* — properties of the population of sessions interacting through shared infrastructure. A retry storm is not a failed session; it is thousands of sessions, each behaving reasonably, collectively saturating a rate limit and then collectively retrying. A thundering herd is not a slow endpoint; it is a synchronised start. Cost blowout is not an expensive session; it is a session that costs ten times its median *while the fleet is also running at ten times its normal concurrency*. None of these can be reproduced with one agent, which is exactly why they are always discovered in production. §8 is devoted to them.

**(iii) Which constraint binds.** At ten sessions the binding constraint is model quality: does the agent do the task correctly? At ten thousand it is usually none of the things the team optimised for. It is the provider's tokens-per-minute quota, the KV-cache memory available at the serving layer, a downstream tool endpoint that was never designed for concurrent access, the cost of the tail, or a governance rule about who may spend what. This is the single most common surprise: teams spend a year improving task quality and then discover the fleet is capped at 400 concurrent sessions by a quota nobody read.

**(iv) The organisational question.** At ten sessions, one team owns the agent — its key, its budget, its failures. At ten thousand, several teams share a limited pool of quota and capacity, and somebody has to decide who gets it when demand exceeds supply. Cost becomes attributable spend; failures become someone's incident; a shared endpoint becomes a shared dependency with no single owner. §12 and §13 cover this, briefly and by cross-reference, because the answer is organisational rather than technical.

**The honest early warning.** Practices that work at ten sessions fail at ten thousand for reasons that have nothing to do with model quality. A retry without jitter that is *correct* at ten sessions is *harmful* at ten thousand, because the retries synchronise. A per-request timeout that is generous at ten sessions is a capacity leak at ten thousand. A context that grows unboundedly is a curiosity at ten sessions and a memory-budget catastrophe at ten thousand. A shared row in a database that two sessions never touch simultaneously at ten sessions becomes a lost-update incident at ten thousand. If you take one thing from this guide: **the fleet's behaviour is not the sum of session behaviours, and the difference is where the money goes and where the outages come from.**

---

## 3. The Concurrency Model

### 3.1 What a session holds while in flight

An "active session" is not a CPU percentage. It is an open set of claims on resources. A single session, mid-flight, can hold all of the following simultaneously:

| Held resource | Typical nature | Failure if over-held |
|---|---|---|
| An open model call | A streaming HTTP request, seconds to minutes | Connection exhaustion at the client; quota consumption at the provider |
| Its **accumulated context** | Tokens in the model's window plus the KV cache serving them | Memory at the serving layer; cost on every subsequent step |
| **Tool calls in flight** | HTTP/SDK calls, often to slow legacy systems | Occupied session slot, thread/goroutine, connection |
| **Session state** | Checkpoints, scratch files, sandbox workspace, credentials | Storage growth, credential sprawl, sandbox churn |
| **Locks and leases** | Rows, files, accounts, idempotency keys | Contention, deadlock, starvation |
| **A queue slot / trace span** | In-memory queue entry, trace storage | Memory and storage growth not visible until it is a page |

The practical consequence is that **the number of sessions you can run concurrently is set by the scarcest of these holdings, not by the one you optimised**. Teams routinely size for compute ("our orchestrator handles 500 pods") and then discover the real limit is 120, because each session holds an open database connection to a legacy core system whose pool is capped at 200.

### 3.2 Where concurrency physically lives

Concurrency exists in two distinct places, and conflating them produces both over-provisioning and mysterious latency:

| Layer | What concurrency means there | What limits it | What you can do about it |
|---|---|---|---|
| **Orchestration layer** (your app, workers, agents, tool runners) | Number of sessions and sub-agents executing/awaiting | CPU/memory of workers, open connections, downstream endpoints, your own semaphores | Add workers; add per-dependency concurrency limits; queue and admit |
| **Model-serving layer** (provider endpoint, or your vLLM/SGLang/TGI/TensorRT-LLM/Triton inference servers) | Number of requests being processed or batched at once, and the KV-cache memory they occupy | GPU memory, batch size, prefill/decode scheduling, provider quota | Buy more quota/capacity; raise concurrency on your own hardware; shrink context |

These are different capacity problems with different currencies — worker pods versus GPU memory and quota — and they are coupled: raising orchestration concurrency without raising serving capacity converts a compute problem into a queueing problem, and raising serving capacity without fixing a downstream tool bottleneck converts it into a timeout problem. §4 handles the serving layer; §10 handles the arithmetic.

### 3.3 Concurrency versus throughput

The most commonly conflated pair in this domain:

- **Concurrency** = how many sessions are in flight *now*. A snapshot count. Bounded by memory, connections, locks and quota *rate-of-use*.
- **Throughput** = how many sessions complete *per hour*. A rate. Bounded by the bottleneck resource's service rate.

They are related by Little's Law: **throughput ≈ concurrency ÷ average session duration**. That single relation explains most fleet surprises:

- If average session duration is 40 minutes and you can hold 400 concurrent sessions, throughput is at most 10 completed sessions per minute — 600 per hour. Nothing about your code changes that; only lowering duration or raising concurrency does.
- If the tail lengthens (sessions start taking 4 hours instead of 40 minutes), *throughput collapses by 6× with no change in concurrency and no change in demand*. A fleet can fail its SLA purely by having slower sessions, which is why latency work ([llm_latency_optimization_guide.md](llm_latency_optimization_guide.md)) is also capacity work.
- Conversely, doubling concurrency to "go faster" raises throughput only until the bottleneck — usually the serving layer or a tool endpoint — saturates. Past that point extra concurrency adds queueing delay, not completions, and can *reduce* throughput by thrashing.

### 3.4 A slow tool call occupies a slot just as surely as a slow model call

This is the consequence most teams learn the hard way. A session waiting 30 seconds on a core-banking SOAP endpoint holds: its worker slot, its connection, its context (already paid for), its locks, and its place in the queue. From the fleet's perspective that session is *fully occupied and producing nothing*. A model call that takes 30 seconds is equivalent. Therefore:

- **Concurrency budgeting must include tool time, not just model time.** If a session spends 70% of its wall-clock waiting on tools, a concurrency of 500 is really only ~150 sessions' worth of model-serving demand — but still 500 sessions' worth of worker, connection and lock demand.
- **Tool latency is a fleet-wide capacity parameter.** One slow shared dependency converts every session that touches it into a slot-holder. This is why per-tool concurrency limits and timeouts are capacity controls, not just reliability controls.
- **Sessions that await human input hold a slot unless you deliberately release it.** Long human-in-the-loop pauses must park the session (durable workflow, released worker) rather than occupy concurrency; [durable_ai_agent_workflows_guide.md](../durable_ai_agent_workflows_guide.md) owns the parking mechanism, this guide owns the capacity consequence.

---

## 4. The Serving Layer

### 4.1 Continuous batching, stated as the sources state it

When many sessions meet the model endpoint, what happens is a serving-layer problem, not a prompt problem. The relevant mechanism is **continuous batching** (also called in-flight batching or iteration-level batching): instead of waiting for a batch of requests to be assembled and running it to completion, the server schedules at the granularity of individual decode steps, admitting new requests as slots free and retiring finished ones.

Verified only to the extent the frameworks' own documentation states it; I have not invented a throughput figure, and I note explicitly that **no public benchmark exists for "agents at scale"** — anyone quoting a concrete "agents per GPU" number is quoting either a vendor's best case or nothing.

| Framework | What its own documentation says about batching/concurrency | Status |
|---|---|---|
| vLLM | Its README lists, in the project's own words: "Continuous batching of incoming requests, chunked prefill, prefix caching"; "Efficient management of attention key and value memory with **PagedAttention**"; and "**Disaggregated prefill, decode, and encode**" — i.e. the prefill/decode split of §4.3 is a first-class deployment option, not an abstraction. No numeric concurrency or throughput figure is stated on that page. Source: `raw.githubusercontent.com/vllm-project/vllm/main/README.md` (project README). | **VERIFIED 2026-09-22** (mechanics); no figure stated |
| HuggingFace TGI | Its documentation lists "**Continuous batching of incoming requests for increased total throughput**" among the server's features, alongside streaming, tensor parallelism and OpenTelemetry/Prometheus instrumentation. The same page states TGI is now in **maintenance mode**, with HF pointing users to vLLM and SGLang as the engines it contributes to — a 2026 fact worth knowing before standardising on it. Source: `huggingface.co/docs/text-generation-inference/index`. | **VERIFIED 2026-09-22** (mechanics); no figure stated |
| SGLang | Its documentation landing page describes the stack as "designed for low-latency, high-throughput inference with **RadixAttention, prefix caching**, and multi-GPU parallelism", and cites "**PD disaggregation**" and "KDA-aware prefix caching" in its newest model recipes (Kimi-K3). Continuous batching is not phrased on the landing page I read; prefix caching is. Source: `docs.sglang.ai`. | **VERIFIED 2026-09-22** (RadixAttention + prefix caching); continuous-batching wording ⚠ not confirmed on that page |
| TensorRT-LLM | Its documentation tree (1.3.0rc27) contains a feature page titled "**In-flight Batching**" under the Paged-Attention/IFB-scheduler section and a "**Disaggregated Serving**" feature with a technical blog covering KV-cache exchange — the term *in-flight batching* is this stack's own, as claimed in §4.3. Source: `nvidia.github.io/TensorRT-LLM`. | **VERIFIED 2026-09-22** (feature exists, term is theirs); mechanics page body not read |
| NVIDIA Triton | Its architecture page describes request routing to a **per-model scheduler**, "multiple scheduling and batching algorithms that can be configured on a model-by-model basis", and an *optional* batching step before the backend — i.e. **request-level dynamic batching**, which is a different mechanism from continuous batching inside an LLM backend, and it states no LLM concurrency semantics of its own. Version read: 2.72.0. Source: `docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/user_guide/architecture.html`. | **VERIFIED 2026-09-22** (request-level batching); no continuous-batching claim |

⚠ **Still honest about the limits of this table:** the *mechanics* above are now verified at each project's own documentation, but **none of those pages states a throughput or concurrency figure** — no "sessions per GPU", no "requests per second". Frameworks publish performance studies on their blogs and in papers, at specific hardware and model configurations that are not your workload; **this guide therefore contains no per-framework number at all**, and §10.4 says what to measure instead.

### 4.2 KV-cache pressure: why long agent contexts are a serving problem

Autoregressive generation needs the attention keys and values for every token already in context. A serving engine holds that KV cache in GPU memory for each in-flight request. Two consequences:

1. **Memory per concurrent request scales with context length.** A request with 40,000 tokens of accumulated agent context occupies far more cache than a chat turn with 800 tokens. So *concurrency at fixed GPU memory is inversely related to context length* — and agent contexts grow monotonically within a session, so a fleet of long sessions has a *shrinking* concurrency ceiling over time, even at constant request rate.
2. **Context growth is therefore a fleet-wide capacity decision, not a prompt-tuning decision.** Trimming, summarising, compacting or retrieving-instead-of-accumulating liberates serving memory and cost at the same time. Context engineering is covered in [context_engineering_guide.md](context_engineering_guide.md); the *fleet* consequence — that unbounded context growth is a slow-motion capacity and cost failure — is owned here and is §8.5.

This is why a naive capacity model ("we serve X requests per second") fails for agents: the unit of serving work is not a request, it is *a request with an arbitrary and growing token footprint*.

### 4.3 Prefill versus decode: why agent workloads stress serving differently from chat

Generation has two phases:

- **Prefill** — process the whole input prompt, compute the first token. Compute-heavy (matmul-bound), highly parallelisable, and scales with *input length*.
- **Decode** — produce subsequent tokens one step at a time, each step attending to all previous tokens. Memory-bandwidth-bound, poorly parallelisable per request, and scales with *output length*.

Chat traffic is output-heavy and modest-input: lots of requests with short prompts. Agent traffic is the inverse proportion: **big, repeatedly-resubmitted, growing inputs, and comparatively short outputs per step** — a plan, a tool call, a sentence of reasoning. Every agent step re-sends the accumulated context, so a 12-step session with 30,000-token context performs ~360,000 tokens of prefill work for maybe 2,000 tokens of useful output. That profile:

- Stresses **prefill compute** and **KV-cache memory** far more than decode throughput;
- Makes **prefix/prompt caching** disproportionately valuable (a shared prefix, re-sent per step, is the ideal cache candidate) — see §6.3 for the discount rules;
- Means chat-shaped capacity assumptions overstate how many agent sessions a given deployment supports, sometimes by an order of magnitude. This is an inference quality statement, ⚠ not a measured benchmark, and I do not attach a number to it.

### 4.4 Concurrency limits that come from the endpoint, not from your code

Fleet size is frequently capped by something outside your repository:

| Source of limit | How it appears | Typical symptom |
|---|---|---|
| Provider RPM/ITPM/OTPM quota | 429 with `retry-after`; headers showing remaining budget | Throughput plateaus; retries climb |
| Provider spend cap | 429 or 400 depending on provider, no retry helpful | Fleet stops mid-month (Anthropic: 429 `enforced_spend_limit_reached` on the tier cap, HTTP 400 for a self-set limit — verified) |
| Providers' per-window spend rate limits | `429 RESOURCE_EXHAUSTED` (Gemini, verified: $10/$50/$200 per 10 minutes on Tiers 1/2/3) | Sharp plateau then errors under load |
| Your own deployment's KV cache | Preemption/queueing inside the engine; rising time-to-first-token | TTFT degrades while token/s looks fine |
| Provisioned capacity | Fixed PTU/model-unit ceiling; capacity, not quota, decides whether a deployment even exists (Azure, verified) | Deployment fails or throughput is hard-capped |
| A downstream tool or database | Connection pool exhaustion, timeouts | Sessions hold slots; model endpoint looks idle |

The design rule that follows: **know which of these is your binding constraint, and put the control where it binds.** If the serving layer is the ceiling, the control is admission control on sessions (§10.4). If a tool endpoint is the ceiling, the control is a per-tool semaphore. If it is quota, the control is a fleet-wide token-rate governor (§5.4). Guessing wrong produces a limiter that never fires while the real ceiling is exceeded.

---

## 5. Rate Limits, Quotas and Backpressure

### 5.1 How the limits are expressed (verified per provider, read 22 September 2026)

Every large provider expresses its ceiling in more than one dimension, and the one you forgot is the one that fires first.

| Provider / product | Dimensions of the limit | Tier structure | Verified value(s) read on 22 Sep 2026 | Source |
|---|---|---|---|---|
| **OpenAI API** | Requests per minute and tokens per minute, with separate limits per model; batch workloads do not consume synchronous rate limits | Per-account limits set by usage | Page read confirms RPM and TPM are separate and enforced simultaneously; the specific RPM/TPM numbers are account-specific and not published as a general table ⚠ | platform.openai.com/docs/guides/rate-limits |
| **Anthropic Claude API** | Requests/min (RPM), **input** tokens/min (ITPM), **output** tokens/min (OTPM), measured per model class; plus spend caps | Start / Build / Scale / Custom; new orgs may start on an Evaluation tier with lower limits | Spend caps: Start $500/month, Build $1,000/month, Scale $200,000/month; Custom has no cap. Per-model table read shows 1,000 RPM / 2,000,000 ITPM / 400,000 OTPM for Claude Sonnet 5 and Opus 5 ⚠ (the page renders these under a tier tab; the extraction could not confirm which tier tab was active — treat the *shape* as verified, the tier label as unconfirmed) | docs.anthropic.com/en/api/rate-limits |
| **Google Gemini API** | RPM, input TPM, **RPD** (requests per day), plus per-model extras (images/min, tokens/day); separate limits for Priority and Batch | Free / Tier 1 / Tier 2 / Tier 3 | Limits are enforced **per project, not per API key**; RPD resets at midnight Pacific. **Spend-based rate limits on a rolling 10-minute window: Tier 1 $10, Tier 2 $50, Tier 3 $200.** Billing tier caps: Tier 1 $250, Tier 2 $2,000, Tier 3 $20,000–$100,000+. Priority inference default = **0.3×** the standard rate limit. Batch: 100 concurrent batch requests, 2 GB input file, 20 GB file storage | ai.google.dev/gemini-api/docs/rate-limits (page last updated 2026-09-02) |
| **Azure OpenAI / Microsoft Foundry Models** | TPM (and RPM derived from it) per model per deployment; separately, **PTU quota** for provisioned deployments | Free tier + Tiers 1–6, auto-upgrading | Quotas are **not enforced at tenant level** — the highest scope is the **subscription**; same-model same-version deployments share one quota pool across regions (Global Standard) or per data zone. RPM is derived: the page states `gpt-chat-latest` versions use **10 RPM per 1,000 TPM**, and version `2026-08-06` uses **1 RPM per 1,000 TPM** | learn.microsoft.com/.../openai/quotas-limits (page last updated 2026-08-20) |
| **AWS Bedrock** | Per-model TPM and RPM quotas, per account per region, for on-demand and cross-region inference; separate input and output token quotas per model | Adjustable (via Service Quotas) vs non-adjustable (support request) | Example read from the quota table: `[bedrock-mantle endpoint] Output tokens per minute for GPT-6 Astra — Each supported Region: 100,000`. Quota-increase requests are prioritised for customers already consuming their allocation | docs.aws.amazon.com/general/latest/gr/bedrock.html |
| **Azure OpenAI provisioned (PTU)** | Dedicated processing capacity, billed per PTU-hour; capacity ≠ quota | Per subscription, per region, per deployment type | PTUs are **model-independent** (the same PTU quota can deploy any supported model), quota is granted per subscription/region/deployment-type, and **TPM delivered per PTU varies by model**; having quota does **not** guarantee capacity is available | learn.microsoft.com/.../openai/concepts/provisioned-throughput (updated 2026-07-15) |

Three structural observations that matter more than any individual number:

1. **Input and output are separate budgets almost everywhere.** Anthropic splits ITPM and OTPM explicitly; Bedrock publishes separate input and output token quotas; OpenAI enforces RPM and TPM together. A fleet that is comfortably inside its token budget can still be blocked by the request-count limit, and vice versa. §5.3 shows why.
2. **Caching changes what counts.** Anthropic states that for most Claude models **only uncached input tokens count toward ITPM** — `cache_read_input_tokens` do not count, though `cache_creation_input_tokens` do — and gives the worked example: *with a 2,000,000 ITPM limit and an 80% cache hit rate you could effectively process 10,000,000 total input tokens per minute*. OpenAI's prompt-caching page states the opposite for its own API: **cached input tokens still count toward tokens-per-minute limits**. ⚠ So "cache to raise your ceiling" is a provider-specific claim, verified for Anthropic and verified-not-to-hold for OpenAI.
3. **Spend limits are a *third* kind of ceiling, and they fail differently.** Anthropic's tier spend cap returns HTTP 429 with `error.details.error_code = enforced_spend_limit_reached` and, notably, **no `retry-after`** — retrying (including the SDKs' automatic retries) just fails until the month rolls over. A self-set spend limit below the tier cap returns HTTP 400 `invalid_request_error` instead. Gemini's spend limit is a rolling 10-minute window, so it recovers in minutes rather than a month. Same word, completely different recovery semantics.

### 5.2 Provider limits that live in your own deployment

If you self-host (vLLM, SGLang, TGI, TensorRT-LLM, Triton), you have no provider quota but you have a harder ceiling: **KV-cache memory and batch slot count on your GPUs**. The limits are then: model weights + KV cache must fit in VRAM; the number of concurrent sequences is whatever fits; and preemption/queueing appear as rising time-to-first-token rather than as a 429. ⚠ I could not verify per-framework concurrency statements in this pass (docs.vllm.ai failed to return content; SGLang, TGI, TensorRT-LLM and Triton pages were not fetched) — see §16. The design consequence is unchanged: your serving layer is the ceiling, and the control is admission control on the session layer (§10.4), because there is no `retry-after` header to obey when the bottleneck is your own memory.

### 5.3 The arithmetic: how much fleet does one quota support?

This is the chain that answers "how many agents can we run?". It has four links, and the answer is the **smallest** of the three provider dimensions.

**Illustrative worked chain.** Take the Anthropic Sonnet 5 figures as read on 22 Sep 2026 (1,000 RPM, 2,000,000 ITPM, 400,000 OTPM), an illustrative session of 8 model calls, 400 output tokens per call, 18,000 input tokens per call (85% of them cache-hit), and an illustrative average session duration of 40 minutes.

| Step | Arithmetic | Result |
|---|---|---|
| Output tokens per session | 8 calls × 400 tokens | 3,200 tokens |
| Sessions/min allowed by OTPM | 400,000 ÷ 3,200 | 125 sessions/min |
| Uncached input tokens per session (cache-aware) | 8 × (18,000 × 15%) = 8 × 2,700 | 21,600 tokens |
| Sessions/min allowed by ITPM, cache-aware | 2,000,000 ÷ 21,600 | **93 sessions/min** ← binds |
| Uncached input per session, **no caching** | 8 × 18,000 | 144,000 tokens |
| Sessions/min allowed by ITPM, no caching | 2,000,000 ÷ 144,000 | **14 sessions/min** ← binds, 6.7× worse |
| Sessions/min allowed by RPM | 1,000 ÷ 8 | 125 sessions/min |
| Steady-state concurrent sessions at 40 min (Little's Law) | sessions/min × 40 min | **3,700** cache-aware / **556** uncached |

Read that table twice. **The same quota supports either ~3,700 or ~556 concurrent sessions, and the only difference is whether the cache is warm.** Every figure above is ILLUSTRATIVE (the session shape is invented; the rate numbers are VERIFIED as read on 22 Sep 2026 for the tier shown on the page, with the tier label ⚠ unconfirmed). Substituting your own session shape is the whole point of showing the chain — do not quote my output as a capacity number. And note the practical implication: **quota arithmetic is a caching argument before it is a capacity argument.**

### 5.4 Handling a rejected request

A rejection is not one thing. The recovery action differs by cause, and treating them alike is how a hiccup becomes an outage.

| Response | Cause | Correct handling | Wrong handling |
|---|---|---|---|
| 429 with `retry-after` | Rate limit (RPM/ITPM/OTPM) | Honour the header; add jitter; retry is *safe for the fleet* only if the retry is spread | Immediate or synchronised retry → storm (§8.1) |
| 429 without `retry-after`, Anthropic `enforced_spend_limit_reached` | Monthly tier spend cap reached | **Stop the fleet and escalate.** No retry will help | Retry loop; SDK auto-retry will burn against a wall for the rest of the month |
| 400 `invalid_request_error`, "reached your specified API usage limits" | Self-set spend limit | Stop, or raise the limit deliberately | Treat as a transient 429 |
| `429 RESOURCE_EXHAUSTED` on a rolling window (Gemini) | Spend-rate limit (10-minute window) | Short backoff, then resume; the window self-heals | Raising retry count without backoff |
| 5xx / connection reset | Serving-layer or transport failure | Retry with jitter and a cap; prefer streaming-aware retry | Unbounded retries |
| 200 with a *worse* result (e.g. degraded fallback) | Routing/fallback | Detect and record; do not silently accept | Silently accepting a quality regression |

Two mechanisms worth naming explicitly, both of which are **configuration toggles** rather than design decisions:

- **`retry-after` is an instruction, not a suggestion.** A client that retries before the header elapses has chosen to add load to a saturated system.
- **SDK automatic retries** (OpenAI, Anthropic and most SDKs ship them) are on by default and are a fleet-wide load policy whether or not you thought about it. Verify the setting, the max attempts and the backoff *before* you have ten thousand sessions. A default of 2–3 retries per call is a 2–3× amplification multiplier at the moment of saturation, which is exactly the wrong time.

### 5.5 Why retrying without jitter turns a rate limit into an outage

This is a mechanism, not a folk warning, and it is the setup for §8.1. Chain it:

1. A shared dependency (provider endpoint, gateway, tool API) becomes slow or starts rejecting. Latency rises above the client timeout, or 429s appear.
2. Every in-flight session that observed the failure **retries at the same time** — because they all failed at the same time and all use the same fixed backoff.
3. The retries arrive as a *synchronised batch*: the offer of load to the dependency becomes a spike rather than a stream. Synchronisation is the whole problem: independent failures plus identical policies equal correlated retries.
4. The spike exceeds the capacity that is *already* reduced (the dependency is degraded precisely because it is overloaded), so a larger fraction of the retry batch fails.
5. Those failures retry again, on the same schedule, and the batch grows (each failing call now spawns up to *k* new calls, where *k* is the retry budget). Amplification factor = expected attempts per request = 1 + Σ p^k.
6. Because the retry population is larger than the original population and still synchronised, the system settles into a **self-sustaining loop** that outlives the original trigger: the dependency cannot recover because it is never given a quiet interval. The load is now generated by the recovery mechanism, not by the workload.
7. **Jitter breaks the synchronisation.** Randomising the backoff decorrelates the batch, converting a spike into a spread. That is why every serious retry design has both exponential growth *and* jitter. OpenAI's own rate-limit guide publishes a Python example with `jitter: bool = True` and `delay *= exponential_base * (1 + jitter * random.random())` — verified, read 22 Sep 2026 — and separately warns that OpenAI makes no guarantees about that example's suitability as a complete solution.

The mechanism generalises: **any fleet-wide policy applied identically by every session at the same instant is a load amplifier at failure time.** Fixed retry backoff, fixed polling interval, fixed session start time, fixed cache TTL expiry — all of them synchronise. Jitter, TTL spreading and randomised start offsets are the countermeasures, and they are cheap.

### 5.6 The design: backpressure, queueing, admission control, shed vs wait

Rate limits are a constraint; the response to them is an architecture. Four mechanisms, in the order they should be applied:

**1. Backpressure (make producers slow down).** Backpressure is a *signal propagated backwards* — when the consumer (serving layer, tool endpoint, queue) saturates, the producer reduces its own rate instead of buffering. In an agent fleet, backpressure means: a fleet-wide token-rate governor that publishes remaining budget, and session workers that self-throttle on it. Without backpressure, the queue absorbs the imbalance until the queue *is* the problem — and an unbounded queue in front of an agent fleet is a latency bomb with a memory leak attached, because queued sessions hold context that was already paid for.

**2. Queueing (bounded, visible, fair).** A queue is not an alternative to backpressure; it is what you do with work you have accepted but cannot yet run. Requirements that matter at fleet scale: the queue is **bounded** (unbounded queues hide overload until they fail catastrophically), **visible** (depth and age are first-class metrics — [agentops_guide.md](agentops_guide.md) §5 owns the alerting), **priority-aware** (interactive before batch), and **age-aware** (a session that has waited 20 minutes may no longer be worth running; its inputs may be stale, which is a business decision as much as a technical one).

**3. Admission control (decide before spending).** Admission control is the only control that costs nothing when it fires, because it rejects *before* doing work. It checks: is there headroom in the binding constraint (quota, serving capacity, tool pool, budget)? Is this tenant inside its per-tenant cap? Is the fleet inside its concurrency ceiling? If not, the session is rejected or parked immediately — with a truthful error, not a slow failure. The alternative — admit everything, fail 40% of the way through — costs full price for no outcome, which is the most expensive failure mode in the whole discipline. Admission control is a **design decision**, not a toggle: it must exist in the request path before the first model call.

**4. Shed vs wait (choose per class of work).** Once saturated, you must choose:

| Strategy | When correct | Cost of being wrong |
|---|---|---|
| **Wait** (queue, retry later) | Work is deferrable and the result remains useful later; e.g. overnight batch enrichment, non-urgent document processing | Stale results; queue growth; SLA breach if the queue cannot drain |
| **Shed** (reject fast) | Work is time-sensitive and a late result is worthless; e.g. interactive assistance in a live customer conversation | Lost work; user-visible errors; retries from the *user* side, which you do not control |
| **Degrade** (route to a cheaper/faster model, shorter context, fewer tools) | Quality is negotiable per request and the business accepts a lower-fidelity answer | Silent quality regression — must be recorded, not hidden (see [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md) §6–§7 for routing; the *admission* decision is here) |

The decision belongs per **workload class**, not per fleet: a bank's overnight reconciliation agent and its in-branch assistant must not share one saturation policy. Also note the asymmetry — **waiting is only safe if the queue is bounded and the work is idempotent.** For non-idempotent work (anything with external side effects), a timeout that fires after the side effect has occurred is not a safe wait; §9 and [durable_ai_agent_workflows_guide.md](../durable_ai_agent_workflows_guide.md) own that.

---

## 6. The Economics

> **This section's prices are a dated snapshot: 22 September 2026.** Provider prices, cache multipliers and batch discounts change, sometimes with a month's notice (the Gemini tables I read contain *published future prices* effective 1 January 2027 — proof that today's number is not tomorrow's). **Get current figures here, and only here:** OpenAI — `platform.openai.com/docs/pricing`; Anthropic — `docs.anthropic.com/en/docs/about-claude/pricing` (and `claude.com/pricing`); Google Gemini — `ai.google.dev/gemini-api/docs/pricing`; Azure OpenAI / Microsoft Foundry — `azure.microsoft.com/pricing/details/cognitive-services/openai-service`; AWS Bedrock — `aws.amazon.com/bedrock/pricing/`. Everything in §6 is either VERIFIED-as-read (with the date) or ILLUSTRATIVE (constructed arithmetic input). Nothing here is a forecast.

### 6.1 The cost model from the unit up

The unit is one **model call within a session**, not one "task". Anything coarser hides the multipliers in §6.4.

**Step 1 — the token split.** A model call is billed on input tokens and output tokens at different rates, and on cached input at a third rate. Verified prices read 22 Sep 2026, per million tokens (MTok):

| Provider / model | Input | Cached input | Output | Output ÷ input ratio | Status |
|---|---|---|---|---|---|
| Anthropic Claude Sonnet 5 | $2.00 | $0.20 (cache read) | $10.00 | **5.0×** | VERIFIED 2026-09-22 |
| Anthropic Claude Haiku 4.5 | $1.00 | $0.10 | $5.00 | 5.0× | VERIFIED 2026-09-22 |
| Anthropic Claude Opus 5 | $5.00 | $0.50 | $25.00 | 5.0× | VERIFIED 2026-09-22 |
| Anthropic Claude Fable 5.1 | $10.00 | $0.25 (0.025×) | $50.00 | 5.0× | VERIFIED 2026-09-22 |
| OpenAI `chat-latest` | $5.00 | $0.50 | $30.00 | 6.0× | VERIFIED 2026-09-22 |
| OpenAI `gpt-5.3-codex` | $1.75 | $0.175 | $14.00 | 8.0× | VERIFIED 2026-09-22 |
| Google `gemini-3.8-flash` (standard) | $0.75 | $0.075 (caching) | $3.75 | 5.0× | VERIFIED 2026-09-22 (page updated 2026-09-16; prices rise to $1.50/$0.15/$7.50 on 2027-01-01 as published) |
| Google `gemini-3.8-flash` (batch / Flex) | $0.375 | $0.0375 | $1.875 | 5.0× | VERIFIED 2026-09-22 |
| AWS Bedrock (example: Writer Palmyra X5, on-demand) | $0.003 / 1K tokens | — | $0.015 / 1K tokens | 5.0× | VERIFIED 2026-09-22 |

**Why agents are OUTPUT-heavy relative to chat — stated precisely.** Per *task*, an agent produces far more output tokens than a chat turn, because every step emits its own output (reasoning, a tool call, a summary) and there are many steps. A 10-step session produces 10 billed outputs; a chat exchange produces one. Because output is priced 5–8× input, a task that is *input-dominated in token count* can still be *output-dominated in cost*. The two statements are not in conflict, and the distinction is important:

- **Token volume per session is input-dominated** (agent contexts are large and re-sent).
- **Cost per session is a blend, and the output side is priced 5–8× higher**, so a session with, say, 20% of its tokens on the output side still pays most of its bill there if caching is not used.

**Step 2 — cached versus uncached input.** This is the single largest controllable lever in the model. Verified rules, read 22 Sep 2026:

| Provider | Cache-write cost | Cache-read (hit) cost | Minimum / conditions stated by the source | Status |
|---|---|---|---|---|
| Anthropic | 1.25× base input (5-minute cache), **2× base input (1-hour cache)** | 0.1× base input (0.025× on Fable 5.1 / Mythos 5.1) | Two modes: automatic caching (`cache_control` at top level) and explicit breakpoints; cached reads **do not count toward ITPM** for most models | VERIFIED 2026-09-22 |
| OpenAI | — | Cached input billed at the cached-input rate (0.1× input for `chat-latest` and `gpt-5.3-codex`, per the pricing table) | Implicit and explicit modes; minimum cacheable prefix **varies by model**; cached tokens **still count toward TPM**; compaction can reduce prefix reuse; explicit breakpoints needed for developer messages after the first block | VERIFIED 2026-09-22 |
| Google Gemini | Storage billed per **token-hour** (e.g. $0.50 per 1M tokens per hour for `gemini-3.8-flash` standard) | Cached input at 0.1× input | **Implicit caching is on by default for Gemini 2.5 and newer**; minimum cacheable input **4,096 tokens** for Gemini 3.x, 2,048 for 2.5 | VERIFIED 2026-09-22 |
| AWS Bedrock | — | Reduces input token cost | Two types: **implicit** caching (automatic, best-effort — "repeating an identical prompt doesn't guarantee a cache hit") and **explicit** caching (you place breakpoints) | VERIFIED 2026-09-22 |

Three fleet-level consequences: (i) a warm cache is worth more than a cheaper model for long-context agents — a 0.1× read rate against a 4.26× saving on input is a bigger lever than a model downgrade; (ii) **cache hit rate becomes a capacity metric as well as a cost metric** on Anthropic (ITPM does not count cached reads) but *not* on OpenAI (TPM counts them); (iii) **cache TTL is a traffic-shaping parameter** — a 5-minute TTL that expires across a fleet-wide pause produces a synchronised cold-cache stampede, which is a cost event *and* a load event (§8.2).

**Step 3 — per-call tool and retrieval costs.** Tools are not free and are usually invisible in the model bill:

| Cost source | How it is billed | Verified example / status |
|---|---|---|
| Hosted search grounding | Per request above a monthly free allowance | Gemini: 5,000 free search requests per month shared across Gemini 3.x, then **$14 per 1,000 requests** (VERIFIED 2026-09-22). Grounding with Google Maps: 5,000 prompts/month free, then $14 / 1,000 queries |
| Agent environment compute | Often not billed during previews, billed later | Gemini states environment compute (CPU, memory, sandbox execution) **is not billed during the preview period** (VERIFIED 2026-09-22) ⚠ — a preview-specific waiver is a future cost you have not seen yet |
| Embeddings for retrieval | Per input token | Priced separately per provider; do not forget it in the model |
| Your own tool execution | Internal cost: compute, DB, licence, per-call vendor fees | ⚠ Not verifiable from provider docs — it is your bill. Model it as a per-call constant |
| Retries | Full price, per attempt | The most expensive line item nobody puts in the spreadsheet |

### 6.2 From step to session to task: the unit chain

Every figure in this sub-section is **ILLUSTRATIVE** — constructed inputs, shown so you can substitute your own. The verified anchors are the *rates* from §6.1 (Claude Sonnet 5: $2/MTok input, $0.20/MTok cache read, $10/MTok output, read 22 Sep 2026).

Assumed session shape (ILLUSTRATIVE): 18,000 input tokens and 400 output tokens per model call; 8 calls for a median session; 85% of the input cache-hit.

| Quantity | Unit chain | Result |
|---|---|---|
| Input cost per call, uncached | 18,000 tokens ÷ 1,000,000 × $2.00 | $0.0360 |
| Output cost per call | 400 ÷ 1,000,000 × $10.00 | $0.0040 |
| **Call cost, no caching** | $0.0360 + $0.0040 | **$0.0400** |
| Uncached input tokens per call | 18,000 × 15% | 2,700 |
| Cached input tokens per call | 18,000 × 85% | 15,300 |
| Input cost per call, cache-aware | 2,700 ÷ 1e6 × $2.00 + 15,300 ÷ 1e6 × $0.20 | $0.00846 |
| **Call cost, cache-aware** | $0.00846 + $0.0040 | **$0.0125** |
| Input-cost reduction from caching | $0.0360 ÷ $0.00846 | **4.26×** |
| **Median session** | 8 calls × $0.0125 | **$0.0997** |
| **p99 session** (40 calls, average context 60,000 tokens) | 40 × ($0.0322) | **$1.29** |
| p99 ÷ median | $1.29 ÷ $0.0997 | **12.9×** |
| **Blended mean, 94/5/1 mix of median/20-call/p99 sessions** | 0.94 × $0.0997 + 0.05 × $0.2492 + 0.01 × $1.29 | **$0.1190** |
| **Cost per completed task (blended mean)** | — | **≈ $0.12** |
| **Cost per million tasks** | $0.1190 × 1,000,000 | **≈ $119,000** |

Two things to read out of that chain. First, **the mean is 19% above the median** ($0.119 vs $0.0997) purely because of 6% of sessions in the tail — and the tail is where the *detection* problem lives, because the median dashboard looks healthy. Second, the blended rate is arithmetic on invented inputs: change the context length, the step count or the cache hit rate and the answer moves by multiples, not percent. That sensitivity *is* the finding.

### 6.3 The multipliers that make a fleet expensive

Three multipliers, each computed. All ILLUSTRATIVE, each shown as a chain so you can substitute.

**(i) The fan-out multiplier.** A session that spawns sub-agents multiplies its own cost, because each sub-agent is an independent context that must be prefilled and decoded, and the parent then *reads* their outputs — paying input cost on the aggregated result too.

| Case | Steps | Cost at $0.0125/step | Multiplier vs median session |
|---|---|---|---|
| Median session, no fan-out | 8 | $0.0997 | 1.0× |
| Session + 3 sub-agents × 4 steps | 8 + 12 = 20 | $0.2492 | **2.5×** |
| Session + 10 sub-agents × 6 steps | 8 + 60 = 68 | $0.8473 | **8.5×** |
| Sub-agents that themselves fan out (depth 2, branching 3) | 8 + 12 + 36 = 56 | $0.6976 | 7.0× (plus parent-read costs on every returned result) |

Note the mechanism: **fan-out is multiplicative in depth and additive in breadth.** A depth-2 tree with branching factor 3 costs 7× a linear session of the same length; the *bounded-breadth* mitigation (cap sub-agents per session, cap depth, cap total steps) is a design decision, and it is the single most effective cost guard in a multi-agent fleet. Topology itself is owned by [hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md) — here it is purely an arithmetic multiplier.

**(ii) The retry multiplier.** Expected cost = base cost × expected attempts. With per-attempt transient failure probability *p* and a retry budget of *k* extra attempts: **amplification = 1 + p + p² + … + p^k**.

| Per-call failure rate *p* | Extra attempts *k* | Amplification | Comment |
|---|---|---|---|
| 5% | 1 | 1.05× | Normal day |
| 5% | 2 | 1.10× | Normal day, auto-retry on |
| 15% | 2 | 1.30× | Degraded provider — and note the failure rate and the retry count rise *together* |
| 30% | 3 | 1.42× | Incident: you are paying 42% extra for work that mostly fails |
| 50% | 5 | 1.97× | ≈2× spend, near-zero throughput — the worst quadrant |

The compound trap: **retries multiply cost at exactly the moment throughput collapses**, so cost *per completed task* explodes much faster than total spend. If 50% of attempts fail and each call is retried up to 5 times, you pay 1.97× for complete-work throughput that has fallen by half: cost per *successful* task rises ~3.9×. That is the arithmetic behind "the month the bill tripled and nothing shipped".

**(iii) The long tail.** The p99 session costs 12.9× the median in the model above, and the tail is structural rather than accidental: harder tasks take more steps, more steps mean more context, more context means more expensive steps — *and* more steps mean more chances to hit a retry, a fan-out, or a human escalation. The tail also compounds:

| Effect | Cost impact on the p99 session | Status |
|---|---|---|
| 40 steps instead of 8 | 5× the steps | ILLUSTRATIVE arithmetic |
| Average context 60,000 instead of 18,000 (≥3.3× per step) | $0.0322 vs $0.0125 per call | ILLUSTRATIVE arithmetic |
| Retries at 15%, 2 extra | ×1.30 | ILLUSTRATIVE arithmetic |
| Fan-out on 3 sub-agents | ×2.5 on the parent's step cost | ILLUSTRATIVE arithmetic |
| **Combined tail multiplier vs the median-session baseline** | **≈16.8×** | ILLUSTRATIVE arithmetic |

**Design consequence:** a percentile-based budget is the only budget that survives a fleet. A *mean* budget is violated by the tail; a *max* budget is violated by the fan-out; a **per-session cap with a hard stop** plus a **fleet-level spend governor** is the pairing that actually holds (§6.5). And because the tail is where cost lives, **the tail must be visible per session** — not averaged away — which is an observability requirement owned by [agentops_guide.md](agentops_guide.md) §3 and §6.

### 6.4 Cost per task through cost per million tasks

Unit chains, so the reader can rebuild with their own numbers:

- **Cost per model call** = (uncached input tokens × input rate) + (cached input tokens × cache-read rate) + (output tokens × output rate) + tool/retrieval cost per call.
- **Cost per session** = Σ calls + Σ tool costs + Σ retrieval + Σ sub-agent costs + retry amplification.
- **Cost per task** = cost per session for *successful* sessions ÷ success rate. (A 70%-success workflow at $0.10/session costs $0.143 per *completed* task. Teams that track per-session spend and celebrate "we cut cost 20%" while success fell 20% have cut nothing.)
- **Cost per million tasks** = cost per task × 1e6. At the ILLUSTRATIVE $0.119 blended, that is **~$119,000 per million tasks** — order-of-magnitude, not a quote.
- **Cost per business outcome** — the only number a budget owner cares about. If 1 in 40 tasks reaches a human-valuable outcome, cost per outcome is 40× cost per task. This conversion is where most agent business cases actually fail, and it is the reason this guide returns repeatedly to *completion* rates rather than token rates.

### 6.5 The governance layer is a design requirement, not an operational afterthought

Cost controls (caps, caching, routing) are owned by [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) §7 — this section states only the *fleet-level* requirements that must be in the architecture on day one, because they cannot be retrofitted after the first runaway month:

| Control | Where it must live | Why it cannot be an afterthought |
|---|---|---|
| **Per-session cap** (tokens and/or currency) with hard stop | In the agent's execution loop, before the next model call | A session that exceeds its cap must stop *mid-flight*, which requires a check in the loop, not a report afterwards |
| **Per-tenant / per-team budget** | In the shared gateway or admission layer | Multiple teams on one provider account share one bill; without per-tenant budgets the first runaway team spends everyone's quota |
| **Fleet-level spend governor** | Cross-cutting, above the gateway | Detects the tail-session and fan-out events that no single session-level cap catches |
| **Kill switch** (per agent, per tenant, per fleet) | Triggers wired to a human-accessible control | The only answer to "the fleet is spending at 10× and we do not know why" |
| **Admission-time budget check** | In the request path | Rejecting an over-budget request costs nothing; failing it at step 30 costs full price |
| **Attribution identifier on every call** | Propagated into every model/tool call | Retrofitting correlation across a fan-out is a multi-quarter data engineering project |

⚠ The honest caveat: a spend governor is a *rate* control, not an accounting control. Provider bills arrive after the fact; the governor only bounds the *future*, using your own metering. If your metering under-counts (a forgotten tool, an unmetered sub-agent), your governor under-protects. Instrument the metering itself and reconcile it against provider invoices monthly — a discipline that [finops_guide.md](../finops_guide.md) covers in general form.

---

## 7. Where the Money Actually Goes

Kept short deliberately: this is a **hand-off**, not a second cost-control chapter. The controls are [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) §7; the observability, attribution mechanics and FinOps operations are [agentops_guide.md](agentops_guide.md) §6 (cost operations, budgets, attribution) and §3 (observability). What belongs here is only the fleet-shaped part of the question.

**The attribution problem is a fan-out problem.** At ten sessions, "who spent this?" has one answer: the same person who ran the agent. At ten thousand, a single business task fans out across a parent session, N sub-agents, three models, a retrieval index, a hosted search tool and an internal pricing service — and the provider sees only a stream of calls against one account. The platform's own bill is therefore *not* the per-team answer: it is a single number for a population of tasks, and it cannot be decomposed without an identifier that was carried through every hop. Two consequences:

1. **The correlation identifier is the whole game.** One identifier minted at admission (task or session id), propagated into every model call, tool call, retrieval and sub-agent, and emitted on every metering event. This must exist before scale, because back-filling correlation across a fleet that did not carry it is not a query you can write — it is a redesign. Sub-agents must *inherit*, not mint, unless the hierarchy is also recorded.
2. **What a finance function will ask for** is not what an engineer wants to show. Expect: cost per task (and per *successful* task) by team/product/tenant; cost trend against volume, so growth in spend can be separated from growth in usage; the share of spend in the tail; spend against budget with a forecast for month-end; the cost of retries and failures as a distinct line; and the cost of the *unattributable* residual. That last one is the tell — a fleet with a large unattributable residual has an attribution design problem, not an accounting problem.

Practical minimum: a per-call metering event carrying (correlation id, tenant, agent version, model, cached/uncached input tokens, output tokens, tool id, latency, retry count, terminal status). Everything downstream — dashboards, budgets, budgets-by-team, anomaly detection — is a query against that event stream. See [agentops_guide.md](agentops_guide.md) §6 for the operations, and [agent_versioning_guide.md](agent_versioning_guide.md) for why the agent *version* must be on the event (a cost regression is frequently a version regression).

---

## 8. The Failure Modes That Only Exist at Volume

Scope note, stated as a boundary: the general agent failure taxonomy (bad tool arguments, hallucinated plans, loops, compounding per-step error) is owned by [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) and [agents_work_fall_apart_guide.md](agents_work_fall_apart_guide.md). This section owns only failures that **cannot exist in a single-agent deployment** — they are produced by the *population* of sessions interacting through shared infrastructure. Where a per-session trigger also exists, I have said so; the *fleet* dynamics are what is new. Each entry is given as mechanism → invisibility at demo scale → detection signal → mitigation, with mitigations labelled **[D]** design decision or **[T]** configuration toggle.

### 8.1 Retry storms

**Mechanism.** A shared dependency degrades (provider latency spike, gateway restart, tool endpoint slow). Many sessions fail at the same time. Each client applies its retry policy — typically exponential backoff with a fixed base and, if unlucky, no jitter — so the retries are *correlated*: the population that failed together retries together. The attempt count per original request is 1 + p + p² + …, so the offered load *rises* while capacity *falls*. If the retry budget exceeds the capacity deficit, the retries keep the dependency saturated and the system does not recover — the load is sustained by the recovery mechanism (§5.5). Deeper variant: the retry storm passes *through* the model gateway into a downstream tool that has no rate limiting at all, so you convert a provider hiccup into a database incident.

**Why it is invisible at demo scale.** With one session, "retry twice" is 2 extra calls and the system heals because there is no coupling. There is no synchronisation to observe, and the loop cannot be self-sustaining because the population is one. Demo traffic also never coincides with a degradation event big enough to make retries visible.

**Detection signal.** Retry *rate* (attempts per original request) diverging from retry *count*; the ratio of total attempts to distinct correlation ids rising; a step-down in success rate accompanied by a step-*up* in offered request rate; p99 latency rising while throughput falls; `retry-after` values clustering at the same second.

**Mitigation.** Exponential backoff **with full jitter** [T]; a fleet-wide retry budget (e.g. retries may not exceed X% of total calls in a rolling window) [D]; **retry only what is safe to retry** — non-idempotent calls must not be retried without an idempotency key, which is a design property ([durable_ai_agent_workflows_guide.md](../durable_ai_agent_workflows_guide.md) owns idempotency and compensation) [D]; circuit breakers per dependency with a half-open probe [D]; and a bulkhead so that a retry storm against one dependency cannot consume the concurrency of unrelated traffic [D].

### 8.2 Thundering herds

**Mechanism.** A large population of sessions starts, resumes or refreshes *simultaneously*, because they share a trigger: a cron schedule, a queue drain after an outage, a cache TTL that expires globally, a deployment that restarts workers, or a nightly batch releasing 10,000 tasks at once. The mechanisms are all forms of **synchronisation**: identical schedules, identical TTLs, identical restart times, or a queue that accumulated 10,000 items and is then released at full speed. The load profile is a step function, and the fleet's capacity was sized for an average — so the step exceeds capacity, causing partial failures, whose retries add a second synchronised hump.

**Why it is invisible at demo scale.** With ten sessions, a cron tick creates ten requests, which is indistinguishable from noise. The synchronisation only becomes harmful when the same instant holds thousands of sessions; the demo never reaches the regime where the *shape* of the load profile matters more than its average.

**Detection signal.** Request rate with a sharp, repeatable periodicity (minute/hour/day boundaries); concurrency far above the trailing mean at predictable instants; cache-hit rate collapsing on a fixed period (cold-cache stampede); queue depth spiking then draining in a sawtooth.

**Mitigation.** **Jitter everything that can be scheduled** — start times, polling intervals, cache TTLs (spread expiry ±10% at minimum) [T]; **queue-drain rate limiting** so a backlog releases at a controlled rate rather than at once [D]; **smooth rollout** of scheduled work (a "scheduler with a sleep distribution", not a cron tick) [D]; warm the cache deliberately before a scheduled burst [T]; and separate *classes* of traffic so interactive work is not starved by a batch herd [D].

### 8.3 Cascading fan-out

**Mechanism.** A parent session spawns sub-agents; sub-agents spawn sub-agents; each level multiplies both *cost* and the number of **simultaneous** demands on the serving layer and tools. Concurrency multiplies in the same instant as cost: a depth-3, branching-3 tree puts up to 27 leaves in flight where one session was planned. Fan-out is multiplicative in depth and additive in breadth, so a single unbounded fan-out point — one agent that decides how many helpers to spawn — can generate a concurrency spike of arbitrary size from arbitrary small demand. Worse, the fan-out is often *triggered by difficulty*, which is correlated across sessions: a hard input (a large document, an outage-affected account) makes many parents fan out at once.

**Why it is invisible at demo scale.** With one parent, "spawn as many helpers as you need" works beautifully — the model decides 4, and 4 is fine. The unbounded decision is never tested against a population, so the absence of a cap is not observable. Depth 3 with branching 3 is 27 sub-agents: trivial in a demo, a 27× concurrency multiplier at fleet scale.

**Detection signal.** Sub-agents per session (distribution, not mean); depth distribution; concurrent-in-flight count vs planned concurrency; serving-layer queue depth correlated with the *sub-agent* population rather than with incoming sessions; tool call rate per parent session; sudden step changes in tokens-per-task.

**Mitigation.** **Hard caps on sub-agents per session, on depth, and on total steps — enforced by the runtime, not requested in the prompt** [D]; admission control applied to *sub-agent* spawns exactly as to sessions [D]; a per-session "work budget" (tokens, currency, wall clock) that the fan-out must share [D]; and a topology chosen for the workload rather than for the demo ([hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md) owns topology) [D]. The cap is a *design decision* because it must be enforced below the model's control — a prompt instruction is a suggestion, and the failure mode is precisely the case where the model ignores suggestions.

### 8.4 Cost blowout

**Mechanism.** Not "an expensive session" but the interaction of three multipliers acting at once on a population: the long tail (many sessions in the expensive region at the same time), fan-out (each tail session also spawning helpers) and retries (rising exactly when the tail is largest). The arithmetic in §6 gives ≈16.8× on a tail session versus a median-session baseline. Because the tail is *traffic-dependent* — hard inputs arrive in batches, incidents degrade success rates — cost and failure correlate: the fleet spends the most in the minutes it accomplishes the least. There is also an asymmetry: cost is bounded by *only* the budget you enforce, because nothing in the request path naturally stops a session that keeps taking steps; the model will keep going as long as it is given a turn.

**Why it is invisible at demo scale.** A demo session is cheap because it is short, has a warm cache, small context, no fan-out and no retries. The cost curve looks linear and benign because none of the three multipliers has "switched on". Nobody discovers that a session has no ceiling until a production session finds the ceiling's absence.

**Detection signal.** Cost per *successful* task rising while total spend is flat (a quality regression masquerading as a cost problem); the share of spend attributable to the top 1% of sessions; retry-attributable spend as a distinct line; spend per tenant diverging from spend per tenant's *volume*; tokens-per-task p99 climbing; the number of sessions hitting their cap.

**Mitigation.** Per-session hard cap with a mid-flight stop [D]; per-tenant budget with admission-time check [D]; fleet-level spend governor with an alert *and* an automatic throttle [D]; kill switch [D]; and a **cost anomaly detector that alerts on rate-of-change, not on absolute spend** — because absolute thresholds arrive after the money is gone ([agentops_guide.md](agentops_guide.md) §6 owns the monitoring; the requirement that it exist is a design decision here) [D]. Note that the toggles (caching on, model routing on, batch API for deferrable work) reduce the *base* but do not stop blowout; only ceilings stop blowout.

### 8.5 Unbounded context growth

**Mechanism.** Within a session, context accumulates: every tool result, every sub-agent output, every retrieved document is appended and re-sent on the next call. Because cost per call is linear in input tokens and the number of calls grows too, session cost is roughly **quadratic in the number of steps** for a session whose context grows linearly. The same growth is simultaneously: (i) a serving-layer memory problem (KV cache for in-flight requests, §4.2), (ii) a latency problem (prefill time grows with input), (iii) a *quality* problem (irrelevant context degrades reasoning), and (iv) — at fleet scale — a **concurrency** problem, because longer contexts mean fewer concurrent requests fit in the same GPU memory. So context growth silently reduces the fleet's capacity ceiling over time. Compaction helps cost but can *break prefix caching* (OpenAI's own documentation notes that compaction changes the prefix and may reduce cache reuse — VERIFIED 2026-09-22), which is a direct example of two mitigations interacting.

**Why it is invisible at demo scale.** In a 5-step demo, quadratic growth is invisible: 5 steps is small, and the demo ends before the context is large. There is no cross-session memory pressure to reveal that context is a shared resource. Context engineering for quality is a different (and well-covered) discipline — [context_engineering_guide.md](context_engineering_guide.md), [agent_runtime_cache_design_guide.md](agent_runtime_cache_design_guide.md) — and the fleet consequence is what is new here.

**Detection signal.** Context tokens per call plotted against step index (should be flat; linear growth is the alarm); input tokens per task p95 vs p50 ratio; TTFT rising over the life of a session; serving-layer batch occupancy falling while request rate is constant; cache-hit rate decaying across a long session.

**Mitigation.** A **context budget** with an enforced ceiling per session, not a hope [D]; explicit context *management* policy (summarise, drop tool outputs older than N steps, retrieve instead of accumulate) [D]; windowed/compacted context with the cache consequences measured rather than assumed [D] (see [agent_runtime_cache_design_guide.md](agent_runtime_cache_design_guide.md) for cache design, [context_engineering_guide.md](context_engineering_guide.md) for the technique); step-count ceiling per session [T once the ceiling exists]; and a serving-side max-context guard so a runaway session is rejected rather than admitted [T].

### 8.6 State collision between concurrent sessions on the same resource

**Mechanism.** Two or more sessions operate on the same underlying resource — the same customer record, the same file, the same account, the same ticket, the same scratch workspace. Each reads state, reasons about it, then writes. Because agent steps are *slow* (seconds to minutes), the read-modify-write window is enormous compared with a normal API call, so lost updates become common rather than rare. Unlike a database transaction, an agent's "transaction" spans **model calls**, which means it also spans retries, human pauses and provider latency; and it is often non-idempotent (the write is an email, a payment, a status change). At fleet scale, the birthday-problem effect makes collisions certain: with thousands of sessions over hundreds of thousands of resources, the probability that two sessions target the same hot resource is not small — and "hot" resources (a large customer, a shared account, a popular document) collide constantly. A second-order collision: two sessions in the *same* tenant both trigger a fan-out that writes to the same downstream system (this is the "same resource" case that hides inside fan-out).

**Why it is invisible at demo scale.** One session cannot collide with itself. A demo runs sequentially against resources that are not hot. Even a small pilot with 10 sessions has a collision probability near zero, so the absence of locking looks like a design choice rather than a gap.

**Detection signal.** Lost updates (a write overwritten; an audit trail showing two writers between two reads); optimistic-concurrency conflicts (`if-match`/version failures) spiking with concurrency; deadlock or lock-timeout events; duplicate side effects (two emails, two payments, two tickets for one incident); resources with anomalously high session counts (hot keys).

**Mitigation.** Optimistic concurrency with version checks *plus* a retry that re-reads and re-plans — cheap when conflicts are rare, disastrous when they are not, so measure conflict rate before choosing [D]; pessimistic locking or a lease for genuinely shared mutable resources, with a **lease expiry** so a crashed session cannot hold a lock forever [D]; idempotency keys on every non-idempotent side effect (**precondition**, see §9) [D]; a **per-resource concurrency limit** (one writer per resource) [D]; and partitioning or sharding of hot resources so collisions become rare rather than merely handled [D]. Section 9 develops the choice; the general mechanism here is that agent steps are long, so the collision window is wide.

### 8.7 The pattern behind all six

Every failure mode above has the same shape: **sessions are not independent, because they share something** — a provider quota, a schedule, a cache, a memory budget, a model's decision to fan out, a resource row. Independence is the assumption that makes "multiply by N" arithmetic look safe; shared resources are what breaks it. Hence the design rule that generalises across all six: **for every shared resource the fleet touches, name the ceiling, instrument the occupancy, and control admission against it.** What is a design decision in every case is the *ceiling and the control point*; what is a toggle is only the parameter.

---

## 9. Concurrency Control Over Shared Resources

### 9.1 What happens when a fleet meets one record

The mechanics of §8.6, sharpened for planning. A single record (a customer profile, an account, a financial instrument, a document) is touched by session A from the onboarding workflow and session B from the service workflow, at overlapping times. Each will: read the record, put part of it in a context, reason, call tools that may themselves mutate state, and finally write. Over that window the record may be stale in both contexts. Nothing in the model's behaviour prevents this — the model cannot see the other session. So the correctness property must come from the *substrate*, not the agent.

### 9.2 Optimistic versus pessimistic, with the agent-specific consequences

| Approach | How it works | Agent-specific consequence | When it is right |
|---|---|---|---|
| **Optimistic** (version/ETag check, abort-and-retry on conflict) | Read version, plan, write only if version unchanged | The *re-plan* is expensive: a conflict costs a whole model call plus context, so conflict-driven retries multiply cost (this is a cost problem, not just a correctness one) | Conflicts rare; work expensive to serialise; writes small |
| **Pessimistic** (lock or lease held across the session) | Acquire exclusive access before read; hold across the model calls | An agent turn can take minutes, so the lock is held for minutes — lock contention at fleet scale is *guaranteed*, and a crashed session leaks the lock without an expiry | Conflicts common on hot resources; correctness critical; the critical section can be made short |
| **Lease / time-boxed ownership** | Exclusive, but expiring and renewable; owner must renew or lose it | Fits agents better than a plain lock: covers the crash case, bounds the contention, forces an explicit "am I still the owner?" check before writing | Default for agent fleets that need mutual exclusion |
| **Serialise by scheduling** (single-writer per resource class, queue) | The resource is served by one worker consuming a queue | Removes the conflict entirely at the cost of throughput on hot resources — a hot record becomes a queue | High-value, low-volume, regulated resources |

The design point that matters at volume: **choose the mechanism per resource class, not per fleet.** A fleet needs at least three classes — *freely shareable* (read-only corpora, no control needed), *optimistically guarded* (most business records), and *serialised* (money movement, regulatory filings, anything where a duplicate is an incident). Applying the strictest mechanism everywhere destroys throughput; applying the loosest everywhere produces duplicates.

### 9.3 Locking, leases and the fairness question

At volume, locking raises a question that single-agent systems never face: **who gets the lock next, and is that fair?** A naive lock is unfair — a hot resource can starve a tenant indefinitely while other sessions repeatedly win the race. Concrete requirements:

- **Every lease expires and is renewable.** A session that dies mid-flight must not hold a resource for the rest of the day. Expiry must be shorter than the maximum tolerable stall, and the holder must re-check ownership immediately before the write.
- **Waiting is bounded and visible.** A lock wait with no timeout becomes an invisible queue occupying session slots (§3.4). Wait time must be a metric and a timeout must exist.
- **Starvation must be prevented at the *scheduler*, not at the lock.** Fair queueing, per-tenant isolation, or a ticket order. Otherwise a large tenant's volume structurally starves a small one, which becomes an organisational problem (§12) as much as a technical one.
- **Lock scope must not span human steps.** A session waiting for human approval must release its lease (park it), or the resource is held for hours by a session that is doing nothing.
- **Lock acquisition itself can stampede.** A popular resource with many readers waiting to write turns a lock into a thundering herd; spread acquisition with jitter and prefer optimistic paths for read-mostly resources.

### 9.4 Idempotency as the precondition for any retry at scale

State it as an invariant: **no retry, no queue, no failover, and no at-least-once delivery is safe on a non-idempotent operation.** Everything in §5.5 and §8.1 assumed the ability to retry; everything in §5.6 assumed the ability to wait and re-run. Without idempotency, the retry *is* the incident — the duplicated payment, the double-sent notice, the file opened twice. This guide therefore treats idempotency as a **precondition** rather than a technique: durable execution, idempotency keys, exactly-once *effects* over at-least-once delivery, sagas and compensation are owned in full by [durable_ai_agent_workflows_guide.md](../durable_ai_agent_workflows_guide.md) (§14 sagas/compensation, §15 fan-out/fan-in, §18 checkpointed long-running agents, §23 delivery semantics). Read that guide before designing retry policy here; the fleet-scale corollaries are:

1. **The idempotency key must carry the correlation identifier** (§7), not a per-attempt UUID, or a retry after a failover will not be recognised as a duplicate.
2. **The window must outlive the retry window.** An idempotency cache with a TTL shorter than the longest retry horizon is decorative.
3. **A "successful" step whose side effect is unconfirmed is unknown, not failed** — unknown outcomes must be *reconciled*, which at fleet scale means an automated reconciliation path, because a human cannot chase thousands of unknowns.

### 9.5 The regulated-resource case

For regulated resources the control must be *evidenced*, not merely implemented: who held which resource, when, under what authority, and what was written. The banking content in this repository owns the substance: [operational_resilience_framework_guide.md](../../banking/operational_resilience_framework_guide.md) for the operational-resilience and critical-operation lens, [financial_risk_compliance_systems_guide.md](../../banking/financial_risk_compliance_systems_guide.md) for the risk and compliance control context, and [mas_regulations_guidelines_guide.md](../../banking/mas_regulations_guidelines_guide.md) for the supervisory expectations a bank must map its agent controls onto. §13 states what volume adds to those obligations; it does not restate them.

---

## 10. Capacity Planning With an Honest Model

### 10.1 The arithmetic, with assumptions stated

The purpose of an explicit model is not to be right; it is to expose which assumption dominates, so you know what to measure. Assumptions are listed first because an unstated assumption is the one that breaks the plan.

**Assumptions (all ILLUSTRATIVE):** business volume V = 20,000 tasks/day; sessions are interactive, arriving over a 10-hour working window; average session duration D = 40 minutes (ILLUSTRATIVE); peak-to-average arrival ratio B = 3 (ILLUSTRATIVE burst factor); session shape as §6 (8 calls, ~18,000 input tokens and 400 output tokens per call); average concurrency factor C_f = 0.6, i.e. at any instant only 60% of "in flight" sessions are actually making a model call (the rest are waiting on tools, humans or locks — ILLUSTRATIVE).

| Step | Arithmetic | Result |
|---|---|---|
| Average arrival rate | 20,000 tasks ÷ 10 h | 2,000 tasks/h ≈ 33 tasks/min |
| Peak arrival rate | 33 × 3 (burst factor) | **100 tasks/min** |
| Concurrent sessions at peak (Little's Law) | 100 tasks/min × 40 min | **4,000 concurrent sessions** |
| Sessions actually calling the model at an instant | 4,000 × 0.6 | 2,400 active calls |
| Model calls per minute | 2,400 calls ÷ (40 min ÷ 8 calls = 5 min per call cycle) | ~480 calls/min |
| Output token demand | 480 calls/min × 400 tokens | 192,000 output tokens/min |
| Input token demand (cache-aware, 15% uncached) | 480 × 2,700 | 1,296,000 uncached input tokens/min |
| Input token demand (no cache) | 480 × 18,000 | 8,640,000 input tokens/min |
| Required quota (worst dimension) | min of RPM/ITPM/OTPM headroom | **~200K OTPM, ~1.3M uncached ITPM, ~480 RPM** |
| Quota with 50% headroom | ×1.5 | ~300K OTPM, ~1.9M ITPM, ~720 RPM |

Compare that final row against §5.3: the read Anthropic figures for the tier shown were 1,000 RPM / 2,000,000 ITPM / 400,000 OTPM. The fleet's **OTPM demand with no headroom is 192K against a 400K limit — comfortable. With the 50% headroom rule it is 288K — still inside. But the no-cache ITPM demand is 4.3× the limit.** One assumption — whether the prefix cache is warm — decides whether this fleet fits inside one quota at all. That is the honest output of a capacity model: not a number, but a ranked list of which assumption you must go and measure.

**Headroom and burst factor are different things.** Headroom absorbs *component* failure and quota variance (a region degrades, a model gets slower). Burst factor absorbs *demand* variance (Monday morning, month-end, an incident that triggers a re-run of last week's work). Sizing for the mean and adding "30% headroom" fails on both: a 3× burst factor is not a 30% margin, and it is usually the *burst* that breaks the fleet, not the average. Also budget the **recovery burst** — after an outage, the accumulated backlog drains at whatever rate you allow, so the post-incident load is a design parameter, not an accident (§8.2).

### 10.2 Where the model breaks

State this plainly: the model above is a **lie in at least five places**, and each one can dominate the answer.

1. **Work per request is variable and unbounded.** "8 calls per session" is an average over a heavy tail (p99 might be 40). Since concurrency is driven by *duration*, and duration is driven by steps, the concurrency estimate inherits the tail: if 1% of sessions run 10× as long, they add ~9% to concurrency on average but 100× to a *single* session's cost. Sizing for the mean is sizing for the median workload and nothing else.
2. **Sessions fan out unpredictably.** A sub-agent tree (depth 2, branching 3) multiplies in-flight demand by up to 7× at the moment it happens. The model assumed one call per session-step; a fan-out session makes several *simultaneous* calls. Concurrent session count is therefore not a sufficient demand statistic — you need **concurrent model calls**, which is what actually queues at the serving layer.
3. **Latency per step is not stationary.** Provider latency, context length and cache-hit rate all drift; a session's own context growth makes its later steps slower than its first (§8.5). A model built from median latency systematically underestimates capacity *at the end of long sessions*, which is exactly when the queue is fullest.
4. **Retries are load.** §6's amplification maths says a 15% failure rate with 2 extra attempts raises *offered* calls by 30% at the moment throughput is falling. Capacity models almost never include retry load; incidents are exactly when it appears.
5. **The bottleneck is often not the model at all.** §3.4: a slow tool, a lock, or a connection pool can be the binding constraint while the model endpoint looks idle. A capacity model that only counts tokens will confidently predict the wrong ceiling.

### 10.3 The practical alternative: capacity where the units are stable, admission where they are not

The reason the model breaks is that the *session layer* has unstable units (variable steps, variable latency, variable fan-out) and the *serving layer* has relatively stable ones (tokens per second per unit of hardware, tokens per minute per quota). So split the problem:

| Layer | Unit of capacity | How to plan it | Control |
|---|---|---|---|
| **Serving / quota** | Tokens per minute, requests per minute, PTUs, GPU memory | Arithmetic is legitimate here: budget OTPM, ITPM and RPM against verified limits, monitor those three, plan headroom against the *limit* | Rate governor, batching, caching, batch API for deferrable work, quota increase requests |
| **Session / orchestration** | Concurrent sessions and concurrent tool calls | Arithmetic is *not* legitimate: sessions are variable and unbounded. Replace a target concurrency with an **admission policy** | Admission control, per-tenant and per-session budgets, bounded queues, shed/wait policy |

Concretely, the design is a **two-tier limiter**: a *session-level* admission controller that decides whether to accept work (against the fleet's own concurrency ceiling and each tenant's fair share), and a *call-level* rate governor that keeps offered model traffic inside the quota. The session limiter protects the fleet from itself; the call limiter protects the fleet from the provider. Neither substitutes for the other, and building only one is the standard mistake.

### 10.4 Explicitly where I am guessing

⚠ The numbers in §10.1 are ILLUSTRATIVE and the model's structure is the deliverable. ⚠ **No serving framework's own documentation states a concurrency-per-GPU or throughput figure** — I read vLLM's README, HF TGI's docs index, SGLang's landing page, TensorRT-LLM's docs tree and Triton's architecture page (§4.1) and found batching *mechanics* asserted and no capacity *number*. The frameworks publish performance studies on their blogs and in papers, at their own hardware and model configurations, which are not your workload; I deliberately did not import one. **There is no public benchmark for "agents at scale"**; any figure presented as one should be treated as either a vendor best case or an advertisement. The correct posture is to measure your own: run a *bounded* load test (§11) and record tokens/second and concurrent-call ceilings for your own context distribution.

---

## 11. Testing and Validating at Scale

### 11.1 Why a production trace cannot simply be replayed at volume

The instinct is to capture production traces and replay them at 10× to see what breaks. It fails for four compounding reasons, each of which is a property of agent sessions rather than of testing tools:

1. **Sessions are long-running.** A trace of a 40-minute session cannot be replayed in a 40-second test without either distorting latency (and therefore the concurrency it produces) or taking 40 minutes per session. Replay at accelerated speed measures a different system.
2. **Sessions are expensive.** Replaying 10,000 production sessions costs real tokens at real prices — this is a load test with a bill attached, and the bill scales with the fidelity of the test. Unlike CPU load tests, you cannot run a million iterations for free.
3. **Sessions are non-idempotent.** A trace that sent emails, moved money or wrote files cannot be replayed faithfully: either you stub the side effects (and then you are not testing the real path, including the locks and the collisions) or you execute them (and you have just duplicated production side effects). §9's idempotency precondition applies to your *tests*, not only your production.
4. **Sessions are nondeterministic.** The same input produces different tool sequences, different step counts and different fan-out on every run. So the replay is not a repeat of the production workload — it is *a* workload with a similar distribution. Comparing "before" and "after" results on nondeterministic workloads without controlling for variance produces confident nonsense.

This is precisely the reproducibility problem that [deterministic_engineering_guide.md](../deterministic_engineering_guide.md) **§5 (deterministic simulation testing)** and the broader deterministic-simulation discipline own: recorded/replayed, time-controlled, dependency-simulated execution so that a run is *repeatable*. Do not re-derive it here — read §5 there, and read the deterministic-simulation testing guide for the tooling. The fleet-level corollary is what this section adds: **reproducibility is what makes a load test bounded in cost**, because a deterministic simulated run does not call the real provider (and therefore does not consume quota or money) for the paths that are simulated.

### 11.2 What can be measured versus what cannot

| Measurable under load (and worth measuring) | Not measurable under load (do not pretend) |
|---|---|
| Serving-layer behaviour: time-to-first-token, tokens/s, queue depth, batch occupancy, preemption | **End-to-end task quality under contention** — whether the agent still gets the right answer when the fleet is saturated. Contention changes timing, timing changes tool sequences, and quality becomes a different question |
| The queueing system: admission latency, wait time distribution, drop/shed rates, queue age | Whether a *specific* production session would have succeeded (nondeterminism again) |
| Tool endpoints: latency percentiles, error rates, connection-pool saturation, timeout rates | The long-tail cost distribution from a short test — the tail takes time to appear |
| Cost per call: tokens in/out per call by model, cache hit rate, retry rate | Cross-tenant fairness under adversarial demand (you can measure it only if you synthesise that demand) |
| Your limiters: does the rate governor actually cap at the intended rate; does admission control fire; do caps stop sessions mid-flight | Retrospective reconstruction of a six-month-old session under a logging configuration you have since changed (§13) |

The asymmetry is the point: **the substrate (serving, queueing, tools, cost per call) is measurable; the emergent behaviour (task quality under contention) is not.** So test the substrate directly and test task quality *separately*, at low concurrency, with the deterministic harness — and accept that the combination is an extrapolation, not a measurement.

### 11.3 Bounding the cost of the load test itself

Treat the load test as a production workload with a budget, because that is what it is:

- **Bound the population, not the duration.** Decide the maximum token spend for the test *first*, then derive how many sessions that buys: at the ILLUSTRATIVE $0.12/task of §6, a $2,000 budget buys ~16,000 sessions. Budget first; scale second.
- **Use simulated providers for the components under test that are not the provider.** If you are testing your queueing, admission control and cost accounting, replace the model with a deterministic simulator (see [deterministic_engineering_guide.md](../deterministic_engineering_guide.md) §5). If you are testing your quota arithmetic, you need the real endpoint — then use the cheapest adequate model and the shortest adequate context, and note that both change the result.
- **Use batch APIs for what does not need to be synchronous.** Verified 22 Sep 2026: OpenAI's Batch API limits a batch to 50,000 requests and 200 MB, caps batch creation at 2,000/hour, and states that batch rate limits are a **separate pool** that does not consume standard per-model limits — so batch work cannot be used to test synchronous-path capacity, but also cannot compete with it. Gemini's batch tier is priced at 50% of standard (verified); Bedrock offers batch inference at a 50% lower price than on-demand (verified). And one agent-specific catch: **Bedrock batch inference does not support tool calling or structured output** and is not supported for provisioned models (verified 22 Sep 2026) — so agent sessions with tool loops *cannot* be replayed through batch at all. That single constraint invalidates a common "just batch the load test" plan for tool-using agents.
- **Stage it:** single session → ten → hundred → thousand, watching the *substrate* metrics at each stage, and stopping at the first stage where a limiter fires unexpectedly. This is the cheapest possible way to find the binding constraint, and it is almost always cheaper than a big test that fails for an uninteresting reason.

### 11.4 Chaos and fault injection at the fleet level

Per-session fault injection ("what if the tool returns garbage?") is not a fleet problem and is covered elsewhere. The fleet-level chaos questions are population questions:

- **What happens to 4,000 concurrent sessions when the provider returns 429 for 30 seconds?** Measure: does the retry policy amplify (§5.5), do the breakers open, does the queue recover, and what did those 30 seconds cost?
- **What happens when the cache goes cold?** Flush the prefix cache (or wait out its TTL) and watch cost per task and ITPM consumption jump together. This tests §6's largest lever under the worst conditions.
- **What happens when one tenant's work goes exponential?** Inject a workload that fans out without bound and confirm the *caps*, not the dashboards, stop it.
- **What happens when a dependency is slow rather than down?** Slow is the harder case: sessions hold slots, concurrency fills, throughput collapses while error rates look normal.
- **What happens when you restart the orchestration layer with 4,000 sessions in flight?** This tests durable execution, checkpointing and idempotency together ([durable_ai_agent_workflows_guide.md](../durable_ai_agent_workflows_guide.md)), and is the single most valuable fleet chaos experiment.

Experiment design, failure taxonomy, tooling and safety are owned by [chaos_engineering_guide.md](../chaos_engineering_guide.md) §4–§8; use those directly rather than inventing a house variant, and take its observability prerequisite (§7) seriously — a fleet chaos experiment without per-session cost and concurrency telemetry produces an outage you cannot explain.

---

## 12. The Organisational Dimension at Volume

Short and pointed, because the answer is organisational and the mechanisms are owned elsewhere.

**Several teams, one pool.** The moment two teams share a provider account or an inference deployment, they share: the quota, the spend cap, the latency (each other's load), the incident (one team's retry storm is everyone's outage), and — most consequentially — the *noisy-neighbour* problem, where one team's volume degrades another's service without any rule being broken. This is not a technology problem with a technology answer; it is a capacity-allocation problem that needs per-tenant quotas, per-tenant budgets, and a fair-share scheduler. Inngest's own documentation names the mechanism precisely, describing concurrency keys as creating separate queue groups that provide "best-effort fairness in multi-tenant systems" and calling out the noisy-neighbour case — VERIFIED 22 Sep 2026 — and its warning that concurrency limits *steps*, not runs, is exactly the distinction a platform team must get right when allocating capacity.

**The platform team as capacity broker.** At scale, the platform team stops being a service provider and becomes a **broker of a scarce resource**: it decides the split of quota across business lines, admits or rejects new workloads against remaining headroom, owns the fleet-wide limiter and the kill switch, negotiates quota increases with providers, and publishes the cost-per-task rate that internal teams are charged. The broker role only works if it has (i) an authoritative view of remaining headroom, (ii) the authority to refuse work, and (iii) a published, non-negotiable allocation process — because the alternative is that the loudest team's agent fleet spends the quarter's budget. The platform-architecture view of this (layers, gateway, quotas as platform services) is [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md); read it for the architecture, not this section.

**The governance question that arrives with volume.** Two questions appear only at scale, and neither is answered by engineering: *who authorised this spend?* (a budget with an owner and an escalation path) and *who is accountable for this outcome?* (an incident owner when a fleet action affects a customer). At ten sessions, both are answered by the person who wrote the agent. At ten thousand, they need a named owner per agent, a named budget per tenant, and a control that a non-engineer can operate. Operations and guardrails are [agentops_guide.md](agentops_guide.md) §5–§6; the platform-level commitments are [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) §5. The fleet-specific addition is only this: **the governance mechanism must operate at the speed of the fleet, not at the speed of a change request.** A kill switch that takes four hours to authorise is not a kill switch.

---

## 13. The Regulated-Enterprise and Banking Angle

What a bank adds to the fleet problem is not a different mechanism — it is *evidence, authority and consequence* attached to the same mechanisms. Five things, in the order they bind.

**1. Cost governance when spend is unbounded relative to a fixed budget.** A bank's technology budget is approved annually by a committee; an agent fleet's spend is incurred per token, per call, per retry, in real time. Those two systems do not have compatible shapes. A fleet that "worked fine in the pilot" can consume a disproportionate share of an annual run-cost line in a month, and — critically — the *mechanism* of doing so is not a mistake by anyone: it is the tail, the fan-out and the retries of §6.3 acting on a population. The governance requirement that follows is structural, not behavioural: a **hard per-session cap** and a **hard per-tenant budget with admission-time enforcement** (§6.5), because a bank cannot rely on "we will notice" — noticing happens after the quarter's run-cost variance is already reported. Where the general FinOps mechanics are concerned, [finops_guide.md](../finops_guide.md) is the reference; the AI-specific cost controls are [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) §7, and the operational cost-governance framing is [agentops_guide.md](agentops_guide.md) §6. What this guide insists on is the fleet-level invariant: **the ceiling must exist before the volume does.**

**2. The audit question that only appears at volume.** For one session, "what did the agent do?" is answerable by a human reading a trace. At fleet volume the question becomes three questions, and only the first is easy:

| Question | Why volume makes it hard | Practical requirement |
|---|---|---|
| Can you find the session? | Thousands of sessions per day, months of history, and a correlation identifier that must have been carried through fan-out (§7) | Correlation id minted at admission and propagated into every call and sub-agent |
| Can you reconstruct what it did? | Reconstruction needs the *inputs as they were* — model version, prompt version, tool versions, retrieved documents. A trace without versions cannot be replayed or defended | Version identifiers on every model, prompt, tool and retrieval in the trace ([agent_versioning_guide.md](agent_versioning_guide.md)) |
| Can you afford to keep it? | Full-fidelity retention of every token of every session at fleet volume is a storage and cost problem in its own right; sampling destroys the ability to answer about a *specific* session | A retention policy that keeps full fidelity for high-consequence classes and metadata-plus-hash for the rest, with the policy chosen before scale, not after |

The honest sentence: **at volume, the fleet's logging volume can make the audit incomplete in practice even when it is complete in principle.** Retention, sampling and tiering are therefore correctness decisions for a regulated actor, not storage optimisations. Compliance and AI-governance context for a bank's GenAI estate, including the supervisory expectations around records and third-party arrangements, is in [ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) and [mas_regulations_guidelines_guide.md](../../banking/mas_regulations_guidelines_guide.md).

**3. Shared capacity between business lines.** A bank's second fleet-scale problem is that capacity is shared across business lines that have *different* criticality, different regulatory clocks and different tolerances for shedding. A retail-facing assistant and an overnight reconciliation fleet cannot share one saturation policy (§5.6) any more than they share one budget. The requirements: per-business-line quotas and budgets, an explicit fairness rule for contention (not first-come-first-served by accident), and a documented admission priority so that a batch surge cannot starve a time-critical process. This is the same broker problem as §12, with the additional weight that the priority order may be a regulatory commitment rather than a preference. Business-impact-derived tolerances for exactly this kind of prioritisation are covered in [operational_resilience_framework_guide.md](../../banking/operational_resilience_framework_guide.md) §4–§5 — use those tolerances as the priority input rather than inventing a severity scale.

**4. The capacity-reservation question against a provider.** Providers offer commitment-backed capacity: Azure provisioned throughput sells PTUs per hour with a model-dependent TPM-per-PTU ratio and states plainly that **having quota does not guarantee capacity** in a region (VERIFIED 22 Sep 2026); Bedrock publishes Reserved tiers alongside Standard, Priority and Flex (VERIFIED 22 Sep 2026); Anthropic's Custom tier has no monthly spend cap and limits arranged with an account team (VERIFIED 22 Sep 2026). For a bank the question is governance before it is engineering: what does a reservation commit us to, what happens if we do not use it, can we evidence that the reserved capacity was used for its intended purpose, and — the third-party-risk angle — how does an outage of the *provider's* provisioned capacity map onto our own impact tolerance? Third-party and supply-chain resilience expectations are [operational_resilience_framework_guide.md](../../banking/operational_resilience_framework_guide.md) §10; the broader risk and control context is [financial_risk_compliance_systems_guide.md](../../banking/financial_risk_compliance_systems_guide.md). ⚠ I did not verify commercial reservation terms (utilisation commitments, cancellation, discounts) from provider documents in this pass — record them as NOT VERIFIED and read the provider's own contract terms.

**5. The honest statement.** For most banks, the binding constraint at scale is **governance and control, not engineering**. The engineering problems in this guide — concurrency accounting, admission control, retry design, cost ceilings, capacity arithmetic — are solvable, and a competent platform team solves them. What is harder, and what actually caps fleet size, is: who is accountable when the fleet acts, whether the spend is authorised, whether the audit is defensible six months later, and whether the business line whose work was shed agrees that it should have been. Those are the constraints to plan against. **Design the control plane first; scale the workload second.** And do not name a real bank in your own version of this paragraph — the only bank persona in this guide is the fictional one in §14.

---

## 14. Worked Example: Cymbal Bank Scales a Fleet

> **Explicitly illustrative and fictional.** Cymbal Bank is a fictional persona used across this repository. Every number in §14 is **ILLUSTRATIVE** — constructed to show the arithmetic, not measured, not published, and not attributable to any real institution. Every *rate* applied to those numbers is a dated snapshot: **provider prices and limits as read on 22 September 2026** (§5.1, §6.1), and the reader should re-read the provider pages before reusing any of it. Nothing here asserts that any real bank runs an agent fleet.

### 14.1 The situation

Cymbal Bank ran a successful pilot: one team, ~12 concurrent agent sessions, three capabilities — a service-operations agent that triages and drafts responses to customer complaints, a document agent that extracts and validates data from trade and onboarding documents, and a relationship-manager assistant that answers questions over the bank's policy corpus. The pilot used one provider account, one model, a single orchestration worker pool, and a warm prefix cache that stayed warm because the traffic was continuous. It worked.

The mandate: take all three to production for three business lines — **Service Ops**, **Document Ops**, **Advisory** — as one shared fleet. Target volume (ILLUSTRATIVE): **12,000 tasks/day**, a 10-hour working window, peak-to-average arrival ratio **3×**, average session duration **25 minutes**, **6 model calls** per session, **15,000 input tokens** per call with **80% prefix-cache hit** on the shared system prompt and policy corpus, **350 output tokens** per call.

### 14.2 Capacity arithmetic (ILLUSTRATIVE)

| Step | Arithmetic | Result |
|---|---|---|
| Average arrivals | 12,000 per day ÷ 10 h ÷ 60 | 20 tasks/min |
| Peak arrivals | 20 × 3 (burst) | 60 tasks/min |
| Peak concurrency (Little's Law) | 60 tasks/min × 25 min | **1,500 concurrent sessions** |
| Sessions making a model call at an instant | ×0.6 (remaining 40% awaiting tools/humans) | 900 |
| Model calls per minute | 900 ÷ (25 min ÷ 6 calls) | 216 calls/min |
| Output token demand | 216 × 350 | 75,600 OTPM |
| Uncached input demand (80% cached) | 216 × 3,000 | 648,000 ITPM |
| Uncached input demand if the cache goes cold | 216 × 15,000 | 3,240,000 ITPM |
| Requests per minute | 216 | 216 RPM |
| **Bottleneck** | compare with the read limits: 1,000 RPM / 2,000,000 ITPM / 400,000 OTPM | **none of the three binds at target — with the cache warm** |

The important output of that table is not "it fits". It is the **4th-to-5th row asymmetry**: the fleet has a 3× headroom margin on ITPM with a warm cache and is **1.6× over the read ITPM limit** the moment the cache goes cold. Cymbal therefore treats cache-hit rate as a *capacity* metric with an alert, not as a cost optimisation, and sets a hard admission rule: if cache-hit rate falls below 60% over a 10-minute window, peak admission is throttled to protect the quota.

### 14.3 The cost model (ILLUSTRATIVE, rates as read 22 Sep 2026)

Cost per call, using the read Claude Sonnet 5 rates ($2/MTok input, $0.20/MTok cache read, $10/MTok output — VERIFIED 22 Sep 2026):

| Component | Unit chain | Result |
|---|---|---|
| Uncached input per call | 3,000 ÷ 1e6 × $2.00 | $0.0060 |
| Cached input per call | 12,000 ÷ 1e6 × $0.20 | $0.0024 |
| Output per call | 350 ÷ 1e6 × $10.00 | $0.0035 |
| **Cost per call** | — | **$0.0119** |
| **Cost per median session** | 6 × $0.0119 | **$0.0714** |
| **Cost per p99-style session** (25 calls, average context 45,000) | 25 × $0.0287 | **$0.7175** |
| p99 ÷ median | $0.7175 ÷ $0.0714 | **10.0×** |
| **Blended mean** (94% median / 5% 15-call / 1% p99) | 0.94×$0.0714 + 0.05×$0.1785 + 0.01×$0.7175 | **$0.0832** |
| **Monthly cost at 12,000 tasks/day × 22 days** | $0.0832 × 264,000 | **≈ $21,970** |

Two ILLUSTRATIVE conclusions Cymbal's finance partner accepted only after seeing the third column: the tail costs 10× the median, and **6% of sessions carry 19% of the bill** — which is why the per-tenant budget is enforced at admission rather than reconciled at month-end. And note the counterfactual: at a 0% cache hit the same fleet's input cost per call rises from $0.0084 to $0.0300 — **total monthly cost would be ~$62,000 instead of ~$22,000.** The cache is worth more than a model downgrade.

### 14.4 The volume-only failure Cymbal hit

**What happened (ILLUSTRATIVE, but the mechanism is §8.1 + §8.2 exactly).** Cymbal scheduled its document re-scoring batch to start at 02:00, releasing **40,000 accumulated tasks** at once. That herd filled the fleet's admission queue to the concurrency ceiling within 90 seconds. At 02:11 the provider's endpoint slowed and began returning 429s. Because the client's retry policy at that time was *exponential backoff with a fixed 1-second base and no jitter*, every failing session retried on the same schedule. Attempts per original request rose from 1.02 to 2.6; offered request rate tripled while success rate fell by half. The retry traffic then passed through the gateway into an internal policy-search endpoint that had no concurrency limit, and *that* became the outage — the model endpoint recovered in four minutes, the internal service took twenty-two. Total: 22 minutes of degraded service, **$1,850 of spend for work that largely did not complete**, and 4,100 duplicate write attempts to the document register that were absorbed only because the idempotency keys had been implemented in the pilot.

**Why it was invisible in the pilot.** At 12 concurrent sessions, a scheduled batch released 12 tasks, the retry policy's lack of jitter produced two retries in the same second, nothing noticed, and the internal endpoint never saw a tenth of its capacity. Every ingredient of the failure existed in the pilot; only the population was missing.

**How they detected it.** Not by latency: a p99 latency alert fired but told them nothing about cause. The signals that worked were (a) **attempts per original request** (1.02 → 2.6, a ratio no dashboard showed before), (b) **distinct correlation ids flat while request count tripled** — the definition of a retry storm in one chart, (c) the internal endpoint's connection-pool saturation, and (d) retry-attributable spend appearing as a distinct cost line for the first time. They added (a) and (b) as first-class fleet metrics afterwards.

### 14.5 The design Cymbal adopted

**Retry and backoff** (all configurable, but the *policy* is a design decision):

| Element | Setting | Rationale |
|---|---|---|
| Backoff | Exponential, base 1 s, **full jitter** | Breaks the synchronisation that caused §14.4 (§5.5) |
| Retry budget | **Fleet-wide cap at 10% of calls in any rolling 10-minute window** | Bounds amplification; the storm shape cannot recur — retries decline as failures rise |
| Retry eligibility | Only calls proven idempotent, all carrying the session's idempotency key | Non-idempotent calls fail to a reconciliation queue instead of retrying |
| Circuit breaker | Per dependency, opens on error-rate threshold, half-open probe | Prevents the model hiccup from reaching the internal endpoint |
| `retry-after` | Always honoured; SDK auto-retry capped at 1 attempt and disabled for the streaming path | Rejection is an instruction (§5.4) |
| Scheduled work | Batch start **spread over a 90-minute window** with per-task jitter; queue drain rate-limited | Removes the thundering herd at its source (§8.2) |

**Admission control** (design decision, in the request path before the first model call):

1. Reject if concurrent sessions ≥ fleet ceiling (set at 1,350 — 90% of the computed 1,500, reserving 10% lane for interactive/incident work).
2. Reject or defer if the tenant's month-to-date spend ≥ 90% of its budget.
3. Defer if cache-hit rate < 60% (protects the ITPM ceiling, §14.2).
4. Reject if the session declares a step budget above the policy maximum, or requires more than the permitted sub-agent fan-out (depth ≤ 2, ≤ 3 sub-agents per parent).
5. **Shed interactive work never; defer batch work always** — with the deferral horizon published so the batch owner can plan.

**Per-tenant budgets and the kill switch.** Cymbal set monthly budgets per business line from the blended cost model, with an explicit oversized contingency for the tail ($12,000 / $6,000 / $4,000 ILLUSTRATIVE against an ILLUSTRATIVE $21,970 expected spend — the sum deliberately exceeds expected spend because the tail, not the mean, is what overruns). Three controls, all operable by a non-engineer: **throttle** a tenant to 20% admission, **freeze** a tenant (no new admissions, in-flight sessions drain), and **kill** a tenant's fleet (in-flight sessions checkpoint and stop, side effects reconciled — the durable-execution machinery in [durable_ai_agent_workflows_guide.md](../durable_ai_agent_workflows_guide.md) makes this a controlled stop rather than a corruption event). Every control has a named owner per business line, and the freeze/kill drill is rehearsed quarterly — an unexercised kill switch is a diagram.

### 14.6 The recommendation — and the one thing Cymbal refuses to scale

**Recommendation.** Scale the three capabilities as one fleet with a shared gateway and a *single* control plane (admission, budget, rate governor, kill switch), per-tenant quotas sized from the §14.3 model rather than from the pilot's observed peak, cache-hit rate treated as a capacity metric, and the retry policy of §14.5 adopted fleet-wide before raising volume. Raise the concurrency ceiling in 200-session increments, each increment gated on the substrate metrics of §11.2 (TTFT, queue age, attempts-per-original, ITPM consumption) rather than on business demand alone — because the pilot's observed peak is the *worst* input to a capacity plan, having been produced by a system whose limits were never reached.

**The one thing Cymbal deliberately refuses to scale: autonomous write access to the core banking system.** The fleet may read, prepare, draft, validate and recommend; it may not post, execute or release. Concretely, the Document Ops agent may validate a payment instruction and prepare the posting, but a human releases it; the Service Ops agent may draft and queue a customer response, but not send it; the Advisory agent may never place an order. Cymbal's stated reason is not distrust of the model — it is that **a fleet's failure modes are population phenomena, and the population makes the *irreversible* action the wrong thing to scale**. A retry storm, a fan-out, a state collision and an unbounded context are all survivable when the worst outcome is a wrong draft, and none are survivable when the worst outcome is a duplicated payment. The refusal is enforced structurally (the write credential is not issued to the agent runtime), not by instruction — because at fleet scale, an instruction is a request and a missing credential is a guarantee. This is also the boundary Cymbal uses to answer the audit question in §13: every agent action in the fleet is reversible by construction, so no session can be six months later *unexplainable in its effects*. The regulator-facing framing of that choice belongs to [ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) and [mas_regulations_guidelines_guide.md](../../banking/mas_regulations_guidelines_guide.md); the operational-tolerance framing to [operational_resilience_framework_guide.md](../../banking/operational_resilience_framework_guide.md).

---

## 15. The Anti-Patterns and the Claims Audit

### 15.1 The anti-pattern catalogue

Symptom → cause → guardrail. The cause column is the mechanism (§8); the guardrail column says whether the fix is a **design decision [D]** or a **configuration toggle [T]**.

| # | Anti-pattern | Symptom | Cause (mechanism) | Guardrail | Type |
|---|---|---|---|---|---|
| 1 | **Retrying without jitter** | Retries cluster in the same second; recovery never completes; latency p99 climbs while offered rate climbs | Fixed backoff synchronises a correlated failure population; amplification 1 + p + p² + … (§5.5, §8.1) | Exponential backoff **with full jitter**; fleet-wide retry budget (retries ≤ X% of calls in a window); breakers | [T] + [D] for the budget |
| 2 | **A fleet with no per-tenant budget** | One tenant's runaway week consumes the shared quota; other tenants' work is shed; month-end invoice is a surprise | Shared account, shared quota, no allocation; fan-out and tail act per-tenant but bill per-account (§6.5, §12) | Per-tenant budget enforced **at admission**; per-tenant quota; published allocation process | [D] |
| 3 | **A cost dashboard that shows the platform's bill and nothing attributable** | Finance asks "which team cost this?" and the answer requires a data engineering project | No correlation identifier carried through fan-out; the provider bill is one stream for a population (§7) | Correlation id minted at admission, inherited by sub-agents, emitted on every metering event; per-call metering schema (tenant, version, model, cached/uncached, tool, retry, status) | [D] |
| 4 | **Unbounded fan-out** | Concurrency and cost spike together with no change in incoming demand; tool endpoints saturate | Fan-out is multiplicative in depth, additive in breadth; the model decides how many helpers it needs (§8.3) | Hard caps on sub-agents per session, depth and total steps, enforced by the runtime; admission control applied to spawns; shared work budget | [D] |
| 5 | **The context that grows until the session is slow and expensive** | Session cost and TTFT rise with step index; batch occupancy falls; cache hits decay across a long session | Session cost ≈ quadratic in steps for linearly growing context; KV-cache memory scales with context; compaction can break prefix reuse (§8.5, §4.2) | Enforced context budget per session; explicit context-management policy; serving-side max-context guard; measure cache-hit impact of compaction rather than assuming it | [D] |
| 6 | **Load-testing with a production trace on a nondeterministic system and believing the result** | "It handled 10× in the test" followed by production failure at 2× | Traces cannot be replayed faithfully (long-running, expensive, non-idempotent, nondeterministic); acceleration distorts the concurrency it produces (§11.1) | Deterministic simulation for the substrate ([deterministic_engineering_guide.md](../deterministic_engineering_guide.md) §5); staged ramp with substrate metrics; budget the test's token spend first | [D] |
| 7 | **The quota that was sized from a demo** | The fleet plateaus in the first hour of real traffic; 429s appear at a fraction of planned concurrency | Demo traffic is continuous, low-concurrency and cache-warm; quota was never computed from tokens-per-session × sessions-per-minute (§5.3) | Compute quota from the chain, per dimension (RPM/ITPM/OTPM), with headroom against the *limit*; treat cache-hit rate as a capacity input; request quota increases before launch, not during the incident | [D] |
| 8 | **One saturation policy for all workloads** | Interactive work is shed while batch work runs; or batch work starves a time-critical process | Shed-vs-wait chosen per fleet instead of per workload class (§5.6) | Per-workload-class policy; documented admission priority derived from impact tolerances | [D] |
| 9 | **A queue with no bound and no age metric** | Latency grows without error rate rising; the eventual failure is total; memory grows | Unbounded queues hide overload and hold already-paid-for context | Bounded queue; queue depth **and age** as first-class metrics; age-based expiry; backpressure to producers | [D] |
| 10 | **A kill switch nobody has pulled** | During the incident, nobody can find the control or is authorised to use it | Governance latency exceeds fleet latency (§12) | Named owner per tenant; throttle/freeze/kill drilled quarterly; operable by a non-engineer | [D] |
| 11 | **Counting concurrency instead of concurrent model calls** | Capacity plan says one number; the serving layer sees another | Sessions awaiting tools are counted as load but generate none; fan-out sessions generate several calls at once (§3, §10.2) | Track concurrent model calls as the serving-layer demand statistic; separate orchestration concurrency from serving concurrency | [D] |
| 12 | **Cost optimised while completion rate is ignored** | "We cut token spend 20%" and business outcomes fell 20% | Cost per session improved by giving sessions less room; cost per *successful task* worsened (§6.4) | Report cost per **successful** task; treat success rate as a first-class cost input | [D] |

Two patterns deserve a note beyond the table. First, **most of these anti-patterns are invisible in the pilot by construction** — items 1, 2, 4, 6 and 7 are all population effects, so a pilot is *structurally incapable* of revealing them. That is why the mitigations are labelled [D]: they must be built before the pilot is scaled, not discovered afterwards. Second, **only two of the twelve are pure configuration toggles** (jitter, and the parameters of a limiter that already exists). The rest are architecture. That ratio is the honest answer to "how much is left to do?".

### 15.2 The claims audit

Every price, rate limit and quota cited in this guide, with provider, product, tier, value, unit, read date and status. **VERIFIED** = read on the provider's or framework's own page on the read date shown. **ILLUSTRATIVE** = a constructed arithmetic input invented for this guide, not a published fact. **NOT VERIFIED** = the page could not be read or the claim was not found on a primary source.

| Provider | Product | Tier | Value | Unit | Read date | Status |
|---|---|---|---|---|---|---|
| Anthropic | Claude Sonnet 5 | Standard API | 1,000 / 2,000,000 / 400,000 ⚠ (tier tab active on the page could not be confirmed) | RPM / ITPM / OTPM | 2026-09-22 | VERIFIED as read; **tier label ⚠** |
| Anthropic | Claude API spend caps | Start / Build / Scale | $500 / $1,000 / $200,000 | USD per month | 2026-09-22 | VERIFIED |
| Anthropic | Cache-aware ITPM rule | Most Claude models | Only uncached input counts toward ITPM; cached reads do not | rule (source example: 2,000,000 ITPM with 80% hit rate ≈ 10,000,000 total input tokens/min) | 2026-09-22 | VERIFIED |
| Anthropic | Prompt caching multipliers | Standard API | 5-minute write 1.25×, 1-hour write 2×, cache read 0.1× (0.025× on Fable 5.1 / Mythos 5.1) | multiplier on base input price | 2026-09-22 | VERIFIED |
| Anthropic | Claude Sonnet 5 / Haiku 4.5 / Opus 5 / Fable 5.1 prices | Standard API | $2 / $1 / $5 / $10 input; $10 / $5 / $25 / $50 output | USD per MTok | 2026-09-22 | VERIFIED |
| Anthropic | Rate-limit mechanism | All tiers | Token-bucket replenishment; per-model limits; 429 with `retry-after` | rule | 2026-09-22 | VERIFIED |
| Anthropic | Spend-limit rejection | Scale/Build/Start cap; self-set limit | 429 `enforced_spend_limit_reached` with **no** `retry-after`; self-set limit returns HTTP 400 | error semantics | 2026-09-22 | VERIFIED |
| Anthropic | `inference_geo: "us"` | Claude 4.6+ | 1.1× pricing multiplier | multiplier | 2026-09-22 | VERIFIED |
| OpenAI | Prompt caching + rate limits interaction | Standard API | Cached input tokens **still count** toward TPM; minimum cacheable prefix varies by model | rule | 2026-09-22 | VERIFIED |
| OpenAI | Batch API | Batch | 50,000 requests per batch; 200 MB input file; 2,000 batches/hour; separate rate-limit pool that does not consume standard per-model limits | limits | 2026-09-22 | VERIFIED |
| OpenAI | Retry guidance example | Documentation | Exponential backoff with jitter (`jitter: bool = True`); "no guarantees" on the example's suitability | code sample | 2026-09-22 | VERIFIED as published |
| OpenAI | `chat-latest` / `gpt-5.3-codex` prices | Standard | $5.00 / $0.50 / $30.00 and $1.75 / $0.175 / $14.00 | USD per MTok (input / cached input / output) | 2026-09-22 | VERIFIED |
| OpenAI | Regional processing uplift | Data residency endpoints | 10% uplift for models released on/after 2026-03-05 | uplift | 2026-09-22 | VERIFIED |
| Google Gemini | Spend-based rate limits | Tier 1 / 2 / 3 | $10 / $50 / $200 per rolling 10 minutes | USD per 10 minutes | 2026-09-22 (page updated 2026-09-02) | VERIFIED |
| Google Gemini | Billing tier caps | Tier 1 / 2 / 3 | $250 / $2,000 / $20,000–$100,000+ | USD | 2026-09-22 | VERIFIED |
| Google Gemini | Rate-limit scoping | All | Per **project**, not per API key; RPD resets midnight Pacific | rule | 2026-09-22 | VERIFIED |
| Google Gemini | Priority inference limits | Priority | Default 0.3× the standard rate limit per model and tier | multiplier | 2026-09-22 | VERIFIED |
| Google Gemini | `gemini-3.8-flash` prices | Standard / Batch / Priority | $0.75 / $3.75 standard (batch $0.375 / $1.875; priority 1.8× standard), rising to $1.50 / $7.50 on 2027-01-01 as published | USD per MTok in/out | 2026-09-22 (page updated 2026-09-16) | VERIFIED |
| Google Gemini | Context caching | Implicit | On by default for 2.5+; minimum 4,096 tokens (Gemini 3.x) / 2,048 (2.5) | tokens | 2026-09-22 | VERIFIED |
| Google Gemini | Grounding with Google Search | Paid | 5,000 free requests/month shared across Gemini 3.x, then $14 per 1,000 requests | USD per 1,000 requests | 2026-09-22 | VERIFIED |
| Google Gemini | Batch API limits | Batch | 100 concurrent batch requests; 2 GB input file; 20 GB file storage | limits | 2026-09-22 | VERIFIED |
| Azure / Microsoft Foundry | Quota scope | All | Not enforced at tenant level; highest scope is the **subscription**; Global Standard pools quota across regions for same model+version | rule | 2026-09-22 (page updated 2026-08-20) | VERIFIED |
| Azure / Microsoft Foundry | Quota tiers | Free + Tiers 1–6 | Seven tiers, auto-upgrading with consumption; Enterprise Agreement status affects the initial tier | rule | 2026-09-22 | VERIFIED |
| Azure / Microsoft Foundry | RPM:TPM ratio | `gpt-chat-latest` | 10 RPM per 1,000 TPM (versions 2026-05-05/05-28/06-24); 1 RPM per 1,000 TPM (version 2026-08-06) | ratio | 2026-09-22 | VERIFIED |
| Azure / Microsoft Foundry | Provisioned throughput | PTU | PTUs are model-independent; quota granted per subscription/region/deployment type; TPM per PTU varies by model; **quota ≠ capacity** | rule | 2026-09-22 (page updated 2026-07-15) | VERIFIED |
| AWS Bedrock | Per-model quota example | On-demand | Output tokens per minute for GPT-6 Astra: 100,000 | tokens per minute per Region | 2026-09-22 | VERIFIED as read (example from the quota table) |
| AWS Bedrock | Batch inference | Batch | 50% lower price than on-demand; **does not support tool calling or structured output**; not supported for provisioned models | price + constraints | 2026-09-22 | VERIFIED |
| AWS Bedrock | Service tiers | Standard / Flex / Priority / Reserved | Four tiers exist for model pricing | tier names | 2026-09-22 | VERIFIED |
| AWS Bedrock | Prompt caching | Implicit / Explicit | Two types; implicit is best-effort ("repeating an identical prompt doesn't guarantee a cache hit") | rule | 2026-09-22 | VERIFIED |
| Inngest | Concurrency control | Product docs | Limits **steps** executing concurrently, not function runs; paused/waiting runs do not count; concurrency keys give per-key virtual queues described as "best-effort fairness in multi-tenant systems"; limits can be shared across functions via scope | rule | 2026-09-22 | VERIFIED |
| OpenAI Agents SDK | Framework primitives | Documentation | Agents, handoffs/agents-as-tools, guardrails, sessions (persistent memory layer), sandbox agents, human-in-the-loop, tracing | feature list | 2026-09-22 | VERIFIED (feature existence only; no concurrency-limit statement on the page read) |
| — (this guide) | Session shape, cost model, capacity arithmetic, Cymbal figures, retry/fan-out/tail multipliers | — | All figures marked "ILLUSTRATIVE" throughout §5.3, §6, §10, §14 | constructed inputs | 2026-09-22 | **ILLUSTRATIVE — not published values** |
| vLLM | Continuous batching, chunked prefill, prefix caching, PagedAttention, disaggregated prefill/decode+encode | Project README (`raw.githubusercontent.com/vllm-project/vllm/main/README.md`) | Feature names as the project lists them; **no throughput or concurrency number stated** | features | 2026-09-22 | VERIFIED as read; ⚠ no figure |
| HuggingFace TGI | Continuous batching; maintenance mode | Docs index (`huggingface.co/docs/text-generation-inference/index`) | "Continuous batching of incoming requests for increased total throughput"; TGI in maintenance mode, HF recommending vLLM/SGLang | feature + status | 2026-09-22 | VERIFIED as read; ⚠ no figure |
| SGLang | RadixAttention, prefix caching, multi-GPU parallelism; PD disaggregation in model recipes | Docs landing page (`docs.sglang.ai`) | As stated; continuous-batching *wording* not present on the page read | features | 2026-09-22 | VERIFIED as read (⚠ continuous-batching wording) |
| TensorRT-LLM | "In-flight Batching" feature page (Paged-Attention/IFB scheduler); Disaggregated Serving with KV-cache exchange blog | Docs tree (`nvidia.github.io/TensorRT-LLM`, 1.3.0rc27) | Feature pages exist with these titles; mechanics page body not read | features | 2026-09-22 | VERIFIED as read (existence + terminology) |
| NVIDIA Triton | Request-level dynamic batching via per-model schedulers | Architecture page (`docs.nvidia.com/.../user_guide/architecture.html`, v2.72.0) | "multiple scheduling and batching algorithms … configured on a model-by-model basis"; no LLM continuous-batching claim | mechanism | 2026-09-22 | VERIFIED as read; ⚠ no LLM concurrency semantics |
| — (this guide) | **No serving-framework throughput or concurrency figure is cited anywhere**, because none of the pages read states one. Framework performance blogs/papers exist but were not imported (different hardware, model and workload). | — | — | — | 2026-09-22 | **DELIBERATELY ABSENT** |
| Azure / AWS / Anthropic | Commercial reservation terms (utilisation commitment, cancellation, discount) | Reserved / Provisioned / Custom | Not read in this pass | — | attempted 2026-09-22 | **NOT VERIFIED** |
| Temporal | Durable execution concurrency/delivery semantics | — | Attempted `docs.temporal.io/workflow-execution/activities` → 404 page-not-found | — | attempted 2026-09-22 | **NOT VERIFIED** |

---

## 16. What Could Not Be Verified, Glossary, Cross-References and Closing

### 16.1 What Could Not Be Verified

Honest ledger of the gaps, so nobody mistakes silence for a claim.

1. **Serving-framework *mechanics* — VERIFIED; serving-framework *capacity figures* — DO NOT EXIST in the docs.** The continuous/in-flight batching mechanics attributed to each framework in §4.1 were re-read at each project's own documentation on 2026-09-22 (vLLM README; HuggingFace TGI docs index; SGLang landing page; TensorRT-LLM docs tree; NVIDIA Triton architecture page, v2.72.0) and are quoted, with the two residual caveats marked ⚠ there. What could **not** be verified is a **numeric** statement: none of those pages states a sessions-per-GPU, tokens-per-second or concurrency figure for a given hardware and context distribution, and this guide accordingly **cites no per-framework number at all**. Framework performance studies exist (project blogs and papers) but are configuration-specific and were deliberately not imported in place of a primary capacity statement. `docs.vllm.ai` itself failed to fetch via this tool (scraper error, all engines) — the project README was used instead, which is first-party.
2. **No public "agents at scale" benchmark exists, and none is cited.** §4.4 and §10.4 state this explicitly. Any figure presented elsewhere as agents-per-GPU or sessions-per-endpoint should be treated as a vendor best case until you measure your own.
3. **A per-provider RPM/TPM table for OpenAI — NOT VERIFIED.** The rate-limits page confirms the *structure* (RPM and TPM enforced together, separate per model, batch in a separate pool) but publishes limits per account rather than as a general table.
4. **The tier label on Anthropic's per-model rate-limit table — ⚠ unconfirmed.** The page renders the table under tier tabs; the extraction could not determine which tab was active. The numbers read are recorded, with the caveat that the tier they belong to is unconfirmed.
5. **Temporal documentation — NOT VERIFIED.** The URL attempted (`docs.temporal.io/workflow-execution/activities`) returned a page-not-found, so no Temporal concurrency or delivery-semantics claim is made in this guide. Durable-execution mechanics are cross-referenced to [durable_ai_agent_workflows_guide.md](../durable_ai_agent_workflows_guide.md) instead.
6. **LangGraph, CrewAI, AutoGen/AG2, LlamaIndex workflow concurrency models — NOT VERIFIED.** These frameworks' own concurrency documentation was not read in this pass. Only Inngest's concurrency-control page and the OpenAI Agents SDK landing page were read, and both are cited only for what they state.
7. **Commercial reservation terms — NOT VERIFIED** (utilisation commitments, cancellation, discounts for Azure provisioned, Bedrock Reserved, Anthropic Custom).
8. **Third-party engineering write-ups from firms running large fleets — NOT READ.** I did not find and did not cite any first-party engineering write-up; nothing in this guide rests on one.
9. **Every number in §5.3, §6, §10, §14 that is labelled ILLUSTRATIVE is a constructed input**, not a measurement. They are arithmetic demonstrations. Do not quote them as capacity, cost or quota figures.
10. **All prices and limits are a dated snapshot (22 September 2026)**, and at least one page I read (Gemini) contains prices that *change on 2027-01-01* — published evidence that re-reading the primary source before reuse is mandatory, not optional.

### 16.2 Glossary

| Term | Definition used in this guide |
|---|---|
| **Admission control** | Deciding whether to accept a new session *before* spending on it; the only control that is free when it fires. |
| **Backpressure** | A signal that slows producers when consumers saturate, instead of buffering. |
| **Backoff with jitter** | Retry delay that grows (usually exponentially) and is randomised to decorrelate a synchronised failure population. |
| **Burst factor** | Peak arrival rate ÷ average arrival rate; the demand-variance multiplier, distinct from headroom. |
| **Batch API** | Asynchronous bulk inference at a discount (OpenAI 50% off list and a separate rate-limit pool; Gemini batch 50% of standard; Bedrock batch 50% lower than on-demand) — with tool-calling constraints on some providers. |
| **Concurrency** | The number of sessions or calls in flight at one instant. A count. |
| **Continuous batching** | Serving technique of admitting and retiring requests at the granularity of decode steps rather than whole batches. |
| **Cost per successful task** | Total attributable spend ÷ successfully completed tasks; the only cost unit that survives a quality regression. |
| **Decode** | The token-by-token generation phase; memory-bandwidth-bound. |
| **Fan-out** | One session spawning sub-agents or parallel branches; multiplicative in depth, additive in breadth. |
| **Headroom** | Spare capacity held against component failure and quota variance (not against demand variance — that is the burst factor). |
| **In-flight work** | Everything a live session is currently holding: an open model or tool call, its context, its state, its locks. |
| **KV cache** | Per-request attention keys/values held in GPU memory during generation; scales with context length and caps concurrency. |
| **Little's Law** | Throughput ≈ concurrency ÷ average duration; the bridge between the two. |
| **Prefill** | Processing the input prompt to produce the first token; compute-bound and proportional to input length. |
| **Prompt/prefix caching** | Reusing a previously processed prefix at a reduced rate (0.1× input on Anthropic/OpenAI tables read; 0.1× cached input on Gemini) — with different rate-limit treatment per provider. |
| **Provisioned throughput (PTU)** | Dedicated, pre-purchased model capacity billed per unit-hour; capacity ≠ quota. |
| **Quota** | Any provider- or platform-imposed ceiling on rate, tokens, spend or provisioned capacity. |
| **Rate governor** | A call-level limiter that keeps offered traffic inside a quota. |
| **Session** | One logical agent run: task, accumulated context, tool calls, state; long-running and stateful. |
| **Shed vs wait** | The saturation choice per workload class: reject fast (shed) or queue for later (wait). |
| **Thundering herd** | A synchronised start (cron, cache TTL, queue drain, deployment restart) producing a load step that exceeds capacity. |
| **Throughput** | Completions per unit time. A rate. |
| **Token bucket** | Rate-limiting algorithm with continuous replenishment (described in Anthropic's own documentation); allows short bursts up to capacity. |
| **The long tail** | The p95/p99 session; at fleet scale the tail, not the median, sets the bill. |

### 16.3 Cross-References

**Owned by this guide:** the concurrency model (§3), the serving layer's fleet consequences (§4), quota and backpressure design (§5), fleet economics (§6), attribution's fleet-specific requirements (§7), volume-only failure modes (§8), fleet concurrency control over shared resources (§9), capacity planning's honest model (§10), fleet-level testing and chaos scope (§11), the organisational and regulated-enterprise fleet questions (§12–§13), and the Cymbal worked example (§14).

**Deferred, with the section that owns it:**

- [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) — §5 Non-Functional Requirements (scaling targets, throughput/latency/cost NFRs) and the platform architecture; cited here, never restated.
- [agentops_guide.md](agentops_guide.md) — §3 Observability, §4 Evaluation, §5 Monitoring & Alerting, §6 Guardrails and Cost (budgets, attribution, cost monitoring), §7 Tooling Landscape.
- [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) — §3 Reliability, §4 Testing, §5 Deployment, §6 Operations, §7 Cost Control (token budgets, caching, routing, cost observability).
- [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md) — §6/§7 multi-backend operation and model routing.
- [hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md) — agent topology and hierarchy.
- [durable_ai_agent_workflows_guide.md](../durable_ai_agent_workflows_guide.md) — durable execution, checkpointing, idempotency, sagas and compensation, fan-out/fan-in, human-step escalation, delivery semantics.
- [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) and [agents_work_fall_apart_guide.md](agents_work_fall_apart_guide.md) — the general failure taxonomy, compounding-error arithmetic, and success/failure conditions.
- [deterministic_engineering_guide.md](../deterministic_engineering_guide.md) §5 — deterministic simulation testing (the reproducibility answer for §11).
- [chaos_engineering_guide.md](../chaos_engineering_guide.md) §4–§8 — experiment design, failure taxonomy, tooling, observability prerequisite, safety.
- [deterministic_simulation_testing_guide.md](../deterministic_simulation_testing_guide.md) — the simulation testing discipline in full.
- [finops_guide.md](../finops_guide.md) — general FinOps mechanics for the cost-governance frame in §6.5 and §13.
- [context_engineering_guide.md](context_engineering_guide.md) and [agent_runtime_cache_design_guide.md](agent_runtime_cache_design_guide.md) — context management and cache design techniques behind §8.5 and §6.1.
- [llm_latency_optimization_guide.md](llm_latency_optimization_guide.md) — latency work, which §3.3 shows is also capacity work.
- [agent_versioning_guide.md](agent_versioning_guide.md) — version identification for cost-regression detection (§7) and audit reconstruction (§13).
- [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md) — sandbox-level isolation, the substrate for per-session state.
- [agentic_engineering_guide.md](agentic_engineering_guide.md), [feedback_mechanisms_llm_agents_guide.md](feedback_mechanisms_llm_agents_guide.md), [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) — engineering practice around the agent loop.
- [operational_resilience_framework_guide.md](../../banking/operational_resilience_framework_guide.md) — §4–§5 impact tolerances, §10 third-party and supply-chain resilience.
- [ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md), [mas_regulations_guidelines_guide.md](../../banking/mas_regulations_guidelines_guide.md), [financial_risk_compliance_systems_guide.md](../../banking/financial_risk_compliance_systems_guide.md) — the regulated-enterprise control, records and third-party-risk context for §13 and §14.6.

### 16.4 Closing Summary

A fleet is not one agent multiplied, and the difference is structural rather than a matter of degree. One agent is a stateful, long-running, expensive, non-idempotent unit of work, and multiplying it produces three shape changes at once: the cost curve becomes heavy-tailed and multiplier-driven (the p99 session costs ~10–13× the median in the ILLUSTRATIVE models here, and fan-out and retries multiply what the tail costs); the failure modes become emergent populations phenomena (retry storms, thundering herds, cascading fan-out, cost blowout, unbounded context growth, state collision — none of which one agent can produce); and the binding constraint moves from model quality to quota arithmetic, KV-cache memory, downstream endpoints, and the governance question of who may spend what. The controls that follow are unglamorous and mostly architectural: compute quota per dimension from tokens per session × sessions per minute with the cache assumption made explicit, because the same quota supported ~3,700 concurrent sessions with a warm cache and ~556 without in the illustrative chain of §5.3; put admission control in the request path where rejecting is free, because failing at step 30 costs full price for no outcome; jitter everything that can be scheduled, because a fleet-wide policy applied identically by every session is a load amplifier at exactly the wrong moment; cap context, fan-out and spend as enforced ceilings rather than prompt suggestions; and instrument concurrent model calls, attempts-per-original-request and cost per successful task, because those three numbers reveal the fleet failures that per-session dashboards structurally cannot see. For a regulated enterprise the decisive constraints are governance and control rather than engineering — authorisation, attribution, audit and priority are what actually cap fleet size — and the Cymbal example's most important decision is its refusal, not its arithmetic: it scales what is reversible and refuses to scale autonomous write access to the system of record. A pilot cannot reveal any of this, because every one of these failures requires a population. The pilot therefore must not be read as evidence about the fleet; it is evidence about one session — and a fleet is not one agent multiplied.
