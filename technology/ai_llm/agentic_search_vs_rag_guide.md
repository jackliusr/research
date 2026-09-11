# Agentic Search vs Classic RAG — A Head-to-Head Comparison for Enterprise Retrieval Architecture

> **Author:** Jack Liu Shurui, Solution Architect
> **Repository:** [jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Applied AI Architecture for the Enterprise
> **Companion Guides:** [RAG evolution timeline](rag/rag_evolution_timeline_guide.md) · [Advanced RAG techniques](rag/advanced_rag_techniques_guide.md) · [RAG optimization techniques](rag/rag_optimization_techniques_guide.md) · [Production-grade agentic RAG](rag/production_grade_agentic_rag_guide.md) · [Beyond RAG](rag/beyond_rag_guide.md) · [RAG vs HyDE](rag/rag_vs_hyde_guide.md) · [Query rewriting for RAG](rag/query_rewriting_rag_guide.md) · [RAG vs long-context LLMs](rag/rag_vs_long_context_llms_guide.md) · [RAG evaluation methodology](rag/rag_evaluation_methodology_guide.md) · [Vector databases](rag/vector_databases_guide.md) · [Agent scaffolding](agent_scaffolding_guide.md) · [Context engineering](context_engineering_guide.md) · [Agent harness engineering](agent_harness_engineering_guide.md) · [Production-ready LLM agents](production_ready_llm_agents_guide.md) · [LLM agent failure modes](llm_agents_failures_production_guide.md) · [Autonomous agents](autonomous_agents_guide.md) · [Hybrid multi-agent systems](hybrid_multi_agent_systems_guide.md) · [Hierarchical multi-agent frameworks](hierarchical_multi_agent_frameworks_guide.md) · [Multi-agent banking](multi_agent_banking_guide.md) · [MCP discovery](mcp_discovery_guide.md) · [AI governance framework](ai_governance_framework_guide.md) · [LLM evaluation frameworks](llm_evaluation_frameworks_guide.md) · [Research agents](../research_agents_guide.md) · [Agentic workflows](../agentic_workflows_guide.md) · [Low-latency GenAI patterns](../low_latency_genai_patterns_guide.md) · [AI for IT](ai_for_it_guide.md) · [GenAI banking compliance](../../banking/ai_genai_banking_compliance_guide.md)
> **Last Updated:** September 2026

---

## Table of Contents

1. [How This Guide Fits the Series](#how-this-guide-fits-the-series)
2. [The Two Architectures at the Primary Literature](#1-the-two-architectures-at-the-primary-literature)
3. [Classic RAG — The Retrieve-Then-Generate Pipeline and Its Lineage](#2-classic-rag--the-retrieve-then-generate-pipeline-and-its-lineage)
4. [Agentic Search — The Reason-Act-Observe Loop](#3-agentic-search--the-reason-act-observe-loop)
5. [The Mechanisms, Compared](#4-the-mechanisms-compared)
6. [The Trade-Offs, With Evidence](#5-the-trade-offs-with-evidence)
7. [Hybrid Patterns](#6-hybrid-patterns)
8. [The Selection Framework](#7-the-selection-framework)
9. [Enterprise Guidance for Regulated and Banking Contexts](#8-enterprise-guidance-for-regulated-and-banking-contexts)
10. [Worked Example — The Cymbal Bank Knowledge Assistant](#9-worked-example--the-cymbal-bank-knowledge-assistant)
11. [Claims Audit](#10-claims-audit)
12. [What Could Not Be Verified](#what-could-not-be-verified)
13. [Glossary](#glossary)
14. [References](#references)

---

## How This Guide Fits the Series

This guide is the **comparison and selection** volume of the retrieval series. It does not re-derive the internals of either architecture; it puts them side by side and answers one question a solution architect actually gets asked in a design review: *for this workload, on this budget, under this audit regime — do I build a classic retrieve-then-generate RAG pipeline, or an agentic search loop, or some hybrid of the two?*

The dedup boundary is deliberate:

- **The RAG fundamentals live elsewhere.** Chunking, embedding models, ANN indexes, reranking, HyDE, query rewriting, hybrid sparse-dense retrieval, and the RAG evaluation methodology are covered in the [RAG series](rag/advanced_rag_techniques_guide.md) — specifically [vector databases](rag/vector_databases_guide.md), [HyDE](rag/rag_vs_hyde_guide.md), [query rewriting](rag/query_rewriting_rag_guide.md), [optimization](rag/rag_optimization_techniques_guide.md), and [evaluation methodology](rag/rag_evaluation_methodology_guide.md). This guide assumes you know what a dense retriever does and does not re-explain it.
- **The agent patterns live elsewhere.** The reason-act-observe loop, scaffolding, harness design, context management, tool protocols, multi-agent topologies, and agent failure modes are covered in the [agent series](agent_scaffolding_guide.md) — [scaffolding](agent_scaffolding_guide.md), [context engineering](context_engineering_guide.md), [harness engineering](agent_harness_engineering_guide.md), [production-ready agents](production_ready_llm_agents_guide.md), [failure modes](llm_agents_failures_production_guide.md), [autonomous agents](autonomous_agents_guide.md), and [MCP discovery](mcp_discovery_guide.md). This guide treats the loop as a *retrieval mechanism* and compares it against the pipeline.
- **The production-agentic-RAG build guide lives elsewhere.** The [production-grade agentic RAG guide](rag/production_grade_agentic_rag_guide.md) is the construction manual for the loop. This guide is the *selection* manual: it assumes such a system can be built and asks whether it should be, for a given workload.
- **The "what comes after RAG" survey lives elsewhere.** [Beyond RAG](rag/beyond_rag_guide.md) surveys nine paradigms including long-context-native and memory-based alternatives. This guide sits inside that landscape — it is the pairwise comparison of exactly two of those paradigms.
- **Governance and compliance live elsewhere.** Audit, model risk, residency, and PII obligations are covered in the [AI governance framework](ai_governance_framework_guide.md), the [banking compliance guide](../../banking/ai_genai_banking_compliance_guide.md), and the [LLM evaluation frameworks guide](llm_evaluation_frameworks_guide.md). Section 8 condenses to them; it does not re-derive them.

A note on method, consistent with the rest of the series: every empirical claim carries a ✅ (verified against a primary source I actually fetched), ⚠ (real but non-peer-reviewed, vendor-published, or attribution-uncertain), or ❌ (could not verify; do not rely on it). The [What Could Not Be Verified](#what-could-not-be-verified) section lists the gaps honestly. Constructed figures — especially cost and latency multipliers — are labelled *illustrative* and are not presented as measurements.

---

## 1. The Two Architectures at the Primary Literature

The comparison only makes sense once both sides are pinned to their canonical papers. Here they are, in the order they entered the literature.

### 1.1 Classic RAG: the retrieve-then-generate structure

The reference point is **Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks** by Patrick Lewis, Ethan Perez, Aleksandra Piktus, Fabio Petroni, Vladimir Karpukhin, Naman Goyal, Heinrich Küttler, Mike Lewis, Wen-tau Yih, Tim Rocktäschel, Sebastian Riedel, and Douwe Kiela — accepted at **NeurIPS 2020** (arXiv:2005.11401, first submitted May 2020) ✅. The paper's contribution is precisely the structure that defines classic RAG:

- A **parametric memory** — a pre-trained seq2seq generator (BART in the original) — and a **non-parametric memory** — a dense vector index of Wikipedia (FAISS over a DPR retriever).
- At inference, the retriever fetches the top-k passages for the input, and the generator conditions its output on them. The paper compares two formulations: one that conditions on the *same* retrieved passages across the whole generated sequence (RAG-Sequence), and one that can use *different* passages per output token (RAG-Token).
- The crucial architectural property: **retrieval is a fixed, single-shot step that precedes generation.** The generator never decides *whether* to retrieve, never issues a second query, and never critiques the passages it was handed. The passages are simply prepended to the context.

The authors framed the motivation in terms the enterprise still repeats today: parametric-only models "lags behind task-specific architectures" on knowledge-intensive tasks, and "providing provenance for their decisions and updating their world knowledge remain open research problems" ✅ (quoted from the abstract). That is, the two original selling points of RAG — provenance and updatable knowledge — are the same two properties that drive enterprise adoption.

**Precursor:** **REALM: Retrieval-Augmented Language Model Pre-Training** by Kelvin Guu, Kenton Lee, Zora Tung, Panupong Pasupat, and Ming-Wei Chang (arXiv:2002.08909, February 2020), commonly cited as **ICML 2020** ⚠ (I verified the arXiv record and its February 2020 date; the ICML 2020 venue attribution is widely reported but I did not fetch the ICML proceedings page). REALM is the earlier move: a latent knowledge retriever trained *end-to-end* with masked-language-modeling as the learning signal, backpropagating through a retrieval step over millions of documents ✅ (from the abstract). REALM matters to this comparison because it establishes the "retrieval is learnable and differentiable" idea; Lewis et al. then froze the retriever into a cleaner single-shot recipe that became the deployable pattern.

### 1.2 The dense-retrieval lineage: DPR

Classic RAG's retrieval half inherits from **Dense Passage Retrieval for Open-Domain Question Answering** by Vladimir Karpukhin, Barlas Oğuz, Sewon Min, Patrick Lewis, Ledell Wu, Sergey Edunov, Danqi Chen, and Wen-tau Yih — **EMNLP 2020** (arXiv:2004.04906) ✅. The paper shows retrieval can be implemented with dense representations learned by a simple dual-encoder framework, outperforming a strong Lucene-BM25 system by roughly **9%–19% absolute in top-20 passage retrieval accuracy** across open-domain QA datasets, and enabling new state of the art end-to-end ✅ (numbers quoted directly from the abstract). Note the ordering: DPR (April 2020) precedes RAG (May 2020), and the two share authors — the dense retriever is the substrate RAG conditions on. For the enterprise this lineage is the origin of the "embed everything, put it in a vector index, retrieve top-k" default that most RAG stacks still run in production.

### 1.3 The multi-hop problem: why single-shot retrieval breaks

The strongest empirical case that retrieve-then-generate is *insufficient* comes from the multi-hop QA literature.

- **HotpotQA: A Dataset for Diverse, Explainable Multi-hop Question Answering** by Zhilin Yang, Peng Qi, Saizheng Zhang, Yoshua Bengio, William W. Cohen, Ruslan Salakhutdinov, and Christopher D. Manning — **EMNLP 2018** (arXiv:1809.09600) ✅. HotpotQA provides **113k Wikipedia-based question-answer pairs** where questions "require finding and reasoning over multiple supporting documents," with sentence-level supporting facts for supervision and explainability ✅ (quoted from the abstract). This is the benchmark on which the "one retrieval step is not enough" argument is measured.
- **Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions** (IRCoT) by Harsh Trivedi, Niranjan Balasubramanian, Tushar Khot, and Ashish Sabharwal — **ACL 2023** (arXiv:2212.10509) ✅. IRCoT's diagnosis is the pivot point for this whole comparison: "this one-step retrieve-and-read approach is insufficient for multi-step QA. Here, *what to retrieve* depends on *what has already been derived*, which in turn may depend on *what was previously retrieved*" ✅ (quoted from the abstract). The paper interleaves retrieval with chain-of-thought steps, using CoT to guide retrieval and retrieved results to improve CoT, and reports gains of **up to 21 points in retrieval and up to 15 points in downstream QA** on HotpotQA, 2WikiMultihopQA, MuSiQue, and IIRC ✅ (numbers quoted from the abstract). That dependency — retrieval driven by partial reasoning state — is the formal justification for every agentic search loop that follows.

### 1.4 The reason-act-observe loop

**ReAct: Synergizing Reasoning and Acting in Language Models** by Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, and Yuan Cao — **ICLR 2023** (arXiv:2210.03629, first submitted October 2022) ✅. ReAct's mechanism is to generate reasoning traces and task-specific actions *interleaved*, "allowing for greater synergy between the two: reasoning traces help the model induce, track, and update action plans as well as handle exceptions, while actions allow it to interface with external sources, such as knowledge bases or environments, to gather additional information" ✅ (quoted from the abstract). On HotpotQA and Fever it "overcomes issues of hallucination and error propagation prevalent in chain-of-thought reasoning by interacting with a simple Wikipedia API" and produces more interpretable trajectories ✅ (quoted from the abstract). On interactive decision-making benchmarks it reports absolute success-rate gains of **34%** (ALFWorld) and **10%** (WebShop) over imitation and RL baselines ✅.

ReAct is the architectural ancestor of agentic search: the model chooses *whether* and *what* to search, observes the result, updates its plan, and iterates. Where classic RAG runs retrieval exactly once with the user's query as-is, ReAct runs retrieval *inside* the reasoning loop, driven by what the model still needs to know.

### 1.5 Self-critique and self-correction

Two papers convert the loop's "observe" step into an explicit quality gate.

- **Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection** by Akari Asai, Zeqiu Wu, Yizhong Wang, Avirup Sil, and Hannaneh Hajishirzi — **ICLR 2024** (arXiv:2310.11511, October 2023) ✅. Self-RAG trains a single LM to "adaptively retrieve passages on-demand, and generate and reflect on retrieved passages and its own generations using special tokens, called reflection tokens" ✅ (quoted from the abstract). The critique is the contribution: the model emits **retrieval** tokens (is retrieval even needed?), **relevance** tokens (is this passage useful?), and **support/grounding** tokens (does my output follow from the evidence?), and can be steered at inference by those tokens ✅. The paper reports that 7B and 13B Self-RAG models outperform ChatGPT and retrieval-augmented Llama2-chat on open-domain QA, reasoning, and fact verification, with gains in factuality and citation accuracy for long-form generation ✅. Two enterprise-relevant properties fall directly out of this: (1) retrieval is *selective*, not unconditional, and (2) the system carries an internal, inspectable verdict on its own grounding.
- **Corrective Retrieval Augmented Generation (CRAG)** by Shi-Qi Yan, Jia-Chen Gu, Yun Zhu, and Zhen-Hua Ling — arXiv:2401.15884 (January 2024) ⚠ (arXiv record verified; I did not confirm a peer-reviewed venue). CRAG adds a "lightweight retrieval evaluator" that scores retrieved-document quality and "returns a confidence degree based on which different knowledge retrieval actions can be triggered," including falling back to large-scale web search when the static corpus yields only "sub-optimal documents," plus a decompose-then-recompose filter over the retrieved set ✅ (quoted from the abstract). It is explicitly "plug-and-play" and couples to existing RAG pipelines ✅. This is the cleanest bridge from classic RAG toward agentic search: the pipeline stays, but gains a decide-branch on retrieval quality and a live-web fallback.

### 1.6 Query decomposition

Multi-hop and compositional questions force a second family of techniques: split the query before you retrieve.

- **Least-to-Most Prompting Enables Complex Reasoning in Large Language Models** by Denny Zhou, Nathanael Schärli, Le Hou, Jason Wei, Nathan Scales, Xuezhi Wang, Dale Schuurmans, Claire Cui, Olivier Bousquet, Quoc Le, and Ed Chi (arXiv:2205.10625) — commonly cited as **ICLR 2023** ⚠ (arXiv record not re-fetched here; venue attribution is widely reported). The method decomposes a problem into easier subproblems and solves them in sequence, each conditioned on prior answers.
- **Self-Ask / Measuring and Narrowing the Compositionality Gap in Language Models** by Ofir Press, Muru Zhang, Sewon Min, Ludwig Schmidt, Noah A. Smith, and Mike Lewis (arXiv:2210.03350) — cited as **EMNLP 2023 (Findings)** ⚠. Self-Ask has the model explicitly ask and answer follow-up sub-questions before answering the composed question.
- **Decomposed Prompting: A Modular Approach for Solving Complex Tasks** (DecomP) by Tushar Khot, Harsh Trivedi, Matthew Finlayson, Yao Fu, Kyle Richardson, Peter Clark, and Ashish Sabharwal (arXiv:2210.02406) — cited as **ICLR 2023** ⚠. DecomP decomposes a task into sub-tasks handled by separate delegated modules/prompts.

For this guide the exact venue of each decomposition paper is less important than the shared mechanism: **query-as-is** (classic RAG) versus **query-decomposed-then-retrieved-per-subquestion** (agentic and near-agentic RAG). Section 2 treats these under [query handling](#4-the-mechanisms-compared) without re-deriving each prompt recipe — see [query rewriting for RAG](rag/query_rewriting_rag_guide.md) for the single-query reformulation family that sits between the two.

### 1.7 The deep-research pattern

The most product-visible form of agentic search in 2025–2026 is "deep research": an agent that plans a research trajectory, runs many searches, reads sources, may fan out to parallel subagents, and synthesizes a cited report. This is vendor-driven and **non-peer-reviewed** — flagged accordingly.

- **OpenAI Deep Research** was launched in **February 2025** ⚠ (vendor announcement; not peer-reviewed). Public description: an agentic browsing/research mode that plans and executes multi-step web research over minutes and returns a cited report. Cite the vendor page, not a paper.
- **Google Gemini Deep Research** and **Perplexity Deep Research** are the directly comparable vendor offerings ⚠ (vendor-published).
- **Anthropic's "How we built our multi-agent research system"** engineering blog (2025) describes a lead agent spawning parallel subagents that search concurrently and return findings to a synthesizer ⚠ (vendor-published engineering blog, not peer-reviewed). This is the canonical public write-up of the *planning → parallel subagents → synthesis → citation* shape.
- **STORM: Synthesis of Topic Outlines through Retrieval and Multi-perspective Question Asking** by Yijia Shao, Yucheng Jiang, Theodore A. Kanell, Peter Xu, Omar Khattab, and Monica S. Lam — commonly cited as **NAACL 2024** ⚠ (attribution widely reported; verify before citing formally). STORM generates a topic outline, then uses multi-perspective question asking to drive retrieval and build a cited long-form article — the academic analogue of the vendor deep-research products.

The structural signature to retain from all of these: **planning → (possibly parallel) retrieval subagents → synthesis → citation**. That signature is what separates agentic search from a single retrieve-then-generate pass, regardless of vendor.

---

## 2. Classic RAG — The Retrieve-Then-Generate Pipeline and Its Lineage

Classic RAG is a **three-stage dataflow** with a hard boundary between stages. The boundary is the architecture's defining property, and it is also the source of every one of its failure modes and every one of its operational virtues.

```
        ┌─────────────┐      ┌──────────────┐      ┌──────────────┐
query ─▶│  RETRIEVE   │─top-k─▶│  AUGMENT     │─ctx─▶│  GENERATE    │─▶ answer
        │ (dense/BM25)│      │ (concat)     │      │ (frozen LLM) │
        └─────────────┘      └──────────────┘      └──────────────┘
             ▲                                            │
             │                   ┌────────────────┐       │
             └───────────────────│ offline INDEX  │◀──────┘ (no feedback edge in classic RAG)
                                 │ build/embed    │
                                 └────────────────┘
```

### 2.1 The three stages

1. **Indexing (offline).** A corpus is chunked, each chunk embedded by a bi-encoder (the DPR dual-encoder lineage ✅), and the vectors loaded into an ANN index. This is a data pipeline: it has a build, a refresh cadence, and a version. The [vector databases guide](rag/vector_databases_guide.md) covers the storage half; [advanced RAG techniques](rag/advanced_rag_techniques_guide.md) covers the chunking half. For this comparison the key point is that **the index is a materialized, point-in-time artifact** — its coverage is frozen at the last build.
2. **Retrieval (online, single-shot).** The user query is embedded (or run through BM25, or both, then fused) and the top-k nearest chunks are returned. In the pure Lewis formulation there is exactly one retrieval call per query, with the query used **as-is**, and k fixed by configuration ✅ (RAG-Sequence / RAG-Token both retrieve once).
3. **Generation (online).** The k chunks are concatenated into the prompt and a frozen/instructed LLM generates the answer. The generator is a pure function of (query, context); it has no ability to request more context.

### 2.2 The extensions that stayed inside the pipeline

Everything the retrieval literature added between 2020 and 2024 fits *inside* this pipeline without changing its single-shot, feed-forward topology:

| Extension | What it fixes | Where the decision is made | Verification |
|---|---|---|---|
| **Reranking** (cross-encoder over top-k) | Retrieval precision; recall/precision trade-off | Between retrieve and augment; still one retrieval call | Covered in [advanced RAG](rag/advanced_rag_techniques_guide.md); BEIR found re-ranking and late-interaction models "on average achieve the best zero-shot performances … at high computational costs" ✅ (Thakur et al., NeurIPS 2021 D&B, arXiv:2104.08663) |
| **Query rewriting / expansion** | Query–document vocabulary mismatch; ambiguous queries | Before retrieval; still one retrieval call, possibly one rewrite LLM call | [Query rewriting for RAG](rag/query_rewriting_rag_guide.md) |
| **HyDE** (hypothetical document embeddings) | Zero-shot dense retrieval with no relevance labels | Before retrieval: generate a hypothetical answer, embed *that*, retrieve | ✅ **Precise Zero-Shot Dense Retrieval without Relevance Labels** (Gao, Ma, Lin, Callan; arXiv:2212.10496, Dec 2022) — HyDE "outperforms the state-of-the-art unsupervised dense retriever Contriever and shows strong performance comparable to fine-tuned retrievers" ✅ (abstract). Venue commonly cited as ACL 2023 ⚠ |
| **Context compression** | Token cost and "lost in the middle" of long retrieved context | Inside augment: compress retrieved docs into summaries | ✅ **RECOMP: Improving Retrieval-Augmented LMs with Compression and Selective Augmentation** (Xu, Shi, Choi; arXiv:2310.04408, Oct 2023) — compression rates "as low as 6% with minimal loss in performance," and a compressor can return an empty string for **selective augmentation** ✅ (abstract). Venue commonly cited as ICLR 2024 ⚠ |
| **Retrieval-quality gating** | Blindly trusting bad retrieval | A branch: retry/web-search on low confidence | ✅ **CRAG** (Yan et al., arXiv:2401.15884, Jan 2024) — "different knowledge retrieval actions can be triggered" by a confidence degree ✅ |

Each extension keeps the pipeline's core property — **there is still one pass, and the generator still cannot ask for more** — even where it adds a branch (CRAG) or an extra pre-query LLM call (HyDE). That is the line, and it is the line that agentic search crosses.

### 2.3 Why classic RAG won the first wave

The properties that made it the default are exactly the properties enterprise architects still value:

- **Determinism-ish and testability.** Given a frozen index, frozen model, and frozen temperature, the same query yields the same context and (near-)same answer. You can regression-test it.
- **Low latency.** One embed + one ANN lookup + one generation call. Sub-second to a few seconds.
- **Bounded cost.** One generation call whose prompt size is `k × chunk_size`; cost per query is fixed and forecastable.
- **Provenance by construction.** The retrieved chunk IDs *are* the citations; the evidence chain is a byproduct of the pipeline.

The price of those virtues is that the system fails the moment the answer is not in the top-k of a single query against a point-in-time index. That is the multi-hop and freshness problem, and it is where the agentic side enters.

---

## 3. Agentic Search — The Reason-Act-Observe Loop

Agentic search replaces the feed-forward pipeline with a **control loop**: the model decides whether to retrieve, what to retrieve, evaluates what comes back, and decides whether to retrieve again. Formally, it is the ReAct loop ✅ applied to retrieval.

```
        ┌──────────────────────────────────────────────────────┐
        │                     AGENT (LLM)                      │
        │  think ──▶ choose tool+args ──▶ (act) ──▶ observe     │
        └───────────────▲──────────────────────┬───────────────┘
                        │  append observation   │  tool call
                        │  to context           ▼
                   ┌────┴─────────────────────────────────────┐
                   │  search · fetch · browse · SQL · code ·   │
                   │  vector-retriever · APIs · calculator     │
                   └───────────────────────────────────────────┘
                        loop until stop condition
```

### 3.1 The reason-act-observe loop (ReAct)

**ReAct** (Yao et al., ICLR 2023, arXiv:2210.03629) interleaves reasoning traces and actions so that "reasoning traces help the model induce, track, and update action plans as well as handle exceptions, while actions allow it to interface with external sources … to gather additional information" ✅ (abstract). The retrieval-relevant consequence: the model's *next* query is a function of *what it just learned from the last one*. On HotpotQA and Fever it reduced hallucination and error propagation relative to CoT by interacting with a Wikipedia API ✅. ReAct is the minimal viable agentic-search architecture: a loop, a tool interface, and a stop condition.

### 3.2 Multi-hop decomposition (IRCoT, Self-Ask, Least-to-Most)

Multi-hop queries need the retrieval *plan* to be dynamic. Two families:

- **Interleaved retrieve-reason** — IRCoT (Trivedi et al., ACL 2023, arXiv:2212.10509) interleaves retrieval with chain-of-thought steps, on the premise that "what to retrieve depends on what has already been derived" ✅. Reported gains: **up to 21 points retrieval, up to 15 points QA** on HotpotQA, 2WikiMultihopQA, MuSiQue, and IIRC ✅.
- **Explicit query decomposition (prompt-side)** — split the question into sub-questions, then retrieve per sub-question:
  - **Self-Ask** — *Measuring and Narrowing the Compositionality Gap in Language Models* (Press, Zhang, Min, Schmidt, Smith, Lewis; **Findings of EMNLP 2023**, arXiv:2210.03350) ✅. The model "explicitly asks itself (and answers) follow-up questions before answering the initial question," and the structured prompting "lets us easily plug in a search engine to answer the follow-up questions" ✅ (abstract). The paper name-drops the exact mechanism agentic search uses.
  - **Least-to-Most Prompting** (Zhou et al., **ICLR 2023**, arXiv:2205.10625) ✅ — "break down a complex problem into a series of simpler subproblems and then solve them in sequence," each conditioned on prior answers ✅.
  - **Decomposed Prompting (DecomP)** (Khot, Trivedi, Finlayson, Fu, Richardson, Clark, Sabharwal; arXiv:2210.02406) — cited as ICLR 2023 ⚠ (arXiv record not re-fetched here; venue widely reported). Decomposes a task into sub-tasks handled by delegated modules.

The important grouping: Self-Ask and Least-to-Most are **prompt-side decompositions** (no explicit tool loop required), whereas IRCoT is a **genuine loop** (each CoT step triggers retrieval). Agentic search usually combines both — decompose, retrieve per hop, re-decompose based on results.

### 3.3 Reflect-and-refine: self-critique as a control signal

- **Self-RAG** (Asai et al., **ICLR 2024**, arXiv:2310.11511) ✅ makes retrieval **selective and self-graded**: the LM emits reflection tokens for whether to retrieve, whether a passage is relevant, and whether the output is supported by the evidence, and can be steered at inference by those tokens ✅. This is the mechanism that turns "always retrieve top-k" into "retrieve when needed, and check the result."
- **CRAG** (Yan et al., arXiv:2401.15884) ✅ adds a lightweight retrieval evaluator and, on low confidence, falls back to **live web search** — the clearest single-step bridge from pipeline to loop.

Both are **verification mechanisms**: they introduce an explicit grounding/support check that the classic pipeline does not have. That mechanism shows up later as the audit evidence chain the enterprise values.

### 3.4 Tool-using search agents: beyond the retriever

Where ReAct interfaces with "external sources, such as knowledge bases or environments" ✅, production agentic search exposes a *tool menu*: web search, page fetch/browse, a vector retriever over the private corpus, a SQL/warehouse tool, code execution, and internal APIs. The control flow is model-directed — the agent picks the tool per hop. This is the architectural expansion from "retriever-only" to "multi-tool," and it is exactly the capability catalogued in the [MCP discovery guide](mcp_discovery_guide.md) and the [agent harness engineering guide](agent_harness_engineering_guide.md).

### 3.5 The deep-research pattern

The 2025–2026 shape of agentic search is the **deep-research report generator**: plan → parallel subagents → synthesis → citation.

Vendor implementations (all ⚠ **vendor-published, not peer-reviewed**):

- **Google Gemini Deep Research** — announced on the Google blog **11 December 2024** ⚠ (verified: blog.google post "Try Deep Research and our new experimental model in Gemini"). Description: "it creates a multi-step research plan for you to either revise or approve," then "browsing the web the way you do: searching, finding interesting pieces of information and then starting a new search based on what it's learned … It repeats this process multiple times" ✅ (vendor blog, quoted). Note the launch date is **December 2024**, not 2025.
- **OpenAI Deep Research** — vendor page `openai.com/index/introducing-deep-research/` ⚠. Reported launch **February 2025** with a stated typical completion time of "5 to 30 minutes" ⚠ (I could **not** fetch the OpenAI page during this pass — blocked by the scraper; the date and time range are unverified here and flagged in [What Could Not Be Verified](#what-could-not-be-verified)).
- **Perplexity Deep Research** ⚠ — vendor offering in the same category; not fetched here.
- **Anthropic Research** ("Claude Research") — ✅ **verified**: engineering blog **"How we built our multi-agent research system," published 13 June 2025**. Architecture: "a lead agent that coordinates the process while delegating to specialized subagents that operate in parallel," with a `LeadResearcher` that "saves its plan to Memory," spawns subagents, and finally a `CitationAgent` that "processes the documents and research report to identify specific locations for citations" ✅ (vendor blog, quoted). It states the contrast with classic RAG explicitly: "Traditional approaches using Retrieval Augmented Generation (RAG) use static retrieval. That is, they fetch some set of chunks that are most similar to an input query … In contrast, our architecture uses a multi-step search that dynamically finds relevant information, adapts to new findings, and analyzes results." ✅ (vendor blog, quoted). ⚠ This is vendor-published and non-peer-reviewed, but it is the most operationally detailed public account of the pattern and it supplies the token-multiplier numbers used in §5.

Academic antecedent: **STORM** — the paper is **"Assisting in Writing Wikipedia-like Articles From Scratch with Large Language Models"** (Shao, Jiang, Kanell, Xu, Khattab, Lam; **NAACL 2024 Main Conference**, arXiv:2402.14207) ✅. Note the title is the long form; "STORM" is the system name (Synthesis of Topic Outlines through Retrieval and Multi-perspective Question Asking) ✅. STORM models the pre-writing stage by discovering diverse perspectives, simulating multi-perspective question-asking grounded on Internet sources, and curating an outline; it reports a **25% absolute increase** in articles judged "organized" and **10%** in "broad in coverage" over an outline-driven RAG baseline ✅ (abstract).

### 3.6 The RL-trained search-agent line

A 2025 research trend trains the search behaviour into the weights rather than prompt-engineering the loop:

- **Search-R1: Training LLMs to Reason and Leverage Search Engines with Reinforcement Learning** (Jin, Zeng, Yue, Yoon, Arik, Wang, Zamani, Han; arXiv:2503.09516, March 2025) ✅. The LLM "learns to autonomously generate (multiple) search queries during step-by-step reasoning with real-time retrieval," using retrieved-token masking and an outcome-based reward. Reported improvement **41% (Qwen2.5-7B) and 20% (Qwen2.5-3B) over various RAG baselines under the same setting** ✅ (abstract).
- **R1-Searcher** and **DeepRetrieval** ⚠ — reported members of the same RL-for-search-retrieval line; not verified in this pass. Treat as a research direction, not a settled result.

The significance for selection: this line argues that a *trained* agent can beat prompt-and-pipeline RAG on QA accuracy, but it does so by internalizing a loop — so it inherits the loop's latency and non-determinism. It does not resolve the trade-off; it moves the trade-off into the model.

---

## 4. The Mechanisms, Compared

The comparison reduces to eight orthogonal mechanism axes. The table is the reference; the prose below it explains each row and, crucially, the *failure modes* that ride along with each choice.

| Axis | Classic RAG | Agentic Search |
|---|---|---|
| **1. Retrieval strategy** | Single-shot: one query → one top-k set | Iterative / multi-turn: many queries, loop-carried state |
| **2. Reasoning integration** | Generate-only; retriever and generator are separable | Reason-act-observe; reasoning and retrieval are interleaved (ReAct ✅) |
| **3. Query handling** | Query-as-is (optionally one rewrite/HyDE pass) | Decomposition (Self-Ask ✅, Least-to-Most ✅) + re-query on observations (IRCoT ✅) |
| **4. Context assembly** | Fixed top-k, concatenated once | Accumulated/dynamic; grows with hops, must be compressed (RECOMP ✅) or summarized |
| **5. Verification** | None by default; CRAG-style gating ✅ is an add-on | First-class: self-critique/reflection tokens (Self-RAG ✅), relevance + support checks, citation attribution |
| **6. Tool use** | Retriever-only (one index, maybe sparse+dense) | Multi-tool: search, browse/fetch, vector retriever, SQL, code, APIs |
| **7. Control flow** | Fixed pipeline (linear DAG) | Model-directed loop; stop condition is a model/controller decision |
| **8. Failure surface** | Deterministic failure modes (miss, stale, truncation) | Stochastic failure modes (divergence, cost, compounding) |

### 4.1 Retrieval strategy

Classic RAG's single-shot retrieval is a *recall bet*: it commits, on the first try, to the k chunks most similar to the raw query. When the query is one hop from the evidence, the bet wins and the pipeline is unbeatable on cost. When it is multi-hop, the bet loses by construction — IRCoT's result ("what to retrieve depends on what has already been derived" ✅) is precisely the proof. Agentic search converts retrieval into a sequence of *conditioned* bets, each informed by prior observations, which is why Search-R1 can report +41%/+20% over RAG baselines ✅ on QA.

### 4.2 Reasoning integration

In classic RAG the LLM reasons *after* retrieval and *about* the retrieved text; it cannot reason *toward* a retrieval decision. In the agentic loop, reasoning and acting are interleaved, and the reasoning trace is also what makes the loop *auditable* — ReAct's stated benefit includes "interpretable decision traces" ✅. The cost is that the trace is another surface that can be wrong or manipulated (see the failure-modes discussion below and the [agent failure modes guide](llm_agents_failures_production_guide.md)).

### 4.3 Query handling

Three tiers, in increasing power and cost:

1. **Query-as-is** (classic): zero extra LLM calls.
2. **Query transformation** (rewrite / HyDE / expansion): one extra LLM call, still one retrieval pass ✅.
3. **Query decomposition + interleaving** (Self-Ask, Least-to-Most, IRCoT): *n* sub-questions, *n* retrieval hops, re-decomposition on observations ✅.

Note that tier 2 can be bolted onto classic RAG without changing its topology; tier 3 cannot — it *is* the agentic loop.

### 4.4 Context assembly

Classic RAG assembles context once: `ctx = concat(top_k)`. Agentic search assembles context *incrementally and cumulatively* — each observation is appended — which creates the loop's central operational problem: **token growth per hop**. Two mitigations exist and both live in the RAG literature: compression/selective augmentation (RECOMP ✅) and summarization/context-engineering policies (see the [context engineering guide](context_engineering_guide.md)). This is also where the loop's cost multiplier originates: because each hop re-sends the accumulated context, per-query cost is roughly the integral of growing context over the number of hops, not a fixed `k × chunk_size`.

### 4.5 Verification

Classic RAG has no built-in check that the retrieved context actually supports the answer; the retrieved chunk IDs are citations by construction, but nothing verifies entailment. The agentic line adds an explicit verification stage:

- **Self-RAG**'s reflection tokens grade relevance and support ✅.
- **CRAG**'s evaluator grades retrieval quality and triggers corrective actions ✅.
- **Anthropic's** `CitationAgent` "identifies specific locations for citations" so that "all claims are properly attributed to their sources" ✅ — a dedicated verification *agent*, not a byproduct.

Verification is agentic search's single strongest enterprise argument: it is the difference between "the model cited a document" and "a component checked that the claim follows from the document."

### 4.6 Tool use

The retriever-only vs multi-tool distinction is architectural, not cosmetic. A multi-tool loop can combine a private vector index (for proprietary facts) with live web search (for freshness) with SQL (for structured/authoritative numbers) with code execution (for arithmetic on those numbers). Classic RAG is confined to whatever index it was built around. This is the mechanism that makes agentic search the default choice for "research" tasks and classic RAG the default for "lookup" tasks.

### 4.7 Control flow

Classic RAG is a **DAG**: retrieve → augment → generate, with a fixed number of calls. Agentic search is a **loop with a model-chosen termination**. That difference is the root of both the accuracy upside and every operational downside: loops can terminate too early, too late, or never.

### 4.8 The failure modes, side by side

| Failure mode | Classic RAG | Agentic Search |
|---|---|---|
| **Retrieval miss** | Query embeds far from the relevant chunk → answer absent from context | Mitigated by re-querying, but the agent can fixate on a wrong sub-goal |
| **Stale index** | Answer missing because index predates the fact | Mitigated by live web search (CRAG ✅); introduced if the agent adds live sources without governance |
| **Hallucination on thin context** | Model fills gaps when top-k is empty/irrelevant | Reduced by grounding checks (Self-RAG ✅) but not eliminated; verification can itself hallucinate |
| **Top-k truncation** | The answer is at rank k+1 and never seen | N/A — the loop can widen search; instead suffers **loop divergence / rabbit-holing** |
| **Cost** | Fixed per query (`k × chunk`) | Multiplies with hops; Anthropic reports agents use **~4×** the tokens of chat and multi-agent systems **~15×** ⚠ (vendor) |
| **Error compounding** | One bad retrieval → one bad answer | A wrong intermediate conclusion propagates into later queries — ReAct's own motivation was to *reduce* this vs CoT ✅, but the risk is structural |
| **Latency variance** | Low, tight distribution | High, long tail (a loop can run seconds to minutes) ⚠ |
| **Non-determinism / audit** | Reproducible given seed + frozen index | Trajectory is path-dependent; harder to reproduce, requires trace capture |
| **Robustness to noisy corpora** | Degraded but predictable | GARAG-style attacks ("Typos that Broke the RAG's Back," Findings of EMNLP, arXiv:2404.13948 ✅) show minor textual perturbations "significantly devastate the performance of each component and their synergy" ✅ — the loop inherits and can amplify this |

The asymmetry to internalize: **classic RAG fails in ways you can enumerate and test for; agentic search fails in ways you must observe.** That is an operational argument as much as an accuracy argument, and it drives §7's selection framework.

---

## 5. The Trade-Offs, With Evidence

### 5.1 Accuracy on complex / multi-hop queries

**Verdict: agentic (iterative) retrieval is measurably better on multi-hop and compositional queries; classic single-shot retrieval is sufficient — and cheaper — for single-hop lookup.** Evidence:

- ✅ **IRCoT** (ACL 2023, arXiv:2212.10509): interleaving retrieval with CoT improves retrieval **up to 21 points** and downstream QA **up to 15 points** on HotpotQA/2WikiMultihopQA/MuSiQue/IIRC vs one-step retrieve-and-read ✅. This is the cleanest peer-reviewed head-to-head of single-shot vs interleaved retrieval.
- ✅ **MultiHop-RAG** (Tang & Yang, arXiv:2401.15391, Jan 2024): the benchmark paper states outright that "existing RAG methods perform unsatisfactorily in retrieving and answering multi-hop queries" across embedding models and LLMs including GPT-4, PaLM, and Llama2-70B ✅. ⚠ arXiv only; no peer-reviewed venue confirmed.
- ✅ **Search-R1** (arXiv:2503.09516): RL-trained iterative search improves **41% (7B) / 20% (3B)** over RAG baselines on seven QA datasets ✅.
- ✅ **STORM** (NAACL 2024, arXiv:2402.14207): +25% absolute "organized," +10% "broad in coverage" vs an outline-driven RAG baseline ✅.
- ⚠ **Anthropic multi-agent research system** (vendor blog, 13 June 2025): a multi-agent system with Claude Opus 4 lead and Sonnet 4 subagents "outperformed single-agent Claude Opus 4 by 90.2% on our internal research eval" ✅ (vendor-published, non-peer-reviewed — the number is real but not independently validated; no public methodology).

Read the evidence correctly: the *magnitudes* differ by task and setup, and the vendor figure (90.2%) is not commensurable with the benchmark figures. What is solid is the *direction* and the *mechanism*: when the answer requires composing evidence that no single query retrieves, iteration wins, and the win scales with hop count.

### 5.2 Latency

**Verdict: classic RAG is sub-second-to-seconds; agentic search is seconds-to-minutes, with high variance.**

- Classic RAG latency is dominated by one embedding call + one ANN lookup + one generation. ⚠ There is no single canonical published measurement; the practical envelope is well established operationally (single query, one LLM completion). Treat "sub-second–seconds" as the design target, not a cited benchmark.
- Agentic search latency scales with hop count. Google's own Deep Research blog describes the agent working "over the course of a few minutes" ⚠ (vendor). OpenAI's reported "5 to 30 minutes" ⚠ (vendor, unfetched here) is the deep-research tier.
- The structural point: **agentic latency is a distribution, not a number.** A p50 of 30 s can come with a p99 of several minutes. If your SLA assumes a tight distribution, the loop violates it intermittently — see [low-latency GenAI patterns](../low_latency_genai_patterns_guide.md) for the techniques that push back on this (streaming, speculative/parallel search, caching).

### 5.3 Cost

**Verdict: the loop is a multiplicative token cost, not additive.** The mechanism: each hop re-sends the accumulated context, so total tokens ≈ Σ(context_i) over hops, and each hop is at least one more generation call.

The best public, quantified, *vendor-published* evidence:

- ⚠ **Anthropic (13 June 2025)**: "in our data, agents typically use about **4× more tokens** than chat interactions, and multi-agent systems use about **15× more tokens** than chats" ✅ (vendor blog, quoted verbatim). They add the economic caveat explicitly: "For economic viability, multi-agent systems require tasks where the value of the task is high enough to pay for the increased performance." ✅
- ⚠ **Anthropic**: "token usage by itself explains **80% of the variance**" in the BrowseComp browsing-agent eval ✅ (vendor). This reframes cost as, partly, *buying accuracy* — and it means the loop's cost is not waste, it is the mechanism.

There is no peer-reviewed paper that publishes a clean token-multiplier for prompt-engineered agentic search vs single-shot RAG. Any figure you see quoted (2×, 5×, 10×, "order of magnitude") should be treated as ⚠ illustrative unless it names its source and setup. The defensible statement is directional: **agentic search costs a multiple of classic RAG per query, and the multiple rises with query complexity.** Section 9's worked example uses *illustrative* multipliers and says so.

### 5.4 Freshness and coverage

**Verdict: live-web agentic search wins on freshness; the private-index classic pipeline wins on authoritative coverage of governed content.**

- Classic RAG can only answer from its index at its last build; anything newer is invisible (the "stale index" failure).
- CRAG ✅ institutionalizes the fix: fall back to large-scale web search when the static corpus is "sub-optimal." ✅
- Google's Deep Research description is explicitly a browsing loop ("starting a new search based on what it's learned … repeats this process") ✅ (vendor).
- The counterweight: for *regulated* content (a bank's own policies, product terms, regulatory texts), the authoritative source is the governed corpus, not the open web. Freshness helps only if the fresh source is trustworthy and within your data-residency boundary. This is the crux of §8.

### 5.5 Controllability and auditability

**Verdict: this axis favours classic RAG, and it is the axis enterprises under-weight.**

- Classic RAG is reproducible: same index version + same model + same query → same context, hence testable and defensible in a model-risk sense.
- Agentic search is path-dependent. The same query can yield different trajectories; reproducing a specific answer means replaying the *trace* (tool calls, observations, intermediate reasoning). Public vendor systems acknowledge this by building citation and memory subsystems (Anthropic's `CitationAgent` and plan-in-`Memory` ✅), and the research literature acknowledges it via reflection/grounding tokens (Self-RAG ✅).
- The enterprise consequence: **agentic systems require observability investment as a *precondition* of deployment, not an enhancement.** See [observability and guardrails](rag/production_grade_agentic_rag_guide.md), [agent harness engineering](agent_harness_engineering_guide.md), and [AI governance framework](ai_governance_framework_guide.md).

### 5.6 Operational complexity

**Verdict: classic RAG's complexity is in the data plane; agentic search's complexity is in the control plane.**

| Dimension | Classic RAG | Agentic Search |
|---|---|---|
| Build burden | Ingestion/chunking/embedding pipeline, index lifecycle, refresh cadence | Orchestration runtime, tool registry, stop conditions, memory/compaction |
| Eval burden | Retrieval metrics (recall@k, nDCG) + generation faithfulness; see [RAG evaluation](rag/rag_evaluation_methodology_guide.md) | Trajectory/end-state eval, tool-call correctness, cost-per-task; see [LLM evaluation frameworks](llm_evaluation_frameworks_guide.md) and [production-grade agentic RAG](rag/production_grade_agentic_rag_guide.md) |
| Guardrails | Input/output filters, citation checks | All of the above + loop bounds, budget caps, tool allow-lists, HITL escalation |
| Team skill | Data/IR engineering | Distributed systems + agent reliability engineering |
| Maturity of tooling | Very high (vector DBs, managed RAG services) | Improving rapidly but younger; introduces framework lock-in risk |

The honest summary: **you are not choosing between "simple" and "complex" — you are choosing which complexity to own.** Classic RAG pushes the complexity into the index; agentic search pushes it into the runtime.

---

## 6. Hybrid Patterns

Production systems rarely choose an extreme. The literature and vendor systems converge on four hybrid patterns. Pattern 1 has peer-reviewed support; patterns 2–4 are **constructed** from the mechanisms in §3–§4 and vendor practice, and are labelled as such.

### 6.1 The router / adaptive pattern (sourced)

A **classifier routes by query complexity**: easy → no-retrieval or single-shot classic RAG; medium → single-shot with query rewriting; hard/multi-hop → iterative agentic search.

- ✅ **Adaptive-RAG: Learning to Adapt Retrieval-Augmented Large Language Models through Question Complexity** (Jeong, Baek, Cho, Hwang, Park; **NAACL 2024**, arXiv:2403.14403) ✅ — verified. A smaller LM is "trained to predict the complexity level of incoming queries," offering "a balanced strategy, seamlessly adapting between the iterative and single-step retrieval-augmented LLMs, as well as the no-retrieval methods" ✅ (abstract). Crucially, the paper's framing is exactly the enterprise efficiency argument: existing approaches "either handle simple queries with unnecessary computational overhead or fail to adequately address complex multi-step queries" ✅. This is the single most important citation for the hybrid recommendation.

The router is the load-bearing component, and it has its own failure mode: a **misclassification**. A hard query routed to the cheap path produces a confidently wrong answer; an easy query routed to the loop wastes money. So the router needs (a) a conservative bias toward escalation and (b) an eval set that measures routing accuracy separately from answer accuracy.

### 6.2 Agentic-RAG hybrids (constructed)

Three concrete topologies, all built by combining §2 and §3 mechanisms:

1. **Retrieve-then-agent.** Run classic retrieval first to seed the agent's context with the obvious evidence, then let the loop expand with web/SQL/code tools. This limits the agent's cold-start flailing and is the natural shape when the private corpus covers 80% of queries.
2. **Agent-wrapped retriever.** Keep the classic index, but expose it to the agent as *one tool among many*, so the agent can decide how many retrieval hops to make and with what rewritten query. This is the minimal-change path to agentic behaviour: the index pipeline is untouched, only the calling convention changes.
3. **Iterative retrieval over a hybrid index.** Combine sparse (BM25) + dense (DPR-lineage ✅) + a reranker (BEIR ✅), and drive them from an interleaved loop (IRCoT ✅). The reranker does per-hop precision work; the loop does multi-hop recall work.

### 6.3 Cheap-first, escalate-on-failure (constructed, with sourced components)

Answer with the classic pipeline, **verify** the answer, and escalate to the agentic loop **only if the verifier is not confident.**

```
query ─▶ classic RAG ─▶ answer + confidence
                             │
                 ┌───────────┴───────────┐
          confident│                      │not confident
                 ▼                        ▼
              return                 agentic loop ─▶ verified answer
```

The verifier is the load-bearing piece and it is sourced: this is the CRAG evaluator pattern ✅ ("a lightweight retrieval evaluator … returning a confidence degree based on which different knowledge retrieval actions can be triggered") and the Self-RAG support/grounding-token pattern ✅. Escalation-on-low-confidence is the enterprise-optimal cost curve because **most queries are easy**, so most queries never pay the loop tax; only the hard tail does. The risk is a verifier that is confidently wrong — which argues for escalating on *retrieval* signals (low top-1 score, contradictory chunks, a query classified as multi-hop) as well as on answer signals.

### 6.4 Cache and memory layers (constructed)

Two amortizations sit across all patterns:

- **Semantic cache** for the classic path — identical/near-identical queries return the cached answer, collapsing the cost of the easy path toward zero.
- **Episodic/session memory** for the agent path — store prior research plans and findings (the pattern Anthropic describes as saving the plan to `Memory` ✅) so a follow-up question does not re-run the whole loop. This is a context-management concern; see [context engineering](context_engineering_guide.md) and [beyond RAG §Paradigm 3 — memory systems](rag/beyond_rag_guide.md).

### 6.5 The hybrid recommendation

For most enterprises the defensible default is **§6.1 router + §6.3 escalate-on-failure**:

```
                         ┌──────────────────────────────────────┐
              query ────▶│  COMPLEXITY / INTENT CLASSIFIER      │
                         └───────┬───────────────┬──────────────┘
                    simple/     │               │  multi-hop /
                    single-hop  ▼               ▼  fresh / comparative
                        ┌──────────────┐   ┌────────────────────┐
                        │ CLASSIC RAG  │   │ AGENTIC SEARCH     │
                        │ (index,tok)  │   │ (loop, multi-tool) │
                        └──────┬───────┘   └─────────┬──────────┘
                               │                     │
                        ┌──────▼───────┐             │
                        │  VERIFIER    │─escalate────▶┘
                        └──────┬───────┘
                               │ ok
                               ▼
                        answer + evidence chain
```

Keep classic RAG as the **default** and the agentic loop as the **escalation path**. This preserves the cheap path's latency and cost for the majority, keeps the audited index authoritative, and pays the loop's price only where it demonstrably buys accuracy.

---

## 7. The Selection Framework

This section is a **constructed** decision framework (not sourced from a single paper) synthesized from the verified mechanisms and evidence in §4–§5. Where a criterion is anchored in a citation, it is marked; where it is architectural judgement, it is stated plainly.

### 7.1 The axes

| Axis | Question to answer | Favours classic RAG | Favours agentic search |
|---|---|---|---|
| **Query complexity** | Single-hop lookup, or multi-hop/comparative/exploratory? | Single-hop; answer lives in one chunk | Multi-hop; needs composition (IRCoT ✅) or decomposition (Self-Ask ✅) |
| **Query volume / throughput** | Hundreds/sec or tens/minute? | High volume, tight per-query budget | Low volume, high value per query |
| **Latency tolerance** | Sub-second SLA, or minutes acceptable? | Strict SLA | Minutes acceptable (deep research ⚠) |
| **Cost tolerance** | Per-query budget in tokens/cents? | Tight budget, predictable | Comfortable budget; Anthropic ⚠ reports ~4× (agent) / ~15× (multi-agent) tokens |
| **Freshness requirement** | Answer must reflect today's world? | Answer is in governed corpus | Live web/news/rates required (CRAG ✅) |
| **Auditability requirement** | Must reproduce/exhibit the evidence chain? | High auditability, deterministic | Requires trace capture; higher governance burden |
| **Failure cost** | What does a wrong answer cost? | Low-stakes lookups | High-stakes analysis where a miss is expensive |

### 7.2 The decision table

| Workload profile | Recommended architecture | Rationale |
|---|---|---|
| High-volume FAQ / product lookup, strict SLA, low failure cost | **Classic RAG** (+ semantic cache) | Single-hop, deterministic, cheap; loop would add latency and cost for no accuracy gain ✅ (Adaptive-RAG's "unnecessary computational overhead") |
| Policy/regulatory lookup over a governed corpus | **Classic RAG + reranker**, audited index | Authoritative source is the corpus, not the web; provenance by construction |
| Multi-hop analyst queries (compose 2–5 sources) | **Agentic search**, or router that escalates to it | IRCoT ✅ / MultiHop-RAG ✅ show single-shot underperforms |
| Fresh-market / competitor research | **Agentic search (web tools)** | Requires live web + iteration ✅ (CRAG, vendor deep research) |
| Mixed population (both of the above) | **Router (§6.1) + escalate-on-failure (§6.3)** | Adaptive-RAG ✅; most queries stay cheap, the hard tail escalates |
| High-stakes, must-audit, multi-hop | **Agentic search with mandatory citation/evidence chain + HITL** | Verification mechanism is the point (Self-RAG ✅, CitationAgent ✅); human sign-off on material outputs |

### 7.3 Decision-tree reasoning

```
1. Is the answer available in a single governed chunk (one hop)?
     YES ─▶ Classic RAG. Stop. (Do not pay the loop tax.)
     NO  ─┐
2.        Does the answer require freshness beyond the index refresh cycle?
     YES ─▶ Agentic search with live-web tool (or CRAG-style web fallback).
     NO  ─┐
3.        Does the answer require composing ≥2 sources or ≥2 reasoning hops?
     YES ─▶ Iterative/decomposed retrieval. If audit required, enforce a
             citation/evidence chain and cap the loop.
     NO  ─▶ Classic RAG with query rewriting/reranking.
4. Does the population mix easy and hard queries at volume?
     YES ─▶ Router + escalate-on-failure. Measure routing accuracy separately.
5. Is the per-query cost/latency SLA tight?
     YES ─▶ Prefer the cheap path + escalation; only run the full loop off the
             critical path (async "deep research" report).
```

### 7.4 The two-question heuristic

If a design review gives you thirty seconds, ask two questions:

1. **"Is the answer one hop away?"** If yes, classic RAG — always. The entire accuracy case for agentic search is about *multi-hop* and *freshness*; if neither applies, the loop is pure cost.
2. **"Can I afford a non-deterministic, minutes-long, audit-heavy process for this query class?"** If no, classic RAG (or router+escalation with a deterministic cheap path). If yes, agentic search earns its keep.

Everything else is tuning.

---

## 8. Enterprise Guidance for Regulated and Banking Contexts

This section condenses to the governance guides; it does **not** re-derive policy. The points below are the *retrieval-architecture-specific* implications.

### 8.1 Auditability: the evidence chain is the deliverable

For regulated workloads the answer is not enough — the *derivation* must be exhibitable. Agentic search makes this concrete and expensive:

- Capture, per query: the routing decision, every tool call with arguments, every observation, the reasoning trace, and the final citations. This is the loop's version of "provenance by construction."
- Classic RAG's evidence chain is the retrieved chunk IDs plus the index version. Cheaper to produce; narrower in coverage.
- The enterprise bar is *reproducibility*: given the trace, an auditor (or a regression test) can replay the derivation. See the [AI governance framework](ai_governance_framework_guide.md) and [production-grade agentic RAG — observability](rag/production_grade_agentic_rag_guide.md).

### 8.2 Cost governance: budgets, caps, observability

Because the loop's cost is multiplicative (§5.3, Anthropic ⚠ ~4×/~15× tokens), cost must be a **first-class, enforced control**, not a report:

- **Per-query token/tool-call budget** with a hard cap and a graceful degradation (return the best-so-far answer with a "budget exhausted" flag).
- **Loop bound**: max hops, max wall-clock, max subagents (Anthropic's own scaling rule — "simple fact-finding requires just 1 agent with 3–10 tool calls," complex research "more than 10 subagents" ✅ — is a budget policy, not just a prompt hint).
- **Cost attribution** per team/use-case, plus anomaly alerting, because a misconfigured loop can spend orders of magnitude more than forecast.

### 8.3 Data residency and PII

- **Keep retrieval inside the trust boundary.** Live-web tools must not receive PII, customer data, or confidential deal terms; query sanitization and an allow-list of external tools are the guardrails.
- **Residency constrains the index and the tools.** A corpus that must stay in-region constrains where the agent runtime and any tool endpoints run; a vendor deep-research agent that browses the open web is generally *out of bounds* for confidential data, however good its answer quality.
- See the [banking compliance guide](../../banking/ai_genai_banking_compliance_guide.md).

### 8.4 Vendor lock-in

- **Index lock-in** (embeddings + vector store + chunking conventions) is real but largely portable: embeddings can be re-computed, and the corpus is yours.
- **Agent-runtime lock-in is stickier**: tool schemas, orchestration framework, memory format, and evaluation harness are bespoke. Mitigate by standardizing on open tool-interface conventions (see [MCP discovery](mcp_discovery_guide.md)) and keeping the retrieval index provider-independent.
- The asymmetry matters: adopting agentic search usually deepens coupling to a runtime, whereas classic RAG couples you mainly to a data store.

### 8.5 Human-in-the-loop

- **Classic RAG:** HITL is the review step *after* the answer; the pipeline itself is safe to run unattended.
- **Agentic search:** HITL is a **loop control** — approval gates before high-consequence tool calls (e.g. writing to a system of record, calling a live trading/market data API, sending an outbound action), and mandatory human sign-off on material analytical outputs. The failure mode that forces this is loop divergence and error compounding (§4.8). See [autonomous agents](autonomous_agents_guide.md) and [production-ready LLM agents](production_ready_llm_agents_guide.md).

### 8.6 Evals and regression testing

- Run **two eval layers**: retrieval/generation evals for the classic path ([RAG evaluation methodology](rag/rag_evaluation_methodology_guide.md); RAGAS ✅ — reference-free metrics for retrieval quality, faithfulness, and generation quality) and **trajectory/end-state evals** for the agent path ([LLM evaluation frameworks](llm_evaluation_frameworks_guide.md)).
- **Measure routing accuracy separately** (§6.1) — a router that misclassifies can silently destroy accuracy while looking efficient.
- **Pin and version** the index, the prompts, the tools, and the model; regression-test on a frozen set so a model upgrade or prompt edit cannot silently regress the escalation threshold.
- **Adversarial/robustness testing** belongs in the gate: GARAG-style low-level perturbations degrade RAG components and "their synergy" ✅ (arXiv:2404.13948), and the loop must be tested with noisy corpora, prompt injections in retrieved documents, and hostile tool outputs.

---

## 9. Worked Example — The Cymbal Bank Knowledge Assistant

> **Illustrative, not measured.** Cymbal Bank is an illustrative bank persona. Every number in this section is a **constructed** planning figure chosen to make the trade-off arithmetic visible. None of the latency, cost, or accuracy numbers below come from a published benchmark; they are labeled ⚠ illustrative throughout. The *mechanisms* and *directions* they illustrate are sourced in §4–§5.

### 9.1 The workload

Cymbal Bank's knowledge assistant serves three distinct workloads, and they are not the same architecture problem:

| Workload | Shape | Typical query | Fraction of traffic ⚠ illustrative | Latency need | Freshness need |
|---|---|---|---|---|---|
| **W1 — Product & policy lookup** | Single-hop | "What is the fee on the Cymbal Premier account?" | ~70% | Interactive (≤3 s) | Governed corpus (refreshed nightly) |
| **W2 — Multi-hop regulatory analysis** | Multi-hop, compositional | "Which of our onboarding controls are affected by the [X] rule change, and where do our policy docs diverge?" | ~20% | Analyst-tolerated (≤60–90 s) | Governed corpus + regulatory feeds |
| **W3 — Fresh-market research** | Exploratory, live | "Summarize competitor pricing changes announced this month and their likely impact on our deposit products." | ~10% | Minutes acceptable (async report) | Live web required |

The critical observation: **W1 dominates volume but W2/W3 dominate risk.** A single architecture sized for W1 underserves W2/W3; an architecture that always runs the loop overspends on W1.

### 9.2 The routing design

The design is **router (§6.1) + escalate-on-failure (§6.3)**, with W3 optionally run as an async deep-research job.

```
                          ┌─────────────────────────────────────┐
  query ────────────────▶ │ CLASSIFIER (complexity + intent)    │
                          └────┬──────────┬──────────┬──────────┘
                      W1 simple│   W2 multi│   W3 fresh│
                               ▼           ▼            ▼
                    ┌──────────────┐ ┌────────────┐ ┌──────────────────┐
                    │ CLASSIC RAG  │ │ AGENTIC    │ │ ASYNC DEEP       │
                    │ product/policy│ │ SEARCH     │ │ RESEARCH (loop,  │
                    │ index + rerank│ │ + SQL +    │ │ web tool, multi- │
                    │              │ │ regulatory │ │ agent fan-out)   │
                    └──────┬───────┘ │ feeds      │ └────────┬─────────┘
                           │         └─────┬──────┘          │
                    ┌──────▼───────┐       │                 │
                    │ VERIFIER     │─escalate (W2)──▶         │
                    └──────┬───────┘                         │
                           │ ok                              │
                           ▼                                 ▼
                    inline answer + evidence chain   report + evidence chain
```

Routing rules for Cymbal Bank (constructed):

- **Default to W1 (classic RAG).** Cheapest, fastest, most auditable. Escalate on low verifier confidence or a classifier signal of multi-hop intent.
- **W2 runs the agentic loop with a hard budget cap** (max hops, max wall-clock) and produces a mandatory evidence chain. It is *allowed* to be slow; it is *not* allowed to be unreproducible.
- **W3 runs asynchronously**, off the interactive critical path, as a deep-research report. Users get a notification when the cited report is ready — the vendor deep-research UX ⚠.
- **No live-web tool for W1/W2.** Confidential content stays inside the trust boundary (§8.3); only W3's public-web research touches external sources, and its queries are sanitized.

### 9.3 The cost/quality trade-off — illustrative numbers

The table below is a **constructed** planning model. The *structure* (loop multiplies cost; routing keeps the average near the cheap path) is the sourced insight; the *values* are illustrative ⚠ and should be replaced with Cymbal Bank's own telemetry after a pilot.

| Workload | Architecture | Latency (p50 / p99) ⚠ | Relative cost/query (classic = 1×) ⚠ | Accuracy on workload ⚠ | Evidence chain |
|---|---|---|---|---|---|
| W1 (70%) | Classic RAG + rerank | 1.5 s / 3 s | **1×** | high (single-hop) | chunk IDs + index version |
| W1 escalated (est. 3% of W1) | Agentic loop | 40 s / 120 s | ~6–15× | higher on the hard tail | full trace |
| W2 (20%) | Agentic search + SQL | 45 s / 150 s | ~8–15× | materially better than single-shot ✅ direction (IRCoT) | full trace + citations |
| W3 (10%) | Async deep research | 5–30 min ⚠ (vendor) | ~15×+ ⚠ (multi-agent, Anthropic) | best on exploratory/multi-source | multi-agent trace + per-claim citations |

Blended cost, walking the arithmetic: `0.70×1 + 0.30×1 (+~0.03 escalation × ~10) + 0.20×11 + 0.10×15 ≈ **4.5×**` the all-classic baseline ⚠ illustrative. Run the same arithmetic assuming *all* traffic on the loop and it is `~10–15×` — the router is what keeps the blended multiplier in single digits. This is the quantitative version of the router argument.

The decision this table encodes: **pay the loop's multiple on the 30% of traffic that needs it, not on 100%.** If W2/W3 were less than ~1% of traffic, even the router might not be worth the orchestration complexity — send those to a separate analyst tool.

### 9.4 The audit path

For a regulated bank the architecture must produce, per material answer:

1. **Routing record** — which workload class, which classifier version, which confidence, and whether it escalated.
2. **For W1:** the retrieved chunk IDs, similarity scores, reranker scores, and the pinned index version. Reproducible by re-running the same query against the same index.
3. **For W2/W3:** the full trace — plan, every tool call and argument, every observation, the reasoning steps, loop bounds hit, and the final per-claim citations. Non-reproducible by seed alone; reproducible only by **replaying the trace**, which is why trace capture is mandatory.
4. **Verifier verdict** — the grounding/support check result (Self-RAG-style ✅) and, for material outputs, the human sign-off record.
5. **Cost record** — tokens and tool calls consumed, against the per-query budget.

Retention and residency follow the governance guides, not this one: see the [AI governance framework](ai_governance_framework_guide.md), the [banking compliance guide](../../banking/ai_genai_banking_compliance_guide.md), and [production-grade agentic RAG — observability](rag/production_grade_agentic_rag_guide.md). The design principle specific to this comparison: **the evidence chain's cost is proportional to the architecture.** Classic RAG's chain is cheap because it is a byproduct; the loop's chain is expensive because it must be *instrumented*. Budget for that instrumented trace up front — it is a precondition of the W2/W3 deployment, not a nice-to-have.

---

## 10. Claims Audit

Every load-bearing empirical claim in this guide, with its verification status.

| # | Claim | Status | Source |
|---|---|---|---|
| 1 | RAG's canonical structure is parametric + non-parametric memory, single-shot retrieval then generation (RAG-Sequence / RAG-Token) | ✅ | Lewis et al., NeurIPS 2020, arXiv:2005.11401 (abstract fetched) |
| 2 | RAG's stated motivations include provenance and updatable world knowledge | ✅ | Lewis et al. abstract (quoted) |
| 3 | REALM trains a latent retriever end-to-end with MLM signal | ✅ | Guu et al., arXiv:2002.08909 (abstract fetched) |
| 4 | REALM is ICML 2020 | ⚠ | Widely reported; ICML proceedings page not fetched |
| 5 | DPR uses a dual-encoder dense retriever, +9–19% absolute top-20 accuracy over Lucene-BM25 | ✅ | Karpukhin et al., EMNLP 2020, arXiv:2004.04906 (abstract fetched) |
| 6 | HotpotQA = 113k multi-hop QA pairs with sentence-level supporting facts | ✅ | Yang et al., EMNLP 2018, arXiv:1809.09600 (abstract fetched) |
| 7 | One-step retrieve-and-read is insufficient for multi-step QA; "what to retrieve depends on what has already been derived" | ✅ | IRCoT, Trivedi et al., ACL 2023, arXiv:2212.10509 (abstract fetched) |
| 8 | IRCoT improves retrieval up to 21 points and QA up to 15 points | ✅ | IRCoT abstract (quoted) |
| 9 | ReAct interleaves reasoning and acting; improves interpretability; reduces hallucination vs CoT on HotpotQA/Fever | ✅ | Yao et al., ICLR 2023, arXiv:2210.03629 (abstract fetched) |
| 10 | ReAct gains: 34% ALFWorld, 10% WebShop absolute success rate | ✅ | ReAct abstract (quoted) |
| 11 | Self-RAG uses reflection tokens (retrieve/relevance/support) and is steerable at inference | ✅ | Asai et al., ICLR 2024, arXiv:2310.11511 (abstract fetched) |
| 12 | Self-RAG (7B/13B) beats ChatGPT and retrieval-augmented Llama2-chat on QA/reasoning/fact verification | ✅ | Self-RAG abstract (quoted) |
| 13 | CRAG uses a lightweight retrieval evaluator + web search fallback + decompose-then-recompose | ✅ | Yan et al., arXiv:2401.15884 (abstract fetched). Peer-reviewed venue ⚠ unconfirmed |
| 14 | Self-Ask is Findings of EMNLP 2023 | ✅ | arXiv:2210.03350 "Comments: To appear at Findings of EMNLP 2023" (fetched) |
| 15 | Least-to-Most is ICLR 2023 | ✅ | arXiv:2205.10625 "Comments: ICLR 2023" (fetched) |
| 16 | DecomP is ICLR 2023 | ⚠ | Widely reported; arXiv:2210.02406 not re-fetched |
| 17 | HyDE generates a hypothetical document and embeds it; beats Contriever, comparable to fine-tuned retrievers | ✅ | Gao et al., arXiv:2212.10496 (abstract fetched). ACL 2023 venue ⚠ |
| 18 | RECOMP compresses retrieved context to as low as 6% with minimal loss; supports selective augmentation | ✅ | Xu et al., arXiv:2310.04408 (abstract fetched). ICLR 2024 venue ⚠ |
| 19 | BEIR: BM25 robust baseline; re-ranking/late-interaction best zero-shot but costly | ✅ | Thakur et al., NeurIPS 2021 D&B, arXiv:2104.08663 (abstract fetched) |
| 20 | RAGAS provides reference-free RAG evaluation metrics | ✅ | Es et al., arXiv:2309.15217 (abstract fetched) |
| 21 | MultiHop-RAG: existing RAG methods perform unsatisfactorily on multi-hop queries | ✅ | Tang & Yang, arXiv:2401.15391 (abstract fetched). Venue ⚠ |
| 22 | Adaptive-RAG trains a classifier to route by query complexity between no-retrieval / single-step / iterative | ✅ | Jeong et al., NAACL 2024, arXiv:2403.14403 (abstract fetched) |
| 23 | Search-R1 RL-trains multi-search reasoning; +41% (7B) / +20% (3B) over RAG baselines | ✅ | Jin et al., arXiv:2503.09516 (abstract fetched) |
| 24 | GARAG shows low-level perturbations devastate RAG components and their synergy | ✅ | Cho et al., Findings of EMNLP, arXiv:2404.13948 (abstract fetched) |
| 25 | STORM = "Assisting in Writing Wikipedia-like Articles From Scratch…"; NAACL 2024; +25% organized / +10% coverage | ✅ | Shao et al., arXiv:2402.14207 (abstract fetched; comments confirm NAACL 2024 Main) |
| 26 | Gemini Deep Research announced 11 Dec 2024; multi-step plan + iterative browsing | ✅ (vendor) | blog.google post (fetched) |
| 27 | Anthropic Research: lead agent + parallel subagents + CitationAgent; "static retrieval" vs "multi-step search" | ✅ (vendor) | anthropic.com engineering blog, 13 Jun 2025 (fetched) |
| 28 | Anthropic: agents ~4× tokens vs chat; multi-agent ~15× tokens vs chats | ⚠ (vendor) | Anthropic blog (quoted). Vendor-published, not peer-reviewed |
| 29 | Anthropic: multi-agent beats single-agent by 90.2% on internal research eval; token usage explains 80% of BrowseComp variance | ⚠ (vendor) | Anthropic blog (quoted). Internal eval, no public methodology |
| 30 | OpenAI Deep Research launched Feb 2025 with 5–30 min runtime | ❌/⚠ | Page could not be fetched (scraper blocked). Reported but unverified here |
| 31 | Perplexity Deep Research exists as a vendor offering | ⚠ | Assumed from vendor presence; not fetched |
| 32 | R1-Searcher / DeepRetrieval are RL-search agents | ⚠ | Not verified this pass |
| 33 | Classic RAG latency = sub-second–seconds; agentic = seconds–minutes with long tail | ⚠ | Directional architectural claim; no single canonical measurement cited |
| 34 | The Cymbal Bank tables (latencies, cost multipliers, blended 4.5×) | ⚠ illustrative | **Constructed** planning figures, explicitly not measurements |

---

## What Could Not Be Verified

Honest list of gaps and blocked verifications in this pass:

1. **OpenAI Deep Research page.** `https://openai.com/index/introducing-deep-research/` would not scrape ("All scraping engines failed"). The **February 2025** launch date and the **"5 to 30 minutes"** runtime figure are therefore ⚠ reported-but-unverified here. Anyone citing them should fetch the vendor page directly.
2. **Perplexity Deep Research.** Not fetched. The product's existence and description are asserted from general knowledge only — treat as ⚠. Not used for any load-bearing claim.
3. **R1-Searcher and DeepRetrieval.** Named in the task as members of the RL-trained search-agent line; I verified **Search-R1** ✅ but did **not** fetch pages for R1-Searcher or DeepRetrieval. Their specific claims are unverified here.
4. **Venue attributions for several papers.** REALM → ICML 2020 ⚠; CRAG → peer-reviewed venue ⚠; HyDE → ACL 2023 ⚠; RECOMP → ICLR 2024 ⚠; DecomP → ICLR 2023 ⚠; MultiHop-RAG → peer-reviewed venue ⚠. The **arXiv records are verified**; the conference attributions are widely reported but were not confirmed against the proceedings pages in this pass.
5. **A clean, peer-reviewed token-cost multiplier for agentic search vs single-shot RAG.** No such paper was found. The only quantified figure located is Anthropic's ⚠ vendor blog (~4× agent vs chat, ~15× multi-agent vs chat). Any other multiplier quoted in this guide is labeled illustrative.
6. **Independent (non-vendor) latency benchmarks for deep research.** None found. The only latency descriptions are vendor-published (Google "a few minutes"; OpenAI reported "5–30 minutes"). Treat both as ⚠.
7. **A single canonical peer-reviewed head-to-head of "prompt-engineered agentic RAG vs classic RAG."** The closest verified comparisons are **IRCoT** (single-shot vs interleaved, ✅) and **Search-R1** (RL-loop vs RAG baselines, ✅), plus **Adaptive-RAG** (✅) on the routing side. No single paper cleanly isolates the "loop vs pipeline" variable across enterprise-style corpora; the evidence is assembled from adjacent results.
8. **Accuracy of agentic search on *private, governed* corpora.** All verified benchmarks use open datasets (HotpotQA, 2WikiMultihopQA, MuSiQue, IIRC, news corpora). The transfer of these gains to a bank's policy/regulatory corpus is an architectural expectation, not a measured result — which is exactly why §8.6 insists on your own eval harness.

If any of the above matters to a decision, verify it at the primary source before relying on it.

---

## Glossary

| Term | Definition |
|---|---|
| **Agentic search** | Retrieval driven by a model-directed reason-act-observe loop, where the model chooses whether/what to retrieve and iterates on observations (ReAct-lineage). |
| **Classic RAG** | The retrieve-then-generate pipeline: single-shot retrieval of top-k chunks, concatenated into the prompt, followed by one generation pass (Lewis et al. 2020). |
| **ANN index** | Approximate-nearest-neighbour vector index (e.g. FAISS/HNSW/IVF) holding chunk embeddings for similarity search. |
| **CRAG** | Corrective RAG — a lightweight retrieval evaluator that triggers different retrieval actions (including web-search fallback) by confidence (Yan et al.). |
| **Deep research** | An agentic pattern that plans a research trajectory, runs (possibly parallel) retrieval subagents, and synthesizes a cited report (OpenAI/Gemini/Perplexity products; Anthropic multi-agent system). |
| **Dense retrieval / DPR** | Retrieval by learned dense embeddings from a dual-encoder, contrasted with sparse lexical retrieval (BM25). |
| **DPR** | Dense Passage Retrieval — the dual-encoder bi-encoder retriever lineage (Karpukhin et al. 2020). |
| **Escalate-on-failure** | A hybrid where the cheap classic path answers first and the agentic loop runs only when a verifier flags low confidence. |
| **Evidence chain** | The recorded derivation of an answer: retrieved IDs/trace, tool calls, verifier verdict, and citations — the audit artifact. |
| **GARAG** | Genetic Attack on RAG — a low-level-perturbation attack showing RAG brittleness to noisy documents (Cho et al.). |
| **HyDE** | Hypothetical Document Embeddings — generate a hypothetical answer, embed it, and retrieve with that vector (Gao et al.). |
| **IRCoT** | Interleaving Retrieval with Chain-of-Thought — alternate CoT steps and retrieval for multi-step QA (Trivedi et al.). |
| **Multi-hop query** | A question requiring composition of evidence from ≥2 sources or reasoning steps (HotpotQA lineage). |
| **ReAct** | Reason + Act — interleave reasoning traces and tool actions so the plan updates on observations (Yao et al.). |
| **Reflection tokens** | Self-RAG's special tokens marking whether to retrieve, passage relevance, and output support — enabling steerable inference. |
| **Reranker** | A cross-encoder (or late-interaction model) that re-scores retrieved candidates for precision, at higher cost than the first-stage retriever. |
| **Router (adaptive RAG)** | A classifier that assigns a query to the cheapest sufficient strategy (no-retrieval / single-step / iterative) by predicted complexity (Adaptive-RAG). |
| **Self-RAG** | A framework where an LM adaptively retrieves and self-critiques with reflection tokens (Asai et al.). |
| **Top-k truncation** | The classic failure where the needed evidence ranks below k and is never retrieved. |
| **STORM** | A pre-writing system that builds a topic outline via retrieval and multi-perspective question asking (Shao et al.). |

---

## References

Primary literature (arXiv records verified in this pass):

1. Lewis, P., Perez, E., Piktus, A., Petroni, F., Karpukhin, V., Goyal, N., Küttler, H., Lewis, M., Yih, W., Rocktäschel, T., Riedel, S., Kiela, D. — *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*, NeurIPS 2020. https://arxiv.org/abs/2005.11401 ✅
2. Guu, K., Lee, K., Tung, Z., Pasupat, P., Chang, M.-W. — *REALM: Retrieval-Augmented Language Model Pre-Training*, arXiv:2002.08909 (ICML 2020 ⚠). https://arxiv.org/abs/2002.08909 ✅
3. Karpukhin, V., Oğuz, B., Min, S., Lewis, P., Wu, L., Edunov, S., Chen, D., Yih, W. — *Dense Passage Retrieval for Open-Domain Question Answering*, EMNLP 2020. https://arxiv.org/abs/2004.04906 ✅
4. Yang, Z., Qi, P., Zhang, S., Bengio, Y., Cohen, W. W., Salakhutdinov, R., Manning, C. D. — *HotpotQA: A Dataset for Diverse, Explainable Multi-hop Question Answering*, EMNLP 2018. https://arxiv.org/abs/1809.09600 ✅
5. Yao, S., Zhao, J., Yu, D., Du, N., Shafran, I., Narasimhan, K., Cao, Y. — *ReAct: Synergizing Reasoning and Acting in Language Models*, ICLR 2023. https://arxiv.org/abs/2210.03629 ✅
6. Trivedi, H., Balasubramanian, N., Khot, T., Sabharwal, A. — *Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions* (IRCoT), ACL 2023. https://arxiv.org/abs/2212.10509 ✅
7. Asai, A., Wu, Z., Wang, Y., Sil, A., Hajishirzi, H. — *Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection*, ICLR 2024. https://arxiv.org/abs/2310.11511 ✅
8. Yan, S.-Q., Gu, J.-C., Zhu, Y., Ling, Z.-H. — *Corrective Retrieval Augmented Generation* (CRAG), arXiv:2401.15884 (venue ⚠). https://arxiv.org/abs/2401.15884 ✅
9. Press, O., Zhang, M., Min, S., Schmidt, L., Smith, N. A., Lewis, M. — *Measuring and Narrowing the Compositionality Gap in Language Models* (Self-Ask), Findings of EMNLP 2023. https://arxiv.org/abs/2210.03350 ✅
10. Zhou, D., Schärli, N., Hou, L., Wei, J., Scales, N., Wang, X., Schuurmans, D., Cui, C., Bousquet, O., Le, Q., Chi, E. — *Least-to-Most Prompting Enables Complex Reasoning in Large Language Models*, ICLR 2023. https://arxiv.org/abs/2205.10625 ✅
11. Khot, T., Trivedi, H., Finlayson, M., Fu, Y., Richardson, K., Clark, P., Sabharwal, A. — *Decomposed Prompting: A Modular Approach for Solving Complex Tasks* (DecomP), arXiv:2210.02406 (ICLR 2023 ⚠). https://arxiv.org/abs/2210.02406 ⚠
12. Gao, L., Ma, X., Lin, J., Callan, J. — *Precise Zero-Shot Dense Retrieval without Relevance Labels* (HyDE), arXiv:2212.10496 (ACL 2023 ⚠). https://arxiv.org/abs/2212.10496 ✅
13. Xu, F., Shi, W., Choi, E. — *RECOMP: Improving Retrieval-Augmented LMs with Compression and Selective Augmentation*, arXiv:2310.04408 (ICLR 2024 ⚠). https://arxiv.org/abs/2310.04408 ✅
14. Thakur, N., Reimers, N., Rücklé, A., Srivastava, A., Gurevych, I. — *BEIR: A Heterogenous Benchmark for Zero-shot Evaluation of Information Retrieval Models*, NeurIPS 2021 Datasets & Benchmarks. https://arxiv.org/abs/2104.08663 ✅
15. Es, S., James, J., Espinosa-Anke, L., Schockaert, S. — *Ragas: Automated Evaluation of Retrieval Augmented Generation*, arXiv:2309.15217. https://arxiv.org/abs/2309.15217 ✅
16. Tang, Y., Yang, Y. — *MultiHop-RAG: Benchmarking Retrieval-Augmented Generation for Multi-Hop Queries*, arXiv:2401.15391 (venue ⚠). https://arxiv.org/abs/2401.15391 ✅
17. Jeong, S., Baek, J., Cho, S., Hwang, S. J., Park, J. C. — *Adaptive-RAG: Learning to Adapt Retrieval-Augmented Large Language Models through Question Complexity*, NAACL 2024. https://arxiv.org/abs/2403.14403 ✅
18. Jin, B., Zeng, H., Yue, Z., Yoon, J., Arik, S., Wang, D., Zamani, H., Han, J. — *Search-R1: Training LLMs to Reason and Leverage Search Engines with Reinforcement Learning*, arXiv:2503.09516. https://arxiv.org/abs/2503.09516 ✅
19. Cho, S., Jeong, S., Seo, J., Hwang, T., Park, J. C. — *Typos that Broke the RAG's Back: Genetic Attack on RAG Pipeline by Simulating Documents in the Wild via Low-level Perturbations* (GARAG), Findings of EMNLP. https://arxiv.org/abs/2404.13948 ✅
20. Shao, Y., Jiang, Y., Kanell, T. A., Xu, P., Khattab, O., Lam, M. S. — *Assisting in Writing Wikipedia-like Articles From Scratch with Large Language Models* (STORM), NAACL 2024. https://arxiv.org/abs/2402.14207 ✅

Vendor and product sources (⚠ non-peer-reviewed, vendor-published):

21. Anthropic — *How we built our multi-agent research system*, engineering blog, 13 June 2025. https://www.anthropic.com/engineering/built-multi-agent-research-system ✅ (fetched; ⚠ vendor)
22. Google — *Try Deep Research and our new experimental model in Gemini*, The Keyword, 11 December 2024. https://blog.google/products/gemini/google-gemini-deep-research/ ✅ (fetched; ⚠ vendor)
23. OpenAI — *Introducing deep research*. https://openai.com/index/introducing-deep-research/ ⚠ (could not fetch; reported Feb 2025)
24. Perplexity — Deep Research. ⚠ (vendor offering; not fetched)

Companion and sibling guides (this repo):

25. [RAG evolution timeline](rag/rag_evolution_timeline_guide.md) · [Advanced RAG techniques](rag/advanced_rag_techniques_guide.md) · [RAG optimization](rag/rag_optimization_techniques_guide.md) · [Production-grade agentic RAG](rag/production_grade_agentic_rag_guide.md) · [Beyond RAG](rag/beyond_rag_guide.md) · [RAG vs HyDE](rag/rag_vs_hyde_guide.md) · [Query rewriting for RAG](rag/query_rewriting_rag_guide.md) · [RAG vs long-context LLMs](rag/rag_vs_long_context_llms_guide.md) · [RAG evaluation methodology](rag/rag_evaluation_methodology_guide.md) · [Vector databases](rag/vector_databases_guide.md)
26. [Agent scaffolding](agent_scaffolding_guide.md) · [Context engineering](context_engineering_guide.md) · [Agent harness engineering](agent_harness_engineering_guide.md) · [Production-ready LLM agents](production_ready_llm_agents_guide.md) · [LLM agent failure modes](llm_agents_failures_production_guide.md) · [Autonomous agents](autonomous_agents_guide.md) · [Hybrid multi-agent systems](hybrid_multi_agent_systems_guide.md) · [Hierarchical multi-agent frameworks](hierarchical_multi_agent_frameworks_guide.md) · [Multi-agent banking](multi_agent_banking_guide.md) · [MCP discovery](mcp_discovery_guide.md) · [AI governance framework](ai_governance_framework_guide.md) · [LLM evaluation frameworks](llm_evaluation_frameworks_guide.md)
27. [Research agents](../research_agents_guide.md) · [Agentic workflows](../agentic_workflows_guide.md) · [Low-latency GenAI patterns](../low_latency_genai_patterns_guide.md) · [AI for IT](ai_for_it_guide.md) · [GenAI banking compliance](../../banking/ai_genai_banking_compliance_guide.md)

---

The architecture question is never which technique is more advanced, but which one is *sufficient* for the query class in front of it — pay for the loop only where it buys accuracy, and never mistake a pipeline's cheap flat cost for a limitation, nor a loop's expensive reasoning for a guarantee. Choose the cheapest architecture that still returns a grounded, attributable, reproducible answer — **the retrieved answer.**
