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

## 13. The Failure Playbook

§9 above is the **design** view — the anti-patterns not to build into a pipeline. §13 is the **operating** view: what to do when something breaks in production. They are different artefacts. A pipeline can be built exactly as §9 prescribes and still fail at 02:00 because a source changed, a credential expired, or a vendor quietly redefined a field. §9 asks *what not to build*; §13 asks *what to do once it is built and wrong*.

Every failure class below is written as **symptom → cause → detection → handling**. The field that earns this section its place is **detection** — how the operator learns the pipeline is wrong *before* a business user complains. The hardest class is the failure nobody detects at all; that is §13.7. Mechanism is described directly, and a tool is named only where its own documentation owns the mechanism. No failure rate, downtime percentage or cost-of-downtime figure appears in this section — where an illustration is needed it is a labelled hypothetical with its assumptions visible.

### 13.0 The symptom–detection quick map

| § | Failure class | First symptom an operator sees | Detection that does not wait for a complaint |
|---|---------------|-------------------------------|----------------------------------------------|
| 13.1 | Upstream source changes or disappears | Task fails on a missing column — or a column is silently null | Schema/contract validation plus source freshness check |
| 13.2 | Ingestion or authentication failure | 401/403, timeout, throttling — or a short result that looks complete | Row-count reconciliation against the source; cursor-exhaustion assertion |
| 13.3 | Transformation or logic failure | A metric moves while task status stays green | Per-stage row-count and distribution deltas vs a trailing baseline; regression aggregate |
| 13.4 | Late, duplicate or out-of-order data | Totals change after close; sums inflate | Lateness/completeness metrics; duplicate-key count before load |
| 13.5 | Orchestration or scheduling failure | A DAG stuck on a dependency; a job runs twice | Output provenance per task (rows written); run-id / partition uniqueness check |
| 13.6 | Load or sink failure | Constraint violation, partial batch, or a write that "succeeded" but never landed | Write-side count assertion plus read-back of the target |
| 13.7 | **Silent data-quality failure** | **None** — the data is complete, well-formed and wrong | Volume/distribution assertions, cross-source reconciliation, drift on key measures, business-rule assertions |
| 13.8 | Resource or infrastructure failure | OOM, shuffle spill, full disk, node loss, preemption | Memory, spill, disk and duration-percentile metrics vs a trailing baseline |
| 13.9 | Dependency failure (another team's pipeline) | Your job fails with a message that blames your job | Freshness check on the *source table* as the first task; a shared reconciliation control |

The rows are ordered by how hard the class is to detect, not by how often it occurs — no frequency claim is made. §13.7 is the last row and the first problem: it is the class where the pipeline's own status is green and the number is still wrong.

**Three levels of detection, in increasing order of strength.**

| Level | What it compares | What it catches | What it misses |
|-------|------------------|-----------------|----------------|
| **Assertion** (per run, local) | this run's output against an expected shape, range or rule | missing rows, unexpected nulls, out-of-range values, a broken business rule | a change that stays inside the expected range |
| **Reconciliation** (cross-source) | this pipeline's aggregate against an independent computation | missing or duplicated records, unit and meaning changes, a source that stops sending part of its population | a change the independent source shares |
| **Drift** (statistical, trailing) | this run against its own recent history | gradual shifts, threshold creep, a distribution that moves without any rule firing | an abrupt but in-range change, and a change present since the first run |

Most teams implement the first row and call the pipeline monitored. The second row is the cheapest strong control and the one §13.7 depends on; the third is the only one that catches a drift slow enough to be mistaken for business growth.

### 13.1 The upstream source changes or disappears

**Symptom.** A task fails with a column-not-found or type error after a source deploy. Or — worse — the run succeeds and a column that used to carry a value is now uniformly null. A file that used to arrive at a fixed time arrives late or not at all. A source system is decommissioned and its feed simply stops. A format shifts: CSV to JSON, a changed delimiter, a changed encoding, a date-format change inside the same column.

**Cause.** The source team changed a schema, renamed a column, changed nullability, or changed a format without notice to consumers. A scheduled file drop was missed. A feed was retired. None of these are pipeline bugs; they are contract violations or contract absences.

**Detection.** Two mechanisms, both cheap and both at ingestion rather than downstream: **schema validation** — compare the arriving columns, types and nullability against an expected contract and fail or flag on divergence — and a **freshness check** — assert that `max(event_time)`, or the arrival time of the expected file, is within its SLA. The detection gap to know about: if the pipeline is schema-on-read and does not validate, a renamed column frequently surfaces as a *null* rather than as an error, because the reader finds no matching field and fills nothing. A column that is null where it was never null is a schema-change signal, not a data-quality one. Freshness and schema tests are the mechanisms the testing table in §8 already lists; the operational discipline of asserting them on every run is what turns them from a test into an alarm.

**Detection mechanics.** Schema validation is a comparison between the arriving schema and the registered one — column list, types and nullability — where a removed or renamed column and a newly nullable column are findings rather than noise. Freshness is an assertion on a timestamp: `max(event_time)`, or the arrival time of the expected file, must fall inside the SLA window, evaluated every run and alerted at the same tier as a failure. Both run **at ingestion**, so the failure names the source instead of surfacing three models later as an empty table.

**Handling.** Fail fast rather than load a half-populated batch; quarantine the offending file or batch to a dead-letter location and keep the last good version in place; route the alert to the **source owner**, not only the platform team — a platform on-call cannot fix a contract they do not own. Then apply the schema-evolution discipline deliberately rather than adapting silently: `../schema_evolution_data_drift_guide.md` owns it (§4 schema-evolution approaches, §5 compatibility types, §9 schema drift in pipelines, §10 data contracts). The decision the owner of that guide forces is explicit adapt-or-reject, logged, rather than an accidental COALESCE that hides the change for a month.

### 13.2 Ingestion and authentication failures

**Symptom.** A hard 401/403 after a key rotation. A connection refused after an endpoint change or an API version deprecation. Throttling — 429s or a rate-limit error under a schedule that used to fit. And the nastiest variant: **a connection that succeeds and returns partial data that looks complete** — the request returns HTTP 200 with a truncated result set because the query timed out server-side, a page loop ended early, or a pagination cursor was not exhausted.

**Cause.** An expired credential or a rotated key the pipeline was not told about; a changed hostname, path or API version; a rate limit that a new backfill volume now exceeds; a partial read caused by a truncated result set, an unbounded query hitting a server-side timeout, or a pagination loop that stopped on a non-empty-but-final page.

**Detection.** Authentication and connection failures are loud — they fail fast, and the correct response is to let them. The partial read is the class that needs a real control: **reconcile the ingested row count against the source's own count** as an assertion rather than a log line. Where the source cannot be counted directly, assert **cursor exhaustion** (the last page carried the end-of-results marker, not merely an empty page) and compare each partition's count against its trailing median. A completeness ratio — arrived divided by expected — belongs on the same alert tier as a freshness breach (§7), because a partial load and a missing load feed the same wrong number.

**Handling.** Never promote a batch whose completeness assertion failed; mark the batch incomplete and re-pull from the last verified cursor rather than writing the short file. For throttling, back off with jitter and persist the cursor so the retry resumes instead of restarting. For credentials, rotate through a secret manager rather than editing a config value in place, and version the non-secret configuration as code — `data_pipeline_versioning.md` §6 covers configuration and secret versioning, and §7 the run metadata that tells you *which* credential and endpoint a given run actually used.

The signatures differ enough to warrant their own detectors:

| Signature | What it looks like | Detection that fits | Response |
|-----------|--------------------|--------------------|----------|
| Expired credential | 401/403, immediate and total | the failure itself — fail fast | rotate through the secret manager, re-run; escalate if it recurs on a schedule boundary |
| Changed endpoint or API version | a connection error, or a 404 on a path that used to work | the failure itself | pin the version where the API allows it; treat a silent fall-back to the newest version as a defect |
| Throttling | 429s, or a run that only succeeds when it is slow | rate-limit responses; run-duration trend | backoff with jitter, persist the cursor, move the schedule off the peak |
| Partial read that looks complete | HTTP 200, no error, fewer rows than the source holds | source row-count reconciliation; cursor-exhaustion assertion; per-partition count vs trailing median | mark the batch incomplete, re-pull from the last verified cursor, never promote it |

### 13.3 Transformation and logic failures

**Symptom.** A number moves and nobody changed the code: a metric halves, a join returns fewer rows than last run, a date lands a day off, a cohort disappears, a total drifts by a fixed proportion. The task status is green throughout.

**Cause.** A **bad join key** — a type mismatch between a string key and a numeric key, untrimmed whitespace, case or collation differences — that drops rows via an inner join or fans them out via a duplicate key. A **silently changing data type**: a numeric string that still parses, a decimal precision or scale change that rounds, an integer that became a float. A **timezone or locale assumption**: naive timestamps read as local in one stage and UTC in another, a DST boundary, a date truncation in the wrong zone, a thousands separator or decimal comma in a locale-formatted field. A **filter that starts excluding rows** because a status column gained a new value the `IN` list never had. A **NULL-handling change**: a NULL that now fails a join predicate, or a `COALESCE` default that starts hiding a real value — which is the same mechanism as §13.7's default-value failure arriving through the pipeline's own code.

The symptom shapes map onto causes closely enough to be worth memorising:

| Symptom shape | Most likely cause | Confirming check |
|---------------|-------------------|------------------|
| An inner join suddenly returns fewer rows | join-key type, collation or whitespace mismatch | compare key formats and match rates on both sides |
| Row count right, one measure wrong by a constant factor | unit, currency-scale or rounding change | reconcile the measure against an independent total (§13.7) |
| Dates land a day off, or a cohort shifts at midnight | timezone or DST assumption | truncate the same date in UTC and in local time and compare |
| A cohort disappears entirely | a filter over an enum that gained a value | list the distinct values of the filtered column |
| One segment's measure falls to zero | NULL handling in a join predicate or an aggregate | null-rate check on the key and on the measure's inputs |
| Rows multiply | a duplicate key on the right-hand side of the join | distinct-key count on the join key before the join |

**Detection.** Not task status. A green status answers "did the code run", not "did it compute the right thing". The controls are **row-count and distribution deltas per stage** against a trailing baseline — rows in vs rows out of each model, join hit rate, distinct-key count, per-column null rate and cardinality — and a **regression aggregate** compared against a known-good reference for the same period, which §8 already lists as a test type and which here runs as a production assertion. Lineage is what makes the delta actionable: when the counts diverge, you need to know which models are downstream before you know what to correct — `data_lineage_tools.md` (§3 collection approaches, §9 implementation patterns) owns that.

**Handling.** Isolate the stage that first diverged, correct the logic, and re-run the **affected date range**, not only the latest partition — a logic fix that touches history is a backfill, and the mechanism and its testing live in `backfill_data_engineering.md` (§5 scheduling, §7 backfill testing, §10 backfill for data-quality fixes). Then convert the incident into a permanent assertion so the next silent change fails a test instead of a report: `data_pipeline_versioning.md` §10 covers testing on versioned pipelines, and §2 covers what to version besides the code — the SQL, the config, and the expected output.

### 13.4 Late, duplicate and out-of-order data

**Symptom.** A total changes after a period is closed. A window's figure is restated. A sum is inflated because the same business event was counted twice. A record that belongs to yesterday arrives today and lands in no window at all.

**Detection.** Lateness and completeness metrics: on-time vs late arrival, and completeness against the expected population for a period.

**Handling.** This class has owners and this section does not re-derive them. Watermarks, allowed lateness, side outputs and warehouse restatement are `../late_arriving_data_guide.md` (§2–§4 streaming foundations and late-event handling, §5 corrections and restatement, §8 Kappa/Lambda design). Duplicate candidate keys, matching, survivorship and idempotent loading are `handling_duplicate_keys_data_warehousing.md` (matching techniques §4, survivorship §5, dedup and load patterns §7). Send the reader there; the failure playbook's only job for this class is to name it and to note that its detection is a metric, not an exception.

### 13.5 Orchestration and scheduling failures

**Symptom.** A DAG stops progressing on a dependency that never completes. A task reports success while writing nothing. A retry storm: a failing task retried on a fixed interval until the retries themselves become the outage. The scheduler runs the job twice. A backfill races the scheduled run and the two write the same partition. A single poisoned record fails one task and, by blocking the branch, halts unrelated downstream work.

**Cause.** A sensor waiting on a file that never arrives, or a task whose wrapper swallowed the exception and returned success. **Unbounded retries** with no backoff and no dead-letter destination. A scheduler that re-triggers on a delayed heartbeat, an operator who manually re-runs while the schedule also fires, or overlapping schedule intervals when a run takes longer than its period. A backfill launched against the same window the daily run is producing. A poisoned row with no side-output or task-level error isolation.

The signatures are distinct, and each has its own detector:

| Signature | What it looks like | Detection that fits |
|-----------|--------------------|---------------------|
| A dependency that never completes | a task waiting indefinitely on a sensor | a sensor timeout that fails explicitly — never an unbounded wait |
| A task that succeeds doing nothing | green status, zero rows written | output-provenance assertion per task |
| A retry storm | the same task failing and retrying on a fixed interval | a retry-count metric, alerted on retries and not only on the final state |
| The job runs twice | duplicate partitions, or counts that double | a run-id or partition-key uniqueness check |
| A backfill races the schedule | two writers on one partition, non-deterministic result | an overlap check between the backfill window and the live window |
| One poisoned task blocks everything | an unrelated branch halts | per-dataset freshness alerting, so a blocked branch is visible on its own |

**Detection.** Per-task **duration and last-success time** (a task that "succeeds while doing nothing" shows a suspiciously short duration), **output provenance** — assert each task's row output so zero rows written is an alert rather than a quiet success — and **duplicate-run detection** via a run-id or partition-key uniqueness check. Alert on "no new rows for this partition by time T" as a distinct signal from "a task failed", because the second is loud and the first is not. Watching per-task status alone is what misses exactly this family.

**Handling.** Make every task report an artefact it produced, so success-without-output is detectable. Cap retries with exponential backoff and route the exhausted failure to a dead-letter queue instead of retrying forever. Make runs **idempotent so a double-run is harmless** — the mechanism is the idempotent load in `handling_duplicate_keys_data_warehousing.md`, and `data_pipeline_versioning.md` §2 and §8 cover versioning and orchestrator-level controls. Isolate a poisoned task so it cannot block unrelated branches. Serialise backfill against the scheduled run with a separate window or an explicit lock — `backfill_data_engineering.md` §5 scheduling and orchestration, §6 idempotency, §8 risks.

### 13.6 Load and sink failures

**Symptom.** A constraint violation on write — null into a NOT NULL, a foreign key with no parent, a unique violation. A batch that committed halfway. A sink that accepted the write and dropped it somewhere downstream. A target table whose schema no longer matches what the writer produces.

**Cause.** Newly arriving data violating a written constraint; a multi-row write with no transaction boundary that failed part-way; an asynchronous or fire-and-forget sink with no acknowledgement, so the client's "success" means only that the request was accepted; DDL drift on the target table since the pipeline was built.

**Detection.** **Write-side assertion**: rows written equals rows read, and the target's row-count delta equals the expected delta. **Read-back verification**: read the target after the write and assert on the result, rather than trusting the write's return value — this is the only check that catches a sink which silently drops. And a schema check on the target before the load, so a mismatch fails at the boundary rather than corrupting the load.

Four checks, in order of strength:

- **Write-side assertion.** Rows read into the transformation versus rows written to the sink, and the target's row-count delta versus the expected delta, for the same batch.
- **Read-back verification.** After the write, read the target and assert on the values. This is the only check that catches an asynchronous sink that acknowledges and then drops.
- **Boundary schema check.** Validate the target's schema against the writer's expectation before the load, so a mismatch is a boundary failure rather than a corrupted batch.
- **Commit-boundary awareness.** Know whether the write is one transaction, one partition swap, or a stream of independent inserts — that determines whether a failure can leave a partial batch at all.

**Handling.** Wrap the load so a partial batch cannot land — a transaction, or an atomic partition overwrite/swap — and verify by reading back. The atomic-write mechanisms and the anti-patterns that defeat them are in `backfill_data_engineering.md` §6 (MERGE/UPSERT, INSERT OVERWRITE partition, deduplication, common anti-patterns). When the sink is a queue, the "accepts and drops" failure and its delivery-guarantee analysis are owned by `../message_queue_data_loss_guide.md` (§2 delivery semantics, §4 broker durability, §5 consumer commit discipline). The sink that accepts and drops is the write-side mirror of §13.2's partial read: in both, a successful API response is not evidence that the data arrived.

### 13.7 Silent data-quality failure — the headline class

**Symptom.** None. The data is complete, well-formed, passes every not-null and type check, and is wrong. A metric moves by an amount that looks like business variation. A report is confidently incorrect.

**Cause.** The **meaning** of a field changed while its shape did not. A vendor redefines a field; a currency or unit changes (minor units to major units, cents to dollars, thousands to ones); a default value starts being written where a real value used to be; a source begins sending placeholders — a literal `"N/A"`, a `0`, a `1900-01-01`, an empty string that parses as zero; an upstream filter halves the population the pipeline ever sees. The schema is the wrong contract for this class, because a schema constrains shape and says nothing about meaning.

**Detection — and this is the whole problem.** Name the mechanisms honestly, then admit their limit.

- **Volume and distribution assertions.** The §7 volume pillar, generalised: not only "did rows arrive" but "did the distribution of the key measures stay in range". A distribution shift on an amount, a rate or a duration column is detectable even when the row count is exactly right.
- **Cross-source reconciliation.** Compare an aggregate against an independent system of record — a ledger total against the warehouse total, an order count against a payments count, a control total published by the source. This is the strongest silent-failure detector precisely because it does not depend on the pipeline's own code being correct; it depends on two independent computations agreeing.
- **Statistical drift on key measures.** Compare each key measure against its trailing median (or a trailing distribution), and flag a shift that the business volume does not explain. The methods — distribution comparison, drift metrics and their thresholds — are owned by `../schema_evolution_data_drift_guide.md` §7, with drift handling in §8; they are not re-derived here.
- **A business-rule assertion on the result.** A rule stated in business language and evaluated as a test on the output: no negative settlement amount; every settled trade has a matching instruction; the sum of the parts equals the whole; a currency code is in the permitted set. Business rules catch semantic errors that statistical checks see only as noise.

**What to assert, by field shape.** Each field type has a cheap assertion that catches a meaning changed in place:

| Field shape | Assertion that catches a meaning change |
|-------------|----------------------------------------|
| Money or other measure | plausible range plus a trailing-median comparison; reconcile the period total against an independent source |
| Currency or unit code | membership in a permitted code set; a new code, or a shift in the mix of codes |
| Date or timestamp | plausible range — not a sentinel such as a zero date — plus density per day, and monotonicity where expected |
| Enumerated status | membership in the known value set; a value appearing, and a value disappearing |
| Identifier | format check plus a uniqueness or expected-cardinality check |
| Free text | length and character-set distribution; a spike in one repeated value signals a placeholder |
| Any column that used to be populated | non-null rate against its trailing norm — a rising frequency of the default value is a signal, not a data point |

**Segmented reconciliation beats a bare total.** A control total can agree while its composition is wrong; comparing the same aggregate per segment — per currency, per channel, per entity — makes a partial-population change visible even when the total is preserved.

**Be honest about the limit.** Some silent failures are found only when a consumer notices — when a human says "that number looks wrong". A semantic change that stays inside the observed distribution is invisible to every automated check above, by construction. The durable response is to convert the consumer's complaint into a permanent assertion and into a contract with the source, so that this specific failure is detectable next time even though the general class is not.

**Hypothetical (assumptions labelled).** Suppose a settlement feed's `amount` field has always been minor units (cents) and one release begins sending major units (dollars), with the field's name, type and nullability unchanged. Assume the pipeline asserts only non-null and numeric. The column stays numeric, non-null and plausible; every per-record transform succeeds. Only a **cross-source reconciliation** against the ledger, or a **distribution check** on the trailing median of `amount`, reveals the shift — and the median moves by the ratio between the units, so a volume-based threshold alone would not catch it. The assumptions are deliberately visible: change any one of them (a name change, a type change, an existing reconciliation control) and a different mechanism becomes the detector.

**Handling.** Freeze the affected output rather than let it propagate. Quantify the blast radius — which datasets, which dates, which measures, which consumers — and lineage is what answers it: `data_lineage_tools.md` §2 (operational use cases) and §9 (implementation patterns). Restate downstream. Then fix at the source with the semantic owner, and make the meaning explicit: a per-field definition and a data contract, `../schema_evolution_data_drift_guide.md` §10. Backfill the affected window to repair history — `backfill_data_engineering.md` §10 (backfill for data-quality fixes). Remember the split between **remedy** and **detect**: you cannot alert on a rule you never wrote, so the permanent fix is a new assertion plus a documented meaning, not only a corrected number.

### 13.8 Resource and infrastructure failures

**Symptom.** The job is killed out of memory. Runtime climbs and the shuffle spills to disk. A node's disk fills with logs or temp files and the write fails. A timeout fires under peak load that never fired before. A cluster loses a node mid-job. A spot instance is reclaimed.

**Cause.** Partition skew and key hotspots concentrating a few keys onto one task; a dataset that grew past a memory boundary the job was sized for; unbounded log retention or temp growth filling a disk; a fixed timeout tuned at low volume meeting peak volume; and, for preemption, the deliberate, expected reclaim of spot capacity.

**Detection.** Infrastructure metrics — memory, shuffle spill, disk usage, and task-duration percentiles — compared against a trailing baseline, so "this job now takes twice as long" is visible before it crosses its SLA. §7 lists the metric tier for this (pipeline duration, rows, failures, resources). On the streaming side, the corresponding signals are checkpoint and commit failures, which the streaming guides treat in depth.

**Detect it as a trend, not an event.** Task duration, spill volume, peak memory and disk usage tracked per run and compared with the same job's own recent history make a job that is drifting toward its limit visible while it still succeeds. A job that fails at the memory limit and a job whose duration is creeping toward its SLA deadline are the same failure at different points on a curve.

**Handling.** First, classify: a **correctness failure** needs a data fix; a **capacity failure** needs a resource fix. A retry that succeeds after a node loss is a capacity event — fix the sizing, do not chase a data bug. A reproducible OOM usually needs partition salting or a memory/serialisation change, not retries. Treat preemption as expected and design the job to checkpoint and resume, so a lost instance is a pause, not a restart from scratch — `../apache_flink_guide.md` §3 (state machinery and checkpoints), §4 (exactly-once and where the guarantee stops), §7 (deployment and operations) own the streaming mechanics, and the batch equivalent is the idempotent partition overwrite of `backfill_data_engineering.md` §6. One caution that connects to the rest of this section: before re-running after an infrastructure failure, establish whether the failed attempt wrote anything partial (§13.6); a crash mid-write is a load failure wearing a resource failure's clothes.

### 13.9 The dependency failure

**Symptom.** Your pipeline fails with a message that blames your pipeline. A join returns nothing; a quality check fires; a freshness check trips — but the root cause is an upstream team's pipeline that produced stale, empty or wrong data, or produced nothing at all.

**Cause.** There is no visibility across the team boundary. The upstream's failure and its SLA are not propagated to you, and your own checks fire on data the upstream never delivered, so the error surfaces at your stage and points at the wrong owner.

**Detection.** Cross-team detection is the hard part, and it should be framed as such. What helps: a **freshness check on the source table as the first task of your DAG**, so the failure message reads "upstream dataset X is stale since T" rather than "join returned zero rows"; a **reconciliation control** on the shared dataset (§13.7) that compares your view against an independent total; and consuming a published signal — a last-successful-load timestamp or a source SLA status — rather than inferring the upstream's health from the damage it does to you. Be honest: if the upstream publishes no signal, you are detecting the failure late, as it manifests in your own pipeline, and no amount of local cleverness fixes that. The remedy is a contract, not a check.

A failure message that names the cause costs nothing and saves a re-triage:

- the dataset it could not read, and the last-good timestamp it expected;
- the freshness or schema assertion that fired, in the assertion's own words;
- the stage that failed, so nobody hunts through the DAG;
- the owning team, where a contract has established one.

**Handling.** Communicate with the owner rather than silently re-running; make the failure message name the upstream dataset and its last-good timestamp so the next on-call person does not re-triage from your logs. Then agree the signal or contract that would let you detect it before your consumers do. The discipline that covers monitoring across the boundary is `dataops_guide.md` §7.6 (monitoring freshness, volume, schema) and §12 (common pitfalls).

### 13.10 Incident response: triage, blast radius, and the idempotent re-run

A data-pipeline incident is not a web-service incident. The service may be "up" — the pipeline ran, the job succeeded — while the data is wrong. That inverts the usual instincts: uptime is not the signal, and a green dashboard is not an all-clear.

**Triage — three questions, in order.** *What is broken?* The code, the schedule, or the data — the answer determines whether you are deploying, re-running, or investigating meaning. *What data is affected?* Which datasets, which dates or partitions, which measures. *What is still correct?* This third question is the one that gets skipped and matters most: identifying a known-good cut is what lets consumers keep working and what bounds the blast radius. Lineage (`data_lineage_tools.md` §2, §9) is the tool for answering the second and third at once.

**Triage checklist, in order.**

1. **Is the pipeline down, or is the data wrong?** A failed task is a scheduling problem; a successful task with wrong output is a data problem — the harder incident, and the one this section exists for.
2. **When did it start?** Bracket the last good output and the first bad one; that window is the candidate blast radius, and it is defined by data rather than by the alert time.
3. **What changed near that window?** A deploy, a source release, a key rotation, a schedule change, a backfill, a volume spike. Most incidents have a change attached; the absence of one is itself information.
4. **What is still correct?** Establish a known-good cut before communicating, so consumers have something to stand on.
5. **Who needs to know now?** Consumers of the affected data first, then the source owner if the cause is upstream, then the platform owner.

**Severity and blast radius.** Ask what is wrong, who consumes it, and what decision it feeds. A silently wrong figure feeding a regulatory return, a settlement or a payment is a different severity from a broken internal dashboard, even when the code failure is identical. Blast radius is measured in consumers and decisions, not in rows.

**What to communicate, and to whom.** To data consumers: what is affected and what to use instead. To the source owner, if the cause is upstream (§13.1, §13.9). To the platform on-call owner. State what is *known* and what is *not* — an honest "we have confirmed the period from the 3rd onward and are still checking the 1st and 2nd" is more useful than a guess, and it prevents a second wave of bad decisions taken on a number you have not cleared.

**Freeze or continue.** Decide whether to let the pipeline keep running with its outputs marked suspect, or to halt it. Continuing risks propagating wrong data further downstream and deepening the re-run; halting risks a growing backlog and stale consumers. The deciding factor is whether downstream consumers can *detect* a bad batch themselves: if they cannot, halt, because the alternative is silent propagation.

**Hotfix versus correct re-run.** A hotfix unblocks the schedule and leaves the range it ran over still wrong. The correct action is a re-run of the affected range after the fix. Do not let the hotfix's green status stand in for the correction — schedule the re-run explicitly, and close the incident only when the affected window has been reprocessed and reconciled.

**The re-run must be idempotent, or it becomes a second incident.** A re-run that inserts instead of upserting double-counts; one that overwrites a wider range than intended destroys newer data; one that races the scheduled run writes a partition twice (§13.5). The mechanisms are an idempotent load keyed on a deterministic business key — `handling_duplicate_keys_data_warehousing.md` — and the backfill discipline of `backfill_data_engineering.md` §6 (idempotency) and §7 (row-count reconciliation, checksum comparison). Reconcile the re-run's output against the pre-incident baseline before declaring it done; a re-run nobody verified is an assumption, not a fix.

**The post-incident action that prevents a repeat.** The action is a **detection**: the incident belongs to one of the classes above, and the fix is the assertion, alert or contract that would have caught it — a new quality check, a freshness assertion, a reconciliation control, a business-rule test, or a schema/meaning contract with a source. Record the runbook step that was missing, and add the incident to the regression set so the same silent change fails a test next time. `dataops_guide.md` §7.8 (post-deployment validation) and §12 (common pitfalls) own the discipline. The goal is not zero failures; it is shorter detection and a bounded blast radius — because the failures that hurt are the ones nobody saw.

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
