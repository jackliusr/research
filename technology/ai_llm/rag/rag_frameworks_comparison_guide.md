# RAG Frameworks: Comprehensive Comparison Guide

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Crédit Agricole CIB
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** LLM/AI Engineering Guides
> **Companion Guides:** [LLM Application Frameworks](../llm_frameworks_comparison_guide.md) · [RAG vs HyDE](rag_vs_hyde_guide.md) · [Query Rewriting in RAG](query_rewriting_rag_guide.md) · [LLM Evaluation Frameworks](../llm_evaluation_frameworks_guide.md) · [Enterprise AI Gateway](../../enterprise_ai_gateway_guide.md) · [MCP Frameworks & Tools](../mcp_framework_tools_guide.md)
> **Last Updated:** July 2026

---

## Table of Contents

1. [Introduction: What RAG Frameworks Are](#1-introduction-what-rag-frameworks-are)
2. [Anatomy of a RAG System — Where the Frameworks Fit](#2-anatomy-of-a-rag-system--where-the-frameworks-fit)
3. [The RAG Tooling Landscape: Five Categories](#3-the-rag-tooling-landscape-five-categories)
4. [LangChain — The Most Popular RAG Framework](#4-langchain--the-most-popular-rag-framework)
5. [LlamaIndex — The RAG-First Data Framework](#5-llamaindex--the-rag-first-data-framework)
6. [Haystack — Production Pipelines by deepset](#6-haystack--production-pipelines-by-deepset)
7. [Orchestration Frameworks: Head-to-Head](#7-orchestration-frameworks-head-to-head)
8. [Vector Databases with RAG Features](#8-vector-databases-with-rag-features)
9. [Retrieval Libraries and Reranking](#9-retrieval-libraries-and-reranking)
10. [Full RAG Platforms](#10-full-rag-platforms)
11. [RAG Evaluation Frameworks](#11-rag-evaluation-frameworks)
12. [Managed RAG Services](#12-managed-rag-services)
13. [Selection Guide: Decision Factors and Scenarios](#13-selection-guide-decision-factors-and-scenarios)
14. [Recommended Stack for Banking](#14-recommended-stack-for-banking)
15. [Banking Considerations Deep Dive](#15-banking-considerations-deep-dive)
16. [Conclusion and Decision Framework](#16-conclusion-and-decision-framework)

---

## 1. Introduction: What RAG Frameworks Are

**Retrieval-Augmented Generation (RAG)** grounds an LLM's answer in an external corpus: the system retrieves the most relevant passages for a query, stuffs them into the prompt as context, and conditions generation on them. The architecture sounds simple — embed, search, prompt, generate — but a production RAG system hides a surprising amount of plumbing: document parsing and chunking, embedding model management, vector store administration, retrieval and reranking logic, prompt construction, generation orchestration, memory, and evaluation.

**RAG frameworks are software libraries and platforms that provide these building blocks as reusable, composable components.** They abstract away the plumbing so developers can focus on application logic rather than on re-implementing chunking strategies, retriever wiring, or prompt templates from scratch.

**What a RAG framework typically provides:**

- **Document loading and chunking** — connectors for PDF, HTML, DOCX, Markdown, databases, and web sources, plus splitters (fixed-size, recursive character, sentence-aware, semantic, markdown-aware).
- **Embedding management** — unified interfaces over OpenAI, Hugging Face, Cohere, Ollama, Voyage, and self-hosted embedding models, with caching and batch strategies.
- **Vector store integrations** — adapters for FAISS, Chroma, pgvector, Qdrant, Milvus, Pinecone, Weaviate, Elasticsearch, and dozens more, so the same application code can switch backends.
- **Retrieval logic** — vector search, BM25/sparse, hybrid, ensemble, multi-query, self-query, contextual compression, parent-document, and reranking hooks.
- **Prompt construction** — templates, context formatting, and dynamic prompt assembly per query.
- **Generation orchestration** — chains, pipelines, or workflows that sequence retrieval → augmentation → generation, including agentic and multi-step flows.
- **Evaluation** — metrics for retrieval quality (context precision/recall) and generation quality (faithfulness, answer relevance), plus observability hooks.

**Why this matters architecturally:** the framework choice largely *is* the application skeleton. Retrievers, stores, and prompts can be swapped later; the orchestration model (chains vs. pipelines vs. graphs) shapes how the team thinks about, tests, and operates the system. For a bank, this decision also has governance implications — the framework must be documented, versioned, and validated under model-risk processes (see §15).

### 1.1 The RAG maturity curve

The RAG space has evolved in three waves, and the frameworks reflect it:

| Wave | Era | Characteristic | Representative tooling |
|------|-----|----------------|------------------------|
| **Naive RAG** | 2023 | Fixed pipeline: chunk → embed → top-k → prompt. Simple, but brittle to chunking and query phrasing. | Early LangChain chains, LlamaIndex index APIs |
| **Advanced RAG** | 2023–2024 | Pre-retrieval (query rewriting, HyDE), post-retrieval (reranking, compression), hybrid search, chunk optimization (parent-child, auto-merging). | MultiQueryRetriever, QueryFusionRetriever, RRF ensembles, FlashRank/Cohere Rerank |
| **Modular / Agentic RAG** | 2024–2026 | Graph-based orchestration, routing, multi-step retrieval, tool use, reflection; GraphRAG for entity-heavy corpora. | LangGraph, LlamaIndex Workflows, Haystack 2.x pipelines, RAGFlow GraphRAG |

Modern evaluation — RAGAS, TruLens, DeepEval (§11) — grew out of the Advanced RAG wave and is now a non-negotiable part of any production stack.

### 1.2 What a framework actually saves you

A fair question: "why not just write it in Python directly?" — you can, and for a one-off demo you should. Frameworks pay off when a system has many moving parts and will evolve:

| Task | Without a framework | With a framework |
|------|--------------------|-------------------|
| Load + chunk 10 PDFs | Hand-rolled PDF library loop + regex splitter | `loader.load()` + `splitter.split_documents()` |
| Swap OpenAI → self-hosted embeddings | Rewrite all embedding calls | Change one configuration line |
| Switch FAISS → Qdrant | Rewrite insert/search code | Swap the vector-store adapter |
| Hybrid retrieval (BM25 + dense) | Write two retrievers + RRF fusion yourself | One component (Haystack `HybridRetriever`, `EnsembleRetriever`) |
| Add reranking | Write cross-encoder glue code | Add a ranker component |
| Trace a bad answer | Print-debug each step | LangSmith trace / pipeline inspector |
| Evaluate a new chunking config | Hand-built eval script | RAGAS integration + dataset |

Frameworks also standardize the *vocabulary* of the team: "retriever", "node", "pipeline step", and "query engine" mean the same thing across projects. For a bank, that shared vocabulary flows directly into internal engineering standards and model documentation (§15.3).

---

## 2. Anatomy of a RAG System — Where the Frameworks Fit

A RAG system decomposes into stages, and each framework category owns a slice of them:

```
                  ┌─────────────────── INGESTION (offline) ───────────────────┐
                  │  load → parse → chunk → embed → index → (optional) KG     │
                  └───────────────────────────────────────────────────────────┘
                                             │
                  ┌─────────────────── QUERY TIME (online) ──────────────────┐
                  │  query → rewrite/HyDE → retrieve → rerank → compress →    │
                  │          prompt → generate → evaluate → log               │
                  └───────────────────────────────────────────────────────────┘
```

| Stage | Typical components | Who provides it |
|-------|--------------------|-----------------|
| Document loading | PDF/DOCX/HTML/web loaders, OCR, layout analysis | Orchestration frameworks (LangChain, LlamaIndex, Haystack), full platforms (RAGFlow), managed services |
| Chunking | Recursive/semantic/markdown splitters, template-based chunking | Orchestration frameworks, RAGFlow's deep-document chunking |
| Embedding | OpenAI/HF/Cohere/Ollama models, batch embedding | Frameworks (interface), sentence-transformers (models), vector DBs (some built-in) |
| Vector storage | FAISS, Chroma, pgvector, Qdrant, Milvus, Pinecone, Weaviate, ES | Vector databases (§8) |
| Retrieval | Dense, sparse (BM25), hybrid, fusion (RRF), multi-query, parent-doc | Frameworks' retriever libraries + vector DB native features |
| Reranking | Cross-encoders, Cohere Rerank, bge-reranker, FlashRank, ColBERT | Retrieval libraries (§9) |
| Prompt construction | Templates, context assembly, instruction formatting | Frameworks; prompt mgmt tools |
| Generation | LLM calls, streaming, citations, structured output | Frameworks + LLM providers |
| Orchestration | Chains, pipelines, workflows, agents, state | LangChain/LangGraph, LlamaIndex Workflows, Haystack pipelines |
| Evaluation | Faithfulness, relevance, context metrics, observability | RAGAS, TruLens, DeepEval, LangSmith (§11) |
| Serving/ops | APIs, deployment, tracing, guardrails | LangServe, deepset Cloud, managed services, AI gateways |

### 2.1 Chunking — the first quality lever

Chunking is the most underrated determinant of retrieval quality: if the chunk doesn't contain the answer, no retriever can find it. The major strategies:

| Strategy | How it works | Strengths | Weaknesses | Good for |
|----------|--------------|-----------|------------|----------|
| Fixed-size | N tokens per chunk with overlap | Trivial, predictable token counts | Splits mid-sentence/thought; loses structure | Quick baselines |
| Recursive character | Split on nested separators (`\n\n` → `\n` → space) | Respects natural boundaries; LangChain default | Still ignores semantics | General-purpose default |
| Sentence-aware | Split on sentence boundaries, group sentences | Clean units of meaning | Long sentences dominate chunks | Prose-heavy corpora |
| Semantic | Embed sentences, split where similarity drops | Boundaries follow topic shifts | Extra embedding cost; threshold tuning | Mixed-topic documents |
| Markdown/HTML-structure | Split on headers, preserve hierarchy | Keeps structure + metadata (titles, sections) | Only for structured formats | Regulatory text, docs, wikis |
| Template-based (RAGFlow) | Parse layout into headline/table/figure blocks | Table-safe chunking, figure capture | Platform-specific | PDF-heavy, table-dense filings |
| Parent-child | Retrieve small child chunks, return parent | Precision + context in one | Two-level index management | Long, dense documents |

**Banking note:** regulatory filings and prospectuses are table- and header-dense — precisely where naive fixed-size chunking fails and layout-aware parsing (RAGFlow, LlamaParse) pays for itself (§14).

### 2.2 Where retrieval quality actually comes from

Empirically, production retrieval quality is dominated by four levers, in rough order of impact:

1. **Hybrid retrieval** (dense + BM25 fused) — fixes exact-term misses; the single biggest recall win.
2. **Reranking** (cross-encoder over top-50) — fixes "relevant at position 12"; the biggest precision win.
3. **Chunking strategy** — layout-aware parsing for structured documents beats any retriever tuning.
4. **Query preprocessing** — rewriting, HyDE, and metadata filters for ambiguous queries.

Frameworks matter because they make levers 1–4 *composable*; no framework makes them unnecessary.

**Key architectural insight:** no single tool covers every stage well. A typical production stack *composes*: an orchestration framework for the pipeline, a vector database for storage, a reranker library for precision, an evaluation library for quality gates, and an observability platform for operations. The framework's job is to make those pieces replaceable.

---

## 3. The RAG Tooling Landscape: Five Categories

The RAG ecosystem is commonly organized into five categories. Teams usually pick one primary category and compose across the others:

| Category | Examples | Abstraction level | Primary job |
|----------|----------|-------------------|-------------|
| **End-to-end orchestration frameworks** | LangChain, LlamaIndex, Haystack | Library (code-first) | Assemble and run the full RAG pipeline in application code |
| **Retrieval libraries** | sentence-transformers, FlashRank, RAGatouille, BM25s | Library (specialized) | Embedding models, fast reranking, ColBERT late interaction, sparse retrieval |
| **Vector databases with RAG features** | Weaviate, Qdrant, Milvus, Pinecone, pgvector, Chroma, FAISS | Service/library (storage layer) | Store vectors; increasingly ship hybrid search, rerankers, and generative modules |
| **Full RAG platforms** | RAGFlow, AnythingLLM, Dify, FastGPT, Verba, txtai, Kotaemon | Application (UI + backend) | Turnkey RAG: document management, retrieval, chat UI out of the box |
| **RAG evaluation frameworks** | RAGAS, TruLens, DeepEval | Library (quality) | Measure retrieval + generation quality; drive regression gates |
| **Managed RAG services** | Bedrock Knowledge Bases, Azure AI Search, Vertex AI Search, watsonx Discovery | Cloud service | Fully managed RAG — bring data, get retrieval + generation APIs |

### 3.1 Ecosystem size snapshot (mid-2026)

GitHub star counts are a rough proxy for community size and mindshare — useful context, not a quality verdict (approximate snapshots, mid-2026; treat as magnitudes, not facts):

| Project | Approx. stars | Category | Notes |
|---------|--------------|----------|-------|
| Dify | ≈148k | Full platform | Fastest-growing; visual LLMOps, passed 100k in June 2025 |
| LangChain | ≈92k | Orchestration | Largest integration ecosystem; v1.0 GA (Oct 2025) |
| RAGFlow | ≈78k | Full platform | GitHub 2025 top-10 project; v0.26.x (July 2026) |
| LlamaIndex | ≈40k | Orchestration | RAG-first; LlamaCloud/LlamaParse |
| Haystack | ≈25k | Orchestration | deepset; stable 2.x |
| Milvus | ≈30k | Vector DB | Distributed; LF AI & Data project |
| Qdrant | ≈20k | Vector DB | Rust; native hybrid search |
| Weaviate | ≈12k | Vector DB | GraphQL API; generative/reranker modules |
| RAGAS | ≈7k | Evaluation | Reference-free RAG metrics |
| txtai | ≈10k | Platform/DB | Embedded semantic database |

Star *growth* is itself informative: Dify and RAGFlow's rise reflects demand for application-level RAG, while LangChain's sustained scale reflects demand for building blocks. Neither signal says which is right for a given bank use case — §13 turns these categories into decisions.

**How to read this guide:** §4–§7 cover the orchestration frameworks (the decision most teams make first); §8–§9 cover the storage and retrieval layer; §10 covers turnkey platforms; §11 covers evaluation; §12 covers managed services; §13–§15 turn it all into a selection and banking recommendation.

---

## 4. LangChain — The Most Popular RAG Framework

LangChain (launched late 2022 by Harrison Chase) is the most widely used RAG framework — the "default first choice" for LLM applications. Its bet: *composability through a large integration surface* — 1000+ integrations spanning models, vector stores, tools, and document sources.

### 4.1 Core components

| Component | What it is |
|-----------|-----------|
| **Document loaders** | 100+ integrations: PDF (PyPDF, PDFPlumber, Unstructured), HTML, DOCX, Markdown, CSV, S3, Notion, Confluence, SharePoint, YouTube transcripts, web crawlers |
| **Text splitters** | RecursiveCharacterTextSplitter (default), Character, Token, Sentence, MarkdownHeader, HTML, Semantic chunking, code-aware splitters |
| **Embedding models** | Unified interface over OpenAI, Hugging Face, Cohere, Ollama, Voyage, Bedrock, Google, plus self-hosted via sentence-transformers |
| **Vector stores** | 50+ adapters: FAISS, Chroma, pgvector, Qdrant, Milvus, Pinecone, Weaviate, Elasticsearch, Redis, Supabase, MongoDB Atlas, LanceDB |
| **Retrievers** | VectorStoreRetriever (similarity, MMR), BM25Retriever, EnsembleRetriever, MultiQueryRetriever, SelfQueryRetriever, ContextualCompressionRetriever, ParentDocumentRetriever, TimeWeightedVectorStoreRetriever |
| **Chains / LCEL** | LangChain Expression Language — declarative pipe (`|`) composition of runnables: `prompt | model | parser`; streaming, batching, async, and retries built in |
| **Agents** | ReAct, tool-calling agents, OpenAI/Anthropic function calling, plan-and-execute |
| **LangGraph** | Graph-based, stateful, low-level orchestration layer (cycles, branching, checkpointing, human-in-the-loop); the recommended path for complex/agentic flows |
| **LangSmith** | Observability + evaluation: tracing every step, dataset management, online/offline evals, feedback collection |
| **LangServe** | Deployment: turn LCEL runnables into REST APIs with auto-generated OpenAPI docs |
| **Memory & output parsers** | Conversation buffers/summaries, structured output (Pydantic), JSON/XML parsing, retry parsers |

### 4.2 RAG-specific offerings

- **MultiQueryRetriever** — LLM generates several paraphrases of the query, retrieves for each, and unions results (mitigates query-phrasing brittleness).
- **SelfQueryRetriever** — LLM extracts a semantic query *plus metadata filters* ("documents about swaps after 2023") and applies them to the store.
- **ContextualCompressionRetriever** — wraps a base retriever with a compressor (e.g., an LLM or an embedding-filter) to keep only the most relevant passages.
- **EnsembleRetriever** — combines dense + BM25 with weights; pairs naturally with reciprocal rank fusion.
- **ParentDocumentRetriever** — retrieves small child chunks, returns their parent chunks (more context, fewer tokens than naive big chunks).
- **VectorStoreRetriever with MMR** — maximal marginal relevance for diversity, reducing redundant passages.
- **RAGAS integration** — official RAGAS package works on LangChain components directly for evaluation.
- **PDF/markdown handling** — loader + splitter combinations tuned for structured docs (markdown header splitters preserve hierarchy — useful for regulatory text).
- **HyDE support** — via prompt templates + embeddings (see [RAG vs HyDE](rag_vs_hyde_guide.md)).

### 4.3 Strengths

- **Largest ecosystem by far** — 1000+ integrations; almost any model, store, or tool has a connector.
- **Maximum flexibility** — components are granular and swappable; nothing is forced.
- **LCEL is genuinely clean** — declarative pipelines are readable, testable, and support streaming/async natively.
- **LangGraph enables complex flows** — cycles, branching, checkpointing, and human-in-the-loop for production-grade agentic RAG.
- **LangSmith** — best-in-class tracing/evals for RAG debugging ("why did the retriever return this?").
- **Active development + huge community** — fast-moving feature set; enterprise support tier exists.
- **Broad language support** — Python, JS/TS, plus community ports.

### 4.4 Weaknesses

- **Steep learning curve** — the sheer API surface is overwhelming; there are five ways to do any one thing.
- **API churn (historically)** — breaking changes between versions (0.1 → 0.2 → 0.3 → 1.0 migrations caused wide disruption). **Since LangChain 1.0 (GA October 22, 2025), the framework commits to no breaking changes until 2.0** — the churn problem is officially behind it, though pre-1.0 codebases still carry migration debt and the ecosystem churns faster than the core.
- **Abstraction leaks** — you eventually need to understand the underlying integrations anyway.
- **"Black box" complexity** — default behavior can hide surprising token counts or retrieval behavior.
- **Over-engineering risk** — teams reach for agents/LangGraph when a plain chain would do.
- **Inconsistent documentation** — tutorials age out quickly as APIs move.

### 4.5 Best for

Teams wanting maximum flexibility and integration coverage; complex RAG workflows and agentic retrieval; production apps needing observability (LangSmith); teams comfortable with rapid framework evolution. **Not ideal** for teams needing API stability and low maintenance overhead — though 1.0's stability commitment (see §4.4) narrows this gap.

### 4.6 A minimal RAG chain in LCEL

```python
from langchain_community.document_loaders import PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_chroma import Chroma
from langchain_core.runnables import RunnablePassthrough
from langchain_core.prompts import ChatPromptTemplate

# 1. Ingest
docs = PyPDFLoader("prospectus.pdf").load()
chunks = RecursiveCharacterTextSplitter(chunk_size=512, chunk_overlap=64).split_documents(docs)
vectorstore = Chroma.from_documents(chunks, OpenAIEmbeddings())

# 2. Retrieve
retriever = vectorstore.as_retriever(search_kwargs={"k": 6})

# 3. Prompt + chain (LCEL)
prompt = ChatPromptTemplate.from_template(
    "Answer using ONLY the context. Cite clause numbers.\n\nContext:\n{context}\n\nQuestion: {question}")
chain = ({"context": retriever, "question": RunnablePassthrough()}
         | prompt | ChatOpenAI(model="gpt-4o-mini"))

print(chain.invoke("What are the redemption terms?"))
```

Production upgrades are incremental: swap `Chroma` for `Qdrant`/`pgvector`, add an `EnsembleRetriever` (BM25 + dense) with RRF, wrap retrieval with a `ContextualCompressionRetriever` + FlashRank reranker, and move to LangGraph when multi-step control flow appears. Two production caveats: the cookbook pattern above is a starting point, not a blueprint — pin versions (LangChain 1.x is the stable baseline) and enable LangSmith tracing before shipping.

---

## 5. LlamaIndex — The RAG-First Data Framework

LlamaIndex (formerly GPT Index, by Jerry Liu) is a data framework *built around RAG*: it treats documents, their chunking, indexing, and retrieval as first-class concepts. Where LangChain is a general LLM framework with RAG capabilities, LlamaIndex is a RAG framework, period.

### 5.1 Core concepts

| Concept | What it is |
|---------|-----------|
| **Documents → Nodes** | Loaded documents are parsed into *nodes* (chunks) with metadata and relationships — the fundamental unit |
| **Indices** | VectorStoreIndex (default), SummaryIndex, TreeIndex, KeywordTableIndex, KnowledgeGraphIndex, PropertyGraphIndex, DocumentIndex |
| **Retrievers / Query Engines** | QueryEngine is the clean, uniform query interface (retrieve → synthesize); retrievers (VectorIndexRetriever, BM25Retriever, etc.) plug underneath |
| **Response synthesizers** | Refine, compact-and-refine, tree-summarize, accumulate — control how retrieved nodes become the answer |
| **Data connectors (LlamaHub)** | 300+ loaders/readers for file types, apps, databases, APIs |
| **Agent framework** | Function-calling and ReAct agents over tools (query engines as tools) |
| **Workflows** | Event-driven orchestration (steps + events), architecturally similar to LangGraph; the modern replacement for the older `QueryPipeline` |
| **LlamaCloud / LlamaParse** | Managed parsing (LlamaParse: layout-aware PDF parsing with tables, figures) and managed ingestion/indexing |
| **Observability** | Callbacks, `set_global_handler` for LangSmith/Arize/Phoenix/W&B integrations |

### 5.2 RAG-specific offerings (the deepest in the ecosystem)

- **Node parsers & relationships** — parent-child, prev-next, hierarchical nodes; chunking is a first-class data-model decision, not an afterthought.
- **QueryFusionRetriever** — ensemble retrieval with reciprocal rank fusion (RRF) or weighted sum across multiple retrievers/embeddings.
- **AutoMergingRetriever** — retrieves leaf chunks, then merges them back into larger parent contexts when siblings hit.
- **Recursive retrieval** — index of summaries → retrieve summary → dive into the underlying nodes (great for long documents).
- **Router query engine** — LLM routes each query to the right engine (vector, summary, SQL, KG).
- **Sub-question query engine** — decomposes a complex question into sub-questions, answers each, composes the final answer.
- **Citation query engine** — answers with explicit source-node citations (useful for auditability in banking).
- **Structured data query engines** — Text-to-SQL over relational data; PandasQueryEngine for dataframes.
- **Metadata extraction** — LLM-extracted metadata (entities, dates, sentiment) attached to nodes for filtering.
- **Property graph index** — combines vector search with knowledge-graph traversal (GraphRAG-style) using Neo4j, NebulaGraph, or in-memory graphs.

### 5.3 Strengths

- **RAG-first design** — advanced retrieval patterns (fusion, auto-merge, sub-questions, citations) are built-in, not bolted on.
- **Strong document processing** — LlamaParse + node parsers + rich metadata handling.
- **Clean abstractions** — `QueryEngine` gives a uniform interface; swapping indices/retrievers is low-friction.
- **Good abstraction/flexibility balance** — high-level engines for speed, low-level primitives when needed.
- **Excellent for document-heavy apps** — PDF-heavy corpora, research, enterprise document QA.

### 5.4 Weaknesses

- **Smaller ecosystem than LangChain** — fewer general-purpose integrations (though LlamaHub is broad).
- **Less general-purpose** — it is RAG/agents-focused; generic LLM app patterns (e.g., pure tool orchestration) are better served elsewhere.
- **API changes across versions** — 0.9 → 0.10 → 0.11 introduced notable renames (index.query → QueryEngine, etc.).
- **Less enterprise support** — no heavyweight commercial support org; LlamaCloud is the managed path.

### 5.5 Best for

RAG-first applications; teams wanting structured query engines (SQL, citations); document-heavy apps; advanced retrieval patterns (fusion, auto-merge); research/academic projects.

### 5.6 LlamaIndex in ~20 lines

```python
from llama_index.core import VectorStoreIndex, Settings
from llama_index.core.node_parser import SentenceSplitter
from llama_index.embeddings.openai import OpenAIEmbedding
from llama_index.llms.openai import OpenAI
from llama_index.readers.file import PDFReader

# Ingest: documents -> nodes -> index (one line each)
documents = PDFReader().load_data("prospectus.pdf")
Settings.node_parser = SentenceSplitter(chunk_size=512)
index = VectorStoreIndex.from_documents(documents)   # embeds + indexes

# Query engine: retrieve + synthesize behind one interface
query_engine = index.as_query_engine(similarity_top_k=6)
print(query_engine.query("What are the redemption terms?"))

# Advanced retrieval in a few lines: fusion + reranking
from llama_index.core.retrievers import QueryFusionRetriever
from llama_index.core.postprocessor import SentenceTransformerRerank
fusion = QueryFusionRetriever([index.as_retriever()], num_queries=2, use_async=True)
query_engine = index.as_query_engine(
    retriever=fusion, node_postprocessors=[SentenceTransformerRerank(top_n=5)])
```

The contrast with LangChain is visible in the code: LlamaIndex makes *indexing* (documents → nodes → index) and *query engines* the core abstraction, so RAG-specific patterns (fusion, citations, sub-questions) are options on those primitives rather than separately assembled integrations.

---

## 6. Haystack — Production Pipelines by deepset

Haystack (deepset, Berlin) is an open-source LLM framework whose identity is *production-grade pipelines*: typed components wired into Directed Acyclic Graphs (DAGs), with a stable, versioned API. It predates the LLM boom (2021, extractive QA era) and has evolved into a general LLM framework with unusually strong RAG support.

### 6.1 Core concepts

| Concept | What it is |
|---------|-----------|
| **Pipelines** | DAGs of typed components (`pipeline.add_component("retriever", retriever)`; `connect(...)`); data flows as typed dicts between named connections |
| **Components** | Retrievers, rankers, readers, generators, converters, preprocessors, routers — each with declared inputs/outputs |
| **Document stores** | Elasticsearch, OpenSearch, Qdrant, Pinecone, Weaviate, Chroma, InMemory, SQL, Astra; unified `DocumentStore` protocol |
| **Agents** | Tool-based agents (function calling) — present, but less mature than LangChain's agent ecosystem |
| **deepset Cloud** | Managed platform: hosted pipelines, evaluation, observability, and model serving |
| **Haystack 2.x** | Fully typed, DAG-native rewrite (2024); 1.x legacy pipeline style deprecated |

### 6.2 RAG-specific offerings

- **Retrievers** — BM25Retriever (sparse), EmbeddingRetriever (dense), and a built-in **HybridRetriever** (BM25 + dense fused, with configurable weights) — hybrid retrieval out of the box, no assembly required.
- **Rankers** — CohereRanker, SentenceTransformersRanker (cross-encoders), LostInTheMiddleRanker — plug in after retrieval.
- **Reader (extractive QA)** — the classic SQuAD-style span-extraction reader (historical strength).
- **Generator (generative QA)** — prompt-based generative answer synthesis over retrieved docs.
- **Evaluation pipelines** — built-in evaluation components (SQuAD-style exact match/F1, faithfulness/groundedness metrics, retrieval recall) that run inside the same pipeline machinery.
- **File converters** — PDF, DOCX, TXT, Markdown, sitemap, and audio via Whisper (speech-to-text for meeting/video corpora).
- **Preprocessors** — chunking (fixed-length, sentence-aware), metadata extraction, cleaning (HTML tags, whitespace), language detection.

### 6.3 Strengths

- **Clean pipeline abstraction** — DAGs are explicit, inspectable, and testable; typed components reduce wiring bugs.
- **Production-ready** — used by enterprises; robust error handling, streaming, and async support.
- **Built-in hybrid retrieval** — BM25 + dense fusion is a first-class feature, not a DIY integration.
- **Good evaluation support** — evaluation components are part of the framework, not an afterthought.
- **Stable, versioned API** — far fewer breaking changes than LangChain; 2.x is deliberately backward-compatible in spirit.
- **Strong typing and documentation** — excellent DX for teams that value explicitness.
- **European/enterprise presence** — deepset is German; strong in DACH and EU-regulated industries (relevant for a French bank's EU footprint).

### 6.4 Weaknesses

- **Smaller ecosystem** — fewer integrations than LangChain (document loaders ~20 vs 100+).
- **Less flexible for exotic patterns** — the DAG model constrains cyclic/recursive flows (agents exist but are less mature than LangGraph).
- **Smaller community** — fewer tutorials/community answers than LangChain.
- **Fewer agent features** — tool orchestration is functional but not the differentiator.

### 6.5 Best for

Production RAG in enterprise environments; teams wanting stable APIs and gentle learning curves; hybrid retrieval out of the box; structured, auditable pipelines; European enterprise deployments.

### 6.6 A Haystack RAG pipeline

```python
from haystack import Pipeline
from haystack.components.retrievers import HybridRetriever
from haystack.components.rankers import SentenceTransformersRanker
from haystack.components.builders import PromptBuilder
from haystack.components.generators import OpenAIGenerator

pipe = Pipeline()
pipe.add_component("hybrid", HybridRetriever(document_store=store, top_k=50))
pipe.add_component("ranker", SentenceTransformersRanker(
    model="cross-encoder/ms-marco-MiniLM-L-6-v2", top_k=5))
pipe.add_component("prompt", PromptBuilder(template=TEMPLATE))
pipe.add_component("llm", OpenAIGenerator(model="gpt-4o-mini"))
pipe.connect("hybrid", "ranker")
pipe.connect("ranker", "prompt")
pipe.connect("prompt", "llm")
result = pipe.run({"hybrid": {"query": "What are the redemption terms?"}})
```

Note what Haystack gives you for free: the hybrid retriever (BM25 + dense fused) and the cross-encoder ranker are **stock components**, not integrations you assemble — and the pipeline is serializable (YAML) and typed, which is why it slots into enterprise change-control processes.

---

## 7. Orchestration Frameworks: Head-to-Head

| Dimension | LangChain | LlamaIndex | Haystack |
|-----------|-----------|------------|----------|
| **Architecture** | Chains / LCEL runnables + LangGraph stateful graphs | Indices + QueryEngines + event-driven Workflows | Typed DAG Pipelines |
| **Document loaders** | 100+ | 300+ via LlamaHub | ~20 |
| **Retrievers** | Many (multi-query, self-query, ensemble, compression, parent-doc, MMR) | Advanced RAG-focused (fusion/RRF, auto-merge, sub-question, citations, recursive) | Hybrid built-in (BM25+dense), rankers, classic readers |
| **Orchestration** | LangGraph (graphs, cycles, checkpointing, HITL) | Workflows (event-driven steps) | Pipelines (DAGs) |
| **Evaluation** | LangSmith (tracing + evals), RAGAS integration | Callbacks/observability (LangSmith, Arize, Phoenix), RAGAS integration | Built-in eval components + deepset Cloud evals |
| **Stability** | Historically rapid; 1.0 (Oct 2025) promises no breaking changes until 2.0 | Moderate (rename-heavy majors) | Stable, versioned |
| **Learning curve** | Steep | Moderate | Gentle |
| **Agent support** | Mature (ReAct, tool-calling, LangGraph agents) | Function-calling agents over query engines | Tool-based agents (younger) |
| **Language support** | Python + JS/TS | Python (+ TS partial) | Python (+ TypeScript in 2.x) |
| **Managed offering** | LangSmith/LangGraph Platform | LlamaCloud | deepset Cloud |
| **Enterprise support** | Yes (LangChain Inc.) | Via LlamaCloud | Yes (deepset) |
| **Best for** | Maximum flexibility, complex/agentic workflows, observability-first teams | RAG-first apps, advanced retrieval patterns, document-heavy | Stable enterprise production, hybrid retrieval, structured pipelines |

### 7.1 Practical guidance on choosing

- **Default to LangChain** when the team is Python-heavy, needs the widest integration coverage, and accepts API churn; pair with LangGraph for anything cyclic/agentic and LangSmith for observability.
- **Choose LlamaIndex** when retrieval quality is the product — fusion, auto-merging, citations, Text-to-SQL, GraphRAG-style indexing are all one import away.
- **Choose Haystack** when the system must be operated like software: typed, versioned, documented, and stable; EU-regulated environments value this.
- **You can mix.** LlamaIndex inside a LangGraph flow, or Haystack pipelines behind a LangChain agent, are both common. The framework is the skeleton, not the straitjacket.
- **Avoid the "second framework" trap.** Teams often import a second framework for one nice retriever. Prefer replicating that one pattern with your primary framework — or standardize the glue via a thin internal layer (see §14's banking stack).

### 7.2 Community and ecosystem size

- **LangChain** — the largest community by far: ≈92k GitHub stars, 1000+ integrations, the most tutorials and answers; hiring LangChain-experienced engineers is easy; LangSmith/LangGraph platform momentum continues.
- **LlamaIndex** — ≈40k stars, 300+ LlamaHub loaders, an active Discord; the RAG-research and document-heavy crowd converges here.
- **Haystack** — ≈25k stars; smaller but high-quality community; deepset runs the project with enterprise support and SLAs; strong EU enterprise presence.

All three are permissively licensed (LangChain MIT, LlamaIndex MIT, Haystack Apache-2.0), so licensing is not a differentiator for bank adoption. A pragmatic tiebreaker: **which framework does the team's next hire already know?** In 2026 the answer is usually LangChain, with LlamaIndex second.

---

## 8. Vector Databases with RAG Features

The vector store is the persistence and retrieval backbone of RAG. The category spans embedded libraries (FAISS, Chroma), database extensions (pgvector), dedicated vector DBs (Qdrant, Milvus, Weaviate, Vespa), fully managed services (Pinecone), and search engines with vector capability (Elasticsearch, OpenSearch, Redis). Modern vector DBs increasingly absorb RAG features — hybrid search, reranking modules, and even generative modules — blurring the line with frameworks.

| DB | Type | Open-source | Deployment | Hybrid search | Filtering | Scaling | Best for |
|----|------|-------------|------------|---------------|-----------|---------|----------|
| **FAISS** | Library (in-memory index) | ✅ (MIT) | Embedded | ❌ (manual) | ✅ metadata (IVF/PQ via ids) | Single-node, RAM-bound | Prototyping, research, GPU batch search |
| **Chroma** | Embedded DB | ✅ | Embedded/local server | Partial (late) | ✅ metadata + document filters | Single-node | Python-first local apps, quick prototypes, notebooks |
| **pgvector** | PostgreSQL extension | ✅ | In existing Postgres | ✅ (v0.7+ sparse/hybrid) | ✅ full SQL | Vertical + read replicas; scales with Postgres | Apps already on Postgres; ACID + SQL needs |
| **Qdrant** | Dedicated vector DB (Rust) | ✅ | Self-host or Qdrant Cloud | ✅ native (BM25 + dense) | ✅ rich payload filtering, geo | Horizontally scalable, quantization | Production at scale; filtering-heavy workloads |
| **Milvus** | Distributed vector DB | ✅ | Self-host (K8s) or Zilliz Cloud | ✅ (sparse+ dense) | ✅ rich scalar filters | Massive scale, GPU index, distributed | Huge corpora (100M+ vectors), cloud-native teams |
| **Weaviate** | Dedicated vector DB (Go) | ✅ | Self-host or Weaviate Cloud | ✅ native (BM25+vector) | ✅ filters + multi-tenancy | Horizontally scalable | Production RAG; GraphQL API; built-in modules (generative, reranker) |
| **Pinecone** | Managed vector DB | ❌ | SaaS only (serverless) | ✅ (2024) | ✅ metadata | Serverless auto-scale | Teams wanting zero-ops managed search; fast startup |
| **Elasticsearch / OpenSearch** | Search engine | ✅ | Self-host or cloud | ✅ (kNN + BM25, RRF) | ✅ full Lucene queries | Mature, large-cluster ops | Existing ELK users; hybrid enterprise search |
| **Redis** | Key-value store + modules | ✅ | Self-host or Redis Cloud | ✅ (RediSearch) | ✅ | Vertical + clustering | Teams already on Redis; caching + vector in one |
| **Vespa** | Big-data serving engine | ✅ | Self-host (complex) | ✅ (BM25 + vector + tensor) | ✅ | Very large scale, low latency | High-scale personalized/ranking workloads (Spotify) |
| **LanceDB** | Embedded columnar DB | ✅ | Embedded, serverless | ✅ (late) | ✅ | Multi-modal, blob + vector | Developer-friendly local/multi-modal RAG |

### 8.1 Profiles of the leading choices

- **FAISS** — a *library*, not a server: in-memory indexes (IVF, HNSW, PQ), extremely fast, GPU-accelerated search via `faiss-gpu`. No persistence, no concurrency, and no built-in metadata filtering beyond ID-based subsetting — you manage serialization, reload, and sharding yourself. Ideal for prototyping and research; production use means wrapping it in your own service.
- **Chroma** — embedded, Python-first, persistent by default; the simplest "get a vector store running" experience, with metadata and document filters plus a local server mode. Great for prototypes and local tools; less suited to multi-service production (single-node, Python-centric).
- **pgvector** — a Postgres extension adding a `vector` type, HNSW/IVFFlat indexes, and distance operators. If you already run Postgres, this is the lowest-ops choice: relational data + vectors in one ACID database, full SQL filtering, and hybrid (full-text `tsvector` + vector) in the same query. Scales with Postgres — which means it needs Postgres expertise (partitioning, read replicas) at scale.
- **Qdrant** — Rust-based with high performance, rich payload filtering (nested, geo, full-text), scalar/product/binary quantization, native hybrid search (sparse + dense), and snapshotting for backups. A strong default for production self-hosted RAG; cloud and hybrid deployment options exist.
- **Milvus** — distributed, cloud-native (Kubernetes), GPU-accelerated indexing, and enormous scale (100M+ vectors common); a Linux Foundation (LF AI & Data) project. The cost is operational complexity — it is a real distributed system with coordinators, message queues, and object storage.
- **Weaviate** — Go-based with a GraphQL (and REST) API, built-in hybrid search (BM25 + vector), modules for generative and reranking steps (Cohere, OpenAI, local rerankers), and multi-tenancy for per-client isolation. Feature-complete as a RAG store; a common production choice.
- **Pinecone** — fully managed, serverless API with metadata filtering and hybrid search (since 2024). Zero ops and fast to start, but no self-hosting and cost-per-query scales with usage — effectively "vector search as a service."
- **Elasticsearch/OpenSearch** — BM25-native search engines with kNN vector search and RRF hybrid (Lucene HNSW). If the org already runs the ELK stack, this reuses existing ops muscle and licensing; also the backend of choice for Haystack enterprise deployments.
- **Redis** — vector similarity via RediSearch modules on top of an existing Redis deployment; convenient when Redis is already the cache/queue standard, but vector search is not Redis's core strength.
- **Vespa** — full-text + vector + tensor ranking in one engine; used by Spotify and Yahoo at massive scale for production ranking. Extremely powerful, notoriously complex to operate — for teams that need ranking beyond simple top-k.
- **LanceDB** — embedded, columnar, multi-modal (images, audio, vectors, text in one store), serverless-friendly; developer-friendly and rising for local-first and multi-modal RAG.

### 8.2 Selection heuristics

- **Prototype** → FAISS or Chroma (zero ops).
- **Already on Postgres** → pgvector (one less system to run).
- **Production, self-hosted, need filtering + hybrid** → Qdrant or Weaviate.
- **100M+ vectors, GPU, distributed** → Milvus (or managed Zilliz).
- **Zero-ops managed** → Pinecone (or managed Qdrant/Weaviate).
- **Existing ELK stack** → Elasticsearch/OpenSearch hybrid.
- **Latency-critical high-scale ranking** → Vespa.
- **Multi-modal or local-first** → LanceDB.

### 8.3 How to evaluate a vector database for your workload

Don't choose on vendor benchmarks alone — run a structured bake-off before committing:

| Dimension | What to measure | How |
|-----------|-----------------|-----|
| Recall@k | Fraction of ground-truth hits in top-k | Build a gold set of 100–500 queries with labeled relevant docs (or use BEIR-style datasets); tune HNSW `ef`/`efConstruction` |
| Latency | p50/p95/p99 query latency at target QPS | Load-test with your real payload and filter mix |
| Filtering | Latency impact of metadata filters (tenant, date, product) | Run queries with and without filters; watch index degradation |
| Ingestion | Docs/sec for your chunked corpus | Time a full reindex of a representative corpus |
| Hybrid quality | RRF hybrid vs. dense-only recall | Compare on the same gold set |
| Ops | Backup/restore, snapshots, scaling, upgrade path | Hands-on evaluation; talk to existing operators |

For banking, the filter test matters more than raw recall — multi-tenant access control (desk, legal entity, jurisdiction) must not collapse latency — and verify the hybrid path works with your BM25 settings before trusting vendor "hybrid search" claims.

---

## 9. Retrieval Libraries and Reranking

Orchestration frameworks integrate retrieval, but the *retrieval quality* often comes from specialized libraries. This layer is where accuracy battles are won or lost.

### 9.1 Embedding and reranking libraries

- **sentence-transformers** (SBERT) — the de facto standard library for embedding models and cross-encoders: `all-MiniLM-L6-v2`, `bge-*`, `gte-*`, `e5-*`, multilingual models. Basis for most self-hosted embedding pipelines and for cross-encoder rerankers. Hugging Face `sentence-transformers` models are what most frameworks call under the hood.
- **FlashRank** — ultra-fast reranker: ONNX-optimized, quantized cross-encoder distillation; **30–100× faster than full cross-encoders**, CPU-friendly, no GPU or torch dependency needed. The pragmatic default for reranking on CPU in production.
- **RAGatouille** — ColBERT implementation: *late interaction* (token-level matching between query and document embeddings) with a MaxSim scoring mechanism. Strong accuracy — often the best non-cross-encoder reranker — at higher compute and index storage cost. The go-to for research-grade retrieval accuracy (ColBERTv2).
- **BM25s / rank-bm25** — sparse lexical retrieval: BM25 (and variants) still wins for exact-term, ID-heavy, and regulatory queries. `rank-bm25` is the classic Python lib; `bm25s` is the fast, memory-efficient rewrite (uses sparse matrices, multithreading).

### 9.2 Hybrid retrieval — dense + sparse

Hybrid retrieval combines dense (semantic) and sparse (lexical) signals. **Why:** dense embeddings miss exact terms (clauses, product codes, ISINs, names); BM25 misses paraphrases and synonyms. Combining them is the single most reliable retrieval-quality lift in production RAG.

Fusion methods:

| Method | How it works | Notes |
|--------|--------------|-------|
| **Reciprocal Rank Fusion (RRF)** | Score = Σ 1/(k + rankᵢ) across result lists | Simple, robust, no score normalization needed; the default choice |
| **Weighted sum (score fusion)** | α · dense_score + β · sparse_score (normalized) | More tunable, more fragile to score distributions |
| **Learning-to-rank** | Train a model (e.g., LightGBM) on features from both retrievers | Best accuracy, requires training data + MLOps |

Implementation: LangChain `EnsembleRetriever` (weights + RRF), LlamaIndex `QueryFusionRetriever` (RRF/weighted), Haystack `HybridRetriever` (built-in), and native hybrid in Qdrant/Weaviate/ES/OpenSearch. RRF across dense + BM25 typically recovers most of the recall of both.

### 9.3 Reranking — the highest-ROI retrieval upgrade

The standard pattern: **retrieve top-50–100 candidates cheaply (bi-encoder/BM25/hybrid), rerank to top-5–10 with a more expensive model, then prompt.** Reranking directly attacks the "right answer buried at position 12" failure mode.

| Approach | Speed | Accuracy | Use case |
|----------|-------|----------|----------|
| **Bi-encoder (embedding similarity)** | Fastest | Lower | First-stage retrieval only |
| **Cross-encoder reranker** | Slow (per-pair forward pass) | Highest | Second stage, small candidate sets; GPU or FlashRank ONNX on CPU |
| **ColBERT (late interaction)** | Medium | Near cross-encoder | Token-level matching; good when query/doc phrasing diverge |

Common rerankers:

- **Cohere Rerank** — managed API, multilingual, excellent out-of-box quality; easy integration via LangChain/LlamaIndex/Haystack.
- **bge-reranker** (BAAI) — strong open-source cross-encoder family (bge-reranker-v2-m3 multilingual), self-hostable.
- **cross-encoder/ms-marco-MiniLM-L-6-v2** — the classic lightweight cross-encoder; a reliable baseline on CPU.
- **FlashRank** — the CPU production pick when latency matters (see §9.1).
- **Jina Reranker / mixedbread / Qwen3-Reranker** — newer open models with strong MTEB Reranking scores.

**Pipeline rule of thumb:** hybrid retrieval (RRF) → cross-encoder rerank top-50 → top-5/10 into the prompt. In banking, this combination is what makes regulatory-document QA usable (see §14).

### 9.4 Reranking in practice

```python
from flashrank import Ranker, RerankRequest

ranker = Ranker(model_name="ms-marco-MiniLM-L-12-v2", cache_dir="/opt/rerank-models")

def rerank(query: str, hits: list[dict], top_n: int = 10) -> list[dict]:
    req = RerankRequest(query=query, passages=hits)
    return ranker.rerank(req)[:top_n]   # ~10-30 ms per 50 passages on CPU

# Wire into LangChain as a compressor
from langchain.retrievers import ContextualCompressionRetriever
from langchain.retrievers.document_compressors import FlashrankRerank
compressor = ContextualCompressionRetriever(
    base_compressor=FlashrankRerank(), base_retriever=hybrid_retriever)
```

FlashRank's ONNX models run on CPU with no GPU and no torch dependency — the pragmatic production choice when the rerank budget is tens of milliseconds (§14).

### 9.5 Embedding model selection

| Model family | Dims | Notable strengths | Notes |
|--------------|------|-------------------|-------|
| OpenAI text-embedding-3-small/large | 1,536 / 3,072 (matryoshka down to 256+) | Strong general quality, API-simple | Managed; data leaves premises |
| Cohere embed-v3 / embed-4 | 1,024 / 2,048 | Strong multilingual; int8 compression | Managed; good for EU languages |
| BGE (bge-m3, bge-large) | 1,024 | Multilingual (100+ langs), sparse + dense in one model | Self-hostable; strong MTEB |
| GTE (gte-large, gte-Qwen2) | 1,024–3,584 | Strong English/Chinese | Self-hostable |
| E5 (multilingual-e5) | 384–1,024 | Strong retrieval-tuned | Self-hostable |
| Voyage (voyage-3-large) | 1,024 | RAG-tuned, code support | Managed |
| Domain fine-tunes | varies | Banking/code/legal corpora | Requires eval + MLOps |

Selection rules for banking: (1) test on *your* corpus and QA set, not MTEB — domain jargon changes rankings; (2) prefer multilingual when the corpus mixes EN/FR/DE (CACIB footprint); (3) check the embedding model's context window against your chunk size; (4) for on-prem, choose a self-hostable family (BGE/GTE/E5) and validate GPU sizing; (5) treat the embedding model as a governed model artifact (SR 11-7, §15.3) — pin and revalidate versions.

---

## 10. Full RAG Platforms

Full RAG platforms are *applications* rather than libraries: they ship UI, document management, retrieval, and generation bundled, so a team can stand up a RAG system without writing pipeline code. They trade flexibility for speed-to-value.

| Platform | Open-source | Document parsing | Vector store | GraphRAG | UI | Deployment | Best for |
|----------|-------------|------------------|--------------|----------|-----|------------|----------|
| **RAGFlow** (infiniflow) | ✅ (Apache 2.0) | **Deep document understanding**: OCR, layout analysis, template-based chunking (headline/table/figure) | Built-in (Elasticsearch/Infinity backend) | ✅ built-in GraphRAG | ✅ web UI | Self-host (Docker/K8s), cloud | Document-heavy enterprises; Chinese + global market; GraphRAG out of the box |
| **AnythingLLM** | ✅ | Basic (PDF/txt/docx) | Many (Chroma, LanceDB, Qdrant, Pinecone, Weaviate, local) | ❌ | ✅ desktop + web | Desktop app or Docker server | Individual/local-first use; quick private assistants |
| **Dify** | ✅ (Apache 2.0) | Via plugins (Unstructured etc.) | Many (Weaviate, Qdrant, Milvus, pgvector, etc.) | Partial (KG via plugins) | ✅ visual workflow builder | Self-host or cloud | LLMOps teams building RAG *and* agents/workflows visually |
| **FastGPT** | ✅ (Apache 2.0) | Good (file QA, knowledge base mgmt) | pgvector, Milvus, ES | ❌ (KG roadmap) | ✅ | Self-host | RAG + workflow automation; strong Chinese ecosystem |
| **Verba** (Weaviate) | ✅ | Basic + Weaviate modules | Weaviate only | ❌ | ✅ | Self-host | Weaviate users wanting a ready RAG assistant |
| **txtai** | ✅ (Apache 2.0) | Basic | Built-in embedded | ❌ | ❌ (API/notebook) | Embedded/library, lightweight | Semantic search + RAG in lightweight/edge environments |
| **Kotaemon** | ✅ | Good (multi-format) | Many (via LlamaIndex) | ❌ | ✅ | Self-host (Gradio) | Document QA with **Citoid-style citations**; research-style workflows |
| **NetRaven** | ✅ | Via connectors | Graph-backed (Neo4j etc.) | ✅ (graph-native) | ✅ | Self-host | RAG over knowledge graphs (entity-heavy corpora) |

### 10.1 Profiles

- **RAGFlow** — the standout for document parsing. Its deep-document-understanding chunking (layout, tables, figures via template-based chunking) solves the "PDFs with tables and headers chunk badly" problem that plagues naive frameworks. Includes OCR, knowledge-graph support, built-in GraphRAG, and an agent/workflow UI, backed by Elasticsearch/Infinity for storage. The strongest open-source *document-centric* RAG platform of 2025–2026 (≈78k stars, GitHub 2025 top-10) and a natural reference for banking document pipelines (§14).
- **AnythingLLM** — desktop/server app with a simple UI over many LLM providers and vector stores; local-first and privacy-friendly. Good for personal assistants and small-team pilots; not built for enterprise-scale governance (no deep access control or audit).
- **Dify** — an LLMOps platform: visual workflow builder, RAG pipelines, agents, observability, self-hosting. Bigger than RAG — it is a full app platform — which is both its strength (end-to-end productization, ≈148k stars) and its complexity (workflow nodes constrain what you can express).
- **FastGPT** — RAG + workflow with strong knowledge-base management; very popular in the Chinese ecosystem; good pgvector/Milvus support and a clean admin UI for KB operations.
- **Verba** — Weaviate's open-source RAG assistant: chat over documents with modular retrieval built on Weaviate's hybrid search. Best as a reference implementation and fast demo for Weaviate-based stacks.
- **txtai** — an embedded AI database ("everything semantic") combining semantic search, LLM orchestration, and RAG in one lightweight Python library; no server required. Good for edge/embedded deployments and quick internal tools.
- **Kotaemon** — open-source RAG UI for document QA with Citoid-style citations (clickable, verifiable references) — the closest OSS analog to the citation UX enterprises want for auditable answers; built on LlamaIndex primitives.
- **NetRaven** — RAG over knowledge graphs: retrieval traverses graph structure rather than flat chunks; for entity-heavy, relationship-dense domains (financial contracts, org structures, instrument hierarchies) where "who relates to whom" is the question.

### 10.2 Platform vs. framework

| Question | Framework (LangChain/LlamaIndex/Haystack) | Platform (RAGFlow/Dify) |
|----------|-------------------------------------------|--------------------------|
| Speed to first working system | Days–weeks | Hours–days |
| Customization ceiling | Very high | Medium (workflow nodes constrain) |
| Document parsing depth | Manual assembly (or LlamaParse) | Built-in (RAGFlow best-in-class) |
| Governance/auditability | Code-reviewed, testable | UI-driven, harder to version |
| Ops burden | You operate everything | Platform operates the pipeline |
| Best when | RAG is core IP; needs deep integration | RAG is a feature; speed-to-value dominates |

---

## 11. RAG Evaluation Frameworks

RAG evaluation is split between *retrieval metrics* (did we fetch the right context?) and *generation metrics* (is the answer faithful and useful?). Dedicated frameworks operationalize this with LLM-as-judge scoring, synthetic test data, and CI integration.

| Framework | Core metrics | Reference-free | Integration | CI | Best for |
|-----------|--------------|----------------|-------------|-----|----------|
| **RAGAS** | Faithfulness, answer relevance, context precision, context recall, context entity recall, noise sensitivity, hallucination | ✅ (LLM-judged, no gold labels) | LangChain, LlamaIndex, Haystack, OpenSearch; `RAGAS` package | ✅ pytest/scriptable | Standard RAG quality gates; synthetic test set generation |
| **TruLens** | Groundedness, answer relevance, context relevance (**RAG triad**), QA correctness | ✅ | LangChain, LlamaIndex, Haystack, custom | ✅ | Streaming evaluation, app-integrated feedback, dashboards |
| **DeepEval** | Faithfulness, answer relevancy, contextual precision/recall, hallucination, G-Eval, LLM-as-judge | ✅ | Any LLM app (pytest-style asserts) | ✅ pytest-native | CI-embedded regression testing; developer workflows |
| **LangSmith** | Custom eval sets, online/offline evals, feedback, pairwise | Mixed (needs datasets) | LangChain-native (also LlamaIndex) | ✅ | Observability + evaluation in one; production tracing |

### 11.1 RAGAS — the reference standard

RAGAS (Retrieval-Augmented Generation Assessment) computes RAG-specific metrics with LLM judges, **without requiring ground-truth answers** (reference-free), plus synthetic test-data generation from your corpus:

- **Faithfulness** — is every claim in the answer supported by the retrieved context? (The headline hallucination metric.)
- **Answer relevance** — does the answer actually address the question? (Also penalizes unhelpful-but-factual replies.)
- **Context precision** — are the retrieved chunks relevant, and ranked with the relevant ones first?
- **Context recall** — did retrieval surface the chunks needed to answer? (Needs ground truth.)
- **Context entity recall** — entity-level overlap between retrieved context and reference answer (sharp for banking names/identifiers).
- **Noise sensitivity / hallucination** — degradation of faithfulness when irrelevant chunks are injected; direct hallucination detection.
- **Synthetic test data** — generate question–answer–context triples from the corpus (with or without ground truth) to bootstrap eval sets.

```python
from ragas import EvaluationDataset, SingleTurnSample
from ragas.metrics import faithfulness, answer_relevancy, context_precision
from ragas.evaluate import evaluate

samples = [SingleTurnSample(user_input=q, response=a,
                            retrieved_contexts=ctx, reference=gold)
           for q, a, ctx, gold in banking_qa_set]
dataset = EvaluationDataset(samples=samples)
score = evaluate(dataset, metrics=[faithfulness, answer_relevancy, context_precision])
# -> {"faithfulness": 0.91, "answer_relevancy": 0.87, "context_precision": 0.82}
```

Usage pattern: evaluate on a 50–200-question sample per release; track faithfulness + context precision as the two headline numbers; fail the build on regressions. Typical acceptance thresholds for a banking assistant pilot: faithfulness ≥ 0.85, context_precision ≥ 0.70, answer_relevancy ≥ 0.80 — exact numbers set per use case and revalidated on every release (§15.6). This is the evaluation backbone recommended for banking QA.

### 11.2 TruLens and the RAG triad

TruLens organizes evaluation around the **RAG triad**: context relevance (are the retrieved docs relevant?), groundedness (is the answer supported by context?), answer relevance (does the answer satisfy the question?). Feedback functions run *inline* with app execution (streaming), which makes it the easiest framework for continuous, in-production evaluation dashboards. Integrations: LangChain, LlamaIndex, Haystack, plus custom apps.

### 11.3 DeepEval — pytest for LLM apps

DeepEval makes evaluation feel like unit testing: `assert_test(test_case, [assert_metric...])` in pytest, with LLM-as-judge metrics (faithfulness, answer relevancy, contextual precision/recall, hallucination), G-Eval for open-ended quality, and synthetic data generation. The pytest-native model slots directly into CI pipelines — the best fit for developer-driven regression gates.

### 11.4 LangSmith — observability + evals

LangSmith couples tracing (every retriever call, prompt, and generation) with evaluation: create datasets, run offline evals on chain versions, and collect online feedback from production. If the app is built on LangChain, LangSmith is the lowest-friction observability + evaluation package; it also serves LlamaIndex apps via callbacks. For non-LangChain stacks, TruLens or DeepEval + a tracing backend (Phoenix, Arize) is the alternative.

### 11.5 An evaluation harness in CI

```
release → build golden QA set (200 labeled questions) → run pipeline
        → compute RAGAS metrics + DeepEval asserts
        → gate: faithfulness ≥ 0.85 && context_precision ≥ 0.70
        → on regression: diff failed samples, triage (retriever vs. prompt vs. model)
```

DeepEval makes the gate a pytest test (`assert_test`); RAGAS provides the scores; LangSmith/TruLens provide the traces that make regressions debuggable. The three are complementary, not competing: **RAGAS for metrics, DeepEval for CI ergonomics, LangSmith/TruLens for tracing.**

---

## 12. Managed RAG Services

Managed RAG services remove infrastructure entirely: bring documents, configure chunking/embedding, and consume retrieval-and-generation APIs. They suit organizations that want RAG without a platform team — at the price of vendor lock-in and less customization.

| Service | Cloud | Vector search | Hybrid | Reranking | Data sources | Integration | Pricing | Best for |
|---------|-------|---------------|--------|-----------|--------------|-------------|---------|----------|
| **AWS Bedrock Knowledge Bases** | AWS | ✅ (OpenSearch Serverless, Aurora PostgreSQL, Neptune) | ✅ (2024) | ✅ (Bedrock rerank models) | S3, web crawler, Salesforce, Confluence, SharePoint, etc. | Bedrock models (Claude, Titan, Llama…), `RetrieveAndGenerate` API | Per-request + vector-store infra | AWS-native enterprises; fully managed RAG on Bedrock |
| **Azure AI Search** | Azure | ✅ | ✅ vector + full-text + **semantic ranker** | ✅ semantic ranker | Azure Blob, Cosmos DB, SQL, SharePoint, web; **indexers + knowledge store** | Azure OpenAI (GPT-4o family) data ingestion/grounding | Per-index/query tiers | Enterprise Azure shops; OpenAI-heavy stacks |
| **Google Vertex AI Search** | GCP | ✅ | ✅ | ✅ (semantic reranking) | Websites, Cloud Storage, BigQuery, Drive, structured data | Gemini grounding, agent builder | Per-query + storage | GCP-native enterprises; website/document enterprise search with Gemini |
| **IBM watsonx Discovery** | IBM Cloud / on-prem | ✅ | ✅ | ✅ (relevancy training) | Enterprise docs, web, structured; document understanding (OCR/layout) | watsonx.ai models (Granite), NLU | Per-document/query tiers | Regulated industries needing on-prem options; IBM shops |

### 12.1 Profiles

- **AWS Bedrock Knowledge Bases** — fully managed RAG on AWS: create a knowledge base over S3/web sources, choose chunking + embedding strategy, back it with OpenSearch Serverless, Aurora PostgreSQL (pgvector), or Neptune (graph), then call `RetrieveAndGenerate` (retrieval + generation in one API) with any Bedrock model (Claude, Titan, Llama, Mistral). Hybrid search support added 2024, with Bedrock rerank models for the post-retrieval stage. The default for AWS-native organizations; IAM, CloudTrail, and KMS encryption align with bank-grade controls.
- **Azure AI Search** — a full search service (not just vectors): vector + hybrid + full-text with the *semantic ranker* (Microsoft's transformer reranker) — among the strongest built-in reranking in managed services. Tight integration with Azure OpenAI: indexers ingest from Blob/Cosmos/SQL into a *knowledge store*, and the "chat with your data" pattern is first-class. The natural choice for Azure shops and OpenAI-heavy stacks.
- **Google Vertex AI Search** — managed enterprise search with Gemini grounding; indexes websites, Cloud Storage, BigQuery, Drive, and structured data; supports agent builder for conversational search. Strong for GCP-native enterprises and for grounding Gemini-based agents.
- **IBM watsonx Discovery** — enterprise search + RAG with document understanding (OCR, layout), NLP enrichment, relevancy training, and **on-prem deployment options** — a differentiator for regulated industries with data-sovereignty constraints; integrates with watsonx.ai Granite models.

### 12.2 Managed vs. self-built

Managed services win on speed, ops reduction, and built-in security controls; they lose on customization (chunking strategies, reranking pipelines, fine-tuned embeddings), cost transparency at scale, and portability. A common enterprise pattern: **start with a managed service for the pilot, then move to a self-hosted stack (framework + Qdrant/Weaviate + reranker) when retrieval quality and cost become the binding constraints.** For banks, the managed path is attractive where region-locked deployment and compliance attestations are requirements (e.g., Bedrock in a designated AWS region); see §15.

---

## 13. Selection Guide: Decision Factors and Scenarios

### 13.1 Decision factors

| Factor | Questions to ask | Impact on choice |
|--------|------------------|------------------|
| **Team skills** | Python-heavy or ops-heavy? Do they know Postgres/Elasticsearch? | Python devs → LangChain/LlamaIndex; ops-heavy → managed services or Haystack on existing infra |
| **Deployment** | Cloud, on-prem, or hybrid? Data-residency rules? | On-prem/sovereignty → self-hosted (Qdrant/Weaviate + frameworks); cloud → managed services or cloud DBs |
| **Data types** | PDF-heavy? Structured (SQL)? Code? Audio? | PDF-heavy → RAGFlow parsing or LlamaParse; SQL → LlamaIndex Text-to-SQL; code → repo-aware loaders |
| **Retrieval needs** | Exact-term matches? Semantic only? Hybrid? Reranking? Fusion? | Regulatory/ID-heavy → hybrid + rerank mandatory; semantic-only → dense is fine |
| **Scale** | Prototype vs. production? Vector count? QPS? | Prototype → Chroma/FAISS; production → Qdrant/Milvus/Weaviate/managed |
| **Observability needs** | Do we need to trace retrieval→generation? Audit logs? | Yes → LangSmith/TruLens + logging; minimal → built-in callbacks |
| **Cost constraints** | Token spend on embeddings/generation; infra vs. managed fees | Managed services cost-per-query adds up; self-host amortizes infra |
| **Ecosystem alignment** | What does the org already run? | Postgres → pgvector; Elasticsearch → ES hybrid + Haystack; AWS → Bedrock KB; Azure → AI Search; GCP → Vertex AI Search |

### 13.2 The nine reference scenarios

| # | Scenario | Recommended stack | Rationale |
|---|----------|-------------------|-----------|
| 1 | **Python team, maximum flexibility, complex workflows** | LangChain + LangGraph + LangSmith | Widest integration surface; graphs for complex flows; tracing for debugging |
| 2 | **RAG-first, advanced retrieval, document-heavy** | LlamaIndex (+ LlamaParse) | Fusion, auto-merge, citations, sub-questions built in |
| 3 | **Enterprise production, stable API, hybrid retrieval** | Haystack (+ deepset Cloud optional) | Typed pipelines, stable API, built-in hybrid, EU-friendly vendor |
| 4 | **Quick prototype** | LangChain or LlamaIndex + Chroma/FAISS | Minutes to a working pipeline; swap store later |
| 5 | **Already on Postgres** | pgvector + LlamaIndex/Haystack | One less system; SQL + vector in one DB |
| 6 | **Already on Elasticsearch** | Haystack + ES hybrid | ES BM25 + kNN with RRF; mature ops |
| 7 | **Large-scale managed** | Bedrock KB / Azure AI Search / Vertex AI Search | Zero-ops, compliance attestations, cloud-native integration |
| 8 | **Open-source full platform** | RAGFlow or Dify | Turnkey UI + pipeline; RAGFlow for parsing depth, Dify for LLMOps |
| 9 | **Research / academic** | LlamaIndex + RAGAS + ColBERT (RAGatouille) | Deep retrieval experimentation + rigorous evaluation |

### 13.3 Migration and consolidation paths

- **Prototype → production:** Chroma/FAISS → Qdrant/pgvector is a config-level change in all three frameworks (adapter swap); do it early so the production store gets load-tested.
- **Managed → self-hosted:** Bedrock KB → Qdrant + framework: export the corpus, re-chunk with your own configuration, and compare RAGAS scores before cutting over.
- **Framework swap (rare):** LangChain → LlamaIndex/Haystack is a rewrite of the retrieval layer but not of the corpus or embeddings — keep the index and embedding model stable to decouple the migration.
- **Consolidation:** standardize on one orchestration framework per business unit; let platforms (RAGFlow/Dify) serve no-code teams; keep evaluation centralized in a single RAGAS harness so scores are comparable across teams.

---

## 14. Recommended Stack for Banking

For Crédit Agricole CIB-style environments (regulated, document-heavy, hybrid-retrieval-needing, data-sovereignty-constrained), the recommended reference stack composes the categories above:

```
┌─ INGESTION ─────────────────────────────────────────────────────┐
│  RAGFlow-style parsing (or LlamaParse) → template chunking      │
│  → metadata extraction → hybrid index (pgvector/Qdrant)          │
│  → optional GraphRAG (property graph) for entity-heavy domains   │
├─ QUERY TIME ────────────────────────────────────────────────────┤
│  guardrails + PII redaction → query rewriting / HyDE routing    │
│  → hybrid retrieval (BM25 + dense, RRF)                          │
│  → rerank top-50→10 (FlashRank on CPU or Cohere Rerank)          │
│  → prompt w/ citations → generation (self-hosted or managed LLM) │
│  → evaluation hooks (RAGAS/DeepEval) + tracing (LangSmith/TruLens)│
├─ OPERATIONS ────────────────────────────────────────────────────┤
│  audit logging · PII redaction · model registry · versioning     │
│  · fallback/failover · human-in-the-loop for high-stakes answers │
└──────────────────────────────────────────────────────────────────┘
```

**The concrete stack:**

| Layer | Recommendation | Why |
|-------|----------------|-----|
| Orchestration | **LangChain or LlamaIndex**, with LangGraph or LlamaIndex Workflows | Either works; pick by team preference (§7.1). LangChain for breadth + LangSmith; LlamaIndex for retrieval depth |
| Vector store | **pgvector or Qdrant**, self-hosted on-prem | Data sovereignty; pgvector if Postgres already standard, Qdrant if filtering/scale demands |
| Retrieval | **Hybrid (BM25 + dense) with RRF** | Regulatory text: BM25 catches exact clauses/IDs, dense catches semantics |
| Reranking | **FlashRank on CPU** (or Cohere Rerank where managed is acceptable) | 30–100× faster than cross-encoders; cost- and latency-friendly |
| Document parsing | **RAGFlow-style deep parsing or LlamaParse** | Tables, headers, figures in prospectuses and regulatory filings chunk correctly |
| GraphRAG | Property-graph index for entity-heavy domains | Counterparty networks, instrument hierarchies, org structures |
| Query side | **Query rewriting + HyDE routing** for abstract queries | See [Query Rewriting in RAG](query_rewriting_rag_guide.md) and [RAG vs HyDE](rag_vs_hyde_guide.md) |
| Evaluation | **RAGAS + DeepEval** (pytest gates) on banking QA sets | Faithfulness/context-precision regression gates per release |
| Observability | **LangSmith or TruLens** + audit logging | Trace retrieval→generation; log queries for compliance |
| Deployment | **Self-hosted on-prem**, region-locked managed where allowed | Data residency and model risk governance |
| Security | Guardrails, **PII redaction**, access control, audit trail | Non-negotiable in banking (§15) |

**Why hybrid retrieval is mandatory here:** regulatory and legal documents are retrieved by *exact* references ("Article 22 of MAS Notice 645", "ISIN FR0010415783", "Clause 4.2 of the ISDA Master Agreement") that dense embeddings frequently miss, and by *semantic* queries ("what are the margin requirements for OTC derivatives?") that BM25 misses. RRF over both, plus reranking, is the combination that makes regulatory QA usable.

### 14.1 Reference implementation checklist

| # | Item | Done when |
|---|------|-----------|
| 1 | Corpus inventory | Sources identified, PII-screened, access-classified |
| 2 | Parsing pipeline | Layout-aware chunking validated on 50 representative documents |
| 3 | Retrieval baseline | Dense-only vs. hybrid (RRF) recall comparison on a gold set |
| 4 | Reranker | Top-50→10 quality + latency budget measured (FlashRank/Cohere) |
| 5 | Prompt + citations | Answers cite clause/document IDs; citation accuracy spot-checked |
| 6 | Evaluation harness | RAGAS + DeepEval gates in CI on the banking QA set |
| 7 | Observability | LangSmith/TruLens tracing + query/answer audit log |
| 8 | Security | PII redaction, retrieval-level access control, encryption |
| 9 | Governance | SR 11-7 documentation, model registry, version pinning |
| 10 | Operations | Fallback mode, reindex runbook, latency/cost dashboards |

---

## 15. Banking Considerations Deep Dive

Beyond framework mechanics, RAG in a bank is governed by data, risk, and compliance requirements. These constraints should shape the stack *before* the framework choice is finalized.

### 15.1 Data residency and sovereignty

- **On-prem or region-locked managed services only.** Client data, transaction data, and deal documents must not leave approved jurisdictions. This rules out global-SaaS vector stores unless a compliant region is contractually guaranteed.
- Self-hosted vector stores (pgvector, Qdrant, Weaviate) inside the bank's VPC/data center give the cleanest residency story; if using managed services, verify region locking, encryption at rest (KMS/HSM-backed), and data-deletion SLAs.

### 15.2 Security: PII redaction, access control, audit

- **PII redaction** at ingestion *and* at query time (names, account numbers, identifiers) — redact before embedding/indexing so PII never enters the vector store, and redact in prompts/responses.
- **Access control** must be enforced at retrieval, not just at the UI: row/tenant-level filters in the vector store (Qdrant payload filters, Weaviate multi-tenancy, Postgres RLS with pgvector) so a trader only retrieves documents their entitlements allow.
- **Audit logging** of every query, retrieved context, and generated answer — the vector store, the gateway, and the LLM call must all feed the audit trail (ties into the [Enterprise AI Gateway](../../enterprise_ai_gateway_guide.md) pattern).

### 15.3 Model risk governance (SR 11-7)

Under US FRB SR 11-7 / OCC 2011-12 (and analogous EU/APAC regimes), the RAG stack is part of the model inventory:

- **Document the stack**: framework version, chunking configuration, embedding model, retriever, reranker, prompt templates, generation model — all versioned and registered.
- **Validate components**: embedding models and rerankers are *models* too; they need validation, benchmarking on bank data, and periodic revalidation when versions change.
- **Independent review**: the retrieval pipeline's behavior (top-k, fusion weights, reranker cutoffs) should be reviewed like model parameters, with change control.
- **Version pinning**: freezing embedding-model and framework versions in the registry is both an SR 11-7 and an operational requirement.

### 15.4 Latency SLAs and cost control

- **Latency budget example:** retrieval (50–100 ms) + rerank (50–200 ms) + generation (1–3 s) must fit the SLAs of trading-floor tooling; cache embeddings and (where safe) generation results.
- **Cost control:** embedding tokens are cheap but run over the whole corpus at every ingestion; generation tokens dominate. Mitigate with prompt/context trimming (rerank to top-5/10), caching, and batch ingestion windows. Monitor cost per query per business line.

### 15.5 Compliance: MAS TRM, BCBS 239 lineage

- **MAS Technology Risk Management (TRM)** — the Singapore context: the RAG platform is an IT system; it needs change management, security testing, business continuity, and incident response, with records.
- **BCBS 239** — risk-data lineage: the answer must be traceable to source documents (citations, chunk IDs, document version), and the pipeline must preserve provenance metadata end-to-end. Citation-capable engines (LlamaIndex citation query engine, Kotaemon-style UI) directly support this.
- **Model-risk inventory** for any LLM + embedding + reranker combination used in credit, market, or operational risk contexts.

### 15.6 Observability and evaluation

- **Trace retrieval → generation** end-to-end (LangSmith/TruLens); log queries, retrieved chunks, and answers for compliance and incident review.
- **Evaluate continuously** with RAGAS metrics (faithfulness, context precision, context recall) on a banking QA set covering the highest-risk question types; fail CI on regressions (DeepEval pytest gates).
- **Drift monitoring** on the corpus (new regulations, updated contracts) — chunking and embedding refresh must be tracked and revalidated.

### 15.7 Fallback, failover, and versioning

- **Degraded mode when the vector DB is down** — fail over to BM25-only or to a secondary replica, or answer with a clear "retrieval unavailable" rather than hallucinating. Design the pipeline so retrieval failure is explicit, not silent.
- **Versioning of the retrieval stack**: chunking config, embedding model, reranker, prompts — all versioned; canary deployments of new embedding models (embedding changes invalidate the index — reindex or dual-index during transition).
- **Human-in-the-loop** for high-stakes answers: flag low-confidence or high-impact queries (large exposures, client communication) for review before finalization.

### 15.8 Domain-specific retrieval tuning

- **Hybrid retrieval for regulatory text**: BM25 catches exact terms and clause references; dense catches semantic paraphrases; RRF fusion + reranking on top.
- **Domain-specific embedding models**: fine-tune (or evaluate carefully) embeddings on banking corpora — generic embeddings underperform on financial jargon, entity names, and abbreviation-heavy text. Evaluate on the banking QA set before adopting (§11).
- **Query rewriting + HyDE** for abstract queries ("how do we mitigate counterparty risk in commodity derivatives?") while keeping exact-match retrieval for clause lookups — route between the two (see companion guides).

### 15.9 Governance checklist — quick reference

| Control area | Requirement | Evidence |
|--------------|-------------|----------|
| Data residency | Corpus and indexes in approved regions | Deployment manifest, cloud region attestations |
| PII | Redaction at ingestion + query; no PII in the index | Redaction test suite, index sampling |
| Access control | Retrieval-level entitlements (desk/entity/jurisdiction) | Filter unit tests, penetration test results |
| Model risk (SR 11-7) | Stack documented, components validated, independent review | Model inventory entry, validation report |
| Lineage (BCBS 239) | Answer → chunk → document version traceable | Citation IDs, provenance metadata |
| Latency SLAs | p95 retrieval + generation within budget | Load test reports, SLO dashboards |
| Audit | Every query, context, and answer logged | Log retention policy, SIEM integration |
| Change control | Chunking/embedding/prompt versions gated | Version registry, canary rollout records |
| BCP | Degraded mode when vector DB or LLM unavailable | Failover drill report |
| Evaluation | RAGAS gates per release; drift monitoring | CI report, monthly quality review |

---

## 16. Conclusion and Decision Framework

**The RAG framework landscape is mature enough that the choice is about trade-offs, not capabilities.** Every major option can build a working RAG system; they differ in flexibility, stability, retrieval depth, and operational posture.

**Decision framework in one screen:**

| Your situation | Pick |
|----------------|------|
| Need maximum integrations + flexibility; team tolerates churn | **LangChain** (+ LangGraph, LangSmith) |
| Retrieval quality is the product; document-heavy | **LlamaIndex** (+ LlamaParse) |
| Stable enterprise production; hybrid out of the box | **Haystack** |
| Prototype this week | LangChain/LlamaIndex + **Chroma/FAISS** |
| Already on Postgres / Elasticsearch | **pgvector** / **ES hybrid** |
| Zero-ops managed; cloud-native | **Bedrock KB / Azure AI Search / Vertex AI Search** |
| Turnkey with UI; deep document parsing | **RAGFlow** / Dify |
| Evaluate rigorously | **RAGAS + DeepEval + LangSmith/TruLens** |
| Banking/regulated production | §14 stack: framework + pgvector/Qdrant + hybrid + FlashRank + RAGAS + on-prem + full governance (§15) |

**Final guidance for the banking context:** start the pilot on a managed service or a minimal LangChain/LlamaIndex + Chroma stack to prove value; then converge on the §14 production stack — hybrid retrieval, reranking, RAGAS-gated evaluation, on-prem storage, and SR 11-7/MAS TRM/BCBS 239 governance from day one. The framework is the skeleton; retrieval quality, observability, and governance are the organs — and all three must be designed together.

---

*End of guide. Companion references: [LLM Application Frameworks](../llm_frameworks_comparison_guide.md) · [RAG vs HyDE](rag_vs_hyde_guide.md) · [Query Rewriting in RAG](query_rewriting_rag_guide.md) · [LLM Evaluation Frameworks](../llm_evaluation_frameworks_guide.md) · [Enterprise AI Gateway](../../enterprise_ai_gateway_guide.md).*
