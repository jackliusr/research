# Apache Paimon vs Apache Iceberg vs Delta Lake: Comprehensive Three-Way Comparison (2026)

> **Last updated:** July 2026
>
> A thorough, research-backed comparison of the three leading open table formats for lakehouse architectures — Apache Paimon, Apache Iceberg, and Delta Lake. Covers architecture, features, ecosystem, performance, and decision guidance for streaming, batch, and hybrid workloads.

---

## Table of Contents

1. [Overview: The Three Table Formats](#1-overview-the-three-table-formats)
2. [Apache Paimon Deep Dive](#2-apache-paimon-deep-dive)
3. [Apache Iceberg Deep Dive](#3-apache-iceberg-deep-dive)
4. [Delta Lake Deep Dive](#4-delta-lake-deep-dive)
5. [Three-Way Comparison Table](#5-three-way-comparison-table)
6. [Architecture Deep Dive: The Key Differentiator](#6-architecture-deep-dive-the-key-differentiator)
7. [Ecosystem Support Comparison](#7-ecosystem-support-comparison)
8. [When to Choose Which](#8-when-to-choose-which)
9. [Performance Considerations](#9-performance-considerations)
10. [Future Directions](#10-future-directions)
11. [Data Sources & Further Reading](#11-data-sources--further-reading)

---

## 1. Overview: The Three Table Formats

### What Are Table Formats?

A table format is a metadata layer that sits on top of data files (Parquet, Avro, ORC) stored in distributed object storage (S3, ADLS, GCS, HDFS) and brings **relational database-like features** to data lakes. Before table formats, data lakes were loose collections of files that lacked transactional guarantees, schema enforcement, and efficient pruning for analytics.

Table formats solve four critical problems that raw file storage cannot:

| Capability | What It Does | Why It Matters |
|------------|-------------|----------------|
| **ACID transactions** | Atomic, consistent, isolated, durable multi-operation commits | No dirty reads, no partial writes, safe concurrent access |
| **Schema evolution** | Add, drop, rename, reorder columns without rewriting data | Schema-on-write without the pain of full rewrites |
| **Time travel** | Query data as-of any previous snapshot or timestamp | Audits, rollbacks, reprocessing, debugging |
| **Metadata management** | Column-level stats, partition pruning, file skipping | Queries read only relevant files — orders of magnitude faster |

### Why Three Formats? Different Origins, Different Philosophies

There is no single "best" table format because the three major formats were built for different problems by different organizations. Understanding their origins explains their design choices:

| Format | Creator | Open-Sourced | Governance | Core Philosophy |
|--------|---------|-------------|------------|-----------------|
| **Delta Lake** | Databricks | 2019 | Linux Foundation (2022) | Databricks-optimized, Spark-centric lakehouse |
| **Apache Iceberg** | Netflix | 2018 (to Apache) | Apache Software Foundation | Engine-agnostic, multi-platform, batch-first |
| **Apache Paimon** | Alibaba | 2022 (to Apache) | Apache Software Foundation | Streaming-first, LSM-tree, Flink-native |

**Delta Lake** was born from Databricks' internal "Delta Engine" and open-sourced in 2019. It moved to the Linux Foundation in 2022 to signal broader governance, but Databricks remains the primary driver. Delta is deeply integrated with the Databricks platform (Unity Catalog, Spark, MLflow) and optimized for Spark-centric workloads. Its architecture is the simplest of the three — a flat JSON transaction log — making it the easiest to understand and debug.

**Apache Iceberg** was created at Netflix by Ryan Blue and Dan Weeks to overcome Hive table format limitations at petabyte scale. It was contributed to the Apache Software Foundation in 2018 and graduated as a top-level project in 2020. Iceberg is widely regarded as the most vendor-neutral format, with contributions from Netflix, Apple, Airbnb, Dremio, Snowflake, Google, and AWS. Its tree-structured metadata enables powerful query planning optimizations. As of mid-2026, Iceberg has the broadest native ecosystem support across engines and clouds.

**Apache Paimon** originated as "Flink Table Store" at Alibaba and was donated to the Apache Software Foundation in 2022, graduating as a top-level project in April 2024. It is fundamentally different from Iceberg and Delta: built on an LSM-Tree (Log-Structured Merge-Tree) architecture, Paimon optimizes for **streaming ingestion** rather than batch writes. This makes it dramatically faster for high-frequency, small-batch writes from CDC pipelines and streaming sources. Paimon is the deepest Flink integration of any table format and is purpose-built for the streaming data lakehouse use case.

### The Big Picture (Mid-2026)

As of mid-2026, the table format landscape has converged in several important ways:

- **Iceberg v3 spec** is ratified (mid-2025) — adds deletion vectors, VARIANT type, geospatial, row lineage, and encryption. Engine support is rolling out across the ecosystem.
- **Delta Lake 4.0.x** is current — adds Liquid Clustering (GA), Coordinated Commits, type widening, UniForm GA for Iceberg compatibility, and Rust-based Delta Kernel.
- **Apache Paimon 1.0+** is production-ready — validated at Alibaba, ByteDance, TikTok, and Xiaomi handling petabytes and millions of rows/second of streaming writes.
- **Interoperability** is now first-class: Delta UniForm generates Iceberg metadata; Apache XTable (Incubating) provides omni-directional conversion; Iceberg REST catalog enables true multi-platform access.
- **Convergence**: Iceberg v3 borrows deletion vectors from Delta; Delta UniForm adopts Iceberg metadata; Paimon adds Iceberg compatibility features. The formats are growing closer, but their architectural foundations remain distinct.

---

## 2. Apache Paimon Deep Dive

### Origins & Governance

- **Created at Alibaba** as "Flink Table Store" (FTS) — a storage layer purpose-built for Apache Flink streaming pipelines.
- **Donated to the Apache Software Foundation** in March 2022 and renamed to Apache Paimon.
- **Graduated as an Apache Top-Level Project** in April 2024 — a rapid ascent reflecting strong community and production maturity.
- **Primary contributor**: Alibaba (Jingsong Li, PMC Chair, leads the team). Growing contributions from ByteDance, Xiaomi, and other Chinese tech giants.
- **Philosophy**: "Real-time lakehouse" — unify streaming and batch on a single storage format, with streaming as the first-class citizen.
- **Adoption signals**: Production deployments at Alibaba, ByteDance, TikTok, and Xiaomi handling petabytes of data. Strong adoption in China, expanding globally. Paimon 1.0+ releases demonstrate production readiness.

### Core Architecture: LSM-Tree

Paimon's architecture is its key differentiator. Unlike Iceberg and Delta (which use copy-on-write), Paimon uses a **Log-Structured Merge-Tree (LSM-Tree)**:

```
Paimon Table
  └── Buckets (each bucket = independent LSM tree)
       ├── Base files (sorted Parquet files, compacted)
       │    ├── Level 0: Freshly flushed data (small, sorted runs)
       │    ├── Level 1: Partially compacted
       │    └── Level N: Fully compacted (large, columnar Parquet)
       └── Changelog files (row-level log for streaming reads)
```

**How writes work:**
1. Streaming data arrives in an in-memory buffer per bucket.
2. Buffer is flushed to a sorted Parquet "run" on object storage (Level 0).
3. Background compaction threads merge Level-0 runs into larger Level-1+ files (similar to RocksDB's compaction).
4. Compaction is continuous and automatic — no separate maintenance job required.

**How reads work:**
- **Batch reads**: Query only the compacted base files (Level N) — columnar Parquet, efficient for analytics.
- **Streaming reads**: Consume from the changelog files — row-level, low-latency, ideal for streaming consumers.
- **Merge-on-read**: Files at multiple levels are merged at query time using the primary key.

**Why this matters:** The LSM approach means Paimon can absorb millions of small writes per second without the per-microbatch file rewrite overhead that plagues Iceberg and Delta in streaming scenarios.

### Key Features

| Feature | Paimon Implementation |
|---------|----------------------|
| **Streaming ingestion** | ✅ Native — LSM-tree absorbs high-frequency writes without per-batch file rewrites |
| **Streaming read** | ✅ Native — changelog files enable real-time row-level consumption |
| **Batch read** | ✅ Columnar Parquet from compacted files — good for analytics |
| **ACID transactions** | ✅ Serializable isolation via two-phase commit protocol |
| **Time travel** | ✅ Snapshot-based — each snapshot is an immutable LSM tree state |
| **Schema evolution** | ✅ Add/drop/rename/reorder columns |
| **Partition evolution** | ✅ Can change partition strategy without rewriting data |
| **Primary key upserts** | ✅ Native — LSM naturally handles upserts via merge-on-read |
| **Row-level deletes** | ✅ LSM deletion vectors — compact bitmaps for logical row deletion |
| **Auto compaction** | ✅ Background, continuous, tier-based (Level 0 → Level N) |
| **Partition pruning** | ✅ Bucket + partition metadata for query optimization |

### Flink Integration: The Deepest of Any Table Format

Paimon's roots in Flink Table Store give it the deepest Flink integration of any table format:

- **Flink CDC sink**: Native Flink CDC connector — consume Debezium/Kafka CDC streams directly into Paimon with primary key deduplication.
- **Flink streaming sink**: Write continuous streaming data with sub-second commit latency.
- **Flink streaming source**: Read Paimon changelog files as a streaming source — Flink consumers see changes in real time.
- **Flink SQL integration**: Full Flink SQL support — `CREATE TABLE`, `INSERT`, `SELECT`, `MERGE` with Paimon as storage.
- **Flink savepoint/checkpoint compatibility**: Paimon integrates with Flink's checkpoint mechanism for exactly-once semantics.

### Storage: Columnar + Row-Level

Paimon stores data in two complementary formats:

| Storage Type | Format | Purpose | Access Pattern |
|-------------|--------|---------|---------------|
| **Base files** | Columnar Parquet (sorted, compacted) | Analytics, batch queries, time travel | Columnar scans, predicate pushdown |
| **Changelog files** | Row-level (Avro or JSON) | Streaming reads, incremental consumption | Row-level, low-latency sequential reads |

This dual-storage approach is unique — Iceberg and Delta store only columnar data and rely on the engine for streaming reads.

### Current Status

- **Latest release**: Apache Paimon 1.0.x (2025) and 1.2.x (2026) — production-grade.
- **Maturity**: Young compared to Iceberg (1.x, v3 spec) and Delta (4.x), but rapidly maturing with large-scale production validation.
- **Community**: Growing — GitHub stars increasing, global adoption expanding beyond China.
- **Interoperability**: Adding Iceberg compatibility (V3 deletion vectors compatibility) for gradual migration paths.
- **Best for**: Streaming data lakehouse, Flink-native pipelines, real-time analytics on streaming data, CDC ingestion at scale.

---

## 3. Apache Iceberg Deep Dive

### Origins & Governance

- **Created at Netflix** by Ryan Blue and Dan Weeks to address the pain points of Hive table format at petabyte scale.
- **Contributed to Apache Software Foundation** in 2018 and graduated as a top-level project in 2020.
- **Vendor-neutral governance** under the Apache Foundation's meritocratic model. No single company controls direction.
- **Strong community base**: Netflix, Apple, Airbnb, Expedia, Stripe, Dremio, Snowflake, Google, and AWS all contribute significantly.
- **Snowflake** ships a native Iceberg catalog (first-class Iceberg tables).
- **AWS** has made Iceberg the default recommendation for Athena, Glue, and EMR.
- **Google** contributed significantly to the Iceberg v3 spec and published v3 test datasets.
- **Adoption signals**: Apple uses Iceberg "wall-to-wall" across its data infrastructure. Netflix manages exabytes. Adobe, Stripe, and Expedia have large-scale deployments.

### Architecture: Tree of Metadata Files

Iceberg uses a tree of metadata files for atomicity and isolation:

```
Table Root (Catalog pointer)
  └── Metadata File (version-N.metadata.json)
       ├── Current schema, partition spec, sort order
       ├── Snapshot references (snapshot-id → manifest list)
       └── Manifest List (snapshot-N.avro)
            ├── Manifest File (partition-data.avro)
            │    └── Data File (Parquet/Avro/ORC)
            │         └── Column stats (min/max, null count)
            ├── Manifest File (partition-data.avro)
            │    └── Data File + Delete File (v2 positional deletes)
            └── Manifest File (v3: Deletion Vectors in Puffin files)
```

**Metadata file**: The root — each commit creates a new metadata file via atomic catalog swap. Contains the schema, partition spec, sort order, and pointers to all snapshots.

**Manifest list**: Avro file listing all manifest files in a snapshot, with partition range info for pruning.

**Manifest files**: Avro/Parquet files listing data files per partition. Each entry includes column-level statistics (min/max values) for predicate pushdown.

**Data files**: Parquet (default), Avro, or ORC — Iceberg is the most flexible in file format support.

**Delete files (v2)**: Position-based delete files for row-level deletes (merge-on-read).

**Deletion Vectors (v3)**: Roaring Bitmap-based deletion vectors stored in Puffin files — more compact than positional delete files.

### Catalogs

Iceberg supports the widest range of catalog implementations:

| Catalog | Description |
|---------|-------------|
| **Hive Metastore** | Legacy, widely supported across engines |
| **AWS Glue Catalog** | Native AWS integration — default for Athena, Glue, EMR |
| **Nessie** | Git-like catalog with branching, merging, tagging |
| **JDBC** | Relational database-backed catalog (PostgreSQL, MySQL) |
| **REST Catalog** | **Key differentiator** — standard REST API enables cross-engine, cross-platform access from any client with HTTP |
| **In-memory** | For testing/development only |

The **REST catalog** is Iceberg's most significant architectural advantage over Delta Lake. It provides a standardized API that any engine (Spark, Flink, Trino, Snowflake, Dremio) can use to read and write the same table. This makes true multi-engine, multi-platform, cross-cloud lakehouse architectures practical.

### Key Features

- **Full ACID transactions** with serializable isolation via optimistic concurrency control.
- **Time travel and rollback**: Query by snapshot ID or timestamp. Each snapshot is immutable.
- **Schema evolution**: Add, drop, rename, reorder, update column types (safe promotions int→long). No data rewriting.
- **Hidden partitioning**: Partition values computed automatically from partition spec — users never reference partition columns in queries. This is a major usability advantage over Delta and Paimon.
- **Partition evolution**: Change partition spec without rewriting existing data. Old and new specs co-exist.
- **Compaction**: `rewrite_data_files` (merge small files) and `rewrite_manifests` (compact metadata tree). Powerful but requires explicit scheduling.
- **File skipping**: Column-level min/max statistics in manifests for efficient predicate pushdown and partition pruning.
- **Row-level deletes**: Position-based delete files (v2) and Deletion Vectors (v3) for MERGE, UPDATE, DELETE.
- **Multi-engine**: Spark, Flink, Trino, Presto, Hive, Dremio, Snowflake (native), BigQuery (native), Athena (native), StarRocks, Impala.
- **REST catalog**: Standardized API for multi-engine, multi-platform access.

### Iceberg v3 (Spec Ratified Mid-2025)

Iceberg v3 is the most significant evolution since v2. Key additions:

| Feature | Description |
|---------|-------------|
| **Deletion Vectors** | Roaring Bitmap structures replacing positional delete files — reduces read amplification and metadata overhead for frequent updates. Converges with Delta Lake's approach. |
| **VARIANT type** | Native semi-structured data support (JSON, schemaless records) without string storage overhead. Similar to Snowflake's VARIANT. |
| **Geospatial types** | Native geometry types and spatial indexing for location-based queries. |
| **Row lineage** | Track operation and source provenance for each row — fine-grained CDC audit trail. |
| **Type widening** | Safe, automatic column type widening (int→long→float→double). |
| **Default column values** | Specify default values for omitted INSERT columns. |
| **Table encryption** | Native encryption at rest specification. |

> **Status (mid-2026)**: v3 spec finalized and ratified mid-2025. Databricks Unity Catalog supports Iceberg v3 natively. AWS has rolled out v3 support across EMR, Glue, Athena, and S3 Tables. Spark, Trino, and Flink have v3 support in various stages of implementation. Latest library: **Iceberg 1.11.0** (May 2026).

### Current Status

- **Latest release**: Iceberg 1.11.0 (May 2026) — supports v2 and v3 format versions.
- **Maturity**: Most mature of the three — years of production use at the largest scale.
- **Community**: Largest and most diverse contributor base across all three formats.
- **Best for**: Multi-engine environments, cloud-agnostic lakehouse, ad-hoc analytics, organizations prioritizing vendor neutrality.

---

## 4. Delta Lake Deep Dive

### Origins & Governance

- **Created by Databricks** as the storage layer for their lakehouse platform, tracing back to internal "Delta Engine" technology.
- **Open-sourced April 2019** under Apache 2.0 license — immediately became the most widely adopted table format by GitHub stars.
- **Moved to Linux Foundation in 2022** — signaling broader governance, though Databricks drives the vast majority of development.
- **Community**: Largest by GitHub stars and downloads (20M+ monthly downloads). Strongest within Databricks ecosystem (Spark, MLflow, Unity Catalog). Growing outside via connectors for Trino, Flink, Snowflake, but the open-source community outside Databricks is smaller than Iceberg's.
- **Delta Kernel project**: Rust-based engine for non-JVM Delta reads — part of the push for broader ecosystem adoption.

### Architecture: The Transaction Log

Delta Lake uses the simplest architecture of the three — a single, flat transaction log:

```
Delta Table Root
  └── _delta_log/
       ├── 00000000000000000001.json  (Commit 1: initial write)
       ├── 00000000000000000002.json  (Commit 2: append)
       ├── 00000000000000000003.json  (Commit 3: delete + add)
       ├── ...
       └── _last_checkpoint          (pointer to latest checkpoint file)
  └── Checkpoint Files (periodic Parquet snapshots)
       ├── 00000000000000000010.checkpoint.parquet
       └── 00000000000000000020.checkpoint.parquet
  └── Data Files (*.parquet)
  └── Deletion Vector Files (*.dv — binary bitmaps)
```

**Transaction log (delta log)**: A directory of ordered JSON files, each representing one atomic commit. Each commit records:
- Which data files were added (with column stats, row counts, file sizes)
- Which data files were logically removed
- Schema changes, partition changes, protocol version
- Application metadata (user, timestamp, operation, transaction IDs)

**Checkpoint files**: Periodic Parquet snapshots of the full log state. Dramatically speeds up table load — readers load the latest checkpoint then apply only trailing JSON commits.

**Data files**: Parquet is the only native file format. Delta 4.0 adds the VARIANT type for semi-structured data, stored within Parquet.

**Deletion Vectors** (v2.3+): Compact binary bitmaps (.dv files) marking logically deleted rows within a Parquet file. Enables row-level deletes without rewriting the entire file — merge-on-read approach.

### Catalogs

| Catalog | Description |
|---------|-------------|
| **Hive Metastore** | Legacy, widely supported |
| **Unity Catalog** | Databricks' own — rich governance, lineage, RBAC, AI integration |
| **AWS Glue Catalog** | Supported via connector |
| **Filesystem** | Path-based tables — no external catalog needed |
| **REST Catalog** | ❌ No equivalent to Iceberg's REST catalog — Delta uses Unity Catalog |

The lack of a lightweight REST catalog standard is Delta's most notable architectural gap compared to Iceberg.

### Key Features

- **Full ACID transactions** with serializable isolation via optimistic concurrency control.
- **Time travel and rollback**: Query by version (`VERSION AS OF`) or timestamp (`TIMESTAMP AS OF`).
- **Schema evolution**: Add, drop, rename, update column types. Type widening in 4.0.
- **Schema enforcement**: Strict — rejects writes with mismatched schemas.
- **Generated columns**: Auto-computed column values for partition key pre-computation.
- **Deletion Vectors** (v2.3+): Row-level MERGE, UPDATE, DELETE without rewriting entire Parquet files.
- **Liquid Clustering** (v3.0+, GA in 4.0): Continuous, incremental data layout optimization without full rewrites. Adapts clustering strategy based on query patterns — more flexible than Z-Order. Clustering columns can be changed over time without rewriting.
- **Change Data Feed (CDC)**: Built-in row-level change tracking exposed as a streaming source — enable once, and every write produces a queryable change log.
- **Constraints**: NOT NULL and CHECK constraints for data quality.
- **Delta Sharing**: Open protocol for cross-organization data sharing.
- **VARIANT type** (4.0): Native semi-structured data support (JSON, schemaless).
- **Coordinated Commits** (4.0): Atomic multi-table commits across Unity Catalog for cross-table consistency.

### Delta Lake 4.0.x (Current, Mid-2026)

Delta Lake 4.0.0 was released alongside Apache Spark 4.0.0 in April 2025. Key improvements:

| Feature | Description |
|---------|-------------|
| **Liquid Clustering (GA)** | Incremental, adaptive data layout optimization. No more full-table OPTIMIZE rewrites. |
| **UniForm (GA)** | Delta tables auto-generate Iceberg metadata — Iceberg-native engines (Trino, Snowflake, Athena) can read Delta tables. Write-Delta, read-Iceberg model. |
| **Delta Kernel (Rust)** | Native non-JVM engine for reading Delta tables without Spark. Enables lightweight access from Python, Rust, and embedded systems. |
| **Type widening** | Automatic numeric type widening (INT→LONG→DECIMAL→FLOAT). |
| **Coordinated commits** | Atomic commits across multiple Delta tables via Unity Catalog. |
| **VARIANT type** | Native semi-structured data support within Parquet files. |
| **Spark Connect support** | Remote Spark execution via gRPC. |
| **Improved Deletion Vectors** | Better read performance for heavy merge/update workloads. |

### UniForm (Universal Format)

Delta Lake 3.0+ introduced **UniForm**, which writes Iceberg-compatible metadata alongside the Delta transaction log:

```
Delta Table with UniForm Enabled
  ├── _delta_log/             (Delta metadata — always authoritative)
  ├── metadata/               (Iceberg metadata files)
  │   ├── 00001-a1b2.metadata.json
  │   ├── snap-12345-1-manifest-list.avro
  │   └── snap-12345-2-manifest.avro
  └── *.parquet               (Shared data files — read by both formats)
```

**How it works**: Every Delta write also generates the Iceberg metadata tree pointing to the same Parquet data files. Iceberg-native engines see a valid Iceberg table.

**Caveats**:
- **Read-only from Iceberg engines**: Iceberg engines can read but cannot write UniForm tables.
- **Latency**: Iceberg metadata is generated after the Delta commit — brief window of divergence.
- **Feature gaps**: Some Delta-specific features (Liquid Clustering, CDF) lack Iceberg equivalents in UniForm.

### Current Status

- **Latest release**: Delta Lake 4.0.1 (current, mid-2026).
- **Community**: Largest by GitHub stars and monthly downloads (20M+). Strongest in Spark/Databricks ecosystem.
- **Maturity**: Very mature — years of production use at massive scale (Databricks customers, Azure Databricks).
- **Best for**: Databricks-native lakehouse, Spark-intensive workloads, organizations already invested in the Databricks platform.

---

## 5. Three-Way Comparison Table

| Feature | Apache Paimon | Apache Iceberg | Delta Lake |
|---------|--------------|----------------|------------|
| **Governance** | Apache Foundation | Apache Foundation | Linux Foundation (Databricks-led) |
| **Original Creator** | Alibaba | Netflix | Databricks |
| **Core Architecture** | LSM-Tree (merge-on-read) | Copy-on-write + merge-on-read (v2/v3) | Copy-on-write + Deletion Vectors |
| **Streaming Ingestion** | ✅ **Native (best)** — LSM absorbs high-frequency writes | ⚠️ Requires compaction tuning for streaming | ⚠️ Requires optimization for high-frequency writes |
| **Streaming Read** | ✅ Native via changelog files | ✅ Incremental read API (v2+) | ✅ Change Data Feed |
| **Batch Query Perf** | Good (after compaction) | ✅ Excellent (native columnar) | ✅ Excellent (native columnar) |
| **Hidden Partitioning** | ❌ Manual — users must specify | ✅ **Automatic** — transparent to users | ❌ Manual — users must know partition scheme |
| **Partition Evolution** | ✅ Yes — change without rewrite | ✅ Yes — change without rewrite | ❌ Requires full REPARTITION (rewrites data) |
| **Row-level Deletes** | ✅ LSM deletion vectors (compact bitmaps) | ✅ Position delete files (v2) / Deletion Vectors (v3) | ✅ Deletion Vectors (binary bitmaps, v2.3+) |
| **Merge (Upsert)** | ✅ **LSM-native** — merge-on-read with primary key | ✅ MERGE with position deletes / Deletion Vectors | ✅ MERGE with write-audit-publish |
| **Clustering / Ordering** | ✅ Auto compaction (continuous, tier-based) | ✅ `rewrite_data_files` (explicit, scheduled) | ✅ Liquid Clustering (incremental, adaptive, GA) |
| **File Formats** | Parquet + Changelog (Avro/JSON) | Parquet, Avro, ORC | Parquet only (UniForm → Iceberg compat) |
| **Flink Integration** | ✅ **Deepest (native)** — CDC, streaming sink, streaming source | ✅ Native connector (good) | ⚠️ Connector (basic, bucketed writes) |
| **Spark Integration** | ✅ Good — Spark SQL, DataFrame | ✅ Native — Spark connector | ✅ **Deepest (native)** — Databricks-optimized |
| **Trino / Presto** | ✅ Connector | ✅ **Native** — Trino Iceberg connector | ✅ Connector (delta-lake) |
| **Snowflake** | ❌ Limited | ✅ **Native catalog** — first-class Iceberg tables | ✅ Read connector / UniForm |
| **Google BigQuery** | ❌ Not supported | ✅ **Native Iceberg tables** — managed | ⚠️ Read via Delta connector |
| **AWS Athena** | ❌ Limited | ✅ **Native** — recommended format on AWS | ⚠️ Limited SELECT via connector |
| **AWS Glue** | ⚠️ Limited | ✅ **Native** — default recommendation | ✅ Catalog + ETL via Spark |
| **Databricks** | ❌ Not supported | ✅ UniForm read / native v3 in Unity Catalog | ✅ **Native** — first-class, most optimized |
| **Dremio** | ❌ Not supported | ✅ **Native** Iceberg support | ⚠️ Limited via connector |
| **StarRocks** | ✅ Connector | ✅ **Native** Iceberg catalog | ❌ Not supported |
| **REST Catalog** | ✅ (newer, less mature) | ✅ **Mature** — key differentiator for multi-engine | ❌ No equivalent — uses Unity Catalog |
| **UniForm (Interop)** | ❌ N/A (Paimon is its own format) | N/A (Iceberg is the interop target) | ✅ Writes Iceberg metadata — read-only for others |
| **CDC Output** | ✅ Changelog files (native) | ⚠️ Incremental read API (less user-friendly) | ✅ Change Data Feed (built-in, streamable) |
| **Schema Enforcement** | ✅ Strict | ✅ Strict | ✅ Strict |
| **Auto Compaction** | ✅ **Continuous background** — tiered LSM compaction | ❌ Manual — requires `rewrite_data_files` | ⚠️ Semi-auto — Liquid Clustering helps, OPTIMIZE needed |
| **Delta Sharing** | ❌ Not supported | ❌ Not supported | ✅ Native (open protocol) |
| **Open Source License** | Apache 2.0 | Apache 2.0 | Apache 2.0 |
| **Community Size** | Growing (strong in Asia, expanding globally) | Very Large (diverse, global, multi-vendor) | Very Large (GitHub stars leader, Databricks-driven) |
| **Maturity** | Young (1.x) — rapidly maturing | Mature (1.x, v3 spec) — most production-tested | Mature (4.x) — most widely deployed by downloads |
| **Learning Curve** | Medium (LSM concepts, bucket design) | Medium-High (manifest tree, catalog setup) | Low-Medium (JSON log is intuitive) |

---

## 6. Architecture Deep Dive: The Key Differentiator

The single most important difference between these three formats is their **write architecture**. This determines everything about performance, operational complexity, and workload suitability.

### Paimon's LSM Approach

```
[Streaming Data] → [In-Memory Buffer] → [Flush to Sorted Run (L0 Parquet)]
                                              ↓
                                    [Background Compaction]
                                              ↓
                              [L1 → L2 → ... → LN (Large Parquet)]
                                              ↓
                              [Batch reads: compacted L-N files]
                              [Streaming reads: changelog files]
```

- **Write path**: Data lands in an in-memory buffer per bucket. When the buffer is full (or a commit triggers), it's flushed as a sorted Parquet run to S3.
- **No per-microbatch file rewrite**: Unlike Iceberg/Delta, each micro-batch does not create new metadata files or rewrite existing data. The LSM absorbs tiny writes efficiently.
- **Compaction**: Runs continuously in the background — merging small Level-0 runs into larger Level-1+ files. This is automatic and tiered (similar to RocksDB/LSM-Tree in databases).
- **Result**: **10-100x faster streaming ingestion** compared to Iceberg/Delta for high-frequency writes. Paimon handles millions of primary-key updates per second without the write amplification that plagues copy-on-write formats.
- **Trade-off**: Batch read performance requires compaction to reach the top level. Heavy write workloads may create read amplification if compaction lags behind.

### Iceberg's Tree Approach

```
[Write] → [New Data Files] + [New Manifest File] + [New Manifest List] + [New Metadata File]
                                                                                    ↓
                                                                     [Atomic catalog swap]
                                                                                    ↓
                                                                           [New Snapshot]
```

- **Write path**: Each write creates new data files (Parquet/Avro/ORC), a new manifest file listing them, a new manifest list, and a new metadata file. The catalog pointer is atomically swapped to the new metadata file.
- **Metadata tree**: Five-layer structure (Catalog → Metadata File → Manifest List → Manifest File → Data File) enables powerful pruning — queries can skip entire partitions at the manifest list level.
- **Compaction**: Requires explicit `rewrite_data_files` procedure — scheduled as a separate maintenance job. For streaming workloads, small file accumulation requires ongoing compaction management.
- **Result**: Excellent query planning (manifest pruning can skip 99%+ of files for selective queries), but write overhead per microbatch is high. Streaming ingestion generates many small files that must be compacted.
- **Trade-off**: Unmatched for ad-hoc analytics on large datasets. Requires operational investment for streaming ingestion compaction.

### Delta's Log Approach

```
[Write] → [New Data Files] + [New JSON Entry in _delta_log]
                                                          ↓
                                          [Atomic append to log]
                                                          ↓
                                               [New Version]
```

- **Write path**: Each write creates new Parquet data files and appends a single JSON entry to the `_delta_log` directory. The JSON entry is the commit — no other metadata files needed.
- **Flat log structure**: Simpler than Iceberg's tree. The trade-off is that loading table state requires reading the log (or the latest checkpoint).
- **UniForm**: Optionally writes Iceberg metadata files alongside the Delta log, enabling Iceberg-native engines to read the table.
- **Compaction**: Requires OPTIMIZE command (rewrites small files). Liquid Clustering (v3.0+) provides incremental optimization during writes, reducing the need for bulk OPTIMIZE.
- **Deletion Vectors**: Small binary bitmaps enable row-level deletes without rewriting Parquet files — a merge-on-read capability added in v2.3.
- **Result**: Simplest architecture — easy to understand, debug, and operate. Good for batch/Spark workloads. Streaming requires tuning (write mode, file size thresholds, compaction scheduling).
- **Trade-off**: No hidden partitioning, no partition evolution, no REST catalog. The flat log is simpler but doesn't scale as well for very large tables with thousands of partitions.

### Architecture Summary

| Aspect | Paimon (LSM) | Iceberg (Tree) | Delta (Log) |
|--------|-------------|----------------|-------------|
| Write paradigm | Merge-on-read (LSM) | Copy-on-write + merge-on-read | Copy-on-write + Deletion Vectors |
| Streaming write perf | ✅ Excellent | ⚠️ Needs compaction | ⚠️ Needs tuning |
| Batch write perf | Good | ✅ Excellent | ✅ Excellent |
| Query planning | Partition + bucket pruning | ✅ Manifest tree pruning (best) | Checkpoint + stats |
| Compaction | ✅ Automatic, background | ❌ Manual, scheduled | ⚠️ Semi-auto (Liquid Clustering) |
| Metadata complexity | Medium (buckets, levels) | High (5-layer tree) | Low (flat JSON log) |
| Operational overhead | Low (auto everything) | Medium (manual compaction) | Medium (OPTIMIZE, Liquid) |

---

## 7. Ecosystem Support Comparison

This table shows the current state of native integration for each format across major query engines and platforms:

| Platform / Engine | Apache Paimon | Apache Iceberg | Delta Lake |
|------------------|--------------|---------------|------------|
| **Apache Flink** | ✅ **Best (native)** — CDC source/sink, streaming source/sink, Flink SQL | ✅ Good — streaming sink, CDC support | ⚠️ Basic — bucketed writes sink only |
| **Apache Spark** | ✅ Good — Spark SQL, DataFrame API | ✅ Native — Spark connector (full feature parity) | ✅ **Best (native)** — Databricks-optimized, Spark SQL |
| **Trino** | ✅ Connector | ✅ **Native** — Trino Iceberg connector | ✅ Connector (delta-lake) |
| **Presto** | ✅ Connector | ✅ **Native** — Presto Iceberg connector | ✅ Connector |
| **Snowflake** | ❌ Limited | ✅ **Native catalog** — managed Iceberg tables, read/write | ✅ Read connector / UniForm (read-only) |
| **Google BigQuery** | ❌ Not supported | ✅ **Native Iceberg tables** — managed, query directly | ⚠️ Read via Delta connector |
| **AWS Athena** | ❌ Limited | ✅ **Native** — recommended table format on AWS | ⚠️ Limited SELECT via connector |
| **AWS Glue** | ⚠️ Limited | ✅ **Native** — Glue catalog + ETL support | ✅ Catalog + ETL via Spark |
| **AWS EMR** | ⚠️ Limited | ✅ **Native** — built-in Iceberg support | ✅ Connector |
| **Databricks** | ❌ Not supported | ✅ UniForm read / native v3 in Unity Catalog | ✅ **Native** — first-class, most optimized |
| **Dremio** | ❌ Not supported | ✅ **Native** Iceberg support | ⚠️ Limited via connector |
| **StarRocks** | ✅ Connector | ✅ **Native** Iceberg catalog | ❌ Not supported |
| **Apache Hive** | ⚠️ Limited | ✅ Hive catalog + direct read | ✅ Connector |
| **Apache Impala** | ❌ Not supported | ✅ Native Iceberg support | ❌ Limited |
| **Delta Sharing** | ❌ Not supported | ❌ Not supported | ✅ Native (open protocol) |
| **Apache XTable** | ⚠️ Experimental | ✅ Converted to/from | ✅ Converted to/from |

> **Key takeaway**: Iceberg has the broadest native ecosystem support — if you use 3+ query engines (Spark + Trino + Snowflake), Iceberg gives you native read/write everywhere without connectors. Delta Lake has the deepest integration with Databricks and Spark but requires connectors for other engines (though UniForm is closing this gap). Paimon is strongest in the Flink ecosystem but has limited support outside it — its ecosystem is growing but still the narrowest of the three.

---

## 8. When to Choose Which

There is no universal "best" format. The right choice depends on your workload patterns, ecosystem, and organizational priorities.

### Decision Matrix

| If You... | Choose | Rationale |
|-----------|--------|-----------|
| Run **Flink for streaming** pipelines (CDC, Kafka → lake) | **Apache Paimon** | Unmatched streaming ingestion performance — LSM absorbs high-frequency writes natively |
| Need **multi-engine** access (Spark + Trino + Snowflake + Athena) | **Apache Iceberg** | Broadest native support — no connectors needed, REST catalog works everywhere |
| Are a **Databricks** shop (Databricks SQL, MLflow, Unity Catalog) | **Delta Lake** | Deepest integration, first-class performance, full feature parity with the platform |
| Need **streaming + batch** on the same data pipe | **Paimon** (Flink pipelines) or **Iceberg** (if streaming volume is moderate) | Paimon for write-heavy streaming; Iceberg if Flink streaming + multi-engine batch reads |
| Are primarily on **AWS** (Athena, Glue, EMR) | **Apache Iceberg** | Native AWS integration — recommended format across all AWS data services |
| Value **vendor neutrality** above all else | **Apache Iceberg** | Apache Foundation governance, broadest contributor base, no single-company control |
| Want the **simplest architecture** to operate | **Delta Lake** | Flat JSON log is intuitive — easy to understand, debug, and troubleshoot |
| Need **real-time lakehouse** (streaming ingestion + analytics) | **Paimon** for streaming writes, **Iceberg** for batch analytics downstream | Paimon handles the streaming ingestion layer; Iceberg serves the analytical queries |
| Have **mixed infrastructure** (multi-cloud, multi-engine) | **Apache Iceberg** | REST catalog + native support everywhere = write once, query anywhere |
| Run **Snowflake + Spark** together | **Apache Iceberg** | Snowflake has native Iceberg catalog — first-class read/write without external tables |
| Need **cross-organization data sharing** | **Delta Lake** | Delta Sharing is an open protocol with growing adoption across platforms |
| Need **CDC output** for streaming downstream consumers | **Delta Lake** (CDF) or **Paimon** (changelog) | Delta has the most mature built-in CDC; Paimon has native changelog files |
| Prioritize **ad-hoc analytics** with many small queries | **Apache Iceberg** | Hidden partitioning + manifest pruning give the best ad-hoc query performance |
| Want **automatic data optimization** with minimal ops | **Paimon** (auto LSM compaction) or **Delta** (Liquid Clustering) | Paimon compacts continuously; Delta Liquid Clustering is incremental and adaptive |
| Need **file format flexibility** (Parquet + Avro + ORC) | **Apache Iceberg** | Only format with native support for all three file types |

### Decision Flowchart

```
Is streaming ingestion (Flink, CDC, high-frequency writes) your PRIMARY workload?
├── YES → Is multi-engine read access (Trino, Snowflake, Athena) also required?
│         ├── YES → Paimon (streaming writes) + Iceberg (batch reads)
│         └── NO  → Apache Paimon (single-format streaming lakehouse)
└── NO  ──> Are you all-in on Databricks?
           ├── YES → Delta Lake (native, optimized, full feature parity)
           └── NO  ──> Are you primarily on AWS?
                      ├── YES → Apache Iceberg (Athena, Glue, EMR native)
                      └── NO  ──> Do you use 3+ different query engines?
                                 ├── YES → Apache Iceberg (native multi-engine)
                                 └── NO  ──> Do you prioritize simplicity over features?
                                            ├── YES → Delta Lake (JSON log is easiest)
                                            └── NO  → Apache Iceberg (safest default)
```

### The "Safe Choice" Recommendations

- **For most new multi-engine lakehouse deployments (2026+)**: **Apache Iceberg** is the safest default. Broadest native ecosystem, vendor-neutral governance, REST catalog for cross-platform access, and the v3 spec convergence (deletion vectors, VARIANT) closes most feature gaps with Delta.
- **For streaming-first architectures**: **Apache Paimon** is uniquely suited. If you run Flink for CDC pipelines, Paimon's LSM architecture will save significant operational cost on compaction.
- **For Databricks shops**: **Delta Lake** remains the obvious and correct choice. UniForm provides the escape hatch for Iceberg-engine reads.
- **For mixed environments with streaming + analytics**: Consider **Paimon for the ingestion layer** and **Iceberg for the serving layer** — write streaming data to Paimon, then batch-compact/export to Iceberg for multi-engine analytics.

---

## 9. Performance Considerations

Performance differences depend heavily on workload patterns, data volume, and operational practices.

### Write Performance

| Scenario | Paimon | Iceberg | Delta |
|----------|--------|---------|-------|
| **Streaming (high-frequency, small batches)** | ✅ **Best** — LSM absorbs tiny writes, no per-batch metadata rewrite | ⚠️ Moderate — each microbatch creates new files + metadata; compaction overhead | ⚠️ Moderate — similar to Iceberg; requires write mode tuning |
| **Streaming (CDC, upserts, primary key updates)** | ✅ **Best** — LSM merge-on-read handles upserts naturally | ⚠️ Moderate — position deletes / DVs add read overhead | ⚠️ Moderate — Deletion Vectors help but compaction needed |
| **Batch (large, sorted writes)** | Good — comparable after compaction | ✅ Excellent — optimized file creation | ✅ Excellent — Databricks-optimized write path |
| **Small file accumulation** | ✅ Minimal — LSM continuously merges | ⚠️ Significant — requires explicit compaction | ⚠️ Moderate — Liquid Clustering mitigates |

> **Key insight**: Paimon's LSM advantage for streaming is not incremental — it's architectural. Benchmarks from Alibaba and ByteDance show Paimon sustaining **40M+ rows/second** of streaming writes with sub-minute commit latency, while Iceberg/Delta under similar streaming loads see write amplification from per-batch file creation and require aggressive compaction scheduling.

### Read Performance

| Scenario | Paimon | Iceberg | Delta |
|----------|--------|---------|-------|
| **Analytic queries (no deletes)** | Good — after compaction to top LSM level | ✅ Excellent — column stats + manifest pruning | ✅ Excellent — column stats in checkpoint |
| **Analytic queries (with deletes/updates)** | ⚠️ Merge-on-read overhead from LSM levels | ⚠️ Merge-on-read overhead from DVs/delete files | ⚠️ Merge-on-read overhead from DVs |
| **Point lookups (by primary key)** | ✅ Excellent — LSM can skip levels with bloom filters | ⚠️ Moderate — full scan or partition scan | ⚠️ Moderate — no native index |
| **Streaming reads (incremental)** | ✅ Excellent — native changelog files | ✅ Good — incremental read API | ✅ Good — Change Data Feed |
| **Time travel queries** | ✅ Snapshot-based LSM state | ✅ Snapshot-based metadata tree | ✅ Version-based log replay |

### Query Planning

| Aspect | Paimon | Iceberg | Delta |
|--------|--------|---------|-------|
| **Partition pruning** | ✅ Bucket + partition metadata | ✅ Manifest list partition ranges | ✅ Checkpoint partition metadata |
| **File-level skipping** | ✅ Column stats in manifest | ✅ **Best** — per-file stats in manifests | ✅ Column stats in checkpoint |
| **Large table planning speed** | Good | ✅ **Best** — tree structure prunes at manifest list level | ⚠️ Slower — flat log requires more scanning |
| **Metadata overhead per file** | Low (compact LSM metadata) | Medium (multiple manifest layers) | Low (single JSON entry per commit) |

### Compaction Overhead

| Aspect | Paimon | Iceberg | Delta |
|--------|--------|---------|-------|
| **Strategy** | ✅ **Automatic, continuous, tiered** | ❌ Manual — `rewrite_data_files` | ⚠️ Semi-auto — Liquid Clustering + OPTIMIZE |
| **Overhead during writes** | Low — LSM absorbs small files | Medium — must compact separately | Medium — Liquid Clustering mitigates |
| **Read performance impact** | ⚠️ Possible lag if compaction falls behind | ✅ Consistent (compact on schedule) | ✅ Consistent (Liquid Clustering is incremental) |
| **Operational cost** | Low (hands-off) | Medium (requires scheduling, tuning) | Low-Medium (Liquid Clustering reduces need) |

### Summary Performance Table

| Aspect | Paimon (LSM) | Iceberg (Tree) | Delta (Log) |
|--------|-------------|----------------|-------------|
| Streaming writes (high-freq) | ⬆️ **Best** | ⬇️ Needs compaction | ⬇️ Needs tuning |
| Batch writes | ➡️ Good | ⬆️ Excellent | ⬆️ Excellent |
| Analytic reads (no deletes) | ➡️ Good (after compaction) | ⬆️ Excellent | ⬆️ Excellent |
| Analytic reads (with deletes) | ➡️ Moderate (merge overhead) | ➡️ Moderate (DV overhead) | ➡️ Moderate (DV overhead) |
| Streaming reads | ⬆️ Excellent (changelog) | ⬆️ Good (incremental API) | ⬆️ Good (CDF) |
| Query planning (large tables) | ➡️ Good | ⬆️ **Best** (manifest tree) | ➡️ Good |
| Point lookups (PK) | ⬆️ Excellent (bloom filters) | ➡️ Moderate | ➡️ Moderate |
| Compaction ops | ⬆️ Auto, background | ⬇️ Manual, scheduled | ➡️ Semi-auto (Liquid) |
| Small file mitigation | ⬆️ Auto (LSM merges) | ⬇️ Requires compaction | ➡️ Liquid Clustering |

---

## 10. Future Directions

### Apache Paimon

- **Maturing multi-engine support**: Expanding beyond Flink → deeper Spark, Trino, and StarRocks integration. REST catalog support is newer and less mature than Iceberg's.
- **Iceberg compatibility**: Adding features to interoperate with the Iceberg ecosystem (e.g., V3 deletion vector compatibility) for gradual migration paths.
- **Global expansion**: Growing adoption beyond China — more documentation, community events, and enterprise case studies in English.
- **Catalog options**: Adding more catalog implementations and improving the REST catalog to compete with Iceberg's mature ecosystem.
- **Performance optimization**: Continued optimization of LSM compaction strategies for even lower read latency.

### Apache Iceberg

- **v3 adoption across engines**: Deletion Vectors, VARIANT, geospatial, and row lineage are rolling out across Spark, Trino, Flink, Athena, BigQuery, and Snowflake throughout 2026.
- **REST catalog becoming standard**: Growing adoption of the REST catalog as a cross-platform standard — more engines, more clouds, more SaaS platforms.
- **v4 specification discussions**: Early community discussions around v4 features including further convergence with Delta Lake semantics.
- **Deletion Vectors maturity**: As DVs become the default for row-level deletes (replacing v2 positional delete files), read performance for mutable tables will improve.
- **AI/ML integration**: Native support for ML feature stores and vector embeddings within Iceberg metadata.

### Delta Lake

- **UniForm expansion**: Making UniForm write-path compatible (not just read-only) — enabling Iceberg engines to write to Delta tables. This is the logical next step for full bidirectional compatibility.
- **Deeper non-Spark integration**: Delta Kernel (Rust) enables lightweight reads from Python, Node.js, and embedded systems. Expect broader ecosystem support via Kernel.
- **Liquid Clustering improvements**: AI-driven compaction scheduling, adaptive clustering strategies based on workload patterns — targeting 90% latency reduction in compaction operations.
- **REST catalog gap**: The community may develop a REST catalog equivalent for Delta to compete with Iceberg's cross-platform standard.
- **Type system evolution**: Further VARIANT type support, complex nested types, and improved type widening.

### Convergence Trend

All three formats are adding features from each other — the formats are converging:

| Iceberg is adopting from Delta | Delta is adopting from Iceberg | Paimon is adopting from both |
|--------------------------------|-------------------------------|------------------------------|
| ✅ Deletion Vectors (v3) | ✅ UniForm (writes Iceberg metadata) | ✅ Iceberg compatibility features |
| ✅ Row lineage (v3) | ✅ VARIANT type (4.0) | ✅ REST catalog (newer) |
| ✅ Type widening (v3) | ✅ Geospatial support roadmap | ✅ Expanding multi-engine |

**The industry trajectory**: By 2027–2028, the format distinction may matter less for end users as interoperability tools (UniForm, XTable, REST catalog) abstract the differences. The architectures will remain distinct — LSM vs Tree vs Log — but the developer experience will converge on "write once, read anywhere."

---

## 11. Data Sources & Further Reading

### Official Documentation

- **Apache Paimon**: https://paimon.apache.org/ — official docs, GitHub, community resources
- **Apache Iceberg**: https://iceberg.apache.org/ — spec, docs, ecosystem integrations
- **Delta Lake**: https://delta.io/ — official docs, blog, GitHub, release notes

### Engineering Blogs & Conference Talks

- **Alibaba Cloud Blog**: "Apache Paimon: Real-Time Lake Storage with Iceberg Compatibility 2025" — production benchmarks and architecture deep dive
- **Databricks Blog**: "Delta Lake 4.0: Liquid Clustering, UniForm, and the Future of Lakehouse Storage"
- **Databricks Blog**: "Apache Iceberg v3: Moving the Ecosystem Towards Unification" (June 2025)
- **Flink Forward Conference**: Xiaomi's Real-Time Lakehouse Implementation with Apache Paimon (2025)
- **Capital One Tech Blog**: "Lakehouse Convergence: Delta Lake & Iceberg" — industry convergence analysis
- **Ververica Blog**: "Apache Paimon: The Streaming Lakehouse" — architecture and Flink integration

### Community Benchmarks & Comparison Articles

- **BladePipe**: "Choosing Your Data Lake Format in 2025: Iceberg vs Delta Lake vs Paimon" — practical comparison from a data integration platform
- **LLMS3**: "Real-Time Lakehouse: Paimon vs Hudi" — LSM architecture deep dive
- **OLake**: "Apache Paimon vs Apache Iceberg: A Detailed Comparison"
- **Atlan**: "Apache Paimon vs Apache Iceberg: 2026 Evaluation Guide"
- **Jacar**: "Delta Lake and Apache Iceberg: 2025 Comparison" — detailed feature-by-feature comparison
- **Alex Merced (DEV)**: "Understanding Iceberg, Delta Lake, Hudi, Paimon, and DuckLake" — 2025 guide to open table formats
- **Medium (BladePipe)**: "Choosing Your Data Lake Format in 2025: Iceberg vs Delta Lake vs Paimon"

### Specification Documents

- **Iceberg v3 Spec**: Apache Iceberg documentation — deletion vectors, VARIANT, geospatial, row lineage, encryption
- **Delta Lake Protocol**: delta.io/protocol — transaction log specification
- **Paimon Documentation**: paimon.apache.org/docs/1.0/ — LSM architecture, bucket design, file layout

### Industry Reports & News

- **MinIO**: "AIStor Tables and Iceberg V3: Genuine Engineering" — v3 adoption analysis
- **brickster.ai**: Delta Lake and Iceberg roadmap tracking
- **AWS News**: "AWS Adds Support for Apache Iceberg v3 & Deletion Vectors" (November 2025)
- **Apache Software Foundation**: Press release on Paimon graduating to Top-Level Project (April 2024)
- **Koantek**: "Choose Your Own Adventure: Three Paths to Iceberg Success on Databricks"
- **StorageMath**: MinIO AIStor Tables and Iceberg V3 — first mover analysis
