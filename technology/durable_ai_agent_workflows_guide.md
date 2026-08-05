# Durable AI Agent Workflows: A Comprehensive Guide

**Reliable, Recoverable, Stateful Agent Execution — Durable Execution Engines, Workflow Patterns, and Agent-Specific Durability**

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Crédit Agricole CIB
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** LLM/AI Engineering Guides · **Topic:** AI Agent Infrastructure / Workflow Orchestration
> **Focus:** Banking & regulated industries (Singapore, EU, global)
> **Companion Guides:** [Hybrid Multi-Agent Systems](ai_llm/hybrid_multi_agent_systems_guide.md) · [Hierarchical Multi-Agent Frameworks](ai_llm/hierarchical_multi_agent_frameworks_guide.md) · [Research Agents](research_agents_guide.md) · [Agent Runtime Cache Design](ai_llm/agent_runtime_cache_design_guide.md) · [MCP Framework & Tools](ai_llm/mcp_framework_tools_guide.md) · [Event Stream Processing](event_stream_processing_guide.md) · [LLM Latency Optimization](ai_llm/llm_latency_optimization_guide.md) · [Beyond RAG](ai_llm/beyond_rag_guide.md) · [LLM Development Risks & Security](llm_development_risks_security_guide.md) · [Payments Hub](../banking/payments_hub_guide.md) · [Financial Risk & Compliance Systems](../banking/financial_risk_compliance_systems_guide.md)
> **Last Updated:** August 2026

---
## Table of Contents

1. [Why Durability Matters for Agents](#1-why-durability-matters-for-agents)
2. [The Core Problem: Non-Deterministic, Expensive LLM Calls](#2-the-core-problem-non-deterministic-expensive-llm-calls)
3. [The Standard Solution: Workflow / Durable Execution Engines](#3-the-standard-solution-workflow--durable-execution-engines)
4. [The Durable Execution Model](#4-the-durable-execution-model)
5. [Key Concepts of Durable Execution](#5-key-concepts-of-durable-execution)
6. [Engine Landscape: Temporal and Cadence](#6-engine-landscape-temporal-and-cadence)
7. [Engine Landscape: Inngest and Restate](#7-engine-landscape-inngest-and-restate)
8. [Engine Landscape: AWS Step Functions and Azure Durable Functions](#8-engine-landscape-aws-step-functions-and-azure-durable-functions)
9. [Engine Landscape: Conductor, Camunda, and Data Pipeline Orchestrators](#9-engine-landscape-conductor-camunda-and-data-pipeline-orchestrators)
10. [Engine Comparison Table](#10-engine-comparison-table)
11. [The Canonical Agent Workflow as a Durable Workflow](#11-the-canonical-agent-workflow-as-a-durable-workflow)
12. [Pattern 1 — The Durable Agent Loop](#12-pattern-1--the-durable-agent-loop)
13. [Pattern 2 — Human-in-the-Loop (HITL)](#13-pattern-2--human-in-the-loop-hitl)
14. [Pattern 3 — Saga / Compensation](#14-pattern-3--saga--compensation)
15. [Pattern 4 — Fan-Out / Fan-In](#15-pattern-4--fan-out--fan-in)
16. [Pattern 5 — Event-Driven Agents](#16-pattern-5--event-driven-agents)
17. [Pattern 6 — Scheduled / Cron Agents](#17-pattern-6--scheduled--cron-agents)
18. [Pattern 7 — Checkpointed Long-Running Agents](#18-pattern-7--checkpointed-long-running-agents)
19. [Pattern 8 — Versioned Agent Workflows](#19-pattern-8--versioned-agent-workflows)
20. [Agent-Specific Concerns: LLM Calls as Activities](#20-agent-specific-concerns-llm-calls-as-activities)
21. [Agent-Specific Concerns: Tool Calls, External APIs, Human Steps](#21-agent-specific-concerns-tool-calls-external-apis-human-steps)
22. [State Management: Workflow State vs Agent Memory](#22-state-management-workflow-state-vs-agent-memory)
23. [Checkpointing vs Durable Execution; Delivery Semantics](#23-checkpointing-vs-durable-execution-delivery-semantics)
24. [Tooling and Stack: Temporal in Depth](#24-tooling-and-stack-temporal-in-depth)
25. [Tooling and Stack: Inngest, Restate, Step Functions](#25-tooling-and-stack-inngest-restate-step-functions)
26. [Choosing the Engine](#26-choosing-the-engine)
27. [Reference Architecture](#27-reference-architecture)
28. [Case Study: A Durable Loan-Approval Agent](#28-case-study-a-durable-loan-approval-agent)
29. [The Workflow History as an Audit Trail](#29-the-workflow-history-as-an-audit-trail)
30. [Operational Concerns](#30-operational-concerns)
31. [Failure Modes](#31-failure-modes)
32. [Testing Durable Workflows](#32-testing-durable-workflows)
33. [Anti-Patterns and Pitfalls](#33-anti-patterns-and-pitfalls)
34. [The Future: Durable Execution as the Default Substrate for Agents](#34-the-future-durable-execution-as-the-default-substrate-for-agents)
35. [Conclusion](#35-conclusion)
36. [References](#36-references)

---
## 1. Why Durability Matters for Agents

### 1.1 What "Durable" Means for Agent Execution

A **durable AI agent workflow** is an agent run that survives failure. The workflow's state is persisted, execution resumes from the last *completed* step rather than from scratch, and retries are safe because every side-effecting step is idempotent. The term "durable" comes from **durable execution** — a programming model in which a function's execution is recorded as a log of events, so the runtime can reconstruct where it was and continue, no matter how many times the underlying process crashes, is redeployed, or is restarted. For ordinary request/response services, durability is a database concern; for agents, it is the difference between a production system and a demo.

### 1.2 Agents Are Long-Running, Multi-Step, and Failure-Prone

Three properties of production agents make durability non-negotiable:

| Property | Reality for agents | Consequence |
|---|---|---|
| **Long-running** | Minutes to hours to days (research agents, document pipelines, approval workflows with human steps) | Cannot hold state in memory; the process will restart before the run ends |
| **Multi-step** | Sequences of LLM calls, tool calls, API calls, database writes, and human interactions | Each step is a failure point; a chain of 20 steps with 99% per-step reliability fails ~18% of the time |
| **Failure-prone** | LLM API errors (429 rate limits, 5xx, timeouts), tool failures, worker crashes, redeploys, network blips | Failures are the *normal* case, not the exception |

A typical agent run in production is not one LLM call — it is a loop: plan → call LLM → call tool → observe → call LLM again, repeated dozens of times, possibly interleaved with human approval gates. Any of those steps can fail at any time, and the process hosting the loop can die at any point. Without durability, every one of those failure modes loses the entire run.

### 1.3 What Happens Without Durability

Three failure consequences, in increasing order of severity:

1. **The whole run is lost.** A crash mid-workflow means the user restarts the agent from zero. For a 40-minute research run, that is 40 minutes of work discarded — and the user may not even notice until the result never arrives.
2. **Retries restart from scratch — and burn money.** The naive fix for a crash is "just run it again." But an agent run can cost **$1–50 in LLM tokens** (deep research with hundreds of thousands of tokens routinely lands in the tens of dollars). Every restart re-pays that cost, and because LLM calls are non-deterministic, the restarted run may take a different path — different searches, different conclusions.
3. **Partial side effects with lost state.** The agent may have *already done* consequential things before the crash: moved money, sent an email, created a ticket, booked a transfer. The side effect happened; the state describing it is gone. The result is double payments, duplicate emails, or a system that believes a transfer never happened when it did. This is the worst failure mode in banking: **the effect is real, the record is lost.**

### 1.4 What Durability Gives You

Durability converts all three failure modes into a single, boring outcome: the workflow pauses, and when the worker comes back, it *resumes from the last completed step*.

- **State persisted** — every completed step's output is recorded in the engine's event history.
- **Resume, not restart** — after a crash, the engine replays recorded history to reconstruct workflow state, then continues with the next step. Already-paid LLM calls are *not* re-executed.
- **Safe retries** — activities are retried with backoff; because tool side effects are idempotent, a retried activity does not double-execute.
- **Observable** — every step, retry, and failure is visible in the workflow history, which doubles as an audit trail (see Section 29).

For a bank running agentic credit assessment, trade surveillance, or client onboarding, this is not a convenience — it is the property that makes the agent auditable, resumable, and safe to point at money movement.

---
## 2. The Core Problem: Non-Deterministic, Expensive LLM Calls

### 2.1 Why "Just Retry" Is the Wrong Answer for Agents

Traditional batch jobs can be retried from scratch because their steps are deterministic: re-running step 3 after a crash produces the same output. LLM calls break this assumption in two ways:

1. **Non-determinism.** The same prompt can produce different outputs across calls (temperature, model updates, load). A restarted agent does not replay its old reasoning — it *reasons anew*, possibly diverging into a different tool path or a worse outcome.
2. **Cost.** Every re-executed LLM call re-spends tokens. A $10 agent run that crashes at 60% and is restarted from scratch costs $16–20 before it finishes — and may finish *differently*.

The interaction is the killer: **rerunning from scratch gives different results and burns cost.** The naive retry loop is therefore both financially wasteful and semantically unsafe — in regulated workflows, "the second run reached a different credit decision" is a governance incident, not a feature.

### 2.2 The Insight: Record, Don't Re-Execute

The fix is to stop treating an agent run as a *process* and start treating it as a *log*. The workflow engine records every step's output as it completes. On failure, it replays the log to rebuild state and continues — it never re-executes completed work. This is **event sourcing applied to the workflow itself**: the workflow's current state is a deterministic fold over its recorded history, and the history is the source of truth.

LLM calls become **recorded facts**: the first execution of "generate risk summary" produced output X; that output is stored in the history; a crash later in the workflow replays X from the log instead of asking the model again. Non-determinism becomes harmless — the engine does not depend on the model producing the same output twice, because it only ever needs the output *once*, and thereafter reads it from the log.

### 2.3 The Economics

| Scenario | Cost of one failed run (typical) | Cost with durable execution |
|---|---|---|
| Simple Q&A agent, 3 LLM calls | ~$0.05–0.30 | Same — failure happens before completion |
| Research agent, 30–50 LLM calls + retrieval | ~$3–15 | Failed run resumes at the failed step: **no re-spend** on the ~90% of completed work |
| Deep analysis agent (multi-day, HITL) | ~$10–50 | Resume after human approval or overnight crash: **no re-spend** on completed stages |
| High-volume ops (10k runs/day, 5% failure) | $500–2,500/day of wasted retries | Wasted spend drops to near zero; only the failed step's retries cost |

The durable-execution model does not eliminate failures — it eliminates *re-doing work*. That is the entire economic argument.

---
## 3. The Standard Solution: Workflow / Durable Execution Engines

### 3.1 A Mature Category, Newly Relevant to Agents

The problem of "reliable, resumable, stateful multi-step execution" is not new — it is the classic **workflow orchestration** problem, solved for decades in enterprise integration (BPM), batch processing, and microservices choreography. What is new is applying it to *agent loops*. The engine category has converged on a set of products with the same core contract, differing in authoring model, deployment, and ecosystem:

| Engine | Origin / Steward | Authoring | Notes |
|---|---|---|---|
| **Temporal** | Open source (MIT), Temporal Technologies (2019, ex-Uber Cadence team) | Code (Go/Java/Python/TS) | The de-facto standard for durable execution; 3,000+ paying customers incl. Nvidia, Netflix, Snap, Stripe, Coinbase; used by OpenAI |
| **Cadence** | Uber, open source (MIT) | Code (Go/Java) | Temporal's predecessor; still maintained, production at Uber scale |
| **Inngest** | Commercial + open source | Code (TS/Python/Go), steps as functions | Serverless-first; heavily marketed for AI/agent workflows (`step.ai.*`) |
| **Restate** | Open source (BSL-core / Apache-2.0 components), Restate Labs | Code (TS/Python/Java/Kotlin/Go/Rust) | Durable execution + durable RPC + virtual objects; newer, developer-friendly |
| **AWS Step Functions** | AWS, managed | JSON state machines (ASL) | Enterprise default on AWS; Standard (durable) vs Express (high-throughput) |
| **Azure Durable Functions** | Microsoft, managed | Code (C#/JS/Python/PowerShell) | Azure-native; orchestrator functions with checkpoints |
| **Conductor** | Netflix, open source (Apache-2.0) | JSON workflow defs + Java workers | Battle-tested at Netflix; JSON-authoring model |
| **Camunda / Zeebe** | Camunda, open source + enterprise | BPMN 2.0 (visual) + code | Enterprise BPM: human tasks (HITL), DMN decisions, Zeebe cloud-native engine |
| **Prefect / Dagster / Airflow** | Open source | Code / DAGs / assets | Data-pipeline-first orchestration; durable task execution, retries, caching |
| **DBOS** | Open source | Code (TS/Python) | Postgres-backed durable execution; smaller ecosystem (2026) |

### 3.2 Why This Category, Not a Homegrown Queue

Teams often reach for a message queue (Kafka, SQS, RabbitMQ) plus a state database and build "durability" themselves — a re-implementation of a workflow engine with the hard parts (replay determinism, exactly-once semantics, durable timers, saga compensation, history management) done poorly. Those hard parts are genuinely hard. The rule for agent platforms: **use a workflow engine; do not write one.**

### 3.3 Pipeline-First vs Workflow-First

One distinction matters for agent work. **Data pipeline orchestrators** (Prefect, Dagster, Airflow) are built for batch data movement: DAG-shaped, schedule-driven, retry-per-task. They can host agent steps — Prefect 3.x in particular has genuine durable task execution — but they are *pipeline-first, not workflow-first*: no first-class signals, no durable timers as a core primitive, no saga/HITL semantics. Use them when the dominant workload is data pipelines with agent steps embedded; use a workflow engine when the *agent loop itself* is the product (Section 9.3).

---
## 4. The Durable Execution Model

### 4.1 Workflow as Code

In the durable execution model, you write **workflow functions** in a normal language — ordinary Python, TypeScript, Go, or Java. The workflow is the orchestration logic: it calls activities, waits on timers, receives signals, spawns child workflows, and branches on results. The code *looks* like a synchronous function, but the engine executes it as a state machine.

```python
# Temporal-style durable workflow (conceptual)
@workflow.defn
class LoanApprovalWorkflow:
    @workflow.run
    async def run(self, application_id: str) -> str:
        # Each of these calls is a durable step:
        credit = await workflow.execute_activity(credit_check, application_id)
        risk = await workflow.execute_activity(llm_risk_assessment, credit)
        await workflow.execute_activity(fraud_screen, application_id)
        # Human approval: pause here until a signal arrives
        approved = await workflow.wait_for_signal("officer_decision", timeout=48h)
        ...
```

### 4.2 The Recording and Replay Loop

The engine wraps every step boundary and records an **event history**:

1. The workflow worker starts executing the function.
2. At each step (activity call, timer, signal wait, child workflow), the engine appends an event to the history: *step X started*, *step X completed with output Y*.
3. If the worker crashes, the engine marks the workflow as pending.
4. A worker picks the workflow up again and **replays the history**: the engine re-executes the workflow function, but every step boundary is resolved from the recorded event (the recorded output is returned) rather than re-executed.
5. When replay reaches the last completed event, the engine switches to *forward* execution and continues with the next step.

Replay is what makes crashes invisible: from the workflow's perspective, nothing happened — it simply continued.

### 4.3 Determinism Is the Contract

Replay only works if the workflow function is **deterministic**: given the same history, re-executing the code must produce the same decisions. This means workflow code cannot call `random()`, read the wall clock, hash unordered sets, or make network calls directly. Everything non-deterministic — LLM calls, API calls, DB writes, clock reads — must happen inside **activities** (or engine-managed commands), whose outputs are recorded once and replayed thereafter. Section 33 covers the violations that break replay.

### 4.4 Steps Are Engine-Managed Side Effects

The engine does not just record — it *manages* execution: retries activities with backoff, deduplicates activity execution (an activity that completed but whose completion event was lost is not re-run if the output was durably recorded), fires durable timers across restarts, and delivers signals exactly once to the replay. Deterministic orchestration + managed side effects is the whole model.

---
## 5. Key Concepts of Durable Execution

| Concept | Definition | Agent relevance |
|---|---|---|
| **Workflow** | The orchestration logic — a deterministic function that sequences activities, timers, signals, child workflows | The agent's plan/loop, expressed as code |
| **Activity** | A single unit of work executed reliably with retries: an LLM call, an API call, a DB write | Every LLM call and tool call becomes an activity |
| **Signal** | An external event injected into a *running* workflow (async, from outside) | Human approval, new data arrived, user cancelled, webhook |
| **Query** | A synchronous read of a running workflow's state, without side effects | "What is this agent doing right now?" dashboards |
| **Timer / Sleep** | A durable timer that survives restarts and worker crashes | Approval timeouts, escalation windows, scheduled sub-steps |
| **Child workflow** | A sub-orchestration spawned by a parent, with its own history and lifecycle | Parallel research tasks, per-account sub-agents |
| **Saga / Compensation** | Registered per-step undo actions executed when the workflow fails | Reverse a transfer, release a hold, send failure notice |
| **Heartbeat** | Periodic liveness signal from an activity; distinguishes "still working" from "dead" | Long LLM streams, long-running tool jobs |
| **Event history** | The recorded log of every step — the source of truth for state and audit | The audit trail (Section 29) |
| **Search attributes / visibility** | Indexed metadata for finding workflows by state, customer, type | "All stuck loan approvals for customer X" |
| **Versioning** | Markers (patch / getVersion) so in-flight workflows survive code deploys | Safe rollout of new prompts and agent logic |
| **Worker** | The process that executes workflow and activity code | Your agent runtime, horizontally scalable |

## 6. Engine Landscape: Temporal and Cadence

### 6.1 Temporal — The Standard

**Temporal** is the reference implementation of durable execution: an open-source (MIT) workflow engine founded in 2019 by the team behind Uber's Cadence, with SDKs in Go, Java, Python, and TypeScript. It runs self-hosted (Docker Compose, Kubernetes Helm charts, or a bare cluster) or as **Temporal Cloud** (managed, SaaS). As of mid-2026 it reports 3,000+ paying customers including Nvidia, Netflix, Snap, Stripe, and Coinbase, and is widely reported as the orchestration layer inside OpenAI's production tooling — the strongest possible endorsement for *agent* workloads specifically.

**Feature set (all first-class):**

| Feature | What it gives an agent platform |
|---|---|
| Durable timers | Approval timeouts that survive restarts; "wait 24h then escalate" |
| Signals & queries | HITL (Section 13), mid-run context injection, live state reads |
| Child workflows | Parallel sub-agents with isolated histories (Section 15) |
| Saga support | Compensation activities via explicit `try/except` compensation logic |
| Versioning | `getVersion` / patch markers + worker versioning for safe prompt deploys (Section 19) |
| Search attributes | Query running workflows by customer ID, loan type, risk band |
| Visibility + UI | Web UI (`temporal.io`) with a workflow-history viewer — the single most useful debugging tool for agents |
| Multi-language SDKs | Python/TS/Go/Java workers can coexist; polyglot agent teams |

**Cadence** is Temporal's open-source predecessor, still maintained by Uber and used at Uber scale (tens of millions of workflows/day historically). Teams starting fresh in 2026 should choose Temporal (active development, larger ecosystem, Cloud offering); Cadence matters for organizations that already run it.

### 6.2 When Temporal Fits Agents

Temporal is the strongest general-purpose choice when: the agent is *enterprise-scale* (many concurrent runs, strict SLOs); workflows are complex (signals + sagas + child workflows + versioning); the team can run workers (it is not serverless — you operate worker processes); and the organization values a proven, boring, well-documented substrate. Its costs: operational weight (a cluster to run or a Cloud subscription), a determinism discipline that takes a week to internalize, and a Python/TS SDK that is solid but occasionally verbose.

```python
# Temporal Python SDK — workflow + activity skeleton
from temporalio import workflow, activity

@activity.defn
async def llm_call(prompt: str) -> str:      # non-deterministic work lives here
    return await call_model(prompt)

@workflow.defn
class AgentLoop:
    @workflow.run
    async def run(self, task: str) -> str:
        return await workflow.execute_activity(llm_call, task,
            retry_policy=workflow.RetryPolicy(maximum_attempts=5, initial_interval=2.0))
```

---
## 7. Engine Landscape: Inngest and Restate

### 7.1 Inngest — Developer-First Durable Workflows for Serverless

**Inngest** is a durable execution platform built for the serverless world: TypeScript, Python, and Go SDKs, functions composed of individually retriable **steps**, event-driven triggers (HTTP/webhooks, cron, queues), built-in retries, and managed observability. There is no cluster to operate — functions deploy to your existing serverless/container runtime (Vercel, Netlify, AWS Lambda, Cloudflare Workers, Fly.io, Docker) and Inngest runs the durable execution plane.

Inngest is *deliberately* marketed at AI/agent workflows, with AI-native primitives:

```typescript
// Inngest TypeScript — steps are retriable; outputs are recorded
import { inngest } from "./client";

export const researchAgent = inngest.createFunction(
  { id: "research-agent" }, { event: "agent/research.requested" },
  async ({ step, event }) => {
    const plan = await step.ai.infer("plan", planPrompt(event.data.topic));
    const findings = await step.run("search", () => searchWeb(plan));
    const report = await step.ai.infer("synthesize", reportPrompt(findings));
    await step.run("notify", () => notify(event.data.userId, report));
  });
```

Each `step.run` / `step.ai.infer` boundary is durable: on failure, the step retries with backoff, and completed steps replay from recorded output — never re-executing paid LLM calls. `step.ai.infer` is an LLM-specific step type that proxies long-running inference (reducing serverless function timeout pressure and cost) and records the completion for replay. Inngest also ships an agent framework (AgentKit) with memory/planning/tool-use primitives layered on top of the durable workflow core.

**Best for:** serverless-first teams (especially TypeScript on Vercel), agent workflows that are event/cron-triggered, and teams that want durability without operating infrastructure. **Trade-offs:** younger platform than Temporal, less control over low-level semantics, and the serverless model imposes function-timeout thinking even though steps are durable.

### 7.2 Restate — Durable Execution with Strong Guarantees

**Restate** (Restate Labs) is a newer durable-execution engine with an unusual architecture: it combines durable execution (journaled steps, replay) with **durable RPC**, **virtual objects** (keyed stateful actors with single-writer semantics), and built-in event sourcing — plus SDKs for TypeScript, Python, Java/Kotlin, Go, and Rust. Services are deployed as containers, serverless functions, or Kubernetes pods; the engine runs as a single binary, on K8s, or as Restate Cloud.

```typescript
// Restate TypeScript — a virtual object with durable steps
export const agent = restate.object({
  name: "agent",
  handlers: {
    async run(ctx: ObjectContext, task: string) {
      const plan = await ctx.run("plan", () => planTask(task));
      const result = await ctx.run("execute", () => executePlan(plan));
      await ctx.set("lastResult", result);   // keyed state, single-writer
      return result;
    },
  },
});
```

Restate's differentiators: **keyed concurrency** (a virtual object for customer X serializes access per key — natural for per-customer agent state), durable RPC as a first-class primitive (call another service and have the call journaled), and event-sourcing baked in (state as a log). It is developer-friendly — "write normal async code, get durability for free" — and its single-writer semantics map cleanly onto agent-per-entity patterns (one agent per loan application, per account, per client).

**Best for:** teams that want strong consistency guarantees and per-entity agent state; TypeScript/Java/Kotlin shops; greenfield platforms. **Trade-offs:** younger ecosystem and community than Temporal; fewer production war stories; the virtual-object model is a mental shift for teams used to stateless functions.

---
## 8. Engine Landscape: AWS Step Functions and Azure Durable Functions

### 8.1 AWS Step Functions — The Managed Enterprise Default

**AWS Step Functions** is Amazon's managed workflow service: you author a **state machine** in JSON (Amazon States Language — ASL), and AWS executes it with integrated retries, timeouts, and 200+ AWS service integrations. It is the path of least resistance for anything already on AWS, and its visual workflow designer is genuinely useful for review with non-engineers (compliance, audit).

**Standard vs Express workflows:**

| | Standard | Express |
|---|---|---|
| Durability | Durable — execution recorded, survives restarts | Non-durable — in-memory, best-effort |
| Max duration | 1 year | 5 minutes |
| Delivery semantics | Exactly-once | At-least-once |
| Throughput | Moderate | Very high (100k+ starts/sec) |
| Cost | Per state transition | Per execution (cheaper) |
| Use for agents | Any real agent run | High-volume, idempotent, short steps |

```json
{
  "Comment": "Loan triage agent",
  "StartAt": "CreditCheck",
  "States": {
    "CreditCheck": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke",
      "Retry": [{ "ErrorEquals": ["States.ALL"], "IntervalSeconds": 3, "MaxAttempts": 3 }],
      "Next": "RiskAssessment"
    },
    "RiskAssessment": { "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke", "End": true }
  }
}
```

**Limits:** authoring is JSON state-machine syntax (not code) — branching, loops, and error handling are expressive but verbose and hard to unit-test; there is no first-class signal/query model (callbacks and Task Tokens approximate it); LLM-step ergonomics are absent (you build them). Step Functions is best for *simple-to-moderate* durable chains inside AWS; for agent loops with HITL and sagas, teams usually outgrow ASL and move to Temporal or Inngest.

### 8.2 Azure Durable Functions — The Azure-Native Choice

**Azure Durable Functions** brings durable execution to Azure Functions: **orchestrator functions** (C#/JS/Python/PowerShell) with checkpointed execution, fan-out/fan-in patterns, durable timers, HTTP APIs, and Event Grid integration. Every orchestrator step is checkpointed to Azure Storage, so orchestrator functions are replayed deterministically on restart — the same durable-execution model as Temporal, native to Azure.

```csharp
// Azure Durable Functions (C#) — orchestrator with fan-out/fan-in
[FunctionName("ResearchOrchestrator")]
public static async Task<List<string>> RunOrchestrator(
    [OrchestrationTrigger] IDurableOrchestrationContext ctx)
{
    var topics = await ctx.CallActivityAsync<List<string>>("PlanResearch", null);
    var tasks = topics.Select(t => ctx.CallActivityAsync<string>("ResearchTopic", t));
    var results = await Task.WhenAll(tasks);          // fan-out / fan-in
    return await ctx.CallActivityAsync<List<string>>("Synthesize", results);
}
```

**Best for:** organizations already committed to Azure Functions/App Service; teams wanting checkpoints, retries, and HITL via durable external events (`RaiseEventAsync` ≈ signals) with minimal new infrastructure. **Trade-offs:** Azure-bound, orchestrator determinism rules similar to Temporal's, and the agent ecosystem (LLM-step primitives, agent frameworks) is thinner than Temporal's or Inngest's.

---
## 9. Engine Landscape: Conductor, Camunda, and Data Pipeline Orchestrators

### 9.1 Conductor (Netflix) — Battle-Tested JSON Orchestration

**Conductor** is Netflix's open-source (Apache-2.0) workflow orchestration engine, running Netflix's media pipelines in production for years. Workflows are defined as **JSON**; workers are code (Java, Python, Go, and others) that poll for tasks, with dynamic fork/join, retries, timeouts, and a visualization UI.

**For agents:** Conductor is viable inside Netflix-style engineering cultures that already standardize on it, but its JSON-authoring model, worker-polling model, and minimal AI-specific tooling make it a *workflow engine that can host agent steps*, not an agent-native platform. Migrating existing Conductor workloads to include LLM activities is straightforward — an LLM call is just another worker task.

### 9.2 Camunda — The Enterprise BPM Choice

**Camunda** is the enterprise business-process-management platform: **BPMN 2.0** process models (visual, executable), the cloud-native **Zeebe** engine, **human task management** (first-class user tasks — the strongest HITL story of any engine here), **DMN** decision tables, and extensive compliance/governance tooling. Processes are modeled visually and executed by Zeebe with built-in retries, timers, and message events.

For banking specifically, Camunda is often the right call when the *process* is the asset: loan origination, onboarding, exception handling — processes with defined human roles, SLAs, and audit requirements, which an LLM agent augments (drafting documents, pre-screening, summarization) rather than owns. The BPMN model is inspectable by risk and operations in a way that code-first workflows are not. **Trade-offs:** BPMN modeling is heavier than writing code, and LLM-heavy dynamic loops fit awkwardly in a visual process model — Camunda excels at *structured processes with human gates*, Temporal/Inngest excel at *dynamic agent loops*.

### 9.3 Prefect / Dagster / Airflow — Data-Pipeline Orchestrators

These three are **pipeline-first, not workflow-first**: built for batch data movement with DAG-shaped execution, schedule-driven runs, and per-task retries.

| | Prefect 3.x | Dagster | Airflow |
|---|---|---|---|
| Model | Flow/task, code-first | Asset-based (data as the unit) | DAGs (Python) |
| Durability | Durable task execution, retries, caching, timeouts | Asset materialization + retries | Task retries, backfills |
| Agent fit | Can host agent steps in flows; caching helps LLM costs | Asset lineage is great for data products produced by agents | Batch agent jobs (nightly reports); weak for interactive loops |
| Signals / HITL | No first-class signals | No | No |
| Best for | Data pipelines with embedded agent steps | Data-platform teams where agents produce/consume datasets | Traditional batch scheduling |

The guidance: if the dominant workload is a *data pipeline* (ETL, feature engineering, batch scoring) with agent steps inside, Prefect/Dagster/Airflow are appropriate and durable enough. If the *agent loop itself* is the product — interactive, signal-driven, HITL, sagas — use a workflow engine. The two categories coexist: Airflow schedules the nightly batch, Temporal runs the interactive agent.

---
## 10. Engine Comparison Table

| Engine | Languages / Authoring | Durability model | Retries | Timers | Signals | HITL | Scalability | Self-host vs Managed | Best for |
|---|---|---|---|---|---|---|---|---|---|
| **Temporal** | Go/Java/Python/TS; code | Event-sourced replay | Built-in, per-activity policies | Durable | First-class | Via signals | Very high (cluster) | Both (Cloud or self-host) | Complex agent orchestration, enterprise scale |
| **Cadence** | Go/Java; code | Event-sourced replay | Built-in | Durable | First-class | Via signals | Very high | Self-host | Existing Cadence estates; Uber-style scale |
| **Inngest** | TS/Python/Go; steps | Journaled steps | Built-in, per-step | Durable (sleep) | Event-triggered | Via steps + wait | High (serverless) | Managed (BYO functions) | Serverless-first AI agents, TS teams |
| **Restate** | TS/Python/Java/Kotlin/Go/Rust; code | Journaled + virtual objects | Built-in | Durable | Via RPC/events | Via RPC | High (keyed sharding) | Both (binary/K8s/Cloud) | Strong-consistency, per-entity agent state |
| **AWS Step Functions** | JSON (ASL) | Standard: durable, exactly-once; Express: at-least-once | Built-in | Wait states | Via Task Tokens / callbacks | Via callbacks | Very high (Express) | Managed | Simple durable chains on AWS |
| **Azure Durable Functions** | C#/JS/Python/PowerShell; code | Orchestrator checkpoints | Built-in | Durable | Via external events | Via external events | High | Managed | Azure-native agent orchestration |
| **Conductor** | JSON defs + code workers | Durable task queues | Built-in | Yes | Via events | Via tasks | High | Both | Netflix-style estates; JSON governance |
| **Camunda/Zeebe** | BPMN 2.0 + code | Process instance state | Built-in | Durable | Message events | **First-class user tasks** | High (Zeebe) | Both (Cloud/self-host) | Structured processes with human gates |
| **Prefect** | Python; flows/tasks | Durable tasks + caching | Built-in | Sleep | No | No | Medium-High | Both | Data pipelines with agent steps |
| **Dagster** | Python; assets | Asset materialization | Built-in | Schedules | No | No | Medium-High | Both | Asset-lineage data platforms |
| **Airflow** | Python DAGs | DAG metadata + retries | Built-in | Schedules | No | No | Medium | Self-host/managed | Batch scheduling of agent jobs |

## 11. The Canonical Agent Workflow as a Durable Workflow

### 11.1 The Agent Loop, Re-Expressed

The canonical agent is a loop: **LLM call → tool call → LLM call → tool call …** until the agent decides it is done. Expressed as a durable workflow, that loop becomes:

```
Agent workflow (durable)
  └─ loop:
       ├─ LLM step   → activity (recorded output, retried with backoff)
       ├─ tool step  → activity (idempotent side effect, recorded output)
       └─ done?      → workflow decision (deterministic branch on recorded state)
```

Every LLM call and every tool call is an **activity**. The workflow code is only the deterministic glue — the loop, the branching, the state. Crash mid-loop? Replay reconstructs the loop's position from history and the next activity executes. The agent *never* re-does a completed LLM or tool call.

### 11.2 Why the Boundary Matters

The workflow/activity split is the durability contract: **workflow code** holds only deterministic orchestration (no network, no clock, no randomness); **activities** hold everything real — LLM inference, HTTP calls, DB writes, file operations, human notifications. Putting an LLM call directly in workflow code breaks replay (Section 33) and forfeits retries, heartbeats, and recording. The discipline: *if it touches the outside world, it is an activity.*

### 11.3 The Step Outputs Are the State

In the durable model, the agent's conversational context and intermediate artifacts should be reconstructible from step outputs. Each activity's result is recorded in the event history; the workflow can carry accumulated context as variables between steps (replayed deterministically). For large context (embeddings, documents), store payloads in an external store and pass references — history stays small and replay stays fast (see Section 22 on memory separation).

---
## 12. Pattern 1 — The Durable Agent Loop

### 12.1 The Simplest Durable Agent

The minimal durable agent: loop over LLM and tool activities until the agent signals completion.

```python
# Temporal Python — durable agent loop
@workflow.defn
class DurableAgent:
    @workflow.run
    async def run(self, task: str) -> str:
        messages = [{"role": "user", "content": task}]
        for turn in range(MAX_TURNS):
            # LLM step: activity, 3-5 retries, exponential backoff + jitter
            response = await workflow.execute_activity(llm_complete, messages,
                retry_policy=workflow.RetryPolicy(maximum_attempts=5,
                    initial_interval=1.0, backoff_coefficient=2.0, maximum_interval=30.0))
            messages.append({"role": "assistant", "content": response})
            tool_calls = parse_tool_calls(response)
            if not tool_calls:
                return response                     # agent is done
            for call in tool_calls:                 # tool step: idempotent
                result = await workflow.execute_activity(execute_tool,
                    {"name": call.name, "args": call.args})
                messages.append({"role": "tool", "name": call.name, "content": result})
        raise workflow.ApplicationError("Max turns exceeded")
```

### 12.2 Retry Policy for LLM Activities

LLM APIs fail in predictable ways — 429 rate limits, 5xx, timeouts — and the retry policy is the durability workhorse. Follow the guidance in the [LLM Latency Optimization guide](ai_llm/llm_latency_optimization_guide.md) for backoff math:

- **3–5 attempts** per LLM activity (more for long/expensive calls where re-execution is cheap relative to workflow failure), with **exponential backoff + jitter** (initial 1–2s, coefficient 2.0, ceiling 30–60s) to avoid thundering-herd retry storms after a provider incident.
- **Respect `Retry-After`** where the provider sends it (429s): cap the engine's interval at the server's suggested delay.
- **Classify errors**: rate-limit (retryable), 5xx (retryable), 4xx auth/validation (not retryable — fail fast and alert).
- **Timeouts**: set an activity start-to-close timeout generous enough for the model (e.g. 5 min for long generations), with heartbeating for streaming calls.

### 12.3 What a Crash Looks Like Now

Worker dies at turn 7 of 12. The engine replays turns 1–6 from history — the recorded LLM and tool outputs are returned instantly, no re-inference, no re-executed tool calls — and executes turn 7's LLM activity fresh. To the user, the agent "paused for a moment." Cost impact: zero re-spend on completed turns.

---
## 13. Pattern 2 — Human-in-the-Loop (HITL)

### 13.1 The Durable Wait for a Human

The signature HITL pattern: the workflow reaches a consequential decision, **pauses**, and waits for a human decision delivered as a **signal**. The pause is durable — it survives restarts, redeploys, and days of wall-clock time, because the engine persists the timer/signal wait in history.

**Loan-approval example:** the agent prepares a recommendation (credit check, LLM risk assessment, fraud screen) and then *stops* — no money moves until a credit officer approves via signal.

```
receive application (signal) → credit check (activity)
  → LLM risk assessment (activity) → fraud screen (activity)
  → WAIT for officer_decision signal ← durable pause (HITL gate)
       ├─ approved → proceed to disbursement
       ├─ rejected → compensation + notify
       └─ 48h timeout → escalate to senior officer (activity)
```

### 13.2 Approval Timeouts and Escalation

Human steps need a **timeout with an escalation path**, not an indefinite wait. The pattern: a durable timer (48h) races the signal; on timeout, an escalation activity notifies the next authority (email, ticketing, paging) and the workflow waits again — optionally with a different timeout.

```python
# Temporal Python — HITL with timeout and escalation
@workflow.defn
class LoanApproval:
    @workflow.run
    async def run(self, application_id: str) -> str:
        recommendation = await workflow.execute_activity(assess, application_id)
        for level in ("officer", "senior_officer", "committee"):
            try:
                decision = await workflow.wait_for_signal(f"decision_{level}",
                    timeout=timedelta(hours=48 if level != "committee" else 24))
                return apply(decision)          # approve / reject path
            except asyncio.TimeoutError:
                await workflow.execute_activity(escalate,
                    {"application": application_id, "level": level})
        await workflow.execute_activity(notify, application_id, "rejected: no decision")
        return "REJECTED_TIMEOUT"
```

### 13.3 HITL as First-Class Workflow State

The approval step is **state in the workflow**, not a side table: the history records that the workflow is waiting, who the decision came from, when, and the recommendation it was based on. This is exactly what banking compliance requires (Section 29). Queries let dashboards show "42 loan agents awaiting officer decision, 7 escalated."

### 13.4 Banking Compliance Context

The [LLM Development Risks & Security guide](llm_development_risks_security_guide.md) covers **excessive agency** — autonomous agents taking consequential actions without oversight. Durable HITL is the mechanical remedy: consequential actions (money movement, external communications, data deletion) are gated behind human signals, auditable in the history. In banking the pattern is *mandatory* above a threshold: agent recommends, human disposes.

---
## 14. Pattern 3 — Saga / Compensation

### 14.1 The Problem: Multi-Step Side Effects

A multi-step business transaction has side effects that cannot be atomically rolled back: book a transfer, notify the client, update the CRM. If the CRM update fails, steps 1–2 have already happened. A **saga** defines compensating actions per step and runs them in reverse on failure — reverse the transfer, send a failure notice.

```
Normal path:       book transfer → notify → update CRM → DONE
CRM fails:         book transfer → notify → [update CRM FAILS]
Compensation:      reverse transfer ← send failure notice ← (failure detected)
```

### 14.2 Implementing Sagas in a Workflow Engine

Workflow engines make sagas explicit and auditable. In Temporal, the standard approach is a `try/except` around the transaction with compensation activities registered as you go:

```python
@workflow.defn
class TransferSaga:
    @workflow.run
    async def run(self, txn: TransferRequest) -> str:
        compensations: list[Callable] = []
        try:
            await workflow.execute_activity(book_transfer, txn)      # step 1
            compensations.append(lambda: reverse_transfer(txn.id))
            await workflow.execute_activity(notify_client, txn)      # step 2
            compensations.append(lambda: send_failure_notice(txn.id))
            await workflow.execute_activity(update_crm, txn)         # step 3
            return "COMPLETED"
        except Exception:
            for compensate in reversed(compensations):               # undo in reverse
                await workflow.execute_activity(compensate)
            raise
```

### 14.3 The Saga Pattern for Agents

The [Payments Hub guide](../banking/payments_hub_guide.md) discusses sagas for payment flows — the same pattern applies to agent workflows, with an extra wrinkle: **compensating an LLM activity means compensating its side effects, not the inference.** An LLM call that only produced text has no side effect to undo; an LLM-driven action (an email sent, a hold placed, an order submitted) does. The durable version: every agent activity with side effects registers a compensation, and the workflow's failure path is as tested as its success path (Section 32).

Banking rule of thumb: **any agent step that moves money, changes a limit, or sends external communication must have a registered compensation.**

---
## 15. Pattern 4 — Fan-Out / Fan-In

### 15.1 Parallel Sub-Tasks as Child Workflows

The research agent splits one task into N parallel sub-tasks, each runs independently, and the results aggregate. In a durable engine, each sub-task is a **child workflow** — its own history, its own retries, its own failure isolation — and the parent awaits the set (fan-in).

```
Research agent (parent workflow)
  ├─ plan (activity): split topic into N research questions
  ├─ fan-out: N child workflows (research_subtask_i) — each search→read→synthesize
  ├─ fan-in: await all children (parallel)
  └─ synthesize (activity): aggregate N findings into the report
```

### 15.2 Child Workflow Semantics

- **Isolation:** a failed child retries independently without killing siblings; the parent decides policy (continue with partial results, fail all, retry). **Cancellation:** the parent can cancel children mid-flight (user cancelled the run). **Recovery:** children are durable too — a crash mid-child resumes that child from its own history. **Bounds:** cap parallelism and per-child budgets (turns, tokens, time).

The [Research Agents guide](research_agents_guide.md) documents the parallel-researchers pattern in depth; the durable-engine version adds exactly one thing: **the fan-out/fan-in structure survives crashes**, so a 30-minute, 8-parallel-researcher run cannot be lost to a redeploy.

```python
# Temporal Python — fan-out/fan-in with child workflows
results = []
for i, question in enumerate(questions):
    results.append(workflow.execute_child_workflow(
        ResearchSubtask, question, id=f"research-{run_id}-{i}"))
answers = await asyncio.gather(*results)          # fan-in
report = await workflow.execute_activity(synthesize, answers)
```

---
## 16. Pattern 5 — Event-Driven Agents

### 16.1 Workflows That Wake on Events

Not every agent run starts with a user prompt. Durable engines are event-driven by construction: **Kafka messages, webhooks, queue payloads, and schedules trigger workflow runs**, and **signals inject mid-run context** into already-running workflows.

```
Triggers → workflow start:
  Kafka "trade.executed"     → trade surveillance agent
  Webhook "application.submitted" → loan origination agent
  S3 object created          → document processing agent
  Cron "0 2 * * *"           → nightly report agent
Signals → mid-run context:
  "new_data.arrived" → research agent extends its search
  "user.cancelled"   → agent aborts gracefully (compensation)
  "market.alert"     → monitoring agent re-prioritizes
```

### 16.2 Trigger vs Signal — The Distinction

| | Trigger | Signal |
|---|---|---|
| When | Starts a new workflow | Injects into a *running* workflow |
| Semantics | Like a queue consumer | Like an interrupt / message to a process |
| Examples | Kafka event, webhook, cron | Human approval, user cancel, late data |

The [Event Stream Processing guide](event_stream_processing_guide.md) covers stream ingestion; the durable-engine layer adds: **a trigger event starts a resumable run, and a signal event is durably recorded** — if the workflow is mid-replay when a signal arrives, it is re-delivered correctly, never lost, never doubled.

### 16.3 The Idempotent Consumer

Event-driven agents must dedupe at the boundary: the same Kafka message may be delivered twice (at-least-once brokers). Durable engines give you **workflow ID uniqueness** — a workflow started with `id=<dedup-key>` (e.g., `trade-surveillance-<trade-id>`) will not double-start on a re-delivered event, and an already-completed workflow with that ID rejects the duplicate. This is the durable equivalent of a consumer-group offset, but per-logical-unit rather than per-partition.

---
## 17. Pattern 6 — Scheduled / Cron Agents

### 17.1 The Schedule Itself Is Durable

Nightly report agents, monitoring agents, compliance sweeps — these run on schedules. In a durable engine, **the schedule is a first-class durable object** (not a host cron that dies with the box):

- **Temporal:** Schedule API — cron expressions, calendar intervals, pause/resume, backfill, overlap policy. **Inngest:** `{ cron: "0 2 * * *" }` on any function, missed runs caught up per policy. **Step Functions:** EventBridge Scheduler targets the state machine. **Durable Functions:** `TimerTrigger` / `CreateTimer`.

### 17.2 Patterns Around the Schedule

- **Missed-run policy:** if the scheduler was down at 02:00, does the agent run late, skip, or backfill? Durable schedulers answer explicitly (Temporal: `Backfill`; catch-up windows).
- **Overlap control:** prevent two nightly runs overlapping when the previous one ran long (Temporal: `OverlapPolicy.SKIP` / `BUFFER_ONE`).
- **Schedule → workflow → child fan-out:** the nightly compliance agent spawns per-entity child workflows (per-counterparty, per-book), each durable.
- **Jittered schedules for provider load:** stagger agent cron runs so N agents do not hit the LLM provider simultaneously (rate-limit hygiene, per the [LLM Latency Optimization guide](ai_llm/llm_latency_optimization_guide.md)).

Example — nightly monitoring agent in Inngest:

```typescript
export const nightlyComplianceSweep = inngest.createFunction(
  { id: "compliance-sweep", cron: "0 2 * * *" }, { event: "none" },  // cron-only
  async ({ step }) => {
    const entities = await step.run("list-entities", () => listEntities());
    const results = await step.run("sweep-all", () =>
      Promise.all(entities.map(e => sweepEntity(e))));
    await step.run("report", () => publishReport(results));
  });
```

---
## 18. Pattern 7 — Checkpointed Long-Running Agents

### 18.1 Agents That Run for Days

Some agents run for days: document processing pipelines with human reviews at each stage, KYC remediation tracks, multi-day research programs. The durable-engine answer: **the workflow state IS the checkpoint**. Every completed step is recorded; a run paused overnight, through a deploy, or across a holiday resumes exactly where it stopped.

```
Day 1: ingest docs → extract → LLM classify → [HUMAN review stage 1]
Day 2: [HUMAN review stage 2] → LLM draft summary → [HUMAN sign-off]
Day 3: sign-off received → generate report → notify → DONE
```

### 18.2 Checkpoints = Workflow State, Not Just Memory

The distinction matters: **checkpoints are the execution state** (which steps completed, with what outputs, what timers are pending, what signals are awaited). This is *not* the same as the agent's conversational memory (facts, context, user preferences), which lives in a memory store (Section 22). The workflow checkpoint answers "where is this run?", the memory store answers "what does this agent know?" Both are needed; they are different systems.

### 18.3 Long-Running Practicalities

- **History growth:** days-long workflows accumulate history; use Temporal's compaction/archival or keep per-stage payloads external (Section 30); compact old stages into a summary activity result.
- **Human stages as durable waits:** each review stage is a signal wait with a timeout — the whole pipeline is a chain of durable HITL gates, each auditable.
- **Business-time vs wall-time:** "wait 2 business days" needs a calendar-aware activity (workflow timers are wall-clock); compute the next business-day timestamp in an activity, then set a durable timer to it.

---
## 19. Pattern 8 — Versioned Agent Workflows

### 19.1 The Deploy Problem

Agent logic changes constantly — new prompts, new tools, new loop behavior. But a workflow started under code v1 must not be replayed against v2 mid-flight: replay could branch differently and corrupt the run. Without versioning, **a deploy breaks every in-flight workflow** (the "poisoned history" failure mode, Section 31).

### 19.2 Temporal's Versioning Model

Temporal solves this with **version markers** — the workflow records which code version executed each decision point, and replay honors the recorded version:

```python
# Temporal Python — patch/version marker for a changed step
if workflow.patch("risk-v2-prompt"):
    risk = await workflow.execute_activity(llm_risk_assessment_v2, credit)
else:
    risk = await workflow.execute_activity(llm_risk_assessment_v1, credit)
```

- **`getVersion` / `patch` markers:** in-flight workflows replay the branch recorded in history (v1); new workflows take the new branch (v2). Both versions must remain deployable until all v1 runs complete.
- **Worker versioning (build-id based):** workers advertise a build ID; the engine pins each workflow to the build that started it, so a v1 workflow is only ever executed by v1 workers — the deployment can drain old workers once in-flight runs finish.
- **Practical rule:** never change a workflow's step sequence or activity signatures without a version marker; adding new activities is safe (they simply don't exist in old history); *changing* existing steps requires the marker.

### 19.3 Migration Discipline

- Keep old branches alive until the last in-flight workflow completes (Temporal UI shows remaining runs per version).
- Version *prompts and agent logic* explicitly: a prompt change inside an activity is safe (activities are not replayed) — only *workflow-code* changes need markers. This is a powerful property: **iterate on prompts freely, iterate on orchestration carefully.**
- Test version transitions with **replay tests** (Section 32): replay production histories from v1 against v2 code to prove compatibility before deploying v2.

## 20. Agent-Specific Concerns: LLM Calls as Activities

### 20.1 The LLM Activity Contract

LLM calls are the most failure-prone and most expensive steps in an agent workflow, so they get the most careful durability treatment:

- **Retries:** LLM APIs fail with 429 rate limits, 5xx, and timeouts. Retry with exponential backoff + jitter, respect `Retry-After`, cap attempts (3–5), and classify non-retryable errors (4xx auth, invalid request) to fail fast. See [LLM Latency Optimization](ai_llm/llm_latency_optimization_guide.md) for the full backoff playbook.
- **Non-idempotency is inherent:** an LLM call cannot be made idempotent — the same prompt may yield different outputs on retry. Accept **at-least-once** semantics for the *activity* (it may execute more than once) and mitigate:
  - **Cache deterministic parts:** prompt prefix, retrieved context, tool results — per the [Agent Runtime Cache Design guide](ai_llm/agent_runtime_cache_design_guide.md). Caching makes retries cheaper and outputs more stable.
  - **Record outputs in history:** replay uses the *recorded* output of the first successful execution; replay never re-executes a completed LLM call.
- **Determinism of the loop, not the model:** the workflow's *control flow* is deterministic (it branches on recorded outputs); the model's outputs may vary — that variance is confined to activity results, which is exactly where non-determinism is allowed.

### 20.2 Streaming and Heartbeating

Long generations (multi-minute streaming responses) exceed typical activity timeouts. Patterns:

- **Heartbeat the stream:** the activity sends heartbeats while tokens flow; if the worker dies, the engine knows the activity was alive and retries from scratch with recorded progress.
- **Checkpoint partial generations:** write streamed output to object storage as it arrives; on retry, resume from the last persisted chunk and tell the model to continue (or restart with the partial text in context).
- **Use engine-native LLM steps:** Inngest's `step.ai.infer` proxies long-running inference outside the function timeout — a serverless workaround worth copying even on other engines.

### 20.3 Cost Controls Per Workflow

- Per-workflow **token/turn budgets**: a max-turns counter in the loop (Section 12) plus a token-usage activity that accumulates and enforces caps.
- **Model tiering as workflow logic:** route cheap/fast models for triage, expensive models for final synthesis — the routing decision is deterministic workflow code, the inference is an activity.
- **Provider failover as retry policy:** on repeated 5xx from provider A, retry against provider B (a routing activity) — durable retries make cross-provider failover a configuration, not an emergency.

---
## 21. Agent-Specific Concerns: Tool Calls, External APIs, Human Steps

### 21.1 Tool Side Effects Must Be Idempotent

The single most important durability rule for agents: **every tool with side effects must be idempotent.** Because activities are at-least-once (may retry), a non-idempotent tool duplicates side effects: two payments, two emails, two tickets. The mechanics:

- **Idempotency keys:** every side-effecting call carries a key derived from the workflow/activity context (e.g., `payment-<workflow_id>-<activity_id>`). The tool stores the key and returns the original result on a repeated call with the same key. See the [Payments Hub guide](../banking/payments_hub_guide.md) for the full idempotency-key design.
- **Banking absolute:** money movement MUST be idempotent — a retried transfer activity must not double-debit. The payment ID dedup is non-negotiable (same pattern as payment rails' own idempotency).
- **Examples:**
  - Create payment → dedup on `payment_request_id`; retry returns the original payment reference.
  - Send email → dedup on `message_id`; retry returns the original message ID.
  - Create ticket, place hold, update CRM → same key discipline.
- **Read-only tools** (search, lookup, retrieve) need no idempotency — they are naturally safe to retry, though caching still helps cost.

### 21.2 External API Calls

Same discipline as tools, plus:

- **Retry with the same key:** if the first attempt may have succeeded server-side (timeout before response), retrying with the same idempotency key lets the remote dedup. Never regenerate the key on retry.
- **Classify failures:** 5xx → retryable; 4xx → likely permanent (validation, auth) — fail fast, alert; timeouts → retry with same key (ambiguous outcome).
- **Rate limits:** per-provider quotas in the retry policy (respect `Retry-After`), plus a workflow-level limiter activity if multiple agents share a quota.

### 21.3 Human Steps: Durable Wait, Timeout, Escalation

Covered in depth in Section 13; the durability-specific rules:

- Human decisions arrive as **signals**, recorded in history with payload, timestamp, and sender identity; for regulated processes the payload carries the operator ID, so the history becomes the approval evidence (Section 29).
- Every human wait has a **timeout** and an **escalation path** — never an unbounded wait.
- After a crash, replay re-delivers the recorded signal — a human who approved pre-crash is not asked twice.

---
## 22. State Management: Workflow State vs Agent Memory

### 22.1 Two Different Stores, Two Different Jobs

| | Workflow state | Agent memory |
|---|---|---|
| **What it is** | Execution state: steps completed, step outputs, pending timers, awaited signals | Business context: conversation history, facts, user preferences, domain knowledge |
| **Where it lives** | In the engine (event history / workflow variables) | In a memory store: Redis, vector DB, key-value store, RAG index |
| **Who writes it** | The engine, automatically, at every step boundary | The agent's activities, explicitly |
| **Reconstructible?** | Yes — deterministic replay of history | No — must be persisted by design |
| **Example** | "Loan-approval workflow is at step 4, awaiting officer signal, 23h left on timer" | "Applicant prefers email, has existing mortgage, risk tolerance low" |

### 22.2 The Conflation Trap

Do **not** conflate the two. Concretely: don't stuff conversation transcripts into workflow variables — history bloat slows replay (Section 30); store transcripts in Redis/vector DB and pass a reference. Don't make the memory store the source of execution truth — a crash cannot be recovered from memory alone; recovery needs the engine's history. The workflow carries *references and summaries*; the memory store carries *bulk content and embeddings* (see [Beyond RAG](ai_llm/beyond_rag_guide.md) for memory-system design and [Agent Runtime Cache Design](ai_llm/agent_runtime_cache_design_guide.md) for what to cache where).

### 22.3 Passing State Between Steps

Activities receive inputs and return outputs; the workflow holds results in variables (deterministic — replayed from history). For large payloads (documents, embeddings), pass **references** (object-store keys, DB IDs) and let the activity fetch the payload. This keeps history small, replay fast, and state unambiguous.

---
## 23. Checkpointing vs Durable Execution; Delivery Semantics

### 23.1 Checkpointing vs Event-Sourced Replay

| | Checkpointing (snapshot/restore) | Durable execution (event-sourced replay) |
|---|---|---|
| Mechanism | Periodic snapshot of full process state; crash → restore from last snapshot | Append-only event history; crash → replay full history, then continue |
| State loss | Up to the last snapshot — work between snapshots is lost | None — every recorded step is preserved |
| Determinism | Not required (restores exact memory) | Required (replay re-executes workflow code) |
| Design burden | You design snapshot points, serialization, restore | The engine does it — steps are the checkpoint boundaries |
| Failure window | Depends on snapshot frequency | Zero (steps are atomic boundaries) |

**For agents, durable execution is strictly better:** no manual checkpoint design, no snapshot-frequency trade-off, and exactly-once semantics for workflow steps via deterministic replay. "Checkpointed agent" in the durable world just means *long-running workflow with many recorded steps* (Section 18) — the checkpoints come free with every step boundary.

### 23.2 Exactly-Once vs At-Least-Once — The Honest Semantics

| Layer | Semantics | Why |
|---|---|---|
| Workflow steps (orchestration decisions) | **Exactly-once** (via deterministic replay) | Replay re-executes code, but every decision is resolved from recorded history — same inputs, same branch, once |
| Activities (side effects) | **At-least-once** (executed reliably, may retry) | An activity may start, crash, and be retried; the engine dedupes only *recorded completions* |
| Your end-to-end business operation | **Effectively-once** — if activities are idempotent | At-least-once activities + idempotent tools = each side effect lands exactly once |

The rule: **workflow steps are exactly-once; activities are at-least-once; idempotency converts activities to effectively-once.** Never assume an activity ran at most once — design every side-effecting activity as if it will run twice, because under a crash it may.

### 23.3 What the Engine Guarantees — and Doesn't

The engine guarantees: recorded steps survive crashes; timers and signals are durable; replay is deterministic; a completed activity is not re-executed. The engine does **not** guarantee: that your tool's side effect is idempotent (your job), that the LLM returns the same output on retry (impossible — see Section 20), or that external systems observe the workflow's success atomically (saga's job, Section 14).

---
## 24. Tooling and Stack: Temporal in Depth

### 24.1 SDK Surface

Temporal SDKs (Python, TypeScript, Go, Java) share the same shape — workflow and activity decorators, `Client` for starting/signaling/querying, `Worker` for hosting code:

```python
# Temporal Python SDK — worker sketch
from temporalio.client import Client
from temporalio.worker import Worker

async def main():
    client = await Client.connect("localhost:7233")   # dev server
    await Worker(client, task_queue="agent-queue", workflows=[DurableAgent],
                 activities=[llm_complete, execute_tool]).run()
```

### 24.2 Deployment Choices

- **Dev server:** `temporal server start-dev` runs a full single-node engine locally — the fastest way to iterate; the UI (`http://localhost:8233`) shows workflow histories live.
- **Self-hosted:** Docker Compose (single node) or Helm chart on Kubernetes (HA cluster: frontend, history, matching services + Cassandra/PostgreSQL/MySQL). You operate it; you own the SLAs.
- **Temporal Cloud:** managed SaaS — no cluster ops, per-workflow pricing, UI and observability included. The pragmatic enterprise default unless there is a data-residency reason to self-host (common in banking: on-prem/private-cloud mandates, as discussed in the [On-Prem LLM Deployment guide](on_prem_llm_deployment_guide.md)).

### 24.3 The Workflow History Viewer — The Agent Debugger

Temporal's UI is the killer tool for agent debugging: every workflow shows its full event history — each activity, input, output, retry, timer, signal, and failure — as a timeline. When an agent misbehaves, the history answers: *what did the LLM return at step 7? which tool ran twice? why did the approval time out? what was the exact input that produced the bad output?* This visibility is the operational argument for durable execution: **agents become debuggable artifacts, not black boxes.**

---
## 25. Tooling and Stack: Inngest, Restate, Step Functions

### 25.1 Inngest

- **Functions as steps:** `inngest.createFunction` + `step.run` / `step.ai.infer` / `step.waitForEvent`; each step is durably recorded and retried. **Event-driven triggers:** HTTP/webhook, cron, queues, custom events — `{ event: "agent/..." }` wiring; the dev server (`npx inngest-cli dev`) runs functions locally with the full execution model.
- **Serverless integration:** deploy to Vercel/Netlify/Lambda/Cloudflare Workers — the durable plane is managed, the function code runs in your own environment. **AI primitives:** `step.ai.infer` (LLM step with completion recording), `step.ai.tools` (durable tool-call steps), AgentKit for agent-loop scaffolding.

### 25.2 Restate

- **Virtual objects:** keyed, single-writer stateful objects — `restate.object({ name, handlers })`; per-key serialization maps to per-entity agents.
- **Durable RPC:** `ctx.call(service, "method", arg)` — journaled service calls that survive restarts.
- **Durable timers and state:** `ctx.sleep`, `ctx.set/get` — keyed state is part of the journaled log (event sourcing built in).
- **Deployment:** single binary (great for a laptop dev loop), Kubernetes operator, or Restate Cloud.

### 25.3 AWS Step Functions

- **Authoring:** Amazon States Language (ASL) JSON — Task/Choice/Parallel/Map/Wait states; retry and catch blocks per state; visual designer in the console.
- **Integrations:** 200+ AWS services via `.sync` / `.waitForTaskToken` patterns (Lambda, Bedrock for LLM calls, SQS, DynamoDB, EventBridge).
- **Operational notes:** Standard workflows cap at 1 year and are exactly-once; Express is cheap but at-least-once and capped at 5 minutes. Task Tokens approximate signals for HITL (human approves → resumes the state machine via the token).
- **When it's the right call:** everything already on AWS, simple-to-moderate chains, strong need for console-based review; migrate to code-first engines when the agent loop outgrows JSON.

## 26. Choosing the Engine

### 26.1 Decision Dimensions

| Requirement | Recommendation |
|---|---|
| Language/team stack: TypeScript | Inngest (serverless) or Temporal TS / Restate TS (self-host-capable) |
| Language: Python | Temporal Python (deepest durable-execution docs); Inngest Python for serverless |
| Language: Java/Go | Temporal (first-class SDKs); Restate Java/Kotlin |
| Language: C#/.NET | Azure Durable Functions |
| Deployment: serverless-first | Inngest, Step Functions, Durable Functions |
| Deployment: self-host on K8s | Temporal, Restate, Conductor |
| Workflow complexity: simple chains | Step Functions, Inngest |
| Complex orchestration + signals + sagas | Temporal |
| BPMN/human-task-centric processes | Camunda (Zeebe) |
| AI-agent focus | Inngest (serverless-first), Temporal (enterprise-scale) |
| Existing AWS estate | Step Functions |
| Existing Azure estate | Durable Functions |
| Netflix-style stack | Conductor |
| Team maturity / docs / community | Temporal — the deepest durable-execution ecosystem |

### 26.2 A Worked Decision

For a bank building a production agent platform: on Kubernetes with on-prem data-residency needs, **Temporal self-hosted** wins on maturity, signals/sagas/versioning, and audit-grade history. For a lean TypeScript team on Vercel/Lambda wanting durability without ops, **Inngest** wins. For a regulated human workflow (loan origination with defined roles), **Camunda** may be the right home with an agent augmenting it. The default for *agent-native* orchestration in 2026 is Temporal or Inngest; everything else is a stack-fit exception.

---
## 27. Reference Architecture

### 27.1 The Durable Agent Workflow Stack

```
API Gateway (HTTP/WSS)
  │ start workflow / stream status via queries
  ▼
Workflow Engine (Temporal / Inngest) ──▶ Agent Workflow (durable)
                                            ├─ LLM activities (retries)
                                            ├─ Tool activities (idempotent)
                                            └─ Human steps (signals)
       ▼                        ▼                        ▼
Observability          Tool Integrations          Memory Store
(UI, LangSmith/        (idempotent APIs)          (Redis / vector DB)
 Langfuse tracing)
```

- **API gateway** — receives user/event input, starts workflows (durable start: record the intent even if the engine is briefly down), streams status back via queries.
- **Workflow engine** — the durability substrate: event history, replay, timers, signals, retries, versioning.
- **Agent workflow** — deterministic orchestration: LLM activities, tool activities, human gates (Sections 12–19).
- **Tool integrations** — every side-effecting tool behind an idempotency-key layer (Section 21).
- **Memory store** — Redis/vector DB for conversation state and embeddings, separate from workflow state (Section 22; [Agent Runtime Cache Design](ai_llm/agent_runtime_cache_design_guide.md)).
- **Observability** — engine UI for workflow histories + tracing (LangSmith/Langfuse) for LLM spans; see [RAG Frameworks Comparison](ai_llm/rag_frameworks_comparison_guide.md) for tracing-tool guidance.

**Data flow on failure:** a worker dies mid-run — the gateway sees "workflow running" (durable), the engine reschedules, replay reconstructs state, the next activity executes, the gateway streams the result. No user-visible restart, no duplicate side effects, no lost cost.

---
## 28. Case Study: A Durable Loan-Approval Agent

### 28.1 The Workflow

A bank deploys a loan-approval agent on Temporal. The workflow:

```
receive application (signal) → credit check (activity, idempotent, retry 3x)
  → LLM risk assessment (activity, retry 5x, output recorded)
  → fraud screen (activity) → WAIT officer approval (signal, 48h → escalate)
  → notify applicant (activity) → DONE / compensation path on any rejection
```

```python
@workflow.defn
class LoanApprovalAgent:
    @workflow.run
    async def run(self, application_id: str) -> str:
        compensations: list[Callable] = []
        try:
            # 1. Credit check — idempotent external API, 3 retries
            credit = await workflow.execute_activity(credit_check, application_id,
                retry_policy=workflow.RetryPolicy(maximum_attempts=3, initial_interval=2.0))
            compensations.append(lambda: release_credit_hold(application_id))
            # 2. LLM risk assessment — 5 retries, backoff, output recorded
            risk = await workflow.execute_activity(llm_risk_assessment, credit,
                retry_policy=workflow.RetryPolicy(maximum_attempts=5, backoff_coefficient=2.0))
            await workflow.execute_activity(store_assessment, risk)
            # 3. Fraud screen
            await workflow.execute_activity(fraud_screen, application_id)
            # 4. HITL gate — durable wait, 48h timeout, escalation
            decision = None
            for level in ("officer", "senior_officer"):
                try:
                    decision = await workflow.wait_for_signal(
                        f"approval_{level}", timeout=timedelta(hours=48))
                    break
                except asyncio.TimeoutError:
                    await workflow.execute_activity(escalate,
                        {"app": application_id, "to": level})
            if decision is None:
                raise workflow.ApplicationError("no decision within SLA")
            if decision["approved"]:
                await workflow.execute_activity(disburse, application_id,
                    idempotency_key=f"loan-{application_id}")
                await workflow.execute_activity(notify, application_id, "APPROVED")
                return "APPROVED"
            raise workflow.ApplicationError("rejected by officer")
        except Exception:
            for compensate in reversed(compensations):   # saga
                await workflow.execute_activity(compensate)
            await workflow.execute_activity(notify, application_id, "REJECTED")
            raise
```

### 28.2 What Durability Buys This Agent

| Scenario | Without durability | With durability |
|---|---|---|
| LLM provider 5xx at step 2 | Run fails; restart re-spends credit + LLM cost | Retry with backoff; run continues |
| Worker pod evicted mid-run | Whole run lost; applicant re-submits | Replay; resumes at next activity |
| Officer approves, crash before notify | Applicant never notified; approval lost | Signal recorded; replay delivers notify |
| Officer never responds | Run stuck forever | 48h timer escalates to senior |
| Duplicate application event | Two parallel runs, double disbursement risk | Workflow ID dedup; one run |
| Disbursement retried | Double payment risk | Idempotency key; one payment |

**Fraud-screen integration:** the fraud step is a candidate for a child workflow (parallel vendor checks) reusing the [Financial Fraud Detection at Scale guide](../banking/financial_fraud_detection_at_scale_guide.md) models; durable execution makes the screen *repeatable evidence* — the same inputs always produce the recorded result, so a fraud decision can be replayed and defended in audit.

---
## 29. The Workflow History as an Audit Trail

### 29.1 History = The Complete Record

For banking, the workflow event history is not a debugging artifact — it is **the audit log**. Every step, input, output, retry, timer, signal, and human decision is recorded, in order, with timestamps. The loan-approval history shows: who submitted the application, what the credit check returned, which model produced the risk assessment (and the exact prompt/output), which officer approved (operator ID in the signal), how long the wait took, and what was notified. This maps directly onto regulatory expectations:

- **SR 11-7 / model governance** — model inputs, outputs, and versions are captured per run, supporting validation and ongoing monitoring of the LLM risk-assessment component.
- **BCBS 239 risk-data lineage** — every decision traces to its source data and model step; the history is a lineage graph.
- **Audit replay** — an auditor can replay a production history against the same code version to reproduce exactly what happened (the same mechanism as replay tests, Section 32).

### 29.2 Compliance Properties

- **Immutability:** history is append-only — steps cannot be silently edited post-hoc (Temporal supports namespace-level retention/archival to object storage for long-term custody).
- **Human approvals recorded:** signal payloads carry who/what/when — the four-eyes evidence is native to the record.
- **Retries documented:** every transient failure and its resolution is visible — operational resilience is auditable, not just claimed.
- **Searchable:** search attributes (customer ID, product, risk band) let compliance query "all loan approvals for customer X in Q3" directly.

### 29.3 Practical Compliance Design

Set retention/archival policies that match the jurisdiction's record-keeping horizon (MAS, GDPR, EU record rules); treat the history store as regulated data (encryption at rest, access control, no deletion paths); and export histories to the bank's enterprise archive for multi-year custody. See [Financial Risk & Compliance Systems](../banking/financial_risk_compliance_systems_guide.md) for the surrounding control framework.

---
## 30. Operational Concerns

### 30.1 Monitoring

| Metric | What it catches |
|---|---|
| Workflow started/completed/failed rates | Overall health; sudden failure spikes |
| Stuck workflows (no progress) | HITL waits without timeout, dead workers, engine stalls — **alert on age since last event** |
| Activity failure rates | LLM errors vs tool errors, partitioned by type (see [LLM Development Risks & Security](llm_development_risks_security_guide.md)) |
| Queue backlog (workers) | Worker starvation — more workers or faster activities needed |
| History size | Long workflows approaching limits — compact/archive (below) |
| Cost per workflow | LLM spend + engine infra per run — see [On-Prem LLM Deployment](on_prem_llm_deployment_guide.md) cost analysis |

### 30.2 History Size Management

Days-long workflows accumulate history; the fix is compaction (Temporal: history compaction / continue-as-new — reset history to a checkpoint while preserving state), archival (move old histories to object storage for audit), and passing references instead of payloads (Section 22). Design rule: keep per-step payloads out of history; keep the history as a *record*, not a store.

### 30.3 Cost

Two cost lines: **engine infrastructure** (cluster/Cloud subscription, workers) and **LLM spend per workflow** (the dominant line). Durable execution cuts LLM spend by eliminating re-execution; add per-workflow budgets (Section 20.3) and cache aggressively ([Agent Runtime Cache Design](ai_llm/agent_runtime_cache_design_guide.md)).

---
## 31. Failure Modes

| Failure mode | What happens | Mitigation |
|---|---|---|
| **Worker crash** | Workflow pauses; engine reschedules; replay reconstructs state | Safe by design — replay is the recovery mechanism |
| **Engine outage** | Self-host: workflows pause until cluster recovers; managed: provider SLA | Self-host HA (Temporal cluster, multi-AZ); managed SLA with failover |
| **Stuck workflow** | No progress (dead signal wait, no timeout) | Every wait has a timeout + escalation; monitor age-since-last-event |
| **Poisoned history** | Deploy changed workflow code; in-flight replay branches differently → corrupt state | Versioning (getVersion/patch, worker versioning — Section 19); replay tests before deploy |
| **Activity storm** | Unbounded retries hammer a failing dependency | Cap attempts, backoff ceilings, circuit breakers, per-provider quotas |
| **History overflow** | Giant workflow hits history limits | Compaction / continue-as-new / pass references |
| **Double side effects** | Non-idempotent activity retried | Idempotency keys on every side-effecting tool (Section 21) |
| **Lost signals** | Signal delivered during replay window | Engine re-delivers signals deterministically during replay |

---
## 32. Testing Durable Workflows

### 32.1 The Test Pyramid

- **Unit tests for workflow logic:** run the workflow deterministically with mocked activities — the engine's test SDK executes the workflow function directly and fast-forwards timers/signals (Temporal: `WorkflowEnvironment.start_local` + `temporalio.testing`). Assert the step sequence, branches, and compensation path.
- **Integration tests with real activities:** use Testcontainers (Temporal has official testcontainers support) to run a real engine + real activities against test doubles for external systems; verify retries, idempotency keys, and signal handling end-to-end.
- **Golden path + failure path tests:** every workflow gets two suites — the happy path (approve → disburse → notify) and every failure path (credit check fails, officer times out, saga compensates). The failure paths are where durability bugs live.

### 32.2 Replay Tests — The Killer Feature

Replay a *production event history* against *new code* to prove the new version is compatible with in-flight workflows (Temporal: `replayer`). This is the safety net that makes versioned deploys (Section 19) and prompt/orchestration changes routine: if replay of last week's real histories succeeds against today's code, deploys are safe. **Run replay tests in CI on every workflow-code change.**

### 32.3 Chaos Testing

Kill workers mid-workflow, restart the engine, partition the network, inject activity failures — and assert the workflow still completes with exactly-once side effects. The [Low-Latency C++ Development guide](low_latency_cpp_development_guide.md) covers failure-injection testing discipline generally; for agents, the chaos suite should specifically verify: resume-after-crash, no double side effects, signal re-delivery, and timer accuracy across restarts.

---
## 33. Anti-Patterns and Pitfalls

| Pitfall | Why it breaks | Fix |
|---|---|---|
| **LLM calls in workflow code** | Non-deterministic → replay diverges → corrupted state | All LLM/tool/network work goes in activities |
| **`random()`, `time()`, `hash()` in workflow code** | Breaks replay determinism | Use activities for any non-determinism; use engine timers for time |
| **Unbounded retries** | Retry storms on failing dependencies | Cap attempts; backoff ceilings; circuit breakers |
| **Missing idempotency on tool side effects** | Duplicate payments/emails on retry | Idempotency keys everywhere (Section 21) |
| **No versioning strategy** | Deploys corrupt in-flight workflows | getVersion/patch markers; worker versioning; replay tests |
| **Giant monolith workflow** | History bloat; slow replay; hard to test | Split into child workflows (Section 15) |
| **Ignoring history size** | Slow replay, engine limits, storage cost | Compaction/archival; pass references not payloads |
| **Blocking I/O in workflows** | Violates determinism; freezes replay | Activities and durable timers only |
| **Treating durable execution as a cache** | It is not a cache — it is the source of truth; evicting history loses the audit trail | History is the record; caches sit in front of it |
| **No monitoring of stuck workflows** | Human waits without escalation rot silently | Timeouts + escalation + age-based alerting (Section 30) |
| **Regenerating idempotency keys on retry** | Defeats dedup — double side effects | Derive keys from workflow/activity context, stable across retries |
| **Non-deterministic iteration over sets/dicts** | Replay orders differ → different branches | Sort/order iteration deterministically in workflow code |

---
## 34. The Future: Durable Execution as the Default Substrate for Agents

### 34.1 2026 and Beyond

- **Durable execution as the default substrate:** agents are becoming *workflows with LLM activities* — leading agent frameworks layer planning/tool-use on durable cores (Inngest AgentKit, Temporal-centric SDKs), and the "agent = durable workflow" framing is replacing hand-rolled loops.
- **Agent-native workflow SDKs:** AI-specific abstractions on Temporal/Inngest — LLM steps, tool-call steps, memory steps, HITL steps as first-class SDK primitives rather than DIY activities.
- **Durable memory:** workflow state and memory stores are converging — engines gaining native keyed state (Restate virtual objects), memory systems gaining journaled writes; see [Beyond RAG](ai_llm/beyond_rag_guide.md).
- **MCP + durable execution:** MCP tool calls become durable activities — the protocol supplies the tool surface, the engine supplies retries/idempotency/recording; see [MCP Framework & Tools](ai_llm/mcp_framework_tools_guide.md).
- **Cost-aware orchestration:** engines optimizing LLM token spend — step-level caching, model tiering, budget-aware routing as built-in features.
- **Multi-agent durable orchestration:** agent teams as durable workflow graphs — parent/child workflows for coordination, sagas for cross-agent transactions; see [Hybrid Multi-Agent Systems](ai_llm/hybrid_multi_agent_systems_guide.md).

### 34.2 The Trajectory for Banking

For banks, the direction is clear: regulated agent platforms will standardize on durable execution because it is the only model that simultaneously delivers reliability, cost control, HITL governance, and audit-grade history. The 2026 question is not *whether* to make agents durable — it is *which* engine, and how quickly the compliance framework (Section 29) can be stood up around it.

---
## 35. Conclusion

Durable AI agent workflows are the difference between agents as demos and agents as production infrastructure. The core idea — record every step, replay on failure, resume from the last completed step — transforms the failure profile: LLM calls become recorded facts instead of re-spend risks, tool calls become safe under retry through idempotency, human approvals become durable auditable gates, and crashes become invisible pauses.

The practice: put every LLM and tool call in an activity; keep workflow code deterministic; retry with capped exponential backoff; make every side-effecting tool idempotent; gate consequential actions behind human signals with timeouts and escalation; version workflow code; treat the history as both recovery mechanism and audit trail. Choose the engine on stack fit — Temporal for enterprise scale, Inngest for serverless-first teams, Step Functions/Durable Functions for cloud estates, Camunda for structured human processes — then invest in replay tests, chaos tests, and stuck-workflow monitoring. Durability is only as good as the discipline around it.

**Decision guidance:** in a regulated environment, adopt a durable execution engine from day one — retrofitting durability after an agent has moved money is not an engineering task, it is an incident.

---
## 36. References

### 36.1 Sibling Guides in This Repository

- [Hybrid Multi-Agent Systems](ai_llm/hybrid_multi_agent_systems_guide.md) — multi-agent coordination, agent teams as workflow graphs
- [Hierarchical Multi-Agent Frameworks](ai_llm/hierarchical_multi_agent_frameworks_guide.md) — hierarchy patterns for agent systems
- [Research Agents](research_agents_guide.md) — the research-agent loop and parallel-researcher pattern
- [Agent Runtime Cache Design](ai_llm/agent_runtime_cache_design_guide.md) — caching LLM calls and deterministic parts
- [MCP Framework & Tools](ai_llm/mcp_framework_tools_guide.md) — MCP tool surface for durable activities
- [Event Stream Processing](event_stream_processing_guide.md) — Kafka/streams ingestion for event-driven agents
- [LLM Latency Optimization](ai_llm/llm_latency_optimization_guide.md) — retry/backoff guidance for LLM calls
- [Beyond RAG](ai_llm/beyond_rag_guide.md) — memory systems for agents
- [LLM Development Risks & Security](llm_development_risks_security_guide.md) — excessive agency, HITL for consequential actions
- [Payments Hub](../banking/payments_hub_guide.md) — sagas, idempotency keys, money-movement safety
- [Financial Fraud Detection at Scale](../banking/financial_fraud_detection_at_scale_guide.md) — fraud-screening models for agent steps
- [Financial Risk & Compliance Systems](../banking/financial_risk_compliance_systems_guide.md) — SR 11-7, BCBS 239, model governance
- [On-Prem LLM Deployment](on_prem_llm_deployment_guide.md) — self-hosted LLM cost analysis
- [RAG Frameworks Comparison](ai_llm/rag_frameworks_comparison_guide.md) — tracing/observability tooling (LangSmith/Langfuse)
- [Low-Latency C++ Development](low_latency_cpp_development_guide.md) — chaos/failure-injection testing discipline

### 36.2 External Sources

- Temporal — [temporal.io](https://temporal.io), docs at [docs.temporal.io](https://docs.temporal.io); SDKs (Python/TS/Go/Java), dev server, Cloud, UI
- Inngest — [inngest.com](https://www.inngest.com/docs); durable steps, `step.ai.infer`, AgentKit
- Restate — [restate.dev](https://restate.dev); virtual objects, durable RPC, SDKs
- AWS Step Functions — [docs.aws.amazon.com/step-functions](https://docs.aws.amazon.com/step-functions); ASL, Standard vs Express
- Azure Durable Functions — [learn.microsoft.com/azure/azure-functions/durable](https://learn.microsoft.com/azure/azure-functions/durable)
- Netflix Conductor — [conductor.netflix.com](https://conductor.netflix.com)
- Camunda / Zeebe — [camunda.com](https://camunda.com)
- Cadence — [cadenceworkflow.io](https://cadenceworkflow.io)

---
*Product facts and version details in this guide are as of August 2026. Verify against the official vendor documentation before procurement or architecture decisions.*




