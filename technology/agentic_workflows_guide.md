# Agentic Workflows: The Canonical Design Pattern Catalog

**Workflows vs Agents — the Pattern Taxonomy (Prompt Chaining, Routing, Parallelization, Orchestrator-Workers, Evaluator-Optimizer, the Agent Loop), the Decision Framework, and Implementation Guidance**

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Cymbal Bank
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** LLM/AI Engineering Guides · **Topic:** Agentic Workflow & Agent Design Patterns
> **Focus:** Banking & regulated industries (Singapore, EU, global)
> **Companion Guides:** [Hybrid Multi-Agent Systems](ai_llm/hybrid_multi_agent_systems_guide.md) · [Hierarchical Multi-Agent Frameworks](ai_llm/hierarchical_multi_agent_frameworks_guide.md) · [Durable AI Agent Workflows](durable_ai_agent_workflows_guide.md) · [Research Agents](research_agents_guide.md) · [Beyond RAG](ai_llm/beyond_rag_guide.md) · [MCP Framework & Tools](ai_llm/mcp_framework_tools_guide.md) · [Constrained Decoding Frameworks](constrained_decoding_frameworks_guide.md) · [LLM Development Risks & Security](llm_development_risks_security_guide.md) · [Financial Risk & Compliance Systems](../banking/financial_risk_compliance_systems_guide.md) · [Payments Hub](../banking/payments_hub_guide.md)
> **Last Updated:** August 2026

---

## Table of Contents

1. [What This Guide Covers](#1-what-this-guide-covers)
2. [The Foundational Distinction: Workflows vs Agents](#2-the-foundational-distinction-workflows-vs-agents)
3. [Workflows vs Agents: The Decision Framework](#3-workflows-vs-agents-the-decision-framework)
4. [The Workflow Pattern Catalog: Overview](#4-the-workflow-pattern-catalog-overview)
5. [Pattern 1 — Prompt Chaining](#5-pattern-1--prompt-chaining)
6. [Pattern 2 — Routing](#6-pattern-2--routing)
7. [Pattern 3 — Parallelization](#7-pattern-3--parallelization)
8. [Pattern 4 — Orchestrator-Workers](#8-pattern-4--orchestrator-workers)
9. [Pattern 5 — Evaluator-Optimizer](#9-pattern-5--evaluator-optimizer)
10. [The Agent Pattern](#10-the-agent-pattern)
11. [Hybrid Approaches: Workflows + Agents](#11-hybrid-approaches-workflows--agents)
12. [The Pattern Selection Framework](#12-the-pattern-selection-framework)
13. [Combining Patterns: Real System Architectures](#13-combining-patterns-real-system-architectures)
14. [Workflow-as-Code: Implementing Patterns with Frameworks](#14-workflow-as-code-implementing-patterns-with-frameworks)
15. [Banking Applications](#15-banking-applications)
16. [Anti-Patterns and Pitfalls](#16-anti-patterns-and-pitfalls)
17. [The Future: Agentic Workflows in 2026 and Beyond](#17-the-future-agentic-workflows-in-2026-and-beyond)
18. [Conclusion](#18-conclusion)
19. [References](#19-references)

---

## 1. What This Guide Covers

This guide is the **canonical design pattern catalog for LLM-driven workflows and agents** — the building blocks, the decision framework for choosing among them, and workflow-as-code implementation guidance. It answers three questions every architect building on LLMs must answer before writing code:

1. **What are the building blocks?** A small, finite set of reusable patterns — five workflow patterns (prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer) plus the agent loop — covering the overwhelming majority of production LLM systems.
2. **Which pattern do I choose?** A decision framework grounded in task structure: well-defined? parallelizable? decomposable? iteratively improvable? open-ended? — which selects the pattern and, critically, decides whether you need an *agent* at all.
3. **How do I implement it?** Mapping patterns to frameworks (LangGraph, LlamaIndex Workflows, Haystack, Agent SDKs, durable engines) with concrete code sketches.

The taxonomy follows Anthropic's **"Building Effective Agents"** essay (December 2024), the de-facto vocabulary for LLM application design.

**Scope and sibling guides.** This guide deliberately does *not* cover: multi-agent topologies (supervisor hierarchies, peer collaboration) — see [Hybrid Multi-Agent Systems](ai_llm/hybrid_multi_agent_systems_guide.md) and [Hierarchical Multi-Agent Frameworks](ai_llm/hierarchical_multi_agent_frameworks_guide.md); reliability mechanics (retries, checkpoints, sagas, HITL as a durability primitive) — see [Durable AI Agent Workflows](durable_ai_agent_workflows_guide.md); research-agent discipline — see [Research Agents](research_agents_guide.md); security and risk controls — see [LLM Development Risks & Security](llm_development_risks_security_guide.md). The patterns here are the *shapes*; the sibling guides supply the *controls*.

---

## 2. The Foundational Distinction: Workflows vs Agents

### 2.1 Anthropic's Framing (December 2024)

In December 2024, Anthropic published **"Building Effective Agents"**, consolidating production experience (Claude Code, customer deployments) into a vocabulary the industry adopted nearly verbatim. Its core move was a crisp two-way taxonomy:

> **Workflows** are systems where LLMs and tools are orchestrated through **predefined code paths**. **Agents** are systems where LLMs **dynamically direct their own processes and tool usage**, maintaining control over how they accomplish tasks.

The distinction is not about intelligence or capability — it is about **where control flow lives**:

| Dimension | Workflow | Agent |
|---|---|---|
| Control flow | Written in code, fixed at design time | Decided by the LLM at runtime |
| Path | Predefined orchestration path | Dynamic, emergent, adaptive |
| Predictability | Deterministic structure | Probabilistic structure |
| Audit story | Steps known and documented in advance | Steps emerge at runtime; must be logged |
| Best tasks | Well-defined, stable, repeatable | Open-ended, ambiguous, changing |

The essay's headline guidance — often quoted, frequently ignored — anchors every decision in this guide:

> **Find the simplest solution possible, and only increase complexity when needed.**

If you can implement a task with a prompt, do that. If you need multiple LLM calls, prefer a workflow. Only when the task genuinely requires the model to decide *how* to proceed should you build an agent. Agents trade **control** (determinism, auditability, cost predictability) for **flexibility** (adaptation, open-endedness) — a trade worth making only when flexibility is the requirement.

### 2.2 Workflows: Predefined Orchestration Paths

A workflow is a **program** in which some steps are LLM calls. The sequence, branching, and merging are written by a developer; the LLM contributes output at each step but never decides the *shape* of the process.

```
        ┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
Input → │ LLM A   │ →   │ LLM B   │ →   │ LLM C   │ →   │ Output  │
        └─────────┘     └─────────┘     └─────────┘     └─────────┘
        The path is fixed in code; the LLM never chooses the path.
```

Workflow characteristics: **known steps** (enumerable before the run), **deterministic skeleton** (same input, same path, modulo constrained routing), **validation points** (output checkable before handoff), **bounded latency/cost** (SLA-quotable: total ≈ Σ steps), and **auditable by construction** (the step list *is* the documentation). Workflows are the right default for any task whose process is defined — in banking, that is most tasks: KYC, credit, compliance screening, trade validation.

### 2.3 Agents: Dynamically Directed Processes

An agent is an LLM **empowered to decide what to do next** — which tool to call, in what order, whether to retry, whether to change approach — until it judges the task complete.

```
   Task ──► ┌────────┐ ──► ┌────────┐ ──► ┌────────┐ ──► Result
            │Reason  │     │  Act   │     │ Observe│
            │ (LLM)  │     │(tool)  │     │ (result)│
            └────────┘     └────────┘     └────────┘
                ▲              │               │
                └──────────────┴───────────────┘
            loop until the LLM decides "done"
```

Agent characteristics: **unknown steps** (number and order emerge at runtime), **adaptive behavior** (course-corrects on results), **tool selection is a model decision** (which tool, when, with what arguments — the essence of agency), **unbounded cost/latency** (iterations × step cost), and **harder audit** (the path exists only in the transcript). Agents fit tasks whose process does not exist yet — open-ended research, novel problem-solving, exploratory analysis — where writing the control flow by hand costs more than tolerating the model's control.

### 2.4 The Spectrum: Prompts → Workflows → Agents

Not three disjoint categories but a **spectrum of control handed to the model**:

| Stage | Structure | Who decides the path? | Cost / latency |
|---|---|---|---|
| Simple prompt | One LLM call, no tools | Developer (only one step) | Lowest |
| Workflow | 2–N LLM calls, fixed graph | Developer (code) | Σ steps, bounded |
| Agent | LLM + tools in a loop | Model (runtime) | Variable, unbounded |

**The spectrum is a staircase you climb only when the step below fails.** Start with a single well-engineered prompt; chain calls in a workflow when the task needs multiple passes; let an agent decide only when the branches would be too numerous or unknowable to write by hand. Most teams climb too fast — the #1 mistake in the field (Section 16.1).

### 2.5 The Guiding Principle: Use the Simplest Solution That Works

Three operational corollaries: **(1)** a workflow that solves the task beats an agent that might — predictability and control are features, not shortcomings; **(2)** agents are a complexity budget you spend deliberately — every agent adds loop risk, tool-abuse surface, cost variance, and audit burden; **(3)** the question is never "agent or not" but "where on the spectrum does this task live?" — most production systems end up mixed (Section 11).

---

## 3. Workflows vs Agents: The Decision Framework

### 3.1 Use Workflows When...

- **The task is well-defined with known, stable steps** — you can enumerate them in advance even if each step's *content* varies ("extract invoice fields, validate, translate, format"), and **predictability matters** (QA, demos, regulators).
- **Latency and cost must be predictable** — you quote an SLA per transaction (see [On-Prem LLM Deployment](on_prem_llm_deployment_guide.md) for cost modeling).
- **Compliance and auditability are required** — the process definition must be reviewable and each run must map to a documented procedure (SR 11-7 model-risk expectations, MAS guidelines — see [LLM Development Risks & Security](llm_development_risks_security_guide.md) and [Financial Risk & Compliance Systems](../banking/financial_risk_compliance_systems_guide.md)).

### 3.2 Use Agents When...

- **The task is open-ended with unknown steps** — no fixed procedure, and you cannot enumerate what must happen ("investigate why this trade failed", "research this sector", "draft a regulator response"); the model must discover the path.
- **The task must adapt and select tools** — requirements change mid-task, inputs vary wildly, environments are unfamiliar, and value comes from the model *choosing* among tools and combining them.
- **You can tolerate unpredictability** — cost/latency/behavior variance is acceptable or mitigated by guardrails (budgets, timeouts, human gates).

The full selection tree (which pattern within each territory) is Section 12.2.

---

## 4. The Workflow Pattern Catalog: Overview

Anthropic's essay distilled production practice into **five workflow patterns** — the vocabulary of LLM application design. Name them in design reviews, map them to frameworks (Section 14), and combine them (Section 13) as you would software design patterns.

| # | Pattern | Core idea | One-line when-to-use |
|---|---|---|---|
| 1 | **Prompt Chaining** | Sequential LLM calls; each processes the previous output | Task decomposes into a fixed sequence of dependent steps |
| 2 | **Routing** | Classify input, dispatch to specialized handler | Distinct input categories need different handling |
| 3 | **Parallelization** | Multiple independent LLM calls run concurrently | Subtasks independent; speed or variance reduction matters |
| 4 | **Orchestrator-Workers** | Central LLM decomposes, dispatches workers, synthesizes | Complex task with unknown subtask structure needing synthesis |
| 5 | **Evaluator-Optimizer** | Generator + evaluator iterate until quality bar met | Clear evaluation criteria and a high quality bar |

The patterns are **composable** — production pipelines are usually several nested (Section 13). Ordering is roughly increasing in structural complexity and in control delegated to the model.

**Common production hygiene** (every pattern): structured inputs/outputs at each step ([Constrained Decoding Frameworks](constrained_decoding_frameworks_guide.md)); validation gates between steps; per-step error handling (retries with backoff, typed fallbacks); per-step observability (tokens, latency, output hashes — [RAG Frameworks Comparison](ai_llm/rag_frameworks_comparison_guide.md)); durability for anything long-running or state-touching (Section 14.5).

---

## 5. Pattern 1 — Prompt Chaining

### 5.1 The Pattern

**Prompt chaining** decomposes a task into a **fixed sequence of subtasks**, each LLM call processing the previous output:

```
  Input ──► [draft] ──► [extract] ──► [translate] ──► [format] ──► Output
             LLM A        LLM B         LLM C          LLM D
```

The chain's power is **separation of concerns**: each call has one job, a narrow context, a tight schema. A mega-prompt doing all four steps in one call is harder to control, expensive to iterate, and fails opaquely; the chain makes each failure local and fixable.

### 5.2 Canonical Examples

- **Document generation**: outline → sections → tone review → format; and **extract → translate → format**: extract structured fields, translate, render into a target format — narrow competencies, one pipeline.
- **Query rewriting in RAG**: rewrite query → hypothetical answer → retrieve → synthesize (see [Beyond RAG](ai_llm/beyond_rag_guide.md)).
- **Long-document Q&A**: extract relevant passages (step 1) → answer from passages only (step 2), keeping the answer step's context small and grounded.

### 5.3 When to Use

- The task **decomposes into a fixed sequence**, each step improving on the previous output, and **each step adds value** — remove any step that doesn't measurably improve the result.
- **Latency is acceptable** — steps are sequential (total = sum); if too long, parallelize independent steps (Section 7) or cache intermediates.
- You want **per-step control**: different models, validation, and budgets per step.

### 5.4 Implementation Guidance

- **Validate output at each step** — use structured outputs (JSON schema) so step N+1 receives typed data, not prose ([Constrained Decoding Frameworks](constrained_decoding_frameworks_guide.md)).
- **Pass validated output, not raw output** — a failing step triggers a bounded retry (re-prompt with the validation error) or a fallback; never forward garbage (Section 16.3).
- **Keep each prompt narrow** — one job per step; don't re-describe the whole pipeline.
- **Model-tier the chain** — cheap/fast models for mechanical steps (extract, format), powerful models for judgment steps (draft, synthesize) — the cost-optimization core of chaining (see [Hybrid Multi-Agent Systems](ai_llm/hybrid_multi_agent_systems_guide.md) model routing).
- **Make the chain observable** — log input/output hashes, tokens, latency per step so failures identify the step (Section 16.10).

### 5.5 Code Sketch

```python
# Prompt chaining — extract → translate → format, with validation gates.
class ExtractedInvoice(BaseModel):
    vendor: str; currency: str; amount: float; line_items: list[str]
def step_extract(raw_doc: str) -> ExtractedInvoice:
    out = llm.complete(model="cheap-fast-model",               # mechanical → cheap tier
        system="Extract invoice fields as JSON. Follow the schema exactly.",
        messages=[{"role": "user", "content": raw_doc}],
        response_format=ExtractedInvoice)                      # structured output
    return ExtractedInvoice.model_validate_json(out)           # validation gate

def step_translate(inv: ExtractedInvoice, lang: str) -> ExtractedInvoice:
    out = llm.complete(model="cheap-fast-model",
        system="Translate the line items. Return the same JSON schema.",
        messages=[{"role": "user", "content": inv.model_dump_json()}],
        response_format=ExtractedInvoice)
    return ExtractedInvoice.model_validate_json(out)
def step_format(inv: ExtractedInvoice, template: str) -> str:
    out = llm.complete(model="mid-tier-model",
        system="Render this invoice data into the provided report template.",
        messages=[{"role": "user", "content": f"{template}\n\n{inv.model_dump_json()}"}])
    if len(out) < 10: raise StepValidationError("empty formatter output")
    return out

def run_pipeline(raw_doc: str, lang: str, template: str) -> str:
    inv = step_translate(step_extract(raw_doc), lang)   # sequential by construction
    return step_format(inv, template)
```

**LangGraph form** (linear graph — Section 14.7.1): `extract → translate → format` as three nodes with plain edges; state carries `doc → extracted → translated → formatted`.

---

## 6. Pattern 2 — Routing

### 6.1 The Pattern

**Routing** classifies an input and dispatches it to a **specialized handler** — a different prompt, model, or downstream agent/workflow:

```
        Input ────► [Classifier] ──┬─► [Handler A: billing prompt]
                    (LLM constrained │─► [Handler B: technical agent]
                     output or       │─► [Handler C: fraud workflow]
                     embeddings)     └─► [Fallback: generalist]
```

Routing buys **specialization**: one generalist that does everything passably loses to several specialists each tuned for their slice. A **fallback** ensures unknown inputs are still handled (Section 16.4).

### 6.2 Canonical Examples

- **Customer support triage**: billing → billing prompt; technical → technical agent; fraud → fraud workflow; else → generalist fallback.
- **Model routing (cost optimization)**: easy → cheap small model; hard → powerful flagship. In production 70–90% of traffic is easy, so routing saves the bulk of the LLM budget (see [Hybrid Multi-Agent Systems](ai_llm/hybrid_multi_agent_systems_guide.md) model routing; [On-Prem LLM Deployment](on_prem_llm_deployment_guide.md)).
- **Query-type routing in RAG**: factual lookup → retrieval pipeline; synthesis → reasoning pipeline; code → code assistant (see [Beyond RAG](ai_llm/beyond_rag_guide.md), [Advanced RAG Techniques](ai_llm/advanced_rag_techniques_guide.md)).
- **Risk-tiered routing in banking**: applications classified into risk tiers, each dispatched to a different approval workflow (Section 15.1).

### 6.3 When to Use

- **Distinct categories with materially different handling** — if all categories would use the same prompt/model, routing is pointless.
- **Accuracy matters** — classifier accuracy is the ceiling; each handler is tuned for its slice.
- **Cost optimization** — route by difficulty or modality to the cheapest adequate model.

### 6.4 Implementation Guidance

- **Classifier options**: (a) LLM with constrained output (single enum token — cheap, robust to paraphrasing; default), (b) embedding classifier (nearest-centroid over labeled examples — fast, no LLM latency; good at high volume), (c) rules/regex for unambiguous signals.
- **Routing table as data** — categories → handlers in a dict/config/DB, not if/else spaghetti; adding a category is a config change.
- **Fallback is mandatory** — every router needs a default handler for out-of-distribution inputs.
- **Emit a confidence score** — below threshold → fallback instead of a forced guess; this single practice prevents most routing failures.
- **Log the routing decision** — category, confidence, handler chosen (prime audit material: which process ran, and why).

### 6.5 Code Sketch

```python
Category = Literal["billing", "technical", "fraud", "general"]

def classify(request: str) -> tuple[Category, float]:
    out = llm.complete(model="cheap-fast-model",
        system='Classify the request as one of: billing, technical, fraud, '
               'general. Return JSON: {"category": "...", "confidence": 0-1}',
        messages=[{"role": "user", "content": request}],
        response_format=ClassificationResult)              # constrained output
    r = ClassificationResult.model_validate_json(out)
    return r.category, r.confidence
ROUTING_TABLE = {"billing": handle_billing_prompt,
                 "technical": handle_technical_agent,
                 "fraud": handle_fraud_workflow}

def route(request: str) -> str:
    category, confidence = classify(request)
    handler = ROUTING_TABLE.get(category) if confidence >= 0.6 else None
    return (handler or handle_general_fallback)(request)   # fallback never crashes
```

**LangGraph form** (conditional edges — Section 14.7.2): a `classify` node; `add_conditional_edges` maps the category string to the handler node, with `general` as the fallback destination.

---

## 7. Pattern 3 — Parallelization

### 7.1 The Pattern

**Parallelization** runs multiple independent LLM calls concurrently, then aggregates. Two canonical forms (per Anthropic):

1. **Sectioning**: split a task into independent sections, process in parallel, assemble — e.g., five report sections written concurrently, then merged.
2. **Voting**: run the *same* task N times, pick the best/majority — variance reduction through redundancy (the LLM analog of ensembles; see [Advanced RAG Techniques](ai_llm/advanced_rag_techniques_guide.md) on self-consistency).

```
Sectioning:                          Voting:
  Task ─┬─► [Worker: section A]      Task ─┬─► [Attempt 1] ─┐
        ├─► [Worker: section B]           ├─► [Attempt 2] ─┼─► [Aggregate:
        └─► [Assemble] ──► Output         └─► [Attempt 3] ─┘   merge/vote] ─► Output
```

The win is **latency** (wall-clock = slowest worker, not the sum) and, for voting, **quality** (independent attempts make different mistakes).

### 7.2 Canonical Examples

- **Report generation**: independent sections in parallel, assemble with intro/conclusion pass; ~N× latency cut versus a chain.
- **Self-consistency in RAG**: sample N answers, select the most consistent/majority — a documented accuracy booster (see [Advanced RAG Techniques](ai_llm/advanced_rag_techniques_guide.md)).
- **Multi-perspective review**: N reviewers critique from different angles (compliance, clarity, risk), merge feedback.
- **Chunked document analysis**: analyze chunks in parallel, aggregate findings.

### 7.3 When to Use

- **Subtasks are genuinely independent** — shared state or ordering dependencies produce wrong results (Section 16.6).
- **Speed matters** — wall-clock latency is the constraint (banking: batch screening windows, intraday reporting).
- **Variance reduction helps** — high-stakes or high-noise tasks where voting measurably improves quality; voting multiplies cost by N, so only where the quality gain justifies it.

### 7.4 Implementation Guidance

- **Async/parallel calls** — `asyncio.gather` or a worker pool with per-worker timeouts so one stuck call doesn't stall the fan-in.
- **Aggregate with a merge step** — sectioning: a final pass assembling/polishing sections; voting: majority, self-consistency score, or a judge LLM.
- **Consistency checks in merge** — detect contradictions between workers (two sections quoting different numbers) and flag, not silently pass through (Section 16.6).
- **Respect rate limits** — N parallel calls hit the provider simultaneously; cap concurrency, add jitter (see [Durable AI Agent Workflows](durable_ai_agent_workflows_guide.md), [On-Prem LLM Deployment](on_prem_llm_deployment_guide.md)).
- **Make the fan-out durable** — the canonical fan-out/fan-in shape; on a durable engine, workers are child activities that survive crashes (Section 14.5).

### 7.5 Code Sketch

```python
async def write_section(outline: str, section: str) -> str:
    return await llm.acomplete(model="mid-tier-model",
        system=f"Write section '{section}' following the outline.",
        messages=[{"role": "user", "content": outline}])

async def write_report_parallel(outline: str, sections: list[str]) -> str:
    results = await asyncio.gather(*(write_section(outline, s) for s in sections),
                                   return_exceptions=True)
    drafts = {s: r for s, r in zip(sections, results) if not isinstance(r, Exception)}
    return await llm.acomplete(model="powerful-model",        # merge + polish
        system="Assemble these section drafts into a coherent report. "
               "Flag any contradictions between sections.",
        messages=[{"role": "user", "content": json.dumps(drafts)}])
async def vote_answer(question: str, n: int = 3) -> str:
    attempts = await asyncio.gather(*(llm.acomplete(
        model="powerful-model", messages=[{"role": "user", "content": question}])
        for _ in range(n)))
    return most_self_consistent(attempts)   # majority / consistency / judge
```

**LangGraph form** (branches + join — Section 14.7.3): one node per section, all edged into a `merge` node; the merge node fires only when all upstream branches complete (join semantics).

---

## 8. Pattern 4 — Orchestrator-Workers

### 8.1 The Pattern

**Orchestrator-workers** uses a central orchestrator LLM to **decompose** a complex task into subtasks, **dispatch** them to worker LLMs/tasks, and **synthesize** the results:

```
                        ┌────────────────┐
                        │  ORCHESTRATOR  │  LLM: plan sub-tasks,
                        └───┬─────┬─────┬┘  dispatch, synthesize
              ┌─────────────┘     │     └─────────────┐
              ▼                   ▼                   ▼
        [Worker A]          [Worker B]          [Worker C]
              └─────────────┐     │     ┌─────────────┘
                            ▼     ▼     ▼
                        ┌────────────────┐
                        │   SYNTHESIS    │  orchestrator integrates outputs
                        └────────────────┘  → final result
```

This is the pattern for tasks **too big or too open for a fixed workflow** — the subtask structure is *unknown until the orchestrator plans it*. Contrast with chaining (structure fixed by the developer): here the *orchestrator* decides the structure at runtime.

### 8.2 Canonical Examples

- **Research**: orchestrator plans sub-questions → workers research each independently → orchestrator synthesizes with sources (see [Research Agents](research_agents_guide.md) for question decomposition, evidence handling, citation discipline).
- **Coding**: orchestrator decomposes a feature into files/modules → workers implement each → orchestrator integrates and reviews (see [Coding Agents](ai_llm/coding_agents_research.md)).
- **Multi-source analysis**: "analyze credit risk / market risk / operational risk" dispatched to specialists, then synthesized into a risk assessment.
- **Large report generation**: orchestrator plans sections and assigns each to a worker with the right model tier.

### 8.3 When to Use

- **Complex tasks with unknown subtask structure** — the decomposition varies per input, so it cannot be written by hand.
- **Subtasks are parallelizable** — workers run concurrently, cutting wall-clock time.
- **Synthesis is required** — someone must integrate, reconcile, and polish.

### 8.4 Implementation Guidance

- **Orchestrator loop**: plan → dispatch → collect → (re-plan if needed) → synthesize. The orchestrator re-plans on worker failure or poor results — retry, re-decompose, or proceed.
- **Worker dispatch**: workers are child tasks — child workflows on a durable engine (Section 14.5; [Durable AI Agent Workflows](durable_ai_agent_workflows_guide.md)), sub-graphs in a framework (Section 14.7.4). Workers may themselves be agents or workflows — the pattern is recursive.
- **Result synthesis under partial failure**: design the synthesis prompt to work with whatever subset succeeded; record what was missing.
- **Bounded planning**: cap subtask count and worker budget — an orchestrator planning 50 subtasks for a 5-minute task is over-orchestrating (Section 16.2).
- **Pass context, not history**: each worker gets a self-contained task spec (goal + inputs + output schema), not the whole conversation.

### 8.5 Code Sketch

```python
async def orchestrator(task: str, max_workers: int = 5) -> str:
    # 1. PLAN — orchestrator LLM decomposes the task.
    plan = OrchestratorPlan.model_validate_json(await llm.acomplete(
        model="powerful-model",
        system=(f"Decompose this task into at most {max_workers} independent "
                "subtasks. For each: id, self-contained instruction, output "
                "schema. Return JSON matching OrchestratorPlan."),
        messages=[{"role": "user", "content": task}],
        response_format=OrchestratorPlan))
    # 2. DISPATCH — workers run in parallel (each a task / child workflow).
    async def run_worker(sub: Subtask) -> WorkerResult:
        try:
            out = await llm.acomplete(model=sub.model_tier, system=sub.instruction,
                messages=[{"role": "user", "content": sub.inputs}],
                response_format=sub.output_schema)
            return WorkerResult(sub.id, ok=True, data=out)
        except Exception as e:
            return WorkerResult(sub.id, ok=False, error=str(e))
    results = await asyncio.gather(*(run_worker(s) for s in plan.subtasks))
    # 3. SYNTHESIZE — integrate, tolerating partial failure.
    ok, failed = [r for r in results if r.ok], [r for r in results if not r.ok]
    return await llm.acomplete(model="powerful-model",
        system="Synthesize the worker results into one coherent answer. "
               "Note explicitly anything that could not be completed.",
        messages=[{"role": "user", "content": json.dumps(
            {"task": task, "results": [r.model_dump() for r in ok],
             "failed": [r.model_dump() for r in failed]})}])
```

**LangGraph form** (sub-graphs — Section 14.7.4): `plan` node → `dispatch` node using `Send()` to spawn one worker sub-graph invocation per subtask → `synthesize` node joining all workers.

### 8.6 Orchestrator-Workers vs Hierarchical Multi-Agent

The two concepts are not the same axis:

| | **Orchestrator-Workers** | **Hierarchical Multi-Agent** |
|---|---|---|
| What is structured | **The task** — LLM decomposition into subtasks | **The authority** — who directs whom |
| Role of central node | Planner/coordinator of work | Supervisor/manager of agents |
| Relationship | Task assignments (work → output) | Authority (directive → accountability) |
| Workers | LLM calls, workflows, *or* agents | Agents by definition |
| See also | This section | [Hierarchical Multi-Agent Frameworks](ai_llm/hierarchical_multi_agent_frameworks_guide.md) |

A supervisor hierarchy is a *governance structure*; orchestrator-workers is a *task-decomposition pattern*. A flat peer group of agents is neither; a single orchestrator dispatching plain LLM workers is orchestrator-workers without any agent hierarchy. Banking systems typically pair them: orchestrator-workers for the work shape, a hierarchy for accountability and approvals.

---

## 9. Pattern 5 — Evaluator-Optimizer

### 9.1 The Pattern

**Evaluator-optimizer** pairs a **generator** LLM with an **evaluator** LLM and iterates: generate → evaluate → (if failing) feed critique back → regenerate — until the output passes or the iteration budget is exhausted.

```
   ┌──────────┐  draft   ┌────────────┐  score + reasons
   │GENERATOR │ ───────► │ EVALUATOR  │ ───────────────┐
   │   (LLM)  │ ◄─────── │    (LLM)   │ revised draft  │
   └──────────┘ (critique)└────────────┘               │
        ▲  loop while score < threshold and            │
        └─────────────── iters < max ──────────────────┘
```

The evaluator is a **separate** LLM call with an explicit rubric — not the generator grading its own homework. That separation is the pattern's entire value.

### 9.2 Canonical Examples

- **Writing**: draft → critique (structure, clarity, tone, factual accuracy) → revise until pass; **code**: generate → review (correctness, security, style) → fix → re-review.
- **Extraction**: extract → validate against ground-truth rules → re-extract with validation errors as feedback — beats one-shot extraction on messy documents.
- **RAG answer quality**: generate → evaluate groundedness/completeness → regenerate if ungrounded (self-RAG style; see [Advanced RAG Techniques](ai_llm/advanced_rag_techniques_guide.md)).
- **Translation**: draft → evaluate fidelity/fluency → refine.

### 9.3 When to Use

- **Clear evaluation criteria exist** — you can write an explicit rubric (dimensions, thresholds). Without one, the evaluator is just another opinion and the loop can oscillate (Section 16.5).
- **Iterative improvement helps** — feedback measurably raises quality (writing, code, extraction), not a task where the first pass is already optimal.
- **The quality bar is high** — N iterations' cost is justified (regulatory-grade documents, client-facing copy, production code).

### 9.4 Implementation Guidance

- **Generator and evaluator are separate prompts** — the evaluator gets the rubric, the draft, and possibly source material; not the generator's chain-of-thought.
- **Structured evaluator output** — score + reasons (+ specific fix instructions). The score must be parseable (JSON) for threshold comparison; the reasons feed the next generator call.
- **Max iterations with a hard cap** — 2–4 is typical; beyond, accept best-so-far and escalate. An unbounded evaluator loop is an infinite-loop bug (Section 16.5).
- **Pass criteria defined before the loop** — threshold (e.g., ≥8/10 on all dimensions), stopping rules (pass, max-iters, diminishing returns: stop if improvement < ε).
- **Log the iteration history** — each draft's scores are audit gold ("revised 3 times; scores recorded").

### 9.5 Code Sketch

```python
class Evaluation(BaseModel):
    score: float; passes: bool; reasons: list[str]

def evaluate(draft: str, rubric: str) -> Evaluation:
    out = llm.complete(model="powerful-model",
        system=f"Evaluate against this rubric:\n{rubric}\n"
               'Return JSON: {"score": 0-10, "passes": bool, "reasons": []}',
        messages=[{"role": "user", "content": draft}],
        response_format=Evaluation)
    return Evaluation.model_validate_json(out)
def generate(subject: str, critique: str | None, rubric: str) -> str:
    msgs = [{"role": "system", "content": "Write the document. Improve on "
            "the critique if provided."},
            {"role": "user", "content": subject}]
    if critique:
        msgs.append({"role": "user", "content": f"Critique to address:\n{critique}"})
    msgs.append({"role": "user", "content": f"Rubric:\n{rubric}"})
    return llm.complete(model="powerful-model", messages=msgs)
def evaluator_optimizer(subject: str, rubric: str,
                        threshold: float = 8.0, max_iters: int = 3) -> str:
    draft, best, best_score = generate(subject, None, rubric), None, -1.0
    for _ in range(max_iters):
        ev = evaluate(draft, rubric)
        if ev.score > best_score: best, best_score = draft, ev.score
        if ev.passes or ev.score >= threshold: return draft
        draft = generate(subject, "\n".join(ev.reasons), rubric)
    return best                                    # budget exhausted → best effort
```

**LangGraph form** (loop with condition — Section 14.7.5): `generate → evaluate`; a conditional edge returns to `generate` while score < threshold and iters < max, else `END`.

---

## 10. The Agent Pattern

### 10.1 The Agent Loop

An agent is **an LLM with tools, running in a loop** — the atomic unit of agency:

```
  ┌─────────────────────────────────────────────────────────┐
  │ PERCEIVE ──► REASON ──► ACT ──► OBSERVE ──► (repeat)    │
  │ (context,   (LLM:    (tool    (tool    until the model  │
  │  tool       tool     call or  result   decides "done")  │
  │  results)   call or  final    feeds back                │
  │             answer)  answer)                            │
  └─────────────────────────────────────────────────────────┘
```

- **Perceive** (task + conversation + tool results) → **Reason** (LLM: a tool call — which tool, what arguments — or the final answer) → **Act** (runtime executes the tool: API, search, code interpreter, DB, filesystem) → **Observe** (result appended to context) → repeat.

The loop terminates on the model's final answer — or when a guardrail terminates it (max iterations, budget, human interrupt; Section 10.6).

### 10.2 The Canonical Agent Implementations

Three loop architectures dominate; they differ in *when* and *how* the model plans.

#### ReAct: Reason + Act, Interleaved

**ReAct** (Yao et al., 2022) interleaves thinking and acting: **think → act → observe → think → ...** Each step produces a short reasoning trace plus one action; the observation feeds the next thought.

```
Thought: I need the customer's recent transactions to check for anomalies.
Action: query_transactions(account_id="...")
Observation: [12 rows, 3 flagged] → Thought: matches the fraud typology
Action: search_risk_rules(typology="unusual_velocity") → Observation: ...
Thought: Sufficient evidence. Final Answer: ...
```

ReAct is the default for **tool-using agents**: simple, observable (the reasoning trace *is* the audit log), and works with any tool set. Agentic RAG builds on it — retrieve, reason over results, retrieve more, answer (see [Beyond RAG](ai_llm/beyond_rag_guide.md) and [Advanced RAG Techniques](ai_llm/advanced_rag_techniques_guide.md) on agentic RAG).

#### Plan-and-Execute: Plan First, Then Execute

**Plan-and-execute** separates planning from doing: the model first produces a **plan** (list of steps), executes the steps (possibly with tools), and **re-plans when execution fails or the situation changes**.

```
Plan: 1) Identify counterparties. 2) Retrieve trade records. 3) Check margin calls. 4) Draft notice.
Step 2 fails (records not in ledger A) → REPLAN → 2) Query ledger B instead ...
```

For tasks with **long horizons and many steps**, where interleaved reasoning (ReAct) drifts or burns tokens re-deriving the plan each step. The plan is a first-class artifact — reviewable by a human before execution, versionable, auditable. Cost: the plan can be wrong, so **replanning on failure is not optional** — it is the pattern's error handling.

#### Reflection: Generate, Critique, Revise

**Reflection** is the single-agent version of evaluator-optimizer: the *same* model generates, critiques its own output, revises. Self-RAG makes it retrieval-aware: retrieve, reflect on whether the passages support the answer, regenerate when not (see [Advanced RAG Techniques](ai_llm/advanced_rag_techniques_guide.md) on self-RAG).

```
Generate: draft the credit memo... Critique: omits the covenant breach discussion; tone too assertive...
Revise: incorporate the covenant analysis, soften tone...
```

Reflection is cheap (no second model) and effective where a **clear rubric exists** — but the model critiques with the same blind spots it generated with; where stakes justify it, prefer the two-model evaluator-optimizer (Section 9).

**Choosing among them:** ReAct (think → act → observe, interleaved) for tool-using tasks with unknown paths — watch for token drift on long tasks; plan-and-execute for long multi-step tasks — watch for stale plans; reflection for drafting against a rubric — but it critiques with the same blind spots it generated with.

### 10.3 Tool Use: The Agent's Core Capability

Tools turn an LLM from a text generator into an agent — the loop is meaningless without an environment to act on. Two integration mechanisms matter:

- **Function calling** (native tool use): the model emits structured tool calls; the runtime executes them and returns results. Schema quality is the whole game — precise tool descriptions, tight argument schemas, constrained decoding on the tool-call JSON (see [Constrained Decoding Frameworks](constrained_decoding_frameworks_guide.md)). This is the substrate for ReAct in all major SDKs.
- **MCP** (Model Context Protocol): a standard for exposing tool servers to agents — one agent, many tool servers, no bespoke integrations. In banking, MCP is how an agent gains read-only access to reference/market data and internal systems under centrally managed permissions (see [MCP Framework & Tools](ai_llm/mcp_framework_tools_guide.md)).

Tool design rules: **(1)** narrow tools, not mega-tools — `get_account_balance(id)` is debuggable, `do_everything(json)` is a footgun; **(2)** read-only by default — write/money tools gated behind human approval (Section 10.6); **(3)** every tool has a contract — inputs, outputs, failure modes, permissions — and validate tool-call JSON before execution; **(4)** tool results are data — validate/sanitize before they re-enter context (tool results are a prompt-injection vector; [LLM Development Risks & Security](llm_development_risks_security_guide.md)).

### 10.4 When to Use Full Agents vs Workflow Patterns

Use an **agent** when the task cannot be pre-decomposed — subtasks, order, or tools depend on what the model discovers (research, incident investigation, novel document analysis). Use a **workflow pattern** whenever the task *can* be pre-decomposed, even if complex — a 30-step fixed process is still a workflow, not an agent. The boundary is **who owns the control flow**: if you can draw the process diagram before the run, it's a workflow; if it can only be drawn *after* the run (from the transcript), it's an agent.

### 10.5 The Trade-Offs (Why Agents Are Expensive)

Every agent buys flexibility with four currencies: **unpredictability** (may loop, choose wrong tools, take unplanned paths → guardrails: iteration caps, timeouts, budgets, tool allowlists, output contracts — Section 16.7; [LLM Development Risks & Security](llm_development_risks_security_guide.md) on excessive agency); **cost** (each iteration is a full LLM call; a 10-step agent ≈ 10× a single prompt → iteration budgets, model tiering, caching — [On-Prem LLM Deployment](on_prem_llm_deployment_guide.md), [Agent Runtime Cache Design](ai_llm/agent_runtime_cache_design_guide.md)); **latency** (loop count × step latency; hard to quote an SLA → concurrency, early-exit criteria, hybrid skeleton — Section 11); **debugging difficulty** (failures are emergent; you debug a *transcript*, not a code path → full logging + tracing — Section 16.10). Plus the **compliance burden**: agent decisions are hard to audit — the path exists only in the run record, so full decision logging, HITL for consequential actions, and SR 11-7 model documentation are mandatory (Section 15.7). In regulated environments that last item is often decisive: **an agent that cannot explain its path is a model-risk finding waiting to happen.**

### 10.6 Guardrails (Non-Negotiable for Production Agents)

- **Iteration and budget caps** — max loop count (e.g., 25) and max tokens/cost per run; escalate to a human on exhaustion.
- **Tool allowlist + output contract** — only registered, permissioned tools (blocked tools return a denial the agent must handle); the final answer must match a schema before the loop ends.
- **Human-in-the-loop for consequential actions** — money movement, external communications, data deletion, high-value approvals: the agent *proposes*, a human *disposes* (Section 15.6; [Durable AI Agent Workflows](durable_ai_agent_workflows_guide.md) on HITL as a durability primitive).

---

## 11. Hybrid Approaches: Workflows + Agents

The dichotomy is a *design axis*, not a constraint. The most common production architectures in 2026 are hybrids — use the simplest structure per *step*, not per *system*.

### 11.1 Workflows with Agentic Steps

A **fixed pipeline** in which **one step is an agent** — the open-ended step gets a model-directed loop; everything around it stays deterministic:

```
  Input ──► [Router] ──► [Chain: normalize + enrich] ──► [AGENT STEP: investigate] ──► [Validate + format]
            (deterministic)                            (open-ended tool loop)         (deterministic)
```

Banking example: client-complaint handling — parse and route the complaint (fixed), enrich with account data (fixed), then an **agent investigates** (open-ended: decides which systems to query, which documents to read, how to corroborate), then the workflow validates the findings and drafts the response (fixed). The agent is sandboxed between deterministic gates.

### 11.2 Agents with Structured Phases

An **agent loop that follows a fixed skeleton** — free *within* each phase, but the phases are fixed: **plan phase (agent) → execution phase (workflow) → review phase (agent)**.

```
  ┌──────────────┐    ┌──────────────────┐    ┌──────────────┐
  │ PLAN (agent) │───►│ EXECUTE          │───►│ REVIEW (agent│
  │ drafts plan; │    │ (workflow: fixed │    │ checks output│
  │ rubric/human │    │  steps + tools)  │    │ vs reqs;     │
  │ approves     │    │                  │    │ routes back  │
  └──────────────┘    └──────────────────┘    └──────────────┘
```

This is plan-and-execute (Section 10.2) made *structural*: plan is an agentic decision, execution is a deterministic workflow (testable, observable), review is an agentic judgment. Replanning happens at phase boundaries, not mid-step — execution stays auditable step-by-step while preserving adaptivity.

**Why hybrids win in practice:** the pipeline stays testable (deterministic segments unit-test; only agentic segments need transcript-based testing); cost lands where it pays (the expensive loop runs only on the open-ended slice); audit tells a clean story ("the process is fixed; within step 4 the system investigated adaptively — here is its logged reasoning"); failures are contained (a misbehaving agent is caught by surrounding gates). **Hybrid decision rule: write the workflow skeleton first; carve agentic holes only where the skeleton cannot be specified.**

---

## 12. The Pattern Selection Framework

### 12.1 The Five Questions

Walk the questions in order; each answer either selects a pattern or moves you down:

**Q1. Is the task well-defined with known steps?** → Yes: **workflow territory** (Q2–Q3). No: proceed to Q4/Q5. *"Well-defined" = you can enumerate the steps before the run — if you can, you do not need an agent.*

**Q2. Does the task have distinct categories needing different handling?** → Yes: **routing** (Section 6) — a classifier dispatches to specialized handlers; composes with everything (route, then chain within each branch).

**Q3. Does the task decompose into a fixed dependent sequence?** → Yes: **prompt chaining** (Section 5); if subtasks are independent, use **parallelization** (Section 7) instead.

**Q4. Can the task be parallelized?** → Yes: **parallelization** (Section 7) — sectioning for speed, voting for variance reduction; if subtasks are *unknown until runtime* but parallelizable and need synthesis: **orchestrator-workers** (Section 8).

**Q5. Is there a clear quality bar with iterative improvement?** → Yes: **evaluator-optimizer** (Section 9) — generate, evaluate against an explicit rubric, revise until pass or budget.

**Q6. Is the task open-ended / adaptive, with unknown steps and tool selection?** → Yes: **agent** (Section 10) — the model directs the loop. The *last resort*, not the first instinct.

### 12.2 The Decision Tree (ASCII)

```
 START: describe the task to yourself. Can you enumerate its steps?
 │
 ├─ YES ─────────────────────────────────────────────────────────────┐
 │   Q: Single competency (one LLM call)?
 │   ├─ YES → Single prompt (maybe + one tool). DONE.
 │   └─ NO  → Q: Distinct categories with different handling?
 │           ├─ YES → ROUTING (then handle each branch) ────────────┐
 │           └─ NO  → Q: Dependent sequence?                        │
 │                   ├─ YES → PROMPT CHAINING                       │
 │                   └─ NO  → Q: Independent subtasks?              │
 │                           ├─ YES → Q: Subtask set known?         │
 │                           │      ├─ YES → PARALLELIZATION        │
 │                           │      └─ NO  → ORCHESTRATOR-WORKERS   │
 │                           └─ NO  → Q: Clear rubric + high bar? ─ YES → EVALUATOR-OPTIMIZER
 │                                    └ NO → (re-examine the task; missing step def.)
 ├─ NO (steps unknown / open-ended) ─────────────────────────────────┤
 │   Q: Can you bound it with a fixed skeleton?
 │   ├─ YES → HYBRID: fixed phases with agentic phases
 │   │        (plan agent → execute workflow → review agent)
 │   └─ NO  → FULL AGENT (ReAct / plan-and-execute loop)
 │            + mandatory guardrails (budgets, allowlists, HITL)
 └───────────────────────────────────────────────────────────────────┘
```

### 12.3 Pattern Characteristics Table

| Pattern | Predictability | Flexibility | Relative cost | Latency | Complexity | Best for |
|---|---|---|---|---|---|---|
| Prompt chaining | High | Low | Low–mid (Σ steps) | Σ steps (sequential) | Low | Fixed dependent sequences; per-step model tiering |
| Routing | High (structure); classifier-dependent | Low–mid | Low (cheap classifier) | Classifier + 1 handler | Low | Category specialization; cost optimization |
| Parallelization | High | Low | N × step cost | Slowest worker (≈1×) | Mid | Independent subtasks; speed; voting |
| Orchestrator-workers | Mid (decomposition varies) | Mid–high | Plan + N workers + synthesis | Plan + workers + synth | Mid–high | Unknown decomposition; parallel subtasks + synthesis |
| Evaluator-optimizer | Mid (loop count varies) | Mid | Iterations × 2 calls | Iterations × 2 calls | Mid | High quality bar with explicit rubric |
| Agent loop | Low | High | Iterations × calls (unbounded) | Iterations (unbounded) | High | Open-ended, adaptive, tool-selecting tasks |
| Hybrid (Section 11) | High where fixed, low where agentic | High | Bounded by skeleton | Bounded by skeleton | High | Production systems with defined + open-ended parts |

**Reading the table:** workflow cost/latency are *bounded expressions*; agent cost/latency are *unbounded variables*. That single difference drives most architecture decisions in regulated industries.

---

## 13. Combining Patterns: Real System Architectures

Production systems are **compositions** — patterns nest like design patterns: routing at the entrance, chaining for fixed sequences, parallelization for fan-out, evaluator loops at quality gates, agents in open-ended holes.

### 13.1 Customer Support Bot

```
 Request ──► [ROUTING] classify: billing / technical / fraud / general
                  │
     ┌────────────┼──────────────────┐
     ▼            ▼                  ▼
 [Billing]   [Technical]      [Fraud] / [General]
 CHAINING    AGENT (tool      CHAINING + HITL
 identify→   loop over        detect → verify →
 explain→    knowledge)       draft → escalate
 resolve
     └────────────┼──────────────────┘
                  ▼
          [EVALUATOR-OPTIMIZER] quality gate (accurate, compliant,
          empathetic? revise if not) → reply
```

Patterns: **routing → chaining / agent → evaluator-optimizer**. The router sends each request to the right machinery; the evaluator gate polices quality before the reply ships. This exact shape is the baseline for bank support bots (Section 15.3).

### 13.2 Research Assistant

```
 Question ──► [ROUTING] fact lookup / analysis / synthesis
                  ▼
          [ORCHESTRATOR-WORKERS] plan sub-questions; parallel researcher
          workers (mini agent loops); orchestrator synthesizes + cites
                  ▼
          [EVALUATOR-OPTIMIZER] groundedness/completeness check;
          regenerate until grounded (self-RAG style)
                  ▼
           Final answer + sources
```

Patterns: **routing → orchestrator-workers → evaluator-optimizer**. The research discipline — question decomposition, evidence, citations — is in [Research Agents](research_agents_guide.md); the *shape* here is the composition.

### 13.3 Coding Agent

```
 Feature request ──► [AGENT LOOP (planner)] plan files/modules/tests
                          ▼
                  [ORCHESTRATOR-WORKERS] file/module tasks;
                  worker agents implement in parallel
                          ▼
                  [EVALUATOR-OPTIMIZER] code review loop until pass
                          ▼
                  [AGENT LOOP (integrator)] integrate, run tests, fix
```

Patterns: **agent loop + orchestrator-workers + evaluator-optimizer**. The outer loop owns the mission; orchestrator-workers parallelizes implementation; the evaluator gate polices quality (see [Coding Agents](ai_llm/coding_agents_research.md)).

### 13.4 Composition Rules of Thumb

1. **Route at the entrance** — classification is cheap and simplifies everything downstream.
2. **Deterministic core, agentic edges** — fixed chains carry the bulk; agents only at genuinely open-ended steps.
3. **Quality gates at handoffs** — every subsystem boundary is a validation point (schema, rubric, human).
4. **One pattern per concern** — decomposition + iteration + routing is three nested components, not one confused one.

---

## 14. Workflow-as-Code: Implementing Patterns with Frameworks

The patterns are framework-agnostic shapes; frameworks are their mechanical realization.

### 14.1 The Framework Landscape

| Framework | Model | Sweet spot | Pattern fit |
|---|---|---|---|
| **LangGraph** | StateGraph: nodes + edges + shared state; conditional edges, Send/parallel branches, sub-graphs, checkpointing | Complex graphs; the workflow/agent framework | All 5 patterns + agent loop natively expressible |
| **LlamaIndex Workflows** | `Workflow` class: steps (`@step`) communicating via typed events | Data/RAG pipelines; event-driven composition | Chaining, routing, parallelization, evaluator loops; agent loop via `AgentWorkflow` |
| **Haystack** | Pipelines: components connected in a DAG | Retrieval-augmented pipelines; enterprise integration | Chaining, routing, parallelization; agent loop via tool components |
| **OpenAI Agents SDK / Claude Agent SDK** | Agent objects with tools + handoffs; built-in agent loop | Single- and multi-agent loops with minimal ceremony | Agent loop; handoffs ≈ routing between agents; guardrails built in |
| **Temporal / Inngest / Restate** | Durable execution: deterministic workflow code, replayed on failure | Long-running, stateful, crash-safe processes | Any pattern *made durable* (child workflows = orchestrator-workers; signals = HITL) — see [Durable AI Agent Workflows](durable_ai_agent_workflows_guide.md) |

**Choosing:** LangGraph or LlamaIndex when the *shape* of the process is the product (graphs, branching, loops, state); Agent SDKs when the *agent loop* is the product and you want maximum leverage on tool use and handoffs; durable engines when *reliability* is the product. The two compose: a durable engine can host LangGraph/LlamaIndex logic as child workflows (Section 14.5).

### 14.2 Pattern-to-Framework Mapping

| Pattern | LangGraph | LlamaIndex | Haystack | Agent SDKs | Durable engine |
|---|---|---|---|---|---|
| Prompt chaining | Linear graph (edges) | Chained `@step` handlers | `Pipeline.add` in sequence | Sequential tool calls | Sequential activities |
| Routing | Conditional edges | Router step (event-based) | Conditional path/router | Handoffs / guardrails | Branch on activity result |
| Parallelization | Parallel branches + join | `asyncio.gather` in a step | Pipeline parallel branches | Concurrent runs | Fan-out/fan-in activities |
| Orchestrator-workers | Orchestrator node + `Send()` to sub-graphs | Dynamic `Workflow` spawning | Pipeline per worker | Handoffs to worker agents | Child workflows (native) |
| Evaluator-optimizer | Loop edge with condition | Loop step + state | Loop component | Guardrails / custom loop | Loop with retry policy |
| Agent loop | ReAct node + tool node (checkpointed) | `AgentWorkflow` | Agent components | **Native** (built-in loop) | Agent as durable activity |

### 14.3 LangGraph Sketches for the Canonical Patterns

LangGraph's mental model: nodes = functions, edges = control flow, shared state dict. The same shapes map to LlamaIndex/Haystack with their syntax.

#### 14.3.1 Prompt Chain — Linear Graph

```python
class State(TypedDict):
    doc: str; extracted: dict; translated: dict; formatted: str

def extract(s):  s["extracted"]  = extract_step(s["doc"])        # validated
def translate(s): s["translated"] = translate_step(s["extracted"])
def format_(s):  s["formatted"] = format_step(s["translated"])
g = StateGraph(State)
g.add_node("extract", extract); g.add_node("translate", translate)
g.add_node("format", format_)
g.set_entry_point("extract")
g.add_edge("extract", "translate"); g.add_edge("translate", "format")
g.add_edge("format", END)
```

#### 14.3.2 Router — Conditional Edges

```python
def classify(s): s["category"] = classify_input(s["input"])   # constrained output
def route_by_category(s): return s["category"]                # key of the map below
g = StateGraph(State)
g.add_node("classify", classify)
g.add_node("billing", billing_handler); g.add_node("tech", tech_handler)
g.add_node("fraud", fraud_handler);     g.add_node("general", general_handler)
g.set_entry_point("classify")
g.add_conditional_edges("classify", route_by_category, {
    "billing": "billing", "technical": "tech",
    "fraud": "fraud", "general": "general",   # fallback included in the table
})
```

#### 14.3.3 Parallelization — Branches + Join

```python
g = StateGraph(State)
g.add_node("section_a", write_a); g.add_node("section_b", write_b)
g.add_node("merge", merge_and_polish)
g.set_entry_point("section_a")
g.add_edge("section_a", "merge")   # join semantics: merge fires only after
g.add_edge("section_b", "merge")   # BOTH branches complete
g.add_edge("merge", END)
```

#### 14.3.4 Orchestrator-Workers — Orchestrator Node + Sub-Graphs

```python
from langgraph.types import Send

def dispatch(s):
    return [Send("worker", {"subtask": t}) for t in s["plan"]["subtasks"]]
g = StateGraph(State)
g.add_node("plan", orchestrator_plan)         # LLM: subtask list
g.add_node("dispatch", dispatch)              # Send() per subtask
g.add_node("worker", worker_subgraph)         # worker = another compiled graph
g.add_node("synthesize", orchestrator_synthesize)
g.set_entry_point("plan")
g.add_edge("plan", "dispatch"); g.add_edge("dispatch", "synthesize")
g.add_edge("synthesize", END)
```

#### 14.3.5 Evaluator-Optimizer — Loop with Condition

```python
def generate(s): s["draft"] = generator(s["topic"], s.get("critique"))
def evaluate(s):  s["eval"]  = evaluator(s["draft"])   # {score, passes, reasons}
def should_continue(s):
    if s["eval"]["passes"] or s["iters"] >= s["max_iters"]: return "accept"
    s["critique"] = s["eval"]["reasons"]; s["iters"] += 1; return "revise"
g = StateGraph(State)
g.add_node("generate", generate); g.add_node("evaluate", evaluate)
g.set_entry_point("generate"); g.add_edge("generate", "evaluate")
g.add_conditional_edges("evaluate", should_continue,
                        {"revise": "generate", "accept": END})
```

#### 14.3.6 Agent — ReAct Loop with Tool Node

```python
g = StateGraph(State)
g.add_node("reason", llm_reason)          # model: tool call or final answer
g.add_node("tools", execute_tool)         # validated tool execution
g.set_entry_point("reason")
g.add_conditional_edges("reason", decide_next, {"call_tool": "tools", "finish": END})
g.add_edge("tools", "reason")             # observe → reason → ...
# Checkpointed: the LangGraph checkpointer makes this loop resumable/durable.
```

### 14.4 Durability: Run Patterns on an Engine

Workflow-as-code becomes *production* workflow-as-code when the pattern runs on a **durable engine** (Temporal, Inngest, Restate, Step Functions, Azure Durable Functions), which gives every pattern crash-recovery (replay from last completed step), exactly-once side effects, timers/waiting (HITL sleeps), and child workflows (orchestrator-workers fan-out that survives redeploys). Mapping: chaining → sequential activities; routing → branch on activity result; parallelization → fan-out/fan-in activities; orchestrator-workers → **child workflows**; evaluator-optimizer → loop with retry policy + iteration cap; agent loop → agent as durable activity, HITL via signals. See [Durable AI Agent Workflows](durable_ai_agent_workflows_guide.md) for retry/backoff math, idempotency keys, state-vs-memory separation, and HITL as a durability primitive. **Rule of thumb: any workflow running > a few minutes, touching production state, or spending real money per run belongs on a durable engine.**

---

## 15. Banking Applications

Banks are the canonical *workflow-first* industry: processes are defined, regulated, and audited; predictability and explainability are requirements, not preferences (MAS guidelines, EU AI Act, SR 11-7 — see [Financial Risk & Compliance Systems](../banking/financial_risk_compliance_systems_guide.md) and [LLM Development Risks & Security](llm_development_risks_security_guide.md)).

### 15.1 Account Opening

**Patterns: chaining + routing (risk-tiered).**

```
 Application ──► [KYC check] ──► [Credit check] ──► [Compliance screen] ──► [Account creation]
                    │                │                  │
                    └── routing: risk tier ── low → fast lane (cheap model)
                                          ├─ standard → full checks
                                          └─ EDD → enhanced workflow (agentic
                                                    investigation + human approval)
```

Every step is a fixed, documented sub-process (the bank's procedure as a chain); routing sends each applicant down the risk-tiered path. The EDD branch is where an *agentic step* legitimately appears — open-ended investigation of the applicant's profile, sanctions lookups, narrative building — sandboxed between deterministic gates (Section 11.1).

### 15.2 Loan Origination

**Patterns: chaining + routing + evaluator-optimizer + HITL.**

```
 Application ──► [Eligibility routing] ──► (auto-decline / full review)
                              │
                              ▼
                  [Credit assessment: EVALUATOR-OPTIMIZER]
                  underwriting draft → credit-risk evaluation (covenants,
                  DSCR, collateral) → revise until rubric passes (or escalate)
                              │
                              ▼
                  [Approval: HUMAN-IN-THE-LOOP] agent recommends;
                  approver decides (sign-off recorded, auditable)
```

The credit memo is drafted by a generator and scored by an evaluator against the bank's credit rubric — the iteration history (draft 1: 6/10, missing covenant analysis → draft 2: 8/10 → pass) is itself audit evidence. Final approval is always human (HITL as a durability primitive; [Durable AI Agent Workflows](durable_ai_agent_workflows_guide.md)).

### 15.3 Customer Support

**Patterns: routing → specialized workflows/agents → evaluator quality gate.** Section 13.1's reference architecture with banking categories: **billing** (chained: identify → explain → resolve, deterministic), **fraud** (chained workflow + escalation with HITL on refunds), **technical** (agent loop over diagnostics tools), **general** (fallback to human). The evaluator gate checks every drafted reply for accuracy *and* regulatory tone before it ships — misstatements in support replies are compliance incidents.

### 15.4 Compliance Monitoring

**Patterns: chaining (screen → evaluate → report).**

```
 Transactions / events ──► [Screen] ──► [Evaluate] ──► [Report / Alert] ──► (case mgmt)
                          (rules + LLM   (LLM risk     (fixed template,
                           classification scoring vs    routed by risk)
                           fixed pipeline) rubric)
```

The screen→evaluate→report chain is deterministic by design: screening decisions must be reproducible and defensible — a screening result that changes run-to-run is a regulatory liability. See [Financial Risk & Compliance Systems](../banking/financial_risk_compliance_systems_guide.md) for the surrounding control framework.

### 15.5 Trade Processing

**Patterns: chaining (validate → enrich → route → settle).**

```
 Trade ──► [Validate] ──► [Enrich] ──► [Route] ──► [Settle]
          (schema +        (LLM: fill    (by product/   (fixed settlement
           counterparty    missing       counterparty   steps, sagas)
           checks)         fields, flag  — fixed table)
                           anomalies)
```

Each stage is a bounded, observable step; LLMs contribute at the enrich stage (extracting/normalizing trade details) under strict schema constraints. Routing is a deterministic table, not a model decision — settlement paths are business rules. See [Payments Hub](../banking/payments_hub_guide.md) for the hub-and-saga architecture.

### 15.6 The Banking Pattern Preference

- **Workflows first** — predictable, auditable, cheaper, and the *regulatory preference*: a defined, documented process is reviewable; an emergent one is not.
- **Agents only where needed** — genuine open-endedness: investigation, research, unstructured analysis (see [Research Agents](research_agents_guide.md)). Every agent must justify its unpredictability budget.
- **HITL for consequential actions** — approvals, payments, external communications, data changes: the LLM proposes, a named human disposes, recorded (see [LLM Development Risks & Security](llm_development_risks_security_guide.md) on excessive agency; [Durable AI Agent Workflows](durable_ai_agent_workflows_guide.md) on HITL).

### 15.7 Auditability: The Decisive Criterion

- **Workflows are audit-friendly by construction**: the process definition *is* the documentation — the step list maps 1:1 to the bank's written procedure; a run's log shows which steps ran, in what order, with what inputs/outputs; replay shows the same result. SR 11-7-style model documentation is straightforward for workflow steps: prompt, schema, validation rules are reviewable artifacts.
- **Agents require full logging of reasoning and tool calls**: every loop iteration — stated reasoning, tool called, arguments, raw result — must be captured, retained (MAS/GDPR record-keeping horizons), and reproducible in audit. An agent whose path cannot be reconstructed from logs fails model-risk review.

**The banking decision rule:** *use workflows for anything with a defined process — which in a bank is almost everything; use agents only for genuine open-endedness, wrapped in guardrails, gates, and full logging.* When in doubt, the workflow is the compliant answer; the hybrid skeleton (Section 11) keeps the open-ended parts without surrendering the defined parts.

---

## 16. Anti-Patterns and Pitfalls

### 16.1 Using Agents Where Workflows Suffice — the #1 Mistake

The most expensive error in the field: an open-ended-looking task that is actually a fixed process gets an agent — incurring unnecessary cost (loop iterations × tokens), unpredictability (behavior varies per run), and audit burden (an emergent path to document). **Fix:** run Section 12's questions before building; if you can enumerate the steps, chain them.

### 16.2 Over-Orchestration (BPMN Hell)

The mirror image: a workflow so fine-grained that orchestration overhead dwarfs the work — 40 nodes for a task with 4 real steps, a graph nobody can trace (the [Payments Hub](../banking/payments_hub_guide.md) calls this out for integration design; it applies equally to LLM workflows). **Fix:** keep the graph at the granularity of *business steps*; fold micro-steps into their parent step; if the graph needs a legend, it is too big.

### 16.3 Workflow Without Validation Gates

Chains that pass raw LLM output downstream let garbage flow through: malformed JSON from an extractor, hallucination over it downstream, confidently wrong final output. **Fix:** every step emits structured output (schema-enforced; [Constrained Decoding Frameworks](constrained_decoding_frameworks_guide.md)) and every handoff validates — schema, business rules, sanity bounds — with retry or fallback on failure.

### 16.4 Routing Without Fallback

A router with no default handler turns out-of-distribution inputs into crashes or silent misfires. **Fix:** the routing table always includes a fallback handler; the classifier emits a confidence score with a below-threshold → fallback rule (Section 6.4).

### 16.5 Evaluator Without Pass Criteria

An evaluator loop with no threshold or stopping rule oscillates forever — or terminates on vibes. **Fix:** define the rubric, threshold, max iterations, and a diminishing-returns rule before the loop; cap at 2–4 iterations and accept best-so-far (Section 9.4).

### 16.6 Parallelization of Dependent Tasks

Fanning out tasks that secretly share state or ordering constraints produces wrong results merged confidently (two sections that each recomputed a number differently; two workers writing the same file). **Fix:** verify independence before parallelizing (shared reads fine, shared writes not) and add consistency checks in the merge step (Section 7.4).

### 16.7 Agent Without Guardrails

An agent with no iteration cap, no tool allowlist, and no budget is a cost and safety incident waiting to happen — loops, tool abuse, runaway spend, unplanned side effects: the "excessive agency" risk class in [LLM Development Risks & Security](llm_development_risks_security_guide.md). **Fix:** iteration caps, token/cost budgets, tool allowlists, output contracts, HITL for consequential actions (Section 10.6) — non-negotiable in production, mandatory in banking.

### 16.8 No Human Review for Consequential Actions

An agent that moves money, sends external mail, or deletes data without a human gate is a regulatory and reputational liability, however good the model. **Fix:** consequential actions sit behind HITL; the agent proposes, a named human approves, the decision is recorded ([Durable AI Agent Workflows](durable_ai_agent_workflows_guide.md) HITL pattern).

### 16.9 Ignoring Cost/Latency Accumulation

Each step adds tokens and latency — a 6-step chain with 2 evaluator iterations and a 4-way fan-out is 12+ LLM calls per run; at scale that is a real budget line, and sequential latency may break the SLA. **Fix:** budget the pipeline end-to-end before building — per-step token estimates, model tiering (cheap model per mechanical step), caching deterministic parts ([On-Prem LLM Deployment](on_prem_llm_deployment_guide.md), [Agent Runtime Cache Design](ai_llm/agent_runtime_cache_design_guide.md)), parallelism where independent, iteration caps on loops.

### 16.10 No Observability

An unobserved pipeline cannot be debugged: "the workflow failed" with no indication of which step, which model call, or why. **Fix:** trace every step — LLM call, tokens, latency, input/output hashes, validation outcomes — with per-step spans and run-level correlation (see [RAG Frameworks Comparison](ai_llm/rag_frameworks_comparison_guide.md) on LLM tracing). In banking, observability is also the audit trail (Section 15.7).

### 16.11 Pitfall Checklist

| Pitfall | Symptom | Prevention |
|---|---|---|
| Agents where workflows suffice | Cost blowup, flaky behavior, audit pain | Section 12 decision tree |
| Over-orchestration | Unreadable graphs, orchestration overhead | Business-step granularity |
| No validation gates | Confidently wrong output | Structured output + schema checks per handoff |
| No routing fallback | Unknown inputs crash/misfire | Fallback handler + confidence threshold |
| Evaluator without pass criteria | Infinite loops / vibe termination | Rubric + threshold + caps defined upfront |
| Parallelizing dependent tasks | Contradictory merged output | Independence check + merge consistency checks |
| Unguarded agents | Loops, tool abuse, runaway spend | Caps, allowlists, budgets, HITL |
| No HITL on consequential actions | Regulatory/compliance incidents | Human gate on money/mail/data actions |
| Ignored cost/latency accumulation | Budget overruns, SLA misses | Pipeline budget + tiering + caching |
| No observability | Undebuggable failures | Per-step tracing, run correlation |

---

## 17. The Future: Agentic Workflows in 2026 and Beyond

### 17.1 Agentic Workflows Standardize

The catalog is consolidating into **standard vocabulary and tooling**: framework-native pattern primitives (LangGraph pattern templates, LlamaIndex workflow libraries), published pattern libraries, and named compositions (Section 13's reference architectures become off-the-shelf templates). "Prompt chaining vs orchestrator-workers" will be as normal a design-review question as "queue vs topic."

### 17.2 Workflow-Agent Hybrids Become the Norm

Pure agents and pure workflows are increasingly recognized as endpoints, not targets. Production designs converge on **structured workflows with agentic escape hatches** — deterministic skeletons, model-directed holes (Section 11). Not a compromise but the cost/control optimum — and where most 2026 greenfield banking AI will land.

### 17.3 Cost-Aware Pattern Selection

Routing by cost (easy → cheap model, hard → flagship) is becoming a first-class design dimension, not an optimization afterthought — with per-pattern cost models (Section 12.3's bounded-vs-unbounded column) used at design time. Combined with on-prem/private deployment for regulated workloads (see [On-Prem LLM Deployment](on_prem_llm_deployment_guide.md)), cost-aware selection increasingly drives *where* each pattern's steps run.

### 17.4 Workflow Observability Standards

LLM observability is converging on standards: **OpenTelemetry-style tracing for LLM steps** (spans per LLM call, tool call, and validation gate), token/latency/cost metrics per pattern, run-level correlation — see [RAG Frameworks Comparison](ai_llm/rag_frameworks_comparison_guide.md). Expect agentic-workflow telemetry to become a procurement checkbox — the observability layer *is* the audit layer.

### 17.5 Durable Agentic Workflows

The durability layer is absorbing the patterns: durable engines ship pattern primitives (child workflows for orchestrator-workers, signals for HITL, retry policies for evaluator loops), and agent frameworks checkpoint to durable stores. The 2026 shape is **patterns + durability as one design** — pick the durable form of the pattern from the start (see [Durable AI Agent Workflows](durable_ai_agent_workflows_guide.md)).

### 17.6 Regulated-Industry Pattern Guidance

Banking regulators are moving from "is AI allowed?" to "what structure of AI is acceptable?" — and the emerging answer favors **predictability**: documented processes, bounded behavior, reproducible decisions. Expect supervisory expectations to reward workflow/hybrid designs (defined steps, validation gates, HITL, full logging) and to demand extra justification for fully agentic components — naming patterns, justifying agent use, and keeping the audit trail becomes the compliance baseline, not a differentiator.

---

## 18. Conclusion

The agentic-workflow pattern catalog is small but complete: **five workflow patterns** (prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer) plus the **agent loop** cover the design space of production LLM systems. The discipline is not in knowing the patterns — it is in *choosing* between them, and in resisting the pull toward the most complex option.

The decision framework in one breath: **if you can enumerate the steps, build a workflow; if you cannot, build an agent — and if you can bound the unknown part with a skeleton, build a hybrid.** Workflows buy predictability, auditability, and bounded cost at the price of flexibility; agents buy flexibility at the price of all three — which is why banking runs workflow-first with agentic escape hatches, human gates on consequential actions, and full logging everywhere.

Three rules to design by:

1. **Simplest first** — single prompt → workflow → agent, climbing only when the current step fails.
2. **Name your patterns** — "routing into a chain with an evaluator gate" is a design-review sentence that catches most errors before code does.
3. **Durability, observability, and guardrails are part of the pattern** — a pattern without validation gates, tracing, budgets, and human gates is not the pattern; it is the pitfall (Section 16).

The catalog will keep evolving — frameworks will package it, regulators will codify it — but the underlying geometry is stable: control flow lives in code or in the model, and the right answer is the simplest thing that works.

---

## 19. References

1. Anthropic — *Building Effective Agents* (Dec 2024): the workflow/agent taxonomy, the five workflow patterns, and the "simplest solution that works" guidance. https://www.anthropic.com/research/building-effective-agents
2. Yao et al. — *ReAct: Synergizing Reasoning and Acting in Language Models* (2022): the think → act → observe loop.
3. Wang et al. — *Self-Consistency Improves Chain of Thought Reasoning in Language Models* (2022): majority voting over samples — basis of the voting parallelization form (see [Advanced RAG Techniques](ai_llm/advanced_rag_techniques_guide.md)).
4. Asai et al. — *Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection* (2023): self-reflection with retrieval grounding (see [Advanced RAG Techniques](ai_llm/advanced_rag_techniques_guide.md)).
5. Anthropic — *Effective Guardrails for Language Models* (2025): agent guardrail taxonomy (input/output/tool/agent-level).
6. Companion guides (this series): [Hybrid Multi-Agent Systems](ai_llm/hybrid_multi_agent_systems_guide.md) · [Hierarchical Multi-Agent Frameworks](ai_llm/hierarchical_multi_agent_frameworks_guide.md) · [Durable AI Agent Workflows](durable_ai_agent_workflows_guide.md) · [Research Agents](research_agents_guide.md) · [Beyond RAG](ai_llm/beyond_rag_guide.md) · [MCP Framework & Tools](ai_llm/mcp_framework_tools_guide.md) · [Constrained Decoding Frameworks](constrained_decoding_frameworks_guide.md) · [LLM Development Risks & Security](llm_development_risks_security_guide.md) · [Financial Risk & Compliance Systems](../banking/financial_risk_compliance_systems_guide.md) · [Payments Hub](../banking/payments_hub_guide.md) · [On-Prem LLM Deployment](on_prem_llm_deployment_guide.md) · [RAG Frameworks Comparison](ai_llm/rag_frameworks_comparison_guide.md)
