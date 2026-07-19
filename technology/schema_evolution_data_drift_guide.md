# Schema Evolution & Data Drift in Data Engineering: A Comprehensive Guide

> **Author:** Jack Liu Shurui | **Role:** Solution Architect, Crédit Agricole CIB
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Version:** 1.0 | **Last Updated:** July 2026

---

## 1. Introduction

Modern data platforms operate in constant change. Source systems evolve, business requirements shift, and regulatory mandates impose new reporting structures. Two categories of change—**schema evolution** and **data drift**—are among the most dangerous failure modes in data engineering. Left unmanaged, they silently corrupt pipelines, break dashboards, degrade ML models, and trigger regulatory reporting failures.

---

## 2. Schema Evolution vs Data Drift — Definitions

### 2.1 Schema Evolution

Schema evolution refers to **structural changes to a table or dataset's schema** — column definitions, data types, nesting, and constraints. These are deliberate, versioned, and ideally governed.

| Operation | Description | Example |
|-----------|-------------|---------|
| **Add column** | Introduce a new field | `ALTER TABLE txn ADD COLUMN channel STRING` |
| **Drop column** | Remove an existing field | `ALTER TABLE txn DROP COLUMN legacy_flag` |
| **Rename column** | Change a field name | `ALTER TABLE txn RENAME COLUMN cust_id TO customer_id` |
| **Change data type** | Widen or narrow a type | `INT → BIGINT`, `STRING → DATE` |
| **Restructure nested fields** | Add/drop/rename inside structs | `STRUCT<old:INT>` → `STRUCT<new:BIGINT>` |
| **Add/drop partitions** | Change partitioning scheme | Remove or add a partition column |

### 2.2 Data Drift

Data drift refers to **changes in the content, distribution, or semantics of data values** within a stable schema. Drift is often unplanned and emerges from upstream or real-world changes.

| Drift Type | Description | Example |
|------------|-------------|---------|
| **Distribution shift** | Statistical properties change | Age distribution shifts from 25–35 to 45–55 |
| **Novel values** | New categorical values appear | Country column suddenly includes `"SG"` |
| **Null rate shift** | Null % changes dramatically | `email` from 2% null to 80% null |
| **Format change** | Same field, new encoding | `phone` from `+65XXXXXXXX` to `65-XXXXXXXX` |
| **Semantic drift** | Same name, different meaning | `status` changes from `active/inactive` to `active/suspended/closed` |
| **Volume drift** | Record count anomalies | 1M rows/day → 100K or 10M unexpectedly |

### 2.3 Key Distinction

```
┌──────────────────────────────────────────────────┐
│                  CHANGE DETECTED                   │
├─────────────────┬────────────────────────────────┤
│ Schema Evolution │          Data Drift            │
├─────────────────┼────────────────────────────────┤
│ Structural       │  Content / distribution        │
│ Explicit         │  Implicit / emergent           │
│ Versioned        │  Continuous / sliding window    │
│ Controlled       │  Monitored                     │
│ Schema Registry  │  Statistical tests             │
│ ALTER TABLE      │  KL divergence, PSI, KS test   │
└─────────────────┴────────────────────────────────┘
```

---

## 3. Why It Matters

**Pipeline failures.** A new column added upstream breaks an ingestion job expecting a fixed schema. Renamed columns propagate as `NULL` downstream.

**Silent corruption.** More dangerous than crashes — pipelines that produce wrong data without errors. A column silently dropped or default-filled corrupts dependent datasets with no signal.

**Data quality degradation.** Aggregates produce incorrect values — wrong sums, counts, averages — because the expected schema no longer matches actual data.

**Regulatory reporting breaks.** BCBS 239 mandates traceability from reported figures to source transactions. Schema changes that break lineage can produce incorrect regulatory filings with severe consequences (capital add-ons, fines).

**ML model decay.** ML models are hypersensitive to both schema and data drift. Feature drift alone can reduce model accuracy from 95% to 60% without any code change:

```
Model AUC
1.0 │                    ●──●──●──●──●──●──●
    │                  ●                    ●
0.8 │                ●                       ●──●  Drift starts
    │               ●                              ●──●
0.6 │              ●                                     ●──●
    │             ●                                            ●──●
0.4 │            ●                                                   ●
    └──────────────────────────────────────────────────────────────► Time
```

---

## 4. Schema Evolution Approaches

### 4.1 Schema-on-Read vs Schema-on-Write vs Schema-on-Query

| Approach | Description | Pros | Cons | Typical Use |
|----------|-------------|------|------|-------------|
| **Schema-on-Read** | Schema applied at read time | Flexible; no ingestion failures | Schema mismatch detected too late | Data lakes, raw JSON/Parquet |
| **Schema-on-Write** | Schema validated at write time | Fail fast; predictable output | Inflexible; breaks on upstream changes | Schema Registry, OLTP |
| **Schema-on-Query** | Schema defined by query engine | Zero storage migration | Query-time failures; governance blind spot | Trino/Presto views |

**Reality:** Most platforms use a hybrid — schema-on-write at ingestion boundaries with schema-on-read flexibility in exploration zones.

### 4.2 Schema Registry

A schema registry stores, versions, and validates schemas across a platform. Core capabilities: schema storage with versioning, compatibility checking, serialization/deserialization, and evolution governance.

| Registry | Formats | Native Platforms |
|----------|---------|-----------------|
| **Confluent Schema Registry** | Avro, Protobuf, JSON Schema | Kafka / Confluent |
| **AWS Glue Schema Registry** | Avro, JSON Schema | AWS Kafka, Kinesis |
| **Apicurio Registry** | Avro, Protobuf, JSON, AsyncAPI | Kafka, multi-cloud |

### 4.3 Apache Avro Schema Evolution

Avro's approach is unique: **both writer and reader schemas are present at deserialization**. Fields are matched by **name**, not position. A field in the reader but not the writer is filled from a default. A field in the writer but not the reader is ignored. Types can be promoted (int → long → float → double).

### 4.4 Apache Iceberg Schema Evolution

Iceberg implements schema evolution as **metadata-only** — no data files are rewritten:

| Operation | Metadata Cost | Notes |
|-----------|--------------|-------|
| Add column | ~zero | Existing rows get null for new column |
| Drop column | ~zero | Soft delete; data files still contain it |
| Rename column | ~zero | Metadata pointer update |
| Widen type | ~zero | Types: int→long, float→double, decimal precision increase |
| Add nested field | ~zero | Inside struct types |

**Blocked:** Dropping columns referenced by a partition transform, changing map key types, narrowing types.

### 4.5 Delta Lake Schema Evolution

Delta Lake supports schema evolution through write-time options:

1. **`mergeSchema`** — auto-merge: new columns in incoming DataFrame are automatically added
2. **`overwriteSchema`** — replaces entire table schema (destructive)
3. **Explicit ALTER TABLE** — standard SQL column changes

**Trade-off:** `mergeSchema` is convenient but dangerous in production — silently adds unintended columns.

### 4.6 Protobuf & JSON Schema Evolution Rules

**Protobuf:** Never reuse field numbers; use `reserved` for retired numbers/names; adding fields with defaults is backward-compatible; type changes break wire compatibility.

**JSON Schema:** `additionalProperties: false` enables strict validation; adding to `required` breaks backward compatibility; adding enum values is safe; removing values breaks.

---

## 5. Schema Evolution Compatibility Types

| Mode | Definition | Allowed | Blocked | When to Use |
|------|-----------|---------|---------|-------------|
| **BACKWARD** | New data readable by old consumers | Add field w/ default, remove field w/ default | Add required field, remove required field | Update consumers first, then producers |
| **FORWARD** | Old data readable by new consumers | Remove field w/ default, add new field | Add required field, remove required field from reader | Update producers first |
| **FULL** | Both backward + forward | Only add fields with defaults | Remove any field, change types | Zero-downtime any-order upgrades |
| **NONE** | No checking | Anything | Nothing | Dev/exploration only |

**Confluent adds transitive variants:** `BACKWARD_TRANSITIVE`, `FORWARD_TRANSITIVE`, `FULL_TRANSITIVE` — check against ALL prior schemas, not just the latest. Essential for regulated environments.

### Backward Compatibility Example

```json
// V1: Original schema
{"name": "txn_id", "type": "long"},
{"name": "amount", "type": "double"},
{"name": "currency", "type": "string"}

// V2: Add 'channel' with default (BACKWARD compatible)
{"name": "txn_id", "type": "long"},
{"name": "amount", "type": "double"},
{"name": "currency", "type": "string"},
{"name": "channel", "type": ["null", "string"], "default": null}
```

---

## 6. Schema Evolution in Practice — Code Examples

### 6.1 Apache Iceberg

```sql
-- Create partitioned Iceberg table
CREATE TABLE iceberg_catalog.transactions (
    txn_id     BIGINT,
    txn_date   DATE,
    amount     DECIMAL(18,2),
    currency   STRING,
    status     STRING,
    customer   STRUCT<cust_id: STRING, name: STRING, email: STRING>
) USING iceberg PARTITIONED BY (bucket(16, txn_id));

-- Add column
ALTER TABLE iceberg_catalog.transactions
ADD COLUMN channel STRING;

-- Add column with positional control
ALTER TABLE iceberg_catalog.transactions
ADD COLUMN fee_amount DECIMAL(18,2) AFTER amount;

-- Drop column (metadata-only soft delete)
ALTER TABLE iceberg_catalog.transactions
DROP COLUMN legacy_reference;

-- Drop nested field
ALTER TABLE iceberg_catalog.transactions
DROP COLUMN customer.email;

-- Rename column
ALTER TABLE iceberg_catalog.transactions
RENAME COLUMN currency TO settlement_currency;

-- Widen type
ALTER TABLE iceberg_catalog.transactions
ALTER COLUMN amount TYPE DECIMAL(22,6);

-- Evolve nested struct
ALTER TABLE iceberg_catalog.transactions
ADD COLUMN customer.phone STRING;

ALTER TABLE iceberg_catalog.transactions
RENAME COLUMN customer.name TO customer.full_name;
```

**PySpark — physically remove dropped PII columns:**

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .config("spark.sql.extensions",
            "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions") \
    .config("spark.sql.catalog.iceberg_catalog",
            "org.apache.iceberg.spark.SparkCatalog") \
    .config("spark.sql.catalog.iceberg_catalog.type", "hadoop") \
    .config("spark.sql.catalog.iceberg_catalog.warehouse",
            "s3://data-warehouse/") \
    .getOrCreate()

# Physically rewrite files to remove dropped columns
spark.sql("REWRITE DATA FILES FOR iceberg_catalog.transactions")
```

### 6.2 Avro — Writer/Reader Schemas

```python
# avro_schema_evolution.py
import fastavro

# Writer schema — V2: adds 'channel' with default
writer_schema = {
    "type": "record",
    "name": "Transaction",
    "namespace": "com.bank.risk",
    "fields": [
        {"name": "txn_id",   "type": "long"},
        {"name": "amount",   "type": "double"},
        {"name": "currency", "type": "string"},
        {"name": "channel",  "type": ["null", "string"], "default": None}
    ]
}

# Reader schema — V1: no 'channel' field
reader_schema = {
    "type": "record",
    "name": "Transaction",
    "namespace": "com.bank.risk",
    "fields": [
        {"name": "txn_id",   "type": "long"},
        {"name": "amount",   "type": "double"},
        {"name": "currency", "type": "string"}
    ]
}

# Write data with V2 schema
records = [
    {"txn_id": 1001, "amount": 150.00, "currency": "SGD", "channel": "ONLINE"},
    {"txn_id": 1002, "amount": 250.50, "currency": "USD", "channel": None},
]

with open("/tmp/transactions.avro", "wb") as f:
    fastavro.writer(f, writer_schema, records)

# Read with V1 schema (old consumer) — 'channel' is silently ignored
with open("/tmp/transactions.avro", "rb") as f:
    for record in fastavro.reader(f, reader_schema=reader_schema):
        print(record)  # {"txn_id": 1001, "amount": 150.0, "currency": "SGD"}
```

### 6.3 Avro — Rename with Alias

```json
// V1: field "channel"
{"name": "channel", "type": ["null", "string"], "default": null}

// V2: renamed to "origination_channel" with alias (backward compatible)
{"name": "origination_channel", "type": ["null", "string"],
 "default": null, "aliases": ["channel"]}
```

The `aliases: ["channel"]` tells Avro's resolution: "this field used to be called `channel` — map old data to this new name."

### 6.4 Protobuf — Field Number Discipline

```protobuf
syntax = "proto3";
package com.bank.risk;

message Transaction {
  int64  txn_id    = 1;
  double amount    = 2;
  string currency  = 3;
  int64  timestamp = 4;
  string channel   = 5;          // New field (never used before)
  reserved 6, 7;                  // Reserve future numbers
  reserved "legacy_field", "obsolete_code";
}
```

### 6.5 Delta Lake — mergeSchema and overwriteSchema

```python
from pyspark.sql import SparkSession
from pyspark.sql.types import (
    StructType, StructField, StringType, DoubleType
)

spark = SparkSession.builder \
    .appName("delta-schema-evolution") \
    .config("spark.sql.extensions",
            "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog",
            "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
    .getOrCreate()

# Initial write
data_v1 = [("TXN001", 150.00, "SGD")]
schema_v1 = StructType([
    StructField("txn_id", StringType()),
    StructField("amount", DoubleType()),
    StructField("currency", StringType()),
])
df_v1 = spark.createDataFrame(data_v1, schema_v1)
df_v1.write.format("delta").save("/mnt/delta/transactions")

# Evolve: new data with extra column
data_v2 = [("TXN002", 250.00, "USD", "ONLINE")]
schema_v2 = StructType([
    StructField("txn_id", StringType()),
    StructField("amount", DoubleType()),
    StructField("currency", StringType()),
    StructField("channel", StringType()),
])
df_v2 = spark.createDataFrame(data_v2, schema_v2)

# Safe evolution: mergeSchema adds 'channel' to table
df_v2.write.format("delta") \
    .mode("append") \
    .option("mergeSchema", "true") \
    .save("/mnt/delta/transactions")
```

### 6.6 Confluent Schema Registry — Compatibility Check

```python
# confluent_schema_registry.py
import requests
import json

SCHEMA_REGISTRY_URL = "https://schema-registry.internal:8081"
SUBJECT = "transactions-value"
HEADERS = {"Content-Type": "application/vnd.schemaregistry.v1+json"}

# Schema V2 to register
schema_v2 = json.dumps({
    "type": "record",
    "name": "Transaction",
    "namespace": "com.bank.risk",
    "fields": [
        {"name": "txn_id",   "type": "long"},
        {"name": "amount",   "type": "double"},
        {"name": "currency", "type": "string"},
        {"name": "channel",  "type": ["null", "string"], "default": None}
    ]
})

# Step 1: Test compatibility against latest schema
compat_resp = requests.post(
    f"{SCHEMA_REGISTRY_URL}/compatibility/subjects/{SUBJECT}/versions/latest",
    headers=HEADERS,
    data=json.dumps({"schema": schema_v2})
)
compat_result = compat_resp.json()
print(f"Backward compatible: {compat_result['is_compatible']}")

if not compat_result["is_compatible"]:
    raise RuntimeError("Schema is NOT backward compatible — deployment blocked!")

# Step 2: Register the new schema version
register_resp = requests.post(
    f"{SCHEMA_REGISTRY_URL}/subjects/{SUBJECT}/versions",
    headers=HEADERS,
    data=json.dumps({"schema": schema_v2})
)
schema_id = register_resp.json()["id"]
print(f"Registered schema ID: {schema_id}")

# Step 3: List all schema versions (audit trail)
versions = requests.get(
    f"{SCHEMA_REGISTRY_URL}/subjects/{SUBJECT}/versions"
)
print(f"All versions: {versions.json()}")
```

---

## 7. Data Drift Detection

### 7.1 Statistical Methods

**Population Stability Index (PSI)** — the most widely used metric in financial services:

```
PSI = Σ (P_i - Q_i) × ln(P_i / Q_i)

P_i = proportion of reference data in bin i
Q_i = proportion of current data in bin i
```

| PSI Value | Interpretation |
|-----------|---------------|
| < 0.10 | No significant drift |
| 0.10 – 0.25 | Moderate drift — investigate |
| > 0.25 | Severe drift — action required |

**Kullback-Leibler (KL) Divergence:** `KL(P||Q) = Σ P(x) × log(P(x)/Q(x))` — asymmetric, good default for large datasets.

**Jensen-Shannon (JS) Divergence:** Symmetric, bounded [0, 1]. Square root is a true distance metric.

**Kolmogorov-Smirnov (KS) Test:** `KS = sup_x |F_P(x) - F_Q(x)|` — non-parametric, good for continuous features.

**Chi-Square Test:** Best for categorical features — tests observed vs expected category frequencies.

### 7.2 Feature Drift vs Prediction Drift vs Label Drift

```
┌──────────────────┬──────────────────┬───────────────────────┐
│  Feature Drift   │ Prediction Drift │    Label Drift        │
├──────────────────┼──────────────────┼───────────────────────┤
│ Input X changes  │ Output ŷ changes │ Ground truth y changes│
│ Age distribution │ Score distribution│ Default rate changes  │
│ Amount range     │ Class balance     │ Churn rate changes    │
├──────────────────┴──────────────────┴───────────────────────┤
│ Feature → prediction drift (causal chain)                    │
│ Prediction drift w/o feature drift → concept drift           │
│ Label drift requires ground truth (delayed feedback)         │
└─────────────────────────────────────────────────────────────┘
```

Feature drift is the **leading indicator** (earliest warning). Prediction drift is easist to monitor (no ground truth). Label drift requires delayed ground truth data.

### 7.3 Drift Detection Tools

| Tool | Focus | Key Strengths | Licensing |
|------|-------|---------------|-----------|
| **Evidently AI** | ML monitoring | Rich stat tests, interactive dashboards, Python API | OSS free |
| **Great Expectations** | Data quality & schema | Expectations framework, CI/CD integration | OSS free |
| **Soda** | Data quality | Declarative checks, cross-platform, SQL-based | OSS free |
| **WhyLabs** | ML observability | Automated profiling, pre-built monitors | Free + paid |
| **Arize AI** | ML observability | Deep model insights, embedding drift, real-time traces | Free + paid |
| **Monte Carlo** | Data observability | End-to-end lineage, SLAs, incident management | Enterprise |
| **Alibi Detect** | OSS drift | State-of-the-art (MMD, CVM, LLR) | OSS free |

### 7.4 Evidently AI — Drift Detection Example

```python
# evidently_drift_detection.py
import pandas as pd
import numpy as np
from sklearn.datasets import make_classification

from evidently.metrics import DataDriftTable
from evidently.report import Report
from evidently.test_suite import TestSuite
from evidently.tests import TestNumberOfDriftedFeatures

# Generate reference data (production baseline)
X_ref, y_ref = make_classification(
    n_samples=10000, n_features=5, random_state=42
)
reference_df = pd.DataFrame(X_ref, columns=[f"feat_{i}" for i in range(5)])
reference_df["target"] = y_ref

# Generate current data with distribution shift
X_cur, y_cur = make_classification(
    n_samples=10000, n_features=5,
    shift=0.3,  # Introduce moderate drift
    random_state=99
)
current_df = pd.DataFrame(X_cur, columns=[f"feat_{i}" for i in range(5)])
current_df["target"] = y_cur

# Method 1: Drift Report (visual/analytical)
report = Report(metrics=[
    DataDriftTable(stattest='psi', stattest_threshold=0.1)
])
report.run(reference_data=reference_df, current_data=current_df)
report.save_html("/tmp/drift_report.html")

# Method 2: Drift Test Suite (assert-based for CI/CD)
suite = TestSuite(tests=[
    TestNumberOfDriftedFeatures(
        stattest='psi',
        stattest_threshold=0.15
    )
])
suite.run(reference_data=reference_df, current_data=current_df)

# Assert in CI/CD pipeline
assert suite.as_dict()["summary"]["all_passed"], \
    f"Drift test failed: {suite.as_dict()['summary']['failed_tests']} tests failed"

print("All drift tests passed")
print(f"Drifted features: "
      f"{suite.as_dict()['summary']['total_tests'] - suite.as_dict()['summary']['passed_tests']}")
```

### 7.5 Real-Time Monitoring with Prometheus/Grafana

```python
# prometheus_drift_exporter.py
from prometheus_client import start_http_server, Gauge
from evidently.metrics import DataDriftTable
from evidently.report import Report

feature_drift_gauge = Gauge(
    'feature_drift_psi',
    'PSI drift score per feature',
    ['feature_name']
)
overall_drift = Gauge('overall_drift_share', 'Proportion of drifted features')

def export_drift(reference_df, current_df):
    report = Report(metrics=[
        DataDriftTable(stattest='psi', stattest_threshold=0.1)
    ])
    report.run(reference_data=reference_df, current_data=current_df)
    result = report.as_dict()
    drift_cols = result['metrics'][0]['result']['drift_by_columns']
    n_drifted = 0
    for name, stats in drift_cols.items():
        psi = stats.get('stattest_value', 0.0)
        feature_drift_gauge.labels(feature_name=name).set(psi)
        if stats.get('drift_detected', False):
            n_drifted += 1
    overall_drift.set(n_drifted / max(len(drift_cols), 1))

start_http_server(8000)
# Schedule: call export_drift() every batch/stream window
```

---

## 8. Handling Data Drift

### 8.1 Automated Retraining Triggers

```python
def should_retrain(reference_df, current_df, threshold=0.25):
    report = Report(metrics=[DataDriftTable(stattest='psi', stattest_threshold=0.1)])
    report.run(reference_data=reference_df, current_data=current_df)
    drift = report.as_dict()['metrics'][0]['result']['drift_by_columns']
    max_psi = max((s.get('stattest_value', 0.0) for s in drift.values()), default=0.0)
    return max_psi > threshold
```

### 8.2 Alerting Thresholds

| Metric | Threshold | Severity | Response |
|--------|-----------|----------|----------|
| PSI (any feature) | > 0.25 | CRITICAL | Page, block downstream pipelines |
| PSI (any feature) | 0.10 – 0.25 | WARNING | Slack alert, create JIRA |
| Null rate shift | > 5% | WARNING | Investigate upstream source |
| Missing column | Any | CRITICAL | Fail pipeline immediately |
| Novel category | Any | INFO | Log and monitor; auto-add if from trusted source |

### 8.3 Fallback Strategies

1. **Fallback to simpler model:** Switch from complex ensemble to robust baseline (e.g., logistic regression)
2. **Ensemble with stale model:** Blend current predictions with frozen reference, weighting toward reference as drift increases
3. **Reject option:** Route high-uncertainty predictions to human review
4. **Rule override:** Apply business rules that override model predictions when drift is high

### 8.4 Feature Store for Consistency

A feature store decouples feature computation from training, providing a single source of truth. Benefits: feature registry with versioning, point-in-time correctness, automatic drift monitoring per feature, and validation of transformations. Tools: Feast, Tecton, Databricks Feature Store.

---

## 9. Schema Drift Handling in Pipelines

### 9.1 Enforce vs Evolve vs Ignore

| Strategy | Behaviour | When to Use |
|----------|-----------|-------------|
| **Enforce** | Pipeline fails on schema mismatch | Regulated tables; strict SLAs |
| **Evolve** | Pipeline adapts schema automatically | Trusted sources; low-regret datasets |
| **Ignore** | Unknown columns silently dropped | High-volume streaming; enrichment |

```python
# PySpark — three modes
from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()
df = spark.read.parquet("/mnt/raw/landing/")

# ENFORCE (default — no mergeSchema)
df.write.format("delta").mode("append").save("/mnt/delta/transactions")

# EVOLVE
df.write.format("delta").mode("append")\
    .option("mergeSchema", "true").save("/mnt/delta/transactions")

# IGNORE — select only known columns
known = set(spark.table("transactions").columns)
df.select(*[c for c in df.columns if c in known])\
    .write.format("delta").mode("append").save("/mnt/delta/transactions")
```

### 9.2 dbt Contracts

```yaml
# models/risk/transactions.yml
models:
  - name: transactions_enriched
    config:
      contract:
        enforced: true   # Pipeline fails if contract violated
    columns:
      - name: txn_id
        data_type: bigint
        constraints:
          - type: not_null
      - name: amount
        data_type: decimal(18,2)
      - name: currency
        data_type: varchar(3)
      - name: channel
        data_type: varchar(20)
```

### 9.3 Great Expectations — Schema Validation

```python
# ge_schema_validation.py
import great_expectations as gx
import pandas as pd

context = gx.get_context(mode="ephemeral")
suite = context.add_expectation_suite("transactions_schema")

suite.add_expectation(
    gx.expectations.ExpectTableColumnsToMatchSet(
        column_set=["txn_id", "amount", "currency", "channel"],
        exact_match=True
    ))
suite.add_expectation(
    gx.expectations.ExpectColumnTypeToBe(column="txn_id", type_="long"))
suite.add_expectation(
    gx.expectations.ExpectColumnTypeToBe(column="amount", type_="double"))
suite.add_expectation(
    gx.expectations.ExpectColumnValuesToNotBeNull(column="txn_id", mostly=1.0))
suite.add_expectation(
    gx.expectations.ExpectColumnValuesToBeInSet(
        column="channel",
        value_set=["ONLINE", "BRANCH", "ATM", "MOBILE"]))

# Validate batch
df = pd.read_parquet("s3://data-warehouse/transactions/latest_batch.parquet")
validator = context.sources.pandas_default.read_dataframe(df)
validator.expectation_suite = suite
results = validator.validate()
assert results["success"], f"Schema validation failed"
```

### 9.4 Schema Validation in CI/CD

```yaml
# .github/workflows/schema-validation.yml
on:
  pull_request:
    paths: ['schemas/**', 'models/**']
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Check Avro compatibility
        run: |
          curl --fail -X POST \
            -H "Content-Type: application/vnd.schemaregistry.v1+json" \
            --data "{\"schema\": \"$(cat schemas/transactions-v3.avsc | jq -Rs .)\"}" \
            "https://schema-registry:8081/compatibility/subjects/transactions-value/versions/latest"
      - name: Validate dbt contracts
        run: dbt compile && dbt test --select transactions_enriched
      - name: Run GE schema checks
        run: python ci/ge_schema_validation.py
```

---

## 10. Data Contracts as a Solution

### 10.1 What Are Data Contracts?

A **data contract** is a formal, versioned agreement between a data producer and consumer defining:

- **Schema:** Field names, types, constraints, nesting
- **Semantics:** Business meaning, allowed values, encoding rules
- **SLAs:** Freshness (max age), completeness (min %), volume (expected row count)
- **Ownership:** Producer team, consumer teams, escalation contacts
- **Change policy:** Notice period, deprecation windows, approval gates

```
┌─────────────────────────────────────────────────────────────┐
│                     DATA CONTRACT                            │
├─────────────────────────────────────────────────────────────┤
│ Schema:    txn_id (BIGINT, NOT NULL)                         │
│            amount (DECIMAL(18,2), NOT NULL)                   │
│            channel (VARCHAR(20), ENUM [ONLINE,...])           │
│ SLA:       Freshness ≤15min (99.9%), Completeness ≥99.5%    │
│ Owner:     Core Payments Team (#payments-prod)               │
│ Consumers: AML Team, Regulatory Reporting Team               │
│ Change:    Additive → immediate; Breaking → 30-day notice    │
└─────────────────────────────────────────────────────────────┘
```

### 10.2 Open Data Contract Standard (ODCS)

```yaml
# data_contract.yaml
data_contract: 1.0
dataset:
  name: "risk.transactions"
  version: "2.3.0"
producer:
  team: "Core Payments"
  contact: "#payments-prod"
consumers:
  - team: "AML Investigations"
    sla: {freshness: "15m", completeness: 0.995}
schema:
  - name: txn_id
    type: bigint
    required: true
  - name: amount
    type: decimal(18,2)
    required: true
    constraints: [{min: 0.01}]
  - name: currency
    type: varchar(3)
    required: true
    enum: [SGD, USD, EUR, GBP]
  - name: channel
    type: varchar(20)
    required: false
    enum: [ONLINE, BRANCH, ATM, MOBILE]
quality:
  - expectation: "no_null_txn_id"
    type: not_null
    column: txn_id
    severity: critical
change_management:
  notice_period_days: 30
  approval_required: true
```

### 10.3 Tools for Data Contracts

- **Open Data Contract Standard:** YAML spec; growing tooling ecosystem
- **Apache Avro + Schema Registry:** Schema-as-contract with compatibility enforcement
- **dbt Contracts:** SQL-level contract enforcement at build time
- **Great Expectations:** Runtime schema and quality contract validation
- **Soda:** Declarative contract enforcement via checks YAML

---

## 11. Testing for Schema and Data Drift

### 11.1 Unit Tests for Drift

```python
# tests/test_data_drift.py
import pytest
import pandas as pd
from scipy.stats import ks_2samp
from evidently.test_suite import TestSuite
from evidently.tests import TestNumberOfDriftedFeatures


class TestDriftDetection:
    @pytest.fixture
    def ref_df(self):
        return pd.read_parquet("tests/fixtures/reference.parquet")

    @pytest.fixture
    def cur_df(self):
        return pd.read_parquet("tests/fixtures/current.parquet")

    def test_amount_distribution(self, ref_df, cur_df):
        stat, p = ks_2samp(ref_df["amount"], cur_df["amount"])
        assert p > 0.05, f"Amount drifted (KS={stat:.4f}, p={p:.6f})"

    def test_null_rates_stable(self, ref_df, cur_df):
        shifts = (ref_df.isnull().mean() - cur_df.isnull().mean()).abs()
        assert (shifts < 0.05).all(), f"Null rate shifts: {shifts}"

    def test_evidently_drift(self, ref_df, cur_df):
        suite = TestSuite([TestNumberOfDriftedFeatures(stattest='psi')])
        suite.run(reference_data=ref_df, current_data=cur_df)
        assert suite.as_dict()["summary"]["all_passed"]
```

### 11.2 Contract Tests

```python
def test_contract_schema():
    contract = yaml.safe_load(open("data_contract.yaml"))
    df = pd.read_parquet("s3://warehouse/transactions/latest/")
    contract_cols = {s["name"] for s in contract["schema"]}
    actual_cols = set(df.columns)
    assert contract_cols == actual_cols, \
        f"Missing: {contract_cols - actual_cols}, Extra: {actual_cols - contract_cols}"
```

---

## 12. Monitoring and Observability

### 12.1 Schema Registry Audit Trail

Every schema registration, update, and compatibility check must be logged to an immutable audit trail — especially in regulated environments where schema version history must be retained for 7+ years.

### 12.2 Lineage Tracking for Impact Analysis

When a schema changes, lineage graphs enable rapid impact analysis:

```sql
-- DataHub / OpenMetadata lineage (conceptual)
WITH impacted AS (
    SELECT downstream_urn FROM lineage_edges
    WHERE upstream_urn = 'urn:table:risk.transactions'
    AND depth <= 5
)
SELECT asset_name, asset_type, owner FROM assets
WHERE urn IN (SELECT downstream_urn FROM impacted);
```

**Key questions:** Which dashboards consume this column? Which ML models use this feature? Which regulatory reports depend on this table?

### 12.3 Prometheus Alerting Rules

```yaml
groups:
  - name: data_drift
    rules:
      - alert: FeatureDriftHigh
        expr: feature_drift_psi > 0.25
        for: 2h
        labels: {severity: critical}
        annotations:
          summary: "Feature {{ $labels.feature_name }} PSI is {{ $value }}"

      - alert: FeatureDriftWarning
        expr: feature_drift_psi > 0.10
        for: 4h
        labels: {severity: warning}
```

---

## 13. Banking-Specific Considerations (BCBS 239)

### 13.1 BCBS 239 Overview

BCBS 239 (Principles for Effective Risk Data Aggregation and Risk Reporting) applies to G-SIBs. Key principles relevant to schema evolution:

- **Principle 2:** Data lineage must survive schema changes — every figure in a risk report must be traceable to its source transaction
- **Principle 3:** Risk data must be accurate, complete, and timely — schema drift that introduces nulls or defaults can break accuracy
- **Principle 6:** Adaptability — banks must adapt to new regulatory requirements, which means schema evolution is inevitable and must be controlled

### 13.2 Schema Evolution in a Regulated Environment

```
BCBS 239 Rule: "Data lineage must survive schema changes."

Implications:
- Column renames require lineage metadata to track the rename
- Column drops must trigger impact analysis on downstream reports
- Schema Registry audit logs are regulatory evidence
- Schema history must be immutable and retained 7+ years
- FULL_TRANSITIVE compatibility ensures all prior schemas remain readable
```

### 13.3 Controlled Change Management Pipeline

```
1. Developer creates schema branch
2. CI checks FULL_TRANSITIVE compatibility
3. Automated lineage impact analysis runs
4. PR submitted with impact report
5. Data Steward reviews
6. Regulatory Reporting reviewer signs off
7. Two-person approval (maker-checker)
8. Schema deployed to non-prod, integration tests pass
9. Schema deployed to prod in approved window
10. Post-deployment monitoring (24h observation)
```

### 13.4 Impact Analysis on Schema Change

When a schema change is proposed, the lineage system must answer:

| Question | Answer |
|----------|--------|
| Which regulatory templates use this field? | CR-101 (Credit Risk Daily), CE-202 (Counterparty Exposure) |
| What is materiality? | HIGH — feeds column 8 of CR-101 |
| Mitigation? | Dual-write for 14 days; both old and new field populated |
| Report owners? | Jane (CR-101), Mark (CE-202) — both must approve |

---

## 14. Best Practices

### 14.1 Design for Schema Flexibility

- **Use nullable fields wisely.** New fields should be nullable to preserve backward compatibility. Required fields are for true business keys only.
- **Prefer additive changes.** Adding a column is always safer than removing or renaming one. Plan deprecation windows for deletions (30–90 days).
- **Use extension fields.** A `MAP<STRING, STRING>` or JSON column can absorb unpredictable changes without schema evolution.
- **Don't partition on volatile columns.** Never partition on a column that might be renamed or dropped.

### 14.2 Schema Versioning

- Every schema change gets a **new version number** — no exceptions
- **Semantic versioning:** MAJOR (breaking), MINOR (additive), PATCH (metadata)
- **Store schema history immutably** in Schema Registry and Git
- **Include schema version in data lineage metadata**

### 14.3 Deprecation Periods for Breaking Changes

| Change | Notice | Practice |
|--------|--------|---------|
| Column removal | 30–90 days | Keep column marked deprecated; populate both old and new (dual-write) |
| Column rename | 30 days | Avro: use aliases; Iceberg: one-step rename with consumer notification |
| Type narrowing | Avoid | Not recommended; if required, full pipeline rebuild |
| Partition change | 90 days | New table with new scheme; dual-read during migration |

### 14.4 CI/CD Automation

- Every schema PR triggers compatibility checks (REST API to Schema Registry)
- dbt contract enforcement on model changes
- Great Expectations suites run as CI step
- **Block merges** that fail compatibility checks

### 14.5 Monitor Drift in Production

- Compute drift metrics on every batch/stream window (not retroactively)
- Store in Prometheus for real-time alerting and historical trends
- Recompute reference distributions after every model retraining
- Multi-window monitoring: short window (1h) for rapid detection, long window (7d) for trends

### 14.6 Have a Rollback Plan

- **Schema Registry:** Re-register previous version
- **Iceberg:** `SELECT * FROM t FOR SYSTEM_VERSION AS OF <snapshot-id>`
- **Delta Lake:** `RESTORE TABLE t TO VERSION AS OF <version>`
- **Consumer rollback:** Redeploy consumers pinned to previous schema version

---

## 15. Conclusion

Schema evolution and data drift are inevitable. The goal is not to prevent change — it's to **manage change safely, transparently, and with minimal disruption**.

**Key takeaways:**

1. **Schema evolution is structural** — govern it with Schema Registry, compatibility checks, versioned contracts.
2. **Data drift is content-based** — detect it statstically (PSI, KL, KS) and monitor it continuously.
3. **Know your compatibility modes** — BACKWARD (safe default), FULL (maximum safety), transitive (regulatory).
4. **Choose format-specific rules carefully** — Iceberg metadata-only evolution, Avro writer/reader resolution, Protobuf field number discipline.
5. **Data contracts bridge producers and consumers** — codify schema, semantics, SLAs, and change policies.
6. **Test everything** — schema validation in CI/CD, drift detection as regression tests, contract tests as acceptance gates.
7. **In banking, schema evolution is a regulatory concern** — BCBS 239 demands tracked, approved, impact-assessed changes.
8. **Design for flexibility** — nullable fields, additive changes, deprecation periods, always version.

The most resilient data platforms treat schema evolution and data drift not as problems to be eliminated, but as natural phenomena to be **detected, measured, governed, and absorbed** with grace.

---

## Further Reading

- [Apache Iceberg Schema Evolution](https://iceberg.apache.org/docs/latest/evolution/)
- [Confluent Schema Evolution & Compatibility](https://docs.confluent.io/platform/current/schema-registry/fundamentals/schema-evolution.html)
- [Delta Lake Schema Evolution Guide](https://delta.io/blog/2023-02-08-delta-lake-schema-evolution/)
- [Evidently AI Drift Detection](https://docs.evidentlyai.com/)
- [Great Expectations Schema Validation](https://docs.greatexpectations.io/docs/reference/learn/data_quality_use_cases/schema/)
- [Open Data Contract Standard](https://openmetadatastandards.org/data-contracts/overview/)
- [BCBS 239 Principles](https://www.bis.org/publ/bcbs239.pdf)
- [dbt Contracts](https://docs.getdbt.com/docs/collaborate/govern/model-contracts)
