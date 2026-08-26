# RAG with Data Streaming: A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore  
> **Context:** LLM/AI Engineering · Real-Time Data Infrastructure — Streaming RAG, CDC, Event-Driven Indexing, Freshness  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Series:** LLM/AI Engineering Guides — bridges the [RAG series](rag_frameworks_comparison_guide.md) · [RAG vs HyDE](rag_vs_hyde_guide.md) · [RAG Optimization](rag_optimization_techniques_guide.md) and the [Event Stream Processing](../../event_stream_processing_guide.md) · [Closed-Loop Data Engineering](../closed_loop_data_engineering_guide.md) series  
> **Last Updated:** July 2026

---

## Table of Contents

1. [Why Streaming RAG Matters](#1-why-streaming-rag-matters)
2. [What Streaming RAG Is](#2-what-streaming-rag-is)
3. [The Freshness Spectrum](#3-the-freshness-spectrum)
4. [Architecture 1 — Streaming Ingestion with CDC](#4-architecture-1--streaming-ingestion-with-cdc)
5. [Architecture 2 — Event-Driven Indexing](#5-architecture-2--event-driven-indexing)
6. [Architecture 3 — Streaming Context Retrieval at Query Time](#6-architecture-3--streaming-context-retrieval-at-query-time)
7. [Architecture 4 — Streaming RAG with Stateful Stream Processing](#7-architecture-4--streaming-rag-with-stateful-stream-processing)
8. [Architecture 5 — Streaming-Aware Agentic RAG](#8-architecture-5--streaming-aware-agentic-rag)
9. [Architecture 6 — Serverless / Event-Driven Index Refresh](#9-architecture-6--serverless--event-driven-index-refresh)
10. [Technology Stack](#10-technology-stack)
11. [Streaming-Specific Considerations](#11-streaming-specific-considerations)
12. [The Freshness-Latency Tradeoff](#12-the-freshness-latency-tradeoff)
13. [RAG Pipeline Integration Patterns](#13-rag-pipeline-integration-patterns)
14. [Multi-Source Fusion and Mixed Freshness](#14-multi-source-fusion-and-mixed-freshness)
15. [Real-World Patterns and Use Cases in Banking](#15-real-world-patterns-and-use-cases-in-banking)
16. [Implementation Guidance](#16-implementation-guidance) — incl. streaming RAG with LangChain / LlamaIndex / Haystack (§16.4)
17. [Operational Concerns](#17-operational-concerns)
18. [Comparison and Decision Framework](#18-comparison-and-decision-framework)
19. [Anti-Patterns to Avoid](#19-anti-patterns-to-avoid)
20. [Conclusion](#20-conclusion)

---

## 1. Why Streaming RAG Matters

### 1.1 Standard RAG Indexes Are Snapshots

Classic RAG pipelines follow a batch lifecycle: extract documents → chunk → embed → upsert into a vector store → query. The index is built from a snapshot of the source data at a point in time. It is refreshed by re-running the pipeline on a schedule — nightly, hourly, or on manual trigger.

This design has three structural problems when the underlying data changes continuously:

**Batch re-indexing latency.** The gap between "a fact changed in the source system" and "the vector index reflects it" is measured in minutes to hours. A nightly refresh means the freshest possible answer is built on data up to 24 hours old. Even hourly refreshes leave a window where the index and reality disagree.

**The stale context problem.** Documents change (policies get amended), prices move (rates, FX, credit spreads), and events happen (trades execute, breaches occur, regulators publish). A RAG system that retrieves from a stale index will happily produce a confident, fluent, and *wrong* answer — the embedding is a frozen representation of a world that no longer exists.

**The freshness gap.** In fast-moving domains — financial markets, fraud, trading, news, regulatory change, customer events — the difference between "as of yesterday" and "as of now" is not a nicety; it is the difference between a useful answer and a dangerous one. An assistant that tells a trader yesterday's P&L, a risk manager last night's exposure, or a compliance officer last quarter's policy text has failed at its core job.

### 1.2 Concrete Failure Modes of Snapshot RAG

| Failure mode | Example | Consequence |
|---|---|---|
| Stale facts | Policy updated this morning; index still serves yesterday's version | Compliance answer cites revoked clause |
| Tombstoned docs linger | Document deleted from source; still retrieved by vector search | Hallucinated citations of removed content |
| Missing recent events | Trade executed 10 minutes ago; index knows nothing about it | Trader copilot reports flat position that is actually large |
| Ordering inversion | Update A applied after update B in the index (out-of-order batch) | Final state in index is a mix of versions |
| Price/state drift | Indexed price from T-1; live price moved 2% | Risk answer off by material amount |

### 1.3 Why This Matters for Banking

Banks are extreme cases of the freshness problem: the data that matters most — market data, positions, exposures, transactions, regulatory publications — is precisely the data that changes fastest and has the highest cost of being wrong. A batch RAG assistant answering from an 8-hour-old index is not an assistant; it is a liability. This is why streaming RAG — keeping the retrieval context continuously in sync with the state of the world — is an architectural requirement, not an optimization, for real-time banking use cases (see §15).

---

## 2. What Streaming RAG Is

**Streaming RAG** is the family of architectures and patterns that feed real-time streaming data into Retrieval-Augmented Generation systems so that LLM answers reflect the current state of the world rather than a stale snapshot. It has two definitions that are often conflated and both worth building:

1. **Continuous ingestion of data changes into the retrieval system** — the *index itself* is updated in near-real-time as sources change.
2. **Real-time context retrieval at query time** — fresh context is fetched from live sources *when a query arrives*, in addition to (or instead of) vector search.

A complete streaming RAG system typically does both: the vector index is kept fresh by a streaming pipeline, and the query path supplements it with live lookups.

### 2.1 Two Distinct Aspects

**Aspect (a) — Streaming the INGESTION side.** Data changes flow into the index continuously. The sources of change are database change data capture (CDC), domain events, and message queues. A pipeline of connectors → brokers → processors → embedders keeps the vector store in sync with inserts, updates, and deletes. The index is a *materialized view* of the source systems, refreshed in seconds rather than hours. This is "streaming the index."

**Aspect (b) — Streaming the INFERENCE side.** At query time, the RAG orchestrator fetches fresh context from live sources — time-series databases (last price), feature stores (real-time features), event stores (recent events), APIs (market data), or streaming joins — and assembles it into the prompt alongside vector-retrieved static knowledge. This is "streaming the context," and it directly bounds how old the freshest part of an answer can be.

| | Streaming the INGESTION side | Streaming the INFERENCE side |
|---|---|---|
| **What moves** | Data changes → index | Live data → prompt (at query time) |
| **When it runs** | Continuously, event-driven | Per query, on demand |
| **Latency driver** | Source → CDC → broker → embed → index | Live source lookup → prompt assembly |
| **Typical latency** | Seconds | Milliseconds to seconds |
| **Failure symptom** | Index lag (retrieves old docs) | Live lookup timeout/error |
| **Best for** | Docs that change: policies, wikis, news, records | State that must be current: prices, balances, positions, statuses |

The two aspects are complementary. Ingestion-side streaming keeps *knowledge* fresh; inference-side streaming keeps *state* fresh. A trading copilot needs both: the research note (ingestion-side, updated when analysts publish) and the current position (inference-side, fetched live per query).

---

## 3. The Freshness Spectrum

Freshness requirements are not binary ("streaming" vs "batch") — they are a spectrum. Choosing where your pipeline sits on it should be driven by the SLA of the business question, not by fashion.

| Tier | Latency | Mechanism | Banking use cases |
|---|---|---|---|
| **Batch** | Hours (nightly/periodic) | Scheduled jobs: nightly ETL → re-index | Regulatory corpus snapshots, product documentation, onboarding manuals, annual report Q&A |
| **Micro-batch** | Minutes | Scheduled jobs at 5–15 min intervals (Spark micro-batch, scheduled Lambda) | Intraday risk summaries, market news digests, AML typology updates, pricing sheet refreshes |
| **Near-real-time** | Seconds | Streaming pipeline: CDC → Kafka → embed → upsert | Policy updates, regulatory publications, trade/position changes, customer profile changes, alert generation |
| **Real-time** | Sub-second | Query-time live lookup: API, feature store, Redis, time-series DB | Current price/P&L, live exposure, account balance/status, fraud screening of the transaction in flight |

**Key principle:** spend streaming infrastructure only where the SLA demands it. A stable reference document (e.g., a product brochure) does not need CDC; a position table that changes every second cannot be served from a nightly index. §19 gives the decision framework.

---

## 4. Architecture 1 — Streaming Ingestion with CDC

### 4.1 Overview

The canonical streaming-RAG ingestion pattern: capture every change to a source database and flow it into the vector index within seconds.

```
Source DB (PostgreSQL/Oracle/SQL Server)
        │  WAL / redo log
        ▼
Debezium (Kafka Connect CDC connector)
        │  change events
        ▼
Kafka  topics: per-table (e.g., policies, trades, customers)
        │  keyed by primary key
        ▼
Index Update Worker (Kafka consumer group)
        │  derive doc → chunk → embed (batched, async)
        ▼
Vector Store  (upsert / delete by doc ID)
```

**Flow:** a database write/update/delete → the CDC connector reads the transaction log (WAL / redo log) → publishes a change event to Kafka → the index update worker consumes it → derives and embeds the affected document/chunk → upserts or deletes in the vector store. DB writes, updates, and deletes reach the index within seconds, end to end.

### 4.2 Pattern: Outbox Pattern for Reliable CDC

A subtlety of CDC: the connector only sees committed changes, and only if the database retains the log long enough. For systems where CDC is not available (no WAL access, cloud-managed DBs with restricted log access, or application-level business rules that must gate what gets indexed), use the **transactional outbox pattern**:

1. The application writes its normal record **and** an "outbox" record (the event to publish) in the **same database transaction**.
2. A relay (Debezium on the outbox table, or a poller) publishes outbox rows to Kafka.
3. The index worker consumes from the topic exactly as in pure CDC.

This guarantees that "committed in the DB" and "published to the stream" are atomic — no lost events, no events for rolled-back transactions. Kafka Connect's Debezium outbox event router is the reference implementation.

### 4.3 Pattern: Kafka Topics per Table/Entity

One topic per table or entity (or one topic with a table-type key) keeps ordering and consumer logic simple:

- **Ordering:** Kafka preserves order *per partition key*. Keying messages by the primary key (or the doc ID) guarantees that all changes to one document are processed in order, so update-before-delete inversions cannot corrupt the index.
- **Isolation:** a burst on the high-volume `trades` topic does not delay the `policies` topic; each topic can have its own consumer group, parallelism, and lag budget.
- **Schema:** per-topic schemas in the Schema Registry keep payload evolution manageable (§11.7).

### 4.4 Upsert Semantics (Insert / Update / Delete)

The worker must translate the three CDC operation types into vector-store operations:

| CDC operation | Vector store operation | Detail |
|---|---|---|
| `c` (create) | Upsert by doc ID | Embed the new doc/chunk, write with `created_at` |
| `u` (update) | Upsert by doc ID | Re-embed changed chunks, bump version; optionally delete-unchanged chunks (§11.4) |
| `d` (delete) | Delete by doc ID | Delete all chunks with that doc ID (§11.3) |
| `r` (snapshot) | Upsert | Initial snapshot phase — treat as create |

**Tombstone handling:** Kafka messages with a `null` value are tombstones — they signal "this key is deleted" for log compaction. The worker must interpret a tombstone (or a delete CDC event) as a vector-store delete, not skip it. Missing tombstone handling is a top anti-pattern (§20): deleted documents linger in the index and keep being retrieved.

### 4.5 When to Use

- Source of truth is a **relational database** whose rows map to documents/chunks.
- You need to index **any** change, including changes made outside the application (SQL scripts, other services writing the same DB).
- You need **replayability** — Kafka retention lets you rebuild the index from the stream without touching the source.

### 4.6 Limitations

- Only sees row-level changes — a column that derives from a join of several tables requires enrichment in the worker (see Architecture 4).
- CDC captures *state changes*, not necessarily *business events* — a "trade executed" domain event may need Application-level derivation (Architecture 2) instead.
- High write volume produces embedding load proportional to change rate — plan capacity (§18.3).

---

## 5. Architecture 2 — Event-Driven Indexing

### 5.1 Overview

Where CDC captures *database* changes, event-driven indexing consumes *domain events* — the business facts the application itself emits: `OrderPlaced`, `TradeExecuted`, `CustomerChanged`, `PolicyAmended`. The application is the source of truth for what *happened*; the index is a derived artifact.

```
Application (order service, trading system)
        │  domain events
        ▼
Event Bus (Kafka / Pulsar / Redpanda)
        │  topics per event type
        ▼
Event Processors (Kafka Streams / Flink / workers)
        │  derive document → chunk
        ▼
Embedding Service (async, batched)
        ▼
Vector Store (upsert by doc ID)
```

**Flow:** the application publishes a domain event when something meaningful happens → processors consume it → derive one or more documents/chunks (e.g., a `TradeExecuted` event → a "trade summary" document) → embed → upsert.

### 5.2 Pattern: Events as Source of Truth

The event stream is the system of record for *what happened*; the vector index is a *derived, queryable projection* of it. Advantages:

- **Semantic richness:** events carry business meaning ("trade executed at 1,024.50 for 5,000 contracts") that row-level CDC does not — the derived document can be written directly in language the LLM will use.
- **No coupling to DB schema:** renaming a column does not break the event contract if the event schema is versioned (§11.7).
- **Event sourcing compatibility:** if the domain already uses event sourcing, the RAG index is just another projection — replay the event log to rebuild it.

### 5.3 Pattern: Derived Documents

One event → one or more documents, shaped for retrieval:

| Domain event | Derived document | Chunking |
|---|---|---|
| `TradeExecuted` | Trade summary: instrument, side, qty, price, counterparty, venue, timestamp | One doc per trade, few chunks |
| `CustomerChanged` | Updated customer profile note | One doc per customer |
| `PolicyAmended` | Policy delta + new clause text | Doc per policy, chunk per clause |
| `FraudAlertRaised` | Alert summary + linked evidence | One doc per alert |
| `RegulatorPublication` | Publication digest | Doc per publication, chunk per section |

The derivation logic is where domain knowledge lives: decide *what* an LLM user would want to retrieve about this event, and write the document accordingly.

### 5.4 Pattern: Idempotency (Event Replay Is Safe)

Streams redeliver (at-least-once is the default). The pipeline must be **idempotent**: processing the same event twice must not create duplicate chunks. Concretely:

- **Doc ID derived from the event key** — e.g., `doc_id = f"trade:{trade_id}"`. Upserting the same ID twice overwrites, never duplicates.
- **Event ID recorded in chunk metadata** — `source_event_id` lets you trace which event produced a chunk and dedupe at the worker if needed.
- **Version the document** — each event carries a sequence/version; the worker writes `doc_version` and can ignore events older than the currently indexed version (out-of-order protection, §11.9).

### 5.5 Pattern: Versioning

Documents are versioned, not just overwritten. The index stores `doc_version`, `updated_at`, and optionally retains the previous version for "as-of" queries (§13.3). Versioning is what makes late-arriving data, corrections, and rollback tractable: a correction event bumps the version; a replay that processes events in the original order lands on the same final version regardless of arrival order.

### 5.6 When to Use

- The application already emits domain events or can be refactored to (event-driven architecture in place).
- Documents should reflect *business meaning* rather than raw row state.
- You want replay-safe, versioned indexing with clear provenance (event IDs in metadata).

---

## 6. Architecture 3 — Streaming Context Retrieval at Query Time

### 6.1 Overview

No matter how fast the ingestion pipeline runs, the freshest possible *indexed* data is bounded by pipeline latency. When the SLA is sub-second or the data is too volatile to index at all (prices changing every tick), the answer is **query-time retrieval**: fetch live context at query time and fuse it with vector search results.

```
User query
   │
   ▼
RAG Orchestrator
   ├──► Vector search  (static knowledge, freshly indexed docs)
   ├──► Time-series DB (last price, 5-min returns)
   ├──► Feature store (real-time features — current exposure, velocity)
   ├──► Event store   (recent events — last 10 trades, recent alerts)
   ├──► API calls     (market data, account status, position service)
   └──► Streaming join (combine vector results + live data)
                │
                ▼
        Context assembly → Prompt → LLM → Answer
```

### 6.2 Pattern: Hybrid Context Assembly (Static + Live)

The orchestrator builds the context from two classes of sources:

- **Static/slow:** vector-retrieved documents — policies, research, typologies. Freshness governed by the ingestion pipeline (minutes at best).
- **Live/fast:** point lookups — current price, current exposure, account balance, latest status, recent events. Freshness governed by the source system itself (sub-second).

The prompt is assembled with both, **labeled by source and as-of time** so the LLM can reason about freshness (§14.3).

### 6.3 Pattern: Freshness-Bounded Retrieval

Define a **freshness budget**: the maximum acceptable age of any context element, per type. For each source, declare `max_age`:

| Context type | Source | Max age (SLA) | Enforcement |
|---|---|---|---|
| Current market price | Time-series DB / market data API | 1 second | Live lookup per query |
| Current exposure | Risk service / feature store | 5 seconds | Live lookup per query |
| Policy text | Vector index | 15 minutes | Index lag monitoring (§12.5) |
| Research note | Vector index | 1 day | Index metadata timestamp |

The orchestrator checks the `as_of` timestamp on every retrieved item and **discards or re-fetches** anything older than its budget. This is the retrieval-side enforcement of the freshness SLA.

### 6.4 Pattern: Multi-Source Fusion

Fusion is more than concatenation — it is deciding *what to include* and *in what order*:

1. **Retrieve:** vector top-k (static) + live lookups (fast sources).
2. **Filter:** drop items outside freshness budget; drop duplicates by doc ID (a doc may appear in both vector results and live lookup).
3. **Rank/order:** live, high-stakes context (current position, price) goes *first* in the prompt — near the question — so it dominates; static background follows.
4. **Label:** tag each item `[live as-of 14:32:07]` / `[static as-of 2026-07-31]`.
5. **Generate:** prompt the LLM to prefer live items for state questions and to state as-of times in the answer (§14.3).

### 6.5 When to Use

- Data changes faster than you can (or want to) index it — prices, positions, balances, statuses.
- The answer depends on *current state* more than on accumulated knowledge.
- You want a hard, sub-second bound on the freshest part of the answer.
- Cost control: indexing every tick is wasteful; live lookup costs only per query.

---

## 7. Architecture 4 — Streaming RAG with Stateful Stream Processing

### 7.1 Overview

Between "raw event" and "context for the LLM" there is often a **computation**: rolling aggregates, latest values, running totals, alert states. Flink / Kafka Streams compute continuously updated state; the RAG query path reads that state by key at query time.

```
Kafka (events: trades, positions, market data)
        │
        ▼
Flink / Kafka Streams (stateful, keyed)
   ├── rolling aggregates (portfolio value, 5-min returns)
   ├── latest-value tables (last price per instrument)
   ├── alert states (breach flags, thresholds crossed)
   └── enriched streams (events + reference data join)
        │
        ▼
Stateful stores (RocksDB, Redis, state store + changelog)
        │
        ▼
RAG query path ──► point lookup by key ──► prompt context
```

### 7.2 Pattern: State as Context

The stateful store is a **live feature source for RAG** — the LLM never computes; it reads pre-computed state:

- **Current portfolio value / running totals** — Flink aggregates the trade stream, keeps the latest total in Redis/RocksDB; the copilot reads it per query.
- **Windowed aggregates** — 5-minute returns, 1-hour volume, running 7-day total; pre-computed so the prompt gets numbers instantly.
- **Alert states** — "counterparty X breached the intraday limit at 14:05"; the risk assistant retrieves the alert state and the policy text (vector) and explains both.

This is the pattern that turns "an LLM that talks about data" into "an LLM that talks about *current* data" without query-time joins over raw streams.

### 7.3 Pattern: Serving Layer Choices

| Store | Strengths | RAG serving pattern |
|---|---|---|
| **Flink keyed state (RocksDB)** | Exactly-once, co-located with computation | Queryable state (limited); better: emit to a serving store |
| **Redis** | Sub-ms reads, TTL, simple | `GET portfolio:{id}` per query; feature cache (§18.3) |
| **ClickHouse** | Analytical queries over recent data | "last 10 trades for this counterparty" as context |
| **Kafka tables (KTables)** | Replayable, joins with streams | Push-table → compacted topic → read latest per key |
| **Materialize / RisingWave** | SQL-defined materialized views, incremental | Expose live views via SQL/Postgres wire protocol to RAG |

### 7.4 When to Use

- Context requires **computation over many events** (aggregates, ratios, alerts), not just the latest row.
- The same computed state serves many queries — precompute once, read many times.
- You already run Flink/Kafka Streams in the estate (common in banking risk platforms) and want to reuse the state.

---

## 8. Architecture 5 — Streaming-Aware Agentic RAG

### 8.1 Overview

An agentic RAG system gives the LLM **tools** that query live systems — a market-data API, a database, a Kafka consumer that reads the last N events, a position service. The agent decides *when* to fetch fresh data: it can answer from memory/context for stable questions, and call a live tool when the question demands current state.

```
User query
   │
   ▼
Agent (LLM + tools)
   ├── VectorSearch(policy, typology, research)      ← static knowledge
   ├── GetLivePrice(ticker)                           ← market data API
   ├── GetPositions(desk)                             ← position service / DB
   ├── GetRecentEvents(entity, window)                ← Kafka consumer / event store
   ├── GetFeature(feature, entity)                    ← feature store
   └── CheckAlertState(entity)                        ← stateful store
        │
        ▼
  Tool results labeled as-of → final answer
```

### 8.2 Pattern: Tool-Augmented RAG

The agent's system prompt declares: *for anything time-sensitive — prices, positions, balances, statuses, recent activity — call the corresponding tool rather than guessing or using context.* The vector index remains the knowledge base; live tools are the state layer. This is Architecture 3 with **adaptive** lookup: the agent decides which live sources matter for this particular query, instead of the orchestrator fetching everything every time.

### 8.3 Pattern: Live Lookups on Demand

- **Lazy fetching:** don't pre-fetch all live sources; let the agent request only what the question needs. A question about a policy clause needs no price tick; a question about "current exposure to X" needs the exposure service, not the news index.
- **Bounded tool calls:** cap the number of live tool calls per query (budget) to control latency and cost; a query that needs 6 live calls is probably two queries.
- **Tool results as context:** each tool result enters the prompt as labeled context (`[live as-of 14:32:07] exposure EUR 12.4M`), subject to the same freshness labeling as Architecture 3.

### 8.4 Pattern: Budgeted Freshness (Agent Checks Recency)

Give the agent explicit freshness reasoning:

- Every retrieved item carries an `as_of` timestamp (from index metadata or tool response).
- The agent is instructed: *if the item's as_of is older than the question's freshness requirement, and a tool can get fresher data, call it.*
- The freshness budget per context type (§6.3) is exposed to the agent as part of its instructions — the agent enforces the SLA by deciding when to refresh.

This is the most flexible architecture: it adapts freshness spend to the question, but it is also the hardest to make deterministic and testable (§16.5).

### 8.5 When to Use

- Queries are heterogeneous — some purely knowledge-based, some state-based, some both.
- You want a conversational assistant (fraud analyst, trader copilot, risk monitor) rather than a fixed pipeline.
- You can tolerate (and test for) the nondeterminism of agentic tool choice.

---

## 9. Architecture 6 — Serverless / Event-Driven Index Refresh

### 9.1 Overview

For document-oriented sources (object storage, CMS, SharePoint, internal wikis, regulatory file drops), the pipeline can be fully serverless: a *change event* triggers re-embedding, with each stage a message-driven function.

```
Document source (S3 / Azure Blob / CMS / wiki)
        │  object-created / object-modified / object-deleted event
        ▼
Event source (S3 event notification / webhook / poller)
        ▼
Queue / Event Bus (SQS / Event Hubs / NATS)
        ▼
Trigger: Lambda / Knative function / Cloud Run
        │  fetch object → parse → chunk → diff vs existing
        ▼
Embedding service (async, batched)
        ▼
Vector store (upsert / delete by doc ID)  +  metadata store
```

### 9.2 Pattern: Event-Driven Architecture with Message-Driven Pipeline Stages

Each stage is decoupled by a queue; each stage scales independently:

1. **Trigger stage:** object events (S3 `s3:ObjectCreated:*`, `s3:ObjectRemoved:*`) land in a queue.
2. **Parse/chunk stage:** a function fetches the object, parses, chunks, and *diffs* against the previous version to determine which chunks changed (§11.4).
3. **Embed stage:** changed chunks go to an embedding queue; a worker batch-embeds respecting API rate limits (§11.1).
4. **Index stage:** upserts/deletes hit the vector store.

The S3-event → Lambda → vector-store path is the classic "document changed → re-index just that document" pattern; the same shape works with CMS webhooks, SharePoint change events, or wiki publish events.

### 9.3 When to Use

- Document sources are file/object based with change notifications available.
- You want minimal always-on infrastructure (pay per event).
- Change rate is moderate (bursty, not continuous high-volume) — serverless is cost-optimal there; a continuous Kafka pipeline is better for sustained high rates.

---

## 10. Technology Stack

### 10.1 Event Backbone

| System | Type | Notes for streaming RAG |
|---|---|---|
| **Apache Kafka** | Distributed commit log | De facto standard. Replayable (rebuild index from stream), per-key ordering, log compaction (keep latest per key), huge ecosystem. Best default. |
| **Apache Pulsar** | Log + queue | Multi-tenant, separate storage/compute, geo-replication built in. |
| **Redpanda** | Kafka-compatible | Kafka API without ZooKeeper/JVM; lower ops burden, good for smaller estates. |
| **Amazon Kinesis** | Managed stream | AWS-native; Data Streams (real-time) + Firehose (buffered) |
| **Azure Event Hubs** | Managed stream | Kafka-API compatible; Azure-native. |
| **NATS / JetStream** | Lightweight message bus | Low latency, simple; JetStream adds persistence. |
| **RabbitMQ** | Message broker | Queues, not a replayable log — fine for triggers, weak for replay/rebuild. |

### 10.2 CDC

| Tool | Notes |
|---|---|
| **Debezium (Kafka Connect)** | The reference CDC: Postgres/Oracle/SQL Server/MySQL/MongoDB → Kafka; outbox router; schema events. |
| **AWS DMS** | Managed CDC to Kinesis/S3 for AWS sources. |
| **Fivetran / Airbyte** | Managed connectors with CDC modes; good when you also want warehouse sync. |
| **Striim** | Commercial CDC + streaming analytics. |
| **Maxwell's Daemon** | Lightweight MySQL CDC to JSON on Kafka. |

### 10.3 Stream Processing

| System | Notes |
|---|---|
| **Apache Flink** | Stateful, exactly-once, event-time/watermarks; the serious choice for Architecture 4. |
| **Kafka Streams** | Embedded library, KTable/KStream, exactly-once in Kafka; lowest ops for Kafka shops. |
| **Spark Structured Streaming** | Micro-batch; good if Spark estate exists; minutes-tier freshness. |
| **ksqlDB** | SQL on Kafka Streams; pull queries = live lookups for RAG. |
| **RisingWave / Materialize** | SQL materialized views with incremental updates; expose live views to RAG over Postgres wire protocol. |

### 10.4 Vector Stores with Streaming Support

| Store | Streaming-relevant features |
|---|---|
| **Qdrant** | Streaming upserts, point IDs (deterministic doc IDs), snapshots, replication; excellent per-point update/delete semantics. |
| **Weaviate** | Streaming import API, auto-schema, cross-references; per-object updates. |
| **Milvus** | High-throughput ingestion, bulk insert, partitions, dynamic schema; good at scale. |
| **Elasticsearch / OpenSearch** | Ingest pipelines, per-doc updates/deletes, mature ops; hybrid BM25+vector. |
| **pgvector** | Upsert via `INSERT ... ON CONFLICT (id) DO UPDATE`; joins with relational data in the same DB. |
| **Pinecone** | Serverless, upsert API by ID, namespace deletion; managed ops. |
| **LanceDB** | Embedded, streaming writes, columnar storage; good for edge/local. |

### 10.5 Caching / State

| Store | Role in streaming RAG |
|---|---|
| **Redis** | Feature cache, recent-events ring buffer, live-lookup cache with TTL (§18.3), semantic-cache for answers (§12.6). |
| **RocksDB** | Flink/Kafka Streams state backend; embedded. |
| **ClickHouse** | Real-time analytics over recent events; "last N events" context queries. |
| **InfluxDB / TimescaleDB** | Time-series: last price, windowed aggregates served to RAG. |

---

## 11. Streaming-Specific Considerations

These are the engineering concerns that batch RAG never hits and that make or break a streaming pipeline.

### 11.1 Embedding Throughput and Rate Limits

Embedding APIs (OpenAI, Cohere, Voyage, Azure) impose rate limits (RPM/TPM) and latency per call. A streaming pipeline can easily outrun them:

- **Batch embeddings** — send chunks in batches (e.g., 64–256 per request) rather than one call per chunk.
- **Async, decoupled embedding** — the index worker enqueues embedding jobs; an embedding worker consumes them with concurrency control. Never embed inline in the ingestion hot path (§20).
- **Retry with exponential backoff + jitter** on 429/5xx; honor `Retry-After`.
- **Dead-letter queue (DLQ)** for permanently failing chunks (poison payloads, schema errors) — §16.6.
- **Capacity math:** if the source produces 10 changes/sec × 4 chunks each = 40 chunks/sec ≈ 3.5M chunks/day. With a 1,000 chunk/min embedding rate limit you need ~7 embedding workers with batching — or a self-hosted embedder (sentence-transformers/vLLM) with no external limit.

### 11.2 Idempotent Upserts

**Doc ID as key.** Every chunk is addressed by a deterministic ID: `doc_id = f"{source}:{entity_id}"`, `chunk_id = f"{doc_id}:{chunk_index}"` (or hash of content for stable chunks). Upserting the same ID is an overwrite, never a duplicate — this makes at-least-once delivery safe. Never auto-generate IDs (UUID per embed) — replays would duplicate.

### 11.3 Delete Propagation (Tombstones → Vector Delete)

Deletes are the most commonly broken part of streaming RAG:

- CDC delete event or Kafka tombstone → worker must issue `delete(doc_id)` to the vector store.
- **Tombstone handling:** Kafka log compaction removes tombstoned keys after retention, but only after consumers have seen them — so the consumer must act on `null` values, not skip them.
- **Soft delete alternative:** mark `is_deleted=true` in metadata and filter at query time — safer when you need audit history, but stale chunks still cost retrieval time unless filtered.
- Verify your vector store actually reclaims space (Qdrant deletes points; Milvus needs `delete` + compaction; ES deletes are lazy).

### 11.4 Partial Updates (Changed Chunks Only)

Re-embedding a whole document on any change is wasteful (§20). Prefer **chunk-level diffing**:

1. On update, re-parse the document and chunk it.
2. Compare chunk hashes against the stored version's chunk hashes.
3. Upsert only changed/new chunks; delete only removed chunks; leave identical chunks untouched.

For structured docs (policy clause changed), a section-anchored chunking scheme makes diffs trivial. Version metadata (`doc_version`, per-chunk `content_hash`) enables this.

### 11.5 Index Consistency

- Vector stores are **eventually consistent**: an upsert is not immediately visible to all readers; replication lag varies by store.
- **Read-your-writes:** if a user just edited a doc and immediately asks the assistant, they should see the edit. Options: (a) accept seconds of lag with a "as-of" disclaimer; (b) query-time live lookup for the specific doc ID when the user just modified it; (c) strong-consistency stores for critical docs.
- Decide *which* documents need read-your-writes (usually: user-edited docs, compliance-sensitive docs) and *which* tolerate eventual consistency (news, reference data).

### 11.6 Ordering Guarantees

- **Per-key ordering in Kafka** is the backbone: key change events by doc/entity ID so updates and deletes for the same doc are processed in order.
- **Event time vs processing time:** events carry `event_time` (when the fact happened) and arrive at `processing_time` (when the pipeline sees them). Out-of-order arrival (a slow CDC batch delivering an older update after a newer one) must be handled — see §11.9 and §13.2.
- Never rely on broker arrival order across partitions for the same entity — key by entity.

### 11.7 Schema Evolution

Payloads change; the pipeline must not break:

- **Schema Registry** (Confluent/Apache) with compatibility modes (backward/forward/full) at the Kafka boundary.
- **Payload versioning:** events carry `schema_version`; the index worker handles multiple versions.
- **Chunk/document schema version** in metadata (`doc_schema_version`) so old and new chunks are both queryable and distinguishable.
- Unknown fields: ignore-and-log (forward compatible) or fail-and-DLQ per policy.

### 11.8 Chunk Stability

The enemy of streaming indexing is re-chunking instability — a one-word edit that shifts chunk boundaries and invalidates every chunk. Mitigations:

- **Anchor-based chunking** (headers, sections, paragraphs) so edits localize to one chunk.
- **Content-hash chunk IDs** (`chunk_id = sha256(normalized text)`) so unchanged chunks keep the same ID and need no re-embed.
- **Chunk-level diffing** (§11.4) on top of stable chunking.

### 11.9 Exactly-Once vs At-Least-Once

| Semantics | Meaning | Streaming RAG consequence |
|---|---|---|
| At-least-once | Events may redeliver | Safe only with idempotent upserts (doc IDs) — the default you should design for |
| Exactly-once (Kafka Streams/Flink) | No duplicates, no losses in stream processing | Useful for aggregates (Architecture 4); vector-store writes remain effectively idempotent via doc IDs |
| Effectively-once (end to end) | Duplicate events → same final index state | What idempotent upserts give you; this is the goal |

Duplicates in the vector store are the symptom of ignoring idempotency (§20).

### 11.10 Backpressure

Slow embedding → queue backlog → index lag → stale answers. Handle by:

- Buffering queues with bounded depth and **backpressure signals** (worker pauses consumption when the embedding queue exceeds a threshold).
- Autoscaling embedding workers (KEDA on queue depth, §18.3).
- Monitoring queue depth as a first-class alert (§18.2).
- Degradation policy: if lag exceeds budget, serve queries from the last-good index with an explicit staleness banner rather than failing or serving an empty index.

### 11.11 Metadata Freshness

Every chunk carries freshness metadata — the *minimum* for freshness-aware retrieval (§12):

- `as_of` / `updated_at` (when the underlying fact was true)
- `indexed_at` (when it entered the index — lag = `indexed_at - as_of`)
- `doc_version`, `source_event_id`, `source` (system of record)
- `content_hash` (for diffing)

This metadata is what makes freshness filtering (§6.3), staleness monitoring (§12.5), and provenance auditing (§18.4) possible at all. No freshness metadata = you cannot tell how fresh context is = the anti-pattern in §20.

---

## 12. The Freshness-Latency Tradeoff

### 12.1 How Fresh Does Context Need to Be?

Freshness is a **requirement, not a feature**. Derive the SLA from the business question:

| Question type | Staleness tolerance | Example |
|---|---|---|
| "What is the current price/exposure/position?" | Seconds | Trading, risk |
| "What happened in the last hour?" | Seconds–minutes | Event-driven Q&A |
| "What does the current policy say?" | Minutes | Compliance, policies |
| "How does this typology work?" | Days–weeks | AML knowledge base |

**Freshness budget = max age of any context element** in an answer. It decomposes into per-source budgets (§6.3). Budgets that are tighter than the slowest stage are unachievable — design the stage latencies to fit the budget, or move that context type to query-time live lookup.

### 12.2 Latency of Each Stage

End-to-end index freshness = sum (with concurrency) of:

```
source commit → CDC poll/log read → publish → Kafka → consumer → derive → embed → vector store write → query visible
   ms–s            s                  ms        ms       s        ms       100ms–s      s–10s          ms–s
```

Rule of thumb for a tuned pipeline: **5–30 seconds** from DB commit to query-visible index entry; **1–10 seconds** is achievable with hot embedders and fast stores; **sub-second requires query-time live lookup** (Architecture 3) — no indexing pipeline can beat it.

### 12.3 Measuring End-to-End Freshness (Event Time → Index Time)

- **Event-time lag:** `indexed_at − event_time` — true staleness of the *content*.
- **Pipeline lag:** `indexed_at − processing_time` — how far behind the pipeline is.
- **Consumer lag:** Kafka consumer lag (offsets) — backlog indicator.
- Instrument every stage with timestamps in the chunk metadata (§11.11); compute percentiles (p50/p95/p99) per topic.

### 12.4 Monitoring Staleness

- **Index lag metric:** max/p95 `(now − as_of)` across chunks of each collection — "oldest thing I could serve for this collection."
- **Last-updated timestamps:** expose per-collection `last_indexed_at` in the query API so callers (and the LLM) know the index's freshness ceiling.
- **Per-source budgets:** alert when a collection's p95 lag exceeds its budget (§18.2).

### 12.5 Cache Invalidation

Caches (semantic answer caches, per-query context caches) multiply staleness problems:

- **Semantic cache TTL:** cached answers get a TTL tied to the freshness budget of the question type. A cached "current exposure" answer is poison after 5 seconds; a cached policy answer can live 15 minutes.
- **Versioned chunks:** cache keys include `doc_version`/`content_hash` — when the streaming pipeline updates a doc, the cache key changes and the stale entry is bypassed.
- **Explicit invalidation events:** the pipeline can publish `doc_changed` events that a cache layer consumes to evict affected entries.

### 12.6 Freshness vs Consistency

- Vector stores are eventually consistent — a freshly upserted doc may not be query-visible everywhere for seconds. Accept it and label answers with as-of times.
- **Query-time merging of live + static** (§14) trades a small consistency risk (live value and indexed doc may disagree momentarily) for freshness. Label the tension explicitly: the live price is the truth for "now"; the doc is the truth for "as of its timestamp."
- For regulatory answers, prefer *correct-as-of* over *current*: label the as-of date and let the reader decide.

---

## 13. RAG Pipeline Integration Patterns

### 13.1 Streaming + Batch Hybrid

Batch is not the enemy — it is the right tool for bulk/historical:

- **Daily corpus + live updates:** nightly batch loads the full historical corpus (research archives, regulatory libraries); the streaming pipeline handles deltas (today's changes). The streaming worker treats the batch load as its initial snapshot (`r` events) and continues incrementally.
- **Batch for bulk, streaming for deltas:** batch re-embeds the stable 99%; streaming only touches changed docs. This keeps embedding cost proportional to change rate, not corpus size.
- **Reconciliation:** periodic batch job verifies index vs source (missing docs, orphan chunks) and repairs — a safety net for streaming gaps.

### 13.2 Late-Arriving Data

Streams deliver events out of order and late:

- **CDC replays:** restarting a connector re-reads the log; use idempotent upserts (§11.2) so replays are safe; version checks (§11.9) ignore stale arrivals.
- **Out-of-order events (event-time watermarks):** when event-time ordering matters (aggregates in Architecture 4), use Flink watermarks; for index updates, per-doc version comparison at the worker decides whether an older event overwrites a newer one. Rule: **apply if `event_version > indexed_version`, else drop.**
- **Corrections:** a correction is `delete(old) + insert(new)` (or a versioned upsert). Never "patch" — vector embeddings are not diffable; re-embed the corrected chunks and bump version.

### 13.3 Document Lifecycle

```
create ──► active ──► update ──► active (version +1)
              │                       │
              ▼                       ▼
          delete ──► tombstone (soft) / hard delete / version history
```

- **Tombstones:** Kafka null-value message or CDC delete → vector delete (§11.3).
- **Soft-delete:** keep the chunk, set `is_deleted=true`, filter at query time — preserves audit trail.
- **Version history:** keep the last N versions (or a time-partitioned "as-of" collection) for point-in-time Q&A ("what did the policy say last week?"). Cost: storage × versions. Store old versions in a cold collection, serve current from the hot one.

### 13.4 Re-Indexing Strategy

- **Incremental chunk updates** (§11.4) — the default; only changed chunks are re-embedded.
- **Chunk-level versioning** — `doc_version` on chunks; queries filter `doc_version = latest` per doc.
- **Full rebuild** — reserved for embedding-model upgrades, chunking-scheme changes, corruption recovery (§18.3). Run as: build shadow index → verify → swap.

### 13.5 Query-Time Freshness Techniques

- **Metadata filters:** `filter: updated_at >= now - 1h` narrows retrieval to recent content for "what changed recently" queries — cheap and effective.
- **Recency boosting:** score = similarity + `w × recency(updated_at)` — mildly favors newer content when the query is ambiguous between versions. Tune `w`; too high degrades semantic quality.
- **Live context injection:** append live data to the prompt (current price, latest status) regardless of vector results (§6.2).
- **Freshness-aware prompting:** tell the LLM the as-of date of the context ("Context is as of 2026-07-31 14:32; the live price is from 14:32:07") so it can hedge, flag, or refuse stale-based claims.

---

## 14. Multi-Source Fusion and Mixed Freshness

### 14.1 The Fusion Pattern

The strongest streaming-RAG answers combine:

1. **Vector search** — accumulated knowledge: policies, typologies, research, historical patterns.
2. **Structured live query** — SQL/API: current positions, balances, exposures.
3. **Feature store lookup** — real-time features: velocity scores, anomaly scores, running aggregates.

Each source has its own freshness; the orchestrator assembles them into **one context block with mixed freshness**, labeled per source.

### 14.2 Assembly Order and Policy

1. Resolve the entities the query mentions (instrument, counterparty, customer, desk).
2. Parallel-fetch: vector top-k + live lookups for the resolved entities.
3. Filter by freshness budget per source type (§6.3).
4. Order: live state first (it answers "now"), then recent events, then static knowledge.
5. Dedupe by doc ID; collapse same-source items.

### 14.3 Prompt Design for Mixed Freshness

Label sources explicitly; teach the LLM to respect the labels:

```
Context:
[live as-of 2026-07-31 14:32:07] EURUSD mid = 1.0842; desk P&L today = +EUR 412k
[live as-of 2026-07-31 14:32:05] Exposure to Counterparty A = EUR 12.4M (limit 15M)
[event 14:28] Trade executed: 5,000 lots EURUSD @ 1.0839 (client X)
[static as-of 2026-07-31] Internal policy POL-221: intraday limit breach must be
reported within 30 minutes...
[static as-of 2026-07-29] Research note: EURUSD rangebound, ECB path dependent...

Instructions: Prefer [live] items for "current" questions; use [static] for
background. State the as-of time of the freshest data you relied on.
```

This makes the answer's freshness legible to the user and lets the LLM refuse or hedge when the freshest available data is too old for the question.

---

## 15. Real-World Patterns and Use Cases in Banking

### 15.1 Pattern Taxonomy

Four recurring patterns cover most banking streaming-RAG use cases:

| Pattern | Freshness driver | Example |
|---|---|---|
| **"Fresh knowledge" RAG** | Docs updated continuously | Internal wikis, policies, product changes streamed to index |
| **"Live state" RAG** | Current state per query | Balances, positions, statuses fetched live at query time |
| **"Event-driven Q&A"** | Recent events | "What happened in the last hour?" answered from event store |
| **"Hybrid freshness"** | Mix in one answer | Static knowledge + live state fused (§14) |

### 15.2 Real-Time Fraud Detection Assistant

- **Vector for patterns:** historical fraud cases, typologies, red-flag narratives — the accumulated knowledge base.
- **Live for current:** the transaction in flight, its velocity features (feature store), the customer's recent behavior (event store), current device/location signals.
- **Flow:** fraud analyst asks "why was TXN-8821 flagged?" → agent retrieves similar historical cases (vector) + the live transaction's features and recent events (live) → answer explains the flag with current evidence and cites precedent cases.
- **Freshness need:** seconds. A fraud decision made on yesterday's patterns is how fraud happens.

### 15.3 Trading Desk Copilot

- **Live:** P&L, positions, market data, order status (stateful stream + live lookups).
- **Static:** research notes, desk playbooks, product docs (indexed, ingestion-side streaming for new research).
- **Query example:** "What's our EURUSD exposure and what does recent research say?" → live position + live price + research note in one answer.
- **Freshness need:** sub-second for prices; minutes for research ingestion.

### 15.4 Risk Monitoring

- **Live exposure** per counterparty/desk from the risk platform (stateful aggregates).
- **Regulatory docs** and **breach thresholds** from the vector index.
- **Query example:** "Are we near any limit breach?" → agent reads live exposure table + retrieves the limit policy → answers with headroom, threshold text, and as-of times.
- **Freshness need:** seconds; breach questions are exactly when stale data is most dangerous.

### 15.5 Customer Support with Live Account State

- **Live:** account balance, status, recent transactions (API/DB lookup per query).
- **Static:** product documentation, fees, terms (indexed).
- **Query example:** "Why is my transfer pending?" → live transaction status + product doc on processing times.
- **Freshness need:** seconds for state; weeks for docs. Classic hybrid freshness — and the highest-volume use case.

### 15.6 Regulatory Change Monitoring

- **Ingestion-side streaming:** regulator feeds (publications, consultation papers, final rules) → stream → parse → index → **alert + Q&A**.
- **Query example:** "Summarize what changed in MAS Notice 637 this quarter" → retrieves the streamed-in delta documents with as-of dates.
- **Freshness need:** minutes; a bank is expected to be aware of regulatory change the day it is published.

### 15.7 Market News Analysis

- **Ingestion-side:** news stream → entity extraction → per-entity digest documents → index.
- **Query example:** "What has moved EURUSD and why?" → news digests (vector) + price action (time-series) fused.
- **Freshness need:** minutes for news; seconds for prices.

### 15.8 AML Transaction Monitoring Assistant

- **Live:** transaction alerts, current screening results.
- **Static:** typology knowledge base, red-flag indicators, past investigation write-ups (vector).
- **Query example:** "Is this structuring pattern worth escalating?" → live alert details + typology docs + similar past cases.
- **Freshness need:** seconds (alerts) / weeks (typologies).

### 15.9 Compliance Q&A over Live Policies

- **Ingestion-side:** policy updates streamed to the index via CDC/events (Architecture 1/2) — the compliance assistant always quotes the current version, and can answer "what changed in version 4.2?" via version history (§13.3).
- **Freshness need:** minutes.

---

## 16. Implementation Guidance

### 16.1 Reference Architecture

A complete streaming RAG stack, combining ingestion-side streaming with query-time live lookup:

```
SOURCE LAYER
  Postgres/Oracle (policies, customers) ──► Debezium (CDC) ──► Kafka
  Trading system ──► domain events ──► Kafka
  News/regulator feeds ──► ingest service ──► Kafka
                          │
STREAM LAYER            Kafka (topics per entity, keyed by ID, Schema Registry)
                          │
PROCESS LAYER   Kafka Streams / Flink: enrich, transform, derive documents,
                aggregates → Redis / ClickHouse (live state)
                          │
INDEX LAYER     Embedding service (async, batched, DLQ) → Vector store
                (Qdrant/Weaviate/Milvus) upsert/delete by doc ID
                          │
QUERY LAYER     RAG orchestration (LangChain / LlamaIndex / Haystack —
                see rag_frameworks_comparison_guide.md)
                ├─ vector search (indexed docs)
                ├─ live lookup (Redis/ClickHouse/API: price, exposure, status)
                └─ freshness filter + labeling
                          │
                          ▼
                       LLM → answer (with as-of labels)
```

### 16.2 Implementation Checklist

- [ ] **Choose event backbone** — Kafka (default), Pulsar, Redpanda, or managed (Kinesis/Event Hubs). Decide retention (replay window) and compaction strategy.
- [ ] **Set up CDC** — Debezium connectors per source table; outbox pattern if CDC is unavailable or business rules gate events; test snapshot + incremental phases.
- [ ] **Define document derivation** — events/rows → documents → chunks. Decide chunking scheme with stability in mind (§11.8); define doc IDs (`source:entity_id`) and chunk IDs.
- [ ] **Idempotency** — doc IDs deterministic; event IDs in metadata; version comparison on write (§11.2, §11.9).
- [ ] **Embedding batching** — batch size, rate-limit handling (backoff/retry), async workers, DLQ (§11.1).
- [ ] **Vector store upsert** — per-chunk versioning; `content_hash` for diffing; verify upsert semantics of the chosen store (§10.4).
- [ ] **Deletion handling** — tombstones → deletes; soft-delete policy; verify space reclamation (§11.3).
- [ ] **Query-time live lookup design** — define what is *live* vs *static*; per-source freshness budgets (§6.3); caching with TTL (§12.5).
- [ ] **Freshness monitoring** — lag metrics, staleness percentiles, last-updated timestamps (§12.4); SLOs defined per collection.
- [ ] **Failure handling** — queue backlog alerts, embedding failures → DLQ, consumer rebalance behavior, degradation policy (serve last-good index with staleness banner).
- [ ] **Rollback** — index versioning; ability to re-index from snapshot + CDC replay; shadow-index rebuild for model upgrades (§13.4).
- [ ] **Testing** — E2E streaming test (event → index → query), chaos tests (broker down, lag spike, duplicate delivery), idempotency test (replay the same event twice → same index state) (§16.5).

### 16.3 Sample Pipeline Pseudocode

```python
# --- CDC consumer + index worker (simplified) ---
def consume_change_events(topic, vector_store, embedder):
    for msg in kafka_consumer(topic):            # keyed by doc_id
        event = parse(msg.value, schema_version=msg.headers["schema_version"])
        if msg.value is None:                    # tombstone → delete
            vector_store.delete_chunks(doc_id=event.doc_id)
            continue
        if event.event_version < indexed_version(event.doc_id):
            continue                             # out-of-order guard (idempotent)
        doc = derive_document(event)             # event → document text
        chunks = chunk(doc)                      # stable, anchor-based chunking
        changed = diff_chunks(chunks, stored_hashes(event.doc_id))
        if changed:
            vectors = embed_batch(changed, retry=3, backoff=exp)   # async batch
            vector_store.upsert(
                points=[Point(id=chunk.chunk_id, vector=v,
                              payload={doc_id, doc_version, as_of,
                                       content_hash, source_event_id})
                        for chunk, v in zip(changed, vectors)])
        vector_store.delete_chunks(ids=removed_chunk_ids)          # partial update

# --- Query path ---
def answer(query, rag_orchestrator, live_sources):
    plan = rag_orchestrator.plan(query)          # decide which sources
    static = vector_search(query, filter=plan.static_filters)
    live = {name: src.fetch(plan.entities)       # parallel live lookups
            for name, src in live_sources.items() if plan.needs(name)}
    ctx = assemble_context(static=static, live=live,   # label + order + budget-check
                           freshness_budget=plan.budget)
    return llm.generate(prompt=build_prompt(query, ctx))
```

### 16.4 Streaming RAG with LangChain / LlamaIndex / Haystack

**LangChain**

- **Kafka message loader** — `KafkaChatMessageHistory` for conversational memory over Kafka; document loaders pull from topics/queues.
- **Streaming document loaders** — custom loader that consumes from Kafka/SQS and yields `Document` objects per message; the standard `VectorstoreIndexCreator`/`add_documents` accepts incremental additions.
- **Vector store upsert** — `vectorstore.add_documents(docs, ids=[...])` and `vectorstore.delete(ids=[...])`; Qdrant/Weaviate integrations accept deterministic IDs — re-adding the same ID overwrites (idempotent).
- **Live tools for agents** — `@tool`-decorated functions wrapping market-data APIs, DB lookups, Kafka consumers (Architecture 5).

**LlamaIndex**

- **Streaming ingestion** — `IngestionPipeline` with incremental `run()` calls (upsert mode); supports `doc_id`-based dedup via `Document.doc_id`.
- **Document versioning** — `Document` metadata carries `version`/`as_of`; `VectorStoreIndex.from_documents` with `show_progress`; async embeddings (`AsyncEmbedding`).
- **Query-time live** — `QueryEngineTool`/`FunctionTool` for live lookups; `VectorIndexRetriever` + custom `ToolRetriever` fusion.

**Haystack**

- **Document store streaming writes** — `DocumentStore.write_documents(documents, policy=DuplicatePolicy.OVERWRITE)`; Qdrant/Weaviate/Elasticsearch document stores support incremental writes and deletes by ID.
- **Pipeline composition** — DAG pipeline: `KafkaReader → PreProcessor → Embedder → DocumentWriter`; live lookups as custom components; `MetadataFilter` for freshness filtering.

**Vector store Python clients (pattern)**

```python
# Qdrant — deterministic point IDs, idempotent upsert, delete by doc_id
client.upsert(collection_name="docs", points=[
    PointStruct(id=chunk_id, vector=vec, payload=payload)
])
client.delete(collection_name="docs",
              points_selector=FilterSelector(filter=Filter(must=[FieldCondition(
                  key="doc_id", match=MatchValue(value=doc_id))])))

# Weaviate — batched import, replace by idempotent key
client.batch.configure(batch_size=200, dynamic=True)
with client.batch as batch:
    for doc in docs:
        batch.add_data_object(data_object=doc.payload, class_name="Doc",
                              uuid=doc.chunk_id)   # same uuid → replace
```

### 16.5 Testing Streaming RAG

- **E2E freshness test:** emit an event at `T`, assert the query returns the new content by `T + SLA`. Run continuously (canary) — this is the real SLO test.
- **Idempotency test:** replay the same event twice / deliver duplicates; assert index state unchanged (no duplicate chunks).
- **Ordering test:** deliver updates out of order; assert final state matches version ordering, not arrival order.
- **Delete test:** delete source row → tombstone → assert chunks gone from retrieval results.
- **Chaos tests:** broker down (consumer rebalance), lag spike (backpressure), embedding API 429 storm (retry/DLQ), CDC connector restart (replay). Assert: no corruption, recovery within SLA, DLQ captures poison messages.
- **Freshness-budget test:** inject an index with artificially old `as_of`; assert the orchestrator filters/rejects stale context.
- **Evaluation:** reuse the RAG evaluation stack (RAGAS etc. — see rag_frameworks_comparison_guide.md) with a *time-aware* golden set: queries whose correct answer depends on the current version of a doc, asserting the answer uses the freshest version.

---

## 17. Operational Concerns

### 17.1 Monitoring

| Metric | What it tells you | Typical alert threshold |
|---|---|---|
| Kafka consumer lag (per topic) | Pipeline backlog | > 2× freshness budget |
| Index staleness (max/p95 `now − as_of`) | Oldest retrievable content | > freshness SLA per collection |
| Embedding queue depth | Backpressure at embed stage | > capacity × recovery time |
| Upsert error rate | Store/derivation problems | > 0.1% sustained |
| DLQ depth | Poison events accumulating | > 0 (non-empty) |
| Query-time live lookup latency (p95) | Live-source health | > 500 ms |
| Live lookup error rate | Source availability | > 1% |
| Staleness SLO attainment | % of queries answered within budget | < 99.9% → investigate |

### 17.2 Alerting

- **Lag > threshold** — pipeline falling behind; freshness budget at risk.
- **Staleness > SLA** — users may be getting stale answers *right now*.
- **DLQ non-empty** — events silently not indexed; investigate payload/schema.
- **Embedding failures** — rate-limit storms or embedder outage.
- **Live lookup latency/errors** — Architecture 3 answers degrade.
- **Alert-to-answer mapping:** every alert should name the collection/topic and the freshness impact ("policies topic lag 4 min; policy answers up to 4 min stale").

### 17.3 Scaling

- **Partition scaling:** Kafka partitions per topic → parallel consumers; key by doc ID so scaling preserves per-doc ordering.
- **Embedding worker autoscaling:** KEDA on queue depth — scale embed workers with backlog, scale in when drained.
- **Vector store sharding/replication:** Qdrant/Weaviate/Milvus shard by doc ID; replication factor per availability requirements; watch for replication lag affecting read-your-writes (§11.5).
- **Capacity planning:** size embedding capacity for *peak change rate × chunks per change*, not average; batch for historical loads (hybrid, §13.1). Batch and streaming share the embedder budget — plan the batch window so it doesn't starve streaming deltas.

### 17.4 Disaster Recovery

- **Replay from Kafka:** with sufficient retention, rebuild the index by replaying topics (idempotency makes replay safe). Retention sizing = rebuild SLA vs storage cost.
- **Rebuild from source:** full snapshot + CDC replay — the ultimate recovery; needs CDC replay window large enough to cover the rebuild duration.
- **Point-in-time restore:** versioned index (or time-partitioned collections) lets you restore "as of T" for forensics/regulatory questions (§13.3).
- **Cross-region replication:** replicate Kafka topics and vector store (Qdrant/Weaviate/Milvus support replication) for region-loss recovery; plan the lag budget for failover (replicated index is slightly behind — acceptable if SLA allows).

### 17.5 Security

- **PII in streams — masking/redaction before indexing:** redact PII (names, account numbers, passport IDs) at the CDC/event boundary *before* chunks are embedded or stored. Embeddings of PII are still PII-ish (invertible-ish); mask first, index second. Tokenization for searchable-but-encrypted fields where needed.
- **Access control on live lookups:** the RAG query path is a privileged client — enforce per-user/per-role authorization on live lookups (a support bot must not read another customer's balance). Never let the LLM's tool layer bypass entitlement checks.
- **Audit logging:** log *what was retrieved, when, by whom* — vector retrieval, live lookups, and the final answer should be traceable for SR 11-7 / model-risk and regulatory inquiries.
- **Data residency:** keep streams and indexes in-region (MAS TRM for Singapore operations); CDC connectors must not exfiltrate data across borders; replication policies must respect residency requirements.

---

## 18. Comparison and Decision Framework

### 18.1 Batch RAG vs Streaming RAG

| Dimension | Batch RAG | Streaming RAG |
|---|---|---|
| **Freshness** | Hours (nightly/periodic) | Seconds (index) / sub-second (live lookup) |
| **Latency to reflect a change** | Minutes–hours | Seconds |
| **Infrastructure** | Scheduler + job runner | Kafka, CDC, stream processor, workers |
| **Cost** | Cheap to run, expensive to run often | More infra; cost proportional to change rate + query rate |
| **Complexity** | Low | High (ordering, idempotency, lag, DLQs) |
| **Consistency** | Strong (single batch build) | Eventual (streaming writes) |
| **Best for** | Stable corpora, historical Q&A, low change rate, tight budget | Fast-changing data, state-dependent answers, freshness SLA < minutes |

### 18.2 Choosing an Architecture

| Decision driver | CDC-based (Arch 1) | Event-driven (Arch 2) | Query-time live (Arch 3) | Stateful stream (Arch 4) | Agentic (Arch 5) | Serverless refresh (Arch 6) |
|---|---|---|---|---|---|---|
| **Data source type** | Relational DB rows | Domain events | Volatile state (prices, balances) | Event streams needing aggregates | Heterogeneous (all) | Files/objects (S3, CMS) |
| **Update frequency** | Any (captures all changes) | Any (business events) | N/A (fetch per query) | High (events) | N/A | Moderate, bursty |
| **Query patterns** | Doc-centric Q&A | Event-centric Q&A | State-centric ("what is…") | Aggregate-centric ("totals, alerts") | Mixed, conversational | Doc-centric |
| **Latency SLA** | Seconds–minutes | Seconds–minutes | Sub-second | Seconds | Seconds+ | Minutes |
| **Infra available** | DB + Kafka + workers | Event bus + processors | Live sources + orchestrator | Flink/Kafka Streams + state store | Orchestrator + tools | Serverless platform |

**Common combinations:** Arch 1 + 3 (indexed docs + live state), Arch 2 + 4 (event-driven indexing with stream-computed aggregates), Arch 5 as the query-time brain over any combination. Most banking deployments are hybrids — the decision table picks per *context type*, not per system.

### 18.3 When Streaming RAG vs Batch RAG

Choose **streaming** when: freshness SLA is minutes or less; data changes at high frequency (positions, prices, transactions); queries are recent-data-heavy ("what happened," "current status," "latest version"); the cost of a stale answer exceeds the cost of streaming infra.

Choose **batch** when: docs are stable (brochures, manuals, archived research); SLA is hours; change rate is negligible; the team cannot yet operate streaming infra — and revisit when the requirements move.

**Cost note:** streaming infra (brokers, CDC, workers, replication) is real money and real ops. The freshness budget is the *only* legitimate justification — if the business accepts a 6-hour staleness SLA, batch is the correct engineering answer (§20 anti-pattern: streaming everything).

---

## 19. Anti-Patterns to Avoid

| # | Anti-pattern | Why it fails | Fix |
|---|---|---|---|
| 1 | **Streaming everything** | Over-engineering; stable docs get Kafka pipelines, ops burden, and lag alerts for nothing | Freshness-budget-driven choice (§18.3); batch for stable, streaming for volatile |
| 2 | **Ignoring idempotency** | Duplicate events (at-least-once redelivery) → duplicate chunks → duplicate answers and bloated index | Deterministic doc IDs, version comparison (§11.2, §11.9) |
| 3 | **Full re-embed on every change** | A one-word edit re-embeds 50 chunks; embedding cost × change rate, embedder becomes the bottleneck | Chunk-level diffing + stable chunking (§11.4, §11.8) |
| 4 | **Unbounded index growth** | No deletes, no compaction — deleted docs and old versions accumulate forever; retrieval quality decays and cost grows | Tombstone handling, compaction, version retention policy (§11.3, §13.3) |
| 5 | **Synchronous embedding in the hot path** | Embedding latency added to ingestion latency; rate limits throttle the pipeline; backpressure stalls consumers | Async, batched embedding workers with queues (§11.1) |
| 6 | **Missing tombstones** | Deleted docs linger in the index and keep being retrieved — the stale-answer failure mode (§1.2) | Treat null-value messages as deletes; test delete E2E (§11.3, §16.5) |
| 7 | **No freshness metadata** | Cannot tell how fresh context is; no staleness filtering, no monitoring, no labeling | as_of/version/source_event_id on every chunk (§11.11) |
| 8 | **Query-time live lookups in the hot path without caching** | Every query pays full live-source latency; a burst of queries thrashes source systems | TTL caches for live lookups, semantic cache with versioned keys (§12.5) |
| 9 | **Ignoring watermark/ordering** | Out-of-order updates overwrite newer state with older state → wrong "current" values | Per-key ordering, event-time handling, version guards (§11.6, §11.9, §13.2) |
| 10 | **No staleness monitoring** | Lag grows silently; answers degrade from "seconds fresh" to "hours fresh" with nobody noticing | Index lag/staleness metrics + alerting (§12.4, §17.1) |

---

## 20. Conclusion

Streaming RAG closes the gap between what an LLM *knows* (its index) and what is *true* (the world). The two levers are complementary and most production systems pull both: **streaming ingestion** keeps the knowledge base continuously in sync (CDC → Kafka → embed → upsert, or events → derived documents), and **query-time live retrieval** guarantees sub-second freshness for state (prices, positions, balances, statuses) that no indexing pipeline can beat.

The engineering discipline is the same one the streaming world has already learned, applied to a new consumer: idempotent, versioned, ordered, monitored, replayable pipelines — with the vector index treated as just another derived projection of the event stream. For a bank, this is not an optimization: real-time risk, fraud, trading, and compliance questions are precisely the questions where "as of yesterday" is a wrong answer.

**Start small:** pick one collection with a real freshness SLA, run CDC → Kafka → embed → index, add query-time live lookup for the state that must be sub-second, instrument lag and staleness, and let the freshness budget — not the technology — decide where streaming stops and batch begins.

---

*Companion guides: [RAG Frameworks Comparison](rag_frameworks_comparison_guide.md) · [RAG vs HyDE](rag_vs_hyde_guide.md) · [RAG Optimization Techniques](rag_optimization_techniques_guide.md) · [Event Stream Processing](../../event_stream_processing_guide.md) · [Closed-Loop Data Engineering](../closed_loop_data_engineering_guide.md)*
