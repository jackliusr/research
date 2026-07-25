# LLM Application Frameworks: Comprehensive Comparison Guide

> **A detailed comparison of 8 major frameworks for building LLM-powered applications,**
> **with particular focus on Neo4j integration — vector stores, knowledge graphs, and graph-based RAG.**
>
> **Author:** Jack Liu Shurui  
> **Category:** AI/LLM Framework Comparison  
> **Part of:** Research / technology / llm-frameworks  
> **Date:** July 2026

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Framework Profiles (A–H)](#2-framework-ah)
   - [A. LangChain (Python)](#a-langchain-python)
   - [B. LangChainJS](#b-langchainjs)
   - [C. LlamaIndex](#c-llamaindex)
   - [D. Spring AI](#d-spring-ai)
   - [E. LangChain4j](#e-langchain4j)
   - [F. Haystack](#f-haystack)
   - [G. Semantic Kernel](#g-semantic-kernel)
   - [H. DSPy](#h-dspy)
3. [Cross-Cutting Comparisons](#3-cross-cutting-comparisons)
   - [3.1 Architecture Approach](#31-architecture-approach)
   - [3.2 RAG Capabilities](#32-rag-capabilities)
   - [3.3 Agent and Tool Support](#33-agent-and-tool-support)
   - [3.4 Neo4j Integration Depth](#34-neo4j-integration-depth)
   - [3.5 Learning Curve](#35-learning-curve)
   - [3.6 Production Readiness](#36-production-readiness)
   - [3.7 Ecosystem Size](#37-ecosystem-size)
4. [Selection Guide](#4-selection-guide)
   - [4.1 Decision Matrix](#41-decision-matrix)
   - [4.2 Decision Tree](#42-decision-tree)
   - [4.3 Framework-to-Scenario Mapping](#43-framework-to-scenario-mapping)

---

## 1. Executive Summary

The LLM application framework landscape has matured dramatically through 2025–2026. Eight
frameworks dominate the conversation, each with distinct architectural philosophies, language
ecosystems, and integration depth with graph databases — particularly Neo4j. This guide provides:

- **Detailed profiles** of each framework covering origin, architecture, features, Neo4j
  integration, strengths, and limitations.
- **Cross-cutting comparisons** across 8 dimensions: architecture, RAG capabilities, agent
  support, Neo4j depth, learning curve, production readiness, ecosystem size, and best
  use cases.
- **A selection guide** with decision matrices and a decision tree to help choose the right
  framework based on language preference, use case, Neo4j needs, production maturity, and
  team expertise.

### Why Neo4j Integration Matters

Neo4j is the leading graph database in the GenAI ecosystem. Its native vector index (HNSW-based),
property graph model, and Cypher query language make it uniquely suited for:

- **Vector search** alongside graph traversal (hybrid retrieval)
- **Knowledge graph construction** from unstructured text using LLMs
- **Graph-based RAG** — retrieving structured graph context alongside vector-similar documents
- **Document graph** representations of enterprise content
- **GraphCypherQA** — natural-language-to-Cypher for graph database querying

All 8 frameworks have some level of Neo4j integration, but the depth varies enormously —
from basic vector store connectors (Semantic Kernel, DSPy) to full-spectrum graph RAG
ecosystems (LangChain, LlamaIndex).

---

## 2. Framework Profiles

---

### A. LangChain (Python)

**"The agent engineering platform"**

| Attribute | Detail |
|-----------|--------|
| Creator | Harrison Chase / LangChain Inc. |
| GitHub Stars | ~130,000+ |
| License | MIT |
| First Release | Oct 2022 |
| Stable Version | LangChain 1.0 / LangGraph 1.0 (2025) |
| Primary Language | Python |
| Package | `langchain`, `langchain-neo4j`, `langgraph` |

#### Origin and Community

LangChain is the largest and most widely adopted LLM framework by a significant margin.
Created by Harrison Chase in late 2022 and backed by LangChain Inc. ($25M+ raised),
it has ~3,800+ contributors and is used by 275,000+ downstream projects. The 1.0 release
(2025) marked a stability milestone. Commercial offerings include LangSmith (observability)
and LangGraph Cloud (hosted agents).

#### Core Architecture and Philosophy

LangChain models LLM interactions as **composable chains** — sequences of prompt templates,
LLM calls, tool invocations, and data transformations. LangGraph extends this with
**stateful directed graphs** supporting cycles, state persistence, and human-in-the-loop.
Key abstractions:

- **Runnables** — Unified `.invoke()`, `.stream()`, `.batch()` interface for chains,
  retrievers, and agents.
- **LangGraph** — Cyclic agent workflows with state persistence, branching, checkpointing.
- **AgentExecutor** — Classic think → act → observe → repeat loop.
- **Tool calling** — Native OpenAI/Anthropic tool calling + `@tool` decorator.

#### Key Features

- **Agents:** ReAct, OpenAI function-calling, Structured Chat, custom agents via LangGraph
- **Tools:** 700+ integrations; `@tool` decorator; MCP (Model Context Protocol) support
- **RAG:** Full pipeline — document loaders (160+), text splitters, embedding models,
  vector stores, retrievers, multi-query, self-query, contextual compression
- **Chains:** Legacy `Chain` API (deprecated but still functional), modern `LCEL`
  (LangChain Expression Language) pipelines, `RunnableSequence`, `RunnableParallel`
- **Streaming:** Full token-by-token streaming via `StreamingCallbackHandler` and
  LangGraph streaming modes (`updates`, `events`, `messages`)
- **Multi-modal:** Support for vision models (GPT-4V, Claude 3 Vision), audio transcription,
  image generation
- **Function calling:** First-class support via `bind_tools()`, `with_structured_output()`
- **LangGraph Studio:** Visual debugger for agent workflows
- **LangSmith:** Production observability, tracing, evaluation, datasets

#### Neo4j Integration Depth — ★★★★★ (5/5)

LangChain has the deepest and broadest Neo4j integration of any framework, formalized
in the dedicated `langchain-neo4j` integration package:

| Integration | Capability |
|-------------|------------|
| **Vector Store** | `Neo4jVector` class — full CRUD, similarity search, max-marginal-relevance (MMR), metadata filtering |
| **Knowledge Graph** | `LLMGraphTransformer` — extract entities and relationships from text, construct a knowledge graph in Neo4j |
| **Document Graph** | `Neo4jDocumentGraph` — store document chunks with rich graph metadata linking them |
| **GraphCypherQAChain** | Natural language → Cypher query generation, execution, and answer synthesis |
| **GraphRAG Pipeline** | Hybrid retrieval — combine vector search results with graph traversal context |
| **Checkpoint Saver** | LangGraph state persistence using Neo4j (time-travel, branching) |
| **Full-text Search** | Neo4j full-text indexes via `Neo4jVector` |
| **Community Examples** | Neo4j Labs starter kit, Tomaz Bratanic's extensive blog series |

The `LLMGraphTransformer` is particularly notable — it uses an LLM to extract a labeled
property graph from unstructured documents and stores entities as nodes, relationships as
edges directly into Neo4j. The `GraphCypherQAChain` generates Cypher queries from natural
language, executes them, and passes results through an LLM for final answer generation.

#### Strengths

- Largest ecosystem and deepest Neo4j integration
- LangGraph enables production-grade agent workflows with state persistence
- LangSmith offers best-in-class observability
- 50+ model providers, 700+ integrations

#### Limitations

- Complexity — the massive surface area can overwhelm newcomers
- Python-first; LangChainJS lags in features
- LangGraph state model has a steep initial curve
- Debugging is hard without LangSmith

#### Learning Curve and Documentation

- **Learning curve:** Moderate-to-steep. Easy to start (5-line RAG app), but mastering
  LangGraph and production patterns requires significant investment.
- **Documentation:** Excellent — well-organized cookbooks, API reference, and integration
  guides. Neo4j integration has dedicated docs and community blog posts.

---

### B. LangChainJS

**"LangChain for the JavaScript/TypeScript ecosystem"**

| Attribute | Detail |
|-----------|--------|
| Creator | LangChain Inc. |
| GitHub Stars | Shared with LangChain org |
| License | MIT |
| First Release | Early 2023 |
| Primary Language | TypeScript |
| Package | `langchain`, `@langchain/community` |

#### Origin and Community

LangChainJS is the TypeScript port of LangChain maintained by the same LangChain Inc. team.
It targets Node.js server-side and browser-based applications. While the Python version is
more mature and has more integrations, LangChainJS is the most widely used TS/JS LLM
framework. It follows the Python version's architecture but with TypeScript idioms.

#### Core Architecture and Philosophy

Same as Python — Runnables, LCEL, Agents, Chains — but with some caveats:
- LangGraphJS is available but less feature-complete than the Python version
- Some advanced features (graph checkpointing, complex state) trail Python
- The `@langchain/core` package provides base abstractions as a separate dependency
- Compatible with Vercel Edge Functions and Cloudflare Workers

#### Neo4j Integration Depth — ★★★☆☆ (3/5)

| Integration | Capability |
|-------------|------------|
| **Vector Store** | `Neo4jVectorStore` — embeddings storage, similarity search, metadata filtering |
| **Graph QA** | `GraphCypherQAChain` ported to TypeScript |
| **Knowledge Graph** | No `LLMGraphTransformer` equivalent |
| **Document Graph** | Not available |
| **LangGraph Checkpoint** | Not available in JS |

The JS integration covers the most common use cases (vector search and graph QA) but
lacks the advanced graph construction and document graph features of the Python version.

#### Strengths

- Only mature TS/JS LLM framework with Neo4j support
- Edge-compatible — runs on Vercel, Cloudflare, Netlify
- TypeScript-first with strong typing via `zod`
- Familiar API for Node.js developers

#### Limitations

- Features trail Python by 3–9 months
- LangGraphJS is significantly less mature than LangGraph (Python)
- Smaller community and fewer third-party tutorials
- Neo4j integration is limited to vector store + basic Cypher QA
- Edge runtime constraints limit some features

#### Learning Curve and Documentation

- **Learning curve:** Moderate. Familiarity with TypeScript helps; Neo4j-specific docs are
  available but sparse compared to Python.
- **Documentation:** Good. The LangChain docs site includes JS examples alongside Python.
  The Neo4j JS-specific documentation is adequate but thinner.

---

### C. LlamaIndex

**"The data framework for LLM applications"**

| Attribute | Detail |
|-----------|--------|
| Creator | Jerry Liu / LlamaIndex Inc. (formerly GPT Index) |
| GitHub Stars | ~51,000 |
| License | MIT |
| First Release | Nov 2022 |
| Stable Version | 0.12.x (continual iteration) |
| Primary Language | Python |
| Package | `llama-index`, `llama-index-graph-neo4j` |

#### Origin and Community

LlamaIndex began as "GPT Index" in late 2022, created by Jerry Liu. It has grown to ~51K
GitHub stars with 7.3M monthly PyPI downloads. LlamaIndex Inc. offers LlamaCloud (hosted
indexing) and LlamaParse (document parsing). The community is research-forward — many RAG
innovations (agentic RAG, structured extraction, knowledge graph indexing) originate here.

#### Core Architecture and Philosophy

LlamaIndex is **data-centric** — its fundamental abstraction is the `Index`, which ingests
documents, chunks them, extracts metadata, generates embeddings, and builds a
queryable structure. Key abstractions:

- **Documents / Nodes** — `Document` is the raw input; `Node` is a parsed chunk + metadata.
- **Indices** — `VectorStoreIndex`, `SummaryIndex`, `KnowledgeGraphIndex`,
  `PropertyGraphIndex`
- **Retrievers** — `VectorIndexRetriever`, `KnowledgeGraphRAGRetriever`, custom retrievers
- **Query Engines** — Combines retriever + response synthesizer
- **Chat Engines** — Multi-turn conversational interfaces
- **Workflows** — Event-driven DAG-based orchestration layer (2024+)
- **Agents** — `AgentRunner` / `ReActAgent` with tool calling

#### Key Features

- **RAG:** Comprehensive — ingestion pipeline, chunking, embedding, dense/sparse/hybrid
  retrieval, re-ranking, query routing
- **Knowledge Graph Index:** Build KGs from documents via LLM-extracted entities/relations
- **PropertyGraphIndex:** Modern property graph index with Neo4j backend
- **Structured Extraction:** `PydanticProgram` / `LLMProgram` for structured output
- **Agentic RAG:** `RouterQueryEngine` routes queries to different indices/retrievers
- **Workflows:** Event-driven DAG orchestration for complex pipelines
- **Multi-modal:** Image docs, vision models, multi-modal embeddings
- **Streaming:** Token-by-token streaming on queries and chat

#### Neo4j Integration Depth — ★★★★★ (5/5)

LlamaIndex offers the second-deepest Neo4j integration after LangChain:

| Integration | Capability |
|-------------|------------|
| **Vector Store** | `Neo4jVectorIndex` — embeddings, similarity search, metadata filtering |
| **Knowledge Graph Index** | `KnowledgeGraphIndex` — extract entities/relationships into Neo4j KG |
| **Property Graph Index** | `PropertyGraphIndex` (with `Neo4jPropertyGraphStore`) — modern property graph support |
| **Graph RAG Retriever** | `KnowledgeGraphRAGRetriever` — traverse KG context to enrich RAG results |
| **Text2Cypher** | Natural language → Cypher generation with LLM error recovery |
| **Graph Schema Integration** | Auto-detect Neo4j graph schema for improved Cypher generation |
| **LlamaCloud** | Hosted indexing into Neo4j for enterprise deployments |
| **Agentic KG Agents** | Multi-step Text2Cypher with retry loops using LlamaIndex Workflows |

The `PropertyGraphIndex` is a standout — it defines a typed graph schema, extracts structured
data via LLM, and stores/retrieves from Neo4j. `KnowledgeGraphRAGRetriever` traverses the
graph from seed entities found in the query, collecting graph context before synthesizing
answers.

#### Strengths

- Best-in-class data ingestion and indexing pipeline
- Deepest knowledge graph integration alongside LangChain
- `PropertyGraphIndex` is the most sophisticated graph index available
- Strong structured output / Pydantic extraction support
- Research-forward — new RAG patterns land here first

#### Limitations

- Agent capabilities are simpler than LangGraph (no cyclic graphs, limited state persistence)
- API surface is large and can be confusing (multiple ways to do the same thing)
- Documentation sometimes trails the code (fast-moving framework)
- Index-centric thinking can feel odd for non-RAG applications

#### Learning Curve and Documentation

- **Learning curve:** Moderate. Index abstraction is intuitive for data-heavy tasks, but the
  number of index types, retrievers, and query engines creates confusion.
- **Documentation:** Very good — hundreds of examples, cookbooks, and a dedicated Neo4j
  starter kit.

---

### D. Spring AI

**"Enterprise AI for the Spring ecosystem"**

| Attribute | Detail |
|-----------|--------|
| Creator | Spring / VMware (now Broadcom) |
| GitHub Stars | ~8,000 |
| License | Apache 2.0 |
| First Release | 2023 (incubation), 1.0 GA 2025 |
| Primary Language | Java |
| Package | `spring-ai-core`, `spring-ai-neo4j-store` |

#### Origin and Community

Spring AI is the official AI framework from the Spring team at VMware/Broadcom. It reached
1.0 GA in 2025 with ~8K GitHub stars and 2.7K forks, making it the fastest-growing Java
AI framework. It leverages the Spring ecosystem's maturity — dependency injection,
auto-configuration, and production-ready patterns (metrics, health checks, retry).

#### Core Architecture and Philosophy

Spring AI is built on **Spring's template pattern** with abstractions familiar to Spring
developers:

- **AiClient / ChatClient** — Primary LLM interface (similar to `JdbcTemplate`)
- **Prompt** — Input message abstraction with system/user/assistant roles
- **EmbeddingClient** — Interface for embedding models
- **VectorStore** — Abstraction over vector databases
- **ToolCalling** — Function calling via `@Tool` annotation
- **Advisors** — AOP-style interceptors for prompts (context injection, safety checks)
- **Pipeline** — Simple ETL chain for RAG ingestion
- **MCP Support** — Model Context Protocol client/server (1.0 GA)

#### Key Features

- **ChatClient:** Fluent API for chat completions with streaming, tool calling, and vision
- **RAG:** `DocumentReader` → `DocumentTransformer` → `DocumentWriter` pipeline pattern
- **Vector Store:** Neo4j, Pinecone, Weaviate, Chroma, Qdrant, Milvus, PGVector
- **Function Calling:** `@Tool` annotation-based, Spring Bean integration
- **Streaming:** Full `Flux`-based streaming (Project Reactor)
- **Multi-modal:** Image inputs in chat, multi-modal embedding clients
- **Observability:** Native Micrometer metrics, Spring Actuator, distributed tracing
- **Auto-configuration:** Spring Boot starter for all integrations
- **ETL Pipeline:** `DocumentReader` → `DocumentTransformer` → `DocumentWriter` for
  RAG ingestion
- **Advisors:** Prompt enrichment, context injection, auditing

#### Neo4j Integration Depth — ★★★★☆ (4/5)

| Integration | Capability |
|-------------|------------|
| **Vector Store** | `Neo4jVectorStore` — HNSW index, similarity search, filters |
| **Knowledge Graph** | `Neo4jGraphStore` — query Neo4j as a graph source for RAG |
| **Document Graph** | Via `Neo4jTemplate` — generic Neo4j OGM for document metadata |
| **GraphCypherQA** | Build your own via `ChatClient` + `Neo4jTemplate` (no dedicated chain) |
| **Spring Data Neo4j** | Full compatibility — use SDN entities alongside AI abstractions |
| **Auto-configuration** | `spring.ai.neo4j.*` properties in `application.yml` |

Spring AI does not have a dedicated `GraphCypherQAChain` equivalent — you build it
yourself by combining `ChatClient` with `Neo4jTemplate` to generate and execute Cypher.
However, the Spring Data Neo4j integration means existing Spring Boot applications with
Neo4j can add AI features with minimal friction. The vector store integration is solid
and production-tested.

#### Strengths

- Natural fit for Spring Boot / Java enterprise shops
- Production-ready patterns built in (retry, circuit-breaker, metrics)
- Auto-configuration reduces boilerplate dramatically
- Spring Data Neo4j compatibility for existing Neo4j users

#### Limitations

- Smaller ecosystem than LangChain or LlamaIndex
- No dedicated graph QA chain — must build Cypher generation from scratch
- Agent/multi-agent support is nascent
- Documentation is improving but trails Python frameworks

#### Learning Curve and Documentation

- **Learning curve:** Low-to-moderate for Spring Boot developers. The abstractions mirror
  Spring Data patterns. Non-Spring Java developers will need to learn Spring basics first.
- **Documentation:** Good. Official Spring AI reference docs are thorough. Fewer
  community tutorials compared to Python frameworks. Neo4j-specific docs exist but
  are minimal.

---

### E. LangChain4j

**"LangChain for the JVM ecosystem"**

| Attribute | Detail |
|-----------|--------|
| Creator | Community (maintained by langchain4j org) |
| GitHub Stars | ~12,400 |
| License | Apache 2.0 |
| First Release | 2023 |
| Primary Languages | Java, Kotlin |
| Package | `langchain4j`, `langchain4j-neo4j` |

#### Origin and Community

LangChain4j is a community-driven Java port of LangChain that has grown significantly
faster than expected. With 12.4K stars and 427 contributors, it has become the second
most popular Java AI framework (after Spring AI). It is not officially affiliated with
LangChain Inc. but follows the LangChain design philosophy. It integrates with both
Spring Boot and Quarkus for enterprise deployment.

#### Core Architecture and Philosophy

LangChain4j follows the LangChain pattern but adapted for Java idioms:

- **Chain / Chain abstraction** — `Chain<T>` interface with input/output typing
- **LLM abstraction** — `ChatLanguageModel`, `StreamingChatLanguageModel`,
  `LanguageModel`, `EmbeddingModel`
- **Tool/Function Calling** — `@Tool` annotation on Java methods
- **RAG Components** — `ContentRetriever`, `ContentAggregator`, `Document`, `DocumentSplitter`
- **AiServices** — High-level service that generates an implementation from an interface,
  binding tools and RAG components
- **Memory** — `ChatMemory` for conversational context
- **Agent** — Basic agent loops via `AiServices` with tools

#### Key Features

- **AiServices:** Interface → Implementation generation (the hallmark feature)
- **RAG:** `ContentRetriever` interface, document loaders, splitters, embedding stores
- **Tool Calling:** `@Tool` annotation on any Java method
- **Streaming:** `StreamingChatLanguageModel` for token-by-token streaming
- **Function Calling:** OpenAI-compatible function calling
- **MCP Support:** Added in 2025
- **Spring Boot / Quarkus integration:** Auto-configuration, `@Inject` support
- **Multiple vector stores:** Neo4j, Pinecone, Chroma, Qdrant, Milvus, Weaviate
- **Structured Output:** `StructuredOutputParser` for JSON/POJO extraction

#### Neo4j Integration Depth — ★★★☆☆ (3/5)

| Integration | Capability |
|-------------|------------|
| **Vector Store** | `Neo4jEmbeddingStore` — embeddings storage, similarity search, filters |
| **Knowledge Graph** | Limited — community extensions exist |
| **Document Graph** | Not available |
| **GraphCypherQA** | No dedicated chain; build via tools |
| **Graph RAG** | Via community tools and custom content retrievers |

Neo4j integration covers the vector store use case well, but deeper graph features
(knowledge graph construction, GraphCypherQA, property graph indexing) are not available
out of the box. Developers can build these by writing custom Java classes that use the
Neo4j Java Driver directly.

#### Strengths

- Java-native with minimal framework baggage (no Spring required)
- Works with Quarkus (GraalVM native-image) for low-footprint deployments
- `AiServices` pattern is elegant — define a Java interface, get an LLM-backed implementation
- Good Spring Boot integration for enterprise shops
- Well-designed for Kotlin as well
- Active community with frequent releases

#### Limitations

- No LangGraph equivalent — no stateful agent workflows or cyclic graphs
- Smaller ecosystem than Python LangChain
- Neo4j integration is limited to vector storage
- Documentation is decent but has gaps (some features documented only in tests)
- Community is smaller than Spring AI for Java
- Less production battle-testing than Spring AI
- Some abstractions feel forced into Java patterns unsuited to async/streaming

#### Learning Curve and Documentation

- **Learning curve:** Moderate. Familiarity with LangChain concepts helps. Java developers
  will find `AiServices` intuitive. Graph-specific features require custom implementation.
- **Documentation:** Good-to-adequate. The LangChain4j docs site is well-organized. Neo4j
  integration docs are basic. Community examples on GitHub fill some gaps.

---

### F. Haystack

**"Production-ready LLM orchestration framework"**

| Attribute | Detail |
|-----------|--------|
| Creator | deepset GmbH |
| GitHub Stars | ~26,000 |
| License | Apache 2.0 |
| First Release | 2021 (v1), v2 rewrite 2024 |
| Stable Version | 2.31.x (July 2026) |
| Primary Language | Python |
| Package | `haystack-ai`, `neo4j-haystack` |

#### Origin and Community

Haystack was created by deepset (Berlin-based), originally for search and QA with BERT
models. It was rewritten as Haystack 2.0 in 2024 to embrace LLMs. With 26K GitHub stars
and 1M+ monthly PyPI downloads, it has strong European enterprise adoption. deepset also
offers Haystack Enterprise Platform (formerly deepset Cloud).

#### Core Architecture and Philosophy

Haystack's architecture is **pipeline-centric** — typed component graphs with explicit data
flow:

- **Component** — Typed processing node with input/output sockets
- **Pipeline** — DAG of connected components
- **DocumentStore** — Abstraction for storing/retrieving documents
- **Retriever** — Dense (embedding), sparse (BM25), or hybrid retrieval
- **Agent** (2025+) — `Tool` + `Agent` + `ChatAgent` components
- **Async** — Native asyncio throughout (v2.0+)

#### Key Features

- **Pipelines:** Explicit typed component wiring — easy to reason about
- **RAG:** Full ingestion → preprocessing → indexing → retrieval → generation pipeline
- **Hybrid Search:** Dense + sparse retrieval with weighting
- **Ranking:** Re-ranking via Cohere, cross-encoders, sentence-transformers
- **Document Stores:** Neo4j, Elasticsearch, OpenSearch, Qdrant, Weaviate, Chroma, Pinecone
- **Agents:** `Tool` + `Agent` + `ChatAgent` (newer, less mature)
- **Evaluation:** Built-in metrics (SAS, faithfulness, etc.)
- **Async:** Native pipeline execution in async mode

#### Neo4j Integration Depth — ★★★★☆ (4/5)

| Integration | Capability |
|-------------|------------|
| **Vector Store** | `Neo4jDocumentStore` — embeddings storage, dense retrieval, filters |
| **Hybrid Search** | Neo4j vector + full-text index combined retrieval |
| **Document Store** | Full `DocumentStore` protocol — write, filter, delete documents |
| **Knowledge Graph** | Via custom components (no dedicated KG index) |
| **GraphCypherQA** | Via `Neo4jClient` component |
| **Graph RAG** | Build custom pipelines combining Neo4j retrieval with graph traversal |

Haystack's Neo4j integration is noteworthy for its full `DocumentStore` protocol
implementation — you can use Neo4j as the primary document store for all your Haystack
pipelines. The hybrid search capability (combining Neo4j vector search with full-text
indexes) is unique among these frameworks. However, there is no dedicated knowledge
graph construction or property graph index — those require custom components.

#### Strengths

- Best-in-class pipeline visualization and reasoning
- Excellent hybrid search support with Neo4j
- Explicit, typed component architecture is great for debugging and testing
- Strong European enterprise adoption
- DocumentStore protocol is clean and well-designed
- Evaluation pipeline is built-in, not an add-on
- Good async support

#### Limitations

- Agent support is newer and less capable than LangGraph or LlamaIndex agents
- Pipeline DAG model can be rigid for complex non-linear workflows
- Smaller integration library compared to LangChain
- Neo4j graph construction / KG features require custom development
- Community is smaller than LangChain or LlamaIndex
- Learning curve for the Pipeline/Component model if coming from LangChain

#### Learning Curve and Documentation

- **Learning curve:** Low-to-moderate. The Pipeline model is very visual and
  component-oriented. Developers with background in data pipelines will find it intuitive.
- **Documentation:** Excellent. deepset produces some of the best documentation in the
  LLM ecosystem — clear, well-organized, with interactive tutorials. Neo4j integration
  docs are good with concrete examples.

---

### G. Semantic Kernel

**"Microsoft's AI orchestration SDK"**

| Attribute | Detail |
|-----------|--------|
| Creator | Microsoft |
| GitHub Stars | ~28,000 |
| License | MIT |
| First Release | Feb 2023 |
| Stable Version | 1.x (GA as Microsoft Agent Framework 2025) |
| Primary Languages | C#, Python, Java |
| Package | `Microsoft.SemanticKernel` (NuGet), `semantic-kernel` (PyPI) |

#### Origin and Community

Semantic Kernel is Microsoft's open-source AI orchestration framework, initially released
in early 2023. In 2025, Microsoft rebranded it as the **Microsoft Agent Framework**,
reaching GA and integrating deeply with Azure AI Foundry. With ~28K GitHub stars, it's the
primary AI framework for the .NET ecosystem, positioned for agents embedded in existing
enterprise applications rather than as a standalone runtime.

#### Core Architecture and Philosophy

Built on **orchestration** — designed to be embedded in existing apps:
- **Kernel** — Central orchestrator holding AI services, memory, and plugins
- **Plugins** — Units of functionality (native code or semantic prompts) exposed to LLMs
- **Memory** — Vector storage abstraction for semantic memory and retrieval
- **Planner** — LLM-driven plan generation (auto-decomposition of tasks into steps)
- **Agents** — `ChatCompletionAgent`, `AgentGroupChat` for multi-agent
- **Auto Function Calling** — Automatic selection/invocation of kernel functions by the LLM
- **Filters** — Middleware-style interception for prompts, responses, function calls

#### Key Features

- **Agents:** `ChatCompletionAgent`, `AgentGroupChat` (multi-agent), `OpenAIAssistantAgent`
- **Plugins:** Semantic (prompt-based) and native (code-based) plugins
- **Memory:** Vector store abstraction for embeddings and semantic search
- **Planner:** `HandlebarsPlanner`, `FunctionCallingStepwisePlanner`
- **RAG:** Via memory stores — embed, retrieve, inject context into prompts
- **Auto Function Calling:** Seamless OpenAI tool calling integration
- **Multi-modal:** Vision support, audio via Azure AI Speech
- **Streaming:** Async streaming of LLM responses
- **Enterprise:** Azure OpenAI, Azure AI Search, Entra auth, responsible AI filters

#### Neo4j Integration Depth — ★★☆☆☆ (2/5)

| Integration | Capability |
|-------------|------------|
| **Vector Store** | Community connector available (not official SK connector) |
| **Knowledge Graph** | No official integration |
| **Document Graph** | No official integration |
| **GraphCypherQA** | No official integration |
| **Semantic Memory** | Memory connector pattern could point to Neo4j (community) |
| **Vector Store Connectors** | Azure AI Search, Chroma, Qdrant, Pinecone, Elasticsearch (no Neo4j in main repo) |

Semantic Kernel's official vector store connectors do not include Neo4j. Community-maintained
connectors exist (via `Microsoft.SemanticKernel.Connectors.Neo4j` and third-party packages),
but none of the deeper graph features (KG construction, graph RAG, Cypher generation) are
available. For Neo4j-centric workloads, Semantic Kernel requires building custom memory
connectors using the Neo4j .NET/C# driver.

#### Strengths

- Best choice for .NET / C# ecosystem (no other .NET LLM framework comes close)
- Deep Azure integration for enterprise customers
- Agent Framework (GA 2025) provides robust multi-agent orchestration
- Kernel/Plugin architecture is clean and modular
- Enterprise-grade security and authentication (Microsoft Entra)
- Strong governance and responsible AI tooling
- Filters pattern enables cross-cutting concerns (logging, safety, telemetry)

#### Limitations

- Minimal Neo4j integration — no official connector, no graph features
- Smaller plugin/integration ecosystem than LangChain
- Planner approach can be unreliable for complex multi-step tasks
- Python/Java SDKs lag behind C# in features and maturity
- Documentation is Microsoft-typical (extensive but sometimes hard to navigate)
- Community is smaller outside .NET ecosystem
- Azure bias — non-Azure LLM providers have less first-class support

#### Learning Curve and Documentation

- **Learning curve:** Moderate for C# developers familiar with Microsoft ecosystems.
  Kernel/Plugin/Filters concepts are well-designed. Python/Java versions have thinner docs.
  Neo4j users will need to build custom connectors.
- **Documentation:** Good. Microsoft Learn documentation is thorough with tutorials.
  The sheer volume can be overwhelming. Neo4j documentation is essentially absent.

---

### H. DSPy

**"Programming — not prompting — language models"**

| Attribute | Detail |
|-----------|--------|
| Creator | Stanford NLP (Omar Khattab, Christopher Potts, etc.) |
| GitHub Stars | ~32,000 |
| License | Apache 2.0 |
| First Release | Late 2023 |
| Stable Version | 3.2.x (May 2026) |
| Primary Language | Python |
| Package | `dspy-ai` |

#### Origin and Community

DSPy (Declarative Self-improving Python) emerged from Stanford NLP research by Omar Khattab
and Prof. Christopher Potts. It pioneered **programming LLMs through optimization** rather
than prompt engineering. DSPy 3.0 (2025) added MLflow tracing, Adabelief, and improved
optimizers. With 32K stars, it has a strong research community following.

#### Core Architecture and Philosophy

DSPy flips the traditional model on its head — you define **signatures** (typed I/O schemas)
and **modules** (reusable building blocks), then **compile** the program with an optimizer:

- **Signature** — Typed input/output spec (e.g., `question -> answer: str`)
- **Module** — Building block: `dspy.Predict`, `dspy.ChainOfThought`, custom classes
- **Program** — Python class combining modules into a pipeline
- **Optimizer (Teleprompter)** — Auto-tunes prompts, few-shot examples, or model weights
- **Compile** — Optimization step using training data
- **Retrieval Model** — `dspy.Retrieve`, `Neo4jRM` for RAG
- **LM** — Abstraction over LLM providers

#### Key Features

- **Signatures:** Declarative typed I/O specs — no manual prompt writing
- **Optimizers:** MIPROv2, BootstrapFewShot, BSO, COPRO, LabeledFewShot
- **Modules:** `dspy.Predict`, `dspy.ChainOfThought`, `dspy.ReAct`, `dspy.MultiChainComparison`
- **Compile-time optimization:** Automatic prompt engineering, few-shot selection
- **RAG:** `dspy.Retrieve`, `Neo4jRM` for optimizable retrieval pipelines
- **Agentic:** `ReAct` module (Thought → Act → Observe)
- **Tracing:** MLflow integration (DSPy 3.0+)

#### Neo4j Integration Depth — ★★★☆☆ (3/5)

| Integration | Capability |
|-------------|------------|
| **Vector Store (RM)** | `Neo4jRM` — retriever module for vector search over Neo4j |
| **Knowledge Graph** | Community projects only (e.g., `dspy-neo4j-knowledge-graph`) |
| **Document Graph** | Not available |
| **GraphCypherQA** | Not available |
| **Graph RAG** | Build via `Neo4jRM` + custom modules |
| **Optimized RAG** | Combine `Neo4jRM` with DSPy optimizers for auto-tuned retrieval pipelines |

DSPy's Neo4j integration is focused on the retriever pattern via `Neo4jRM`, which wraps
Neo4j vector search as a DSPy retriever module. This allows DSPy to optimize the retrieval
step as part of an overall RAG pipeline compilation. Deeper graph features (knowledge
graph construction, graph traversal as part of retrieval, Cypher generation) are not
available out of the box — they require community extensions or custom modules. The
`dspy-neo4j-knowledge-graph` community project demonstrates automated KG construction
using DSPy but is not part of the official framework.

#### Strengths

- Revolutionary approach: prompts are *learned*, not *written*
- MIPROv2 optimizer produces high-quality prompts with minimal human effort
- Well-suited for teams doing systematic LLM evaluation and iteration
- Signature-based design is elegant and maintainable
- Strong research backing from Stanford
- MLflow integration makes tracing and evaluation production-ready
- Excellent for RAG pipeline optimization

#### Limitations

- Not a general-purpose application framework — no agents (beyond ReAct), no chat memory
- No production-grade features (no built-in chat memory, no state persistence)
- Steep learning curve — the compile/optimize model requires a mindset shift
- Smaller community than LangChain or LlamaIndex
- Neo4j integration limited to vector retrieval only
- Documentation is more research-paper style than developer docs
- Compile step requires labeled training data (or synthetic data generation)
- Less suitable for real-time conversational applications

#### Learning Curve and Documentation

- **Learning curve:** High. The compile-time optimization model is unlike conventional
  frameworks. Developers comfortable with ML training loops will adapt faster. The
  signature/module abstraction is elegant but unfamiliar.
- **Documentation:** Good-for-academic. The dspy.ai website is clean but sparser than
  LangChain or LlamaIndex docs. The Stanford paper and Jupyter notebook examples are
  the primary learning resources. Community cookbooks fill some gaps.

---

## 3. Cross-Cutting Comparisons

---

### 3.1 Architecture Approach

| Framework | Paradigm | Core Abstraction | Pipeline Model |
|-----------|----------|-----------------|----------------|
| LangChain | Chain + Graph | Runnable, LangGraph | LCEL chain or cyclic graph |
| LangChainJS | Chain + Graph | Runnable (subset) | LCEL chain or basic graph |
| LlamaIndex | Data-centric Index | Index / QueryEngine | DAG (Workflows) |
| Spring AI | Template/Service | ChatClient, VectorStore | ETL Pipeline (DocumentReader → DocumentTransformer → DocumentWriter) |
| LangChain4j | Chain + Service | AiServices, Chain | Linear chains |
| Haystack | Pipeline DAG | Component / Pipeline | Explicit typed DAG |
| Semantic Kernel | Orchestration | Kernel / Plugin / Memory | Planner-generated or manual |
| DSPy | Compile-time Optimization | Signature / Module / Optimizer | Python class compiled via optimizer |

**Key observations:**

- **Chain-based** (LangChain, LangChainJS, LangChain4j): Linear composable steps, extended
  with cyclic graphs for agents.
- **Graph/pipeline-based** (Haystack, LangGraph): Explicit DAGs with typed data flow.
- **Data-centric** (LlamaIndex): Index-first — data ingestion drives application design.
- **Optimization-heavy** (DSPy): Compile-time prompt/weight optimization over runtime flexibility.
- **Orchestration** (Semantic Kernel, Spring AI): Embed AI into existing app architecture.

---

### 3.2 RAG Capabilities

| Capability | LangChain | LangChainJS | LlamaIndex | Spring AI | LChain4j | Haystack | SK | DSPy |
|------------|:---------:|:-----------:|:----------:|:---------:|:--------:|:--------:|:--:|:----:|
| **Basic RAG** (retrieve → generate) | ★★★★★ | ★★★★ | ★★★★★ | ★★★★ | ★★★★ | ★★★★★ | ★★★ | ★★★★ |
| **Advanced RAG: Query Routing** | ★★★★★ | ★★★ | ★★★★★ | ★★ | ★★ | ★★★★ | ★★ | ★ |
| **Advanced RAG: Query Construction** | ★★★★ | ★★ | ★★★★★ | ★★ | ★★★ | ★★★ | ★ | ★★ |
| **Advanced RAG: Indexing** | ★★★★★ | ★★★ | ★★★★★ | ★★★ | ★★★ | ★★★★★ | ★★★ | ★★ |
| **Advanced RAG: Retrieval Strategies** | ★★★★★ | ★★★★ | ★★★★★ | ★★ | ★★ | ★★★★★ | ★★ | ★★★ |
| **Agentic RAG** | ★★★★★ | ★★★ | ★★★★ | ★ | ★ | ★★★ | ★★★ | ★★ |
| **Hybrid Search** | ★★★★ | ★★★ | ★★★★ | ★★ | ★★ | ★★★★★ | ★★ | ★ |
| **Graph-backed RAG** | ★★★★★ | ★★ | ★★★★★ | ★★★ | ★★ | ★★★★ | ★ | ★★★ |
| **Evaluation** | ★★★★ (LangSmith) | ★★★ | ★★★ | ★★ | ★ | ★★★★ | ★★ | ★★★★★ |

**RAG archetypes:**

- **Basic RAG:** Chunk → embed → retrieve → generate. All frameworks support this.
- **Advanced (routing):** Dynamic selection of retriever/index based on query.
- **Advanced (query construction):** Transform NL to structured queries (Cypher, SQL).
- **Advanced (indexing):** Sophisticated ingestion — hierarchical indices, recursive retrieval.
- **Advanced (retrieval):** Multi-query, self-query, contextual compression, HyDE, fusion.
- **Agentic RAG:** LLM decides which tools/retrievers to call in multi-round loops.
- **Graph-backed RAG:** Enrich vector retrieval with graph traversal results.

---

### 3.3 Agent and Tool Support

| Capability | LangChain | LangChainJS | LlamaIndex | Spring AI | LChain4j | Haystack | SK | DSPy |
|------------|:---------:|:-----------:|:----------:|:---------:|:--------:|:--------:|:--:|:----:|
| **Single Agent** | ★★★★★ | ★★★★ | ★★★★ | ★★★ | ★★★ | ★★★ | ★★★★★ | ★★ |
| **Multi-Agent** | ★★★★★ (LangGraph) | ★★★ | ★★★ (Workflows) | ★ | ★ | ★★ | ★★★★★ (AgentGroupChat) | ★ |
| **Tool Calling** | ★★★★★ | ★★★★ | ★★★★★ | ★★★★ | ★★★★ | ★★★ | ★★★★★ | ★★ |
| **Agentic Loops** | ★★★★★ | ★★★ | ★★★ | ★★ | ★★ | ★★ | ★★★★ | ★★★ (ReAct) |
| **State Persistence** | ★★★★★ (LangGraph) | ★★ | ★★ | ★★ | ★ | ★ | ★★ | ★ |
| **Human-in-the-Loop** | ★★★★★ | ★★★ | ★★ | ★★ | ★ | ★ | ★★★ | ★ |
| **MCP Support** | ★★★★★ | ★★★★ | ★★★ | ★★★ | ★★★ | ★★ | ★★★ | ★★ |

*SK = Semantic Kernel, LChain4j = LangChain4j*

**Notable implementations:**

- **LangChain (LangGraph):** Gold standard — cyclic state graphs, time-travel debugging,
  checkpoint persistence, multi-agent (supervisor + sub-agents).
- **Semantic Kernel (Agent Framework):** GA as Microsoft Agent Framework. Strong multi-agent
  via `AgentGroupChat`. Deep Azure integration.
- **LlamaIndex Workflows:** Event-driven DAG for agentic patterns. Less sophisticated than
  LangGraph but improving rapidly.
- **Haystack:** Agent support is new (2025+). Basic single-agent with tools.
- **Spring AI / LangChain4j:** Basic `@Tool` calling. No multi-agent.
- **DSPy:** ReAct module for basic agentic loops.

---

### 3.4 Neo4j Integration Depth

| Dimension | LangChain | LangChainJS | LlamaIndex | Spring AI | LChain4j | Haystack | SK | DSPy |
|-----------|:---------:|:-----------:|:----------:|:---------:|:--------:|:--------:|:--:|:----:|
| **Vector Store** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ⚠️ (community) | ✅ |
| **Knowledge Graph Construction** | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ⚠️ (community) |
| **Document Graph** | ✅ | ❌ | ✅ | ⚠️ (via SDN) | ❌ | ❌ | ❌ | ❌ |
| **GraphCypherQA** | ✅ | ✅ | ✅ | ⚠️ (DIY) | ❌ | ⚠️ (DIY) | ❌ | ❌ |
| **Graph RAG (hybrid)** | ✅ | ⚠️ | ✅ | ❌ | ❌ | ✅ | ❌ | ⚠️ (DIY) |
| **Property Graph Index** | ⚠️ (via LLMGraph) | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **Checkpoint/State Persistence** | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |

| Framework | Rating | Summary |
|-----------|:------:|---------|
| LangChain | **★★★★★** | Full spectrum — vector, KG construction, GraphCypherQA, document graph, LangGraph checkpoint saver |
| LlamaIndex | **★★★★★** | PropertyGraphIndex, KnowledgeGraphRAGRetriever, KG construction, vector store, Text2Cypher |
| Haystack | **★★★★☆** | Full DocumentStore protocol on Neo4j, hybrid search (vector + full-text), excellent pipeline support |
| Spring AI | **★★★☆☆** | Solid vector store + Spring Data Neo4j compatibility, no Cypher QA or KG construction |
| LangChain4j | **★★★☆☆** | Good vector store integration, no deeper graph features |
| DSPy | **★★★☆☆** | Neo4jRM retriever for optimized RAG, no KG or graph QA features |
| LangChainJS | **★★★☆☆** | Vector store + GraphCypherQA ported, no KG construction or document graph |
| Semantic Kernel | **★★☆☆☆** | No official Neo4j connector; community packages only; no graph features |

---

### 3.5 Learning Curve

```
Beginner-Friendly                          Expert-Only
  Spring AI (Spring devs)
  Haystack
  LangChain (basic) / LangChainJS / LangChain4j
  LlamaIndex / Semantic Kernel (C#)
  LangChain + LangGraph (production agents)
  DSPy / LlamaIndex Workflows
```

| Framework | Learning Curve | Best Audience |
|-----------|:-------------:|---------------|
| Spring AI | Low (Spring devs) | Java/Spring Boot developers |
| Haystack | Low–Moderate | Search/QA teams, pipeline thinkers |
| LangChain (basic) | Moderate | General Python developers |
| LangChainJS | Moderate | TypeScript/Node.js developers |
| LangChain4j | Moderate | Java/Kotlin developers |
| LlamaIndex | Moderate | Python data engineers, RAG specialists |
| Semantic Kernel | Moderate (C#) / Hard (Python/Java) | .NET developers, Azure shops |
| LangChain + LangGraph | Moderate–Hard | Production agent developers |
| DSPy | High | Research teams, optimization-focused devs |
| LlamaIndex Workflows | High | Complex orchestration needs |

---

### 3.6 Production Readiness

| Dimension | LangChain | LangChainJS | LlamaIndex | Spring AI | LChain4j | Haystack | SK | DSPy |
|-----------|:---------:|:-----------:|:----------:|:---------:|:--------:|:--------:|:--:|:----:|
| **Deployment Patterns** | Docker, LangGraph Cloud, K8s | Serverless (Vercel, CF) | Docker, LlamaCloud | Spring Boot JAR, Docker, K8s | JAR, Spring Boot, Quarkus | Docker, Haystack Platform | Azure, Docker, K8s | Docker, MLflow |
| **Observability** | LangSmith, OpenTelemetry | LangSmith | LlamaTrace, OpenTelemetry | Micrometer, Actuator, OpenTelemetry | SLF4J, Logback | Built-in tracing, OpenTelemetry | Azure Monitor, OpenTelemetry | MLflow Tracing |
| **Scaling** | Horizontal (stateless retriever) + stateful (LangGraph) | Horizontal (stateless) | Horizontal (stateless) | Thread pool, reactive (WebFlux) | Thread pool | Horizontal (stateless) | Horizontal (stateless) | Horizontal (batch compile) |
| **Caching** | Built-in cache (LLM, embeddings) | ✅ | ✅ | Spring Cache | ✅ | ✅ | ✅ | ❌ |
| **Rate Limiting** | Via LangSmith / custom | Custom | Custom | Resilience4j | Resilience4j | Custom | Azure rate limiting | ❌ |
| **Retry/Backoff** | ✅ (tenacity) | ✅ | ✅ | ✅ (Spring Retry) | ✅ (resilience4j) | ✅ | ✅ (Polly) | ❌ |
| **Auth/Security** | API key management | API key management | API key management | Spring Security, OAuth2, Entra | Spring Security | API key management | Entra, Managed Identity | API key management |
| **Evaluation** | LangSmith (production eval) | LangSmith | LlamaTrace | Custom | Custom | Built-in eval pipelines | Azure AI Evaluation | MLflow + compile metrics |
| **CI/CD** | LangChain templates | Standard | Standard | Standard Maven/Gradle | Standard Maven/Gradle | Standard | Standard | Standard |
| **Production Maturity** | ★★★★☆ | ★★★☆☆ | ★★★★☆ | ★★★★★ | ★★★☆☆ | ★★★★★ | ★★★★★ | ★★☆☆☆ |

**Observation:** Spring AI has the highest production maturity, inheriting Spring Boot's
health checks, metrics, security, and configuration management. Haystack and Semantic
Kernel are close behind. DSPy is still emerging — MLflow tracing helps but it lacks
caching, rate limiting, and deployment tooling.

---

### 3.7 Ecosystem Size

| Dimension | LangChain | LangChainJS | LlamaIndex | Spring AI | LChain4j | Haystack | SK | DSPy |
|-----------|:---------:|:-----------:|:----------:|:---------:|:--------:|:--------:|:--:|:----:|
| **GitHub Stars** | ~130K | Shared | ~51K | ~8K | ~12.4K | ~26K | ~28K | ~32K |
| **Contributors** | ~3,800 | Shared | ~5,000+ | ~200+ | ~427 | ~500+ | ~600+ | ~400+ |
| **Monthly Downloads** | 10M+ (PyPI) | 1M+ (npm) | 7.3M (PyPI) | Maven: ~500K | Maven: ~300K | 1M+ (PyPI) | 2M+ (NuGet) | ~500K (PyPI) |
| **LLM Provider Integrations** | 50+ | 40+ | 30+ | 10+ | 15+ | 15+ | Azure + 10+ | 10+ |
| **Vector Store Integrations** | 30+ | 20+ | 20+ | 15+ | 12+ | 15+ | 10+ | 5+ |
| **Document Loaders** | 160+ | 80+ | 100+ | 20+ | 15+ | 40+ | 10+ | N/A |
| **Third-Party Tutorials** | Thousands | Hundreds | Hundreds | Growing | Growing | Hundreds | Hundreds | Moderate |
| **Discord/Slack Community** | Very large | Medium | Large | Small | Small | Medium | Medium | Medium |

**Language ecosystem summary:**

| Language | Best Framework(s) | Notes |
|----------|-------------------|-------|
| **Python** | LangChain, LlamaIndex, Haystack, DSPy | Richest ecosystem, most integrations |
| **TypeScript/JS** | LangChainJS | Only mature TS LLM framework with Neo4j |
| **Java** | Spring AI or LangChain4j | Spring AI for production; LangChain4j for versatility |
| **C# / .NET** | Semantic Kernel | Only serious option in .NET ecosystem |
| **Go / Rust / Ruby** | None | No mature LLM frameworks with Neo4j

---

## 4. Selection Guide

---

### 4.1 Decision Matrix

Use this table to identify candidate frameworks based on your primary requirements.

| If you need... | Lead candidate | Also consider |
|----------------|----------------|---------------|
| **Python ecosystem, broadest integration** | **LangChain** | LlamaIndex |
| **Deepest Neo4j integration (KG + vector + Cypher)** | **LangChain** or **LlamaIndex** | Haystack |
| **Production RAG with excellent evaluation** | **Haystack** | LangChain + LangSmith |
| **Agentic workflows with state persistence** | **LangChain + LangGraph** | Semantic Kernel (Agent Framework) |
| **Knowledge graph construction from text** | **LlamaIndex** (PropertyGraphIndex) | LangChain (LLMGraphTransformer) |
| **Java/Spring Boot enterprise deployment** | **Spring AI** | LangChain4j |
| **Lightweight Java (Quarkus, GraalVM)** | **LangChain4j** | Spring AI |
| **.NET / C# ecosystem** | **Semantic Kernel** | — |
| **Serverless / Edge (Vercel, CF Workers)** | **LangChainJS** | — |
| **Optimized RAG pipelines (auto-prompting)** | **DSPy** | LlamaIndex |
| **Search/QA system (not general app)** | **Haystack** | LlamaIndex |
| **Multi-agent systems** | **LangGraph** | Semantic Kernel (Agent Framework) |
| **Minimal boilerplate for Spring devs** | **Spring AI** | — |
| **Budget-constrained (no commercial add-ons)** | **Haystack** | LangChain (OSS) |
| **Graph RAG (vector + graph traversal)** | **LangChain** or **LlamaIndex** | Haystack |
| **Hybrid search (dense + sparse)** | **Haystack (Neo4j)** | LangChain |

---

### 4.2 Decision Tree

```
START HERE
│
├─ What's your primary ecosystem?
│   │
│   ├─ Python ──────────────────────────────────────────────────────────────┐
│   │   │                                                                   │
│   │   ├─ Need general-purpose LLM framework?                             │
│   │   │   ├─ Yes → LangChain (most integrations, LangGraph for agents)   │
│   │   │   └─ No → continue                                                   │
│   │   │                                                                     │
│   │   ├─ Primary use is RAG / data indexing?                                │
│   │   │   ├─ Yes → LlamaIndex (best data-centric RAG)                      │
│   │   │   └─ No → continue                                                  │
│   │   │                                                                     │
│   │   ├─ Building search/QA system?                                         │
│   │   │   ├─ Yes → Haystack (best pipeline visibility + built-in eval)      │
│   │   │   └─ No → continue                                                  │
│   │   │                                                                     │
│   │   ├─ Need to optimize prompts automatically?                            │
│   │   │   ├─ Yes → DSPy (compile-time optimization)                         │
│   │   │   └─ No → continue                                                  │
│   │   │                                                                     │
│   │   └─ Need deep Neo4j integration?                                       │
│   │       ├─ KG construction + Vector + Cypher QA → LangChain or LlamaIndex │
│   │       ├─ DocumentStore + Hybrid Search → Haystack                       │
│   │       └─ Vector-only → Any                                              │
│   │                                                                         │
│   ├─ TypeScript / JavaScript ───────────────────────────────────────────────┘
│   │   │
│   │   └─ LangChainJS (only mature option)
│   │
│   ├─ Java ─────────────────────────────────────────────────────────────────┐
│   │   │                                                                     │
│   │   ├─ Using Spring Boot already?                                         │
│   │   │   ├─ Yes → Spring AI (natural fit, auto-config, production-ready)   │
│   │   │   └─ No → continue                                                  │
│   │   │                                                                     │
│   │   ├─ Using Quarkus / need GraalVM native? → LangChain4j                 │
│   │   ├─ Need maximum flexibility? → LangChain4j                            │
│   │   └─ Need production maturity + Spring Data Neo4j? → Spring AI          │
│   │                                                                         │
│   └─ C# / .NET ─────────────────────────────────────────────────────────────┘
│       │
│       └─ Semantic Kernel (only serious option)
│
└─ Neo4j integration depth required?
    │
    ├─ ★★ (Basic vector store) → Any framework
    ├─ ★★★ (Vector + Cypher QA) → LangChain, LlamaIndex, LangChainJS, Haystack
    ├─ ★★★★ (Vector + KG + Graph RAG) → LangChain, LlamaIndex, Haystack
    └─ ★★★★★ (Full spectrum) → LangChain or LlamaIndex
```

---

### 4.3 Framework-to-Scenario Mapping

| Scenario | Recommended Framework | Why |
|----------|----------------------|-----|
| **Enterprise RAG with Neo4j in Java/Spring** | Spring AI | Spring Data Neo4j compatibility, auto-config, production patterns |
| **Graph RAG PoC with KG construction** | LlamaIndex | PropertyGraphIndex makes KG construction straightforward |
| **Production agent with Neo4j state persistence** | LangChain + LangGraph | LangGraph checkpoint saver on Neo4j is unique |
| **Optimized retrieval pipeline on Neo4j** | DSPy | Compile-time optimization of retrieval + generation |
| **Search engine / QA system on Neo4j** | Haystack | Best pipeline reasoning, built-in evaluation, hybrid search |
| **Multi-agent system with human-in-the-loop** | LangChain + LangGraph | Most mature agent framework with time-travel debugging |
| **Minimal prompt-engineering, systematic eval** | DSPy | Compile-time optimization reduces prompt-tuning effort |
| **Edge/serverless RAG with Neo4j** | LangChainJS | Only mature TS option, Vercel/Cloudflare compatible |
| **Full-stack Neo4j + LLM with .NET backend** | Semantic Kernel | Only .NET option; build custom Neo4j connector |
| **Knowledge graph extraction + query** | LlamaIndex | PropertyGraphIndex + KnowledgeGraphRAGRetriever is unmatched |
| **Cypher QA chatbot** | LangChain | GraphCypherQAChain is the most mature and documented |
| **Quick RAG prototype with Neo4j** | LangChain or LlamaIndex | Both have starter kits and 30-minute setup guides |
| **Enterprise Azure AI with Neo4j** | Semantic Kernel | Azure AI Foundry integration, Entra auth |
| **Document graph with rich metadata** | LlamaIndex or LangChain | Both support storing document chunks as graph nodes |
| **Hybrid search (dense + full-text) on Neo4j** | Haystack | Native Neo4j hybrid search component |

---

### Quick Reference: GitHub Repositories

| Framework | Repository |
|-----------|------------|
| LangChain (Python) | https://github.com/langchain-ai/langchain |
| LangChain Neo4j | https://github.com/langchain-ai/langchain-neo4j |
| LangChainJS | https://github.com/langchain-ai/langchainjs |
| LlamaIndex | https://github.com/run-llama/llama_index |
| Spring AI | https://github.com/spring-projects/spring-ai |
| LangChain4j | https://github.com/langchain4j/langchain4j |
| Haystack | https://github.com/deepset-ai/haystack |
| Semantic Kernel | https://github.com/microsoft/semantic-kernel |
| DSPy | https://github.com/stanfordnlp/dspy |

---

### Quick Reference: Neo4j Integration Packages

| Framework | Package / Connector |
|-----------|---------------------|
| LangChain (Python) | `langchain-neo4j` (PyPI) |
| LangChainJS | `@langchain/community` → `Neo4jVectorStore` |
| LlamaIndex | `llama-index-graph-neo4j` (PyPI) |
| Spring AI | `spring-ai-neo4j-store` (Maven) |
| LangChain4j | `langchain4j-neo4j` (Maven) |
| Haystack | `neo4j-haystack` (PyPI) |
| Semantic Kernel | Community: `Microsoft.SemanticKernel.Connectors.Neo4j` |
| DSPy | `Neo4jRM` (built into `dspy`) |

---

### Key Takeaways

1. **For Neo4j power users**, LangChain and LlamaIndex are the clear leaders — both offer
   the full spectrum from vector search through knowledge graph construction and
   GraphCypherQA. Choose LangChain if agent workflows and stateful graphs matter more;
   choose LlamaIndex if data indexing and property graph management are the priority.

2. **For search/QA pipelines**, Haystack's explicit Pipeline architecture and built-in
   evaluation give it an edge, especially with its hybrid (vector + full-text) Neo4j
   document store.

3. **For enterprise Java/Spring shops**, Spring AI is the natural choice — it inherits
   Spring Boot's production readiness and Spring Data Neo4j compatibility. LangChain4j is
   the alternative for non-Spring JVM projects or Quarkus deployments.

4. **For .NET shops**, Semantic Kernel is the only game in town, but expect to build
   custom Neo4j connectors.

5. **For TypeScript/Edge deployments**, LangChainJS is the only mature option with
   Neo4j support.

6. **For optimization-focused teams**, DSPy's compile-time approach is revolutionary but
   requires a mindset shift and is not a general-purpose framework.

7. **No framework covers all languages** — plan for polyglot architectures if your
   organization spans Python, Java, and .NET.

---

*This guide was compiled from framework documentation, GitHub repositories, Neo4j Labs
ecosystem pages, and community resources as of July 2026. Frameworks evolve rapidly —
check individual repositories for the latest features and Neo4j integration status.*
