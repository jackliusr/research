# Alibaba Data Platform: Architecture, Lakehouse, Lineage & Practices

> A comprehensive guide to Alibaba Group's data platform — MaxCompute, DataWorks, Apache Paimon,
> Hologres, AnalyticDB, Data Vault methodology, lineage, ETL, data quality, and the OneData
> governance framework.

---

## 1. Alibaba Data Platform Overview

### Scale

Alibaba processes **exabytes of data daily** across e-commerce (Taobao, Tmall, AliExpress),
cloud computing (Alibaba Cloud), logistics (Cainiao), payments (Ant Group/Alipay), entertainment
(Youku), and food delivery (Ele.me). During Double 11 (Singles' Day), the platform handles
over 500,000 transactions per second with real-time analytics pipelines maintaining sub-second
latency at peak.

### Core Technologies

| Component | Role | Notes |
|---|---|---|
| **MaxCompute (ODPS)** | Core data engine | Petabyte-scale SQL + MapReduce |
| **DataWorks** | IDE + orchestration + governance | All-in-one data development platform |
| **Flink (Realtime Compute)** | Stream processing | Alibaba is the #1 corporate contributor |
| **Hologres** | Real-time interactive analytics | PostgreSQL-compatible, HSAP |
| **AnalyticDB** | Real-time OLAP engine | MySQL/PG compatible, sub-second queries |
| **EMR** | Open-source big data platform | Hadoop/Spark/Hive/Presto managed |
| **Paimon** | Streaming lakehouse table format | Formerly Flink Table Store, Apache TLP |
| **RocketMQ** | Distributed messaging | Eventing for data pipelines |

### Evolution

1. **Hadoop Era (2009–2012):** Self-built Hadoop clusters. ODPS project started September 2009.
2. **MaxCompute Native (2012–2017):** ODPS → MaxCompute. DataWorks emerged. Data middle platform concept.
3. **Cloud-Native (2018–2022):** Core systems fully migrated to cloud. MaxCompute 2.0 serverless.
   OneData methodology standardized. 538K peak TPS at Double 11.
4. **AI-Powered (2023–Present):** NL-to-SQL, intelligent data discovery, LLM integration,
   agentic streaming (Flink Forward 2026 vision). Open lakehouse with Paimon.

---

## 2. MaxCompute (ODPS) — The Core Compute Engine

MaxCompute (formerly ODPS) is Alibaba's **massive-scale, fully managed, multi-tenant data
processing platform** — comparable to BigQuery or Redshift but originally built for on-prem
deployment at Alibaba before becoming a cloud service.

### Architecture

Three-layer MPP architecture:
- **Client Layer:** SDKs, CLI, JDBC/ODBC drivers, DataWorks web interface
- **Master/Scheduler Layer:** Distributed scheduler for job acceptance, DAG construction,
  fault tolerance, and resource arbitration across tenants
- **Worker/Compute Layer:** Thousands of worker nodes organized into resource groups (quotas)
  for multi-tenant isolation. Online scaling — nodes add/remove without downtime.

### Key Features

- **MaxCompute SQL:** ANSI SQL 2003 with UDFs, window functions, CTEs, MERGE. SQL is the
  primary ETL interface (>90% of jobs).
- **MapReduce:** Native API for complex transformations not expressible in SQL.
- **Graph/ML:** Distributed graph algorithms; built-in ML functions on tables without data movement.
- **Streaming:** Tunnel SDK + Flink integration for real-time ingestion.
- **Multi-language UDF:** Python, Java, SQL UDFs.

### Storage

- **Columnar, compressed:** Auto-compression per column (gzip, snappy, zstd)
- **Auto-partitioning:** By time or custom keys. Partition pruning for optimization.
- **Clustering:** Sorted within partitions for query acceleration.
- **Tiered storage:** Hot (SSD) → warm (HDD) → cold (archive), auto-managed.

### Scale

- Thousands of worker nodes per cluster
- **Hundreds of petabytes processed daily**
- Tens of thousands of concurrent queries

### MaxCompute 2.0

Full ACID, Delta Lake capabilities (time travel, merge, schema evolution), materialized views,
auto-tuning, serverless mode, native DataWorks integration for lineage + quality.

---

## 3. DataWorks — Data Development Platform

DataWorks is Alibaba's **all-in-one data development and governance platform** — combining SQL
IDE, workflow orchestrator (Airflow-like), data catalog, lineage system, quality monitor, and
governance console in one web app. Think **DataGrip + Airflow + DataHub + dbt + Monte Carlo**
integrated by default.

### Core Capabilities

**Data Integration:** Batch sync from 50+ sources (MySQL, PG, SQL Server, Oracle, HDFS, S3)
into MaxCompute/Hologres/AnalyticDB. Streaming via Flink CDC integration and logtail agents.
Schema drift detection with configurable behavior (alert/ignore/auto-evolve).

**ETL Development:** Multi-engine SQL editor (MaxCompute, Hologres, EMR Hive/Spark). Visual
drag-and-drop ETL. Flink SQL for stream processing. Python/Java script nodes. MCQA for
ad-hoc interactive queries.

**Workflow Orchestration:** DAG-based scheduling with cron, event-driven triggers (file arrival,
RocketMQ signals), cross-cycle dependencies, retry logic, DingTalk alerting, resource governance
(concurrency, queue priority, compute cost).

**Data Catalog & Metadata:** Auto-ingestion of schemas, partitions, statistics, comments.
Business glossary, tag-based classification (PII/internal/public), full-text search.

**Data Modeling:** Star/snowflake schema (Kimball), Data Vault 2.0, or custom. Visual ER
diagrams, model-to-physical forward engineering, reverse engineering from databases.

---

## 4. Alibaba's Lakehouse Architecture

### Evolution

1. **Gen 1 (2009–2012):** Hive-on-Hadoop, batch-only, hours-late data.
2. **Gen 2 (2012–2021):** MaxCompute warehouse with ODS-CDM-ADS layers. Batch-first, faster.
3. **Gen 3 (2022–present):** Open lakehouse on Apache Paimon — streaming-first, compute-storage
   decoupled, engine-agnostic reads.

### Apache Paimon — Streaming-First Table Format

Paimon (formerly Flink Table Store, graduated to Apache TLP in 2025) combines the **lake format**
(open file-based storage on OSS/S3/HDFS) with the **Log-Structured Merge-Tree (LSM-Tree)**
approach. This enables high-frequency writes from streaming jobs with sub-minute visibility.

**Key features:** ACID transactions, streaming ingestion (CDC/append-only/upsert), time travel,
schema evolution, dynamic bucket scaling, auto-compaction, multiple engine support (Flink, Spark,
Hive, Trino, StarRocks, Doris), changelog producers, lookup joins.

### Hologres — Real-Time Analytics

PostgreSQL-compatible **HSAP (Hybrid Serving & Analytical Processing)** engine:
- Row-store (point lookups) + column-store (analytics) in same table
- Sub-second queries on billions of rows
- Separation of storage and compute
- Direct Flink/MaxCompute writes
- Performance advantage over ClickHouse on complex joins and subqueries

### AnalyticDB — Real-Time OLAP

Cloud-native MPP OLAP database (MySQL and PostgreSQL variants):
- All-column indexes maintained asynchronously for low latency
- Petabyte-scale analysis in seconds
- Materialized views with auto-refresh

### Lakehouse Integration

```
Paimon tables (OSS object store)
  ↕ MaxCompute (batch ETL) | Flink (streaming) | Hologres (real-time queries) | AnalyticDB (dashboards)
  ↕ DataWorks (lineage, quality, catalog, orchestration)
  ↕ BI tools, dashboards, data APIs, ML notebooks, AI agents
```

This replaces Lambda Architecture with a **single copy of data** queryable by any engine —
eliminating duplication and inconsistency.

---

## 5. Data Lineage at Alibaba

### Automatic Extraction

DataWorks extracts **column-level lineage** from every execution engine via SQL parsing:

- **MaxCompute SQL:** Every SELECT, INSERT, CTAS, MERGE parsed for input→output column mappings
- **Flink SQL:** Stream INSERT INTO...SELECT parsed
- **Data Integration jobs:** Schema mappings translated to column lineage
- **Visual ETL nodes:** Each transformation component defines column mapping contracts

### Granularity

For every column: upstream source columns, downstream dependents, transformation logic
(SQL expression), job/pipeline metadata (which job, what time, what parameters).

### Use Cases

**Impact analysis:** "What breaks if I change table X?" — lineage traversal finds every
downstream table, view, dashboard, and ML feature, with severity and owner info.

**Root cause analysis:** "Where did this wrong KPI come from?" — backward traversal identifies
the upstream table, ETL step, and job run that produced the erroneous value.

**Compliance:** PII tag propagation through lineage (tagged columns auto-propagate sensitivity
to all downstream derived columns). BCBS 239 audit trails for financial reporting.

### Visualization

Table-level DAGs, column-level drill-down, forward/backward navigation, time-travel lineage
(as it existed at a specific point).

### Governance Integration

Lineage as a governance primitive: quality failures auto-alert downstream consumers; table
retirement uses lineage to find dependencies; sensitivity tags propagate automatically; cost
attribution via data flow tracing.

---

## 6. ETL Practices at Alibaba

### Batch ETL

**SQL-based batch processing on MaxCompute** (>90% of ETL jobs). Partition-based incremental
loading (daily/hourly partitions). Watermark management tracks processed partitions.

### Streaming ETL

**Flink SQL** for stream processing. **Flink CDC** for real-time change data capture from
MySQL/PG/Oracle/SQL Server to Paimon/Hologres/MaxCompute. YAML-based low-code ingestion jobs.
Unified batch/stream API via Flink's stream-table duality.

### ELT Pattern

1. **Extract & Load:** Raw data into MaxCompute ODS tables with minimal transformation
2. **Transform:** All business logic via SQL within MaxCompute (dbt-like pattern, natively
   in DataWorks)
3. **No data movement:** SQL engine handles everything in-place

### Orchestration

DataWorks DAG workflows: nodes as SQL/MR/shell/HTTP units, dependency graph with topological
sort, cron + event-driven scheduling, conditional branching, for-each loops, cross-cycle
dependencies.

### Incremental Loading Strategies

| Strategy | Description | Use Case |
|---|---|---|
| Partition-based | Process new/modified partitions | Time-series data |
| Watermark tracking | High-water mark per table | Append-only data |
| CDC (Flink CDC) | Binlog change events | Tables with updates/deletes |
| Snapshot comparison | MD5 hash comparison | Slowly changing dimensions |
| Full refresh | Truncate and reload | Small dimension tables |

### Data Modeling Choice

**Kimball star schema** for BI/reporting domains (simplicity, query performance). **Data Vault 2.0**
for enterprise cross-domain integration (auditability, scalability, source change tolerance).

---

## 7. Data Vault at Alibaba

### Adoption

Data Vault 2.0 is used where: data comes from multiple heterogeneous sources, full audit trail
required, sources change unpredictably, cross-domain integration needed (e.g., Customer 360
across e-commerce + logistics + payments).

### Implementation on MaxCompute

Hubs/Links/Satellites as MaxCompute tables:
- **Hash keys:** MD5 of natural business keys — deterministic, enables fully parallel ingestion
  without coordination
- **Partitioning:** Time-based partitions on satellites for manageability
- **Scale:** Billions of rows per satellite, trillions total, thousands of satellites per hub
  in largest models

### Automation via DataWorks

- Hub/Link/Satellite templates with correct hash key columns, load date, record source
- SQL macros: `HASH_MD5(business_key)`, hash diff comparison for satellite change detection
- Visual source-to-target mapping → auto-generated ETL code
- Automated satellite comparison (hash diff → insert changed records only)

### Key Use Case: Customer 360

Unifies customer data across e-commerce (purchases), logistics (addresses), payments (profile),
customer service (tickets). Shared hubs (customer, product, order, address). Links connect hubs.
Satellites capture time-varying attributes per source. Single auditable representation queryable
for analytics, ML, and operations.

---

## 8. Data Quality & Profiling at Alibaba

### DataWorks Data Quality Center

Built-in quality monitoring as part of ETL workflows — not a separate system:
- Rules defined at table/column level, executed as DAG nodes
- Workflow can stop or alert on quality failure
- **Lineage integration:** failure auto-identifies downstream consumers for targeted alerting
- Real-time dashboards with pass/fail rates, trends, domain scores

### Quality Dimensions

| Dimension | Example Rule |
|---|---|
| Completeness | NOT NULL, null rate < 1% |
| Accuracy | Range checks, reference data validation |
| Uniqueness | Primary key uniqueness, duplicate detection |
| Consistency | Cross-table referential integrity |
| Timeliness | Data freshness (max age of newest partition) |
| Validity | Format checks (email, phone, date) |

### Auto-Profiling

On table creation: row count, null count, distinct count, min/max/mean/median/stddev, top-N
frequencies, histogram for numeric columns, data distribution for categorical, outlier detection.
Scheduled re-profiling tracks distribution drift over time.

### Rule Library

System built-in (NOT NULL, UNIQUE, referential integrity, row count comparison, volatility check),
template rules (parameterized), custom SQL (full expressions with reference tables), cross-table
rules (every order_id must exist in dimension).

### Monitoring & Alerting

- DingTalk integration for alerts (to pipeline owner, domain owner, downstream consumers)
- Severity levels: Error (blocking), Warning (non-blocking), Info (advisory)
- SLA tracking for data freshness and completeness

### Governance Coupling

PII/financial columns have mandatory quality rules. Each data domain has an owner responsible
for quality SLAs. Quality impact analysis via lineage.

---

## 9. Data Warehouse Architecture

### Layered Architecture (OneData Standard)

```
ADS — Application Data Store (domain-specific marts for BI/reporting)
CDM — Common Data Model (conformed dimensions and facts)
  DIM (dimensions) | DWD (detail fact) | DWS (summary fact) | FACT (pre-joined)
ODS — Operational Data Store (raw data, minimal transformation)
SOURCES — OLTP databases, logs, IoT streams, APIs
```

**ODS (Operational Data Store):** Raw data landing zone. Mirrors source tables with type coercion
and column renaming only. Naming: `ods_{source}_{table}`. Retention: 7–30 days hot + archive.

**CDM (Common Data Model):** Single conformed layer shared across all business units.
- DIM: Conformed dimensions (customer, product, date, geography) with SCD Type 2
- DWD: Atomic-level fact tables (most granular)
- DWS: Pre-aggregated summaries for repeated queries
- FACT: Pre-joined fact tables for BI tool consumption

**ADS (Application Data Store):** Domain-specific marts. Star schemas or flattened tables for
BI tools. Examples: `ads_sales_daily_report`, `ads_customer_lifetime_value`.

### OneData Methodology

Alibaba's data governance framework, codified in the **Dataphin** product:

**Principles:**
1. **One source of truth:** Every data element has one authoritative definition and source
2. **Separation of concerns:** ODS (raw) → CDM (conformed) → ADS (application) with different
   ownership and quality SLAs
3. **Domain-driven ownership:** Each data domain has a designated owner
4. **Standardize before innovate:** Enforce common models before customizations
5. **Automated governance:** Lineage, quality, catalog are automatic

**Outcomes:** Unified business glossary, standardized KPI definitions with documented formulas,
conformed dimensions across all marts, formal deprecation process with lineage-based impact
analysis.

---

## 10. Key Open Source Contributions

### Apache Flink (Largest Corporate Contributor)
- Stream-table duality (unified batch/stream model)
- Flink CDC: Distributed, no-lock CDC from MySQL/PG/Oracle/SQL Server
- Flink SQL Gateway for interactive sessions
- Async I/O, MATCH_RECOGNIZE (CEP), state backend optimization
- Paimon donation (Flink Table Store)
- Realtime Compute for Apache Flink (managed cloud service)

### Apache Paimon
Streaming lakehouse table format with LSM-tree. Apache TLP (graduated 2025). Native integration
with Flink, Spark, Hive, Trino, StarRocks, Doris. Key innovation: lake format + LSM-tree for
high-frequency streaming writes on object storage.

### Apache RocketMQ
Distributed messaging platform. High-throughput pub/sub, exactly-once delivery, financial-grade
transactions. Used in data platform for event-driven pipeline triggers and CDC event distribution.

### Apache Dubbo
High-performance RPC framework. Part of broader Alibaba ecosystem; data platform services
integrate with it for API exposure.

### Flink CDC (Standalone)
YAML-based data ingestion tool. Distributed snapshot scanning (no-lock), auto schema evolution,
sharding table merge, dynamic table addition. Direct sink to Kafka, Paimon, Hologres, MaxCompute.

---

## 11. Comparison: Alibaba Stack vs Western Equivalents

| Alibaba | Western Equivalent | Key Differentiator |
|---|---|---|
| **MaxCompute** | BigQuery / Redshift | Originally on-prem, deeper DataWorks/PAI integration |
| **DataWorks** | Airflow + dbt + DataHub + Monte Carlo | Single platform vs composition. Built-in lineage + quality |
| **Hologres** | ClickHouse | PG-compatible, better at complex joins, HSAP hybrid model |
| **AnalyticDB** | Snowflake / Redshift | MySQL+PG ecosystems, lower latency for real-time ingestion |
| **Paimon** | Iceberg / Delta Lake | LSM-tree designed for streaming writes, tighter Flink integration |
| **EMR** | Amazon EMR / Dataproc | Broader engine support (StarRocks, Doris included) |
| **OneData/Dataphin** | dbt + Collibra + Alation | Prescriptive methodology, lineage-quality coupling built-in |
| **RocketMQ** | Kafka / SQS | Better transactional messaging, designed for order processing |

**Alibaba advantages:** Integration density (DataWorks collapses 5-7 tools), streaming-first
lakehouse (Paimon LSM-tree), battle-tested at Double 11 scale, governance enforcement built
into platform.

**Western advantages:** Multi-cloud ecosystem integration, tool choice flexibility, broader
community adoption (Iceberg/Delta), richer dbt ecosystem.

---

## 12. Key Lessons from Alibaba's Approach

### 1. Integrated Platform > Best-of-Breed
DataWorks combines lineage, quality, ETL, catalog, and orchestration in one tool. Integration
debt compounds faster than best-of-breed advantages — Alibaba's bet on a single platform
eliminates context-switching and ensures governance artifacts are always connected.

### 2. Data Vault 2.0 at Exabyte Scale
Proven at billions of rows per satellite, trillions total. Keys to success: automation (templates,
macros, auto-ETL), deterministic hash keys on MaxCompute for parallel ingestion, time-based
partitioning, collision detection monitoring.

### 3. Streaming-First Lakehouse
Paimon's LSM-tree design proves table formats can support streaming natively. Sub-minute
freshness on object stores, single data copy replaces Lambda Architecture, engine-agnostic
reads across Flink/Spark/Trino.

### 4. Column-Level Lineage as Infrastructure
Not a value-add — it's foundational. Impact analysis is free (every schema change surfaces
blast radius). Root cause in minutes. Quality failures propagate to the right people automatically.
PII tagging propagates through lineage graph. Build lineage early, at platform level, from
SQL parsing.

### 5. OneData: Standardize the Skeleton, Customize the Organs
Unified definitions at the model/metric level, flexibility at application/consumption level.
ODS/CDM governed centrally, ADS governed per domain. Single metric definitions enforced through
the business glossary. Domain ownership with accountability.

### 6. Automate Quality from Day One
Auto-profiling on table creation. Quality rules as part of ETL DAG, not a separate process.
DingTalk alerts to the right person (pipeline owner, not central ops). Trend tracking for
drift detection, not just point-in-time failures.

---

## Data Sources & Further Reading

- **MaxCompute:** alibabacloud.com/help/en/maxcompute
- **DataWorks:** alibabacloud.com/help/en/dataworks
- **Hologres:** alibabacloud.com/help/en/hologres
- **AnalyticDB:** alibabacloud.com/help/en/analyticdb
- **Paimon:** paimon.apache.org | **Flink:** flink.apache.org
- **Flink CDC:** github.com/apache/flink-cdc
- **RocketMQ:** rocketmq.apache.org
- **Papers:** "AnalyticDB: Real-time OLAP at Alibaba" (VLDB 2019), "Hologres: Cloud-Native HSAP"
  (VLDB 2020), MaxCompute 2.0 evolution blog series
- **Conferences:** Flink Forward Asia, ApacheCon Asia, QCon presentations from Alibaba engineers

---

> **Last Updated:** July 2026 — Compiled for research reference
