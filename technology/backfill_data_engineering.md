# Backfill Strategies in Data Engineering

> A comprehensive guide to reprocessing historical data when pipeline logic, schema, or sources change.

---

## 1. What Is Backfill and Why Is It Needed

**Backfill** is the process of reprocessing historical data through a pipeline — re-running transformations over past time ranges to produce corrected or newly computed results.

### Common Triggers

| Trigger | Description |
|---|---|
| **Schema changes** | New column added, data type changed, column renamed |
| **Business logic changes** | Revenue calc changes from `price * qty` to `price * qty * (1 - discount)` |
| **Bug fixes** | Off-by-one error, wrong join key, null-handling bug in past outputs |
| **New data sources** | A new CRM is ingested; historical snapshots must include it for consistent reporting |
| **Historical corrections** | Source system corrected past records (e.g., payment provider resend) |
| **Regulatory requirements** | GDPR, SOX, HIPAA require reprocessing with updated anonymization or retention |
| **Late-arriving data** | Events arriving days late must merge into correct historical partitions |

---

## 2. Types of Backfill

**Full Backfill** — reprocesses all history. Simple but expensive. Use for pervasive schema changes, small datasets, or when raw data is guaranteed immutable.

**Incremental Backfill** — reprocesses a specific time window (e.g., `2024-01-01` to `2024-03-31`). Faster, cheaper, lower risk. Requires partition-aware pipelines and boundary condition handling.

**Selective Backfill** — reprocesses specific entities or partitions (e.g., `region='APAC'`). Most granular; minimal blast radius but may leave derived tables inconsistent.

---

## 3. Backfill Strategies

### Time-Travel / Reprocessing from Raw
Replay raw staging data through new pipeline logic, overwriting old output. Requires immutable raw storage and parameterized pipelines. Best for batch pipelines. Risk: raw data may have retention limits.

### Incremental Recomputation
Recalculate only changed partitions using new logic on a subset of raw data. Requires partition-level idempotency (MERGE, dynamic overwrite). Best for large datasets where full replay is infeasible.

### Snapshot-Based Rollback
Restore a previous known-correct state, then replay incremental changes. Requires versioned table format (Iceberg, Delta Lake, Hudi). Best for lakehouse architectures.

---

## 4. Backfill in Different Architectures

### 4.1 Batch — dbt

Supports: `--full-refresh`, `replace_where` (Snowflake/BigQuery/Databricks), microbatch strategy (dbt >=1.9), and date-range vars.

```sql
{{ config(
    materialized='incremental',
    unique_key='order_id',
    replace_where=['order_date >= ' ~ var('backfill_start', '1900-01-01')
                   ~ ' and order_date <= ' ~ var('backfill_end', '1900-01-01')]
) }}
SELECT order_id, customer_id, order_date, amount, tax,
       amount * (1 - COALESCE(discount_rate, 0)) AS net_amount
FROM {{ ref('stg_orders') }}
{% if is_incremental() and not var('backfill', false) %}
WHERE order_date >= (SELECT MAX(order_date) FROM {{ this }})
{% endif %}
```

```bash
# Full refresh
dbt run --select fct_orders --full-refresh

# Date-range backfill
dbt run --select fct_orders --vars '{
    "backfill": true,
    "backfill_start": "2024-06-01",
    "backfill_end": "2024-06-30"
}'

# Microbatch (dbt >= 1.9)
dbt run --select fct_orders \
    --event-time-start 2024-06-01 --event-time-end 2024-06-30
```

### 4.2 Batch — Apache Spark

Uses partition pruning + dynamic partition overwrite for efficient backfills.

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, coalesce, lit, current_timestamp

spark = SparkSession.builder.appName("backfill") \
    .config("spark.sql.sources.partitionOverwriteMode", "dynamic").getOrCreate()

raw = spark.table("bronze.orders") \
    .filter(col("order_date").between("2024-06-01", "2024-06-30"))

corrected = raw \
    .withColumn("net_amount", col("amount") * (1 - coalesce(col("discount_rate"), lit(0)))) \
    .withColumn("_backfilled_at", current_timestamp())

corrected.write \
    .mode("overwrite").format("delta") \
    .option("partitionOverwriteMode", "dynamic") \
    .partitionBy("order_date") \
    .saveAsTable("gold.fct_orders")
```

### 4.3 Batch — Apache Airflow

```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.bash import BashOperator
from airflow.models import Param

with DAG(dag_id="backfill_orders", schedule_interval=None,
         start_date=datetime(2024, 1, 1), catchup=False,
         params={"backfill_start": Param("2024-06-01", type="string", format="date"),
                 "backfill_end": Param("2024-06-30", type="string", format="date")}) as dag:

    run_spark = BashOperator(task_id="spark_backfill", bash_command=
        "spark-submit --conf spark.sql.sources.partitionOverwriteMode=dynamic "
        "/opt/pipelines/backfill.py --start {{ params.backfill_start }} "
        "--end {{ params.backfill_end }}")

    notify = BashOperator(task_id="notify",
        bash_command='echo "Backfill {{ params.backfill_start }} to {{ params.backfill_end }} done"')

    run_spark >> notify
```

```bash
# Trigger
airflow dags trigger backfill_orders \
    --conf '{"backfill_start":"2024-06-01","backfill_end":"2024-06-30"}'

# CLI backfill with concurrency limit
airflow dags backfill --dag-id backfill_orders \
    --start-date 2024-06-01 --end-date 2024-06-10 --max-active-runs 4

# Chunked backfill
for i in $(seq 0 7 30); do
    s=$(date -d "2024-06-01 + $i days" +%Y-%m-%d)
    e=$(date -d "2024-06-01 + $((i+6)) days" +%Y-%m-%d)
    airflow dags backfill --dag-id heavy_etl --start-date "$s" --end-date "$e" --max-active-runs 2
    sleep 30
done
```

### 4.4 Streaming — Apache Flink

Backfill uses **savepoints** — consistent snapshots of operator state.

```bash
# Savepoint then redeploy with new logic
flink savepoint <jobId> s3://flink-savepoints/orders
flink run -s s3://flink-savepoints/savepoint-<id> -d /opt/pipelines/orders-v2.jar

# Reset Kafka offsets for full history replay
kafka-consumer-groups.sh --bootstrap-server kafka:9092 \
    --group flink-orders --topic orders \
    --reset-offsets --to-datetime 2024-06-01T00:00:00.000 --execute
```

### 4.5 Streaming — Kafka Offset Reset

```bash
kafka-consumer-groups.sh --bootstrap-server kafka:9092 \
    --group orders-consumer --topic orders \
    --reset-offsets --to-earliest --execute

kafka-consumer-groups.sh --bootstrap-server kafka:9092 \
    --group orders-consumer --topic orders \
    --reset-offsets --to-datetime 2024-06-01T00:00:00.000 --execute
```

### 4.6 Lakehouse — Apache Iceberg

```sql
-- Time travel query
SELECT COUNT(*) FROM gold.fct_orders FOR SYSTEM_TIME AS OF '2024-06-15 10:00:00';

-- Rollback to undo a bad backfill
CALL spark_catalog.system.rollback_to_snapshot('gold.fct_orders', 784621904872139538);

-- Branch-based backfill (Iceberg >= 1.5)
CALL spark_catalog.system.create_branch('gold.fct_orders', 'backfill_jun');
CALL spark_catalog.system.fast_forward('gold.fct_orders', 'main', 'backfill_jun');
```

### 4.7 Lakehouse — Delta Lake

```sql
SELECT COUNT(*) FROM gold.fct_orders VERSION AS OF 42;
SELECT COUNT(*) FROM gold.fct_orders TIMESTAMP AS OF '2024-06-15 10:00:00';
DESCRIBE HISTORY gold.fct_orders;
RESTORE TABLE gold.fct_orders TO VERSION AS OF 42;
```

---

## 5. Scheduling and Orchestration

**Airflow:** Set `catchup=False`; always trigger backfills explicitly. Use `max_active_runs` (start with 2-4). Parameterize date ranges via DAG params. Log every run to an audit table.

**dbt:** Slim CI (`dbt build --select state:modified+ --defer --state target/`). Staggered backfill for large tables:

```bash
for WEEK in $(seq 1 52); do
    START=$(date -d "2024-01-01 + $(( (WEEK-1)*7 )) days" +%Y-%m-%d)
    END=$(date -d "2024-01-01 + $(( WEEK*7 - 1 )) days" +%Y-%m-%d)
    dbt run --select fct_orders \
        --vars "backfill: true, backfill_start: $START, backfill_end: $END"
done
```

**Partitioning:** Date-partitioned = high efficiency (partition pruning). Date + entity = medium (more files, granular). Hash = low (full scan). No partitioning = very low.

---

## 6. Data Idempotency

Idempotency ensures the same result whether a pipeline runs once or ten times. The single most critical property for backfill-safe pipelines.

### MERGE / UPSERT

```sql
MERGE INTO gold.fct_orders AS target
USING staging.corrected_orders AS source ON target.order_id = source.order_id
WHEN MATCHED THEN UPDATE SET target.amount = source.amount,
    target.net_amount = source.net_amount, target.updated_at = CURRENT_TIMESTAMP()
WHEN NOT MATCHED THEN INSERT
    (order_id, amount, net_amount, updated_at)
VALUES (source.order_id, source.amount, source.net_amount, CURRENT_TIMESTAMP());
```

### INSERT OVERWRITE Partition

```sql
BEGIN;
DELETE FROM gold.fct_orders WHERE order_date BETWEEN '2024-06-01' AND '2024-06-30';
INSERT INTO gold.fct_orders SELECT * FROM staging.corrected_orders
    WHERE order_date BETWEEN '2024-06-01' AND '2024-06-30';
COMMIT;
```

### Deduplication

```sql
WITH deduped AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY order_id ORDER BY ingested_at DESC) AS rn
    FROM bronze.orders WHERE order_date BETWEEN :start AND :end
)
SELECT * EXCEPT (rn) FROM deduped WHERE rn = 1;
```

### Common Anti-Patterns

| Anti-Pattern | Why It Breaks | Fix |
|---|---|---|
| `INSERT INTO` without dedup | Doubles data every rerun | MERGE or INSERT OVERWRITE |
| `CREATE OR REPLACE TABLE` | Blocks reads, costly | Partition-level operations |
| Hardcoded `CURRENT_TIMESTAMP` as partition key | New partitions each run | Use source-event timestamp |
| Non-deterministic functions | Different results each run | Snapshot timestamps at start |

---

## 7. Backfill Testing

### Row Count Reconciliation

```sql
SELECT COALESCE(a.order_date, b.order_date) AS order_date,
       a.row_count AS before_count, b.row_count AS after_count,
       a.row_count - b.row_count AS delta
FROM (SELECT order_date, COUNT(*) row_count, SUM(amount) total
      FROM gold.fct_orders WHERE order_date BETWEEN '2024-06-01' AND '2024-06-30' GROUP BY order_date) a
FULL OUTER JOIN (SELECT order_date, COUNT(*) row_count, SUM(net_amount) total
                 FROM gold.fct_orders_backfilled WHERE order_date BETWEEN '2024-06-01' AND '2024-06-30'
                 GROUP BY order_date) b ON a.order_date = b.order_date
WHERE a.row_count != b.row_count OR a.total != b.total;
```

### Checksum Comparison

```sql
SELECT COUNT(*) AS mismatches FROM (
    SELECT FARM_FINGERPRINT(TO_JSON_STRING(t)) FROM prod.table t WHERE dt BETWEEN '2024-06-01' AND '2024-06-30'
    EXCEPT DISTINCT
    SELECT FARM_FINGERPRINT(TO_JSON_STRING(t)) FROM staging.table t WHERE dt BETWEEN '2024-06-01' AND '2024-06-30'
);
```

### Tools & Staging Pattern

**Tools:** `data-diff` (Datafold) for cross-DB comparison, Great Expectations for validation suites, dbt tests (`unique`, `not_null`, custom), `pandas.testing.assert_frame_equal` for small datasets.

**Pattern:** Backfill into staging first, validate, then promote:

```bash
dbt run --select fct_orders --vars 'backfill: true, start: 2024-06-01, end: 2024-06-30' --target staging
dbt test --select fct_orders
# Only promote to prod after validation
```

---

## 8. Backfill Risks and Mitigation

| Risk | Impact | Mitigation |
|---|---|---|
| **Compute cost** | Cloud bill spikes | Off-peak scheduling; spot instances; limit parallelism |
| **Execution time** | Too long to complete | Chunk windows; partition pruning; incremental over full |
| **Downstream impact** | Dashboards read inconsistent data | Shadow tables; Blue-Green swap; communicate downtime |
| **Consistency** | Derived tables get out of sync | Atomic swaps; backfill in dependency order |
| **Idempotency failure** | Duplicates after retry | Enforce unique keys; test idempotency during CI |
| **Retention limits** | Raw data purged | Increase raw retention; archive to cost-effective storage |
| **Concurrent writes** | Backfill collides with incremental | Exclusive lock; separate Airflow pool; tag runs |

**Backfill Window Policy:** Hot (last 7d, auto-approved), Warm (7-90d, team lead), Cold (90d+, EM + cost review), Archival (1yr+, executive).

---

## 9. Tools and Patterns

### 9.1 dbt Backfill Pattern

```sql
{{ config(materialized='incremental', unique_key='order_id',
          on_schema_change='append_new_columns') }}
SELECT order_id, customer_id, order_date, amount,
       amount * (1 - COALESCE(discount_rate, 0)) AS net_amount,
       CURRENT_TIMESTAMP() AS _updated_at
FROM {{ ref('stg_orders') }}
{% if is_incremental() %}
    {% if var('backfill', False) %}
        WHERE order_date >= '{{ var("backfill_start") }}' AND order_date <= '{{ var("backfill_end") }}'
    {% else %}
        WHERE order_date >= (SELECT DATEADD(day, -3, MAX(order_date)) FROM {{ this }})
    {% endif %}
{% endif %}
```

### 9.2 Spark Partition Backfill Script

```python
import argparse
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, coalesce, lit, current_timestamp

def run_backfill(start_date, end_date):
    spark = SparkSession.builder.appName("order_backfill") \
        .config("spark.sql.sources.partitionOverwriteMode", "dynamic") \
        .enableHiveSupport().getOrCreate()

    corrected = spark.table("bronze.orders") \
        .filter(col("order_date").between(start_date, end_date)) \
        .withColumn("net_amount", col("amount") * (1 - coalesce(col("discount_rate"), lit(0)))) \
        .withColumn("_backfilled_at", current_timestamp())

    corrected.write.mode("overwrite").format("delta") \
        .option("partitionOverwriteMode", "dynamic") \
        .partitionBy("order_date").saveAsTable("gold.fct_orders")

    spark.table("gold.fct_orders") \
        .filter(col("order_date").between(start_date, end_date)) \
        .groupBy("order_date").agg({"*": "count", "net_amount": "sum"}) \
        .orderBy("order_date").show()
    spark.stop()

if __name__ == "__main__":
    p = argparse.ArgumentParser()
    p.add_argument("--start", required=True); p.add_argument("--end", required=True)
    args = p.parse_args(); run_backfill(args.start, args.end)
```

### 9.3 Airflow Chunked Backfill DAG

```python
from datetime import datetime, timedelta
from airflow.decorators import dag, task

@dag(schedule_interval=None, start_date=datetime(2024, 1, 1), catchup=False)
def chunked_backfill():
    @task
    def generate_chunks(**ctx):
        params = ctx["params"]
        start = datetime.strptime(params["backfill_start"], "%Y-%m-%d")
        end = datetime.strptime(params["backfill_end"], "%Y-%m-%d")
        cd = params.get("chunk_days", 7); chunks = []; cur = start
        while cur < end:
            ce = min(cur + timedelta(days=cd - 1), end)
            chunks.append({"chunk_start": cur.strftime("%Y-%m-%d"),
                           "chunk_end": ce.strftime("%Y-%m-%d")})
            cur = ce + timedelta(days=1)
        return chunks

    @task
    def process_chunk(chunk: dict):
        print(f"Processing {chunk['chunk_start']} to {chunk['chunk_end']}")
        # Insert real operator here

    generate_chunks() >> process_chunk.expand(chunk=generate_chunks())

dag = chunked_backfill()
```

### 9.4 Flink Savepoint Workflow

```bash
flink savepoint <jobId> s3://flink-savepoints/orders-v2 --kill
flink run -s s3://flink-savepoints/orders-v2/savepoint-<id> -d /opt/pipelines/orders-v2.jar
kafka-consumer-groups.sh --bootstrap-server kafka:9092 --group flink-orders-v2 \
    --topic orders --reset-offsets --to-datetime 2024-06-01T00:00:00.000 --execute
```

### 9.5 Kafka Replay (Python)

```python
from confluent_kafka import Consumer, TopicPartition
import datetime

def reset_to_timestamp(bootstrap, group, topic, timestamp):
    c = Consumer({"bootstrap.servers": bootstrap, "group.id": group})
    parts = [TopicPartition(topic, p) for p in c.list_topics(topic).topics[topic].partitions]
    target = int(datetime.datetime.strptime(timestamp, "%Y-%m-%dT%H:%M:%S").timestamp() * 1000)
    for tp in c.offsets_for_times([TopicPartition(topic, p.partition, target) for p in parts]):
        if tp.offset != -1: c.commit(offsets=[tp])
    c.close()

reset_to_timestamp("kafka:9092", "flink-orders-v2", "orders", "2024-06-01T00:00:00")
```

---

## 10. Backfill for Data Quality Fixes

### Correcting Bad Data

```sql
-- Identify affected days
SELECT order_date, COUNT(*) total, COUNT(NULLIF(amount, 0)) non_null
FROM gold.fct_orders WHERE order_date BETWEEN '2024-06-10' AND '2024-06-12' GROUP BY order_date;

-- Re-ingest and merge
MERGE INTO gold.fct_orders AS target USING (
    SELECT order_id, COALESCE(amount, 0) amount FROM bronze.orders
    WHERE order_date BETWEEN '2024-06-10' AND '2024-06-12'
) AS source ON target.order_id = source.order_id
WHEN MATCHED THEN UPDATE SET target.amount = source.amount;
```

### Compensating Transactions (Append-Only Pipelines)

```sql
INSERT INTO gold.fct_orders_audit
SELECT order_id, 'CORRECTION', old_amount, new_amount, CURRENT_TIMESTAMP() FROM corrected_orders;
```

---

## 11. Best Practices

1. **Design for idempotency from day one.** Every pipeline must produce identical output whether run once or ten times. Use MERGE, INSERT OVERWRITE partition, or DELETE+INSERT. Never use blind `INSERT INTO`.

2. **Maintain a raw/landing zone for replay.** Keep raw data immutable in a bronze layer. Set retention policies based on backfill window needs, not just cost.

3. **Use partition boundaries.** Partition tables by date. Align granularity with backfill windows (daily is standard). Avoid single partitions >100 GB.

4. **Validate before and after.** Capture row counts, sum checksums, distinct key counts before backfill. Compare against after-state automatically. Run dbt tests on backfilled ranges.

5. **Communicate with downstream consumers.** Notify dashboard owners, ML engineers, analytics teams. Maintain a backfill audit log:

```sql
CREATE TABLE IF NOT EXISTS audit.backfill_log (
    backfill_id UUID DEFAULT gen_random_uuid(), model_name STRING,
    start_date DATE, end_date DATE, reason STRING, triggered_by STRING,
    started_at TIMESTAMP, completed_at TIMESTAMP,
    rows_before BIGINT, rows_after BIGINT, status STRING,
    PRIMARY KEY (backfill_id)
);
```

6. **Document every backfill.** What was backfilled? Why? When? Who? What changed (PR link)? Validation results? Was rollback needed?

7. **Test in staging first.** Always run on non-production before touching production.

8. **Use Blue-Green swap.** Build corrected table under shadow name, then rename atomically.

9. **Limit concurrency.** Backfill 2-4 partitions at a time to avoid warehouse contention.

10. **Prefer incremental over full.** Full backfill is last resort. Try date-range first.

11. **Set cost alerts.** Tag backfill runs so cloud cost monitoring flags anomalous spend.

12. **Automate triggering.** Use CI/CD to detect model changes and auto-create backfill tickets.

---

## 12. Decision Framework

```
┌─ Need to backfill? (schema change, bug, logic change?) ─┐
│                          Yes                              │
└─────────────────────────┬────────────────────────────────┘
                          ▼
┌─ Data volume? ──────────────────────────────────────────┐
│  < 10 GB    → Full backfill                              │
│  10 GB–1 TB → Incremental window                        │
│  > 1 TB     → Selective / chunked                       │
└─────────────────────────┬────────────────────────────────┘
                          ▼
┌─ Time range? ──────────────────────────────────────────┐
│  Last 7 days  → Incremental window                      │
│  7–90 days    → Chunked incremental                    │
│  90+ days     → Raw-replay + audit                     │
└─────────────────────────┬────────────────────────────────┘
                          ▼
┌─ Change complexity? ────────────────────────────────────┐
│  Simple column add  → Incremental + schema evolution     │
│  Deterministic logic → Time-travel / raw replay         │
│  Non-deterministic   → Snapshot-based rollback           │
│  Stateful (Flink)   → Savepoint + offset reset           │
└─────────────────────────┬────────────────────────────────┘
                          ▼
┌─ Recommended strategy ─────────────────────────────────┐
│  Low volume + short range     → Full backfill            │
│  Medium vol + medium range    → Incremental + prune     │
│  High volume + long range     → Chunked + shadow table  │
│  Streaming pipeline           → Savepoint + reset       │
│  Lakehouse (Iceberg/Delta)    → Branch + time travel    │
└─────────────────────────────────────────────────────────┘
```

### Strategy Selection Matrix

| Volume | Time Range | Strategy | Cost |
|---|---|---|---|
| <10 GB | Any | Full backfill | Low |
| 10 GB–1 TB | Days–Weeks | Incremental window | Low |
| 10 GB–1 TB | Months | Chunked incremental | Medium |
| 1–10 TB | Days | Incremental + partition prune | Medium |
| 1–10 TB | Months | Chunked + shadow swap | High |
| >10 TB | Any | Branch-based + staggered | Very High |
| Streaming | Real-time | Savepoint + offset reset | Varies |
| Lakehouse | Any | Branch + time travel | Medium |

### Runbook Template

```yaml
backfill_id: bf-2024-001
model: gold.fct_orders
date_range: 2024-06-01 to 2024-06-30
reason: Bug fix — discount_rate not applied in net_amount calc (PR #1234)
triggered_by: @engineer
strategy: incremental_chunked (7-day chunks)
target: staging → prod
validation:
  rows_before: 1234567 | rows_after: 1234567
  total_before: $12345678 | total_after: $11987654
rollback_plan: RESTORE TABLE gold.fct_orders TO VERSION AS OF 42
notify: #data-eng-alerts
```

---

## References

- [dbt Incremental Models](https://docs.getdbt.com/docs/build/incremental-models)
- [Airflow Backfill Docs](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/backfill.html)
- [Apache Iceberg Time Travel](https://iceberg.apache.org/docs/latest/spark-queries/#time-travel)
- [Delta Lake Versioning](https://docs.delta.io/latest/delta-batch.html)
- [Flink Savepoints](https://nightlies.apache.org/flink/flink-docs-stable/docs/ops/state/savepoints/)
- [Kafka Offset Reset](https://kafka.apache.org/documentation/#consumerconfigs_auto.offset.reset)

---

*Last updated: July 2026*
