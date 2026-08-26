# Production-Ready LLM Agents: The Design & Operations Playbook
> **A comprehensive guide to building agents that survive production — the production requirements, the design-for-production principles, the reliability engineering, the testing, the deployment, the operations, the cost control, the security, the governance, and a worked production launch.**

**Author:** Jack Liu Shurui
**Role:** Solution Architect, Cymbal Bank
**Date:** August 2026
**Version:** 1.0
**Repository:** github.com/jackliusr/research
**Series:** AI/LLM Engineering Guides — Agent & Platform track

**Reading time:** ~60 minutes

---
## Table of Contents
1. [The Production Requirements](#1-the-production-requirements)
2. [Design for Production](#2-design-for-production)
3. [Reliability Engineering](#3-reliability-engineering)
4. [Testing](#4-testing)
5. [Deployment](#5-deployment)
6. [Operations](#6-operations)
7. [Cost Control](#7-cost-control)
8. [Security and Governance](#8-security-and-governance)
9. [The Production-Readiness Framework](#9-the-production-readiness-framework)
10. [Worked Example: Launching a Banking Customer-Support Agent](#10-worked-example-launching-a-banking-customer-support-agent)
11. [Summary: Production-Ready Agents in One Page](#11-summary-production-ready-agents-in-one-page)
12. [Glossary](#12-glossary)

---
## How This Guide Fits the Series
This guide is the **positive design playbook**: how to *build* an agent that survives production. Its sibling, [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md), is the **negative diagnosis**: why agents *fail* in production. Read them as a pair — the failure guide's catalogue of failure modes is the "why" that motivates every principle here. Where the failure guide says "agents fail because of unbounded autonomy, no eval gates, no cost caps," this guide says "therefore: narrow scope, deterministic cores, eval gates, budget caps." The failure guide's checklist is the negative image; Section 9's readiness checklist is the positive.

Companion guides in the series:

| Guide | Role in this playbook |
|---|---|
| [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) | The failure catalogue — the "why" behind every design rule here |
| [autonomous_agents_guide.md](autonomous_agents_guide.md) | The umbrella: architectures, control loops, evaluation |
| [ai_agent_drift_guide.md](ai_agent_drift_guide.md) | Drift & monitoring — the ops-side companion (loop detection, drift alerts) |
| [agent_scaffolding_guide.md](agent_scaffolding_guide.md) | The code scaffold: loop, tools, config, prompt versioning |
| [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) | The platform: gateways, observability, governance, FinOps |
| [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md) | Multi-agent orchestration patterns |
| [llm_agent_use_cases.md](llm_agent_use_cases.md) | Where agents make sense (and where they don't) |
| [mcp_framework_tools_guide.md](mcp_framework_tools_guide.md) | MCP tool layer — tool permissions and allow-lists |
| [agent_runtime_cache_design_guide.md](agent_runtime_cache_design_guide.md) | Prompt/response caching economics |
| [llm_guard_models_guide.md](llm_guard_models_guide.md) | Guard models: input/output filtering, classifiers |
| [implementing-responsible-ai.md](implementing-responsible-ai.md) | HITL, approvals, audit, governance |
| [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md) and [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) | Eval design and frameworks |
| [rag/rag_evaluation_methodology_guide.md](rag/rag_evaluation_methodology_guide.md) | Golden sets, regression evals (retrieval side) |
| [llm_latency_optimization_guide.md](llm_latency_optimization_guide.md) | Latency: caching, streaming, speculative decoding |
| [coding_agents_research.md](coding_agents_research.md) | The most mature agent category, as a case study |
| [finops_guide.md](../finops_guide.md) | FinOps practice for cost attribution (technology root) |

**Verification note.** This guide was written against the 2025–2026 state of practice. Claims are flagged inline as **(verified)** where the practice is well-established in published engineering guidance, **(emerging)** where it is newer and still consolidating (e.g. "LLM SRE"), and **(opinion)** where it is my synthesis as a solution architect. Unverifiable claims are flagged honestly — production-agent practice is young, and some "standard" patterns are more convention than consensus.

---
## 1. The Production Requirements
### 1.1 What "Production-Ready" Actually Means
**Production-ready** is not a feature. It is a **contract** — the set of properties an agent must exhibit before it is allowed to touch real users, real data, and real money. The definition that anchors this guide:

> **An agent is production-ready when its reliability, availability, cost, latency, security, and governance characteristics are specified, measured, and meet agreed targets — and when every change to it (prompt, model, tool, config) goes through a defined pipeline with evals, gates, and rollback.**

The difference between a demo and a production agent is not the model — it is everything around the model:

| | Demo agent | Production agent |
|---|---|---|
| **Success** | "Works" on 10 curated examples | ≥95% success on a golden set + live traffic, measured continuously |
| **Failure** | Crashes, retries forever, or hallucinates silently | Fails fast, degrades gracefully, escalates to a human |
| **Cost** | A few dollars of API spend, nobody cares | Cost per task is budgeted, metered, and attributed |
| **Latency** | Whatever it takes | p95 and p99 are agreed, monitored, and under budget |
| **Security** | Direct tool access, no checks | Guardrails, allow-lists, injection defense, data protection |
| **Change** | Edit the prompt, redeploy, hope | Versioned, eval-gated, canaried, rollback-able |
| **Visibility** | A console log | Traces, metrics, alerts, dashboards, audit trail |
| **Accountability** | Nobody knows what it did | Every action is logged, attributable, and auditable |

The failure guide ([llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md)) documents what happens when teams ship the left column: unbounded loops, silent wrong answers, cost blow-ups, data leaks, and trust loss. This guide is the antidote.

### 1.2 The Five Production Requirements
Production agents are judged on five dimensions, and they are all *system* properties — not model properties:

**1. Reliability — success rates and error rates.** The agent must complete its task correctly and consistently. Reliability is measured as success rate (task completed with verified-correct outcome), error rate (failed/completed-wrongly), and task-level outcomes — not just "the model didn't crash." Because LLMs are non-deterministic, reliability must be *engineered*: deterministic cores, verifiers, retries, and fallbacks (Sections 2–3). The reliability bar is set and enforced via SLOs.

**2. Availability — uptime and degraded modes.** The agent's *service* must be up when users need it: the API gateway, the runtime, the tools. But availability for an agent has a second dimension: **degraded modes**. When the LLM provider has an outage, or the vector store is down, or the model returns garbage, the agent must have a defined degraded behavior — a fallback model, a reduced-scope path, or a clean handoff to a human — rather than a 500 or a silent wrong answer. An agent that is "up" but answering nonsense is not available; it is a liability.

**3. Cost — cost per task and budgets.** Every agent run consumes tokens, tool calls, and infrastructure. The unit that matters is **cost per completed task** — the all-in cost of one successful resolution (prompt tokens + completion tokens + tool/API fees + retries + human review time). Production agents carry a **per-task budget** and a **per-agent/per-team monthly budget**, because a runaway loop or an injection-driven cost bomb can turn a 2¢ task into a $25 bill (see Section 7).

**4. Latency — p95 and p99, the user experience.** Users experience latency as percentiles, not averages. An agent that answers in 2 seconds on average but takes 90 seconds at p99 is broken for the slowest 1% of users. Production agents specify **p95 and p99 targets per task** (e.g. p95 ≤ 10s for a support agent) and design for them: routing, caching, streaming, parallel tool calls, and timeouts (see [llm_latency_optimization_guide.md](llm_latency_optimization_guide.md)). Latency and reliability trade off: retries improve success but add tail latency — the SLOs must encode the trade.

**5. Security — guardrails and data protection.** Agents hold tools, and tools touch data. Production agents enforce **input/output filters** (guard models), **tool allow-lists** (an agent can only call what its role permits), **prompt-injection defense** (tool outputs are untrusted data, never instructions), and **data protection** (PII handling, encryption, DLP, retention) (Section 8). In a bank, security is a non-negotiable gate, not a scoring dimension.

### 1.3 SLIs, SLOs, and Error Budgets for Agents
The SRE discipline of **SLI → SLO → error budget** transfers directly to agents, with agent-specific SLIs. *(Verified — applying SLI/SLO practice to LLM agents is established 2025–2026 practice: multiple engineering playbooks define the same core SLI set.)*

- **SLI (Service Level Indicator)** — a *measured* quantity. For agents, the canonical SLI set is:
  - **Task success rate** — fraction of tasks completed with verified-correct outcome (the agent's own "correctness," not just HTTP 200).
  - **Error rate** — fraction of tasks that failed (tool errors, timeout, escalation, or wrong answer caught by the verifier).
  - **Tool-call success rate** — fraction of tool invocations that succeeded (the reliability of the agent's *hands*, which degrades independently of the model).
  - **Latency percentiles** — p50/p95/p99 end-to-end per task.
  - **Cost per task** — tokens + tool fees per completed task (a cost SLI is unusual in classic SRE but essential for agents).
  - **Loop/step counts** — steps per task and stuck-loop rate (agents have a failure mode classic services don't: infinite reasoning loops).
- **SLO (Service Level Objective)** — the *target* on an SLI over a window: "task success rate ≥ 95% over 30 days," "p95 latency ≤ 10s," "cost per ticket ≤ $0.50."
- **Error budget** — the tolerated deviation: `100% − SLO`. A 95% success SLO gives a 5% error budget per window. Error budget is *spendable*: teams may ship riskier changes while budget remains, and must stop/slow when it's exhausted. Error budgets convert "how reliable must this be?" into an operational decision rule, and — critically for agents — they bound **autonomy**: when the budget is burning, reduce autonomy (lower temperature, more HITL, narrower tools) before the blast radius grows.

**The agent-specific SLO practice** (verified, with judgment): teams set *two tiers* of SLOs — a **service SLO** (availability/latency of the agent endpoint, owned by the platform team) and a **task SLO** (success rate/cost per task, owned by the agent team). A support agent might have:

| Tier | SLI | SLO |
|---|---|---|
| Service | Endpoint availability | 99.9% monthly |
| Service | p95 latency | ≤ 10s |
| Task | Task success rate | ≥ 95% (30-day window) |
| Task | Cost per ticket | ≤ $0.50 |
| Task | Loop rate | < 1% of tasks exceed 20 steps |

**Design implication:** every SLO target is a design input, not a reporting output. "Task success ≥ 95%" forces the verifier and the fallback design; "p95 ≤ 10s" forces caching and model routing; "cost ≤ $0.50" forces the budget cap and the cheap-model path; "loop rate < 1%" forces the max-steps policy.

### 1.4 Availability and Degraded Modes
Availability is measured as uptime of the *agent service* — the runtime, not just the model. But the interesting agent-specific work is **degraded modes**: the pre-defined, tested behaviors for when a dependency fails:

| Failure | Naive behavior | Degraded mode |
|---|---|---|
| LLM provider outage | 500 errors / queue buildup | Route to fallback provider or fallback model (Section 3.4) |
| Vector store down | Agent hallucinates from memory | Disable retrieval-dependent tools; answer only from deterministic data |
| Rate limiting | Retry storm | Backoff + circuit breaker; shed low-priority traffic |
| Model quality collapse (drift) | Silent bad answers | Increase verifier strictness; increase HITL; alert (see [ai_agent_drift_guide.md](ai_agent_drift_guide.md)) |
| Tool API down | Agent flails retrying a dead tool | Circuit breaker opens; agent takes the fallback path (rule-based answer, human handoff) |

**The rule:** every agent ships with a written degraded-mode matrix, and the degraded modes are *tested* — a chaos exercise that kills the LLM provider should produce the degraded path, not an error page (Section 4.4).

### 1.5 The Requirements Table
| Requirement | Metric | Typical target | Design implication |
|---|---|---|---|
| Reliability | Task success rate (verified-correct) | ≥ 95% (30-day) | Verifier steps, deterministic core, retries, fallbacks |
| Reliability | Error rate (failed/wrong tasks) | ≤ 5% | Fail-fast + escalate; verifier catches the wrong answers |
| Reliability | Tool-call success rate | ≥ 98% | Retry/backoff, circuit breakers, idempotent tools |
| Availability | Service uptime | ≥ 99.9% | Platform redundancy, runtime health checks |
| Availability | Degraded-mode coverage | Every failure mode has a defined path | Degraded-mode matrix, chaos-tested |
| Cost | Cost per completed task | $0.05–$2 depending on task | Token budgets, caching, model routing |
| Cost | Monthly spend per agent | Agreed with owner | Budget caps, alerting on spikes |
| Latency | p95 task latency | ≤ 10s (support); ≤ 3s (inline) | Caching, routing, streaming, parallel tools |
| Latency | p99 task latency | p95 × 2–3 | Timeouts, max-steps policy, fail-fast |
| Security | Guardrail coverage | 100% of tool calls pass allow-list | Input/output filters, tool allow-lists |
| Security | Injection resistance | No tool-executing injections on red-team set | Untrusted-tool-output discipline (Section 8) |
| Security | Data protection | PII redacted/encrypted; DLP enforced | Redaction, encryption, retention policy |
| Governance | Auditability | Every action logged & attributable | Audit trail, trace IDs, prompt/model versioning |
| Governance | Change safety | Zero un-gated production changes | CI eval gates, canary, rollback (Section 5) |

---
## 2. Design for Production
### 2.1 The Master Principle: Narrow Scope
**Do one thing well.** The single highest-leverage design decision for a production agent is its **scope**: the narrower the scope, the higher the achievable success rate, the lower the cost, the smaller the attack surface, and the easier the evals.

- A **narrow-scope agent** handles one task family: "answer balance and transaction questions," "triage support tickets into categories," "draft payment-recall letters for review." Its tool set is small and permissioned; its success criteria are crisp; its golden set is cheap to build; its failure modes are enumerable.
- A **broad-scope agent** ("handle all customer support") is a demo darling and a production nightmare: unbounded inputs, unbounded tools, unbounded eval surface, and unbounded failure modes — most of them unseen until production.

**The practice** (verified — this is the consensus of 2025–2026 production-agent engineering): decompose broad ambitions into a **portfolio of narrow agents** — one per task family — plus a small **router** (which can itself be a narrow classifier or a tiny LLM) that sends each request to the right agent. Routing is deterministic-ish and cheap; each downstream agent stays narrow. A bank does not build "the support agent"; it builds `balance_query_agent`, `transaction_dispute_agent`, `payment_status_agent`, `ticket_triage_agent` — each with its own SLOs, evals, and budget (see [autonomous_agents_guide.md](autonomous_agents_guide.md) and [llm_agent_use_cases.md](llm_agent_use_cases.md) for the decomposition patterns).

Scope also bounds **autonomy**: the agent's freedom (how many steps, which tools, whether it can mutate state) must be proportional to its scope. A read-only FAQ agent can be fully autonomous; a payment agent cannot.

### 2.2 Deterministic Cores: Workflow-First
**Workflows for the core, agents for the edges.** This is the defining design principle of 2025–2026 production practice *(verified — it is the direct application of Anthropic's "Building Effective Agents" guidance: find the simplest solution and only add complexity when needed; use *workflows* (predefined code paths) when the task is predictable, and *agents* (model-directed steps) only where genuine flexibility is required; the "workflows for the core, agents for the edges" phrasing is the community synthesis of that guidance)*.

**Workflow-first** means: before you let the model decide anything, ask *"can this step be code?"* The production agent's anatomy:

- **The core is deterministic.** The task's skeleton — ticket lookup, account validation, entitlement checks, the order of operations, the retry policy, the escalation path — is **human-designed code**, not model-reasoned steps. In a banking agent, "look up the account → check the caller's entitlement → fetch the transaction → format the answer" is a fixed pipeline; the LLM fills in the conversational parts.
- **The edges are agentic.** The model is used only where flexibility is genuinely required: understanding a free-text request, extracting intent, summarizing, drafting a nuanced reply. Each of those model calls is a *bounded* decision inside a deterministic frame.
- **Deterministic where possible, model where necessary.** Every place you replace a model decision with a rule (a lookup, a regex, a classifier, a policy check) you buy: reproducibility (same input → same output → debuggable), testability (deterministic tests, no flakiness), cost (a rule is free; a model call is not), latency (a lookup is milliseconds; a generation is seconds), and auditability (a rule has a spec; a model has a probability).

The choice rule of thumb (verified, per Anthropic's guidance): **if the task can be solved with a workflow, don't build an agent.** Use the augmented-LLM or workflow patterns (prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer) before reaching for the autonomous loop. The autonomous loop is the most powerful *and* the least predictable pattern — it belongs at the edges, not in the core.

### 2.3 Verifier Steps: Second-Pass Checks
**Every production agent has a verifier.** A verifier is a second-pass check on the agent's output — a distinct mechanism (not the same model patting itself on the back) that confirms the output meets the contract before it is released to the user or to a tool. *(Verified — the verifier/evaluator-optimizer pattern is standard: Anthropic's evaluator-optimizer workflow is one form; independent output validation is consensus production practice.)*

Verifier forms, weakest to strongest:

| Verifier | What it checks | When it's enough |
|---|---|---|
| **Schema/format check** (code) | Output is valid JSON, has required fields, IDs exist | Structured tool outputs |
| **Deterministic business rule check** (code) | Amount ≤ limit, account matches entitlement, date is in range | Banking/transaction outputs — non-negotiable |
| **Guard-model classifier** | Toxicity, PII leakage, jailbreak, refusal (see [llm_guard_models_guide.md](llm_guard_models_guide.md)) | Safety-critical content |
| **Separate LLM pass (evaluator)** | A different prompt/model reviews answer quality, groundedness, instruction compliance | Free-text answers |
| **Human review (HITL)** | The strongest verifier, for high-value/low-tolerance actions | Payments, disclosures, legal (Section 2.6) |

The verifier **closes the loop**: a failed verification routes to a fix path (regenerate with feedback), a fallback (simpler model, deterministic answer), or an escalation (human) — it never silently passes. Verifier failures are a **first-class metric**: a rising verifier-failure rate is the earliest signal of drift ([ai_agent_drift_guide.md](ai_agent_drift_guide.md)) and the primary input to the golden set.

### 2.4 Guardrails
**Guardrails are structural, not stylistic** (see [llm_guard_models_guide.md](llm_guard_models_guide.md) and [implementing-responsible-ai.md](implementing-responsible-ai.md)). A production agent runs every input and output through filter layers:

- **Input guardrails:** what is allowed *in* — PII redaction before tool calls, prompt-injection screening, input validation against the agent's scope ("this is a balance query; it does not take file paths").
- **Tool guardrails:** what the agent may *do* — a tool allow-list per agent, parameter validation, per-call permission checks, read-only vs. mutating tool classes.
- **Output guardrails:** what is allowed *out* — PII leakage detection, forbidden-content filters, format validation, and (in banking) the "no unverified financial advice" rule.

Guardrails are **enforced in code at the runtime boundary** — in the tool registry and the I/O layer of the scaffold ([agent_scaffolding_guide.md](agent_scaffolding_guide.md)) — never as "please don't" clauses in the system prompt. A prompt is advice; a guardrail is a check that runs.

### 2.5 Fallbacks: Degraded Modes by Design
**Every step of the agent has a fallback path** — a pre-defined, cheaper, or more conservative way to complete the step when the primary fails (see Section 3.4 for the runtime mechanics). The fallback ladder, in order:

1. **Retry** (transient failures — same path, with backoff).
2. **Simpler model** (the small/cheap model can often answer the straightforward 80%; if the strong model failed, the simple one may still resolve it — and it is cheaper).
3. **Deterministic path** (if the model step fails, use the rule: lookup tables, templated answers, the last-known-good answer for FAQ-class requests).
4. **Reduced scope** (answer only the verifiable part; decline the rest explicitly: "I can confirm your balance; I cannot change your address").
5. **Human handoff** (escalate with full context — the transcript, the attempted steps, the failure reason).

The fallback design is *the* thing that turns an agent outage into an agent hiccup. A support agent whose LLM provider is down but which can still answer "what is my balance?" from the deterministic core is worth more than one that returns 503s.

### 2.6 HITL Checkpoints
**Human-in-the-loop checkpoints** are approval gates where a human must confirm before the agent takes a high-stakes action: sending a payment, changing an address, sending an external email, executing a trade, or disclosing information. (See [implementing-responsible-ai.md](implementing-responsible-ai.md) for the governance treatment; HITL is the operational mechanism of responsible AI.)

HITL design decisions:

- **Where:** at the *action boundary* — between the agent's draft and the execution of a mutating or high-value tool call. Reads don't need approval; writes and external effects do.
- **What the human sees:** the proposed action, its justification, the evidence (fetched data), and the confidence — not a raw transcript. The human approves or rejects in one click; the agent then continues or takes the rejection path.
- **Theater vs. substance:** a checkbox the human clicks without information is worse than no gate (it creates false accountability). HITL is only real when the human has the context to veto intelligently.
- **Cost:** every checkpoint adds latency and human cost — so checkpoints are *scoped*: auto-approve low-risk actions (read-only, reversible), checkpoint medium (irreversible-but-small), escalate high (payments, disclosures). This is the same risk-tiering the failure guide prescribes for autonomy.

### 2.7 The Design Principles Table
| Principle | What it prevents | Implementation |
|---|---|---|
| Narrow scope | Unbounded inputs/tools/failure modes; eval surface explosion | One agent per task family; router upstream; bounded tool set |
| Deterministic core / workflow-first | Model chaos in the critical path; non-reproducible behavior | Fixed pipelines; code for the skeleton; LLM only at the edges ([autonomous_agents_guide.md](autonomous_agents_guide.md)) |
| Verifier steps | Silent wrong answers reaching users | Second-pass checks: schema → rules → guard model → evaluator → HITL |
| Guardrails | Injection, PII leakage, out-of-scope actions | Enforced input/output/tool filters in the runtime ([llm_guard_models_guide.md](llm_guard_models_guide.md)) |
| Fallbacks | Outages and model failures becoming user-facing failures | Retry → simpler model → deterministic → reduced scope → human handoff |
| HITL checkpoints | Irreversible or high-value actions taken without approval | Approval gates at action boundaries, risk-tiered ([implementing-responsible-ai.md](implementing-responsible-ai.md)) |

**The design test:** write the agent's failure story before writing its success story. For every step, ask: *what happens if this step fails, if the model lies, if the tool is down, if the user is hostile?* If the answer is "we'll see," the design is not production-ready.

---
## 3. Reliability Engineering
### 3.1 The Reliability Stack
Reliability for agents is layered: **the model can be wrong** (handled by verifiers and fallbacks, Section 2), **the infrastructure can fail** (handled here), and **the agent can misbehave** (loops, overruns — handled by timeouts, step limits, and budgets). This section covers the infrastructure and behavior layers: retries, idempotency, timeouts, circuit breakers, degraded modes, and state management.

### 3.2 Retries with Exponential Backoff
**Retry transient failures; never retry everything.** Network blips, rate limits (429), and provider 5xx are transient — a retry with **exponential backoff + jitter** is the standard treatment. *(Verified — retry/backoff/jitter is consensus practice for LLM API calls.)*

The practice:

- Retry only idempotent-safe calls (see 3.3) and only transient errors (429, 5xx, timeouts, connection resets). **Never retry 400-class errors** (bad request, auth failure) — they will fail identically.
- Backoff schedule: `min(initial × 2^attempt, cap)` with jitter (randomize the delay ±20%) to avoid thundering-herd synchronized retries. Typical: 500ms → 1s → 2s → 4s, capped at ~30–60s, 2–4 attempts max.
- **Respect `Retry-After`** headers when the provider sends them.
- Model calls and tool calls retry differently: a failed *tool call* may also need a *model re-prompt* (the agent must observe the failure and re-plan), not just a silent repeat.
- Track **retry rate** as an SLI: a rising retry rate is an early warning of a degrading dependency (see [ai_agent_drift_guide.md](ai_agent_drift_guide.md)).

### 3.3 Idempotency: Retry Safety
**Idempotency is what makes retries safe.** A tool call is idempotent if calling it twice has the same effect as calling it once. This is the retry-safety contract:

- **Reads are naturally idempotent** — safe to retry.
- **Mutations are not** — "send payment," "create ticket," "debit account" executed twice is a double-spend. Every mutating tool must accept an **idempotency key** (the agent's task ID + step ID) and deduplicate: the tool checks whether this key was already applied and returns the stored result instead of re-executing. *(Verified — idempotency keys are standard distributed-systems practice, adopted directly into agent tool design.)*
- The **task ID** must be stable across retries: the runtime generates it at task start and threads it through every tool call, so a retried step reuses the same key.
- Test idempotency explicitly: replay a failed run's tool calls against the same keys and assert single application (this belongs in the integration test suite, Section 4).

### 3.4 Timeouts: Per-Step and Per-Task
**Time is the agent's scarcest reliability resource.** A loop without timeouts is a bill without a ceiling:

- **Per-call timeout:** each model call and tool call gets a hard timeout (e.g. model 30s, tool 10s) — the agent never waits indefinitely on a hung dependency.
- **Per-step timeout:** each agent step (plan → act → observe cycle) gets a budget; exceeding it triggers the fallback ladder.
- **Per-task timeout:** the whole task has a deadline (e.g. 60s for a support interaction); on expiry the agent stops, saves state, and hands off.
- **Max-steps policy:** the loop counter is the agent-specific timeout — a hard cap on reasoning iterations (e.g. 10 steps for a support agent). Exceeding it is a **loop event**: stop, log, alert, escalate. Loop rate is an SLI (Section 1.3). *(Verified — max-iteration caps are universal production-agent practice; they exist precisely because agents loop.)*
- **Streaming + cancel:** for long generations, stream tokens so the user sees progress and the runtime can cancel mid-generation when a step times out — the difference between a 30s hang and a 2s fail-fast.

### 3.5 Circuit Breakers
**When a dependency is failing, stop hitting it — and stop the agent from flailing against it.** The circuit breaker pattern (from distributed-systems resilience) applies directly to agent tools and providers: *(Verified — circuit breakers on external tools/APIs are consensus practice in production agent engineering.)*

- **Closed:** the tool is healthy; calls flow.
- **Open:** after N consecutive failures (or an error-rate threshold within a window), the breaker opens — subsequent calls *fail fast* (no attempt) for a cooldown period. The agent gets an immediate, predictable "tool unavailable" and takes the fallback path (Section 3.6) instead of burning retries and tokens against a dead dependency.
- **Half-open:** after the cooldown, a small probe traffic passes through; success closes the breaker, failure reopens it.

For agents specifically:

- Break per **tool**, per **provider**, per **model** — a dead search API should not take down the whole agent.
- A breaker protects two things: the dependency (from a retry storm) and the **budget** (a failing tool consumed tokens + time with zero progress; the breaker converts "slow death" into "fast fallback").
- Circuit state is **shared across agent instances** (in the runtime/platform layer — see [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md)): each agent instance breaking its own circuit independently multiplies the retry storm by the instance count.

### 3.6 Degraded Modes: The Fallback Ladder at Runtime
Section 2.5 defined the fallback *design*; this is the *runtime*: the degraded mode is a **pre-programmed response to circuit state, budget state, and verifier state**:

| Runtime condition | Trigger | Degraded behavior |
|---|---|---|
| Provider down | Circuit open on provider | Route to fallback provider/model (router config) |
| Tool down | Circuit open on tool | Disable the tool's tools from the agent's schema; answer from deterministic data; say what you cannot do |
| Model quality bad | Verifier failure rate > threshold | Raise HITL rate, tighten verifier, alert (see [ai_agent_drift_guide.md](ai_agent_drift_guide.md)) |
| Budget nearing cap | Token/step counter > threshold | Switch to cheaper model path, reduce steps, or stop + escalate |
| Latency breach | p95 breach window | Shed load, disable expensive steps (RAG re-rank), cache aggressively |

Degraded modes are **tested** (chaos, Section 4.4) and **logged** — every time the agent degrades, that is an observable event, not a silent improvisation.

### 3.7 State Management: Persistence and Recovery
**An agent is a state machine, and its state is the only thing that survives a crash.** Production agents persist state at step boundaries:

- **What is persisted:** task ID, goal, the conversation/step transcript, tool results, the plan, budget/step counters, and the next action to take. Enough to resume.
- **Why:** agents are long-running (support conversations, investigations, multi-hour workflows). A crash, a redeploy, or a provider timeout mid-task must not lose the task. **Checkpoint-and-resume** (durable execution) beats restart-from-zero on cost, latency, and user experience. *(Verified — durable execution/checkpointing is standard practice for long-running agents; see [durable_ai_agent_workflows_guide.md](../durable_ai_agent_workflows_guide.md).)*
- **Where:** state lives in the runtime's state store (Redis/Postgres/durable-execution engine), keyed by task ID, versioned, and TTL'd per retention policy. For a bank, state *is* a data-protection artifact: it may contain PII and must be encrypted at rest and in transit (Section 8.3).
- **Recovery semantics:** on restart, the agent loads the checkpoint, verifies the plan is still valid (tools may have changed), and resumes at the next step — re-running *idempotent* steps if needed, never re-running *mutating* ones (the idempotency keys make this safe).
- **Stateless-when-possible:** the cheapest state is no state. Prefer agents whose steps are stateless functions over a persisted transcript, so horizontal scaling and crash recovery are trivial; persist only what must survive.

### 3.8 The Reliability Patterns (Resilience Patterns, Agent-Applied)
The classic resilience patterns apply to agents with specific agent-flavored implementations. *(Verified — the retry/circuit-breaker/bulkhead/fallback quartet is standard; the agent implementation details below are the synthesis.)*

| Pattern | Failure it handles | Agent implementation |
|---|---|---|
| **Retry** | Transient failures (429, 5xx, blips) | Exponential backoff + jitter, idempotent-safe only, Retry-After respected |
| **Circuit breaker** | Persistent dependency failure | Per-tool/per-provider breakers; fail-fast + fallback path; shared state across instances |
| **Bulkhead** | One agent/task exhausting shared resources | Per-agent rate limits and concurrency caps; separate model quotas; per-tenant isolation (a runaway customer agent can't starve the trade agent) |
| **Fallback** | Step failure of any kind | Simpler model → deterministic → reduced scope → human handoff (Section 2.5) |
| **Timeout / deadline** | Hangs, loops, provider stalls | Per-call, per-step, per-task timeouts; max-steps policy; cancel via streaming |
| **Idempotency** | Duplicate side effects from retries | Idempotency keys on all mutating tools; task-ID threading |
| **Bulkhead (agent-specific)** | Prompt-injection/cost-bomb blast radius | Per-task budget caps; per-agent token throttles; sandboxed tool execution |
| **Checkpoint/state persistence** | Crashes, redeploys, mid-task failures | Durable state at step boundaries; resume-from-checkpoint |

### 3.9 The Reliability Table
| Pattern | Failure handled | Implementation | Where enforced |
|---|---|---|---|
| Retry + backoff | Transient API/tool failures | 500ms→4s backoff + jitter, ≤4 attempts, transient-only | Model/tool call layer |
| Idempotency | Duplicate mutations on retry | Idempotency keys on mutating tools | Tool registry ([agent_scaffolding_guide.md](agent_scaffolding_guide.md)) |
| Timeouts | Hangs, stalls | Per-call 30s / per-step / per-task; cancel via streaming | Runtime loop |
| Max-steps | Reasoning loops | Hard step cap (e.g. 10); loop event = alert + escalate | Runtime loop |
| Circuit breaker | Dead tools/providers | Per-tool open/half-open/closed; shared circuit state | Runtime/platform layer |
| Degraded modes | Any dependency failure | Fallback ladder: simpler model → deterministic → reduced scope → HITL | Agent config + runtime |
| State persistence | Crashes/redeploys | Checkpoint at step boundaries; resume; idempotent replay | Runtime state store |
| Bulkhead | Resource exhaustion / blast radius | Per-agent rate limits, concurrency caps, budget caps | Platform/gateway layer |
---
## 4. Testing
### 4.1 Eval-Driven Testing: Golden Sets and Regression
**The eval set is the agent's contract with its own future.** Testing a production agent means testing its *behavior* (what it does with real inputs) — not just its code. The foundation is the **golden set**: a curated, labeled, versioned collection of representative tasks with expected outcomes, used to measure and compare agent versions.

Golden set practice *(verified — golden-set regression evals in CI are consensus 2025–2026 practice; see [rag/rag_evaluation_methodology_guide.md](rag/rag_evaluation_methodology_guide.md) for the retrieval-side methodology, which this section extends to agents)*:

- **Content:** 100–1,000+ real tasks drawn from production traffic (not hand-written fantasies): the common cases, the edge cases, the adversarial cases, the failure cases that actually happened. The failure guide's catalogue is a checklist of what to include.
- **Labels:** for each task, the expected outcome(s): correct answer, correct tool calls, correct refusal, correct escalation. Multi-dimensional scoring beats a single pass/fail (see [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md)).
- **Versioning:** the golden set is versioned with the agent; every release records the scores against the set. **Scores are pinned**: a release may not regress the golden set (see the eval gate, Section 5.1).
- **Growth:** production failures and live-traffic edge cases are **added back** into the golden set (regression capture). A frozen golden set decays — it measures the past, not the present.
- **Tooling:** eval harnesses (see [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md)), LLM-as-judge with calibrated judges, and agentic-trace evals (score the whole trace — plan, tool calls, output — not just the final answer). Statistical honesty matters: a 1-point score change on 50 samples is noise; report confidence intervals before gating on it.

**Regression testing** is the eval gate's engine: every prompt change, model change, tool change, or config change runs the golden set and compares against the pinned baseline. Any regression blocks the change. This is the single most effective reliability investment available — it converts "we hope the change is fine" into "the change is measured."

### 4.2 Pre-Production Testing: Staging, Integration, Load
- **Unit tests** — the deterministic parts are plain code and get plain tests: the workflow skeleton, the verifier rules, the tool wrappers, the retry/backoff logic, the budget enforcement, the prompt *templating* (variables, escaping, rendering).
- **Integration tests** — the agent against real (or realistic-mocked) dependencies: tools, the vector store, the auth layer. The agent's full path — request → intent → tool calls → verifier → response — executes in a staging environment with production-shaped data. Integration tests include: idempotency replay tests (Section 3.3), degraded-mode tests, and permission/allow-list tests.
- **Load tests** — concurrency is where agents surprise you: N parallel agents × M steps × K tokens each hits rate limits, queues, and budgets. Load-test with production-shaped traffic *and* production-shaped step counts: a support agent that averages 6 model calls per ticket under load is 6× the token traffic of a simple Q&A. Verify: throughput at target latency, rate-limit behavior, and cost per task under load. *(Verified — load/concurrency testing for agent services is standard practice; the agent-specific twist — token and step amplification under concurrency — is the synthesis.)*
- **Staging hygiene:** staging must mirror production *configuration* (models, prompts, tool versions — same versions, same prompts) or staging evals measure a different agent than the one shipping. Prompt/model drift between staging and prod is a classic silent failure (see [ai_agent_drift_guide.md](ai_agent_drift_guide.md)).

### 4.3 Canary Testing
**Canary testing = small real traffic, measured against the SLOs, before wider rollout.** *(Verified — canary/progressive rollout for LLM features is established practice; see also Section 5.2.)* The canary is both a deployment strategy and a *testing* stage: the new agent version serves 1–5% of live traffic, and its task success rate, verifier-failure rate, cost per task, and latency are compared against the incumbent *on real traffic* — the only test that sees the real distribution.

- **Shadow mode** (the prerequisite): run the candidate against production traffic but discard its outputs — validate safety and behavior with zero user impact, and diff candidate vs. incumbent on the same inputs.
- **Canary:** promote to a small live slice; promote further only while SLOs hold; auto-rollback on SLO breach (Section 5.4).
- **Statistical honesty:** compare on matched pairs (same request to both versions where possible) and use significance thresholds — canary decisions on 200 samples are noise (see [rag/rag_evaluation_methodology_guide.md](rag/rag_evaluation_methodology_guide.md) for the sampling discussion).

### 4.4 Chaos Testing: Kill the Tools
**The only way to trust the degraded modes is to exercise them.** Chaos testing for agents means deliberately failing the dependencies and asserting the agent takes the designed degraded path: *(Verified — failure-injection/chaos practice for LLM systems is established in engineering orgs; the agent-specific curriculum below is the synthesis.)*

- Kill the LLM provider → assert fallback model path, no 5xx to users, alert fired.
- Kill the vector store → assert retrieval-dependent tools disabled, deterministic path taken.
- Kill a tool API → assert circuit breaker opens, agent takes fallback path, no retry storm.
- Inject 429s / latency → assert backoff behavior, no synchronized retry stampede.
- Corrupt tool outputs (simulated prompt injection, malformed JSON) → assert guardrails catch them, no tool execution of injected instructions.
- Saturate the budget → assert the budget cap stops the agent mid-task and escalates.

Chaos tests run in staging, on a schedule, with a **chaos scorecard**: for each injected failure, did the agent degrade as designed? A failure mode not on the scorecard is a failure mode you will meet in production, unannounced.

### 4.5 The Agent Testing Pyramid
The agent testing pyramid *(verified as the emerging consensus shape — deterministic base, behavioral middle, live top; the exact naming is this guide's synthesis)*:

```
        ▲  CANARY (live)          — real traffic, SLO comparison, 1–5% slice
       ▲▲  EVALS (behavioral)     — golden set + regression gates, per release
      ▲▲▲  INTEGRATION (systems)  — tools, state, idempotency, degraded modes, load
     ▲▲▲▲  UNIT (deterministic)   — workflow skeleton, verifiers, retries, budgets
```

- **Bottom (unit):** the deterministic core — cheap, fast, exhaustive. If it fails, nothing above it matters.
- **Middle (integration):** the agent as a system — tools, state, failures. Expensive but essential; run in CI per PR (scoped) and per release (full).
- **Upper (evals):** the behavioral contract — golden set, regression gates. Run per release and per *any* prompt/model change.
- **Top (canary):** the live truth — small real traffic, measured against SLOs, auto-rollback.

Rule of the pyramid: **a change cannot skip a level.** A prompt tweak that ships straight to prod without the eval level is how production agents regress silently.

### 4.6 The Testing Table
| Stage | What it tests | Tooling / practice |
|---|---|---|
| Unit | Workflow skeleton, verifiers, retries, budget logic, prompt templating | Standard test framework (pytest/Jest), property tests for rules |
| Integration | Tools, state persistence, idempotency, allow-lists, degraded modes | Testcontainers/mocks, staging env with prod-shaped data |
| Load | Concurrency, rate limits, latency under load, cost per task under load | k6/Locust; measure tokens × steps per task |
| Eval (golden set) | Behavioral contract: success rate, verifier failures, refusals, escalations | Eval harness + judges ([llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md), [rag/rag_evaluation_methodology_guide.md](rag/rag_evaluation_methodology_guide.md)) |
| Regression gate | No behavioral regression vs. pinned baseline | CI eval gate; block on regression (Section 5.1) |
| Shadow | Candidate behavior on real traffic, discarded | Traffic shadowing at the gateway |
| Canary | SLOs on small live slice | Progressive rollout, sticky-percent routing, auto-rollback (Section 5.2) |
| Chaos | Degraded modes under real failures | Failure injection; chaos scorecard (Section 4.4) |

---
## 5. Deployment
### 5.1 CI/CD with Eval Gates
**The pipeline is the governance.** Every change to the agent — prompt, model, tool, config, code — flows through CI/CD, and the behavioral gate sits inside CI: *(verified — eval gates in CI are established practice: golden-set runs, regression thresholds, and promotion gating on metrics)*

1. **PR stage:** lint, unit tests, integration tests (scoped). Prompt changes run the golden set (subset for speed, full set for release).
2. **Eval gate:** full golden set + regression comparison against the pinned baseline. **Block on regression**: success rate, verifier-failure rate, refusal accuracy, escalation accuracy, cost per task on the eval set. Thresholds are pre-agreed numbers in the pipeline config, not a human's mood.
3. **Security gate:** dependency scan, secret scan, guardrail tests, injection red-team subset (Section 8).
4. **Build & release artifact:** the agent *bundle* — code + prompt versions + model config + tool versions + golden-set version, all pinned and immutable. The artifact is what deploys, so rollback is a version switch, not a reconstruction.
5. **Deploy stage:** blue-green/canary/feature-flag promotion (5.2), gated on live SLOs.

Prompts are code: they live in the repo, versioned, reviewed, and diffable ([agent_scaffolding_guide.md](agent_scaffolding_guide.md) scaffolds `prompts.py` with versioned prompt constants). A prompt edited in the provider console is a prompt that bypassed every gate.

### 5.2 Deployment Strategies: Blue-Green, Canary, Feature Flags
*(Verified — blue-green and canary deployments are standard; the agent-specific content below is the synthesis.)*

- **Blue-green:** two environments (blue = live, green = new). Deploy green, smoke-test it, switch traffic, keep blue for instant rollback. Good for the *service* (runtime, versioned bundles); the eval gate decides *before* the switch.
- **Canary:** progressive traffic shift — 1% → 5% → 25% → 100% — with an **SLO dashboard per slice**: task success rate, verifier-failure rate, cost per task, p95. Promote only while every metric holds; **auto-rollback** on breach (Section 5.4). Canary is the deployment of choice for agent changes because agent quality is empirical — the golden set says "probably fine," the canary says "actually fine."
- **Feature flags:** the agent's *behavioral knobs* — model version, prompt version, tool enablement, autonomy level (max steps), HITL rate, budget caps — are runtime-configurable flags, not redeploys. Flags let you: enable a new agent for a pilot tenant only; dial autonomy down during an incident without a deploy; A/B two prompt versions live. Feature flags are the operational face of the maturity model (Section 9.2): the same agent artifact can run in "cautious" mode (low autonomy, high HITL) or "scale" mode.
- **Shadow deployments** (run candidate on real traffic, discard output) are the zero-risk pre-stage before canary — especially valuable for agent *behavior* changes where the golden set cannot cover the tail.

### 5.3 Rollback: Model Version, Prompt Version, Everything Versioned
**A production agent is a bundle of versions; rollback is switching the bundle.** Because the artifact pins code + prompt + model + tools + config, rollback is: point the feature flag / router at the previous artifact. Minutes, not archaeology.

- **Model rollback:** model providers change behavior silently between versions and even within a version (temperature of the day). The router holds model versions; on eval/canary/incident signal, flip back. Model changes are *release events* — they get eval-gated like any other change (5.4).
- **Prompt rollback:** prompt versions are immutable artifacts; rollback = flag flip.
- **Data rollback:** the state store and golden set are versioned too — recovery must be able to say "revert to the behavior validated against golden set v7."
- **The rollback test:** every release rehearses rollback. If rolling back requires a deploy, it is not a rollback — it is an incident extension.

### 5.4 The Release Process: Eval Before the Model Swap
The release process for *any* change — especially model changes:

1. **Candidate:** new bundle (prompt v12 + model gpt-5-class-A) built from CI.
2. **Eval gate:** full golden set vs. pinned baseline. A model swap must clear the same bar as a prompt edit — **eval before the model swap, always**. Model "upgrades" regress agents constantly (behavior shifts, format changes, refusal changes); the golden set is the only honest referee.
3. **Shadow:** candidate on real traffic, diffed against incumbent.
4. **Canary:** 1–5% live, SLO-monitored.
5. **Rollout:** progressive to 100%, flags permitting instant dial-down.
6. **Freeze & review:** 48h observation window, then the change is declared stable and the *previous* version becomes the rollback target.

### 5.5 The Deployment Table
| Stage | Practice | Gate |
|---|---|---|
| PR | Lint, unit, scoped integration | CI green |
| Eval gate | Full golden set vs. baseline | No regression on any pinned metric |
| Security gate | Dep scans, injection subset, guardrail tests | Clean |
| Artifact | Immutable bundle: code + prompts + models + tools + config | Built from CI, hash-pinned |
| Shadow | Real traffic, discarded outputs | Behavior diff acceptable |
| Canary | 1–5% live slice, SLO dashboard | All SLOs hold on the slice |
| Rollout | Progressive to 100% via flags | Canary clear + budget OK |
| Rollback | Flag flip to previous artifact | Rehearsed, minutes, automated |

---
## 6. Operations
### 6.1 Monitoring: The Agent Dashboards
**You cannot operate what you cannot see — and an agent is far more see-able than a model.** Agent monitoring instruments the *whole trace*: request → intent → plan → tool calls → verifier → response, with metrics derived from traces. *(Verified — trace-based agent observability with OTel GenAI conventions and Langfuse/LangSmith/Phoenix-class tooling is established practice; see [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) for the platform treatment.)*

The dashboard set (one row per concern):

| Dashboard | Metrics |
|---|---|
| **Success & errors** | Task success rate, error rate by type (tool error, timeout, verifier failure, escalation), refusal rate |
| **Latency** | p50/p95/p99 per task, per step; time-in-tool vs. time-in-model |
| **Cost** | Cost per task, tokens per task (in/out), cost by model, by tool, by agent, by tenant (Section 7.4) |
| **Behavior** | Steps per task, loop rate (max-steps events), tool-call distribution, fallback/degraded-mode event rate |
| **Quality (eval on live)** | Live-sampled eval scores: groundedness, completeness, verifier agreement — sampled, not exhaustive (see [ai_agent_drift_guide.md](ai_agent_drift_guide.md) for drift dashboards) |
| **Dependencies** | Provider/tool health: error rates, latency, circuit states, retry rates |

**The loop-detection monitor** is agent-specific: max-steps events, repeated identical tool calls, step-count percentiles, and budget-exhaustion events. Loops are the agent's signature failure — they burn cost, latency, and trust simultaneously (see [ai_agent_drift_guide.md](ai_agent_drift_guide.md) for drift/loop detection depth, and the failure guide's loop catalogue).

**Tracing:** every run emits a trace: agent spans, model spans (per the OpenTelemetry GenAI semantic conventions — `gen_ai.*` attributes: provider, model, tokens, latency; **emerging/standardizing** — the conventions are maturing and widely implemented in Langfuse, LangSmith, Phoenix, Honeycomb, Datadog-class tools), tool spans, and the trace ID flows into every downstream system (logs, audit trail, support tickets). *(Verified — OTel GenAI semantic conventions exist and are the convergence point for LLM observability.)*

### 6.2 Alerting: SLO Burn, Cost Spikes, Loops
Alerts are derived from the SLOs (Section 1.3) — an alert is the SLO saying "act now."

- **SLO burn alerts:** error budget burning faster than expected (burn-rate alerts: 2x/14x windows — classic SRE technique, directly applicable). Alert on *trend*, not single events: one failed task is noise; a rising verifier-failure rate over 30 minutes is an incident.
- **Cost spike alerts:** cost per task above budget, total spend above daily cap, token counts anomalous (a 10× step-count jump is either a loop or a new user behavior — both need eyes).
- **Loop alerts:** max-steps events per hour above threshold; repeated tool-call identity; stuck agents (no progress despite steps).
- **Degraded-mode alerts:** fallback-path usage rate rising (the agent is quietly living in degraded mode — that is an incident about the *dependency*, not the agent).
- **Quality alerts:** live-eval scores dropping, verifier-failure rate rising, refusal rate shifting — the earliest drift signals ([ai_agent_drift_guide.md](ai_agent_drift_guide.md)).
- **Alert hygiene:** every alert has an owner, a runbook link, and a severity that matches its actionability. An alert nobody acts on is noise that buries the alerts that matter.

### 6.3 Incident Response: The Agent Incident
**An agent incident is any event where the agent violates its contract at a scale that matters** — success rate collapse, wrong-answer wave (verifier failures spiking), cost blow-up, loop storm, data exposure, or a dependency outage cascading into user-facing failures. *(Verified — treating agent failures as incidents with the standard IR lifecycle is established; what differs is the agent-specific content below.)*

The agent incident lifecycle:

1. **Detect** — alert fires (6.2). Trace IDs let you pull every affected run in seconds.
2. **Contain** — the first action is *de-risk, then diagnose*: **feature flag the agent to degraded mode** (lower autonomy, higher HITL, fallback model) or **rollback the bundle** (Section 5.3). Containment for agents is unusually cheap because flags and versioned bundles exist — use them before root-causing.
3. **Diagnose** — with traces: which prompt/model/tool/input class? Was it a model behavior shift, a tool outage, a prompt regression, an injection, or drift? Compare against the last-known-good bundle on the golden set — the eval harness doubles as the incident forensic tool.
4. **Resolve & verify** — deploy the fix through the *same gates* (eval first — an incident fix that regresses the golden set is a second incident); verify on the canary.
5. **Postmortem** — the failure guide's catalogue is the taxonomy: which failure mode fired, what design gap let it through, what gets added to the golden set and the chaos scorecard. **Every agent incident grows the golden set** — this is the closed loop that makes the agent measurably more robust over time.

**The runbook** is the pre-written incident playbook: symptoms → likely causes → containment steps (exact flag/config commands) → diagnostic queries → escalation paths. Every alert has one. An agent runbook is *more* valuable than a classic service runbook because agent failure modes are non-obvious (the system looks fine while being wrong).

### 6.4 On-Call: The Agent On-Call
**Agent on-call is a distinct discipline** *(emerging practice — "LLM SRE" is consolidating; treat the specifics below as the 2026 synthesis)*. The on-call engineer for agents needs a different toolkit than classic service on-call:

- **The agent SRE's core skill:** reading traces and *reasoning about model behavior*, not just stack traces. "Why did the agent call the wrong tool?" is a prompt/model/guardrail question, not a code question.
- **The autonomy dial:** the on-call's primary tool is the **autonomy/risk dial** — reduce max steps, raise HITL rate, switch to the conservative model, disable a tool — not "restart the pod." The incident response for "agent misbehaving" is usually *turn down the autonomy*, which no classic SRE runbook covers.
- **Escalation tiers:** Tier 1 (agent engineer, flag/rollback authority), Tier 2 (platform + model team for provider/dependency incidents), Tier 3 (data/security for exposure incidents). Pre-agree who has flag authority — in an incident, waiting for approval to flip a safety flag is the classic failure.
- **Humans are part of the agent's availability:** if the agent escalates to humans (HITL queue), the humans are a dependency with their own SLO — an overflowing review queue is an agent incident too. On-call coverage for the *human review queue* is a real operational requirement, often forgotten.

### 6.5 The Operations Table
| Signal | Alert | Response |
|---|---|---|
| Task success rate < SLO | SLO burn alert | Diagnose via traces; flag to degraded mode; rollback if regression |
| Verifier-failure rate spike | Quality alert | Pull affected runs; check model/prompt change recency; fix or rollback |
| Cost per task > budget | Cost spike alert | Check steps/tokens per task; loop hunt; tighten budget; route to cheap model |
| Max-steps events rising | Loop alert | Contain loops (kill flag); add inputs to golden set; fix prompt/tool |
| Tool/provider error rate rising | Dependency alert | Check circuit states; fallback path confirm; provider incident coordination |
| Fallback usage rate rising | Degraded-mode alert | Treat as dependency incident; restore primary; review why degraded for so long |
| Live-eval score dropping | Drift alert | Drift playbook ([ai_agent_drift_guide.md](ai_agent_drift_guide.md)); retrain/reprompt; re-eval |

---
## 7. Cost Control
### 7.1 Token Budgets: Per-Task Caps
**The budget cap is the agent's financial circuit breaker.** Every agent has a **per-task token/dollar cap** — a hard ceiling on what one task may consume, enforced in the runtime, not hoped for. *(Verified — per-task budget caps and per-agent token throttles are consensus practice; the cap is simultaneously an anti-loop mechanism, a FinOps control, and a security control against injection-driven cost bombs.)*

- **Cap structure:** per-task max tokens (in + out), max steps, max tool calls, and max dollars. For a support ticket: 50K tokens / 10 steps / 12 tool calls / $0.50 hard ceiling.
- **Enforcement:** counters incremented at every model call and tool call, checked *before* each step. On exhaustion: stop, save state, log a budget event, escalate or answer with the deterministic path — never a silent partial answer.
- **Budget layering:** per-task cap (hard) + per-agent daily cap (hard) + per-tenant/per-team monthly budget (soft, alerting) + organization-wide spend ceiling. The layers bound the blast radius of any single runaway (see [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) for the platform-level enforcement).
- **Budget events are metrics:** budget-exhaustion rate tells you which tasks are systematically over-budget (prompt too big? wrong model? tool loop?) — the data for the next optimization.

### 7.2 Caching: Prompt Caching and Response Caching
*(Verified — prompt caching is a first-class provider feature: Anthropic's prompt caching discounts cached input tokens by up to 90%; OpenAI's automatic/variable caching discounts cached input by ~50%; see [agent_runtime_cache_design_guide.md](agent_runtime_cache_design_guide.md) for the full design treatment, and [llm_latency_optimization_guide.md](llm_latency_optimization_guide.md).)*

- **Prompt caching:** the system prompt + tool schemas + conversation prefix are the same across calls — providers cache the prefix's KV state, and cached input tokens cost a fraction. Agents are *ideal* prompt-caching workloads: long stable prefixes, many calls per task, repeated tools. Cache the invariant prefix; keep the mutable tail (user input, latest tool results) short.
- **Response caching:** deterministic-safe requests (identical inputs, cacheable intents — FAQ-class queries, status lookups) can serve from a response cache keyed by a semantic hash of the request. **Never cache** anything user-specific, time-sensitive, or mutating — a cached "balance" or cached "payment confirmation" is a data-protection incident waiting to happen. Response caching is most valuable at the *router* layer: the router's classification of a known request type can be cached, skipping the LLM entirely.
- **Cache placement:** provider-side (prompt caching, automatic), gateway-side (shared cache across agent instances — a 10× traffic agent fleet shares one cache), and agent-side (in-task memoization: same tool result needed twice = fetched once).

### 7.3 Model Routing: Cheap for Simple, Strong for Hard
**Route the task to the smallest model that can do it.** *(Verified — model routing is established practice; a router (classifier or small LLM) sends simple queries to cheap models and hard ones to strong models, with reported savings of 50–70% on routed workloads; see [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) for the platform router design.)*

- **The routing tiers:** a support agent might run: tiny model for intent classification and slot filling; mid model for routine answers (balance, status — the 80% volume); frontier model only for hard reasoning (disputes, edge cases). The *step-level* version: within one task, classify with the small model, draft with the mid model, and call the frontier model only when the verifier rejects the mid model's output (escalate-on-failure routing).
- **Routing correctness is an eval problem:** the router misclassifying a hard task to the cheap model is a quality regression — the router's confusion matrix is part of the golden set (routing decisions get eval'd like answers).
- **Routing and fallback compose:** the fallback ladder (Section 2.5) *is* routing in reverse — the cheap model is both the cost optimization and the degraded mode. One mechanism, two purposes.
- **The frontier-model tax:** defaulting everything to the strongest model is the most common cost mistake in agent systems. Measure the incremental value of the frontier model per task class (A/B on the golden set: does the frontier model beat the mid model on *this* class?) and route accordingly — most task classes do not need it.

### 7.4 Cost Observability: Attribution and FinOps
**You can only optimize what you can attribute.** Cost observability means every dollar is traceable to: agent, tenant/team, task class, model, tool, and prompt version. *(Verified — per-call metering with token/price/attribution is standard in agent platforms; FinOps practice applies directly — see [finops_guide.md](../finops_guide.md) and [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md).)*

- **Metering:** the model gateway meters every call: tokens in/out, model, price, agent ID, tenant ID, trace ID. Cost is *derived* from the metered call records, not estimated afterwards.
- **Cost per task (the unit that matters):** sum all calls + tool fees + retries + human-review time for one completed task. This is the number the business understands ("it costs us $0.40 to resolve a balance query") and the number SLOs are set on (Section 1.3).
- **Chargeback/FinOps:** per-agent, per-team attribution enables budget ownership — each LOB pays for its agents and optimizes them; the platform team publishes unit costs per task class. Anomaly detection on the spend series catches cost bombs in minutes (Section 6.2).
- **The cost-quality dashboard:** cost per task × success rate per task class is the decision surface for routing and model choices — "class X costs $0.15 at 96% success on the mid model; the frontier model costs $0.90 at 97%." Most teams take the $0.15.

### 7.5 The Cost Controls Table
| Control | Mechanism | Saving |
|---|---|---|
| Per-task token/dollar cap | Runtime counters, hard stop + escalate | Bounds runaway loops & cost bombs; kills worst-case spend |
| Prompt caching | Cache stable prefix KV state | Up to ~90% off cached input tokens (Anthropic), ~50% (OpenAI) |
| Response caching | Semantic-hash cache for deterministic-safe intents | Eliminates repeated identical work; big on FAQ/status traffic |
| Model routing | Classifier routes simple→cheap, hard→strong | 50–70% on routed workloads (reported) |
| Escalate-on-failure routing | Cheap model first, strong model on verifier rejection | Front-loads cheap calls; frontier only when needed |
| Step/tool budgets | Max steps + max tool calls per task | Cuts loop-driven spend; forces efficient plans |
| Cost attribution | Per-call metering + per-task aggregation | Enables FinOps, chargeback, anomaly alerts |
| Cost-quality reviews | Unit-cost per task class vs. success rate | Data-driven model/tier choices |
---
## 8. Security and Governance
### 8.1 The Security Posture: Assume Hostile Inputs
**The threat model for a production agent is: the input is hostile, the tool outputs are hostile, and the model is a fallible intermediary with tool access.** The failure guide's security catalogue (injection, data leakage, tool abuse) is the "why"; this section is the "how." The three defense layers:

1. **Input/output filters (guardrails)** — enforced at the runtime boundary ([llm_guard_models_guide.md](llm_guard_models_guide.md)).
2. **Tool allow-lists and permissioning** — the agent can only call what its role allows ([mcp_framework_tools_guide.md](mcp_framework_tools_guide.md)).
3. **Prompt-injection discipline** — tool outputs and retrieved content are *data*, never instructions.

### 8.2 Injection Defense
**Prompt injection is the agent's signature vulnerability**: the model cannot reliably distinguish "instructions from the operator" from "text that happens to look like instructions" inside tool outputs, retrieved documents, or user content. *(Verified — prompt injection is the most widely documented agent attack class; the defenses below are consensus practice; see [prompt_injection_guide.md](prompt_injection_guide.md) and [llm_development_risks_security_guide.md](../llm_development_risks_security_guide.md) for depth.)*

Defense in depth, each layer independent:

- **Structural separation:** system instructions and tool outputs are *marked* (delimiters, structured fields) — marking is not protection, but it gives downstream checks something to key on.
- **Trust-tiering:** content is classified by trust: (1) operator instructions — the only tier that may instruct; (2) tool outputs / retrieved data — *untrusted data*, may never trigger tool calls or mutate state; (3) user input — untrusted, may influence but not instruct. Enforcement: **tool-call policy checks** — every tool call's *arguments* must be validated against the task's data model, so even a successful injection cannot smuggle a forbidden parameter.
- **Tool-call allow-list + parameter validation:** the runtime validates each call against the agent's permission set (which tool, which resource scope, read vs. write) *at execution time*, regardless of what the model "decided." An injected "call the transfer tool" dies at the permission check even if it survives the prompt.
- **Output-side verification:** guard models scan outputs for PII leakage and injection artifacts before anything is released or logged (see [llm_guard_models_guide.md](llm_guard_models_guide.md)).
- **Red-team testing:** a standing injection test suite (hostile user inputs, hostile documents, hostile tool outputs) in the CI security gate and the chaos scorecard. The test set grows with every real-world injection attempt seen (Section 4.4).
- **Containment:** even a successful injection is bounded by the *capability* design — the agent's tools, its budget, its sandbox. A read-only balance agent that gets injected can be manipulated into wrong answers; it cannot be manipulated into transferring money, because the transfer tool is not in its allow-list. Capability containment is the backstop of all injection defense.

### 8.3 Data Protection
**Agents amplify data risk: they ingest, hold, and emit more data per interaction than any classic service.** The data-protection requirements:

- **PII minimization & redaction:** the agent sees only the data its task requires (a support agent sees the account's *ticket-relevant* fields, not the full customer file). PII is redacted from prompts before model calls where possible, and from logs/traces by default — traces are a data store and must be treated as one (see the OTel practice of prompt-content sampling with PII redaction).
- **Encryption:** data at rest (state store, memory, traces, golden sets — golden sets containing real production data are *the* forgotten data store) and in transit. Encryption keys managed by the platform's KMS, not the agent.
- **DLP (Data Loss Prevention):** output filters block PII/formulas/credentials from leaving the agent in responses, logs, or tool calls; guard models scan the output channel; policy blocks model-training data flows entirely (no production data to training pipelines).
- **Retention & deletion:** state, traces, and audit records have retention policies; "delete the user's data" must be operable (the audit trail is the legal exception — it is retained, the operational data is deleted).
- **Tenant isolation:** in a multi-tenant deployment, one tenant's data never reaches another's context window — isolation is enforced at the platform layer, and *tested* (a chaos/red-team case: tenant A's data in tenant B's response).

### 8.4 Governance: Compliance, Audit Trails, Approval Workflows
**Governance is the operational face of responsible AI** ([implementing-responsible-ai.md](implementing-responsible-ai.md)); in banking it is also the regulatory face (MAS/EBA expectations on AI use; see [financial_risk_compliance_systems_guide.md](../../banking/financial_risk_compliance_systems_guide.md)). The mechanisms:

- **The audit trail:** an append-only, tamper-evident record of *every* agent action: task ID, request, model calls (prompt + model + version), tool calls (tool + arguments + result), verifier results, HITL decisions (who approved what), budget events, degraded-mode events, and the agent/prompt/model versions that produced them. The trace ID is the join key. In a bank, the audit trail is not a nice-to-have — it is the answer to "what did the AI do and who is accountable?" *(Verified — immutable audit logging for agent platforms is established enterprise practice; the platform guide's audit design applies directly.)*
- **Approval workflows:** HITL checkpoints (Section 2.6) are governed: who may approve which action class, separation of duties (the person who requested the action is not the approver), and approval decisions themselves are audited.
- **The agent registry:** every agent in production is registered — owner, purpose, scope, tools, model, SLOs, budget, risk tier, approval authority — and the registry is the source of truth for what agents exist (the answer to shadow-AI). Registry entries are *gated*: no registry entry, no production traffic.
- **Compliance gates in the pipeline:** the CI security gate (Section 5.1) includes compliance checks: guardrail tests, injection suite, PII-redaction tests, audit-log completeness (a release that stops logging is a release that fails the gate).
- **Human accountability:** an agent does not "decide" — it proposes within a governed frame. Every consequential action traces to: the agent's approved scope, the approving human (for HITL actions), and the owning team. This is what makes agent autonomy acceptable to regulators and auditors: autonomy *within a governed envelope*.

### 8.5 The Security Table
| Control | Threat it mitigates | Implementation |
|---|---|---|
| Input filters / guard models | Injection, malicious content in | Guard-model screening of inputs ([llm_guard_models_guide.md](llm_guard_models_guide.md)) |
| Output filters / guard models | PII leakage, forbidden content out | Output scanning + DLP before release/logging |
| Tool allow-lists | Tool abuse, capability escalation | Per-agent permissioned tool registry ([mcp_framework_tools_guide.md](mcp_framework_tools_guide.md)) |
| Tool-call parameter validation | Injected tool arguments | Schema-validate every call's args at runtime |
| Trust-tiering (untrusted tool output) | Prompt injection via tools/data | Tool outputs never instruct; policy checks on all calls |
| Capability containment | Blast radius of any compromise | Narrow scope + read-only defaults + budget caps |
| PII redaction & minimization | Data exposure in prompts/logs | Redact before model calls; redact traces by default |
| Encryption (at rest/in transit) | Data theft | Platform KMS; state/traces/golden sets encrypted |
| Retention & deletion | Data over-retention | TTL policies; operable deletion |
| Tenant isolation | Cross-tenant leakage | Platform-level isolation; tested |
| Audit trail | Accountability gaps | Append-only, tamper-evident, trace-joined ([implementing-responsible-ai.md](implementing-responsible-ai.md)) |
| Approval workflows | Unauthorized high-value actions | Risk-tiered HITL, separation of duties |
| Agent registry | Shadow AI | Gated registration; no registry, no traffic |
| Red-team suite | Injection & abuse regressions | Standing adversarial tests in CI + chaos |

---
## 9. The Production-Readiness Framework
### 9.1 Readiness Gates: The Release Gates
**A readiness gate is a pre-agreed, evidence-based checkpoint that a release must pass.** Gates make "is it production-ready?" a *decision procedure* instead of a debate. The standard gate set:

| Gate | Criteria | Evidence |
|---|---|---|
| **Eval gate** | Golden-set scores meet targets; no regression vs. baseline | Eval report pinned to golden-set version |
| **SLO gate** | Service + task SLOs defined, measurable, and met in staging | SLI dashboard, error-budget baseline |
| **Cost gate** | Cost per task ≤ budget; budget caps enforced and tested | Cost report from metered runs; budget tests green |
| **Security gate** | Guardrails, injection suite, redaction, allow-lists pass | Security review + CI security gate green |
| **Reliability gate** | Timeouts, retries, circuit breakers, degraded modes implemented and chaos-tested | Chaos scorecard, integration tests green |
| **Governance gate** | Registry entry, audit trail verified, HITL/approvals defined, compliance checks pass | Registry record, audit-log test, approval workflow test |
| **Operability gate** | Dashboards, alerts, runbooks, on-call defined and exercised | Alert drill, runbook walkthrough, on-call roster |
| **Launch decision (go/no-go)** | All gates green + sign-off from owner, security, compliance | Signed launch checklist (Section 10.8) |

Each gate has an **owner** (who can veto) and an **evidence artifact** (what must exist). Gates without evidence are theater; evidence without gates is archaeology.

### 9.2 The Maturity Model: Pilot → Production → Scale
**Production-readiness is a journey with defined stages — the maturity model is the map** *(verified as the common shape: pilot → production → scale; the gate criteria per stage below are the synthesis)*:

**Stage 1 — Pilot.** Bounded users, bounded scope, heavy HITL, live monitoring. *Purpose:* prove value and collect real traffic for the golden set. *Defaults:* small tenant set, read-only tools where possible, high approval rate, canary-style rollout from day one, manual go/no-go per feature. *Exit criterion:* pilot traffic meets the SLOs for a sustained window (e.g. 4 weeks), golden set grown from pilot failures, cost per task measured and under budget.

**Stage 2 — Production.** Full target users, autonomous within the governed envelope, full observability, alerting live, runbooks rehearsed. *Purpose:* deliver the value at contract. *Defaults:* autonomy dial raised per risk tier, HITL only at action boundaries, eval gates mandatory for all changes, on-call rotation staffed. *Exit criterion:* SLOs met for a sustained window at production scale; incident rate trending down; error budget healthy.

**Stage 3 — Scale.** Multiple agents, platform-level routing, FinOps chargeback, portfolio-level governance, drift monitoring automated. *Purpose:* multiply the value across the organization without multiplying the operational burden. *Defaults:* everything rides the platform ([enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md)); new agents onboard through the platform's gated registry; per-team budgets and dashboards; drift detection automated ([ai_agent_drift_guide.md](ai_agent_drift_guide.md)).

**The maturity lever is the autonomy dial.** The same agent artifact runs in all three stages — what changes is the *envelope*: step limits, tool sets, HITL rate, budget caps, model tier. Maturity is "the dial moves up because the evidence says it can," never "the dial moves up because it's Tuesday."

### 9.3 The Production-Readiness Checklist (Positive)
Where the failure guide's checklist is the **negative** ("are you doing any of these dangerous things?"), this is the **positive** ("is the production envelope in place?"). Run it at every release and at every stage transition:

**Design**
- [ ] Scope is narrow: one task family, bounded tools, crisp success criteria
- [ ] Core is deterministic: workflow skeleton in code; model only at the edges
- [ ] Verifier exists for every output class, with a defined failure path
- [ ] Fallback ladder defined for every step; degraded-mode matrix written
- [ ] HITL checkpoints placed at action boundaries, risk-tiered

**Reliability**
- [ ] Retries with backoff + jitter; idempotent tools with keys; transient-only retry
- [ ] Timeouts at call/step/task level; max-steps policy set
- [ ] Circuit breakers on tools/providers; shared circuit state
- [ ] State persisted at step boundaries; recovery from checkpoint tested

**Testing**
- [ ] Golden set exists, versioned, labeled; includes production-failure cases
- [ ] Regression gates run in CI on every prompt/model/tool change
- [ ] Integration tests cover idempotency, allow-lists, degraded modes
- [ ] Load tests at expected concurrency with real step counts
- [ ] Chaos tests cover: LLM down, tools down, 429s, corrupt outputs, budget saturation

**Deployment**
- [ ] CI/CD with eval gate + security gate; immutable versioned bundles
- [ ] Canary/shadow path exists; auto-rollback rehearsed
- [ ] Feature flags for model, prompt, autonomy, HITL rate
- [ ] Rollback drill: rollback of last release took minutes, not deploys

**Operations**
- [ ] Dashboards: success, latency, cost, behavior, quality, dependencies
- [ ] Alerts: SLO burn, cost spikes, loops, degraded-mode, drift — all with owners
- [ ] Runbooks for every alert; incident lifecycle rehearsed (game day)
- [ ] On-call staffed; autonomy-dial training done; human-review queue covered

**Cost**
- [ ] Per-task budget cap enforced; per-agent daily cap; monthly budgets set
- [ ] Prompt caching and response caching designed (or deliberately skipped)
- [ ] Model routing in place; cost per task measured and attributed
- [ ] Cost dashboards per agent/team; anomaly alerts live

**Security & Governance**
- [ ] Guardrails: input/output filters, tool allow-lists, parameter validation
- [ ] Injection red-team suite passing; trust-tiering enforced
- [ ] PII redaction, encryption, DLP, retention all implemented and tested
- [ ] Audit trail verified complete; registry entry exists; approvals defined

### 9.4 The Framework Table
| Gate | Criteria | Evidence |
|---|---|---|
| Eval | Golden set ≥ targets, no regression | Eval report (pinned set) |
| SLO | SLIs defined, measured, met in staging | SLI dashboard + error budget |
| Cost | Unit cost ≤ budget; caps tested | Metered cost report; cap tests |
| Security | Guardrails, injection, redaction pass | Security review + CI gate |
| Reliability | Resilience patterns + chaos-tested degraded modes | Chaos scorecard |
| Governance | Registry, audit, approvals in place | Registry record + audit test |
| Operability | Dashboards, alerts, runbooks, on-call live | Drill + walkthrough record |
| Go/No-Go | All gates green + sign-offs | Signed checklist (§10.8) |

---
## 10. Worked Example: Launching a Banking Customer-Support Agent
> This is the **positive** version of the failure-guide scenario (same bank, same agent — this is what "done right" looks like). Read it against the failure guide's version of the same story: where that story died of unbounded scope and missing gates, this one ships.

### 10.1 The Scenario
**Banque de l'Exemple** (a retail bank) wants an AI customer-support agent for the **balance-and-transaction query** channel: customers ask "what's my balance?", "where did this charge come from?", "when did this payment post?" over web chat. The business case: deflect 40% of these tickets from human agents. The constraint: it is a bank — wrong answers and data exposure are regulatory events, not bugs.

### 10.2 The Requirements (SLOs)
Agreed with the business and risk teams before a line of code is written:

| Requirement | Target |
|---|---|
| Task success rate (verified-correct answer) | ≥ 95% (30-day window) |
| Error rate (wrong or failed) | ≤ 5% |
| Cost per resolved ticket | ≤ $0.50 |
| p95 latency | ≤ 10s |
| Loop rate (tasks > 20 steps) | < 1% |
| Service availability | ≥ 99.9% |
| Escalation rate to human | ≤ 20% (and every escalation answered within 15 min) |
| Security | Zero data exposure on red-team suite; injection test pass |

### 10.3 The Design (Per Section 2)
- **Narrow scope:** one task family — balance/transaction *queries*. Read-only tools only. The router (a small classifier) sends balance/transaction questions here; everything else goes to the human queue or other agents. The agent cannot write anything.
- **Deterministic core:** the workflow is code: *authenticate caller → validate account entitlement → fetch balances/transactions (core-banking service via the sanctioned API) → [LLM] interpret the free-text question and map it to the fetched data → verifier → answer*. The lookup, the entitlement check, the formatting skeleton are deterministic; the LLM fills the "understand the question and phrase the answer" edges.
- **The verifier:** a rule-based check (all figures in the answer match the fetched data exactly — a *deterministic* check, not a second LLM) + a guard model on the output (no PII leakage, no advice-like phrasing: the agent states balances; it never says "you should"). Any mismatch → regenerate once → else escalate to human with context.
- **Guardrails:** tool allow-list = exactly two tools (`get_balances`, `get_transactions`), parameter-validated, read-only; input filter screens for injection and out-of-scope requests ("change my address" → polite redirect).
- **Fallbacks:** provider down → fallback model; core-banking API down → circuit opens, agent answers "I can't fetch your balance right now" + human handoff; low-confidence → reduced scope (confirm balance only, decline the rest).
- **HITL:** no write actions exist, so approvals are minimal — but *disclosure* checkpoints apply: anything beyond the caller's own account data requires human confirmation, and every escalation is human-answered. High-value tiers (large-sum queries, disputed charges) route straight to humans with the agent's draft as context.

### 10.4 The Testing (Per Section 4)
- **Golden set:** 800 tasks from real (consented, PII-redacted) support tickets: common cases, edge cases (joint accounts, closed accounts, FX charges), adversarial cases (injection attempts, hostile phrasing), and every pilot failure added back.
- **Eval gates in CI:** every prompt/model/tool change runs the golden set; regression blocks the PR. The router's classification accuracy is eval'd too.
- **Integration:** entitlement checks, allow-list enforcement, idempotency (read-only here, but replay-tested), degraded-mode tests.
- **Load:** 50 concurrent users × 6 model calls/ticket — verifies p95 ≤ 10s and the provider rate limits.
- **Chaos:** kill the LLM provider (fallback model path), kill the core-banking API (circuit + reduced scope + handoff), inject hostile tool outputs (guardrails catch), saturate the budget (cap stops the task).

### 10.5 The Deployment (Per Section 5)
- **CI/CD:** PR → unit/integration → **eval gate** → security gate → immutable bundle (code + prompt v3 + model config + tool v2 + golden set v7).
- **Feature flag:** the agent ships behind `agent.balance_query.enabled` with per-tenant targeting; autonomy dial (max steps, HITL rate) is flag-controlled.
- **Shadow → canary:** shadow on real traffic for 2 weeks (outputs discarded, diffed vs. human answers) → canary 2% of chat traffic for 1 week → 10% → 50% → 100%, each step gated on the SLO dashboard.
- **Rollback:** flag off = instant full rollback; bundle rollback rehearsed weekly (measured: 4 minutes).
- **Model change discipline:** a "free" model upgrade candidate ran the golden set and *failed* (regressed refusal phrasing) — it never shipped. Eval before the model swap, always.

### 10.6 The Monitoring and Alerting (Per Section 6)
- **Dashboards:** task success rate, verifier-failure rate, p50/p95/p99, cost per ticket, steps/ticket, loop events, tool-circuit states, fallback-usage rate, live-eval scores, human-review queue depth.
- **Alerts (with runbooks):** SLO burn on success rate → flag to degraded + triage; cost per ticket > $0.50 → step/token hunt; loop events > threshold → kill flag + golden-set add; core-banking API errors → dependency incident; live-eval drop → drift playbook ([ai_agent_drift_guide.md](ai_agent_drift_guide.md)).
- **On-call:** agent engineer on-call with the autonomy-dial authority; human-review queue has its own SLA and coverage.

### 10.7 The Cost Controls (Per Section 7)
- **Budget caps:** 30K tokens / 10 steps / $0.50 per ticket hard cap; daily agent cap $500; alert at 80%.
- **Caching:** prompt caching on the stable system prompt + tool schema (the agent's calls are a textbook prompt-caching workload); response cache for the top-20 canonical queries (e.g. "what's my balance?") — deterministic-safe, per-user-cache-keyed with strict invalidation on balance changes.
- **Routing:** intent classifier on the tiny model; mid model drafts answers; frontier model only on verifier-rejected drafts (escalate-on-failure). Pilot data showed 85% of tickets never need the frontier model.
- **Attribution:** metered per ticket — model, tokens, cost, agent, tenant — feeding the FinOps dashboard and the unit-cost SLO.

### 10.8 The Launch Checklist (Go/No-Go)
| # | Check | Status |
|---|---|---|
| 1 | Golden-set success ≥ 95% on final bundle (v3/prompt) | ☐ |
| 2 | No golden-set regression vs. baseline; eval report pinned | ☐ |
| 3 | Cost per ticket ≤ $0.50 in canary (measured, not estimated) | ☐ |
| 4 | p95 ≤ 10s at expected concurrency (load test + canary) | ☐ |
| 5 | All chaos tests pass; degraded-mode scorecard complete | ☐ |
| 6 | Security: guardrails, injection suite, redaction, allow-lists pass; security sign-off | ☐ |
| 7 | Audit trail verified complete on canary traffic | ☐ |
| 8 | Registry entry, owner, risk tier, approvals on record | ☐ |
| 9 | Dashboards, alerts, runbooks live; on-call staffed; review-queue SLA covered | ☐ |
| 10 | Rollback drill passed; flag-off rehearsed | ☐ |
| 11 | Business + risk + security sign-off | ☐ |
| 12 | **GO** — all boxes checked; else **NO-GO** with named blockers | ☐ |

### 10.9 The Before/After
| Metric | Before (human-only / failed pilot) | After (gated launch, 3 months) |
|---|---|---|
| Balance-query resolution cost | $3.20 per ticket (human) | $0.38 per ticket (agent) — **88% cheaper** |
| Resolution time (p95) | 4 minutes (human queue) | 9 seconds (agent) |
| Success rate (verified-correct) | — | 96.4% (30-day window) |
| Escalation to human | 100% | 14%, answered within SLA |
| Deflection of simple tickets | 0% | 52% of balance/transaction queries |
| Injection/red-team failures | — | 0 in launch suite; 2 real-world attempts caught by guardrails |
| Agent-related incidents | (failed pilot: loop storm, cost blow-up, wrong-answer wave) | 3 minor incidents, all contained by flags/rollback in < 15 min |
| Regression-caught changes | — | 4 prompt/model changes blocked by the eval gate before reaching users |

**The honest footnote:** none of this was magic. The agent is a narrow-scope read-only tool with a deterministic core, a rule verifier, budget caps, and gates everywhere. The 96.4% success rate is the *system's* success rate — model + core + verifier + fallbacks — not the model's. That is the entire point of the playbook.

---
## 11. Summary: Production-Ready Agents in One Page
**The whole playbook, compressed:**

1. **Set the contract first.** Define the SLOs — success rate, cost per task, p95, loop rate — *before* building. SLOs are design inputs, not reporting outputs. Error budgets bound autonomy: burning budget → dial autonomy down.
2. **Design for production: narrow scope, deterministic core, verifiers everywhere.** One agent per task family. Workflows for the core, agents for the edges. Every output passes a second-pass check. Every step has a fallback. High-value actions have HITL checkpoints.
3. **Engineer reliability like a distributed-systems engineer.** Retries with backoff, idempotency keys on mutations, timeouts at every level, max-steps policy, circuit breakers on tools, persisted state, and — above all — *tested degraded modes*.
4. **Test behavior, not just code.** Golden sets, regression gates in CI, integration and load tests, chaos tests that kill the tools. Every incident grows the golden set.
5. **Deploy like the change is dangerous (it is).** Immutable versioned bundles, eval gates before any prompt/model swap, shadow → canary → rollout, feature flags for autonomy, rehearsed rollback.
6. **Operate what you observe.** Trace everything (OTel GenAI conventions), dashboard the SLOs, alert on burn/cost/loops/drift, runbooks per alert, on-call with an autonomy dial, human-review queue covered.
7. **Control cost structurally.** Per-task budget caps, prompt and response caching, model routing (cheap for simple, strong for hard), per-agent/team cost attribution.
8. **Secure the envelope.** Guardrails at the boundaries, tool allow-lists, injection defense via trust-tiering and capability containment, PII/encryption/DLP, and a complete audit trail.
9. **Gate the journey.** Readiness gates with evidence, a maturity model (pilot → production → scale), and a positive checklist that mirrors the failure guide's negative one.
10. **Launch like a bank would.** The worked example: narrow, read-only, gated, canaried, monitored — 96.4% success at $0.38/ticket, because it was engineered, not wished.

### The Final Word
**Production agents are engineering, not magic.** The demo's magic is the model; the production agent's *reliability* is the system around the model — the deterministic core, the verifier, the fallback, the gate, the budget, the trace, the runbook. The model will change, drift, lie, and get injected; the system is what keeps each of those from becoming a user-facing incident. Teams that treat agents as "just an API call with a loop" fail the way the failure guide documents; teams that treat them as distributed systems with a probabilistic component — and engineer them accordingly — ship the numbers in the worked example. The bar is not "make the model smarter." The bar is **"make the system around the model strong enough that the model's failures are contained, measured, and survivable."** That is what production-ready means.

---
## 12. Glossary
| Term | Definition |
|---|---|
| **Production-ready** | An agent whose reliability, availability, cost, latency, security, and governance are specified, measured, and meet agreed targets, and whose changes flow through eval-gated, rollback-able pipelines. |
| **SLO** (Service Level Objective) | The agreed target for a service metric over a window — e.g. "task success rate ≥ 95% over 30 days." |
| **SLI** (Service Level Indicator) | The measured quantity an SLO is set on — e.g. success rate, p95 latency, cost per task, loop rate. |
| **Availability** | The proportion of time the agent *service* is up and answering correctly — including its degraded modes, not just HTTP 200s. |
| **Degraded mode** | A pre-defined, tested behavior for when a dependency or the model fails: fallback model, reduced scope, or human handoff. |
| **Narrow scope** | Designing an agent for one task family with bounded tools and crisp success criteria; the highest-leverage reliability decision. |
| **Deterministic core** | The human-designed, code-implemented skeleton of the agent (lookups, validation, order of operations) that does not depend on the model. |
| **Workflow-first** | Preferring predefined code paths (workflows) over free-form agent loops; using agents only where genuine flexibility is required. |
| **Verifier** | A second-pass check on the agent's output (schema, rules, guard model, evaluator, human) with a defined failure path. |
| **Guardrail** | A boundary-enforced filter on input, tools, or output — code checks, not prompt advice (see [llm_guard_models_guide.md](llm_guard_models_guide.md)). |
| **Fallback** | A pre-defined alternative path (simpler model, deterministic answer, reduced scope, human handoff) taken when a step fails. |
| **HITL** (Human-in-the-Loop) | A human approval gate at an action boundary, with the context needed to veto intelligently (see [implementing-responsible-ai.md](implementing-responsible-ai.md)). |
| **Checkpoint** | Persisted agent state at a step boundary, enabling resume-after-crash without restarting the task. |
| **Retry** | Re-attempting a failed call; safe only for transient errors and idempotent operations. |
| **Backoff** | Exponential delay growth between retries (with jitter) to avoid synchronized retry storms. |
| **Idempotency** | The property that executing an operation twice has the same effect as once; enabled by idempotency keys on mutating tools. |
| **Timeout** | A hard time limit on a call, step, or task; the agent's protection against hangs and loops. |
| **Circuit breaker** | A resilience pattern that stops calling a failing dependency (open circuit) and fails fast into the fallback path until it recovers. |
| **Bulkhead** | Isolation that prevents one agent/task from exhausting shared resources; per-agent limits and quotas. |
| **State management** | Persisting, versioning, and recovering agent state (transcript, plan, counters) across crashes and redeploys. |
| **Golden set** | The curated, labeled, versioned eval tasks that define the agent's behavioral contract. |
| **Regression** | A measured degradation against the golden-set baseline; blocks releases in the eval gate. |
| **Canary** | Progressive rollout: a small live-traffic slice of a new version, SLO-monitored, promoted only while it holds. |
| **Chaos testing** | Deliberately failing dependencies (killing tools/providers) to verify degraded modes under real failure. |
| **Blue-green** | Two environments (live + new) with instant traffic switch and instant rollback for the service layer. |
| **Feature flag** | A runtime-configurable behavioral knob (model, prompt, autonomy, HITL rate) that changes behavior without a deploy. |
| **Rollback** | Switching to the previous versioned bundle; for agents, a flag flip, not a redeploy. |
| **Prompt versioning** | Treating prompts as versioned, reviewed code artifacts that flow through the same gates as code (see [agent_scaffolding_guide.md](agent_scaffolding_guide.md)). |
| **Monitoring** | Continuous measurement of the agent's SLOs and behavior via traces, metrics, and dashboards. |
| **Alerting** | Automated notification when an SLI breaches its target or trend — SLO burn, cost spikes, loops, drift. |
| **Incident response** | The lifecycle for agent failures: detect, contain (flags/rollback), diagnose (traces), resolve (gated), postmortem (grow the golden set). |
| **Runbook** | The pre-written playbook for an alert: symptoms, causes, containment steps, diagnostic queries, escalation. |
| **On-call** | Staffed operational coverage for the agent, including autonomy-dial authority and the human-review queue. |
| **Token budget** | A hard per-task (and per-agent) cap on tokens/steps/cost, enforced in the runtime; the financial circuit breaker. |
| **Caching** | Reusing computed results: prompt caching (provider KV cache on stable prefixes) and response caching (deterministic-safe requests). |
| **Model routing** | Sending each task/step to the cheapest model that can do it; simple→cheap, hard→strong (see [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md)). |
| **Cost attribution** | Metering and assigning every dollar to agent, tenant, team, model, and task class — the basis of FinOps. |
| **FinOps** | The practice of managing cloud/AI spend: budgets, chargeback, unit economics, anomaly detection (see [finops_guide.md](../finops_guide.md)). |
| **Prompt injection** | An attack where hostile content inside user input or tool output manipulates the model into unintended actions (see [prompt_injection_guide.md](prompt_injection_guide.md)). |
| **PII** (Personally Identifiable Information) | Data that identifies an individual; must be minimized, redacted, encrypted, and retained per policy. |
| **DLP** (Data Loss Prevention) | Controls that stop sensitive data from leaving the system via outputs, logs, or tool calls. |
| **Audit trail** | The append-only, tamper-evident record of every agent action — the answer to "what did the AI do and who is accountable?" |
| **Readiness gate** | A pre-agreed, evidence-based checkpoint (eval, SLO, cost, security, governance, operability) that releases must pass. |
| **Maturity model** | The staged journey pilot → production → scale, with the autonomy dial raised only on evidence. |
| **Go/No-Go** | The launch decision: all readiness gates green plus owner/security/compliance sign-off, or launch is blocked with named blockers. |

---
*Part of the LLM/AI Engineering Guides series. Companion guides: [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) (the failure catalogue — the "why" companion), [autonomous_agents_guide.md](autonomous_agents_guide.md), [ai_agent_drift_guide.md](ai_agent_drift_guide.md), [agent_scaffolding_guide.md](agent_scaffolding_guide.md), [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md), [llm_guard_models_guide.md](llm_guard_models_guide.md), [implementing-responsible-ai.md](implementing-responsible-ai.md), [rag/rag_evaluation_methodology_guide.md](rag/rag_evaluation_methodology_guide.md), [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md), [llm_latency_optimization_guide.md](llm_latency_optimization_guide.md), [finops_guide.md](../finops_guide.md).*
