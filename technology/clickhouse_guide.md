# ClickHouse — The Merge Tree

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Technology Architecture — real-time analytical databases, columnar storage, regulated-enterprise operations
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Facts checked against primary sources:** 15 September 2026 (UTC). Every version, date, licence and funding claim is dated and sourced; the ones I could not verify are listed in §13.
> **Scope:** ClickHouse the product and the system: its storage and query architecture, its distributed and reliability machinery, its ingest and lifecycle reality, its SQL and ecosystem surface, its shipped AI/vector capability, its known failure modes, and the position of ClickHouse Inc. as the commercial vendor behind it. This is the engine guide, not the landscape guide.
> **Out of scope:** the peer-by-peer analytical-database comparison (that belongs to `technology/data/mpp_databases_guide.md` §4/§5), latency-engineering patterns for generative AI (`technology/low_latency_genai_patterns_guide.md`), vector-database comparison and RAG patterns (`technology/ai_llm/rag/vector_databases_guide.md` and the rest of the `ai_llm/rag/` cluster), and data-platform/lakehouse architecture (`technology/data/enterprise_data_platforms_guide.md`, `technology/advanced_analytics_solutions_guide.md`).

**Related guides in this series:** [MPP Databases — The Landscape](data/mpp_databases_guide.md) owns the six-peer comparison and the ClickHouse profile inside it; [Low-Latency GenAI Patterns](low_latency_genai_patterns_guide.md) owns TTFT/TPOT and serving latency; [Vector Databases](ai_llm/rag/vector_databases_guide.md) and [RAG Patterns](ai_llm/rag/rag_patterns_guide.md) own retrieval-store comparison; [Enterprise Data Platforms](data/enterprise_data_platforms_guide.md) and [Advanced Analytics Solutions](advanced_analytics_solutions_guide.md) own platform and lakehouse architecture; [Apache Flink — The Stream's State](apache_flink_guide.md) owns stream-processing state and checkpointing; [Operational Resilience Framework](../banking/operational_resilience_framework_guide.md) owns RTO/RPO and disaster-recovery doctrine; [Kafka](../banking/kafka_guide.md) owns the event backbone that usually feeds a ClickHouse cluster; [Cloud Object Storage and the Lakehouse](cloud_object_storage_lakehouse_guide.md) owns object-storage table formats.

---

## Table of Contents

1. [The Overview and the Identity](#1-the-overview-and-the-identity)
   - 1.1 [What ClickHouse Is](#11-what-clickhouse-is)
   - 1.2 [What ClickHouse Is Not](#12-what-clickhouse-is-not)
   - 1.3 [The Origin Story, Dated](#13-the-origin-story-dated)
   - 1.4 [The Company Position](#14-the-company-position)
   - 1.5 [The One-Page Orientation](#15-the-one-page-orientation)
2. [The Storage Architecture](#2-the-storage-architecture)
   - 2.1 [Parts, Granules and the Sparse Primary Index](#21-parts-granules-and-the-sparse-primary-index)
   - 2.2 [The MergeTree Family](#22-the-mergetree-family)
   - 2.3 [Compression and Codecs](#23-compression-and-codecs)
   - 2.4 [Partitioning, Ordering and Skipping Indices](#24-partitioning-ordering-and-skipping-indices)
   - 2.5 [Projections, Materialised Views and Statistics](#25-projections-materialised-views-and-statistics)
3. [The Query Engine](#3-the-query-engine)
   - 3.1 [Vectorised Execution and the Pipeline](#31-vectorised-execution-and-the-pipeline)
   - 3.2 [The SQL Extensions That Matter](#32-the-sql-extensions-that-matter)
   - 3.3 [Joins and the Optimiser](#33-joins-and-the-optimiser)
   - 3.4 [Profiling and EXPLAIN](#34-profiling-and-explain)
4. [The Distributed and Reliability Machinery](#4-the-distributed-and-reliability-machinery)
   - 4.1 [Replication and ClickHouse Keeper](#41-replication-and-clickhouse-keeper)
   - 4.2 [Sharding and the Distributed Engine](#42-sharding-and-the-distributed-engine)
   - 4.3 [Consistency: What Is and Is Not Guaranteed](#43-consistency-what-is-and-is-not-guaranteed)
   - 4.4 [Backup, Restore and What One Node Cannot Survive](#44-backup-restore-and-what-one-node-cannot-survive)
5. [Ingestion and the Data Lifecycle](#5-ingestion-and-the-data-lifecycle)
   - 5.1 [The Insert Model and Why Batching Is Everything](#51-the-insert-model-and-why-batching-is-everything)
   - 5.2 [Asynchronous Inserts, Buffers and Streams](#52-asynchronous-inserts-buffers-and-streams)
   - 5.3 [Deduplication and the Exactly-Once Question](#53-deduplication-and-the-exactly-once-question)
   - 5.4 [TTL, Tiering and Mutations](#54-ttl-tiering-and-mutations)
6. [The SQL Surface, the Tooling and the Ecosystem](#6-the-sql-surface-the-tooling-and-the-ecosystem)
   - 6.1 [Interfaces and Drivers](#61-interfaces-and-drivers)
   - 6.2 [The Observability Surface](#62-the-observability-surface)
   - 6.3 [Integrations and the Ecosystem](#63-integrations-and-the-ecosystem)
7. [The AI and Vector Capability](#7-the-ai-and-vector-capability)
   - 7.1 [Vector Search as Shipped](#71-vector-search-as-shipped)
   - 7.2 [Hybrid Search and Full-Text](#72-hybrid-search-and-full-text)
   - 7.3 [LLM-over-ClickHouse as Shipped](#73-llm-over-clickhouse-as-shipped)
   - 7.4 [The Practical Trade-Offs](#74-the-practical-trade-offs)
8. [The Operational Reality and the Known Hard Parts](#8-the-operational-reality-and-the-known-hard-parts)
   - 8.1 [Small Parts and Merge Pressure](#81-small-parts-and-merge-pressure)
   - 8.2 [Memory and the OOM Class](#82-memory-and-the-oom-class)
   - 8.3 [Joins, Lookups, Upgrades and Keeper](#83-joins-lookups-upgrades-and-keeper)
   - 8.4 [Skills and Cost](#84-skills-and-cost)
9. [The Comparison — A Positioning Table Only](#9-the-comparison--a-positioning-table-only)
10. [The Banking and Regulated-Estate Angle](#10-the-banking-and-regulated-estate-angle)
11. [The Cymbal Bank Worked Example](#11-the-cymbal-bank-worked-example)
   - 11.1 [The Workload](#111-the-workload)
   - 11.2 [Schema and Keys](#112-schema-and-keys)
   - 11.3 [Topology, Retention and Failure Handling](#113-topology-retention-and-failure-handling)
   - 11.4 [Illustrative Cost Comparison](#114-illustrative-cost-comparison)
   - 11.5 [What This Design Cannot Guarantee](#115-what-this-design-cannot-guarantee)
12. [The Claims Audit](#12-the-claims-audit)
13. [What Could Not Be Verified](#13-what-could-not-be-verified)
14. [Glossary](#14-glossary)
15. [Cross-References and Further Reading](#15-cross-references-and-further-reading)
16. [Closing Summary](#16-closing-summary)

---

## Scope: What This Guide Owns, and What Its Siblings Own

This guide owns ClickHouse as a system. It explains how a MergeTree table is physically laid out on disk, what the merge scheduler is actually doing, why the primary index is not a B-tree, where the query engine spends its time, what the coordination layer is for, what happens to a cluster when an insert pattern is wrong, and what the vendor sells on top of an Apache-2.0 core. Where a topic is already owned elsewhere in this repository, this guide names the sibling and moves on rather than restating it.

The boundary is deliberate. `technology/data/mpp_databases_guide.md` (1,871 lines) carries the six-peer landscape and its §4.5 ClickHouse profile plus the §5 comparison table; **the peer-by-peer analysis lives there and is not repeated here**. §9 below is a positioning table only. `technology/low_latency_genai_patterns_guide.md` owns TTFT/TPOT and serving-layer latency; `technology/ai_llm/rag/vector_databases_guide.md` owns the vector-database comparison matrix; §7 here covers only what ClickHouse itself ships. `../banking/operational_resilience_framework_guide.md` owns RTO/RPO doctrine; §10 cross-references it in condensed form.

One consequence of that boundary is worth stating up front. The difference between this guide and a vendor landing page is not tone, it is what gets included. ClickHouse's storage engine is genuinely excellent at one narrow thing — scanning and aggregating very large append-only datasets fast, for many concurrent readers. It is genuinely weaker at joins, at row-level mutation, at point lookups, and at anything that needs a transaction. Those weaknesses are documented by the project itself, and §1.2, §3.3 and §8 quote the documentation rather than paraphrasing it.

---

## 1. The Overview and the Identity

### 1.1 What ClickHouse Is

ClickHouse is an open-source, column-oriented SQL database management system built for online analytical processing. The project's own description is narrower than the marketing: it is a system "designed for high data ingest rates and huge data volumes", where "insert operations create table parts which are merged by a background process with other table parts" (clickhouse.com/docs, MergeTree engine reference, last modified 11 September 2026, checked 15 September 2026). Everything else about the product follows from that sentence.

The problem class it is built for has four properties held simultaneously, and it is the simultaneity that makes the problem hard:

| Property | What it means operationally | Where ClickHouse sits |
|---|---|---|
| **High ingest rate** | Millions of rows per second, continuously, without a batch window | Append-only insert path; batches converted to parts; background merges |
| **Large data volume** | Tens to hundreds of terabytes per node, petabytes per cluster | Columnar files, codec-compressed, sparse primary index that stays resident |
| **High query concurrency** | Hundreds of interactive analytical queries at once, not one analyst at a time | Per-query threading, vectorised operators, no shared write lock on reads |
| **Freshness** | Seconds, not hours — data must be queryable shortly after it arrives | Inserts are visible on commit; async inserts trade a flush delay for batching |

Most systems optimise for two of those four. A row-store optimises freshness and concurrency but collapses on volume and scan throughput. A batch warehouse optimises volume and scan throughput but gives up freshness and often concurrency. A search engine optimises latency and freshness but pays ten times the storage and compute for structured aggregation, as ClickHouse's own Series C announcement argued (clickhouse.com/blog, 29 May 2025). ClickHouse's claim is that it holds all four, and the architecture in §2 and §3 is the mechanism by which it tries.

### 1.2 What ClickHouse Is Not

Being precise here is more useful than being enthusiastic, because the failure mode is a team adopting ClickHouse for a workload it was never built for and then discovering the gap eighteen months later, after the migration cost is sunk.

**It is not a transactional database.** The documentation states the position without hedging: ClickHouse "is optimized for append-only operations and offers only eventual consistency guarantees", in explicit contrast to OLTP databases such as PostgreSQL which are "specifically optimized for transactional inserts with full ACID compliance" (clickhouse.com/docs/guides/inserting-data, last modified 28 August 2026). There is no multi-statement transaction, no rollback, and no isolation level to choose, and `KILL MUTATION` cancels a running mutation but explicitly does not roll it back (clickhouse.com/blog, 20 February 2026). **It is not a point-lookup store either.** A query for one row by a unique identifier is served by the sparse primary index, which points at granules of 8,192 rows, not at rows. Serving a million single-row lookups per second is a row-store workload, and ClickHouse will do it far more expensively than a key-value store will.

**It is not a general warehouse replacement in every respect.** Three specific gaps are documented rather than inferred. First, joins: the engine has no traditional cost-based optimiser as its baseline behaviour, join ordering is limited, and large joins are memory-bound (see §3.3); the new cost-based optimiser is documented as shipping inside a private-preview feature, not as the default planner (see §1.4). Second, updates and deletes are expensive by construction: classic `ALTER TABLE ... UPDATE` "rewrite[s] entire data parts" and is described by the project as suited "to infrequent, bulk changes"; the newer lightweight patch-part update path is the mitigation. Third, mutation pressure shares the merge thread pool, so a careless update pattern causes the "too many parts" failure described in §5.1 and §8.1.

**It is not always the cheaper option.** ClickHouse's cost advantage comes from scan efficiency and compression, and it is real for large append-only datasets. It is not real for small datasets, for workloads dominated by single-row reads, or for teams that cannot staff the operating discipline that parts, merges and Keeper demand.

**It is not merely a Snowflake or BigQuery competitor.** Much of ClickHouse's commercial traction is in observability and log analytics, where the competitor set is Elasticsearch and a metrics stack, not a warehouse. The 2026 product releases in §1.4 make that direction explicit.

### 1.3 The Origin Story, Dated

The timeline matters because two of ClickHouse's distinguishing properties — the sparse index and the merge model — were designed in 2009 to answer one specific question, and they were never retrofitted.

| Milestone | Date | Source |
|---|---|---|
| Experimental project begins at Yandex; hypothesis: real-time analytics over non-aggregated, continuously arriving data | 2009 | Wikipedia (fetched 15 September 2026), attributing the project to Alexey Milovidov and developers |
| First production deployment, powering Yandex.Metrica web analytics | 2012 | Wikipedia (fetched 15 September 2026); internally-reported, marked as needing citation on the page |
| **First open-source release, under Apache-2.0** | **15 June 2016** | Wikipedia infobox (fetched 15 September 2026) |
| **ClickHouse, Inc. incorporated in San Francisco**; initial US$50M from Index Ventures and Benchmark, with participation from Yandex N.V. | **September 2021** | Wikipedia (fetched 15 September 2026) |
| Series B — US$250M at a US$2B valuation, led by Coatue Management and Altimeter Capital | 28 October 2021 | Wikipedia (fetched 15 September 2026) |
| Series C — US$350M led by Khosla Ventures; total funding past US$650M | 29 May 2025 | clickhouse.com/blog/clickhouse-raises-350-million-series-c-to-power-analytics-for-ai-era, dated 29 May 2025, checked 15 September 2026 |
| Series D — US$400M led by Dragoneer Investment Group; the company acquires Langfuse (LLM observability) and introduces a native Postgres service | 16 January 2026 | clickhouse.com/blog/clickhouse-raises-400-million-series-d-acquires-langfuse-launches-postgres, dated 16 January 2026, checked 15 September 2026 |
| Current LTS release **v26.8.5.13-lts**, marked Latest | **15 September 2026** | github.com/ClickHouse/ClickHouse/releases, checked 15 September 2026 |

The three-stage shape — internal tool, then open source, then company — is the same shape as several other infrastructure projects, but the seven-year gap between production deployment (2012) and open sourcing (2016) is unusually long, and it is why the codebase carries the assumptions of its environment. Yandex.Metrica's workload was: enormous append-only event volume, one sort order that answered the majority of questions, almost no updates, and a hard latency requirement for interactive dashboard queries. ClickHouse was built to that shape and the shape is still visible in the `ORDER BY`-centric design, the merge model, and the deliberate refusal to implement row-level transactional semantics. The legal mechanism of the spin-out — carve-out, trademark transfer or licence — is not documented in a single primary source and is flagged in §13.3.

### 1.4 The Company Position

The commercial entity sells a managed service and assurance; the open-source project gives away the engine. The split is unusually clean and unusually generous by 2020s infrastructure standards.

**What is Apache-2.0 and free.** The entire engine: `github.com/ClickHouse/ClickHouse/LICENSE` reads, verbatim, "Copyright 2016-2026 ClickHouse, Inc." followed by the unmodified Apache License Version 2.0 text (fetched 15 September 2026). There is no Business Source License, no "ClickHouse Public License", no Commons Clause, no source-available delay, and no field-of-use restriction. There is no separate enterprise edition with feature-gated engine capabilities. This is **(verified, raw LICENSE file at github.com/ClickHouse/ClickHouse, 15 September 2026)** and it is the single most commercially important fact about the project, because it means the exit cost of leaving the vendor is low.

**What the company sells.** The commercial surface as of September 2026:

| Offering | Status as of 15 Sep 2026 | Source |
|---|---|---|
| ClickHouse Cloud (managed service); Enterprise support and tier features | GA — the company's core revenue product | clickhouse.com; Series C announcement, 29 May 2025 |
| Replica-aware routing | **Public Beta, Enterprise tier only** | clickhouse.com/blog/replica-aware-routing-public-beta, 15 September 2026 |
| On-Demand Compute (shared worker pool) | **Private preview** | clickhouse.com/blog/on-demand-compute, 10 September 2026 |
| ClickHouse Managed Postgres on Google Cloud | **Private preview** | clickhouse.com/blog/postgres-managed-by-clickhouse-gcp-private-preview, 9 September 2026 |
| TimeSeries table engine and PromQL in ClickHouse Cloud | **Private preview** | clickhouse.com/blog/introducing-promql, 15 September 2026 |
| AI Functions | **Beta**; **private preview** in ClickHouse Cloud | clickhouse.com/blog/ai-functions-in-clickhouse, 11 September 2026 |
| WalShadow (Postgres physical-WAL replication) and the chdb Postgres extension | Announced 10 and 8 September 2026 | clickhouse.com/blog/introducing-walshadow; clickhouse.com/blog/introducing-chdb-postgres |
| Data agent launch partnership in ChatGPT Work, and the ClickHouse MCP server | Announced 10 September 2026; MCP server shipped | clickhouse.com/blog/chatgpt-data-plugin; clickhouse.com/docs (MCP setup action) |
| Langfuse (LLM observability) — acquired and run alongside Cloud | **Acquired 16 January 2026**; integration status not verified here | clickhouse.com/blog/clickhouse-raises-400-million-series-d-acquires-langfuse-launches-postgres, 16 January 2026 |

The pattern in that table is the honest finding. Between 7 and 15 September 2026 the company announced eleven separate things, and **only one of them — replica-aware routing — is even a public beta, and it is gated to Enterprise customers**. On-Demand Compute, Managed Postgres, the TimeSeries/PromQL engine and AI Functions are all private preview or beta. Reading a launch blog as a shipped capability is the most common error made about this vendor's 2026 releases, and §12 records it as such.

**Funding.** Series C (29 May 2025) raised US$350M led by Khosla Ventures, reported at approximately US$6.35 billion and bringing total funding past US$650 million, plus a US$100 million credit facility led by Stifel and Goldman Sachs, with over 2,000 customers and named adopters including Anthropic, Tesla, Mercado Libre, Sony, Meta, Lyft, Instacart, Memorial Sloan Kettering and Sierra (clickhouse.com/blog, 29 May 2025). **Series D closed on 16 January 2026: US$400 million led by Dragoneer Investment Group**, with Bessemer Venture Partners, GIC, Index Ventures, Khosla Ventures, Lightspeed Venture Partners, accounts advised by T. Rowe Price, and WCM Investment Management participating. The same announcement reported **more than 3,000 customers on ClickHouse Cloud and ARR growing more than 250% year over year**, and disclosed two strategic moves — the **acquisition of Langfuse**, the open-source LLM-observability platform, and a **native Postgres service** to sit beside the analytical engine **(verified, clickhouse.com/blog/clickhouse-raises-400-million-series-d-acquires-langfuse-launches-postgres, 16 January 2026)**.

**Flagged:** the **US$15 billion valuation** attached to Series D in press coverage comes from secondary outlets (Tech Company News and private-market aggregators) and was **not stated in the vendor's own announcement** I fetched — the round and the amount are verified, the valuation is secondary. The September 2021 incorporation, the US$50M seed and Series B (US$250M at a US$2B valuation, 28 October 2021) rest on Wikipedia alone and are marked verified-secondary.

**Cadence.** The release train is monthly and heavily patched. On a single day — 15 September 2026 — the releases page showed `v26.8.5.13-lts` (marked Latest), `v26.7.9.12-stable` and `v26.6.7.18-stable`, meaning three different monthly branches all received patches that day, while `v26.3.33.24-lts` (9 September 2026) shows the previous LTS still receiving them on a slower track. The repository shows 86 pages of releases and 49.9k stars. **(verified, github.com/ClickHouse/ClickHouse/releases, 15 September 2026)**

**Flagged, and it matters:** the exact version number above will be stale within weeks. Any institution pinning a version must read the releases page itself on the day it decides, and must decide between the `-stable` monthly train and the `-lts` slower track deliberately. The LTS in this snapshot is 26.8; the previous LTS was 26.3.

### 1.5 The One-Page Orientation

```
        ClickHouse: one node, end to end

  clients ─► [ native | HTTP | MySQL/Postgres wire ]
                       │
                       ▼
             ┌───────────────────────┐
             │ Query pipeline         │ vectorised, per-block,
             │ (pipeline executor)    │ multi-core
             └──────────┬────────────┘
                        │ reads
      ┌─────────────────┴──────────────────┐
      ▼                                    ▼
  ┌────────────┐  background  ┌────────────────────┐
  │  PARTS     │◄── merges ───│  MERGE SCHEDULER    │
  │  immutable │              │  thread pool shared │
  │  ORDER BY  │              │  with mutations     │
  └──────┬─────┘              └────────────────────┘
         │ per column: data + marks + skip indexes; sparse index
         ▼
   [ local disk | tiered volumes (TTL) | object storage ]
         │
  replicated? ─► Keeper (RAFT): replication metadata, DDL queue, dedup log
  sharded?    ─► Distributed engine fans out, merges partial results
```

Read that diagram once and the rest of this guide is elaboration: data is immutable once written, sorted by a declared key, held in separate files per column, indexed sparsely at granule granularity and merged in the background forever, while queries scan columns rather than rows. Everything ClickHouse is good at comes from that layout; everything it is bad at comes from the absence of what a row-store would put there instead.

---

## 2. The Storage Architecture

### 2.1 Parts, Granules and the Sparse Primary Index

The unit of storage is the **part**. Every insert produces one or more parts; a part is immutable, and within a part the rows are stored in lexicographic order of the primary key. That ordering is the whole trick. Because the data is sorted on write, the index does not need to reference rows — the project's documentation states it plainly: the primary key "does not reference individual rows but blocks of 8192 rows called granules. This makes primary keys of huge data sets small enough to remain loaded in main memory, while still providing fast access to on-disk data" (clickhouse.com/docs, MergeTree engine reference, last modified 11 September 2026).

The operational consequence is the important part. A thousand-column table with a billion rows has a primary index measured in single-digit megabytes, and that index is resident in memory on every node, always. A B-tree index on the same data, with a pointer per row, would not fit in memory on any machine, and its maintenance cost would be paid on every insert. ClickHouse made that trade explicitly: fewer index entries, coarser navigation, and a mandatory full scan of the matching granule.

The trade bites in three places. Query performance is granularity-bound — a query that matches one row still reads 8,192 rows of columns to find it. `index_granularity` is configurable per table, and lowering it improves point-query precision at the cost of a larger index and more index entries to merge. And because the index is only *navigable* on the declared sort prefix, a filter on a non-prefix column gets no help from it at all; that is what skipping indices in §2.4 are for.

The other consequence of immutability is that **deleting and updating are not storage operations, they are rewrites**. A part cannot be edited. A delete is either a partition drop (cheap, and the reason partitioning exists) or a mutation that rewrites parts (expensive, and the reason §5.4 exists).

| Concept | Default / value | Operational consequence |
|---|---|---|
| Part | One per insert (or per async-insert flush) | Too many parts = slow queries, slow startup, merge pressure |
| Granule | 8,192 rows (`index_granularity`) | Minimum read unit; point lookups read a granule's columns |
| Primary index | Sparse, one entry per granule, in memory | Tiny and always resident; useless off the sort prefix |
| Part mergeability / mutability | Merges only within the same partition; parts are immutable | A high-cardinality partition key freezes merging; updates and deletes are rewrites (§5.4) |

### 2.2 The MergeTree Family

`MergeTree` and its siblings are the engines that matter; the other engine families exist for special cases. The family is a set of behaviours layered on the same part-and-merge storage, and the choice of engine is a choice about what happens when two rows with the same sort key collide during a merge.

| Engine | Merge-time behaviour | Use it when |
|---|---|---|
| `MergeTree` | Keeps all rows | Default; pure append analytics |
| `ReplacingMergeTree` | Collapses rows with identical sort key, keeping the last by version | Latest-state tables where eventual deduplication is acceptable |
| `SummingMergeTree` / `AggregatingMergeTree` | Sums numeric columns, or merges aggregate-function states | Pre-aggregated counters; materialised-view targets holding `...State` columns |
| `CollapsingMergeTree` / `VersionedCollapsingMergeTree` | Cancels sign-paired rows | CDC-style change streams |
| `Replicated*` variants / `SharedMergeTree` | Replicate parts through Keeper; or compute/storage separation on object storage | Multi-node deployments; ClickHouse Cloud |

The operational reality of the family is the sentence practitioners repeat until it becomes instinct: **`ReplacingMergeTree` does not guarantee uniqueness at query time**. Deduplication happens during merges, on the merge schedule, and a query issued before the relevant parts have merged will see duplicates unless it uses `FINAL` or an `argMax`-style pattern. `FINAL` forces deduplication at read time and is correspondingly more expensive. A designer who assumes `ReplacingMergeTree` gives them a primary-key constraint will be wrong in production, and not subtly.

The second reality is that merges are forever. A cluster that stops merging does not become a read-only database, it becomes a slow one — query latency degrades as the part count grows, and one day inserts start failing outright. That failure mode is treated in full in §5.1 and §8.1 because it is the single most common way a ClickHouse deployment goes wrong.

### 2.3 Compression and Codecs

Column-oriented storage compresses well because a column holds values of one type with local similarity, and ClickHouse attaches a codec per column rather than a global compression setting. The supported codecs and what each is for:

| Codec | Mechanism | Use it on |
|---|---|---|
| `LZ4` | Fast general-purpose block compression | Default when nothing else fits; speed over ratio |
| `ZSTD(level)` | Slower, higher-ratio general compression | Cold or archival columns |
| `Delta` / `DoubleDelta` | Differences, then differences of differences | Monotonic IDs; timestamps at regular intervals (very high ratio) |
| `Gorilla` / `FPC` | XOR of float bit patterns; predicted float deltas | Gauges, metrics and scientific float series |
| `T64` | Transposes and trims 64-bit integer ranges | Integer columns with bounded ranges |

The operational consequence is that codec choice is a real engineering decision with a measurable return, and it is also a decision that must be made at `CREATE TABLE` time and can only be changed afterwards with an `ALTER` that rewrites data. The documentation supports column-level `max_compress_block_size` and `min_compress_block_size` settings, mutable via `ALTER TABLE ... MODIFY COLUMN ... MODIFY SETTING` (clickhouse.com/docs, MergeTree engine reference, 11 September 2026). A timestamp column left on `LZ4` is a permanently worse table than the same column on `DoubleDelta`, and the difference compounds at petabyte scale.

### 2.4 Partitioning, Ordering and Skipping Indices

Three separate mechanisms control which data a query touches, and conflating them is a common design error.

**`ORDER BY` is the dominant one.** It defines the sort order inside every part and, unless `PRIMARY KEY` is set separately, it *is* the primary key. It determines which granule range a filter can skip. Choosing it is the single highest-leverage schema decision in a ClickHouse table.

**`PARTITION BY` is a data-management tool, not a query-acceleration tool.** The documentation is unusually blunt about this: "Partitioning does not speed up queries (in contrast to the ORDER BY expression). You should never use too granular partitioning" (clickhouse.com/docs, MergeTree engine reference, 11 September 2026). The reason it does not accelerate queries is that skipping indices and the primary index already handle that; the reason not to over-partition is that **parts in different partitions are never merged**, so a high-cardinality partition key permanently freezes the merge process and produces the "too many parts" error. The project's guidance is to keep partition-key cardinality below roughly 1,000 and to use month-level granularity via `toYYYYMM(date)` (clickhouse.com/blog, "13 mistakes", 20 February 2026).

**Data-skipping indices** are the mechanism for accelerating filters on columns that are not in the sort prefix. They are per-granule summaries stored alongside the column, and they let the engine prove that a granule cannot contain a match and skip reading it.

| Skip index type | What it stores per granule | Filters it helps |
|---|---|---|
| `minmax` | Min and max value | Range predicates |
| `set(N)` | Up to N distinct values | Equality on low-cardinality columns |
| `bloom_filter` | Probabilistic membership | Equality on high-cardinality columns |
| `ngrambf_v1` / `tokenbf_v1` | Bloom over character n-grams; bloom over whitespace-tokenised tokens | `LIKE '%substring%'`; full-text-ish token matching |
| `vector_similarity` | HNSW proximity graph | Vector search (§7.1) |

The honest caveat is that skip indices are consulted after the primary index and before the read, they cost write time and disk, and they help only when the predicate is selective enough that whole granules can be excluded. Adding a skip index to a column that matches most granules produces no speedup and a slower insert path.

### 2.5 Projections, Materialised Views and Statistics

Three pre-aggregation and optimisation surfaces exist, and they differ in who is responsible for keeping them correct.

**Projections** are alternative physical orderings stored inside the same table, declared in `CREATE TABLE` and selected automatically by the query planner when the projection's sort order matches a query's needs. They are the cleanest way to serve a second access pattern without a second table, at the cost of duplicated storage and more merge work.

**Materialised views** in ClickHouse are not the materialised views of other databases. They are **insert triggers**: the view's `SELECT` runs on each inserted block, and the result is written to a target table. Combined with `AggregatingMergeTree` and `...State` columns, this is the standard way to maintain rolling aggregates at write time. The cost is honest and documented: an excessive number of materialised views is itself listed as a cause of the "too many parts" failure, because each view produces its own parts in its target table (clickhouse.com/blog, 20 February 2026).

**Column statistics** are a comparatively recent addition and they show up in the current documentation as `STATISTICS(...)` column clauses with types `basic`, `countmin`, `minmax`, `tdigest`, `uniq` and `uniq_v2`, used by the optimiser for cardinality estimation and part pruning (clickhouse.com/docs, MergeTree engine reference, last modified 11 September 2026). Notably, `basic` answers an equality filter exactly only when the compared value matches the column's storage default, and its range support is limited to numeric columns. This is the storage-side plumbing that the new cost-based optimiser described in §3.3 and §1.4 depends on, and its presence in the docs is evidence that the optimiser work is real rather than marketing.

---
## 3. The Query Engine

### 3.1 Vectorised Execution and the Pipeline

ClickHouse does not interpret a query row by row. It compiles a pipeline of operators connected by queues, and each operator processes a whole **block** of columnar values at a time. A filter applied to a block is a tight loop over contiguous memory, and because the values are already in the correct column order, that loop is auto-vectorisable into SIMD instructions on modern x86 and ARM. The project's own framing of the consequence is that ClickHouse "was designed from the ground up to utilize the full resources of a machine", with one successful deployment pattern described as servers holding "hundreds of cores, terabytes of RAM, and petabytes of disk space" (clickhouse.com/blog, "13 mistakes", 20 February 2026).

The parallelism model is one query, many threads, with `max_threads` defaulting to the core count. Most analytical queries have a filter, a sort and an aggregation stage, and each of those stages can be parallelised independently, so a single query on a 128-core machine will use 128 cores. This is why the project's own guidance is emphatic that horizontal scaling should come second: "go vertical before going horizontal" (clickhouse.com/blog, 20 February 2026). Two nodes with maximum-sized machines will beat twenty small ones for most analytical workloads, because the twenty-node version moves data across the network for every join and aggregation.

| Design choice | What it buys | What it costs |
|---|---|---|
| Block-at-a-time vectorised operators | SIMD throughput; no per-row interpreter overhead | Accurate row-at-a-time error context is harder |
| One query spans all cores; pipeline of queues between operators | Vertical scaling is genuinely effective; back-pressure is natural and stages overlap | A single heavy query can monopolise the machine; profiling must follow the pipeline, not a call stack (§3.4) |
| Separate query and merge thread pools | Reads are not blocked by merges | Reads and merges compete for memory and I/O |

The flip side of the design is fairness. Because a query grabs as many threads as it can, concurrency and per-query latency are in direct tension, and the control mechanism is `max_threads` (and, in a cluster, admission control at the proxy layer). A deployment serving 200 concurrent dashboard queries must lower `max_threads` per query or accept that each query is slower. This is the same trade every MPP engine makes; what is unusual is that ClickHouse makes it the operator's explicit problem rather than hiding it behind a queue.

### 3.2 The SQL Extensions That Matter

ClickHouse speaks SQL, but the dialect is a superset with idioms that do not exist elsewhere. Six of them carry real design weight.

**Aggregate functions are first-class values via `-State` / `-Merge`.** Any aggregate function has a `-State` variant that returns a serialised intermediate state, and a `-Merge` variant that consumes such states. Combined with `AggregatingMergeTree`, this allows partial aggregation to be stored and later combined — which is how a table can hold pre-aggregated summaries that remain exactly reconstructible rather than approximately so. The sibling `mpp_databases_guide.md` §4.5 covers the same mechanism from the comparative angle; here the point is that this is the mechanism that makes ClickHouse materialised views trustworthy for exact aggregates.

**Combinators modify aggregate functions without rewriting them.** The `-If` combinator adds a condition (`countIf(x > 0)`), `-Array` runs an aggregate over array elements, `-Merge` consumes states, `-State` produces them, and combinations nest. The practical effect is that a great deal of what would be a `CASE` expression or a second query in another dialect is one function call. It is also a readability hazard: a nested combinator is terse to the point of being unmaintainable, and there is no style guidance in the documentation that constrains it.

**Approximate aggregation functions are provided as a family.** `uniq` returns an approximate count of distinct values, `uniqCombined` uses HyperLogLog, `uniqExact` gives the exact answer at propor­tionately higher cost, `quantiles` and `quantileTiming` give approximate quantiles, and `topK` gives the most frequent values. The design intent is that approximate answers at petabyte scale are usually good enough and dramatically cheaper. The honest caveat is that the approximation is a property of the function, not of the query, and an institution that reports a number to a regulator must use `uniqExact` and accept the cost. There is no "make it exact" switch that preserves performance.

**Window functions are supported** (`OVER (PARTITION BY ... ORDER BY ...)`), but they are a later addition than the aggregate machinery and they are materially more expensive than the aggregation paths — a window function forces a sorted intermediate state that the pipeline has to materialise. For rolling-window analytics at scale, the idiomatic ClickHouse answer is a materialised view with `AggregatingMergeTree`, or a `WINDOW`-free formulation, rather than a window function over the raw data.

**Array and higher-order functions** (`arrayMap`, `arrayFilter`, `arrayJoin`, `groupArray`, nested `Array(Tuple(...))` columns) let denormalised data be modelled richly. `arrayJoin` in particular converts an array column into rows and is the standard tool for exploding nested events — at the cost of a fan-out that can be surprising in size.

**Table functions** let a query read from outside the database: `s3()`, `url()`, `file()`, `remote()`, `mysql()`, `postgresql()`, `iceberg()`, `deltaLake()`, `numbers()`. This is what makes ClickHouse usable as a query engine over a lake rather than only over its own storage, and it is why the 2026 On-Demand Compute feature lists Iceberg and Delta among its supported sources (clickhouse.com/blog/on-demand-compute, 10 September 2026).

### 3.3 Joins and the Optimiser

Joins are where ClickHouse's design assumptions show most clearly, and where the honest reading of the documentation differs most from the marketing.

The engine implements several join algorithms — hash join, partial merge join, grace hash join, and a full sorting merge join — and the default is chosen based on the query. The classic behaviour builds the **right-hand side** into an in-memory hash table, which means two things follow directly: a join against a large table costs memory proportional to that table's build side, and the order in which the user writes the join determines the cost. There is no automatic reordering in the classic path. A designer who writes `large_table JOIN small_table` on a version without the optimiser pays for building the large side, and the fix is to write it the other way round or use `GLOBAL JOIN` and explicit hints.

Memory limits interact badly with joins. A hash join that exceeds its allowance will fail or spill depending on settings and version, and the join is the most common cause of the out-of-memory class described in §8.2. The diagnostic pattern practitioners use is to put the small dimension into a dictionary or an `IN` subquery, or to raise the join to the application layer, rather than to tune the join.

The optimiser picture is now genuinely two-track. The baseline planner is documented as rule-based, with join ordering limited. A **cost-based optimiser (CBO) exists but is documented as a component of On-Demand Compute, which is in private preview** (clickhouse.com/blog/on-demand-compute, 10 September 2026). The CBO "evaluates different execution plans and chooses a more efficient way to run your queries", and the same post states that on the stateful comparison cluster "neither is enabled". So the accurate statement as of 15 September 2026 is: ClickHouse has a cost-based optimiser, and it is not yet the default query planner in general availability. Any claim that ClickHouse "has a CBO" without that qualifier is incomplete.

The column-statistics feature in §2.5 — `countmin`, `minmax`, `tdigest`, `uniq`, `uniq_v2` — is the cardinality-estimation input a cost-based optimiser needs. Its appearance in the current MergeTree documentation is the clearest evidence that the optimiser work is deliberate rather than a single-feature release.

### 3.4 Profiling and EXPLAIN

ClickHouse's observability is good and it is deeper than most databases offer, but it is not the same shape as an OLTP query plan.

`EXPLAIN` exists in several forms — `EXPLAIN SYNTAX` (what the parser produced), `EXPLAIN AST`, `EXPLAIN PLAN` (the logical plan), `EXPLAIN PIPELINE` (the operator graph that will actually execute) and `EXPLAIN ESTIMATE` (how many rows and granules the engine expects to read). `EXPLAIN PIPELINE` is the one that matters for performance work, because it shows the queues between operators and therefore shows where a stage is not parallelising.

The real evidence, though, is in the system tables, and specifically in `system.query_log`. It records every query with its timing, its rows and bytes read, the number of parts touched, peak memory, and the full settings that were in effect. This is unusual and valuable: the settings are recorded per query, so a regression caused by a settings change is diagnosable after the fact. `SELECT ... FROM system.query_log WHERE ... ORDER BY event_time DESC` is the standard starting point for any performance investigation.

Beyond that, the surface includes `system.trace_log` for stack samples (usable to build flamegraphs), per-query progress over the native protocol, and profile events. The practical limitation is that the profiler produces data, not conclusions — there is no automated "this query is slow because of X" surface, and turning trace data into a verdict is a real skill (see §8.4).

---

## 4. The Distributed and Reliability Machinery

### 4.1 Replication and ClickHouse Keeper

A `ReplicatedMergeTree` table replicates by shipping **parts**, not by shipping rows or re-executing SQL. The local node writes the part, records its intent in a coordination log, and the replicas fetch the part from the writer. This is fundamentally different from statement-based replication and it is why replicated inserts are cheap and why deduplication is possible at all (§5.3).

The coordination layer is **ClickHouse Keeper**, and its current status is clear from the documentation: "ClickHouse Keeper provides the coordination system for data replication and distributed DDL queries execution. ClickHouse Keeper is compatible with ZooKeeper" (clickhouse.com/docs, ClickHouse Keeper guide, last modified 12 September 2026).

What Keeper actually holds is worth stating precisely, because it is often described as "just a ZooKeeper replacement" when it is a stateful dependency in the write path:

| Keeper holds | Consequence if Keeper is unavailable |
|---|---|
| Replication log (which parts exist, which replica has them) | Replicated inserts stall; readers keep serving existing data |
| Replica metadata and session state | Replicas cannot coordinate fetches |
| Distributed DDL queue (`ON CLUSTER` operations) | Schema changes across the cluster block or queue |
| Insert deduplication log | Idempotent retry of inserts is no longer guaranteed |
| **Runtime, unsupported ZK features, and Cloud** | Snapshots and logs use an incompatible format (the `clickhouse-keeper-converter` tool converts ZooKeeper data); several ZooKeeper features are unimplemented (e.g. TTL on `create`, `PERSISTENT` watches, `removeWatch`, `setWatches`, CONTAINER znodes, SASL); and Keeper is **not supported as a user-facing component in ClickHouse Cloud** |

The implementation difference from ZooKeeper is documented and material: Keeper is written in C++ and uses the **RAFT** algorithm via eBay's NuRaft, whereas ZooKeeper is Java and uses ZooKeeper Atomic Broadcast (ZAB). The docs state that "by default, ClickHouse Keeper provides the same guarantees as ZooKeeper: linearizable writes and non-linearizable reads". It speaks a compatible client-server protocol, so standard ZooKeeper clients work against it, and `clickhouse-keeper-converter` can convert ZooKeeper data to Keeper snapshots. Critically, **"the interserver protocol in ClickHouse Keeper is also incompatible with ZooKeeper so a mixed ZooKeeper / ClickHouse Keeper cluster is impossible"** — a migration is a cut-over, not a rolling change. Keeper can run standalone or embedded in the ClickHouse server, and defaults include `tcp_port` 2181, `election_timeout_lower/upper_bound_ms` 1000/2000, `snapshots_to_keep` 3, `force_sync` true, and `async_replication` false.

**Flagged:** the documentation describes Keeper as the ZooKeeper-compatible coordination system and states that a mixed cluster is impossible, but no page I retrieved declares ZooKeeper support formally deprecated or removed. Reporting "ZooKeeper is deprecated" would overstate what the sources say; the defensible statement is that Keeper is the project's coordination system and that mixed clusters are not supported.

**Operational surface.** In a self-managed cluster, Keeper is three or five extra processes the operator must run, monitor, back up, and size. Its own memory is a tunable (`max_memory_usage_soft_limit`, defaulting to 90% of physical memory via `max_memory_usage_soft_limit_ratio = 0.9`), its log and snapshot paths are separate volumes, and it is the component that sits in the write path of every replicated insert. It is a genuine availability dependency that many teams under-scope. In **ClickHouse Cloud, Keeper is not a user-facing component at all** — the documentation marks the Keeper page "Not supported in ClickHouse Cloud", because the procedure is automated. That is a real part of what the managed service sells.

### 4.2 Sharding and the Distributed Engine

Replication gives you the same data on every node. Sharding gives you different data on each node. ClickHouse handles the second with the `Distributed` engine, which is a table that owns no data and instead fans a query out across the shards of a cluster, then merges the partial results on the coordinating node.

The honest performance note is that the merge step is a bottleneck and a memory consumer. A `Distributed` table doing a `GROUP BY` must combine partial aggregation states from every shard, and a `ORDER BY ... LIMIT` must merge sorted streams. For most analytical queries this is fine and the fan-out wins. For queries with high-cardinality grouping it can be worse than a single big node.

The insert-path guidance is asymmetric and worth quoting because it is counter-intuitive. The documentation recommends inserting **directly into a MergeTree or Replicated table**, balancing requests across nodes client-side, and setting `internal_replication = true`, rather than inserting through a `Distributed` table. Inserting through the distributed layer works but "is a little less performant as writes have to be made locally on the node with the distributed table and then sent to the shards" (clickhouse.com/docs/guides/inserting-data, last modified 28 August 2026).

### 4.3 Consistency: What Is and Is Not Guaranteed

This section is the one a regulated institution should read twice, because ClickHouse's guarantees are strong where they are advertised and absent where people assume they exist.

| Property | Guarantee | Source |
|---|---|---|
| Single-insert atomicity and idempotency | A part is written atomically; if the part is truncated the table is unchanged. By default, inserts are deduplicated, so an identical retry is ignored | clickhouse.com/docs/guides/inserting-data, 28 August 2026 |
| Cross-statement transaction | **None.** No multi-statement transaction, no rollback | same |
| Read-your-write across replicas | **Not by default.** A replica may lag; the 2026 fix is replica-aware routing, Enterprise public beta | clickhouse.com/blog/replica-aware-routing-public-beta, 15 September 2026 |
| Row-level uniqueness | **Not guaranteed.** `ReplacingMergeTree` deduplicates at merge time only | clickhouse.com/docs, MergeTree reference, 11 September 2026 |
| Consistency model overall | "append-only operations and ... only eventual consistency guarantees" | clickhouse.com/docs/guides/inserting-data, 28 August 2026 |

The read-your-write gap is worth unpacking because it surprised even the vendor into a product feature. The blog introducing replica-aware routing opens by describing the symptom: a temporary table is created, a query one second later says it does not exist, because the request was load-balanced to a different replica. The fix routes requests to the same replica using an `X-ClickHouse-Replica-Tag` header on HTTP or a `--tls-sni-override` SNI value on the native protocol, hashed through an Envoy/Istio proxy layer. Two caveats from the same post matter: stickiness is **"best-effort, not a guarantee"** and breaks on upgrades, restarts and scaling; and it is **"not workload isolation"** — the replica still serves other traffic. It is available on Enterprise tier plans only.

### 4.4 Backup, Restore and What One Node Cannot Survive

Backups are taken with `BACKUP` and `RESTORE` statements, which write to a destination that can be a local disk, a mounted filesystem, or an S3-compatible object store, and can be incremental at part granularity. The practical properties that matter in an operational review:

- Replication is **not** a backup. A replicated cluster protects against node loss; it does not protect against a bad `DROP`, a bad mutation, or a corrupted part, because those propagate to every replica.
- Backups are part-granular and therefore sensitive to part count. The project explicitly lists slow backups and "no free inodes on the filesystem" among the consequences of a poor partitioning key (clickhouse.com/blog, 20 February 2026). A table with hundreds of thousands of parts produces a slow, fragile backup.
- Object-storage destinations make backups operationally tractable at scale, but restore time is bounded by download bandwidth and by how many parts must be re-fetched and re-merged.
- ClickHouse Cloud automates backup and offers point-in-time restore on its services; self-managed clusters do not get this for free, and the community `clickhouse-backup` tool is the usual third-party answer.

**What a single-node deployment cannot survive.** Stated plainly, because it is easy to forget while a single node is performing beautifully:

1. **Loss of the node loses the data and the service.** There is no second copy. Recovery is bounded by the time to restore the last backup — hours, typically, and longer for a large dataset.
2. **Loss of the disk silently degrades to data loss** if the last backup is old. The gap between the last backup and the failure is unrecoverable.
3. **No zero-downtime upgrade.** A version upgrade on a single node is an outage.
4. **No failover for a hung query, and no solution to 1, 2 or 3 that does not involve a second copy.** A query that saturates memory cannot be failed over; it must be killed, and the kill may itself be delayed. The remediation is standard resilience practice: a minimum of two nodes with replication, an odd Keeper quorum of three or five, backups to object storage on a schedule tested by an actual restore, and a documented recovery procedure with a measured restore time. §10 cross-references `../banking/operational_resilience_framework_guide.md` for the RTO/RPO framing rather than restating it.

---
## 5. Ingestion and the Data Lifecycle

### 5.1 The Insert Model and Why Batching Is Everything

An insert in ClickHouse is not a row-level operation. It is a block-level operation that produces a **part**, and every part is a separately indexed, separately compressed, separately read unit of storage. This single fact generates the most common production failure in the entire product, and the project's own documentation treats it as the first of thirteen mistakes to avoid.

The guidance is specific and quoted rather than paraphrased: "By default, each insert sent to ClickHouse causes ClickHouse to immediately create a part of storage containing the data from the insert together with other metadata that needs to be stored. Therefore, sending a smaller amount of inserts that each contain more data, compared to sending a larger amount of inserts that each contain less data, will reduce the number of writes required. Generally, we recommend inserting data in fairly large batches of at least 1,000 rows at a time, and ideally between 10,000 to 100,000 rows" (clickhouse.com/docs/guides/inserting-data, last modified 28 August 2026).

Why part count matters, mechanically:

| Effect | Mechanism |
|---|---|
| Query slows, startup slows | Every part contributes its own index to evaluate and its own files to open; the engine reconciles every part on start |
| Merge pressure rises | More parts means more merge work, competing with queries for I/O and CPU |
| Inserts start failing | `parts_to_throw_insert` and `max_parts_in_total` limits are exceeded and the insert errors |
| Keeper load rises | In replicated configurations, more parts means more entries in the replication log |
| Backups slow and can fail | Part count translates to file count, and file count translates to inode pressure |

The "too many parts" error is therefore not an edge case. It is the structural consequence of an insert pattern that treats ClickHouse like a row-store, and the documentation names four causes explicitly: a poorly chosen partition key with high cardinality (parts in different partitions are **never** merged), many small inserts, excessive materialised views, and mutations (clickhouse.com/blog, "13 mistakes", 20 February 2026).

The partition-key cause deserves emphasis because it is an easy mistake to make for good reasons. Partitioning looks like an optimisation, and a developer who partitions by `toStartOfHour(timestamp)` because the dashboards filter by hour believes they are helping. They are not: partitioning is a data-management tool, and a high-cardinality key creates thousands of directories whose parts can never be merged with each other. The documented guidance is a partition-key cardinality under roughly 1,000 and month-level granularity.

**Defaults and settings that govern this.** The docs page groups the relevant settings under `parts_to_*`, `max_parts_*`, `max_suspicious_broken_parts` and `merge_selector_*` (clickhouse.com/docs/reference/settings/merge-tree-settings, last modified 14 August 2026). The widely-cited defaults for `parts_to_throw_insert` (300) and `parts_to_delay_insert` (150) appear in practitioner material and in older documentation; I could not extract the numbers from the current settings page in this pass, so treat the specific values as **(flagged)** and read them from the live page before relying on them. What is not in doubt is that both a delay threshold and a hard-throw threshold exist, and that operators should alert on part count long before either is reached.

### 5.2 Asynchronous Inserts, Buffers and Streams

When client-side batching is genuinely impossible — hundreds of agents each shipping one log line at a time — ClickHouse can do the batching server-side. Asynchronous inserts buffer incoming blocks in memory and flush them as a single part.

The documented flush triggers and their defaults are exact: `async_insert_max_data_size` (default **1 MB**), `async_insert_busy_timeout_ms` (default **1 second**), and `async_insert_max_query_number` (default **100**), whichever fires first (clickhouse.com/blog, 20 February 2026). Two properties of this mode must be understood before enabling it:

1. **Buffered data is not queryable.** It becomes searchable only once flushed to storage. A freshness requirement measured in milliseconds is incompatible with a one-second busy timeout unless the timeout is lowered.
2. **Acknowledgement semantics are a settings choice.** `wait_for_async_insert = 1` is the **default** and means the client is told the write succeeded only after the data is durably on disk. Setting it to 0 returns an acknowledgement before the flush, which is faster and can silently lose data if the server dies in the window. The default is the safe one; a configuration that changes it should be a deliberate decision with a stated reason.

The **`Buffer`** table engine is the older mechanism and the documentation is candid about its drawbacks: it is "not replicated, is not compatible with FINAL or SAMPLE, and can lose data on abnormal server restarts", and in ClickHouse Cloud with `SharedMergeTree` each node maintains independent buffer state, which adds "further complexity". The single genuine advantage is that buffered rows are queryable before the flush. The project's own recommendation is that for "nearly all use cases, async inserts are the recommended approach" (clickhouse.com/blog, 20 February 2026).

**Stream sources.** ClickHouse can consume from Kafka and similar brokers through dedicated engine integrations, typically by an engine table reading the topic plus a materialised view writing into a MergeTree target. The 2026 product line also adds Prometheus remote write as an ingest protocol for the `TimeSeries` engine, which is a private preview (clickhouse.com/blog/introducing-promql, 15 September 2026). The architectural point is that ClickHouse's stream ingestion is a *pull-and-insert* pattern, not a checkpointed stream operator with the end-to-end semantics that `apache_flink_guide.md` §4 describes. The consequence is covered next.

### 5.3 Deduplication and the Exactly-Once Question

ClickHouse offers a real and useful guarantee here, and it is narrower than "exactly-once" in the streaming sense.

For `MergeTree` family tables, inserts are deduplicated by default, keyed on the content and order of the inserted block. The documentation's statement of the benefit: "By default, inserts into ClickHouse are synchronous and idempotent (i.e. performing the same insert operation multiple times has the same effect as performing it once)" — so a client that sends an insert, receives no acknowledgement because the network failed, and retries the identical block, will not have the data written twice (clickhouse.com/docs/guides/inserting-data, last modified 28 August 2026).

Three qualifications make the guarantee precise rather than rhetorical:

1. **It is at-least-once delivery plus deduplication, not exactly-once processing.** A message consumed from Kafka, transformed, and inserted with a different value on each attempt will not deduplicate, because the block differs.
2. **The deduplication window is bounded.** For replicated tables the deduplication log is retained for a configured number of blocks and a configured time window; an insert retried outside that window is a new insert. The relevant settings are grouped under `replicated_deduplication_window_*` in the MergeTree settings reference (last modified 14 August 2026).
3. **The retry must be byte-identical.** Same data, same order. `insert_deduplication_token` exists as the explicit mechanism for the producer to assert identity when it cannot rely on block-level hashing.

The correct architectural summary for a regulated institution is: ClickHouse gives **idempotent insertion**, which is exactly what a retry-on-failure ingestion pipeline needs, and it does **not** give transactionally coupled read-modify-write processing. If the workload requires the latter, the transformation belongs in a stream processor and ClickHouse is the sink.

### 5.4 TTL, Tiering and Mutations

**TTL and tiered storage.** `TTL` clauses on a table (or on individual columns) express a lifespan, and the action on expiry is one of `DELETE`, `TO DISK 'x'`, `TO VOLUME 'x'`, or `GROUP BY` for aggregation of expired rows. There can be at most one `DELETE` rule. This is the mechanism behind the pattern nearly every log-analytics deployment uses: hot data on NVMe for seven days, warm data on spinning disk or network storage for ninety days, deletion after a year. It is declarative and enforced by background parts movement, so it is an operational policy rather than an application responsibility.

The engineering value of `TO VOLUME` is larger than it first appears. Because the movement is part-granular and driven by the TTL expression, a deployment can honestly claim a tiering policy without an ETL job, and the data remains queryable throughout. The consequence to watch is that moving parts consumes the same I/O and background-thread budget as merges, so a TTL that expires a large fraction of the data at once produces an I/O spike — a scheduled or staggered expiry is kinder than a uniform one.

**Mutations.** ClickHouse performs best on immutable data, and the project says so: "any design pattern which requires data to be updated post-insert should be reviewed carefully" (clickhouse.com/blog, 20 February 2026). Two mechanisms exist and they are not equivalent.

| Mechanism | Statement | Cost profile | Rollback |
|---|---|---|---|
| Classic mutation — `ALTER TABLE ... UPDATE` / `DELETE` | Rewrites whole data parts containing the affected columns | CPU- and I/O-intensive; uses **the same thread pool as merges**; in self-managed replicated clusters **each replica applies the mutation independently** | **Cannot be rolled back**; `KILL MUTATION` cancels a running mutation, it does not undo one |
| Lightweight update — `UPDATE` with patch parts | Writes a compact patch part holding only the changed column values plus row locators; applied during reads and materialised during merges | Much cheaper; the vendor claims "up to 1,000× faster" for many workloads (vendor claim, not independently verified) | Same — still not transactional |

`system.mutations` shows scheduled and running mutations, and it is the table an operator should alert on. For deletes, the same patch mechanism uses a `_row_exists = 0` mask rather than rewriting columns.

The practical guidance that follows from the table is unambiguous: **model for immutability**. Where a correction is inevitable — a mis-priced trade, a mis-attributed transaction — prefer an append-only correction record that the query layer reconciles, or a `ReplacingMergeTree` versioned pattern, over an `ALTER ... UPDATE`. A monthly mass update on a large table is a scheduled incident, and the incident is that merges fall behind while the mutation runs, parts accumulate, and inserts begin to fail (§8.1).

---

## 6. The SQL Surface, the Tooling and the Ecosystem

### 6.1 Interfaces and Drivers

ClickHouse exposes four ways in, and the choice between them is a real engineering decision rather than a style preference.

| Interface | Transport | When it is the right choice | Notes |
|---|---|---|---|
| **Native protocol** | TCP, binary columnar | Highest-throughput ingest and long-running queries | Least per-request overhead; query progress tracking; official `clickhouse-client` |
| **HTTP** | HTTP/HTTPS | Default for integrations, proxies and serverless clients | Load-balancer friendly; insert and query both supported; also the path the MCP server uses |
| **MySQL / PostgreSQL wire** | Compatibility protocols | Existing BI tools and drivers that only speak those protocols | Compatibility layers, not clones; the Postgres wire is distinct from the Managed Postgres product in §1.4 |

The documented preference for ingest throughput: prefer the **Native** format because it is already column-oriented, so "the server has to do a minimal amount of work"; `RowBinary` is an acceptable row-format alternative and more efficient than JSON; `JSONEachRow` "will incur a CPU overhead in ClickHouse for parsing" (clickhouse.com/docs/guides/inserting-data, last modified 28 August 2026). The HTTP interface is documented as "often preferable to ClickHouse's native protocol as it allows traffic to be easily switched with load balancers", at the cost of "small differences in insert performance" (same source).

Official language clients exist for Python, Go, Java, Node.js and Rust, alongside JDBC and ODBC drivers for the JVM and BI ecosystems. The practical selection advice is to use an official client rather than hand-rolling the protocol, because the official clients implement asynchronous inserts natively and handle the retry-idempotency contract of §5.3 correctly.

### 6.2 The Observability Surface

ClickHouse's introspection surface is one of its strongest features and is genuinely unusual in depth. The tables an operator lives in:

| Table | What it answers | Typical use |
|---|---|---|
| `system.parts` | Which parts exist, in which partition, at what level, how many rows and bytes | Part-count alerting; detecting a stalled merge |
| `system.merges` | Which merges are running, on what, and for how long | Diagnosing merge pressure before it becomes an incident |
| `system.mutations` | Which mutations are scheduled or running | Guarding against an unbounded update |
| `system.query_log` | Every executed query with timing, bytes, parts read, peak memory, and the **effective settings** | Post-hoc performance investigation; settings-regression diagnosis |
| `system.metrics`, `system.asynchronous_metrics`, `system.replication_queue`, `system.trace_log` | Current gauges; sampled trends; pending replication actions per replica; stack samples | Health and capacity dashboards; diagnosing a lagging replica; flamegraph construction |

The single most useful of these is `system.query_log`, because it records the settings in effect per query. That turns a class of problem — "performance changed on Tuesday and we do not know why" — into a query rather than a guess. The gap is that there is no automated analysis on top of it; the operator has to know what to look for.

For metrics, the standard pattern is an exporter over these tables into Prometheus and Grafana, which is the same shape as any other database's operational tooling; the sibling `technology/low_latency_genai_patterns_guide.md` owns the latency-instrumentation framing and is not duplicated here.

### 6.3 Integrations and the Ecosystem

The integration surface is broad, and it is worth separating the parts that are official from the parts that are community-maintained, because the support expectations differ.

**Official and first-party:** the ClickHouse Cloud console and API; ClickStack, the observability stack built around ClickHouse (used by the PromQL preview, clickhouse.com/blog/introducing-promql, 15 September 2026); the chdb embedded engine and its Postgres extension (clickhouse.com/blog/introducing-chdb-postgres, 8 September 2026); WalShadow for Postgres physical-WAL replication (clickhouse.com/blog/introducing-walshadow, 10 September 2026); and the ClickHouse MCP server, which the documentation pages surface directly as a "Set up the ClickHouse documentation MCP server" action and which the vendor blog covers in an agent-vector context (clickhouse.com/blog/mcp-toolbox-clickhouse-vectors, 7 September 2026).

**Connector and BI ecosystem:** the `dbt-clickhouse` adapter for SQL transformation workflows; ELT connectors through Airbyte and Fivetran; native `s3()`, `url()`, `file()`, `iceberg()` and `deltaLake()` table functions for lake reads; Kafka and Parquet ingestion; and BI/dashboard connectors for Superset, Grafana, Tableau and Power BI. The pattern to expect is that these connectors are usually thin — they speak HTTP or the MySQL/Postgres wire — and their quality varies by vendor, so the due-diligence question for any connector is who maintains it and at what cadence.

**Community projects that matter operationally:**

| Project | What it is | Status note |
|---|---|---|
| `Altinity/clickhouse-operator` | Kubernetes operator for provisioning and managing ClickHouse clusters | The de-facto standard operator; community-maintained by Altinity |
| `Altinity/clickhouse-backup` | Backup and restore tooling for self-managed deployments | Widely used alongside the `BACKUP`/`RESTORE` statements |
| Altinity; `chproxy` | Commercial support for self-managed ClickHouse; HTTP proxy and load balancer with caching and user routing | A genuine alternative vendor, which matters for procurement leverage; `chproxy` is common fronting a self-managed cluster |
| Vector, Fluent Bit, Grafana, Superset, Tableau, Power BI | Log and metric shippers with ClickHouse sinks; dashboards and BI | The usual observability ingest path; connector quality varies, so test before committing |

**Flagged:** I did not fetch the repositories for `clickhouse-operator`, `clickhouse-backup` or `chproxy` in this pass, and their current release status, maintainer activity and ClickHouse-version support are therefore not verified. Their existence and general role are widely reported and consistent across sources; treat the specific versions and support matrices as unverified and check the repositories directly. Altinity's commercial offering is likewise asserted on the basis of its public presence rather than a fetched page.

**The honest ecosystem read.** The ecosystem is large enough that almost any integration exists, and uneven enough that almost any integration might be abandoned. The engine itself is unusually self-sufficient — table functions cover lake reads, HTTP covers almost everything else, and the official clients cover the main languages — so the practical strategy is to depend on the engine and the official surfaces, and to treat every third-party connector as a component with an owner and a review date rather than as infrastructure.

---
## 7. The AI and Vector Capability

The repository's `ai_llm/rag/` cluster owns vector-database comparison and RAG patterns, and `technology/low_latency_genai_patterns_guide.md` owns TTFT/TPOT and serving latency. This section covers only what ClickHouse itself ships, and it distinguishes shipped from announced throughout, because the gap between the two is unusually wide in this vendor's 2026 output.

### 7.1 Vector Search as Shipped

Vector search reached general availability in **ClickHouse 25.8**. The documentation's own framing is that "vector similarity indexes are available in ClickHouse version 25.8 and higher" (clickhouse.com/docs/reference/engines/table-engines/mergetree-family/annindexes, last modified 15 August 2026). Third-party release-notes coverage describes GA vector search in the 25.8 LTS line (Aiven for ClickHouse 25.8 release post). **(Verified for the version and availability claim from the docs; the exact GA announcement date is flagged in §13.)**

What is actually there, precisely:

**Storage.** Embeddings live in a column of type `Array(Float32)`, `Array(Float64)` or `Array(BFloat16)`. The current documentation "generally recommend[s] using BFloat16 arrays" for practical usage. There is also a `QBit` data type for quantised storage with query-time precision control (see below).

**Exact search.** With no index, a nearest-neighbour query is a brute-force scan written as a normal SQL `ORDER BY <distance>(...) LIMIT N`. The documentation is explicit about the cost profile: runtime "is generally proportional to the number of stored vectors, the number of vector elements (= the dimension), and the bit width of the vector elements", and because it "performs a brute-force scan of all vectors, the runtime depends also on the number of threads by the query". This is not a weakness to be glossed over — a brute-force scan over a columnar array column with SIMD distance kernels is genuinely fast, and for datasets in the low millions of vectors it is often the right answer with no index at all.

**Approximate search.** The mechanism is the **vector similarity index**, declared as `INDEX <name> vectors TYPE vector_similarity(...)`. It is implemented as a **special kind of skip index**, not as a separate index family. The parameters:

| Parameter | Value / default | Note |
|---|---|---|
| `<type>` | **`'hnsw'` is the only method available as of the 15 August 2026 docs** | HNSW = hierarchical navigable small world proximity graph |
| Distance function / `dimensions` | `L2Distance`, `cosineDistance`, `dotProduct`; dimensions must match the array cardinality | Mismatch causes the index to be discarded and an error returned; ASC for L2/cosine, DESC for dotProduct |
| `quantization` | `bf16` default; also `f64`, `f32`, `f16`, `i8`, `b1` | Affects the proximity graph only, not the stored column |
| `M` / `ef_construction` | 32 and 128 defaults | Docs advise not tuning unless you understand the implications |
| `GRANULARITY` | **100,000,000 default**, versus 1 for ordinary skip indexes | Deliberately enormous so few graphs are built even for large parts |

Three operational facts follow, and they are the ones a designer needs:

1. **The index is a skip index, so it must be materialised.** `ALTER TABLE ... ADD INDEX` builds the index only for subsequently inserted data. Existing data requires `ALTER TABLE ... MATERIALIZE INDEX <name> SETTINGS mutations_sync = 2` — which is a mutation, with the cost profile described in §5.4.
2. **Sort direction is constrained by the distance function.** `L2Distance` and `cosineDistance` are used with `ORDER BY ... ASC`; `dotProduct` is used with `ORDER BY ... DESC`, because a higher dot product means higher similarity. An index built for one direction cannot serve the other.
3. **Column type and cardinality are constrained.** Only `Array(Float32)`, `Array(Float64)` and `Array(BFloat16)`; a differing array cardinality at index-creation time discards the index.

I did **not** find an "experimental" or "not production-ready" label on the ANN index page in this pass. The accurate statement is that the vector similarity index is a documented, GA-since-25.8 feature with a single HNSW implementation, default parameters the vendor advises against tuning, and a note inviting users to open an issue if problems arise — which is not the same as an experimental label and should not be reported as one.

**`QBit` and query-time precision.** ClickHouse also ships a `QBit` data type with transposed distance functions such as `L2DistanceTransposed(vec, reference, precision_bits)`, where the third argument selects precision at query time — 64 bits for full precision, 12 bits for a faster approximate answer. The documented benefit is reduced I/O, plus reduced computation when the underlying type is `Float32` and precision is 16 or below. This is a distinctive capability: it lets one stored column answer both an approximate and a near-exact query, which most vector stores make you choose at write time.

### 7.2 Hybrid Search and Full-Text

The combination that matters in practice is vector similarity plus keyword filtering in one query, and ClickHouse supports it natively in the sense that all of it is SQL over the same table. A query can filter on a token or n-gram index (`tokenbf_v1`, `ngrambf_v1`), on a scalar predicate, on a date range, and then order by vector distance — in one statement, over one storage layer, with no cross-system join.

The current MergeTree settings reference includes a `text_index_*` group with seven settings, and the documentation includes text index material, which indicates that a fuller inverted-index capability has shipped alongside the bloom-based skip indexes. **(Verified that text-index settings exist in the settings reference, last modified 14 August 2026; the precise feature scope and GA status of the text index were not read in full and are flagged in §13.)**

The honest summary of hybrid search in ClickHouse: the *mechanism* is native and the *ranking semantics* are yours to build. ClickHouse gives you distance functions, token filters and a `LIMIT`; it does not give you reciprocal rank fusion, a relevance score combining BM25 with vector similarity, or a learned ranker out of the box. That is a deliberate scope boundary — those belong to a search system — and a team that wants hybrid ranking will be writing the fusion logic themselves, most likely against `tokenbf_v1`/`text_index` recall plus an HNSW-accelerated vector candidate set.

### 7.3 LLM-over-ClickHouse as Shipped

Three distinct things exist here and they are at three different maturity levels.

**AI Functions.** A family of SQL functions that call an LLM or an embedding provider directly from the engine. They shipped across three releases — `aiGenerate()`, `aiClassify()`, `aiExtract()` and `aiTranslate()` in 26.4; `aiEmbed()` in 26.6; then `aiFilter()`, `aiRedact()` and `aiSimilarity()` in 26.8 — and the vendor describes the family as **"currently in Beta"** with functionality "constantly being added and improved". In ClickHouse Cloud they are in **private preview**, and the post states that Cloud requires no extra configuration while self-managed requires two named collections holding provider credentials plus the settings `ai_function_text_default_credentials` and `ai_function_embedding_default_credentials` (clickhouse.com/blog/ai-functions-in-clickhouse, 11 September 2026).

Two operational details from the same source are the ones an architect needs. First, the implementation makes **a remote HTTP call to the configured provider per input** (batched for the embedding functions), so latency and cost are governed by an external API and the engine is holding a connection and a thread while it waits — the placement of such a call inside a scan over millions of rows is a cost decision, not a convenience. Second, any OpenAI-compatible endpoint works by setting `provider = 'openai'` and pointing at your service, so a local model is supported, which matters enormously for an institution that cannot send data to a third-party API.

The genuinely interesting function is `aiFilter()`, which returns `UInt8` and is therefore usable in `WHERE`, `PREWHERE` and `JOIN ... ON`. That turns a natural-language predicate into a query operator. The vendor's own guidance on it is the practical caveat: "Since AI Function calls can be slow to process, it's best to apply cheap predicates first in a subquery, then call the LLM predicate in the outer query" — i.e. filter cheaply, then pay for inference on the survivors.

**MCP server.** ClickHouse ships an official Model Context Protocol server, exposed from its documentation pages as a one-click "Set up the ClickHouse documentation MCP server" action and covered on the vendor blog in an agent-tooling context (clickhouse.com/blog/mcp-toolbox-clickhouse-vectors, 7 September 2026). This is the agent-facing integration surface, not a text-to-SQL feature in itself.

**The ChatGPT Data agent launch partnership.** ClickHouse announced that it is a launch partner for the Data agent in ChatGPT Work (clickhouse.com/blog/chatgpt-data-plugin, 10 September 2026). This is a distribution and integration announcement, and the caution is to read it as exactly that. It is not a statement that ClickHouse generates mSQL from natural language, nor that questions in the ChatGPT interface are safely answerable against an arbitrary schema. Natural-language-to-SQL accuracy is bounded by schema quality, naming, and the presence of documentation — the same bounds as any other SQL engine, and the vendor's marketing does not remove them.

**The honest position on text-to-SQL.** ClickHouse does not ship a text-to-SQL engine of its own that I could verify. What it ships is: SQL functions that invoke an LLM (Beta), an MCP server for agent tooling, and a launch partnership with a consumer AI product. Everything else in the text-to-SQL story is built by someone else on top of the engine.

### 7.4 The Practical Trade-Offs

Using an analytical database as a retrieval store is a real architectural choice with real consequences, and the trade is best expressed as a set of conditions under which it wins.

**It wins when the retrieval is one part of a larger analytical question.** The characteristic ClickHouse-shaped retrieval problem is: filter a large corpus by structured predicates and time range, apply a keyword filter, then rank a candidate set by vector similarity, in one query over data that is already there. No cross-system data movement, no consistency window between the operational store and the retrieval store, no second copy of the corpus. This is a genuinely strong position and it is why the combination of `WHERE ... AND ORDER BY cosineDistance(...) LIMIT k` is compelling.

**It wins on operational surface area.** One system to run instead of two. For an organisation that already operates ClickHouse for observability or analytics, adding retrieval is nearly free. For an organisation that does not, adding ClickHouse solely as a vector store is a poor trade — the operational cost of a ClickHouse cluster (§8) is not small, and a purpose-built vector store is simpler for pure similarity search. **It is weaker on ranking quality:** the engine gives recall-oriented primitives, not relevance engineering — no BM25-with-vector fusion, no learned rerankers, no relevance tuning surface — so a search-quality-critical application will end up building those, and building them well is a specialism.

**It is weaker for frequent single-vector updates.** Embedding refreshes are mutations, with the cost profile in §5.4. A corpus that is re-embedded continuously — because the model changed, or because documents change constantly — collides with the immutable-part design. A corpus that is append-mostly does not.

**It is exposed to the shape of the recall.** HNSW over a skip index with a default granularity of 100 million means the index geometry is coarse by design, and with a single method available and default parameters the vendor advises against tuning, the recall/latency knob is largely fixed. Measuring recall on your own data is not optional.

**And the cost of the AI Functions path is external.** Remote inference is billed by the provider and bounded by the provider's latency, and putting it inside a scan multiplies both. For a large backfill, a batch job calling an embedding API is usually the better instrument than `aiEmbed()` over a table.

The synthesis: **ClickHouse is a strong retrieval store when retrieval is a filter over data you already hold and a purpose-built vector database would be a second system with no other purpose. It is a weaker retrieval store when similarity search is the product.** That is the same conclusion the repository's `ai_llm/rag/vector_databases_guide.md` reaches from the vector-database side, and it is stated here only in the ClickHouse-specific form.

---

## 8. The Operational Reality and the Known Hard Parts

This is the section that decides whether a deployment succeeds. Everything above is architecture; everything here is what goes wrong in production, and where the failure is documented by the project it is cited as such, while where it is practitioner consensus it is labelled as such.

### 8.1 Small Parts and Merge Pressure

This is the defining failure mode of the product. It is not an edge case and it is not rare; it is the first item on the vendor's own list of common mistakes, and it is worth understanding as a *cascade* rather than an error.

```
  wrong insert pattern ─► many small parts ─► merge queue backs up
                                                      │
                     merges compete with queries for CPU + I/O
                                                      │
                        query latency degrades ◄───────┘
                                                      │
                     parts_to_delay_insert trips ─► inserts THROTTLED
                                                      │
                     parts_to_throw_insert trips ─► INSERTS FAIL
                                                      │
                     producer retries ─► MORE SMALL INSERTS ─► worse
```

The critical property of the cascade is that the producer's retry behaviour makes it worse. An ingest pipeline that receives an insert failure and immediately retries with the same small batch is driving the system further into the failure. The correct response to a throttling signal is backoff and batch coalescence, and the correct response long before that is a part-count alert at a threshold well below either limit.

The four documented causes, restated as fixes:

| Cause | Fix |
|---|---|
| High-cardinality partition key (parts in different partitions never merge) | Partition by month or not at all; keep partition cardinality in the hundreds |
| Many small inserts | Batch to 10,000–100,000 rows per insert; or enable async inserts with an honest timeout |
| Too many materialised views | Consolidate views; each view writes its own parts into its target table |
| Mutations | Avoid them; if unavoidable, schedule them and watch `system.mutations` |

**Alerting that actually works:** part count per table (`system.parts` where `active`), merge queue depth and age (`system.merges`), and insert failure rate at the producer. Those three cover the cascade. Alerting only on the error message means alerting after inserts are already failing.

**Practitioner consensus, not documentation:** the most effective single change in most ClickHouse rescues is to stop inserting row-by-row from an application and insert from a batch producer instead. This is consistent with everything the documentation says, but the documentation phrases it as guidance rather than as the diagnosis of a live incident.

### 8.2 Memory and the OOM Class

The second hard part is memory, because a query's memory use is a property of the query rather than of the data volume, and it is not visible until it fails.

The mechanisms the documentation names:

| Setting / mechanism | What it governs | Operational use |
|---|---|---|
| `max_memory_usage` | Per-query memory ceiling | The first line of defence; set per user or per profile |
| `max_bytes_before_external_group_by` / `max_bytes_before_external_sort` | Thresholds at which `GROUP BY` and `ORDER BY` spill to disk | Trades latency for survival on high-cardinality aggregation and large sorts |
| Memory tracker | Per-query and per-server accounting that backs the limits | The mechanism by which the limits are enforced rather than advisory |
| `memory_overcommit` | Whether the server may overcommit memory across queries | Decides whether a burst of concurrent queries is admitted or refused |

The honest characterisation: ClickHouse's memory limits are enforced, not advisory, which is better than a system that dies from OOM-killer intervention. The consequence is that a query over its ceiling **fails** rather than completing slowly, and a deployment with a tight `max_memory_usage` will see query failures under concurrency that look like instability but are actually admission control working. The tuning question is whether to fail the query or to spill to disk, and there is no universally right answer — spilling to disk on NVMe at high concurrency can produce worse total behaviour than failing fast.

The join interaction from §3.3 is the same problem in a specific shape: a hash join's build side is an in-memory data structure sized by one of the joined tables, so a join is the query type most likely to trip the limit, and the fix is to change the join, not the limit.

**Practitioner consensus, not documentation:** the recurring operational pattern is that per-query memory limits are set generously to stop legitimate queries failing, which then allows a burst of concurrent heavy queries to collectively exhaust the machine. The defence is concurrency limiting at the proxy layer plus a realistic per-query ceiling, which is a control-plane decision rather than a database setting.

### 8.3 Joins, Lookups, Upgrades and Keeper

**Joins.** As §3.3 sets out: the baseline planner is rule-based, cost-based optimisation is tied to a private-preview feature, join ordering is limited, and large joins are memory-bound. The practitioner mitigation set is to keep dimensions small and put them in dictionaries or `IN` subqueries, to denormalise at ingest time where the schema allows it, and to accept that some joins belong in the application or in a lakehouse engine rather than in ClickHouse. Denormalisation is genuinely the idiomatic answer here and it is not a hack — it is what the storage model rewards.

**Lookups.** Point lookups by primary key are served from the sparse index at granule granularity, which means an 8,192-row read for a one-row answer. For low-to-moderate rates this is invisible. For high rates — a serving path doing thousands of lookups per second — it is an expensive way to fetch a row, and the right architecture is a cache, a key-value store, or a dictionary.

**Upgrades.** The release cadence is the operational hazard. Monthly stable branches with three branches patched on a single day (verified 15 September 2026, §1.4) means the software changes fast, and the project's own guidance on beta and experimental features is that users should not build core functionality on them because "both require the user to explicitly enable them via a setting" and on ClickHouse Cloud "experimental features must be requested through support" (clickhouse.com/blog, 20 February 2026). In practice this means: pin to LTS for the core, test monthly stable in a staging cluster, and treat every upgrade as a change with a rollback plan. A replicated cluster does support rolling, zero-downtime upgrades — `MergeTree` engines enable "high availability, failover, and zero downtime upgrades" per the MergeTree reference — but that requires each node to be upgradeable independently, which is a property of the *schema and client versions*, not just of replication.

**Keeper as an operational surface.** Three or five additional processes, a separate log and snapshot volume, a raft quorum that must preserve its majority, and a component that sits in the write path of every replicated insert (§4.1). Losing Keeper quorum stops replicated inserts and distributed DDL while reads continue, which is a partial failure that is easy to misdiagnose. Keeper is not optional in a self-managed replicated deployment, and it is the component most often under-provisioned and under-monitored. In ClickHouse Cloud it is not a user-facing component at all, which is a genuine part of the managed service's value.

### 8.4 Skills and Cost

**Skills scarcity is real and it is the most under-estimated cost.** ClickHouse is not difficult to start with and it is difficult to operate well. The specific competencies that matter are: schema design around `ORDER BY` (a decision that is expensive to reverse because it requires a rewrite), partition-key discipline, merge and part-count monitoring, query-log forensics, memory-limit tuning, and Keeper operation. Practitioners with all six are uncommon, and the shortage is structural rather than temporary because the disciplines are specific to this engine's model. The mitigation is to adopt ClickHouse Cloud or a supported distribution if the organisation cannot staff the operating discipline, and to accept that self-managing a replicated ClickHouse cluster for a team of two is a choice with a high ongoing cost.

**Cost profile.** The honest comparison is not "ClickHouse is cheap". It is:

- **Versus a general-purpose warehouse:** ClickHouse wins on scan-heavy, high-concurrency, frequent-refresh workloads, and by a large margin on storage cost because columnar compression with per-column codecs is more aggressive than generic warehouse compression. It does not necessarily win on elastic, infrequent, large batch-joins with modest concurrency, which is what a cloud warehouse is optimised for.
- **Versus a search engine for log analytics:** this is where the vendor's own argument is strongest — ClickHouse's Series C post argues that search-oriented technologies "become prohibitively expensive for structured analytics—using 10x more in storage and compute" (clickhouse.com/blog, 29 May 2025). That is a vendor claim, but it is directionally consistent with the architectural difference, and log analytics is where ClickHouse displacement of search engines actually happens.
- **Versus running it yourself:** ClickHouse Cloud transfers Keeper operation, autoscaling, backups and part management to the vendor. The premium over raw compute is real and is usually cheaper than the headcount the alternative requires.
- **The hidden cost is denormalisation and schema iteration.** Because `ORDER BY` and partition keys are expensive to change, and immutable data makes corrections awkward, getting the model wrong is paid in rewrites. Teams that iterate on their schema weekly will feel this; teams whose analytical model is stable will not.

**Vendor benchmarks are vendor benchmarks.** The September 2026 CostBench posts and the ClickHouse-versus-Snowflake performance-per-dollar post are published by ClickHouse about ClickHouse (clickhouse.com/blog, 8 and 10 September 2026). They are not worthless and the methodology is at least published, but any decision that rests on them should be re-run against the institution's own data and query mix. That is true of every vendor benchmark and it is said here because this vendor publishes a great many of them.

---
## 9. The Comparison — A Positioning Table Only

**The peer-by-peer analysis lives in `technology/data/mpp_databases_guide.md`.** That guide's §4 places ClickHouse among six peers (§4.1 Greenplum, §4.2 Redshift, §4.3 Snowflake, §4.4 Doris, §4.5 ClickHouse, §4.6 BigQuery) and its §5 carries the detailed comparison table, at a depth this guide deliberately does not repeat. What follows is a positioning summary for a reader who arrived here first.

| System | Concurrency | Ingest | Join capability | Operational burden | Cost model | Ecosystem |
|---|---|---|---|---|---|---|
| **ClickHouse** | Very high on scan-and-aggregate; bounded by per-query thread budget | Very high, append-only, batched | Weakest axis: rule-based baseline planner, memory-bound large joins | High self-managed (Keeper, parts, merges); low on Cloud | Self-managed: commodity hardware + headcount. Cloud: consumption | Broad and uneven; official clients strong |
| **Apache Druid / Apache Pinot** | High to very high, purpose-built for concurrent slice-and-dice and serving-latency workloads | High, streaming-native | Weak; denormalise or pre-aggregate before ingest (Pinot uses star-tree aggregation) | High; several component types to run | Cluster size | Observability-centric (Druid); strong in user-facing analytics (Pinot) |
| **Apache Doris** | High | High | **Better than ClickHouse**; more mature optimiser and join paths | Moderate | Self-managed cheap; commercial via SelectDB | Growing; MySQL protocol native |
| **StarRocks** | Very high | High | **Better than ClickHouse**; explicit CBO, vectorised joins | Moderate | Self-managed; commercial CelerData | Growing |
| **Snowflake / BigQuery / Redshift** | Moderate to high; concurrency is a cost lever | Good, batch-oriented | Strong; mature optimisers | Very low to moderate (fully managed) | Credit consumption; slots; provisioned or serverless | Very broad; cloud-native to each vendor |
| **Lakehouse engines (Databricks, Trino/Iceberg)** | Moderate | Good (Trino: none — query-only) | Strong; mature optimisers | Moderate-to-high | Compute consumption; Trino avoids storage lock-in | Open, wide connector set |

**Competitor verification:** all of the above exist and are actively developed as of September 2026 — Druid, Pinot and Doris as Apache Software Foundation projects, StarRocks with commercial arm CelerData and Doris with SelectDB, Trino as a query engine over Iceberg, and the cloud warehouses as incumbents. Their specific versions and feature claims are not re-verified here because the sibling guide owns that comparison.

The two rows worth reading twice are **Doris and StarRocks**. They occupy the same real-time-OLAP niche as ClickHouse and are generally reported to be ahead on join capability and optimiser maturity; ClickHouse's counter-position is scan throughput, compression, and a larger and longer-standing deployment base. That is an architectural trade, not a marketing one, and it is resolved by workload rather than preference.

---

## 10. The Banking and Regulated-Estate Angle

**The workloads that justify it.** ClickHouse earns its place in a regulated institution in four places, and the common property in all four is high-volume append-only data with a latency requirement.

| Workload | Why ClickHouse fits | Regulatory relevance |
|---|---|---|
| **Observability and log analytics** | Very high ingest, cheap retention, fast ad-hoc search over structured logs | Audit-trail retention, incident forensics, evidence production |
| **Trade and transaction analytics** | Append-only event stream, one dominant sort key, interactive drill-down | Best-execution analysis, counterparty exposure, surveillance pre-filtering |
| **Risk aggregation and reporting acceleration** | Materialised views with `AggregatingMergeTree` keep exact rolling aggregates at write time, so a report reads a pre-aggregated table instead of scanning | Intraday risk dashboards, reporting latencies, scenario comparison |
| **Metrics and time-series** | `TimeSeries` engine and PromQL ingest are **private preview** as of 15 Sep 2026; today this means remote write into a schema of your own design | Platform SLI evidence (see `../banking/operational_resilience_framework_guide.md`) |

**Deployment and residency.** Three options exist and the choice is usually made by the residency policy rather than by preference.

| Option | Residency posture | Operational cost | When to choose it |
|---|---|---|---|
| **Self-managed, own infrastructure or private cloud** | Full control; data never leaves the perimeter | Highest — Keeper, merges, parts, upgrades, backups are yours | Hard residency rules, sovereign-cloud mandate, or an air-gapped requirement |
| **ClickHouse Cloud** | Data lives where the service is provisioned; region is the control | Lowest | No residency constraint and no appetite for operating discipline |
| **BYOC / customer-cloud** | Data stays in the institution's own cloud account; vendor operates the control plane | Moderate | The usual compromise where a managed service is acceptable but a third-party data plane is not |

**What an institution should NOT put in ClickHouse**, stated plainly because each is a plausible mistake: the **system of record** (no multi-statement transactions, no rollback, eventual consistency); **mutable entity master data** (constantly updated, read by point lookup — the exact inverse of the engine's strengths); **ACID transactional ledgers and settlement state** (no transaction, no rollback); **point-lookup serving paths** (an 8,192-row granule read per one-row answer); and the **only copy of anything regulated** (replication is not a backup, and a cluster without a tested restore is a single point of failure with extra steps).

**Operational resilience.** `../banking/operational_resilience_framework_guide.md` owns the RTO/RPO doctrine and this guide does not restate it. Applied to a ClickHouse estate the condensed expectations are: two nodes minimum with `ReplicatedMergeTree` plus a three-node Keeper quorum; backups to object storage on a schedule with a **restore test that is actually run**; a measured restore time against the stated RTO; part-count, merge-depth, Keeper-quorum and insert-failure alerting; and a tested upgrade procedure for the pinned LTS. The figure that matters most is the measured restore time, because it is the input to whether the RTO can be met and it is the figure most often assumed rather than measured.

---

## 11. The Cymbal Bank Worked Example

**Everything in this section is fictional and illustrative.** Cymbal Bank is a fictional institution; the volumes, costs, node counts and recovery figures below are invented to demonstrate a design method, not derived from any real deployment or any real institution's data. No number here should be reused as a planning input. Cymbal Bank is the only bank persona in this guide.

### 11.1 The Workload

Cymbal Bank needs a real-time analytics store for **one** workload, chosen because it is the strongest fit and because choosing one forces the design to be concrete: **transaction and card-authorisation analytics with an operational-observability overlay**.

| Characteristic | Illustrative figure | Design consequence |
|---|---|---|
| Event volume | ~2.4 billion events/day (authorisations, transfers, fee postings) | Batched ingest is mandatory; per-row inserts are impossible |
| Peak ingest rate / event size | ~90,000 events/second at the peak hour; ~340 bytes per event | ~0.8 TB/day raw, ~120–180 GB/day compressed; async inserts sized to the peak |
| Retention | 24 months hot-queryable, 7 years archived | TTL tiering to warm storage; archive outside the hot cluster |
| Query concurrency | ~120 concurrent analyst and dashboard queries at peak | Per-query thread budget must be capped, not defaulted |
| Freshness | Under 60 seconds from authorisation to queryable | Async inserts with a busy timeout well under 60 s, or micro-batches |
| Query shape | Time-bucketed aggregates by merchant, channel, BIN range and status; occasional drill-down to one customer's recent events | Time plus one high-cardinality dimension in `ORDER BY`; no joins on the hot path |

### 11.2 Schema and Keys

```sql
CREATE TABLE cymbal.auth_events
(
    event_time      DateTime CODEC(DoubleDelta, LZ4),   -- monotonic, near-regular
    event_date      Date     CODEC(Delta, LZ4),          -- partition expression only
    merchant_id     UInt64   CODEC(T64, LZ4),
    channel         LowCardinality(String),
    bin_range       LowCardinality(String),
    status          LowCardinality(String),
    amount_minor    Int64    CODEC(T64, LZ4),
    currency        FixedString(3),
    customer_id     UInt64   CODEC(T64, LZ4),
    auth_id         UInt64,
    venue_country   LowCardinality(String),
    INDEX idx_merchant merchant_id TYPE bloom_filter GRANULARITY 4
)
ENGINE = ReplicatedMergeTree('/clickhouse/cymbal/auth_events', '{replica}')
PARTITION BY toYYYYMM(event_date)        -- month granularity, cardinality in the hundreds
ORDER BY (merchant_id, event_time)       -- merchant first: the dominant drill-down dimension
TTL event_time + INTERVAL 24 MONTH TO VOLUME 'warm'
SETTINGS index_granularity = 8192;
```

Five decisions in that schema are each the fix for a specific failure in §5 and §8. **`PARTITION BY toYYYYMM`** keeps partition cardinality in the tens, well under the documented guidance of roughly 1,000, so parts remain mergeable within a month. **`ORDER BY (merchant_id, event_time)`** puts the selective dimension first; a time-only sort would force a scan for every merchant query. **Per-column codecs** (`DoubleDelta` on the near-regular timestamp, `T64` on bounded-range IDs) are the difference between a table affordable to keep for two years and one that is not. **`LowCardinality(String)`** on enumerable dimensions is the standard dictionary-encoding answer. **`TTL ... TO VOLUME 'warm'`** implements retention declaratively, with the seven-year obligation handled by a separate archive path.

### 11.3 Topology, Retention and Failure Handling

```
                 producers (micro-batches, ~90k/s peak)
                              │
                  [ load balancer / chproxy ]
                              │
        ┌─────────────────────┼─────────────────────┐
  ┌───────────┐         ┌───────────┐         ┌───────────┐
  │ shard 1   │         │ shard 2   │         │ shard 3   │
  │ repl A+B  │         │ repl A+B  │         │ repl A+B  │
  └───────────┘         └───────────┘         └───────────┘
        └──────── 3-node ClickHouse Keeper quorum ─────────┘
                              │
                  [ object storage — BACKUP target ]
```

- **Sharding:** three shards × two replicas, sharded on `cityHash64(merchant_id)` so a merchant's events land on one shard and merchant-scoped queries stay local. The six-node shape is illustrative, not a recommendation.
- **Replication and coordination:** `ReplicatedMergeTree` with `internal_replication = true`; producers insert directly into the replicated table with client-side balancing, per the documented preference over inserting through a `Distributed` engine; three Keeper nodes on separate hosts with separate log and snapshot volumes and quorum alerting.
- **Retention:** TTL moves parts to a warm volume at 24 months; a separate archive job exports to object storage for the seven-year obligation; the delete rule governs the hot cluster alone.
- **Monitoring:** part count per table, merge queue depth and age, `system.replication_queue` lag per replica, insert failure rate at the producer, per-query memory headroom, and Keeper quorum health. Part count and merge depth are the leading indicators; insert failure is the lagging one.
- **Backup and DR.** Illustrative objectives: **RTO 4 hours, RPO 15 minutes.** The design meets the RPO by replicated parts plus `BACKUP` to object storage every 15 minutes. It meets the RTO only if a measured full-shard restore completes inside four hours, and that figure must be measured rather than assumed. The honest outcome of this exercise is often that the RTO requires a warm standby rather than a restore.

### 11.4 Illustrative Cost Comparison

All figures invented for method demonstration; unit prices are deliberately unspecified because they would be wrong.

| Approach | Illustrative cost driver | Trade |
|---|---|---|
| Self-managed, 6 nodes + 3 Keeper | Compute + NVMe + warm volume + 1.0–1.5 FTE operating time | Cheapest raw compute; highest operational and headcount cost |
| ClickHouse Cloud, equivalent capacity | Consumption-based compute + storage, backups included | Higher direct cost; no Keeper, parts, merges or backups to operate |
| BYOC | Consumption-based control plane + own cloud compute | The middle position; the usual institutional compromise |
| Keeping the incumbent batch warehouse | Existing contract plus a separate streaming pipeline for the freshness gap | Lower change cost; the freshness requirement is not met |

The point is not the numbers — they are fake — it is the shape: **the self-managed option's dominant cost is people; the managed option's dominant cost is consumption.** For one workload with an interactive dashboard and a 60-second freshness requirement, that is usually the deciding comparison.

### 11.5 What This Design Cannot Guarantee

- **No exactly-once end-to-end guarantee.** The pipeline gets idempotent insertion; any transformation happens upstream. A duplicate that differs in content is a duplicate row.
- **No read-your-write across replicas by default** unless it uses replica-aware routing (Enterprise public beta) or reads the writer's replica directly.
- **No row-level correction at low cost.** A mis-attributed authorisation is corrected by an append-only record or a scheduled mutation; a mass correction is a scheduled incident, not a transaction.
- **No uniqueness constraint.** Nothing prevents two rows for the same `auth_id`; the schema relies on producer discipline and insert deduplication.
- **No guarantee the RTO is met, and no protection from a schema mistake.** The four-hour figure holds only if the restore is measured and inside budget, otherwise the design needs a warm standby; and `ORDER BY` plus the partition key are expensive to change, so the choice of `merchant_id` first is defensible and could still be wrong once real query patterns arrive.

---

## 12. The Claims Audit

| # | Claim | Status | Source | Source quality | Date |
|---|---|---|---|---|---|
| 1 | Licence is Apache-2.0; copyright "2016-2026 ClickHouse, Inc."; no BSL, source-available delay or field-of-use restriction | **Verified** | raw.githubusercontent.com/ClickHouse/ClickHouse/master/LICENSE | Primary artefact, verbatim | 15 Sep 2026 |
| 2 | Current LTS is v26.8.5.13-lts, marked Latest; the train is monthly stable plus a slower LTS track | **Verified** | github.com/ClickHouse/ClickHouse/releases (26.8/26.7/26.6 patched the same day; 26.3 LTS still patched) | Primary | 15 Sep 2026 |
| 3 | Technology originated at Yandex 2009; production for Yandex.Metrica 2012; first open-source release 15 June 2016 | **Verified (single secondary source)** | Wikipedia (one clause marked as needing citation) | Secondary; not confirmed at a primary source | 15 Sep 2026 |
| 4 | Incorporated San Francisco September 2021; $50M seed from Index Ventures and Benchmark with Yandex N.V.; Series B $250M at $2B valuation, 28 October 2021 | **Verified (secondary)** | Wikipedia | Secondary | 15 Sep 2026 |
| 5 | Series C: $350M led by Khosla Ventures; total funding >$650M; plus a $100M credit facility | **Verified** | clickhouse.com/blog/clickhouse-raises-350-million-series-c-… | Primary for amounts; ~$6.35B valuation is secondary | 29 May 2025 |
| 6 | **Series D: US$400M closed 16 January 2026, led by Dragoneer Investment Group**; company acquires Langfuse (LLM observability) and introduces a native Postgres service; >3,000 Cloud customers, ARR +250% YoY | **Verified** | clickhouse.com/blog/clickhouse-raises-400-million-series-d-acquires-langfuse-launches-postgres | Primary (vendor announcement) | 16 Jan 2026 |
| 7 | Vector similarity index from 25.8; HNSW the only method; defaults bf16, M 32, ef_construction 128, GRANULARITY 100,000,000 | **Verified** | clickhouse.com/docs/…/annindexes | Primary | docs modified 15 Aug 2026 |
| 8 | Vector search GA in 25.8 | **Version verified; GA date flagged** | Docs say "25.8 and higher"; third-party notes say GA in 25.8 LTS | Primary + secondary | 15 Sep 2026 |
| 9 | On-Demand Compute is in **private preview**, and the cost-based optimiser ships inside it rather than as the default planner | **Verified** | clickhouse.com/blog/on-demand-compute | Primary | 10 Sep 2026 |
| 10 | Replica-aware routing is **public beta, Enterprise only**; stickiness is best-effort, not a guarantee | **Verified** | clickhouse.com/blog/replica-aware-routing-public-beta | Primary | 15 Sep 2026 |
| 11 | PromQL + `TimeSeries` engine are **private preview** on Cloud; AI Functions are **Beta**, also private preview on Cloud, and shipped across 26.4/26.6/26.8 | **Verified** | clickhouse.com/blog/introducing-promql; clickhouse.com/blog/ai-functions-in-clickhouse | Primary | 11 & 15 Sep 2026 |
| 12 | Managed Postgres on Google Cloud is **private preview** | **Verified** | clickhouse.com/blog/postgres-managed-by-clickhouse-gcp-private-preview | Primary | 9 Sep 2026 |
| 13 | ClickHouse is a launch partner for the Data agent in ChatGPT Work | **Verified as an announcement** | clickhouse.com/blog/chatgpt-data-plugin | Primary; the announcement is verified, its capability is not assessed | 10 Sep 2026 |
| 14 | Keeper is RAFT-based and ZooKeeper-protocol-compatible; a mixed cluster is impossible; **ZooKeeper is formally deprecated** | Keeper facts **Verified**; deprecation **Rejected — not supported by any fetched source** | clickhouse.com/docs/…/keeper/clickhouse-keeper | Primary | docs modified 12 Sep 2026 |
| 15 | Primary index is sparse at 8,192-row granules; partitioning does not accelerate queries | **Verified** | clickhouse.com/docs/…/mergetree | Primary | docs modified 11 Sep 2026 |
| 16 | Batch sizes: ≥1,000 rows, ideally 10,000–100,000; async flush defaults 1 MB / 1 s / 100 queries; `wait_for_async_insert=1` default | **Verified** | clickhouse.com/docs/guides/inserting-data; blog "13 mistakes" | Primary + vendor blog | 28 Aug / 20 Feb 2026 |
| 17 | `parts_to_throw_insert` ≈300 and `parts_to_delay_insert` ≈150 | **Flagged — not extracted** | Settings page lists the `parts_to_*` group; defaults not read | Primary page, unread value | docs modified 14 Aug 2026 |
| 18 | Classic mutations rewrite whole parts, share the merge thread pool, apply per-replica and cannot be rolled back; the lightweight patch-part `UPDATE` is claimed "up to 1,000× faster" | Mutations **Verified**; the 1,000× figure is a **vendor claim, not independently verified** | clickhouse.com/blog "13 mistakes" | Vendor blog, consistent with docs | 20 Feb 2026 |
| 19 | Column statistics `basic`/`countmin`/`minmax`/`tdigest`/`uniq`/`uniq_v2` exist as `STATISTICS(...)` | **Verified** | clickhouse.com/docs/…/mergetree | Primary | docs modified 11 Sep 2026 |
| 20 | ClickHouse's own performance-per-dollar benchmarks favour ClickHouse | **Vendor-produced; treat as a claim** | clickhouse.com/blog CostBench and vs-Snowflake posts | Vendor benchmark | 8 & 10 Sep 2026 |

Everything about the licence, the storage engine, the settings and the vendor's own preview/GA labels was verifiable at a primary source and was verified. **The Series D round, which an aggregator asserted and this guide initially flagged, was subsequently confirmed at the vendor's own announcement (16 January 2026) and is recorded as verified.** The genuinely open item is the exact `parts_to_*` defaults. **Version numbers and funding figures are the fastest-decaying claims in this file; the licence and architecture claims are the slowest.**

---

## 13. What Could Not Be Verified

1. **The valuation attached to Series D, and whether a later round has closed.** The round itself — US$400 million, closed 16 January 2026, led by Dragoneer Investment Group — is verified at the vendor's announcement. The **US$15 billion** valuation quoted in press coverage is secondary and was not stated in that announcement, and whether any round has closed since January 2026 was not established. Total funding is therefore stated as "more than US$1 billion" rather than as a precise figure, and any figure quoted here should be re-read from the vendor's site before it is relied on.
2. **The exact default values of `parts_to_throw_insert` and `parts_to_delay_insert`.** The current settings reference lists the `parts_to_*` group but the values were not extracted. The commonly cited 300 and 150 appear in practitioner material and older documentation; they are flagged rather than asserted. Read the live settings page before relying on them.
3. **The legal mechanism of the Yandex spin-out.** The incorporation date (September 2021) and Yandex N.V.'s participation in the seed round are verifiable. Whether the separation was a carve-out, a trademark and repository transfer, or a licence arrangement is not documented in a single primary source I could retrieve.
4. **The official GA announcement date for vector search, and the full scope of the text/inverted index.** The docs say vector similarity indexes are available in 25.8 and higher and third-party release notes describe GA vector search in 25.8 LTS, but the vendor's own GA announcement post was not located — availability is verified, the announcement date is not. Separately, text-index settings exist in the MergeTree settings reference (`text_index_*`, seven settings), indicating a fuller inverted-index capability has shipped, but its precise scope, stability and GA status were not read in full.
5. **Whether ZooKeeper support is formally deprecated.** The documentation presents Keeper as the coordination system and states a mixed cluster is impossible, but no page declares ZooKeeper deprecated or removed. The defensible statement is narrower than "ZooKeeper is deprecated".
6. **The operational status of specific ecosystem projects, and their vendors.** `Altinity/clickhouse-operator`, `Altinity/clickhouse-backup` and `chproxy` were not fetched in this pass. Their existence and role are widely and consistently reported; their current versions, maintainer activity and ClickHouse-version support are unverified here, as is the same for the dbt-clickhouse adapter and the third-party BI connectors. Altinity's and SelectDB's commercial support offerings are likewise named on the basis of public presence rather than a fetched page.
7. **The maturity of the ANN index beyond its documented constraints.** The docs give the implementation (HNSW), the defaults and the constraints, and invite issues rather than labelling the feature experimental. I found no explicit production-readiness statement either way, and recall/latency performance on realistic data is not something the documentation quantifies.
8. **Any specific 2026 roadmap commitment beyond announced launches.** Everything cited in §1.4 is an announcement with a dated blog post. Forward-looking roadmap items not already announced are not cited here, because the repository's public discussions were not read.

---

## 14. Glossary

| Term | Meaning |
|---|---|
| **Part / granule** | A part is an immutable, sorted unit of storage produced by an insert — the thing merges operate on. A granule is a block of 8,192 rows (`index_granularity` default), the finest unit the sparse primary index can address. |
| **Sparse primary index** | One index entry per granule rather than per row; small enough to stay in memory, coarse enough to force granule reads. |
| **Merge / MergeTree family** | The background process combining parts into larger sorted parts, running forever and competing with queries for I/O and CPU; and `MergeTree` plus its behavioural variants (`Replacing`, `Summing`, `Aggregating`, `Collapsing`, `Replicated`, `Shared`). |
| **`ORDER BY` / sorting key, and partition** | The sorting key declares the physical sort order inside every part and, unless `PRIMARY KEY` is set separately, *is* the primary key. A partition is a directory-level data-management split; parts in different partitions never merge. |
| **Data-skipping index** | A per-granule summary (`minmax`, `set`, `bloom_filter`, `ngrambf_v1`, `tokenbf_v1`, `vector_similarity`) used to exclude granules before reading. |
| **Projection / materialised view** | An alternative physical ordering chosen automatically by the planner; or, in ClickHouse, an insert trigger that runs a `SELECT` per inserted block into a target table. |
| **`AggregatingMergeTree` / `-State` / `-Merge`** | Engine plus aggregate-function state variants allowing exact pre-aggregation that stays reconstructible. |
| **Mutation / patch part** | `ALTER TABLE ... UPDATE/DELETE` rewrites parts, shares the merge thread pool and cannot be rolled back; patch parts are the cheap delta form used by lightweight `UPDATE`. |
| **`parts_to_delay_insert` / `parts_to_throw_insert`** | Thresholds at which inserts are first throttled then rejected as part count grows. Defaults flagged (§13). |
| **Async insert** | Server-side buffering of small inserts, flushed on a size, time or query-count threshold. |
| **ClickHouse Keeper** | The RAFT-based coordination service holding replication metadata, the distributed-DDL queue and the dedup log. ZooKeeper-protocol-compatible; mixed clusters impossible. |
| **`Distributed` engine / `ReplicatedMergeTree` / `SharedMergeTree`** | The fan-out layer that owns no data; the replicated engine where replicas ship parts, not statements; and Cloud's compute/storage-separated engine on object storage. |
| **HNSW / `QBit`** | The only approximate-search method in the `vector_similarity` index; and a data type with transposed distance functions allowing precision to be chosen at query time. |
| **AI Functions / CBO** | SQL functions calling an external LLM or embedding provider (Beta, September 2026); and the cost-based optimiser, documented as a component of the private-preview On-Demand Compute feature rather than the default planner. |
| **`system.query_log`** | Per-query log recording timing, bytes, parts read, peak memory and the effective settings — the primary forensics surface. |

---

## 15. Cross-References and Further Reading

**Primary sources** — documentation index <https://clickhouse.com/docs>; MergeTree reference <https://clickhouse.com/docs/reference/engines/table-engines/mergetree-family/mergetree>; MergeTree settings <https://clickhouse.com/docs/reference/settings/merge-tree-settings>; inserting data <https://clickhouse.com/docs/guides/inserting-data>; vector search and ANN indexes <https://clickhouse.com/docs/reference/engines/table-engines/mergetree-family/annindexes>; AI Functions <https://clickhouse.com/docs/reference/functions/regular-functions/ai-functions>; ClickHouse Keeper <https://clickhouse.com/docs/guides/sre/keeper/clickhouse-keeper>; releases <https://github.com/ClickHouse/ClickHouse/releases>; licence <https://raw.githubusercontent.com/ClickHouse/ClickHouse/master/LICENSE>.

**Vendor blog posts cited** (all checked 15 September 2026) — Series C, 29 May 2025 <https://clickhouse.com/blog/clickhouse-raises-350-million-series-c-to-power-analytics-for-ai-era>; Series D, Langfuse acquisition and the native Postgres service, 16 January 2026 <https://clickhouse.com/blog/clickhouse-raises-400-million-series-d-acquires-langfuse-launches-postgres>; "13 mistakes and how to avoid them", 20 February 2026 <https://clickhouse.com/blog/common-getting-started-issues-with-clickhouse>; On-Demand Compute, 10 September 2026 <https://clickhouse.com/blog/on-demand-compute>; replica-aware routing public beta, 15 September 2026 <https://clickhouse.com/blog/replica-aware-routing-public-beta>; TimeSeries engine and PromQL, 15 September 2026 <https://clickhouse.com/blog/introducing-promql>; AI Functions, 11 September 2026 <https://clickhouse.com/blog/ai-functions-in-clickhouse>; WalShadow, 10 September 2026 <https://clickhouse.com/blog/introducing-walshadow>; ChatGPT Work Data agent, 10 September 2026 <https://clickhouse.com/blog/chatgpt-data-plugin>; Managed Postgres on GCP, 9 September 2026 <https://clickhouse.com/blog/postgres-managed-by-clickhouse-gcp-private-preview>.

**Sibling guides in this repository**

- [MPP Databases — The Landscape](data/mpp_databases_guide.md) — §4.5 is the ClickHouse profile, §5 the peer comparison table; **the peer-by-peer analysis belongs there**
- [Low-Latency GenAI Patterns](low_latency_genai_patterns_guide.md) — TTFT/TPOT and serving latency
- [Vector Databases](ai_llm/rag/vector_databases_guide.md) and the `ai_llm/rag/` cluster — retrieval-store comparison and RAG patterns
- [Enterprise Data Platforms](data/enterprise_data_platforms_guide.md) and [Advanced Analytics Solutions](advanced_analytics_solutions_guide.md) — data-platform and lakehouse architecture
- [Apache Flink — The Stream's State](apache_flink_guide.md) — stream processing, checkpoints, end-to-end consistency
- [Kafka](../banking/kafka_guide.md) and [Cloud Object Storage and the Lakehouse](cloud_object_storage_lakehouse_guide.md) — the event backbone that usually feeds a ClickHouse cluster, and Iceberg/object-storage table formats
- [Operational Resilience Framework](../banking/operational_resilience_framework_guide.md) — RTO/RPO doctrine, DR design

---

## 16. Closing Summary

**ClickHouse is a merge tree with a SQL front end, and almost every design decision either follows from that or compensates for it.** The immutable-part model, the sparse index, the permanent background merging and the weakness at row-level mutation are one fact seen from four angles; a designer who internalises the storage model infers most of the rest correctly.

**Its strongest real-world position is append-only analytical volume at high concurrency, which in practice means observability and transaction analytics rather than warehousing.** The system it displaces most often is a search engine in log analytics, not a cloud warehouse, and the 2026 releases — a metrics engine, an observability stack, a Postgres replication product — say the same thing.

**The licence is the most commercially important fact about the project, and it is unambiguous.** Apache-2.0, unmodified, no source-available delay, no field-of-use restriction, verified from the LICENSE file on 15 September 2026; the exit cost of leaving the vendor is low, which is worth more to an institution than any feature comparison. **The 2026 launch announcements should be read as announcements, not capabilities.** Between 7 and 15 September 2026 the vendor announced eleven things; only one was even a public beta and it was gated to Enterprise. On-Demand Compute, Managed Postgres, the TimeSeries/PromQL engine and AI Functions are private preview or beta.

**The cost-based optimiser exists and is not yet the default planner.** That is the honest sentence and it corrects the most common overclaim about ClickHouse's joins; the column-statistics feature shows the optimiser work is real, and its shipping inside a private-preview feature shows it is not yet general. **Vector search is genuinely shipped, GA since 25.8, and narrower than the marketing implies** — one method (HNSW), three distance functions, an index that must be materialised as a mutation, and defaults the vendor advises against tuning. It is a strong retrieval store for filtering data you already hold and a weaker one when similarity search is the product.

**"Too many parts" is the defining operational failure, and it is nearly always self-inflicted.** High-cardinality partition keys, row-by-row inserts, view sprawl and mutations produce it, and the producer's own retry behaviour makes it worse. Batching to 10,000–100,000 rows and month-level partitioning prevent almost all of it. **The hardest cost to budget for is skill, not compute** — schema design around `ORDER BY`, part and merge monitoring, memory tuning and Keeper operation are specific disciplines, uncommon in the market, and the shortage is structural. An institution that cannot staff them should buy ClickHouse Cloud or a supported distribution and treat the premium as the real price of the service.

**For a regulated institution the fit is real but bounded.** Log and audit retention, transaction and trade analytics, and risk-aggregation acceleration are genuine matches. The system of record, mutable master data, transactional ledgers and point-lookup serving paths are not, and putting them there is a mistake that surfaces years later when correction cost and recovery time collide.

**And the thing to remember about any engine whose excellence is concentrated in one dimension** is that the merge model makes the reads fast and the writes cheap, and it is also what makes the updates expensive and the parts a permanent operational surface. The speed and the fragility are the same property, and an architect who wants one must accept the other — the merge tree.
