# Domain-Specific Harnesses - where a domain's standards become executable

> **Byline:** Jack Liu Shurui, Solution Architect  
> **Repo:** personal research library - **Series:** LLM/AI Engineering & Operations - **Domain:** AI Engineering - Agent Harnesses  
> **Date:** October 2026 - **Sources read:** 2026-10-28

> **Standfirst.** This guide owns one question: what changes in an agent harness when the domain the agent works in is specific rather than general. It does not own the harness architecture - the seven ETCLOVG layers, the taxonomy, the trilemma, the ecosystem - because [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) owns that. It does not own the assembly - the project skeleton, the build path, the component-by-component anatomy - because [agent_scaffolding_guide.md](agent_scaffolding_guide.md) owns that. It does not own the vertical's business case - the corpus moat, the ground-truth economics, the buy decision, the workflow embedding - because [building_vertical_ai_agents_guide.md](building_vertical_ai_agents_guide.md) owns that. What is left, and what this guide holds, is the seam: which harness layers a domain changes the *meaning* of rather than merely the *contents* of, what it demands of each, and where a general harness silently fails a domain.
>
> The finding that justifies the file is a dangling pointer. The vertical guide tells its reader, in three separate places, that the harness guides own the harness. The two harness-owning guides contain zero mentions of "domain-specific" between them. The promise and the delivery do not meet, and section 1.5 states the mismatch precisely, with the three quotations and their line numbers.
>
> On the term: "domain-specific harness" is not a product category and not a vendor segment. It is a claim about which layers of a harness changed meaning when the domain was fixed. A harness can carry a domain's tools and a domain's corpus and still be a general harness in every layer that matters. That is the failure this guide is written against.

> **How to read it.** Sections 1 to 3 frame: the decoder, the coverage finding, the boundary, the thesis (1); the four assumptions a general harness encodes and the mechanism by which each is encoded (2); the four facts a domain imposes, translated into harness obligations (3). Section 4 is the analytical core - the seven ETCLOVG layers walked one at a time, each with an honest verdict of *contents* or *meaning*, including the layers where the answer is "contents only" and where no apology is offered for saying so. Sections 5 and 6 take the two layers whose meaning changes hardest - verification and governance - at depth. Section 7 separates what is genuinely bespoke from what is merely configuration, and refuses to inflate either. Sections 8 and 9 are the two decisions that follow: configure, fork or build; and what happens to the artifact when the domain moves. Sections 10 to 13 are practice: failure modes, one fictional worked example, anti-patterns, and an honest state of play. Sections 14 and 15 are the honesty apparatus - claims audit, unverified items, glossary, cross-references. Section 16 closes.

> **Verification policy.** Three marks, and only three. ✅ means verified against a primary source read in this pass (2026-10-28). ❌ marks a negative finding, stated as a negative rather than softened into a hedge. ⚠ marks something asserted but not independently confirmed, or confirmed only from a secondary or vendor source. Every inline factual claim carries one of the three, and section 14 dates and kinds every source behind them. No benchmark score, no vendor improvement claim, no market size, no adoption percentage and no ranking appears anywhere in this file; benchmarks are cited for their *mechanism* only. Where this guide reasons rather than cites, the sentence says so.

---

**Table of contents**

1. [The Overview - the Decoder, the Boundary, the Dangling Pointer and the Thesis](#1-the-overview---the-decoder-the-boundary-the-dangling-pointer-and-the-thesis)
2. [What a General Harness Assumes](#2-what-a-general-harness-assumes)
3. [What the Domain Changes](#3-what-the-domain-changes)
4. [The Layer-by-Layer Analysis - Contents or Meaning](#4-the-layer-by-layer-analysis---contents-or-meaning)
5. [Verification, in Depth](#5-verification-in-depth)
6. [Governance, in Depth](#6-governance-in-depth)
7. [Context and Tools, Honestly](#7-context-and-tools-honestly)
8. [The Build Decision](#8-the-build-decision)
9. [The Portability Question](#9-the-portability-question)
10. [The Failure Modes](#10-the-failure-modes)
11. [The Cymbal Bank Worked Example](#11-the-cymbal-bank-worked-example)
12. [The Anti-Patterns](#12-the-anti-patterns)
13. [The State of Practice](#13-the-state-of-practice)
14. [The Claims Audit](#14-the-claims-audit)
15. [What Could Not Be Verified, Glossary and Cross-References](#15-what-could-not-be-verified-glossary-and-cross-references)
16. [The Closing Summary](#16-the-closing-summary)

---

## 1. The Overview - the Decoder, the Boundary, the Dangling Pointer and the Thesis

### 1.1 The question this guide owns

The question is not "what tools does a domain need", and it is not "what should the domain prompt say". Both of those are configuration questions, answerable in an afternoon by anyone with the domain's vocabulary, and both are answered badly all over the market because they are the easy questions to answer. The question this guide owns is narrower and harder:

**Which harness layers does a domain change the *meaning* of, rather than merely the *contents* of?**

The distinction is the whole guide, so state it plainly. *Contents* change means the layer receives different material and behaves as it always did: a domain adds twelve tools and the tool layer registers twelve tools; a domain adds a sandbox egress rule and the sandbox enforces one more rule. Nothing about the layer's contract, its success criterion, or its failure mode moves. *Meaning* change means the layer's own definition of correct is now supplied by the domain, and a harness that satisfies the general definition satisfies nothing: the verification layer's job stops being "score the output" and becomes "score the output against an authority the domain recognises", and a harness that scores without that authority is not a weaker verifier, it is a different instrument measuring a different thing.

A guide that answered the configuration question would be short and useless. A guide that answers the meaning question has to argue layer by layer, and has to be willing to conclude, for some layers, that nothing changed. Section 4 does that, adjusts the opening hypothesis for one layer at 4.6, and records at 4.11 where the thesis survives only in a weakened form.

### 1.2 The decoder

Seven terms carry the argument. Each is defined here narrowly, in the sense this guide uses it, with a pointer to the guide that owns the fuller treatment. Where a term is used in two senses in the literature, both are given, because the overloading is a real source of confusion rather than a pedantic complaint.

| Term | What it means in this guide | Fuller treatment |
|---|---|---|
| **Harness** | Two senses. (1) The **agent harness**: the infrastructure surrounding the model that turns model calls into bounded, stateful, tool-mediated task execution - execution substrate, tool interface, context control, orchestration, observability, verification, governance. (2) The **eval harness** (also task harness): the rig that presents a task to an agent, runs it in a controlled environment, and scores the attempt. This guide means sense (1) throughout, and says "eval harness" explicitly when it means sense (2). | section 2.1 of [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) for sense (1); section 1.3 below for the distinction |
| **Scaffold** | The project skeleton an agent is built on: the directory layout, the config file, the tool-registration module, the prompt module, the memory module, the entry point, the test file. The scaffold is what you start from; the harness is what runs. | [agent_scaffolding_guide.md](agent_scaffolding_guide.md), section 1 and section 2 |
| **Domain** | A body of practice that has its own definition of correct, its own authority structure (who may say what is correct), its own currency rules (which version of the rule is in force), and its own liability if the answer is wrong. A domain is not a topic and not a vocabulary. | section 3; [building_vertical_ai_agents_guide.md](building_vertical_ai_agents_guide.md) section 2 |
| **Vertical** | The commercial axis: an agent sold into one industry, with one industry's budget-holder, workflow and liability frame. A vertical usually implies a domain, but the two are independent - a domain agent can be built in-house with no market, and a vertical can be sold with a thin domain claim. | [building_vertical_ai_agents_guide.md](building_vertical_ai_agents_guide.md) section 2.1, section 2.3 |
| **Layer** | One of the seven ETCLOVG engineering surfaces: Execution, Tooling, Context, Lifecycle, Observability, Verification, Governance. A layer is a surface with its own contract, not a component in a diagram. | [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) section 2.1 |
| **Guardrail** | A runtime control that inspects or constrains an action or an output before it takes effect. In general use, a content classifier over text against a safety taxonomy. In domain use, potentially a compliance control with a subject, a condition, an evidence record and an owner. The gap between those two sentences is section 6. | section 6; [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) section 8 |
| **Eval** | A scored suite: a set of inputs, an expected outcome per input, and a scoring procedure that produces a number or a verdict. An eval is only as good as the provenance of its expected outcomes, which is why a domain eval is a different kind of object from a general one. | section 5; [building_vertical_ai_agents_guide.md](building_vertical_ai_agents_guide.md) section 5 |

### 1.3 The two senses of "harness", and why the distinction is load-bearing

The word is overloaded in a way that quietly corrupts arguments. METR's Task Standard defines a task as three things: an environment for the agent to interact with, a string of instructions, and optionally a way to automatically score the attempt based on a submitted string and the state of the environment afterwards ✅ (METR Task Standard, README, specification repository archived 7 October 2026). That is the eval-harness sense: the harness is the thing that *runs and scores a task*, and the agent under test is a component inside it.

The agent-harness sense is the inverse. The harness surrounds the model and the agent is the whole assembly; the eval is one layer of it. Anthropic's engineering post on long-running agents calls the Claude Agent SDK "a powerful, general-purpose agent harness" and treats compaction, progress files and git history as harness features ✅ (Anthropic engineering blog, November 2025). That is sense (1).

Why it matters here: a domain changes both senses, but by different mechanisms. It changes the eval harness by supplying a scoring authority (section 5). It changes the agent harness by supplying rules the governance layer must enforce and context the retrieval layer must rank by authority (sections 4, 6, 7). Conflating the two produces the most common category error in vendor material - the claim that a domain-tuned *evaluation* proves a domain-capable *harness*, when the eval is scoring a different definition of correct than the one the domain uses.

### 1.4 The boundary, and the coverage finding

**The boundary, declared by name and section.** Four guides, one seam, and no overlap by design.

| Guide | Owns | Sections it owns | What this guide takes from it |
|---|---|---|---|
| [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) | **The harness architecture** - the layers, the taxonomy, the trilemma, the ecosystem | sections 2 and 2.1 (ETCLOVG at a glance, the seven-layer table); 3 (E and T); 4 (C); 5 (L); 6 (O); 7 (V); 8 (G); 9 (cross-layer synthesis: trilemma, capability-control, coupling); 11.3 (what that guide adds and the gaps it flags); 12 (the Cymbal harness assessment) | The layer contracts. Cited by section, re-derived nowhere |
| [agent_scaffolding_guide.md](agent_scaffolding_guide.md) | **The assembly** - the project structure and the build path | sections 1 (concept); 2 (anatomy); 3 (known projects); 4 (principles); 5 (components); 6 (enterprise and banking); 7 (worked scaffold); 8 (trends); 9 (glossary) | The build path and the component layout. Cited, not repeated |
| [building_vertical_ai_agents_guide.md](building_vertical_ai_agents_guide.md) | **The vertical** - the business, the corpus, the ground truth, the liability | sections 3 (corpus moat); 4 (ground truth adjudication); 5 (evaluation by domain standards, the refusal test); 6 (workflow embedding); 7 (build decision); 8 (integration); 9 (liability and professional duty); 10 (economics); 11 (failure modes); 12 (readiness); 13 (the Cymbal vertical) | The domain's argument, translated into harness obligations in section 3. Cited, not re-argued |
| **This guide** | **The seam** - what changes in the harness when the domain is specific | sections 4 to 9 | - |

If a paragraph here is about harness architecture in general, it belongs to the first guide. If it is about the vertical's business case, it belongs to the third. What remains - and what sections 4 to 9 hold - is the seam: which layers the domain reaches into, what it demands of each, and where a general harness silently fails.

**The coverage finding.** The repository pass of 2026-10-28 counted matching lines, not occurrences, across 679 markdown files. The counts below are reproduced as measured; they are the reason this file exists rather than a paragraph inside an existing one.

| Measure (matching lines, `grep -c`) | Count |
|---|---|
| Markdown files in the repository | 679 |
| `domain.specific` | 65 |
| `harness` | 109 |
| `agent scaffolding\|scaffold` | 73 |
| `system prompt\|tool definition` | 59 |
| `eval suite\|evaluation harness` | 33 |
| `ETCLOVG` | 5 |
| `ETCLOVG\|harness layer` | 7 |
| `vertical ai\|vertical agent` | 2 |

The two guides that own the harness have no domain content at all:

| Guide | Lines | `harness` matching lines | `domain.specific` matching lines |
|---|---|---|---|
| [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) | 777 | 146 | **0** |
| [agent_scaffolding_guide.md](agent_scaffolding_guide.md) | 934 | 17 | **0** |

And the only place where "domain-specific" and "harness" appear on the *same line* is three files, of which exactly one is a substantive discussion: the vertical guide. The other two - `technology/deterministic_engineering_guide.md` and `technology/antithesis_guide.md` - are incidental collisions where "harness" is the *test* harness in the systems-engineering sense sitting next to "domain-specific" in a systems sense. Same-line proximity is not a join. The repository's actual coverage of this subject is one guide's passing references, and that guide explicitly disclaims the subject. State it precisely: a same-line join, three files, one substantive.

### 1.5 The dangling pointer

The vertical guide declares its boundary by name, and in doing so it makes a promise it does not keep - not through error, but because the guide it points at has never contained the material. Three quotations, at the line numbers measured this pass:

> "It is deliberately not a guide to the agent loop, the harness, the scaffolding, memory, tools, context assembly, evaluation tooling, or production operations."
> - [building_vertical_ai_agents_guide.md](building_vertical_ai_agents_guide.md), section 1.1, line 42

> | Harness / scaffold | `technology/ai_llm/agent_harness_engineering_guide.md`, `agent_scaffolding_guide.md` | that a harness exists and what it does; cited, not rebuilt |
> - [building_vertical_ai_agents_guide.md](building_vertical_ai_agents_guide.md), section 1.6, line 106

> "The vertical evaluates the refusal behaviour as a first-class metric (section 5.2), which most generic harnesses do not frame as a domain requirement."
> - [building_vertical_ai_agents_guide.md](building_vertical_ai_agents_guide.md), section 7.4, line 426

Quotations are verbatim, with one mechanical exception: the section sign in the original text is rendered as the word "section" throughout this file, which is otherwise ASCII-only by house rule. No wording is altered.

Read the three together and the structure of the gap is visible. The first sentence hands the harness away wholesale. The table row names the receiving guides as the owners of "that a harness exists and what it does". The third sentence then observes, in passing, that generic harnesses do not frame refusal as a domain requirement - which is precisely a statement *about the seam*, made in a guide that has just disclaimed the seam.

The pointer is not wrong in intent. The harness guides do own the harness, thoroughly: the taxonomy, the layers, the ecosystem, the trilemma, the coupling problem. The pointer is wrong in fact, because neither receiving guide has a single section, paragraph or sentence about what a domain does to a harness. A reader who follows the pointer arrives at 777 lines of layer architecture that never mentions the word "domain", and reasonably concludes one of two things: that nothing changes (false), or that the question belongs to someone else (also false, since the vertical guide has just declined it).

That is what a dangling cross-reference does. It does not produce an error message. It produces a reader who believes the subject is covered.

### 1.6 The thesis

**The harness is where a domain's standards become executable - or where they quietly stop applying.**

The argument in one paragraph: a general harness encodes generic assumptions about tools, context, verification and governance. A domain's standards live in exactly those four places - the documents it treats as authoritative, the tools it permits, the way it decides correctness, and the rules it enforces. A harness not built for the domain will therefore not merely underperform. It will run a suite, produce a green result, and pass a test that has nothing to do with the domain's definition of correct. The failure is not a low score. The failure is a high score on the wrong instrument, delivered with the confidence that a green build confers.

A thesis that cannot lose is not a thesis, so here is how this one can lose. If the evidence showed that domain standards live *outside* the harness - entirely in the corpus and the prompt, with every layer behaving generically - then the thesis would be false, and this guide should have been a corpus guide. Section 4 tests that possibility layer by layer, and section 4.11 records where the thesis survives only in a weakened form. Section 9.2 records where it inverts: a domain harness can be *worse* than a general one, because its errors are confident rather than vague.

### 1.7 What this guide is not

It is not a harness tutorial; it assumes the reader knows what a tool call, a context window, a trace and an eval are, and it re-derives none of them. It is not a domain survey; it names no vertical's business model and argues no market. It is not a compliance mapping; section 6 names two regimes to establish that logging and oversight are harness obligations under law, and explicitly does not assert that either applies to any institution, and gives no legal advice. It is not a vendor comparison; no product is ranked, scored or recommended. And it is not a scaffold guide - if the question is "what files do I create", the answer is in [agent_scaffolding_guide.md](agent_scaffolding_guide.md) section 2 and section 7, and this guide will not repeat it.

## 2. What a General Harness Assumes

A general harness is not neutral. It is a set of decisions about what the interesting problems are, frozen into interfaces. This section names four of those decisions, why each is reasonable for the purpose it was made for, why it is wrong for a domain, and - the part that makes the problem tractable - *where in the artifact the assumption is encoded*. An assumption encoded in a schema can be replaced by a schema. An assumption encoded in a metric cannot be replaced without changing what the harness measures.

### 2.1 Assumption one: a general tool set

**The assumption.** A capability is a function with a name, a description, a JSON schema for arguments, and a structured return. Any domain capability can be expressed this way, and the hard problem is the model's choice among tools, not the tools themselves.

**Why it is reasonable.** It is the assumption that made the tool-calling interface universal. Anthropic's guidance on agent-computer interfaces is explicitly about exactly this: tool definitions should include example usage, edge cases, input format requirements and clear boundaries from other tools, and the authors report spending more time optimizing tools than the overall prompt when building their SWE-bench agent ✅ (Anthropic, "Building effective agents", December 2024). For general-purpose work the assumption holds well: a file read is a file read.

**Why it is wrong for a domain.** Because the domain's most important tool fact is not its schema but its *authority*: whether invoking it constitutes a read or an act, whether it moves money, whether it creates a record that a third party will later rely on. A general tool layer has no representation for "this tool may be called by an agent, and that tool may only be called by a human, or by an agent with a second approval". It has a schema and a scope. The distinction is real and it is not a schema distinction - which is why section 4 reallocates it to governance rather than pretending it is a tooling problem.

**Where it is encoded.** The registry entry, the argument schema, and the per-tool auth scope. All three are replaceable by configuration, which is precisely why "domain-specific tools" is such a weak claim (section 7.3).

### 2.2 Assumption two: a general context window

**The assumption.** Context is a relevance problem. Fetch the chunks most similar to the query, rank them, fit them in the window, compact when full.

**Why it is reasonable.** For open-domain question answering it is close to correct, and the entire retrieval industry is built on it. The default is defensible because in most corpora the most similar passage is the most useful passage.

**Why it is wrong for a domain.** Because a domain corpus has a property general corpora do not: *authority order that is independent of, and can invert, lexical similarity*. The superseded guidance is frequently the most similar document to the question - it was written to answer that question, in the domain's own words, before it was withdrawn. A retriever optimising similarity will rank it first, confidently, and the harness will have no way to know it did anything wrong. A domain context layer has to optimise *admissibility*, which is a different objective function with a different failure mode: it will sometimes return the less relevant document because it is the one in force, and that is the correct behaviour.

**Where it is encoded.** The retrieval default (top-k, the similarity function, the chunker), the index's metadata schema, and the compaction policy. The first two are configurable. The third usually is not, because compaction is written to preserve *salient* content, and salience is generic.

### 2.3 Assumption three: an average-case evaluation

**The assumption.** Correctness is a distribution over a benchmark. A suite of tasks with expected outputs, scored and averaged, represents quality; improvement means a higher average.

**Why it is reasonable.** It is the only way to compare systems at scale, and it produced the entire measurement culture of the field. SWE-bench is the canonical case: 2,294 problems from real GitHub issues, resolved by editing the codebase, verified by running the repository's tests ✅ (arXiv 2310.06770, October 2023; ICLR 2024). The mechanism is honest because the oracle is executable - the tests either pass or they do not.

**Why it is wrong for a domain.** Two reasons, and they are separable. First, the average is the wrong statistic when errors are asymmetric: a suite that reports one number cannot express "this error is a regulatory breach and that error is an inconvenience" (section 3.2). Second, and worse, the *expected outputs* are the product of whoever wrote the suite. In a domain, that is the whole question. A general benchmark's labels are cheap; a domain benchmark's labels are adjudications by people with standing to make them, which is why LegalBench's 162 tasks were hand-crafted by legal professionals through an interdisciplinary construction process rather than scraped ✅ (arXiv 2308.11462, August 2023).

**Where it is encoded.** The metric definition and the suite's provenance. The metric is code; the provenance is a document, usually absent.

### 2.4 Assumption four: a default governance posture

**The assumption.** The risks are generic: harmful content, prompt injection, data exfiltration, excessive agency. The controls are therefore generic: a content classifier on input and output, permission scoping on tools, and a human approval gate on actions flagged as dangerous.

**Why it is reasonable.** Those risks are real for every agent, they are the ones with published taxonomies, and a general guardrail is cheap to deploy. Llama Guard is the clean example of the shape: a safety risk taxonomy, a classifier over prompts and responses, with the taxonomy categories adjustable and the output format adaptable ✅ (arXiv 2312.06674, December 2023, vendor paper).

**Why it is wrong for a domain.** Because a domain's rule is usually not a content class. "The recommendation must not be made without a recorded suitability assessment" is a rule with a subject, a condition, a piece of evidence and an owner. No taxonomy over text expresses it, because the offending output can be entirely benign text - indeed, it can be the *correct* text produced at the wrong moment, for the wrong client, without the record. A content classifier will pass it. Section 6 is the full argument.

**Where it is encoded.** The policy file and the hook wiring. This is the assumption that looks most configurable and is least so: the hook points are generic, but what a compliance control needs at a hook point - an evidence record, a version, an approver - is not what a content filter produces.

### 2.5 The meta-assumption: that the four are separable

There is a fifth assumption underneath the other four, and it is the one that makes "just configure it" sound plausible: that a team can change the tools without changing the evaluation, or the context policy without changing the governance, because the layers are independent engineering surfaces.

They are independent as *surfaces* and coupled as *artifacts*. [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) section 9.3 names this as the harness coupling problem: a change in one layer changes the behaviour measured by another, so a harness change is a system change and must be tested as one. The domain makes the coupling tighter, not looser, because the domain's standard is asserted in several layers at once - a single rule about advice may need a context rule (which documents may be cited), a tool rule (which action requires approval), a governance rule (which output is blocked) and a verification rule (which refusal is correct). Configure one and not the others and the harness is internally inconsistent in a way no single-layer test will show.

### 2.6 Summary

| Assumption | Encoded in | Replaceable by configuration? | Where the domain breaks it |
|---|---|---|---|
| General tool set | Registry entry, argument schema, auth scope | Yes | Tool authority (read vs act) is not a schema property; it is a governance fact |
| General context window | Retrieval default, index metadata, compaction policy | Partly - compaction usually is not | Authority order and currency invert similarity |
| Average-case evaluation | Metric definition, suite provenance | The metric, yes; the provenance, no | Asymmetric error, and ground truth that must be adjudicated |
| Default governance posture | Policy file, hook wiring | The wiring, yes; the control semantics, no | A domain rule is not a content class |
| That the four are separable | The architecture's own modularity | No | The domain asserts one standard across several layers at once |

## 3. What the Domain Changes

Four facts about a domain do the work of this section. Each is argued at length in the vertical guide, and none is re-derived here; what this section does is translate each into an obligation on the harness, because that translation is the seam this guide owns.

### 3.1 Error asymmetry

[building_vertical_ai_agents_guide.md](building_vertical_ai_agents_guide.md) section 4.4 makes the point that a domain's errors are not symmetric: a false positive and a false negative do not cost the same, and in most regulated domains one of them is a reportable event while the other is an inconvenience. The translator's obligation follows immediately and is a harness obligation, not a prompt instruction:

**A harness that scores with a symmetric metric cannot represent the domain's cost function.** Accuracy, F1, and even a well-built LLM-judge rubric averaging over a suite all treat the two directions as exchangeable. The domain requires the scoring schema to carry the direction of the error as a first-class field, so that "declined when it should have answered" and "answered when it should have declined" are separate counters with separate thresholds, and so that a regression in the expensive direction cannot be offset by an improvement in the cheap one. This is a schema change in the verification layer, and it is the smallest possible example of a contents-to-meaning shift: same suite, same tasks, different arithmetic, because the domain said the arithmetic was wrong.

### 3.2 Adjudicated ground truth

section 4.2 and section 4.6 of the vertical guide establish that the domain's ground truth is adjudicated rather than given: experts are scarce, they disagree, the disagreement is the normal case rather than a defect, and the architecture that results is expert-in-the-loop rather than expert-as-oracle. The harness obligation:

**The harness must carry the adjudication, not just its result.** A label of "correct" with no record of who decided, on what basis, with what dissent, is unusable in a domain - not because it is imprecise but because it is unevidenced. The concrete requirement is that the golden set is a versioned artifact with a named signing authority, per-item provenance, and a recorded disagreement distribution; and that the scoring path can be run by someone who did not build the agent. Section 5.4 turns this into mechanics. The reason it is a harness obligation and not a data-science nicety: the harness is the only artifact that both holds the set and produces the run record, so if the harness does not carry the provenance, nothing downstream can recover it.

### 3.3 Refusal as a requirement

section 5.2 of the vertical guide argues that the refusal is a domain metric, and section 7.4 observes (line 426) that most generic harnesses do not frame it as a domain requirement. The harness obligation is blunt:

**Refusal must be a modelled outcome, not an error state.** In almost every general harness, a refused or escalated turn is an exception: the tool call failed, the model declined, the run ended without a result. The scoring pipeline treats it as a non-answer and the dashboard shows it as a gap. In a domain where declining is the *correct* behaviour in defined circumstances, that pipeline inverts the truth: the harness records the best behaviour as a failure and pressures the system toward the behaviour the domain forbids. Making refusal first-class means a refusal category in the scoring schema, an expected-refusal label on the tasks where declining is correct, and a distinct treatment in the lifecycle layer so that an escalation is a completed transition rather than an aborted run.

### 3.4 Professional duty

section 9 of the vertical guide argues that the professional duty does not transfer to the tool, and that the load-bearing distinction between assistance and advice is a liability distinction rather than a quality one. The harness obligation is the least intuitive of the four:

**The harness must produce the record that the professional signs.** If a human remains accountable for the output, then the harness's job includes making accountability possible: what the agent saw, what it cited, what it recommended, what it declined, who reviewed it, and when. This is an observability obligation with a governance purpose, and it is the reason section 4.5 concludes that "what counts as a complete record" is a meaning change rather than a volume change. A trace that records spans and token counts and not the basis of the decision is complete by the tracing library's definition and empty by the professional's.

### 3.5 The four facts, as obligations

| Domain fact | What it changes about "correct" | Harness obligation | Primary layer | Argument owned by |
|---|---|---|---|---|
| Error asymmetry | Correctness has a direction and a price | Asymmetric scoring schema; separate counters; no netting across directions | V | vertical guide section 4.4 |
| Adjudicated ground truth | Correctness is assigned by an authority | Versioned golden set with named signer, per-item provenance, recorded dissent, independently runnable | V (with C for the corpus the set is drawn from) | vertical guide section 4.2, section 4.6 |
| Refusal as requirement | Declining can be the right answer | Refusal as a scored outcome; expected-refusal labels; escalation as a completed transition | V and L | vertical guide section 5.2 |
| Professional duty | The human stays accountable | Evidence-bearing record: basis, citation, recommendation, review, timestamp | O and G | vertical guide section 9 |

### 3.6 What this translation does not do

It does not tell you which of the four matters most in your domain, because that is a domain judgement and the guide has no standing to make it. It does not tell you that all four apply everywhere: a domain with an executable oracle (a compiler, a tax computation, a reconciliation) may have almost no adjudication problem and a very sharp asymmetry problem; a domain with no regulator may have professional duty without a logging obligation. And it does not claim that a harness which satisfies all four is safe - it claims only that a harness which satisfies none of them is measuring something else. Section 4 takes the four obligations and asks, layer by layer, whether they change the layer's meaning or merely its inputs.

## 4. The Layer-by-Layer Analysis - Contents or Meaning

### 4.1 The test applied to every layer

Three questions, asked of each layer in turn. They are ordered by how much they cost to answer.

1. **Does the layer's contract contain a success criterion that the domain can overrule?** A sandbox promises "bounded by the stated constraints"; a verifier promises "a verdict on correctness". If the domain can restate the criterion, the layer's meaning is in play.
2. **If the criterion changes, does the mechanism change, or only its inputs?** This is the question that separates a real re-engineering job from a configuration task, and it is the question most vendor material skips.
3. **When the harness gets it wrong, is the failure visible as an error or invisible as a pass?** This is the sharpest of the three, and it is the one that connects the layer analysis to the thesis. A contents-only layer fails loudly - the tool is not found, the egress is denied, the schema rejects the argument. A meaning-changed layer fails silently, and the silence is the product: a green suite, a compliant-looking output, an audit that passes because it asked a question the harness could answer.

The verdicts below use two labels only, and they are deliberately coarse. *Contents* means the layer receives different material and behaves as before. *Meaning* means the layer's own definition of correct is now domain-supplied.

### 4.2 E - Execution: contents only

**The layer's contract**, per [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) section 3.1: where agent code runs, and what constraints bound it.

**What the domain changes.** The estate boundary - which systems may be reached from inside the sandbox, at what data classification, from which network position; the approved substrate (a regulated workload may be required to run on infrastructure that is itself in a control scope); residency; and the separation of the evaluation substrate from the production substrate, so that a scoring run cannot touch live records.

**Does the contract change?** No. The promise is "bounded by the stated constraints", and the domain states more constraints. The mechanism - namespaces, mounts, egress allowlists, seccomp profiles - does not know what the code means and is not asked to.

**Failure visibility.** Visible. A denied egress produces a denial, and a denial is an error with a rule attached.

**Verdict: contents only.** No apology. The sandbox is the clearest case in the taxonomy of a layer that a domain reconfigures rather than redefines, and pretending otherwise would inflate the guide.

**One precision, so the verdict is not overread.** A domain may require that a denial be *evidenced* - that the log record which rule fired and against which request. That is a real requirement, and it lands on the observability layer (section 4.6), not on execution. The sandbox supplies the event; the domain supplies the obligation to keep it. Attributing the obligation to E would be the same category error as attributing a compliance rule to a tool schema.

### 4.3 T - Tooling: contents only, and explicitly configuration

**The layer's contract**, per section 3.4 and section 3.5 of the harness guide: how external capabilities are described, discovered, selected and invoked.

**What the domain changes.** The nouns in the schemas (`get_facility`, `screen_party`, `fetch_disclosure_text`), the number of tools, the auth scopes, the return shapes. All of it is real work and none of it is a change to the layer.

**Does the contract change?** No. Describe, discover, invoke. A domain tool is described the same way a general tool is described, discovered by the same registry, invoked over the same transport. The Model Context Protocol is the clearest statement of the layer's self-understanding: it standardises how tools and context are exposed, and it says of its own security principles that "MCP itself cannot enforce these security principles at the protocol level" and that implementors *should* build consent, access control and documentation around them ✅ (MCP specification 2025-06-18). A protocol that explicitly delegates enforcement is telling you where the layer ends.

**The reallocation that keeps this honest.** The domain's most important tool facts - is this call a read or an act; may an agent call it or only a human; must a second approval exist; what must be recorded when it fires - are *not* tooling facts. They are governance facts that happen to be annotatable on a tool. Section 4.8 owns them. If they are implemented in the tool adapter (an `if tool == "move_money"` branch inside the wrapper), the rule exists, but it exists unevidenced, unversioned, and invisible to the policy layer - which is failure mode (g) in section 10.

**Failure visibility.** Visible. Wrong argument, missing scope, unknown tool: all produce errors.

**Verdict: contents only, and this is the layer the market inflates most.** "Domain-specific tooling" in a vendor deck usually denotes a REST wrapper with domain nouns and a system-prompt paragraph describing when to use it. The test that settles it: swap the domain tool set for an equivalently shaped generic set. If the harness still passes its suite, the tools were configuration. If it fails, the domain semantics were encoded in the adapters, which is not a virtue - it means the rules live somewhere the policy layer cannot see.

### 4.4 C - Context: meaning changes

**The layer's contract**, per section 4.1 and section 4.2 of the harness guide: what the model can see over short-term, session and persistent horizons, and how that is selected and compressed.

**What the domain changes.** Four properties, none of which a similarity score represents:

- **Authority.** Which document governs when two documents disagree. A regulator's rule outranks an internal policy, which outranks a desk convention, which outranks a vendor's FAQ. The order is a property of the domain, not of the text.
- **Currency.** Which version is in force, from which date, and what superseded what. The corpus is not a set of documents; it is a partially ordered set with effective dates and supersession edges.
- **Provenance.** Which clause is the operative one, and the obligation to carry it into the answer as a citation rather than paraphrasing it away. A domain answer without its basis is not a weaker answer; it is an unusable one.
- **Negative knowledge.** What the domain says may *not* be surfaced or relied on: the withdrawn circular, the template that was never approved, the jurisdiction the firm does not operate in. A retriever has no representation for "must not be returned" - only for "ranked lower", which is a different and weaker statement.

**Does the contract change?** Yes, and this is the point that looks solved and is not. The layer's implicit objective is *relevance*: return the passages most similar to the query. The domain's objective is *admissibility*: return the passages the domain permits as a basis, ordered by authority, current as of the transaction date, with their citations attached. Those objectives disagree in a specific and predictable way - the superseded guidance is often the most lexically similar document to the question, because it was written to answer that question in the domain's own words before it was withdrawn.

**Mechanism change, not just input change.** The index needs metadata the general index does not carry (authority tier, effective-from, effective-to, supersedes, superseded-by, exclusion flag), the assembly step needs a precedence resolver rather than a top-k cut, and the output path needs citation binding so that the claim and its basis travel together. That is three mechanisms, not one configuration file.

**Failure visibility.** Invisible. A superseded document retrieved as current produces a fluent, well-cited, confidently wrong answer, and every general metric - groundedness, citation validity, answer relevance - scores it well, because the citation really does support the claim. This is the archetype of the thesis.

**Verdict: meaning changes.**

**What is *not* the test.** Corpus size is orthogonal. Two hundred documents with an authority structure and effective dates is a domain corpus. Twenty million undifferentiated chunks is a large retrieval set, and no amount of it supplies currency or negative knowledge. [building_vertical_ai_agents_guide.md](building_vertical_ai_agents_guide.md) section 3.5 adds the decay dimension: the corpus ages, and a domain harness that does not date its sources ages with it.

### 4.5 L - Lifecycle: contents, with a domain-supplied termination predicate

**The layer's contract**, per section 5.1 of the harness guide: the control flow that reads and writes state, single-agent to multi-agent, plus operational state (checkpoint, resume, handoff).

**What the domain changes.** Three things. The **set of approval points**: which decisions require an authorised human, at what level of authority, and what the escalation path is when that human is unavailable. The **stopping conditions**: a general agent stops when the task looks done; a domain agent must also stop when it reaches the boundary of its authorisation, even though it could continue - the assistance-to-advice line in the vertical guide's section 9.2 is exactly this boundary expressed as a termination predicate. And the **handoff contract**: what the receiving human must be told, which is a domain's list, not a generic summary.

**Does the contract change?** The mechanism does not: pause, checkpoint, escalate, resume, hand off. But the *predicate* that decides when to do each is domain-supplied, and a predicate is not an input - it is the layer's decision logic. That is why this section does not get the clean "contents only" label.

**Failure visibility.** Partly visible. A run that completes when it should have paused looks like a success in every dashboard: no error, no exception, a result.

**Verdict: contents, with a domain-supplied termination predicate.** The honest phrasing matters. The layer's machinery is unchanged and a team that tried to rebuild it for a domain would be wasting money; the layer's *stopping rule* is domain, and a team that left the default in place has a harness that will advise when it should assist.

### 4.6 O - Observability: meaning changes, and this adjusts the opening hypothesis

**The layer's contract**, per section 6.1 of the harness guide: system-wide traces, costs, failures and reliability signals.

**What the domain changes.** Not the volume of telemetry. The *definition of a complete record*. The domain's question is not "what happened" but "can an accountable person reconstruct the decision and their own oversight of it from this record, months later, to an evidentiary standard".

The law is explicit that this is a property of the system rather than of the model. Article 12 of Regulation (EU) 2024/1689 requires high-risk AI systems to "technically allow for the automatic recording of events (logs) over the lifetime of the system", and requires logging capabilities that enable recording of events relevant for identifying risk situations, and for "the identification of the natural persons involved in the verification of the results, as referred to in Article 14(5)" ✅ (EU AI Act, Article 12(1) and 12(3)(d); in force 1 August 2024). The application dates are the one caveat, because this is the part of the Act with a moving timetable: the text as amended and currently published applies the high-risk obligations from 2 December 2027 for Annex III systems and 2 August 2028 for Annex I, where the regulation as originally published put them at 2 August 2026 and 2 August 2027 ✅ (Article 113(c) as amended, read 2026-10-28; the amendment is pending-sensitive, so treat the dates as current rather than settled ⚠). Article 13 then requires the instructions for use to describe the mechanisms that let deployers "properly collect, store and interpret the logs" ✅ (Article 13(3)(f)).

**Does the contract change?** Yes. A trace schema is a definition of completeness. An OpenTelemetry-shaped trace defines a complete record as full span coverage: every model call, tool call, retry and handoff, with timings and token counts. The domain defines a complete record as one from which a named human's verification can be identified and reconstructed. Those are different schemas, and no amount of span coverage produces the second: spans record that a tool ran, not that a person with authority approved the result, on what basis, against which rule version.

**Why this section departs from the opening hypothesis.** The candidate verdict this guide started with was that O "drags toward meaning" - contents mostly, with a regulatory pull. That undersells it, and the argument that changed the verdict is the one above: the layer's contract is "a complete record of what happened", and *completeness* is precisely the term the domain redefines. That the transport (OTel, a log store, a SIEM) is generic is true and irrelevant - a generic transport carrying a domain-complete schema is a domain observability layer, and a domain transport carrying a generic schema is not. Transport genericity is not layer invariance.

**Failure visibility.** Invisible until audit, which is the worst possible time.

**Verdict: meaning changes.**

### 4.7 V - Verification: meaning changes

**The layer's contract**, per section 7.1 of the harness guide: turning tasks and traces into evaluation, failure attribution and regression feedback.

**What the domain changes.** The scoring authority. The layer's promise is "a verdict on correctness", and correctness is not a property the layer can invent. In a general harness it is inherited from the benchmark, which inherits it from whoever built the benchmark. In a domain it must be supplied, with provenance, by someone with standing. Section 5 is the full argument; the verdict here is that this is the least contestable meaning change in the taxonomy, and also the one most often simulated rather than performed.

**Failure visibility.** Invisible, and worse than invisible: a passing suite actively misleads, because it is evidence of internal consistency presented as evidence of correctness.

**Verdict: meaning changes.**

### 4.8 G - Governance: meaning changes

**The layer's contract**, per section 8.1 of the harness guide: constraining behaviour through permission, identity, policy, hardening, audit and human oversight.

**What the domain changes.** Whose rules these are, and what counts as evidence that they were enforced. A general harness's policy file is written by the deploying team about generic risks. A domain's policy is written by a regulator, a professional body or a contract, transcribed into controls, and it must survive the question "by what authority". Section 6 is the full argument.

**Failure visibility.** Invisible. A content filter that passes a compliant-looking but non-compliant output produces no signal at all - it produced a "pass", which is the layer's normal output.

**Verdict: meaning changes.**

### 4.9 The two axes that predict the verdicts

The seven verdicts are not arbitrary, and naming what produces them makes the analysis portable to a domain this guide has not seen. Two properties of a domain predict, before any engineering, which layers will change meaning.

**Axis one: oracle availability.** Does the domain possess a check that decides correctness without human judgement - a test suite, a reconciliation, a rule engine, a state comparison? Where it does, the verification layer barely changes: the domain's correctness is computable and the harness's generic scoring machinery is adequate. This is why coding agents received domain-shaped harnesses first: software is the domain with a free oracle, and SWE-bench's mechanism - edit the repository, run the tests - is a domain eval that a general harness can host without modification ✅ (arXiv 2310.06770, October 2023). Where correctness is adjudicated rather than computable, V changes meaning, and the harness must carry an authority it cannot compute.

**Axis two: liability exposure.** Does an error expose the operator to a regulator, a professional body or a contract? Where it does, G and O change meaning, because the rules must be attributable and the record must be evidentiary. Where it does not, G can be a genuinely generic posture - acceptable use, data handling, injection defence - and O can be ordinary telemetry.

|  | **Executable oracle** | **Adjudicated correctness** |
|---|---|---|
| **High liability exposure** | V near-contents (the oracle decides); G and O change meaning. Rare, and usually the easiest domain to build a credible harness for. | V, G, C and O all change meaning. The hardest case, and the one where a general harness fails most quietly. |
| **Low liability exposure** | V near-contents; G and O contents. A general harness with domain tools genuinely suffices. | V and C change meaning; G and O may not. Internal research and analysis tools live here. |

The bottom-left cell deserves emphasis because it is where this guide's thesis does not bite. A team building an internal coding agent has a free oracle, low liability exposure, and no domain rule that a policy file cannot express. For them, a general harness configured with domain tools is the correct engineering answer, and a guide that told them to fork would be costing them money to satisfy a thesis.

### 4.10 The verdict table

| Layer | What the domain changes | Contents or meaning | Does the mechanism change? | Failure visibility |
|---|---|---|---|---|
| **E** Execution | Estate boundary, substrate approval, residency, eval/prod segregation | Contents | No | Visible |
| **T** Tooling | Names, schemas, scopes, count; authority reallocated to G | Contents (configuration) | No | Visible |
| **C** Context | Authority, currency, provenance, negative knowledge | **Meaning** | Yes - index metadata, precedence resolution, citation binding | Invisible |
| **L** Lifecycle | Approval points, handoff contract, stopping conditions | Contents, plus a domain-supplied termination predicate | Partly - predicates are decision logic | Partly visible |
| **O** Observability | What counts as a complete, evidentiary record | **Meaning** (adjusted from the opening hypothesis) | Yes - schema and retention, not transport | Invisible until audit |
| **V** Verification | The scoring authority and the direction of error | **Meaning** | Yes - scoring schema, golden-set provenance, adjudication | Invisible, and misleading |
| **G** Governance | Whose rules, and what counts as evidence of enforcement | **Meaning** | Yes - controls with subject, condition, evidence, owner | Invisible |

Read the table as a map of where money should go. Two layers absorb domain content at configuration cost, one sits between with a domain-supplied predicate, and four change meaning - and each of those four fails silently, or silently until an audit, which is worse. The lifecycle layer's failure is the one that will be discovered by a customer rather than by a test.

### 4.11 Where the thesis is qualified

The thesis as stated in section 1.6 is **"the harness is where a domain's standards become executable - or where they quietly stop applying."** The layer analysis supports it, but not uniformly, and the differences are worth stating rather than smoothing.

**Qualification one: the strong form holds for three layers, not seven.** For V, G and C the claim is exact: the standard genuinely stops applying, silently, in a harness that was not built for it. For O it holds at the schema level and not at the transport level. For E and T the strong form is simply false - a general harness does not fail a domain there, it merely does not help. Nothing "stops applying" in the sandbox, because the domain's standard was never expressed in the sandbox to begin with. Saying so costs the thesis some sweep and buys it the credibility to be believed about V and G.

**Qualification two: where an oracle exists, V stops being the interesting layer.** The thesis's mechanism - a suite that has nothing to do with the domain's definition of correct - presupposes that the definition is not computable. Where it is computable, the general suite and the domain suite converge, and the thesis has little to say. The mechanism is a claim about adjudicated correctness, not about domains in general.

**Qualification three: the failure is not always quieter.** Section 9.2 argues the inversion - a domain harness can fail *more* confidently than a general one. If confidence is the measure, the thesis is about a different axis than it first appears: not loud versus quiet, but *attributable* versus *unattributable*. A general harness that fails gives you nothing to reason with; a domain harness that fails gives you a rule, a version and an evidence trail pointing at the wrong conclusion. Which is worse depends on whether you need to explain the failure or prevent it.

The refined statement, which section 16 carries forward: **the harness is where a domain's standards become executable, and the layers where that happens are predictable from two properties of the domain - whether correctness is computable, and whether an error is accountable.**

## 5. Verification, in Depth

### 5.1 Why a general eval cannot score a domain claim

A general eval asks a question like "did the agent complete the task", "does the answer match the reference", or "does a judge model rate this response as good". Each of those is answerable only relative to a definition of correct, and in a general harness that definition arrives by inheritance: from the benchmark's authors, from a crowd of annotators, or from a judge model's preferences. None of the three has standing in a domain.

Three things break, and they break in order.

**The question is wrong before the answer is.** A general suite asks whether the agent did the task. A domain asks whether the answer is one the domain would permit - which is not the same predicate. An agent can complete the task of drafting a client recommendation, in fluent prose, with accurate figures, and produce something the domain forbids, because the disclosure was the wrong version or the suitability record was absent. The suite scores completion. The domain scores permission. A harness that has only the first has no way to express the second, and no way to notice it is missing.

**The ground truth is a label, and the domain's is an adjudication.** A label is a fact about a dataset. An adjudication is an act by a person with standing, recorded with a basis, and revisable. The difference is not philosophical: a label can be averaged, and an adjudication cannot, because averaging away the dissent destroys the information the domain needs (section 5.5).

**The versioning is against the wrong calendar.** A general suite is versioned against a benchmark release; its numbers are comparable because the suite is frozen. A domain suite is versioned against rules that change on a legislator's or a regulator's calendar, and a frozen suite becomes a false statement about the present the moment the rule it encodes is amended. The domain harness needs a suite-to-rule mapping that a benchmark release cadence cannot provide.

### 5.2 What the domain must supply

Five artifacts, and the harness has to hold all five or the eval is decorative:

| Artifact | What it is | Where it lives in the harness |
|---|---|---|
| **The rule text** | The policy, regulation, standard or contract clause whose satisfaction is being tested | Versioned reference corpus; cited by the scoring schema |
| **The rubric** | How a human applies the rule to an output - the operative test, not a summary | Scoring code plus a written rubric the signer approved |
| **Adjudicated examples** | Cases where an authorised person decided correct or incorrect, with the basis recorded | The golden set, with per-item provenance |
| **Refusal categories** | The circumstances in which declining is correct, named and enumerated | Expected-outcome labels; scoring schema |
| **The boundary set** | Cases deliberately at the edge of the domain's rules, where reasonable experts differ | Held-out set, reported separately, never averaged in |

The fifth is the one teams skip. A suite composed only of clear cases measures whether the agent can do the easy thing; the domain's risk lives at the boundary, and a suite without boundary cases reports a number that is structurally optimistic. The vertical guide's section 4.3 argues that disagreement between experts is the normal case; the harness consequence is that disagreement must be *sampled*, not assumed away.

### 5.3 The tau-bench mechanism, cited for mechanism only

The best published example of a domain eval shipping its own scoring authority is tau-bench. It emulates dynamic conversations between a simulated user and an agent that is given "domain-specific API tools and policy guidelines", and it evaluates by comparing the database state at the end of the conversation against an annotated goal state, with a reliability metric over multiple trials ✅ (arXiv 2406.12045, June 2024).

Three mechanism lessons, no scores:

1. **The policy document is an input to the agent and the reference for scoring.** The same artifact that tells the agent what it may do is what the eval tests it against. This is what makes the eval domain-shaped rather than generic: the rule text is not background, it is the specification.
2. **The check is a state comparison, not a text comparison.** The agent's prose is not graded. What is graded is whether the world ended in the state the policy permits. This is a far stronger oracle than any rubric, and it is available only because the domain's transactions have a database behind them.
3. **Reliability is scored over repeated trials, not just success once.** A domain that cannot tolerate intermittent correctness needs a metric that expresses consistency, and a single-run pass rate does not.

The lesson has a hard boundary, and it is the most useful sentence in this section: **tau-bench works because the policy is text but the oracle is executable.** The agent reads prose and the scorer compares database rows. A domain whose correctness cannot be reduced to a state comparison cannot copy the mechanism - it can only copy the first half, the policy document, and then it must adjudicate the second half by hand. That is the fork in the road between a domain eval that scales and one that is a standing expert panel.

### 5.4 Expert-built benchmarks: the provenance is the product

Two benchmarks make the same point from different directions.

LegalBench was built through an interdisciplinary process in which legal professionals designed and hand-crafted the tasks - 162 of them, covering six types of legal reasoning - and the paper's justification is explicit: because subject-matter experts took the leading role, the tasks measure reasoning that is either practically useful or professionally interesting ✅ (arXiv 2308.11462, August 2023). The value is not the size. It is the authorship.

FinanceBench supplies the second mechanism: its 10,231 questions ship with "corresponding answers and evidence strings", and the questions are described as ecologically valid and intended as a minimum performance standard, with human review of model answers ✅ (arXiv 2311.11944, November 2023). The evidence string is the transferable idea: the expected outcome is not only *what* the answer is but *what supports it*, so an answer that reaches the right number from the wrong document is distinguishable from one that reached it correctly.

For a harness, those two mechanisms become two requirements: the golden set must record **who adjudicated each item and on what basis**, and each item must carry **the evidence the answer is required to cite**. Neither is a scoring metric. Both are schema.

### 5.5 The golden set as a harness artifact

The mechanics are unglamorous and they are where domain evals actually fail. Six properties, each of which a general harness has no place to store:

- **Held out, and held out from the people who build.** A set the builders can see is a set they will optimise against, and the optimisation is invisible because it looks like diligence.
- **Sealed with a named signer.** Not "the team" and not "the model risk function" - a role and a person, with a date, because the first audit question is who certified this and the answer must be a name.
- **Versioned against the rule version.** Every item records the rule text version it was adjudicated under. A rule amendment produces a re-adjudication queue, not a silent drift.
- **Disagreement recorded, not resolved away.** Where two adjudicators differed, both judgements are stored, and the item carries a flag. A suite that reports only the majority label has thrown away the information that predicts production failure.
- **Independently runnable.** Someone who did not build the agent can run the suite and get the same verdict. This is the operational meaning of independence in a model-risk regime (section 6.5), and it is a harness property: it requires a pinned environment, a fixed seed policy, and a scoring path that does not depend on the builder's local state.
- **Re-adjudication triggered by events, not by calendar.** A rule change, a professional standard revision, a corpus decay event (vertical guide section 3.5), or a model change (vertical guide section 11.3) each invalidates some items, and the harness must know which.

### 5.6 What a suite that passes without domain ground truth proves

It proves the harness is internally consistent. That the pipeline runs, the tools return, the schema validates, the format holds, the loop terminates, and the scoring code does what the scoring code says. All of that is worth knowing and none of it is the domain's question.

The dangerous part is not the gap. It is the *direction of the signal*. A generic suite rewards fluent, well-formatted, confident output, because those are the properties a general rubric can see. That is precisely the profile of a wrong domain answer: the superseded clause quoted accurately, the recommendation made without the suitability record, the disclosure omitted in a beautifully structured document. The suite does not merely fail to detect the domain error - it actively selects for the presentation that accompanies it.

This is the mechanism behind the thesis, stated as narrowly as it can be: **a green suite is evidence of agreement between the harness and itself.** Where the harness's definition of correct was supplied by the domain, that agreement is worth something. Where it was not, the green is a statement about internal coherence being read as a statement about the world.

### 5.7 Refusal belongs in the schema

Section 3.3 gave the obligation; here is the mechanism. Three changes, all small, all usually absent:

1. A **refusal outcome** in the scoring schema, distinct from failure and from success.
2. **Expected-refusal labels** on the items where declining is the correct answer, drawn from the domain's refusal categories (vertical guide section 5.2), not from a generic harm taxonomy.
3. A **separate report line** for false refusals and false answers, because section 3.1's asymmetry means they cannot share a denominator.

A harness that gets these three right can tell you the thing the domain actually wants to know: not "how often was the agent right", but "in which direction does it fail, and is that the direction we can survive".

### 5.8 The minimum credible domain suite

| Element | Present in a general eval | Required by a domain | Consequence if missing |
|---|---|---|---|
| Rule text, versioned | No | Yes | The suite tests a rule that may no longer exist |
| Rubric approved by a named authority | No | Yes | Scoring is the builder's opinion in code |
| Per-item adjudication provenance | No | Yes | The set cannot be defended in an audit |
| Evidence strings on expected answers | Rarely | Yes | Right answer from the wrong document scores as correct |
| Refusal categories and labels | No | Yes | The best behaviour is recorded as failure |
| Boundary set, reported separately | No | Yes | The reported number is structurally optimistic |
| Suite-to-rule version map | No | Yes | Rule changes silently invalidate the suite |
| Independently runnable scoring path | Partly | Yes | Independence is asserted rather than demonstrated |

Eight rows, and a general harness supplies at most one and a half of them. That is the size of the verification gap, and it is why section 8's first build test is about the oracle rather than about the tools.

## 6. Governance, in Depth

### 6.1 Whose rules the harness enforces

Governance is the layer most often skipped in domain harness work, for a structural reason: it is the only layer whose *content* originates outside the engineering organisation. The rules a domain harness enforces come from three provenances, and they differ in cadence and in what counts as proof.

| Provenance | Example of the instrument | How it changes | What counts as evidence | Who can waive it |
|---|---|---|---|---|
| **The regulator** | A statute, a supervisory statement, a binding technical standard | On a legislative or supervisory calendar, with transition periods | Logs, documented controls, independent validation, notifications | Usually nobody; at most a forbearance or an exemption with conditions |
| **The professional body** | A practice standard, a code of conduct, an accounting or valuation standard | On the profession's cycle, often annually | The professional's own sign-off, and the record they relied on | The professional, explicitly, and never silently |
| **The contract** | A service schedule, a delegation of authority, an outsourced-service agreement | At renewal or variation | Audit rights, reports, records produced on request | The counterparty |

The distinction matters because a harness cannot treat these as one policy file. A regulator's rule cannot be waived by a product owner, a professional duty cannot be delegated to a model, and a contractual obligation may require records the regulator does not ask for. A single "policies.yaml" with a boolean per rule flattens three different authority structures into one, and the flattening is invisible until the first audit asks who approved a specific waiver.

### 6.2 The anatomy of a compliance control

The difference between a guardrail and a control is not the technology. It is whether six things exist.

| Element | Question it answers | Present in a content filter? |
|---|---|---|
| **Subject** | What or who the rule applies to - this output, this client class, this jurisdiction, this product | No - a classifier applies to text |
| **Condition** | The predicate that makes the rule operative | Partly - a taxonomy category is a condition on content, not on circumstances |
| **Evidence** | What record proves the condition was evaluated and the outcome | No |
| **Owner** | The named role accountable for the control being right | No |
| **Version** | Which version of the rule text this control transcribes, and from when | No |
| **Remedy** | What happens on a breach - block, route to a human, notify, unwind | Rarely - a filter returns a score |

A content filter has a condition and an action. A control has all six, and the four it usually lacks are the four an auditor asks about. This is why section 6.6's questions are not adversarial: they are the questions the artifact was never built to answer.

### 6.3 A content filter is not a compliance control

The cleanest way to see the gap is to hold the output constant and vary the rule.

Take a sentence: "Based on your objectives and risk appetite, this fund is suitable for you." A content filter sees a financial-services statement. It contains no prohibited content class: no personal data, no harmful instruction, no toxicity, no prompt injection. A generic guardrail passes it, correctly, by its own definition - and it is the exact output the domain may forbid, because it is a personal recommendation made without a recorded suitability assessment, to a client in a jurisdiction where that assessment is a precondition, without the disclosure version in force.

The rule that catches it has the shape: *subject* = any output to an external party; *condition* = the output contains a recommendation predicate over a product for an identified client AND no suitability record id is attached AND the disclosure version is not the current one; *evidence* = the suitability record id, the disclosure version, the approver; *owner* = a named compliance role; *version* = the rule text and the disclosure text versions; *remedy* = withhold and route to the relationship manager, with the reason recorded.

Nothing in that sentence is a taxonomy category. That is the whole argument of this section. And note the direction of the error: the content filter is not wrong about the text. It is answering a different question - "is this text safe" - and the domain asked "is this act permitted". A harness that only has the first will produce a passing grade on the second question it never asked.

Llama Guard is a fair representative of the general shape and is described by its own authors as incorporating a safety risk taxonomy and performing prompt and response classification, with taxonomy categories adjustable and output formats adaptable ✅ (arXiv 2312.06674, December 2023, vendor paper). The adjustability is real and useful; it lets a deployer add categories. It does not let the deployer express a subject, an evidence record, or an owner's accountability, because those are not categories.

### 6.4 The law names the harness, not the model

The strongest available evidence that governance and observability are harness obligations rather than model properties is that a statute assigns them to the *system*.

**Article 12 - record-keeping.** High-risk AI systems must "technically allow for the automatic recording of events (logs) over the lifetime of the system"; the logging capabilities must enable recording of events relevant for identifying situations that may present a risk, and for "the identification of the natural persons involved in the verification of the results, as referred to in Article 14(5)" ✅ (Regulation (EU) 2024/1689, Article 12(1), 12(3)(d)). A model cannot log the identity of the person who verified its output. Only the harness knows that.

**Article 13 - transparency.** The instructions for use must describe the mechanisms that allow deployers to "properly collect, store and interpret the logs" ✅ (Article 13(3)(f)). The obligation is on the provider, but the artifact that satisfies it is a harness capability, documented and shipped.

**Article 14 - human oversight.** High-risk systems must be designed, including with appropriate human-machine interface tools, so that they "can be effectively overseen by natural persons during the period in which they are in use" ✅ (Article 14(1)). Effective oversight is not a prompt instruction to ask for confirmation; it requires that the overseer can see what the system is doing, understand it, and intervene. That is a lifecycle and interface obligation, and it is why section 4.5's approval points are a domain decision rather than a UX preference. Article 14(5) goes further in one case, requiring that no action be taken on the basis of an identification unless it has been separately verified and confirmed by at least two natural persons with the necessary competence, training and authority, with a narrow set of exceptions ✅ (Article 14(5)).

**Article 15 - accuracy, robustness and cybersecurity.** Systems must achieve an appropriate level of accuracy and robustness and "perform consistently in those respects throughout their lifecycle", with the Commission encouraged to support the development of benchmarks and measurement methodologies ✅ (Article 15(1), 15(2)). Consistency through the lifecycle is a claim about the harness's regression discipline, not about the weights.

**Dates and phasing**, stated so the citation is not mistaken for a claim about the present: the Regulation entered into force on 1 August 2024, and on the amended text currently published the high-risk obligations of Articles 12 to 15 apply from 2 December 2027 for Annex III systems and 2 August 2028 for Annex I systems, per Article 113(c) ✅ (Article 113(c) as amended, read 2026-10-28 ⚠ on whether the amendment is final; the original text set 2 August 2026 and 2 August 2027). Nothing in this section asserts that any particular system is high-risk, or that any institution is in scope.

Read the four articles together and the structural point is unavoidable: the obligations are phrased as properties of a *system that logs, discloses, is overseen and stays consistent*. Those are four harness functions. A team that reads them as model-quality requirements will satisfy none of them and believe it has.

### 6.5 A financial-domain regime, named for its shape

The EU AI Act is horizontal. A domain harness also faces sectoral regimes, and one is worth naming for the shape of its obligations rather than their content.

The PRA's Supervisory Statement 1/23 on model risk management, published 17 May 2023 and effective from 17 May 2024 (current version 23 April 2026), sets out five principles: model identification and model risk classification; governance; model development, implementation and use; independent model validation; and model risk mitigants ✅ (Bank of England / PRA, SS1/23, May 2023). Two of its features are directly load-bearing for a harness argument. First, the expectations apply to all models used to inform business decisions "regardless of technology and including models used for financial reporting purposes", and the statement's sub-principles explicitly include identifying and managing the risks associated with the use of AI techniques such as machine learning ✅ (SS1/23, as published). Second, the existence of an *independent validation* principle makes independence a control, not a culture.

That second point is the harness argument. Independent validation requires that a validator, who did not build the model, can run it, inspect it and challenge it. For an agent, "run it" means a pinned environment, a versioned artifact, a scoring path and a golden set the validator can execute without the build team's cooperation. A harness whose eval can only be run by the team that built the agent fails the principle structurally - not because the team is untrustworthy, but because the artifact does not permit the test. Section 5.5's "independently runnable" property is the same requirement stated as engineering.

**The limit, stated plainly.** Naming a regime is not asserting its applicability. This guide does not know any institution's permissions, booking model, or supervisory relationship, and it gives no legal or regulatory advice. The claim made here is narrow and structural: regimes of this shape impose obligations that land on harness capabilities - identification, validation, evidence, oversight - and a harness designed without those capabilities cannot satisfy them by adding documentation later.

### 6.6 What an auditor asks a guardrail it cannot inspect

Six questions. A generic policy file typically answers one.

| Question | What a content-filter guardrail can say | What a compliance control must be able to say |
|---|---|---|
| **What rule?** | "A policy was applied" | The rule text, by citation and version |
| **Which version, from when?** | Usually nothing | The rule version and its effective date, mapped to the control version |
| **Who approved it?** | The team that deployed it | A named role, with a date and a decision record |
| **What did it block?** | A count, or nothing retained | The blocked case, the matched condition, the evidence state |
| **What did it let through?** | Nothing - passes are not recorded | The population of permitted outputs, sampled, with the basis |
| **Where is the record?** | Logs, if enabled | A retention-defined evidence store, retrievable by case |

The fifth question is the one that exposes the design. A filter is built to act on the small number of bad cases; a control must also be able to describe the good ones, because "we blocked what we should have" is unprovable without a denominator. That requires the harness to record passes as well as blocks - which is an observability decision (section 4.6) taken for a governance reason, and a good illustration of the coupling problem the harness guide names at section 9.3.

### 6.7 The honest limits

**A control is not a legal conclusion.** A harness can enforce "no recommendation without a suitability record". Whether that rule, so encoded, satisfies a given regime in a given jurisdiction is a legal question the harness cannot answer and this guide does not attempt.

**Rule drift is the failure mode with no natural detector.** The rule text is amended; the control is not; the control continues to enforce last year's rule, correctly, with a green dashboard. Nothing in the harness's own telemetry shows this, because the harness is doing exactly what it was told. The only defence is the version mapping in section 6.2 - a control that cannot state which rule version it transcribes cannot be checked for drift, and a control that can be checked is one an audit can fail on schedule rather than in an incident.

**A policy layer with no owner decays into folklore.** The most common real-world state is a set of rules in a prompt, maintained by whoever last touched it, with no version, no test and no approver. That is anti-pattern 7 in section 12, and its tell is that nobody can say when a specific rule was added or why.

### 6.8 Why this layer is the one most often missed

Three reasons, each with a different remedy.

**The vocabulary is borrowed from the wrong domain.** Teams reach for the guardrail products they know, and those products speak in taxonomies of harm. The domain's rules do not fit the vocabulary, so they get expressed as prompt instructions instead - which is a real expression of intent and not a control.

**The demo passes.** A content filter demonstrably blocks a nasty output, which is visible and satisfying. A compliance control's value shows up in an audit that did not happen, so it never competes for the demo slot.

**Ownership is split across a boundary no one owns.** The rule belongs to compliance, the hook belongs to engineering, and the mapping between them belongs to nobody. The mapping is the artifact that matters, and section 8's build tests are written to force someone to own it.

## 7. Context and Tools, Honestly

### 7.1 The separation test

Two claims are routinely made together and should be pulled apart: "the harness is domain-specific because the corpus is domain-specific" and "the harness is domain-specific because the tools are domain-specific". The first is usually true and consequential. The second is usually true and inconsequential.

The test is a deletion test. Remove the thing and re-run the suite.

- Remove the domain **tools**, substitute an equivalently shaped generic set (a lookup that returns a record, a fetch that returns a document, a submit that writes a row). If the suite still passes, the tools were configuration. It almost always still passes.
- Remove the corpus's **authority structure** - the precedence order, the effective dates, the supersession links, the exclusions - and keep the same documents. If the suite now passes answers that cite withdrawn guidance as current, the context layer was carrying the domain. It almost always does.

One of these deletions changes what the harness can be correct about. The other changes what it can be correct *with*.

### 7.2 What is genuinely bespoke in context

| Property | What it is | Why a generic retriever cannot hold it | Harness mechanism it forces |
|---|---|---|---|
| **Authority** | A precedence order over sources, independent of similarity | Ranking is a single scalar; precedence is a partial order over source classes | Precedence resolution after retrieval, before assembly |
| **Currency** | Effective-from and effective-to per document, per clause where clauses are amended separately | A vector has no date | Temporal filter at query time, keyed to the transaction date rather than to today |
| **Provenance** | The obligation to carry the operative clause into the answer | Retrieval returns passages; it does not bind a claim to its basis | Citation binding in the output path, and a check that every material claim has one |
| **Negative knowledge** | Documents and statements that must not be surfaced or relied on | A ranking can lower a document; it cannot forbid one | An exclusion set with a hard filter, and an audit of what was suppressed |
| **Decay** | The corpus ages out; some of it becomes wrong rather than merely old | The index does not know what it does not know | A currency review cadence, and a harness check that flags answers built on sources past their review date |

The first four are the argument of section 4.4; the fifth is [building_vertical_ai_agents_guide.md](building_vertical_ai_agents_guide.md) section 3.5 arriving in the harness as a maintenance obligation. Note that all five are *schema* properties. That is what makes them tractable: a domain context layer is mostly a metadata discipline, not a new retrieval algorithm. The algorithm is fine. It is optimising the wrong function.

### 7.3 Why "domain-specific tools" is usually a configuration claim

A domain tool is a function with domain nouns. The tool layer registers it, describes it, exposes it, checks the argument schema, applies the auth scope, and returns the result - all identically to a general tool. Nothing about the layer's behaviour is domain-shaped, and that is a good thing: it means a domain can add forty tools without touching the harness's tool semantics.

The inflation is worth naming because it is the most common way a general harness gets described as a domain harness in a procurement document. Three tells:

1. **The tool count is presented as evidence of domain depth.** It is evidence of integration work. A team that has integrated forty systems has done forty integrations; it has not necessarily built a single domain control.
2. **The tool descriptions carry the domain rules.** If the rule "do not quote a rate without a timestamp" lives in a tool's description string, it is advice to a model, not a constraint on the harness - unevidenced, unversioned, and invisible to the policy layer.
3. **The tools are the demo.** A tool that returns a domain record is visually compelling in a way that a precedence resolver is not, so the demo selects for the tools.

What the tool layer *does* carry that matters is the **authority annotation**: this call reads, that call acts; this call may be made by an agent, that one requires a human; this one creates a record a third party will rely on. Those annotations are how the governance layer learns what it is governing. They are not tooling semantics - they are governance facts attached to tools - and section 4.3 reallocates them for exactly that reason. A harness where the annotation exists is a harness where the governance layer can be written; a harness where the adapter branches on the tool name is one where it cannot.

### 7.4 What the two owners already say

[agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) section 3.4 and section 3.5 own the tool layer's depth - protocols, description and discovery, selection, training, scalability - and section 4.1 and section 4.2 own the context layer's - short-term window, session state, long-term memory, long-horizon techniques, drift. None of that is re-derived here. What this section adds is the separation: the tool layer's depth is a depth of *mechanism*, and a domain adds nothing to it; the context layer's depth is a depth of *selection*, and a domain replaces its objective function. That is why the same guide can be excellent on both layers and still have nothing to say about a domain - mechanism depth and domain meaning are orthogonal axes, and the harness guide occupies the first.

## 8. The Build Decision

### 8.1 The three options

When a domain arrives, three things can be done to the harness, and they are not a continuum - they have different cost structures and different failure modes.

| Option | What it means | What it buys | What it costs |
|---|---|---|---|
| **Configure** | Same harness, domain tools, corpus, policies and scoring plugged into existing hook points | Speed; upstream improvements keep arriving; a small surface to maintain | Only the layers expressible through existing hooks can change; the rest stay generic |
| **Fork** | Copy the harness, modify the layers that cannot be configured | Freedom in the layers that matter most (usually C, V, O) | You inherit an upstream you no longer control; every upstream fix becomes a merge; divergence is invisible until it is expensive |
| **Build** | A harness designed for the domain from the start | Coherence; the layers agree with each other | Cost, and the loss of a general test suite as a sanity check |

The decision is not a matter of taste and it should not be made by seniority. It is decidable with six tests, each of which an engineer can answer in a meeting with the domain's owner in the room.

### 8.2 Six tests

| # | Test | The question | If yes | If no |
|---|---|---|---|---|
| 1 | **Oracle expressibility** | Does the domain's definition of correct require a check the general harness cannot express - a state comparison against a system of record, a rule engine, a two-person sign-off? | Fork or build. Configuration cannot add an oracle. | Continue |
| 2 | **Rule expressibility at hooks** | Can every domain rule be written as a predicate over an existing hook point (pre-tool, post-generation, pre-commit), with an evidence record? | Configure | Fork |
| 3 | **Evidence sufficiency** | After configuration alone, can the harness's record answer the auditor's six questions in section 6.6? | Continue | Fork - and O is almost always the reason |
| 4 | **Corpus semantics** | Does the domain require precedence, currency, or negative knowledge that the retrieval layer cannot represent? | Fork or build - C is the reason | Continue |
| 5 | **Adjudication ownership** | Is there a named authority who will sign the golden set, and can that authority's delegate run it without the build team? | Continue | Stop. None of the three options is ready; the eval is the blocker, and building the harness first is how the eval ends up fitting the harness |
| 6 | **Rule-change rate** | Do the domain's rules change faster than a fork could be rebased? | Configure, or build with rules as data rather than code | Fork is viable |

The tests are ordered so that the cheap ones come first, and so that test 5 can stop the project. It is worth being blunt about test 5: a team that cannot name who signs the ground truth will build a harness, score it on a set they wrote themselves, and discover the problem at acceptance. The build decision is downstream of the eval decision, not upstream of it - which is [building_vertical_ai_agents_guide.md](building_vertical_ai_agents_guide.md) section 7.4's point, arriving in the harness.

**What the tests are not.** They are not a scoring rubric; a team that fails test 2 and passes the rest forks, and the fork is narrow. They are not permanent; test 6 means a fork can become wrong later, and section 9 is about what to do then. And they do not replace judgement about the domain's stability - test 6 is where that judgement enters, as an engineering estimate of rebase cost rather than a feeling about the market.

### 8.3 The cost of the third option

A fork inherits an upstream you no longer control, and the inheritance is asymmetric: you receive every upstream change as a merge conflict and none of the upstream's testing. [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) section 9.3 names the coupling problem - a change in one layer changes the behaviour measured by another - and a fork makes the coupling harder to see, because the fork's tests were written against the fork's behaviour, so a regression looks like a passing build.

Three costs that are usually omitted from the fork's business case:

- **The security inheritance.** Upstream fixes a sandbox escape, a tool-permission bypass, a prompt-injection vector. The fork does not get the fix; it gets a diff against code it has modified. This is the cost that makes forks of *execution* and *tooling* layers expensive and forks of *verification* and *context* layers comparatively cheap - which is a useful rule: fork the layers where the mechanism is stable, configure the layers where the mechanism is under active security pressure.
- **The protocol inheritance.** If the tool layer tracks an external protocol, a fork that pins an old protocol version accumulates incompatibility silently until an integration partner moves.
- **The evidence inheritance.** A fork's records diverge from upstream's schema, so a shared audit tooling that reads upstream traces stops reading the fork's. In a regulated setting this is the expensive one, because it is discovered by the auditor.

### 8.4 The decision record

The output of this section is one page, and it should be written before the work starts: which option, which tests drove it, which layers are expected to change meaning, which are expected to stay contents, and what would have to become true for the decision to be revisited. The last item is the one that keeps the decision honest, because it names the trigger - a rule-change rate increase, an oracle becoming available, an upstream protocol revision - rather than leaving the fork to be defended by sunk cost.

### 8.5 The neighbouring decisions

[building_vertical_ai_agents_guide.md](building_vertical_ai_agents_guide.md) section 7.1 asks a different buy-versus-build question: whether to buy or build the *vertical* - the product, the corpus, the workflow position. This section asks whether to configure, fork or build the *harness*. The two decisions are independent and are routinely conflated in procurement: buying a vertical product tells you nothing about whether its harness can be configured to your rules, and building your own vertical does not tell you whether to fork a harness or write one.

[agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) section 11.1 supplies the reference profiles - six implementations with their ETCLOVG layer sets - which is the practical input to the configure-or-fork question: a harness whose profile already covers the layers your domain changes meaning in (typically C, V, O and G) is a candidate for configuration; one that covers only E, T and L is not, however good it is at those three.

## 9. The Portability Question

### 9.1 What does not transfer

A domain harness is an asset while the domain holds still and a liability when it does not. The question is not "is it portable" - it is "which parts are bound to the domain, and what does un-binding cost".

| Artifact | Bound to the domain? | Why | Porting cost |
|---|---|---|---|
| Execution substrate, egress policy | Weakly | Constraints are stated as data; a new domain states new constraints | Re-wire |
| Tool registry, schemas | Weakly | Nouns change; the mechanism does not | Re-wire |
| Context index metadata | **Strongly** | Authority tiers, effective dates and exclusions encode one domain's legal structure | Re-derive if the authority structure differs; re-wire if it is only a new document set |
| Precedence resolver | **Strongly** | The order itself is the domain's | Re-derive |
| Golden set | **Strongly, and irrecoverably** | Every item was adjudicated under one rule set by one authority | Re-derive. Nothing of it survives a rule change except the method |
| Adjudication authority | **Strongly** | The people with standing in domain A have none in domain B | Re-derive, and it is usually the long pole |
| Refusal categories | **Strongly** | The assistance-to-advice boundary is jurisdiction- and profession-specific | Re-derive |
| Evidence schema | **Strongly** | Shaped by a regulator's questions | Re-wire the schema, reuse the transport |
| Lifecycle machinery, approval hooks | Weakly | The mechanism is domain-neutral; the approval set is not | Re-wire |
| Loop, retries, checkpointing | No | Genuinely generic | Reuse |

The pattern is clean enough to be a rule: **the layers that changed meaning are the layers that do not port.** Execution, tooling and the loop re-wire in days; context, verification and governance re-derive over months, and the re-derivation is mostly not engineering work - it is expert time, which is the scarce input the vertical guide's section 4.2 identifies.

### 9.2 The asymmetry

A general harness degrades gracefully when the domain shifts. It becomes vaguer, asks more, hedges, and produces answers that are visibly unhelpful. The degradation is *legible*: the user notices, the failure looks like a failure, and the system's incompetence is on the surface where a human can compensate for it.

A domain harness fails confidently. It applies last year's rule correctly to this year's facts. It cites the superseded clause with a precise citation. It produces a record that satisfies the audit schema, with the wrong rule version in the version field, and every check passes because every check was written against the old rule.

The asymmetry is not symmetric in cost, either. A general harness that is unhelpful gets escalated to a human, and the human does the work. A domain harness that is confidently wrong gets *accepted*, because the output looks like the work - correctly formatted, properly cited, evidenced. The failure is discovered downstream, by whoever relied on it, which is the most expensive place and the latest moment.

**The qualification this forces on the thesis** (section 4.11): "quietly stop applying" describes the failure better than "fail", but it is not the same as "fail silently". The domain harness's failure is loud in the record and quiet in the signal. It leaves a trail. The trail points at the wrong rule.

### 9.3 What triggers a re-derivation

Four events, and a harness that cannot detect them cannot claim to be maintained:

- **A rule change.** A statute amended, a supervisory statement revised, a professional standard reissued. The trigger is external and dated; the harness must hold the mapping from rule version to control version and to golden-set items, or it cannot know what the change invalidated.
- **A professional standard revision.** Subtler, because professional bodies revise practice without a commencement date. The trigger is the body's publication, and the owner is the professional, not the platform team.
- **Corpus decay.** [building_vertical_ai_agents_guide.md](building_vertical_ai_agents_guide.md) section 3.5 owns the argument: the corpus ages, sources are withdrawn, rates and thresholds go stale. In the harness this arrives as a currency check on the sources behind an answer, and as a review date per source class.
- **A model change.** [building_vertical_ai_agents_guide.md](building_vertical_ai_agents_guide.md) section 11.3 names the vertical that breaks when the model is swapped. In harness terms the change is a re-run of the golden set with a *fixed* harness, which is precisely what a domain harness is for - and precisely what fails when the golden set was never independent of the harness.

### 9.4 The porting plan

For a team taking a domain harness into a second domain, the plan is three columns and a signature:

| Class | Layers | Work | Owner |
|---|---|---|---|
| **Reuse** | Loop, retries, checkpointing, sandbox mechanism, tool transport | None, beyond regression | Platform |
| **Re-wire** | Egress policy, tool schemas, evidence schema, approval set | Days, by an engineer who knows the target domain's vocabulary | Platform plus domain SME |
| **Re-derive** | Precedence order, currency rules, refusal categories, rubric, golden set, adjudication authority | Months, by the domain's experts, with the platform team in support | Domain authority |

The failure mode in porting is treating the third column as the second. A team that "adapts" the golden set by relabelling items, or "adapts" the precedence order by editing the YAML, is producing an artifact that looks like a domain harness and has none of the properties - and it will pass its own suite, for the reason in section 5.6.

### 9.5 The honest counter-argument

There is a real case for the general harness in a domain, and it should be stated rather than dismissed: a general harness that is *known* to be generic invites the human to check, and a domain harness that is *believed* to be authoritative invites the human not to. If the choice is between a well-built general harness with a visible competence boundary and a badly built domain harness with a confident one, the general harness is safer. The domain harness earns its cost only when the layers that changed meaning were actually built - which is the difference between the configure column of section 8.1 and a fork nobody rebases.

## 10. The Failure Modes

Seven, with the symptom an operator would actually observe, the cause in harness terms, and the guardrail that prevents it. The first six are the ones this guide's argument predicts; the seventh is the one that shows up in almost every real deployment.

| # | Symptom | Cause | Guardrail |
|---|---|---|---|
| **a** | The suite is green and the domain's experts reject the output on first review | The harness's definition of correct was inherited from a general benchmark; no domain ground truth was ever loaded | Test 5 of section 8.2 before the build: name the authority who signs the set, or stop |
| **b** | The guardrail blocks a harmless output and passes a non-compliant one | The control is a content classifier over a vendor taxonomy, not a rule with a subject, condition, evidence and owner | Section 6.2's six-element control; the six auditor questions in 6.6 as an acceptance test |
| **c** | Answers are fluent, well-cited, and cite guidance that was withdrawn | Retrieval optimises relevance; the corpus has no authority or currency metadata; the superseded document is the most similar one | Index metadata plus a precedence resolver plus an exclusion set (section 7.2); a currency check on every cited source |
| **d** | Everything works in the demo and nothing works on the second system of record | The tool set was built for the demo's happy path: one system, read-only, clean data, no pagination, no partial failure | Tool contracts written against the domain's real failure modes; a tool test suite with degraded backends |
| **e** | The eval improves every sprint and production does not | The ground truth was produced by the same model, or by the same team, or by a judge model with the same priors - the set grades its own homework | Adjudication by a named external authority; disagreement recorded; the held-out set never seen by the builders |
| **f** | The harness is described as portable, and behaves identically in a domain it was not built for | The domain content lives in prompts and tool descriptions; no layer changed meaning, so nothing signals the domain is different | The deletion test of section 7.1, run in reverse: add a domain and see whether any layer's contract changes |
| **g** | A rule exists, is enforced, and cannot be produced in an audit | The rule was written into a prompt or a tool adapter rather than a policy hook with a version and an owner | Rules as data in the governance layer; the version mapping of section 6.2; a control that cannot state its rule version fails acceptance |

**On (a), the most expensive.** The symptom is not a bad score; it is a good score followed by an expert's disbelief. The harness did exactly what it was asked and the asking was wrong. This is the failure the thesis is about, and the only reliable guardrail is organisational: someone with standing signs the definition of correct before the harness is built, and that signature is an artifact rather than a meeting.

**On (f), the most deceptive.** A harness whose domain content is entirely in prompts and tool descriptions behaves *identically* in a new domain, because it has no domain semantics to get wrong. That invariance reads as robustness in a demo and is the absence of a domain harness in production. The distinguishing question is not "does it work in domain B" but "what in it would have to change if domain B's rules differed" - and if the answer is "the prompt", the harness is general.

## 11. The Cymbal Bank Worked Example

*Conventions: Cymbal Bank is the fictional institution used across this repository's worked examples. Everything below is illustrative engineering fiction written to exercise the layer analysis - not a description of any real system, and not a claim about any institution's obligations. No other institution is named in this section.*

### 11.1 The scenario

Cymbal Bank's wealth division runs an agent that supports relationship managers: it drafts client correspondence, retrieves product terms, and answers questions about a client's existing holdings. It runs on a third-party general harness, configured with the bank's tools and corpus. The compliance function has one rule it cares about more than the rest, and it is not a rule about content.

**The rule.** Any communication to an external client that recommends a product must (i) be grounded in a suitability assessment recorded for that client, (ii) carry the disclosure text version currently in force for that product class and jurisdiction, and (iii) be released by a relationship manager with the authority to make the recommendation. If any of the three is absent, the communication is withheld and routed to the relationship manager with a reason.

### 11.2 The layer whose meaning changed: governance

The agent produces: *"Based on your objectives and risk appetite, the Cymbal Global Income Fund is suitable for you. Expected yield is in line with the fund's stated objective."* Accurate, well-written, and - absent a suitability record id and the current disclosure - not permitted.

**The generic guardrail's verdict: pass.** The configured content filter applies the vendor's taxonomy. The text contains no personal data, no harmful instruction, no toxicity, no injection attempt, and no prohibited topic. It is a financial-services sentence in a financial-services application, which is what the filter expects to see. Confidence: high. Record: a pass, not retained.

**The compliance control's verdict: withhold.** Subject: an output addressed to an external client. Condition: contains a recommendation predicate over a product for an identified client, AND no suitability record id is bound to the turn, AND the disclosure version does not match the version in force for that product class and jurisdiction. Evidence: the suitability record id, the disclosure version, the RM identity and authority level. Remedy: withhold, route to the RM, record the reason and the missing element. Owner: a named compliance role. Version: the rule text version and the disclosure text version, both mapped.

Same output. Same model. Same harness binary. Two opposite verdicts, and the difference is entirely in what the layer's contract was.

**Why it is a meaning change and not configuration.** The hook points existed. The harness could already intercept a pre-release output and call a policy. What it could not do was *carry a subject, an evidence set and a version through the interception* - because its policy interface took a string and returned a boolean. Getting from the generic guardrail to the control required four changes that are not configuration:

1. A **control registry** with one entry per rule, each naming its rule text, version and effective date.
2. An **evidence binder** that attaches the suitability record id and disclosure version to the turn, so the policy can evaluate a condition over facts rather than over text.
3. A **decision record** written on pass as well as on block, so that "we blocked what we should have" has a denominator (section 6.6, question five).
4. An **authority model** for the releasing human, so that "released by an RM with authority" is checkable rather than asserted.

That is the layer's contract changing from "apply a policy to a string" to "evaluate a rule over a subject with evidence and record the outcome". The mechanism changed because the meaning changed.

### 11.3 The layer whose meaning did not change: tooling

The same programme added fourteen domain tools: facility lookup, holdings by client, product terms by class and jurisdiction, disclosure text by version, suitability record retrieval, and nine more. Each arrived as a schema, a description, an auth scope and a return type.

**What the tool layer did.** It registered them. It discovered them at session start, described them to the model, enforced their argument schemas, applied their scopes, and returned their results. The tool-layer test suite changed in exactly one way: the fixtures. No assertion about discovery, selection, invocation or error handling changed, because nothing about the layer's contract had.

**What the tool layer did not do, and should not have.** It did not learn that `facility_lookup` is a read and `disclosure_publish` is an act; that one may be called freely and the other may not; that a call to the suitability record is itself an event that must be recorded. Those facts were attached to the tools as **authority annotations** and consumed by the governance layer, which is where they belong. Had they been implemented as branches inside the tool adapters, the harness would have had the rule, unevidenced and unversioned, and the audit would have found a wrapper instead of a control - failure mode (g) in section 10.

**The honest reading.** Cymbal Bank added a large amount of domain integration work in the tool layer, and it was configuration. The programme's own status reporting called this "domain-specific tooling", which was true and meant almost nothing. The thing that made the harness domain-specific was the governance layer, and it was four weeks of design work that produced no visible demo.

### 11.4 On the thesis

The example is deliberately small, because the argument does not need a large one. One rule, one output, two verdicts. The generic harness did not merely underperform on the domain's standard. It produced a *pass* - a green result, on a suite that had nothing to do with the domain's definition of correct, and it would have done so indefinitely, at scale, with a dashboard showing compliance.

The tool layer shows the other half: a layer can absorb a great deal of domain content and remain exactly what it was, and saying so is not a criticism of the layer. It is what makes the analysis useful. **The harness is where a domain's standards become executable - and the tool layer is where a domain's nouns become callable, which is a different and much cheaper thing.**

## 12. The Anti-Patterns

Nine, each stated as the failure it looks like from the outside.

**1. The domain prompt wearing a harness costume.** A long system prompt containing the domain's rules, its vocabulary and its prohibitions, on an otherwise untouched general harness. It looks like a domain agent because it talks like one. It is a general harness with a briefing document, and the rules it recites are unenforced, unversioned and unrecorded.

**2. The guardrail that enforces the vendor's taxonomy.** A content filter deployed as the compliance story, with a dashboard of blocked categories. It looks like governance because it blocks things. It is answering "is this text safe" when the domain asked "is this act permitted", and section 6.3 is the worked demonstration that the two verdicts can differ on the same sentence.

**3. The eval that grades its own homework.** A suite whose expected outputs were produced by the same model, labelled by the same team, or judged by a model with the same priors. It looks rigorous because it has a number and a trend line. It measures agreement between the harness and itself, and the trend line measures the agreement improving.

**4. The corpus that is big and wrong.** A very large retrieval set, generously indexed, with no authority order, no effective dates and no exclusions. It looks like a moat because it is large. It returns the withdrawn circular first, because the withdrawn circular was written to answer the question.

**5. The demo tool set.** Twelve tools wired to one clean system of record, read-only, no pagination, no partial failure, no rate limit. It looks like integration because the demo completes. The second system of record arrives and the tool layer's error handling - which was never designed, only inherited - becomes the project.

**6. The portable-looking harness.** A harness whose domain content lives entirely in prompts and tool descriptions, and which therefore behaves identically in a new domain. It looks robust because nothing breaks. Nothing breaks because nothing domain-specific was built; section 10's failure mode (f).

**7. The rule in the prompt.** A domain prohibition expressed as an instruction to the model rather than a control at a hook. It looks like a policy because it is written in imperative language and everyone has read it. It has no version, no test, no owner and no evidence, and the first audit question - "what rule, which version, who approved it" - has no answer.

**8. The fork with no rebase budget.** A harness forked for good reasons, then left to drift while upstream fixes security issues the fork never receives. It looks like ownership because the team controls the code. It is an unmaintained dependency with a compliance narrative attached.

**9. The refusal scored as an error.** A pipeline where declining is recorded as a failed run, so the dashboard shows the domain's correct behaviour as a gap. It looks like a quality metric. It is a metric pushing the system toward the behaviour the domain forbids, and it will win, because metrics do.

## 13. The State of Practice

### 13.1 What is established

Five things are settled enough to build on.

**The harness, not only the model, decides a large part of agent behaviour.** [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) section 1.4 carries the evidence base; this guide takes it as given and does not restate the numbers. The practical consequence is accepted practice now: teams iterate on tools, context policy and verification before they iterate on weights.

**Agent-computer interface design is engineering, not prompting.** Anthropic's account of building its SWE-bench agent - that it spent more time optimizing tools than the overall prompt, and that "incorrect assumptions about what's under the hood are a common source of customer error" - is a vendor's own report and is dated December 2024 ✅ (Anthropic, "Building effective agents", December 2024, vendor engineering blog). It has held up as practice.

**Where an executable oracle exists, test-based verification is the strongest available eval.** SWE-bench's mechanism - edit the repository, run its tests - is the reference implementation, and it is honest because the oracle is the domain's own ✅ (arXiv 2310.06770, October 2023).

**Domain evals built by domain experts are a different kind of artifact from general benchmarks, and the difference is authorship.** LegalBench's construction process and FinanceBench's evidence strings are the two clearest published statements of the mechanism ✅ (arXiv 2308.11462, August 2023; arXiv 2311.11944, November 2023).

**Regulation assigns logging, transparency and oversight to the system, not the model.** Articles 12 to 15 of the EU AI Act are explicit about this and are phased rather than immediate ✅ (Regulation (EU) 2024/1689, Article 12-15, 113(c)). A harness is the only artifact that can discharge them.

### 13.2 What is contested

**Configure versus fork.** There is no published comparison, and this guide found none (section 15.1). The argument in section 8 is engineering judgement expressed as tests, not an empirical result, and a team that reaches a different conclusion on the same tests is not being unreasonable.

**Whether a model judge can stand in for expert adjudication.** The practice is widespread and the objection is structural rather than empirical: a judge model's preferences are drawn from the same distribution as the model being judged, so the agreement it measures is partly self-agreement. The domain's answer is a named human authority; the economics push the other way; the disagreement is live.

**Whether refusal is a metric or a policy.** In some regimes declining is a scored behaviour; in others it is a control with a legal basis. The harness treats both, but which one dominates changes the design of the scoring schema, and practitioners disagree about which should lead.

**Whether a "domain-specific harness" is a distinct artifact or a configuration pattern.** This is the guide's own subject and it would be dishonest not to name it as contested. The position argued here - that four layers change meaning, two do not, and one sits between - is one reading. The opposing reading, that a harness is a generic engine and all domain content is data, is coherent and is held by serious people; section 4.8 is the argument against it, and section 4.11 records where the strong form of the thesis fails.

**Whether tau-bench-style state oracles generalise.** They generalise exactly as far as the domain has a state that can be compared. That is a real constraint, and it is the reason the domain-eval literature is thin outside transactional domains.

### 13.3 What is vendor positioning

Named as categories rather than vendors, because the claims are not comparable and the vendors are not the point.

- **Platform vendors claiming vertical agents.** The claim is usually about the workflow and the corpus, and it is usually silent on the provenance of the ground truth. The question that separates the two is section 8.2's test 5: who signs the golden set.
- **Guardrail vendors claiming compliance.** The claim is about coverage of a taxonomy of harm. Section 6.3 is the argument that coverage of a taxonomy is not coverage of a rule, and no vendor's taxonomy contains a subject, an evidence record or an owner.
- **Harness vendors claiming domain readiness from a tool count.** Section 7.3. Integration depth is real and it is not a domain control.
- **"Domain-tuned" model claims.** Tuning changes behaviour; it does not supply an authority. A tuned model with no adjudicated set is a model with a different prior, and section 5.1's three failures apply unchanged.

### 13.4 The honest state of the literature

The harness literature is young. The taxonomy this guide borrows is from a 2026 survey, the engineering writing is 2024 to 2026, and the domain-specific material is mostly conference talks, vendor documentation and benchmark papers that were built to measure models rather than to specify harnesses. There is no textbook, no standard for what a domain harness must contain, and - the finding that matters for section 15 - no published study comparing a domain harness against a general one on the same task. What exists is mechanism-level work: benchmarks that show how to ship a policy document, regulations that specify what a record must contain, and regimes that require independent validation. That is enough to build from and not enough to be confident about, and the difference should show in how the argument is held.

## 14. The Claims Audit

| # | Claim | Verdict | Source - kind - date |
|---|---|---|---|
| 1 | Repository has 679 `.md` files; `domain.specific` 65, `harness` 109, `scaffold` 73, `system prompt\|tool definition` 59, `eval suite\|evaluation harness` 33, `ETCLOVG` 5, `ETCLOVG\|harness layer` 7, `vertical ai\|vertical agent` 2 matching lines | Verified | Repository measurement, matching-line counts, 2026-10-28 |
| 2 | `agent_harness_engineering_guide.md` (777 lines) and `agent_scaffolding_guide.md` (934 lines) each contain zero matching lines for `domain.specific` | Verified | Repository measurement, re-run this pass, 2026-10-28 |
| 3 | The only same-line co-occurrence of "domain-specific" and "harness" is 3 files, of which 1 is substantive (the vertical guide) and 2 are incidental test-harness collisions | Verified | Repository measurement, 2026-10-28 |
| 4 | The three boundary quotations sit at lines 42, 106 and 426 of `building_vertical_ai_agents_guide.md`, in section 1.1, section 1.6 and section 7.4 | Verified | Repository measurement, line-exact check, 2026-10-28 |
| 5 | A METR task specifies an environment, an instruction string, and optionally an automatic scoring method | Verified | METR Task Standard, README, specification repository, archived 2026-10-07 |
| 6 | The METR Task Standard is no longer maintained; METR uses Inspect to define tasks | Verified | METR Task Standard, README notice, 2026-10-07 |
| 7 | Anthropic describes the Claude Agent SDK as "a powerful, general-purpose agent harness" and reports an initializer-plus-coding-agent design for long-running work | Verified (as vendor editorial) | Anthropic engineering blog, "Effective harnesses for long-running agents", November 2025 (post is undated; the month is flagged, see section 15.1) |
| 8 | Anthropic reports spending more time optimizing tools than the overall prompt on its SWE-bench agent, and that incorrect assumptions about what is "under the hood" are a common source of customer error | Verified (as vendor editorial) | Anthropic engineering blog, "Building effective agents", December 2024 |
| 9 | Anthropic leaves the transfer of its long-running-agent findings to other fields (its examples: scientific research, financial modelling) as future work | Verified (as vendor editorial) | Anthropic engineering blog, November 2025 |
| 10 | SWE-bench comprises 2,294 real GitHub-issue problems across 12 Python repositories, resolved by editing the codebase and verified by execution | Verified | arXiv 2310.06770, benchmark paper, submitted 2023-10-10, v3 2024-11-11, ICLR 2024 |
| 11 | tau-bench gives the agent domain-specific API tools and policy guidelines and scores by comparing final database state to an annotated goal state, with a reliability metric over trials | Verified | arXiv 2406.12045, benchmark paper, 2024-06-17 |
| 12 | LegalBench contains 162 tasks built through an interdisciplinary process with tasks designed and hand-crafted by legal professionals | Verified | arXiv 2308.11462, benchmark paper, 2023-08-20 |
| 13 | FinanceBench contains 10,231 questions with corresponding answers and evidence strings, intended as a minimum performance standard, with human review of answers | Verified | arXiv 2311.11944, benchmark paper, 2023-11-20 |
| 14 | Llama Guard is an LLM-based input-output safeguard built on a safety risk taxonomy, performing prompt and response classification, with adjustable taxonomy categories | Verified (as vendor editorial) | arXiv 2312.06674, vendor paper, 2023-12-07 |
| 15 | EU AI Act Article 12 requires automatic recording of events over the system's lifetime and logs enabling identification of risk situations and of the natural persons involved in verification under Article 14(5) | Verified | Regulation (EU) 2024/1689, Article 12(1), 12(3)(d), legislation, in force 2024-08-01 |
| 16 | Article 13 requires instructions for use describing mechanisms to collect, store and interpret logs; Article 14 requires design enabling effective oversight by natural persons, with two-person verification in the case described in 14(5); Article 15 requires accuracy, robustness and consistency through the lifecycle | Verified | Regulation (EU) 2024/1689, Articles 13(3)(f), 14(1), 14(5), 15(1)-(2), legislation |
| 17 | High-risk obligations apply from 2027-12-02 (Annex III) and 2028-08-02 (Annex I) on the amended text, where the regulation as originally published set 2026-08-02 and 2027-08-02 | Verified, with a flagged amendment | Regulation (EU) 2024/1689, Article 113(c) as amended, official Act explorer, read 2026-10-28 ⚠ on whether the amendment is final |
| 18 | PRA SS1/23 sets five model risk management principles - identification and classification, governance, development/implementation/use, independent validation, mitigants - applies regardless of technology including vendor models, and names AI/ML risk | Verified | Bank of England / PRA, Supervisory Statement 1/23, published 2023-05-17, effective 2024-05-17, current version 2026-04-23 |
| 19 | NIST AI RMF 1.0 organises activity into GOVERN, MAP, MEASURE and MANAGE, and describes itself as law- and regulation-agnostic, outcome-focused and non-prescriptive | Verified | NIST AI 100-1, AI RMF 1.0, standard, January 2023 |
| 20 | MCP standardises exposure of tools and context and states that it cannot enforce its security principles at the protocol level | Verified | Model Context Protocol specification 2025-06-18, protocol specification |
| 21 | ISO/IEC 42001:2023 distinguishes an AI management system from a product specification | Unverified - not asserted | Primary text not retrievable this pass; no secondary source used |
| 22 | SR 11-7 / OCC Bulletin 2011-12 is the model-risk regime a financial harness would answer to | Unverified - not asserted | Primary text returned 404 on three URLs; replaced by PRA SS1/23 (row 18) |
| 23 | A published study compares a domain-specific harness against a general harness on the same task | Rejected - negative finding | Search across the sources read this pass; none found |
| 24 | Any benchmark score, vendor improvement percentage, market size, adoption rate or leaderboard ranking | Rejected - not asserted | Deliberate exclusion; benchmarks cited for mechanism only |

**How to read the table.** Rows 1 to 4 are measurements of this repository, reproducible with `grep -c` and `sed -n`, and they are the empirical basis for the guide's existence rather than for its argument. Rows 5 to 20 are primary-source verifications: specifications, papers, statutes and supervisory statements read in this pass, with the vendor-authored items (7, 8, 9, 14) marked as vendor editorial because their claims are about the authors' own systems and are not independently reproducible. Rows 21 and 22 are claims the guide declined to make because the primary text could not be reached - row 22 is the honest record of a substitution, not a silent one. Row 23 is a negative finding from a bounded search and should be read as "not found here", not as "does not exist". Row 24 records the class of claim the guide refuses: the subject attracts performance figures, and a guide that printed one would be borrowing authority it has not earned.

No figure in this file is a performance claim. The only numbers that appear are counts, dates, article numbers, task counts and corpus sizes - all of them properties of a text or a dataset, none of them a comparison of systems.

## 15. What Could Not Be Verified, Glossary and Cross-References

### 15.1 What could not be verified

**The primary text of SR 11-7 / OCC Bulletin 2011-12.** Three URLs on the Federal Reserve's site returned 404 this pass, and a web search for the document returned an empty result set. Both are recorded as tool limitations, not as evidence that the document does not exist or has been withdrawn. The guide substituted PRA SS1/23 for the financial-sector example because that text was retrievable and citable; the substitution is declared in the claims audit, row 22, rather than made quietly.

**ISO/IEC 42001:2023.** The ISO catalogue page failed to scrape. The management-system-versus-product distinction is therefore not asserted anywhere in this guide, and the glossary does not use it.

**DORA (Regulation (EU) 2022/2554).** The EUR-Lex page failed to scrape, and no claim rests on it.

**Whether any published study compares a domain-specific harness against a general harness on the same task.** None was found across the sources read. This is the absence that matters most for the guide's confidence: the layer analysis in section 4 is an argument from mechanism, not a measured result, and section 13.4 says so. A negative finding from a bounded search is not proof of absence.

**The internal rule sets of any vendor's guardrail product.** Not inspectable. The characterisation in section 6.3 is of the *shape* - a taxonomy over content - taken from a published paper by the vendor of one such system, and it is marked as vendor editorial wherever it is load-bearing.

**The exact publication date of Anthropic's long-running-agent post.** The extracted page carries no date. November 2025 is used throughout, consistent with the post's own references to a model released that month, and should be treated as ⚠ flagged rather than verified to the day.

**Whether a specific rule applies to a specific institution.** Out of scope by design. Section 6.4 and 6.5 name regimes to establish that logging, oversight and independent validation are obligations that land on harness capabilities; neither section asserts applicability, and neither is legal advice.

### 15.2 Glossary

**Harness.** Two senses, both used in this guide and distinguished wherever it matters. The *agent harness* is the infrastructure surrounding the model that turns model calls into bounded, stateful, tool-mediated execution - the seven ETCLOVG layers. The *eval harness* (task harness) is the rig that presents a task, runs an agent in a controlled environment and scores the attempt. The guide means the first by default.

**Scaffold.** The project skeleton an agent is built on - layout, config, tool registration, prompt module, memory module, entry point, tests. The scaffold is what you start from; the harness is what runs. Owned in depth by [agent_scaffolding_guide.md](agent_scaffolding_guide.md) section 2.

**Domain.** A body of practice with its own definition of correct, its own authority structure, its own currency rules and its own liability if the answer is wrong. Not a topic and not a vocabulary.

**Vertical.** The commercial axis: an agent sold into one industry, with that industry's buyer, workflow and liability frame. Independent of "domain" - a vertical may be sold with a thin domain claim, and a domain agent may be built with no market at all.

**Layer.** One of the seven ETCLOVG engineering surfaces - Execution, Tooling, Context, Lifecycle, Observability, Verification, Governance - each with its own contract rather than its own component. Owned by [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) section 2.1.

**Ground truth.** The record of what the correct answer is for a given input, in a form a scorer can use. In a domain it is adjudicated rather than given: it has an author with standing, a basis, and a version.

**Adjudication.** The act of deciding correctness by a person with the authority to decide it, recorded with its basis and its dissent. Distinct from labelling, which is a fact about a dataset; adjudication is an act whose provenance is part of its meaning.

**Guardrail.** A runtime control that inspects or constrains an action or output before it takes effect. In general use a content classifier over a safety taxonomy; in domain use a compliance control with a subject, a condition, an evidence record, an owner and a version. The gap between the two is section 6.

**Golden set.** The held-out, versioned, signed collection of adjudicated cases against which a domain harness is scored, carrying per-item provenance, the rule version it was adjudicated under, and the recorded disagreement. Section 5.5.

**Eval.** A scored suite: inputs, expected outcomes, and a procedure that produces a verdict. In a domain, the expected outcomes are the artifact and the scoring code is the easy part.

### 15.3 Cross-references

| Owner guide | Sections this guide cites | What is taken, and what is not re-derived |
|---|---|---|
| [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) | sections 2 and 2.1 (ETCLOVG at a glance; the seven-layer table); section 3 (E and T); section 4 (C); section 5 (L); section 6 (O); section 7 (V); section 8 (G); sections 9.1-9.3 (trilemma, capability-control, coupling); section 11.1 (six profiles); section 11.3 (what that guide adds and the gaps it flags); section 12 (Cymbal harness assessment) | The layer definitions and the architecture. This guide takes each layer's contract as given and asks only what a domain does to it. It does not restate the taxonomy, the trilemma or the ecosystem. |
| [agent_scaffolding_guide.md](agent_scaffolding_guide.md) | section 1 (the concept); section 2 (the scaffold anatomy); section 3 (known scaffold projects); section 4 (design principles); section 5 (components in depth); section 6 (enterprise and banking scaffolding); section 7 (a worked research-agent scaffold); section 8 (trends); section 9 (glossary) | The assembly and the build path. This guide adds nothing about file layout, config structure or component wiring; when it says "the governance layer" it means a component the scaffold guide already knows how to place. |
| [building_vertical_ai_agents_guide.md](building_vertical_ai_agents_guide.md) | section 3.5 (decaying corpora); section 4.2 and section 4.6 (expert scarcity; expert-in-the-loop); section 4.3 (inter-annotator disagreement); section 4.4 (asymmetric error); section 5.2 (the refusal test); section 7.1 (buy vs build the vertical); section 7.4 (the eval question); section 9 (liability and professional duty); section 11.3 (the model swap); section 13 (the Cymbal vertical example) | The vertical's business case and the ground-truth argument. This guide translates four of its findings into harness obligations (section 3) and does not re-argue them. |

## 16. The Closing Summary

**The verdicts, compactly.** Execution and tooling change contents, not meaning - the sandbox does not know what the code means and the tool layer does not know what the tool is for. Context changes meaning, because a domain corpus is an authority structure rather than a large retrieval set, and the superseded document is the most similar one. Lifecycle changes contents with a domain-supplied termination predicate, which is enough to make an agent advise when it should assist. Observability changes meaning at the schema, because a complete record is defined by the domain's evidentiary standard and not by the tracing library. Verification changes meaning, because correctness is adjudicated and the harness must carry the adjudication. Governance changes meaning, because the rules belong to a regulator, a profession or a contract, and a content filter is not a control.

**The build test, compactly.** Six questions, three of which decide most cases. Does the domain's definition of correct require an oracle the general harness cannot express - if yes, fork or build. Can every domain rule be written as a predicate over an existing hook with an evidence record - if no, fork. Can you name the person who signs the ground truth - if no, stop, because the eval is the blocker and a harness built before it will be shaped to fit it.

**The portability warning.** The layers that changed meaning are the layers that do not port. Execution, tooling and the loop re-wire in days; context, verification and governance re-derive over months, and the re-derivation is expert time rather than engineering time. The asymmetry is the risk: a general harness degrades into vagueness, which a human can see and correct, while a domain harness applies last year's rule correctly to this year's facts, cites it precisely, records it properly, and is believed. The failure leaves a trail, and the trail points at the wrong rule.

**The last word on the seam.** The vertical guide promised the harness guides owned this, and they did not. What was missing was not a chapter on domain tools or a paragraph on domain prompts. It was the observation that four of seven layers take their definition of correct from outside the harness - three of them wholly, one of them at the schema - that those four are exactly the layers which fail without telling you, and that the two layers everyone points at when they say "domain-specific" are the two where nothing changed.

**The harness is where a domain's standards become executable - or where they quietly stop applying.**
