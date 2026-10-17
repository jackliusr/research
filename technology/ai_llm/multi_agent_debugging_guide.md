# Debugging Multi-Agent AI Systems — The Handoff Is the Crime Scene

> **A 2026 deep-dive on one narrow question: what actually changes about diagnosing a failure when the run spans several agents that hand work to each other?** Almost everything written about agent observability assumes one agent in one loop. Almost everything written about multi-agent systems assumes the topology is the interesting part. Neither assumption survives contact with a failed run. This guide isolates the diagnostic delta — the specific things you cannot do, and the specific things you must build, when the failing artifact is not a single agent's reasoning but a *handoff* between agents. It defends one thesis: **a multi-agent failure is almost never where it surfaces; it is at the handoff that corrupted the context three steps earlier.** It builds a five-mode taxonomy of interaction-layer failures, re-derives trace-context propagation at the agent boundary from the W3C and OpenTelemetry specifications rather than from vendor marketing, gives a concrete 2×2 isolation procedure for separating a broken member from a correct member fed a poisoned input, and is honest in §11 about how much of the tooling actually exists today versus how much is a screenshot and a roadmap. Where the evidence complicates the thesis, the section that complicates it says so.

> **Series context and boundary (read this first).** This guide owns exactly one thing: **the diagnosis of a failure whose causal chain crosses an agent boundary.** It does not re-derive material owned by sibling guides in this repository, and it cross-references them by name throughout:
>
> - **The single-run trace is owned elsewhere.** [agentops_guide.md](agentops_guide.md) §3.1–3.2 defines the agent trace as *"the complete, structured record of one task run: every reasoning step, tool call, LLM call, and outcome, in order, with timings, tokens, and costs"*, and owns the span hierarchy TRACE → SPAN (agent run) → SPAN (step: plan / tool / verify / respond) → SPAN (LLM call or tool execution), the OpenTelemetry span model, the `gen_ai.*` semantic conventions, and the rule *"capture structure always, content selectively."* **That artifact is single-run and single-tree. This guide does not restate it**; it asks what happens when the tree becomes a forest. agentops §3.5 owns the SLI/metric set, §4 the eval harness, §5 monitoring and loop detection, §7 the tooling landscape.
> - **The single-agent failure taxonomy is owned elsewhere.** [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) (704 lines) owns the compounding-error arithmetic (§1), the task / control / integration / evaluation failure taxonomy (§2, including §2.6 on context drift, confusion and token limits), demo-versus-production (§3), reliability and observability (§4), cost and latency (§5), mitigations (§8), and the production-readiness checklist (§10). **Its taxonomy is single-agent-shaped** — each mode is a property of *one* loop. This guide does not rebuild it; it asks which failures are properties of the *interaction* and therefore invisible to that taxonomy.
> - **The success conditions and the chaos warning are owned elsewhere.** [agents_work_fall_apart_guide.md](agents_work_fall_apart_guide.md) (774 lines) owns success conditions (§1), working patterns (§2), failure conditions (§4), failure patterns — including §5.2 *"The Multi-Agent Chaos — Communication Overhead"*, which is roughly six lines — the boundary spectrum (§7), and the *"should this be an agent?"* decision framework (§8).
> - **Orchestration topology is owned elsewhere.** [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md) (854 lines) and [multi_agent_banking_guide.md](multi_agent_banking_guide.md) (802 lines) own supervisor / hierarchical / peer topologies, control patterns, coordination mechanisms, model routing, and the banking deployment and regulatory considerations over them. [hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md) §12.5 *"Debugging"* is four lines: hierarchical systems are *"harder to debug — error source identification requires traversing the hierarchy,"* mitigated by *"mandatory trace IDs on every message, structured logging with hierarchy level, replay capability, visualization tools."* That is, to be precise, the thinnest possible statement of this guide's entire subject, and it names the right three ingredients without saying how any of them works. This guide is the expansion of that paragraph.
> - **The multi-agent failure literature and the sober verdict are owned elsewhere.** [agent_skills_vs_multi_agent_systems_guide.md](agent_skills_vs_multi_agent_systems_guide.md) carries the MAST-sourced failure discussion (§6.2, §7.2) and the verdict that multi-agent is *"the most expressive and most expensive topology,"* justified only when knowledge is genuinely partitioned or review separation is a governance requirement. This guide accepts that verdict and does not relitigate it.
>
> The question was verified as unowned, not assumed. Counted across 669 `.md` files in this repository: `debug` near `multi-agent` appears in **1** file; `trace` near `multi-agent` in **5**; `agent.*attribution` in **23** (nearly all of them about *credit* attribution in banking models, not failure attribution); `root cause.*agent` in **6**; `multi.agent` in 65; `agent` in 373. The instrumentation vocabulary is everywhere. The diagnostic procedure for a cross-agent failure is nowhere. That gap is why this guide exists.

> **Verification policy.** Every proposition in this guide is **dated and attributed**: who published it, when, and of what kind. Kinds are **(paper)** for peer-reviewed publication, **(preprint)** for an unreviewed arXiv manuscript, **(standard)** for a W3C Recommendation or an OpenTelemetry specification/concept document, **(vendor post)** for a vendor engineering blog, and **(vendor documentation)** for product docs describing a shipped capability. Where a source is a vendor's claim about its own system — including any self-evaluation figure — it is labelled in the sentence that carries it and never presented as independent evidence. arXiv identifiers were resolved against the arXiv API over HTTPS with a browser user-agent; abstracts were read at source. No date, identifier, percentage, or feature-availability claim in this guide is written from memory. Claims that could not be traced to a primary source are recorded in §15 rather than asserted. **This guide prints no vendor ranking, no adoption statistic, and no benchmark of products** — for the reasons given in §14.4.

**Jack Liu Shurui, Solution Architect**

*Baseline October 2026 (written 17 October 2026). Source dates are as published; re-verify every capability statement in §11 against the vendor's own current documentation before designing a control around it.*

---

**Contents**

1. [Overview — The Decoder, the Thesis, and the Boundary](#1-overview--the-decoder-the-thesis-and-the-boundary)
2. [What Changes When There Are N Agents](#2-what-changes-when-there-are-n-agents)
3. [The Multi-Agent Failure Taxonomy — Five Interaction-Layer Modes](#3-the-multi-agent-failure-taxonomy--five-interaction-layer-modes)
4. [The Trace as a Tree Across Agents](#4-the-trace-as-a-tree-across-agents)
5. [Attribution — Which Agent Failed](#5-attribution--which-agent-failed)
6. [The Reproduction Problem](#6-the-reproduction-problem)
7. [The Instrumentation Delta](#7-the-instrumentation-delta)
8. [The Diagnosis Patterns](#8-the-diagnosis-patterns)
9. [The Evaluation Dimension](#9-the-evaluation-dimension)
10. [The Failure Modes of Debugging Itself](#10-the-failure-modes-of-debugging-itself)
11. [The State of Practice — Demonstrated, Partial, and Promised](#11-the-state-of-practice--demonstrated-partial-and-promised)
12. [The Cymbal Bank Worked Example](#12-the-cymbal-bank-worked-example)
13. [The Anti-Patterns](#13-the-anti-patterns)
14. [The Claims Audit](#14-the-claims-audit)
15. [What Could Not Be Verified](#15-what-could-not-be-verified)
16. [Glossary, Cross-References, and Closing Summary](#16-glossary-cross-references-and-closing-summary)

---

## 1. Overview — The Decoder, the Thesis, and the Boundary

### 1.1 The claim, stated plainly

A single agent fails *in* its loop. You read the loop, you find the step where the plan diverged or the tool returned something the agent misread, and you are done. The failure and its cause occupy the same trace, usually within a few spans of each other.

A multi-agent system does not fail that way. It fails at a *seam*, and the seam is not where the error appears. The agent that produces the wrong final answer is, in the overwhelming majority of real cases, behaving correctly: it was handed a context in which the wrong answer was the right answer. The corruption happened upstream, at a handoff, and every hop after it did two things simultaneously — it propagated the corruption, and it *laundered* it, converting a missing fact into fluent, confident, well-formatted prose that no longer looks like a defect.

That is the thesis this guide defends:

> **A multi-agent failure is almost never where it surfaces; it is at the handoff that corrupted the context three steps earlier.**

Three consequences follow immediately, and they organize the whole guide.

**First, the surfacing point is anti-correlated with the cause.** The agent furthest downstream is the one with the least information about the original task and the most polished output. It is the worst possible place to start looking and the only place the symptom is visible. Diagnostic effort must be spent walking *backwards*, against the direction of information flow.

**Second, the failure can be a property of the interaction rather than of any member.** No single agent's specification is violated when two agents make locally reasonable but mutually incompatible assumptions. Every member passes its own contract. The composition fails. This is not a metaphor; it is the mechanism Cognition names as *"actions carry implicit decisions, and conflicting decisions carry bad results"* (vendor post, undated page — see §15 on the date).

**Third, the artifact you need is not the one standard observability produces.** A trace of one agent is a tree. A trace of a multi-agent run is a tree *if and only if* trace context was propagated correctly at every boundary — and the boundaries are exactly where runtimes, queues, third-party tools and vendor SDKs are most likely to start a fresh trace. Where propagation is broken, you do not get a worse trace; you get *N unrelated traces* that each look internally healthy. That is the failure mode of observability itself, and it is the reason §4 exists.

### 1.2 The decoder

Nine terms carry the guide. They are defined here once, precisely, because the discourse uses several of them loosely and the looseness is where diagnoses go wrong.

**Multi-agent system (MAS).** A system in which more than one agent — an LLM autonomously using tools in a loop — contributes to a single task outcome, and in which at least one agent's output becomes another agent's input. The definitional clause is the second one. Two agents that never exchange work are two systems, not one multi-agent system, and debugging them is two independent single-agent problems.

**Trace.** A single end-to-end operation, composed of spans sharing one `trace_id`. In the OpenTelemetry model a trace is a set of spans with a common trace identifier and a parent/child hierarchy (standard, OpenTelemetry traces concept, page last modified 14 Jan 2026). The agent-specific elaboration of this — the span hierarchy, the `gen_ai.*` attributes, the content-capture policy — is owned by [agentops_guide.md](agentops_guide.md) §3.1–3.2 and is not restated here.

**Span.** A unit of work with a start time, an end time, a parent, and attributes. The agent-relevant span kinds are the agent run, the step, the LLM call, and the tool execution. A **handoff span** is the span that represents the transfer of work from one agent to another; it is the object this entire guide is about, and it is a first-class span type in at least one shipped SDK (§11.1).

**Trace context.** The minimal set of identifiers that lets a downstream process attach its work to an upstream trace: trace ID, the calling span's ID, trace flags, and vendor state. Standardized as the `traceparent` and `tracestate` HTTP headers by W3C Trace Context, a W3C Recommendation dated 23 November 2021, with `traceparent` carrying `<version>-<trace-id>-<parent-id>-<trace-flags>` (standard).

**Handoff.** The moment one agent's output becomes another agent's input. A handoff has three parts that are routinely conflated: the *payload* (what data crossed), the *context* (which trace it belongs to and what provenance it carries), and the *contract* (what the receiving agent is entitled to assume about the payload). Most production handoffs specify only the payload. The other two are where the bugs live.

**Shared state / blackboard.** A store that multiple agents read and write, as opposed to a message passed between two. The distinction matters diagnostically: a message handoff corrupts *downstream*, while a blackboard corrupts *laterally* — an agent can be poisoned by a write that happened in a branch it never interacted with, and no message trace shows the connection.

**Orchestrator / supervisor.** The agent that decomposes the task and assigns work. In diagnostic terms the orchestrator is a *suspect with special privileges*: it is the only member that sees the whole task, so it is the only member whose errors are invisible to everyone else. §3.5 is about this.

**Attribution.** The answer to two questions, not one: *which agent* produced the error, and *which input* made that agent's correct behaviour wrong. Answering only the first is how teams come to "fix" agents that were never broken.

**Emergence.** Behaviour of the composition that is not specified by, and not predictable from, any member's specification. Anthropic states the property directly for its own system: *"Multi-agent systems have emergent behaviors, which arise without specific programming… small changes to the lead agent can unpredictably change how subagents behave"* (vendor post, 13 Jun 2025). Emergence is why per-agent testing cannot be complete, and why §9's per-agent evals are an instrument rather than a guarantee.

**Replay.** Re-execution of a prior run, or a prefix of one, from recorded state. The word hides a critical distinction that §6.3 makes explicit: a replay that re-issues LLM calls is a *re-run*, not a replay, and it will diverge. LangGraph's own documentation says so in as many words — replay *"re-executes nodes—it doesn't just read from cache. LLM calls, API requests, and interrupts fire again and may return different results"* (vendor documentation, page read Oct 2026).

### 1.3 The boundary, by name

This guide is deliberately narrow, and the narrowness is enforced by four named owners. Each of the following is *cited and not re-derived*:

| What | Owner | Why this guide does not restate it |
|---|---|---|
| The single-run trace: span hierarchy, OTel span model, `gen_ai.*` conventions, content-capture policy, SLIs, monitoring, loop detection, tooling landscape | [agentops_guide.md](agentops_guide.md) §3.1–3.2, §3.5, §4, §5, §7 | The subject here begins where the single trace ends. Repeating the span model would produce a longer guide and a worse one. |
| The failure taxonomy: task / control / integration / evaluation, plus context drift, confusion and token limits; compounding math; demo-vs-production; cost and latency; mitigations and readiness checklist | [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) §1, §2 (esp. §2.6), §3–§5, §8, §10 | Its taxonomy is single-agent-shaped: every mode is a property of one loop. This guide adds only the modes that are properties of the *interaction*. |
| Success conditions, working patterns, failure conditions, the communication-overhead warning (§5.2), the boundary spectrum and the "should this be an agent?" framework | [agents_work_fall_apart_guide.md](agents_work_fall_apart_guide.md) §1, §2, §4, §5.2, §7, §8 | Whether to build the topology is settled there. This guide assumes it was built and asks how to debug it. |
| Topology: supervisor / hierarchical / peer, control and coordination patterns, model routing, banking deployment and regulatory considerations | [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md); [multi_agent_banking_guide.md](multi_agent_banking_guide.md); [hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md) §12.5 | Topology is the *shape* of the handoff graph. This guide asks what the shape costs you when you have to find a bug in it. |

One upstream fact is imported rather than re-derived, because every section depends on it: **a multi-agent pipeline is a series system, and a series system's reliability is the product of its members'.** The arithmetic is owned by [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) §1 (0.95¹⁰ ≈ 0.5987, exact arithmetic, canonical framing; the origin is Lusser's Law, 1957). This guide uses exactly one consequence of it, in §9.2, and no more.

### 1.4 The test every section had to pass

A guide about multi-agent debugging is easy to write badly: you write a good guide about debugging, add the word "agents" to the headings, and nobody notices for a year. The filter applied to every section here was a single question —

> **Could this paragraph have been written for a single agent?**

If yes, it was cut, or reduced to a cross-reference to the owner. The instrumentation checklist in §7 is a *delta* on agentops, not a repeat of it. The taxonomy in §3 defers the task/control/integration/evaluation modes entirely. The evaluation material in §9 keeps to one line of compounding math. What survives is what is genuinely different when there are N agents: the forest, the seam, the laundering, the attribution ambiguity, the combinatorics of the replay, and the fact that a correct agent can be the bug.

---

## 2. What Changes When There Are N Agents

This section is the foundation and the rest of the guide leans on it. It makes four structural claims, each of which changes a diagnostic technique rather than merely a diagnostic vocabulary.

### 2.1 The span tree becomes a forest joined by handoffs

A single agent produces one rooted tree: the run span contains step spans, step spans contain LLM and tool spans. You open the tree, you read downward, you find the step. Every debugging instinct you have — zoom into the failing leaf, walk up to the parent, look at the sibling that ran just before — assumes this shape.

A multi-agent run *should* produce the same shape with agent-run spans in the middle of it. In practice it produces a forest, and the trees in it are joined by handoffs whose strength varies from a real parent/child edge down to nothing at all. Four join qualities exist, and they are not equally useful:

| Join quality | How it appears | What you can conclude from it |
|---|---|---|
| **Parent/child across the boundary** | Sub-agent's root span is a child of the handoff span | Full causality. You can walk from symptom to cause in one traversal. |
| **Span link** | Sub-agent's trace records a link to the upstream span context | Causality is *asserted* but not hierarchically navigable. OpenTelemetry defines links for exactly this: associating spans with "a causal relationship" where parent/child is the wrong shape (standard, OTel traces concept). Sufficient for attribution, weaker for bisection. |
| **Shared `trace_id`, no parent** | Same trace, orphaned subtree | The run is findable as a unit, but the hop is not identified. You know these spans belong together; you cannot tell which output fed which input. |
| **New `trace_id`** | Sub-agent appears as a separate, internally healthy trace | Nothing. This is the silent failure of §4.3, and it makes attribution impossible in principle, not merely difficult. |

The diagnostic consequence is blunt: **the quality of your join determines the ceiling on your diagnosis, and it is decided at instrumentation time, not at incident time.** A team that discovers during an incident that its sub-agent runtime starts a fresh trace has no remedy that day. This is why §7 is short and §7.1 is the one line that matters.

### 2.2 State is passed or shared, not held

In a single agent, the context window *is* the state. It is one object, monotonically appended, and you can read it. The agent's belief at step 7 is the transcript up to step 7. This makes a whole class of question trivial: "what did it know when it decided that?" — read the transcript.

With N agents, state is either **passed** (serialized into a payload and handed across) or **shared** (written to a blackboard that others read). Both break the trivial question.

- With **passing**, the receiving agent's belief is a *function of a lossy projection*. What it knows is what the sender chose to serialize, and the sender's choice is made by an LLM under a token budget, which means the omission is not random — it is systematically biased toward whatever the sender found uninteresting. The dropped fact is precisely the fact the sender did not understand was load-bearing.
- With **sharing**, the receiving agent's belief depends on write ordering. The same agent, run twice against the same blackboard with a different interleaving, reads different state. This is a concurrency bug wearing an LLM costume, and it is invisible in any per-agent transcript because no agent ever saw an inconsistent view.

Either way, the question "what did agent 4 know when it decided that?" now requires a *reconstruction* rather than a read. That reconstruction is the payload snapshot at the handoff, and if you did not record it, the answer is gone. §7.3 makes this concrete.

### 2.3 Behaviour can emerge that no member's spec contains

Per-agent correctness does not compose. Two agents can each satisfy their contract and jointly produce an outcome neither would produce alone: a planner that asks for a summary every time it is uncertain, and a summarizer that omits uncertainty markers to keep summaries short, will loop — the summarizer's compliance removes the signal the planner needed to stop. The planner is correct. The summarizer is correct. The system does not terminate.

This is not a hypothetical shape; it is the mechanism behind Anthropic's report that early versions of its system spawned far more subagents than intended and that *"small changes to the lead agent can unpredictably change how subagents behave"* (vendor post, 13 Jun 2025). The diagnostic consequence is the sharpest one in this guide:

> **A per-agent test suite cannot, even in principle, cover the failure space of a multi-agent system, because part of that space is not a property of any agent.**

This is why §9 treats per-agent evals as a localization instrument and refuses to treat them as coverage, and why §8's procedure always includes a step that varies the *handoff* while holding both members fixed.

### 2.4 The failure can be a property of the interaction

Putting 2.3 in attribution terms: the object that fails may be neither agent. It may be the *edge*.

Consider the concrete case from Cognition's post (vendor post, undated page): two subagents are told to build parts of the same game. One builds a background in the style of a different game; the other builds a character asset in a third style. Neither agent failed. Each was given a subtask with an under-specified style, made a reasonable implicit decision, and delivered. The composition is incoherent because *the decisions were never reconciled*, and the reconciliation had no owner. Cognition's principle is the diagnosis: *"actions carry implicit decisions, and conflicting decisions carry bad results."*

The edge has no transcript. There is no agent whose reasoning you can read to find the bug, because the bug is the absence of a reasoning step that nobody was assigned to perform. **You cannot debug an edge by reading an agent.** You debug it by comparing the two payloads and the two sets of implicit assumptions — which means the edge itself must have been instrumented as a first-class object.

### 2.5 The compact table

The single-agent debugging question, and its multi-agent replacement. This is the section in one screen.

| The single-agent question | The multi-agent replacement | Why the replacement is harder |
|---|---|---|
| Which step failed? | Which **handoff** first delivered a context in which the next agent's correct behaviour was wrong? | The failing step is usually correct given its input. |
| What did the agent know at step 7? | What did agent *k* receive, in what version, after what projection, and what was dropped? | Requires a recorded payload snapshot; the transcript no longer exists as one object. |
| Where did the plan diverge? | Where did two agents' implicit decisions become incompatible? | Neither agent's plan diverged. The divergence is between them. |
| Is this run reproducible? | Reproducible at which hop, under which interleaving, with which side effects re-fired? | Determinism is per-boundary now, and §6.3 shows replay ≠ re-run. |
| Was the tool wrong? | Was the tool wrong, or did the orchestrator route a task to an agent whose tools could not serve it? | Misroute is an orchestrator error that presents as a tool error. |
| Does it pass the eval? | Which link failed, and does the end-to-end score say anything about which? | It does not. §9.2. |
| Fix the prompt. | Fix the **contract** — schema, provenance field, and who is entitled to assume what. | A prompt fix at the writer cannot restore a fact the writer never received. |

The last row is the one that saves the most money, and §12 is built around it.

---

## 3. The Multi-Agent Failure Taxonomy — Five Interaction-Layer Modes

The failure taxonomy in [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) §2 is organized by *where in the loop* something went wrong: the task, the control flow, the integration with the world, the evaluation of the result. Every one of those modes can occur in a multi-agent system, and every one of them is diagnosed the same way it is in a single agent — inside one transcript.

This section adds only the modes that cannot be diagnosed that way. **Each of the five is a property of an edge or of the graph, not of a node.** If a mode can be found by reading one agent's transcript end to end, it belongs to the failures guide and is not here.

The taxonomy is corroborated at the category level by the largest independent study available. MAST — Cemri et al., *"Why Do Multi-Agent LLM Systems Fail?"* (preprint, arXiv:2503.13657, first posted 17 Mar 2025, v3 26 Oct 2025) — derives 14 failure modes from 1,600+ annotated traces across 7 frameworks, clustered into three categories named in the abstract as **system design issues, inter-agent misalignment, and task verification**, with inter-annotator agreement κ = 0.88 over the 150 traces used to build the taxonomy. Two of its three categories are about *interaction* rather than about any individual agent's reasoning; the category the paper calls **inter-agent misalignment** is, by its name and by the study's framing, the interaction layer. **That a taxonomy derived bottom-up from real traces put a whole category — and much of its mass — on misalignment between agents is the strongest available support for this guide's framing.** (The 14 individual modes are not enumerated here: the abstract was read at source, the full text was not. §15 records the limit.)

| # | Mode | Where it lives | One-line signature | Single-agent analogue |
|---|---|---|---|---|
| 1 | **Handoff corruption** | The edge | The payload lost or mangled a fact the receiver needed | None — a single agent cannot lose a fact between two contexts it does not have |
| 2 | **Cross-agent context poisoning** | The edge, propagating | An unsupported claim becomes the next agent's premise and is never re-checked | Hallucination, but *laundered into authority* |
| 3 | **The emergent loop** | The graph | Two agents indefinitely satisfy each other's exit conditions | Control-flow loop, but no member's loop counter fires |
| 4 | **The orchestrator misroute** | The graph | Work assigned to an agent whose tools or remit cannot serve it | Wrong-tool selection, but decided one level up |
| 5 | **Attribution failure** | The observer | You cannot tell which of the above happened | Not a system failure — a *diagnostic* failure, and the one that costs the most |

### 3.1 Handoff corruption

**What it looks like in a trace.** A handoff span whose outbound payload attribute is smaller than the upstream agent's own working state, followed by a downstream agent whose reasoning is internally consistent but premised on absence. The tell is a *non-sequitur that isn't one*: agent 4 excludes a facility, and the exclusion is correct given what agent 4 was told, and what agent 4 was told is missing the figure that would have changed it. Nothing in agent 4's span is red. The handoff span is where the colour is.

**Mechanism.** The sending agent summarizes. Summarization is an LLM call under a token budget, so it performs lossy compression with no schema. Facts are dropped not by importance but by *apparent* importance to the summarizer, which has the sender's local view and not the receiver's needs. Three variants recur:

- *Truncation* — a context limit silently clips the tail of the payload. The reason this one is so common is that the two failure modes look identical downstream: a fact that was never extracted and a fact that was extracted and then dropped both arrive as silence.
- *Summarization loss* — a structured finding is compressed to prose, and structure is what carried the meaning. "Covenant 4.2 breached: leverage 4.1× vs 3.5× limit" becomes "some covenant concerns."
- *Reformatting loss* — the payload is re-rendered into a template that has no slot for the fact. The fact is not forgotten; it is *unrepresentable* in the receiving schema. This variant is the most insidious because it is deterministic, reproducible, and invisible: the sender did its job perfectly and the format ate the data.

**What distinguishes it from a single-agent failure.** A single agent's context loss is *self-inflicted and self-visible*: the agent that dropped the fact is the agent that needed it, so its subsequent reasoning shows the confusion directly. In a handoff, the agent that dropped the fact is *not* the agent that needed it, and the agent that needed it has no way to know anything was dropped. The information needed to detect the loss exists only in the comparison of two agents' states — which is exactly the comparison nobody instruments.

### 3.2 Cross-agent context poisoning

**What it looks like in a trace.** A claim appears in agent 2's output with no supporting span beneath it — no tool call returned it, no retrieved document contains it. It reappears in agent 3's input as an established fact, in agent 4's input as a constraint, and in the final output as a reason. Each hop increases its authority and decreases its provenance. By the last hop it is indistinguishable, in form, from a fact that was actually verified.

**Mechanism.** Agents are trained to be helpful and to produce coherent continuations, and a downstream agent receiving "the borrower's leverage is 4.1×" has no disposition to ask whether that was computed or guessed. Fluency is not evidence, but it is treated as evidence by every subsequent reader, human or model. The result is an *epistemic ratchet*: claims only gain confidence as they travel, never lose it. Anthropic's engineering post names the mitigation as a design pattern rather than a check — a dedicated citation agent that *"processes the documents and research report to identify specific locations for citations"* so that *"all claims are properly attributed to their sources"* (vendor post, 13 Jun 2025). The existence of a separate agent whose only job is provenance is itself an admission that the pipeline does not preserve it by default.

**What distinguishes it from a single-agent failure.** A single agent's hallucination is *contained*: it is one model's unsupported claim, and a reviewer reading that model's transcript can see it emerge from nothing. In a chain, the hallucination is *re-authored* at every hop. Each downstream agent restates it in its own voice with its own confidence, and the final text is not a hallucination at all — it is a faithful report of what the system believed. You cannot find the origin by reading the output, and you often cannot find it by reading the last agent, because the last agent was told.

### 3.3 The emergent loop

**What it looks like in a trace.** A cycle in the handoff graph — agent 2 → agent 3 → agent 2 — with each traversal's payload marginally different but semantically identical. No agent's iteration counter trips, because no agent iterates. The loop lives in the graph, and the graph has no counter. The run ends on a wall-clock timeout, a token budget, or a human, and the trace's most diagnostic feature is its *shape*: a repeating motif, not a long sequence.

**Mechanism.** Two agents each hold an exit condition that the other is responsible for satisfying. The reviewer returns work to the drafter with "add supporting evidence"; the drafter adds evidence the reviewer's rubric does not recognize as evidence; the reviewer returns it again. Both are executing their instructions correctly. The termination condition is a *joint* property that neither agent's prompt expresses, so neither can detect that it is unsatisfiable.

This is the mode that most directly refutes the "debug each agent" approach, and it is why [agentops_guide.md](agentops_guide.md) §5's loop detection — which operates on a single run's step sequence — is necessary but not sufficient here. A cross-agent loop may contain only two spans per cycle, each of which is a perfectly normal step.

**What distinguishes it from a single-agent failure.** A single-agent loop is a repetition *within* a transcript, so its count is visible in one place. A cross-agent loop's count is distributed across N transcripts, and each one shows the agent doing something slightly different each time, which looks like progress.

### 3.4 The orchestrator misroute

**What it looks like in a trace.** A dispatch span assigning a task to an agent whose tool set, data access, or remit cannot serve it. Downstream, that agent does something reasonable with the wrong instruments — searches the wrong index, calls a tool that returns a plausible-but-irrelevant result, or declines for lack of access. The failure presents *at* the agent as a competence problem and is *caused* one level up as a routing problem.

**Mechanism.** The orchestrator decomposes the task and describes the subtask. Anthropic reports this failure directly from its own system's early versions: subagents received instructions vague enough that they *"misinterpreted the task or performed the exact same searches as other agents,"* with one subagent investigating a 2021 shortage while two others duplicated work on the current one (vendor post, 13 Jun 2025). The fix they describe is not better subagents; it is a better *delegation contract* — each subagent needs *"an objective, an output format, guidance on the tools and sources to use, and clear task boundaries."* That is a specification of the handoff, again.

**What distinguishes it from a single-agent failure.** A single agent choosing the wrong tool is visible in its own trace, at the step where it chose. An orchestrator misroute is a *correct* tool choice by an agent that was given the wrong job: the sub-agent's trace shows a well-executed, useless task. To see the error you must compare the dispatch span's description against the agent's actual capability, which means you need both in one view — the capability as declared, and the task as assigned.

### 3.5 Attribution failure — the fifth mode, which is about the observer

The first four are failures of the system. The fifth is a failure of the *diagnosis*, and it deserves to be in the taxonomy because in practice it is the one that dominates: the run failed, the trace exists, and nobody can say which of modes 1–4 occurred.

**What it looks like.** A post-mortem that concludes "the writer hallucinated." A fix that adds a constraint to the writer's prompt. A recurrence two weeks later with the same root cause and a different surfacing agent.

**Mechanism.** Three compounding effects. (i) *Laundering*, as in §3.2: fluency destroys the evidence. (ii) *Symmetry*: in a chain where every agent restates its input, a claim's origin is not marked, so any hop is an equally plausible candidate. (iii) *Absence of counterfactuals*: the run happened once, and the alternative — what the downstream agent would have done with a clean input — was never observed and cannot be inferred from a single execution.

**Why it belongs here.** Because it is not a human failing that better engineers avoid; it is a structural property of the artifact. And it is measurable. Zhang et al., *"Which Agent Causes Task Failures and When? On Automated Failure Attribution of LLM Multi-Agent Systems"* (preprint, arXiv:2505.00212, first posted 30 Apr 2025, v3 2 Jun 2025), introduces the Who&When dataset — failure logs from 127 multi-agent systems annotated with the responsible agent and the decisive error step — and evaluates three automated attribution methods on it. **On that dataset, as the paper reports it, the best method reaches 53.5% accuracy at identifying the responsible agent and 14.2% at pinpointing the failure step, with some methods performing below random, and the authors report that even frontier reasoning models do not reach practical usability.** Those are the paper's own results on its own dataset and generalize to nothing else; they are cited here for one purpose only — to establish that *step-level* attribution is currently an open research problem, not an engineering checklist item. §5 and §11.2 draw the consequences.

### 3.6 Where this taxonomy stops, and one caution

Three limits, stated so the taxonomy is not over-read.

**It is not exhaustive of multi-agent failures.** MAST's first category, system design issues, contains modes that are properties of how the system was built rather than of an edge — and those are real. This taxonomy covers the interaction layer only, because that is the layer the existing literature does not cover.

**It is a taxonomy of *observable* modes, not of causes.** Handoff corruption has three mechanisms (§3.1) and one signature. A taxonomy organized by mechanism would be more precise and less usable at 2 a.m.

**The MAST corroboration is category-level, not mode-level.** MAST's 14 modes are its own; this guide's 5 are this guide's. The claim being made is narrow and defensible: that an independent, bottom-up taxonomy derived from 1,600+ real traces placed the majority of its categories on interaction rather than on individual reasoning. Anyone who needs the specific 14 modes should read the paper, not this guide's summary of its categories.

---

## 4. The Trace as a Tree Across Agents

Sections 3.1–3.5 are only diagnosable if the trace is one tree. This section is about the mechanism that makes it one tree — and it is the most strongly grounded material in the guide, because it does not come from agent vendors at all. It comes from a decade of distributed-systems tracing that solved this problem for microservices, and the agent ecosystem is currently re-learning it.

### 4.1 What a handoff must propagate

W3C Trace Context, a W3C Recommendation of 23 November 2021, defines the interoperable minimum. `traceparent` is a fixed-length header carrying `<version>-<trace-id>-<parent-id>-<trace-flags>`; `tracestate` carries vendor-specific key/value data. The specification's stated purpose is precisely the failure this guide is about: without a shared identifier, *"traces that are collected by different tracing vendors cannot be correlated"* and *"vendor specific metadata might be dropped by intermediaries"* (standard). Substitute "agent runtime" for "tracing vendor" and the paragraph is a description of a multi-agent system built from three frameworks.

OpenTelemetry's context-propagation concept page (vendor-neutral standard documentation, last modified 10 Aug 2026) states the mechanism in the form an agent implementer needs:

> When Service A calls Service B, Service A includes a trace ID and a span ID as part of the context. Service B uses these values to create a new span that belongs to the same trace, setting the span from Service A as its parent.

Substitute "agent A" and "agent B" and you have the requirement. A correct handoff propagates four things:

1. **Trace ID continuity.** The sub-agent's spans carry the run's `trace_id`. Without this there is no run, only a set of runs.
2. **Parent or link.** Either the sub-agent's root span is a child of the handoff span (full causality, navigable), or it carries a span link to it (causality asserted, not navigable). OpenTelemetry defines links for the case where parent/child is the wrong shape — a causal relationship that is not a containment relationship — which is exactly the async fan-out case in §4.3.
3. **Baggage.** Arbitrary key/value pairs propagated across boundaries. W3C Baggage and the OTel baggage concept are the standard mechanism for carrying *task* scope across a hop: which task, which tenant, which approval, which policy version. The OTel documentation is explicit about the constraint that matters for banking: *"avoid putting sensitive information (like user credentials, API keys, or PII) in baggage, as it might be logged or sent to untrusted downstream services"* (standard documentation). Baggage carries *identifiers* for scope, never the scope's contents.
4. **The handoff payload as a span attribute — by reference, not by value.** A hash, a length, a schema version, a pointer to the stored payload. See §7.3 for why by-reference and not by-value.

### 4.2 The handoff span

The handoff must be an object in the trace, not an inference from timestamps. A handoff span should carry, at minimum: `from_agent`, `to_agent`, `payload_hash`, `payload_schema_version`, `payload_bytes`, `context_tokens_in`, `context_tokens_out`, `dropped_context` (boolean, and why), and `trace_context_present` (boolean). The last is the self-check that catches §4.3's first failure.

This is not a novel design. OpenAI's Agents SDK documents that by default *"handoffs are wrapped in a `handoff_span()`"* alongside spans for agents, generations, function calls, guardrails and turns, with each span carrying `trace_id`, `parent_id` and a typed `span_data` payload (vendor documentation, page read Oct 2026). A shipped SDK treating the handoff as a first-class span is the strongest single piece of evidence that the mechanism is available rather than aspirational. §11.1 lists what else is demonstrated.

### 4.3 What breaks the tree — six mechanisms

Each of these is common, each is silent, and each produces the same symptom: a sub-agent that looks healthy in isolation.

**(i) The sub-agent runtime starts a new trace.** The most common cause. A framework's "run this agent" entry point initializes a tracer and, absent an explicit parent context, mints a fresh `trace_id`. The sub-agent is then perfectly instrumented and completely detached. Guardrail: assert `trace_context_present` at every handoff span and alert on false. This is a one-line check that most teams do not have.

**(ii) A summarized context is passed instead of a span reference.** When agent 1 hands agent 2 a prose summary, agent 2 has no way to reference agent 1's spans — the summary contains no identifiers. The trace still joins (if the runtime propagates context) but the *evidence* does not: agent 2 cannot cite, and you cannot walk from agent 2's claim back to the tool call that produced it. This is the deepest form of the problem, because the tree looks correct while the provenance is gone. Anthropic's own mitigation is architectural and worth quoting exactly: subagents *"pass lightweight references back to the coordinator"* rather than copying outputs through the conversation, which *"prevents information loss during multi-stage processing"* — the section is literally titled *"Subagent output to a filesystem to minimize the 'game of telephone'"* (vendor post, 13 Jun 2025). The vendor that runs the largest public multi-agent research system names information loss in multi-stage processing as the reason to pass references.

**(iii) Asynchronous fan-out where the parent closes before children return.** The orchestrator dispatches three sub-agents and returns. Its span ends. The children start later and finish later. A strict parent/child edge would produce children that outlive their parent — which many backends handle badly or not at all. OpenTelemetry's span-kind model addresses this directly: a `Consumer` span *"represent[s] the processing of a job created by a producer and may start long after the producer span has already ended"* (standard, OTel traces concept). The correct instrumentation is a producer span at dispatch plus a consumer span at execution, joined by a span link. The incorrect instrumentation — and the common one — is a parent/child edge that the backend truncates or the SDK drops.

**(iv) A queue or worker boundary drops headers.** Any hop through a message queue, a task queue, or a serverless invocation is a place where HTTP headers do not automatically survive. The context must be serialized into the message envelope. Celery, RQ, Dramatiq and FastAPI background tasks are named in OpenAI's own tracing documentation as the case where export needs an explicit flush (vendor documentation, Oct 2026) — a good sign that the boundary is understood, and a reminder that the boundary is where instrumentation is easiest to lose.

**(v) A third-party tool or MCP server does not propagate.** The tool call is instrumented; the tool's own internal work is not, and the tool's downstream calls start a new trace. This is acceptable — you cannot instrument a vendor's server — but it must be *recorded as a boundary*, so that a gap in the trace is explainable as a known truncation rather than mistaken for a missing span.

**(vi) Two agents write to a shared store and the write is not attributed.** The blackboard case from §1.2. Nothing breaks the tree here; the tree is fine. What breaks is causality: agent 5 read a value agent 2 wrote, and no edge exists because the connection was through storage, not through a message. Guardrail: every blackboard read/write is a span with the store key as attribute, so the connection is reconstructible by key rather than by edge.

### 4.4 What a correct tree must let you reconstruct

The test of instrumentation is not "do we have spans." It is whether three specific questions are answerable from the trace alone, without re-running anything.

**The handoff graph.** Which agent handed what to which, in what order, and how many times. This is the *shape*, and shape alone diagnoses §3.3 (a cycle is visible) and §3.4 (a dispatch to an agent that never should have received the task is visible). If you cannot draw the graph from the trace, you cannot see either mode.

**The payload version each agent saw.** For every agent run, the exact input it received — by reference to a stored payload plus a hash, so the reference is verifiable. This is what makes §3.1 diagnosable: comparing agent 3's payload against agent 2's output state is the *only* way to see a dropped fact, because the drop is an absence and absences are invisible unless you have both sides.

**The first divergence.** The earliest span where the system's behaviour stops being consistent with the task. Not the earliest *error* — there may be no error spans at all — but the earliest point at which a claim, a payload, or a routing decision became inconsistent with the ground truth. Identifying this requires the two prior reconstructions plus a ground-truth reference, which is why §8's procedure is ordered the way it is.

A trace that supports all three is a debugging instrument. A trace that supports only the first is a diagram. Most production systems today produce the second and believe they have the first.

### 4.5 The single-run boundary, restated

Everything in this section is about *joining* traces. It deliberately does not describe the internal structure of an agent run's spans — the step decomposition, the LLM-call attributes, the `gen_ai.*` conventions, the content-capture policy. Those are [agentops_guide.md](agentops_guide.md) §3.1–3.2, and they are correct as written for a single agent. The point of this section is that they are *necessary and not sufficient*: a perfectly instrumented agent, wrapped in a runtime that does not propagate context, produces a perfectly instrumented trace of nothing in particular.

---

## 5. Attribution — Which Agent Failed

### 5.1 Why the surfacing agent is the wrong suspect

The agent that produced the bad output is the last writer. It had the least information, the most compressed context, and the strongest instruction to produce something finished. It is also the only member whose output anyone looked at. Every structural property of the pipeline points suspicion at it, and every one of those properties is an artifact of position rather than of fault.

Three mechanisms make the misattribution systematic rather than occasional.

**Laundering through fluency.** As §3.2 sets out, each hop re-authors its input in its own voice. By the final hop, a hallucinated figure and a computed figure are typographically, grammatically and tonally identical. The surfacing agent's output contains no marker of provenance, so the reviewer attributes the claim to the agent that wrote it. This is not carelessness; the evidence that would correct it was destroyed upstream.

**Correct behaviour on corrupt input is indistinguishable from incorrect behaviour.** Consider an eligibility agent that receives a summary omitting a covenant figure and therefore excludes a facility. Given its input, exclusion is the right answer, and its trace shows a clean, well-reasoned, correctly-cited exclusion. The only way to see that it failed is to know what it *should* have received — and that requires the upstream payload, not the downstream trace.

**Position bias in the humans doing the reviewing.** Post-mortems start where the symptom is, because that is where the alert fired. The same bias that makes "the last service in the call chain" the first suspect in a microservice incident operates here, and it is worse, because a microservice's error is at least a stack trace whereas an agent's error is a well-formed paragraph.

### 5.2 The distinction that does the work: defect versus victim

Before any technique, one definition, because it collapses most of the ambiguity:

> **A member is *defective* if it would produce the wrong behaviour given a clean, contract-conforming input. A member is a *victim* if its behaviour was correct and its input was not.**

Attribution is not the question "which agent produced the bad output." It is the pair of questions "which agent is defective, and which input made a correct agent's behaviour wrong." A single answer is almost always incomplete, and the incomplete answer is the one that leads to a prompt tweak at the wrong agent.

This distinction has a direct cost consequence. Fixing a victim costs nothing and achieves nothing — you will add a constraint to an agent that was already behaving correctly, degrade its performance on clean inputs, and leave the corruption intact. Fixing a defect costs a code change. **The 2×2 in §8.3 exists to decide which one you are looking at before you spend either.**

### 5.3 The counterfactual test

The only technique that answers attribution *directly* rather than by inference is the counterfactual: **hold everything else fixed, swap one agent, re-run, and see whether the failure survives.**

Formally, for a pipeline of N agents that produced a wrong outcome, and for agent *k*:

1. Take the recorded input that agent *k* actually received in the failing run.
2. Replace agent *k* with a variant — a different prompt, a different model, a pinned deterministic stub, or a human — and re-run agents *k* through *N* from that input.
3. If the failure disappears, agent *k* is defective (or at least sufficient to cause it).
4. If the failure persists unchanged, agent *k* is a victim, and the cause is upstream of *k*.

This is a clean experimental design and it is expensive in three specific ways, each of which must be budgeted rather than discovered:

**It costs N re-runs for one incident.** To localize a single failure across a 5-agent chain you need up to five counterfactuals, each of which re-executes the tail of the pipeline. And a single re-run is not evidence, because §6.1 applies: you need a distribution, not a sample.

**It is non-deterministic, so "disappeared" is not binary.** A failure that vanishes in one re-run may have vanished because the sampling changed, not because the swap was causal. The honest form of the test is a *rate* comparison — failure rate under the swap versus failure rate under the baseline, on the same recorded input, over enough runs to separate the two — and that multiplies the cost by the number of runs per condition.

**It re-fires side effects.** Any tool call in the tail of the pipeline executes again. In a read-only research pipeline that is harmless. In a pipeline that writes to a core banking system, it is not. This is not a footnote: **the counterfactual test is only available to the extent that your pipeline is replayable without side effects**, which is why §6.4's recording requirements are a prerequisite for §5 rather than a separate concern. Where side effects cannot be suppressed, the counterfactual must be run against a sandboxed copy of the tool surface, and the result is a counterfactual about the *sandbox*, not about production.

### 5.4 What automated attribution can and cannot do today

The research position is more advanced than the tooling position, and both are worth stating precisely.

**What exists.** The Who&When work (Zhang et al., preprint, arXiv:2505.00212, 30 Apr 2025; v3 2 Jun 2025) formulates automated failure attribution as a task, releases 127 annotated failure logs, and evaluates three methods on it. **As the paper reports on its own dataset, the best method reaches 53.5% agent-level accuracy and 14.2% step-level accuracy, with some methods below random, and frontier reasoning models do not reach practical usability.** Those numbers are the paper's own and are not generalized here. What they establish is a *shape*: agent-level localization is roughly a coin flip plus a bit, and step-level localization is not working.

**What that means operationally.** An automated attribution tool can be a *triage aid* — it can rank suspects, and a 53.5% top-1 with a ranked list behind it is genuinely useful for cutting a search space. It cannot be an *authority*. A pipeline that closes a ticket with "attribution: agent 3" and no counterfactual is reporting a guess with a decimal point on it.

**Where the difficulty actually lives.** Two of the three reasons are structural rather than algorithmic. First, the ground truth itself is ambiguous: in a chain where agent 2 dropped a fact and agent 4 acted on its absence, "the responsible agent" is a modelling choice, not a fact — agent 2 caused it, agent 4 surfaced it, and both statements are true. Second, the decisive step is often an *omission*, and omissions have no span. A span-based attributor can only score the steps that exist; the step that did not happen is invisible to it. The third reason is scale: longer chains mean more candidate hops and more opportunities for a claim to be restated, and no retrieved source measures how attribution accuracy degrades with agent count — a gap recorded in §15.

### 5.5 Why the counterfactual is still worth it

Given the cost, the obvious question is whether to bother. The answer is a comparison, not an assertion: the counterfactual costs N re-runs, and *not* running it costs a fix at the wrong agent plus a recurrence. In the Cymbal case in §12, the surfacing agent was the writer and the defective component was a handoff contract — a prompt fix at the writer would have been cheap, plausible, ineffective, and would have consumed the credibility budget that the correct fix needed.

The disciplined version is not "run the full counterfactual on every incident." It is: **run it whenever the fix you are about to make is at a different agent than the one whose input you have not checked.** That is a cheap heuristic and it catches the majority of misattributions.

---

## 6. The Reproduction Problem

### 6.1 A multi-agent run is less deterministic than a single one, for three compounding reasons

Determinism degrades with N, and it does so multiplicatively rather than additively.

**More sampling points.** Every LLM call is a sampling event. A single agent with 8 steps has 8. A 5-agent pipeline where each agent takes 4 steps has 20, and the tail of the pipeline conditions on the head, so a divergence at agent 1 propagates through everything after it. Even at a temperature of zero, agents are non-deterministic between runs — Anthropic states this directly for its own system: agents *"make dynamic decisions and are non-deterministic between runs, even with identical prompts"* (vendor post, 13 Jun 2025; claims audit #19) — and the reasons are structural: batched inference introduces floating-point non-associativity, and provider-side versioning can change under a stable model name. **That mechanism is this guide's synthesis, not a sourced claim.**

**More scheduling interleavings.** Any concurrency in the graph — parallel fan-out, async sub-agents, a shared blackboard — introduces orderings that are not part of the specification. Two agents racing to write the same key produce different outcomes depending on who wins, and the winner is not recorded unless you record it. A single agent has one ordering by construction.

**More tool surface.** Each agent brings its own tools, so the number of external calls that can return something different grows with N. Worse, the tools are *stateful in the world*: a rate lookup at 09:59 and the same lookup at 10:01 may legitimately differ, and the pipeline's behaviour may legitimately differ with them.

### 6.2 "It failed once in ten" is a measurement, not a mystery

The single most common mis-handling of a multi-agent failure is treating a low reproduction rate as a property of the system rather than as a gap in the investigation. It is worth doing the arithmetic explicitly, because the arithmetic changes the decision:

- A 10% per-run failure rate means the probability that ten consecutive runs all pass is 0.9¹⁰ ≈ 0.35. **Roughly a third of the time, ten clean runs is exactly what a 10% bug looks like.** "I couldn't reproduce it in ten tries" is therefore not evidence of rarity; it is the expected observation for a bug that will hit one run in ten forever.
- A 10% failure rate is not a 10% problem. It is a *deterministic* bug whose trigger you have not identified — a race, a specific input shape, a particular length at which truncation bites, a model output that only occasionally takes a branch. The correct framing is not "flaky"; it is "deterministic, with an unidentified trigger."

What a low reproduction rate should trigger, concretely:

1. **Record the denominator.** How many runs, over what period, with what input distribution. A rate without a denominator is an anecdote.
2. **Capture the failing trace, not a reproduction.** The failing run already happened and its trace is the primary evidence. Trying to reproduce before reading the trace you already have is the most common wasted day in this discipline.
3. **Treat it as a distribution and set a determinism budget.** "Runs are reproducible within X% at temperature 0 with tools pinned" is a testable property; "runs are reproducible" is not, and believing it leads directly to §10.4.
4. **Sample the trigger, not the run.** If the failure correlates with input length, document count, or a particular agent's step count, the useful experiment is a sweep over that variable, not more uniform re-runs.

### 6.3 Replay is not re-run, and the difference is the whole point

The word "replay" is used for two different things and the confusion is expensive.

**True replay** re-executes the *orchestration* against recorded model outputs and recorded tool responses. Nothing is sampled; nothing is called. The run is deterministic because the stochastic components have been replaced by their recorded results. This is what VCR-style cassettes do at the HTTP layer, and it is the only form that supports a genuine counterfactual: if you change agent 3's prompt and replay, any difference in outcome is attributable to the prompt.

**Re-run from checkpoint** restores state and then executes forward, re-issuing model calls and tool calls. It is what most frameworks mean by "replay," and it is not deterministic. LangGraph's time-travel documentation is unusually candid about this: replay *"re-executes nodes—it doesn't just read from cache. LLM calls, API requests, and interrupts fire again and may return different results"* (vendor documentation, page read Oct 2026). The same page documents a second, sharper limitation that maps directly onto sub-agents: a subgraph without its own checkpointer is treated by the parent as *"a single super-step,"* so *"you cannot time travel to a point between nodes in a default subgraph — you can only time travel from the parent level."* **The granularity of your checkpoint determines the granularity of your diagnosis**, and a sub-agent without its own persistence is a black box you can only re-run whole. That is a design decision with a debugging cost, and it is rarely made with that cost in view.

Practically: **use true replay for hypothesis testing and checkpoint re-run for exploration**, and never confuse the two when you are about to claim a cause. A cause established by checkpoint re-run is a cause established on a different sample.

### 6.4 What must be recorded for a replay to mean anything

Nine items. The list is short because most of it is cheap, and the items that are not cheap are the ones that decide whether replay is possible at all.

| # | Item | Why it is required | Cheap? |
|---|---|---|---|
| 1 | Model identifier **and version** | A silent provider upgrade invalidates every prior replay | Yes |
| 2 | Full prompt **as sent**, post-templating | The template is not the prompt; variables, system text and tool schemas all matter | Yes (storage cost) |
| 3 | Decoding parameters, and seed where the provider exposes one | Temperature/top-p differences change the sample; seeds give partial reproducibility where offered | Yes |
| 4 | Every tool request **and** response, verbatim | The tool surface is where the world enters; without it, replay calls the world | Yes (volume) |
| 5 | Shared-state snapshots at each handoff | The only way to reconstruct "what did agent *k* know" | Moderate |
| 6 | The topology version | Which agents, which prompts, which edges — a graph is code and it changes | Yes (git SHA) |
| 7 | Wall-clock-dependent inputs | Dates, rate effective-dates, "today" in a prompt, cache TTLs, token expiry | Often missed |
| 8 | Ordering of concurrent writes | Without it, the interleaving is unreconstructible | Hard |
| 9 | Payload schema version per handoff | Distinguishes "the fact was dropped" from "the receiver's schema had no slot for it" | Yes |

Items 7 and 8 are the ones that quietly ruin replays. A pipeline that injects today's date into a prompt is not replayable tomorrow, and the failure it produces will look like a model regression. A pipeline with unsynchronized parallel writers is not replayable at all without a recorded ordering, and the failure it produces will look like a flake.

### 6.5 The honest limits

Three things replay cannot fix, stated so that no one promises them:

- **Server-side model nondeterminism.** You cannot replay a model you cannot pin. Where the provider exposes a seed, it is a partial mitigation; where it does not, true replay requires recorded outputs, which means the replay tests the *orchestration* and not the *model*.
- **Tool side effects.** A replayed payment is a second payment. Tools must be stubbed, and a stubbed tool is a different tool — its malformed responses, latencies and partial failures are exactly what you needed to reproduce.
- **Timestamp and PII divergence.** Recorded payloads contain personal data; the recording itself becomes a regulated artifact. And a payload scrubbed for storage is no longer the payload that ran. The usual resolution is to record hashes plus a secured reference for the content, which preserves verifiability while moving the compliance problem to the store — which is the right place for it, and is not free.

---

## 7. The Instrumentation Delta

This section is deliberately short. [agentops_guide.md](agentops_guide.md) §3.1–3.2 owns what to capture inside an agent run and how; §3.5 owns the metric set. What follows is only what must be *added* for the tree, the attribution and the replay in §4, §5 and §6 to exist. If an item here duplicates something agentops already requires, it has been cut.

### 7.1 The three fields that decide whether anything else works

Before any of the rest: `trace_id` continuity, `parent_span_id` or `span_link`, and a boolean asserting that trace context arrived. The third is the one teams omit, and it is the only one that fails loudly. An instrumentation that records `trace_context_present=false` at a handoff has converted an invisible architectural defect into a counter you can alert on.

### 7.2 Per-handoff attributes

On the handoff span, not on either agent's span — the edge is the object:

| Attribute | Type | What it diagnoses |
|---|---|---|
| `handoff.from_agent` / `handoff.to_agent` | string | The graph shape; §3.3 cycles, §3.4 misroutes |
| `handoff.payload_hash` | string | Verifiable reference to the stored payload; the basis of §3.1 comparison |
| `handoff.payload_schema_version` | string | Reformatting loss; contract drift across deploys |
| `handoff.payload_bytes` | int | Truncation; a payload far smaller than the sender's state is the signature |
| `handoff.context_tokens_in` / `_out` | int | The projection ratio. A large drop is where facts go |
| `handoff.dropped_context` | bool + reason | Truncation versus summarization versus schema (§3.1) |
| `handoff.contract_id` | string | Which handoff contract was in force; makes contract changes testable |
| `handoff.trace_context_present` | bool | §4.3(i), the silent detachment |
| `handoff.orientation` | enum: request / response / critique | Distinguishes a forward handoff from a review loop; §3.3 needs it |

Nine attributes on one span. That is the whole delta, and it is a morning's work per boundary — which is exactly why the absence of it is a choice rather than a constraint.

### 7.3 Payload by reference, never by value

The payload should be stored once, addressed by hash, and referenced from the span. Three reasons, in order of importance:

1. **Size.** A payload copied into span attributes will blow attribute limits and cost, and will be truncated by the backend — producing a trace that *looks* complete and is not.
2. **PII.** Trace backends are not usually in the data-classification scope that the payload store is. Putting the payload in the span silently reclassifies your telemetry as customer data.
3. **Comparability.** A hash makes "is this the same payload as last time" a cheap operation, which is what §8.4's freeze-and-diff depends on.

The pattern is not agent-specific; it is the same reasoning behind Anthropic's *"pass lightweight references"* design (§4.3(ii)) and behind content-addressed storage generally.

### 7.4 The one metric worth adding

Beyond agentops §3.5's SLI set, the multi-agent-specific indicator is **handoff fan-out per run and payload shrinkage per hop**. Fan-out catches the runaway-dispatch pattern that Anthropic reports from its own early system (subagents spawned for simple queries); shrinkage catches §3.1 before it produces an incident. Both are derivable from the handoff spans above, which is the argument for putting them there rather than in a separate pipeline.

---

## 8. The Diagnosis Patterns

This is the practical core. Five patterns, then a decision procedure, then a worked mini-example.

### 8.1 Bisect the pipeline

Cut the chain in half and replay from the midpoint with a known-good downstream.

Concretely: for a 5-agent pipeline that produced a wrong answer, take the recorded input to agent 3 (the midpoint), feed it to agents 3–5 in *true replay* mode with downstream agents pinned to their known-good behaviour, and check whether the wrong answer still appears.

- **It appears.** The cause is at or after the midpoint. Bisect the tail.
- **It does not appear.** The cause is before the midpoint. Bisect the head.

This is binary search over the handoff graph and it costs O(log N) replays rather than O(N). It requires exactly two things: recorded payloads at each handoff (§6.4 item 5) and the ability to run the tail in true replay (§6.3). Without the first you cannot start; without the second the bisection is measuring noise.

The subtlety that makes bisection work here and not in ordinary code: **"known-good downstream" is defined behaviourally, not structurally.** You are not asserting that agents 3–5 are correct; you are asserting that *given a clean input they produce the right answer*, which is a property you established by testing them on clean inputs (§9.1). If you have not established that, the bisection has no reference point.

### 8.2 Isolate the handoff — the 2×2

The pattern that separates a poisoned input from a faulty member. Take agent *k* and run it four times, crossing two variables:

| | **Input from the failing run** | **Clean input (known-good)** |
|---|---|---|
| **Agent *k* as deployed** | **A.** Reproduces the failure | **C.** Correct? |
| **Agent *k* stubbed / pinned to known-good** | **B.** Correct? | **D.** Baseline |

Read the four cells:

- **A fails, B passes, C passes.** The member is fine and the input is bad. *Agent k is a victim.* The cause is upstream — walk back one hop and repeat at agent *k−1*.
- **A fails, B fails, C fails, D passes.** The member is defective. *Agent k is the defect.* Fix agent *k* (prompt, model, tools, contract), then re-check whether the upstream poisoning still matters — it may, and may have been masked.
- **A fails, B passes, C fails.** Both. The input is poisoned *and* the member is fragile to that poisoning. This is the most common real result and the most useful one, because it gives you two independent fixes with different costs.
- **A and C both pass.** You have not reproduced. Either the failure is elsewhere (§8.1 was wrong about the midpoint) or the run is non-deterministic and one sample per cell is not evidence (§6.1). Repeat with a rate, not a single run.

Cell B is the one people skip, and it is the one that answers the question. Stubbing agent *k* — replacing it with a fixed correct output for that input — is how you hold the member constant while varying only the input's provenance.

### 8.3 Swap an agent

The counterfactual of §5.3, applied as a routine rather than an escalation. The variant can be a different prompt, a different model, a pinned stub, or a second implementation of the same contract. What matters is that exactly one agent changes.

The diagnostic value is asymmetric: a swap that *fixes* the failure localizes the defect to the swapped agent or its contract; a swap that *changes nothing* localizes the cause upstream of it. When the pipeline is expensive, swap the cheapest agent first — not because it is likeliest to be at fault, but because a null result eliminates an entire downstream region cheaply.

### 8.4 Freeze-and-diff

Hold the downstream fixed and vary only the handoff payload. Take the recorded payload from the failing run and the payload from a passing run of the same task, diff them structurally, and identify the fields that differ.

This is the fastest way to find §3.1, because handoff corruption is a *difference between two payloads* and nothing else in the trace shows it. It is also the pattern that most often produces the "oh" — the diff is usually one field, and the field is usually one the team did not know was load-bearing.

Preconditions: payloads stored by reference with hashes (§7.3), a passing run of a comparable task, and a structural (schema-aware) diff rather than a textual one. A textual diff of two prose summaries will show every sentence as changed and tell you nothing; a diff over a structured payload shows the one missing key.

### 8.5 Replay from checkpoint at each hop

The exploration pattern. Walk the chain hop by hop, re-running from each checkpoint with everything downstream re-executed, and watch for the first hop where the outcome stops matching the ground truth. This is the pattern that most frameworks' checkpointing actually supports (§11.3), and it is the one to reach for when you do not yet know where to bisect.

Its limitation is §6.3: a checkpoint re-run is a different sample, so the "first divergence" it finds may be a sampling difference rather than a cause. Use it to *generate hypotheses*, then confirm with true replay or the 2×2.

### 8.6 The decision procedure

Ordered, cheapest-first. Stop as soon as the answer is established.

1. **Read the trace you have.** Do not reproduce first. Identify the surfacing span and the handoff graph. If the graph is a forest (§4.3), stop and fix instrumentation — you cannot diagnose further, and everything below requires the tree.
2. **Establish ground truth.** What was the correct answer, and which specific claim in the output is wrong? A diagnosis needs a named wrong claim, not "the memo was bad."
3. **Walk back one hop at a time, asking one question per hop:** *would this agent's output be correct if its input had been correct?* If yes at every hop, the cause is in a payload — go to 5. If no at hop *k*, agent *k* is the suspect — go to 4.
4. **Run the 2×2 at hop *k*.** This decides defect versus victim. Victim → go back one hop and repeat 3. Defect → you have your answer; fix and re-verify that the upstream poisoning is also addressed.
5. **Freeze-and-diff the payloads.** Find the field that differs between the failing and a passing run. That field is the corruption.
6. **Confirm with true replay** — not checkpoint re-run. Change one thing and show the outcome changes.
7. **Fix the contract, not the prompt** (§12.6), then re-run the per-handoff eval (§9.3).

### 8.7 A worked mini-example, at the level of the pattern

A three-agent pipeline: planner → retriever → writer. The writer produced a report citing a policy that does not apply. The team's first instinct is the writer.

- Step 1: the trace is one tree; the handoff graph is planner → retriever → writer, three handoffs, no cycles.
- Step 3: would the writer's output be correct if its input had been correct? Read the writer's input payload: it contains "Policy B applies (ref: P-2024-11)". Given that, the citation is correct. The writer is a victim. Go back one hop.
- Step 3 again, at the retriever: would the retriever's output be correct if its input had been correct? The retriever's input was "find the applicable policy for a 7-year facility." The retriever returned Policy B, which applies to facilities of 5 years or less. Given a query with no term in it, that is a defensible retrieval. The retriever is *also* a victim — of a query that dropped the term.
- Step 3 again, at the planner: the planner's input was the user's request, which said "7-year facility." The planner's output query omitted the term. **The planner is the defect.**
- Step 5 confirms it: diffing the planner's output payload against a passing run shows `term_years: null` in the failing run and `term_years: 7` in the passing one.
- Step 7: the fix is not "planner, always include the term." It is a handoff contract — the query schema requires `term_years` as a non-null field, so an omission fails validation at the boundary instead of becoming a plausible retrieval three hops later.

Two things are worth noticing. The corruption was a *dropped field*, not a wrong one — nothing in the trace was ever wrong, only absent. And the fix moved the failure from "silent, three hops downstream" to "loud, at the boundary" — which is the general shape of every good fix in this discipline.

---

## 9. The Evaluation Dimension

### 9.1 Per-agent evals as a localization instrument

An end-to-end eval answers "did the system work." A per-agent eval answers "does this agent, given a clean contract-conforming input, produce a correct output." Those are different questions, and only the second is usable during an incident.

The reason is that the per-agent eval is the *reference point that makes §8.1's bisection possible*. "Known-good downstream" is not an assumption; it is a test result. A team with per-agent evals on clean inputs can bisect a pipeline in O(log N) replays. A team without them can only bisect against intuition.

Two properties a per-agent eval needs to be useful this way:

- **It grades the agent against its own contract**, not against the end outcome. Agent 3's job is to produce a valid eligibility determination from a valid facility record; whether the memo that eventually cites it is good is agent 5's problem.
- **It includes adversarial inputs drawn from real upstream corruption.** Not synthetic noise — actual truncated payloads, actual omitted fields, actual prose summaries where structure was expected. An agent that handles a truncated payload by *refusing* is better than one that handles it by confabulating, and only an eval with those inputs can tell you which you have.

### 9.2 What an end-to-end score cannot tell you

One line of arithmetic, and the citation does the rest. Five links at 95% each give 0.95⁵ = 0.7738 — about 77% end-to-end (arithmetic; the compounding framing and its caveats are owned by [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) §1).

The number is not the point. The point is what the end-to-end score is *silent* about: it reports the product, and the product contains no information about which factor degraded. A pipeline at 77% is consistent with five links at 95%, with one link at 73% and four at 100%, and with every distribution in between. Worse, the product is *causally ambiguous in the direction that matters*: a regression in the end-to-end score tells you the composition got worse and tells you nothing about where, and the first place you will look is the last agent, which is the least likely culprit.

This is the evaluation-layer statement of the thesis, and it is why §9.3 exists.

### 9.3 Eval of the handoff

The unit that neither per-agent nor end-to-end evaluation covers: **does the payload preserve the facts the next agent needs?**

A handoff eval takes a set of (upstream state, downstream requirement) pairs and grades the projection. Concretely, three testable properties:

- **Fact preservation.** For each fact the downstream agent's contract declares as required, is it present in the payload? This is a schema check plus a content check, and it catches §3.1 deterministically.
- **Provenance preservation.** For each claim in the payload, is there a reference to the span or artifact that supports it? This catches §3.2 at the boundary rather than three hops later, and it is the property that Anthropic's citation-agent pattern exists to restore post hoc (§3.2).
- **No silent widening.** Does the payload assert anything the upstream agent did not actually establish? A summary that upgrades "the document suggests" to "the document states" is a handoff defect even though every word is defensible.

Handoff evals are cheap, they run without a model in most cases, and they fail at the boundary where the fix is. They are the single highest-return addition to a multi-agent test suite, and they are almost never present — because the mental model of "a test suite" is per-component, and the handoff is not a component.

### 9.4 What this changes about eval design

Three shifts, stated compactly:

**From outcome to contract-plus-outcome.** The end-to-end score stays as the release gate. Per-agent and per-handoff evals are added as the *diagnostic* layer, and they are run on every pipeline change, not only on release.

**From "did it pass" to "where did it degrade."** An eval suite that reports one number is a gate. An eval suite that reports per-hop pass rates is an instrument, and only the second one shortens incidents.

**From clean inputs to realistic corruption.** The inputs that matter are the ones the system actually produces — truncated, summarized, reordered, and paraphrased. An agent evaluated only on clean inputs is an agent whose failure modes you have never seen.

---

## 10. The Failure Modes of Debugging Itself

The failures in §3 are failures of the system. These are failures of the *investigation*, and they are more common, more expensive, and harder to notice — because a bad diagnosis produces a fix that appears to work.

| # | Symptom | Cause | Guardrail |
|---|---|---|---|
| 1 | Fix applied at the surfacing agent; same class of failure recurs | Attribution stopped at the last writer (§5.1) | The 2×2 in §8.2 before any prompt change |
| 2 | Weeks of "added more logging"; trace still cannot answer §4.4's three questions | Adding volume instead of propagation | One assertion: `trace_context_present` at every handoff |
| 3 | Every agent passes its own eval; the pipeline still fails | Locally correct members, incompatible implicit decisions (§2.3) | Handoff evals (§9.3); vary the edge, not the node |
| 4 | Replay "reproduces" the bug, the fix works in replay, production unchanged | A checkpoint re-run is a different sample (§6.3) | True replay for confirmation; label which mode was used |
| 5 | The fix suppresses the symptom at the point of detection | Detector widened, corruption untouched | Re-run the corrupt input through the fixed pipeline, not the clean one |
| 6 | Post-mortem names two agents; nobody can say which | "Which agent" asked as a single question (§5.2) | Ask for the pair: defect, and the input that made a correct agent wrong |

### 10.1 Fixing the wrong agent

**Symptom.** A prompt constraint is added to the writer. The memo stops saying the wrong thing in the specific way it said it. Three weeks later the same memo excludes a different facility for a differently-worded wrong reason.

**Cause.** The fix was applied where the symptom was legible, which is where the output is, which is the worst place to apply it. The writer was not defective; it faithfully reported an exclusion it was told to report. Constraining it does not restore the fact it never had — it only changes the wording of the wrong answer, which is worse, because the wrong answer is now harder to recognize.

**Guardrail.** The rule from §5.5: no fix at an agent whose input you have not inspected. It costs one payload read.

### 10.2 Adding logging instead of fixing propagation

**Symptom.** Log volume triples. The team can now see every prompt and every response. The trace still cannot answer "which agent's output did agent 4 actually receive" — because the logs of five agents are five separate streams with no join key.

**Cause.** The instinct that more data resolves ambiguity. It does not, when the ambiguity is *relational*: the missing information is not a fact about any agent, it is the identity relation between one agent's output and another's input. Logging cannot express a relation; trace context can.

**Guardrail.** Instrument the boundary before instrumenting the interior. A single `traceparent` propagated correctly is worth more than a full prompt log per agent, because it is the only thing that makes the prompt logs joinable.

### 10.3 The locally correct agent whose correctness is the bug

**Symptom.** Every agent's eval passes. Every agent's trace is clean. The pipeline fails, intermittently, and nobody can reproduce it because there is nothing to reproduce — each execution is correct.

**Cause.** §2.3's emergence. The failure is a joint property. Two agents, each satisfying its contract, jointly produce an outcome neither would produce alone: a summarizer that omits uncertainty to keep summaries short, feeding a planner that uses uncertainty as its stop signal, in a loop that terminates only on a token budget.

**Guardrail.** Two things, and neither is a code change. First, an *interaction* test: run the two agents against each other with their real contracts and a small input set, and assert on termination and on the joint invariant, not on either output. Second, accept that this class cannot be covered by per-agent tests (§2.3) and therefore budget for it in incident response rather than pretending the suite covers it.

### 10.4 A replay that silently differs from the original run

**Symptom.** The bug reproduces in replay. The fix works in replay. It does not work in production.

**Cause.** The replay was a checkpoint re-run, not true replay. LangGraph's documentation says this happens by design: replay re-executes nodes and *"LLM calls, API requests, and interrupts fire again and may return different results"* (vendor documentation, Oct 2026). If the re-executed LLM call happens to take a different branch, your "reproduction" is a different run that happens to look similar, and your "fix verified" is a fix verified on a run that never had the bug.

**Guardrail.** Label every replay with the mode that produced it. If a fix is claimed on the basis of a checkpoint re-run, the claim is a hypothesis. The confirmation must be true replay — recorded model outputs, recorded tool responses, nothing sampled. Where true replay is impossible (no recorded outputs), say so in the post-mortem rather than letting the word "replay" carry an assurance it has not earned.

### 10.5 Suppressing at the detector

**Symptom.** The incident stops recurring. The class of error does not.

**Cause.** The cheapest fix is always at the last point where the error is visible, and the last point is a detector — a validator, a guardrail, a review step. Widening the detector to catch the specific string makes the specific string disappear.

**Guardrail.** After any fix, re-run *the corrupt input*, not the clean one. If the corrupt input now produces a correct answer, the corruption was addressed. If it now produces a *different* wrong answer that the detector does not catch, you have moved the symptom and left the cause.

### 10.6 The two-agent post-mortem

**Symptom.** "The retriever gave the writer a bad policy, and the writer cited it." Both named, neither fixed, and the same failure recurs with the roles reversed.

**Cause.** "Which agent failed" was asked as a single question (§5.2). In a chain, the answer is a pair: the agent that is defective, and the input that made a correct agent behave wrongly. Naming both without distinguishing them produces an action list with two items where only one is a fix and the other is a symptom.

**Guardrail.** Every post-mortem states the pair explicitly and labels each: *defect* (would fail on clean input) or *victim* (correct given what it received). A post-mortem with two unlabelled agents has not finished.

---

## 11. The State of Practice — Demonstrated, Partial, and Promised

Three claims are made here and no more: what demonstrably works today, what works partially, and what does not exist. **No product is ranked, no benchmark is printed, no adoption figure is cited.** The reason is in §14.4, and it is not modesty — a feature comparison of debugging tools is not a diagnosis, and printing one would imply an evidence base that does not exist.

### 11.1 Demonstrated: a trace tree that spans agents

The capability exists and is documented, not promised.

- **Handoffs as first-class spans.** OpenAI's Agents SDK documents `handoff_span()` alongside `agent_span()`, `generation_span()`, `function_span()`, `guardrail_span()` and `turn_span()`, all carrying `trace_id` and `parent_id` and a typed `span_data` payload, with custom trace processors able to route the data to a third-party backend (vendor documentation, Oct 2026). Whatever else is true, the *object* this guide argues for — the handoff as an instrumented span — is shipping.
- **Trace context as a standard.** W3C Trace Context is a Recommendation (23 Nov 2021) and OpenTelemetry's default propagator uses its headers (standard documentation). This is the least glamorous and most important fact in the guide: the mechanism for joining a cross-agent trace is a five-year-old web standard, not a research problem.
- **Checkpointing with inspectable state.** LangGraph exposes `get_state_history`, replay from a checkpoint, and fork via `update_state` — which is what makes §8.5 possible at all (vendor documentation, Oct 2026).
- **Backend-agnostic ingestion.** The observability backends in this space are reached either by an OpenTelemetry export or by an SDK's trace-processor hook — OpenAI's documentation lists a long set of such integrations, and states that inclusion is not an endorsement (vendor documentation, Oct 2026). The practical consequence is that instrumentation is separable from backend choice, which is the reason to instrument to the standard rather than to a vendor.

### 11.2 Partial: automated attribution

Research-grade, useful as triage, not usable as authority. The Who&When results (§3.5) are the clearest statement available: **on that paper's own dataset, 53.5% agent-level and 14.2% step-level accuracy for the best of three methods, with some below random.** TRAIL — Deshpande et al., *"TRAIL: Trace Reasoning and Agentic Issue Localization"* (preprint, arXiv:2505.08638, 13 May 2025; v3 23 Jun 2025) — supplies a complementary data point from the other direction: it builds a formal taxonomy of error types in agentic traces and 148 human-annotated traces drawn from both single- and multi-agent systems, and reports that long-context models perform poorly at trace debugging, with the best model scoring 11% on its benchmark. Again: the paper's own result on its own dataset, not a general claim.

Read together, and without over-reading either, the shape is consistent: traces are now *collectable* across agents, and *interpretable* across agents is still an open problem. A tool that localizes a failure to an agent is a search-space reducer. A tool that localizes it to a step is, today, mostly guessing with a confidence score.

### 11.3 Mostly promise: deterministic replay of a multi-agent run

This is the honest gap, and it is a gap in the *product category*, not merely in this guide's research.

- Checkpoint replay exists and re-executes (§6.3). It is not deterministic and does not claim to be.
- Record-and-replay at the provider boundary (cassette-style) exists as a testing technique, but it is a technique a team assembles, not a capability a multi-agent platform advertises.
- **No vendor documentation retrieved for this guide describes deterministic replay of a complete multi-agent run** — model calls, tool calls, concurrency ordering and all — as a shipped feature. That absence is recorded in §15, and it should be read as "not found in this pass," not as "does not exist anywhere."
- The blockers are structural, not commercial: server-side sampling that cannot be pinned, tools whose effects cannot be re-fired, and concurrent writes whose order was never recorded (§6.4 items 8–9).

### 11.4 What this means for a build decision

A defensible position, stated as a position rather than a finding:

1. Instrument to W3C/OTel, not to a vendor SDK's proprietary format, even if you use the vendor's SDK. It is the only decision here that is expensive to reverse.
2. Require the handoff span with the §7.2 attributes. It is the one instrumentation delta that no framework gives you for free and every diagnosis needs.
3. Treat automated attribution as triage and keep the 2×2 (§8.2) as the authority.
4. Do not plan on deterministic replay. Plan on recorded payloads plus true replay at the orchestration layer, and accept that model behaviour is sampled.
5. Budget the side-effect suppression *before* the incident, because the counterfactual test (§5.3) is unavailable in a pipeline that cannot be re-run safely, and the incident is the wrong time to discover that.

---

## 12. The Cymbal Bank Worked Example

**Cymbal Bank is a fictional institution, and everything in this section is illustrative.** No real institution's system, incident, or documentation is described. The pipeline is simplified to the minimum that still shows the diagnostic structure.

### 12.1 The system and the wrong answer

Cymbal Bank runs a multi-agent pipeline that assembles a corporate-lending credit memo for a facility request. Five agents:

| Agent | Job | Output |
|---|---|---|
| **Supervisor** | Decompose the request, dispatch, assemble | Final memo package |
| **Extraction** | Pull structured facts from the document set: facility terms, financial covenants, financials | `FacilityFacts` payload |
| **Eligibility** | Apply credit-policy eligibility rules to the facts | `EligibilityDetermination` |
| **Covenant-check** | Verify each financial covenant against the compliance certificate | `CovenantAssessment` |
| **Memo-writer** | Compose the memo, citing determinations | Memo text |

The task: a USD 40m revolving facility for borrower B-4471. The document set is a 42-page facility agreement, audited FY2025 statements, and a compliance certificate.

The wrong answer: the memo excludes the facility from the eligible population, stating — *"The facility is excluded because the documentation does not contain a leverage covenant."*

That sentence is false. The facility agreement contains a net leverage covenant of 4.25× at clause 22.3, and the actual leverage is 3.9× — headroom of 0.35×, comfortably inside policy. The facility should have been eligible. A credit officer caught it before the committee, which is the only reason this is an incident report and not a loss.

**The wrong answer surfaces at the writer.** That is where every reviewer starts, and it is the wrong place to start.

### 12.2 The trace evidence, hop by hop

*Hop 0 — Supervisor → Extraction.* Dispatch span: `from=supervisor to=extraction`, task "extract facility terms and financial covenants." Clean; no finding.

*Hop 1 — Extraction → Eligibility.* The handoff span carries a `FacilityFacts` payload with `covenant_schedule` containing **six** rows: interest cover 2.8×, minimum tangible net worth, restrictions on disposal, cross-default, negative pledge, change of control. The leverage covenant is **not among them**.

What the extraction agent's own final step shows: its working context held **nine** covenant rows, including `net_leverage: limit 4.25x`. The payload it emitted contains six. `payload_bytes` out is roughly a third of the bytes its own extraction step produced. **The projection dropped three rows, and the leverage covenant was one of them.** The extraction agent did not fail to find it — it found it, held it, and lost it in serialization under a token budget on the covenant table.

*Hop 2 — Eligibility → Covenant-check.* The eligibility agent's determination: *"Excluded. Financial covenants evidenced: interest cover, tangible net worth, disposals, cross-default, negative pledge, change of control. **Leverage covenant not evidenced.** Policy 4.2 requires all financial covenants evidenced with ≥0.25× headroom; condition not met."*

Read that carefully. **This is correct behaviour on the input it received.** Policy 4.2 says what it says; the leverage covenant genuinely was not in the payload; the determination follows. The eligibility agent is a victim, and its trace is clean — every rule citation resolves, every step is justified.

*Hop 3 — Covenant-check → Memo-writer.* The covenant-check agent's assessment contains a sentence with **no supporting span beneath it**: *"No leverage covenant identified in the documentation."* No tool call returned that. No retrieved passage says it. The agent converted an absence in its input into an affirmative claim about the document set. This is §3.2 exactly: a claim manufactured from a gap, stated with the confidence of a claim manufactured from evidence.

*Hop 4 — Memo-writer → output.* The writer received an `EligibilityDetermination` marked "excluded" and a `CovenantAssessment` asserting no leverage covenant exists. It wrote a memo that reports both, accurately, in good prose. **The writer is a victim twice over**, and it is the agent a reviewer would have blamed.

### 12.3 The 2×2, applied at the eligibility agent

Run §8.2 at hop 2, because that is the first hop where the output is wrong in a way that matters:

| | **Input from the failing run** | **Clean input (all nine covenants)** |
|---|---|---|
| **Eligibility agent as deployed** | **A.** Excludes the facility. Reproduces. | **C.** Approves: all covenants evidenced, leverage headroom 0.35× ≥ 0.25×. Correct. |
| **Eligibility agent pinned to known-good determination** | **B.** Downstream (covenant-check, writer) produces a correct memo. | **D.** Baseline. Correct. |

A fails, B passes, C passes → **the eligibility agent is a victim.** Walk back one hop.

At the extraction agent the 2×2 is degenerate in an informative way: the truncation is deterministic, so A reproduces every time, B (a pinned correct `FacilityFacts`) yields a correct memo, and C — a clean input — has no meaning, because the extraction agent *is* the input stage. The defect is not in the agent's reasoning; it is in the projection from its working state to its payload. That is a contract defect, and it is not fixable by prompting the agent to "be careful."

### 12.4 Freeze-and-diff

Diff the failing `FacilityFacts` against the payload from a passing run on a comparable facility. Structural, not textual. The diff is one array: `covenant_schedule` has 6 entries where the passing run has 9, and the missing three are contiguous — rows 7, 8 and 9 of the schedule. Contiguous missing rows is the signature of truncation, not of summarization; summarization loses scattered facts, truncation loses the tail.

That single observation is worth more than the rest of the trace, because it identifies the mechanism (§3.1, truncation under a token budget) rather than merely the site, and the mechanism determines the fix.

### 12.5 What the incident actually was

Two failures, in series, and only the first is the cause:

1. **Handoff corruption (§3.1)** at Extraction → Eligibility: three covenant rows dropped by a token-budget truncation during serialization, with no record that truncation occurred.
2. **Cross-agent context poisoning (§3.2)** at Covenant-check: the absence was converted into the affirmative claim "no leverage covenant identified," which travelled to the writer with more authority than the underlying facts ever had.

And one thing that was **not** a failure, despite being where the wrong answer appeared: the writer. It did what it was built to do.

The sequence is worth stating as the thesis instance it is. The writer was wrong. The covenant-check agent invented a fact. The eligibility agent applied policy correctly. **The corruption happened three handoffs before the surface** — extraction → eligibility, eligibility → covenant-check, covenant-check → writer. A team that started at the writer would have spent the incident on the one component that behaved correctly.

### 12.6 The fix, and the decision the multi-agent nature changed

**The tempting fix** is a prompt change at the writer: "never state that documentation lacks a covenant; state that the covenant was not found in the extracted facts." That fix is cheap, it makes the sentence less false, and it is wrong — it suppresses the symptom at the detector (§10.5) and leaves both the truncation and the absence-to-assertion conversion in place.

**The actual fix is a handoff contract change**, in three parts:

- **Schema.** `CovenantSchedule` requires a `coverage_complete: bool` and a `rows_expected: int`. An extractor that truncated must emit `coverage_complete=false`. Omitting the field fails validation at the boundary — the failure becomes loud and one hop from its cause.
- **Provenance.** Every covenant row carries `source_span_id` pointing at the extraction span that read it, and the eligibility agent's determination must cite a `source_span_id` per rule application. A claim without provenance cannot be emitted, which makes "no leverage covenant identified" unrepresentable rather than merely discouraged.
- **Consumer semantics.** `coverage_complete=false` means *cannot determine*, not *not evidenced*. This is a change to the eligibility agent's contract, not its prompt: the two states were previously collapsed into one, and the collapse is what turned a data gap into a policy violation.

**And the decision that the multi-agent nature changed.** In a single-agent design, this incident has a different shape: one agent holding all 42 pages would have truncated its context too, and would have decided on the truncated context — a wrong answer, but a *first-order* one, where the loss and the decision live in the same transcript and the reasoning visibly rests on six rows rather than nine. The multi-agent design added a **second-order** error that the single agent could not have produced: the covenant-check agent, seeing only a gap, manufactured an affirmative claim about a document it had never read. **Absence became assertion, and assertion became authority, because there was an agent whose only view was the absence.**

That is the general lesson, and it is why the fix is at the contract: the harm was created by a boundary that admitted a claim it could not support. Closing the boundary is the only fix that addresses the harm rather than its wording.

**The evaluation moved with it.** Before the incident, Cymbal's gate was the end-to-end memo score — a rubric over the finished memo. After, the gate has three components: the end-to-end score (still the release gate), a per-handoff eval on Extraction → Eligibility (*does the payload contain every covenant row present in the extraction span, each with provenance, and is `coverage_complete` truthful?*), and a provenance check on Eligibility → Covenant-check (*does every emitted claim cite a span that supports it?*). Both new checks are deterministic, run without a model call, and fail at the boundary where the fix is — which is the whole argument of §9.3, made concrete. The end-to-end score would have stayed at its previous level through this entire incident, because it was measuring the product and the product cannot localize.

The writer, for the record, was never changed.

---

## 13. The Anti-Patterns

Ten, each with the wrong move and its replacement.

1. **Debug from the symptom backwards through the code, not through the handoffs.** *Wrong:* start at the agent that produced the bad output and read its prompt. *Right:* draw the handoff graph first, then walk back one hop at a time (§8.6 step 3). The graph is cheap and it eliminates the wrong half of the pipeline before you read a single prompt.

2. **Treat a fluent output as a verified one.** *Wrong:* accept the writer's citation of an upstream determination as evidence that the determination was sound. *Right:* follow the claim to a span. If the claim has no supporting span, it is a fabrication regardless of how it reads (§3.2).

3. **Add constraints to the agent where the error is visible.** *Wrong:* forbid the specific wrong sentence at the writer. *Right:* find the input that made the sentence correct, and fix the boundary (§10.1, §10.5).

4. **Instrument the interior and not the boundary.** *Wrong:* capture every prompt and response per agent. *Right:* propagate trace context, and emit a handoff span with the §7.2 attributes. Nine attributes beat a full prompt log, because they are joinable (§10.2).

5. **Call a checkpoint re-run a replay.** *Wrong:* "we replayed it and the fix works." *Right:* state the mode. Checkpoint re-run is exploration; true replay is confirmation (§6.3, §10.4).

6. **Treat a low reproduction rate as low priority.** *Wrong:* "it's flaky, we'll watch it." *Right:* 0.9¹⁰ ≈ 0.35 means ten clean runs is the *expected* observation for a 10% bug. Record the denominator, capture the failing trace, look for the input shape that triggers it (§6.2).

7. **Let the orchestrator grade its own routing.** *Wrong:* the supervisor decides the decomposition and the supervisor's eval scores the outcome. *Right:* eval the dispatch span against the receiving agent's declared capability (§3.4). The orchestrator is the only member whose errors are invisible to every other member.

8. **Use one end-to-end score as the release gate and the diagnostic.** *Wrong:* a single rubric over the finished artifact. *Right:* the end-to-end score as the gate, per-handoff checks as the instrument (§9.4). A product cannot localize a factor.

9. **Pass prose where structure is needed.** *Wrong:* hand the next agent a summary. *Right:* hand it a schema-validated payload with provenance, and a reference to the spans behind it (§4.3(ii), §12.6). Prose is where facts go to become unfalsifiable.

10. **Close the post-mortem with one agent's name.** *Wrong:* "root cause: the retriever." *Right:* the pair — the defect, and the input that made a correct agent wrong — each labelled (§5.2, §10.6).

---

## 14. The Claims Audit

Every factual proposition this guide rests on, with source, date, kind, verdict and a quality note. Verdicts: **Verified** (checked against the primary source named), **Flagged** (real but a vendor claim, an abstract-level reading, or otherwise requiring caution), **Rejected** (unsupported, and not to be cited).

### 14.1 Verified at source

| # | Claim | Source | Date | Kind | Verdict | Quality note |
|---|---|---|---|---|---|---|
| 1 | MAST derives 14 failure modes in 3 categories — system design, inter-agent misalignment, task verification — from 1,600+ annotated traces across 7 frameworks, κ = 0.88 | Cemri et al., "Why Do Multi-Agent LLM Systems Fail?", arXiv:2503.13657 | 17 Mar 2025 (v3 26 Oct 2025) | preprint | **Verified** | ID, title, dates and abstract read via arXiv API. Abstract-level; the 14 modes themselves were not enumerated from the full text. |
| 2 | MAST places one of its three categories on misalignment *between* agents | same | 17 Mar 2025 | preprint | **Verified** | The category name is in the abstract. Used only as category-level corroboration (§3). |
| 3 | Who&When: 127 multi-agent systems annotated with responsible agent and decisive error step; best of three methods at 53.5% agent-level and 14.2% step-level; some methods below random; frontier reasoning models not practically usable | Zhang et al., "Which Agent Causes Task Failures and When?", arXiv:2505.00212 | 30 Apr 2025 (v3 2 Jun 2025) | preprint | **Verified as the paper's own reported result on its own dataset** | Quoted from the abstract. Not generalized beyond that dataset (§3.5, §5.4, §11.2). |
| 4 | TRAIL: formal taxonomy of agentic trace error types; 148 human-annotated traces from single- and multi-agent systems; best model scored 11% | Deshpande et al., "TRAIL: Trace Reasoning and Agentic Issue Localization", arXiv:2505.08638 | 13 May 2025 (v3 23 Jun 2025) | preprint | **Verified as the paper's own reported result** | Abstract read via API. The taxonomy's contents were not read from the full text. |
| 5 | `traceparent` carries `<version>-<trace-id>-<parent-id>-<trace-flags>`; `tracestate` carries vendor data; purpose is cross-vendor trace correlation | W3C, "Trace Context", W3C Recommendation | 23 Nov 2021 | standard | **Verified** | Specification read at source; format and problem statement quoted. |
| 6 | Without a shared identifier, traces from different vendors cannot be correlated and vendor metadata may be dropped by intermediaries | W3C Trace Context §2.1 | 23 Nov 2021 | standard | **Verified** | Quoted. The guide substitutes "agent runtime" for "tracing vendor"; that substitution is this guide's, and is labelled as such. |
| 7 | OTel's default propagator uses the W3C TraceContext headers; a downstream service sets the caller's span as parent to join the same trace | OpenTelemetry, "Context propagation" | page last modified 10 Aug 2026 | standard documentation | **Verified** | Read at source; mechanism quoted. |
| 8 | Span links associate spans with a causal relationship where parent/child is the wrong shape; `Consumer` spans may start long after the `Producer` span ended | OpenTelemetry, "Traces" | page last modified 14 Jan 2026 | standard documentation | **Verified** | Both quoted from the concept page. Basis for §4.3(iii). |
| 9 | Baggage propagates arbitrary key/value pairs and must not carry credentials, keys or PII | OpenTelemetry, "Context propagation" (security practices) | 10 Aug 2026 (mod.) | standard documentation | **Verified** | Quoted; the banking consequence is drawn in §4.1. |
| 10 | OpenAI Agents SDK wraps handoffs in `handoff_span()`, alongside agent/generation/function/guardrail/turn spans; spans carry `trace_id` and `parent_id`; custom trace processors are supported | OpenAI, "Tracing — Agents SDK" | page read Oct 2026 | vendor documentation | **Verified** | Read at source. Capability description only; no ranking, no comparison. |
| 11 | LangGraph replay re-executes nodes — LLM calls, API requests and interrupts fire again and may return different results | LangChain, "Use time-travel" | page read Oct 2026 | vendor documentation | **Verified** | Quoted verbatim. The single most load-bearing quote in §6.3 and §10.4. |
| 12 | A subgraph without its own checkpointer is treated as a single super-step by the parent; you cannot time-travel between its nodes | same | page read Oct 2026 | vendor documentation | **Verified** | Quoted. The "checkpoint granularity = diagnostic granularity" reading is this guide's inference, marked as such in §6.3. |
| 13 | Cognition principle 1: share context, and share full agent traces, not just individual messages | Cognition (Walden Yan), "Don't Build Multi-Agents" | page undated — see §15 | vendor post | **Verified as published** | Page read in full; principle quoted. |
| 14 | Cognition principle 2: actions carry implicit decisions, and conflicting decisions carry bad results; parallel subagents make conflicting implicit decisions | same | page undated | vendor post | **Verified as published** | Quoted. Basis for §2.4 and §3.3's mechanism. |
| 15 | Anthropic: multi-agent systems "have emergent behaviors, which arise without specific programming"; small changes to the lead agent unpredictably change subagent behaviour | Anthropic, "How we built our multi-agent research system" | 13 Jun 2025 | vendor post | **Verified** | Date on page. Quoted. Basis for §2.3. |
| 16 | Anthropic: full production tracing was what let them diagnose "not finding obvious information"; they monitor decision patterns and interaction structures without monitoring conversation contents | same | 13 Jun 2025 | vendor post | **Verified** | Quoted. A vendor stating that structural, not content, observability was the diagnostic lever. |
| 17 | Anthropic: subagents pass lightweight references rather than copying outputs through the coordinator, to "prevent information loss during multi-stage processing" (the "game of telephone") | same, appendix | 13 Jun 2025 | vendor post | **Verified** | Quoted verbatim. Direct vendor corroboration of the handoff thesis (§4.3(ii)). |
| 18 | Anthropic: subagents with vague instructions "misinterpreted the task or performed the exact same searches"; the remedy is objective, output format, tool guidance, task boundaries | same | 13 Jun 2025 | vendor post | **Verified** | Quoted. Basis for §3.4's "delegation contract". |
| 19 | Anthropic: agents are non-deterministic between runs even with identical prompts | same | 13 Jun 2025 | vendor post | **Verified** | Quoted. Basis for §6.1. |
| 20 | 0.95⁵ = 0.7738 | arithmetic | computed 17 Oct 2026 | my synthesis | **Verified** | Computed and checked; the framing is owned by the failures guide §1, which derives it from Lusser's Law (1957). |
| 21 | 0.9¹⁰ ≈ 0.35 — ten clean runs is the expected observation for a 10% failure rate | arithmetic | computed 17 Oct 2026 | my synthesis | **Verified** | Computed. Used in §6.2 to reframe "flaky". |

### 14.2 Flagged and rejected

| # | Claim | Source | Date | Kind | Verdict | Quality note |
|---|---|---|---|---|---|---|
| 22 | Cognition's post was published 12 June 2025 | secondary sources only | — | — | **Flagged — date only** | The page read for this guide exposes no publication date. In-repo precedent records 12 Jun 2025 from secondary sources. Content verified; date not. Re-source before citing the date in a filing. |
| 23 | Anthropic's system "outperformed single-agent Claude Opus 4 by 90.2%" on its internal research eval | Anthropic multi-agent post | 13 Jun 2025 | vendor post | **Flagged — not used** | Vendor self-evaluation with no eval name, size, baseline or error bars. Recorded here to show it was checked and deliberately excluded; it is not evidence for anything this guide argues. |
| 24 | Multi-agent systems use ~15× the tokens of chat | Anthropic multi-agent post | 13 Jun 2025 | vendor post | **Flagged — not used** | Vendor-internal analysis. The cost verdict is owned by the sibling guides; this guide does not need the number and does not print it as a finding. |
| 25 | The five-mode taxonomy of §3 is a complete taxonomy of multi-agent failures | — | — | my synthesis | **Flagged — scoped** | It is explicitly a taxonomy of the *interaction layer* only (§3.6). Task/control/integration/evaluation modes are owned by the failures guide and are not covered. |
| 26 | "Handoff errors dominate multi-agent failures" | — | — | — | **Rejected** | No retrieved source measures the share of multi-agent failures attributable to handoffs. The thesis is an argument from mechanism plus the category-level MAST result, not a measured prevalence claim. See §15. |
| 27 | Multi-agent systems fail more often (or less often) than single-agent systems | — | — | — | **Rejected** | Not claimed anywhere in this guide, and no source retrieved would support the comparison at equal task and tool surface. |
| 28 | A benchmark or ranking of multi-agent debugging/observability tools | — | — | — | **Rejected / not found** | No such evaluation exists in a form that could be cited. See §14.4. |
| 29 | Cymbal Bank's pipeline, incident, or figures describe a real deployment | — | — | — | **Rejected** | Fiction. Declared illustrative at the head of §12, and the only institution named anywhere in this guide. |

### 14.3 The row that records a deliberate absence

| # | Claim | Source | Date | Kind | Verdict | Quality note |
|---|---|---|---|---|---|---|
| 30 | **No benchmark number, no vendor ranking, no adoption statistic, and no claim that multi-agent systems fail more or less often than single-agent ones is printed anywhere in this guide** | — | — | — | **Deliberate omission, recorded** | Reasons in §14.4. The only percentages in the guide are: two papers' own results on their own datasets (#1, #3, #4), two arithmetic identities (#20, #21), and one flagged vendor figure explicitly marked *not used* (#23). |

### 14.4 Why no benchmark, no ranking, and no adoption statistic appear in this guide

Three separate reasons, and each one is sufficient on its own.

**The evidence does not support a benchmark.** There is no controlled evaluation of multi-agent *debugging* capability — no shared task set, no held-out incidents, no agreement on what counts as a correct attribution. The closest artifacts are research datasets on attribution accuracy (Who&When's 127 logs; TRAIL's 148 traces), and both are research instruments with their own task definitions, not product tests. Printing a comparative number derived from them would be a category error: those datasets measure a *method's* accuracy on a curated set, not a *product's* fitness for your pipeline.

**A benchmark of products is not a diagnosis.** The question this guide answers is "what changes about diagnosing a failure when the run spans several agents." A feature matrix answers "which tool has more checkboxes," which is a procurement question. Conflating them produces the worst outcome available: a team that selected a tool on a matrix and believes it has solved attribution.

**An adoption statistic would be a distraction dressed as evidence.** Prevalence tells you what other people did, not what is true about the mechanism. It also ages badly and invites the reader to substitute fashion for argument. The claims that matter here — that trace context must be propagated, that a correct agent can be the bug, that a product cannot localize a factor — are true or false independently of how many teams have noticed.

**What is offered instead:** a mechanism-level argument, dated primary sources for every factual premise, and a procedure (§8.6) that a team can run on its own pipeline and check against its own results. That is a weaker artifact than a benchmark and a more useful one.

## 15. What Could Not Be Verified

Recorded as explicit negatives, because a silent omission reads as coverage. **None of these should be asserted elsewhere.**

1. **No measurement of how attribution accuracy degrades with agent count.** The Who&When results (#3) give accuracy on a dataset spanning 127 systems; they do not stratify by pipeline depth in the abstract, and no source retrieved measures the curve. Whether step-level attribution gets worse at 3 agents or at 12, and by how much, is **unknown to this guide**. §5.4 asserts only that the difficulty is structural, not that its magnitude has been measured.

2. **No source for the claim that handoff errors *dominate* multi-agent failures.** This is the thesis's weakest empirical link and it is stated plainly here. The thesis is defended by mechanism (§3.1–3.4), by vendor testimony from two organisations that built production multi-agent systems (#14, #17), and by the category-level MAST result (#2). None of those is a prevalence measurement. A reader who requires a prevalence figure will not find one in this guide, and should not be told one exists.

3. **No vendor documentation retrieved describes deterministic replay of a complete multi-agent run** — model outputs pinned, tool responses recorded, concurrency ordering captured — as a shipped feature. LangGraph documents the opposite for its own replay (#11). This is recorded as "not found in this pass"; it is not a claim that no such capability exists.

4. **The full texts of MAST, Who&When and TRAIL were not read.** Identifiers, titles, dates and abstracts were resolved via the arXiv API over HTTPS with a browser user-agent. The 14 MAST modes, the Who&When annotation scheme, and TRAIL's error taxonomy are therefore described at *category* level, and §3 does not enumerate sub-modes for that reason. Treat every statement about those papers' internal structure as abstract-level.

5. **The publication date of Cognition's "Don't Build Multi-Agents" could not be established from the page itself.** The page exposes no date. The content — both principles, quoted — is verified from the primary page. The date is flagged (#22) and should not be cited without re-sourcing.

6. **No controlled measurement of the 2×2 procedure's effectiveness.** §8.2 is this guide's design, assembled from standard experimental practice (hold one variable, cross two conditions). No source retrieved evaluates it as a debugging intervention, and no claim is made about how often it resolves an incident. It is offered as a procedure whose *logic* is sound, not as an empirically validated one.

7. **The "three steps earlier" in the thesis is illustrative, not measured.** It describes the *structure* — corruption upstream, surfacing downstream, with hops in between — and the Cymbal example is constructed to have exactly three. No source establishes a typical distance between cause and surface, and no such number should be read into the thesis.

8. **Tool limitation affecting this pass.** `web_search` was not relied upon for any claim in this guide, following the known unreliability of the search backend on this host; primary sources were fetched directly by URL and by the arXiv API over HTTPS. One extraction failed outright: `https://www.w3.org/TR/trace-context/` returned a scraping error (all engines failed) and was worked around by extracting the versioned Recommendation at `/TR/trace-context-1/`, which resolved. **An empty or failed retrieval was treated as a tool limitation, not as absence of material**, which is why §11.3's "no vendor documents deterministic replay" is phrased as "not found in this pass."

9. **No verification of any specific bank's, regulator's, or vendor's internal practice.** No primary case study of a real multi-agent debugging incident was found or sought. The Cymbal example is fictional and is the only institution named in the guide.

---

## 16. Glossary, Cross-References, and Closing Summary

### 16.1 Glossary

**Attribution** — The answer to two questions, not one: which agent is *defective* (would fail on a clean, contract-conforming input), and which *input* made a correct agent's behaviour wrong. Answering only the first is the most common diagnostic error in this discipline (§5.2).

**Baggage** — Arbitrary key/value context propagated across process boundaries alongside trace context; the standard mechanism for carrying task, tenant and approval scope. Must not carry credentials, keys or PII (OpenTelemetry, standard documentation).

**Blackboard / shared state** — A store multiple agents read and write, as distinct from a message passed between two. Corrupts laterally rather than downstream; requires key-based rather than edge-based reconstruction (§4.3(vi)).

**Counterfactual test** — Hold the pipeline fixed, swap one agent, re-run the tail, compare failure *rates*. The only technique that answers attribution directly rather than by inference, and expensive in re-runs, non-determinism and side effects (§5.3).

**Defect vs victim** — A member is defective if it would fail given a clean input; a victim if its behaviour was correct and its input was not. The distinction that decides whether a fix has any effect (§5.2).

**Emergence** — Behaviour of the composition not specified by, and not predictable from, any member's specification. The reason per-agent tests cannot cover the failure space (§2.3).

**Handoff** — The moment one agent's output becomes another's input. Three parts, routinely conflated: the *payload* (what data crossed), the *context* (its trace and provenance), and the *contract* (what the receiver may assume). Most production handoffs specify only the first (§1.2).

**Handoff corruption** — Mode 1 of §3: the payload loses or mangles a fact the receiver needed, by truncation, summarization loss, or reformatting loss. Signature: a downstream non-sequitur that is correct given its input.

**Handoff span** — A span representing the transfer itself: `from_agent`, `to_agent`, payload hash, schema version, token counts, dropped-context flag, trace-context presence. A first-class span type in at least one shipped SDK (§11.1); absent from most production systems.

**Interaction-layer failure** — A failure that is a property of an edge or of the graph rather than of any node. The subject of §3; invisible to a per-agent transcript read.

**Laundering** — The conversion of a corrupted input into fluent, confident, well-formatted output at each hop, which destroys the evidence of corruption by making the wrong answer indistinguishable in form from a right one (§5.1).

**MAST** — Multi-Agent System Failure Taxonomy: 14 modes in 3 categories from 1,600+ traces across 7 frameworks (Cemri et al., arXiv:2503.13657, 2025, preprint). Category-level corroboration for §3, not the source of its five modes.

**Orchestrator / supervisor** — The agent that decomposes and dispatches. Diagnostically privileged: the only member that sees the whole task, and therefore the only member whose errors are invisible to every other member (§3.4).

**Poisoning (cross-agent)** — Mode 2 of §3: an unsupported claim becomes the next agent's premise and gains authority while losing provenance. An epistemic ratchet: claims gain confidence as they travel, never lose it.

**Replay (true)** — Re-execution of orchestration against recorded model outputs and recorded tool responses. Deterministic, because the stochastic components are replaced by their recorded results. The only form that supports a counterfactual (§6.3).

**Re-run from checkpoint** — Restore state, execute forward, re-issuing model and tool calls. Non-deterministic, and what most frameworks mean by "replay." LangGraph states it plainly: *"LLM calls, API requests, and interrupts fire again and may return different results."*

**Span link** — An association between spans expressing a causal relationship where parent/child is the wrong shape — notably asynchronous fan-out, where the child outlives the parent (OpenTelemetry, standard documentation).

**Trace context** — Trace ID, calling span ID, trace flags and vendor state, standardized as `traceparent`/`tracestate` (W3C Recommendation, 23 Nov 2021). The mechanism that turns N traces into one; its absence is the silent failure of §4.3(i).

**2×2 (input × member)** — The isolation procedure of §8.2: deployed member on failing input, pinned member on failing input, deployed member on clean input, baseline. Separates a poisoned input from a faulty member in four runs.

### 16.2 Cross-references

| For | Go to |
|---|---|
| The span hierarchy, `gen_ai.*` conventions, content-capture policy, SLIs, monitoring, loop detection, tooling landscape | [agentops_guide.md](agentops_guide.md) §3.1–3.2, §3.5, §4, §5, §7 |
| Compounding math, the task/control/integration/evaluation taxonomy, context drift and token limits, demo-vs-production, cost, mitigations, readiness checklist | [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) §1, §2 (esp. §2.6), §3–§5, §8, §10 |
| Success conditions, working patterns, failure conditions, communication overhead, the boundary spectrum, "should this be an agent?" | [agents_work_fall_apart_guide.md](agents_work_fall_apart_guide.md) §1, §2, §4, §5.2, §7, §8 |
| Supervisor / hierarchical / peer topologies, control and coordination patterns, model routing | [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md) |
| Banking deployment and regulatory considerations for multi-agent systems | [multi_agent_banking_guide.md](multi_agent_banking_guide.md) |
| The four-line statement this guide expands, and hierarchy-specific pitfalls | [hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md) §12.5 |
| The MAST-sourced failure literature and the "most expressive, most expensive" verdict on multi-agent | [agent_skills_vs_multi_agent_systems_guide.md](agent_skills_vs_multi_agent_systems_guide.md) §6.2, §7.2 |
| Context budgets, compression and positional effects — the mechanism behind truncation loss | [context_engineering_guide.md](context_engineering_guide.md) |

### 16.3 Closing summary

Seven propositions, in the order the guide establishes them.

**A multi-agent trace is a forest, and the joins are the subject.** The quality of the join — parent/child, span link, shared trace ID, or new trace — sets a hard ceiling on what any later diagnosis can conclude, and it is decided at instrumentation time (§2.1, §4.1).

**State is passed or shared, never held.** The question "what did this agent know" stops being a read and becomes a reconstruction, which requires a recorded payload at the handoff (§2.2, §7.3).

**Five failure modes live on the interaction layer** — handoff corruption, cross-agent poisoning, the emergent loop, the orchestrator misroute, and attribution failure — and none of them is diagnosable by reading one agent's transcript (§3).

**A member can be perfectly correct and still be the bug.** The pipeline fails at the seam, and the seam has no transcript, which is why the edge must be instrumented as an object rather than inferred from timestamps (§2.4, §4.2).

**Attribution is a pair, and the surfacing agent is usually the victim.** Defect versus victim is the distinction that decides whether a fix does anything, and the 2×2 is the four runs that settles it (§5.2, §8.2).

**Replay is not re-run.** A checkpoint re-run is a different sample, and a cause established on one is a hypothesis (§6.3, §10.4).

**Fix the contract, not the prompt.** Cymbal's fix was a schema, a provenance field and a consumer semantic — not a sentence added to the writer's instructions — and the eval moved from the product to the boundary because a product cannot localize a factor (§12.6, §9.3).

The guide began with a claim and tested it in every section. The claim survives, with one honest qualification recorded in §15: it is an argument from mechanism and from category-level evidence, not a measured prevalence. What it lacks in a number it makes up for in consequence — because if it is even approximately right, then almost all of the effort a team spends at the point of failure is spent at the one place the cause is not.

**A multi-agent failure is almost never where it surfaces; it is at the handoff that corrupted the context three steps earlier.**
