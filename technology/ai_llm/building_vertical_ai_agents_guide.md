# Building a Vertical AI Agent — the model is the least vertical thing about it

> **Byline:** Jack Liu Shurui, Solution Architect  
> **Repo:** personal research library · **Series:** LLM/AI Engineering & Operations · **Domain:** AI Engineering · Vertical Agents  
> **Date:** October 2026 · **Sources read:** 2026-10-11

> **Standfirst.** A vertical AI agent is an agent whose domain, data, workflow, evaluation, buyer and liability are all specific to one industry — and this guide owns exactly one question about it: *what changes when the agent is vertical?* It is deliberately **not** another how-to-build-an-agent guide. The loop, the harness, the scaffold, the context budget, the tools, the evaluation mechanics and the operations are already owned by a large and specific shelf in this repository, and every one of them is **cited here by name and not restated**. What is unowned is the vertical *framing* itself: what a domain corpus is and why it is the real moat, who is permitted to decide what "correct" means in a domain, how a domain's asymmetric errors enter an acceptance test, how the agent embeds in a process it does not own, and what liability a regulated vertical imports. The phrase **"vertical AI" is a framing, not received terminology**: it is current in venture and vendor material and has one single-author arXiv preprint that *attempts* standardisation, but no peer-reviewed definition and no standards-body usage. This guide treats it as an organising frame — following this repository's own precedent in `agent_experience_guide.md` for a term coined before its canon — and says plainly where the evidence is thin.

> **How to read it.** Section 1 handles the term honestly, gives the decoder, states the thesis and declares the boundary against the shelf by name. Section 2 lays out the axes on which a vertical agent differs from a general one, and separates *vertical* from *AI-native* — two different axes that get blurred. Section 3 develops the domain corpus as the moat: what proprietary data concretely is, how it is obtained and prepared, and its honest limits. Section 4 — the central and longest section — is the ground truth problem: who decides what correct is, the expert bottleneck, the golden set, inter-annotator disagreement, adjudication, and asymmetric error. Section 5 is evaluation by *domain* standards, cross-referenced to the evaluation shelf for mechanics. Section 6 is the workflow embedding. Section 7 is the build decision in the vertical's terms — buy-vs-build, model, retrieval, eval — each cross-referenced to the guide that owns its mechanics and never re-derived. Section 8 is domain integration against systems of record. Section 9 is liability and the professional duty. Section 10 is the economics and the buyer, structural and figureless, cross-referenced to the AI-native guide for the economics frame. Sections 11–13 are the failure modes, a readiness assessment, and the Cymbal Bank worked example. Sections 14–16 are the anti-patterns, the claims audit, and the honest negatives, glossary and closing. Every source carries who, when and of what kind.

> **Verification policy.** Facts marked **(verified)** were checked against a primary source during writing (October 2026) — a peer-reviewed paper, an established reference work, a specification, or a vendor's own documentation. Everything else carries a label: **(vendor claim)** for a statement a vendor makes about its own product or practice; **(flagged)** where a claim is repeated but not traceable to a measurement; **(unverified-flag)** where a page could not be extracted or a source could not be resolved; **(negative finding)** where a search or a repository pass established that something does *not* appear in the sources checked. Named companies appear only as dated factual illustrations, never ranked and never recommended; a company's claim about its own product is that company's claim, attributed and dated. **No market size, growth rate, funding figure, adoption percentage or valuation is printed anywhere in this guide**, and no vertical is crowned best. Where a rule or a jurisdiction's requirement could not be sourced, it is *marked unverified rather than stated*. No citation in this guide is written from memory.

---

**Table of contents**

1. [The Overview — the Term, the Decoder, the Thesis, and the Boundary Declared](#1-the-overview--the-term-the-decoder-the-thesis-and-the-boundary-declared)
2. [What Makes an Agent Vertical](#2-what-makes-an-agent-vertical)
3. [The Domain Corpus as the Moat](#3-the-domain-corpus-as-the-moat)
4. [The Ground Truth Problem — Who Decides What Correct Is](#4-the-ground-truth-problem--who-decides-what-correct-is)
5. [Evaluation by Domain Standards](#5-evaluation-by-domain-standards)
6. [The Workflow Embedding](#6-the-workflow-embedding)
7. [The Build Decision, in the Vertical's Terms](#7-the-build-decision-in-the-verticals-terms)
8. [The Domain Integration](#8-the-domain-integration)
9. [Liability and the Professional Duty](#9-liability-and-the-professional-duty)
10. [The Economics and the Buyer](#10-the-economics-and-the-buyer)
11. [The Failure Modes of Vertical Agents](#11-the-failure-modes-of-vertical-agents)
12. [A Vertical-Agent Readiness Assessment](#12-a-vertical-agent-readiness-assessment)
13. [The Cymbal Bank Worked Example](#13-the-cymbal-bank-worked-example)
14. [The Anti-Patterns](#14-the-anti-patterns)
15. [The Claims Audit — Verified, Flagged, Rejected](#15-the-claims-audit--verified-flagged-rejected)
16. [What Could Not Be Verified, Glossary, Cross-References and Closing Summary](#16-what-could-not-be-verified-glossary-cross-references-and-closing-summary)

---

## 1. The Overview — the Term, the Decoder, the Thesis, and the Boundary Declared

### 1.1 The question this guide owns

Strip the buzzwords away and a vertical AI agent is an agent that works inside one industry. It reads a domain's documents, respects a domain's rules, sits in a domain's process, is judged by a domain's experts, is bought by a domain's budget-holder, and — if it is wrong — exposes its owner to that domain's liability. Every one of those clauses is a *consequence of being vertical*, and the honest question is not "how do I build an agent" but **"what does verticality change about how the agent is built, evaluated, embedded and sold?"**

This guide owns exactly that question and nothing around it. It is deliberately not a guide to the agent loop, the harness, the scaffolding, memory, tools, context assembly, evaluation tooling, or production operations. Each of those is a large, mature discipline already covered elsewhere in this library, and §1.6 names the specific guide that owns each one. What this guide holds is the vertical *delta* — the things that are different because the domain is specific.

### 1.2 The term: "vertical AI" in published use

The phrase has a short and thin published record, and an honest guide states its status before building on it.

**What exists (all verified from the primary page during writing):**

- **arXiv:2501.00881** — Fouad Bousetouane, *"Agentic Systems: A Guide to Transforming Industries with Vertical AI Agents"*, submitted **1 January 2025**, 31 pages, cs.MA, **single author**. Its abstract says the work "attempts to define a level of standardization for Vertical AI agent design patterns" and proposes a **"Cognitive Skills Module"** for "domain-specific, purpose-built inference capabilities" ✅ (abstract read at arxiv.org/abs/2501.00881, 2026-10-11). This is a **preprint**: not peer-reviewed, not a standards body, and it *attempts* definition rather than producing an accepted one.
- **IBM Think topic page**, "What are vertical AI agents?" — a **vendor/editorial definitional page**. It defines vertical AI agents as "specialized artificial intelligence systems designed to perform specific tasks or functions within a particular industry," and states in its own words that they are "sometimes called **domain-specific agents or function-focused agents**" ✅ (page read at ibm.com/think/topics/vertical-ai-agents, 2026-10-11). That a vendor's own definitional page needs to equate the term with two other phrases is itself the evidence that the term is unsettled.
- **a16z Big Ideas 2026** — an entry titled *"Vertical AI evolves from information retrieval and reasoning to multiplayer"* (Alex Immerman), catalogued in this repository at `technology/a16z_big_ideas_2026_guide.md` §3.4 item 8 ✅. This is a **venture essay**; it cites sector growth claims, and **no figure from it is printed here**.
- A **cluster of venture and vendor marketing pages** (botpress, monday.com, CRV's "Why Domain-Specific AI Wins", hyscaler, decasoftsolutions, emergentmind) use the term loosely ⚠ — marketing and VC material, not peer-reviewed, not a taxonomy.

**The verdict.** The phrase is current in venture and vendor material and has one arXiv preprint attempting standardisation, but there is **no peer-reviewed definition and no standards-body usage**. So "vertical AI" is a **framing, not received terminology**. This guide presents it explicitly as such. It does **not** manufacture a canon, does not claim a taxonomy is settled, and does not treat a vendor's definitional page as a definition. This is the same posture `agent_experience_guide.md` took when it found "agent experience" coined before its canon, and the reader should expect the same discipline here. **Contrary evidence note:** no peer-reviewed venue, standards body or stable industry taxonomy using the phrase was found in this pass; if one appears, the framing should be re-anchored to it.

### 1.3 The decoder

Terms used throughout, defined once so the rest of the guide can be precise.

- **Horizontal agent** — an agent that works across many domains, with no single industry's rules, corpus or process; a general assistant, a general coding agent, a general research agent. Its advantage is breadth.
- **Vertical agent** — an agent whose function is defined by one industry or one business function, and which is measured against that domain's own standard of correctness.
- **Domain corpus** — the body of documents, records, cases, logs and artefacts that a domain's expertise is made of, and that a vertical agent must read, retrieve from or be trained on. §3.
- **Ground truth** — the answer that is *correct for this domain*, as decided by whoever holds the authority to decide it. §4.
- **Golden set** — the curated, adjudicated set of inputs with their correct outputs, against which a vertical agent is accepted. §4.5.
- **Expert in the loop** — the domain professional whose judgement defines correctness and who must therefore be inside the build, not consulted at the end. §4.2.
- **Workflow embedding** — the vertical agent's placement inside a business process it does not own, attached to systems of record, handing off to humans and other systems. §6.
- **System of record** — the authoritative store of a process's true state (a core banking ledger, a CRM, an ITSM system), defined here only as a pointer: this repository owns the concept elsewhere (§1.6).
- **The moat** — the durable, defensible advantage a vertical agent holds. This guide's thesis is that it is the ground truth, not the model. §1.4, §3, §4.
- **Assistance vs advice** — the regulatory-relevant distinction between surfacing information to a professional and giving advice to a client; developed in §9, and asserted of no jurisdiction's rule without a source.

### 1.4 The thesis

**A vertical agent's moat is not the model. It is the ground truth nobody else has.**

The argument for it is developed across the whole guide and summarised here so the reader can hold it sceptically:

1. **The model layer is shared and improving.** The foundation model a vertical agent calls is, with few exceptions, the same model its competitor calls. Capability at that layer is commoditised and rising for everyone — which means it is a *falling*, not a rising, basis for advantage (§3, §7).
2. **What is not shared is the domain's corpus and its adjudicated correctness.** The documents, the cases, the workflow logs and — above all — the *decisions about what counts as right* are specific to a firm, a regulator or a profession, and are expensive to assemble and hard to copy (§3, §4).
3. **The embedding is where the value is realised and where the switching cost lives.** An agent wired into a process and a system of record is defended by the cost of removing it, not by its benchmark (§6, §8).

**The honest counter-argument, stated in full because the thesis must survive it:** a foundation-model improvement can erase a domain-data advantage overnight. If a general model learns a domain's public corpus, or learns to reason well enough on the domain from general knowledge, then a vertical built only on *public* domain data has no moat at all — the next model release swallows it. Verticals whose ground truth is shallow, public, or easily licensed by a competitor may sustain **no moat**. The thesis is therefore not "verticals always win"; it is the sharper claim that **a vertical's moat exists exactly to the extent that its ground truth is proprietary, adjudicated, and embedded in a workflow** — and each of those three can fail independently. Where this guide finds the evidence thin, it says so (§3.6, §4.7, §10.6).

### 1.5 The coverage finding (dedup)

A repository pass run for this guide establishes the gap it fills. Counting `.md` files matching each term (case-insensitive, whole repo):

| Term | Files matching | Reading |
|---|---|---|
| `vertical AI` | **1** | only `technology/a16z_big_ideas_2026_guide.md` (the venture entry) |
| `vertical agent` | **0** | the exact phrase is absent from the library |
| `vertical SaaS` | 2 | `banking/airwallex_guide.md`, `technology/singapore_saas_companies_guide.md` — both use it in passing |
| `domain-specific agent` | 3 | `llm_agents_quantitative_finance_guide.md`, `llm_agent_use_cases_finance_guide.md`, `coding_agents_research.md` |
| `domain expert` | 43 | widely present, but as a *role*, never as a section subject |
| `ground truth` | 63 | widely present as an evaluation *mechanic*, never as a domain-authority problem |
| `golden dataset` | 15 | present as an eval *artefact*, never tied to expert adjudication |

The reading is unambiguous. **The build half of this topic is owned by a large shelf** — "agent loop", "agent eval", "agent architecture", "scaffold" and "harness" together appear across many files. "Ground truth" and "golden dataset" appear often, but always as *mechanics* of evaluation. **No guide owns the question of what changes when the agent is vertical.** A filename search confirms **no `building_vertical*` guide exists** (0 matches). This guide is that file, and only that file.

### 1.6 The boundary, declared by name

This guide borrows mechanisms; it does not restate them. Each is owned by the named guide, and the reader should go there for the mechanism and return here only for the vertical consequence.

| Mechanism | Owned by (this repository) | What this guide borrows and does **not** re-derive |
|---|---|---|
| Harness / scaffold | `technology/ai_llm/agent_harness_engineering_guide.md`, `agent_scaffolding_guide.md` | that a harness exists and what it does; cited, not rebuilt |
| The loop itself | `technology/ai_llm/agent_loop_implementation_guide.md` | the control loop |
| Context assembly | `technology/ai_llm/context_engineering_guide.md` | what goes in the window; §3 borrows it for *corpus selection* |
| Tools & protocols | `technology/ai_llm/mcp_discovery_guide.md`, `mcp_framework_tools_guide.md`, `mcp_progressive_disclosure_guide.md` | tool exposure; §8 borrows it for *systems integration* |
| Operations | `technology/ai_llm/agentops_guide.md`, `agent_versioning_guide.md`, `agent_sandboxing_strategies_guide.md`, `agent_runtime_cache_design_guide.md` | run, version, isolate, cache |
| Production reality | `technology/ai_llm/production_ready_llm_agents_guide.md`, `llm_agents_failures_production_guide.md`, `agents_work_fall_apart_guide.md` | that agents fail in production and how |
| Posture (ask/recommend/act/wait) | `technology/ai_llm/agent_decision_policy_guide.md` | the posture taxonomy; §6 cites it and does **not** re-derive it |
| Designing for agent consumers | `technology/ai_llm/agent_experience_guide.md` | the coined-term precedent and agent-facing design |
| Evaluation mechanics | `technology/ai_llm/llm_evaluation_frameworks_guide.md`, `llm_evaluation_vs_validation_guide.md` | metrics, harnesses, validation vs evaluation |
| Retrieval & RAG eval | `technology/ai_llm/rag/rag_evaluation_methodology_guide.md`, `rag_evaluation_tools_comparison_guide.md`, `production_grade_agentic_rag_guide.md`, `advanced_rag_techniques_guide.md` | retrieval quality and its measurement |
| AI-native businesses (a **different axis**) | `technology/ai_native_companies_guide.md` | the AI-native economics frame; §10 cites it — the two axes are **not** blurred |
| Domain projects & long tail | `technology/ai_llm/real_world_ai_projects_guide.md`, `long_tail_domains_guide.md` | see the adjacent-axis note below |
| Systems of record | `banking/posting_engine_core_banking_guide.md`, `banking/core_banking_processes_guide.md`, `banking/data_models_banking_insurance_guide.md`, `banking/fix_protocol_guide.md` | what a system of record *is*; §6/§8 cite, not re-derive |
| Systems-of-record claim (dated) | `technology/a16z_big_ideas_2026_guide.md` §7 (Sarah Wang) | the dated venture claim; quoted, not adopted |

**Two adjacent framings, named so they are not duplicated.** `technology/ai_llm/long_tail_domains_guide.md` owns the **head/tail of domains** (how rare a domain is in pre-training), which is a *different axis* from vertical/horizontal (how specific a deployment is to an industry); its §12 explicitly sets aside adjacent framings, and this guide does not re-open that question. `technology/domain_driven_design_guide.md` owns the DDD/bounded-context view, and `technology/maritime_domain_awareness_guide.md` is a worked domain-awareness case — both adjacent, neither duplicated here. `technology/ai_llm/real_world_ai_projects_guide.md` (1277 lines, 12 sections) is the repository's project walkthrough shelf and is cited at that exact path, not at the repo root.

### 1.7 What this guide is not

It is not a claim that vertical beats horizontal. It is not a market report. It is not a product comparison. It does not claim a specific vendor, model or vertical is best. It does not state any jurisdiction's regulatory rule without a source, and where none was found it says so in §9 and §16. It is the guide that asks what verticality *changes*, answers it section by section, and marks every claim with who said it, when, and of what kind.

---

## 2. What Makes an Agent Vertical

"Vertical" is not one property; it is a bundle of six, and a deployment can be vertical on some axes and horizontal on others. Naming the axes is what stops "vertical" from becoming a marketing word.

### 2.1 The six axes

| Axis | Horizontal agent | Vertical agent | Where it bites |
|---|---|---|---|
| **Domain** | many fields, no single rulebook | one field's rules, terminology and precedent | correctness is judged by the field's own standard (§5) |
| **Data** | public / general corpus | a domain corpus, often proprietary (§3) | the moat, if the corpus is defensible (§3) |
| **Workflow** | invoked ad hoc, stands alone | embedded in one process, attached to a system of record (§6) | the switching cost, and the integration bill (§8) |
| **Evaluation** | generic benchmarks | the domain's own golden set and error asymmetry (§4, §5) | you cannot accept what you cannot measure in-domain |
| **Buyer** | a developer or a general tooling budget | a domain budget-holder: a compliance head, a claims director, a partner (§10) | adoption, pricing, and what "good" means to whom pays |
| **Liability** | the user's problem | possibly the vendor's problem, in a regulated domain (§9) | design and insurance, not just accuracy |

### 2.2 The axes are independent, and most real agents are mixed

A vertical agent may be vertical on **domain** and **workflow** while being horizontally *sourced* on **model** (it calls a general foundation model). That is the norm, not an exception: the model is usually the *least* vertical component (§7.2). Conversely an agent can be vertical on **data** (a proprietary corpus) yet horizontal on **workflow** (a chat surface) — and that combination is exactly the one the honest counter-argument targets, because a data-only vertical with no workflow embedding is the easiest to erase with a model upgrade (§3.6).

The useful discipline is to score a proposed agent on all six axes and find the axes on which it is *actually* differentiated. Usually it is one or two, and the rest are inherited.

### 2.3 Vertical is **not** AI-native — two different axes

This is the blur this guide most wants to prevent. **AI-native** describes *how a business or product is built* — around AI as the core, with AI-shaped economics, org design and data loops; it is owned as a business concept by `technology/ai_native_companies_guide.md`. **Vertical** describes *how specific a product is to an industry*. The two are orthogonal:

- A company can be **AI-native and horizontal** (a general AI product, built AI-first).
- A company can be **vertical and not AI-native** (a long-standing industry software vendor that adds an agent).
- A company can be **both** (an AI-first company that sells into one industry).
- A company can be **neither** (a traditional general-purpose ISV).

Blurring them causes a specific error: treating "AI-native" economics — data flywheels, usage-based pricing, fast iteration — as automatic advantages of being vertical. They are not. A vertical with no defensible corpus has no flywheel no matter how AI-native its engineering. §10 cites the AI-native guide for the *economics* frame and keeps the two axes apart.

### 2.4 The consequence

If verticality is a bundle, then "build a vertical agent" is under-specified. The buildable statement is "be vertical on **these** axes, with **this** corpus, judged by **this** standard, embedded in **this** process, sold to **this** buyer, under **this** liability." The rest of this guide works each clause. The next section takes the clause the thesis rests on: the corpus.

---

## 3. The Domain Corpus as the Moat

If the moat is not the model, the first candidate is the data. This section says what "proprietary domain data" concretely is, how it is obtained, how it is prepared, and — at length, because the thesis depends on it — the ways the corpus *fails* to be a moat.

### 3.1 What "proprietary domain data" concretely is

"Data" is too vague to reason about. A vertical agent's corpus is made of specific artefact classes, and they differ enormously in defensibility:

1. **Published domain knowledge.** Standards, statutes, guidance, textbooks, professional literature, public filings. *Readily available* — the same documents the competitor reads. This is the **weakest** basis for a moat: a general model may already contain much of it, and anyone can license it.
2. **Licensed proprietary content.** Paywalled case law, proprietary research, market data feeds, standardised product data. *Available to whoever pays*; the question is whether the licence is **exclusive** (rare) or **non-exclusive** (usual), because a non-exclusive licence that a competitor can also buy is not a moat (§3.6).
3. **In-house documents and rules.** A firm's own policies, precedents, manuals, decision records. *Genuinely firm-specific* and, if well-curated, defensible — the ceiling on how much of it exists is a real consideration (§3.5 footnote on the long-tail guide).
4. **Workflow logs and outcome traces.** What the firm's own process actually did — the cases, the dispositions, the overrides, the outcomes. This is the closest thing to a **proprietary ground-truth generator** (§4.4), and it is the artefact class this guide ranks highest, with the caveat that a log records what *happened*, not what was *correct*.
5. **Expert-labelled data.** The output of §4's expert process: inputs paired with adjudicated correct answers. The most expensive to make and the most defensible once made.

The defensibility ranking, roughly, runs 5 ≈ 4 > 3 > 2 > 1. Most verticals start at the bottom of that list and claim a moat from the top; the honest test is **where the deployment's advantage actually sits on this ladder.**

### 3.2 How a corpus is obtained

Four routes, each with a different cost structure and a different defensibility profile.

- **Licensed.** Purchase a non-exclusive or (rarely) exclusive right to use proprietary content. *Fast, expensive, usually non-exclusive* — and a non-exclusive licence is a moat only if you *do something with the data a competitor cannot* (curate it, adjudicate it, structure it). The licence buys the raw material, not the edge.
- **Generated in-house.** Build the corpus from the firm's own documents and decisions. *Slow, cheap in licence cost, high in curation cost* — and defensible precisely because it is not purchasable.
- **Harvested from workflow logs.** Instrument the process and capture its traces (see the operations shelf — `agentops_guide.md`, `agent_runtime_cache_design_guide.md` — for how traces are captured; this section takes only the *vertical* consequence). *Continuous, cheap to collect, hard to label* — the log is a signal source, not a labelled set.
- **Purchased / crowdsourced.** Buy a dataset or commission annotation. *Quick, and the weakest moat* — the same purchase is available to a competitor, and purchased annotation rarely encodes *this domain's* notion of correctness.

A vertical's realistic strategy is a **stack**, not a single route: a licensed base, an in-house layer, a workflow-log loop, and an expert-labelled golden set (§4.5). No single route alone produces a defensible corpus.

### 3.3 How a corpus is prepared

Raw artefacts are not a corpus. The transformation is where most of the effort — and most of the defensibility — actually is:

- **Normalisation.** Reconciling the domain's messy surface forms to stable entities and units. Borrow the context-assembly mechanics from `context_engineering_guide.md`; the *vertical* point is that a domain's normalisation rules are themselves proprietary knowledge (what counts as the same counterparty, the same instrument, the same diagnosis code).
- **Structure extraction.** Turning documents into retrievable structure — chunking, metadata, linking (mechanics in `advanced_rag_techniques_guide.md` and `production_grade_agentic_rag_guide.md`; cited, not re-derived).
- **Labelling.** Attaching correct answers. This is §4's subject and the expensive part.
- **Provenance.** Recording *where every artefact came from, when, under what licence, and who asserted it correct.* Provenance is not bookkeeping; it is what makes the corpus auditable (§9) and what lets you re-derive ground truth when the domain changes. A corpus without provenance is a corpus you cannot defend.

The vertical point: the *preparation rules* are domain knowledge, and they are frequently the thing that is actually proprietary — more so than the raw documents.

### 3.4 Why the corpus, not the model, is the candidate moat

Three properties make a corpus a better candidate for durable advantage than a model:

1. **It is not shared.** A competitor calling the same foundation model gets the same model; they do not get your adjudicated corpus.
2. **It improves with use, in-domain.** A workflow-log loop can grow the corpus as the process runs, which a shared model's improvement does not do *for your domain specifically*.
3. **It is expensive to replicate.** Assembling and adjudicating a domain corpus is skilled, slow work that a competitor cannot shortcut by buying more compute.

But every one of those three has a failure mode, and the thesis requires they be stated.

### 3.5 The honest limit — corpora that decay

A domain corpus is not a fixed asset; it rots. Statutes change, guidance is superseded, a firm's policy is revised, a standard is re-versioned. A corpus that is accurate in year one can be **actively misleading** in year three, and — worse for a conservative vertical — *confidently wrong in a way the agent will not flag*, connecting directly to the silent-failure material in `llm_agents_failures_production_guide.md` and `agents_work_fall_apart_guide.md`. Decay is why provenance (§3.3) and dated artefacts matter: a corpus without dates cannot tell you which of two conflicting documents is current. The vertical consequence is that a corpus is a **maintained process**, not a one-time acquisition, and the maintenance cost is a permanent expense against the moat.

### 3.6 The honest limit — corpora a competitor can license too, and duplicated corpora

This is the failure the honest counter-argument names first:

- **The non-exclusive licence is not a moat.** If the differentiating content is a non-exclusive licence to a public-ish dataset, a competitor buys the same licence tomorrow. The licence may still be *necessary* (you cannot build without it), but it is not *sufficient* for advantage.
- **Duplicated corpora.** Two verticals that both scrape the same public sources, both license the same feeds, and both train on the same papers have **the same corpus** — and therefore no data moat. Duplication is the common case, not the exception, precisely because the easy corpora are the ones everyone can get.
- **The public-corpus vertical.** A vertical whose entire corpus is public is the exact target of the counter-argument: a future foundation model trained on more of the public web may absorb the advantage outright. The model is shared *and improving*; a public corpus is shared *and static*. That is a losing combination.
- **The model may simply learn the domain.** Where a domain's knowledge is public, better general reasoning can substitute for a curated corpus. The claim that a curated corpus always beats a better model is **not established**; it holds only where the corpus is genuinely proprietary and the task genuinely requires it (A caveat marked ⚠ — this is this guide's reasoning, not a published result).

### 3.7 Where the corpus argument leaves the thesis

The corpus is a *candidate* moat, and it is a real one **only when it is proprietary (not licensable by a competitor), adjudicated (labelled with correct answers, not just collected), maintained (not decaying), and embedded (used inside a workflow whose switching cost defends it)**. Drop any of the four and the corpus stops being a moat and becomes a cost. That is why the next section — who decides what the correct answer is — is the load-bearing one: a corpus is only as defensible as the authority behind its labels.

---

## 4. The Ground Truth Problem — Who Decides What Correct Is

This is the central and longest section, because it is where the thesis is won or lost. A domain corpus is raw material; **ground truth is the decision about what the material means**, and in a vertical domain that decision belongs to an *expert with authority*, not to a dataset builder. The mechanics of evaluation are owned elsewhere (`llm_evaluation_frameworks_guide.md`, `llm_evaluation_vs_validation_guide.md`, and the rag/ shelf) and are cited here, not restated. What this section develops is the **domain-specific problem**: who is allowed to decide, what happens when experts disagree, how a golden set is actually built, and why a domain's errors are not symmetric.

### 4.1 The problem stated

For a general agent, "correct" is usually decided by whoever builds the eval — a reasonable engineer with a rubric. For a vertical agent, "correct" is a **professional judgement**: what a competent practitioner in the domain would conclude, under the domain's rules, given the facts. The agent cannot define that standard, the engineer cannot invent it, and — the subtle part — **a single dataset builder cannot ratify it either.** Ground truth in a vertical is an act of *authority*.

### 4.2 The expert and the expert scarcity problem

The people who hold the authority to define correctness in a domain are the domain's scarce, expensive professionals: the clinician, the lawyer, the underwriter, the compliance officer, the senior engineer of record. Three consequences follow, and they are the reason this problem is under-discussed:

1. **The experts are the bottleneck, not the data.** You can machine-generate inputs; you cannot machine-generate *adjudicated correct answers in a profession* without the professional. Expert time is the binding constraint on the entire programme, which inverts the usual "get more data" instinct.
2. **Their time is worth more elsewhere.** The expert's day-job produces revenue or serves clients; labelling is a cost to them. Every hour of expert labelling is an hour of expert non-practice, and the organisation must *choose* that trade explicitly.
3. **Their judgement is not uniform.** Two qualified experts do not always agree — which is the next subsection, and the real reason ground truth is hard.

The design implication (developed in §4.6): the expert must be **inside the loop of building**, not consulted at acceptance. A vertical agent whose experts were only consulted at the end was built on the builder's opinion of the domain. This is the *vertical* form of a pattern the repository already names elsewhere — see `feedback_mechanisms_llm_agents_guide.md` for feedback loops generally and `llm_evaluation_vs_validation_guide.md` for the validation/separation discipline; the vertical delta is that the human on the validating side is a *licensed professional*, not a reviewer.

### 4.3 Inter-annotator disagreement — the normal case

When two domain experts independently label the same input, they disagree some of the time. This is not a defect to be engineered away; it is a **property of the domain**, and it is the single most under-modelled fact in vertical-agent ground truth.

- **Some disagreement is noise** (inattention, fatigue, ambiguity in the labelling instructions) — reducible by better instructions and more annotators.
- **Some disagreement is signal** — the domain genuinely admits two defensible answers, or the input is under-specified, or the experts are applying different *precedents* (a clinician following two different guidelines; a lawyer in two jurisdictions; an underwriter under two risk appetites). This kind of disagreement is **irreducible** by adding annotators, and it is information: it tells you the domain's *irreducible ambiguity*.
- **Aggregating disagreement is itself a decision.** Majority vote is not neutral — it is a choice that privileges the modal expert and hides the dissenter. The vertical must decide, per labelled item, whether it wants *the consensus answer*, *the conservative answer*, or *all defensible answers with their disagreement recorded*. That decision is domain-specific and cannot be defaulted to a generic annotation tool.

**The consequence for the moat:** a competitor duplicating your corpus will **not** duplicate your disagreement resolution. Two teams can hold the *same* documents and produce *different* ground truth, because they resolved the ambiguous cases differently. This is one of the few places a vertical's advantage is genuinely a *judgement* moat, not a *data* moat.

### 4.4 Asymmetric error — why a domain's errors are not symmetric

A general evaluation assumes a wrong answer is a wrong answer. In a vertical domain, **the two kinds of error have different costs**, and the domain's own practice says so.

- **A false positive and a false negative rarely cost the same.** A missed regulatory breach (§9) and an over-flagged one are different failures with different consequences for different parties. The domain — not the builder — decides the *relative* cost, and it is frequently extreme (a missed finding is catastrophic; a spurious one is an inconvenience, or vice versa).
- **The assessment side is not neutral.** In a regulated domain, a domain's *own standards* (a professional body's guidance, an institution's policy) often explicitly treat omission and commission differently. Where a source establishes that stance for a domain, cite it; where one was not found, this guide marks it **unverified** and does not assert it (§9.6).
- **The metric must encode the asymmetry.** A single accuracy number destroys exactly the information the domain cares about. §5 develops what the acceptance test must therefore preserve; the *mechanics* of precision/recall-style measurement live in `llm_evaluation_frameworks_guide.md` and `rag_evaluation_methodology_guide.md`, and are cited, not re-derived.

The vertical point: **you cannot borrow a general benchmark's definition of "correct" into a regulated domain**, because the benchmark assumed symmetric errors and the domain does not.

### 4.5 The golden dataset — how it is actually built

A "golden dataset" is the curated, adjudicated set of inputs with their accepted outputs against which a vertical agent is accepted. The repository uses the term 15 times as an *artefact*; here is how a *vertical* golden set is built, step by step:

1. **Sampling by domain reality, not convenience.** The items must be drawn from the *actual* distribution of the workflow (§6) — including the rare, ugly, ambiguous cases — not from the easy cases near the user interface. A golden set that over-samples clean inputs overstates the agent.
2. **Expert labelling, by the authority.** The professional (§4.2) supplies the correct answer under the domain's rules, with the reasoning recorded, not just the answer.
3. **Multiple annotators on the cases that matter.** Overlap the experts on the ambiguous and high-consequence items — not on every item, which would waste expensive time — to *measure* disagreement (§4.3).
4. **Adjudication.** A defined process resolves disagreement: a senior expert, a panel, or a written rule that says which precedent governs. Adjudication is where ground truth is *made*, and it must be **documented**, because it is the thing audited later (§9).
5. **Provenance and versioning.** Every item carries who labelled it, when, under which rule, and which version. When the domain changes (§3.5), the golden set is re-versioned, not silently edited. Versioning mechanics are owned by `agent_versioning_guide.md`; the vertical point is that ground truth is a *dated artefact*.
6. **Held-out honesty.** The golden set is not the tuning set. The separation discipline is owned by `llm_evaluation_vs_validation_guide.md`; here, the vertical point is that an expert who helped *build* the agent must not be the only expert who *accepts* it (§4.6).

### 4.6 The expert-in-the-loop architecture

Combining 4.2–4.5, the vertical ground-truth architecture has four roles, and conflating any two is a common failure:

- **Labeller** — the domain expert who produces correct answers (scarce, expensive).
- **Adjudicator** — the authority who resolves disagreement and owns the standard (rarer still; often a single named person or committee).
- **Acceptor** — a *different* expert (or the adjudicator) who signs off the agent against the held-out golden set, so that the agent is not graded by whoever built it.
- **Maintainer** — whoever keeps the corpus and golden set current as the domain decays (§3.5).

The vertical failure this architecture prevents is subtle and important: **an agent validated only by its builders is validated against the builders' model of the domain, not the domain's.** The authority to decide correctness must sit with the profession — and, in a regulated vertical, potentially with a regulator or a professional body, which is §9's problem, not the builder's to solve alone.

### 4.7 Where the ground-truth argument is thin

Honesty requires marking the limits of this section:

- **"Shallow ground truth" is not formally defined here.** It describes ground truth that is easy to obtain, publicly inferable, or not genuinely adjudicated. The claim that shallow-ground-truth verticals sustain **no moat** is this guide's reasoning ⚠, consistent with the honest counter-argument in §1.4, but it is **not a measured result** and no source read for this guide establishes it.
- **Irreducible disagreement has no accepted measurement method in this guide's sources.** The *existence* of inter-annotator disagreement in expert labelling is well attested in evaluation practice, but the *right way to resolve it for a vertical agent* is not settled, and this guide does not invent a formula.
- **The cost of expert labelling is real but unpriced here.** No figure is printed; the reader should treat expert time as the dominant, unquantified cost of a vertical programme.

The next section takes the asymmetry of §4.4 and asks how a domain's *own standards* enter the acceptance test.

---

## 5. Evaluation by Domain Standards

Section 4 established that a domain's errors are asymmetric and its ground truth is adjudicated. This section turns that into the *acceptance test*: what a vertical must measure, what it must refuse, and how the domain's own standards enter the test. The **mechanics** of evaluation — harnesses, metrics, statistical validity, tooling — are owned by `llm_evaluation_frameworks_guide.md`, `llm_evaluation_vs_validation_guide.md`, `rag_evaluation_methodology_guide.md` and `rag_evaluation_tools_comparison_guide.md`, and are **cited here, not re-derived**. This section develops only the *domain-specific* consequence.

### 5.1 What must be preserved from §4

The acceptance test must encode three things a generic benchmark drops:

1. **The asymmetric cost of the two error directions** (§4.4) — reported separately, never averaged into a single score that destroys the distinction.
2. **The domain's irreducible ambiguity** (§4.3) — an agent that confidently picks one defensible answer where the domain admits two is *not* unambiguously better; the test must be able to see that.
3. **The domain's own standard of correctness** — not the builder's. The rubric must be the profession's, sourced and dated.

### 5.2 The refusal test — what the agent must decline to answer

This is the vertical-specific evaluation the shelf does not cover as a *domain* problem. In a vertical domain, **some questions must not be answered by the agent at all**, and the acceptance test must measure the agent's ability to *refuse*:

- **Out-of-scope requests.** A vertical agent is defined by its domain boundary; a request outside it should be declined, not answered with a plausible general-model guess. Measured as a first-class metric, not a footnote.
- **Requests that require authority the agent does not hold.** Where answering would constitute *advice* rather than *assistance* (§9), the correct behaviour is to escalate or decline. The posture for "who is permitted to commit the institution to what" is owned by `agent_decision_policy_guide.md`; this section cites it and does **not** re-derive the ask/recommend/act/wait taxonomy.
- **Requests on which the domain requires a licensed human.** Where the domain's rules reserve a decision to a professional, the agent's job is to prepare, not to decide. (Asserted here as a *design pattern*; any *regulatory* dimension is §9's, and is marked unverified where no source was found.)

The refusal metric has an obvious failure mode — an agent optimised to refuse everything — so the test must measure refusal *and* coverage together, which is the domain form of the precision/recall tradeoff the evaluation guides own.

### 5.3 How a domain's own standards enter the acceptance test

The distinctive move of a vertical evaluation is that the **rubric is imported from the domain, not invented by the builder**:

- **Source the standard.** The domain's correct answers should be traceable to the domain's authority — a professional body's guidance, an institution's policy, a published standard, a statute where applicable. The evaluation rubric is a *domain artefact*, dated.
- **Adjudicate against the standard, not taste.** When the golden set's answer and the standard disagree, the standard governs, and the disagreement is a *finding about the golden set* (§4.5), not a licence to pick the convenient answer.
- **State the version.** A rubric tied to a superseded standard is a rubric that will pass a non-compliant agent (§3.5's decay applies to the *rubric* too).

### 5.4 The acceptance test, assembled

Putting 5.1–5.3 together, a vertical acceptance test has five components a generic eval does not:

| Component | What it measures | Owned mechanics |
|---|---|---|
| **Directional error** | false-positive and false-negative rates *separately*, weighted by the domain's own cost asymmetry | `llm_evaluation_frameworks_guide.md` |
| **Refusal / escalation** | correct declines and escalations, and their coverage cost | this guide (§5.2); posture from `agent_decision_policy_guide.md` |
| **Irreducible-ambiguity handling** | behaviour on items the domain admits two answers for | this guide (§4.3) |
| **Standard-traceability** | whether correct answers trace to the domain's authority | this guide (§5.3) |
| **Retrieval faithfulness** | whether in-domain answers are grounded in the corpus, not the model's memory | `rag_evaluation_methodology_guide.md`, `production_grade_agentic_rag_guide.md` |

### 5.5 What the domain standard does *not* licence

Two cautions, because verticals are prone to both:

- **A domain standard is not a benchmark you can borrow wholesale.** A standard tells you what correct *is*; it does not give you a representative sample, a threshold, or a pass mark. Those are still engineering decisions, and no source read for this guide establishes a domain-independent threshold.
- **"It passed our golden set" is not "it is correct in production."** The web surfaces it will meet in production are larger than the golden set; production failure modes are owned by `production_ready_llm_agents_guide.md` and `agents_work_fall_apart_guide.md`. A domain acceptance test is a gate, not a guarantee.

---

## 6. The Workflow Embedding

A vertical agent is not a chat window; it lives **inside a process it does not own**, attached to a **system of record**, with handoffs to humans and to other systems. This is the section where the vertical's *switching cost* — the second half of the moat (§1.4) — is created. The posture of the agent at each handoff (ask, recommend, act, wait) is owned by `agent_decision_policy_guide.md`; what a system of record *is* is owned by the banking/ shelf and by `technology/a16z_big_ideas_2026_guide.md` §7. This section **re-derives neither** and develops only the vertical consequences.

### 6.1 The agent is a participant, not the process

The process exists before the agent and will exist after it. The agent is inserted as one participant among: the human operator, the downstream system, the upstream data source, and the system of record that holds the process's true state. Three consequences:

1. **The agent inherits the process's constraints.** Latency budgets, an approval step, a reconciliation window, a shift pattern — these come from the process, not the model, and the agent must fit them (latency mechanics: `llm_latency_optimization_guide.md`).
2. **The agent's output is an input to someone else.** A summary, a flag, a draft decision — its value is realised when the *next* participant acts on it, which is the workflow form of the "designed for a consumer" point in `agent_experience_guide.md`.
3. **The process, not the agent, defines success.** A "correct" answer that arrives too late, or without the metadata the next step needs, is a failure in the workflow even if it is correct in isolation.

### 6.2 The attachment to the system of record

The pivotal architectural fact of a vertical agent is its attachment to a **system of record** — the authoritative store of the process's state. The banking/ shelf owns concrete examples and the *nature* of such systems: `posting_engine_core_banking_guide.md` (the ledger as record), `core_banking_processes_guide.md` (the processes), `data_models_banking_insurance_guide.md` (the data models), and `fix_protocol_guide.md` (an industry message standard). Venture framing that systems of record "lose primacy" as models read operational data directly is a **dated, attributed claim, not an adopted one**: a16z Big Ideas 2026, Sarah Wang, entry "Systems of record lose ground" ✅ (catalogued in `technology/a16z_big_ideas_2026_guide.md` §3.4 item 7). This guide quotes it as a *claim* and takes no position on its truth.

The vertical consequences of the attachment:

- **The record is the source of truth; the agent must not become a second one.** A vertical agent that maintains its own competing state creates reconciliation debt — exactly the failure `agents_work_fall_apart_guide.md` describes.
- **Writes are the dangerous half.** Reads are safely repeatable; writes to the record are consequential and belong under the write-boundary controls the repository covers in `agent_sandboxing_strategies_guide.md` and the harness guide. The *vertical* point is which writes the domain permits — a domain's rules decide whether the agent may post, or only propose a posting.
- **The attachment is where switching cost lives.** Once the agent is wired into the record and the surrounding process, replacing it means touching the process — which is the durable, workflow-shaped half of the moat (§1.4, §8).

### 6.3 The handoffs

A vertical agent hands off in four directions, and each is a place verticals fail:

| Handoff | Direction | Vertical failure |
|---|---|---|
| **To the human** | agent → operator | output a human cannot act on (wrong granularity, no rationale, no confidence) |
| **To the system of record** | agent → record | a write the domain did not authorise, or a second source of truth |
| **To the next system** | agent → downstream | output the next system cannot consume (missing schema, missing provenance) |
| **To a human decision-maker** | agent → professional | advice the agent should not have given (§9), or a recommendation without the evidence behind it |

The posture at each handoff is a decision-policy question, and the taxonomy is borrowed, not invented (`agent_decision_policy_guide.md`). The *vertical* input to that decision is the domain's own rule about who may commit the process to what — which is §9's subject for regulated domains.

### 6.4 What "the workflow never got embedded" looks like

The failure named in §11 is common enough to describe here: the agent works in a demo, passes its golden set, and is **never actually inserted into the process** — it remains a side tool someone uses separately, so it creates no switching cost and no compounding corpus (§3.1 class 4), and its ground-truth loop never closes. An un-embedded vertical agent has neither half of the moat. This is the single most expensive and least visible failure in the whole discipline, because the agent still "works" on the screen.

---

## 7. The Build Decision, in the Vertical's Terms

A vertical builder makes the same *categories* of decision as any agent builder — buy vs build, which model, retrieval or not, how to evaluate — but the *inputs* to those decisions are different because the domain is specific. This section states the vertical input; the **mechanics of each decision are owned by the named guide and are not re-derived here.**

### 7.1 Buy vs build

- **What a vertical buys.** A domain often already has vendors selling the corpus, the workflow software, or a domain model. Buying the *commodity* layer (hosting, the base model, retrieval infrastructure, observability) is usually rational.
- **What a vertical must build.** The thing that is the moat — the adjudicated ground truth (§4.5) and the workflow embedding (§6) — is precisely the thing no vendor can sell you, because it is *your* domain's correctness decided by *your* experts. A vertical that buys its ground truth has bought a non-exclusive asset (§3.6) and has no moat.
- **The vertical rule of thumb.** Buy the shared, built layer; build the proprietary, judged layer. Mechanics of the shared layer: `ai_agent_platform_selection_guide.md`, `enterprise_agentic_platform_architecture_guide.md`.

### 7.2 The model question — hosted vs open, frontier vs fine-tuned

The model is, for most verticals, the **least vertical component** — which is the guide's title and its quiet main claim. The vertical decision inputs:

- **Hosted vs open is usually a data-governance decision, not an accuracy decision.** In a regulated vertical, where the corpus may not leave a jurisdiction or a tenancy, the model choice follows the data constraint. (Data-residency and governance framing: `ai_governance_framework_guide.md`; this guide takes only the vertical input.)
- **Frontier vs fine-tuned is a question of *what is proprietary*.** Fine-tuning helps most where the *behaviour* is domain-specific and stable (format, tone, procedure) — mechanics in `llm_instruction_tuning_guide.md` and `fine_tuning_frameworks_comparison_guide.md`, cited not re-derived. But fine-tuning on a *public* corpus produces a public-corpus vertical (§3.6), which the honest counter-argument dissolves. The vertical rule: **fine-tune to encode proprietary behaviour, retrieve to reach proprietary knowledge** — and note this is a *heuristic*, ⚠, not a measured law.
- **Model swap risk.** A vertical whose advantage sits in *prompt-level* domain tuning rather than in its corpus is fragile to exactly the failure §11 names: a model change that silently removes the advantage. If a model swap can erase the vertical, the moat was never the corpus.

### 7.3 The retrieval question

- **Retrieval is the vertical's default reach mechanism into a corpus** because it does not require the corpus to be public or fixed. The mechanics are owned thoroughly: `advanced_rag_techniques_guide.md`, `production_grade_agentic_rag_guide.md`, `agentic_search_vs_rag_guide.md`, `query_rewriting_rag_guide.md`, `rag_conflict_resolution_guide.md` (the last is directly relevant to §3.5's decaying, conflicting corpus — cited, not re-derived).
- **The vertical input is freshness and provenance, not chunk size.** A vertical's retrieval must respect the corpus's *dates and authority* (§3.3, §3.5): retrieving a superseded document as if current is a domain error, and conflict resolution among domain documents is a domain problem, not a generic one.
- **Retrieval does not create a moat by itself.** A retrieval pipeline over a public corpus is a commodity (mechanics are well documented and widely available); the moat is *what* is retrieved, not *how*.

### 7.4 The eval question

- **The vertical builds its eval first, from its experts (§4, §5), before it builds the agent.** The generic eval tooling is abundant (`llm_evaluation_frameworks_guide.md`, `rag_evaluation_tools_comparison_guide.md`, `ragas_guide.md`, `deepeval_guide.md`); the *vertical* work is sourcing the rubric and the adjudicated set, which no tool provides.
- **The vertical accepts against the domain's standard, not the builder's** (§5.3), with a held-out set and an independent acceptor (§4.6).
- **The vertical evaluates the refusal behaviour as a first-class metric** (§5.2), which most generic harnesses do not frame as a domain requirement.

### 7.5 The decision, restated

The vertical builder who internalises this section ends up asking, for every component: **"Is this component shared and therefore buyable, or is it specific and therefore the moat?"** The model, the retrieval infrastructure and the observability are usually shared. The corpus, the adjudicated ground truth and the workflow embedding are usually specific. Spend the scarce expert time on the specific ones and rent the shared ones — and keep the two from being confused, because the shared ones are the least vertical thing about the whole build.

---

## 8. The Domain Integration

§6 established *that* a vertical attaches to a system of record and hands off in four directions. This section develops the integration as an engineering and cost reality: what a vertical must connect to, how data-access and permissions work in a domain, and what integration actually costs. The *protocol* mechanics are owned elsewhere (`mcp_discovery_guide.md`, `mcp_framework_tools_guide.md`, `mcp_progressive_disclosure_guide.md`) and the *isolation* mechanics by `agent_sandboxing_strategies_guide.md`; both are cited, not restated.

### 8.1 What a vertical must attach to

A vertical agent rarely attaches to one system; it attaches to a **constellation**:

- **The system of record** — the authoritative state of the process (§6.2). What it is: banking/ shelf (`posting_engine_core_banking_guide.md`, `core_banking_processes_guide.md`, `data_models_banking_insurance_guide.md`, `fix_protocol_guide.md`).
- **The corpus stores** — the documents, cases and knowledge it retrieves from (§3), which are often *not* the system of record and are frequently messier.
- **The surrounding tooling** — the ticketing system, the email thread, the spreadsheet a team actually uses. The unglamorous integrations are where the workflow embedding succeeds or fails (§6.4).
- **The audit and log store** — which in a regulated vertical is *mandatory*, not optional (§9.3).

### 8.2 Data access and permissions — the vertical's sharpest constraint

Access control in a vertical is not a generic RBAC exercise, because the domain's permissions are *substantive*:

- **Need-to-know is domain-defined.** Who may see a client's data, a patient's record, a case file is fixed by the domain (§9), not by the builder's role model. The agent must inherit those rules, not invent a coarser ones.
- **The agent should hold no standing authority it does not need.** Least-privilege is a general principle, but the vertical input is *which* privileges the domain grants to the role the agent is standing in for — the agent acting as a junior analyst must not carry a partner's rights (posture and authority: `agent_decision_policy_guide.md`, cited not re-derived).
- **Read/write separation is domain-critical.** The domain decides what the agent may *read* (broad, often) versus what it may *write* (narrow, sometimes not at all). Enforce the boundary server-side; mechanics of isolation and enforcement: `agent_sandboxing_strategies_guide.md`, `agent_harness_engineering_guide.md`.
- **Credentials do not live in the model.** Tool exposure should be through a governed surface (`mcp_framework_tools_guide.md`), and the progressive-disclosure principle (`mcp_progressive_disclosure_guide.md`) applies to *permissions* as much as to tools — the agent sees the access it is granted, no more.

### 8.3 What integration actually costs

Integration is usually the largest and least-forecast line item in a vertical programme, and it is a *domain* cost more than a *technical* one:

- **Schema and semantic mismatch.** The domain's data was not modelled for your agent; reconciling it is the normalisation work of §3.3 and it is where the majority of the effort sits.
- **Latency and availability of the record.** The agent must fit the record's response times and outages; mechanics: `llm_latency_optimization_guide.md`, `agent_runtime_cache_design_guide.md`, cited not re-derived.
- **Change management against the process.** Because the agent lives in *someone else's* process (§6.1), the integration includes negotiating with the process's owners — a human cost, not a technical one.
- **The permanent attachment.** Once embedded, the integration must be maintained as the record, the corpus and the domain evolve (§3.5). Integration is a subscription, not a project.

The vertical point: **the integration is where the switching cost is built (§6.2), so under-investing in it both risks the deployment and forgoes the moat.** A vertical agent that is easy to detach was never embedded.

---

## 9. Liability and the Professional Duty

When a vertical is *regulated*, being vertical changes the legal shape of the product, not just its accuracy. This section develops what changes — **assistance vs advice, the audit surface, and the duty that does not transfer** — and it does so under a strict discipline: **no jurisdiction's rule is asserted without a source.** Where a rule could not be sourced in this pass, it is marked **unverified** rather than stated, as §9.6 records. This guide is not legal advice and does not pretend to be.

### 9.1 What regulation changes

In an unregulated vertical, an error is a quality problem. In a regulated one, an error can be a **compliance event** with consequences for the deploying institution — and sometimes for the vendor. Three things change:

1. **The correctness standard is external.** The domain's regulator or professional body defines acceptable practice, and the agent's ground truth (§4) must trace to it (§5.3).
2. **Conduct is constrained, not just accuracy.** Who may do what, what may be recorded, what may be retained and for how long, and what may be delegated — these are constraints on the *process*, which the agent inherits (§6.1).
3. **The failure is auditable after the fact.** A regulated domain expects to reconstruct why an outcome happened, which makes the agent's logs, rationale and retention part of the product (§9.3).

### 9.2 Assistance versus advice — the load-bearing distinction

The most consequential regulatory distinction for a vertical agent is between **assisting a professional** and **advising a client**:

- **Assistance** — surfacing information, drafting, summarising, flagging, preparing — supports a professional's own judgement and leaves the judgement with the licensed human.
- **Advice** — a recommendation or conclusion given to a client or a counterparty — may constitute a regulated activity reserved to a licensed or authorised person, or trigger duties (suitability, disclosure, best-interest) that attach to the *advisor*, not the tool.

**The vertical design consequence:** many regulated verticals are viable *only* in the assistance posture, with an explicit, enforced boundary before advice. The agent must be able to *refuse* to cross it (§5.2), and the boundary must be enforced in the system, not requested in the prompt (posture and enforcement: `agent_decision_policy_guide.md`; enforcement mechanics: `agent_sandboxing_strategies_guide.md`).

**The honesty caveat, stated plainly:** *which* activity counts as advice, and *who* may provide it, is defined by a specific jurisdiction and a specific regulated activity. **This guide asserts no jurisdiction's rule, because none was verified during writing.** Any team applying this must source its own jurisdiction's rule with counsel. The distinction is offered as a *design axis*, not as a legal conclusion.

### 9.3 The audit surface

A regulated vertical must be able to answer, after the fact, *why* the agent produced what it produced. That makes four things part of the product, not afterthoughts:

- **Logs.** What went in, what came out, which tools were called, which corpus items were retrieved, and under which version of the agent and the corpus. Operations mechanics: `agentops_guide.md`; versioning: `agent_versioning_guide.md`. The vertical point: in a regulated domain, an unlogged decision is an indefensible decision.
- **Explainability, to the domain's standard.** The explanation a regulator or a professional needs is a *domain* explanation (which rule, which precedent, which corpus item), not a model-internals explanation. This is a domain artefact and cannot be bought generically.
- **Retention.** How long records of the agent's activity must be kept is a domain question, and this guide **asserts no retention period** — no source verified one here. Retention is listed as a *required decision*, with the period marked unverified.
- **Provenance (§3.3).** The ability to trace an output back to a dated, licensed source is what makes the audit surface real; without provenance the logs record *what* happened but not *on what authority*.

### 9.4 The professional duty that does not transfer

This is the sharpest vertical-liability point, and it is a *subtraction*, not an addition:

- **The professional's duty to the client does not move to the vendor by delegating a task to software.** A licensed professional remains answerable for the professional judgement; the tool's builder does not thereby inherit the profession's duty. (Stated here as a *general principle of professional regulation*, ⚠ — the specific allocation of liability between professional, institution and vendor is **jurisdiction-specific and was not verified**; no source read for this guide establishes it for any jurisdiction.)
- **The corollary for design.** Because the duty stays with the professional, the agent must be designed to *preserve* the professional's ability to exercise it — which means the agent must show its working, must not launder its uncertainty into false confidence, and must escalate where the domain requires a licensed judgement (§5.2).
- **The trap.** A vendor that markets the agent as "making the professional decision" is, in a regulated domain, both over-claiming capability *and* potentially disturbing the duty allocation. The safe framing is the assistance posture (§9.2) and an explicit statement of the boundary.

### 9.5 Where the liability frame is deliberately silent

Following the repository's discipline for its other regulated-domain guides (see the negative finding carried in `agent_decision_policy_guide.md` §15 item 7 and `banking/ai_risk_register_guide.md`), this guide:

- **cites no statute, no regulation and no supervisory rule** by name, because none was verified for this guide;
- **ranks no jurisdiction** by friendliness to vertical agents;
- **asserts no retention, disclosure or suitability requirement** as fact;
- and **flags every liability statement as design reasoning** rather than law.

### 9.6 What a team must verify itself

Before deploying a vertical agent in a regulated domain, the following are **unverified here and must be sourced locally, with counsel**: (i) whether the agent's function constitutes a regulated activity; (ii) whether it is assistance or advice in the jurisdiction's terms; (iii) who is liable when it errs, and how that is allocated between professional, institution and vendor; (iv) what must be logged, explained and retained, and for how long; (v) what disclosures are required. None of these is answered by this guide, and none should be inferred from it.

---

## 10. The Economics and the Buyer

Who buys a vertical agent, and on what metric? This section answers **structurally** — the *shape* of the demand and the *form* of the metric — with **no market size, growth rate, funding figure, adoption percentage or valuation printed anywhere.** The AI-native *business* frame (data flywheels, usage economics, org shape) is owned by `technology/ai_native_companies_guide.md` and is **cited, not re-derived**; as §2.3 insists, AI-native and vertical are different axes and are not blurred here.

### 10.1 Who the buyer is

A vertical's buyer is a **domain budget-holder**, and the domain shapes the buyer's behaviour:

- **The buyer owns the workflow the agent embeds in** (§6), not a general "AI" line. That means the buyer has a *process* to defend and a *risk* to manage, not a curiosity budget.
- **The buyer's metric is the domain's metric.** Not "accuracy" in the abstract, but the number the domain already tracks — a cycle time, a queue, a defect rate, a review burden, a compliance exposure. The agent is bought where it moves *that* number.
- **The buyer may not be the user.** The person who signs is often a head of function, while the person who operates is a specialist (§6.3). Adoption depends on the *operator*, so a vertical that is bought by the head and refused by the operator fails on the floor.

### 10.2 On what metric a vertical agent is bought — structurally

Without printing a figure, the *form* of the buying decision is: **the agent is bought against the domain's own cost of the error and the domain's own cost of the work.**

- **Value ≈ (work volume) × (per-unit cost of the work) × (share the agent credibly takes) − (cost of the agent's errors as the domain prices them) − (integration and maintenance).**
- The **dominant term is usually the domain's cost of error**, not the per-unit work cost — which is why §4's asymmetric error and §5's directional metrics are *economic* requirements, not just quality ones. A vertical that reduces work cost but raises the domain's error cost has negative value regardless of its accuracy.
- **The metric is domain-specific and must be agreed with the buyer before the build.** A general "pass rate" is not a currency the domain buyer accepts.

### 10.3 The pricing question, structurally

This guide prints **no pricing** and **no pricing model recommendation**, but the *vertical* structure of the question is:

- **The agent's value concentrates where the domain's error cost concentrates** (§10.2), so pricing tied to *outcomes the domain already measures* is coherent with the vertical's nature — while pricing tied to usage alone ignores the error asymmetry that dominates the value.
- **The switching cost (§6.2, §8.3) supports durability of the relationship, not a licence to over-price.** A workflow-embedded agent has a real moat against *replacement* (§1.4), but the same embeddedness makes an over-priced agent a *target* for the buyer's own build-or-buy review (§7.1).
- **Cost lives in the corpus and the experts (§3, §4), not the model.** A vertical's gross-margin structure is set by how much expert and curation time each unit of value consumes — the model is the *small* line. This is the guide's title in economic form.

### 10.4 The AI-native economics frame — cited, not adopted as vertical

The data-flywheel, usage-based, iterate-fast economics of AI-native businesses are a genuine frame, and `technology/ai_native_companies_guide.md` owns it. Two cautions keep the axes apart:

- **A flywheel needs the loop to be proprietary.** The "data flywheel" only compounds if the data is yours and adjudicated (§3, §4). A vertical with a public corpus has a flywheel that spins in place.
- **AI-native economics are not a vertical advantage.** They are a *build* choice (§7) that a horizontal competitor can also make. Being AI-native does not make you vertical, and being vertical does not automatically make the economics work.

### 10.5 The demand-side asymmetry

Why verticals attract builders, stated without a figure: the *buyer's* problem is specific and painful (a domain budget with a named cost), while the *builder's* commodity layer is shared and cheap (§7.1). That asymmetry — specific pain, shared cost — is the structural reason vertical agents are pursued. It is also exactly why the honest counter-argument matters: the same shared, cheap, improving model layer that makes a vertical cheap to build makes it easy to build *twice*, and the difference between two verticals is only the moat of §3 and §4.

### 10.6 Where the economics claim is thin

- **No source read for this guide** establishes that vertical agents outperform general ones economically, for any vertical (this is the §11-style negative, carried to §15).
- **The value formula in §10.2 is this guide's structural reasoning** ⚠, offered as a way to frame the decision, not as a measured model. No coefficients are asserted.
- **The buyer-side account is structural, not empirical.** No study read for this guide measures vertical-agent adoption by buyer type or budget; the account is a reasoning frame and is marked as such.

---

## 11. The Failure Modes of Vertical Agents

Six failures recur in vertical programmes, and they are *vertical-specific* variants of general agent failures (the general forms are owned by `production_ready_llm_agents_guide.md`, `llm_agents_failures_production_guide.md`, `agents_work_fall_apart_guide.md`). Each is stated as symptom → cause → guardrail.

### 11.1 The demo trap

- **Symptom.** The agent dazzles on curated inputs, is signed off, and underperforms or misbehaves on the messy real distribution.
- **Cause.** The demo set was drawn from the easy region near the interface, not from the workflow's actual distribution (§4.5 step 1); the golden set, if it exists, shares the same bias.
- **Guardrail.** Sample the acceptance set from production-reality inputs *including the rare and ugly*; do not accept against a demo set; the eval discipline is `llm_evaluation_vs_validation_guide.md`'s, applied in-domain.

### 11.2 The domain-prompt-only agent

- **Symptom.** A general model with a long domain prompt, no corpus, no ground truth, marketed as a vertical.
- **Cause.** Treating verticality as a *prompt* rather than a *corpus and a judged standard*. The prompt is shared and copyable; there is no moat (§3.6).
- **Guardrail.** Ask where the ground truth is (§4); if the answer is "in the system prompt," there is no vertical, only a persona.

### 11.3 The model swap that breaks the vertical

- **Symptom.** A model upgrade (or vendor change) silently removes the advantage the vertical depended on.
- **Cause.** The advantage sat in *prompt-level or model-level* domain tuning rather than in the corpus and the embedding (§7.2). The moat was never the ground truth; it was borrowed behaviour.
- **Guardrail.** Hold the advantage in the corpus, the golden set and the workflow embedding (§3, §4, §6); re-run the domain acceptance test on *every* model change (the versioning discipline is `agent_versioning_guide.md`'s; the trigger is the vertical's own).

### 11.4 The expert bottleneck

- **Symptom.** The programme stalls waiting on the domain experts; labelling never finishes; the golden set stays small.
- **Cause.** Expert time is the scarce resource (§4.2) and was not budgeted as the *dominant* cost. The plan assumed engineers could substitute.
- **Guardrail.** Budget expert time first and explicitly; label the high-consequence and ambiguous items with overlap (§4.5 steps 3–4) rather than trying to label everything; accept that the bottleneck is *by design* and plan the programme around it.

### 11.5 The ground truth that turned out to be one person's opinion

- **Symptom.** The agent conforms to *one* expert's practice; other qualified experts reject its outputs; the eval "passed" against a standard that was one person's habit.
- **Cause.** No adjudication, no multiple annotators, no domain-sourced rubric (§4.3, §4.5, §5.3). A single expert's answer was treated as ground truth.
- **Guardrail.** Overlap annotators on the items that matter; adjudicate disagreement against a *sourced* domain standard, not seniority; make the adjudicator role explicit and named (§4.6).

### 11.6 The vertical whose workflow never got embedded

- **Symptom.** The agent works, is used casually, and produces no compounding corpus and no switching cost; it is a side tool. (§6.4.)
- **Cause.** The integration into the process and the system of record (§6, §8) was treated as optional, or the process's owners were never brought in (§8.3).
- **Guardrail.** Treat embedding as the deliverable, not the code; measure the loop closing (is the workflow-log corpus growing? §3.1 class 4); if the agent can be removed without touching a process, it was never embedded.

---

## 12. A Vertical-Agent Readiness Assessment

A checklist with **named inputs**, no benchmark figures and no thresholds. Every item is answered with a *named artefact or person*, or it is open. An assessment that cannot name an input is a "no."

### 12.1 The corpus

1. **Corpus inventory** — a written list of the artefact classes (§3.1) and where each sits on the licensing ladder. *Named input: the inventory document.*
2. **Licence position** — for each licensed source, exclusive or non-exclusive (§3.6). *Named input: the licence, or "we own it."*
3. **Provenance** — can every artefact be traced to a dated source and a licence? (§3.3.) *Named input: the provenance schema.*
4. **Maintenance owner** — who keeps the corpus current as the domain decays? (§3.5.) *Named input: a person.*

### 12.2 The ground truth

5. **Named experts** — who are the labellers, with their domain credentials? (§4.2.) *Named input: the expert names.*
6. **Adjudicator** — who owns the resolution of disagreement, and by what rule? (§4.6.) *Named input: a named adjudicator or panel and a written rule.*
7. **Sourced standard** — which domain authority does "correct" trace to, and its version? (§5.3.) *Named input: the standard, cited.*
8. **Golden set with overlap** — is there an adjudicated set with multiple annotators on the items that matter? (§4.5.) *Named input: the golden-set artefact, with annotator overlaps shown.*
9. **Independent acceptor** — is the agent accepted by someone other than its builder? (§4.6.) *Named input: a named acceptor.*
10. **Directional metrics** — are false-positive and false-negative reported separately, weighted by the domain's cost? (§5.4.) *Named input: the metric definition.*

### 12.3 The workflow

11. **Process named** — which process, and who owns it? (§6.1.) *Named input: the process and its owner.*
12. **System of record named** — which record does the agent attach to, and read/write scope? (§6.2, §8.2.) *Named input: the system and the permission grant.*
13. **Handoffs specified** — the four handoffs of §6.3, each with its expected output form. *Named input: the handoff specification.*
14. **Loop closure** — does using the agent feed the corpus (§3.1 class 4)? *Named input: the trace/log design.*
15. **Undo / write boundary** — what may the agent write, and what is the compensating action? (§6.2; posture from `agent_decision_policy_guide.md`.) *Named input: the write-boundary policy.*

### 12.4 The liability frame (regulated verticals)

16. **Regulated-activity screen** — has counsel determined whether the function is a regulated activity? (§9.6.) *Named input: a counsel finding. Absent it, this is open.*
17. **Assistance/advice boundary** — where does the agent's function sit, and how is the boundary enforced? (§9.2.) *Named input: the boundary policy.*
18. **Audit surface** — logs, explanation standard, provenance, retention — each named, with retention marked as a local decision (§9.3). *Named input: the audit specification.*
19. **Duty allocation** — who is liable when the agent errs, as allocated by the institution's counsel? (§9.4.) *Named input: the counsel finding.*

### 12.5 The economics

20. **Buyer's metric** — the domain metric the buyer accepts, named before the build (§10.2). *Named input: the metric, agreed with the named buyer.*
21. **Error-cost weighting** — how the domain prices the two error directions (§10.2). *Named input: the buyer's own weightings.*
22. **Cost-of-ownership** — expert and curation cost treated as the dominant line (§10.3). *Named input: the programme budget by category.*

Every "named input" above is a real artefact, person or agreement. The assessment's verdict is not a score; it is the list of items for which a name could not be produced — and those are the risks.

---

## 13. The Cymbal Bank Worked Example

> **This section is explicitly fictional and illustrative.** Cymbal Bank is the fictional institution used across this repository as a worked-example persona, and it is the **only institution, and the only bank, in any example in this guide.** Every detail below — the workflow, the artefacts, the decisions, the people — is **invented for illustration**, chosen to be plausible, and marked as illustrative. Nothing here is a measurement, a benchmark, a figure, a vendor claim or a regulatory requirement, and none of it should be cited as evidence. The liability discussion in §9 applies: the regulatory statements made here are illustrative and a real deployment must source its own jurisdiction's rules with counsel.

### 13.1 The workflow

Cymbal Bank builds a vertical agent for **one** workflow: reviewing the periodic **covenant compliance certificates** that commercial borrowers submit under their loan agreements. A certificate states the borrower's financial-covenant ratios for a period; a covenant specialist at Cymbal reads the certificate and the loan agreement, checks the stated ratios against the agreed covenant definitions and thresholds, flags apparent breaches, and hands the result to a **credit officer**, who owns the credit decision. The workflow attaches to Cymbal's **loan system of record** (§6.2), which holds the facility, the covenant schedule and the disposition.

This is one workflow, deliberately narrow, because verticality is specificity (§2.4).

### 13.2 The corpus

Working §3.1's ladder, Cymbal's corpus is:

- **Published domain knowledge** (weak floor): accounting standards and general covenant practice — *shared with any competitor.*
- **In-house documents** (defensible): Cymbal's **credit policy**, the standard covenant-definition library, and the **loan agreements** for its own book — Cymbal-specific, not licensable by a competitor.
- **Workflow logs** (a proprietary ground-truth generator): years of past covenant reviews — the certificates, the specialist's checks, the dispositions, the credit officer's decisions. Records what *happened*, not necessarily what was *correct* (§3.1 class 4).
- **Expert-labelled set**: the covenant specialists' adjudicated reviews (§13.3).

**The illustrative moat decision:** Cymbal's *credit policy* and *its own loan agreements* are the differentiating artefacts, and neither is purchasable. A competitor could buy the same accounting standards (class 1) and the same generic covenant library (class 2) — and would then have **no moat** on the same corpus, which is the §3.6 warning made concrete.

### 13.3 The ground truth, with its experts

Following §4:

- **Experts.** Cymbal names its **covenant specialists** as labellers, and its **senior credit officer** as the adjudicator who owns how ambiguous covenants are read (a written adjudication rule, per §4.5 step 4).
- **Disagreement is real here.** Two specialists will genuinely disagree on whether a certificate "complies" when a covenant definition is under-specified or a ratio is computed on a contested basis. The disagreement is *signal* (§4.3): it marks the covenants whose definitions are ambiguous — which is itself a finding Cymbal can act on by tightening the definitions.
- **Asymmetry is stark** (§4.4). *Missing* an actual breach is a materially worse error for Cymbal than *flagging* a certificate that turns out compliant: an over-flag costs a specialist's review time; a missed breach is an unmanaged credit exposure and, potentially, a governance failure. The domain therefore weights the two directions **very** unequally, and §5.4's directional metrics are mandatory, not optional.
- **The golden set** is sampled from the *real* certificate distribution — including the ugly ones (restated figures, partial periods, amended agreements) — with overlapping specialists on the ambiguous and high-consequence items, and adjudicated against the **credit policy** as the sourced standard (§5.3).

### 13.4 The workflow embedding

- The agent is inserted as a **participant** in the covenant-review process (§6.1): it reads the certificate and the agreement, checks ratios against definitions, and produces a **structured finding with its evidence** — not a free-text opinion.
- It **attaches to the loan system of record** (§6.2): it *reads* the facility and covenant schedule, and it **writes a proposed finding**, never a final disposition. The write boundary is the credit officer's decision.
- Its **handoffs** (§6.3): to the specialist (a reviewable finding with rationale), to the record (a proposed disposition, with provenance), to the credit officer (a recommendation with evidence) — and it must **refuse** to communicate a covenant conclusion to the *borrower*, which is §13.5's point.

### 13.5 The liability frame — and the one decision the vertical's nature changed

The regulated-domain frame (§9) applies. In the assistance posture (§9.2), Cymbal's agent **assists** its own specialists and credit officers; it does **not advise** the borrower. (That this is the correct legal characterisation is, per §9.6, a matter for Cymbal's counsel — the example illustrates the *design*, not a legal conclusion.)

**The one decision the vertical's nature changed that a general build would have got wrong:**

> A general build — an agent evaluated on "did it get the covenant right" — treats the covenant conclusion as the product and would naturally surface it wherever the user asks, including directly to the borrower or on a self-service channel, and would grade itself with a single accuracy number. **The vertical build changes the deliverable and the grader.** The *deliverable* is not a conclusion for the borrower; it is a **reviewable finding routed to Cymbal's specialist and credit officer**, because the domain's error asymmetry (§4.4) and the assistance/advice boundary (§9.2) mean the consequence of a *confident, direct* covenant conclusion to a borrower is different in kind from an internal flag. And the *grader* is not one accuracy number; it is the **split false-positive / false-negative rates weighted by Cymbal's cost asymmetry**, adjudicated against Cymbal's own credit policy by an acceptor who did not build the agent (§4.6).

So the same model, the same documents, and the same ratio-check produce a **materially different product** once the agent is vertical: an internal, evidence-bearing, refusal-capable assistant whose errors are measured in the direction the domain prices, rather than a self-service answer machine graded on an average. That single re-framing — *what the agent is for and who it is allowed to answer* — is the vertical's contribution, and it lives nowhere in the model.

### 13.6 Ending on the thesis

Cymbal's advantage, if it has one, is not the LLM it calls — that is shared, improving, and the least vertical thing in the build. It is the **credit policy and loan agreements no competitor holds** (§13.2), the **adjudicated covenant ground truth its specialists produced** (§13.3), and the **loan-record embedding that makes the agent expensive to remove** (§13.4). Strip any of the three and Cymbal's "vertical agent" becomes a general model with a banking prompt — and a future model release can erase it overnight. The model is shared; the ground truth is not. That is the thesis, and Cymbal is one illustration of it.

---

## 14. The Anti-Patterns

Each as **symptom / cause / guardrail**.

### 14.1 The verbatim-domain-prompt vertical

- **Symptom.** A large prompt that recites the domain, no corpus, no golden set, launched as a "vertical agent."
- **Cause.** Verticality treated as instruction-following rather than a proprietary corpus and a judged standard (§3.6, §11.2).
- **Guardrail.** Demonstrate the corpus and the adjudicated ground truth, or drop the "vertical" label.

### 14.2 The single-expert ground truth

- **Symptom.** Every label traces to one specialist; other qualified specialists disagree with the agent.
- **Cause.** No overlap annotation, no adjudication, no sourced rubric (§4.3–4.5, §11.5).
- **Guardrail.** Overlap annotators on the items that matter; adjudicate against a *source*, not seniority; name the adjudicator.

### 14.3 The single-number acceptance

- **Symptom.** The agent is accepted on one accuracy score; the domain's cost asymmetry is invisible to the test.
- **Cause.** Borrowing a general benchmark's symmetric-error assumption into an asymmetric domain (§4.4, §5.4).
- **Guardrail.** Report false positives and false negatives separately, weighted by the domain's own cost of each.

### 14.4 The model-dependent vertical

- **Symptom.** A model change silently degrades the domain behaviour.
- **Cause.** The advantage lived in model/prompt tuning, not the corpus and embedding (§7.2, §11.3).
- **Guardrail.** Re-run the domain acceptance test on every model change; hold the advantage in corpus + golden set + embedding.

### 14.5 The un-embedded agent

- **Symptom.** The agent works but is used off to the side; no process changed; no corpus loop closed.
- **Cause.** Integration treated as optional; the process owners never engaged (§6.4, §8.3, §11.6).
- **Guardrail.** Make embedding the deliverable; verify the loop closes; if removal costs no process change, it was never embedded.

### 14.6 The advice the agent was never authorised to give

- **Symptom.** The agent answers a client or counterparty directly on a matter the domain reserves to a licensed human.
- **Cause.** The assistance/advice boundary was left to the prompt instead of enforced in the system (§9.2).
- **Guardrail.** Enforce the boundary server-side; make refusal a first-class, tested behaviour (§5.2); keep the licensed judgement with the licensed person.

### 14.7 The corpus that was one frozen snapshot

- **Symptom.** The agent confidently cites superseded policy; accuracy decays quietly.
- **Cause.** No maintenance owner, no provenance, no re-versioning as the domain changes (§3.5, §3.3).
- **Guardrail.** Assign a maintainer; make provenance and dates mandatory; re-version the corpus and the rubric together.

### 14.8 The moat invented in a narrative

- **Symptom.** A pitch that claims a data moat whose data is public or non-exclusively licensed.
- **Cause.** Confusing *a* corpus with a *defensible* corpus (§3.6).
- **Guardrail.** Score the corpus on the ladder (§3.1) and the four tests of §3.7 — proprietary, adjudicated, maintained, embedded — or call it a cost, not a moat.

---

## 15. The Claims Audit — Verified, Flagged, Rejected

Every load-bearing claim, with its source, its **date**, and its **kind**. A vendor's definitional page is never recorded as a definition; a venture essay is never recorded as a research finding; a figure is never printed.

| # | Claim | Verdict | Source · kind · date |
|---|---|---|---|
| 1 | A preprint titled "Agentic Systems: A Guide to Transforming Industries with Vertical AI Agents" exists — Fouad Bousetouane, submitted 1 Jan 2025, 31 pp, cs.MA, single author; its abstract proposes a "Cognitive Skills Module" and says the work "attempts to define a level of standardization for Vertical AI agent design patterns" | **Verified** | arXiv:2501.00881 abstract page, read 2026-10-11 — **preprint**, single-author, not peer-reviewed ✅ |
| 2 | A vendor's definitional page defines vertical AI agents as industry-specific and equates the term with "domain-specific agents or function-focused agents" | **Verified (as vendor editorial)** | IBM Think, "What are vertical AI agents?", ibm.com/think/topics/vertical-ai-agents, read 2026-10-11 — **vendor editorial page** ✅ |
| 3 | A venture essay titled "Vertical AI evolves from information retrieval and reasoning to multiplayer" (Alex Immerman) appears in a16z Big Ideas 2026 | **Verified** | `technology/a16z_big_ideas_2026_guide.md` §3.4 item 8 — **venture essay**, 2026 (catalogued in this repo) ✅ |
| 4 | A venture claim that systems of record "lose primacy" (Sarah Wang) appears in a16z Big Ideas 2026 | **Verified (as a dated, attributed claim)** | `technology/a16z_big_ideas_2026_guide.md` §3.4 item 7 — **venture essay**, 2026; quoted, not adopted ✅ |
| 5 | The repository's coverage counts for the vertical terms, and that no `building_vertical*` guide exists | **Verified** | Repo-wide `.md` grep, run 2026-10-11 (§1.5) ✅ |
| 6 | Every cross-referenced shelf file exists at the stated path | **Verified** | Repo file listing, 2026-10-11 (§1.6) ✅ |
| 7 | "Vertical AI" is an established, received term | **Rejected — negative finding** ❌ | Searched; no peer-reviewed definition and no standards-body usage found. One preprint *attempts* standardisation; vendor/venture material uses the term loosely (§1.2) |
| 8 | A vendor's definitional page constitutes a definition or a taxonomy | **Rejected — category error** ❌ | Vendor editorial describes marketing usage, not an accepted definition (§1.2) |
| 9 | Any market size, growth rate, funding figure, adoption percentage or vendor ranking | **Rejected — not asserted** ❌ | By policy (§10, §11); no such figure is printed, and none is treated as evidence |
| 10 | Inter-annotator disagreement occurs in expert domain labelling | **Flagged** | Well attested in evaluation practice generally; **no specific source for it was verified in this pass**, so no rate or study is cited (§4.3) |
| 11 | "Shallow-ground-truth verticals sustain no moat"; the corpus-defensibility ladder; the §10.2 value formula | **Flagged — this guide's reasoning** | Original analysis, marked ⚠ throughout (§3.7, §4.7, §10.6); not a published or measured result |
| 12 | Any jurisdiction's rule on regulated activity, assistance vs advice, liability allocation, retention or disclosure | **Unverified — not asserted** | §9 asserts no jurisdiction's rule; no source verified in this pass (§9.6) |
| 13 | A vertical agent outperforms a general agent for any specific domain | **Rejected — not asserted** ❌ | No source establishing this for any domain was found; the thesis is about *moats*, not measured superiority (§10.6) |

Every row carries a date and a kind. Rows 1–6 are verified existence/date/kind facts; rows 7–9 and 13 are the honest negatives and rejected claims; rows 10–11 are flagged reasoning; row 12 is the standing unverified regulatory boundary.

---

## 16. What Could Not Be Verified, Glossary, Cross-References and Closing Summary

### 16.1 What could not be verified

Recorded as explicit negatives, because a silent omission reads as coverage. **None of these should be asserted elsewhere.**

1. **A peer-reviewed definition, or any standards-body usage, of "vertical AI" / "vertical AI agent."** No such source was found. The nearest is a single-author 2025 preprint that *attempts* standardisation (row 1 of §15) and vendor/venture material that uses the term loosely. The term is a framing (§1.2).
2. **Any market size, growth rate, funding figure, adoption percentage or valuation for vertical AI.** By policy none is printed, and none was used as evidence (§10).
3. **Any jurisdiction's rule** on whether a vertical agent's function is a regulated activity, whether it constitutes advice, how liability is allocated, or what must be logged/retained/disclosed. §9 asserts no such rule; §9.6 lists what a team must source with counsel (row 12 of §15).
4. **A verified source on inter-annotator disagreement specific to expert domain labelling.** The phenomenon is attested in evaluation practice, but no specific study was resolved in this pass, so no rate is cited (§4.3).
5. **Any measurement that vertical agents outperform general agents for a specific domain**, or that a proprietary corpus *always* beats a better general model. Neither is asserted; the corpus-vs-model subordination is this guide's reasoning only (§3.6, §4.7, §10.6).
6. **A domain-independent definition of, or threshold for, "shallow ground truth."** Flagged reasoning only (§4.7).
7. **Any retention period, disclosure requirement or suitability standard.** Named as required *decisions*, with no value asserted (§9.3).

**Standing note on the honest counter-argument.** The thesis of §1.4 is defended, but its refutation condition is real and stated: a foundation-model improvement can erase a domain-data advantage overnight, and a vertical whose ground truth is public, shallow or non-exclusively licensed may sustain **no moat at all** (§3.6). This guide does not claim verticals always win; it claims a vertical's moat exists exactly to the extent its ground truth is proprietary, adjudicated, maintained and embedded — and each can fail independently.

**Research-tooling note.** `web_search` has been unreliable on this host in prior passes; where a search returned nothing it was recorded as a **tool limitation, not absence of material** (per the standing hazard), and the load-bearing method was direct extraction of named primary URLs — the arXiv abstract page, the IBM Think topic page, and the repository's own a16z catalogue. Vendor pages that would not extract were left **unverified** rather than filled from memory.

### 16.2 Glossary

**Assistance vs advice** — the distinction between supporting a professional's own judgement and giving a regulated conclusion to a client; developed in §9.2, asserted of no jurisdiction's rule.
**Corpus (domain corpus)** — the documents, records, cases and logs a domain's expertise is made of; §3.1 gives its artefact classes.
**Golden set** — the curated, adjudicated inputs-with-correct-outputs against which a vertical agent is accepted; §4.5.
**Ground truth** — the answer that is correct *for this domain*, as decided by whoever holds the authority to decide it; §4.
**Horizontal agent** — an agent spanning many domains with no single industry's rules, corpus or process; §1.3.
**Moat (the)** — the durable advantage of a vertical; this guide's thesis locates it in the ground truth, not the model; §1.4.
**Provenance** — the record of where an artefact came from, when, under what licence, and who asserted it correct; §3.3.
**System of record** — the authoritative store of a process's state; defined here by pointer only (§1.6, §6.2).
**Vertical agent** — an agent whose domain, data, workflow, evaluation, buyer and liability are specific to one industry; §2.
**Workflow embedding** — the agent's placement inside a process it does not own, attached to a system of record; §6.

### 16.3 Cross-references

**Owned mechanics cited, not re-derived:** `technology/ai_llm/agent_harness_engineering_guide.md`, `agent_scaffolding_guide.md`, `agent_loop_implementation_guide.md`, `context_engineering_guide.md`, `mcp_discovery_guide.md`, `mcp_framework_tools_guide.md`, `mcp_progressive_disclosure_guide.md`, `agentops_guide.md`, `agent_versioning_guide.md`, `agent_sandboxing_strategies_guide.md`, `agent_runtime_cache_design_guide.md`, `production_ready_llm_agents_guide.md`, `llm_agents_failures_production_guide.md`, `agents_work_fall_apart_guide.md`, `agent_decision_policy_guide.md`, `agent_experience_guide.md`, `llm_evaluation_frameworks_guide.md`, `llm_evaluation_vs_validation_guide.md`, `agentic_search_vs_rag_guide.md`, `query_rewriting_rag_guide.md`, `rag_conflict_resolution_guide.md`, `ragas_guide.md`, `deepeval_guide.md`, `llm_latency_optimization_guide.md`, `llm_instruction_tuning_guide.md`, `fine_tuning_frameworks_comparison_guide.md`, `ai_agent_platform_selection_guide.md`, `enterprise_agentic_platform_architecture_guide.md`, `ai_governance_framework_guide.md`, `feedback_mechanisms_llm_agents_guide.md`.

**Retrieval/eval shelf:** `technology/ai_llm/rag/rag_evaluation_methodology_guide.md`, `rag_evaluation_tools_comparison_guide.md`, `production_grade_agentic_rag_guide.md`, `advanced_rag_techniques_guide.md`.

**Adjacent axes, distinguished not duplicated:** `technology/ai_native_companies_guide.md` (AI-native business — a different axis, §2.3, §10.4), `technology/ai_llm/long_tail_domains_guide.md` (head/tail of domains — §1.6), `technology/domain_driven_design_guide.md`, `technology/maritime_domain_awareness_guide.md`, `technology/ai_llm/real_world_ai_projects_guide.md`.

**Systems of record and the dated venture claim:** `banking/posting_engine_core_banking_guide.md`, `banking/core_banking_processes_guide.md`, `banking/data_models_banking_insurance_guide.md`, `banking/fix_protocol_guide.md`, and `technology/a16z_big_ideas_2026_guide.md` §7 (Sarah Wang, systems of record — dated, attributed, quoted not adopted).

### 16.4 Closing summary

The build half of a vertical agent — the loop, the harness, the tools, the ops, the eval mechanics — is a solved and well-documented discipline, and this guide deliberately restates none of it. What is unowned, and what this guide holds, is the vertical *delta*: a domain corpus that is defensible only when it is proprietary, adjudicated, maintained and embedded (§3); a ground truth that belongs to a domain's experts and to whoever holds the authority to decide what correct is (§4); an acceptance test that preserves the domain's asymmetric errors and its refusal boundaries (§5); a workflow embedding whose switching cost *is* the second half of the moat (§6); a build decision that buys the shared layer and builds the judged one (§7, §8); and a liability frame that keeps the licensed judgement with the licensed person (§9). The economics follow from that structure, not from a printed figure (§10); the failures follow from getting the structure wrong (§11); and the readiness assessment asks only for named inputs, because a vertical without a named corpus, expert, standard and process is a general agent with a new label (§12).

The honest counter-argument stands beside the thesis, not beneath it: a better model can erase a public-corpus vertical, and a shallow ground truth sustains no moat. That is not a refutation of the thesis; it is the thesis stated precisely. The model is shared, commoditised, and improving for everyone — which is why it is the wrong place to look for advantage. What is not shared is the adjudicated correctness no one else has assembled, and the process embedding no one else can cheaply remove.

**Closing line.** the model is the least vertical thing about a vertical agent.
