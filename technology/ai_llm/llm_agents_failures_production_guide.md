# Why Most LLM Agents Fail in Production

> **A Structural Failure Analysis of Production LLM Agents — the Compounding-Error Problem, the Failure Taxonomy, the Reliability/Cost/Latency Realities, the Known Discourse, the Success Factors, the Mitigations, and the Production-Readiness Checklist**

**Author:** Jack Liu Shurui  
**Series:** LLM/AI Advanced Topics — *Agents Series (Structural Failure Analysis)*  
**Domain:** AI Engineering · Agent Reliability · Enterprise AI  
**Reading time:** ~60 minutes  
**Version:** 1.0 — August 2026

> **Series positioning.** This guide is the **structural failure analysis** for the agents series: *why* production LLM agents fail, *how* the failure modes compound, and *what* the successful production agents do differently. It is the structural counterpart to [ai_agent_drift_guide.md](ai_agent_drift_guide.md), which covers the **temporal** dimension (degradation *over time*): drift is what happens to an agent *between* Tuesday and Friday; this guide is about the failure modes that are present *on day one* — baked into the architecture, the eval, and the operating model. The umbrella reference for the series is [autonomous_agents_guide.md](autonomous_agents_guide.md); the code-level scaffold is [agent_scaffolding_guide.md](agent_scaffolding_guide.md); the platform that operationalizes reliability is [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md).

> **How this guide is organized.** The argument runs in one direction: §1 establishes the *math* that makes long-horizon agents fragile (the compounding-error problem); §2 names the *failure modes* that math produces (the taxonomy); §3–§5 show why the failures appear in production and not in the demo (evaluation, reliability, cost/latency); §6 surveys the *known discourse* and its data; §7–§8 give the *counterpoint and the playbook* (success factors, mitigations, patterns); §9 walks a *worked example* (a bank's customer-support agent); §10 is the *production-readiness checklist*; §11–§12 are the glossary and references. Readers in a hurry: read §1.1, §2.5, §6.5, and §10, then return to the rest.

---

## Table of Contents

1. [The Compounding-Error Problem](#1-the-compounding-error-problem)
2. [The Failure Taxonomy](#2-the-failure-taxonomy)
3. [Works in the Demo, Fails in Production](#3-works-in-the-demo-fails-in-production)
4. [Reliability and Observability](#4-reliability-and-observability)
5. [Cost and Latency](#5-cost-and-latency)
6. [The Known Discourse](#6-the-known-discourse)
7. [The Success Factors: What Production Agents Do Right](#7-the-success-factors-what-production-agents-do-right)
8. [Mitigations and Production Patterns](#8-mitigations-and-production-patterns)
9. [Worked Example: A Bank's Customer-Support Agent](#9-worked-example-a-banks-customer-support-agent)
10. [The Production-Readiness Checklist](#10-the-production-readiness-checklist)
11. [Glossary](#11-glossary)
12. [References & Further Reading](#12-references--further-reading)

---

## 1. The Compounding-Error Problem

### 1.1 The Core Math: Per-Step Errors Compound

The single most important number in agent engineering is not any benchmark score — it is the product rule of independent step reliability, known in reliability engineering as **Lusser's Law** (Robert Lusser, 1957): *the reliability of a series system is the product of the reliabilities of its components.*

An LLM agent is a series system. It executes a chain of steps — parse the request, choose a tool, format the arguments, call the API, interpret the result, decide the next action — and **every step carries a probability of being wrong**. If the steps were independent, the probability that the *whole task* succeeds is the product of the per-step probabilities:

> **p_task = p₁ × p₂ × … × p_N = p^N** (for uniform per-step success probability p over N steps)

The classic framing, used across practitioner writing in 2025–2026:

> **A 95%-per-step success rate over 10 steps gives roughly 60% overall success.**

**Verified arithmetic:** 0.95¹⁰ = 0.5987. The framing is exact where it counts: 0.95¹⁰ ≈ 0.5987 ≈ 60%. Extend it and the picture darkens fast:

- 0.95²⁰ ≈ 0.3585 — a 95%-reliable agent finishes a 20-step task only about **36%** of the time.
- 0.90¹⁰ ≈ 0.3487 — a 90%-per-step agent is down to **~35%** over 10 steps.
- 0.99¹⁰ ≈ 0.9044 — a 99%-per-step agent holds **~90%** over 10 steps, but decays to ~61% by step 50.

> **Verification note.** The arithmetic is exact and verified. The *assumption* of uniform, independent steps is a model, not reality — real steps are correlated (a wrong step often poisons the next), which makes the product rule a **best case** for well-behaved independent steps. If errors are positively correlated, actual end-to-end reliability is *worse* than p^N, not better. Treat the product rule as the optimistic bound.

### 1.2 The Math: Exponential Decay

The success rate decays **exponentially** in the number of steps, because it is a power function of p:

- The **per-step error rate** is e = 1 − p. A 95% step has a 5% error rate.
- The **task error rate** is 1 − p^N ≈ N·e for small e·N — errors *add* linearly in the exponent and compound multiplicatively in the failure probability.
- Doubling the steps squares the survival probability: going from 10 to 20 steps at 95% drops 60% → 36%, i.e. roughly (0.60)².

Two structural consequences follow directly from the exponent:

1. **Per-step reliability is worth more than raw model quality.** Raising p from 0.95 to 0.99 (a 4-point improvement in per-step accuracy) more than doubles a 10-step task's success rate (0.60 → 0.90). A model that is *slightly* more reliable at each step is *dramatically* more reliable over a horizon.
2. **Step count is the leverage point you control.** You cannot easily move p by 4 points, but you *can* cut the step count. Removing 5 steps from a 15-step chain at 95% per-step takes the task from 0.95¹⁵ ≈ 46% to 0.95¹⁰ ≈ 60%.

### 1.3 Why It Matters: Long-Horizon Agents

The compounding problem is why the *impressive demos* are short. Research agents, deep-analysis agents, and multi-step workflows routinely execute **10–50+ model calls per task**: query planning, retrieval, re-ranking, tool calls, code execution, verification, summarization. The literature and practitioner write-ups consistently find:

- **Agentic coding:** on Terminal-Bench (2025), even strong models solve only a fraction of tasks, and LangChain's trace analysis identified a signature failure pattern: agents that *wrote* a solution, *reviewed their own output*, and **stopped — declaring success without running the tests** (false completion at the final step is still a task failure; see §2.1).
- **Computer use:** a widely reported June 2025 study by **Carnegie Mellon and Anthropic researchers** tracked computer-using agents on thousands of real-world tasks and concluded that agents "make too many mistakes for businesses to rely on them for any process involving real money" — press coverage put the failure rate around **~85%** on the studied tasks (see §6.1; the exact figure varies by task sample and is as-reported).
- **Research agents:** each retrieval + synthesis + verification cycle is 3–5 steps; a 10-cycle research task is a 30–50-step chain where a 95% per-step agent is mathematically near-certain to fail *somewhere* — and a single wrong fact or wrong citation in the final report is a task failure.

The error accumulation is not hypothetical: it is the difference between "the agent works in the notebook" (2–3 steps, 0.95³ ≈ 86%) and "the agent works in production" (15–30 steps, 0.95¹⁵–0.95³⁰ ≈ 46%–21%).

### 1.4 The Implication: Design for Fewer Steps and Higher Per-Step Reliability

The math prescribes the design agenda:

- **Fewer steps.** Prefer single-shot, well-engineered prompts over multi-step improvisation where the task allows; the industry term for the *optimal* structure is the **workflow** — deterministic, code-defined chains — versus the **agent** — model-decided loops. Anthropic's widely cited *Building Effective Agents* essay (December 2024) makes exactly this point: "the simplest solution that works" and "don't build agents when a workflow suffices." The fewer model-decided steps, the less compounding there is to fight.
- **Higher per-step reliability.** Improve the *step*, not just the model: structured tool schemas, argument validation, retrieval quality, verifier steps, and constraint enforcement all move p up (see §8).
- **Scoped agents.** Split a 30-step generalist into five 6-step specialists, each with a narrow contract, a known toolset, and an eval (see §7.1 and the scoped-sub-agent pattern in [autonomous_agents_guide.md](autonomous_agents_guide.md) §4). Five 6-step agents at 0.97 per-step give 0.97³⁰ ≈ 40% end-to-end only if *all* must succeed in series — but scoping converts many *serial* dependencies into *parallel or hierarchical* ones, and each specialist is independently verifiable and independently fixable.
- **Budget the steps.** If the task needs N steps, design the eval and the SLO around p^N, not around the demo (see §10).

### 1.5 The Honest Framing: 95% Per Step Is Optimistic

The 95%-per-step framing is the most commonly cited version of the compounding argument in 2025–2026 practitioner writing — verified across multiple independent pieces (Lens, Growthengineer, zalt.me, Anand Taralika, algorithme.site — see §12). The *arithmetic* is exact; the *assumption* deserves scrutiny:

- **95% per step is a stylized, optimistic figure for many real tasks.** Practitioner estimates for *well-scoped* agents on *narrow* tasks put per-step accuracy in the **88–97%** range; for broad, tool-heavy, uncurated tasks the realistic per-step reliability is frequently **lower than 90%** — which puts a 10-step task at **≤35%**.
- **The steps are not all equally hard.** Retrieval and well-specified tool calls can be near-deterministic (95–99%+); *planning* steps, *judgment* steps, and *free-text generation* steps carry the real error mass. Averaging hides which steps are killing you — measure per-step (see §4).
- **The "95%" framing is directionally honest even when the number is soft.** Whether the real per-step number is 97% or 88%, the qualitative conclusion is identical and robust: *long chains fail, and they fail in production long before they fail in the demo.*

> **Flag.** The 95%/0.5987 framing is verified as *the* canonical arithmetic of the 2025–2026 "agents fail" discourse; the 88–97% realistic-range figure is a practitioner estimate (zalt.me, 2026) and should be treated as an indicative range, not a measured constant.

### 1.6 The Math Table

Overall task success = p^N, for uniform per-step success probability p over N steps. **Verified values (computed):**

| Steps (N) | 90% per step | 95% per step | 99% per step |
|---|---|---|---|
| 1 | 90.0% | 95.0% | 99.0% |
| 2 | 81.0% | 90.3% | 98.0% |
| 3 | 72.9% | 85.7% | 97.0% |
| 5 | 59.0% | 77.4% | 95.1% |
| 10 | **34.9%** | **59.9%** | 90.4% |
| 20 | 12.2% | **35.8%** | 81.8% |
| 50 | 0.5% | 7.7% | 60.5% |

Reading the table: a "95% reliable" agent is a coin flip at 20 steps; a "90% reliable" agent is a coin flip at **5** steps; only a 99%-per-step agent survives a 20-step horizon with dignity. Most production agents are not 99%-per-step agents — which is precisely why most production agents fail (see §6).

**The design corollary, stated once:** *every step you remove is worth more than every point of per-step accuracy you can squeeze out of the prompt — and both are worth more than upgrading the model.*

---

## 2. The Failure Taxonomy

A production agent fails in a bounded set of recognizable ways. Naming them is the first half of managing them: **failure taxonomy → detection → mitigation** is the entire reliability playbook (the mapping table is in §8.1). The taxonomy below is the consolidated one used throughout this guide; it is derived from practitioner post-mortems, the survey data in §6, and the platform operating experience documented in [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) §4–§5.

### 2.1 Task Failures — the Agent Does the Wrong Thing

The agent completes its loop, but the *outcome* is wrong. These are the failure modes the compounding math predicts, observed concretely:

- **Wrong tool selection.** The agent calls the semantically wrong tool — searching transactions instead of accounts, writing to prod instead of staging, calling the read API when the write API was needed. Symptom: correct-looking output derived from the wrong source of truth.
- **Wrong arguments.** The right tool, wrong parameters: an account lookup with a mistyped customer ID, a date range with swapped bounds, a currency field populated with the account number. Because the API "succeeds" (200 OK, empty or plausible result), the error is silent — the classic *garbage-in-garbage-out* amplification: one bad argument poisons every downstream step.
- **Hallucinated steps.** The agent invents a step that does not exist in the toolset or the domain: a "verification" it did not perform, a regulatory check it fabricates, an API it imagines. The output is fluent, structured, and false.
- **Incomplete task.** The agent stops early, declaring success without satisfying the full contract — the Terminal-Bench signature pattern (wrote code, self-reviewed, stopped without running tests; §1.3), or a research agent that answers 2 of 5 sub-questions and reports done.

### 2.2 Control Failures — the Agent Does Not Stop

Where task failures are about *quality*, control failures are about *execution*: the loop itself misbehaves.

- **Loops (infinite and near-infinite).** The agent repeats the same tool call with the same arguments — search → nothing found → search again → nothing found — sometimes with slight mutation, sometimes exact. Every iteration burns tokens and latency; the output never converges.
- **Stuck states.** The agent is not looping but is not progressing either: it re-plans the same plan, asks the same clarifying question, or waits on a tool result that will never arrive. Symptoms: flat activity, no new tool calls, no terminal answer.
- **Runaway (unbounded action).** The agent takes actions without an upper bound — thousands of tool calls, writes to an increasing set of resources, escalating retries after failures. The runaway loop is the *cost* failure mode (see §5.2) and, in the worst case, the *security* failure mode (see §2.4).
- **Poor error recovery (cascading errors).** The agent encounters an error and recovers badly: it retries a failing call with the same input, it papers over a failed step with a *plausible hallucination* ("the API returned 2,340 records" when it returned an exception), or it lets one failed step derail the whole plan. Cascades are how a single flaky tool call becomes a full task failure — and how a single task failure becomes an outage of user trust.

### 2.3 Integration Failures — the Environment Is Wrong

The agent is fine; the world it touches is not. These failures are *not* the model's fault, which is exactly why they are so common: the model is blamed for what the integration layer broke.

- **API changes.** Upstream endpoints change shape (renamed fields, new required headers, removed endpoints) and the agent's tool definitions or its mental model of the API go stale.
- **Auth failures.** Expired tokens, rotated keys, expired service accounts, scope changes. Symptom: a burst of 401/403s; the agent interprets them as "empty result" and hallucinates an answer instead of failing loudly.
- **Flaky tools.** Timeouts, rate limits, partial responses, eventual-consistency windows. A search that returns stale or empty results *looks* like "no data exists" to the model. Flaky tools are the single most underrated source of agent unreliability — the model is deterministic given its context, and the context was wrong.
- **Schema drift.** Databases and message schemas evolve; the agent's retrieval and tool layers were built against the old shape. Symptom: type errors, empty result sets, and — worst — silently *mis-mapped* fields (account number read as balance).

### 2.4 Evaluation Failures — the Safety Net Is Wrong

The failure taxonomy above describes the agent. The next three categories describe the *system around* the agent — and they are where most production blowups actually originate. The evaluation failure modes:

- **The demo-vs-production gap.** The eval set matches the demo, not the deployed reality: golden sets curated by the builders, distribution shift between test queries and real queries, and a long tail of edge cases the golden set never sampled. Dedicated section: §3.
- **Non-determinism.** The eval passes once, then fails on re-run: temperature, sampling, and — critically — **model updates that silently change behavior** without any code change. This is the temporal dimension, fully covered in [ai_agent_drift_guide.md](ai_agent_drift_guide.md); the structural point here is that *a single-pass eval is not evidence* (see §4.1).
- **Cost and latency blindness.** The eval measures correctness on 100 curated cases and ignores that the agent spends 40k tokens per task or takes 90 seconds p95. In production, cost and latency are *correctness*: an agent that exceeds budget gets killed, and an agent that makes users wait gets abandoned (see §5).
- **Security gaps.** The eval does not include adversarial inputs — prompt injection, tool-abuse attempts, exfiltration probes. Production will include them, on day one, from the first bot. See [llm_guard_models_guide.md](llm_guard_models_guide.md) and [prompt_injection_guide.md](prompt_injection_guide.md).
- **Human factors.** No human-in-the-loop checkpoint for high-value actions, and unclear ownership of the agent's decisions ("the model did it" is not an owner). These are organizational failures that surface as production incidents.

### 2.5 The Failure Taxonomy Table

| Category | Examples | Symptom | Detection |
|---|---|---|---|
| **Task failure** | Wrong tool selected; wrong arguments; hallucinated step; incomplete task | Confident, fluent, *wrong* output | Per-step verification, output contracts, verifier model, golden-set evals, human review sampling |
| **Control failure** | Infinite loop; stuck state; runaway actions; cascading retries | No convergence; repeated identical tool calls; token/step explosion; escalating error rate | Step counters, loop detectors (identical-call hash), max-step/max-token caps, timeouts, idle detection |
| **Integration failure** | API change; auth expiry; flaky tool; schema drift | 401/403 bursts; empty/partial results; type errors; agent "confidently" filling gaps | Tool-level SLIs (error rate, latency, empty-rate), schema tests, contract tests, integration health dashboards |
| **Evaluation failure** | Golden-set gap; distribution shift; non-determinism; no cost/latency/security/human checks in eval | Passes eval, fails production; flaky eval results; surprise incidents | Prod-sampled evals, eval re-runs with seeds, adversarial sets, cost/latency asserts in the eval harness |
| **Reliability failure** | Non-deterministic outputs; silent wrong-but-confident answers; model-update regressions | Same input → different output; wrong answers with high confidence | Determinism checks, drift monitoring ([ai_agent_drift_guide.md](ai_agent_drift_guide.md)), trace-level sampling |
| **Cost & latency failure** | Token blowup; runaway loop spend; multi-step p95 latency | Cost per task spikes; users abandon mid-task; budget alarms | Cost-per-task metering, loop alarms, latency SLOs, FinOps dashboards ([../finops_guide.md](../finops_guide.md)) |
| **Security failure** | Prompt injection; tool abuse; data exfiltration via tool args | Agent follows instructions embedded in tool output; unusual tool call patterns | Guard models, tool allow-lists, input/output filters, injection test suites ([llm_guard_models_guide.md](llm_guard_models_guide.md)) |
| **Human-factor failure** | Missing HITL for high-value actions; unclear ownership | Wrong actions executed autonomously; incidents with no accountable owner | Approval gates for high-risk tool calls, ownership registry, incident runbooks ([implementing-responsible-ai.md](implementing-responsible-ai.md)) |
| **Context failure** | Context drift, context confusion, token-limit truncation, lost-in-the-middle | Agent "forgets" the goal, contradicts itself, drops early constraints, or acts on truncated context (§2.6) | Context budgets, compaction, pinned instructions, retrieval hygiene ([agent_runtime_cache_design_guide.md](agent_runtime_cache_design_guide.md)) |

### 2.6 Context Management Failures: Drift, Confusion, Token Limits

The agent's context window is its entire working memory — and it is a *bounded, degrading* resource. Context failures are structural: they are guaranteed by the architecture of every long-horizon agent, not caused by any single bad step.

- **Context drift.** As the run grows, early content (the original goal, the constraints, the user's true intent) is pushed out of effective attention by later content — instruction decay in long-horizon runs. The agent starts executing the task it *thinks* it is on, which drifts from the task it was given. Symptom: late steps violate early constraints (e.g., the agent answers in the wrong language, or queries the wrong entity, because the goal has been diluted).
- **Context confusion.** Conflicting information accumulates in context — a retrieved document contradicts a tool result, which contradicts the conversation history — and the model resolves the conflict by trusting the *wrong* source, usually the most recent or most verbose one. Symptom: internally contradictory reasoning in a single confident answer.
- **Token limits.** The window fills: history gets truncated, retrieval results get dropped, early tool outputs get summarization losses. The agent then *silently* operates on partial context — and produces confident answers from the parts that survived. Symptom: correct early steps, degraded late steps, no error anywhere.
- **Lost-in-the-middle.** Models attend less reliably to the middle of long contexts than to the head and tail; the most important constraint, placed mid-context, is the one most likely to be ignored.
- **The interaction with the taxonomy.** Context failures amplify every other category: a truncated context produces wrong-argument calls (§2.1), a drifted context produces loops (the agent re-derives the plan it forgot, §2.2), and a confused context produces silent failures (§4.2). This is why context management is a *reliability* concern, not a performance nicety.

| Context failure | Symptom | Mitigation |
|---|---|---|
| Context drift (instruction decay) | Late steps violate early constraints | Pinned goal/constraint block re-injected at intervals; compaction that preserves the instruction layer; step-boundary goal checks |
| Context confusion | Contradictory reasoning; wrong source trusted | Source tagging on all context (tool result vs doc vs history); retrieval hygiene; verifier groundedness checks |
| Token-limit truncation | Silently partial operation; degraded late steps | Context budgets per phase; summarization with loss tracking; compaction ([agent_runtime_cache_design_guide.md](agent_runtime_cache_design_guide.md)); refuse-to-answer when evidence was dropped |
| Lost-in-the-middle | Mid-context constraints ignored | Place critical constraints at head/tail; repetition with variation; structured instruction blocks |
---

## 3. Works in the Demo, Fails in Production

"Works in the demo" is the most expensive sentence in agent engineering. The demo-vs-production gap is not one problem but four, and they compound exactly like step errors do.

### 3.1 Golden-Set Blindness

The agent is evaluated — and demoed — against a **golden set**: a curated collection of inputs with known-good outputs, usually written by the same people who built the agent, usually drawn from the same few scenarios that motivated the project. Golden sets are necessary but systematically blind:

- They encode the builders' *expectations* of user behavior, not user behavior.
- They are small (tens to hundreds of cases), so the pass rate is statistically fragile — a 90% pass on 50 cases is ±8 points of noise.
- They are static, so they cannot detect distribution shift or model drift (see [ai_agent_drift_guide.md](ai_agent_drift_guide.md) §5).
- Most damaging: they are the *same distribution as the demo*, so the demo and the eval agree with each other — and neither agrees with production.

### 3.2 Distribution Shift: Real Queries vs Demo Queries

**Verified as the standard failure narrative:** production queries differ from demo queries in measurable ways — longer, noisier, multi-intent, adversarial, domain-slang-heavy, or simply *unseen categories* ("long tail of user queries"). A customer-support agent demoed on 40 clean tickets ("I want to change my address") meets production tickets like "why did my card get declined twice in Singapore after I changed my phone number" — a query that requires cross-system reasoning the golden set never exercised. The eval said 92%; the first production week says otherwise.

### 3.3 The Long Tail of Failures

The distribution of real-world failures is a **long tail**: a handful of common failure modes account for most incidents, but the *remaining* incidents are a long, nearly flat tail of one-off edge cases — each rare, collectively inevitable. The practical consequences:

- A 95% golden-set pass rate *feels* like success but means **one in twenty real interactions is broken**, and the broken ones are not randomly distributed — they cluster in exactly the unusual, high-stakes cases (unusual requests, edge accounts, ambiguous inputs).
- The tail is unbounded: you cannot golden-set your way out of it. You can only (a) shrink it with scoping (§7.1), (b) catch it with production-sampled evals (§8.5), and (c) survive it with fallbacks and HITL (§7.3, §7.6).
- The tail is where the *expensive* failures live: rare tool misuse, rare policy misstatements, rare injection attempts. "Rare" is not "never" at production volume — a 0.1% failure rate is 1,000 failures per million interactions.

### 3.4 The Environment: Real Data, Real APIs, Real Users

The demo runs in a clean room; production runs in the actual environment, and every environmental difference is a failure-mode generator:

- **Real data** — dirty, duplicated, permission-scoped, PII-laden, schema-drifted (see §2.3 schema drift).
- **Real APIs** — rate-limited, flaky, versioned, sometimes *down*; the demo's mock or sandbox never failed, so the agent never learned to handle failure.
- **Real users** — impatient, adversarial, typo-prone, and capable of prompt injection; the demo's users were the product team.
- **Real scale** — concurrency, timeouts, quotas, and the cost/latency economics of §5, none of which exist in a notebook.

### 3.5 The Demo vs Production Table

| Dimension | Demo | Production |
|---|---|---|
| **Inputs** | Curated golden set; clean, single-intent, well-formed | Long tail of real queries; noisy, multi-intent, adversarial, domain-specific |
| **Data** | Sample or synthetic data; clean schemas | Real data; dirty, duplicated, PII-scoped, schema-drifted |
| **APIs/tools** | Mocks, sandboxes, happy paths | Real systems; flaky, rate-limited, versioned, occasionally down |
| **Users** | The builders | Real users — impatient, typo-prone, injection-capable |
| **Model** | One pinned version | Updates happen without you; behavior drifts ([ai_agent_drift_guide.md](ai_agent_drift_guide.md)) |
| **Step count** | 2–3 steps, demoed once | 10–50 steps, executed thousands of times (compounding: §1) |
| **Success bar** | "It worked once, on video" | SLOs: success rate, cost per task, p95 latency, incident-free operation |
| **Failure handling** | None needed — it worked | Required: retries, fallbacks, checkpoints, HITL, guardrails (§8) |
| **Observability** | Print statements | Traces, metrics, logs, evals-on-production-samples (§4.3) |
| **Consequences of error** | Re-run the demo | Real money, real trust, real regulatory exposure |

---

## 4. Reliability and Observability

### 4.1 Non-Determinism: The Same Input Is Not the Same Output

LLM agents are stochastic end to end: sampling temperature, token-level randomness, non-deterministic tool result ordering, and — the big one — **model version changes**. The reliability consequences are structural:

- **A single successful run is not evidence.** "It worked in testing" with N=1 is an anecdote. Eval runs must be repeated (multiple seeds/temperatures) and reported with variance — see the evaluation methodology in [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md) and [rag/rag_evaluation_methodology_guide.md](rag/rag_evaluation_methodology_guide.md).
- **Model updates are silent behavior changes.** The vendor ships a new version; your agent's tool-call format habits, refusal patterns, and even *which* hallucinations it produces shift — with no code change, no deploy, no alert. This is the *drift* failure mode in its purest form, and it is why the temporal guide ([ai_agent_drift_guide.md](ai_agent_drift_guide.md)) is the required companion to this one: structural failure analysis explains *where* agents break; drift explains *when* they break again after you fixed them.
- **Determinism is a feature you can buy.** Lower temperature, pinned model versions, cached exact-match steps, and structured outputs (JSON schema, tool-call constraints) all trade a little flexibility for a lot of reproducibility — and reproducibility is what makes SLOs meaningful.

### 4.2 Silent Failures: Wrong but Confident

The most dangerous agent output is not the refusal — it is the **wrong-but-confident answer**: a plausible number, a fabricated tool result, a confident policy statement that is unfaithful to the source. Silent failures are dangerous precisely because nothing in the pipeline *notices*: the agent reports success, the tool returned "success," the trace looks clean, and the user (or the downstream process) acts on a lie. Mechanisms:

- **Hallucinated tool results** — the agent "remembers" an API response that never came (see the cascading-error pattern in §2.2).
- **Unfaithful grounded answers** — retrieval found document A, but the answer asserts document B's content (groundedness failures; see the RAG evaluation methodology in [rag/rag_evaluation_methodology_guide.md](rag/rag_evaluation_methodology_guide.md)).
- **Premature completion** — the Terminal-Bench signature: self-review says "done," tests never ran (§1.3).

The countermeasure set is the *verifier pattern* (§7.7, §8.3): a second model or a deterministic checker validates tool args, groundedness, and completion criteria — because the agent's own self-assessment is the least reliable judge of its own output (self-correction limits are flagged in §8.3).

### 4.3 Observability: Blind Agents Fail in the Dark

A production agent without tracing is a production incident you will find out about from the customer. The observability baseline for agents:

- **Tracing** — every step of every run: prompt version, model version, tool calls (arguments in, results out), tokens, latency, and the decision chain. OpenTelemetry GenAI semantic conventions are the emerging standard; the platform-side implementation is covered in [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) §4.5.
- **Metrics** — success rate, cost per task, step count per task, loop rate, tool error rates, p95 latency (the §5 table).
- **Logs** — full input/output at the trace level, retained per compliance policy (banks: immutable audit trails — see [implementing-responsible-ai.md](implementing-responsible-ai.md)).
- **The correlation key** — one request/agent/trace ID across model calls, tool calls, guardrail decisions, and human approvals. Without correlation, every post-incident question ("what did the agent do and why?") is unanswerable.

The structural point: **you cannot SLO what you cannot see.** Observability is the prerequisite for every other reliability practice in this guide — evals, error budgets, loop detectors, and drift monitoring all consume traces.

### 4.4 Reliability Engineering: SRE for Agents

Agents deserve the same discipline as any production service — adapted to the fact that the "unit" of reliability is a *task*, not a request:

- **SLIs (Service Level Indicators).** Task success rate (defined by an *outcome* check, not by "agent said done"), step success rate, cost per task, p95 latency, loop rate, silent-failure rate (measured by sampling + human review — you cannot count what you cannot detect).
- **SLOs (Service Level Objectives).** e.g. "≥95% of support tickets resolved correctly end-to-end," "≤$0.05 per task p95," "≤30s p95 completion." Set the task-level SLO with the compounding math in mind: a 10-step agent with 95% per-step reliability *cannot* meet a 95% task SLO (it is at ~60%) — the SLO must be set against the achievable number *after* scoping, and the eval must be built to enforce it (§8.5).
- **Error budgets.** Track the gap between observed success and the SLO; when the budget is burning, freeze feature work and fix reliability (retries, scoping, verifiers). This is standard SRE practice — error budgets and SLOs are well-established Google SRE concepts — applied to agent tasks.
- **Change management.** Prompts, model versions, tool definitions, and eval sets are *code*: versioned, reviewed, rolled out, rolled back. A prompt change is a deploy; a model update is a deploy you did not make — which is why pinning and drift monitoring exist.

### 4.5 Testing: Evals Are the Only Tests That Matter

Agent "unit tests" (does the function exist, does the API call validate) are necessary but near-useless for reliability — the failure surface is semantic. The testing stack that matters, from [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md), [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md), and [rag/rag_evaluation_methodology_guide.md](rag/rag_evaluation_methodology_guide.md):

1. **Golden-set evals** — regression canaries; know their blindness (§3.1).
2. **Synthetic evals** — generated adversarial and edge-case inputs to probe the long tail cheaply.
3. **Production-sampled evals** — continuously score *real* traffic (sampled, labeled by humans or LLM-judges) — the only eval that sees the actual distribution (§3.2). This is the eval that catches distribution shift, model updates, and the long tail.
4. **Trajectory evals** — score the *path* (tool selection, arg validity, step ordering), not just the final answer; the taxonomy in §2 is only detectable at the trajectory level.

### 4.6 Regression: Eval Gates in the CI

Every change — prompt, model, tool schema, agent logic — must pass the eval suite before deploy, and the suite must include production-sampled cases, not just the golden set. The eval gate is the *only* mechanism that converts "it worked in the demo" into "it still works in production": without it, the natural history of an agent is monotonic degradation under a stream of well-intentioned changes (see the regression-testing and eval-gate patterns in §8.2, and the drift guide for the ongoing dimension).

---

## 5. Cost and Latency

Cost and latency are not "operational details" of agent systems — they are **correctness constraints**. An agent that exceeds its budget gets killed mid-task (task failure); an agent that takes 90 seconds fails the interaction even when the final answer is right (user abandonment). The economics:

### 5.1 Token Costs: The Multi-Step Multiplier

Each agent step is a full model call — input tokens (context: system prompt + history + tool results, growing every step) plus output tokens (reasoning, tool calls). The cost profile:

- **Per-step tokens accumulate.** A 10-step agent with ~2k input + ~500 output tokens per step spends ~25k tokens *per task*; the *input* side grows with the context window as history and tool results append (see [agent_runtime_cache_design_guide.md](agent_runtime_cache_design_guide.md) for the caching and context-compaction patterns that fight this).
- **Cost per task = steps × tokens-per-step × price.** Worked example: a mid-tier frontier model at ~$3/M input + ~$15/M output, 10 steps × (2k in + 500 out) ≈ 25k tokens ≈ **$0.15–0.25 per task** — before any loop. At 100k tasks/month that is $15–25k/month for *one* agent, before retries and loops.
- **Compounding applies to cost too.** The same p^N structure that kills reliability multiplies spend: every failed task that retries is another full-chain cost, and every *retry loop* is unbounded spend.

### 5.2 Runaway Loops: Unbounded Spend

The control failure of §2.2 is the cost event: an agent in a search→no-result→search loop spends a token stream per iteration with no ceiling. **The single most important cost control is a hard budget cap: max steps, max tokens, max dollars per task, enforced by the runtime, not by the prompt.** ("Please don't loop" is not a control; a step counter that terminates the run is.) Verified as standard practice: every production agent framework and platform ships max-iteration and budget controls; the loop detector (identical-call hashing, §8.2) turns "budget blew out" post-mortems into "loop detected and terminated at step 12" alerts.

### 5.3 Cost Control: FinOps for Agents

- **Budget caps** — per-task token/dollar ceilings (hard, enforced).
- **Metering** — per-agent, per-tenant token/cost attribution at the gateway (see [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) §2.4 and the FinOps practice in [../finops_guide.md](../finops_guide.md)).
- **Model tiering** — route easy steps to cheap models, hard steps to frontier models (the RouteLLM line reports ~85% cost savings at ~95% quality retention on controlled evals; see [llm_latency_optimization_guide.md](llm_latency_optimization_guide.md)).
- **Caching** — exact-match and semantic caching of repeated sub-steps (40–90% cost reduction on repeat traffic is commonly reported; [agent_runtime_cache_design_guide.md](agent_runtime_cache_design_guide.md)).
- **Anomaly alerts** — cost-per-task rising week over week is a *reliability* signal (distribution shift, drift, new failure mode), not just a finance signal.

### 5.4 Latency: The Per-Step Accumulation

Latency multiplies the same way cost does. Each step adds model time (TTFT + generation) plus tool time (API round-trips, retrieval). A 10-step chain at 2–4 seconds per step is **20–40 seconds p95** — before retries, before loops. Latency budget math: if the UX contract is 10 seconds, the agent can afford *at most* 3–4 steps on a frontier model. Worked example: a balance-inquiry task — step 1 auth check (~1.5s), step 2 customer lookup (~2s), step 3 account query (~2s), step 4 generation (~3s) — totals ~8.5s p95 *in the happy path*; one retry on the customer lookup pushes it past 10.5s and out of budget. The design response is not a faster model; it is *fewer steps* (batch the auth and the lookup, cache the customer record) and *parallelizable calls* where the steps are independent. Latency optimization techniques (speculative decoding, prompt caching, model tiering, streaming, parallelizable tool calls) are covered in [llm_latency_optimization_guide.md](llm_latency_optimization_guide.md); the structural point here is that **step count is a latency budget line item**, and the compounding table (§1.6) is also a latency table.

### 5.5 User Experience: Waiting, Abandonment, Streaming

- **Abandonment is the p95 tax.** Users wait a few seconds for a chatbot; they do not wait 40. Multi-step agents must either be fast (fewer steps, cheaper models) or *look* fast (streaming partial results, progress indicators, staged delivery).
- **Streaming partial results** is the standard mitigation: emit the first useful token fast (streaming, TTFT optimization), show the agent's progress ("searching accounts… verifying…"), and deliver intermediate artifacts (a draft, a checklist) rather than one late monolith. Verified as standard UX practice for agentic products in 2025–2026 — every major agentic product ships progressive disclosure of agent activity.
- **Degraded modes** — when the full agent cannot complete within budget, fall back to a cheaper, narrower path (§7.6): answer from a fixed knowledge base, escalate to a human, or deliver a partial result with an explicit caveat. Better a 90%-complete answer in 8 seconds than a perfect answer in 45.

### 5.6 The Cost-Latency Table

| Failure | Symptom | Control |
|---|---|---|
| Token blowup on long tasks | Cost per task drifts upward as context grows | Context compaction, caching, step budgets ([agent_runtime_cache_design_guide.md](agent_runtime_cache_design_guide.md)) |
| Runaway loop | Token/cost spike; no convergence | Max-step + max-token + max-dollar caps; loop detector; timeout |
| Retry storms after flaky tools | Cost multiplies on failure paths | Bounded retries with backoff, circuit breakers, fallback tools (§8.2) |
| Multi-step p95 latency | Users abandon mid-task | Step reduction, model tiering, streaming, progress UX, degraded modes |
| Cost-per-task creep | Week-over-week cost rise with no volume change | Metering + anomaly alerts; treat as drift/reliability signal |
| Expensive final step fails | Frontier-model verification step doubles cost of failures | Verifier tiering (cheap verifier for most cases), only escalate |
| Unbounded autonomy | Agent takes high-cost actions (bulk writes, external calls) without limit | HITL approval gates for high-cost/high-risk tool calls (§7.3) |

---

## 6. The Known Discourse

### 6.1 The Practitioner Essays: "Why Agents Fail" (2025–2026)

The "agents fail" discourse is real, verifiable, and unusually consistent. It emerged in two waves: the **capability wave** (late 2024 — Anthropic's *Building Effective Agents*, December 2024, which popularized the *workflow vs agent* distinction and the "simplest solution that works" principle) and the **production wave** (2025–2026 — the reliability post-mortems). Verified practitioner pieces from the production wave include:

- **"Why Agents Fail (and How to Notice)"** (Tendril, 2025) — a six-failure-mode catalogue with a memorable framing: *"agents fail in weird, quiet, expensive ways."*
- **"Why Agents and Agentic Systems Can Fail"** (Niklas Heidloff / IBM, 2025) — a top-5 architectural reasons list from a working travel-agent example.
- **"Why LLM Agents Break in the Real World"** (Medium, 2025–2026) — the *"Demo Illusion vs Production Reality"* gap, matching §3 of this guide.
- **"The Math Behind Why Multi-Step AI Agents Fail in Production"** (Lens, 2025–2026) — the compounding arithmetic, with the key observation that *"demos usually hide this because they only show 2 or 3 steps."*
- **"The Reliability Compounding Problem in Multi-Agent Systems"** (Growthengineer, 2026) — traces the product rule to **Lusser's Law (1957)** and applies it to chained agents.
- **"AI Agent Reliability: Setting Honest Accuracy Targets"** (zalt.me, 2026) — the 88–97% realistic per-step range for well-scoped narrow tasks.
- **"The 10:1 Rule: Why Production Agentic Systems Need Validation"** (Anand Taralika, 2026) — *"a system that's 95% reliable per step is only 60% reliable over a 10-step task. At 90% per step, you're down to 35%."*
- **"Why LLM Agents Keep Failing (and It's Not the Prompt)"** (Hugging Face forums, 2026) — argues the failure is structural (reasoning embedded in prompts instead of executable components), representative of the "structure, not prompt" strand of the discourse.
- **"How to Stop AI Agents from Failing"** (GitHub sample repo, 2026) — the four-failure catalogue (fabricated statistics, wrong tool choice, ignored business rules, false success claims) with detection/containment techniques; representative of the tooling-heavy practitioner response.

The **research** side of the discourse: the June 2025 **Carnegie Mellon + Anthropic** study of computer-using agents concluded agents "make too many mistakes for businesses to rely on them for any process involving real money," with press coverage reporting ~85% failure on the studied tasks (as-reported figure; varies by task sample). And **Terminal-Bench** (2025) showed even frontier agents solving only a minority of real terminal tasks — with LangChain's trace analysis identifying false-early-completion as the most common failure pattern.

> **Verification note.** The math pieces are *verified* (arithmetic exact, sources multiple and independent). The failure-rate figures (Anthropic/CMU ~85%, Terminal-Bench solve rates) are *as-reported* — directionally reliable, exact numbers sensitive to task sampling. Treat all practitioner percentages as indicative, not measured constants.

### 6.2 Industry Data: The Surveys

- **LangChain — State of Agent Engineering 2025** (survey of **1,340 practitioners**, fielded Nov 18 – Dec 2, 2025): **57.3%** of organizations report agents in production (up from **51%** in the prior survey); **~89%** report observability tooling; but only **~52%** report actually *testing* their agents. The gap between "in production" and "tested" is the discourse in one statistic.
- **Datadog — State of AI Engineering** (telemetry from **1,000+ production customers**, 2025–2026): corroborates agents moving from experimentation to production at scale, with reliability and observability gaps as the dominant production themes.

> **Verification note.** These are *self-reported* survey numbers (LangChain) and vendor-telemetry summaries (Datadog) — approximate, directionally sound, and consistent with the practitioner essays. Flag: no independent third-party audit of agent production failure rates exists; treat survey percentages as order-of-magnitude evidence, not measurement.

### 6.3 The Consensus

Across the essays, the surveys, and the platform operating experience, the discourse converges on five claims:

1. **Compounding is the root cause.** Long chains fail; the failure rate of a multi-step agent is structurally higher than any single-step benchmark suggests (§1).
2. **Evals are the weak link.** Most production agents are not evaluated against production reality; golden-set-only evals are the norm, production-sampled evals the exception (§3, §4.5).
3. **Scope is the differentiator.** The agents that work are narrow; the agents that fail are the generalists (§7.1).
4. **Control is non-negotiable.** Loops, budgets, and termination guarantees are basic hygiene, not advanced features (§5.2, §8.2).
5. **HITL is not a compromise.** For high-value actions, human approval is a reliability feature, not a scaling concession (§7.3).

### 6.4 The Counterpoints: What "Agents Work" Looks Like

The discourse also has a consistent counterpoint — not "agents are fine," but *"agents work when the engineering is honest"*:

- The same surveys showing 43% not-yet-in-production also show 57% *are*, and the successful cohort shares a profile: **narrow scope, eval-first development, HITL checkpoints, guardrails, and observability** (§7).
- The compounding math is a *design input*, not a death sentence: it tells you exactly what to build (fewer steps, higher per-step reliability, scoped agents) and what to measure (task success, cost per task, silent-failure rate).
- Production agent deployments with discipline — support triage, document processing, coding assistants with test gates — report real, sustained value. The failure mode is *uncontrolled* autonomy, not autonomy itself.

### 6.5 Discourse Summary Table

| Claim | Source type | Status |
|---|---|---|
| Per-step errors compound: 0.95¹⁰ ≈ 60%, 0.95²⁰ ≈ 36% | Practitioner math essays (Lens, zalt.me, Taralika, Growthengineer, algorithme.site) | **Verified** (arithmetic exact; framing canonical) |
| Compounding traces to Lusser's Law (series reliability = product) | Reliability engineering (1957) + practitioner citations | **Verified** |
| Realistic per-step accuracy 88–97% (narrow, well-scoped tasks) | Practitioner estimate (zalt.me, 2026) | **Indicative** — flag as estimate |
| Agents "make too many mistakes for real-money processes" | CMU + Anthropic study, June 2025 | **Verified directionally** (widely reported) |
| Computer-use agents fail ~85% of studied real-world tasks | Press reporting of CMU/Anthropic study | **As-reported** — task-sample sensitive |
| Terminal-Bench: most common failure = false early completion | LangChain trace analysis; benchmark ecosystem | **As-reported** — consistent across sources |
| 57.3% of orgs have agents in production; only ~52% test | LangChain State of Agent Engineering 2025 (N=1,340) | **Self-reported survey** — approximate |
| ~89% of agent teams have observability; gaps remain in testing | LangChain 2025; Datadog State of AI Engineering | **Self-reported/vendor telemetry** — approximate |
| Demos hide compounding (2–3 steps) and the long tail | Practitioner essays; universal post-mortem experience | **Verified directionally** |
| Production agents that work: narrow scope + evals + HITL + guardrails | Discourse consensus across essays, surveys, platforms | **Consensus** — consistent, not measured |
---

## 7. The Success Factors: What Production Agents Do Right

The counterpoint to §1–§6 is not "agents work" — it is *"agents work when you engineer them as production systems."* The successful cohort in the surveys (§6.2) shares a consistent profile, verified across the discourse and the platform operating experience:

### 7.1 Narrow Scope: Single-Purpose Agents

**Verified as the single strongest predictor of production success.** The agents that survive contact with production are narrow: one task, one domain, one bounded toolset — a *ticket triager*, not a *universal assistant*; a *policy-lookup responder*, not a *general knowledge agent*. Why it works:

- Narrow scope shrinks the input distribution (fewer long-tail surprises, §3.3) and the action space (fewer wrong-tool and wrong-arg opportunities, §2.1).
- Narrow scope makes evals meaningful: a well-defined task has a well-defined success criterion, which is the prerequisite for eval gates (§8.2) and SLOs (§4.4).
- Narrow scope shortens the chain: 5–8 steps instead of 20–50, which is the difference between ~60% and ~20% end-to-end at 95% per-step (§1.6).
- Narrow scope makes failure *containable*: a triager's worst case is a mis-routed ticket, not a mis-executed trade.

### 7.2 Strong Evals: Eval-First Development

Successful teams build the eval before the agent, not after. The eval suite is the product spec: golden set (regression canary), synthetic adversarial set (long-tail probing), and **production-sampled evals** (continuous scoring of real traffic — the only set that sees the real distribution). Full methodology in [rag/rag_evaluation_methodology_guide.md](rag/rag_evaluation_methodology_guide.md); the eval-vs-validation distinction in [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md).

### 7.3 HITL: Human Checkpoints and Approval Gates

**Verified as a defining trait of the successful cohort.** Humans are placed at the decision points where errors are expensive: high-value actions (payments, writes, external sends), irreversible actions, and actions with regulatory weight. HITL is an *approval gate* in the control flow (enforced by the runtime, not requested politely), not a review queue after the fact. Governance and accountability framing: [implementing-responsible-ai.md](implementing-responsible-ai.md).

### 7.4 Guardrails: Input/Output Filters and Tool Allow-Lists

Successful agents run inside guardrails: input/output filtering (PII, injection, policy), **tool allow-lists** (the agent can only call what it is permitted to call — the single most effective security control), and refusal/redaction logic. Full catalogue in [llm_guard_models_guide.md](llm_guard_models_guide.md) and [prompt_injection_guide.md](prompt_injection_guide.md).

### 7.5 Observability: Traced from Day One

Successful teams treat traces as the primary deliverable of every run: per-step traces with tool arguments and results, correlated by request ID, feeding both dashboards and production-sampled evals. The platform-side pattern: [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) §4.5.

### 7.6 Fallbacks: Degraded Modes Are a Feature

Every successful production agent has a defined failure path: when the agent cannot complete within budget or confidence, it *degrades gracefully* — answers from a static knowledge base, returns a partial result with caveats, or escalates to a human. The fallback is designed, tested, and monitored like a first-class path (the control-loop and fallback patterns are in [autonomous_agents_guide.md](autonomous_agents_guide.md) §5).

### 7.7 Design Patterns: Router, Verifier, Scoped Sub-Agents

The structural patterns that recur in successful deployments (deep-dive: [autonomous_agents_guide.md](autonomous_agents_guide.md) §4, [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md), [hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md)):

- **Router** — a cheap classifier/embedding step sends each request to the right specialist (or to a human, or to a no-agent answer path). Routing *removes steps* from every task it correctly shortcuts, directly fighting the compounding problem.
- **Verifier** — a second model (or deterministic checker) validates the primary agent's output: tool-argument sanity, groundedness, completion criteria. The verifier is the structural answer to silent failures (§4.2) and false early completion (§2.1).
- **Scoped sub-agents** — a supervisor decomposes a task into specialist sub-agents, each with its own narrow contract, toolset, and eval; each is independently testable and replaceable (see §1.4 and the hierarchical patterns guide).

### 7.8 The Success Factors Table

| Factor | Why it works | Implementation |
|---|---|---|
| **Narrow scope** | Shrinks input distribution, action space, step count; makes evals and SLOs meaningful | One task per agent; bounded toolset; explicit refusal for out-of-scope requests |
| **Strong evals** | The only mechanism that detects regression before users do | Eval-first: golden + synthetic + production-sampled sets; eval gates in CI (§8.2) |
| **HITL** | Prevents expensive/irreversible errors; creates accountability | Approval gates on high-value tool calls; human review sampling; ownership registry |
| **Guardrails** | Enforces policy at the chokepoint; contains worst cases | Input/output filters, tool allow-lists, injection defense ([llm_guard_models_guide.md](llm_guard_models_guide.md)) |
| **Observability** | You cannot SLO, eval, or fix what you cannot see | Per-step tracing, metrics, logs; request-ID correlation (§4.3) |
| **Fallbacks** | Turns inevitable failures into graceful degradation | Degraded modes, escalation paths, partial-result delivery (§7.6) |
| **Design patterns** | Router cuts steps; verifier catches silent failures; sub-agents isolate risk | Router + verifier + scoped sub-agents (§7.7) |

---

## 8. Mitigations and Production Patterns

### 8.1 Failure-Mode → Mitigation Mapping

The consolidated mapping from the taxonomy (§2) to concrete mitigations:

| Failure | Symptom | Mitigation | Tooling / pattern |
|---|---|---|---|
| Wrong tool selection | Correct-looking output from wrong source | Tool allow-lists; router; tool descriptions with explicit contracts | Guard models, router, structured tool schemas |
| Wrong arguments | Silent garbage-in-garbage-out | **Tool-argument validation** (schema/type/range checks before the call); verifier model | Pydantic/JSON-schema validation, verifier step |
| Hallucinated steps | Fluent false narrative | Groundedness checks; verifier; tool-result tracing | Verifier model, RAG groundedness evals ([rag/rag_evaluation_methodology_guide.md](rag/rag_evaluation_methodology_guide.md)) |
| Incomplete task | Premature "done" | Explicit completion criteria; verifier; outcome-checked evals | Completion contract, verifier, trajectory evals |
| Infinite loop | Repeated identical calls | **Loop detector** (hash of tool-call signatures); max-step cap | Runtime step counter, loop-detection service |
| Stuck state | Flat activity, no progress | Idle/timeout detection; replan with bounded retries | Timeouts, progress monitors |
| Runaway actions | Unbounded tool calls/writes | **Budget caps** (steps/tokens/dollars); HITL on high-cost calls | Runtime budget enforcement, approval gates |
| Cascading errors | One failure poisons the chain | Bounded retries with backoff; circuit breaker; fallback path | Retry policy, circuit breaker, fallback tools/models |
| API change / schema drift | Type errors, empty results, mis-mapped fields | Contract tests; schema versioning; integration health SLIs | Contract-test suite, schema registry, tool SLIs |
| Auth failure | 401/403 bursts | Credential rotation automation; loud failure (never "empty result") | Secret manager, error semantics |
| Flaky tools | Stale/empty/partial results read as "no data" | Retries, fallback tools, result-quality checks; never let errors look like data | Retry policies, fallback chains, verifier |
| Golden-set blindness | Passes eval, fails production | Production-sampled evals; long-tail synthetic sets | Eval harness with prod sampling (§8.5) |
| Non-determinism | Same input, different output | Pinned models, low temperature, seeded eval runs, determinism checks | Model pinning, eval repeatability |
| Model-update regression | Silent behavior change | Pin + drift monitoring + eval gates on model upgrades | [ai_agent_drift_guide.md](ai_agent_drift_guide.md) |
| Silent wrong answer | Confident falsehood | Verifier; groundedness checks; human review sampling | Verifier step, sampling pipeline |
| Blind agent | Incident found by customer | Per-step tracing; request-ID correlation; dashboards | OpenTelemetry GenAI, platform (§4.3) |
| Token/cost blowup | Cost per task spikes | Budget caps; caching; context compaction; model tiering | [agent_runtime_cache_design_guide.md](agent_runtime_cache_design_guide.md), [../finops_guide.md](../finops_guide.md) |
| Slow multi-step UX | User abandonment | Step reduction; streaming; progress UX; degraded modes | Streaming, tiering ([llm_latency_optimization_guide.md](llm_latency_optimization_guide.md)) |
| Prompt injection / tool abuse | Agent follows hostile instructions | Input/output filters; tool allow-lists; injection test suites | [llm_guard_models_guide.md](llm_guard_models_guide.md), [prompt_injection_guide.md](prompt_injection_guide.md) |
| Missing HITL | Wrong autonomous action | Approval gates on high-value actions; ownership registry | [implementing-responsible-ai.md](implementing-responsible-ai.md) |

### 8.2 Production Patterns

The operational patterns that turn the mitigations into a system:

- **Eval gates in CI** — every change (prompt, model, tool schema, agent code) must pass the eval suite before deploy; the gate includes golden, synthetic, and production-sampled cases. This is the only mechanism that makes "it still works" a *verified* claim instead of a hope.
- **Budget caps** — hard per-task ceilings on steps, tokens, and dollars, enforced by the runtime (verified standard practice across frameworks and platforms).
- **Loop detectors** — hash recent tool-call signatures; if the same (tool, args) recurs, terminate or branch. Catches infinite loops in seconds, not at budget exhaustion.
- **HITL checkpoints** — approval gates for high-value/irreversible/high-cost actions, enforced by the runtime (§7.3).
- **Fallback models and tools** — a degraded chain: primary model → cheaper/secondary model → static knowledge base → human. Model outages and tool failures become non-events.
- **Circuit breakers** — when a tool or an upstream model fails repeatedly, open the circuit (stop calling it, fail fast to the fallback) instead of retrying into the outage. Verified as standard reliability engineering (the classic circuit-breaker pattern from distributed systems) applied to agent dependencies.
- **Max-step timeouts and idle detection** — a run that neither progresses nor terminates gets killed and reported.
- **Change management** — prompts, model versions, tool definitions, and eval sets versioned and rolled out like code (§4.4).

### 8.3 Agentic Design Patterns: What the Architecture Should Look Like

- **Narrow scopes** (§7.1): the default unit of deployment is the specialist, not the generalist.
- **Verifier steps**: the primary agent's output passes through a verifier (second model or deterministic check) before it is trusted — the structural fix for silent failures (§4.2).
- **Self-correction — with limits (flagged).** The "agent critiques its own output and fixes it" pattern is real but *not* a reliability tool: self-review is subject to the same blind spots as the original reasoning, and it adds steps (which compounding punishes). Evidence: the Terminal-Bench false-completion pattern is *self-review declaring success too early* — the agent reviewed its own work and stopped, wrongly. **Use self-correction only where the correction signal is external** (test failures, schema errors, verifier feedback) rather than the model's own opinion; and treat "the agent will catch its own mistakes" as an anti-pattern.
- **Deterministic outer loop, stochastic inner steps**: route, validate, and terminate with *code*; reserve the model for the semantic decisions. The more of the loop that is deterministic, the less of it that compounds.

### 8.4 Guardrail Patterns

- **Input filters** — detect and neutralize prompt injection, PII, and policy-violating inputs before they reach the model.
- **Output filters** — validate the model's output against policy (no PII leakage, no disallowed content, format conformance) before it reaches the user or a tool.
- **Tool allow-lists** — the agent's tool namespace is bounded by policy; anything outside it is uncallable. The highest-leverage security control, and a scoping control (§7.1) in one.
- **Tool-argument validation** — schemas with types, ranges, and cross-field rules (e.g., "end date > start date") checked *before* the call; the direct mitigation for the wrong-arguments failure (§2.1).
- Full catalogue and trade-offs: [llm_guard_models_guide.md](llm_guard_models_guide.md).

### 8.5 Testing Patterns

- **Golden sets** — regression canaries; know their blindness (§3.1).
- **Synthetic sets** — generated adversarial and edge-case inputs (injections, boundary cases, malformed input, out-of-scope requests) to probe the long tail cheaply and repeatedly.
- **Production sampling** — continuously sample real traffic, label (human or LLM-judge), and score. The only eval that sees the real distribution and the only one that catches distribution shift and model updates. Full methodology: [rag/rag_evaluation_methodology_guide.md](rag/rag_evaluation_methodology_guide.md).
- **Trajectory evals** — score tool selection, argument validity, and step ordering, not just final answers (the §2 taxonomy is trajectory-level).

### 8.6 Monitoring Patterns

- **Success-rate SLOs** with error budgets (§4.4) — task-level outcome checks, not "agent said done."
- **Cost per task** metered and alarmed (a rise is a reliability signal, §5.3).
- **Loop alerts** — loop-detector firings, max-step terminations, budget-cap hits: each is a *code* smell in the agent's design, not just an incident.
- **Drift monitoring** — model updates, prompt drift, distribution shift over time; the operational continuation of this guide: [ai_agent_drift_guide.md](ai_agent_drift_guide.md).

---

## 9. Worked Example: A Bank's Customer-Support Agent

### 9.1 The Scenario

A retail bank deploys an LLM agent to handle tier-1 customer-support tickets: answer account-balance and transaction questions, explain fees, and draft responses for the support team. It is a **common scenario** and a **common failure**: the agent was demoed on 40 clean tickets, passed a 30-case golden set at 93%, and was promoted to production. The tools: a customer-lookup API, an account API, a transactions API, and a policy knowledge base (RAG).

### 9.2 The Failures Observed

- **Wrong account lookup (tool args).** The agent parsed "my savings account" from a customer who held two savings accounts and called the lookup with an ambiguous identifier; the API returned the *first* match. The agent answered confidently about the wrong account. Silent failure — no error, no alert (§4.2).
- **The loop.** For "where is my missing deposit," the agent searched transactions, found nothing, searched again with a slightly different filter, found nothing — 14 iterations, no termination, until the runtime's (generous) step limit. Each iteration appended the full history to the context, so tokens per iteration grew.
- **Cost spike.** Between the loops and the growing context, cost per ticket ran **6–8×** the modeled $0.20/task — and the month-end billing anomaly is how the team found out.
- **Hallucinated policy.** Asked about a fee reversal, the agent generated a confident, well-formatted policy statement that the policy KB did not support (groundedness failure — the answer was unfaithful to the retrieved document, §4.2). A support agent would have quoted it to the customer.

### 9.3 The Root Causes

- **Eval gap.** The golden set contained no multi-account queries, no "missing transaction" cases, and no fee-reversal questions; 93% on 30 curated cases said "ship it" while the production distribution said otherwise (§3).
- **Missing HITL.** Nothing in the pipeline required a human for anything — including account-specific answers and policy statements that the bank's own governance rules would have required to be reviewed ([implementing-responsible-ai.md](implementing-responsible-ai.md)).
- **No budget cap.** The loop ran to the step limit because there was no per-task token/dollar ceiling and no loop detector (§5.2).
- **No argument validation.** The ambiguous-identifier lookup sailed through because nothing validated tool arguments before the call (§8.4).
- **No observability.** The wrong-account answer and the hallucinated policy were invisible in logs; the only signals were the billing anomaly and, later, a customer complaint (§4.3).

### 9.4 The Fixes

1. **Tool-argument validation** — the customer-lookup tool now requires a disambiguated identifier (exactly one match; if ambiguous, ask the customer or escalate to a human). Wrong-args failures became impossible instead of probable.
2. **Loop detector + budget caps** — identical-call hashing terminates repeat searches at step 3; hard caps on steps (8), tokens (12k), and cost ($0.60) per ticket, enforced by the runtime.
3. **HITL for high-value actions** — account-specific answers and fee/policy statements now route through a human approval gate before being sent; general FAQs remain fully automated.
4. **Verifier step** — a cheap verifier model checks groundedness of policy answers against the retrieved KB before delivery (the hallucinated-policy failure mode).
5. **Eval additions** — the golden set grew with the production post-mortems (multi-account, missing-transaction, fee-reversal cases); a **production-sampled eval** now scores a sample of real tickets weekly; eval gates in CI block changes that regress any set.
6. **Observability** — per-step tracing with request-ID correlation; dashboards for success rate, cost per ticket, loop firings; the billing anomaly became a dashboard alert.

### 9.5 The Before/After

Illustrative metrics, consistent with the mechanisms in this guide (the *shape* of the improvement is the point, not the exact numbers):

| Metric | Before | After |
|---|---|---|
| Task success rate (outcome-checked) | ~61% (and unmeasured — this is the estimate from sampling after the fact) | **~91%** |
| Silent-failure rate (sampled human review) | ~9% of sampled tickets | **~1.5%** |
| Cost per ticket (p95) | ~$1.30 (6–8× model) | **~$0.28** |
| Loop/termination events per 1,000 tickets | ~38 (step-limit kills) | **~4** (loop detector + budget caps) |
| p95 end-to-end latency | ~47s (loops dominated) | **~14s** |
| Human review load | 0% (nothing was checked) | ~18% of tickets (approval gates on high-value classes) |
| Time-to-detection of bad output | Days–weeks (billing anomaly / complaints) | **Minutes** (traces + production evals) |

The honest trade-off: HITL review on ~18% of tickets is real operational cost — but it is the price of the silent-failure reduction, and it is a *designed* cost rather than an incident-driven one.

---

## 10. The Production-Readiness Checklist

### 10.1 "Is My Agent Ready for Production?"

Nine categories, one question each, with pass criteria. If any item fails, the agent is not ready — fix the category, not the demo.

| # | Category | The question | Pass criteria |
|---|---|---|---|
| 1 | **Scope** | Is the agent's task narrow enough to be eval-able and containable? | Single well-defined task; bounded toolset; explicit out-of-scope refusal; worst-case failure is containable (§7.1) |
| 2 | **Evals** | Do evals see production reality, not just the demo? | Golden + synthetic + **production-sampled** sets; trajectory-level scoring; eval gates in CI (§8.2, §8.5) |
| 3 | **Control** | Can the agent be stopped, bounded, and recovered? | Max-step/token/dollar caps; loop detector; timeouts; circuit breakers; bounded retries (§8.2) |
| 4 | **Cost** | Is cost per task modeled, metered, and capped? | Per-task cost budget; metering/attribution; anomaly alerts; FinOps review ([../finops_guide.md](../finops_guide.md)) |
| 5 | **Latency** | Does the agent meet the UX latency contract? | p95 within budget; streaming/progress UX; step count within latency budget; degraded modes (§5.4–5.5) |
| 6 | **Security** | Are injection, tool abuse, and data exposure contained? | Input/output filters; tool allow-lists; injection test suite in CI ([llm_guard_models_guide.md](llm_guard_models_guide.md), [prompt_injection_guide.md](prompt_injection_guide.md)) |
| 7 | **Observability** | Can every run be traced and every failure explained? | Per-step traces with tool args/results; request-ID correlation; success/cost/latency dashboards (§4.3) |
| 8 | **HITL** | Are humans at the expensive decision points? | Approval gates on high-value/irreversible/high-cost actions; ownership registry; governance sign-off ([implementing-responsible-ai.md](implementing-responsible-ai.md)) |
| 9 | **Fallbacks** | Does the agent degrade gracefully when it fails? | Designed fallback path (alt model → KB → human); partial-result delivery; tested failure drills (§7.6) |
| — | **Ongoing** | Will it stay reliable next quarter? | Drift monitoring; model pinning; eval re-runs; error budgets; incident process ([ai_agent_drift_guide.md](ai_agent_drift_guide.md)) |

### 10.2 The 30-Day Path to Production Readiness

The checklist is the destination; this is the route. For a team with an agent working in a demo, the disciplined sequence — each step gated on the previous:

1. **Days 1–5: Scope and contract.** Write the one-paragraph task contract (what the agent does, what it refuses, what its worst case is). Cut every tool not needed for that contract. If the contract cannot be written in one paragraph, the scope is too wide (§7.1).
2. **Days 5–10: Eval baseline.** Build the three eval sets (golden + synthetic + a production-sampled pilot), run them, and record the *task-level* success rate. If the number is below the p^N expectation for the step count (§1.6), cut steps before proceeding.
3. **Days 10–15: Controls.** Enforce budget caps (steps/tokens/dollars), loop detection, timeouts, and tool-argument validation. Run the agent against its own failure modes from §2 and watch the controls fire.
4. **Days 15–20: Verifier + HITL + fallback.** Add the verifier step, put approval gates on high-value tool calls, and build the degraded path. Sample and human-review outputs; count silent failures (§4.2).
5. **Days 20–25: Observability.** Wire per-step tracing with request-ID correlation; stand up the success-rate, cost-per-task, and loop-alert dashboards (§4.3, §8.6).
6. **Days 25–30: Gate and launch.** Put the eval gates in CI, pin the model version, and launch to a limited cohort with the monitoring live. The first production-sampled eval cycle — not the launch — is the real go/no-go.

### 10.3 The Honest Bottom Line

- **Agents fail predictably.** The failure modes are not mysterious and not infinite: they are the compounding of per-step errors, the wrong-tool/wrong-arg/hallucinated-step task failures, the loop/runaway control failures, the integration failures of a messy world, and the eval failures of a demo-shaped safety net. Every one of them is named in this guide, and every one of them has a mitigation.
- **The known failure modes are manageable — with discipline, not with hope.** The discipline is: scope the agent, eval it against production reality, cap its budget, validate its tool calls, verify its outputs, trace its runs, gate its high-value actions, and give it a fallback. None of this is exotic; all of it is *engineering*.
- **Production agents are possible.** The surveys say over half of organizations already run agents in production — and the successful ones share the profile in §7. The compounding math is not a verdict against agents; it is the specification for how to build them. **The agents that fail in production are the ones whose owners treated the demo as the eval and the prompt as the design. The agents that work are the ones that were designed as systems, measured like services, and bounded like liabilities.**

**Final word:** if you remember one thing from this guide, remember the number **0.5987** — a 95%-reliable agent over 10 steps succeeds 60% of the time. Then design the agent that makes that number a *floor*, not a ceiling: fewer steps, verified steps, bounded steps — and a human at the steps where the money moves.

---

## 11. Glossary

- **Compounding error** — the multiplicative accumulation of per-step failure probabilities over a multi-step chain; task reliability = product of step reliabilities (Lusser's Law).
- **Per-step accuracy (per-step reliability)** — the probability that a single agent step (tool call, reasoning step, retrieval) succeeds; p in the p^N formula.
- **p^N** — the compound task success probability for N independent steps with uniform per-step success p; the core reliability model for agents.
- **Error accumulation** — the process by which small per-step errors become large task-level failure rates as step count grows; the reason long-horizon agents fail.
- **Failure taxonomy** — the enumerated, categorized set of ways an agent system fails (task, control, integration, evaluation, reliability, cost/latency, security, human-factor failures).
- **Task failure** — the agent completes its loop with a wrong outcome: wrong tool, wrong arguments, hallucinated steps, incomplete task.
- **Control failure** — the execution loop misbehaves: infinite loops, stuck states, runaway/unbounded actions, cascading errors.
- **Integration failure** — the environment breaks the agent: API changes, auth failures, flaky tools, schema drift.
- **Evaluation failure** — the eval/safety net is wrong: golden-set blindness, distribution shift, non-determinism, missing cost/latency/security/human checks.
- **Demo-vs-production gap** — the systematic difference between curated demo conditions and real production conditions (inputs, data, APIs, users, step counts, success bars).
- **Golden set** — a curated eval set of inputs with known-good outputs; a regression canary with known blindness.
- **Distribution shift** — the difference between the distribution of eval/demo queries and the distribution of real production queries.
- **Long tail** — the many individually rare, collectively inevitable edge-case failures; the unbounded set of failures no golden set can cover.
- **Non-determinism** — the stochasticity of LLM outputs (temperature, sampling) and of the surrounding environment (tool results, model updates) that makes single-run evals non-evidence.
- **Silent failure** — a wrong-but-confident output that nothing in the pipeline detects; the most dangerous agent failure class.
- **Observability** — the capability to inspect system behavior via traces, metrics, and logs; the prerequisite for SLOs, evals, and incident response.
- **Tracing** — recording each step of an agent run (prompt version, model version, tool calls, results, tokens, latency) correlated by request ID.
- **SRE (Site Reliability Engineering)** — the discipline of running services to explicit reliability objectives; applied to agents via task-level SLIs/SLOs and error budgets.
- **Error budget** — the acceptable gap between observed reliability and the SLO; burning the budget freezes feature work in favor of reliability work.
- **SLI (Service Level Indicator)** — a measured reliability signal, e.g. task success rate, cost per task, p95 latency.
- **SLO (Service Level Objective)** — a target value for an SLI, e.g. "≥95% of tickets resolved correctly."
- **Token cost** — the dollar cost of model tokens consumed per task; scales with steps × context growth.
- **Runaway loop** — an unbounded agent loop (repeat searches, escalating retries) that burns tokens without converging.
- **Budget cap** — a hard, runtime-enforced ceiling on steps, tokens, or dollars per task; the primary cost and control safeguard.
- **Latency** — end-to-end task time; accumulates per step (model time + tool time); p95 is the operational measure.
- **p95 (95th percentile)** — the latency (or cost) value below which 95% of tasks fall; the standard tail measure for UX contracts.
- **Streaming** — emitting partial results progressively so users see progress while the agent continues; the standard UX mitigation for multi-step latency.
- **HITL (Human-in-the-Loop)** — human checkpoints/approval gates at high-value decision points; a reliability and accountability feature.
- **Guardrail** — a policy-enforcing control around the agent: input/output filters, tool allow-lists, refusal logic.
- **Fallback** — a designed degraded path (alternative model, static KB, human escalation) used when the primary path fails or exceeds budget.
- **Circuit breaker** — a reliability pattern that stops calling a failing dependency and fails fast to a fallback instead of retrying into the outage.
- **Loop detector** — a component that detects repeated identical tool calls (signature hashing) and terminates/branches the loop.
- **Eval gate** — a CI check that blocks deploys (prompt/model/tool/agent changes) that regress the eval suite.
- **Success rate** — the fraction of tasks with correct outcomes (outcome-checked, not "agent said done").
- **Cost per task** — the average/p95 dollar cost of one completed task; the core FinOps metric for agents.
- **Tool allow-list** — the bounded, policy-approved set of tools an agent may call; the highest-leverage security + scoping control.
- **Verifier** — a second model or deterministic checker that validates the primary agent's output (args, groundedness, completion).
- **Router** — a cheap classifier/embedding step that dispatches each request to the right specialist (or human, or no-agent path).
- **Scoped agent** — a single-purpose agent with a narrow task, bounded toolset, and dedicated eval; the proven production unit.
- **Self-correction** — the pattern of an agent reviewing and fixing its own output; reliable only with *external* correction signals (test failures, schema errors, verifier feedback), not the model's own opinion.

---

## 12. References & Further Reading

**Repository series (cross-references):**

- [autonomous_agents_guide.md](autonomous_agents_guide.md) — umbrella: architectures §3, patterns §4, control & safety §5, evaluation §6
- [ai_agent_drift_guide.md](ai_agent_drift_guide.md) — the temporal dimension: degradation over time (the companion to this structural analysis)
- [agent_scaffolding_guide.md](agent_scaffolding_guide.md) — the code-level agent scaffold
- [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) — the platform: gateway, tracing, guardrails, cost metering
- [llm_guard_models_guide.md](llm_guard_models_guide.md) — guardrails: input/output filters, tool allow-lists, injection defense
- [prompt_injection_guide.md](prompt_injection_guide.md) — prompt injection taxonomy and defenses
- [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md) — evaluation vs validation for LLM systems
- [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) — evaluation frameworks and tooling
- [rag/rag_evaluation_methodology_guide.md](rag/rag_evaluation_methodology_guide.md) — RAG/groundedness evaluation methodology
- [implementing-responsible-ai.md](implementing-responsible-ai.md) — governance, HITL, accountability, regulatory framing
- [llm_latency_optimization_guide.md](llm_latency_optimization_guide.md) — latency: tiering, caching, streaming
- [agent_runtime_cache_design_guide.md](agent_runtime_cache_design_guide.md) — runtime caching and context management
- [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md) · [hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md) — multi-agent orchestration patterns
- [coding_agents_research.md](coding_agents_research.md) — software-engineering agents (SWE-bench, Terminal-Bench context)
- [llm_agent_use_cases.md](llm_agent_use_cases.md) — use cases and industry metrics
- [mcp_framework_tools_guide.md](mcp_framework_tools_guide.md) — MCP: tool/context standardization
- [../finops_guide.md](../finops_guide.md) — FinOps: cost metering, budgeting, chargeback
- [../durable_ai_agent_workflows_guide.md](../durable_ai_agent_workflows_guide.md) — durable execution and checkpointing
- [../../banking/financial_risk_compliance_systems_guide.md](../../banking/financial_risk_compliance_systems_guide.md) — banking risk/compliance context

**External sources (verified during research, August 2026):**

- Anthropic — *Building Effective Agents* (Dec 2024) — workflows vs agents; "the simplest solution that works"
- Carnegie Mellon + Anthropic — computer-using agents study (June 2025); widely reported conclusion: agents "make too many mistakes for businesses to rely on them for any process involving real money" (as-reported; press-reported failure rate ~85% on studied tasks)
- LangChain — *State of Agent Engineering 2025* (survey of 1,340 practitioners, Nov–Dec 2025): 57.3% agents in production; ~89% observability; ~52% testing (self-reported)
- Datadog — *State of AI Engineering* (telemetry from 1,000+ production customers, 2025–2026): corroborates production shift and reliability/observability gaps
- Terminal-Bench (2025) — terminal-task benchmark; LangChain trace analysis: false early completion the most common failure pattern
- Practitioner essays on the compounding-error framing (arithmetic verified): Lens — *The Math Behind Why Multi-Step AI Agents Fail in Production*; Growthengineer — *The Reliability Compounding Problem in Multi-Agent Systems* (Lusser's Law, 1957); zalt.me — *AI Agent Reliability: Setting Honest Accuracy Targets* (88–97% realistic per-step range); Anand Taralika — *The 10:1 Rule*; algorithme.site — *The Compounding-Error Problem*
- Practitioner failure catalogues: Tendril — *Why Agents Fail (and How to Notice)*; Niklas Heidloff (IBM) — *Why Agents and Agentic Systems Can Fail*; Medium — *Why LLM Agents Break in the Real World* (Demo Illusion vs Production Reality)
- Lusser's Law (1957) — series-system reliability = product of component reliabilities; the mathematical origin of the compounding argument

---

*End of guide. The number to remember: 0.95¹⁰ ≈ 0.5987 — and the design that makes it a floor.*
