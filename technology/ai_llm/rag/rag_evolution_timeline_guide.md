# The RAG Evolution Timeline — From Retrieval to Agentic Systems (2014–2026+)

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Crédit Agricole CIB
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** LLM/AI Engineering Guides — the *timeline/narrative* guide of the RAG series (the WHEN and the WHY-the-shifts; the sibling guides own the HOW)
> **Companion Guides:** [Advanced RAG Techniques](advanced_rag_techniques_guide.md) (the technique deep dives) · [RAG Optimization Techniques](rag_optimization_techniques_guide.md) (the practical playbook) · [RAG Frameworks Comparison](rag_frameworks_comparison_guide.md) (the orchestration tooling) · [RAG Evaluation Methodology](rag_evaluation_methodology_guide.md) (the eval playbook) · [RAG vs Long-Context LLMs](rag_vs_long_context_llms_guide.md) · [RAG vs HyDE](rag_vs_hyde_guide.md) · [Query Rewriting in RAG](query_rewriting_rag_guide.md) · [Beyond RAG](beyond_rag_guide.md) (the forward-looking capstone) · [Vector Databases](vector_databases_guide.md) · [Autonomous Agents](../autonomous_agents_guide.md) · [Fine-Tuning Frameworks Comparison](../fine_tuning_frameworks_comparison_guide.md) · [Enterprise AI Platforms](../enterprise_ai_platforms_guide.md) · [Responsible AI](../implementing-responsible-ai.md)
> **Last Updated:** August 2026

---

## Table of Contents

1. [Introduction — Why a Timeline](#1-introduction--why-a-timeline)
2. [The Pre-RAG Era — Why Retrieval Was Inevitable (Before 2020)](#2-the-pre-rag-era--why-retrieval-was-inevitable-before-2020)
3. [The Birth of RAG (2020) — Lewis et al.](#3-the-birth-of-rag-2020--lewis-et-al)
4. [The Naive RAG Era (2020–2023) — The Baseline That Built an Industry](#4-the-naive-rag-era-2020-2023--the-baseline-that-built-an-industry)
5. [The Advanced RAG Era (2023) — Pre- and Post-Retrieval Optimization](#5-the-advanced-rag-era-2023--pre--and-post-retrieval-optimization)
6. [The Modular RAG Era (2024) — RAG as a LEGO System](#6-the-modular-rag-era-2024--rag-as-a-lego-system)
7. [The Agentic RAG Era (2024–2026) — RAG as an Agent Skill](#7-the-agentic-rag-era-2024-2026--rag-as-an-agent-skill)
8. [The Challenges and the Debates — Long Context, Fine-Tuning, Evaluation](#8-the-challenges-and-the-debates--long-context-fine-tuning-evaluation)
9. [The 2026+ State — RAG 2.0 and RAG as a Feature](#9-the-2026-state--rag-20-and-rag-as-a-feature)
10. [The Master Timeline and the Lessons](#10-the-master-timeline-and-the-lessons)
11. [Summary — RAG Evolution in One Page](#11-summary--rag-evolution-in-one-page)
12. [Glossary](#12-glossary)
13. [References and Verification Notes](#13-references-and-verification-notes)

---

## 1. Introduction — Why a Timeline

RAG — Retrieval-Augmented Generation — is by now the most successful pattern for grounding large language models in real knowledge. Every enterprise AI architecture in 2026 contains one: a vector index, a retriever, a reranker, an LLM, and a prompt template. But the thing we now call "RAG" has a **history with hard dates**, and that history explains the present better than any technique catalogue does.

This guide is the **timeline and narrative** of the RAG series. The sibling guides answer *how*: [advanced_rag_techniques_guide.md](advanced_rag_techniques_guide.md) dissects the techniques, [rag_optimization_techniques_guide.md](rag_optimization_techniques_guide.md) is the production playbook, [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md) the evaluation playbook. **This guide answers *when* and *why-the-shifts*:** what happened in which year, which paper moved the paradigm, and — most importantly — what limitation of the previous paradigm forced each shift.

### 1.1 The five-act arc (read this if you read nothing else)

| Act | Era | Years | One-liner |
|---|---|---|---|
| Act 0 | **Pre-RAG** | before 2020 | Retrieval is 50 years old (BM25, tf-idf); neural memory research begins (NTM, Memory Networks); REALM and RETRO prove retrieval + pretraining works |
| Act 1 | **RAG birth** | 2020 | Lewis et al. coin the term "RAG" and fuse parametric + non-parametric memory |
| Act 2 | **Naive RAG** | 2020–2023 | retrieve-then-generate becomes the canonical pipeline; ChatGPT turns it into an industry ("chat with your data") |
| Act 3 | **Advanced RAG** | 2023 | Pre/post-retrieval optimization: chunking, HyDE, RAG-Fusion, reranking |
| Act 4 | **Modular RAG** | 2024 | RAG becomes a reconfigurable module system; GraphRAG adds structure |
| Act 5 | **Agentic RAG** | 2024–2026 | Self-RAG, CRAG, Adaptive-RAG; RAG becomes a skill inside agent loops |
| Act 6 | **Embedded RAG** | 2025–2026+ | "RAG 2.0", RAG as a default feature of agent platforms, long-context-native hybrids |

### 1.2 What this guide covers

- **The pre-RAG context** — why context windows made retrieval inevitable; the traditional IR lineage (tf-idf → BM25); the neural-memory precursors (Memory Networks, Neural Turing Machines); the direct ancestors REALM and RETRO.
- **The birth** — the Lewis et al. 2020 paper, its two formulations (RAG-Sequence / RAG-Token), and why it coined the term.
- **The four waves** — naive → advanced → modular → agentic, each with its canonical papers, its production reality, and the limitation that ended it.
- **The challenges and debates** — long context (Gemini 1.5 et al.), *Lost in the Middle*, RAG vs long-context, RAG vs fine-tuning, and the evolution of evaluation.
- **The 2026+ state** — the informal "RAG 2.0", RAG as a feature, the trends, and the banking lens.
- **The master timeline** — one table from 2014 to 2026, the paradigm shifts, the constants, and the lessons.

> **Honesty note.** Where a claim is verified (paper, arXiv ID, date), it is stated as fact and listed in §13. Where the record is fuzzy — most notably the informal terms "RAG 2.0" and "agentic RAG" — the guide **flags it explicitly** rather than inventing a canonical citation.

### 1.3 How this guide fits the RAG series

This guide is the **chronological spine** of the series. Each sibling guide owns a deep dive; this guide owns the story — it cross-references rather than duplicates:

| Sibling guide | What it owns | How this timeline uses it |
|---|---|---|
| [Advanced RAG Techniques](advanced_rag_techniques_guide.md) | The technique catalogue (GraphRAG, Self-RAG, CRAG, adaptive, agentic RAG, RAG-Fusion, contextual retrieval) | The "what" behind every era's era-defining technique |
| [RAG Optimization Techniques](rag_optimization_techniques_guide.md) | The production playbook (chunking, hybrid retrieval, reranking, eval) | The naive/advanced stack that every era builds on |
| [RAG Frameworks Comparison](rag_frameworks_comparison_guide.md) | LangChain / LlamaIndex / Haystack / DSPy orchestration | The modular era's substrate; the agentic era's framework vocabulary |
| [RAG vs Long-Context LLMs](rag_vs_long_context_llms_guide.md) | The context-window arms race, effective context, hybrids | The evidence base for §8.1–8.3 |
| [RAG vs HyDE](rag_vs_hyde_guide.md) / [Query Rewriting in RAG](query_rewriting_rag_guide.md) | Query-side levers in depth | The advanced-era pre-retrieval story (§5.3) |
| [RAG Evaluation Methodology](rag_evaluation_methodology_guide.md) / [RAG Evaluation Tools Comparison](rag_evaluation_tools_comparison_guide.md) | Eval playbook and tooling (Ragas, TruLens) | The evaluation evolution (§8.5) |
| [Vector Databases](vector_databases_guide.md) | The storage and indexing layer | The naive-era infrastructure (§4.1) |
| [Beyond RAG](beyond_rag_guide.md) | The forward-looking capstone (nine paradigms) | The future (§9.5) |
| [Autonomous Agents](../autonomous_agents_guide.md) / [LLM Agent Use Cases](../llm_agent_use_cases.md) | The agent substrate and use cases | The agentic era's orchestration side (§7) |
| [Fine-Tuning Frameworks Comparison](../fine_tuning_frameworks_comparison_guide.md) | RAG vs fine-tuning head-to-head | The §8.4 debate |
| [Enterprise AI Platforms](../enterprise_ai_platforms_guide.md) / [Responsible AI](../implementing-responsible-ai.md) | Platform and governance layers | The enterprise/banking endpoint (§9.2, §9.4) |

---

## 2. The Pre-RAG Era — Why Retrieval Was Inevitable (Before 2020)

### 2.1 The context: early LLMs and tiny context windows

The single most important fact for understanding RAG's origin is that **early neural language models could not hold enough text in memory to answer questions about it**. The transformer architecture that made LLMs possible was built with a fixed, small attention window:

| Model | Year | Context window |
|---|---|---|
| BERT | 2018 | 512 tokens |
| GPT-1 | 2018 | 512 tokens |
| GPT-2 | 2019 | 1,024 tokens |
| GPT-3 | 2020 | 2,048 tokens |
| Early instruction-tuned models (InstructGPT era) | 2021–2022 | 2,048–4,096 tokens |

A 2,048-token window is roughly **1,500 words** — a few pages. An entire book, a regulatory handbook, a bank's policy corpus, a decade of earnings reports: none of it could fit. The model's only "knowledge" was whatever had been compressed into its weights during pretraining — the **parametric memory**. If the answer wasn't in the weights, the model had no way to reach it. This is the structural constraint that made retrieval not an option but a **necessity**.

> **Banking aside.** A CACIB policy library is measured in thousands of pages; even today's 1M-token windows cannot hold it all. The pre-RAG constraint — *weights-only knowledge* — is why the bank's first generative-AI pilots were all retrieval projects. The timeline below is the intellectual history of that decision.

### 2.2 "Retrieval" before RAG — the traditional IR lineage

Long before neural networks, the information-retrieval (IR) field had solved "find the right document for a query" as a standalone discipline:

- **tf-idf** (term frequency–inverse document frequency, rooted in Salton's vector space model, 1960s–70s): scores a document by how distinctive its terms are. Cheap, interpretable, still everywhere in search.
- **BM25** (Robertson & Walker, 1994, "Okapi BM25"): the probabilistic ranking function that became the default for lexical search — still the sparse baseline against which every dense retriever is measured. It is also, in 2026, still a live component of hybrid RAG (see [rag_optimization_techniques_guide.md](rag_optimization_techniques_guide.md)).
- **Search engines and inverted indexes**: the retrieval infrastructure (tokenization, indexing, ranking, caching) that RAG would later borrow wholesale.

The key insight RAG inherited: **a small, fast, imperfect retriever plus a large corpus beats any system that must read everything**. That is the oldest lesson in IR, and the deepest reason RAG works.

### 2.3 The memory-augmented precursors — neural networks with external memory

A second lineage — neural networks with *explicit external memory* — runs through the 2010s. These are the direct intellectual ancestors of "parametric + non-parametric memory":

- **Neural Turing Machines** (Graves, Wayne, Danihelka, 2014, arXiv:1410.5401): an RNN augmented with a differentiable external memory bank and read/write heads. Shows a network can *learn* to store and retrieve — but at toy scale.
- **Memory Networks** (Weston, Chopra, Bordes, arXiv:1410.3916, October 2014; published ICLR 2015): an external memory of facts plus attention-based addressing, aimed at question answering. The follow-up **End-to-End Memory Networks** (Sukhbaatar et al., 2015) removed the supervision on the memory addressing, making it trainable end-to-end.
- **Differentiable Neural Computers** (Graves et al., 2016): NTM + Memory Networks, applied to graph traversal and reasoning.

These systems were important conceptually — *models should be able to consult external knowledge* — but they scaled terribly. None of them survived contact with large corpora. What they contributed to the RAG story is the **vocabulary**: "memory", "read", "retrieve", "external knowledge". When Lewis et al. describe RAG as "parametric memory + non-parametric memory", they are standing on this lineage.

### 2.4 REALM (2020) — retrieval-augmented *pretraining*

**REALM — Retrieval-Augmented Language Model Pre-Training** (Guu, Lee, Tung, Pasupat, Chang; arXiv:2002.08909, February 2020; ICML 2020) is the closest direct ancestor of RAG. Key moves:

- **Differentiable retrieval during pretraining**: while masked-LM pretraining, the model learns a neural retriever over Wikipedia in the same gradient step as the language model. The retrieved document becomes part of the masked-token prediction.
- **Knowledge goes into a corpus, not only into weights**: the model can answer questions about facts that appear in the corpus but were never memorized.
- **The retriever is trained jointly** — not bolted on — which is the crucial advance over earlier "retrieve then fine-tune" attempts.

REALM showed retrieval could be baked into the *learning* process itself, and beat much larger parametric-only models on open-domain QA benchmarks (Natural Questions, WebQuestions, CuratedTrec).

### 2.5 RETRO (2021) — retrieval at trillion-token scale

**RETRO — RETrieval-TransfORmer** (Borgeaud et al., DeepMind; arXiv:2112.04426, December 2021) scaled the idea up: an autoregressive transformer that conditions on chunks retrieved from a **2-trillion-token database**, using frozen retrieval + chunked cross-attention. The headline result: a 7.5B-parameter RETRO achieved performance comparable to **GPT-3 (175B)** on the Pile — a ~25x parameter-efficiency gain from retrieval. RETRO proved retrieval could substitute for model scale, which was the strongest possible argument for the whole paradigm.

### 2.6 The open-domain QA bridge — DrQA, ORQA, DPR (2017–2020)

A third, pragmatic lineage fed directly into the RAG paper: open-domain question answering, where "retrieve then read" had been the winning recipe for years:

- **DrQA** (Chen, Fisch, Weston, Bordes, 2017) — a "retriever-reader": tf-idf retrieve passages, then a reading-comprehension model extracts the answer. The template RAG would later neuralize.
- **ORQA** (Lee, Chang, Toutanova, 2019) — an *inverse cloze task* pretrained the retriever jointly with the reader; retrieval treated as a latent variable with no explicit supervision on retrieval.
- **DPR — Dense Passage Retrieval** (Karpukhin et al., 2020) — a bi-encoder dense retriever trained with in-batch negatives, which crushed BM25 on open-domain QA and became the retriever inside the RAG paper.

The RAG paper is literally *DPR + BART with a latent-variable marginalization*: the QA lineage supplied the retrieval machinery, the memory line supplied the framing, and Lewis et al. supplied the generation.

### 2.7 The pre-RAG timeline

| Year | Milestone | Significance |
|---|---|---|
| 1960s–70s | Vector space model, tf-idf (Salton et al.) | The formal foundation of ranked retrieval |
| 1994 | BM25 (Robertson & Walker) | The lexical ranking baseline that survives into hybrid RAG |
| 2014 | Neural Turing Machines (Graves et al., arXiv:1410.5401) | Neural networks with differentiable external memory |
| 2014–15 | Memory Networks (Weston, Chopra, Bordes; ICLR 2015) | Attention-addressed external memory for QA — the "memory" vocabulary |
| 2015 | End-to-End Memory Networks (Sukhbaatar et al.) | Trainable memory without supervision on addressing |
| 2016 | Differentiable Neural Computer (Graves et al.) | Memory + reasoning; the last big gasp of this line |
| 2018 | BERT / GPT-1 (512-token windows) | The context-window ceiling that makes retrieval necessary |
| 2019 | kNN-LM (Khandelwal et al., arXiv:1911.00172) | Nearest-neighbour lookup over a datastore *during* generation |
| 2020 | REALM (Guu et al., arXiv:2002.08909) | Differentiable retrieval inside pretraining; SOTA open-domain QA |
| 2021 | RETRO (Borgeaud et al., arXiv:2112.04426) | 7.5B ≈ GPT-3 175B via a 2T-token retrieval database |

### 2.8 Why the pre-RAG era matters

Three threads — **lexical IR** (tf-idf → BM25), **neural memory** (NTM → Memory Networks), and **retrieval-augmented pretraining** (REALM → RETRO) — converge in 2020. RAG is not a bolt from the blue; it is the point where a 50-year-old retrieval discipline meets a 5-year-old generative revolution. Knowing this reframes the "is RAG a hack?" debate: retrieval is not a workaround for weak LLMs, it is the *original* answer to the knowledge-access problem, and RAG is simply the version of it that can talk.

---

## 3. The Birth of RAG (2020) — Lewis et al.

### 3.1 The paper

**"Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks"** — Patrick Lewis, Ethan Perez, Aleksandra Piktus, Fabio Petroni, Vladimir Karpukhin, Naman Goyal, Heinrich Küttler, Mike Lewis, Wen-tau Yih, Tim Rocktäschel, Sebastian Riedel, Douwe Kiela (Facebook AI Research, UCL, NYU). **arXiv:2005.11401, May 2020; NeurIPS 2020.**

This is the paper that **coined the term "RAG"** — no earlier work uses the name. Every "RAG" in this repository's series traces back to this single title.

### 3.2 The concept — parametric memory + non-parametric memory

The paper's central framing: a pretrained LM is a **parametric memory** — knowledge is stored in weights, and it is *frozen at a point in time, expensive to update, and opaque*. RAG augments it with a **non-parametric memory** — a dense vector index over a corpus (they used Wikipedia) that can be updated by swapping documents, inspected by looking at retrieved chunks, and cited by pointing at sources.

The architecture in one sentence: **use DPR** (Dense Passage Retrieval, Karpukhin et al. 2020) **to retrieve the top-k passages, then feed them to BART (Lewis et al. 2019) to generate the answer, marginalizing over the retrieved passages.** The retriever and generator are trained jointly, end-to-end, with the retrieval treated as a latent variable.

### 3.3 RAG-Sequence vs RAG-Token — the two formulations

The latent-variable framing produced two ways to marginalize over retrieved documents:

| Formulation | Mechanism | Character |
|---|---|---|
| **RAG-Sequence** | Retrieve top-k once; generate the *entire* output sequence conditioned on the same retrieved document; marginalize the sequence probability over the k documents | Coherent, document-level grounding; better when one document contains the whole answer |
| **RAG-Token** | At *each generation step*, marginalize the next-token probability over the k retrieved documents | More flexible; can blend evidence across documents token-by-token; higher compute |

Both formulations beat non-augmented BART and T5 on three open-domain QA benchmarks (Natural Questions, WebQuestions, CuratedTrec) and on FEVER fact verification — at the time, **state of the art, with a model a fraction of the size of the alternatives**.

### 3.4 Why RAG — the motivation

The paper targets **knowledge-intensive tasks**: open-domain QA, fact verification, entity extraction — tasks where the answer is *not in the weights* and cannot be recalled, only retrieved. The motivation is threefold, and every one of these arguments still drives RAG decisions in 2026:

1. **Parametric-only LMs hallucinate and go stale** — knowledge frozen at pretraining time, no source, no update path.
2. **Retrieval grounds the answer** — the model can point at evidence, and the system's knowledge is *the corpus*, which the operator controls.
3. **Retrieval is cheaper than scale** — RETRO had shown retrieval substitutes for parameters; RAG showed it also substitutes for fine-tuning on downstream knowledge.

### 3.5 The birth's significance

- **The term.** "Retrieval-Augmented Generation" is coined here; before this there were retrieval-*pretrained* models (REALM), retrieval-*enhanced* transformers (RETRO), and retrieval-*interpolated* language models — none of them "RAG".
- **The template.** Index → retrieve → generate becomes the canonical recipe that every naive RAG system, every framework, and every enterprise pilot would copy for the next three years.
- **The tension.** RAG was born at Facebook AI Research in the era of *small* models (BART, T5). Its authors could not have predicted that three years later a chatbot would make "RAG" a board-level term — but the paper's architecture proved robust enough to survive the transition to instruction-tuned giant models, which is exactly what happened in 2023.

### 3.6 Reception and legacy (2020–2022)

Between publication and the ChatGPT era, RAG was a research technique with three notable afterlives:

- **The HuggingFace integration** — `transformers` shipped a RAG implementation (`RagTokenizer`, `RagRetriever`, `RagSequenceForGeneration`, `RagTokenForGeneration`) in late 2020, making both formulations directly usable and seeding a generation of tutorials, forks, and enterprise experiments.
- **The knowledge-intensive benchmark wave** — RAG variants became reference systems on Natural Questions / TriviaQA / WebQuestions and FEVER fact verification, and the paper's "parametric vs non-parametric" framing was adopted across the field (e.g., Atlas, 2022, which pushed retrieval-augmented pretraining further at scale).
- **The quiet years** — 2021–2022 RAG research concentrated on efficiency and retriever quality (dense-retrieval improvements, hybrid search, better indexing); the *application* wave had to wait for instruction-tuned models with chat interfaces. The architecture was proven, but the market was not ready — the 2023 tooling wave was the demand side finally arriving.

---

## 4. The Naive RAG Era (2020–2023) — The Baseline That Built an Industry

### 4.1 The canonical pipeline — retrieve-then-generate

Between the paper and the chatbot era, RAG's architecture ossified into a fixed three-stage pipeline — the pattern that the [Gao et al. 2023 survey](https://arxiv.org/abs/2312.10997) later named **"Naive RAG"**:

```
Corpus ──► Indexing ──► Retrieval ──► Generation ──► Answer
            chunk       top-k        prompt-
            embed       query        stuff
            store       match
```

The three stages, and their naive defaults:

1. **Indexing.** Split the corpus into chunks (**fixed-size chunking**, typically 256–512 tokens, with **overlap** of 10–20% to avoid cutting answers at boundaries), embed each chunk with a dense model, and store the vectors in a vector database with an inverted-index fallback (see [vector_databases_guide.md](vector_databases_guide.md) for the storage layer).
2. **Retrieval.** Embed the user query and retrieve the **top-k** chunks by vector similarity. Sparse (BM25) and dense routes existed side by side: the **sparse** route used lexical BM25/tf-idf; the **dense** route used embeddings — OpenAI's `text-embedding-ada-002` (January 2023) and the BAAI **BGE** family (September 2023 onwards, with BGE-M3 in early 2024) became the de facto defaults, alongside E5 and later frontier embedding models.
3. **Generation.** Stuff the top-k chunks into the prompt ("use only the following context…"), ask the LLM to answer, and return the text. No feedback, no iteration, no routing — **context-stuffed, single-shot generation**.

### 4.2 The limitations of naive RAG

Naive RAG worked for demos and failed in production in three predictable ways — the exact failure catalogue that the *next* era would be built to fix:

| Limitation | What goes wrong | Root cause |
|---|---|---|
| **Retrieval quality** | The right chunk is not in the top-k; the wrong chunk is (vocabulary mismatch, chunk-boundary cuts, embedding drift) | A lossy filter between corpus and model — *garbage in, gospel out* |
| **Hallucination / unfaithfulness** | The generator ignores or contradicts the retrieved context — especially when the context is weak or the model is confident | No mechanism forces the answer to stay grounded; single pass, no verification |
| **Integration / context misuse** | The answer is buried in the *middle* of the stuffed context, exactly where models attend worst; too much context dilutes the signal | Position bias + prompt bloat (see *Lost in the Middle*, §8.2) |

There were two further structural gaps: **no routing** (every query, however trivial, pays the full retrieval cost) and **no iteration** (multi-hop questions cannot be answered by one retrieve-pass). These four limitations — quality, faithfulness, context integration, and rigidity — are the *reasons* the advanced, modular, and agentic eras exist. The [RAG evaluation playbook](rag_evaluation_methodology_guide.md) formalizes exactly these failure modes into metrics.

### 4.3 Naive RAG in production — the ChatGPT-era explosion (2023)

Academically, naive RAG was already the template from 2020. Commercially, it exploded in **2023**:

- **ChatGPT (November 2022)** made LLMs mainstream; within weeks the first "chat with your data" tutorials appeared, and by spring 2023 **RAG was the default way to make GPT answer questions about your documents** — no fine-tuning required.
- **LangChain and LlamaIndex** (both launching in force during 2023) productized the naive pipeline into a few lines of code: load, split, embed, store, retrieve, prompt. Their defaults *were* naive RAG — fixed-size chunking, top-k, no reranking.
- **Vector databases** (Pinecone, Weaviate, Milvus, Qdrant, pgvector) turned from niche research tools into infrastructure, priced and marketed specifically for RAG (see [vector_databases_guide.md](vector_databases_guide.md)).
- **Enterprise pilots** — banks included — stood up their first "ask your policy manual" bots on exactly this stack.

### 4.4 The early systems — from ChatPDF to enterprise copilots

The 2023 naive wave produced a recognizable product genre, all sharing the same three-stage skeleton:

- **Document-chat SaaS** (ChatPDF, AskYourPDF, and the "chat with your PDF" feature inside every productivity suite) — upload a document, get a chatbot over it. The corpus is one file; naive RAG is genuinely sufficient.
- **Open-source starter stacks** (LangChain + a vector DB + OpenAI embeddings) — the canonical tutorial combination; thousands of internal tools were built on it within months.
- **Enterprise copilots** — the bank's first-generation assistants: policy Q&A, HR handbooks, IT knowledge bases. Typically: PDF ingestion → fixed-size chunking → vector store → top-k → GPT-4. And typically: demos that impressed and production behaviour that did not — retrieval misses on regulatory questions, unsourced claims, stale indexes.

The genre's lesson (which the advanced era then productized): **the naive stack is a UI for a demo, not an architecture for a bank** — but it was the necessary on-ramp that taught the whole industry (and the whole bank) what RAG was, and what it wasn't.

### 4.5 The era's significance

Naive RAG is the **baseline**, and that is its permanent value: it is the null hypothesis every better system is measured against, and the [Gao et al. 2023 survey](https://arxiv.org/abs/2312.10997) (*Retrieval-Augmented Generation for Large Language Models: A Survey*, Tongji/Fudan, arXiv:2312.10997) institutionalized the **naive / advanced / modular** taxonomy that this guide's structure follows. By late 2023 the industry-wide experience was unanimous: *naive RAG demos well and disappoints under load* — which is precisely the observation that opens the advanced era.

---

## 5. The Advanced RAG Era (2023) — Pre- and Post-Retrieval Optimization

### 5.1 The framing — pre-retrieval vs post-retrieval

The advanced era's organizing insight is that naive RAG's three failure modes live in *specific stages*, so the fixes are stage-local. The [Gao et al. 2023 survey](https://arxiv.org/abs/2312.10997) and the follow-up *Searching for Best Practices in RAG* (Gao et al., July 2024, arXiv:2407.01219) codified two families:

- **Pre-retrieval optimizations** — improve what enters the retriever (chunking, query rewriting, metadata).
- **Post-retrieval optimizations** — improve what enters the generator (reranking, compression, selection).

### 5.2 Pre-retrieval — chunking strategies

Chunking went from "fixed size + overlap" to a design decision with its own literature:

- **Semantic chunking** — split at meaning boundaries (sentence/paragraph boundaries, topic shifts detected by embedding similarity) rather than fixed token counts, so chunks are self-contained units of meaning.
- **Small-to-big / parent-child** — retrieve small precise chunks (high recall on exact evidence) but feed the *parent* document or larger window to the LLM (enough context to answer well). This is the "small-to-big" family, now a default pattern in production RAG (see [rag_optimization_techniques_guide.md](rag_optimization_techniques_guide.md) for the full chunking playbook).

### 5.3 Pre-retrieval — query rewriting: multi-query, HyDE, RAG-Fusion

The retriever is only as good as the query. Three query-side techniques define this era:

- **Multi-query retrieval** — an LLM expands one user question into several paraphrases/variants; retrieve for each; merge results. Cheap, effective, widely adopted.
- **HyDE — Hypothetical Document Embeddings** (Gao, Ma, Zhao, Lin, Callan; **arXiv:2212.10496, December 2022**; ACL 2023): instead of embedding the query, *generate a hypothetical answer document* with the LLM and embed *that*. Zero-shot, no relevance labels needed — it made dense retrieval work on queries whose vocabulary doesn't overlap the corpus. Its known weakness: it inherits the generator's biases (see the dedicated [rag_vs_hyde_guide.md](rag_vs_hyde_guide.md)).
- **RAG-Fusion** — generate multiple sub-queries, retrieve for each, then merge the ranked lists with **Reciprocal Rank Fusion** (RRF, from Cormack, Clarke & Büttcher, SIGIR 2009). The method was popularized in mid-2023 by Zachary Liu's "Forget RAG, the Future is RAG-Fusion" essay and Adrian Raudaschl's open-source implementation (900+ stars), and received a formal write-up in *RAG-Fusion: a New Take on Retrieval-Augmented Generation* (**arXiv:2402.03367, February 2024**). Note the honest dating: RRF is 2009, the RAG-Fusion *name* is 2023, the *paper* is 2024. (See also [query_rewriting_rag_guide.md](query_rewriting_rag_guide.md).)

### 5.4 Post-retrieval — reranking with cross-encoders

The single highest-leverage post-retrieval fix is **reranking**: the first stage (bi-encoder, fast, recall-oriented) returns a wide top-k (50–100); a **cross-encoder** — which scores the query and each passage jointly, with full attention between them — reorders the list (precision-oriented). Cross-encoder rerankers dominated the field: **Cohere Rerank** (API launched 2023, v2/v3 through 2024), `bge-reranker` (BAAI), `monoT5`/`RankT5`, and later listwise rerankers. The effect is dramatic: reranking top-50 to top-10 routinely lifts end-to-end answer quality by 5–15 points on QA-style evals because the generator sees better context. (Practical reranking guidance: [rag_optimization_techniques_guide.md](rag_optimization_techniques_guide.md).)

### 5.5 The advanced RAG summary

| Optimization | Family | Era-defining technique | Effect |
|---|---|---|---|
| Semantic / small-to-big chunking | Pre-retrieval | Parent-child retrievers | Chunks that are self-contained and answer-sized |
| Query rewriting | Pre-retrieval | Multi-query, **HyDE (2022)**, **RAG-Fusion (2023)** | Fixes vocabulary mismatch; expands recall |
| Metadata & filtering | Pre-retrieval | Time/entity/source filters | Shrinks the search space |
| Reranking | Post-retrieval | Cross-encoders (Cohere Rerank, bge-reranker) | Precision jump before generation |
| Context compression | Post-retrieval | Summarize/compress retrieved chunks | Less noise, less token cost |

### 5.6 The era's significance

The advanced era is when **"RAG" became a performance engineering discipline rather than a wiring exercise**. The quality jump came from retrieval-*aware* generation: the system no longer just *finds* context, it *curates* it — rewriting the query to match the index, reranking what it found, and shaping what the LLM sees. By late 2023, "naive RAG" had become a pejorative, and every production architecture diagram in the industry grew the familiar pre-/post-retrieval boxes around the canonical pipeline. (The full technique map lives in [advanced_rag_techniques_guide.md](advanced_rag_techniques_guide.md).)

---

## 6. The Modular RAG Era (2024) — RAG as a LEGO System

### 6.1 The Modular RAG paradigm — the paper

**"Modular RAG: Transforming RAG Systems into LEGO-like Reconfigurable Frameworks"** (Yunfan Gao, Yun Xiong, Meng Wang, Haofen Wang; **arXiv:2407.21059, July 2024**). The paper's argument: the fixed linear pipeline — and even the pre/post-retrieval taxonomy — is too rigid. It proposes decomposing RAG into **independent modules and specialized operators** that can be recombined per use case: a reconfigurable framework rather than a pipeline.

### 6.2 The modules and operators

| Module family | Operators it contains | Example instantiations |
|---|---|---|
| **Indexing** | chunk, embed, store, update | semantic chunking, small-to-big, incremental indexing |
| **Pre-retrieval** | rewrite, expand, route, filter | HyDE, multi-query, metadata filters |
| **Retrieval** | search (sparse/dense/hybrid), graph-walk, SQL, API | BM25 + dense hybrid, GraphRAG traversal |
| **Post-retrieval** | rerank, compress, select, dedupe | cross-encoder reranking, context compression |
| **Generation** | predict, verify, cite, refine | grounded generation, citation formatting |
| **Orchestration** | plan, loop, decide, fuse | linear → branching → conditional → looping patterns |

The key vocabulary shift: **search, memory, routing, fusion, predict** — RAG is no longer a single path but a design space of *composable* capabilities. A system can be "RAG" and route simple queries straight to generation, send multi-hop questions through an iterative loop, and fuse results from a vector index *and* a knowledge graph — all within one framework. This is the intellectual bridge from the fixed pipeline to the agentic era: **modularity is what makes autonomy expressible**.

### 6.3 GraphRAG (2024) — Microsoft

**"From Local to Global: A Graph RAG Approach to Query-Focused Summarization"** (Darren Edge, Ha Trinh, Newman Cheng, Joshua Bradley, Alex Chao, Apurva Mody, Steven Truitt, Jonathan Larson; Microsoft Research; **arXiv:2404.16130, April 2024**; open-sourced July 2024). GraphRAG attacks naive RAG's blind spot: **global questions** — "what are the main themes across the entire corpus?" — which chunk retrieval cannot answer because no single chunk contains the answer.

The pipeline:

1. **Entity and relation extraction** — an LLM reads the corpus and extracts entities and relationships into a **knowledge graph** (with entity linking to resolve mentions).
2. **Community detection** — the **Leiden algorithm** partitions the graph into hierarchical communities of related entities.
3. **Community summarization** — an LLM summarizes each community's subgraph (map-reduce over the graph, *not* over raw text).
4. **Global answer synthesis** — for a query, relevant community summaries are retrieved/synthesized, then combined into a final answer.

GraphRAG beat naive RAG decisively on query-focused summarization (QFS) benchmarks and comprehensiveness/diversity of answers — at a significant indexing cost (LLM-driven extraction is expensive), which is the trade-off every adopter must price (see [advanced_rag_techniques_guide.md](advanced_rag_techniques_guide.md) §GraphRAG for the cost/quality analysis).

### 6.4 RAG + knowledge graphs — the broader graph-enhanced family

GraphRAG is the flagship of a wider movement in 2024: **KG-RAG** (graph-constrained retrieval over biomedical ontologies), **HippoRAG** (personalized PageRank over a KG built from the corpus), **LightRAG** (lightweight dual-level retrieval), RAPTOR (tree-structured summaries — a sibling of the same impulse). The common thread: for corpora where *relationships* carry the answer (who owns what, which regulation governs which product, which counterparty links to which exposure), graph structure beats vector similarity. For a bank, that is a large fraction of the interesting questions.

### 6.5 The era's significance

The modular era is when **RAG stopped being an architecture and became a design space**. The [frameworks](rag_frameworks_comparison_guide.md) (LangChain, LlamaIndex, Haystack, DSPy) had been quietly building modular abstractions all along; the Modular RAG paper gave the practice a name and a taxonomy. Two consequences for the timeline: (1) the vocabulary of *modules* prepared the field for agents, and (2) GraphRAG showed that RAG could absorb *structured* knowledge, ending the assumption that RAG == vector search.

---

## 7. The Agentic RAG Era (2024–2026) — RAG as an Agent Skill

### 7.1 The framing — agents + RAG

The fourth wave fuses RAG with the agent paradigm: instead of one fixed retrieve-then-generate pass, an **LLM agent decides when to retrieve, what to retrieve with, whether the evidence is sufficient, and whether to loop** — tool use, multi-step reasoning, and self-correction around a retrieval core. The deep dive on the agent substrate lives in [autonomous_agents_guide.md](../autonomous_agents_guide.md) and [llm_agent_use_cases.md](../llm_agent_use_cases.md); this section dates the retrieval side of that story.

> **Naming honesty.** "Agentic RAG" is a *community* term (survey papers, vendor blogs, talks) — there is no single canonical "agentic RAG" paper. What does have canonical status are its three building-block papers: Self-RAG, CRAG, and Adaptive-RAG, all verified below. The term itself is flagged as informal-but-ubiquitous.

### 7.2 Self-RAG (2023) — retrieval by self-reflection

**"Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection"** (Akari Asai, Zeqiu Wu, Yizhong Wang, Avirup Sil, Hannaneh Hajishirzi; **arXiv:2310.11511, October 2023**; ICLR 2024). The model is trained to emit **reflection tokens** alongside text:

- *Retrieve* — should I retrieve at all for this step? (adaptive retrieval: only retrieve when useful)
- *IsRel* — is the retrieved passage relevant?
- *IsSup* — is the passage supporting the statement?
- *IsUse* — is the statement useful to the answer?

The generator critiques its own output at generation time, and inference searches over retrieve/no-retrieve and supporting/non-supporting options. This is the first system where **retrieval is decided by the model at runtime, per token-step**, and groundedness is *checked*, not assumed.

### 7.3 CRAG (2024) — corrective retrieval

**"Corrective Retrieval Augmented Generation"** (Shi-Qi Yan, Jia-Chen Gu, Yun Zhu, Zhen-Hua Ling; **arXiv:2401.15884, January 2024**). A lightweight **retrieval evaluator** scores the retrieved documents *before* generation and triggers one of three actions:

| Evaluator verdict | Action |
|---|---|
| **Correct** | Keep the retrieved docs and generate |
| **Incorrect** | Discard them; **decompose** the query into sub-queries and re-retrieve; optionally **fall back to web search** as a second retrieval source |
| **Ambiguous** | Combine retrieved docs with web-search results |

CRAG is "plug-and-play" — it wraps *any* RAG pipeline and adds the corrective loop without retraining the generator. The web-search fallback is the historically important bit: retrieval is no longer confined to the private index.

### 7.4 Adaptive-RAG (2024) — routing by question complexity

**"Adaptive-RAG: Learning to Adapt Retrieval-Augmented Large Language Models through Question Complexity"** (Soyeong Jeong, Jinheon Baek, Sukmin Cho, Sung Ju Hwang, Jong C. Park; **arXiv:2403.14403, March 2024**; NAACL 2024). A **complexity classifier** routes each query to the cheapest sufficient strategy:

- **No retrieval** — trivial questions (e.g., greetings, parametric facts the model knows);
- **Single-step retrieval** — standard RAG;
- **Multi-step retrieval** — iterative, multi-hop retrieval loops.

This is query *routing* as a first-class mechanism — the direct ancestor of today's "decide per query how much retrieval to spend", which the [beyond_rag_guide.md](beyond_rag_guide.md) identifies as the 2026 consensus.

### 7.5 Agentic retrieval — beyond the three papers

From 2024–2026 the pattern generalized: RAG became one *tool* among many in an agent's toolbox — alongside SQL, web search, calculators, and APIs — and retrieval itself became agentic:

- **Multi-tool search agents** — OpenAI **Deep Research** (February 2025) and its clones run multi-step search-and-read loops, deciding queries iteratively; the "research agent" is, structurally, an agentic RAG system with a web-scale index.
- **Contextual retrieval** (Anthropic, September 2024) — a retrieval-side technique (contextual chunk headers + BM25/dense hybrid) showing that even in the agentic era, the *indexing* stage still moves quality.
- **Agentic RAG surveys** (2025) consolidated the pattern: planning → retrieval → reflection → iteration, with DSPy-style *optimized* agentic RAG pipelines automating the loop design (see [rag_frameworks_comparison_guide.md](rag_frameworks_comparison_guide.md)).

A compact timeline of the agentic wave:

| Date | Milestone | Why it matters |
|---|---|---|
| Oct 2023 | **Self-RAG** (arXiv:2310.11511) | The model gains a *decision* to retrieve and a *critique* of groundedness |
| Jan 2024 | **CRAG** (arXiv:2401.15884) | Retrieval failure becomes *recoverable* (decompose, web fallback) |
| Mar 2024 | **Adaptive-RAG** (arXiv:2403.14403) | Query complexity becomes a *routing* signal |
| Sep 2024 | Anthropic contextual retrieval | Indexing quality still matters inside agentic stacks |
| Feb 2025 | OpenAI Deep Research | Agentic retrieval at web scale, productized |
| 2025–26 | Agentic RAG as framework default | LangGraph / LlamaIndex workflows standardize the loop |

### 7.6 The era's significance

The agentic era reframes RAG from a *pipeline* into a *skill*: the LLM (or a small orchestrator model) holds the retrieval capability and spends it deliberately — retrieve, check, correct, loop, stop. The three canonical papers form a clean lineage of autonomy: **Self-RAG** gave the model a *decision* to retrieve; **CRAG** gave it *correction* when retrieval fails; **Adaptive-RAG** gave it *routing* across strategies. Together they are the mechanism behind the industry's 2025–2026 shift from "RAG systems" to "agents that use retrieval" — and they are the bridge to the "RAG 2.0" discussion in §9.
---

## 8. The Challenges and the Debates — Long Context, Fine-Tuning, Evaluation

### 8.1 The context-window arms race (2023–2025)

While RAG matured, the constraint that created it — small context windows — was being dismantled:

| Model | Date | Context window |
|---|---|---|
| GPT-3 | 2020 | 2,048 |
| GPT-3.5 / ChatGPT | 2022 | 4,096 |
| GPT-4 | Mar 2023 | 8,192 (32k variant) |
| Claude 2 | May 2023 | 100,000 |
| Llama 2 | Jul 2023 | 4,096 |
| Mistral 7B | Sep 2023 | 8,192 (32k later) |
| GPT-4 Turbo | Nov 2023 | 128,000 |
| **Gemini 1.5 Pro** | **Feb 2024** | **1,000,000 (1M; 128k standard release)** |
| Claude 3 | Mar 2024 | 200,000 |
| Gemini 1.5 Pro (2M) | May 2024 | 2,000,000 |
| Llama 3.1 | Jul 2024 | 128,000 |
| GPT-4.1 | Apr 2025 | 1,000,000 |
| Gemini 2.5 / Claude Opus 4.1 | 2025 | 1,000,000 |

**Gemini 1.5 Pro (announced 15 February 2024)** is the inflection point: one million tokens — roughly the size of a long book — in a single context, with a research-preview extension to 10M. (Verified: 1M experimental/preview at launch, 128k standard.) The era of "the model cannot see the document" was over; a new question replaced it.

### 8.2 Lost in the Middle (2023) — the position bias that humbles long context

**"Lost in the Middle: How Language Models Use Long Contexts"** (Nelson F. Liu, Kevin Lin, John Hewitt, Ashwin Paranjape, Michele Bevilacqua, Fabio Petroni, Percy Liang; **arXiv:2307.03172, July 2023**). The finding that reframed both RAG and long-context: LLMs exhibit a **U-shaped position bias** — they use information at the *start* and *end* of a long context well, and information in the *middle* poorly, degrading steadily as context grows. For RAG this means: where you *place* the retrieved chunks in the prompt is a quality lever (see [rag_optimization_techniques_guide.md](rag_optimization_techniques_guide.md) — order retrieved chunks by relevance, put the best evidence at the edges). For long-context it means: a big window is not the same as a *well-used* big window.

### 8.3 The RAG vs long-context debate — "do we still need RAG?"

Once models can hold 1M+ tokens, the naive version of the argument writes itself: *just put the whole corpus in the prompt and delete the retriever.* The 2024–2026 evidence and practice converged on a more nuanced consensus (deep-dived in the dedicated [rag_vs_long_context_llms_guide.md](rag_vs_long_context_llms_guide.md)):

| Dimension | RAG | Long-context stuffing |
|---|---|---|
| **Cost** | Retrieves only what's needed — cheap per query | Pays for every token of every document, every query — O(corpus) per query |
| **Latency** | Sub-second to seconds | Linear in corpus size; 1M tokens is minutes of prefill |
| **Accuracy** | Top-k is a lossy filter (recall ceiling) | *Lost in the Middle* + distraction: accuracy drops with context size |
| **Freshness** | Swap corpus / index — instant | Retrain or re-prompt — the model's weights are frozen |
| **Auditability** | Citations point to retrievable chunks | Citations still needed but "the whole corpus" is not a citation |
| **When it wins** | Large corpora, frequent updates, strict audit | Small corpora (a few documents), one-shot analysis, exact long-form tasks |

The practical synthesis, which the [beyond_rag_guide.md](beyond_rag_guide.md) calls **long-context-native RAG**: use the big window as a *second* stage (retrieve a wide candidate set, then let the long-context model read it fully), or stuff small corpora and retrieve large ones. **Retrieval did not die; it moved up the stack.**

### 8.4 The RAG vs fine-tuning debate

The other perennial debate: should the knowledge go into the *prompt* (RAG) or into the *weights* (fine-tuning)? The field's answer by 2024–2025 was "both, for different jobs" — the full comparison lives in [fine_tuning_frameworks_comparison_guide.md](../fine_tuning_frameworks_comparison_guide.md); the timeline-relevant summary:

- **RAG wins** on factual freshness, citation/audit requirements, per-user knowledge isolation, and rapid iteration — everything a bank needs for regulatory-grade answers.
- **Fine-tuning wins** on behaviour: style, tone, output format, tool-use competence, domain *reasoning* patterns — the things that are *how* the model behaves rather than *what* it knows.
- **The synthesis (2024–2026):** fine-tune for behaviour, retrieve for knowledge — "RAG-then-distill" for repeated queries. The debate largely dissolved into a division of labour.

### 8.5 The evaluation evolution

Evaluation grew with the paradigm — the full playbook is [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md), the tooling is [rag_evaluation_tools_comparison_guide.md](rag_evaluation_tools_comparison_guide.md), [ragas_guide.md](ragas_guide.md), [trulens_guide.md](trulens_guide.md), and [llm_evaluation_frameworks_guide.md](../llm_evaluation_frameworks_guide.md). The timeline of *how* RAG is evaluated:

| Era | Dominant evaluation | Canonical artefacts |
|---|---|---|
| Naive RAG (2020–2022) | Retrieval metrics + benchmark QA | recall@k, MRR@k, NDCG@k; Natural Questions, TriviaQA |
| Advanced RAG (2023) | Generation metrics; LLM-as-judge emerges | faithfulness, answer relevance; GPT-4-as-judge evals |
| Production RAG (2023–2024) | The **RAG triad** (retrieval / generation / end-to-end); tooling matures | TruLens triad (context relevance → groundedness → answer relevance); Ragas metric families |
| Modular/Agentic RAG (2024–2026) | Pipeline-level and agent-level evals | per-step retrieval evals + end-to-end task success; agent trace evaluation; evals-as-code, golden sets, regression suites |

The constant across every era: **evaluation is the thing that makes RAG engineering scientific** — each paradigm shift was validated by better evals, and each eval advance exposed the next bottleneck (e.g., LLM-as-judge exposed hallucination-in-eval; agentic evals exposed loop inefficiency).

### 8.6 The debates, summarized

Three debates — **retrieval vs long context, RAG vs fine-tuning, and (the meta one) is RAG dead?** — recurred every year from 2023 onward, and every year the answer was the same: the *hybrid* wins. Retrieval necessity was never disproven (cost, freshness, audit); long-context never replaced retrieval (attention quality, economics); fine-tuning never replaced retrieval (knowledge is data, not weights). The "RAG is dead" headlines of 2025 were, in every case, arguments for a *better* RAG — agentic, graph-based, or long-context-native.

### 8.7 The debates, quantified — what the evidence actually shows

The recurring "X kills RAG" claims each had a falsifiable core, and the evidence trended the same way each time:

- **"Long context kills RAG" (2024)** — tested directly in *Lost in the Middle* and the subsequent effective-context literature: answer quality degrades as stuffing grows, cost scales linearly with corpus size, and per-query latency grows with prefill; retrieval keeps quality flat while keeping cost flat. On open benchmarks, the hybrid (retrieve a wide candidate set, then let the long-context model read it fully) outperformed both pure strategies. The deep evidence base lives in [rag_vs_long_context_llms_guide.md](rag_vs_long_context_llms_guide.md).
- **"Fine-tuning kills RAG" (2023–2025)** — parameter-efficient fine-tuning (LoRA et al.) made weight updates cheap, but FT cannot add knowledge absent from its training data, cannot cite a source, and cannot update without retraining; every head-to-head on knowledge-grounded tasks (regulatory QA, entity-heavy banking data) favoured retrieval on factual accuracy and FT on style/format/behaviour. See [fine_tuning_frameworks_comparison_guide.md](../fine_tuning_frameworks_comparison_guide.md).
- **"Agents kill RAG" (2025)** — agents *are* the latest RAG: the retrieval core is inside every research agent (Deep Research, agentic copilots). The debate dissolved into "what wraps the retriever", not "whether there is a retriever".

The pattern across all three challengers is identical: **the challenger never removed the retrieval core — it moved the retrieval *call* to a different layer of the stack** (into the prompt, into the weights, or into the agent loop). That single sentence explains why RAG, alone among 2023's architectural fads, is still the substrate of enterprise AI in 2026.

---

## 9. The 2026+ State — RAG 2.0 and RAG as a Feature

### 9.1 "RAG 2.0" — an informal term, honestly flagged

**Verification flag:** "RAG 2.0" has **no canonical paper** as of August 2026. It circulates as an industry shorthand (vendor blogs, conference talks, LinkedIn discourse) for the post-2024 RAG: self-reflective, self-optimizing, learning, and agentic — the direct lineage of Self-RAG → CRAG → Adaptive-RAG. When people say "RAG 2.0" they usually mean one or more of:

1. **Self-reflective RAG** — the system checks its own retrieval and generation (reflection tokens, evaluators, corrective loops);
2. **Self-optimizing / learning RAG** — DSPy-style programmatic optimization of the prompts/retrieval pipeline, plus feedback loops from production traffic;
3. **Agentic RAG** — retrieval as a tool inside an agent loop with routing, multi-step search, and web fallback;
4. **Structured RAG** — GraphRAG-style knowledge graphs on top of vectors.

Treat the term as a *cluster of capabilities*, not a dated event. The capability cluster is real; the brand is not a paper.

### 9.2 RAG as a feature — embedded in platforms

The 2025–2026 structural change: **RAG stopped being a system you build and became a feature you inherit.** Enterprise AI platforms — and the agent platforms in [enterprise_ai_platforms_guide.md](../enterprise_ai_platforms_guide.md) — ship retrieval, vector storage, reranking, and citation as *platform defaults*: you connect a data source and the platform indexes it, retrieves for the agent, and formats citations (often via MCP tool bindings — see [mcp_framework_tools_guide.md](../mcp_framework_tools_guide.md)). The consequence for architects: the RAG *differentiation* moved from "wiring retrieval" to "curating corpora, designing evaluation, and controlling the loop" — exactly the skills the rest of this series covers.

### 9.3 The trends (2025–2026, flagged)

- **Agentic RAG is mainstream.** The 2024 research trio is now the default production pattern; "agentic RAG" is standard framework vocabulary (LangGraph, LlamaIndex workflows, Haystack agents).
- **GraphRAG in production.** Microsoft open-sourced GraphRAG (July 2024) and shipped GraphRAG 1.0 (November 2024); graph-enhanced RAG is a standard option for entity- and relationship-heavy corpora (a bank's counterparty, product, and regulatory data). Cost-conscious variants (LightRAG, HippoRAG) are adopted where full GraphRAG indexing is too expensive.
- **Long-context-native RAG.** The hybrid from §8.3: wide retrieval + full-context reading; small-corpus stuffing; "retrieval as a second stage".
- **RAG + reasoning models (o1-style).** Reasoning models that *think* before answering have made retrieval-in-the-chain-of-thought a pattern: the model decides mid-reasoning to search, read, and continue (the mechanism behind Deep Research-style products). Flag: this is a 2025–2026 trend with strong adoption signals but no settled taxonomy — "test-time compute + retrieval" is still being named.
- **RAG + token economics.** As frontier token prices and context lengths rise, the *cost* of stuffing vs retrieving is being re-optimized per query (light cross-ref: [chinese_tokenization_guide.md](../chinese_tokenization_guide.md) for the tokenization angle).

### 9.4 RAG in the enterprise — the banking lens

For a bank, the timeline's endpoint is a *governed* retrieval architecture (deep dive: [implementing-responsible-ai.md](../implementing-responsible-ai.md)):

- **Where RAG lives in the bank (2026):** regulatory/policy Q&A, credit-memo grounding, KYC and AML research assistants, client-onboarding knowledge, trade/rates desk playbooks, and internal-knowledge copilots. All of these are knowledge-intensive tasks — the same task family that motivated the 2020 paper.
- **Why RAG won the governance argument:** citations are auditable; corpus updates are controlled; user-level knowledge isolation is possible; and the *non-parametric memory* is inspectable — regulators can ask "what did the model read?" and get chunks, not weights.
- **What the timeline teaches the bank:** naive RAG fails under production scrutiny (hallucination, stale corpus, no routing) — the *evaluated* advanced/modular/agentic stack is the production minimum; compliance guardrails (SR 11-7, MAS TRM, PII handling) must be part of the retrieval design from day one, not bolted on; and "RAG 2.0" capabilities (self-reflection, corrective retrieval) are precisely what the audit trail needs — a record of *why retrieval was accepted or rejected*.
- **The trajectory:** the bank's 2026 RAG is a *platform feature* (retrieval as infrastructure), differentiated by corpus curation, evaluation suites, and human-in-the-loop controls — see [enterprise_ai_platforms_guide.md](../enterprise_ai_platforms_guide.md) for the platform layer.

**The banking timeline at a glance** — how the bank's own RAG journey mirrors the field's:

| Bank stage | When | What it was | Which era's failure modes it inherited |
|---|---|---|---|
| Pilot | 2023 | "Ask the policy manual" on naive RAG | Hallucination, retrieval misses (naive-era) |
| Hardening | 2023–2024 | Chunking, hybrid search, reranking, eval suites | Quality plateau (advanced-era fixes) |
| Scale-out | 2024–2025 | Multi-corpus platforms, GraphRAG for entity-heavy data | Cost and flexibility (modular-era concerns) |
| Agentic + governed | 2025–2026 | Agentic copilots with reflection, routing, human-in-the-loop | Autonomy and audit (agentic-era concerns) |

### 9.5 The future

The forward-looking map — what comes *after* classical RAG, including when to leave retrieve-then-generate behind — is the job of [beyond_rag_guide.md](beyond_rag_guide.md) (nine paradigms: test-time compute, long-context-native, memory systems, agentic retrieval, retrieval-as-training, context engineering, neuro-symbolic, parametric-only, self-improving). The timeline's one-sentence handoff: **RAG's future is not the pipeline — it is the retrieval *capability* dissolved into the agent, the graph, the platform, and the training loop.**

---

## 10. The Master Timeline and the Lessons

### 10.0 The era cheat-sheet

Before the full table, the era summary — every RAG conversation reduces to this:

| Era | Years | Canonical papers (arXiv) | Signature technique | Era-defining limitation it answered |
|---|---|---|---|---|
| Pre-RAG | before 2020 | NTM (1410.5401), Memory Networks (1410.3916), REALM (2002.08909), RETRO (2112.04426) | Retrieval + pretraining | Tiny context windows; weights-only knowledge |
| Naive RAG | 2020–2023 | Lewis et al. (2005.11401) | Index → retrieve → generate | Knowledge grounding, period |
| Advanced RAG | 2023 | HyDE (2212.10496), RAG-Fusion (2402.03367) | Pre/post-retrieval optimization | Retrieval quality, hallucination |
| Modular RAG | 2024 | Modular RAG survey (2407.21059), GraphRAG (2404.16130) | Reconfigurable modules, knowledge graphs | Rigidity; global/relational questions |
| Agentic RAG | 2023–2026 | Self-RAG (2310.11511), CRAG (2401.15884), Adaptive-RAG (2403.14403) | Self-reflection, correction, routing | No autonomy; no recovery from failure |
| Embedded RAG | 2025–2026+ | (industry, informal) | RAG as platform feature, "RAG 2.0" | Deployment cost; differentiation |

### 10.1 The master timeline (2014–2026)

| Year | Milestone | Significance |
|---|---|---|
| 2014 | Neural Turing Machines (Graves et al.) | Neural models with external memory — the "memory" line begins |
| 2014–15 | Memory Networks (Weston, Chopra, Bordes) | Attention-addressed memory for QA; the vocabulary RAG inherits |
| 2015 | End-to-End Memory Networks (Sukhbaatar et al.) | Trainable memory end-to-end |
| 1994/1960s | BM25 / tf-idf | The lexical retrieval foundation that survives in hybrid RAG |
| 2018–19 | BERT/GPT-1/GPT-2 (512–1,024 token windows) | The context ceiling that makes retrieval necessary |
| 2020 | **REALM** (Guu et al., arXiv:2002.08909) | Differentiable retrieval inside pretraining |
| 2020 | **RAG is born** (Lewis et al., arXiv:2005.11401) | The term is coined; parametric + non-parametric memory; RAG-Sequence/RAG-Token |
| 2020 | kNN-LM (Khandelwal et al.) | Retrieval during *generation* at the token level |
| 2021 | **RETRO** (Borgeaud et al., arXiv:2112.04426) | 7.5B ≈ GPT-3 175B via a 2T-token database; retrieval ≈ scale |
| 2022 | **HyDE** (Gao et al., arXiv:2212.10496) | Hypothetical-document embeddings; query-side fix for vocabulary mismatch |
| 2022–23 | ChatGPT (Nov 2022) → enterprise "chat with your data" | Naive RAG becomes an industry default; LangChain/LlamaIndex productize it |
| 2023 | OpenAI `text-embedding-ada-002`; BGE family | Dense embeddings become commodity infrastructure |
| 2023 | **RAG-Fusion** (multi-query + RRF) | Popularized; formal paper arXiv:2402.03367 (2024) |
| 2023 | Cohere Rerank / cross-encoder rerankers | Post-retrieval precision jump; advanced RAG's signature move |
| 2023 | **Lost in the Middle** (Liu et al., arXiv:2307.03172) | Position bias quantified; context placement becomes a design lever |
| 2023 | **Self-RAG** (Asai et al., arXiv:2310.11511) | The model decides *whether* to retrieve; reflection tokens |
| 2023 | **RAG survey** (Gao et al., arXiv:2312.10997) | The naive/advanced/modular taxonomy; "RAG" as a discipline |
| 2024 | **CRAG** (Yan et al., arXiv:2401.15884) | Corrective retrieval: evaluate, decompose, fall back to web |
| 2024 | **Gemini 1.5** (Feb 15) — 1M-token context | The context-window inflection point; "do we still need RAG?" |
| 2024 | **Adaptive-RAG** (Jeong et al., arXiv:2403.14403) | Query-complexity routing: no retrieval / single / multi-step |
| 2024 | **GraphRAG** (Edge et al., arXiv:2404.16130) | Knowledge graphs + community detection (Leiden); global questions |
| 2024 | **Modular RAG** (Gao et al., arXiv:2407.21059) | RAG as LEGO-like reconfigurable modules; pipeline → design space |
| 2024 | Anthropic contextual retrieval (Sep) | Retrieval-side quality still moves the needle |
| 2025 | OpenAI Deep Research (Feb); agentic RAG mainstream | RAG as a *tool* inside multi-step research agents |
| 2025–26 | "RAG 2.0" discourse; 1M-token models standard; RAG as platform feature | Self-reflective, self-optimizing, embedded RAG (informal term — flagged) |

### 10.2 The paradigm shifts — why they happened

| Shift | Triggering limitation | What the shift added |
|---|---|---|
| **Naive → Advanced** (2023) | Retrieval quality and hallucination: wrong chunks, unfaithful answers | Pre-retrieval (chunking, query rewriting) and post-retrieval (reranking) optimization — *curation* |
| **Advanced → Modular** (2024) | Rigidity: the fixed pipeline couldn't express routing, memory, fusion, or graphs | Decompose into reconfigurable modules/operators — *flexibility* |
| **Modular → Agentic** (2024–2026) | No autonomy: the system couldn't decide, correct, or loop | Self-reflection (Self-RAG), correction (CRAG), routing (Adaptive-RAG) — *autonomy* |

The pattern is consistent: **each era is a response to the previous era's bottleneck** — first quality, then flexibility, then autonomy. The naive baseline never disappeared; it became the leaf-node execution step inside increasingly intelligent orchestrators.

### 10.3 The constants — what never changed

- **The retrieval core.** Every era still does *index → retrieve → generate*. The pipeline became modules, the modules became agent skills, but the retrieval core — a corpus, a ranking function, a top-k — is invariant. BM25, 1994, is still in production at banks in 2026, inside hybrid retrievers.
- **The knowledge-vs-weights split.** Parametric + non-parametric memory (2020's framing) remains the fundamental architecture of grounded AI; the debates in §8 all reduce to where to put knowledge.
- **Evaluation as the scientific substrate.** Every paradigm shift was validated — and every new bottleneck exposed — by better evaluation (see [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md) and [rag_evaluation_tools_comparison_guide.md](rag_evaluation_tools_comparison_guide.md)).
- **The context-window pressure.** The constraint that created RAG never vanished — it *moved*: from "the model can't see the document" to "the model can see everything, but can't attend to all of it well or cheaply". Retrieval is the answer to both versions of the problem.

### 10.4 The lessons

1. **RAG is not one thing — it is a spectrum.** "RAG" spans a 2020 latent-variable paper, a 2023 five-line LangChain script, a 2024 GraphRAG knowledge-graph pipeline, and a 2026 agent loop. When architects argue "RAG vs X", they are usually arguing about one point on the spectrum; the [advanced_rag_techniques_guide.md](advanced_rag_techniques_guide.md) decision tree exists precisely because the right point depends on the task.
2. **The evolution is a living architecture, not a succession of replacements.** Naive RAG did not die when advanced RAG arrived; it became the fallback path. Advanced techniques did not die when agents arrived; they became the tools agents call. Adopting "agentic RAG" without the chunking/reranking/eval discipline underneath is just expensive naive RAG with extra steps.
3. **The retrieval bottleneck is the permanent enemy.** Every paradigm shift re-engineered *where* the lossy filter sits, never removed it. The winning systems in every era are the ones that make retrieval failures visible and recoverable (CRAG's evaluator, Self-RAG's reflection tokens, the eval triad).
4. **For the enterprise, the timeline compresses.** A bank in 2026 can — and should — adopt the *endpoint* of the timeline (agentic, modular, evaluated RAG as a platform feature) without reliving 2020–2023. The timeline's value is knowing which era's failure modes you are inheriting by skipping ahead, and pricing the mitigations accordingly.

### 10.5 The bank's reading of the timeline

For a Solution Architect at a global bank, the timeline compresses into five practical conclusions:

1. **Skip the archaeology, keep the mitigations.** Adopt the 2026 endpoint (agentic, modular, evaluated RAG as a platform feature) — but the failure modes of every skipped era still apply, so price them: naive-era hallucination (→ grounding evals), advanced-era retrieval misses (→ reranking + hybrid retrieval), modular-era flexibility (→ deliberate framework choice), agentic-era autonomy (→ guardrails and audit).
2. **The audit trail is the product.** CRAG's "why did we accept this retrieval?" evaluator and Self-RAG's reflection tokens are not just accuracy tricks — they are the machine-readable record a regulator asks for. Non-parametric memory is the bank's structural advantage: *what the model read is inspectable*.
3. **Corpus curation is the new differentiation.** As RAG becomes a platform feature, the differentiator is the corpus (entity-linked, versioned, permissioned) and the evaluation suite — the discipline of §8.5, applied continuously, in production.
4. **The constants are the cost model.** Retrieval economics (per-query cost vs corpus stuffing) decided every debate in §8 and will decide the 2026–2028 choices too; token economics (see [chinese_tokenization_guide.md](../chinese_tokenization_guide.md)) matter acutely for multilingual and Chinese-language corpora.
5. **The timeline keeps moving.** The 2020 paper solved 2020's constraint; the 2026 stack solves 2026's. The architect's job is not to pick the final architecture — there isn't one — but to keep the retrieval core, the evaluation loop, and the governance layer stable while the surrounding stack evolves.

---

## 11. Summary — RAG Evolution in One Page

**The arc.** RAG was born in 2020 (Lewis et al., arXiv:2005.11401) as the fusion of two older lines — 50 years of lexical IR and a decade of neural memory research — motivated by the hardest constraint of the early LLM era: tiny context windows and weights-only knowledge. It spent 2020–2022 as a research technique for knowledge-intensive tasks, then 2023 turned it into an industry under ChatGPT's gravity: the *naive* retrieve-then-generate pipeline became the "chat with your data" default, and its predictable failures — retrieval quality, hallucination, context integration — drove the *advanced* era of pre/post-retrieval optimization (chunking, HyDE, RAG-Fusion, reranking). In 2024 the *modular* era decomposed the pipeline into reconfigurable LEGO-like modules (arXiv:2407.21059) and grafted structure onto vectors with GraphRAG (arXiv:2404.16130). From late 2023 through 2026 the *agentic* era gave retrieval autonomy — Self-RAG's reflection tokens (arXiv:2310.11511), CRAG's corrective loops (arXiv:2401.15884), Adaptive-RAG's complexity routing (arXiv:2403.14403) — and RAG became a skill inside agents, a default feature of platforms, and (informally) "RAG 2.0". The challenges that shaped each era — the 1M-token context windows (Gemini 1.5, Feb 2024), *Lost in the Middle* (arXiv:2307.03172), RAG vs long-context, RAG vs fine-tuning, and the maturing evaluation discipline — never killed retrieval; they repositioned it as the *cost-efficient, auditable, fresh* knowledge layer underneath everything else.

**The eras in one line each:**

- **Pre-RAG (before 2020):** retrieval was old, neural memory was small, and tiny context windows made external knowledge inevitable.
- **Naive (2020–2023):** the pipeline that grounded LLMs — and the baseline every better system is measured against.
- **Advanced (2023):** curation — rewrite the query, chunk the corpus, rerank the results.
- **Modular (2024):** flexibility — RAG became LEGO, and knowledge graphs gave it structure.
- **Agentic (2023–2026):** autonomy — the model decides to retrieve, checks the evidence, corrects, and loops.
- **Embedded (2025–2026+):** ubiquity — retrieval became a platform default and a governance advantage.

**The final word.** RAG's history is the history of a good idea being forced to grow up: the naive pipeline that made LLMs usable in the enterprise became the curated pipeline, then the modular system, then the agent skill, then the platform feature — and at every step the retrieval core stayed. The lesson for the architect — and for the bank — is that the timeline is not a museum: **each era's techniques are still live options, and the current best practice is the endpoint of the arc, chosen deliberately per use case, evaluated relentlessly, and governed from day one.**

---

## 12. Glossary

| Term | Definition |
|---|---|
| **RAG (Retrieval-Augmented Generation)** | A framework that retrieves relevant passages from an external corpus and conditions an LLM's generation on them; coined by Lewis et al. (2020). |
| **Parametric memory** | Knowledge stored in the model's weights from pretraining; frozen, opaque, expensive to update. |
| **Non-parametric memory** | Knowledge stored externally — a vector index/corpus; updatable, inspectable, citable. |
| **RAG-Sequence** | RAG formulation that retrieves once and marginalizes the whole output sequence over the top-k documents. |
| **RAG-Token** | RAG formulation that marginalizes each generated token over the top-k documents. |
| **Naive RAG** | The canonical fixed pipeline: chunk → embed → retrieve top-k → stuff prompt → generate; no feedback or routing. |
| **Advanced RAG** | Naive RAG plus pre-retrieval (chunking, query rewriting) and post-retrieval (reranking, compression) optimization. |
| **Modular RAG** | RAG decomposed into reconfigurable modules/operators (search, memory, routing, fusion, predict); the LEGO paradigm (2024). |
| **Agentic RAG** | RAG where an agent decides when/how to retrieve, evaluates evidence, corrects, and iterates; informal umbrella term. |
| **GraphRAG** | Microsoft's graph-based RAG (2024): entity extraction → knowledge graph → community detection → global summarization. |
| **Knowledge graph** | A graph of entities and relations extracted from a corpus; the structured alternative to flat vector chunks. |
| **Entity linking** | Resolving mentions in text to canonical entities in a knowledge base (GraphRAG's extraction step). |
| **Community detection** | Partitioning a graph into clusters of related nodes (GraphRAG uses the **Leiden** algorithm). |
| **Leiden** | A fast, high-quality graph community-detection algorithm used by GraphRAG for hierarchical partitioning. |
| **REALM** | Retrieval-Augmented Language Model Pre-Training (2020): differentiable retrieval inside masked-LM pretraining. |
| **RETRO** | DeepMind's RETrieval-TransfORmer (2021): 7.5B params ≈ GPT-3 175B using a 2T-token retrieval database. |
| **Memory Networks** | Weston, Chopra & Bordes (2014/15): neural networks with attention-addressed external memory for QA. |
| **Neural Turing Machines** | Graves et al. (2014): RNNs with differentiable external memory and read/write heads. |
| **BM25** | Okapi BM25 (1994): the probabilistic lexical ranking function; the sparse baseline of hybrid retrieval. |
| **tf-idf** | Term frequency–inverse document frequency: the classic lexical relevance score (vector space model, 1960s–70s). |
| **Dense retrieval** | Retrieval by embedding similarity (neural vectors) rather than lexical match. |
| **Sparse retrieval** | Lexical retrieval over term matching (BM25/tf-idf), typically with an inverted index. |
| **Embedding** | A dense vector representation of text; similarity between embeddings approximates semantic similarity. |
| **Chunking** | Splitting a corpus into indexable units; naive = fixed-size with overlap, advanced = semantic/small-to-big. |
| **Semantic chunking** | Splitting at meaning boundaries (sentences, topic shifts) so chunks are self-contained. |
| **Small-to-big** | Retrieve small precise chunks; feed their parent/larger context to the generator. |
| **Query rewriting** | Transforming the user query before retrieval (expansion, decomposition, paraphrasing) to improve recall. |
| **HyDE** | Hypothetical Document Embeddings (2022): embed an LLM-generated hypothetical answer instead of the query. |
| **RAG-Fusion** | Multi-query retrieval merged by Reciprocal Rank Fusion; popularized 2023. |
| **Reciprocal Rank Fusion (RRF)** | A rank-merging method (Cormack et al. 2009): 1/(k+rank) scores summed across ranked lists. |
| **Reranking** | A second, more precise scoring pass over a wide retrieved set before generation. |
| **Cross-encoder** | A model that scores query+passage jointly (full attention) — the standard reranker architecture. |
| **Self-RAG** | Self-reflective RAG (2023): models emit reflection tokens to decide retrieval and critique groundedness. |
| **CRAG** | Corrective RAG (2024): a retrieval evaluator triggers keep/decompose/web-fallback actions. |
| **Adaptive-RAG** | Routing by question complexity (2024): no retrieval / single-step / multi-step. |
| **Long-context** | Models with very large context windows (128k–1M+ tokens); the 2024+ alternative/complement to RAG. |
| **Context window** | The maximum number of tokens a model can attend to per call. |
| **Lost in the Middle** | The 2023 finding that LLMs use the middle of long contexts worse than the edges (position bias). |
| **Gemini 1.5** | Google's model family (Feb 2024) that introduced 1M-token contexts — the long-context inflection point. |
| **RAG 2.0** | Informal 2025–26 term for self-reflective/self-optimizing/agentic RAG; no canonical paper (flagged). |
| **Fine-tuning** | Updating model weights on task data; the "knowledge into weights" alternative/complement to RAG. |
| **Evaluation** | Measuring retrieval quality, generation faithfulness, and end-to-end correctness (the RAG triad). |
| **Survey** | A synthesis paper that maps a field; the RAG surveys (2023–2024) defined the naive/advanced/modular taxonomy. |

---

## 13. References and Verification Notes

**Verified primary sources (paper, arXiv ID, date — confirmed via web check, August 2026):**

- Lewis et al., *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*, **arXiv:2005.11401**, May 2020 (NeurIPS 2020) — the origin of the term "RAG".
- Guu et al., *REALM: Retrieval-Augmented Language Model Pre-Training*, **arXiv:2002.08909**, Feb 2020 (ICML 2020).
- Borgeaud et al., *Improving Language Models by Retrieving from Trillions of Tokens* (RETRO), **arXiv:2112.04426**, Dec 2021.
- Weston, Chopra & Bordes, *Memory Networks*, **arXiv:1410.3916**, Oct 2014 (ICLR 2015); Graves, Wayne & Danihelka, *Neural Turing Machines*, **arXiv:1410.5401**, 2014.
- Gao et al., *Precise Zero-Shot Dense Retrieval without Relevance Labels* (HyDE), **arXiv:2212.10496**, Dec 2022 (ACL 2023).
- Liu et al., *Lost in the Middle: How Language Models Use Long Contexts*, **arXiv:2307.03172**, Jul 2023.
- Asai et al., *Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection*, **arXiv:2310.11511**, Oct 2023 (ICLR 2024).
- Gao et al., *Retrieval-Augmented Generation for Large Language Models: A Survey*, **arXiv:2312.10997**, Dec 2023 — source of the naive/advanced/modular taxonomy.
- Yan et al., *Corrective Retrieval Augmented Generation* (CRAG), **arXiv:2401.15884**, Jan 2024.
- Jeong et al., *Adaptive-RAG: Learning to Adapt Retrieval-Augmented LLMs through Question Complexity*, **arXiv:2403.14403**, Mar 2024 (NAACL 2024).
- Edge et al., *From Local to Global: A Graph RAG Approach to Query-Focused Summarization*, **arXiv:2404.16130**, Apr 2024.
- Gao, Xiong, Wang & Wang, *Modular RAG: Transforming RAG Systems into LEGO-like Reconfigurable Frameworks*, **arXiv:2407.21059**, Jul 2024.
- Gao et al., *Searching for Best Practices in Retrieval-Augmented Generation*, **arXiv:2407.01219**, Jul 2024.
- Cormack, Clarke & Büttcher, *Reciprocal Rank Fusion Outperforms Condorcet and Individual Rank Learning Methods*, SIGIR 2009.
- RAG-Fusion lineage: multi-query + RRF popularized mid-2023 (Zachary Liu essay; Raudaschl open-source implementation); formal write-up *RAG-Fusion: a New Take on Retrieval-Augmented Generation*, **arXiv:2402.03367**, Feb 2024.
- Gemini 1.5 Pro: announced 15 Feb 2024, 1M-token context (128k standard, 1M experimental preview); 2M variant May 2024.
- Context-window milestones (GPT-3 2k 2020 → Claude 100k May 2023 → GPT-4 Turbo 128k Nov 2023 → Gemini 1.5 1M Feb 2024 → Claude 3 200k Mar 2024 → 1M-class 2025 models): assembled from vendor announcements; dates are launch dates of public availability.

**Flagged as informal / unverifiable as canonical:**

- **"RAG 2.0"** — industry shorthand only; no canonical paper (see §9.1).
- **"Agentic RAG"** — ubiquitous community term; the canonical artefacts are the Self-RAG/CRAG/Adaptive-RAG papers, not a single "agentic RAG" paper (see §7.1).
- **"RAG + reasoning models (o1-style)"** — a live 2025–2026 trend without a settled taxonomy; described as a trend, not a dated event.
- **Early-LLM context windows** (BERT/GPT-1/GPT-2/GPT-3): well-established figures from model cards and papers.
- **BGE / text-embedding-ada-002 / Cohere Rerank dates**: product-launch dates from vendor documentation; month-level accuracy.
