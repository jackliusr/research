# Apache Spark Tuning — A Practical Guide

A comprehensive reference for configuring, optimizing, and debugging Apache Spark applications in production. Covers resource management, configuration parameters, data optimization, code patterns, job design, and use-case-specific best practices.

---

## Table of Contents

1. [What Is Spark Tuning?](#1-what-is-spark-tuning)
2. [Spark Architecture Fundamentals for Tuning](#2-spark-architecture-fundamentals-for-tuning)
3. [Resource Configuration](#3-resource-configuration)
4. [Memory Configuration & the Unified Memory Model](#4-memory-configuration--the-unified-memory-model)
5. [Adaptive Query Execution (AQE)](#5-adaptive-query-execution-aqe)
6. [Concurrency & Partition Tuning](#6-concurrency--partition-tuning)
7. [Shuffle Tuning](#7-shuffle-tuning)
8. [Data Serialization](#8-data-serialization)
9. [Caching & Persistence](#9-caching--persistence)
10. [Join Optimization](#10-join-optimization)
11. [Partitioning Strategies](#11-partitioning-strategies)
12. [Spark UI & Metrics](#12-spark-ui--metrics)
13. [Common Tuning Scenarios by Use Case](#13-common-tuning-scenarios-by-use-case)
14. [Scenario-Specific Configuration Reference](#14-scenario-specific-configuration-reference)
15. [Tuning Workflow Checklist](#15-tuning-workflow-checklist)

---

## 1. What Is Spark Tuning?

Spark tuning is the iterative practice of configuring Spark applications and clusters to achieve optimal performance — faster execution, better resource utilisation, higher stability, and lower cost. It spans five dimensions:

| Dimension | What It Covers |
|---|---|
| **Resource Allocation** | CPU, memory, cores per executor, number of executors |
| **Configuration Parameters** | Hundreds of `spark.*` settings that control behaviour |
| **Data Optimisation** | Partitioning, serialization, storage format, compression |
| **Code Optimisation** | Shuffle minimisation, broadcast joins, caching, UDF reduction |
| **Job Design** | Stages, tasks, DAG structure, checkpointing |

### Key Principles

- **Tuning is iterative:** Measure → identify bottleneck → adjust → repeat. There is no single "best" config for every job.
- **The cost–performance trade-off:** Faster jobs consume more resources. The optimal configuration delivers the best performance *within your budget*, not infinite resources.
- **Start with data, not config:** Understanding your data volume, distribution, and skew informs every tuning decision. Config changes without data insight are guesses.
- **One change at a time:** Change one parameter, observe the effect, then proceed. Changing ten things at once tells you nothing about causality.
- **Know your bottleneck:** CPU-bound, memory-bound, I/O-bound, or network-bound jobs require different treatments. The Spark UI tells you which.

---

## 2. Spark Architecture Fundamentals for Tuning

Understanding the architecture is essential because every tuning decision maps to a component in the execution model.

### 2.1 Driver

The **driver** is the master process that manages job execution. It runs the `main()` method, builds the DAG, and schedules tasks on executors.

| Driver Component | Role |
|---|---|
| **DAG Scheduler** | Converts a logical plan (RDD lineage / DataFrame plan) into a physical DAG of stages separated by shuffle boundaries. |
| **Task Scheduler** | Takes stages from the DAG scheduler and launches tasks on executors, respecting data locality. |
| **Backend Scheduler** | Interfaces with the cluster manager (YARN, K8s, standalone) to request and release executor resources. |

**Driver memory** (`spark.driver.memory`) must be sufficient for:
- Collecting results from executors (`df.collect()`)
- Broadcasting small tables to all executors
- Building and maintaining the DAG (complex plans with many stages)
- Storing checkpoint metadata

**When to increase driver memory:**
- Large broadcast joins (driver collects the broadcast table before distributing it)
- Large `collect()` / `take()` operations
- High number of output partitions (driver coordinates all tasks)
- Complex DAGs with many stages and tasks
- Heavy checkpointing

### 2.2 Executors

**Executors** are worker processes that run tasks and store data. Each executor:

- Runs on a node in the cluster
- Hosts multiple cores (`spark.executor.cores`)
- Has a JVM heap of size `spark.executor.memory`
- Requires off-heap overhead (`spark.executor.memoryOverhead`) for JVM internals, PySpark processes, string internals, and direct buffers

| Config | Purpose | Typical Value |
|---|---|---|
| `spark.executor.memory` | JVM heap per executor | 16g – 64g |
| `spark.executor.memoryOverhead` | Off-heap memory (JVM overhead, Python processes, native buffers) | 10–20% of executor memory, min 384 MB |
| `spark.executor.cores` | CPU cores per executor | 4–5 (max ~5 for HDFS throughput) |
| `spark.executor.instances` | Number of executors (static allocation) | Depends on cluster size |
| `spark.dynamicAllocation.enabled` | Allow Spark to add/remove executors | `true` (default Spark 3+) |
| `spark.dynamicAllocation.minExecutors` | Minimum executors with dynamic allocation | 1–2 |
| `spark.dynamicAllocation.maxExecutors` | Maximum executors with dynamic allocation | Cluster capacity – 1 |

### 2.3 Stages & Tasks

| Concept | Definition |
|---|---|
| **Stage** | A set of tasks that can run in parallel, separated from other stages by a **shuffle boundary** (wide dependency). One stage must complete before the next can start. |
| **Task** | A unit of work sent to one executor core. One task processes one **partition**. Parallelism within a stage = total available cores. |

**Key insight for tuning:** If a stage has 200 partitions but only 50 cores, only 50 tasks run at a time. The stage's wall-clock time = ceil(200 / 50) × average task duration.

### 2.4 Data Locality

Spark prefers to schedule a task on the node where its input data resides. Locality levels (best to worst):

| Level | Meaning |
|---|---|
| `PROCESS_LOCAL` | Data is in the same JVM (best — no data movement) |
| `NODE_LOCAL` | Data is on the same machine (e.g., HDFS replica) |
| `RACK_LOCAL` | Data is on the same rack |
| `ANY` | Data must be fetched across the network (worst) |

If a compute slot is not available on the preferred node, Spark falls back to a lower locality level.

### 2.5 Shuffle

A **shuffle** redistributes data across partitions whenever a wide dependency is required. It is the most expensive operation in Spark.

**Operations that trigger a shuffle:** `groupBy`, `reduceByKey`, `join`, `repartition`, `coalesce` (if increase), `distinct`, `sortBy`, `orderBy`, `window` with `partitionBy`.

**Shuffle phases:**

| Phase | Description |
|---|---|
| **Shuffle Write (map side)** | Each mapper sorts its output, may spill to disk, and writes shuffle files to `spark.local.dir`. |
| **Shuffle Read (reduce side)** | Reducer fetches shuffle files from all map tasks, deserializes, aggregates, and may spill again. |

**Why shuffle is expensive:**
- Disk I/O: writing and reading intermediate shuffle files
- Network I/O: transferring data between nodes (especially for remote reads)
- Serialization/deserialization: converting data between wire and in-memory formats
- Sorting: data must be sorted (or at least hash-partitioned) for downstream operations

**External Shuffle Service (ESS):** In YARN mode, ESS (`spark.shuffle.service.enabled=true`) allows executors to serve shuffle data even after they are removed. Required when dynamic allocation is enabled on YARN.

### 2.6 Cluster Manager Options

| Manager | Best For | Trade-offs |
|---|---|---|
| **Standalone** | Small clusters, dev/test, simple deployments | No resource sharing, no dynamic allocation of executors (Spark's own manager) |
| **YARN** | Enterprise Hadoop ecosystems | Mature, supports dynamic allocation via ESS, resource sharing with other YARN apps |
| **Kubernetes** | Cloud-native deployments, container orchestrator | Native Spark 3.x support, operator for lifecycle, better isolation, complex networking |
| **Mesos** | — | Deprecated. Do not use for new deployments. |

**Choosing a manager:**
- **YARN** if your organisation already runs Hadoop/HDFS and needs resource sharing across multiple engines.
- **Kubernetes** if you are cloud-native, use containers as a first-class primitive, and want fine-grained resource control and auto-scaling.
- **Standalone** only for small, dedicated clusters where simplicity matters more than resource efficiency.

---

## 3. Resource Configuration

### 3.1 Executor Sizing Formula

```
Total memory per executor = spark.executor.memory
                          + spark.executor.memoryOverhead
                          + spark.memory.offHeap.size (if enabled)

Total cluster memory usage = (driver memory + driver overhead)
                           + executors × (executor memory + executor overhead)

Total cores = executors × spark.executor.cores

Stage parallelism = min(partitions, total cores)
```

### 3.2 Cores Per Executor

**Rule of thumb: 4–5 cores per executor.**

- **Why 5 max?** HDFS throughput bottlenecks at ~5 concurrent readers per node. More cores increase intra-executor parallelism but also increase JVM GC pressure and memory overhead.
- **Why not 1 core?** You waste the JVM's shared optimisation (JIT, class loading) and increase the number of executors (more heartbeats, more scheduling overhead).
- **Why not 8+?** GC pauses become problematic, HDFS throughput is saturated, and shuffle concurrency overwhelms network bandwidth.

### 3.3 Common Executor Sizing Patterns

| Pattern | Cores | Heap Memory | Overhead (10%) | Total |
|---|---|---|---|---|
| Small | 4 | 16 GB | 2 GB | ~18 GB |
| Medium | 4 | 32 GB | 4 GB | ~36 GB |
| Large | 5 | 64 GB | 6 GB | ~70 GB |
| Extra Large | 8 | 64 GB | 6 GB | ~70 GB (8 cores, high parallelism, high GC) |
| Memory-Heavy | 2 | 64 GB | 6 GB | ~70 GB (fewer tasks, more memory per task) |

**When to use each:**
- **Small (4c, 16 GB):** Small ETL jobs, low data volume (< 100 GB), limited cluster resources.
- **Medium (4c, 32 GB):** General-purpose, moderate shuffle volume, most production ETL.
- **Large (5c, 64 GB):** High-shuffle jobs, aggregations, joins on large datasets (1–10 TB).
- **Extra Large (8c, 64 GB):** CPU-bound compute (ML training, complex UDFs) where memory per task is not a bottleneck.
- **Memory-Heavy (2c, 64 GB):** Jobs with large per-partition memory needs (e.g., wide windows, large aggregations).

### 3.4 Driver Configuration

| Config | Default | When to Increase |
|---|---|---|
| `spark.driver.memory` | 1g | Large collects, broadcast of big dimension tables, complex DAGs |
| `spark.driver.cores` | 1 | Many concurrent tasks in scheduling queue |
| `spark.driver.memoryOverhead` | 10% of driver.memory or 384 MB min | Heavy broadcast, large collect results, many output files |

**Guidelines:**
- For most ETL, 4g–8g driver memory is sufficient.
- For ML training with large model parameters collected to the driver, use 16g–32g.
- For heavy broadcast joins (e.g., 500 MB dimension table), ensure driver memory is at least 2× the broadcast table size.

### 3.5 Dynamic Allocation

```
spark.dynamicAllocation.enabled = true       # default in Spark 3+
spark.dynamicAllocation.minExecutors = 2
spark.dynamicAllocation.maxExecutors = 100   # or cluster capacity
spark.dynamicAllocation.initialExecutors = 5
spark.dynamicAllocation.executorIdleTimeout = 60s   # scale down after idle
```

**How it works:**
- **Scale up:** When pending tasks are backed up, Spark requests additional executors from the cluster manager.
- **Scale down:** When executors remain idle beyond `executorIdleTimeout`, Spark releases them.

**Requirements:**
- YARN: Enable the **External Shuffle Service** (`spark.shuffle.service.enabled=true`) so removed executors can still serve shuffle data.
- K8s: No external service needed; shuffle data is managed by the shuffle manager.

**When to use static allocation instead:**
- Jobs with predictable, constant resource needs.
- When you need guaranteed capacity (no risk of executors being taken by other apps).
- When the cluster manager doesn't support dynamic allocation well.

---

## 4. Memory Configuration & the Unified Memory Model

### 4.1 Memory Regions

Spark's JVM heap is divided into three regions:

```
┌─────────────────────────────────────────────────┐
│  Reserved Memory (300 MB default)                │  Internal Spark storage, not tunable
├─────────────────────────────────────────────────┤
│  User Memory (spark.memory.fraction × 0.4)       │  User-defined data structures, UDF internals
├─────────────────────────────────────────────────┤
│  Unified Pool (spark.memory.fraction × 0.6)      │  Subdivided into:
│  ├── Execution Memory    (0.5 of pool default)    │  Shuffle, joins, aggregations, sorts
│  ├── Storage Memory      (0.5 of pool default)    │  Cache, broadcast variables
│  └── (Dynamic borrowing between both)            │  Execution evicts storage; storage cannot evict execution
└─────────────────────────────────────────────────┘
```

### 4.2 Key Memory Configs

| Config | Default | Description |
|---|---|---|
| `spark.memory.fraction` | 0.6 | Fraction of (heap – 300 MB) used for execution + storage. Remainder is "user memory" for UDFs, row objects, etc. |
| `spark.memory.storageFraction` | 0.5 | Fraction of the unified pool that is reserved for storage. If storage is low, execution can borrow, but cached blocks may be evicted. |
| `spark.memory.offHeap.enabled` | false | Enable off-heap memory for Tungsten encoding. |
| `spark.memory.offHeap.size` | 0 | Off-heap memory size when enabled. |

### 4.3 Tuning Memory Configurations

**spark.memory.fraction:**
- **Default 0.6** works well for most jobs.
- **Increase (0.7–0.8)** when you need more execution/storage space and your UDFs use little user memory.
- **Decrease (0.4–0.5)** when your UDFs allocate large objects or have high overhead.

**spark.memory.storageFraction:**
- **Increase (0.6–0.7)** when caching large datasets is critical and execution can tolerate eviction.
- **Decrease (0.3–0.4)** when shuffle volume is high and execution memory is the bottleneck.

**Memory pressure symptoms:**
- Frequent `spark.memory.storageFraction` evictions in Spark UI Storage tab
- High GC time (> 10–15% of task time)
- Shuffle spills to disk
- OOM errors (executor lost, broadcast OOM)

---

## 5. Adaptive Query Execution (AQE)

Adaptive Query Execution is the single most impactful tuning feature in Spark 3.x. It adjusts the execution plan at runtime based on intermediate statistics.

### 5.1 Enabling AQE

```
spark.sql.adaptive.enabled = true         # Spark 3.x, default true in Spark 3.2+
spark.sql.adaptive.coalescePartitions.enabled = true  # automatically coalesce shuffle partitions
spark.sql.adaptive.skewJoin.enabled = true            # handle skewed joins automatically
spark.sql.adaptive.localShuffleReader.enabled = true  # convert SMJ to BHJ when one side small
```

### 5.2 What AQE Fixes

| Problem | AQE Solution |
|---|---|
| **Wrong partition count guess** | `spark.sql.adaptive.coalescePartitions.enabled` coalesces or splits shuffle output partitions based on actual data size. Target partition size: `spark.sql.adaptive.advisoryPartitionSizeInBytes` (default 64 MB). |
| **Skewed join keys** | `spark.sql.adaptive.skewJoin.enabled` splits skewed partitions into sub-partitions for balanced processing. Config: `spark.sql.adaptive.skewJoin.skewedPartitionFactor` (default 5) and `spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes` (default 256 MB). |
| **Wrong join strategy** | If the runtime statistics show a table is small enough for broadcast, AQE switches from SortMergeJoin to BroadcastHashJoin via `spark.sql.adaptive.localShuffleReader`. |
| **Static partition count** | If `spark.sql.shuffle.partitions=200` is too high or too low for actual data, AQE adjusts it per stage. |

### 5.3 AQE Configuration Reference

```properties
# Enable AQE (recommended for all Spark 3.x jobs)
spark.sql.adaptive.enabled=true

# Target partition size after coalescing (default 64 MB)
spark.sql.adaptive.advisoryPartitionSizeInBytes=64m

# Coalesce minimum partition size (partitions below this are merged)
spark.sql.adaptive.coalescePartitions.minPartitionSize=1m

# Skew join settings
spark.sql.adaptive.skewJoin.enabled=true
spark.sql.adaptive.skewJoin.skewedPartitionFactor=5
spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes=256m

# Force AQE to re-optimise after a shuffle
spark.sql.adaptive.reOptimizeQuery=true
```

---

## 6. Concurrency & Partition Tuning

### 6.1 Key Concurrency Configs

| Config | Default | Description |
|---|---|---|
| `spark.sql.shuffle.partitions` | 200 | Number of partitions (output files) after a shuffle operation for SQL/DataFrame |
| `spark.default.parallelism` | total cores × 2 | Default partitions for RDD operations (shuffle, `parallelize`, etc.) |
| `spark.sql.files.maxPartitionBytes` | 128 MB | Max bytes per partition when reading files |
| `spark.sql.files.openCostInBytes` | 4 MB | Cost to open a file (affects whether small files are packed together) |

### 6.2 Determining Partition Count

**For shuffle operations:**
```
Target partition size: 100–200 MB after shuffle
shuffle.partitions = ceil(data size / target partition size)

Example: 100 GB of shuffled data → ceil(100 × 1024 / 200) = ceil(512) = 512 partitions
```

**For file reads:**
```
Number of partitions ≈ total data / spark.sql.files.maxPartitionBytes
=
128 MB default means roughly 1 partition per 128 MB of input data
```

**For RDD operations:**
```
spark.default.parallelism = 2× to 3× total cores in the cluster

Example: 50 executors × 4 cores = 200 cores → parallelism = 400–600
```

### 6.3 Problems from Wrong Partition Count

**Too few partitions:**
- Not all cores are utilised (some cores idle)
- High memory pressure on each task (more data per partition)
- High GC pressure
- Risk of OOM (executor running out of memory)
- Long shuffle writes and reads (large blocks)

**Too many partitions:**
- High scheduling overhead (more tasks to schedule)
- Small output files (HDFS small-file problem)
- High shuffle I/O overhead (many small shuffle blocks)
- Slow task serialization
- Wasted resources (task overhead dominates compute)

### 6.4 General Rules

- **Target 100–200 MB per partition after shuffle** (AQE advisory partition size can be set to 64 MB if more parallelism is needed).
- **Target 2–3× parallelism of available cores** during compute-heavy stages so that small skews don't leave cores idle.
- **For reads, 128 MB partitions** is a good baseline. Increase if reading many small files, decrease if each file is large and you want more parallelism.

---

## 7. Shuffle Tuning

Shuffle is the single biggest performance bottleneck in Spark jobs. Every wide dependency (`groupBy`, `join`, `repartition`, `distinct`, `sortBy`, `window`) triggers a shuffle.

### 7.1 Shuffle Write Tuning

| Config | Default | Tuning Guidance |
|---|---|---|
| `spark.shuffle.file.buffer` | 32 KB | Increase to 64 KB–128 KB for larger shuffle writes (reduces disk I/O syscalls) |
| `spark.shuffle.spill.compress` | true | Keep enabled; compression reduces spill size |
| `spark.shuffle.compress` | true | Keep enabled; compresses shuffle output |
| `spark.io.compression.codec` | lz4 | `lz4` (fast), `snappy` (fast), `zstd` (best compression, slower). Use `zstd` for shuffle-heavy jobs. |
| `spark.io.compression.lz4.blockSize` | 32 KB | Increase to 64 KB–128 KB for better compression ratio (higher memory use) |
| `spark.io.compression.zstd.level` | 1 | Increase to 3–6 for better compression at cost of speed |
| `spark.shuffle.sort.bypassMergeThreshold` | 200 | If number of mapper partitions ≤ this threshold, bypass merge-sort (faster writes). Increase for jobs with few partitions per mapper. |
| `spark.shuffle.spill.numElementsForceSpillThreshold` | Long.MAX_VALUE | Not normally tuned; spill is governed by memory pressure |

**Compression codec selection:**
- **lz4 or snappy:** Fast compression/decompression, moderate ratio. Best for CPU-limited shuffle.
- **zstd:** Better compression (30–50% smaller), slower. Best for network/disk I/O-limited shuffle (common in cloud environments).

### 7.2 Shuffle Read Tuning

| Config | Default | Tuning Guidance |
|---|---|---|
| `spark.reducer.maxSizeInFlight` | 48 MB | Increase to 96–192 MB if network bandwidth is high and shuffle read is bottleneck (more concurrent fetches) |
| `spark.reducer.maxReqSizeShuffleToMem` | Long.MAX_VALUE | Decrease if reducers OOM during shuffle fetch (forces spilling) |
| `spark.shuffle.detectCorrupt` | true | Keep enabled |
| `spark.shuffle.registration.timeout` | 10 s | Increase to 60–120 s for large shuffles with many map outputs |
| `spark.shuffle.io.maxRetries` | 3 | Increase to 6–10 for unstable networks |
| `spark.shuffle.io.retryWait` | 5 s | Increase to 10–30 s for networks with transient failures |
| `spark.shuffle.io.numConnectionsPerPeer` | 1 | Increase to 2–4 for higher shuffle throughput on fast networks |

### 7.3 Spill Tuning

When the executor memory is insufficient to hold all shuffle data in memory, Spark **spills** to disk. Spill is orders of magnitude slower than in-memory processing and is the #1 performance killer.

**Metrics that indicate spill (Spark UI → Stages):**
- `Shuffle Spill (Memory)`: data written to memory before spilling
- `Shuffle Spill (Disk)`: data actually spilled to disk

**If spills are detected, try (in order of preference):**
1. **Increase partition count** — smaller partitions = less data per task
2. **Increase executor memory** — more headroom for in-memory operations
3. **Increase `spark.memory.fraction`** (up to 0.8) — more of heap usable for execution
4. **Tune `spark.reducer.maxSizeInFlight`** — reduce per-fetch size

### 7.4 Shuffle Health Metrics in Spark UI

| Metric | What It Indicates |
|---|---|
| **Shuffle Write Size / Records** | Volume of data written by map side |
| **Shuffle Read Size / Records** | Volume of data read by reduce side |
| **Shuffle Spill (Memory)** | Data written to spill buffer before going to disk |
| **Shuffle Spill (Disk)** | Data actually spilled to disk — **this should be 0 in a well-tuned job** |
| **Shuffle Read Blocked Time** | Time reducers spent waiting for data from mappers — high values indicate straggler mappers or network congestion |
| **Shuffle Remote Reads** | Data fetched from other nodes — prefer **Local Reads** (data on same node) |
| **Shuffle Local Reads** | Data fetched from the same node — no network cost |

### 7.5 Shuffle Tuning by Pattern

| Scenario | Recommended Actions |
|---|---|
| **High shuffle volume (> 100 GB)** | Increase partition count (reduce spill), enable `zstd` compression, increase `spark.reducer.maxSizeInFlight`, ensure sufficient executors |
| **Data skew in shuffle** | Enable AQE skew join (`spark.sql.adaptive.skewJoin.enabled=true`), apply salting technique, broadcast the smaller side |
| **Memory pressure during shuffle** | Increase `spark.memory.fraction` (max 0.8), increase executor memory, reduce `spark.reducer.maxSizeInFlight` |
| **Slow shuffle fetch (blocked time high)** | Check network bandwidth, enable ESS locality, increase `spark.shuffle.io.numConnectionsPerPeer`, check for straggler map tasks |
| **Shuffle OOM on reducer** | Increase partition count, reduce `spark.reducer.maxReqSizeShuffleToMem`, increase executor memory |

### 7.6 Salting Technique for Skewed Keys

When a single key (e.g., `user_id = 42`) dominates the dataset:

```python
from pyspark.sql import functions as F
from pyspark.sql.types import *

# Add a random prefix (salt) to the skewed key to distribute across partitions
salted_df = skewed_df.withColumn(
    "salted_key",
    F.concat(F.col("join_key"), F.lit("_"), (F.rand() * num_salts).cast("int"))
)

# For the smaller table, replicate rows for each salt value
small_df_replicated = small_df.withColumn(
    "salt_range", F.array([F.lit(i) for i in range(num_salts)])
).select(F.explode("salt_range").alias("salt"), "*")

salted_small = small_df_replicated.withColumn(
    "salted_key", F.concat(F.col("join_key"), F.lit("_"), F.col("salt"))
)

# Join on salted key
result = salted_df.join(salted_small, "salted_key")
```

---

## 8. Data Serialization

Serialization affects CPU, memory, and network bandwidth. The wrong choice can double task times.

### 8.1 Serialization Options

| Serializer | Speed | Size | Usage |
|---|---|---|---|
| **Java Serialization** | Slow (1×) | Verbose (1×) | Default for RDD. Not recommended for performance-critical paths. |
| **Kryo Serialization** | 10× faster | 2–5× smaller | `spark.serializer=org.apache.spark.serializer.KryoSerializer`. Use for all RDD-based code. |
| **Tungsten (Spark SQL)** | Optimised | Compact | Used automatically by DataFrames/Datasets. Not affected by `spark.serializer`. |

### 8.2 Configuring Kryo Serialization

```properties
spark.serializer=org.apache.spark.serializer.KryoSerializer
spark.kryo.registrationRequired=true           # force registration (best performance)
spark.kryo.classesToRegister=com.myapp.MyClass,com.myapp.OtherClass
spark.kryoserializer.buffer.max=64m
spark.kryoserializer.buffer=64k
spark.kryo.referenceTracking=false             # disable if no cyclic references
```

**Registering custom classes** avoids Kryo writing fully-qualified class names with each object, saving both space and time. Set `registrationRequired=true` and list classes in `spark.kryo.classesToRegister`.

### 8.3 Spark SQL / DataFrame Serialization

Spark SQL uses **Tungsten** — an optimised binary format that operates on compressed row data (`UnsafeRow`). Tungsten serialization:

- Is **automatic** for DataFrames and Datasets
- Uses code generation for expression evaluation
- Can store data off-heap (`spark.memory.offHeap.enabled=true`)
- Is **not affected** by `spark.serializer` setting

**Impact of UDFs:**
- Python UDFs: break Tungsten optimisation — data must be deserialised from Tungsten, converted to Python objects, then re-serialised.
- Java/Scala UDFs: less overhead but still break some code generation optimisations.
- **Prefer built-in Spark SQL functions** over UDFs whenever possible.

### 8.4 Serialisation Tuning Summary

- **Use Kryo** for RDD-based operations (MLlib, GraphX, custom RDD workflows)
- **Use DataFrames/Datasets** for SQL/ETL operations — Tungsten handles it automatically
- **Minimise UDFs** — they force serialisation round-trips
- **For external connectors** (Kafka, Redis), use connector-specific serializers (e.g., Kafka's Avro/Protobuf serde)
- **Set `spark.kryo.referenceTracking=false`** if your data has no cyclic object references (saves CPU)

---

## 9. Caching & Persistence

Caching avoids recomputing DataFrames/RDDs that are used multiple times. Used well, it accelerates iterative algorithms. Used poorly, it wastes memory and increases GC pressure.

### 9.1 Storage Levels

| Level | Space | CPU | Memory | Disk | Notes |
|---|---|---|---|---|---|
| `MEMORY_ONLY` | High | Low | Full | No | Fastest, but recompute if evicted. Default for `cache()`. |
| `MEMORY_ONLY_SER` | Medium | Medium (serialize) | Less | No | Saves memory at cost of CPU on access. Uses `spark.serializer`. |
| `MEMORY_AND_DISK` | High | Low | Full | Spilled | Safer — spills to disk instead of recomputing. |
| `MEMORY_AND_DISK_SER` | Medium | Medium | Less | Spilled | Best balance for large datasets. |
| `DISK_ONLY` | Low | High (recompute) | None | Full | Memory-efficient but slow. Only if you want to avoid recomputation at any memory cost. |
| `OFF_HEAP` | Medium | Medium | Off-heap | No | Offloads GC. Requires `spark.memory.offHeap.enabled=true`. |

### 9.2 When to Cache

**Cache if the same DataFrame/RDD is used multiple times:**
```python
# ✅ Good: used twice
df_clean = df.filter(...).select(...).cache()
result1 = df_clean.groupBy("region").agg(...)
result2 = df_clean.join(dim_table, "key")

# Always unpersist when done
df_clean.unpersist()
```

**Don't cache if:**
- The data is used only once (caching adds overhead with no benefit)
- The data would be evicted before reuse (too large for available memory)
- You can recompute faster than reading from cache (rare, but possible with filter pushdown)

### 9.3 Caching Best Practices

| Practice | Why |
|---|---|
| **Cache at the right granularity** | Too early → large cache that may be evicted. Too late → not enough reuse. Cache after expensive transformations, before wide dependencies. |
| **Prefer `MEMORY_AND_DISK_SER` for large datasets** | Avoids OOM while preserving recomputation safety. |
| **Always unpersist** | `df.unpersist()` frees memory immediately. Leftover cache causes GC pressure in subsequent stages. |
| **Use Spark SQL table caching** | `spark.catalog.cacheTable("table_name")` lets Catalyst optimise reuse across queries. Uncache with `spark.catalog.uncacheTable("table_name")`. |
| **Use checkpointing for complex DAGs** | Checkpoints truncate the lineage graph, preventing stack overflow on very long DAGs. `df.checkpoint()` saves to reliable storage (e.g., HDFS) and breaks lineage. |
| **For iterative algorithms** | Cache training data, unpersist intermediate results no longer needed. MLlib calls `cache()` on input data automatically. |

### 9.4 Caching and GC

Large cached datasets in `MEMORY_ONLY` mode are stored as deserialised Java objects, which:
- Consume more memory than serialised forms
- Fragment the old generation heap space more quickly
- Trigger more frequent and longer GC pauses

**Mitigations:**
- Use `MEMORY_ONLY_SER` — serialised objects are compact and GC-friendly
- Use `OFF_HEAP` — data is outside the JVM heap entirely, no GC impact
- Unpersist aggressively when the cached data is no longer needed

---

## 10. Join Optimization

Joins are among the most resource-intensive operations. Choosing the right strategy can mean the difference between a 5-minute job and a job that OOMs.

### 10.1 Join Strategies in Spark SQL

| Strategy | Abbr | Shuffle? | When to Use |
|---|---|---|---|
| **Broadcast Hash Join** | BHJ | No | One table small enough to broadcast (default < 10 MB). Fastest. |
| **Sort Merge Join** | SMJ | Yes (both sides) | Default for large tables. Stable, scales well. |
| **Shuffled Hash Join** | SHJ | Yes (one side) | One side much smaller after shuffle but too big for broadcast. |
| **Broadcast Nested Loop Join** | BNLJ | No | Cross join, non-equi joins. Very expensive. |

### 10.2 Broadcast Hash Join (BHJ)

**When:** One table fits within `spark.sql.autoBroadcastJoinThreshold` (default 10 MB).

**Characteristics:**
- No shuffle — driver collects the small table, broadcasts it to all executors
- Each executor builds an in-memory hash table and probes with the large table
- Fast and resource-efficient

**Hint syntax:**
```python
from pyspark.sql import functions as F

# SQL hint
spark.sql("SELECT /*+ BROADCAST(d) */ * FROM fact f JOIN dim d ON f.key = d.key")

# DataFrame hint
fact_df.join(dim_df.hint("broadcast"), "key")
```

**Configuration:**
```properties
spark.sql.autoBroadcastJoinThreshold=10m    # default; increase to 100 MB if driver memory allows
spark.sql.adaptive.localShuffleReader.enabled=true  # AQE converts SMJ→BHJ if one side is small
```

**⚠️ Warning:** Broadcasting a 500 MB table requires the driver to hold 500 MB + overhead. Ensure driver memory is sufficient. If the broadcast table is too large, the driver will OOM.

### 10.3 Sort Merge Join (SMJ)

**When:** Both tables are large and no broadcast is possible.

**Characteristics:**
- Both sides are shuffled by join key
- Each partition is sorted
- A merge pass joins the sorted streams
- Default in Spark 3.x (`spark.sql.join.preferSortMergeJoin=true`)

**Tuning for SMJ:**
- Ensure sufficient partitions (avoid spilling during sort)
- Enable AQE to coalesce partitions if needed
- Use bucketing to avoid the shuffle entirely

### 10.4 Shuffled Hash Join (SHJ)

**When:** SMJ is not possible (e.g., non-sortable join condition) and one side is significantly smaller after the shuffle.

**Characteristics:**
- The smaller shuffled side builds a hash table partition-by-partition
- The larger side probes the hash table
- Less common than SMJ in Spark 3.x

### 10.5 Broadcast Nested Loop Join (BNLJ)

**When:** Cross joins or non-equi joins (e.g., `a.value < b.value`).

**Characteristics:**
- Extremely expensive — nested loops over all combinations
- Avoid if possible. Restructure the query to use equi-joins.

### 10.6 Bucketing for Join Optimisation

**Bucketing** pre-partitions data by a join key at write time, so joins on that key require **no shuffle**.

```python
# Write bucketed table
fact_df.write.bucketBy(64, "user_id").sortBy("user_id").saveAsTable("fact_bucketed")
dim_df.write.bucketBy(64, "user_id").sortBy("user_id").saveAsTable("dim_bucketed")

# Read and join — no shuffle if both tables use the same bucket count
fact_bucketed = spark.table("fact_bucketed")
dim_bucketed = spark.table("dim_bucketed")
result = fact_bucketed.join(dim_bucketed, "user_id")  # no shuffle!
```

**Bucketing rules:**
- Both tables must be bucketed on **the same key** with **the same bucket count** (or a multiple)
- `spark.sql.bucketing.enabled=true` (default)
- Bucket count should match the data size: 1–2 GB per bucket is a good starting point
- Compaction is needed over time as data accumulates (bucketing degrades with many small files)

### 10.7 Skew Join Handling

**AQE skew join (automatic):**
```properties
spark.sql.adaptive.skewJoin.enabled=true
spark.sql.adaptive.skewJoin.skewedPartitionFactor=5        # a partition is skewed if it is 5× larger than median
spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes=256m  # minimum skewed partition size
```

**Salting (manual, for extreme skew):**
See [Section 7.6](#76-salting-technique-for-skewed-keys) for the salting pattern. Use when AQE skew join is insufficient (one key dominates > 50% of data).

---

## 11. Partitioning Strategies

Partitioning decisions affect every operation — reads, writes, shuffles, and joins.

### 11.1 Partition Discovery (Directory Partitioning)

When reading from HDFS/S3 with a directory structure like:

```
/data/year=2024/month=01/day=15/
```

Spark discovers the partition columns (`year`, `month`, `day`) and can **prune** partitions when filters are applied.

```python
# Spark reads only the relevant partitions
df = spark.read.parquet("s3://data/")
filtered = df.filter("year = 2024 AND month = 01 AND day = 15")
```

**Key configs:**
```properties
spark.sql.sources.partitionPrune=true          # default — enables partition pruning
spark.sql.parquet.filterPushdown=true          # default — pushes filters to Parquet reader
```

**Best practices for directory partitioning:**
- Partition by **high-cardinality filter columns** (date > region > device)
- Limit partition count to **thousands, not millions** (metadata overhead grows with partition count)
- Test with a small date range first, then scale
- Avoid too many nested levels (3–4 levels max: `year/month/day/hour` is fine)

### 11.2 Repartition vs Coalesce

| Operation | Shuffle? | Use Case |
|---|---|---|
| `repartition(n)` | Full shuffle | Increase or decrease partitions, even distribution |
| `coalesce(n)` | No shuffle (merge adjacent) | Decrease partitions only, avoid full shuffle |

**When to use each:**
```python
# ✅ Coalesce: decrease partitions before write (avoids shuffle)
df.coalesce(50).write.parquet("output/")

# ✅ Repartition: increase parallelism for a join
df.repartition(500, "join_key").join(other_df.repartition(500, "join_key"), "join_key")

# ✅ Repartition: even distribution after filter (smaller dataset)
filtered_df.repartition(200).write.parquet("filtered_output/")
```

### 11.3 Target Partition Sizes

| Context | Target Partition Size | Why |
|---|---|---|
| **After shuffle** | 100–200 MB | Balances parallelism with task overhead |
| **File reads** | 128–256 MB | Matches HDFS block size, good I/O pattern |
| **Bucketing** | 1–2 GB per bucket | Large enough for efficient merge, small enough for parallelism |
| **Broadcast hash table** | < 10 MB (up to 100 MB with care) | Must fit in executor memory for BHJ |
| **ML training** | 10–100 MB | Smaller partitions enable finer-grained parallelism for iterative algorithms |

### 11.4 Data Organisation for Performance

| Technique | Benefit | When to Use |
|---|---|---|
| **Directory partitioning** (`year=.../month=...`) | Filter pruning at read time | Queries filtered by date/region |
| **Bucketing** (`bucketBy`) | Shuffle-free joins | Frequent joins on the same key |
| **Columnar storage** (Parquet/ORC) | Compression, predicate pushdown, vectorised reads | Analytical queries, ETL |
| **Sorting within partitions** (`sortBy`) | Better compression, faster merges | After bucketing, or before writing large tables |
| **Compaction** (merge small files) | Improved read throughput | After streaming writes or incremental loads |

---

## 12. Spark UI & Metrics

The Spark UI (port 4040 by default) is the single most important tool for identifying performance bottlenecks.

### 12.1 Key Tabs

| Tab | What It Shows | Tuning Value |
|---|---|---|
| **Jobs** | Job-level view: success/failure, duration, event timeline, DAG visualisation | High-level overview, identify which job stage is slowest |
| **Stages** | Per-stage details: task metrics (duration, input size, shuffle read/write, spills, GC time, serialisation time) | **Most important tab for tuning.** Identify stragglers, skew, spills, GC pressure. |
| **Storage** | Cached RDD/DataFrame info: memory usage, disk usage, replication, partition size | Verify caching is effective; detect evictions |
| **Environment** | Spark config values — useful to verify your settings are applied | Debug configuration issues |
| **Executors** | Per-executor summary: tasks, cores, memory, GC time, shuffle read/write, storage memory, failed tasks, dead executors | Identify under/over-utilised executors, detect OOM |
| **SQL** | Physical and optimized query plans, per-operator metrics, whole-stage codegen status, job ID mapping | Deep-dive into SQL performance, see AQE plan changes |

### 12.2 Metrics to Monitor

| Metric | Location | What It Tells You |
|---|---|---|
| **Task Duration** | Stages tab | Distribution: are most tasks similar duration? If some are much longer → data skew. |
| **GC Time** | Stages tab, Executors tab | > 10–15% of task time → memory pressure, too much garbage |
| **Shuffle Read Size / Records** | Stages tab | Volume of data read per task |
| **Shuffle Write Size / Records** | Stages tab | Volume of data written per task |
| **Shuffle Spill (Memory / Disk)** | Stages tab | **If non-zero, you have a memory problem.** Fix first. |
| **Input Size** | Stages tab | Data read from source per task |
| **Shuffle Remote Reads** | Stages tab | High ratio → tasks not co-located with data → locality issue |
| **Scheduler Delay** | Stages tab | Time spent waiting for slot → cluster contention, too few executors |
| **Task Deserialisation Time** | Stages tab | High → serialization bottleneck (use Kryo) |
| **Shuffle Read Blocked Time** | Stages tab | Reducers waiting for mapper output → straggler mappers or network |
| **Peak Execution Memory** | Stages tab | Did any task approach executor memory limit? |

### 12.3 Identifying Bottlenecks from Spark UI

| Symptom | Likely Cause | Fix |
|---|---|---|
| **High GC time (> 15%)** | Too many cached objects, executors too small | Increase executor memory, switch to MEMORY_ONLY_SER, reduce cache size, use Kryo |
| **Shuffle Spill (Disk) > 0** | Partitions too large, executors too small, memory fraction too low | Increase partition count, increase executor memory, increase `spark.memory.fraction` |
| **Skewed task durations** (some tasks 10× others) | Data skew — uneven key distribution | Enable AQE skew join, salting, increase partitions |
| **High Scheduler Delay** | Not enough executors for task volume | Increase executor count, enable dynamic allocation |
| **Shuffle Remote Reads >> Local Reads** | Data locality problem, or job inherently cross-node | Check cluster topology, reduce shuffle if possible |
| **Low CPU utilisation** | Spilling, I/O wait, or serialization bottleneck | Check spills first, then serialization configuration |
| **Executor Lost / OOM** | Memory configuration insufficient | Increase executor memory/overhead, increase partitions, reduce `spark.memory.fraction` if UDF-heavy |

### 12.4 DAG Visualisation

The DAG in the Jobs tab shows the physical plan as a graph of stages connected by shuffle dependencies.

**What to look for:**
- A single stage with a very long tail (data skew)
- Many small stages that could be coalesced (too many shuffles)
- Stages with no parallel computation (single partition — `collect()` or `coalesce(1)`)
- Repeated shuffle patterns (could be optimised with caching or bucketing)

---

## 13. Common Tuning Scenarios by Use Case

### 13.1 ETL (Read → Transform → Write)

**Common bottlenecks:**
- Reading many small files (metadata overhead)
- Shuffle during transformations
- Writing many small files (HDFS small-file problem)

**Recommended configuration:**
```properties
# File reading — pack small files into fewer partitions
spark.sql.files.maxPartitionBytes=256m
spark.sql.files.openCostInBytes=8m

# Shuffle
spark.sql.shuffle.partitions=ceil(total_data_gb / 0.2)  # 200 MB per partition
spark.sql.adaptive.enabled=true
spark.sql.adaptive.coalescePartitions.enabled=true

# Write — use coalesce to control output file count
# In code: df.coalesce(num_output_files).write.parquet("...")
```

**Best practices:**
- Use columnar formats: **Parquet** or **ORC** with compression (`zstd` or `snappy`)
- Coalesce before write to produce files of 128–256 MB each
- Enable AQE for automatic partition coalescing
- Compact small files in source if possible (this is a data engineering concern upstream)

### 13.2 ML Training (Iterative Algorithms)

**Common bottlenecks:**
- Repeated data loading across iterations
- Large broadcast (model parameters)
- Shuffle in gradient aggregation

**Recommended configuration:**
```properties
# Cache training data (reused each iteration)
spark.sql.adaptive.enabled=true
spark.serializer=org.apache.spark.serializer.KryoSerializer
spark.kryo.classesToRegister=org.apache.spark.ml.linalg.Vector,org.apache.spark.ml.linalg.DenseVector

# Driver memory for model collection
spark.driver.memory=16g

# Executor — memory heavy
spark.executor.memory=32g
spark.executor.cores=4
```

**Best practices:**
- **Cache training data** with `MEMORY_AND_DISK` (avoid recomputing across iterations)
- **Use DataFrame-based ML** (`spark.ml`) — better optimisation than RDD-based MLlib
- **Unpersist intermediate results** after each iteration
- **Increase driver memory** for model collection (some algorithms collect parameters to driver)
- **Enable Kryo serialization** for RDD-based MLlib (spark.mllib)

### 13.3 Joins (Fact × Dimension)

**Common bottlenecks:**
- Data skew (some keys dominate)
- Large shuffle volumes
- OOM on broadcast (dimension table too big)

**Recommended configuration:**
```properties
# AQE for automatic skew handling and strategy switching
spark.sql.adaptive.enabled=true
spark.sql.adaptive.skewJoin.enabled=true
spark.sql.adaptive.skewJoin.skewedPartitionFactor=5
spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes=256m

# Broadcast — increase if driver memory permits
spark.sql.autoBroadcastJoinThreshold=50m

# Bucketing
spark.sql.bucketing.enabled=true
```

**Best practices:**
- **Broadcast small dimension tables** — use broadcast hints
- **Bucket fact and dimension tables** by the join key to avoid shuffle entirely
- **Enable AQE skew join** for automatic skew handling
- **Salt extreme skew** manually when AQE is insufficient
- **For very large dimension tables** (hundreds of GB), stick with SMJ and focus on partition balance

### 13.4 Aggregations

**Common bottlenecks:**
- Shuffle for `groupBy`
- Memory pressure for hash map (high-cardinality groups)
- Spilling during aggregation

**Recommended configuration:**
```properties
spark.sql.adaptive.enabled=true
spark.sql.adaptive.coalescePartitions.enabled=true
spark.sql.shuffle.partitions=ceil(aggregated_data_size / 0.2)  # 200 MB per partition
```

**Best practices:**
- **Increase partition count** for high-cardinality grouping keys
- **Enable AQE** for adaptive partition coalescing after aggregation
- **Use `reduceByKey`** instead of `groupByKey` for RDD-based code (map-side combine)
- **Use approximate functions** where exact counts are not needed: `approx_count_distinct`, `percentile_approx`
- **Pre-aggregate** upstream when possible (reduce data before shuffle)

### 13.5 Window Functions

**Common bottlenecks:**
- Shuffle for `PARTITION BY`
- Sort for `ORDER BY`
- Memory pressure for wide windows (many rows per partition)

**Recommended configuration:**
```properties
# Increase executor memory for wide windows
spark.executor.memory=32g

# Ensure sufficient parallelism
spark.sql.shuffle.partitions=ceil(data_size / 0.2)
```

**Best practices:**
- **Increase executor memory** for wide windows (many rows per partition)
- **Ensure partition column cardinality is balanced** — avoid one partition containing 90% of data
- **Filter before the window** to reduce data volume
- **Check if unsorted windows are feasible** — if ordering is not needed, remove `ORDER BY` to skip the sort step
- **A single window with `PARTITION BY` and `ORDER BY`** triggers one shuffle and one sort. Multiple windows on the same partition key share the same shuffle.

### 13.6 Graph Processing (GraphX)

**Common bottlenecks:**
- Message passing (aggregation and distribution across partitions)
- Multiple iterations (each iteration triggers shuffles)
- Checkpointing overhead in long iterative computations

**Recommended configuration:**
```properties
# Kryo for vertex/edge types
spark.serializer=org.apache.spark.serializer.KryoSerializer
spark.kryo.classesToRegister=org.apache.spark.graphx.impl.EdgeRDDImpl,org.apache.spark.graphx.impl.VertexRDDImpl

# Cache across iterations
# GraphX internally manages caching — ensure vertices/edges are cached
# Use MEMORY_AND_DISK to avoid OOM
```

**Best practices:**
- **Use checkpoints** for iterative graph algorithms (break lineage after N iterations)
- **Cache vertices/edges** across iterations (GraphX does this, but verify with Spark UI)
- **Choose the right partition strategy:** `PartitionStrategy.EdgePartition2D` for skewed graphs, `CanonicalRandomVertexCut` for uniform graphs
- **Kryo serialization** for vertex and edge types is essential for performance
- **Monitor spill** in Spark UI — graph algorithms are memory-intensive

### 13.7 Structured Streaming (Kafka → Spark)

**Common bottlenecks:**
- Balancing latency vs throughput
- State store size (stateful processing)
- RocksDB configuration for stateful streams
- Watermark and late data handling

**Recommended configuration:**
```properties
# Trigger interval
# "ProcessingTime/10s" or "AvailableNow" for batch-style

# State store
spark.sql.streaming.stateStore.providerClass=org.apache.spark.sql.execution.streaming.state.RocksDBStateStoreProvider
spark.sql.streaming.stateStore.rocksdb.compactOnCommit=true

# Checkpoint
# spark.sql.streaming.checkpointLocation = ...

# Kafka offsets
spark.streaming.kafka.maxRatePerPartition=1000   # rate limit per partition
```

**Best practices:**
- **Use RocksDB state store** for large state (default in Spark 3.4+). Configure RocksDB compaction settings.
- **Set appropriate watermarks** to control late data handling and state retention
- **Use `AvailableNow` trigger** for batch-style processing with automatic partitioning
- **Checkpoint frequently** to resilient storage for fault tolerance
- **Monitor state store size** in Spark UI Streaming tab
- **Co-locate Kafka partitions with Spark partitions** when possible (avoid unnecessary re-partitioning)
- **Rate-limit with `maxRatePerPartition`** to avoid overwhelming downstream systems

---

## 14. Scenario-Specific Configuration Reference

| Scenario | Config Adjustments | Typical Settings |
|---|---|---|
| **ETL — Small dataset (< 100 GB)** | Defaults + AQE | `executor.memory=16g`, `executor.cores=4`, `shuffle.partitions=200`, AQE on |
| **ETL — Large dataset (1–10 TB)** | Higher partitions, AQE, zstd compression | `executor.memory=32g`, `executor.cores=4`, `shuffle.partitions=1000`, `maxPartitionBytes=256m`, codec=zstd |
| **ETL — Many small files** | Increase `maxPartitionBytes`, `openCostInBytes` | `maxPartitionBytes=256m`, `openCostInBytes=16m`, coalesce before write |
| **ML — Logistic Regression (iterative)** | Cache data, Kryo, high driver memory | `executor.memory=32g`, `driver.memory=16g`, Kryo, `MEMORY_AND_DISK` cache |
| **ML — KMeans / clustering** | Cache data, Kryo, many iterations | Same as logistic regression + checkpoint every 10 iterations |
| **Star Schema Join** | Broadcast dimension tables, bucketing, AQE skew join | `autoBroadcastJoinThreshold=50m`, bucket fact and dim on key, `skewJoin.enabled=true` |
| **High-Cardinality Aggregation** | High partition count, AQE coalesce | `shuffle.partitions=2000`, AQE with `advisoryPartitionSizeInBytes=64m` |
| **Wide Window Function** | High executor memory, balanced partition key | `executor.memory=64g`, `shuffle.partitions=500`, verify partition balance |
| **Graph Processing (PageRank)** | Kryo, checkpoint, cache vertices/edges | Kryo with registered classes, checkpoint every 5 iterations, `MEMORY_AND_DISK` |
| **Structured Streaming — Low latency (< 1 s)** | Small batches, fast state store | Trigger `ProcessingTime/500ms`, heap state store (or RocksDB for larger state) |
| **Structured Streaming — High throughput** | Larger batches, RocksDB, AvailableNow | Trigger `AvailableNow` or `ProcessingTime/30s`, RocksDB state store |
| **Data Skew — General** | AQE skew join, salting | `skewJoin.enabled=true`, `skewedPartitionFactor=5`, salting for extreme cases |
| **Memory Constrained Environment** | Reduce cache, use MEMORY_ONLY_SER, reduce overhead | `memory.fraction=0.7`, use `MEMORY_AND_DISK_SER`, `executor.memory=16g`, `executor.cores=2` |
| **High Concurrency (many concurrent jobs)** | Dynamic allocation, moderate executor sizes | `dynamicAllocation.enabled=true`, `maxExecutors=50`, `executor.memory=16g`, `executor.cores=4` |

---

## 15. Tuning Workflow Checklist

### Phase 1: Establish a Baseline

- [ ] Run the job with **default settings** (Spark defaults + AQE enabled)
- [ ] Note the total duration for each job/stage
- [ ] Check Spark UI for spills, GC time, task distribution
- [ ] Identify the **longest stage** — this is the bottleneck
- [ ] Identify any **failed tasks** and their error messages

### Phase 2: Analyse the Bottleneck

- [ ] **Spills detected?** → Partition count or memory issue
- [ ] **GC time > 15%?** → Memory configuration or serialization issue
- [ ] **Task duration skewed?** → Data skew
- [ ] **Scheduler delay high?** → Not enough executors / cluster contention
- [ ] **Shuffle remote reads dominate?** → Locality issue
- [ ] **Input size per task unbalanced?** → Wrong partitioning

### Phase 3: Apply Tuning

| Bottleneck | First Action | Second Action | Third Action |
|---|---|---|---|
| **Spills** | Increase partition count | Increase executor memory | Increase `memory.fraction` |
| **GC pressure** | Use `MEMORY_ONLY_SER` | Increase executor memory | Enable Kryo serialization |
| **Data skew** | Enable AQE skew join | Apply salting | Increase partitions |
| **Slow shuffle** | Tune compression (`zstd`) | Increase `reducer.maxSizeInFlight` | Increase partition count |
| **Too many output files** | `coalesce()` before write | Enable AQE coalesce | Compact upstream |
| **High scheduler delay** | Increase executors | Enable dynamic allocation | Check cluster capacity |

### Phase 4: Verify

- [ ] Re-run with tuned settings
- [ ] Compare duration against baseline
- [ ] Check Spark UI: spills eliminated? GC time reduced? Task distribution even?
- [ ] Confirm output quality unchanged (row counts match)
- [ ] If not improved, **revert the change** and try the next hypothesis

### Phase 5: Automate & Document

- [ ] Save the working configuration in the SparkSession builder or a config file
- [ ] Document why each non-default config was chosen (future you will thank you)
- [ ] Consider parameterising the config (e.g., data-size-based partition count logic)
- [ ] Set up monitoring alerts for commonly broken metrics (e.g., spill events)

---

## Quick Reference: The 10 Most Impactful Configurations

Ranked by impact on real-world Spark job performance:

1. **`spark.sql.adaptive.enabled=true`** — enables AQE (automatic partition coalescing, skew join, join strategy switching)
2. **`spark.sql.shuffle.partitions`** — set based on data volume (target 100–200 MB per partition)
3. **`spark.executor.memory` + `spark.executor.memoryOverhead`** — right-size the JVM for your workload
4. **`spark.executor.cores`** — 4–5 per executor, never more
5. **`spark.sql.adaptive.skewJoin.enabled=true`** — fix data skew automatically
6. **`spark.sql.autoBroadcastJoinThreshold`** — broadcast dimension tables (if driver memory permits)
7. **`spark.serializer=KryoSerializer`** — 10× faster serialization for RDD-based code
8. **`spark.dynamicAllocation.enabled=true`** — right-size cluster resources dynamically
9. **`spark.io.compression.codec=zstd`** — better shuffle compression (30–50% smaller)
10. **`spark.sql.files.maxPartitionBytes`** — control partition size when reading files

---

> **Final word:** No guide can replace empirical testing. Every cluster, dataset, and workflow has unique characteristics. Use this guide as a structured checklist — measure first, change one thing, measure again, and always let the data (and the Spark UI) tell you what to do next.
