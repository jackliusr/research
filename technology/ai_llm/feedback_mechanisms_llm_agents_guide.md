# Feedback Mechanisms in LLM Agents — the signal, not the loop

> **A feedback mechanism turns an agent's own behaviour into its next strategy — but a loop is only as good as the signal it closes on.** This guide is a critical reading of *A Survey on the Feedback Mechanism of LLM-based AI Agents* (Zhipeng Liu, Xuefeng Bai, Kehai Chen, Xinyang Chen, Xiucheng Li, Yang Xiang, Jin Liu, Hong-Dong Li, Yaowei Wang, Liqiang Nie, Min Zhang; IJCAI-25, Survey Track, pp. 10582–10592), one survey's organising scheme for a field that currently has no consensus scheme. It walks the survey's four-category taxonomy (internal, external, multi-agent, human feedback), its evaluation protocols and benchmark landscape, its own experimental comparison, and its six challenges — and then does the thing a survey cannot do for you: it puts the field's negative results next to the field's optimistic ones, checks a sample of the survey's summaries against the papers it cites, and separates a **quality mechanism** (a loop that makes output better) from a **control** (an independent check with accountability attached). The load-bearing claim is the one most badly served by the "add a reflection step" folk wisdom: in a loop where the model evaluates its own work, the model is grading itself, and that is a weak signal — not a control. The signal determines whether the loop is engineering or theatre.

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Cymbal Bank  \
> **Repo:** personal research library · **Series:** LLM/AI Engineering & Operations · **Domain:** AI Engineering · Agent Feedback  \
> **Date:** 21 September 2026

> **Companion guides (relative siblings in `technology/ai_llm/`).** This guide owns the **feedback-mechanism taxonomy and its evidentiary status** — the four categories, the representative systems and their learning formats, the evaluation protocols, the benchmark landscape, the survey's comparison, and the critical reading of what these loops can and cannot prove. It does **not** re-derive pipeline mechanics, agent architecture, or failure conditions; those live in: [multi_pass_llm_pipelines_guide.md](multi_pass_llm_pipelines_guide.md) (multi-pass and refinement **pipeline** patterns — its §4 Multi-Agent Patterns, §5 Reasoning Patterns, §11 Multi-Pass for Agentic Systems), [autonomous_agents_guide.md](autonomous_agents_guide.md) (agent **architecture** taxonomy §3 and evaluating agents §6), [agents_work_fall_apart_guide.md](agents_work_fall_apart_guide.md) (success conditions §1, failure conditions §4), [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) (production failure taxonomy §2, mitigations §8), [agentic_engineering_guide.md](agentic_engineering_guide.md) (lifecycle; §8 Phase IV Evaluation, §15 Open Problems), [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) (testing §4), [agent_scaffolding_guide.md](agent_scaffolding_guide.md), [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md), [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md), [llm_guard_models_guide.md](llm_guard_models_guide.md) (guard models **are** a feedback mechanism, and a stronger one than a prompt), [closed_loop_data_engineering_guide.md](closed_loop_data_engineering_guide.md), [ai_agent_drift_guide.md](ai_agent_drift_guide.md), [agentops_guide.md](agentops_guide.md), [mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md) and the rest of the MCP cluster for protocol standardisation, [agentic_search_vs_rag_guide.md](agentic_search_vs_rag_guide.md) and [ai_verify_toolkit_guide.md](ai_verify_toolkit_guide.md) for the retrieval and verification side, [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md) and [hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md) for orchestration, and [../physical_ai_guide.md](../physical_ai_guide.md) for embodied feedback.

> **How to read it.** If you have fifteen minutes: read §1's thesis and decoder, then §8 (the counter-evidence) and §14 (the anti-patterns) — those two sections are where the guide earns its keep. If you are designing a loop this quarter: read §4–§7 for the four categories, then §8, §9 and §13 (the worked example) in that order. If you are being asked to **govern** a loop: read §12 before §13, and treat §8 as the evidence base for every claim you make about "the system checks itself". If you are auditing a survey: read §2 and §15–§16 first, because this guide's central discipline is the distinction between *what the survey reports* and *what the primary source says*, and the audit section shows that discipline applied line by line. Claims are flagged inline: **⚠** marks a claim that is uncertain, imprecise in the survey, or unverified at the primary source; **❌** marks a claim that could not be verified at all. Every table that carries numbers says where the numbers came from and whether anyone independently ran them.

---

## Table of Contents

1. [Overview, Identity and Decoder](#1-overview-identity-and-decoder)
2. [The Anchor Paper and How to Read It](#2-the-anchor-paper-and-how-to-read-it)
3. [The Agent Framework and Where Feedback Sits](#3-the-agent-framework-and-where-feedback-sits)
4. [Taxonomy Category 1 — Internal Feedback](#4-taxonomy-category-1--internal-feedback)
5. [Taxonomy Category 2 — External Feedback](#5-taxonomy-category-2--external-feedback)
6. [Taxonomy Category 3 — Multi-Agent Feedback](#6-taxonomy-category-3--multi-agent-feedback)
7. [Taxonomy Category 4 — Human Feedback](#7-taxonomy-category-4--human-feedback)
8. [The Counter-Evidence — What Self-Correction's Own Literature Says](#8-the-counter-evidence--what-self-corrections-own-literature-says)
9. [The Survey's Comparison and What It Does Not Show](#9-the-surveys-comparison-and-what-it-does-not-show)
10. [Evaluation and Benchmarks](#10-evaluation-and-benchmarks)
11. [The Six Challenges and Future Directions](#11-the-six-challenges-and-future-directions)
12. [The Banking and Regulated Angle](#12-the-banking-and-regulated-angle)
13. [The Cymbal Bank Worked Example](#13-the-cymbal-bank-worked-example)
14. [The Anti-Patterns](#14-the-anti-patterns)
15. [The Claims Audit](#15-the-claims-audit)
16. [What Could Not Be Verified; Glossary; Cross-References; Closing](#16-what-could-not-be-verified-glossary-cross-references-closing)

---

## 1. Overview, Identity and Decoder

### 1.1 The thesis in one line and what it is doing here

**A feedback mechanism turns an agent's own behaviour into its next strategy — but a loop is only as good as the signal it closes on.**

Everything else in this guide follows from that sentence. Most of the practitioner literature on agent reflection stops at the first half: the loop exists, it iterates, the output got better on the demo. The second half is the part with the evidence — and the evidence says the *quality of the signal* is the variable that decides whether the loop helps, does nothing, or actively damages the answer. A compiler error is deterministic and verifiable. A unit test either passes or it does not. A web search returns text that may or may not be relevant. A world model is another learned model with its own error mode. And a model asked to grade its own reasoning is, in the most literal sense, marking its own homework. Those are not four flavours of the same thing, and the survey's taxonomy — which is a *source-of-signal* taxonomy — treats the first two and the last one as neighbouring sub-categories. §5 and §12 argue that this flattening is exactly the thing a practitioner cannot afford.

The thesis is also a governance claim, not just an engineering one. If a loop's signal is the model's own opinion, then the loop is a quality mechanism with no independence. Independence is what turns a check into a control, and independence is what an assurance framework asks for. A self-graded system described as "governed" is a category error with an audit trail, and §12 takes that apart.

### 1.2 What the anchor paper is

| Property | Value |
|---|---|
| Title | *A Survey on the Feedback Mechanism of LLM-based AI Agents* |
| Authors | Zhipeng Liu, Xuefeng Bai (corresponding), Kehai Chen, Xinyang Chen, Xiucheng Li, Yang Xiang, Jin Liu, Hong-Dong Li, Yaowei Wang, Liqiang Nie, Min Zhang |
| Venue | IJCAI-25 — Proceedings of the Thirty-Fourth International Joint Conference on Artificial Intelligence, **Survey Track** |
| Pages | 10582–10592 (11 pages) |
| Affiliations | Harbin Institute of Technology (Shenzhen), China; Peng Cheng Laboratory, China; Central South University, China |
| Abstract claim | Four-type taxonomy — internal, external, multi-agent, human feedback — plus a five-module agent framework, evaluation protocols, benchmarks, challenges and future directions |
| Companion artefact | A maintained paper list at `github.com/kevinson7515/Agents-Feedback-Mechanisms`. **Verified to exist** (checked 21 Sep 2026): public repo, owner `kevinson7515`, **a single `README.md` and no other files**, 11 commits, created 05/2025, last update 09/2025 announcing the IJCAI-2025 acceptance, 5 stars / 0 forks. It is a categorised bibliography — *Survey*, *Internal Feedback*, *External Feedback*, *Multi-Agent Feedback*, *Human Feedback*, *Benchmarks*, *Others* — of roughly 64 arXiv-linked entries, several of which carry no paper link. Two observations this guide can use: the list's own category placement puts **Reflexion under both internal and external feedback** and Self-Refine/ReAct under internal, echoing §4's duplication finding; and **the list contains no counter-evidence works** — no Huang et al., Kamoi et al., Stechly et al. or Valmeekam et al. — so it is a map of the optimistic literature, not of the evidence about it |
| Structure | §2 unified five-module agent framework · §3 four-category taxonomy · §4 evaluation and benchmarks · §5 experimental comparison · §6.1–§6.6 challenges and future directions · §7 conclusion |
| Tables | Table 1 (overview of existing feedback mechanisms) · Table 2 (common benchmarks) · Table 3 (comparison of different feedback mechanisms) |

The survey's own stated contributions are three: (1) a structured taxonomy of four categories; (2) described as the first comprehensive survey of feedback mechanisms for LLM-based AI agents, covering methodologies, evaluation protocols and benchmarks; (3) a discussion of limitations and future directions. Note what (1) and (2) are and are not. A taxonomy is a **classification act**, not a measurement. "First comprehensive survey" is a claim about the literature landscape at submission time, and it is the kind of claim that is true in the way that "first" claims usually are — true of the field as the authors bounded it, and not a statement that no one else has organised the space differently.

### 1.3 What kind of evidence a survey is — and what it is not

This matters more than usual here, because almost every sentence in the anchor paper that a practitioner wants to act on is a *reported* sentence rather than a *measured* one. Three distinct evidentiary objects live inside a survey paper, and they have different strengths:

1. **The survey's classifications.** "The survey classifies X as intra-task internal feedback." These are the survey's characterisations of other people's work. They are useful as a map and worthless as a warranty. Where this guide reports a classification, it says *the survey classifies* — and §15 records the cases where checking the primary source showed the classification to be arguable, imprecise, or internally inconsistent with the survey's own tables.
2. **The survey's own experiments.** §5 of the anchor paper is the one place where the survey authors put numbers in a table themselves. It is also, by their own parenthetical admission, partly an aggregation: "some experimental results are borrowed from ExpeL [Zhao et al., 2024] and AutoGuide [Fu et al., 2024]". §9 handles this in detail. The short version: Table 3 is *partly second-hand and partly illustrative of a literature*, not an independent benchmark run.
3. **The primary works the survey cites.** These are where the real evidence lives: a paper that reports a result on its own benchmark, with its own setup. This guide verifies a sample of ten-plus of these at the primary source (§15) precisely because a survey's summary of a paper is not the paper's position — and in at least one case in this literature the survey's summary omits the paper's central negative finding.

The survey is, in addition, **broadly optimistic about feedback loops**, which is a normal posture for a survey written in a rising field: the organising question is "how do we categorise and improve these mechanisms", not "do they work". That posture is legible in the paper's structure — the four-category section ends each sub-taxonomy with a *Discussion* paragraph that names a trade-off, but the trade-offs are about efficiency, memory cost and coordination, not about whether the signal is trustworthy. The one place the survey concedes a trust problem is §6.6, and §8 of this guide quotes it as the concession it is.

### 1.4 The vocabulary decoder

The field's terms are overloaded and the survey's usage is specific. This is the working vocabulary for the rest of the guide.

| Term | What it means here | The trap |
|---|---|---|
| **Feedback mechanism** | The component that converts evidence about the agent's own behaviour — a trace, an environment response, a critic, a human — into a change in what the agent does next. The anchor paper's unit of analysis. | "Mechanism" hides whether the change is in the *prompt/context* (this run) or in the *weights* (all future runs). That difference is the whole of §7's critique. |
| **Internal feedback** | Feedback generated by the agent itself — self-generated critique, self-consistency, verbal reflection — without an explicit external goal or tool. Survey's category 1. | "Internal" describes the *source*, not the *grounding*. A self-critique that happens to be grounded in a tool result the agent already fetched is still self-generated text. |
| **External feedback** | Signals defined outside the agent: environment scores, compilers, search engines, APIs, other learned models. Survey's category 2. | Spans a compiler (deterministic, verifiable) and a world model (learned, wrong-able). The survey's category does not separate them (§5). |
| **Multi-agent feedback** | Multiple agents interacting — collaboratively or adversarially — to refine a solution. Survey's category 3. | Same-model instances debating are not independent. Independence has to come from somewhere: different tools, different information, a verifier outside the debate. |
| **Human feedback** | Human input as the feedback source: instructional, corrective, preference. Survey's category 4. | Mixes a **training-time** procedure (preference shapes weights) with a **runtime** intervention (corrective steers this task). §7. |
| **Intra-task feedback** | Feedback used within a single task attempt, refining the current plan/answer. | Optimises fast; does not accumulate. The survey's own framing. |
| **Inter-task feedback** | Feedback carrying experience across tasks, requiring memory. | Accumulates, but demands substantial memory — and drags its old errors forward with its old lessons. |
| **In-context learning (ICL)** | Learning from examples/feedback inside the prompt, no weight change. | The dominant learning format in this literature. "Learning" without a gradient — reversible, cheap, and bounded by context. |
| **Supervised learning (SL) / reinforcement learning (RL) / imitation learning (IL)** | Weight-level or policy-level learning; the survey's other format values in Table 1. | In this field the label often describes the *original paper's* training regime, not a regime you can reproduce without their data. |
| **Outcome-based evaluation** | Measuring a feedback mechanism by end-to-end task success. The survey: "most evaluations are Outcome-based". | Aggregates away the loop. Success rate cannot tell you whether the loop helped or the base model was better. |
| **Process-based evaluation** | Evaluating intermediate steps as well as final outcomes. | Harder to ground truth, easier to game. |
| **LLM-as-a-judge** | Using an LLM to score outputs, usually with a rubric. The survey's substitute for continuous human feedback. | The primary paper behind the survey's citation documents positional, verbosity and self-enhancement biases plus limited reasoning ability — i.e. the judge is a weak signal (§8). |
| **Reflexion** | Verbal reinforcement learning: the agent writes a reflection into an episodic buffer after a failed attempt and re-attempts. | Not weight updates. The reflection text is the signal, and it is self-authored. |
| **Self-Refine** | Iterative refinement where one LLM acts as generator, feedback provider and refiner. | The canonical intrinsic loop, and the one the negative results are about. |
| **Reward model** | A learned scorer used as a training or search signal. | Another learned model. Its errors are systematic, not noise — and it is often the thing being gamed. |

### 1.5 The boundary — what this guide does not own

Declared by name so the series stays coherent. This guide owns **the feedback-mechanism taxonomy and its evidentiary status**: the four categories as this survey defines them, the representative systems and their learning formats, the evaluation protocols, the benchmark landscape as the survey maps it, the survey's comparison and its caveats, the limitations, and the critical reading of the evidence.

It does **not** own:

- **Multi-pass and refinement pipeline mechanics** — the engineering of passing output forward through stages, chunking, context hand-off, cost/latency budgeting for repeats. That is [multi_pass_llm_pipelines_guide.md](multi_pass_llm_pipelines_guide.md) (its §4 Multi-Agent Patterns, §5 Reasoning Patterns, §11 Multi-Pass for Agentic Systems). This guide names loops as *signals*; that guide names them as *pipelines*.
- **Agent architecture taxonomy and agent evaluation** — the autonomy spectrum, the architectural families, the evaluation methods. That is [autonomous_agents_guide.md](autonomous_agents_guide.md) (§3 architecture, §6 evaluating agents). This guide does not re-derive the architecture; §3 here is only a locator for where feedback *sits*.
- **Success and failure conditions** — when an agent works at all and when it falls apart, including the compounding-error arithmetic. That is [agents_work_fall_apart_guide.md](agents_work_fall_apart_guide.md) (§1 Success Conditions, §4 Failure Conditions) and [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md). This guide's §14 anti-patterns are loop-specific symptoms, not a general failure catalogue.
- **Testing, evaluation design and guard models** as engineering disciplines — [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md), [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md), [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md), [llm_guard_models_guide.md](llm_guard_models_guide.md). This guide borrows their vocabulary and points at them; it does not re-explain them.

---

## 2. The Anchor Paper and How to Read It

### 2.1 Scope, scheme and stated contributions

The anchor paper's scope is feedback mechanisms **for LLM-based AI agents**, explicitly distinguished from feedback for bare LLMs. The introduction's motivation is a specific gap: earlier feedback surveys (the paper names Pan et al. 2024 and, in the same breath, Madaan et al. 2024 — the Self-Refine paper) "focus on LLMs rather than the AI agents which operate within a broader ecosystem of interactions and decision-making processes." The survey's argument for why agent feedback is a distinct object is worth quoting in substance: the mechanism "involves not only the refinement of language generation but also the agent's ability to interpret, learn from, and adapt to feedback continuously." That is a reasonable scoping move, and it is the reason the survey's unit of analysis is a *mechanism in a loop* rather than a *prompting technique*.

The four-category scheme, as the survey states it:

| Category | Survey's definition | Survey's internal split |
|---|---|---|
| Internal feedback | "AI agents generate feedback internally to refine their strategies and actions" | intra-task / inter-task |
| External feedback | "AI agents leverage external models or tools to enhance feedback quality" | web knowledge / game API / code interpreter / world model |
| Multi-agent feedback | "multiple agents interact and provide feedback to one another" | collaborative / adversarial |
| Human feedback | "humans act as the primary source of feedback to guide agent behavior" | instructional / corrective / preference-based |

### 2.2 The reading rule

**The survey's categorisations are the survey's.** Every taxonomy assignment, every "learning" format, every benchmark size and every comparison figure in this guide is attributed to the survey until a primary source is named. When this guide reports a result *of* a cited work, it says which work and, in §15, whether the primary was checked. This is not pedantry about attribution; it is the difference between two sentences that read identically and mean opposite things:

- "Reflexion is an inter-task internal feedback mechanism." — the survey's classification.
- "Reflexion obtains significant improvements over a baseline agent across sequential decision-making, coding and language reasoning tasks, using linguistic feedback rather than weight updates." — the primary paper's own claim (verified at the arXiv record; §15).

The first sentence tells you where a work sits in someone's map. The second tells you what its authors assert they showed. A practitioner who treats the first as if it were the second will build the wrong thing, because the taxonomy does not carry the caveats — and the caveats are where the failures live.

### 2.3 One survey's scheme in a survey track — not a consensus

State this plainly: **this is one organising scheme, published in a survey track, and it is not the field's consensus.** Three reasons to hold it lightly:

1. **Survey tracks are synthesis venues.** A survey's acceptance criterion is comprehensiveness and clarity of organisation, not the resolution of contested questions. The four categories are a *proposed* partition, and the paper presents them as "a structured taxonomy that organizes existing research" — an organising act, offered. Nothing in the paper claims the field has converged on four categories.
2. **Other organisation schemes exist**, and they cut the same literature differently. The most directly relevant alternative is by *signal function* rather than *signal source*: verifiable vs non-verifiable signal, deterministic vs learned evaluator, internal vs external ground truth. Kamoi et al.'s critical survey of self-correction (TACL 2024, §8) organises the same underlying literature by **research question and feedback source validity**, and arrives at a very different headline. Pan et al. (TACL 2024) organise automated correction strategies by *correction mechanism type*. A guide that treated the four-category scheme as settled would be importing one author team's framing as field consensus.
3. **The scheme's own boundary cases are load-bearing.** §7 shows the human-feedback category collapsing two things that behave differently in production (weight-shaping preference data vs runtime corrective intervention), and §5 shows external feedback spanning a deterministic compiler and a learned world model. A consensus taxonomy would not put those in the same cell; a first-pass organising scheme reasonably might.

**What this means practically.** Use the four categories as a *vocabulary* — "which category is this loop in?" is a useful first question because it tells you what kind of machinery to expect. Do not use them as a *quality judgement*. Category membership tells you nothing about whether the signal is trustworthy, and §5 and §12 make the case that trustworthiness is the only axis that predicts whether the loop will do what you want.

### 2.4 What the survey does not do — the four gaps this guide fills

Read against its own ambitions, the survey leaves four things on the table. Each one is a section of this guide:

- **It does not carry the negative evidence.** The paper's framing is consistently pro-loop. The primary literature that contradicts it is well established by 2024 and is either uncited or cited for a different purpose. → §8.
- **It does not separate signal quality from signal source.** The taxonomy is about where feedback comes from. Practitioners need to know what it is worth. → §5, §12.
- **It does not audit its own table.** Table 3 borrows results, and the survey's own sentence says so; the numbers also saturate in a way the text does not remark on. → §9.
- **It does not connect to the operational layer** — what evidence a loop leaves, what it costs, what an auditor could reconstruct. → §12, §13, §14.

---

## 3. The Agent Framework and Where Feedback Sits

### 3.1 The five modules, as the survey states them

The survey's §2 proposes a unified framework "inspired by previous studies [Cheng et al., 2024; Durante et al., 2024]" with five modules and one structural claim.

| Module | Survey's stated role | Where it interacts with feedback |
|---|---|---|
| **Perception** | "analyze and understand environmental inputs, identifying key patterns to support task execution" | Produces the raw material a critic or verifier reasons over. Weak perception is an un-fixable input to a strong loop. |
| **Planning** | "break down complex tasks into simpler sub-tasks, generating plans and guiding actions" | Where internal feedback lands, and where AdaPlanner-style in-plan/out-of-plan refinement acts. |
| **Memory** | "stores past experiences"; long-term preserves stable knowledge, short-term handles real-time task information | The substrate for inter-task feedback. Without it, reflection has nowhere to persist. |
| **Action** | "translates decisions into operational outputs, interacting with the environment through tools, APIs, and embodied actions" | The source of the most trustworthy feedback in the entire field: the environment's reaction to a real action. |
| **Feedback** | "self-reflection and self-optimization"; "critically evaluate its prior decisions and actions, enabling dynamic adjustments" | The module under study. |

### 3.2 The one structural claim: feedback couples to memory and planning

The survey makes exactly one structural assertion about the framework, and it is the assertion that matters:

> "the feedback module is tightly coupled with the memory and planning modules, collectively optimizing the agent's decision-making and execution."

Read this as an engineering constraint, not an architecture diagram. Feedback that cannot write to memory cannot accumulate — it can only improve the current attempt. Feedback that cannot influence planning can only polish the current output. Those are two very different products, and they have very different failure modes: a non-accumulating loop is bounded and predictable (you pay per attempt, you gain per attempt); an accumulating loop is a system that changes its own future behaviour, which brings drift, poisoning, and the question of what happens when it learns the wrong lesson from a fluke ([ai_agent_drift_guide.md](ai_agent_drift_guide.md) owns that temporal failure mode). Any design conversation about a loop should start by asking which coupling is intended, because "we add reflection" is ambiguous between them.

### 3.3 The Voyager iron-smelting illustration — and why it is the right example

The survey illustrates the coupling with Voyager's iron-smelting task, and the illustration is worth understanding because it is a *complete* feedback trace:

```
Check furnace inventory → Discover no furnace → Retrieve skills
→ Refer to chest crafting skill to craft a furnace → Place the furnace
→ Check raw iron inventory → Smelt 5 raw iron
```

The survey's gloss: "When detecting the absence of a furnace in the inventory, the feedback module critiques this failure state, triggering a skill retrieval mechanism, thereby guiding the agent to learn furnace crafting."

Two things make this the survey's best example. First, the signal is **a state assertion about the world** (the inventory does not contain a furnace) — not a critique, not an opinion. It is checkable. Second, the loop closes on a **retrieved skill**, so the feedback writes to memory (skill library) and influences planning (the next plan uses the skill). Both couplings are live. This is the shape every loop in this guide is being compared against, and it is not a coincidence that it is a case where the signal is externally verifiable. Hold that thought for §8.

⚠ **Cross-reference, don't re-derive.** The survey's five modules are one decomposition among several; this repo already owns the architecture discussion. For the autonomy spectrum and architectural families, see [autonomous_agents_guide.md](autonomous_agents_guide.md) §3 — do not read the five-module figure as the field's architectural consensus. For the *pipeline* view of the same modules in motion, [multi_pass_llm_pipelines_guide.md](multi_pass_llm_pipelines_guide.md). For what happens to memory in operation — retention policy, staleness, context budgets — [context_engineering_guide.md](context_engineering_guide.md) and [agent_runtime_cache_design_guide.md](agent_runtime_cache_design_guide.md).

### 3.4 The one thing to take from §2 of the survey

The framework's practical value is as a **checklist for a loop design**: which module produces the signal, which module consumes it, and does the consumption path actually exist? A loop whose feedback module is tightly coupled to planning but has no write path to memory will silently re-derive the same lesson every time. A loop coupled to memory but not planning accumulates opinions it never acts on. Both are common in practice, and neither is visible from the outside — which is why §14's anti-pattern list starts with symptoms rather than with mechanisms.

---

## 4. Taxonomy Category 1 — Internal Feedback

### 4.1 What the survey means by internal

The survey's §3.1 defines internal feedback as feedback that "originates from the agent itself and serves to motivate proactive exploration, learning, and self-improvement", explicitly not relying "on explicit external goals", and manifesting "as signals generated and utilized by the model itself". The canonical example given is Chain-of-Thought-based feedback: prompt the model to produce an answer with explanations, "identify and reinforce reasoning chains that lead to correct answers", and fine-tune the behaviour — "improving performance even without explicit external supervision".

That last clause is doing a lot of work in the survey's sentence and it is precisely the claim the negative literature attacks. Hold it.

The survey splits internal feedback by whether cross-task knowledge is used:

- **Intra-task feedback** — "feedback derived from an agent's historical steps during its trial-and-error interactions with the environment". The survey calls it "the most relevant and informative signals to guide the agent's future actions". Its examples: **ReAct** (interleaved reasoning traces and task-specific actions) and **Self-Refine** (iterative feedback and refinement within the same task).
- **Inter-task feedback** — "the transfer of knowledge and experience across tasks". Its examples: **Reflexion** (derives experiences from past tasks, applies them to subsequent tasks), **ExpeL** (retrieves and analyses similar past trajectories, "identifying success patterns by contrasting positive and negative examples").

### 4.2 The representative systems, per the survey's Table 1

Table 1 gives each system a **format** (NL = natural language, Scalar; with slash notation where both), a **learning** strategy (ICL in-context learning, SL supervised learning, RL reinforcement learning, IL imitation learning), an **iteration** marker (✓/✗) and a **domain**. Reproduced for the internal-feedback rows, attributed to the survey:

| Sub-category | System | Format | Learning | Iteration | Domain |
|---|---|---|---|---|---|
| Intra-task | ReAct [Yao et al., 2023] | NL | ICL | ✗ | QA, Fact Verification |
| Intra-task | "Reflextion" [Shinn et al., 2024] *(survey's rendering — see §16)* | NL | ICL | ✓ | QA, Code Generation |
| Intra-task | Self-Refine [Madaan et al., 2024] | NL | ICL | ✓ | Multiple Tasks |
| Intra-task | Self-Check [Miao et al., 2024] | NL | ICL | ✓ | Arithmetic Reasoning |
| Intra-task | AdaPlanner [Sun et al., 2023] | Code | ICL | ✓ | Embodied Action, Web Browsing |
| Intra-task | Mirror [Yan et al., 2024] | NL | ICL | ✓ | Reasoning, Fact Verification |
| Intra-task | Self-Correct [Welleck et al., 2023] | NL / Scalar | SL | ✓ | Reasoning, Generation, Toxicity |
| Inter-task | Reflexion [Shinn et al., 2024] | NL | ICL | ✓ | QA, Code Generation |
| Inter-task | ExpeL [Zhao et al., 2024] | NL | ICL | ✓ | Multiple Tasks |
| Inter-task | Retroformer [Yao et al., 2024] | NL | SL | ✓ | QA, Embodied Action, Web Browsing |
| Inter-task | RAP [Kagaya et al., 2024] | Scalar | ICL | ✓ | Planning, Reasoning |
| Inter-task | Generative Agents [Park et al., 2023] | NL | ICL | ✗ | Social Simulation |
| Inter-task | PreAct [Fu et al., 2025] | NL | ICL | ✓ | Multiple Tasks |
| Inter-task | Trial and Error [Song et al., 2024] | Scalar | IL & RL | ✓ | Web Navigation, Embodied Action |

Three observations the survey does not make about its own table, all of which a reader needs:

**(a) Reflexion appears twice — once as intra-task and once as inter-task.** The survey lists "Reflextion [Shinn et al., 2024]" under intra-task and "Reflexion [Shinn et al., 2024]" under inter-task. Reflexion's mechanism is retrospective reflection across attempts, which the survey itself describes as inter-task in §3.1 ("derives experiences from past tasks and applies them in subsequent tasks"). The duplicate is either a drafting artefact or a deliberate note that Reflexion can operate in both registers; the survey does not say which. Either way, **do not read the table as a partition** for this row. ⚠

**(b) The iteration markers contradict the survey's own body text and its own Table 3.** Table 1 marks **ReAct** as non-iterating (✗). ReAct's interleaved reason-act loop is the mechanism; the survey's §4.2 also lists ReAct's "Feedback Round" in Table 3 as **Multi-round**. Two tables, two answers. Table 1 also marks **Voyager** (✗), **JARVIS-1** (✗) and **MetaGPT** (✗) as non-iterating while the survey's body describes each as iterative — MetaGPT with "an executable feedback loop for iterative code refinement", and Voyager as an "iterative hinting mechanism". The primary Voyager paper's own abstract describes "a new iterative prompting mechanism that incorporates environment feedback, execution errors, and self-verification for program improvement" **— the primary source directly contradicts the survey's table cell.** ⚠❌ *(primary says iterative; the survey's table says ✗)*

**(c) "Iteration" is an ambiguous column.** It is marked at the mechanism level, but iteration in these systems happens at two different levels — refining *this* attempt, or re-attempting the *task*. A ✓ in one row can mean either. Table 3's "Feedback Round" column is the survey's attempt to make this concrete for a subset of systems, and §9 examines what it actually measures.

### 4.3 The survey's own trade-off discussion

The survey's §3.1 Discussion is the clearest piece of analysis in the taxonomy sections, and it is worth taking at face value:

> "Intra-task feedback enables rapid optimization for the current task, but its task-specific nature limits broader knowledge learning and accumulation. Conversely, inter-task feedback facilitates cross-task learning and long-term experience integration, while it demands substantial memory to store historical task experience. The fundamental challenge lies in effectively combining these complementary approaches to harness their respective strengths."

Translated into design terms: **intra-task is a quality lever with a per-task ceiling; inter-task is a capability lever with a memory and hygiene bill.** Intra-task refinement can only make the current attempt as good as the model can judge the current attempt — and §8 says that judgement is the weak link. Inter-task accumulation can change what the system is capable of, and it also means the system's behaviour is a function of its history, which is exactly what [ai_agent_drift_guide.md](ai_agent_drift_guide.md) exists to make observable.

The survey frames the combination question as *the* fundamental challenge. That framing is honest but incomplete: the harder version of the question is not "how do we combine intra- and inter-task feedback" but "how do we know whether either is helping", and the survey's §4 (evaluation) does not answer it for internal feedback specifically.

### 4.4 The honest boundary question: where does internal stop and external begin?

This is the place where the survey's category boundary is most porous, and it deserves to be stated as a genuine open question rather than resolved.

The survey's definitional line is *source*: internal feedback "originates from the agent itself". But consider four mechanisms that all produce "the agent's own critique":

1. **A self-critique with nothing behind it.** The model is asked "is your answer correct?" and says "no, let me reconsider". The signal is the model's prior over its own output. No new information enters the system. This is the case the negative literature is about, and it is the case where the loop can make answers worse.
2. **Self-consistency-style agreement.** The model samples several answers and takes the majority. The signal is the distribution over its own generations — still no external information, but a form of aggregation that can be more robust than a single sample. Internal by the survey's definition; empirically less vulnerable than case 1 because it does not require the model to *recognise* its own error.
3. **Self-critique grounded in a tool result the agent obtained.** The model fetched a document, and the critique reasons over the fetched document. The critique is written by the model, but the *evidence* is external. The survey would file this under internal feedback (the mechanism generates the critique) or under external feedback (the agent "leverages external models or tools") depending on the level of description — the taxonomy genuinely does not decide.
4. **Self-verification against a formal artefact.** Voyager's self-verification runs the generated code and checks behaviour. Again the critiquing machinery is internal; the ground truth is not.

Cases 3 and 4 are the interesting ones, because they are the mechanisms that actually work in practice, and the survey's taxonomy cannot tell the difference between them and case 1. **A self-generated critique grounded in nothing is not the same as a self-generated critique grounded in a tool result** — the first is the model asking itself, the second is the model reading the world and then asking itself. Collapsing both into "internal feedback" loses the distinction that determines whether the loop helps.

The practical rule this guide proposes, and which the survey does not state: **classify by the grounding of the signal, not by who wrote the critique.** The question to ask of any internal-feedback candidate is not "does the agent critique itself?" but "what does the critique have access to that the original answer did not?"

### 4.5 What internal feedback is structurally good at

It would be a mistake to read this section as "internal feedback is bad". Two uses are well-founded even under the strongest negative results:

- **Cheap format and constraint compliance.** Checking whether an output satisfies a schema, contains all required fields, avoids a banned pattern, or matches a template is a task where the check is easier than the generation — and where a weak verifier is adequate because the property is surface-level and enumerable. No external information is needed because the criterion is in the spec.
- **Search-space widening under a stopping rule.** Generating several candidate approaches and selecting among them is not the same as "correcting a mistake" — it does not require the model to be able to *detect* its own error, only to generate variation. ⚠ Where the selection is by the model's own judgement, the signal problem returns; where selection is by an external check, it is genuinely external feedback and belongs in §5.

What internal feedback is *not* reliably good at is exactly what it is sold as: **detecting and repairing errors in multi-step reasoning.** That is the finding of §8, and it is not a niche result — it is the mainstream position in the sub-literature by 2024.

---

## 5. Taxonomy Category 2 — External Feedback

### 5.1 What the survey means by external

The survey's §3.2 defines external feedback as "environment-defined signals that evaluate and guide an agent's behavior toward goal achievement", notes that in reinforcement learning "scores, task completion indicators, and success rates are all forms of external feedback", and states that in AI agents external feedback "is usually provided by independent feedback models or external tools (such as compilers, search engines, or world models)". It then splits the category by "the external modules involved" into four sub-sources, and the survey's overview table marks all but one of them as iterative.

| Sub-source | Survey's description | Representative systems per the survey | Format / learning (survey's Table 1) |
|---|---|---|---|
| **Web knowledge** | Searching and browsing online content, then refining answers | WebGPT [Nakano et al., 2021]; AutoWebGLM [Lai et al., 2024]; **WebShop** [Yao et al., 2022] | NL, IL & RL / Scalar, IL & RL |
| **Game API** | Execution errors and self-validation from a game's programmatic interface | **Voyager** [Wang et al., 2023]; JARVIS-1 [Wang et al., 2024c]; CALM [Yao et al., 2020]; CRADLE [Tan et al., 2024] | NL, ICL / NL, Scalar, RL |
| **Code interpreter** | Compiler and execution feedback on generated code | **StepCoder** [Dou et al., 2024]; CodeAgent [Zhang et al., 2024]; QueryAgent [Huang et al., 2024]; CodeRL [Le et al., 2022] | Scalar, RL / NL+Code, ICL |
| **World model** | Sensory/environmental data and learned environment models | RT-1 [Brohan et al., 2022]; RT-2 [Zitkovich et al., 2023]; **PaLM-E** [Driess et al., 2023]; WorldCoder [Tang et al., 2024] | Scalar, IL & SL / Code, RL |

The survey's own examples are worth quoting because they show the category's span in a single paragraph. **WebGPT**: "enables web-based question answering by searching and browsing online content, then refining answers through user feedback." **Voyager**: "introduces an iterative hinting mechanism that combines execution errors and self-validation to enhance program quality." **StepCoder**: "learns from compiler feedback, improving its ability to understand complex requirements and generate better code." **PaLM-E**: "integrates real-world sensor feedback and visual feedback into a language model to establish a link between words and perceptions." ⚠ Note that the Voyager row is filed under **external** feedback (game API) in §3.2 while Voyager's own described mechanism is "execution errors **and self-validation**" — the survey itself has placed a composite internal/external mechanism in the external cell, which is the boundary porosity of §4.4 showing up again, this time from the other side.

### 5.2 The survey's discussion

The survey's §3.2 Discussion is short and, read literally, concedes the whole problem:

> "External feedback offers LLM-based AI agents additional insights that they might not obtain through self-improvement alone. However, the effectiveness of such feedback is highly dependent on the quality and relevance of external information. Therefore, optimizing the acquisition and processing mechanism of external feedback is crucial to boosting the performance of LLM-based AI agents."

"The effectiveness of such feedback is highly dependent on the quality and relevance of external information." That sentence is the survey's own statement of this guide's thesis — for this category. It is stated as an optimisation remark rather than a design principle, and the survey does not follow it to its conclusion, which is that a taxonomy organised by *where the signal comes from* is not the taxonomy a practitioner needs. The needed taxonomy is organised by *what the signal is worth*.

### 5.3 The analysis the survey does not give: which signals can be trusted

Here is the claim the survey's four sub-sources obscure. A compiler's error message and a world model's prediction are not near-neighbours in usefulness; they are at opposite ends of a spectrum, and the spectrum has at least five measurable dimensions.

| Dimension | What it asks | Compiler / interpreter / game API | Search / web knowledge | Learned world model | LLM critic / judge |
|---|---|---|---|---|---|
| **Verifiability** | Can the signal's correctness be decided independently of any model? | Yes — by construction | No — relevance is a judgement | No — the model is the ground truth being approximated | No — it is another model's opinion |
| **Determinism** | Same input, same output? | Yes | No (index, ranking, content drift) | Usually deterministic given weights, but wrong in unpredictable places | No (sampling, version drift) |
| **Cost / latency** | What does one signal cost? | Microseconds to seconds, bounded, cheap | Network round-trip, rate limits, sometimes expensive per call | High — a rollout or a forward pass through a policy | One to many model calls; scales with the number of critique rounds |
| **Failure mode** | How does it go wrong? | Fails loudly and locally (a type error, a non-zero exit) | Fails quietly (returns plausible irrelevant text) | Fails plausibly — a confident wrong prediction | Fails plausibly **and** sycophantically — agrees with the output it is shown |
| **Blast radius if trusted blindly** | What happens when the agent believes it? | The next attempt is wrong in a visible way | Confidently wrong answers with citations that do not support them | Silent policy drift into a state the model imagines to exist | The loop ratifies errors instead of removing them (§8) |

Read down the last row and the category collapses. Two of the four sub-sources (compiler/interpreter, and most game APIs) give the agent a signal it **cannot argue with**; one (web search) gives text that may or may not bear on the question; one (world model) gives the output of a system that is itself wrong sometimes and has no way to tell you when. Treating those as one category — "external feedback" — actively misleads, because it suggests the design question is *which external module to wire in*, when the design question is *whether the signal can be trusted when the agent disagrees with it*.

**The practitioner's first question, therefore, is not "which category does this system fall into?" but "which signals can be trusted?"** Three tests, in order:

1. **Is the signal decidable without a model?** A unit test, a schema validator, a type checker, a SQL `EXPLAIN` that fails, an HTTP 4xx, a reconciliation that does not balance. If yes, close the loop on it and stop reading this guide's §8 — that loop is a control-shaped loop (§12) because it has an independent oracle.
2. **If not decidable, is the signal sourced from something the agent did not author?** A document it retrieved, a log it read, a second implementation produced by different tooling. The critique can still be the model's, but the evidence must not be. This is the boundary case of §4.4 and it is where the useful loops live.
3. **If neither, the signal is the model's own opinion.** Then the loop's value is not established by the loop's existence. §8 says why, and §14 gives the symptoms.

⚠ **Cross-reference.** For building the verification side of a loop — what to check, with what, and how to make failures loud — see [ai_verify_toolkit_guide.md](ai_verify_toolkit_guide.md). For what a *trained, independent* checker looks like (guard models, classifiers, reward-free verifiers) see [llm_guard_models_guide.md](llm_guard_models_guide.md) — a guard model is a feedback mechanism, and in most production stacks it is a better one than a prompt because it is a separate artefact with its own evaluation. For the retrieval half of "web knowledge", see [agentic_search_vs_rag_guide.md](agentic_search_vs_rag_guide.md), which owns the question of when retrieval is a signal and when it is noise.

### 5.4 The one thing to take from §3.2 of the survey

The external-feedback category is where the survey's taxonomy is *most* useful and *least* sufficient at the same time. It is useful as an inventory — the four sub-sources are a good checklist of where signals come from in a modern agent stack, and the representative systems give you something to read. It is insufficient because the inventory flattens the quality axis that decides outcomes. Carry the inventory; build against the trust ladder.

---

## 6. Taxonomy Category 3 — Multi-Agent Feedback

### 6.1 What the survey means by multi-agent

The survey's §3.3 defines multi-agent feedback as a mechanism that "mimics multi-agent interactions where diverse perspectives converge to improve solutions", promoting "knowledge sharing and stronger decision-making" in "either collaborative or adversarial ways." The two sub-categories as the survey states them:

- **Collaborative** — "each agent presents and discusses its own points of view in multiple rounds, with the optimal solution as the final answer." Representative: **MetaGPT** ("employs an executable feedback loop for iterative code refinement, with agents collaboratively enhancing code quality using execution history and debug logs"); **InteRecAgent** ("a dual-role feedback mechanism, pairing a recommender agent with a reviewer agent to evaluate results and identify execution errors").
- **Adversarial** — "multiple agents to debate the same viewpoints in multiple rounds until a consensus is reached." Representative: **Du et al. [2024a]**'s multi-agent debate framework ("multiple agents expressing their arguments in a 'tit-for-tat' state, and a judge manages the debate process toward final resolutions").

Table 1's rows for this category, attributed to the survey:

| Sub-category | System | Format | Learning | Iteration | Domain |
|---|---|---|---|---|---|
| Collaborative | MetaGPT [Hong et al., 2024] | NL / Code | ICL | ✗ | Program |
| Collaborative | InterAct [Chen and Chang, 2023] | NL | ICL & RL | ✗ | Embodied Action |
| Collaborative | InteRecAgent [Huang et al., 2023] | NL | RL | ✓ | Recommendation |
| Collaborative | AnyTool [Du et al., 2024b] | Scalar | RL | ✗ | Tool Usage |
| Adversarial | ChatLLM [Hao et al., 2023] | NL | ICL | ✓ | Classification, Sentiment Reversal |
| Adversarial | Multiagent Debate [Du et al., 2024a] | NL | ICL | ✓ | Reasoning, Factuality |
| Adversarial | ChatEval [Chan et al., 2023] | NL | ICL | ✓ | Text Evaluation |

Again the iteration column fights the body text: MetaGPT is marked ✗ while the survey's §3.3 describes it as having "an executable feedback loop for iterative code refinement". ⚠ See §16 for the full list of internal inconsistencies in the source.

### 6.2 The three challenges the survey names — and the systems it cites for each

This is the most concrete passage in the survey's taxonomy sections, and it maps cleanly onto production concerns. The survey names three persistent challenges and cites one solution-system per challenge. Report them as the survey's selections, not as settled remedies:

| Challenge (survey's words) | Survey's description of the failure | Survey's cited solution | What the survey says it does |
|---|---|---|---|
| **Coordination** | "when three or more agents freely express their views, discussions may become uncontrolled" | **ChatLLM** [Hao et al., 2023] | "introducing a dedicated coordination agent … can integrate responses to refine the final answer" |
| **Reward alignment** | "misaligned goals and reward functions among agents may lead to conflicts or suboptimal outcomes" | **CollaQ** [Zhang et al., 2020] | "decomposing reward allocation to provide an innovative solution for decentralized Q-functions" |
| **Learning stability** | "multi-agent negotiations may converge on incorrect consensuses" | **PRD** [Li et al., 2023] | "peer-ranking algorithm analyzes pairwise preferences to generate a final ranking, ensuring more accurate consensus answers" |

Note the shape of the third challenge. "Multi-agent negotiations may converge on incorrect consensus" is structurally the same failure as the one in §8 — a group of models agreeing on something false because agreement is what the procedure rewards. The survey names the failure and prescribes a peer-ranking fix; what it does not do is connect this to the single-agent version of the same problem, where the fix has to come from outside the set of participants rather than from a cleverer aggregation of their votes.

### 6.3 The protocol-standardisation thread, and where it belongs in this repo

The survey's §6.1 raises standardisation as a *future direction* under the real-time/multi-agent challenge: "a lack of standardized protocols also complicates interactions, especially when agents come from different vendors and use different architectures", with the direction "promoting protocol standardization, such as **MCP, A2A, ANP, and Agora**, for building large-scale agent systems."

Two things follow. First, standardisation is a **feedback-infrastructure** concern, not a side issue: a feedback signal that can only be produced by one vendor's agent cannot be part of a control surface you can audit. Interoperable tool calls are what make external, verifiable feedback available to an agent you did not build. Second, this repo already owns the mechanics — the survey is naming the direction, and the MCP cluster covers the implementation. ⚠ **Cross-reference by name:** [mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md), [mcp_discovery_guide.md](mcp_discovery_guide.md), [mcp_framework_tools_guide.md](mcp_framework_tools_guide.md). For orchestration patterns above the protocol layer, [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md) and [hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md); for the multi-agent banking-domain view specifically, [multi_agent_banking_guide.md](multi_agent_banking_guide.md).

### 6.4 The honest note the survey does not give: same-model debate is not independent verification

This is the single most over-claimed mechanism in the field, and it is worth being blunt about.

**A debate between instances of the same model is not independent verification unless something makes it so.** The instances share: the same training data, the same pretraining biases, the same failure modes, and — critically — the same tendency to be persuaded by fluent arguments and to cave to confident interlocutors. Du et al.'s multi-agent debate reports improved factuality and reasoning, and the survey reports that faithfully. But the mechanism's *decorrelation* is the open question: if five samples of the same model are wrong in the same way, a vote among them will be confidently wrong, and the debate procedure guarantees the wrongness gets a hearing rather than a challenge.

What actually creates independence in a multi-agent setup:

| Source of independence | Real? | Why |
|---|---|---|
| Different model families | Partial | Different pretraining corpora and RLHF stacks decorrelate some errors, not all — and shared web-scale training data means shared blind spots. |
| Different **tools** | Yes | If one agent can run the code and another cannot, the first has evidence the second lacks. The independence is in the tool, not the agent. |
| Different **information** | Yes | If agents retrieve from different sources, disagreement carries information. |
| A **verifier outside the debate** | Yes | The only mechanism that makes the debate's output checkable — and per §8, this is where the gains actually come from. |
| Different **roles/prompts** | Mostly not | Role-play is a style change, not an epistemic one. |
| Different **sampling seeds** | No | Diversity of phrasing, not of belief. |

The practical rule: **count the number of independent evidence channels in a multi-agent system, not the number of agents.** A three-agent debate with one shared tool and one shared model family has one evidence channel and three opinions; it is a committee, not a verification. §14 files the confusion of the two as an anti-pattern. ⚠ For the orchestration-side consequences of treating debate as verification (cost, latency, and the compounding-error arithmetic across more calls), see [agents_work_fall_apart_guide.md](agents_work_fall_apart_guide.md) §4 and [multi_pass_llm_pipelines_guide.md](multi_pass_llm_pipelines_guide.md) §4.

---

## 7. Taxonomy Category 4 — Human Feedback

### 7.1 The three forms the survey names

The survey's §3.4 states that human feedback "enables agents to iteratively refine their behavior according to human feedbacks while performing the task", and names three primary forms. Table 1's rows, attributed to the survey:

| Form | Survey's definition | Representative systems | Format / learning (Table 1) |
|---|---|---|---|
| **Instructional** | "provides direct task guidance through explicit human instructions" | InstructGPT [Ouyang et al., 2022]; WebGPT [Nakano et al., 2021]; COACH [Arumugam et al., 2019] | NL, SL & RL (✗) / NL, IL & RL (✓) / NL, RL (✗) |
| **Corrective** | "is provided when the agent's behavior deviates or contains errors, and humans intervene to correct these mistakes or guide improvements" | IBT [Abramson et al., 2022]; TAMER [Warnell et al., 2018]; ReHAC [Feng et al., 2024b] | NL, IL & RL (✓) / Scalar, RL (✗) / NL, RL (✓) |
| **Preference** | "shapes agent behavior through human preferences or choices" | PrefCLM [Wang et al., 2025]; DRLHF [Christiano et al., 2017]; Sparrow [Glaese et al., 2022] | Scalar, RL (✗) / Scalar, RL (✗) / NL, SL & RL (✓) |

The survey's illustrative examples are consistent with the primaries in orientation: WebGPT "uses imitation learning to train models by setting tasks in a way that humans can perform them and then optimizes the quality of answers through user feedback"; Abramson et al. "developed a 3D simulation framework capturing human corrections when agents stray from goals"; ReHAC "includes a policy model designed to determine the most opportune stages for human intervention within the task-solving process"; PrefCLM "introduces a human-in-the-loop pipeline that facilitates collective refinements based on user comparative feedback to train robots."

### 7.2 The category analysis the survey does not give: preference trains, corrective steers

**(f)(i) — the survey's human-feedback category mixes a training-time procedure with a runtime mechanism, and collapsing the two hides a difference that matters.**

Look at the examples side by side. **InstructGPT's** preference data (via RLHF) changes the *weights* of a model that is then deployed. **Sparrow's** human judgements change the weights. **Christiano et al.'s** preference comparisons train a reward model. Every one of those is a **training-time procedure**: expensive, batch, offline, done by a labelling organisation, and it produces a model whose behaviour changes for all future inputs. **ReHAC's** intervention policy decides *at what stage during a running task* a human should step in — a runtime mechanism. **IBT's** captured human corrections are runtime interventions recorded for later training. **WebGPT's** user feedback during browsing is runtime.

These two things differ along every axis a practitioner cares about:

| Axis | Preference feedback (trains) | Corrective feedback (steers) |
|---|---|---|
| When it acts | Before deployment | During execution |
| Who does it | A labelling/training organisation, at volume | An operator or user, in the moment |
| What it changes | Weights — all future behaviour | The current task trajectory (and possibly the context of this session) |
| Cost structure | Fixed, high, batch, amortised over many sessions | Marginal, unbounded, tied to the number of interventions |
| Failure mode | Bias amplification and overfitting to rater preferences; evaluation needs to detect it *before* deployment | Fatigue, inconsistency, alarm fatigue, and the operator becoming the de-facto decision-maker without the accountability of one |
| Governance question | "Whose preferences shaped this model, and how do we know?" | "Who is on the hook for this specific decision?" |
| Reversibility | A rollback means retraining or reverting a model version | A correction can be undone; the trajectory just diverges |
| Evidence it leaves | Rater data, reward model, training run — versionable | An intervention in a trace — reconstructable only if you log it |

Collapsing them under one heading means that a design conversation about "human feedback" can silently switch from "we will source preference data for a future fine-tune" to "a human will watch every run" — two proposals with wildly different costs, staffing implications and governance consequences. Keep them separate in your own vocabulary even if the survey does not. ⚠ For the weight-shaping half, the repo's owners are [llm_instruction_tuning_guide.md](llm_instruction_tuning_guide.md) and [fine_tuning_frameworks_comparison_guide.md](fine_tuning_frameworks_comparison_guide.md); for the data pipeline that carries human labels back into training, [closed_loop_data_engineering_guide.md](closed_loop_data_engineering_guide.md).

### 7.3 What the survey says about the impracticality of continuous human feedback

The survey's §3.4 Discussion is unusually direct, and it is the passage that a sponsor needs to read before approving a human-in-the-loop design:

> "relying solely on human feedback is often impractical due to limited resources or real-time constraints, as it often requires continuous monitoring, evaluation, and adjustment by experts to ensure the agent aligns with desired outcomes."

That is the survey conceding the cost problem in one sentence. Taken together with §7.2, the practical conclusion is that human feedback is a **scarce resource to be allocated**, not a design pattern to be sprinkled on. Two consequences the survey does not draw: (a) a runtime human-in-the-loop is only affordable where each intervention buys something a cheaper signal cannot; (b) an approval gate that is *always* triggered is not a control, it is a queue — and queues get bypassed or rubber-stamped under load (§12, §14).

### 7.4 The substitute, and its named biases

The survey reports the field's answer to the cost problem: **LLM-as-a-judge** [Zheng et al., 2023], which it describes as leveraging "the scalability and interpretability of LLMs to reduce the need for human intervention, providing scores and explanations, **but it suffers from positional bias, verbosity bias, and limitations in evaluating mathematical problems**."

✅ **Verified at the primary source, with one omission.** The primary is Zheng et al., *Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena*, and its abstract names the limitations explicitly: "we examine the usage and limitations of LLM-as-a-judge, including **position, verbosity, and self-enhancement biases, as well as limited reasoning ability**." Two things follow. First, the survey's rendering is accurate on positional and verbosity bias and on the mathematical-reasoning limitation ("limited reasoning ability" in the primary). Second, the primary names a bias the survey omits — **self-enhancement bias**, the tendency of a judge to favour outputs from models like itself — which is the one that matters most for a loop where the judge and the judged are the same model. The survey's list is shorter than the primary's; use the primary's.

The survey also reports **Agent-as-a-Judge** [Zhuge et al., 2024] as a step beyond scoring: "integrates agent capabilities to provide intermediate feedback during task resolution, achieving better consistency with human evaluators." ✅ Verified in substance at the primary: the Agent-as-a-Judge paper introduces the framework and its own DevAI benchmark (55 tasks, 365 hierarchical user requirements) and reports that it "dramatically outperforms LLM-as-a-Judge and is as reliable as our human evaluation baseline". ⚠ Two caveats: (a) the survey cites "DevAI [Zhuge et al., 2024]" and "Agent-as-a-Judge [Zhuge et al., 2024]" as two separate works — they are **the same paper**, DevAI being that paper's benchmark; (b) "as reliable as our human evaluation baseline" is a claim about *the authors' own benchmark*, evaluated by the authors — which is the internal-validity limit of every judge paper, and not a licence to treat an agent judge as an independent control (§12).

### 7.5 The ethics passage

The survey's §3.4 closes with an ethical concern worth reproducing because it is one of the few places the paper steps outside its pro-loop framing:

> "human feedback mechanisms have the potential to introduce biases into the training data, particularly along lines of gender, race, and culture, which in turn can amplify the inherent biases within LLMs. To mitigate these risks, future research should prioritize the development of more diverse learning methodologies, such as the integration of culturally representative datasets and the enhancement of cultural awareness in model design."

That is a correct and underweighted point: human feedback is the *canonical* mechanism by which a model inherits its raters' blind spots, and a feedback loop that learns from human corrections learns from whoever happens to be on shift. The survey's proposed mitigation (diverse datasets, cultural awareness in design) is directional rather than operational; the operational version lives in this repo at [ai_governance_bias_redteaming_guide.md](ai_governance_bias_redteaming_guide.md), [ai_red_teaming_guide.md](ai_red_teaming_guide.md) and [implementing-responsible-ai.md](implementing-responsible-ai.md). §12 of this guide treats the same material as a **model-risk and data-governance** question rather than an ethics remark.

---

## 8. The Counter-Evidence — What Self-Correction's Own Literature Says

> This is the guide's most important section. The anchor survey is broadly optimistic about feedback loops; the primary literature is not. Everything below is stated with its own venue, verified at the primary source, and reported without resolving the tension in either direction.

### 8.1 The headline finding: reasoning does not self-correct without an external signal

The well-known 2024 result, verified at the primary source:

| Field | Value |
|---|---|
| Title | *Large Language Models Cannot Self-Correct Reasoning Yet* |
| Authors | Jie Huang, Xinyun Chen, Swaroop Mishra, Huaixiu Steven Zheng, Adams Wei Yu, Xinying Song, Denny Zhou |
| Venue | **ICLR 2024** (arXiv:2310.01798; submitted 3 Oct 2023, revised 14 Mar 2024) |
| The exact claim (abstract, verbatim) | "Central to our investigation is the notion of **intrinsic self-correction**, whereby an LLM attempts to correct its initial responses based solely on its inherent capabilities, without the crutch of external feedback. In the context of reasoning, our research indicates that **LLMs struggle to self-correct their responses without external feedback, and at times, their performance even degrades after self-correction**." |

Two words in that abstract do the work: *intrinsic* and *degrades*. The paper is not claiming that all refinement fails; it is claiming that refinement without an external signal fails, in reasoning, and can make things worse. "At times, their performance even degrades after self-correction" is the sentence that kills the folk version of the loop — the version where iteration is assumed to be monotone improvement.

### 8.2 Self-critique degrades planning, and the verifier is unreliable

| Field | Value |
|---|---|
| Title | *Can Large Language Models Really Improve by Self-critiquing Their Own Plans?* |
| Authors | Karthik Valmeekam, Matthew Marquez, Subbarao Kambhampati |
| Venue | arXiv preprint (12 Oct 2023), arXiv:2310.08118 — ⚠ **peer-reviewed venue not verified in this guide** |
| The exact findings (abstract, verbatim) | "Using GPT-4, a state-of-the-art LLM, for both generation and verification, our findings reveal that **self-critiquing appears to diminish plan generation performance**, especially when compared to systems with external, sound verifiers and **the LLM verifiers in that system produce a notable number of false positives, compromising the system's reliability**. Additionally, **the nature of feedback, whether binary or detailed, showed minimal impact on plan generation**. Collectively, our results cast doubt on the effectiveness of LLMs in a self-critiquing, iterative framework for planning tasks." |

Three findings, three consequences for anyone building a loop: self-critique made planning *worse*; the model-as-verifier produced a notable rate of **false positives** (it passed bad plans — the direction of error that matters most, because a false positive ends the loop and ships the output); and **the richness of the feedback did not matter** — detailed critique did no better than a binary signal. That last finding is the quiet one. If the content of the critique does not affect the outcome, then the loop is not using the critique as reasoning; it is using it as a perturbation.

### 8.3 Self-critique collapses performance; sound external verification produces the gains

| Field | Value |
|---|---|
| Title | *On the Self-Verification Limitations of Large Language Models on Reasoning and Planning Tasks* |
| Authors | Kaya Stechly, Karthik Valmeekam, Subbarao Kambhampati |
| Venue | arXiv:2402.08115 (12 Feb 2024; v2 3 Aug 2024) — ⚠ **peer-reviewed venue not verified in this guide**; the arXiv record notes text overlap with arXiv:2310.12397 |
| Domains | GPT-4 on Game of 24, Graph Coloring, and STRIPS planning |
| The exact findings (abstract, verbatim) | "**We observe significant performance collapse with self-critique and significant performance gains with sound external verification. We also note that merely re-prompting with a sound verifier maintains most of the benefits of more involved setups.**" |

This is the cleanest single piece of evidence in the section, and it is the source of this guide's central claim. The experiment compared self-critique against an external correct reasoner verifying proposed solutions, analysed whether the *content* of criticisms affected bottom-line performance, and ablated elements of the augmented system. The results: self-critique → performance collapse. Sound external verification → significant gains. **And the elaborate apparatus is not where the gain lives** — "merely re-prompting with a sound verifier maintains most of the benefits of more involved setups."

Read that against a typical loop design document — a critique stage, a debate stage, a memory of reflections, a sophistication budget — and it says: most of the value is in having a **sound verifier**, and the machinery wrapped around it is mostly decoration. This is the "the gains come from the external evaluation rather than from the model's introspection" finding, stated by the primary authors in their own abstract. The ablation is the argument.

### 8.4 Self-repair gains are modest, cost-sensitive, and bottlenecked by self-feedback

| Field | Value |
|---|---|
| Title | *Is Self-Repair a Silver Bullet for Code Generation?* |
| Authors | Theo X. Olausson, Jeevana Priya Inala, Chenglong Wang, Jianfeng Gao, Armando Solar-Lezama |
| Venue | **ICLR 2024** (poster; OpenReview `y0GJXRungR`; arXiv:2306.09896) |
| The exact findings (abstract, verbatim) | "when **the cost of carrying out repair is taken into account, performance gains are often modest, vary a lot between subsets of the data, and are sometimes not present at all**. We hypothesize that this is because **self-repair is bottlenecked by the model's ability to provide feedback on its own code**; using a stronger model to artificially boost the quality of the feedback, **we observe substantially larger performance gains**. Similarly, a small-scale study in which we provide GPT-4 with feedback from human participants suggests that **even for the strongest models, self-repair still lags far behind what can be achieved with human-level debugging**." |

Four findings, and note how they interlock with §8.2 and §8.3. Gains are modest **once you count the cost** of the repair attempt — the standard presentation of a self-repair loop omits the compute of the extra attempts, which is exactly the accounting error that makes a decoration look like a mechanism. Gains **vary a lot between subsets** — a single average is hiding a distribution that includes catastrophic cases. The bottleneck is **the feedback**, not the repair — and when you improve the *feedback* (a stronger model as critic, or human feedback), the gains get substantially larger. In other words: **the loop is not the improvement; the signal is.**

⚠ **Important flag on this citation.** The survey cites this same work as **"Olausson et al., 2023. Demystifying GPT Self-Repair for Code Generation. Arxiv, 2023"** — the pre-publication title and status — and it appears in the survey's §4.1 as an example of *outcome-based evaluation*: "analyze LLMs' self-repair capabilities on the APPS coding benchmark, where the feedback mechanism is evaluated by the pass rate of programming tasks." That description is not false, but it is the paper's **set-up**, not its finding. The survey's summary omits the paper's central negative result entirely. This is the single clearest case in the anchor paper where *what the survey reports* and *what the primary source says* diverge in substance rather than in detail — and §15 records it as such.

### 8.5 The critical survey: which self-correction claims survive scrutiny

| Field | Value |
|---|---|
| Title | *When Can LLMs Actually Correct Their Own Mistakes? A Critical Survey of Self-Correction of LLMs* |
| Authors | Ryo Kamoi, Yusen Zhang, Nan Zhang, Jiawei Han, Rui Zhang |
| Venue | **TACL 2024, 12:1417–1440** (arXiv:2406.01297) |
| The exact findings (abstract, verbatim) | "Our critical survey based on the newly categorized research questions shows that **(1) no prior work demonstrates successful self-correction with feedback from prompted LLMs, except for studies in tasks that are exceptionally suited for self-correction, (2) self-correction works well in tasks that can use reliable external feedback, and (3) large-scale fine-tuning enables self-correction.**" |
| Also (abstract, verbatim) | "prior studies often do not define their research questions in detail and involve **impractical frameworks or unfair evaluations that over-evaluate self-correction**" — with a checklist offered for designing appropriate experiments |

This is the field's own meta-analysis of the same literature the anchor survey summarises, and it is the reason this guide treats "self-correction improves the answer" as a **conditional** claim rather than a general one. The conditions: *reliable external feedback available* (finding 2), *a task exceptionally suited to self-correction* (finding 1's exception), or *large-scale fine-tuning that installs the capability in the weights* (finding 3). Everything else — prompted LLM critiquing a prompted LLM's answer, no external signal — has, in this survey's reading of the literature, **no demonstrated success**.

### 8.6 The survey's own concession: §6.6

The anchor survey is not silent on this. Its §6.6 is titled **"Inherent Limitations of LLM Feedback"** and states:

> "LLMs and VLMs often produce hallucinations due to biases and spurious features in training data, such as incorrectly associating objects with visual cues or generating factually inaccurate text. Moreover, models may generate biased or incorrect outputs due to limitations in pre-trained knowledge or insufficient understanding of the dynamics of the deployment environment. Such issues are especially noticeable in AI agents with minimal fine-tuning, leading to **unreliable feedback and reduced trust in the agent system**."

That is the concession: when the feedback source is the model, the feedback is subject to the model's failure modes, and the trust consequence is stated in the survey's own words. Its proposed future direction is to bring in *other* signals — "retrieval-augmented generation (RAG) to cross-validate outputs with external sources, or integrating external databases to improve factual accuracy"; in multi-agent systems, "iterative interaction and debate among agents can help correct individual hallucinations and reasoning errors"; and "incorporating world models (environmental models) can also support systematic fact-checking of generated content."

Read §6.6 next to §8.1–§8.5 and the survey's own remedy is the counter-evidence's prescription: **get a signal from outside the model.** RAG cross-validation is external feedback (§5). External databases are external feedback. World models are external feedback — with the caveat of §5.3 that they are a learned model with their own error mode. Only the "multi-agent debate" item needs the §6.4 asterisk: debate decorrelates only if something in it is independent. So the survey's §6.6 and the counter-evidence agree on the mechanism; they differ on how much of the field's optimistic prose survives the concession. ⚠ Cross-reference the repo's owners of that direction: [agentic_search_vs_rag_guide.md](agentic_search_vs_rag_guide.md) (RAG cross-validation), [ai_verify_toolkit_guide.md](ai_verify_toolkit_guide.md) (verification harnesses), [llm_guard_models_guide.md](llm_guard_models_guide.md) (independent trained checkers).

### 8.7 The state of the evidence — no resolution

Do not read §8 as "loops don't work". The literature does not say that, and it would be dishonest to report it that way. Report the tension:

- **Self-Refine's own claim** (Madaan et al., verified at the arXiv record): across 7 diverse tasks, "outputs generated with Self-Refine are preferred by humans and automatic metrics over those generated with the same LLM using conventional one-step generation, improving by **~20% absolute on average** in task performance", with a single LLM as generator, feedback provider and refiner, and no training. That is a strong positive result for an intrinsic loop.
- **The critical literature's claim**: those gains are (a) task-conditional — Kamoi et al. find the successes concentrated in "tasks that are exceptionally suited for self-correction"; (b) evaluation-sensitive — Kamoi et al. identify unfair evaluations that over-evaluate self-correction; (c) cost-sensitive — Olausson et al. find gains "often modest … sometimes not present at all" once repair cost is counted; and (d) in reasoning specifically, absent or negative — Huang et al. find degradation without external feedback.

The reconciliation the evidence supports, stated as the state of the field rather than as a verdict: **tasks where the failure is locally checkable from the output itself** (format/constraint compliance, translation fluency, style, code that a test can exercise) behave like Self-Refine's tasks, because the model *does* have enough information to judge. **Tasks where correctness depends on facts or multi-step reasoning the model cannot verify internally** behave like Huang et al.'s and Stechly et al.'s tasks. The loop's utility tracks the checkability of the error class. ⚠ This synthesis is this guide's reading of the cited primaries, not a finding any one paper states in these terms.

### 8.8 The synthesis: when a loop helps, when it is theatre

Every row below carries its own verified source. This table is the operating core of the guide.

| Condition | Does the loop help? | Evidence |
|---|---|---|
| The error is **decidable from the artefact** (schema, type, test, balance, constraint) | Yes, and cheaply | Stechly et al. 2024 (arXiv:2402.08115): "significant performance gains with sound external verification"; Kamoi et al. TACL 2024 finding (2) |
| An **external sound verifier** is available and used | Yes; most of the gain is the verifier, not the scaffolding | Stechly et al. 2024: "merely re-prompting with a sound verifier maintains most of the benefits of more involved setups" |
| The **feedback quality is raised** (stronger critic, human feedback) | Yes — larger gains | Olausson et al. ICLR 2024: "using a stronger model to artificially boost the quality of the feedback, we observe substantially larger performance gains" |
| The task is **exceptionally suited to self-correction** | Yes | Kamoi et al. TACL 2024 finding (1)'s stated exception |
| **Large-scale fine-tuning** installs the capability | Yes | Kamoi et al. TACL 2024 finding (3) |
| Error class requires **reasoning the model cannot verify internally**; no external signal | **No — and sometimes worse** | Huang et al. ICLR 2024: "struggle to self-correct … at times, their performance even degrades after self-correction" |
| **Model judges its own plan**, no sound verifier | **No — degrades** | Valmeekam et al. 2023 (arXiv:2310.08118): "self-critiquing appears to diminish plan generation performance" |
| The model is used as the **verifier** of the model | **Unreliable — false positives** | Valmeekam et al. 2023: "the LLM verifiers … produce a notable number of false positives, compromising the system's reliability" |
| The **loop's cost is not counted** | Looks like a gain; may not be one | Olausson et al. ICLR 2024: "when the cost of carrying out repair is taken into account, performance gains are often modest … sometimes not present at all" |
| Feedback is made **richer/more detailed** | **Minimal effect** — do not expect gain here | Valmeekam et al. 2023: "the nature of feedback, whether binary or detailed, showed minimal impact on plan generation" |
| The judge and the judged are the **same model** | Weak signal; self-enhancement bias | Zheng et al. NeurIPS 2023 D&B (abstract): "position, verbosity, and self-enhancement biases, as well as limited reasoning ability" |
| The signal is **the model's opinion with nothing behind it** | **Theatre** — unless one of the rows above applies | The conjunction of Huang et al. 2024, Valmeekam et al. 2023, Stechly et al. 2024, Kamoi et al. TACL 2024 |

**The honest headline, in one sentence:** *a loop is only as good as the signal it closes on — and a model grading itself is a weak signal.*

⚠ **One caveat about coverage.** The "gains come from the stopping rule" half of the folk explanation is **partially** supported here: Stechly et al.'s ablation shows the gain survives when only the verifier's signal is retained and the critique machinery is removed, which is the same phenomenon a stopping-rule explanation predicts. This guide did **not** verify a dedicated primary-source study isolating stopping rules as the source of self-correction gains, and it does not assert one. Recorded as a limitation in §16.

---

## 9. The Survey's Comparison and What It Does Not Show

### 9.1 The caveat comes first

The survey's §5 selects representative mechanisms and compares them on HotpotQA, ALFWorld and WebShop — with an admission that changes how every number in the table must be read. The survey's own sentence, verbatim:

> "This section selects several representative feedback mechanisms for comparative experiments on HotpotQA [Yang et al., 2018], ALFWorld [Shridhar et al., 2020], and WebShop [Yao et al., 2022] (**some experimental results are borrowed from ExpeL [Zhao et al., 2024] and AutoGuide [Fu et al., 2024]**), as shown in Table 3."

**Table 3 is partly second-hand.** Some rows are the survey authors' own runs; some are lifted from the ExpeL and AutoGuide papers, which ran their own baselines under their own setups, with their own model versions, prompts, and evaluation harnesses. The parenthesis does not say *which* rows are borrowed. Therefore:

- The table is **not** a like-for-like benchmark comparison the survey performed and it must not be presented as one.
- Cross-row comparisons carry an unknown amount of harness variance — a 5-point gap between two rows may be a mechanism difference or a harness difference, and the survey gives no way to tell.
- The numbers are best read as **one team's summary of what the literature reports on three benchmarks**, with the survey's own reading applied on top.

⚠ This is normal for surveys and it is honest of the authors to say so. The problem is downstream: a table titled "Comparison of different feedback mechanisms" with a percentage under every method reads as a measurement, and the sentence that qualifies it is in the prose above it. Cite the caveat whenever you cite the table.

### 9.2 Table 3, reproduced, with the caveat attached

Attributed to the survey (Table 3, page 10588). Where a cell spans several rows in the original, the shared value is shown once with a note; "—" marks an empty cell in the original.

| Method | HotpotQA | ALFWorld | WebShop | Feedback Round | Computational Cost | Stability | Multi-agent task transferability | Real-time |
|---|---|---|---|---|---|---|---|---|
| Act | 29% | 28% | 34% | Single-round | Low | Low stability | Transferable | ✓ |
| CoT | 29% | — | — | Multi-round | Medium | Low stability | Transferable | ✓ |
| ReAct | 28% | 40% | 35% | Multi-round | Medium | Low stability | Transferable | ✓ |
| Reflexion-R1 | 33% | 48% | 43% | One-round with experiment replay | High | Low stability in short-term, High stability in long-term | Require shared experience pool | ✗ |
| Reflexion-R2 | 40% | 52% | 46% | Two-round with experiment replay | *(shared)* | *(shared)* | *(shared)* | ✗ |
| Reflexion-R3 | 40% | 54% | 48% | Three-round with experiment replay | *(shared)* | *(shared)* | *(shared)* | ✗ |
| ExpeL | 39% | 59% | 41% | Multi-round with experiment replay | High | Medium stability | Require shared experience pool | ✗ |
| AdaPlanner | — | 63% | — | Multi-round | Medium | High stability | Transferable | ✓ |
| AutoGuide | — | 79% | 46% | Multi-round with experiment replay | High | High stability | Require shared experience pool | ✗ |

### 9.3 The saturation the survey does not remark on

Three things jump out of the numbers themselves, and the survey's §5 prose does not mention any of them.

**(a) Reflexion saturates on HotpotQA: R1 = 33%, R2 = 40%, R3 = 40%.** The third round adds **zero** percentage points on HotpotQA, while still adding 2 points on ALFWorld (52→54) and 2 on WebShop (46→48). So the survey's headline "Reflexion further improves ALFWorld's performance … with the most notable improvements observed between the first and second trials" is correct for ALFWorld and **false as a general statement about the mechanism**: on the reasoning benchmark, the mechanism is done improving after round two. ⚠ This is not proof of useless iteration — the second round bought 7 points, so rounds do work — but it is direct evidence of **diminishing and then zero returns**, and every additional round is paid for in full. The table's own "Computational Cost: High" row for Reflexion is the bill for exactly those rounds.

**(b) On HotpotQA, ReAct (28%) is worse than Act (29%) and worse than CoT (29%).** The survey's prose half-acknowledges this — "ReAct demonstrates limited performance improvement on complex reasoning tasks such as HotpotQA and StrategyQA" — but the table is stronger than the prose: the interleaved reasoning-and-acting agent is *below* the simple action baseline and *level with* plain CoT on that benchmark. Reading across, the survey's own table supports the view that the reason-act interleaving helps in sequential decision-making tasks (ReAct 40% ALFWorld vs Act 28%) and does not help in multi-hop reasoning. Which is a mechanism result, not a benchmark artefact — and it is exactly what §8's literature predicts about promoting a reasoning trace into a self-checking loop.

**(c) The "best" method is not best everywhere.** AutoGuide is the survey's high-stability, high-cost winner at 79% on ALFWorld — and on WebShop its 46% is *below* Reflexion-R3's 48%, at the highest cost column in the table. The survey's §5 stability narrative ("Reflexion, ExpeL, and AutoGuide exhibit greater stability in long-term dynamic tasks through iterative learning mechanisms") treats the three as a class; the numbers do not rank them consistently across benchmarks. ⚠ Anyone selecting a mechanism on the basis of this table should select on their own task distribution, not on ALFWorld.

### 9.4 What the columns do and do not measure

The columns are the most useful part of Table 3 and also the least defined. What each one can and cannot tell you:

| Column | What it reportedly measures | What it does not measure |
|---|---|---|
| **HotpotQA / ALFWorld / WebShop** (success rate) | End-to-end task success on three benchmarks, mixed harnesses | Whether the loop caused the delta; whether the base model alone (with the same compute budget) would match it; error bars, or one seed vs many |
| **Feedback Round** | The *shape* of the iteration — single-round, multi-round, "with experiment replay" | The number of model calls per task, the token cost, or the latency. "One-round with experiment replay" is not one round of compute. |
| **Computational Cost** | A three-level judgement (Low / Medium / High) | Any actual cost. No tokens, no GPU-seconds, no money. Two "High" methods may differ by an order of magnitude. |
| **Stability** | A qualitative judgement; for Reflexion, explicitly split "Low stability in short-term, High stability in long-term" | The metric. No variance measure, no seed count, no drift measurement. §8's problem — that a loop can make things *worse* — is not visible in a stability column that has no downward direction. |
| **Multi-agent task transferability** | "Transferable" vs "Require shared experience pool" | These are not points on one axis. The first is a property of the mechanism; the second is an architectural requirement. A reader cannot rank them. |
| **Real-time** | ✓/✗ suitability for real-time use | The latency budget assumed. A ✗ is doing a lot of implicit work about how much time a task permits. |

**The missing column is the one that matters most.** Table 3 has no column for *signal trustworthiness*, no column for *whether the feedback can be wrong*, and no column for *what happens when the feedback is wrong*. Compare it against a compiler-feedback row: the signal is decidable, so the loop is bounded; a self-reflection row costs more and its signal is the model's opinion. The table cannot express that difference, because the taxonomy it comes from cannot either. That is the gap this guide exists to name.

### 9.5 The four advantages, reported as the survey's claim

The survey's §5 closes with a four-part claim that feedback mechanisms like Reflexion have "significant advantages over traditional reinforcement learning", and the claim is worth reproducing because it is a claim — from the survey, not from a benchmark:

1. **Trial-and-error learning** — "improves performance by reflecting on errors and incorporating experience into subsequent decisions".
2. **Efficiency** — "it does not require fine-tuning language models, offering higher data and computational efficiency".
3. **Detailed feedback** — "it uses linguistic feedback, which is more specific and detailed than the scalar rewards of traditional RL".
4. **Interpretability and memory** — "explicit, interpretable staged memory, storing self-reflections to facilitate analysis of the learning process, outperforming traditional black-box RL".

Points 1 and 3 are the survey's reading, and both are in tension with §8: point 1 is the claim that reflection improves subsequent decisions, which Huang et al. and Valmeekam et al. qualify sharply; point 3 asserts linguistic feedback is *more* detailed and therefore better, while Valmeekam et al. found "the nature of feedback, whether binary or detailed, showed minimal impact on plan generation". Point 2 is the strongest of the four and mostly holds — not updating weights is genuinely cheaper than fine-tuning, which is precisely why these mechanisms are popular. Point 4 is plausible but unmeasured: "interpretable staged memory" is an architectural claim, and interpretability buys auditability (§12) without buying correctness. ⚠ Report point 3 as contested.

---

## 10. Evaluation and Benchmarks

### 10.1 Outcome-based vs process-based, as the survey frames them

The survey's §4.1 splits evaluation in two, and states its own honest problem in the first sentence: "Although LLM-based AI agents have demonstrated remarkable capabilities in downstream tasks, the question of how to effectively evaluate feedback mechanisms remains unresolved."

- **Outcome-based evaluation** — "most evaluations are Outcome-based evaluation, relying on the assessment of feedback mechanisms through end-to-end agent task performance … based on task success rates." Survey examples: **Olausson et al. [2023]** on APPS, where "the feedback mechanism is evaluated by the pass rate of programming tasks"; **TravelPlanner** measuring "tool usage error rates and planning failure rates". The survey's own critique: "these indirect evaluations often oversimplify the assessment process by focusing on aggregate outcomes rather than granular performance details."
- **Process-based evaluation** — "Traditional agent evaluation methods tend to focus solely on final outcomes, overlooking critical details during execution or relying heavily on manual assessment." Survey examples: **DevAI/Zhuge et al. [2024]**, where "agents … evaluate each other, assessing both the final outcomes and intermediate execution steps for richer feedback"; **AMOR** [Guan et al., 2024], which "solves problems through autonomous executions and transitions over disentangled modules, allowing to provide feedback to the individual modules, and thus naturally forms process supervision."

The survey's discussion is the sharpest methodological passage in the paper:

> "Gaps remain in developing robust quantitative indicators for self-correction across diverse domains. Factors like learning stage, task complexity, and environmental diversity can significantly impact feedback effectiveness, highlighting the need for a more comprehensive evaluation system."

⚠ **The distinction has a fatal hole and the survey does not name it.** Outcome-based evaluation **cannot attribute** the outcome. If a Reflexion agent scores 40% on HotpotQA and the Act baseline scores 29%, the difference is not evidence that the *feedback* worked — it could be the extra compute, the extra attempts, the reflection text acting as a random perturbation, or a harness difference. This is the exact accounting error Olausson et al. identified and quantified (§8.4): once the cost of repair is counted, gains are "often modest … sometimes not present at all". **Outcome-based evaluation of a feedback mechanism is measuring the mechanism plus everything else in the loop plus the harness,** and the survey's own reported problem — no robust quantitative indicators — is a symptom of that, not an oversight in the field's tooling.

Process-based evaluation does not fix it either: scoring intermediate steps requires a ground truth for intermediate steps, which usually means a judge (§8, §7.4) or a hand-built rubric — reintroducing the weak-signal problem at a finer granularity. ⚠ Cross-reference: [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) and [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md) own eval design; the point here is that a loop needs an **attribution design** (base model + equal compute budget + the loop) that neither framework provides out of the box.

### 10.2 The benchmark landscape, by the survey's domains

The survey's §4.2 maps benchmarks across eleven domain families and names a weakness for each. The pattern in the right-hand column is worth seeing in one place, because it is the survey's most consistently critical writing:

| Domain (survey's) | Benchmarks named | Survey's named weakness |
|---|---|---|
| Reasoning | HotPotQA, ScienceQA, FEVER, StrategyQA | Option bias in multiple-choice sets ("favoring option C"); "this Q&A format overlooks the reasoning process, where an incorrect process might still yield a correct result" |
| Virtual world | ALFWorld, IGLU, Minecraft | "ALFWorld's grid-world simplicity may fail to capture real-world ambiguity"; Minecraft's "sim-to-real" transfer problem |
| Embodied action | Franka-Kitchen, Meta-World, RT-X | Franka-Kitchen's kitchen focus "limit[s] generalization to other physical scenarios" |
| Web navigation | WebShop, WebArena, Mind2Web, WebVoyager | "Mind2Web interacts only with static website states"; WebArena uses simulated sites; WebVoyager uses Selenium against real pages |
| Code generation | DevAI, MBPP, HumanEval, SWE-Bench | "these rely heavily on success rates, which fail to provide specific feedback on each stage or capture the dynamic performance of agent systems" |
| Social simulation | SocialBench, SocKET | "overly idealized social scenarios, lacking the complexity and unpredictability of real-world social interactions" |
| Tool usage | ToolBench, TravelPlanner, ToolEyes | "overly focus on specific tools, limiting flexibility in tool selection and combination" |
| Multi-agent collaboration | RoCo/RocoBench, PARTNR, VillagerBench | VillagerBench relies on Minecraft; **RoCo "assumes perfect perception (e.g., object detection, pose estimation, collision checking), which may fail in real-world scenarios like industrial production or medical collaboration"** |
| Machine translation | WMT, FLORES-200 | "FLORES-200 covers multiple languages but focuses on document translation, falling short in general translation" |
| Financial | FiQA_SA, FinQA | "these tasks limit the ability to test agents in dynamic market scenarios like stock trading" |
| Multidimensional | AgentBench | — (the survey treats it as the general-capability family; eight task scenarios listed) |

Note how many of those weaknesses are the same weakness in different clothes: **the benchmark removes the environment's inconvenient properties** — perception noise, ambiguity, dynamism, interaction with the world's actual state. Which is precisely the environment that produces the *trustworthy* signals of §5.3. There is a real irony worth stating: the benchmarks that are easiest to construct are the ones whose feedback is least informative about the properties that make real feedback hard.

### 10.3 The benchmark sizes: a spot-check against the primaries

The survey's Table 2 gives a size for every benchmark. These are the kind of numbers that get copied across papers without checking, so a sample was verified at the primary source. ✅ verified at the primary; ⚠ approximate, qualified, or attributed differently; ❌ not verified here.

| Benchmark | Survey's Table 2 size | What the primary source says | Verdict |
|---|---|---|---|
| HotPotQA [Yang et al., 2018] | 113k Q&A pairs | "**113k** Wikipedia-based question-answer pairs" (EMNLP 2018) | ✅ |
| FEVER [Thorne et al., 2018] | 185k claims | "It consists of **185,445 claims**" (NAACL 2018) | ✅ (survey rounds down) |
| ScienceQA [Lu et al., 2022] | 21k Q&A pairs | "**~21k multimodal multiple choice questions**" (NeurIPS 2022) | ✅ size; note it is **multiple choice**, which is the survey's own later criticism |
| StrategyQA [Geva et al., 2021] | 2.7k Q&A pairs | "StrategyQA includes **2,780 examples**" (TACL 2021) | ✅ size; ⚠ survey's reference says "In ACL, 2021" — the primary says TACL 2021 |
| Mind2Web [Deng et al., 2024] | 2k tasks | "over **2,000** open-ended tasks collected from **137 websites** spanning **31 domains**" | ✅; ⚠ survey's reference says "In NeurIPS, 2024" — the primary's own record says **NeurIPS'23 Spotlight** |
| SWE-Bench [Jimenez et al., 2024] | 2k problems | "**2,294** software engineering problems … across **12** popular Python repositories" (ICLR 2024) | ✅ |
| MBPP [Austin et al., 2021] | 1k problems | "The Mostly Basic Programming Problems (MBPP) dataset contains **974** programming tasks" | ✅ approximately (974, not 1k) |
| ALFWorld [Shridhar et al., 2020] | 3.1k examples | Not stated in the primary abstract; ⚠ survey's reference says "In ICLR, 2020" — the primary's own record says **ICLR 2021** | ⚠ size ❌ / venue ❌ |
| WebArena [Zhou et al., 2023] | 812 tasks | Abstract confirms four domains and reports human 78.24% vs best GPT-4 agent 14.41%; **the abstract does not state 812** | ⚠ size not confirmed at the abstract |
| HumanEval [Chen et al., 2021a] | 164 problems | Not retrieved in this run | ❌ |
| AgentBench [Liu et al., 2024] | 1.4k samples | Abstract confirms "**8 distinct environments**" (ICLR 2024 ✅); sample count not stated | ❌ size |
| ToolBench [Qin et al., 2024] | 16k API, 3k tools | ToolLLM's ToolBench: "we collect **16,464 real-world RESTful APIs** spanning **49 categories** from RapidAPI Hub" | ✅ 16k APIs; ❌ "3k tools"; ⚠ see §16 on the citation mismatch |
| FinQA [Chen et al., 2021b] | 2.8k reports, 8k Q&A pairs | Sizes not stated in the abstract; primary record says **EMNLP 2021**, survey's reference says "arXiv, 2021" | ❌ sizes / ⚠ venue |
| FLORES-200 [Guzmán et al., 2019] | 842 articles, 3k sentences | The cited primary is the FLORES **Nepali–English and Sinhala–English** datasets paper | ⚠ **attribution problem** — see §16 |
| RT-X [Padalkar et al., 2023] | 1m trajectories | Not verified in this run ("Open X-Embodiment" is the relevant primary) | ❌ |

Patterns worth noting in the verified set: the survey's sizes are **broadly reliable** where they could be checked — three exact matches and two defensible roundings. What is *not* reliable is the **venue/year column of the reference list**: three of the ten reference entries checked carried a venue or year that the primary source contradicts (Mind2Web NeurIPS'23 vs the survey's NeurIPS 2024; ALFWorld ICLR 2021 vs the survey's ICLR 2020; StrategyQA TACL vs the survey's ACL; and ChatEval's ICLR 2023 vs an arXiv posting of August 2023, which predates the conference). ⚠ None of these damages the taxonomy; all of them damage anyone citing the survey's reference list as a bibliography without checking.

---

## 11. The Six Challenges and Future Directions

The survey's §6 is six challenges with a paired future direction each. Below, each is summarised in one sentence for the challenge and one for the direction — then mapped to the repo guide that owns the adjacent material. **The mapping is the repository's value-add on top of the survey**: the survey names the research direction, and these guides cover the engineering.

| § | Challenge (one sentence) | Survey's future direction (one sentence) | Repo guide that owns the adjacent material |
|---|---|---|---|
| **6.1** Real-time feedback and multi-agent system implementation | More agents mean sharply rising compute, slower information flow and harder coordination, compounded by the absence of standard protocols between vendors | Build efficient, scalable frameworks (e.g. MARL-based, citing Ma et al. 2024 in *Nature*) and **promote protocol standardisation — MCP, A2A, ANP, Agora** | [mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md), [mcp_discovery_guide.md](mcp_discovery_guide.md), [mcp_framework_tools_guide.md](mcp_framework_tools_guide.md); orchestration at [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md) and [hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md) |
| **6.2** Integration of multi-modal feedback | Aligning text, image, audio and sensor signals is hard because each modality has its own representation, so fusion strategies are not established | Develop unified multimodal feedback frameworks — the survey cites **Tactical Rewind** (self-correction by backtracking in vision-and-language navigation) and **MM-React** (multimodal reasoning via a pool of visual experts) — and pursue cross-modal representation learning | [vision_language_models.md](vision_language_models.md); embodied/sensor feedback at [../physical_ai_guide.md](../physical_ai_guide.md) |
| **6.3** Meta-learning adaptive feedback | Balancing learning from multiple feedback sources while keeping behaviour stable and consistent is unsolved; the survey cites **AutoGen** for the observation that agents must keep adapting to evolving needs, which "can result in **feedback conflicts or overfitting**" | Develop more robust adaptive mechanisms; the survey names **mutual learning** (a two-stage framework, Wang et al. 2024a) and learning unified latent representations to integrate multi-source feedback and minimise conflicts | [ai_agent_drift_guide.md](ai_agent_drift_guide.md) (temporal behaviour change); [closed_loop_data_engineering_guide.md](closed_loop_data_engineering_guide.md); [agent_versioning_guide.md](agent_versioning_guide.md) |
| **6.4** Explainability of feedback mechanisms | Mechanisms lack transparency in how they make and adjust decisions, which "undermines user trust — especially in critical areas like healthcare and transportation" | Build explainable feedback models with visualisation tools and transparency metrics; the survey cites natural-language explanation tools (Feng et al. 2024a) and **AMOR** [Guan et al., 2024], which "improves explainability and safety by providing step-by-step feedback similar to a chain of thought" | [ai_governance_framework_guide.md](ai_governance_framework_guide.md), [implementing-responsible-ai.md](implementing-responsible-ai.md), [agentops_guide.md](agentops_guide.md) (traces as evidence) |
| **6.5** Feedback in embedded AI agents | Models trained in simulation hit "sim-to-real" gaps — disturbances, lighting, gravity and other physical properties are not replicated, so embodied agents underperform in reality | Adaptive learning that adjusts online or via meta-learning; hybrid simulated + real training; high-fidelity simulators integrating vision, touch and mechanics. The survey lists domain randomisation, domain adaptation and simulation improvement as current remedies | [../physical_ai_guide.md](../physical_ai_guide.md) |
| **6.6** Inherent limitations of LLM feedback | LLMs and VLMs hallucinate from biases and spurious training-data features, and pre-trained-knowledge limits produce biased or incorrect outputs — "leading to **unreliable feedback and reduced trust in the agent system**" | RAG cross-validation against external sources, external databases, multi-agent debate to correct individual hallucinations, and world models for systematic fact-checking | [agentic_search_vs_rag_guide.md](agentic_search_vs_rag_guide.md), [ai_verify_toolkit_guide.md](ai_verify_toolkit_guide.md), [llm_guard_models_guide.md](llm_guard_models_guide.md) |

Three observations on the set.

**(a) §6.1 is the only one whose direction is an infrastructure standard, and it is the one with the most immediate practitioner payoff.** Protocol standardisation is not a research question; it is a procurement and integration question, and it is what makes an independently-produced feedback signal reachable from an agent you did not build. It is also the item where the survey's own examples (MCP, A2A, ANP, Agora) have moved fastest since submission.

**(b) §6.3's cited failure — "feedback conflicts or overfitting" — is the inter-task feedback problem of §4.3 with a name.** Once a loop writes to memory, multiple feedback sources can disagree, and a system that follows whichever arrived last is not learning, it is oscillating. The survey names the failure and stops at latent representations; the operational question is which source wins, on what evidence, and how you would know the answer changed. [ai_agent_drift_guide.md](ai_agent_drift_guide.md) owns that.

**(c) §6.4 (explainability) and §6.6 (unreliable feedback) are the two that a regulated environment cares about most, and the survey treats them as research gaps rather than as controls.** "Explainable feedback with visualisation tools and transparency metrics" is, in a regulated setting, a *supervisory* requirement: how do you demonstrate to someone outside the team what the loop did and why? §12 takes that up directly.

---

## 12. The Banking and Regulated Angle

> This section is the guide's argument about regulated deployment. It names no real bank; **Cymbal Bank** is the only institution persona in this repo and §13 uses it for the worked example. Real systems, papers and protocols may be named factually, including the survey's own citations.

### 12.1 A feedback loop in which the model evaluates its own work is not a control

State the distinction plainly, because it is the most expensive category error in this space:

| | **Quality mechanism** (what an agent's feedback loop is) | **Control** (what an assurance framework asks for) |
|---|---|---|
| Purpose | Make the output better | Prevent or detect an unacceptable outcome |
| Independence | Not required | **Required** — the checker must not share the failure mode of the thing checked, and must not report to it |
| Fails how | Silently — the output is worse but plausible | Visibly — a control that fails is a finding, with an owner |
| Accountability | Diffuse: "the agent's self-critique step" | Named: a person or function owns the control and its failure |
| Evidence | A trace, if you logged one | A defined artefact, retained, reconstructable, testable |
| Assurance language | "We improved quality by 12%" | "This control operated in N of N cases; here is the sample; here is the exception log" |

A self-graded loop cannot be a control, because **independence is the defining property of a control and the model's judgement of its own output is the definition of non-independence.** Huang et al. (ICLR 2024) found the model's own correction can degrade reasoning; Valmeekam et al. found self-critique diminishing plan quality and LLM verifiers producing false positives; Zheng et al.'s primary documents self-enhancement bias in model judges. Take any of those three and the conclusion is the same: making a control out of the model's opinion of itself is making a control out of a signal whose error is correlated with the error you are trying to catch.

And this is exactly how a self-graded system ends up *described* as governed. The sentence forms are recognisable: "the agent validates its own output before submission"; "a reviewer agent checks the recommender agent's work"; "reflection catches errors the base model makes". Each describes a quality mechanism in the vocabulary of assurance. Conflating them is how a system with no independent check acquires the appearance of one — and the appearance is what gets signed off.

**The test to apply, in one question:** *if the agent's output is wrong in exactly the way the model is prone to be wrong, will this mechanism catch it?* If the mechanism is the model grading itself, the honest answer is no — by construction, the shared failure mode is invisible to the shared judgement. Only an independent signal can catch it, and §5.3's ladder is the way to find one.

### 12.2 Human-in-the-loop as a control, not a feature

The survey names the impracticality in its own words (§7.3): human feedback "requires continuous monitoring, evaluation, and adjustment by experts", which is "often impractical due to limited resources or real-time constraints."

Read that as a constraint on **control design**, not as an argument against humans. A human check is a real control only when all of the following hold:

- **It is scoped.** The gate covers defined decision types at defined risk levels, not "all agent output".
- **It has authority.** The reviewer can reject, and rejection is the default when in doubt — not a comment box that the loop ignores.
- **It has a rate.** The volume is bounded such that the reviewer can actually read what they approve. An approval gate that fires on 100% of cases is a queue; queues get rubber-stamped, and a rubber-stamped control is worse than no control because it manufactures a record of scrutiny that did not happen.
- **It is instrumented.** Approval and edit rates, time-per-review, and override outcomes are logged. A gate whose override rate is ~0% is either unnecessary or not being read.
- **It is owned.** A named function is accountable for the gate's failure, not the loop's designer.

⚠ The survey's material on **ReHAC** is directly relevant here and underused: "a policy model designed to determine the most opportune stages for human intervention within the task-solving process." Learning *when a human is needed* is the difference between a control and a tax on the reviewer's attention. ⚠ Note the source: that description is the survey's characterisation of ReHAC; the primary's own framing was not verified in this run.

### 12.3 Preference and corrective data are a data-governance and model-risk question

The survey's §7.2 collapse (from this guide's §7.2) becomes a governance problem the moment a bank is involved, because the two halves of "human feedback" attract different obligations.

**Preference feedback is a model-risk and data-governance matter.** Once human preferences shape weights, you have: a dataset of human judgements with provenance, consent and quality questions; a training procedure that may be external; a model whose behaviour changed in ways that require re-evaluation; and a documented tendency to amplify rater bias, which the survey itself names in its ethics passage ("particularly along lines of gender, race, and culture"). The governance questions are the model-risk questions: who were the raters, what were they asked to prefer, how was preference quality assessed, what evaluation gates the resulting model passed, how do you roll it back. The operational owner is [closed_loop_data_engineering_guide.md](closed_loop_data_engineering_guide.md) with [llm_instruction_tuning_guide.md](llm_instruction_tuning_guide.md); the assurance owner is [ai_governance_framework_guide.md](ai_governance_framework_guide.md) and [ai_governance_bias_redteaming_guide.md](ai_governance_bias_redteaming_guide.md). **A feedback loop that quietly becomes a training pipeline has crossed a boundary that needs an approval, not a changelog entry.**

**Corrective feedback is an operations and accountability matter.** A runtime human correction is a decision about a specific customer or transaction. It needs the same evidential treatment as any other decision: who corrected what, when, on what information, and what the system did next. In a regulated setting the correction is not only a training signal — it is a **record**, and if it only exists as a message in a log that nobody reads, the loop has consumed a human decision without preserving it.

### 12.4 The explainability gap is a supervisory problem

The survey's §6.4 frames explainability as a trust and research problem: mechanisms "lack transparency in how they make and adjust decisions, which undermines user trust — especially in critical areas like healthcare and transportation", and the direction is explainable feedback models with visualisation tools and transparency metrics.

In a regulated institution the same gap is a **supervisory** problem with a different question: *can you explain to an independent reviewer how this decision was reached, using evidence you retained?* That is not the same as "is the model interpretable". A loop makes it harder than a single model call, for three reasons: the decision path now includes intermediate judgements the model made about itself; those judgements are natural language that may or may not be faithful to the actual basis of the output; and each iteration can change the answer without leaving a marker of what changed and why. A trace that shows five reflections and a final answer also shows *which reflection mattered* only if you recorded it.

Three practical requirements follow, none of which the survey states:

1. **Log the loop's decision points, not just its outputs** — each critique, each accepted/rejected revision, each round's result, with timestamps and model versions. This is the [agentops_guide.md](agentops_guide.md) layer applied to loops, and [agent_versioning_guide.md](agent_versioning_guide.md) covers the version dimension (a loop's behaviour is a function of model version + prompt + memory state; all three must be pinned).
2. **Distinguish "the loop improved it" from "the loop changed it"** in the evidence — a human-readable diff of the substantive change, not just a similarity score.
3. **Make the loop's *silence* auditable.** An iteration that found nothing wrong and one that was skipped are indistinguishable in most traces, and the difference matters when reconstructing a decision after the fact.

### 12.5 The auditability question: what evidence does a loop leave?

Ask of any proposed loop: **if you had to reconstruct a decision six months later, what would you have?** Below is what the four feedback categories typically leave, and where the gaps are. This table is this guide's synthesis; the categories are the survey's.

| Category | Artefact typically left | What an auditor can reconstruct | The gap |
|---|---|---|---|
| **Internal** (self-critique, reflection) | Critique text, revision, final output | That a revision happened and what it said | Whether it improved anything; whether the reflection was faithful to the real reason; the base model's answer before the loop |
| **External — deterministic** (compiler, test, API, game) | Exit code, error message, test result | The exact verifiable failure, reproducibly | Usually nothing material — this is the best-documented loop |
| **External — non-deterministic** (search, retrieval, world model) | Retrieved text, ranked results, model prediction | What the agent saw | Whether the retrieval was adequate; the ranking at the time; how the world model was validated |
| **Multi-agent** (debate, role review) | Transcript of the exchange | That positions were argued and a conclusion reached | Whether the agreement carried information (per §6.4 — number of independent channels, not agents) |
| **Human** | Approval/override records, edits, chat | The human's decision and often its rationale | Whether the human had enough context to decide; whether they were reading |

**Which loops in a bank are worth building at all.** The honest answer follows from §5.3 and §8.8 rather than from enthusiasm:

| Loop | Worth it? | Why |
|---|---|---|
| Close on a **deterministic, verifiable** signal (schema validation, ledger balance, reconciliation, policy rule engine, test suite) | **Yes** — build it | The signal cannot be argued with; the loop is bounded and cheap; auditability is nearly free |
| Close on an **external non-deterministic** signal where a retrieval/vendor error is tolerable (drafting, summarisation with citations) | Yes, with an explicit check on the *source* | The loop helps if the retrieved evidence is real; add citation-grounding checks ([ai_verify_toolkit_guide.md](ai_verify_toolkit_guide.md)) |
| Close on a **human control** for a bounded, high-risk decision class | Yes, if the four conditions of §12.2 hold | The control is genuine; the cost is the scarce resource, so scope it |
| Close on **the model's own judgement of its own reasoning** | **No — do not deploy it as a control or as a claimed improvement** | §8; and describing it as a control is the failure mode of §12.1 |
| Close on **same-model debate** as evidence of correctness | No | §6.4 — same-model instances are not independent; report it as a committee, not a check |
| Close on a **score from a model trained on your own outcomes** (a learned reward/quality model) | Sometimes — but as a *monitoring* signal | It is a learned model with its own error mode; validate it like a model, not like a rule ([llm_guard_models_guide.md](llm_guard_models_guide.md)) |

**The one-line position:** build loops on signals that can be decided without asking the model whether it agrees; use human review as a scoped, instrumented control where the stakes justify the scarcity; and never book a model grading itself as a control.

---

## 13. The Cymbal Bank Worked Example

> **Explicitly illustrative and fictional.** Cymbal Bank is this repo's worked-example institution. Every figure below is labelled **ILLUSTRATIVE** and is constructed to show the *arithmetic and the reasoning*, not to describe a real system, a real vendor, or a real volume. Real systems and papers are named factually where they are the subject matter.

### 13.1 The process

**Cymbal** runs periodic KYC reviews. An analyst opens a customer file, gathers evidence (identity documents on record, transaction-behaviour summary, adverse-media search results, source-of-funds notes), and writes a **risk narrative** that supports one of three recommendations: *clear*, *clear with monitoring*, or *escalate for enhanced due diligence*. The narrative is the artefact a second-line reviewer and, on escalation, an investigator read.

The team proposes an agent that drafts the narrative and the recommendation, with a "self-refinement loop" that critiques and revises the draft before it reaches a human. The stated goal is fewer analyst hours per review and fewer escalations that get sent back for rework.

**ILLUSTRATIVE: process shape.** 12,000 periodic reviews per year; 8% escalated today; ~90 minutes of analyst time per review; a second-line reviewer who currently reads every escalated case.

### 13.2 The candidate loops, and what each one actually closes on

Six candidate mechanisms were put on the table. The question asked of each was the one from §5.3, not "which category is this?":

| # | Candidate loop | The signal it closes on | Verifiable without a model? | Verdict |
|---|---|---|---|---|
| **L1** | **Evidence-pack completeness check** — every required item present, IDs valid format, dates in range, documents machine-readable | Presence/format of artefacts | **Yes** — deterministic validation | **Keep.** Cheap, cannot be argued with, near-free to audit |
| **L2** | **Policy rule engine** — the recommendation is checked against the bank's codified review policy (jurisdiction, risk band, thresholds, mandatory-escalation triggers) | A rule set owned by Compliance | **Yes** — a rule engine's output is decidable | **Keep, and treat it as control-shaped.** The oracle is independent of the model *and* the rules have an owner |
| **L3** | **Grounding check** — every factual assertion in the narrative must be traceable to a span in a retrieved source document, and unsupported sentences are flagged | Span-level traceability into retrieved evidence | **Mostly** — a checker can verify that a span exists and overlaps; it cannot verify that the span *supports* the claim | **Keep with a caveat.** The span-existence half is external; the support half is a judgement |
| **L4** | **Self-refinement of the narrative** — the model critiques its own draft for "quality, completeness and internal consistency" and revises, up to 3 rounds | **The model's opinion of its own narrative** | **No** | **DROP** — see §13.3 |
| **L5** | **Two-agent debate** on whether to clear or escalate, using two instances of the same model with the same retrieval | Agreement between correlated instances | **No** — one evidence channel, two opinions | **DROP as evidence.** May be retained as a drafting aid only |
| **L6** | **Scoped human gate** on *escalate* recommendations and on any adverse-media hit | A named reviewer's decision | Not applicable — this is the control | **Keep, scoped and instrumented** |

### 13.3 The loop the team drops, and why — the counter-evidence applied honestly

**L4 is dropped.** This is the decision the guide exists to support, so the reasoning is spelled out.

Cymbal's narrative agent is exactly the shape of system the §8 literature is about. The failure it must catch is a **substantive omission or misjudgement**: an adverse-media item that was retrieved but not weighed, a source-of-funds explanation that does not actually explain the pattern, a risk band misapplied. Those failures have two properties that make a self-refinement loop useless against them:

1. **The error class is not locally checkable.** The model cannot verify from its own output whether it missed an adverse-media item — the information it would need is in the documents, not in its draft. Kamoi et al. (TACL 2024) found self-correction works where reliable external feedback is available and has no demonstrated success via prompted-LLM feedback otherwise.
2. **The self-critique shares the failure mode.** If the retrieval returned an item that the model failed to weigh as adverse media, the critique pass — same model, same retrieval, same priors about what counts as adverse — is likely to make the same misjudgement. Valmeekam et al. found LLM verifiers producing "a notable number of false positives"; Zheng et al.'s primary names **self-enhancement bias** in model judges. A false positive here is a *clear* recommendation that should have been an escalation, which is the direction of error Cymbal least wants, and the loop's own output is the thing that ends the loop.

And the mechanism has a demonstrated failure mode in exactly this direction: Huang et al. (ICLR 2024) found that without external feedback, LLMs "struggle to self-correct their responses … and at times, their performance even degrades after self-correction." A degraded narrative that *reads confidently* is worse than a first draft, because it removes the hesitation that prompts an analyst to look harder.

**What the team does instead of L4.** The three functions L4 was supposed to provide are re-homed on real signals:

- "Is the pack complete and internally consistent?" → **L1**, deterministic, plus a cross-document consistency check implemented as a rule set (dates, names, amounts that must agree across documents).
- "Does the narrative follow policy?" → **L2**, the rule engine with a Compliance owner.
- "Is every claim grounded?" → **L3**, with the grounding check rebuilt as an *external* signal: for each sentence asserting a fact, require a citation to a retrieved span; if no span is found, flag the sentence for the human. The signal is then "no supporting span exists", which is decidable — not "the model thinks this sentence is fine", which is not.

⚠ **The KPI consequence, stated up front.** Cymbal will not be able to claim "the agent self-corrects its narratives" — because it does not, and the mechanism that would have carried that claim was dropped on evidence. The claim it *can* make is narrower and defensible: "the agent's draft is validated for completeness, policy compliance and citation-grounding before a human ever sees it, and every one of those checks is independent of the model that wrote the draft."

### 13.4 The evaluation design that would distinguish a real gain from a decoration

Before any loop ships, Cymbal runs the design below. Its purpose is to answer one question: **if the loop were removed and the compute spent elsewhere, would anything get worse?**

| Element | Design | Why |
|---|---|---|
| **Paired cases** | Same 400 reviews (ILLUSTRATIVE sample), same inputs, same model version, run with and without the loop | Removes case-mix as an explanation |
| **Equal compute budget control** | A third arm: base model, no loop, but given the same number of tokens/completions to spend (e.g. self-consistency over the recommendation) | Answers Olausson et al.'s objection — if a plain re-sample on the same budget matches the loop, the loop's structure bought nothing |
| **Verifier-only ablation** | A fourth arm: the base model re-prompted *only with the L1/L2/L3 signals*, no critique prose | This is Stechly et al.'s finding operationalised: "merely re-prompting with a sound verifier maintains most of the benefits of more involved setups". If this arm matches the full loop, the loop is decoration |
| **False-positive count** | For every case the loop passed, did the human reviewer or later investigation find a defect? | Valmeekam et al.'s false-positive finding — the direction of error that ends the loop |
| **Reviewer override rate** | Share of the agent's recommendations the reviewer changes | A loop that reduces analyst time but raises the override rate has moved work, not removed it |
| **Cost per case** | Model calls, verification calls, reviewer minutes, all in one number | The accounting error Olausson et al. identified |
| **Stability across rounds** | Does the loop's output change between rounds on the same input? | The Reflexion-R2/R3 saturation of §9.3: a round that changes nothing costs the same as a round that helps |

**The decision rule Cymbal adopts:** no loop ships unless the *verifier-only ablation* is meaningfully worse than the full loop. If the ablation matches it, the loop is not doing anything the signal was not already doing — and the honest response is to ship the verifier and spend the saved compute on retrieval quality.

### 13.5 The human-feedback cost, quantified

**All figures in this subsection are ILLUSTRATIVE and constructed for arithmetic, not drawn from any institution's data.** The point is the *shape* of the cost, and the fact that the survey's own §7.3 warning ("often impractical due to limited resources or real-time constraints") becomes an explicit number the moment you scope a control.

| Quantity | ILLUSTRATIVE value | Note |
|---|---|---|
| Periodic reviews per year | 12,000 | — |
| Current escalation rate | 8% → **960** cases/year | Escalated cases today |
| Reviewer time per escalated case | 12 minutes | Read, decide, record |
| **Human control cost as scoped** | **192 hours/year ≈ 0.12 FTE** | Bounded and affordable — this is the L6 design |
| **Human control cost if every case is gated** | **2,400 hours/year ≈ 1.5 FTE** | The same process, "human in the loop" without scoping — 12× the cost |
| **Cost of over-escalation** | 8% → 14% (a plausible effect of a loop that produces confidently-worded but under-analysed narratives) = **+720 cases = +144 hours ≈ 0.09 FTE** | A quality loop that inflates escalation volume pays for itself in reviewer hours |
| **Cost of L1–L3 verification** | ~4 extra calls per case ≈ 48,000 calls/year ≈ **$144/year** ILLUSTRATIVE at $0.003/call | The deterministic checks are effectively free |
| **Cost of L4 self-refinement if it had been kept** | 2 rounds × 12,000 ≈ 24,000 calls ≈ **$480/year** ILLUSTRATIVE | The money is *not* the problem. The unbounded cost is the risk — see below |

Two conclusions follow from the arithmetic, and the second is the one that matters:

1. **Verification is cheap; human judgement is the scarce resource.** L1–L3 cost roughly a rounding error against 0.12 FTE of reviewer attention. This is why "close the loop on a verifier" is almost always the right first move: the constraint is attention, not compute.
2. **The self-refinement loop's cost was never the compute — it was the false confidence.** At ILLUSTRATIVE pricing, L4 would have cost a few hundred dollars a year. What it would have cost is the ability to say, credibly, that the narrative had been checked. A loop that reports improvement the evaluation cannot confirm (§14, anti-pattern 7) is expensive in a currency that does not appear on the invoice.

### 13.6 The recommendation, including what Cymbal refuses to automate

**Ship:** L1 (deterministic completeness and cross-document consistency), L2 (policy rule engine with a Compliance owner), L3 (citation-grounding with the *span-existence* check as an external signal), and L6 (scoped human gate on every escalation and every adverse-media hit, instrumented with approval/edit/override rates and time-per-review).

**Refuse to automate:** the *escalate-or-clear judgement itself* on any case with an adverse-media hit, an unexplained transaction pattern, or a jurisdiction on the enhanced-due-diligence list. The agent drafts; the decision is a named human's. Cymbal also refuses to build a self-grading quality loop (L4) and refuses to cite same-model debate (L5) as evidence of correctness anywhere in its assurance documentation.

**The one-sentence rationale to put in the design record:** *the loops Cymbal keeps close on signals the model cannot argue with; the loop it dropped closed on the model's opinion of itself, and the evidence says that signal can make things worse* — which is the signal-versus-loop distinction this guide exists to make, and §14–§16 close it out.

---

## 14. The Anti-Patterns

Each row is a symptom you can observe, a cause, and a guardrail that prevents it. The guardrails are the operating content of this guide.

### 14.1 The loop whose feedback is the model's own opinion

- **Symptom:** the design document says "the agent critiques and revises its own output"; the trace shows critique text that is fluent, plausible, and generic; the revision is cosmetic; a base-model-without-the-loop run would have produced something similar.
- **Cause:** the folk model of reflection, imported from human practice, where a person who reviews their own work brings *knowledge they did not use the first time*. A model asked to review its own output is usually being asked the same question with the same information available.
- **Guardrail:** apply the §5.3 trust ladder before writing the loop. Ask the one question from §12.1: *if the agent's output is wrong in the way this model is typically wrong, will this critique catch it?* If no, the loop is not a check. Then run the §13.4 verifier-only ablation — if re-prompting with the external signals alone matches the loop, the critique prose is not carrying the gain.

### 14.2 Iteration count raised instead of the signal improved

- **Symptom:** "we turned up the reflection rounds"; the change request raises `max_iterations`; nobody can say what round 4 is for.
- **Cause:** iteration count is the cheapest available knob and it looks like effort. It is also the knob that §9.3's data specifically disfavours: Reflexion's HotpotQA score is identical at rounds 2 and 3, so the marginal round is pure cost. And Olausson et al. found gains "often modest" once repair cost is counted.
- **Guardrail:** instrument the marginal round. If round *n* does not change the output on a sample of cases, stop at *n−1*. Treat the round count as a cost parameter with a measured ceiling, not a quality dial.

### 14.3 An internal loop used where an external verifier exists and is cheap

- **Symptom:** a model is asked to check its own JSON when a schema validator is installed; a model is asked whether its SQL is plausible when the query could be executed against a read replica; a model is asked whether its code works when a test suite exists.
- **Cause:** the deterministic verifier lives in another team's repo, or the API is awkward, or "self-check" was in the model's tool list and the validator was not.
- **Guardrail:** for every proposed internal loop, write down the cheapest external signal that could answer the same question. Compilers, schemas, type checkers, unit tests, reconciliation rules, policy engines and `EXPLAIN` plans are deterministic, auditable (§12.5) and effectively free (§13.5). Stechly et al.'s ablation result says the external verifier is where the gain is — so getting one is worth more than any amount of critique scaffolding. ⚠ Cross-reference [ai_verify_toolkit_guide.md](ai_verify_toolkit_guide.md) and [llm_guard_models_guide.md](llm_guard_models_guide.md).

### 14.4 Multi-agent debate treated as independent verification

- **Symptom:** the assurance note reads "two agents independently reviewed the recommendation"; the two agents are the same model, the same prompt scaffold, and the same retrieval index; the transcript mostly converges.
- **Cause:** counting agents instead of evidence channels. §6.4: shared model family, shared tools and shared information mean correlated failure; a debate among correlated instances can reach a confidently wrong consensus, and the survey itself names that failure ("multi-agent negotiations may converge on incorrect consensuses") before prescribing a peer-ranking fix.
- **Guardrail:** count the independent evidence channels, not the participants. A debate is evidence of correctness only if at least one participant has information or a tool the others lack, or if a verifier outside the debate can adjudicate. Otherwise describe it as a drafting aid, and never as a control.

### 14.5 A human-in-the-loop that is a feature rather than a control

- **Symptom:** the approval gate fires on everything; median review time is seconds; the override rate is ~0%; when asked, reviewers say they mostly click through; the gate's existence is cited as evidence of oversight.
- **Cause:** "human in the loop" as a checkbox. §7.3 and §12.2 both point the same way: the survey calls continuous human feedback "impractical due to limited resources or real-time constraints" — and a gate that ignores that constraint is the mechanism by which the impracticality is hidden rather than managed.
- **Guardrail:** the five conditions of §12.2 — scoped, authorised, rate-bounded, instrumented, owned. Watch two numbers in particular: **time per review** (a collapse means nobody is reading) and **override rate** (near-zero means the gate is decoration or unnecessary). And remember the ILLUSTRATIVE arithmetic of §13.5: full gating of 12,000 cases is ~1.5 FTE, scoped gating is ~0.12 FTE. Scoping is the difference between a control and a tax.

### 14.6 The benchmark borrowed rather than run

- **Symptom:** a mechanism is adopted because it scored well on a published benchmark — often the exact benchmarks in §9.2 — with no run on the institution's own task distribution; the design document cites a percentage from someone else's paper.
- **Cause:** benchmark results are the cheapest available evidence, and the survey's own Table 3 is *partly borrowed* (the survey says so). Cross-benchmark comparisons carry unknown harness variance, and §9.3 shows the rankings are not stable across the three benchmarks in that very table (AutoGuide best on ALFWorld at the highest cost, worse than Reflexion-R3 on WebShop).
- **Guardrail:** run the mechanism on a sample of your own cases, with the §13.4 paired + equal-compute + verifier-only design, before adoption. If the mechanism is reported to be excellent on ALFWorld, that is a reason to test it, not a reason to believe it. ⚠ [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md) owns this discipline.

### 14.7 The feedback mechanism that reports improvement the evaluation cannot confirm

- **Symptom:** the dashboard shows "quality score +12%" from a score produced by the same model that produced the output; the eval has no control arm, no cost accounting, and no human anchor; nobody can state what the counterfactual was.
- **Cause:** the judge and the judged are the same system, and the metric is computed inside the loop that is being evaluated. Zheng et al.'s primary names self-enhancement bias; §10.1 explains that outcome-based evaluation of a feedback mechanism cannot attribute the outcome to the mechanism; Olausson et al. showed the attribution changes materially when cost is counted.
- **Guardrail:** no loop's improvement claim ships without a counterfactual arm. Minimum viable trio: **base model**, **base + equal compute**, **base + the loop**. Add a human-anchored spot check on a sample, and report the false-positive rate alongside the gain. If the evaluation cannot confirm the improvement, the correct reporting is "we cannot confirm an improvement" — which is a useful, publishable, budget-defensible result.

### 14.8 The cross-cutting pattern

Seven anti-patterns, one cause. **In every case the team optimised the loop and left the signal alone.** More rounds, more agents, more critique, more reviewers, better benchmark numbers — and the same untrustworthy evidence flowing through. Which is the guide's thesis arriving at its least comfortable form: if the signal is weak, no amount of loop makes it strong, and a stronger-looking loop makes the weakness harder to see.

---

## 15. The Claims Audit

**The central evidentiary discipline of this guide:** *what the survey reports* and *what the primary source says* are different objects, and they are kept separate in every row below. "Survey's claim" is a quotation or close paraphrase of the anchor paper; "What the primary actually says" is what was read at the primary source in this run. Where the two agree, the row is ✅. Where the survey is imprecise, incomplete or wrong, the row is ⚠. Where nothing could be confirmed, it is ❌.

### 15.1 Verification against primary sources

| Work | Survey's claim / classification | What the primary actually says | Venue verified? | Verdict |
|---|---|---|---|---|
| **Huang et al., *Large Language Models Cannot Self-Correct Reasoning Yet*** | The survey does not cite this work at all | "LLMs struggle to self-correct their responses without external feedback, and at times, their performance even degrades after self-correction"; framework named **intrinsic self-correction** | ✅ **ICLR 2024** (arXiv:2310.01798, v1 3 Oct 2023, v2 14 Mar 2024), confirmed at the arXiv record | ✅ verified; **omission from the survey flagged** |
| **Kamoi et al., *When Can LLMs Actually Correct Their Own Mistakes?*** | Not cited by the survey | (1) no prior work shows successful self-correction with prompted-LLM feedback "except for studies in tasks that are exceptionally suited for self-correction"; (2) it works with reliable external feedback; (3) large-scale fine-tuning enables it; plus: prior work "over-evaluate[s] self-correction" | ✅ **TACL 2024, 12:1417–1440** (arXiv:2406.01297) | ✅ verified; **omission from the survey flagged** |
| **Valmeekam, Marquez, Kambhampati, *Can LLMs Really Improve by Self-critiquing Their Own Plans?*** | Not cited by the survey | "self-critiquing appears to diminish plan generation performance"; LLM verifiers "produce a notable number of false positives"; binary vs detailed feedback "showed minimal impact" | ⚠ arXiv:2310.08118 (12 Oct 2023); peer-reviewed venue **not verified** | ✅ verified at arXiv; venue ⚠ |
| **Stechly, Valmeekam, Kambhampati, *On the Self-Verification Limitations of LLMs*** | Not cited by the survey | "significant performance collapse with self-critique and significant performance gains with sound external verification"; "merely re-prompting with a sound verifier maintains most of the benefits of more involved setups" | ⚠ arXiv:2402.08115 (12 Feb 2024, v2 3 Aug 2024); venue **not verified** | ✅ verified at arXiv; venue ⚠ |
| **Olausson et al., self-repair** | Cited as "Demystifying GPT Self-Repair for Code Generation. Arxiv, 2023", and used in §4.1 as an example of *outcome-based evaluation* ("evaluated by the pass rate of programming tasks") | Gains "often modest, vary a lot between subsets of the data, and are sometimes not present at all" once repair cost is counted; self-repair "bottlenecked by the model's ability to provide feedback on its own code"; stronger-model feedback → substantially larger gains; human feedback still far ahead | ✅ **ICLR 2024** poster (OpenReview `y0GJXRungR`; arXiv:2306.09896) — the survey's reference gives the pre-publication title/status | ⚠ **survey's summary omits the paper's central negative finding**; venue in survey understated |
| **Reflexion (Shinn et al.)** | Classified intra-task **and** inter-task; Table 1 iteration ✓ | "reinforce language agents **not by updating weights**, but instead through linguistic feedback"; reflections kept in an episodic memory buffer; "91% pass@1 accuracy on the HumanEval coding benchmark, surpassing … GPT-4 that achieves 80%" | ⚠ survey's reference says "In NeurIPS, 2024"; arXiv:2303.11366 (20 Mar 2023) — the arXiv record states no venue | ⚠ duplicated classification; ⚠ venue/year discrepancy |
| **Self-Refine (Madaan et al.)** | Intra-task internal feedback; Table 1 ✓; also cited as a prior *survey* in the introduction | Iterative feedback and refinement by one LLM acting as generator, feedback provider and refiner; "improving by **~20% absolute on average**" across 7 tasks; no training, no RL | ⚠ survey's reference says "In NeurIPS, 2024"; arXiv:2303.17651 (30 Mar 2023); no venue in the arXiv record | ✅ substance verified; ⚠ venue/year |
| **ExpeL (Zhao et al.)** | Inter-task internal feedback (NL, ICL, ✓) | "Accepted by the 38th Annual AAAI Conference on Artificial Intelligence (**AAAI-24**)" stated at the primary; autonomously gathers experiences and extracts natural-language knowledge; no parametric policy updates | ✅ **AAAI-24** | ✅ |
| **AdaPlanner (Sun et al.)** | Intra-task internal feedback; Table 3: ALFWorld 63%, multi-round, high stability | Closed-loop refinement of a self-generated plan "in response to environmental feedback", with in-plan and out-of-plan refinement; code-style prompt to mitigate hallucination; evaluated on ALFWorld and MiniWoB++ | The survey cites NeurIPS 2023; the arXiv record (2305.16653) states no venue | ✅ mechanism/description verified; ⚠ venue not confirmed here |
| **AutoGuide (Fu et al.)** | Table 3: ALFWorld 79%, WebShop 46%, high cost/stability; a source of borrowed results | Generates context-aware natural-language guidelines from offline experience, each with a conditional structure; "significantly outperforms competitive baselines in complex benchmark domains, including real-world web navigation" | The survey cites it as arXiv 2024; the arXiv record (2403.08978) states no venue | ✅ description verified; ⚠ the specific 79%/46% figures were **not** verified at the primary |
| **Voyager (Wang et al.)** | External feedback (game API); Table 1 marks iteration **✗** ("non-iterating") | "a new **iterative** prompting mechanism that incorporates environment feedback, execution errors, and self-verification for program improvement" | The survey cites NeurIPS 2023; the arXiv record (2305.16291) states no venue | ❌ **survey's Table 1 iteration marker contradicts the primary abstract** |
| **MetaGPT (Hong et al.)** | Multi-agent collaborative; "an executable feedback loop for iterative code refinement"; Table 1 marks iteration **✗** | MetaGPT encodes SOPs into prompt sequences so that "agents with human-like domain expertise **verify intermediate results and reduce errors**"; assembly-line role assignment for collaborative software engineering | The survey cites ICLR 2024; the arXiv record (2308.00352) states no venue | ⚠ internal inconsistency between survey body and Table 1; ⚠ venue not confirmed here |
| **LLM-as-a-judge / MT-Bench (Zheng et al.)** | "suffers from positional bias, verbosity bias, and limitations in evaluating mathematical problems" | "we examine the usage and limitations of LLM-as-a-judge, including **position, verbosity, and self-enhancement biases, as well as limited reasoning ability**" | ✅ **NeurIPS 2023 Datasets and Benchmarks Track** (arXiv:2306.05685) | ✅ accurate but incomplete: the survey **omits self-enhancement bias**, the most relevant one for self-grading loops |
| **Agent-as-a-Judge / DevAI (Zhuge et al.)** | Cited as two separate works: "DevAI [Zhuge et al., 2024]" (55 tasks, process-based) and "Agent-as-a-Judge [Zhuge et al., 2024]" | **One paper.** Agent-as-a-Judge framework + DevAI benchmark of "**55 realistic automated AI development tasks**" with "a total of **365 hierarchical user requirements**"; claims it "dramatically outperforms LLM-as-a-Judge and is as reliable as our human evaluation baseline" | NeurIPS 2024 per common citation; the arXiv record (2410.10934, Oct 2024) states no venue | ⚠ **double-citation of a single work**; ✅ the 55-task figure matches the primary exactly |
| **ToolBench (Qin et al.)** | Table 2: "16k API, 3k tools"; reference list cites "Tool learning with foundation models. ACM Comput. Surv, 2024" | ToolLLM's ToolBench: "we collect **16,464 real-world RESTful APIs** spanning **49 categories** from RapidAPI Hub" | arXiv:2307.16789; the survey's cited reference is a **different paper** by the same first author | ✅ 16k APIs; ❌ "3k tools" unverified; ⚠ **citation–subject mismatch** |
| **HotPotQA (Yang et al.)** | 113k Q&A pairs; reasoning domain | "**113k** Wikipedia-based question-answer pairs" | ✅ EMNLP 2018 | ✅ |
| **FEVER (Thorne et al.)** | 185k claims | "It consists of **185,445 claims**" | ✅ NAACL 2018 | ✅ |
| **ScienceQA (Lu et al.)** | 21k Q&A pairs; text, image | "**~21k** multimodal multiple choice questions" | ✅ NeurIPS 2022 | ✅ (note: multiple choice, which the survey later criticises) |
| **StrategyQA (Geva et al.)** | 2.7k Q&A pairs | "StrategyQA includes **2,780 examples**" | ⚠ the survey's reference says "In ACL, 2021"; the primary record says **TACL 2021** | ✅ size; ⚠ venue |
| **Mind2Web (Deng et al.)** | 2k tasks; text, image; survey's reference "In NeurIPS, 2024" | "over **2,000** open-ended tasks … from **137 websites** spanning **31 domains**" | ⚠ primary's own record: "**NeurIPS'23 Spotlight**" | ✅ size; ❌ **venue wrong in the survey** |
| **SWE-bench (Jimenez et al.)** | 2k problems | "**2,294** software engineering problems … across **12** popular Python repositories" | ✅ **ICLR 2024** (stated at the primary) | ✅ |
| **MBPP (Austin et al.)** | 1k problems | "The MBPP dataset contains **974** programming tasks" | ✅ arXiv 2021 (the survey's own citation) | ✅ approximately |
| **ALFWorld (Shridhar et al.)** | 3.1k examples; survey's reference "In ICLR, 2020" | Size not stated in the abstract; primary's own record: "**ICLR 2021**" | ❌ **venue wrong in the survey (ICLR 2021, not ICLR 2020)** | ❌ venue; ❌ size unverified |
| **ChatEval (Chan et al.)** | Adversarial multi-agent; survey's reference "In ICLR, 2023" | arXiv:2308.07201 posted **14 Aug 2023**; no venue at the record | ❌ an August-2023 posting cannot be an ICLR 2023 paper (ICLR 2023 decisions predate it) | ❌ **venue implausible in the survey** |
| **AgentBench (Liu et al.)** | 1.4k samples; multidimensional evolution | "**8 distinct environments**"; sample count not stated in the abstract | ✅ **ICLR 2024** (stated at the primary) | ✅ venue; ❌ "1.4k samples" unverified |
| **FinQA (Chen et al.)** | 2.8k reports, 8k Q&A pairs; survey's reference "arXiv, 2021" | Sizes not stated in the abstract; primary's own record: **EMNLP 2021** | ⚠ survey understates (arXiv vs EMNLP) | ❌ sizes; ⚠ venue |
| **Reflexion's Table 3 figures (R1/R2/R3)** | 33/40/40 % HotpotQA; 48/52/54 % ALFWorld; 43/46/48 % WebShop | Not verifiable here: the survey states results are **borrowed** from ExpeL/AutoGuide, and the specific rows are not attributed | n/a | ⚠ **partly second-hand by the survey's own admission** (§9.1) |

### 15.2 How to read this audit

- **Coverage.** The primary source was located and read for **26 of the 27 works** in the table (all except the Table 3 figures row, which is not a single work). Across them, the survey's **descriptions of mechanisms held on substance in the large majority of cases** — the survey characterises other people's systems carefully. The defects are concentrated in the reference list and in two citation-construction cases, not in the characterisations.
- **Venue/year defects the primary contradicts or renders implausible (4 clear, 2 understated).** ❌ Mind2Web — primary says *NeurIPS'23 Spotlight*, survey says NeurIPS 2024. ❌ ALFWorld — primary says *ICLR 2021*, survey says ICLR 2020. ⚠ StrategyQA — primary says *TACL 2021*, survey says ACL 2021. ❌ ChatEval — survey says ICLR 2023, but the arXiv posting is 14 Aug 2023, after ICLR 2023. ⚠ FinQA — primary says *EMNLP 2021*, survey says arXiv 2021. ⚠ Olausson et al. — survey gives the pre-publication title and status for an ICLR 2024 paper. **Do not copy the survey's reference list as a bibliography without checking each entry.**
- **One table cell contradicts its primary source.** Voyager marked as *non-iterating* (✗) in Table 1 while the primary abstract describes "a new **iterative** prompting mechanism".
- **Three citation-construction defects.** The Agent-as-a-Judge paper and its DevAI benchmark are cited as *two separate works*; ToolBench is attributed in the reference list to a different paper by the same first author ("Tool learning with foundation models", ACM Comput. Surv. 2024) rather than to ToolLLM; and FLORES-200 is attributed to Guzmán et al. 2019, which is a different FLORES resource.
- **The one substantive divergence is Olausson et al.** The survey cites the paper as a *method for outcome-based evaluation* and omits the paper's central negative finding about self-repair. This is the guide's clearest example of the difference between a categorisation and a finding.
- **Two structural defects in the survey's own tables are documented above**: the iteration column contradicted by its own body text and by Table 3 (ReAct, Voyager, MetaGPT, JARVIS-1), and the Reflexion duplicate across the intra-task and inter-task rows.
- **The negative literature the survey omits is the audit's most consequential finding.** Four verified primary sources (§8.1–§8.5) — two of them strong venue-verified results — bear directly on the mechanisms the survey is optimistic about, and **none of the four appears in the survey's reference list**.

---

## 16. What Could Not Be Verified; Glossary; Cross-References; Closing

### 16.1 What could not be verified (❌), and the tool limitations behind it

Recorded honestly, because a gap in verification is not evidence of absence.

**Unverified claims, with the reason:**

| Item | Status | Why |
|---|---|---|
| Table 2 sizes: **HumanEval 164** | ❌ unverified | Primary retrieved but the abstract was not reached in this run (long author list pushed it past the extraction window); the figure is widely repeated but was not confirmed here |
| Table 2 sizes: **ALFWorld 3.1k examples** | ❌ unverified | Not stated in the primary abstract |
| Table 2 sizes: **WebArena 812 tasks** | ❌ unverified | Not stated in the primary abstract (four domains and human/agent success rates are) |
| Table 2 sizes: **AgentBench 1.4k samples** | ❌ unverified | Not stated in the primary abstract |
| Table 2 sizes: **ToolBench 3k tools** | ❌ unverified | The primary abstract gives 16,464 APIs across 49 categories but no tool count |
| Table 2 sizes: **FinQA 2.8k reports / 8k QA pairs** | ❌ unverified | Not stated in the primary abstract |
| Table 2 sizes: **RT-X 1m trajectories** | ❌ unverified | Primary not retrieved in this run |
| Table 2 sizes: **FLORES-200 842 articles / 3k sentences** | ❌ unverified, and the **attribution is suspect** | The survey cites Guzmán et al. 2019, which is the FLORES **Nepali–English and Sinhala–English** datasets paper; FLORES-200 is a different, later resource. Either the figure or the citation is wrong |
| Table 3 figures (all rows) | ❌ not independently verified | The survey states some results are borrowed; no per-row attribution is given |
| Venues for: Multiagent Debate (ICML 2024 per survey), AdaPlanner (NeurIPS 2023), AutoGuide (arXiv 2024 per survey), MetaGPT (ICLR 2024), Voyager (NeurIPS 2023), Agent-as-a-Judge (ICLR/NeurIPS 2024), Valmeekam et al., Stechly et al. | ⚠ survey-cited or unstated; primary records retrieved here carried no venue field | The arXiv records for these works do not state a venue |
| The companion paper list at `github.com/kevinson7515/Agents-Feedback-Mechanisms` | ✅ exists — contents reviewed 21 Sep 2026 | Public repo; single `README.md`, 11 commits, created 05/2025, last update 09/2025; ~64 arXiv-linked entries in taxonomy-matching categories. Not a code artefact and not a dataset. Its coverage is the optimistic literature only — no counter-evidence works listed, and the link status of several entries is incomplete |
| Reflexion's claimed round-by-round setup (what "R1/R2/R3 with experiment replay" means operationally) | ❌ unverified | The survey's column is not defined; Reflexion's primary abstract describes reflective memory, not numbered rounds |

**Tool limitations encountered (recorded as limitations, not as evidence of absence):**

- **`web_search` returned empty result sets for six queries in the second half of this run** — including queries for Stechly et al., Valmeekam et al., Reflexion's NeurIPS venue, Voyager's venue, Mind2Web's NeurIPS year, Retroformer's venue and Agent-as-a-Judge's venue. Stechly and Valmeekam were then verified by **direct `web_extract` on their arXiv identifiers**, which is what the audit table reflects. **The empty searches are a rate-limit/degradation artefact, not a finding that those works are unpublishable or unindexed.**
- **`dblp.org` was unavailable for venue lookups** — the site served an Anubis anti-bot proof-of-work challenge page, so the five planned dblp queries returned the challenge, not results. **No venue conclusion in this guide rests on dblp being silent.**
- **One arXiv identifier guess was wrong**: `2310.08529`, attempted for Retroformer, is *GaussianDreamer* (CVPR 2024). **Retroformer's venue was therefore not verified**, and the survey's "In ICLR, 2024" for it remains unconfirmed.
- **Extraction window truncation**: several arXiv abstract pages returned head+tail windows that cut the abstract and the "Comments" field. Those cases were resolved by reading the cached page at the offset the tool reported, which worked for all but `HumanEval` (Chen et al. 2021).
- **Per instruction, the anchor paper was read from the cached file** at `/tmp/feedback_survey_ijcai25.md` (2,123 lines) and was **not re-fetched or re-extracted** from ijcai.org.

**Source typos and rendering artefacts in the anchor paper (flagged as the source's rendering — not silently fixed, not reproduced as canonical):**

The parent verified all four of the following present in the cached text; this guide re-confirmed each at its location. **These are the survey's renderings; the corrected forms are given so that nobody indexes the typos as if they were canonical names.**

| Survey's rendering | Location in the survey | What it means |
|---|---|---|
| "**Reflextion**" (for *Reflexion*) | Table 1, intra-task row | The Reflexion paper by Shinn et al. |
| "**iamge**" (for *image*) | Table 2, DevAI row, modality column | Modality value "text, image" |
| "**Mathine Translation**" (for *Machine Translation*) | §4.2, domain heading | The machine-translation benchmark family |
| "**RocoBench**" (for the *RoCo* benchmark) | Table 2 (Multi-agent Collaboration) and §4.2 | The RoCo multi-robot collaboration benchmark (Mandi et al., ICRA 2024) |

Three further abbreviation artefacts were observed in the same read and are recorded for completeness: "**ACRL**" used for CoRL in the RT-2 reference entry; "**ICCL**" used for the venue of PreAct [Fu et al., 2025] and ToolEyes [Ye et al., 2025]; and the reference-list venue/year discrepancies in §15.1 (Mind2Web, ALFWorld, StrategyQA, ChatEval).

**Internal inconsistencies in the anchor paper's own tables** (all verified in the cached text):

1. **Table 1's iteration column vs the survey's body text.** ReAct marked ✗ while Table 3 lists ReAct's feedback round as "Multi-round"; Voyager, JARVIS-1 and MetaGPT marked ✗ while the body describes them as iterative ("executable feedback loop for iterative code refinement", "iterative hinting mechanism"). Voyager's primary abstract states "iterative prompting mechanism" explicitly.
2. **Reflexion appears twice in Table 1** — as "Reflextion" under intra-task and as "Reflexion" under inter-task — and the survey's own §3.1 prose describes Reflexion as inter-task.

### 16.2 Glossary

| Term | Definition |
|---|---|
| **Feedback mechanism** | The component converting evidence about the agent's own behaviour into a change in what it does next. The anchor survey's unit of analysis. |
| **Internal feedback** | Survey category 1: feedback generated by the agent itself, without explicit external goals or tools. Split into intra-task and inter-task. |
| **External feedback** | Survey category 2: environment-defined signals and external tools. Split into web knowledge, game API, code interpreter, world model. |
| **Multi-agent feedback** | Survey category 3: agents interacting to refine a solution. Split into collaborative and adversarial. |
| **Human feedback** | Survey category 4: human input as the feedback source. Split into instructional, corrective, preference-based. |
| **Intra-task / inter-task feedback** | Intra-task: feedback used within one task attempt — optimises fast, does not accumulate. Inter-task: feedback carried across tasks via memory — accumulates, at a memory and hygiene cost. |
| **In-context learning (ICL) / supervised (SL) / reinforcement (RL) / imitation (IL)** | The survey's Table 1 learning formats: ICL is learning inside the prompt with no weight change — the dominant format in this literature; SL, RL and IL are weight- or policy-level learning, and in this field the label usually describes the original paper's training regime rather than one you can reproduce. |
| **Outcome-based evaluation** | Measuring a mechanism by end-to-end success. The survey: most evaluations are of this kind. Cannot attribute the outcome (§10.1). |
| **Process-based evaluation** | Evaluating intermediate steps as well as outcomes; needs an intermediate ground truth, which usually means a judge. |
| **LLM-as-a-judge** | Using an LLM to score outputs. Its own primary documents position, verbosity and self-enhancement biases and limited reasoning ability. |
| **Agent-as-a-judge** | Extending the judge with agentic features to give intermediate feedback — the DevAI paper's framework. Same-work-as-DevAI (§15.1). |
| **Reflexion** | Verbal reinforcement learning: post-attempt reflection stored in an episodic buffer, used in subsequent attempts; no weight updates. |
| **Self-Refine** | Iterative refinement with one LLM as generator, feedback provider and refiner; no training. |
| **Self-critique / intrinsic self-correction** | The model judging and revising its own output with no external signal. The mechanism the negative literature is about. |
| **Reward model** | A learned scorer used as a training or search signal; another model with its own systematic error mode. |
| **Verifier** | An independent check on the output — a test, a schema, a rule engine, a policy engine, a guard model. A *sound* verifier is one whose correctness does not depend on a model's opinion. |
| **Control** | An independent check with a defined scope, an owner, retained evidence and a visible failure mode. Distinguishable from a quality mechanism by independence and accountability (§12.1). |
| **Quality mechanism** | A mechanism that improves output, without the independence or accountability requirements of a control. |
| **Independent evidence channel** | A source of information that does not share the model's failure mode for the question at hand — a different tool, different information, or a non-model oracle. Count these, not agents (§6.4). |
| **Verifier-only ablation** | An evaluation arm in which only the external signal is retained and the critique/loop scaffolding removed. If it matches the full loop, the loop is decoration (Stechly et al.; §13.4). |
| **Signal trustworthiness** | Whether a feedback signal's correctness can be decided without a model, and what happens when it is wrong. The axis this guide substitutes for the survey's source-based taxonomy (§5.3). |
| **Saturation** | The point at which another iteration round produces no measurable change — e.g. Reflexion-R2 = R3 = 40% on HotpotQA (§9.3). |
| **Sim-to-real** | The transfer gap between simulation-trained behaviour and real-world physics; the survey's §6.5. |

### 16.3 Cross-references

**Owned by this guide:** the feedback-mechanism taxonomy and its evidentiary status — the four categories as this survey defines them, the representative systems and their learning formats, the evaluation protocols, the benchmark landscape, the survey's comparison and its caveats, the limitations and future directions, and the critical reading of the evidence.

**Owned by sibling guides (cross-reference, do not re-derive):**

| Neighbour | What it owns | Where this guide touches it |
|---|---|---|
| [multi_pass_llm_pipelines_guide.md](multi_pass_llm_pipelines_guide.md) | Multi-pass/refinement **pipeline** patterns (§4 Multi-Agent Patterns, §5 Reasoning Patterns, §11 Multi-Pass for Agentic Systems) | §3.3, §6.4, §14 |
| [autonomous_agents_guide.md](autonomous_agents_guide.md) · [agents_work_fall_apart_guide.md](agents_work_fall_apart_guide.md) · [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) | Agent architecture taxonomy §3 and evaluating agents §6; success conditions §1 and failure conditions §4; production failure taxonomy §2 and mitigations §8 | §1.5, §3.3, §6.4, §14 |
| [agentic_engineering_guide.md](agentic_engineering_guide.md) · [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) · [agent_scaffolding_guide.md](agent_scaffolding_guide.md) | Lifecycle (§8 Phase IV Evaluation, §15 Open Problems); testing §4; scaffolding patterns | §1.5, §3, §10.1, §14 |
| [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) · [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md) | Evaluation frameworks; evaluation vs validation discipline | §10.1, §14.6 |
| [llm_guard_models_guide.md](llm_guard_models_guide.md) · [ai_verify_toolkit_guide.md](ai_verify_toolkit_guide.md) · [agentic_search_vs_rag_guide.md](agentic_search_vs_rag_guide.md) | Guard models — themselves feedback mechanisms; verification harnesses; retrieval as signal vs noise and RAG cross-validation | §5.3, §8.6, §12.5, §14.3 |
| [closed_loop_data_engineering_guide.md](closed_loop_data_engineering_guide.md) | The data pipeline carrying feedback back into training | §7.2, §12.3 |
| [ai_agent_drift_guide.md](ai_agent_drift_guide.md) | Temporal behaviour change and drift | §3.2, §4.3, §11 |
| [agentops_guide.md](agentops_guide.md) · [agent_versioning_guide.md](agent_versioning_guide.md) · [context_engineering_guide.md](context_engineering_guide.md) · [agent_runtime_cache_design_guide.md](agent_runtime_cache_design_guide.md) | Ops layer — traces, budgets, alerts; pinning model/prompt/memory versions; context budgets; runtime caching for repeated calls | §3.3, §11, §12.4 |
| [llm_instruction_tuning_guide.md](llm_instruction_tuning_guide.md) · [fine_tuning_frameworks_comparison_guide.md](fine_tuning_frameworks_comparison_guide.md) | The weight-shaping half of human feedback | §7.2, §12.3 |
| [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md) · [hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md) · [multi_agent_banking_guide.md](multi_agent_banking_guide.md) | Multi-agent orchestration | §6.3, §6.4 |
| [mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md) · [mcp_discovery_guide.md](mcp_discovery_guide.md) · [mcp_framework_tools_guide.md](mcp_framework_tools_guide.md) | MCP and protocol standardisation | §6.3, §11 |
| [ai_governance_framework_guide.md](ai_governance_framework_guide.md) · [implementing-responsible-ai.md](implementing-responsible-ai.md) · [ai_governance_bias_redteaming_guide.md](ai_governance_bias_redteaming_guide.md) · [ai_red_teaming_guide.md](ai_red_teaming_guide.md) | Governance, oversight and audit; bias amplification and adversarial testing | §7.5, §11, §12 |
| [prompt_injection_guide.md](prompt_injection_guide.md) · [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md) | Injection — a loop amplifies injected instructions in retrieved content; containment for execution-based feedback | §5.3 (implied), §14 |
| [vision_language_models.md](vision_language_models.md) · [../physical_ai_guide.md](../physical_ai_guide.md) · [rag/rag_evaluation_methodology_guide.md](rag/rag_evaluation_methodology_guide.md) | Multimodal models; embodied and physical AI; RAG-specific evaluation methodology | §10.1, §11 (§6.2, §6.5) |

### 16.4 Closing

The survey gives the field a map, and the map is useful: four categories, a five-module framework, a benchmark landscape with honest weaknesses named for almost every family, and six challenges that a practitioner can recognise in their own backlog. Take the map. Take the *paper list* it maintains. Take the survey's own criticism of the benchmarks, which is better than most surveys manage.

Then take the counter-evidence, because the map's organising axis is not the axis that determines whether a loop works. The survey sorts feedback by **where it comes from** — internal, external, multi-agent, human. The evidence sorts it by **what it is worth**: whether the signal's correctness can be decided without asking a model, and what happens when it is wrong. A compiler's error is a different object from a world model's prediction; a human preference that retrains weights is a different object from a human correction applied mid-task; and a model's judgement of its own reasoning is a different object from all of them, because its error correlates with the error you are trying to catch. The survey's four categories flatten those distinctions. The evidence does not.

That is why the drop decision in §13.3 matters more than any gain in §9.2, and why §12.1's control-versus-quality-mechanism distinction is the one to carry into a design review. A self-grading loop can be a real quality mechanism — Self-Refine's own results and Olausson et al.'s stronger-critic result both show refinement doing work when something informative is at the bottom of it. What it cannot be is a control, and what it must not be is evidence that the output was checked. Huang et al. found reasoning degrading after self-correction; Valmeekam et al. found self-critique diminishing plans and model verifiers passing bad ones; Stechly et al. found the gains sitting in the sound verifier while the scaffolding around it contributed little; Kamoi et al. found no demonstrated success for prompted-LLM feedback outside tasks with reliable external feedback. Four independent results, one direction: **get the signal from outside the model, or expect the loop to ratify what is already there.**

Build the loop. Measure the round. Count the cost. Run the ablation. Then check what it closed on — because that, and not the number of iterations, is what decides whether you built an improvement or a rehearsal.

**The signal, not the loop.**
