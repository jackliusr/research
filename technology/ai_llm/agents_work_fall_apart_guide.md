# Where Autonomous Agents Work and Where They Fall Apart

> **The Boundary Analysis of Agent Viability — the success conditions (where agents work), the working patterns and domains, the failure conditions, the failure patterns and domains (where they fall apart), the boundary spectrum (deterministic → assisted → autonomous), the 'should this be an agent?' decision framework, and banking worked examples.**

**Author:** Jack Liu Shurui
**Role:** Solution Architect, Crédit Agricole CIB
**Date:** August 2026
**Version:** 1.0
**Repository:** github.com/jackliusr/research
**Series:** AI/LLM Engineering Guides — Agent & Platform track
**Reading time:** ~55 minutes

---

## Table of Contents

1. [The Success Conditions: Where Agents Work](#1-the-success-conditions-where-agents-work)
2. [The Working Patterns](#2-the-working-patterns)
3. [The Working Domains](#3-the-working-domains)
4. [The Failure Conditions: Where They Fall Apart](#4-the-failure-conditions-where-they-fall-apart)
5. [The Failure Patterns](#5-the-failure-patterns)
6. [The Failure Domains](#6-the-failure-domains)
7. [The Boundary Spectrum](#7-the-boundary-spectrum)
8. [The Decision Framework: Should This Be an Agent?](#8-the-decision-framework-should-this-be-an-agent)
9. [The Worked Examples: Banking](#9-the-worked-examples-banking)
10. [The Summary: The Boundary in One Page](#10-the-summary-the-boundary-in-one-page)
11. [Glossary](#11-glossary)
12. [References & Further Reading](#12-references--further-reading)

---

## How This Guide Fits the Series

This guide is the **synthesis and boundary analysis** for the agents series. The series already contains the negative diagnosis and the positive playbook; this guide draws the **line between them**:

- [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) — the **failure catalogue**: the 8-category taxonomy of why agents fail, the compounding-error math (0.95¹⁰ ≈ 0.60), the demo-vs-production gap. That guide answers *"why do agents fail?"* This guide answers *"given that, where should you use one at all?"*
- [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) — the **production playbook**: narrow scope, deterministic cores, verifier patterns, eval gates. That guide answers *"how do I build an agent that survives?"* This guide answers *"which tasks deserve an agent in the first place?"*
- [llm_agent_use_cases.md](llm_agent_use_cases.md) — the **use-case catalogue**: who deploys agents, where, with what evidence. This guide does *not* re-catalogue use cases; it explains *why* those use cases sit on the "works" side of the boundary.
- [autonomous_agents_guide.md](autonomous_agents_guide.md) — the umbrella: definitions, the autonomy spectrum, architectures, control, and evaluation (GAIA, τ-bench). This guide is the *decision layer* on top of that architecture layer.
- [ai_agent_drift_guide.md](ai_agent_drift_guide.md) — drift over time; the temporal failure mode that long-horizon autonomy amplifies (see §5.1 here).
- [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md) — multi-agent orchestration, and why the *overhead* of multi-agent designs is itself a failure condition (§5.2 here).
- [agentops_guide.md](agentops_guide.md) — the ops layer (traces, budgets, alerts) that makes an agent's feedback loop observable.
- [rag/production_grade_agentic_rag_guide.md](rag/production_grade_agentic_rag_guide.md) — the agentic-RAG viability analysis: when retrieval loops are worth the autonomy.
- [implementing-responsible-ai.md](implementing-responsible-ai.md) — HITL, approval gates, audit, governance: the human-oversight condition (§1.6 here).
- [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md) — eval design that the "sufficient exemplars" condition (§1.5) depends on.
- Banking context: [../../banking/core_banking_systems_guide.md](../../banking/core_banking_systems_guide.md) and [../../banking/financial_risk_compliance_systems_guide.md](../../banking/financial_risk_compliance_systems_guide.md) for the system context the worked examples (§9) assume.

**The one-sentence thesis.** *Agents work where the loop is checkable: a bounded task space, verifiable outputs, deterministic feedback, low blast radius, enough exemplars to evaluate, and a human in the loop where stakes rise. Agents fall apart where any of those break — most violently at the intersection of unbounded scope, unverifiable output, and irreversible action.*

**How to read this guide.** Architects and platform leads: read §1 (conditions), §7 (spectrum), §8 (framework), and §9 (worked examples) — the decision core. Operations and reliability engineers: read §4.7 and §5.5 (how failure announces itself and how to see it in telemetry), then the ops companions [agentops_guide.md](agentops_guide.md) and [ai_agent_drift_guide.md](ai_agent_drift_guide.md). Business sponsors: read §10 (one page) and the §9 example table. Skeptics (the healthy ones): read §4-§6 first — the failure side is where the boundary actually gets drawn. Every section is written to stand alone, and cross-references name the sibling guides explicitly.

**Verification note.** Claims are flagged inline as **(verified)** where they are well-established in published engineering guidance or documented incidents, **(emerging)** where practice is newer and consolidating, **(synthesis)** where they are this guide's framework drawn from the verified material, and **(flag)** where the claim circulates in practitioner discourse but lacks a canonical verifiable source. Practitioner heuristics are labeled as heuristics, not laws.

---

## 1. The Success Conditions: Where Agents Work

### 1.1 The Bounded Task Space — The Narrow Scope

**(verified)** The first and least negotiable success condition is a **bounded task space**: the agent's mission is a single well-defined task type whose input space is finite and enumerable enough that a human can specify what "done" looks like. Anthropic's *Building Effective Agents* (December 2024) draws the canonical line: use a **workflow** when the path is predictable and hardcoded; use an **agent** when the number of steps is unpredictable but the *goal* is still concrete — "agents can be used for open-ended problems where it's difficult or impossible to predict the required number of steps." Note the careful phrasing: the *steps* may be open-ended, but the *task* must not be. Every step count in the compounding-error math ([llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) §1) is only defined relative to a bounded mission — an unbounded mission has no defined "success," so no reliability target can be set at all.

This is the **narrow scope** principle that anchors the production playbook: [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) makes "narrow scope" the first design principle precisely because the bounded task space is the *precondition* for everything else — evals, budgets, guardrails, and the drift monitoring of [ai_agent_drift_guide.md](ai_agent_drift_guide.md) all assume a defined mission envelope.

Operational test for boundedness: **can you write the task's acceptance criteria in one paragraph, and would two competent humans agree on whether a given output meets them?** If yes, the space is probably bounded enough. If the answer requires judgment calls about the *goal itself* (not just the path), it is not.

### 1.2 Verifiable Outputs — The Checkable Results

**(verified)** The second condition: the agent's **outputs are verifiable** — there exists an objective checker: a test suite, a schema validator, a deterministic rubric, a ground-truth comparison, or a human reviewer with a checklist. The presence of a checker is what converts an LLM's probabilistic output into a *gateable* artifact.

The strongest evidence is the category that advanced fastest: **coding agents**. SWE-bench Verified exists *because* code has tests — a machine-checkable definition of correct. The agents that score well on it (and, more importantly, the agents deployed in production inside companies like GitHub, Google, and Meta) are evaluated by running tests, not by vibes. The checkable loop is the reason code assistance is the most mature agent domain (§3.2).

By contrast, "draft a compelling strategy memo" has no checker — taste, context, and audience all modulate "good." An agent whose output cannot be checked by anything other than another opinion is an agent whose failures are invisible until a human reads the output carefully — which is exactly the failure mode of §5.4 (silent confidence).

**The verifier pattern (see §2.4) is the engineered remedy for weak checkability**: a second model or a deterministic tool validates the first pass, converting a soft output into a checked one. But a verifier that is itself an LLM only moves the checkability problem one hop — the *outer* loop still needs a ground truth. Checkability can be borrowed, but it cannot be manufactured from nothing.

### 1.3 Deterministic Feedback — The Clear Success/Failure Signals

**(verified)** The third condition is **deterministic feedback within the loop**: after each step, the agent receives a signal that is unambiguous — a tool return code, a test pass/fail, a schema validation result, an API response — and it receives it *while the loop is still running*. Feedback is what makes an agent an agent rather than a one-shot LLM call; deterministic feedback is what makes the agent's loop *converge* rather than wander.

The contrast that matters: coding agents get a compiler and a test suite (deterministic); retrieval agents get relevance scores (soft, model-judged); sales outreach agents get... a reply in three days or never (delayed, stochastic — see §4.3). The agent architectures catalogued in [autonomous_agents_guide.md](autonomous_agents_guide.md) §3 (ReAct, plan-and-execute, reflexion) all assume a feedback channel; the quality of that channel is what separates the patterns that work from the ones that spin.

**(synthesis)** Rule of thumb: **the more deterministic the per-step signal, the more autonomy you can safely grant; the softer the signal, the more you must constrain the loop** — shorter horizons, tighter budgets, more human checkpoints. Feedback quality and autonomy budget trade off almost one-for-one.

### 1.4 Low Blast Radius — The Contained Failures

**(verified)** The fourth condition: **when the agent is wrong, the damage is contained** — actions are reversible (read-only, draft-only, undo-able), the blast radius is small (a failed draft, a misrouted ticket, a wrong suggestion), and there is no irreversible or regulated consequence.

Blast radius is the property that lets an organization *ship* an agent whose error rate is still material. A support-triage agent that misroutes 5% of tickets costs a few minutes of human rework — the blast radius of each mistake is a re-queue. A loan-decision agent that mis-scores 5% of applications creates regulatory, fairness, and reputation exposure that no re-queue can absorb (§6.3). The same underlying error rate is acceptable in one context and disqualifying in the other — *the difference is blast radius, not model quality*.

Containment techniques (from [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) §2 and [implementing-responsible-ai.md](implementing-responsible-ai.md)): read-only toolsets, draft-and-approve output modes, sandboxed execution, human-in-the-loop gates on consequential actions, and hard reversibility (audit logs, rollback). Each technique *moves* a task down the blast-radius axis — which is why §7's spectrum has a "contained" column rather than a binary.

### 1.5 Sufficient Exemplars — The Examples for the Evals

**(verified)** The fifth condition: **you have enough historical exemplars to build the eval set**. The production playbook's eval gates ([production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) §4) and the eval design in [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md) both presuppose a golden set — and golden sets come from *history*. Support-ticket archives, resolved GitHub issues, past extraction jobs with verified outputs, recorded triage decisions: any task with a paper trail of past human-corrected outcomes is a task you can evaluate agents against. A task with no history (a brand-new process, a novel output type, a one-off analysis) is a task you cannot gate — and an ungated agent is an unverifiable one (§1.2 again, now with a data problem rather than a checker problem).

Note the interaction with §1.2: exemplars give you the *ground truth*; a checker gives you the *mechanism* to compare against it. Both are needed. (This is also why agent-eval benchmarks such as τ-bench were built with realistic domain policy sets — see [autonomous_agents_guide.md](autonomous_agents_guide.md) §6 — they exist to supply exactly this condition for benchmarking.)

### 1.6 Human Oversight — The HITL

**(verified)** The sixth condition is the one that scales with stakes: **human oversight, human-in-the-loop (HITL)** — approval gates on consequential actions, escalation paths when confidence is low, and review workflows on high-impact outputs. [implementing-responsible-ai.md](implementing-responsible-ai.md) is the series reference for the governance machinery (approvals, audit trails, roles); [agentops_guide.md](agentops_guide.md) covers the observability that makes oversight *possible* at volume (traces, alerts, budget dashboards).

HITL is not a concession to weakness; it is a **blast-radius multiplier reducer**. A gate on "send payment instruction" turns a high-blast-radius action into a contained one — the agent proposes, the human disposes, and the failure cost collapses to the cost of reviewing a proposal. This is precisely why every enterprise agent platform ([enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md)) ships approval workflows as a first-class primitive, and why Anthropic's and OpenAI's agent-design guidance both recommend explicit confirmation steps for consequential actions.

### 1.7 The Condition Table

| # | Condition | Rationale | Evidence | Counter-example |
|---|---|---|---|---|
| 1 | **Bounded task space** (narrow scope) | Defines "done"; makes reliability targetable; precondition for evals, budgets, drift monitoring | Anthropic *Building Effective Agents*: agents for open-ended *steps*, concrete *goals*; narrow scope is design principle #1 in [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) | "Do my research and tell me what I should know" — no acceptance criteria, no stopping rule |
| 2 | **Verifiable outputs** | A checker converts probabilistic output into a gateable artifact | SWE-bench Verified runs tests, not vibes; coding is the most mature agent domain | "Draft a compelling strategy memo" — no objective checker exists |
| 3 | **Deterministic feedback** | In-loop signals let the agent converge instead of wander | Coding: compilers/tests; τ-bench measures *policy adherence* against domain rules ([autonomous_agents_guide.md](autonomous_agents_guide.md) §6) | Cold-outreach agent: signal arrives days later or never — loop can't learn |
| 4 | **Low blast radius** | Contained failures make material error rates shippable | Draft/read-only/sandboxed agents ship with 5-10% error rates; regulated actions don't | Agent that posts to a live customer account — irreversible, reputational |
| 5 | **Sufficient exemplars** | Historical outcomes supply the golden set for eval gates | Support archives, GitHub issues, verified extraction jobs are the raw material of every eval suite | Brand-new process with no history — nothing to gate against |
| 6 | **Human oversight (HITL)** | Gates collapse blast radius; escalation handles low confidence | Approval workflows are a first-class primitive in [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md); governance in [implementing-responsible-ai.md](implementing-responsible-ai.md) | Agent with full execution rights and no reviewer on payment actions |

**Reading the table.** Conditions 1-3 determine whether the agent *can* work (a learnable, checkable loop); conditions 4-6 determine whether you *should let it* (contained, evaluated, overseen). An agent that fails 1-3 will produce wrong results *silently*; an agent that fails 4-6 will produce wrong results *expensively*. The worst case is failing both — which is precisely the profile of the failure domains in §6.

### 1.8 The Conditions as a Pre-Flight Checklist

(**synthesis**) Before committing engineering time, run the six conditions as a pre-flight checklist with *diagnostic questions* and *minimum bars*. The checklist is deliberately blunt: it is meant to be answered by the business owner and a lead engineer in one meeting, not by a committee.

| Condition | Diagnostic question | Minimum bar | Fails if |
|---|---|---|---|
| Bounded task space | "Write 'done' in one paragraph." | Two humans agree on the paragraph | The paragraph contains words like "insights", "improve", "understand" |
| Verifiable outputs | "What checks the output?" | At least one objective checker exists or can be built | The only check is "a senior person reads it" |
| Deterministic feedback | "What does the agent observe after each step?" | An unambiguous signal arrives within the run | The signal is a human reply, a quarterly metric, or "we'll know later" |
| Low blast radius | "What's the worst single failure?" | The worst failure costs under an hour of human rework (or is gated) | The worst failure is irreversible, regulated, or customer-visible in a harmful way |
| Sufficient exemplars | "Where does the golden set come from?" | At least a few hundred historical, human-corrected examples | No history exists; the task is brand new |
| Human oversight | "Who reviews the consequential 10%?" | A named person/team with the gate as part of their job | The gate is "IT will monitor it" with no owner |

**How to use it.** Score each row pass/fail. **Six passes: build the agent.** **One or two fails: build the copilot or the gated version** and re-run the checklist after the first release (the pilot itself generates exemplars and staffing experience, which often converts fails into passes). **Three or more fails: do not build anything agentic** — the task, not the implementation, is the problem. This mirrors the §8 tree; the checklist is the tree compressed into a meeting agenda.

---
## 2. The Working Patterns

The success conditions in §1 describe *where* agents work. The patterns in this section describe *how* successful production agents are structured — the four shapes that recur across every mature deployment. These are not architecture taxonomy (that lives in [autonomous_agents_guide.md](autonomous_agents_guide.md) §3-4); they are the *organizational* shapes that keep the loop checkable.

### 2.1 The Single-Purpose Agents — One Task

**(verified)** The dominant pattern in production: **one agent, one task, one narrow toolset**. The agent's prompt is a job description; its tools are the minimum needed for that job; its eval set is that job's history; its budget is that job's cost envelope.

Why it works — the math is blunt. The compounding-error product rule ([llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) §1) says success decays as p^N; a single-purpose agent keeps N small by construction. A ticket-triage agent has ~3 steps (classify → retrieve → route); a general "assistant" that might do anything has an unbounded N and therefore an unbounded failure rate. Narrow scope is not conservatism — it is the *only* known way to hold the reliability curve. ([production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) §2 makes this its first design principle.)

Single-purpose also shrinks the drift surface ([ai_agent_drift_guide.md](ai_agent_drift_guide.md) §3): one task type means the input distribution is stable enough to monitor, and drift alerts mean something. A generalist agent's inputs drift constantly *by design*, so drift detection degenerates into noise.

**Examples:** a classifier that routes support tickets; an extractor that pulls trade-confirmation fields into a schema; a summarizer that condenses board-meeting minutes into action items. Each is boring. Each works.

### 2.2 The Workflow-Embedded Agents — The Step in the Workflow

**(verified)** The second pattern: **the agent is a step inside a deterministic workflow** — the surrounding code owns the control flow, the ordering, the retries, and the error handling, and the agent owns exactly one cognitive sub-task. This is the *workflow-with-agent-steps* shape: a fixed pipeline (e.g., ingest → normalize → **agent-classify** → validate → route) where each agent call is sandwiched between deterministic checks.

Why it works: the deterministic shell supplies everything the agent lacks. The workflow validates the agent's output against a schema before it propagates (satisfying §1.2 *from the outside*); it retries or escalates on failure; it enforces ordering so the agent never has to plan; it can even run the agent in a "suggest then verify" mode where a deterministic rule (e.g., "amount must equal sum of line items") vetoes the agent's output. Anthropic's *Building Effective Agents* calls this class the **workflow patterns** (prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer) and advises: *"when you can predict the path, use a workflow — agents add complexity for no benefit."* The production playbook's "deterministic cores" principle ([production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) §2) is the same idea from the reliability side: put the parts that *can* be deterministic in deterministic code, and use the LLM only where judgment is genuinely required.

**Examples:** a KYC pipeline where the agent extracts entity names and a deterministic database check validates them; an invoice workflow where the agent reads the line items and a reconciliation rule checks the totals; a report pipeline where the agent drafts and a deterministic style/schema linter gates the draft.

### 2.3 The HITL Agents — Human-in-the-Loop with Approval Gates

**(verified)** The third pattern: **the agent proposes, the human disposes** — the agent executes up to a defined point, then stops at an **approval gate** where a human reviews and either approves, edits, or rejects before the consequential action proceeds. This is the operational form of §1.6.

Why it works: the gate converts high blast radius into contained failure (§1.4) *without* requiring the agent to be more accurate than it is. The agent absorbs the 80-90% of cases that are routine; the human absorbs the 10-20% that are consequential, ambiguous, or novel — and in doing so, the human also *labels* those cases, feeding the exemplar pool (§1.5) that improves the agent. The gate is also the cheapest possible escape hatch for every failure pattern in §5: drift, loops, and silent confidence all get caught at the gate.

The evidence base is the enterprise platform reality: approval workflows are first-class primitives in [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md), and the responsible-AI machinery in [implementing-responsible-ai.md](implementing-responsible-ai.md) (approvals, audit trails, four-eyes principles) is *designed around* HITL gates rather than around fully autonomous agents. Regulated sectors (banking, insurance, healthcare) have made HITL the default operating model not out of caution but because the regulators' expectation of *accountability* requires a named human owner for consequential actions.

**Examples:** draft-outbound-communications agents that stop before send; payment-instruction agents that stop at an approver queue; credit-decision support that stops at an underwriter. (See §9 for the banking worked examples.)

### 2.4 The Verifier-Augmented Agents — The Second-Pass Check

**(verified)** The fourth pattern: **a second pass checks the first** — the agent's output is validated by a verifier before it is accepted. The verifier can be a deterministic rule (schema check, business rule, checksum), a different model instance with an evaluation prompt, a retrieval-augmented fact-check (does the output's claims appear in the source documents?), or a guard model ([llm_guard_models_guide.md](llm_guard_models_guide.md)). Anthropic's *Building Effective Agents* formalizes the LLM-as-verifier variant as the **evaluator-optimizer** workflow pattern, and notes it only works *"when there is a clear criterion to evaluate against"* — which is, precisely, the checkability condition (§1.2) restated.

Why it works: the verifier breaks the two structural weaknesses of a single-pass LLM. First, **silent confidence** (§5.4): a single model emits wrong-but-confident output with no internal alarm; an independent second pass has decorrelated errors, so it catches a meaningful fraction of first-pass mistakes. Second, **self-review blindness**: asking the same model to review its own output rarely helps (the error is in its weights, not its attention); an *independent* verifier (different prompt, different model, different modality — deterministic where possible) is where the improvement comes from. The verifier also creates the checkable artifact that eval gates need — verified outputs can be scored against golden sets, closing the loop of §1.5.

**Examples:** code agents that run tests after editing (the test suite is the verifier — the single most effective verifier in the industry); extraction agents validated by schema checks; summarization agents validated by a "claims must appear in source" retriever; RAG answers validated by citation coverage ([rag/production_grade_agentic_rag_guide.md](rag/production_grade_agentic_rag_guide.md)).

### 2.5 The Pattern Table

| Pattern | Mechanism | Why it works | Example |
|---|---|---|---|
| **Single-purpose** | One agent, one task, minimal toolset, per-task budget & eval | Keeps step count N small so p^N stays high; small drift surface | Ticket-triage agent (classify → retrieve → route) |
| **Workflow-embedded** | Agent is one step in a deterministic pipeline with external validation & retries | The deterministic shell supplies checkability, ordering, and error handling the agent lacks | KYC pipeline: agent extracts entities, DB check validates them |
| **HITL** | Agent proposes, human approves at gates before consequential actions | Converts high blast radius into contained failure; humans label the hard cases | Payment-instruction agent stopping at an approver queue |
| **Verifier-augmented** | Independent second pass (tests, rules, second model) checks the first | Decorrelated errors catch silent confidence; creates gateable artifacts | Coding agent runs the test suite after every edit |

**Reading the table.** The four patterns are not mutually exclusive — the strongest production systems combine them: a single-purpose agent (§2.1) embedded in a workflow (§2.2), with a verifier gate (§2.4) and a human gate on the top 10% of cases (§2.3). Notice that all four are *containment* patterns: each one exists to make the agent's inevitable errors cheap and visible. That is the entire game.

### 2.6 When the Patterns Break — And How to Tell

Each pattern has a characteristic way of degrading, and each degradation is observable early if you look:

- **Single-purpose breaks by scope creep.** Feature requests accumulate ("can it also handle...") until the agent is a generalist with a specialist's eval set — its eval suite no longer covers its inputs, and reliability silently drops. **Tell:** the backlog for the agent contains more feature requests than the roadmap for any deterministic service.
- **Workflow-embedded breaks by bypassed checks.** Someone adds an "agent escape hatch" (free-text overrides, direct tool access) to handle edge cases, and the deterministic shell stops being the gatekeeper — the agent's unvalidated output starts propagating. **Tell:** production incidents trace to outputs that skipped the validation step.
- **HITL breaks by unstaffed gates.** The approval queue exists and nobody owns it; reviews become rubber stamps or batch approvals at midnight. This is the failure mode of §8.2's sixth question, and it is the most common *organizational* cause of agent incidents. **Tell:** median gate latency drifts up, and gate-rejection rate trends toward zero (nobody is actually reading).
- **Verifier-augmented breaks by verifier blindness.** The verifier is the same model family, same prompt style, same blind spots as the generator — a second opinion that agrees with the first because it is the same brain. **Tell:** the verifier's rejection rate is near zero on classes of error the generator is known to make.

**The general rule:** every pattern works by *containment* — the moment the containment is bypassed, unstaffed, or blind, the agent reverts to its raw failure rate, and §4's conditions re-apply in full.

### 2.7 Choosing Among the Patterns

(**synthesis**) The four patterns are not interchangeable styles; they are answers to different boundary problems. Selection guide:

- **Single-purpose is the default for every agent.** Start there; scope is the first condition (§1.1), and the pattern *is* the scope. If the task cannot be expressed as one narrow mission, no pattern will save it.
- **Workflow-embedded is a gift from the process — take it whenever the task sits inside an existing pipeline.** If a deterministic flow already surrounds the task (a queue, a validation stage, a routing step), embedding is nearly free and buys external checkability (§2.2). The question is not "should I embed?" but "which step exactly is the agent?"
- **HITL is the answer whenever blast radius exceeds tolerance (§4.4) or checkability is partial.** Add the gate where the consequence is, and only there — gating every step (approval theater) is as corrosive as gating none.
- **Verifier-augmented is the answer whenever a checker exists or can be built cheaply — and it almost always can.** Tests, schemas, business rules, guard models ([llm_guard_models_guide.md](llm_guard_models_guide.md)): the verifier is the highest-leverage pattern because it directly repairs the two weakest links — silent confidence (§5.4) and weak checkability (§1.2).

**The combination rule.** If the budget allows one pattern: **single-purpose + verifier** (this is the core of every successful agent). Two patterns: **add HITL** on the consequential 10%. Three: **embed in the workflow** if one exists. The order reflects leverage: containment of *errors* (verifier), then containment of *consequences* (gate), then containment of *context* (workflow).

---

## 3. The Working Domains

Where do the six conditions and four patterns actually hold in the wild? Five domains have accumulated enough production evidence to be called "working" — with maturity varying from proven to consolidating. (The full company-by-company catalogue is [llm_agent_use_cases.md](llm_agent_use_cases.md); this section explains the *boundary reasons* each domain sits on the "works" side.)

### 3.1 Customer Support — Triage and Retrieval

**(verified, maturity: proven)** The canonical working domain. An agent classifies the customer's intent, retrieves the relevant policy/article/account data, and either answers, drafts, or routes. Every success condition is satisfied *by the shape of the domain itself*:

- **Bounded:** support intents are a finite, known set (password reset, refund status, fee query, card dispute...).
- **Verifiable:** answers must match the knowledge base; retrieval can be citation-checked.
- **Feedback:** customer resolution/escalation outcomes are recorded — the domain generates its own labels.
- **Blast radius:** a wrong answer is annoying, not catastrophic — and there is always a human agent behind the queue (HITL by construction).
- **Exemplars:** ticket archives are the largest labeled dataset in most enterprises.

The scale evidence is substantial but should be read with care (**(flag)** vendor-reported metrics are directional, not audited): Klarna's assistant handled ~two-thirds of customer-service chats in its first month (~2.3M conversations, ~700 FTE-equivalent claim — contested in later reporting, with the company subsequently re-hiring human agents and moving to a hybrid model); Intercom's Fin has published ~51% resolution-rate case studies; Sierra and Decagon run enterprise deployments for large brands. The τ-bench benchmark ([autonomous_agents_guide.md](autonomous_agents_guide.md) §6) exists specifically because support triage is the domain where agent reliability is worth measuring rigorously — its retail/airline/banking domains simulate exactly this loop, and frontier models still show material policy-adherence gaps at long horizons, which is why production support agents are kept *narrow and escalated*, not fully autonomous.

### 3.2 Code Assistance — Suggestions and Review

**(verified, maturity: most proven)** The most mature agent domain, and the one that best demonstrates the conditions working together. The feedback loop is deterministic (compiler, linter, test suite), outputs are machine-verifiable (diffs, passing tests), blast radius is low (code changes are reviewable and revertible), and exemplars are abundant (every repo's issue/PR history). Anthropic's 2026 *State of AI Agents* report found ~90% of surveyed organizations already use AI to assist with coding — the highest penetration of any agentic category.

The shape of the domain matters as much as the maturity: the *dominant* deployment mode is **suggest/review, not autonomous ship**. Copilot-style inline suggestions, review agents that flag issues on PRs, and agentic coding tools (Claude Code, Codex, Cursor agent mode) that *propose* changes for the developer to accept. Even the most autonomous coding agents run inside a loop whose verifier is the test suite (§2.4) and whose gate is the developer's review (§2.3). The domain "works" exactly because the checkability conditions are physical — code is the one artifact where correctness has a machine definition.

### 3.3 Data Analysis — Exploration

**(emerging → consolidating)** Agents that explore data: write queries, run analyses, produce charts and summaries. The success conditions are *conditionally* met: code execution provides deterministic feedback (the query either runs and returns, or errors), outputs are verifiable against the data (numbers can be re-computed), and blast radius is low (read-only analysis, draft reports). The boundary wrinkle: **exploration is where scope creep lives**. "Explore this data and find insights" is unbounded (§4.1) — the *goal* is open, so the agent wanders. The deployments that work bound the exploration ("answer these five questions from this dataset") and embed the agent in a workflow with a human analyst reviewing findings (HITL by review). Notebook-style agents (Jupyter-interactive) are the pattern of record because the human sees every intermediate step — checkability by transparency.

### 3.4 Document Processing — Extraction and Summarization

**(verified, maturity: proven)** Extraction (fields → schema) and summarization (documents → structured briefs) are the quiet workhorses. Both are bounded (the schema or the brief format defines "done"), verifiable (extracted fields can be validated against ground truth; summaries can be checked against sources — see the citation-coverage verifier in §2.4), low-blast-radius (drafts and extracted data, human-reviewed before use), and exemplar-rich (every enterprise has a filing cabinet of past documents and their human-verified extractions). The agentic-RAG viability analysis in [rag/production_grade_agentic_rag_guide.md](rag/production_grade_agentic_rag_guide.md) reaches the same verdict for the retrieval side: the loop is checkable when the corpus is bounded and answers are citation-checked. This domain is where most "agent pilot that quietly went into production" stories live.

### 3.5 Internal Workflows — The Emerging Long Tail

**(emerging)** The fastest-growing category is agents operating *inside* a company's own tooling: approvals routing, report generation, ticket enrichment, onboarding checklists, internal knowledge retrieval. The pattern is workflow-embedded (§2.2) almost by definition — these agents live in the middle of existing deterministic processes, which is precisely why they work. The full catalogue of internal use cases (with company evidence) is in [llm_agent_use_cases.md](llm_agent_use_cases.md) §12-13; the boundary point here is that internal workflows inherit their checkability from the process they sit inside — if the surrounding workflow validates, the agent can be sloppy; if it doesn't, the agent inherits the workflow's lack of gates. **Maturity caveat:** most internal-workflow agents are still measured on throughput (tickets handled, hours saved) rather than accuracy (error rate, escalation rate) — the metrics gap is real and is exactly what [agentops_guide.md](agentops_guide.md) and [ai_agent_drift_guide.md](ai_agent_drift_guide.md) exist to close.

### 3.6 What the Working Domains Have in Common

The five working domains look different, but they share five structural features — and those features are the boundary reasons in disguise:

1. **They produce artifacts, not decisions.** The output is a draft, a suggestion, a classification, an extraction — something a human reviews before it becomes consequential. The domains that fail (§6) are the ones where the output *is* the decision.
2. **The intents are enumerable.** Support intents, code changes, extraction schemas, analysis questions, workflow steps: finite sets a human can write down. Unbounded domains have no such set.
3. **A human queue exists behind the agent.** The support team, the developer, the analyst, the ops reviewer — the working domains all have an existing human workforce whose workflow the agent accelerates, not replaces. Escalation is never a new invention.
4. **History exists.** Ticket archives, repos, filing cabinets, past reports: the exemplars for evals (§1.5) are byproducts of the domain having existed before the agent.
5. **Failure is cheap and visible.** A wrong suggestion costs minutes; a wrong draft is caught in review. Cheapness is what lets these domains tolerate the error rates agents actually have.

(**synthesis**) The common denominator, in one phrase: **the working domains are all "assist the worker" shapes.** That is not a coincidence — it is the boundary doing its work. Agent value concentrates where the loop is checkable (§1) and the human is the backstop (§1.6); the domains that organize themselves that way win, and the ones that don't, fail.

### 3.7 The Domain Table

| Domain | Agent role | Why it works (boundary reasons) | Maturity |
|---|---|---|---|
| **Customer support** | Triage, retrieval, draft, escalate | Bounded intents; KB-verifiable answers; recorded outcomes as feedback; human queue behind the agent | **Proven** (metrics vendor-reported, (flag)) |
| **Code assistance** | Suggestions, review, test-driven edits | Deterministic feedback (tests/compiler); machine-verifiable diffs; revertible | **Most proven** |
| **Data analysis** | Query, explore, chart, brief | Code-execution feedback; recomputable outputs; read-only | Emerging → consolidating |
| **Document processing** | Extraction, summarization | Schema-verifiable; source-checkable; exemplar-rich | **Proven** |
| **Internal workflows** | Steps inside company processes | Inherits checkability from the surrounding deterministic workflow | Emerging (metrics gap) |

---

## 4. The Failure Conditions: Where They Fall Apart

Now the other side of the boundary. Each success condition has a mirror-image failure condition — and the failure conditions are not just "the absence of the success conditions." They are *active* properties of the task that pull agents apart even when the model is strong. The full 8-category failure taxonomy is in [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) §2; this section is the *boundary* view: the five conditions that predict failure before you write a line of code.

### 4.1 The Open-Ended Tasks — The Unbounded Scope

**(verified)** An **open-ended task** has no defined acceptance criteria and no natural stopping point: "research this topic," "improve our process," "find insights," "draft a strategy." Unbounded scope breaks the agent at the *goal* level, not the step level:

- **No stopping rule.** The agent cannot decide it is done, so it keeps going — consuming budget, adding steps, and compounding error (0.95¹⁰ applies *and* N is unbounded, so reliability → 0).
- **No eval set.** You cannot score "did the research succeed" against a golden set (§1.5 fails).
- **No drift baseline.** An unbounded mission has an unbounded input distribution; [ai_agent_drift_guide.md](ai_agent_drift_guide.md)'s monitoring assumes a defined envelope (§2.1 fails).

**(flag)** Practitioner discourse in 2025-2026 contains a circulating heuristic sometimes called the **"70% rule"** — roughly, "don't build an agent for a task unless a competent human can complete it reliably ~70%+ of the time, because an agent needs a learnable target." **Verification: no canonical published source exists for this specific heuristic** (the term collides with the real-estate 70% rule and the 40-70 decision heuristic); treat it as folk wisdom, directionally consistent with the compounding math, but not a rule to gate engineering decisions on. The grounded version of the same intuition: if the task's success is not *definable*, an agent cannot *learn* it — boundedness first, thresholds later.

### 4.2 The Unverifiable Outputs — No Ground Truth

**(verified)** When the output has **no ground truth** — no test, no schema, no objective rubric, no historical answer — the agent operates with an invisible error rate. Every failure is silent until a human happens to notice (§5.4). The structural problem is not that the model is wrong; it is that *nobody can tell*, including the eval pipeline, so the agent cannot be improved, gated, or trusted. Unverifiable outputs also defeat the verifier pattern (§2.4): a second LLM "checking" an essay for quality is a second opinion, not a verification. (§1.2's checkability is the positive statement; this is its negative.)

### 4.3 The Delayed Feedback — No Signal in the Loop

**(verified)** When the feedback for an action arrives **after the loop has closed** — or never arrives — the agent cannot learn within the episode and cannot converge. Sales outreach (reply in days, if ever), hiring decisions (outcome in weeks), marketing campaigns (results in quarters): the agent acts, the loop ends, and the signal arrives too late to shape the run. Delayed feedback is distinct from weak feedback: a soft signal (a relevance score) is at least *in* the loop and can steer; a delayed signal is *outside* the loop and can only steer the *next* run — which means each run is a blind shot, and the compounding math applies with p estimated from nothing. The only remedy is to shorten the loop (use proxy signals: engagement rates, open rates, intermediate metrics) or to take the human back into the loop as the in-episode judge.

### 4.4 The High Blast Radius — The Costly Failures

**(verified)** When a wrong action is **irreversible, expensive, regulated, or reputationally damaging**, the agent's error rate — however low — is multiplied by a cost that no reliability budget can absorb. High blast radius is the failure condition that *regulators* care about, which is why it dominates the §6 failure domains. Two properties to distinguish:

- **Irreversibility** — the action cannot be undone (payment sent, contract signed, message published, account closed).
- **Stakes concentration** — even reversible actions can be catastrophic when the individual failure is large (a wrongly rejected loan application, a wrong dosage recommendation).

Both are gated by the HITL pattern (§2.3) — which is why the *presence of a mandatory human gate* is the single most reliable predictor of whether a high-stakes task can host an agent at all.

### 4.5 The Compounding Steps — 0.95¹⁰ ≈ 0.60

**(verified — arithmetic; the series' canonical treatment is [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) §1)** The deepest failure condition is structural: **agent reliability decays exponentially in step count** (Lusser's law for series systems). 0.95¹⁰ ≈ 0.5987 — a 95%-per-step agent finishes a 10-step task only ~60% of the time; 0.95²⁰ ≈ 0.36; 0.90¹⁰ ≈ 0.35. The failure guide's verification note adds the crucial pessimism: real steps are *correlated* (a wrong step poisons the next), so the product rule is the **optimistic** bound — real long-horizon agents do worse, not better.

The boundary consequence: **long-horizon tasks fail structurally, not because of any one mistake.** A 30-step autonomous research mission at 95% per-step reliability succeeds ~21% of the time *even if every individual step is "good."* No prompt, model, or framework fix changes the exponent — only cutting steps, adding per-step verifiers, or putting a human at the checkpoints does. This is why §8's decision framework treats step count and feedback quality as the two levers that actually move the verdict.

### 4.6 The Failure Condition Table

| # | Condition | Failure mode | Example |
|---|---|---|---|
| 1 | **Open-ended task** (unbounded scope) | No stopping rule → budget bleed, unbounded N → reliability → 0; no eval set; no drift baseline | "Research our competitors and recommend a strategy" |
| 2 | **Unverifiable outputs** (no ground truth) | Invisible error rate; failures silent until a human notices; agent cannot be gated or improved | "Draft a persuasive investor update" |
| 3 | **Delayed feedback** (no in-loop signal) | Agent acts blind within each episode; cannot converge; each run is a shot in the dark | Cold outreach agent — reply arrives days after the loop ends |
| 4 | **High blast radius** (costly failures) | Small error rates × large per-failure cost; regulatory and reputational exposure | Wrong loan rejection, wrong payment, wrong clinical advice |
| 5 | **Compounding steps** (long horizon) | Reliability decays exponentially: 0.95¹⁰ ≈ 60%, 0.95²⁰ ≈ 36% — even with good steps | 30-step autonomous research mission ≈ 21% success at 95%/step |

**Reading the table.** Conditions 1-3 predict *silent* failure (the agent looks busy while being wrong); conditions 4-5 predict *expensive* failure (the agent is wrong in ways that cost). The five conditions also interact: open-ended + unverifiable + high blast radius is the toxic triplet that defines §6's failure domains — which is why those domains are not "hard problems to solve" but "wrong places to put an agent at all."

### 4.7 How the Failure Conditions Announce Themselves

Failure conditions are diagnosable *before* launch if you know the symptoms — each leaves fingerprints in the demo phase and the first weeks of production. ([llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) §3 covers the same gap from the "works in the demo" side; this is the boundary-side view.)

| Condition | Early symptom (pre-launch) | Production signature |
|---|---|---|
| Open-ended task | Demo script needs constant steering; the agent invents new sub-goals; reviewers disagree on whether the demo "worked" | Token/cost per task climbs run over run; no two runs end the same way |
| Unverifiable outputs | No one can point to the eval set; demo success is judged by taste; "it looks good" is the review standard | Complaints arrive with no corresponding eval failure — the error was never measurable |
| Delayed feedback | No proxy metric exists; the team plans to "see how it goes" | Runs are unlearnable; the same mistake recurs every cycle because nothing corrected it in-loop |
| High blast radius | Security, legal, or risk review grows longer each round; approval routing gets redesigned repeatedly | Every failure escapes the technical domain into reputation or regulatory space |
| Compounding steps | Long runs fail in the demo at a rate that is blamed on randomness rather than counted; retries hide the pattern | Failure rate correlates with run length; short tasks pass, long tasks fail, and per-step quality looks fine |

**The diagnostic habit:** count, don't narrate. If you cannot produce a number for *"what fraction of end-to-end runs succeeded in the last 100 trials, split by task length,"* you do not yet know which failure conditions apply to your task — and you are not ready to decide where it sits on the boundary.

---
## 5. The Failure Patterns

The failure conditions in §4 are properties of the *task*. The failure patterns in this section are properties of the *agent design* — the recurring behavioral shapes that emerge when an agent runs under the wrong conditions. They are the four most common "ways agents actually die in production" (the complete taxonomy is [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) §2; here they are mapped to the boundary).

### 5.1 The Long-Horizon Autonomy — Drift

**(verified)** Give an agent a long mission and leave it alone, and it **drifts**: the goal interpretation slips, the tool choices wander, the outputs slowly stop matching the original intent. The temporal dimension is the subject of [ai_agent_drift_guide.md](ai_agent_drift_guide.md) in full — its taxonomy distinguishes input drift, goal drift, tool drift, and output drift — but the *boundary* point is simpler: **drift is the behavioral signature of §4.1 + §4.5**. Unbounded scope gives the agent room to wander (§4.1); long horizons give it time to compound every small misstep (§4.5). A 10-step agent that finishes in 40 seconds drifts negligibly; a 40-step agent that runs for an hour drifts measurably; a multi-day "autonomous research" agent drifts into a different task entirely.

The failure guide's framing is worth repeating here: drift is what happens *between* Tuesday and Friday — it is not a day-one bug, it is a time-dependent one. That makes it the failure pattern least likely to show in a demo and most likely to show in month three of production. The mitigations are all boundary moves: shorten the horizon (workflow-embed, §2.2), gate at checkpoints (HITL, §2.3), monitor against the drift baselines ([ai_agent_drift_guide.md](ai_agent_drift_guide.md) §5-6), and re-run evals on schedule ([production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) §4).

### 5.2 The Multi-Agent Chaos — Communication Overhead

**(verified)** The failure pattern that tempts every architecture diagram: **many agents, each specialized, talking to each other.** The mechanism of failure is overhead: every handoff is a step (so N grows and p^N collapses), every message is a chance for miscommunication (so per-step p drops), and no agent owns the whole outcome (so accountability vanishes). [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md) documents the orchestration patterns and their costs; the boundary-relevant conclusion is that **multi-agent designs multiply the compounding math**: a 5-agent team with 3 handoffs per task is a 15+ step chain, and 0.95¹⁵ ≈ 46% — before any cross-agent miscommunication is counted. The communication overhead is not just latency; it is error surface.

This is why the most successful "multi-agent" systems in production are actually **single-orchestrator systems** ([enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md)) where one controller delegates narrow sub-tasks to narrow agents and *validates each result before the next step* — i.e., multi-*step* but single-*mind*, with the workflow shell of §2.2 providing the checks. True peer-to-peer agent swarms (debate, market, gossip topologies) remain research shapes; the boundary verdict: **add agents only when the task genuinely requires parallel specialization, and only with a deterministic supervisor and per-handoff validation.** If a single agent with a good toolset can do the job, it should — the single agent is the lower-N design by construction.

### 5.3 The Tool Loops — Runaway

**(verified)** The **tool loop**: the agent repeatedly calls the same tool — retrying, re-querying, re-fetching, sometimes re-triggering the same side effect — without making progress. Mechanisms: ambiguous tool results (the agent can't tell failure from "try again"), missing loop budgets (no max-iterations, no cost cap, no time bound), and occasionally genuine pathology (an agent that *enjoys* calling a tool because each call yields tokens of "progress"). The cost is not just money — though runaway loops are the classic cost-bomb incident ([agentops_guide.md](agentops_guide.md) treats budget caps as table stakes) — it is that a looping agent *blocks the queue*: rate limits, degraded service for real users, and paged on-call engineers.

The boundary reading: tool loops are **deterministic feedback gone wrong at the orchestration layer**. The *signal* exists (the tool returns), but the *policy* for acting on it is missing. The fixes are structural, not prompt-level: hard iteration caps, budget ceilings, idempotency keys on side-effecting tools, and "same-tool-N-times" circuit breakers. [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) §2.4 catalogues the loop failure mode; the design-side countermeasures are in [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) §2 (deterministic cores, loop budgets) and [agent_scaffolding_guide.md](agent_scaffolding_guide.md) (the loop scaffold with configurable stop conditions).

### 5.4 The Silent Confidence — Wrong-but-Confident

**(verified)** The most dangerous pattern, because it is invisible by construction: **the agent is wrong and does not know it, and nothing in its output signals the error** — no hedge, no low-confidence flag, no request for help. LLMs are poorly calibrated: the probability mass assigned to a confident-sounding wrong answer is often indistinguishable from a correct one. The pattern is the behavioral signature of §4.2 (unverifiable outputs) — when nothing can check the agent, every error is a silent-confidence error.

The documented incidents cluster exactly where checkability is absent. **(verified)** Air Canada's chatbot (2024) fabricated a bereavement-fare refund policy that did not exist and the airline was held liable for the chatbot's hallucination in a B.C. tribunal ruling; DPD's chatbot (2024) was prompted into profanity by a customer and the exchange went viral before the bot was suspended; New York City's MyCity chatbot (2024) told small-business owners they could break the law (refuse service based on race) — a "wrong-but-confident" failure that a checker would have caught. **(flag)** These are chatbot-class incidents rather than multi-step agent incidents (the latter are mostly internal and undisclosed), but the mechanism is identical: confident output, no verification, consequential harm.

The countermeasures are the whole §2 toolkit: verifiers (§2.4) to catch what the first pass misses, HITL gates (§2.3) to catch what verifiers miss, and confidence escalation — train the agent to *stop and ask* when evidence is thin, which turns silent confidence into *loud uncertainty* that a human can resolve. The reminder for every reader: **if you cannot tell from the output that the agent was wrong, you are running silent confidence right now.**

### 5.5 Detecting the Patterns: The Observability Tell

Every failure pattern leaves a trace in the telemetry before it becomes an incident — if the telemetry exists. This is the operational bridge to [agentops_guide.md](agentops_guide.md) (traces, budgets, alerts) and the drift-detection machinery of [ai_agent_drift_guide.md](ai_agent_drift_guide.md) §5:

| Pattern | The tell in the telemetry | What to watch |
|---|---|---|
| Long-horizon autonomy (drift) | Output distribution shifts while inputs stay stable; goal-token similarity decays over a run | Drift alerts on output embeddings; per-run goal-relevance score ([ai_agent_drift_guide.md](ai_agent_drift_guide.md)) |
| Multi-agent chaos | Handoff message volume grows per task; retry rate spikes at handoff boundaries; no agent owns end-to-end latency | Per-handoff error rate; handoff count distribution |
| Tool loops | Same-tool call count per run spikes (the loop signature); cost per task climbs without task-length growth | Tool-call frequency histograms; max-iterations budget hits ([agentops_guide.md](agentops_guide.md)) |
| Silent confidence | Escalation/review rejection rate trends to zero while downstream complaints stay flat — the checker is not catching what the users see | Gate-rejection rate vs. complaint rate cross-plot; eval regression on adversarial sets |

**The rule:** the absence of a tell is not evidence of health — it is evidence of missing telemetry. Patterns that cannot be seen cannot be stopped; the four patterns in §5.1-5.4 are all *detectable in principle*, and every production agent program that survives past quarter one has built exactly these dashboards.

### 5.6 The Pattern Table

| Pattern | Mechanism | Why it fails | Example |
|---|---|---|---|
| **Long-horizon autonomy** | Long missions, little supervision; goal/tool/output drift over time | Unbounded scope + compounding steps; failures appear in month 3, not the demo | Multi-day "autonomous research" agent that ends up solving a different problem |
| **Multi-agent chaos** | Specialized agents hand off work to each other | Handoffs add steps (p^N collapses) and miscommunication drops per-step p; accountability vanishes | 5-agent team, 3 handoffs per task: a 15+ step chain at ~46% reliability |
| **Tool loops** | Agent re-calls the same tool without progress | No loop budget/circuit breaker; ambiguous tool results read as "try again"; cost bombs and queue blocking | Agent re-fetching a flaky API 40 times, burning budget and rate limits |
| **Silent confidence** | Wrong output emitted with full confidence, no signal of error | No checker exists (§4.2); calibration is poor; harm surfaces only when a human happens to notice | Air Canada chatbot's fabricated refund policy (2024) — airline held liable |

---

## 6. The Failure Domains

Some domains fail *structurally* — not because implementations are bad, but because the domain's shape violates the conditions. These are the places where the honest architectural answer is "don't put a fully autonomous agent here; put a copilot or a gated workflow instead." (The use-case catalogue in [llm_agent_use_cases.md](llm_agent_use_cases.md) treats these same sectors from the opportunity side; this section is the risk side of the same coin.)

### 6.1 Open-Ended Research — The Unbounded

**(verified, status: pilots only)** Academic literature review, market landscape analysis, open-ended "what should we do about X" research. The agent has no stopping rule (§4.1), no ground truth (§4.2), and feedback arrives only when a human reads the output (§4.3) — the toxic triplet in one domain. The evidence is the benchmark data: GAIA ([autonomous_agents_guide.md](autonomous_agents_guide.md) §6) was designed so humans answer with ~100% reliability while published assistants scored under 50% for years after launch — and GAIA tasks are *bounded*; real open-ended research is strictly harder. **(synthesis)** The production compromise is the **research copilot**: the agent gathers, organizes, and drafts (bounded sub-tasks with retrieval-verifiable outputs), and the human owns the research question end-to-end. That shape works; the "autonomous research analyst that delivers conclusions" shape does not. (Compare the honest framing of researcher agents in [../research_agents_guide.md](../research_agents_guide.md).)

### 6.2 Unregulated Actions — Real-World Effects

**(verified, status: gated only)** Any agent whose actions touch the real world without a check — posting to social media, sending messages, booking, ordering, buying, publishing — fails the blast-radius condition by construction (§4.4), and the failure is *visible to the public*: DPD's chatbot incident (§5.4) is the canonical case of an agent whose "action" (a published reply) had real-world effect with no gate. **(flag)** As of 2025-2026 there is no large corpus of published *fully-autonomous* real-world-action agent failures — vendors gate such agents by default and incidents are usually settled privately — but the near-miss class (ungated chatbots publishing harmful output, agents triggering unwanted transactions in testing) is documented repeatedly in practitioner postmortems. The boundary rule: **the more real-world the effect, the more mandatory the human gate** — an agent may *propose* anything, but *execute* only what a gate allows. This is not a reliability position; it is a control-position: §2.3 exists precisely for this domain.

### 6.3 High-Stakes Financial Decisions — Loan Decisions

**(verified, status: assist-only under regulation)** Credit decisions, loan approvals, limit increases, pricing decisions: the regulated, high-blast-radius core of banking. The failure conditions are structural: outputs are unverifiable in the *accountability* sense (there is no "correct" loan decision — there is a fair, documented, regulatorily-sound one), blast radius is high (a wrong rejection harms a customer and invites regulatory action; fair-lending law prohibits disparate outcomes), and the feedback is delayed (default data arrives months later — §4.3). The sector's answer is the **assist shape**: the agent (or model) produces a decision *recommendation* with explanation, and a human underwriter owns the decision — exactly the HITL pattern of §2.3, which is why banking regulation (e.g., EU AI Act high-risk classification, local fair-lending rules) effectively mandates it. [implementing-responsible-ai.md](implementing-responsible-ai.md) and the banking context guides ([../../banking/financial_risk_compliance_systems_guide.md](../../banking/financial_risk_compliance_systems_guide.md)) cover the governance machinery. The boundary verdict: the *data pipeline* and *recommendation* can be agentic; the *decision* cannot — see §9.2 for the worked example.

### 6.4 Medical Decisions — Clinical Decisions

**(verified, status: assist-only, flag on specifics)** Clinical decision-making — diagnosis, treatment selection, dosage — sits firmly on the failure side for fully autonomous agents: blast radius is life-critical (§4.4), ground truth is contested (medicine is probabilistic and case-dependent), and regulators have drawn the boundary explicitly (clinical decision support is high-risk under the EU AI Act; FDA has not cleared autonomous diagnostic agents for general use as of 2026). **(flag)** Specific "autonomous medical agent harmed a patient" incidents are not publicly documented in verifiable form as of writing — the deployment reality is that nothing fully autonomous ships in clinical paths, which is itself the evidence: the domain is so gated that the failure mode is pre-empted rather than observed. What *does* ship is the assist shape everywhere: documentation agents, triage-support tools, literature-retrieval agents, radiology *triage* aids — all with a clinician in the loop and a regulatory frame around them. The boundary rule matches §6.3: **agents may prepare, humans must decide.**

### 6.5 The Domain Table

| Domain | Agent role (attempted) | Why it fails | Status |
|---|---|---|---|
| **Open-ended research** | "Autonomous research analyst" | Unbounded scope, no ground truth, delayed feedback — the toxic triplet | Pilots only; production shape is a research *copilot* |
| **Unregulated real-world actions** | Posting, messaging, booking, transacting | Blast radius by construction; public, irreversible effects; no gate | Gated only — propose yes, execute behind approval |
| **High-stakes financial decisions** | Loan/credit decisions | Unverifiable in the accountability sense; regulated; delayed feedback (defaults) | Assist-only — recommendation + human underwriter |
| **Medical decisions** | Diagnosis/treatment | Life-critical blast radius; contested ground truth; regulatory walls (EU AI Act, FDA) | Assist-only — prepare, never decide |

**Reading the table.** Note the pattern: in every failure domain, the *workable* shape is the same — **assisted, not autonomous**. The domain's failure conditions are not soluble by a better model; they are soluble only by moving the agent to the copilot side of the spectrum (§7). That is the boundary in its most practical form: *the fix for a failing domain is usually to change the agent's role, not to improve the agent.*

### 6.6 Is the Boundary Permanent? The Moving Line

(**synthesis**) The failure domains are not *permanently* off-limits — the boundary moves — but it moves more slowly and more conditionally than the marketing suggests. Three forces shift it:

1. **Model reliability.** As per-step reliability rises, the compounding math (§4.5) gives back some of the long-horizon territory. The GAIA trajectory illustrates the direction: published assistants moved from well under 15% at launch toward much higher scores by 2025-2026 ([autonomous_agents_guide.md](autonomous_agents_guide.md) §6) — but the gains concentrated on *bounded* tasks, precisely the ones already on the "works" side. Unbounded scope (§6.1) is not fixed by a better model, because the problem is the absence of a stopping rule, not the quality of the steps.
2. **Verification technology.** Better checkers — deterministic validators, guard models ([llm_guard_models_guide.md](llm_guard_models_guide.md)), formal-style verification for code — *borrow* checkability for tasks that lack it (§1.2's "checkability can be borrowed"). This is the most promising shifter, and it is why §2.4's verifier pattern is the highest-leverage design investment.
3. **Regulation and trust.** The EU AI Act's high-risk classification, fair-lending rules, and medical-device regulation have already *fixed* the location of the boundary for §6.3-6.4: assist-only, until the rules change. Regulation moves the line left (less autonomy), not right.

**Verdict.** Expect the 2026-2028 movement to be *within* the assisted region — copilots gaining scope and speed — rather than a mass migration into full autonomy for the §6 domains. The "agentic where it pays" consensus (see §7.4) is exactly this: autonomy expands where checkability grows, and stalls where it doesn't.

---

## 7. The Boundary Spectrum

### 7.1 The Spectrum: Deterministic Automation → Assisted → Autonomous

**(synthesis — this guide's organizing frame, built from the verified material above)** The boundary between "works" and "falls apart" is not a line; it is a **three-region spectrum** of how much decision authority you grant the machine:

1. **Deterministic automation** — rules, scripts, RPA. The machine executes a fixed path; no LLM, no judgment, no divergence. Maximum reliability, minimum flexibility. This is the default choice for anything that *can* be expressed deterministically — the "deterministic cores" principle of [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) §2 says: put it here if you can.
2. **Assisted (copilots)** — the machine proposes, the human disposes. The LLM drafts, retrieves, summarizes, suggests; the human reviews, edits, approves, decides. Judgment stays with the human; the machine's value is speed and coverage. This region *contains* most successful production "agents": §2.2's workflow-embedded, §2.3's HITL, §2.4's verifier-augmented are all copilot-shaped or copilot-gated.
3. **Autonomous agents** — the machine receives a goal, plans, acts, and completes the loop with minimal human involvement. This region is *smaller than the marketing*: it is populated only where §1's conditions hold strongly — bounded, verifiable, deterministic feedback, low blast radius — and where the pattern is single-purpose (§2.1) with verifier and budget guards. (The autonomy spectrum's architecture-side taxonomy is [autonomous_agents_guide.md](autonomous_agents_guide.md) §2.)

The critical insight: **the regions are separated by the boundary markers, not by the model.** The same GPT/Claude-class model sits in all three regions; what changes is the *control envelope* — how much of the loop is checked, by what, and who owns the final action.

### 7.2 The Boundary Markers: Checkability, Reversibility, Stakes

**(verified → synthesis)** Three markers place any task on the spectrum. They are the §1/§4 conditions compressed into three questions:

- **Checkability — can a checker verify the output?** (Tests? Schema? Ground truth? Human with a checklist?) High checkability pushes toward autonomy; low checkability forces the human into the loop — there is no machine gate where no check exists (§1.2, §4.2).
- **Reversibility — can the action be undone?** Reversible actions (drafts, read-only queries, proposals) tolerate autonomy; irreversible actions (payments, publications, deletions, binding decisions) require a gate (§1.4, §4.4).
- **Stakes — what does a failure cost?** Time? Money? Reputation? Regulatory action? A life? Stakes set the *acceptable error rate*; the agent's actual error rate must clear that bar, and where it cannot, the gate moves in (§4.4, §6.3-6.4).

The three markers interact multiplicatively: a task with high checkability *and* reversibility *and* low stakes is the ideal autonomous pocket (e.g., code suggestions: tests check, diffs revert, a failed suggestion costs minutes). A task with any marker failing badly is pushed to assisted or deterministic. **The markers are the decision framework in miniature** — §8 expands them into the full tree.

### 7.3 The Spectrum Table

| Mode | Characteristics | General examples | Banking examples |
|---|---|---|---|
| **Deterministic automation** | Fixed path, rules/scripts/RPA; no LLM judgment; max reliability, min flexibility | ETL jobs, straight-through processing, rule-based validations | Statement reconciliation rules; ISO 20022 message validation ([../../banking/iso_20022_core_processes_guide.md](../../banking/iso_20022_core_processes_guide.md)); posting-engine arithmetic ([../../banking/posting_engine_core_banking_guide.md](../../banking/posting_engine_core_banking_guide.md)) |
| **Assisted (copilot)** | Machine proposes, human decides; HITL gates; verifier checks; judgment stays human | Draft-and-review documents; retrieval summaries; triage suggestions; code review assistants | Compliance-monitoring copilot: agent flags suspicious patterns, compliance officer decides (see [autonomous_agents_guide.md](autonomous_agents_guide.md) §9 and §9.3 here); credit-decision recommendation with human underwriter (§9.2); KYC extraction with human verification |
| **Autonomous agent** | Goal-in, result-out within a bounded, verifiable, low-blast-radius envelope; budget + verifier + (often) gate | Support ticket triage; doc extraction; test-driven code fixes; alert enrichment | Customer-support triage agent (§9.1) — classify/retrieve/route within ticket system, human queue behind; trade-confirmation extraction with schema validation |

**Reading the table.** Notice that banking's autonomous column is *thin* — and deliberately so. Banking's combination of irreversibility, regulation, and accountability pushes nearly everything to the assisted region; the autonomous pockets that exist (triage, extraction) are precisely the low-stakes, high-checkability tasks. This is not a banking-specific limitation; it is the boundary markers doing their job. (§8's framework produces exactly this distribution.)

### 7.4 What Moves the Boundary

(**synthesis**) The spectrum positions are not fixed; three forces shift tasks between regions, and understanding them is what separates a static diagram from a working map:

1. **Reliability gains move the autonomous frontier right.** As per-step p rises (better models, better scaffolds, better tooling), the compounding math (§4.5) tolerates longer horizons — but only for tasks that already satisfy checkability. Reliability buys *reach*, not *checkability*; an unverifiable task stays in the assisted region no matter how good the model gets.
2. **Verification technology borrows checkability.** Deterministic validators, guard models ([llm_guard_models_guide.md](llm_guard_models_guide.md)), citation checkers, and test suites are the single most effective boundary-shifters: they convert "we can't check this" into "we can check this," which is precisely the §1.2 condition. Every task moved from assisted to autonomous in the last two years moved because a checker appeared, not because a model got smarter.
3. **Regulation and trust move the line left for high-stakes tasks.** The EU AI Act's high-risk classification and fair-lending rules have frozen loan decisions and clinical decisions in the assisted region regardless of capability (§6.6). Organizational trust moves the same direction: a bank's first agent incident moves every pending candidate one region left for a year.

(**synthesis, verified direction**) The 2025-2026 industry consensus — "**agentic where it pays**" — is the boundary's market echo: investment and production deployments concentrate in the regions where the markers already hold (coding, support triage, extraction), while general-purpose autonomy remains dominated by pilots and demos. The Anthropic 2026 *State of AI Agents* survey's headline finding (near-universal AI-assisted coding) is a *copilot* adoption story; the celebrated support-triage deployments are *gated* agents. **(flag)** Market-size forecasts for "agentic AI" (e.g., Gartner's multi-hundred-billion-dollar agent-software spend figures) are vendor-side projections and should not be read as production-reliability evidence.

### 7.5 The Copilot Default

(**synthesis**) The pragmatic operating rule that follows from the spectrum: **default to assisted; earn autonomy.** Ship the copilot shape first — the agent proposes, the human disposes — with verifiers and evals in place, measure the real error rate, and only then widen the agent's authority region by region as the data justifies it. Autonomy is granted by evidence, not by ambition. This is not the cautious path; it is the fast path — the copilot generates the exemplars (§1.5), the staffing model (§8.2 Q6), and the telemetry (§5.5) that an autonomous deployment would need anyway, and it does so while the blast radius is still contained.

### 7.6 One Task, Three Regions — Invoice Processing

(**synthesis**) The spectrum is not a taxonomy of tasks; it is a set of *choices within a task*. The same invoice-processing task illustrates all three regions:

- **Deterministic:** standard invoices matching templates are processed by rules — field mapping, totals check, straight through. No LLM needed; this is the deterministic core ([production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) §2).
- **Assisted:** non-standard invoices (foreign layouts, scanned handwriting, unusual line structures) go to an extraction agent that drafts the structured record; a human verifies and releases it. The agent handles volume, the human handles judgment — §2.3's gate at the point of consequence.
- **Autonomous (justified):** if and only when schema validation and reconciliation checks are airtight — every field checked against the source, totals reconciled against the purchase order — the extraction step can run without per-item review, because the *verifier* has replaced the *reviewer* (§2.4). Autonomy here is not a modeling achievement; it is a verification achievement.

The point: **the same task moves between regions as checkability, reversibility, and stakes change** (§7.2) — and the smart design builds all three layers at once, routing each invoice to the cheapest region that can handle it safely. That routing *is* the boundary, operationalized.

---
## 8. The Decision Framework: Should This Be an Agent?

### 8.1 The Decision Tree

**(synthesis — this guide's framework, built from the verified conditions of §1/§4 and the markers of §7.2)** Before building anything, run the task through the tree. Each node is a yes/no question whose "no" answer redirects you to a cheaper mode — this is the entire value of the framework: **it tells you what *not* to build.**

```
START: A task you want to automate with an LLM.

Q1. Is the task space BOUNDED?
    (Can you write acceptance criteria in one paragraph?
     Would two humans agree on "done"?)
    │
    ├── NO ──► STOP. Not an agent.
    │          Narrow the scope first (split the task, fix the goal,
    │          constrain the input). Re-run the tree on the narrowed task.
    │          (Failure condition §4.1; narrow-scope principle,
    │           production_ready_llm_agents_guide.md)
    │
    └── YES ──► Q2. Are the outputs VERIFIABLE?
                (Test? Schema? Ground truth? Human checklist?)
                │
                ├── NO ──► BUILD A COPILOT, not an agent.
                │          Machine proposes, human checks every output.
                │          (Unverifiable = silent confidence §5.4;
                │           copilot = assisted region §7.1)
                │
                └── YES ──► Q3. Is the FEEDBACK deterministic and in-loop?
                            (Tool returns? Test pass/fail? Immediate?)
                            │
                            ├── NO ──► BUILD A COPILOT or a workflow
                            │          with human judgment per step.
                            │          (Delayed feedback §4.3; proxy
                            │          signals only if they steer the run)
                            │
                            └── YES ──► Q4. Is the BLAST RADIUS low?
                                        (Reversible? Contained?
                                         Regulated? Irreversible?)
                                        │
                                        ├── NO ──► BUILD WITH A HUMAN GATE
                                        │          (HITL §2.3; agent proposes,
                                        │           human approves execution;
                                        │           keep the agent in the
                                        │           assisted region)
                                        │
                                        └── YES ──► Q5. Do you have ENOUGH
                                                    EXEMPLARS for an eval set?
                                                    │
                                                    ├── NO ──► Pilot as copilot,
                                                    │          collect labels,
                                                    │          re-run when data exists.
                                                    │          (§1.5)
                                                    │
                                                    └── YES ──► BUILD THE AGENT
                                                                — single-purpose
                                                                (§2.1), verifier
                                                                (§2.4), loop
                                                                budget, eval gates.
                                                                (production_ready_
                                                                llm_agents_guide.md)
```

**Reading the tree.** Every "no" is a redirect, not a dead end — it lands you one region to the left on the §7 spectrum (autonomous → assisted → deterministic). The tree's bias is deliberate: it is cheaper to build a copilot and discover you could have gone autonomous than to build autonomous and discover you can't check it. The tree is also a *step-count* machine: tasks that pass all five nodes are precisely the low-N, high-checkability tasks where p^N stays acceptable (§4.5).

### 8.2 The Framework Questions

**(verified → synthesis)** The tree operationalizes five questions; a sixth covers the human layer. The first three trace to Anthropic's *Building Effective Agents* guidance ("use a workflow when you can predict the path; use an agent when steps are unpredictable but the goal is concrete; agents are best for open-ended problems **where you can verify their work**") and to the checkability/feedback conditions that every agent-eval methodology assumes ([llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md)); the blast-radius and oversight questions trace to the responsible-AI and platform literature ([implementing-responsible-ai.md](implementing-responsible-ai.md), [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md)).

1. **Is the task bounded?** Can "done" be defined in one paragraph that two humans would agree on?
2. **Are the outputs verifiable?** Does a checker exist — test, schema, ground truth, or human-with-checklist — that can score the output?
3. **Is the feedback deterministic and in-loop?** Does the agent get an unambiguous signal *during* the run that it can act on?
4. **Is the blast radius low?** Is a wrong action reversible, contained, unregulated, cheap?
5. **Do we have enough exemplars?** Is there historical data to build an eval set and a regression harness?
6. **(The human layer) Can a human be in the loop where it matters?** Is there a gate, a reviewer, an escalation path — *and a team that will actually staff it?*

**(verified)** Question 6 carries the least-discussed failure mode in the literature: the **unstaffed gate**. Many agent programs fail not because the agent is bad but because the approval queue is nobody's job — the gate exists on paper and is rubber-stamped or ignored in practice. Anthropic's and OpenAI's agent guidance both stress that confirmation steps only work if a human is genuinely available; the failure guide's operations section ([llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) §4) makes the same point from the observability side. **A gate without a staffing model is a liability, not a control.**

### 8.3 The Scoring: The Agent-Appropriateness Score

**(flag — heuristic, not canonical)** As a compact decision aid, the six questions can be scored 0-2 each (0 = condition absent, 1 = partially/emerging, 2 = clearly met):

- **10-12 (≥ ~80%):** **Autonomous agent** — build it (§8.1's full green path), single-purpose with verifier and budget.
- **7-9:** **Assisted** — copilot or gated agent; human owns the consequential steps.
- **0-6:** **Not an agent** — deterministic automation or workflow with LLM steps; revisit only if the task is reshaped.

**(flag)** **Verification:** this scoring is this guide's heuristic synthesis — there is no canonical published "agent-appropriateness score" to cite; it is a way to make the tree's qualitative verdicts comparable across a portfolio, not a measurement. Use it to *rank* candidate tasks against each other (which of these ten candidates deserves the engineering investment first?), not as a pass/fail oracle. The "70% rule" (§4.1) is similarly flagged: folk heuristics are useful as conversation starters and dangerous as gates.

### 8.4 Using the Framework Across a Portfolio

(**synthesis**) The framework's real use is not single-task analysis — it is **portfolio triage**: ranking a backlog of candidate automations so that engineering effort goes to the tasks most likely to clear the boundary. The workflow:

1. **Score every candidate** with the §8.3 heuristic (six questions, 0-2 each). This takes an hour per task with the business owner in the room — and the *discussion* is the deliverable, because it forces the owner to answer the checkability questions they have been avoiding.
2. **Sequence by score band.** Run the 10-12 band first (autonomous pockets: fast, contained wins — they build organizational trust and eval infrastructure). Then the 7-9 band as copilots. Put the 0-6 band on an explicit *not-now* list with the blocking condition written next to each ("needs a verifier we don't have", "needs three months of ticket history", "needs a staffing model").
3. **Re-score quarterly.** The boundary moves (§7.4): a task that failed "verifiable" in Q1 may pass in Q3 because a checker appeared; a task that passed "blast radius" may fail after a new regulation. The re-score is the governance mechanism that keeps the portfolio honest.

| Illustrative portfolio (banking) | Q1 score | Verdict | Blocking condition / next action |
|---|---|---|---|
| Ticket triage | 12/12 | Autonomous agent | Ship now |
| Trade-confirmation extraction | 11/12 | Autonomous agent | Ship now (worked example §9.4) |
| Compliance memo drafting | 8/12 | Copilot | Staff the review queue first |
| Loan-decision recommendation | 7/12 | Copilot (HITL) | Underwriter staffing model; regulation watch |
| Open-ended market research | 4/12 | Not an agent | Needs bounded questions + retrieval verifier |
| Autonomous loan decisions | 2/12 | Not an agent | Blocked by regulation and blast radius — permanently, at current rules |

**The point of the table:** the framework produces a *work plan*, not a verdict on AI. The same portfolio contains a ship-now agent, two copilots, and two explicit refusals — and every line is explainable by the boundary markers, which is exactly what a solution architect needs when a sponsor asks "why can't we automate this one?"

### 8.6 Common Mistakes in Applying the Framework

(**synthesis**) The framework fails in predictable ways — usually because the questions are answered with optimism instead of evidence. The seven most common mistakes:

1. **Answering "bounded?" with the current implementation.** "We already have a form for this" is not boundedness; the *goal* must be definable independently of how it is done today.
2. **Confusing manual review with verifiability.** "A senior person checks it" is a copilot arrangement, not a verifier — if the check cannot be automated or staffed at volume, the verifiable question must be answered "no" (§1.2, §8.2 Q2).
3. **Scoring blast radius by the expected failure instead of the worst failure.** "It will usually be right" is irrelevant; the marker is what one wrong action costs (§4.4, §7.2).
4. **Deferring exemplars.** "We'll build the eval set later" is how agents ship with no gate — the eval set is the gate (§1.5); without it, the agent is an unverified claim.
5. **Skipping the staffing question.** The gate that is nobody's job is worse than no gate — it manufactures false confidence (§8.2 Q6, §2.6's HITL tell).
6. **Letting the demo override the tree.** "It looked great in the demo" is the exact failure the failure guide's §3 ("works in the demo, fails in production") diagnoses — the tree exists to be run *before* the demo, not after it.
7. **Re-scoring on vendor news.** A model release does not change your task's checkability, reversibility, or stakes (§7.4: reliability buys reach, not checkability). When a vendor announces "agents can now do X," re-run the tree on *your* task with *your* checker — usually nothing moves. 

### 8.5 The Framework Table

| Question | Pass criteria | If it passes | If it fails |
|---|---|---|---|
| **1. Bounded?** | Acceptance criteria writable in one paragraph; humans agree on "done" | Proceed to Q2 | Not an agent — narrow the scope, then re-run |
| **2. Verifiable?** | A checker exists (test/schema/ground truth/checklist) | Proceed to Q3 | Copilot — human checks every output |
| **3. Deterministic feedback?** | Unambiguous in-loop signal each step | Proceed to Q4 | Copilot/workflow — human judgment per step |
| **4. Low blast radius?** | Reversible, contained, unregulated | Proceed to Q5 | Gated agent — HITL on execution (§2.3) |
| **5. Enough exemplars?** | Historical outcomes for an eval set | **Build the agent** (§2.1 + §2.4 + budget) | Pilot as copilot, collect labels, re-run |
| **6. Staffed human layer?** | A real, staffed gate/reviewer exists | Ship with confidence | Fix the staffing before shipping anything |

---

## 9. The Worked Examples: Banking

Three banking use cases, run through the framework. (System context: [../../banking/core_banking_systems_guide.md](../../banking/core_banking_systems_guide.md); risk context: [../../banking/financial_risk_compliance_systems_guide.md](../../banking/financial_risk_compliance_systems_guide.md).)

### 9.1 Customer-Support Triage — The Agent Works

**The task.** Classify an incoming customer query (password reset / balance inquiry / fee dispute / card issue / loan status...), retrieve the relevant account and policy data, answer if the answer is fully determined by policy, otherwise route to the right team with a summary.

**Framework run.** Bounded? Yes — a finite intent taxonomy, and "done" (correctly classified, correctly answered-or-routed) is checkable. Verifiable? Yes — answers must match the KB; routing can be compared against human routing history. Feedback? Yes — resolution/escalation outcomes are recorded, so the loop learns. Blast radius? Low — a wrong answer is a re-queue, and a human agent sits behind the queue. Exemplars? Yes — the ticket archive is a golden set. Human layer? Yes — escalation queue staffed by the existing support org.

**Verdict: build the agent** — single-purpose (§2.1), workflow-embedded in the ticketing system (§2.2), verifier on retrieval citations (§2.4), escalation gate to humans (§2.3). This is the domain with the strongest production evidence base (§3.1) — including banking's own τ-bench results, which show the shape is right and the remaining gap is policy adherence at long horizons, which the escalation gate absorbs.

### 9.2 Loan Decisions — The Agent Falls Apart (HITL Required)

**The task.** Approve or reject a loan application — or set the limit — from applicant data, credit history, income, collateral, and policy.

**Framework run.** Bounded? Partially — the *inputs* are structured, but "done" is not checkable in the sense that matters: there is no ground-truth correct decision, only a fair, documented, regulatorily-sound one. Verifiable? **No** — in the accountability sense; a checker can validate the *process* (was the policy applied? is the explanation complete?) but not the *decision*. Feedback? **Delayed** — default data arrives months later (§4.3), so the agent cannot converge in-episode. Blast radius? **High** — a wrong rejection harms a customer, a wrong approval loses money, and fair-lending law forbids disparate outcomes (§4.4, §6.3). Exemplars? Plenty of historical decisions — but they encode the *bank's* decisions, not ground truth. Human layer? Mandatory under regulation (EU AI Act high-risk; fair-lending accountability).

**Verdict: the agent falls apart; the copilot works.** Do not build an autonomous decision agent. Build the **assist shape**: agentic pipeline assembles the credit file, extracts, summarizes, applies policy rules, and produces a recommendation with a full explanation trail; the **human underwriter decides**. The agent's step count is bounded (pipeline with deterministic checks, §2.2), its output is verifiable (recommendation checked against policy rules), and its blast radius is collapsed by the gate (§2.3). Every "no" in the framework run is a redirect — and the redirect lands exactly here.

### 9.3 Compliance Monitoring — The Assisted (Copilot)

**The task.** Monitor transactions and behaviors for patterns that suggest money laundering, sanctions exposure, or policy breaches; investigate and escalate.

**Framework run.** Bounded? The *pattern space* is bounded (typologies are enumerable) but the *investigation* is open. Verifiable? Partially — alert quality is measurable (precision/recall against confirmed cases), but the *judgment* (is this genuinely suspicious?) is not machine-checkable. Feedback? Partially — confirmed/rejected alerts give the loop labels, but with latency (investigations take days). Blast radius? **High** — a missed SAR (suspicious activity report) is a regulatory failure; a false positive is a customer-relationship cost; both are expensive. Exemplars? Yes — years of confirmed/unconfirmed alert history. Human layer? Mandatory — the compliance officer owns the SAR decision by law and by [../../banking/financial_risk_compliance_systems_guide.md](../../banking/financial_risk_compliance_systems_guide.md) operating models.

**Verdict: the assisted copilot** — and this matches the series' own worked example in [autonomous_agents_guide.md](autonomous_agents_guide.md) §9 (a banking compliance *research* agent). The agent does the bounded, checkable work: triage the alert queue, retrieve account and transaction context, draft the investigation memo, rank candidates by typology match — with verifiers (the typology rules) and retrieval citations. The compliance officer reviews, amends, and owns the disposition. The copilot shape converts the high-blast-radius parts into reviewed drafts and keeps the agent inside the checkable loop. It is not "less ambitious" than an autonomous agent; it is the only shape that satisfies the boundary markers.

### 9.4 Trade-Confirmation Extraction — The Quiet Autonomous Pocket

**The task.** Extract trade-confirmation fields (counterparty, instrument, notional, value date, rates) from incoming confirmations into the booking schema; validate against the source and the trade record.

**Framework run.** Bounded? Yes — the schema defines "done" field by field. Verifiable? Yes — every field is checkable against the source document, and the extracted record must reconcile against the trade data ([../../banking/posting_engine_core_banking_guide.md](../../banking/posting_engine_core_banking_guide.md) for the record context). Feedback? Yes — schema validation and reconciliation are deterministic, in-loop. Blast radius? Low — extraction errors surface as validation failures before booking, and a human verifier reviews before release. Exemplars? Yes — every historical confirmation is a labeled example. Human layer? Yes — the existing ops verification team reviews flagged cases.

**Verdict: the agent works** — and quietly: this is the shape of the "extraction agents that nobody writes blog posts about" (§3.4). Single-purpose, schema-verified, embedded in the booking workflow. The autonomous pocket exists precisely because every marker passes; it is the banking analogue of the code-suggestion agent — bounded, checked, and cheap to be wrong.

### 9.5 Real-Time Anti-Fraud Decisioning — Deterministic Core, Agent Assist

**The task.** Decide, in milliseconds, whether an incoming transaction is fraudulent and should be declined.

**Framework run.** The *decision* fails the framework outright: blast radius is high (a wrong decline is a customer incident; a wrong approval is a fraud loss — [../../banking/financial_risk_compliance_systems_guide.md](../../banking/financial_risk_compliance_systems_guide.md)), feedback is delayed (fraud confirmation arrives in dispute reports days later), and latency forbids a human gate per transaction. But note what passes: the *core* is deterministic — rule engines and calibrated ML scorecards have decided fraud in milliseconds for a decade, with the "deterministic cores" principle ([production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) §2) already in production. The agent's role is the *long tail*: enrich the case file, draft the investigation summary, retrieve the customer's behavior history, propose next steps for the fraud team — the assisted shape (§7.1) on top of the deterministic core.

**Verdict: deterministic core + agent assist — never an autonomous agent.** The decision stays in the deterministic engine (with human review on declines), and the agent works the evidence trail around it. This worked example shows the boundary's full spectrum in one domain: deterministic automation (the decision), assisted (the casework), and a deliberate *no* to autonomy — because the markers, not the model, decide.

| Use case | Boundary assessment (framework run) | Verdict |

### 9.6 The Example Table
|---|---|---|
| **Customer-support triage** | Bounded ✓ · Verifiable ✓ · Feedback ✓ · Blast radius low ✓ · Exemplars ✓ · Staffed human queue ✓ | **Agent works** — build it (single-purpose, workflow-embedded, escalation gate) |
| **Loan decisions** | Bounded ~ · Verifiable ✗ · Feedback ✗ (delayed) · Blast radius high ✗ · Exemplars ~ · Human layer mandatory | **Agent falls apart** — HITL required; build the recommendation copilot, human decides |
| **Compliance monitoring** | Bounded ~ · Verifiable ~ · Feedback ~ · Blast radius high ✗ · Exemplars ✓ · Human layer mandatory | **Assisted** — copilot: agent triages/drafts, compliance officer owns the disposition |

| **Trade-confirmation extraction** | Bounded ✓ · Verifiable ✓ · Feedback ✓ · Blast radius low ✓ · Exemplars ✓ · Human layer ✓ | **Agent works** — the quiet autonomous pocket (schema-verified, workflow-embedded) |
| **Real-time anti-fraud decisioning** | Decision: high blast radius ✗ + delayed feedback ✗ → deterministic core; casework: assisted | **Deterministic + assist** — the decision is never an autonomous agent |

**The pattern across all three.** The same model class, the same company, three different verdicts — decided entirely by the boundary markers, not by model capability. That is the framework's point: **agent viability is a property of the task-and-control-envelope pair, not of the model.**

---

## 10. The Summary: The Boundary in One Page

**The thesis.** *Agents work where the loop is checkable.* An agent is a loop — goal, plan, act, observe, repeat. Every loop needs six things to survive contact with production: a **bounded task space** (you can define "done"), **verifiable outputs** (a checker can score them), **deterministic feedback** (the loop learns from an unambiguous signal), **low blast radius** (failures are contained), **sufficient exemplars** (evals have a golden set), and **human oversight** where stakes rise (gates, reviewers, escalation). Where all six hold — support triage, code assistance, document extraction, bounded internal workflows — agents work, and they work best as single-purpose, workflow-embedded, verifier-augmented systems with a human queue behind them.

**The failure side.** Agents fall apart where the loop stops being checkable: **open-ended tasks** (no stopping rule), **unverifiable outputs** (silent confidence), **delayed feedback** (blind runs), **high blast radius** (cheap mistakes become expensive ones), and **long horizons** (0.95¹⁰ ≈ 0.60 — reliability decays exponentially in step count; see [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md)). The characteristic death patterns follow from those conditions: drift on long missions, multi-agent communication overhead, runaway tool loops, and wrong-but-confident output — and the domains that combine the conditions (open-ended research, unregulated real-world actions, high-stakes financial decisions, medical decisions) fail structurally, not because of bad implementation.

**The spectrum.** Deterministic automation → assisted (copilot) → autonomous agent. The regions are separated by three markers — **checkability, reversibility, stakes** — not by model quality. The same model that safely triages tickets must be a gated copilot for loan decisions.

**The framework.** Run the decision tree before building: *bounded? → verifiable? → feedback? → blast radius? → exemplars? → staffed human layer?* Every "no" redirects you one region left on the spectrum — and the redirect is a win, because a copilot that ships beats an autonomous agent that spins.

**The boundary at a glance.**

| The "works" side | The "falls apart" side |
|---|---|
| Bounded task space — "done" is definable | Open-ended task — no stopping rule (§4.1) |
| Verifiable outputs — a checker exists | Unverifiable outputs — no ground truth (§4.2) |
| Deterministic feedback — the loop learns | Delayed feedback — the loop runs blind (§4.3) |
| Low blast radius — errors are cheap | High blast radius — errors are expensive (§4.4) |
| Short horizons — p^N stays high | Long horizons — 0.95¹⁰ ≈ 0.60 (§4.5) |
| Sufficient exemplars — evals exist | No history — nothing to gate against (§1.5) |
| Staffed human gates where stakes rise | Unstaffed or missing gates (§8.2 Q6) |

**Decide with:** the three markers (checkability, reversibility, stakes — §7.2) and the five-question tree (bounded → verifiable → feedback → blast radius → exemplars — §8.1). **Deploy with:** the four patterns (single-purpose, workflow-embedded, HITL, verifier — §2). **Monitor with:** the observability tells (§5.5) and drift baselines ([ai_agent_drift_guide.md](ai_agent_drift_guide.md)).

**The final word.** The boundary is not a criticism of agents; it is the map of where they pay. **Put agents where their errors are cheap, visible, and correctable — and put humans where they are not.** The organizations winning with agents in 2025-2026 are not the ones with the most autonomous systems; they are the ones that draw this line honestly, ship the checkable loops, and staff the gates. Where the loop is checkable, delegate. Where it is not, assist. That distinction *is* the boundary — and it is the whole job.

---

## 11. Glossary

| Term | Definition |
|---|---|
| **Agent-appropriateness** | The degree to which a task satisfies the conditions for agent viability (bounded, verifiable, feedback, blast radius, exemplars, oversight); measured by the §8 framework. |
| **Autonomous agent** | A system that receives a goal and plans, acts, and completes the loop with minimal human involvement; viable only within a bounded, verifiable, low-blast-radius envelope (§7.1). |
| **Blast radius** | The cost and reach of a single agent failure — reversibility, containment, regulation, and reputational exposure combined (§1.4, §4.4). |
| **Bounded task space** | A task whose input space and acceptance criteria are definable enough that "done" is agreed by two competent humans (§1.1). |
| **Checkability** | The property that a checker exists and can verify an output — the first boundary marker (§7.2). |
| **Compounding error** | The exponential decay of task reliability in step count: 0.95¹⁰ ≈ 0.60; the core structural limit of long-horizon agents (§4.5; full treatment in [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md)). |
| **Compliance monitoring** | The regulated practice of watching transactions/behaviors for suspicious patterns; in this guide, the canonical *assisted* banking use case (§9.3). |
| **Copilot** | The assisted mode: the machine proposes and the human disposes — the dominant viable shape for high-stakes tasks (§7.1). |
| **Decision tree** | The §8.1 framework: bounded? → verifiable? → feedback? → blast radius? → exemplars? → human layer? — with a redirect on every "no." |
| **Delayed feedback** | A signal that arrives after the agent's loop has closed, so the agent cannot converge within the episode (§4.3). |
| **Deterministic feedback** | An unambiguous, in-loop signal (test pass/fail, tool return, schema result) that the agent can act on immediately (§1.3). |
| **Drift** | The gradual degradation of an agent's behavior over time (goal, input, tool, output) — the signature failure of long-horizon autonomy (§5.1; full treatment in [ai_agent_drift_guide.md](ai_agent_drift_guide.md)). |
| **HITL / Human-in-the-loop** | The operating mode in which a human reviews, approves, or decides at defined points — the blast-radius reducer and the mandated shape for regulated decisions (§1.6, §2.3). |
| **Human oversight** | The sixth success condition: staffed gates, reviewers, and escalation paths (§1.6). |
| **Long-horizon** | Tasks requiring many steps over an extended run; structurally penalized by compounding error and drift (§4.5, §5.1). |
| **Loan decision** | The canonical *falls-apart* banking use case: high stakes, delayed feedback, unverifiable in the accountability sense — HITL required (§6.3, §9.2). |
| **Multi-agent** | Designs with multiple communicating agents; the overhead of handoffs adds steps and error surface — the failure pattern of §5.2 ([hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md)). |
| **Narrow scope** | The design principle that an agent's mission must be a single, well-defined task type — the operational form of the bounded task space ([production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md)). |
| **Open-ended task** | A task with no definable acceptance criteria or stopping rule — the first failure condition (§4.1). |
| **Reversibility** | Whether a wrong action can be undone — the second boundary marker (§7.2). |
| **Silent confidence** | Wrong-but-confident output with no internal signal of error; the failure pattern that arises wherever checkability is absent (§5.4). |
| **Single-purpose** | The pattern of one agent, one task, one minimal toolset — the lowest-step-count design (§2.1). |
| **Spectrum** | The deterministic → assisted → autonomous continuum of machine decision authority; the boundary's organizing frame (§7). |
| **Stakes** | The cost of failure — time, money, reputation, regulation, life — which sets the acceptable error rate (§7.2). |
| **Stopping rule / budget** | The hard cap on steps, cost, or time that every agent loop needs; the missing piece in tool-loop failures (§5.3; [agentops_guide.md](agentops_guide.md)). |
| **Triage** | The bounded, verifiable, low-stakes task of classifying and routing — the canonical *agent-works* banking use case (§3.1, §9.1). |
| **Tool loop** | The runaway pattern of repeated tool calls without progress, caused by missing budgets and ambiguous results (§5.3). |
| **Unverifiable** | The property of outputs with no ground truth or checker — the second failure condition (§4.2). |
| **Verifier** | The second-pass checker (tests, rules, second model, guard) that validates an agent's output — the pattern of §2.4. |
| **Verifiable output** | An output that a checker can objectively score against a standard (§1.2). |
| **Workflow-embedded** | The pattern of an agent as one step inside a deterministic pipeline whose shell provides checks and error handling (§2.2). |

---

*This guide is part of the agents series of the AI/LLM Engineering Guides

---

## 12. References & Further Reading

**Primary sources cited in this guide**

1. Anthropic. *Building Effective Agents.* December 2024 — the workflows-vs-agents distinction and the five workflow patterns; the canonical "when to use an agent" guidance (§1.1, §2.2, §2.4, §8.2).
2. Yao, S., et al. (Sierra). *τ-bench: A Benchmark for Tool-Agent-User Interaction in Real-World Domains.* arXiv:2406.12045, June 2024 — tool-agent-user benchmark with retail, airline, and banking domains and pass^k reliability measurement (§3.1; full treatment in [autonomous_agents_guide.md](autonomous_agents_guide.md) §6).
3. Mialon, G., et al. (Meta AI / Hugging Face). *GAIA: A Benchmark for General AI Assistants.* arXiv:2311.12983, November 2023 — general-assistant benchmark; humans ~100%, assistants far lower at publication (§6.1).
4. Anthropic. *The 2026 State of AI Agents Report.* (**(flag)** vendor survey; directional — ~90% of surveyed organizations report AI-assisted coding) (§3.2, §7.4).
5. Air Canada chatbot tribunal ruling (2024) — airline held liable for chatbot-fabricated refund policy (**(flag)** media-reported legal outcome) (§5.4).
6. DPD chatbot incident (January 2024) — customer-prompted profanity, widely reported (**(flag)**) (§5.4, §6.2).
7. NYC MyCity chatbot (2024) — incorrect legal guidance to business owners, reported by press (**(flag)**) (§5.4).
8. Klarna AI assistant and Intercom Fin published metrics — two-thirds-of-chats and ~51% resolution figures respectively (**(flag)** vendor-reported, contested/partial; use as directional only) (§3.1).
9. Vellum, *State of GenAI* builder survey (2025) — widely cited finding that a single-digit percentage of gen-AI pilots reached production (**(flag)** approximate, self-reported sample) (§7.4 context).
10. Gartner agentic-AI market forecasts (2025-2026) — vendor-side projections, not reliability evidence (**(flag)**) (§7.4).

**Series companions (this repository)**

- [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) — the failure catalogue and the compounding-error math (0.95¹⁰ ≈ 0.60)
- [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) — the production playbook (narrow scope, deterministic cores, verifiers, eval gates)
- [autonomous_agents_guide.md](autonomous_agents_guide.md) — architectures, the autonomy spectrum, agent evaluation
- [llm_agent_use_cases.md](llm_agent_use_cases.md) — the use-case catalogue with company-level evidence
- [ai_agent_drift_guide.md](ai_agent_drift_guide.md) — drift over time; detection and mitigation
- [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md) — multi-agent orchestration patterns and costs
- [agentops_guide.md](agentops_guide.md) — observability, budgets, alerts
- [agent_scaffolding_guide.md](agent_scaffolding_guide.md) — the loop scaffold and stop conditions
- [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) — gateways, governance, FinOps
- [rag/production_grade_agentic_rag_guide.md](rag/production_grade_agentic_rag_guide.md) — agentic-RAG viability
- [implementing-responsible-ai.md](implementing-responsible-ai.md) — HITL, approvals, audit
- [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md) — eval design

*This guide is part of the agents series of the AI/LLM Engineering Guides ([research](https://github.com/jackliusr/research)). Boundary companions: [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) (why agents fail), [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) (how to build them), [llm_agent_use_cases.md](llm_agent_use_cases.md) (where they're deployed). Verification flags: (verified) = established in published guidance/incidents; (emerging) = consolidating practice; (synthesis) = this guide's framework; (flag) = circulating heuristic without a canonical source.*
