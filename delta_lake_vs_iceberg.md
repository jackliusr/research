# Delta Lake vs Apache Iceberg: A Comprehensive Comparison Guide

> **Last updated:** July 2026
>
> A thorough, research-backed comparison of the two leading open table formats for data lakes and lakehouses. Covers architecture, features, ecosystem, performance, migration, and decision guidance.

---

## Table of Contents

1. [Overview](#1-overview)
2. [Apache Iceberg Deep Dive](#2-apache-iceberg-deep-dive)
3. [Delta Lake Deep Dive](#3-delta-lake-deep-dive)
4. [Side-by-Side Comparison](#4-side-by-side-comparison)
5. [Ecosystem Support Matrix](#5-ecosystem-support-matrix)
6. [Performance Considerations](#6-performance-considerations)
7. [Migration & Interoperability](#7-migration--interoperability)
8. [Which One to Choose?](#8-which-one-to-choose)
9. [Getting Started](#9-getting-started)
10. [Data Sources & Further Reading](#10-data-sources--further-reading)

---

## 1. Overview

### What Are Table Formats?

A table format is a metadata layer on top of data files (Parquet, Avro, ORC) in distributed storage (S3, ADLS, GCS, HDFS) that provides **relational database-like features** to data lakes. Before table formats, data lakes were loose file collections — you could read them with Spark or Hive, but got none of the guarantees expected from a database.

Table formats solve four critical problems:

- **ACID transactions**: Concurrent readers and writers get consistent, isolated views. No more dirty reads mid-write.
- **Schema evolution**: Add, rename, drop, or reorder columns without rewriting the entire dataset.
- **Time travel**: Query data as it existed at any point — essential for audits, reprocessing, and debugging.
- **Incremental processing**: Read only changed data since the last run, enabling efficient streaming and CDC.

### The Lakehouse Architecture

The **lakehouse** architecture — a term coined by **Databricks** (2020) — combines data lake flexibility with warehouse reliability. Data stays in open formats on object storage, but a transactional metadata layer brings ACID, schema enforcement, indexing, and optimization.

| Data Lake | Data Warehouse | Lakehouse |
|-----------|---------------|-----------|
| Cheap storage | ACID transactions | Cheap storage + ACID |
| Schema-on-read | Schema-on-write | Schema evolution |
| No transactions | Fixed schemas | Open formats |
| Raw files | Proprietary storage | Multi-engine access |

### Brief History

| Format | Creator | Open-Sourced | Governance |
|--------|---------|-------------|------------|
| **Delta Lake** | Databricks | 2019 | Linux Foundation (2022) |
| **Apache Iceberg** | Netflix | 2018 (contributed to Apache) | Apache Software Foundation |
| **Apache Hudi** | Uber | 2016 (contributed to Apache) | Apache Software Foundation |

- **Delta Lake** was born from Databricks' proprietary "Delta Engine" technology. It was open-sourced in April 2019 and moved to the Linux Foundation in 2022 to signal vendor neutrality. Despite the Linux Foundation governance, Databricks remains the primary contributor and drives most of the development.
- **Apache Iceberg** was created by Ryan Blue and Dan Weeks at Netflix to overcome the limitations of Hive tables. It was contributed to the Apache Software Foundation in 2018 and became an Apache top-level project in 2020. It is widely regarded as the most vendor-neutral table format, with significant contributions from Netflix, Apple, Airbnb, Expedia, Dremio, Snowflake, and Google.
- **Apache Hudi** (Hadoop Upserts Deletes and Incrementals) was created at Uber in 2016 and contributed to Apache in 2019. It pioneered many features like upserts and incremental queries, but has a smaller ecosystem footprint than Delta or Iceberg. This guide focuses on Delta Lake vs Apache Iceberg but acknowledges Hudi as a capable third option.

> **Note on Hudi**: Apache Hudi remains strong for workloads prioritizing upsert performance and incremental processing on streaming data, with features like record-level indexing (HFile, Bloom Filters) and advanced clustering. However, its ecosystem support lags behind Delta and Iceberg — particularly on Trino, Snowflake, and AWS Athena.

### The Big Picture (Mid-2026)

As of mid-2026, the table format landscape has matured significantly:

- **Iceberg v3** spec is finalized (deletion vectors, VARIANT type, geospatial, row lineage). Iceberg 1.11.0 is the latest library release.
- **Delta Lake 4.0.1** is current (Liquid Clustering, type widening, improved deletion vectors, UniForm).
- **Interoperability** is now first-class: Delta UniForm generates Iceberg metadata; Apache XTable (Incubating) provides omni-directional conversion.
- **Cloud alignment**: AWS is all-in on Iceberg (Athena, Glue, EMR). Databricks backs Delta but also supports Iceberg v3 natively in Unity Catalog.

---

## 2. Apache Iceberg Deep Dive

### Origins & Governance

- **Created at Netflix** by Ryan Blue and Dan Weeks to address the pain points of Hive table format at petabyte scale.
- **Contributed to the Apache Software Foundation** in 2018 and graduated as a top-level project in 2020.
- **Vendor-neutral governance** under the Apache Foundation's meritocratic model. No single company controls the project's direction.
- **Strong community base**: Netflix, Apple, Airbnb, Expedia, Stripe, Dremio, Snowflake, Google, and AWS are all significant contributors.
- **Snowflake** ships a native Iceberg catalog (Iceberg tables are a first-class citizen).
- **AWS** has made Iceberg the default recommendation for Athena, Glue, and EMR.
- **Adoption signals**: Apple uses Iceberg "wall-to-wall" across its data infrastructure. Netflix manages exabytes of Iceberg data. Adobe, Stripe, and Expedia have all publicly discussed large-scale Iceberg deployments.

### Architecture

Iceberg's table format uses a **tree of metadata files** to provide atomicity and isolation:

```
Table Root
  └── Metadata File (version-N.metadata.json)
       ├── Snapshot reference
       ├── Schema definition
       ├── Partition spec
       ├── Sort order
       └── Manifest List (snapshot-N.avro)
            ├── Manifest File (data file metadata)
            │    └── Data File (Parquet/Avro/ORC)
            ├── Manifest File
            │    └── Data File
            └── Manifest File (delete manifest for v2+)
                 └── Delete File (position deletes / equality deletes)
```

**Metadata file**: The root of the table. Each commit creates a new metadata file (atomic swap in the catalog). Contains the current schema, partition spec, sort order, and references to all snapshots.

**Manifest list**: An Avro file that lists all manifest files in a snapshot, along with partition range information for pruning.

**Manifest files**: Avro or Parquet files that list all data files (and delete files in v2+) belonging to a partition. Each manifest contains column-level statistics (min/max values, null counts) for predicate pushdown.

**Data files**: The actual data stored in Parquet (default), Avro, or ORC format.

**Catalogs**: Iceberg uses a catalog service to track the current metadata file pointer. Supported catalog implementations:

| Catalog | Description |
|---------|-------------|
| **Hive Metastore** | Legacy, widely supported |
| **AWS Glue Catalog** | Native AWS integration |
| **Nessie** | Git-like catalog with branching/merging |
| **JDBC** | Relational database-backed catalog |
| **REST Catalog** | Standard REST API — the most flexible, enables cross-engine, cross-platform access |
| **In-memory** | For testing only |

### Partitioning: Hidden Partitioning

Iceberg's **hidden partitioning** stores partition metadata separate from data — unlike Hive-style directory paths (`year=2024/month=01/`). Users define a partition spec (`PARTITION BY days(ts)`), and Iceberg automatically computes partition values during writes. Partition values are **invisible to query writers** — queries don't need to reference partition columns.

**Benefits**: Users don't need to know the scheme to write efficient queries. Partition specs can evolve without rewriting data. No more "partition column mismatch" bugs.

### Key Features

- **Full ACID transactions** with serializable isolation via optimistic concurrency control.
- **Time travel and rollback** by snapshot ID or timestamp. Each snapshot is immutable.
- **Schema evolution**: Add, drop, rename, reorder, update column types (safe promotions int→long). No data rewriting.
- **Partition evolution**: Change partition spec without rewriting existing data. Old and new partitions co-exist.
- **Hidden partitioning**: Partition values computed automatically and transparently.
- **Compaction**: `rewrite_data_files` (merge small files), `rewrite_manifests` (compact metadata).
- **File skipping**: Column-level min/max statistics in manifests for predicate pushdown.
- **Row-level deletes** (v2): Position-based delete files for efficient MERGE, UPDATE, DELETE.
- **Multi-engine support**: Spark, Flink, Trino, Presto, Hive, Dremio, Snowflake, StarRocks, Impala.
- **REST catalog**: Standardized API for multi-engine, multi-platform, cross-cloud access.

### Iceberg v3 (Spec Finalized 2025–2026)

Iceberg v3 is the most significant evolution since v2. Key additions:

- **Deletion Vectors**: Roaring Bitmap structures replacing positional delete files — reducing read amplification and metadata overhead for frequent updates, converging with Delta Lake's approach.
- **VARIANT data type**: Native semi-structured data support (JSON, schemaless records) without string storage.
- **Geospatial types**: Native geometry types and indexing for spatial queries.
- **Row lineage**: Track operation/source provenance for each row.
- **Type widening**: Safe, automatic column type widening (int→long→float).
- **Default column values**: Specify defaults for omitted insert columns.

> **Status (mid-2026)**: v3 spec finalized. Databricks Unity Catalog supports Iceberg v3. Engine ecosystem (Spark, Trino, Flink) in various stages of v3 implementation. Latest library release: **Iceberg 1.11.0** (May 2026).

---

## 3. Delta Lake Deep Dive

### Origins & Governance

- **Created by Databricks** as the storage layer for their lakehouse platform, tracing back to internal "Delta Engine" technology.
- **Open-sourced April 2019** under Apache 2.0.
- **Moved to Linux Foundation in 2022** — signaling broader governance, though **Databricks still drives the vast majority of development**.
- **Community**: Strong within Databricks ecosystem (Spark, MLflow, Unity Catalog). Growing outside with connectors for Trino, Presto, Flink, Snowflake, but smaller and less active than Iceberg's.

### Architecture

Delta Lake uses a **single transaction log** (delta log) rather than a tree of metadata files:

```
Delta Table Root
  └── _delta_log/
       ├── 00000000000000000001.json  (commit 1)
       ├── 00000000000000000002.json  (commit 2)
       ├── 00000000000000000003.json  (commit 3)
       ├── ...
       └── _last_checkpoint          (points to latest checkpoint)
  └── Checkpoint Files (periodic)
       ├── 00000000000000000010.checkpoint.parquet
       └── 00000000000000000020.checkpoint.parquet
  └── Data Files (*.parquet)
  └── Deletion Vector Files (*.dv)
```

**Transaction log**: A directory of ordered JSON files, each representing a single atomic commit. Each commit file records:
- Which data files were added
- Which data files were removed (logical deletion)
- Schema changes
- Commit metadata (user, timestamp, operation type)
- Application-level transaction identifiers

**Checkpoint files**: Periodically, Delta Lake writes a Parquet snapshot of the transaction log state. This dramatically speeds up table load times — instead of replaying thousands of JSON commits, the reader loads the latest checkpoint and applies only the trailing commits.

**Data files**: Parquet format (default and only native format). With UniForm, Delta can also write Iceberg-compatible manifest files against the same Parquet data.

**Deletion Vectors** (v2.3+): Small bitmaps that mark which rows within a Parquet file are logically deleted. This enables row-level deletes without rewriting the entire Parquet file — a "merge-on-read" approach.

**Catalogs**:

| Catalog | Description |
|---------|-------------|
| **Hive Metastore** | Legacy, widely supported |
| **Unity Catalog** | Databricks' own catalog — rich governance, lineage, RBAC |
| **AWS Glue Catalog** | Supported via connector |
| **File system** | Path-based tables (no catalog required) |

### Key Features

- **Full ACID transactions** with serializable isolation via optimistic concurrency control.
- **Time travel and rollback**: Query by version (`VERSION AS OF`) or timestamp (`TIMESTAMP AS OF`).
- **Schema evolution**: Add, rename, drop, update column types. Type widening in Delta 4.0.
- **Schema enforcement**: Strictly rejects writes with mismatched schemas.
- **Generated columns**: Auto-computed column values (useful for pre-computed partition keys).
- **Deletion Vectors** (v2.3+): Row-level MERGE, UPDATE, DELETE without rewriting entire Parquet files. Merge-on-read approach.
- **Liquid Clustering** (v3.0+): Continuous, incremental data layout optimization without full rewrites. Adapts to write patterns over time.
- **Change Data Feed** (CDC): Built-in row-level change tracking exposed as a stream — no external tools needed.
- **Constraints**: NOT NULL and CHECK constraints for data quality.
- **Multi-engine support**: Spark (native), Flink, Trino, Presto, Hive, Snowflake, BigQuery (via connectors).

### Delta Lake 4.0.x (Current, Mid-2026)

Delta Lake 4.0.1 is the latest release. Key improvements over 3.x:

- **Liquid Clustering GA**: More efficient incremental clustering, better log integration.
- **Coordinated commits**: Atomic multi-table commits across Unity Catalog for cross-table consistency.
- **Type widening**: Automatic numeric type widening (INT→LONG→DECIMAL).
- **Improved Deletion Vectors**: Better read performance for heavy merge/update workloads.
- **UniForm GA**: Delta tables auto-generate Iceberg metadata for Iceberg-native engines (Trino, Snowflake, Athena).
- **Delta Kernel**: Stable modular API for non-Spark engine integration.

> **UniForm in practice**: When UniForm is enabled on a Delta table, Delta Lake writes Iceberg metadata files (metadata.json, manifest lists, manifest files) side-by-side with the Delta log. Iceberg readers see a valid Iceberg table. This is a **write-Delta, read-Iceberg** model — it does not make Delta tables writable from Iceberg engines.

---

## 4. Side-by-Side Comparison

| Feature | Delta Lake | Apache Iceberg |
|---------|------------|---------------|
| **Governance** | Linux Foundation (Databricks-led) | Apache Foundation (vendor-neutral) |
| **Open Source License** | Apache 2.0 | Apache 2.0 |
| **ACID Transactions** | ✅ Serializable isolation | ✅ Serializable isolation |
| **Time Travel** | ✅ Version + timestamp | ✅ Snapshot ID + timestamp |
| **Schema Evolution** | ✅ Add/drop/rename/update + type widening (4.0) | ✅ Add/drop/rename/reorder/update + type widening (v3) |
| **Hidden Partitioning** | ❌ Manual (users must know partition columns) | ✅ Automatic — hidden from queries |
| **Partition Evolution** | ❌ Must REPARTITION (rewrites data) | ✅ Can change partition spec without rewriting |
| **Row-level Deletes** | ✅ Deletion Vectors (v2.3+) | ✅ Position deletes (v2) / Deletion Vectors (v3) |
| **Merge (Upsert)** | ✅ MERGE with optimized write-audit-publish | ✅ MERGE with row-level deletes |
| **Clustering / Ordering** | ✅ Liquid Clustering (v3.0+) — incremental, adaptive | ✅ SortOrder + rewrite_data_files (manual) |
| **CDC Output** | ✅ Change Data Feed (built-in) | ❌ No built-in CDC (incremental read API available) |
| **Schema Enforcement** | ✅ Strict (rejects mismatched schemas) | ✅ Strict (rejects mismatched schemas) |
| **Optimistic Concurrency** | ✅ Write conflict detection via transaction log | ✅ Optimistic locking via catalog |
| **Catalog Options** | Hive Metastore, Unity Catalog, AWS Glue, filesystem | Hive Metastore, AWS Glue, Nessie, REST, JDBC |
| **Native File Formats** | Parquet only | Parquet, Avro, ORC |
| **Incremental Queries** | ✅ Via Change Data Feed | ✅ Via incremental read API |
| **Spark Support** | ✅ Native (Databricks-optimized) | ✅ Native |
| **Flink Support** | ✅ (bucketed writes sink) | ✅ (first-class connector with streaming sink) |
| **Trino / Presto** | ✅ (Delta Lake connector) | ✅ (native Trino connector) |
| **Snowflake** | ✅ (read via connector) | ✅ (native Iceberg catalog) |
| **AWS Athena** | ❌ No native support; limited via connector | ✅ Native (recommended format) |
| **AWS Glue** | ✅ Catalog + ETL support | ✅ Catalog + ETL support (native) |
| **Multi-cloud / Multi-engine** | Via UniForm (read-only for Iceberg engines) | Via REST catalog (read + write from any engine) |
| **Open Standard** | Linux Foundation project (Databricks-driven) | Apache TLP (community-driven) |
| **Learning Curve** | Low–Medium (JSON transaction log is intuitive) | Medium (manifest tree has more concepts) |
| **Documentation Quality** | Excellent (Databricks docs + delta.io) | Excellent (iceberg.apache.org + community blogs) |

### Key Differentiating Features Explained

**Hidden Partitioning (Iceberg win)**: Users write `WHERE ts >= '2024-01-01'` and Iceberg automatically prunes partitions. Delta requires explicit partition columns in queries — users must know the scheme to write efficient filters.

**Partition Evolution (Iceberg win)**: Iceberg changes partition specs without rewriting data. Delta requires a full `REPARTITION` (all data rewritten).

**CDC / Change Data Feed (Delta win)**: Delta's Change Data Feed is built-in — enable it and every write produces a streamable change log. Iceberg lacks a built-in CDC mechanism.

**Liquid Clustering (Delta win)**: Delta's incremental clustering continuously optimizes layout without full rewrites. Iceberg requires manual `rewrite_data_files` calls.

**REST Catalog (Iceberg win)**: A standard API for multi-engine, multi-cloud catalog access. Delta lacks an equivalent cross-platform catalog standard.

**File Format Flexibility (Iceberg win)**: Iceberg natively supports Parquet, Avro, and ORC. Delta supports Parquet only.

**Deletion Vectors — Converged (both)**: Iceberg v3 introduced Deletion Vectors (Roaring Bitmaps), converging with Delta's approach. Delta's DVs are more mature; Iceberg v3 adoption across engines is ongoing as of mid-2026.

---

## 5. Ecosystem Support Matrix

| Platform / Engine | Delta Lake | Apache Iceberg |
|-------------------|-----------|---------------|
| **Databricks** | ✅ Native (first-class, most optimized) | ✅ Read/write via UniForm; native Iceberg v3 in Unity Catalog |
| **Apache Spark** | ✅ Native (Spark SQL, DataFrame API) | ✅ Native (Spark connector) |
| **Apache Flink** | ✅ Connector (bucketed writes sink) | ✅ Native connector (streaming sink, CDC) |
| **Trino** | ✅ Connector (delta-lake) | ✅ Native (iceberg connector) |
| **Presto** | ✅ Connector | ✅ Native (iceberg connector) |
| **Snowflake** | ✅ Read via DeltaLake connector (external tables) | ✅ Native Iceberg catalog (managed Iceberg tables) |
| **Google BigQuery** | ✅ Read via Delta connector | ✅ Native Iceberg tables (managed) |
| **AWS Athena** | ⚠️ Limited SELECT via connector | ✅ Native Iceberg tables (recommended) |
| **AWS Glue** | ✅ Catalog + ETL (Spark jobs) | ✅ Catalog + ETL (native Iceberg support) |
| **AWS EMR** | ✅ Connector | ✅ Native (built-in Iceberg support) |
| **Dremio** | ⚠️ Limited via connector | ✅ Native Iceberg support |
| **StarRocks** | ❌ Not supported | ✅ Native Iceberg catalog |
| **Apache Hive** | ✅ Connector | ✅ Hive catalog support |
| **Apache Impala** | ❌ Limited | ✅ Native Iceberg support |
| **Apache Paimon** | ❌ N/A | ⚠️ Competes in stream-table space |
| **Delta Sharing** | ✅ Native (open protocol) | ❌ No built-in sharing protocol |
| **Apache XTable** | ✅ Converted to/from | ✅ Converted to/from |

> **Key takeaway**: Iceberg has significantly broader native ecosystem support. If you use multiple query engines (Spark + Trino + Snowflake), Iceberg gives you native support everywhere. Delta Lake requires connectors — which add latency, version-matching constraints, and feature gaps.

---

## 6. Performance Considerations

Performance differences between Delta Lake and Iceberg depend heavily on your workload patterns. Here are the key areas where they differ:

### Query Planning Speed

**Iceberg** has an advantage due to its manifest list structure enabling **partition-level pruning** at planning time — only manifests for relevant partitions are read. For tables with thousands of partitions, this dramatically reduces planning time.

**Delta Lake** must replay the transaction log (or load the latest checkpoint) to build state, then scan all active file entries. Checkpoints mitigate this, but the flat structure means more data to scan.

**Winner**: Iceberg (for large, highly-partitioned tables)

### Read Performance (Row-Level Deletes)

**Iceberg v2** uses position delete files — separate files listing deleted row positions, applied as merge-on-read at query time.

**Delta Lake** uses Deletion Vectors — compact bitmaps embedded alongside data files. More compact than Iceberg v2's approach but still adds read overhead.

**Iceberg v3** introduces Deletion Vectors using Roaring Bitmaps — more compact and faster to evaluate than Delta's DVs in many benchmarks.

**Winner**: Tie (both use merge-on-read; Iceberg v3 Roaring Bitmaps may edge ahead)

### Write Performance

**Delta Lake** has optimized write paths in Databricks Runtime (optimized writes, auto-compaction, bin-packing). Liquid Clustering incrementally optimizes layout during writes.

**Iceberg** write performance is engine-dependent. Spark writes are equivalent. Flink streaming writes to Iceberg are more mature than Delta's Flink sink.

**Winner**: Delta Lake (on Databricks); Tie (open-source Spark)

### Compaction and Optimization

**Delta Lake's Liquid Clustering** (v3.0+) continuously and incrementally optimizes data layout without separate jobs. Clustering columns can be changed over time.

**Iceberg** requires explicit `rewrite_data_files` (Spark procedures or Trino `ALTER TABLE EXECUTE optimize`). Powerful but manual.

**Winner**: Delta Lake (Liquid Clustering is more automated)

### File Skipping (Predicate Pushdown)

**Both** support column-level statistics for file skipping. Iceberg stores min/max stats in manifest files per data file. Delta stores stats in the transaction log. Both achieve excellent skip performance on selective filters.

**Winner**: Tie (Iceberg may be faster for very large tables due to tree structure)

### Small File Management

**Delta Lake's Liquid Clustering** addresses small files implicitly during clustering. OPTIMIZE also supports bin-packing.

**Iceberg** requires explicit `rewrite_data_files` with bin-packing strategies. More manual but finer-grained control.

**Winner**: Delta Lake (more automated)

### Summary Performance Table

| Aspect | Delta Lake | Iceberg |
|--------|-----------|---------|
| Query planning (large tables) | ⬇️ Slower (flat log) | ⬆️ Faster (manifest tree) |
| Read perf (no deletes) | ✅ Parquet native | ✅ Parquet native |
| Read perf (with row deletes) | ⬇️ DV overhead | ⬇️ Position delete/V3 DV overhead |
| Write throughput (Spark) | ✅ Databricks-optimized | ✅ Competitive |
| Write throughput (Flink) | ⚠️ Basic | ✅ Mature streaming sink |
| Compaction / clustering | ✅ Liquid Clustering (auto) | ⚠️ Manual rewrite required |
| File skipping | ✅ Good | ✅ Excellent (manifest stats) |
| Small file handling | ✅ Liquid Clustering | ⚠️ Manual compaction |

---

## 7. Migration & Interoperability

### The Interoperability Landscape (Mid-2026)

One of the biggest shifts in the table format space has been the growing emphasis on **interoperability**. Rather than forcing a single format on an entire organization, the industry is developing tools and standards that allow formats to coexist and data to flow across them.

### Delta Lake UniForm (Universal Format)

Delta Lake 3.0+ introduced **UniForm**, which writes Iceberg-compatible metadata alongside the Delta transaction log during every write operation:

```
Delta Table with UniForm
  ├── _delta_log/             (Delta metadata - always authoritative)
  ├── metadata/               (Iceberg metadata files)
  │   ├── 00001-a1b2.metadata.json
  │   ├── snap-12345-1-manifest-list.avro
  │   └── snap-12345-2-manifest.avro
  └── *.parquet               (Shared data files)
```

**How it works**: When UniForm is enabled, every Delta write also generates the Iceberg metadata tree pointing to the same Parquet data files. Iceberg-native engines (Trino, Snowflake, Athena) can read the table as if it were a native Iceberg table.

**Important caveats**:
- **Read-only from Iceberg engines**: Iceberg engines can read, but cannot write to UniForm tables.
- **Latency**: Iceberg metadata is generated after the Delta commit, creating a brief window where Delta and Iceberg views may diverge.
- **Feature gaps**: Some Delta-specific features (Liquid Clustering, Change Data Feed) don't have Iceberg equivalents in UniForm.
- **Performance**: Iceberg engines reading UniForm tables may not see optimal layout since the data was organized by Delta's clustering strategy.

### Apache XTable (Incubating)

Apache XTable provides **omni-directional interoperability** across Delta Lake, Apache Iceberg, and Apache Hudi. Unlike UniForm (which is one-directional: Delta → Iceberg), XTable can convert between any two formats in either direction.

**Use cases**:
- One-time migration from Iceberg to Delta (or vice versa)
- Synchronizing metadata across formats for multi-engine environments
- Gradual migrations where different teams use different formats
- Cross-format data sharing between organizations

**How it works**: XTable reads one format's metadata, builds a common conceptual model, and generates the other format's metadata from that model. It operates as a standalone CLI or Spark job — not during writes like UniForm.

**Current status**: Apache XTable is in incubation at the Apache Software Foundation. It supports Delta Lake, Iceberg, and Hudi. The project is gaining traction as a neutral interoperability layer.

### Direct Conversion

For one-time migrations, direct conversion tools exist — read from format A, write to format B via Spark (`df.read.format("delta").write.format("iceberg")`). This rewrites all data. For in-place conversion (no data rewrite), tools like XTable are the safer option.

### Migration Best Practices

**Multi-engine** (Spark + Trino + Snowflake): Start with **Iceberg** natively — REST catalog gives every engine native read/write.

**Databricks-centric**: Stay with **Delta Lake**. Use UniForm for occasional Iceberg-engine access. For two-way Iceberg compatibility, use native Iceberg tables in Unity Catalog (supported since 2025).

**Cloud alignment**:
- **AWS**: Iceberg is the recommended format for Athena, Glue, EMR.
- **Azure**: Both work; Delta has tighter integration with Azure Databricks.
- **GCP**: Both work; BigQuery has native Iceberg tables and a Delta connector.

**Both co-existing**: Many organizations run both — Delta for Databricks workloads (ETL, ML, Databricks SQL) and Iceberg for multi-engine/cloud-agnostic workloads (Trino dashboards, Snowflake analytics, Flink streaming). This hybrid approach avoids lock-in.

### Future Outlook

The industry is moving toward **interoperability as a standard** rather than convergence on a single format:
- Delta UniForm for write-Delta-read-Iceberg
- Iceberg REST catalog for multi-engine access
- Apache XTable for omni-directional conversion
- REST catalog and table format abstractions becoming standard

By 2027–2028, the format distinction may matter less for end users as tools abstract it away. For now, the choice still has significant operational implications.

---

## 8. Which One to Choose?

There is no universal "better" format — the right choice depends on your specific environment, workloads, and priorities.

### Decision Guide

| If You... | Choose | Rationale |
|-----------|--------|-----------|
| Are primarily a **Databricks** user (Databricks SQL, MLflow, Unity Catalog) | **Delta Lake** | Best integration, first-class performance, full feature set |
| Need **multi-engine** support (Spark + Trino + Flink + Snowflake) | **Iceberg** | Widest native support — no connectors needed |
| Run primarily on **AWS** (Athena, Glue, EMR) | **Iceberg** | Native AWS integration; AWS recommends Iceberg |
| Value **hidden partitioning** for ad-hoc queries | **Iceberg** | Automatic, transparent, user-friendly |
| Need **CDC output** for streaming downstream consumers | **Delta Lake** | Built-in Change Data Feed — no extra tooling |
| Value **vendor neutrality** above all else | **Iceberg** | Apache Foundation governance, no single-company control |
| Want the **simplest architecture** | **Delta Lake** | Flat JSON transaction log is intuitive and easy to debug |
| Need **wide file format support** (Parquet + Avro + ORC) | **Iceberg** | Native support for all three formats |
| Are running **Snowflake + Spark** together | **Iceberg** | Snowflake has native Iceberg catalog support |
| Prioritize **incremental data optimization** | **Delta Lake** | Liquid Clustering automates what Iceberg does manually |
| Want **cross-cloud / multi-platform** access | **Iceberg** | REST catalog is the only true multi-platform standard today |
| Have **many small, ad-hoc analytics queries** | **Iceberg** | Hidden partitioning + manifest pruning provide better ad-hoc performance |
| Operate a **streaming-heavy pipeline** (Flink/Kafka) | **Iceberg** | Flink connector is more mature for streaming sinks |
| Need **data sharing** across organizations | **Delta Lake** | Delta Sharing is an open protocol with growing adoption |

### Decision Flowchart (Simplified)

```
Are you all-in on Databricks?
├── YES → Delta Lake (native, optimized, full feature set)
└── NO  ──> Are you on AWS?
           ├── YES → Iceberg (Athena, Glue, EMR native)
           └── NO  ──> Do you use 3+ query engines?
                      ├── YES → Iceberg (native multi-engine)
                      └── NO  ──> Do you need CDC?
                                 ├── YES → Delta Lake (Change Data Feed)
                                 └── NO  ──> Tie — evaluate based on team expertise
```

### The "Safe Choice" Recommendations

- **For most new lakehouse deployments (2026+)**: Apache Iceberg is the safer default. It has broader native ecosystem support, vendor-neutral governance, and the REST catalog provides true multi-engine access.
- **For Databricks shops**: Delta Lake is the obvious and correct choice. The performance and feature integration advantages are significant.
- **For organizations already running both**: Don't force a migration. Use UniForm for Delta → Iceberg reads, and keep both formats for their respective strongest use cases.

---

## 9. Getting Started

### Delta Lake

**Prerequisites**: Apache Spark 3.x, Python 3.8+, or a Databricks workspace.

```python
from pyspark.sql import SparkSession
spark = SparkSession.builder \
    .appName("delta-lake-demo") \
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
    .getOrCreate()

# Create table
spark.sql("CREATE TABLE events (id INT, ts TIMESTAMP, event STRING) USING DELTA")

# Write
df = spark.createDataFrame([(1, "2024-01-01", "click"), (2, "2024-01-02", "view")],
                           schema="id INT, ts STRING, event STRING")
df.write.format("delta").mode("append").save("/path/to/delta-table")

# Time travel
spark.read.format("delta").option("versionAsOf", 5).load("/path/to/delta-table")
spark.read.format("delta").option("timestampAsOf", "2024-01-05").load("/path/to/delta-table")

# MERGE (upsert)
spark.sql("""MERGE INTO events AS target USING updates AS source ON target.id = source.id
             WHEN MATCHED THEN UPDATE SET target.event = source.event
             WHEN NOT MATCHED THEN INSERT *""")

# Enable Change Data Feed
spark.sql("ALTER TABLE events SET TBLPROPERTIES (delta.enableChangeDataFeed = true)")

# Enable UniForm (Iceberg read-compatibility)
spark.sql("ALTER TABLE events SET TBLPROPERTIES (delta.universalFormat.enabledFormats = 'iceberg')")

# Liquid Clustering
spark.sql("ALTER TABLE events CLUSTER BY (id)")
```

### Apache Iceberg

**Prerequisites**: Apache Spark 3.x with Iceberg runtime bundle, or any supported engine (Trino, Flink, etc.).

```python
from pyspark.sql import SparkSession
spark = SparkSession.builder \
    .appName("iceberg-demo") \
    .config("spark.sql.extensions", "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions") \
    .config("spark.sql.catalog.demo", "org.apache.iceberg.spark.SparkCatalog") \
    .config("spark.sql.catalog.demo.type", "hadoop") \
    .config("spark.sql.catalog.demo.warehouse", "/path/to/warehouse") \
    .getOrCreate()

# Create table
spark.sql("CREATE TABLE demo.db.events (id INT, ts TIMESTAMP, event STRING) USING iceberg")

# Write
df = spark.createDataFrame([(1, "2024-01-01", "click"), (2, "2024-01-02", "view")],
                           schema="id INT, ts STRING, event STRING")
df.writeTo("demo.db.events").append()

# Time travel
spark.read.format("iceberg").option("snapshot-id", 123456789).load("demo.db.events")
spark.read.format("iceberg").option("as-of-timestamp", "2024-01-05T00:00:00Z").load("demo.db.events")

# MERGE
spark.sql("""MERGE INTO demo.db.events AS target USING updates AS source ON target.id = source.id
             WHEN MATCHED THEN UPDATE SET target.event = source.event
             WHEN NOT MATCHED THEN INSERT *""")

# Compaction and maintenance
spark.sql("CALL demo.system.rewrite_data_files(table => 'demo.db.events', strategy => 'binpack')")
spark.sql("CALL demo.system.rewrite_manifests('demo.db.events')")
spark.sql("SELECT * FROM demo.db.events.snapshots")  # list snapshots
```

### Quick Start with Trino (Iceberg)

```sql
USE iceberg.demo_schema;
CREATE TABLE events (id BIGINT, ts TIMESTAMP(6), event VARCHAR)
WITH (format = 'PARQUET', partitioning = ARRAY['day(ts)']);
INSERT INTO events VALUES (1, TIMESTAMP '2024-01-01 00:00:00', 'click');
-- Time travel
SELECT * FROM events FOR TIMESTAMP AS OF TIMESTAMP '2024-01-02 00:00:00';
```

---

## 10. Data Sources & Further Reading

This guide synthesizes information from the following authoritative sources (accessed mid-2026):

### Official Documentation
- **Delta Lake**: https://delta.io/ and https://docs.delta.io/
- **Apache Iceberg**: https://iceberg.apache.org/ and https://iceberg.apache.org/spec/
- **Delta Lake UniForm**: https://docs.delta.io/delta-uniform/
- **Apache XTable**: https://xtable.apache.org/

### Engineering Blogs
- Databricks: https://www.databricks.com/blog
- Netflix Tech: https://netflixtechblog.com/
- Dremio (Iceberg): https://www.dremio.com/blog/
- Google Open Source (Iceberg v3): https://opensource.googleblog.com/
- Onehouse (Hudi/Delta/Iceberg): https://www.onehouse.ai/blog

### Vendor Docs
- **Snowflake**: Iceberg catalog docs
- **AWS Athena/Glue**: Iceberg integration docs
- **GCP BigQuery**: Iceberg tables docs
- **Trino/Flink**: Iceberg connector docs

### Community Comparisons
- Onehouse: "Apache Hudi vs Delta Lake vs Apache Iceberg" (Feb 2026)
- Mitzu: "Delta Lake vs Apache Iceberg: Which Table Format Wins?"

### Releases
- Delta Lake: https://github.com/delta-io/delta/releases
- Iceberg: https://iceberg.apache.org/releases/

---

> **Disclaimer**: The data lakehouse landscape evolves rapidly. While this guide reflects the state of both formats as of mid-2026, features, performance characteristics, and ecosystem support may change. Always consult official documentation for the most current information.
>
> **Contributing**: If you find errors or omissions, please open an issue or pull request at the research repository where this guide is maintained.

---

*Generated via research-backed analysis. This document is part of a personal research repository at https://github.com/jackliusr/research.*
