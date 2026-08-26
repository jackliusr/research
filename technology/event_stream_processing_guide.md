# Event Stream Processing: A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore  
> **Context:** Core Data Infrastructure / Architecture — Banking, Real-Time Systems, Event-Driven Architecture  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** July 2026

---

## Table of Contents

1. [What Is Event Stream Processing?](#1-what-is-event-stream-processing)
2. [Evolution of Data Processing Paradigms](#2-evolution-of-data-processing-paradigms)
3. [Core ESP Concepts and Terminology](#3-core-esp-concepts-and-terminology)
4. [Event Stream Processing Patterns](#4-event-stream-processing-patterns)
5. [ESP Technologies Comparison](#5-esp-technologies-comparison)
6. [Time Semantics and Windowing](#6-time-semantics-and-windowing)
7. [State Management and Fault Tolerance](#7-state-management-and-fault-tolerance)
8. [ESP in the Event-Driven Architecture](#8-esp-in-the-event-driven-architecture)
9. [ESP for Banking](#9-esp-for-banking)
10. [Pattern Detection and Complex Event Processing in Banking](#10-pattern-detection-and-complex-event-processing-in-banking)
11. [Conclusion](#11-conclusion)

---

## 1. What Is Event Stream Processing?

Event Stream Processing (ESP) is the continuous processing of unbounded streams of data records — events — in real-time, as opposed to batch processing where data is processed in discrete chunks at rest. ESP enables organizations to react to events as they happen — within milliseconds to seconds — rather than after they have been stored and processed in batch cycles.

### 1.1 Core Concepts

**Event.** An immutable record of something that happened, at a specific time, with specific data. An event is a fact — it cannot be changed, only superseded by later events. Examples: a credit card transaction, a sensor reading, a trade execution, a user clicking a button. Events are schema-defined (structure known ahead of time), timestamped (when the event occurred), and carry a payload (the event's data).

**Stream.** An unbounded, ordered sequence of events. "Unbounded" means the stream has no defined end — events arrive continuously. "Ordered" means events have a defined sequence (typically by timestamp or offset), though ordering may not be total across all partitions. Streams are the foundational abstraction in ESP.

**Stream Processing.** Continuous computation on one or more streams. Unlike a database query that runs once and returns results, a stream processing query runs forever, producing results incrementally as new events arrive. The computation is persistent and stateful — it remembers what it has seen across events.

### 1.2 How ESP Differs from Other Paradigms

| Paradigm | Data State | Latency | Example |
|---|---|---|---|
| **Batch Processing** | Data at rest (files, tables) | Minutes to days | Nightly ETL, MapReduce |
| **Request-Response** | Data queried on demand | Milliseconds | REST API, gRPC |
| **Message Queuing** | In-flight messages | Milliseconds | RabbitMQ, ActiveMQ |
| **ETL (Batch)** | Data at rest | Hours | SQL transforms on warehouses |
| **Event Stream Processing** | Data in motion | Sub-second to seconds | Flink, Kafka Streams, ksqlDB |

**Batch processing** operates on data at rest. You collect data over a period, then process it in a discrete job. Results are stale by definition — the freshest data in a nightly batch is hours old. Batch is appropriate for historical analysis, reporting, and computations where latency doesn't matter.

**Request-response** is synchronous — a client sends a request and waits for a response. The server processes the request immediately, typically by querying a database. This is the dominant pattern for APIs but does not scale to continuous event flows and breaks down when multiple consumers need the same data.

**Message queuing** delivers messages from producers to consumers, typically with point-to-point semantics (one message, one consumer). Queues provide reliable delivery, load leveling, and decoupling, but they are not stream processors — they do not compute on streams, maintain state, or support windowing and joins.

**ETL** is batch-oriented extract-transform-load. It moves data from source systems to warehouses, transforming it along the way. Streaming ETL is a hybrid — using stream processors to continuously transform data into warehouse-ready formats with sub-minute latency.

**ESP** is fundamentally different: it treats data as a continuous flow, applies transformations on the fly, maintains state across events, and produces results immediately. The computation never stops; it runs for the lifetime of the application.

### 1.3 The Four Questions of Stream Processing

Stream processing frameworks answer four fundamental questions:

**What** — computing on streams. What operations are performed? Map, filter, aggregate, join, pattern match? This is the transformation logic — the business rules applied to events.

**Where** — stream processor topology. Where does computation happen? Stream processing applications are structured as directed acyclic graphs (DAGs) of operators: sources (input streams), transformation operators (map, filter, window, join), and sinks (output streams). The topology defines the data flow.

**When** — time semantics. When is a result considered complete? This is the most subtle and important question. Event time (when the event occurred) differs from processing time (when the event was processed). Watermarks estimate event time completeness so the system knows when it is safe to emit results. Incorrect time semantics produce incorrect results.

**How** — state management and fault tolerance. How does the system remember what it has seen? How does it recover from failures without losing or duplicating work? State backends (RocksDB, in-memory), checkpointing, savepoints, and exactly-once semantics answer this question.

---

## 2. Evolution of Data Processing Paradigms

### 2.1 The Batch Era

For decades, data processing meant batch. Organizations ran nightly ETL jobs, monthly reports, quarterly reconciliations. The data warehouse was the center of gravity — data landed in staging tables, was transformed through dimensional models, and surfaced in dashboards and reports. This worked when "fresh enough" meant "yesterday's data."

**Limitations of batch-only processing:** Data always stale (freshest point was the last cutoff); operational decisions cannot use current data; fraud detection, alerting, and real-time monitoring impossible; infrastructure underutilized (peak during batch window, idle otherwise); time-to-insight measured in hours or days.

### 2.2 Lambda Architecture

The Lambda architecture (Nathan Marz, 2011) acknowledged that batch was insufficient for real-time needs by adding a "speed layer" alongside the batch layer. The batch layer processed all historical data for accuracy; the speed layer processed recent data with low latency for freshness. A serving layer merged results from both.

**Strengths:** Combined historical accuracy with real-time freshness; allowed incremental adoption of streaming; influenced design of many real-time systems.

**Weaknesses:** Two code paths — same logic in batch (Hadoop/Hive) and streaming (Storm) with different APIs and deployment; results from two layers must be merged with potential inconsistency; hard to reason about correctness; maintenance burden of two independent systems; speed layer could not easily reconsume old data.

### 2.3 Kappa Architecture

The Kappa architecture (Jay Kreps, 2014) proposed a radical simplification: use a single stream processing pipeline for all data. "Batch" is just a special case of streaming where the input stream happens to be bounded — historical data replayed from the beginning. Apache Kafka, as a distributed commit log that can replay data from any point, makes this possible.

**Strengths:** Single code path — one processing engine, one API, one deployment; historical reprocessing is trivial (rewind and replay); simplified operational model (one system); all data treated uniformly.

**Weaknesses:** Requires stream processor capable of handling historical volumes at real-time throughput; stream processors must support exactly-once and state management for large replays; mature implementations only became practical with Flink's savepoints and Kafka's log compaction.

### 2.4 Streaming-First / Event-Driven Architecture

The natural evolution of Kappa is the streaming-first architecture where all data flows through event streams as the primary representation. Kafka becomes the central data bus. Stream processors (Flink, ksqlDB, Kafka Streams) perform all transformations. Data products are event streams — consumers subscribe to the streams they need. There is no "load data to warehouse first" step.

**Key characteristics:** Kafka as the system of record for in-flight and recent data; stream processors (Flink, ksqlDB, Kafka Streams) for all transformations (ETL is streaming, not batch); data products published as event streams with defined schemas; consumers subscribe to streams instead of querying databases; databases become materialized views of event streams (event sourcing); the warehouse becomes a secondary store for historical analytics.

### 2.5 Key Milestones

| Year | Milestone | Significance |
|---|---|---|
| 2003-2004 | Google MapReduce / GFS | Batch processing at scale on commodity hardware. All modern systems descend from MapReduce's data-parallel model. |
| 2010 | Apache Kafka (LinkedIn) | Distributed commit log. Persistent, replayable, partitioned message storage. Now the de facto event backbone. |
| 2011 | Apache Storm (BackType/Twitter) | First open-source stream processor — DAG topology (spouts and bolts). Proved real-time streaming at scale. Limited by at-least-once and manual state management. |
| 2013 | Apache Samza (LinkedIn) | Stream processing on Kafka. First to couple processing with Kafka's partitioning. Local state with RocksDB backed by Kafka changelog. Influenced Kafka Streams. |
| 2014 | Apache Flink | True streaming with exactly-once — event-time, watermarks, Chandy-Lamport snapshots. Most sophisticated open-source stream processor. |
| 2015 | Apache Spark Structured Streaming | Micro-batch streaming as unbounded tables. Brought streaming to Spark with SQL-compatible API. Accepted ~100ms latency for API simplicity. |
| 2016 | Apache Beam (Google) | Unified batch+streaming API (Beam Model: what/where/when/how). Portable — write once, run on Flink/Spark/Dataflow. |
| 2018 | Kafka Streams (Confluent) | Stream processing as embedded library — no separate cluster. Accessible to every Java developer with a Kafka dependency. |
| 2019 | ksqlDB (Confluent) | SQL on Kafka Streams with pull/push queries. Accessible to analysts and data scientists. |
| 2020-present | Cloud-Native Stream Processing | RisingWave, Materialize, Flink on K8s, Kafka on K8s. Auto-scaling, rolling upgrades, declarative management on ephemeral infrastructure. |
| 2022-present | Real-Time Data Products / Data Mesh | Event streams as domain-owned data products with schema registry contracts, SLAs, and ownership. |

---

## 3. Core ESP Concepts and Terminology

### 3.1 Foundational Concepts

**Event (Record / Message).** An immutable fact — something that happened at a point in time. Events have a schema (defined structure), a timestamp (when the event occurred), and a payload (the event data). Events are never mutated; new events supersede old events.

**Stream.** An unbounded, ordered sequence of events. Streams are the primary abstraction in ESP — all computation operates on streams. Streams are typically partitioned for parallelism.

**Producer / Publisher.** The source of events — an application or system that writes events to a stream. Producers attach to a broker (e.g., Kafka) and publish records.

**Consumer / Subscriber.** The sink for events — an application or system that reads events from a stream. Consumers subscribe to topics and process records. Consumer groups enable parallel consumption partitioned streams.

**Broker.** The intermediate storage system that accepts events from producers, stores them durably, and serves them to consumers. Apache Kafka is the dominant event broker. Brokers provide persistence, replication, partitioning, and ordering guarantees.

**Stream Processor.** The compute engine that continuously processes streams. A stream processor connects to a broker (or reads directly from a source), applies transformations, and writes results to a sink or back to the broker.

**Topology.** The directed acyclic graph (DAG) of stream processors — sources, operators, and sinks connected by data flows. A topology defines the complete processing pipeline.

**Source.** An input to a topology — a stream of events consumed from a broker, file, socket, or other origin.

**Sink.** An output from a topology — results are written to a broker, database, file, or other destination.

**Operator.** A transformation function applied to events. Common operators include map, filter, flatMap, window, join, aggregate, and processFunction. Operators can be stateless (applied per-event) or stateful (maintain state across events).

**Key.** A partitioning unit that determines ordering guarantees and state grouping. Events with the same key are routed to the same partition, ensuring sequential processing and enabling partitioned state. In Kafka, keys are used to assign records to partitions.

**Partition.** A unit of parallelism in a stream. Streams are divided into partitions that can be processed independently. Partitions enable horizontal scaling — each partition can be processed by a different task/thread/machine. Within a partition, events are totally ordered.

**Offset.** A position in a stream, specific to a partition. Offsets are monotonically increasing integers that identify the event's position. Consumers track offsets to know which events have been consumed and where to resume after a restart.

### 3.2 Time Concepts

**Event Time.** The timestamp embedded in the event by the producer — when the event actually occurred in the real world. Event time is the semantically meaningful timestamp. For example, a credit card transaction's event time is when the card was swiped, not when the authorization message reached the processing system.

**Processing Time.** The timestamp assigned by the stream processor — when the event was processed. Processing time is the current system clock. It is the simplest to work with (no watermarking needed) but depends on system load and event arrival order. Results computed on processing time are not reproducible — replaying the same stream may produce different results.

**Ingestion Time.** The timestamp assigned when the event entered the system — typically when the broker receives the event (Kafka's record timestamp). A compromise between event time and processing time. The ingestion timestamp is consistent (all consumers see the same timestamp) and is not affected by delays before ingestion, but does not reflect when the event actually occurred.

**Watermark.** An estimate of event time completeness in a stream. A watermark with value T means "no events with event time < T will arrive after this point." Watermarks allow the system to determine when it is safe to emit window results. Practical watermarks are heuristic — they estimate completeness based on observed event time skew.

**Allowed Lateness.** A configurable threshold for how late events are accepted. If an event arrives with event time before the watermark plus the allowed lateness, it is still processed and may update window results. Events beyond the threshold are discarded or sent to a side output.

### 3.3 Windowing Concepts

**Window.** A temporal grouping of events. Windows define the boundaries for aggregation and computation — events within the same window are processed together. Without windows, unbounded streams cannot be aggregated (you cannot count "all events" on an unbounded stream).

**Tumbling Window.** Fixed-size, non-overlapping windows. Each event belongs to exactly one window. Example: 1-hour tumbling windows — events from 00:00-01:00 go to window 1, 01:00-02:00 to window 2, etc.

**Sliding Window.** Fixed-size, overlapping windows with a fixed slide interval. Each event belongs to multiple windows. Example: a 1-hour window sliding every 5 minutes — an event at 00:15 belongs to windows [23:15-00:15], [23:20-00:20], ..., [23:55-00:55].

**Session Window.** Gap-based, variable-size windows. A session window starts when an event arrives and ends after a defined period of inactivity (gap). All events within the gap period belong to the same session. Common for user sessions, website activity, and call center interactions.

**Global Window.** A single window spanning the entire stream lifetime. All events go into one window. Global windows must specify a trigger (when to emit results), since the window never closes naturally.

**Custom Window.** User-defined window logic for domain-specific windowing. Enables arbitrary window assignment and lifecycle.

**Trigger.** The condition that causes a window's results to be computed and emitted. Triggers can be event-time-based (when watermark passes window end), processing-time-based (every N seconds), element-count-based (every N elements), or compound (combinations of the above). Early firings emit preliminary results before the window closes. Late firings update results when late events arrive within the allowed lateness.

### 3.4 State and Consistency Concepts

**State.** Data maintained by a stream processor across events. State is what distinguishes stream processing from simple message passing. Examples: running counts, window buffers, machine learning model parameters, session data. State can be keyed (partitioned by key) or operator-scoped (shared across keys within a parallel instance).

**Checkpoint / Snapshot.** A consistent, point-in-time copy of all operator state and the current position in each source stream. Checkpoints enable recovery — after a failure, the processor restores from the last checkpoint and resumes processing from the saved source positions.

**Savepoint.** A manually triggered checkpoint that serves as a consistent snapshot for operational purposes: application version upgrades, parallelism changes, bug fix rollback. Savepoints are stored externally and can be used to start or stop applications.

**Exactly-Once Semantics.** The guarantee that each event is processed exactly once despite failures, even with restarts and reprocessing. Achieving exactly-once requires coordination across three layers: source (Kafka transactional producer / idempotent writes), state checkpoint (consistent save of operator state via distributed snapshots), and sink (idempotent writes or transactional sinks with two-phase commit).

**At-Least-Once Semantics.** Events may be reprocessed after failures but are never dropped. The state may be temporarily inconsistent (same event applied twice) but eventually reaches a correct state if operations are idempotent. Simpler and higher throughput than exactly-once.

**At-Most-Once Semantics.** Events may be dropped after failures but are never reprocessed. Highest performance, lowest consistency. Suitable for monitoring and alerting where occasional event loss is acceptable.

**Backpressure.** A mechanism for downstream components to signal upstream components to slow down when they cannot keep up. Backpressure prevents unbounded buffering and out-of-memory errors. Stream processors handle backpressure through bounded buffers, credit-based flow control, or reactive pull models.

**Schema Registry.** A central store for event schemas. The schema registry stores schemas (Avro, Protobuf, JSON Schema) and enforces compatibility rules (backward, forward, full) as schemas evolve. Producers register schemas; consumers fetch schemas to deserialize events. The schema registry is critical for maintaining contract compatibility between producers and consumers in event-driven architectures.

---

## 4. Event Stream Processing Patterns

### 4.1 Event-Driven Architecture Patterns

#### Event Notification

The simplest EDA pattern. A service publishes an event when something happens. Interested consumers receive the event and act independently. The producer knows nothing about consumers — it just fires events.

**Characteristics:** Loose coupling (producer and consumer fully decoupled); producer sets no expectation on consumer behavior; events carry minimal data — just notification; consumers fetch additional data if needed. Example: A `TradeExecuted` event published — risk, reporting, and compliance systems each react independently.

#### Event-Carried State Transfer

Events carry the full state of the entity involved, not just a notification. Consumers build their own local state from the event stream without needing to query the producer.

**Characteristics:** Events are self-contained (carry all data needed); consumers maintain local state mirroring the producer's; no back-channel queries needed; higher event throughput (larger payloads) but simpler consumer architecture. Example: Customer profile change event carries the complete record — downstream services update local caches without querying the customer master.

#### Event Sourcing

All changes to application state are captured as an append-only event stream. Current state is derived by replaying all events from the beginning. The event stream is the source of truth — the current state is a derived materialized view.

**Characteristics:** Complete audit trail (every state change recorded); temporal queries (reconstruct state at any point); debugging/forensics (replay to understand what happened); CQRS integration (write model = event stream, read model = replay); no destructive operations (events appended, never updated/deleted). Example: A ledger records every transaction as an event — account balance is summing all transactions. To undo, append a compensating transaction; never delete the original.

**Trade-offs:** Event store size grows unboundedly (snapshotting mitigates this); replaying from scratch is expensive for long-lived entities (snapshot+delta pattern); schema evolution must handle old events with old schemas.

#### CQRS — Command Query Responsibility Segregation

Separate the write model (commands) from the read model (queries). Commands are handled by the command side, which produces events. Events are consumed by the query side to build materialized views optimized for reading.

**Characteristics:** Commands change state and produce events; queries read from materialized views (never command side); events bridge command side to query side; two sides scale independently with different data models; command side uses write-optimized stores; query side uses read-optimized stores (caches, materialized views, search indexes).

**Example:** In a payment system, the command side accepts `AuthorizePayment` commands, validates them, and produces `PaymentAuthorized` events. The query side consumes these events and updates a materialized view of current payment status. The risk dashboard queries the materialized view, never the command-side system.

### 4.2 Stream Processing Patterns

#### Filter

Pass through only events that match a condition. The simplest stream processing pattern — no state, no windowing.

**Example:** Filter out all wire transfer events below $10,000 (non-reportable threshold). Only events above $10,000 proceed for further processing.

```java
stream.filter(event -> event.getAmount() > 10000)
```

#### Map / Enrich

**Map / Enrich.** Transform or augment each event. Map applies a pure function (one event in, one event out). Enrichment adds data from an external source — typically a stream-table join with a reference data stream or lookup table. Example: enrich a trade event with counterparty credit rating from a reference data stream.

```java
trades
  .keyBy(trade -> trade.getCounterpartyId())
  .connect(counterpartyRatingStream)
  .flatMap(new EnrichWithRating())
```

#### Windowed Aggregation

Compute counts, sums, averages, minimums, maximums over time windows. Windowed aggregation is the backbone of real-time analytics. Examples: count of transactions per merchant per hour (tumbling), 15-minute moving average of trade volume (sliding), sum of payment amounts by currency per hour.

```sql
SELECT COUNT(*), AVG(amount), merchant_id
FROM transaction_stream
GROUP BY TUMBLE(event_time, INTERVAL '1' HOUR), merchant_id
```

#### Stream-Stream Join

Correlate two event streams based on a key and a temporal condition. Stream-stream joins require windowing — events from both streams within the same window are matched.

**Example:** Join a `TransactionStream` with an `AuthorizationStream` on `transactionId` within a 5-minute window to detect authorizations that never resulted in a completed transaction.

```java
transactions
  .join(authorizations)
  .where(t -> t.getTransactionId())
  .equalTo(a -> a.getTransactionId())
  .window(TumblingEventTimeWindows.of(Time.minutes(5)))
  .apply(new JoinFunction<>() { ... })
```

**Join types:** Inner join (both sides present), left outer join (left side present, right side optional), right outer join, full outer join.

#### Stream-Table Join

Enrich a stream with lookup data. The "table" can be a static dataset, a slowly updating reference data stream, or a database changelog stream (CDC).

**Example 1 — Static enrichment:** Enrich transaction stream with country ISO codes from a static reference data table.

**Example 2 — CDC enrichment:** Join the trade stream with the latest customer risk rating from a database changelog stream (Debezium CDC from the customer database).

```java
trades
  .keyBy(trade -> trade.getCustomerId())
  .connect(customerRiskCdcStream)
  .process(new EnrichWithLatestRisk())
```

#### Pattern Detection / Complex Event Processing

Detect sequences of events matching a specified pattern. Used extensively in fraud detection, trade surveillance, AML monitoring, and operational anomaly detection.

**Example:** Detect potential money laundering — a deposit followed by immediate withdrawal within 5 minutes, where the deposit amount is just below the reporting threshold ($10,000).

```java
Pattern.<Transaction>begin("deposit")
  .where(t -> t.getType() == DEPOSIT && t.getAmount() < 10000)
  .next("withdrawal")
  .where(t -> t.getType() == WITHDRAWAL && t.getAmount() > t.getAmount("deposit") * 0.9)
  .within(Time.minutes(5))
```

#### Anomaly Detection

Identify unusual events or patterns in streaming data. Anomaly detection can be threshold-based (simple rules), statistical (z-score, moving average deviation), or ML-based (isolation forest, autoencoders running on streaming features).

**Examples:**
- Flag a transaction amount as anomalous if it exceeds 3 standard deviations from the customer's 30-day rolling average
- Alert when API error rate exceeds 5% over a 5-minute window
- Detect sudden account activity changes from a dormant account

#### Dead Letter Queue

Route processing failures to a separate stream (DLQ — Dead Letter Queue) for investigation and reprocessing. The main processing pipeline continues uninterrupted. The DLQ preserves the original event, the error, and metadata.

**Example:** A stream processor validates payment messages. Messages that fail validation are sent to a `payment-dlq` topic with the validation error attached. Operations teams monitor the DLQ and can reprocess fixed messages.

#### Event Rekeying

Repartition a stream by a different key than the original. Rekeying changes the partitioning and ordering guarantees.

**Example:** A transaction stream is partitioned by `transactionId`. You rekey it by `accountId` so that all transactions for the same account go to the same partition for account-level processing.

```java
stream
  .map(tx -> /* parse and extract accountId */)
  .keyBy(event -> event.getAccountId())
```

#### Branching (Split)

Split a stream into multiple streams based on conditions. Each branch receives only events matching its condition.

**Example:** Split a transaction stream into three branches: `highValueTransactions` (amount > $1M), `mediumValueTransactions` ($10K - $1M), `lowValueTransactions` (below $10K). Each branch undergoes different processing.

#### Merging (Union)

Combine multiple streams with the same schema into one. Merging is the inverse of branching.

**Example:** Merge SWIFT MT103, SEPA, and domestic wire transfer streams into a unified payment stream for processing.

```java
stream1.union(stream2, stream3)
```

#### Deduplication

Remove duplicate events within a window. Deduplication requires state — the system remembers event IDs it has seen within the dedup window.

**Example:** Deduplicate payment instructions based on `paymentId` within a 24-hour sliding window. The second arrival of the same paymentId is silently dropped.

```java
stream
  .keyBy(event -> event.getPaymentId())
  .process(new DeduplicateWithinWindow(Time.hours(24)))
```

#### Sessionization

Group events into sessions based on gaps in activity. Sessions are variable-length windows closed by a period of inactivity.

**Example:** Group website clickstream events into user sessions — a session starts with the first event and ends after 30 minutes of inactivity.

```sql
SELECT userId, COUNT(*) AS pageViews, SESSION_START(session, INTERVAL '30' MINUTE)
FROM clickstream
GROUP BY SESSION(event_time, INTERVAL '30' MINUTE), userId
```

#### Stateful Computation

Maintain state across events for counters, aggregators, machine learning model inference, or any computation that needs to remember previous events.

**Examples:**
- Running counter of transactions per account (updated on each event)
- Rolling feature vector for ML model inference (updated on each event, model scores on demand)
- State machine tracking the lifecycle of a payment (initiated → authorized → settled → failed)
- Cache of recent events for pattern matching

#### Materialized View

Maintain a real-time, always-up-to-date view derived from an event stream. Materialized views are the serving layer for stream processing — they make streaming results queryable.

**Examples:**
- A real-time ledger showing current balance per account, updated as transactions arrive
- A dashboard showing current fraud detection metrics (transactions scored, alerts generated, false positive rate)
- ksqlDB pull queries against a materialized view for low-latency lookups

#### Schema Evolution

Handle changes in event schema over time without breaking downstream consumers. Schema evolution is managed by the schema registry, which tracks schema versions and enforces compatibility rules.

**Backward compatible:** New schema can read data written with the old schema (field added with default).
**Forward compatible:** Old schema can read data written with the new schema (field added — old consumer ignores unknown fields).
**Full compatible:** Both backward and forward compatible.

#### Event Versioning

Strategies for managing event schema changes over time. Common approaches:
- **Versioned events with schema registry:** Each event carries a schema version identifier. Consumers choose how to handle each version.
- **Upcasting / Schema migration:** Old events are rewritten (upcast) to the latest schema version when read. The event store contains only the latest schema version.
- **Dual writing:** During a transition period, producers write both old and new format events. Consumers migrate from old to new.

### 4.3 Stream Processing with State

#### Local State

State is stored embedded in the stream processor using an embedded state backend (RocksDB, in-memory heap, or filesystem). State is partitioned by key — each parallel task processes a subset of keys, keeping its state local.

**Characteristics:**
- Low latency — microseconds for state access
- High throughput — no network round-trips
- State is tied to the task instance — if the task moves, state must be migrated
- Scaling requires state redistribution (rebalancing)

**Embedded backends:**
- **RocksDB:** The most common production state backend. Disk-based, uses LSM-tree for efficient writes. Supports incremental checkpointing. Can handle terabytes of state per node. Requires careful tuning (memory, compression, bloom filters, compaction).
- **In-memory (Heap):** Fastest but not durable. State lost on failure. Suitable for small, transient state or development.
- **Filesystem:** Simple file-based storage. Limited to small state sizes. Useful for development and testing.

#### Remote State

State is stored in an external system — Redis, DynamoDB, Cassandra, or a relational database. The stream processor queries remote state on demand.

**Characteristics:**
- Higher latency — milliseconds or more for state access
- State and compute scale independently
- State is shared across tasks — any task can access any key's state
- Simpler scaling — no state migration during rebalancing
- More operational complexity — another system to manage

**When to use remote state:**
- State is too large for local storage (hundreds of TB)
- State needs to be shared across multiple stream processing applications
- The team already has expertise in the remote store
- State access patterns favor external query (sporadic, not per-event)

#### Hybrid State / Checkpointing

The most common pattern in production Flink deployments. Local state (RocksDB) provides fast access. Checkpoints are written to durable remote storage (S3, HDFS, GCS). On recovery, the system restores local state from the remote checkpoint.

**Characteristics:**
- Low latency for state access (local RocksDB)
- Durable recovery (checkpoints on S3)
- Incremental checkpointing only sends changes, not full state
- More complex tuning — RocksDB memory, checkpoint interval, storage cost

**Effect of state backend choice:**

| Factor | Local State | Remote State | Hybrid |
|---|---|---|---|
| Latency | Microseconds | Milliseconds+ | Microseconds (local access) |
| Throughput | Very high | Network-bound | Very high |
| State size | TB per node (RocksDB) | Virtually unlimited | TB per node (RocksDB) |
| Scaling | State migration needed | No state migration | State migration needed |
| Shared state | No — partitioned by key | Yes — any task can access | No |
| Operational complexity | Moderate (RocksDB tuning) | Higher (external stores) | Moderate + checkpoint store |
| Recovery | From checkpoint | From remote store | From checkpoint |
| Cost | Local disk | Network I/O + remote storage | Local disk + durable storage |

---

## 5. ESP Technologies Comparison

### 5.1 Apache Kafka

**Type:** Distributed event streaming platform (publish-subscribe, durable storage, replay, partitioning)

Kafka is the backbone of most event-driven architectures — a distributed commit log that stores events durably in partitioned topics with ordering, exactly-once semantics, and replay capability. Kafka Connect provides source/sink integration, and Kafka Streams (library) plus ksqlDB (SQL) add stream processing on top.

**Strengths:**
- Durable event store with configurable retention — replay from any offset
- High throughput (millions of messages/second), ordered, partitioned, replicated
- Kafka Streams runs embedded — no separate processing cluster
- Mature ecosystem: hundreds of connectors, Schema Registry, REST Proxy, large community

**Weaknesses:**
- Not a true stream processor — minimal windowing, limited CEP, no savepoints
- Kafka Streams best for simple-to-moderate processing; complex stateful processing is harder
- State management less sophisticated than Flink (RocksDB tuning, checkpoint depth, savepoints)
- Operational complexity at scale (brokers, KRaft/ZooKeeper, rebalancing, mirroring)

**Best for:**
- Event bus / message backbone, event sourcing / CQRS implementations
- Simple transforms, joins, aggregations via Kafka Streams
- Log compaction for keyed state, data integration via Kafka Connect

### 5.2 Apache Flink

**Type:** Unified stream and batch processing engine (true streaming, event-time, exactly-once)

Flink is the most sophisticated open-source stream processor. It treats streaming as a first-class primitive (not micro-batches), supports event-time processing with watermarks, provides exactly-once state consistency, and offers the richest windowing and CEP capabilities.

**Key capabilities:**
- True streaming (not micro-batch) — processes each event as it arrives
- Event-time processing with watermarks, allowed lateness, side outputs
- Exactly-once semantics for state and output (including end-to-end)
- Rich windowing: tumbling, sliding, session, global, custom windows with arbitrary triggers
- Complex Event Processing (CEP) library with NFA-based pattern matching
- Savepoints for application versioning, upgrades, parallelism changes
- SQL and Table API for declarative stream processing
- Batch on top of streaming (treats batch as bounded stream)
- Rich set of connectors (Kafka, Kinesis, Pulsar, filesystems, databases)
- PyFlink for Python stream processing
- Diverse deployment options (standalone cluster, YARN, Kubernetes, AWS KDA, Confluent Cloud)

**Strengths:**
- Best-in-class event-time handling — watermarks, late data, out-of-order events
- Most sophisticated stream processing engine — richest windowing and CEP, exactly-once end-to-end
- Savepoints for operational flexibility — upgrade, rollback, rescale without data loss
- Mature Kubernetes deployment (Flink Operator) and strong community (Alibaba, Uber, Netflix, ING)

**Weaknesses:**
- Steeper learning curve — significant conceptual overhead
- Separate cluster to manage unless using managed Flink (KDA, Confluent Cloud, Ververica)
- State management can be complex — RocksDB tuning, checkpointing costs, state size management
- Overkill for simple processing (filter, stateless map) — Kafka Streams is simpler
- Java/Scala primary SDKs; PyFlink and SQL lag in feature parity

**Best for:**
- Complex event processing / pattern detection
- Fraud detection and real-time scoring
- Real-time ML feature computation and model inference
- Stateful stream processing requiring exactly-once semantics
- Event-time processing with late data and out-of-order events
- Applications needing operational flexibility (savepoints, rolling upgrades)
- High-throughput, low-latency streaming at scale

### 5.3 Apache Spark Structured Streaming

**Type:** Micro-batch stream processing engine (treats streams as unbounded tables)

Spark Structured Streaming brought streaming to the Spark ecosystem by treating streams as "unbounded tables" with SQL-compatible semantics. It shares the DataFrame/Dataset API with Spark's batch processing, enabling code reuse between batch and streaming jobs.

**Key capabilities:**
- Micro-batch processing model (default) with continuous processing mode (experimental)
- SQL-compatible API (streaming DataFrames registered as tables)
- Same batch/streaming API — DataFrame/Dataset transformations work for both
- Spark ecosystem integration — MLlib for streaming ML, GraphX, Spark SQL
- Kafka, Kinesis, and file source/sink connectors
- Watermark support for event-time processing
- Stateful streaming via mapGroupsWithState and flatMapGroupsWithState

**Strengths:**
- Unified batch and streaming API — same DataFrame transformations work for both
- Easy for existing Spark users; SQL is first-class
- Rich ecosystem integration: MLlib, Delta Lake, Spark SQL

**Weaknesses:**
- Micro-batch latency (~100ms minimum) — not suitable for sub-100ms use cases
- Not true streaming — events buffered into micro-batches
- Event-time handling less sophisticated than Flink; state management less flexible (mapGroupsWithState)
- Continuous processing mode (sub-millisecond) experimental, rarely used in production

**Best for:**
- Near-real-time processing where 5-second+ latency is acceptable
- Teams already invested in the Spark ecosystem
- ETL-style stream processing (transform, filter, write to warehouse)
- ML inference on streams using Spark MLlib models
- Use cases that benefit from the unified batch/streaming API

### 5.4 Apache Beam

**Type:** Unified batch and streaming programming model (portable API layer, runner-agnostic)

Beam provides a portable API (the Beam Model — what/where/when/how) that runs on multiple runners: Flink, Spark, Google Dataflow, Samza. Write once, run on any supported runner. SDKs in Java, Python, Go, and SQL.

**Strengths:**
- Runner portability — avoid vendor lock-in; choose runtime per workload
- Clean, well-designed API — the Beam Model is the reference model for stream processing
- Unified batch/streaming — same code for bounded and unbounded data

**Weaknesses:**
- Abstraction overhead — cannot use runner-specific features without losing portability
- Runner-specific behavior differences — abstraction layer cannot hide all implementation details
- Smaller community than Flink or Spark; Python SDK lags behind Java

**Best for:**
- Multi-platform deployments, runner flexibility
- Google Cloud Dataflow users wanting portability options
- Organizations standardizing on the Beam Model

### 5.5 ksqlDB

**Type:** Stream processing with SQL on Kafka (built on Kafka Streams)

ksqlDB provides a SQL interface for stream processing on Kafka. It is built on top of Kafka Streams, inheriting its tight integration with Kafka's partitioning and exactly-once semantics. ksqlDB supports both pull queries (query current state) and push queries (subscribe to stream changes).

**Key capabilities:**
- SQL interface for stream processing
- Pull queries — query current state of materialized views (low-latency lookups)
- Push queries — subscribe to continuous query results (push-based streaming)
- Stream and table abstractions (stream = append-only, table = changelog/current state)
- Built on Kafka Streams — inherits exactly-once, partitioning, state management
- Connectors for external systems (JDBC, Elasticsearch, S3, and others)
- Schema Registry integration for Avro, Protobuf, JSON Schema

**Strengths:**
- SQL interface accessible to analysts and data scientists
- Tight Kafka integration — no separate processing cluster
- Pull queries for low-latency lookups on materialized views
- Built on Kafka Streams — inherits exactly-once, partitioning, state management

**Weaknesses:**
- Less powerful than Flink for complex processing — no CEP library, limited windowing
- SQL-only — limited expressiveness for complex logic
- Smaller ecosystem; not for sub-millisecond event-time processing

**Best for:**
- Simple to moderate stream processing with SQL
- Kafka-native teams
- Operational analytics — real-time dashboards and monitoring on Kafka data
- Quick streaming transformations without standing up a Flink cluster
- Materialized views on Kafka streams with pull query access patterns

### 5.6 RisingWave

**Type:** Cloud-native streaming database (SQL-based stream processing, PostgreSQL-compatible)

RisingWave is a newer entrant in the stream processing space, positioning itself as a "streaming database" with PostgreSQL compatibility. It provides SQL-based stream processing with materialized views that stay up-to-date as data flows in.

**Key capabilities:**
- PostgreSQL-compatible SQL interface
- Real-time materialized views on streaming data
- Cloud-native architecture (Kubernetes-native, compute-storage separation)
- Exactly-once processing semantics
- Combined real-time + batch processing on the same system
- Rich set of connectors (Kafka, Kinesis, Pulsar, databases, and filesystems)

**Strengths:**
- PostgreSQL compatibility — familiar SQL, easy migration for PG users
- Cloud-native (compute-storage separation, auto-scaling)
- Real-time materialized views without manual state management
- Lower operational complexity than Flink clusters

**Weaknesses:**
- Newer project — smaller ecosystem, fewer connectors, smaller community
- Less mature for complex stateful processing
- Windowing support still evolving
- Smaller community means fewer resources, fewer production case studies

**Best for:**
- Real-time analytics on streaming data
- PostgreSQL teams wanting to adopt stream processing
- Simpler stream processing workflows with SQL

### 5.7 Technology Comparison Table

| Factor | Kafka | Flink | Spark Streaming | Beam | ksqlDB | RisingWave |
|---|---|---|---|---|---|---|
| **Processing** | Pub-sub + embedded lib | True streaming | Micro-batch | Abstraction layer | SQL on Kafka Streams | SQL streaming DB |
| **Latency** | Sub-ms (transport) | Sub-ms to seconds | ~100ms min | Runner-dep. | Sub-ms to seconds | Sub-ms to seconds |
| **State Mgmt** | RocksDB (Kafka Streams) | RocksDB, heap; most sophisticated | mapGroupsWithState | Runner-dep. | Kafka Streams RocksDB | System-managed |
| **Exactly-Once** | End-to-end (transactions) | End-to-end | End-to-end | Runner-dep. | End-to-end | Supported |
| **Event-Time** | Basic (Kafka Streams) | Best-in-class | Supported | Runner-dep. | Basic | Supported |
| **Windowing** | Tumbling, sliding, session | All types + custom | Tumbling, sliding, session | All + custom (runner-dep.) | Tumbling, hopping, session | Tumbling, hopping |
| **CEP** | Via custom code | Flink CEP library | Via custom code | Runner-dep. | SQL MATCH_RECOGNIZE | SQL MATCH_RECOGNIZE |
| **SQL** | ksqlDB (separate) | Table API, SQL Client | Native SQL | Beam SQL (limited) | Native SQL | PG-compatible |
| **Deployment** | Kafka + embedded | Cluster / K8s | Spark cluster | Runner-dep. | Kafka + embedded | K8s-native |
| **Learning Curve** | Moderate | Steep | Moderate (if Spark-exp.) | Moderate | Low (SQL) | Low (SQL) |
| **Community** | Very large | Large | Very large | Moderate | Moderate | Small (growing) |
| **Best For** | Event backbone, simple transforms, event sourcing | Complex, stateful, CEP, fraud detection | Near-real-time, Spark ecosystem | Multi-runner portability | SQL analytics on Kafka | Real-time analytics, PG teams |

---

## 6. Time Semantics and Windowing

### 6.1 Time Semantics

Time is the most subtle concept in stream processing. Choosing the right time semantics determines whether your results are correct.

#### Event Time

When the event actually occurred — the timestamp embedded in the event by the producer. Accurate — results are correct regardless of event arrival. Reproducible — replaying the same stream produces the same results. Handles out-of-order events and late arrivals. Challenges: requires watermarks, out-of-order buffering, late data strategies, added complexity.

**When to use:** Almost always. Any use case where result accuracy matters — fraud detection, analytics, reporting, regulatory compliance.

**When to use processing time:** Monitoring and alerting where absolute accuracy is unnecessary. Use cases where "right now" is the right semantic — current rate, current latency, current error count.

**When to use ingestion time:** When event time is unavailable or unreliable but processing time is too inaccurate. Kafka's default timestamp is ingestion time (when the broker received the record). Good for measuring system latency.

### 6.2 Window Types

Windows group events into finite buckets for aggregation. The choice of window type determines the semantics of your results.

#### Tumbling Windows

Fixed-size, non-overlapping windows. Each event belongs to exactly one window.

```
[00:00 - 01:00)  [01:00 - 02:00)  [02:00 - 03:00) ...
```

- **Use cases:** Hourly reports, daily totals, periodic snapshots.
- **Characteristics:** Simple, predictable, no overlap. Easy to reason about and implement.
- **Example:** Count of transactions per hour.

#### Sliding Windows

Fixed-size, overlapping windows with a fixed slide interval. Each event belongs to multiple windows.

```
Window 1: [00:00 - 01:00)
Window 2: [00:05 - 01:05)
Window 3: [00:10 - 01:10) ...
```

- **Use cases:** Moving averages, rolling metrics, trend detection.
- **Characteristics:** Smooth output (no hard window boundaries). Each event processed for every window it falls into — more computational overhead.
- **Example:** 15-minute moving average of trade volume, updated every 1 minute.

#### Session Windows

Gap-based, variable-size windows. A session window starts with an event and closes after a period of inactivity.

```
[event1 -- gap -- event2 -- event3]  ...  [event4]  ...  [event5 -- event6]
```

- **Use cases:** User sessions, website behavior analysis, customer journey tracking.
- **Characteristics:** Variable-size windows. No regular schedule — windows are event-driven. Gaps define boundaries.
- **Example:** Group clickstream events into sessions with a 30-minute inactivity timeout.

#### Global Windows

All events in a single window. Must specify a trigger for emission.

- **Use cases:** Global aggregations (total unique visitors, running total).
- **Characteristics:** Window never closes naturally — trigger controls when results are emitted. Usually combined with early and late firings.
- **Example:** Running total of all transactions, with results emitted every minute.

#### Custom Windows

User-defined window assignment and lifecycle for domain-specific requirements.

- **Use cases:** Custom business calendars, non-uniform windows, event-driven windows.
- **Characteristics:** Full control over window assignment, merging, and lifecycle. Maximum flexibility, maximum complexity.

### 6.3 Triggers

Triggers determine *when* a window's results are computed and emitted.

**Event time trigger:** Emit when the watermark passes the window end. The default and most common trigger — results are emitted when the system believes all events in the window have arrived.

**Processing time trigger:** Emit at regular processing time intervals regardless of event time progress. Useful for periodic "heartbeat" output.

**Element count trigger:** Emit after N elements have been received for the window. Useful for batching for efficiency when event count matters more than time.

**Early and late firings:** Early firings emit preliminary results before the window closes (before watermark passes window end). Late firings update results when late events arrive within the allowed lateness. Together they provide incremental results: early results for freshness, on-time results for final accuracy, late updates for completeness.

**Compound triggers:** Combine multiple triggering conditions. Example: emit early results every minute, final result on watermark, and update for late events within 1 hour.

### 6.4 Watermarks in Depth

A watermark is an estimate of event time completeness. It tells the stream processor: "I have seen all events with event timestamp <= watermark value."

**Perfect watermarks:** Guarantee that no event with event time < watermark will ever arrive. Impossible to achieve in practice without infinite buffering or oracle knowledge of event ordering.

**Heuristic watermarks:** Best-guess estimate based on observed event time skew. The processor observes the gap between event time and processing time and estimates the maximum expected skew. This is the common approach in production — it may emit results before all events have arrived, but the allowed lateness threshold catches most late events.

**Periodic watermarks:** Computed at regular intervals by the stream processor. The watermark is updated every N milliseconds based on the maximum event timestamp seen so far, minus an expected delay.

**Punctuated watermarks:** Inserted into the stream at specified intervals. A watermark event is published as a special record. The processor can use these as progress indicators.

**Idle source handling:** If a source partition receives no events for a period, its watermark stalls — it cannot advance because no new data provides a timestamp to advance from. The processor must handle idle sources by either marking them idle (and ignoring them for watermark computation) or assuming progress after a timeout.

### 6.5 Late Data and Out-of-Order Events

**Late data** arrives after the watermark has passed the event's timestamp. Strategies:

1. **Allowed lateness:** Configure a threshold for how late events are accepted. Events within this threshold update window results (if the window has already been emitted, it is retracted and re-emitted with updated results).
2. **Side output:** Route very late events to a separate stream for monitoring and offline processing.
3. **Discard:** Silently drop events beyond the lateness threshold.

**Out-of-order events** arrive in non-event-time order. Strategies:

1. **Buffering:** The stream processor buffers events until the watermark advances, then emits events in event time order.
2. **Allowed lateness:** Configure how long to wait for stragglers before closing a window.
3. **Side outputs:** Send extreme outliers to a side stream.

**Example: Fraud detection window configuration:**

```
{sliding window: 5 minutes, slide: 1 minute, allowed lateness: 30 seconds}
```

This configuration computes a rolling count of transactions over the last 5 minutes, updated every minute, accepting authorization messages up to 30 seconds late. If an authorization arrives 40 seconds after the transaction, it is routed to a side output for investigation.

---

## 7. State Management and Fault Tolerance

### 7.1 State Types

**Keyed State.** State partitioned by key. Each key has its own independent state. Operations on keyed state are scoped to the current key being processed. Keyed state is the most common form of state in stream processing.

Types of keyed state:
- **ValueState:** Single value per key (e.g., running counter).
- **ListState:** List of elements per key (e.g., recent transactions buffer).
- **MapState:** Key-value map per key (e.g., session attributes).
- **ReducingState:** Single value that aggregates incrementally using a ReduceFunction (e.g., running sum).
- **AggregatingState:** Like ReducingState but with an intermediate accumulator type (e.g., running average with count + sum accumulator).

**Operator State.** State scoped to one parallel instance of an operator. Shared across all keys processed by that instance. Less common but useful for specific patterns.

Types of operator state:
- **ListState:** List of elements per operator instance.
- **UnionListState:** Like ListState but on restore, partitions are distributed using a union (all states are merged and redistributed).
- **BroadcastState:** State that is replicated across all parallel instances. Used for broadcast patterns (e.g., distributing a reference data snapshot to all tasks).

### 7.2 State Backends

**In-Memory (HashMapStateBackend).** State stored in the JVM heap. Fastest access but not durable — state is lost on failure (unless checkpoints are enabled, in which case state is checkpointed but stored in heap). Maximum state size is limited by heap memory. Best for development, testing, and small state.

**Filesystem / RocksDB (EmbeddedRocksDBStateBackend).** State stored in local RocksDB instances. RocksDB is an embedded key-value store using an LSM-tree for efficient disk-based storage. Supports incremental checkpointing. Can handle terabytes of state per node. The standard production state backend.

**Checkpoint storage (durable):** State is checkpointed to durable storage (S3, HDFS, GCS). The checkpoint store holds distributed snapshots of operator state. On recovery, local state is restored from the checkpoint. Common patterns: RocksDB as local state backend with S3 for checkpoint storage.

### 7.3 Exactly-Once Semantics

End-to-end exactly-once requires coordination across three layers:

**Source:** The source must support replay without duplication. Kafka's transactional producer and idempotent writes enable exactly-once source semantics. The stream processor commits source offsets atomically with state checkpoints — on recovery, it resumes from the committed offset.

**State checkpoint:** The stream processor takes a consistent snapshot of all operator state and the current source positions. In Flink, this uses the Chandy-Lamport distributed snapshot algorithm with barrier alignment. On recovery, all operators are restored from the latest checkpoint, and source positions are rewound to the checkpoint's offsets.

**Sink:** The sink must be idempotent or transactional. Idempotent sinks (e.g., writes to a KV store) can safely receive the same event multiple times. Transactional sinks use a two-phase commit protocol (e.g., Kafka's exactly-once sink via the transactional producer — writes are committed only when the checkpoint succeeds).

### 7.4 Checkpointing

**Aligned checkpoints:** The stream processor inserts checkpoint barriers into the stream. When an operator receives a barrier from one input, it waits for barriers from all other inputs before taking a snapshot. This ensures consistency but adds latency (barrier alignment can hold up processing).

**Unaligned checkpoints:** Barriers are processed immediately — the operator does not wait for all input barriers. Instead, in-flight events between the barrier position and the operator are included in the checkpoint. Reduces checkpoint time but increases checkpoint size.

**Incremental checkpointing:** Only state changes since the last checkpoint are persisted. RocksDB supports incremental checkpoints by tracking which SST files have changed. This reduces checkpoint time and storage cost significantly for large state.

### 7.5 Savepoints vs Checkpoints

| Aspect | Checkpoint | Savepoint |
|---|---|---|
| Purpose | Automatic recovery | Manual operations |
| Trigger | Automatic (interval) | Manual (user-initiated) |
| Lifecycle | Deleted on next checkpoint (unless retained) | Persisted until manually deleted |
| Use cases | Failure recovery | Version upgrade, parallelism change, rollback |
| Consistency | Exactly-once | Exactly-once |
| Format | Internal (optimized for speed) | Self-contained (optimized for portability) |

Savepoints are used for operational flexibility:
- Application version upgrade (stop with savepoint, upgrade, resume from savepoint)
- Parallelism change (stop with savepoint, change parallelism, resume from savepoint)
- Bug fix rollback (resume from savepoint taken before the bug was introduced)
- A/B testing (resume different versions from the same savepoint)

### 7.6 State Consistency Trade-offs

| Guarantee | Checkpointing | Output | Use Case |
|---|---|---|---|
| **At-most-once** | None | May lose events | Monitoring, alerting (tolerates loss) |
| **At-least-once** | Yes (state consistent) | May duplicate | Reporting (tolerates duplicates) |
| **Exactly-once** | Yes + idempotent/transactional sinks | Each event processed once | Payments, fraud, compliance |

**The cost of exactly-once:**
- Checkpoint overhead: periodic distributed snapshots consume CPU and network I/O
- Barrier alignment adds latency to processing (mitigated by unaligned checkpoints or incremental checkpointing)
- Transactional sinks add latency (two-phase commit)
- Storage cost for checkpoints (mitigated by incremental checkpointing)
- Configuration complexity — tuning checkpoint interval, timeout, parallelism

### 7.7 State Management at Scale

**State partitioning:** Keyed state is partitioned by key — each parallel task manages state for a subset of keys. This is the primary scaling mechanism. Adding parallelism redistributes keys across more tasks (rebalancing).

**RocksDB tuning for production:**
- **Memory budget:** Partition between block cache (read-heavy) and write buffer (write-heavy). Typical split: 70% block cache, 30% write buffer.
- **Bloom filters:** Enable on the last level for point lookups. Reduces read amplification at the cost of memory.
- **Compression:** LZ4 for speed (low CPU overhead), ZSTD for space (better compression, higher CPU). Typical: LZ4 for active write buffer, ZSTD for older levels.
- **Compaction style:** Level-based (default) for most workloads. Universal for write-heavy patterns.
- **Parallelism:** Configure max_background_jobs. Monitor compaction pressure to avoid stalls.
- **Monitoring:** Track disk usage, read amplification (bloom filter miss rate), write amplification (compaction bytes written / data ingested), compaction pending bytes.

**State TTL:** Configure time-to-live for stale state. Old keys are cleaned up automatically. Prevents unbounded state growth for keyed state where keys become inactive.

**State compaction:** In Kafka Streams, log compaction for changelog topics. In Flink, RocksDB compaction and state TTL.

**When local state doesn't suffice:**
- State exceeds available disk per node
- Cross-task state sharing is required
- State needs to survive application termination (beyond checkpoint retention)
- External state stores (Redis, Cassandra, S3-backed) scale independently
- Flink's remote state access is limited — consider external databases for very large shared state

---

## 8. ESP in the Event-Driven Architecture

### 8.1 Event-Driven Architecture (EDA)

Event-driven architecture is an architectural style where services communicate through events rather than direct calls (REST, gRPC). Events are published to an event bus (typically Kafka), and consumers react to events asynchronously.

**Request-driven vs. event-driven:**

| Aspect | Request-Driven | Event-Driven |
|---|---|---|
| Communication | Synchronous (caller waits) | Asynchronous (fire-and-forget) |
| Coupling | Tight (caller knows callee) | Loose (producer doesn't know consumers) |
| Scaling | Must scale together | Independent scaling |
| Resilience | Caller affected by callee failure | Event bus buffers; consumers tolerate failure |

**When to choose event-driven:** Multiple consumers need the same data, consumers have independent availability, data needs async processing, you need an audit trail, or you need replay capability.

**When to choose request-driven:** The caller needs an immediate response, the operation is transactional with rollback, the consumer is a single well-known service, or latency must be sub-millisecond.

### 8.2 Key EDA Patterns

**Event notification.** A service publishes an event to notify others that something happened. The event carries minimal data — consumers fetch additional data if needed. Simplest pattern, maximum decoupling.

**Event-carried state transfer.** Events carry the full state of the entity. Consumers build local state from the event stream without querying the producer. Higher throughput (bigger payloads), but consumers are self-sufficient.

**Event sourcing.** All state changes are captured as immutable events appended to a log. Current state is derived by replaying events. The event log is the source of truth — any other state is a derived materialized view. Enables complete audit trails, temporal queries, and CQRS integration. Trade-offs: unbounded event store growth (mitigated by snapshots), replay cost for long-lived entities, schema evolution complexity.

**CQRS.** Separate the write model (commands — validating business rules, producing events) from the read model (consuming events, building materialized views, serving queries). The two sides can use different data stores, schemas, and scale independently.

### 8.3 ESP Platforms as the EDA Backbone

In a mature event-driven architecture, ESP platforms play specific roles:

**Kafka as the event log (source of truth):** All events are durably stored in Kafka topics. Kafka provides partitioned ordering, replay capability, and configurable retention. It is the system of record for in-flight and recent event data.

**Stream processors (Flink, ksqlDB) as the transformation and enrichment layer:** Stream processors consume events from Kafka, apply business logic, and produce result events back to Kafka. They handle enrichment, aggregation, filtering, pattern detection, and stateful computation.

**Materialized views for the serving layer:** Results of stream processing are made queryable through materialized views. ksqlDB pull queries, materialized views in databases, or Kafka-backed caches serve low-latency queries on streaming results.

**Schema Registry for contracts:** The Schema Registry (Avro, Protobuf, JSON Schema) stores and manages event schemas. It enforces compatibility as schemas evolve, ensures producers and consumers agree on event formats, and provides a catalog of available event types.

### 8.4 Event Contracts and Schema Evolution

In an event-driven architecture, the schema of your events is your API contract. Schema evolution must be managed carefully:

**Schema compatibility rules:**
- **Backward compatible:** A new schema can read data written with the old schema. Required for consumers to upgrade before producers. Achieved by adding optional fields with defaults.
- **Forward compatible:** An old schema can read data written with the new schema. Required for producers to upgrade before consumers. Achieved by ignoring unknown fields.
- **Full compatible:** Both backward and forward compatible simultaneously. Required for rolling upgrades without coordination.
- **Transitive compatibility:** Compatibility is checked transitively across all schema versions, not just adjacent versions.

**Event versioning strategies:**
- **Schema Registry versioning:** Each event schema is versioned in the registry. Events in Kafka carry the schema ID. Consumers deserialize with the appropriate schema version.
- **Upcasting:** Old events are transformed (upcast) to the latest schema version on read. The event store retains only the latest schema version. Simplifies consumers (they only handle one version) but loses original event format.
- **Dual writing:** During migration, producers write both old and new format events to different topics or with different schema IDs. Consumers migrate incrementally. Common for breaking schema changes.

### 8.5 Streaming Data Pipelines for Data Mesh

In a data mesh architecture, domains own their data and publish it as event streams — data products. Stream processing enables cross-domain transformation and integration:

- **Domain data ownership:** Each domain publishes its data as well-defined event streams with schemas, SLAs, and documentation.
- **Stream processing for cross-domain transformations:** A shared stream processing layer transforms data from source domains into domain-specific formats for consumption. Example: the "customer risk" stream processor consumes events from the trading domain and the payments domain to produce an enriched customer risk stream.
- **Data catalog for stream discovery:** A data catalog (e.g., schema registry + data product documentation) enables domains to discover and subscribe to available event streams.
- **Serving layer:** Downstream consumers subscribe to the streams they need or query materialized views built from streams.

### 8.6 Enterprise Event Bus

The enterprise event bus pattern positions Kafka as the central nervous system of the organization, connecting transactional systems to analytical systems and enabling real-time data sharing across departments:

- **Transactional systems produce events:** Core banking systems, trading platforms, payment processors, and CRM systems publish business events to Kafka.
- **Analytical systems consume events:** Data warehouses, data lakes, dashboards, and ML platforms consume events for reporting, analytics, and model training.
- **Real-time data sharing:** Departments subscribe to events from other departments. Risk consumes trading events. Compliance consumes payment events. Finance consumes all events for P&L.
- **Event enrichment and transformation:** Stream processors enrich raw events with reference data, aggregate metrics, and transform formats as events move between domains.

---

## 9. ESP for Banking

Banking is one of the most demanding domains for event stream processing. The combination of high transaction volumes, sub-second latency requirements, regulatory scrutiny, and the need for exactly-once correctness makes banking an ideal use case for sophisticated ESP platforms.

### 9.1 Fraud Detection

**Real-time transaction scoring:** Every transaction (card payment, wire transfer, digital payment) must be scored for fraud risk in under 100 milliseconds. Flink processes transaction events as they arrive from the card network or payment gateway, computing features and scoring against ML models.

**Key ESP patterns in fraud detection:**
- **Windowed features (transaction count, amount, velocity):** Sliding windows compute transaction count, total amount, and velocity per account/card/IP address. Example: detect 10+ transactions in 5 minutes as potential velocity fraud.
- **Stream-table join with customer profile:** Transaction stream is joined with customer profile (risk tier, historical behavior, device fingerprints, geographic patterns). Feature store data arrives as a CDC stream or reference data broadcast.
- **Pattern detection for fraud sequences:** CEP patterns detect sequences indicative of fraud: "card not present test transaction followed by large withdrawal" or "account takeover attempt pattern" (multiple login failures, password reset, then large transaction).

### 9.2 Trade Surveillance

**Real-time market abuse detection:** Trading systems produce a continuous stream of orders, executions, cancellations, and modifications. Regulators (MAS, FCA, SEC) require real-time surveillance for market abuse.

**Key ESP patterns in trade surveillance:**
- **Pattern detection for spoofing / layering:** CEP patterns detect spoofing (placing orders with no intention to execute, to create false market depth) and layering (multiple orders at different price levels to create false depth). Pattern: "cancel large order after executing on the opposite side."
- **Pattern detection for insider trading:** Detect trading before material non-public information is released. Pattern: "trades in security X by insiders before X's price-sensitive announcement."
- **Order-to-trade ratio monitoring:** Sliding windows compute order-to-trade ratios. High ratios (many orders, few trades) indicate potential market manipulation.
- **Stream enrichment with market data:** Trade events are enriched with market data (bid/ask spread, market depth, news sentiment) for contextual analysis.

### 9.3 Payments Processing

**ISO 20022 payment messages:** Cross-border and domestic payments increasingly use ISO 20022 XML messages. Kafka ingests payment messages from SWIFT, SEPA, FAST (Singapore), and other payment systems.

**Key ESP patterns in payments:**
- **Real-time validation:** Validate payment messages against schema, business rules, and reference data before forwarding. Invalid messages are routed to a dead letter queue.
- **Enrichment and routing:** Enrich payment messages with exchange rates, correspondent bank details, and routing information.
- **Compliance checks (OFAC, AML):** Screen payment beneficiaries against sanctions lists (OFAC, UN), politically exposed persons (PEP) lists, and watchlists. Stream-table join with sanctions data stream (updated via CDC).
- **Exactly-once processing:** Payment processing requires end-to-end exactly-once guarantees. A payment must be processed exactly once — never lost (at-least-once is insufficient) and never duplicated (could result in double settlement).

### 9.4 Regulatory Reporting

**Real-time regulatory submission data:** Regulators increasingly require near-real-time transaction reporting. MAS (Monetary Authority of Singapore) requires daily or near-real-time transaction reporting. ESMA's MiFIR requires trade reporting within minutes.

**Key ESP patterns in regulatory reporting:**
- **Streaming ETL:** Transform transaction data from internal formats to regulatory formats (COREP, FINREP, MAS returns, MiFIR trade reports).
- **Data quality and validation:** Validate transaction data against regulatory rules in-stream. Flag and quarantine data quality issues immediately.
- **Event-time processing for accurate cut-off handling:** Reporting windows (T+1, T+cut-off) require event-time processing. A transaction at 23:59 belongs to today's reporting window even if it arrives at 00:01.
- **Late data handling:** Transactions arriving after the reporting cut-off must be handled according to regulatory rules (late submission flag, next reporting period).

### 9.5 Market Data Processing

**Market data ticker plants:** Market data (bid/ask prices, trade prices, indices, corporate actions) arrives as high-volume, high-velocity streams from exchanges (SGX, NYSE, LSE, CME).

**Key ESP patterns in market data:**
- **Normalization:** Normalize market data from different exchanges and channels to a unified schema. Map exchange-specific message formats to an internal canonical model.
- **Aggregation:** Compute VWAP (volume-weighted average price), TWAP (time-weighted average price), market depth, and implied volatility on streaming data.
- **Real-time VaR:** Compute value-at-risk on streaming positions using market data feeds. Windowed computation of P&L distribution.
- **Reference data distribution:** Distribute reference data (corporate actions, dividend dates, index composition changes) as event streams for downstream consumption.

### 9.6 Real-Time Risk

**Credit limit monitoring:** Track credit exposures in real-time as trades, payments, and collateral movements occur. Alert when limits are approached or breached.

**Market risk position tracking:** Maintain real-time positions from the trade stream. Compute mark-to-market on streaming security prices. Aggregate across desks, books, and legal entities.

**Liquidity monitoring:** Track cash flows from the payment stream. Monitor intraday liquidity positions against regulatory requirements (LCR, NSFR). Detect liquidity shortfalls in real-time.

**Counterparty credit exposure:** Compute current exposure, potential future exposure, and credit valuation adjustment (CVA) on streaming trades. Stream-table join with market data and credit spreads.

---

## 10. Pattern Detection and Complex Event Processing in Banking

### 10.1 Typical CEP Use Cases in Banking

**Fraud detection:** Real-time identification of fraudulent patterns on transaction streams:
- Rapid succession of transactions across multiple accounts (money mule detection)
- Test transactions followed by large withdrawals (card testing detection)
- Unusual geographic sequence (card used in Singapore, then UK, then Singapore within hours)
- Round-dollar amounts just below reporting threshold ($9,999) — structuring detection

**Trade surveillance:** Detection of market abuse patterns:
- Spoofing: orders placed and rapidly canceled to create false market depth
- Layering: multiple orders at different price levels to create false demand/supply
- Insider trading: unusual trading before price-sensitive announcements
- Wash trading: simultaneous buy and sell of same instrument to create fake volume

**AML (Anti-Money Laundering) monitoring:** Detection of money laundering patterns:
- Structuring: multiple transactions below reporting threshold that aggregate to significant amounts
- Rapid movement of funds through multiple accounts (layering stage)
- Transactions with high-risk jurisdictions
- Unusual transaction velocity for a dormant account

**Payment compliance:** Detection of compliance violations:
- Sanctions screening matches (OFAC, UN, EU sanctions lists)
- PEP (Politically Exposed Person) connections
- Unusual payment patterns consistent with trade-based money laundering

### 10.2 CEP Implementations

**Apache Flink CEP (NFA-based pattern matching):** Flink's CEP library implements a Non-deterministic Finite Automaton (NFA) for pattern matching. Patterns are defined using a fluent Java/Scala API and compiled into an NFA that processes events as they arrive.

```java
// Flink CEP pattern for AML structing detection
Pattern<Transaction, ?> structuringPattern = Pattern
    .<Transaction>begin("first")
        .where(t -> t.getAmount() < 10000)
    .next("second")
        .where(t -> t.getAmount() < 10000)
    .next("third")
        .where(t -> t.getAmount() < 10000)
    .within(Time.minutes(30));
```

**Custom state machines on Kafka Streams:** Kafka Streams can implement CEP using custom state machines with state stores. A state machine tracks the progress of pattern matching per key (e.g., per account). Each event advances or resets the state.

```java
// Simplified state machine in Kafka Streams
KStream<String, Transaction> stream = builder.stream("transactions");
stream.groupByKey().process(() -> new AMLStateMachine(), "aml-state-store");
```

**Drools Fusion:** Drools' CEP engine provides rule-based pattern matching for event streams. Rules are expressed in DRL (Drools Rule Language) with temporal operators. Suitable for rule-heavy CEP where patterns are expressed as declarative rules maintained by business analysts.

**Esper:** Esper is a dedicated CEP engine with its own Event Processing Language (EPL). EPL is SQL-like with temporal operators for pattern matching on streams. Esper is mature (first released in 2006) and used in several investment banks.

**Siddhi (WSO2):** Siddhi is an open-source CEP engine with a SQL-like query language. It supports event windows, sequences, patterns, and joins. Used in WSO2's middleware stack.

### 10.3 Typical CEP Patterns

**Sequence patterns:** A then B then C within time T. The classic temporal sequence pattern. Example: "Login → Trade → Logout within 10 minutes."

**Conjunction patterns:** A and B within time T. Both events must occur. Example: "Wire transfer initiated AND compliance approval received within 5 minutes."

**Negation patterns:** A without B within time T. A must occur without B. Example: "Trade executed without prior client authorization within 24 hours."

**Proximity patterns:** A and B within N events. Events are related by proximity rather than time. Example: "Two round-dollar transactions within 5 events."

**Conditional patterns:** Event property > threshold. Example: "Transaction amount > 3x historical average for this merchant."

### 10.4 CEP Implementation Considerations

**Pattern complexity:** CEP patterns range from simple (single-event condition) to extremely complex (multi-event, multi-branch, temporal, conditional). Complex patterns require more state and processing. The NFA state space grows with pattern complexity, increasing memory requirements.

**State management for patterns:** Each pattern instance maintains state — the current position in the pattern for each key. For high-cardinality keys (millions of accounts), the state can be large. State must be checkpointed for fault tolerance. State TTL should be configured to expire incomplete patterns.

**Handling out-of-order events:** CEP must handle events arriving out of event time order. A "B" event arriving before the corresponding "A" event for the same pattern must still be matched correctly. Watermarks and allowed lateness are essential for CEP accuracy.

**Pattern evaluation latency:** The time between receiving the last event in a pattern and emitting the match. For real-time use cases (fraud detection), pattern evaluation must complete within milliseconds. NFA processing is typically fast for simple patterns but can slow down for complex, deeply nested patterns.

**Scaling CEP across large event volumes:** CEP is keyed — each key's pattern matching is independent. Partitioning by key (account ID, trader ID, payment ID) enables horizontal scaling. Each parallel task handles a subset of keys. The bottleneck is typically state size per key and the number of concurrent partial matches.

### 10.5 CEP vs. Simple Stream Filtering

| Aspect | Simple Filtering | CEP |
|---|---|---|
| Logic | Single event condition | Multi-event sequence |
| State | Stateless | Stateful (partial matches) |
| Time | Current event only | Temporal (past events) |
| Complexity | O(1) per event | O(state size) per event |
| Example | `amount > $10K` | `deposit → 3 withdrawals < 5 min` |

CEP is necessary when the pattern involves multiple events, temporal relationships, and state. Simple filtering is sufficient when the decision is based on a single event.

### 10.6 CEP-based Detection vs. Rule Engines

**CEP vs. Drools Fusion.** Drools Fusion is a rule engine with CEP capabilities. It evaluates declarative rules against a working memory of events. CEP in Flink uses programmatic pattern definitions compiled into NFAs.

| Aspect | Flink CEP | Drools Fusion |
|---|---|---|
| API | Programmatic (Java/Scala DSL) | Declarative (DRL rules) |
| Execution model | NFA-based, streaming native | Rule engine with inference |
| State management | Flink state backend (RocksDB) | Working memory (in-memory) |
| Scalability | Partitioned, horizontal | Single-node, limited scale-out |
| Latency | Sub-millisecond | Milliseconds to sub-second |
| Community | Large (Apache Flink) | Moderate (Red Hat) |

**When to choose Flink CEP:** High-throughput, low-latency; stable patterns; integration with broader stream processing pipeline; horizontal scalability required.

**When to choose Drools Fusion:** Heavy business analyst involvement; complex decision trees; frequent rule changes; lower throughput (thousands, not millions of events/second); existing Drools investment.

### 10.7 CEP in ksqlDB (SQL Patterns)

For simpler CEP scenarios, ksqlDB provides `MATCH_RECOGNIZE` in SQL. This is suitable for moderate-complexity patterns on Kafka streams.

```sql
-- ksqlDB MATCH_RECOGNIZE for AML pattern
SELECT *
FROM transaction_stream
MATCH_RECOGNIZE (
    PARTITION BY account_id
    ORDER BY event_time
    MEASURES
        FIRST(step1.amount) AS first_amount,
        SUM(step2.amount) AS total_subsequent
    PATTERN (step1 step2+)
    DEFINE
        step1 AS amount < 10000,
        step2 AS amount < 10000 AND event_time < step1.event_time + INTERVAL '30' MINUTE
);
```

**Trade-off: CEP-based detection in Flink vs. SQL patterns in ksqlDB for simpler scenarios:**
- Flink CEP is more powerful (richer pattern semantics, arbitrary state, manual control, RocksDB for high-cardinality state)
- ksqlDB patterns are simpler to write and maintain, accessible to SQL-literate teams
- Choose Flink CEP for complex, high-volume, stateful patterns
- Choose ksqlDB for simpler patterns where SQL expressiveness suffices and operational simplicity matters

### 10.8 CEP Deployment Best Practices

**Flink CEP as a microservice per use case:** Each CEP use case (fraud, AML, trade surveillance) should be a separate Flink job deployed independently. This isolates failures, enables independent scaling, and separates operational concerns.

**State backend and checkpointing:** Use RocksDB as the state backend with incremental checkpointing to S3. Configure checkpoint intervals of 10-30 seconds. State TTL should match pattern windows — patterns older than the maximum window are cleaned.

**Monitoring:** Track active partial matches per pattern, pattern matching latency, timeout counts, state size per operator, and backpressure on CEP operators.

**Pattern wait time and timeout handling:** CEP patterns have a `.within(time)` clause — patterns that don't match within the window are automatically discarded. For long windows (days, weeks), accumulated partial match state can be large. Use appropriate timeouts, TTL for state cleanup, side outputs for timed-out partial matches, and alerting on excessive partial match accumulation.

---

## 11. Conclusion

Event Stream Processing has evolved from a niche technology for low-latency use cases to a fundamental architectural paradigm for modern data systems. The shift from batch to streaming — driven by the Lambda and Kappa architectures, enabled by Kafka's durable commit log, and powered by sophisticated stream processors like Flink — represents a fundamental change in how organizations think about data.

For banking, ESP is not optional. Fraud detection, trade surveillance, payments processing, regulatory reporting, and real-time risk all require continuous processing of unbounded event streams with sub-second latency and exactly-once correctness. The technologies and patterns described in this guide — Flink for complex stateful processing, Kafka for the event backbone, ksqlDB for SQL-based stream processing, and CEP for pattern detection — form the foundation of modern banking infrastructure.

As data volumes grow, latency requirements shrink, and regulatory scrutiny intensifies, the organizations that invest in robust event stream processing architectures will be the ones that can detect fraud faster, comply with regulations more accurately, and serve customers with lower friction. Streaming is not the future — it is the present, and the organizations that treat data as a continuous flow rather than discrete batches will have a decisive competitive advantage.

### Key Takeaways

1. **Streaming is not batch.** Event stream processing is fundamentally different from batch processing — it operates on unbounded data in real-time, with state, time semantics, and fault tolerance.

2. **Time is the hardest problem.** Event time, processing time, and watermarks are the most subtle and important concepts in stream processing. Get time semantics right, or your results will be wrong.

3. **State is what makes stream processing powerful.** Stateful stream processing — maintaining counters, aggregations, patterns, and ML features across events — is where ESP creates value.

4. **Choose the right technology.** Kafka for the event backbone, Flink for complex stateful processing, ksqlDB for SQL-based stream processing, Spark Streaming for Spark-native teams. There is no one-size-fits-all.

5. **Exactly-once semantics are essential for banking.** Payments, fraud, and compliance require end-to-end exactly-once processing. The cost of exactly-once (checkpointing, transactional sinks, complexity) is justified by the correctness guarantees.

6. **CEP is a core banking capability.** Pattern detection for fraud, AML, trade surveillance, and compliance requires complex event processing. Flink CEP provides the most sophisticated open-source CEP implementation.

7. **Event-driven architecture is the architectural pattern for ESP.** Kafka as the central event bus, stream processors for transformation, materialized views for serving — this is the proven architectural pattern for real-time data at scale.

8. **Start streaming where it hurts most.** The most impactful banking use cases — fraud, payments, trade surveillance, risk — are also the most demanding. Begin with a high-value, high-correctness use case, prove the architecture, then expand.

---

> *This guide is maintained by Jack Liu Shurui. For questions, corrections, or additions, please open an issue or PR on the [research repository](https://github.com/jackliusr/research).*
