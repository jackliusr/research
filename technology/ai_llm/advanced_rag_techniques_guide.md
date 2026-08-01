# Advanced RAG Techniques — From Research Papers to Production

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Crédit Agricole CIB
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** LLM/AI Engineering Guides
> **Companion Guides:** [RAG Optimization Techniques](rag_optimization_techniques_guide.md) · [RAG Frameworks Comparison](rag_frameworks_comparison_guide.md) · [RAG vs HyDE](rag_vs_hyde_guide.md) · [Query Rewriting in RAG](query_rewriting_rag_guide.md) · [Constrained Decoding Frameworks](../constrained_decoding_frameworks_guide.md) · [LLM Development Risks & Security](../llm_development_risks_security_guide.md)
> **Last Updated:** July 2026

---

## Table of Contents

1. [Introduction: From Naive to Agentic RAG](#1-introduction-from-naive-to-agentic-rag)
2. [The RAG Technique Evolution](#2-the-rag-technique-evolution)
3. [The RAG Taxonomy — Five Families of Techniques](#3-the-rag-taxonomy--five-families-of-techniques)
4. [Pre-Retrieval Advanced Techniques](#4-pre-retrieval-advanced-techniques)
5. [Retrieval-Side Advanced Techniques](#5-retrieval-side-advanced-techniques)
6. [Post-Retrieval Techniques](#6-post-retrieval-techniques)
7. [Generation-Side Techniques](#7-generation-side-techniques)
8. [End-to-End / Agentic Techniques](#8-end-to-end--agentic-techniques)
9. [Technique Comparison Matrix](#9-technique-comparison-matrix)
10. [Combining Techniques — Architectures That Work](#10-combining-techniques--architectures-that-work)
11. [Implementation Guidance](#11-implementation-guidance)
12. [Banking-Specific Guidance](#12-banking-specific-guidance)
13. [The Technique Selection Decision Tree](#13-the-technique-selection-decision-tree)
14. [Conclusion](#14-conclusion)

---

## 1. Introduction: From Naive to Agentic RAG

Naive RAG — *embed the corpus, embed the query, retrieve the top-k, stuff them into the prompt, generate* — is the "Hello World" of grounded LLM applications. It works well enough for demos, and it fails in production in predictable ways: vocabulary mismatch between queries and documents, retrieval that returns the *related* chunk instead of the *right* chunk, context that overwhelms the model, answers that drift from the evidence, and zero ability to handle questions that require multiple hops, aggregation across a corpus, or tools beyond a vector index.

This guide is a **research-to-practice map of the techniques that fix those failures**. It is the advanced companion in the RAG guide series: the [RAG Optimization Techniques guide](rag_optimization_techniques_guide.md) is the practical playbook for the baseline stack (chunking, hybrid retrieval, reranking, evaluation); [RAG vs HyDE](rag_vs_hyde_guide.md) and [Query Rewriting in RAG](query_rewriting_rag_guide.md) deep-dive two pre-retrieval levers; and [RAG Frameworks Comparison](rag_frameworks_comparison_guide.md) covers the orchestration tooling. **This guide covers the techniques those guides only mention in passing** — GraphRAG, RAPTOR, Self-RAG, CRAG, adaptive RAG, agentic RAG, RAG-Fusion, ColBERT late interaction, contextual retrieval, iterative retrieval, and the rest of the post-naive RAG landscape.

### 1.1 How this guide is organized

Every technique is treated consistently, so you can compare them on equal footing:

- **What it is** — the one-paragraph essence.
- **Provenance** — the paper, authors, year, and venue, so you can read the primary source.
- **Mechanism** — how it actually works under the hood.
- **Trade-offs** — quality, latency, cost, and complexity impact, stated as explicitly as the literature allows.
- **When to use** — the query patterns and corpus characteristics it targets.
- **Implementation notes** — libraries, code sketches, and pitfalls.

The final third of the guide is decision-oriented: a comparison matrix (§9), sensible ways to *combine* techniques instead of stacking everything (§10), a framework support matrix with code sketches (§11), banking-specific and regulatory guidance (§12), and a decision tree (§13).

### 1.2 The one-paragraph summary (for busy architects)

If you read nothing else: **start from a measured baseline** (see the [optimization guide](rag_optimization_techniques_guide.md)), and add advanced techniques **one at a time, only where the baseline measurably fails**. The single highest-leverage advanced upgrades, in rough order of popularity in production:

1. **Query rewriting / HyDE** — fixes vocabulary mismatch cheaply, at query time.
2. **Hybrid retrieval + reranking** — the production baseline itself (BM25 + dense + cross-encoder rerank).
3. **Contextual retrieval** — a one-time indexing cost that improves every downstream retrieval.
4. **GraphRAG or RAPTOR** — when questions are relational or corpus-level, not just passage-level.
5. **Adaptive routing / self-RAG / CRAG** — when the query mix is heterogeneous or the corpus is noisy.
6. **Agentic RAG** — when queries are genuinely multi-step and multi-source.

Everything else in this guide is a specialization of one of these six.

---

## 2. The RAG Technique Evolution

The advanced-technique landscape is best understood as four generations of architecture, each layering more *control* and *adaptivity* onto the last. The canonical framing comes from Gao et al.'s 2023 survey *Retrieval-Augmented Generation for Large Language Models: A Survey* (arXiv:2312.10997), which divides the field into **Naive RAG**, **Advanced RAG**, and **Modular RAG**; the fourth generation, **Agentic RAG**, emerged from the 2023–2024 wave of LLM agent frameworks.

### 2.1 Naive RAG — embed → retrieve → generate

The original architecture from Lewis et al. 2020 (*Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*, NeurIPS 2020): a retriever (BM25 in the original paper; dense retrievers later) fetches passages, and a generator conditions on them. The pipeline is linear and *static*: the same indexing and the same retrieval happen regardless of query type, corpus noise, or answer requirements.

| Property | Naive RAG |
|---|---|
| Pipeline | `index → embed query → retrieve top-k → concatenate → generate` |
| Adaptivity | None — every query gets the same treatment |
| Strengths | Simple, cheap, understandable, easy to debug |
| Failure modes | Low precision (wrong chunks retrieved), low recall (right chunk missed), no handling of multi-hop or aggregate questions, no self-correction, stale index |

Naive RAG is not "wrong" — it is the *baseline every other technique is measured against*. The optimization guide's [failure taxonomy](rag_optimization_techniques_guide.md) is essentially a catalog of naive RAG's failure modes.

### 2.2 Advanced RAG — optimize before and after retrieval

Advanced RAG keeps the linear pipeline but adds optimization at both ends (again per Gao et al. 2023):

- **Pre-retrieval optimization:** chunk size/strategy, metadata, query rewriting, query expansion — so the *query* and the *index* are better matched before a single search runs.
- **Post-retrieval optimization:** reranking (cross-encoders, ColBERT), context compression/filtering, reordering — so the *prompt* receives fewer, better, better-ordered chunks.

Advanced RAG is where most production systems live. It is the "production baseline" in this guide's combination playbook (§10).

### 2.3 Modular RAG — composable, swappable modules

Modular RAG (Gao et al. 2023) decomposes the pipeline into **modules** — indexing, pre-retrieval, retrieval, post-retrieval, generation, and *orchestration* — and defines **patterns** for composing them:

| Pattern | What it composes | Example techniques |
|---|---|---|
| **Rewriting–Retrieval** | A rewrite module transforms the query before retrieval | Multi-query, step-back, HyDE |
| **Routing** | A router picks the retriever/index/engine per query | Semantic routing, adaptive RAG |
| **Fusion** | Multiple retrievers/queries merged into one result set | Hybrid BM25+dense, RAG-Fusion, RRF |
| **Memory** | Conversation/episodic state feeds retrieval and generation | Chat history as query context, memory-augmented RAG |
| **Prediction** | The model generates candidate content to *drive* retrieval | HyDE, FLARE, GenRAG, ITER-RETGEN |
| **Search/Iteration** | Multiple retrieve-generate rounds | Multi-hop retrieval, iterative retrieval |

The key property of modular RAG: **each module can be upgraded, replaced, or skipped independently**. This is what makes A/B testing techniques feasible — and it is the architectural precondition for everything in §4–§8.

### 2.4 Agentic RAG — the LLM orchestrates retrieval itself

Agentic RAG hands the *control flow* to the LLM. Instead of a fixed pipeline, an agent decides: which query to run, against which retriever/tool, whether to retrieve at all, whether to retrieve again, and when to stop and answer. This is enabled by function calling / tool use (OpenAI function calling, Anthropic tool use), and by agent frameworks (LangGraph, LlamaIndex agents, AutoGen, CrewAI).

The evolution is really a story of **where the intelligence lives**:

| Generation | Where decisions are made | Control flow |
|---|---|---|
| Naive RAG | Nowhere (fixed pipeline) | Linear, one-shot |
| Advanced RAG | In hand-tuned pre/post steps | Linear, one-shot, optimized |
| Modular RAG | In the module composition | Configurable, still mostly linear |
| Agentic RAG | In the LLM at runtime | Looped, conditional, tool-using |

> **Read next:** §8 covers agentic RAG in depth; [LLM Development Risks & Security](../llm_development_risks_security_guide.md) covers the guardrails agentic systems require (excessive agency, tool abuse).

---

## 3. The RAG Taxonomy — Five Families of Techniques

Every technique in this guide belongs to one of five families, defined by *where in the pipeline it acts*. Use this taxonomy to navigate: when a system misbehaves, locate the failure in the pipeline first (retrieval failure vs. generation failure vs. orchestration failure), then pick techniques from the matching family.

| Family | Stage | Question it answers | Techniques in this guide |
|---|---|---|---|
| **Pre-retrieval** | Before search | "Is the query the best possible expression of the information need?" | Query rewriting, HyDE, query routing, query expansion (RAG-Fusion), query decomposition, step-back prompting, metadata enrichment (§4) |
| **Retrieval** | The search itself | "Is the index and the matching function finding the right evidence?" | Hybrid+RRF, multi-stage retrieval, ColBERT late interaction, contextual retrieval, sentence-window / parent-document, GraphRAG, RAPTOR, multi-hop retrieval (§5) |
| **Post-retrieval** | After search, before generation | "Is the context the model sees the minimal, most relevant, best-ordered evidence?" | Reranking, LLMLingua compression, contextual compression, lost-in-the-middle reordering, summarization, citation enforcement, dedup/filtering (§6) |
| **Generation** | The answer itself | "Is the answer grounded, verified, and structured?" | Grounded prompting, self-consistency, CoVe, attribution, constrained decoding, prompt caching, answer-aware retrieval, ensembling (§7) |
| **End-to-end / agentic** | Across the whole loop | "Who decides what to do next?" | Self-RAG, CRAG, adaptive RAG, FLARE, ITER-RETGEN, RARR, agentic RAG patterns (§5.7–§5.10 for the adaptive retrievers, §8 for agents) |

Two notes on placement. First, the boundaries are fuzzy by design: Self-RAG and CRAG are listed under retrieval-side techniques because their core mechanism is *evaluating and correcting retrieval*, but they span into generation (self-critique) — treat them as **pipeline-level techniques**. Second, **do not confuse the taxonomy with a maturity ladder**: pre-retrieval techniques are not "less advanced" than agentic ones. The right family is the one that matches the *failure you measured*.

## 4. Pre-Retrieval Advanced Techniques

Pre-retrieval techniques change the *query* or the *index* before a single search runs. They are the cheapest advanced techniques in the guide: almost all of them are query-time LLM calls (a few hundred tokens) with zero indexing changes, which makes them ideal first experiments. The baseline material on chunking and query transformations lives in the [optimization guide](rag_optimization_techniques_guide.md) and the [query rewriting guide](query_rewriting_rag_guide.md) — this section covers the *advanced* pre-retrieval techniques and references those guides rather than duplicating them.

### 4.1 Query rewriting — multi-query, step-back, decomposition

**What it is:** transforming the user's raw query into one or more better queries before retrieval.

**Provenance:** query rewriting predates LLMs (pseudo-relevance feedback, Rocchio), but the LLM-driven variants are catalogued in the [Query Rewriting in RAG guide](query_rewriting_rag_guide.md): multi-query (generate N paraphrases, retrieve for each, union the results), step-back prompting (Zheng et al. 2023, *Take a Step Back*, arXiv:2310.06117), and query decomposition (split into sub-questions, retrieve per sub-question).

**Mechanism:** a small LLM call rewrites `query → {q₁…qₙ}` or `query → abstraction → q_abstract`; each variant is embedded and searched; results are merged (union, RRF, or per-sub-question synthesis). No index change.

**Trade-offs:** low latency/cost (one small LLM call); large quality gains when the query is ambiguous, terse, or domain-jargon-heavy; risk of *over-rewriting* a query that was already good, and multiplication of retrieval calls (N variants = N searches).

**When to use:** vocabulary mismatch symptoms — the right documents exist in the index but rank poorly for the literal query. See the full treatment and code in [query_rewriting_rag_guide.md](query_rewriting_rag_guide.md).

### 4.2 HyDE — hypothetical document embeddings

**What it is:** generate a *fake document* that answers the query, embed the fake document instead of the query, and retrieve with it.

**Provenance:** Gao et al. 2022, *Precise Zero-Shot Dense Retrieval without Relevance Labels* (arXiv:2212.10496). Deep-dived in [RAG vs HyDE](rag_vs_hyde_guide.md) — reference it rather than re-reading here.

**Mechanism:** LLM writes a hypothetical passage (in the style of the target corpus); the passage embedding sits closer to *document* embeddings than the query embedding does, so dense retrieval finds relevant chunks even when query and document vocabularies don't overlap.

**Trade-offs:** one extra LLM call per query; works best on factual corpora where the model can plausibly draft the answer; can *mislead* retrieval when the hypothetical document is hallucinated or stylistically alien to the corpus; adds latency (~0.5–2s). The [RAG vs HyDE guide](rag_vs_hyde_guide.md) has the head-to-head and the "when not to" list.

**When to use:** short, underspecified queries against corpora with heavy domain vocabulary (contract clauses, regulatory text, internal procedures). Not recommended for questions whose answer depends on up-to-the-minute data the model doesn't know.

### 4.3 Query routing — send each query to the right engine

**What it is:** classify the query and dispatch it to the best retriever, index, or engine — vector search, SQL, an API, a web search, a knowledge graph, or *no retrieval at all* (direct LLM answer).

**Provenance:** routing is an engineering pattern formalized by agent frameworks (LlamaIndex `RouterQueryEngine`, LangChain `RouteQuery`) and popularized as a standalone library by Semantic Router (Aurelio AI, 2023–2024).

**Mechanism:** three common classifiers, in increasing order of sophistication:

| Router type | Mechanism | Latency | Cost | Accuracy ceiling |
|---|---|---|---|---|
| **Rules** | Keyword/regex patterns map to routes | ~0ms | 0 | Low but predictable |
| **Embedding similarity** | Query embedding vs. route exemplar embeddings; nearest route wins | ~5ms | ~0 | Good for *topically distinct* routes |
| **LLM classifier** | Prompt the model to return a route label (optionally with function calling) | +100–500ms | +1 small call | Highest; can handle nuance ("is this a factual query or an opinion?") |

**Trade-offs:** the router is a new failure point — a misrouted query is *worse* than no routing, because it goes to the wrong engine with confidence; the accuracy ceiling of the router caps the whole system; adds one decision hop and its latency.

**When to use:** heterogeneous data sources (a corpus split across a vector index, a data warehouse, and live APIs), or heterogeneous query types (chitchat vs. factual lookup vs. aggregation). This is the backbone of adaptive RAG (§5.9) and of tool-selecting agentic RAG (§8).

**Implementation note (LlamaIndex):**

```python
from llama_index.core.query_engine import RouterQueryEngine
from llama_index.core.selectors import LLMSingleSelector

router = RouterQueryEngine(
    selector=LLMSingleSelector.from_defaults(llm=llm),
    query_engine_tools=[vector_tool, sql_tool, web_tool],
)
response = router.query(user_query)
```

### 4.4 Query expansion and RAG-Fusion — more queries, fused results

**What it is:** generate *related terms and question variants*, retrieve for each, and fuse the result lists into one ranked list.

**Provenance:** classic query expansion (Robertson & Spärck Jones, 1970s–90s) predates LLMs; the modern LLM variant is **RAG-Fusion** (Rackauckas 2024, *RAG-Fusion: A New Take on Retrieval-Augmented Generation*, arXiv:2402.03367), built on **Reciprocal Rank Fusion (RRF)** (Cormack et al. 2009).

**Mechanism:**

1. LLM generates 3–5 query variants (paraphrases, synonyms, related sub-questions).
2. Each variant retrieves its own top-k list.
3. Lists are merged with RRF: each document scores `Σ 1/(k + rank)` across lists (k ≈ 60). Documents appearing in *several* lists at *high* ranks win — this is what makes fusion more robust than union-then-dedupe.

**Trade-offs:** increases **recall** substantially (catches chunks any single phrasing would miss); costs N× retrieval calls and one extra LLM call per query; fusion adds a bit of latency but is deterministic and cheap; slight risk of surfacing *more* irrelevant chunks (mitigated by post-retrieval reranking — §6.1).

**When to use:** queries with many valid phrasings; user queries that are short and underspecified; any system where recall failure (the right chunk simply not retrieved) is the dominant measured error.

**Implementation note (LlamaIndex `QueryFusionRetriever`, which wraps RRF):**

```python
from llama_index.core.retrievers import QueryFusionRetriever
from llama_index.core.query_engine import RetrieverQueryEngine

fusion_retriever = QueryFusionRetriever(
    [vector_retriever, bm25_retriever],
    similarity_top_k=10,
    num_queries=4,               # generate 4 query variants
    mode="reciprocal_rerank",    # RRF fusion
)
engine = RetrieverQueryEngine.from_args(fusion_retriever)
```

### 4.5 Query decomposition and the sub-question engine

**What it is:** break a complex query into sub-questions, retrieve per sub-question, then synthesize an answer from the combined evidence.

**Provenance:** multi-hop QA research (HotpotQA, Yang et al. 2018) and LlamaIndex's **SubQuestionQueryEngine** pattern; the LLM-driven decomposition variant is catalogued in [query_rewriting_rag_guide.md](query_rewriting_rag_guide.md).

**Mechanism:** LLM splits `query → [q₁, q₂, q₃]`, each sub-question is retrieved and answered (possibly against *different* indexes/tools), then a final LLM pass synthesizes the answer. This is the pre-retrieval cousin of multi-hop retrieval (§5.11): decomposition happens *up front* rather than iteratively.

**Trade-offs:** dramatically better on multi-part questions ("Compare the liquidity coverage ratios of Bank A and Bank B, and list which of their subsidiaries hold the most HQLA"); costs N retrievals + N generation passes (latency and cost scale with the number of sub-questions); sub-questions can be badly formed, and the final synthesis can lose the thread.

**When to use:** questions that contain multiple distinct information needs, comparisons, or "and then..." chains. See the code sketch in the [query rewriting guide](query_rewriting_rag_guide.md) (§3, Query Decomposition).

### 4.6 Step-back prompting — retrieve on the abstraction, not just the question

**What it is:** before retrieving, ask the LLM to abstract the question ("what is the general principle behind this?"), then retrieve using *both* the abstraction and the original question.

**Provenance:** Zheng et al. 2023, *Take a Step Back: Evoking Reasoning via Abstraction in Large Language Models* (arXiv:2310.06117).

**Mechanism:** two retrieval queries per user query — the abstraction (e.g., "What are the general principles of collateral management?") pulls in background/principles documents; the original query pulls in specific evidence. Both are concatenated into context. The step-back prompt is a fixed template, so this is cheap to implement.

**Trade-offs:** +1 retrieval call and slightly larger context; the abstraction can retrieve *too* general documents that crowd out specifics (keep top-k small for the abstraction leg, or rerank); excellent for questions that need a principle plus its application.

**When to use:** "why"-questions, questions about policies/principles, and domain questions where the answer requires a framework the user didn't name. Implemented in LlamaIndex via `StepBackQueryRewrite` or by hand with two retrievers.

### 4.7 Chunk optimization — reference the playbook

Chunk size, chunking strategy (fixed vs. semantic vs. recursive), overlap, and document structure awareness are the highest-leverage *indexing* optimizations — and they are fully covered in the [optimization guide](rag_optimization_techniques_guide.md) (§5, Indexing Optimization). The advanced techniques in this guide assume you already have a sane chunking strategy; **do not add GraphRAG or ColBERT to rescue a broken chunker.** The one advanced chunking-adjacent idea worth flagging here is that chunking and retrieval technique must be chosen *together*: sentence-window and parent-document retrieval (§5.4) implicitly change what a "chunk" is, and GraphRAG (§5.5) replaces chunks with graph entities as the retrieval unit.

### 4.8 Metadata enrichment — structure the index for filtering

**What it is:** at index time, extract structured attributes (entities, dates, document type, jurisdiction, counterparty, regulatory reference) and attach them to chunks; at query time, use them as **filters** or as **structured keys** for retrieval.

**Provenance:** engineering practice (LlamaIndex metadata extractors, LangChain metadata, vector DB metadata filtering); the "metadata filtering beats better embeddings" result is a recurring finding in RAG evaluation blogs and the optimization guide's indexing section.

**Mechanism:** an LLM (or a spaCy/GLiNER-style extractor) tags each chunk with entities/dates/summaries at index time; retrieval runs *pre-filter* (e.g., `jurisdiction == "SG" AND effective_date <= 2026-07-01`) before or after vector search, or uses metadata for hybrid retrieval weighting.

**Trade-offs:** one-time indexing cost; filter mistakes (wrong jurisdiction tag) cause *silent* misses — validate extraction on a sample; powerful when queries consistently carry a filterable dimension (date ranges, entity names, document classes).

**When to use:** corpora with strong latent structure — regulatory texts (filter by regulation, chapter, version), legal (by contract, clause type), finance (by counterparty, product, vintage). Combined with GraphRAG entities (§5.5) for entity-anchored retrieval.

---

## 5. Retrieval-Side Advanced Techniques

This is the largest family in the guide, and it splits into two groups: **matching-function upgrades** (hybrid, multi-stage, ColBERT, contextual retrieval, sentence-window/parent-document) and **structural upgrades** (GraphRAG, RAPTOR — techniques that change *what* is being searched), plus the **adaptive retrievers** (self-RAG, CRAG, adaptive RAG, FLARE, ITER-RETGEN, GenRAG, multi-hop, RARR, FLIP, REPLUG) that decide *whether and how* to retrieve.

### 5.1 Dense + sparse hybrid with RRF — the production baseline

BM25 (sparse, lexical) and dense embeddings (semantic) fail on disjoint query types: BM25 misses paraphrases, dense misses exact identifiers and rare terms. Combining them with RRF (same fusion math as §4.4) is now the default production retriever. **This is fully covered in the [optimization guide](rag_optimization_techniques_guide.md) (§6, Retrieval Optimization) — reference it; do not re-read here.** All advanced techniques in this section are assumed to sit *on top of* a hybrid baseline.

### 5.2 Multi-stage retrieval — retrieve wide, rerank narrow

Retrieve a wide candidate set (top-50–100) with a cheap retriever, then narrow to top-5–10 with a more expensive, more accurate reranker. Also covered in the [optimization guide](rag_optimization_techniques_guide.md) (§6.4–6.5). The reason it matters for this guide: **several advanced techniques are just specializations of multi-stage retrieval** — ColBERT as the second stage (§5.3), GraphRAG local search as a graph-based second stage (§5.5), and CRAG's evaluator as a *corrective* second stage (§5.8).

### 5.3 Late interaction and ColBERT — token-level matching

**What it is:** a retrieval model that matches at the **token level** rather than the whole-passage level: each query token and each document token gets its own embedding, and the query–document score is the sum of per-token *max* similarities.

**Provenance:** Khattab & Zaharia 2020, *ColBERT: Efficient and Effective Passage Search via Contextualized Late Interaction over BERT* (SIGIR 2020); **ColBERTv2** (Santhanam et al. 2022, NAACL) adds residual compression and the **PLAID** engine (~10× faster indexing/search); the reference implementation library is **RAGatouille**.

**Mechanism:** query tokens × document tokens produce a similarity matrix; for each query token, take the max over document tokens (`MaxSim`), then sum. This preserves fine-grained lexical-semantic evidence (e.g., "collateral" matching "collateralized") that whole-vector bi-encoders blur. ColBERTv2 compresses token embeddings (residuals + clustering) so indexes stay manageable; PLAID accelerates MaxSim search.

```
Query:  [collateral] [management] [rules]
Doc:    [collateral] [ised] [loan] [obligations] [management] ...
Score = max(col·col, col·ised, ...)  +  max(mgmt·mgmt, ...)  +  max(rules·rules, ...)
```

**Trade-offs:** measurably more accurate than bi-encoder dense retrieval on passage ranking (and it *is* a retriever, not just a reranker — it can replace the dense stage entirely); more compute per query than a bi-encoder (token×token, mitigated by PLAID); larger index than a single embedding per chunk; the model is BERT-sized (a few hundred MB) and runs locally on CPU reasonably well via RAGatouille.

**Usage patterns:** (a) **reranker** — hybrid BM25+dense retrieves top-100, ColBERT reranks to top-10 (very common, big precision win); (b) **direct retriever** — ColBERT alone as the dense stage, especially when exact-ish terminology matters.

**When to use:** token-level precision matters — legal terms, product names, regulatory identifiers, mixed-language corpora; or as the highest-accuracy reranker when cross-encoder GPU latency is unacceptable.

**Implementation note (RAGatouille):**

```python
from ragatouille import RAGPretrainedModel

colbert = RAGPretrainedModel.from_pretrained("colbert-ir/colbertv2.0")
colbert.index(name="my_index", collection=chunks, max_document_length=256)

results = colbert.search(query, k=10)          # direct retriever
reranked = colbert.rerank(query=query, documents=top_100_texts)  # reranker
```

LlamaIndex and Haystack both wrap RAGatouille (`ColbertIndex` / `ColbertRetriever`); LangChain uses it via `RAGatouilleRetriever` (community package).

### 5.4 Contextual retrieval (Anthropic) — put each chunk in its document context

**What it is:** at **index time**, prepend a short LLM-generated context sentence to each chunk before embedding it, so chunks that are ambiguous in isolation (a table of numbers, a "see clause 4.2") become self-explanatory.

**Provenance:** Anthropic, September 2024, *Introducing Contextual Retrieval* (engineering blog). The technique is prompt-engineering + a one-time indexing pass; it complements (and is orthogonal to) the embedding model itself.

**Mechanism:** for each chunk, call an LLM with the chunk plus surrounding document context and a prompt like: *"Here is the document and a chunk. Write a 50–100 word context for the chunk that situates it in the document, for use in search."* Prepend the returned context to the chunk, then embed (and also index for BM25) the *context + chunk* pair.

**Trade-offs:**

- **Cost:** one LLM call **per chunk at index time** — one-time, amortized; Anthropic's published example cost ~$1.02 per million tokens of context generation (well under $1k for a mid-size corpus).
- **Quality:** in Anthropic's tests, contextual embeddings alone reduced top-20 retrieval failure from 5.7% → 3.3% (**~49% reduction**); adding contextual BM25 on top reached 1.5% failure (**~67% reduction** combined).
- **Latency:** zero query-time impact — the context is baked into the index.
- **Complexity:** trivial pipeline addition; the main risk is context-generation prompt quality and the extra index storage.

**When to use:** any corpus where chunks are ambiguous in isolation — tables, numbered lists, "as described above" language, chunked PDFs. It is arguably the best *quality-per-effort* advanced technique that exists, because it upgrades every downstream retriever with zero query-time cost.

**Implementation note:** no framework ships this as a one-liner; it's a loop over chunks at index time:

```python
for chunk in chunks:
    context = llm.complete(CONTEXT_PROMPT.format(doc=full_doc, chunk=chunk))
    indexed_chunk = context + "\n\n" + chunk   # embed + BM25-index this
```

### 5.5 Sentence-window retrieval — retrieve small, return wide

**What it is:** index **sentence-level** chunks for retrieval precision, but return the sentence *plus a window* of surrounding sentences as context.

**Provenance:** LlamaIndex `SentenceWindowNodeParser` (2023); the "small-to-big" family of retrieval strategies.

**Mechanism:** parse documents into sentences; embed each sentence (or a small window) as the retrieval unit; on retrieval, expand each hit back to a configurable window (±N sentences, default 3) before passing to the generator. The *retrieval unit* and the *context unit* are deliberately different sizes.

**Trade-offs:** sharper retrieval (a query matches one precise sentence, not a 500-token chunk); window expansion restores context the generator needs; cheap to implement; slightly more index entries; window size is a tunable that affects precision vs. context.

**When to use:** dense reference documents (regulations, contracts) where the answer is a single clause buried in a long paragraph. Its sibling, **parent-document retrieval** (retrieve small chunks, return the full parent document), is covered in the [optimization guide](rag_optimization_techniques_guide.md) — choose sentence-window when a paragraph is enough context, parent-document when the whole section is needed.

### 5.6 GraphRAG — knowledge graphs for relational and global questions

**What it is:** build a **knowledge graph** from the corpus at index time — entities and relationships extracted by an LLM — then answer queries by traversing the graph and summarizing *communities* of related entities, rather than by similarity search over chunks.

**Provenance:** Microsoft Research, 2024 — Edge et al., *From Local to Global: A Graph RAG Approach to Query-Focused Summarization* (arXiv:2404.16130). Community detection uses the **Leiden algorithm** (Traag et al., 2019). Reference implementations: Microsoft GraphRAG (Python, `graphrag`), **LightRAG** (Guo et al. 2024, arXiv:2410.05779), **LazyGraphRAG** (Microsoft, 2025), and the community `nano-graphrag`.

**Mechanism (index time):**

1. **Entity/relationship extraction** — LLM reads the corpus (chunk by chunk) and extracts entities (people, orgs, instruments, clauses) and typed relationships, with source citations.
2. **Graph construction** — entities become nodes, relationships become edges; text units stay attached as evidence.
3. **Community detection** — Leiden finds hierarchical communities of densely-connected entities.
4. **Community summarization** — for each community (recursively, at multiple hierarchy levels), an LLM map-reduce pass writes a summary ("what is this community about?").

**Mechanism (query time)** — two search modes:

| Mode | How it works | Best for |
|---|---|---|
| **Global search** | Map-reduce over *all* community summaries (each summary is scored against the query, top ones feed a final synthesis) | Corpus-level questions: *"What are the main themes of this corpus?"*, *"Summarize the risk factors discussed."* |
| **Local search** | Anchor on query-matched entities (via vector similarity of entity descriptions + graph traversal of neighbors), then assemble entity, relationship, and community-summary context | Specific, entity-anchored questions: *"What entities are connected to X and how?"* |

**Strengths:**

- **Relational reasoning** — multi-hop entity questions ("which counterparties are connected to Company X through guarantees?") that vector search cannot answer because the evidence spans documents.
- **Global sensemaking** — answers *about the corpus itself* (themes, overviews, aggregate patterns), where similarity search over chunks fails because no single chunk contains the answer.
- Handles abstract questions that no passage-level technique can ground.

**Weaknesses:**

- **Expensive indexing** — LLM extraction over the *entire corpus* (token cost roughly proportional to corpus size × extraction prompt size; typically the most expensive indexing step of any technique here).
- **Complex pipeline** — extraction quality gates everything; graph construction, dedup of entities, community detection, and summary generation are all failure points; tuning is a project.
- **Slower queries** — global search in particular is a multi-LLM-call map-reduce (seconds, not the ~100ms of vector search).
- Entity-extraction LLM cost and latency make it unsuitable for fast-changing corpora (mitigated by LightRAG's incremental updates).

**vs. vector RAG:** vector RAG answers *"which passages are similar to this query?"*; GraphRAG answers *"how are these entities related, and what does the corpus say as a whole?"*. They are complements, not competitors — **GraphRAG local search itself is hybrid**: it uses vector similarity to find seed entities, then graph traversal to expand. A common architecture runs vector RAG for passage questions and GraphRAG for relational/global questions behind a router (§4.3).

**LightRAG** (2024) is the budget variant: a lighter extraction schema, **dual-level retrieval** (low-level entity links + high-level topic summaries, both used at query time), incremental graph updates, and significantly lower indexing cost — the usual choice when GraphRAG's indexing bill is unjustifiable.

**When to use:** entity-heavy domains (research, legal, finance — see §12), global/aggregate questions, and multi-hop entity questions. **When not to use:** small corpora, simple factual lookup workloads, corpora that change hourly, or teams that can't maintain an extraction-quality feedback loop.

**Implementation note (Microsoft GraphRAG quickstart):**

```bash
pip install graphrag
graphrag init --root ./ragproject        # creates settings + prompts
# edit settings.yaml (llm endpoint, model, chunk size)
graphrag index --root ./ragproject      # extract entities → graph → communities
graphrag query --root ./ragproject --method global  --query "What are the main themes?"
graphrag query --root ./ragproject --method local   --query "Who are the key entities linked to X?"
```

LlamaIndex ships a `GraphRAG` pack (and `PropertyGraphIndex` for knowledge-graph RAG); LangChain has community integrations for both GraphRAG and LightRAG.

### 5.7 RAPTOR — recursive abstraction trees

**What it is:** build a **hierarchical tree of summaries** over the corpus — cluster chunks, summarize each cluster, cluster the summaries, repeat — and retrieve at the abstraction level that matches the question.

**Provenance:** Sarthi et al. 2024, Stanford — *RAPTOR: Recursive Abstractive Processing for Tree-Organized Retrieval* (ICLR 2024).

**Mechanism (index time):**

1. Embed all leaf chunks.
2. Cluster them with a **Gaussian Mixture Model (GMM)** on the embeddings (using k-nearest-neighbor distances to pick the component count).
3. LLM-summarize each cluster into a *summary node* (the summary is embedded too).
4. Repeat on the summary embeddings — recursively — until one root summary remains. Result: a tree whose leaves are chunks and whose internal nodes are progressively more abstract summaries.

**Mechanism (query time):** two strategies — (a) **tree traversal**: descend from the root, choosing children most similar to the query, until you reach an appropriate level; (b) **collapsed tree**: retrieve over *all* nodes at once with a unified top-k, letting relevant abstracts and leaves compete directly. Both return a mix of summary-level and chunk-level context, so the generator gets "the big picture plus the details."

**Strengths:**

- **Multi-level abstraction** — answers "big picture" questions from summaries and detail questions from leaves, from one index.
- Handles long documents and corpus-level questions better than flat chunk retrieval.

**Weaknesses:**

- **Indexing cost** — a summarization LLM call per cluster per level (cheaper than GraphRAG's full extraction, but real).
- **Summary-quality dependency** — a bad summary poisons everything beneath it; the tree is only as good as the clustering *and* the summarizer.
- **Complexity** — clustering hyperparameters (GMM component selection, thresholds) need tuning; retrieval strategy (traversal vs. collapsed) is workload-dependent.

**vs. GraphRAG:** RAPTOR builds a hierarchy of *summaries* (abstraction over content); GraphRAG builds a graph of *entity relationships* (structure over content). RAPTOR answers "what is this corpus about, at every level of detail"; GraphRAG answers "how are these things connected". For a document set with strong entities but weak narrative (a contracts vault), GraphRAG wins; for long narrative documents with "big picture" questions (a research report corpus), RAPTOR wins.

**When to use:** long documents, "big picture"/overview questions, question sets spanning multiple granularities. **Implementation:** LlamaIndex `RaptorPack`; the `langchain-rag` package includes a RAPTOR implementation; a standalone `raptor` PyPI package exists.

### 5.8 Self-RAG — reflection tokens and retrieval on demand

**What it is:** an LLM trained (or prompted) to emit **reflection tokens** that make retrieval *adaptive*: it decides whether to retrieve, critiques whether retrieved passages are relevant/supported, and critiques whether its own generation is grounded and helpful.

**Provenance:** Asai et al. 2023 — *Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection* (arXiv:2310.11511, ICLR 2024).

**Mechanism (as designed):** the model is fine-tuned with reinforcement learning on a corpus of reflection-token annotations. At inference it can output tokens like:

| Reflection token | Meaning |
|---|---|
| `Retrieve` / `NoRetrieve` | Whether retrieval is needed for this step |
| `ISREL` | Is the retrieved passage relevant? |
| `ISSUP` | Is the passage supported by the source? |
| `ISUSE` | Is the passage useful for the answer? |
| `Groundness` | Is the final answer grounded in the passages? |
| `Helpfulness` | Is the answer useful? |

The model may retrieve at *multiple points* during generation, and the final answer is selected by a critic that weighs groundness + helpfulness.

**Mechanism (in practice without fine-tuning):** most production deployments **emulate** Self-RAG with prompting — a router decides retrieve/no-retrieve, a relevance check scores retrieved passages, and a final groundedness check gates the answer. This costs extra LLM calls but needs no fine-tuned weights.

**Strengths:** adaptive retrieval (fewer unnecessary retrievals → cost/latency savings on the queries that don't need grounding); self-critique catches unsupported answers; works on retrieval-optional query mixes (chitchat + factual).

**Weaknesses:** the full design requires a fine-tuned model you may not have; prompting emulation multiplies LLM calls; the critique steps themselves can be wrong (a "grounded" label on an ungrounded answer).

**When to use:** cost/latency-sensitive systems with a mixed query mix, and any system that needs an automatic "is this answer actually grounded?" check. Implementation: fine-tune for the full design; or prompt-emulate with a router + relevance judge + groundedness judge (see CRAG's evaluator, §5.9, for the judge pattern).

### 5.9 CRAG — Corrective RAG

**What it is:** evaluate the quality of retrieved documents *before* generating; if retrieval looks bad, **correct** it (web search or query decomposition + re-retrieval); if it looks good, **refine** the documents (strip irrelevant content, keep the relevant).

**Provenance:** Yan et al. 2024 — *Corrective Retrieval Augmented Generation* (arXiv:2401.15884).

**Mechanism:**

1. **Retrieval evaluator** — a lightweight scorer (a fine-tuned small LM or T5-based relevance model) scores each retrieved document against the query, producing a composite retrieval score.
2. **Route by score:**
   - **Good** (score above threshold): pass documents to a **knowledge refinement** step — decompose each doc into knowledge strips, filter out strips irrelevant to the query, keep the rest.
   - **Ambiguous/bad**: **correct** — either run a web search for supplementary evidence, or decompose the query and re-retrieve; the corrected set then also goes through refinement.
3. Generation proceeds on the refined/corrected evidence.

**Strengths:** **robustness to retrieval failures** — the system knows when it doesn't know; self-correcting behavior turns silent wrong answers into retrieved-corrected answers; knowledge refinement shrinks the prompt to the relevant bits.

**Weaknesses:** adds latency and cost (evaluator call + possibly a correction path); the evaluator's accuracy *is* the system — a bad evaluator routes good retrievals into expensive corrections; the web-search correction path is nondeterministic and needs guardrails (see [LLM Development Risks & Security](../llm_development_risks_security_guide.md)).

**When to use:** noisy or uncertain corpora (OCR garbage, mixed-quality sources, stale documents) and high-stakes correctness workloads where a retrieval miss must not silently become a confident wrong answer. Implementation: the canonical reference is the LangGraph **Corrective RAG** example (evaluator → route → web search/correct branch), easily adapted; LlamaIndex has a `CorrectiveRAG` pack.

### 5.10 Adaptive RAG — route by question complexity

**What it is:** classify each query by **complexity** and route to the appropriate strategy: **no retrieval** (LLM answers directly), **single-step retrieval**, or **multi-step retrieval** (iterative retrieval + reasoning).

**Provenance:** Jeong et al. 2024 — *Adaptive-RAG: Learning to Adapt Retrieval-Augmented Large Language Models through Question Complexity* (arXiv:2403.14403, NAACL 2024).

**Mechanism:** a **router** (trained classifier — small LM fine-tuned on complexity-labeled queries, or an LLM judge) labels the query: *"simple — answerable from parametric knowledge"*, *"moderate — one retrieval round"*, or *"complex — needs iterative retrieval"*. Each route uses a different (cheaper/faster vs. more expensive/thorough) pipeline.

**Strengths:** **cost/latency optimization** — simple queries skip retrieval entirely; the *right tool per query* means average cost drops without hurting hard-query quality; pairs naturally with any retrieval stack.

**Weaknesses:** router accuracy is the ceiling — misclassifying a simple query as complex wastes money, misclassifying a complex one as simple produces a hallucinated answer; needs labeled training data or careful prompt calibration; the three-way threshold is a hyperparameter.

**When to use:** mixed query difficulty (customer-support bots, internal helpdesk, analyst question portals) where many queries don't need retrieval at all. Implementation: LangGraph's **Adaptive RAG** reference pattern (classifier node → branch to direct-LLM / vector-RAG / multi-step); or LlamaIndex `RouterQueryEngine` with three engines.

### 5.11 FLARE — active retrieval during generation

**What it is:** generate sentence by sentence; **check confidence**; when the model is uncertain, retrieve and regenerate with fresh context — retrieval happens *during* generation, not before it.

**Provenance:** Jiang et al. 2023 — *Active Retrieval Augmented Generation* (FLARE, arXiv:2305.06983).

**Mechanism:** the model generates a sentence; if the *probability of its tokens* is low (uncertainty signal), FLARE prompts the model to produce a "clean" version of the sentence as a query, retrieves relevant documents, and regenerates the sentence conditioned on them. Repeat for every sentence. A deterministic variant retrieves on a fixed schedule (every N sentences).

**Strengths:** **retrieves only when needed** — targeted, on-topic evidence at the exact moment the generation needs it; ideal for long-form generation where information needs *evolve* (a report that first discusses X, then Y).

**Weaknesses:** requires **token-probability access** (not available on all hosted APIs); latency — retrieval mid-generation interrupts the decode loop; cost of repeated retrieval; confidence thresholds need tuning.

**When to use:** long-form generation with evolving information needs (reports, analyses, drafts) where a single upfront retrieval is stale by paragraph three. Implementation: LangChain `FLARE` (`langchain_experimental.flare`), or hand-rolled with any API that exposes logprobs.

### 5.12 ITER-RETGEN — iterate generation and retrieval

**What it is:** interleave generation and retrieval: generate an answer using retrieved documents, *use the generated text as the new query*, retrieve again, regenerate — repeat for 2–4 iterations.

**Provenance:** Shao et al. 2022/2023 — *Enhancing Retrieval-Augmented Large Language Models with Iterative Retrieval-Generation Synergy* (arXiv:2305.15294, EMNLP 2023 Findings).

**Mechanism:** round 1: retrieve with the user query, generate a draft. Round 2: use the draft as the query (it contains the model's current understanding and terminology), retrieve, regenerate. The *query comes from the generation*, so it captures information needs the user never expressed.

**Strengths:** strong gains on knowledge-intensive generation tasks (the paper reports consistent improvement over single-shot RAG on multiple benchmarks); naturally captures evolving needs like FLARE, but without token-probability requirements.

**Weaknesses:** **latency and cost** — each iteration is a full generate + retrieve round (2–4× the baseline); **drift risk** — a wrong early draft steers retrieval toward irrelevant content (the "garbage query" problem); needs an iteration cap and a stop condition.

**When to use:** complex reasoning + generation tasks where the first draft is expected to be incomplete (drafting, analysis, summarization with verification). Implementation: no first-class framework primitive — build it as a loop in LangGraph or LlamaIndex (`retrieve → generate → query=output → retrieve → generate`), or see the paper's reference code.

### 5.13 Self-Reflective RAG, GenRAG, multi-hop retrieval, RARR, FLIP, REPLUG

The remaining retrieval-side techniques are shorter, so they share this section.

**Self-Reflective RAG** (2023–2024, engineering synthesis of Self-RAG + CRAG): generate → *self-reflect* (is the answer supported by the retrieved evidence?) → if not, *correct*: retrieve more or reformulate the query → regenerate. Where Self-RAG learns the reflection and CRAG evaluates retrieval, Self-Reflective RAG applies reflection to the *answer* in a loop until it passes or a budget is exhausted. When to use: as the "self-check" layer of a correctness-critical pipeline (§10, layer 7). Implementation: a LangGraph loop with a judge node.

**GenRAG** (2024–2025, *GenRAG: Rethinking Retrieval for RAG*): **generate-then-retrieve** — generate a *full hypothetical answer* first (a longer cousin of HyDE), then retrieve documents that match the generated answer, and ground the final response in them. The rationale: the hypothetical answer encodes exactly what evidence is needed, so retrieval aligns with the answer's information needs better than the user's (often terse) query does. Weakness: a hallucinated hypothetical misleads retrieval — the same failure mode as HyDE, amplified by answer length. When to use: queries where the user's phrasing is a poor proxy for the answer content; always pair with a groundedness check.

**Multi-hop retrieval** (iterative, from multi-hop QA literature — HotpotQA, MuSiQue; tooling: LlamaIndex `RecursiveRetriever`, `KnowledgeGraphIndex`; LangChain): retrieve → extract entities/links from the hits → follow the links (neighbor nodes in a knowledge graph, or cross-references in text) → retrieve more. This is the *retrieval-side* answer to questions whose evidence spans documents ("Which of Company X's subsidiaries guarantees the loan to Y?"). vs. query decomposition (§4.5): decomposition plans the hops *up front*; multi-hop retrieval discovers them *as it goes*. When to use: entity-linking questions on corpora where documents reference each other; implement with LlamaIndex `RecursiveRetriever` over a property graph.

**RARR** (Gao et al. 2023, *Enabling Large Language Models to Generate Text with Citations*, EMNLP 2023): **attribution as a post-hoc loop** — generate the answer freely, then *search for evidence* supporting each claim, and *edit* the answer to add citations (or drop unsupported claims). Strengths: improves factual precision and gives verifiable citations; weaknesses: latency (a second full pipeline after generation) and cost; when to use: high-stakes factual answers where every claim must be attributable (§12). Related family: **attribution-aware generation** (§7.3) moves citation *into* generation rather than after it.

**FLIP** (2024, *FLIP: Preference Alignment for Factuality*): use **preference alignment (RL)** to train the model to *prefer grounded answers* over fluent-but-ungrounded ones — a generation-side fine-tuning that makes faithfulness a model property rather than a prompt instruction. Weaknesses: requires fine-tuning infrastructure and preference data; when to use: when you control the model and faithfulness failures persist despite prompting (see the [fine-tuning guide](fine_tuning_frameworks_comparison_guide.md) for tooling).

**REPLUG** (Shi et al. 2023, *REPLUG: Retrieval-Augmented Black-Box Language Models*, arXiv:2301.12652): the simplest possible retrieval-augmentation — *prepend* retrieved documents to the LM input ("LM as a black box"), no training needed; optionally train a *retriever* against the LM's own likelihoods (self-supervised LM-retriever training). Strengths: works with any LM, trivially simple, and the retriever-training variant anticipates 2024–2025's "LM-as-retriever" line of work; weaknesses: no adaptation of retrieval mid-generation, context-length pressure. When to use: as a fast baseline when you cannot modify the model or the pipeline; conceptually, nearly every RAG system today is "REPLUG with better retrieval."

## 6. Post-Retrieval Techniques

Post-retrieval techniques shape the *context window* — what the generator actually sees. The baseline material (reranking, contextual compression, lost-in-the-middle reordering) is covered in the [optimization guide](rag_optimization_techniques_guide.md) (§6–§7); this section covers the advanced extensions and adds what the optimization guide doesn't.

### 6.1 Reranking — the highest-leverage post-retrieval step

Cross-encoder reranking, ColBERT reranking, FlashRank, and the "retrieve 50–100, rerank to 5–10" pattern are the single most reliable quality win in RAG — **fully covered in the [optimization guide](rag_optimization_techniques_guide.md) (§6.4–6.5). Reference it.** Two advanced notes: (1) every advanced technique in this guide that returns a *candidate set* — RAG-Fusion, multi-hop retrieval, graph traversal — should end in a rerank step, because all of them widen recall at the cost of precision; (2) reranking is also your cheapest *diagnostic*: if reranking a fixed candidate set doesn't move answer quality, the failure is upstream (index/chunking), not in ranking.

### 6.2 Context compression — LLMLingua

**What it is:** compress the retrieved context (and prompt) with a **small language model** before sending it to the big LLM — removing tokens that contribute little while preserving the ones that matter, under a user-specified budget.

**Provenance:** Jiang et al. 2023 — *LLMLingua: Compressing Prompts for Accelerated Inference of Large Language Models* (EMNLP 2023); **LongLLMLingua** (Jiang et al. 2024) extends it to long contexts and explicitly targets the lost-in-the-middle problem by preserving question-relevant positions.

**Mechanism:** a small LM (e.g., a lightweight GPT-2-class model) scores every token's *perplexity contribution* in context; tokens that are predictable given their neighbors are pruned, subject to a budget (e.g., keep 20× smaller); the compressed prompt is sent to the big LLM. Compression can be *coarse-grained* (drop whole sentences/segments) and *fine-grained* (drop tokens within kept segments), and a **decompression** step re-expands the compressed output if needed.

**Trade-offs:**

- Up to **20× compression** with minor quality loss in the paper's evaluations (quality degradation grows as compression ratio rises past ~10–20×).
- **Cost/latency win** is roughly proportional to compression: fewer input tokens = cheaper, faster big-LLM calls; on long-context workloads the win is large.
- Costs: the compression pass itself adds latency (small LM, but still a forward pass); aggressive compression *can* strip nuance; the quality-vs-compression trade-off is workload-specific and must be measured (compressing regulatory text is riskier than compressing internal chatter).
- LongLLMLingua's position-aware variant also mitigates lost-in-the-middle by allocating budget toward the beginning/end of context.

**When to use:** long retrieved contexts, expensive flagship models, strict latency budgets, and workloads where most retrieved tokens are boilerplate. Implementation: `llmlingua` PyPI package; LangChain `LLMLinguaCompressor` / `LongContextReorder`; works with any RAG pipeline as a wrapper around the context-assembly step.

### 6.3 Contextual compression (LangChain) — keep only the relevant parts

LangChain's `ContextualCompressionRetriever` wraps any base retriever with a compressor that strips each retrieved document down to the sentences relevant to the query — extractive (an LLM extracts the relevant passages) or LLM-based (a `LLMChainExtractor`). Covered in the [optimization guide](rag_optimization_techniques_guide.md). The design point worth repeating: **compression is a per-query LLM call**, so it trades one generation-stage token bill for a smaller main-generation bill — measure before adopting on high-volume workloads.

### 6.4 Lost-in-the-middle mitigation — order matters

LLMs attend to the **beginning and end** of their context far better than the middle (Liu et al. 2023, *Lost in the Middle: How Language Models Use Long Contexts*). Mitigations — reorder retrieved chunks so the most relevant sit at the edges, use LongLLMLingua's position-aware compression, or use CoLT-style context compression — are covered in the [optimization guide](rag_optimization_techniques_guide.md). It belongs in this guide's layered architecture (§10) as a **mandatory context-assembly step**, because every advanced technique that widens retrieval (fusion, graph, multi-hop) makes context longer and the middle-loss problem worse.

### 6.5 Document summarization instead of raw chunks

**What it is:** when retrieved documents are long or heterogeneous, summarize each retrieved document and pass *summaries plus key details* instead of raw chunks.

**Mechanism:** after retrieval, run a map-reduce or refine summarization over each retrieved document (or the fused set), producing a distilled context; optionally keep verbatim quotes for the claims the query hinges on. This is the "compression by synthesis" complement to LLMLingua's "compression by pruning."

**Trade-offs:** summaries fit more *documents* into the context window (better coverage of fused/global results); summarization is an LLM call per document (cost/latency, mitigated by caching); summary drift can hide the exact number/clause the user needs — keep quotes for critical spans.

**When to use:** GraphRAG global search (which is literally this technique over community summaries), multi-document synthesis, and "what do these N documents collectively say" queries. Implementation: LangChain `load_summarize_chain` (map-reduce/refine), LlamaIndex `Refine`/`TreeSummarize` response modes, or the summarizer node in LangGraph.

### 6.6 Citation enforcement — make sources first-class

**What it is:** require the generator to emit **structured citations** — every claim tied to a retrieved document (id, page, clause) — and *enforce* that citations resolve to actually-retrieved sources.

**Mechanism:** prompt-level (instruct the model to cite `[n]` mapped to provided sources), schema-level (structured-output/constrained decoding — see [Constrained Decoding Frameworks](../constrained_decoding_frameworks_guide.md) — for citation spans), and verification-level (post-check that every cited source was in the retrieved set and that the claim is supported — the RARR loop, §5.13).

**Trade-offs:** citations add output tokens and can be *fabricated* unless verified (models will happily invent `[3]`); verified citations cost a post-generation check; but citations are the single cheapest way to convert "probably grounded" into "auditable" — which is why banking deployments treat them as mandatory (§12).

**When to use:** every production system, and non-negotiably in regulated industries; pair prompt-level citation with a verification pass for high-stakes claims.

### 6.7 Deduplication and relevance filtering

Retrieved sets from fusion, multi-hop, or graph pipelines contain **duplicates** (the same chunk via two query variants, the same entity via two hops) and **near-misses** (similar-but-wrong chunks). Filtering them before generation: (a) dedupe by content hash or embedding similarity (keep the highest-ranked copy); (b) relevance-filter with a similarity threshold or a lightweight judge; (c) **MMR** (Maximal Marginal Relevance, Carbonell & Goldstein 1998) — greedily select chunks that are *relevant* and *diverse*, trading redundancy for coverage. Covered in part in the [optimization guide](rag_optimization_techniques_guide.md); the advanced-system point is that **fusion and graph pipelines need dedup more than single-retriever pipelines do** — their whole mechanism is generating overlapping candidates.

---

## 7. Generation-Side Techniques

Generation-side techniques change *how the answer is produced and verified*. Grounded prompting is the baseline and is covered in the [optimization guide](rag_optimization_techniques_guide.md) (§7); what follows are the advanced generation techniques.

### 7.1 Self-consistency — sample, vote, answer

**What it is:** generate **N answers** (with sampling), take a **majority vote** (or weighted vote) over them.

**Provenance:** Wang et al. 2022 — *Self-Consistency Improves Chain of Thought Reasoning in Language Models* (ICLR 2023).

**Mechanism:** with a reasoning/CoT prompt, sample N decoding paths (temperature > 0), cluster the answers, pick the most frequent. For RAG, each sample may also use a *slightly different context ordering or retrieval top-k*, making the vote a crude ensemble over both generation and context.

**Trade-offs:** improves reasoning/consistency, especially on multi-step questions; costs **N× generation** (latency and tokens); majority voting is meaningless for open-ended questions with many valid answers — apply it to questions with a canonical answer.

**When to use:** high-stakes factual/quantitative questions ("What is the LCR requirement under MAS 644?"), where N=3–5 samples give most of the gain.

### 7.2 Chain-of-Verification (CoVe) — verify, then answer

**What it is:** a **draft → verify → refine** loop: draft the answer, generate *verification questions* whose answers would confirm each claim, answer them (with retrieval), then produce the final answer revised in light of the verification.

**Provenance:** Dhuliawala et al. 2023 — *Chain-of-Verification Reduces Hallucination in Large Language Models* (arXiv:2309.11495, ACL 2024 Findings).

**Mechanism:**

1. **Draft**: generate the answer.
2. **Plan verification**: generate a list of verification questions (one per factual claim).
3. **Execute verification**: answer each question (ideally *with retrieval* — the RAG-aware CoVe variant grounds the verification).
4. **Revise**: produce the final answer, correcting claims the verification contradicted.

**Trade-offs:** a large, measurable reduction in hallucination (the paper reports major drops on list/closed-book-style questions); costs 2–3 extra LLM passes (latency, tokens); verification questions must be well-formed — a vague draft yields vague verification. For RAG, the retrieval-grounded verification pass converts this from a reasoning trick into a *factuality control loop*.

**When to use:** correctness-critical answers (regulatory Q&A, due-diligence summaries) where the marginal cost of 2 extra passes is acceptable. Implementation: a four-node LangGraph flow, or the prompt-only version in ~40 lines.

### 7.3 RAG with attribution — cite sources per claim

**What it is:** generate with *per-claim source links* — the generation-side counterpart of §6.6, and the in-generation version of RARR (§5.13).

**Provenance:** attribution-aware generation was popularized by OpenAI's *WebGPT* (2021) and *GopherCite* (Menick et al. 2022), then standardized in RAG practice (e.g., LlamaIndex `CitationQueryEngine`, LangChain `create_citation_fitted_runnable`, Airtable-style citation blocks).

**Mechanism:** instruct the model to emit claims with inline citation markers that must reference the provided retrieved documents; verify post-hoc that each marker maps to a real source and (optionally) that the source supports the claim.

**Trade-offs:** same as §6.6 — token overhead, fabrication risk without verification, big auditability win.

**When to use:** all production RAG, mandatory in regulated environments (§12). Combine with RARR for high-stakes claims.

### 7.4 Constrained decoding for structured answers

When the answer must be a schema (JSON, a fixed enum, a data-extraction form), constrain the decoder to valid outputs — grammar-constrained sampling, JSON-schema decoding, or tool/function-call schemas — instead of hoping the model formats correctly. **Fully covered in [Constrained Decoding Frameworks](../constrained_decoding_frameworks_guide.md)** (outlines, jsonformer, guidance, LMQL, Outlines, vLLM structured outputs). For RAG specifically, constrained decoding is the mechanism behind *enforced* citations (§6.6), structured extraction of entities (GraphRAG pipelines), and tool calls in agentic RAG (§8).

### 7.5 Prompt caching — pay once for shared prefixes

**What it is:** cache the KV state of the *shared prefix* of prompts (system prompt + static instructions + frequently reused context) across queries, so repeated tokens cost a fraction (cached-input pricing) and prefill latency drops.

**Provenance:** engineering practice (Anthropic prompt caching, OpenAI automatic caching, vLLM prefix caching, Llama.cpp cache reuse); covered in [LLM Latency Optimization](llm_latency_optimization_guide.md).

**Mechanism:** order the prompt so that everything static comes first — system prompt, task instructions, grounding rules, citation format — followed by the query-specific context; cache the prefix. In agentic RAG this matters doubly: every agent loop iteration re-sends the same system prompt, so caching directly cuts loop cost.

**Trade-offs:** near-free to adopt (prompt ordering + a caching-enabled serving layer); savings grow with prefix length and request volume; cache invalidation hygiene needed when instructions change.

### 7.6 Answer-aware retrieval — retrieve against the draft

**What it is:** retrieve *again* using the draft answer (or its claims) as the query — the generation-side hook for ITER-RETGEN (§5.12), GenRAG (§5.13), and CoVe's verification pass (§7.2). Where §4 techniques rewrite the *user query*, answer-aware retrieval uses the *model's own output* as the retrieval query, because it expresses the information need the model still has.

**Trade-offs:** one more retrieve+generate round (latency/cost); quality gain concentrates on questions where the draft is partially right — a fully wrong draft steers retrieval wrong (the drift risk of all generation-as-query techniques).

### 7.7 Ensembling RAG pipelines — vote or merge

**What it is:** run **multiple RAG pipelines** (different retrievers, chunkings, or models) and merge their answers — majority vote for canonical questions, or a synthesizer that merges drafts.

**Trade-offs:** the most expensive generation technique here (N full pipelines per query); gains are real but saturating — two well-chosen pipelines (e.g., vector-RAG + GraphRAG) beat five similar ones; complexity of reconciling conflicting answers; useful mainly for correctness-critical, low-volume workloads, or as an offline *judge* that labels training/eval data.

**When to use:** high-stakes answers where you can afford 2–3× cost; never as a first optimization — it is the "throw money at it" technique, and the [optimization guide](rag_optimization_techniques_guide.md) 80/20 list deliberately leaves it out.

---

## 8. End-to-End / Agentic Techniques

Agentic RAG replaces the fixed pipeline with an **agent**: an LLM that holds the conversation state, chooses tools (retrievers, databases, APIs, web search), calls them, observes results, and iterates until it decides the answer is ready — or the budget is spent.

### 8.1 What agentic RAG adds (and what it costs)

| Dimension | Fixed-pipeline RAG | Agentic RAG |
|---|---|---|
| Who picks the query | The pipeline (one query, rewritten at most) | The agent (reformulates, decomposes, re-queries) |
| Who picks the retriever | Fixed (one index or a routed choice) | The agent (vector, SQL, graph, web, API — per step) |
| When to retrieve | Always, once | On demand, possibly multiple times |
| When to stop | After one generation | After the agent judges the answer sufficient (or budget runs out) |
| Failure handling | None (or CRAG-style fixed corrections) | Self-correcting: retry, re-query, change tool |
| Latency/cost | Predictable, bounded | Variable, can balloon (loop risk) |
| Explainability | Full (every step visible) | Partial (agent decisions are opaque) |

**Strengths:** handles complex, multi-step, heterogeneous queries that fixed pipelines cannot express — "research counterparty X: pull their filings from Edgar, their news from the news feed, their limits from the internal system, and summarize the risk picture with citations"; adapts per query; composes tools.

**Weaknesses:** latency (each reasoning step is an LLM call; complex queries = 5–20 calls), cost (same), **unpredictability** (agents loop, change strategy mid-query, or burn budget), and **safety surface** (tool abuse, excessive agency, prompt injection through retrieved content — see [LLM Development Risks & Security](../llm_development_risks_security_guide.md) §on agent guardrails).

**When to use:** complex queries, heterogeneous data sources, multi-step reasoning. **When not to use:** simple factual lookup at high volume — a fixed pipeline with adaptive routing (§5.10) is cheaper, faster, and auditable.

### 8.2 Patterns

**ReAct-style RAG** (Yao et al. 2022, *ReAct: Synergizing Reasoning and Acting in Language Models*): the canonical loop — **Reason** ("I need X; the vector index has it") → **Act** (call the retrieval tool) → **Observe** (tool result) → repeat → **Answer**. Implementations: LangGraph `create_react_agent` with retrieval tools; LlamaIndex `ReActAgent` with query-engine tools. Simple, robust, and the default agentic-RAG pattern.

**Plan-and-execute RAG** (Wang et al. 2023 plan-and-solve lineage; LangGraph `PlanAndExecute`): a planner LLM writes a *plan of retrieval steps* ("1. retrieve filings; 2. retrieve news; 3. synthesize"), an executor runs the steps, and a replanner fixes the plan when a step fails. More structured than ReAct — better for multi-source research where the plan is knowable in advance; costs an extra planner LLM call.

**Tool-selecting RAG (router agent):** the agent's first decision is *which tool* — vector search, SQL, a knowledge graph, a web search, an internal API — via function calling with tool descriptions (§4.3's router, promoted to an agent). This is how §12's banking research agent combines filings + news + internal docs.

**Memory-augmented RAG:** the agent keeps **conversation memory** (chat history, rewritten queries — see [Query Rewriting in RAG](query_rewriting_rag_guide.md) §9) and optionally **episodic memory** (summaries of past interactions, stored and retrieved like documents). Critical for multi-turn research ("now do the same analysis for the subsidiary").

**Multi-agent RAG:** split roles across agents — a **retriever agent**, a **critic agent** (checks groundedness/relevance), a **synthesizer agent** — coordinated by an orchestrator. See [Hybrid Multi-Agent Systems](hybrid_multi_agent_systems_guide.md) and [Hierarchical Multi-Agent Frameworks](hierarchical_multi_agent_frameworks_guide.md) for the full treatment. Multi-agent RAG is rarely needed before single-agent + strong post-retrieval checks fail; it is the most expensive and least deterministic option in this guide.

### 8.3 Frameworks and guardrails

- **LangGraph** — graph-based state machines; the reference implementations for ReAct, adaptive RAG, CRAG, and self-reflective RAG all ship as official examples; best control over loops and budgets.
- **LlamaIndex agents** — `ReActAgent`, `FunctionCallingAgent`, tool abstractions over its query engines; tightest integration with RAG primitives.
- **AutoGen** — multi-agent conversation (conversable agents); good for multi-agent RAG research; heavier orchestration.
- **CrewAI** — role-based crews (researcher/analyst/critic); quick to prototype; abstraction hides some control.
- **OpenAI Assistants / function calling, Anthropic tool use** — the raw substrate; fine for a single retrieval tool, insufficient control for loops — wrap with a framework.

Every agentic deployment needs guardrails: **max iterations and timeouts** (budget caps), **tool allow-lists and read-only tools** by default, **human-in-the-loop checkpoints** for high-stakes actions, and **prompt-injection defenses on retrieved content** (treat tool output as data, not instructions). See [LLM Development Risks & Security](../llm_development_risks_security_guide.md).

## 9. Technique Comparison Matrix

The full field at a glance. Impact ratings are directional, not precise: they describe the *typical* effect on top of a working hybrid-RAG baseline, and every cell should be re-measured on your own golden dataset (§11.3). Ratings: **▲▲▲** large, **▲▲** moderate, **▲** small, **—** neutral/negligible.

| Technique | What it solves | Mechanism | Quality | Latency | Cost | Complexity | Paper / Source | When to use |
|---|---|---|---|---|---|---|---|---|
| Query rewriting (multi-query, step-back, decomposition) | Vocabulary mismatch, terse queries | LLM transforms query into variants/abstractions before retrieval | ▲▲ | ▲ | ▲ | ▲ | [query_rewriting_rag_guide](query_rewriting_rag_guide.md); Zheng 2023 | Ambiguous queries; corpus jargon |
| HyDE | Query–doc semantic gap | Embed a hypothetical answer instead of the query | ▲▲ | ▲ | ▲ | ▲ | Gao 2022 (arXiv:2212.10496) | Short queries, domain vocab |
| Query routing / semantic router | One retriever for all queries | Classify query → dispatch to best engine (or no retrieval) | ▲▲ | ▲ | ▲ | ▲▲ | LlamaIndex/LangChain; Semantic Router | Heterogeneous sources/query types |
| Query expansion + RAG-Fusion | Single phrasing misses evidence | N variants retrieved, RRF-fused | ▲▲ | ▲▲ | ▲▲ | ▲ | Rackauckas 2024 (arXiv:2402.03367) | Recall-limited systems |
| Query decomposition | Multi-part/comparison queries | Split into sub-questions, synthesize | ▲▲ | ▲▲ | ▲▲ | ▲▲ | HotpotQA lineage; LlamaIndex SubQuestion | "A vs B", multi-part asks |
| Step-back prompting | Questions needing principles | Retrieve on abstraction + original query | ▲ | ▲ | ▲ | ▲ | Zheng 2023 (arXiv:2310.06117) | "Why"-questions, policy QA |
| Metadata enrichment | Chunks lack structure | Extract entities/dates/summaries for filtering | ▲▲ | — | ▲ (index-time) | ▲▲ | Engineering practice | Regulatory/legal/finance corpora |
| Hybrid dense+sparse + RRF | Disjoint recall failure modes | BM25 + dense fused by RRF | ▲▲ | ▲ | — | ▲ | Cormack 2009; [optimization guide](rag_optimization_techniques_guide.md) | **Production baseline** |
| Multi-stage retrieval (wide→rerank) | Precision after recall | Retrieve 50–100, rerank to 5–10 | ▲▲ | ▲ | ▲ | ▲ | [optimization guide](rag_optimization_techniques_guide.md) | **Production baseline** |
| ColBERT late interaction | Token-level matching precision | Token×token MaxSim scoring | ▲▲ | ▲▲ | ▲▲ | ▲▲ | Khattab & Zaharia 2020; ColBERTv2/PLAID | Legal/regulatory terminology |
| Contextual retrieval (Anthropic) | Ambiguous isolated chunks | LLM context prepended at index time | ▲▲▲ | — | ▲ (one-time) | ▲ | Anthropic 2024 | **Any corpus**; tables, PDFs |
| Sentence-window / parent-doc retrieval | Chunk granularity mismatch | Retrieve small, return wide context | ▲▲ | — | — | ▲ | LlamaIndex small-to-big | Dense reference docs |
| GraphRAG | Relational & global questions | KG extraction + Leiden communities + map-reduce search | ▲▲▲ (relational/global) | ▲▲▲ (global) | ▲▲▲ (index) | ▲▲▲ | Edge 2024 (arXiv:2404.16130) | Entity-heavy, corpus-level Qs |
| LightRAG | GraphRAG cost | Lighter schema, dual-level retrieval, incremental updates | ▲▲ | ▲▲ | ▲▲ | ▲▲ | Guo 2024 (arXiv:2410.05779) | GraphRAG on a budget |
| RAPTOR | Big-picture questions, long docs | Recursive cluster-summarize tree | ▲▲ | ▲▲ | ▲▲ (index) | ▲▲▲ | Sarthi 2024 (ICLR) | Overview questions |
| Self-RAG | Unnecessary retrieval, ungrounded answers | Reflection tokens decide retrieve/critique | ▲▲ | ▲▲ | ▲▲ | ▲▲▲ | Asai 2023 (arXiv:2310.11511) | Mixed query difficulty; groundedness gates |
| CRAG | Silent retrieval failures | Evaluate → correct (web/decompose) or refine | ▲▲ | ▲▲ | ▲▲ | ▲▲ | Yan 2024 (arXiv:2401.15884) | Noisy corpora, high-stakes |
| Adaptive RAG | Wrong tool per query | Complexity router → direct/single/multi-step | ▲▲ | ▲ | ▲ (saves) | ▲▲ | Jeong 2024 (arXiv:2403.14403) | Mixed query difficulty |
| FLARE | Evolving needs in long generation | Sentence-level confidence → retrieve on demand | ▲▲ | ▲▲ | ▲▲ | ▲▲ | Jiang 2023 (arXiv:2305.06983) | Long-form drafting |
| ITER-RETGEN | Draft incompleteness | Generate → use draft as query → regenerate | ▲▲ | ▲▲▲ | ▲▲▲ | ▲▲ | Shao 2022 (arXiv:2305.15294) | Complex generate tasks |
| Multi-hop retrieval | Cross-document evidence | Retrieve → follow entity links → retrieve | ▲▲ | ▲▲ | ▲▲ | ▲▲▲ | HotpotQA lineage; LlamaIndex RecursiveRetriever | Entity-linking questions |
| RARR | Ungrounded claims, no citations | Generate → search evidence → edit with citations | ▲▲ | ▲▲▲ | ▲▲▲ | ▲▲ | Gao 2023 (EMNLP) | High-stakes factual answers |
| LLMLingua compression | Oversized context, cost | Small-LM token pruning under budget | ▲ (quality) / ▲▲ (cost) | ▲ (saves) | ▲▲ (saves) | ▲▲ | Jiang 2023 (EMNLP) | Long contexts, costly models |
| Self-consistency | Reasoning variance | N samples, majority vote | ▲▲ | ▲▲▲ | ▲▲▲ | ▲ | Wang 2022 (ICLR 2023) | Canonical-answer questions |
| CoVe | Hallucination | Draft → verify questions → revise | ▲▲ | ▲▲▲ | ▲▲▲ | ▲▲ | Dhuliawala 2023 (arXiv:2309.11495) | Correctness-critical QA |
| Agentic RAG | Multi-step, multi-source queries | Agent loops over retrieval tools | ▲▲ | ▲▲▲ | ▲▲▲ | ▲▲▲ | Yao 2022 ReAct; LangGraph/LlamaIndex | Complex heterogeneous queries |

**Reading the matrix:** the bottom-left quadrant (quality ▲▲, cost ▲) is where the best ROI lives — contextual retrieval, hybrid+rerank, query rewriting, HyDE. The top-right quadrant (cost/latency ▲▲▲, complexity ▲▲▲) — agentic RAG, GraphRAG global, ITER-RETGEN — should be adopted only for workloads that measurably *need* them.

---

## 10. Combining Techniques — Architectures That Work

### 10.1 The "kitchen sink" caution

The naive way to "use advanced RAG" is to stack every technique in this guide into one pipeline. This fails in three compounding ways:

1. **Cost/latency** — each technique adds LLM calls, retrieval calls, or index time; ten techniques can multiply per-query cost by 10–30× while improving quality by a few points.
2. **Debugging hell** — with N interacting modules, a wrong answer has N possible causes; you can no longer attribute failures (the optimization guide's first discipline, §1.1, becomes impossible).
3. **Interaction effects** — techniques can *fight*: query rewriting plus HyDE plus RAG-Fusion each distort the query differently; GraphRAG output plus reranking can duplicate context; FLARE plus agentic loops multiply latency unpredictably.

**The discipline:** every technique must earn its place against the *measured* failure it fixes (§11.3). Add one at a time, measure before/after, keep what pays for itself.

### 10.2 Sensible combinations (recipes that are known to compose well)

**(a) Production baseline** — hybrid (BM25+dense+RRF) → multi-stage rerank → query rewriting → golden-dataset evaluation. This is the floor for everything below; see the [optimization guide](rag_optimization_techniques_guide.md) for the exact playbook.

**(b) Entity-heavy corpus + global questions** — add **GraphRAG** (or **LightRAG** on a budget), routed alongside vector RAG (§4.3): vector RAG for passage questions, GraphRAG local search for entity questions, GraphRAG global search for corpus-level questions.

**(c) Long documents + "big picture" questions** — add **RAPTOR**; its tree gives summary-level and leaf-level retrieval from one index. Pairs well with (a): use RAPTOR for overview queries, flat hybrid retrieval for pinpoint queries.

**(d) Retrieval-optional queries / cost-sensitive** — add **adaptive RAG** routing (direct LLM for simple, single-step for moderate, multi-step for complex), optionally with a **Self-RAG-style groundedness gate** on the direct path.

**(e) Noisy corpus / correctness-critical** — add **CRAG**: the retrieval evaluator catches misses before generation and routes to correction; combine with a **CoVe or self-reflective check** on the answer (§7.2).

**(f) Complex multi-step queries** — add **agentic RAG** (LangGraph ReAct agent over retrieval tools) with iteration/time budgets; put (d)'s router *inside* the agent as its first decision.

**(g) Token-level precision** — add **ColBERT as the reranker** on top of (a)'s wide candidate set, or as the dense stage when terminology matters (legal, regulatory identifiers).

**(h) High-stakes attribution** — add **RARR** (post-hoc evidence + citation editing) or in-generation citation enforcement (§6.6, §7.3); combine with CoVe for claim-level verification.

**(i) Long-form generation with evolving needs** — add **FLARE** (token-probability variant) or **ITER-RETGEN** (no token-probability requirement) as the retrieval loop around generation; cap iterations at 2–4 to bound drift.

### 10.3 A recommended layered architecture

The layers compose cleanly because each has a single responsibility. This diagram is the union of the recipes above; a given deployment activates only the layers its measured failures justify.

```
                        ┌─────────────────────────────────────────────┐
                        │              Query (user)                    │
                        └───────────────────┬─────────────────────────┘
                                             ▼
                        ┌─────────────────────────────────────────────┐
                        │  L1  Router / Adaptive RAG classifier       │  complexity + intent
                        │      (rules · embeddings · LLM)             │  (d)
                        └──────┬──────────────┬──────────────┬────────┘
                               ▼              ▼              ▼
                        ┌──────────┐   ┌──────────────┐   ┌──────────────────┐
                        │ Direct   │   │ Single-step  │   │ Multi-step       │
                        │ (no RAG) │   │ RAG          │   │ (agentic loop)   │  (f)
                        └──────────┘   └──────┬───────┘   └────────┬─────────┘
                                              ▼                    ▼
                        ┌───────────────────────────────────────────────┐
                        │  L2  Query preparation                          │
                        │      rewrite · HyDE · expand/fuse · decompose   │  §4
                        └───────────────────────┬────────────────────────┘
                                                ▼
                        ┌───────────────────────────────────────────────┐
                        │  L3  Retriever                                 │
                        │      hybrid (BM25+dense+RRF) → wide top-50-100 │  §5.1-5.2
                        │      optional: ColBERT stage · sentence-window │  §5.3-5.5
                        └───────┬───────────────────────┬───────────────┘
                                ▼                       ▼
                        ┌────────────────┐      ┌───────────────────────────┐
                        │  L4  Rerank    │      │  L4b  Graph / RAPTOR path │
                        │  cross-enc /   │      │  GraphRAG local+global    │
                        │  ColBERT / RRF │      │  or RAPTOR tree           │  §5.6-5.7
                        └───────┬────────┘      └────────────┬──────────────┘
                                └───────────────┬────────────┘
                                                ▼
                        ┌───────────────────────────────────────────────┐
                        │  L5  Context assembly                           │
                        │      dedup · relevance filter · compress        │  §6.2-6.7
                        │      (LLMLingua) · order (lost-in-middle)       │
                        └───────────────────────┬────────────────────────┘
                                                ▼
                        ┌───────────────────────────────────────────────┐
                        │  L6  Generation                                 │
                        │      grounded prompt · citations · constrained │  §7
                        │      decoding for structured answers            │
                        └───────────────────────┬────────────────────────┘
                                                ▼
                        ┌───────────────────────────────────────────────┐
                        │  L7  Self-check (optional)                     │
                        │      groundedness judge · CoVe verify ·        │  §5.8-5.9,
                        │      CRAG-style correction loop                │  §7.2
                        └───────────────────────┬────────────────────────┘
                                                ▼
                        ┌───────────────────────────────────────────────┐
                        │            Answer (+ citations)                │
                        └───────────────────────────────────────────────┘
```

**Notes on the diagram:** L1 routing is where adaptive RAG, self-RAG's retrieve/no-retrieve, and agentic entry all live; L4b is a *parallel* path selected by L1 (or by an internal tool choice in the agentic loop); L5 is mandatory — every advanced retriever widens candidates and needs dedup/order; L7 is where "correctness-critical" workloads spend their extra budget.

---

## 11. Implementation Guidance

### 11.1 Framework support matrix

Where each technique lives in the major frameworks and libraries. "Custom" means the technique is a pattern you assemble from primitives (LangGraph nodes, LlamaIndex modules) rather than a one-call API.

| Technique | LangChain | LlamaIndex | Haystack | Other / notes |
|---|---|---|---|---|
| Query rewriting / multi-query | `MultiQueryRetriever`, `StepBackQAChain` | `QueryTransform`/`StepBackQueryRewrite` | Prompt-based | [query_rewriting_rag_guide.md](query_rewriting_rag_guide.md) |
| HyDE | `HypotheticalDocumentEmbedder` | `HyDEQueryTransform` | Prompt-based | [rag_vs_hyde_guide.md](rag_vs_hyde_guide.md) |
| Query routing | `RouteQuery`, `create_router` | `RouterQueryEngine`, `RouterRetriever` | `Router` component | Semantic Router (Aurelio) |
| RAG-Fusion | `ensemble_retriever` (RRF) | `QueryFusionRetriever` (RRF) | Custom | RRF via `rank_bm25`/`llama_index` fusion |
| Query decomposition | `create_decompose_qa` | `SubQuestionQueryEngine` | Custom | Agent-based variants |
| Metadata enrichment | Metadata mappers | `MetadataExtractor` + extractors | `MetadataTagger` | spaCy/GLiNER extractors |
| Hybrid + RRF | `ensemble_retriever` | `QueryFusionRetriever` (hybrid mode) | `BM25Retriever` + `EmbeddingRetriever` → `RankFusion` | [optimization guide](rag_optimization_techniques_guide.md) |
| Reranking | `ContextualCompressionRetriever` + cross-encoder | `Postprocessor` (`CrossEncoderRerank`) | `Ranker` (cross-encoder) | FlashRank, Cohere rerank, Jina |
| ColBERT | `RAGatouilleRetriever` (community) | `ColbertIndex` | `ColbertRetriever` | **RAGatouille** (colbertv2.0, PLAID) |
| Contextual retrieval | Custom (index-time loop) | Custom (index-time loop) | Custom | Anthropic blog pattern |
| Sentence-window / parent-doc | `ParentDocumentRetriever` | `SentenceWindowNodeParser`, `AutoMergingRetriever` | `SentenceWindowRetriever`-style custom | [optimization guide](rag_optimization_techniques_guide.md) |
| GraphRAG | Community integrations (graphrag-community) | `GraphRAG` pack, `PropertyGraphIndex` | Custom | **Microsoft `graphrag`** CLI; LazyGraphRAG |
| LightRAG | Community integration | Custom | Custom | **LightRAG** (PyPI) |
| RAPTOR | `langchain-rag` RAPTOR | `RaptorPack` | Custom | `raptor` PyPI package |
| Self-RAG | Custom (prompt emulation) | `SelfRAG` pack (community) | Custom | Fine-tuned variant: `selfrag` checkpoints |
| CRAG | **LangGraph Corrective RAG** example | `CorrectiveRAG` pack | Custom | Official LangGraph pattern |
| Adaptive RAG | **LangGraph Adaptive RAG** example | RouterQueryEngine (3 engines) | Custom | Official LangGraph pattern |
| FLARE | `langchain_experimental.flare.FlareChain` | Custom | Custom | Requires logprobs |
| ITER-RETGEN | Custom LangGraph loop | Custom retriever | Custom | Paper reference code |
| Multi-hop retrieval | Custom (KG + retriever chains) | `RecursiveRetriever`, `KnowledgeGraphIndex` | Custom | Property-graph stores |
| RARR | Custom | Custom | Custom | `rarr` reference repo |
| LLMLingua | `LLMLinguaCompressor` | Custom | Custom | `llmlingua` PyPI; LongLLMLingua |
| Self-consistency | Prompt pattern | `generate` + vote | Prompt pattern | Any sampler |
| CoVe | Custom chain | Custom | Custom | 4-step prompt flow |
| Constrained decoding | `PydanticOutputParser`, structured output | `StructuredOutput` | `OutputAdapter` | [constrained_decoding_frameworks_guide.md](../constrained_decoding_frameworks_guide.md) |
| Prompt caching | Provider-level | Provider-level | Provider-level | Anthropic/OpenAI caching, vLLM prefix cache |
| Agentic RAG | **LangGraph** (ReAct/plan-execute) | `ReActAgent`, `FunctionCallingAgent` | Agents (deepset) | AutoGen, CrewAI, OpenAI Assistants |

**Frameworks choice:** see [RAG Frameworks Comparison](rag_frameworks_comparison_guide.md) for stack selection; the pattern that recurs across the matrix is *LangChain/LangGraph = best breadth of official advanced patterns; LlamaIndex = best RAG-native primitives; Haystack = cleanest production componentization*.

### 11.2 Code sketches for the techniques that need them

The techniques with first-class APIs (routing, fusion, ColBERT, GraphRAG, LLMLingua) already have sketches in their sections. Three patterns that are frequently hand-rolled:

**CRAG-style evaluator + correction (LangGraph sketch):**

```python
def evaluate_retrieval(state):                      # node 1
    docs = state["docs"]
    score = relevance_judge(state["question"], docs)   # small LM or LLM judge
    return {"retrieval_score": score}

def route(state):                                   # branch
    return "correct" if state["retrieval_score"] < 0.5 else "refine"

def correct(state):                                 # node: web search / re-retrieve
    return {"docs": web_search(state["question"]) + state["docs"]}

def refine(state):                                  # node: strip irrelevant strips
    return {"docs": [strip_irrelevant(d, state["question"]) for d in state["docs"]]}

# graph: evaluate -> route -> (correct | refine) -> generate
```

**Adaptive RAG router (complexity classifier):**

```python
def classify_complexity(question: str) -> str:
    # small LLM judge, or a fine-tuned classifier
    label = judge.complete(
        "Classify as simple | moderate | complex: " + question)
    return label.strip().lower()      # -> route to direct / single / multi-step

# LangGraph: START -> classifier -> conditional_edge(
#   {"simple": direct_llm, "moderate": vector_rag, "complex": agent_loop})
```

**ITER-RETGEN loop (2–4 iterations):**

```python
query = user_query
for i in range(4):
    docs = retriever.retrieve(query)                 # hybrid + rerank
    draft = llm.generate(prompt(query, docs))
    if converged(draft) or i == 3:                   # stop condition + cap
        return draft
    query = draft                                    # generation becomes the query
```

### 11.3 Evaluating advanced techniques

Every technique in this guide must justify itself against the same measurement discipline the [optimization guide](rag_optimization_techniques_guide.md) establishes for the baseline:

- **Retrieval quality** — recall@k, precision@k, MRR, nDCG@k, hit rate on your golden set. GraphRAG/RAPTOR change the *retrieval unit* (entities/communities/summaries), so define "hit" at the *evidence* level (did the system surface the right source), not the chunk level.
- **Generation quality** — faithfulness (groundedness), answer relevance, context precision/recall — the **RAGAS** metrics (see [RAG Frameworks Comparison](rag_frameworks_comparison_guide.md) §11 for RAGAS and the other eval frameworks).
- **Latency budget** — p50/p95 end-to-end, with per-stage breakdown; agentic and multi-step techniques need explicit *budget caps* (max LLM calls per query).
- **Cost per query** — tokens in/out per query × model prices; don't forget **index-time cost** (GraphRAG extraction, contextual retrieval, RAPTOR summarization) amortized over expected query volume — a $5,000 index that serves 1M queries is $0.005/query; one that serves 1k queries is $5/query.
- **Complexity/maintenance** — new failure modes, extraction-quality feedback loops, tuning surface. A technique you can't debug is a liability even at zero cost.

**Process:** keep the golden dataset; A/B test one technique at a time against the baseline; record before/after for every metric; adopt only techniques that move a metric you care about within budget. Start with the [80/20 list](rag_optimization_techniques_guide.md) — hybrid, rerank, chunking, eval — before any technique in this guide. **Don't optimize what you haven't measured.**

## 12. Banking-Specific Guidance

### 12.1 Recommended banking stack

For a Crédit Agricole CIB context (regulatory Q&A, KYC/AML, counterparty risk research, sanctions screening, internal policy lookup), the layered architecture of §10.3 maps onto concrete workloads:

| Workload | Techniques (on top of the production baseline) | Why |
|---|---|---|
| **Regulatory Q&A** (MAS, PRA, ECB, Basel — "What are the LCR requirements under MAS 644?") | Production baseline + **CRAG** + **CoVe/self-check** + mandatory citations | Correctness is critical and the corpus is noisy (amendments, repealed annexes, version drift). CRAG catches retrieval misses before they become confident wrong answers; CoVe verifies claims; citations make every number auditable. |
| **Sanctions lists / counterparty networks / AML typologies** | **GraphRAG (or LightRAG)** + entity-resolution metadata | These are *entity-relationship* problems: "is counterparty X connected to a sanctioned entity through ownership chains?", "which entities share an address with a watchlisted name?" Vector search over chunks cannot answer hop-chains; the graph can. Entity extraction doubles as the evidence trail for audit. |
| **Customer support / internal helpdesk (mixed difficulty)** | **Adaptive RAG routing** + Self-RAG-style groundedness gate | Most tickets are simple and need no retrieval (direct LLM), a minority are complex. Adaptive routing cuts average cost/latency; the groundedness gate stops the direct path from hallucinating policy. |
| **Risk research (filings + news + internal docs)** | **Agentic RAG** (plan-and-execute or ReAct with tool-selecting router) + **RARR** attribution | Multi-source, multi-step research: SEC/EDGAR filings, market news, internal risk policies. The agent sequences tools per counterparty; RARR adds citations per claim; human-in-the-loop checkpoint before the research note is published. |
| **Corpus-level questions** ("What emerging risks does our Q3 submissions corpus discuss?") | **GraphRAG global search** or **RAPTOR** | No single passage contains the answer; these techniques synthesize over the whole corpus. |
| **Contract/ISDA clause lookup** | **ColBERT** (rerank or dense stage) + sentence-window retrieval | Token-level terminology ("force majeure", "cross-default") and clause-level answers; ColBERT's token matching is measurably better than bi-encoder embeddings on legal text. |

**Non-negotiables for every banking workload:**

- **Citations mandatory** — every claim maps to a retrieved source (document id, page, clause); citations are verified, not just prompted (§6.6, §7.3).
- **Human-in-the-loop for high-stakes** — sanctions screening results, credit decisions, regulatory filings, and any *action* (not just an answer) require a human checkpoint before downstream use. See [LLM Development Risks & Security](../llm_development_risks_security_guide.md) for the guardrail patterns (approval gates, escalation paths, read-only defaults).
- **No web-search correction without controls** — CRAG's web-search path, agentic web tools, and any external data source must sit behind the same data-governance controls as internal sources: provenance logging, allow-lists, and injection defenses.

### 12.2 Regulatory notes

**SR 11-7 (model risk management)** — the US Federal Reserve/OCC guidance on model risk applies to RAG systems used in decisions with material impact. Practical reading for RAG:

- **Document which techniques you use** — the model inventory should record each component (retriever type, reranker, routing logic, agent loops), its version, and its role in the decision chain.
- **Validate changes** — every technique change (a new reranker, GraphRAG indexing, a router threshold) is a model change and should go through the same validation discipline: golden-dataset evaluation (§11.3), backtesting against historical queries, and sign-off.
- **Evidence of effectiveness** — regulators will ask "how do you know this system is correct?" Keep the golden dataset, the metric history, and the failure-mode register. The [optimization guide](rag_optimization_techniques_guide.md) evaluation section is the operationalization of this.

**Explainability** — prefer explainable techniques where the decision matters: hybrid retrieval, reranking, and citation-based generation produce *auditable reasoning* ("these 5 documents, ranked by this cross-encoder, support this answer"). Opaque agent loops (multi-agent systems, unbounded iterative retrieval) are harder to explain and should be reserved for low-stakes or human-verified outputs — and always capped so the decision path is reconstructable.

**Audit trail** — log the full decision path per query: rewritten queries, retrieved document ids and scores, rerank scores, router decisions, agent tool calls, generation input (prompt + context), and final answer with citations. This is what makes "explainability" concrete and is the prerequisite for both SR 11-7 documentation and incident investigation.

**MAS TRM alignment** — Singapore's MAS Technology Risk Management guidelines (and the associated Notice 658 cyber hygiene requirements) map onto RAG operations: system availability (latency budgets and circuit breakers for agent loops), change management (technique changes go through controlled release), data protection (corpus access control, PII handling in extracted entities — GraphRAG extracts *entities*, which may include personal data; tokenize/anonymize before indexing), and third-party risk (managed LLM services and hosted embedding/reranking APIs are third-party dependencies — document them).

---

## 13. The Technique Selection Decision Tree

Start with the question **"what is the measured failure?"** — not "which technique is newest." Every branch below presumes you have already fixed the basics (chunking, hybrid retrieval, reranking, golden-dataset evaluation — the [optimization guide](rag_optimization_techniques_guide.md) 80/20 list) and are looking at a residual, *measured* problem.

```
Start: what problem did the golden dataset measure?
│
├─ Similarity retrieval failing on vocabulary mismatch
│   (right docs exist, literal query misses them)
│   → Query rewriting / multi-query          [§4.1]
│   → + HyDE for short/underspecified queries [§4.2 / rag_vs_hyde_guide]
│   → + RAG-Fusion if recall-limited          [§4.4]
│
├─ Multi-hop / relational questions
│   (evidence spans documents; entity links matter)
│   → GraphRAG (local search) or LightRAG      [§5.6]
│   → or multi-hop retrieval (RecursiveRetriever) [§5.13]
│
├─ Global / corpus-level questions
│   ("what does this corpus say as a whole?")
│   → GraphRAG global search                   [§5.6]
│   → or RAPTOR for long-document corpora      [§5.7]
│
├─ Retrieval-optional query mix (some queries need no grounding)
│   → Adaptive RAG routing                     [§5.10]
│   → + Self-RAG groundedness gate             [§5.8]
│
├─ Noisy corpus / retrieval sometimes returns garbage
│   → CRAG (evaluate → correct/refine)         [§5.9]
│
├─ Token-level precision needed
│   (legal/regulatory identifiers, product names)
│   → ColBERT as reranker or dense stage       [§5.3]
│
├─ Long-form generation with evolving information needs
│   → FLARE (if logprobs available)            [§5.11]
│   → or ITER-RETGEN (2–4 iterations)          [§5.12]
│
├─ Correctness critical (hallucination unacceptable)
│   → CoVe (draft → verify → refine)           [§7.2]
│   → + RARR / citation enforcement            [§5.13, §6.6]
│   → + self-consistency for quantitative Qs   [§7.1]
│
├─ Complex, multi-source, multi-step queries
│   → Agentic RAG (LangGraph ReAct / plan-and-execute) [§8]
│   → with tool-selecting router + budgets + HITL [§8.3]
│
└─ Multiple of the above? → combine per §10.2, one technique at a
   time, measuring each addition against the golden dataset (§11.3).
```

**Two warnings on the tree.** First, the branches are not mutually exclusive — a sanctions-screening system legitimately hits "multi-hop/relational" *and* "correctness critical" (combine GraphRAG + CoVe + citations per §12). Second, the tree is a *starting hypothesis*, not a verdict: the whole point of §11.3 is that you A/B test each chosen technique and discard it if it doesn't move the metric.

---

## 14. Conclusion

The advanced RAG landscape is not a menu of tricks — it is a **spectrum of control**, from fixed pipelines (naive RAG) through optimized pipelines (advanced RAG), composable modules (modular RAG), and self-directed retrieval (agentic RAG). Each rung buys capability at the price of cost, latency, complexity, and explainability.

Five rules to carry away:

1. **Measure first.** The technique that fixes your *measured* failure is the right one; the newest paper is not. Baseline (hybrid + rerank + golden dataset) before anything in this guide.
2. **Match the technique to the failure family.** Pre-retrieval for query-side failures, retrieval-side for index/match failures, post-retrieval for context-shaping failures, generation-side for answer failures, agentic for orchestration failures.
3. **Prefer one-time indexing costs over per-query costs.** Contextual retrieval and GraphRAG are expensive once and free forever per query; agent loops and ensembling are expensive *every* query.
4. **Combine deliberately, not exhaustively.** The layered architecture of §10.3 with the §10.2 recipes composes; the kitchen sink does not.
5. **In banking, auditability is a feature, not a constraint.** Citations, explainable retrieval, SR 11-7 documentation, and MAS TRM-aligned operations turn the same techniques that improve quality into the evidence regulators require.

The field moves quickly — GraphRAG and LightRAG arrived in 2024, contextual retrieval and agentic RAG matured through 2024–2025 — but the discipline of this guide is version-independent: **understand the mechanism, measure the failure, adopt one technique at a time, and keep the receipts.**

---

*End of guide. Companion material: [RAG Optimization Techniques](rag_optimization_techniques_guide.md) (the baseline playbook) · [RAG Frameworks Comparison](rag_frameworks_comparison_guide.md) (stack selection) · [RAG vs HyDE](rag_vs_hyde_guide.md) · [Query Rewriting in RAG](query_rewriting_rag_guide.md) · [Constrained Decoding Frameworks](../constrained_decoding_frameworks_guide.md) · [Hybrid Multi-Agent Systems](hybrid_multi_agent_systems_guide.md) · [LLM Development Risks & Security](../llm_development_risks_security_guide.md).*





