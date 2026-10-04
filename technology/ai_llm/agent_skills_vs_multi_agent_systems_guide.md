# Agent Skills vs Multi-Agent Systems — Replace, Compile, or Neither

> **A 2026 deep-dive on one question, asked precisely: can reusable procedural guidance — the versioned instruction artifact known as an *agent skill* — stand in for a multi-agent topology, or be the thing a topology is compiled from?** The question sounds like a single decision. It is two decisions wearing the same coat. *Replace* asks whether a skill set can do what a multi-agent system does, so that the topology becomes unnecessary. *Compile* asks whether a topology can be **emitted from** a procedural specification, so that the topology becomes a build artifact rather than a design. These claims have different evidence, different failure modes, and different answers. This guide gives each its own verdict, decomposes a multi-agent system into the **seven properties** it actually buys, and works out which of those a skill can absorb, which it can only partly absorb, and which are structural. The honest conclusion is a split verdict, not a winner.

> **Series context and boundary (read this first).** This guide owns exactly one thing: **the analysis of whether a skill artifact can SUBSTITUTE FOR, or serve as the COMPILATION TARGET OF, a multi-agent topology.** It does not re-derive the material owned by sibling guides in this repository, and it cross-references them by name throughout:
>
> - **The multi-agent topologies themselves** — hierarchy types, control patterns, multi-backend and model-routing, and the banking guardrails over them — are owned by [hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md) (708 lines), [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md) (854 lines), and [multi_agent_banking_guide.md](multi_agent_banking_guide.md) (802 lines). This guide uses topologies in outline only.
> - **The harness and the scaffold** — the agent loop, memory tiers, harness-generation and self-refinement work (including Meta-Harness / Self-Harness) — are owned by [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) (777 lines) and [agent_scaffolding_guide.md](agent_scaffolding_guide.md) (934 lines).
> - **Progressive disclosure**, the instruction-level descendant of which is the skills pattern, is owned by [mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md) (900 lines) — its section 6 is "Instruction-Level Disclosure — The Skills Pattern".
> - **Context discipline** — budgets, compression, positional effects, retrieval-fed assembly — is owned by [context_engineering_guide.md](context_engineering_guide.md) (785 lines).
> - **The engineering discipline and its lifecycle** are owned by [agentic_engineering_guide.md](agentic_engineering_guide.md) (898 lines).
> - **Banking controls** — the AI risk register, agent governance, and the agentic-engineering control cycle — are owned by [ai_risk_register_guide.md](../banking/ai_risk_register_guide.md) and the repository's agent-governance and agentic-engineering guides. This guide states only the *consequence for this specific decision* and does not re-derive their frameworks.
> - **The vendor product-feature sense of "agent skills"** is documented for one platform by [servicenow_agentic_platform_guide.md](../servicenow_agentic_platform_guide.md); this guide cross-references it rather than re-explaining it.
>
> The question is unowned in this repository, and this was verified rather than assumed: across the tree, `"skills replace"` and `"instead of multi-agent"` return **0 files**, `"verification agent"` returns **0**, `"context isolation"` returns **1**, and `"agent skills"` appears in **9 files only as passing mentions**. The components exist; the question was never asked here. That absence is why this guide exists.

> **Verification policy.** Every proposition in this guide is **dated and attributed**: who published it, when, and of what kind. The kinds used are **(paper)** for a peer-reviewed publication, **(preprint)** for an unreviewed arXiv or similar manuscript, and **(vendor post)** for a vendor's engineering blog, product announcement, or documentation. Where a source is a vendor's own claim about its own product — most notably a self-evaluation percentage — it is labelled as such in the sentence that carries it and never presented as an independent result. Anything that could not be traced to a primary source is marked **unverified** and is listed in §15 rather than asserted. arXiv identifiers were resolved against the arXiv API over HTTPS with a browser user-agent; where a page could not be extracted, the claim is recorded as unverified rather than sourced from memory. **No percentage, date, identifier, count, or feature-availability claim in this guide is written from memory.**

**Jack Liu Shurui, Solution Architect**

*Baseline October 2026 (written 4 October 2026). Source dates are as published; re-verify every figure and availability status against the primary sources in §15 before building a policy on it.*

---

## Contents

1. [Overview — The Artifact Homonym, Decoded](#1-overview--the-artifact-homonym-decoded)
2. [What a Skill Is, as an Artifact](#2-what-a-skill-is-as-an-artifact)
3. [What a Multi-Agent System Is, for This Question — The Seven Properties](#3-what-a-multi-agent-system-is-for-this-question--the-seven-properties)
4. [The Replace Claim, Property by Property](#4-the-replace-claim-property-by-property)
5. [The Compile Claim — Emitting a Topology from a Procedure](#5-the-compile-claim--emitting-a-topology-from-a-procedure)
6. [The Evidence Base — Four Literatures](#6-the-evidence-base--four-literatures)
7. [The Mechanisms That Decide It](#7-the-mechanisms-that-decide-it)
8. [The Enterprise and Banking Consequence](#8-the-enterprise-and-banking-consequence)
9. [The Practical State of the Art](#9-the-practical-state-of-the-art)
10. [The Open Questions](#10-the-open-questions)
11. [Documentation, Versioning, and Evaluation of Skills as a Catalogue](#11-documentation-versioning-and-evaluation-of-skills-as-a-catalogue)
12. [The Cymbal Bank Worked Example](#12-the-cymbal-bank-worked-example)
13. [The Anti-Patterns](#13-the-anti-patterns)
14. [The Claims Audit](#14-the-claims-audit)
15. [What Could Not Be Verified](#15-what-could-not-be-verified)

---

## 1. Overview — The Artifact Homonym, Decoded

### 1.1 The question contains two claims, and blurring them destroys the analysis

The sentence *"can skills replace or compile multi-agent systems?"* compresses two claims that must be kept apart from the first paragraph onward:

- **REPLACE.** A set of skills can *do what a multi-agent topology does*, so the topology is unnecessary. The unit of comparison is **capability**: does the skill artifact, loaded by a single agent, deliver the functional properties that motivated building several agents in the first place?
- **COMPILE.** A multi-agent topology can be *emitted from a procedural specification*, so the topology is a build artifact rather than a design. The unit of comparison is **provenance**: is the graph a thing a human draws, or a thing a compiler produces from a procedure that already says what has to happen?

These are not the same claim, and the evidence for one is not evidence for the other. The strongest existing result behind the *compile* claim — automated search over agent code (ADAS) and edge optimization of agent graphs (GPTSwarm) — says almost nothing about whether a *skill* can replace a topology. The strongest argument against blanket *replace* — that independent verification and context isolation are structural — says nothing about whether a topology can be generated. **This guide gives each claim its own verdict (§4 and §5), and does not offer one blanket answer.**

A useful way to hold the two apart: *replace* is a statement about a runtime with **fewer boxes**; *compile* is a statement about a runtime with **generated boxes**. The first is a simplification argument; the second is a codegen argument. A team can reasonably accept one and reject the other.

### 1.2 The artifact homonym — three things called "agent skills"

"Agent skills" names at least three different things, and almost every confused conversation about this topic is a conversation in which two participants mean different ones. Separating them is the first job of this guide.

**(a) The reusable procedural artifact.** A versioned, human-readable instruction bundle — the `SKILL.md`-style directory of instructions, optional reference files, and optional scripts that a coding or general agent discovers and loads on demand. This is the sense the user's question means, and it is the sense this guide addresses. Its defining property is that it is **guidance**: text the model reads and follows, not code the machine executes on its behalf (though it may *bundle* code the model chooses to run).

**(b) A platform product feature of the same name.** Several vendors now ship a feature literally called "skills" inside their agentic platforms. One of them — the ServiceNow surface, where the skill sits alongside the agent layer and the MCP path — is already documented in this repository by [servicenow_agentic_platform_guide.md](../servicenow_agentic_platform_guide.md). That guide owns the product-feature treatment; this guide cites the phenomenon only to disambiguate the word and does not re-explain any vendor's implementation. The important distinction: a *product feature* named "skills" is generally a governed artifact type inside one platform, whereas the **open format** in sense (a) is portable across clients — the Agent Skills format was published as an open standard on 18 December 2025 (vendor standard, agentskills.io).

**(c) Human professional skills.** The organisational and labour literature on what people who work with AI need to be able to do — a different literature, owned elsewhere in the repository. It has nothing to do with the artifact that an agent loads, and it is out of scope here except as a warning: search for "skills" and you will drown in sense (c) before you find sense (a).

**This guide addresses sense (a) exclusively** — the versioned procedural artifact — and treats sense (b) as a separately-governed deployment surface to be cross-referenced, and sense (c) as out of scope.

### 1.3 The decoder table

The words that carry the argument, decoded precisely. Several of them are used loosely in the field; the definitions here are the ones the rest of the guide relies on.

| Term | What it means in this guide | What it is not |
|---|---|---|
| **The skill** | A versioned procedural artifact: metadata + an instruction body + optional references and scripts, loaded on demand by an agent. Sense (a). | Not a tool, not an agent, not a product SKU. |
| **The procedural artifact** | Any human-authored description of *how to carry out a task* — a runbook, a playbook, a template, a skill. The generic category that includes the skill. | Not a capability by itself; it changes behaviour, it does not take actions. |
| **The topology** | The graph of agents — how many, in what arrangement, communicating how (orchestrator-worker, hierarchical, peer debate, pipeline). | Not the prompts, not the tools, not the model choice. The *arrangement*. |
| **The supervisor / orchestrator** | The agent (or the code) that decomposes a task and dispatches work to others; the node that holds the plan. | Not always an LLM; can be a deterministic controller. |
| **The sub-agent** | A separate agent invocation with its own context window, dispatched by a supervisor. | Not a skill, not a tool call, not a chain step in one context. |
| **Context isolation** | The property that a sub-agent's working memory is *separate* from the supervisor's — that its intermediate reasoning does not have to re-enter the supervisor's window. | Not merely "a narrower prompt"; the isolation is the point. |
| **Compilation** | Producing an executable artifact (here, a topology) from a higher-level specification by a mechanical transformation. | Not "configuration" and not "parameter tuning"; see §5 for the boundary. |
| **Subsumption** | The relation where a skill set delivers a property that a topology was built to provide, making the topology unnecessary *for that property*. | Not a global equivalence between the two; subsumption is per-property. |
| **Progressive disclosure** | Loading the surface on demand rather than up front — for skills, the three-tier metadata → body → resources model. Mechanism owned by the progressive-disclosure guide. | Not the same as context isolation; it bounds *instruction* tokens, not *reasoning* state. |
| **Hand-off** | The transfer of task, state, and results from one agent to another across a context boundary. | Not a function call that returns a value inside one context. |
| **Absorbed / structural** | The two poles of §4's verdict per property: *absorbed* means a skill set can provide it; *structural* means it follows from the arrangement, not the instructions. | Not good versus bad; structural properties are not defects. |

### 1.4 The thesis

**A skill set can absorb a substantial share of why teams build multi-agent systems, and it cannot absorb all of it; and a topology is only partially compilable from a procedure, because what the current generation of automation demonstrates is *search for a topology against a metric*, not *translation of a procedure into a topology*.** Stated as a split verdict in advance, to be earned property by property in §4 and §5:

- **Replace — mostly partial.** Four of the seven properties (§4) are *absorbed* or *partially absorbed* by a well-run skill library: specialisation and prompt hygiene, shared state and hand-off (partly), per-role model and tool choice (partly, and only where the harness supports it), and parallelism/latency (only when the skill bundles code the model runs itself). Two are **structural**: context isolation (i) and independent verification/adversarial separation (iv). Fault isolation (v) is structural but its *consequences* are often manageable. The verdict is a split, weighted against blanket replacement by exactly the two properties that are hardest to fake.
- **Compile — mostly unproven, with real prior art.** Automated search can generate agent designs (ADAS, preprint, 2024) and optimize agent-graph connectivity (GPTSwarm, preprint, 2024); workflow optimization has been reformulated as search over code-represented workflows (AFlow, preprint, 2024); declarative pipelines can be compiled (DSPy, preprint, 2023) — but DSPy compiles **prompts and parameters**, not topologies. What has been demonstrated is *search*, *optimization*, and *parameterisation*. What has **not** been demonstrated is the clean thing the compile claim asserts: take a complete human procedure and emit a correct multi-agent topology from it. That gap is the finding.

### 1.5 The boundary declared by name

So that the boundary is enforceable in review, stated explicitly: this guide re-derives none of the following and points to its owner instead. **Topology families and control patterns** → [hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md) and [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md). **Banking-specific multi-agent design and guardrails** → [multi_agent_banking_guide.md](multi_agent_banking_guide.md). **Harness loop, memory, and harness/self-harness generation** → [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) and [agent_scaffolding_guide.md](agent_scaffolding_guide.md). **Progressive disclosure including the skills pattern** → [mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md). **Context economics** → [context_engineering_guide.md](context_engineering_guide.md). **The discipline and lifecycle** → [agentic_engineering_guide.md](agentic_engineering_guide.md). **Skill and agent versioning** → [agent_versioning_guide.md](agent_versioning_guide.md). **Evaluation frameworks** → [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md). **Banking controls and risk register** → [ai_risk_register_guide.md](../banking/ai_risk_register_guide.md) and the repository's agent-governance guides. **The vendor product-feature sense of "skills"** → [servicenow_agentic_platform_guide.md](../servicenow_agentic_platform_guide.md).

The one thing this guide owns is the decision analysis that sits between them: *given everything those guides establish, which separations between agents are structural, and which were only ever about context?*

---

## 2. What a Skill Is, as an Artifact

This section is deliberately short and structural. The *mechanism* of skill loading — progressive disclosure, the token budget of a descriptor, trigger-rate evaluation — is owned by [mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md), §6. What belongs here is the shape of the artifact, because the compile and replace arguments both depend on what kind of thing a skill *is*.

### 2.1 Structure

A skill, in sense (a), is a **directory** whose entry point is a markdown file — `SKILL.md` — beginning with YAML frontmatter that carries at minimum a `name` and a `description`, followed by an instruction body and, optionally, bundled reference files, templates, and executable scripts (vendor post, Anthropic engineering, "Equipping agents for the real world with Agent Skills", 16 October 2025; vendor standard, agentskills.io, published as an open standard 18 December 2025). The engineering post describes a skill as "organized folders of instructions, scripts, and resources that agents can discover and load dynamically" and draws the analogy of an onboarding guide for a new hire.

The structural facts that matter for this guide:

- **The unit of authoring is a document, not a program.** A skill is text a human writes and reviews. It composes with the filesystem and with version control in the ordinary way.
- **It is self-describing.** The frontmatter `name` and `description` are the only part guaranteed to be in context; they double as the retrieval key. The format specification caps `name` and `description` (per the platform documentation, `description` at 1,024 characters) precisely because the descriptor is a retrieval key, not prose.
- **It may bundle code.** The engineering post is explicit that a skill can carry a pre-written script that "reads a PDF and extracts all form fields" and that Claude "can run this script without loading either the script or the PDF into context" — and that because the code is deterministic, "this workflow is consistent and repeatable." **This is the single most important structural fact in the replace argument**: a skill is not only guidance, it is a vehicle for *deterministic execution*, and determinism is one of the things a topology is otherwise built to buy.
- **It is portable.** The open-standard publication (18 December 2025) means the format is not one vendor's private shape; the standard's own client showcase lists a broad set of agent clients and IDEs that read it (vendor standard, agentskills.io).

### 2.2 The trigger and loading model

A skill loads in three stages the standard names **discovery, activation, and execution** (vendor standard, agentskills.io): at startup only the name and description of each installed skill are loaded; when a task matches the description the full `SKILL.md` body is read into context; the agent then follows the instructions and may execute bundled code or read referenced files. The engineering post gives the same model as the three levels of **progressive disclosure** — metadata always present, body on trigger, resources only as needed — and states that because the agent has a filesystem and code execution, "the amount of context that can be bundled into a skill is effectively unbounded" (vendor post, 16 October 2025).

Two consequences bear directly on this guide:

1. **The trigger is a retrieval decision made by the model, and it can fail silently.** If the descriptor under-specifies, the skill never loads and the agent simply does the task *without the procedure* — no error is raised. This is the skills pattern's characteristic failure and it is owned by the progressive-disclosure guide; here it matters because it means *a skill that is not triggered is indistinguishable, at the task level, from a topology that was never built*.
2. **Loading is additive to one context, not a new context.** When a skill activates, its body enters the *same* window as everything else. There is no new context window, no separate memory, no isolation. Everything the replace analysis concludes about context isolation (§4.1) and verification (§4.4) flows from this single fact.

### 2.3 Versioning

Because a skill is a document, its versioning story is a document-control story. The platform announcement notes an API `/v1/skills` endpoint "for programmatic control over custom skill versioning and management" and skill versions createable, viewable, and upgradeable through a console (vendor post, Anthropic, "Introducing Agent Skills", 16 October 2025). The open format is naturally version-controlled in the same repository as the code it supports. The repository's own treatment of versioning — for skills, agents, and the artifacts around them — is owned by [agent_versioning_guide.md](agent_versioning_guide.md); this guide states only the consequence: **a skill has a review diff and a version number, which is exactly the shape a change-control process already knows how to handle.** That is a genuine advantage over a topology (see §8).

### 2.4 The relationship to tools and to the harness

A skill is neither a tool nor a harness feature; it sits between them.

- It is **not a tool**: a tool is a callable function with a schema, invoked to take an action; a skill is an instruction bundle that tells the model *how to work*. The progressive-disclosure guide states the contrast crisply — a hidden tool can take an action, a hidden instruction changes behaviour.
- It **depends on the harness**: the trigger is a model decision made inside an agent loop that has a filesystem and, for script execution, a code-execution surface. Without a harness that can read files and run code, a skill degrades to a paragraph of prose. The platform announcement states plainly that at the API level "Skills require the Code Execution Tool beta, which provides the secure environment they need to run" (vendor post, 16 October 2025).
- It **references tools, rather than replacing them**: the skills documentation requires fully qualified MCP tool names inside a skill so the references resolve (documented behaviour, per the progressive-disclosure guide's reading of the skills docs).

The relationship that matters most for the compile claim is this: **a skill is the natural *source language* for a compiled topology if one exists, because it already encodes "what has to happen, in what order, with what checks."** §5 examines whether that source language has ever actually been compiled into a graph.

### 2.5 Guidance, not execution

The decisive structural statement, and the one the rest of the guide keeps returning to:

**A skill is guidance. It changes what the model does; it does not, by itself, run anything.** Its bundled scripts are execution, but the *decision to run them* is a model decision inside the same context that is following the guidance. A skill cannot, on its own, create a second context window, dispatch a parallel worker, hold a plan in memory separate from the work, or grade the work with a model that has not seen the grader's reasoning. Those are properties of an *arrangement of agents*, not of a document. Whether they can be *obtained another way* — by code the skill bundles, by the harness, by the model's own competence — is precisely what §4 tests, property by property.

---

## 3. What a Multi-Agent System Is, for This Question — The Seven Properties

### 3.1 The topologies, in outline only

This guide does not re-derive the taxonomy of multi-agent arrangements; it is owned by [hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md) and [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md). What is needed here is only the outline, so that "topology" has a referent:

- **Orchestrator–worker.** A supervisor decomposes a task, dispatches sub-agents, and synthesises. Anthropic's production research system is the canonical documented instance: an orchestration pattern where "a lead agent coordinates the process while delegating to specialized subagents that operate in parallel" (vendor post, Anthropic, "How we built our multi-agent research system", 13 June 2025), which the vendor's earlier patterns post had already catalogued as the *orchestrator–workers* workflow (vendor post, Anthropic, "Building effective agents", 19 December 2024).
- **Hierarchical.** Supervisors of supervisors; a tree of decomposition. Owned by the hierarchical guide.
- **Peer / debate.** Multiple instances propose and argue; a synthesis is reached. The canonical preprint is Du et al., "Improving Factuality and Reasoning in Language Models through Multiagent Debate" (arXiv:2305.14325, 23 May 2023, preprint), where independent model instances "propose and debate their individual responses and reasoning processes over multiple rounds."
- **Pipeline / chaining.** The vendor patterns post's *prompt chaining* and *parallelization* workflows both sit here (vendor post, 19 December 2024): fixed steps, or parallel branches aggregated programmatically.

Three families of motivation recur for building these — **parallelism, separation, and verification** — and the seven properties below are the decomposition of "why" into claims that can each be tested against a skill.

### 3.2 The decomposition — seven properties, each a claim about what the topology buys

A multi-agent topology is not one thing. The value it delivers decomposes into at least seven separable properties, and the entire replace argument turns on treating them separately. Each is stated below as a **claim** — the claim a team implicitly makes when it builds the topology for that reason. §4 tests each claim against a skill.

| # | Property | The claim the topology makes | In one line |
|---|---|---|---|
| **i** | **Context isolation** | Each worker reasons in its own window; the supervisor's window is not polluted by intermediate reasoning, and a worker cannot be confused by another worker's context. | Separate windows buy focus and capacity. |
| **ii** | **Parallelism and latency** | Independent subtasks run at the same time, so wall-clock time falls and throughput rises. | More workers, less waiting. |
| **iii** | **Specialisation and prompt hygiene** | A role can carry its own prompt, and a prompt tuned for one job does not have to also serve another. | One job per prompt. |
| **iv** | **Independent verification and adversarial separation** | A verifier that has *not* seen the producer's chain-of-thought is a genuinely independent check, not a self-marking. | Grade with fresh eyes, or with an adversary. |
| **v** | **Fault isolation** | A failure in one worker does not corrupt the others; failures are contained and can be retried or compensated. | Blast radius of a unit. |
| **vi** | **Per-role model and tool choice** | A cheap model can run the easy roles and an expensive one the hard roles; each role can hold only the tools it needs. | Right model, right tools, per role. |
| **vii** | **Shared state and hand-off** | There is a mechanism — a plan, a blackboard, a filesystem, a message — by which a coordinator and its workers exchange state. | How the parts stay coherent. |

### 3.3 Why these seven, and why property (iv) is the pressure point

The seven are chosen because each is independently *observable* — you can ask of any candidate design "does it have this?" — and independently *falsifiable* against the evidence in §6. Two are load-bearing for the verdict:

- **Context isolation (i)** is the property most often presented as the reason multi-agent systems exist at all. Anthropic's research-system post states the mechanism plainly: sub-agents "facilitate compression by operating in parallel with their own context windows," and each also provides "separation of concerns — distinct tools, prompts, and exploration trajectories" (vendor post, 13 June 2025). Cognition's counter-post argues the opposite failure: because context is *not* shared, parallel sub-agents make "conflicting decisions" and produce inconsistent work (vendor post, Cognition / Walden Yan, "Don't Build Multi-Agents", 12 June 2025). The two vendors disagree about context isolation's *value*, which is exactly why the guide treats it as a property to be analysed rather than a slogan.
- **Independent verification (iv)** is where the strongest argument against blanket replacement sits, because a skill loaded into a single context cannot, by construction, give the verifier a *context the producer never saw*. The literature on LLM self-evaluation makes this concrete: evaluators "recognize and favor their own generations," exhibiting a measurable self-preference bias (Panickssery et al., arXiv:2404.13076, 15 April 2024, preprint). A verification step that shares the producer's context is not independent in the sense that matters. §4.4 develops this carefully and does not hand-wave it away.

The other five are real but more often *contingent*: they attach to the topology only when the workload and the harness make them necessary, and several of them can be obtained without a second agent at all. §4 takes them one at a time, and for each returns a verdict — **ABSORBED**, **PARTIALLY ABSORBED**, or **STRUCTURAL** — with its evidence and its thinness.
---

## 4. The Replace Claim, Property by Property

This is the analytical hinge of the guide. For each of the seven properties defined in §3.2, this section states what a skill set can do, what the evidence supports, where the evidence is **thin**, and returns one of three verdicts:

- **ABSORBED** — a skill set, on the documented evidence, can deliver this property to a single-agent runtime.
- **PARTIALLY ABSORBED** — a skill set delivers some of it, or delivers it only in specific conditions, and the residue is real.
- **STRUCTURAL** — the property follows from *having a second context/agent*, not from instructions, and a skill cannot manufacture it.

The verdicts are deliberately weighted against blanket replacement. The point of the exercise is not to make skills win; it is to find out where the argument actually holds.

### 4.1 Context isolation — verdict: STRUCTURAL

**The claim.** A sub-agent reasons in its own window; the supervisor's window stays clean; intermediate reasoning does not re-enter where it is not needed. Anthropic's research-system post states this as the mechanism of the whole design: sub-agents "operate in parallel with their own context windows," each providing "separation of concerns — distinct tools, prompts, and exploration trajectories" (vendor post, 13 June 2025).

**What a skill can do.** A skill is *the opposite* of this: it loads into the existing window (vendor post, 16 October 2025; vendor standard, agentskills.io). It reduces the *marginal instruction cost* — via progressive disclosure, only the descriptor is always present and the body loads on trigger — but it does not create a second window. When a skill activates, its body is added to the same context that already holds the system prompt, the conversation, and every other loaded skill. **Progressive disclosure bounds instruction tokens; it does not isolate reasoning state.** These are different budgets, and conflating them is the most common error in this debate.

**Where it is thin — the honest caveat.** There is a real counter-argument, and it should not be waved away. A skill that bundles **script execution** moves *work* out of the model's window: the engineering post's PDF example has Claude run a Python script "without loading either the script or the PDF into context" (vendor post, 16 October 2025). That is genuine context saving — but it is the *tool-execution* form of context economy (the mechanism owned by [mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md) §5, code execution over tools), not context *isolation*. The distinction is precise: **filtering a result before it enters context is not the same as a second window in which independent reasoning can occur.** A sub-agent can *decide what to search next* based on what it has already found, because it has private state; a script cannot, because it has no reasoning state at all.

**Why the verdict is STRUCTURAL.** Isolation exists only where there are two contexts. A skill set is a single context with better-disciplined contents. The only ways to obtain genuine isolation are to spawn a second agent (a sub-agent — which is a topology, however small) or to run a deterministic program (which is not isolation, it is computation). The repository's context guide owns the general theory of context budgets; the property-specific conclusion here is that **context isolation is not a token-budget problem and therefore not a skill problem.**

### 4.2 Parallelism and latency — verdict: PARTIALLY ABSORBED

**The claim.** Independent subtasks run simultaneously; wall-clock time falls. The vendor patterns post presents *parallelization* as a first-class workflow precisely for speed (vendor post, 19 December 2024), and the research-system post notes sub-agents search "simultaneously" (vendor post, 13 June 2025).

**What a skill can do.** Two partial routes exist, and both are real:

1. **Deterministic parallelism in bundled code.** A skill can carry a script that fans out — issues several retrievals, then reduces — and the *fan-out happens inside the script*, not inside the model's context. This delivers parallelism for the parts of a task that are expressible as code, and the engineering post explicitly frames bundled code as the answer for "operations [that] are better suited for traditional code execution" (vendor post, 16 October 2025).
2. **The model's own tool-calling concurrency.** A single agent that can issue multiple tool calls in one turn already parallelises *tool* work without a second agent. This is a property of the harness and the provider API, not of skills, and it is owned by [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md).

**Where it is thin.** Neither route parallelises **model reasoning**. If the parallel work is "generate two independent analyses of the same document," a single context cannot run two reasoners at once; it can only interleave, sequentially, within one window. And the evidence for the *value* of multi-agent parallelism is itself vendor-internal: the 90.2% figure the research-system post reports ("a multi-agent system with Claude Opus 4 as the lead agent and Claude Sonnet 4 subagents outperformed single-agent Claude Opus 4 by 90.2% on our internal research eval") is a **vendor self-evaluation** with no external eval name, sample size, or error bars — and the same post records that multi-agent systems "use about 15× more tokens than chats," so the parallelism is bought at a large cost (vendor post, 13 June 2025).

**Verdict rationale.** PARTIALLY ABSORBED: a skill set absorbs parallelism for the deterministic, code-expressible share of a workload, and does not absorb parallelism of model reasoning. The share is workload-dependent and, on the available evidence, is not the majority for open-ended tasks.

### 4.3 Specialisation and prompt hygiene — verdict: ABSORBED

**The claim.** A role carries its own prompt, so a prompt tuned for one job need not also serve another. The vendor patterns post presents *routing* and *orchestrator–workers* partly for this: "separation of concerns, and building more specialized prompts" (vendor post, 19 December 2024).

**What a skill can do.** This is the property a skill library absorbs most cleanly, and the argument is straightforward. A skill *is* a scoped instruction bundle with its own body, its own references, and its own bundled tools/scripts, triggered by its own description (vendor post, 16 October 2025). The "specialisation" the topology bought by giving each worker a narrower prompt is precisely what a skill gives by keeping a procedure's instructions out of the general prompt until the matching task arrives. The vendor announcement even frames composability explicitly — skills "stack together," with "Claude automatically identifies which skills are needed and coordinates their use" (vendor post, 16 October 2025). Where a topology used a *role* to scope a prompt, a skill scopes the same prompt without a second agent.

**Where it is thin.** Two residual gaps, both modest but real: (a) **prompt hygiene and context cleanliness are not identical.** A topology's specialisation includes *not carrying the other roles' instructions at all*; a skill library keeps every descriptor in context permanently (~100 tokens each, per the platform documentation cited in the progressive-disclosure guide), so a very large library re-creates a small version of the bloat the topology avoided. (b) **Conflict between skills is unresolved by the format.** Two skills whose descriptions match the same task compete nondeterministically; the format provides no arbitration. Neither gap changes the verdict — both are catalogue-management problems (§11), not structural limits.

**Verdict rationale.** ABSORBED. This is the property with the shortest distance between "what the topology did" and "what a skill does," and the evidence for it is the definition of the skill itself.

### 4.4 Independent verification and adversarial separation — verdict: STRUCTURAL

**The claim.** A verifier that has not seen the producer's reasoning is an independent check. The vendor patterns post's *evaluator–optimizer* workflow and *voting* parallelization both rest on this, and the research-system post describes multi-agent systems as aiding "verification" (vendor posts, 19 December 2024; 13 June 2025). The canonical preprint is Multiagent Debate, where independent instances debate toward a common answer (Du et al., arXiv:2305.14325, 23 May 2023, preprint).

**What a skill can do.** A skill can *encode a verification step* — "after drafting, check the following checklist; if any item fails, revise." That is instruction-level verification, and it is useful. It is also, by construction, **self-verification**: the same context that produced the work applies the check, and it cannot un-see its own reasoning. The self-evaluation literature is the reason this matters rather than being a philosophical nicety: LLM evaluators "recognize and favor their own generations," exhibiting a self-preference bias in which an evaluator scores its own outputs higher while human annotators rate them equal (Panickssery et al., arXiv:2404.13076, 15 April 2024, preprint). Self-Refine and Reflexion (Madaan et al., arXiv:2303.17651, 30 March 2023; Shinn et al., arXiv:2303.11366, 20 March 2023, both preprints) show self-critique *helps* — but they also define the ceiling: refinement by the same model is not independent grading.

**Where it is thin — the honest caveat, and it is substantial.** A skill set can obtain *some* independence without a second agent, and pretending otherwise would be dishonest. Three partial routes:

1. **Adversarial instructions.** A skill can instruct the verifier to argue the *opposite* case ("list three reasons this reconciliation is wrong before accepting it"). This is a genuinely different behaviour and it is cheap — but it is a *persona change within one context*, so the producer's reasoning is still present. Evidence that this helps as much as an independent grader is **absent**; the self-preference result suggests it does not fully substitute.
2. **A deterministic checker bundled in the skill.** Where the check is expressible as code (does the total reconcile? is the date in range?), a script is a *better* verifier than any agent — deterministic and repeatable (bundled-code mechanism, vendor post, 16 October 2025). Note the scope: this covers checks that can be written down, which is not the same as judging the quality of an open-ended output.
3. **A separate verification *call*.** A single agent can make a fresh, minimal-context call whose prompt contains only the artifact and the rubric — no prior reasoning. This is *close* to independent verification and it is achievable without a designed topology. But it is still one context and one model: the same model grading, with the known self-preference bias, and no adversary. It is independence-lite.

**Why the verdict remains STRUCTURAL.** Genuine verification requires a grader whose *information state* differs from the producer's. Neither an instruction nor a fresh call within one model reliably delivers that; a separate agent call — ideally a different context and, for the strongest form, a different model — does. A skill can *describe* verification; it cannot *be* a separate evaluator. This, with §4.1, is the strongest argument against blanket replacement, and the guide does not soften it: **the two properties most often touted as the reasons for a multi-agent system are the two a skill set cannot manufacture.**

### 4.5 Fault isolation — verdict: STRUCTURAL (consequences often manageable)

**The claim.** A worker's failure is contained; the supervisor can retry, compensate, or proceed without it. The vendor research-system post describes sub-agents as reducing path dependency and states that multi-agent systems "introduce new challenges in agent coordination, evaluation, and reliability" (vendor post, 13 June 2025), and the MAST taxonomy's "system design" category is, in effect, a catalogue of arrangements that failed to contain faults (Cemri et al., arXiv:2503.13657, 17 March 2025, preprint).

**What a skill can do.** A skill provides *no* fault isolation between parts of one task within one context: if the model goes down a bad path, the whole context is on that path. What a skill *can* do is bundle **error handling in code** — the engineering post's script either extracts the fields or raises, deterministically (vendor post, 16 October 2025) — which isolates faults *within the scripted portion*. And a skill can *ask* the harness to retry a tool call.

**Where it is thin.** The interesting honest reading is that **fault isolation and multi-agent systems are in tension.** The MAST work found that many multi-agent failures arise *because* of coordination, not despite it: the taxonomy identifies 14 failure modes in three categories — system design, inter-agent misalignment, and task verification — from 1,600+ annotated traces across 7 frameworks (Cemri et al., arXiv:2503.13657, 2025, preprint). Cognition's post argues the same from practice: parallel sub-agents make conflicting decisions that the combiner then has to reconcile (vendor post, 12 June 2025). So the topology *buys* fault isolation for one worker's failure and *pays* with a new class of coordination failure.

**Verdict rationale.** STRUCTURAL — isolation of failure follows from separate contexts — but the *practical* consequence is often modest: for a single-agent, skill-driven workflow, the failure mode is "the run went wrong," which is retried at the run level; the topology's finer-grained retry is a real but frequently non-essential advantage. The verdict names the structural fact without overstating its cost.

### 4.6 Per-role model and tool choice — verdict: PARTIALLY ABSORBED

**The claim.** The cheap model runs the easy roles, the strong model the hard ones; each role holds only the tools it needs. The vendor patterns post presents routing to "smaller, cost-efficient models … and hard/unusual questions to more capable models" as a core pattern (vendor post, 19 December 2024), and the research-system post explicitly runs a strong lead with weaker sub-agents (vendor post, 13 June 2025).

**What a skill can do.** Two partial routes:

1. **Tool scoping via the harness.** A skill references only the tools it needs, and where the harness supports per-skill or per-command tool restriction, the effective toolset narrows. This is real and directly reduces both token cost and selection error (the mechanism the progressive-disclosure guide treats as the tool-surface problem).
2. **Model choice via the harness, sometimes.** Some harnesses let a *sub-task or sub-command* run on a cheaper model. Where that exists, a skill can trigger it — but note carefully: the moment the work runs on a different model **in a different invocation**, it is a (small) topology, not a skill substituting for one. If the model switch happens *within one context*, it is not a model switch at all in the sense the property means.

**Where it is thin.** Within a single agent loop driven by a single model, a skill cannot change the model — the model is chosen by the harness for the whole loop. So the property is absorbed only to the extent the harness exposes per-step routing, and the strongest forms of the property (a genuinely cheaper worker model) require exactly the separate invocation that constitutes a topology.

**Verdict rationale.** PARTIALLY ABSORBED: tool scoping yes; true per-role model choice only by reintroducing the second invocation the property originally justified.

### 4.7 Shared state and hand-off — verdict: PARTIALLY ABSORBED

**The claim.** There is a mechanism by which the parts exchange task, plan, and results. The research-system post names the mechanism and its failure: the "game of telephone" in which sub-agent output is copied through the coordinator, and the fix — "artifact systems where specialized agents can create outputs that persist independently," with sub-agents storing work externally and "pass[ing] lightweight references back to the coordinator" (vendor post, 13 June 2025). Cognition makes the opposite-but-complementary point: hand-off fails when implicit decisions are not shared, so "share full agent traces, not just individual messages" (vendor post, 12 June 2025).

**What a skill can do.** Hand-off *as an artifact exchange* is largely absorbed: a single agent with a filesystem already passes state through files rather than through the prompt — the same filesystem pattern the research-system post endorses for sub-agents. A skill can *define the hand-off protocol in prose* ("persist the intermediate result to `/work/recon.json` and continue from there") and can bundle the code to do it. Where a topology used a filesystem hand-off, a skill-driven single agent uses the same filesystem.

**Where it is thin — and this is where the two claims meet.** Hand-off *between agents* presupposes two agents. If all the work is in one context, "hand-off" collapses into "sequence": there is no boundary to cross, and the *coherence* benefits the topology might buy (a fresh, clean receiver) do not appear — but neither does the "game of telephone" loss Cognition warns about, because there is no telephone. So the property is partly moot: a skill set does not *need* hand-off for the work it keeps in one context, and cannot *provide* hand-off for work it splits out, because splitting it out is a topology.

**Verdict rationale.** PARTIALLY ABSORBED — absorbed for the single-context case (via filesystem state), not applicable to the multi-context case by construction.

### 4.8 The replace ledger

| # | Property | Verdict | The honest residuum |
|---|---|---|---|
| i | Context isolation | **STRUCTURAL** | Two contexts cannot be conjured by a document; only code-execution result-filtering looks similar, and it is not reasoning isolation. |
| ii | Parallelism and latency | **PARTIALLY ABSORBED** | Deterministic fan-out in bundled code yes; parallel *model reasoning* no. The vendor's 90.2% is a self-eval. |
| iii | Specialisation and prompt hygiene | **ABSORBED** | The skill *is* the scoped prompt; residual cost is permanent descriptors and unresolved skill conflicts. |
| iv | Independent verification / adversarial separation | **STRUCTURAL** | Self-preference bias is measured; a different information state requires a different context, typically a different model. |
| v | Fault isolation | **STRUCTURAL** (cost often modest) | Isolation follows from separate contexts; the topology also buys a new coordination failure class (MAST). |
| vi | Per-role model and tool choice | **PARTIALLY ABSORBED** | Tool scoping yes; true model routing re-introduces a second invocation. |
| vii | Shared state and hand-off | **PARTIALLY ABSORBED** | Filesystem hand-off absorbed for one context; between-agent hand-off entails the second context. |

**The verdict on REPLACE, stated precisely.** A skill set absorbs fully one property (iii), partly three (ii, vi, vii), and cannot manufacture three (i, iv, v). The three it cannot manufacture include the two — isolation and verification — that most often motivate a multi-agent design in the first place. **Therefore the replace claim is true for a bounded class of workloads** — those whose difficulty is *procedural* (the task is known, the steps are known, the checks are expressible, the value is in doing it consistently) — **and false for workloads whose difficulty is *epistemic* (the task is open, the answer must be found, the work must be judged independently).** That is the split verdict, and it is a consequence of which properties are structural, not of preference.

---

## 5. The Compile Claim — Emitting a Topology from a Procedure

The second claim is about **provenance**: not "do we need the graph," but "where does the graph come from?" If a topology can be emitted from a procedural specification, then designing one by hand is as obsolete as hand-writing a compiled binary's assembly. §5 places the claim in its real lineage, states exactly what has been demonstrated versus proposed, and draws the boundary between emitting a topology and merely parameterising one.

### 5.1 What "compilation" would mean here

Compilation, precisely, is a mechanical transformation from a **higher-level specification** to an **executable artifact**, where the specification is complete enough that the transformation need not invent requirements. For this question:

- **Specification** = a procedural description of what must happen: the steps, their order, their dependencies, the checks, and the roles each step needs. A skill is exactly this shape of artifact — it is a procedure in prose plus optional code.
- **Artifact** = a topology: which agents exist, how they connect, what each is told, which tools and models each holds, and how state passes between them.
- **The transformation** = a compiler that reads the procedure and *emits the graph*.

The boundary that must hold throughout (and that most of the prior art does not cross): **emitting a topology from a procedure** means the compiler decides the *arrangement*; **merely parameterising one** means the arrangement was already fixed by a human and the compiler only fills in prompts, demonstrations, or numeric parameters. These are different kinds of automation and they are frequently conflated.

### 5.2 The prior art — named, dated, and scoped

| Work | Who / kind / date | What it actually does | Does it emit a topology? |
|---|---|---|---|
| **DSPy** ("Compiling Declarative Language Model Calls into Self-Improving Pipelines") | Khattab et al., **preprint**, arXiv:2310.03714, 5 Oct 2023 | Abstracts LM pipelines as "text transformation graphs"; a compiler "optimize[s] any DSPy pipeline to maximize a given metric," creating demonstrations and tuning parameters. | **No.** It compiles **prompts and parameters within a graph the human wrote**; the graph (the program) is authored, not emitted. |
| **ADAS** ("Automated Design of Agentic Systems") | Hu et al., **preprint**, arXiv:2408.08435, 15 Aug 2024 (v2 2 Mar 2025) | Defines "Meta Agent Search," where a meta agent "iteratively programs interesting new agents based on an ever-growing archive of previous discoveries," aiming to invent "novel building blocks and/or combin[e] them in new ways." | **Partly, by search — not by compilation.** It *generates agent designs* in code, but from a metric and an archive, **not from a procedural specification**. |
| **GPTSwarm** ("Language Agents as Optimizable Graphs") | Zhuge et al., **preprint**, arXiv:2402.16823, 26 Feb 2024 | Describes LLM agents as computational graphs; "automatic graph optimizers" do **node optimization** (refine prompts) and **edge optimization** ("improve agent orchestration by changing graph connectivity"). | **Partly.** It *optimizes* connectivity — an existing graph's edges — rather than emitting a graph from a procedure. |
| **AFlow** ("Automating Agentic Workflow Generation") | Zhang et al., **preprint**, arXiv:2410.10762, 14 Oct 2024 | Reformulates "workflow optimization as a search problem over code-represented workflows, where LLM-invoking nodes are connected by edges"; searches (MCTS-style) for better workflows. | **Partly, by search.** It *generates workflows*, but as the optimum of a search, not as the compilation of a human procedure. |
| **MetaGPT** ("Meta Programming for A Multi-Agent Collaborative Framework") | Hong et al., **preprint**, arXiv:2308.00352, 1 Aug 2023 | "Encodes Standardized Operating Procedures (SOPs) into prompt sequences," instantiating an assembly-line of role agents (product manager, architect, engineer, …). | **Closest to yes — but for a fixed library of SOPs.** It maps *encoded, human-designed* SOPs onto a *predefined* role pipeline; it is not a general compiler from an arbitrary procedure to an arbitrary graph. |

Two further pieces frame the lineage. **Self-Refine** (Madaan et al., arXiv:2303.17651, 30 March 2023, preprint) and **Reflexion** (Shinn et al., arXiv:2303.11366, 20 March 2023, preprint) supply the *self-critique loop* that most generated pipelines use as an internal building block — the "evaluator" node that topology search then chooses to place. And **Voyager** (Wang et al., arXiv:2305.16291, 25 May 2023, preprint) is the origin point of the *skill-library* idea in this lineage: an "ever-growing skill library of executable code for storing and retrieving complex behaviors," with "an iterative prompting mechanism that incorporates environment feedback, execution errors, and self-verification." Voyager's library is *discovered* by an agent, not compiled from a procedure — which is the same distinction, inverted, that governs the whole section.

### 5.3 What has been demonstrated versus what is proposed

Being precise about scope is the whole value of this section, because the literature's abstracts are more confident than its results.

**Demonstrated (with scope):**

- **Search over topology space finds designs that beat human baselines on benchmarks.** ADAS reports that Meta Agent Search "can progressively invent agents with novel designs that greatly outperform state-of-the-art hand-designed agents," and that the invented agents "maintain superior performance even when transferred across domains and models" — across coding, science, and math domains (preprint, arXiv:2408.08435, 2024). *Scope:* benchmark tasks with a scalar metric to search against; the "invention" is over code representations of single-agent-style designs, not a proof about open-ended enterprise workflows.
- **Graph connectivity can be optimized as a variable.** GPTSwarm reports node- and edge-level optimization of agent graphs (preprint, arXiv:2402.16823, 2024). *Scope:* optimizing an arrangement against a task metric; the *existence* of a graph is a premise, not an output of a procedure.
- **Workflows can be generated by search over code.** AFlow reformulates workflow generation as search over code-represented workflows (preprint, arXiv:2410.10762, 2024). *Scope:* same as above — a search objective, not a procedure to be compiled.
- **Declarative pipelines can be compiled and self-improve.** DSPy's compiler "optimize[s]" a program to a metric and reports that compiled DSPy programs on small models become "competitive with approaches that rely on expert-written prompt chains" (preprint, arXiv:2310.03714, 2023). *Scope:* **prompts and parameters**, within an authored graph.
- **A procedure can be hard-mapped onto a role pipeline.** MetaGPT encodes SOPs into prompt sequences and reports reduced cascading errors (preprint, arXiv:2308.00352, 2023). *Scope:* a *designed* mapping for software-engineering SOPs, not a general transformation.

**Proposed but not demonstrated:**

- That a **complete human procedure can be compiled into a correct multi-agent topology**. No source retrieved for this guide demonstrates this general capability. What exists is search (ADAS, AFlow), optimization (GPTSwarm), parameterisation (DSPy), and one fixed SOP-to-roles encoding (MetaGPT). Calling any of these "compiling a procedure into a topology" overstates what was shown.
- That a **generated topology is governed** by virtue of its generator being governed — a claim addressed in §8 and §13 and found to be an open, unsupported assumption.

### 5.4 The boundary, stated as a test

To keep the two claims apart in any future discussion, apply this test:

> **Ask what the automation is allowed to change.** If it changes *prompts, demonstrations, parameters, or the choice among fixed steps*, it is **parameterisation** — DSPy's territory, and the arrangement was designed. If it changes **which agents exist and how they connect**, it is **topology generation** — ADAS/GPTSwarm/AFlow territory — and it is *search or optimization against a metric*, not translation from a procedure. If it changes the arrangement **as a deterministic function of a procedure's text**, it is **compilation** — and no retrieved source demonstrates this in the general case.

The distinction matters operationally. Search-generated topologies are *evaluated artifacts*: their provenance is a metric, and their behaviour is only as good as the search's coverage and the metric's fidelity. Compiled topologies would be *derived artifacts*: their provenance is a specification, and their correctness would be a property of the specification plus the compiler. **The field currently has a great deal of the first kind and essentially none of the second.**

### 5.5 The verdict on COMPILE

**PARTIALLY DEMONSTRATED, and not in the sense the claim asserts.** Emitting a topology from a *procedure* is unproven; generating one by *search*, and parameterising one by *compilation*, are demonstrated within narrow benchmark scopes. The practical reading for an engineering team:

- **You can automate topology discovery** against a metric you trust (the ADAS/AFlow/GPTSwarm direction) — this is a real, if young, capability, and it is "generated, not compiled."
- **You can compile prompts and parameters** inside a topology you designed (the DSPy direction) — mature enough to use, and it is not topology generation.
- **You cannot, today, hand a complete procedure to a tool and receive a correct, governed multi-agent topology.** Anyone who tells you they can is, on the evidence in this guide, describing a demo over a benchmark, not a production capability.

The compile claim and the replace claim therefore fail in different ways and for different reasons — replace fails on two structural properties (§4.1, §4.4), compile fails on a missing general mechanism (§5.3) — which is exactly why the guide refuses to merge them into one verdict.
---

## 6. The Evidence Base — Four Literatures

Four separate literatures bear on this question. They were produced by different communities, for different purposes, and none of them was written to answer *this* question. Reading them together is the work; blurring them is the error. Each is presented below with who published it, when, of what kind, and what it shows — and, where it exists, what it does **not** show. No unsourced proposition appears in this section.

### 6.1 The skill-library literature — "an agent can accumulate reusable procedures"

| Work | Who / kind / date | What it shows | What it does not show |
|---|---|---|---|
| **Voyager** | Wang et al., preprint, arXiv:2305.16291, 25 May 2023 | An LLM agent builds "an ever-growing skill library of executable code" and reuses it; the library is "temporal[ly] extended, interpretable, and compositional." Reports 3.3× more unique items and up to 15.3× faster tech-tree progress than prior state of the art. | Minecraft-specific; measures *single-agent* skill accumulation, not substitution for a multi-agent topology. |
| **ExpeL** | Zhao et al., preprint, arXiv:2308.10144, 20 Aug 2023 | An agent improves by "experiential learning" — gathering trajectories and distilling reusable insights across tasks, without fine-tuning. | Shows experience becomes reusable text; does not compare against a multi-agent arrangement. |
| **SkillWeaver** | Zheng et al., preprint, arXiv:2504.07079, 9 Apr 2025 | Web agents "autonomously synthesiz[e] reusable skills as APIs"; reports relative success-rate improvements of 31.8% (WebArena) and 39.8% (real websites), and up to 54.3% improvement when synthesized APIs are transferred to weaker agents. | The skills are synthesized *APIs* (code), not prose; the comparison is against the same agent without the library, not against a topology. |
| **Cradle** | Tan et al., preprint, arXiv:2403.03186, 5 Mar 2024 | A modular LMM framework for "General Computer Control" via screenshots and keyboard/mouse — evidence for the harness-plus-skill direction at the environment level. | Not a skill-library result per se; included as lineage for the "skill as reusable capability" idea. |
| **Agent Skills** | Anthropic, vendor post, 16 Oct 2025; open standard, agentskills.io, 18 Dec 2025 | The artifact *the user's question means*: folders of instructions, scripts, and resources, loaded on demand by progressive disclosure; published as an open cross-platform standard. | A *format and a loading model*. It makes **no claim** about replacing multi-agent systems, and its own vendor documentation is silent on that comparison. |

**What the literature establishes:** reusable procedural artifacts are real, they accumulate, they transfer, and they measurably improve task performance in their domains. **What it does not establish:** that a skill library delivers the *structural* properties of §4. The skill-library papers measure capability on tasks the agent already attempted; they do not measure the isolation or verification properties, because those require a comparison against a multi-agent baseline that these papers do not construct.

### 6.2 The multi-agent failure literature — "topologies fail in patterned ways"

**Cemri et al., "Why Do Multi-Agent LLM Systems Fail?" — MAST (preprint, arXiv:2503.13657, 17 March 2025; v3 26 October 2025).** The most directly relevant failure study. It introduces MAST-Data, "1,600+ annotated traces collected across 7 popular MAS frameworks," and the Multi-Agent System Failure Taxonomy (MAST), built from analysis of 150 traces "validated by high inter-annotator agreement (kappa = 0.88)." The taxonomy identifies **14 failure modes in three categories**: (i) system design issues, (ii) inter-agent misalignment, and (iii) task verification.

What this establishes for this guide, and it is a great deal:

- **Most multi-agent failures are not model failures; they are arrangement failures.** The categories are specification/design, inter-agent misalignment, and verification — i.e., failures *of the topology*, not of the underlying model. This is the strongest single piece of evidence that a topology is a *design object with its own failure surface*, and it cuts both ways: it explains why someone might prefer a skill (fewer arrangement failure modes) *and* why a topology is not free.
- **Verification is named as a first-class failure category** — the third cluster is literally "task verification," which corroborates §4.4's structural reading: verification is something topologies are built to provide and get *wrong* in patterned ways (incomplete verification, incorrect verification).
- **The traces span 7 frameworks** — a breadth that makes the taxonomy credible as a description of the field rather than of one codebase.

What it does **not** establish: that a single-agent, skill-driven arrangement would score better on the same tasks. MAST diagnoses *why multi-agent systems fail*; it does not run the counterfactual of "same task, one agent, a good skill library." **No retrieved source runs that controlled comparison at scale** — this is the central evidence gap for the replace claim, and it is recorded again in §10 and §15.

### 6.3 The context-centric practitioner argument — "the failure is context, not agent count"

**Cognition (Walden Yan), "Don't Build Multi-Agents" (vendor post, 12 June 2025).** The clearest statement of the belief that the multi-agent topology solves the wrong problem. Its two principles:

1. **"Share context, and share full agent traces, not just individual messages."** Its worked example — a Flappy Bird clone decomposed into a background sub-agent and a bird sub-agent that produce visually inconsistent results because neither saw the other's decisions — is the argument that parallel sub-agents make *implicit decisions* that cannot be reconciled at combination time.
2. **"Actions carry implicit decisions, and conflicting decisions carry bad results."**

Its prescription is a **single-threaded linear agent** with careful context management, escalating to context compression (potentially a small fine-tuned model) rather than to more agents, because "in 2025, running multiple agents in collaboration only results in fragile systems."

**How to weigh it.** This is a **vendor post**, and the vendor (Cognition, maker of Devin) has a product interest in the single-agent story — so it is evidence of a *practitioner's considered position*, not an independent measurement. But it is a *strong* argument because it is *mechanistic*: it names why parallel sub-agents lose coherence (unshared implicit decisions), and that mechanism is exactly the phenomenon MAST classifies as "inter-agent misalignment." Read together, the context-centric argument and the MAST taxonomy agree on the *failure mode* while disagreeing on the *remedy*; the disagreement is itself the open question of §10.

### 6.4 The harness-generation and automated-design literature — "the arrangement can be searched or machine-generated"

This is the literature that bears on the **compile** claim, and it sits closest to the repository's harness guides. Its members were scoped in §5.2; what matters here is the *kind* of knowledge it produces.

- **Automated Design of Agentic Systems / Meta Agent Search** (Hu et al., preprint, arXiv:2408.08435, 2024): a meta agent programs new agents in code; generated designs beat hand-designed baselines on coding, science, and math, and transfer across domains and models.
- **GPTSwarm** (Zhuge et al., preprint, arXiv:2402.16823, 2024): agent graphs with node- and edge-level optimizers — connectivity itself is a variable.
- **AFlow** (Zhang et al., preprint, arXiv:2410.10762, 2024): workflow generation as search over code-represented workflows.
- **MetaGPT** (Hong et al., preprint, arXiv:2308.00352, 2023): SOPs encoded into a multi-agent role pipeline.
- **DSPy** (Khattab et al., preprint, arXiv:2310.03714, 2023): declarative pipeline compilation of prompts/parameters.

**What this literature establishes:** that agent *designs* and *graphs* are objects a machine can generate or optimize against a metric — a genuine, demonstrated capability. **What it does not establish:** compilation from a *procedure*. Every member generates by search/optimization/parameterisation; none performs the general procedure→topology translation (§5.4). The repository's harness guides own the harness-generation and self-refinement material proper; this guide uses it only to place the compile claim in its true lineage.

### 6.5 What the four literatures, read together, do and do not settle

| Question | Settled? | By what |
|---|---|---|
| Are reusable procedural artifacts real and useful? | **Yes** | Skill-library literature (§6.1) and the Agent Skills format (§6.1). |
| Do multi-agent topologies fail in patterned, arrangeable ways? | **Yes** | MAST (arXiv:2503.13657, §6.2). |
| Is context isolation/verification obtainable without a second agent? | **No** | No source demonstrates it; theory and self-preference work (§4.4) argue against. |
| Can a topology be generated? | **Yes, by search** | ADAS / GPWSwarm / AFlow (§6.4). |
| Can a topology be *compiled from a procedure*? | **No** | Absence across the surveyed lineage (§5.3). |
| Does a skill library beat a topology on the same tasks? | **Not established** | No controlled comparison found (§6.2); the gap is the point. |

---

## 7. The Mechanisms That Decide It

Properties are the *what*; mechanisms are the *why*. This section isolates the five mechanisms that actually determine where the line between absorbed and structural falls. It is short by design: each mechanism points to the sibling guide that owns its general theory, and states only the conclusion specific to the skills-versus-topology decision.

### 7.1 Context economics — the budget that a skill changes and the budget it does not

Two different budgets are at stake, and the whole confusion lives in not separating them.

- **The instruction budget.** Skills change this, and change it well. Progressive disclosure means a library of procedures costs ~one descriptor each until triggered (vendor post, 16 Oct 2025; documented tier budgets in the progressive-disclosure guide), and bundled code runs "without loading either the script or the PDF into context." A skill library is a *scaling mechanism for instructions*.
- **The reasoning-state budget.** Skills do not change this at all. The model's working state — what it has reasoned, what it has tried, what it has ruled out — remains one growing context regardless of how many skills have loaded. A sub-agent's whole point is to give a *fresh* reasoning state, and fresh reasoning state is not a resource a document can supply.

**The decision rule:** if the bottleneck is *how much instruction the model must carry*, a skill library wins and a topology is often over-engineering. If the bottleneck is *how much reasoning state one window can hold coherently*, a skill library does not touch it. The general theory is owned by [context_engineering_guide.md](context_engineering_guide.md); the property-specific conclusion is §4.1.

### 7.2 State and hand-off — the mechanism that changes character at the boundary

Inside one context, "state" is just the conversation, and "hand-off" is just sequence; there is no protocol to design. Across a context boundary, hand-off becomes a genuine engineering problem — and the sources show it is where topologies bleed. The research-system post's "game of telephone" fix (persist artifacts, pass references) is a hand-off design (vendor post, 13 June 2025). Cognition's "share full agent traces" is a hand-off design (vendor post, 12 June 2025). MAST's "inter-agent misalignment" category is hand-off failure (preprint, arXiv:2503.13657, 2025). **The mechanism that decides it:** a skill set avoids the hand-off problem by not crossing the boundary; a topology must solve it to exist. For a procedure-heavy workflow, *not needing hand-off* is a feature. For a workflow whose parts genuinely cannot share a window, the hand-off is the price of the capability, and a skill can only describe it, not provision it.

### 7.3 Verification independence — the mechanism that no instruction can manufacture

Restated here as a mechanism rather than a property because it is the decisive one. Independence of verification is a function of **information state**, not of instructions:

- Same context, same model, adversarial instruction → not independent (the producer's reasoning is present).
- Same context, same model, "grade this" instruction → self-preference bias applies (Panickssery et al., arXiv:2404.13076, 2024).
- Fresh minimal-context call, same model → independence-lite: the reasoning is absent, the bias is not.
- Separate agent, different context, ideally different model → the strongest available independence, and it is a topology.

Deterministic checkers bundled in a skill (scripts) are *fully* independent for the class of checks they can express — and this is the one place a skill beats a naive verifier agent. But expressible checks are not open-ended judgement, and the hardest verification tasks are the open-ended ones. **Mechanism conclusion:** verification independence scales with *how different the grader's information state is*, and a skill set can increase that difference only up to the "fresh call" ceiling; beyond it, independence requires a separation that is a topology. This is the mechanism behind the second STRUCTURAL verdict.

### 7.4 Failure modes and blast radius — the shape a skill failure shares with a topology failure

A skill failure and a topology failure are the same in one respect and different in another, and knowing which is which prevents both over- and under-reacting.

**Shared:** both fail **silently at the task level**. MAST's verification failures, and a skill whose trigger never fires, both present as "the task didn't get done," not as an exception (MAST, arXiv:2503.13657, 2025; skill trigger failure, §2.2). Both therefore require the same kind of instrument — an evaluation that asserts *outcomes*, not running status — owned by [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md).

**Different:** the *blast radius* differs. A skill failure is typically **behavioural and broad within its trigger class**: a wrong instruction changes behaviour across *every* task that triggers it (documented failure asymmetry, progressive-disclosure guide §6.3). A topology failure is **structural and localised**: one worker fails, the coordinator may absorb or retry, and the blast radius is bounded by the boundary — but the topology also adds the coordination failure class that MAST catalogues. So the skill failure is *wide but shallow* (wrong behaviour everywhere it applies), and the topology failure is *narrow but deep* (one part fails, plus new coordination modes). Neither is uniformly safer; the question is whether the workload can tolerate a broad behavioural error or a localised structural one. For a regulated workflow, a broad behavioural error is the worse of the two, which is why §11's catalogue governance matters more than it first appears.

### 7.5 The five mechanisms in one line each

| Mechanism | What it decides |
|---|---|
| Context economics (§7.1) | Skills scale instructions, not reasoning state. |
| State and hand-off (§7.2) | Skills avoid the hand-off problem; topologies must solve it. |
| Verification independence (§7.3) | Independence is an information state, not an instruction. |
| Failure blast radius (§7.4) | Skill failure is wide-and-shallow; topology failure is narrow-and-deep. |
| Provenance of the arrangement (§5, §7) | Generated topologies are evaluated artifacts; compiled ones would be derived artifacts. |

---

## 8. The Enterprise and Banking Consequence

The engineering analysis has a governance shadow, and for a regulated institution the shadow may matter more than the analysis. The controls asymmetry between a topology and a skill set is the practical heart of this section; the framework material itself — the risk register, the control taxonomy, the assurance workflow — is owned by [ai_risk_register_guide.md](../banking/ai_risk_register_guide.md) and the repository's agent-governance and [agentic_engineering_guide.md](agentic_engineering_guide.md) guides. This section states only what is specific to the skills-versus-topology decision.

### 8.1 A topology is a system; a skill set is a set of documents

The asymmetry, stated plainly:

| Dimension | A multi-agent topology | A skill set |
|---|---|---|
| What it is, in control terms | A **system**: components, interfaces, orchestration, runtime behaviour. | **Documents** plus, where scripts are bundled, small units of code. |
| Reviewable artifact | The architecture — the graph, the roles, the hand-off contracts, the model/tool assignments. | Each skill's instructions, descriptor, and bundled files, individually. |
| Change-control shape | An architecture change: design review, interface review, sometimes a new risk entry. | A document change: a diff, a reviewer, a version bump. |
| Owner | A **system owner** — one named party accountable for the arrangement. | A **procedure owner** — a named party accountable for the instructions. |
| Failure surface | Structural failures + coordination failures (MAST). | Behavioural failures within the trigger class. |
| Where the controls live | Runtime: orchestration, hand-off, per-agent permissions. | Authoring and review: the descriptor, the body, the bundled code. |

The asymmetry is real and it favours skills on *administrative* grounds and topology on *capability* grounds — which is precisely why the decision cannot be made on governance convenience alone. A skill set that quietly takes on work a topology used to do is *still governed* — it just needs a different review than an architecture. The mistake is to assume that because a skill is "just a document," it needs no review at all; its failure mode is broad (§7.4), so the document discipline has to be real (§11).

### 8.2 Who owns a generated topology?

If a topology is **generated** — by search (ADAS, arXiv:2408.08435, 2024) or optimization (GPTSwarm, arXiv:2402.16823, 2024) — the ownership question has no clean answer in the surveyed material. Three candidate positions and their problems:

1. **"The generator's owner owns the output."** Intuitive, but the output is a benchmark-optimized artifact, not a designed one; its behaviour is a function of the metric and the search coverage, neither of which the system owner authored. A named owner accountable for a graph they did not design and cannot fully explain is a governance fiction.
2. **"The searcher's metric is the control."** Treat the objective function as the specification and the search as the implementation. This is the most defensible position, and it implies that **the metric — not the graph — is the reviewable artifact** and must be owned, versioned, and reviewed. But it also means the graph can change whenever the metric or the search changes, which is a silent-change risk (see §8.3).
3. **"Nobody owns it until someone signs it."** The conservative position: a generated topology is a *candidate* until a named human accepts it as a design, at which point it acquires an owner and the generated provenance becomes a documented input. **On the evidence, this is the only position that survives an audit**, because it is the only one that produces an accountable party for an artifact whose behavior is not fully derivable from a specification.

**The finding:** generated topologies break the assumption that the arrangement has an author. The controls that assume "someone designed this" do not transfer cleanly to "a search found this," and no retrieved source provides a template for governing the latter. That absence is recorded in §15.

### 8.3 What does an auditor review when the graph was produced rather than designed?

If the topology was *compiled* or *generated*, the classic review object — a human-drawn architecture with a rationale — may not exist. The auditor's questions, and what the evidence says about answering them:

- **"What is the specification the graph derives from, and is it complete?"** For a compiled topology, the procedure is the answer — and the procedure becomes a control artifact. For a *generated* one, there is no specification, only a metric and an archive; the honest answer is "a search objective," which is weaker evidence of intent.
- **"Is the graph reproducible?"** Compiled or generated deterministically, yes in principle; generated by a stochastic search, the same objective may yield a different graph, which means the reviewed artifact is a *sample*, not a *function*. This is the concrete reason to prefer deterministic generation where it is available.
- **"Was the graph reviewed, or was its metric reviewed?"** The defensible position from §8.2: review the metric, version it, and treat any change to it as a change to the system. This mirrors the progressive-disclosure guide's rule that a *description edit* or an *index-generation change* is a behaviour-affecting change.
- **"Which agent made the decision, with what context?"** For a topology, this is per-step telemetry across agents; for a skill-driven single agent, it is one trace. Counter-intuitively, the single-context case is *easier* to evidence (one trace) and *harder to isolate* (no boundary to attribute to). Auditability favours the skill set on trace simplicity and the topology on attribution granularity.

### 8.4 The reviewable artifact, restated

For the regulator-facing record, the artifact to review depends on which claim is being deployed:

- **Replace deployment (skill set does the work).** The reviewable artifact is the **skill catalogue** — every descriptor, body, and bundled script, versioned and diffed — plus the evaluation that asserts task outcomes (§11). The system owner is whoever owns the workflow; the procedure owners are whoever owns each skill.
- **Compile deployment (topology derives from a procedure).** The reviewable artifacts are the **procedure** (the source) and the **metric** (the objective), both versioned, plus the generated graph *as a build product* with its generation inputs recorded.
- **Neither deployment.** The guidance maps to the banking risk-register and governance guides by name rather than being re-derived here: [ai_risk_register_guide.md](../banking/ai_risk_register_guide.md), the repository's agent-governance material, and [agentic_engineering_guide.md](agentic_engineering_guide.md).

### 8.5 The controls-specific finding

Two controls-specific propositions survive the evidence, and both are stated with their thinness:

1. **The skills route is *administratively lighter* and *behaviourally riskier per unit*.** Lighter because a skill is a document with a diff (vendor product framing, 16 Oct 2025); riskier because a wrong skill fails broadly across everything it triggers (§7.4). The control that reconciles these is catalogue-level review and evaluation (§11), not per-skill ad-hoc reading.
2. **The generated-topology route is *capability-heavier* and *accountability-thinner*.** Heavier because it can produce designs a human would not (ADAS, 2024); thinner because ownership, reproducibility, and intent are all weaker when the arrangement was found rather than designed (§8.2). The control that reconciles these — own the metric, not the graph — is a **defensible inference from the evidence, not a prescription from any retrieved standard**, and it should be labelled as such in any filing.
---

## 9. The Practical State of the Art

Away from the argument, what is actually deployed, as at the writing date? The honest answer is **a hybrid**, and the hybrid is not a compromise — it is the shape the ecosystem's own vendors shipped. Three layers compose in practice.

### 9.1 Layer one — skills loaded by a general agent

The skill artifact is deployed and standardised. Anthropic's announcement states that Agent Skills are "supported today across Claude.ai, Claude Code, the Claude Agent SDK, and the Claude Developer Platform" and that skills "use the same format everywhere" (vendor post, 16 October 2025); the format was published as an open standard on 18 December 2025, and the standard's own client list spans a wide set of agent clients and IDEs (vendor standard, agentskills.io). This is a *cross-vendor* format, not one product's feature. Its stated purpose is specialisation of a *general-purpose* agent — "transforming general-purpose agents into specialized agents" — which is the single-agent specialisation property of §4.3, delivered as shipped capability.

### 9.2 Layer two — sub-agents, including sub-agents that hold skills

The multi-agent layer did not go away; it moved up. Anthropic's production research system runs an orchestrator-worker arrangement with a lead agent and parallel sub-agents (vendor post, 13 June 2025). Cognition's post, which argues *against* multi-agent systems, still describes Claude Code (as of June 2025) as "an agent that spawns subtasks" — with the crucial constraint that it "never does work in parallel with the subtask agent," and the subtask is "usually only tasked with answering a question, not writing any code," precisely because "the subtask agent lacks context from the main agent" (vendor post, 12 June 2025). That is the hybrid in one sentence: **a sub-agent used as a context-isolation device for investigation, not as a parallel producer of artefacts.**

The combined form — a sub-agent that itself loads skills — is the arrangement the ecosystem is converging on: the supervisor holds the plan and the vocabulary of procedures; sub-agents are dispatched into narrower contexts, and each may trigger the skills its sub-task needs. Nothing in the sources retrieved prevents or mandates this composition, and it follows from the two layers being independent mechanisms.

### 9.3 Layer three — the harness and the code-execution surface underneath both

Skills depend on a harness that provides a filesystem and, for script execution, a code-execution surface (vendor post, 16 October 2025). The harness layer is owned by [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) and [agent_scaffolding_guide.md](agent_scaffolding_guide.md); the code-execution form of context economy is owned by [mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md) §5. The practical point for this guide: **the hybrid is enabled by the harness, not by a choice between skills and agents.** The harness is what lets one context run scripts (absorbing part of §4.2 and §4.7), and it is what lets a supervisor dispatch a sub-agent (providing §4.1 and §4.4).

### 9.4 Where the industry actually landed

| Claim | Status at the writing date | Source / kind / date |
|---|---|---|
| Skills became a portable standard, not a vendor silo | **True** | Open standard, agentskills.io; vendor post 16 Oct 2025; standard 18 Dec 2025 |
| Multi-agent systems remained deployed for open-ended, parallel, high-value work | **True** | Orchestrator-worker research system, vendor post 13 Jun 2025 |
| The strongest single-agent advocacy still uses sub-tasks for context isolation | **True** | Cognition, vendor post 12 Jun 2025 |
| Anyone claims a skill set *replaces* a topology wholesale | **Not found** | No retrieved source makes this claim (negative finding) |
| Anyone demonstrates a topology *compiled from a procedure* | **Not found** | Prior art is search/optimization/parameterisation (§5.3) |

**The practical finding:** the deployed hybrid is *skills for procedure, sub-agents for isolation and verification, harness for execution* — which is exactly the split the seven-property analysis predicts. The industry did not choose a side; it specialised each mechanism to the property it actually provides.

---

## 10. The Open Questions

Everything this guide could not settle, stated as questions rather than smuggled into the answers. These are the questions on which the replace and compile claims genuinely turn.

1. **Does a skill-driven single agent beat a multi-agent topology on the same tasks, holding tools and models constant?** MAST diagnoses why topologies fail (arXiv:2503.13657, 2025); no retrieved source runs the controlled counterfactual. Until it exists, both the replace *for* and the replace *against* are supported only by mechanism and practitioner argument.
2. **How much of a workload is "procedure" and how much is "epistemic"?** The split verdict (§4.8) depends on a share this guide can characterise but not measure. Is 80% of enterprise work procedural, or 40%? The answer likely varies by domain and is, at present, unsourced.
3. **Can a fresh minimal-context verification call close enough of the independence gap to matter?** §4.4 marks this as "independence-lite." Whether it degrades gracefully — i.e., how much of the self-preference bias (Panickssery et al., 2024) survives a prompt-only reset — is untested in any source retrieved here.
4. **Is a generated topology reproducible, and does reproducibility even matter if the metric is stable?** ADAS/AFlow/GPTSwarm generate against metrics (2024); whether two runs yield governably-equivalent artifacts is not addressed in their abstracts or in any governance source found.
5. **What is the correct control object for a generated arrangement?** §8.2 offers "own the metric, not the graph" as an inference, not a documented standard. Is that sufficient for an auditor, or is a named human-design sign-off required? No retrieved framework answers this for generated agentic artifacts.
6. **Do skills and sub-agents compose without re-introducing the failures each was meant to avoid?** A sub-agent with skills is a natural hybrid (§9.2); whether the composition inherits both the skill-trigger failure and the hand-off failure, or cancels them, is unmeasured.
7. **At what library size does a skill catalogue become its own context problem?** Each descriptor is ~always present (progressive-disclosure guide's reading of the skills docs); the crossover point between "skills scale instructions" and "skills are a new bloat" is unmeasured.
8. **Is there a class of task where compilation from a procedure is well-defined at all?** Compilation needs a complete specification (§5.1). Many business procedures are intentionally incomplete, leaving judgement to the operator — which would make them uncompilable in principle, not merely uncompiled.

Each of these is a genuine fork in the analysis. A guide that answered them confidently would be inventing; this one leaves them open and names what would close them.

---

## 11. Documentation, Versioning, and Evaluation of Skills as a Catalogue

If a skill set is to absorb the procedural share of a workload (§4.8), it must be governed as a **catalogue** — the same discipline a topology receives as a system, applied to documents. The general versioning and evaluation machinery is owned by [agent_versioning_guide.md](agent_versioning_guide.md) and [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md); what is specific here is what makes a *skill* reviewable and testable. This is where the skill route's administrative lightness (§8.1) is either earned or lost.

### 11.1 What makes a skill reviewable

A skill is reviewable when four things are true of it, each directly traceable to its structure:

1. **The descriptor states the trigger conditions, not the technology.** Because the descriptor is the retrieval key and the only always-present part (vendor post, 16 Oct 2025), it is the highest-leverage text in the artifact. Review must ask: does this description fire on the tasks it should, and not on tasks it should not? The failure asymmetry is documented — under-specified descriptors never trigger; over-broad descriptors always trigger (progressive-disclosure guide §6.2).
2. **The body is scoped.** The format recommends keeping the body under a line budget and splitting overflow into referenced files, so that the *paths* a model reads are deliberate. A body that inlines everything re-creates the bloat skills exist to avoid.
3. **Bundled code is treated as code.** A skill's scripts execute (vendor post, 16 Oct 2025); they are therefore subject to ordinary code review, dependency review, and the vendor's own explicit warning that malicious skills "may introduce vulnerabilities … or direct Claude to exfiltrate data." A script inside a skill is not "documentation."
4. **Overlap is reviewed.** Two skills whose descriptors match the same task compete nondeterministically; nothing in the format arbitrates. Overlap detection is a catalogue-level review task.

### 11.2 What makes a skill versionable

The versioning story is a document-control story, and it is genuinely easier than a topology's:

- **A skill has a diff.** Adding a step, tightening a condition, or fixing an instruction produces a reviewable change with a version number (vendor product framing, 16 Oct 2025: an API versioning endpoint and console version management). This is the same shape change control already handles for policies and runbooks.
- **The catalogue is the versioned unit, not the skill alone.** A change to a skill's descriptor is a change to *when the whole catalogue fires*; a change to one procedure owner's body can shift triggering for others. Version the catalogue's *set* and *ordering*, not only individual skills.
- **Three event classes are behaviour-affecting and deserve the review path of a code change:** a descriptor edit (changes triggering), a bundled-script change (changes execution), and a change to how the model is told to select among skills. This mirrors the progressive-disclosure guide's treatment of a description edit as a functional change.

### 11.3 What makes a skill catalogue testable

Evaluation for a skill catalogue has three separable layers, and conflating them is the common error:

| Layer | Question | Failure it catches |
|---|---|---|
| **Trigger layer** | For a labelled set of tasks, did the skill that *should* fire actually fire? Did it fire when it should not? | The silent never-triggered skill; the always-triggered skill. |
| **Behaviour layer** | Given the skill loaded, does the agent behave as the procedure requires on representative inputs? | A wrong or ambiguous procedure producing broad behavioural drift (§7.4). |
| **Outcome layer** | Does the end-to-end task succeed, on tasks that exercise the *catalogue*, not just one skill? | The catalogue-level interactions (overlap, conflict, bloat) that per-skill tests miss. |

The discipline that makes this real is the same one the topology case demands: **assert specific outcomes, not running status**, because both skill failure and topology failure are silent at the task level (§7.4). The evaluation frameworks, metric design, and harness for this are owned by [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md); the specific finding here is that a skill catalogue needs its *trigger* layer evaluated separately from its *outcome* layer, because the trigger failure and the procedure failure have different causes and different fixes.

### 11.4 The governance of a skill library

Distilled to the controls specific to a skills catalogue:

- **Named procedure owners** per skill, distinct from the system owner of the workflow (§8.1).
- **Catalogue-level review** of descriptors, overlaps, and loading order — not per-skill reading in isolation.
- **Bundled-code review** to the same standard as any executable artifact.
- **Evaluation at all three layers**, re-run on every behaviour-affecting change.
- **A retirement path.** A stale skill is worse than a missing one: it triggers and applies an obsolete procedure with no error (§7.4). Catalogue maintenance is a control, not housekeeping.

---

## 12. The Cymbal Bank Worked Example

> **This section is explicitly fictional and illustrative.** Cymbal Bank is the fictional institution used across this repository as the worked-example persona, and it is the only bank named in this guide. Every workflow detail, count, and figure below is **invented for illustration** and marked as illustrative. **This is a design decision worked through the seven properties — not a measurement, not a benchmark, and not a claim about any real deployment.** Nothing here should be cited as evidence; the real, dated, attributable material is in §4–§7 and §14.

### 12.1 The decision Cymbal must make

Cymbal Bank must deliver a workflow: **counterparty onboarding exception review**. An analyst receives an onboarding case that failed automated checks; the case carries a document pack (regulatory filings, ownership structure, sanctions-screening hits), and the analyst must work it to a disposition — clear, escalate, or reject — with a documented rationale.

The starting design is a **three-agent topology** (illustrative): a supervisor, a **document-extraction worker** that pulls structured facts from the pack, and an **independent-review agent** that challenges the analyst's draft disposition before it is filed. The question Cymbal's engineering lead asks: *can this be a skill set run by one agent instead?*

The rule the lead sets: **work the decision through the seven properties, not by preference.** Below, each property is decided on the property's own terms.

### 12.2 Property-by-property, for this workflow

**(i) Context isolation — STRUCTURAL, but partially avoidable here.** The document pack is large (illustrative: 200–400 pages). A single context can hold *extracted facts* rather than raw pages — and extraction is expressible as a bundled script (vendor post, 16 Oct 2025), so the *raw pages* need never enter the window. That absorbs the bulk of the isolation need. **Residue:** the *review* reasoning — challenging the disposition — benefits from a context that did not build the disposition (§12.3, property iv), and that residue is not absorbable by a script.

**(ii) Parallelism and latency.** Not material here (illustrative): the workflow is a sequence (extract → assess → review → file), not a fan-out, so the topology was not buying meaningful parallelism for this case (consistent with the research-system post's note that "most coding tasks involve fewer truly parallelizable tasks than research," vendor post, 13 Jun 2025). **Decision: irrelevant to this workflow.**

**(iii) Specialisation and prompt hygiene — ABSORBED.** Three roles become three skills: an **extraction** skill (with its bundled script), an **assessment** skill (the disposition rubric), and a **challenge** skill (the adversarial checklist). Each is a scoped body with its own references; the general agent loads only what the current step needs (vendor post, 16 Oct 2025). This is the property a skill set wins most cleanly.

**(iv) Independent verification — STRUCTURAL, and decisive.** Cymbal's supervisory context does not permit a self-graded high-risk disposition. The self-preference result (Panickssery et al., 2024) means the same context cannot give the challenge step the independence the control framework expects (§8.3). A bundled **deterministic checker** can verify the mechanical facts (does the ownership chain reconcile? is the sanctions hit cleared?) — and for those checks, Cymbal prefers the script precisely because it is deterministic (vendor post, 16 Oct 2025). But the **judgement** of whether a disposition is defensible is open-ended, and only a *separate context, ideally a different model*, gives genuine independent challenge (§4.4, §7.3).

**(v) Fault isolation.** Modest (illustrative): a failed extraction is retried at the step level; the workflow is short enough that whole-run retry is acceptable. The topology's finer retry is a real but non-essential advantage here (§4.5).

**(vi) Per-role model and tool choice — PARTIALLY ABSORBED.** Extraction runs cheaply (and mostly in a script); assessment and challenge are the expensive judgement steps. A single-model harness cannot route per role without re-introducing a second invocation; where the harness supports per-command model choice, Cymbal uses it — which is, precisely, a small topology.

**(vii) Shared state and hand-off — PARTIALLY ABSORBED.** The case state (extracted facts, draft disposition) lives in files, not in the prompt; a single agent sequences through the files. For the review boundary, files carry the artifact across — the same "persist and pass references" pattern the research-system post recommends (vendor post, 13 Jun 2025).

### 12.3 The split verdict

**Skill-shaped (absorb into one agent, run by a skill catalogue):** the *procedural* majority — extraction (with bundled deterministic script), assessment against the disposition rubric, mechanical fact-checking, file-based state, and the whole sequencing of the workflow. That is most of the workflow by *volume*, and it is exactly the class §4.8 predicts a skill set handles well.

**Structural (keep as a separate agent):** **one separation only — the independent-review agent**, which must run in a context (and ideally a model) that did not build the disposition. This is property (iv), and it is the separation that is *structural*: it is not about the prompt, it is about the information state of the grader.

**The recommendation is therefore neither "replace" nor "keep":** run the workflow as **one skill-driven agent plus a single verification agent** — a topology of two, reduced from three, with the reduced separation chosen because it was the one that was structural and the others because they were not. Note what was *not* done: Cymbal did not keep the document-extraction worker (its isolation need was absorbed by a bundled extraction script) and did not keep a separate supervisor (its decomposition need was a fixed sequence, not a dynamic plan). Those were **context-and-sequence separations, and context-and-sequence were the things a skill could cover.** The review boundary was a **verification separation, and verification was the thing it could not.**

Cymbal must still govern the result as a **hybrid**: a skill catalogue (§11) with named procedure owners, plus one system-owned verification agent with its own controls (§8). All figures illustrative; the reasoning is the deliverable.

### 12.4 The thesis, reached through the example

Cymbal's answer is the guide's answer, derived rather than asserted: **a skill set absorbed everything that was about context, instruction, and sequence, and it could not absorb the one separation that was about independent judgement.** The decision was made property by property, and the property that survived was the structural one.
---

## 13. The Anti-Patterns

Five anti-patterns, each presented as symptom → cause → guardrail. They are the ways this specific decision goes wrong in practice; the repository's broader failure catalogues ([llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md), [ai_agent_drift_guide.md](ai_agent_drift_guide.md)) own agent failure in general and are not duplicated here.

| # | Anti-pattern | Symptom | Cause | Guardrail |
|---|---|---|---|---|
| 1 | **Treating skills as agents** | A team expects a skill to give independent review, fresh state, or parallel reasoning, and is surprised when it does not. | The homonym (§1.2): "skill" sounds like a capability, so it is assumed to provide the *structural* properties (§4.1, §4.4) that only a separate context can. | State the property being sought, then ask whether it is an *instruction* property or an *information-state* property. If the latter, it is not a skill. |
| 2 | **Treating topology choice as preference** | "We prefer single agents" or "we prefer multi-agent" replaces any analysis of the workload. | Choosing the arrangement by taste rather than by testing it against the seven properties (§3.2), which are observable individually. | Run the seven-property ledger (§4.8) for the specific workflow before choosing. Preference is not a property. |
| 3 | **Compiling a procedure that was never complete** | A generated workflow looks plausible but silently omits a judgement step the human procedure left implicit. | Compilation needs a complete specification (§5.1); many procedures are intentionally incomplete, delegating judgement to an operator. An incomplete procedure compiles into an incomplete topology. | Before compiling, test the procedure for completeness — every decision point either specified or explicitly flagged as human judgement. If it is incomplete, it is not a compilation input. |
| 4 | **Assuming a generated topology is governed because the generator is** | A search-generated graph is deployed under the generator's approval, with no system owner for the *output*. | Provenance confusion: the tool is reviewed, so the artifact is assumed reviewed. Generated arrangements have no author and no stated intent (§8.2). | Own the **metric**, not the graph; require a named human to accept the generated design before deployment; record the generation inputs. |
| 5 | **Rebuilding as a topology what a skill already covers** | Three agents where one agent and a skill set would do; extra hand-off failures appear for no capability gained. | The reverse error: preferring the more "serious" architecture (or mirroring an org chart) for work that is procedural and sequential (§4.3, §4.7). | Apply the ledger the other way too: if a property is ABSORBED, do not pay for a second context to get it. This is the error the Cymbal example avoided for extraction and supervision (§12.3). |

A sixth, subtler error deserves mention because it cuts across all five: **treating "skills vs multi-agent" as one decision at all.** The failure that produces anti-patterns 2 and 5 is a single mental switch, when the actual object is a *ledger of seven separable properties*, three of which a skill set cannot provide and four of which it can provide partly or fully. The whole guide exists to replace the switch with the ledger.

---

## 14. The Claims Audit

Every factual proposition this guide rests on, with its source, date, kind, verdict, and a quality note. Verdicts: **Verified** (checked against the primary source named), **Flagged** (real but a vendor claim, an abstract-level reading, or otherwise requiring caution), **Rejected** (unsupported and not to be cited).

| # | Claim | Source | Date | Kind | Verdict | Quality note |
|---|---|---|---|---|---|---|
| 1 | Agent Skills is a format of folders of instructions, scripts, and resources, loaded dynamically; `SKILL.md` begins with YAML `name`/`description` frontmatter | Anthropic engineering, "Equipping agents for the real world with Agent Skills" | 16 Oct 2025 | Vendor post | **Verified** | Primary vendor engineering post read in full; the definitional source. |
| 2 | Skills load by three-stage progressive disclosure: discovery (metadata), activation (body), execution (resources/code) | Anthropic engineering post; agentskills.io standard | 16 Oct 2025 / 18 Dec 2025 | Vendor post / vendor standard | **Verified** | Both primary pages read; the standard states the three stages by name. |
| 3 | Bundled code can run "without loading either the script or the PDF into context"; code is deterministic and repeatable | Anthropic engineering post | 16 Oct 2025 | Vendor post | **Verified** | Quoted from the post; the key structural fact for §4.2, §4.7. |
| 4 | Agent Skills published as an open standard for cross-platform portability | Anthropic announcement + agentskills.io | 16 Oct 2025; open standard 18 Dec 2025 | Vendor post / vendor standard | **Verified** | Date from both primary pages; client list on standards site. |
| 5 | At the API level, Skills "require the Code Execution Tool beta"; `/v1/skills` endpoint provides versioning/management | Anthropic, "Introducing Agent Skills" | 16 Oct 2025 | Vendor post | **Verified** | Primary product announcement read. |
| 6 | Anthropic research system: orchestrator-worker; multi-agent "outperformed single-agent Claude Opus 4 by **90.2%** on our internal research eval" | Anthropic, "How we built our multi-agent research system" | 13 Jun 2025 | Vendor post | **Flagged** | **Vendor self-evaluation**; no eval name, size, baseline, or error bars. Label whenever cited. |
| 7 | Multi-agent systems "use about 15× more tokens than chats"; token usage explains 80% of BrowseComp variance | Anthropic research-system post | 13 Jun 2025 | Vendor post | **Flagged** | Vendor-internal analysis; directionally informative, not independently reproduced here. |
| 8 | Sub-agents provide separation of concerns through "their own context windows" | Anthropic research-system post | 13 Jun 2025 | Vendor post | **Verified as the source's stated mechanism** | This is the vendor's design rationale; used here as the definition of property (i). |
| 9 | Patterns catalogue: prompt chaining, routing, parallelization (sectioning/voting), orchestrator-workers, evaluator-optimizer | Anthropic, "Building effective agents" | 19 Dec 2024 | Vendor post | **Verified** | Primary post read; used here only as the outline of topologies (§3.1). |
| 10 | Cognition's two principles: share full agent traces; actions carry implicit decisions; parallel sub-agents yield conflicting decisions; recommends single-threaded agent | Cognition (Walden Yan), "Don't Build Multi-Agents" | 12 Jun 2025 (see §15 on the date) | Vendor post | **Verified as published; date Flagged** | Page read in full; the page itself shows no date — 12 Jun 2025 is from secondary sources (§15). |
| 11 | MAST: 14 failure modes in 3 categories (system design, inter-agent misalignment, task verification); 1,600+ traces across 7 frameworks; kappa = 0.88 | Cemri et al., "Why Do Multi-Agent LLM Systems Fail?" | 17 Mar 2025 (v3 26 Oct 2025) | Preprint (arXiv:2503.13657) | **Verified** | Resolved via arXiv API; abstract read at source. The most important independent failure source here. |
| 12 | Voyager builds an "ever-growing skill library of executable code"; 3.3× unique items, up to 15.3× faster tech-tree progress | Wang et al., arXiv:2305.16291 | 25 May 2023 | Preprint | **Verified as published** | arXiv API resolves ID/title/date; figures are the paper's own Minecraft results. |
| 13 | ExpeL: LLM agents as experiential learners, distilling reusable insights without fine-tuning | Zhao et al., arXiv:2308.10144 | 20 Aug 2023 | Preprint | **Verified as published** | Resolved via arXiv API. |
| 14 | SkillWeaver: agents synthesize reusable skills as APIs; +31.8% (WebArena), +39.8% (real sites), up to +54.3% transfer to weaker agents | Zheng et al., arXiv:2504.07079 | 9 Apr 2025 | Preprint | **Verified as published** | Resolved via arXiv API; figures are the paper's own. |
| 15 | Cradle: modular LMM framework for General Computer Control via screenshots + keyboard/mouse | Tan et al., arXiv:2403.03186 | 5 Mar 2024 | Preprint | **Verified as published** | Resolved via arXiv API; lineage only. |
| 16 | ADAS defines Meta Agent Search, in which a meta agent programs new agents; generated agents outperform hand-designed ones and transfer across domains/models | Hu et al., arXiv:2408.08435 | 15 Aug 2024 (v2 2 Mar 2025) | Preprint | **Flagged — abstract-level** | ID/date verified via API; scope read from the abstract, not the full text (§15). |
| 17 | GPTSwarm describes agents as computational graphs with node optimization and **edge optimization** (changing connectivity) | Zhuge et al., arXiv:2402.16823 | 26 Feb 2024 | Preprint | **Flagged — abstract-level** | ID/date verified; "edge optimization" quoted from abstract. |
| 18 | AFlow reformulates workflow optimization as search over code-represented workflows (nodes connected by edges) | Zhang et al., arXiv:2410.10762 | 14 Oct 2024 | Preprint | **Flagged — abstract-level** | ID/date verified; abstract read. |
| 19 | MetaGPT encodes Standardized Operating Procedures into prompt sequences in a multi-agent framework | Hong et al., arXiv:2308.00352 | 1 Aug 2023 | Preprint | **Flagged — abstract-level** | ID/date verified; abstract read. Closest to procedure→topology, but a fixed SOP library. |
| 20 | DSPy compiles declarative LM pipelines; the compiler "optimizes" a program to a metric; abstracts pipelines as text-transformation graphs | Khattab et al., arXiv:2310.03714 | 5 Oct 2023 | Preprint | **Verified as published** | ID/date via API; abstract read. **Compiles prompts/parameters, not topologies** — stated in §5.2. |
| 21 | Self-Refine improves outputs by iterative self-feedback from the same LLM | Madaan et al., arXiv:2303.17651 | 30 Mar 2023 | Preprint | **Verified as published** | ID/date via API. |
| 22 | Reflexion reinforces agents via linguistic feedback rather than weight updates | Shinn et al., arXiv:2303.11366 | 20 Mar 2023 | Preprint | **Verified as published** | ID/date via API. |
| 23 | LLM evaluators "recognize and favor their own generations" — self-preference bias | Panickssery et al., arXiv:2404.13076 | 15 Apr 2024 | Preprint | **Verified as published** | ID/date via API; abstract read. Basis for the verification-independence argument (§4.4). |
| 24 | Multiagent Debate: independent model instances debate over rounds to reach a common answer, improving factuality/reasoning | Du et al., arXiv:2305.14325 | 23 May 2023 | Preprint | **Verified as published** | ID/date via API. |
| 25 | The Agent Skills format is supported across multiple products and, per the standard, multiple clients | Anthropic post; agentskills.io | 16 Oct 2025 / 18 Dec 2025 | Vendor post / standard | **Verified as published** | Product list from vendor post; client showcase from standards site. |
| 26 | A skill library's descriptor is always present (~100 tokens/skill), body loads on trigger, resources load on access | Recorded via [mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md) §6, reading the Agent Skills docs | docs checked 2026 | Vendor documentation (as recorded) | **Flagged** | Carried from the sibling guide's verified record rather than re-fetched in this pass (§15). |
| 27 | "A skill set replaces a multi-agent topology" | — | — | — | **Rejected** | No retrieved source makes this blanket claim; the evidence supports a bounded, split verdict only (§4.8). |
| 28 | "ADAS compiles a procedure into a topology" | — | — | — | **Rejected** | ADAS *searches* over agent code against a metric; it does not compile from a specification (§5.2, §5.4). |
| 29 | "DSPy compiles agent topologies" | — | — | — | **Rejected** | DSPy compiles prompts and parameters **within** an authored graph (arXiv:2310.03714, abstract). |
| 30 | A controlled head-to-head benchmark of a skill-driven single agent versus a multi-agent topology on the same tasks | — | — | — | **Rejected / not found** | No such study found; the central evidence gap (§6.2, §10, §15). |

---

## 15. What Could Not Be Verified

Recorded as explicit negatives, because a silent omission reads as coverage. **None of these should be asserted elsewhere.**

1. **A controlled comparison of a skill-driven single agent against a multi-agent topology, on the same tasks with tools and models held constant.** No source retrieved for this guide runs it. MAST (arXiv:2503.13657, 2025) diagnoses multi-agent failures but does not run the single-agent counterfactual. Every "skills are enough" and every "you still need agents" statement in the discourse is, on the evidence found, an inference from mechanism or a practitioner position, not a measurement.
2. **The full texts of ADAS, GPTSwarm, AFlow, and MetaGPT.** Their identifiers, titles, and dates were verified via the arXiv API and their abstracts were read at source, but the *detailed* scope of what each generated — architectures, baselines, ablations — was **not** re-derived in this pass. Treat the scope statements in §5.3 and §6.4 as abstract-level.
3. **The exact publication day of Cognition's "Don't Build Multi-Agents."** The page read for this guide exposes no publication date; secondary sources (a search result and a third-party summary page) give 12 June 2025. The claim content is verified from the primary page; **the date is not** and should be re-sourced before use in a filing.
4. **The methodology behind Anthropic's 90.2% research-system figure.** The post presents it as "our internal research eval" with no eval name, dataset size, or baseline definition. It should be cited only as a labelled vendor self-evaluation.
5. **Any governance template, standard, or supervisory text addressing a machine-generated agent topology.** No retrieved source provides controls for an arrangement whose provenance is a search metric rather than an author. The "own the metric, not the graph" position in §8.2 is this guide's inference, not a documented standard.
6. **Whether a fresh, minimal-context verification call meaningfully reduces self-preference bias.** §7.3 calls it "independence-lite"; no source retrieved tests a prompt-only reset against an independent-grader baseline. The magnitude of the residual bias is unmeasured here.
7. **The Agent Skills documentation's numeric budgets** (descriptor ~100 tokens, `description` ≤1,024 characters, body <500 lines) were carried from the sibling guide's verified record of the Agent Skills docs rather than re-fetched from the docs pages in this pass. The *engineering-post* claims used in §2 were read directly; the *documentation* numbers were not.
8. **The ServiceNow product-feature sense of "agent skills"** is cross-referenced by name to [servicenow_agentic_platform_guide.md](../servicenow_agentic_platform_guide.md) and was **not** re-read for this guide; any vendor-specific detail belongs to that guide, not this one.
9. **Research-tooling note affecting reproducibility:** the web search backend returned usable results for some queries but was inconsistent; direct extraction of named primary URLs (the vendor engineering posts, the standards site) and the arXiv API over HTTPS with a browser user-agent were the reliable paths. Several vendor pages were JS-rendered or truncated, which is why two items above (the Cognition date; the exact skills-docs budgets) are marked unverified rather than sourced from memory. An empty or thin search result was treated as a **tool limitation, not as absence of material.**
10. **Any claim that a *specific* enterprise has replaced a multi-agent topology with a skill set (or vice versa) in production.** No primary case study was found; the Cymbal example in §12 is explicitly fictional and illustrative, and no real institution's deployment is asserted anywhere in this guide.

---

### Glossary

**Agent Skills** — An open format for reusable procedural artifacts: a directory whose `SKILL.md` carries `name`/`description` metadata, an instruction body, and optional reference files, templates, and executable scripts. Announced 16 October 2025; published as an open standard on 18 December 2025 (vendor post / standard). Sense (a) of §1.2; the subject of this guide.

**Absorbed / Partially absorbed / Structural** — The three verdicts of §4. *Absorbed*: a skill set can deliver the property. *Partially absorbed*: it delivers part, in specific conditions. *Structural*: the property follows from having a second context, not from instructions.

**ADAS (Automated Design of Agentic Systems)** — A research direction, and its "Meta Agent Search" algorithm, in which a meta agent programs new agents in code against an archive of prior discoveries (preprint, arXiv:2408.08435, 2024). Key *compile-claim* prior art, but it *searches* rather than *compiles*.

**Blast radius** — The scope of damage a failure causes. A skill failure is *wide and shallow* (behaviour changes across every task it triggers); a topology failure is *narrow and deep* (one part fails, plus coordination modes) — §7.4.

**Compilation** — Mechanical transformation from a complete higher-level specification to an executable artifact; here, procedure → topology. Distinct from *parameterisation* (fixing the arrangement already chosen) and from *search* (finding an arrangement against a metric) — §5.1, §5.4.

**Context isolation** — The property that a sub-agent's reasoning occurs in a separate context window, isolated from the supervisor's. Property (i); verdict STRUCTURAL because it follows from the number of contexts, not from instructions — §4.1.

**Descriptor** — The `name` and `description` frontmatter of a skill; the only part always in context, and the retrieval key that triggers the body. Its under- or over-specification is the skills pattern's characteristic failure — §2.2, §11.1.

**DSPy** — A programming model that abstracts LM pipelines as text-transformation graphs and provides a compiler that optimizes them against a metric (preprint, arXiv:2310.03714, 2023). Compiles **prompts/parameters**, not topologies — §5.2.

**Fail closed / fail open** — In controls terms, whether a mismatch or error blocks action (closed) or permits it (open). Relevant to catalogue governance and generated artifacts.

**GPTSwarm** — Work describing LLM agents as optimizable computational graphs with node- and edge-level optimizers (preprint, arXiv:2402.16823, 2024). Edge optimization changes connectivity — topology generation by optimization — §5.2.

**Hand-off** — The transfer of task, state, and results across a context boundary. Avoided by a single-context skill set; must be solved by a topology; a named MAST failure category (inter-agent misalignment) — §4.7, §7.2.

**Independent verification** — Verification whose *information state* differs from the producer's — ideally a different context, and for the strongest form a different model. Property (iv); verdict STRUCTURAL; grounded in measured self-preference bias (Panickssery et al., 2024) — §4.4, §7.3.

**MAST (Multi-Agent System Failure Taxonomy)** — The 14-mode, 3-category taxonomy from Cemri et al. (preprint, arXiv:2503.13657, 2025), derived from 1,600+ traces across 7 frameworks. Categories: system design, inter-agent misalignment, task verification.

**Metacognition / self-verification** — An agent grading its own work; useful but not independent. The ceiling of self-verification, not its replacement — §4.4.

**Orchestrator–worker** — A topology in which a supervisor decomposes a task, dispatches sub-agents, and synthesises; the pattern of Anthropic's documented research system (vendor post, 13 June 2025). Outline owned by the hierarchical/hybrid guides.

**Parameterisation** — Automation that fixes prompts, demonstrations, or parameters within an arrangement a human already chose. DSPy's territory; *not* topology generation — §5.4.

**Progressive disclosure** — Loading a surface on demand rather than up front; for skills, the metadata → body → resources tiers. Bounds *instruction* tokens, not *reasoning state* — §4.1, §7.1. Mechanism owned by [mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md).

**Replace / Compile** — The two claims of the question. *Replace*: a skill set does what a topology does (capability). *Compile*: a topology is emitted from a procedure (provenance). Separated throughout; verdicts in §4.8 and §5.5.

**Skill library** — A catalogue of reusable procedural artifacts an agent can load on demand; the lineage runs from Voyager (arXiv:2305.16291, 2023) through SkillWeaver (arXiv:2504.07079, 2025) to the Agent Skills standard — §6.1.

**Self-preference bias** — An LLM evaluator's tendency to score its own generations higher; a measured effect (Panickssery et al., arXiv:2404.13076, 2024) that undercuts self-marking as verification — §4.4.

**Sub-agent** — A separate agent invocation with its own context window, dispatched by a supervisor. The unit that provides isolation and independent verification; not a skill and not a tool call.

**Subsumption** — The per-property relation in which a skill set delivers a property a topology provided, making the topology unnecessary *for that property* (never globally) — §1.3, §4.8.

**Topology** — The arrangement of agents — how many, connected how, communicating through what. For this question, the object the replace and compile claims are about.

**Trigger failure** — A skill whose descriptor never matches (silent: the work proceeds without the procedure) or always matches (the body loads on everything). Silent at the task level; requires outcome-asserting evaluation — §2.2, §7.4.

---

### Cross-References and Further Reading

**Primary sources used in this guide**

- Anthropic engineering, *Equipping agents for the real world with Agent Skills* (16 Oct 2025): https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills
- Anthropic, *Introducing Agent Skills* (16 Oct 2025): https://www.anthropic.com/news/skills — and the Agent Skills open standard (18 Dec 2025): https://agentskills.io/
- Anthropic, *Building effective agents* (19 Dec 2024): https://www.anthropic.com/engineering/building-effective-agents
- Anthropic, *How we built our multi-agent research system* (13 Jun 2025): https://www.anthropic.com/engineering/multi-agent-research-system
- Cognition (Walden Yan), *Don't Build Multi-Agents* (June 2025): https://cognition.ai/blog/dont-build-multi-agents

**Preprints cited (all resolved via the arXiv API)**

- MAST, *Why Do Multi-Agent LLM Systems Fail?* — arXiv:2503.13657 (17 Mar 2025)
- Voyager — arXiv:2305.16291 (25 May 2023); ExpeL — arXiv:2308.10144 (20 Aug 2023); SkillWeaver — arXiv:2504.07079 (9 Apr 2025); Cradle — arXiv:2403.03186 (5 Mar 2024)
- ADAS — arXiv:2408.08435 (15 Aug 2024); GPTSwarm — arXiv:2402.16823 (26 Feb 2024); AFlow — arXiv:2410.10762 (14 Oct 2024); MetaGPT — arXiv:2308.00352 (1 Aug 2023); DSPy — arXiv:2310.03714 (5 Oct 2023)
- Self-Refine — arXiv:2303.17651 (30 Mar 2023); Reflexion — arXiv:2303.11366 (20 Mar 2023); Multiagent Debate — arXiv:2305.14325 (23 May 2023); LLM Evaluators Favor Their Own Generations — arXiv:2404.13076 (15 Apr 2024)

**Repository companions (do not duplicate)**

- [hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md) · [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md) · [multi_agent_banking_guide.md](multi_agent_banking_guide.md) — the multi-agent topologies this guide uses only in outline
- [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) · [agent_scaffolding_guide.md](agent_scaffolding_guide.md) — the harness, the scaffold, and harness/self-harness generation
- [mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md) — progressive disclosure and §6, the skills pattern
- [context_engineering_guide.md](context_engineering_guide.md) — context economics and budgeting
- [agentic_engineering_guide.md](agentic_engineering_guide.md) — the discipline and its lifecycle
- [agent_versioning_guide.md](agent_versioning_guide.md) · [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) — skill/agent versioning and evaluation
- [ai_risk_register_guide.md](../banking/ai_risk_register_guide.md) — the banking risk register and controls
- [servicenow_agentic_platform_guide.md](../servicenow_agentic_platform_guide.md) — the vendor product-feature sense of "agent skills"

---

### Closing Summary

The question was, and remains, two questions. Read as **REPLACE**, it asks whether a skill set can do what a multi-agent topology does; the answer earned in §4 is a **split verdict** — one property fully absorbed (specialisation and prompt hygiene), three partly absorbed (parallelism, per-role model/tool choice, shared state and hand-off), and three structural (context isolation, independent verification, fault isolation). Because the structural three include the two that most often motivate a topology in the first place — isolation and verification — a skill set replaces a topology for **procedural** work and does not for **epistemic** work. Read as **COMPILE**, it asks whether a topology can be emitted from a procedure; the answer earned in §5 is **partially demonstrated and not in the sense the claim asserts** — the lineage (DSPy, ADAS, GPTSwarm, AFlow, MetaGPT) shows prompt/parameter compilation, search, and optimization, but no general procedure→topology compiler, and the boundary between *emitting* a topology and *parameterising* one is the boundary the field has not crossed.

Three findings are durable. First, **the mechanisms decide it**: context economics scales instructions but not reasoning state; verification independence is an information state, not an instruction; and a skill failure is wide-and-shallow where a topology failure is narrow-and-deep. Second, **the deployed reality is already a hybrid** — skills for procedure, sub-agents for isolation and verification, the harness for execution — and the industry's own vendors published into exactly that split rather than choosing a side. Third, **the governance shadow matters**: a topology is a system needing an owner and an architecture review; a skill set is a catalogue of documents needing named procedure owners, catalogue-level review, and outcome-asserting evaluation; and a *generated* topology owns neither cleanly, which is the gap the evidence leaves open.

What a reader should take away is not a winner but a procedure. Work the seven properties for the specific workload; ask of each whether it is an *instruction* property (a skill can carry it) or an *information-state* property (only a separate context can). Keep the separations that are structural — the independent verifier is almost always one — and collapse the separations that were only ever about context, instruction, or sequence. That is not a compromise between two camps; it is what the evidence, read honestly, requires. The question was never whether one mechanism should win.

The question is not whether skills can replace agents, but which separations are structural and which were only ever about context.
