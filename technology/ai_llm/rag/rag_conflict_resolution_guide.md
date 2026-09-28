# RAG Conflict Resolution — Retrieval Does Not Resolve Conflicts, Authority Does

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Cymbal Bank
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** LLM/AI Engineering Guides
> **Companion Guides:** [Advanced RAG Techniques](advanced_rag_techniques_guide.md) · [RAG Evaluation Methodology](rag_evaluation_methodology_guide.md) · [RAG Benchmarks](rag_benchmarks_guide.md) · [Hierarchical Multi-Agent Frameworks](../hierarchical_multi_agent_frameworks_guide.md) · [Feedback Mechanisms for LLM Agents](../feedback_mechanisms_llm_agents_guide.md)
> **Last Updated:** September 2026

---

## Table of Contents

1. [The Overview, the Decoder and the Boundary](#1-the-overview-the-decoder-and-the-boundary)
2. [The Kinds of Conflict](#2-the-kinds-of-conflict)
3. [What the Research Actually Shows About Model Behaviour](#3-what-the-research-actually-shows-about-model-behaviour)
4. [Why the Pipeline Hides the Conflict](#4-why-the-pipeline-hides-the-conflict)
5. [Detecting That a Conflict Exists At All](#5-detecting-that-a-conflict-exists-at-all)
6. [The Resolution Ladder](#6-the-resolution-ladder)
7. [Authority and Entitlement — the Enterprise Mechanism](#7-authority-and-entitlement--the-enterprise-mechanism)
8. [Temporal Conflict and the Point-in-Time Question](#8-temporal-conflict-and-the-point-in-time-question)
9. [The Design Patterns](#9-the-design-patterns)
10. [The Evaluation Question](#10-the-evaluation-question)
11. [The Regulated-Institution Angle](#11-the-regulated-institution-angle)
12. [The Cost and Performance Angle](#12-the-cost-and-performance-angle)
13. [Worked Example — Cymbal Bank, Two Procedures and One Regulation](#13-worked-example--cymbal-bank-two-procedures-and-one-regulation)
14. [The Anti-Patterns](#14-the-anti-patterns)
15. [The Claims Audit](#15-the-claims-audit)
16. [What Could Not Be Verified, Glossary, Cross-References and Closing Summary](#16-what-could-not-be-verified-glossary-cross-references-and-closing-summary)

---

## 1. The Overview, the Decoder and the Boundary

### 1.1 The thesis in one line

**Retrieval does not resolve conflicts; authority does.**

Retrieval decides *what is in the context window*. It says nothing about *what deserves to win* when two retrieved passages contradict each other. A vector index is a similarity machine, and similarity is not authority: a superseded procedure can be a near-perfect match for the query that the current procedure should have answered, and a fluent internal wiki page written last Tuesday can out-score the regulator's own instrument that the page paraphrases. Every pipeline that treats retrieval as the resolution step has quietly outsourced a governance decision to a cosine score.

This guide is about that gap. It is not about making retrieval better. Better retrieval makes a conflict *more* likely to be surfaced, not less — which is progress, but only if something downstream knows what to do with it.

### 1.2 Why this guide exists — the shelf is empty where it matters

The repository carries a nineteen-guide RAG shelf in `technology/ai_llm/rag/`. Read all nineteen and you will find, at most, a single passing use of the word "conflict" in any one of them. The techniques are all there — hybrid retrieval, reranking, query rewriting, corrective loops, chunking strategy, evaluation, benchmarking, long-context comparison — and not one of them addresses the case where **two retrieved documents disagree and the pipeline must decide which to speak with**.

This is worth stating plainly because repository-wide keyword counts invite a false conclusion. A grep for `conflict` across the repo returns a large number, and `conflicting` and `contradict` likewise. Those counts are dominated by subject matter that has nothing to do with retrieved evidence: agent-versus-agent disagreement, concurrency and write conflicts, requirements conflicts, conflict-of-interest sections in compliance material. Keyword density is not coverage. The shelf's coverage of *conflicting retrieved sources* is zero, and this guide is written to fill exactly that hole and no other.

### 1.3 The boundary, drawn by name

***This guide is about conflicting sources at retrieval time, not conflicting agents.*** Almost every "conflict resolution" artefact in the repository is about the other thing, and the distinction is not pedantic — the mechanisms are different, the failure modes are different, and the owners are different.

**Owned elsewhere — do not look for it here:**

| Subject | Owner |
| --- | --- |
| Agent-versus-agent disagreement: arbitration, voting, hierarchical resolution between agents | [Hierarchical Multi-Agent Frameworks](../hierarchical_multi_agent_frameworks_guide.md) · [Agents at Scale](../agents_at_scale_guide.md) · [Hybrid Multi-Agent Systems](../hybrid_multi_agent_systems_guide.md) |
| Concurrency conflicts — optimistic and pessimistic write conflict, lost updates, locking | [Agents at Scale](../agents_at_scale_guide.md) — this is a *write* conflict between concurrent mutators, a different problem class from two documents disagreeing at read time |
| Progressive disclosure of tool and context surface | [MCP Progressive Disclosure](../mcp_progressive_disclosure_guide.md) |
| Verifier and critique loops as a mechanism | [Feedback Mechanisms for LLM Agents](../feedback_mechanisms_llm_agents_guide.md) — a critique loop is *one response* to a conflict, but that guide owns the mechanism in the agent-output setting; this guide will not re-derive it, only reference it as rung 5 of the ladder in [§6](#6-the-resolution-ladder) |
| Evaluation methodology and metric definitions | [RAG Evaluation Methodology](rag_evaluation_methodology_guide.md) |
| Evaluation tooling | [RAG Evaluation Tools Comparison](rag_evaluation_tools_comparison_guide.md) |
| Retrieval optimisation techniques | [RAG Optimization Techniques](rag_optimization_techniques_guide.md) — conflict resolution is **not** an optimisation technique and must not be framed as one; optimisation makes the answer better on average, conflict resolution decides what is *true* when the corpus disagrees with itself |
| The advanced-technique landscape, including corrective patterns | [Advanced RAG Techniques](advanced_rag_techniques_guide.md) — cross-referenced, not re-derived |
| Public benchmark landscape | [RAG Benchmarks](rag_benchmarks_guide.md) |
| Beyond-RAG architectures, long-context alternatives, streaming ingestion, query rewriting, vector-store selection | [Beyond RAG](beyond_rag_guide.md) · [RAG vs Long-Context LLMs](rag_vs_long_context_llms_guide.md) · [RAG with Data Streaming](rag_with_data_streaming_guide.md) · [Query Rewriting for RAG](query_rewriting_rag_guide.md) · [Vector Databases](vector_databases_guide.md) |
| Point-in-time correctness as a data-management discipline | [Market Data Integrity](../../banking/market_data_integrity_guide.md) · [Data Pipeline Versioning](../../technology/data/data_pipeline_versioning.md) — [§8](#8-temporal-conflict-and-the-point-in-time-question) cross-references this rather than re-deriving it, because a **version conflict and a point-in-time question are the same problem seen from two directions** |

**This guide owns:** the conflict among retrieved evidence — its taxonomy, the research on model behaviour in the face of it, the pipeline mechanics that hide it, detection, the resolution mechanisms ordered by honesty, authority and entitlement as the enterprise mechanism, and the regulated-institution consequences.

### 1.4 The decoder

Terms used throughout, defined once so the rest of the guide can be read quickly.

- **Knowledge conflict** — the general phenomenon in which a language model is exposed to information that contradicts either what it already holds or other information in the same context. The field's survey (Xu et al., *Knowledge Conflicts for LLMs: A Survey*, 2024, [arXiv:2403.08319](https://arxiv.org/abs/2403.08319)) organises the space and is the reference for the two-family taxonomy used in [§2.1](#21-what-the-literature-actually-classifies).
- **Context–memory conflict** — retrieved evidence contradicts what the model holds parametrically (from training). This is the family the behavioural studies test most often.
- **Inter-context conflict** — retrieved passages contradict *each other*. Same query, same context window, two incompatible assertions. **This is the family an enterprise RAG system meets in production**, and the family the literature has the least to say about.
- **Parametric knowledge** — what the model carries in its weights, as distinct from what it is given at inference time. It is undated, unattributed, and unauditable; that is the whole problem with letting it arbitrate.
- **Retrieved evidence** — the passages actually placed in context for this query. Not the corpus: the corpus is what you own, the evidence is what arrived.
- **Corroboration** — the degree to which multiple retrieved passages assert the same thing. Counts agreement. **Does not measure truth**; see [§6](#6-the-resolution-ladder) rung 4.
- **Authority tier** — the rank a source holds in an institution's source register, expressing its *entitlement to speak* on a subject. A governance artefact, not a model setting. See [§7](#7-authority-and-entitlement--the-enterprise-mechanism).
- **Source register** — the inventory of source systems, documents and instruments an enterprise will answer from, each carrying an authority tier, an owner, a version, and a jurisdiction. The register is what makes authority retrievable as metadata.
- **Superseded version** — a document that was correct for its period and has been replaced. It is not "wrong for all time"; it is wrong *for now*, and it is a live retrieval hazard precisely because it matches queries well.
- **Abstention** — declining to answer, with the reason stated, rather than asserting a resolution the evidence does not support. Treated in this guide as a first-class output, not a failure.

---

## 2. The Kinds of Conflict

Two layers are presented here in order. First the **literature's own taxonomy**, taken from the sources. Then the **enterprise inventory** — the kinds of conflict a bank actually meets — which is this guide's own analysis, offered as a practical inventory rather than a citable classification.

### 2.1 What the literature actually classifies

The survey (Xu et al., *Knowledge Conflicts for LLMs: A Survey*, 2024, [arXiv:2403.08319](https://arxiv.org/abs/2403.08319)) organises the field into **three categories** — it states its focus as context–memory, inter-context and intra-memory conflict — and reviewing causes, model behaviours under conflict and available solutions is precisely what it sets out to do:

1. **Context–memory conflict** — the retrieved or supplied context disagrees with the model's parametric knowledge. The survey's framing includes both the case where the context is right and the model's memory is stale or wrong, and the case where the context is wrong and the model's memory is right.
2. **Inter-context conflict** — the information within the supplied context disagrees, or the supplied context disagrees with information supplied by another source or tool in the same interaction. **This is the family this guide is about.**
3. **Intra-memory conflict** — the model's own encoded knowledge is internally inconsistent across prompts, phrasings or elicitations. It is real and it is the reason a model's answer can change between two identical-looking asks, but it is not the enterprise's primary problem and no retrieval discipline fixes it; the guides that own model behaviour and evaluation are the right home for it.

The behavioural paper that names the two failure modes is Xie et al., *Adaptive Chameleon or Stubborn Sloth: Revealing the Behavior of Large Language Models in Knowledge Conflicts*, 2023, [arXiv:2305.13300](https://arxiv.org/abs/2305.13300): a model that capitulates to whatever context it is handed is a **chameleon**; a model that holds its parametric position against supplied evidence is a **sloth**. Both are failure modes, and the paper's contribution is showing that which one you get depends on how coherent and convincing the supplied evidence is, and on whether part of that evidence agrees with the model's own memory. [§3](#3-what-the-research-actually-shows-about-model-behaviour) treats this in detail.

The connection between that taxonomy and the enterprise problem is direct and worth stating: **the enterprise inventory below is almost entirely inter-context conflict.** Version conflict, authority conflict, jurisdictional conflict, terminology conflict, unit conflict, and temporal conflict are all cases of two retrieved passages disagreeing with each other. Context–memory conflict is real, but in a governed deployment the model's parameters are the source of last resort, not the thing under adjudication.

### 2.2 The enterprise inventory

Seven recurring kinds, each with how it arises, how it is detected, and how it is resolved. Detection is treated in depth in [§5](#5-detecting-that-a-conflict-exists-at-all); resolution refers to the ladder in [§6](#6-the-resolution-ladder) and the authority mechanism in [§7](#7-authority-and-entitlement--the-enterprise-mechanism).

| # | Kind of conflict | Arises because | Cheapest detection | Correct resolution basis |
| --- | --- | --- | --- | --- |
| 1 | Version | A superseded document remains in the index | Version metadata on the chunk | Version / recency ([§8](#8-temporal-conflict-and-the-point-in-time-question)) |
| 2 | Authority | A summary is retrieved alongside the instrument it summarises | Source register tier on the chunk | Authority tier ([§7](#7-authority-and-entitlement--the-enterprise-mechanism)) |
| 3 | Jurisdictional | Two regulators, two regimes, one query | Jurisdiction metadata | Scope the question to a jurisdiction; both are correct within their scope |
| 4 | Terminology / definitional | Same word, two meanings; or two words, one meaning | Term registry lookup on the query and the passages | Definitional authority — which instrument defines the term |
| 5 | Unit and scale | USD against SGD; millions against billions | Typed quantity extraction from the passage | Normalise before comparing; never let the model eyeball it |
| 6 | Temporal | Statement true at t₁, false at t₂, both retrievable | Effective-date metadata | Point-in-time retrieval ([§8](#8-temporal-conflict-and-the-point-in-time-question)) |
| 7 | Genuine factual disagreement | The sources really do disagree — no ordering resolves it | Entailment check finds the contradiction; nothing resolves it | Surface the disagreement ([§6](#6-the-resolution-ladder) rung 1) |

**1. Version conflict.** A superseded procedure is still retrievable. This is the single most common conflict in a mature enterprise corpus and it is a *corpus hygiene* problem wearing a model-costume: the losing document should not have been in the index. It arises whenever document lifecycle management and index lifecycle management are separately owned — the document management system knows the procedure is retracted, the index does not, because nobody built the deletion path. Detection is trivial if version metadata is carried on the chunk, and impossible if it is not. Resolution is by version and effective date, which is the same machinery as the temporal kind ([§8](#8-temporal-conflict-and-the-point-in-time-question)).

**2. Authority conflict.** A summary retrieved against its own source instrument. A bank's internal "AML red flags" page and the regulator's published guidance both match a query about suspicious-transaction indicators, and the internal page is *more fluent, more specific and better matched* because it was written to be read. The regulator's instrument is the thing that has the right to speak. This is the conflict this guide is named for and the one the enterprise mechanism in [§7](#7-authority-and-entitlement--the-enterprise-mechanism) exists to settle.

**3. Jurisdictional conflict.** Two regulators, two rules, one question. A group-level policy answers a query in one jurisdiction's terms while the local regime requires another; both documents are correct and both are retrievable. There is no single winner — the correct resolution is usually to *scope* the answer, state which regulatory perimeter it assumes, and (where the user's jurisdiction is known) retrieve within that perimeter first. Authority tiers do not resolve this kind; jurisdiction does. Note that jurisdiction is a metadata filter, which makes this one of the cheapest conflicts to handle and one of the most commonly mishandled, because the jurisdiction of the *asker* is rarely captured.

**4. Terminology and definitional conflict.** The same word with two meanings — "customer" in a KYC procedure, a data-protection instrument and a marketing brief are three different populations — and the same meaning with two words ("disposal" and "retirement" for the same asset lifecycle step). Detection requires a term registry or a glossary artefact consulted *at query time*, not a similarity score; the passages will look mutually relevant precisely because they share vocabulary. Resolution is definitional authority: within an instrument's scope, that instrument's definition governs. This kind is under-recognised because it rarely produces an obviously contradictory answer — it produces a plausible answer computed on the wrong population, which is worse.

**5. Unit and scale conflict.** USD against SGD; millions against billions; a rate expressed as a percentage in one document and as basis points in another; a fiscal year in one and a calendar year in another. This is the most mundane entry in the table and one of the highest-frequency production failures. It is invisible to every semantic mechanism in the stack, because the passages are *not contradictory in shape* — they are numerically incompatible and lexically identical. Detection must be typed: extract quantities with units and scale as structured fields, compare, and flag. If this is left to the model, the model will silently pick one and produce a confident, wrong number. The correct handling is to normalise before comparison and to refuse to answer when the unit of the retrieved evidence cannot be established.

**6. Temporal conflict.** Two statements, both true, at different dates. See [§8](#8-temporal-conflict-and-the-point-in-time-question) — this is the kind where a bank's answer can be simultaneously correct and non-compliant.

**7. Genuine factual disagreement.** The sources really do disagree and no ordering resolves it. Two internal investigations reach different conclusions; two external studies measure differently; an expert's assessment contradicts a model's output. Every rung of the ladder except the first is unavailable or dishonest here, because the conflict is not about *which source has the right to speak* but about *which account of the world is correct*, and no metadata answers that. The correct output is the disclosed disagreement, and the correct system behaviour is to make the disclosure easy — which is a design goal, not a prompt.

The practical consequence of the inventory: **six of the seven kinds are settled by metadata and governance, and only the seventh is a genuine epistemic impasse.** That asymmetry is the guide's argument in miniature. If your conflict strategy is "ask the model to pick", you have brought a model to six fights that were never about judgement.

---

## 3. What the Research Actually Shows About Model Behaviour

This section reports what the sources report, with their settings, and separates that from what an enterprise would *like* to be true. The honest summary up front: **model behaviour in the face of conflicting evidence varies — with the model, with the strength and specificity of the evidence, with the setting and with the phrasing of the prompt — and no study cited here demonstrates a reliable general solution.**

### 3.1 The chameleon and the sloth

Xie et al. (*Adaptive Chameleon or Stubborn Sloth: Revealing the Behavior of Large Language Models in Knowledge Conflicts*, 2023 — ICLR 2024 Spotlight, [arXiv:2305.13300](https://arxiv.org/abs/2305.13300)) present a controlled investigation of what a model does when external evidence conflicts with its parametric memory. Their framework elicits high-quality parametric memory from the model and constructs the corresponding counter-memory, so the two can be set against each other deliberately. They report two behaviours, and the pairing is their title:

- **Receptivity.** Contrary to prior wisdom, they find that models **can be highly receptive to external evidence *even when it conflicts with their parametric memory*** — provided the supplied evidence is coherent and convincing. This is the **chameleon** failure: a fluent, internally consistent but wrong passage gets adopted.
- **Confirmation bias.** At the same time, models show a **strong confirmation bias when the supplied evidence contains information consistent with their parametric memory** — the model leans on the agreeing fragment and holds its position *despite* conflicting evidence presented alongside it. This is the **sloth** failure, and it is triggered by *partial agreement*, not by anything about the source's standing.

The design-relevant finding is the *dependence*: which behaviour a case produces turns on the **coherence and convincingness of the supplied evidence**, and on **whether part of that evidence echoes the model's own memory**. Both conditions are properties of the *writing* and of the *model* — not of the *authority* of the document. That is the warning this section carries into the rest of the guide: a system whose behaviour is a function of how convincingly the retrieved passage reads is a system with no control over which source wins.

### 3.2 Prior versus evidence — ClashEval

Wu, Wu and Zou (*ClashEval: Quantifying the tug-of-war between an LLM's internal prior and external evidence*, 2024, [arXiv:2404.10198](https://arxiv.org/abs/2404.10198)) build a dataset of **over 1,200 questions across six domains** (drug dosages, Olympic records, locations among them), attach answer-bearing content to each question, and perturb the answers in that content from subtle to blatant — then measure what six top-performing LLMs, including GPT-4o, do when prior and content disagree. Three reported results bear directly on enterprise design:

- **Models readily adopt incorrect retrieved content over their own correct prior knowledge** — in their benchmark, the models override a correct prior **over 60% of the time**. A wrong passage that is retrieved and read can displace the right answer the model already had.
- **The blatant error is *less* likely to be adopted than the subtle one.** The more the retrieved content deviates from truth, the *less* likely the model is to adopt it. Subtle corruption is therefore the dangerous case, and a superseded-but-plausible procedure is exactly that shape.
- **Adoption rises as the model's own confidence falls.** The less confident a model is in its initial response (measured through token probabilities), the more likely it is to adopt what the retrieved content says.

Read together, the deciding variables are properties of the *content* and of the *model's state* — how far the passage deviates, and how confident the model already was — and never what the passage *is*. The paper's own framing is that this is a difficult, benchmarked task: correctly discerning when the model is wrong in light of correct retrieved content, and rejecting content that is incorrect. It demonstrates methods for improving accuracy where retrieved content conflicts with a prior; it does not claim to resolve the general case. (The inference that this makes source-authority a retrieval-layer obligation rather than a model-side one is **this guide's**, not the paper's — see [§7](#7-authority-and-entitlement--the-enterprise-mechanism).)

### 3.3 The RGB benchmark — counterfactual robustness

Chen, Lin, Han and Sun (*Benchmarking Large Language Models in Retrieval-Augmented Generation*, 2023, [arXiv:2309.01431](https://arxiv.org/abs/2309.01431)) introduce the RGB benchmark and evaluate four abilities: **noise robustness**, **negative rejection**, **information integration** and **counterfactual robustness**. The counterfactual-robustness test is the conflict test: it presents retrieved evidence that contradicts the true answer and asks whether the model can resist it. Negative rejection is the abstention test: does the model decline when the retrieved evidence does not contain the answer? Both are reported as areas where the evaluated models are weak.

The RGB finding should be read with its setting in mind: the benchmark is constructed, the counterfactual context is *injected*, and the task shape is retrieval-augmented QA. It tells you that models fail on constructed counterfactuals. It does not tell you the failure rate on your corpus, because your corpus's conflicts are not injected — they are the residue of years of document management, and they carry metadata RGB does not model.

### 3.4 What corpus-scale conflict measurement adds — ConflictBank

Su et al. (*ConflictBank: A Benchmark for Evaluating the Influence of Knowledge Conflicts in LLM*, 2024, [arXiv:2408.12076](https://arxiv.org/abs/2408.12076)) present what they describe as the first comprehensive benchmark for systematically evaluating knowledge conflicts — from three aspects: conflicts encountered in **retrieved knowledge**, conflicts within the models' **encoded knowledge**, and the **interplay** between the two. It covers four model families and twelve LLM instances, and analyses conflicts stemming from **misinformation, temporal discrepancies and semantic divergences**, built through a construction framework that produces claim–evidence pairs and QA pairs at scale. Two contributions matter here. First, the benchmark's stated conflict *causes* — temporal discrepancy and semantic divergence — map directly onto two of the enterprise conflict types in [§2.2](#22-the-enterprise-inventory): the literature and the corpus agree on the shapes. Second, the instrument is **constructed**: conflicts are generated by a framework in order to be encountered, which is exactly what makes measurement possible and exactly what limits transfer to a production corpus whose conflicts arise from document lifecycle failure rather than from a generator.

The limit is the same as RGB's, and it is the limit this guide returns to in [§10](#10-the-evaluation-question): **the conflicts are constructed.** A constructed conflict is generated *for* the model to encounter; a production conflict is generated by document lifecycle failure and encountered by accident, usually without any indication that a competing source exists.

### 3.5 Irrelevant context and noise

Two further results bear on the pipeline rather than the model.

Yoran et al. (*Making Retrieval-Augmented Language Models Robust to Irrelevant Context*, 2023, [arXiv:2310.01558](https://arxiv.org/abs/2310.01558)) analyse five open-domain QA benchmarks to characterise when retrieval *reduces* accuracy, and propose two mitigations: filtering out retrieved passages that do not entail question–answer pairs according to a natural-language-inference model — which prevents the degradation but also discards relevant passages — and fine-tuning the model on an automatically generated mix of relevant and irrelevant contexts, which they report is effective with as few as 1,000 examples while preserving performance on relevant-context examples. Two things are worth carrying forward: retrieval augments can already *hurt* without any conflict being present, and NLI-based entailment filtering is the established mechanical response — which is the same machinery [§5](#5-detecting-that-a-conflict-exists-at-all) proposes for detecting contradiction, pointed at a harder target. The relevance here is directional: context that is *irrelevant* already degrades answers, and a conflicting passage is worse than irrelevant, because it is highly relevant and wrong.

Cuconasu et al. (*The Power of Noise: Redefining Retrieval for RAG Systems*, 2024, [arXiv:2401.14887](https://arxiv.org/abs/2401.14887)) examine the retrieval strategy itself, considering the relevance of the passages placed in context, their position and their number. Two of their reported results matter here. The retriever's **highest-scoring documents that are not directly relevant to the query** (documents that do not contain the answer) **negatively impact the LLM's effectiveness** — a near-miss that scores well does harm. And, more surprisingly, **adding random documents to the prompt improved accuracy by up to 35%** in their setting. Both directions are worth holding: **the retrieved set's *composition*, not only its relevance, is a behavioural variable.**

### 3.6 What the research does not show

Stated flatly, because the temptation to over-read is strong:

- **No study cited here shows a reliable, general resolution of inter-context conflict.** The literature's subject is predominantly context–memory conflict — model versus its own training — while the enterprise problem is predominantly inter-context.
- **The settings are largely synthetic or benchmark-shaped.** Injected counterfactuals, constructed claim–evidence pairs, controlled evidence-strength manipulations. Valuable for mechanism, not for rate estimation on a production corpus.
- **Behaviour is model-dependent and condition-dependent.** A finding about one model on one conflict type is a hypothesis about another, not a transferable rule.
- **Confidence and agreement are conditions, not controls.** Several studies vary how strongly the evidence is asserted or how much the model agrees with itself; those conditions are part of the reported result and must travel with it.

The conclusion the guide draws from this literature — and labels as its own conclusion — is not that models cannot help with conflict, but that **model behaviour is too variable to be the *basis* of conflict resolution in an institution.** It can be a rung. It cannot be the ladder.

---

## 4. Why the Pipeline Hides the Conflict

A conflict must be *retrieved* before it can be *detected*, and must be *attended to* before it can be *resolved*. Five mechanisms in the ordinary pipeline break one or both of those conditions. For each, the thing to know is what evidence would reveal it — because the failure is invisible by construction to the metrics most teams monitor.

### 4.1 The top-k cut — the losing evidence is never retrieved

The single most consequential limit in the guide: **a conflict you never retrieve is a conflict you cannot detect, and no reranking of a truncated candidate set fixes that.**

Retrieval returns `k` passages. If both sides of a conflict are in the corpus and only one of them fits inside `k` — or if the two sides sit at ranks 4 and 40 under a top-10 cut — then the context window is internally consistent and confidently wrong. The model cannot report a disagreement it was not shown. Every downstream mechanism in this guide, including the authority register in [§7](#7-authority-and-entitlement--the-enterprise-mechanism), operates on the retrieved set; none of them can recover a source that was cut before the prompt was built.

*What would reveal it:* recall measurement on queries where a conflict is known to exist, with the *intended* losing source labelled as expected evidence. A retrieval metric computed without such labels cannot see this failure at all, because it scores the retrieved set against what the query asked for and never against what the query *should also have surfaced*. This is a labelling problem, and [§5](#5-detecting-that-a-conflict-exists-at-all) treats it as one.

### 4.2 Chunking — the contradiction is split across chunks

A passage that states a rule and a passage that revises it may each be chunked into fragments, and the fragments that best match the query may be the two *non-conflicting halves*. The conflicting sentences land in chunks that did not clear the relevance bar, or the boundary falls between the assertion and its qualifying sentence, so a conditional rule is retrieved as an unconditional one.

This is a fragmentation failure rather than a ranking failure, and it is why chunking is a conflict-relevant decision and not merely a retrieval-quality one. Overlap mitigates the boundary case and does nothing for the relevance-case; semantic chunking can keep an assertion with its qualification, or can split exactly there, depending on the splitter's notion of topic. *What would reveal it:* chunk-level provenance with enough structure to reassemble the parent document, and a comparison stage that can ask whether two chunks came from the same parent — which is cheap if parent references are carried and impossible if they are not.

### 4.3 The reranker — the minority view is demoted

A cross-encoder reranker rescores the candidate set against the query and reorders it (Nogueira and Cho, *Passage Re-ranking with BERT*, 2019, [arXiv:1901.04085](https://arxiv.org/abs/1901.04085)). Its objective is query relevance, and relevance is not authority, recency or correctness. A reranker trained and tuned on relevance can therefore do something perverse with precision: promote the fluent, well-matched, superseded summary, and demote the terse authoritative instrument — and, if the cut follows the reranking, push the losing side below the `k` threshold entirely.

This is the mechanism by which a *quality* improvement makes a conflict-related failure more likely. It is not a bug in the reranker; it is the reranker doing precisely the job it was given, on an objective that does not contain the institution's notion of authority. *What would reveal it:* diffing the pre-rerank and post-rerank order for queries where a known conflict exists, and specifically checking whether the authoritative-but-terse side survives the cut. A reranker evaluation that scores only relevance will show an improvement.

### 4.4 Context position — retrieved, present, ignored

Lost in the Middle (Liu et al., 2023, [arXiv:2307.03172](https://arxiv.org/abs/2307.03172)) documents the positional effect: model performance in long-context settings is highest when the relevant information is near the beginning or the end of the supplied context, and degrades when it sits in the middle. Cuconasu et al. ([arXiv:2401.14887](https://arxiv.org/abs/2401.14887)) report a related dependence on where relevant documents land in the ranking.

For conflict resolution the implication is sharp: **the losing side can be retrieved, in the context window, and still ignored.** The conflict is present in the prompt; attention does not distribute to it as it distributes to the winning side, and the model produces an answer consistent with the emphasized source without ever engaging the competing passage. There is no error to point at — the retrieval metrics were perfect, the context was assembled correctly, and the answer is still one-sided. *What would reveal it:* placement differentials — deliberately swapping the order of the two sides on a known-conflict query and observing whether the answer flips. If flipping the order flips the conclusion, the resolution is being determined by position rather than by any property of the sources. That experiment is nearly free and is run far too rarely.

### 4.5 The unstated silent choice

The last mechanism is not a component but a behaviour: **the model resolves a conflict without disclosing that one existed.** Handed two incompatible passages and asked a question, the model answers. It does not typically say "the retrieved sources disagree; I am following passage 2 because it is dated later." It picks, silently, using whatever internal disposition the situation happens to evoke — and per [§3](#3-what-the-research-actually-shows-about-model-behaviour), that disposition is model-dependent and condition-dependent.

This is the failure that makes the whole class of problem hard to operate. A pipeline that retrieves one wrong document produces a wrong answer, and the wrongness is at least *single-sourced* and traceable. A pipeline that retrieves two contradictory documents and answers confidently produces an answer whose evidentiary basis is unknowable from the output. *What would reveal it:* requiring the model to state, for every claim, which retrieved passage supports it — the citation discipline whose automatic evaluation is the subject of ALCE (Gao et al., *Enabling Large Language Models to Generate Text with Citations*, 2023, [arXiv:2305.14627](https://arxiv.org/abs/2305.14627)) — and then checking whether the cited support set is internally consistent. An answer citing both sides of a contradiction while asserting one of them is the signature of the silent choice.

### 4.6 The informational limit, stated once

- Conflicts are only visible downstream of the **retrieval cut**, and the cut is where they die.
- Reranking is an ordering of what survived the cut; it cannot restore what did not.
- Position effects mean a present conflict can still be an unattended one.
- A silent resolution is indistinguishable, from the outside, from an answer that was never in conflict.

Any design that addresses conflict must therefore invest first in **recall of the competing sources and metadata that makes them comparable** — not in a cleverer prompt. The prompt is the part of the stack that is downstream of every one of the five mechanisms above.

---

## 5. Detecting That a Conflict Exists At All

Detection precedes resolution, and it comes in two families: **metadata signals**, which flag a probable conflict before any model is involved, and **content signals**, which compare passages for contradiction. The ordering matters — run metadata first, because it is cheaper, more reliable and more explainable, and because a flagged conflict can then be examined with the content machinery aimed at the right pair instead of every pair.

### 5.1 Metadata first — the cheapest detector and the most often omitted

A conflict between two retrieved passages is frequently *predictable from their metadata alone*, without reading either one. Given each retrieved chunk as a record, the following fields turn an unknown into a known:

| Field | What it pre-discloses | Conflict kind it flags |
| --- | --- | --- |
| Document version / supersession status | One passage is retracted, the other current | Version ([§2.2](#22-the-enterprise-inventory) #1) |
| Effective date and, where applicable, expiry | The two passages were never simultaneously true | Temporal (#6) |
| Source register ID and authority tier | The two passages do not have equal standing | Authority (#2) |
| Jurisdiction or regulatory perimeter | The two passages answer to different regimes | Jurisdictional (#3) |
| Owning function and approval status | One passage is approved, the other is draft | Authority (#2) |
| Term/definition registry key | The passages define the same term differently | Terminology (#4) |
| Unit and scale of embedded quantities | The passages cannot be compared as-is | Unit (#5) |
| Parent document ID | The passages came from the same instrument | Chunking artefact ([§4.2](#42-chunking--the-contradiction-is-split-across-chunks)) |

Two properties make this the right first line. It is **cheap** — a field comparison per retrieved record, no model call. And it is **explainable** — "the answer cited v3 of the procedure, which was superseded on 14 March" is a sentence an auditor can act on, whereas "the model appeared more confident in passage 2" is not.

The failure is almost always one of omission rather than engineering: the metadata exists in the source system and is thrown away at ingest, so the index holds text without the fields that would make conflict detectable. Carrying provenance through the ingestion pipeline is a smaller job than any of the model-side mechanisms in [§6](#6-the-resolution-ladder), and it is the precondition for all of them. See [§9.1](#91-the-authority-annotated-store) for the store design.

### 5.2 Content signals — contradiction and entailment across passages

Where metadata does not settle it, the passages must be compared for *incompatibility*. The discipline to borrow is claim verification: take one retrieved passage, decompose it into claims, and test whether those claims are entailed, contradicted, or neither by the other passage. FEVER (Thorne et al., *FEVER: a large-scale dataset for Fact Extraction and VERification*, 2018, [arXiv:1803.05355](https://arxiv.org/abs/1803.05355)) is the reference dataset for this task shape — claim in, verdict out, with support and refutation as labelled outcomes — and it is useful here for the shape of the problem rather than as a benchmark to hit. Detection does not need a verdict-grade classifier to be valuable: a flag that says "these two passages are unlikely to be jointly true" is enough to route the pair to [§6](#6-the-resolution-ladder).

Three practical cautions. First, **negation and modality are where this breaks** — "must be retained for five years" against "may be retained for five years", or "the exception does not apply" against "the exception applies". An entailment model that scores both pairs as merely *not entailed* has failed to detect a real conflict. Second, **numeric incompatibility is not textual contradiction** — "1.2 million" and "SGD 1.2 million" are lexically near-identical and materially different, so quantity-and-unit extraction must run alongside the entailment check, not after it. Third, **cost scales quadratically** with the number of passages, so pair selection must be narrowed: compare across metadata groups (different version, different tier, different jurisdiction) and within-topic clusters rather than all-pairs.

### 5.3 Clustering comparable passages before comparing them

Comparing every passage with every other is both expensive and mostly wasted, because most pairs are on different subjects and cannot conflict. The efficient shape is three steps:

1. **Cluster** the retrieved passages into topic groups — by shared entities, by cited instrument, by declared subject code, or by embedding proximity within the retrieved set.
2. **Compare within clusters**, where a contradiction is meaningful, using the entailment and quantity checks above.
3. **Compare across metadata groups explicitly** — that is, deliberately pair the current version against the superseded one, the policy against the procedure, the jurisdiction-A instrument against the jurisdiction-B instrument, since these are the conflict-generating pairings and they may not share topic clusters after chunking.

Step 3 is the one teams skip, and it is the one the enterprise inventory demands: the conflicts in [§2.2](#22-the-enterprise-inventory) are overwhelmingly *cross-group* pairings that topic clustering will not surface, because a policy and its summary are written to look alike.

### 5.4 Reading the detection signal without over-trusting it

Detection is a **triage** mechanism, not an adjudicator. The output of this section is a set of pairs flagged as *possibly in conflict*, with the metadata evidence attached. What follows — which source speaks, whether to abstain, whether to escalate — is [§6](#6-the-resolution-ladder) and [§7](#7-authority-and-entitlement--the-enterprise-mechanism).

The honest limits: entailment detection on long procedural text is noisy; a contradiction expressed across differently structured sentences is easy to miss; and a conflict between a *retrieved* passage and the *model's parametric belief* — the context–memory family from [§2.1](#21-what-the-literature-actually-classifies) — is not detectable by passage comparison at all, because one side is not a passage. That last point is a further argument for the authority mechanism: it operates on sources you can see.

---

## 6. The Resolution Ladder

Six rungs, ordered by **honesty** — by how faithfully the resolution reflects something true about the sources — and explicitly *not* by cleverness. Rung 1 is the least clever and the most often correct. Rung 5 is the most technically impressive and, for an enterprise, the weakest.

### 6.1 Rung 1 — Do not resolve it; surface it

Present the disagreement, with its sources and their provenance, and let the reader decide. This is often the correct behaviour in a regulated setting, and it is the only available behaviour for the seventh kind in [§2.2](#22-the-enterprise-inventory) — genuine factual disagreement.

*Why it is first:* it asserts nothing beyond what is known. It is the only rung that cannot be wrong about *which source wins*, because it does not claim a winner. It is also the rung that preserves the human's information: a reader told "these two procedures differ on retention" can act; a reader told one procedure's retention period cannot.

*Failure mode:* it pushes work to the user and, done badly, produces a wall of quoted text with no synthesis — an unhelpful answer with a clean conscience. Surfacing must be *structured*: state the question, state that the sources conflict, name the specific point of conflict, cite each side, and state what would resolve it. A refusal to choose is not the same as a refusal to be useful.

### 6.2 Rung 2 — Authority and entitlement

Resolve by *which source has the right to speak* on the subject. A regulator's instrument outranks an internal policy; a policy outranks a procedure; a procedure outranks a wiki page; a wiki page outranks the open web. This is [§7](#7-authority-and-entitlement--the-enterprise-mechanism), the centre of this guide, and for the enterprise it is the rung that does the real work.

*Why it is second:* it is the earliest rung whose resolution reflects a property of the *sources* rather than a property of the *text* or the *model*. It is auditable, because the ordering is a written artefact with owners.

*Failure mode:* it is only as good as the register behind it. Applied to sources that have not been classified, it degrades into an improvised hierarchy — usually "the one that looks more official" — which is rung 5 wearing rung 2's clothes. An authority tier assigned on the day it is needed is not an authority tier.

### 6.3 Rung 3 — Version and recency

Where the sources are the same instrument at different points, the current version speaks, and the superseded one speaks only as history. This is close to rung 2 in kind — it is a governance ordering, not a judgement — and it is separated because recency is a different ordering from authority: a newer wiki page does not outrank an older instrument, which is precisely why recency alone is not a conflict strategy.

*Failure mode:* recency-first applied universally inverts authority, promoting the newest thing over the most entitled thing. And the version question is not always "which is newer" — it is often "which was in force *then*", which is [§8](#8-temporal-conflict-and-the-point-in-time-question).

### 6.4 Rung 4 — Corroboration and provenance

Where no ordering exists, count what agrees: if five retrieved passages assert one figure and one asserts another, the majority is the better bet. Provenance sharpens it — five passages derived from one originating document are one source, not five.

**The explicit warning, which is the reason this rung sits above the model-side rungs and not at the top: corroboration counts popularity, not truth.** A widely repeated error remains an error; a fifth restatement of a wrong figure is not evidence about the figure. Corroboration is a tie-breaker of last resort among sources of equal and unknown authority, and it is actively dangerous when the corpus contains a dominant summary that many documents paraphrase — at which point the majority *is* the derivation chain, and it will confidently outvote the instrument.

*Failure mode:* mistaking volume for verification, and mistaking independent agreement for derived agreement. Provenance analysis must strip the derivation chain before the count means anything.

### 6.5 Rung 5 — Model-side mechanisms

The techniques: **cite-and-choose** (require the model to attribute each claim to a passage and select on the attribution), **self-consistency** (sample multiple reasoning paths and take the majority answer; Wang et al., *Self-Consistency Improves Chain of Thought Reasoning in Language Models*, 2022, [arXiv:2203.11171](https://arxiv.org/abs/2203.11171)), **multi-agent debate** (Du et al., *Improving Factuality and Reasoning in Language Models through Multiagent Debate*, 2023, [arXiv:2305.14325](https://arxiv.org/abs/2305.14325)), **self-critique and reflection** (Asai et al., *Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection*, 2023, [arXiv:2310.11511](https://arxiv.org/abs/2310.11511)), **verification loops** (Dhuliawala et al., *Chain-of-Verification Reduces Hallucination in Large Language Models*, 2023, [arXiv:2309.11495](https://arxiv.org/abs/2309.11495)), and **corrective retrieval** (Yan et al., *Corrective Retrieval Augmented Generation*, 2024, [arXiv:2401.15884](https://arxiv.org/abs/2401.15884), which evaluates retrieved material and triggers corrective action when it is judged inadequate). The critique-loop family is owned by [Feedback Mechanisms for LLM Agents](../feedback_mechanisms_llm_agents_guide.md) and is referenced, not re-derived, here.

**What these can establish.** They can improve consistency, reduce the chance of a hallucinated claim, and produce an attribution that makes an answer checkable. Cite-and-choose, in particular, produces the audit trail that [§11](#11-the-regulated-institution-angle) requires.

**What they cannot establish.** They cannot establish authority. Every one of these mechanisms operates on the *text* and on the *model's dispositions*, and none of them has access to the fact that one of the passages is a superseded draft. A debate between two model instances over two conflicting passages is a debate over which passage reads more convincingly. A self-consistency vote over a context containing a conflict resolves the conflict *in favour of whichever source the model is already predisposed to follow* — and per [§3](#3-what-the-research-actually-shows-about-model-behaviour), that predisposition varies by model, evidence strength and phrasing.

**The specific failure to name:** a model asked to pick a winner between two conflicting sources tends to prefer **the most fluent, most specific and most contextually salient** source. Fluency and specificity are properties of writing quality. The fluent passage is frequently the internal summary — the one written to be read — and the authoritative passage is frequently the terse instrument whose phrasing is legal and dry. Rung 5, used alone, systematically selects against authority. This is the single most important sentence in this section.

**The verdict:** model-side mechanisms are the **weakest rung** for an enterprise and must not be the only one. They are appropriate as a *supplement* — improving attribution, flagging their own uncertainty, generating the comparison a human reads — and inappropriate as the *basis* of resolution. A pipeline whose only conflict mechanism is a prompt instruction for the model to prefer good sources has no conflict mechanism.

### 6.6 Rung 6 — Abstain and escalate to a human

Where the conflict is material, unresolvable by ordering, and the consequence of a wrong answer is high, the correct output is to decline to resolve and route to a human owner. Abstention is not a failure state here; it is the designed outcome. The lens for this is **sufficient context**: Joren et al. (*Sufficient Context: A New Lens on Retrieval Augmented Generation Systems*, 2024, [arXiv:2411.06037](https://arxiv.org/abs/2411.06037)) frame the question as whether the retrieved context is sufficient to answer at all, which is the right question to ask before any resolution is attempted — if the context is insufficient (or contradictory, which is a specific kind of insufficiency), abstention is the correct answer rather than a degraded one. Negative rejection as a distinct capability is also one of the four RGB abilities that Chen et al. ([arXiv:2309.01431](https://arxiv.org/abs/2309.01431)) report as weak, which is why abstention must be *engineered* rather than assumed.

*Failure mode:* abstention as a blanket policy destroys utility — a system that declines whenever two sources differ, including on immaterial points, is unusable and will be routed around. The design decision is therefore not *whether* to abstain but *where* ([§12](#12-the-cost-and-performance-angle)) and *on whose authority the escalation goes*.

### 6.7 The ladder as a discipline

The rungs are ordered and should be tried in order; skipping to rung 5 is the most common enterprise error because it is the most available. The rule to carry:

> **Try the ordering rungs before the judgement rungs.** Metadata before entailment. Authority before fluency. Governance before generation.

---

## 7. Authority and Entitlement — the Enterprise Mechanism

This is the centre of the guide. Everything above is the problem; this is the mechanism.

### 7.1 The ordering

An institution that can answer from its own sources must have a written answer to *which source speaks* on a subject. The reference ordering, to be adapted rather than adopted:

| Tier | Source class | Entitlement |
| --- | --- | --- |
| A1 | Regulator's instrument — legislation, rules, published guidance | Speaks with the force of the regime; overrides everything internal |
| A2 | Contractual or external obligation the institution is bound by | Speaks to the institution's own obligations |
| A3 | Internal policy approved by the accountable committee | Speaks for the institution, within the regime |
| A4 | Procedure or standard operating instruction implementing a policy | Speaks to *how*, subordinate to A3 |
| A5 | Internal wiki, runbook, FAQ, training material | Speaks as guidance only; the most fluent tier and the least entitled |
| A6 | Open web, vendor documentation, third-party commentary | Speaks as external context; never as the institution's answer |
| — | **Superseded version of any tier** | **Outranks nothing at all.** A superseded A1 is not a weak A1; it is a historical object |

The final row is the one that gets mishandled. A superseded version is not a lower-relevance document — it is a *non-authoritative* document, and it retains enough textual similarity to the current one that every similarity-based mechanism will happily rank it near the top. Treating a superseded instrument as "merely less relevant" is one of the anti-patterns in [§14](#14-the-anti-patterns), and it is the failure the worked example turns on.

Note also what the ordering is *not*: it is not a global ranking to be applied to every query. It is a **subject-scoped entitlement**. The regulator speaks on the rule; it does not speak on Cymbal Bank's internal escalation phone tree. Authority is a property of a source *on a subject*, which is why the register below is keyed by (source, subject) and not by source alone.

### 7.2 The source register

The artefact that makes the ordering operational is a **source register**: the inventory of everything the system is entitled to answer from, each entry carrying the fields that make authority retrievable.

Minimum viable entry:

- **Source ID** and human-readable name.
- **Subject scope** — what this source has the right to speak about. The entitlement is scoped; an unscoped tier is a blank cheque.
- **Authority tier** — from the table above, assigned by the accountable owner, not inferred at query time.
- **Owner** — the named function accountable for the content and the tier.
- **Jurisdiction / perimeter** — which regime it answers within.
- **Lifecycle state** — current, superseded, withdrawn, draft. The field that prevents version conflict.
- **Effective dates** — from and (where applicable) to.
- **Version identifier** and a link to the parent document.
- **Ingestion provenance** — which pipeline ingested it, when, under what transformation.

The register should also record the **derivation graph**: which sources are summaries of which. This is what makes rung 4 of the ladder ([§6.4](#64-rung-4--corroboration-and-provenance)) honest, because corroboration counts independent agreement, and a register that knows five wiki pages derive from one policy can collapse them to one source before counting. Governance material in this repository covers register-style ownership and data stewardship in more depth than this guide should — see [Data Governance Framework](../../technology/data/data_governance_framework.md) and [Data Compliance Frameworks](../../technology/data/data_compliance_frameworks.md).

### 7.3 The retrieval consequences

Authority is useless if it is a document nobody queries. The register must reach retrieval in three ways.

**1. Filter before you rank.** Where the query has a known subject, jurisdiction and point in time, retrieve *within the entitled subset* first — current versions, correct jurisdiction, at the right effective date — and only widen if the entitled subset does not contain an answer. This is metadata-first filtering, and for version and jurisdiction conflicts it resolves the conflict by never retrieving the loser. It is the cheapest resolution available and the one that most directly addresses [§4.1](#41-the-top-k-cut--the-losing-evidence-is-never-retrieved).

**2. Boost within a tier, not across tiers.** After the cut, a tier-aware rerank should keep the entitled side above the threshold rather than relying on the model to prefer it. The crucial restriction is *within* a tier: comparing across tiers on a similarity score is precisely how a well-written A5 document beats an A1 instrument. Cross-tier comparison should be a comparison of entitlement, not of text.

**3. Carry the tier into the prompt, as a label.** When both sides of a conflict are deliberately presented ([§9.3](#93-the-conflict-aware-prompt)), each passage is presented with its source, its tier, its version and its date. The model is not asked to infer authority from prose quality; it is told the entitlement and asked to attribute. This converts an inference the model is bad at into a lookup it can perform.

### 7.4 Version-pinning discipline

A retrieval system that answers questions about obligations must be able to answer *as of a date*. Version-pinning means: the query carries (or defaults to) a point in time, the entitled subset is computed for that point in time, and the answer states the as-of date. This is the same discipline as point-in-time correctness in data management, and [§8](#8-temporal-conflict-and-the-point-in-time-question) cross-references the repository's material on it rather than re-deriving it. The conflict-relevant consequence: **a retrieval system without version pinning cannot distinguish "the document changed" from "the document contradicts itself", and will conflate a version conflict with a factual one.**

### 7.5 The honest point

**This is a governance artefact that must be owned, not a prompt instruction.**

A prompt that says "prefer authoritative sources" is a hope, not a control. It has no register behind it, so the model's notion of "authoritative" is whatever salience the passage carries — the failure mode named in [§6.5](#65-rung-5--model-side-mechanisms). It cannot be audited, because there is nothing to inspect but a sentence of instruction. It cannot be versioned, so a change in the institution's source hierarchy does not reach the system. It cannot be tested, because "did the model prefer the authoritative source" has no ground truth without a register to define authoritative.

Every one of those properties is fixed by making authority data. The register is work — subject scoping, ownership assignment, tier adjudication between functions that disagree about who outranks whom — and it is *governance* work, which is why it is chronically deferred in favour of the prompt, which is a paragraph of writing. The deferral is the reason enterprise conflict handling is weak. **The mechanism is a spreadsheet with owners; the prompt is the decoration on top.**

---

## 8. Temporal Conflict and the Point-in-Time Question

### 8.1 The two directions of one problem

Two questions that look different:

- **A version conflict:** the same instruction exists in two versions, and both are retrievable.
- **A point-in-time question:** what was the instruction in force on the date of the event?

They are the same problem seen from two directions. The first is asked at ingestion time ("which of these should be live?") and the second at query time ("which of these was live *then*?"). Both are answered by the same metadata — lifecycle state and effective dates — and both are failed by the same omission, which is an index that holds text without dates.

The repository already owns point-in-time correctness as a data-management discipline, and this guide cross-references rather than re-derives it: see [Market Data Integrity](../../banking/market_data_integrity_guide.md) for point-in-time in the market-data setting, and [Data Pipeline Versioning](../../technology/data/data_pipeline_versioning.md) for versioning mechanics in pipelines. The temporal-generalisation line of work — Dhingra et al., *Time-Aware Language Models as Temporal Knowledge Bases*, 2021, [arXiv:2106.15110](https://arxiv.org/abs/2106.15110) — is the reference for the model-side question of whether a model can be made to answer *as of* a time rather than in the present; it is relevant because it establishes the problem of temporal knowledge as a research subject distinct from retrieval, and because it is a caution against assuming the model can perform the as-of computation internally when it has not been trained for it.

### 8.2 Wrong versus right-for-its-date

The distinction to hold: **a superseded document is not simply wrong; it is right-for-its-date.** This matters for three reasons.

1. **It changes the correct answer to historical questions.** "What retention period applied to this record in 2023" has a correct answer, and it is the 2023 procedure, which the current-version filter would hide. A system that aggressively excludes superseded documents answers the *present* question well and the *historical* question wrongly.
2. **It makes deletion the wrong default in some estates.** The corpus hygiene fix in [§13](#13-worked-example--cymbal-bank-two-procedures-and-one-regulation) is to remove the superseded procedure from the *answerable* set, not necessarily from the *archive*. The document must remain retrievable for audit and for historical questions; it must not be retrievable as though it were current.
3. **It reframes the conflict.** A version conflict is not a factual dispute between two documents. It is a lifecycle failure, and it is resolved by knowing which was in force, not by judging which reads better.

### 8.3 The regulated consequence

Here is where the enterprise problem becomes sharp, and it is the point [§11](#11-the-regulated-institution-angle) develops: **a bank's answer can be simultaneously correct and non-compliant.** An answer drawn faithfully from a superseded procedure is *correct against its source* and *wrong against the institution's obligation* at the time it was given. No amount of model quality fixes this, because the model did exactly what it was asked: it answered from the evidence it was given. The failure is upstream — in the index, in the lifecycle, in the absence of an entitled subset — which is why the fix in the worked example is corpus hygiene rather than a better prompt.

The regulatory-version dimension compounds this. Regulators revise instruments; internal policies lag; procedures implement both. A query that lands between a revised instrument and the not-yet-updated procedure will retrieve a set that is internally coherent and collectively obsolete. Temporal conflict is therefore not a niche retrieval concern but a *compliance* concern, and the repository's compliance and regulatory material ([AI and GenAI Banking Compliance](../../banking/ai_genai_banking_compliance_guide.md) and the banking shelf, cross-referenced in [§11.6](#116-cross-references)) is where the obligation to answer from the current instrument lives.

### 8.4 What to do

- **Carry effective dates on every chunk**, from and to, and a lifecycle state.
- **Default the query to the institution's present** unless the user asks otherwise, and make the as-of date visible in the answer.
- **Filter by as-of date before ranking**, so the entitled subset is temporally correct by construction ([§7.3](#73-the-retrieval-consequences)).
- **Preserve the archive separately** from the answerable set — same texts, different permission.
- **Pin the version in the answer**, so that "which procedure did this answer come from" is answerable after the fact. This is the audit requirement of [§11](#11-the-regulated-institution-angle) in its temporal form.

---

## 9. The Design Patterns

Concrete and implementable, each with what it costs and — importantly — what it does **not** fix. These are patterns for the retrieval-time conflict problem specifically; the general technique landscape these sit beside — the naive/advanced/modular organisation familiar from the RAG survey literature (Gao et al., *Retrieval-Augmented Generation for Large Language Models: A Survey*, 2023, [arXiv:2312.10997](https://arxiv.org/abs/2312.10997)) — is [Advanced RAG Techniques](advanced_rag_techniques_guide.md)'s subject and is not re-derived here.

### 9.1 The authority-annotated store

**What it is.** Every chunk carries its provenance as first-class metadata: source ID, authority tier, subject scope, jurisdiction, lifecycle state, effective dates, version, parent document, and derivation lineage. The store's schema makes these queryable and filterable, not merely stored.

**What it costs.** Ingestion work — you must capture the fields at the point where they are known, which is the source system, and preserve them through chunking. An ownership exercise to assign tiers and subjects. Ongoing maintenance when sources change, which is real recurring cost because a stale register is worse than none: it produces confident wrong answers with a governance trace.

**What it does not fix.** It does not resolve anything by itself. It is an enabler: it makes [§5.1](#51-metadata-first--the-cheapest-detector-and-the-most-often-omitted) detection possible, [§7.3](#73-the-retrieval-consequences) filtering possible, and [§9.3](#93-the-conflict-aware-prompt) presentation honest. A register with no filtering and no presentation is documentation, not a control.

### 9.2 Metadata-first filtering — retrieve within a tier before comparing across tiers

**What it is.** Build the query's entitled subset first: current versions, relevant jurisdiction, correct as-of date, and the subject's highest applicable tiers. Retrieve within it. Widen only if the subset yields nothing, and say so when you widen.

**What it costs.** Recall risk, which is the real and serious cost. Scoping too tightly — a wrong subject code, a jurisdiction mis-inference, an as-of date parsed wrongly from the question — returns an empty or thin subset for questions the wider corpus would have answered. This demands a fallback path and a visible widening statement, not a silent widening.

**What it does not fix.** It does not fix conflicts *within* a tier, which are real: two current A3 policies can genuinely contradict each other, and no filter resolves that. Nor does it fix the case where the entitled subset is empty because the corpus lacks the source — that is a coverage problem, and it will present as abstention.

### 9.3 The conflict-aware prompt

**What it is.** When detection has flagged a conflict, the prompt *presents both sides with provenance* rather than asking for a winner. The passage set includes each side labelled with its source, tier, version and date, and the task is framed as "state what each source says and which has the right to speak, citing both" — not "choose the correct source".

**What it costs.** Context budget: two sources where one would do, plus the provenance labels. Latency, since the answer is longer. And a change in output shape that user-facing products may resist, because a disclosed conflict is a less satisfying answer than a confident one.

**What it does not fix.** It does not make the model a reliable authority adjudicator. The framing reduces the fluency bias described in [§6.5](#65-rung-5--model-side-mechanisms) by supplying entitlement as a label rather than leaving it to be inferred, but it does not eliminate the model's preference for salient text, and it cannot surface a conflict that detection never flagged. The prompt is downstream of detection; it is never the detector.

### 9.4 Retrieving both sides deliberately

**What it is.** For known conflict-prone subjects — retention periods, thresholds, escalation paths, definitions — issue a second, targeted retrieval aimed at the *competing* source class: fetch the instrument alongside the summary, the current version alongside the superseded, the jurisdiction-B rule alongside jurisdiction-A's. Do not rely on a single relevance-ranked query to have surfaced both.

**What it costs.** Extra retrieval calls and extra context. It requires knowing which subjects are conflict-prone, which comes from incident history rather than from theory — a bank learns its conflict subjects by being wrong about them.

**What it does not fix.** It does not scale to unknown conflicts: deliberate competitor retrieval only finds conflicts you anticipated. It is a targeted mitigation for the [§4.1](#41-the-top-k-cut--the-losing-evidence-is-never-retrieved) cut, not a general recall guarantee.

### 9.5 Cite-or-abstain

**What it is.** Every asserted claim carries a citation to a retrieved passage, and if no retrieved passage supports the claim, the system abstains rather than asserting. Evaluated in the citation-quality terms of ALCE (Gao et al., [arXiv:2305.14627](https://arxiv.org/abs/2305.14627)) — citation precision and recall as the measures of whether the attributions are the right ones. The sufficient-context lens (Joren et al., [arXiv:2411.06037](https://arxiv.org/abs/2411.06037)) supplies the abstention half: if the context cannot support the answer, the answer is not given.

**What it costs.** Citation adds generation overhead and makes answers wordier. Strict abstention produces more non-answers, and the acceptable rate is estate-dependent ([§12](#12-the-cost-and-performance-angle)). It also requires the citation to be *checked* rather than produced — an unverified citation is worse than none, because it manufactures an audit trail that does not exist.

**What it does not fix.** It does not resolve conflicts. An answer can cite both sides of a contradiction and still assert one; the citation discipline makes that visible, which is valuable, but it is the visibility and not the resolution. It also does not detect a conflict between a cited passage and an uncited parametric belief.

### 9.6 The escalation path

**What it is.** A defined route for unresolvable or material conflicts to reach a human owner — the register's owner for the subject — with the conflict, the sources and the evidence of the conflict attached. The escalation is a workflow, with a queue, an owner and an SLA, not a message to a support inbox.

**What it costs.** Operations. A queue requires staffing, and the escalations that arrive are precisely the ones that need judgement, so they cannot be triaged by a script. It also creates an obligation: a raised conflict that is never resolved leaves the corpus conflicted.

**What it does not fix.** It does not scale to high conflict volume, and a system with a high conflict rate is really telling you that its corpus lifecycle is broken — the escalation path is a safety net, not a remediation. If escalations are frequent, the finding is [§13](#13-worked-example--cymbal-bank-two-procedures-and-one-regulation)'s: the fix is hygiene.

### 9.7 How the patterns compose

| Pattern | Resolves | Enables | Chief cost |
| --- | --- | --- | --- |
| Authority-annotated store ([§9.1](#91-the-authority-annotated-store)) | Nothing directly | Every other pattern | Ingestion + ongoing ownership |
| Metadata-first filtering ([§9.2](#92-metadata-first-filtering--retrieve-within-a-tier-before-comparing-across-tiers)) | Version, jurisdiction, temporal | Cheaper ranking | Recall, when scoped wrongly |
| Conflict-aware prompt ([§9.3](#93-the-conflict-aware-prompt)) | Nothing; it discloses | Auditability | Context + latency + output shape |
| Deliberate competitor retrieval ([§9.4](#94-retrieving-both-sides-deliberately)) | Anticipated conflicts | Detection coverage | Extra calls; only anticipates |
| Cite-or-abstain ([§9.5](#95-cite-or-abstain)) | Nothing; it evidences | Audit + honest silence | Abstention rate; unchecked citations |
| Escalation path ([§9.6](#96-the-escalation-path)) | Material conflicts, by a human | Accountability | Ops load; unresolved queues |

The composition matters more than any single pattern. The store enables detection; detection routes to filtering (resolvable) or to the prompt (disclosable) or to escalation (material); cite-or-abstain makes the outcome auditable. Nothing in the set asks the model to decide what the institution has decided.

---

## 10. The Evaluation Question

Evaluation methodology and tooling are owned elsewhere in this repository — [RAG Evaluation Methodology](rag_evaluation_methodology_guide.md) for metric definitions and the golden-set method, [RAG Evaluation Tools Comparison](rag_evaluation_tools_comparison_guide.md) for the tooling. This section does not re-derive them. It states what the conflict benchmarks actually test, what to borrow, and the limit that matters.

### 10.1 What the conflict benchmarks test

**ConflictBank** (Su et al., [arXiv:2408.12076](https://arxiv.org/abs/2408.12076)) tests model behaviour on conflicts produced by its own construction framework across retrieved-knowledge, encoded-knowledge and interplay settings, and across stated conflict causes including temporal discrepancy and semantic divergence. **RGB** (Chen et al., [arXiv:2309.01431](https://arxiv.org/abs/2309.01431)) tests counterfactual robustness among four abilities, with injected counterfactual context. **ClashEval** (Wu et al., [arXiv:2404.10198](https://arxiv.org/abs/2404.10198)) tests the prior-versus-evidence tug-of-war. Each measures model disposition under a controlled conflict, which is a genuine and useful subject.

Their honest limit, and the reason a score on them is not evidence about a deployment: **the conflicts are constructed.** A constructed conflict is placed in front of the model on purpose, usually with the competing source clearly present and the question shaped to force a choice. A production conflict arrives by accident, may be missing one side from the context entirely ([§4.1](#41-the-top-k-cut--the-losing-evidence-is-never-retrieved)), is typically undeclared ([§4.5](#45-the-unstated-silent-choice)), and carries no label saying that the passage is superseded. The benchmark tests the model's behaviour in a situation your pipeline may never create.

### 10.2 What to borrow, by name

- **From [RAG Evaluation Methodology](rag_evaluation_methodology_guide.md):** the golden-set method applied to *conflict* cases specifically — a set of queries where a conflict is known to exist, each labelled with the entitled answer, the competing source, and the expected disposition (resolve to source X / disclose / abstain). Without such labels, no conflict evaluation is possible, because there is no way to know whether an answer is wrong or merely unsatisfying.
- **From [RAG Evaluation Tools Comparison](rag_evaluation_tools_comparison_guide.md):** the harnesses to run it; this guide does not name tools.
- **From ALCE** (Gao et al., [arXiv:2305.14627](https://arxiv.org/abs/2305.14627)): citation precision and recall as the measures that make the attribution checkable — did the answer cite the passage that supports it, and was the citation warranted.
- **From RGB's four abilities** (Chen et al., [arXiv:2309.01431](https://arxiv.org/abs/2309.01431)): negative rejection and counterfactual robustness as *shapes* of test to replicate on your own corpus, since they correspond directly to "does it decline when it should" and "does it resist a wrong-but-present source".
- **From RAGTruth** (Niu et al., [arXiv:2401.00396](https://arxiv.org/abs/2401.00396)): the practice of span-level annotation of unsupported output, which is the natural labelling scheme for the conflict-specific case — "the answer asserted something no retrieved passage supports" — as applied to answers produced over a contradictory context.
- **From [RAG Benchmarks](rag_benchmarks_guide.md):** the discipline of reading a benchmark claim — which corpus, which construction, which metric — which is exactly the discipline needed when someone offers a conflict-benchmark number as deployment evidence.

No metric invented for this guide is proposed. Where a measurement is described above it is the golden-set method of the evaluation guide or a metric named in a cited paper.

### 10.3 The limit to state to stakeholders

**A system can score well on a conflict benchmark and still answer from a superseded source in production.** The benchmark conflict is visible to the model; the production conflict may be a single retrieved passage whose competing version was cut by top-k, is a version whose supersession is recorded in a source system the index never ingested, or is a summary whose source instrument was never retrieved at all. Every one of those is invisible in a benchmark result, because the benchmark *never had the hidden-side problem* — it constructed both sides.

The corollary: **conflict evaluation is primarily an evaluation of the pipeline, not the model.** Measure whether the entitled source was retrieved, whether the superseded source was excluded, whether the citation names the entitled source, and whether the abstention fired where it should. Those are pipeline properties, testable against a labelled golden set of conflict cases, and they are the measurements that would have caught the failure in [§13](#13-worked-example--cymbal-bank-two-procedures-and-one-regulation).

---

## 11. The Regulated-Institution Angle

Here conflict resolution stops being a feature and becomes a **control**.

### 11.1 Answering from a superseded policy is a compliance failure, not a model error

The reframing is the section's point. When an assistant answers a procedure question from a retracted procedure, the incident is not "the model chose badly". The model answered from the evidence in front of it, correctly, as instructed. The incident is that **the institution's answerable corpus contained a document the institution had withdrawn**, and the institution's system surfaced it as current. Framing that as a model error sends the remediation to the wrong place — to prompt engineering — while the corpus remains broken.

The control framing follows: the withdrawn document should have been in a state that the retrieval layer could not serve as current. That is a lifecycle control with a testable property ("no withdrawn document is retrievable as current"), which is what a control is, as opposed to "the model should prefer good sources", which is a wish.

### 11.2 The regulatory-version problem and jurisdiction

Regulators revise instruments. Internal policies and procedures implement them, often with lag, often across multiple jurisdictions with different rules and different timetables. Three consequences:

- A **period of legitimate divergence** exists after a revision, during which the internal document is knowingly behind the instrument. Answers in that window should follow the instrument (authority tier A1) and should *say* that the internal document lags — a disclosure, rung 1, not a silent choice.
- **Jurisdiction is not a detail.** A group-level answer applied in a jurisdiction whose regime differs is a conflict the user may not know exists. The correct output scopes the answer to a perimeter and states it.
- **Escalation ownership differs by jurisdiction**, so the escalation path of [§9.6](#96-the-escalation-path) is scoped by the register's jurisdiction field rather than centralised.

### 11.3 Customer-data conflict

A specific and under-discussed case: the retrieved sources disagree *about a customer*. One system of record says one thing, a downstream derived dataset another, a document a third. This is the version-conflict shape applied to entities rather than instruments, and its resolution is the same: the entitled source for the field determines the answer, which means the register must name the **system of record per data field**, not merely per document. Predicting this onto the data governance material: the register's authority-tier field and a data catalogue's system-of-record designation are the same idea applied to different objects, which is why [Data Governance Framework](../../technology/data/data_governance_framework.md) and [Data Compliance Frameworks](../../technology/data/data_compliance_frameworks.md) are the natural companions to this section.

### 11.4 The audit requirement

After the fact, the institution must be able to identify **which source an answer came from**. That requires, at minimum: the answer's cited sources, their versions and effective dates, the as-of date of the retrieval, and the authority tier applied. Without these, an answer cannot be defended to a supervisor or reproduced for an audit, and the institution cannot distinguish "we answered correctly from the then-current procedure" from "we answered from a document we had withdrawn". This is a reason to prefer rung 1 and rung 2 ([§6.1](#61-rung-1--do-not-resolve-it-surface-it), [§6.2](#62-rung-2--authority-and-entitlement)) that has nothing to do with answer quality: **they are the rungs that leave a defensible record.**

### 11.5 The correct output is often the disclosed conflict

In a regulated setting, the correct output is frequently **not the confident answer**. Where sources genuinely conflict, a confidently-resolved answer is a liability: it asserts a determination the institution has not made, hides the existence of the disagreement from the person who needs to know about it, and removes the trigger for the human decision that resolves it properly. A disclosed conflict — "these two procedures differ on X; the current procedure says Y and the regulation says Z; here is what would resolve it" — is the safer and often the *more correct* output, because it is accurate about the state of the institution's own knowledge.

### 11.6 Cross-references

The obligation, regulatory and governance material lives elsewhere in the repository and is not re-derived here: [AI and GenAI Banking Compliance](../../banking/ai_genai_banking_compliance_guide.md), [MAS Regulations and Guidelines](../../banking/mas_regulations_guidelines_guide.md), [RegTech](../../banking/regtech_guide.md), [Data Governance Framework](../../technology/data/data_governance_framework.md), [Data Compliance Frameworks](../../technology/data/data_compliance_frameworks.md), and the point-in-time material in [Market Data Integrity](../../banking/market_data_integrity_guide.md). What this guide adds to that material is the retrieval-time mechanism: which source speaks, and how the system knows.

---

## 12. The Cost and Performance Angle

Conflict handling is not free, and the honest framing is that it is a **trade against coverage and latency**, not a pure improvement.

### 12.1 What it costs

| Cost | Where it comes from | Rough shape |
| --- | --- | --- |
| Extra retrieval | Both-sides retrieval ([§9.4](#94-retrieving-both-sides-deliberately)), widened entitled subsets | Additional query volume per request |
| Extra comparison | Metadata joins, entailment checks, quantity extraction ([§5](#5-detecting-that-a-conflict-exists-at-all)) | Per-passage-pair work, reduced by clustering |
| Extra latency | Additional calls and longer prompts | User-visible where the interaction is synchronous |
| Context consumption | Two sources plus provenance labels ([§9.3](#93-the-conflict-aware-prompt)) | Fewer passages of other evidence fit |
| More abstentions | Cite-or-abstain ([§9.5](#95-cite-or-abstain)) | Lower answer rate; higher escalation volume |
| Operations | Escalation queue ([§9.6](#96-the-escalation-path)), register maintenance ([§9.1](#91-the-authority-annotated-store)) | Recurring staffing |
| Output shape | Disclosed conflicts are less satisfying than confident answers | Adoption risk, which is real and under-modelled |

The output-shape cost is the one that is usually forgotten in design and dominant in practice: a system that answers most questions confidently and occasionally discloses a conflict is *perceived* as worse than a system that always answers confidently, even when the latter is answering from withdrawn documents.

### 12.2 An abstention has a cost too

Abstention is not a zero-cost conservative choice. Every abstention is:

- a question the user must now answer elsewhere — often by asking a colleague, which is *less* controlled than the assistant, not more;
- a routing event into a human queue that must be staffed;
- an incentive for the user to route around the assistant entirely, taking their question to unstructured channels where no provenance exists at all;
- and, in a high-volume estate, a utility collapse if the abstraction rate crosses the point where users stop trusting the system to answer anything.

So the design decision is not "abstain when uncertain" but **where in the estate an abstention is acceptable**.

### 12.3 Where each trade is defensible

| Estate location | Character | Acceptable trade |
| --- | --- | --- |
| Regulated-advice surfaces — policy obligations, compliance interpretation | Low volume, high consequence, external audit | **Abstain and disclose freely.** Latency and answer-rate costs are small next to the cost of a confidently wrong obligation |
| High-volume operational lookup — procedure lookup, form guidance | High volume, moderate consequence, users with alternatives | **Resolve aggressively by authority and version; abstain rarely.** A frequent abstention here ends adoption and pushes users to less controlled channels |
| Internal exploratory search — "find me documents about X" | High volume, low consequence, user is browsing | **Do not resolve at all; rank and present.** This is retrieval, not adjudication; the user wants both sides |
| Analytic and reporting surfaces | Batch, no human waiting | **Resolve by authority; log disclosures.** Latency is nearly free; make the conflict visible in the output artefact |
| Escalation and case handling | Low volume, high consequence, a human is already in the loop | **Disclose the conflict and attach it to the case.** The human is the resolver |

The pattern behind the table: **abstention is cheap where a human is already involved or the consequence is high, and expensive where volume is high and the user has an uncontrolled alternative.** A single global abstention policy is a policy that is wrong in several places at once.

### 12.4 The cheap wins, in order

If the budget is limited, the ordering is clear because the costs and benefits are asymmetric:

1. **Carry version, date and source metadata through ingestion.** Cheap relative to every model-side mechanism, and it is the precondition for everything else.
2. **Filter by lifecycle state and as-of date before ranking.** Cheap, and it resolves the most common conflict kind outright.
3. **Run the position-swap experiment** ([§4.4](#44-context-position--retrieved-present-ignored)) on known-conflict queries. Nearly free, and it detects the failure that looks like success.
4. **Require citations and check them.** Moderate cost, and it is what makes the audit requirement of [§11.4](#114-the-audit-requirement) satisfiable.
5. Only then consider rung-5 mechanisms, and never as the only rung.

---

## 13. Worked Example — Cymbal Bank, Two Procedures and One Regulation

**Explicitly illustrative and fictional.** Cymbal Bank is the repository's bank persona and the only institution used here. The scenario below is constructed to exercise the guide's mechanisms; it is not a report about Cymbal Bank or about any real institution, and no real bank is asserted to use any technique described. Regulatory bodies are named where the mechanism requires a concrete regulator, and nothing is asserted about their rules' content beyond the construction of the example.

### 13.1 The scenario

Cymbal Bank's internal assistant answers policy and procedure questions for operations staff. A user asks:

> "How long do we retain customer identification records after the relationship ends?"

The assistant retrieves, at `k = 8`:

| Rank | Retrieved item | What it is | Metadata actually present in the index |
| --- | --- | --- | --- |
| 1 | "Customer Records Retention — Quick Guide" (wiki page) | A one-page summary written for operations, fluent, with a clear table | Title, ingestion date |
| 2 | "KYC Operations Procedure v3.1" (internal procedure) | The current procedure | Title, ingestion date |
| 3 | "KYC Operations Procedure v2.4" (internal procedure) | **Superseded** 14 months ago | Title, ingestion date |
| 4 | The regulator's record-keeping guidance (external instrument) | The instrument the procedures both claim to implement | Title, ingestion date |
| 5–8 | Assorted relevant material | Related but not on the retention point | Title, ingestion date |

Every record carries a title and the date the *document* was ingested. None carries: the version, the lifecycle state, the supersession date, the effective date, the authority tier, or the parent-document link. The ingestion pipeline extracted text and discarded everything else.

### 13.2 What the pipeline does, and why it is not a model failure

The prompt is assembled with ranks 1–8 in order. The model answers with a retention period. Two things have already happened that no prompt can fix:

- **The superseded procedure is retrievable and was retrieved** (rank 3), because a similarity index has no concept of supersession. It is a near-perfect match for the query — it is the same document, on the same subject, in the same vocabulary.
- **The fluent guide outranks both the procedure and the instrument** (rank 1), because it was written to be read and relevance ranking rewards that. Per [§6.5](#65-rung-5--model-side-mechanisms), a model choosing between them is predisposed to prefer the fluently-written one.

Note that the retrieved set contains a conflict of *two kinds at once*: a version conflict (v3.1 against v2.4, [§2.2](#22-the-enterprise-inventory) #1) and an authority conflict (the quick guide and both procedures against the instrument, #2). The pipeline notices neither, because it has no metadata with which to notice. And if v2.4 had ranked ninth, the version conflict would have been invisible entirely — the [§4.1](#41-the-top-k-cut--the-losing-evidence-is-never-retrieved) cut, silent.

### 13.3 Detection, if the metadata had existed

With the register fields of [§5.1](#51-metadata-first--the-cheapest-detector-and-the-most-often-omitted), four of the seven retrieved records would have carried a flag before any model was called:

| Signal | Value | Flags |
| --- | --- | --- |
| Lifecycle state on rank 3 | `superseded`, effective-to date 14 months ago | Version conflict |
| Effective dates on ranks 2 and 3 | Disjoint intervals | Version conflict, temporal |
| Authority tier: rank 1 = A5, rank 4 = A1 | Cross-tier pairing | Authority conflict |
| Parent document: rank 1 derives from rank 2 | Derivation edge | Corroboration is not independent ([§6.4](#64-rung-4--corroboration-and-provenance)) |

The fourth row is the one teams find surprising: the quick guide is *not* a second opinion. It restates the procedure. Counting it as corroboration for the procedure's figure would be counting one source twice — the popularity-versus-truth trap of [§6.4](#64-rung-4--corroboration-and-provenance).

### 13.4 The ladder, applied rung by rung

**Rung 1 — do not resolve it; surface it.** *Partly applicable.* The authority conflict between the guide and the instrument is a disclosure case: the answer should state the instrument's position and note that the internal guide is a summary. The version conflict is not a disclosure case — it has a determinate answer.

**Rung 2 — authority and entitlement.** *Decisive.* The register says rank 4 is tier A1, ranks 2–3 are A4, rank 1 is A5. Rank 4 speaks on the rule; rank 2 speaks on Cymbal's implementation; rank 1 speaks as guidance only. The answer is built from A1 for the obligation and A4 for the procedure, with A5 supporting or omitted.

**Rung 3 — version and recency.** *Decisive, and applicable to the same document.* v3.1 is current; v2.4 is superseded with an effective-to date. v2.4 outranks nothing ([§7.1](#71-the-ordering)) and must not be presented as a competing current view. Note the direction: this is *not* "prefer the newer document" — it is "the superseded version of this document is not authoritative for now". The two orderings coincide here only because they concern the same instrument.

**Rung 4 — corroboration and provenance.** *Rejected on inspection.* The apparent two-of-three agreement (quick guide and v3.1 say the same) collapses to one source once the derivation edge is known. Corroboration is unavailable, which is the correct outcome — it should be unavailable whenever sources derive from one another.

**Rung 5 — model-side mechanisms.** *Not needed, and would have been actively harmful.* If the question had been put to the model as "which of these sources is correct", the fluent rank-1 guide is the likely winner — the failure named in [§6.5](#65-rung-5--model-side-mechanisms). Rung 2 and rung 3 settle it without any model judgement about which source is *better*, which is the whole argument of this guide.

**Rung 6 — abstain and escalate.** *Not needed for the version conflict; needed for a different question.* For "what retention applied to a 2023 relationship?" the correct answer requires the superseded v2.4, retrieved **as of 2023**, presented as the then-current rule ([§8.2](#82-wrong-versus-right-for-its-date)). A system that had simply deleted v2.4 answers the present question and cannot answer the historical one. The distinction between *answerable-as-current* and *archived-but-retrievable* is the fix below.

### 13.5 The authority register Cymbal had never built

The remediation begins with an artefact that is not software: a source register ([§7.2](#72-the-source-register)). Cymbal's first version lists, for each source: source ID, subject scope, authority tier, owning function, jurisdiction, lifecycle state, effective dates, version, parent document, and derivation edges. Assigning tiers requires an actual decision about who outranks whom — operations procedure owners do not enjoy learning that the quick guide they maintain is tier A5, and the resulting argument is the governance work this guide has been insisting is the mechanism.

The register's second job is the derivation graph, which is what makes the quick guide properly subordinate: it is a summary of the procedure, so it cannot be a corroborating source for it, and it cannot be retrieved as the primary answer to an obligation question while the procedure exists.

The register's third job is *statement of the unsaid*: it must say, per subject, whether the internal tier lags the external instrument and what the answer should do in the gap ([§11.2](#112-the-regulatory-version-problem-and-jurisdiction)) — a disclosure, not a silent choice.

### 13.6 The honest turn — the fix is corpus hygiene, not a better prompt

The obvious remediation is a prompt: instruct the model to prefer the instrument and the current procedure, and to disregard superseded documents. That remediation fails, and the reason is structural rather than a matter of prompt quality:

- The instruction requires information the model does not have. "Superseded" **is not a property of the text.** v2.4 reads perfectly current; nothing in its words says it was withdrawn.
- It requires the model to be a reliable authority judge, and per [§3](#3-what-the-research-actually-shows-about-model-behaviour) behaviour is model-dependent and disposition varies with fluency and salience — which is exactly the axis on which the fluent guide wins.
- It cannot repair the version conflict at all, because the losing document survives the cut and remains in context competing for the answer.
- It is unauditable and untestable: there is no ground truth in an instruction.

**The real fix is that the superseded procedure should not have been in the answerable index.** Concretely:

1. **Remove v2.4 from the answerable set**, not from the archive — its lifecycle state in the register becomes `superseded`, and the retrieval filter of [§9.2](#92-metadata-first-filtering--retrieve-within-a-tier-before-comparing-across-tiers) excludes it by default while a separate historical route can still serve it with an as-of date.
2. **Carry version, lifecycle, effective dates, tier and parent into the index at ingest**, so that the exclusion is possible and so that the authority conflict between guide and instrument is detectable.
3. **Mark the quick guide as derived** from the procedure, so it can never stand as independent corroboration and never answers an obligation question as primary.
4. **Add the as-of route**, so historical questions are answered from historical documents *as historical* rather than being answered wrongly or refused.
5. **Only then** consider the prompt framing of [§9.3](#93-the-conflict-aware-prompt) — which, with the register beneath it, is presentation of facts rather than a hope that the model chooses well.

The lesson generalises and is the argument this guide has been making from [§1.1](#11-the-thesis-in-one-line): **the conflict was created by document lifecycle management and index ingestion, and it is resolved there.** The prompt was never the problem, and it is not the solution.

---

## 14. The Anti-Patterns

Seven. Each with its symptom, its cause, and the guardrail that prevents it. The guardrails are artefacts and tests, not instructions — which is the pattern across all of them.

### 14.1 Resolving a conflict silently

**Symptom.** Answers are confident and single-sourced in appearance; the user has no indication that the corpus disagreed. Incidents surface only when someone reads the sources.

**Cause.** The default pipeline has no conflict stage at all. A model handed two contradictory passages answers without narrating the choice ([§4.5](#45-the-unstated-silent-choice)), and no component downstream is looking for the omission.

**Guardrail.** Require attribution and check it ([§9.5](#95-cite-or-abstain)): an answer whose cited support set contains contradictory passages while asserting one of them is the detectable signature. Log the flag, and make the silence visible in evaluation.

### 14.2 Asking the model to pick a winner between an authoritative and a fluent source

**Symptom.** The well-written summary wins over the terse instrument; the internal wiki beats the regulator's guidance; behaviour looks fine on relevance metrics and wrong on content.

**Cause.** Selection by the model is selection on text properties. Fluency, specificity and salience correlate with writing quality, and the authoritative instrument is usually the driest document in the room ([§6.5](#65-rung-5--model-side-mechanisms)).

**Guardrail.** Resolve by entitlement *before* the model sees the set ([§9.2](#92-metadata-first-filtering--retrieve-within-a-tier-before-comparing-across-tiers)), and where both sides are presented, supply the tier as a label rather than leaving it to inference ([§9.3](#93-the-conflict-aware-prompt)). Where this pattern appears, the register is missing.

### 14.3 Counting corroboration as truth

**Symptom.** A widely-restated figure is treated as verified; the fifth paraphrase of a wrong number outvotes the instrument that contradicts it.

**Cause.** Majority is a cheap heuristic, and retrieval surfaces the most repeated material most easily — repetition is a relevance signal. The derivation graph is not known, so derived documents count as independent.

**Guardrail.** Provenance-first counting ([§6.4](#64-rung-4--corroboration-and-provenance)): collapse derived sources to their origin before counting, and treat corroboration as a tie-breaker among equal-authority sources only, never as authority itself.

### 14.4 Top-k that hides the losing evidence

**Symptom.** The answer is consistent and confident, and the competing document is provably in the corpus. Recall on the retrieval set looks healthy.

**Cause.** `k` is a truncation. If only one side of a conflict survives it, the conflict is not a conflict — it is an unopposed passage ([§4.1](#41-the-top-k-cut--the-losing-evidence-is-never-retrieved)). Relevance-based recall cannot see this, because the metric does not know the losing side should have been retrieved.

**Guardrail.** Label conflict cases in the golden set with *both* expected sides, and measure whether both were retrieved. Add deliberate competitor retrieval for known conflict-prone subjects ([§9.4](#94-retrieving-both-sides-deliberately)).

### 14.5 A prompt instruction standing in for a source register

**Symptom.** The system prompt contains a sentence about preferring authoritative sources. Nobody can say what the tiers are, who assigned them, or when they last changed. Nobody can produce a test that would fail if the preference stopped working.

**Cause.** The prompt is a paragraph of writing and the register is months of governance work. The prompt gets done ([§7.5](#75-the-honest-point)).

**Guardrail.** Make authority data, not prose: a register with owners, tiers and lifecycle states, versioned like any other control, with a test attached. If the tier exists only in a sentence, the control does not exist.

### 14.6 Treating a superseded document as merely less relevant

**Symptom.** A superseded procedure appears in answers with a lower score than the current one, occasionally contributing figures; a "relevance-boost" tweak is proposed as the fix.

**Cause.** Supersession is a *lifecycle state*, not a *relevance level*. A superseded document is a non-authoritative object, not a slightly-worse match, and a fully-superseded document has no correct place in the answerable set at all ([§7.1](#71-the-ordering)).

**Guardrail.** Lifecycle state as a filter, not a weight ([§9.2](#92-metadata-first-filtering--retrieve-within-a-tier-before-comparing-across-tiers)); archived documents on a separate as-of route ([§13.6](#136-the-honest-turn--the-fix-is-corpus-hygiene-not-a-better-prompt)). A relevance boost leaves the document competing; a filter removes it from the field.

### 14.7 Believing a conflict-benchmark score proves production safety

**Symptom.** A benchmark result on a public conflict benchmark is cited as evidence that the deployment handles disagreement. The production corpus has never been tested for conflicted queries.

**Cause.** The benchmark measures behaviour on *constructed* conflicts with both sides present and labelled ([§10.1](#101-what-the-conflict-benchmarks-test)); production conflicts are undeclared, often one-sided in context, and carry no labels.

**Guardrail.** Evaluate the pipeline on your own labelled conflict cases — entitled source retrieved, superseded source excluded, entitled source cited, abstention fired where required ([§10.3](#103-the-limit-to-state-to-stakeholders)). Treat a public conflict score as external context, never as assurance.

---

## 15. The Claims Audit

Every cited source is listed with its verified title, authors, year and identifier. Verification was performed against the arXiv API over HTTPS; the titles below match the API response exactly. Findings are reported with the setting in which they were obtained.

### 15.1 Verified — primary sources cited in this guide

| # | Source (verified title) | Authors | Year | Identifier | Where used | Quality / caveat |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | Knowledge Conflicts for LLMs: A Survey | Rongwu Xu, Zehan Qi, Zhijiang Guo, Cunxiang Wang, Hongru Wang, Yue Zhang, Wei Xu | 2024 | [arXiv:2403.08319](https://arxiv.org/abs/2403.08319) | Taxonomy in [§2.1](#21-what-the-literature-actually-classifies) | Survey; states its focus as three conflict categories (context–memory, inter-context, intra-memory) and covers causes, model behaviour and solutions. Secondary by nature, cited as the field's own organisation of the space |
| 2 | Adaptive Chameleon or Stubborn Sloth: Revealing the Behavior of Large Language Models in Knowledge Conflicts | Jian Xie, Kai Zhang, Jiangjie Chen, Renze Lou, Yu Su | 2023 | [arXiv:2305.13300](https://arxiv.org/abs/2305.13300) | [§2.1](#21-what-the-literature-actually-classifies), [§3.1](#31-the-chameleon-and-the-sloth) | Controlled experimental study (ICLR 2024 Spotlight); elicits parametric memory and constructs counter-memory, so it is predominantly context–memory; behaviour reported as dependent on evidence coherence/convincingness and on partial agreement with memory |
| 3 | ConflictBank: A Benchmark for Evaluating the Influence of Knowledge Conflicts in LLM | Zhaochen Su, Jun Zhang, Xiaoye Qu, Tong Zhu, Yanshu Li, Jiashuo Sun, Juntao Li, et al. | 2024 | [arXiv:2408.12076](https://arxiv.org/abs/2408.12076) | [§3.4](#34-what-corpus-scale-conflict-measurement-adds--conflictbank), [§10.1](#101-what-the-conflict-benchmarks-test) | Benchmark; **conflicts are constructed** by its own generation framework, across retrieved-knowledge/encoded-knowledge/interplay aspects and across stated causes (misinformation, temporal discrepancy, semantic divergence) — which limits transfer to production (the limit is stated in §10.1). Title reproduced exactly |
| 4 | Lost in the Middle: How Language Models Use Long Contexts | Nelson F. Liu, Kevin Lin, John Hewitt, Ashwin Paranjape, Michele Bevilacqua, Fabio Petroni, Percy Liang | 2023 | [arXiv:2307.03172](https://arxiv.org/abs/2307.03172) | [§4.4](#44-context-position--retrieved-present-ignored) | Empirical positional study; long-context QA/multi-document settings |
| 5 | Corrective Retrieval Augmented Generation | Shi-Qi Yan, Jia-Chen Gu, Yun Zhu, Zhen-Hua Ling | 2024 | [arXiv:2401.15884](https://arxiv.org/abs/2401.15884) | [§6.5](#65-rung-5--model-side-mechanisms) | Method paper; corrective action on judged-inadequate retrieval. Cross-referenced to the advanced-technique guide for landscape |
| 6 | Self-Consistency Improves Chain of Thought Reasoning in Language Models | Xuezhi Wang, Jason Wei, Dale Schuurmans, Quoc Le, Ed Chi, Sharan Narang, Aakanksha Chowdhery, Denny Zhou | 2022 | [arXiv:2203.11171](https://arxiv.org/abs/2203.11171) | [§6.5](#65-rung-5--model-side-mechanisms) | Reasoning-task setting; the mechanism is majority-over-samples, which does not import authority |
| 7 | Improving Factuality and Reasoning in Language Models through Multiagent Debate | Yilun Du, Shuang Li, Antonio Torralba, Joshua B. Tenenbaum, Igor Mordatch | 2023 | [arXiv:2305.14325](https://arxiv.org/abs/2305.14325) | [§6.5](#65-rung-5--model-side-mechanisms) | Debate-improves-factuality result; does not test authority ordering |
| 8 | ClashEval: Quantifying the tug-of-war between an LLM's internal prior and external evidence | Kevin Wu, Eric Wu, James Zou | 2024 | [arXiv:2404.10198](https://arxiv.org/abs/2404.10198) | [§3.2](#32-prior-versus-evidence--clasheval) | Prior-vs-evidence benchmark: 1,200+ questions, six domains, six top-performing LLMs incl. GPT-4o, answer content perturbed from subtle to blatant. Reports adoption of incorrect retrieved content against a correct prior in over 60% of cases, *lower* adoption as deviation from truth grows, and higher adoption as initial model confidence falls. Findings reported as the abstract reports them |
| 9 | The Power of Noise: Redefining Retrieval for RAG Systems | Florin Cuconasu, Giovanni Trappolini, Federico Siciliano, Simone Filice, Cesare Campagnano, Yoelle Maarek, Nicola Tonellotto, Fabrizio Silvestri | 2024 | [arXiv:2401.14887](https://arxiv.org/abs/2401.14887) | [§3.5](#35-irrelevant-context-and-noise), [§4.4](#44-context-position--retrieved-present-ignored) | Reports position dependence and harm from near-miss distractors, with a counterintuitive random-document result; both directions reported |
| 10 | Benchmarking Large Language Models in Retrieval-Augmented Generation | Jiawei Chen, Hongyu Lin, Xianpei Han, Le Sun | 2023 | [arXiv:2309.01431](https://arxiv.org/abs/2309.01431) | [§3.3](#33-the-rgb-benchmark--counterfactual-robustness), [§6.6](#66-rung-6--abstain-and-escalate-to-a-human), [§10.1](#101-what-the-conflict-benchmarks-test) | The RGB benchmark's four abilities; counterfactual context is injected, so the setting is constructed |
| 11 | Making Retrieval-Augmented Language Models Robust to Irrelevant Context | Ori Yoran, Tomer Wolfson, Ori Ram, Jonathan Berant | 2023 | [arXiv:2310.01558](https://arxiv.org/abs/2310.01558) | [§3.5](#35-irrelevant-context-and-noise) | Sensitivity to irrelevant context and a training-based robustness intervention; relevant-context baselines preserved per the paper |
| 12 | Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection | Akari Asai, Zeqiu Wu, Yizhong Wang, Avirup Sil, Hannaneh Hajishirzi | 2023 | [arXiv:2310.11511](https://arxiv.org/abs/2310.11511) | [§6.5](#65-rung-5--model-side-mechanisms) | Critique/reflection mechanism; owned in agent-output terms by the feedback-mechanisms guide |
| 13 | Chain-of-Verification Reduces Hallucination in Large Language Models | Shehzaad Dhuliawala, Mojtaba Komeili, Jing Xu, Roberta Raileanu, Xian Li, Asli Celikyilmaz, Jason Weston | 2023 | [arXiv:2309.11495](https://arxiv.org/abs/2309.11495) | [§6.5](#65-rung-5--model-side-mechanisms) | Hallucination-reduction result; not a conflict-resolution mechanism |
| 14 | Sufficient Context: A New Lens on Retrieval Augmented Generation Systems | Hailey Joren, Jianyi Zhang, Chun-Sung Ferng, Da-Cheng Juan, Ankur Taly, Cyrus Rashtchian | 2024 | [arXiv:2411.06037](https://arxiv.org/abs/2411.06037) | [§6.6](#66-rung-6--abstain-and-escalate-to-a-human), [§9.5](#95-cite-or-abstain) | Provides the context-sufficiency lens and the abstention framing |
| 15 | Enabling Large Language Models to Generate Text with Citations | Tianyu Gao, Howard Yen, Jiatong Yu, Danqi Chen | 2023 | [arXiv:2305.14627](https://arxiv.org/abs/2305.14627) | [§4.5](#45-the-unstated-silent-choice), [§9.5](#95-cite-or-abstain), [§10.2](#102-what-to-borrow-by-name) | The ALCE work; source of citation precision/recall as named metrics |
| 16 | Time-Aware Language Models as Temporal Knowledge Bases | Bhuwan Dhingra, Jeremy R. Cole, Julian Martin Eisenschlos, Daniel Gillick, Jacob Eisenstein, William W. Cohen | 2021 | [arXiv:2106.15110](https://arxiv.org/abs/2106.15110) | [§8.1](#81-the-two-directions-of-one-problem) | Temporal-generalisation line; model-side, cited as the research framing for as-of answering |
| 17 | RAGTruth: A Hallucination Corpus for Developing Trustworthy Retrieval-Augmented Language Models | Cheng Niu, Yuanhao Wu, Juno Zhu, Siliang Xu, Kashun Shum, Randy Zhong, Juntong Song, Tong Zhang | 2023 | [arXiv:2401.00396](https://arxiv.org/abs/2401.00396) | [§10.2](#102-what-to-borrow-by-name) | Hallucination corpus with word-level annotation; borrowed for the **span-annotation labelling practice**, not for a conflict finding |
| 18 | FEVER: a large-scale dataset for Fact Extraction and VERification | James Thorne, Andreas Vlachos, Christos Christodoulopoulos, Arpit Mittal | 2018 | [arXiv:1803.05355](https://arxiv.org/abs/1803.05355) | [§5.2](#52-content-signals--contradiction-and-entailment-across-passages) | Claim-verification dataset; borrowed for the task *shape* (entailed / refuted), not used as a benchmark target here |
| 19 | Passage Re-ranking with BERT | Rodrigo Nogueira, Kyunghyun Cho | 2019 | [arXiv:1901.04085](https://arxiv.org/abs/1901.04085) | [§4.3](#43-the-reranker--the-minority-view-is-demoted) | The reranking mechanism; cited for the mechanism, with the observation that relevance is not authority being this guide's own point |
| 20 | Retrieval-Augmented Generation for Large Language Models: A Survey | Yunfan Gao, Yun Xiong, Xinyu Gao, Kangxiang Jia, Jinliu Pan, Yuxi Bi, Yi Dai, Jiawei Sun, Meng Wang, Haofen Wang | 2023 | [arXiv:2312.10997](https://arxiv.org/abs/2312.10997) | [§9](#9-the-design-patterns) intro, technique-landscape framing | The RAG survey; cited as context for the technique landscape owned by other guides |

### 15.2 Findings reported with their settings

| Finding | Reported in | Setting it was obtained in | Limitation stated in this guide |
| --- | --- | --- | --- |
| Conflict behaviour splits into receptivity to convincing evidence (chameleon) and confirmation bias when evidence partly agrees with memory (sloth); which appears depends on evidence coherence and on partial agreement | [arXiv:2305.13300](https://arxiv.org/abs/2305.13300) | Controlled experiments with elicited parametric memory and constructed counter-memory; predominantly context–memory | Predominantly model-vs-memory, not passage-vs-passage; the dependence conditions are the paper's, not a rate |
| Models adopt incorrect retrieved content over a correct prior in over 60% of cases; adoption falls as the content deviates further from truth, and rises as the model's initial confidence falls | [arXiv:2404.10198](https://arxiv.org/abs/2404.10198) | Benchmark of 1,200+ questions across six domains; six top-performing LLMs incl. GPT-4o; answer content perturbed from subtle to blatant | The rates are the paper's, measured on its constructed perturbations; not transferable as rates for a given corpus |
| Evaluated models are weak at noise robustness, negative rejection, information integration, counterfactual robustness | [arXiv:2309.01431](https://arxiv.org/abs/2309.01431) | Constructed benchmark with injected counterfactual context | Constructed conflicts; the rates are the benchmark's, not a production estimate |
| Long-context performance degrades when relevant information sits in the middle | [arXiv:2307.03172](https://arxiv.org/abs/2307.03172) | Long-context and multi-document QA probing | Position effects observed in the paper's settings; not a universal law, and not about conflict specifically |
| Retrieval composition and position affect outcomes; near-miss distractors can hurt | [arXiv:2401.14887](https://arxiv.org/abs/2401.14887) | Controlled retrieval-composition experiments | Includes a counterintuitive random-document improvement, reported; not a conflict study |
| Retrieval can reduce accuracy rather than help; NLI entailment filtering prevents the degradation but discards relevant passages; fine-tuning on a generated mix of relevant and irrelevant contexts is reported effective with ~1,000 examples | [arXiv:2310.01558](https://arxiv.org/abs/2310.01558) | Five open-domain QA benchmarks with annotated irrelevant context | Irrelevance, not contradiction; the two are related but distinct |

### 15.3 Labelled as the guide's own analysis

The following are **not** attributed to any cited source. They are this guide's analysis and should be read as engineering judgement, not as verified findings:

- The **seven-kind enterprise inventory** in [§2.2](#22-the-enterprise-inventory) — version, authority, jurisdictional, terminology, unit-and-scale, temporal, genuine factual disagreement.
- The **authority tier ordering** and the assertion that a superseded version outranks nothing ([§7.1](#71-the-ordering)).
- The **resolution ladder** and its honesty ordering ([§6](#6-the-resolution-ladder)), including the claim that a model asked to pick a winner prefers the most fluent or salient source. That claim is consistent with, but not demonstrated by, the cited behavioural work; it is labelled here as the guide's analysis.
- The **five pipeline hiding mechanisms** in [§4](#4-why-the-pipeline-hides-the-conflict) and the assertion that a conflict you never retrieve cannot be detected.
- The **design patterns** in [§9](#9-the-design-patterns), their costs, and their stated non-coverage.
- The **estate-location cost table** in [§12.3](#123-where-each-trade-is-defensible).
- The **anti-patterns** in [§14](#14-the-anti-patterns) and the corpus-hygiene remediation in [§13.6](#136-the-honest-turn--the-fix-is-corpus-hygiene-not-a-better-prompt).
- The **inference in [§3.2](#32-prior-versus-evidence--clasheval)** that source authority is therefore a retrieval-layer obligation rather than a model-side one. The behavioural findings are the papers' as reported above; the organisational conclusion drawn from them is this guide's, and it is labelled as such where it is drawn.

### 15.4 Rejected claims

| Claim considered | Why rejected |
| --- | --- |
| "Better retrieval solves conflict" | Not supported by the cited behavioural findings: models adopt incorrect retrieved content over a correct prior in the majority of cases in the ClashEval benchmark ([arXiv:2404.10198](https://arxiv.org/abs/2404.10198)) — retrieving more of the same wrong thing does not help; and IR improvement cannot address a conflict within a tier, a superseded-source lifecycle failure, or a losing side that was never retrieved |
| "Model-side conflict resolution is reliable" | Not supported by any cited source; [§3.6](#36-what-the-research-does-not-show) states the literature shows variability and no general solution |
| "A conflict-benchmark score establishes production safety" | Constructed conflicts do not reproduce the hidden-side and metadata-absent conditions of production ([§10.3](#103-the-limit-to-state-to-stakeholders)) |
| "A prompt instruction to prefer authoritative sources is a control" | Untestable and unauditable without a register; an instruction is not a control ([§7.5](#75-the-honest-point)) |
| "Corroboration establishes truth" | Corroboration counts agreement, not truth; derivation collapse removes the apparent agreement ([§6.4](#64-rung-4--corroboration-and-provenance)) |
| "Recency resolves conflicts" | Recency is neither authority nor point-in-time correctness; the newest document may be the unauthorised one, and the correct question is often what was in force *then* ([§8](#8-temporal-conflict-and-the-point-in-time-question)) |
| Any assertion about a named real institution's conflict handling | Not verifiable and excluded by policy; the worked example is fictional and the only institution used is Cymbal Bank |

---

## 16. What Could Not Be Verified, Glossary, Cross-References and Closing Summary

### 16.1 What could not be verified

Stated plainly, because the guide's argument depends on not overclaiming:

- **No source consulted demonstrates a reliable general solution to inter-context conflict.** The cited literature is predominantly about context–memory conflict; the enterprise problem is predominantly inter-context. No cited paper closes that gap, and none is claimed to.
- **The findings are model-, setting- and phrasing-dependent.** The behavioural studies report dependence explicitly. This guide therefore does **not** state any conflict-behaviour rate as applying to a production system, to a specific model in a specific deployment, or to a bank's corpus.
- **ConflictBank's conflicts are constructed** by a purpose-built generation framework across retrieved-knowledge, encoded-knowledge and interplay settings. That is the benchmark's own design, and it is why [§10.1](#101-what-the-conflict-benchmarks-test) treats its results as measures of behaviour on constructed conflicts rather than as a production estimate. Its *causes* (temporal discrepancy, semantic divergence, misinformation) and its *coverage* (model families and instances) are quoted here from the paper's abstract — the source actually read for this guide; **no figure from the paper's results tables is reproduced**, because a number taken from a secondary description rather than the paper's own tables would be a citation of a summary.
- **No claim is made about any real institution's conflict-handling practice.** No real bank is asserted to use any technique here. The worked example is fictional, uses Cymbal Bank as the sole institution, and is labelled as illustrative in the section itself.
- **The authority tier ordering is this guide's analysis, not a standard.** No cited source proposes tier tables of this shape. It is offered as an engineering pattern with the honest caveat that the real work is the institutional argument about who outranks whom ([§7.5](#75-the-honest-point)).
- **The claim that a model prefers the most fluent source in a conflict is labelled as the guide's analysis.** It is consistent with the cited behavioural results on salience and evidence strength, and it is plausible mechanism as the guide reads it — but it is not a verified finding of a cited paper, and it is flagged as such in [§15.3](#153-labelled-as-the-guides-own-analysis).
- **No metric proposed here is an invention of this guide.** Measurements named are citation precision/recall ([arXiv:2305.14627](https://arxiv.org/abs/2305.14627)), the four RGB abilities ([arXiv:2309.01431](https://arxiv.org/abs/2309.01431)), the context-sufficiency framing ([arXiv:2411.06037](https://arxiv.org/abs/2411.06037)), and the golden-set method owned by [RAG Evaluation Methodology](rag_evaluation_methodology_guide.md).

### 16.2 Glossary

| Term | Definition |
| --- | --- |
| **Abstention** | Declining to answer, with the reason given, rather than asserting an unsupported resolution. A designed output ([§6.6](#66-rung-6--abstain-and-escalate-to-a-human)), not a failure |
| **As-of date** | The point in time a query is answered against; the pivot of the temporal discipline in [§8](#8-temporal-conflict-and-the-point-in-time-question) |
| **Authority tier** | The rank a source holds in a source register expressing its entitlement to speak on a subject. Governance data, not a model setting |
| **Chameleon behaviour** | A model's disposition to adopt supplied context readily, including when it is wrong (Xie et al., [arXiv:2305.13300](https://arxiv.org/abs/2305.13300)) |
| **Context–memory conflict** | Retrieved or supplied context contradicting parametric knowledge |
| **Corroboration** | Agreement among retrieved passages. Counts popularity, not truth |
| **Derivation graph** | The record of which sources summarise or derive from which; the precondition for honest corroboration counting |
| **Disclosed conflict** | An output that presents the disagreement between sources with provenance instead of resolving it ([§6.1](#61-rung-1--do-not-resolve-it-surface-it)) |
| **Entitlement** | The right of a source to speak on a subject, as recorded in the register and scoped to (source, subject) |
| **Inter-context conflict** | Retrieved passages contradicting each other; the family that dominates enterprise RAG |
| **Knowledge conflict** | The general phenomenon of model exposure to contradictory information; the field's organising subject (Xu et al., [arXiv:2403.08319](https://arxiv.org/abs/2403.08319)) |
| **Lifecycle state** | Current / superseded / withdrawn / draft — the field that prevents version conflict, and a filter rather than a weight ([§14.6](#146-treating-a-superseded-document-as-merely-less-relevant)) |
| **Parametric knowledge** | What the model carries in its weights; undated, unattributed, unauditable |
| **Point-in-time retrieval** | Retrieving the sources in force at a specified time; the correct discipline for historical questions ([§8](#8-temporal-conflict-and-the-point-in-time-question)) |
| **Retrieved evidence** | The passages actually placed in context for a query — not the corpus |
| **Sloth behaviour** | A model's disposition to hold parametric knowledge stubbornly against supplied evidence (Xie et al., [arXiv:2305.13300](https://arxiv.org/abs/2305.13300)) |
| **Source register** | The inventory of answerable sources with tier, owner, jurisdiction, lifecycle and dates ([§7.2](#72-the-source-register)) |
| **Sufficiency** | Whether the retrieved context can support an answer at all (Joren et al., [arXiv:2411.06037](https://arxiv.org/abs/2411.06037)) |
| **Superseded version** | A document correct for its period and replaced; not weak, but non-authoritative for now |
| **Version pinning** | Recording and honouring the version and as-of date a query was answered against; the basis of the audit requirement ([§11.4](#114-the-audit-requirement)) |

### 16.3 Cross-references

| Subject | Where it lives |
| --- | --- |
| Advanced retrieval techniques, corrective patterns, the technique landscape | [advanced_rag_techniques_guide.md](advanced_rag_techniques_guide.md) |
| Evaluation methodology, metric definitions, the golden-set method | [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md) |
| Evaluation tooling | [rag_evaluation_tools_comparison_guide.md](rag_evaluation_tools_comparison_guide.md) |
| Public benchmark landscape and how to read a benchmark claim | [rag_benchmarks_guide.md](rag_benchmarks_guide.md) |
| Retrieval optimisation (not conflict resolution) | [rag_optimization_techniques_guide.md](rag_optimization_techniques_guide.md) |
| Vector-store selection and embedding metadata capacity | [vector_databases_guide.md](vector_databases_guide.md) |
| Long-context instead of retrieval | [rag_vs_long_context_llms_guide.md](rag_vs_long_context_llms_guide.md) |
| Ingestion and streaming | [rag_with_data_streaming_guide.md](rag_with_data_streaming_guide.md) |
| Query rewriting | [query_rewriting_rag_guide.md](query_rewriting_rag_guide.md) |
| Beyond-retrieval architectures | [beyond_rag_guide.md](beyond_rag_guide.md) |
| Agent-versus-agent disagreement, arbitration, hierarchical resolution | [../hierarchical_multi_agent_frameworks_guide.md](../hierarchical_multi_agent_frameworks_guide.md) · [../agents_at_scale_guide.md](../agents_at_scale_guide.md) · [../hybrid_multi_agent_systems_guide.md](../hybrid_multi_agent_systems_guide.md) |
| Write and concurrency conflicts | [../agents_at_scale_guide.md](../agents_at_scale_guide.md) |
| Verifier and critique loops in the agent-output setting | [../feedback_mechanisms_llm_agents_guide.md](../feedback_mechanisms_llm_agents_guide.md) |
| Context and tool-surface disclosure | [../mcp_progressive_disclosure_guide.md](../mcp_progressive_disclosure_guide.md) |
| Point-in-time correctness in market data | [../../banking/market_data_integrity_guide.md](../../banking/market_data_integrity_guide.md) |
| Pipeline and data versioning | [../../technology/data/data_pipeline_versioning.md](../../technology/data/data_pipeline_versioning.md) |
| Data governance, ownership, stewardship | [../../technology/data/data_governance_framework.md](../../technology/data/data_governance_framework.md) |
| Data compliance frameworks | [../../technology/data/data_compliance_frameworks.md](../../technology/data/data_compliance_frameworks.md) |
| AI and GenAI compliance in banking | [../../banking/ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) |
| Regulatory guidelines | [../../banking/mas_regulations_guidelines_guide.md](../../banking/mas_regulations_guidelines_guide.md) |
| Regulatory technology | [../../banking/regtech_guide.md](../../banking/regtech_guide.md) |

### 16.4 Closing summary

- **Retrieval decides what is in the context window. It does not decide what deserves to win.**
- **The literature's taxonomy has three categories** — context–memory, inter-context and intra-memory — and the **enterprise** problem is overwhelmingly inter-context, the family with the least published guidance (Xu et al., [arXiv:2403.08319](https://arxiv.org/abs/2403.08319)).
- **Model behaviour in the face of conflict varies — and the deciding variables are not the source's standing.** It varies with the model, with how coherent and convincing the supplied evidence is, with whether part of that evidence agrees with the model's own memory, with how far the content deviates from truth, and with the model's own confidence in its first answer (Xie et al., [arXiv:2305.13300](https://arxiv.org/abs/2305.13300); Wu et al., [arXiv:2404.10198](https://arxiv.org/abs/2404.10198)). Chameleon and sloth are both failure modes; in neither case is the *authority* of the passage the variable that decides the outcome. That variability is the finding, and it is why no prompt is a control.
- **The pipeline hides conflicts before the model ever sees one** — the top-k cut, chunk boundaries, the relevance reranker demoting the terse authority, position effects, and the silent undeclared resolution. A conflict you never retrieve is a conflict you cannot detect, and reranking a truncated candidate set cannot fix it.
- **Detection is mostly metadata.** Version, lifecycle state, effective dates, jurisdiction, authority tier and derivation lineage do the work before any model call — and they are the fields most often discarded at ingest.
- **The ladder runs in order of honesty:** surface it, then authority, then version, then corroboration, then model-side mechanisms, then abstain and escalate. **Model-side mechanisms are the weakest rung for an enterprise** and must never be the only one; a model asked to pick a winner prefers the fluent source, and the fluent source is usually the summary, not the instrument.
- **Authority is a governance artefact, owned by someone, not a sentence in a prompt.** The source register — tiered, scoped, versioned, with a derivation graph — is the mechanism. A prompt that says "prefer authoritative sources" is a hope, not a control.
- **Six of the seven enterprise conflict kinds are settled by metadata and governance.** Only genuine factual disagreement is a real epistemic impasse, and for it the correct output is the disclosed conflict.
- **In a regulated institution this is a control, not a feature.** Answering from a superseded policy is a compliance failure, not a model error — and the fix, as the worked example shows, is corpus hygiene: the superseded procedure should never have been in the answerable index.
- **An abstention has a cost too.** The design question is where in the estate each trade is defensible, not whether abstention is good.
- **A conflict-benchmark score does not make a deployment safe.** Constructed conflicts have both sides present and labelled; production conflicts are undeclared and often one-sided in context.
- And the rule that governs all of it: **retrieval does not resolve conflicts; authority does.**
