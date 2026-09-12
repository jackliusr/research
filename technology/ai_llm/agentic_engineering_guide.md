# Agentic Engineering — the engineered agent

> **A discipline-level guide to agentic engineering as a profession: what it is and where the term came from, how the engineering lifecycle for agentic systems runs end-to-end (spec → design → build → eval → ship → operate → retire), the craft practices (eval-driven development, regression discipline, review of prompts/context/harness), the roles and team topologies that deliver it, the maturity models that situate it, and the open problems and critiques that remain honestly unresolved.**

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Cymbal Bank  \
> **Repo:** personal research library · **Series:** LLM/AI Engineering & Operations · **Domain:** AI Engineering · Agent Platform · Software Engineering Discipline  \
> **Date:** 12 September 2026

> **Companion guides — this is a synthesis, read them for depth.** This guide is the **discipline-level** synthesis of the agent cluster: it owns the *profession*, the *end-to-end lifecycle*, the *craft*, the *org*, *maturity/adoption*, and *open problems*. It deliberately does **not** re-derive the patterns and layers that siblings own. Harness taxonomy → [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md). Project scaffold → [agent_scaffolding_guide.md](agent_scaffolding_guide.md). Context discipline → [context_engineering_guide.md](context_engineering_guide.md). Versioning and release depth → [agent_versioning_guide.md](agent_versioning_guide.md). Operations depth → [agentops_guide.md](agentops_guide.md). Drift → [ai_agent_drift_guide.md](ai_agent_drift_guide.md). Eval tooling depth → [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md). Eval vs validation → [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md). Spec craft → [spec_driven_development_for_llms_guide.md](spec_driven_development_for_llms_guide.md). Artefact taxonomy → [agentic_solution_artifacts_guide.md](agentic_solution_artifacts_guide.md). Multi-agent topologies → [hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md), [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md), [multi_agent_banking_guide.md](multi_agent_banking_guide.md). Failure modes → [agents_work_fall_apart_guide.md](agents_work_fall_apart_guide.md), [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md). Autonomy spectrum → [autonomous_agents_guide.md](autonomous_agents_guide.md). Platform choice → [ai_agent_platform_selection_guide.md](ai_agent_platform_selection_guide.md), [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md). Governance/security → [ai_governance_framework_guide.md](ai_governance_framework_guide.md), [prompt_injection_guide.md](prompt_injection_guide.md), [ai_red_teaming_guide.md](ai_red_teaming_guide.md). Adoption → [ai_adoption_strategies_guide.md](ai_adoption_strategies_guide.md). Career path → [ai_engineer_roadmap_2026.md](ai_engineer_roadmap_2026.md).

> **How to read it.** Sections 1–3 frame the discipline and its status. Sections 4–11 walk the lifecycle phase by phase, cross-referencing the sibling that owns each phase's depth. Sections 12–15 cover craft, org, maturity, and open problems. Section 16 is an illustrative worked example at a fictional bank. Sections 17–19 are the honesty apparatus. Where a claim could be confirmed only at a secondary source (or not at all this pass), it is flagged **⚠** or **❌** inline and collected in the claims audit — this guide prefers an honest gap to a fabricated citation.

---

## Table of Contents

1. [The Discipline and the Term](#1-the-discipline-and-the-term)
2. [The Demarcation: Against the Adjacent Disciplines](#2-the-demarcation-against-the-adjacent-disciplines)
3. [The Discipline's Status Today](#3-the-disciplines-status-today)
4. [The Engineering Lifecycle for Agentic Systems — Overview](#4-the-engineering-lifecycle-for-agentic-systems--overview)
5. [Phase I — Problem Framing and the Spec](#5-phase-i--problem-framing-and-the-spec)
6. [Phase II — Architecture and Design](#6-phase-ii--architecture-and-design)
7. [Phase III — Build](#7-phase-iii--build)
8. [Phase IV — Evaluation](#8-phase-iv--evaluation)
9. [Phase V — Release](#9-phase-v--release)
10. [Phase VI — Operations](#10-phase-vi--operations)
11. [Phase VII — Retirement and Succession](#11-phase-vii--retirement-and-succession)
12. [The Craft Practices](#12-the-craft-practices)
13. [The Roles, the Teams and the Org](#13-the-roles-the-teams-and-the-org)
14. [Maturity and Adoption](#14-maturity-and-adoption)
15. [The Open Problems and the Critiques](#15-the-open-problems-and-the-critiques)
16. [Worked Example: The Cymbal Bank Agent-Platform Team](#16-worked-example-the-cymbal-bank-agent-platform-team)
17. [Claims Audit](#17-claims-audit)
18. [What Could Not Be Verified](#18-what-could-not-be-verified)
19. [Glossary and References](#19-glossary-and-references)

---

## 1. The Discipline and the Term

### 1.1 The working definition

**Agentic engineering is the discipline of specifying, building, evaluating, releasing, operating, and retiring software systems whose behaviour is produced by an LLM-driven agent — a program that runs tools in a loop toward a goal — and whose correctness is a property of the whole system (model + prompt + tools + context + harness), not of any one artifact in it.**

Two definitions anchor the term as it circulates today, and they are **not the same definition** — a fact worth noticing before adopting either.

| Definition | Author / source | Scope | What it centres |
|---|---|---|---|
| "The practice of developing software **with the assistance of** coding agents." | Simon Willison, *Agentic Engineering Patterns* ✅ | Narrow: the engineering *activity* is still building ordinary software; agents are the tool | Human engineer as orchestrator; code execution in a loop |
| AI engineering = "the process of building applications that use **foundation models**," distinguished from ML engineering (building models in-house). | Chip Huyen, *AI Engineering: Building Applications with Foundation Models* (O'Reilly, 2025) ✅ | Broad: an entire application discipline | Adaptation, evaluation, and systems work once the model already exists |

Willison's coinage is about **how software gets made** (the craft of working *with* agents); Huyen's is about **what kind of software gets made** (applications *on top of* models). This guide uses the broader sense — agentic engineering as the discipline that builds *and* operates agentic systems — and treats Willison's as the practitioner sub-discipline focused on the coding-agent workflow. That both are called "engineering" and neither is standardised is the field's characteristic early-stage condition (see §3 and §15).

**The agent definition underneath.** Willison's working definition — "**Agents run tools in a loop to achieve a goal**" ✅ (simonwillison.net, *What is agentic engineering?*) — is the one this guide assumes. It is deliberately modest, and its history is worth one line: he notes that clearly defining "agent" "has frustrated AI researchers since at least the 1990s" ✅ (ibid.), citing Michael Wooldridge's 1990s work on agent definitions. The term's semantic debt to pre-LLM agent research is real and routinely forgotten.

### 1.2 Provenance — the term is new, and its coinage is contested

- **"Vibe coding"** was coined by **Andrej Karpathy in February 2025** — per Willison, "coincidentally just three weeks prior to the original release of Claude Code" — to describe prompting LLMs to write code while you "forget that the code even exists" ✅ (Karpathy, X, cited in Willison's guide).
- **"Agentic engineering"** as a named practice was popularised by **Simon Willison's *Agentic Engineering Patterns*** guide, whose first chapter defines it as above ✅ (simonwillison.net/guides/agentic-engineering-patterns/). The guide is explicitly "very much a work in progress": Willison writes that "no chapter should be considered finished" ✅ (ibid.).
- **⚠ Contested provenance.** The phrase "agentic engineering" is a *practitioner coinage from a personal weblog*, not a term with an academic origin, formal standards body, or peer-reviewed first use that this pass could locate. Treat "who coined it" as unresolved: multiple authors use it independently, and one secondary write-up (theagenttimes.com) already frames Willison's guide as defining "a discipline" — a normalisation move that outruns the source. ⚠ *Reason: practitioner-only sources; no primary academic coinage found.*

### 1.3 The practitioner discourse

The debate that gives the term its meaning is the **vibe-coding-vs-agentic-engineering** distinction, and Willison's framing of it is the crispest:

> "**Vibe coding** is more useful in its original definition — we need a term to describe **unreviewed, prototype-quality** LLM-generated code that distinguishes it from code that the author has brought up to a **production-ready standard**." ✅ (Willison)

On that reading the two terms are a **risk scale**, not synonyms: vibe coding is the low-discipline end (you accept the code you don't read), agentic engineering is the high-discipline end (the professional engineer amplifies existing expertise with coding agents and still owns quality). Willison's newsletter puts it directly: agentic engineering "represents the other end of the scale: professional software engineers using coding agents to improve and accelerate their work by amplifying their existing expertise" ⚠ (Simon Willison's Newsletter — author's own framing, opinion).

The corollary that matters for a discipline guide: **agentic engineering is defined by what the human still guarantees** — problem specification, tool provision, and verification. Willison: "We need to provide our coding agents with the tools they need to solve our problems, specify those problems in the right level of detail, and verify and iterate on the results" ✅ — and "LLMs don't learn from their past mistakes, but coding agents can, provided we deliberately update our instructions and tool harnesses" ✅ (ibid.). That last clause is the whole lifecycle in miniature: the *harness* and *instructions* are versioned artifacts, and the discipline is the practice of maintaining them.

### 1.4 Academic and vendor equivalents

| Framing | Where it lives | Relationship to agentic engineering |
|---|---|---|
| **"AI engineering"** as a technical discipline | Wikipedia: "a technical discipline that focuses on the design, development, and deployment of AI systems… merges aspects of data engineering and software engineering" ✅ | The closest academic-neutral equivalent; broad enough to contain agentic engineering |
| **"AI engineering"** as a *field* (productionising foundation-model apps) | Chip Huyen (O'Reilly, 2025) ✅; discussed on *The Pragmatic Engineer* ⚠ (secondary) | Huyen's framing is the one most cited in industry; she notes AI engineering and ML engineering are "often used together or interchangeably in practice" ✅ (per Wikipedia citing Huyen) |
| **"Agentic engineering"** as the coding-agent workflow | Simon Willison, *Agentic Engineering Patterns* ✅ | Practitioner sub-discipline; narrower and tool-centric |
| **"Harness engineering"** | used in industry discourse, e.g. Thoughtworks Radar ⚠ | Overlaps agentic engineering but scoped to the harness layer — see [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) |

**⚠ Semantic diffusion — the field cannot yet agree on its own vocabulary.** Thoughtworks' Technology Radar Vol 34 (April 2026) names this as a top theme: "One contributing factor is **semantic diffusion**: the rapid emergence of new terms for evolving practices, often before their meanings have stabilized. For example, terms such as **spec-driven development** and **harness engineering** are sometimes used inconsistently or overlap in meaning." ✅ (thoughtworks.com/radar, Vol 34). The Radar's conclusion is the honest one for this guide too: "Without shared definitions, it's difficult to determine whether we're seeing distinct techniques or simply different labels for similar ideas" ✅ (ibid.).

**⚠ Flag on all of §1.** Everything above on the *term* rests on practitioner sources (a personal weblog, a newsletter, a Wikipedia article citing a trade book) plus one consultancy Radar. There is no standards body, no ISO/IEC usage, and no canonical academic first-use. This is not a defect of the research — it is the state of the field, and a discipline guide that pretended otherwise would be lying about its own maturity.

---

## 2. The Demarcation: Against the Adjacent Disciplines

A discipline is defined as much by its boundaries as its content. This section asks, for each neighbour: **what transfers wholesale, what transfers in modified form, and what is genuinely new?**

| Adjacent discipline | What transfers unchanged | What changes for agentic systems | New capability not in the neighbour |
|---|---|---|---|
| **Software engineering** | Version control, code review, CI/CD, testing pyramids, requirements/specs, incident response | The *artifact under review* is no longer just code: prompts, context bundles, tool schemas, and harness config are first-class, change without a code deploy, and are often edited by non-engineers ✅ (LangChain, Prompt & Context Hub, via [agent_versioning_guide.md](agent_versioning_guide.md)) | Behaviour is a property of a stochastic, non-deterministic artifact; "passing" is a distribution, not a boolean |
| **ML engineering** | Experiment tracking, data/version lineage, model registries, monitoring for drift | You usually **do not train the model** — you adapt a vendor's foundation model, so the work shifts to adaptation, evaluation, and systems ✅ (Huyen, via Wikipedia) | Prompt/context engineering in place of feature engineering; LLM-as-judge evaluation; tool and permission design |
| **Data engineering** | Pipelines, schema contracts, lineage, quality gates, storage/compute separation | The "data" now includes eval datasets, traces, and retrieval corpora that feed *behaviour* directly | Retrieval/context assembly as a runtime engineering surface → [context_engineering_guide.md](context_engineering_guide.md) |
| **Platform engineering** | Internal developer platforms, golden paths, self-service, paved roads | The platform must now serve *non-deterministic workloads* with tool registries, MCP servers, sandboxes, and eval-as-a-service | Agent runtimes, tool governance, trace stores → [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) |
| **Prompt engineering** | Iterative instruction writing; empirical iteration | Prompt engineering was a *single-artifact* craft; agentic engineering is a *system* craft where the prompt is one of five versioned families ✅ ([agent_versioning_guide.md](agent_versioning_guide.md)) | Harness design, tool-loop control, evaluation-driven iteration |
| **DevOps / SRE** | SLOs, error budgets, canaries, rollback, on-call, postmortems | Non-determinism breaks the "same input → same output" assumption behind classic SLOs; you need *quality* SLOs and eval gates, not only latency/error SLOs ⚠ | Eval-gated promotion, drift detection, judge calibration → [agentops_guide.md](agentops_guide.md), [ai_agent_drift_guide.md](ai_agent_drift_guide.md) |
| **Vibe coding / cursor-driven prompting** | Speed of generation; willingness to let the model propose | The defining difference is **review and verification**: agentic engineering requires that the author "brought [the code] up to a production-ready standard" ✅ (Willison) | The discipline of saying "no" — a verification gate that vibe coding deliberately omits |

### 2.1 Where the demarcation is opinion, not fact ⚠

The table above is this guide's synthesis. Much of the boundary-setting discourse — "is prompt engineering dead?", "is this a real discipline?", "is vibe coding fine for serious work?" — is **opinion published by people with audience incentives**, on both the hype and the sceptic side. The only *structural* claims here that are well-supported are: (a) the artifact families that change without a deploy ✅, and (b) the non-determinism of agentic behaviour. The rest is framing. ⚠ *Reason: boundary claims are practitioner opinion; treat the table as a lens, not a taxonomy.*

**One-line cross-references for the adjacent-depth guides:** the harness layer this discipline builds → [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md); the scaffold it starts from → [agent_scaffolding_guide.md](agent_scaffolding_guide.md); the context it maintains → [context_engineering_guide.md](context_engineering_guide.md); the platform it runs on → [ai_agent_platform_selection_guide.md](ai_agent_platform_selection_guide.md).

---

## 3. The Discipline's Status Today

### 3.1 Job market evidence

**⚠ Thin, secondary, and self-serving evidence — read with care.** The honest summary of what this pass could verify: the *role* exists and is widely advertised under several names, but the numbers circulating are vendor/recruiter content-marketing and the primary labour-market datasets were not accessible this pass.

| Evidence type | What it shows | Confidence |
|---|---|---|
| "AI engineer" as a named job title | Widely used in industry postings and career content; "AI engineer" is the standard informal name for the role ✅ (Wikipedia's AI-engineering article cites Indeed career guidance for the how-to-become path) | High that the title exists; low confidence in any specific growth statistic |
| Specific growth percentages, salary bands, or LinkedIn/Indeed demand reports | **Not verified this pass** | ❌ Omitted rather than fabricated |

**⚠ What can be stated honestly:** the discipline has a *recognised role name* (AI engineer / agent engineer), a *recognised toolchain*, and a *recognised body of practice* — but it does **not** yet have a certification that employers treat as meaningful, a professional body, or a formal accreditation pathway. Do not let any source tell you otherwise without a primary citation.

### 3.2 Curriculums, courses, and training

- **Trade books now define the canon.** Chip Huyen's *AI Engineering* (O'Reilly, 2025) is the most-cited text for the "build applications on foundation models" discipline ✅ (Wikipedia, book pages). Willison's *Agentic Engineering Patterns* is the most-cited *free practitioner* curriculum ✅ (simonwillison.net), explicitly framed as an evolving guide rather than a finished course.
- **Evaluations training is emerging as its own track.** Hamel Husain and Shreya Shankar teach "AI Evals For Engineers & PMs" (Maven), "continuously updated with the latest techniques" ✅ (hamel.dev, *Your AI Product Needs Evals*). This is significant: **eval is being packaged as the discipline's core teachable skill**, not an appendix to prompt engineering.
- **⚠ Vendor education is abundant and conflicted.** Every major model vendor publishes engineering blogs and courses that double as demand generation. Treat vendor curricula as *technique sources*, not neutral evidence of the field's shape. ⚠ *Reason: financial interest in the field's expansion.*

### 3.3 Professional bodies and certifications

**⚠ Absent.** This pass found **no** professional engineering body that accredits "agentic engineers," and **no** certification with demonstrable employer recognition. Compare this to established engineering disciplines where licensure exists. The nearest analogues are:
- vendor certifications (cloud AI tracks) — recognised as *skills signals*, not licensure ⚠;
- the Maven-style cohort course completions above — training evidence, not credentialling ✅;
- conference/academic venues (NeurIPS/ACL workshops on agents) — research community, not professional body ⚠.

**Consequence for practitioners:** in the absence of a credential, **evidence of the discipline is a portfolio of shipped, evaluated, operated agents — plus the eval suites, traces, and postmortems that prove it.** That is the de-facto certification (see §13 for how to assess it in hiring).

### 3.4 The state of the practice, honestly

Thoughtworks' Radar Vol 34 (April 2026) captures the industry's self-assessment better than any vendor: it observed tools "less than a month old — some promising, but ultimately too young to assess," often "maintained by a single contributor working with a coding agent," which "stretches the traditional rhythm of the Radar" ✅ (thoughtworks.com/radar). Its named risk is **"codebase cognitive debt"**: "As more code is generated by AI, it's easier to adopt solutions without developing the mental models needed to understand how they work… This gap in understanding accumulates, making systems harder to reason about, debug and evolve." ✅ (ibid.) The one-line summary of §3: the discipline is *practised at scale, taught commercially, and named consistently* — but it is *not yet institutionalised* (no body, no credential, no settled vocabulary), and its most candid observers describe it as young, fast-moving, and accumulating debt.

---

## 4. The Engineering Lifecycle for Agentic Systems — Overview

### 4.1 The need for a lifecycle, and the honesty about its provenance

**⚠ This lifecycle is this guide's own synthesis.** No single published, canonical lifecycle for "agentic engineering" was located this pass that spans spec → design → build → eval → ship → operate → retire. What exists instead is a set of *partial* lifecycle framings that this synthesis assembles:

- Wikipedia's AI-engineering "workload" section gives one order: problem definition and requirements analysis → data acquisition and preparation → model design and training → system integration → testing and validation → deployment and monitoring ✅ (Wikipedia) — a classic **ML** lifecycle, which under-serves the *agentic* parts (tools, context, non-determinism, retirement).
- Thoughtworks frames AI-assisted development as a change to *how teams make and assess technology* rather than as a phase model ✅ (Radar Vol 34) — it warns against treating technique terms as mature methodology before definitions settle.
- The sibling guides each own a **slice**: the spec slice → [spec_driven_development_for_llms_guide.md](spec_driven_development_for_llms_guide.md); the build/scaffold slice → [agent_scaffolding_guide.md](agent_scaffolding_guide.md); the eval slice → [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md); the release slice → [agent_versioning_guide.md](agent_versioning_guide.md); the operate slice → [agentops_guide.md](agentops_guide.md).

So the seven-phase spine below is **assembled, not cited**. Where a phase's content is sourced, it says so; where the *framing* is ours, it says that too. Treat it as a **planning scaffold**, not a claim of industry consensus. ⚠ *Reason: no canonical published lifecycle found; this is synthesis.*

### 4.2 The spine

```text
  I. SPEC ─► II. DESIGN ─► III. BUILD ─► IV. EVAL ─► V. RELEASE ─► VI. OPERATE ─► VII. RETIRE
  eval-as-spec   harness,     harness-as-   offline +   eval-gated    tracing,      decommission,
  + should-it-   context,     code, AGENTS  online eval promotion,    drift, SLOs,  retain, migrate
  be-an-agent    tools        prompts       judge cal.  canary        incidents
```

The lifecycle is **not** strictly sequential. It is a **spiral**: operation produces traces → traces become eval cases → eval cases re-specify the agent (the loop that Hamel Husain calls the core flywheel: "Evaluation systems create a flywheel that allows you to iterate very quickly" ✅). Think of the diagram as the *first pass*, and of §8.4's trace→eval→spec loop as the engine that runs for the life of the agent.

### 4.3 Phase / primary artifact / exit gate

| Phase | Question it answers | Primary artifacts | Exit gate | Depth lives in |
|---|---|---|---|---|
| **I. Problem framing & spec** | *Should this be an agent at all, and what must it do?* | Problem statement, success criteria, the spec, the **eval set as executable spec** | Spec reviewed; a non-agent alternative considered and rejected in writing | [spec_driven_development_for_llms_guide.md](spec_driven_development_for_llms_guide.md) |
| **II. Architecture & design** | *What structure, tools, context, and topology?* | Harness design, tool schemas, context strategy, topology choice, design review record | Design review passed; failure modes enumerated | [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) |
| **III. Build** | *How do we make the artifacts first-class and reviewable?* | Prompts-as-code, harness-as-code, scaffolding, AGENTS.md, tool implementations | Repo conventions met; artifacts versioned; unit-level checks green | [agent_scaffolding_guide.md](agent_scaffolding_guide.md), [agentic_solution_artifacts_guide.md](agentic_solution_artifacts_guide.md) |
| **IV. Evaluation** | *Does it actually work, and did we break anything?* | Offline eval suite, judge calibration, online eval signals, regression baseline | Eval thresholds met; regressions triaged | [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md), [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md) |
| **V. Release** | *How does a candidate become production safely?* | Version manifest, promotion record, canary plan, rollback target | Eval-gated promotion signed; rollback rehearsed | [agent_versioning_guide.md](agent_versioning_guide.md) |
| **VI. Operations** | *Is it still working, and what is it costing?* | Traces, drift monitors, SLOs, incident postmortems, cost dashboards | SLOs green; drift within band; incidents closed | [agentops_guide.md](agentops_guide.md), [ai_agent_drift_guide.md](ai_agent_drift_guide.md) |
| **VII. Retirement & succession** | *How do we remove it without breaking anything or losing evidence?* | Deprecation notice, data/artifact retention record, migration plan | Successor shadowing complete; retention obligations met | *this guide, §11* |

**Gates, not stages.** The middle column is what auditors and incident reviewers actually ask for. A phase that produces no artifact is a phase that did not happen — the same principle the sibling artifact guide makes explicit → [agentic_solution_artifacts_guide.md](agentic_solution_artifacts_guide.md).

---

## 5. Phase I — Problem Framing and the Spec

### 5.1 The first gate is a question, and the answer is often "no"

The most valuable thing an agentic engineer does in Phase I is **refuse to build an agent** when a simpler system suffices. This is not a stylistic preference; it is the explicit guidance from the vendor with the most production exposure:

> "When building applications with LLMs, we recommend **finding the simplest solution possible, and only increasing complexity when needed**. This might mean **not building agentic systems at all**. Agentic systems often trade latency and cost for better task performance, and you should consider when this tradeoff makes sense." ✅ (Anthropic, *Building effective agents*, 19 Dec 2024)

Anthropic draws the architectural line that Phase I must resolve before Phase II begins:

- **Workflows** are "systems where LLMs and tools are orchestrated through predefined code paths" ✅;
- **Agents** are "systems where LLMs dynamically direct their own processes and tool usage, maintaining control over how they accomplish tasks" ✅.

The decision procedure follows from that distinction: use a workflow when the task decomposes into *known* steps and predictability matters; use an agent when "flexibility and model-driven decision-making are needed at scale" ✅ — and for "many applications… optimizing single LLM calls with retrieval and in-context examples is usually enough" ✅ (ibid.).

> **⚠ Freshness flag.** The *Building effective agents* post is dated December 2024 and now carries an editor's note: "Much of the tooling landscape described in this post has changed since December 2024." ✅ The **architecture guidance** (workflows vs agents, composable patterns) remains the most cited framing in the field; the **tooling list** is stale. Cite the framing, not the framework list.

### 5.2 The decision table: should this be an agent?

| Signal | Suggests a **non-agent** (single call / workflow) | Suggests an **agent** |
|---|---|---|
| Task decomposition | Steps known in advance, stable | Subtasks unpredictable, discovered at runtime |
| Failure cost | High; needs determinism | Tolerable with verification gates |
| Latency/cost budget | Tight | Loose enough to absorb a tool loop |
| Tool requirement | Retrieval or one tool call | Multi-step tool orchestration, side effects |
| Correctness definition | Deterministic assertions suffice | Requires judged, probabilistic criteria |
| Ambient trust | Untrusted input, high blast radius | Contained blast radius, approval gates |

This table is **this guide's synthesis** ⚠ — it operationalises Anthropic's published trade-off ("trade latency and cost for better task performance" ✅) rather than quoting a decision framework. Cross-reference the failure-side view before committing → [agents_work_fall_apart_guide.md](agents_work_fall_apart_guide.md) and [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md).

### 5.3 The spec, and the spec-as-executable-eval

Two artifacts come out of Phase I, and the second is what makes the discipline rigorous:

1. **The written spec** — problem statement, scope, non-goals, success criteria, constraints, and the safety/blast-radius boundary. Craft depth (spec types, spec-as-code, SDD maturity) lives in [spec_driven_development_for_llms_guide.md](spec_driven_development_for_llms_guide.md); do not re-derive it here.
2. **The eval-as-spec** — the spec's success criteria expressed as *tests that can be run*. This is the discipline's signature move: a criterion that cannot be evaluated is not a criterion, it is a wish. Hamel Husain's formulation is the practitioner standard: you must have processes "for evaluating quality (ex: tests), debugging issues, and changing the behavior" — and the first is what makes the others fast ✅ (hamel.dev, *Your AI Product Needs Evals*).

> "Write lots of tests and frequently update them." ✅ (Hamel Husain)

**Why spec and eval must be authored together.** Because the eval set is the *specification in executable form*, a spec written without evals is unverifiable by construction, and an eval set written without a spec optimises for whatever is easy to measure. The Phase I exit gate is not "the spec is written" — it is **"the spec is written, and a first cut of its eval asserts each success criterion."**

**Required in the spec (this guide's checklist ⚠):** the user-facing goal; the non-goals (what the agent must *not* do); the escalation path (when it hands off to a human); the blast radius (which tools write, and with what permission); the retirement owner. The last two are what §11 depends on.

### 5.4 The framing anti-patterns

- **Agent-washing**: labelling a lookup as an "agent" to attract budget. The cost is real — you inherit non-determinism for no benefit.
- **Spec drift with no eval**: a spec that grows features nobody can test (see §15 on the reliability/evaluation gap).
- **Eval-after-build**: writing evals only once the agent "works" — this inverts the discipline and produces evals that merely *describe* current behaviour (Hamel Husain's exact failure pattern: teams that "focus exclusively on #3" — changing behaviour — "which prevents them from improving their LLM products beyond a demo" ✅).

---

## 6. Phase II — Architecture and Design

### 6.1 What Phase II decides

Phase II converts a spec into a **system structure**: the harness, the context strategy, the tool surface, and the topology. It does **not** re-derive any of these — the sibling guides own the depth:

| Design surface | What Phase II decides | Depth lives in |
|---|---|---|
| **Harness** (execution, tools, context, lifecycle, observability, verification, governance layers) | Which layers exist, which are delegated to a framework, which are hand-rolled | [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) |
| **Scaffold** (repo structure, components) | Project layout, conventions, where prompts/tools/evals live | [agent_scaffolding_guide.md](agent_scaffolding_guide.md) |
| **Context** (retrieval, memory, skills, bundles) | What enters the context window, when, and how it is curated | [context_engineering_guide.md](context_engineering_guide.md) |
| **Tools** (schemas, permissions, side effects) | The agent-computer interface (ACI) and its permission boundaries | [mcp_framework_tools_guide.md](mcp_framework_tools_guide.md), [mcp_discovery_guide.md](mcp_discovery_guide.md) |
| **Topology** (single, hierarchical, hybrid, multi-agent) | How many agents, and whether delegation earns its coordination cost | [hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md), [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md) |
| **Autonomy level** | Where on the human-approval spectrum the agent sits | [autonomous_agents_guide.md](autonomous_agents_guide.md) |

**One-line cross-reference rule.** At each design surface, cite the sibling and move on. Restating the harness taxonomy here would defeat the purpose of a discipline-level synthesis (the dedup boundary the cluster maintains).

### 6.2 Design patterns worth naming (because they are sourced)

Anthropic's catalogue of composable patterns is the *de facto* shared vocabulary for Phase II ✅ (Anthropic, *Building effective agents*):

| Pattern | Shape | When it fits |
|---|---|---|
| **Prompt chaining** | Sequence of calls with programmatic "gates" between steps | Task cleanly decomposable into fixed subtasks |
| **Routing** | Classifier directs input to a specialised follow-up | Distinct input categories, better handled separately |
| **Parallelization** | Sectioning (independent subtasks) or voting (repeated attempts) | Speed, or higher confidence through diverse attempts |
| **Orchestrator–workers** | A central LLM decomposes, delegates, synthesises | Subtasks *unpredictable* — the key difference from parallelization |
| **Evaluator–optimizer** | One call generates, another evaluates in a loop | Iterative refinement with a reliable critic |

Two cautionary notes from the same source matter for the design review: the most successful implementations "weren't using complex frameworks or specialized libraries… they were building with **simple, composable patterns**" ✅, and frameworks "often create extra layers of abstraction that can obscure the underlying prompts and responses, making them harder to debug" ✅ — with the decisive advice: "If you do use a framework, ensure you understand the underlying code. Incorrect assumptions about what's under the hood are a common source of customer error." ✅

### 6.3 The agent-computer interface (ACI) is a first-class design artifact

The single most transferable design insight in the Anthropic post is that **tool design is prompt engineering**:

> "One rule of thumb is to think about how much effort goes into **human-computer interfaces (HCI)**, and plan to invest just as much effort in creating good **agent-computer interfaces (ACI)**." ✅ (Anthropic)

The supporting evidence is specific: while building their SWE-bench agent, Anthropic "actually spent more time optimizing our tools than the overall prompt," and fixed a model error class by changing a tool to always require absolute filepaths — "the model used this method flawlessly" ✅ (ibid.). Design review should therefore treat tool schemas, descriptions, and error surfaces as **reviewed artifacts**, not glue code.

### 6.4 The design review gate

A design review for an agentic system should answer, in writing (this guide's synthesis ⚠):

1. Why is this an agent (or workflow) and not a single call? (§5.2)
2. Which harness layers exist, and who owns each? → [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md)
3. What is the tool surface, and what is the permission boundary on each tool?
4. What is the context strategy, and what is the token/latency budget? → [context_engineering_guide.md](context_engineering_guide.md)
5. What are the enumerated failure modes and their detections? → [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md)
6. What is the eval plan, and which criteria are judged vs asserted? (§8)
7. What is the versioning and rollback plan? → [agent_versioning_guide.md](agent_versioning_guide.md)

A design that cannot answer (5) and (6) is not ready to build.

---

## 7. Phase III — Build

### 7.1 The central build principle: everything is code, everything is reviewed

Phase III's discipline is one sentence: **the artifacts that determine an agent's behaviour must be treated as code** — versioned, reviewed, tested, and promoted through the same gates as source. The artifact families are exactly those of the versioning guide (prompt, tools, model, parameters, knowledge) ✅ → [agent_versioning_guide.md](agent_versioning_guide.md).

| Build practice | What it means | Why it is a discipline, not a preference |
|---|---|---|
| **Prompts-as-code** | Prompts live in the repo (or a registry), with history and review | They "change more often than the application code around them" ✅ (LangChain, via [agent_versioning_guide.md](agent_versioning_guide.md)) |
| **Harness-as-code** | The loop, tool wiring, and guards are declarative and diffable | Enables the "deliberately update our instructions and tool harnesses" loop Willison names as the whole point ✅ |
| **Eval-as-code** | Eval cases and assertions are repo artifacts that run in CI | Otherwise regressions are invisible (§8, §12) |
| **AGENTS.md / instructions-as-code** | Repo-root instructions that the coding agent reads | The context discipline's entry point → [context_engineering_guide.md](context_engineering_guide.md) |
| **Scaffold conventions** | Fixed places for prompts, tools, evals, traces config | → [agent_scaffolding_guide.md](agent_scaffolding_guide.md) |

### 7.2 "Harness engineering" — the build-phase framing that stuck

Thoughtworks' Birgitta Böckeler gives the build-phase activity a name that has since circulated widely. Reviewing OpenAI's write-up on "Harness Engineering," she summarises the harness as including "**context engineering, architectural constraints, and garbage collection of the code base**" ✅ (martinfowler.com, *Harness Engineering — first thoughts*, 17 Feb 2026). Two things make this framing useful for Phase III:

- It reframes the build from "writing the agent" to **maintaining the environment the agent runs in** — instructions, constraints, and cleanup. The "garbage collection of the code base" clause is the interesting one: as agents generate more code, keeping the codebase *legible* becomes an engineering cost in itself.
- It converges with Thoughtworks' Radar warning about **codebase cognitive debt** — "the mental models needed to understand how [solutions] work" eroding as code is generated faster than it is understood ✅ (Radar Vol 34). Phase III's true deliverable is not "code that runs" but **code that a team can still reason about**.

> **⚠ Attribution note.** The "Harness Engineering" phrase here is credited to an OpenAI write-up as *described by* Böckeler; this pass read her memo, not the original OpenAI piece. Cite the memos you have read; flag the second-hand layer. The harness **taxonomy** (ETCLOVG) lives in [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md).

### 7.3 New habits for cheap code

The macro-change in Phase III is economic, and it is the one Willison names first: "**writing code is cheap now**," which "disrupts *so many* of our existing personal and organizational intuitions about which trade-offs make sense" ✅. But — and this is the discipline's whole tension — "delivering new code has dropped in price to almost free… but delivering ***good*** code remains significantly more expensive than that" ✅ (ibid.).

Willison's definition of the nine properties of "good code" (works; *we know* it works; solves the right problem; handles errors; simple and minimal; protected by tests; documented accurately; affords future change; the "ilities") is the best available checklist for what Phase III must still guarantee by hand ✅ (simonwillison.net, *Writing code is cheap now*). The build-phase corollary: **the agent writes the code; the engineer owns the nine properties.** No tool offloads that.

### 7.4 Build-phase artefacts

The taxonomy of design-time artefacts Phase III produces — prompts, tool schemas, harness config, eval suites, runbooks — is owned by [agentic_solution_artifacts_guide.md](agentic_solution_artifacts_guide.md). Use it as the build checklist; do not duplicate it here.

---

## 8. Phase IV — Evaluation

### 8.1 Eval-driven development is the discipline's core

If this guide had to name the one practice that separates agentic engineering from prompt-tinkering, it is this: **build the evaluation before you build the feature, and let the evaluation drive the iteration.** The practitioner canon is Hamel Husain's *Your AI Product Needs Evals*, whose diagnosis is blunt:

> "Unsuccessful products almost always share a common root cause: **a failure to create robust evaluation systems**." ✅ (hamel.dev)

And whose theory of change is the flywheel: "Like software engineering, success with AI hinges on how fast you can iterate. You must have processes and tools for: (1) Evaluating quality (ex: tests). (2) Debugging issues (ex: logging & inspecting data). (3) Changing the behavior or the system… **Many people focus exclusively on #3**, which prevents them from improving their LLM products beyond a demo." ✅ (ibid.)

### 8.2 The three levels, and their cadence

Husain's three-level model is the most-cited structure for agent evaluation, and its cadence discipline is the practical part ✅ (hamel.dev):

| Level | What it is | Cadence (as recommended) | Cost |
|---|---|---|---|
| **L1 — Unit tests / assertions** | Code assertions on outputs (regex, counts, schema, invariants) | "on every code change" | Cheapest |
| **L2 — Model & human eval** | LLM-judge and human review, including debugging | "on a set cadence" | Medium |
| **L3 — A/B testing** | Online experiment against real users | "only after significant product changes" | Most expensive |

Two sequencing rules from the same source: "The cost of Level 3 > Level 2 > Level 1… **conquer a good portion of your Level 1 tests before you move into model-based tests**" ✅. And the diagnostic heuristic that a suite is working: **"the model struggles to pass them"** — if evals are trivial, they are theatre ✅.

### 8.3 The rules that keep evals honest

Husain's closing takeaway list is effectively a profession's code of conduct for eval work, and each line is a sourced claim ✅ (hamel.dev):

- "Remove ALL friction from looking at data."
- "You are doing it wrong if you aren't looking at lots of data."
- "**Don't rely on generic evaluation frameworks to measure the quality of your AI. Instead, create an evaluation system specific to your problem.**"
- "Re-use your eval infrastructure for debugging and fine-tuning."
- "Keep it simple. Don't buy fancy LLM tools. Use what you have first."

That third line is the discipline's most important warning against tool-worship: the eval *tooling* market is real and useful, but a framework will not know your failure modes. Tooling depth (DeepEval, LangSmith, RAGAS, Arize) lives in [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) and is cross-referenced, not re-derived. The distinction between evaluation and validation is owned by [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md).

### 8.4 Offline vs online, and the trace→eval→spec loop

- **Offline eval** runs against a fixed dataset before release; it is the **executable spec** from §5.3 and the gate in §9.
- **Online eval** runs against production traffic and signals — the "A/B testing" level above, plus production monitors → [agentops_guide.md](agentops_guide.md).
- **The loop that makes the discipline live:** production traces reveal a failure → the failure is *captured as an eval case* → the eval case re-specifies the agent → a fix is built and gated against the enlarged suite. This is Hamel Husain's flywheel, and it is why "frequently update them [tests] based on new failures we observe in the data" is a stated requirement, not a nicety ✅ (hamel.dev — Rechat "continuously update them based on new failures").

**Regression discipline.** Because the eval set only grows, a candidate that fixes one failure and breaks another is *caught at the gate* — provided the suite runs in CI (§12.3) and the baseline is versioned. Drift and regression *detection* depth: [ai_agent_drift_guide.md](ai_agent_drift_guide.md).

### 8.5 The evaluation crisis ⚠

**⚠ This is a live controversy, and the guide should not pretend it is settled.** The field has a self-acknowledged measurement problem:

- **Judges are not ground truth.** LLM-as-judge evaluation is cheap and scalable but unreliable in ways that are hard to bound; the practitioner consensus is that judges must be *calibrated against human labels* — an activity whose depth the eval-tooling sibling owns → [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md).
- **Benchmarks saturate and leak.** Public benchmarks age quickly and can appear in training data; agentic benchmarks (e.g. SWE-bench-style task suites) are closer to the work but expensive to run ⚠ *Reason: broad practitioner claim; no single primary source verified this pass.*
- **Vendor-reported evals are not independent.** Any model card's numbers should be treated as marketing until reproduced ⚠.
- **The vocabulary itself is unstable.** Thoughtworks names semantic diffusion — terms "used inconsistently or overlap in meaning" — and this is as true of "eval" as of anything ✅ (Radar Vol 34).

The honest Phase IV position: **prefer a small, hand-curated, problem-specific eval suite you trust over a large generic benchmark you do not** — which is precisely Hamel Husain's advice ✅ — and treat every number you did not produce yourself as a hypothesis.

---

## 9. Phase V — Release

> **Condensed — release depth lives in [agent_versioning_guide.md](agent_versioning_guide.md).** This section states only what the *discipline* requires at the release gate; it does not re-derive the pipeline.

### 9.1 The release principle: promotion, not deployment

An agent release is a **promotion of a versioned artifact set**, not a code deploy. Because behaviour is a function of the five artifact families (prompt, tools, model, parameters, knowledge), a "release" is a **snapshot of all five**, and a rollback is a **re-pointing of a pointer**, not a redeploy ✅ → [agent_versioning_guide.md](agent_versioning_guide.md).

### 9.2 The eval gate, in one table

| Gate | What must be true | Owner | Depth |
|---|---|---|---|
| **Offline eval** | Candidate clears the full eval suite; no regression vs the versioned baseline | Eval/quality engineer | [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) |
| **Shadow** | Candidate runs alongside production on live traffic, compared, no user impact | Platform/ops engineer | [agent_versioning_guide.md](agent_versioning_guide.md) |
| **Canary** | A bounded traffic slice, monitored against quality + latency + cost SLOs | Ops engineer | [agentops_guide.md](agentops_guide.md) |
| **Full rollout** | Canary metrics within band; rollback target rehearsed | Release owner | [agent_versioning_guide.md](agent_versioning_guide.md) |
| **Rollback ready** | The previous good version is one pointer-move away, and the drill was actually run | Release owner | [agent_versioning_guide.md](agent_versioning_guide.md) |

### 9.3 Change management across a volatile artifact set

The release discipline's hardest problem is that artifacts change **without** a deploy — a prompt edit, a model version bump from the vendor, a knowledge-base refresh. The versioning guide's answer is the discipline's answer: **treat the whole binding as versioned config, and gate every change through the same eval chain** ✅ → [agent_versioning_guide.md](agent_versioning_guide.md).

**Model-lifecycle fact every release owner needs:** the provider, not you, retires the model artifact. Anthropic's model lifecycle documentation defines four states — **Active, Legacy, Deprecated, Retired** — and commits to at least **60 days' notice** before retiring publicly released models ✅ (Anthropic docs, as cited in [agent_versioning_guide.md](agent_versioning_guide.md)). Unpinned or unmonitored agents can therefore break on a date the vendor chose.

---

## 10. Phase VI — Operations

> **Condensed — operations depth lives in [agentops_guide.md](agentops_guide.md); drift depth in [ai_agent_drift_guide.md](ai_agent_drift_guide.md).**

### 10.1 The four operational surfaces

| Surface | The discipline's question | Depth |
|---|---|---|
| **Tracing / observability** | Can I reconstruct exactly which prompt + model + tools + context produced this output? | [agentops_guide.md](agentops_guide.md) |
| **Drift / regression** | Has behaviour decayed since release — from model updates, knowledge staleness, or traffic shift? | [ai_agent_drift_guide.md](ai_agent_drift_guide.md) |
| **Incident response** | What is the runbook when the agent produces a harmful or wrong output at scale? | [agentops_guide.md](agentops_guide.md) |
| **Cost / economics** | What does a successful task cost, and what does a *failed* task cost in wasted tokens? | [agentops_guide.md](agentops_guide.md), §14.4 here |

### 10.2 What changes versus classic ops

Non-determinism breaks the "same input → same output" assumption behind classic SLOs. The discipline therefore needs **quality SLOs** (task success rate, escalation correctness, groundedness) in addition to latency/error/cost SLOs ⚠ *Reason: this is a widely repeated practitioner framing, not a single published standard.* The operational loop is the §8.4 flywheel running in production: **traces → eval cases → spec updates → gated releases.**

**The one-line discipline rule for ops:** an agent in production with no trace store and no eval-refresh loop is *unmaintained*, regardless of how well it performs today.

---

## 11. Phase VII — Retirement and Succession

### 11.1 Why retirement is a real phase, not an afterthought

Agents accumulate: a pilot becomes a product, the product gets superseded, and nobody switches it off. The retirement phase exists because agentic systems hold **three kinds of liability that outlive them**:

1. **Behavioural liability** — a still-reachable agent that can call tools and mutate the outside world.
2. **Evidence liability** — traces, eval results, and audit records that a regulated context must retain for a defined period.
3. **Data liability** — conversational data, retrieved context, and memory that may contain personal data subject to retention limits. (Governance depth: [ai_governance_framework_guide.md](ai_governance_framework_guide.md).)

### 11.2 The decommissioning checklist (this guide's synthesis ⚠)

| Step | Action | Evidence produced |
|---|---|---|
| 1. **Declare** | Publish a deprecation notice with a sunset date to all consumers | Deprecation notice, stakeholder sign-off |
| 2. **Freeze** | Stop promotable changes; the agent is feature-frozen | Version manifest frozen at the final good version |
| 3. **Revoke** | Remove tool credentials, MCP server access, and write permissions | Access-revocation record |
| 4. **Succeed** | Stand up the successor; run it in shadow against the incumbent | Shadow comparison report |
| 5. **Migrate** | Move users/consumers; migrate or archive state (memory, sessions) | Migration log |
| 6. **Retain** | Preserve traces, eval results, and audit artifacts per policy | Retention record with expiry |
| 7. **Purge** | Delete conversational data / memory at the retention boundary | Purge certificate |
| 8. **Switch off** | Remove the runtime; archive the repo and artifacts read-only | Decommission record |

### 11.3 The one published retirement lifecycle, and its lesson

The closest thing to a published agent-retirement lifecycle is the **vendor model lifecycle** — Active → Legacy → Deprecated → Retired, with ≥60 days' notice before retirement ✅ (Anthropic docs, via [agent_versioning_guide.md](agent_versioning_guide.md)). Its lesson generalises: **retirement is a scheduled, announced state transition, not an event.** An agent your team cannot announce the retirement of is an agent your team does not fully own.

**Succession versus deletion.** A retired *agent* is often still a live *asset*: its eval suite, its failure catalogue, and its traces are the training data for the successor. Retirement should therefore preserve the **eval suite and the postmortem corpus** while deleting the reachable runtime and the personal data — a distinction that matters the moment the successor is audited.

---

## 12. The Craft Practices

The discipline's craft is the set of day-to-day habits that make the lifecycle real. Six practices, each with a cross-reference to the sibling that goes deeper.

### 12.1 Spec-writing

**Practice:** every agent starts from a spec whose success criteria are *stated as evaluations*. The spec names the goal, the non-goals, the escalation path, and the blast radius (§5.3). A spec that cannot be turned into a test is rewritten, not built.

**Depth:** [spec_driven_development_for_llms_guide.md](spec_driven_development_for_llms_guide.md) owns spec types, spec-as-code, and the SDD maturity model. **Discipline rule:** no spec, no build.

### 12.2 Eval-driven development

**Practice:** write the failing eval first; build until it passes; never delete a passing eval to make a build go green. This is §8 applied as a daily habit, and it is the closest analogue in this discipline to test-driven development.

**⚠ Caveat — is this TDD?** Thoughtworks ran an "informal evaluation of whether prompting a coding agent to follow TDD inside its own agentic loop actually improves the quality of the resulting code and tests, compared to no TDD instructions at all" (Böckeler, 10 Aug 2026) — i.e. whether *TDD-in-the-loop* has value is an **open empirical question**, not settled methodology ⚠ (martinfowler.com, *TDD inside the agent loop* — the memo exists; its conclusion is not summarised here because it was not verified this pass).

### 12.3 The regression / CI discipline for agents

**Practice:** the eval suite runs **in CI on every candidate**, and the CI result is a *distribution comparison*, not a boolean. A candidate that improves aggregate score while regressing a previously-passing case is **not** green.

| CI layer | Runs | Speed | Blocks |
|---|---|---|---|
| **L1 assertions** | Every commit | Seconds–minutes | Merge |
| **L2 judged eval** | On a set cadence / on prompt changes | Minutes–hours | Release candidate |
| **L3 online** | Post-release, real traffic | Continuous | Full rollout (§9.2) |

**Cross-reference:** drift-aware regression detection (detecting decay that is *not* caused by your change) → [ai_agent_drift_guide.md](ai_agent_drift_guide.md).

### 12.4 Code review of prompts, context, and harness

**Practice:** prompts, context bundles, tool schemas, and harness config go through **the same review process as code**. This is the practice that most distinguishes agentic engineering from vibe coding — recall Willison's dividing line: production-grade code is code "the author has brought up to a production-ready standard" ✅.

### 12.5 The review checklist (a practical table)

| Artifact under review | The reviewer must ask | Why |
|---|---|---|
| **Tool schema / description** | Would a junior developer know when and how to call this, from the description alone? Is the parameter set poka-yoke (hard to misuse)? | Anthropic: ACI deserves HCI-level effort ✅ |
| **Tool permissions** | What is the worst action this tool can take, and is that permission justified? | Blast radius (§5.3) |
| **Context strategy** | What enters the window, why, and what is the token budget? | → [context_engineering_guide.md](context_engineering_guide.md) |
| **Harness / loop control** | What stops the loop (step cap, cost cap, timeout)? What happens on tool error? | Unbounded loops are a cost and safety failure |
| **Eval change** | Is a passing case being weakened or deleted? Is a new failure being added as a case? | Eval integrity (§12.2) |
| **Release manifest** | Are all five artifact families pinned and recorded? Is a rollback target named? | §9.2 |

### 12.6 Security and threat modelling (condensed)

**Practice:** agents are an **attack surface**, and threat modelling is a craft habit, not a phase. The two structural reasons: agents consume *untrusted input* (user text, retrieved documents, tool output) into the same channel as their *instructions* — the prompt-injection problem — and agents *hold permissions* that let a successful injection act. Depth: [prompt_injection_guide.md](prompt_injection_guide.md), [ai_red_teaming_guide.md](ai_red_teaming_guide.md), [ai_governance_bias_redteaming_guide.md](ai_governance_bias_redteaming_guide.md).

**⚠ One sourced warning worth carrying into review:** Thoughtworks' Radar Vol 34 carries the theme "**Securing permission-hungry agents**" ✅ — the permission surface, not the model, is the growing risk. Thoughtworks also records that "agentic coding assistants expand the attack surface of software supply chains" (Gumbley and Ryan, 13 May 2025) ✅ — a coding-agent-specific supply-chain risk that standard dependency review does not cover.

### 12.7 The two craft anti-patterns

- **Eval theatre** — a large suite of easy assertions that always pass. Detection: the model never struggles ✅ (Husain's heuristic).
- **Prompt hoarding** — critical instructions living in a chat window or an individual's head rather than the repo. Detection: nobody can answer "which prompt version produced this?" → [agent_versioning_guide.md](agent_versioning_guide.md).

---

## 13. The Roles, the Teams and the Org

### 13.1 The problem: a discipline with no role definitions yet

**⚠ There is no published, standard role taxonomy for this discipline.** No professional body, no accredited curriculum, and no industry consensus job architecture exists that this pass could verify (§3.3). What follows is therefore **constructed from the lifecycle in §4** — each phase implies an owner — and must be read as a *proposal*, not a citation. It is offered because teams are hiring *now* and need a starting scaffold, and it is flagged because pretending a standard exists would be exactly the fabrication this guide refuses.

### 13.2 The five core roles

| Role | Owns (lifecycle phases) | The one question they exist to answer |
|---|---|---|
| **Agent Engineer** | Phase II–III (architecture, build) | *Does the system do what the spec says, robustly, and can we still reason about it?* |
| **Eval / Quality Engineer** | Phase I (eval-as-spec), IV (evaluation), §12.3 CI | *How do we know it works, and did we break it?* |
| **AI Product Manager** | Phase I (spec, framing), V (change management) | *Should this exist, for whom, and what is the blast radius?* |
| **Context / Knowledge Engineer** | Phase II–III (context), VI (knowledge drift) | *What does the agent know, and is that knowledge current and correct?* |
| **Platform / Ops Engineer** | Phase V–VII (release, operate, retire) | *Is it running, safe, observable, affordable, and can we roll it back?* |

Two notes on the set:

- **The eval/quality engineer is the role most specific to this discipline.** Every adjacent discipline has an analogue, but eval-as-spec (§5.3) and the eval-refresh flywheel (§8.4) exist *only* here — and the practitioner canon treats evaluation as the single root cause of product failure ✅ (Hamel Husain: "a failure to create robust evaluation systems"). Organisations that treat eval as a *task* rather than a *role* tend to discover this the hard way.
- **The context/knowledge engineer is a role that did not exist in classical ML engineering.** It is the person who owns retrieval corpora, memory, skills, and AGENTS.md-style instructions → [context_engineering_guide.md](context_engineering_guide.md).

### 13.3 The skills matrix (constructed ⚠)

**⚠ This matrix is this guide's construction.** Rows are the five roles; columns are the craft competencies from §12 and §4. Legend: **●** core/ownership, **○** working competence, **·** awareness.

| Competency | Agent Engineer | Eval Engineer | AI PM | Context Eng. | Platform/Ops |
|---|---|---|---|---|---|
| Problem framing & "should this be an agent?" | ○ | ○ | ● | · | · |
| Spec-writing (with eval-as-spec) | ○ | ● | ● | · | · |
| Harness & loop design | ● | ○ | · | ○ | ○ |
| Context/retrieval engineering | ● | ○ | · | ● | ○ |
| Tool & ACI design | ● | ○ | · | ○ | ○ |
| Eval design (assertions, judges, calibration) | ○ | ● | ○ | ○ | · |
| Regression/CI discipline | ○ | ● | · | ○ | ● |
| Release/versioning & rollback | ○ | ○ | ○ | · | ● |
| Observability, drift, incidents | ○ | ○ | ○ | ○ | ● |
| Cost engineering | ○ | ○ | ● | ○ | ● |
| Governance, security, threat modelling | ○ | ○ | ● | ○ | ○ |
| Domain knowledge (the business) | ● | ● | ● | ● | ○ |

**Reading the matrix.** The same *discipline* spans all five roles; what changes is the centre of gravity. This matters for hiring: an "AI engineer" who is strong at harness design but cannot design an eval is a *builder*, not a full agentic engineer — the gap is exactly where Hamel Husain says products fail ✅.

### 13.4 Team topologies

Three topologies appear in practice; the framework for naming them is Skelton & Pais's *Team Topologies* (stream-aligned / platform / enabling / complicated-subsystem) — **⚠ applying it to agentic teams is this guide's extrapolation, not the authors' claim.**

| Topology | Shape | Best when | Risk |
|---|---|---|---|
| **Embedded** | Agent engineer + PM sit inside the business domain team | A small number of high-value agents; deep domain context needed | Duplicated harness work; inconsistent eval rigour |
| **Platform + enabling** | Central agent platform team builds the paved road (harness templates, eval-as-a-service, tool registry); embedded teams build agents | Many agents across the enterprise | Platform becomes a bottleneck; central team loses domain contact |
| **Federated with a guild** | Embedded teams + a cross-cutting eval/quality guild that sets standards | Regulated contexts needing consistent gate discipline | Guild has no authority; standards decay |

**One sourced pointer for the platform case:** Böckeler wonders whether "harnesses — with custom linters, structural tests, basic context and knowledge documentation, and additional context providers — [will] become the new service templates," while noting that service templates historically suffer "forking and synchronization" problems ✅ (martinfowler.com, *Harness Engineering — first thoughts*, containing *her* framing of OpenAI's write-up). A platform team's real product is therefore **the harness a stream team starts from**, not a framework it must adopt.

**And one sourced reframing of the whole org question:** Kief Morris argues the right human position is **"on the loop," not "in the loop"** — humans *build and manage* the harness rather than inspecting every artefact:

> "The difference between in the loop and on the loop is most visible in what we do when we're not satisfied with what the agent produces… The 'in the loop' way is to fix the artefact… The **'on the loop' way is to change the harness** that produced the artefact so it produces the results we want." ✅ (Kief Morris, martinfowler.com, 4 March 2026)

That reframing has an org consequence: the team's leverage is its **harness quality**, so the org chart should reward harness maintenance as much as feature delivery. Thoughtworks' Radar states the broader shift plainly: "how we collaborate and structure teams will need to evolve… We may need to consider **agent topologies alongside team topologies**, and rethink feedback cycles accordingly." ✅ (Radar Vol 34).

### 13.5 Hiring and assessment (⚠ practitioner opinion)

**⚠ No validated assessment method exists.** The following is this guide's synthesis, offered as a practice, not evidence:

- **Ask for a failure story, not a success story.** The discipline is defined by verification; a candidate who cannot narrate a failure they *detected and instrumented* has not done the work.
- **Give them a broken agent and its traces.** Debugging against a trace store is the daily job.
- **Ask them to design the eval before the feature.** Tests eval-as-spec thinking (§5.3) directly. Watch for "I'd use a generic framework" — Hamel Husain's advice is the opposite ✅.
- **Ask them to say no.** Present a use case that should *not* be an agent (§5.1) and see whether they build it anyway.
- **Portfolio over credential.** In the absence of a professional body (§3.3), the evidence is a shipped agent plus its eval suite, traces, and postmortems.

### 13.6 Growth paths, and the distinction from a learning path

| From | To | The bridging skill |
|---|---|---|
| Agent Engineer | Staff/Principal Agent Engineer | Designing harness *templates* others adopt (platform thinking) |
| Agent Engineer | Eval / Quality Engineer | Shifting from "make it work" to "prove it works and keep proving it" |
| Eval Engineer | AI Product Manager | Owning the *why* as well as the *whether* |
| Platform/Ops Engineer | Agent Platform Architect | From running agents to defining the paved road |
| Any role | Domain Agent Specialist | Depth in one regulated domain (e.g. financial services) |

> **Important distinction — the learning path is not the lifecycle.** [ai_engineer_roadmap_2026.md](ai_engineer_roadmap_2026.md) is a **self-study career roadmap** (maths → deep learning → LLMs → MLOps → certifications → job search). That is a *learning-path artifact* — a sequence a person follows to become employable — and it is categorically different from the **engineering lifecycle** in §4, which is the sequence a *system* follows from spec to retirement. Do not confuse the two: completing the roadmap does not mean you have run the lifecycle, and running the lifecycle repeatedly is what the roadmap is ultimately preparing you for.

---

## 14. Maturity and Adoption

### 14.1 Don't re-derive the maturity models — use them

**Three sibling maturity models already exist and this guide references them rather than inventing a fourth:**

- **Adoption maturity** (enterprise AI phases, org models, talent, build-vs-buy, change management, 12–24 month roadmaps) → [ai_adoption_strategies_guide.md](ai_adoption_strategies_guide.md).
- **Operations maturity** (observability, prod eval, guardrails, cost) → [agentops_guide.md](agentops_guide.md).
- **Spec maturity** (SDD workflow and spec-as-code maturity) → [spec_driven_development_for_llms_guide.md](spec_driven_development_for_llms_guide.md).

What is genuinely missing — and what this section adds — is a **discipline-level maturity model**: how mature is *the engineering practice itself*, independent of how many agents you run?

### 14.2 The agentic-engineering maturity model (constructed ⚠)

**⚠ This model is this guide's synthesis.** It sequences the craft practices of §12 into levels, in the same spirit as the sibling maturity models but scoped to the discipline.

| Level | Name | The tell | What is missing to advance |
|---|---|---|---|
| **0** | **Ad hoc / vibe** | Agents built by prompting; code and outputs unreviewed; no evals | A spec and a first eval |
| **1** | **Defined** | Specs exist; a first eval suite exists; prompts live in the repo | Any *gate* — evals run locally, never block anything |
| **2** | **Gated** | Eval-gated promotion; prompts/context/harness reviewed as code; versioned artifacts | The refresh loop — evals do not grow from production |
| **3** | **Instrumented** | Traces everywhere; drift monitored; incidents have runbooks; cost tracked per task | Systematic iteration — the flywheel runs by hand |
| **4** | **Self-improving** | The trace→eval→spec→release flywheel runs on a cadence (Morris's "agentic flywheel") | Nothing — level 4 is a direction, not a destination |

**Level 4 is sourced in spirit, not in name.** Morris describes exactly this progression as the *agentic flywheel*: "The flywheel becomes more powerful as we feed it richer signals. Add pipeline stages that measure performance and validate failure scenarios. Feed operational data from production, user journey logs, and commercial results" ✅ (martinfowler.com, 4 March 2026). He explicitly frames the endpoint as agents proposing (and eventually auto-applying) harness improvements ✅ (ibid.). The **level names are ours** ⚠; the *direction* is his.

### 14.3 Enterprise failure patterns (⚠ VERIFY, partly sourced)

| Pattern | What it looks like | Evidence |
|---|---|---|
| **Hype-driven adoption** | Buying an "agentic platform" before defining a problem | **Sourced**: Gartner's Hype Cycle for Agentic AI (published 02 April 2026) states that in agentic AI "rapid progress… is exceeded by **hype and confusion**" ✅ (gartner.com/documents/7671861) |
| **Pilot purgatory** | Many proofs-of-concept, few in production ⚠ widely reported; no specific statistic verified this pass | — |
| **Cognitive debt** | Codebase generated faster than understood; debugging slows | **Sourced**: Thoughtworks Radar Vol 34's "codebase cognitive debt" theme ✅ |
| **Eval debt** | Shipping agents whose quality nobody can measure | **Sourced in root cause**: "a failure to create robust evaluation systems" ✅ (Hamel Husain) |
| **Permission sprawl** | Agents holding credentials with no least-privilege review | **Sourced as a theme**: Radar Vol 34's "Securing permission-hungry agents" ✅ |
| **Harness neglect** | Prompts and constraints in individuals' heads, not the repo | **Sourced as a practice**: "When the agent struggles, we treat it as a signal: identify what is missing — tools, guardrails, documentation — and feed it back into the repository" (the anti-pattern is *not* doing this) ✅ (Böckeler on OpenAI's write-up) |

> **⚠ Strong caveat on hype statistics.** A Gartner Hype Cycle for Agentic AI dated April 2026 was verified to exist, with the "hype and confusion" line quoted from Gartner's own summary ✅. Secondary blogs reporting a "17% have deployed" figure were **not** traceable to the primary document this pass — **treat that number as unverified** and do not cite it. ⚠

### 14.4 Build vs buy, and the cost of the discipline

**Build-vs-buy (condensed).** The full decision framework lives in [ai_agent_platform_selection_guide.md](ai_agent_platform_selection_guide.md) and [enterprise_ai_platforms_guide.md](enterprise_ai_platforms_guide.md). The discipline-level rule: **buy the commodity layers (model access, tracing, eval infrastructure, sandboxing); build the harness, the evals, and the domain tools** — because the last three *are* your differentiation and cannot be bought. Böckeler's observation supports the harness half: OpenAI "built extensive tooling for the deterministic part of the harness" rather than adopting it ✅ (martinfowler.com).

**⚠ The cost of the discipline is real and under-modelled.** Three cost centres that a naive TCO misses:

1. **Iteration cost** — agent development is eval-loop-heavy; runs are paid for in tokens and judge calls, not CPU.
2. **Failure cost** — a failed agent task is not free: it burns the tokens of every step taken before the failure, and tool calls may have had *side effects*. Cost dashboards that only count successful executions under-report.
3. **Harness maintenance cost** — Böckeler notes the OpenAI team "worked on their harness for 5 months" before reaching scale ✅, and asks pointedly whether retrofitting a harness to legacy systems is "worth the effort," warning it can resemble "running a static code analysis tool on a codebase that's never had one, and then drowning in alerts" ✅ (ibid.).

These are framed as cost *centres to budget for*, not a claim about exact amounts — **⚠ no reliable per-agent cost benchmark was verified this pass.**

---

## 15. The Open Problems and the Critiques

A discipline guide that lists only its strengths is marketing. These are the problems the field has *not* solved, stated with the evidence available and flagged where the discussion is opinion.

### 15.1 The reliability gap

Agents fail in ways that ordinary software does not: they take wrong-but-plausible actions, they loop, they hallucinate tool arguments, and they are sensitive to input phrasing. The architectural mitigation is to *not build an agent* where a workflow suffices (§5.1) and to gate what you do build (§9). But the gap is not closed, and the harness discourse has a visible hole in it. Böckeler, reviewing the leading "harness engineering" write-up, states the deficiency plainly:

> "All of the described measures focus on increasing long-term internal quality and maintainability. **What I am missing in the write-up is verification of functionality and behaviour.**" ✅ (martinfowler.com, 17 Feb 2026)

That is the discipline's central unsolved problem in one sentence: a large body of practice exists for making agent-produced *code* maintainable, and much less for proving agent *behaviour* is correct. The eval practices in §8 are the response; they are not yet a solution.

### 15.2 Non-determinism as a first-class engineering problem

Because behaviour varies across runs, classic assumptions break: a passing test today is not evidence of a passing test tomorrow; an SLO on "output" requires a distribution; and reproduction of a reported defect may require the exact artifact snapshot. The engineering responses — version everything (§9), snapshot the artifact set, evaluate distributions rather than points (§8.3) — are all *adaptations* of classical practice, not replacements. Whether they are sufficient is open. ⚠ *Reason: no settled standard for non-deterministic SLOs.*

### 15.3 The cost of the loops

Agents are loops, and loops cost: tokens per step, judge calls per eval run, and — in the flywheel — evaluation compute proportional to how often you iterate. Hamel Husain's cadence advice (L1 on every change, L3 rarely ✅) is explicitly a **cost-management** discipline as much as a quality one. The unmodelled part is failure cost (§14.4): a failed multi-step task has already spent its budget, and its tool calls may have had irreversible effects. ⚠ *Reason: no verified benchmark; the practitioner consensus is qualitative.*

### 15.4 The capability-vs-control tension

This is the most interesting *technical* critique, and it is sourced from two independent directions:

- **Böckeler**, on the OpenAI harness: "for maintainable, AI-generated code at scale that we can trust, something has to give… increasing trust and reliability required **constraining the solution space**: specific architectural patterns, enforced boundaries, standardized structures. That means **giving up some 'generate anything' flexibility**." ✅ (martinfowler.com, 17 Feb 2026)
- **OpenAI's own team, quoted by Böckeler**: "Our most difficult challenges now center on **designing environments, feedback loops, and control systems**." ✅ (ibid.)

The tension is that the more capability you grant an agent, the more control apparatus you must build around it — and the control apparatus constrains the capability it was built to enable. There is no published resolution; there is only the arrangement of constraints (§6.4, §12.5) that a given risk appetite accepts.

### 15.5 Hype versus substance

The evidence that this is a real concern comes from unaligned sources:

- **Gartner** (Hype Cycle for Agentic AI, 02 April 2026): in agentic AI, "rapid progress… is exceeded by **hype and confusion**" ✅.
- **Thoughtworks Radar Vol 34** (April 2026): tools "less than a month old — some promising, but ultimately too young to assess," often maintained by one contributor; and the emerging risk of "codebase cognitive debt" ✅.
- **Kief Morris** (Thoughtworks, March 2026) on productivity: "Reports on developer productivity with AI show mixed results, which may be at least partly because of humans spending more time specifying and reviewing code than they save by getting LLMs to generate it." ✅
- **Böckeler on incentive**: "assuming we can trust OpenAI's representation of the success of this (with respect to the author and the team, **OpenAI do have a vested interest in us believing in AI-maintainable code**)" ✅.

**⚠ Where this guide stops and opinion begins.** Claims that "agents will/won't replace engineers," that "prompt engineering is dead," or that a given framework is "the future" are **opinion pieces with audience incentives**, on both sides. This guide takes no position on them. It takes a position only on the *structural* claims that are verifiable: that non-determinism exists, that artifacts change without deploys, that eval root-causes failures, and that verification is under-served.

### 15.6 The evaluation crisis (cross-reference)

The measurement problem — uncalibrated judges, saturated benchmarks, vendor-reported numbers — is treated in §8.5 and is, in this guide's assessment, the discipline's **most consequential open problem**: without trustworthy measurement, none of the other gates (§9) can be trusted either. Tooling depth → [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md).

### 15.7 The vocabulary problem

A discipline that cannot agree on what to call its own practices cannot accredit, teach, or regulate them. Thoughtworks names this **semantic diffusion** ✅ and pairs it with a concrete consequence: "Without shared definitions, it's difficult to determine whether we're seeing distinct techniques or simply different labels for similar ideas" ✅ (Radar Vol 34). This guide's response is structural rather than rhetorical: it defines its terms operationally (§1, §19) and cites the sibling that owns each term's depth rather than re-defining it.

### 15.8 The list of honest unknowns

The unknowns this guide cannot resolve: how well LLM judges correlate with human judgement *in your domain* (does your eval gate mean anything?); whether agents *actually* improve developer productivity net of review time (Morris reports "mixed results" ✅); at what agent count coordination cost exceeds the benefit; where the maturity ceiling for AI-maintained codebases sits (n=1: OpenAI's reported 1M+ LOC over 5 months ✅); and what the correct unit economics of an agent task are (the §14.4 model is qualitative ⚠).

---

## 16. Worked Example: The Cymbal Bank Agent-Platform Team

> **⚠ This worked example is explicitly illustrative and fictional.** Cymbal Bank is a fictional persona, and the team, headcount, gates, and timeline below are **constructed by this guide** to demonstrate how the lifecycle (§4), the craft (§12), the roles (§13), and the maturity model (§14.2) fit together in one narrative. Nothing here is a case study of a real institution, and no figure is a measured benchmark. Technology identifiers use the repo `cyb-` identifier convention.

### 16.1 The starting position

A fictional retail bank — **Cymbal Bank** — has, by the start of the period, run four agent pilots. Two died in demo, one is a brittle rules-plus-prompt hybrid serving internal users, and one — a customer-service triage agent, `cyb-agent-servicing-triage` — is in production with no eval suite, no trace store, and a prompt that lives in a wiki page. That production agent is the forcing function: it works well enough that nobody can switch it off, and badly enough that nobody can defend it in an audit.

The Cymbal platform team's first act is diagnostic, and it produces a one-page maturity assessment: **the practice is at Level 0–1** on the §14.2 model — specs are informal, evals do not exist, and there is nothing between a prompt edit and production.

### 16.2 The team topology

Cymbal adopts the **platform + enabling** topology (§13.4): a five-person `cyb-agent-platform` team owns the harness templates, the permissioned tool registry/MCP gateway, and eval-as-a-service; three embedded domain teams (`cyb-servicing`, `cyb-risk-ops`, `cyb-wealth`) each pair one agent engineer with one eval engineer.

```text
  cyb-agent-platform (5) ── harness templates · tool registry / MCP gateway · eval-as-a-service
        ▲ enabling                    ▲ enabling                  ▲ enabling
  cyb-servicing (1 agent eng + 1 eval eng)   cyb-risk-ops (1+1)   cyb-wealth (1+1)
```

Each embedded team owns its agent and its evals; the platform team owns the harness template and the gate tooling. This is precisely the choice Böckeler anticipates when she asks whether "harnesses… [will] become the new service templates" ✅ — Cymbal bets that they will, and therefore invests in the template rather than in each agent.

### 16.3 The lifecycle gates as the team lived them

| Gate | What Cymbal required | Failure it caught |
|---|---|---|
| **G1 — Framing** | A written spec with non-goals, escalation path, and blast radius (§5.3); a signed "not-a-workflow" justification | Killed a proposed "agentic reconciliation" feature that was a bounded batch job |
| **G2 — Design review** | Tool permission boundaries stated per tool; failure modes enumerated; eval plan named (§6.4) | Forced the servicing agent to lose its account-write tool before launch |
| **G3 — Build** | Prompts, harness config, and tool schemas in the repo; L1 assertions green (§12.3) | Caught a prompt that instructed the agent to "be helpful" with no testable meaning |
| **G4 — Offline eval** | L2 judged eval vs a versioned baseline; no passing case regressed | Caught a "fix" that improved tone but broke a compliance refusal |
| **G5 — Release** | Eval-gated promotion; shadow then canary; rollback target rehearsed (§9.2) | Canary flagged a latency regression from a longer context bundle |
| **G6 — Operate** | Traces on every run; drift monitor; cost per task dashboard (§10.1) | Detected a knowledge-staleness drift after a policy update |
| **G7 — Retire** | Deprecation notice, access revocation, retention record (§11.2) | — (scheduled, not yet exercised) |

### 16.4 The eval and release discipline

The Cymbal teams run the §8 model with a deliberate split:

- **L1** — cheap assertions (schema validity, refusal keywords for regulated topics, no account numbers in output) run on every commit ✅ (cadence per Husain).
- **L2** — judged evals with a **hand-curated, domain-specific suite** for servicing, risk, and wealth, *plus judge calibration against human labels*. The team explicitly rejected a large generic benchmark, following the rule "create an evaluation system specific to your problem" ✅ (Hamel Husain).
- **L3** — canary and post-release online signals, feeding the flywheel: every production failure becomes an eval case before its fix is promoted (§8.4).

Release is a **promotion** of a pinned artifact set, never a bare deploy (§9.1). Rollback is a pointer move, and the team rehearsed it — the versioning guide's core drive, that the fix "must be *re-point a pointer*, not *redeploy code*" ✅ → [agent_versioning_guide.md](agent_versioning_guide.md).

### 16.5 The roles Cymbal hired

| Role (§13.2) | Headcount | Where | First hire rationale |
|---|---|---|---|
| Platform / Ops Engineer | 2 | Platform | Build the paved road and the trace/eval infrastructure |
| Eval / Quality Engineer | 2 | Platform (1) + embedded (1) | Eval is the root cause of failure ✅ — staffed as a role, not a task |
| Agent Engineer | 3 | Embedded (1 per domain) | Domain-context ownership |
| Context / Knowledge Engineer | 1 | Platform | Retrieval corpora, policy knowledge, AGENTS.md conventions |
| AI Product Manager | 1 | Risk-ops | Owns the "why," the blast radius, and change management |

Two deliberate choices: **the eval engineers report to their embedded teams, not to a distant QA org** (so eval cases stay close to failure modes), and **one platform eval engineer holds the guild's standards** (the federated-guild risk in §13.4 is mitigated by giving the guild a *tooling* mandate, not just advisory authority).

### 16.6 The 12-month maturity path (illustrative ⚠)

| Month | Maturity target (§14.2) | Milestones | Evidence produced |
|---|---|---|---|
| **1** | Baseline | Diagnostic; maturity assessment; `cyb-agent-servicing-triage` given a trace store | One-page assessment; first traces |
| **2** | **1 — Defined** | Spec template adopted; prompts moved into repos for all 3 domains | Specs; prompts-as-code |
| **3** | **1 — Defined** | L1 assertion suites green in CI for the production agent | First eval suite; CI pass |
| **4** | **2 — Gated** | Judged (L2) eval suite + judge calibration; eval-gated promotion enforced in CI | Calibration report; first gated release |
| **5** | **2 — Gated** | Design-review gate (G2) standardised; tool permission review for every agent | Permission review records |
| **6** | **2 — Gated** | Rollback drill executed for the servicing agent; all versions pinned | Rollback drill log |
| **7** | **3 — Instrumented** | Drift monitors live; cost-per-task dashboard; SLOs defined (incl. quality SLOs) | Dashboards; SLO doc |
| **8** | **3 — Instrumented** | Incident runbooks; first postmortem published internally | Runbooks; postmortem |
| **9** | **3 — Instrumented** | Harness templates published; second domain adopts the template unchanged | Harness template v1 |
| **10** | **3 → 4** | Flywheel formalised: production failure → eval case → gated fix, on a cadence | Flywheel cadence log |
| **11** | **4 — Self-improving** | Eval suite grows from production signals automatically; regression baseline versioned | Growing eval suite; baselines |
| **12** | **4 — Self-improving** | Retirement runbook exercised on a decommissioned pilot; quarterly maturity review instituted | Retirement record; review cadence |

The path is deliberately **gated before instrumented**: Cymbal refuses to advance to Level 3 until Level 2's promotion gates are real, because instrumenting an agent that has no gate produces exactly the false confidence Thoughtworks warns about.

**What Cymbal got wrong (an honest postscript).** Following this guide's own standard, the illustrative team should not escape critique: month 4 slipped when judge calibration took longer than modelled (two releases shipped on L1-only gates with a documented exception); the platform team initially over-built its first harness template so that two domains forked it — the exact "forking and synchronization" failure Böckeler predicts for service templates ✅ — and reverted to a thinner template in month 9; and **verification remained the weakest area**, the team's confidence resting mostly on L2 judged evals. Its own month-12 assessment: **Level 4 on process, Level 2 on verification.**

---

## 17. Claims Audit

Every load-bearing claim in this guide, with its verdict. **✅ verified** = traced to the cited source this pass; **⚠ flagged** = true-with-caveats, opinion, or synthesis; **❌ unverified** = could not be confirmed and is *not* relied upon.

| # | Claim | Verdict | Basis |
|---|---|---|---|
| 1 | Willison defines "agentic engineering" as developing software with coding agents | ✅ | simonwillison.net, *Agentic Engineering Patterns* |
| 2 | Willison's agent definition: "run tools in a loop to achieve a goal" | ✅ | ibid. |
| 3 | "Vibe coding" coined by Karpathy, Feb 2025, ~3 weeks before Claude Code's release | ✅ | Karpathy via Willison |
| 4 | Agentic engineering is distinguished from vibe coding as reviewed vs unreviewed code | ✅ | Willison |
| 5 | "LLMs don't learn from mistakes, but agents can if you update instructions and harnesses" | ✅ | Willison |
| 6 | Huyen: AI engineering = building apps with foundation models, distinct from ML engineering | ✅ | Huyen, *AI Engineering* (via Wikipedia) |
| 7 | AI engineering "merges aspects of data engineering and software engineering" | ✅ | Wikipedia, AI engineering |
| 8 | The *coinage/provenance* of "agentic engineering" is settled | ⚠ | practitioner-only sources; no academic first-use found |
| 9 | Semantic diffusion of field terms (spec-driven dev, harness engineering) is real | ✅ | Thoughtworks Radar Vol 34 |
| 10 | Anthropic: find the simplest solution; "might mean not building agentic systems at all" | ✅ | Anthropic, *Building effective agents*, 19 Dec 2024 |
| 11 | Workflows vs agents distinction and the five composable patterns | ✅ | ibid. |
| 12 | Tool design deserves HCI-level effort (ACI); SWE-bench: more time on tools than prompt | ✅ | ibid. |
| 13 | The Anthropic post's *tooling list* is current | ⚠ | post carries an editor's note that tooling changed |
| 14 | "A failure to create robust evaluation systems" is the common root cause of product failure | ✅ | hamel.dev, *Your AI Product Needs Evals* |
| 15 | Three eval levels (unit / model+human / A/B) with cost-ordered cadence | ✅ | ibid. |
| 16 | "Don't rely on generic evaluation frameworks… create an evaluation system specific to your problem" | ✅ | ibid. |
| 17 | The eval flywheel; tests must be updated from observed failures | ✅ | ibid. |
| 18 | Prompts/context change more often than surrounding code | ✅ | LangChain, via [agent_versioning_guide.md](agent_versioning_guide.md) |
| 19 | Anthropic model lifecycle has four states with ≥60 days' retirement notice | ✅ | Anthropic docs, via [agent_versioning_guide.md](agent_versioning_guide.md) |
| 20 | Harness = context engineering + architectural constraints + codebase "garbage collection" | ✅ | Böckeler, martinfowler.com, 17 Feb 2026 |
| 21 | OpenAI team: no manually typed code; ~1M LOC in ~5 months; harness took 5 months | ✅ | Böckeler's account of OpenAI's write-up (second-hand — see #22) |
| 22 | The OpenAI write-up's own primary text matches Böckeler's summary | ⚠ | read the memo, not the original OpenAI piece |
| 23 | "What I am missing… is verification of functionality and behaviour" | ✅ | Böckeler, 17 Feb 2026 |
| 24 | Trust at scale required *constraining the solution space* (flexibility given up) | ✅ | Böckeler, 17 Feb 2026 |
| 25 | "On the loop" vs "in the loop": change the harness, not the artefact | ✅ | Morris, martinfowler.com, 4 Mar 2026 |
| 26 | Developer-productivity reports with AI show "mixed results" | ✅ | Morris, 4 Mar 2026 |
| 27 | Gartner: agentic AI's progress is "exceeded by hype and confusion" | ✅ | Gartner, Hype Cycle for Agentic AI, 02 Apr 2026 |
| 28 | "Only 17% have deployed" agentic AI | ❌ | secondary blog; not traceable to the primary Gartner doc |
| 29 | "Codebase cognitive debt" as an AI-era risk | ✅ | Thoughtworks Radar Vol 34 |
| 30 | "Securing permission-hungry agents" is a Radar theme | ✅ | Thoughtworks Radar Vol 34 |
| 31 | Coding assistants expand the software supply-chain attack surface | ✅ | Gumbley & Ryan, martinfowler.com, 13 May 2025 |
| 32 | Whether TDD-inside-the-agent-loop improves output is settled | ⚠ | an informal Thoughtworks evaluation exists; its conclusion was not read this pass |
| 33 | Specific job-market growth/salary figures for agentic roles | ❌ | no primary labour-market source verified this pass |
| 34 | A professional body or meaningful certification exists | ⚠ | absence of evidence found; asserted as *absent* |
| 35 | The seven-phase lifecycle (spec→retire) is a published standard | ⚠ | this guide's synthesis (§4.1) |
| 36 | "Quality SLOs" as a settled standard alongside latency/error/cost SLOs | ⚠ | practitioner framing, not a published standard |
| 37 | Any exact per-agent-task cost benchmark | ❌ | no reliable benchmark verified this pass |

**Counts: 27 ✅ verified · 7 ⚠ flagged · 3 ❌ unverified.** The ❌ rows are listed precisely *because* they were tempting and unverifiable; none of them is load-bearing in the guide's argument.

---

## 18. What Could Not Be Verified

This section exists so that the guide's gaps are explicit rather than hidden. Each item below was searched for and **could not be confirmed to a primary source** this pass. Do not treat any of them as fact without independent verification.

- **The provenance-first-use of "agentic engineering."** No academic paper, standards document, or dated first-use was located. The best available attribution is Willison's guide (§1.2) ⚠.
- **Any quantitative job-market evidence** — posting-growth percentages, salary bands, or demand surveys for agentic/AI engineering roles. Not verified; the entire §3.1 is qualitative as a result.
- **The existence of a professional engineering body or a certification with employer recognition.** Not found; §3.3 states the absence, which is itself an absence-of-evidence claim ⚠.
- **The Gartner "17% have deployed" figure.** Attributed to Gartner by secondary blogs; not visible in the primary document summary this pass. **Omitted from the guide's claims** ⚠.
- **The conclusion of Thoughtworks' TDD-in-the-agent-loop evaluation.** The memo was confirmed to exist; its findings were not read, so §12.2 flags the question as open rather than answering it.
- **The original OpenAI "Harness engineering" write-up.** Read only through Böckeler's summary (§7.2, §13.4); the primary text was not extracted this pass. The 1M-LOC / 5-month figures are therefore second-hand ⚠.
- **Any per-agent-task cost benchmark** or verified unit-economics model. §14.4 and §15.3 are qualitative by necessity.
- **A canonical published lifecycle for agentic engineering.** §4's seven phases are this guide's synthesis; no single published framing spans spec→retire ⚠.
- **Whether AI-assisted development improves net developer productivity.** Kief Morris reports "mixed results" ✅; the net effect remains unmeasured.
- **The role definitions, skills matrix, team topologies, and hiring practice in §13, and the maturity model and worked example in §14–§16.** These are **constructed** from the lifecycle; they are proposals, not findings, and are flagged as such throughout ⚠.

**Method note.** Where a search returned nothing (rate-limited retrievals occurred this pass), fallbacks were used: direct extraction of primary URLs (simonwillison.net, hamel.dev, anthropic.com, martinfowler.com, thoughtworks.com/radar, gartner.com summaries, wikipedia.org). No claim was manufactured to fill a gap.

---

## 19. Glossary and References

### 19.1 Glossary

| Term | Working definition (this guide's usage) |
|---|---|
| **Agent** | Software that calls an LLM with a prompt and tool definitions, runs the tools the model requests, and loops toward a goal ✅ (Willison) |
| **Agent-computer interface (ACI)** | The design surface of a tool *as the model experiences it* — names, descriptions, parameters, error surfaces ✅ (Anthropic) |
| **Agentic engineering** | The discipline of specifying, building, evaluating, releasing, operating, and retiring LLM-driven agentic systems (§1.1) |
| **Context engineering** | The discipline of deciding what enters the model's context window, when, and how it is curated → [context_engineering_guide.md](context_engineering_guide.md) |
| **Eval-as-spec** | Encoding a spec's success criteria as runnable evaluations; the Phase I exit condition (§5.3) |
| **Eval-driven development** | Writing the evaluation before/with the feature and letting it drive iteration (§8.1) |
| **Eval theatre** | A suite of easy assertions that always pass; detected because the model never struggles (§12.7) |
| **Flywheel (eval / agentic)** | Production traces → eval cases → spec updates → gated releases, on a cadence (§8.4; Morris ✅) |
| **Harness** | The specifications, quality checks, workflow guidance, and tooling that constrain an agent ✅ (Morris) |
| **Harness engineering** | Building and maintaining the harness as the human's "on the loop" activity ✅ (Morris; Böckeler) |
| **LLM-as-judge** | Using a model to score another model's output; requires calibration against human labels (§8.5) |
| **Non-determinism** | The property that identical inputs can yield different behaviour/runs; the root of the discipline's adaptations (§15.2) |
| **On the loop / in the loop** | Whether a human *builds the harness* (on) or *inspects each artefact* (in) ✅ (Morris) |
| **Prompt-as-code** | Treating prompts as versioned, reviewed repository artifacts (§7.1) |
| **Regression baseline** | The versioned eval result set a candidate is compared against at the release gate (§9.2) |
| **Semantic diffusion** | The spread of terms before their meanings stabilise ✅ (Thoughtworks Radar) |
| **Spec** | The written problem statement, non-goals, escalation path, and success criteria (§5.3) → [spec_driven_development_for_llms_guide.md](spec_driven_development_for_llms_guide.md) |
| **Vibe coding** | Prompting LLMs to write code without reviewing it; the low-discipline end of the scale ✅ (Karpathy via Willison) |
| **Workflow** | LLM-and-tool orchestration through predefined code paths; the non-agent option ✅ (Anthropic) |

### 19.2 References and further reading

**Primary practitioner and vendor engineering sources (accessed this pass):**

1. Simon Willison, *Agentic Engineering Patterns* — "What is agentic engineering?" and "Writing code is cheap now" — simonwillison.net/guides/agentic-engineering-patterns/
2. Anthropic, *Building effective agents* (19 Dec 2024) — anthropic.com/engineering/building-effective-agents
3. Hamel Husain, *Your AI Product Needs Evals* — hamel.dev/blog/posts/evals/
4. Birgitta Böckeler, *Harness Engineering — first thoughts* (17 Feb 2026) — martinfowler.com/articles/exploring-gen-ai/harness-engineering-memo.html
5. Kief Morris, *Humans and Agents in Software Engineering Loops* (4 Mar 2026) — martinfowler.com/articles/exploring-gen-ai/humans-and-agents.html
6. Thoughtworks, *Technology Radar Vol 34* (April 2026) — thoughtworks.com/radar
7. Gartner, *Hype Cycle for Agentic AI* (02 Apr 2026) — gartner.com/en/documents/7671861
8. Jim Gumbley & Lilly Ryan, *Coding Assistants Threaten the Software Supply Chain* (13 May 2025) — martinfowler.com/articles/exploring-gen-ai/software-supply-chain-attack-surface.html

**Books and reference framing:**

9. Chip Huyen, *AI Engineering: Building Applications with Foundation Models* (O'Reilly, 2025)
10. Wikipedia, *Artificial intelligence engineering* — en.wikipedia.org/wiki/Artificial_intelligence_engineering
11. Matthew Skelton & Manuel Pais, *Team Topologies* (topology vocabulary used in §13.4) ⚠ extrapolated

**Sibling guides in this cluster (depth on each surface):** [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) · [agent_scaffolding_guide.md](agent_scaffolding_guide.md) · [context_engineering_guide.md](context_engineering_guide.md) · [agent_versioning_guide.md](agent_versioning_guide.md) · [agentops_guide.md](agentops_guide.md) · [ai_agent_drift_guide.md](ai_agent_drift_guide.md) · [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) · [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md) · [spec_driven_development_for_llms_guide.md](spec_driven_development_for_llms_guide.md) · [agentic_solution_artifacts_guide.md](agentic_solution_artifacts_guide.md) · [agents_work_fall_apart_guide.md](agents_work_fall_apart_guide.md) · [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) · [autonomous_agents_guide.md](autonomous_agents_guide.md) · [hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md) · [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md) · [multi_agent_banking_guide.md](multi_agent_banking_guide.md) · [ai_governance_framework_guide.md](ai_governance_framework_guide.md) · [prompt_injection_guide.md](prompt_injection_guide.md) · [ai_red_teaming_guide.md](ai_red_teaming_guide.md) · [ai_adoption_strategies_guide.md](ai_adoption_strategies_guide.md) · [ai_agent_platform_selection_guide.md](ai_agent_platform_selection_guide.md) · [ai_engineer_roadmap_2026.md](ai_engineer_roadmap_2026.md)

---

**The discipline, in one line:** the model is borrowed, the intelligence is emergent, and the verification is ours — which is why the job is no longer writing the code but *engineering the agent*, honestly and accountably, as the engineered agent.
