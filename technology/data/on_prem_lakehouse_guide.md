# On-Premises Lakehouse Architecture: A Comprehensive Guide

## Table of Contents
1. [What Is an On-Premises Lakehouse?](#what-is-an-on-premises-lakehouse)
2. [Why Choose On-Premises?](#why-choose-on-premises)
3. [Core Architecture Components](#core-architecture-components)
4. [Storage Layer Deep Dive](#storage-layer-deep-dive)
5. [Table Format Considerations](#table-format-considerations)
6. [Compute Layer](#compute-layer)
7. [Catalog and Governance](#catalog-and-governance)
8. [Deployment Patterns](#deployment-patterns)
9. [Hardware Sizing Guidelines](#hardware-sizing-guidelines)
10. [Networking](#networking)
11. [Performance Optimization](#performance-optimization)
12. [Backup and Disaster Recovery](#backup-and-disaster-recovery)
13. [Migration from Cloud Back to On-Premises](#migration-from-cloud-back-to-on-premises)
14. [Reference Architecture](#reference-architecture)
15. [Case Studies](#case-studies)
16. [On-Prem vs Cloud Lakehouse Comparison](#on-prem-vs-cloud-lakehouse-comparison)
17. [Best Practices](#best-practices)

---

## What Is an On-Premises Lakehouse?

A **data lakehouse** unifies data lake flexibility (schema-on-read, cheap object storage, multi-modal data) with warehouse reliability (ACID transactions, schema enforcement, SQL analytics). An **on-premises lakehouse** implements this architecture entirely within an organization's own data center, using self-hosted hardware rather than public cloud services.

The on-premises lakehouse replaces the cloud-vendor stack — S3 becomes MinIO or Ceph; Glue becomes Nessie or Polaris; Athena/EMR becomes Trino or Spark — while preserving the same patterns: a single copy of data in object storage, a table format layer for ACID, decoupled compute and storage, and a catalog for metadata.

The **MIT Stack** (MinIO + Iceberg + Trino) has emerged as the dominant open-source on-prem combination. Production deployments add a catalog (Nessie or Polaris), orchestration (Airflow or Dagster), governance (Atlas), and optionally a caching tier (Alluxio).

---

## Why Choose On-Premises?

**Data Sovereignty & Regulatory Compliance.** Many jurisdictions mandate data never leave national borders: GDPR (EU), Data Security Law (China), DPDP Act (India), PDPL (Saudi Arabia). Banks under MAS, BaFin, or Fed/OCC regulations often require on-prem processing for customer financial data. On-prem infrastructure guarantees data stays within a known, auditable physical perimeter.

**Cost Predictability.** Cloud lakehouse costs scale with query volume, egress, and API calls — creating unpredictable bills. On-prem hardware is CapEx with predictable depreciation; OpEx is limited to power, cooling, and staffing. At petabyte scale, on-prem TCO often undercuts cloud by 40-60% over 3 years for steady-state workloads.

**Existing Infrastructure Investment.** Organizations with data centers and storage arrays (Dell EMC, NetApp, Pure Storage) can retrofit Hadoop/HDFS clusters into a lakehouse by adding object storage and open table formats without forklift migration.

**Air-Gapped Environments.** Defense, intelligence, and critical national infrastructure operate with zero internet connectivity. An on-prem lakehouse is the only viable architecture — every component must deploy from internal registries and function without phoning home.

**Latency.** When data sources (IoT sensors, trading systems, PLCs) are on-prem, sending data to the cloud and back adds latency. An on-prem lakehouse keeps the entire pipeline in one data center, reducing end-to-end latency from milliseconds to microseconds.

---

## Core Architecture Components

An on-premises lakehouse comprises five logical layers. Below are the dominant technology choices for each.

### Layer 1: Object Storage

| Technology | Best For | Notes |
|---|---|---|
| **MinIO** | Greenfield, K8s-native | S3-compatible, erasure coding, lightweight, high perf |
| **Ceph (RGW)** | Existing Ceph shops | Unified block/file/object; heavier ops |
| **Dell EMC ECS** | Enterprise appliance | Turnkey but proprietary and expensive |
| **NetApp StorageGRID** | NetApp shops | S3-compatible, integrates with NetApp fabric |
| **Pure Storage FlashBlade** | High-perf analytics | All-flash, good for mixed file+object |

**Recommendation:** MinIO for most deployments. Purpose-built for S3-compatible on-prem object storage, runs on commodity hardware, integrates directly with Iceberg's S3FileIO.

### Layer 2: Table Format

| Format | Strengths | On-Prem Fit |
|---|---|---|
| **Apache Iceberg** | Engine-agnostic open spec, multi-engine | Best: no vendor lock, all engines can read/write |
| **Delta Lake** | Unified batch/streaming, Databricks | Only if running Databricks on-prem |
| **Apache Hudi** | Real-time upserts, incremental queries | CDC-heavy workloads; narrower engine support |

**Recommendation:** Iceberg. Its engine independence is the single most important property for on-prem architectures where you want to mix tools.

### Layer 3: Compute Engine

| Engine | Role | On-Prem Sweet Spot |
|---|---|---|
| **Apache Spark** | Heavy ETL, data engineering, ML | Primary ingestion/transformation engine |
| **Trino** | Interactive SQL, BI, federated queries | Default SQL engine for ad-hoc analytics |
| **Apache Flink** | Stream processing, CDC ingestion | Real-time event-driven pipelines |
| **Dremio** | BI acceleration, self-service | Reflections for BI tool sub-second queries |

### Layer 4: Catalog

| Catalog | Type | Notes |
|---|---|---|
| **Hive Metastore** | Legacy | Historical; single point of failure |
| **Nessie** | Git-like versioned catalog | Branching/tagging; implements Iceberg REST |
| **Polaris (Apache)** | Iceberg REST catalog | Open-sourced by Snowflake; multi-engine |
| **Unity Catalog** | Databricks | Tied to Databricks compute |

**Recommendation:** Nessie for git-like version control; Polaris for Snowflake compatibility without cloud lock-in.

### Layer 5: Orchestration

| Tool | Strengths | On-Prem Fit |
|---|---|---|
| **Apache Airflow** | Mature, huge ecosystem | Default; broadest connector support |
| **Dagster** | Asset-aware, software-defined assets | Better for declarative data engineering teams |
| **Apache NiFi** | Visual data flow, source connectors | Good for ingestion from many source systems |

**Recommendation:** Airflow for existing investment; Dagster for new builds prioritizing asset lineage and testability.

---

## Storage Layer Deep Dive

### MinIO for S3-Compatible On-Prem Object Storage

MinIO is the dominant on-prem object store. Key properties:

**S3 API Compatibility.** Any tool speaking S3 (Spark, Trino, Flink, boto3) works without modification. Set `endpoint=http://minio:9000`.

**Erasure Coding.** MinIO splits objects into data + parity shards across drives. Default `EC:4:2` (4 data + 2 parity across 6 drives) tolerates 2 drive failures at ~66% efficiency. EC:8:4 tolerates 4 failures from 12 (~50%). Use the MinIO Erasure Code Calculator to tune for your capacity/failure-tolerance needs.

**Bitrot Protection.** SHA-256 checksums verified on every read; corrupted shards auto-repaired from parity.

**Encryption.** SSE-S3 (MinIO-managed key), SSE-C (customer-provided), SSE-KMS (HashiCorp Vault, AWS KMS).

### Hardware Considerations

| Drive Type | Read (seq) | IOPS | $/TB | Use Case |
|---|---|---|---|---|
| NVMe (U.2/E3.S) | 5-7 GB/s | 500K-1M | $150-250 | Hot tier, Trino workers, Spark shuffle |
| SATA/SAS SSD | 500 MB/s | 50-100K | $70-120 | Warm tier, general lakehouse storage |
| HDD (7200 RPM) | 150 MB/s | 100-200 | $15-20 | Cold tier, archival |

**Guideline:** NVMe for compute-layer local caches and metadata/transaction log paths. Enterprise SSDs (Samsung PM9A3, Solidigm D7-P5510) for MinIO data drives. HDDs for cold archival via lifecycle policies.

**MinIO reference hardware tiers:**
- **Tier 1 (Performance):** 4-8 nodes x 8x NVMe (3.84TB), dual 25GbE, 256GB RAM
- **Tier 2 (Standard):** 8-16 nodes x 12x SSD (7.68TB), dual 25GbE, 128GB RAM
- **Tier 3 (Capacity):** 12-24 nodes x 12x HDD (20TB), dual 10/25GbE, 64GB RAM

### Tiered Storage

Implement lifecycle-based tiering:
1. **Hot (NVMe/SSD):** Active datasets, recent partitions, Iceberg metadata
2. **Warm (SATA SSD):** Weekly/monthly access, older partitions
3. **Cold (HDD/tape):** Archival, raw logs after aggregation

MinIO ILM policies auto-transition objects between tiers. Trino's Iceberg connector supports per-tablespace storage locations.

---

## Table Format Considerations for On-Premises

### Apache Iceberg (Recommended)

Iceberg is the strongest on-prem choice because:
- **Engine independence:** Spark, Trino, Flink, Dremio, Hive, DuckDB all read/write it
- **Open specification:** Standalone spec; no vendor dependency
- **File-level statistics:** Min/max stats per column per file enable efficient predicate pushdown
- **Hidden partitioning:** No manual partition directory management
- **Schema evolution without rewrites:** Add/drop/rename/reorder columns without rewriting data files

**Integration:** Use `S3FileIO` — `spark.sql.catalog.my_catalog.io-impl=org.apache.iceberg.aws.s3.S3FileIO`. No HDFS needed.

### Delta Lake

Right choice when:
- Committed to Databricks (on-prem via Databricks Container Services)
- Using Unity Catalog (ties to Databricks)
- Spark is your only compute engine

**Limitation:** Delta Lake's metadata is not an open specification. Multi-engine access via the "Uniform" format bridge adds latency and version risk.

### Apache Hudi

Strong for real-time upsert/CDC workloads (designed at Uber for high-volume streaming). Trade-offs: steepest learning curve, three table types (CoW, MoR, WAL) need careful configuration, narrower engine support.

### Recommendation Matrix

| Requirement | Best Format |
|---|---|
| Multi-engine access | Iceberg |
| Databricks ecosystem | Delta Lake |
| Real-time CDC | Hudi or Iceberg with streaming |
| Minimum complexity | Iceberg |
| Cloud vendor independence | Iceberg |

---

## Compute Layer

### Trino: Query Engine of Choice for On-Prem SQL

Trino is purpose-built for the on-prem lakehouse:
- **Connector architecture:** Iceberg, Hive, Delta Lake, JDBC connectors. Query multiple sources in a single SQL statement.
- **Separate scaling:** Coordinator + workers scale independently. Add workers for concurrency.
- **Cost-based optimizer:** Leverages Iceberg statistics for optimal join/aggregation strategies.
- **Memory-first execution:** Spills to disk only when exhausted.

**Typical deployment:** 1 coordinator (16 vCPU, 64GB) + 5-20 workers (32 vCPU, 256GB each). Workers are memory-bound — over-provision RAM.

### Spark: The ETL Workhorse

- **Iceberg integration:** Spark 3.x with the Iceberg runtime provides full ACID writes
- **Structured Streaming:** Stream from Kafka/Pulsar into Iceberg tables
- **Spark on K8s:** Executors run as pods with configurable resources
- **Sizing:** 4-8 GB per executor core. Place `spark.local.dir` on local NVMe SSDs

### Dremio: BI Acceleration

Dremio provides **Reflections** — pre-computed, auto-refreshed materialized views in Arrow format. BI queries (Tableau, Power BI, Superset) transparently hit Reflections instead of scanning raw Iceberg files, reducing latency from seconds to milliseconds. Runs on bare-metal or K8s. Requires coordination cluster (3-node HA) + executors.

### Apache Flink: Streaming Compute

- **Iceberg sink:** Exactly-once streaming into Iceberg tables via Iceberg Flink connector
- **CDC ingestion:** Debezium + Kafka + Flink for real-time database replication
- **Flink on K8s:** Native operator handles job lifecycle and checkpointing

---

## Catalog and Governance

### Nessie: Git-like Versioning

Nessie brings Git semantics to Iceberg:
- **Branching:** `CREATE BRANCH dev FROM main` — isolated environments without data copies
- **Tagging:** `CREATE TAG release_2026_Q1` — freeze table state for auditing
- **Time travel:** Query any commit hash or timestamp
- **Merge with conflict detection:** `MERGE dev INTO main` catches concurrent modifications

**Deployment:** Standalone server backed by PostgreSQL. Exposes Iceberg REST API — any Iceberg engine uses it by pointing to the Nessie endpoint.

### Polaris: Iceberg REST Catalog

Open-sourced by Snowflake:
- Multi-catalog support with per-catalog storage/auth config
- RBAC at principal/role/table level
- Any Iceberg engine authenticates through the same Polaris instance

**Deployment:** Java service, Docker image, PostgreSQL backend. No direct Iceberg metadata storage — delegates to the file system (MinIO).

### Apache Atlas: Governance Framework

Atlas provides the governance layer for regulatory compliance:
- **Metadata catalog:** Auto-captures from Hive, Spark, Trino via hooks
- **Data lineage:** Tracks source → transformation → output → BI dashboard
- **Classification:** Tag columns as PII, PCI, HIPAA, GDPR; propagates through lineage
- **Business glossary:** Link business terms to physical columns
- **Audit:** All metadata changes logged

**Integration:** Atlas hooks into Hive/Spark/Trino via Kafka. Uses JanusGraph (HBase/Cassandra + Elasticsearch).

### Governance Stack

```
Apache Atlas (Lineage, Classification, Glossary, Audit)
    Nessie / Polaris (Catalog + RBAC)
    Apache Iceberg (Table Metadata)
    MinIO (Data Files + Manifest)
```

---

## Deployment Patterns

### 1. Kubernetes-Native (Recommended)

Deploy all components on K8s: MinIO Operator, Trino as Deployments/StatefulSets, Spark via Spark Operator, Nessie/Polaris as Deployments, Airflow/Dagster as Deployments.

**Benefits:** Unified ops (kubectl + Helm), resource isolation (namespaces), elastic scaling, rolling upgrades, self-healing.

**Storage:** MinIO on PVCs backed by local NVMe (Local Persistent Volumes or TopoLVM). Avoid NFS-backed PVCs. **Networking:** Cilium with eBPF for high throughput.

### 2. Bare-Metal for Performance-Critical

Dedicated hardware, no virtualization overhead: CPU governor on `performance`, kernel tuned for storage (noop scheduler), NUMA-pinned processes, huge pages.

**Benefits:** 10-20% better throughput, no noisy-neighbor, deterministic latency.
**Trade-offs:** Manual provisioning, no self-healing, harder to scale.

### 3. Hybrid with Cloud Burst

Steady-state on-prem, burst compute to cloud during peak demand:
- On-prem: MinIO (primary), Trino cluster, Nessie catalog
- Cloud: Second Trino cluster reads same Iceberg tables via MinIO endpoint (VPN/Direct Connect)
- Data stays on-prem; cloud compute reads over WAN

**Mitigation:** Alluxio caching on the cloud side reduces WAN traffic for hot data.

---

## Hardware Sizing Guidelines

### Compute-to-Storage Ratio

| Workload | CPU : Storage | Memory : Core | Typical Cluster |
|---|---|---|---|
| Heavy ETL (Spark) | 1:4 | 4-8 GB/core | 200 cores, 800 TB |
| Interactive SQL (Trino) | 1:10 | 8-16 GB/core | 100 cores, 1000 TB |
| Mixed | 1:6 | 8 GB/core | 150 cores, 900 TB |
| BI-heavy (Dremio) | 1:20 | 16-32 GB/core | 50 cores, 1000 TB |

### Memory Per Core

- **Spark:** 4-8 GB/core. Executor + off-heap within NUMA boundaries.
- **Trino:** 8-16 GB/core. Memory under-provisioning causes 10-100x spill-to-disk slowdown.
- **Dremio:** 16-32 GB/core. Reflections and metadata caching consume significant memory.
- **Flink:** 4-8 GB/task-slot. RocksDB checkpoint state adds memory pressure.

### Networking Requirements

| Scenario | Minimum | Recommended |
|---|---|---|
| MinIO erasure coding | 10 GbE | 25 GbE |
| Spark shuffle (100+ TB) | 25 GbE | 100 GbE or RDMA |
| Trino worker-worker | 10 GbE | 25 GbE |
| Trino → MinIO reads | 25 GbE | 2x 25 GbE bonded |
| Cross-rack | 40 GbE spine | 100 GbE spine |

---

## Networking

**25GbE vs 100GbE.** For most deployments up to 500 TB, 2x 25GbE bonded (3.1 GB/s per link, $200-400/NIC) is sufficient. Beyond 1 PB, upgrade to 100GbE spine-leaf (12.5 GB/s, $800-1500/NIC). 400GbE ($3K+/NIC) is edge-case for extreme scale only.

**RDMA vs TCP.** TCP adds ~1-3% CPU overhead per 10 Gbps. RDMA bypasses the kernel, reducing latency from ~50µs (TCP) to ~1-5µs. Two forms:
- **InfiniBand:** Native RDMA fabric. Highest performance. Requires Mellanox/Nvidia ConnectX NICs and switches (~$2K+/port). Expensive and introduces friction: Spark's netty shuffle doesn't natively support IB, MinIO uses standard TCP (IB is wasted), and Trino's HTTP/2 communication has no RDMA support. IPoIB (TCP over IB) negates most RDMA benefits.
- **RoCE (RDMA over Converged Ethernet):** RDMA on standard Ethernet with lossless networking (PFC/DCB). Significantly cheaper. Performance close to IB under ideal conditions, degrades under packet loss.

**Recommendation:** Use RoCE (25/100GbE) for the lakehouse data network. Reserve InfiniBand for ML training clusters running NCCL operations.

---

## Performance Optimization

**Data Locality.** MinIO is shared-nothing — every node serves every object, so locality is less critical than HDFS. Practical optimization: place MinIO and compute on the same rack/top-of-rack switch pair.

**Rack Awareness.** MinIO supports rack-aware erasure coding via topology labels. With EC:16:8 across 3 racks, a full rack failure is survivable (remaining 16 shards on 2 racks are sufficient).

**Alluxio Caching Layer.** Alluxio sits between compute and MinIO. Cache hits serve at NVMe/DRAM speed (3-10 GB/s per node); misses fetch from MinIO (1-2 GB/s per 25GbE link). Deploy Alluxio workers co-located with Trino workers with tiered storage: DRAM → NVMe → SSD → MinIO fallback.

**Parquet Tuning.** Use ZSTD compression, 128-256 MB row groups, dictionary encoding for low-cardinality columns.

**Iceberg Compaction.** Run regular OPTIMIZE jobs targeting 256 MB-1 GB files. Use SORT BY to co-locate related rows for better predicate pushdown.

**Result Caching.** Enable Trino's built-in result caching for repeated dashboard queries.

**Spark AQE.** Adaptive Query Execution auto-coalesces shuffle partitions and adjusts join strategies.

---

## Backup and Disaster Recovery

### MinIO Replication

**1. Bucket Replication (active-active or active-passive):**
- Per-bucket rules replicating to target MinIO (same or remote site)
- Async mode (seconds RPO) or sync mode (zero data loss, added latency)

**2. Site Replication (multi-site active-active):**
- Identity federation with global namespace
- Changes on one site propagate to all sites

### DR Architecture
```
Primary DC                  DR DC (100+ km)
MinIO Cluster A  ─async──  MinIO Cluster B
EC:8:4, 12 nodes           EC:8:4, 12 nodes
                            Cold standby (tape/HDD)
```

### Catalog DR

- **Nessie:** Back up PostgreSQL (WAL archiving or PgBackRest). Catalog metadata is small — references to Iceberg snapshots, not data.
- **Polaris:** Back up Polaris database similarly.
- **Snapshot export:** Nessie exports all commits as Git-style pack files.

### DR Plan

- **RPO:** Seconds (async) to zero (sync)
- **RTO:** Minutes (warm standby) to hours (cold restore)
- **Regular drills:** Quarterly full-site failover tests
- **Validation:** `mc admin heal` for checksum verification after replication

---

## Migration from Cloud Back to On-Premises

### Phase 1: Assessment
- Inventory S3 buckets, Glue catalogs, Athena queries, EMR jobs
- Profile data access patterns, compliance requirements
- Size on-prem hardware

### Phase 2: Stand Up On-Prem Infrastructure
- Deploy MinIO with equivalent capacity
- Deploy Nessie/Polaris; migrate metadata (Iceberg JSON files copy directly)
- Deploy Trino cluster; test MinIO connectivity
- Set up Airflow/Dagster

### Phase 3: Data Transfer
- **Bulk:** `mc mirror` or `rclone`. Multi-PB requires physical transfer (Snowball, Data Box).
- **Initial sync:** Full copy of Iceberg metadata + data files
- **Incremental sync:** Pipe new cloud writes to on-prem via replication
- **Cutover window:** Brief downtime for final incremental sync

### Phase 4: Validation
- Compare row counts and checksums on key tables
- Run 10 most critical production queries on both environments
- Validate BI dashboards return identical numbers

### Phase 5: Cutover
- Point apps to on-prem MinIO endpoint + Nessie/Polaris catalog URL
- Decommission cloud services

### Key Challenges

| Challenge | Mitigation |
|---|---|
| Network bandwidth | Physical transfer (Snowball), parallel sync |
| Hive→Iceberg migration | Iceberg `migrate`/`register` table ops |
| Schema compatibility | Test all source tables before cutover |
| Client compatibility | Verify S3 API compatibility (no proprietary headers) |
| BI tool reconfiguration | Pre-configure ODBC/JDBC to Trino before cutover |

---

## Reference Architecture

Below is the complete on-premises lakehouse reference architecture. The design is Kubernetes-native with MinIO, Iceberg, Nessie, Trino, Spark, Airflow, and Alluxio, following the Medallion (Bronze→Silver→Gold) pattern.

```
                           USERS & CLIENTS
  Tableau  Power BI  Superset  Jupyter  Python  dbt  RStudio
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     ORCHESTRATION LAYER                              │
│     Apache Airflow / Dagster                                        │
│     (ingest → transform → optimize → expire → BI refresh)           │
└─────────────────────────────────────────────────────────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        ▼                  ▼                  ▼
┌───────────────┐  ┌───────────────┐  ┌───────────────┐
│    Trino      │  │ Apache Spark  │  │ Apache Flink  │
│ Coordinator   │  │ Driver +      │  │ Job Manager   │
│ + Workers     │  │ Executors     │  │ + Tasks       │
│ (SQL, BI)     │  │ (ETL, ML)     │  │ (Streaming)   │
└───────┬───────┘  └───────┬───────┘  └───────┬───────┘
        └──────────────────┼──────────────────┘
                           │
                    ┌──────▼──────┐
                    │   Alluxio   │
                    │ (Cache)     │
                    └──────┬──────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         STORAGE LAYER                                │
│   MinIO Object Store                                                │
│   Node1  Node2  Node3  Node4  Node5  ...  NodeN                     │
│   8xNVMe 8xNVMe 8xNVMe 8xNVMe 8xNVMe      8xNVMe                    │
│   EC:8:4 Erasure Coding                                             │
│                                                                     │
│   s3://bronze/  — Raw ingested data                                 │
│   s3://silver/  — Cleaned (Iceberg tables)                          │
│   s3://gold/    — Aggregated (Iceberg tables)                       │
│   s3://metadata/ — Iceberg metadata + Nessie catalog store          │
└─────────────────────────────────────────────────────────────────────┘
                           ▲
                           │
┌─────────────────────────────────────────────────────────────────────┐
│                   CATALOG & GOVERNANCE                               │
│   ┌─────────────────────┐  ┌──────────────────────────────────┐     │
│   │ Nessie / Polaris    │  │ Apache Atlas                     │     │
│   │ (Iceberg REST API,  │  │ (Lineage, Classification,       │     │
│   │  branches/tags)     │  │  Glossary, Audit)                │     │
│   └─────────────────────┘  └──────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────────────┘
                           ▲
                           │
┌─────────────────────────────────────────────────────────────────────┐
│                      INFRASTRUCTURE LAYER                            │
│   Kubernetes: lakehouse/minio/trino/spark/airflow/atlas namespaces  │
│   CNI: Cilium (eBPF)  |  Monitoring: Prometheus + Grafana          │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐                         │
│   │ Rack 1   │  │ Rack 2   │  │ Rack 3   │    25/100 GbE spine    │
│   │ MinIO +  │──│ Trino +  │──│ Spark +  │── leaf-spine fabric     │
│   │ Storage  │  │ Compute  │  │ Streaming│                         │
│   └──────────┘  └──────────┘  └──────────┘                         │
└─────────────────────────────────────────────────────────────────────┘
```

### Data Flow

1. Source → Kafka → Flink → **Bronze** (raw data)
2. Airflow triggers Spark ETL → **Silver** (cleaned Iceberg tables)
3. Airflow triggers Spark/Trino → **Gold** (aggregated Iceberg tables)
4. User queries Gold via Trino/Dremio → Alluxio cache → MinIO
5. BI tools → Trino → Reflections / result cache
6. Atlas captures lineage: Flink→Bronze→Spark→Silver→Trino→BI

---

## Case Studies

### European Bank — Regulatory Compliance
**Challenge:** Top-20 EU bank required all customer data to stay within EU borders. Cloud providers couldn't guarantee data wouldn't transit US under the CLOUD Act.
**Architecture:** MinIO across Frankfurt + Amsterdam DCs. Iceberg tables (Spark ETL, Trino reporting). Nessie catalog with branch isolation per business unit. Atlas tracking PII lineage.
**Results:** Passed BaFin/ECB audits with full data residency evidence. 80% faster reporting. 3-year TCO 55% lower than proposed AWS EU-region solution.

### Government Intelligence — Air-Gapped
**Challenge:** Completely air-gapped environment. Zero internet. All software from internal artifact registries.
**Architecture:** 4 bare-metal MinIO clusters (16 nodes each). Iceberg + Parquet + ZSTD. Trino for SQL, Spark for ML. Nessie catalog with mirrored internal git repos. All container images physically transferred.
**Results:** 500+ TB processed with zero external connectivity. Nessie branching enabled parallel analyst workflows. Zero data exfiltration incidents.

### Healthcare Provider — HIPAA Compliance
**Challenge:** US health network with 15 hospitals. PHI subject to HIPAA. AWS costs and audit overhead became unsustainable.
**Architecture:** MinIO + SSE-KMS (Vault). Iceberg + Nessie. Dremio for BI (Tableau dashboards). Alluxio for patient cohort cache. Airflow for EHR ETL.
**Results:** 60% cost reduction vs AWS. Sub-second BI dashboard queries. HIPAA audit passed with no findings in the data platform domain.

### Online Retailer — Cloud Repatriation
**Challenge:** Top SE Asian e-commerce company. AWS costs exceeded $2M/month, projected $5M/month in 18 months.
**Architecture (post-repatriation):** MinIO (32 nodes, 8x 7.68TB SSD, EC:8:4). Iceberg migrated from Glue to Polaris. Trino replacing Athena. Spark on K8s replacing EMR. Airflow replacing Step Functions.
**Migration:** 200 TB, 1500 tables migrated in 6 months. Zero application changes (S3 API transparent).

---

## On-Prem vs Cloud Lakehouse Comparison

| Dimension | On-Premises | Cloud |
|---|---|---|
| **Initial Cost** | High CapEx | Zero CapEx; pay-as-you-go |
| **TCO (1 PB+, 3 yrs)** | 40-60% lower | Higher (egress, API markup) |
| **Data Residency** | Full control | Region-dependent |
| **Scalability** | Bounded by procurement (days-weeks) | Elastic (minutes) |
| **Elasticity** | Over-provision or hybrid | Auto-scaling |
| **Performance** | Deterministic, no noisy neighbors | Variable, shared infra |
| **Ops Overhead** | Dedicated platform engineers | Managed services |
| **Security** | Full physical control | Shared responsibility |
| **Auditability** | Complete HW/network/log visibility | Cloud provider controls infra layer |
| **Engine Choice** | Any; no vendor lock | Often tied to cloud-native services |
| **Egress Costs** | None | Significant |
| **Time to Market** | Slower (procurement, install) | Faster (API provisioning) |
| **Regulatory Fit** | Easier for strict regimes | Challenging for air-gapped |
| **DR** | Requires dedicated DR site | Built-in multi-region |
| **Staffing** | Infra + data engineering skills | Focus on data engineering |

---

## Best Practices

### Architecture

1. **Decouple everything.** Storage, compute, catalog, orchestration — each independently deployable and scalable.
2. **Medallion architecture.** Bronze (raw) → Silver (cleaned) → Gold (aggregated). Makes data contracts explicit.
3. **Prefer Iceberg.** Engine independence is the most valuable property for on-prem.
4. **One catalog, one source of truth.** Choose Nessie or Polaris, not both.

### Operations

5. **Automate Iceberg maintenance.** OPTIMIZE, EXPIRE_SNAPSHOTS, REMOVE_ORPHAN_FILES on schedule (daily active, weekly less active).
6. **Monitor everything.** Prometheus + Grafana for MinIO usage, Trino P50/P95/P99 latency, catalog latency, network utilization.
7. **Plan for failure.** MinIO erasure coding + Nessie DB backups + quarterly DR drills.
8. **Validate data integrity.** `mc admin heal` periodically. Iceberg manifest-list checksums.

### Performance

9. **Right-size erasure coding.** EC:8:4 for standard workloads. Avoid EC:2:1 in production.
10. **Co-locate compute and storage.** Minimize network hops. Avoid cross-DC queries for hot paths.
11. **Tune Parquet.** ZSTD level 3-7, 128-256 MB row groups, enable column statistics.
12. **Use Alluxio for hot data.** Co-locate workers with Trino. Tiered storage: DRAM → NVMe → MinIO.

### Security

13. **Encrypt everything.** TLS for all connections. SSE-KMS for objects at rest. Rotate keys.
14. **RBAC everywhere.** MinIO IAM + IdP integration. Polaris catalog-level RBAC. Atlas classification-based access.
15. **Audit access.** MinIO access logs, Trino query audit, Nessie commit log (tamper-evident).

### Team

16. **Hire for the stack.** Storage engineers (MinIO/Ceph), platform engineers (K8s/networking), data engineers (Spark/Trino/Iceberg).
17. **Build an internal platform.** Wrap the lakehouse in a developer portal (Backstage) for self-service bucket/table/pipeline provisioning.

### Migration

18. **Test the cutover.** At least two full rehearsals. Validate every critical query returns identical results.
19. **Keep a cloud escape hatch.** Maintain reduced cloud footprint or S3 replication for 6+ months post-migration.
20. **Document the architecture.** ADR for every major choice. This guide is a starting point — make it your own.

---

## Further Reading

- Apache Iceberg Spec: https://iceberg.apache.org/spec/
- MinIO Docs: https://min.io/docs
- Trino Docs: https://trino.io/docs
- Project Nessie: https://projectnessie.org
- Apache Polaris: https://polaris.apache.org
- Apache Atlas: https://atlas.apache.org
- Alluxio: https://www.alluxio.io
- Dagster Lakehouse: https://dagster.io/blog/building-a-better-lakehouse-from-airflow-to-dagster

---

*This guide was written based on publicly available documentation, real-world case studies, and community best practices as of 2026. Every on-premises deployment is unique; use this as a starting framework, not an off-the-shelf blueprint.*
