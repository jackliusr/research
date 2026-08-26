# The Data Pipeline Handbook: Architecture, Patterns, Tools & Best Practices

> A comprehensive guide to designing, building, and operating data pipelines — from batch ETL to real-time streaming.

---

## 1. What is a Data Pipeline?

### Definition

A **data pipeline** is an automated sequence of steps that move and transform data from source systems to destination systems. It reads data from a source, applies processing logic, and writes the result somewhere useful.

### Pipeline vs ETL vs ELT

| Concept | Description |
|---------|-------------|
| **Pipeline** | Umbrella term — any automated data movement and processing. Includes ETL, ELT, streaming, CDC, API ingestion, event-driven patterns. |
| **ETL** (Extract, Transform, Load) | Data extracted from sources, transformed in a staging layer, then loaded. Traditional approach, predates modern cloud warehouses. |
| **ELT** (Extract, Load, Transform) | Data extracted and loaded raw into the destination, then transformed in-destination. Enabled by powerful warehouses (Snowflake, BigQuery). |

**Key insight:** All ETL/ELT processes are pipelines, but not all pipelines are ETL/ELT. A real-time fraud detection stream or a CDC feed into Kafka are pipelines but don't fit neatly into the ETL paradigm.

### Core Components

| Layer | Description | Examples |
|-------|-------------|----------|
| **Source** | Where data originates | Databases (PostgreSQL, MySQL, MongoDB), APIs, files (CSV, Parquet, JSON), streams (Kafka, Kinesis), SaaS platforms (Salesforce, Shopify) |
| **Processing** | What happens to data in transit | Transform (clean, normalize, enrich, aggregate), Validate (schema checks, quality gates), Route (partition, filter), Compute (windowed aggregations, ML inference) |
| **Destination** | Where processed data lands | Data warehouses (Snowflake, BigQuery, Redshift), data lakes (S3/ADLS/GCS), BI tools (Tableau, Looker), ML feature stores, reverse ETL targets |

### Pipeline Characteristics

| Dimension | Options |
|-----------|---------|
| **Trigger** | Scheduled (cron, time-based) vs Event-driven (webhook, CDC, message queue) |
| **Processing mode** | Batch (bounded data sets) vs Streaming (unbounded, continuous) vs Micro-batch |
| **Hop count** | Single-hop (source → destination) vs Multi-hop (bronze → silver → gold) |
| **Shape** | Linear, Fan-out (one-to-many), Fan-in (many-to-one) |

---

## 2. Pipeline Architecture Patterns

### 2.1 Batch Pipeline

Data is collected over a time window and processed in bulk on a schedule (nightly, hourly).

**Common tools:** Apache Airflow + dbt, Spark batch jobs, SQL-based ETL, AWS Glue

**Best for:** Reporting/BI, historical analysis, large-volume transformations, compliance/audit.

**Pros:** Simple to reason about, easy to debug (deterministic re-runs), well-understood semantics, lower infrastructure cost (compute spins up/down).

**Cons:** High latency (data is hours/days old), not suitable for real-time decisions, boundary effects with late-arriving data, expensive full reprocesses at scale.

### 2.2 Streaming Pipeline

Data is processed continuously as it arrives, with millisecond-to-second latency.

**Common tools:** Apache Flink, Kafka Streams, Spark Structured Streaming, RisingWave, Materialize, ksqlDB

**Best for:** Real-time dashboards, fraud detection, alerting, event-driven applications, real-time ML features.

**Pros:** Sub-second latency, real-time operational insights, handles unbounded data naturally, event-time processing with late-data handling.

**Cons:** Significantly more complex than batch, hard state management (backpressure, checkpointing), challenging exactly-once semantics end-to-end, expensive always-on compute, smaller talent pool.

### 2.3 Micro-Batch Pipeline

A middle ground: data is buffered for a short interval (30s–5min) and processed in small batches.

**Common tools:** Spark Streaming (micro-batch mode), Flink (can toggle), Kafka with batch consumers

**Best for:** Near-real-time needs where exactly-once is critical; organizations transitioning batch→streaming.

**Pros:** Simpler than pure streaming (each micro-batch is deterministic), natural exactly-once semantics via batch boundaries, good balance of latency vs complexity, mature tooling.

**Cons:** Not true real-time (always has window-bound latency), windowing complexity for event-time processing, micro-batch overhead per cycle.

### 2.4 Lambda Architecture

Pioneered by Nathan Marz: two parallel pipelines merged at query time.

- **Batch layer:** Full historical reprocessing. Complete, accurate, high-latency.
- **Speed layer:** Real-time recent data. Approximate, low-latency.
- **Serving layer:** Merges batch + speed results at query time.

**Pros:** Handles both batch and streaming, batch layer provides verifiable truth, human fault tolerance.

**Cons:** Two codebases to maintain, complex reconciliation at query time, redundant compute. Jay Kreps (Kafka creator) argued against it — see Kappa below.

### 2.5 Kappa Architecture

Introduced by Jay Kreps (2014): **single streaming pipeline** handles everything. Batch is treated as a stream from the beginning.

**How it works:** All data flows through one stream processor. Historical reprocessing replays the event log from the origin. No separate batch codebase.

**Key enablers:** Durable replayable event log (Kafka), stream processor with exactly-once + state snapshots, ability to reset consumer offsets.

**Pros:** Single codebase, simpler than Lambda, natural reprocessing (replay stream), event log as source of truth enables point-in-time reconstruction.

**Cons:** Streaming infra is inherently complex; full stream replay is expensive at petabyte scale; requires Kafka with long retention and sufficient throughput; smaller pure-streaming ecosystem.

### 2.6 Data Mesh (Organizational Pattern)

Introduced by Zhamak Dehghani (ThoughtWorks) — an **organizational** pattern, not a technology choice.

**Four principles:**
1. **Domain Ownership** — Each business domain owns its data end-to-end.
2. **Data as a Product** — Data has defined SLAs, contracts, schemas, documentation.
3. **Federated Governance** — Global standards for interoperability; domains implement autonomously.
4. **Self-Serve Infrastructure** — Platform team provides tools for domains to build/serve data products.

**Pros:** Scales with organization growth, domain expertise applied to data quality, reduces central bottleneck, faster time-to-insight.

**Cons:** Requires significant organizational maturity, not a technology change (org structure change), governance overhead, potential duplicated infra, can be overkill for <100-person orgs.

---

## 3. Pipeline Lifecycle

1. **Ingestion** — Extract data from source (full load, incremental, CDC).
2. **Validation** — Schema checks, quality checks, row-level validation, freshness/volume checks.
3. **Transformation** — Clean, enrich, aggregate, model (dimensional modeling, Data Vault, OBT).
4. **Storage** — Load to warehouse, lake (Parquet/Iceberg/Delta), or serving layer.
5. **Orchestration** — Schedule, monitor, alert, retry, manage dependencies, backfill.
6. **Observability** — Track data freshness, volume, quality, lineage, costs.

---

## 4. Ingestion Patterns

| Pattern | Method | Tools | Latency | Use Case |
|---------|--------|-------|---------|----------|
| **Full Table Load** | `SELECT *` + `TRUNCATE`/`REPLACE` | SQL-based ETL, Spark | Batch | Small dimension tables |
| **Incremental Load** | Filter by timestamp/sequence | Airflow + SQL, dbt | Batch (scheduled) | Large fact tables |
| **CDC (Change Data Capture)** | Read DB changelogs (binlog/WAL) | Debezium, Maxwell, AWS DMS, Fivetran | Near-real-time | Transactional OLTP databases |
| **Log Tail** | Stream application logs | Fluentd, Logstash, Filebeat | Near-real-time | Observability, monitoring |
| **API Polling** | Scheduled HTTP requests | Airflow, Airbyte, Meltano | Batch | SaaS data (CRM, marketing) |
| **API Webhook** | Push-based event delivery | Custom endpoints, Zapier, Svix | Real-time | Event-driven integrations |
| **Message Queue** | Pub/sub consumption | Kafka, RabbitMQ, Pulsar | Real-time | Event streaming |
| **File Drop** | Watch cloud storage for new files | S3 + Lambda, Airflow sensors | Varies | Partner data exports |

### CDC Deep Dive

CDC is the most reliable way to capture DB changes without impacting source performance.

**Approaches:**

| Method | Mechanism | Source Impact | Latency |
|--------|-----------|---------------|---------|
| **Log-based** (binlog/WAL) | Reads transaction log | Negligible | Sub-second |
| **Trigger-based** | DB triggers write to tracking table | Small overhead | Real-time |
| **Query-based** | Poll timestamp/version columns | Moderate SELECT load | Seconds-minutes |
| **Table diff** | Periodic full comparison | Heavy | Minutes-hours |

**Use CDC when:** Source is critical OLTP (no extra load allowed), near-real-time replication needed, deletes/DDL changes must be captured, schema changes frequently.

**Avoid CDC when:** Infra complexity is too high (Kafka cluster + connectors), full CDC history requires significant storage, schema evolution handling is non-trivial.

---

## 5. Transformation Patterns

### 5.1 SQL-Based (dbt Model)

```sql
-- dbt model: stg_orders.sql
WITH source AS (
    SELECT * FROM {{ source('shopify', 'orders') }}
),
renamed AS (
    SELECT
        id AS order_id,
        customer_id,
        total_price::DECIMAL(10,2) AS order_amount,
        created_at::DATE AS order_date,
        CASE WHEN total_price > 100 THEN 'high_value'
             WHEN total_price > 50  THEN 'medium_value'
             ELSE 'standard' END AS order_tier,
        COALESCE(status, 'unknown') AS order_status
    FROM source WHERE _deleted = FALSE
)
SELECT * FROM renamed
```

**Key SQL patterns:** Deduplication (`ROW_NUMBER() OVER ... = 1`), date spine (`GENERATE_SERIES`), pivot/unpivot, Type-2 SCD (dbt snapshots), windowed aggregations, incremental merge (`NOT EXISTS` on timestamp).

### 5.2 PySpark (Distributed)

```python
from pyspark.sql import functions as F

df = spark.read.table("bronze_events")
result = (
    df
    .filter(F.col("event_type").isin(["purchase", "refund"]))
    .withWatermark("event_time", "10 minutes")
    .groupBy(
        F.window("event_time", "1 hour"),
        F.col("user_id"), F.col("event_type")
    )
    .agg(
        F.sum("amount").alias("total_amount"),
        F.count("*").alias("event_count"),
        F.avg("amount").alias("avg_ticket_size")
    )
    .join(user_segments_df.select("user_id", "segment"), on="user_id", how="left")
)
result.write.mode("overwrite").partitionBy("event_type") \
    .option("compression", "zstd").saveAsTable("silver_hourly_events")
```

**Use PySpark when:** 100s GB–PBs of data, complex UDFs/ML inference, operating on data lakes (Parquet/Iceberg/Delta). **Avoid when:** Data fits in a single-node warehouse (use SQL+dbt), low-latency needed, simple transformations.

### 5.3 Flink SQL (Streaming)

```sql
CREATE TABLE orders (
  order_id STRING, amount DECIMAL(10,2), ts TIMESTAMP(3),
  WATERMARK FOR ts AS ts - INTERVAL '5' SECOND
) WITH ('connector' = 'kafka', 'topic' = 'orders', 'format' = 'json');

INSERT INTO hourly_revenue
SELECT TUMBLE_START(ts, INTERVAL '1' HOUR) AS window_start,
       product_id, SUM(amount) AS total_revenue, COUNT(*) AS order_count
FROM orders
GROUP BY TUMBLE(ts, INTERVAL '1' HOUR), product_id;
```

**Streaming concepts:** Windowing (tumbling, hopping, sliding, session), watermarks (declare lateness tolerance), state (RocksDB-backed, TTL-configurable), temporal joins, pattern matching (CEP), backpressure handling.

### 5.4 Data Modeling Patterns

| Pattern | Description | Best For | Tools |
|---------|-------------|----------|-------|
| **Kimball (Dimensional)** | Star schemas: fact + conformed dimensions | BI reporting, OLAP | dbt, SQL |
| **Data Vault** | Hubs (keys), Links (relationships), Satellites (attributes) | Auditability, agility | dbt + automation |
| **One Big Table (OBT)** | Denormalized single table per entity | ML training, simple queries | Spark, SQL |
| **Medallion (Lakehouse)** | Bronze (raw) → Silver (cleaned) → Gold (aggregated) | Modern data lakes, AI | Spark, Delta Lake, Iceberg |

---

## 6. Orchestration & Scheduling

| Tool | Type | Best For | Key Differentiator |
|------|------|----------|--------------------|
| **Apache Airflow** | DAG-based scheduler | Complex dependencies, large ecosystem | Largest community, Python DAGs, most integrations |
| **Prefect** | Modern workflow engine | Python-native teams, event-driven | Better DX than Airflow, hybrid execution, built-in retries |
| **Dagster** | Asset-focused orchestrator | Quality + lineage as first-class concerns | Software-defined assets, dbt integration, auto-materialize |
| **Apache Flink** | Stream processor | Streaming orchestration | Exactly-once state, event-time processing, unified compute+orchestration |
| **AWS Step Functions** | Serverless workflow | AWS-native architectures | Zero infra, built-in error handling, Lambda orchestration |
| **Temporal** | Durable execution | Long-running workflows, microservices | Automatic retry, compensation, saga patterns |

### Selection Guide

| Scenario | Choose |
|----------|--------|
| Existing Airflow knowledge | Airflow |
| Starting fresh, modern DX | Prefect or Dagster |
| Built-in data quality needed | Dagster |
| Heavy streaming focus | Flink |
| AWS-native, serverless | Step Functions |
| Saga/long-running workflows | Temporal |
| Zero-infrastructure | Managed services (Composer, MWAA) |

---

## 7. Pipeline Observability

**Three pillars:** Freshness (is data up to date?), Volume (row count in expected range?), Quality (null rates, uniqueness, distributions acceptable?).

| Tool | Category | Key Features |
|------|----------|-------------|
| **Great Expectations** | Data quality (OSS) | Expectations as code, Data Docs, profiling |
| **dbt Tests** | Data quality (built-in) | `not_null`, `unique`, `relationships`, freshness checks |
| **Soda Core** | Data quality (OSS) | YAML-based checks, cross-source validation, anomaly detection |
| **Monte Carlo** | Observability SaaS | Freshness, volume, schema, lineage-based impact |
| **Sifflet** | Observability SaaS | Column-level quality, lineage integration |
| **Databand (IBM)** | Observability SaaS | Pipeline monitoring, quality, cost tracking |
| **Prometheus + Grafana** | Infrastructure monitoring | Pipeline metrics (duration, rows, failures, resources) |
| **OpenLineage** | Lineage (open standard) | Column-level lineage, standard integration protocol |

### Alert Tiers

| Level | Trigger | Channel |
|-------|---------|---------|
| **Critical** | Pipeline failure, freshness SLA breach | PagerDuty, SMS, Slack @channel |
| **Warning** | >5% nulls on non-null column, volume drop >20% | Slack channel, email |
| **Info** | Schema change detected | Slack, ticket |
| **Anomaly** | Statistical outlier in distribution | Dashboard alert |

---

## 8. Pipeline Testing Strategies

| Test Type | What It Checks | How | Frequency |
|-----------|---------------|-----|-----------|
| **Schema tests** | Column presence, data types, nullable flags | dbt `schema.yml`, Great Expectations | Every run |
| **Data quality tests** | Nulls, uniqueness, referential integrity | dbt `not_null`, `unique`, `relationships` | Every run |
| **Freshness tests** | Data arrived on time | dbt `freshness` config | Every run |
| **Regression tests** | Row counts, aggregations match expected | Compare production vs test output | On model change |
| **Unit tests** | Individual transform logic | Python `pytest`, dbt test macros | On code change |
| **Integration tests** | Full pipeline with test subset | CI/CD with test environment | Every PR |
| **Performance tests** | Completes within SLA at scale | Production-scale data in staging | On major changes |
| **Contract tests** | Source/sink match expected schemas | JSON schema, Avro/Protobuf registry | Every deploy |

### Testing Principles

1. Test at boundaries — edges cases (nulls, duplicates, late data, schema changes).
2. Automate everything — no manual QA in deployment.
3. Data contracts first — agree on schemas with source owners before building.
4. Test in production-like environments — tiny test datasets miss scaling issues.
5. Make tests deterministic — avoid flaky tests based on `CURRENT_TIMESTAMP` or random sampling.

---

## 9. Common Pipeline Anti-Patterns

### 🔴 Orchestration as Transformation
Writing complex transforms in Airflow/Python operators instead of dbt/Spark/Flink. **Instead:** Airflow calls dbt/Spark/Flink — transformation logic lives in those tools.

### 🔴 One Pipeline to Rule Them All
Monolithic pipeline doing everything. **Instead:** Decompose into domain-specific pipelines (orders, payments, inventory).

### 🔴 No Data Quality Checks
Bad data propagates silently until business users report problems weeks later. **Instead:** Quality gates at every stage: source, staging, output.

### 🔴 Ignoring Backfills
No strategy for reprocessing historical data when logic changes. **Instead:** Design idempotent pipelines that support backfill by date range. Use dbt `--full-refresh` or Spark partition overwrites.

### 🔴 Manual Recovery
Pipeline failure requires engineer to diagnose and re-run manually. **Instead:** Automatic retries with exponential backoff, automated backfill on success, clear failure alerts.

### 🔴 No SLA Monitoring
No tracking of data freshness, volume, or quality. **Instead:** Define SLAs and alert on breaches. Track in a dashboard.

### 🔴 Tightly Coupled Sources
Pipeline assumes source schema never changes. **Instead:** Schema-on-read with column mapping layers. Validate schemas at pipeline start. Notify on schema drift.

### 🔴 Reinventing the Wheel
Building custom ingestion/transformation/orchestration in-house. **Instead:** Use dbt for transforms, Airflow/Prefect/Dagster for orchestration, Fivetran/Airbyte for ingestion.

### 🔴 Fire-and-Forget Pipelines
Build, deploy, never revisit unless it breaks. **Instead:** Treat pipelines as products — with owners, SLAs, documentation, regular health reviews.

---

## 10. Modern Data Stack (MDS) Reference

| Layer | Tools |
|-------|-------|
| **Ingestion** | Fivetran, Airbyte, Meltano, Debezium, Stitch |
| **Storage** | Snowflake, BigQuery, Redshift, Databricks, S3/ADLS/GCS |
| **Table Formats** | Apache Iceberg, Delta Lake, Apache Hudi |
| **Transformation** | dbt, Spark, Flink, SQLMesh, Materialize |
| **Orchestration** | Airflow, Prefect, Dagster, Temporal, Kestra |
| **Observability** | Great Expectations, Monte Carlo, Sifflet, Databand, Soda |
| **Reverse ETL** | Hightouch, Census, Grouparoo |
| **Catalog & Governance** | DataHub, Atlan, Collibra, Amundsen |
| **BI & Visualization** | Looker, Tableau, Metabase, Superset, Mode |
| **Feature Store** | Feast, Tecton, Hopsworks |
| **Stream Processing** | Flink, Kafka Streams, RisingWave, Materialize, ksqlDB |

### Medallion Architecture (Lakehouse)

```
Bronze (Raw) ──▶ Silver (Cleaned) ──▶ Gold (Aggregated)
  Immutable,        Deduplicated,       Business-level
  append-only,      validated,          aggregates,
  as-received       entity-enriched     BI-ready
```

- **Bronze:** Raw ingested data, append-only, never modified after landing.
- **Silver:** Deduplicated, validated, standardized, schema-enforced, business entities joined.
- **Gold:** Business-level aggregates, denormalized for BI, curated with clear ownership.

---

## 11. Key Design Decisions

### Batch vs Streaming Decision Matrix

| Factor | Batch | Streaming |
|--------|-------|-----------|
| **Data freshness** | Hours to days | Seconds to minutes |
| **Data volume** | Any (bounded) | Unbounded (continuous) |
| **Source latency tolerance** | Can wait for ready | Must ingest as events arrive |
| **Exactly-once semantics** | Easy (transactional) | Hard (state + watermarks) |
| **Development complexity** | Low-Medium | Medium-High |
| **Debugging** | Easy (reproducible) | Hard (stateful, temporal) |
| **Infrastructure cost** | Lower (spin up/down) | Higher (always-on compute + state) |
| **Talent availability** | High (SQL/Python) | Lower (specialized) |
| **Backfill** | Easy (re-run batch) | Medium (replay from event log) |

### Decision Flowchart

```
Required freshness?
├── Seconds–minutes → Streaming
│   ├── Simple transforms? → Kafka Streams / ksqlDB
│   └── Complex? → Flink / RisingWave
└── Hours–days → Batch
    ├── <500GB? → SQL + dbt on warehouse
    ├── 500GB–10TB? → Spark batch
    └── >10TB? → Distributed Spark + partitioning

Small team (<3 engineers)?
├── Yes → Managed services (Fivetran, dbt Cloud, Prefect Cloud)
└── No → Self-managed (Airflow + open source)
```

### Tool Selection Criteria

Evaluate tools across: maturity (community size, production track record), integration (source/target connectivity), operational overhead (SaaS vs self-managed), team skills (SQL/Python/Spark), cost model (compute, storage, license, egress), scalability (horizontal, volume limits), exactly-once support, schema evolution handling, built-in observability, backfill support.

---

## 12. Operational Best Practices

### Pipeline Checklist

**Before building:**
- [ ] Clear business objective with measurable outcomes
- [ ] Source access confirmed, schema documented, owners/POCs identified
- [ ] SLA requirements documented (freshness, completeness, uptime)
- [ ] Data contract agreed between producer and consumer

**During build:**
- [ ] Pipeline is **idempotent** — re-running produces the same result
- [ ] Incremental loading for large tables
- [ ] Quality checks at every stage (source, staging, output)
- [ ] Error handling with automatic retry + dead-letter queues
- [ ] Schema evolution handled (detect, log, adapt or reject)
- [ ] Backfill strategy designed and tested
- [ ] Monitoring and alerting configured
- [ ] Documentation written (purpose, owner, dependencies, runbook)

**Post-deployment:**
- [ ] Freshness SLAs tracked and verified
- [ ] Cost monitoring set up (compute, storage, transfer)
- [ ] Incident response runbook created
- [ ] Regular health reviews scheduled
- [ ] Pipeline owner(s) assigned
- [ ] Deprecation plan drafted

### Metrics to Track

| Metric | Definition | Target |
|--------|-----------|--------|
| **Data freshness** | Time between event occurrence and availability | Within SLA |
| **Completeness** | % of expected rows that arrived | >99.9% |
| **Uptime** | % of scheduled runs that succeeded | >99.5% |
| **MTTD** | Failure to alert time | <5 min |
| **MTTR** | Alert to recovery time | <30 min |
| **Quality pass rate** | % of quality checks passing | >99% |
| **On-time delivery** | % of runs completing before SLA deadline | >99% |

---

## References & Further Reading

**Books:**
- "Designing Data-Intensive Applications" — Martin Kleppmann
- "The Data Warehouse Toolkit" — Ralph Kimball
- "Streaming Systems" — Akidau, Chernyak, Lax
- "Fundamentals of Data Engineering" — Joe Reis, Matt Housley
- "Data Mesh" — Zhamak Dehghani

**Articles:**
- "Questioning the Lambda Architecture" — Jay Kreps (O'Reilly, 2014)
- "The Log: What Every Software Engineer Should Know About Real-Time Data's Unifying Abstraction" — Jay Kreps
- "Streaming 101: The World Beyond Batch" — Tyler Akidau
- "The Modern Data Stack: Past, Present, and Future" — Tristan Handy

**Documentation:**
- [Apache Airflow](https://airflow.apache.org/docs/)
- [Apache Flink](https://nightlies.apache.org/flink/flink-docs-stable/)
- [dbt](https://docs.getdbt.com/)
- [Apache Kafka](https://kafka.apache.org/documentation/)
- [Debezium](https://debezium.io/documentation/)
- [Great Expectations](https://docs.greatexpectations.io/)
- [Confluent Blog — Streaming Pipelines](https://www.confluent.io/blog/)

**Communities:**
- dataengineeringcentral.substack.com
- seattledataguy.substack.com
- startdataengineering.com
- dataengineer.io
- r/dataengineering

---

> **Final thought:** The best data pipeline is the one that reliably delivers trustworthy data on time, with clear ownership and simple operations. Favor simplicity over cleverness, choose the right tool for the job, and treat your pipelines as products — not projects.
