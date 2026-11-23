# The Orchestration Layer - Who Owns the Record of a Run

> **A 2026 deep-dive on the one service in an enterprise agentic platform that can fail without emitting an error: what the orchestration layer is, why its failure signature differs from the seven services beside it, and why its architecture reduces to a single question - who owns the record of what happened.**

**This guide develops eight things and defers everything else:** the run as the unit of record (section 3), durable execution and the two distinct mechanisms that implement it (section 4), the handoff contract (section 6), registry resolution at dispatch (section 7), tenancy and quota enforcement (section 8), attribution by run and step (section 9), the operator's view of a run that is stuck with nothing to alert on (section 10), and recovery plus reconciliation (section 11). Sections 2, 5, 12, 13 and 14 supply the argument, the topology context, the protocol boundary, the failure catalogue and the worked example that hold those eight together.

> **Series context and boundary (read this first).** This guide owns exactly one thing: **the orchestration layer, and the record of a run that it must own.** It does not re-derive what sibling guides own, and it cross-references by section number throughout.
>
> - **The layer's place is owned elsewhere.** [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) section 1.1 defines the platform as eight services; section 1.2 separates platform from framework from application (a platform hosts applications, an application embeds a framework); section 2.5 is the architecture-level statement of this layer, including the engine comparison table (LangGraph / Temporal / Airflow); section 3.3 covers A2A; 4.5 observability; 4.7 governance; section 8 the worked example. That guide says where orchestration sits and what it is for. This guide says what it is.
> - **The agent inventory is owned elsewhere.** Platform guide section 2.6 owns the agent registry as an inventory - metadata, compliance status, lifecycle. Section 7 below takes only the orchestration-time view: resolution at dispatch, and registry-versus-reality during a run.
> - **Topology choice is owned elsewhere.** [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md) owns control patterns (sections 3, 4, 5), multi-backend and model routing (6, 7), comparison (8) and design (9). [hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md) owns hierarchy as topology (sections 3, 6, 8, 9, 13). [multi_agent_banking_guide.md](multi_agent_banking_guide.md) owns the banking deployment of those patterns (sections 3, 5, 6, 9, 11, 12, 14).
> - **Failure attribution is owned elsewhere.** [multi_agent_debugging_guide.md](multi_agent_debugging_guide.md) owns the taxonomy (3), the trace tree (4), attribution (5), reproduction (6), instrumentation delta (7), diagnosis patterns (8) and the Cymbal example (12). Section 6 below borrows its vocabulary and builds something different: the contract a handoff must carry, not the diagnosis of a handoff that failed.
> - **Ops telemetry is owned elsewhere.** [agentops_guide.md](agentops_guide.md) owns the single-run trace definition and span hierarchy (3.1-3.2), monitoring and alerting (5), cost operations (6) and tooling (7). Section 10 below depends on 5 and does not rebuild it; section 9 depends on 6 the same way.
> - **The skills argument and the selection matrix are owned elsewhere.** [agent_skills_vs_multi_agent_systems_guide.md](agent_skills_vs_multi_agent_systems_guide.md) owns the seven properties (3), the replace claim (4), the compile claim (5) and the enterprise consequence (8). [ai_agent_platform_selection_guide.md](ai_agent_platform_selection_guide.md) owns criteria (3), scoring (4), the decision matrix (5) and pitfalls (7).
> - **The rule that keeps this guide honest:** if a passage explains how to choose a topology, or how to debug a run, it has left this guide. Every section below was checked against that rule.
>
> The gap was counted, not assumed. Platform guide section 1.1 declares **eight** services and its section 4 develops **seven**; orchestration is the one with no 4.x subsection. That count is reproduced with line references in 1.1.

> **Verification policy.** Every proposition in this guide is dated and attributed: who published it, on what date it was read, and of what kind it is. Kinds used are **(vendor documentation)** for product docs describing a shipped capability, **(standard)** for an open specification, **(my synthesis)** for an argument assembled here, and **(arithmetic)** for counts and totals. Engine mechanisms are taken only from the pages listed in section 15, read on 23 November 2026, and are never described from memory; where a page exposes no version string, section 15 says so rather than inventing one. Marks used throughout: ✅ = verified at the source named; ⚠ = flagged, true as read but with a caveat stated in place; ✗ = rejected, and the rejection is argued in place. **This guide prints no benchmark, no duration measurement, no cost-per-unit figure, no statistic about how widely anything is used, and no vendor ranking** - for the three reasons given in 15.3.

**Jack Liu Shurui, Solution Architect**

Baseline November 2026 (written 23 November 2026). Source dates are as published; re-verify capability statements against current documentation before designing a control around them.

---

**Contents**

1. [Overview - The Decoder, the Asymmetry, the Thesis, and the Boundary](#1-overview---the-decoder-the-asymmetry-the-thesis-and-the-boundary)
2. [Why Orchestration Is Different from the Seven Siblings](#2-why-orchestration-is-different-from-the-seven-siblings)
3. [The Run as the Unit of Record](#3-the-run-as-the-unit-of-record)
4. [Durable Execution](#4-durable-execution)
5. [The Coordination Topologies, Cross-Referenced](#5-the-coordination-topologies-cross-referenced)
6. [Handoffs](#6-handoffs)
7. [Registry and Discovery](#7-registry-and-discovery)
8. [Tenancy, Quotas and Isolation](#8-tenancy-quotas-and-isolation)
9. [Attribution](#9-attribution)
10. [The Operator's View](#10-the-operators-view)
11. [Recovery and Reconciliation](#11-recovery-and-reconciliation)
12. [Enterprise Integration and the A2A Boundary](#12-enterprise-integration-and-the-a2a-boundary)
13. [The Failure Modes](#13-the-failure-modes)
14. [The Cymbal Bank Worked Example](#14-the-cymbal-bank-worked-example)
15. [The Claims Audit and What Could Not Be Verified](#15-the-claims-audit-and-what-could-not-be-verified)
16. [Glossary, Cross-References, and the Closing](#16-glossary-cross-references-and-the-closing)

---

## 1. Overview - The Decoder, the Asymmetry, the Thesis, and the Boundary

### 1.1 The asymmetry, counted rather than assumed

The platform guide in this repository opens by defining an enterprise agentic platform as eight services. Section 1.1 lists them in a table: model gateway, agent runtime, tool registry and MCP hub, memory layer, orchestration, observability, governance, security. Line 43 of that file carries the orchestration row: "Workflow + multi-agent execution", delivered as "Service + engines".

Section 4, titled "Components in Depth", then develops seven of the eight, each in its own subsection: 4.1 Model Gateway in Depth (line 351), 4.2 Agent Runtime (line 381), 4.3 Tool Layer (line 422), 4.4 Memory Layer (line 432), 4.5 Observability (line 452), 4.6 Security (line 478), 4.7 Governance (line 503). There is no 4.8.

| Service, as declared in 1.1 | In-depth subsection in section 4 |
|---|---|
| Model gateway | 4.1, line 351 |
| Agent runtime | 4.2, line 381 |
| Tool registry / MCP hub | 4.3, line 422 |
| Memory layer | 4.4, line 432 |
| Orchestration | **none** |
| Observability | 4.5, line 452 |
| Security | 4.6, line 478 |
| Governance | 4.7, line 503 |

**Eight services declared, seven developed, one absent.** That is the count, and it is a count of headings, not an opinion about quality.

Orchestration does appear in that file, in six kinds of place. First, one row in the 1.1 table (line 43). Second, a subgraph label in the section 1.3 diagram: ORCHESTRATION LAYER, "workflows and multi-agent" (line 113). Third, one sentence in the prose that walks the reader through that diagram (line 152). Fourth, its own architecture-level subsection, 2.5 "Orchestration Layer" (lines 245-261), which contains the three-engine comparison table at lines 253-259. Fifth, a mention in the worked example's component table (line 679), where the bank's platform uses one engine for deterministic spines, another for agentic paths, and an event bus for triggers. Sixth, the glossary and verification apparatus: a glossary entry at line 790, a mention at line 824, and the closing verification note at line 831.

One of those references is not about the layer at all. Line 824 defines Kubernetes as "the container-orchestration substrate the platform runs on". It is a homonym, and the homonym is part of the finding: the same word names a scheduling substrate and a platform service, and only one of the two is a service the enterprise must specify.

What section 2.5 does is real work, and it is architecture-level work: it separates deterministic workflows from agentic orchestration from multi-agent orchestration, compares three engines by paradigm, strength, weakness and fit, and states the platform pattern - one engine for the deterministic spine, an agent graph engine for the agentic paths, an event bus for triggers, all emitting to the same observability and audit pipelines. What it does not do is develop the layer. There is no run record, no handoff contract, no replay semantics, no recovery procedure, no tenancy rule, no attribution key. Three paragraphs and a table.

So: **that guide owns the layer's place; this guide owns the layer.** The absence is not a defect in that guide - declaring a service and specifying it are different jobs, and a reference architecture is entitled to stop at the boundary of a component it has placed. But the honest reading is that orchestration was declared a service before it was specified as one, in that document and, in this author's experience of platform programmes, in most of the platforms built from documents like it. The service that owns the completion state of a business process is routinely the service with the thinnest specification.

### 1.2 The decoder

Eight terms carry this guide. Each is defined here once, in the sense used throughout, and each ends with the question it forces on the architect - the question that the rest of the guide answers.

**Run.** A run is one instance of a business process that must reach exactly one terminal state. It is not a request (one message, answered or not), not a session (a conversation that may contain many requests), not a trace (evidence about execution, owned by [agentops_guide.md](agentops_guide.md) section 3.1-3.2), and not a task (A2A's unit of work, whose lifecycle is defined in the protocol and which may be one step of a run; see section 12). The question: for each of your workflows, can you name the terminal states, and can you name the component that writes them?

**Step.** A step is the unit at which the record advances: one coordinator decision plus the external effects that decision authorizes. Steps are what the record nests; a step is not the same as a model call (one step may contain several) or an agent invocation (one step may be a whole subgraph). The question: what is the smallest unit your record can re-drive, and does it match the unit your engine actually checkpoints?

**Handoff.** A handoff is a state transition between agents that transfers three things - context, authority, and obligation - and it is the only place in a multi-agent system where accountability changes hands (section 6). The question: when agent B receives work from agent A, which record says B accepted it, and when does A stop being accountable?

**Checkpoint.** A checkpoint is a durable snapshot of execution state taken at a step boundary and scoped to an identity that can be used to retrieve it. In LangGraph the scope is a thread, and the thread identifier is the primary key for storing and retrieving state ("Checkpointers", LangChain/LangGraph documentation, retrieved 23 November 2026). The question: what is your checkpoint scoped to, and what becomes unrecoverable the moment that scope ends?

**Replay.** Replay is the reconstruction of a run's progress by re-executing recorded work against recorded results. It is not the same as re-running the world, and in at least one major engine it is explicitly not reproducible: LangGraph's time-travel page states that replay "re-executes nodes - it doesn't just read from cache. LLM calls, API requests, and interrupts fire again and may return different results" ("Use time-travel", retrieved 23 November 2026). The question: after a replay, will your external systems have been written twice?

**Coordinator.** The coordinator is the component that decides the next step and owns the transition record. It may be a library inside an agent, a service with its own store, or an engine's workflow definition. The question: is your coordinator inside the thing it coordinates - because a coordinator that lives inside an agent cannot outlive that agent, and cannot attest to what the agent did.

**Registry.** The registry is the mapping from a name to an endpoint, a version and a contract, resolved at dispatch time (section 7). It is an inventory of what exists, not a statement of what is currently running. The question: can you reconstruct which version of each agent participated in a run that finished last quarter?

**Tenancy.** Tenancy is the partition within which runs compete for capacity, authority and budget (section 8). The question: if a tenant is suspended at 14:00, what happens to the runs it had in flight, and who writes their terminal state?

### 1.3 The thesis

> **Every other platform service fails loudly; orchestration fails by leaving a run half-done - so its architecture is really a decision about who owns the record of what happened.**

Section 2 argues that service by service, then concedes the two places where the claim is too strong - memory retrieval and observability both have quiet failure modes - and amends the wording accordingly. The amendment is stated in 2.4, and both the original and amended forms are recorded in the claims audit (section 15).

### 1.4 The boundary

| This guide owns | Sibling owns | Sibling section |
|---|---|---|
| The run record: state, ownership, terminal semantics | The single-run trace and span hierarchy | agentops 3.1-3.2 |
| Durable execution mechanisms and their failure modes | Engine selection and the engine comparison table | platform 2.5 |
| The handoff contract (context, authority, obligation) | Diagnosis of a failed handoff, attribution | debugging 3, 5 |
| What each topology does to the unit of record | Which topology to choose, and hybridization | hybrid 3-5; hierarchical 3, 6; banking 3 |
| Registry resolution at dispatch; registry-versus-reality | The agent inventory, metadata, compliance, lifecycle | platform 2.6 |
| Tenancy, quota and suspension semantics | Runtime sandbox isolation | platform 4.2 |
| Attribution by run and step (chargeback keys) | Cost operations and the cost model | agentops 6 |
| The operator's view of a run with no error | Monitoring and alerting mechanics | agentops 5 |
| Recovery, reconciliation, duplicate effects | Evaluation, quality gates, deployment lifecycle | banking 6, 9, 11 |
| The A2A message boundary as it affects the record | A2A's provenance and role in the platform | platform 3.3 |
| - | Platform and vendor selection, scoring, pitfalls | platform_selection 3-5, 7 |
| - | Skills versus multi-agent: seven properties, claims | agent_skills 3, 4, 5, 8 |

### 1.5 How to read this guide

Sections 2 through 4 are the argument and the mechanism: why the layer is different, what the run is, and how durability is actually implemented in the two engine families this repository documents. Sections 5 through 12 are the layer's contracts, one per concern, each written as "the record must contain X because Y fails without it". Section 13 is the failure catalogue, written as symptoms an operator would observe rather than as categories. Sections 14 through 16 are the example, the audit and the glossary.

A reader who wants only the engineering can start at section 3 and read to section 12, then use section 13 as a checklist. A reader who wants the argument should read 1.1, then 2. A reader who is deciding whether to build this at all should read section 14 and then section 15, in that order, because the example is fiction and the audit is not.

## 2. Why Orchestration Is Different from the Seven Siblings

### 2.1 The claim, service by service

The thesis is not a claim about likelihood. It is a claim about the *shape* of failure, and shape can be compared service by service. The comparison below uses five columns, and the fourth is the one that matters: loudness is not a property of severity, it is a property of whether the failure is addressable - whether the error names the failing thing to a component that can act on it.

| Service | Failure signature | Detectable by | Who notices | Blast radius |
|---|---|---|---|---|
| Model gateway | Typed error: refusal, provider failure, quota rejection | The caller, in the same call | The agent, immediately | One call; the step retries or fails |
| Agent runtime | Process exit, kill, failed health check | The supervisor that started it | The runtime supervisor | One execution |
| Tool layer / MCP hub | Typed protocol error, tool-level failure, permission denial at the tool boundary | The call site | The agent or the tool client | One invocation |
| Memory layer | **Nothing.** A recall that returns nothing | Nobody, unless the caller asserts an expectation | Possibly nobody | One step's input |
| Observability | **Nothing.** A span that is never emitted | Something else must assert a trace should exist | Nobody, by default | Evidence about a run |
| Security | Denial before execution: identity rejected, authorization refused | The policy decision point, and the caller | The caller, the security team | One request |
| Governance | Refusal: approval not granted, policy verdict negative | The refusal itself, which is recorded | The requester, the risk function | One decision |
| **Orchestration** | **No error at all.** A business process that silently did not complete, or completed twice | Nothing in the failing path | The customer, or an auditor, later | The completion state of the business process |

Two rows in that table are the concession this section owes, and they get their own subsection. The other six are the argument.

### 2.2 The six that fail loudly, and why their loudness is local

**The model gateway** fails inside the call that asked for the completion. A refusal, a provider error and a quota rejection are all returned to the caller with enough identity attached that the caller can decide: retry, fall back, or fail the step. The gateway's failure is benign not because it is rare but because it is *co-located with the decision it informs*, and because the gateway's log and the caller's log can be joined on a request identifier. The error names the failing component to the component that can act on it. That is the whole definition of loud.

**The agent runtime** fails at a process boundary: an exit status, a kill, a failed health check. This is the sharpest lesson in the whole table, because it shows that loudness is not the same as safety. When a runtime crashes, you learn that an execution died. You do not learn what happened to the business process, because the runtime has no concept of the business process - by design, since platform guide section 4.2 scopes the runtime to executing one agent in a sandbox. The crash is an assertion about a process, and the run continues to be someone else's problem. Loudness buys you a local fact, not a global one.

**The tool layer** raises typed errors at a named boundary: a protocol error, a tool-level failure, an authorization denial when the tool is invoked. The property that makes it loud is attribution - the failure arrives attached to a tool name, a caller identity and a request, so the record can say which tool refused whom.

**Security** is loud because it acts *before* execution. A policy decision point returns a verdict, and a verdict is a recordable object; a denied request never runs, so there is no half-done state to reconcile. This is a design choice rather than a law of nature: a platform that logs denials only at the gateway and not at the decision point has moved the silence, not removed it, and section 10's operator will not be able to tell a denial from an absence.

**Governance** is loud by construction, because the artifact of governance *is* a record. An approval that was not granted, or a policy that returned a negative verdict, leaves a trace of its own refusal. Governance fails visibly for the same reason a ledger fails visibly: its output is the thing you would use to check it.

**Observability** belongs in the concession, not here - and that is the surprise of this section. The service whose entire purpose is to make failure visible has its own silent mode.

### 2.3 The concession: two siblings also fail quietly

A guide that claimed orchestration is the only quiet service would be overselling, and the overstatement would be easy to catch. Two siblings have genuine silent-failure modes, and both are worth stating precisely because the precision is what saves the thesis.

**Memory retrieval misses are quiet.** A recall that returns nothing is indistinguishable, at the API boundary, from nothing to recall. The memory layer cannot raise "not found" as an error, because empty is a legitimate and common answer: most queries against a memory store should return little or nothing. The mechanism makes it worse. Similarity search over an embedding index always returns the nearest neighbours it has; there is no representation in that interface for "the fact you needed was never stored, or was stored under a different phrasing". So the agent proceeds, builds a plausible answer on an absent context, and every component in the path reports success. This is a real silent failure and it is common.

**Observability has its own silent mode.** A span that is never emitted is invisible to the tracing system. The absence of a trace is detectable only if something else asserts that a trace should exist - a count of expected runs, a completeness check, an assertion in a pipeline. Three ordinary mechanisms produce the absence: a sampler decides not to record the trace, an exporter drops it when the collector is unavailable, or the process dies before the span is flushed. In all three cases the trace store is internally consistent and simply does not contain the run. Nothing in the store is in an error state, because the store never knew.

Both concessions are load-bearing, and neither is repaired by wishing. The thesis survives them, but only after it is reworded.

### 2.4 Why the thesis survives, and the amended wording

The rebuttal has three parts, and they are applied in order: scope of damage, terminal state, existence of a record.

**Scope of damage.** A memory miss degrades one step's input. The run continues, other steps run, and the run still terminates - with a worse answer, and with a record that shows the retrieval step returning nothing. The damage is *inside* the run and bounded by it. Orchestration's failure is not inside the run; it is the run's completion state. The difference is category, not degree.

**Terminal state.** An observability gap removes evidence about a run; it does not change whether the business process completed. The money moved or it did not; the dispute was resolved or it was not. You have lost your ability to prove what happened, which is serious, but the business outcome is untouched and the systems of record still hold it. Orchestration's failure changes the completion state itself: the process did not complete, or it completed twice. That is the outcome, not the evidence about the outcome.

**Existence of a record.** This is the part that decides it. A memory miss leaves the run record intact - and if the retrieval is recorded as a step, the miss is reconstructible after the fact, which is why section 3 insists the step ledger includes retrieval. An observability gap leaves the systems of record intact and the run ledger intact; you reconcile from those and you can even detect that the trace is missing. Orchestration's failure destroys the only artifact that would have told you: if no one wrote the terminal state, there is nothing to reconcile against unless the reconciliation deliberately looks at the systems of record rather than at the platform's own tables. That is exactly why section 11 makes reconciliation a platform function with an owner and a schedule.

So the precise form of the thesis, amended where the concession requires it:

> **Most platform services fail loudly. Two - memory and observability - can fail quietly, but their quiet failures degrade one step's input or remove evidence about a run. Orchestration alone fails quietly in a way that changes the completion state of the business process and destroys the record that would have revealed it; so its architecture is really a decision about who owns the record of what happened.**

The headline form used in section 1.3 and as the closing line of this guide is a shorthand. Where the two differ, this paragraph governs, and the claims audit records both. The amendment does not weaken the conclusion - it sharpens it, because it identifies the exact property that makes orchestration different: not silence in general, but silence about completion.

## 3. The Run as the Unit of Record

### 3.1 Defining the run by what it is not

A run is one instance of a business process, from the trigger that started it to the terminal state it must reach exactly once. The word "exactly once" is doing the work: it is what makes a run a different kind of object from the four artifacts it is usually confused with.

| Artifact | Question it answers | Typical owner | Terminal? |
|---|---|---|---|
| Request | Was this message handled? | The API layer | Answered, not terminal |
| Session | What has been said so far? | The conversation service | Expires, does not complete |
| Trace | How did this execution proceed? | Observability ([agentops_guide.md](agentops_guide.md) 3.1) | Ends when the process ends |
| Task | What work has an agent accepted? | The agent protocol (section 12) | Yes, within one agent's scope |
| Run | Did the business obligation discharge? | **This layer** | Yes, exactly once |

The distinctions have consequences rather than being pedantry. A request can be answered twice without harm if the answer is idempotent; a run cannot complete twice without a duplicate effect somewhere. A session can be abandoned with no record; a run abandoned is an obligation in default. A trace can be sampled and still be a trace; a run cannot be sampled, because the sample is the obligation. A task can reach COMPLETED and leave the run unfinished, because the task belonged to one agent and the obligation belonged to the business.

The last row of that table is the argument of this guide in miniature: the run is the only artifact in the list whose terminal state is a *business* fact, and therefore the only one whose owner must be the platform rather than a component.

### 3.2 The state a run must carry

The record is not a log. A log is an ordered sequence of observations; a record is a set of fields that must be present for a specific question to be answerable later. The fields below are the minimum, and each is justified by the question that becomes unanswerable without it.

| Field | Contains | Unanswerable without it |
|---|---|---|
| Identity | Run id, business-process id, tenant, trigger, creation time | "Which obligation is this?" and every join to a system of record |
| Intent | The objective as stated at admission, and the acceptance test | "Was the outcome the one that was asked for?" |
| Inputs | References plus digests to the admitted payload and documents | "What was it working from?" after the source is updated or deleted |
| Step ledger | Ordered steps: step id, agent version, decision, effects authorized, outcome | "What happened, in what order, and who did it?" |
| Authority grants | Scope, limits, expiry and delegation depth per step | "Was it allowed to do that, on whose behalf?" |
| External effects | Each write to a system of record, with its idempotency key | "Did we already do this?" - the question recovery depends on (section 11) |
| Terminal state | The one terminal value, who wrote it, when, and why | "Did it finish?" - the question the whole guide is about |
| Version set | Agent versions, model versions, policy versions, registry snapshot | "Can this be reproduced, and was it compliant at the time?" |

Two of those rows deserve emphasis because they are the ones most often missing in real platforms. **External effects with idempotency keys** exist so that recovery has something to check against; a ledger that records "step 4 called the case-management API" without recording the key it used cannot tell a resumed run whether step 4 already landed. **The version set** exists because a run that outlives a deployment is normal, not exceptional: a long-running workflow will cross at least one agent release and often one model retirement, and a record that names only the agent's *name* cannot say which behaviour it got.

### 3.3 Who does not own it

**Not the agent.** The agent is the wrong owner for three independent reasons, and each is sufficient. Agents are replaced: the version that ran is not the version that exists. Agents are retried: a retried agent would write its own retry into its own record. And an agent that has gone wrong must not be able to write its own acquittal - the record is the thing an investigation reads, so the subject of the investigation cannot be its author. This is not a comment on agent trustworthiness; it is the same separation that puts the ledger outside the teller's drawer.

**Not the trace.** The trace is evidence about execution, and it is owned by the observability discipline - [agentops_guide.md](agentops_guide.md) section 3.1-3.2 defines it and section 5 operates it. A run is a business obligation, and the differences are practical: traces are sampled, traces are dropped by exporters, traces are shaped for debugging rather than for settlement, and a trace ends when the process ends while a run may wait days for a human. A platform that treats its trace store as its run record has made its audit depend on a sampling policy.

**Not the workflow engine's private history, alone.** This is the subtlest of the three and the one that section 14 costs money to learn. An engine's event history is an implementation artifact: its schema is the engine's, its retention is set by whoever operates the engine, and its lifetime is bounded by the engine's own lifecycle. The run record must outlive the engine, because the obligation outlives the engine - a dispute opened in one year may be examined in another, long after a migration. Section 3.5 develops this as the two-records problem.

What the run record *is*, positively: a contract with audit. It says, in a form that survives the components that produced it, that this obligation was admitted, handled by these actors under these grants, produced these effects, and reached this terminal state. Everything in sections 4 through 12 is either a mechanism for writing that contract reliably or a guardrail against writing it falsely.

### 3.4 The write model

A record that can be edited is not a record. Three properties make the difference, and they are properties of the store rather than of the schema.

**Append-only.** Corrections are new entries, not overwrites. This is the same property LangGraph's documentation describes for its own state, from the other direction: `update_state` "does not roll back a thread; it creates a new checkpoint branching from the point" ("Use time-travel", retrieved 23 November 2026). The equivalent in a ledger is that a correction is an appended entry naming the entry it supersedes, which means the history of the correction is itself evidence.

**Actor-attributed.** Every entry names the principal that wrote it, including writes made by platform components. An entry with no principal is indistinguishable from one written by an intruder, and the first question in any investigation is which of the two it is.

**Digest per entry, chained.** Each entry carries a digest over its own content and the previous entry's digest. This is what makes tampering detectable rather than merely unlikely, and it is cheap. Without it, "the record says the refund was issued" is a claim about a row that anyone with write access could have changed.

### 3.5 The two-records problem

Every platform built on a durable engine ends up with two records of the same run. The **engine event history** is the engine's own ordered record of the commands and events that drove execution - Temporal records each state transition in a persistence store ("Workflow Execution", Temporal documentation, retrieved 23 November 2026). The **platform run ledger** is the field set in 3.2, written by the coordinator as the run advances.

They are not redundant, and the mistake is to assume one can be derived from the other at will.

The engine history knows things the ledger does not: exactly which commands were issued, in what order, and what the engine did on replay. The ledger knows things the engine does not: the business-process identity, the authority grant under which a step ran, the idempotency key used for an external write, and the terminal state as a business fact rather than as a workflow status. The engine's status vocabulary is about the workflow; the ledger's terminal state is about the obligation.

The operational rule that follows is short: **the ledger is authoritative for "did the obligation discharge", the engine history is authoritative for "what did the engine do", and the two must be joinable by run id.** An audit that reads only the engine history is reading an implementation artifact whose retention is set by an operations team that has never been asked an audit question. An audit that reads only the ledger cannot reconstruct a replay. Section 14 is the story of a platform that had the second record and called it the first.

## 4. Durable Execution

### 4.1 What durability buys, and what the word does not claim

Durability, in the sense this guide uses, is the property that a run survives the death of the process, the host and the engine without losing the obligation. Temporal's documentation gives the sharpest available definition: a Workflow Execution is "a durable, reliable, and scalable function execution", and durability is defined there as "the absence of an imposed time limit", with the execution running "effectively once and to completion - whether your code executes for seconds or years" ("Workflow Execution", Temporal documentation, retrieved 23 November 2026).

Read that definition carefully, because it is narrower than the marketing that usually surrounds the word. It is a statement about what the platform declines to impose - a deadline - and about a completion guarantee that is qualified by the word "effectively". It contains no claim about speed, no claim about correctness of the work, and no claim that external effects happen once. Everything in this section follows from taking those absences seriously.

LangGraph's documentation lists what its checkpointing enables: human-in-the-loop, memory, time travel and fault tolerance ("Checkpointers", LangChain/LangGraph documentation, retrieved 23 November 2026). Four capabilities, one mechanism - and that is a warning rather than a boast. The four have different requirements. Human-in-the-loop needs a durable pause. Time travel needs retained history. Fault tolerance needs resumability from the last boundary. Memory, as that same page is careful to say, is not the checkpointer at all: a checkpointer holds graph-state snapshots scoped to one thread, while a store holds application-defined key-value data across threads. A team that scopes its memory requirement to the checkpointing mechanism will discover at the first new conversation that it has built the wrong thing.

### 4.2 Two mechanisms, not one

"Durable execution" names at least two mechanisms with genuinely different semantics, and a platform that claims the property without naming the mechanism has said nothing yet. This repository documents both families, and the two are not variations on a theme.

**Snapshot-checkpoint.** A checkpointer "saves a snapshot of graph state at each super-step, organized into threads". The thread identifier is the primary key for storing and retrieving that state, and without it the checkpointer cannot save state or resume after an interrupt. The mechanism has a detail that matters more than its headline: pending writes. When a node fails mid-execution at a super-step, LangGraph stores the pending checkpoint writes from the other nodes that completed successfully at that super-step, so that resuming from that super-step does not re-run the successful nodes (all from "Checkpointers", retrieved 23 November 2026). Durability here is *positional*: the system records a position and the partial results at it.

**Event-history replay.** Temporal does not work this way, and its documentation says so in as many words: "Temporal doesn't restore memory from a snapshot. It starts the Workflow code from the beginning, replays the Event History step by step, and uses that history to guide the code back to the exact state as before" ("Workflows", retrieved 23 November 2026). Durability here is *derivational*: the system records the inputs and the ordering, and re-derives the state by running the code again. Each state transition is recorded in a persistence store ("Workflow Execution", retrieved 23 November 2026).

| Dimension | Snapshot-checkpoint (LangGraph) | Event-history replay (Temporal) |
|---|---|---|
| What is stored | Graph state at each super-step, per thread | Ordered event history of commands and results |
| What is restored | The snapshot, applied to the graph | State re-derived by re-executing the code |
| What re-executes | Nodes after the checkpoint | The workflow code, from the beginning |
| What does not | Nodes before the checkpoint | Recorded Activity results |
| Scope identity | Thread id, primary key for retrieval | Workflow Id plus Run Id |
| Failure mode if scope is wrong | Pending work invisible; resume re-does unseen steps | Non-deterministic error, or replay of an evicted cache |

The two have opposite dispositions toward re-execution, and that is not a detail - it determines what your tool boundary must tolerate. In the replay family, "when a Workflow calls an Activity, the Activity runs once, its result is recorded in the Event History. During replay, that result is reused, not recomputed. So Activities aren't executed again during replay" ("Workflows", retrieved 23 November 2026). In the checkpoint family, replay re-executes. Section 4.3 takes the consequence; section 11 takes the idempotency consequence.

A platform may host both. Platform guide section 2.5 states exactly that pattern: one engine for the deterministic spine, an agent graph engine for the agentic paths, joined by an event bus. The architecture consequence is under-discussed: if a LangGraph subgraph runs inside a Temporal workflow, the run then has *two* recovery mechanisms with different granularities, and the platform must decide which one is the unit of recovery for which step. Left undecided, the answer defaults to the outermost, and the inner graph's checkpoints become expensive decoration.

### 4.3 The determinism requirement

Event-history replay imposes a constraint that snapshot-checkpoint does not, and it is the single most demanding requirement in this section. The workflow "must take care to ensure that any time your Workflow code is executed it makes the same Workflow API calls in the same sequence, given the same input" ("Workflow Definition", Temporal documentation, retrieved 23 November 2026). Non-deterministic operations - "like API calls, LLM/AI invocations, database queries, and other external interactions" - belong in Activities, which "execute outside the replay path". Commands are compared with the existing Event History position by position, and a mismatch produces a non-deterministic error.

The documentation names three concrete sources of the problem. Because the workflow is re-executed, "It has to make the same decisions when given the same history. It shouldn't depend on any values not recorded in the history which would be different between runs" - and the examples given are `Date.now()`, a random number, and a network call not performed inside an Activity ("Workflows", retrieved 23 November 2026). The mitigation is a set of replay-safe forms: time read from workflow context, timers recorded as events, randomness captured once. It names two causes of a non-deterministic error: code changes to a running workflow, and intrinsic non-deterministic logic. Versioning and patching exist for the first.

The rule that generalises beyond either engine is this: **any interaction with the outside world must either be moved outside the replay path or have its result recorded.** Temporal's mechanism is the Activity. LangGraph's mechanism is the checkpoint plus re-execution - and here the two families diverge in a way that no amount of abstraction hides, because LangGraph's replay page states that "Replay re-executes nodes - it doesn't just read from cache. LLM calls, API requests, and interrupts fire again and may return different results" ("Use time-travel", retrieved 23 November 2026).

That sentence is the sharpest published statement in this repository's source set that **replay is not reproducibility**. In the replay family, the model is not called again; in the checkpoint family, it is, and it may answer differently. Two platforms can both truthfully say "we replay runs" while meaning opposite things about whether the model is invoked. An architect who does not ask which one is meant will design a control - say, a deterministic re-derivation of a decision for audit - that holds on one platform and silently fails on the other.

### 4.4 The four conditions for a faithful replay

A replay is a faithful reconstruction only if four conditions hold. Each has a specific breaker, and each breaker is a normal event in a production platform rather than an exotic one.

| Condition | What breaks it | Mechanism that pins it |
|---|---|---|
| Same code version | A deploy while runs are live | Versioning or patching ("Workflow Definition") |
| Same inputs | Reading a mutable source at replay time | Inputs recorded, not re-read ("Workflows") |
| Same ordering of external results | Concurrent completion arriving in a different order | Results recorded positionally in the history |
| Same non-determinism pinned | Wall clock, randomness | Time from workflow context; randomness captured once |

There is a fifth condition that cannot be satisfied by any mechanism documented in this pass, and honesty requires naming it separately: **the model must return the same output**. In the checkpoint family the call fires again (section 4.3), so a replay reconstructs the *path* and not the *answer*. Pinning model outputs is possible in principle - record the completion, replay from the record - but no source read for this guide documents a platform that does it for a complete multi-agent run, and section 15 records that as an unverified gap rather than a capability.

The practical lesson is that the four conditions are properties of the *record*, not of the engine. An engine provides the hooks; whether the run's record actually contains the code version, the inputs, the result ordering and the pinned non-determinism is an architecture decision made by whoever designed the workflow. Section 3.2's version set exists for exactly this reason.

### 4.5 Consequences for the platform

**Checkpoint granularity is the unit of recovery, and therefore the unit of duplicate risk.** If the checkpoint falls after an external write, recovery re-does the write. If it falls before, recovery re-does the work that led to the write. No choice of granularity avoids both; this is why section 11 puts the burden on the idempotency key rather than on tuning the boundary. Granularity decides how much work is repeated; the key decides whether repeating it is safe.

**A subgraph without its own checkpointer collapses to one super-step.** LangGraph's documentation is explicit: the parent treats such a subgraph as a single super-step, and "You cannot time travel to a point between nodes in a default subgraph"; compiling the subgraph with `checkpointer=True` creates per-step checkpoints inside it ("Use time-travel", retrieved 23 November 2026). The operational consequence is a disagreement the operator will meet as a bug: the run ledger records six steps, the engine can resume from two positions, and a recovery re-does four steps the ledger says already succeeded. That disagreement is not an engine defect - it is the record's granularity and the checkpoint's granularity having been designed by different people.

**Checkpoint retention is a records-management decision, not a storage-tuning decision.** Checkpoints accumulate and need a retention policy, and the in-memory saver holds them in RAM and loses them on process restart, while the persistent savers (Postgres, SQLite) do not ("Persistence", retrieved 23 November 2026). Two things follow. First, an in-memory saver gives the run record and the checkpoint different lifetimes, which means a restart produces a record that claims progress the engine cannot resume from. Second, the retention window on checkpoints is the window in which a run can be reconstructed at all - so it belongs to whoever owns the records, not to whoever owns the disk. Note also that `update_state` does not roll back a thread: it creates a new checkpoint branching from the point. Correction in this model is append, not erase, which is what audit wants and what a naive storage budget does not anticipate.

### 4.6 The honest limit

Durability preserves progress. It does not prove correctness, and it does not provide exactly-once external effects. Temporal's own phrasing - "effectively once" - carries the qualification: the platform makes duplicate *execution* unlikely and makes the recorded path single, but an external system that was called outside the recorded path is not thereby protected from a second call.

So durability answers "will this run still exist tomorrow", and it leaves open "did this run do what it claimed, once". The first question is answered by the mechanism in this section. The second is answered by the idempotency and reconciliation machinery in section 11, and catalogued as failure modes in section 13.

## 5. The Coordination Topologies, Cross-Referenced

This section exists for one reason: the topology determines what the unit of record has to look like, and nothing else in this repository says how. It does **not** say which topology to choose. Topology selection and hybridization belong to [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md) sections 3, 4 and 5; hierarchy as a topology belongs to [hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md) sections 3 and 6; the banking deployment of either belongs to [multi_agent_banking_guide.md](multi_agent_banking_guide.md) section 3. Four shapes are enough to situate the layer, and each is described here only in terms of what it does to the record.

**Single loop.** One agent, one objective, iterations until a stop condition. The run record collapses to a linear step ledger with one owner, and there is no handoff and therefore no orphan. The trap is granularity: the natural step boundary in a loop is a model turn, and a model turn is not a business step, so the ledger over-records (forty turns for one decision) and under-specifies (nothing in the record says what the decision was). Worse, the most common way a loop ends is by exhausting its turn budget, and that outcome is recorded as a completion unless the record distinguishes "reached the objective" from "stopped iterating". Arguable claim: a single-loop run needs an explicit terminal state vocabulary *more* than a multi-agent run does, precisely because nothing else in the architecture will produce one.

**Graph.** Nodes and edges, with the path decided at run time. The record must add three things a linear ledger does not need: the node identity of each step (so the step can be joined to the code that ran), the graph version (so a replay knows which edge set it traversed - section 4.4), and the edge taken (because in a graph the path is data, and a record that stores only the nodes loses the decision). Fan-out is where graphs break records: a node that spawns N branches produces N child steps under one parent step id, and a record that flattens them cannot answer "which branch produced the write". Join semantics must be recorded explicitly, because "all branches completed" and "the first branch to succeed completed" are different obligations with different duplicate risk.

**Supervisor and worker.** Two levels of record, and the relationship between them is the whole problem. The supervisor's step and the worker's step are both real events with different owners, so the record must nest rather than flatten (section 9 develops why flattening destroys attribution). The orphan risk is structural and symmetric: if the supervisor dies after delegating, the worker keeps working on an obligation nobody is waiting for; if the worker dies, the supervisor waits on a result that will never arrive and, in the common implementation, waits without a deadline. The record's defence is a lease on the delegated step with an expiry that produces a recorded transition - not a timeout that produces a log line.

**Peer and mesh.** N agents, no single owner, work moving laterally. This is the only topology in which "who writes the terminal state" is an open design question rather than an inherited one. The record cannot be a tree; it is a set of handoffs plus a join, and the join needs an appointed closer - a component whose only job is to decide that the obligation is discharged, refused or abandoned. Peer topologies also concentrate the double-act risk of section 6, because every lateral edge is a handoff between agents that each believe they can act.

| Topology | What the record must add | Dominant record risk |
|---|---|---|
| Single loop | Terminal-state vocabulary distinct from iteration exhaustion | A budget stop recorded as a completion |
| Graph | Node id, graph version, edge taken, explicit join semantics | Flattened fan-out; ambiguous join |
| Supervisor / worker | Nested steps; lease with expiry per delegated step | Orphan on either side of the delegation |
| Peer / mesh | Handoff set plus an appointed closer for the join | No owner of the terminal state; double act |

The one sentence to carry out of this section: a topology is a statement about *where the record branches*, and every branch is a place where a step can be lost without an error. Sections 6, 7 and 11 are the three mechanisms that make the branches safe.

### 5.1 The step identity scheme that fan-out forces

Fan-out looks like a scheduling detail and is actually a record-design decision, because a branch that names itself cannot be reconciled with a branch that was re-driven.

The rule is that branch step ids are assigned **by the parent at dispatch**, from the parent step id plus a branch index, and are never generated by the branch. The reason is section 11: a re-driven branch that invents its own identity produces two ledger steps for one obligation, and the ledger then cannot tell a duplicate from legitimate parallel work. Parent-assigned identity makes the second attempt recognisable as the same branch.

The join needs three recorded fields, not one. The number of branches expected, so a join cannot silently proceed on a partial fan-out. The join condition - all, any, or quorum - because "all branches completed" and "the first branch to succeed completed" are different obligations. And the identity of the branch whose result was accepted, because under an any-condition the losing branches may still be running and may still write: a join that records only "satisfied" leaves in-flight work that no step owns.

There is one more orphan risk specific to fan-out, and it sits on the parent. The parent holds the join, so if the parent dies the children continue executing toward a result nobody will collect. The record must therefore carry the parent's lease alongside the children's step entries, so that a recovery process sees the join obligation as outstanding rather than inferring it from a set of completed children.

## 6. Handoffs

### 6.1 The gap this section fills

Nothing in this repository specifies handoff semantics. That is a strong statement, so here is what the nearest siblings actually own. [multi_agent_debugging_guide.md](multi_agent_debugging_guide.md) section 3 catalogues interaction-layer failures, section 5 attributes a failure to an agent, and section 8 gives diagnosis patterns - all of it about a handoff that has already gone wrong. [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md) section 5 describes control patterns, which say who decides, not what travels between agents. The A2A specification defines a message and a task lifecycle (section 12), not an accountability transfer.

So the contract below is **this guide's design**, marked as such in the claims audit (section 15) with the note that no standard was found for it. It is offered as a design because the alternative - each platform inventing one per workflow - produces the failure in 6.5 at every seam.

### 6.2 A handoff is a state transition with three transfers

It is tempting to model a handoff as a message. That model is wrong in a way that is expensive, because a message can be duplicated, retried, dropped and acknowledged ambiguously, while an accountability transfer must happen exactly once. The useful model is a state transition of the run in which three distinct things move.

**Context - what the receiver may know.** Not what the sender knows: a subset, chosen, with a manifest that says what was handed over. The receiver that receives everything receives the sender's uncertainty as well as its facts, and in a dispute nobody can say what the receiver was working from.

**Authority - what the receiver may do, on whose behalf, with what limits.** A handoff that transfers work without transferring (or explicitly declining to transfer) authority produces either a receiver that cannot act or a receiver that acts under its own identity against the sender's obligation. The second is worse, because the audit trail then attributes the effect to the wrong principal.

**Obligation - who is accountable for the step, and when that accountability is discharged.** This is the transfer almost every implementation omits, and it is the one this guide exists to insist on. When A hands work to B, A's accountability does not end when the message is sent; it ends when B accepts, or when the deadline passes and the handoff is recorded as failed. The record must therefore contain the acceptance, not just the dispatch.

### 6.3 The contract fields

| Field | Why the handoff is unsafe without it |
|---|---|
| Handoff id | It is the idempotency key for the transfer; without a stable one, a retry creates a second obligation |
| Run id and parent step id | Joins the handoff to the run record; without it a cross-agent step cannot be attributed (section 9) |
| From and to, with agent versions | Accountability needs a principal and a version, not a name (section 7) |
| Objective | What the receiver is being asked to achieve, stated once, in the run's terms |
| Acceptance test | Who judges completion, and by what test; without it "done" is the receiver's opinion |
| Authority grant | Scope, budget, expiry, delegation depth; without limits a delegated agent can delegate without bound |
| Context manifest | References plus digests to what was handed over, not a copy (see 6.4) |
| Deadline | When the obligation returns to the sender; without it a lost handoff waits forever |
| Return contract | The shape of a result and the shape of a refusal; a refusal is a valid outcome and must be expressible |
| Single-writer rule | Nomination of which side writes the ledger for this step; without it both sides write, or neither does |

The single-writer rule is the least obvious and the most load-bearing. At any instant exactly one side owns the obligation, and only the owner may write the step's record. Transitions - accept, decline, complete, fail, expire - are writes by the current owner. Everything else in the table describes *what* is transferred; this row describes *who is allowed to say it happened*.

### 6.4 References versus copies, and the counter-argument

The manifest holds references plus digests rather than payload copies, for three good reasons: the run record stays small enough to retain for years, the referenced document remains the single authoritative version, and the digest detects drift without storing content. This is the right default.

But the counter-argument is real and should be answered rather than waved: **references can be lost.** Permissions change, documents are deleted or superseded, retention expires on the source system, and a reference that resolved at dispatch resolves to nothing at audit. A manifest of pure references is a record that says "the receiver was told to look at document 4471" and cannot say what 4471 said.

The resolution is not to copy everything - that produces a record nobody can retain. It is to keep the reference plus digest, and additionally capture the *subset of content the acceptance test depends on*: the fields the receiver must have seen to decide, the values it must have compared. That subset is small by construction, because an acceptance test is small. A handoff whose acceptance test cannot be evaluated from a bounded extract has an acceptance test that is really a judgement, and the record should say so.

### 6.5 The double-act failure

The characteristic handoff defect is not a lost message. It is a message received twice, or a message received once and acted on by both sides. The mechanism is mundane and worth spelling out because it recurs in every platform that hand-rolls delegation.

The sender dispatches, then times out waiting. The receiver had in fact accepted and is working. The sender, treating the timeout as failure, retries - and because the retry carries a new message identity rather than the handoff identity, the receiver treats it as a second piece of work. Both copies now run. If the receiver's idempotency key for its external writes is derived from its own local task id rather than from the handoff id, the two copies produce two writes. And because neither side emits a terminal acknowledgement that the *other* side records, both sides believe they hold the obligation. The run has two active holders, one of which will eventually finish and report, and one of which will finish later and report over the top.

The guardrail has four parts, and all four are needed. The handoff id is the idempotency key, so a retry is recognisable as the same transfer. Accept and decline are explicit recorded transitions, not inferred from the absence of an error. At-most-one active holder is enforced *by the record*, as a compare-and-set on the obligation - not by convention and not by the agents' good behaviour. And the holding is a lease with an expiry that produces a recorded transition when it lapses, so a dead holder does not block the obligation forever and a live holder is not silently dispossessed.

### 6.6 What A2A provides, and what it leaves to the platform

The A2A specification (version 1.0.0, retrieved 23 November 2026) supplies a task lifecycle that maps well onto the middle of this contract. A Task is "the fundamental unit of work managed by A2A, identified by a unique ID. Tasks are stateful and progress through a defined lifecycle", with states SUBMITTED, WORKING, COMPLETED, FAILED, CANCELED, INPUT_REQUIRED, REJECTED and AUTH_REQUIRED; COMPLETED, FAILED, CANCELED and REJECTED are terminal, while INPUT_REQUIRED and AUTH_REQUIRED are interrupted. Sending a message to a task in a terminal state raises `UnsupportedOperationError`.

That vocabulary covers acceptance (SUBMITTED, WORKING), progress, human or system interruption, and terminal outcomes. It gives the platform a state machine to map onto its ledger (section 12). What it does not do is decide **who owns the obligation**. A task can be COMPLETED while the run is unfinished, because the task belonged to one agent's scope and the obligation belonged to the business process. Ownership is a platform decision expressed in the run record, not a protocol property.

The second A2A property matters just as much here. The specification's guiding principle of **Opaque Execution** states that agents collaborate "based on declared capabilities and exchanged information, without needing to share their internal thoughts, plans, or tool implementations", so that they can interact "without needing access to each other's internal state, memory, or tools". The consequence for the record is direct: the platform cannot infer completion from the receiver's reasoning, because it is not entitled to see the reasoning. It can only read the receiver's *declared state*. Sections 7 and 12 draw the conclusion - the record must distinguish steps the platform attested locally from steps a remote agent merely declared.

## 7. Registry and Discovery

### 7.1 The narrow view this section takes

Platform guide section 2.6 owns the agent registry as an inventory: what agents exist, their metadata, their compliance status, their lifecycle from proposed to retired. That is an ownership question and it is settled there. This section takes one slice of it - the orchestration-time view - and asks two questions only: how does a name become a running agent at the moment of dispatch, and what happens when the registry and reality disagree during a run.

The distinction is not academic. An inventory is read by people and by governance processes; a resolution is read by the coordinator, once per dispatch, on the critical path of a business process. They can be the same store and usually are, but they have different correctness requirements, and conflating them produces the failure in 7.4.

### 7.2 Resolution at dispatch, not at build

The wrong default, and the one most platforms ship with, is resolution at build time: the workflow is configured with an endpoint, the endpoint is baked into a deployment, and dispatch is a call to a constant. It is wrong because of a fact this guide has already established twice - a run outlives deployments. A workflow started in March may dispatch to a worker agent in June, and the endpoint compiled in March may name a version that no longer exists or, worse, a version that has been replaced by one with different behaviour under the same name.

Resolution at dispatch means the coordinator takes a *name* and resolves it, at that moment, to three things: an endpoint, a version, and a contract. All three are recorded in the step. The version is what section 3.2's version set needs; the contract is what the handoff needs (section 6.3's context manifest is meaningless if the receiver's accepted input shape is unknown); and the endpoint is the only part that is genuinely ephemeral.

The snapshot is the point. Because resolution happens at dispatch, the registry's state at dispatch time must be written into the run record - not referenced, written, at least for the fields the step depended on. A record that says "resolved via the registry" tells an auditor nothing, because the registry of today is not the registry of the run. A record that carries the resolved version and contract digest can answer the only question that matters: was this step executed by the agent the platform believed it was executing?

### 7.3 Three ways the registry and reality disagree

| Disagreement | What the operator observes | Guardrail |
|---|---|---|
| The registry lists an agent that does not exist | Dispatch fails at resolution, or succeeds and the call fails | Resolve and verify at dispatch; treat resolution failure as a typed step failure, not a retry loop |
| Reality includes an agent the registry does not list | A caller appears in the ledger with no inventory entry | Unknown-agent refusal by default; the platform cannot govern what it cannot name |
| Both exist, but the contract differs | The step runs and returns something the caller cannot use | Contract digest in the handoff; reject on mismatch rather than adapt silently |

The second row is the one that costs governance rather than engineering. A shadow agent - deployed outside the registry, reachable, functional - is a genuine capability the platform has no inventory entry for, which means no owner, no compliance status and no lifecycle. It will be discovered during reconciliation (section 11) as an unattributed principal, and the fix is not a better registry read but a refusal: an agent that is not in the inventory is not dispatched to, even if it is reachable. That refusal is unpopular precisely when it is working, which is why it needs to be a platform default rather than a per-workflow choice.

The third row is version skew, and it is the reason the contract must be carried as a digest rather than as a promise. The registry says agent X accepts a dispute reference and returns a decision; the deployed agent X accepts a dispute reference and returns a decision *plus* a required follow-up action the caller does not implement. Silent adaptation - the caller ignoring the extra field - produces a run that completes and has not done what it claimed. Rejecting on digest mismatch is the only guardrail that fails loudly, and section 2's whole argument is that this layer needs more loud failures, not fewer.

A registry that is authoritative for existence but not for contract is a name service, not a registry. That is the sentence to take from this subsection: the inventory's job is to say what exists, the resolution's job is to say what was actually used.

### 7.4 Discovery versus registry: self-description and attestation

A2A provides discovery in the form of the Agent Card: "a JSON metadata document published by an A2A Server" describing an agent's "identity, capabilities, skills, service endpoint and authentication requirements" (A2A Protocol Specification, version 1.0.0, retrieved 23 November 2026). The card also carries an optional context identifier "to logically group related tasks and messages".

An Agent Card is **self-description by the provider**. A platform registry is **attestation by the owner**. The difference matters exactly once, and then it matters completely: under audit. A card says what the agent claims it can do; a registry entry says that the enterprise has accepted that claim, assigned it an owner, and recorded a compliance status against it. When an auditor asks why an agent was permitted to touch a customer record, the card is the agent's own testimony and the registry entry is the enterprise's decision. Section 12 returns to this when it maps remote declared state onto the local ledger.

Neither is redundant. Cards are how agents find each other across organisational boundaries where no shared registry exists - the case A2A was designed for. The registry is how an enterprise knows what it has. A platform that treats a card as an inventory has outsourced its governance to the thing it is governing.

## 8. Tenancy, Quotas and Isolation

### 8.1 What a runtime cannot enforce

The agent runtime isolates one execution. Platform guide section 4.2 scopes it to that: a sandbox, one agent, bounded resources. The runtime is constitutionally unable to answer the questions tenancy asks, because every one of them is a question about *relationships between executions* rather than about an execution.

Four decisions belong to this layer and have no home in the runtime. How many executions a tenant may have in flight at once, which is a statement about a shared pool and therefore about other tenants. Which tenants may call which agents, which is a statement about the graph, not about a node. Whose authority a cross-tenant handoff carries, which is a statement about delegation across a boundary the runtime does not know exists. And what happens to in-flight runs when a tenant is suspended, which is a statement about obligations the runtime is holding open.

The fourth is the one that produces the guide's signature failure. Everything above it in this section is capacity management; the fourth is about whether an administrative action silently creates a class of runs that never reach a terminal state.

### 8.2 Three resources to quota

Quotas are usually expressed as one number - concurrent runs - and one number is not enough, because the three resources this layer controls fail differently.

| Resource | Unit | What breaks without the quota |
|---|---|---|
| Concurrent runs | Active runs per tenant | One tenant's backlog consumes the shared worker pool; others queue |
| Step budget | Steps per run, and steps per tenant per period | A runaway loop is bounded per run but unbounded in aggregate |
| Effect budget | Writes to a system of record per tenant per period | The wrong agent, or a duplicated loop, generates writes at machine pace against a system that cannot refuse them |

The third is the one architects forget, and it is the one that damages an institution rather than a platform. The first two quotas protect the platform from its tenants; the effect budget protects the *system of record* from the platform. A dispute-handling agent that writes to a case-management system has an effect budget whether or not anyone declared one - the only question is whether the platform enforces it or the downstream system discovers it as an incident.

Each quota needs the same three properties, and a quota missing any of them is a number in a dashboard rather than a control: it must be *enforced at admission* rather than reported after the fact; it must be *attached to the run record*, so the run that hit it can be identified; and it must have a *defined behaviour at the limit* - queue, refuse, or degrade. A quota that is exceeded and silently ignored is indistinguishable from no quota, and belongs in section 13 as mode (e).

### 8.3 Fairness is not the same as isolation

Two tenants can each be within quota and still starve each other, because quota is a per-tenant ceiling while starvation is a property of the shared queue between them. A single long run from tenant A occupying the only worker capable of a particular agent keeps tenant B's short runs waiting, and no per-tenant number reflects that.

The control is queue and lease design, not quota size. Per-tenant admission queues with weighted service, plus leases on the workers that a run holds, bound the time a single run can occupy a shared resource. This is a place where the layer's design and an operations team's tuning meet, and the argument worth making is that the *design* has to exist first: a platform with per-tenant queues can be tuned toward fairness, while a platform with one global queue can only be tuned toward aggregate utilisation. Given that a run must reach a terminal state, fairness is the property that matters, because queueing behind another tenant is not a failure - waiting forever is.

Isolation, properly, is a different axis: it is the guarantee that one tenant's data, authority and failure cannot reach another's. The runtime's sandbox gives you process isolation. The layer gives you authority isolation, by refusing cross-tenant dispatch unless an explicit grant exists (section 6.3's authority field, scoped to a tenant). A cross-tenant handoff is legitimate - a group-wide fraud agent serving several business units is a normal design - but only when the grant names both tenants and the record shows it. An implicit cross-tenant call, made because both tenants happen to share a worker pool and a service account, is an isolation failure that no quota will detect.

### 8.4 Suspension must not orphan

When a tenant is suspended - for a compliance hold, a commercial dispute, or an incident - the platform has three options and only one of them is defensible. It can pause admission, drain in-flight runs to a recorded terminal state, and resume later. It can pause admission and abandon in-flight runs, which creates exactly the population section 10 is about: runs with no terminal state, owned by a tenant nobody may contact. Or it can keep running in-flight work, which means suspension did not suspend anything.

The defensible option requires a terminal vocabulary that includes an administrative state, and this is the design decision most platforms miss. "Completed" and "failed" are outcomes of the business process. A suspended tenant's run has not reached either - it has been *parked*, and parking must be a recorded state with an owner, a reason, and a resume condition. Without that value, the platform's only options are to lie (record failed) or to orphan (record nothing).

The record fields that make parking safe are the same as the ones that make recovery safe: the last completed step, the authority grant's expiry (which may lapse while parked, requiring re-grant on resume), the lease state, and the effects already written. A parked run resumed with a lapsed grant and a stale lease is a duplicate-effect incident waiting for a schedule, which is why section 11's decision tree takes the lease and the grant as inputs before it decides to resume.

## 9. Attribution

### 9.1 Why the layer is the only accurate place

Attribution means answering two questions about every unit of work: what did it cost, and who is accountable for it. Three services hold partial answers, and none of them can hold the whole one.

**The agent** knows its own model calls. It does not know the run's total, because the run may span agents it never sees, retries it is unaware of, and steps that happen after it has returned. An agent's self-report is also the one number in the system produced by the party with an interest in it being low.

**The gateway** meters calls with high fidelity - it sees every completion request (platform guide section 4.1). But a gateway meters *calls*, and a business obligation is not a call. It cannot say which dispute a completion served, because the request that arrives at the gateway carries a model name and a prompt and, unless the caller supplies it, no run identity. A gateway is the right meter and the wrong ledger.

**The trace** has spans, and spans are not billing units. Two reasons, both structural. Spans are shaped for diagnosis, so their boundaries follow the instrumentation rather than the obligation - a retried step may be one span or three depending on where the retry was implemented. And spans can be sampled, which is correct for diagnosis and fatal for chargeback: an unsampled run has no cost, and a cost report that silently omits some runs is worse than no report, because it looks complete. [agentops_guide.md](agentops_guide.md) sections 3 and 6 own trace and cost operations, and this section does not rebuild them; it only states the boundary - those signals are derived from emitted telemetry, and attribution needs a record that is emitted unconditionally.

The coordinator is the only component present at every step of a run, and therefore the only one that can attribute. That is not a claim about competence; it is a claim about position.

### 9.2 The attribution key

The key is emitted by the coordinator as part of the step record, not computed later by joining logs. Six fields:

| Field | Why it is in the key |
|---|---|
| Run id | Joins cost to the business obligation, which is the only unit anyone budgets in |
| Step id | Separates the cost of a decision from the cost of its retries |
| Tenant | Makes the chargeback boundary enforceable rather than negotiated |
| Agent version | Separates the cost of a behaviour change from the cost of more work |
| Model | Separates cost driven by task from cost driven by routing (platform guide section 2.5) |
| Tool | Attributes effects, not just tokens; a step that writes to a system of record costs something the model never sees |

The last field is the one that makes the key an *accountability* key rather than a billing key. A key that stops at model and tokens answers "what did this run cost". A key that includes the tool and the step answers "which step spent it, and what did that step do" - which is the question asked when a business unit disputes a charge or an auditor asks who authorized an effect.

### 9.3 Chargeback, showback, and the audit consequence

Showback reports cost to a tenant; chargeback moves it. The difference is organisational, but the record requirement is identical, and both fail in the same way: an unattributable run is an unallocatable cost, which in practice means it is absorbed centrally, which means the incentive to control it disappears. That is a governance outcome produced by a missing field.

The audit consequence is stronger than the financial one. An unattributable run is also an unattributable *action*: if the ledger cannot say which tenant's authority produced a write to a system of record, then the write has no principal, and a write with no principal is indistinguishable from one produced by an intruder or by a defect. Section 13's mode (h) is the general form of this; attribution is its cheapest prevention.

### 9.4 Delegation depth and double counting

A supervisor's step and its worker's step are both real. The supervisor spent tokens deciding to delegate and evaluating the result; the worker spent tokens doing the work. Both costs exist, and both belong to the run.

The record must **nest** rather than flatten. Flattened, the two steps are siblings and the reader cannot tell whether the supervisor's cost includes the worker's or is additional to it - and in practice implementations get this wrong in both directions, double-counting the worker under the supervisor and again under the run, or counting only the supervisor and losing the work. Nested, the structure is unambiguous: the worker's step is a child of the delegation step, its cost rolls up, and the sum is computed once at the root by a rule rather than by a report writer's judgement.

The nesting has a second use that costs nothing extra: depth becomes a recorded property. Delegation depth is bounded by the authority grant (section 6.3), and the record's nesting is what makes the bound checkable after the fact - an auditor can see that a chain of four delegations occurred under a grant that permitted two, which is a governance finding no aggregate report would surface.

### 9.5 Steps, attempts, and what a retry does to attribution

A retry is not a new step. It is a new attempt of the same step, and the distinction has to survive into the record or attribution breaks in one of two directions.

Collapse attempts into the step and a step that was retried four times looks like one expensive decision, with the retry cost folded in and its cause - a tool that kept failing - invisible. Promote attempts to steps and the ledger's step count stops meaning anything, delegation nesting breaks, and section 10's stuck-run analysis starts counting healthy retries as progress.

The record therefore carries an attempt identifier under the step, and the attribution key of 9.2 gains it as a dimension while cost still rolls up at the step. The argument for doing this is not bookkeeping neatness: retry cost is the only cost in this layer that engineering can reduce rather than negotiate, and it can only be reduced if it is separable from the cost of the decision being retried. A platform that reports one number per step has made its most tractable inefficiency invisible.

## 10. The Operator's View

### 10.1 What "stuck" means when nothing failed

The operational signature of this layer is a run that has a non-terminal state and has had no step transition within its expected window. Nothing failed. No error was raised, no budget was exceeded, no policy was violated, no sandbox died. The coordinator simply did not schedule the next step - because a message was lost, a lease lapsed without a reclaim, a condition was never satisfied, a worker acknowledged and then did nothing, or a human task was assigned to someone who left.

There is nothing in that description to alert on, and that is the whole difficulty. Every alerting mechanism an operations team already has is a predicate over emitted events: an error rate, a saturation threshold, a failed health check. This failure emits no event, so a rule that watches events cannot fire. The platform has to be engineered to notice an *absence*, and an absence is only noticeable against an expectation.

The expectation has to be written down at admission. This is why section 3.2 puts the intent and the acceptance test in the record and why this section insists on an expected duration per workflow type: a run whose expected window is unknown cannot be classified as stuck, only as old.

### 10.2 Three observables that must be invented

| Observable | Definition | Alert form |
|---|---|---|
| Run age against expected duration | Time since admission, compared with the window declared for that workflow type | Count of runs past their window; should be zero |
| Absence of the next expected transition | The record names the next state it is waiting for and the deadline for arriving there | A heartbeat with a deadline, asserted by a checker |
| Terminal-state census | Runs per cohort that have not reached any terminal state within the cohort's window | Expected completions minus observed completions |

The second is the one that turns a log line into a control. A heartbeat that is merely *written* is a log line: it tells you the run was alive at a moment. A heartbeat with a deadline is an assertion - the writer declares when it will next write, and a checker treats the missed deadline as an event. The difference is not instrumentation quality; it is which component holds the expectation. When the writer holds it, a dead writer cannot report its own death. When a separate checker holds it, the death is observable.

The third observable is the strongest because it does not depend on the platform's own timers. If the platform knows how many runs were admitted for a workflow type in a window - from the intake queue, from the trigger source, from the ledger's own admission records - and how many reached a terminal state, the difference is the stuck population, computed from two independent counts. That is a much harder signal to fool than a per-run timer, because a run that never wrote anything at all still appears in the admitted count.

### 10.3 Why log-based alerting cannot find this

[agentops_guide.md](agentops_guide.md) section 5 owns monitoring and alerting mechanics, and this section does not rebuild it. But one property of that machinery has to be stated plainly here: its signals are derived from emitted telemetry. Error rates come from error events, saturation from resource metrics, loop detection from repeated spans. All of them require that something was emitted.

When a coordinator stops scheduling, nothing is emitted. There is no error event because no error occurred; there is no saturation metric because no resource is under pressure; there is no repeated span because no span was started. A "no logs for this service" rule catches the case where the whole service dies, and that case is already covered by the platform's health checks. It does not catch the case where the service is healthy, serving other runs, and has silently stopped advancing one of them - which is the common case, because coordinators usually fail per run rather than per process.

The rule that follows is uncomfortable for teams raised on log-based alerting: **alerts for this layer must be assertions with an expected value, not patterns in what was emitted.** "This count should be zero" and "this rate should be non-zero" are the only two forms that work, and both require the platform to know what it expected. That expectation is an architecture artifact, produced at design time, not a query written during an incident.

### 10.4 The operator's three questions

An operator facing a stuck run asks three questions in order, and the record either answers them or the operator escalates to reading code.

**What is stuck?** Answered by run id, business-process id, workflow type, admission time, last transition and its timestamp, and current state. Without the last-transition timestamp the operator cannot tell a run that is progressing slowly from one that stopped, and the first is not an incident.

**What is it waiting for?** Answered by the pending dependency: the outstanding handoff id and its holder, the lease and its expiry, the child step ids that have not returned, the human task and its assignee. This is the field set that separates a platform from a dashboard. A dashboard shows that a run is old; the record shows that it is waiting on a lease held by a worker that was decommissioned.

**What will it do next?** Answered by the next scheduled action and its deadline - the transition the coordinator intends to make, and when. If the record cannot answer this, the run is not merely stuck, it is unrecoverable without reading the workflow definition, which in a versioned deployment may no longer be the version that is running (section 4.4).

A platform that can answer all three has a stuck-run queue that a support function can work. A platform that can answer only the first has a queue that only the original author can work, which is the operational form of the bus-factor problem and the subject of section 14's closing cost.

## 11. Recovery and Reconciliation

### 11.1 Orphaned runs and the decision tree

A run is orphaned when no component is advancing it and no component will. Five causes account for almost all of them: the owner process is gone and its lease has not been reclaimed; the lease expired while the holder was alive but slow; the engine lost or never persisted the history it needed; the tenant was suspended (section 8.4); or a deployment removed the code version the run was executing (section 4.4's first breaker).

For each, there are four possible actions and only one of them is safe given what the record contains. This is the decision tree, and it is a tree over the record's fields rather than over the failure's cause.

| Action | Safe when | What the record must contain |
|---|---|---|
| Resume | The holder is confirmably gone and no effect after the last completed step was written | Last completed step id, lease state, and a confirmed negative on external effects |
| Re-drive from a checkpoint | A checkpoint exists and the steps after it are known to be safe to repeat | Checkpoint position, and the idempotency key of every effect since |
| Compensate | Effects were written that must be undone, and each has a defined inverse | The effect list with keys, and the compensation registered for each |
| Declare failed | None of the above can be established within the deadline | The deadline, and the authority to write a terminal state on someone else's behalf |

The fourth row is the one that requires a governance decision rather than an engineering one. Declaring a run failed is an assertion about a business obligation made by a party that did not perform it, and someone must hold that authority explicitly. A platform where any operator can write "failed" has made the terminal state meaningless; a platform where nobody can has made orphaning permanent.

The dangerous confusion in this tree is between the first two rows. "Resume" means the run continues from where it is; "re-drive" means work is repeated from a known-good position. They look interchangeable and they are not: resume assumes the state is intact, re-drive assumes it may not be and reconstructs it. Choosing resume when the state was lost produces a run that believes it has done work it has not; choosing re-drive when it was not lost produces duplicates. The record's last-completed-step field, plus the effect log, is what distinguishes the cases - which is the concrete reason section 3.2 lists effects with keys as a first-class field.

### 11.2 Duplicate effects

The fundamental problem is short and has no tidy solution: recovery re-executes from a position, and the external system may already have acted on the work after that position. Whether it did is not knowable from the platform's side without asking, and asking is itself an operation that can fail.

The two engine families differ here in a way that determines what the tool boundary must tolerate, and section 4.2 set up the contrast. In the replay family, an Activity "runs once, its result is recorded in the Event History. During replay, that result is reused, not recomputed. So Activities aren't executed again during replay" ("Workflows", Temporal documentation, retrieved 23 November 2026). Re-execution of the *workflow* does not re-execute the *Activity*, so the external call is not repeated. In the checkpoint family, replay "re-executes nodes - it doesn't just read from cache. LLM calls, API requests, and interrupts fire again and may return different results" ("Use time-travel", LangChain/LangGraph documentation, retrieved 23 November 2026). The external call is repeated.

The conclusion is not that one engine is better; it is that **the tool boundary must be idempotent in the checkpoint family and must be idempotent in the replay family anyway, for a different reason.** In the checkpoint family, replay re-fires the call, so the call must tolerate being made twice with the same intent. In the replay family, the Activity's result is reused on replay, but an Activity "orchestrates its full lifecycle, including retries" ("Activity Execution", Temporal documentation, retrieved 23 November 2026) - so the call is retried within normal operation, and a retry can land twice. Both families require the same property; they require it for different mechanisms.

The property is a stable idempotency key derived from run id and step id, and *stable across retries* is the hard part. A key generated inside the retried function is a new key on every attempt and provides no protection at all - the external system sees two distinct requests. The key must be computed from the run's identity before the first attempt and carried into every attempt, which means the coordinator, not the tool, owns key generation.

### 11.3 Reconciliation as the proof

Idempotency reduces duplicates. It does not prove their absence, and it cannot detect the other direction - work that was claimed and never done. Reconciliation is the mechanism that does, and it is a periodic comparison of the run ledger's claimed effects against the systems of record.

| Outcome | Meaning | What it produces |
|---|---|---|
| Claimed and present | The ledger and the system of record agree | Nothing; this is the expected case |
| Claimed and absent | The ledger says an effect happened; the system of record has no such effect | The silent non-completion - the thesis's failure, found |
| Present and unclaimed | The system of record holds an effect no run claims | A duplicate, or a shadow write from an agent outside the record |

The second row is the reason reconciliation is not optional, and it is worth being precise about why no other control finds it. A run that recorded an effect it did not perform has a complete, internally consistent ledger with a terminal state. Nothing in the platform is in an error state. Only a comparison against an independent system reveals the discrepancy, which means the mechanism is inherently cross-system and cannot be built from the platform's own tables.

The third row catches a concrete mechanism documented for the replay family: a Task Token is unique for an Activity execution, and "retries can cause a remote service using the Task Token to end up with an invalid one" - an Activity may fail after passing its token to a remote service but before returning, leaving that service holding a token that is no longer valid, and passing Activity Id plus Workflow Id is the documented alternative ("Activity Execution", retrieved 23 November 2026). The remote service then completes work using a token the platform no longer recognises. The platform's record shows an Activity that failed and retried; the remote system shows an effect that succeeded. That is "present and unclaimed", produced by an ordinary retry rather than by any defect in either system.

### 11.4 Reconciliation is a platform function

The temptation is to treat reconciliation as an incident activity: something an engineer does when a discrepancy is suspected. That framing guarantees the discrepancy is found late, by someone with the least context, under time pressure - and only for the runs someone happened to suspect.

Reconciliation is a platform function with three properties, and each is a design decision rather than an operational habit. It has an **owner** - a named function, not a team that happens to have database access. It has a **schedule**, frequent enough that the window between an effect and its detection is bounded by policy rather than by luck. And it has a **disposition** for each outcome: claimed-and-absent escalates as a possible non-completion, present-and-unclaimed escalates as a possible duplicate, and both write back to the run record so that the discrepancy is part of the record rather than part of a spreadsheet. A reconciliation that produces a report nobody acts on has converted a detectable failure back into a silent one.

## 12. Enterprise Integration and the A2A Boundary

### 12.1 What is owned elsewhere

Platform guide section 3.3 owns A2A's provenance and role in the platform, and section 3.1 owns the gateway path that model traffic takes. Neither is restated here. This section answers one narrow question: what must the orchestration layer add on top of the protocol, given that the protocol was designed for collaboration between agents that do not share an owner. The specification is version 1.0.0, retrieved 23 November 2026.

### 12.2 Four things the layer must add

**Run and step identity in the message metadata.** A2A's Task carries a unique ID and an optional context identifier "to logically group related tasks and messages". Neither is a run id, and a context identifier groups messages without asserting an obligation. If a cross-platform task is to be joinable to a local run record - which section 9's attribution and section 11's reconciliation both require - the run id and the parent step id must travel in the message metadata. Otherwise a remote task is a fact about a remote system that the local ledger cannot place.

**The authority grant travelling with the task.** The protocol carries authentication requirements and an AUTH_REQUIRED state, which establish *who is calling*. They do not carry *what the caller's obligation permits the callee to do on whose behalf*. Section 6.3's grant - scope, budget, expiry, delegation depth - must travel with the task, because a remote agent that receives work without limits has been delegated the caller's authority implicitly, and implicit delegation is exactly the thing an auditor will ask about.

**A terminal-state mapping that does not confuse a task with a run.** The protocol's states are a good vocabulary and a poor terminal answer.

| A2A TaskState | What the ledger records |
|---|---|
| SUBMITTED, WORKING | Step open; obligation held by the remote agent |
| INPUT_REQUIRED, AUTH_REQUIRED | Step interrupted; the ledger keeps an open obligation with a deadline, not a completion |
| COMPLETED | **Step** complete; the run's terminal state is decided locally |
| FAILED, CANCELED, REJECTED | Step terminal with a reason; the run may continue, compensate, or terminate |
| TASK_STATE_UNSPECIFIED | Not a state; refuse to record it rather than silently treating it as pending |

The COMPLETED row is the one that matters. A task is "the fundamental unit of work managed by A2A" and its lifecycle closes when the task is done - but the task belonged to one agent's scope (section 6.6). Mapping COMPLETED straight onto a run's terminal state converts a local step completion into a business conclusion the remote agent never made.

**The distinction between attested and declared steps.** The specification's principle of Opaque Execution means the platform "cannot" - is not entitled to - see a remote agent's internal steps. So the ledger must mark provenance: a step executed by an agent the platform runs is *attested* (the platform saw the invocation and the effect); a step reported by a remote agent is *declared* (the platform saw a state transition and nothing else). Both belong in the record, and they must not be indistinguishable, because an audit that treats a declared step as attested is relying on the counterparty's own testimony. The specification does help here: sending a message to a task in a terminal state raises `UnsupportedOperationError`, and streaming closes when the task reaches a terminal state, so a terminal declaration cannot be silently continued - but a declaration is still a declaration.

### 12.3 Blocking, polling, and who holds the deadline

The specification offers two interaction modes: blocking, which waits for a terminal or interrupted state, and non-blocking, which returns immediately and leaves the caller to poll (A2A Protocol Specification, version 1.0.0, retrieved 23 November 2026). The choice looks like an implementation preference and is actually a decision about who holds the deadline for the obligation.

In blocking mode the caller's call is the deadline: if the remote task never terminates, the call eventually fails and the step has an outcome. In non-blocking mode the caller must own a poll schedule *and* a deadline, because nothing else in the system will end the wait. Each poll should refresh the step's lease, which makes the remote task's liveness visible as a heartbeat rather than inferred from the absence of a result - and makes a remote agent that stopped responding appear as a stalled step instead of as a run that is waiting normally.

The failure this prevents is the most common form of mode (a) in section 13: non-blocking dispatch, no deadline, and an open obligation with no clock. The record shows a step in progress, the trace shows a successful send, and nothing in the platform is in an error state. Note also what a poll is not: polls are not steps and not progress. A record that logs each poll as a step inflates the ledger and breaks the step-to-attempt separation of section 9.5, which is why the lease, not the ledger entry, is the right place for liveness.

### 12.4 MCP is not a handoff

The specification is explicit that the two protocols are complementary: A2A is agent-to-agent collaboration, MCP is agent-to-tool and agent-to-resource (Appendix B). The orchestration consequence is a modelling rule: **a tool call is a step in this layer's ledger, not a handoff.** It has no obligation transfer, no authority transfer to a peer, and no second accountable party - the tool acts under the caller's authority and returns a result. A platform that models MCP calls as handoffs will produce two records per tool call, two owners for one obligation, and a double-act failure (section 6.5) that no guardrail in section 6 was designed to prevent. Tool calls belong in the step's effect list, with their idempotency keys, and nowhere else.

## 13. The Failure Modes

Eight modes, each written as the thing an operator would actually observe rather than as a category. Every one of them is silent at the moment it occurs; that is why they are here rather than in a debugging guide. Sections 3 through 12 are the guardrails; this section is the checklist to test them against.

### (a) A run with no terminal state

**Symptom.** The exception report lists a dispute opened eleven days ago as in progress. A customer has called twice about it. The trace for the run is complete, showing every step and no error.

**Cause.** The final transition was never written. The usual mechanism: the coordinator wrote the last external effect, then died before writing the terminal state, and recovery resumed from a checkpoint that already contained the effect - so resuming produces no new transition and the run stays open. The second most common cause is a human task that was created and assigned to nobody.

**Guardrail.** The terminal transition must be written by a component that does not share a failure domain with the work it terminates, and the intent to write it must be recorded before the last effect, so recovery knows a completion was pending. The terminal-state census (section 10.2) is what finds this class at scale.

### (b) A replay that duplicates an external effect

**Symptom.** Two case notes with identical text and different identifiers, minutes apart. The run record shows one step. The downstream system's audit shows two writes.

**Cause.** A replay re-fired a call that was not idempotent. In the checkpoint family, replay explicitly re-executes nodes and re-fires API requests (section 4.3). In the replay family the Activity result is reused, but the Activity's own retry lifecycle can land a second call (section 11.2). Either way, the key that was supposed to prevent it was generated inside the retried function, so each attempt carried a different key.

**Guardrail.** The idempotency key is derived from run id and step id and computed before the first attempt; the effect log is consulted before any re-drive; reconciliation detects what the key missed.

### (c) A handoff both agents accept

**Symptom.** Two agents' logs both claim the same step, with overlapping windows. The ledger has one step with two writers, and the second write overwrote the first. Downstream, two writes to the same record.

**Cause.** The sender timed out and retried with a new message identity; the receiver had already accepted and keyed its work on its own local task id; no terminal acknowledgement was recorded on the sender's side (section 6.5).

**Guardrail.** The handoff id is the idempotency key; accept and decline are recorded transitions; at most one active holder is enforced by a compare-and-set on the obligation; the holding is a lease with an expiry that produces a recorded transition.

### (d) A registry listing an agent that no longer exists

**Symptom.** One workflow type fails at dispatch, all day, with a connection error; every other type is healthy. Or worse: the run retries resolution indefinitely and never fails at all.

**Cause.** The inventory and the deployment drifted. The entry was retired from the deployment before it was retired from the registry, or the endpoint moved and the registry was not updated (section 7.3, first row).

**Guardrail.** Resolve and verify at dispatch; treat resolution failure as a typed step failure with a terminal outcome rather than a retry loop; carry the contract digest so a mismatch fails loudly instead of adapting silently.

### (e) A quota nobody enforced

**Symptom.** One run in the ledger with an implausible number of steps, all successful. Then a call from the downstream system's operator asking why their intake queue is filling.

**Cause.** The quota existed as a configured number and was reported on, but never enforced at admission, or the effect budget was never declared at all (section 8.2). Nothing in the platform is in an error state, because nothing exceeded a limit that was being checked.

**Guardrail.** Enforce at admission, attach the quota to the run record so the offending run is identifiable, and define the behaviour at the limit - queue, refuse or degrade. A quota without a defined behaviour at the limit is a dashboard.

### (f) A stuck run discovered by the customer

**Symptom.** A complaint arrives before any internal alert. The run is nine days old; its last transition was eight days ago; its trace is complete and clean.

**Cause.** There was no absence alerting, because the platform's alerts were predicates over emitted events and this failure emitted none (section 10.3). The expected duration for the workflow type was never declared, so nothing could classify the run as late - only as old.

**Guardrail.** Expected duration per workflow type, declared at design time and recorded at admission; expected-completions-minus-observed as a standing count; a stuck-run queue with a named owner.

### (g) An orphan cleaned up while still executing

**Symptom.** Work appears to have been done twice following a routine cleanup. The original worker's log shows it completing normally, some time after a reaper had reclaimed its lease and a second worker had taken over.

**Cause.** The lease expired while the holder was alive but slow, and the reclaim had no fencing: the old holder's writes were still accepted after it had been dispossessed. The record cannot distinguish "lease lapsed" from "holder confirmed dead", so the reaper treated the first as the second.

**Guardrail.** A fencing token incremented on every lease acquisition, carried on every external write, and rejected by the system of record when stale. Reclaim only on confirmed death where confirmation is possible; otherwise treat the run as contested and route it to the stuck-run queue rather than silently re-driving it.

### (h) A run whose record disagrees with the system of record

**Symptom.** An auditor samples ten runs and finds one where the ledger records a refund issued and the payment system has no such refund. The run's terminal state is COMPLETED.

**Cause.** Either the effect was written outside the recorded path - a tool that writes on its own rather than through the coordinator's effect log - or the effect was recorded from the call's return value rather than from the system of record's acknowledgement, so an attempted write was recorded as a completed one.

**Guardrail.** Record effects from the acknowledgement, not from the attempt; keep every write inside the recorded path so that the ledger's effect list is complete by construction; and run reconciliation on a schedule with a disposition for each outcome (section 11.3).

The pattern across all eight is the same and it is the thesis restated operationally: in every case the platform was healthy, the trace was clean, and the record was the only thing that could have told the difference - which is why the record, and not the alerting, is the artifact this layer exists to produce.

## 14. The Cymbal Bank Worked Example

**This example is illustrative and fictional. Cymbal Bank is not a real institution, the platform described is not a case study, and no figure in this section is a measurement.** It exists to show the mechanism of sections 3, 4 and 11 operating together under a constraint that is not technical.

Cymbal Bank runs a multi-agent platform for dispute intake. A customer dispute arrives through a channel, an intake agent extracts the claim and the transaction references, a policy agent decides which dispute category applies, a supervisor agent assigns the investigation to one of several specialist agents by card scheme and merchant type, and a resolution agent writes the outcome to the case-management system and issues any provisional credit. The platform has durable execution on a workflow engine for the deterministic spine and a graph engine for the agentic paths, exactly the pattern platform guide section 2.5 describes. It has distributed tracing across every agent, with spans for each model call and tool call. By the ordinary standards of the discipline it is well built.

Then the supervisor asks a question that the platform cannot answer. For a sample of customer disputes from the last two years, prove that each was handled once, and show every step and every external write.

The platform team's first answer is that the evidence exists. It does not, and the reasons are worth stating precisely because each one is a design decision that seemed reasonable at the time.

**The engine histories expired.** Workflow histories were retained on the engine's operating schedule, set by the platform operations team to control storage. The schedule was shorter than the dispute lifecycle, and much shorter than the audit horizon. Runs from eighteen months ago had no engine history at all.

**The traces were sampled.** Sampling is correct for diagnosis and fatal for proof: for the sampled runs there is a clean tree, and for the unsampled runs there is nothing, and nothing is indistinguishable from a run that never happened. The tracing system was internally consistent throughout.

**The agents' own logs were not evidence.** Each agent logged its reasoning and its tool calls to its own log stream, in its own format, with no run identifier that survived a retry. They were excellent for debugging a single agent and useless for proving a business obligation, because nothing joined them to the dispute.

**No one had recorded the authority.** When a specialist agent issued a provisional credit, the platform could show that the call was authorized in the moment - the policy decision point had returned a positive verdict - but not what grant the step had run under, at what limit, or on whose delegation. The verdict was an event in a log, not a field in a record.

There was one more gap, and it was the one that decided the design. The engine could tell the team that a workflow had closed. It could not tell them that the *dispute* had been resolved, because those are different facts (section 3.5). A workflow that closed by exhaustion of its retry budget was, to the engine, closed.

The decision was to make the run record the primary artifact and demote the engine history to what it is - an implementation record. Five design choices followed, and the first is the one that carries the duplicate guarantee.

**The business-process identifier became the workflow id.** Temporal guarantees "at most one Workflow Execution with a given ID running at any point in time, a constraint that helps to protect against unexpected duplication", and the default conflict policy is Fail, returning "Workflow execution already started" (alternatives being Use Existing and Terminate Existing) ("Workflow Id and Run Id", retrieved 23 November 2026). Using the dispute reference as the workflow id converted a duplicate intake into a *failed start* rather than a second run - and the failed start was recorded as a duplicate attempt against the existing run, so the audit question "was this handled once" had a positive answer with evidence. Run Id, being system-generated and mutable across retry, reset and continue-as-new, was used for nothing logical, exactly as the documentation warns.

**The registry snapshot and the authority grant were written into the run**, not referenced. At dispatch, the supervisor resolved each specialist agent to a version and a contract digest and wrote both into the step, together with the grant's scope, limit, expiry and delegation depth (sections 7.2 and 6.3). This is what allowed the platform to answer the "on whose authority" question without reconstructing it from logs two years later.

**The step ledger became the unit that the case-management system is reconciled against.** A scheduled job compares the ledger's claimed writes - each with its idempotency key derived from run id and step id - against the case-management system's own records, and produces the three outcomes of section 11.3. The first run of that job found both a claimed-and-absent write and a present-and-unclaimed one. The first was a step that had recorded an attempt as a completion; the second was a provisional credit issued by a specialist agent through a retried call whose task token had gone stale.

**The terminal state was separated into business outcomes plus an administrative state.** Resolution, rejection, withdrawal, and parked - the last for runs whose tenant or case was suspended, so that a suspension did not silently create a population of runs with no terminal value (section 8.4).

**Retention became a records class, not a storage setting.** The run ledger is retained on the records schedule for the business process, which is longer than the engine's history retention and independent of it. The checkpoint retention policy for the graph engine was set to be at least as long as the period in which a run could still be recovered (section 4.5).

The costs were real and none of them were technical surprises. In-flight workflows had to be migrated, which meant dual-writing during a transition window. A new retention class had to be negotiated with the records function, which took longer than the engineering. The team lost the ability to deploy a workflow code change while runs were live without versioning - Temporal's determinism constraint makes a deploy to a running workflow a non-deterministic error unless the change is versioned or patched ("Workflow Definition", retrieved 23 November 2026), and the deployment practice had to change to match. And a new operational burden appeared: someone now owns the stuck-run queue, with a rota, a set of three questions to answer per run (section 10.4), and the authority to declare a run failed when it cannot be recovered.

That last cost is the one worth dwelling on, because it is the one that reveals what the platform actually bought. Before the change, a stuck run was invisible and therefore cost nothing operationally - it simply sat in a database and was discovered, if at all, by a customer. After the change, a stuck run is a work item with an owner and a clock. The platform did not become cheaper to run; it became *legible*, and legibility has a staffing cost that no architecture diagram shows.

The example ends where the thesis does. Cymbal Bank did not need better orchestration logic. Its coordinators worked. What it needed was for the layer to own the record of what happened, because every other service in the platform could fail in a way that somebody would notice, and this one could fail by leaving a dispute half-done with a clean trace and a closed workflow. The mechanism is the illustration; the institution is fiction.

## 15. The Claims Audit and What Could Not Be Verified

Every claim that carries weight in this guide appears below with its source, the date it was read, the version where one is exposed, its kind, a verdict, and a note on the quality of the evidence. Verdicts are **Verified at source** (the sentence is on the page cited), **Flagged** (true as read, with a caveat stated in the note), or **Rejected** (argued and discarded). No claim in this guide was written from memory.

### 15.1 The audit

| # | Claim | Source | Date | Version | Kind | Verdict | Quality note |
|---|---|---|---|---|---|---|---|
| 1 | Platform declares eight services in 1.1 | platform guide, section 1.1 table, line 43 | 23 Nov 2026 | n/a (in-repo doc) | my synthesis (count) | ✅ Verified at source | Counted from the table, not inferred |
| 2 | Section 4 develops seven, with no 4.8 | platform guide, lines 351-503 | 23 Nov 2026 | n/a (in-repo doc) | my synthesis (count) | ✅ Verified at source | Counted from headings 4.1 to 4.7 |
| 3 | Orchestration appears in six kinds of place, lines 43, 113, 152, 245-261, 679, 790/824/831 | platform guide | 23 Nov 2026 | n/a (in-repo doc) | my synthesis | ✅ Verified at source | Each line read individually |
| 4 | Line 824 is a homonym: Kubernetes as "container-orchestration substrate" | platform guide, line 824 | 23 Nov 2026 | n/a (in-repo doc) | my synthesis | ✅ Verified at source | Not a reference to the layer |
| 5 | Checkpointer saves a snapshot at each super-step, organised into threads | LangGraph "Checkpointers" | 23 Nov 2026 | not exposed on page | vendor documentation | ✅ Verified at source | Quoted sentence; version flagged |
| 6 | thread_id is the primary key; without it no save and no resume after interrupt | LangGraph "Checkpointers" | 23 Nov 2026 | not exposed on page | vendor documentation | ✅ Verified at source | Version flagged |
| 7 | Pending writes from nodes that completed at a failed super-step are stored and reused | LangGraph "Checkpointers" | 23 Nov 2026 | not exposed on page | vendor documentation | ✅ Verified at source | The one mechanism that makes partial super-steps safe |
| 8 | Checkpointing enables HITL, memory, time travel, fault tolerance | LangGraph "Checkpointers" | 23 Nov 2026 | not exposed on page | vendor documentation | ✅ Verified at source | Four capabilities on one mechanism; see 4.1 |
| 9 | Checkpointer is per-thread graph state; store is cross-thread key-value | LangGraph "Checkpointers" | 23 Nov 2026 | not exposed on page | vendor documentation | ✅ Verified at source | The distinction 4.1 relies on |
| 10 | Replay re-executes nodes; LLM calls, API requests and interrupts fire again and may differ | LangGraph "Use time-travel" | 23 Nov 2026 | not exposed on page | vendor documentation | ✅ Verified at source | Sharpest published statement that replay is not reproducibility |
| 11 | A subgraph without its own checkpointer is one super-step; no time travel between its nodes | LangGraph "Use time-travel" | 23 Nov 2026 | not exposed on page | vendor documentation | ✅ Verified at source | Basis of 4.5's granularity argument |
| 12 | `checkpointer=True` on the subgraph creates per-step checkpoints inside it | LangGraph "Use time-travel" | 23 Nov 2026 | not exposed on page | vendor documentation | ✅ Verified at source | |
| 13 | `update_state` does not roll back; it branches a new checkpoint | LangGraph "Use time-travel" | 23 Nov 2026 | not exposed on page | vendor documentation | ✅ Verified at source | Correction is append, not erase |
| 14 | MemorySaver holds checkpoints in RAM and loses them on restart; Postgres and SQLite persist | LangGraph "Persistence" | 23 Nov 2026 | not exposed on page | vendor documentation | ✅ Verified at source | |
| 15 | Checkpoints accumulate and require a retention policy | LangGraph "Persistence" | 23 Nov 2026 | not exposed on page | vendor documentation | ✅ Verified at source | Basis of 4.5's records-management point |
| 16 | Durability is "the absence of an imposed time limit"; execution completes "effectively once" | Temporal "Workflow Execution" | 23 Nov 2026 | not exposed on page | vendor documentation | ✅ Verified at source | A definition, not a performance claim |
| 17 | Status is Open (Running, Paused) or Closed (Cancelled, Completed, Continued-As-New, Failed, Terminated, Timed Out) | Temporal "Workflow Execution" | 23 Nov 2026 | not exposed on page | vendor documentation | ✅ Verified at source | |
| 18 | Each state transition is recorded in a persistence store | Temporal "Workflow Execution" | 23 Nov 2026 | not exposed on page | vendor documentation | ✅ Verified at source | |
| 19 | A worker cache with sticky execution avoids replay from scratch; eviction forces replay | Temporal "Workflow Execution" | 23 Nov 2026 | not exposed on page | vendor documentation | ✅ Verified at source | |
| 20 | Temporal restores no snapshot; it re-executes code from the beginning against the history | Temporal "Workflows" | 23 Nov 2026 | not exposed on page | vendor documentation | ✅ Verified at source | Direct contradiction of the snapshot model |
| 21 | The workflow must make the same decisions from the same history; must not read unrecorded values (Date.now, randomness, an un-Activity-ed network call) | Temporal "Workflows" | 23 Nov 2026 | not exposed on page | vendor documentation | ✅ Verified at source | Three examples are the vendor's own |
| 22 | An Activity runs once, its result is recorded, and is reused rather than recomputed on replay | Temporal "Workflows" | 23 Nov 2026 | not exposed on page | vendor documentation | ✅ Verified at source | The contrast with claim 10 |
| 23 | Workflow code must issue the same API calls in the same sequence given the same input | Temporal "Workflow Definition" | 23 Nov 2026 | not exposed on page | vendor documentation | ✅ Verified at source | Quoted constraint |
| 24 | Non-deterministic operations belong in Activities, which execute outside the replay path | Temporal "Workflow Definition" | 23 Nov 2026 | not exposed on page | vendor documentation | ✅ Verified at source | Basis of 4.3's general rule |
| 25 | Commands are compared position by position; mismatch is a non-deterministic error; two causes named | Temporal "Workflow Definition" | 23 Nov 2026 | not exposed on page | vendor documentation | ✅ Verified at source | Code change and intrinsic logic |
| 26 | At most one Workflow Execution per Workflow Id; conflict policy default is Fail | Temporal "Workflow Id and Run Id" | 23 Nov 2026 | not exposed on page | vendor documentation | ✅ Verified at source | The duplicate guard in section 14 |
| 27 | Run Id is system-generated and mutable across retry, reset, continue-as-new; not for logical choices | Temporal "Workflow Id and Run Id" | 23 Nov 2026 | not exposed on page | vendor documentation | ✅ Verified at source | |
| 28 | Workflow Ids are stored in plain text and must not contain PII | Temporal "Workflow Id and Run Id" | 23 Nov 2026 | not exposed on page | vendor documentation | ✅ Verified at source | Constrains section 14's choice of business id |
| 29 | A Task Token can go stale across retries; Activity Id plus Workflow Id is the alternative | Temporal "Activity Execution" | 23 Nov 2026 | not exposed on page | vendor documentation | ✅ Verified at source | The concrete duplicate mechanism in 11.3 |
| 30 | Opaque Execution: collaboration without sharing internal thoughts, plans or tool implementations | A2A specification, guiding principles | 23 Nov 2026 | 1.0.0 | standard | ✅ Verified at source | Basis of attested versus declared steps |
| 31 | Agent Card is a JSON metadata document published by the server; context identifier groups related tasks | A2A specification, Agent Card | 23 Nov 2026 | 1.0.0 | standard | ✅ Verified at source | Self-description, not attestation |
| 32 | Task is the fundamental unit of work, stateful, with the nine listed states and terminal semantics | A2A specification, Task | 23 Nov 2026 | 1.0.0 | standard | ✅ Verified at source | Terminal message raises UnsupportedOperationError |
| 33 | SendMessage returns a Task or a Message; blocking waits for terminal or interrupted, non-blocking returns and the caller polls | A2A specification, message send | 23 Nov 2026 | 1.0.0 | standard | ✅ Verified at source | |
| 34 | Streaming closes when a task reaches a terminal state | A2A specification | 23 Nov 2026 | 1.0.0 | standard | ✅ Verified at source | Supports the terminal-declaration argument in 12.2 |
| 35 | A2A is agent-to-agent; MCP is agent-to-tool and resource; they are complementary | A2A specification, Appendix B | 23 Nov 2026 | 1.0.0 | standard | ✅ Verified at source | Basis of 12.3's modelling rule |
| 36 | Memory retrieval misses are quiet, and a miss is indistinguishable from nothing to recall | none | - | - | my synthesis | ⚠ Flagged | Reasoned from the shape of a similarity interface, not measured |
| 37 | Observability has a silent mode: an unemitted span is invisible, and absence is detectable only against an assertion | none | - | - | my synthesis | ⚠ Flagged | Argument, not a sourced finding |
| 38 | The amended thesis in 2.4 | none | - | - | my synthesis | ⚠ Flagged | Replaces the headline form where they differ |
| 39 | Durability does not provide exactly-once external effects | Temporal "Activity Execution" and LangGraph "Use time-travel", reasoned | 23 Nov 2026 | - | my synthesis | ⚠ Flagged | An argument from two mechanisms, not a vendor guarantee |
| 40 | The handoff contract in 6.3 | none | - | - | my synthesis | ⚠ Flagged | No standard was found; this is this guide's design |
| 41 | Replay is not reproducibility | LangGraph "Use time-travel", generalised | 23 Nov 2026 | - | my synthesis | ✅ Verified at source | The generalisation rests on a direct quote |
| 42 | A trace store can serve as the run record | agentops 3.1, read against 3.3 | 23 Nov 2026 | - | my synthesis | ✗ Rejected | Traces are sampled, can be dropped by an exporter, and end with the process; see 3.3 |

### 15.2 What could not be verified

**1. That orchestration fails more often than other services.** No source read in this pass establishes prevalence, and none was sought after the first search. The thesis is an argument from *failure signature* - what a failure looks like and what it leaves behind - not from frequency. A reader who wants to reject this guide can reject the thesis on those grounds without touching the mechanism sections, which stand on their own.

**2. Deterministic replay of a complete multi-agent run with model outputs pinned.** No vendor documentation read in this pass documents it. Section 4.4's fifth condition is therefore stated as unsatisfiable by any documented mechanism rather than as a solved problem, and the Cymbal example does not claim it.

**3. Any quantification of duplicate effects.** No source read gives a rate, and this guide deliberately prints none. The mechanism is documented; the incidence is not.

**4. Versions for the LangGraph and Temporal pages.** Neither family of pages exposes a release version string - the Temporal pages carry a documentation tag reading "current" and nothing more. The version column therefore reads "not exposed on page", and every fact from those sources is dated by retrieval only. A reader checking a claim later may find the page changed; the quote may not survive a documentation rewrite.

**5. A standard handoff contract.** No specification or vendor document read in this pass defines the three-transfer contract of section 6. The nearest artefact is A2A's task lifecycle, which supplies a state vocabulary and explicitly leaves ownership to the implementer. Section 6 is this guide's design and is marked as such here and in 6.1.

**6. Whether A2A implementations honour terminal-state semantics in practice.** The specification says a message to a terminal task raises `UnsupportedOperationError`. Whether a given deployment enforces that, rather than logging it, was not verified, and it is the kind of gap between specification and implementation that section 12's attested-versus-declared distinction is designed to survive.

**7. That the connection between checkpoint granularity and duplicate risk is more than an argument.** Section 4.5's claim that granularity is the unit of recovery is verified from the mechanism; the further claim that it is therefore the unit of duplicate risk is reasoned and unmeasured.

**8. The Cymbal Bank example.** Fiction, declared in section 14's first line and repeated here so that no reader arrives at the audit without having seen it.

Search was used only to locate pages. Every sentence quoted in this guide was read on the source page rather than taken from a search result snippet, because snippets strip the surrounding qualification that determines whether a claim is true.

### 15.3 Why no benchmark and no ranking appear

**A benchmark would test the product, not the mechanism.** The questions in this guide are of the form "does this record survive the engine's retirement" and "does this replay duplicate an effect". A measurement of one platform's behaviour on a workload answers neither, and would be quoted for years after both the workload and the platform changed.

**A matrix would answer a procurement question, and that question is owned elsewhere.** Whether one engine is a better buy than another is [ai_agent_platform_selection_guide.md](ai_agent_platform_selection_guide.md) sections 3 to 5, which does it properly with criteria, weights and a decision matrix. A ranking smuggled into an architecture guide would be a worse version of that work, in a place where the reader is not equipped to check it.

**Prevalence is not truth.** Even a correct measurement of how often each service fails would not settle the thesis, because the thesis is about consequence rather than frequency. A service that fails rarely and silently is a different architectural problem from one that fails often and loudly, and collapsing the two into a number would lose the distinction this guide exists to draw.

## 16. Glossary, Cross-References, and the Closing

### 16.1 Glossary

| Term | Definition in this guide | Often confused with |
|---|---|---|
| Run | One instance of a business process that must reach exactly one terminal state | A request, a session, or a trace; only a run carries an obligation |
| Step | The unit at which the record advances: one coordinator decision plus the effects it authorises | A model call, or an agent invocation; a step may contain several of either |
| Handoff | A state transition between agents transferring context, authority and obligation | A message; a message can be duplicated, while an accountability transfer must happen once |
| Checkpoint | A durable snapshot of execution state at a step boundary, scoped to an identity | A trace (evidence, not state) or a store (cross-thread application data) |
| Replay | Reconstruction of progress by re-executing recorded work against recorded results | Reproduction; in one engine family the model is called again and may answer differently |
| Determinism | The property that the same code, given the same history, issues the same calls in the same order | Reproducibility of outputs, which replay does not guarantee |
| Registry | The mapping from a name to an endpoint, version and contract, resolved at dispatch | A discovery card, which is the provider's self-description rather than the owner's attestation |
| Tenancy | The partition within which runs compete for capacity, authority and budget | Runtime isolation; a sandbox isolates one execution, tenancy governs relations between executions |
| Quota | An enforced limit at admission, with a defined behaviour at the limit | A reported metric; an unenforced quota is a dashboard |
| Reconciliation | A scheduled comparison of the ledger's claimed effects against the systems of record | Recovery; recovery advances a run, reconciliation proves what it did |

### 16.2 Cross-references, by guide and section

- [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) - 1.1 the eight services (and this guide's asymmetry count); 1.2 platform, framework, application; 2.5 the layer plus its engine comparison; 2.6 the agent registry inventory; 3.3 A2A; 4.2 runtime isolation; 4.5 observability; 4.7 governance; 8 worked example.
- [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md) - 3 dimensions, 4 static versus adaptive versus hybrid hierarchy, 5 control patterns, 6 multi-backend, 7 model routing, 8 comparison, 9 designing. Topology choice lives here.
- [hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md) - 3 hierarchy types, 6 core patterns, 8 designing, 9 communication patterns, 13 comparison. Hierarchy as topology lives here.
- [multi_agent_banking_guide.md](multi_agent_banking_guide.md) - 3 orchestration patterns, 5 guardrails, 6 evaluation, 9 deployment, 11 production standards, 12 worked example, 14 claims audit. The banking deployment lives here.
- [multi_agent_debugging_guide.md](multi_agent_debugging_guide.md) - 3 taxonomy, 4 trace tree, 5 attribution, 6 reproduction, 7 instrumentation delta, 8 diagnosis patterns, 12 Cymbal example. Diagnosing a failed run lives here.
- [agentops_guide.md](agentops_guide.md) - 3.1-3.2 the single-run trace and span hierarchy, 5 monitoring and alerting, 6 cost operations, 7 tooling. Operating emitted telemetry lives here.
- [agent_skills_vs_multi_agent_systems_guide.md](agent_skills_vs_multi_agent_systems_guide.md) - 3 seven properties, 4 the replace claim, 5 the compile claim, 8 enterprise consequence. Whether to use agents at all lives here.
- [ai_agent_platform_selection_guide.md](ai_agent_platform_selection_guide.md) - 3 criteria, 4 scoring, 5 decision matrix, 7 pitfalls. Whether to buy a platform lives here.

### 16.3 The closing

The place of this layer was already settled. The platform guide declares it as one of eight services, gives it a subsection in its architecture chapter, a row in its component table and a line in its diagram, and there the specification stops. That is not a criticism; it is a division of labour, and this guide took the other half. **Eight services declared, seven developed, one absent** - and the one absent is the one whose failure nobody sees.

The asymmetry is the whole argument. A gateway that refuses returns a refusal; a runtime that dies leaves an exit status; a tool that fails raises a typed error; security and governance deny in writing. Two siblings do fail quietly, and this guide conceded both rather than pretending otherwise - but a memory miss degrades one step's input inside a run that still finishes, and an observability gap removes evidence about a run that still completed. Only orchestration's quiet failure changes whether the business process completed at all, and takes with it the record that would have said so.

What follows from that is architectural rather than operational. The run, not the agent and not the trace, must be the unit of record, and it must carry the identity, the step ledger, the authority grants, the effects with their keys, the terminal state and the version set. Durability must be chosen by mechanism rather than by name, because snapshot-checkpointing and event-history replay fail in opposite directions and one of them calls the model again on replay. Handoffs must carry obligation as well as context and authority, with a single holder enforced by a compare-and-set rather than by good behaviour.

Around that core, the layer owes six more things: resolution at dispatch with the snapshot written into the run; quotas enforced at admission with a defined behaviour at the limit; an attribution key that makes an unattributable action impossible rather than merely expensive; alerts that are assertions about expected counts, because this failure emits nothing to match a pattern against; a recovery tree that chooses between resume, re-drive, compensate and declare failed on the evidence in the record; and reconciliation as a scheduled platform function, because it is the only control that finds a write that was claimed and never made.

The cost is not small and it is honest. Someone has to own the stuck-run queue. A new retention class has to be negotiated with the records function. Deploying to a running workflow stops being free. These are the prices of legibility, and a platform that declines to pay them has not avoided the problem - it has chosen to discover it from a customer, or an auditor, or a supervisor, some months later.

**every other platform service fails loudly; orchestration fails by leaving a run half-done - so its architecture is really a decision about who owns the record of what happened.**
